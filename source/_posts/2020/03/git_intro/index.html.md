---
layout: post
title: 讲明白-git-底层原理
date: 2020-03-08
tags: ["--mixed","blob","commit","git","git add","git cat-file","git hash-object","git index","git init","git ls-files —stage","git object","git objects","git reset","git update-index","git 原理","git 底层","Git与版本控制","HEAD","heads","index","objects","日志","refs","SHA-1","tree","文件系统","版本控制","计算机基础知识"]
categories:
- 计算机
---

作者：xiaobo

* * *

![git](branching-illustration@2x.png "git")

本篇主要从版本控制的基本概念出发，试图说清楚，Git 的底层原理。文章篇幅很长，这也是没办法的事，不可能简单几句就能说清楚一个如此强大的 Git 的原理，我也会尽量避免少说废话，所以请耐心读完，我相信你会有所收获。

## 一：版本控制的概念

* * *

尽管 Git 最初的作者 [Linus Torvalds](https://en.wikipedia.org/wiki/Linus_Torvalds) ，在他的一篇回复中称 Git 是一个文件系统：

> In many ways you can just see git as a filesystem - it's content-addressable, and it has a notion of versioning, but I really really designed it coming at the problem from the viewpoint of a _filesystem_ person (hey, kernels is what I do)
> 
>   -- https://marc.info/?l=linux-kernel&m=111314792424707

但随着 Git 的普及，大家还是称 Git 为版本控制系统；另外大部分情况，我们对 Git 拿来即用，很少考虑过**什么是版本控制** ？所以我想先展开讨论下**版本控制**的概念，其实很简单，每个用过电脑的人都做过版本控制：

假设李雷同学快大学毕业了，有份_毕业论文_需要修改，较为灵活的做法是：

> 1：复制一份该文件，假设复制后的文件名叫做 '毕业论文 副本'。
>   2：李雷对 '毕业论文 副本' 文件进行了修改。
>   3：万一改乱了，也没事，大不了李雷再拷贝一份原始 '毕业论文' 文件，重新改。

上述过程，我们就可以认为李雷同学完成了一次版本控制。上述操作中，李雷总共有两份文件，即源文件 '毕业论文' 与副本文件 '毕业论文 副本'；然后，我们在给它加上版本号：

> 源文件 '毕业论文' ，1.0.0 版本。
>   副本文件 '毕业论文 副本'，1.1.0 版本。

这样就是一个完整的版本控制了。当然，实际情况可能是下图这样的：

![](lunwen.png)

这种版本控制方式，可以称之为 "**命名式版本控制**"。这种命名式版本控制操作简单，没有任何学习成本，但有很明显的缺点：

> 1: 文件个数庞大的时候，无法通过这种方式管理。
>   2: 版本的粒度基本上只能是以文件为单位。
>   3: 回溯历史不方便。
>   4: 无法多人合作编辑同一个文件。

使用版本控制软件，就能很好的解决上述问题、还能做得更好：

> 1: 版本控制软件能够自动追踪被它管理的目录下的所有文件，我们只需要专注于文件内容自身的增删改查即可。
>   2: 粒度可控，你可以只修改某个文件的几行内容后，就更新版本；或者修改若干文件后更新版本。
>   3: 版本信息明确、丰富。
>   4: 可以快速回溯历史。
>   5: 可以很好的完成多人协作。

同样以上述管理毕业论文的场景举例，使用 Git 来管理版本，就会显得有序又优雅：

![git](git_dir.png "git")

当然，Git 主要还是用来管理文件的，所以我们暂且就听作者的话，Git 就是个文件系统，只不过它刚好有版本控制的概念。

## 二：Git 的工作目录 (如何不使用`git init`来创建一个 Git 的工作目录?)

* * *

当我们开始使用 Git 的时候，第一个命令就是 `git init`，这个命令可以帮助我们创建一个 git 目录。
创建成功后目录下会自动生成一个 .git 的隐藏文件夹，我们在终端通过 `tree .git/` 可以看到如下的目录结构:

    .git
    ├── HEAD
    ├── config
    ├── description
    ├── hooks
    │   ├── applypatch-msg.sample
    │   ├── commit-msg.sample
    │   ├── fsmonitor-watchman.sample
    │   ├── post-update.sample
    │   ├── pre-applypatch.sample
    │   ├── pre-commit.sample
    │   ├── pre-push.sample
    │   ├── pre-rebase.sample
    │   ├── pre-receive.sample
    │   ├── prepare-commit-msg.sample
    │   └── update.sample
    ├── info
    │   └── exclude
    ├── objects
    │   ├── info
    │   └── pack
    └── refs
        ├── heads
        └── tags

(注意，在不同的 Git 版本与操作系统平台下，上述结构可能存在一些小的差异）所以，你可以完全不用`git init`命令来创建一个 git 的工作目录，具体办法就是参照上边的 .git 文件夹结构，依次创建上述文件和文件夹；事实上只需要创建好下列的目录结构，就已经是一个可用的 git 目录了：

    .git
    ├── HEAD
    ├── objects/
    └── refs
        ├── heads
        └── tags

(文末的附录1-1中，给出了如何通过`mkdir` `touch` `echo` 等基本的 Shell 命令来创建一个 git 工作目录。)

1、 HEAD 是一个文件，表示头指针，指向当前的分支，切换分支，会改变文件内容：

> ref: refs/heads/develop

你也可以通过直接修改文件内容的方式，来完成分支切换。

2、config 配置文件，里边存储着 username、email、remote 等配置信息。
3、objects 是一个文件夹，里边存储着工作目录下所有的对象，对象分为三类：

> 1: Blob 对象：二进制文件对象。
>   2: Tree 对象：文件夹对象。
>   3: Commit 对象：提交对象。

关于这三个对象，暂且介绍到这里，后文详细展开分析。

4、refs 是一个文件夹，通常有 2-3 个文件夹：

> 1: heads 文件夹：记录了所有的 branch 信息。
>   2: tags 文件夹: 记录了所有 tag 信息。
>   3: remotes 文件夹：如果关联了远程仓库，会看到这个文件夹。

我们通过 tree refs/，再次展开 refs/ 文件夹看一下：

    refs
    ├── heads
    │   ├── develop
    │   ├── feat
    │   │   └── 200301
    │   │       └── dependency
    │   │           └── mid
    │   └── master
    ├── remotes
    │   └── origin
    │       ├── HEAD
    │       ├── develop
    │       └── feat
    │           └── 200301
    │               └── dependency
    │                   └── mid
    └── tags
        └── 0.0.2

之所以嵌套这么多层，是和我们的分支命名有关，比如 heads 文件下可以看出，当前仓库有 3 个分支：

> 1: develop 分支
>   2: feat/200301/dependency/mid 分支  (`/` 会让 heads 下产生多级目录)
>   3: master 分支

每个分支都是一个文件，比如上边的 develop 就是一个文件，master 、mid 也是文件，里边只存放了一行数据：

    $ cat .git/refs/heads/master
    b80988c144729733bba9fc6be209760b3fbecebb

这串字符串是一个 40 位的 `SHA-1` 值，指向的是一个 Commit 对象。.git 的目录结构，暂且就分析到这里。

## 三：完成一次完整的提交，搞清楚工种目录、暂存区与本地仓库的概念

* * *

下边三个概念是我们使用 git 的时候，经常提到的概念：

> 1: 工作目录
>   2: 暂存区
>   3: 本地仓库

![git](git_stage.png "git")

1: 工作目录，就是我们需要编辑的文件的根目录；
2: 本地仓库你可以简单的理解为 .git 目录，或者.git/objects 文件夹;
3: 三个概念中最难理解的就是"暂存区"，但其实，如果我们仅仅通过`git init`新创建一个 git 工种目录，此时还没有 "暂存区"。只有第一次使用了 `git add` 命令后，git 才会创建暂存区，具体就是在 .git 目录下创建一个 index 文件，所以暂存区的物理结构就是一个 index 文件:

    .git
    ├── HEAD
    ├── index (暂存区文件)
    ├── objects/
    └── refs
        ├── heads
        └── tags

为了一探究竟，我们通过在目录下新建一个 a.txt 文件，并执行 `git add a.txt`

    $ touch a.txt
    $ git add a.txt

执行完成后，.git 目录就会新生成一个 index 文件，查看该文件内容可以通过`git ls-files --stage`命令：

    $ git ls-files --stage
    100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0   a.txt

    // e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 是 a.txt 的 SHA-1 值

该信息表明，a.txt 被暂存区记录，并生成了一个 Blob 对象。如果你的终端使用`oh my zsh`，你应该还会看到如下信息：

![git status](git_status.png "git status")

绿色的提示，也向我们传达了 a.txt 被 git 暂存区记录。

搞清楚这些概念，我们来看看一次完整的 Git 本地提交，都做了些什么事情：

![git_add](git_09.png "git_add")

上图应该很直观了，需要说明的是 git reset 的默认参数是 `--mixed`。

## 四：Git 是个 Key-Value 文件系统

* * *

如果用一句话描述 Git 的底层原理，那就是：Git 是一个通过 Key-Value 方式进行存储的文件系统。Key 就是我们前文提到的 `SHA-1`，Value 一共有 4 (通过可能是前三种)种：

> 1: Blob 对象：二进制文件对象。
>   2: Tree 对象：文件夹对象。
>   3: Commit 对象：提交对象。
>   4: 文件(夹)名称：刚才我们在暂存区中有看到 `e69de29`: a.txt 的 Key Value 格式。

为了专注理解 Git 原理，我们一般只说前 3 个对象，即：Blob/Tree/Commit。如果用一张图表示，就是这样：

![git key value](git_value.png "git key value")

SAH1 是对文件或文件夹使用 SHA-1 hash 算法求得的字符串结果，一共 40 位； Blob/Tree/Commit 这三个对象全部存放到上文提到的 .git/objects/ 目录下：
![sha1](git_sha1.png "sha1")

info 和 pack 两个文件夹，可以忽略，它们用于存放一些对象信息和压缩信息，是 Git 对存储空间做出的优化。另外三个文件夹分别是 Blog/Tree/Commit 对象。**文件夹**名称是 SHA-1 值的前两位，文件名称是 SAH-1 的后 38 位：
![sha1](git_40_sha1.png "sha1")

要想查看这些文件的具体内容，需要借助一个 Git 的底层命令：

> git cat-file -p // 查看文件内容
>   git cat-file -t // 查看文件类型

**4.1、查看 Commit 对象的信息**

> git cat-file -p 1d905ab5b9287ce2ea4dfa6aaae0fe7de6d68634

![git commit](git_commit.png "git commit")

上图信息结果表明，Commit 对象总共包含了：

> 1: 指向了一个 Tree 对象
>   2: author 信息
>   3: 提交人 committer 信息
>   4: commit 的描述信息
>   5: parent 指向父 Commit 结点 (上图没有展示出来，实际存在)

**4.2、接着查看 Tree 对象的信息**

> git cat-file -p 65a457425a679cbe9adf0d2741785d3ceabb44a7

![git Tree](git_tree.png "git tree")

一个 Tree 对象里包含所有若干 Blob(或者其他Tree) 对象，这就好比一个**文件夹**包含了若干**子文件夹**和**子文件**。因为图示的例子中只有一个 a.txt，所有只有一个 Blob 对象。

到这里，我们应该解开了这三个对象的秘密和它们的关系：

> 1: 一个 Commit 对象除了一些基本信息，最重要的是它指向了一个顶层的 Tree 对象；注意是顶层的 Tree，也就是最外层的文件夹。
>   2: 一个 Tree 对象又包含了若干其他 Tree 对象或者 Blob 对象；
>   3: Blob 对象就是一个文件的二进制内容。

结合上分支，用一张图表示它们的关系，如下：

![git object](git_all_ob.png "git object")

## 五：Git 是如何存储文件差异的

* * *

先说答案：**Git 从不存储文件"差异"，它保存的是完整的文件。**这是 Git 的特点，我没记错的话 SVN 存储的是 diff。为了更直观的理解，我们假设通过 Git 提交了如下 3 个版本的信息：

![git file](git_file.png "git file")

3 个版本，每个版本都指向了在2 个文件：

> 1: 版本1 有两个文件 A 和 B。
>   2: 版本2 相对版本1 只修改了 A 文件，得到 A1 文件。
>   3: 版本3 相对版本2 只修改了 B 文件，得到 B1 文件。相对版本1 修改了 A、B 两个文件。

此时 Git 会存储 4 个文件：

> 1: A 完整文件
>   2: A1 完整文件
>   3: B 完整文件
>   24: B1 完整文件

并不是 6 个文件，也不保存文件差异；但不同版本下，相同的文件只保存一份，以此来减小 Git 的存储空间太大的问题，另外 Git 会对 Blob 文件进行压缩处理，用图表示如下：

![git snapshot](git_snapshot.png "git")

另外一个重要信息是，Git 只对文件内容做 Hash 计算，和文件名、创建时间等都没有关系，这意味着，如果 a.txt 和 b.txt 两个文件的内容完全相同，那么 Git 只会生成一个 Blob 对象存储在 .git/objects/ 文件夹下。这样能进一步确保 Git 的存储空间不会很大。

你可能会问，如果是这样，Git 如何区分它们是不同的文件？答案是，通过**存储 SHA-1 和 文件名的 Key-Value 对**，这是一个非常巧妙的做法，这一点也很好验证：同时新建两个空文件 a.txt 和 b.txt，然后通过 git add 把它们都提交到暂存区，然后通过 `git ls-files --stage`。

    $ git ls-files --stage
    100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0   a.txt
    100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0   b.txt

同时你可以观察 .git/objects/ 文件夹下，只有一个 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 的Blob 对象。

## 附录

* * *

**1-1 如何不使用`git init`来创建一个 Git 的工作目录?**

    1：创建 objects 和 refs/heads 文件夹
    $ mkdir -p .git/objects/ .git/refs/heads

    2: 创建 HEAD 文件，并指向 .git/refs/heads/master
    $ echo 'ref: refs/heads/master' > .git/HEAD

经过上边两步，你的文件夹就会变成一个 git 的工作目录。

**1-2 Git 的的一些底层命令**

    1. 计算文件 hash: git  hash-object (-w) -stdin

    2. 通过 SHA-1 查看文件(类型)内容: git  cat-file -t/-p -stdin

    3. 浏览暂存区(index)文件内容:  git ls-files -stage

    4. 检出文件: git  checkout - [filename]

    5. 更新暂存区内容：git  update-index -add -cacheinfo 100644 [commit] [filename]

(全文完)

## 相关阅读

* * *

1、[Git merge commit 与 revert merge commit](https://www.xiaobotalk.com/2019/06/git-merge-revert/)

2、[Git 进阶知识点](https://www.xiaobotalk.com/2018/12/git-advanced/)

## 参考文档

* * *

1: https://git-scm.com/
2: https://github.com/git/git/blob/master/README.md