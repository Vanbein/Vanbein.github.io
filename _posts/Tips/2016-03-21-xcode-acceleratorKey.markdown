---
layout: post
date: 2016-03-21 23:52:02 +0800
title: Xcode基础快捷键 -- 提高你的编码速率
category: Tips
tags: Xocde iOS
image: http://o7rxin1of.qnssl.com/images/head-800x400/-19.png
description: 在 Xcode 中有许多快捷键，它可以使得我们的编码工作更为高效，对于在代码文件中快速导航、定位Bug以及新增应用特性都是极有效的。
homepage: false
toc: true
---


在 Xcode 中有许多快捷键，它可以使得我们的编码工作更为高效，对于在代码文件中快速导航、定位Bug以及新增应用特性都是极有效的。

我相信想要成为一名出色的iOS开发，Xcode 快捷键是我们应该熟悉掌握的。而且其中大部分快捷键在其他一些编辑器也是通用的，所以在学习到如何使用之后，也许就再也离不开它们了。

### 一、编码快捷键

下面的快捷对提高我们编码速率非常有帮助，值得我们慢慢熟练使用并牢记。

#### 1. 上下左右 移动选中代码
* 代码上移：option + command +[ ;
* 代码下移：option + command +] ;
* 代码左缩进 **command + [**
* 代码右缩进 **command + ]**
* 自动排版代码：Xcode默认的是 Control + shift + \, 但是为了方便，我推荐重新设置为**Command + =**

#### 2. 使选择的内容上下左右移来选择更多

* **shift + 上/下/左/右 方向键**


#### 3. 将方法或者注释收起、展开

使用时只要鼠标在方法或注释的范围内就好

* 收起：**option + command + <—**
* 展开：**option + command + —>**


#### 4.光标相关操作

* **Command + <—**  光标跳到行首，推荐
* **Command + —>**  光标跳到行尾，推荐
* **Command + 上方向键**  光标跳到页面顶部，推荐
* **Command + 下方向键**  光标跳到页面底部，推荐

* **Option + <—**  光标回退到左边一个词组前，推荐
* **Option + —>**  光标回退右边一个词组尾，推荐
* **Option + 上方向键**  光标移到上一行最前面，推荐
* **Option + 下方向键**  光标移到下一行最后面，推荐

* Ctrl + A (Ahead),  光标跳到行首
* Ctrl + E (End),      光标跳到行尾

* **Ctrl + B (Back)**, 光标回退一个字符
* **Ctrl + F (Forward)**,  光标前进一个字符

* **Ctrl + N (Next)**,  光标跳到下一行
* **Ctrl + P (Previous)**,  光标跳到上一行
 
* Ctrl + D (), 删除光标右边的一个字符
* Ctrl + H (), 删除光标左边的一个字符
 
* Ctrl + K (Kill), 删除光标后面所有内容

* 删除光标所在行，使用组合键
  * Ctrl + AK (Kill), ，
  * **Command + —> + delete ** 推荐

* 删除光标所在的词，比如变量名，方法名等，使用组合键  
  * **Option + —> + Delete**  推荐


#### 5. 一次性修改一个 Scope 里的变量名：

点击该变量，出现下划虚线，然后 **command + control + E** 激活所有相同变量，然后进行修改。

#### 6. 删除
* 删除一个词：Option + Delete，会删除光标的位置所在行的左边一个词
* 删除一句话：Command + Delete，会删除光标的位置所在行的左边的内容

#### 7. 快捷搜索：
先点亮想要搜索的词，然后** Command + E **将该次放入剪贴板，然后使用** Command + G **来向下遍历该词，**Shift + Command + G** 向上遍历。

#### 8. Debug调试常用快捷键：

* 断点时，执行下一行：F6，如果发现按F6没效果，那请使用Fn + F6，下面类似
* 断点时，进入方法：F7
* 断点时，跳出方法：F8
* 全速执行到下一断点：Command + Control + Y
* 清除 Debug Console 全部内容：**Command + K**  推荐


### 二、以下快捷键可帮你在代码编写过程中尽可能少地使用鼠标或触控板。

#### 1. 运行程序: **Command + R**, 停止运行**Command + .**

在编写代码的过程中，我通常会使用该快捷键来自由运行应用程序。尽可能地测试应用程序，这样你可以在早期找到并修复应用中的bug。

![Command + R](http://o7rxin1of.qnssl.com/images/2015/11/Command+R.png)

#### 2. 清除工程: Command + Shift + K

当Xcode运行出现问题，比如应用无法响应，或者出现了意料之外的情况，我们应该首先去清除工程并再次运行它。如果这样还不能解决问题，那就只能重启Xcode了。

![Command + Shift + K](http://o7rxin1of.qnssl.com/images/2015/11/Command+Shift+K.png)

#### 3. 构建应用程序: Command + B

检查所写代码以确保其正常工作是我们经常要做的事情，编译app工程可让你在编写下一个特性之前确定其是否正常工作。即便 Xcode 在代码编写后会很快进行检查，但也有所延迟，或者给出一些不恰当的错误提示。

因此假如仅仅做一些小的改变，我们无需总是运行应用程序，那么编译工作可帮你做一个快速检查，这样可以返回添加下一行代码。

![Command + B](http://o7rxin1of.qnssl.com/images/2015/11/Command+B.png)

### 三、Xcode导航快捷键

#### 1. 工程导航控制器：Command + 1

快速浏览代码、图片以及用户界面文件。

![Command+1](http://o7rxin1of.qnssl.com/images/2015/11/Command+1.png)

> 还可以试试 Command + 2/3/4/5/6/7/8

#### 2.显示/隐藏导航器面板:Command+0

当在对屏幕进行截图的时候可能会想要隐藏起与我们感兴趣内容的无关的部分。假如想要使用辅助编辑器或者想要设计用户界面并将其连接到代码的时候，这个快捷键会相当有用

![Command+0](http://o7rxin1of.qnssl.com/images/2015/11/Command+0.png)
#### 3.显示/隐藏实用工具面板:Command+Option+0

实用工具面板主要用于编辑用户界面文件时，在只考虑写代码的时候，就可以隐藏它。

![Command+Option+0.png](http://o7rxin1of.qnssl.com/images/2015/11/Command+Option+0.png)

#### 4.在辅助编辑器中打开文件:在项目导航器中选中文件执行Option+(左键)点击操作。

一个快速打开`Assistant Editor`的方式--只需要按住Option键并点击你想要在当前编辑框右边打开的文件即可。

![Option+Left-click.png](http://o7rxin1of.qnssl.com/images/2015/11/Option+Left-click.png)

#### 5.搜索导航器(Find Navigator，也就是搜索):Command+Shift+F

使用项目搜索可以找到某个变量或方法名的被提到的次数。可以依据实例来匹配，并可忽略大小写字母。另外还可以对查找的变量名进行替换。

![Command+Shift+F.png](http://o7rxin1of.qnssl.com/images/2015/11/Command+Shift+F.png)

#### 6.快速打开: Command + Shift + O

喜欢使用键盘但不喜欢使用鼠标的人会大爱这个快捷方式，可以直接跳转到某个方法定义或者指定的代码文件。
另外，键入第一个字母即可快速切换至某个文件或者找到特定的代码行。

![Command+Shift+O.png](http://o7rxin1of.qnssl.com/images/2015/11/Command+Shift+0.png)

#### 7. .h & .m文件间的快速切换: Control + Command + 上下或左右方向键

如果你用Objective-C或Swift来编写程序，或者使用其他语言编写。我们可以使用**Control + Command + 方向键**组合键操作在两个文件间相互切换

![Control+Command+方向键.png](http://o7rxin1of.qnssl.com/images/2015/11/Control+Command+方向键.png)

### 四、文档和帮助

> 在学习过程中，自助学习非常重要，对于没有浏览过Xcode文档帮助的开发者来说，这些快捷键可帮忙查看相关的代码参考，更好地理解苹果提供的代码，从而开发出更优秀的APP。

#### 11.文档和参考: Command + Shift + 0 (Zero)

使用Xcode在后台安装文档，并支持离线搜索查看，非常适合外出办公。打开文档和参考，并键入代码中的某个关键字，Xcode文档还提供了一些额外的资源和示例工程。

通过 Documentation and Reference 指南了解如何使用代码

![Command+Shift+0.png](http://o7rxin1of.qnssl.com/images/2015/11/Command+Shift+0.png)

#### 12.快速帮助: 在类或者方法名上执行Option + Left-click操作

内联帮助可帮开发者快速学习类或代码片段的用法。在变量、类、或者方法名上执行Option + Left-click操作来获得更多细节信息。假使你点击了弹出视图底部的参考链接，那么就可以方便地跳转到Xcode提供的文档中。你还可以在变量、类或者方法名上执行Option+双击名称操作，从而更方便地跳转至文档。

编写代码时获得快速帮助

![Option+Leftclick.png](http://o7rxin1of.qnssl.com/images/2015/11/Option+Leftclick.png)

#### 13.打开'Show Related Items弹出菜单：Control + 1
该快捷键可打开 **Show Related Items** 弹出菜单。当把光标放在任何方法中，并按下 **CTRL + 1**，就可以很方便地通过弹出的视图访问该方法的所有调用者和被调用者。我们可以通过浏览方法的调用者从而了解如何使用该方法。

![Control+1.png](http://o7rxin1of.qnssl.com/images/2015/11/Control+1.png)

### 五、其他一些常用的快捷键

* 新建项目 **Command + Shift + N**

* 新建文件 **Command + N**

* 新建空文件 Command + Control + N

* 打开 Command + O

* 关闭窗口 **Command + W**

* 保存所有文件 Command + Option + S

* 还原到保存时状态 Command + U

* 撤销操作 **Command + Z**

* 创建快照 Command + Control + S  (保存文件快照，以后可进行对比修改情况)

> 欢迎留言补充