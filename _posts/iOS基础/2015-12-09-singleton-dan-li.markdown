---
layout: post
title: Singleton--单例
category: iOS基础
tags: iOS 单例 Objective-C
image: /images/head-800x400/-45.png
description: 说到单例首先要提到单例模式，因为单例模式是单例存在的目的：单例模式是一种常用的软件设计模式。在它的核心结构中只包含一个被称为单例类的特殊类。通过单例模式可以保证系统中一个类只有一个实例而且该实例易于外界访问，从而方便对实例个数的控制并节约系统资源。如果希望在系统中某个类的对象只能存在一个，单例模式是最好的解决方案。
toc: true
homepage: false
---

## 一、单例介绍

### 1. 什么是单例

说到单例首先要提到**单例模式**，因为单例模式是单例存在的目的

* 单例模式是一种常用的软件设计模式。在它的核心结构中只包含一个被称为单例类的特殊类。通过单例模式可以保证系统中一个类只有一个实例而且该实例易于外界访问，从而方便对实例个数的控制并节约系统资源。如果希望在系统中某个类的对象只能存在一个，单例模式是最好的解决方案。

* 单例，顾名思义：单独的实例。

* 简单的说，单例是一个特殊的实例，在单例所属的类中只存在单例这么一个实例，并且单例类似全局变量，在系统任意地方都能访问单例

* 获取单例的规律一般以 default、stand、share开头

> 单例对象
> 我们使用一个类获取对象时，多次创建对象或者多次获取对象，得到的都是同一份对象，那么该对象就是 **单例对象** -- 同时只存在一个实例对象。
> 创建对象时使用的类被称为单例类。

* 单例对象的特点
	+ 可以被全局访问
	+ 不会被释放
	+ 创建过程只执行1次

### 2. 单例用处

根据单例模式的定义，我们知道一般两种情况下使用单例：

> 1. 系统中某种对象只能存在一个，多了就会出问题

> 2. 系统中某种对象实例只需要一个就够用了，多了占内存

对于第一种情况，我们必须使用单例，对于第二种情况，我们虽然可以不用单例，但是单例是更优的选择

iOS的系统中有很多地方用的都是单例:


{% highlight objc linenos %}
[UIApplication sharedApplication];
[NSNotificationCenter defaultCenter];
[NSFileManager defaultManager];
[NSUserDefaults standardUserDefaults];
[NSURLCache sharedURLCache];
[NSHTTPCookieStorage sharedHTTPCookieStorage];
{% endhighlight %}

## 二、单例的创建

单例有伪单例和完整的单例之分，一般使用伪单例就足够了 每次都用 sharedDataHandle 创建对象。

### 1. 伪单例

伪单例，可以理解为通过我们特定的方法获取的对象才是单例对象。
其实现分为两步：
1、 获取单例对象的方法

{% highlight objc linenos %}
// 创建单例对象的方法。类方法 
// 命名规则： default/standard/shared + 类名
+ (DataHandle *)sharedDataHandle; 
{% endhighlight %}

2、 方法的实现，实现可有下面的三种方式
#### （1）单线程单例

我们知道对于单例类，我们必须留出一个接口来返回生成的单例，由于一个类中只能有一个实例，所以我们在第一次访问这个实例的时候创建，之后访问直接取已经创建好的实例

{% highlight objc linenos %}
@implementation DataHandle
//
// 因为实例是全局的 因此要定义为全局变量，且需要存储在静态区，不释放。不能存储在栈区。
static DataHandle *handle = nil;
//
+ (DataHandle *)sharedDataHandle{
	if(!handle) {
		handle = [[DataHandle alloc] init];
		    }
	return handle;
}
@end
{% endhighlight %}

> 备注：严格意义上来说，我们还需要将`alloc`方法封住，因为严格的单例是不允许再创建其他实例的，而`alloc`方法可以在外部任意生成实例。但是考虑到`alloc`属于`NSObject`，iOS中无法将`alloc`变成私有方法，最多只能覆盖`alloc`让其返回空，不过这样做也可能会让使用接口的人误解，造成其他问题。所以我们一般情况下对`alloc`不做特殊处理。系统的单例也未对`alloc`做任何处理。理论上对`alloc`进行处理的单例才算真正意义上完整的单例，下面会有说明的。

#### （2）@synchronized 单例

对于一个实例，我们一般并不能保证他一定会在单线程模式下使用，所以我们得适配多线程情况。在多线程情况下，上面的单例创建方式可能会出现问题。如果两个线程同时调用`sharedDataHandle`,可能会创建出2个 handle 来。所以对于多线程情况下，我们需要使用`@synchronized`来加锁。

{% highlight objc linenos %}
@implementation DataHandle
//
static DataHandle *handle = nil;
//
+ (DataHandle *)sharedDataHandle{
//
	@synchronized(self){
		if(!handle) {
            handle = [[DataHandle alloc] init];
         }
    }
    return handle;
}
@end
{% endhighlight %}

这样的话，当多个线程同时调用`sharedDataHandle`时，由于`@synchronized`已经加锁，所以只能有一个线程进入创建 handle。这样就解决了多线程下调用单例的问题

### 3. dispatch_once单例

使用`@synchronized`虽然解决了多线程的问题，但是并不完美。因为只有在handle未创建时，我们加锁才是有必要的。如果handle已经创建.这时候锁不仅没有好处，而且还会影响到程序执行的性能（多个线程执行`@synchronized`中的代码时，只有一个线程执行，其他线程需要等待）。那么有没有方法既可以解决问题，又不影响性能呢？

这个方法就是GCD中的`dispatch_once`

{% highlight objc linenos %}
@implementation DataHandle
//
static DataHandle *handle = nil;
//
+ (DataHandle *)sharedDataHandle{
//
	staticdispatch_once_tonceToken;//①onceToken = 0;
	dispatch_once(&onceToken, ^{			
		NSLog(@"%ld",onceToken);//②onceToken = 140734731430192
		handle = [[DataHandle alloc] init];		
	    });
	NSLog(@"%ld",onceToken); //③onceToken = -1;
	return handle;
}
@end
{% endhighlight %}

`dispatch_once`为什么能做到既解决同步多线程问题又不影响性能呢？

下面我们来看看`dispatch_once`的原理：

`dispatch_once`主要是根据onceToken的值来决定怎么去执行代码。

当onceToken= 0时，线程执行`dispatch_once`的block中代码

当onceToken= -1时，线程跳过`dispatch_once`的block中代码不执行

当onceToken为其他值时，线程被线程被阻塞，等待onceToken值改变

当线程首先调用`shareInstance`，某一线程要执行block中的代码时，首先需要改变onceToken的值，再去执行block中的代码。这里onceToken的值变为了 140734731430192。

这样当其他线程再获取onceToken的值时，值已经变为 140734731430192。其他线程被阻塞。

当block线程执行完block之后。onceToken变为-1。其他线程不再阻塞，跳过block。

下次再调用`shareInstance`时，block已经为-1。直接跳过block。

这样`dispatch_once`在首次调用时同步阻塞线程，生成单例之后，不再阻塞线程。`dispatch_once`是创建单例的最优方案

### 4. 完整的单例

单例在应用程序运行期间**只会存在一份**，因此不管是alloc、init 还是 copy，mutableCopy 都应当只有一份实例，由于 alloc 方法内部是调用`allocWithZone`，重写 `allocWithZone`，让这个方法生成实例的代码只能运行一次即可。

**完整的单例**要做到四个方面：

1. 为单例对象实现一个静态实例,然后设置成nil，
2. 构造方法检查静态实例是否为nil，是则新建并返回一个实例，
3. 重写allocWithZone方法，用来保证其他人直接使用alloc和init试图获得一个新实例的时候不会产生一个新实例，
4. 适当实现copyWithZone，,retain,retainCount,release和autorelease 等方法

注意： 此处完整的单例在进行方法实现时，略有不同 目的是为了避免死循环，如果 在单例类里面重写了 `allocWithZone` 方法 ， 在创建单例对象时 使用 `[[DataHandle alloc] init]` 创建，会死循环，具体如下：

{% highlight objc linenos %}
@implementation DataHandle
//
static DataHandle *handle = nil;
//
+ (DataHandle *)sharedDataHandle{
	staticdispatch_once_tonceToken;
	dispatch_once(&onceToken, ^{			
           handle = [[super allocWithZone:nil] init]; // 避免死循环
	    });
	return handle;
}
@end
{% endhighlight %}

除了alloc 方法还需要考虑 copy 和 mutableCopy，这两个不需要使用到 alloc 就能得到一个实例，因此也需要重写这两个实例。（需要遵守各自的协议才能实现方法）
	
	
{% highlight objc linenos %}
- (id)copy
{
    return self;
}
//
- (id)mutableCopy
{
    return self;
}
//
+ (instancetype)allocWithZone:(struct _NSZone *)zone
{
    return [DataHandle sharedDataHandle];
}
//
+ (id)copyWithZone:(struct _NSZone *)zone
{
    return self;
}
{% endhighlight %}

如果是 ARC，单例设计就已经完成，但在 MRC 环境下的时候，还需要做一些额外的事情,因为单例会在应用程序运行时一直存在,因此不需要 retain和 release 操作,而且在 release 之后,如果单例对象被释放掉，由于生成实例的方法只会运行一遍，释放掉之后再运行，返回的实例会是空的，这个并不是预期结果,因此 retain 和 release 方法也都需要重写，另外新需要将 retainCount 方法也重写，以提示这是个特殊的对象。

{% highlight objc linenos %}
+ (instancetype)alloc{
    return [DataHandle sharedDataHandle];
}
//
-(oneway void)release{
	//nothing
	//  因为只有一个实例， 一直不释放，所以不增加引用计数。增加无意义。
 }
//
- (instancetype)retain{	 
    return self;
}
//
- (instancetype)autorelease{
   		return self;
}
//
- (NSUInteger)retainCount{	    
    return NSUIntegerMax;
}
{% endhighlight %}

> __注意__: 
> 
> * 可以使用宏来抽取单例的代码,但要注意区分 ARC 和 MRC
> * `if __has_feature(objc_arc)` 可判断当前项目的环境是否是ARC
> * 单例不可以被继承，由于生成实例的方法只会执行一次，如果有继承，父类跟子类最后都会是一样的实例。


## 三、总结：

* 单例模式是一个很好的设计模式，他就像一个全局变量一样，可以让我们在任何地方都使用同一个实例。

* 如果要自己创建单例模式，最好使用`dispatch_once`方法，这样即可解决多线程问题，又能达到高效的目的

* 单例虽然好用，不过他并不适合继承和扩展，所以使用单例的时候要注意这点。千万不要任何东西都使用单例，要适可而止



















