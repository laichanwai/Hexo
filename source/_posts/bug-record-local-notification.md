---
title: 记录一个很神奇的 BUG
tags:
  - Blog
date: 2018-03-28 16:13:34
permalink: bug_record_local_notification
---

在 ios 11 之后，在 `applicationDidEnterBackground` 加入这段代码，会导致 app 的生命周期异常。

这个 bug 会在某次（一般是第一次）退出时候会保留当前页的内容，app 杀死，重新打开之后，启动图会变成这个保留页。

这个 bug 目前还在研究中，`某次退出` 的时机还不不准确，具有相当的随机性。 

```objc
- (void)applicationDidEnterBackground:(UIApplication *)application {
    UILocalNotification *noti = [[UILocalNotification alloc] init];
    noti.applicationIconBadgeNumber = -1;
    [[UIApplication sharedApplication] scheduleLocalNotification:noti];
}
```



