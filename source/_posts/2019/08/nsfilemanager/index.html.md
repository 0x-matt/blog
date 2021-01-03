---
layout: post
title: NSFileManager
date: 2019-08-17
tags: ["defaultManager","Doucment","fileExistsAtPath","iOS","NSFileManager","NSFileManagerDelegate","日志","Tmp","withIntermediateDirectories","客户端开发知识","计算机基础知识"]
categories:
- 计算机
---

![fileManager](filemanager.png "fileManager")

文件系统的管理非常复杂，是每个操作系统的基础功能。NSFileManager 是 Foundation 的高级 API，专门用来处理文件系统。它抽象了 Unix 和 Finder 的内部构成，集成了创建、删除、拷贝、移动等文件操作功能。虽然文件系统很复杂，但是 NSFileManager 的使用却很简单，拿来即用。

## 一、使用 [NSFileManager defaultManager]

* * *

在 iOS 开发的大部分场景，只需要使用 `[NSFileManager defaultManager]` 来获取一个文件管理的单例对象即可。同时也建议尽量这样做，因为 `[NSFileManager defaultManager]` 来操作文件，及时在多线程环境中也是安全的，省去开发者自己关心线程安全问题。

## 二、NSFileManagerDelegate

* * *

当我们自己通过`alloc init` 的方式创建 NSFileManager 实例，此时可能需要实现 `NSFileManagerDelegate` 协议，`NSFileManager` 可以设置一个遵循了 `NSFileManagerDelegate` 的 `delegate`，这个 delegate 定义了下列方法：

    -fileManager:shouldMoveItemAtURL:toURL:
    -fileManager:shouldCopyItemAtURL:toURL:
    -fileManager:shouldRemoveItemAtURL:
    -fileManager:shouldLinkItemAtURL:toURL:

通过实现协议方法，我们可以拦截文件的移动、删除、拷贝、链接过程，来判断是否阻止这个过程。

## 三、常用操作

* * *

**1. 判断文件是否存在**

    NSFileManager *fileManager = [NSFileManager defaultManager];
    NSString *documentsPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject];
    NSString *filePath = [documentsPath stringByAppendingPathComponent:@"file.txt"];
    BOOL fileExists = [fileManager fileExistsAtPath:filePath];

或者使用 fileExistsAtPath:isDirectory: ，同时判断目标路径是文件还是文件夹:

    BOOL isDirectory;
    BOOL exist = [fileManager fileExistsAtPath:directoryPath isDirectory:&isDirectory];

**2、创建目录**

    NSString *documentsPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject];
    NSString *sources = [documentsPath stringByAppendingPathComponent:@"sources"];
    if (![fileManager fileExistsAtPath:sources]) {
        [fileManager createDirectoryAtPath:sources withIntermediateDirectories:NO attributes:nil error:nil];
    };

withIntermediateDirectories 用来设定是否需要创建中间路径，看下列代码：

    // 想创建的路径为 /Doucment/sources/images/

    NSString *sources = [documentsPath stringByAppendingPathComponent:@"sources/images"];
    if (![fileManager fileExistsAtPath:sources]) {
        BOOL bl = [fileManager createDirectoryAtPath:sources withIntermediateDirectories:NO attributes:nil error:nil];
        if (bl) {
            NSLog(@"创建成功");
        } else {
            NSLog(@"创建失败");
        }
    };

    // 输出：创建失败

上边的代码，在没有 `sources/` 这个文件路径的情况下会创建失败，设置 `withIntermediateDirectories:NO` ，遇到中间路径没有的情况，就会创建失败，如果已经存在 `sources/` 目录，那么上述代码就能成功的在 `sources/` 下创建 `images/` 目录。

相反使用 `withIntermediateDirectories:YES` 则会一路创建下去，包括中间文件夹：

    // 想创建的路径为 /Doucment/assets/images/

    NSString *sources = [documentsPath stringByAppendingPathComponent:@"assets/images"];
    if (![fileManager fileExistsAtPath:sources]) {
        BOOL bl = [fileManager createDirectoryAtPath:sources withIntermediateDirectories:YES attributes:nil error:nil];
        if (bl) {
            NSLog(@"创建成功");
        } else {
            NSLog(@"创建失败");
        }
    };

    // 输出：创建成功，成功创建出：/Doucment/assets/images/

**3、移动文件或目录**

    NSFileManager *fileManager = [NSFileManager defaultManager];
    BOOL bl = [fileManager removeItemAtPath:filePath1 error:nil];

**4、移动文件或目录**

    NSFileManager *fileManager = [NSFileManager defaultManager];
    BOOL bl = [fileManager moveItemAtPath:filePath1 toPath:filePath2 error:&fileError];

**5、拷贝文件或目录**

    NSFileManager *fileManager = [NSFileManager defaultManager];
    BOOL copyBool = [fileManager copyItemAtPath:filePath1 toPath:filePath2 error:©Error];

**6、遍历目录文件**

    NSFileManager *fileManager = [NSFileManager defaultManager];
    NSURL *bundleURL = [[NSBundle mainBundle] bundleURL];
    NSArray *contents = [fileManager contentsOfDirectoryAtURL:bundleURL
                                   includingPropertiesForKeys:@[]
                                                      options:NSDirectoryEnumerationSkipsHiddenFiles
                                                        error:nil];

    NSPredicate *predicate = [NSPredicate predicateWithFormat:@"pathExtension == 'png'"];
    for (NSURL *fileURL in [contents filteredArrayUsingPredicate:predicate]) {
        // 在目录中枚举 .png 文件
    }

**7、递归遍历目录文件**

    NSFileManager *fileManager = [NSFileManager defaultManager];
    NSURL *bundleURL = [[NSBundle mainBundle] bundleURL];
    NSDirectoryEnumerator *enumerator = [fileManager enumeratorAtURL:bundleURL
                                          includingPropertiesForKeys:@[NSURLNameKey, NSURLIsDirectoryKey]
                                                             options:NSDirectoryEnumerationSkipsHiddenFiles
                                                        errorHandler:^BOOL(NSURL *url, NSError *error)
    {
        if (error) {
            NSLog(@"[Error] %@ (%@)", error, url);
            return NO;
        }

        return YES;
    }];

    NSMutableArray *mutableFileURLs = [NSMutableArray array];
    for (NSURL *fileURL in enumerator) {
        NSString *filename;
        [fileURL getResourceValue:&filename forKey:NSURLNameKey error:nil];

        NSNumber *isDirectory;
        [fileURL getResourceValue:&isDirectory forKey:NSURLIsDirectoryKey error:nil];

        // Skip directories with '_' prefix, for example
        if ([filename hasPrefix:@"_"] && [isDirectory boolValue]) {
            [enumerator skipDescendants];
            continue;
        }

        if (![isDirectory boolValue]) {
            [mutableFileURLs addObject:fileURL];
        }
    }

## 四、关于 path 路径

* * *

NSFileManager 在创建、移动、拷贝、删除文件(夹)的时候，操作的都是末尾 path，举例来说 :

    // 1: 文件路径 末尾 path ：logo.png
    path1 = "/Users/zhangsan/..../Document/Sources/images/logo.png"   
    // 2: 文件夹路径 末尾 path : images
    path2 = "/Users/zhangsan/..../Document/Sources/images/"  

NSFileManager 本身认为**一起皆文件**，文件夹不过是特殊的文件。如果我们使用 `copyItemAtPath:toPath:error:` 拷贝 path2 到其他路径，由于 path2 末尾是 `images`，所以拷贝的是整个文件夹。如果是 path1 ，拷贝的就是文件 `logo.png`。

另外，拷贝或者移动的时候，需要确保一下两点，否则操作失败：

> 1、如果目标文件路径不存在，则操作失败。
>   2、如果目标文件或者文件夹已经存在，则操作失败。

举例来说，我们把想把 `/Users/zhangsan/..../Document/Sources/images/` 拷贝到 `/Users/zhangsan/..../Tmp/Sources/` 下，那么需要确保：

> 1、Sources 路径已经存在
>   2、Sources 下边不能存在 images 文件夹或者文件

所以我们一般在操作前对目标路径进行两个操作：**1、检查目录路径是否存在，不存在，就创建， 2、进行末尾路径删除** 来保证，拷贝或者移动能正确进行：

    // 1、检查创建路径
    if (![fileManager fileExistsAtPath:unzipDirectoryPath]) {
        [fileManager createDirectoryAtPath:path withIntermediateDirectories:YES attributes:nil error:nil];
    }
    // 2、删除末位路径，方便做文件夹拷贝/移动
    [fileManager removeItemAtPath:path error:nil];

这样做，可以减少关心每次 `copyItemAtPath:` 或者 `moveItemAtPath` 调用的的成功和失败结果以及 error，因为这些方法都返回 BOOL 值。

## 参考文章:

* * *

1.  [https://developer.apple.com/documentation/foundation/nsfilemanager](https://developer.apple.com/documentation/foundation/nsfilemanager)
2.  [https://nshipster.cn/nsfilemanager/](https://nshipster.cn/nsfilemanager/)

(全文完)