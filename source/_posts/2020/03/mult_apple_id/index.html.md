---
layout: post
title: 管理一个或多个 Apple ID
date: 2020-03-12
tags: ["App Store","apple","Apple ID","iCloud","Mac 高效使用指南","日志","感想与观点","美区","苹果"]
categories:
- 计算机
---

* * *

作者：xiaobo

![apple_id](apple_screen.png "apple_id")

很多人都有两个 Apple ID，一个国区的，一个美区的，甚至更多。最近由于某些原因，我自己经常需要在美区和国区的 Apple ID 之间切换，总结出了一些使用经验，记录一下，希望能对大家有帮助。

## Apple ID

* * *

一个 Apple ID 关联了很多属性，但总结起来可以归为两类：

1、iCloud 存储相关
2、App Store 、iTunes Store 下载相关

![apple id](apple_setting_id.png)

## Apple ID 登录入口

* * *

**1、手机的设置中登录**

![apple id](ios13-iphone-xs-settings-sign-in-to-your-iphone.jpg)

在这里登录，默认 iCloud 、App Store、iTunes Store 都会使用该账号进行存储、软件下载等。

**2、在 App Store 、iTunes Store 中登录**，最常见的是 App Store 中登录：打开手机中的 App Store 软件，在右上角能看见登录入口：

![apple id](app_store_id.png)

在此登录，该 Apple ID 只用来下载 App Store 软件，不会污染到你的 iCloud 数据。**所以，切换美区 Apple ID 下载软件的时候，不要直接在设置中切换(这会影响你手机中已经存储的 iCloud 数据)；只需要打开 App Store 退出之前的 Apple ID (退出登录按钮需要滑到页面最底端)，再次登录新的 Apple ID 即可。这里可以随意切换 Apple ID，而不会对手机数据有任何影响。**

总结，手机设置里，登录的是你经常使用的主体 Apple ID 账号，一般不需要登出。切换其他区的 Apple ID 进行软件购买和下载的时候，只需要在 App Store 中切换即可。

## iCloud 存储

* * *

1.  照片
2.  备忘录
3.  联系人
4.  提醒事项
5.  ...等

![apple id](icloud_setting.png)

除了图中所示，很多第三方软件也可以在获取 iCloud 授权 (当然，这是你自己授权的) 后，把软件相关的数据传到 iCloud 上。数据同步一般会在后台进行(一般在 wifi 状态下)，同步到 iCloud 服务器，目前国内的 iCloud 服务器苹果托管给了云上贵州。同步完成后，其他设备只要登录了此 Apple ID，同时打开 iCloud 同步，上述的各种数据都会慢慢同步到新的设备上，多久同步完，和你的网速有关。

iCloud 存储的免费空间只有 5G，用完了想继续试用，需要额外付费。5G 其实很少，特别是如果用来存储照片的话，很快就用完了，所以我个人从不打开照片的 iCloud 存储。其他的数据，例如备忘录、信息、邮件等存储起来都很方便。也占用不了多少空间。照片可以想其他办法存储，比如谷歌云相册，或者自己导入到硬盘里。

最后，如果迫切的想知道，自己的 iCloud 上究竟存储了哪些数据，**可以直接登录[https://www.icloud.com/](https://www.icloud.com/) 苹果 iCloud 官方网站进行查看、编辑等，可能还有惊喜。**

## 一个 Apple ID 可以在无限多台设备上登录

* * *

关于一个 Apple ID 可以同时在多少台设备上登录？我专门致电过苹果的客服，结论是可以登录到任意多台设备上。

通过这一点，你可以和你的亲人或者朋友，共享付费软件，从而节省一部分开支；只要某个 Apple ID 账户下购买了某个付费软件，那么其他设备上只需要在 App Store (注意，不需要在手机设置里登录) 里登录后，就可以直接下载该软件。但是希望你不要因此而成为一名淘宝卖家。

因为，很多淘宝卖家就是利用这一点赚钱的，在一个账号下购买付费软件后，通过收取比原软件低很多的价格，把自己的账号租给买家，从而实现付费软件的倒手赚钱。这种做法很不提倡，有两点不好：

1.  破坏正常的交易市场。
2.  软件的开发者费了很大的辛苦编写了软件，却得不到相应的报酬，结果是开发者失去更新和维护软件的动力和资本，导致软件质量越来越低，甚至下架该软件。最终转嫁为用户使用的软件体验越来越差。

最后，提倡有点经济能力的，可以支持一下付费软件，我们都知道，免费的才是最贵的。再说了，你差的可能不是那几十块钱，而是几个亿。

## 关于注册美区 Apple ID

* * *

这个其实很简单，网上教程很多，关键几步如下：

1.  先注册一个 outlook 邮箱 (微软的) 或者 gmail 邮箱，推荐微软的 outlook，国内外访问都不受限。
2.  准备好美国本土的地址信息和电话信息，该网站：http://www.shenfendaquan.com/ 可以帮你快速生成上述信息。
3.  然后，就可以前往 https://appleid.apple.com/account#!&page=create， 进行注册，地区记得选美国。
4.  在设备上，App Store 里进行登录激活，激活的时候，填写已经准备好的美国的地址和电话信息。

愉快的使用吧。

## 相关阅读

* * *

[XiaoboTalk: 使用 Gift Card 给美区 Apple ID 充值](https://www.xiaobotalk.com/2020/02/gift_card/)

(全文完，你可以点赞、分享、我都没有意见)