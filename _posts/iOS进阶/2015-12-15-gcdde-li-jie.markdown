---
layout: post
title: GCD的理解
category: iOS进阶
tags: iOS 多线程
image: /images/head-800x400/-40.png
description: GCD(Grand Central Dispatch)是iOS开发中的一大“利器“，需要仔细研究学习。
homepage: false
toc: true
---

GCD(Grand Central Dispatch)是iOS开发中的一大“利器“，需要仔细研究学习。


## 简介

### 什么是GCD

* 全称是Grand Central Dispatch，可译为“牛逼的中枢调度器”
* 纯C语言，提供了非常多强大的函数

### GCD的优势

* GCD是苹果公司为多核的并行运算提出的解决方案
* GCD会自动利用更多的CPU内核（比如双核、四核）
* GCD会自动管理线程的生命周期（创建线程、调度任务、销毁线程）
* 程序员只需要告诉GCD想要执行什么任务，不需要编写任何线程管理代码

### GCD中有2个核心概念

* 任务：执行什么操作
* 队列：用来存放任务

### GCD的使用就2个步骤

* 定制任务
	+ 确定想做的事情
* 将任务添加到队列中
	* GCD会自动将队列中的任务取出，放到对应的线程中执行
	* 任务的取出遵循队列的FIFO原则：先进先出，后进后出

### 执行任务
#### GCD中有2个用来执行任务的常用函数
* 用同步的方式执行任务

{% highlight objc linenos %}
dispatch_sync(dispatch_queue_t queue, dispatch_block_t block);
{% endhighlight %}

* 用异步的方式执行任务


{% highlight objc linenos %}
dispatch_async(dispatch_queue_t queue, dispatch_block_t block);
{% endhighlight %}

#### 同步和异步的区别
> 同步：只能在当前线程中执行任务，不具备开启新线程的能力
>
> 异步：可以在新的线程中执行任务，具备开启新线程的能力

### 队列的类型
#### 并发队列（Concurrent Dispatch Queue）
* 可以让多个任务并发（同时）执行（自动开启多个线程同时执行任务）
* 并发功能只有在异步（dispatch_async）函数下才有效

#### 串行队列（Serial Dispatch Queue）
* 让任务一个接着一个地执行（一个任务执行完毕后，再执行下一个任务）


### 创建队列
#### 创建队列的方法

{% highlight objc linenos %}
// 使用dispatch_queue_create函数创建队列
dispatch_queue_t dispatch_queue_create(const char *label, dispatch_queue_attr_t attr); 
// const char *label 队列名称  // dispatch_queue_attr_t attr 队列的类型
{% endhighlight %}

#### 创建并发队列
* 使用函数创建并发队列

{% highlight objc linenos %}
// 创建并发队列
dispatch_queue_t queue = dispatch_queue_create("com.zmj.queue", DISPATCH_QUEUE_CONCURRENT);
{% endhighlight %}

* 获取已系统创建好的并发队列
	+ GCD默认已经提供了全局的并发队列，供整个应用使用，可以无需手动创建
	+ 使用dispatch_get_global_queue函数获得全局的并发队列

{% highlight objc linenos %}
dispatch_queue_t dispatch_get_global_queue(
dispatch_queue_priority_t priority,
unsigned long flags); 
// priority 队列的优先级  // flags 此参数暂时无用，用0即可
//
// 获得全局并发队列
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
{% endhighlight %}

* 全局并发队列的优先级

{% highlight objc linenos %}
#define DISPATCH_QUEUE_PRIORITY_HIGH 2 // 高
#define DISPATCH_QUEUE_PRIORITY_DEFAULT 0 // 默认（中）
#define DISPATCH_QUEUE_PRIORITY_LOW (-2) // 低
#define DISPATCH_QUEUE_PRIORITY_BACKGROUND INT16_MIN // 后台
{% endhighlight %}

#### 创建串行队列

* 使用dispatch_queue_create函数创建串行队列

{% highlight objc linenos %}
// 创建串行队列（队列类型传递NULL或者DISPATCH_QUEUE_SERIAL）
dispatch_queue_t queue = dispatch_queue_create("com.zmj.queue", NULL);
{% endhighlight %}

* 使用主队列（跟主线程相关联的队列）
	+ 主队列是GCD自带的一种特殊的串行队列
	+ 放在主队列中的任务，都会放到主线程中执行

{% highlight objc linenos %}
// 使用dispatch_get_main_queue()获得主队列
dispatch_queue_t queue = dispatch_get_main_queue();
{% endhighlight %}

### 线程间通信示例
* 从子线程回到主线程

{% highlight objc linenos %}
dispatch_async(
dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    // 执行耗时的异步操作...
      dispatch_async(dispatch_get_main_queue(), ^{
        // 回到主线程，执行UI刷新操作
        });
});
{% endhighlight %}

## 其它常用函数

#### GCD中还有个用来执行任务的函数：
* 在前面的任务执行结束后它才执行，而且它后面的任务等它执行完成之后才会执行
* 这个queue不能是全局的并发队列

{% highlight objc linenos %}
dispatch_barrier_async(dispatch_queue_t queue, dispatch_block_t block);
{% endhighlight %}

* 实例代码

{% highlight objc linenos %}
- (void)barrier
{
    dispatch_queue_t queue = dispatch_queue_create("12312312", DISPATCH_QUEUE_CONCURRENT);
//
    dispatch_async(queue, ^{
        NSLog(@"----1-----%@", [NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"----2-----%@", [NSThread currentThread]);
    });
//
    dispatch_barrier_async(queue, ^{
        NSLog(@"----barrier-----%@", [NSThread currentThread]);
    });
//
    dispatch_async(queue, ^{
        NSLog(@"----3-----%@", [NSThread currentThread]);
    });
    dispatch_async(queue, ^{
        NSLog(@"----4-----%@", [NSThread currentThread]);
    });
}
{% endhighlight %}

#### iOS常见的延时执行

* 调用NSObject的方法

{% highlight objc linenos %}
[self performSelector:@selector(run) withObject:nil afterDelay:2.0];
// 2秒后再调用self的run方法
{% endhighlight %}

* 使用GCD函数

{% highlight objc linenos %}
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    // 2秒后执行这里的代码...
});
{% endhighlight %}

* 使用NSTimer

{% highlight objc linenos %}
[NSTimer scheduledTimerWithTimeInterval:2.0 target:self selector:@selector(test) userInfo:nil repeats:NO];
{% endhighlight %}


#### 使用dispatch_once函数能保证某段代码在程序运行过程中只被执行1次

{% highlight objc linenos %}
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
    // 只执行1次的代码(这里面默认是线程安全的)
});
{% endhighlight %}

#### 使用dispatch_apply函数能进行快速迭代遍历

{% highlight objc linenos %}
dispatch_apply(10, dispatch_get_global_queue(0, 0), ^(size_t index){
    // 执行10次代码，index顺序不确定
});
{% endhighlight %}

#### 快速迭代的应用-文件剪切
* 传统文件剪切需要遍历for循环，效率低
* 快速迭代，让循环里面的任务同时执行

{% highlight objc linenos %}
// 快速迭代实现文件剪切
- (void)apply
{
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    NSString *from = @"/Users/zhamengjun/Desktop/From";
    NSString *to = @"/Users/zhamengjun/Desktop/To";
//
    NSFileManager *mgr = [NSFileManager defaultManager];
    NSArray *subpaths = [mgr subpathsAtPath:from];
//
    dispatch_apply(subpaths.count, queue, ^(size_t index) {
        NSString *subpath = subpaths[index];
        NSString *fromFullpath = [from stringByAppendingPathComponent:subpath];
        NSString *toFullpath = [to stringByAppendingPathComponent:subpath];
        // 剪切
        [mgr moveItemAtPath:fromFullpath toPath:toFullpath error:nil];
//
        NSLog(@"%@---%@", [NSThread currentThread], subpath);
    });
}
{% endhighlight %}

{% highlight objc linenos %}
// 传统文件剪切
- (void)moveFile
{
    NSString *from = @"/Users/xiaomage/Desktop/From";
    NSString *to = @"/Users/xiaomage/Desktop/To";
//
    NSFileManager *mgr = [NSFileManager defaultManager];
    NSArray *subpaths = [mgr subpathsAtPath:from];
//
    for (NSString *subpath in subpaths) {
        NSString *fromFullpath = [from stringByAppendingPathComponent:subpath];
        NSString *toFullpath = [to stringByAppendingPathComponent:subpath];
//       dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            // 剪切
            [mgr moveItemAtPath:fromFullpath toPath:toFullpath error:nil];
        });
    }
}
{% endhighlight %}

#### 队列组
* 如果有这么1种需求,首先：分别异步执行2个耗时的操作,其次：等2个异步操作都执行完毕后，再回到主线程执行操作
* 如果想要快速高效地实现上述需求，可以考虑用队列组

{% highlight objc linenos %}
dispatch_group_t group =  dispatch_group_create();
dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    // 执行1个耗时的异步操作
});
dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    // 执行1个耗时的异步操作
});
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
    // 等前面的异步操作都执行完毕后，回到主线程...
});
{% endhighlight %}

##### 队列组的应用---合成图片

{% highlight objc linenos %}
- (void)group
{
//
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    // 创建一个队列组
    dispatch_group_t group = dispatch_group_create();
//
    // 1.下载图片1
    dispatch_group_async(group, queue, ^{
        // 图片的网络路径
        NSURL *url = [NSURL URLWithString:@"http://img.pconline.com.cn/images/photoblog/9/9/8/1/9981681/200910/11/1255259355826.jpg"];
//
        // 加载图片
        NSData *data = [NSData dataWithContentsOfURL:url];
//
        // 生成图片
        self.image1 = [UIImage imageWithData:data];
    });
//
    // 2.下载图片2
    dispatch_group_async(group, queue, ^{
        // 图片的网络路径
        NSURL *url = [NSURL URLWithString:@"http://pic38.nipic.com/20140228/5571398_215900721128_2.jpg"];
//
        // 加载图片
        NSData *data = [NSData dataWithContentsOfURL:url];
//
        // 生成图片
        self.image2 = [UIImage imageWithData:data];
    });
//
    // 3.将图片1、图片2合成一张新的图片
    dispatch_group_notify(group, queue, ^{
        // 开启新的图形上下文
        UIGraphicsBeginImageContext(CGSizeMake(100, 100));
//
        // 绘制图片
        [self.image1 drawInRect:CGRectMake(0, 0, 50, 100)];
        [self.image2 drawInRect:CGRectMake(50, 0, 50, 100)];
//
        // 取得上下文中的图片
        UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
//
        // 结束上下文
        UIGraphicsEndImageContext();
//
        // 回到主线程显示图片
        dispatch_async(dispatch_get_main_queue(), ^{
            // 4.将新图片显示出来
            self.imageView.image = image;
        });
    });
}
{% endhighlight %}


> 参考文章：
>
> * [zhazha - GCD](http://www.jianshu.com/p/c56b614db49d)











