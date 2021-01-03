---
layout: post
title: 有趣且有用的网站
date: 2019-11-26
tags: ["api","Mac 高效使用指南","日志","random","使用佳软","图片","在线","感想与观点","网站"]
categories:
- 实用软件
---

作者：xiaobo

* * *

![random](900x300 "random")

最近发现了一些比较不错的在线网站或者 api；例如，上边的这张图片，是通过 unsplash 网站的随机图片接口`https://source.unsplash.com/user/erondu/900x300` 返回，刷新一次网页，换一张图片，请随便刷新 (F5 或者 Cmd + R)。

## 高清图片 API

* * *

我们经常需要一些高清的图片链接，作为网站的背景图，或者丰富网站的内容；搜了一下，才发现这样的服务有好多，大都可以免费试用，并且可以定制大小、种类、甚至还有简单的滤镜效果。

**1、[Lorem Picsum](https://picsum.photos/)**

支持 path 大小设定，毛玻璃，灰色系列，使用简单，[直接看官网示例](https://picsum.photos/)即可。

**2、必应图片服务，按天更新**

    https://api.dujin.org/bing/1366.php 

**3、[unsplash](https://source.unsplash.com/)**

和 picsum 类似，通过不同的 url 的 path 获取不同种类的图片，特点是可以根据用户，和点赞数获取。[点击跳转官网](https://source.unsplash.com/)

**4、[shibe.online](https://shibe.online/)**
该网站，主要是提供图片 api 接口，图片种类主要是阿猫阿狗，可以批量获取一个图片列表；[点击跳转官网](https://shibe.online/)直接看使用方法。

## 在线压缩图片、gif、pdf 网站

* * *

`https://docsmall.com/` 是一个在线压缩图片和 gif、pdf 的网站，再保持清晰度的情况下，压缩比例能达到 70% 以上，也就是说一张 5M 的图片可以压缩到 3M 以内，还保留清晰度。[网站链接](https://docsmall.com/)

## 在线录屏网站

* * *

`https://www.p2hp.com/screenrecord.html` 是一个在线录制屏幕网站，作为网页版录屏，功能非常丰富和强大。[网站链接](https://www.p2hp.com/screenrecord.html)
![screenrecord](screenrecord.png)

## YES or No 结果网站

* * *

遇到某个问题，犹豫选 yes 还是 no ？不如问问 `https://yesno.wtf/`，这个网站非常有意思，每次访问，随机告诉你一个 yes 或者 no。同时配有一张经典电影的动图，非常喜感。说明下，随机不是平均。[网站链接](https://yesno.wtf/)

同时你还可以使用它的 api 服务获取 json 数据：

    https://yesno.wtf/api

返回的 json  格式：

    {
        "answer": "yes",
        "forced": false,
        "image": "https://yesno.wtf/assets/yes/0-c44a7789d54cbdcad867fb7845ff03ae.gif"
    }

## JsonApiPlaceholder: Json Api 占位服务

* * *

学习软件开发的时候，找不到线上 json 服务做调试？访问  [JsonApiPlaceholder](https://jsonplaceholder.typicode.com/guide.html)，然后专注于你的学习。

## 开放 API 搜索

* * *

在写一些小软件的时候，我们经常需要找一些免费开放的 API 进行调用，现在好了，有网站帮我们做了开发 API 搜索服务：
1. [http://apis.io/ ](http://apis.io/)  

2. [https://www.programmableweb.com/](https://www.programmableweb.com/)

## 发现中国，志愿者运营网站

* * *

这是一个志愿者们开发的[发现中国](https://www.ageeye.cn/)网站，代码开源在 gitee 上。[网站链接](https://www.ageeye.cn/)

## 中国国家数字图书馆

* * *

网站其实做的很简陋，不过可以检索各种图书，还是挺方便的。[网站链接](http://www.nlc.cn/)

## 在线 PS

* * *

我自己不会 ps，不知道功能到底如何。[网站链接](https://ps.gaoding.com/#/)

## 马克飞象

* * *

最后再次推荐下马克飞象，我经常使用它来写一些简单的文档；拿来即用，导出 pdf 也很方便，有浏览器缓存，能同步印象笔记，唯一的成本是需要学习下 Markdown，而这个网站还有学习教程，基本 10 分钟就学会了。但是我发现周围并没有什么人使用。 [网站链接](https://maxiang.io/)

* * *

还有更多好玩的网站，欢迎在评论区留言。

（全文完）