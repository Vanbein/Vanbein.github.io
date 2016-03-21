---
layout: post
title: 动画CAAnimation系列--CABasicAnimation
category: iOS-Pro
tags: iOS Animation
image: /images/head-800x400/-9.png
description: CABasicAnimation 类是 CAAnimation 的子类，CABasicAnimation通过设定起始点，终点，时间，动画会沿着你这设定点进行移动。可以看做特殊的 CAKeyFrameAnimation
homepage: false
---

在iOS中，展示动画可以类比于显示生活中的“拍电影”。拍电影有三大要素：演员+剧本+开拍，概念类比如下：

* 演员 ---> CALayer，规定电影的主角是谁
* 剧本 ---> CAAnimation，规定电影该怎么演，怎么走，怎么变换
* 开拍 ---> addAnimation，开始执行

####1、CALayer是什么呢？

* CALayer是个与UIView很类似的概念，同样有layer,sublayer...，同样有backgroundColor、frame等相似的属性，我们可以将UIView看做一种特殊的CALayer，只不过UIView可以响应事件而已。
* 一般来说，layer可以有两种用途，二者不互相冲突：
	* 一是对view相关属性的设置，包括圆角、阴影、边框等参数，更详细的参数请点击这里；
	* 二是实现对view的动画操控。
* 因此对一个view进行core animation动画，本质上是对该view的.layer进行动画操纵。

####2、CAAnimation是什么呢？

CAAnimation可分为四种：

1. CABasicAnimation
通过设定起始点，终点，时间，动画会沿着你这设定点进行移动。可以看做特殊的CAKeyFrameAnimation
2. CAKeyframeAnimation
Keyframe顾名思义就是关键点的frame，你可以通过设定CALayer的始点、中间关键点、终点的frame，时间，动画会沿你设定的轨迹进行移动
3. CAAnimationGroup
Group也就是组合的意思，就是把对这个Layer的所有动画都组合起来。PS：一个layer设定了很多动画，他们都会同时执行，如何按顺序执行我到时候再讲。
4. CATransition
这个就是苹果帮开发者封装好的一些动画

####CABasicAnimation用法

`CABasicAnimation` 是`CAPropertyAnimation`的子类，自己只有三个property

```
@property(nullable, strong) id fromValue; //keyPath相应属性的初始值
@property(nullable, strong) id toValue; //keyPath相应属性的结束值
@property(nullable, strong) id byValue;
```

当你创建一个`CABasicAnimation`时,你需要通过`-setFromValue` 和`-setToValue` 来指定一个开始值和结束值。 当你增加基础动画到层中的时候,它开始运行。当用属性做动画完成时,例如用位置属性做动画,层就会立刻返回到它的初始位置，

> 随着动画的进行，在长度为`duration`的持续时间内，`keyPath`相应属性的值从`fromValue`渐渐地变为`toValue`
> 
> 如果`fillMode = kCAFillModeForwards`和`removedOnCompletion = NO`，那么在动画执行完毕后，图层会保持显示动画执行后的状态。但在实质上，**图层的属性值还是动画执行前的初始值，并没有真正被改变**。
> 比如：CALayer的`position`初始值为(0,0)，`CABasicAnimation`的`fromValue`为(10,10)，`toValue`为(100,100)，虽然动画执行完毕后图层保持在(100,100)这个位置，实质上图层的`position`还是为(0,0)


####下面是一些继承的常用的属性:

* Autoreverses：当你设定这个属性为 YES 时,在它到达目的地之后,动画的返回到开始的值,代替了直接跳转到 开始的值。

* Duration：这个参数你已经相当熟悉了。它设定开始值到结束值花费的时间。期间会被速度的属性所影响。 

* RemovedOnCompletion：这个属性默认为 YES,那意味着,在指定的时间段完成后,动画就自动的从层上移除了。这个一般不用。假如你想要再次用这个动画时,你需要设定这个属性为 NO。这样的话,下次你在通过-set 方法设定动画的属 性时,它将再次使用你的动画,而非默认的动画。

* Speed：默认的值为 1.0.这意味着动画播放按照默认的速度。如果你改变这个值为 2.0,动画会用 2 倍的速度播放。 这样的影响就是使持续时间减半。如果你指定的持续时间为 6 秒,速度为 2.0,动画就会播放 3 秒钟---一半的 持续时间。

* BeginTime：这个属性在组动画中很有用。它根据父动画组的持续时间,指定了开始播放动画的时间。默认的是 0.0.组 动画在下篇文章中讨论“Animation Grouping”。

* TimeOffset：如果一个时间偏移量是被设定,动画不会真正的可见,直到根据父动画组中的执行时间得到的时间都流逝 了。

* RepeatCount：默认的是 0,意味着动画只会播放一次。如果指定一个无限大的重复次数,使用 MAXFLOAT。这个不应该和`repeatDration`属性一块使用。

* RepeatDuration：这个属性指定了动画应该被重复多久。动画会一直重复,直到设定的时间流逝完。它不应该和 `repeatCount `一起使用。 



####3、一个小例子

比如我们想实现一个类似心跳的缩放动画可以这么做，分为演员初始化、设定剧本、电影开拍三个步骤：

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view from its nib.
    [self initScaleLayer];
}
- (void)initScaleLayer
{
    //演员初始化
    CALayer *scaleLayer = [[CALayer alloc] init];
    scaleLayer.backgroundColor = [UIColor blueColor].CGColor;
    scaleLayer.frame = CGRectMake(60, 20, 50, 50);
    scaleLayer.cornerRadius = 10;
    [self.view.layer addSublayer:scaleLayer];
    //
    //设定剧本
    CABasicAnimation *scaleAnimation = [CABasicAnimation animationWithKeyPath:@"transform.scale"];
    scaleAnimation.fromValue = [NSNumber numberWithFloat:1.0];
    scaleAnimation.toValue = [NSNumber numberWithFloat:1.5];
    scaleAnimation.autoreverses = YES;
    scaleAnimation.fillMode = kCAFillModeForwards;
    scaleAnimation.repeatCount = MAXFLOAT;
    scaleAnimation.duration = 0.8;
    //
    //设置代理: 设置动画的代理，可以监听动画的执行过程，这里设置控制器为代理。
    scaleAnimation.delegate = self;
    //
    //开演
    [scaleLayer addAnimation:scaleAnimation forKey:@"scaleAnimation"];
}
//
//代理方法
- (void)animationDidStart:(CAAnimation *)anim{
    NSLog(@"开始执行动画");
}
- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag{
	//动画执行完毕，
	NSLog(@"动画执行完毕");
}
```

其他效果可以依葫芦画瓢轻松实现。想要实现不同的效果，最关键的地方在于`CABasicAnimation`对象的初始化方式中`keyPath`的设定。

* 在iOS中有以下几种不同的`keyPath`，代表着不同的效果：

	* transform.scale = 比例转换（x, y, z，value是NSNumber）
    * transform.rotation = 平面旋转（x, y, z）
    * transform.translation 按轴旋转（x, y, z，value是CGSize）
    * opacity (透明度，value是NSNumber0
    * margin
    * zPosition
    * backgroundColor  (背景颜色)
    * cornerRadius   (圆角value是NSNumber)
    * borderWidth (边框宽度)
    * bounds  (value是CGRect)
    * contents  (value是id,比如(id)image.CGImage)
    * contentsRect
    * frame 
    * hidden 
    * mask 
    * masksToBounds 
    * position  (相对于父图层的位置)
    * shadowColor  (value是(id)CGColor)
    * shadowOffset  (value是CGSize)
    * shadowOpacity  (value是NSNumber)
    * shadowRadius  (value是NSNumber)


> 附代码github地址：[CABasicAnimation](https://github.com/Vanbein/CABasicAnimation)

参考文章：

* [CABasicAnimation用法](http://www.cnblogs.com/bucengyongyou/archive/2012/12/20/2826590.html)

* [【原】iOSCoreAnimation动画系列教程（一）：CABasicAnimation【包会】](http://www.cnblogs.com/wengzilin/p/4250957.html)

* [iOS开发UI篇—核心动画(基础动画)](http://www.cnblogs.com/wendingding/p/3801157.html)





