---
layout: post
title: iOS 响应者链与事件处理
date: 2020-03-29
tags: ["action","Application","first reponder","hitTest","iOS","iPhone","nextResponder","pointInside","日志","quickSort","Responder Chain","Runloop","sendEvent","target","target-action","touchesBegan","UIButton","UIControl","UIEvent","UIResponder","UIWindow","_UISystemGestureGateGestureRecognizer","事件处理","响应者链","客户端开发知识","摇一摇","第一响应者"]
categories:
- 计算机
---

* * *

作者：xiaobo

![first_reponder](touch_iphone_screen.png "first_reponder")

当我们讨论 "iOS 事件处理与响应者链" 的时候，其实我们想要了解的是：**触摸事件是如何被正确的视图识别并且被正确的处理**。当然我们都知道，iOS 设备上事件有三种(为了便于说明问题，全文都使用触摸事件来举例)：

> 1: 触摸事件
>   2: 摇一摇
>   3: 远程控制事件

从软件层面开始，当手指触摸到屏幕后，触摸事件会被 SpringBoard 这个系统进程拿到并处理：

> 1: 如果没有 App 在前台，则触摸事件交由桌面系统处理，即主屏幕。
>   2: 否则，交给 App 处理，即 UIApplication 这个单例对象。

## App 处理触摸事件的大致过程

* * *

整个处理过程的概括起来看非常简单，我觉得用两句话，就能说透彻：

> 1、寻找第一响应者(First Responder)视图。
>   2、把触摸事件封装为 UIEvent 并将其交给第一响应者，然后从第一响应者开始决策处理该 UIEvent。

**注意顺序**一定是先 1 后2。即先找到最佳响应事件的视图(称为第一响应者)，然后 UIApplication 通过 UIWindow 把封装好的 UIEvent 直接交给第一响应者，让其决策如何处理这个 UIEvent。

## 1、寻找第一响应者

* * *

第一响应者(First Responder)的含义很简单，简单说就是 UIWindow 下，第一个接收到 UIEvent 的视图对象(除 UIWindow)，往往也是最适合处理当前触摸事件的视图(一般是一个 UIView)。比如点击事件发生在一个按钮上，那么默认情况下，该按钮就是第一响应者。如果点击到输入框，那么输入框就是第一响应者。

![first_reponder](i_first_responder.png "first_reponder")

当 Application 开始寻找第一响应者时，会首先询问当前的 UIWindow 对象。UIWindow 会遍历它所有的子视图，来找到最佳响应者， UIWindow 的子视图，在数据结构上是一棵树：

![first_reponder](click_button.png "first_reponder")

假设，UIButton 被点击，那么算法需要从 Root View 开始找到 UIButton；简单来讲，就是要找到**包含了触摸点坐标**的**最远的后裔**节点。是否满足这个条件，通过下边两个方法来判断：

    1：- (nullable UIView *)hitTest:(CGPoint)point withEvent:(nullable UIEvent *)event;

    2: - (BOOL)pointInside:(CGPoint)point withEvent:(nullable UIEvent *)event;

这两个方法定义在 UIView 上；hitTest 内部会调用 pointInside 来判断，触摸点是否被当前视图包含(被子视图包含也算)：

> 1、如果不包含触摸点，放弃当前树杈分支，回到最近的**包含了触摸点的根视图**(hitTest 方法会返回该视图对象)，从另一个分叉继续寻找。
>   2、如果包含触摸点，**递归遍历**当前视图子视图，直到某个满足条件的视图没有子视图(hitTest 方法 return nil)的时候，那么该视图就是**第一响应者**。

这个递归函数其实也很好写，递归的过程就是数据结构中树的递归(这里可以采用深度优先的策略进行后序遍历)，另外**递归的结束条件很明确**：

> 1、pointInside 返回 True。
>   2、当前视图没有子视图，hitTest 返回 nil。

同时满足 1、2，递归结束，第一响应者被找到。另外，在寻找的过程中，一些属性会影响视图是否能成为第一响应者：

> 1、视图 hidden ，即使满足递归条件，也不能被做为第一响应者。
>   2、视图的 userInteractionEnabled 为 false，视图放弃响应用户交互，所以也没有资格成为第一响应者。

最后，开发中，经常会通过重写 hitTest 方法，来扩大按钮的点击区域。

## 2、开始响应 UIEvent 事件

* * *

经过上边的步骤，Application 已经找到了第一响应者对象，接下来 UIWindow 会将封装好的 UIEvent 对象，**直接**交给第一响应者，让其处理。

![event process](event_process.png "event process")

所以，理论上，我们可以通过拦截`-[UIWindow sendEvent:]`拿到 App 内所有的事件 (包括手势识别)。

### 2.1、Responder 对象

视图之所以能处理 UIEvent，是因为它们都是 Responder 对象；iOS API 设计了 UIResponder 对象(Mac 上是 NSReponder)，UIApplication、UIView、UIViewcontroller 都继承了它。这样屏幕上显示的视图对象同时也都是 Reponder 对象。

![first_reponder](ios_responder.jpg "first_reponder")

UIResponder 中定义了 4 个处理事件的重要方法，用来处理 UIEvent 对象：

    - (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
    - (void)touchesMoved:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
    - (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;
    - (void)touchesCancelled:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;

如果视图响应处理 UIEvent 事件，就需要实现上述方法；默认情况下**UIView 视图(除 UIControl、UIScrollView 及其子类，下同)** 并没有实现该方法，此时会走 UIResponder 的默认实现，默认实现具体做的是：**传递响应者链**。

### 2.2、响应者链 Responder Chain

当第一响应者不能处理 UIEvent 的时候，事件会被转发给下一个响应者(next Responder)，下一个响应者不能处理，则继续传递给 next Responder ，以此类推下去，便形成了**响应者链**。UIResponder 的属性 nextResponder 是建立响应者链的桥梁:

> @property(nonatomic, readonly, nullable) UIResponder *nextResponder;

响应者链的链路是: subview --> (viewController) --> parentView --> ... --> UIWindow --> UIApplication (下图的右半部分是 macOS 上的链路) :

![responder_chain](responder_chain.png "responder_chain")

当执行`addSubView:`或`addChildViewController:`方法的时候，parentView 和 parentViewController 会自动被设置为 **nextResponder**。

当然，可以通过重写`-touchesBegan`等方法，来切断响应者链条。具体做法是自己的实现中不调用`-[super touchesBegan ]`即可。

## UIControl 的 Target-Action 设计模式

* * *

在 UIControl 及其子类(UIButton ...)的设计上，iOS Api 采用了 Target-Action 的设计模式。宏观上来看，这并不属于响应者链的一部分，它只是事件处理的一个末端机制。

    -[button addTarget:self action:@selector(buttonClicked:) forControlEvents:UIControlEventTouchUpInside];

上边的方法调用，几乎是开发者天天写的代码，也是典型的 Target-Action 设计模式，从方法名就能看出来：

> 第一参数就是`Target`
>   第二参数就是`Action`
>   第三参数是对 UIEvent 的抽象映射

Target 就是 UIEvent 的新的作用对象(响应者)，Action 是对该 UIEvent 做出响应的具体动作(响应)。UIControl 通过这种 Target-Action 的方式对 UIEvent 进行了转发，从而可以把 UIEvent 事件转发给任意对象处理(原本只有 UIReponder 对象和手势识别器对象才能处理)。

UIControl 实现的具体做法是，重写`-touchesBegan`相关方法，改变响应者链来实现：

1、首先在`-touchesBegan`方法中调用`-sendAction:to:forEvent:`把消息先转发给 UIApplication，让其统一处理；
2、然后 UIApplication 调用`sendAction:to:from:forEvent:`把消息交给具体的 Target 处理。

    - (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
        [self sendAction:@selector(buttonClicked:) to:target forEvent:event];
    }

    - (void)sendAction:(SEL)action to:(id)target forEvent:(UIEvent *)event {
        [[UIApplication sharedApplication] sendAction:action to:target from:self forEvent:event];
    }

![Target Action](control_target_action.png "Target Action")

如果需要拦截 App 内所有的 UIControl 的事件，可以通过 Hook`-[UIApplication sendAction:to:from:forEvent:]`方法实现；Target 是弱引用，如果 Target 不存在，则响应链终止，UIEvent 也不被处理。

## 关于手势识别器

* * *

手势识别器对事件的处理，和 UIResponder 没有直接关系，是一条单独的链路。事实上，在系统找到第一响应者后，UIAppliction 会对产生的同一个 UIEvent 进行**至少两次的分发处理**：

> 1、一次是交给第一响应 UIResponder 对象，让其处理。
>   2、另一次则发送给手势识别系统，让其处理。
>   3、如果有自定义手势识别器，会进行第三次触发

每次发送 UIEvent 都是通过`-[UIWindow sendEvent:]`进行处理的，可以通过断点该方法进行验证。并且每次调用发生在不同的 Runloop 上。

第一次 Runloop 会先发送给 UIResponder 的`-touchesBegan`等方法处理，第二次才会发送给手势识别系统处理。但 UIControl 在实现上做了特殊处理，自身被添加手势的时候，会终止自己的 Target-Action 机制，只响应手势。 看到网上有资料说 UIEvent 事件会先被手势识别器响应，经过验证该说法是错误的，事实刚好相反。

关于为什么 UIWindow 会**至少转发两次同一个 UIEvent 事件**(即使没有给视图添加任何自定义的手势识别器)，是通过断点`-[UIWindow sendEvent:]`验证的，UIEvent(实际上是 UITouchesEvent，UIEvetn 类簇中的一员)属性中有一个装有`_UISystemGestureGateGestureRecognizer`的集合：

![GestureRecognizer](gesture_event.png "GestureRecognizer")

上图断点的视图，并没有手动添加任何自定义的手势识别器，但依然发现同一个 UIEvent 会被`-[UIWindow sendEvent:]`转发两次，第一次结束，会吃掉一个`_UISystemGestureGateGestureRecognizer`，大概猜测该手势是系统需要给视图添加一些默认手势，例如 3D Touch、左滑返回等，当然这也只是猜测，不同的系统版本可能有所不同。

(全文完，可以点赞，转发，转载请注明出处，谢谢)

## 参考文档

* * *

1.  https://developer.apple.com/library/archive/documentation/General/Conceptual/Devpedia-CocoaApp/Responder.html#//apple_ref/doc/uid/TP40009071-CH1-SW1
2.  https://developer.apple.com/library/archive/documentation/General/Conceptual/Devpedia-CocoaApp/EventHandlingiPhone.html#//apple_ref/doc/uid/TP40009071-CH13-SW1
3.  https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/Target-Action/Target-Action.html#//apple_ref/doc/uid/TP40010810-CH12