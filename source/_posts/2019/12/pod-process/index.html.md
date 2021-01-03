---
layout: post
title: Cocoapods 是如何工作的
date: 2019-12-01
tags: ["cocoapods","install","iOS","Mac 高效使用指南","outdated","pod","pod repo","podspec","日志","specs","specs 源","update","Xcode","二进制","包管理","客户端开发知识","版本依赖","私有"]
categories:
- 计算机
---

作者：xiaobo

* * *

最近需要把 iOS 工程的 pod 库二进制化，但发现这方面的资料相对比较少；甚至一些介绍 cocoapods 的文章，也很少从宏观上梳理 cocoapods 的工作流程；大都是关于 pod 指令，以及如何安装使用的文章。

但指令只是达到目的工具而已，我觉得需要先把思路捋清楚，所以尝试简单直白的描述下 cocoapods 的工作流程。

## Cocoapods 是什么

* * *

抛开平台差异，cocoapods 就是个包管理软件，它和 homebrew，npm，等包管理软件类似，区别就是它管理的是 iOS 开发中， Xcode 工程的软件包；

和其他包管理软件类似，cocoapods 也有它的核心终端指令：`pod`；执行包的安装通过: `pod install`。

## 工作流程

* * *

cocoapods 的运行原理，其实很简单，它依赖三个核心概念：

> 1、Podfile 文件，用来写清楚需要安装哪些 pod 库。
> 
>   2、Specs 中央仓库，托管在 [github 上的文件仓库](https://github.com/CocoaPods/Specs)。类似一个 "路由表"，用来查询某个 pod 库的详细信息。specs 也常被称为"源"，或者"specs 源"。
> 
>   3、podspec 描述文件，安装某个 pod 库所需的详细信息，主要的几项包括：代码地址、作者、版本、以及代码文件是哪些...等等。

作为库的使用方，我们最常见的就是 Podfile 文件；如果库是的提供方，就需要编写 podspec，同时把写好的 podspec 提交到 specs 重要仓库。下图是它们三者的关系：

![cocoapods](cocoapods_01.jpg)

## 使用方: Podfile 文件

* * *

Podfile 文件描述了工程需要引入哪些的 pod 库，一般写法如下：

    # CocoaPods Source
    source 'https://github.com/CocoaPods/Specs.git'

    #..其他部分..

    pod 'Alamofire', '1.1.0',
    pod 'WebViewBridge', '1.0.0'

    #..其他部分..

上边是我们经常写的 Podfile 文件格式，需要说明两点：

**1、`source  'https://github.com/CocoaPods/Specs.git'`**

source 函数，用来告诉 cocoapods 去哪个 Specs 仓库中查询 pod 库，大部分情况下是 cocoapods 官方的 specs 仓库，也就是上边写的 git 地址。

如果有需要，可以自己创建一个 specs 仓库，同时也可以写多个 source 源，cocoapods 会依次查询:

> source  'https://github.com/CocoaPods/Specs.git'
>   source 'https://git.mywebsite.com/demo1.git'
>   source 'https://git.mywebsite.com/demo12.git'

**2、`pod 'Alamofire', '1.1.0'`**

这是我们常用的 pod 库的引入方式：通过版本号引入；这种方式下，当执行 `pod install` 的时候，cocoapods 就会在 Specs 仓库中去查找对应版本，然后根据该版本下的 `podspec` 描述去安装 (安装之前，会先计算依赖)。

**你可以通过下来三个简单的指令来查看 pod 命令的具体内容:**

1、找到 pod 命令的具体路径

    $ type pod

    pod is /Users/zhangsan/.rvm/gems/ruby-2.6.3/bin/pod

2、查看它的文件类型

    $ file /Users/zhangsan/.rvm/gems/ruby-2.6.3/bin/pod

    /Users/zhangsan/.rvm/gems/ruby-2.6.3/bin/pod: Ruby script text executable, ASCII text

3、浏览 pod 命令的具体代码

    $ cat /Users/zhangsan/.rvm/gems/ruby-2.6.3/bin/pod

    ... 内容省略 ...

所以 pod 是个 ruby 的可执行脚本，你可以简单理解为一个 ruby 函数，仓库名是它的第一参数，版本号是第二参数(不写版本号，默认引入最新版本)；

**除了例子中版本号引入 pod 的方式，pod 还支持其他方式引入:**

1、自己指定 git 仓库，以及 branch 或者 tag:

> pod 'Alamofire', :git => 'https://github.com/Alamofire/Alamofire.git', :branch => 'dev'
>   或者 tag
>   pod 'Alamofire', :git => 'https://github.com/Alamofire/Alamofire.git', :tag => '1.0.0'

2、还可以指向本地 path，`:path` 这种方式经常用在开发调试中：

> pod 'demoPod', `:path` => '/Users/zhangsan/....'
> 
>   这种方式下，pod 会自动查找该目录下的 podspec 文件进行安装。

3、或者，用最直接的方式，指定 `:podspec` ，使用这种方式，podspec 文件路径可以是本地 (file 资源)，也可以是一个 http 资源：

> pod 'demoPod', `:podspec` => '/Users/zhangsan/...'
> 
>   或者使用 http 的方式
> 
>   pod 'demoPod', `:podspec` => 'https://gitlab.some.com/whose/demopod/raw/master/demopod.podspec'
> 
>   注意：http 的方式下，该 http 服务必须是一个文件下载服务!

上边通过 `:branch`，`:tag`，`:path`, `:podsepc` 传递参数时候，都需要在最前边加 `:`，这其实是 ruby 语法中 Hash 的 `key`，也就是常说的字典的 `key`， `=>` 后边的值是 `value`，pod 函数会把所有用 `:` 开始传递的 key value 存入到一个 hash 中。

## Specs 源

* * *

前边提到，cocoapods 官方的 specs 源(或者叫 specs repo)，本质就是托管在 github 上的一个 git 仓库。里边记录了非常多的 pod 库的 podspec 信息。[点击去 github 查看](https://github.com/CocoaPods/Specs)。

1、实际开发中，我们可以添加私有的 specs repo，而不去直接使用上边提到的 cocoapods 官方的 specs repo。这也是比较推荐的一种方式，原因是官方 specs 越来越大，而实际我们只用到了其中的一小部分。直接 source 它，会严重拖慢 pod install 的时间；建立私有的 specs repo 也是加速 pod update/install 最有效的方式。使用下边的命令可以快速的自建 specs repo :

    $ pod repo add REPO_NAME SOURCE_URL

2、第一次执行 pod install/update/search/outdated ，会先把所有用到的远端 specs repos 拉取到本地，使用下边命令可以查看本地所有的 specs 源仓库:

    $ l ~/.cocoapods/repos

## 库提供方：podspec 描述文件

* * *

当我们需要把一个自己的代码库提供给别人使用时，就需要编写 .podspec 描述文件，下边是开源三方库 WebViewJavascriptBridge 的 podspec：

    Pod::Spec.new do 's'
      s.name         = 'WebViewJavascriptBridge'
      s.version      = '6.0.3'
      s.summary      = 'An iOS & OSX bridge for sending messages between Obj-C/Swift and JavaScript in WKWebViews, UIWebViews & WebViews.'
      s.homepage     = 'https://github.com/marcuswestin/WebViewJavascriptBridge'
     ...省略部分...
     s.ios.source_files         = 'WebViewJavascriptBridge/*.{h,m}'
     ...省略部分...
    end 

上边我们可以看到一些关于这个库的引入信息：版本号、homepage、源码文件有哪些等。

**1、使用 ruby 解析 podspec**

上边的第一行内容：

> Pod::Spec.new do 's'

表示，该文件里边描述的是一个 spec 对象 s。该对象定义在 `Pod` module 下，这种写法是 ruby 的语法。我们使用下边的 ruby 脚本，可以把这个文件的对象解析出来:

    // 导入 cocoapods 核心库文件
    require 'cocoapods-core'

    spec_obj = Pod::Specification.from_file(podspec_path)

`podspec_path` 是 podspec 文件的本地路径；解析后，我们就可以直接访问 spec_obj 中的所有属性：

    spec_obj.name (仓库名)
    spec_obj.version (版本号)
    spec_obj.attributes_hash (得到所有信息的 hash 字典)

`Specification` 这个类的具体使用，可以查看 [cocoapods 的官方 API](https://www.rubydoc.info/github/CocoaPods/Core/master/Pod/Specification)，访问该网站需要一些"上网常识"。

**2、使用 pod 命令生成标准的 podspec 文件**

一般，我们需要通过 pod 提供的标准命令来生成这个 podspec 文件：

> pod spec create [NAME ' https://github.com/USER/REPO]
> 
>   支持本地命名，或者一个通过一个远程仓库

**3、提交 podspec 到 specs 中**

写好 podspec 后，我们需要把这个文件提交到 specs 中，由于 specs 是一个 git 仓库，所以我们完全可以通过 git 命令提交。但这并不是 cocoapods 推荐的做法，建议使用 cocoapods 的命令:

> pod repo push artsy-specs ~/Desktop/Artsy+OSSUIFonts.podspec --allow-warngins

上边 artsy-specs 是 specs 仓库名。使用这种方式，cocoapods 会进行一次 lint 检查，来避免 podspec 文件书写格式出错，`--allow-warngins` 表示忽略警告，可以不加，也建议最好不加。

## 使用 pod 命令总结

* * *

**1、限制版本号的写法**

通过版本号引入仓库的时候，有三种写法：

a: 固定版本的方式引入

    pod 'demoPod', '1.1.0'

这种方式，引入时，版本固定为 1.1.0。

b: 大于或者等于版本号

    pod 'demoPod', '>= 1.1.0'

这种也比较好理解，只要比 `1.1.0` 大或者相等的版本，都可以引入。

c: 末尾变化的方式

    pod 'demoPod', '~> 1.1.0'

`~>` 表示的是末尾`大于等于`。也就是从 `1.1.0` 到 `1.1.99999`。 表示从 0 到无限大(理论上)。

**2、Pods 文件夹**

`Pods/` 是一个文件夹，所有已经安装的 pod 库的资源都在该文件夹下，类似 `npm install` 后的 `node_modules` 文件夹。一般该文件夹会被加到 `.gitignore` 中，不进行 git 提交，目的是为了节省 git 仓库的存储空间。

**3、Podfile.lock 文件**

这是一个 lock 文件，再每次执行过 pod install / update 后生成。该文件的内容是记录**已经安装**的 pod 库版本号，和版本之间的依赖问题。

    PODS:
      - GCDWebServer (3.5.3):
        - GCDWebServer/Core (= 3.5.3)
      - GCDWebServer/Core (3.5.3)

    DEPENDENCIES:
      - GCDWebServer (= 3.5.3)

    SPEC REPOS:
      https://github.com/CocoaPods/Specs.git:
        - GCDWebServer
    ......

多人开发中，提交代码的时候，最好每次都提交该文件，这样一定程度上减少不同人的开发环境下 pod 库版本不一致的问题，特别是当 Podfile 中不是使用写死版本号的情况。

**4、pod install / outdated / update**

**pod install**

一般情况，我们都执行的是 `pod install` 来安装 pod 库，

> 首次安装: 由于没有 podfile.lock 文件，pod install 会安装满足 Podfile 文件限制条件下的最新的 pod 库。首次安装如果本地还没有 specs repo，该命令会先下载 specs repos。
> 
>   非首次安装：这种情况，pod install 则会按照 podfile.lock 指定的版本进行安装，前提是 Podfile 没有发生变化。
> 
>   非首次安装，且 Podfile 中 Pod 库版本发生变化：此时，pod install 需要尝试下载满足条件的最新的 pod 库，如果找到，就用新版本和 podfile.lock 文件中的版本做依赖计算，通过后，进行安装。

除了首次安装，pod install 命令永远不会更新 specs repo，所以，大部分情况，Podfile 发生变更后，我们都需要执行 `pod update`。

**pod update**

一般 pod update 有两种用法：

    $ pod update (更新全部 pod 库)

    $ pod update PodName (更新执行 Pod 库)

pod update 要做的第一件事情，就是更新 specs repo。然后更新满足限制条件的最新的 pod 库。Podfile.lock 的版本限制会被忽略。

**pod outdated**

一般很少被用到，该命令是一条查询命令，不会对已经安装的 pod 库做任何更新。它的作用是检查已经安装的 pod 库是否有更新。它只做两件事情：

> 1、更新 specs repo
> 
>   2、计算出哪些 pod 库发生了更新，然后输出到 terminal。

## 参考文档

* * *

https://guides.cocoapods.org/

(全文完)