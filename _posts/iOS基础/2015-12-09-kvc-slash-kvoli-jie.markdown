---
layout: post
title: KVC/KVO理解
category: iOS基础
tags: iOS KVC KVO Objective-C
image: /images/head-800x400/-47.png
description: KVC 与 KVO 是 Objective C 的关键概念，是必须要理解的东西。
homepage: false
---

KVC 与 KVO 是 Objective C 的关键概念，是必须要理解的东西
<!--more-->

下面是实例讲解。

##一、KVC -- Key-Value Coding 

KVC，即是指 [NSKeyValueCoding](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Protocols/NSKeyValueCoding_Protocol/index.html#//apple_ref/occ/cat/NSKeyValueCoding)，一个非正式的 Protocol，提供一种机制来间接访问对象的属性。KVO 就是基于 KVC 实现的关键技术之一。

一个对象拥有某些属性。比如说，一个 Person 对象有一个 name 和一个 address 属性。以 KVC 说法，Person 对象分别有一个 value 对应他的 name 和 address 的 key。 key 只是一个字符串，它对应的值可以是任意类型的对象。从最基础的层次上看，KVC 有两个方法：

* 一个是设置 key 的值，
* 另一个是获取 key 的值。

如下面的例子：

```objc Person.h
#import <Foundation/Foundation.h>
//
@interface Person : NSObject
//
@property(nonatomic, strong)NSString *name;
@property(nonatomic, strong)NSString *address;
@property(nonatomic, strong)Person *spouse;
//
@end
```

```objc ViewController.m
- (void)viewDidLoad {
    [super viewDidLoad];
//
    Person *person = [[Person alloc] init];
    person.name = @"Jack";
    person.address = @"chengdu";
//    
    [self changePerson:person Name:@"Kevin"];
}
//
- (void)changePerson:(Person *)person Name:(NSString *)newName{
	// using the KVC accessor (getter) method
    NSString *originalName = [person valueForKey: @"name"];
//
	// using the KVC  accessor (setter) method.    
    [person setValue:newName forKey:@"name"];
//   
    NSLog(@"changed %@`s name to: %@", originalName, person.name);
//输出：changed Jack`s name to: Kevin    
}
```

现在，如果 Person 有另外一个 key 配偶（spouse），spouse 的 key 值是另一个 Person 对象，用 KVC 可以这样写：

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
//
    Person *person = [[Person alloc] init];
    person.name = @"Jack";
    person.address = @"chengdu";
//    
    [self changePerson:person Name:@"Kevin"];
//
    Person *spouse = [[Person alloc] init];
    spouse.name = @"Rose";
    spouse.address = @"chengdu";
//    
    person.spouse = spouse;
//    
    [self logMarriage:person];
}
//
- (void) logMarriage:(Person *)person{
    // just using the accessor again, same as example above
    NSString *personName = [person valueForKey:@"name"];  
//
	// this line is different, because it is using 
	// a "key path" instead of a normal "key"
    NSString *spouseName = [person valueForKeyPath:@"spouse.name"];
//    
    NSLog(@"%@ is happliy married to %@", personName, spouseName);
	//
}
```

`key` 与 `key path` 要区分开来，`key` 可以从一个对象中获取值，而 `key path` 可以将多个 key 用点号 “.” 分割连接起来，比如上面例子中的：

```
[p valueForKeyPath:@"spouse.name"];
```

相当于这样……

```
[[p valueForKey:@"spouse"] valueForKey:@"name"];
```

好了，以上是 KVC 的基本知识，接着看看 KVO。

##二、KVO -- Key-Value Observing

`Key-Value Observing (KVO)` 建立在 KVC 之上，它能够观察一个对象的 KVC key path 值的变化。举个例子，用代码观察一个 person 对象的 address 变化，以下是实现的三个方法：

* `watchPersonForChangeOfAddress: `实现观察

* `observeValueForKeyPath:ofObject:change:context: `在被观察的 key path 的值变化时调用。

* `dealloc`停止观察

举个例子如下：

```objc
static NSString *const KVO_CONTEXT_ADDRESS_CHANGED = @"KVO_CONTEXT_ADDRESS_CHANGED";
// 
@implementation ViewController
// 
- (void)watchPersonForChangeOfAddress:(Person *)person
{
    // this begins the observing
    [person addObserver:self
        forKeyPath:@"address"
           options:0
           context:(__bridge void * _Nullable)(KVO_CONTEXT_ADDRESS_CHANGED)];
    //
    // keep a record of all the people being observed,
    // because we need to stop observing them in dealloc
    [m_observedPeople addObject:person];
}
//
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSString *,id> *)change context:(void *)context{
//    
    if(context == (__bridge void * _Nullable)(KVO_CONTEXT_ADDRESS_CHANGED)) {
        NSString *name = [object valueForKey:@"name"];
        NSString *address = [object valueForKey:@"address"];
        NSLog(@"%@ has a new address: %@", name, address);
    }
}
//
- (void)dealloc{
//    
    // must stop observing everything before this object is
    // deallocated, otherwise it will cause crashes
    for(Person *person in m_observedPeople){
        [person removeObserver:self forKeyPath:@"address"];
    }
    m_observedPeople = nil;
}
@end
```
在这个例子中，当 person 的 aaddress 一旦发生改变，就会 `NSLog(@"%@ has a new address: %@", name, address);`，这就是 KVO 的作用，它通过 `key path` 观察对象的值，当值发生变化的时候会收到通知。

附上本文代码的github链接：[KVC/KVO理解](https://github.com/XcodeTalk/KVC-KVO)






















