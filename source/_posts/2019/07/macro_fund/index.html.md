---
layout: post
title: 宏定义的方方面面
date: 2019-07-21
tags: ["#","##","#宏","C语言","C语言宏","macro","日志","__COUNTER__","__VA_ARGS__","前端知识","参数宏","字符串宏","客户端开发知识","对象宏","带参宏","计算机基础知识"]
categories:
- 计算机
---

宏作为 C 语言强大的预处理特性，在 C 系列开发中有着举足轻重的作用，一个显而易见的作用是底层 API 的跨平台能力。例如  iOS 开发平台的 CGFloat 定义：

    #if defined(__LP64__) && __LP64__
    #define CGFLOAT_TYPE double
    #else
    #define CGFLOAT_TYPE float
    #endif

    typedef CGFLOAT_TYPE CGFloat;

上述宏定义，让 CGFloat 在 64 位操作系统中变为 double 类型，在 32 位操作系统中变为 float 类型。

一直以来我对 C 的宏定义理解，都停留在表面；最近花了点时间，搞清楚了宏的大部分写法和使用场景，我主要参考的的是 C 语言进阶这本书

![c 语言进阶](c_advanced.png)

和 Xcode 以及一些开源代码([WebViewJavascriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge))，下面是我的笔记:

正确的使用宏的关键其实只有一点：**宏定义只做替换，不做任何运算和表达式求解，带参宏的参数也遵循这个规则**。

## 利用 Xcode 预处理功能展开宏

* * *

有时候，我们写完一个宏，想知道展开后到底是什么样的，可以利用 Xcode 的 Preprocess (预处理)功能，下图是展开系统自带宏 MIN(A, B) 示例：

![preprogress](macro.png)

## 一、简单宏替换

* * *

简单宏替换的定义如下：

    #define AINT 10

上边定义的宏 AINT ，就是一个简单的宏定义，有的教材也称简单宏为对象宏。简单宏可以用来提升代码可读性，例如：

    #define PI 3.1415

也能增强代码的可移植性：

    #define INT_SIZE sizeof(int)

不同的编译环境，sizeof(int) 结构可能不相同，用宏来表示，能提高代码的可移植性。

## 二、带参数宏

* * *

带参数宏，因其使用起来像函数调用，有时也被称为函数宏，但和函数调用完全不一样，宏只做替换。带参数定义格式如下：

    #define 宏名(参数表)

定义时需要注意以下几点：

> 1: 宏名和参数表之间不能有括号
>   2: 宏只做替换，不做计算和表达式求解。

示例：

    #define ADD(_A, _B) (_A+_B)

    int add = ADD(10, 20); // 宏展开：int add = (10+20);

上边的宏用来计算两个数的和，定义时需要注意：

1.  宏名和参数名最好都大写，必要的情况下命名以 _ 开头。
2.  定义体 `(_A+_B)` 中的括号不能少。

**第一点**，关于命名，要格外注意，因为宏展开时会匹配相同字符，匹配到后进行替换，所以尽量大写字母配合下划线命名，来避免宏替换过程的误伤。这一点在 OC 语言中容易发生：

    #define ADDNUM(num1, num2) [self addNum1:num1 num2:num2]

    - (int)addNum1:(int)num1 num2:(int)num2 {
        return num1 + num2;
    }

    ADDNUM(1, 2); // 此时宏展开会变为：[self addNum1:1 2:2]

上边代码，编译报错，因为宏展开时，会找到所有的 num2 统一替换为 2。很显然 `[self addNum1:num1 num2:num2]` 中匹配到了两个 num2，展开后变为：`[self addNum1:1 2:2]`

**第二点**，宏定义中如果有表达式，一定要加括号，下边示例演示不加括号会发生什么：

    #define ADD(_A, _B) _A+_B

    int c = ADD(1, 2) * 3; // c 为 7

上边的代码，预期 c 是 `3*3 = 9`；但实际并不是，宏展开会变为：`int c = 1+2 * 3`; 结果会先做乘法运算 `2*3`，然后 `+ 1` 得到： 7。因为宏只做替换，不进行表达式求解。

## 三、宏定义特殊字符：'#'

* * *

'#' 在宏定义中，用来将其后出现的宏，变为字符串，举例来说：

    #define NAME(x) #x

    char *name = NAME(Bob); // 展开为：char *name = "Bob";

上边的宏中，'#' 将 Bob 转为为一个字符串 "Bob" 。'#' 宏用来在 C 代码中插入其他代码：

    #define _JSCODEINJECT_(x) #x

    NSString *jsCode = @_JSCODEINJECT_(
        ;(function() {
            if (window.InjectObject) {
                return;
            }
            var globalText = "call from c code";
            window.InjectObject = globalText;
        })(); 
    );

上边的代码，由于使用 '#' ，使代码的可读性和美观性大大增加。同时我们使用了 '@' 语法糖。

**`#` 定义宏的时候注意点:**

一般定义一个 `#` 宏，都会用如下格式：

    #define _STRINGYFY(x) #x
    #define STRINGYFY(x) _STRINGYFY(x)

使用的时候，使用不带下划线的 `STRINGYFY(x)`，这样做，并非多此一举。而是为了产生宏嵌套的时候，也能按照预期展开宏。

我用下边的例子，来说明问题：

    #define NAME Bob

    #define _STRINGYFY(x) #x
    #define STRINGYFY(x) _STRINGYFY(x)

    char *myName1 = _STRINGYFY(NAME);
    char *myName2 = STRINGYFY(NAME);

    // myName1: "NAME"
    // myName2: "Bob" ，为预期结果

上边的代码，先定义了对象宏 `NAME`，随后定义了 `_STRINGYFY` 和 `STRINGYFY`，用来转换字符串；
1: `myName1` 我们用 `_STRINGYFY` 宏来定义，结果为 "NAME"，原因是宏展开的时候，从左向右依次展开，所以先展开 `_STRINGYFY` 为 `#NAME` ，此时编译器遇到了 `#` ，所以直接把 `#` 后边的宏 NAME 替换为字符串 "NAME" 。
2: `myName2` 我们用 `STRINGYFY` 宏来定义，展开时，先展开`STRINGYFY`为：`_STRINGYFY(NAME)`，然后编译器会展开 `NAME` 为 `Bob`，最后再次展开`_STRINGYFY(Bob)`，得到 "Bob"。

## 四、宏定义特殊字符：'##'

* * *

'##' 在宏定义中用来连接前后两项参数，例如：

    #define CONCAT(A, B) A##B
    int main(int argc, const char * argv[]) {
        BOOL CONCAT(is, Correct) = NO; // 展开 BOOL isCorrect = NO;
        return 0;
    }

上边的例子，`CONCAT` 宏将 is Correct 合并为 isCrorrect，但是注意，`##` 不能直接用来连接字符串，想要连接字符串，需要配合前边说的 `#` 宏：

    #define _STRINGIFY(x) #x
    #define STRINGIFY(x) _STRINGIFY(x)

    #define _CONCAT(A, B) A##B
    #define CONCAT(A, B) _CONCAT(A, B)

    #define CONCATCOUNTER(A) CONCAT(A, __COUNTER__)

    int main(int argc, const char * argv[]) {

        char *param0 = STRINGIFY(CONCATCOUNTER(param));
        char *param1 = STRINGIFY(CONCATCOUNTER(param));
        char *param2 = STRINGIFY(CONCATCOUNTER(param));
        /** 上边宏展开为:
         char *param0 = "param0";
         char *param1 = "param1";
         char *param2 = "param2";
         */
        return 0;
    }

类似`#`的定义，为了在嵌套宏中也能让 `##` 正常工作，一般也需要先定义`_CONCAT(A, B)`，再次定义`CONCAT(A, B)`。上边的例子，很好理解，需要说明的是 `__COUNTER__`，这个宏是预编译宏，初始值为 0，预编译展开一次，值自动自增一次，所以上边的例子会出现 `param0` `param1` `param2`，0、1、2 就是`__COUNTER__`的功劳。

`##` 的另一个使用场景就是自定义 log 的时候：

    #define MYLog(format, ...) NSLog(format, ##__VA_ARGS__)

`__VA_ARGS__` 宏表示的是宏定义中的`...`中的所有剩余参数，这里用 `##` 来连接可变参数，当可变参数的个数为零时，`##` 会自动吃掉 `format` 后边的 `,` 号，保证编译正常。

## 五、换行宏符号 `\`

* * *

`\` 宏使用很简单，就是让宏内容能后换行：

    #define _STRINGYFY(x) #x
    #define STRINGYFY(x) \
        _STRINGYFY(x)

## 六、文章开头的 MIN(A, B) 宏

* * *

我们将系统的宏 MIN(a, b) 展开如下：

    ({
        __typeof__(a) __a0 = (a); 
        __typeof__(b) __b0 = (b); 
        (__a0 < __b0) ? __a0 : __b0; 
    });

这也正是 GNU C 中 MIN 的标准写法，MIN 宏先用一对 `({...})` 来限制一个作用域，同时这种语句会把最后一行的结果作为返回值，然后在作用域中定义零时变量 `__a0`, `__b0`，它们的值就是宏参数 a, b 的值。然后比较这两个零时变量的值，然后进行返回。这样的写法，基本可以兜底各种使用方式，比如：

    float a = 1.0f;
    float b = MIN(a++, 1.5f);

读者可以试着自己写写 MIN(A, B) 宏，然后去体会 GNU C 中 MIN 的标准写。

(全文完)