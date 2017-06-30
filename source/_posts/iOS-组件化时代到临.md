---
title: iOS 组件化时代到临
tags:
  - Blog
  - iOS
date: 2017-01-04 14:07:52
permalink: ios-component
---

## 前言

iOS 组件化的时代已经到临，但是在组件化的道路上确还存在很多分歧的意见。

关于组件的概念，我这里不多说了，大家可以看下面三位大大文章，他们对组件的理解十分到位。

Limboy 的 [蘑菇街 App 的组件化之路 http://limboy.me/tech/2016/03/10/mgj-components.html](http://limboy.me/tech/2016/03/10/mgj-components.html)
Casa 的 [iOS应用架构谈 组件化方案 http://casatwy.com/iOS-Modulization.html](http://casatwy.com/iOS-Modulization.html)
lincode 的 [豆瓣App的模块化实践 http://lincode.github.io/Modularity](http://lincode.github.io/Modularity)

## 分析

- **Limboy**

 1. 组件通过 URL 管理，做了一个专门的后台来管理 URL。
 2. 所有的模块都是依赖于 URL、 [MGJRouter](https://github.com/mogujie/MGJRouter) 和 `ModuleManager`。
 3. URL 和模块需要注册才能使用。

- **Casa**

 1. 模块依赖 `CTMediator`。
 2. 利用 `Target-Action` 的方式调用模块组件，`Target-Action` 在分类中实现，利用 `runtime` 来调用。
 3. 区分远程和本地调用入口

- **lincode**

 1. 模块之间使用 `FRDModuleManager` 管理。
 2. 使用 `FRDIntent` 对 `ViewController` 进行解耦。
 3. 可以通过 `className` 来调用 `ViewController`
 4. 使用 `plist` 来管理 `URL <-> Module` 和 `URL <-> ViewController`。

三位大大理解上有个共同点，就是把模块拆分成独立的 `repo`，可以利用 `cocoapods` 来管理。

其中 Limboy 和 lincode 的实现方案都是需要一个 `ModuleManager` 来管理模块，而且都是可以通过 URL 的方式调用组件。
Casa 是通过 `Target-Action` 的方式调用组件，所以只需要维护一个公共的 `Categories` 和模块内独立的 `Actions`。

对于三位大大的方案，我几点提出个人看法：

1. Limboy 的方案灵活，但是有点过于依赖 URL，而且 URL 的维护成本过于高。
2. Casa 的方案实现过程有点繁琐，对于新上手的开发者来说学习成本有点高。而且我觉得，维护 `Target-Action` 的分类和维护 URL 其实是殊途同归的，`Target-Action` 的维护成本要比 URL 高。
3. 我对实现组件化的理解跟 lincode 大大的相似，只是方式不一样。

## 组件化实践

### 模块

对于模块的划分，最好的方式就是拆分成独立的 repo，每个 repo 都能单独编译。但是在拆分模块的时候会发现困难重重，细分下去的时候会发现，某些功能到底属于 A 模块还是 B 模块，这就需要好好商榷了。

在划分模块前期，可以先在项目中拆分成一个一个 section，等到这个 section 拆分完了，就可以把它挪出去，用一个独立的 repo 来管理。

### YFKit

> 我写 `YFKit` 的初衷，是为了能够快速搭建起一个 app。

在 `YFKit` 中，远程和本地的逻辑是分开的，远程的是使用 `YFRouter` 来管理 `URL`；本地的是通过 `YFMediator` 和 `className` 来调用组件。

使用 `URL` 的成本比较高，因为每个 `URL` 都需要注册，而且还得对 `URL` 进行管理。

而使用 `className` 的话，开发者只需要专注自己的组件和规定好传递的参数即可，调用着，只需要知道对应组件的 `className` 和传递的参数就可以了。

##### YFMediator

`YFMediator` 是管理本地 `ViewController` 的跳转，更多功能可以看这里：[https://github.com/laichanwai/YFMediator](https://github.com/laichanwai/YFRouter)

通过 `className` 创建 `ViewController`。

```objc
UIViewController *vc = [[YFMediator shared] viewController:@"ViewController" params:@{@"name" : @"vc"}];
```

也可以利用 `YFMediator` 快捷方法做跳转，`Push` 或者 `Present`。

```objc
[[YFMediator shared] push:@"ViewController" animate:YES params:nil];
[[YFMediator shared] present:@"ViewController" animate:YES params:@{@"id" : @1}];
```

如果同时也使用了 `YFRouter`，那么可以利用 `YFRouter` 来进行 `URL` <=> `ViewController` 的绑定。

```objc
// binding
[[YFMediator shared] mapURL:@"/login" toViewController:@"LoginViewController"];
// jump
[[YFMediator shared] present:@"/login" animate:YES params:@{
                                                            @"mobile" : @"13500000000",
                                                            @"password" : @"123456"
                                                            }];
```

*注: `YFMediator` 在版本 0.1.3 之后和 `YFRouter` 分离，需要使用 `URL` 绑定的请自行添加 `YFRouter`。

##### YFRouter 

`YFRouter` 专门用来处理 `URL`。更多功能可以看这里：[https://github.com/laichanwai/YFRouter](https://github.com/laichanwai/YFRouter)

`URL` 是各平台协商使用 `Target-Action` 的方式：`scheme://target/action?query`。

```objc
[YFRouter registerURL:@"/:target/:action" handler:^(NSDictionary *params) {
    id target = params[@"target"];
    id action = params[@"action"];

    // 根据 target 和 action 来处理业务
    // ...
}];

[YFRouter route:@"/user/login" params:...];
[YFRouter route:@"/user/search?key=laizw" params:...];
```

## 尾声

`YFMediator` + `YFRouter` 这套方案适用于中型以上的 App，如果一款 App 只有少数接口和页面的话，其实是没有必要组件化的，这样子反而加大了劳动力。

如果你发现你的 App 里面出现了大量的 `#import "XXXViewController.h"`，不妨尝试使用 `YFMediator` 来解放你的双手。

如果你发现你的 App 里面需要服务端通过 URL 来控制业务的话，不妨尝试一下上述的 `YFRouter` 方案。

如果你觉的我的代码写的不错，记得给我个 [star](https://github.com/laichanwai) ~