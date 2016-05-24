---
layout: post
title: NSOperation的基本使用
category: iOS进阶
tags: iOS 多线程
image: /images/head-800x400/-39.png
description: NSOperation是在GCD的基础上进行的一层面向对象的包装.，需要仔细研究学习。
homepage: false
toc: true
---


NSOperation:是在GCD的基础上进行的一层面向对象的包装.

### 核心概念:

任务和队列.和GCD基本上是一样的,只不过更加的面向对象.用起来比较爽.

### NSOperation的作用

* 配合使用NSOperation和NSOperationQueue也能实现多线程编程
* NSOperation:就是任务
* NSOperationQueue:就是队列

### NSOperation的子类

* NSOperation是个抽象类，并不具备封装操作的能力，必须使用它的子类

### 使用NSOperation子类的方式有3种:

#### 1. NSInvocationOperation,  不常用

* 第一个参数：目标对象
* 第二个参数：该操作要调用的方法，最多接受一个参数
* 第三个参数：调用方法传递的参数，如果方法不接受参数，那么该值传nil

{% highlight objc linenos %}
 //1.封装任务
    NSInvocationOperation *op1 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run) object:nil];
    //2.要想执行任务必须调用start
    [op1 start];
//
    NSInvocationOperation *op2 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run2) object:nil];
    [op2 start];
}
- (void)run
{
    NSLog(@"%@", [NSThread currentThread]);
}
- (void)run2
{
    NSLog(@"%@", [NSThread currentThread]);
}
{% endhighlight %}

#### 2. NSBlockOperation,

NSBlockOperation提供了一个类方法，在该类方法中封装操作

{% highlight objc linenos %}
//1. 封装任务
  NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{      
      NSLog(@"1---%@", [NSThread currentThread]);
  }];
  //2.追加其它任务
  //注意: 在没有队列的情况下, 如果给BlockOperation追加其它任务, 那么其它任务会在子线程中执行
  [op1 addExecutionBlock:^{
      NSLog(@"2---%@", [NSThread currentThread]);
  }];
  [op1 addExecutionBlock:^{
      NSLog(@"3---%@", [NSThread currentThread]);
  }];
  //3.启动任务
  [op1 start];
{% endhighlight %}

> **注意:**
> 
> 默认情况下，调用了start方法后并不会开一条新线程去执行操作，而是在当前线程同步执行操作,只有将NSOperation放到一个NSOperationQueue中，才会异步执行操作


#### 3. 自定义NSOperation

**自定义NSOperation的步骤:**

* 自定义一个继承自NSOperation的子类,通过重写 `- (void)main` 方法实现封装操作,在里面实现想执行的任务.
* 只要将任务添加到队列中, 那么队列在执行自定义任务的时候,就会自动调用main方法

{% highlight objc linenos %}
//1.创建队列
 NSOperationQueue *queue = [[NSOperationQueue alloc] init];
  //2.创建任务
  //自定义任务的好处: 提高代码的复用性
 YFYOperation *op1 = [[YFYOperation alloc] init];
 YFYOperation *op2 = [[YFYOperation alloc] init];
  //3.添加任务到队列
 [queue addOperation:op1];
 [queue addOperation:op2];
{% endhighlight %}
 
* 重写 `- (void)main` 方法的注意点:
	* 自己创建自动释放池（因为如果是异步操作，无法访问主线程的自动释 放池）
	* 经常通过 `- (BOOL)isCancelled` 方法检测操作是否被取消，对取消做出响应

### NSOperationQueue
#### NSOperationQueue的作用
* NSOperation可以调用start方法来执行任务，但默认是同步执行的,如果将NSOperation添加到
* NSOperationQueue（操作队列）中，系统会自动异步执行NSOperation中的操作

### NSOperation和NSOperationQueue实现多线程的具体步骤:

1. 先将需要执行的操作封装到一个NSOperation对象中,
2. 然后将NSOperation对象添加到NSOperationQueue中,
3. 系统会自动将NSOperationQueue中的NSOperation取出来,
4. 将取出的NSOperation封装的操作放到一条新线程中执行;

#### 自定义NSOperation与NSBlockOperation

{% highlight objc linenos %}
-(void)customOperation
{
    //1.创建队列
    NSOperationQueue *queue = [[NSOperationQueue alloc]init];
    //2.封装操作
    //好处：1.信息隐蔽
    //2.代码复用
    //
    XMGOperation *op1 = [[XMGOperation alloc]init];
    XMGOperation *op2 = [[XMGOperation alloc]init];
    //3.添加操作到队列中
    [queue addOperation:op1];
    [queue addOperation:op2];
}
{% endhighlight %}

#### NSOperationQueue与NSBlockOperation

{% highlight objc linenos %}
//1.创建队列
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
//2.创建任务
NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
   NSLog(@"1 == %@", [NSThread currentThread]);
}];
NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
   NSLog(@"2 == %@", [NSThread currentThread]);
}];
[queue addOperationWithBlock:^{
   NSLog(@"3 == %@", [NSThread currentThread]);
}];
/*如果是使用block来封装任务, 那么有一种更简便的方法,只要利用队列调用addOperationWithBlock:方法, 系统内部会自动封装成一个NSBlockOperation,
然后再添加到队列中*/
// 3.添加任务到队列
[queue addOperation:op1];
[queue addOperation:op2];
{% endhighlight %}

#### NSOperationQueue与 NSInvocationOperation

{% highlight objc linenos %}
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
     //2.创建任务
     //只要是自己创建的队列, 就会在子线程中执行,而且默认就是并发执行
    NSInvocationOperation *op1 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(download1) object:nil];
    NSInvocationOperation *op2 = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(download2) object:nil];
    //3.添加任务到队列中,只要将任务添加到队列中, 队列会自动调用start
    [queue addOperation:op1];
    [queue addOperation:op2];
- (void)download1
{
    NSLog(@"1 == %@", [NSThread currentThread]);
}
- (void)download2
{
    NSLog(@"2 == %@", [NSThread currentThread]);
}
{% endhighlight %}

### 最大并发数

**什么是并发数?**

* 同时执行的任务数,比如，同时开3个线程执行3个任务，并发数就是3

最大并发数的相关方法:

{% highlight objc linenos %}
- (NSInteger)maxConcurrentOperationCount;
- (void)setMaxConcurrentOperationCount:(NSInteger)cnt;
{% endhighlight %}

自己创建的队列默认是并发, 如果按下面设置，就是串行

{% highlight objc linenos %}
NSOperationQueue *queue = [[NSOperationQueue alloc] init];
queue.maxConcurrentOperationCount = 1;//
{% endhighlight %}
> 注意: 不能设置为0, 如果设置为0就不执行任务,
> 默认情况下maxConcurrentOperationCount = -1
> 在开发中并发数最多尽量不要超过5~6条

### 队列的取消、暂停、恢复
* 只要设置队列的suspended为YES, 那么就会暂停队列中其它任务的执行

{% highlight objc linenos %}
self.queue.suspended = YES;
{% endhighlight %}

也就是说不会再继续执行没有执行到得任务

* 注意: 设置为暂停之后, 不会立即暂停,会继续执行当前正在执行的任务, 直到当前任务执行完毕, 就不会执行下一个任务了,也就是说, 暂停其实是暂停下一个任务, 而不能暂停当前任务

* 注意: 暂停是可以恢复的,只要设置队列的suspended为NO, 那么就会恢复队列中其它任务的执行
取消队列中所有的任务的执行

{% highlight objc linenos %}
[self.queue cancelAllOperations];
{% endhighlight %}

取消和暂停一样, 是取消后面的任务, 不能取消当前正在执行的任务

* 注意: 取消是不可以恢复的

### 操作依赖
* NSOperation之间可以设置依赖来保证执行顺序,比如一定要让操作A执行完后，才能执行操作B，可以这么写

{% highlight objc linenos %}
[operationB addDependency:operationA]; // 操作B依赖于操作A
{% endhighlight %}

* 注意：不能相互依赖,比如A依赖B，B依赖A.
* 可以在不同 queue 的 NSOperation 之间创建依赖关系

{% highlight objc linenos %}
// 1.创建队列
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    NSOperationQueue *queue2 = [[NSOperationQueue alloc] init];
//
    // 2.创建任务
    NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"1-------%@", [NSThread currentThread]);
    }];
    NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"2-------%@", [NSThread currentThread]);
    }];
    NSBlockOperation *op3 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"3-------%@", [NSThread currentThread]);
    }];
    NSBlockOperation *op4 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"4-------%@", [NSThread currentThread]);
        for (int i = 0; i < 1000; i++) {
            NSLog(@"%i", i);
        }
    }];
    NSBlockOperation *op5 = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"5-------%@", [NSThread currentThread]);
    }];
    // 3.添加依赖
    [op5 addDependency:op1];
    [op5 addDependency:op2];
    [op5 addDependency:op3];
    [op5 addDependency:op4];
    // 4.监听op4什么时候执行完毕
    op4.completionBlock = ^{
        NSLog(@"op4中所有的操作都执行完毕了");
    };
   // 5.添加任务到队列
    [queue addOperation:op1];
    [queue addOperation:op2];
    [queue2 addOperation:op3];
    [queue2 addOperation:op4];
    [queue addOperation:op5];
{% endhighlight %}

## NSOperation实现线程间通信
### （1）开子线程下载图片

{% highlight objc linenos %}
   //1.创建队列
   NSOperationQueue *queue = [[NSOperationQueue alloc]init];
//
   //2.使用简便方法封装操作并添加到队列中
   [queue addOperationWithBlock:^{
//
   //3.在该block中下载图片
   NSURL *url = [NSURL URLWithString:@"http://news.51sheyuan.com/uploads/allimg/111001/133442IB-2.jpg"];
   NSData *data = [NSData dataWithContentsOfURL:url];
   UIImage *image = [UIImage imageWithData:data];
   NSLog(@"下载图片操作--%@",[NSThread currentThread]);
//
   //4.回到主线程刷新UI
   [[NSOperationQueue mainQueue] addOperationWithBlock:^{
       self.imageView.image = image;
       NSLog(@"刷新UI操作---%@",[NSThread currentThread]);
   }];
}];
{% endhighlight %}

### （2）下载多张图片合成综合案例（设置操作依赖）

{% highlight objc linenos %}
//02 综合案例
(void)download2
{
	NSLog(@"----");
	//1.创建队列
	NSOperationQueue *queue = [[NSOperationQueue alloc]init];
//
	//2.封装操作下载图片1
	NSBlockOperation *op1 = [NSBlockOperation blockOperationWithBlock:^{
//
 	NSURL *url = [NSURL URLWithString:@"http://h.hiphotos.baidu.com/zhidao/pic/item/6a63f6246b600c3320b14bb3184c510fd8f9a185.jpg"];
  	NSData *data = [NSData dataWithContentsOfURL:url];
//
	//拿到图片数据
	self.image1 = [UIImage imageWithData:data];
}];
//
	//3.封装操作下载图片2
	NSBlockOperation *op2 = [NSBlockOperation blockOperationWithBlock:^{
    NSURL *url = [NSURL URLWithString:@"http://pic.58pic.com/58pic/13/87/82/27Q58PICYje_1024.jpg"];
    NSData *data = [NSData dataWithContentsOfURL:url];
//
    //拿到图片数据
    self.image2 = [UIImage imageWithData:data];
}];
//
//4.合成图片
NSBlockOperation *combine = [NSBlockOperation blockOperationWithBlock:^{
//
    //4.1 开启图形上下文
    UIGraphicsBeginImageContext(CGSizeMake(200, 200));
//
    //4.2 画image1
    [self.image1 drawInRect:CGRectMake(0, 0, 200, 100)];
//
    //4.3 画image2
    [self.image2 drawInRect:CGRectMake(0, 100, 200, 100)];
//
    //4.4 根据图形上下文拿到图片数据
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
//  NSLog(@"%@",image);
//
    //4.5 关闭图形上下文
    UIGraphicsEndImageContext();
//
    //7.回到主线程刷新UI
    [[NSOperationQueue mainQueue]addOperationWithBlock:^{
        self.imageView.image = image;
        NSLog(@"刷新UI---%@",[NSThread currentThread]);
    }];
}];
//5.设置操作依赖
[combine addDependency:op1];
[combine addDependency:op2];
//6.添加操作到队列中执行
[queue addOperation:op1];
[queue addOperation:op2];
[queue addOperation:combine];
}
{% endhighlight %}













