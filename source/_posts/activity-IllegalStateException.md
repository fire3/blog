title: "该解决这个IllegalStateException的问题了"
date: 2015-04-07 23:05:30
tags: [android]
---

[水手放映室](http://www.sailorcast.com) 偶尔会遭遇这个问题：

```
java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState
   at android.support.v4.app.FragmentManagerImpl.checkStateLoss(FragmentManager.java:1365)
   at android.support.v4.app.FragmentManagerImpl.enqueueAction(FragmentManager.java:1383)
   at android.support.v4.app.BackStackRecord.commitInternal(BackStackRecord.java:636)
   at android.support.v4.app.BackStackRecord.commit(BackStackRecord.java:615)
   at com.crixmod.sailorcast.view.AlbumDetailActivity$3.run(AlbumDetailActivity.java:343)
```

这个错误造成的原因是因为在异步的网络请求返回后操作了fragment的commit。该请求返回时可能当前的activity已经在Pause或者Stop状态了，就会报这个状态不一致的异常。详细解释参见：
http://www.androiddesignpatterns.com/2013/08/fragment-transaction-commit-state-loss.html

准备参考 http://stackoverflow.com/questions/8040280/how-to-handle-handler-messages-when-activity-fragment-is-paused  这篇文章来解决一下这个问题。


