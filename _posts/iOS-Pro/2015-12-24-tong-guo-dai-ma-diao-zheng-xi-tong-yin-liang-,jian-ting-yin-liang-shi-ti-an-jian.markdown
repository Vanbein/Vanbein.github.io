---
layout: post
title: 通过代码调整系统音量，监听音量实体按键
category: iOS-Pro, tips
tags: iOS
image: /images/head-800x400/-14.png
description: 最近项目有个功能需要用到监听音量实体键，并能够通过滑动应用内的UISlider调整系统的音量，其中遇到不少问题，所以记录下这个学习过程。
homepage: false
---

最近项目有个功能需要用到监听音量实体键，并能够通过滑动应用内的UISlider调整系统的音量，其中遇到不少问题，所以记录下这个学习过程。

> 尽管`AVPlayer`和`AVPAudiolayer`这些类提供了音量调节功能，但这些音量控制属于App级别的控制。好处就是音量调节独立于系统音量，调节大小时不会影响系统音量。但有时候我们可能希望修改系统音量，以免在调节声音的时候，如果系统音量过小，App调节音量效果不明显。

##一、监听手机实体音量按键

* 和监听键盘弹出类似的，Apple提供了一个通知给我们，只要在合适的地方添加对这个通知的监听即可。

```objc
- (void)registeNotification{images
    //1.注册监听系统音量变化，记得在适当的地方移除监听
    [[NSNotificationCenter defaultCenter] addObserver:self
    								selector:@selector(volumeChanged:) name:@"AVSystemController_SystemVolumeDidChangeNotification" object:nil];
    	//让 UIApplication 开始响应远程的控制，必须添加，不然没效果
    	[[UIApplication sharedApplication] beginReceivingRemoteControlEvents];
}
- (void)volumeChanged:(NSNotification *)notification{
    //2.获取到当前音量
    float volume = [[[notification userInfo] objectForKey:@"AVSystemController_AudioVolumeNotificationParameter"] floatValue];
    //do something here
}
- (void) removeNotification{
    //3.移除监听
    [[NSNotificationCenter defaultCenter] removeObserver:self name:@"AVSystemController_SystemVolumeDidChangeNotification" object:nil];
}
	//结束远程的控制
	[[UIApplication sharedApplication] endReceivingRemoteControlEvents];
```

###虽然有效果，但是问题来了
####1.实体键按下的时候，会出现系统的"铃声"，而我想要显示"音量"

![出现系统铃声控制](/images/2015/12/systemRingVC.PNG "按下音量实体键系统显示铃声")

这个时候可以添加一句代码，即可让在这个页面时，按下音量实体键显示**音量**，而不是铃声

```objc
//使音量控制实体键响应“音量”而不是”铃声”
[[AVAudioSession sharedInstance] setActive:YES error:NULL];
```

![出现系统铃声控制](/images/2015/12/systemVolumeVC.PNG "按下音量实体键系统显示音量")


####2.出现了"音量"，但是因锁屏、进入后台等原因 ResignActive 重新激活后，又显示了"铃声"而不是"音量"

这是因为程序进入后台后，上述一行代码 [AVAudioSession sharedInstance] 又不再是 Active 状态，所以需要在`AppDelegate.m`里面重新设置，这样即可一直响应“音量”了，比如：

```objc AppDelegate.m
- (void)applicationWillEnterForeground:(UIApplication *)application {
      //使程序重新激活后，对音量实体键响应为“音量”而不是“铃声”
      [[AVAudioSession sharedInstance] setActive:YES error:NULL];
}
```

####3.我不想出现"音量"

这个时候就需要用到 `MPVolumeView` 了,它不仅可以不显示“音量”，还可以修改设置手机音量。

> 这个方法是苹果官方推荐的方法。MPVolumeView是`Media Player Framework`中的一个UI组件，直接包含了对系统音量和Airplay设备的音频镜像路由的控制功能。其中包含一个**MPVolumeSlider**的subview用来控制音量。这个`MPVolumeSlider`是一个私有类，我们无法手动创建此类，但这个类是UISlider的子类。MPVolumeView的使用很简单，只需要将其加入到一个父视图中，给予父视图合适的大小，再创建 MPVolumeView 示例，将其加入到父视图中即可，[苹果官方的文档](https://developer.apple.com/library/ios/documentation/MediaPlayer/Reference/MPVolumeView_Class/index.html)中有示例代码可以参考。

* 这个方法的缺点如下：

	* UI可定制的的程度低。 MPVolumeView只提供了有限的几个方法来定制其中的Slider和Route Button的样式，而且基本上只能靠换图片解决。如果你想把Slider操作换成Button或者其他的UI组件，那是不可能的。
	* 没有额外的音量控制API。 目前为止没有发现iOS的公开API中有可以直接操作系统音量的，所以修改系统音量只能使用这个UI组件。如果还想给UI加入手势操作来控制音量，这种直接使用MPVolumeView 是做不到的，那么有没有什么方法可以绕过这限制呢？办法还是有的。

* 实际上MPVolumeView没有提供任何接口来调节是否需要显示系统音量提示。但是我们发现一点：当MPVolumeView处在当前视图的层级之中时，系统就不会显示音量提示。那么事情好办了，我们只要确保两点：
	* MPVolumeView视图处在屏幕上看不见的地方，比如某个不透明视图的下方，或者本视图的非可见区域，一个常见的做法就是把该视图的frame设置为区域以外的地方，比如` volumeView.frame = CGRectMake(-1000, -100, 100, 100);`
	* 确保MPVolumeView视图的`hidden`属性值为**NO**。因为当hidden为YES时，同样会弹出提示。


于是想要隐藏"音量提示框"就可以通过添加以下代码实现：

```objc
//隐藏"音量提示框"
//注意使用之前需要添加`MediaPlayer.framework`
MPVolumeView *volumeView = [[MPVolumeView alloc] initWithFrame:CGRectMake(-100, -100, 100, 100)];
volumeView.hidden = NO;
[self.view addSubview:volumeView];
```

##代码实现调节系统音量，不通过实体按键

上面我们提到了MPVolumeView这个组件中，有一个subview来控制音量，即**MPVolumeSlider**。于是我们可以通过遍历 MPVolumeView 实例的subviews来得到MPVolumeSlider的实例，从而通过这个UI组件来操作系统音量。

* 步骤如下：
	* 1.通过创建一个MPVolumeView
	* 2.遍历找出MPVolumeSlider的实例
	* 3.可以通过`volumeSlider.value`这个属性来获取当前的系统音量。
	* 4.这个实例提供`setValue:animated:`方法来设置系统音量
	* 5.添加一个自定义的视图比如一个slider，通过这个视图来改变系统音量
	
具体的代码如下：

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
	//不显示“铃声”，显示“音量”
    [[AVAudioSession sharedInstance] setActive:YES error:NULL];
    //1. 获得 MPVolumeView 实例，
    MPVolumeView *volumeView = [[MPVolumeView alloc] initWithFrame:CGRectMake(-100, -100, 100, 100)];
    //volumeView.hidden = NO;
    //[self.view addSubview:volumeView]; //添加后不显示“音量”
    volumeViewSlider = nil;
    //2. 遍历MPVolumeView的subViews得到MPVolumeSlider
    for (UIView *view in [volumeView subviews]){
        if ([view.class.description isEqualToString:@"MPVolumeSlider"]){
            volumeViewSlider = (UISlider*)view;
            break;
        }
    }
    //3.获取系统音量
    float systemVolume = volumeViewSlider.value;
    //4.添加一个全局的 slider，滑动时同步改变系统音量
    VolSlider = [[UISlider alloc] initWithFrame:CGRectMake(30, 200, 300, 20)];
    VolSlider.value = systemVolume;  //初始值
    [VolSlider setMinimumValue:0.0]; //最小值
    [VolSlider setMaximumValue:1.0]; //最大值
    [VolSlider addTarget:self action:@selector(sliderValueChange:) forControlEvents:UIControlEventValueChanged]; //添加事件
    [self.view addSubview:VolSlider];
    //注册监听实体键按下事件
    [self registeNotification];
}
//
//VolSlider滑动事件
- (void)sliderValueChange:(UISlider *)slider{ 
    //得到当前用户设置的value
    float value = slider.value; 
    //改变系统音量大小，默认音量大小从 0.0 - 1.0
    [volumeViewSlider setValue:value animated:NO];
    //使setValue立即生效，
    [volumeViewSlider sendActionsForControlEvents:UIControlEventTouchUpInside];
}
//
//监听音量实体键按下后，响应事件
- (void)volumeChanged:(NSNotification *)notification{   
    //获取到当前系统音量
    float volume = [[[notification userInfo] objectForKey:@"AVSystemController_AudioVolumeNotificationParameter"] floatValue];
    //同步设置自定义视图的值
	[VolSlider setValue:volume animated:YES];
}
```

设置后即可使 VolSlider 与系统的音量提示框同步了，效果如下：

![同步效果](/images/2015/12/synchronizeControl.PNG "实体按键与自定义UI同步控制")


附上代码github链接：[changeSystemVolume](https://github.com/Vanbein/changeSystemVolume)



