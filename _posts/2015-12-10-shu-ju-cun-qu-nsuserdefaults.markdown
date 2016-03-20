---
layout: post
title: "NSUserDefaults--数据存取"
date: 2015-12-10 13:37:24 +0800
comments: true
categories: [Objective-C, iOS高级]
---

关于` NSUserDefaults `首先要看苹果官方的定义：[NSUserDefault官方文档](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSUserDefaults_Class/index.html)

<!--more-->

###一、NSUserDefaults简介
对于应用来说，每个用户都有自己的独特偏好设置，而好的应用会让用户根据喜好选择合适的使用方式，把这些偏好记录在应用包的plist文件中，通过 NSUserDefaults 类来访问，这是 NSUserDefaults 的常用姿势。如果有一些设置你希望用户即使升级后还可以继续使用，比如玩游戏时得过的最高分、喜好和通知设置、主题颜色甚至一个用户头像，那么你可以使用 NSUserDefaults 来存储这些信息，如果有更多需求，可以了解数据持久化相关的知识。

具体来说 NSUserDefaults 是iOS系统提供的一个单例类(iOS提供了若干个单例类)，通过类方法`standardUserDefaults `可以获取 NSUserDefaults 单例。如：

```objc
NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
```

NSUserDefaults单例以`key-value`的形式存储了一系列偏好设置，key是名称，value是相应的数据。存/取数据时可以使用方法`objectForKey:` 和 `setObject:forKey:`来把对象存储到相应的**plist** 文件中，或者读取，既然是plist文件，那么对象的类型则必须是plist文件可以存储的类型，正如官方文档中提到的——

> * NSData
>
> * NSString
>
> * NSNumber
>
> * NSDate
> 
> * NSArray
> 
> * NSDictionary

而如果需要存储plist文件不支持的类型，比如图片，可以先将其归档为NSData类型，再存入plist文件，需要注意的是，即使对象是NSArray或NSDictionary，他们所存储的对象类型也应该是以上范围包括的。

###二、存数据
NSUserDefaults 保存数据的方法可根据不同类型的对象使用更简便方法，

```
- setBool:forKey:
- setFloat:forKey:
- setInteger:forKey:
- setDouble:forKey:
- setURL:forKey:
```

比如保存一个整数、字符串、BOOL值。

```objc
NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
[defaults setObject:@”jack“ forKey:@"firstName"];
[defaults setInteger:10 forKey:@"Age"];
[defaults setBool:YES forKey:@"isMarried"];
[defaults synchronize];
```

> **注意**：方法 `synchronise` 是为了强制存储，其实并非必要，因为这个方法会在系统中默认调用，但是你确认需要马上就存储，这样做是可行的。**但是如果程序运行占用比较大的内存的时候不加这行代码，可能会造成无法写入plist文件中**，

* 保存**自定义的对象**时，可先转换为NSDate，再保存，比如保存一张图片

```objc
UIImage *image =[UIImage imageNamed:@"somename"];
NSData *imageData = UIImageJPEGRepresentation(image, 100);//把image归档为NSData
NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
[defaults setObject:imageData forKey:@"image"];
[defaults synchronize];
```

###三、读数据
数据读取和数据保存一样，不同的对象可用不同的方法，比如读取上面保存的几个对象：

```objc
NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
NSString *firstName = [defaults objectForKey:@"firstName"]
NSInteger age = [defaults integerForKey:@"Age"];
BOOL isMarried = [defaults boolForKey:@"isMarried"];
//读取自定义对象
NSData *imageData = [defaults dataForKey:@"image"];
UIImage *image = [UIImage imageWithData:imageData];
```

* **注意**保存到 NSUserDefaults 的对象一定是不可变的，因此读取出来也是不可变的，比如想要保存和读取一个可变数组mutableArray时，应该是这样的：

```objc
//保存：应该另存为转换为不可变数组再保存
NSMutableArray *mutableArray = [NSMutableArray arrayWithObjects:@"123",@"234",@"345", nil];
//
NSArray * array = [NSArray arrayWithArray:mutableArray];
NSUserDefaults *user = [NSUserDefaults standardUserDefaults];
[user setObject:array forKey:@"myArray"];
//...
//读取，不可直接用 objectForKey: 读取，应该用 alloc 方法代替
NSUserDefaults *user = [NSUserDefaults standardUserDefaults];
//NSMutableArray *mutableArray = [user objectForKey:@"myArray"]; //这样读取出来是不可变数组
NSMutableArray *mutableArray = [NSMutableArray arrayWithArray:[user objectForKey:@"myArray"]];
``

###四、删除数据
由于 NSUserDefaults 它是将信息写入到本地的一个plist文件里， 所以和删除plist里的某一项内容一样，直接用下面的方法就可以直接删除 NSUserDefaults 中的某一个特定的项的内容了，

```objc
[[NSUserDefaults standardUserDefaults] removeObjectForKey:key];
```

但是，如何把整个plist文件删除？
其实也不难，我们要知道删除整个plist文件实际上就是把plist文件中的所有item删除就行了，

我们先要取到 plist文件里的所有的 Key 否则是不能用 `removeObjectForKey：key `这个方法来删除的，dictionary 有一个方法 `[dictionary allKeys];`返回值是一个数组，这样我们能拿到dictionary 中所有的 key，

我们知道我们写入的plist文件中的项目是以一个字典的形式保存的，所以，

代码如下：

```objc
- (void)removeAllNSUserDefaultsObject{
//
NSUserDefaults *userDefatluts = [NSUserDefaults standardUserDefaults];
NSDictionary *dictionary = [userDefaults dictionaryRepresentation];
//
for(NSString* key in [dictionary allKeys]){
//
[userDefaults removeObjectForKey:key];
[userDefaults synchronize];
}
```

###五、NSUserDefaults域
考虑这么一种情况：

```
BOOL showTutorialOnLaunch = [[NSUserDefaults standardUserDefaults] boolForKey:@"ShowTutorial"];
```

这种情况下，当key值 `@“ShowTutorial”` 已设置后会运行正确。但如果默认数据库没有这个key的默认值时，将会返回NO，这或许就不一定是你需要的值了，因为无法区分 **NO** 和**no value**，前一段所提到的简便方法大多有这种问题。
解决方式：使用 `registerDefaults:`方法
首先创建一个包含用户偏好设置信息的`DefaultPreferences.plist`文件，添加到`target`中。在运行时，app就可以加载这个文件并且把内容传到 `registerDefaults :`

```objc
NSURL *defaultPrefsFile = [[NSBundle mainBundle]
URLForResource:@"DefaultPreferences" withExtension:@"plist"];
NSDictionary *defaultPrefs = [NSDictionary dictionaryWithContentsOfURL:defaultPrefsFile];
[[NSUserDefaults standardUserDefaults] registerDefaults:defaultPrefs];
```

注意需要在每次启动app并且没有在`user defaules` 中读取数据的时候调用以上方法，因为`registerDefaults:` 不能把这些默认数据存储到硬盘上，所以`application:didFinishLaunchingWithOptions`是最合适的地方。
> 这样做的原因是：默认情况下，应用域是空的，没见键也没有值。当应用第一次设置某项用户偏好设置的值时，相应的值会通过指定的键加入应用域。当通过 `NSUserDefaults` 获取某项用户偏好设置的值时，`NSUserDefaults` 会先在应用域中查找，如果找到了值，`NSUserDefaults`就会返回这个值。如果没有找到，`NSUserDefaults` 就会在注册域中查找并返回默认值。

####域
`user defaults`数据库中其实是由多个层级的域组成的，当你读取一个键值的数据时，`NSUserDefaults`从上到下透过域的层级寻找正确的值，不同的域有不同的功能，有些域是可持久的，有些域则不行。

* 应用域（application domain）是最重要的域，它存储着你app通过NSUserDefaults set...forKey添加的设置。

* 注册域（registration domain）仅有较低的优先权，只有在应用域没有找到值时才从注册域去寻找。

* 全局域（global domain）则存储着系统的设置

* 语言域（language-specific domains）则包括地区、日期等

* 参数域（ argument domain）有最高优先权











