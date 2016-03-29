---
layout: post
title: "UIBezierPath详解"
date: 2016-01-12 10:30:07 +0800
comments: true
categories: iOS高级
---

<!--more-->

使用`UIBezierPath`类可以创建基于矢量的路径，这个类在UIKit中。此类是`Core Graphics`框架关于path的一个封装。使用此类可以定义简单的形状，如椭圆或者矩形，或者有多个直线和曲线段组成的形状。

###一.Bezier Path 基础
   `UIBezierPath`对象是`CGPathRef`数据类型的封装。path如果是基于矢量形状的，都用直线和曲线段去创建。我们使用直线段去创建矩形和多边形，使用曲线段去创建弧（arc），圆或者其他复杂的曲线形状。每一段都包括一个或者多个点，绘图命令定义如何去诠释这些点。每一个直线段或者曲线段的结束的地方是下一个的开始的地方。每一个连接的直线或者曲线段的集合成为subpath。一个`UIBezierPath`对象定义一个完整的路径包括一个或者多个subpaths。
 
* 创建和使用一个path对象的过程是分开的。创建path是第一步，包含以下步骤：
	1. 创建一个`Bezier path`对象。
	2. 使用方法`moveToPoint:`去设置初始线段的起点。
	3. 添加line或者curve去定义一个或者多个subpaths。
	4. 改变`UIBezierPath`对象跟绘图相关的属性。
例如，我们可以设置`stroked path`的属性`lineWidth`和`lineJoinStyle`。也可以设置`filled path`的属性`usesEvenOddFillRule`。
 
   当创建path，我们应该管理path上面的点相对于原点（0，0），这样我们在随后就可以很容易的移动path了。为了绘制path对象，我们要用到stroke(描绘)和fill(填充)方法。这些方法在`current graphic context`下渲染path的line和curve段。
   
###二、使用UIBezierPath创建多边形---在path下面添加直线条形成多边形

多边形是一些简单的形状,这些形状是由一些直线线条组成，我们可以用`moveToPoint:` 和 `addLineToPoint:`方法去构建。

* 方法`moveToPoint:`设置我们想要创建形状的起点。
* 从这点开始，我们可以用方法`addLineToPoint:`去创建一个形状的线段。
* 我们可以连续的创建line，每一个line的起点都是先前的终点，终点就是指定的点。
 
下面的代码描述了如何用线段去创建一个五边形。第五条线通过调用`closePath`方法得到的，它连接了最后一个点（0，40）和第一个点（100，0）
> 说明：
> 
> * `closePath`方法不仅结束一个shape的subpath表述，它也在最后一个点和第一个点之间画一条线段，如果我们画多边形的话，这个一个便利的方法我们不需要去画最后一条线。
> * 注：这个五边形类要继承自UIView。

 
```objc
- (void)drawRect:(CGRect)rect  
{  
    UIColor *color = [UIColor redColor];  
    [color set]; //设置线条颜色  
    //  
    UIBezierPath* aPath = [UIBezierPath bezierPath];  
    aPath.lineWidth = 5.0;  
    // 
    aPath.lineCapStyle = kCGLineCapRound; //线条拐角  
    aPath.lineJoinStyle = kCGLineCapRound; //终点处理  
    // 
    // Set the starting point of the shape.  
    [aPath moveToPoint:CGPointMake(100.0, 0.0)];  
    //
    // Draw the lines  
    [aPath addLineToPoint:CGPointMake(200.0, 40.0)];  
    [aPath addLineToPoint:CGPointMake(160, 140)];  
    [aPath addLineToPoint:CGPointMake(40.0, 140)];  
    [aPath addLineToPoint:CGPointMake(0.0, 40.0)];  
    [aPath closePath];//第五条线通过调用closePath方法得到的  
  	//
    [aPath stroke];//Draws line 根据坐标点连线 
    //[aPath fill]; //填充 
}   
```
   
###三、使用UIBezierPath创建图形

如果是绘制矩形或者椭圆之类的特殊图形，可以不用像上面一样，`UIBezierPath`包含了几个特殊形状的类方法，我们可以直接使用：

```objc
// 创建矩形
+ (UIBezierPath *)bezierPathWithRect:(CGRect)rect
//
// 创建圆角矩形
+ (UIBezierPath *)bezierPathWithRoundedRect:(CGRect)rect cornerRadius:(CGFloat)cornerRadius
//
// 创建矩形内切圆
+ (UIBezierPath *)bezierPathWithOvalInRect:(CGRect)rect
//
// 创建弧形
//其中的参数分别指定：这段圆弧的中心，半径，开始角度，结束角度，是否顺时针方向。
+ (UIBezierPath *)bezierPathWithArcCenter:(CGPoint)center radius:(CGFloat)radius startAngle:(CGFloat)startAngle endAngle:(CGFloat)endAngle clockwise:(BOOL)clockwise
//
//下面举个例子,绘制矩形
- (void)drawRect:(CGRect)rect  
{  
    UIColor *color = [UIColor redColor];  
    [color set]; //设置线条颜色  
    // 
    UIBezierPath* aPath = [UIBezierPath bezierPathWithRect:CGRectMake(20, 20, 100, 50)];  
    //
    aPath.lineWidth = 5.0;  
    aPath.lineCapStyle = kCGLineCapRound; //线条拐角  
    aPath.lineJoinStyle = kCGLineCapRound; //终点处理  
    //
    [aPath stroke];  
}  
```

**除了这些闭合的特殊路径，也有一些方法用来添加子路径。**

```objc
// 添加直线
- (void)addLineToPoint:(CGPoint)point
//
// 添加弧形线段
- (void)addArcWithCenter:(CGPoint)center radius:(CGFloat)radius startAngle:(CGFloat)startAngle endAngle:(CGFloat)endAngle clockwise:(BOOL)clockwise
//
// 添加二阶贝塞尔曲线
- (void)addQuadCurveToPoint:(CGPoint)endPoint controlPoint:(CGPoint)controlPoint
//
// 添加三阶贝塞尔曲线
- (void)addCurveToPoint:(CGPoint)endPoint controlPoint1:(CGPoint)controlPoint1 controlPoint2:(CGPoint)controlPoint2
//
//一个二阶贝塞尔曲线例子
- (void)drawRect:(CGRect)rect  
{  
    UIColor *color = [UIColor redColor];  
    [color set]; //设置线条颜色  
    //
    UIBezierPath* aPath = [UIBezierPath bezierPath];  
    aPath.lineWidth = 5.0;  
    aPath.lineCapStyle = kCGLineCapRound; //线条拐角  
    aPath.lineJoinStyle = kCGLineCapRound; //终点处理  
    //
    [aPath moveToPoint:CGPointMake(20, 100)];  
    //
    [aPath addQuadCurveToPoint:CGPointMake(120, 100) controlPoint:CGPointMake(70, 0)];  
    //三阶
    //[aPath addCurveToPoint:CGPointMake(200, 50) controlPoint1:CGPointMake(110, 0) controlPoint2:CGPointMake(110, 100)];  
   	//   
    [aPath stroke];  
} 
```

###四、使用Core Graphics函数去修改path

`UIBezierPath`类只是CGPathRef数据类型和path绘图属性的一个封装。虽然通常我们可以用`UIBezierPath`类的方法去添加直线段和曲线段，`UIBezierPath`类还提供了一个属性`CGPath`，我们可以用来直接修改底层的`path data type`。如果我们希望用`Core Graphics `框架函数去创建path，则我们要用到此属性。
 
* 有两种方法可以用来修改和UIBezierPath对象相关的path。
	1. 可以完全的使用Core Graphics函数去修改path
	2. 也可以使用Core Graphics函数和UIBezierPath函数混合去修改。

第一种方法在某些方面相对来说比较容易。我们可以创建一个CGPathRef数据类型，并调用我们需要修改path信息的函数。

下面的代码就是赋值一个新的CGPathRef给UIBezierPath对象。

```objc
// Create the path data
CGMutablePathRef cgPath = CGPathCreateMutable();
CGPathAddEllipseInRect(cgPath, NULL, CGRectMake(0, 0, 300, 300));
CGPathAddEllipseInRect(cgPath, NULL, CGRectMake(50, 50, 200, 200));
// 
// Now create the UIBezierPath object
UIBezierPath* aPath = [UIBezierPath bezierPath];
aPath.CGPath = cgPath;
aPath.usesEvenOddFillRule = YES;
//
// After assigning it to the UIBezierPath object, you can release
// your CGPathRef data type safely.
CGPathRelease(cgPath);
```

如果我们使用`Core Graphics`函数和`UIBezierPath`函数混合方法，我们必须小心的移动path 信息在两者之间。因为`UIBezierPath`类拥有自己底层的`CGPathRef data type`，我们不能简单的检索该类型并直接的修改它。相反，我们应该生成一个副本，然后修改此副本，然后赋值此副本给CGPath属性，如下代码：

```objc
UIBezierPath*    aPath = [UIBezierPath bezierPathWithOvalInRect:CGRectMake(0, 0, 300, 300)];
// 
// Get the CGPathRef and create a mutable version.
CGPathRef cgPath = aPath.CGPath;
CGMutablePathRef  mutablePath = CGPathCreateMutableCopy(cgPath);
// 
// Modify the path and assign it back to the UIBezierPath object
CGPathAddEllipseInRect(mutablePath, NULL, CGRectMake(50, 50, 200, 200));
aPath.CGPath = mutablePath;
// 
// Release both the mutable copy of the path.
CGPathRelease(mutablePath);
```
 
###五、UIBezierPath与keyframeAnimation

我们可以使用贝塞尔曲线绘制一条路径，然后通过keyframe动画让动画沿着所设置的路径执行，下面举个例子说明：

```objc
- (void)viewDidLoad{
	[super viewDidLoad];
	//路径动画
    UIView *view8 = [[UIView alloc]initWithFrame:CGRectMake(100, 460, 30, 30)];
    [view8 setBackgroundColor:[UIColor purpleColor]];
    view8.layer.cornerRadius = 15;
    //创建路径
    UIBezierPath *movePath = [UIBezierPath bezierPath];
    [movePath moveToPoint:CGPointMake(100.0, 460.0)];
//    [movePath addQuadCurveToPoint:CGPointMake(200, 200) controlPoint:CGPointMake(300, 350)];
    [movePath addCurveToPoint:CGPointMake(300, 460) controlPoint1:CGPointMake(170, 360) controlPoint2:CGPointMake(240, 550)];
    CGPathRef cgPath = movePath.CGPath;
    CGMutablePathRef path = CGPathCreateMutableCopy(cgPath);
	//添加动画
    [view8.layer addAnimation:[self animationKeyframeWithPath:path duration:2.0 repeatTimes:MAXFLOAT] forKey:nil];
    [self.view addSubview:view8];
}
- (CAKeyframeAnimation *)animationKeyframeWithPath:(CGMutablePathRef)path duration:(float)duration repeatTimes:(float)repeatTimes{
    CAKeyframeAnimation *animation=[CAKeyframeAnimation animationWithKeyPath:@"position"];
    animation.path = path;
    animation.removedOnCompletion = NO;
    animation.fillMode = kCAFillModeForwards;
    animation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseIn];
    animation.autoreverses = NO;
    animation.duration = duration;
    animation.repeatCount = repeatTimes;   
    return animation;
}
```

* 一个在线生成三阶贝塞尔曲线的网站很不错，可以做可视化调整：[网站传送门](http://cubic-bezier.com/#.17,.74,.53,.43) 
* 附本文代码github地址：[UIBezierPath](https://github.com/XcodeTalk/UIBezierPath)
* 参考文章
	* [iOS的UIBezierPath类和贝塞尔曲线](http://www.yeolar.com/note/2015/03/25/ios-uibezierpath/)
	* [iOS开发 贝塞尔曲线UIBezierPath](http://www.cnblogs.com/moyunmo/p/3600091.html?utm_source=tuicool&utm_medium=referral)
 
 
 
 
 
 
 
 
 
 
 
   