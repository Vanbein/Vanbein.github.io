---
layout: post
title: "Notification--通知"
date: 2015-12-09 16:30:05 +0800
comments: true
categories: [Objective-C, iOS基础]
---

Notification(通知)是iOS系统下重要的消息传递机制之一，通知封装了诸如窗口获得焦点、网络连接关闭等事件信息，通知的内容可按照我们实际的需求来定制。在实际开发中或多或少都会接触到。

<!--more-->

概览
在对象间进行信息传递的标准方式就是消息(一个对象调起另一对象的方法)。这就要求发送消息的对象必须知道接收者是谁，消息的响应者是谁。在iOS系统下，通过广播机制来达到这个目的，对象向通知中心`NSNotificationCenter`投递一个通知，由通知中心将其收到的通知分发给“感兴趣”的接收者。通知中心的原理非常符合设计模式中的观察者(Observer）模式。

`NSNotificationCenter` 与 `Delegate` 类似，都属于对象之间通信的手段，但也存在不同，开发者需要根据具体应用场景选择恰当的通信方式。不同点在于：

* Delegate通常是对象一对一的通信，耦合性高，而通知可以是一对多个对象，并且不再需要返回值，耦合性相对较低。

* 对象可以从通知中心中接收任意个它感兴趣的通知，而不像代理只能接收预设的方法。

* 发送通知的对象甚至不知道（也许不想知道）通知消息响应者们（Observers）的存在。

##二、通知

通知`NSNotification`结构中主要包括：

```objc
@property (readonly, copy) NSString *name;  // 通知的标识名称(一般为常量字符串)
@property (readonly, retain) id object;  // 任意想要携带的对象，通常为发送者自己
@property (readonly, copy) NSDictionary *userInfo; // 关于通知的附加信息
```

##三、通知中心
Cocoa框架中包括两种通知中心：

```
NSNotificationCenter 单进程通知管理
NSDistributedNotificationCenter 一台机器中多个进程的通知管理
```

###NSNotificationCenter
每个进程都包含一个默认的通知中心，通过［NSNotificationCenter defaultCenter］获取，需要注意的是：

1. 通知中心分发给观察者处理采用同步机制，也就是说，当某一对象发送一个通知时，会一直阻塞在发送方法内，直到通知中心将该通知分发给所有观察者并且全部成功处理返回后，发送者才能执行其所在线程内的后续代码。如果需要异步发送通知，可以使用文章后面的“通知队列”。
2. 在多线程的应用程序中，通知总是在发送的线程中传送，这个线程可能不同于观察者注册所在的线程。

**发送通知方法：**

```objc
- (void)postNotification:(NSNotification *)notification;
- (void)postNotificationName:(NSString *)aName object:(nullable id)anObject;
- (void)postNotificationName:(NSString *)aName object:(nullable id)anObject userInfo:(nullable NSDictionary *)aUserInfo;
```

**注册通知方法:**

```objc
- (void)addObserver:(id)observer selector:(SEL)aSelector name:(nullable NSString *)aName object:(nullable id)anObject;
```

注意到`addObserver:selector:name:object:`不仅指定一个观察者，指定通知中心发送给观察者的消息，还有接收通知的名字，以及指定的对象。一般来说需要指定name和object，但如果仅仅指定了一个object，观察者将收到该对象的所有通知。例如代码中name为nil,那么观察者将接收到object对象的所有消息，但是确定不了接收这些消息的顺序。如果指定了一个通知名称，观察者将只收到它每次发出的通知。例如，上面的代码中object为nil，那么客户对象（self）将收到任何对象发出的通知。如果既没有指定指定object，也没有指定name，那么该观察者将收到所有对象的所有消息。


**移除通知方法**

```objc
- (void)removeObserver:(id)observer;
- (void)removeObserver:(id)observer name:(nullable NSString *)aName object:(nullable id)anObject;
```

其中，`removeObserver:`是删除通知中心保存的调度表一个观察者的所有入口，而`removeObserver:name:object:`是删除匹配了通知中心保存的调度表中观察者的一个入口。










