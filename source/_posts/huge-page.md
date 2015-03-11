title: "Linux内核大页分析"
date: 2015-03-06 08:20:30
tags: [kernel, "huge page"]
---

最近要实现一个技术，打算借鉴透明大页的实现方式，将Transparent HugePage使用的内存限制在特定的内存区域。需要先分析清楚原有Transparent HugePage的实现机制。

<!-- more -->

### 透明大页的申请

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

### 透明大页的释放

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

### hugetlbfs大页的申请

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


### 透明大页使用静态大页的空间

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
