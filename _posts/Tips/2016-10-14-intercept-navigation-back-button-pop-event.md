---
layout: post
title: iOS拦截导航栏返回按钮事件的正确方式
category: Tips
tags: Navigation
image: /images/head-800x400/-61.png
description: 当我们使用了系统的导航栏时，默认点击返回按钮是 pop 回上一个界面。但是在有时候，我们需要在点击导航栏的返回按钮时不一定要 pop 回上一界面，比如一个视频播放界面，进入横屏后，默认点击返回按钮仍然是 pop 返回上一个界面，但是如果我们想要在横屏点击返回按钮的时候是返回竖屏模式，而不是 pop 到上一界面，这该怎么实现呢？
homepage: false
toc: true
---


当我们使用了系统的导航栏时，默认点击返回按钮是 pop 回上一个界面。但是在有时候，我们需要在点击导航栏的返回按钮时不一定要 pop 回上一界面，比如一个视频播放界面，进入横屏后，默认点击返回按钮仍然是 pop 返回上一个界面，但是如果我们想要在横屏点击返回按钮的时候是返回竖屏模式，而不是 pop 到上一界面，这该怎么实现呢？

> 注意：我们要的不是获取点击返回按钮的时机，而是想要拦截点击返回按钮的 pop 操作，使我们可以进行选择性的 pop，而不是必然的 pop。

下面一步步来解决这个问题。

### 一、自定义返回按钮

第一种，自定义导航栏的返回按钮，虽然这看起来是一种方式，但是也不能从根本上解决；比如整个应用的返回键都是统一的，这时候再重写了某个界面的返回按钮感觉就不统一了。而且每有一个界面有这个需求都需要重新自定义一个返回按钮，显得不优雅。

自定义返回按钮的方法很简单，如下：
{% highlight objc %}
// 自定义返回按钮
- (void)customBackButton{
    UIButton *backBtn = [UIButton buttonWithType:UIButtonTypeCustom];
    [backBtn setTitle:@"返回" forState:UIControlStateNormal];
    [backBtn addTarget:self action:@selector(backBtnClicked:) forControlEvents:UIControlEventTouchUpInside];
    backBtn.frame = CGRectMake(0, 0, 60, 40);
    [backBtn setTitleColor:[UIColor blackColor] forState:UIControlStateNormal];
    UIBarButtonItem *item = [[UIBarButtonItem alloc]initWithCustomView:backBtn];
    self.navigationItem.leftBarButtonItem = item;
}
// 返回按钮按下
- (void)backBtnClicked:(UIButton *)sender{
    if (_isLandscape) {
        // 进入竖屏
        [self enterPortrait];   
        return;
    }
    // pop
    [self.navigationController popViewControllerAnimated:YES];
}

{% endhighlight %}

### 二、为 UINavigationController 添加 category

> 此方法来自 github：[UIViewController-BackButtonHandler](https://github.com/onegray/UIViewController-BackButtonHandler)

由于系统的 `UINavigationController` 使用了一个 `UINavigationBar` 来管理 Controller 的 pop 和 push 等操作，所以仔细查看 `UINavigationBar` 的 API，会发现一个 `UINavigationBarDelegate`，它包含了四个方法：

{% highlight objc %}
- (BOOL)navigationBar:(UINavigationBar *)navigationBar shouldPushItem:(UINavigationItem *)item; // called to push. return NO not to.
- (void)navigationBar:(UINavigationBar *)navigationBar didPushItem:(UINavigationItem *)item;    // called at end of animation of push or immediately if not animated
- (BOOL)navigationBar:(UINavigationBar *)navigationBar shouldPopItem:(UINavigationItem *)item;  // same as push methods
- (void)navigationBar:(UINavigationBar *)navigationBar didPopItem:(UINavigationItem *)item;
{% endhighlight %}

我们会惊喜的第一个不就是我们想要的效果吗？因为该方法返回 YES 则 pop，若返回NO，则不POP。

因此我们可以为 UINavigatonController 创建一个 Category，来定制`navigationBar: shouldPopItem:` 的逻辑。这里需要注意的是，我们不需要去设置 delegate，因为 UINavigatonController 自带的 UINavigationBar 的 delegate 就是导航栏本身。这样还有个问题就是，那在实际的 Controller 里面怎么控制呢？因此同样需要对 UIViewController 添加一个 Protocol，这样在 Controller 中使用该 Protocol 提供的方法即可进行控制了，代码如下：

{% highlight objc %}
// UIViewController+BackButtonHandler.h
@protocol BackButtonHandlerProtocol <NSObject>
@optional
// 重写下面的方法以拦截导航栏返回按钮点击事件，返回 YES 则 pop，NO 则不 pop
-(BOOL)navigationShouldPopOnBackButton;
@end

@interface UIViewController (BackButtonHandler) <BackButtonHandlerProtocol>

@end
{% endhighlight %}

{% highlight objc %}
// UIViewController+BackButtonHandler.m 
@implementation UIViewController (BackButtonHandler)

@end

@implementation UINavigationController (ShouldPopOnBackButton)

- (BOOL)navigationBar:(UINavigationBar *)navigationBar shouldPopItem:(UINavigationItem *)item {

	if([self.viewControllers count] < [navigationBar.items count]) {
		return YES;
	}

	BOOL shouldPop = YES;
	UIViewController* vc = [self topViewController];
	if([vc respondsToSelector:@selector(navigationShouldPopOnBackButton)]) {
		shouldPop = [vc navigationShouldPopOnBackButton];
	}

	if(shouldPop) {
		dispatch_async(dispatch_get_main_queue(), ^{
			[self popViewControllerAnimated:YES];
		});
	} else {
		// 取消 pop 后，复原返回按钮的状态
		for(UIView *subview in [navigationBar subviews]) {
			if(0. < subview.alpha && subview.alpha < 1.) {
				[UIView animateWithDuration:.25 animations:^{
					subview.alpha = 1.;
				}];
			}
		}
	}
	return NO;
}
{% endhighlight %}

到这儿，几乎就完美的解决一开始的问题了，使用方法也非常简单

(1). 在一个 Controller 中：

{% highlight objc %}
#import "UIViewController+BackButtonHandler.h"
{% endhighlight %}

(2). 重写 navigationShouldPopOnBackButton 方法

{% highlight objc %}
- (BOOL)navigationShouldPopOnBackButton{
	[[[UIAlertView alloc] initWithTitle:@"提示" message:@"确定返回上一界面?"
							   delegate:self cancelButtonTitle:@"取消" otherButtonTitles:@"确定", nil] show];
	return NO;
}
// UIAlertViewDelegate
- (void)alertView:(UIAlertView *)alertView willDismissWithButtonIndex:(NSInteger)buttonIndex{
	if (buttonIndex==1) {
		[self.navigationController popViewControllerAnimated:YES];
	}
}
{% endhighlight %}
