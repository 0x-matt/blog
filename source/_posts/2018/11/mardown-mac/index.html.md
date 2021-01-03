---
layout: post
title: Mardown 技巧与 Mac 上的写作软件
date: 2018-11-22
tags: ["css","html","img","Mac 高效使用指南","markdown","md","写作","图片大小","图片居中","技巧","排版","马克飞象"]
categories:
- 实用软件
---

<style>
h2{
    text-align:center;
}
</style>

### 一：内嵌简单的 html/css 丰富 Markdown 表现力

* * *

Markdown (简称 md 为了环保，后文中都用 md 代替)已经逐渐成为文档编写的主要手段，md 用及其简单的标记符号就能写出排版优秀的富文本文档，学习成本也非常低，不夸张的说，10 分钟就可以熟练掌握。如果你还没用 Markdown 写过文章，建议去 [maxiang.info](https://maxiang.io/) (马克飞象，一个在线的 md 编辑网站)，练个 10 分钟，然后再来看这篇教程，这篇教程假设你已经熟练掌握了 md 的常用写法；因为 md 对 html 的整合支持，所以可以通过内嵌简单 html 来丰富 md 文档。最后做个声明，我对 html/css 只是知道点皮毛，但是不妨碍我写丰富的 md；

#### 标题居中/右显示

你一定已经注意到，使用 `##..` 标记出来的标题，默认都是居左的，无论你在前面输入多少空格，它已久岿然不动的居左，因为 md 就是对 html 的整合，html 会忽略输入的所有空格(全角空格除外)和换行；更确切的说，`#` 其实表示的是 html 中的 `<h1>` 标签。

    '#' 表示 <h1> 标签
    '##' 表示 <h2> 标签
    '###' 表示 <h3> 标签
    ...
    '######' 表示 <h6> 标签

所以如果你想让某个级别的标题居中，可以在 md 文档开头切入一点 css 样式:

      <style>
        h2{
          text-align:center;
        }
      </style>

      <!-- 接下来，你所有用 '##' 标记出来的标题都居中了，center 改为 right，就居右了；-->

被 style 标签包含的内容就是样式了，在 md 中内嵌样式，不会显示出来，也不会占任何空行。

如果你只是想让某个 `##` 标题居中，而不是全部，那么只能通过 h2 标签来代替 ## 的写法了，如下：

    <h2 style="text-align:center">我是二级居中的标题<h2>

    <!-- 其他级别 h3/4/5/6 标题通用 -->

#### 给文字上色/修改字号...

md 可以用 `*斜体*` 和 `**粗体**` 语法快速实现文字的斜体和粗体，但是没有现成的标记语法来改变文字的字体和颜色，这时候可以使用 `span` 标签(span标签自身是没有任何特殊效果的标签，目的就是用来标记一段内容)来简单的实现，比如下边这段富文本：
<span style="color:purple">我是一段</span>有<span style="color:#a33">颜色</span>和<span style="font-size:30px">胖</span><span style="font-size:12px">瘦</span>的富文本示例。
md 源文件中是这样写的:

    <span style="color:purple">我是一段</span>
    有
    <span style="color:#a33">颜色</span>
    和
    <span style="font-size:30px">胖</span>
    <span style="font-size:12px">瘦</span>
    的富文本示例。

    <!--ps: 当然，如果你学过一些 html/css 知识，span 的样式也可以用 id 或者 类选择器实现；
    为了便于阅读我把每个标签对做了换行，实际写的时候写成一行。-->

#### 调整分割线粗细

`---`可以在 md 中画出一条分割线，不过在某些网站分割线显示比较粗；在 html 中分割线的是一个单标签 `<hr/>`；使用起来也没有比 `---` 复杂很多，并且可以通过也是控制分割线的粗细：

    <hr style="height:1px" />

这样可以画出一条比较细的分割线。

#### 段落开头的空格

     md 的正文一般情况无法通过键盘上的空格键输入空格，正文的文字都是居左的，因为 html 忽略了空格；这时我们可以通过以下 3 种方法解决；
1. 使用 html 中表示空格的字符 ` `，需要注意一定要有分号；
2. 把输入法切换为中文全角，然后再键入空格键；(这种方式不是总生效)
3. 给`<p>`标签加样式:

      <style>
        p{
          text-indent:2em;
        }
      </style>
      <!-- 2em 可以根据自己的需要调节；-->

#### 增加换行

md 中段落与段落或与其他标签之间换行的操作是 `空格 + 空格 + 回车`，但是和空格类似，md 中我们无法通过键盘 `Enter键` 来增加多余的换行；同样我么你可以通过内嵌 html 的 `<br/>` 标签来实现换行。

#### 图片大小与位置

`![图片描述](图片链接)`是 md 中用来显示图片的语法，对应的 html 标签是 ``，通过 `` 和 `<p>` 标签来实现图片的展示，我们可以实现更为丰富的表现力。

一：调整图片大小：图片调整小![](markdown_01.png)可以和谐的和文字放到同一行，具体方式是在 `img` 标签中使用 `width` 属性：

    ![](markdown_01.png)
    <!-- ps：用 width 调整图片的大小；src 后写上图片的资源路径。-->

二：调整图片居中样式，例如下边这张居中的图片：

![](markdown_01.png)

md 源码:

    <p style="text-align:center;">
    ![](marddown_01.png)
    </p>
    <!-- ps: 这里的做法是把 img 标签作为 p 的子标签，然后调整 p 标签的内容居中样式，当然也可以居右。-->

三：图文共处一行(区域大小会跟随图片大小变化而变化): 当然这种比较复杂了，毕竟 md 的初衷就是简化富文本书写；是否使用，自己取舍。

<div style="display:inline-block; border:2px blue solid; ">
 ![](markdown_01.png)

 图片居右，文字在左，示例， 图片居右，文字在左，示例，图片居右，文字在左，示例， 图片居右，文字在左，示例，图片居右，文字在左，示例， 图片居右，文字在左，示例，图片居右，文字在左，示例， 图片居右，文字在左，示例。
</div>

md 源码：

    <div style="display:inline-block; border:2px blue solid; ">
     ![](markdown_01.png)
     文字部分文字部分...
    </div>

#### md 中外链 iFrame 视频

<iframe src="player.html?aid=17494235&cid=28571298&page=1" 
    scrolling="no" 
    border="0" 
    frameborder="no" 
    framespacing="0" 
    allowfullscreen="true">
</iframe>

md 源码

    <iframe src="player.html?aid=17494235&cid=28571298&page=1" 
        scrolling="no" 
        border="0" 
        frameborder="no" 
        framespacing="0" 
        allowfullscreen="true"> 
    </iframe>

**如何从视频网站获取嵌入代码**

以 [bilibili](https://www.bilibili.com/video/av17494235?from=search&seid=13567678272662079887) 为例，打开后点击`分享视频` -> `嵌入代码` -> `复制`，然后把链接拷贝到 md 文件中，其他视频网站类似。

![](markdown_02.png)

### 二：Mac 上 Markdown 写作工具

* * *

付费软件就不说了，付费 md 书写软件数不胜数，貌似 UIysses 是付费系列中口碑不错的书写软件，不过价格也不便宜，所以我没用。免费的软件目前我只用下边两款。

#### [马克飞象](https://maxiang.io/)

就我个人而言，对于一般的需要产出 pdf 或者 md 的需求，我都直接打开 https://maxiang.io/ ，也是文章开头提到的`马克飞象`，打开即可书写，实时预览，分享链接给小伙伴，可以导出 pdf，可以绑定到印象笔记(这个功能需要付费)，同时它还具有浏览器缓存，即使关闭了浏览器，下次打开数据依然存在；最近，马克飞象也出了 Chrome App 和 Mac 客户端软件，免费软件中的首选。

![](markdown_03.png)

#### VS Code

[VS Code](https://code.visualstudio.com/) 功能非常强大，同样支持 md 书写和实时预览(安装插件)，软件大小 60 MB 左右，也比较轻量，推荐。

**本文同时发布与[我的 Gitbook 文集中](https://www.xiaobotalk.cn/)**

(完)

* * *