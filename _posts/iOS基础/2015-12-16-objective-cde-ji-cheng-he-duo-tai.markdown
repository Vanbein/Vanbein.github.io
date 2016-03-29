---
layout: post
title: Objective-C的继承和多态
category: iOS基础
tags: iOS Objective-C
image: /images/head-800x400/-37.png
description: 面向对象编程之所以成为主流的编程思想和他的继承和多态是分不开的，只要是面向对象语言都支持继承和多态，当然不同的OOP语言之间都有其特点。
homepage: false
---

面向对象编程之所以成为主流的编程思想和他的继承和多态是分不开的，只要是面向对象语言都支持继承和多态，当然不同的OOP语言之间都有其特点。
<!--more-->
OC中和Java类似，不支持多重继承，但OOP语言C++就支持多继承，为什么OC不支持多继承稍后将会提到

在Objective-C中 super 是指向直接父类的指针，而self是指向本身的指针，self就相当于java中的this指针。在OC中写类时可以在 `@implementation` 中定义哪些在 `@interface` 中无相应声明的方法，但这个方法是私有的，仅在类的实现中使用。

##一、NSObject
在Objectiv-C中几乎所有的类都是继承自NSObject类，NSObject类中存在大量功能强大的方法。下面对NSObject类中的各种方法进行试验和介绍：

###1. `+(void) load;` 

* 类加载到运行环境时调用该方法

测试：在子类中重写load方法来进行测试, 当重写完load方法，在mian方法中不需要任何实例化任何对象

当类被加载时load就会别调用，load是类方法，可以直接被类调用

```objc
//重写NSObject中的load方法
+(void) load
{
    NSLog(@"load方法被调用了！");
}
//运行结果： load方法被调用了！
```

###2. `+(void) initialize;`

* 在类第一次使用该类时调用该方法，第二次就不调用了

测试：重写initalize方法，

```objc
//重写initialize方法，会在类第一次使用时调用
+(void) initialize
{
    NSLog(@"initialize方法被调用了！");
}
//运行结果
//load方法被调用了！
//initialize方法被调用了！
```

###3. `+(id) alloc:` 、 `-(id) init:` 和 `new`

* +(id) alloc: 返回一个已经分配好的内存对象；  
* -(id) init: 对已经分配了内存的实例进行初始化； 
* new 同时调用了alloc 和 init

可以在子类中把alloc进行重写，然后观察运行情况

举个例子

```objc
//重写alloc方法
+(id) alloc
{
   NSLog(@"alloc函数被调用啦");
    return [super alloc];
}
//重写init
-(id) init
{
    NSLog(@"init被调用了");
    self = [super init];
    return self;
}
//实例化类
ObjectTest *o1 = [[ObjectTest alloc] init];
// 
ObjectTest *o2 = [ObjectTest new];
//运行结果：
//alloc函数被调用啦
//init被调用了
//alloc函数被调用啦
//init被调用了
```

###4. `-(void)dealloc`

* 释放对象自身所占有的内存；

###5. `-(Class)class` 和 `-(Class)superclass`

* -(Class)class或者 +(Class)class 返回当前对象的所属类;  
* -(Class)superclass 或者 +(Class)superclass返回当前类的父类

```objc
//返回当前对象所对应的类
NSString *className =(NSString *) [self class];
NSLog(@"%@类的display方法", className);
// 
//返回当前对象所对应的父类
NSString *superClassName = (NSString *) [self superclass];
NSLog(@"%@类的父类是%@", className, superClassName);
```

###6. `-(BOOL)isKindOfClass : (Class)aClass`
* 判断某个实例是否属于某个类或者子类的对象

示例代码如下：

```objc
//isKindOfClass的用法
BOOL b =  [objct isKindOfClass:[ObjectTest class]];
if (b == YES) {
    NSLog(@"objct是ObjectTest类的对象");
}
```
###7. -(BOOL)isMemberOfClass:(Class)aClass;  
* 这只能判断摸个实例是否属于某个类，不能判断是否属于某个父类；

```objc
//isMemberOfClass
BOOL result = [o2 isMemberOfClass:[NSObject class]];
if (result == NO) {
    NSLog(@"isMemberOfClass不能判断是否为NSObject子类的对象");
}
```

###8. -(NSString *) description; 
* 返回字符串形式对象的描述，方便调试

```objc
//description
NSString *descript = [o2 description];
NSLog(@"%@", descript);
//输出结果： <ObjectTest: 0×100206680>
```

###9. -(NSUInteger) hash; 
* 返回对象的哈希值；

```objc
//hash的用法
NSUInteger hash = [o2 hash];
NSLog(@"%p", hash);
//输出结果：2014-07-30 11:40:35.685 HelloOC[1130:303] 0x100107b70
```

###10. -(BOOL) isEqual:(id)object; 
* 比较两个对象是否相同，默认是使用地址进行比较的，且hash值一定要相同

```objc
//isEqual的用法
NSString *str1 = @"111";
NSString *str2 = str1;
if ([str2 isEqual:str1] == YES)
{
    NSLog(@"str2 == str1");
}
else
{
    NSLog(@"str2 != str1");
}
```

##二、Objective-C中的继承

继承是is-a的关系，比如猫是一个动物，那么动物是父类，而猫是动物的子类。子类具有父类的属性和 行为，以及自身的属性和行为，也就是说“父类更一般，子类更具体”。用一个富二代的类来说明一下类的继承。

####1.先定义一个富人的类

> #####代码说明：
> 1. 声明访问权限为@protected的三个属性，分别为三个属性用@property 加上 getter 和setter 方法
> 2. 并为该类创建便利初始化方法和便利构造器
> 3. 为富人类定义一个刷卡方法


```objc 
@interface Richer : NSObject
{
    @protected
    NSString *name;
    int age;
    NSString *gender;
}
// 
//定义富人的姓名，年龄，性别，并为其提供getter和setter方法
@property (copy, nonatomic) NSString *name;
@property (assign, nonatomic) int age;
@property (copy, nonatomic) NSString *gender;
// 
//定义便利初始化方法
-(id) initWithName : (NSString *)vName
            AndAge : (int)vAge
         AndGender : (NSString *)vGender;
// 
//定义便利构造器
+(id) richerWithName : (NSString *)vName
              AndAge : (int)vAge
           AndGender : (NSString *)vGender;
// 
//定义刷卡方法
-(void) poss;
//
@end
```

####2.为富人类编写实现代码


> #####代码说明：
> 1. 用@synthesize实现getter和setter方法
> 2. 实现便利初始化方法，用[ super init ]初始化富人类的直接父类，也就是NSObject
> 3. 使用便利构造器返回实例化并初始化后的对象

```objc
#import "Richer.h"
@implementation Richer
//实现getter和setter方法
@synthesize name, age, gender;
//实现便利初始化函数
-(id) initWithName : (NSString *)vName
            AndAge : (int)vAge
         AndGender : (NSString *)vGender
{
    if (self = [super init])
    {
        self->name = vName;
        self->age = vAge;
        self->gender = vGender;
    }
    return self;
}
//实现便利构造器
+(id) richerWithName:(NSString *)vName
              AndAge:(int)vAge
           AndGender:(NSString *)vGender
{
    Richer *richer = [[Richer alloc] initWithName:vName
                                           AndAge:vAge
                                        AndGender:vGender];
    return richer;
}
//实现刷卡方法
-(void) poss
{
    NSLog(@"%@有钱你就刷吧", name);
}@end
```

####3.编写富二代的类
富二代和富人有许多相似的属性和方法所以富二代继承于富人类，并添加相应的属性和方法，把需要重写的方法进行重写。

> #####代码说明：
> 1. 为富二代类添加新的爱好属性
> 2. 为富二代添加新的方法

```objc
#import "Richer.h"
// 
@interface Richer2nd : Richer
//Richer2nd继承啦Richer所有的方法
// 
//为富二代添加新的属性
@property (copy, nonatomic) NSString *hobby;
// 
//便利初始化函数
-(id) initWithName : (NSString *) vName
            AndAge : (int)vAge
         AndGender : (NSString *) vGender
          AndHobby : (NSString *)vHobby;
//为Richer2nd编写便利构造器
+(id)richer2ndWithName : (NSString *) vName
                AndAge : (int)vAge
             AndGender : (NSString *) vGender
              AndHobby : (NSString *)vHobby; 
//添加hobby方法
-(void) myHobby;
// 
@end
```

####4.各种方法的实现

> #####代码说明:
> 1. 在编写便利初始化方法时利用super来调用父类的便利初始化方法来把继承到的父类的方法进行初始化
> 2. 用self给新添加的属性进行初始化

```objc
#import "Richer2nd.h"
// 
@implementation Richer2nd
//实现属性的getter和setter方法
@synthesize hobby;
// 
//编写便利初始化函数，复用父类的便利初始化方法
-(id)initWithName:(NSString *)vName
           AndAge:(int)vAge
        AndGender:(NSString *)vGender
         AndHobby:(NSString *)vHobby
{
    if (self = [super initWithName:vName AndAge:vAge AndGender:vGender]) {
        self->hobby = vHobby;
    }
    return self;
}
// 
//编写便利构造函数
+(id)richer2ndWithName:(NSString *)vName
                AndAge:(int)vAge
             AndGender:(NSString *)vGender
              AndHobby:(NSString *)vHobby
{
    Richer2nd *richer = [[Richer2nd alloc] initWithName:vName AndAge:vAge AndGender:vGender AndHobby:vHobby];
    return richer;
}
// 
//重写刷卡方法
-(void)poss
{
    [super poss];
    NSLog(@"我是富二代，我爸有钱，我就刷!");
}
// 
//添加新的方法
-(void) myHobby
{
    NSLog(@"我是富二代%@,我超喜欢%@", name, hobby);
}
// 
@end
```

5. 以下是上面代码的运行结果

```
Bill有钱你就刷吧
BILL`s son有钱你就刷吧
我是富二代，我爸有钱，我就刷!
我是富二代BILL`s son,我超喜欢飙车
```

​##三、Objective-C中的多态

多态简单的说就是对于不同对象响应同一个方法时做出的不同反应。在 OC中动态类型id是实现多态的一种方式，id是一个独特的数据类型，可以转换为任何数据类型，上面的富人和富二代可以这样定义

```objc
id richer = nil;
// 
//测试富人类
richer = [Richer richerWithName:@"Bill" AndAge:40 AndGender:@"Man"];
[richer poss];
// 
//测试富二代的类
richer = [Richer2nd richer2ndWithName:@"BILL`s son" AndAge:16 AndGender:@"男" AndHobby:@"飙车"];
[richer poss];
[richer myHobby];
```

上面程序的输出结果和继承部分的结果是一致的；

​多态的另一个例子: Animal是父类，子类有Cat 和 Dog,子分别重写了父类中的eat方法；实例化对象的时候可以用下面的方法：

```objc
Animal *animal = nil;
 //实例化猫的对象
animal = [Cat new];
[animal eat];
 //实例化狗的对象
animal = [Dog new];
[animal eat];
```

> #####面向对象编程中的OCP原则和LSP原则
> * OCP : Open Closed Principle原则， 对扩展开放，对修改关闭
> * LSP ：里氏代换原则，任何基类可以出现的地方，子类一定可以出现。





