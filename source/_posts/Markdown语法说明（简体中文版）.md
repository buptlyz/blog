---
title: Markdown语法说明（简体中文版）(转)
date: 2019-04-17 18:32:30
tags:
---
[原文链接](https://www.appinn.com/markdown/)

## 概述
#### 宗旨
易读易写
#### 兼容HTML
目标是：成为一种适用于网络的书写语言。
#### 特殊字符自动转换
在HTML文件中，`<`和`&`需要特殊处理，如果想显示这两个字符的原型，需要使用实体的形式，如`&lt;`和`&amp;`。
如果要插入版权`©`，需要写成`&copy;`。

## 区块元素
#### 段落和换行
一个段落是文本行组成的，前后需要一个以上的空行。
如果某一行只包含空格和制表符，也会被视为空行。
Markdown允许段落内插入换行符，只需要在插入处按入两个以上的空格，然后回车。相当于插入一个`<br />`标签。
#### 标题
类Setext形式是用底线的形式，`=`（最高阶标题）`-`（第二阶标题），例如：
```markdown
This is an H1
==============

This is an H2
-------------
```
任何数量的`=`和`-`都可以有这个效果。  

类Atx形式则是在行首插入1到6个`#`，对应H1~H6，例如：
```markdown
# 这是 H1
## 这是 H2
###### 这是 H6`
```

#### 区块引用 Blockquotes
类似于email中用`>`的引用方式。
```markdown
> This is a blockquote with two paragraphs. I
> have 
> a dream today.
>
> by Martin Luther King
```
也可以只在第一行最前面加`>`：
```markdown
> This is a blockquote with two paragraphs. I
have 
a dream today.

> by Martin Luther King
```
引用的区块内也可以使用其它的Markdown语法，包括标题、列表、代码区块等：
```markdown
> ## 这是一个标题。
>
> 1. 这是第一行列表项
> 2. 这是第二行列表项
> 给出一些例子代码：
> 
>     return shell_exec("echo $input | $markdown_script");
```
#### 列表
Markdown支持有序列表和无序列表。
无序列表使用星号、加号或是减号作为标记：
```markdown
* Red
* Green
* Blue
```
等同于：
```markdown
+ Red
+ Green
+ Blue

- Red
- Green
- Blue
```
有序列表则使用数字接着一个英文句点：
```markdown
1. Bird
2. Mchale
3. Parish
```
很重要的一点：列表标记上使用的数字不会影响HTML结果。
```markdown
1. Bird
1. Mchale
1. Parish

3. Bird
1. Mchale
8. Parish
```
都会得到完全相同的HTML输出。
列表项目标记通常是放在最左边，后面一定要接着至少一个空格或制表符。
列表项目可以包含多个段落，每个项目下的段落都必须缩进4个空格或者1个制表符：
```markdown
1. This is a list item with two paragraphs.
    This is the other paragraph.
2. Another item.
```
如果要在列表项目内放进引用，那`>`就需要缩进：
```markdown
* A list item with a blockquote:
    > This is a blockquote
    > inside a list item.
```
如果要放代码区块的话，该区块需要缩进8个空格或者2个制表符：
```markdown
* A list item with code block
        function foo() {}
```
首行出现数字-句点-空白就会被视为有序列表，可以用`\`来转义：
```markdown
1986\. What a great season.
```
#### 代码区块
只要简单的缩进4个空格或者1个制表符就可以在Markdown中建立代码区块。
```markdown
A normal paragraph.

    function foo() {}
```
一个代码区块会一直持续到没有缩进的那一行（或是文件末尾）。
在代码区块里面，`&`、`<`和`>`都会自动转成HTML实体。
代码区块中，一般的Markdown语法不会被转换，像星号就是星号，可以很容易的用Markdown语法撰写Markdown语法相关文件。
#### 分隔线
用三个以上的星号、减号、底线来建立分隔线。行内不能有其它东西。可以在星号或减号中间插入空格。
```
***
* * *
******
___
_ _ _
_____________
```
## 区段元素
#### 链接
链接文字用[方括号]来标记。
要建立一个行内式的链接，只要在方括号后面紧跟插入网址链接的圆括号即可。如果想添加title文字，只要在网址后，用双引号把title文字包起来即可。
```markdown
This is [an example](http://example.com/ "Title") inline link.
[This link](http://example.net/) has no title attribute.
```
也可以使用相对路径：
```markdown
See my [About](/about/) page for details.
```
参考式的链接是在链接文字的括号后再加上一个方括号，并填入用以辨识链接的标记：
```markdown
This is [an example][id] reference-style link.
```
接着，在文件的任意处，可以把这个标记的链接内容定义出来：
```markdown
[id]: http://example.com/ "Optional Title Here"
```
链接内容定义的形式为：
* 方括号（前面可以选择性的加上至多3个空格来缩进）和链接文字
* 冒号
* 一个以上的空格或制表符
* 链接地址，可以用尖括号包起来
* title，可以用单引号、双引号或者括号包起来（可以放到下一行，也可以加一些缩进来美化）

以下五种的定义是相同的：
```markdown
[foo]: http://example.com/  "Optional Title Here"
[foo]: http://example.com/  'Optional Title Here'
[foo]: http://example.com/  (Optional Title Here)
[foo]: <http://example.com/>  "Optional Title Here"
[foo]: http://example.com/longish/path/to/resource/here
    "Optional Title Here"
```
链接辨识标签可以由字母、数字、空白和标点符号组成，不区分大小写。`[link text][a]`和`[link text][A]`是相同的。
隐式链接标记功能可以省略指定链接辨识标签，这时，链接辨识id等同于链接文字：
```markdown
[Google][]

[Google]: https://google.com/

Visit [Daring Fireball][] for more information.

[Daring Fireball]: http://daringfireball.net/
```
#### 强调
用星号（`*`）和底线（`_`）作为强调符号，一个符号相当于`<em>`，两个符号相当于`<strong>`：
```markdown
*single asterisks*

_single asterisks_

**single asterisks**

__single asterisks__
```
可以用`\`转义。
#### 代码
如果要标记行内代码，可以用反引号包起来（`` ` ``）。
如果要在代码区段内插入反引号，可以用多个反引号来开启和结束代码区段。
```markdown
`` A paragraph with (`) in it. ``
```
#### 图片
行内式
```markdown
![Alt text](/path/to/img.jpg)
![Alt text](/path/to/img.jpg "Optional title")
```
* 一个感叹号！
* 接着一个方括号，填入图片的替代文本alt
* 接着一个普通括号，填入图片的地址，并加上选填的title

参考式
```markdown
![Alt text][id]

[id]: url/to/image "Optional title"
```
到目前为止，没有办法指定图片的宽高，如果需要的话，可以使用普通的`<img>`标签。
## 其它
#### 自动链接

    <http://example.com/>

#### 反斜杠
Markdown支持转义的符号：
```markdown
\ 
`
*
_
{}
[]
()
#
+
-
.
!
```