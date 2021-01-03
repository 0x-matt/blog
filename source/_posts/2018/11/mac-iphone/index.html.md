---
layout: post
title: Mac 与 iPhone 互帮互助
date: 2018-11-22
tags: ["airdrop","Apple configurator 2","configurator 2","handoff","iPhone","itunes","Keynote","Mac","Mac 高效使用指南","PPT","互联","备份","通信","遥控"]
categories:
- 实用软件
---

<style>
p{text-indent:2em;}`
</style>

近些年，苹果都在极力完善苹果生态系统，希望自家的 iPhone iPad Mac 都能互通有无，给用户更好的科技生态体验，本节重点介绍几个 Mac 和 iPhone 相互协作的一些功能。

![Mac 与 iPhone](Xnip2018-11-22_16-17-50.png)

### AirDrop 使用

* * *

不知道从哪个版本开始，苹果给 AirDrop 起来个中文名叫`隔空投送`；利用"隔空投送"，您可以通过无线方式将文稿、照片、视频、网站、地图位置等发送到附近的 iPhone、iPad、iPod touch 或 Mac。<span style="color:red">使用 AirDrop 的需要开启蓝牙，并且在同一网络环境中</span>；具体使用介绍可以参考：[苹果官方关于 AirDrop 的使用说明](https://support.apple.com/zh-cn/HT203106)。

AirDrop 几乎是我每天都会使用的功能，但奇怪的是我周围的好多人要么不知道，要么知道了也不怎么使用；据说现在有种社交，叫做  AirDrop 社交，打开你的 AirDrop，随便给周围开着 AirDrop 的人传一张漂亮的风景图；说起来也是一种奇妙的体验。如果是同事之间玩，还可以缓解一下紧张的工作气氛。

### Handoff

* * *

想象一种场景，在用你的 Mac 电脑工作，并且用 Command + C 拷贝了一段文字，这时你拿起旁边的 iPhone，打开备忘录，长按 -> 粘贴，刚才在 Mac 上拷贝的文字已经粘贴到你的手机里了，如果你凑巧还有一部 iPad，同样无需其他任何操作，可以直接粘贴文字到 iPad 里。

如果说`真正的科技就是让你感觉不到科技的存在`，那么上边这种场景就是这句话最好的案例说明；没错这就是 `Handoff` 为 Apple 设备间提供的`公用剪切板`功能，也是我最喜欢的`Handoff`的一个功能。

AirDrop 可以让你附近的 Apple 设备互传文件，那么 Handoff 则是更进一步的打通个人 Apple 设备之间的操作连贯性，比如你正在手机上用 Safari 浏览某个网页，到了电脑旁边，关闭手机，Mac 的 Dock 栏左侧会自动弹出一个 Safari 图标，打开后继续浏览你刚才的网页。这就是 Handoff 为我们带来的便利。

关于使用 Handoff 简单说明几点，<span style="color:red">Handoff 是在私人的 Apple 设备之间互联使用的，所以需要各个设备登录同一个 Apple ID，除此之外还需要接入相同的网络，蓝牙都处于打开状态；</span>

另外 Mac 和 iPhone 等设备上都有 Handoff 开关:
* Mac：选取苹果 () 菜单 >"系统偏好设置"，然后点按"通用"。选中"允许在这台 Mac 和 iCloud 设备之间使用接力"。
* iPhone、iPad、iPod touch：前往"设置">"通用">"接力"，然后开启"接力"。
* Apple Watch：在 iPhone 上的 Apple Watch 应用中，轻点"通用"，然后开启"启用接力"。

同样的 Apple 给 Handoff 起了个中文名叫 "连接互通"， 更多关于 Handoff 细节可查看苹果官方文档：[使用"连续互通"连接 Mac、iPhone、iPad、iPod touch 和 Apple Watch](https://support.apple.com/zh-cn/HT204681#hotspot)。

### 关于 iTunes

早期的 iTunes ，我想大部分用户主要是用来给 iPhone 下载软件，虽然当时 iPhone 上的 App Store 也很成熟，但是得益于电脑的大屏幕，在 Mac 上使用 iTunes 下载软件高加高效，体验也更佳；然而在 2017 年，苹果在新版本(12.7 版本)的 iTunes 中彻底取消了 App Store 功能，现在的 iTunes 已经不能用来给 iPhone 下载软件了，只能搜刮下它的剩余价值。

**给 iPhone 做备份**

首先用数据线连接上 iTunes，在 `摘要` 选项中可以看到同步相关的选项，操作起来也很简单便捷，建议定期用此功能为 iPhone 做备份。

<image src="Mac_iphone02.png" width="60%"/>

**给部分 iPhone 软件传输文件 (然而这个功能完全可以使用 AirDrop 来代替)**  

首先用数据线连接上 iTunes，选择 `文件共享` ，右边列表会出现支持文件传输的软件，选择你要使用软件，然后将文件拖入右侧区域，然后点击同步。

<image src="Mac_iphone01.png" width="60%"/>

**使用 Wi-Fi 来同步连接 iPhone**

首先用数据线连接上 iTunes，然后在 `摘要` 选项中，打钩 `通过 Wi-Fi 与此 iPhone 同步` ，然后点击 `应用` 来生效，然后你就可以拔掉数据线，开始你的无线 iTunes 生活；同时建议取消打钩 `连接此 iPhone 时自动同步`。

<image src="Mac_iphone03.png" width="60%"/>

### 使用 Apple configurator 2 来管理你的 Apple 设备

* * *

[Apple configurator 2](https://itunes.apple.com/us/app/apple-configurator-2/id1037126344?mt=12) <image style="text-align:center" src="Mac_iphone07.png" width="3%"/>也是 Apple 出品的一款用来批量管理 iPhone/iPad/iPod touch 的软件，相较与 iTunes ，Apple configurator 2 则显得不怎么为人所知，原因之一可能是 Apple configurator 2 的初衷主要是为企业或者学校提供的移动设备批量管理，但是据我观察，即使是企业或者学校也很少使用 Apple configurator 2 ，但其实 Apple configurator 2 才是一款功能丰富且强大的 iPhone 管理软件。

要想全面使用 Apple configurator 2 ，需要先让 Apple configurator 2 监督你的设备，在 Apple configurator 2 中点 '准备' <image style="text-align:center" src="Mac_iphone06.png" width="5%"/>，依次点击下一步，完成监督，需要注意的是，在开始之前，先做好备份，因为准备的过程会抹掉你的 iPhone 数据，完成监督之后就可以完全用 Apple configurator 2 来控制这台 iPhone 了，这下好玩了；你可以通过 Apple configurator 2 修改 iPhone 的名称，在锁屏上写上你的个性签名；

<image src="Mac_iphone05.png" width="60%"/>

配置任何你想配置的描述文件:

<image src="Mac_iphone04.png" width="20%"/>

即使不想用 Apple configurator 2 监督你的手机，Apple configurator 2 依然有很多作用；

*   查看和导出手机信息
*   数据备份和恢复备份
*   调整手机桌面布局
*   <span style="color:red">删除和添加描述文件</span>，这一项功能，普通用户目前只能通过 Apple configurator 2 来完成；很多时候用你的 iPhone 或浏览某些网站或做测试的过程中，会安装一堆的描述文件到手机里，具体路径: `设置` -> `通用` -> `描述文件`。有些描述文件无法在手机里删除，这个时候你可以用 Apple configurator 2 来删除无用的描述文件。

Apple configurator 2 的功能非常多，它的初衷就是给企业和学校来批量管理苹果设备的，但是如果你的个人 Apple 设备也比较多，不妨也试试用 Apple configurator 2 来管理；我能想到的一个很大的用处就是，如果你想让家里的老人或者小孩子也体验 iPhone ，但是他们又不太能控制好这台智能设备，那么你完全可以用 Apple configurator 2 来管理你新买的每一台 iPhone，方便家里人，使用起来也更加安全；

关于 Apple configurator 2 更多的使用说明，Apple 也有详尽的文档：[Apple Configurator 帮助](https://help.apple.com/configurator/mac/2.8/#/cadbf9c93f) 或者 [PDF 文档](http://www.king-on.com.tw/upload/liam/kingon/_manage_download_Z5AF373DA4D204145C23.pdf)。

### 使用 iPhone 遥控 Keynote 播放

* * *

对于经常大屏幕讲演 Keynote 的人来说，一般都需要一款遥Keynote 翻页笔；但其实除了翻页笔，我们也可以用 iPhone 来充当 Keynote 翻页笔，功能毫不逊色于翻页笔，功能上 iPhone 相较于翻页笔有过之而无不及，学会使用 iPhone 控制Keynote 播放，可以省下去买 1 百多块的翻页笔了。

连接使用起来也非常简单，打开手机上的 Keynote 软件 (如果没有去 App Store 下载一个)，点击右上角的遥控器按钮 <image src="IL_Remote_2x.png" width="3%"/> ，就可以搜索到你的 Mac ，点击开始播放。

Apple 也给出了官方的关于 iPhone 操控 Mac Keynote 的详细教程和说明: [Keynote for iPhone: 使用遥控器控制演示文稿](https://support.apple.com/kb/PH24289?locale=en_US&viewlocale=zh_CN)。

**本文同时发布与[我的 Gitbook 文集中](https://www.xiaobotalk.cn/)**

(完)