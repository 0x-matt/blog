---
layout: post
title: Alfred 软件介绍和使用简介
date: 2018-11-22
tags: ["Alfred","Clipboard","Mac 佳软","Mac 神软","Mac 高效使用指南","Packal","wordflow","上古神器","使用佳软","剪切板","工作流","黑帽子"]
categories:
- 实用软件
---

Alfred 是 Mac 上的最强的工作流软件，它几乎 Mac 一切程序和开始的入口，是 Mac 的大管家。有了它，你的 Mac 生涯才算完整。

> Alfred 官网地址：https://www.alfredapp.com/  
> 
>   本文截图示例中 Alfred 3 版本：v3.7 [938]

使用 `Ctrl + 空格`(快捷键自己可配置)，可以在任意界面呼出 Alfred 用户界面，它看上去就是一个小窗口，很低调，和系统的 Spotlight 有几分相似。  

![](alfred_02.png)

但是别被它瘦小简单的用户界面蒙蔽了双眼，正所谓 `大道至简` ，就是这个简单界面斩获了 Mac 电脑的第一神软的称号，从未被超越。

## Alfred 都能干些什么

* * *

1.  计算器:

    ![](alfred_01.png)</p>
2.  强大的文件查找功能

    `find : 查找文件所在目录`
`open : 查找文件并且打开目录`
`in : 文件内查找`
    find 示例图片:
    ![](alfred_06.png)

3.  启动软件

    ![](alfred_05.png)

以上功能你可能觉得系统自带的 Spotlight 也有，那么我们展示一些 Spotlight 没有的功能

1.  锁屏

    ![](alfred_12.png)

2.  关机

    ![](alfred_13.png)

3.  退出所有程序

    ![](alfred_14.png)

4.  清空废纸篓

    ![](alfred_15.png)

还有很多，就不一一列举了。

## Alfred 的工作流能干些什么

* * *

workflow 也叫工作流，即把一系列操作或者功能压缩成一种快捷的使用方式，以此来提高使用效率，比如下边的示例。

快速查单词  

![](alfred_16.png)

查询天气

![](alfred_11.png)

一名会计工作者使用 Alfred 快速转换大写的 RMB

![](alfred_10.png)

一名设计人员使用 Alfred 来取色

![](alfred_07.png)

另一名程序员要用 Alfred 快速生成 .gitignore

![](alfred_09.png)

一名 Xcode 程序员使用 Alfred 快速清理 DerivedData

![](alfred_04.png)

...这里只列举了几个我自己安装的 workflow。

## Alfred 配置和使用详解

* * *

首先打开 Alfred 软件配置界面，最新版的 Alfred 3 只需要在输入框输入 alfred 然后回车即可。

![](alfred_03.png)

老版本的先快捷键呼出 Alfred 输入框，点击输入框的右上角的小齿轮。

![](alfred_22.png)

Alfred 软件界面我们需要关心的主要有: 
* General : 启动快捷键等功能。
* Features : 各种系统级别功能设置，lock 锁屏，emptyTrash 清空废纸篓等都可以在这里找到并设置。
* workflows : 安装、配置、或自己编写工作流的地方。
* Appearance ：主题设置。
* Advanced : 高级设置。

Features 中大部分功能使用都简单易用，下图比如 System 选项里展示了系统的一些常用操作;

![](alfred_27.png)

Features 除了上边展示的这些易用的功能，我们还可以自定义其他简单的快捷搜索，比如快速用`京东`搜索某商品。

![](alfred_18.png)

上图所展示的行为是自动打开浏览器，展示京东的搜索结果；

![](alfred_19.png)

完成这个配置我们只需要做几个简单的配置:  

1. 在 Features 中有个 web search 选项；
2. 点击右下角的 Add Custom Search
3. 在弹出框里写入 url 来查询，定义使用的快捷键。
![](alfred_17.png)

如何获得 Search URL ？

> 我们需要先打开浏览器去京东官网，进行一次正常搜索查找，查找的内容可随意，搜索后，拷贝浏览器地址栏的地址，比如用京东搜索手机，浏览器地址栏的 URL 如下：  
> 
>   ![](alfred_21.png)  
> 
>   然后把刚才的搜索内容`手机`替换为 `{query}` ，如果还有其他多余的参数可以直接删除；
>   `https://search.jd.com/Search?keyword={query}&enc=utf-8&wq={query}`
>   然后填写到 Search URL，然后保存，Done。

**Alfred 强大的剪切板功能**

也许你曾经遇到过在 Mac 上，拷贝了一串内容，想要粘贴到某个地方，但是在你粘贴之前，却又有其他的拷贝需求，最新的拷贝覆盖了你之前的拷贝，导致你不得不重新去拷贝；而有了 Alfred 的剪切板功能，这个问题就变得愉快了许多。下图是我写这篇文章时 Alfred 的剪切板，它记录你最近 n 次 (具体展示几条可自己设置) 的拷贝记录，并且有预览窗口，选中后敲键盘上的 `Enter` 键就会把内容拷贝到你当前的光标位置，不得不说这个功能大大提高了我写作的效率。

![](alfred_28.png)

Alfred 剪切板的具体设置在 Features -> Clipboard 选项。

除了这些 Alfred 的基础功能还有很多，上述介绍的都是冰山一角，例如 Features -> Snippets 也可以帮你做好多事情，这里就不一一列举了，更多功能等待着你的探索。

**使用 workflow**

Alfred 的 workflow 是付费之后才可以使用的， workflow 的使用类似于插件的方式，需要我们自己下载使用。任何人都可以给 Alfred 写 workflow，并且上传共享出来。workflow 下载市场现在主要在 [Packal](http://www.packal.org) 和 [workflow list](http://alfredworkflow.com/) 网站，这里像一个 App store ，展示着来自世界各地 Alfred workflow 贡献者的作品。 下载下来的文件是 .workflow 后缀名文件，双击会自动安装到 workflow 的面板里，面板中会介绍如何使用，通常使用都非常简单，看一下使用的快捷键就可以了，下图是我的 workflow 列表。

![](alfred_20.png)

我自己也写了个简单的 workflow ，功能是为 Xcode 开发者清理 DerivedData 数据。
![](alfred_04.png)

欢迎下载使用：[Packal 下载地址](http://www.packal.org/workflow/xcdd) 、[Github 链接](https://github.com/ChopinChao/xcdd_workflow)。

如果你想通过 Alfred 制作自己的 workflow， Alfred 有官方的教程教你一步步制作一个 workflow ：[workflow tutorial](http://www.deanishe.net/alfred-workflow/tutorial.html)，你可以用 shell/python/ruby/php/perl/AppleScript 任何一种语言编写。

**编写一个自己的 workflow 主要步骤**

如果上边的英文教程看着吃力的话，我写了个简明的中文教程；

1.  打开 workflow 界面，点击左下角的 `+`， 然后选 Blank Workflow ；
![](alfred_23.png)

2.  在弹出框中填写基本信息，重点是 name 、Bundle ID 和 icon 三个选项，Bundle ID 是你的 workflow 的唯一标识，icon 可以从 Packal 官方给的 [icon 网站](http://www.iconarchive.com/show/simple-icons-by-danleech/pinboard-icon.html) 挑选；
![](alfred_24.png)

3.  添加 Filter Script

    ![](alfred_25.png)

4.  完成你的编码

    ![](alfred_26.png)

<p>写完后，save 即可使用，恭喜你，已经完成了自己的 workflow。同时你也可以导出你的 workflow 文件共享到 Packal 社区或者 Github-release。

**通过 workflow 定制 App 启动快捷键**

第一步，创建 hotkey 工作流，Templates -> Files and Apps -> Launch file group from hotkey

![](alfred_35.png)

第二步，填写 workflow 基本信息

![](alfred_34.png)

完成后生成 hotkey -> launch Apps/Files，接下来分别配置

![](alfred_33.png)

第三步，配置你的 hotkey (快捷启动热键)

![](alfred_30.png)

第四步，配置要启动的 App ，可以直接从应用程序拖入。

![](alfred_31.png)

Done.

Alfred 的功能还有很多，如果你已经读到这里了，就探索探索其他功能吧，本文只是做个指引和简易教程，更多好玩的还需自己体验和开发。

**本文同时发布与[我的 Gitbook 文集中](https://www.xiaobotalk.cn/)**

(完)