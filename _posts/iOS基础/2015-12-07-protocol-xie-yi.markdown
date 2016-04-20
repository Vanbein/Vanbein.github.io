---
layout: post
title: Protocol--协议
category: iOS基础
tags: iOS Protocol Objective-C
image: /images/head-800x400/-50.png
description: 在写jave的时候都会有接口interface这个概念，接口就是一堆方法的声明没有实现，而在Obejective-C里面interface是一个类的头文件的声明，并不是真正意义上的接口的意思，在Objective-C中接口是由一个叫做协议的protocol来实现的模式，表示将一个对象的部分功能转交给另外一个对象，借助协议protocol可以很方便的实现这种设计模式。
toc: true
homepage: false
---

### 1. Protocol
* #### 概念
	+ 在写jave的时候都会有接口interface这个概念，接口就是一堆方法的声明没有实现，而在	Obejective-C里面interface是一个类的头文件的声明，并不是真正意义上的接口的意思，在	Objective-C中接口是由一个叫做协议的protocol来实现的
	+ protocol它可以声明一些必须实现的方法和选择实现的方法。这个和java是完全不同的

* #### 注意
	+ 协议不引用任何类！它是无类的（classless），任何类都可以都可以遵守某个协议 
	+ 一个类可以遵守1个或者多个协议
	+ 任何类只要遵守了某个协议protocol，就相当于拥有了该protocol的所有方法声明
	+ 就一个用途，用来声明一大堆的方法，不能声明成员变量，不能写实现
	+ 只要父类遵守了某个协议，那么子类也遵守
	+ Objective-C不能继承自多个类，但是能够遵循多个协议。继承（:）,遵守协议（< >）
	+ 协议可以遵守协议，一个协议遵守了另外一个协议，就可以拥有另一个协议中的方法声明
	+ 和类名称一样，协议名必须是唯一的

* #### protocol 和 inheritance 的区别
	+ protocol 协议， inheritance 继承
	+ 继承之后默认就有实现，而协议只有声明没有实现
	+ 相同类型的类可以使用继承，但是**不同类型的类**只能使用协议protocol
	+ protocol可以用于存储方法的声明，可以将多个类中共同的方法抽取出来，以后让这些类遵守协议即可
	
* #### 基协议
	+ 基协议就是`NSObject`协议，是最根本最基本的协议，NSObject类也要遵循它
	+ NSObject协议中声明了很多最基本的方法
		- description
		- retain
		- release

* #### protocol中的修饰符
	+ @required: 修饰这个方法必须要实现，若不实现，编译器会发出警告
	+ @optional: 修饰这个方法不一定要实现
	+ protocol默认都是required的
	+ 修饰符用于程序员之间的交流，不是强制性的要求
	+ 判断这个类是否实现某个方法则只需调用 `[self.delegate respondToSelector:@selector(selectorName)]`

### 2. protocol用途

* #### 1.类型限制
	+ 如何在代码中要求对象必须具备这些行为？
		- 数据类型<协议名称> 变量名
		
{% highlight objc linenos %}
//如果没有遵循协议，则会报警告
id<StudentCondition> sutdent = [[Person alloc] init];
{% endhighlight %}

* #### 2.代理模式
	+ 应用场景：
		- 当对象A发生了一些行为，想告知对象B（让对象B成为对象A的代理对象）
		- 对象B想监听对象A的一些行为（让对象B成为对象A的代理对象）
		- 当对象A无法处理某些行为的时候，想委托对象B帮忙处理（让对象B成为对象A的代理对象）
		




















