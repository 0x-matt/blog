---
layout: post
title: 适配 iPad OS 吧，iPad 将会是最好的个人助理
date: 2020-04-18
tags: ["Application Scene Manifest","iPad","iPad OS","Multiple Windows","日志","SceneDelegate","分屏","客户端开发知识","感想与观点","无纸化学习"]
categories:
- 计算机
---

* * *

作者：xiaobo

![iPad Pro](blog_ipad_pro.jpg "iPad Pro")

对于 iPad，广为流传的一句广告词恐怕是 "买前生产力，买后爱奇艺"。我自己也使用了一段时间的 iPad，在我手里它并不是"爱奇艺"，而是变身为一台"优雅的学习利器"。由于它还不能用来写代码，所以对我来说还不具备多少生产力。

近期 iPad OS 的更新，让科技圈一阵 "骚乱"，纷纷表示，iPad 将取代部分轻量级 Windows 本，成为生产力；我决定购买 iPad ，一多半原因也是因为 iPad OS。iPad OS 有几大强势的新功能：

> 1、支持了鼠标和键盘操作
>   2、支持了多任务窗口操作
>   3、文件系统具备了一定的桌面属性
>   4、浏览器的使用体验几乎和桌面浏览器一致。
>   5、另外，相比桌面电脑多了根手写笔。触屏操作自然也不用多说了。

以上种种，我相信，假以时日，iPad 必将成为星巴克咖啡馆里的新宠。

## iPad 变身为"爱奇艺"的可能的原因之一

* * *

在使用了 3 个月后，我开始思考，之所以在大多数人手里，iPad 依旧是"爱奇艺"，可能不是用户自己的使用习惯问题，而是各互联网科技公司决策的结果。原因是这样的，当开发者开发了一款 iPhone 应用后，他几乎无需额外的开发成本，就能把这款 App 迁移到 iPad 上，让 iPad 用户也能在 App Store 下载到；但不做额外的适配就投放到 iPad 上，其的结果就是，App 在 iPad 上呈现的效果很差，基本就是个大屏的 iPhone，体验自然也很差，甚至大部分国民级应用连基本的横屏都不支持，更别提其他 App 了。

所以，你应该明白了我的意思，目前 iPad 上适配的最好的三方 App ，大都是娱乐软件：爱奇艺、腾讯视频、优酷、Bilibili、QQ 音乐，这些 App 也适配的最积极。用户自然的在 iPad 上最喜欢打开的就是这些软件，再加上自己本来也有一定的倾向。

## iPad 与学习教育软件

* * *

这次新冠疫情，极大的促进了 iPad 销量，原因是很多老师和学生需要在家里上网课，用一台屏幕更大的轻量设备，体验会更好，效率和效果也更佳，显然这些新增的用户是想用 iPad 来高效的工作或学习；对学习者来讲，配合上手写笔，iPad 完全能胜任无纸化学习了:

![iPad study](ipad_to_learn.png)

不过可惜的是，很多互联网教育公司并没有打算适配自家在 iPad 上的应用，依然热衷于天天通过各种运营手段来激励学生付费。但我认为花点时间适配好 iPad，应该能吸引不少用户留存下来；如果适配的比较好，可能还会带来新用户的增加。

## 适配 iPad OS 的几个核心功能

* * *

对开发者来说，额外的适配 iPad OS，工作量其实并不是很大。适配后主要达到的目标是以下几点：

**1、多任务分屏**

![iPad OS](ipad_adapt02.png "iPad OS")

**2、窗口悬浮**

![iPad OS](ipad_adapt03.png "iPad OS")

**3、横屏的布局适配**

![iPad OS](ipad_adapt05.png "iPad OS")

## 适配的准备工作

* * *

**1、在 Xcode 工程中启用全部横竖屏的支持**

建议4个方向全部启用，如果只支持竖屏，或者没有支持全部方向(4个)，则无法和其他支持了 4个屏幕方向的 iPad 应用一起分屏：

![iPad OS](iPad_ori.png "iPad OS")

**2、取消 Requires Full Screen 勾选，勾选支持多窗口**

如果勾选了 Requires Full Screen，表示 App 要求全屏，即无法和其他应用一起分屏：

![iPad OS](ipad_adapt11.png "iPad OS")

**3、勾选后，需要在 info.plist 中填写相关配置**

![iPad OS](ipad_scene.png "iPad OS")

**4、从 AppDelegate 迁移相关代码到 SceneDelegate**

从 iOS13 开始，iOS API 中新增了 UIWindowScene 来辅助实现 App 多窗口的功能：

![iPad OS](ipad_adapt01.png "iPad OS")

相应的，如果用 Xcode 11 来构建应用，你会发现多了一个叫 SceneDelegate 的类，我们要做的就是从 AppDelegate 中迁移部分实现到 SceneDelegate 中，方法对应的关系如下：

![iPad OS](ipad_adapt04.png "iPad OS")

需要注意，新的方法是 iOS13 才支持的，如果 App 需要继续兼容老版本，就需要用一些宏来判断系统版本，实现兼容。做完了上述步骤，iPad OS 的所有多窗口功能都支持了，适配的开始阶段完成；剩下的就是单个页面去调整横竖屏的布局。相关适配细节，可参考 WWDC 的视频教程：[Introducing Multiple Windows on iPad](https://developer.apple.com/videos/play/wwdc2019/212/)

(全文完)