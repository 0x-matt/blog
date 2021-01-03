---
layout: post
title: Shell 使用简介与 Mac 特有的命令
date: 2018-12-11
tags: ["&gt;","&gt;&gt;","alias","cat","cd","find","grep","history","launchctl","Mac 高效使用指南","mkdir","open","pbcopy","pbpaste","日志","shell","type","unix","which","内建命令","指令","标准输入，标准输出","管道","重定向"]
categories:
- 计算机
---

几乎所有的 Unix 系统及其衍生系统 (Linux macOS BSD...)，都会内置 shell，所以从来不需要编译或者安装 shell，开盖即用。

shell 也是一门编程语言，想要全盘掌握，并非一朝一夕，所以本文并不是介绍 shell 的语法细节；因为很多时候，对于实际使用，我们不需要全面掌握 shell，不了解它的语法细节也不会太影响我们使用它的**内建命令**。

## 内建命令

* * *

**内建命令**就是由 Bash 自身提供的命令，更像是其他编程语言中自身已经实现的函数库，等着我们调用。例如 `cd` 切换目录的命令，就是一个内建命令。

> 说明：本文使用的 shell 类型是 zsh ([上一篇文章](https://www.xiaobotalk.com/archives/393)已经提到过)，系统是 macOS。

**1、`type` 查询命令**

type 应该是第一个需要掌握的命令，这个命令就是用来查询某个命令是不是内建命令，例如上文中提到的 `cd`:

    -> type cd
    cd is a shell buildin

当我们在终端输入 `type cd` 的时候，打印出 cd is a shell buildin ；说明 cd 是一个内建命令。

当然 type 命令的功能不只是用来查询内建命令，终端上所有的可用命令，它都可以查询，例如:

    -> type node
    node is /Users/zhangsan/.nvm/versions/node/v10.0.0/bin/node

node 并不是内建命令，这时候，终端会显示命令所在的具体路径；这个功能在实际使用中很有用，一些三方命令，我们想修改它的配置，却找不到它的具体安装路径时，可以借助 type 来查找。

**2、`which` 查询命令**

which 命令和 type 命令，就终端打印来看，功能非常类似，都是用来查询命令的，不过 which 更偏向于查询可执行文件的具体路径。例如我们用 which 查询 cd

    -> which cd 
    cd: shell built-in command

结果是 cd 是一个 command (命令)。

    -> which more 
    /usr/bin/more

more 是一个可执行文件，路径是 /usr/bin/more。

**3、`alias` 别名命令**

alias 是用于创建别名的命令，如果直接输入 `alias`，不带任何参数，会打印出当前用户的所有别名命令。

    -> alias
    l='ls -lah'
    la='ls -lAh'
    ll='ls -lh'
    ls='ls -G'
    lsa='ls -lah'
    md='mkdir -p'
    gcs='git commit -S'
    gcsm='git commit -s -m'
    gd='git diff'
    gdca='git diff --cached'
    gdct='git describe --tags `git rev-list --tags --max-count=1`'
    ...

使用 alias 自定义别名，例如 git add 命令，我们觉得太长，可以使用 alias 进行重新命名:

    -> alias ga='git add' 

最后如果想删除某个别名，使用 `unalias` 命令：

    -> unalias ga 

**4、cd 命令**

cd 命令使用很简单，进行目录跳转:

    -> cd ~

跳转到当前用户目录，`~` 表示用户目录。

但是在 zsh 中，cd 跳转有很多可玩性和便利性，例如，我们输入 `cd -`(减号) ，然后敲 **TAB** 键:

    -> cd - <TAB>
    1 -- ~/Documents/我的Shell脚本
    2 -- ~/Documents

列出本次登录后去过的最近几次路径。然后根据前边的数字编号，进行相应跳转:

    -> cd -2 <回车>
    1 -- ~/Documents/我的Shell脚本
    2 -- ~/Documents

会跳转到 Documents 目录。

**5、常用浏览命令**

浏览命令可能是使用频率仅次于 cd 命令的:

*   `ls -G`: 浏览当前目录下的所有文件和文件夹，oh my zsh 插件中为它起了别名为 `ls`。
*   `ls -lah` : 浏览当前目录下的所有文件和文件夹，包括隐藏文件；别名：`l`
*   `cat` 终端下浏览某个**文件**的所有内容，注意是文件。
*   `head` 终端下浏览某个**文件**的头部内容，注意是文件。如果文件过大，只想浏览前几行，可使用 head。
*   `tail` 与 head 对应，终端下浏览某个**文件**的尾部内容，注意是文件。
*   `history` 会显示出用户输入的所有历史命令。
*   `pwd` 单独使用，显示当前文件夹路径。

**6、`find` 文件查找命令**
find 并不是一个内建命令，是一个可执行文件，方便我们快速查找文件。

`find` `[目录]` `[查找条件]` `[参数]`

例如在 home 目录下查找出所有 .txt 文件:

    find /home -name "*.txt"

-name 表示以文件名进行查找。

**7、其他常用命令**

*   `touch` : 新建文件
*   `mkdir` : 新建文件夹
*   `cp` : 拷贝文件
*   `mv` : 移动文件，可以用来重命名文件
*   `rm (-rf)` : 删除文件(夹)

## 标准输入输出与重定向

* * *

如果要更加高效的使用 shell 及 Terminal，就得了解**管道**，**标准输入输出**，**重定向**等概念。

#### 标准输入输出

系统在启动一个进程的同时会为该进程打开三个文件：标准输入(stdin)、标准输出(stdout)、标准错误输出(stderr)，分别用 0、1、2 来标识(在 Linux 中用 0-9 作为文件标识符，指明了数据流)；

默认情况下，系统会把**标准输入**设定为键盘，**标准输出**和**错误输出**都是显示器。

#### 重定向

重定向就是，将任何文件、命令、脚本、程序的输出重定向到另一个文件、脚本、命令、或者程序中。

重定向分类

<table>
<thead>
<tr>
  <th align="center">符号</th>
  <th align="center">含义</th>
</tr>
</thead>
<tbody>
<tr>
  <td align="center">></td>
  <td align="center">标准输出覆盖重定向：  
将命令的输出重定向到其他文件中，会覆盖原来文件中的所有内容</td>
</tr>
<tr>
  <td align="center">>></td>
  <td align="center">标准输出追加重定向：  
将命令的输出重定向到其他文件中，不会覆盖原文件内容，只会将内容追加到文件尾部。</td>
</tr>
<tr>
  <td align="center"><</td>
  <td align="center">标准输入重定向：  
用来指定标准输入的来源为某个文件，而不是键盘。</td>
</tr>
<tr>
  <td align="center">&#124;</td>
  <td align="center">管道：  
从一个命令中读取输出重定向为下一个命令的输入。</td>
</tr>
<tr>
  <td align="center">>&</td>
  <td align="center">标识输出重定向：  
将一个标识的输出重定向到另一个标识的输入</td>
</tr>
</tbody>
</table>

**1、标准输出重定向 `>` 与 `>>`**

例如我们两个文件: result.txt 和 source.txt

    -> cat source.txt
    hello source

source.txt 文件有一行 'hello source' 。

    -> cat result.txt
    hi result

result.txt 文件有一行 'hi result' 。

默认情况，cat 命令，会把文件内容打印在终端界面上，因为默认标准输出为显示器。

使用 `>` 来把 source.txt 文件内容覆盖重定向到 result.txt。

    -> cat source.txt > result.txt
    -> cat reslut.txt
    hello source

这时 result.txt 内容变为 'hello source'。原来的内容被覆盖掉。

如果使用 `>>` 做相同操作:

    -> cat source.txt >> result.txt
    -> cat reslut.txt
    hi result
    hello source

这时 result.txt 内容变为两行:

    hi result
    hello source

`>>`把内容追加到文件尾部。

**2、标准输入重定向 `<`**

标准输入重定向用来指明输入读取的来源：

    -> pbcopy < demo.txt

上边的指令会把 demo.txt 的文件内容读取到 pbcopy 指令中，pbcopy 是剪切板指令，这时我们在其他地方 Cmd + V 粘贴，就会把 demo.txt 内容粘贴过去。

**3、管道 `'` 与 `grep` 抓取命令**

管道是在使用命令行的时候经常使用的命令间通信机制，从字面意思来看，很容易想到生活中的水管、暖气管等。管道的命令行符号 `'` 也很形象的隔离了各条指令。我们从一条简单的指令连接来理解管道

![](terminal_05.png)

在 `'` 左边的命令 `ls` 会输出当前路径下的所有文件和文件夹，然后将这些输出通过管道传输给下一条命令，并且作为下一条命令的输入；然后通过 `grep` 抓取命令，抓取包含了 'Do' 的文件或文件夹。

管道和 `grep` 经常一起使用，来完成一些快捷操作，例如我们想删除某个 git 仓库下所有的包含了 bugfix 分支名的分支：

![](terminal_06.png)

先复习下使用 grep 抓取出所有的 bugfix 分支:

![](terminal_07.png)

继续使用管道进行分支批量删除操作:

![](terminal_08.png)

完整：`git branch ' grep 'bugfix' ' while read li; do git branch -D $li; done;`

使用了两次管道 `'` 进行数据过滤，最后把要删除的若干分支名传入给 while 循环，执行 `git branch -D`。

**4、标识输出重定向 `>&`**
小节的开头我们知道，0 表示标准输入，1 表示标准输出， 2 表示标准错误输出， 0 1 2 就是文件标识符；

有时候我们可能需要将错误输出和普通输出都写入到同一个文件，这时可以使用 `>&` 进行标识符重定向：

    -> COMMAND > stdout_stderr.txt 2>&1

从左到右：执行 COMMAND 命令，将输出的内容重定向到 stdout_stderr.txt，2>&1，表示如果有错误，将标准错误 (2) 重定向到 标准输出 (1) 中。

## Mac 系统特有的指令

* * *

介绍了 Shell 后，我们来看看 macOS 有的而其他系统没有的一些指令，这些指令同样可以结合重定向使用。

**1、`open` 打开目录、文件、软件。**

    -> open /Applications/Xcode.app
    -> open a.txt
    -> open ./home/ (打开 home 文件夹)

**2、pbcopy 和 pbpaste**
这两个都是剪切板命令，在介绍输入重定向的时候已经见到了 pbcopy:

    -> pbcopy < demo.txt

上边的指令会把 demo.txt 的文件内容读入到剪切板中。

pbpaste 从剪切板中读取内容写入到 demo1.txt 中：

    -> pbpaste >> demo1.txt

上边的指令会把剪切板中内容重定向到 demo1.txt 文件中。

**3、launchctl 开机启动命令脚本**

launchctl 用来控制启动 Mac 时需要开启的服务。和 Linux cron 有些类似。

查看所有的启动服务

    -> sudo launchctl list
    PID Status  Label
    -   0   com.apple.SafariHistoryServiceAgent
    250 0   com.apple.Finder
    312 0   com.apple.homed
    369 0   com.apple.SafeEjectGPUAgent
    78013   0   com.apple.quicklook
    ...

添加 Apache 的开机启动：

    -> sudo launchctl load -w /System/Library/LaunchDaemons/org.apache.httpd.plist

还可以添加定时任务，更多可查看[苹果官方文档](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html)。

**4、say 让命令行说话**
say是一个文本转语音（TTS）的 mac 命令行工具，直接在后边跟上一段话，电脑就会开始朗读：

    -> say "Hello 主人"

使用`-f`参数选择朗读的文本文件，然后用`-o`参数将朗读结果存储为某个音频文件：

    -> say -f demo.txt -o demo.aiff

macOS 的系统命令非常多，感兴趣可以查看[此文档](https://ss64.com/osx/)。

**本文同时发布与[我的 Gitbook 文集中](https://www.xiaobotalk.cn/)**

(完)