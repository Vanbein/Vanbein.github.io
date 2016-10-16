---
layout: post
title: iOS限制输入内容的类型、长度的解决办法
category: Tips
tags: TextField
image: /images/head-800x400/-60.png
description: 在移动开发中经常遇到一个问题，就是一个输入框需要做输入内容或长度的限制，比如用户设置用户名时，限制只能使用2-12位汉字、字母、数字，，那么有没有简单粗暴效果好的解决办法呢？
homepage: false
toc: true
---


在移动开发中经常遇到一个问题，就是一个输入框需要做输入内容或长度的限制，比如用户设置用户名时，限制只能使用2-12位汉字、字母、数字

### 监听输入框字符变化

虽然 UITextField 提供了几个代理方法给我，但是经过我的验证，最佳监听输入框文字变化的办法是给 UITextField 添加 `Target: action:`，如下：

{% highlight objc %}
    [self.textField addTarget:self action:@selector(textFieldDidChange:) forControlEvents:UIControlEventEditingChanged];
{% endhighlight %}

这样，在 UITextField 的内容发生变化时，我们都能第一时间知道，并对其做相应的操作：

1. 过滤非汉字、字母、数字字符
2. 长度限制，对超出部分进行截取

`textFieldDidChange:` 的代码如下：

{% highlight objc %}
- (void)textFieldDidChange:(UITextField *)textField{
    UITextRange *selectedRange = textField.markedTextRange;
    UITextPosition *position = [textField positionFromPosition:selectedRange.start offset:0];
    if (!position) {
        // 没有高亮选择的字
        // 1. 过滤非汉字、字母、数字字符
        self.textField.text = [self filterCharactor:textField.text withRegex:@"[^a-zA-Z0-9\u4e00-\u9fa5]"];
        // 2. 截取
        if (self.textField.text.length >= 12) {
            self.textField.text = [self.textField.text substringToIndex:12];
        }
    } else {
        // 有高亮选择的字 不做任何操作
    }
}

// 过滤字符串中的非汉字、字母、数字
- (NSString *)filterCharactor:(NSString *)string withRegex:(NSString *)regexStr{
    NSString *filterText = string;
    NSError *error = NULL;
    NSRegularExpression *regex = [NSRegularExpression regularExpressionWithPattern:regexStr options:NSRegularExpressionCaseInsensitive error:&error];
    NSString *result = [regex stringByReplacingMatchesInString:filterText options:NSMatchingReportCompletion range:NSMakeRange(0, filterText.length) withTemplate:@""];
    return result;
}
{% endhighlight %}

在这里，我们首先判断输入框是否有高亮选择的文字，有则表示可能正在输入汉字，所以不该做其他的操作判断；如果没有高亮选择的文字，则先使用正则表达式进行文本过滤，去除不合法的字符，其中的 `\u4e00-\u9fa5` 表示汉字类型；最后再对于超出长度限制的字符进行截取。这样，用户最终看到的是操作完成后的字符串，所以无论用户输入的是什么，我们肯定能保证输入内容的格式正确性

当然，这样还不算完成，当用户输入完成，点击确定的时候，我们还需要做一次长度的判断即可，如果设置了键盘的 return 键为 完成键，那么在代理方法中实现：
{% highlight objc %}
// return键按下
- (BOOL)textFieldShouldReturn:(UITextField *)textField{
    [self.view endEditing:YES];
    if (textField.text.length < 2) {
       // 提示用户，名称长度至少为2位
        return NO;
    }
    // 设置用户名
    return YES;
}
{% endhighlight %}

对于点击了“完成”按钮后所做的事，和上述代理方法一样。这样，就完成了对输入框内容类型和长度的限制了。


#### 其他类似的方法

有人也用了其他的方法，那就是在用户进行输入时，不做任何的过滤截取等操作，等待用户输入完成，点击确定的时候，再去判断输入内容的合法性，如果不合法则提示用户自行修改。

个人认为这样的用户体验略差，因为你提示用户不合法后，需要用户自行慢慢修改，或许用户需要修改多次才会合法，如果用了笔者上述的办法，那么将会从源头上保证输入内容是合法的，那么用户就不需要担心后面会被提示再去修改，只需要关注自己的输入框实际显示了什么内容即可。

如果你想用后面这个方法，等待用户输入完成后进行格式判断，那么在确定前调用如下的方法进行判断即可：

{% highlight objc %}
// 判断字符串内容是否合法：2-12位汉字、字母、数字
- (BOOL)validateFormatByRegexForString:(NSString *)string {
    BOOL isValid = YES;
    NSUInteger len = string.length;
    if (len > 0) {
        NSString *numberRegex = @"^[a-zA-Z0-9\u4e00-\u9fa5]{2,12}$";
        NSPredicate *numberPredicate = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", numberRegex];
        isValid = [numberPredicate evaluateWithObject:string];
    }
    return isValid;
}
{% endhighlight %}

