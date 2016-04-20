---
layout: post
title: NSThread的基本使用
category: iOS进阶
tags: iOS 多线程
image: /images/head-800x400/-38.png
description: NSThread是iOS中实现多线程的四种方案之一，这个虽然自己管理线程的生命周期，但这是面向对象的，OC语言，不常用到但要会用
homepage: false
toc: true
---

## 一、iOS中实现多线程的四种方案
* pthread 纯c语言，还要自己管理线程的生命周期，几乎不用

* NSThread 这个虽然自己管理线程的生命周期，但这是面向对象的，OC语言，不常用到但要会用

* GCD 非常重要 自动管理线程的生命周期  使用率非常非常频繁

* NSOpreation 也很重要，必须掌握，没什么好说的，也是自动管理线程的生命周期，使用率也非常高

## 二、关于NSThread

### 创建方法

**1. 手动开启线程**

线程一启动，就会在线程thread中执行self的run方法

{% highlight objc linenos %}
// 1.创建线程
NSThread *thread = [[NSThread alloc] initWithTarget:selfselector:@selector(run) object:nil];
// 2.设置线程的优先级(0.0 - 1.0，1.0最高级)  
thread.threadPriority = 1; 
// 3.启动线程  
[thread start];
{% endhighlight %}

**2. 自动开启线程**

`detachNewThreadSelector:` 不用手动调用start方法, 系统会自动启动  没有返回值, 不能对线程进行更多的设置

{% highlight objc linenos %}
[NSThread detachNewThreadSelector:@selector(run) toTarget:self withObject:nil];
{% endhighlight %}

**3. 隐式创建并启动线程**

系统会自动创建一个子线程, 并且在子线程中自动执行self的@selector方法

{% highlight objc linenos %}
[self performSelectorInBackground:@selector(run) withObject:nil];
{% endhighlight %}

### NSThread用法

* 主线程相关用法

{% highlight objc linenos %}
+ (NSThread *)mainThread; // 获得主线程
- (BOOL)isMainThread; // 是否为主线程
+ (BOOL)isMainThread; // 是否为主线程
NSThread *main = [NSThread mainThread];
{% endhighlight %}

* 获得当前线程

{% highlight objc linenos %}
NSThread *current = [NSThread currentThread];
{% endhighlight %}

* 获取线程的名字

{% highlight objc linenos %}
- (void)setName:(NSString *)name;
- (NSString *)name;
{% endhighlight %}

## 三、线程的状态

* 创建出来 -> 新建状态
* 调用start -> 准备就绪
* 被CPU调用 -> 运行
* sleep -> 阻塞
* 执行完毕, 或者被强制关闭 -> 死亡

#### 启动线程

{% highlight objc linenos %}
- (void)start; 
// 进入就绪状态 -> 运行状态。当线程任务执行完毕，自动进入死亡状态
阻塞（暂停）线程
// sleep方法是一个类方法, 所以说明在哪个线程中调用, 就会给哪个线程设置暂时
+ (void)sleepUntilDate:(NSDate *)date;
+ (void)sleepForTimeInterval:(NSTimeInterval)time;
{% endhighlight %}

#### 阻塞（暂停）线程

{% highlight objc linenos %}
// 暂停1s  
[NSThread sleepForTimeInterval:1]; 
[NSThread sleepUntilDate:[NSDate dateWithTimeInterval:1 sinceDate:[NSDate date]]];
{% endhighlight %}


#### 强制停止线程
注意：一旦线程停止（死亡）了，就不能再次开启任务

{% highlight objc linenos %}
// 进入死亡状态
+ (void)exit;
{% endhighlight %}

## 多线程的安全隐患及解决措施


* ### 资源共享

	+ 一块资源可能会被多个线程共享，也就是多个线程可能会访问同一块资源，比如多个线程访问同一个对象、同一个变量、同一个文件等
	+ 当多个线程访问同一块资源时，很容易引发数据错乱和数据安全问题

* ### 解决措施
	+ 互斥锁
	+ 互斥锁的使用前提：多条线程抢夺同一块资源
	+ 注意：锁定1份代码只用1把锁，用多把锁是无效的

* ### 使用格式

{% highlight objc linenos %}
// (锁对象self)
@synchronized
{ 
  // 需要锁定的代码      
}
{% endhighlight %}

只要被 `@synchronized`的{}包裹起来的代码, 同一时刻就只能被一个线程执行

> 注意:
> 
>1. 只要枷锁就会消耗性能
>2. 加锁必须传递一个对象, 作为锁
>3. 如果想真正的锁住代码, 那么多个线程必须使用同一把锁才行
>4. 加锁的时候尽量缩小范围, 因为范围越大性能就越低
      
* ### 互斥锁的优缺点

	* 优点：能有效防止因多线程抢夺资源造成的数据安全问题
	* 缺点：需要消耗大量的CPU资源

* ### 线程同步

	* 线程同步的意思是：多条线程在同一条线上执行（按顺序地执行任务）
互斥锁，就是使用了线程同步技术

## 原子和非原子属性

* #### OC在定义属性时有nonatomic和atomic两种选择
	* atomic：原子属性，为setter方法加锁（默认就是atomic）
	* nonatomic：非原子属性，不会为setter方法加锁

* #### 原子和非原子属性的选择
	* atomic：线程安全，需要消耗大量的资源
	* nonatomic：非线程安全，适合内存小的移动设备

* #### iOS开发的建议
	* 所有属性都声明为nonatomic
	* 尽量避免多线程抢夺同一块资源
	* 尽量将加锁、资源抢夺的业务逻辑交给服务器端处理，减小移动客户端的压力
	* 注意点: atomic系统自动给我们添加的锁不是互斥锁/ 自旋锁

* 自旋锁和互斥锁对比
	* 相同点：都能够保证多线程在同一时候, 只能有一个线程操作锁定的代码
	* 不同点
		* 如果是互斥锁, 假如现在被锁住了, 那么后面来得线程就会进入”休眠”状态, 直到解锁之后, 又会唤醒线程继续执行
		* 如果是自旋锁, 假如现在被锁住了, 那么后面来得线程不会进入休眠状态, 会一直傻傻的等待, 直到解锁之后立刻执行
		* 自旋锁更适合做一些较短的操作


## 线程间的通信

* ### 什么叫做线程间通信

	* 在1个进程中，线程往往不是孤立存在的，多个线程之间需要经常进行通信
	* 比如在主线程添加imageView，在子线程中下载图片，然后又回到主线程中显示图片
	* 注意点: 更新UI一定要在主线程中更新

* ### 线程间通信的体现
	* 1个线程传递数据给另1个线程
	* 在1个线程中执行完特定任务后，转到另1个线程继续执行任务

* ### 在当前线程

{% highlight objc linenos %}
[self performSelector:@selector(run) withObject:nil];
{% endhighlight %}

* 在主线程

{% highlight objc linenos %}
[self performSelectorOnMainThread:@selector(run) withObject:nil waitUntilDone:YES];
{% endhighlight %}

* 在指定线程

{% highlight objc linenos %}
[self performSelector:@selector(run) onThread:thread withObject:nil waitUntilDone:YES];
{% endhighlight %}
> waitUntilDone的含义: 
> 如果传入的是YES: 那么会等到主线程中的方法执行完毕, 才会继续执行下面其他行的代码 
> 如果传入的是NO: 那么不用等到主线程中的方法执行完毕, 就可以继续执行下面其它行的代码

线程间的通信最常见的就是在子线程中做耗时操作，然后回到主线程刷新UI














