---
layout: post
title: UIApplication详解
category: iOS基础
tags: iOS
image: /images/head-800x400/-22.png
description: 总结一下和 UIApplication 相关的知识点.
homepage: false
---

总结一下和 UIApplication 相关的知识点

#####什么是UIApplication?
* UIApplication对象是应用程序的象征
* 每个app有且只有一个UIApplication对象，而且是单例的
* 当程序启动的时候通过调用UIApplicationMain方法得到的。
* 程序运用过程中可以通过`[UIApplication sharedApplication]`方法得到。
* 一个iOS程序启动后创建的第一个对象就是UIApplication对象images
* 利用UIApplication对象，能进行一些应用级别的操作.

UIApplication对象的**主要任务**:
* 处理用户事件的处理路径，例如分发一个UIEvent到另外一个对象去处理。
* UIApplication对象持有众多的UIWindow对象，因此可以组织app的展示。
* UIApplication对象还能处理一些资源，例如通过openURL:打开邮箱客户端或者设置界面等。

###一、获得UIApplication对象

```objc
[UIApplication sharedApplication]
```

###二、获得UIApplicationDelegate对象

```objc
[[UIApplication sharedApplication] delegate]
```

###三、获得UIWindow对象

```objc
//UIWindow数组
[[UIApplication sharedApplication] windows];   
//UIWindow数组中最后调用makeKeyAndVisible方法的UIWindow对象
[[UIApplication sharedApplication] keyWindow]; 
```
###四、常用属性

#####1. 设置应用程序图标右上角的红色提醒数字

```objc
@property(nonatomic) NSInteger applicationIconBadgeNumber;
```
示例代码如下：

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    //1.获得UIApplication
    UIApplication *app = [UIApplication sharedApplication];
    //2.更改应用图标的前提是需要先注册
    //注册
    UIUserNotificationSettings *setting = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeBadge categories:nil];
    [app registerUserNotificationSettings:setting];
    //3.设置应用的图标的10
    app.applicationIconBadgeNumber = 10;
```

#####2. 设置联网指示器的可见性

```objc
@property(nonatomic,getter=isNetworkActivityIndicatorVisible) BOOL networkActivityIndicatorVisible;
```

联网指示器是显示在手机状态栏的，当我们在使用网络时，就会在状态栏显示一个动态的小圆圈（菊花）指示正在使用互联网

示例代码如下：

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    //1.获得UIApplication
    UIApplication *app = [UIApplication sharedApplication];
	//2.设置是否显示 联网指示器
    app.networkActivityIndicatorVisible = YES;
```

#####3. openURL:

* UIApplication有个功能十分强大的openURL:方法

```objc
- (BOOL)openURL:(NSURL*)url;
//
// openURL:方法的部分功能有
// 打电话
UIApplication *app = [UIApplication sharedApplication];
[app openURL:[NSURL URLWithString:@"tel://10086”]];
//
// 发短信
[app openURL:[NSURL URLWithString:@"sms://10086"]];
//
// 发邮件
[app openURL:[NSURL URLWithString:@"mailto://12345@qq.com"]];
//
// 打开一个网页资源
[app openURL:[NSURL URLWithString:@"http://www.baidu.com"]];
//
//打开URL资源: 打开设置界面配置通知设置
[app openURL:[NSURL URLWithString:UIApplicationOpenSettingsURLString]];
```

#####4. 管理状态栏
* 从iOS7开始，系统提供了2种管理状态栏的方式
	* 通过UIViewController管理（每一个UIViewController都可以拥有自己不同的状态栏）
	* 通过UIApplication管理（一个应用程序的状态栏都由它统一管理）

* iOS7开始，默认情况下，状态栏都是由UIViewController管理的.

* UIViewController实现下列方法就可以轻松管理状态栏的可见性和样式

```objc
// 状态栏的样式
- (UIStatusBarStyle)preferredStatusBarStyle; 
// 状态栏的可见性
(BOOL)prefersStatusBarHidden;
```

* 如果想利用UIApplication来管理状态栏

	* 1, 在`project target`的`Info.plist`中，插入一个新的key，名字为`View controller-based status bar appearance`，并将其值设置为NO。
	* 2, 在代码中设置状态栏样式
	
![利用UIApplication来管理状态栏](/images/2015/12/UIApplicationManageState.png "修改Info.plist")


```objc
[[UIApplication sharedApplication] setStatusBarStyle:UIStatusBarStyleLightContent];
```


###五、控制和处理UIEvent

```objc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
	//分发一个event到另外一个对象去处理
	[[UIApplication sharedApplication] sendAction:@selector(action: forEvent:) to:[CustomHandler sharedCustomHandler] from:self forEvent:event];
}
//
//开始忽略Event
[[UIApplication sharedApplication] beginIgnoringInteractionEvents]; 
//
//...中间调用动画等操作
//
//结束忽略Event
[[UIApplication sharedApplication] endIgnoringInteractionEvents];   
```

###六、晃动是否有撤销或者重做动作

```objc
[UIApplication sharedApplication].applicationSupportsShakeToEdit = YES;  
```

###七、配置通知设置

```objc
//注册远程推送通知
UIUserNotificationType  types = UIUserNotificationTypeBadge | UIUserNotificationTypeSound | UIUserNotificationTypeAlert;
UIUserNotificationSettings  *mySettings  = [UIUserNotificationSettings settingsForTypes:types categories:nil];
[[UIApplication sharedApplication] registerUserNotificationSettings:mySettings]; 
//
//注册
[[UIApplication sharedApplication] registerForRemoteNotifications];
//注销
[[UIApplication sharedApplication] unregisterForRemoteNotifications];
//
NSDate *date = [NSDate dateWithTimeIntervalSinceNow:10];
UILocalNotification *localNotif = [[UILocalNotification alloc] init];
localNotif.fireDate = date;  //时间
localNotif.timeZone = [NSTimeZone localTimeZone]; //时区
localNotif.repeatInterval = NSCalendarUnitMinute; //间隔
localNotif.soundName = UILocalNotificationDefaultSoundName; //声音
localNotif.alertBody = @"Local Test";   //通知内容
localNotif.applicationIconBadgeNumber = 1;  //数字标示
localNotif.userInfo = @{@"key":@"test"};    //info
//注册通知
[[UIApplication sharedApplication] scheduleLocalNotification:localNotif]; 
//立即通知
[[UIApplication sharedApplication] presentLocalNotificationNow:localNotif]; 
//取消所有通知
[[UIApplication sharedApplication] cancelAllLocalNotifications]; 
//取消特定的通知
[[UIApplication sharedApplication] cancelLocalNotification:localNotif];
//所有的通知
NSArray *arr = [[UIApplication sharedApplication] scheduledLocalNotifications];  
```

###八、APP后台运行相关

```objc
//app状态 
[[UIApplication sharedApplication] applicationState]; 
//设置后台运行时间
[[UIApplication sharedApplication] setMinimumBackgroundFetchInterval:3600]; 
//app后台运行的时间
NSTimeInterval remainTime = [[UIApplication sharedApplication] backgroundTimeRemaining]; 
NSLog(@"remainTIme = %f",remainTime);
//后台刷新的状态
int state = [[UIApplication sharedApplication] backgroundRefreshStatus]; 
NSLog(@"state = %d",state);
[[UIApplication sharedApplication] beginBackgroundTaskWithName:@"taskOne" expirationHandler:^{
}];
[[UIApplication sharedApplication] beginBackgroundTaskWithExpirationHandler:^{
}];
[[UIApplication sharedApplication] endBackgroundTask:1];
//远程的控制相关
[[UIApplication sharedApplication] beginReceivingRemoteControlEvents];
[[UIApplication sharedApplication] endReceivingRemoteControlEvents];
//系统的Idle Timer
[UIApplication sharedApplication].idleTimerDisabled = YES; //不让手机休眠
```

###九、APP样式

```objc
//隐藏状态条
[[UIApplication sharedApplication] setStatusBarHidden:YES];
[[UIApplication sharedApplication] setStatusBarHidden:YES withAnimation:UIStatusBarAnimationSlide];
//设置状态条的样式
[[UIApplication sharedApplication] setStatusBarStyle:UIStatusBarStyleDefault];
[[UIApplication sharedApplication] statusBarStyle];
//状态条的Frame
[[UIApplication sharedApplication] statusBarFrame];
//网络是否可见
[[UIApplication sharedApplication] isNetworkActivityIndicatorVisible];
//badge数字
[UIApplication sharedApplication].applicationIconBadgeNumber = 2;
//屏幕的方向
[[UIApplication sharedApplication] userInterfaceLayoutDirection];
```

###十、设置状态条的方向

```objc
[[UIApplication sharedApplication] setStatusBarOrientation:UIInterfaceOrientationLandscapeLeft animated:YES];
```

###十一、数据类型

```objc
UIBackgroundTaskIdentifier : Int
typedef enum : NSUInteger {
   UIRemoteNotificationTypeNone    = 0,
   UIRemoteNotificationTypeBadge   = 1 << 0,
   UIRemoteNotificationTypeSound   = 1 << 1,
   UIRemoteNotificationTypeAlert   = 1 << 2,
   UIRemoteNotificationTypeNewsstandContentAvailability = 1 << 3
} UIRemoteNotificationType;
typedef enum : NSInteger {
   UIStatusBarStyleDefault,
   UIStatusBarStyleLightContent,
   UIStatusBarStyleBlackTranslucent,
   UIStatusBarStyleBlackOpaque
} UIStatusBarStyle;
typedef enum : NSInteger {
   UIStatusBarAnimationNone,
   UIStatusBarAnimationFade,
   UIStatusBarAnimationSlide,
} UIStatusBarAnimation;
typedef enum : NSInteger  {
   UIApplicationStateActive ,
   UIApplicationStateInactive ,
   UIApplicationStateBackground 
} UIApplicationState;
const UIBackgroundTaskIdentifier  UIBackgroundTaskInvalid ;
const NSTimeInterval  UIMinimumKeepAliveTimeout;
typedef enum : NSUInteger  {
   UIBackgroundFetchResultNewData ,
   UIBackgroundFetchResultNoData ,
   UIBackgroundFetchResultFailed 
} UIBackgroundFetchResult;
const NSTimeInterval  UIApplicationBackgroundFetchIntervalMinimum ;
const NSTimeInterval  UIApplicationBackgroundFetchIntervalNever;
typedef enum : NSUInteger  {
   UIBackgroundRefreshStatusRestricted ,
   UIBackgroundRefreshStatusDenied ,
   UIBackgroundRefreshStatusAvailable 
} UIBackgroundRefreshStatus;
typedef enum : NSInteger  {
   UIInterfaceOrientationUnknown             = UIDeviceOrientationUnknown ,
   UIInterfaceOrientationPortrait            = UIDeviceOrientationPortrait ,
   UIInterfaceOrientationPortraitUpsideDown  = UIDeviceOrientationPortraitUpsideDown ,
   UIInterfaceOrientationLandscapeLeft       = UIDeviceOrientationLandscapeRight ,
   UIInterfaceOrientationLandscapeRight      = UIDeviceOrientationLandscapeLeft 
} UIInterfaceOrientation;
typedef enum : NSInteger  {
   UIUserInterfaceLayoutDirectionLeftToRight ,
   UIUserInterfaceLayoutDirectionRightToLeft ,
} UIUserInterfaceLayoutDirection;
NSString *const  UIApplicationOpenSettingsURLString;
NSString *const  UIApplicationStatusBarOrientationUserInfoKey ;
NSString *const  UIApplicationStatusBarFrameUserInfoKey;
NSString *const  UIContentSizeCategoryExtraSmall ;
NSString *const  UIContentSizeCategorySmall ;
NSString *const  UIContentSizeCategoryMedium ;
NSString *const  UIContentSizeCategoryLarge ;
NSString *const  UIContentSizeCategoryExtraLarge ;
NSString *const  UIContentSizeCategoryExtraExtraLarge ;
NSString *const  UIContentSizeCategoryExtraExtraExtraLarge;
NSString *const  UIContentSizeCategoryAccessibilityMedium ;
NSString *const  UIContentSizeCategoryAccessibilityLarge ;
NSString *const  UIContentSizeCategoryAccessibilityExtraLarge ;
NSString *const  UIContentSizeCategoryAccessibilityExtraExtraLarge ;
NSString *const  UIContentSizeCategoryAccessibilityExtraExtraExtraLarge;
NSString *const  UIApplicationInvalidInterfaceOrientationException;
```

###十二、通知

```objc
UIApplicationBackgroundRefreshStatusDidChangeNotification
UIApplicationDidBecomeActiveNotification
UIApplicationDidChangeStatusBarFrameNotification
UIApplicationDidChangeStatusBarOrientationNotification
UIApplicationDidEnterBackgroundNotification
UIApplicationDidFinishLaunchingNotification
UIApplicationDidReceiveMemoryWarningNotification
UIApplicationProtectedDataDidBecomeAvailable
UIApplicationProtectedDataWillBecomeUnavailable
UIApplicationSignificantTimeChangeNotification
UIApplicationUserDidTakeScreenshotNotification
UIApplicationWillChangeStatusBarOrientationNotification
UIApplicationWillChangeStatusBarFrameNotification
UIApplicationWillEnterForegroundNotification
UIApplicationWillResignActiveNotification
UIApplicationWillTerminateNotification
UIContentSizeCategoryDidChangeNotification
```
