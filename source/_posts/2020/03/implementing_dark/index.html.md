---
layout: post
title: iOS13 适配暗黑模式
date: 2020-03-26
tags: ["Assets Catalog","dark mode","iOS","iOS13","日志","semantic","systemColor","trait collection","UITraitCollection","UIUserInterfaceStyle","userInterfaceStyle","客户端开发知识"]
categories:
- 计算机
---

* * *

作者：xiaobo

![dark mode](iOS_Dark_Mode.png "dark mode")

* * *

暗黑模式是 iOS 13 带来的一种 UI 特性，对开发者而言，适配之前需要准备好以下开发环境：

1.  Xcode 11
2.  一部装有 iOS 13 的 iPhone 或 iPad 设备，方便真机调试。

如果你的 App 暂时还不想适配，或者不适合在暗黑模式的 UI 策略。可以暂时通过在工程的 info.plist 中添加`UIUserInterfaceStyle`配置来忽略暗黑模式：

![avoid_dark_mode](iOS_avoid_dark.png "avoid_dark_mode")

但即使这样，暗黑模式下，应用的状态栏依旧会发生一些变化，需要额外处理。再加上苹果最新的审核政策，强调了暗黑模式的重要性，还是趁早适配吧。

## 暗黑模式不是唯一的新 UI 模式

* * *

在 iOS12 更新的时候，苹果就为用户带来了`High Contrast`(高对比度模式) 的界面风格，在`设置 -> 辅助功能 -> 显示与文字大小 -> 增强对比度`里可以找到：

![high contrast](high-contrast.png "high contrast")

我查看了下自己手机上的 App，只有少部分 App 适配了高对比度显示；考虑到高对比度模式，iOS 一共有 4 种界面风格：

1.  默认
2.  默认高对比度
3.  暗黑
4.  暗黑高对比度

上述特性，如果开发者均使用 UIKit 的语义化颜色值，或者 SwiftUI，则适配不需要做任何事情，API 都支持好了。但实际情况恰恰相反，大部分企业 App 色彩管理都比较乱，一波设计人员，一套颜色方案，也是常见的事情。

## 如何适配

* * *

关于如何适配暗黑模式的技术点，并不多，也不复杂，官方的文档就一页，总结起来就 3 点：

1.  颜色适配
2.  图标资源适配
3.  H5

## 一、颜色适配

* * *

基本上完成颜色适配，对客户端来说，基本上完成了 90% 的工作量。

### 1、使用动态的系统颜色和语义化颜色

* * *

原来使用`UIColor.red / .yellow /..`的地方，改为使用`UIColor.systemGrayColor`。具体支持哪些颜色，可查看头文件`UIInterface.h`里的`UIColor (UIColorSystemColors)`，另外 iOS13 上新增了一些语义化颜色值，例如`labelColor、systemBackgroundColor`等，使用时注意添加版本判断：

    if (@available(iOS 13.0, *)) {
        color = UIColor.labelColor;
    } else {
        // other colors
    }

使用这些颜色设置方法，开发者什么都不需要做，系统会自动完成暗黑模式的适配。

### 2、使用`Assets Catalog`处理自定义颜色值

* * *

然而，大部分的 App，使用系统颜色和语义化颜色值的地方可能不到 10%；剩下的大部分自定义颜色值，都需要和设计讨论确认它们的暗黑色值；确定之后，可以使用`Assets Catalog`定义自己的语义化颜色值：

![Semantic_Colors](assets-dark-mode.png "Assets Catalog Colors")

相应的，代码中设置颜色的地方，只需要改为上述`Assets Catalog`中定义好的色值即可：

    [UIColor colorNamed:@"Magenta"];

使用`Assets Catalog`，还有个好处，是方便扩展高对比度颜色值：

![Semantic_Colors](assets-high-contrast.png "Assets Catalog Colors")

需要注意的是，`Assets Catalog`这种方式，只能支持 iOS10 及以上，如果你的 App 还需要适配 iOS9 或者更低版本，那么就无法使用`Assets Catalog`，只能通过代码去适配。

### 3、代码处理自定义颜色值的暗黑色值

* * *

代码处理，需要使用`UIColor`的`colorWithDynamicProvider`初始化方式，进行适配。依然注意该方法只在 iOS13 以上使用。

    if (@available(iOS 13.0, *)) {
        returnColor = [UIColor colorWithDynamicProvider:^UIColor * _Nonnull(UITraitCollection * _Nonnull traitCollection) {
            UIColor *color = nil;
            if (UIUserInterfaceStyleDark == traitCollection.userInterfaceStyle) {
                color = // dark color
            } else {
                color = // normal color
            }
            return color;
        }];
    } else {
        returnColor =// other color
    }

开始之前，如果需要知道 App 中都有哪些颜色值，可以通过在 DEBUG 模式下，swizzling 所有的 UIColor 方法，进行统计收集。

## 图标适配

* * *

图标适配和颜色适配是一样的方式；当然，如果你使用了苹果给开发者提供的 [ SF Symbols](https://developer.apple.com/design/human-interface-guidelines/sf-symbols/overview/) 图标符号库，那么什么都不用做。该方法也是 iOS13 新增的方法：

    [UIImage systemImageNamed:@"star.fill"]

## 如何调试暗黑模式

* * *

Xcode 11 中，新增了一个专门用于调试暗黑模式的按钮：

![Semantic_Colors](environment-overrides.png "Assets Catalog Colors")

在模拟器或者手机开启了暗黑模式设置后，可以在 App 运行的时候，快速的切换 Litght/Dark Mode 进行查看。

## Trait Collections

* * *

在苹果的暗黑模式适配的底层，主要功臣是`Trait Collections`，具体的实现是 UITraitCollection，UITraitCollection 中的属性`userInterfaceStyle`

    @property (nonatomic, readonly) UIUserInterfaceStyle userInterfaceStyle

表明了当前系统的界面模式：

    typedef NS_ENUM(NSInteger, UIUserInterfaceStyle) {
        UIUserInterfaceStyleUnspecified,
        UIUserInterfaceStyleLight,
        UIUserInterfaceStyleDark,
    } API_AVAILABLE(tvos(10.0)) API_AVAILABLE(ios(12.0)) API_UNAVAILABLE(watchos);

当设置里改为暗黑模式的时候，系统会从 UIScreen 开始逐层传递`Trait Collections`，直到最前面的 UIView。

![Trait Collections](trait_transport.png "Trait Collections")

并且，`Trait Collections`的改变总是发生在渲染 UI 之前，所以我们可以提前拿到`Trait Collections`进行颜色设置；并且可以针对某些视图单独设置`Trait Collections`，让这些视图不遵循当前的系统模式，可以直接通过属性`traitCollection`属性获得；另外在任何地方，我们都可以通过`UITraitCollection.currentTraitCollection`获取到当前的 TraitCollection 对象。

    view.traitCollection;
    viewController.traitCollection;
    UITraitCollection.currentTraitCollection;

WWDC 关于适配暗黑模式的视频中，推荐最好再下列方法中获取`Trait Collections`并进行适配：

![Trait Collections](method_trait_collection.png "Trait Collections")

## 最后别忘了调整 CGColor

* * *

`Trait Collections`只会在 UIKit 体系中有效，所有与 CGColor 有关的设置，需要额外处理，业务中与之相关的主要有下列场景：

*   阴影色值
*   富文本颜色
*   CALayer 颜色

处理这些的时候，还是需要通过`view.traitCollection`先获得当前的用户界面模式，然后做出相应的调整。

## 参考文档

* * *

1.  https://developer.apple.com/videos/play/wwdc2019/214/
2.  https://www.fivestars.blog/code/ios-dark-mode-how-to.html

(全文完)