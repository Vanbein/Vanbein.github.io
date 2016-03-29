---
layout: post
title: 动画CAAnimation系列--CAAnimationGroup
category: iOS进阶
tags: iOS Animation
image: /images/head-800x400/-8.png
description: CAAnimationGroup 是 CAAnimation 的子类，可以保存一组动画对象，将CAAnimationGroup 对象加入层后，组中所有动画对象可以同时并发运，默认情况下，一组动画对象是同时运行的，也可以通过设置动画对象的 beginTime 属性来更改动画的开始时间
homepage: false
---


###CAAnimationGroup简介

* CAAnimation的子类，可以保存一组动画对象，将CAAnimationGroup对象加入层后，组中所有动画对象可以同时并发运行

* 默认情况下，一组动画对象是同时运行的，也可以通过设置动画对象的`beginTime`属性来更改动画的开始时间

* 属性解析：
	* animations：用来保存一组动画对象的NSArray

###示例代码

```objc
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    //
    //平移
    CABasicAnimation *moveAnimation = [CABasicAnimation animation];
    moveAnimation.keyPath = @"transform.translation.y";
    moveAnimation.toValue = @300;
    //
    //缩放
    CABasicAnimation *scaleAnimation = [CABasicAnimation animation];
    scaleAnimation.keyPath = @"transform.scale";
    scaleAnimation.toValue = @(0.0);
    //
    //旋转
    CABasicAnimation *rotationAnimation = [CABasicAnimation animation];
    rotationAnimation.keyPath = @"transform.rotation";
    rotationAnimation.toValue = @(M_PI * 2);
    //
    //组动画
    CAAnimationGroup *groupAnimation = [CAAnimationGroup animation];
    groupAnimation.animations = @[moveAnimation, scaleAnimation, rotationAnimation];
	//    
    //设置组动画时间
    groupAnimation.duration = 1.5;
    groupAnimation.fillMode = kCAFillModeForwards;
    groupAnimation.removedOnCompletion = NO;
//    groupAnimation.repeatCount = MAXFLOAT;
    [self.testView.layer addAnimation:groupAnimation forKey:@"group"];
}
```

> 附本文代码github地址：[CAAnimationGroup](https://github.com/Vanbein/CAAnimationGroup)