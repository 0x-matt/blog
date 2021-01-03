---
layout: post
title: Mac 外接磁盘写入问题
date: 2018-11-19
tags: ["exFat","Mac","Mac 高效使用指南","macbook","mbp","NTFS","Paragon","Tuxera","windows","付费","免费","希捷","技巧","电脑","磁盘读写","苹果","苹果电脑","高效"]
categories:
- 实用软件
---

![Mac 外接磁盘写入](macSkills_01.png)

 Mac 可以从一个通过 USB 连接的磁盘 ( U 盘或者移动硬盘这里统称为外接磁盘)里拷贝文件到 Mac 电脑里，但是反过来却不可以，也就是从 Mac 里拷贝文件到外接磁盘。因为大部分的磁盘初始格式都是 NTFS ，Mac 默认未开启 NTFS 格式的文件写入。本篇将介绍 Mac 读写外接磁盘的5种方案，相信总有一种方案适合你。

NTFS 是 New Technology File System 的缩写，简单来讲就是一种文件系统。同样 exFAT 也是一种文件系统，如果你听说过早期 Windows 的 FAT32 文件系统，那么你可以把 exFAT 理解为 FAT32 的 64 位版本。

## 方案1. 希捷 (Seagate) 移动硬盘官方特供的写入 NTFS 的软件

* * *

![Mac 外接磁盘写入](macSkills_08.png)

如果你刚好买的是希捷 (Seagate) 的移动硬盘（如上图，图片仅供参考，理论上只要是希捷品牌的硬盘都可以），Mac 写入的问题就很好解决了，希捷官方提供了磁盘读写软件的免费下载，该软件就是大名鼎鼎的 Paragon NTFS for Mac ，没错正常情况下，该软件是收费的，而且不便宜，现在你可以在希捷的官方网站下载到，但是只是针对希捷的硬盘有效。[点我到希捷官网下载免费的 Paragon NTFS for Mac。](https://www.seagate.com/cn/zh/support/downloads/item/ntfs-driver-for-mac-os-master-dl/)

软件的安装和使用都很简单，先安装软件：

![](macSkills_06.png)

安装后一般都要重启电脑，重启后基本就可以使用了，个别第一次安装的可能勾选了 `只读模式安装` 勾选去掉就可以了。

![Mac 外接磁盘写入](macSkills_07.png)

## 方案2. 新硬盘可以考虑格式化成 exFAT 格式

* * *

如果是新入手的硬盘 (老硬盘，里边存的文件太多，又不好备份，格式化需要先备份里边的数据)，或者硬盘里文件不多，可以考虑格式化成 exFAT ，exFAT 格式的硬盘 Windows 和 Mac 都原生支持读写，如果你需要经常在 Mac 和 Windows 之间切换，可以考虑将硬盘格式化为 exFAT 格式。具体 exFAT 是什么，以及它的历史，可以去[维基百科 exFAT](https://zh.wikipedia.org/wiki/ExFAT)，毕竟本文重点不是介绍各种文件系统的，让我们专注在解决 MAC 读写磁盘上；总之 exFAT 也是一种文件系统， MAC 和 Windows 都支持原生读写。

`格式化会抹掉硬盘里所有的数据，如需备份，请自行做好备份。`

#### Mac 系统自带的磁盘工具软件，就可以进行格式化了：

![Mac 外接磁盘写入](macSkills_09.png)

打开磁盘工具软件

![Mac 外接磁盘写入](macSkills_10.png)

1.  选择你要格式化的磁盘
2.  点击'擦掉'
3.  选择 ExFAT 格式 点击擦掉

#### Windows 资源管理器或我的电脑里面选择格式化：

![Mac 外接磁盘写入](macSkills_11.png)

当然很多人可能不是很乐意格式化为 exFAT，因为这种格式占空间，稳定性比较 NTFS 稍微差了点；使用时，如果没有先在电脑上点弹出，就直接拔掉 USB，可能还会在硬盘里生成一些垃圾文件；不过总体问题不是很大，发生数据丢失也是小概率事件。如果你不是特别有电脑使用洁癖的，格式化成 exFAT 是最省心的办法。

## 方案3. 付费的三方软件

* * *

1: [Paragon NTFS for Mac ](http://www.dpbolvw.net/click-3607085-12975586?sid=ct236055)，已经阅读到这里了，看来你的硬盘很可能不是希捷品牌，那么就要付费使用 Paragon NTFS for Mac 了，虽然可以免费试用几天，但完全享用需要 130 多 RMB (具体价格可能随时间推移而发生变动，最好自己去官网看一下)，还好不是订阅制，购买后便可终身使用，如果官方不再做什么变更的话。

2: [Tuxera NTFS for Mac](http://www.tuxera.com/products/tuxera-ntfs-for-mac/)， 也是一款具有相同功能的付费软件，但是它比 Paragon NTFS for Mac 还要贵上不少，大概 200 RMB (价格说明同上) 左右。

付费软件唯一让人不太乐意选择的原因就是它的 '付费' 属性，很多人可能没有购买软件的习惯，但是付费确实是最省心，最安全的。毕竟 "免费的才是最贵的"。

## 方案4. 免费的三方软件

* * *

[FUSE for macOS](https://github.com/osxfuse/osxfuse/releases) 是一款免费的三方软件，同样是为了解决 Mac 读写 NTFS 问题；但是相比上边两款付费软件，使用起来要麻烦一些，需要做一些额外的工作，手动虽然繁琐一点，但是比较安全；如果想更进一步如果想让 Mac 自动挂载 NTFS 分区，还要替换 Mac 系统内置的 NTFS 工具，则需要承担安全性的风险。

首先下载并且安装 [FUSE for macOS](https://github.com/osxfuse/osxfuse/releases) 。

#### 1. 手动使用命令行 (Terminal) 软件挂载 NTFS 分区:

这种方式相对繁琐，需要操作命令行，不过也就几行命令而已；先来认识一下 Terminal： 应用程序 --> 实用工具 --> 终端。

![Mac 外接磁盘写入](macSkills_12.png)

程序员的话可能习惯用 iTerm2 来代替 Terminal；虽然看起来步骤繁琐，但是请坚持阅读下去，很有可能你仅仅需要执行后边几条命令，并且除去第一次操作会繁琐一些，以后只需要两条命令。

**1.1: 安装 xcode-select ，在终端执行如下指令，如果安装请略过(执行一次，下次启动不用安装):**

    xcode-select --install

安装会弹出对话框

![Mac 外接磁盘写入](macSkills_02.png)

点击"Install"

**1.2: 安装 Homebrew ，在终端执行如下指令，如果安装请略过 (执行一次，下次启动不用安装):**

    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

**1.3: 使用刚才安装的 Homebrew 安装 ntfs-3g，如果安装请略过(执行一次，下次启动不用安装):**

    brew install ntfs-3g

**1.4: 到此，你可以开始手动管理 NTFS 的读写模式了，先创建一个 NTFS 分区文件夹，这个操作只需要执行一次:**

    sudo mkdir /Volumes/NTFS

**1.5: 当 NTFS 磁盘连接到电脑后，通过下面命令查看磁盘分区列表**

    diskutil list

![Mac 外接磁盘写入](macSkills_03.png)

可以看到 Windows_NTFS 字样，它在 /dev/disk2 下边，在最后一列 IDENTIFIER 可以看到 Windows_NTFS 被定义为: disk2s1，已经被 Mac 自动装载。

**1.6: 先解除 disk2s1 自动装载**

    sudo umount /dev/disk2s1

执行完这行命令，桌面上的外接磁盘图案会短暂的消失，别怕，很快会回来的。

![Mac 外接磁盘写入](macSkills_05.png)

**1.7: 然后用 我们在 1.4 步骤创建的 NTFS 分区代替 /dev/disk2s1**

    sudo /usr/local/bin/ntfs-3g /dev/disk2s1 /Volumes/NTFS -olocal -oallow_other

Done!!! 这时候你可以再看看桌面上的外接磁盘图案，你会发现图标发生了神器的变化:

![Mac 外接磁盘写入](macSkills_04.png)

打开它，开始写入吧！ 其实也不是很复杂，除去第一次安装稍显繁琐，以后我们只需要重复 `1.6` 和 `1.7` 步骤即可。当然也有可能你讨厌命令行，已经开始倾向去买付费软件了，也开始感受到了 "免费的才是最贵的"。

**希望上述各种方案你能采纳其中之一，因为后边的方案都缺乏安全性，也不建议去尝试，没必要为了写入磁盘功能承担太大的风险；**

#### 2. 让 Mac 自动挂载 NTFS 分区 (折腾有风险，挂载需谨慎):

接着 1.7 步骤，我们已经结合 FUSE for macOS 和 终端完成了手动控制写入 NTFS 了，但你可能觉得手动还是繁琐，想自动完成，也是可以的，不过想要让 Mac 自动挂载可写入的 NTFS，需要先禁用 '[系统完整性保护](https://www.howtogeek.com/230424/how-to-disable-system-integrity-protection-on-a-mac-and-why-you-shouldnt/)' ，再次申明: 这样做有风险，开始操作前你要考虑清楚。

**2.1: 重启 Mac 按住 Command + R ，让 Mac 进入 recovery 模式，启动终端执行命令:**

    csrutil disable

命令会关闭系统完整性保护。

**2.2: 然后再次正常启动 Mac，再次打开终端，依次执行命令：**

    sudo mv /sbin/mount_ntfs /sbin/mount_ntfs.original  
    sudo ln -s /usr/local/sbin/mount_ntfs /sbin/mount_ntfs

命令会使用 ntfs-3g 工具替换 Mac 中的 NTFS 安装工具。

**2.3: 然后，再次重启进入 recovery ，恢复系统完性保护。**

    csrutil enable

最后再次正常重启 Mac，Done!!! 不同的系统可能会发生失败。

**最后的最后如果撤销上述一系列操作，先重复 1.8.1 步骤关闭系统完整性保护，再命令行依次执行下边命令。**

    sudo rm /sbin/mount_ntfs

    sudo mv /sbin/mount_ntfs.original /sbin/mount_ntfs

    brew uninstall ntfs-3g

## 方案5. 实验性方案 /etc/fstab 修改，不建议

* * *

可能你还听说过，通过 nano /etc/fstab 的方式，实现 NTFS 的读写，但是这种方式安全风险更高，也不一定能成功，很有可能升级完系统后又失效了，普通用户大可不要去尝试。笔者自己也没有尝试过这种方式，就不做介绍了。

## 延伸阅读

[ NTFS 维基百科](https://zh.wikipedia.org/zh-hans/NTFS)  

[ exFAT 维基百科](https://zh.wikipedia.org/wiki/ExFAT)  

**本文同时发布与[我的 Gitbook 文集中](https://www.xiaobotalk.cn/PartTwo/two_outer_disk.html)**

(全文完)