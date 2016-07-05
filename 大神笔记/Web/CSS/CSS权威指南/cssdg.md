本书仅覆盖IE7。

[toc]

本书将inline翻译为行内。

## 1 CSS和文档

本书只涵盖CSS2和CSS2.1，不包括CSS3（除了部分CSS3选择器）。

### 1.3 元素

#### 1.3.1 替换元素与非替换元素

CSS中，元素通常有两种形式：可被替换和不可替换。这两种类型将在第7章做详细讨论。这里简短介绍。

- 可被替换元素：可被替换元素（relaced element）是指元素内容并非由文档内容直接表示。例如img元素和input元素。实际上img元素没有内容：`<img src=”howdy.gif” />`。
- 不可替换元素：大多数html和xhtml元素都是不可替换元素（nonreplaced element），其内容在元素本身的框中显示。如div, heading等。

#### 1.3.2 元素显示角色（Display）

CSS2.1使用两种基本元素类型：块级（block–level）元素和行内（inline–level）元素。注意这里讨论的是CSS的概念（不是html的概念）！

**块级元素**

Block-level elements generate an element box that (by default) fills its parent element’s content area and cannot have other elements at its sides. In other words, it generates “breaks” before and after the element box.

列表项也是块级元素。不过它额外产生一个marker。

**行内元素**

Inline-level elements generate an element box within a line of text and do not break up the flow of that line. These elements do not generate a “break” before or after themselves, so they can appear within the content of another element with-out disrupting its display.

注意块和行内这两个概念与HTML中的块级和行内元素不同！在HTML中，块级元素不能放入行级元素中。但CSS不对嵌套做限制。

CSS中与此相关的属性是`display`。HTML中em只能签入p，反过来不行。但通过CSS反转二者的有效的：

	p {display: inline;}
	em {display: block;}

在XML中元素本身没有显示样式，只能通过CSS控制块级还是行级。

### 1.4 结合CSS和XHTML

	<link rel="stylesheet" type="text/css" href="sheet1.css" media="all" />

link元素只能放入head。

外部样式表不能包含任何HTML标记，只能有CSS规则和注释，否则将导致部分或全部被忽略。

link的rel属性代表关系（relation），这里关系是stylesheet。

media属性的取值。指定多个值时用逗号分隔：

- all: Use in all presentational media.
- aural: 读屏器等
- braille: Use when rendering the document with a Braille device.
- embossed: Use when printing with a Braille printing device.
- handheld: 手持设备，如PDA
- print: 打印或打印预览
- projection: 投影或幻灯
- screen: 屏幕，如计算机屏幕
- tty: Use when delivering the document in a fixed-pitch environment like teletype printers.
- tv: 电视

一个文档可能关联多个外部样式表。最初只会使用rel=stylesheet的样式表。如果有多个样式表rel=stylesheet，它们会被合并后使用。

**候选样式表**

生成候选样式表时会使用到title元素，作为标题显示，供用户选择。

	<link rel="stylesheet" type="text/css" href="sheet1.css" title="Default" />
	<link rel="alternate stylesheet" type="text/css" href="bigtext.css" title="Big Text" />
	<link rel="alternate stylesheet" type="text/css" href="zany.css" title="Crazy colors!" />

**首选样式表（略）**

如果没有为样式表指定title，那么它将作为一个永久样式表（persistent style sheet），始终用于文档显示。

CSS注释：`/* */`。

## 4 值和单位

### 颜色

RGB颜色：`rgb(r,g,b)`。取值0%到100%，或0到255。

十六进制RGB颜色。简写三位，如`#666`。

### 长度单位

绝对长度，如cm，用的不多。

相对长度有三个：em、ex和px。

em是给定字体的font–size值。

ex是小写字母x的高度。相同像素数的不同字体的x的高度可能不同。多数字体没有内置的ex值，一般假设为0.5em。

### URL

- 绝对：url(protocol://server/pathname)
- 相对：url(pathname)

### 关键字

如`none`。

`inherit`是所有属性都有的属性值。虽然多数时候不需要使用inherit——自然会继承，但有时是有用的{{默认不继承的时候}}。如指定锚文本颜色：

	#toolbar {background: blue; color: white;}
	#toolbar a {color: inherit;}

	<div id="toolbar">
		<a href="one.html">One</a> | <a href="two.html">Two</a> |
		<a href="three.html">Three</a>
	</div>





































