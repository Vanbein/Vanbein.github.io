---
layout: post
title: delegate、block、NSNotification的用法区分
category: iOS基础
tags: iOS delegate block
image: http://o7rxin1of.qnssl.com/images/head-800x400/-36.png
description: 对delegate、block、NSNotification的用法做一个总结
homepage: false
toc: true
---


<!--more-->

#### 通知
* “一对多”，在APP中，很多控制器都需要知道一个事件，应该用通知；

#### delegate：
* “一对一”，对同一个协议，一个对象只能设置一个代理delegate，所以单例对象就不能用代理；
* 代理更注重过程信息的传输：比如发起一个网络请求，可能想要知道此时请求是否已经开始、是否收到了数据、数据是否已经接受完成、数据接收失败

#### block：
* 1：写法更简练，不需要写protocol、函数等等
* 2，block注重结果的传输：比如对于一个事件，只想知道成功或者失败，并不需要知道进行了多少或者额外的一些信息
* 3，block需要注意防止循环引用：

ARC下这样防止：

{% highlight objc  %}
__weak typeof(self) weakSelf = self;
  [yourBlock:^(NSArray *repeatedArray, NSArray *incompleteArray) {
       [weakSelf doSomething];
    }];
{% endhighlight %}

非ARC

{% highlight objc  %}
__block typeof(self) weakSelf = self;
  [yourBlock:^(NSArray *repeatedArray, NSArray *incompleteArray) {
       [weakSelf doSomething];
    }];
{% endhighlight %}
