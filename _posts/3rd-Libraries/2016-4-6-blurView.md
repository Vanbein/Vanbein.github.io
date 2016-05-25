---
layout: post
date: 2016-04-06 20:08:02 +0800
title: iOS实现模糊效果的几种方法
category: 3rd-Libraries
tags: iOS Blur
image: /images/head-800x400/-54.png
description: iOS7后，半透明模糊效果得到了广泛的使用，所以iOS开发过程中经常需要用到半透明模糊效果，本文对比列举几种实现半透明模糊效果的方法，包括Core Image、vImage、BlurEffect，第三方库FXBlurView、GPUImage等。 
toc: true
homepage: true
---

## 一、苹果原生API

### 1、Core Image

`Core Image` 是苹果用来简化图片处理的框架；在 iOS 平台上，5.0 之后就出现了 `Core Image` 的 API。`Core Image` 的 API 被放在 `CoreImage.framework` 库中。不过直到iOS6.0才开始支持模糊。这个API调用起来很方便简洁。

在 iOS 和 OS X 平台上，Core Image 都提供了大量的滤镜（Filter），这也是 `Core Image` 库中比较核心的东西之一。按照官方文档记载，在 OS X 上有 120 多种 Filter，而在 iOS 上也有 90 多种。

下面是一段 Core Image 做模糊的示例代码：

{% highlight objc  %}
- (UIImage *)blurryImage:(UIImage *)image withMaskImage:(UIImage *)maskImage blurLevel:(CGFloat)blur {
    
    // 创建属性
    CIImage *ciImage = [[CIImage alloc] initWithImage:image];
    
    // 滤镜效果 高斯模糊
//    CIFilter *filter = [CIFilter filterWithName:@"CIGaussianBlur"];
//    [filter setValue:cimage forKey:kCIInputImageKey];
//    // 指定模糊值 默认为10, 范围为0-100
//    [filter setValue:[NSNumber numberWithFloat:blur] forKey:@"inputRadius"];
    
    /**
     *  滤镜效果 VariableBlur
     *  此滤镜模糊图像具有可变模糊半径。你提供和目标图像相同大小的灰度图像为它指定模糊半径
     *  白色的区域模糊度最高，黑色区域则没有模糊。
     */
    CIFilter *filter = [CIFilter filterWithName:@"CIMaskedVariableBlur"];
    // 指定过滤照片
    [filter setValue:ciImage forKey:kCIInputImageKey];
    CIImage *mask = [CIImage imageWithCGImage:maskImage.CGImage] ;
    // 指定 mask image
    [filter setValue:mask forKey:@"inputMask"];
    // 指定模糊值  默认为10, 范围为0-100
    [filter setValue:[NSNumber numberWithFloat:blur] forKey: @"inputRadius"];
    
    // 生成图片
    CIContext *context = [CIContext contextWithOptions:nil];
    // 创建输出
    CIImage *result = [filter valueForKey:kCIOutputImageKey];
    
    // 下面这一行的代码耗费时间内存最多,可以开辟线程处理然后回调主线程给imageView赋值
    //result.extent 指原来的大小size
//    NSLog(@"%@",NSStringFromCGRect(result.extent));
//    CGImageRef outImage = [context createCGImage: result fromRect: result.extent];
    
    CGImageRef outImage = [context createCGImage: result fromRect:CGRectMake(0, 0, 320.0 * 2, 334.0 * 2)];
    UIImage * blurImage = [UIImage imageWithCGImage:outImage];
    
    return blurImage;
}
{% endhighlight %}

![coreImage](/images/2016/04/coreImage.png)

> 更多的滤镜效果可以参见这个 [Filter官方列表](https://developer.apple.com/library/ios/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/filter/ci/CIGaussianBlur)

### 2、vImage

vImage 也是苹果推出的库，在 Accelerate.framework 中。

Accelerate这个framework主要是用来做数字信号处理、图像处理相关的向量、矩阵运算的库。我们可以认为我们的图像都是由向量或者矩阵数据构成的，Accelerate里既然提供了高效的数学运算API，自然就能方便我们对图像做各种各样的处理。

基于vImage我们可以根据图像的处理原理直接做模糊效果，或者使用现有的工具。UIImage+ImageEffects是个很好的图像处理库，看名字也知道是对UIImage做的分类扩展。这个工具被广泛地使用着，后面会做介绍。

下面是一段使用 vImage 实现模糊效果的代码：

{% highlight objc  %}
// 添加通用模糊效果
// image是图片，blur是模糊度
- (UIImage *)blurryImage:(UIImage *)image withBlurLevel:(CGFloat)blur
{
    if (image==nil)
    {
        NSLog(@"error:为图片添加模糊效果时，未能获取原始图片");
        return nil;
    }
    //模糊度,
    if (blur < 0.025f) {
        blur = 0.025f;
    } else if (blur > 1.0f) {
        blur = 1.0f;
    }
    
    //boxSize必须大于0
    int boxSize = (int)(blur * 100);
    boxSize -= (boxSize % 2) + 1;
    NSLog(@"boxSize:%i",boxSize);
    //图像处理
    CGImageRef img = image.CGImage;
    //需要引入#import <Accelerate/Accelerate.h>
    
    //图像缓存,输入缓存，输出缓存
    vImage_Buffer inBuffer, outBuffer;
    vImage_Error error;
    //像素缓存
    void *pixelBuffer;
    
    //数据源提供者，Defines an opaque type that supplies Quartz with data.
    CGDataProviderRef inProvider = CGImageGetDataProvider(img);
    // provider’s data.
    CFDataRef inBitmapData = CGDataProviderCopyData(inProvider);
    
    //宽，高，字节/行，data
    inBuffer.width = CGImageGetWidth(img);
    inBuffer.height = CGImageGetHeight(img);
    inBuffer.rowBytes = CGImageGetBytesPerRow(img);
    inBuffer.data = (void*)CFDataGetBytePtr(inBitmapData);
    
    //像数缓存，字节行*图片高
    pixelBuffer = malloc(CGImageGetBytesPerRow(img) * CGImageGetHeight(img));
    outBuffer.data = pixelBuffer;
    outBuffer.width = CGImageGetWidth(img);
    outBuffer.height = CGImageGetHeight(img);
    outBuffer.rowBytes = CGImageGetBytesPerRow(img);
    // 第三个中间的缓存区,抗锯齿的效果
    void *pixelBuffer2 = malloc(CGImageGetBytesPerRow(img) * CGImageGetHeight(img));
    vImage_Buffer outBuffer2;
    outBuffer2.data = pixelBuffer2;
    outBuffer2.width = CGImageGetWidth(img);
    outBuffer2.height = CGImageGetHeight(img);
    outBuffer2.rowBytes = CGImageGetBytesPerRow(img);
    //Convolves a region of interest within an ARGB8888 source image by an implicit M x N kernel that has the effect of a box filter.
    error = vImageBoxConvolve_ARGB8888(&inBuffer, &outBuffer2, NULL, 0, 0, boxSize, boxSize, NULL, kvImageEdgeExtend);
    error = vImageBoxConvolve_ARGB8888(&outBuffer2, &inBuffer, NULL, 0, 0, boxSize, boxSize, NULL, kvImageEdgeExtend);
    error = vImageBoxConvolve_ARGB8888(&inBuffer, &outBuffer, NULL, 0, 0, boxSize, boxSize, NULL, kvImageEdgeExtend);
    if (error) {
        NSLog(@"error from convolution %ld", error);
    }
    //    NSLog(@"字节组成部分：%zu",CGImageGetBitsPerComponent(img));
    //颜色空间DeviceRGB
    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
    //用图片创建上下文,CGImageGetBitsPerComponent(img),7,8
    CGContextRef ctx = CGBitmapContextCreate(
                                             outBuffer.data,
                                             outBuffer.width,
                                             outBuffer.height,
                                             8,
                                             outBuffer.rowBytes,
                                             colorSpace,
                                             CGImageGetBitmapInfo(image.CGImage));
    
    //根据上下文，处理过的图片，重新组件
    CGImageRef imageRef = CGBitmapContextCreateImage (ctx);
    UIImage *returnImage = [UIImage imageWithCGImage:imageRef];
    //clean up
    CGContextRelease(ctx);
    CGColorSpaceRelease(colorSpace);
    free(pixelBuffer);
    free(pixelBuffer2);
    CFRelease(inBitmapData);
    //CGColorSpaceRelease(colorSpace);   //多余的释放
    CGImageRelease(imageRef);
    return returnImage;
}
{% endhighlight %}


![vImage.png](/images/2016/04/vImage.png)

### 3、UIVisualEffectView

UIVisualEffectView只支持iOS 8以后的设备，所以有一定的局限性，但是使用起来非常简单，并且能通过代码或 storyboard 实现模糊效果，下面是一段实现模糊效果的示例代码，其中 effectWithStyle 有 Light、ExtraLight、dark 三种，如下：

{% highlight objc  %}
UIVisualEffectView *effectView = [[UIVisualEffectView alloc] initWithEffect:[UIBlurEffect effectWithStyle:UIBlurEffectStyleLight]];
effectView.frame = self.view.frame;
[self.view addSubview:effectView];
{% endhighlight %}

![blurEffect.png](/images/2016/04/blurEffect.png)

实现中间透明文字效果的代码为：

{% highlight objc  %}
- (void)configBlurEffect{
    
    // 原始图片 self.imageView
    // 为了更好的看到UIVisualEffectView的即时渲染效果添加平移手势 此处通过 storyBoard 添加
//    UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(panGesture:)];
//    [self.imageView addGestureRecognizer:pan];
    
    // 创建模糊View
    UIVisualEffectView *effectView = [[UIVisualEffectView alloc] initWithEffect:[UIBlurEffect effectWithStyle:UIBlurEffectStyleLight]];
    effectView.layer.cornerRadius = 10.0f;
    effectView.layer.masksToBounds = YES;
    effectView.frame = CGRectMake(80, 300, 160, 80);
    [self.view addSubview:effectView];
    
    
    UILabel *label = [[UILabel alloc] initWithFrame:effectView.bounds];
    label.text = @"Blur Effect";
    label.textAlignment = NSTextAlignmentCenter;
    label.font = [UIFont systemFontOfSize:20];
    //    [effectView.contentView addSubview:label];
    // 在创建的模糊View的上面再添加一个子模糊View
    UIVisualEffectView *subEffectView = [[UIVisualEffectView alloc] initWithEffect:[UIVibrancyEffect effectForBlurEffect:(UIBlurEffect *)effectView.effect]];
    
    subEffectView.frame = effectView.bounds;
    
    [effectView.contentView addSubview:subEffectView];
    
    [subEffectView.contentView addSubview:label];
}

- (IBAction)panGesture:(UIPanGestureRecognizer *)sender {
    
    CGPoint point = [sender translationInView:sender.view];
    sender.view.center = CGPointMake(sender.view.center.x + point.x,
                                      sender.view.center.y + point.y);
    [sender setTranslation:CGPointZero inView:sender.view];

}
{% endhighlight %}



## 二、第三方库/工具

### 1、FXBlurView

[FXBlurView](https://github.com/nicklockwood/FXBlurView) 是一个UIView的子类，效果和iOS7的背景实时模糊效果一样，但是支持到了 iOS 5.0。

FXBlurView 有两种模式，一种是 `static` 静态模糊：也就是只模糊一次，后面即使背景图片变化了，模糊效果也不会变化；另外就是 `dynamic` 动态模糊：这会实时的对背景图片进行模糊，是会不断变化的。

FXBlurView 的使用非常简单，看一下它的源码就明白了，一个示例代码如下：

{% highlight objc  %}
FXBlurView *blurView = [[FXBlurView alloc] init];
[blurView setFrame:CGRectMake(40.0, 60.0, 240.0, 240.0)];
[blurView setBackgroundColor:[UIColor whiteColor]];
//设置模式
self.blurView.dynamic = YES;
//设置模糊半径
self.blurView.blurRadius = 10.0;
{% endhighlight %}


![FXBlurView.png](/images/2016/04/FXBlurView.png)


### 2、GPUImage

[GPUImage](https://github.com/BradLarson/GPUImage) 是一个基于GPU图像和视频处理开源的iOS framework，适用面很广。

顺便说一下 GPUImage 的一种使用方法：

1. 在 github 上 clone 源码
2. 将它的 framework 整个文件夹拷贝到你的工程文件夹下
3. 将 framework 下的 `GPUImage.xcodeproj` 文件拖到你的 Xcode project 中
4. 到工程的 target 的 `Build Phases` 界面，添加 GPUImage 到 `Target Dependencies`
5. 继续将 libGPUImage.a 添加到Build Phases 界面的 `Link Binary With Libraries`
6. 在 `Link Binary With Libraries` 继续添加 GPUImage 的支持库：
	* CoreMedia
	* CoreVideo
	* OpenGLES
	* AVFoundation
	* QuartzCore
7. 最后，到工程的 `Build Settings` 设置 `Header Search Paths`，搜索 Header Search Paths 后在 `Header Search Paths` 一栏添加 framework 的头文件路径，不想手动输入可以将 framework 文件夹拖入即可自动生成路径，然后将后面的选项设置为 **recursive**
8. 在工程使用 `#import "GPUImage.h"` 即可开始使用了


![HeaderSearchPaths.png](/images/2016/04/HeaderSearchPaths.png)

如果要使用 GPUImage 实现高斯模糊，则非常简单，代码如下：

{% highlight objc  %}
- (UIImage *)blurryGPUImage:(UIImage *)image withBlurLevel:(CGFloat)blur {

    // 高斯模糊
    GPUImageGaussianBlurFilter * blurFilter = [[GPUImageGaussianBlurFilter alloc] init];
    blurFilter.blurRadiusInPixels = blur;
    UIImage *blurredImage = [blurFilter imageByFilteringImage:image];
    
    return blurredImage;
}
{% endhighlight %}


![GPUImage.png](/images/2016/04/GPUImage.png)

### 3、UIImage+ImageEffects

> Abstract: This is a category of UIImage that adds methods to apply blur and tint effects to an image. This is the code you’ll want to look out to find out how to use vImage to efficiently calculate a blur.
>
> 摘要：这是UIImage的category，增加方法来对图像进行模糊和着色效果。这就是你想要的如何有效利用 vImage实现模糊效果的代码


`UIImage+ImageEffects` 的模糊效果非常美观，对 `UIImage+ImageEffects` 进行修改后可以对图片进行局模糊;

`UIImage+ImageEffects` 提供了很多方法可以使用，常用的几个方法使用示例如下：

{% highlight objc  %}
// 通用模糊，默认模糊半径为20.0
self.blurView.image = [[UIImage imageNamed:@"WID-small"] blurImage];
// 局部模糊
self.partBlurView.image = [[UIImage imageNamed:@"WID-small"] blurImageAtFrame:CGRectMake(0.0, 0.0, 155.0*2 , 235.0*4.0)];
// 灰度锐化图
self.grayScaleView.image = [[UIImage imageNamed:@"WID-small"] grayScale];
{% endhighlight %}

![UIImage+ImageEffects.png](/images/2016/04/UIImage+ImageEffects.png)


> 附上代码的github地址 [BlurViewExample](https://github.com/Vanbein/BlurViewExample)

推荐阅读：

* [如何使用iOS 8的虚化效果](http://www.cocoachina.com/ios/20141009/9860.html)


