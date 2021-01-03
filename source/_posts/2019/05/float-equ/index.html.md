---
layout: post
title: float 浮点数判等问题
date: 2019-05-13
tags: ["32","64","double","epsilon","float","日志","尾数","指数","无限循环小数","浮点数判等","精度","计算机基础知识"]
categories:
- 计算机
---

之前在 [二进制趣事](https://www.xiaobotalk.com/2019/02/binary_computer/) 一文中提到过计算机如何存储浮点数，最近对浮点数判断问题有了些新的理解，抛开细节不谈，其实很容易说清楚浮点数判等出错的原因：

> 计算机存储总是有限的，所以在表示无限循环小数时候只能取不同程度的近似值(位数)，以及不同的尾数舍弃标准(ceil, floor, round)，从而导致运算上出现不同的结果。

**具体实现上，可以总结为两个原因：**

1、编译器在 float 和 double 进行隐式转换时候的精度丢失
2、不同硬件平台的[FPU 浮点运算结构](https://zh.wikipedia.org/wiki/%E6%B5%AE%E7%82%B9%E8%BF%90%E7%AE%97%E5%99%A8)在处理浮点运算时的位数不同，例如中 Intel 8087 浮点处理单元中用了 80 位的寄存器做相关浮点运算，而之后的 SSE 则用了 128 位的寄存器做相关浮点运算。
![float](floatRegister.jpg "float")

## 代码示例分析

* * *

来看下列代码的输出：

    int main () {
        float x = 0.1;
        if (x == 0.1) {
            printf("if \n");
        } else if (x == 0.1f) {
            printf("else if \n");
        } else {
            printf("else \n");
        }
        return 0;
    }

    // 代码输出 else if

上边代码输出 `else if`。

**分析：**

1、除非我们写成 0.1f (float 类型) ，否则字面量 0.1 编译器认为是 double 类型，而等号左边是一个 float 变量，当把 0.1 的 double 类型赋值给 float 时，就会产生精度丢失；

2、参照 IEEE 浮点运算标准，float 用 32 位二进制表示，其中 1 位符号位，8 位指数位，23 位尾数位；double 则用 64 位二进制表示，其中 1 位符号位，11 位指数位，52 位尾数位。具体看我之前的文章：[计算机二进制趣事](https://www.xiaobotalk.com/archives/431)

3、float 和 double 比较时，会把 float 位数补齐到和 double 位数一样，具体是 float 23 位以后用 0 补齐，上边代码的 0.1 会有如下补全：

> float 类型 (左边 10 进制，右边 2 进制)
>   => 0.1 = 0.00011001100110011001100
>   float 隐式提升为 double 后，23 位之后用 0 补齐
>   => 0.1 = 0.00011001100110011001100 <span style="color:purple">000000000000000<span/>
> 
>   double 类型无隐式提升，0.1 用二进制表示的时候是无限循环小数
>   => 0.1 = 0.0001100110011001100110011001100110011001100110011001...

实际进行判等比较的时候，cpu 中浮点寄存器会读入浮点数，按照 sse 浮点运算标准，会读入 128 的浮点数，而上述代码的实际情况是从 23 位以后就发生了不同，所以比较结果是不相等。

**接下来，我们用相同的代码来比较 0.5 ：**

    int main () {
        float x = 0.5;
        if (x == 0.5) {
            printf("if \n");
        } else if (x == 0.5f) {
            printf("else if \n");
        } else {
            printf("else \n");
        }
        return 0;
    }

    // 代码输出 if

原因很简单，0.5 用二进制表示的时候，并不会有无数循环小数，1/2 就可以表示 0.5，所以 0.5 的二进制就是 `0.10000000...`，除了以第一个小数位是 1，后边全是 0，所以 float 和 double 表示试并不会因为位数不同而产生差异。

## 注意 iOS 中的 CGFloat

* * *

iOS 开发中，CGFloat 在不同的平台下，精度不同，具体是 32 位机器上，CGfloat 是 float 类型，64 位机器上是 double 类型：

![CGFloat](cgfloat_type.jpg)

iphone 从 5s 起，处理器都是 64 位系统，所以基本上现在的运行环境里，CGFloat 都是 double 类型，所以在代码中，基本上直接赋值一个小数，不需要加 f 尾缀。

## 如何进行浮点判等

* * *

所以浮点数直接用 `==` 判等，存在很大的风险，通常的做法是通过取两个浮点数差值的绝对值，和一个 epsilon(较小的值):

> if( fabs(a-b) < EPSILON )

epsilon 的取值可以根据具体业务进行设置，一般用指数形式表示一个 epsilon，例如 `1e-9`。

> 1e-9 表示: $10^{-9}$

但使用 `if( Math.abs(a-b) < EPSILON )` 方式依然不能绝对的安全，问题就在于 EPSILON 是否足够小，例如把 $10^{-6}$ 作为 epsilon ，也就是 0.000001。但这个数并不是足够小，例如用 (double)0.1 - (float)0.1 得到值是 0.0000000014901161...，远比 $10^{-6}$ 还小，所以为了让 epsilon 足够小，一般会对 epsilon * DBL_MIN ，让 epsilon 更加小:

> if( fabs(a-b) < EPSILON * DBL_MIN)

更加严格判等，可以参考文章：[https://floating-point-gui.de/errors/comparison/](https://floating-point-gui.de/errors/comparison/)。

## 参考文章

* * *

1、[https://www.geeksforgeeks.org/comparison-float-value-c/](https://www.geeksforgeeks.org/comparison-float-value-c/)
2、[https://floating-point-gui.de/errors/comparison/](https://floating-point-gui.de/errors/comparison/)
3、[https://stackoverflow.com/questions/801117/whats-the-difference-between-a-single-precision-and-double-precision-floating-p](https://stackoverflow.com/questions/801117/whats-the-difference-between-a-single-precision-and-double-precision-floating-p)
4、[https://www.cs.uaf.edu/2012/fall/cs301/lecture/11_02_other_float.html](https://www.cs.uaf.edu/2012/fall/cs301/lecture/11_02_other_float.html)
5、[https://stackoverflow.com/questions/10334688/how-dangerous-is-it-to-compare-floating-point-values](https://stackoverflow.com/questions/10334688/how-dangerous-is-it-to-compare-floating-point-values)

如果有时间和精力，可以读这篇文章，关于浮点数的即权威又全面的文章，不过篇幅很长: 
https://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html

(完)