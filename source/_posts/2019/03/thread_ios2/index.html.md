---
layout: post
title: 线程基础与 iOS 中的多线程 (二)
date: 2019-03-09
tags: ["NSCondition","NSConditionLock","日志","signal","wait","函数可重入","前端知识","可重入","客户端开发知识","条件变量","编译器优化","计算机基础知识","读写锁"]
categories:
- 计算机
---

![多线程](multy_thread.png "多线程")

接着 [线程基础与 iOS 中的多线程 (一)](https://www.xiaobotalk.com/2019/03/thread_ios1/) ，继续记录多线程相关的知识点。

## NSCondition 和 NSConditinLock

iOS 中 NSLock NSCondition NSConditinLock NSRecursiveLock 都遵守了 NSLocking 协议，其中 NSCondition 和 NSConditinLock 是比较令人迷惑的两个锁；

其实 NSCondition 相比 NSLock 就多了两个方法 wait() 和 signal()，这样这个锁就可以用 wait 来等待某个条件变量来阻塞线程，阻塞后 (执行 wait 后)，其他线程就能获得该锁，直到发出 signal 信号后且条件满足后，wait 的线程重新获得锁，开始后续执行，摘抄一段 stackOverFlow 的代码来理解一下

    // --- MyTestClass.h File --- //
    @interface MyTestClass

    - (void)startTest;

    @end

    // --- MyTestClass.m File --- //
    @implementation MyTestClass
    {
        NSCondition *_myCondition;
        BOOL _someCheckIsTrue;
    }

    - (id)init
    {
        self = [super init];
        if (self) 
        {
            _someCheckIsTrue = NO;
            _myCondition = [[NSCondition alloc] init];
        }
        return self;
    }

    #pragma mark Public Methods

    - (void)startTest
    {
        [self performSelectorInBackground:@selector(_method1) withObject:nil];

        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            sleep(5);
            [self performSelectorInBackground:@selector(_method2) withObject:nil];
        });
    }

    #pragma mark Private Methods

    - (void)_method1
    {
        NSLog(@"STARTING METHOD 1");

        NSLog(@"WILL LOCK METHOD 1");
        [_myCondition lock];
        NSLog(@"DID LOCK METHOD 1");

        while (!_someCheckIsTrue)
        {
            NSLog(@"WILL WAIT METHOD 1");
            [_myCondition wait];
            NSLog(@"DID WAIT METHOD 1");
        }

        NSLog(@"WILL UNLOCK METHOD 1");
        [_myCondition unlock];
        NSLog(@"DID UNLOCK METHOD 1");

        NSLog(@"ENDING METHOD 1");
    }

    - (void)_method2
    {
        NSLog(@"STARTING METHOD 2");

        NSLog(@"WILL LOCK METHOD 2");
        [_myCondition lock];
        NSLog(@"DID LOCK METHOD 2");

        _someCheckIsTrue = YES;

        NSLog(@"WILL SIGNAL METHOD 2");
        [_myCondition signal];
        NSLog(@"DID SIGNAL METHOD 2");

        NSLog(@"WILL UNLOCK METHOD 2");
        [_myCondition unlock];
        NSLog(@"DID UNLOCK METHOD 2");
    }

    @end

    // --- Output --- //
    /*
    2012-11-14 11:01:21.416 MyApp[8375:3907] STARTING METHOD 1
    2012-11-14 11:01:21.418 MyApp[8375:3907] WILL LOCK METHOD 1
    2012-11-14 11:01:21.419 MyApp[8375:3907] DID LOCK METHOD 1
    2012-11-14 11:01:21.421 MyApp[8375:3907] WILL WAIT METHOD 1
    2012-11-14 11:01:26.418 MyApp[8375:4807] STARTING METHOD 2
    2012-11-14 11:01:26.419 MyApp[8375:4807] WILL LOCK METHOD 2
    2012-11-14 11:01:26.419 MyApp[8375:4807] DID LOCK METHOD 2
    2012-11-14 11:01:26.420 MyApp[8375:4807] WILL SIGNAL METHOD 2
    2012-11-14 11:01:26.420 MyApp[8375:4807] DID SIGNAL METHOD 2
    2012-11-14 11:01:26.421 MyApp[8375:4807] WILL UNLOCK METHOD 2
    2012-11-14 11:01:26.421 MyApp[8375:3907] DID WAIT METHOD 1
    2012-11-14 11:01:26.421 MyApp[8375:4807] DID UNLOCK METHOD 2
    2012-11-14 11:01:26.422 MyApp[8375:3907] WILL UNLOCK METHOD 1
    2012-11-14 11:01:26.423 MyApp[8375:3907] DID UNLOCK METHOD 1
    2012-11-14 11:01:26.423 MyApp[8375:3907] ENDING METHOD 1
    */

NSConditionLock 可以定义一个指定条件的互斥锁，用于线程之间的互斥与同步。这里的条件并不是 bool 表达式中的条件，而是一个特定的 int 值。

    // lockWhenCondition ：用于condition等于特定值的时候加锁，会阻塞当前线程。
    // unlockWithCondition：指定条件时解锁，每次解锁会导致内部的condition值改变为指定的值，同时唤醒其它阻塞的线程检测这里的condition是否满足条件，因此NSConditionLock相对于NSCondition效率更低。

    // 示例代码

    [NSThread detachNewThreadSelector:@selector(createConsumer)
                                       toTarget:sample
                                     withObject:nil];

    [NSThread detachNewThreadSelector:@selector(createProducter)
                                      toTarget:sample
                                    withObject:nil];

    - (void)createConsumer
    {
        NSLog(@"createConsumer");
        while (YES) {
            NSLog(@"createConsumer before lock");
            [self.conditionLock lockWhenCondition:3];
            NSLog(@"createConsumer after lock");
            if([self.products count] > 0)
                [self.products removeObjectAtIndex:0];
            NSLog(@"consume a product,left %@ products", @([self.products count]));
            [self.conditionLock unlockWithCondition:[self.products count]==0?0:10];
            NSLog(@"createConsumer after unlock");
        }
    }

    - (void)createProducter
    {
        NSLog(@"createProducter");
        while (YES) {
            NSLog(@"createProducter before lock");
            [self.conditionLock lock];
            NSLog(@"createProducter after lock");
            [self.products addObject:[[NSObject alloc] init]];
            NSLog(@"produce a product,left %@ products", @([self.products count]));
            [self.conditionLock unlockWithCondition:[self.products count]];
            NSLog(@"createProducter after unlock");
        }
    }

    // 控制台输出打印如下:
    [22232:1943872] createConsumer
    [22232:1943874] createProducter
    [22232:1943872] createConsumer before lock
    [22232:1943874] createProducter before lock
    [22232:1943874] createProducter after lock
    [22232:1943874] produce a product,left 1 products
    [22232:1943874] createProducter after unlock
    [22232:1943874] createProducter before lock
    [22232:1943874] createProducter after lock
    [22232:1943874] produce a product,left 2 products
    [22232:1943874] createProducter after unlock
    [22232:1943874] createProducter before lock
    [22232:1943874] createProducter after lock
    [22232:1943874] produce a product,left 3 products
    [22232:1943874] createProducter after unlock
    [22232:1943872] createConsumer after lock
    [22232:1943874] createProducter before lock
    [22232:1943872] consume a product,left 2 products
    [22232:1943872] createConsumer after unlock
    [22232:1943874] createProducter after lock
    [22232:1943872] createConsumer before lock
    [22232:1943874] produce a product,left 3 products
    [22232:1943874] createProducter after unlock
    [22232:1943872] createConsumer after lock

当生产者生产个数达 3 个时释放锁：produce a product,left 3 products，此后锁被消费者获取，开始消费。

## 读写锁

* * *

读写锁是一种特定场合的锁，多个线程可以同时读取某个资源，但只要有线程开始写入的时候，就必须用同步的手段保护资源，此时其他线程不能读，也不能写。所以读写锁可以避免这种问题，读写锁有 3 种状态：

*   自由状态：此时线程可以以共享的方式获取，也可以以独占的方式获取
*   共享状态：此时说明资源处于共享状态(可能是多线程读取资源中)，所以其他线程可以以共享的状态获取锁，但如果想以独占的方式获取锁，则需要等待。等待共享状态变为自由态，才可以以独占方式获取
*   独占状态：此时很有可能资源被某个线程写入，所以其他线程不管是以共享还是独占方式获取锁都需要等待。

iOS 开发中，可以使用 <pthread.h> 中的 pthread_rwlock_t 来实现读写锁

    pthread_rwlock_t rwlock = PTHREAD_RWLOCK_INITIALIZER; // 初始化
    pthread_rwlock_wrlock(&lock); // 读模式
    pthread_rwlock_rdlock(&lock); // 写模式
    pthread_rwlock_unlock(&lock); // 读模式或者写模式的解锁

## 条件变量

* * *

条件变量的作用类似于栅栏，线程对条件变量的操作有两种，等待和唤醒；一个条件变量可以被多个线程等待，也就是条件变量能够让多个线程一起等待某事件；当某个线程唤醒条件变量后，所有等待的线程可以一起恢复执行。

## 可重入与线程安全

* * *

可重入是线程安全的强力保障，一个可重入函数在多线程环境下可以放心调用。

一个函数被重入，表示函数没有执行完成，由于外部因素或者内部调用，又一次进入函数执行。一个函数被重入，有两种情况：

*   多个线程同时执行这个函数
*   函数自身调用自身

例如下面的这个函数就是一个可重入函数：

    int add (int a, int b)
    {
            return a + b;
    }

一个函数可重入，则具有以下几个特点：

*   不使用任何静态或全局的非 const 变量
*   不返回任何静态或全局的非 const 变量的指针
*   仅依赖于调用方提供的参数
*   不调用任何不可重入的函数
*   不依赖任何单个资源的锁

## 编译器过度优化

* * *

尽管我们会用 lock 和 unlock 来保护资源，但是编译器往往会对代码做出一些优化，调整寄存器读入和回写的代码顺序，从而导致了 CPU 的乱序执行，破坏锁的保护机制。目前还没遇到具体问题，再次先做个记录。

(完)