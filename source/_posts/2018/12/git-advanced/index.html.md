---
layout: post
title: Git 进阶知识点
date: 2018-12-19
tags: ["--hard","--mixed","--soft","checkout","commit","config","conflict","git","git remote","gitconfig","Git与版本控制","Mac 高效使用指南","merge","日志","rebase","remote","reset","修改历史","变基","合并","合并历史","合并历史提交","工作区","工作目录","暂存区","本地仓库","版本库","版本控制","解决冲突","远程仓库","重写历史"]
categories:
- 计算机
---

作为分布式的版本控制系统，Git 的操作指令非常多，但是我们可以灵活的组合使用一些常用指令，就可以玩转大多数的日常 Git 使用场景。

## 1、温习 Git 工作区域

* * *

先来温习一下 Git 的工作区域划分：**工作目录**、**暂存区**、**本地仓库**，实际操作中，我们需要知道每一步操作发生在哪个工作区域，那么再复杂的 Git 问题都能轻松解决。

先看一张逻辑工作区域划分图：

![git进阶](git_07.png)

物理工作区域划分图：

![git进阶](git_08.png)

> .git 是隐藏文件夹，大多版本控制软件都会用一个隐藏的文件夹作为其本地版本仓库，SVN 也是如此。用户的每一步操作都被这个隐藏文件夹里的代码记录着。

## 2、一次完整的本地提交

* * *

虽然 Git 是分布式的版本控制系统，工作中，我们都会有远程仓库的概念，但只需熟练玩转本地仓库即可，因为远程仓库不过是别人电脑上的本地仓库。接下来我们来看看一次完整的提交，是如何在三个工作区域之间转换的：

![git进阶](git_09.png)

上图中也引出了 git reset 回滚命令的三个参数：--soft --mixed --hard，三个参数回滚的程度逐渐增强，实际使用中可以根据具体需求灵活使用。

另外不要忽略了 git commit 只提交那些被记录到了暂存区的修改。

## 3、使用好 checkout

* * *

checkout 可能是 git 中比较多功能的一个命令。

*   `git checkout [分支名]` : 切换分支。
*   `git checkout [commit 序列号]` : 穿越到指定的某次 commit。
*   `git checkout -b [分支名]` : 创建并切换分支。
*   `git checkout --ours/theirs` : 解决冲突时用来检出某方的提交。
*   `git checkout [文件名/路径/.]` : 丢弃某些文件/文件夹/所有修改。

所以 checkout 这个检出操作，不仅仅是用来切换分支，还有很多强大的功能；注意点：`git checkout .` 指令丢弃的文件是无法找回的，使用请谨慎，防止车祸现场。

## 4、使用简单的组合命令进行历史修改

* * *

**修改历史中的某次提交信息**

由于某些原因，有时候总要修改一下提交历史，或者叫重写历史，最简单的修改最近一次提交命令 `git commit --amend`，然后会进入文本编辑状态，修改你的提交信息。

但是如果不是最近一次提交，而是历史中较远的前几条信息，那么单纯使用 `git commit --amend` 就不行了，比如下图这种情况，想要将 "第一次提交" 改为 "首次提交"。

![git进阶](git_10.png)

这时你可能马上会想到 git rebase 变基命令，但是对于这种情况，我们可以使用 `git checkout` 组合 `git commit --amend` 完成同样的工作:

1、先用 `git checkout 36ae20...` 穿越回第一次提交的节点中，此时我们执行 `git log` 看到的只有第一次提交的 commit:

![git进阶](git_11.png)

2、那么接下来你也猜到了，执行 `git commit --amend` 吧。
3、然后再次 `git checkout master` 穿越回当前分支的最新节点。

**合并多次提交历史为一次**

合并多次历史是比较常见的需求，同样我们先不用 rebase，使用一些常用的命令；例如我们要将上边例子中的三次提交合并为一次，我们可以先用 reset 回滚多次历史提交，然后 commit --amend 重写提交即可：

1、 先用 `git reset --soft 36ae20...`，软回滚到第一次提交(36ae20...是历史中第一次的提交序列号，这里也可以使用 `HEAD~3` 这种头指针回数的形式)，这时候 `git log` 又变成了只有第一次的 commit 信息：

![git进阶](git_11.png)

但是通过 `git status` 可以看到其他两次提交都已经被我们使用 reset --soft 将最近两次提交回滚到了暂存区：

![git进阶](git_13.png)

2、接下来，使用 `git commit --amend` 重新提交并修改最近一次的 commit 信息即可。

> 使用 reset --soft 我们将提交回滚到暂存区，这样可以在重新提交的时候，少写一次 `git add`。
>   如果多次合并的 commit 不是从最近一次的提交历史开始，那么久多使用一次 git checkout 。

你看，貌似不需要学习 rebase 也能完成 Git 重写历史。

## 5、使用 git rebase 修改历史

* * *

对于 `4、使用简单的组合命令进行历史修改` 中的案例，我们可以使用更加高级的 git 工具 `rebase`，使用 rebase 来干这些事情会显得更加专业，当然 rebase 的功能也更加强大一些。但是依然要写出来 `4、使用简单的组合命令进行历史修改` 这一部分内容，是想表达，有时候一些问题的解决办法并不唯一，灵活使用一些我们已经学过的知识点也能曲线救国；假如你还不会使用 rebase，眼下又个紧急的 git 历史合并任务要做，马上去学习 rebase 又有些来不及，那么只能急中生智了。

接着 4 中的例子，我们使用 `git rebase -i 36ae20...` ，然后我们会看到如下的文本编辑界面：

    pick 1f737d8 第二次提交
    pick 8e401d5 第三次提交
    pick bc9c6f0 第四次提交

    # Rebase d973330..bc9c6f0 onto d973330 (3 commands)
    #
    # Commands:
    # p, pick = use commit
    # r, reword = use commit, but edit the commit message
    # e, edit = use commit, but stop for amending
    # s, squash = use commit, but meld into previous commit
    # f, fixup = like "squash", but discard this commit's log message
    # x, exec = run command (the rest of the line) using shell
    # d, drop = remove commit
    #
    # These lines can be re-ordered; they are executed from top to bottom.
    #
    # If you remove a line here THAT COMMIT WILL BE LOST.
    #
    # However, if you remove everything, the rebase will be aborted.
    #
    # Note that empty commits are commented out

注释中，我们看到 reword edit squash 等指令，这里我们把 pick 改为 edit。

    edit 1f737d8 第二次提交
    pick 8e401d5 第三次提交
    pick bc9c6f0 第四次提交

然后保存退出，接下来 git 会把你带入第二次的 commit 节点，并有如下提示：

![git进阶](git_14.png)

到这里，我们需要执行 `git commit --amend` 来执行 commit 修改。编辑保存后，再执行：

    git rebase --continue

然后 git 把你带回到 master 分支的最新节点，整个操作完成。

如果要合并提交，我们需要把 edit 改为 squash：

    pick 1f737d8 修改为第二次提交
    squash 8e401d5 第三次提交
    squash bc9c6f0 第四次提交

接下来，git 会进入另一个编辑界面：

    # This is a combination of 3 commits.
    # This is the 1st commit message:

    修改为第二次提交

    # This is the commit message #2:

    第三次提交

    # This is the commit message #3:

    第四次提交

    # Please enter the commit message for your changes. Lines starting
    # with '#' will be ignored, and an empty message aborts the commit.

这个界面里，可以修改你最终要显示的提交信息，这里直接保存退出，完成合并，此时 `git log` 显示如下：

![git进阶](git_15.png)

除了修改和合并提交，还可以使用 git rebase 进行拆分、删除提交操作，就不一一演示。

最终你会发现，使用 git rebase 来修改历史步骤比较繁琐，还不如使用 `4、使用简单的组合命令进行历史修改` 的方式来操作简单。

## 理解分支合并中的 Fast-forward/merge/rebase

* * *

在分支合并的场景中，我们有时会看到 Fast-forward ， Fast-Forword 很好理解，在 Git 中就是它直译过来的意思'快进'，下面我做了四个小视频，来解释 Fast-forward/merge/rebase。

观看视频前，我们以下图做个 demo 背景说明：

![git进阶](git_16.png)

图中有两条分支，master 和 feature1 分支，feature1 是从 master 的 `C2` 节点拉出来的分支，背景说明完毕。

**Fast-forward 合并模式**

<video src="fast-forward.mov" controls="loop" width="80%">
</video>

Fast-forward 的合并模式永远不会有冲突产生。

**非 Fast-forward 合并模式**

<video src="%20nofast-forward.mov" controls="loop" width="80%">
</video>

这种非 Fast-forward 的合并模式下才有可能发生合并冲突。

**merge 过程**

为了和下边的 rebase 做对比，视频中在 feature1 分支中做 git merge master 操作

<video src="from-feature-merge.mov" controls="loop" width="80%">
</video>

**rebase 过程**

<video src="rebase-process.mov" controls="loop" width="80%">
</video>

前边我们用 rebase 来重写历史，这里的 rebase 用来在分支之间**合并变基**，变基可以理解为**改变基点**，基点就是某条分支在另一条分支上的起点，这个点是出现分支的点；rebase 和 merge 在从结果上来看，及其相似，但是其过程却大不相同，我想看了视频，你应该理解了。

rebase 的目的并不是做一次合并，而是为了让某条分支与主分支的当前进程保持同步更新，进而在合入主分支的时候，能够以 Fast-forward 的过程合并，避免冲突；rebase 的过程有冲突的可能，但是这种冲突时发生在当前分支，不会影响主分支，所以也不会影响和你一起合作的其他小伙伴。所以在多人合作的时候经常使用 rebase 是个好习惯。

另外对于一些大型项目，为了回溯历史，定位问题方便，要求主开发分支只能有一个父节点，提交历史显示为一条直线，程序员把自己的开发分支合并进入主分支的时候是 Fast-forword 合并；所以进行合并之前做一次 rebase 操作是很有必要的。

**一次完整的 rebase 场景**

<video src="rebase-demo.mov" controls="loop" width="80%">
</video>

## 解决冲突

* * *

解决冲突其实很简单，理解了 git 本地操作和 git merge 过程，解决冲突就是个体力活了；cherry-pick/merge/rebase/revert 等操作都有可能发生冲突；

冲突产生时我们只需要通过 `git status` 查看具体发生冲突的文件，然后打开编辑文件，选择保留的内容后保存文件，然后继续 merge 或者 rebase，指令分别是：`git merge --continue` `git rebase --continue`；除了 `--continue` 参数，我们还可以使用 `--abort` 来终止 merge 或者 rebase 。

之所以说解决冲突是个体力活主要看，冲突文件的数量和内容多不多，如果冲突较多时，我们一一修改可能就比较累了，这时候可以借助 checkout 指令来保留冲突双方的其中一方修改：

    git checkout --ours // 保留自己的修改 
    git checkout --their // 保留别人的修改

注意点：rebase 的过程，ours 和 theirs 是反过来的。当然，我们也可以借助图像化工具来解决冲突：

![git进阶](git_17.png)

选择后，会进入图像化操作：

![git进阶](git_18.png)

## 关联远程仓库

* * *

当我们需要关联远程仓库的时候，可以通过 `git remote add [远程仓库命名] [ssh/http 地址]`，例如：

    git remote add origin git@git.coding.net:demo/demo.git

这里把远程仓库 git@git.coding.net:demo/demo.git 命名为 origin。一个仓库可以关联多个远程仓库，但是仓库名不能相同：

    git remote add gh git@git.github.com:demo/demo.git

这里，我们把另一个远程仓库 git@git.github.com:demo/demo.git 命名为 gb 关联了本地仓库，当我们需要同时往多个远程仓库推送文件时，可以使用了。

最后通过 `git remote -v` 可以查看本地仓库关联的所有远程仓库。

## git config 配置文件

* * *

在 Mac 系统中，git 的配置文件有两处，一个是全局配置文件：~/.gitconfig ，在用户目录下；另一个是当前仓库的配置文件，在 .git 文件夹下，两种重复时，优先使用 .git 文件下的 config 文件。

![git进阶](git_20.png)

config 文件的内容：

![git进阶](git_19.png)

一些情况下，我们可以直接编辑 config 文件来修改 git 的配置。

## 相关阅读

* * *

1、[讲明白-Git-内部原理](https://www.xiaobotalk.com/2020/03/git_intro/)

2、[Git merge commit 与 revert merge commit](https://www.xiaobotalk.com/2019/06/git-merge-revert/)

## git 常用指令表 参数省略

* * *

*   本地操作

        *   git init
    *   git add/rm
    *   git commit
    *   git squash
    *   git rebase
    *   git checkout
*   状态查看及信息检查和比较

        *   git status
    *   git log
    *   git diff
*   配置及别名

        *   git config
    *   git alias
*   分支操作

        *   git branch
    *   git merge
    *   git rebase
    *   git cherry-pick
    *   git checkout
*   远程操作

        *   git clone
    *   git remote
    *   git push/pull
    *   git fetch
*   git 时间旅行：分支切换、回退历史、保存/清理现场

        *   git reset
    *   git revert
    *   git stash
    *   git checkout
    *   git clean -fdx

(全文完)