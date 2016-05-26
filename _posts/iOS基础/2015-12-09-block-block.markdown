---
layout: post
title: 关于Block代码块
category: iOS基础
tags: iOS Block Objective-C
image: http://o7rxin1of.qnssl.com/images/head-800x400/-48.png
description: 本文主要来阐述下Block是个什么玩意
toc: true
homepage: false
---


#### Block是什么？

用一句话来概括就是带有**自动变量的匿名函数**。


* **匿名函数**
匿名函数顾名思义就是不带名字的函数，在C语言中不允许这样的方法存在，而在OC中的Block则可以用指针来直接调用一个函数，但虽说如此我们还是需要知道指针的名称。
* **自动变量**
自动变量在Block中的具体表现就是截获自动变量，来看下面这一段代码：

{% highlight objc  %}
int b = 0;
void (^blo)() = ^{
    NSLog(@"Input:b=%d",b);
};
b = 3;
blo();
//
// 输出：  Input:b=0
{% endhighlight %}
 
虽然我们在调用blo之前改变了b的值，但是输出的还是Block编译时候b的值，所以截获瞬间自动变量就是：**在Block中会保存变量的值，而不会随变量的值的改变而改变。**

我们再来看一段代码

{% highlight objc  %}
int b = 0;
    void (^blo)() = ^{
        b = 3;
    };
{% endhighlight %}

这段代码**编译出错**，编译器提示的大概就是不能在Block中改变变量的值。因为在Block中截获了变量的瞬间值以后就不能再改变变量的值，如果想要在Block中改变变量的值，那么我们只需要在变量声明的时候加上`__Block`修饰符，像这样

{% highlight objc  %}
__block int b = 0;
    void (^blo)() = ^{
        b = 3;
    };
{% endhighlight %}

然而这样的情况又是允许的：

{% highlight objc  %}
 NSMutableArray *array = [[NSMutableArray alloc]init];
    void (^blo)() = ^{
        [array addObject:@"Obj"];
    };
{% endhighlight %}

为什么呢，因为我们只是对截获的变量进行了操作，而没有进行赋值，所以对于截获变量，可以进行操作而不可以进行赋值。

* 还有一点需要注意，在Block中不可以对C语言数组进行操作，原因是：**不支持**

结合匿名函数和截获自动变量的特性，Block可以做很多事情，我们下面在看。

* * *

我们来具体看一下Block语法的书写，我们首先来看一个完整的Block：

{% highlight objc  %}
 ^ NSString *(NSString *a,NSString *b){
        return a;
    };
{% endhighlight %}

我们来分别解释下每一个部分都是什么东西：

> * “^”这个符号表示这是一个Block；
> * NSString *表示返回值。
> * (NSString a,NSString b)这个括号中是Block的参数，语法和C语言类似。

其实我们可以省略Block的返回值，像这样写：

{% highlight objc  %}
^ (NSString *a,NSString *b){
        return a;
    };
{% endhighlight %}

这样写和上面那种写法是一模一样的，其实如果没有参数列表我们甚至可以省略参数列表，像这样：

{% highlight objc  %}
^ {
        NSLog(@"我没有参数列表");
    };
{% endhighlight %}

如果把这段代码写完整，那么就是这样的：

{% highlight objc  %}
^void(void) {
        NSLog(@"我没有参数列表");
    };
{% endhighlight %}

为什么需要Block变量？我们可以这样理解，我们通过这个Block变量来获取Block的指针，然后通过这个指针就可以来使用Block函数。我们先来看一下如何声明一个Block变量

{% highlight objc  %}
int (^Blo)(NSString *s1,NSString *s2);
{% endhighlight %}

对照前面的Block函数，我们可以比较容易的理解各个部分的含义：

他们分别是：

> * 返回值
> * 变量名
> * 参数列表

好的，然后我们用上面讲到的Block语法来对这个Block变量进行赋值：

{% highlight objc  %}
int (^Blo)(NSString *s1,NSString *s2);
//
Blo = ^(NSString *s1,NSString *s2){
    return 1;
};
{% endhighlight %}

然后我们就可以将这个Block变量当作C语言函数来使用了。

#### 那么Block到底怎么用呢？

Block能够当作函数参数，首先我们声明一个Block类型变量 ，并加上typedef修饰符，像这样：

{% highlight objc  %}
typedef void(^Blo)(NSString *s1,UIColor *c);
{% endhighlight %}

这样我们就可以使用Blo来表示这个Block，然后我就可以将Blo加入到函数参数中，我们来声明一个函数：

{% highlight objc  %}
-(void)func:(Blo)BlockPra
    BlockPra(@"Str",[UIColor redColor]);
}
{% endhighlight %}

然后我们可以这样使用这个函数：

{% highlight objc  %}
[self func:^(NSString *s1, UIColor *c) {
        NSLog(@"%@",s1);
        self.view.backgroundColor = c;
    }];
{% endhighlight %}

是不是觉得十分眼熟，平时使用的许多回调当中大多都是这样的形式，可能其中其较多的就是网络回调了，我们只需要调用方法，然后在回调当中就可以对结果进行操作，很多苹果自己写的API都是使用了这样的方法，这样做的好处就是形式上十份简洁，当然像这种地方你使用delegate肯定也是可以的，但是表现上就没有Block那么简洁，使用起来也没有Block那么方便。

除此之外，Block还可以用来作为控制器之间的一个通信。
前面我们已经知道Blcok是一个匿名函数，同时也是一个指针，那么使用Block就可以弥补在iOS中函数传递的功能。通常是这么用的：

页面B的.h文件中定义了这样一个Block指针，然后声明了一个变量，像这样：

{% highlight objc  %}
typedef void(^Blo)(NSString *s1,UIColor *c);
@property (nonatomic, copy) Blo block;
{% endhighlight %}

然后我们在页面A当中有这么一段代码：

{% highlight objc  %}
ViewController *b = [[ViewController alloc]init];
//
__weak  ViewController *wself = self;
b.block = ^(NSString *s1,UIColor *c){
    NSLog(@"%@",s1);
    wself.view.backgroundColor = c;
};
[self.navigationController pushViewController:b animated:true];
{% endhighlight %}

然后在页面B的任意地方我们调用block变量，像这样：

{% highlight objc  %}
self.block(@"str",[UIColor redColor]);
{% endhighlight %}

都会在A页面中调用B页面传过来的参数，在A页面进行操作，对控制器A进行改变，这样的做法通常用做 控制器 **反向传值**。

* 在这里有一点需要注意就是Block的使用引起的循环引用。如果在Block中使用附有`__strong`修饰符的对象类型自动变量，那么当Block从栈复制到堆时，改对象为Block所有。这样容易引起循环引用，从而发生内存泄漏，然而我们只需要保证当前控制器也就是self在需要释放的时候正确释放就可以，所以我们再来看上面那段代码：

{% highlight objc  %}
__weak  ViewController *wself = self;
{% endhighlight %}

我们定义一个`wself`变量并加上`__weak`修饰符，在Block代码块中，所有需要self的地方都用wself来替代。这样就不会增加引用计数，所以Block持有self对象也就不会造成循环引用，从而造成内存泄漏。
不管是将Block当作函数参数，还是用来反向传值，其实都是对Block的本质，也就是“带有自动变量的匿名函数”的两个修饰，“带有自动变量”、“匿名函数”这两个特性 的应用。

总结一下Block到底是什么、用来干什么：

> * C++中的Struct（本文未提到）。
> * 用来弥补iOS中函数传递的功能。
> * 他是一段代码块的内存的指针。
> * 和delegate一样的功能，但是显的更加简洁。


#### 其他写得比较好的关于block的文章：
 
* [block一点也不神秘————如何利用block进行回调](http://blog.csdn.net/mobanchengshuang/article/details/11751671)

* [Block技巧与底层解析](http://www.jianshu.com/p/51d04b7639f1)



















