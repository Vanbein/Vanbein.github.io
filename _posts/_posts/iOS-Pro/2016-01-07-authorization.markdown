---
layout: post
title: 访问权限：相册、相机、定位，耳机...
category: iOS-Pro, tips
tags: iOS authorization
image: /images/head-800x400/-12.png
description: iOS7以后需要用户授权才能访问相机、相册、定位等信息，所以进行查询现在的授权状态很有必要。
homepage: false
---

iOS7以后需要用户授权才能访问相机、相册、定位等信息，所以进行查询现在的授权状态是很有必要。


###一、访问相册
当第一次在应用中访问相册库的时候，系统会提示用户是否允许应用访问相册，如果用户选择不允许的话，应用以后将无法访问相册。如果想要重新被允许，那么需要用户去“隐私设置”里面去设置。

在程序中怎么获取是否拥有对相册的访问权限，然后做相应地操作呢，首先下面列出了相册的一些权限值和对应的含义：

```objc
typedef NS_ENUM(NSInteger, ALAuthorizationStatus) {
	ALAuthorizationStatusNotDetermined = 0, 
	//用户未对这个应用程序的权限做出选择
	//
	ALAuthorizationStatusRestricted,
	//此应用程序没有被授权访问的照片数据。可能是家长控制权限。
	//        
	ALAuthorizationStatusDenied,
	//用户已经明确拒绝了此应用程序访问照片数据.
	//            
	ALAuthorizationStatusAuthorized
	//用户已授权此应用访问照片数据.         
}
```

我们在应用中只需要通过以下代码即可获得相册的权限值，然后做相应的操作。

```
ALAuthorizationStatus author = [ALAssetsLibrary authorizationStatus];
```

* **注意：**iOS9 以后使用了`PHAuthorizationStatus`代替上面的方法。

###二、访问相机

在iOS7之前,摄像头是一直可以访问的，隐私设置选项中没有关闭相应软件的摄像头功能的选项。在ios7以后摄像头和相册一样增加了访问权限的设置，应用中第一次访问摄像头的时候，系统会询问你是否授权应用访问你的摄像头。摄像头的权限和相册的权限基本上一样，有：

```objc
typedef NS_ENUM(NSInteger, AVAuthorizationStatus) {
	AVAuthorizationStatusNotDetermined = 0,
	AVAuthorizationStatusRestricted,
	AVAuthorizationStatusDenied,
	AVAuthorizationStatusAuthorized
} NS_AVAILABLE_IOS(7_0) __TVOS_PROHIBITED;
```

同相册一样，我们可以通过以下代码获取对摄像头的访问权限：

```objc
AVAuthorizationStatus authStatus = [AVCaptureDevice authorizationStatusForMediaType:AVMediaTypeVideo];  
```

###三、定位服务

若需要检测的是整个的iOS系统的定位服务是否开启，可通过以下方法：

```objc
[CLLocationManager locationServicesEnabled] 
```

但是往往我们需要检测当前应用的定位服务是否开启，此时我们可通过下面方法来检测：

```
	//获取定位权限状态
    CLAuthorizationStatus *authStatus = [CLLocationManager authorizationStatus];
	//
	//定位服务的 Status 有以下值
typedef NS_ENUM(int, CLAuthorizationStatus) {
	//
	//用户未对这个应用程序的权限做出选择
	kCLAuthorizationStatusNotDetermined = 0,
	//
	//此应用程序被禁止使用定位服务。可能整个iOS系统的定位服务都被禁用了
	kCLAuthorizationStatusRestricted,
	// 
	//用户已明确拒绝授权此应用程序，或在设置中关闭了位置服务。
	kCLAuthorizationStatusDenied,
	//
	//用户授权应用程序在任何时候使用位置服务，包括监控区域，参观，或有显著的位置变化。 
	kCLAuthorizationStatusAuthorizedAlways NS_ENUM_AVAILABLE(NA, 8_0),
	//
	//用户授权在使用应用程序期间使用位置服务
	kCLAuthorizationStatusAuthorizedWhenInUse NS_ENUM_AVAILABLE(NA, 8_0),
	// 
	//这个值已经从iOS8后抛弃使用，其值相当于kCLAuthorizationStatusAuthorizedAlways
	kCLAuthorizationStatusAuthorized NS_ENUM_DEPRECATED(10_6, NA, 2_0, 8_0) 
};
```

###四、监听耳机的插入和拔出

一般都是使用通知的方式，添加收到通知的回调方法，在回调方法里面进行判断。实现代码如下：

```
//注册通知
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(audioRouteChangeListenerCallback:) name:AVAudioSessionRouteChangeNotification object:nil];
//通知回调
- (void)audioRouteChangeListenerCallback:(NSNotification*)notification
{
	NSDictionary *interuptionDict = notification.userInfo;
	NSInteger routeChangeReason = [[interuptionDict valueForKey:AVAudioSessionRouteChangeReasonKey] integerValue];
	switch (routeChangeReason) {
		case AVAudioSessionRouteChangeReasonNewDeviceAvailable:{
			NSLog(@"AVAudioSessionRouteChangeReasonNewDeviceAvailable");
			NSLog(@"耳机插入");
			break;
		}
		case AVAudioSessionRouteChangeReasonOldDeviceUnavailable:{
			NSLog(@"AVAudioSessionRouteChangeReasonOldDeviceUnavailable");
			NSLog(@"耳机拔出，停止播放操作");
			break;
		}
		case AVAudioSessionRouteChangeReasonCategoryChange:{
			// called at start - also when other audio wants to play
			NSLog(@"AVAudioSessionRouteChangeReasonCategoryChange");
			break;
		}
	}
}
```












