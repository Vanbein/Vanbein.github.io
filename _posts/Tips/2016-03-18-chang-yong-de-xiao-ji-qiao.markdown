---
layout: post
date: 2016-03-18 23:52:02 +0800
title: iOS开发中一些常用的小知识或技巧
category: Tips
tags: iOS Tip
image: /images/head-800x400/-26.png
description: true本文将记录在学习iOS开发过程中所学到的小知识点，以及一些技巧，会保持更新。
toc: true
homepage: true
---

true本文将记录在学习iOS开发过程中所学到的小知识点，以及一些技巧，会保持更新。

###1. 将图片设置为圆形，或给button，label等设置圆角

```objc
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
```

如果控件是在 stroyBoard 中，可以 `User Defined Runtime Attributes
`中按照下面一样设置即可

![storyBoard设置圆角](/images/2015/12/cornerRadius.png "设置圆角")


###2. 根据内容计算高度，宽度

* 内容的自适应高度方法

	+ @param CGSize 规定文本显示的最大范围, 动态高度就设置为高度不限并固定宽度，若是动态宽度则设置宽度不限，高度固定
	+ @param options 按照何种设置来计算范围
	+ @param attributes 文本内容的一些属性,例如字体大小,字体类型等  (字体不一样,高度也不一样)
	+ @parma context 上下文 可以规定一些其他的设置 但是一般都是nil
     */    
    // 枚举值中的 " | "  意思是要满足所有的枚举值设置.

```objc 
//例子
NSString *contentString = @"String! String!" ; //目标字符串
CGRect rect = [contentString boundingRectWithSize:CGSizeMake(tableView.bounds.size.width, MAXFLOAT) options:NSStringDrawingUsesLineFragmentOrigin | NSStringDrawingUsesFontLeading attributes:@{NSFontAttributeName :[UIFont systemFontOfSize:15]} context:nil] ;
```

###3. 隐藏状态栏 修改状态栏风格

```objc
-(UIStatusBarStyle)preferredStatusBarStyle 
{ 
    return UIStatusBarStyleLightContent;  // 暗背景色时使用
} 
// 是否隐藏状态栏
- (BOOL)prefersStatusBarHidden 
{ 
    return YES; 
}
```

###4. 当有多个导航控制器时,一次设置多个导航控制器

```
UINavigationBar *navBar = [UINavigationBar appearance] ;
// 所有导航条颜色都会改变 -- 一键设置
//navBar.barTintColor = [UIColor yellowColor] ;
[navBar setBackgroundImage:[UIImage imageNamed:@"bg_nav.png"] forBarMetrics:UIBarMetricsDefault] ;
```

###5. Objective-C语法简写

####(1). @
+ @() 代表NSNumber类型	

```objc
@1; 等价于 [NSNumber numberWithInt:1];   
@('c'); 等价于 [NSNumber numberWithChar:'c']; 
```
	
+ @[] 代表数组NSArray类型
	
```objc
@[@"1",@"2",@"3"]; //等价于 
[NSArray arrayWithObjects:@"1",@"2",@"3", nil];
```

+ @{}代表字典NSDictionary类型

```objc
@{@"456":@"123"}; //等价于 
[NSDictionary dictionaryWithObject:@"123" forKey:@"456"];
```

####(2).方法声明
* 返回值如果不写括号，编译器默认是id类型:

```
-sendMessage;  //等价于
-(id)sendMessage;
```

* 参数如果不写类型默认也是id类型

```
-(void)sendMessage:msg; //等价于
-(void)sendMessage:(id)msg;
```

* 有多参数时方法名和参数提示语可以为空

```
-(void):msg1 :msg2; // 不建议这样简写，代码可读性降低
-(void)sendMessage:(id)msg1 message2:(id)msg2; 
```

####(3).结构体

```
CGRect rect = {1, 2};  //等价于
CGRect rect = {1, 2, 0, 0};
```

####(4).三元条件表达式（针对字符串）

```
NSString *string = inputString ?: @"default"; // 等价于
NSString *string = inputString ? inputString : @"default"; 
```

####(5).小括号内联复合表达式

```
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
```

###6. UIImage与字符串互转

```
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
```

###7. 根据汉字字符串 获取该字符串的拼音 然后取得首字母

```objc 
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
```

若需要得到汉字字符串的首字母缩写，只需对上述方法稍作修改即可。


































