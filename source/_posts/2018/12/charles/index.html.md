---
layout: post
title: Charles 抓包工具
date: 2018-12-07
tags: ["breakPoints","charles","http","https","iPhone","Mac","Mac 高效使用指南","mock","日志","使用佳软","修改 Response","反向代理","抓包","抓取网络请求","证书"]
categories:
- 实用软件
---

![charles](charles_01.png)

Charles 也被称为 '青花瓷'，是一款桌面版网络抓包工具，对于开发人员来讲，是一款必不可少的效率软件；不仅能够方便的抓取 http https 网络请求，还能 mock 数据，模拟各种网络情况。[官方网址](https://www.charlesproxy.com/)。(本文以 Mac 系统上的 Charles 为例)

## 初识 Charles

* * *

Charles 界面结构分为两种：Structure 结构 与 Sequence 结构。

Structure 结构的界面下：

![charles 代理](charles_05.png)

左边是网络请求的具体 URL，右侧是具体的请求内容；

在 Sequence 结构的界面下：

![charles 代理](charles_15.png)

界面呈现为上下结构，并且多了一个 Filter 过滤器，方便快速查找要抓取得网络请求。

在使用时，需要勾选 Proxy -> macOS Proxy：

![charles 代理](charles_06.png)

由于大部分浏览器内置了开发者工具，具有很多 Charles 类似的功能，所以charles 更多的是用来抓取其他桌面软件或者移动端网络请求，这对于移动开发人员快速熟悉项目的网络请求，查找问题都大有帮助。

## 抓取移动设备网络请求

* * *

虽然是一款桌面软件，但是依然可以用了抓取移动设备上的网络请求，但要求移动设备和电脑处于同一个局域网网段中；以 iPhone 为例：

打开手机的 **设置** -> **wifi** -> **Configure Proxy** ，然后配置代理 IP 为电脑 ip，端口号默认为 8888。

![charles 代理](charles_02.png)

具体的端口设置，在 Proxy 菜单 -> Proxy Settings 中设置；同时勾选 Enable transparent HTTP proxying。

![charles 代理](charles_09.png)

## https 网络请求抓取

* * *

**1、安装证书**

抓取 https 请求，需要先安装一些证书，在 Help 菜单下，选择 SSL Proxy ，依次安装证书：

![charles 代理](charles_08.png)

手机上安装时打开浏览器，输入："chls.pro/ssl" ，会自动跳转到手机系统设置中进行证书安装，安装完成需要信任证书。

**2、信任证书**

电脑端证书安装后，需要在钥匙串中进行信任。

![charles 代理](charles_17.png)

模拟器或者手机端也同样需要信任证书：

![charles 代理](charles_16.png)

**3、配置 SSL Proxy Settings**

证书安装好以后，在 Proxy 菜单中选择 SSL Proxy Settings ：

![charles 代理](charles_10.png)

然后勾选 Enable SSL Proxying，同时在下边的列表配置一个 &#42;.&#42; ，表示所有域名都拦截:

![charles 代理](charles_11.png)

完成。

## 常用功能介绍

* * *

**1、模拟慢速网络**

打开 Proxy 菜单中的 Throttle Settings 选择，可以进行慢速网络模拟设置，方便我们调试慢速网络环境下程序的健壮性。

![charles 代理](charles_14.png)

**2、修改 Response 数据和设置请求断点**

右键某个请求，选择 Map Remote 或者 Map Local 可以对请求的 Response 做数据 mock，Remote 模式是服务器模式数据 mock，local 模式下可以直接选择一个本地文件。实际开发中，如果服务器某个接口出现故障，可以写一份本地的 json 文件，然后用 Map local 的方式继续调试开发。

![charles 代理](charles_12.png)

右键菜单我们还可以使用 breakPoints 对请求加断点。

**3、反向代理**

打开 Reverse Proxies 选项，可以添加反向代理功能，例如下图，将 56390 端口映射到 www.talkCode.com 的 80 端口。

![charles 代理](charles_18.png)

charles 还有很多其他功能，由于篇幅问题，每个功能就不做具体展开，而且具体的使用需要在实际问题中体验。

**本文同时发布与[我的 Gitbook 文集中](https://www.xiaobotalk.cn/)**

(全文完)