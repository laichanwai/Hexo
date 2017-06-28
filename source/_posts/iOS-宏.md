---
title: iOS 宏
tags:
  - iOS
  - DEBUG
permalink: ios-macro
id: 24
updated: '2016-08-26 12:15:32'
date: 2016-08-26 11:01:57
---

## 文件包含

1. `#include`
2. `#import`
3. `#include_next`

## 宏定义

1. `#define`
2. `#undef`

## 条件编译

1. `#if #else #endif`
2. `#if define #ifdef #ifndef #elif`

## 错误、警告处理

1. `#error`
2. `#warning`

## 编辑器分段

1. `#pragma`
2. `#pragma mark xxx`
3. `#pragma mark - xxx`

## 其他

1. `#line number`
2. `#line number "filename"`

## Xcode 中预定义的宏

1. `__FUNCTION__`
2. `__func__`
3. `__PRETTY_FUNCTION__`
4. `__LINE__`
5. `__FILE__`
6. `__DATE__`
7. `__TIME__`
8. `__TIMESTAMP__`

Objective-C `- (void)runTestMacro:(id)arg1 arg2:(NSInteger)arg2`

```shell
__FUNCTION__        -[TestObj runTestMacro:arg2:]
__func__            -[TestObj runTestMacro:arg2:]
__PRETTY_FUNCTION__ -[TestObj runTestMacro:arg2:]
__LINE__            17
__FILE__            /Users/laizw/Documents/Demo/MacroDemo1/MacroDemo1/TestObj.m
__DATE__            Aug  3 2016
__TIME__            15:58:13
__TIMESTAMP__       Wed Aug  3 15:57:42 2016

```

C/C++ `void TestFunction(int arg1, int arg2)`

```shell
__FUNCTION__        TestFunction
__func__            TestFunction
__PRETTY_FUNCTION__ void TestFunction(int, int)
__LINE__            16
__FILE__            /Users/laizw/Documents/Demo/MacroDemo1/MacroDemo1/main.m
__DATE__            Aug  3 2016
__TIME__            15:59:41
__TIMESTAMP__       Wed Aug  3 15:59:41 2016
```