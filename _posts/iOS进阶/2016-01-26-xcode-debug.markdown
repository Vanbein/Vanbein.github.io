---
layout: post
title: Xcode常用的Debug调试技巧总结
category: iOS进阶
tags: iOS debug
image: /images/head-800x400/-13.png
description: 本文主要记录下Xcode结合LLDB调试命令以及OBJC运行时的调试技巧，不定期更新.
homepage: true
---

本文主要记录下Xcode结合LLDB调试命令以及OBJC运行时的调试技巧，不定期更新 


###常用宏定义
####1、OPTIMIZE，Debug和Release判定

* Release编译时定义
* 当我们想要某些代码只在Debug环境下才运行可以使用此宏定义判别，代码如下：

```objc
- (BOOL)isDebugMode{ 
#ifndef __OPTIMIZE__
    return YES;
#else
    return NO;
#endif
}
```

####2、i386与x86_64，模拟器环境判定
有时工程依赖的Lib库只编译了真机的代码，模拟器编译出错。为了可以模拟器调试，使用此宏略过不能编译的代码

```objc
- (BOOL)isSimulatorMode{
#if defined (__i386__) || defined (__x86_64__)
    return YES;  //模拟器
#else
    return NO; //真机
#endif
}
```

####3、__IPHONE_8_0等，编译SDK的判定

* SDK中会声明当前SDK版本定义，并且保留过往SDK版本定义
* 有时我们编写的代码可能不会在最新的SDK编译，旧版本SDK编译会有潜在问题。使用此判定针对新老SDK版本分别编写代码
* `__IPHONE_OS_VERSION_MAX_ALLOWED`，此宏定义声明了当前编译的SDK版本，可进行比较

```
#define __IPHONE_2_0     20000
#define __IPHONE_2_1     20100
#define __IPHONE_2_2     20200
#define __IPHONE_3_0     30000
#define __IPHONE_3_1     30100
#define __IPHONE_3_2     30200
#define __IPHONE_4_0     40000
#define __IPHONE_4_1     40100
#define __IPHONE_4_2     40200
#define __IPHONE_4_3     40300
#define __IPHONE_5_0     50000
#define __IPHONE_5_1     50100
#define __IPHONE_6_0     60000
#define __IPHONE_6_1     60100
#define __IPHONE_7_0     70000
#define __IPHONE_7_1     70100
#define __IPHONE_8_0     80000
#define __IPHONE_8_1     80100
#define __IPHONE_8_2     80200
#define __IPHONE_8_3     80300
#define __IPHONE_8_4     80400
#define __IPHONE_9_0     90000
#define __IPHONE_9_1     90100
#define __IPHONE_9_2     90200
//······
```

####4、NOP，空语句

* C语言中有nop()函数，表示一条空语句。OC中没有提供。
* 可以设计一个宏进行代替，用于挂载条件断点调试使用。

```objc
#ifndef __OPTIMIZE__
    #define ___NOP___   \assert(1)
#else
    #define ___NOP___
#endif
```

###二、常用调试命令

####1、PO，输出对象信息

* po调用NSObject的description方法打印对象
* po 的使用格式：
	* po 变量名
	* po 内存地址
	* po 可返回对象的表达式
* 如果对象没有被释放，PO会输出内容，否则只输出一个代表野指针的数字。根据此特性可在循环引用* 检查中判定对象是否被正常释放。
* 配合表达式进行复杂查询

####2、P，输出变量的值

* 使用格式：p (类型)表达式
* 除了值类型，也可以直接打印结构体数据

![p,输出变量的值](/images/2016/01/p.png "LLDB--p,输出变量的值")


####3、Call，执行一段代码
* 使用格式：call (返回类型)表达式
* 调试状态下，对于点语法支持不佳。如果发现符号未找到的情况，尝试使用发送消息的方式。如果还不行，根据提示，声明类型。对于PO、P等指令同样也需要注意

![call_1](/images/2016/01/call_1.png)

* 使用call可以修改值。
* gdb下可以使用set实现，但LLDB下set语法含义变化了，用call替代set指令
配合NOP与断点，可以在运行时动态的控制程序的运行状态

![call_2](/images/2016/01/call_2.png)

* 可以执行一段代码

![call_3](/images/2016/01/call_3.png)

####4、bt，打印调用栈

![bt](/images/2016/01/bt.png)


###三、断点的使用技巧

####1、一个普通断点可以做什么事情

1. 可以加入一个条件，当满足此条件时触发断点。
2. 或者当执行某些次后才会触发
3. 可以执行若干种类的action
4. 当经过此断点时，执行action但是不会中断程序

![breakpoint](/images/2016/01/breakpoint_1.png)

####2、略特殊的断点
在`Breakpoint Navigator`栏的左下角，点击加号可以给程序添加几种不同类型的断点：

![breakpoint_2](/images/2016/01/breakpoint_2.png)

####3、Exception Breakpoint 全局异常断点 

* 某些crash是由程序抛出的异常导致的，比如数组越界。可以通过添加异常断点监控异常抛出的位置。

####4、符号断点 Symbolic Breakpoint

* 当符号对应的方法被调用时，中断
* -/+[类 方法]
* 举个例子：当UIViewController被载入时触发，并将当前的调用栈输出

![breakpoint_3](/images/2016/01/breakpoint_3.png)


####5、内存断点

* 当该块内存被调用时中断
* 可通过XCode调试界面简化指令添加

![breakpoint_4](/images/2016/01/breakpoint_4.png)

* 添加时，需要预先打断，找到要监视的内存地址（注意，地址每次启动都会失效）
* 可以输出值的变化。可以找出内存是何时被修改的，值是如何变化的。

![breakpoint_5](/images/2016/01/breakpoint_5.png)

###四、灵活的使用调试手段

以上这些调试手段虽然看起来比较简单，但只要灵活运用，就可以为调试带来很多便利和可能性。
比如下面一个例子

![debug](/images/2016/01/debug.png)

图中有三个变量a，b，c。只要在NOP语句加入一个执行call命令的条件断点，通过调节断点的开闭，就可以在程序运行时动态的控制a,b,c的数值。这使得不用编写调试代码，就可以模拟各个状态，动态的调试程序分支。

此外，当程序发生异常时，一般是通过控制台报错信息，被动的定位问题所在。如果使用调试命令结合OBJC的运行时，可以主动的获知发生异常的状态。在MRC开发的时代，对于多次释放对象的问题，甚至可以一定程度的将僵尸对象还原回原始的对象，从而定位问题所在。

所以只要灵活的运用上面这些调试手段，很快就能成为一个调试高手。


