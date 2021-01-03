---
layout: post
title: 集成 Flutter 到已经存在的 iOS 工程中
date: 2020-01-13
tags: ["App.framework","flutter","Flutter.framework","FlutterEngine","git","integration","iOS","日志","Submodule","Xcode","前端知识","客户端开发知识","混合开发","集成"]
categories:
- 计算机
---

作者：xiaobo

* * *

![flutter](flutter_top.png "flutter")

## 一个纯 Flutter 项目的工程结构

* * *

在开始学习 Flutter 之前，我们往往需要一个 Flutter 工程，用下面的命令，可以快速的创建出一个 Flutter 工程:

    $ flutter create --template module my_flutter

如果一切顺利，就会看到如下的目录结构(部分)

    my_flutter/
    ├─.android/
    │ │-app/
    │ │-gradle/
    ├─.ios/
    │ ├─Runner.xcworkspace
    │ └─Flutter/podhelper.rb
    ├─lib/
    │ └─main.dart
    ├─test/
    └─pubspec.yaml

`.ios` 和 `.android` 分别是 iOS 和 Android 工程目录，他们都是 Flutter 项目的一个子目录。

而在实际开发中，更多的是在一个已经存在的 iOS 工程中，接入 Flutter，实现部分业务用 Flutter 开发。显然，我们要将 Flutter 作为 iOS 工程的子目录，刚好与上边新建的工程结构是反过来的，而 Flutter 在这方面支持的并不友好。

## 用 Git Submodule 让目录结构共存

* * *

假设我们已经存在下列三个 Git 仓库：

![git-repo](flutter_git.png)

上图中的 Flutter 仓库，是需要同时被 iOS 和 Android 引入的。同时还要保证独立的 Git 提交。即，我们只在 Flutter 的 Git 目录下提交代码 Flutter 代码，这样保证 Flutter 的开发只发生在同一个地方。

解决思路是使用 Git Submodule ：

![git-repo](flutter_git_submodule.png)

Git Submodule 使用比较简单，我们先进入到已经存在的 iOS 工程目录下，这里假设 flutter Git 仓库的地址为 `https://github.com/someone/my_flutter`。

**1: 添加 submodule**

    $ git submodule add https://github.com/someone/my_flutter

上边的命令，会在项目目录下创建 .gitmodules 文件，来记录所有的 git submodule :

    [submodule "my_flutter"]
        path = my_flutter
        url = https://github.com/someone/my_flutter
        branch = master

**2: 更新 submodule 模块**

    $ git submodule init / update

注意，首次安装，用 `init`，后续用 `update` 更新。

如果一切都正常，那么包含一个 Flutter Git Submodule 的 iOS 工程目录大概是这个样子：

    ios-app/
    ├─Podfile
    ├─Pods/
    ├─.xcworkspace
    └─my-flutter/

my-flutter 文件夹就是 Flutter 工程的 Git 仓库，如果需要进行 Flutter 开发，`cd my-flutter/` 文件夹下，进行编码，开发完成后，你可以像使用一个正常的 Git 仓库一样，提交代码。

**3: Git Submodule 良好的使用习惯**

1.  Git Submodule 与主项目是通过 commit 建立联系，如果 submodule 下产生了新的 commit 提交，这个 commit 号会被主项目记录，这时候，最好在主项目上再进行一次提交。</p>
2.  可以通过如下配置: `git config submodule.recurse true`，让主工程执行 `git pull` 的时候，自动更新 Submodules。

## Flutter 的三种构建模式

* * *

<p>在开始集成之前，我们先要清楚 Flutter 的三种构建模式：

**1: Debug 模式**

该模式下，为了开发便利，Flutter 会启动一个 Dart 虚拟机，然后在虚拟机中执行 Flutter 源码；每当开发者修改了 Flutter 源码，变化会立即反应到 App 上(模拟器或者真机)，不需要重新运行 App 以及编译所有源码，大幅的提升了开发效率；这也被称为 `Hot Reload`。实现该机制的编译方式称为 JIT 编译，类似 web 开发。

可以想象，该模式导致 App 运行性能极差，但不需要关心，只要开发方便即可。所以当我们在开发 Flutter 程序时，应该使用此种模式集成 Flutter 到项目中。

**2: Release 模式**

该模式用于发布环境中，采用 AOT 编译，提前把所有 Dart 源码编译为机器码，过程中会进行编译优化，提升执行效率，所以该模式 Hot Reload 无效。当我们准备打包 App 的时候，应该使用此模式进行 Flutter 集成。

**3: Profile 模式**

该模式用于对 Flutter 进行性能分析，性能分析一般需要用真机进行调试，使用 DevTool 进行分析。

## 要集成的产物分析

* * *

在 Flutter 工程目录下执行下列：

    $ flutter build ios-framework --output=./f_products

会生成对应模式下的产物 (./f_products 是产物的具体路径，可以自己指定):

    ./f_products
    └── Flutter/
        ├── Debug/
        │   ├── Flutter.framework
        │   ├── App.framework
        │   ├── FlutterPluginRegistrant.framework
        │   └── example_plugin.framework
        │
        ├── Profile/
        │   ├── Flutter.framework
        │   ├── App.framework
        │   ├── FlutterPluginRegistrant.framework
        │   └── example_plugin.framework
        │
        └── Release/
            ├── Flutter.framework
            ├── App.framework
            ├── FlutterPluginRegistrant.framework
            └── example_plugin.framework

所以我们可能需要导入到项目中的 Framework 有 4 个:

> Flutter.framework  (flutter 引擎)
> 
>   App.framework   (Flutter 业务代码和各种资源)
> 
>   FlutterPluginRegistrant.framework (Flutter 插件注册)
> 
>   example_plugin.framework (具体引入的 Flutter 插件，多个插件都会生成多个 `example_plugin`)

如果没有用到插件，则不会有 `example_plugin` 和 `FlutterPluginRegistrant` 两部分，也就不需要集成。

## 较为理想的集成结果

* * *

通常情况，团队内不是每个人都需要开发 Flutter，并且 App 的打包通常会由一台跑着 Jenkins 的机器来完成，所以对集成的结果提出以下几点要求：

> 0: 采用 Cocoapods 集成 Flutter
> 
>   1: flutter 开发人员：具备一键启动 debug 调试模式的能力。
> 
>   2: 非 flutter 开发人员：不需要安装 flutter 环境，对 flutter 无感知。
> 
>   3: 打包的机器上无需安装 Flutter 环境，这一条和 2 要求一致。
> 
>   4: Flutter 项目的 Git 仓库和 iOS (Android) 项目的 Git 仓库关联，方便共同开发和提交。这一点已经通过 `Git Submodule` 解决。

## 集成 Fluter 产物到 iOS 工程中

* * *

使用 Cocoapods 集成，我们需要先创建 Git 仓库，用来存放 Flutter 的 frameworks，然后把这些 Git 库做成 Pod 仓库。我的建议是创建三个 Git 仓库：

1.  Flutter 资源库，存放 App.framework 和 FlutterPluginRegistrant.framework。
2.  FlutterEngine 引擎库，Flutter.framework 不需要经常变化，一个 Flutter 版本对应一个 Engine 库。如果没有升级 Flutter 版本，则不需要更新该引擎。
3.  Flutter 插件库，如果用到了 插件，可以把所有的插件统一放到一个 Git 仓库下。

App.framework 可能每天都需要更新版本，如果和 FlutterEngine 引擎库放在同一个 Git 仓库，那么每次更新都需要重新下载并没有发生变化的 FlutterEngine 引擎库，并且 FlutterEngine 大小在 350 M 左右。所以拆分是非常有必要的。

创建好后，把每个仓库，做成 Pod 库，然后通过 cocoapod 引入到工程中。

后续为了方便说明问题，只以 Flutter 资源库为例，其他两个库原理相同，假设 Flutter 资源库的 Git 仓库为：

> FlutterSources

## 自动化发布 framework

* * *

假设已经创建好 FlutterSources 仓库，接下来我们进行以下步骤，完成 Flutter 库的 Pod 化：

**1. 构建 frameworks**

flutter 命令

    $ flutter build ios-framework --output=./f_products

需要注意，一般我们需要同时拷贝 Debug 和 Release 两个文件夹下的 App.framework。

**2. 为 FlutterSources 创建 .podspec 文件**

podspec 中设置两个 subspec：

> Debug 
>   Release

Debug Subspec 指向 Debug/framework；Release 指向 Release/framework。默认使用 release 即可。

    s.default_subspec = 'release'

    s.subspec 'debug' do 'ss'
       ss.ios.vendored_frameworks = 'debug/App.framework'
    end

    s.subspec 'release' do 'ss'
       ss.ios.vendored_frameworks = 'release/App.framework'
    end

    s.dependency 'FlutterEngine', '1.12.13+hotfix.5' # 需要依赖 FlutterEngine

这里需要依赖 FlutterEngine 库，确保独立库编译通过，所以理论上先制作 FlutterEngine pod 库。

**3. 提交 podspec 到私有的 specs 中**

    pod repo push someone_specs #{spec_path} --allow-warnings

如果没有依赖 `FlutterEngine` 即 `Flutter.framework`，这一步会编译失败，导致提交失败；还有种办法，是在私有的 specs 仓库下 push --force 强行发布(不推荐)。

到这里，仓库制作完成，上述的每一步，都可以用脚本来实现自动化，我用的是 ruby 脚本，使用 ruby 方便操作 spec 对象。

### 脚本的大致思路如下

iOS 下，我一般习惯用 ruby 脚本，原因是 Cocoapods 就是 Ruby 实现，使用 ruby 可以方便的调用 Cocoapods 相关的库函数。

**0: 库导入**

> require 'fileutils'       # 文件操作工具库
> 
>   require 'cocoapods-core'  # pod 核心工具库

**1: 拉取 FlutterSources Git 仓库**

> cmd = " git clone #{FlutterSources_ssh_url} #{path} "

path 可以是用户目录下，自建的一个隐藏文件夹(`.`开头的文件夹)，用于存放 FlutterSources Git 库，只有首次需要 clone。
通过 **`ENV["USER"]`** 环境变量可以获取到当前的用户，从而找到用户目录。

**2: 更新仓库**

> cmd = "git -C #{path} pull -f"

`-C` 表示，在 path 路径下执行 git pull -f，因为执行脚本时，我们处于 flutter 的工作目录，所以需要通过 -C 指向到另一个仓库执行 git 命令。

**3: 生成 flutter frameworks**

> cmd = "flutter build ios-framework --output=./f_products"

**4: 拷贝 App.framework 文件**

此时需要用到 fileutils 工具库:

> FileUtils.cp_r(source_dir, dest_dir)

**5: 更新版本号**

脚本可以接受一个版本号参数，或者版本号自动末尾自增。

这里需要使用 `Pod::Specification` 类：

    origin_spec = Pod::Specification.from_file(spec_path) 

从一个 .podspec 文件中的读取出 Specification 对象，然后通过 `origin_spec.version` 可以获取到原始版本号，修改后写回到 podspec 文件中 ，`Specification` 是 Cocoapod 提供的类库，定义在 `Pod module` 下，[官方文档](https://www.rubydoc.info/github/CocoaPods/Core/master/Pod/Specification) 有详细的使用介绍。

Cocoapod 没有提供把一个 Pod::Specification 写回到 .podspec 文件的方法，我们需要自己实现。

**6: 打 tag ，push 仓库，更新 specs**

> cmd = "git -C #{root_path} add . && git -C #{root_path} commit -m '更新 flutter 到 #{version} 版本' "
> 
>   repo_push_cmd = "pod repo push someone_specs #{spec_path} --allow-warnings --skip-tests"

整个过程略微繁琐，但并不是很复杂。

另外，在 Ruby 中我们需要执行 shell 命令，使用如下:

    cmd = "git -C #{path} pull -f" 
    system cmd # 执行 shell，system 会同步执行 shell

如果想拿到 shell 执行的返回值，可以通过下面的方式：

    flutter_path_cmd = "which flutter ' awk -F 'bin' '{print $1}'"
    flutter_path = `#{flutter_path_cmd}` # flutter_path 就是 shell 的执行结果

## Podfile 中导入

* * *

已经创建好仓库，接着，可以直接在 Podfile 中导入：

    pod 'FlutterSources', '1.0.0'
    pod 'FlutterEngine', '1.0.0'

默认导入的是 Release 包，如果需要导入 Debug 包，则使用 `Debug` Subspec

    pod 'FlutterSources/debug', '1.0.0'
    pod 'FlutterEngine/debug', '1.0.0'

## 结合使用 Debug 环境下从本地导入的方式

* * *

Flutter 官方示例中，展示了通过 Cocoapods 从本地导入 Flutter 的方法：

    flutter_application_path = '../my_flutter'
    load File.join(flutter_application_path, '.ios', 'Flutter', 'podhelper.rb')

    target 'MyApp' do
       install_all_flutter_pods(flutter_application_path)
    end

我们已经通过 `Git Submodule` 把 Flutter 库集成到了工程的当前目录下，所以 Debug 模式下，可以通过此方式直接从本地集成，不需要从远程的 Git 仓库下载，更为方便。

为了能一键切换为本地 Flutter Debug 模式，我创建一个 `flutter_pod` 的 ruby 函数，专门处理 Flutter 库的导入：

    # 检查环境变量 DEVFT
    # DEVFT=1 pod install 命令下生效

    def is_dev_flutter 
        return ENV['DEVFT'] == "1"
    end

    def flutter_pod(fl_path)
        if is_dev_flutter # dev 开发模式
            if is_env_ok 
                puts "********环境 ok 开始安装 flutter 测试代码********"
                unless ENV['FLEVNOK'] == "1"
                    puts "********加载 flutter 工程路径********"
                    load File.join(fl_path, '.ios', 'Flutter', 'podhelper.rb')
                    ENV['FLEVNOK'] = "1";
                end
                install_all_flutter_pods(fl_path)
            else 
                raise "flutter 测试工程未初始化! 请先按照上边提示初始化 flutter 环境，然后再执行 DEVFT=1 pod install"
            end
        else
            # 从远程仓库拉取 release 仓库
            pod 'FlutterSources/debug', '1.0.0'
            pod 'FlutterEngine/debug', '1.0.0'
        end
    end

    target 'MyApp' do
       flutter_pod './my_flutter' # 通过 flutter_pod 函数处理 Flutter 的库导入
    end

然后，如果需要切换为本地 Flutter 调试的时候，只需要执行:

> DEVFT=1 pod install

这样就为 Flutter 开发人员提供了便利；而正常的 `pod install` 环境下，开发人员对 Flutter 的存在无感知。

## 参考文档

* * *

*   [Integrate a Flutter module into your iOS project](https://flutter.dev/docs/development/add-to-app/ios/project-setup)

(全文完)