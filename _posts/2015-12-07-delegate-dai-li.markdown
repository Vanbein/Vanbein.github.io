---
layout: post
title: "delegate--代理"
date: 2015-12-07 15:03:31 +0800
comments: true
categories: Objective-C
---

delegate(委托/代理)是iOS开发中常用的设计模式，表示将一个对象的部分功能转交给另外一个对象，借助协议protocol可以很方便的实现这种设计模式。
<!--more-->
###1. 代理 delegate

#####什么是delegate，苹果的官方文档给了清晰的解释：

> Delegation is a simple and powerful pattern in which one object in a program acts on behalf of, or in coordination with, another object. The delegating object keeps a reference to the other object—the delegate—and at the appropriate time sends a message to it. The message informs the delegate of an event that the delegating object is about to handle or has just handled. The delegate may respond to the message by updating the appearance or state of itself or other objects in the application, and in some cases it can return a value that affects how an impending event is handled. The main value of delegation is that it allows you to easily customize the behavior of several objects in one central object.

纯引用官方解释一点技术含量都没有，也不够通俗易懂，换成大白话可以是下面这样：

都没有，也不够通俗易懂，换成大白话可以是下面这样：

> 你需要做一件事情，你自己不想动手，委托给 delegate 处理，为了避免所托非人，delegate 需要遵守一个你和 delegate 都能接受的 protocol。一个类只要声明它遵守了某个你想要的 protocol，你就能把它信认为是你的 delegate。
生活示例：很多土豪老板都是大忙人，有多个秘书或者助理，老板只想躺着输数钱就好了，接电话、订餐等等这些杂事，统统交给秘书(delegate)去处理就好了，至于接电话或者订餐的具体注意事项，秘书早就跟老板有了约定(protocol)。

说明：

1. 官方解释中delegation是个名词，它本身是一个对象，专门代表被委托对象来和程序中其他对象交互
2. 虽说官方文档有的时候显得抽象，但是很多时候必须读Apple官方文档，因为Apple官方文档的解释是最准确的

#####iOS监听某些事件的方法
* 通知（NSNotificationCenter\NSNotification）
	+ 任何对象之间都可以传递消息
	+ 使用范围
		- 1个对象可以发通知给N个对象
		- 1个对象可以接受N个对象发出的通知
	+ 必须得保证通知的名字在发出和监听时是一致的
* KVO
	+ 仅仅是能监听对象属性的改变（灵活度不如通知和代理）
* 代理delegate
	+ 使用场景
		- 当对象A发生了一些行为，想告知对象B（让对象B成为对象A的代理对象）
		- 对象B想监听对象A的一些行为（让对象B成为对象A的代理对象）
		- 当对象A无法处理某些行为的时候，想委托对象B帮忙处理（让对象B成为对象A的代理对象）
	+ 使用范围
		- 1个对象只能设置一个代理（假设这个对象只有1个代理属性）
		- 1个对象能成为多个对象的代理
	+ 比通知规范，建议使用代理多于使用通知
	
#####delegate的作用
代理是回调监听机制的一种，一个类做不了的事就交给其他类来做。
比如`actionSheet`本身是没办法处理用户交互事件的，所以要让控制器成为其代理来处理。当控制器遵守协议，实现方法，成为它的代理之后，就可以在特定的时间通知控制器进行处理，所以delegate方法里经常有did,should,will这样的词。

###2. Cocoa Touch中的delegate
使用 Xcode 创建一个工程时，Xcode 自动生成的 `AppDelegate.h`& `AppDelegate.m`两个文件，就可以看到 delegate 的使用。`AppDelegate.h`如下所示：


```objc AppDelegate.h
#import <UIKit/UIKit.h>
@interface AppDelegate : UIResponder <UIApplicationDelegate>
@property (strong, nonatomic) UIWindow *window;
@end
```

在`AppDelegate.h` 中，AppDelegate类声明了遵守 `UIApplicationDelegate`协议。查看 `UIApplicationDelegate`协议包含了哪些属性和方法：

```objc UIApplication.h
@protocol UIApplicationDelegate<NSObject>
@optional
- (void)applicationDidFinishLaunching:(UIApplication *)application;
- (BOOL)application:(UIApplication *)application willFinishLaunchingWithOptions:(NSDictionary *)launchOptions NS_AVAILABLE_IOS(6_0);
- (void)applicationDidReceiveMemoryWarning:(UIApplication *)application;
```

由于`UIApplicationDelegate`协议中属性和方法太多，只列出部分供展示在`UIApplicationDelegate`协议中，由关键字@optional可知方法都是可选的。协议所声明的方法可以是必需的(@required)或是可选的(@optional)。默认都是必需的。委托协议中的方法通常都是可选的。
在 `AppDelegate.m`中，实现了应用生命周期的方法(均在协议`UIApplicationDelegate`定义的可选方法):

```objc AppDelegate.m
@implementation AppDelegate
#pragma mark - Application lifecycle methods
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions { return YES;}
-(void)applicationWillResignActive:(UIApplication *)application {}
- (void)applicationDidEnterBackground:(UIApplication *)application {}
- (void)applicationWillEnterForeground:(UIApplication *)application {}
- (void)applicationDidBecomeActive:(UIApplication *)application {}
- (void)applicationWillTerminate:(UIApplication *)application {}
```

那协议`UIApplicationDelegate`有什么作用呢？举个例子：大家都有深刻体会，在 iPhone 上使用一个 App 过程中，一个来电或者锁屏会让 App 进入后台甚至被终止，此外还有其它情况也会导致 App 受到干扰，这时会产生一些系统事件，UIApplication就会通知它的 delegate对象，让 delegate 来处理这些事件。

###3. 自定义 delegate
delegate的使用步骤总的有六步，可分为前三布，后三步：

* 前三步（委托者）：
	1. 声明delegate原型
	2. 声明delegate变量
	3. 调用delegate方法
* 后三步（被委托者）：
	4. 声明服从delegate
	5. 设置delegate的值
	6. 实现delegate方法

前面举了老板和秘书的示例，现在我们就通过简单实现以下这个示例来理解delegate

**Step1：**声明delegate原型，即定义一个protocol

```objc BossDelegate.h
#import <Foundation/Foundation.h>
@class Boss;
@protocol BossDelegate <NSObject>
@optional
//接电话
- (void)tel:(Boss *)theBoss;
//订餐
- (void)buyFood:(Boss *)theBoss;
@end
```

**Step2：**委托者（Boss）声明一个delegate变量，来指明谁是我的委托者，为我办事

```objc Boss.h
#import <Foundation/Foundation.h>
#import "BossDelegate.h"
@interface Boss : NSObject
//声明 delegate 变量
@property (nonatomic, weak) id<BossDelegate> delegate;
@end
```

* 备注：
	+ iOS SDK 中几乎所有的委托都是弱引用属性(weak)，这是为了避免对象及其delegate之间产生强引用循环。
	+ 为了保证任何对象都可以做为代理（被委托者）,所以类型不要写死,用id
	+ 如果让一个控件/控制器成为了代理（被委托者）,那么耦合性会特别强.表现为你离不开我,我离不开你

**Step3：**委托者（Boss）调用delegate内的方法

```objc Boss.h
#import "Boss.h"
@implementation Boss
- (id)init { 
    self = [super init]; 
    if (self) { 
    //
    } 
    return self;
}
- (void)callSecretary {
     //调用 delegate 的方法
    [_delegate tel:self]; 
    [_delegate buyFood:self];
}
@end
```

**Step4：**被委托者（秘书）声明服从delegate

```objc Secretary.h
#import <Foundation/Foundation.h>
#import "BossDelegate.h"
//声明实现 delegate
@interface Secretary : NSObject<BossDelegate>
@end
```

**Step5：**设置delegate的值

```objc main.m
#import <UIKit/UIKit.h>
#import "AppDelegate.h"
#import "Boss.h"
#import "Secretary.h"
int main(int argc, char * argv[]) {
 @autoreleasepool { 
    Boss *boss = [[Boss alloc] init]; 
    Secretary *sec = [[Secretary alloc] init];  
    //设置 delegate 的值
    boss.delegate = sec;  
    [boss callSecretary]; 
    return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class])); 
 }
}
```

**Step6：**被委托者（秘书）实现delegate方法

```objc Secretary.m
#import "Secretary.h"
@implementation Secretary
- (id)init { 
    self = [super init]; 
    if (self) { 
    //
    } 
    return self;
}
- (void)buyFood:(Boss *)theBoss { 
    NSLog(@"秘书正在帮老板订餐...");
}
- (void)tel:(Boss *)theBoss { 
    NSLog(@"秘书正在帮老板接听电话...");
}
@end
```

经过上述六步，一个完整的delegate就实现了，运行程序，结果如下：

```
2015-12-7 17:19:30.487 Protocol_Delegate_Demo[2230:174797] 秘书正在帮老板接听电话...
2015-12-7 17:19:30.489 Protocol_Delegate_Demo[2230:174797] 秘书正在帮老板订餐...
```

由程序运行结果可见，实际的动作是由委托者的 delegate(老板的秘书)去实现的。上述例子只是一个简单的示例，在 iOS 开发中经常用到 delegate 设计模式。比如:实现常见的UITableView就必须遵守`TableViewDataSource`, `UITableViewDelegate`两个协议。原理用法都是相通的，按照delegate使用的前三步、后三步，就可以把delegate这种设计模式应用好。

























