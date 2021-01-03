---
layout: post
title: 图解 @weakify 与 @strongify 宏
date: 2019-09-18
tags: ["macro","日志","rac","ReactiveCocoa","strongify","weakify","前端知识","宏定义","客户端开发知识","计算机基础知识"]
categories:
- 计算机
---

![weakify](weakify_oc.png "weakify")

* * *

之前写过一篇 [C 语言宏定义的方方面面](https://www.xiaobotalk.com/archives/497)，最近研究了一下 ReactiveCocoa 提供的宏 weakify 与 strongify ，让我对宏的写法有了更多的认识，原来宏还有这么多好玩的地方。

## 一：weakify 的最终展开

* * *

我们可以先用 Xcode 的预处理功能，展开 `@weakify(self)` ，看看宏的最终展开是什么：

![weakify_strongify](weakify_expand.png "weakify_strongify")

最终展开形式中的 `__attribute__((objc_ownership(weak)))` ，是编译器把关键字 `__weak` 也展开了，替换回去，单纯的宏展开形式：

    // 最终展开 @weakify(self) 
    @autoreleasepool {} __weak __typeof__(self) self_weak_ = (self);

展开部分的 `@autoreleasepool {}` 帮助实现了语法糖 `@`。这个被定义在 `weakify` 宏的第一行宏定义`rac_keywordify`中：

    // weakify 宏定义
    #define weakify(...) \
        rac_keywordify \
        metamacro_foreach_cxt(rac_weakify_,, __weak, __VA_ARGS__)

    // rac_keywordify 宏定义
    #if DEBUG
    #define rac_keywordify autoreleasepool {}
    #else
    #define rac_keywordify try {} @catch (...) {}
    #endif

## 二：自己动手写个最简单的 weakify 宏

* * *

看到宏展开后，我立马觉得，这不就是我们手动解决循环引用时经常写的 `__weak __typeof__(self) self_weak_ = (self)` 的形式吗 ?  把这个用宏实现非常简单：

    // 语法糖 @ 帮助宏
    #if DEBUG
    #define xt_keywordify autoreleasepool {}
    #else
    #define xt_keywordify try {} @catch (...) {}
    #endif

    // 拼接变量名的工具宏
    #define xt_concat(A, B) xt_concat_(A, B)
    #define xt_concat_(A, B) A ## B

    // 最终展开的宏实现
    #define xt_weakify(__XT_OBJ__) \
            xt_keywordify \
            __weak __typeof__(__XT_OBJ__) xt_concat( __XT_OBJ__, _weak_) = (__XT_OBJ__);

有了上述宏以后，我们就可以马上在代码中写 `@xt_weakify(self)` 其效果和 `@weakify(self)` 的展开完全一样。那么为什么 RAC 的`weakify` 写的如此复杂呢 ? **答案是为了实现可变参数。** 比如我们可以这样写：

    @weakify(self, obj1, obj2, obj3)

而这样的写法，一共可以写 20 个参数，也就是说：RAC 的 `weakify` 宏一共支持最多 20 个参数的写法。

## 三：图解 RAC 的 weakfiy 宏展开

* * *

宏展开比较复杂，所以分为两部分来分析，`weakify` 同时相关的宏被定义在 `RACmetamacros.h` 和 `RACEXTScope.h`这两个头文件里，可以结合着看。

> 1: 宏观的看宏展开
>   2: 如何实现多参数计算

**1: 宏观的看宏展开**

![weakify](weakify_macro.png "weakify")

**2: 如何实现多参数计算**

宏的一个特点是，只做替换，不做任何表达式求解，所以也不能在宏中使用像 `for` 循环这样的结构，虽然我们可以方便的使用 `__VA_ARGS__` 来实现可变参数的宏，但是想知道具体有几个参数就得想其他办法。

上一步分析中，我们看到了 `metamacro_foreach_cxt` 宏及其兄弟宏 `metamacro_foreach_cxt0/1/2/3/..`，`metamacro_foreach_cxt` 宏的作用就是通过计算可变参数，把自己转换为对应的宏 `metamacro_foreach_cxt0/1/2/3/..` :

下图详细分析了 `metamacro_foreach_cxt` 到 `metamacro_foreach_cxt0/1/2/3/..` 的过程：
![weakify](weakify0_1.png "weakify")

## 四：个别宏定义的说明

* * *

### 1、metamacro_foreach_cxt 宏定义

    #define metamacro_foreach_cxt(MACRO, SEP, CONTEXT, ...) \
        metamacro_concat(metamacro_foreach_cxt, metamacro_argcount(__VA_ARGS__))(MACRO, SEP, CONTEXT, __VA_ARGS__)

上边宏定义中的 `metamacro_concat` 宏是个工具宏，用来连接两个字符，而`metamacro_argcount`则真正实现了参数个数的计算。

### 2、metamacro_concat 宏定义

    #define metamacro_concat(A, B) \
        metamacro_concat_(A, B)

    #define metamacro_concat_(A, B) A ## B

这个宏里用到了 `##` ，就是用来拼接字符的，这里定义了两遍，即先定义

    #define metamacro_concat_(A, B) A ## B

然后定义

    #define metamacro_concat(A, B) \
          metamacro_concat_(A, B)

目的是为了，宏嵌套的时候也能正常展开，详细的可以看我之前的文章: [C 语言宏定义的方方面面](https://www.xiaobotalk.com/archives/497)。

### 3、metamacro_argcount 宏定义

而真实实现参数计算的宏是 `metamacro_argcount`，这个宏最终展开结果是 `0,1,2,3...`，即参数个数。

    #define metamacro_argcount(...) \
        metamacro_at(20, __VA_ARGS__, 20, 19, 18, 17, 16, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1)

    #define metamacro_at(N, ...) \
        metamacro_concat(metamacro_at, N)(__VA_ARGS__)

### 4、metamacro_argcount0/1/2... 宏定义

为了用宏计算参数，RAC 巧妙的定义了 `metamacro_argcount0...20` 来进行消参:

    #define metamacro_at0(...) metamacro_head(__VA_ARGS__)
    #define metamacro_at1(_0, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at2(_0, _1, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at3(_0, _1, _2, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at4(_0, _1, _2, _3, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at5(_0, _1, _2, _3, _4, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at6(_0, _1, _2, _3, _4, _5, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at7(_0, _1, _2, _3, _4, _5, _6, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at8(_0, _1, _2, _3, _4, _5, _6, _7, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at9(_0, _1, _2, _3, _4, _5, _6, _7, _8, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at10(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at11(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at12(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at13(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at14(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at15(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at16(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at17(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at18(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at19(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18, ...) metamacro_head(__VA_ARGS__)
    #define metamacro_at20(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18, _19, ...) metamacro_head(__VA_ARGS__)

    #define metamacro_head(...) \
        metamacro_head_(__VA_ARGS__, 0)

    #define metamacro_head_(FIRST, ...) FIRST

`metamacro_at0...20` 宏通过 `__VA_ARGS__` 和手动数字 1、2、3...作为站位参数配合实现了计算参数个数的功能。最后通过 `#define metamacro_head_(FIRST, ...) FIRST` 得到具体的数字，非常巧妙。

### 5、metamacro_foreach_cxt0/1/2... 宏定义

而 `metamacro_foreach_cxt0/1/2...20` 则用了一种像递归的写法，实现了多参数展开：

    #define metamacro_foreach_cxt0(MACRO, SEP, CONTEXT)
    #define metamacro_foreach_cxt1(MACRO, SEP, CONTEXT, _0) MACRO(0, CONTEXT, _0)

    #define metamacro_foreach_cxt2(MACRO, SEP, CONTEXT, _0, _1) \
        metamacro_foreach_cxt1(MACRO, SEP, CONTEXT, _0) \
        SEP \
        MACRO(1, CONTEXT, _1)

    #define metamacro_foreach_cxt3(MACRO, SEP, CONTEXT, _0, _1, _2) \
        metamacro_foreach_cxt2(MACRO, SEP, CONTEXT, _0, _1) \
        SEP \
        MACRO(2, CONTEXT, _2)

    // metamacro_foreach_cxt4 宏定义...
    // metamacro_foreach_cxt5 宏定义...
    // metamacro_foreach_cxt6 宏定义...
    // ......
    // metamacro_foreach_cxt20

(全文完) 转载请注明出处!