---
title: 记录一个很神奇的 BUG
tags:
  - Blog
date: 2018-03-28 16:13:34
permalink: bug_record_local_notification
---

### 问题描述

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

---

### 2018年03月28日17:36 补充：

经过一番测试，目前已经确定这个 bug 会导致的问题：
    
#### 1. app 不退出
    
我写了个空白项目，然后发现，就算已经杀死了 app（双击 home 然后上滑退出）

![2018032843529IMG_0752C6E81934-1.jpg](http://storage.laizw.cn/image/upi/2018032843529IMG_0752C6E81934-1.jpg-watermark)
    
但是仍然能够查看到这个 app 的进程，这说明，它仍在后台。
    
![20180328152223141358041.png](http://storage.laizw.cn/image/upi/20180328152223141358041.png-watermark)
    
真是十分诡异。。。
    
#### 2. app 的运行状态异常
    
这里主要影响的方法是 `- (void)applicationWillEnterForeground:(UIApplication *)application`
    
遇到这个 bug 之前，我开发过的所有 app 中，点击 app 到进入到主页面的状态过程有两种 (简略):

1. `didFinishLaunching` -> `DidBecomeActive`

2. `WillEnterForeground` -> `DidBecomeActive`

按照我的理解 `didFinishLaunching` 和 `WillEnterForeground` 是不会同时出现的。app 杀死之后，就没有后台这个状态了。

> 然后，遇到这个诡异的 bug ... 颠覆了我的认知 ...
    
关于这个方法，官方给出的说法是：

> applicationWillEnterForeground: Lets you know that your app is moving out of the background and back into the foreground, but that it is not yet active.
    
当 app 从后台切换到前台的时候，AppDelegate 会响应这个方法。那杀死 app 重新打开算什么状态？
    
一番测试之后发现它的状态是：

`didFinishLaunching` -> `WillEnterForeground` -> `DidBecomeActive` 

这是不是能猜测此时 app 即是启动，也是从后台进入到前台？（# 我晕了... #）




