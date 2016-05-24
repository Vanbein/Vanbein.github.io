---
layout: post
title: markdown 语法总结
date: 2015-10-30 23:52:02 +0800
categories: Jekyll
tags: Jekyll markdown
image: /images/head-800x400/-23.png
description: 使用 Github pages 搭建好博客后，就需要使用 markdown 编写自己的文章，经过查找资料和不断尝试后，总结了下面的一些要点，若有不当或者遗漏之处，恳请指出。
homepage: true
toc: true
---

使用 Github pages 搭建好博客后，就需要使用 markdown 编写自己的文章，经过查找资料和不断尝试后，总结了下面的一些要点，若有不当或者遗漏之处，恳请指出。


> 比较系统的英文版[markdown语法](http://daringfireball.net/projects/markdown/syntax)。
>
> 简体中文版的看这:[markdown语法](http://www.appinn.com/markdown/)

### 关于 markdown 编辑器

+ 在 Mac OS X 上，我强烈建议你用 **Mou** 这款免费且十分好用的 Markdown 编辑器，它支持实时预览，既左边是你编辑 Markdown 语言，右边会实时的生成预览效果。不仅如此，Mou 还有一些有趣的偏好设置（Preference），例如主题（Themes）与样式（CSS），它们可以配置出定制化的文本编辑效果与导出效果，如果你对自带的主题与样式不满意还可以到 GitHub 上搜索其它爱好者为 Mou 编写的更多主题样式，导入的方式可以在偏好设置的 Themes 或 CSS 选项中 选择 reload。如果你和我一样采用 `octopress + github` 写博客，我建议把 **Mou** 的Preference 中CSS格式设置为 `Github2` 。

## 一、段落、标题、区块代码

一个段落是由一个以上的连接的行句组成，而一个以上的空行则会划分出不同的段落（空行的定义是显示上看起来像是空行，就被视为空行，例如有一行只有空白和 tab，那该行也会被视为空行），一般的段落不需要用空白或换行缩进。

Markdown 支持两种标题的语法，`Setext` 和 `atx` 形式。`Setext` 形式是用底线的形式，利用 `=`（最高阶标题）和 `-` （第二阶标题），`Atx` 形式在行首插入 1 到 6 个 `#` ，对应到标题 1 到 6 阶，记得在 # 后面留一个空格。

区块引用则使用 email 形式的 `>` 角括号。

语法：


		A First Level Header
		====================
		
		A Second Level Header
		---------------------

		Now is the time for all good men to come to
		the aid of their country. This is just a
		regular paragraph.
		
		The quick brown fox jumped over the lazy
		dog's back.
		
		# Header1
		
		### Header 3
		
		###### hearder6

		> This is a blockquote.
		> 
		> This is the second paragraph in the blockquote.
		>
		> ## This is an H2 in a blockquote

效果为：

A First Level Header
====================

A Second Level Header
---------------------

Now is the time for all good men to come to
the aid of their country. This is just a
regular paragraph.

The quick brown fox jumped over the lazy
dog's back.

# Header1
### Header 3
###### hearder6

> This is a blockquote.
> 
> This is the second paragraph in the blockquote.
>
> ## This is an H2 in a blockquote


* 区块引用可以嵌套（例如：引用内的引用），只要根据层次加上不同数量的 > ：

语法：

	> This is the first level of quoting.
	>
	> > This is nested blockquote.
	>
	> Back to the first level.

显示效果：

> This is the first level of quoting.
>
> > This is nested blockquote.
>
> Back to the first level.


* Markdown 也允许你偷懒只在整个段落的第一行最前面加上 > ：

语法：

	> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
	consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
	Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.
	
	> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
	id sem consectetuer libero luctus adipiscing.

显示效果：

> This is a blockquote with two paragraphs. Lorem ipsum dolor sit amet,
consectetuer adipiscing elit. Aliquam hendrerit mi posuere lectus.
Vestibulum enim wisi, viverra nec, fringilla in, laoreet vitae, risus.

> Donec sit amet nisl. Aliquam semper ipsum sit amet velit. Suspendisse
id sem consectetuer libero luctus adipiscing.

引用的区块内也可以使用其他的 Markdown 语法，包括标题、列表、代码区块等：

语法：

	> ## 这是一个标题。
	> 
	> 1.   这是第一行列表项。
	> 2.   这是第二行列表项。
	> 
	> 给出一些例子代码：
	> 	
	>     return shell_exec("echo $input | $markdown_script");	


显示效果：

> ## 这是一个标题。
> 
> 1.   这是第一行列表项。
> 2.   这是第二行列表项。
> 
> 给出一些例子代码：
> 
>     return shell_exec("echo $input | $markdown_script");


## 二、修辞和强调

* Markdown 使用星号和底线来标记需要强调的区段。
* 一个星号/底线便是斜体，两个表示加粗强调
* 但是如果你的 * 和 _ 两边都有空白的话，它们就只会被当成普通的符号。
* 两个 ~~ 中间的文字表示删除，但是我目前的环境中貌似不支持
语法:

		Some of these words *are emphasized*.
		Some of these words _are emphasized also_.
		Use two asterisks for **strong emphasis**.
		Or, if you prefer, __use two underscores instead__.
	    
	    ~~Strike through~~

显示效果：

Some of these words *are emphasized*.
Some of these words _are emphasized also_.
Use two asterisks for **strong emphasis**.
Or, if you prefer, __use two underscores instead__.

~~Use strike through~~

* 如果要在文字前后直接插入普通的星号或底线，你可以用反斜线：
示例语法：

```
\*this text is surrounded by literal asterisks\*
```
显示效果：

\*this text is surrounded by literal asterisks\*

## 三、列表
#### 1.无序列表

无序列表使用星号、加号和减号来做为列表的项目标记，这些符号是都可以使用的，

使用星号：

```
* Candy.
* Gum.
* Booze.
```

加号：

```
+ Candy.
+ Gum.
+ Booze.
```

和减号

```
- Candy.
- Gum.
- Booze.
```

显示效果都为：

* Candy.
* Gum.
* Booze.

#### 2.有序列表 

有序列表则是使用一般的数字接着一个英文句点作为项目标记：
语法：

```
1. Red
2. Green
3. Blue
```

显示效果：

1. Red
2. Green
3. Blue

***很重要的一点是**，你在列表标记上使用的数字并不会影响输出*
如果你的列表标记写成了了：

```
4. Red
1. Green
1. Blue
6. Yellow
```
显示效果为：

4. Red
1. Green
1. Blue
6. Yellow


*要让列表看起来**更漂亮**，你可以把内容用固定的缩进整理好：*

语法：

	*   Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
	    Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
	    viverra nec, fringilla in, laoreet vitae, risus.
	*   Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
	    Suspendisse id sem consectetuer libero luctus adipiscing.
	*   Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
	Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
	viverra nec, fringilla in, laoreet vitae, risus.
	*   Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
	Suspendisse id sem consectetuer libero luctus adipiscing.


显示效果：

*   Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
    Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
    viverra nec, fringilla in, laoreet vitae, risus.
*   Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
    Suspendisse id sem consectetuer libero luctus adipiscing.
*   Lorem ipsum dolor sit amet, consectetuer adipiscing elit.
Aliquam hendrerit mi posuere lectus. Vestibulum enim wisi,
viverra nec, fringilla in, laoreet vitae, risus.
*   Donec sit amet nisl. Aliquam semper ipsum sit amet velit.
Suspendisse id sem consectetuer libero luctus adipiscing.
  
*如果你在项目之间插入空行，那项目的内容会用 <p> 包起来，你也可以在一个项目内放上多个段落，只要在它前面缩排 4 个空白或 1 个 tab 。*

示例语法：


		* A list item.
		With multiple paragraphs.
		
		* Another item in the list.


显示效果：

* A list item.
With multiple paragraphs.

* Another item in the list.

*列表项目可以包含多个段落，每个项目下的段落都必须缩进 4 个空格或是 1 个制表符：*

语法：


	1.  This is a list item with two paragraphs. Lorem ipsum dolor
	    sit amet, consectetuer adipiscing elit. Aliquam hendrerit
	    mi posuere lectus.
	
	    Vestibulum enim wisi, viverra nec, fringilla in, laoreet
	    vitae, risus. Donec sit amet nisl. Aliquam semper ipsum
	    sit amet velit.
	
	2.  Suspendisse id sem consectetuer libero luctus adipiscing.

效果：

1.  This is a list item with two paragraphs. Lorem ipsum dolor
    sit amet, consectetuer adipiscing elit. Aliquam hendrerit
    mi posuere lectus.

    Vestibulum enim wisi, viverra nec, fringilla in, laoreet
    vitae, risus. Donec sit amet nisl. Aliquam semper ipsum
    sit amet velit.

2.  Suspendisse id sem consectetuer libero luctus adipiscing.


_如果要放代码区块的话，该区块就需要缩进两次，也就是 8 个空格或是 2 个制表符：_


```
*   一列表项包含一个列表区块：
        <代码写在这>
```

*当然，项目列表很可能会不小心产生，像是下面这样的写法：*

	1986. What a great season.

换句话说，也就是在行首出现数字-句点-空白，要避免这样的状况，你可以在句点前面加上反斜杠，其他一下情况也类似。

	1986\. What a great season.

## 四、链接

Markdown 支援两种形式的链接语法： `行内` 和 `参考` 两种形式，两种都是使用角括号来把文字转成连结。

#### 行内形式

行内形式是直接在后面用括号直接接上链接：

语法：

```
This is an [example link](http://example.com/)
//你也可以选择性的加上 title 属性：
This is an [example link](http://example.com/ "With a Title").
```

显示效果：

This is an [example link](http://example.com/)

This is an [example link](http://example.com/ "With a Title").

#### 参考形式

参考形式的链接让你可以为链接定一个名称，之后你可以在文件的其他`任意`地方定义该链接的内容：

语法：

		I get 10 times more traffic from [Google][1] than from
		[Yahoo][2] or [MSN][3].
		
		[1]: http://google.com/ "Google"
		[2]: http://search.yahoo.com/ "Yahoo Search"
		[3]: http://search.msn.com/ "MSN Search"

显示效果：

 get 10 times more traffic from [Google][1] than from
[Yahoo][2] or [MSN][3].

[1]: http://google.com/ "Google"
[2]: http://search.yahoo.com/ "Yahoo Search"
[3]: http://search.msn.com/ "MSN Search"

当然，`title` 属性是选择性的，链接名称可以用字母、数字和空格，但是不分大小写：

示例语法：

		I start my morning with a cup of coffee and
		[The New York Times][NY Times].
		
		[ny times]: http://www.nytimes.com/

显示效果：

I start my morning with a cup of coffee and
[The New York Times][NY Times].

[ny times]: http://www.nytimes.com/


#### 自动链接

Markdown 支持以比较简短的自动链接形式来处理网址和电子邮件信箱，只要是用方括号包起来， Markdown 就会自动把它转成链接。一般网址的链接文字就和链接地址一样，例如：
示例语法:

	<http://example.com/>

显示效果：

<http://example.com/>

## 五、图片

图片的语法和链接很像。

#### 1.行内形式（title 是选择性的）：
语法：


	![alt text](/path/to/img.jpg "Title")

详细叙述如下：

* 一个惊叹号 !
* 接着一个方括号，里面放上图片的替代文字
* 接着一个普通括号，里面放上图片的网址，最后还可以用引号包住并加上 选择性的 'title' 文字。

#### 2.参考形式：

语法：

	![alt text][id]

	[id]: /path/to/img.jpg "Title"


到目前为止， Markdown 还没有办法指定图片的宽高，如果你需要的话，你可以使用普通的 <img> 标签。


## 六、代码区块

如果要标记一小段行内代码，你可以用反引号把它包起来（`），例如：
语法示例：

```
Use the `printf()` function.
```

效果：

Use the `printf()` function.


在一般的段落文字中，你可以使用反引号 ` 来标记代码区段，区段内的 &、< 和 > 都会被自动的转换成 HTML 实体，这项特性让你可以很容易的在代码区段内插入 HTML 码：

示例语法：

```
I strongly recommend against using any `<blink>` tags.
I wish SmartyPants used named entities like `&mdash;`
instead of decimal-encoded entites like `&#8212;`.
```

显示效果：

I strongly recommend against using any `<blink>` tags.

I wish SmartyPants used named entities like `&mdash;`

instead of decimal-encoded entites like `&#8212;`.


要在 Markdown 中建立代码区块很简单，只要简单地缩进 4 个空格或是 1 个制表符就可以，例如，下面的输入：
	
	这是一个普通段落：
	
	    这是一个代码区块。

显示效果：

这是一个普通段落：

    这是一个代码区块。
    
* 一个代码区块会一直持续到没有缩进的那一行（或是文件结尾）。
* 代码区块中，一般的 Markdown 语法不会被转换，像是星号便只是星号，这表示你可以很容易地以 Markdown 语法撰写 Markdown 语法相关的文件。

##### 代码区块还可以用三个`号包起来表示

示例语法：

	```语言类型 代码文件名 代码下载链接
	//这是一个代码区块
	//这种会显示代码行数
	//会自动判断代码语言类型进行语法高亮
	NSString *string = @"Hello world!";
	NSLog(@"%@",string);
	```
	
	```ruby Discover if a number is prime http://www.noulakaz.net/weblog/2007/03/18/a-regular-expression-to-check-for-prime-numbers/ Source Article
	class Fixnum
		def prime?
			('1' * self) !~ /^1?$|^(11+?)\1+$/
		end
	end
	```


显示效果：


```objc Test.m
//这是一个代码区块
//这种会显示代码行数
//会自动判断代码语言类型进行语法高亮
NSString *string = @"Hello world!";
NSLog(@"%@",string);
```

或者：

```ruby Discover if a number is prime http://www.noulakaz.net/weblog/2007/03/18/a-regular-expression-to-check-for-prime-numbers/ Source Article
class Fixnum
  def prime?
    ('1' * self) !~ /^1?$|^(11+?)\1+$/
  end
end
```

但是目前我的环境，貌似不支持上述两种代码高亮形式，所以可以用下面这种：(使用时请去掉 % 前的 \)

	{\% highlight objc linenos \%}
	//这是一个代码区块 这种会显示代码行数
	NSString *string = @"Hello world!";
	NSLog(@"%@",string);
	{\% endhighlight \%}
	
	{\% highlight ruby linenos \%}
	class Fixnum
	  def prime?
	    ('1' * self) !~ /^1?$|^(11+?)\1+$/
	  end
	end
	{\% endhighlight \%}
	
显示效果为：

{% highlight objc linenos %}
//这是一个代码区块 这种会显示代码行数
NSString *string = @"Hello world!";
NSLog(@"%@",string);
{% endhighlight %}

{% highlight ruby linenos %}
class Fixnum
  def prime?
    ('1' * self) !~ /^1?$|^(11+?)\1+$/
  end
end
{% endhighlight %}


## 七、分割线

你可以在你可以在一行中用三个以上的星号、减号、底线来建立一个分隔线，行内不能有其他东西。你也可以在星号或是减号中间插入空格。下面每种写法都可以建立分隔线：


	* * *
	
	***
	
	*****
	
	- - -
	
	---------------------------------------

显示效果都为：

* * *



* 不管您有任何想法，欢迎留言
