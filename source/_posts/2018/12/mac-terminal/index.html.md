---
layout: post
title: Mac Terminal 配起来
date: 2018-12-08
tags: ["Homebrew","iTerm","Mac","Mac 高效使用指南","oh my zsh","日志","Terminal","使用佳软","终端"]
categories:
- 计算机
---

![Terminal](terminal_01.png "Terminal")

终端 Terminal ，是每个操作系统都会附带的软件；黑色的背景，简单的界面，里面密密麻麻写了好多符号，其他软件都在不停的更新设计，可终端貌似永远都长这样。

## 要不要用终端

* * *

不会使用终端对于日常的工作，甚至是编程都不会有什么阻碍。但是如果会使用终端会让你使用电脑的效率得到很大的提升，特别是 Mac 系统的终端生来就无比强大。

举例来说，Mac 自身不支持对 rar 压缩文件的解压，需要通过三方软件来实现，而这些三方软件稍微好用些的大都收费；而通过终端，我们只需要先执行 `brew install unrar`，然后 `unrar x [文件路径/文件名]` 就搞定，看起来毫不费力。

如果你在浏览器上找到了某个视频资源，想下载到本地，可是网站并没有提供下载链接，这时我们可以找到视频资源的资源地址，然后使用 `wget URL` 或者 `curl URL` 实现视频资源的下载。

还有一个原因是，终端是每个操作系统出厂自带的，学会终端，也能快速的帮助别人解决电脑问题，即使他没有安装某些软件。

所以我比较推荐学习使用终端，虽然它看上去很难使用，但只要坚持使用一段时间，你会离不开他。

## Homebrew

* * *

Homebrew 是 Mac 的包管理软件，类似 linux 中的 yum，对于一台新 Mac，要打算用终端，那么从安装 Homebrew 开始；安装 Homebrew 只需要一行命令。

    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

使用时，指令都是 `brew` 作为开头，像上文提到的，安装 unrar 解压缩指令，使用 `install`。

    brew install unrar

几乎所有的终端指令都与上边的这条类似，可简化为如下格式：

`[指令名]` `[动作]` `[作用对象]`

*   brew 是指令名
*   brew 做的动作是 install
*   作用对象 unrar

如果复杂点的就是在`[动作]`后边带了一些参数；brew 指令不止 `install` 这一个`[动作]`，更多可参考 [Homebrew 官网](https://brew.sh/)。

网上有人写了一个漂亮的 mac 信息输出指令，可以使用 brew 安装试一试:

    brew install archey

安装完成后，使用 archey 指令：
![Terminal](terminal_09.png "Terminal")

## 选择 zsh

* * *

shell 是就像它的意思一样(贝壳)， 它是 linux/unix 系统的贝壳，被它盖住的是系统内核，我们可以通过它发出指令来让内核干活。

在 Mac 的终端里输入 `cat /etc/shells` ，会得到 shell 的所有种类:

    /bin/bash
    /bin/csh
    /bin/ksh
    /bin/sh
    /bin/tcsh
    /bin/zsh

最强大和使用度最高的是 zsh，系统默认是 bash，zsh（Z shell），是由 Paul Falstad 开发的开源Unix shell。它集成了所有现有shell的思想并增加了许多独到的功能，为程序员创建了一个全功能的高级shell。zsh 与其他 shell 相比有很多优势，比如最长使用的 **命令补全 (tab)** 和 **路径跳转 (cd -)**，在 zsh 里都特别好用，更别说它还帮我们做了些终端主题配色。

使用 zsh，执行:

`chsh -s /bin/zsh`

切换为 zsh 后，终端的配置文件使用 `.zshrc`，该文件在 `~/.zshrc`，`~` 表示当前用户目录。`.`开头的文件或文件夹是隐藏的，我们需要通过 `ls -lah` 才能看到。

## 安装 oh my zsh

* * *

**Oh My Zsh** 是一款社区驱动的命令行工具，基于zsh命令行，提供了主题配置，插件机制，已经内置的便捷操作。**Oh My Zsh** 开源在 [GitHub](https://github.com/robbyrussell/oh-my-zsh) 上，安装也是一行命令:

    sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

或者

    sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"

安装后，第一感觉就是变漂亮了，最让人兴奋的是，如果一个路径下是 git 仓库目录，oh my zsh 做出了美观实用的显示设计:

![oh my zsh](terminal_02.png)

#### 主题

oh my zsh 有很多主题可以选择，每种主题都有不一样的终端显示效果，主题在 ~/.oh-my-zsh/themes/ 路径下，如果想要换主题，打开 ~/.zshrc 文件，修改 ZSH_THEME 即可。

    ZSH_THEME="robbyrussell"

改完后需要

    source ~/.zshrc

生效。

#### 插件

oh my zsh 提供的插件之多几乎涵盖了大部分编程语言和工具，在终端执行 `ls ~/.oh-my-zsh/plugins` ，便可以看到

![oh my zsh](terminal_03.png)

具体想使用某个插件，同样编辑 ~/.zshrc 文件，找到如下代码：

    plugins=(
      git
      bundler
      dotenv
      autojump
      osx
      rake
      rbenv
      ruby
    )

把你需要的插件添加到列表里，即可。

## 使用 iTerm2

* * *

相比于系统的 Terminal，iTerm2 似乎更加广受好评，iTerm2 给用户带了更多的可配置选项，比如终端的外观主题，你可以在终端万年不变的背景放一张你喜欢的壁纸；还有 iTerm 好用且可配置的快捷键；有些东西我们还是需要跟风的，所以就使用 iTerm2 吧，用一段时间就会明白它的好。[iTerm2 官网](https://www.iterm2.com/)

iTerm 的主题以及快捷键配置都在，它的偏好设置里，可以根据自己的喜好进行配置:

![iTerm2](terminal_04.png)

**本文同时发布与[我的 Gitbook 文集中](https://www.xiaobotalk.cn/)**

(完)