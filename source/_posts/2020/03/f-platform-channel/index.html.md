---
layout: post
title: Flutter Platform channel 分析
date: 2020-03-04
tags: ["Android Flutter","BinaryMessageHandler","BinaryMessenger","flutter","FlutterEngine","FlutterViewController","iOS Flutter","MessageCodec","MethodChannel","MethodCodec","Platform channel","日志","前端知识","计算机基础知识"]
categories:
- 计算机
---

作者：xiaobo

* * *

![flutter](flutter_top.png "flutter")

Platform channel 是 Flutter 与 Native 平台通信的桥梁，它能帮助我们调用一些 Flutter 无法支持的平台属性：

> 推送通知、生命周期、deep links...
>   传感器、相机、地理位置、电池、声音...
>   分享到其他 app、打开其他 app 等等...

## 阅读须知

* * *

该篇文档基于的 Flutter 版本：

> Flutter 1.12.13+hotfix.5

该篇文档基于的 Dart 版本：

> Dart 2.7.0

该篇文档的 Native 接口说明及代码示例，均为 iOS 平台。但原理部分 iOS 和 Android 是一致的。

## Platform channel 分类

* * *

> 1、BasicMessageChannel：用于传递字符串和半结构化的信息。
>   2、MethodChannel：用于传递方法调用（method invocation），是我们在开发中最常使用的 channel。
>   3、EventChannel: 用于数据流（event streams）的通信。

## Platform channel 属性

* * *

> 1、name : Channel 的唯一标识，也是 Channel 的名字。
>   2、messager: BinaryMessenger 类型，代表消息信使，是消息的发送与接收的工具。
>   3、codec: MessageCodec 类型或 MethodCodec类型，消息的编解码器。

## BinaryMessenger 类型

* * *

经上所述，Platform channel 有三种类型，但这三种类型的 Channel 使用的是同一个 BinaryMessenger。在 iOS 中  BinaryMessenger 被定义为一个 Protocol: `<FlutterBinaryMessenger>`。实现了该 Protocol 的实例，可以处理 Channel 。

iOS 平台，FlutterEngine 持有一个实现了该协议的属性 binaryMessenger。

    FlutterEngine.h 文件
    /*
      The `FlutterBinaryMessenger` associated with this FlutterEngine (used for communicating with channels).
     */
    @property(nonatomic, readonly) NSObject<FlutterBinaryMessenger>* binaryMessenger;

同样也可以通过 FlutterViewController 的 binaryMessenger 获取该实例，注意它和`FlutterEngine`中的 binaryMessenger 是同一个实例；为了方便获取所以在 FlutterViewController 中也有这样的`readonly` 的属性，这一点在头文件的注释中有明确说明：

    FlutterViewController.h 文件
    /*
     The `FlutterBinaryMessenger` associated with this FlutterViewController (used for communicating with channels).

     This is just a convenient way to get the 'FlutterEngine''s binary messenger.
     */
    @property(nonatomic, readonly) NSObject<FlutterBinaryMessenger>* binaryMessenger;

## Name 与 BinaryMessageHandler

* * *

name 是一个 Channel 的唯一标识，它与 Channel 一一对应。而每个 Channel 又对应一个 BinaryMessageHandler，来处理结果。

![flutter](flutter-channel.JPG)

如上图，Channel 与 BinaryMessageHandler 一一对应，BinaryMessageHandler 接受的数据都是二进制数据，不能直接使用，需要解码。解码后，我们自己定义的实现逻辑的 Handler 会响应具体结果；需要我们实现的 Handler 分为三种，分别对应三种 Channel：

> 1、BasicMessageChannel：MessageHandler。iOS 中具体是一个 Block：FlutterMessageHandler
>   2、MethodChannel：MethoderHandler。 iOS 中具体是一个 Block：FlutterMethodCallHandler
>   3、EventChannel: StreamHandler。iOS 中是一个遵循 <FlutterStreamHandler> 协议的实例：`NSObject<FlutterStreamHandler>*`

上述信息可以在 Flutter iOS 侧的 `FlutterChannels.h` 查看。

## 解码器 Codec

* * *

解码器有两种类型，MessageCodec 和 MethodCodec，每种解码器有多种实现。

> 1: MessageCodec (处理 BasicMessageChannel 的解码)
> 
> > a. BinaryCodec
> >     b. StringCodec
> >     c. JSONMessageCodec
> >     d. StandardMessageCodec
> 
>   2: MethodCodec (处理 MethodChannel 和 EventChannel  的解码)
> 
> > a. JSONMethodCode
> >     b. StandardMethodCodec

相关信息，可以在 FlutterCodecs.h 中查看。

## 详解 binaryMessenger 通信与 Channel

* * *

综上，我们已经了解了 Platform channel 的概况，解码器的作用就是对二进制数据进行格式化，方便后续使用；同样为了方便使用，Flutter 为我们准备了 BasicMessageChannel、MethodChannel、EventChannel，这些你都可以简单的看做是工具类。真正负责通信的是 FlutterBinaryMessenger。

Flutter 在 iOS 平台上的 `FlutterBinaryMessenger.h` 头文件定义了 binaryMessenger 具体行为：
![messenger](flutter_messenger.png)

而实现了该 Protocol 的实例，我们可以从 FlutterEngine 或者 FlutterViewController 的 binaryMessenger 属性获得，它们是同一个 binaryMessenger 实例(前文已经说明)。所以我们可以使用下列代码完成 Dart 和 iOS 平台的通信：

    // Send a binary message from iOS (Swift).
    var message = Data(capacity: 12)
    var x : Float64 = 3.1415
    var n : Int32 = 12345678
    message.append(UnsafeBufferPointer(start: &x, count: 1))
    message.append(UnsafeBufferPointer(start: &n, count: 1))
    flutterView.send(onChannel: "foo", message: message) {(_) -> Void in
      os_log("Message sent, reply ignored")
    }

    // Receive binary messages from the platform.
    BinaryMessages.setMessageHandler('foo', (ByteData message) async {
      final ReadBuffer readBuffer = ReadBuffer(message);
      final double x = readBuffer.getFloat64();
      final int n = readBuffer.getInt32();
      print('Received $x and $n');
      return null;
    });

上述代码完成了最底层的 binary 数据的简单通信，有几点需要注意：

*   通信线程: 在 Native 侧，消息发送和接受必须要在 UI 主线程进行调用。而 Dart 语言这边，一个 isolate 内只跑一个线程，所以不需要考虑在哪个线程调用。
*   异步通信: 通信都是异步的，避免线程死锁和性能损耗。
*   Handler：Handler 和 Channel name 是一对一的，通过Hash Map 存储。赋值为 null ，则不做回调响应。

简单的 binary 数据，我们并不能识别，此时就需要 Platform channel 出场；没错 Platform channel 就是包含了 channel name 和 codec(编解码) 的工具，下边以 BasicMessageChannel 为例：

    // String messages
    // Dart side
    const channel = BasicMessageChannel<String>('foo', StringCodec());

    // Send message to platform and receive reply.
    final String reply = await channel.send('Hello, world');
    print(reply);

    // Receive messages from platform and send replies.
    channel.setMessageHandler((String message) async {
      print('Received: $message');
      return 'Hi from Dart';
    });

    // iOS side
    let channel = FlutterBasicMessageChannel(
        name: "foo",
        binaryMessenger: controller,
        codec: FlutterStringCodec.sharedInstance())

    // Send message to Dart and receive reply.
    channel.sendMessage("Hello, world") {(reply: Any?) -> Void in
      os_log("%@", type: .info, reply as! String)
    }

    // Receive messages from Dart and send replies.
    channel.setMessageHandler {
      (message: Any?, reply: FlutterReply) -> Void in
      os_log("Received: %@", type: .info, message as! String)
      reply("Hi from iOS")
    }

如上所述，Platform channel 仅仅是个轻量级无状态的工具而已：

*   channel 不进行消息通信，通信过程会委托给二进制传递层。
*   channel 不会追踪和保存 Handler。
*   不建议两个 channel 使用同一个 channel name。

以 BasicMessageChannel 为例，整体通信的流程图如下：
![messageChannel](ft_message_channel.png)

MethodChannel 和 EventChannel 原理相同。

(全文完)

## 参考文档

* * *

1.  https://medium.com/flutter/flutter-platform-channels-ce7f540a104e
2.  https://mp.weixin.qq.com/s/FT7UFbee1AtxmKt3iJgvyg