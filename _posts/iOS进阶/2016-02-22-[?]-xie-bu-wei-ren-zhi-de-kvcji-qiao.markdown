---
layout: post
title: KVC的一些不为人知小技巧
category: iOS进阶
tags: iOS KVC
image: http://o7rxin1of.qnssl.com/images/head-800x400/-30.png
description: 本文主要来记录下一些不为人知，也经常被忽略掉，并且很实用的KVC干货小技巧
homepage: false
toc: true
---

本文主要来记录下一些不为人知，也经常被忽略掉，并且很实用的KVC干货小技巧

### 一、获取数组里的,最大、最小、平均、求和

{% highlight objc  %}
NSArray *array = @[@"11", @"9", @"2", @"20", @"8"]; 
NSNumber *sum = [array valueForKeyPath:@"@sum.floatValue"]; 
NSNumber *avg = [array valueForKeyPath:@"@avg.floatValue"]; 
NSNumber *max = [array valueForKeyPath:@"@max.floatValue"]; 
NSNumber *min = [array valueForKeyPath:@"@min.floatValue"];  
NSLog(@"sum:%@",sum); 
NSLog(@"avg:%@",avg);
NSLog(@"max:%@",max); 
NSLog(@"min:%@",min);
{% endhighlight %}

### 二、删除重复数据

{% highlight objc  %}
NSArray *array = @[@"a", @"b", @"c", @"a", @"d"]; //返回的是一个新的数组
NSArray *newArray = [array valueForKeyPath:@"@distinctUnionOfObjects.self"]; 
NSLog(@"%@", newArray);
{% endhighlight %}

### 三、同样可以嵌套使用，先剔除name对应值的重复数据再取值
{% highlight objc  %}
NSArray *array = @[ @{@"title":@"zxp",@"name":@"zhangxiaoping"}, @{@"title":@"zxp2",@"name":@"zhangxiaoping2"}, @{@"title":@"zxp",@"name":@"zhangxiaoping3"}, @{@"title":@"zxp",@"name":@"zhangxiaoping"}];
//根据name字段，来进行重复删除。
NSArray *newArray = [array valueForKeyPath:@"@distinctUnionOfObjects.name"];
//如果要根据title字段来删除重名的写法为`@distinctUnionOfObjects.title` 
NSLog(@"%@", newArray);
// print:( zhangxiaoping3, zhangxiaoping2, zhangxiaoping)是一个字符串数组
{% endhighlight %}

### 四、进行实例方法的调用

{% highlight objc  %}
NSArray *array = @[@"name", @"w", @"aa", @"ZXPing"]; 
NSLog(@"%@", [array valueForKeyPath:@"uppercaseString"]);
{% endhighlight %}

相当于数组中的每个成员执行了`uppercaseString`方法，然后把返回的对象组成一个新数组返回。既然可以用`uppercaseString`方法，那么NSString的其他方法也可以，比如`[array valueForKeyPath:@"length"]`。当然，其他对象的实例方法也可以以此类推来进行调用！

> 参考文章：[高效开发iOS系列 -- 那些不为人知的KVC](http://www.jianshu.com/p/a6a0abac1c4a)

















