---
layout: post
title: addChildViewController 与 addSubview 调用顺序
date: 2019-05-05
tags: ["addChildViewController","addSubview","beginAppearanceTransition","endAppearanceTransition","日志","view","viewDidAppear","viewDidAppear 两次调用","viewWillAppear","willMoveToParentViewController","前端知识","客户端开发知识","生命周期","视图"]
categories:
- 计算机
---

## Container View Controller

* * *

iOS 实际开发中，为了方便管理视图，我们经常会把一个 VC (ViewController) 作为容器 VC，其他子视图的 VC 作为容器 VC 的子 VC。同时 view 也作为容器 View 的子 view (subview)。苹果官方文档称为 Container View Controller。需要注意的是，在调用 addChildViewController 和 addSubview 时需要注意调用顺序，官方文档建议：

1、添加子视图的时候，代码调用顺序：

    - (void) displayContentController: (UIViewController*) content {
        // 1. 添加 子vc，调用 addChildViewController
       [self addChildViewController:content];
        // 2. 设置 子视图的 frame
       content.view.frame = [self frameForContentController];
        // 3. 添加子视图， 调用 addSubview
       [self.view addSubview:self.currentClientView];
        // 4. 调用 didMoveToParentViewController
       [content didMoveToParentViewController:self];
    }

需要注意的是，添加的时候，不需要显式的调用：willMoveToParentViewController，因为 addChildViewController 会帮你调用该方法。

2、移除子视图的时候，代码调用顺序：

    - (void) hideContentController: (UIViewController*) content {
       [content willMoveToParentViewController:nil];
       [content.view removeFromSuperview];
       [content removeFromParentViewController];
    }

与添加的时候相反，这里不需要显示的调用 didMoveToParentViewController，removeFromParentViewController 内部会调用 didMoveToParentViewController ，并且把传入 nil 参数。

## addChildViewController 的重要性

* * *

在向 container view controller 中添加 subview 的时候，往往会同时添加子 vc 关系。除了保证响应链的正常传递外，子 vc 的视图生命周期函数也会跟随父 vc 被触发。例如我们在 viewDidLoad 函数中添加了子 vc 和子 view：

    - (void)viewDidLoad {
        [super viewDidLoad];

        TTViewController *ttvc = [TTViewController new];
        [self addChildViewController:ttvc];
        [self.view addSubview:ttvc.view];
        [ttvc didMoveToParentViewController:self];
    }

虽然在 viewDidLoad 中添加了子 view (此时容器视图的 viewwillAppear 和 viewDidAppear 都没有被触发)，但是随后容器 vc 的生命周期方法(viewWillAppear)被触发的时候，容器 vc 会同时遍历自己的 childVC 来同步 childVC 的视图生命周期方法，这一点可以通过观察调用栈看到：

![viewDidAppear](viewDidAppearStack.png "viewDidAppear")

(viewWillAppear 方法的调用栈类似上图 ) 所以 addChildViewController 后，上图的 `[__NSSingleObjectArrayI enumerateObjectsWithOptions:usingBlock:]` 才能产生有效调用。

通过上边的调用栈，大概能看出，视图的 viewDidAppear 的调用，底层是来自 GraphicsServices 库的 GSEventRunModal 的调用，GSEventRunModal 的调用触发 Runloop 回调，来通知视图处理生命周期。所以理论上 viewWillAppear 和 viewDidAppear 的调用都是在不同的 Runloop 上进行的。

## viewDidAppear 两次调用场景

* * *

实际开发中，当我们通过 addChildViewController 与 addSubview 来添加子视图的时候，存在 viewDidAppear 两次被调用的情况。

产生两次 viewDidAppear 调用的原因，一次是来自 addChildViewController 后，视图跟随容器视图的生命周期方法被调用，另一次触发是来自 addSubview 的调用，并且时机是间于容器视图的 viewWillAppear 和 viewDidAppear 之间。

同时代码顺序是先调用 addSubview 后调用 addChildViewController (猜测 addSubview 内部做了是否触发 viewWillAppear...等方法的判断)。

所以下列代码基本上都会触发子 vc 的 viewDidAppear 两次调用：

    - (void)viewDidLoad {
        [super viewDidLoad];

        // 1、dispatch 到下次 runloop
        dispatch_async(dispatch_get_main_queue(), ^{
            [self addSubViewController];
        });

        // 2、perform 到下一次 runloop 
        [self performSelectorOnMainThread:@selector(addSubViewController) withObject:nil waitUntilDone:NO];

        // 3、延迟 0.1 ~ 0.4 秒之间执行 
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.26 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            [self addSubViewController]; 
        });
    }

    - (void)addSubViewController {
         TTViewController *ttvc = [TTViewController new];
         // 注意这里先调用的是 addSubview
         [self.view addSubview:ttvc.view];
         [self addChildViewController:ttvc];
         [ttvc didMoveToParentViewController:self];
    }

上述代码经过测试，都会触发 viewDidAppear 的两次调用，其中第一次是 addSubview 触发的 （在容器视图的 viewDidAppear 前调用），第二次是容器视图的 viewDidAppear 触发同步给了 childVC 触发的 (上边已经分析过)。

实际开发中，场景更多的可能是网络请求回调的时间刚好是 0.1~0.4 s 之间。而此时刚好执行了 addSubview 操作。

要解决 viewDidAppear 正常的被调用一次，只需要改变调用顺序，即文章开头建议的先 addChildViewController 后调用 addSubvew。

## addSubview 与 viewWillAppear / viewDidAppear

* * *

为了研究 addSubview 的内部逻辑，我们先抛开 addChildViewController 方法不谈。可以断定的是，addSubview 方法内部一定做了是否触发触发 viewWillAppear 的判断。来看下列代码：

    - (void)viewDidLoad {
        [super viewDidLoad];

        TTViewController *ttvc = [TTViewController new];
        [self.view addSubview:ttvc.view];
    }

上面代码不会触发子视图的生命周期方法，原因是此时父视图 (self.view) 没有被渲染到屏幕上，所以子视图也不会立刻显示，addSubview 也只能放弃视图的生命周期方法调用。同样的，在 viewWillAppear 中调用，会得到相同的结果。

在 viewDidAppear 视图 addSubview :

    - (void)viewDidAppear:(BOOL)animated {
        [super viewDidAppear:animated];
        TTViewController *ttvc = [TTViewController new];
        [self.view addSubview:ttvc.view];
    }

此时，子视图 viewWillAppear / viewDidAppear 都会被正常调用， viewWillAppear 断点调用栈如下：

![viewWillAppear](addSubviewWillAppear.png "viewWillAppear")

可以看到 addSubview 内部会调用 vc 的`viewWillMoveToWindow` ，从而立刻触发 viewWillAppear 方法。

在看 viewDidAppear 调用栈：

![viewDidAppear](addSubviewDidAppear.png "viewDidAppear")

可以看到 `UIViewController _executeAfterAppearanceBlock` 调用后，调用了 vc 的内部方法 `viewDidMoveToWindow...`，然后触发 viewDidAppear。底层是一个 observer 的 runloop 回调，所以只能猜测，addSubview 方法注册了 observer 的 runloop 事件，等待 Core Animation 视图渲染完成。

所以理论上，如果我们在父视图生命周期执行完成后(viewDidAppear 之后) addSubview ，那么 addSubview 会自己处理生命周期方法，但是尽管如此，我们还是要在调用 addSubview 前加上 addChildViewController 调用，来保证视图的其他处理正常(例如 viewWillDisAppear 等跟随父 vc 的触发)。

通过分析我们也大概能分析出 addSubview 处理的逻辑：

* * *

1.  addSubview 的时候，父视图的生命周期函数还没有触发，那么此时自己肯定不能被显示在 window 上，所以不处理任何视图周期函数；此时视图生命周期函数的正常触发，就全靠 addChildViewController 来维持了。</p>
2.  addSubview 的时候，父视图的生命周期函数已经执行完成(viewDidAppear 已经执行完)，自己可以立刻显示到 window 上，此时 addSubview 内部会立刻触发 viewWillAppear，同时注册 source0 的 runloop 事件，等待 GraphicsServices 的视图渲染完成的回调，触发 viewDidAppear。

3.  addSubview 的时候，父视图的生命周期函数间于 viewWillAppear 和 viewDidAppear 之间，此时立刻触发 viewWillAppear ，viewDidAppear 则不用处理，跟随父 vc 的调用触发，所以先调用 addChildViewController 的重要性就在这里。

## 手动控制子视图生命周期

* * *

<p>如果想自己手动通过代码严格控制视图生命周期，可以通过:

    - (BOOL) shouldAutomaticallyForwardAppearanceMethods {
        return NO;
    }

来限制视图生命周期方法的自动触发，但同时要自己管理生命周期：

    -(void) viewWillAppear:(BOOL)animated {
        [self.child beginAppearanceTransition: YES animated: animated];
    }

    -(void) viewDidAppear:(BOOL)animated {
        [self.child endAppearanceTransition];
    }

    -(void) viewWillDisappear:(BOOL)animated {
        [self.child beginAppearanceTransition: NO animated: animated];
    }

    -(void) viewDidDisappear:(BOOL)animated {
        [self.child endAppearanceTransition];
    }

## 参考文档

* * *

1、[苹果开发文档 addchildviewcontroller](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621394-addchildviewcontroller)
2、[苹果开发文档 beginappearancetransition](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621387-beginappearancetransition?language=objc)
3、[苹果开发文档 ImplementingaContainerViewController](https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/ImplementingaContainerViewController.html)

(完)