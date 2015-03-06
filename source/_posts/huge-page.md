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


