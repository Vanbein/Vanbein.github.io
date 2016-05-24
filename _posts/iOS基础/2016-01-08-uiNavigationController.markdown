---
layout: post
title: UINavigationController基础
category: iOS基础
tags: iOS
image: /images/head-800x400/-6.png
description: UINavigationController通常被我们称为导航栏，它是视图与视图之间联系沟通的桥梁，几乎所有app都用到了它。这里记录下它的基本使用.
homepage: false
toc: true
---

<!--more-->

## 一、UINavigationController 设置
UINavigationController通常被我们称为导航栏，它是视图与视图之间联系沟通的桥梁，几乎所有app都用到了它。下面我们来看一下如何建立一个navigation。

首先，我们通常新建工程是直接将视图控制器添加到window上，而现在有navigation以后，就多了一层：

{% highlight objc linenos %}
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions  
{  
    self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];  
	//
    self.window.backgroundColor = [UIColor whiteColor];  
    //至少要有一个被管理的试图控制器，这个控制器我们称作:导航控制器的根视图控制器RootViewController
    RootViewController *root = [[RootViewController alloc]init];  
    //先将rootVC添加在navigation上
    UINavigationController *nav = [[UINavigationController alloc]initWithRootViewController:root];
    //navigation加在window上      
    [_window setRootViewController:nav];  
    //
    [self.window makeKeyAndVisible];  
    return YES;  
}
{% endhighlight %}

这样我们的navigation就加载上去了。下面我们来设置navigation的属性：

{% highlight objc linenos %}
- (void)viewDidLoad  
{  
    [super viewDidLoad];  
    //
    //  获取到 UINavigationBar，
    UINavigationBar *bar = self.navigationController.navigationBar;
    //
    //  设置导航条样式
    //  默认的时白色半透明（有点灰的感觉）,
    //  UIBarStyleBlack,UIBarStyleBlackTranslucent,UIBarStyleBlackOpaque都是黑色半透明，
    //  其实它们有的是不透明有的是透明有的是半透明，但不知为何无效果
    [bar setBarStyle:UIBarStyleBlack];
    //
    //  设置navigationbar的半透明
    [bar setTranslucent:NO];
    //
    //  设置navigationbar上显示的标题 两种方式，推荐第一种 
    self.title = @"navigationController"; //1
    [self.navigationItem setTitle:@"navigationController"];  //2
    //
    //  设置navigationbar的tittle颜色
    [bar setTitleTextAttributes:@{NSForegroundColorAttributeName:[UIColor whiteColor]}];
    //  它的dictionary的key定义以及其对应的value类型如下：
    //  Keys for Text Attributes Dictionaries
	//  NSString *const UITextAttributeFont;                       value: UIFont
	//  NSString *const UITextAttributeTextColor;                 value: UIColor
	//  NSString *const UITextAttributeTextShadowColor;       value: UIColor
	//  NSString *const UITextAttributeTextShadowOffset;      value: NSValue wrapping a UIOffset struct.
    //  设置navigationbar左边按钮  
    self.navigationItem.leftBarButtonItem = [[UIBarButtonItem alloc]initWithBarButtonSystemItem:UIBarButtonItemStyleDone target:self action:Nil]; 
    // 
    //  设置navigationbar右边按钮 
    self.navigationItem.rightBarButtonItem = [[UIBarButtonItem alloc]initWithBarButtonSystemItem:UIBarButtonItemStylePlain target:self action:Nil]; 
    //  
    //  设置navigationbar上左右按钮字体颜色
    [bar setTintColor:[UIColor whiteColor]]; 
    //
    //  设置导航条的左右按钮
    //  先实例化创建一个UIBarButtonItem，然后把这个按钮赋值给self.navigationItem.leftBarButtonItem即可
    //  初始化文字的按钮类型有UIBarButtonItemStylePlain和UIBarButtonItemStyleDone两种类型，区别貌似不大
    //  UIBarButtonItem *barBtn1=[[UIBarButtonItem alloc]initWithTitle:@"左边" style:UIBarButtonItemStylePlain target:self action:@selector(changeColor)];
    //  self.navigationItem.leftBarButtonItem=barBtn1;
    //  我们还可以在左边和右边加不止一个按钮，,且可以添加任意视图，以右边为例
    //  添加多个其实就是rightBarButtonItems属性，注意还有一个rightBarButtonItem，前者是赋予一个UIBarButtonItem对象数组，所以可以显示多个。后者被赋值一个UIBarButtonItem对象，所以只能显示一个
    //  显示顺序，左边：按数组顺序从左向右；右边：按数组顺序从右向左
    //  可以初始化成系统自带的一些barButton，比如UIBarButtonSystemItemCamera是摄像机，还有Done，Reply等等，会显示成一个icon图标
    //  还可以initWithImage初始化成图片
    //  还可以自定义，可以是任意一个UIView
    //  UIBarButtonItem *barBtn2=[[UIBarButtonItem alloc]initWithBarButtonSystemItem:UIBarButtonSystemItemCamera target:self action:@selector(changeColor2)];
    //  UIBarButtonItem *barBtn3=[[UIBarButtonItem alloc]initWithImage:[UIImage imageNamed:@"logo-40@2x.png"] style:UIBarButtonItemStylePlain target:self action:@selector(changeColor3)];
    //  UIView *view4=[[UIView alloc]initWithFrame:CGRectMake(10, 10, 20, 20)];
    //  view4.backgroundColor=[UIColor blackColor];
    //  UIBarButtonItem *barBtn4=[[UIBarButtonItem alloc]initWithCustomView:view4];
    //  NSArray *arr1=[[NSArray alloc]initWithObjects:barBtn2,barBtn3,barBtn4, nil nil];
    //self.navigationItem.rightBarButtonItems=arr1;
    //
    //  设置导航栏背景图片
    //  其中forBarMetrics有点类似于按钮的for state状态，即什么状态下显示
    //  UIBarMetricsDefault - 竖屏横屏都有，横屏导航条变宽，则自动repeat图片
    //  UIBarMetricsCompact - 竖屏没有，横屏有，相当于之前老iOS版本里地UIBarMetricsLandscapePhone
    //  UIBarMetricsCompactPrompt和UIBarMetricsDefaultPrompt暂时不知道用处，
    //  官方解释是Applicable only in bars with the prompt property, 
    //  such as UINavigationBar and UISearchBar，
    [bar setBackgroundImage:[UIImage imageNamed:@"bg.png"] forBarMetrics:UIBarMetricsDefault];
    //
    //  如果图片太大会向上扩展侵占状态栏的位置，设置超出导航栏的部分不显示，
    //bar.clipsToBounds = YES;
	//
    //  设置隐藏，由此点击进入其他视图时导航条也会被隐藏，默认是NO
    //[self.navigationController setNavigationBarHidden:YES];
}
{% endhighlight %}

这里还有一个属性常用，就是：

{% highlight objc linenos %}
	NSArray *arr = [NSArray arrayWithObjects:@"1", @"2", nil];  
    UISegmentedControl *segment = [[UISegmentedControl alloc]initWithItems:arr];  
    self.navigationItem.titleView = segment;//设置navigation上的titleview  
{% endhighlight %}


!["UISegmentedControl"](/images/2016/02/navigation.png "UISegmentedControl")

对，我们看到中间的字变成了两个可选的按钮，这就是navigation的另一个属性：`navigationitem.titleview`。


## UINavigationController的工作原理

* UINavigationController通过栈的方式管理控制器的切换，控制入栈和出栈，来展示各个视图控制器

* UINavigationController的ControllersView里始终显示栈顶控制器的view

* viewControllers 属性存储了栈中的所有被管理的控制器

* navigationController属性，父类中的属性，每一个在栈中的控制器都能通过此属性，获取自己所在的UINavigationController对象

### 出栈和入栈的方法
 
* pushViewController: animated   //进入下一个视图控制器
* popViewControllerAnimated:    //返回上一个视图控制器
* popToViewController: animated    //返回到指定的视图控制器
* popToRootViewControllerAnimated     //返回到根视图控制器

### 常用属性有
* viewControllers    //所有处于栈中的控制器
* topViewController     //位于栈顶的控制器
* visibleViewController    //当前正在显示的控制器
* navigationBar     //导航条
* topViewController    代表当前navigation栈中最上层的VC，而visibleViewController代表当前可见的VC，它可能是topViewController，也可能是当前topViewController present出来的VC。因此UINavigationController的这两个属性通常情况下是一样，但也有可能不同。

#### 关于 UINavigationBar
* navigationBar--导航条，iOS7以后默认是透明的，iOS7以前默认是不透明的。
* navigationBar在透明情况下，与contentView会重合一部分区域
* navigationBar在不透明情况，ContentView跟在navigationBar下面
* navigationBar竖屏下默认高度44，横屏下默认高度32


#### 管理UINavigationItem

* UINavigationBar处理能管理一组UINavigationItem
* 与UINavigationController相似，UINavigationBar也是以栈的方式管理一组UINavigationItem。提供push和pop操作item
* 每个视图控制器都有一个navigationItem属性，navigationItem中设置的做按钮、右按钮、标题等，会随着控制器的显示，也显示到navigationBar上
* UINavigationItem属于MVC中的M，封装了要显示在UiNavigationBar上的数据
	* title： 标题
	* titleView ：标题视图
	* leftBarButtonItem ：左按钮
	* rightBarButtonItem ：右按钮

#### UIBarButtonItem
* UIBarButtonItem属于MVC的M，定义了UINavigationItem上按钮的触发事件，外观等
* - initWithBarButtonSystemItem：target：action：设置按钮样式及触发事件
* - initWithTiltle：style：target：action： 设置标题的触发事件
* - initWithImage：style：target：action：设置视图的触发事件
* tintColor  设置tintColor可以影响添加在导航条上的系统样式的按钮的颜色







