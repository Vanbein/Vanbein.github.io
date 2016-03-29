---
layout: post
category: Jekyll
title: 转 - 多说评论框自定义CSS优化
tags: Jekyll
image: /images/head-800x400/-3.png
description: 多说的个性化设置里有个自定义CSS，将你要更改的CSS样式代码粘贴进去保存就好了；注意：有些属性如果修改了没效果的话，要在每个属性后面加上一个!important，将优先级调为最高就可以看到效果了，比如：padding:10px; 这个属性没生效的话就要这样改：padding:10px !important; 这样就可以生效了。
toc: true
homepage: false
---

多说的个性化设置里有个自定义CSS，将你要更改的CSS样式代码粘贴进去保存就好了；注意：有些属性如果修改了没效果的话，要在每个属性后面加上一个`!important`，将优先级调为最高就可以看到效果了，比如：`padding:10px`; 这个属性没生效的话就要这样改：`padding:10px !important`; 这样就可以生效了。

下面将网络上现有的几个不错的css样式整理并分享给大家。

## 圆角（或者圆形）+ 阴影

{% highlight CSS linenos %}
#ds-reset .ds-avatar img{
width:54px;height:54px; /*设置图像的长和宽*/
border-radius:27px; /*设置图像圆角效果,在这里我直接设置了超过width/2的像素，即为圆形了*/
-webkit-border-radius: 27px; /*圆角效果：兼容webkit浏览器*/
-moz-border-radius:27px;
box-shadow: inset 0 -1px 0 #3333sf; /*设置图像阴影效果*/
-webkit-box-shadow: inset 0 -1px 0 #3333sf;
}
{% endhighlight %}

## 鼠标悬浮时，圆形+图像进行360度旋转

{% highlight CSS linenos %}
#ds-reset .ds-avatar img{
width:54px;height:54px; /*设置图像的长和宽*/
border-radius:27px; /*设置图像圆角效果,在这里我直接设置了超过width/2的像素，即为圆形了*/
-webkit-border-radius: 27px; /*圆角效果：兼容webkit浏览器*/
-moz-border-radius:27px;
box-shadow: inset 0 -1px 0 #3333sf; /*设置图像阴影效果*/
-webkit-box-shadow: inset 0 -1px 0 #3333sf;
-webkit-transition: 0.4s;
-webkit-transition: -webkit-transform 0.4s ease-out;
transition: transform 0.4s ease-out; /*变化时间设置为0.4秒(变化动作即为下面的图像旋转360读）*/
-moz-transition: -moz-transform 0.4s ease-out;
}

#ds-reset .ds-avatar img:hover{ /*设置鼠标悬浮在头像时的CSS样式*/
box-shadow: 0 0 10px #fff; rgba(255,255,255,.6), inset 0 0 20px rgba(255,255,255,1);
-webkit-box-shadow: 0 0 10px #fff; rgba(255,255,255,.6), inset 0 0 20px rgba(255,255,255,1);
transform: rotateZ(360deg); /*图像旋转360度*/
-webkit-transform: rotateZ(360deg);
-moz-transform: rotateZ(360deg);
}
{% endhighlight %}

## 鼠标悬浮时：圆形+图像进行360度旋转+半圆遮盖

{% highlight CSS linenos %}
#ds-thread #ds-reset ul.ds-comments-tabs li.ds-tab a.ds-current {border:0px;color:#848568;text-shadow:none;background:#dddfc2}
#ds-thread #ds-reset .ds-highlight {font-family:Arial, Helvetica, sans-serif;font-size:14px;font-weight:bold;color:#848568 !important;}
#ds-thread #ds-reset ul.ds-comments-tabs li.ds-tab a.ds-current:hover {color:#696a52;background:#d4d6ba}
#ds-thread #ds-reset a.ds-highlight:hover {color:#696a52 !important;}

#ds-thread {padding-left:30px;}
#ds-thread #ds-reset li.ds-post,#ds-thread #ds-reset #ds-hot-posts {overflow:visible}
#ds-thread #ds-reset .ds-post-self {padding:10px 0 10px 10px;}
#ds-thread #ds-reset li.ds-post,#ds-thread #ds-reset .ds-post-self {border:0 !important;}
#ds-reset .ds-avatar, #ds-thread #ds-reset ul.ds-children .ds-avatar {position:absolute;top:26px;left:-14px;padding:5px;width:36px;height:36px;box-shadow:-1px 0 1px rgba(0,0,0,.15) inset;border-radius:46px; background:#E5E6D0;}
#ds-thread #ds-reset ul.ds-children .ds-avatar {left:-23px;}
#ds-thread .ds-avatar a {display:inline-block;padding:1px; width:32px;height:32px;border:1px solid #b9baa6;border-radius:50%; background-color:#fff !important}
#ds-thread .ds-avatar a:hover {border-color:#de5a4e}
#ds-thread .ds-avatar > img {margin:2px 0 0 2px}
#ds-thread #ds-reset .ds-replybox {box-shadow:none;}
#ds-thread #ds-reset ul.ds-children .ds-replybox.ds-inline-replybox a.ds-avatar,
#ds-reset .ds-replybox.ds-inline-replybox a.ds-avatar {left: 0;top: 0; padding: 0;width: 32px !important;height: 32px !important; background: none;box-shadow: none; }
#ds-reset .ds-replybox.ds-inline-replybox a.ds-avatar img {width: 32px !important;height: 32px !important; border-radius:50%;}
#ds-reset .ds-replybox a.ds-avatar,
#ds-reset .ds-replybox .ds-avatar img { padding:0;width:50px !important;height:50px !important; border-radius:5px; }
#ds-reset .ds-avatar img {width:32px !important;height:32px !important;border-radius:32px;box-shadow:0 1px 3px rgba(0, 0, 0, 0.22);
-webkit-transition:.4s all ease-in-out;-moz-transition:.4s all ease-in-out;-o-transition:.4s all ease-in-out;-ms-transition:.4s all ease-in-out;transition:.4s all ease-in-out;
}
.ds-post-self:hover .ds-avatar img{
-webkit-transform:rotate(360deg);-moz-transform:rotate(360deg);-o-transform:rotate(360deg);-ms-transform:rotate(360deg);transform:rotate(360deg);}

#ds-thread #ds-reset .ds-comment-body {background: #F0F0E3;padding:15px 15px 12px 32px;border-radius:5px; box-shadow:0 1px 2px rgba(0,0,0,.15), 0 1px 0 rgba(255,255,255,.75) inset;}

#ds-thread #ds-reset .ds-comment-body p{color:#787968}
#ds-thread #ds-reset .ds-comments a.ds-user-name {font-weight:bold;color:#696A52 !important;}
#ds-thread #ds-reset .ds-comments a.ds-user-name:hover {color:#D32 !important;}

#ds-thread #ds-reset #ds-hot-posts {border:0}
#ds-reset #ds-hot-posts .ds-gradient-bg {background:none;}

#ds-reset #ds-bubble #ds-ctx .ds-ctx-entry {padding:0;}
#ds-reset #ds-bubble .ds-avatar, #ds-reset #ds-bubble #ds-ctx-bubble .ds-avatar a {position:static;padding:0;border:0; background:none;box-shadow:none;}
#ds-reset #ds-bubble .ds-avatar img, #ds-reset #ds-bubble #ds-ctx-bubble .ds-avatar a {width:45px !important;height:45px !important;}
#ds-reset #ds-bubble .ds-user-name{padding-left:13px;}

#ds-reset .ds-comment-body #ds-ctx {border-left:1px solid #b9baa6;background-color:#e8e8dc !important}
#ds-reset #ds-ctx {margin-right:-15px}
#ds-reset #ds-ctx .ds-ctx-entry {position:relative;padding:10px 30px 10px 10px;}
#ds-reset #ds-ctx .ds-ctx-entry .ds-avatar {top:6px;left:5px;background:none;box-shadow:none;}
#ds-reset #ds-ctx .ds-ctx-entry .ds-ctx-body {margin-left:46px;}
#ds-reset #ds-ctx .ds-ctx-entry .ds-ctx-content {color:#787968}
#ds-reset #ds-ctx .ds-ctx-entry .ds-ctx-head a {color:#696A52;font-weight:bold}
{% endhighlight %}

简要说明：

配色是非常重要的，特别是记得改「#ds-reset .ds-avatar, #ds-thread #ds-reset ul.ds-children .ds-avatar {background:网站的背景颜色}」。

**用户气泡提示框代码:** 隐藏用户气泡提示框「#ds-thread #ds-reset #ds-bubble {display:none !important}」，这是隐藏鼠标移至用户名称时弹出来的气泡提示框。

**评论盖楼样式代码:** 倒数1-8行。（不用盖楼的评论方式用户可以直接删除这几行，精简一下代码。）

## 多说css自定义代码（用于细微调整）：

### 将评论框底部的分享到微博QQ空间什么的隐藏起来

{% highlight CSS linenos %}
.ds-sync{
display:none !important;
}/*隐藏评论框底部分享*/
{% endhighlight %}

### 定义评论框背景图片

{% highlight CSS linenos %}
.ds-textarea-wrapper.ds-rounded-top{
background: #ffffff url(这里添加图片的网址) no-repeat right bottom !important;
}/*定义评论框背景背景*/
{% endhighlight %}

注意图片最好是透明或者白色的，图片的像素大小最好要去你的评论框大小一样，不然可能会出现拉伸。

### 隐藏评论框底部渐变背景

{% highlight CSS linenos %}
#ds-reset .ds-gradient-bg{
background:none !important;
}/*设置评论框下方渐变*/
{% endhighlight %}

话说渐变色什么的虽然立体感较强但是和主题整体风格不融洽，隐藏之，这样底部就是透明的了。

### 定义评论框内字体

{% highlight CSS linenos %}
#ds-thread #ds-reset .ds-textarea-wrapper textarea, #ds-thread #ds-reset .ds-textarea-wrapper .ds-hidden-text {
font-family: ‘微软雅黑’ ‘Microsoft Yahei’!important;
font-size:12px;
letter-spacing:1px;
}/*定义评论框内字体*/
{% endhighlight %}

这个好像是定义评论框内输入的文字字体的。

### 定义发布按钮字体，以及渐变色背景

{% highlight CSS linenos %}
#ds-thread #ds-reset .ds-post-button{
font-family: ‘微软雅黑’ ‘Microsoft Yahei’!important;
font-weight: bold;
font-size:12px;
background:none !important;
color:#49976b !important;
}/*定义发布按钮字体以及渐变背景*/
{% endhighlight %}

### 隐藏评论右上方 最热 最新 排序 按钮

{% highlight CSS linenos %}
#ds-thread #ds-reset .ds-sort {
display:none;
}/*隐藏最新最热等*/
{% endhighlight %}

### 隐藏评论左上方 评论总数背景色及边框

{% highlight CSS linenos %}
#ds-thread #ds-reset li.ds-tab a.ds-current{
background:none;
border:none;
}/*隐藏评论总数背景及边框*/
{% endhighlight %}

### 隐藏底部多说版权

{% highlight CSS linenos %}
#ds-thread #ds-reset .ds-powered-by{
display:none;
}/*隐藏多说底部版权*/
{% endhighlight %}

### 定义各种文字高亮颜色，以及浮动窗口的高亮颜色

{% highlight CSS linenos %}
#ds-reset .ds-highlight{
color:#49976b !important;
}/*定义高亮字体颜色*/

#ds-thread #ds-reset #ds-bubble a{
color: #49976b !important;
}/*定义评论框内其他高亮颜色*/

#ds-thread #ds-reset #ds-bubble {
color: #49976b !important;
}/*定义评论框内其他高亮颜色*/

#ds-reset #ds-ctx .ds-ctx-entry .ds-ctx-head a{
color: #49976b !important;
}/*定义评论框内其他高亮颜色*/

#ds-thread #ds-reset a.ds-comment-context:hover{
color: #49976b !important;
}/*定义评论框内其他高亮颜色*/

#ds-thread #ds-reset a.ds-comment-context{
color: #49976b !important;
}/*定义评论框内其他高亮颜色*/
{% endhighlight %}

### 评论框左右边距

如果你的评论框左右边距过小（评论框太宽），输入下列代码调整宽度，直到页面上评论框宽度显示合适：

```
#ds-thread  {padding:24px;}
```

或

```
#ds-thread {margin:24px;}
```

如果你的评论框太窄，可能是宽度被设定了不合适的值，输入下列代码让宽度自动拉伸：

```
#ds-thread {width:auto;}
```

### 评论框整体的背景色

多说评论会采用主题的背景色作为整体评论框的背景，这样可能导致评论本身不是很显眼。你可以输入下列代码来更改整体评论框的背景颜色：

```
#ds-thread {background: #ffffff;}
```

这个评论背景的边角默认是直角，如果想改成圆角，请输入下列代码（仅在Firefox，Chrome及高版本ie浏览器下有效，ie6,7,8将仍然为直角显示）：

```
#ds-thread{ border-radius: 5px;}
```
注意：其中的#ffffff可以被替换为你希望的颜色，以便于评论文字相适应。

### 高亮字体的颜色

高亮字体包括“n条评论”，“n条微博”，评论者名字的颜色，想修改它的显示颜色（在大多数情况下默认是红色），输入下列代码：

```
#ds-thread #ds-reset .ds-highlight{color: #ffffff !important;}
```

### 更改评论字体颜色

想修改评论正文的字体颜色，请输入下列代码：

```
#ds-thread #ds-reset .ds-comment-body p {color: #ffffff;}
```

当您在修改一部分上面未示例的标签样式时，遇到无效的情况，请尝试增加`!important`。

### 更改最近访客头像大小

想要修改最新访客头像的大小，请输入下列代码：

```
#ds-recent-visitors .ds-avatar img {width: 30px !important;height: 30px !important;}
```

其中30px是宽和高，请根据效果修改不同高度和宽度。

---
📎 **引用来源：**[http://blog.flycmd.com/archives/29](http://blog.flycmd.com/archives/29)
