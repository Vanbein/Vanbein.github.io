---
layout: post
date: 2016-03-04 23:52:02 +0800
title: 3月的一些知识点笔记
category: Tips
tags: iOS Tip
image: /images/head-800x400/-4.png
description: 2016.3学习过程的一些新知识点笔记
toc: true
homepage: false
---


### 一、调整导航栏自定义的 BarButtonItem 到屏幕边侧的间距

比如我们导航栏右侧有两个自定义的按钮，那就可以通过以下代码调整他们之间的间隔

{% highlight objc linenos %}
	UIButton *rightbutton1;
	UIButton *rightbutton2;
    UIBarButtonItem *backButton1 = [[UIBarButtonItem alloc] initWithCustomView:rightbutton1];
    UIBarButtonItem *backButton2 = [[UIBarButtonItem alloc] initWithCustomView:rightbutton2];
    //
    //调整 rightBarButtonItems 到屏幕右侧的间距
    UIBarButtonItem *rightNegativeSpacer = [[UIBarButtonItem alloc]
                                       initWithBarButtonSystemItem:UIBarButtonSystemItemFixedSpace
                                       target:nil action:nil];
    rightNegativeSpacer.width = -1; 
    /**
     *  width为负数时，相当于 btn 向右移动 width 数值个像素，
     *  由于按钮本身和边界间距为 16 px，所以width设为 -1 时，间距正好调整为 15
     *  width为正数时，正好相反，相当于往左移动width数值个像素
     */
    UIBarButtonItem *rightNegativeSpacer2 = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemFixedSpace
                                            target:nil action:nil];
    rightNegativeSpacer2.width = 18; // rightButton1 和 rightButton2 的间隔默认为0
    //
    [self.navigationItem setRightBarButtonItems:[NSArray arrayWithObjects:rightNegativeSpacer, backButton1, rightNegativeSpacer2, backButton2, nil]];
{% endhighlight %}

### 二、获得任意 view 相对于屏幕的 frame

{% highlight objc linenos %}
    CGRect frame = [[UIApplication sharedApplication].keyWindow convertRect:CGRectMake(0, 0, targetView.frame.size.width, targetView.frame.size.height) fromView:targetView];
    NSLog(@"\n\n targetView frame: x: %f   y: %f  \n\n width: %f   height: %f", frame.origin.x, frame.origin.y, frame.size.width, frame.size.height);
{% endhighlight %}

### 三、通过颜色来生成一个纯色图片

{% highlight objc linenos %}
- (UIImage *)imageFromColor:(UIColor *)color{
    CGRect rect = CGRectMake(0, 0, 100, 100);
    UIGraphicsBeginImageContext(rect.size);
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextSetFillColorWithColor(context, [color CGColor]);
    CGContextFillRect(context, rect);
    UIImage *img = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return img;
}
{% endhighlight %}

### 四、设置 UINavigationBar 背景色，背景图片, 

下面的代码，可以设置 UINavigationBar 背景色和背景图片
   
{% highlight objc linenos %}
	//背景色
    [[UINavigationBar appearance] setBarTintColor:[UIColor colorWithRed:16/255.0 green:126/255.0 blue:219/255.0 alpha:1.0]];
	//背景图
    [[UINavigationBar appearance] setBackgroundImage:[self imageFromColor:[UIColor colorWithRed:16/255.0 green:126/255.0 blue:219/255.0 alpha:1.0]] forBarMetrics:UIBarMetricsDefault];
{% endhighlight %}

如果你发现实际的颜色比设置的颜色淡一点，那是因为导航栏默认带了半透明效果，我们可以通过代码或在storyboard中取消半透明效果。

{% highlight objc linenos %}
    [[UINavigationBar appearance] setTranslucent:NO];
{% endhighlight %}

![storyboard取消Translucent](/images/2016/03/Translucent.png)

但是如果导航栏下方刚好有一个颜色和导航栏背景色一样的View时，比如 SegmentFalut，

![SegmentFalut](images/2016/03/SegmentFalut.png)

此时会在导航栏下方出现一根黑线，比较难看，使用下面的代码可以去除导航栏下方的横线，

{% highlight objc linenos %}
	//#107cdb
	[[UINavigationBar appearance] setBackgroundImage:[self imageFromColor:[UIColor colorWithRed:16/255.0 green:126/255.0 blue:219/255.0 alpha:1.0]] forBarMetrics:UIBarMetricsDefault];
	[[UINavigationBar appearance] setShadowImage:[UIImage new]];
	//
	//如果设置了导航栏背景色，不想要 backgroundImage，则 backgroundImage 可以设置为 `[UIImage new]`，如下
	//[[UINavigationBar appearance] setBackgroundImage: [UIImage new] forBarMetrics:UIBarMetricsDefault];
{% endhighlight %}

![compare](/images/2016/03/compare.png)


### 五、随机生成一种颜色

我们可以使用标准库中的`drand48()`函数，它会随机生成一个0.0-1.0之间的double，因此我们只需要随机生成R, G, B三个值最后用UIColor的`colorWithRed:green:blue:alpha`方法创建UIColor，即可。

需要注意的是，`drand48`函数需要使用`srand48`来初始化随机数种子，示例代码如下：
{% highlight objc linenos %}
- (UIColor *)randomColor{    
    static BOOL seeded = NO;
    if (!seeded) {
        seeded = YES;
        srand48(time(0));
    }
    CGFloat r = (CGFloat)drand48();
    CGFloat g = (CGFloat)drand48();
    CGFloat b = (CGFloat)drand48();
    return [UIColor colorWithRed:r green:g blue:b alpha:1.0];
}
{% endhighlight %}

> demo的github地址：[随机生成一个UIColor](https://github.com/Vanbein/RandomColor)


### 六、获得一款 APP 的所有图片资源

当我们想模仿一款app的时候，通过ipa包，可以拿到基本的图片，但是有一些个别的图片还是不能拿到的，可以通过一些工具插件来完成，推荐使用下面这款, github代码地址:[iOS-Images-Extractor](https://github.com/devcxm/iOS-Images-Extractor)


### 七、Xcode 查看谁改动的代码

如果我们源代码有版本控制，那么当程序出了问题或bug，找到问题代码所在行后，可以通过右键 - Show Blame for Line 查看是谁改动的代码。

![Show Balme for Line](/images/2016/03/ShowBalmeForLine.png)

![Show Balme for Line](/images/2016/03/ShowBalme.png)


### 八、屏幕截图并保存

{% highlight objc linenos %}
//snapshotImage 这个方法效率比较低，
- (UIImage *)snapshotImage {
    UIGraphicsBeginImageContextWithOptions(self.view.bounds.size, self.view.opaque, 0);
    [self.view.layer renderInContext:UIGraphicsGetCurrentContext()];
    UIImage *snap = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return snap;
}
//iOS 7上UIView上提供了drawViewHierarchyInRect:afterScreenUpdates:来截图，速度比renderInContext:快15倍。
- (UIImage *)snapshotImageAfterScreenUpdates:(BOOL)afterUpdates {
    if ([self respondsToSelector:@selector(drawViewHierarchyInRect:afterScreenUpdates:)]) {
        return [self snapshotImage];
    }
    UIGraphicsBeginImageContextWithOptions(self.view.bounds.size, self.view.opaque, 0);
    [self.view drawViewHierarchyInRect:self.view.bounds afterScreenUpdates:afterUpdates];
    UIImage *snap = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return snap;
}
//保存截图到相册
- (void)saveImageToPhotos:(UIImage *)image{
    UIImageWriteToSavedPhotosAlbum(image, self, @selector(image:didFinishSavingWithError:contextInfo:), nil);
}
//状态回调
- (void)image:(UIImage *)image didFinishSavingWithError:(NSError *)error contextInfo:(void *)contextInfo{
    if (!error) {
        NSLog(@"\n\n保存成功\n\n");
    } else {
        NSLog(@"\n\n保存失败\n\n");
    }
}
{% endhighlight %}

### 九、图片高斯模糊，毛玻璃效果

详细请走传送门：[http://www.jianshu.com/p/6dd0eab888a6](http://www.jianshu.com/p/6dd0eab888a6)





