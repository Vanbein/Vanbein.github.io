---
layout: post
title: 我的Objective-C编码规范
category: iOS基础
tags: Objective-C
image: /images/head-800x400/-55.png
description: 这是我根据网络上的一些编码规范，配合自己的偏好，总结的Objective-C编码规范
homepage: true
toc: true
---

### 1. 空格与行宽

(1) 缩进使用 4 个空格,检查 Xcode 中缩进部分的设置

![image](/images/2016/04/text.png)

(2) 行宽为 100 个字符,在 Xcode 中打开行宽提醒

![image](/images/2016/04/text2.png)

### 2. 语言

使用US英语。

应该：

{% highlight objc  %}
UIColor *myColor = [UIColor whiteColor];  
{% endhighlight %}

不应该：

{% highlight objc  %}
UIColor *myColour = [UIColor whiteColor];  
{% endhighlight %}

 
### 3. 代码组织

使用 `#pragma mark -` 来分类方法

* 不同功能组的方法
* protocol/delegate 的实现
* 对父类方法的重写
* 私有方法组

一般建议按照以下示例来分组代码，注意顺序，排序的原则是：越是对其它对象有影响的方法应越靠前，越是私有的方法越靠后。

{% highlight objc  %}
#pragma mark - Instance Life Cycle
- (instancetype)init {..}
- (void)dealloc {..}

#pragma mark - View Life Cycle
- (void)viewDidLoad {..}
- (void)viewWillAppear:(BOOL)animated {..}
- (void)didReceiveMemoryWarning {..}
- 
#pragma mark - Custom Accessors
- (void)setCustomProperty:(id)value {..}
- (id)customProperty {..}
#pragma mark - SuperClass Override Methods- (void)someMethod {..}
#pragma mark - IBActions
- (IBAction)submitData:(id)sender {..}
 
#pragma mark - Public Methods
- (void)publicMethod {..}

#pragma mark - Private Methods
- (void)privateMethod {..}

#pragma mark - Protocol conformance

#pragma mark - UITextFieldDelegate

#pragma mark - UITableViewDataSource

#pragma mark - UITableViewDelegate

#pragma mark - NSCopying
- (id)copyWithZone:(NSZone *)zone {}

#pragma mark - NSObject
- (NSString *)description {}
{% endhighlight %}

### 4. 命名

Apple命名规则尽可能坚持，特别是与这些相关的 [memory management rules](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html) 。

> 命名原则
> 1. 清晰
>     * 清晰并简短，但不为了简短牺牲清晰
>     * 不简写事物名称，即使它很长
>     * 避免有歧义的命名
> 2. 一致性
>     * 在整个工程中保持命名一致性，如果不确定，查找工程中已有的命名定义
>     * 一致性对于用到了多态的类尤其重要,做同样事的方法应有相同的命名
   
长的，描述性的方法和变量命名是好的，命名使用**驼峰式**，命名中应避免出现数字

应该：

{% highlight objc  %}
UIButton *settingsButton;  
{% endhighlight %}

不应该：

{% highlight objc  %}
UIButton *setBtn;  
{% endhighlight %}


* **类名首字母大写**，类名前添加三个大学字母作为个性化前缀，比如"VNB"，

* **方法首字母小写**，方法中的参数首字母小写，同时尽量让方法的命名读起来像一句话，能够传达出方法的意思，同时取值方法前不要加前缀"get"

* **变量名小写字母开头**

{% highlight objc  %}
int chosenButtonIndex = [self.cardButtons indexOfObject:sender];
{% endhighlight %}

* **常量以小写字母k开头，后续首字母大写**

{% highlight objc  %}
static NSTimeInterval const kVNBTutorialViewControllerNavigationFadeAnimationDuration = 0.3;  
{% endhighlight %}

* **通知**（ Notifications ）

通知常用于在模块间传递消息，所以通知要尽可能地表示出发生的事件，通知的命名范式是：

[ 触发通知的类名 ] + [Did/Will] + [ 动作 ] + Notification

示例如下：

{% highlight objc  %}
NSApplicationDidBecomeActiveNotification
NSWindowDidMiniaturizeNotification
NSTextViewDidChangeSelectionNotification
NSColorPanelColorDidChangeNotification
{% endhighlight %}

### 5. 变量

变量尽量以描述性的方式来命名。单个字符的变量命名应该尽量避免，除了在for()循环。

当使用属性时，实例变量应该使用 `self.` 来访问和改变。这就意味着所有属性将会视觉效果不同，因为它们前面都有 `self.`。

但有一个特例：在初始化方法里，实例变量(例如，_variableName)应该直接被使用来避免getters/setters潜在的副作用。

应该尽量使用**属性**来声明全局变量以保持代码一致，非要声明的时候，全局变量应该在最前面加下划线，局部变量不应该包含下划线。

应该：

{% highlight objc  %}
@interface VNBTutorial : NSObject  
@property (strong, nonatomic) NSString *tutorialName;  
@end  
{% endhighlight %}

不应该：

{% highlight objc  %}
@interface VNBTutorial : NSObject {  
  NSString *_tutorialName;  
}  
{% endhighlight %}

### 6. 属性特性

所有属性特性应该显式地列出来，有助于新手阅读代码。属性特性的顺序应该是 storage、atomicity，与在 Interface Builder 连接UI元素时自动生成代码一致。

应该：

{% highlight objc  %}
@property (weak, nonatomic) IBOutlet UIView *containerView;  
@property (strong, nonatomic) NSString *tutorialName;  
{% endhighlight %}

不应该：

{% highlight objc  %}
@property (nonatomic, weak) IBOutlet UIView *containerView;  
@property (nonatomic) NSString *tutorialName;
{% endhighlight %}
  
* NSString应该使用copy而不是strong的属性特性。

为什么？即使你声明一个NSString的属性，有人可能传入一个NSMutableString的实例，然后在你没有注意的情况下修改它。

应该：

{% highlight objc  %}
@property (copy, nonatomic) NSString *tutorialName;  
{% endhighlight %}

不应该：

{% highlight objc  %}
@property (strong, nonatomic) NSString *tutorialName;  
{% endhighlight %}

### 7. 点语法

点语法是一种很方便封装访问方法调用的方式。当你使用点语法时，通过使用 getter 或 setter 方法，属性仍然被访问或修改。想了解更多，[阅读这里](https://developer.apple.com/library/ios/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/EncapsulatingData/EncapsulatingData.html)。

点语法应该**总是**被用来访问和修改属性，因为它使代码更加简洁。[]符号更偏向于用在其他例子。

应该：

{% highlight objc  %}
NSInteger arrayCount = [self.array count];  
view.backgroundColor = [UIColor orangeColor];  
[UIApplication sharedApplication].delegate;  
{% endhighlight %}

不应该：

{% highlight objc  %}
NSInteger arrayCount = self.array.count;  
[view setBackgroundColor:[UIColor orangeColor]];  
UIApplication.sharedApplication.delegate; 
{% endhighlight %}

### 8. 方法

在方法签名中，应该在方法类型(-/+ 符号)之后有一个空格。在方法各个段之间应该也有一个空格(符合Apple的风格)。在参数之前应该包含一个具有描述性的关键字来描述参数。

`and` 这个词的用法应该保留。它不应该用于多个参数来说明，就像`initWithWidth: height:`以下这个例子：

应该：

{% highlight objc  %}
- (void)setExampleText:(NSString *)text image:(UIImage *)image;  
- (void)sendAction:(SEL)aSelector to:(id)anObject forAllCells:(BOOL)flag;  
- (id)viewWithTag:(NSInteger)tag;  
- (instancetype)initWithWidth:(CGFloat)width height:(CGFloat)height; 
{% endhighlight %}

不应该：

{% highlight objc  %}
-(void)setT:(NSString *)text i:(UIImage *)image;  
- (void)sendAction:(SEL)aSelector :(id)anObject :(BOOL)flag;  
- (id)taggedView:(NSInteger)tag;  
- (instancetype)initWithWidth:(CGFloat)width andHeight:(CGFloat)height;  
- (instancetype)initWith:(int)width and:(int)height;  // Never do this.  
{% endhighlight %}

### 9. 注释
当需要注释时，注释应该用来解释这段特殊代码为什么要这样做。任何被使用的注释都必须保持最新或被删除。

一般都避免使用块注释，因为代码尽可能做到自解释，只有当断断续续或几行代码时才需要注释。例外：这不应用在生成文档的注释

注释很重要，但除了开头的版权声明，尽可能把代码写的如同文档一样，让别人直接看代码就知道意思，写代码时别担心名字太长，Xcode的提示功能很强大。

协议、委托的注释要明确说明其被触发的条件：

{% highlight objc  %}
/** Delegate - Sent when failed to init connection, like p2p failed. */
-(void)initConnectionDidFailed:(IPCConnectHandler *)handler;
{% endhighlight %}

推荐喵神写的注释插件：[VVDocumenter-Xcode](https://github.com/onevcat/VVDocumenter-Xcode)


### 10. 字面值

NSString、NSDictionary、NSArray 和 NSNumber 的字面值应该在创建这些类的不可变实例时被使用。请特别注意nil值不能传入 NSArray 和 NSDictionary 字面值，因为这样会导致 crash。

应该：

{% highlight objc  %}
NSArray *names = @[@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul"];  
NSDictionary *productManagers = @{@"iPhone": @"Kate", @"iPad": @"Kamal", @"Mobile Web": @"Bill"};  
NSNumber *shouldUseLiterals = @YES;  
NSNumber *buildingStreetNumber = @10018;  
{% endhighlight %}

不应该:

{% highlight objc  %}
NSArray *names = [NSArray arrayWithObjects:@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul", nil];  
NSDictionary *productManagers = [NSDictionary dictionaryWithObjectsAndKeys: @"Kate", @"iPhone", @"Kamal", @"iPad", @"Bill", @"Mobile Web", nil];  
NSNumber *shouldUseLiterals = [NSNumber numberWithBool:YES];  
NSNumber *buildingStreetNumber = [NSNumber numberWithInteger:10018];  
{% endhighlight %}

### 11. 控制语句

(1) 方法大括号和其他大括号(if/else/switch/while 等)总是在同一行语句打开但在新行中关闭。

如果 if 语句带有 else if 或 else 条件,在 else if 或 else 前后均有一个空格

如果在一个方法中 if 语法嵌套超过3层，应该检查代码逻辑，将子功能代码进行拆分

应该：

{% highlight objc  %}
if (user.isHappy) {  
    //Do something  
} else {  
    //Do something else  
}
{% endhighlight %}
 
不应该：

{% highlight objc  %}
if (user.isHappy)  
{  
    //Do something  
}  
else {  
    //Do something else  
} 
{% endhighlight %}

(2) 在控制语句中如果随后的语句只有一行，不允许省略括号

应该：
{% highlight objc  %}
if (flag == YES) {    NSLog(@“Hello”);}
{% endhighlight %}

不应该：
{% highlight objc  %}
if (flag == YES) NSLog(@“Hello”);
if (flag == YES)    NSLog(@“Hello”);
{% endhighlight %}

(3) Switch 语句中，无论一个 case 语句是否包含了多个语句，都需要加上括号。

{% highlight objc  %}
switch (condition) {    case 1: {        // single line statement    break;    } case 2: {        // ...        //Multi-line example using braces        break;    } default:        // ....        break; 
}
{% endhighlight %}

(4) 应该避免以冒号对齐的方式来调用方法。因为有时方法签名可能有 3 个以上的冒号和冒号对齐会使代码更加易读。请不要这样做，尽管冒号对齐的方法包含代码块，因为 Xcode 的对齐方式令它难以辨认。

应该：

{% highlight objc  %}
// blocks are easily readable  
[UIView animateWithDuration:1.0 animations:^{  
    // something  
} completion:^(BOOL finished) {  
    // something  
}];
{% endhighlight %}
  
不应该：

{% highlight objc  %}
// colon-aligning makes the block indentation hard to read  
[UIView animateWithDuration:1.0  
                 animations:^{  
                     // something  
                 }  
                 completion:^(BOOL finished) {  
                     // something  
                 }];  
{% endhighlight %}


### 12. 常量

常量是容易重复被使用和无需通过查找和代替就能快速修改值。常量应该使用 static 来声明而不是使用 #define，除非显式地使用宏。

应该：

{% highlight objc  %}
static NSString * const RWTAboutViewControllerCompanyName = @"RayWenderlich.com";  
static CGFloat const RWTImageThumbnailHeight = 50.0;  
{% endhighlight %}

不应该：

{% highlight objc  %}
#define CompanyName @"RayWenderlich.com"  
#define thumbnailHeight 2  
{% endhighlight %}

### 13. 枚举类型

当使用 enum 时，推荐使用新的固定基本类型规格，因为它有更强的类型检查和代码补全。现在SDK有一个宏 `NS_ENUM()` 来帮助和鼓励你使用固定的基本类型。

例如：

{% highlight objc  %}
typedef NS_ENUM(NSInteger, RWTLeftMenuTopItemType) {  
  RWTLeftMenuTopItemMain,  
  RWTLeftMenuTopItemShows,  
  RWTLeftMenuTopItemSchedule  
}; 
{% endhighlight %}
 
你也可以显式地赋值(展示旧的k-style常量定义)：

{% highlight objc  %}
typedef NS_ENUM(NSInteger, RWTGlobalConstants) {  
  RWTPinSizeMin = 1,  
  RWTPinSizeMax = 5,  
  RWTPinCountMin = 100,  
  RWTPinCountMax = 500,  
};  
{% endhighlight %}

旧的k-style常量定义应该避免除非编写Core Foundation C的代码。

不应该：

{% highlight objc  %}
enum GlobalConstants {  
  kMaxPinSize = 5,  
  kMaxPinCount = 500,  
};
{% endhighlight %}
 

### 14. 布尔值

Objective-C 使用 YES 和 NO。因为 true 和 false 应该只在 CoreFoundation，C或C++代码使用。既然 nil 解析成 NO，所以没有必要在条件语句比较，同时也不应该拿某样东西直接与YES比较

这是为了在不同文件保持一致性和在视觉上更加简洁而考虑。

应该：

{% highlight objc  %}
if (someObject) {}  
if (![anotherObject boolValue]) {}  
{% endhighlight %}

不应该：

{% highlight objc  %}
if (someObject == nil) {}  
if ([anotherObject boolValue] == NO) {}  
if (isAwesome == YES) {} // Never do this.  
if (isAwesome == true) {} // Never do this.  
{% endhighlight %}

如果BOOL属性的名字是一个形容词，属性就能忽略"is"前缀，但要指定get访问器的惯用名称。例如：

{% highlight objc  %}
@property (assign, getter=isEditable) BOOL editable;  
{% endhighlight %}

同样的，也**不要将其它类型的值作为 BOOL 来返回**，这种情况下， BOOL 变量只会取值的最后一个字节来赋值，这样很可能会取到 0 （ NO ）。但是，一些逻辑操作符比如 &&, ||, ! 的返回是可以直接赋给 BOOL 的。

应该：
{% highlight objc  %}
- (BOOL)isBold {
return ([self fontTraits] & NSFontBoldTrait) ? YES : NO;
}

// 正确，逻辑操作符可以直接转化为 BOOL
- (BOOL)isValid {
    return [self stringValue] != nil;
}
- (BOOL)isEnabled {
    return [self isValid] && [self isBold];
}
{% endhighlight %}

不应该：

{% highlight objc  %}
//错误，不要将其它类型转化为 BOOL 返回
- (BOOL)isBold {
    return [self fontTraits] & NSFontBoldTrait;
}
- (BOOL)isValid {
    return [self stringValue];
}
{% endhighlight %}

另外 BOOL 类型可以和 _Bool,bool 相互转化，但是不能和 Boolean 转化。


### 15. 三元操作符

当需要提高代码的清晰性和简洁性时，三元操作符 ?: 才会使用。单个条件求值常常需要它。多个条件求值时，如果使用 if 语句或重构成实例变量时，代码会更加**易读**。一般来说，在根据条件来赋值的情况下，才应该使用三元操作符。

Non-boolean的变量与某东西比较，加上括号()会提高可读性。但如果被比较的变量是boolean类型，那么就不需要括号。

应该：

{% highlight objc  %}
NSInteger value = 5;  
result = (value != 0) ? x : y;  

BOOL isHorizontal = YES;  
result = isHorizontal ? x : y;  
{% endhighlight %}

不应该：

{% highlight objc  %}
result = a > b ? x = c > d ? c : d : y;  
{% endhighlight %}


### 16. Init和类构造方法

**Init 方法**应该遵循 Apple 生成代码模板的命名规则，返回类型应该使用 instancetype 而不是 id。

{% highlight objc  %}
- (instancetype)init {  
  self = [super init];  
  if (self) {  
    // ...  
  }  
  return self;  
}  
{% endhighlight %}


**类构造方法**

当类构造方法被使用时，它应该返回类型是 instancetype 而不是 id。这样确保编译器正确地推断结果类型。

{% highlight objc  %}
@interface Airplane  
+ (instancetype)airplaneWithType:(RWTAirplaneType)type;  
@end  
{% endhighlight %}

关于更多instancetype信息，请查看 [NSHipster.com](http://nshipster.com/instancetype/)。


### 17. CGRect函数

当访问CGRect里的x, y, width, 或 height时，应该使用CGGeometry函数而不是直接通过结构体来访问。引用Apple的CGGeometry：

> 在这个参考文档中所有的函数，接受CGRect结构体作为输入，在计算它们结果时隐式地标准化这些rectangles。因此，你的应用程序应该避免直接访问和修改保存在CGRect数据结构中的数据。相反，使用这些函数来操纵rectangles和获取它们的特性。

应该：

{% highlight objc  %}
CGRect frame = self.view.frame;  
CGFloat x = CGRectGetMinX(frame);  
CGFloat y = CGRectGetMinY(frame);  
CGFloat width = CGRectGetWidth(frame);  
CGFloat height = CGRectGetHeight(frame);  
CGRect frame = CGRectMake(0.0, 0.0, width, height);  
{% endhighlight %}

不应该：

{% highlight objc  %}
CGRect frame = self.view.frame;  
CGFloat x = frame.origin.x;  
CGFloat y = frame.origin.y;  
CGFloat width = frame.size.width;  
CGFloat height = frame.size.height;  
CGRect frame = (CGRect){ .origin = CGPointZero, .size = frame.size }; 
{% endhighlight %}


### 18. 黄金路径

当编写条件语句的时候,尽量不要多层嵌套if语句,这样可以减少代码复杂性,并且 让代码更加容易阅读。多个return语句是可以的。

应该：

{% highlight objc  %}
- (void)someMethod {  
  if (![someOther boolValue]) {  
    return;  
  }  
  //Do something important  
}  
{% endhighlight %}

不应该：

{% highlight objc  %}
- (void)someMethod {  
  if ([someOther boolValue]) {  
    //Do something important  
  }  
}  
{% endhighlight %}

### 19. 错误处理

当方法通过引用来返回一个错误参数，判断返回值而不是错误变量。

应该：

{% highlight objc  %}
NSError *error;  
if (![self trySomethingWithError:&error]) {  
  // Handle Error  
}  
{% endhighlight %}

不应该：

{% highlight objc  %}
NSError *error;  
[self trySomethingWithError:&error];  
if (error) {  
  // Handle Error  
}  
{% endhighlight %}

在成功的情况下，有些Apple的APIs记录垃圾值(garbage values)到错误参数(如果non-NULL)，那么判断错误值会导致 false 负值和 crash。

### 20. 单例模式

单例对象应该使用线程安全模式来创建共享实例。

{% highlight objc  %}
+ (instancetype)sharedInstance {  
  static id sharedInstance = nil;  
  static dispatch_once_t onceToken;  
  dispatch_once(&onceToken, ^{  
    sharedInstance = [[self alloc] init];  
  });  
  return sharedInstance;  
}  
{% endhighlight %}

这会防止 [possible and sometimes prolific crashes](http://cocoasamurai.blogspot.com/2011/04/singletons-your-doing-them-wrong.html)。




### 21. Xcode工程

物理文件应该与 Xcode 工程文件保持同步来避免文件扩张。任何 Xcode 分组的创建应该在文件系统的文件体现。代码不仅是根据类型来分组，而且还可以根据功能来分组，这样代码更加清晰。

尽可能在target的Build Settings打开"Treat Warnings as Errors，和启用以下[additional warnings](http://boredzo.org/blog/archives/2009-11-07/warnings)。如果你需要忽略特殊的警告，使用 [Clang's pragma feature](http://clang.llvm.org/docs/UsersManual.html#controlling-diagnostics-via-pragmas)。



















