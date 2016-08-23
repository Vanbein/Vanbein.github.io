---
layout: post
title: iOS 数组排序问题
category: iOS进阶
tags: iOS
image: /images/head-800x400/-59.png
description: 本文主要记录下 iOS 开发过程碰到的数组排序问题，对数组中的元素制定一个 key 即可进行升序或降序排序了。
homepage: false
toc: true
---

### 数组元素类型

假设我们这个数组里面的元素是一个 model 类型，比如 **CHSAssetGroup**，其`CHSAssetGroup.h` 文件内容如下：

{% highlight objc  %}
@interface CHSAssetGroup : NSObject

@property (nonatomic, copy) NSString *name; // 名字
@property (nonatomic, copy) NSString *creationDate; // 日期
{% endhighlight %}


CHSAssetGroup 有两个属性，第一个是名字，第二个创建日期

### 数组排序的 key

现在我们创建一个数组，里面有 3 个 `CHSAssetGroup`类型的对象：

{% highlight objc  %}
CHSAssetGroup *group_a = [[CHSAssetGroup alloc] init];
group_a.name = @"a";
group_a.creationDate = @"2016-5-28";

CHSAssetGroup *group_b = [[CHSAssetGroup alloc] init];
group_b.name = @"b";
group_b.creationDate = @"2016-8-23";

CHSAssetGroup *group_c = [[CHSAssetGroup alloc] init];
group_c.name = @"c";
group_c.creationDate = @"2016-6-21";

NSArray *array = @[group_a, group_b, group_c];
{% endhighlight %}

现在我们要让数组里面的元素按照 `CHSAssetGroup` 的 `creationDate` 降序排序，代码如下：

{% highlight objc  %}
// Key: 按照排序的key; ascending: YES为升序, NO为降序。
    NSSortDescriptor *sorter = [[NSSortDescriptor alloc] initWithKey:@"creationDate" ascending:NO];
    NSArray *sortDescriptors = [[NSArray alloc] initWithObjects:&sorter count:1];
    NSArray *sortedArray = [array sortedArrayUsingDescriptors:sortDescriptors];
    NSLog(@"%@", sortedArray);
{% endhighlight %}

NSSortDescriptor 指定用于对象数组排序的对象的属性。sortedArray 即为排序后的数组，其元素顺序为 @[group_b, group_c, group_a];


如果是要求对象按照多可 key 排序，比如先按 name 排序，再按 creationDate 排序。那就创建两个descriptor，将两个 descriptor 放到数组里一起传给需要排序的数组。
{% highlight objc  %}
    NSSortDescriptor *nameSorter = [NSSortDescriptor sortDescriptorWithKey:@"name" ascending:NO];
     NSSortDescriptor *dateSorter = [[NSSortDescriptor alloc] initWithKey:@"creationDate" ascending:NO];
    NSArray *sortDescriptors2 = [[NSArray alloc] initWithObjects:nameSorter, dateSorter, nil];
{% endhighlight %}


### 可变数组排序

如果想要排序的数组是一个可变数组，则只需要调用 `sortUsingDescriptors:` 方法对自身进行排序即可：

{% highlight objc  %}
    NSMutableArray *mutableArray = [NSMutableArray arrayWithObjects:group_a, group_b, group_c, nil];
    NSSortDescriptor *sorter = [[NSSortDescriptor alloc] initWithKey:@"creationDate" ascending:NO];
    NSArray *sortDescriptors = [[NSArray alloc] initWithObjects:&sorter count:1];
    [mutableArray sortUsingDescriptors:sortDescriptors]
    NSLog(@"%@", mutableArray);
 {% endhighlight %}

### 字符串数组排序

如果对象就是NSString，就是字符串数组排序，这就更加简单了，将 `sortdescriptor` 的 **key** 直接指定为 nil，就直接排序对象，而不是对象的某一个属性了。

{% highlight objc  %}
    NSSortDescriptor *descriptor = [NSSortDescriptor sortDescriptorWithKey:nil ascending:YES];
    NSArray *descriptors = [NSArray arrayWithObject:descriptor];
    NSArray *myDataArray = [NSArray arrayWithObjects:@"what", @"xman", @"highligth", @"mountain", @"success", @"Apple", nil];
    NSArray *sortedArray = [myDataArray sortedArrayUsingDescriptors:descriptors];
    NSLog(@"%@", sortedArray);
{% endhighlight %}







