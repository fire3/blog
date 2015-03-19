title: "段页结合的内存管理设计"
date: 2015-03-06 08:20:30
tags: [kernel, "huge page"]
---

### 序言

默认情况下，Linux操作系统是按照页面来管理内存使用的。当用户申请内存时，并不会真正的分配内存，而是根据“Page Fault”来触发异常，进而按照情况分配内存。用户的虚地址连续，而真实使用的物理地址不一定连续，通过“TLB”来进行虚实地址的映射。但在某些情况下，为了一些设备或特殊的计算部件需要，这些部件需要使用大块的连续物理内存，如果动辄数GB，默认的内存管理就不能满足需要了。一般情况下，在系统运行一段时间后，均不能保证可以正常申请到连续的大块物理内存。针对这类应用场景，最常见的做法是在开机阶段预留内存。但预留的内存没有灵活性，成为了专用内存，不能给普通的用户程序使用，即便能够使用，也不能通过标准的接口申请，而是通过自定义的特殊接口申请。所以，在这种场景下，需要一种更加灵活的内存管理策略，将段式内存管理和页式内存管理结合起来。

<!-- more -->

### 设计目标

段页结合的内存管理设计目标是：

* 系统在开机阶段预留大量内存，进入段式管理的内存资源池。
* 针对段式管理的内存空间，支持自定义的段式内存申请、释放接口。
* 在其它内核组件不使用段式空间，或段式空间有余量时，普通用户程序可通过标准接口使用到段式管理的内存资源池中的内存空间。


### 方案

#### 开机阶段预留方案

首先操作系统内核需要能够识别全部内存，然后在开机阶段利用bootmem或者memblock接口进行内存空间的预留。比如利用接口:

```
__alloc_bootmem_nopanic(unsigned long size, unsigned long align,
                                        unsigned long goal)
```

操作系统内核识别全部内存的用意在于memmap中要建立起所有的page结构,后续才可以用`pfn_to_page`之类的接口得到对应的page结构，有了page结构，才好在通用的内存管理框架中进行代码集成。

#### 段页结合设计方案

在最普通的内存管理接口中去结合段式管理是不现实的，而且小页面会导致段式空间使用过于零碎。将段页结合起来，我们重点关注大页的实现。下面首先介绍一下内核中的透明大页机制。

某些对性能敏感的，并且操作大量内存的课题已经可以通过大页和libhugetlbfs库来优化性能，但是libhugetlbfs在将malloc或者shm_get空间转化成大页使用时，如果大页无法申请到的话，将会报错，并导致程序运行错误。而Transparent Hugepage是另外一种使用大页进行优化的方式，它支持自动的使用大页，并且在大页不够的情况下降级为小页，保证对用户透明。

目前Transparent Hugepage（下文简称THP）仅针对匿名内存。匿名内存是一类没有关联文件或设备的内存。用户程序中的栈、堆空间均属于匿名内存。一般情况下，匿名内存申请时最先只分配“虚空间”，当读操作触发“页面故障”时，操作系统会先建立一个指向Zero Page的映射，并在页表中设置写故障位。当用户真正的去写这个页面时，才会分配真正的物理页面。在使用`mmap()`系统调用时，通过传递`MAP_ANONYMOUS`标志来申请匿名页面。

利用Transparent Hugepage的机制实现段页结合的内存管理是在现有内存管理框架下最为简单易行的方式。透明大页实现时在常规的物理页面申请释放路径上增加了一层特殊处理，接口独立且相对完善。我们进行段页结合管理的核心思想是让透明大页申请时可以从我们管理的段式空间资源池中申请，调用我们的申请接口，而不是通用的申请释放接口。



#### 透明大页的申请

以x86 3.8.0内核为例，进入主要的Page Fault入口在 arch/x86/mm/fault.c，函数 __do_page_fault，进过了一长串的统计与合法性检查后，进入体系结构无关的中，如下:

``` c
    fault = handle_mm_fault(mm, vma, address, flags);
```

进入透明大页的处理入口

``` c
    pgd = pgd_offset(mm, address);
    pud = pud_alloc(mm, pgd, address);
    if (!pud)
        return VM_FAULT_OOM;
    pmd = pmd_alloc(mm, pud, address);
    if (!pmd)
        return VM_FAULT_OOM;
    if (pmd_none(*pmd) && transparent_hugepage_enabled(vma)) {
        if (!vma->vm_ops)
            return do_huge_pmd_anonymous_page(mm, vma, address,
                              pmd, flags);
    } else {
    // more code
    }
```

这里我们可以看出，进入透明大页处理的3个先决条件。

* pmd_none(*pmd) 大页的页表标志是放在PMD的，只有还没有初始化PMD的页面才有资格成为大页。
* transparent_hugepage_enabled(vma) 根据vma判断透明大页是否开启。
* (!vma->vm_ops) 代表了匿名页面，只有匿名页面才可以用透明大页实现。


进入 `do_huge_pmd_anonymous_page` 函数后，进行核心的透明大页申请处理。

首先判断虚空间是否满足条件，只有当缺页地址所在的虚空间还有超过一个大页的大小时，才会真正进入大页处理流程。判断条件如下:

``` c
    if (haddr >= vma->vm_start && haddr + HPAGE_PMD_SIZE <= vma->vm_end) {
    }
```

后面一个关键的判断是:

``` c
   if (!(flags & FAULT_FLAG_WRITE) && transparent_hugepage_use_zero_page()) {}
```

如果是读故障进来的，则说明触发缺页的操作是读指令，同时启用了zero page功能的话，应当使用zero page来完成该次映射。这是符合Linux通用的页面管理规范的。
首次是读的页面没有必要建立一个新的页面，指向zero page即可。但是大页需要有专用的zero page实现。

在后面可以看到关键的大页申请函数：

``` c
    page = alloc_hugepage_vma(transparent_hugepage_defrag(vma),
                                vma, haddr, numa_node_id(), 0);
```

然后再调用如下接口对申请到的大页page进行设置：

``` c
    __do_huge_pmd_anonymous_page(mm, vma, haddr, pmd,page)
```

另外在 `do_huge_pmd_anonymous_page` 函数中，还需要针对不满足大页条件的情况进行处理，主要是需要调用`handle_pte_fault`接口。

#### 透明大页的释放

透明大页在申请时是一个compound page类型，这里有必要先说明一下compound page的含义：

* order大于０的page是compound page.
* 第一个页面称之为"head page".
* 第一个页面后面的页面称之为 "tail pages".
* 所有的页面都有PG_compound 标志.
* 所有的tail pages都有指向head page的指针.

初始化一个compound page可以在 `mm/page_alloc.c`中看到`prep_compound_page`函数。

``` c
  void prep_compound_page(struct page *page, unsigned long order)
  {
      int i;
      int nr_pages = 1 << order;

      set_compound_page_dtor(page, free_compound_page);
      set_compound_order(page, order);
      __SetPageHead(page);
      for (i = 1; i < nr_pages; i++) {
          struct page *p = page + i;
          __SetPageTail(p);
          set_page_count(p, 0);
          p->first_page = page;
      }
  }
```

注意其中的接口 `set_compound_page_dtor`，dtor是page中一个用于释放操作的关键函数指针，可以看到`free_compound_page`是这类组合页面的析构函数`dtor`。
可以在 `mm/swap.c`  中看到`dtor`的用法：

``` c
  static void __put_compound_page(struct page *page) {
     compound_page_dtor *dtor;
     __page_cache_release(page);
     dtor = get_compound_page_dtor(page);
     (*dtor)(page);
  }
```

在释放操作进行时会调用这个dtor接口。这是后续实现自定义透明大页释放的关键接口。

#### hugetlbfs大页的申请

hugetlbfs的大页是需要预留的，或者在开机时通过cmdline参数`hugepages=XXX`来设置。
预留hugetlbfs大页时在 mm/hugetlb.c 中的如下接口申请:

``` c
  static struct page *alloc_fresh_huge_page_node(struct hstate *h, int nid)
  {
      struct page *page;

      if (h->order >= MAX_ORDER)
          return NULL;

      page = alloc_pages_exact_node(nid,
          htlb_alloc_mask|__GFP_COMP|__GFP_THISNODE|
                          __GFP_REPEAT|__GFP_NOWARN,
          huge_page_order(h));
      if (page) {
          if (arch_prepare_hugepage(page)) {
              __free_pages(page, huge_page_order(h));
              return NULL;
          }
          prep_new_huge_page(h, page, nid);
      }

      return page;
  }
```

初始化一个预留大页，注意接口 `set_compound_page_dtor`，这里设置了释放时需要调用的函数 `free_huge_page`。

``` c
  static void prep_new_huge_page(struct hstate *h, struct page *page, int nid)
  {
      INIT_LIST_HEAD(&page->lru);
      set_compound_page_dtor(page, free_huge_page);
      spin_lock(&hugetlb_lock);
      set_hugetlb_cgroup(page, NULL);
      h->nr_huge_pages++;
      h->nr_huge_pages_node[nid]++;
      spin_unlock(&hugetlb_lock);
      put_page(page); /* free it into the hugepage allocator */
  }
```

释放函数`free_huge_page`只是从预留大页资源里面释放，将页面置为可被申请的状态，并不是真正的释放预留的大页资源。
释放预留大页资源的接口在`set_max_huge_pages`.


预留好大页后，使用接口`alloc_huge_page`申请大页页面。


#### 透明大页使用静态大页的空间

理论上，透明大页使用预留的静态大页的空间是完全可能的，这里我们先做一个简单的实现。
`mm/hugetlb.c`中的`alloc_huge_page`有些过于复杂，我们实现一套简单的申请释放接口来验证思路：

``` c
  struct page *alloc_huge_page_thp(struct vm_area_struct *vma, unsigned long addr) {
      struct hstate *h = &default_hstate;
      struct page * page;
      spin_lock(&hugetlb_lock);
      page = dequeue_huge_page_node(h,0);
      spin_unlock(&hugetlb_lock);
      if (page) {
          set_compound_page_dtor(page, free_huge_page_thp);
          return page;
      } else
          return ERR_PTR(-ENOSPC);
  }

  void free_huge_page_thp(struct page *page) {
      struct hstate *h = &default_hstate;
      set_page_private(page, 0);
      page->mapping = NULL;
      BUG_ON(page_count(page));
      BUG_ON(page_mapcount(page));
      spin_lock(&hugetlb_lock);
      enqueue_huge_page(h, page);
      spin_unlock(&hugetlb_lock);
  }

```

可以在`alloc_hugepage_vma` 接口中调用我们实现的`alloc_huge_page_thp`函数，返回申请到的页面即可。另外，还需要修改
`__do_huge_pmd_anonymous_page`函数中的`page_add_new_anon_rmap`方法，不能使用这个方法设置rmap，需要使用`hugepage_add_new_anon_rmap`。
如果使用原来的方法，会改变`page->lru`，使得静态大页的申请释放接口失效。
