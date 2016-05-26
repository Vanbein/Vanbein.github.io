---
layout: post
title: UIApplication、UIView生命周期相关
category: iOS基础
tags: iOS Tip
image: http://o7rxin1of.qnssl.com/images/head-800x400/-33.png
description: 记录下和 UIApplication、UIView生命周期相关的知识点
homepage: false
toc: true
---

## UIApplication

1、- (void)**applicationWillResignActive**:(UIApplication *)application

说明：当应用程序将要入非活动状态执行，在此期间，应用程序不接收消息或事件，比如来电话了

2、- (void)**applicationDidBecomeActive**:(UIApplication *)application

说明：当应用程序入活动状态执行，这个刚好跟上面那个方法相反

3、- (void)**applicationDidEnterBackground**:(UIApplication *)application

说明：当程序被推送到后台的时候调用。所以要设置后台继续运行，则在这个函数里面设置即可

4、- (void)**applicationWillEnterForeground**:(UIApplication *)application

说明：当程序从后台将要重新回到前台时候调用，这个刚好跟上面的那个方法相反。

5、- (void)**applicationWillTerminate**:(UIApplication *)application

说明：当程序将要退出是被调用，通常是用来保存数据和一些退出前的清理工作。这个需要要设置UIApplicationExitsOnSuspend的键值。

6、- (void)**applicationDidReceiveMemoryWarning**:(UIApplication *)application

说明：iPhone设备只有有限的内存，如果为应用程序分配了太多内存操作系统会终止应用程序的运行，在终止前会执行这个方法，通常可以在这里进行内存清理工作防止程序被终止

7、- (void)**applicationSignificantTimeChange**:(UIApplication*)application

说明：当系统时间发生改变时执行

8、- (void)**applicationDidFinishLaunching**:(UIApplication*)application

说明：当程序载入后执行

9、- (void)**application**:(UIApplication)application **willChangeStatusBarFrame**:(CGRect)newStatusBarFrame

说明：当StatusBar框将要变化时执行

10、- (void)**application**:(UIApplication*)application **willChangeStatusBarOrientation**:

(UIInterfaceOrientation)newStatusBarOrientation

duration:(NSTimeInterval)duration

说明：当StatusBar框方向将要变化时执行

11、- (BOOL)**application**:(UIApplication \*)application **handleOpenURL**:(NSURL \*)url

说明：当通过url执行

12、- (void)**application**:(UIApplication*)application **didChangeStatusBarOrientation**:(UIInterfaceOrientation)oldStatusBarOrientation

说明：当StatusBar框方向变化完成后执行

13、- (void)**application**:(UIApplication*)application **didChangeSetStatusBarFrame**:(CGRect)oldStatusBarFrame

说明：当StatusBar框变化完成后执行

### 中断情况

接下来考虑中断情况，有以下几种常见情况：

1. 按下home键或双击home键点击任务栏icon运行其它app
2. 有来电通知
3. 有短信或其它app的顶部通知或推送消息。

case 1：**当按下home键**
function:[applicationWillResignActive], state:[Active]
function:[applicationDidEnterBackground], state:[Background]

case 2：**当双击home键，然后选择返回当前app：**
function:[applicationWillResignActive], state:[Active]
function:[applicationDidBecomeActive], state:[Active]

case 3：**当双击home键，然后选择其它app：**
function:[applicationWillResignActive], state:[Active]
function:[applicationDidEnterBackground], state:[Background]

case 4：**当有来电，拒绝接听：**
function:[applicationWillResignActive], state:[Active]
function:[applicationDidBecomeActive], state:[Active]

case 5：**当有来电，接听后并挂断：**
function:[applicationWillResignActive], state:[Active]
function:[applicationDidEnterBackground], state:[Background]
function:[applicationWillEnterForeground], state:[Background]
function:[applicationDidBecomeActive], state:[Active]

case 6：**当有短信或顶部的推送消息，点击查看：**
function:[applicationWillResignActive], state:[Active]
function:[applicationDidEnterBackground], state:[Background]

> 参考文章
> 
> * [http://www.cnblogs.com/wayne23/p/3868441.html](http://www.cnblogs.com/wayne23/p/3868441.html)
> 
> * [http://www.cnblogs.com/pengyingh/articles/2342014.html](http://www.cnblogs.com/pengyingh/articles/2342014.html)



