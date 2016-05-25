---
layout: post
title: 一个简单的 loading 动画
category: Code
tags: Animation
image: /images/head-800x400/-11.png
description: 使用 Core Animation 实现的一个简单的 loading 动画。
homepage: false
toc: true
---

### 一、概览

使用 Core Animation 实现的一个简单的 loading 动画。

效果预览图：

![loading](/images/2016/04/loading.gif)

首先来分析下这个动画的效果，明显它分为两部分动画

1. 图片上下跳动，并且在上升过程中平面旋转 180 度
2. loading... 这几个字的出现动画，从左到右依次出现， 并且每个字有一个从上落下渐入动画
3. 图片上下跳动两次，旋转 360 度的时间，等于 loading... 文字完成一次动画的时间，约1.6s

![resolution](/images/2016/04/resolution.png)


### 二、图片上下跳动旋转动画

#### 1. 创建 imageView

创建一个 imageView，由于最开始从上往下开始动画，所以初始位置在动画效果的顶部

{% highlight objc %}
self.imageView = [[UIImageView alloc] initWithFrame:CGRectMake(25, 2, 18, 20)];
self.imageView.image = [UIImage imageNamed:@"LOADING"];
self.imageView.contentMode = UIViewContentModeScaleAspectFill;
self.imageView.hidden = YES;//默认隐藏
[self addSubview:self.imageView];
{% endhighlight %}

#### 2. 配置动画

由于图片的动画效果比较简单，所以只用 `CABasicAnimation` 结合 `CAAnimationGroup` 即可实现。

初始我们或许会认为图片只需要不断的做“下落，上升并旋转180度”这样一个 Group 动画即可，但是如果这样做的实际效果是图片从底部上升到顶部旋转180度后，再下落时图层会恢复成初始心态，即没有旋转180度的形态。虽然设置了 `fillMode = kCAFillModeForwards` 和 `removedOnCompletion = NO`，但是这样只是让图层保持了动画结束时的状态，图层的真正状态是没改变的，所以第二次开始动画时又会恢复到初始状态。

因此，只能将完成一次 360 度旋转动画作为一个 Group，那么这个动画过程可以分解为：

1. 图片下落到底部
2. 图片上升到顶部，并且这个过程中旋转 180 度
3. 图片下落到底部
4. 图片上升到顶部，并且这个过程中再旋转 180 度

落实到代码上后，关键代码如下：

{% highlight objc  %}
- (CAAnimationGroup *)imageMoveAnimationForImageView:(UIImageView *)imageView{
    
    /* 向下移动 */
    //imageView.layer.position = (34.0, 12.0)
    CABasicAnimation *moveDownAnimation = [CABasicAnimation animationWithKeyPath:@"position"];
    moveDownAnimation.fromValue = [NSValue valueWithCGPoint:imageView.layer.position];
    moveDownAnimation.toValue = [NSValue valueWithCGPoint:CGPointMake(imageView.layer.position.x, 43)];
    moveDownAnimation.duration = 0.4f;
    moveDownAnimation.beginTime = 0.0f;
    moveDownAnimation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseIn];
    
    //2.1 向上移动
    CABasicAnimation *moveUpAnimation = [CABasicAnimation animationWithKeyPath:@"position"];
    moveUpAnimation.fromValue = [NSValue valueWithCGPoint:CGPointMake(34.0, 45.0)];
    moveUpAnimation.toValue = [NSValue valueWithCGPoint:imageView.layer.position];
    moveUpAnimation.duration = 0.4f;
    moveUpAnimation.beginTime = 0.4f;
    moveUpAnimation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseOut];
    
    //2.2 旋转到180
    CABasicAnimation *rotateAnimation = [CABasicAnimation animationWithKeyPath:@"transform.rotation.z"];
    rotateAnimation.fromValue = [NSNumber numberWithFloat:0];
    rotateAnimation.toValue = [NSNumber numberWithFloat: M_PI];
    rotateAnimation.fillMode = kCAFillModeForwards;
    rotateAnimation.removedOnCompletion = NO;
    rotateAnimation.duration = 0.4;
    rotateAnimation.beginTime = 0.4f;
    rotateAnimation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseOut];
 
    //省略了下落，上升并从180度旋转到360度的第二段动画
 
    CAAnimationGroup *group = [CAAnimationGroup animation];
    group.animations = @[moveDownAnimation, moveUpAnimation, rotateAnimation, moveDownAnimation_2, moveUpAnimation_2, rotateAnimation_2];
    group.duration = 1.6 + 0.05;
    group.repeatCount = MAXFLOAT;
    group.removedOnCompletion = NO;
    group.fillMode = kCAFillModeForwards;
    group.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionLinear];
//    group.delegate = self;
    
    [group setValue:@"imageAnimation" forKey:@"name"];
    
    return group;
}
{% endhighlight %}


### 三、文字出现动画

试想一下，如果这是一个 Label，`Label.text = @"Loading..."`，那么想要实现这个动画就很困难，于是我们可以考虑把"Loading..."这 10 个字符分别放在了 10 个 Label里面，让这些 Label 依次做动画即可实现同样的效果。

#### 1. 创建 Label

根据每个字符的 width，依次创建 Lable，一个字符为一个 Label

{% highlight objc  %}
NSArray *textArray = @[@"L", @"o", @"a", @"d", @"i", @"n", @"g",@".", @".", @"."];
UIColor *textTintColor;
if (!self.textTintColor) {
    textTintColor = [UIColor whiteColor]; //默认白色
} else {
    textTintColor = self.textTintColor;
}
self.labelArray = [NSMutableArray arrayWithCapacity:10];

//记录字符串的总长度，用以设定下一个 Label 的位置
CGFloat totalWidth = 0.0;

for (int i = 0; i < 10; i ++) {
    
    //获取字符宽度
    NSString *contentString = textArray[i]; //目标字符串
    CGRect rect = [contentString boundingRectWithSize:CGSizeMake(MAXFLOAT, 15.0) options:NSStringDrawingUsesLineFragmentOrigin | NSStringDrawingUsesFontLeading attributes:@{NSFontAttributeName :[UIFont systemFontOfSize:14]} context:nil] ;
    
    CGFloat charWidth = CGRectGetWidth(rect);

    UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(3 + totalWidth, 55, charWidth, 15)];
    label.text = textArray[i];
    label.font = [UIFont systemFontOfSize:14.0];
    label.textAlignment = NSTextAlignmentCenter;
    label.textColor = textTintColor;
    label.layer.opacity = 0.0f;
    
    totalWidth += charWidth;
    
    [self.labelArray addObject:label];
    [self addSubview:label];
}
{% endhighlight %}

#### 2. 配置动画


把文字动画进行分解后，得到整个动画过程如下：

1. 每个字依次的出现，全部都出现后文字消失
2. 每个字的出现动画有落下、渐入效果

同图片的动画类似，使用 `CAAnimationGroup` 即可，如下：

{% highlight objc  %}
- (CAAnimationGroup *)labelAppearAnimationForLabel:(UILabel *)label delay:(CGFloat)delay{
    
    //1. 透明度渐变
    CABasicAnimation *opacityAnimation = [CABasicAnimation animationWithKeyPath:@"opacity"];
    opacityAnimation.fromValue = [NSNumber numberWithFloat:0.0];
    opacityAnimation.toValue = [NSNumber numberWithFloat:1.0];
    opacityAnimation.duration = 0.25;
    opacityAnimation.beginTime = 0.0f + delay;
    opacityAnimation.removedOnCompletion = NO;
    opacityAnimation.fillMode = kCAFillModeForwards;
    opacityAnimation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseOut];
    
    //2. 向下移动
    CABasicAnimation *moveDownAnimation = [CABasicAnimation animationWithKeyPath:@"position"];
    moveDownAnimation.fromValue = [NSValue valueWithCGPoint:CGPointMake(label.layer.position.x, 61)];
    moveDownAnimation.toValue = [NSValue valueWithCGPoint:label.layer.position];
    moveDownAnimation.fillMode = kCAFillModeForwards;
    moveDownAnimation.duration = 0.15f;
    moveDownAnimation.beginTime = 0.0 + delay;
    moveDownAnimation.removedOnCompletion = NO;
    moveDownAnimation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseIn];
    
    CAAnimationGroup *group = [CAAnimationGroup animation];
    group.animations = @[opacityAnimation, moveDownAnimation];
    group.duration = 1.6 + 0.05;
    group.repeatCount = MAXFLOAT;
    group.removedOnCompletion = NO;
    group.autoreverses = NO;
    group.fillMode = kCAFillModeForwards;
//    group.delegate = self;
    
    return group;
 }
{% endhighlight %}

> 附代码地址：[LoadingAnimation](https://github.com/Vanbein/LoadingAnimation)


