---
layout: post
title: 动画CAAnimation系列--CAKeyframeAnimation
category: iOS进阶
tags: iOS Animation
image: /images/head-800x400/-16.png
description: CAKeyframeAnimation 是 CApropertyAnimation 的子类，跟 CABasicAnimation 的区别是：CABasicAnimation 只能从一个数值(fromValue)变到另一个数值(toValue)，而CAKeyframeAnimation 会使用一个NSArray保存这些数值
homepage: false
toc: true
---

<!--more-->

### 一、简单介绍

`CAKeyframeAnimation` 是`CApropertyAnimation`的子类，跟`CABasicAnimation`的区别是：

* `CABasicAnimation`只能从一个数值(fromValue)变到另一个数值(toValue)，
* 而`CAKeyframeAnimation`会使用一个NSArray保存这些数值

##### 属性解析：

* values：就是上述的NSArray对象。里面的元素称为”关键帧”(keyframe)。动画对象会在指定的时间(duration)内，依次显示values数组中的每一个关键帧

* path：可以设置一个`CGPathRef\CGMutablePathRef`,让层跟着路径移动。path只对CALayer的`anchorPoint`和`position`起作用。**如果你设置了path，那么values将被忽略**

* keyTimes：可以为对应的关键帧指定对应的时间点,其取值范围为0到1.0,keyTimes中的每一个时间值都对应values中的每一帧.当keyTimes没有设置的时候,各个关键帧的时间是平分的

> 说明：`CABasicAnimation`可看做是最多只有2个关键帧的`CAKeyframeAnimation`

### 二、几个简单的例子

{% highlight objc linenos %}
//矩形轨迹动画
- (CAKeyframeAnimation *)rectAnimation{ 
    //1. 创建核心动画
    CAKeyframeAnimation *rectAnimation = [CAKeyframeAnimation animation];  
    //平移
    rectAnimation.keyPath = @"position";
    //2. 要执行什么动画
    NSValue *value1 = [NSValue valueWithCGPoint:CGPointMake(50, 50)];
    NSValue *value2 = [NSValue valueWithCGPoint:CGPointMake(50, 120)];
    NSValue *value3 = [NSValue valueWithCGPoint:CGPointMake(250, 120)];
    NSValue *value4 = [NSValue valueWithCGPoint:CGPointMake(250, 50)];
    NSValue *value5 = [NSValue valueWithCGPoint:CGPointMake(50, 50)];
    rectAnimation.values = @[value1, value2, value3, value4, value5];
    //2.1 设置动画执行完毕后不删除动画
    rectAnimation.removedOnCompletion = NO;
    //2.2 设置保存动画的最新状态
    rectAnimation.fillMode = kCAFillModeForwards;
    //2.3 设置动画执行时间
    rectAnimation.duration = 2.0;
    //2.4 设置动画节奏
    rectAnimation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseIn];
//    rectAnimation.keyTimes = @[@0.0, @0.1, @0.8, @0.9, @1.0];
    rectAnimation.repeatCount = MAXFLOAT;
    //2.5. 设置代理
    rectAnimation.delegate = self;    
    return rectAnimation;
}
//
//圆周动画，
- (CAKeyframeAnimation *)roundAnimation{
    //1. 创建核心动画
    CAKeyframeAnimation *roundAnimation = [CAKeyframeAnimation animation];
    //平移
    roundAnimation.keyPath = @"position";
    //2. 要执行什么动画
    //2.1 创建一条路径path
    CGMutablePathRef path = CGPathCreateMutable();
    //2.2 设置一个圆的路径
    CGPathAddEllipseInRect(path, NULL, CGRectMake(50, 200, 100, 100));
    roundAnimation.path=path;
    //2.3 有creat 就一定要release
    CGPathRelease(path);
    //3.1 设置动画执行完毕后不删除动画
    roundAnimation.removedOnCompletion = NO;
//    roundAnimation.autoreverses = NO;
    //3.2 设置保存动画的最新状态
    roundAnimation.fillMode = kCAFillModeForwards;
    //3.3 设置动画执行时间
    roundAnimation.duration = 3.0;
    roundAnimation.repeatCount = MAXFLOAT;
    //3.4 设置动画节奏
//    roundAnimation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionLinear];
    //    rectAnimation.keyTimes = @[@0.0, @0.1, @0.8, @0.9, @1.0];
    //3.5. 设置代理
    roundAnimation.delegate = self;
    return roundAnimation;
}
//
//抖动动画
- (CAKeyframeAnimation *)shakeAnimation{
    //1. 创建核心动画
    CAKeyframeAnimation *shakeAnimation = [CAKeyframeAnimation animation];
    shakeAnimation.keyPath = @"transform.rotation";//transform.rotation
    //设置动画时间
    shakeAnimation.duration = 0.2;
    //设置图标抖动弧度
    //把度数转换为弧度  度数/180*M_PI
    //#define DEGREES_TO_RADIANS(angle) ((angle) / 180.0 * M_PI)
    shakeAnimation.values = @[@(-DEGREES_TO_RADIANS(4)),@(DEGREES_TO_RADIANS(4)),@(-DEGREES_TO_RADIANS(4))];
    //设置动画的重复次数(设置为最大值)
    shakeAnimation.repeatCount = MAXFLOAT;
    shakeAnimation.fillMode = kCAFillModeForwards;
    shakeAnimation.removedOnCompletion = NO;
    shakeAnimation.delegate = self;
    return shakeAnimation;
}
//
//点击屏幕添加动画
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
	//矩形动画
    [self.view1.layer addAnimation:[self rectAnimation] forKey:@"rect"];
 	//圆周
    [self.view2.layer addAnimation:[self roundAnimation] forKey:@"round"];
	//抖动
    [self.view3.layer addAnimation:[self shakeAnimation] forKey:@"shake"];
}
{% endhighlight %}

> 附本文代码github地址[CAKeyframeAnimation](https://github.com/XcodeTalk/CAKeyframeAnimation)

* 参考文章：
	* [iOS开发UI篇—核心动画(关键帧动画)](http://www.cnblogs.com/wendingding/p/3801330.html)

