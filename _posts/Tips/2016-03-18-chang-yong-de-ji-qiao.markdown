---
layout: post
date: 2016-03-18 23:52:02 +0800
title: iOS开发中一些常用的小知识或技巧
category: Tips
tags: iOS Tip
image: http://o7rxin1of.qnssl.com/images/head-800x400/-26.png
description: 本文将记录在学习iOS开发过程中所学到的小知识点，以及一些技巧，会保持更新。
toc: true
homepage: true
---

本文将记录在学习iOS开发过程中所学到的小知识点，以及一些技巧，会保持更新。

### 1. 将图片设置为圆形，或给button，label等设置圆角

{% highlight objc  %}
UIImageView *testImageView = [[UIImageView alloc] initWithFrame:CGRectMake(10, 10, 40, 40)] ;
// 设置圆角半径，一般要求图片是正方形，
// 若不是则需要将半径设置为宽和高比较大的值的一半即可
testImageView.layer.cornerRadius = self.testImageView.frame.size.width / 2 ;
testImageView.layer.masksToBounds = YES ; // 边界是否允许截取
//
// button label等设置圆角都是同样的方法
UIButton *testButton = [UIButton buttonWithType:UIButtonTypeRoundedRect];
testButton.frame = CGRectMake(50, 80, 80, 40);
testButton.layer.cornerRadius = 5; 
testButton.layer.masksToBounds = YES;
{% endhighlight %}

如果控件是在 stroyBoard 中，可以 `User Defined Runtime Attributes
`中按照下面一样设置即可

![storyBoard设置圆角](http://o7rxin1of.qnssl.com/images/2015/12/cornerRadius.png "设置圆角")


### 2. 根据内容计算高度，宽度

* 内容的自适应高度方法

	+ @param CGSize 规定文本显示的最大范围, 动态高度就设置为高度不限并固定宽度，若是动态宽度则设置宽度不限，高度固定
	+ @param options 按照何种设置来计算范围
	+ @param attributes 文本内容的一些属性,例如字体大小,字体类型等  (字体不一样,高度也不一样)
	+ @parma context 上下文 可以规定一些其他的设置 但是一般都是nil
     */    
    // 枚举值中的 " | "  意思是要满足所有的枚举值设置.

{% highlight objc  %} 
//例子
NSString *contentString = @"String! String!" ; //目标字符串
CGRect rect = [contentString boundingRectWithSize:CGSizeMake(tableView.bounds.size.width, MAXFLOAT) options:NSStringDrawingUsesLineFragmentOrigin | NSStringDrawingUsesFontLeading attributes:@{NSFontAttributeName :[UIFont systemFontOfSize:15]} context:nil] ;
{% endhighlight %}

### 3. 隐藏状态栏 修改状态栏风格

{% highlight objc  %}
-(UIStatusBarStyle)preferredStatusBarStyle 
{ 
    return UIStatusBarStyleLightContent;  // 暗背景色时使用
} 
// 是否隐藏状态栏
- (BOOL)prefersStatusBarHidden 
{ 
    return YES; 
}
{% endhighlight %}

### 4. 当有多个导航控制器时,一次设置多个导航控制器

{% highlight objc  %}
UINavigationBar *navBar = [UINavigationBar appearance] ;
// 所有导航条颜色都会改变 -- 一键设置
//navBar.barTintColor = [UIColor yellowColor] ;
[navBar setBackgroundImage:[UIImage imageNamed:@"bg_nav.png"] forBarMetrics:UIBarMetricsDefault] ;
{% endhighlight %}

### 5. Objective-C语法简写

#### (1). @
+ @() 代表NSNumber类型	

{% highlight objc  %}
@1; 等价于 [NSNumber numberWithInt:1];   
@('c'); 等价于 [NSNumber numberWithChar:'c']; 
{% endhighlight %}
	
+ @[] 代表数组NSArray类型
	
{% highlight objc  %}
@[@"1",@"2",@"3"]; //等价于 
[NSArray arrayWithObjects:@"1",@"2",@"3", nil];
{% endhighlight %}

+ @{}代表字典NSDictionary类型

{% highlight objc  %}
@{@"456":@"123"}; //等价于 
[NSDictionary dictionaryWithObject:@"123" forKey:@"456"];
{% endhighlight %}

#### (2).方法声明
* 返回值如果不写括号，编译器默认是id类型:

{% highlight objc  %}
-sendMessage;  //等价于
-(id)sendMessage;
{% endhighlight %}

* 参数如果不写类型默认也是id类型

{% highlight objc  %}
-(void)sendMessage:msg; //等价于
-(void)sendMessage:(id)msg;
{% endhighlight %}

* 有多参数时方法名和参数提示语可以为空

{% highlight objc  %}
-(void):msg1 :msg2; // 不建议这样简写，代码可读性降低
-(void)sendMessage:(id)msg1 message2:(id)msg2; 
{% endhighlight %}

#### (3).结构体

{% highlight objc  %}
CGRect rect = {1, 2};  //等价于
CGRect rect = {1, 2, 0, 0};
{% endhighlight %}

#### (4).三元条件表达式（针对字符串）

{% highlight objc  %}
NSString *string = inputString ?: @"default"; // 等价于
NSString *string = inputString ? inputString : @"default"; 
{% endhighlight %}

#### (5).小括号内联复合表达式

{% highlight objc  %}
RETURN_VALUE_RECEIVER = {( 
// Do whatever you want
 RETURN_VALUE;  // 返回值
)};
//
//so 我们可以引申为以下这种写法：
UIView *view = ({
        UIView *view = [[UIView alloc] initWithFrame:self.view.bounds];
        view.backgroundColor = [UIColor redColor];
        view.alpha = 0.8f;
        view;
    });
    [self.view addSubview:view];
//这样使得代码量增大时层次仍然能比较明确。
{% endhighlight %}

### 6. UIImage与字符串互转

{% highlight objc  %}
//图片转字符串  
-(NSString *)UIImageToBase64Str:(UIImage *) image  
{  
    NSData *data = UIImageJPEGRepresentation(image, 1.0f);  
    NSString *encodedImageStr = [data base64EncodedStringWithOptions:NSDataBase64Encoding64CharacterLineLength];  
    return encodedImageStr;  
}
//
//字符串转图片  
-(UIImage *)Base64StrToUIImage:(NSString *)_encodedImageStr  
{  
    NSData *_decodedImageData   = [[NSData alloc] initWithBase64Encoding:_encodedImageStr];  
    UIImage *_decodedImage      = [UIImage imageWithData:_decodedImageData];  
    return _decodedImage;  
}
{% endhighlight %}

### 7. 根据汉字字符串 获取该字符串的拼音 然后取得首字母

{% highlight objc  %} 
//第一种方法：先将汉字转换为 拼音 再获取首字母
//获取拼音首字母(传入汉字字符串, 返回大写拼音首字母)
/*
- (NSString *)firstCharactor:(NSString *)aString
{
    //转成了可变字符串
    NSMutableString *str = [NSMutableString stringWithString:aString];
    //先转换为带声调的拼音
    CFStringTransform((CFMutableStringRef)str,NULL, kCFStringTransformMandarinLatin,NO);
    //再转换为不带声调的拼音
    CFStringTransform((CFMutableStringRef)str,NULL, kCFStringTransformStripDiacritics,NO);
    //转化为大写拼音 
    NSString *pinYin = [str capitalizedString];
    //获取并返回首字母
    return [pinYin substringToIndex:1];
}
//
//第二种方法：
NSString *string = @"简书" ;
if ([string length])
{
    NSMutableString *mutableString = [NSMutableString stringWithString:string] ;
    /**
     *  由于此方法是在coreFoundation框架下,
     *  而我们平时所使用的类型都是Foundation框架下的,所以需要转换类型.
     *
     *  @param string#>    string 所需要转换的原字符#>
     *  @param range#>     range 所需要转换字符的范围.
     *      如果为0或者是NULL意思是所有字符都转换#>
     *  @param transform#> transform 转换方式#>
     *  @param reverse#>   reverse 如果为YES,返回原字符串;
     *      如果为NO,返回转换之后的字符串#>
     *
     *  @return return value description
     */
    // 将所有非英文的字符转换为拉丁字母,并且带声调和重音标识
    // __bridge :只改变当前对象的类型,但是不改变对象内存的管理权限
    CFStringTransform((__bridge CFMutableStringRef)mutableString , 0,kCFStringTransformToLatin , NO) ;
    // 去掉声调
    CFStringTransform((__bridge CFMutableStringRef)mutableString , 0,kCFStringTransformStripDiacritics , NO) ;
    // 每个单词的首字母大写 后再截取字符串
    NSString *str = [[mutableString capitalizedString] substringToIndex:1];
}
{% endhighlight %}

若需要得到汉字字符串的首字母缩写，只需对上述方法稍作修改即可。


### 8、调整导航栏自定义的 BarButtonItem 到屏幕边侧的间距

比如我们导航栏右侧有两个自定义的按钮，那就可以通过以下代码调整他们之间的间隔

{% highlight objc  %}
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

### 9、获得任意 view 相对于屏幕的 frame

{% highlight objc  %}
    CGRect frame = [[UIApplication sharedApplication].keyWindow convertRect:CGRectMake(0, 0, targetView.frame.size.width, targetView.frame.size.height) fromView:targetView];
    NSLog(@"\n\n targetView frame: x: %f   y: %f  \n\n width: %f   height: %f", frame.origin.x, frame.origin.y, frame.size.width, frame.size.height);
{% endhighlight %}

### 10、通过颜色来生成一个纯色图片

{% highlight objc  %}
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

### 11、设置 UINavigationBar 背景色，背景图片, 

下面的代码，可以设置 UINavigationBar 背景色和背景图片
   
{% highlight objc  %}
	//背景色
    [[UINavigationBar appearance] setBarTintColor:[UIColor colorWithRed:16/255.0 green:126/255.0 blue:219/255.0 alpha:1.0]];
	//背景图
    [[UINavigationBar appearance] setBackgroundImage:[self imageFromColor:[UIColor colorWithRed:16/255.0 green:126/255.0 blue:219/255.0 alpha:1.0]] forBarMetrics:UIBarMetricsDefault];
{% endhighlight %}

如果你发现实际的颜色比设置的颜色淡一点，那是因为导航栏默认带了半透明效果，我们可以通过代码或在storyboard中取消半透明效果。

{% highlight objc  %}
    [[UINavigationBar appearance] setTranslucent:NO];
{% endhighlight %}

![storyboard取消Translucent](http://o7rxin1of.qnssl.com/images/2016/03/Translucent.png)

但是如果导航栏下方刚好有一个颜色和导航栏背景色一样的View时，比如 SegmentFalut，

![SegmentFalut](images/2016/03/SegmentFalut.png)

此时会在导航栏下方出现一根黑线，比较难看，使用下面的代码可以去除导航栏下方的横线，

{% highlight objc  %}
	//#107cdb
	[[UINavigationBar appearance] setBackgroundImage:[self imageFromColor:[UIColor colorWithRed:16/255.0 green:126/255.0 blue:219/255.0 alpha:1.0]] forBarMetrics:UIBarMetricsDefault];
	[[UINavigationBar appearance] setShadowImage:[UIImage new]];
	//
	//如果设置了导航栏背景色，不想要 backgroundImage，则 backgroundImage 可以设置为 `[UIImage new]`，如下
	//[[UINavigationBar appearance] setBackgroundImage: [UIImage new] forBarMetrics:UIBarMetricsDefault];
{% endhighlight %}

![compare](http://o7rxin1of.qnssl.com/images/2016/03/compare.png)


### 12、随机生成一种颜色

我们可以使用标准库中的`drand48()`函数，它会随机生成一个0.0-1.0之间的double，因此我们只需要随机生成R, G, B三个值最后用UIColor的`colorWithRed:green:blue:alpha`方法创建UIColor，即可。

需要注意的是，`drand48`函数需要使用`srand48`来初始化随机数种子，示例代码如下：
{% highlight objc  %}
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


### 13、获得一款 APP 的所有图片资源

当我们想模仿一款app的时候，通过ipa包，可以拿到基本的图片，但是有一些个别的图片还是不能拿到的，可以通过一些工具插件来完成，推荐使用下面这款, github代码地址:[iOS-Images-Extractor](https://github.com/devcxm/iOS-Images-Extractor)


### 14、Xcode 查看谁改动的代码

如果我们源代码有版本控制，那么当程序出了问题或bug，找到问题代码所在行后，可以通过右键 - Show Blame for Line 查看是谁改动的代码。

![Show Balme for Line](http://o7rxin1of.qnssl.com/images/2016/03/ShowBalmeForLine.png)

![Show Balme for Line](http://o7rxin1of.qnssl.com/images/2016/03/ShowBalme.png)


### 15、屏幕截图并保存

{% highlight objc  %}
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

### 16、模糊效果

详见博文：[iOS实现模糊效果的几种方法](http://www.vanbein.com/posts/3rd-libraries/2016/04/06/blurView.html)































