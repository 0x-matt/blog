---
layout: post
title: Dart 中的泛型 Generics
date: 2019-12-28
tags: ["dart","flutter","generics","parameterized","日志","前端知识","参数化类型","客户端开发知识","泛型","编程","计算机基础知识","跨平台"]
categories:
- 计算机
---

作者: xiaobo

* * *

![horse](photo-1460999158988-6f0380f81f4d?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80 "horse")

Dart 是门类型安全的语言，由于 Dart 同时支持 JIT 编译和 AOT 编译，所以 Dart 类型系统同时工作在编译期和运行时。类型安全的语言，一般都是强类型语言，也就是定义一个变量的时候，该变量需要有明确的类型：

    Int a = 10;

但是，由于 Dart 具有**类型推断**机制，所以类型声明是可选的：

    var arguments = {'argA': 'hello', 'argB': 42}; // Map<String, Object>

## 泛型

* * *

泛型也叫参数化类型(parameterized)，类型安全的语言，通常是需要泛型的；Dart 中的 array 类型 List，就用了泛型：

    class List<E>

`<...>` 就表示了 List 是一个泛型类。类型变量一般都用单一的大写字母表示，常见的有 E、T、S、K、V。

**泛型主要有以下 2 个作用：**

1: 泛型可以让代码的类型更加安全:

    var names = List<String>();
    names.addAll(['Seth', 'Kathy', 'Lars']);
    names.add(42); // Error

上边代码，通过 `List<String>` 的方式指定了，List 内部的元素是 String，如果添加其他类型的值就会编译报错。

除了 List，其他的定义了泛型的类都可以在初始化的时候指定类型：

    var views = Map<int, View>();

2: 使用泛型还能减少代码的重复率，假设，我们设计一个缓存存取的接口，通常会用抽象类来设计：

    abstract class StringCache {
      String getByKey(String key);
      void setByKey(String key, String value);
    }

上边的抽象类，只能解决 String 类型的对象实现 Cache，如果 Number 类型，则需要额外定义一个接口：

    abstract class IntCache {
      IntCache getByKey(IntCache key);
      void setByKey(IntCache key, IntCache value);
    }

使用泛型，就可以解决上述代码重复的问题：

    abstract class Cache<T> {
      T getByKey(String key);
      void setByKey(String key, T value);
    }

上边的代码中，T 表示一种内置类型，是一个类型占位，具体是什么类型，需要在后续具体实现的时候再指定。

## 限定泛型的类型

* * *

当自己实现一个泛型类的时候，我们可以通过 `extends` 限定泛型的类型范围:

    class Foo<T extends SomeBaseClass> {
      // Implementation goes here...
      String toString() => "Instance of 'Foo<$T>'";
    }

    class Extender extends SomeBaseClass {...}

上边的类 Foo 在定义泛型 T 时候，限定了，T 必须是 SomeBaseClass 或者它的子类。所以，初始化 Foo 的时候，下列两种方式都可以：

    var someBaseClassFoo = Foo<SomeBaseClass>();
    var extenderFoo = Foo<Extender>();

甚至可以不指定泛型类型，不指定的情况，类型系统默认是 SomeBaseClass:

    var foo = Foo();
    print(foo); // Instance of 'Foo<SomeBaseClass>'

但是，不可以指定泛型限定范围外的类型：

    var foo = Foo<Object>(); // error

## 泛型方法

* * *

在新版本的 Dart 语法中，泛型被运行定义在方法上：

    T first<T>(List<T> ts) {
      // Do some initial work or error checking, then...
      T tmp = ts[0];
      // Do some additional checking or processing...
      return tmp;
    }

Dart 中泛型类型通常可以以下三个地方：

> 1.  函数返回值 (<T>)
> 2.  参数类型 (List<T>)
> 3.  局部变量  (T tmp)

## Dart 运行时泛型检查

* * *

开头，提到 Dart 的类型系统同时工作在编译时和运行时，所以我们可以在运行时检查一个带泛型的类型：

    var names = List<String>();
    names.addAll(['Seth', 'Kathy', 'Lars']);
    print(names is List<String>); // true ，运行时检查

这一特性是比较少有的，例如在 Java 中，就不可以在运行时检查泛型类型，Java 会在运行时系统中移除泛型参数。所以 Java 中，运行时只能检查是否是 List，不可以检查 List<String>。

## 参考文章

1.  https://dart.dev/guides/language/language-tour#generics
2.  https://dart.dev/guides/language/sound-dart#type-inference

(全文完)