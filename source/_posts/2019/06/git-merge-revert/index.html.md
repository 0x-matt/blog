---
layout: post
title: Git merge commit 与 revert merge commit
date: 2019-06-10
tags: ["-m parent-number","git","git revert","Git与版本控制","merge commit","日志","revert","撤销","父结点"]
categories:
- 计算机
---

## 快进/非快进合并

* * *

Git merge 本身并不难，就是把一个分支合入另一个分支，最常用的概念就是 fast-forward 和非 fast-forward。下面通过几张图来区分这两种情况。

### fast-forward 合并

![git-merge](merge_ff.png)

上图的 branch1 分支从 master 分支的 C2 结点分出来。然后在 branch1 上产生了 C3、C4 两个结点。此时在 master 分支上执行 `git merge branch1` ，git 在合并时就是 fast-forward 模式，即 master 的头指针直接移动到 C4 结点即可：

![git-merge](merge_ff_after.png)

由于是 fast-forward ，所以默认不会产生 merge commit；即提交的最新历史就是 branch1 的 C4。

### 非 fast-forward 合并

![git-merge](merge_no_ff.png)

上图中 branch1 分支也是从 master 的 C2 结点分出来，不过随后 master 分支也有了新的结点 C3，branch1 分支也产生了 C4、C5 两个新结点，此时在 master 分支上执行 `git merge branch1`，git 会产生一个新结点 C6 ，C6 结点有两个父节点：C3 与 C5：

![git-merge](merge_no_ff_after.png)

在这种非 fast-forward 模式下合并，git 默认会生成一个 merge commit ，即 C6 结点，并且 C6 写明两个父节点的 commit 号:

![git-merge](merge2.jpg)

非 fast-forward 下，有可能存在合并冲突的情况，也需要自己解决冲突，关于解决冲突可参考另一篇文章：[Git 进阶知识点](https://www.xiaobotalk.com/archives/402)

## 让 Git merge 总是生成 merge commit

* * *

在进行 git mrege 操作时，有很多可选参数，可以通过命令行 `git merge -h` 查看具体的 merge 可选参数：

![git-merge](merge_help.png)

默认情况下，参数是 `git merge <branch name> --ff` ，所以，如果是 fast-forward 合并，就不会产生 merge commit 。但是当用 gitlab 或者 github 进行合并请求合入的时候，即使是 fast-forward 也会产生 merge commit ，原因是 gitlab 或者 github 在合并请求合入时候的默认选项要求总是生成 merge commit :

![git-merge](merge_commit.jpg)

上图是 gitlab 合并请求时候的默认勾选，申明了为每一次 merge 生成 merge commit ，这样做在查看历史的时候，能清楚的看到哪些 commit 是 merge 进来的。如果我们手动合并也想总是生成 merge commit ，可以使用下面的合并方式：

> git merge <branch name> --no-ff

使用 `--no-ff`，git 会忽略掉 fast-forward 模式，为所有的 merge 生成 merge commit。

## 如何 revert 一次 merge commit

* * *

使用 git revert 可以用一次提交来撤销历史的某次提交，正常情况，我们只需要执行：

> git revert <commit>

即可，但如果 revert 的是 merge commit ，上面的指令就会报错：

> error: commit *** is a merge but no -m option was given
>   fatal: revert failed

原因是一个 merge commit 有两个父节点，所以 git 在 revert 一个 merge commit 的时候，需要知道，我们到底要 revert 那个具体的提交树中，下面是一个 merge commit 的 log 信息：

    commit 7e1c782316d8c2441a96da83113e49311fdbea88 (HEAD -> master)
    Merge: 9610578 9681b7a 
    Author: zhangsan <zhangsan@example.com>
    Date:   Mon Jun 10 22:27:48 2019 +0800

        Merge branch 'branch1'

可以看到第二行以 Merge 开头的信息：

> Merge: 9610578 9681b7a

指明了本次 merge 的两个父节点，在 revert 的时候，我们需要指定具体的 revert 的 parent-number (从 1 开始):

> git revert 7e1c782 -m 1
>   或者 
>   git revert 7e1c782 -m 2

`1` 代表的是当前主线分支的撤销，即回到 9610578 节点的状态，大部分情况下，这也是我们想要的结果。
`2` 代表撤销到 9681b7a 节点状态，一般情况，不会产生任何变化。

## 相关阅读

* * *

1、[讲明白 Git 内部原理](https://www.xiaobotalk.com/2020/03/git_intro/)

2、[Git 进阶知识点](https://www.xiaobotalk.com/2018/12/git-advanced/)

## 参考链接

1: [https://stackoverflow.com/questions/7099833/how-to-revert-a-merge-commit-thats-already-pushed-to-remote-branch](https://stackoverflow.com/questions/7099833/how-to-revert-a-merge-commit-thats-already-pushed-to-remote-branch)
2: [https://git-scm.com/docs/git-revert](https://git-scm.com/docs/git-revert)

(完)