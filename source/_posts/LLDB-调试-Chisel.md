---
title: LLDB 调试 - Chisel
tags:
  - iOS
  - DEBUG
permalink: lldb-debug-chisel
id: 23
updated: '2016-08-24 16:04:33'
date: 2016-08-23 16:41:36
---

## Install

Chisel github: [https://github.com/facebook/chisel](https://github.com/facebook/chisel)

```shell
brew update
brew install chisel
```

在安装完之后会显示安装路径, 这里的 `/path/to/fblldb.py`就是指的那个路径.

```shell
# 在 ~/.lldbinit 中加入下面这句话
command script import /path/to/fblldb.py
```

```shell
# 在命令行中输入
source ~/.lldbinit
```

## Commands

|Command          |Description     |iOS    |OS X   |
|-----------------|----------------|-------|-------|
|pviews           |e: pviews [view]  可以打印指定 view 的所有层级结构, 不指定则打印 window|Yes|Yes|
|pvc              |可以打印 window 的所有层级结构|Yes|No|
|visualize        |预览 UIImage, CGImageRef, UIView, CALayer, NSData(图片类型), UIColor,CIColor,CGColorRef|Yes|No|
|fv               |e: fv scrollView  找view.|Yes|No|
|fvc              |e: fvc ViewController 找ViewController|Yes|No|
|show/hide        |显示或隐藏一个 View 或一个 layer|Yes|Yes|
|mask/unmask      |在 view 或者 layer 上添加一个遮罩.|Yes|No|
|border/unborder  |在 view 或者 layer 上添加一个 border.|Yes|Yes|
|caflush          |重绘界面|Yes|Yes|
|bmessage         |给某个方法添加断点|Yes|Yes|
|wivar            |给某个对象增加 watchpoint 观察值变化|Yes|Yes|
|presponder       |打印某个UIControl 的响应链|Yes|Yes|

