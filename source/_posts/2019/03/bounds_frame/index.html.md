---
layout: post
title: 再次学习 bounds 与 frame
date: 2019-03-30
tags: ["bounds","center","CGPoint","CGRect","CGSize","frame","iOS","日志","subview","superview","前端知识","坐标系","客户端开发知识"]
categories:
- 计算机
---

![bounds-frame](boundsAndFrame.png "bounds-frame")

本文主要记录 iOS UI 布局中的两个重要概念 bounds 和 frame。

* * *

iOS UI 搭建的过程中，苹果给我提供了非常原始和稳定的 UI 布局方式，Frame 布局，也可以称为绝对布局。其中有两个重要概念就是 bounds 和 frame，我们都知道 bounds 和 frame 都是 CGRect 的结构体：

    struct CGRect {
        CGPoint origin;
        CGSize size;
    };

    // origin 又是 CGPoint 结构体，size 是 CGSize 结构体：

    struct CGPoint {
        CGFloat x;
        CGFloat y;
    };

    struct CGSize {
        CGFloat width;
        CGFloat height;
    };

origin 指的是**坐标**，size 是 view 的大小。size 比较容易理解，frame 和 bounds 的大小是一致的，共享的，也就是说改变了 bounds 的 size，frame 的 size 会跟随变化；同样的 frame 的 size 改变了，bounds 的 size 也会跟随变化。

## 坐标系

* * *

重点是 origin，既然是坐标，就需要有坐标系，frame 的 origin 指的是 view 相对父控件坐标系中的坐标，具体值是需要开发人员添加子控件的时候进行赋值。而 bounds 的 origin 是相对自己的视图的坐标系的坐标值，默认值是 (CGPoint){0, 0}。

当一个 view 被添加到 superview 上以后，会自动生成一个从自己左上角延伸的坐标系（坐标系从视图左上角开始，x 轴向右延伸，y 轴向下延伸），并且默认把自己的 bounds 的 origin 设为 {0, 0}，也就是自己坐标系的坐标原点。

![bounds](bounds03.png "bounds")

bounds 是可以手动修改的，当把 superview 的 bounds 的 origin 改为 (25, 0) 的时候，实际是改变 superview 的坐标系位置，这会影响到添加到该视图上的子控件的位置。如下图：

![bounds](bounds02.png "bounds")

注意，上图 superview 的位置(相对其父控件)并没有改变，而是 subview 的 x 相对于 superview 的 左边缘，从 50 变成了 25。原因是 superview 的 bounds 变为 (25, 0, 400, 200)后，相当于把自己的坐标系左移了 25，而 subview 也跟随左移了 25。

结论：改变 superview 的 bounds 的 origin ，添加到其上 subview 会向反方向移动相同的距离。

## center 也是相对父控件坐标系的属性

* * *

一个 view 的 center 是根据 frame 计算得来的，也就是：

> center = (CGPoint) {.x = frame.origin.x + frame.size.width / 2, .y = frame.origin.y + frame.size.height / 2}

center 的值，只有当 frame 改变，才会发生改变，改变 bounds 并不会影响 center；所以改变 bounds.size 后，视图整体会以 center 为中心缩放，从而影响 frame.origin;

![center](center-bounds.png "center")

**clipsToBounds**

clipsToBounds 是根据当前视图的 bounds 进行裁剪视图的，因为 bounds 的改变会影响其子视图的位置，所以当由于 bounds 改变后子视图的显示范围超出了父视图，那么超出的部分就会被 clipsToBounds 属性裁剪。

(完)