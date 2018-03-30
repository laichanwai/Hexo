---
title: 记录一个很神奇的 BUG
tags:
  - Blog
date: 2018-03-28 16:13:34
permalink: bug_record_local_notification
---

## 问题描述

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

### *补充说明一下这段代码的作用：*

`UIApplication` 中有一个属性 `applicationIconBadgeNumber`，它是用来**设置** app 角标的。

`UILocalNotification` 也有一个相同的属性 `applicationIconBadgeNumber`，但是它的作用不一样，它是用来**修改** app 角标的，为 0 代表不改变。

一旦通过 `UIApplication` 将程序的角标从非零置为零，就会清空通知栏的所有通知。如果想**清除角标但不清空通知栏**，有如下方法。


1. 发送一条远程推送，推送内容只有badge，并将badge的值设为负数。此时程序角标会消失但是通知栏的推送消息不清除。

2. 同样的方法，发送一条本地推送。

---


## bug 具体表现:

经过一番测试，目前已经确定这个 bug 具体表现：
    
### 1. app 不退出
    
使用一个空白项目 + bug 代码片段，测试**杀死**app（双击 home 然后上滑退出）

![2018032843529IMG_0752C6E81934-1.jpg](http://storage.laizw.cn/image/upi/2018032843529IMG_0752C6E81934-1.jpg-watermark)
    
所有后台已经退出，剩下设置，但是通过 Xcode -> Debug -> Attach to Process 仍然能够查看到这个 app 的进程，这说明，**它仍在后台**。
    
![20180328152223141358041.png](http://storage.laizw.cn/image/upi/20180328152223141358041.png-watermark)
    
真是十分诡异。。。
    
### 2. app 的运行状态异常
    
会有影响的方法是 `- (void)applicationWillEnterForeground:(UIApplication *)application`

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

这是不是能猜测此时 **app 即是启动，也是从后台进入到前台？**


## 出现 bug 具体的原因:

> 逐渐接近真相了。。。

基于之前出现的问题，我这次补充出现 bug 具体的原因。

### 1. 执行的时机问题:

- 将这段代码放到 `didFinishLaunching`: 

    ✅ 进程退出，app 运行状态正常

- 将这段代码放到 `WillEnterForeground`: 

    ✅ 进程退出，app 运行状态正常
    
- 将这段代码放到 `DidBecomeActive`: 

    ✅ 进程退出，app 运行状态正常
    
- 将这段代码放到 `WillResignActive`: 

    ✅ 进程退出，app 运行状态正常
    
- 将这段代码放到 `WillTerminate`: 

    ✅ 进程退出，app 运行状态正常
    
- 将这段代码放到 `rootViewController` 的 `viewDidLoad`: 

    ✅ 进程退出，app 运行状态正常

- 将这段代码放到 `DidEnterBackground`: 

    ❌ 进程不退出，app 运行状态异常

### 2. notification 设置问题:

先将这段代码放到 `DidEnterBackground` 中，然后修改 `applicationIconBadgeNumber` 的值

- `applicationIconBadgeNumber = -1`:
    
    ❌ 进程不退出，app 运行状态异常
    
- `applicationIconBadgeNumber = 0`:

    ✅ 进程退出，app 运行状态正常
    
- `applicationIconBadgeNumber = 1`:
    
    ❌ 进程不退出，app 运行状态异常
    
- 删掉代码，`applicationIconBadgeNumber` 不赋值

    ✅ 进程退出，app 运行状态正常


## 修改发通知的方法

`UILocalNotification` 的发送方式有两种，分别是 

- `- (void)presentLocalNotificationNow:(UILocalNotification *)notification`
- `- (void)scheduleLocalNotification:(UILocalNotification *)notification`

但是很遗憾，两种方式都是会出现这个 bug。


## 机型或者系统版本问题

苹果官方在 iOS 10 的时候推出 `UserNotifications Framework`，我们验证一下这个问题是不是新 api 导致的，同时用多设备来验证一番

- iPhone 6 iOS 8.1 
    
    ✅ 测试通过

- iPhone 6 iOS 9.0
    
    ✅ 测试通过

- iPhone 6 iOS 11.2.6
    
    ❌ 测试未通过

- iPhone 7 Plus iOS 10.0
    
    ✅ 测试通过

- iPhone 8 Plus iOS 11.2
    
    ❌ 测试未通过

- iPhone X iOS 11.2
    
    ❌ 测试未通过

明显看出，这个 bug **跟机型无关**，而且仅仅会在 **iOS 11 之后**的系统中出现。


--- 


## bug 不止，生命不息，未完待续...
