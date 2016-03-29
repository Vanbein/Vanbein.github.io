---
layout: post
title: UIView动画封装
category: iOS进阶
tags: iOS Tip Animation
image: /images/head-800x400/10.png
description: 虽然了解了核心动画的使用，但其实UIView本身对于基本动画和关键帧动画、转场动画都有相应的封装，在对动画细节没有特殊要求的情况下使用起来也要简单的多。可以说在日常开发中90%以上的情况使用UIView的动画封装方法都可以搞定，因此在熟悉了核心动画的原理之后还是有必要学习一下UIView中各类动画使用方法的。
homepage: false
---

虽然了解了核心动画的使用，但其实UIView本身对于基本动画和关键帧动画、转场动画都有相应的封装，在对动画细节没有特殊要求的情况下使用起来也要简单的多。可以说在日常开发中90%以上的情况使用UIView的动画封装方法都可以搞定，因此在熟悉了核心动画的原理之后还是有必要学习一下UIView中各类动画使用方法的。

###基础动画

####1、UIView动画（首尾）
* UIKit直接将动画集成到UIView类中，当内部的一些属性发生改变时，UIView将为这些改变提供动画支持
* 执行动画所需要的工作由UIView类自动完成，但仍要在希望执行动画时通知视图，为此需要将改变属性的代码放在`[UIView beginAnimations:nil context:nil]`和`[UIView commitAnimations]`之间

* 常见方法解析:

	+ (void)**setAnimationDelegate**:(id)delegate     设置动画代理对象，当动画开始或者结束时会发消息给代理对象
	
	+ (void)**setAnimationWillStartSelector**:(SEL)selector   当动画即将开始时，执行delegate对象的selector，并且把`beginAnimations:context:`中传入的参数传进selector
	
	+ (void)**setAnimationDidStopSelector**:(SEL)selector  当动画结束时，执行delegate对象的selector，并且把`beginAnimations:context:`中传入的参数传进selector
	
	+ (void)**setAnimationDuration**:(NSTimeInterval)duration   动画的持续时间，秒为单位
	
	+ (void)**setAnimationDelay**:(NSTimeInterval)delay  动画延迟delay秒后再开始
	
	+ (void)**setAnimationStartDate**:(NSDate *)startDate   动画的开始时间，默认为now
	
	+ (void)**setAnimationCurve**:(UIViewAnimationCurve)curve  动画的节奏控制
	
	+ (void)**setAnimationRepeatCount**:(float)repeatCount  动画的重复次数
	
	+ (void)**setAnimationRepeatAutoreverses**:(BOOL)repeatAutoreverses  如果设置为YES,代表动画每次重复执行的效果会跟上一次相反
	
	+ (void)**setAnimationTransition**:(UIViewAnimationTransition)transition **forView**:(UIView *)view **cache**:(BOOL)cache  设置视图view的过渡效果, transition指定过渡类型, cache设置YES代表使用视图缓存，性能较好

> #####UIView封装的动画与CALayer动画的对比
> * 使用UIView和CALayer都能实现动画效果，但是在真实的开发中，一般还是主要使用UIView封装的动画，而很少使用CALayer的动画。
> 
> #####CALayer核心动画与UIView动画的区别：
> * UIView封装的动画执行完毕之后不会反弹。即如果是通过CALayer核心动画改变layer的位置状态，表面上看虽然已经改变了，但是实际上它的位置是没有改变的。


####2、block动画
UIView封装的block动画有三种：

##### 方法1

* +(void)`animateWithDuration`:(NSTimeInterval)duration `delay`:(NSTimeInterval)delay `options`:(UIViewAnimationOptions)options `animations`:(void (^)(void))animations `completion`:(void (^)(BOOL finished))completion

* 参数解析:
	* duration：动画的持续时间
	* delay：动画延迟delay秒后开始
	* options：动画的节奏控制
	* animations：将改变视图属性的代码放在这个block中
	* completion：动画结束后，会自动调用这个block

#### 方法2、转场动画

* +(void)**transitionWithView**:(UIView *)view **duration**:(NSTimeInterval)duration **options**:(UIViewAnimationOptions)options **animations**:(void (^)(void))animations **completion**:(void (^)(BOOL finished))completion

* 参数解析:
	* duration：动画的持续时间
	* view：需要进行转场动画的视图
	* options：转场动画的类型
	* animations：将改变视图属性的代码放在这个block中
	* completion：动画结束后，会自动调用这个block

#### 方法3、转场动画

* +(void)**transitionFromView**:(UIView *)fromView **toView**:(UIView *)toView **duration**:(NSTimeInterval)duration **options**:(UIViewAnimationOptions)options **completion**:(void (^)(BOOL finished))completion

* 方法调用完毕后，相当于执行了下面两句代码：

```objc
// 添加toView到父视图
[fromView.superview addSubview:toView]; 
// 把fromView从父视图中移除
[fromView.superview removeFromSuperview];
```

* 参数解析:
	* duration：动画的持续时间
	* options：转场动画的类型
	* completion：动画结束后，会自动调用这个block

比如想要实现一个图片的移动动画：将图片移动到点击的位置；可通过下面的代码实现，下面的代码演示了通过**block**和**静态方法**实现动画控制的过程：

```objc
@interface KCMainViewController (){
    UIImageView *_imageView;
}
@end
@implementation KCMainViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    //
	//设置背景
    self.view.backgroundColor=[UIColor whiteColor];
    //
    //创建图像显示控件
    _imageView=[[UIImageView alloc]initWithImage:[UIImage imageNamed:@"icon"]];
    _imageView.frame = CGRectMake(0, 0, 40, 40);
    _imageView.layer.cornerRadius = 5;
    _imageView.center=CGPointMake(50, 150);
    [self.view addSubview:_imageView];
}
//
#pragma mark 点击事件
-(void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{
    UITouch *touch=touches.anyObject;
    CGPoint location= [touch locationInView:self.view];
    //方法1：block方式
    // 开始动画，UIView的动画方法执行完后动画会停留在重点位置，而不需要进行任何特殊处理
    //duration:执行时间
    // delay:延迟时间
    // options:动画设置，例如自动恢复、匀速运动等
    // completion:动画完成回调方法
    //
//    [UIView animateWithDuration:5.0 delay:0 options:UIViewAnimationOptionCurveLinear animations:^{
//        _imageView.center=location;
//    } completion:^(BOOL finished) {
//        NSLog(@"Animation end.");
//    }];
    // 
    //方法2：静态方法
    //开始动画
    [UIView beginAnimations:@"KCBasicAnimation" context:nil];
    [UIView setAnimationDuration:5.0];
    //改变UIView的属性
    _imageView.center=location;
    //开始动画
    [UIView commitAnimations];
}
```
####补充1 -- 弹簧动画效果

由于在iOS开发中弹性动画使用很普遍，所以在iOS7苹果官方直接提供了一个方法用于弹性动画开发，下面简单的演示一下：

```objc
#pragma mark 点击事件
-(void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{
    UITouch *touch=touches.anyObject;
    CGPoint location= [touch locationInView:self.view];
    //创建弹性动画
    // damping:阻尼，范围0-1，阻尼越接近于0，弹性效果越明显
    // velocity:弹性复位的速度
    [UIView animateWithDuration:5.0 delay:0 usingSpringWithDamping:0.1 initialSpringVelocity:1.0 options:UIViewAnimationOptionCurveLinear animations:^{
        _imageView.center=location; //CGPointMake(160, 284);
    } completion:nil];
}
```

####补充2 -- 动画设置参数

在动画方法中有一个option参数，UIViewAnimationOptions类型，它是一个枚举类型，动画参数分为三类，可以组合使用：

#####1.常规动画属性设置（可以同时选择多个进行设置）

* UIViewAnimationOptionLayoutSubviews：动画过程中保证子视图跟随运动。
* UIViewAnimationOptionAllowUserInteraction：动画过程中允许用户交互。
* UIViewAnimationOptionBeginFromCurrentState：所有视图从当前状态开始运行。
* UIViewAnimationOptionRepeat：重复运行动画。
* UIViewAnimationOptionAutoreverse ：动画运行到结束点后仍然以动画方式回到初始点。
* UIViewAnimationOptionOverrideInheritedDuration：忽略嵌套动画时间设置。
* UIViewAnimationOptionOverrideInheritedCurve：忽略嵌套动画速度设置。
* UIViewAnimationOptionAllowAnimatedContent：动画过程中重绘视图（注意仅仅适用于转场动画）。 
* UIViewAnimationOptionShowHideTransitionViews：视图切换时直接隐藏旧视图、显示新视图，而不是将旧视图从父视图移除（仅仅适用于转场动画）
* UIViewAnimationOptionOverrideInheritedOptions ：不继承父动画设置或动画类型。

#####2.动画速度控制（可从其中选择一个设置）

* UIViewAnimationOptionCurveEaseInOut：动画先缓慢，然后逐渐加速。
* UIViewAnimationOptionCurveEaseIn ：动画逐渐变慢。
* UIViewAnimationOptionCurveEaseOut：动画逐渐加速。
* UIViewAnimationOptionCurveLinear ：动画匀速执行，默认值。

#####3.转场类型（仅适用于转场动画设置，可以从中选择一个进行设置，基本动画、关键帧动画不需要设置）

* UIViewAnimationOptionTransitionNone：没有转场动画效果。
* UIViewAnimationOptionTransitionFlipFromLeft ：从左侧翻转效果。
* UIViewAnimationOptionTransitionFlipFromRight：从右侧翻转效果。
* UIViewAnimationOptionTransitionCurlUp：向后翻页的动画过渡效果。   
* UIViewAnimationOptionTransitionCurlDown ：向前翻页的动画过渡效果。   
* UIViewAnimationOptionTransitionCrossDissolve：旧视图溶解消失显示下一个新视图的效果。   
* UIViewAnimationOptionTransitionFlipFromTop ：从上方翻转效果。   
* UIViewAnimationOptionTransitionFlipFromBottom：从底部翻转效果。

###关键帧动画

从iOS7开始UIView动画中封装了关键帧动画，下面就来看一下如何使用UIView封装方法进行关键帧动画控制，这里实现前面关键帧动画部分对于落叶动画的控制。

```objc
-(void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{
    //UITouch *touch=touches.anyObject;
    //CGPoint location= [touch locationInView:self.view];
    // 关键帧动画
    // options:
    [UIView animateKeyframesWithDuration:5.0 delay:0 options: UIViewAnimationOptionCurveLinear| UIViewAnimationOptionCurveLinear animations:^{
        //第二个关键帧（准确的说第一个关键帧是开始位置）:从0秒开始持续50%的时间，也就是5.0*0.5=2.5秒
        [UIView addKeyframeWithRelativeStartTime:0.0 relativeDuration:0.5 animations:^{
            _imageView.center=CGPointMake(80.0, 220.0);
        }];
        //第三个关键帧，从0.5*5.0秒开始，持续5.0*0.25=1.25秒
        [UIView addKeyframeWithRelativeStartTime:0.5 relativeDuration:0.25 animations:^{
            _imageView.center=CGPointMake(45.0, 300.0);
        }];
        //第四个关键帧：从0.75*5.0秒开始，持所需5.0*0.25=1.25秒
        [UIView addKeyframeWithRelativeStartTime:0.75 relativeDuration:0.25 animations:^{
            _imageView.center=CGPointMake(55.0, 400.0);
        }];
    } completion:^(BOOL finished) {
        NSLog(@"Animation end.");
    }];
}
```

####补充 -- 动画设置参数

对于关键帧动画也有一些动画参数设置`options`，`UIViewKeyframeAnimationOptions`类型，和上面基本动画参数设置有些差别，关键帧动画设置参数分为两类，可以组合使用：

#####1.常规动画属性设置（可以同时选择多个进行设置）

* UIViewAnimationOptionLayoutSubviews：动画过程中保证子视图跟随运动。
* UIViewAnimationOptionAllowUserInteraction：动画过程中允许用户交互。
* UIViewAnimationOptionBeginFromCurrentState：所有视图从当前状态开始运行。
* UIViewAnimationOptionRepeat：重复运行动画。
* UIViewAnimationOptionAutoreverse ：动画运行到结束点后仍然以动画方式回到初始点。
* UIViewAnimationOptionOverrideInheritedDuration：忽略嵌套动画时间设置。
* UIViewAnimationOptionOverrideInheritedOptions ：不继承父动画设置或动画类型。

#####2.动画模式设置（同前面关键帧动画动画模式一一对应，可以从其中选择一个进行设置）

* UIViewKeyframeAnimationOptionCalculationModeLinear：连续运算模式。
* UIViewKeyframeAnimationOptionCalculationModeDiscrete ：离散运算模式。
* UIViewKeyframeAnimationOptionCalculationModePaced：均匀执行运算模式。
* UIViewKeyframeAnimationOptionCalculationModeCubic：平滑运算模式。
* UIViewKeyframeAnimationOptionCalculationModeCubicPaced：平滑均匀运算模式。

> 注意：核心动画里提过关键帧动画有两种形式，上面演示的是**属性值关键帧动画**，路径关键帧动画目前UIView还不支持。


###转场动画

从iOS4.0开始，UIView直接封装了转场动画，使用起来同样很简单。下面举一个图片浏览时的转场动画加以说明：

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    //
    //定义图片控件
    _imageView=[[UIImageView alloc]init];
    _imageView.frame=[UIScreen mainScreen].applicationFrame;
    _imageView.contentMode=UIViewContentModeScaleAspectFit;
    _imageView.image=[UIImage imageNamed:@"0.jpg"];//默认图片
    [self.view addSubview:_imageView];
    //添加手势
    //左滑
    UISwipeGestureRecognizer *leftSwipeGesture=[[UISwipeGestureRecognizer alloc]initWithTarget:self action:@selector(leftSwipe:)];
    leftSwipeGesture.direction=UISwipeGestureRecognizerDirectionLeft;
    [self.view addGestureRecognizer:leftSwipeGesture];
    //右滑
    UISwipeGestureRecognizer *rightSwipeGesture=[[UISwipeGestureRecognizer alloc]initWithTarget:self action:@selector(rightSwipe:)];
    rightSwipeGesture.direction=UISwipeGestureRecognizerDirectionRight;
    [self.view addGestureRecognizer:rightSwipeGesture];
}
//
#pragma mark 向左滑动浏览下一张图片
-(void)leftSwipe:(UISwipeGestureRecognizer *)gesture{
    [self transitionAnimation:YES];
}
//
#pragma mark 向右滑动浏览上一张图片
-(void)rightSwipe:(UISwipeGestureRecognizer *)gesture{
    [self transitionAnimation:NO];
}
// 
#pragma mark 转场动画
-(void)transitionAnimation:(BOOL)isNext{
    UIViewAnimationOptions option;
    if (isNext) {
        option=UIViewAnimationOptionCurveLinear|UIViewAnimationOptionTransitionFlipFromRight;
    }else{
        option=UIViewAnimationOptionCurveLinear|UIViewAnimationOptionTransitionFlipFromLeft;
    }
    //
    [UIView transitionWithView:_imageView duration:1.0 options:option animations:^{
        _imageView.image=[self getImage:isNext];
    } completion:nil];
}
//
#pragma mark 取得当前图片
-(UIImage *)getImage:(BOOL)isNext{
    if (isNext) {
        _currentIndex=(_currentIndex+1)%IMAGE_COUNT;
    }else{
        _currentIndex=(_currentIndex-1+IMAGE_COUNT)%IMAGE_COUNT;
    }
    NSString *imageName=[NSString stringWithFormat:@"%i.jpg",_currentIndex];
    return [UIImage imageNamed:imageName];
}
```

上面的转场动画演示中，其实仅仅有一个视图UIImageView做转场动画，每次转场通过切换UIImageView的内容而已。如果有两个完全不同的视图，并且每个视图布局都很复杂，此时要在这两个视图之间进行转场可以使用下面的方法进行两个视图间的转场

```objc
+ (void)transitionFromView:(UIView *)fromView toView:(UIView *)toView duration:(NSTimeInterval)duration options:(UIViewAnimationOptions)options completion:(void (^)(BOOL finished))completion NS_AVAILABLE_IOS(4_0)
```

需要注意的是默认情况下转出的视图会从父视图移除，转入后重新添加，可以通过`UIViewAnimationOptionShowHideTransitionViews`参数设置，设置此参数后转出的视图会隐藏（不会移除）转入后再显示。

* 注意：转场动画设置参数完全同基本动画参数设置；同直接使用转场动画不同的是使用UIView的装饰方法进行转场动画其动画效果较少，因为这里无法直接使用私有API。


> 附本文代码github地址：[UIViewAnimationEncapsulation](https://github.com/XcodeTalk/UIViewAnimationEncapsulation)


* 参考文章：
	* [iOS开发之让你的应用“动”起来](http://www.cocoachina.com/ios/20141022/10005.html)
