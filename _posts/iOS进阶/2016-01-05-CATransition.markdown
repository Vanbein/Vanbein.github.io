---
layout: post
title: 动画CAAnimation系列--CATransition
category: iOS进阶
tags: iOS Animation
image: http://o7rxin1of.qnssl.com/images/head-800x400/-21.png
description: CATransition 类是 CAAnimation 的子类，用于做转场动画，能够为层提供移出屏幕和移入屏幕的动画效果。CATransition 实现了layer的过渡动画；也就是说它是一个控制layer的过渡动画的类；iOS 比 Mac OS X 的转场动画效果少一点，UINavigationController 就是通过 CATransition 实现了将控制器的视图推入屏幕的动画效果；我们可以通过 CATransition 来实现我们特定的过渡动画；也可以通过一个自定义的 CIFilter 实体来实现过渡动画。
toc: true
homepage: false
---


## CATransition 类
* CAAnimation的子类，用于做转场动画，能够为层提供移出屏幕和移入屏幕的动画效果。
* CATransition实现了layer的过渡动画。也就是说它是一个控制layer的过渡动画的类。
* iOS比Mac OS X的转场动画效果少一点，UINavigationController就是通过CATransition实现了将控制器的视图推入屏幕的动画效果
* 我们可以通过CATransition来实现我们特定的过渡动画。
* 也可以通过一个自定义的CIFilter实体来实现过渡动画。

## CATransition相关属性

#### startProgress & endProgress

##### startProgress

{% highlight objc  %}
@property float startProgress
{% endhighlight %}

定义过渡的开始点，开始点的值必须小于或者等于结束点。默认值为0.0。

##### endProgress

{% highlight objc  %}
@property float endProgress
{% endhighlight %}

定义过渡的结束点，结束点的值必须大于或者等于开始点。默认值为1.0。


 * startProgress & endProgress 这两个属性是float类型的。
 * 通过他们可以控制动画进行的过程，可以让动画从某个动画点上立即开始或结束，值在0.0到1.0之间。
 * endProgress要大于等于startProgress。
 * 比如:立方体转，可以设置`endProgress = 0.5`，让动画在整个动画的一半位置时停止(停止在旋转一半的状态)。
 
##### filter

{% highlight objc  %}
@property(nullable, strong) id filter;
{% endhighlight %}

* 这属性可以为动画添加一个可选的滤镜。
* 如果指定，那么指定的filter必须同时支持x和y，否则该filter将不起作用。
* 属性默认值为nil。
* 如果设置了filter，那么，为layer设置的type和subtype属性将被忽略。
* 该属性只支持iOS 5.0以及以后版本。

##### type

{% highlight objc  %}
@property(copy) NSString *type
{% endhighlight %}

* 指定预定义的过渡效果。
* 和type匹配使用，指定运动的方向。
* 默认为kCATransitionFade
* 如果指定了filter，那么该属性无效。

预定义的过渡效果：

{% highlight objc  %}
NSString * const kCATransitionFade; //淡出
NSString * const kCATransitionMoveIn; //覆盖原图
NSString * const kCATransitionPush; //推出
NSString * const kCATransitionReveal; //从底部显示
{% endhighlight %}

注意：
还有很多私有API效果，使用的时候要小心，可能会导致app审核不被通过

{% highlight objc  %}
//
//以下API效果可以安全使用
fade     //交叉淡化过渡(不支持过渡方向)
push     //新视图把旧视图推出去
moveIn   //新视图移到旧视图上面
reveal   //将旧视图移开,显示下面的新视图
cube //方块,立方体翻滚效果
suckEffect //三角,收缩效果，如一块布被抽走(不支持过渡方向)
rippleEffect //水波抖动,滴水效果(不支持过渡方向)
pageCurl //上翻页
pageUnCurl //下翻页
oglFlip //上下左右翻转效果
cameraIrisHollowOpen //镜头快门开, 相机镜头打开效果(不支持过渡方向)
cameraIrisHollowClose //镜头快门开, 相机镜头关上效果(不支持过渡方向)
//
//以下API效果请慎用，目前测试结论是下面的API设置后没动画效果
spewEffect //新版面在屏幕下方中间位置被释放出来覆盖旧版面.
genieEffect //旧版面在屏幕左下方或右下方被吸走, 显示出下面的新版面
unGenieEffect //新版面在屏幕左下方或右下方被释放出来覆盖旧版面.
twist //版面以水平方向像龙卷风式转出来.
tubey //版面垂直附有弹性的转出来.
swirl //旧版面360度旋转并淡出, 显示出新版面.
charminUltra //旧版面淡出并显示新版面.
zoomyIn //新版面由小放大走到前面, 旧版面放大由前面消失.
zoomyOut //新版面屏幕外面缩放出现, 旧版面缩小消失.
oglApplicationSuspend //像按”home” 按钮的效果.
{% endhighlight %}


##### subtype

{% highlight objc  %}
@property(nullable, copy) NSString *subtype;
{% endhighlight %}

* 指定预定义的过渡方向。
* 默认为nil。
* 如果指定了filter，那么该属性无效。

预定义的过渡方向为：

{% highlight objc  %}
NSString * const kCATransitionFromRight; //从右边开始
NSString * const kCATransitionFromLeft; //从左边开始
NSString * const kCATransitionFromTop; //从顶部开始
NSString * const kCATransitionFromBottom; //从底部开始
{% endhighlight %}

##### timingFunction

{% highlight objc  %}
@property(nullable, strong) CAMediaTimingFunction *timingFunction;
{% endhighlight %}

* 这是CAAnimation的一个属性，通过它设置动画的运动轨迹，用于变化起点和终点之间的插值计算,形象点说它决定了动画运行的节奏,比如是均匀变化(相同时间变化量相同)还是先快后慢,先慢后快还是先慢再快再慢。
* 默认是nil，表示线性

* 动画的开始与结束的快慢,有五个预置分别为(下同):

{% highlight objc  %}
kCAMediaTimingFunctionLinear            //线性,即匀速
kCAMediaTimingFunctionEaseIn            //先慢后快
kCAMediaTimingFunctionEaseOut           //先快后慢
kCAMediaTimingFunctionEaseInEaseOut     //先慢后快再慢
kCAMediaTimingFunctionDefault           //实际效果是动画中间比较快.
//
//示例
CATransition *transition = [CATransition animation];
transition.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionLinear];
{% endhighlight %}

## CATransition使用实例

对于有较多动画效果时，可以选择新建一个动画类，里面封装所有需要用到的动画效果。下面列举一个简单的例子：

{% highlight objc  %}
#pragma CATransition动画封装实现
- (void) transitionWithType:(NSString *) type WithSubtype:(NSString *) subtype ForView : (UIView *) view
{
    //创建CATransition对象
    CATransition *animation = [CATransition animation];
    //
    //设置运动时间
    animation.duration = DURATION;
    //
    //设置运动type
    animation.type = type;
    if (subtype != nil) {   
        //设置子类
        animation.subtype = subtype;
    }
    //
    //设置运动速度
    animation.timingFunction = UIViewAnimationOptionCurveEaseInOut;
    //
    [view.layer addAnimation:animation forKey:@"animation"];
}
{% endhighlight %}

对于有些动画效果，可直接用UIView的block回调实现动画的代码封装, 比如上下翻页，封装例子如下：

{% highlight objc  %}
#pragma UIView实现动画
- (void) animationWithView : (UIView *)view WithAnimationTransition : (UIViewAnimationTransition) transition
{
     [UIView animateWithDuration:DURATION animations:^{
         [UIView setAnimationCurve:UIViewAnimationCurveEaseInOut];
         [UIView setAnimationTransition:transition forView:view cache:YES];
     }];
}
{% endhighlight %}

于是，封装好之后，我们只需要调用封装方法即可，不必每次都写：

{% highlight objc  %}
//淡化效果
[self transitionWithType:kCATransitionFade WithSubtype:kCATransitionFromLeft ForView:self.view];
//波纹效果
//[self transitionWithType:@"rippleEffect" WithSubtype:kCATransitionFromRight ForView:self.view];
//右翻转效果
[self animationWithView:self.view WithAnimationTransition:UIViewAnimationTransitionFlipFromRight];
{% endhighlight %}

> 附本文代码的github地址：[CATransition](http://github.com/Vanbein/CATransition)

参考文章：

* [iOS开发之各种动画各种页面切面效果](http://www.cnblogs.com/ludashi/p/4160208.html)

