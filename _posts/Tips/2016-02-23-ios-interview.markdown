---
layout: post
date: 2016-02-23 23:52:02 +0800
title: iOS 面试题集锦
category: Tips
tags: iOS Tip 
image: /images/head-800x400/-28.png
description: 整理一些面试相关的知识，持续更新
toc: false
homepage: false
---

整理一些面试相关的知识。


### 1.简述OC中内存管理机制。与retain配对使用的方法是dealloc还是release，为什么？需要与alloc配对使用的方法是dealloc还是release，为什么？readwrite，readonly，assign，retain，copy，nonatomic 、atomic、strong、weak属性的作用？ 

管理机制：使用了一种叫做引用计数的机制来管理内存中的对象。OC中每个对象都对应着他们自己的引用计数，引用计数可以理解为一个整数计数器，当使用alloc方法创建对象的时候，持有计数会自动设置为1。当你向一个对象发送retain消息 时，持有计数数值会增加1。相反，当你像一个对象发送release消息时，持有计数数值会减小1。当对象的持有计数变为0的时候，对象会释放自己所占用的内存。

* retain(引用计数加1)->release（引用计数减1）
* alloc（申请内存空间）->dealloc(释放内存空间)
* readwrite: 表示既有getter，也有setter   (默认)
* readonly: 表示只有getter，没有setter
* retain: release旧的对象，将旧对象的值赋予输入对象，再提高输入对象的索引计数为1
* assign: 简单赋值，不更改索引计数    （默认）
* copy: 其实是建立了一个相同的对象,地址不同（retain：指针拷贝  copy：内容拷贝）
* strong:（ARC下的）和（MRC）retain一样    （默认）
* weak:（ARC下的）和（MRC）assign一样， weak当指向的内存释放掉后自动nil化，防止野指针
* unsafe_unretained 声明一个弱应用，但是不会自动nil化，也就是说，如果所指向的内存区域被释放了，这个指针就是一个野指针了。 autoreleasing 用来修饰一个函数的参数，这个参数会在函数返回的时候被自动释放。

* nonatomic:不考虑线程安全
* atomic:线程操作安全   （默认）

线程安全情况下的setter和getter：

{% highlight objc  %}
- (NSString *) value  {     
        @synchronized(self) {         
        return [[_value retain] autorelease];     
}} 
(void) setValue:(NSString *)aValue {     
   @synchronized(self) {         
   [aValue retain];         
   [_value release];         
   _value = aValue;     
}  }
{% endhighlight %}

### 2.类变量的@protected ,@private,@public,@package，声明各有什么含义？
* @private：作用范围只能在自身类
* @protected：作用范围在自身类和继承自己的子类  （默认）
* @public：作用范围最大，可以在任何地方被访问。
* @package：这个类型最常用于框架类的实例变量,同一包内能用，跨包就不能访问

### 3.线程是什么？进程是什么？二者有什么区别和联系？ 

**一个程序至少有一个进程,一个进程至少有一个线程：**

* 进程：一个程序的一次运行，在执行过程中拥有独立的内存单元，而多个线程共享一块内存
* 线程：线程是指进程内的一个执行单元。
* 联系：线程是进程的基本组成单位
* 区别：
	* (1)调度：线程作为调度和分配的基本单位，进程作为拥有资源的基本单位
	* (2)并发性：不仅进程之间可以并发执行，同一个进程的多个线程之间也可并发执行
	* (3)拥有资源：进程是拥有资源的一个独立单位，线程不拥有系统资源，但可以访问隶属于进程的资源.
	* (4)系统开销：在创建或撤消进程时，由于系统都要为之分配和回收资源，导致系统的开销明显大于创建或撤消线程时的开销。

> 举例说明：操作系统有多个软件在运行（QQ、office、音乐等），这些都是一个个进程，而每个进程里又有好多线程（比如QQ，你可以同时聊天，发送文件等）


### 4.谈谈你对多线程开发的理解？ios中有几种实现多线程的方法？

##### 好处：
* 1.使用线程可以把占据时间长的程序中的任务放到后台去处理
* 2.用户界面可以更加吸引人，这样比如用户点击了一个按钮去触发某些事件的处理，可以弹出一个进度条来显示处理的进度
* 3.程序的运行速度可能加快
* 4·在一些等待的任务实现上如用户输入、文件读写和网络收发数据等，线程就比较有用了。

##### 缺点：

* 1.如果有大量的线程,会影响性能,因为操作系统需要在它们之间切换。
* 2.更多的线程需要更多的内存空间。
* 3.线程的中止需要考虑其对程序运行的影响。
* 4.通常块模型数据是在多个线程间共享的，需要防止线程死锁情况的发生。

#### 实现多线程的方法：

* NSObject类方法
* NSThread
* NSOperation
* GCD

### 5.线程同步和异步的区别？IOS中如何实现多线程的同步？
* **异步**：举个简单的例子 就是游戏，游戏会有图像和背景音乐
* **同步**:是指一个线程要等待上一个线程执行完之后才开始执行当前的线程,上厕所
* NSOperationQueue：maxcurrentcount
* NSConditionLock
* GCD -> [(http://blog.csdn.net/onlyou930/article/details/8225906](http://blog.csdn.net/onlyou930/article/details/8225906)


### 6.假设有一个字符串aabcad，请写一段程序，去掉字符串中不相邻的重复字符串，即上述字符串处理之后的输出结果为：aabcd

{% highlight objc  %}
NSMutableString * str = [[NSMutableString alloc]initWithFormat;@“aabcad”];
for (int i = 0 ,i < str.length - 1 ;i++){
    unsigned char a = [str characterAtIndex:i];
    for (int j = i + 1 ,j < str.length ,j++){
        unsigned char b = [str characterAtIndex:j];
        if (a == b ){
            if (j == i + 1){
                }else{
                [str deleteCharactersInRange:NSMakeRange(j, 1)];
                }
            }
        }
    }
NSLog(@"%@",str);
{% endhighlight %}

### 7.获取一台设备唯一标识的方法有哪些？

[http://www.cnblogs.com/max5945/archive/2013/06/24/3152292.html](http://www.cnblogs.com/max5945/archive/2013/06/24/3152292.html)

* (1) UDID
* (2) UUID
* (3) MAC Address
* (4) OPEN UDID
* (5) 广告标识符
* (6) Vindor标示符

> ios7以后使用 keychain 
  
### 8.iOS类是否可以多继承？如果没有，那可以用其他方法实现吗？简述实现过程。

不可以多继承    用protocol实现

### 9.堆和栈的区别？

堆需要用户手动释放内存，而栈则是编译器自动释放内存

> 问题扩展：要知道OC中NSString的内存存储方式


### 10.iOS本地数据存储都有哪几种方式？
* NSKeyedArchiver    
* NSUserDefaults
* Write写入方式
* SQLite3
[http://blog.csdn.net/tianyitianyi1/article/details/7713103](http://blog.csdn.net/tianyitianyi1/article/details/7713103)

> 问题扩展：什么情况下使用什么样的数据存储

* 1.NSKeyedArchiver：采用归档的形式来保存数据，数据对象需要遵守NSCoding协议，对象对应的类必须提供encodeWithCoder:和initWithCoder:方法。缺点：只能一次性归档保存以及一次性解压。所以只能针对小量数据，对数据操作比较笨拙，如果想改动数据的某一小部分，需要解压或归档整个数据。
* 2.NSUserDefaults：用来保存应用程序设置和属性、用户保存的数据。用户再次打开程序或开机后这些数据仍然存在。NSUserDefaults可以存储的数据类型包括：NSData、NSString、NSNumber、NSDate、NSArray、NSDictionary。缺点：如果要存储其他类型，需要转换为前面的类型，才能用NSUserDefaults存储。
* 3.Write写入方式：永久保存在磁盘中。第一步：获得文件即将保存的路径：第二步：生成在该路径下的文件：第三步：往文件中写入数据：最后：从文件中读出数据：
* 4.SQLite：采用SQLite数据库来存储数据。SQLite作为一中小型数据库，应用ios中，跟前三种保存方式相比，相对比较复杂一些。


### 11.写出方法获取iOS内存使用情况。
[http://blog.sina.com.cn/s/blog_698415f20100yjlo.html](http://blog.sina.com.cn/s/blog_698415f20100yjlo.html)

{% highlight objc  %}
// 获取当前设备可用内存及所占内存的头文件
#import <sys/sysctl.h>
#import <mach/mach.h>
//
// 获取当前设备可用内存(单位：MB）
- (double)availableMemory
{
  vm_statistics_data_t vmStats;
  mach_msg_type_number_t infoCount = HOST_VM_INFO_COUNT;
  kern_return_t kernReturn = host_statistics(mach_host_self(), 
                                             HOST_VM_INFO, 
                                             (host_info_t)&vmStats, 
                                             &infoCount);
  if (kernReturn != KERN_SUCCESS) {
    return NSNotFound;
  }
  return ((vm_page_size *vmStats.free_count) / 1024.0) / 1024.0;
}
//
// 获取当前任务所占用的内存（单位：MB）
- (double)usedMemory
{
  task_basic_info_data_t taskInfo;
  mach_msg_type_number_t infoCount = TASK_BASIC_INFO_COUNT;
  kern_return_t kernReturn = task_info(mach_task_self(), 
                                       TASK_BASIC_INFO, 
                                       (task_info_t)&taskInfo, 
                                       &infoCount);
  if (kernReturn != KERN_SUCCESS
      ) {
    return NSNotFound;
  }
  return taskInfo.resident_size / 1024.0 / 1024.0;
}
{% endhighlight %}

> 问题扩展：如何利用Xcode观察内存使用情况

### 12.深拷贝和浅拷贝的理解？
[http://blog.sina.com.cn/s/blog_7b9d64af01019jq8.html](http://blog.sina.com.cn/s/blog_7b9d64af01019jq8.html)
[http://blog.sina.com.cn/s/blog_7b9d64af01019k6n.html](http://blog.sina.com.cn/s/blog_7b9d64af01019k6n.html)

* 对实例进行深拷贝时当前类需要实现NSCopying协议。
* 浅拷贝是复制出来一个跟原对象相同地址的对象
* 深拷贝时复制一个跟源对象不同地址的对象 改变源对象对新对象没有影响


### 13.怎样实现一个singleton的类。
[http://blog.csdn.net/zhugq_1988/article/details/8568033](http://blog.csdn.net/zhugq_1988/article/details/8568033)

> 问题扩展：单例的好处是什么？--> 节省内存 

### 14.什么是安全释放？

**置nil 再释放**

### 15.RunLoop是什么？
[http://blog.csdn.net/jjunjoe/article/details/8313016](http://blog.csdn.net/jjunjoe/article/details/8313016)

### 16.什么是序列化和反序列化，可以用来做什么？如何在OC中实现复杂对象的存储？
[http://blog.csdn.net/zjl201309/article/details/12707979](http://blog.csdn.net/zjl201309/article/details/12707979)

* 序列化是把对象转化成字节序列的过程  反序列化是把字节序列恢复成对象
* 将对象写到文件或者数据库里，并且能读取出来
* 遵循NSCoding协议 实现复杂对象的存储 实现该协议后可以对其进行打包或解包，转化成NSData


### 17.写一个标准宏MIN，这个宏输入两个参数并返回较小的一个？

{% highlight objc  %}
#define MIN(X,Y)  ((X)>(Y)?(Y):(X))
{% endhighlight %}

> 扩展：在定义宏的时候需要注意哪些问题？
> 1. 宏全部大写、 
> 2. 写在#import下，@interface上、
> 3. 结尾无分号


### 18.iOS有没有垃圾回收机制？简单阐述一下OC内存管理。

iOS没有垃圾回收机制  oc的内存管理是**谁创建谁释放**  程序中遇到retain 该对象引用计数+1 遇release该对象引用计数-1 retainCount为0时 内存释放


### 19.简述应用程序按Home键进入后台时的生命周期，以及从后台回到前台时的生命周期？
[http://blog.csdn.net/totogo2010/article/details/8048652](http://blog.csdn.net/totogo2010/article/details/8048652)

自己可以写个demo来测试一下
进入后台时

{% highlight objc  %}
-(void)applicationWillResignActive:(UIApplication *)application;
-(void)applicationDidEnterBackground:(UIApplication *)application;
{% endhighlight %}

进入前台时

{% highlight objc  %}
-(void)applicationDidEnterForeground:(UIApplication *)application;
-(void)applicationWillResignActive:(UIApplication *)application;
{% endhighlight %}

### 20.ViewController 的 alloc，loadView, viewDidLoad,viewWillAppear,viewDidUnload,dealloc、init分别是在什么时候调用的？在自定义ViewController的时候这几个函数里面应该做什么工作？
[http://www.xuebuyuan.com/672935.html](http://www.xuebuyuan.com/672935.html)
自己写代码测试加深理解

* alloc申请内存时调用
* loadView加载视图时调用
* ViewDidLoad视图已经加载后调用
* ViewWillAppear视图将要出现时调用
* ViewDidUnload视图已经加载但没有加载出来调用
* dealloc销毁该视图时调用
* init视图初始化时调用


### 21.描述应用程序的启动顺序。
1. 程序入口main函数创建UIApplication实例和UIApplication代理实例。
2. 在UIApplication代理实例中重写启动方法，设置第一ViewController。
3. 在第一ViewController中添加控件，实现应用程序界面。


### 22.为什么很多内置类如UITableViewControl的delegate属性都是assign而不是retain？请举例说明。

**防止循环引用**


### 23.使用UITableView时候必须要实现的几种方法？

{% highlight objc  %}
-(NSInteger)tableView:(UITableView *)tableView NumberOfRowsInSection:(NSInteger)section;
{% endhighlight %}
 这个方法返回每个分段(section)的行数，不同分段返回不同的行数可以用switch来做，如果是单个列表就直接返回单个你想要的函数即可。

{% highlight objc  %} - (UITableViewCell *)tableView:(UITableView *)tableView CellForRowAtIndexPath:(NSIndexPath)indexPath;
{% endhighlight %}
 这个方法是返回我们调用的每一个单元格。通过我们索引的路径的section和row来确定


### 24.写一个便利构造器。

{% highlight objc  %}
//id代表任意类型指针，这里代表Student *,类方法
+(id)studentWithName:(NSString *)newName  andAge:(int)newAge {     Student *stu=[[Student alloc]initName:newName andAge:newAge];     return [stu autorelease];//自动释放 }
{% endhighlight %}

### 25.UIImage初始化一张图片有几种方法？简述各自的优缺点。
[http://blog.sina.com.cn/s/blog_a843a8850101flo3.html](http://blog.sina.com.cn/s/blog_a843a8850101flo3.html)

三种：

*  `imageNamed:` 系统会先检查系统缓存中是否有该名字的Image，如果有的话，则直接返回，如果没有，则先加载图像到缓存，然后再返回。
*  `initWithContentsOfFile:` 系统不会检查系统缓存，而直接从文件系统中加载并返回。
*  `imageWithCGImage: scale: orientation` 当scale=1


### 26.回答person的retainCount值，并解释为什么

{% highlight objc  %}
Person * per = [[Person alloc] init];
self.person = per;
{% endhighlight %}

### 27.这段代码有什么问题吗:

{% highlight objc  %}
@implementation Person
- (void)setAge:(int)newAge {
	self.age = newAge;
}
@end
{% endhighlight %}

会造成死循环，正确写法：

{% highlight objc  %}
- (void)setAge:(int)newAge {
	if(_age){
		[_age release];
	}
	_age = [newAge retain];
}
{% endhighlight %}

> 扩展：知道如何正确写setter和getter方法

### 28.这段代码有什么问题,如何修改

{% highlight objc  %}
for (int i = 0; i < someLargeNumber; i++) { 
	NSString *string = @"Abc";//常量区
	string = [string lowercaseString];//新的堆区
	string = [string stringByAppendingString:@"xyz"];//新的堆区
	NSLog(@"%@", string);
}
{% endhighlight %}
在for循环里添加自动释放池
> 扩展：常量区的retaincount是怎么个情况


### 29.截取字符串”20 | http://www.baidu.com”中，”|”字符前面和后面的数据，分别输出它们。

使用方法`componentsSeparatedByString:`，示例如下：

{% highlight objc  %}
NSString * str = @“20 | http://www.baidu.com”;
for(NSString*s in [str componentsSeparatedByString:"|"]){
NSLog(@“%@“,s);
{% endhighlight %}

### 30.用obj-c写一个冒泡排序

{% highlight objc  %}
for（int i = 0, i < arr.count - 1,i++){
	for (int j = 0,j < arr.count - 1 - i;j++){
		int a = [[arr objectAtIndex:j]intValue];
		int b=[[arr objectAtIndex:j+1]intValue];
		if(a < b){
			[arr replaceObjectAtIndex:j withObject:[NSString stringWithFormat:@“%d”,b]];
			[arr replaceObjectAtIndex:j+1 withObject:[NSString stringWithFormat:@“%d”,a];
		}
	}
}
{% endhighlight %}


### 31.简述你对UIView、UIWindow和CALayer的理解

[http://blog.csdn.net/kuqideyupian/article/details/7731942](http://blog.csdn.net/kuqideyupian/article/details/7731942)
[http://o0o0o0o.iteye.com/blog/1728599](http://o0o0o0o.iteye.com/blog/1728599)


### 32.写一个完整的代理，包括声明，实现

> 注意手写的准确性


### 33.分析json、xml的区别？json、xml解析方式的底层是如何处理的？

[http://www.open-open.com/bbs/view/1324367918671](http://www.open-open.com/bbs/view/1324367918671)
[http://hi.baidu.com/fevelen/item/a25253ab76f766756cd455b6](http://hi.baidu.com/fevelen/item/a25253ab76f766756cd455b6)


### 34.ViewController 的 didReceiveMemoryWarning 是在什么时候被调用的？默认的操作是什么?

[http://blog.sina.com.cn/s/blog_68661bd80101nn6p.html](http://blog.sina.com.cn/s/blog_68661bd80101nn6p.html)


### 35.面向对象的三大特征，并作简单的介绍

**封装、继承、多态**

* 多态：父类指针指向子类对象。两种表现形式：重写（父子类之间）和重载（本类中）
OC的多态体现是：重写，没有重载这种表现形式

举例说明：
{% highlight objc  %}
// Parent类
@interface Parent : NSObject    //父类
- (void)simpleCall;
@end 
{% endhighlight %}

{% highlight objc  %}
// Child_A类
@interface Child_A : Parent   //子类  Child_A
@end 
@implementation Child_A
- (void)simpleCall
{
    NSLog(@"我是Child_A的simpleCall方法");
}
@end
{% endhighlight %}

{% highlight objc  %}
// Child_B类
@interface Child_B : Parent     //子类Child_B
@end
- (void)simpleCall
{
     NSLog(@"我是Child_的simpleCall方法");
}
@end
{% endhighlight %}
然后，我们就可以看到多态所展示的特性了：

{% highlight objc  %}
Parent * pa=[[Child_A alloc] init];// 父类指针指向子类Child_A对象
Parent * pb=[[Child_B alloc] init]; //父类指针指向子类Child_B对象
[pa simpleCall];// 显然是调用Child_A的方法
[pb simpleCall];// 显然是调用Child_B的方法
{% endhighlight %}

在OC中常看见的多态体现：

{% highlight objc  %}
//
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath  
 {  
     static NSString *CellWithIdentifier = @"Cell";  
      UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:CellWithIdentifier];  
      return cell;  
  }
{% endhighlight %}

(UITableViewCell *)指向cell子类对象


### 36.重写一个NSString类型的，retain方式声明name属性的setter和getter方法

{% highlight objc  %}
-(void)settetName:(NSString *)name{
	if(_name){
		[_name release];
	}
 	_name = [name retain];
}
-(NSString *)getterName{
	return [[_name retain]autorelease];
}
{% endhighlight %}

### 37.简述NotificationCenter、KVC、KVO、Delegate？并说明它们之间的区别？

[http://blog.csdn.net/zuoerjin/article/details/7858488](http://blog.csdn.net/zuoerjin/article/details/7858488)
[http://blog.sina.com.cn/s/blog_bf9843bf0101j5px.html](http://blog.sina.com.cn/s/blog_bf9843bf0101j5px.html)


### 38.What is lazy loading?

**懒加载模式**，只在用到的时候才去初始化。也可以理解成延时加载。我觉得最好也最简单的一个列子就是tableView中图片的加载显示了。一个延时载，避免内存过高，一个异步加载，避免线程堵塞


### 39.什么是Protocol？什么是代理？写一个委托的interface？委托的property声明用什么属性？为什么？

委托的property声明用什么属性是assign（防止循环引用）


### 40.分别描述类别（categories）和延展（extensions）是什么？以及两者的区别？继承和类别在实现中有何区别？为什么Category只能为对象添加方法，却不能添加成员变量？

* 考虑类目比继承的优点
* 类别是把类的实现方法分散到不同的文件中 也可以给类扩展新方法
* 延展是给类添加私有方法 只为自己类所见 所使用
* 继承可以扩展实例变量 而类别不能
* 类别如果可以添加成员变量 就跟继承没什么两样了  而且在上线的项目更新中 用类别笔继承更能维护项目的稳定性


### 41.Objective-C有私有方法么？私有变量呢？如多没有的话，有没有什么代替的方法？

OC没有私有方法 但是有私有变量 @property  私有方法可以用延展代替


### 42.#import、#include和@class有什么区别

* #import 系统文件、自定义文件引用 不用担心重复引用的问题
* #include 跟#import几乎一样 但是他需要注意不能重复引用
* @class 只是告诉系统有这个类 但是如果在实现类中用到这个类 需要重新用#import导入该类头文件 

### 43.谈谈你对MVC的理解？为什么要用MVC？在Cocoa中MVC是怎么实现的？你还熟悉其他的OC设计模式或别的设计模式吗？

mvc - model view controller ，避免了view与model 的强耦合 使代码更灵活 更容易维护 可复用 可扩展   OC其他设计模式有Notification 。target；action.  singleton delegate


### 44.如监测系统键盘的弹出

{% highlight objc  %}
//监听通知
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector( ) name:UIKeyboardWillShowNotification object:nil];
{% endhighlight %}
> 扩展：ios 弹出键盘挡住UITextView的解决方式


### 45.举出5个以上你所熟悉的ios  sdk库有哪些和第三方库有哪些？

AFWorking / WebKit / SQLite / Core Data / Address Book


### 46.如何将产品进行多语言发布？

[http://fengmm521.blog.163.com/blog/static/25091358201291645852889/](http://fengmm521.blog.163.com/blog/static/25091358201291645852889/)


### 47.如何将敏感字变成**

{% highlight objc  %}
	search = @"某某某";
    replace = @"***";
    range = [mstr rangeOfString:search];
    [mstr replaceCharactersInRange:range withString:replace]；    
    NSLog(@"%@",mstr);
{% endhighlight %}


### 48.objc中的减号与加号代表什么？

-表示实例方法
+表示类方法

### 49.单例目的是什么，并写出一个？

避免重复创建  节省内存空间
{% highlight objc  %}
static Model * model;
+(id)singleton{
	if(!model){
	 @synchronized(self){
		model = [[Model alloc]init];
		}
	}
	return model;
}
{% endhighlight %}

### 50.说说响应链

[http://www.tuicool.com/articles/6jmUje](http://www.tuicool.com/articles/6jmUje)

从手指触摸屏幕的地方的最上层控件是第一响应者，事件会沿着响应链一直向下传递直到被接受并作出处理











