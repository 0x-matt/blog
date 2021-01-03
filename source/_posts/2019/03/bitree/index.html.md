---
layout: post
title: 二叉树基础知识
date: 2019-03-29
tags: ["日志","中序遍历","二叉树","前序遍历","后序遍历","完全二叉树","满二叉树","计算机基础知识"]
categories:
- 计算机
---

![二叉树](treeBlog.png "二叉树")

本文主要记录二叉树基本知识点。二叉树是比较常用的一种数据结构，结构特点是一个根节点与两个互不相交的左右节点。比较容易混淆的概念是满二叉树和完全二叉树。

## 满二叉树

* * *

![满二叉树](fullBiTree.png "满二叉树")

满二叉树是是平衡的，左右对称，具备以下特点：

> 1、所有叶子节点都在同一层。
>   2、非叶子节点的度一定是 2 (节点的度表示该节点的子树个数)。
>   3、同样深度的二叉树中，满二叉树的叶子节点最多。

## 完全二叉树

* * *

![满二叉树](entirelyTree.png "完全二叉树")

从概念上来讲，完全二叉树是指，如果对一棵有 n 个节点的二叉树按层次编号(从上到下，从左到右)，如果任意编号 i 的节点与同样深度的满二叉树中编号为 i 的节点位置完全相同，则这棵树为完全二叉树。配合下图，可以更好的理解概念：

所以满二叉树一定是完全二叉树，反过来则不一定是，完全二叉树具备以下特点：

> 1、叶子节点只能出现在最下两层
>   2、如果节点的度为 1，则该节点只能有左子树，不存在只有右子树的情况。

## 二叉树遍历

* * *

遍历一棵二叉树，最长见的是前序遍历、中序遍历、后序遍历。前中后说的是根节点的顺序，前序访问顺序：根 -> 左 -> 右；中序访问顺序：左 -> 根 -> 右；后序访问顺序：左 -> 右 -> 根；

![二叉树](biTree01.png)

> 针对上边二叉树
>   前序遍历：A B D E C F G
>   中序遍历：D B E A F C G
>   后序遍历：D E B F G C A

使用递归的方式遍历二叉树，代码及其简洁：

    // 定义节点
    typedef struct TreeNode
    {
        char *data;
        struct TreeNode *lchild, *rchild;
    } TreeNode, *BiTree;

    // 前序
    void preOrderTree (BiTree T)
    {
        if (T == NULL) {
            return;
        }
        printf("%s", T -> data);
        preOrderTree(T -> lchild);
        preOrderTree(T -> rchild);
    }

    // 中序
    void inOrderTree (BiTree T)
    {
        if (T == NULL) {
            return;
        }
        inOrderTree(T -> lchild);
        printf("%s", T -> data);
        inOrderTree(T -> rchild);
    }

    // 后序
    void postOrderTree (BiTree T)
    {
        if (T == NULL) {
            return;
        }
        postOrderTree(T -> lchild);
        postOrderTree(T -> rchild);
        printf("%s", T -> data);
    }

    // 三种遍历方式，只需要调整代码 printf("%s", T -> data) 的顺序即可。

## 推导遍历结果

* * *

如果我们知道一棵二叉树的**前序遍历和中序遍历**结果，便可推导出这棵二叉树的结构；同样的知道一棵二叉树的**后序遍历和中序遍历**结果，也可推导出这棵二叉树的结构。

例如，已知一棵二叉树的前序遍历序列为：ABCDEF，中序遍历序列为：CBAEDF，那么后序遍历序列是什么呢 ?

1、根据前序遍历 ABCDEF，可以得出树的根节点为 A，那么再用 A 来切割中序遍历序列 CB-A-EDF ('-' 表示分割符，只用来增强可读性)，便可得出下图：

![推导二叉树](tree_log.png)

2、左右两堆节点，继续用相同的方式推导，便可得出最终二叉树：

![推导二叉树](tree_log_res.png)

但要注意的是，已知**前序和后序遍历**，是不能确定一棵二叉树的。

(完)