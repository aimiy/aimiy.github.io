---
layout: post
title:  sass,less,stylus的区别
date:   2021-06-04 10:29:00 +0800
categories: 笔记
tag: css预处理器
---
* content
{:toc}

## CSS预处理器

### 1.什么是 CSS 预处理器

CSS 预处理器是一种语言用来为 CSS 增加一些编程的的特性，无需考虑浏览器的兼容性问题，例如你可以在 CSS 中使用变量、简单的程序逻辑、函数等等在编程语言中的一些基本技巧，可以让CSS 更见简洁，适应性更强，代码更直观等诸多好处。

>sass：2007年诞生，最早也是最成熟的CSS预处理器，拥有ruby社区的支持和compass这一最强大的css框架，目前受LESS影响，已经进化到了全面兼容CSS的SCSS。
>Less：2009年出现，受SASS的影响较大，但又使用CSS的语法，让大部分开发者和设计师更容易上手，在ruby社区之外支持者远超过SASS，其缺点是比起SASS来，可编程功能不够，不过优点是简单和兼容CSS，反过来也影响了SASS演变到了SCSS的时代，著名的Twitter Bootstrap就是采用LESS做底层语言的。
>Stylus：2010年产生，来自Node.js社区，主要用来给Node项目进行CSS预处理支持，在此社区之内有一定支持者，在广泛的意义上人气还完全不如SASS和LESS。

### 2.基本语法区别

在使用 CSS 预处理器之前最重要的是理解语法，幸运的是基本上大多数预处理器的语法跟 CSS 都差不多。

首先 Sass 和 Less 都使用的是标准的 CSS 语法，因此如果可以很方便的将已有的 CSS 代码转为预处理器代码，默认 Sass 使用 .sass 扩展名，而 Less 使用 .less 扩展名。

```css
/*style.scss or style.less*/
h1 {
  color: #0982C1;
}
```

这是一个再普通不过的，不过 Sass 同时也支持老的语法，就是不包含花括号和分号的方式：

```css
/*style.sass*/
h1
  color: #0982c1
```

而 `Stylus` 支持的语法要更多样性一点，它默认使用 .styl 的文件扩展名，下面是 Stylus 支持的语法：

```css
/*style.styl*/
h1 {
  color: #0982C1;
}

/*omit brackets*/
h1
  color: #0982C1;

/*omit colons and semi-colons*/
h1
  color #0982C1
```

可以在同一个样式单中使用不同的变量，例如下面的写法也不会报错：

```css
h1 {
  color #0982c1
}
h2
  font-size: 1.2em
```

### 3.变量

预处理器|开头|变量声明衔接 | css之间衔接
---|---|---|---
sass|$|**:**|**:**
less|@|**:**|**:**
stylus|可以是以$开头，或者任何的字符，不能用@开头|=|:或者空格

#### 1. sass

Sass让人们受益的一个重要特性就是它为css引入了变量。你可以把反复使用的css属性值 定义成变量，然后通过变量名来引用它们，而无需重复书写这一属性值。

>sass变量必须是以$开头的，然后变量和值之间使用冒号（：）隔开，和css属性是一样的，例如：

```sass
$maincolor : #092873;
$siteWidth : 1024px;
$borderStyle : dotted;
body {
  color: $maincolor;
  border: 1px $borderStyle $mainColor;
  max-width: $siteWidth;
}
```

#### 2.less css

在less文件中，当一个值需要反复使用时，可以通过@符号定义变量。已经被赋值的变量以及其他的常量（如像素、颜色等）都可以参与运算。

 Less css中变量都是用@开头的，其余与sass都是一样的，例如：

```less
@maincolor : #092873;
@siteWidth : 1024px;
@borderStyle : dotted;
body {
  color: @maincolor;
  border: 1px @borderStyle @mainColor;
  max-width: @siteWidth;
}
```

#### 3.stylus

stylus对变量是没有任何设定的，可以是以$开头，或者任何的字符，而且与变量之间可以用冒号，空格隔开，但是在stylus中不能用@开头，例如：

```css stylus
maincolor = #092873
siteWidth = 1024px
borderStyle = dotted
body
  color maincolor
  border 1px borderStyle mainColor
  max-width siteWidth
```

以上三种写法都如同一下这种css：

```css
body {
  color: #092873;
  border: 1px dotted #092873;
  max-width: 1024px;
}
```

这样做的好处也是显而易见的，在修改多处相同颜色的时候，这时就只需要修改变量值即可。

### 4.嵌套

如果我们需要在CSS中相同的 parent 引用多个元素，这将是非常乏味的，你需要一遍又一遍地写 parent。例如：

```css
div {
  margin: 10px;
}
div nav {
  height: 25px;
}
div nav a {
  color: #0982C1;
}
div nav a:hover {
  text-decoration: underline;
}
```

如果用 CSS 预处理器，就可以少写很多单词，而且父子节点关系一目了然，并且sass，Less，stylus都支持下面这样的写法，且都是相同的：

```css
//scss style //-----------------------------------
nav {
    ul {
       margin: 0;
       padding: 0;
    }
    li {
       display: inline-block;
    }
    a {
       display: block;
       padding: 6px 12px;
       text-decoration: none;
    }
}
//css style //-----------------------------------
nav ul {
    margin: 0;
    padding: 0;
    list-style: none;
}
nav li {
    display: inline-block;
}
nav a {
    display: block;
    padding: 6px 12px;
    text-decoration: none;
}
```

这样做是非常方便的，也很直观。

### 5.运算符

在 CSS 预处理器中还是可以进行样式的计算如下：

```css
body {
  margin: (14px/2);
  top: 50px + 100px;
  right: 80 * 10%;
}
```

在sass，Less与stylus中都是可以这样做的。

### 6.颜色函数

CSS 预处理器一般都会内置一些颜色处理函数用来对颜色值进行处理，例如加亮、变暗、颜色梯度等。

#### 1.sass的颜色处理函数

```sass
lighten($color, 10%);
darken($color, 10%);  
saturate($color, 10%);
desaturate($color, 10%);
grayscale($color);  
complement($color);
invert($color);
mix($color1, $color2, 50%);
```

实例如下：

```sass
$color: #0982C1;
h1 {
  background: $color;
  border: 3px solid darken($color, 50%);
}
```

#### 2.Less css颜色处理函数

```less
lighten(@color, 10%);
darken(@color, 10%);  
saturate(@color, 10%);  
desaturate(@color, 10%);
spin(@color, 10);
spin(@color, -10);
mix(@color1, @color2);
```

示例如下：

```less
@color: #0982C1;
h1 {
  background: @color;
  border: 3px solid darken(@color, 50%);
}
```

#### 3.Stylus颜色处理函数

```stylus
lighten(color, 10%);
darken(color, 10%);  
saturate(color, 10%);  
desaturate(color, 10%);
```

示例如下:

```stylus
color = #0982C1
h1
  background color
  border 3px solid darken(color, 50%)
```

### 7.导入 (Import)

很多 CSS 开发者对导入的做法都不太感冒，因为它需要多次的 HTTP 请求。但是在 CSS 预处理器中的导入操作则不同，它只是在语义上包含了不同的文件，但最终结果是一个单一的 CSS 文件，如果你是通过 `@ import “file.css”;` 导入 CSS 文件，那效果跟普通的 CSS 导入一样。

注意：导入文件中定义的混入、变量等信息也将会被引入到主样式文件中，因此需要避免它们互相冲突。
例如：
1.css:

```css
//1.css
/*file.{type}*/
body {
  background: #000;
}
```

2.XXX:

```css
@ import "1.css";
@ import "file.{type}";

p {
  background: #092873;
}
```

最终生成的 CSS：

```css
@ import "1.css";
body {
  background: #000;
}
p {
  background: #092873;
}
```

### 8.继承

当我们需要为多个元素定义相同样式的时候，我们可以考虑使用继承的做法.

#### 1.sass和stylus一样

sass可通过@extend来实现代码组合声明，使代码更加优越简洁。

```css sass
.message {
  border: 1px solid #ccc;
  padding: 10px;
  color: #333;
}
.success {
  @extend .message;
  border-color: green;
}
.error {
  @extend .message;
  border-color: red;
}
.warning {
  @extend .message;
  border-color: yellow;
}
```

#### 2.Less css

但是在这方面 Less 表现的稍微弱一些，更像是混入写法：

```css sass
.message {
  border: 1px solid #ccc;
  padding: 10px;
  color: #333;
}
.success {
  .message;
  border-color: green;
}
.error {
  .message;
  border-color: red;
}
.warning {
  .message;
  border-color: yellow;
}
```

上面两种写法其最终呈现的css样式都如下：

```css
.message, .success, .error, .warning {
  border: 1px solid #cccccc;
  padding: 10px;
  color: #333;
}
.success {
  border-color: green;
}
.error {
  border-color: red;
}
.warning {
  border-color: yellow;
}
```

.message的样式将会被插入到相应的你想要继承的选择器中，但需要注意的是`优先级`的问题。

### 9.Mixins（混入）

Mixins 有点像是函数或者是宏，当某段 CSS 经常需要在多个元素中使用时，可以为这些共用的 CSS 定义一个 Mixin，然后只需要在需要引用这些 CSS 地方调用该 Mixin 即可。

#### 1.Sass 的混入语法

sass中可用`mixin`定义一些代码片段，且可传参数，方便日后根据需求调用。比如说处理css3浏览器前缀：

```css sass
@mixin error($borderWidth: 2px) {
  border: $borderWidth solid #F00;
  color: #F00;
}
.generic-error {
  padding: 20px;
  margin: 4px;
  @ include error(); //这里调用默认 border: 2px solid #F00;
}
.login-error {
  left: 12px;
  position: absolute;
  top: 20px;
  @ include error(5px); //这里调用 border:5px solid #F00;
}
```

#### 2.Less CSS 的混入语法

less也支持带参数的混合以及有默认参数值的混合，如下面的例子所示：

```css less
.error(@borderWidth: 2px) {
  border: @borderWidth solid #F00;
  color: #F00;
}
.generic-error {
  padding: 20px;
  margin: 4px;
  .error(); //这里调用默认 border: 2px solid #F00;
}
.login-error {
  left: 12px;
  position: absolute;
  top: 20px;
  .error(5px); //这里调用 border:5px solid #F00;
}
```

#### 3.Stylus 的混入语法

```css stylus
error(borderWidth= 2px) {
  border: borderWidth solid #F00;
  color: #F00;
}
.generic-error {
  padding: 20px;
  margin: 4px;
  error();
}
.login-error {
  left: 12px;
  position: absolute;
  top: 20px;
  error(5px);
}
```

他们最终呈现的效果都如下：

```css
.generic-error {
  padding: 20px;
  margin: 4px;
  border: 2px solid #f00;
  color: #f00;
}
.login-error {
  left: 12px;
  position: absolute;
  top: 20px;
  border: 5px solid #f00;
  color: #f00;
}
```

### 10.3D文本

要生成具有 3D 效果的文本可以使用 `text-shadows` ，唯一的问题就是当要修改颜色的时候就非常的麻烦，而通过 mixin 和颜色函数可以很轻松的实现：

#### 1.sass3D文本

```css sass
@mixin text3d($color) {
  color: $color;
  text-shadow: 1px 1px 0px darken($color, 5%),
               2px 2px 0px darken($color, 10%),
               3px 3px 0px darken($color, 15%),
               4px 4px 0px darken($color, 20%),
               4px 4px 2px #000;
}

h1 {
  font-size: 32pt;
  @ include text3d(#0982c1);
}
```

#### 2.Less 3D文本

```css less
.text3d(@color) {
  color: @color;
  text-shadow: 1px 1px 0px darken(@color, 5%),
               2px 2px 0px darken(@color, 10%),
               3px 3px 0px darken(@color, 15%),
               4px 4px 0px darken(@color, 20%),
               4px 4px 2px #000;
}

span {
  font-size: 32pt;
  .text3d(#0982c1);
}
```

#### 3.Stylus：3D文本

```css stylus
text3d(color)
  color: color
  text-shadow: 1px 1px 0px darken(color, 5%),
               2px 2px 0px darken(color, 10%),
               3px 3px 0px darken(color, 15%),
               4px 4px 0px darken(color, 20%),
               4px 4px 2px #000
span
  font-size: 32pt
  text3d(#0982c1)
```

其生成的css最终的效果如下：

```css
span {
  font-size: 32pt;
  color: #0982c1;
  text-shadow: 1px 1px 0px #097bb7,
               2px 2px 0px #0875ae,
               3px 3px 0px #086fa4,
               4px 4px 0px #07689a,
               4px 4px 2px #000;
}
```

### 11.列 (Columns)

使用数值操作和变量可以很方便的实现适应屏幕大小的布局处理。

#### 1.Sass：列 (Columns)

```css sass
$siteWidth: 1024px;
$gutterWidth: 20px;
$sidebarWidth: 300px;
body {
  margin: 0 auto;
  width: $siteWidth;
}
.content {
  float: left;
  width: $siteWidth - ($sidebarWidth+$gutterWidth);
}
.sidebar {
  float: left;
  margin-left: $gutterWidth;
  width: $sidebarWidth;
}
```

#### 2.Less CSS：列 (Columns)

```css less
@siteWidth: 1024px;
@gutterWidth: 20px;
@sidebarWidth: 300px;

body {
  margin: 0 auto;
  width: @siteWidth;
}
.content {
  float: left;
  width: @siteWidth - (@sidebarWidth+@gutterWidth);
}
.sidebar {
  float: left;
  margin-left: @gutterWidth;
  width: @sidebarWidth;
}
```

#### 3.Stylus：列 (Columns)

```css stylus
siteWidth = 1024px;
gutterWidth = 20px;
sidebarWidth = 300px;

body {
  margin: 0 auto;
  width: siteWidth;
}
.content {
  float: left;
  width: siteWidth - (sidebarWidth+gutterWidth);
}
.sidebar {
  float: left;
  margin-left: gutterWidth;
  width: sidebarWidth;
}
```

其最终生成的css效果如下：

```css
body {
  margin: 0 auto;
  width: 1024px;
}
.content {
  float: left;
  width: 704px;
}
.sidebar {
  float: left;
  margin-left: 20px;
  width: 300px;
}
```

### 12.高级语法

#### 1.在sass中，还支持条件语句

>@if可一个条件单独使用，也可以和@else结合多条件使用

代码如下：

```css sass
$lte7: true;
$type: monster;
.ib{
    display:inline-block;
    @if $lte7 {
        *display:inline;
        *zoom:1;
    }
}
p {
  @if $type == ocean {
    color: blue;
  } @else if $type == matador {
    color: red;
  } @else if $type == monster {
    color: green;
  } @else {
    color: black;
  }
}
```

其最终的css代码如下：

```css
.ib{
    display:inline-block;
    *display:inline;
    *zoom:1;
}
p {
  color: green;
}
```

#### 2.除却条件语句，sass还支持for循环

for循环有两种形式，分别为：

```css sass
1.@for $var from <start> through <end>
2.@for $var from <start> to <end>。
```

其中$i表示变量，start表示起始值，end表示结束值，这两个的区别是关键字through表示包括end这个数，而to则不包括end这个数。

```css sass
@for $i from 1 to 10 {
  .border-#{$i} {
    border: #{$i}px solid blue;
  }
}
```

#### 同时也支持`while`循环

```css sass
$i: 6;
@while $i > 0 {
  .item-#{$i} { width: 2em * $i; }
  $i: $i - 2;
}
```

#### 最后，同时支持each命令，作用与for类似

```css sass
$animal-list: puma, sea-slug, egret, salamander;
@each $animal in $animal-list {
  .#{$animal}-icon {
    background-image: url('/images/#{$animal}.png');
  }
}
```

其css最终效果如下：

```css
.puma-icon {
  background-image: url('/images/puma.png');
}
.sea-slug-icon {
  background-image: url('/images/sea-slug.png');
}
.egret-icon {
  background-image: url('/images/egret.png');
}
.salamander-icon {
  background-image: url('/images/salamander.png');
}
```

参考文档：
[浅谈css预处理器，Sass、Less和Stylus](https://zhuanlan.zhihu.com/p/23382462)
[Sass、LESS 和 Stylus区别总结](https://blog.csdn.net/pedrojuliet/article/details/72887490)
