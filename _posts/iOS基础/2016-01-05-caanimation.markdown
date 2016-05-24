---
layout: post
title: CAAnimation入门
category: iOS基础
tags: iOS Animation
image: /images/head-800x400/-21.png
description: 初次学习使用动画，需要用到`CATransition`，于是就先来了解下`CAAnimation`
homepage: false
toc: true
---

初次学习使用动画，需要用到`CATransition`，于是就先来了解下`CAAnimation`

### 一、简单介绍

* `Core Animation`，中文翻译为核心动画，它是一组非常强大的动画处理API，使用它能做出非常炫丽的动画效果，而且往往是事半功倍。也就是说，使用少量的代码就可以实现非常强大的功能。

* `Core Animation`是跨平台的，可以用在Mac OS X和iOS平台。

* `Core Animation`的动画执行过程都是在后台操作的，不会阻塞主线程。不阻塞主线程，可以理解为在执行动画的时候还能点击（按钮）。

* 要注意的是，`Core Animation`是直接作用在`CALayer`上的，并非`UIView`。

### 二、Core Animation的使用步骤

1. 使用它需要先添加`QuartzCore.framework`框架和引入主头文件`<QuartzCore/QuartzCore.h>`(iOS7以后不需要)

2. 初始化一个CAAnimation对象，并设置一些动画相关属性

3. 通过调用CALayer的`addAnimation:forKey:`方法增加CAAnimation对象到CALayer中，这样就能开始执行动画了

4. 通过调用CALayer的`removeAnimationForKey:`方法可以停止CALayer中的动画

### 三、CAAnimation类的继承结构图


![CAAnimation](/images/2016/01/CAAnimation.png "CAAnimation类继承结构图")

#### CAAnimation类

> CAAnimation is an abstract animation class. It provides the basic support for the CAMediaTiming and CAActionprotocols.

* CAAnimation类，是一个抽象类。遵循CAMediaTiming协议和CAAction协议！

* CAMediaTiming协议: 可以调整时间,包括持续时间,速度,重复次数。

* CAAction协议: 可以通过响应动作的方式来显示动画。

* CAAnimation是所有动画类的父类，但是它不能直接使用，应该使用它的子类。

#### CAAnimation的派生类

* 能用的动画类只有4个子类
	* CATransition 提供渐变效果:(推拉push效果,消退fade效果,揭开reveal效果)。
	* CAAnimationGroup 允许多个动画同时播放。
	* CABasicAnimation 提供了对单一动画的实现。
	* CAKeyframeAnimation 关键桢动画,可以定义行动路线。

* CAPropertyAnimation
	* 是CAAnimation的子类，但是不能直接使用，
	* 要想创建动画对象，应该使用它的两个子类：`CABasicAnimation` 和 `CAKeyframeAnimation`
	* 它有个NSString类型的`keyPath`属性，你可以指定CALayer的某个属性名为`keyPath`，并且对CALayer的这个属性的值进行修改，达到相应的动画效果。比如，指定`@"position"`为`keyPath`，就会修改CALayer的position属性的值，以达到平移的动画效果


##### 另外其他派生类

* CAConstraint 约束类,在布局管理器类中用它来设置属性。
* CAConstraintLayoutManager 约束布局管理器,是用来将多个CALayer进行布局的.各个CALayer是通过名称来区分,而布局属性是通过CAConstraint来设置的。
* CATransaction 事务类,可以对多个layer的属性同时进行修改.它分隐式事务,和显式事务。


### 四、动画实例

看一段动画代码：

{% highlight objc linenos %}
	//给imageView设置新图片
	[_imgView setImage:[UIImage imageNamed:@"QQ"]];
	//添加imageView的切换动画，
    CATransition *animation = [CATransition animation];
    [animation setDuration:1.0];
    [animation setFillMode:kCAFillModeForwards];
    [animation setTimingFunction:[CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseOut]];
    [animation setType:@"rippleEffect"];// rippleEffect
    [animation setSubtype:kCATransitionFromTop];
    [_imgView.layer addAnimation:animation forKey:nil];
{% endhighlight %}

实现功能就是，在UIImageView换新图片的时候，做相应的动画效果！好让，UIImageView转化时，不至于太单调！

首先第一句：

{% highlight objc linenos %}
CATransition *animation = [CATransition animation];
{% endhighlight %}

因为`CATransition`是`CAAnimation`的子类,所以直接通过`CATransition`的`+ (id)animation` 方法创建了一个`CATransition`对象



### 五、CAAnimation属性

* delegate

{% highlight objc linenos %}
@property(retain) id delegate
{% endhighlight %}

为CAAnimation设置代理。默认为nil。
注意：一个CAAnimation实例，不能设置delegate为self。会引起循环引用。

* removedOnCompletion

{% highlight objc linenos %}
@property(getter=isRemovedOnCompletion) BOOL removedOnCompletion
{% endhighlight %}

设置是否动画完成后，动画效果从设置的layer上移除。默认为YES，代表动画执行完毕后就从图层上移除，图形会恢复到动画执行前的状态。如果想让图层保持显示动画执行后的状态，那就设置为NO，不过还要设置fillMode为kCAFillModeForwards

* timingFunction

{% highlight objc linenos %}
@property(retain) CAMediaTimingFunction *timingFunction
{% endhighlight %}

速度控制函数，控制动画运行的节奏，也就是动画自身的“节奏”：比如：开始快，结束时变慢；开始慢，结束时变快；匀速；等，在动画过程中的“时机”效果。

* animation

{% highlight objc linenos %}
+ (id)animation
{% endhighlight %}

创建并返回一个CAAnimation实例。

* defaultValueForKey

{% highlight objc linenos %}
+ (id)defaultValueForKey:(NSString *)key
{% endhighlight %}

根据属性key，返回相应的属性值。

### 六、来自CAMediaTiming协议的属性

* duration：动画的持续时间

* repeatCount：动画的重复次数

* repeatDuration：动画的重复时间

* fillMode：决定当前对象在非active时间段的行为.比如动画开始之前,动画结束之后

* beginTime：可以用来设置动画延迟执行时间，若想延迟2s，就设置为`CACurrentMediaTime()+2`，`CACurrentMediaTime()`为图层的当前时间

### 七、CAAnimation实例方法

* shouldArchiveValueForKey

{% highlight objc linenos %}
- (BOOL)shouldArchiveValueForKey:(NSString *)key
{% endhighlight %}

返回指定的属性值是否可以归档。
> key：指定的属性。
> YES：指明该属性可以被归档；NO：不能被归档。

#### CAAnimation协议方法
* animationDidStart

{% highlight objc linenos %}
- (void)animationDidStart:(CAAnimation *)theAnimation
{% endhighlight %}

动画开始时，执行的方法。
theAnimation：正在执行动画的CAAnimation实例。

* animationDidStop:finished

{% highlight objc linenos %}
- (void)animationDidStop:(CAAnimation *)theAnimation finished:(BOOL)flag
{% endhighlight %}

动画执行完成或者动画为执行被删除时，执行该方法。
> theAnimation：完成或者被删除的动画实例
> flag：标志该动画是执行完成或者被删除：YES：执行完成；NO：被删除。

如果我们有多个动画，在这个代理方法中需要进行判断是那一个动画的delegate，可以通过在配置动画时给动画对象添加一个键值对，通过`setValue: forKey:`方法。
















