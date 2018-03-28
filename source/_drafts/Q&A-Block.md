---
title: Q&A Block
tags:
  - Blog
date: 2018-02-26 16:13:34
permalink: ios-QA-block
---

1. Q: Block 有多少种，分别是什么?
A: 三种，分别是 _NSConcreteStackBlock, _NSConcreteGlobalBlock, _NSConcreteMallocBlock

2. Q: 解释各种 Block 的特点和区别
A: Stack Block 是处于栈上的，Global Block 是处于数据区域（全局区域）上的，Malloc Block 是处于堆上的（调用 copy 产生）。在 ARC 下只有 Global Block 和 Malloc Block，Stack Block 被 Malloc Block 代替，因为 ARC 已经很好的处理对象生命周期的管理，把所有对象在堆上处理对编译器来说会更加方便

3. Q: 简述一下 Block 的结构
A: Block 在编译的时候会生成如下 struct

```
struct block_descriptor {
    size_t reserved;
    size_t size;
    void (*copy)(void *dst, void *src);
    void (*dispose)(void *);
};
struct block_impl {
    void *isa;
    int flags;
    int reserved;
    void *invoke;
};
struct block_layout {
    struct block_impl impl;
    struct block_descriptor *des;
    /* variables */
};
```

4. Q: Block 是怎么捕获外部变量的
A: 在 Block 中使用外部变量，编译时会在 block_layout 中 capture 对应变量（或者变量地址）。例如:

```
int a = 0;
^{ NSLog(@"%d", a); }();
-------------------
struct block_layout {
    struct block_impl impl;
    struct block_descriptor *des;
    int a; // capture 变量
};
```

5. Q: 在 block 中怎么修改外部变量，能具体说明为什么吗
A: 想要在 Block 中能够修改外部变量，需要在声明的时候增加 __block。原因是使用 __block 声明的变量，在编译的时候会生成一个储存当前变量地址的 struct，而 block 不再捕获变量，改为捕获该 struct。

```
__block int i = 0;
^{ i = 1; }();
----------------
struct block_variable_ref_i {
    void *isa;
    void block_variable_ref_i *forward;
    int flags;
    int size;
    int i;
};
struct block_layout {
    struct block_impl impl;
    struct block_descriptor *desc;
    struct block_variable_ref_i *i;
};
```


