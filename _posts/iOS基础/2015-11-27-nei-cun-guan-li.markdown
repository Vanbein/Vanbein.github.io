---
layout: post
title: 内存管理
category: iOS基础
tags: iOS Objective-C
image: /images/head-800x400/-7.png
description: 内存管理是iOS非常重要的一环，只有理解了iOS的内存管理机制，才能掌握好自己App的内存，写出优秀的App应用。
toc: true
homepage: false
---

内存管理是iOS非常重要的一环，只有理解了iOS的内存管理机制，才能掌握好自己App的内存，写出优秀的App应用。

## 概述

我们知道在程序运行过程中要创建大量的对象，和其他高级语言类似，在OC中对象是存储在堆中的，系统并不会自动释放堆中的内存（注意基本类型是由系统自己管理的，放在栈上）。如果一个对象创建并使用后没有得到及时释放那么就会占用大量内存。其他高级语言如C#、Java都是通过垃圾回收来（GC）解决这个问题的，但在OC中并没有类似的垃圾回收机制，因此它的内存管理就需要由开发人员手动维护。今天将着重介绍ObjC内存管理：

1. 引用计数器
2. 属性参数
3. 自动释放池

## 引用计数器

在Xcode4.2及之后的版本中由于引入了ARC（Automatic Reference Counting）机制,程序编译时Xcode可以自动给你的代码添加内存释放代码，如果编写手动释放代码Xcode会报错，因此在今天的内容中如果你使用的是Xcode4.2之后的版本（相信现在大部分朋友用的版本都比这个要高），必须手动关闭ARC，这样才有助于你理解ObjC的内存回收机制。

> ObjC中的内存管理机制跟C语言中指针的内容是同样重要的，要开发一个程序并不难，但是优秀的程序则更测重于内存管理，它们往往占用内存更少，运行更加流畅。虽然在新版Xcode引入了ARC，但是很多时候它并不能完全解决你的问题。在Xcode中关闭ARC：项目属性 —  `Build Settings` --搜索 `garbage` 找到 `Objective-C Automatic Reference Counting`设置为**No**即可。

### 内存管理原理
在ObjC中没有垃圾回收机制，那么ObjC中内存又是如何管理的呢？其实在ObjC中内存的管理是依赖对象引用计数器来进行的：在ObjC中每个对象内部都有一个与之对应的整数（retainCount），叫“引用计数器”，当一个对象在创建之后它的引用计数器为1，当调用这个对象的alloc、retain、new、copy方法之后引用计数器自动在原来的基础上加1（ObjC中调用一个对象的方法就是给这个对象发送一个消息），当调用这个对象的release方法之后它的引用计数器减1，如果一个对象的引用计数器为0，则系统会自动调用这个对象的dealloc方法来销毁这个对象。

如果一个对象被释放之后，那么最后引用它的变量我们手动设置为nil，否则可能造成野指针错误，而且需要注意在ObjC中给空对象发送消息是不会引起错误的。

> 野指针错误形式在Xcode中通常表现为： `Thread 1：EXC_BAD_ACCESS(code=EXC_I386_GPFLT)` 错误, 因为你访问了一块已经不属于你的内存。

### 内存释放的原则

手动管理内存有时候并不容易，因为对象的引用有时候是错综复杂的，对象之间可能互相交叉引用，此时需要遵循一个法则：**谁创建，谁释放**。


## 属性参数

 `@property` 的参数分为三类，也就是说参数最多可以有三个，中间用逗号分隔，每类参数可以从上表三类参数中人选一个。如果不进行设置或者只设置其中一类参数，程序会使用三类中的各个默认参数，默认参数：(`atomic`,`readwrite`,`assign`)

1. 原子性
	* ato 对属性加锁，多线程下线程安全，默认值
	* nonatomic 对属性不加锁，多线程下不安全，但是速度快
2. 读写属性
	* readwrite 生成getter、setter方法，默认值
	* readonly 只生成getter方法
3. set方法处理
	* assign 直接赋值 默认值
	* retain 先release原来的值，再retain新值
	* copy 先release原来的值，再copy新值
	* strong 强引用 = 拥有权，对传递过来的对象申明有拥有权
	* weak 弱引用 = 不引用，和strong配对，解决retainCycle（重复引用）

一般情况下如果在多线程开发中一个属性可能会被两个及两个以上的线程同时访问，此时可以考虑atomic属性，否则建议使用nonatomic，不加锁，效率较高；readwirte方法会生成getter、setter两个方法，如果使用readonly则只生成getter方法；关于set方法处理需要特别说明，假设我们定义一个属性a，这里列出三种方式的生成代码：

`assign`，用于基本数据类型

{% highlight objc  %}
-(void)setA:(int)a{
    _a=a;
}
{% endhighlight %}

`retain`，通常用于非字符串对象

{% highlight objc  %}
-(void)setA:(Car *)a{
    if(_a!=a){
        [_a release];
        _a=[a retain];
    }
}
{% endhighlight %}

`copy`，通常用于字符串对象、`block`、`NSArray`、`NSDictionary`

{% highlight objc  %}
-(void)setA:(NSString *)a{
    if(_a!=a){
        [_a release];
        _a=[a copy];
    }
}
{% endhighlight %}

> 备注：本文基于MRC进行介绍，ARC下的情况不同，请参阅其他文章，例如ARC下基本数据类型默认的属性参数为(atomic,readwrite,assign)，对象类型默认的属性参数为(atomic,readwrite,strong)

## 自动释放池
在ObjC中也有一种内存自动释放的机制叫做“自动引用计数”（或“自动释放池”），与C#、Java不同的是，这只是一种半自动的机制，有些操作还是需要我们手动设置的。自动内存释放使用@autoreleasepool关键字声明一个代码块，如果一个对象在初始化时调用了autorelase方法，那么当代码块执行完之后，在块中调用过autorelease方法的对象都会自动调用一次release方法。这样一来就起到了自动释放的作用，同时对象的销毁过程也得到了延迟（统一调用release方法）。

对于自动内存释放简单总结一下：

1. autorelease方法不会改变对象的引用计数器，只是将这个对象放到自动释放池中；
2. 自动释放池实质是当自动释放池销毁后调用对象的release方法，不一定就能销毁对象（例如如果一个对象的引用计数器>1则此时就无法销毁）；
3. 由于自动释放池最后统一销毁对象，因此如果一个操作比较占用内存（对象比较多或者对象占用资源比较多），最好不要放到自动释放池或者考虑放到多个自动释放池；
4. ObjC中类库中的静态方法一般都不需要手动释放，内部已经调用了autorelease方法；

## 实现
Objective-C提供了三种内存管理方式：`manual retain-release`（MRR，手动管理），`automatic reference counting`（ARC，自动引用计数），`garbage collection`（垃圾回收）。iOS不支持垃圾回收；ARC作为苹果新提供的技术，苹果推荐开发者使用ARC技术来管理内存；下面主要讲的是手动管理。

##### 内存管理的目的是：
1. 不要释放或者覆盖还在使用的内存，这会引起程序崩溃；
2. 释放不再使用的内存，防止内存泄露。iOS程序的内存资源是宝贵的。

MRR手动管理内存也是基于引用计数的，只是需要开发者发消息给某块内存（或者说是对象）来改变这块内存的引用计数以实现内存管理（ARC技术则是编译器代替开发者完成相应的工作）。一块内存如果计数是零，也就是没有使用者（owner），那么objective-C的运行环境会自动回收这块内存。

objective-C的内存管理遵守下面这个简单的策略：
注：文档中把引用计数加1的操作称为“拥有”（own，或者take ownership of）某块对象/内存；把引用计数减1的操作称为放弃（relinquish）这块对象/内存。拥有对象时，你可以放心地读写或者返回对象；当对象被所有人放弃时，objective-C的运行环境会回收这个对象。

1. 你拥有你创建的对象，也就是说创建的对象（使用alloc，new，copy或者mutalbeCopy等方法）的初始引用计数是1。
2. 给对象发送retain消息后，你拥有了这个对象
3. 当你不需要使用该对象时，发送release或者autorelease消息放弃这个对象
4. 不要对你不拥有的对象发送“放弃”的消息

> 注：简单的赋值不会拥有某个对象。比如：

{% highlight objc  %}
NSString *name = person.fullName;
{% endhighlight %}

上面这个赋值操作不会拥有这个对象（这仅仅是个指针赋值操作）；这和C++语言里的某些基于引用计数的类的行为是有区别的。想拥有一个objective-C对象，必须发送“创建”或者retain消息给该对象。

##### dealloc方法

dealloc方法用来释放这个对象所占的内存(包括成员变量)和其它资源。
不要使用dealloc方法来管理稀缺资源，比如文件，网络链接等。因为由于bug或者程序意外退出，dealloc方法不能保证一定会被调用。

##### Accessor Methods和内存管理

 `Accessor Methods`，也就是对象的 `property`（属性）的getter和setter方法。显然，如果getter返回的对象已经被运行环境回收了，那么这个getter的返回值是毫无意义的。这就需要在setter方法里“拥有”相应的property。
比如：

{% highlight objc  %}
@interface Counter : NSObject
@property (nonatomic, retain) NSNumber *count;
@end
{% endhighlight %}

getter方法仅仅返回成员变量就可以：

{% highlight objc  %}
-(NSNumber *)count {
    return _count;
}
{% endhighlight %}

setter方法需要保证对这个成员变量的“拥有”：

{% highlight objc  %}
-(void)setCount:(NSNumber *)newCount {
    [newCount retain];          //拥有新值
    [_count release];             //放弃老值
    _count = newCount;       //简单赋值
}
{% endhighlight %}

使用Accessor Methods
以下是一种使用方式：

{% highlight objc  %}
NSNumber *zero = ［NSNumber alloc] initWithInteger:0];
[self setCount:zero];
[zero release];
{% endhighlight %}

以下是一种可能引发错误的，偷懒的使用方式：

{% highlight objc  %}
NSNumber *zero = ［NSNumber alloc] initWithInteger:0];
[_count release];
_count = zero;  
//这个代码做了不合理的假设－_count对象“拥有”了某个内存。当修饰count属性的
//attribute发声变化时，这个假设就不一定正确了。
{% endhighlight %}


* 不要在初始化方法（ `Initializer` ）和 `dealloc` 方法里使用 `Accessor Methods`

* 不要在初始化方法里使用accessor methods的原因可能是（原文档中没有说明）：在初始化方法里，成员变量处于最初的状态，并没有任何值。考虑到一个成员变量的setter方法一般会对成员变量的旧值发送release消息。这种行为在初始化方法里没有意义。

* 如果需要在Initializer里给成员变量赋值，可参见一开始提到的原始文档里给出的示例代码。

* 使用weak reference（弱引用）来避免retain cycle
对一个对象发送retain消息会创建对这个对象的强引用（strong reference）。如果两个对象都有一个强引用指向对方，那么就形成了一个环（retain cycle）。这个环使得这两个对象都不可能被release。
弱引用（weak reference）指的是一种non-owning（非拥有）的关系，比如简单指针赋值关系。使用弱引用避免了retain cycle。但是需要注意的是，弱引用不能保证弱引用指向的对象是否存在，所以发消息给这个对象时一定要小心。如果弱引用指向的对象已经释放，那么发送消息给它会导致程序崩溃。所以，需要一点点额外的操作来使用弱引用所指的对象。比如，当向notification center注册一个对象时，notification center保存了一个指向这个对象的弱引用。当这个对象被回收时，需要通知下notification center。


* 当你使用对象时，要确保这个对象不会被回收。主要要注意以下两种情形：
	1. 当一个对象从collection对象（collection指的数组之类的集合）移除时，如果这个仅被collection对象拥有，那么移除操作了会被即可回收。所以如果要使用这个将要移除的对象，要先retain。
	2. 当“父”对象回收时。这和情形1类似。


##### Autorelease Pool
Autorelease Pool可以延后发送release消息给一个对象。发送一个autorelease消息给一个对象，相当于说这个对象在“一定时期”内都有效，“一定时期”后再release这个对象。

###### Autorelease Pool几个要点：

1. autorelease pool是一个 `NSAutoreleasePool` 对象。

2. 程序里的所有autorelease pool是以桟（stack）的形式组织的。新创建的pool位于桟的最顶端。当发送autorelease消息给一个对象时，这个对象被加到栈顶的那个pool中。发送drain给一个pool时，这个pool里所有对象都会受到release消息，而且如果这个pool不是位于栈顶，那么位于这个pool“上端”的所有pool也会受到drain消息。

3. 一个对象被加到一个pool很多次，只要多次发送autorelease消息给这个对象就可以；同时，当这个pool被回收时，这个对象也会收到同样多次release消息。简单地可以认为接收autorelease消息等同于：接收一个retain消息，同时加入到一个pool里；这个pool用来存放这些暂缓回收的对象；一旦这个pool被回收（drain），那么pool里面的对象会收到同样次数的release消息。

4. UIKit框架已经帮你自动创建一个 `autorelease pool` 。大部分时候，你可以直接使用这个pool，不必自己创建；所以你给一个对象发送autorelease消息，那么这个对象会加到这个UIKit自动创建的pool里。某些时候，可能需要创建一个pool：
	1. 没有使用UIKit框架或者其它内含 `autorelease pool`的框架，那么要使用pool，就要自己创建。
	2. 如果一个循环体要创建大量的临时变量，那么创建自己的pool可以减少程序占用的内存峰值。（如果使用UIKit的pool，那么这些临时变量可能一直在这个pool里，只要这个pool受到drain消息；完全不使用 `autorelease pool` 应该也是可以的，可能只是要发一些release消息给这些临时变量，所以使用 `autorelease pool` 还是方便一些）
	3. 创建线程时必须创建这个线程自己的 `autorelease pool`。

5. 使用alloc和init消息来创建pool，发送drain消息则表示这个pool不再使用。pool的创建和drain要在同一上下文中，比如循环体内。



