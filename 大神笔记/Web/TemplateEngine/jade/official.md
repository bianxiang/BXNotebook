[toc]

## 参考

### （未）Doctype

```jade
doctype html
```

```html
	<!DOCTYPE htm>
```

### 注释

**单行注释**

```jade
// just some paragraphs
p foo
p bar
```

```html
	<!-- just some paragraphs-->
    <p>foo</p>
    <p>bar</p>
```

不输出（unbuffered）的注释，只要加一个中划线：

```jade
//- will not output within markup
p foo
p bar
```

```html
    <p>foo</p>
    <p>bar</p>
```

**块注释**

```jade
body
  //
    As much text as you want
    can go here.
```

```html
    <body>
      <!--
      As much text as you want
      can go here.
      -->
    </body>
```

**条件注释**

条件注释并没有特殊的语法。If your line begins with `<` then it is treated as plain text. So just use normal HTML syle conditional comments:

```jade
    <!--[if IE 8]>
    <html lang="en" class="lt-ie9">
    <![endif]-->
    <!--[if gt IE 8]><!-->
    <html lang="en">
    <!--<![endif]-->
```

```html
    <!--[if IE 8]>
    <html lang="en" class="lt-ie9">
    <![endif]-->
    <!--[if gt IE 8]><!-->
    <html lang="en">
    <!--<![endif]-->
```

### Attributes

`a(href='google.com')`中，属性`href`的值`'google.com'`看上去像纯文本，实际是Javascript表达式。

```jade
- var authenticated = true
body(class=authenticated ? 'authed' : 'anon')
```

```html
	<body class="authed"></body>
```

如果有多个属性，可以写到多行上：

```jade
input(
  type='checkbox'
  name='agreement'
  checked
)
```

**不转义的属性**

默认所有的属性都会被转义以防止攻击（如跨域脚本）。若不想转义，可以用`!=`替代`=`。

```jade
    div(escaped="<code>")
    div(unescaped!="<code>")
```

```html
    <div escaped="&lt;code&gt;"></div>
    <div unescaped="<code>"></div>
```

**布尔属性**

Boolean attributes are mirrored by Jade, and accept bools, aka `true` or `false`. When no value is specified `true` is assumed.

```jade
input(type='checkbox', checked)
input(type='checkbox', checked=true)
input(type='checkbox', checked=false)
input(type='checkbox', checked=true.toString())
```

```html
    <input type="checkbox" checked="checked"/>
    <input type="checkbox" checked="checked"/>
    <input type="checkbox"/>
    <input type="checkbox" checked="true"/>
```

如果doctype是`html`，jade knows not to mirror the attribute，使用简洁的风格（所有浏览器都支持）。

```jade
doctype html
input(type='checkbox', checked)
input(type='checkbox', checked=true)
input(type='checkbox', checked=false)
input(type='checkbox', checked=true && 'checked')
```

```html
    <!DOCTYPE html>
    <input type="checkbox" checked>
    <input type="checkbox" checked>
    <input type="checkbox">
    <input type="checkbox" checked="checked">
```

**Style属性**

`style`属性可以取一个字符串，但也可以取一个对象

```jade
	a(style={color: 'red', background: 'green'})
```

**Class属性**

类树形可以是一个字符串，但也可以是一个类名的数组。

```jade
- var classes = ['foo', 'bar', 'baz']
a(class=classes)
//- the class attribute may also be repeated to merge arrays
a.bing(class=classes class=['bing'])
```

值还可以是一个映射，类名映射到true或false。

```jade
- var currentUrl = '/about'
a(class={active: currentUrl === '/'} href='/') Home
a(class={active: currentUrl === '/about'} href='/about') About
```

**Class字面量**

Classes may be defined using a `.classname` syntax:

```jade
a.button
```

```html
	<a class="button"></a>
```

若标签名是div时，可以省略：

```jade
.content
```

```html
<div class="content"></div>
```

**ID字面量**

IDs may be defined using a `#idname` syntax:

```jade
a#main-link
```

```html
	<a id="main-link"></a>
```

此时，若标签名是div，也可以省略：

```jade
	#content
```

```html
	<div id="content"></div>
```

**&attributes**

The `&attributes` syntax can be used to explode an object into attributes of an element.

```jade
	div#foo(data-bar="foo")&attributes({'data-foo': 'bar'})
```

```html
	<div id="foo" data-bar="foo" data-foo="bar"></div>
```

The object does not have to be an object literal. It can also just be a variable that has an object as its value (see also Mixin Attributes)

```jade
- var attributes = {'data-foo': 'bar'};
div#foo(data-bar="foo")&attributes(attributes)
```

```html
	<div id="foo" data-bar="foo" data-foo="bar"></div>
```

> Attributes applied using `&attributes` are not automatically escaped. You must be sure to sanatize any user inputs to avoid Cross Site Scripting. This is done for you if you are passing in `attributes` from a mixin call.

### 代码

可以在模板中写内联JavaScript代码。有三种类型的代码。

**不输出的（Unbuffered）代码**

不输出的代码以`-`开头，不直接输出，例如：

```jade
    - for (var x = 0; x < 3; x++)
      li item
```

```html
    <li>item</li>
    <li>item</li>
    <li>item</li>
```

**输出的代码**

输出的代码以`=`开头。输出JavaScript表达式求值的结果。处于安全考虑，会转义HTML：

```jade
    p
      = 'This code is <escaped>!'
```

```html
    <p>This code is &lt;escaped&gt;!
    </p>
```

It can also be written inline with attributes, and supports the full range of JavaScript expressions:

```jade
	p= 'This code is' + ' <escaped>!'
```

**不转义的输出代码**

输出不转义的代码，以`!=`开头：

```jade
    p
      != 'This code is <strong>not</strong> escaped!'
```

```html
    <p>This code is <strong>not</strong> escaped!
    </p>
```

It can also be written inline with attributes, and supports the full range of JavaScript expressions:

```jade
	p!= 'This code is <strong>not</strong> escaped!'
```

### 条件

Jade's first-class conditional syntax allows for optional parenthesis, and you may now omit the leading `-` otherwise it's identical, still just regular javascript:

```jade
    - var user = { description: 'foo bar baz' }
    - var authorised = false
    #user
    if user.description
      h2 Description
      p.description= user.description
    else if authorised
      h2 Description
      p.description.
        User has no description,
        why not add one...
    else
      h1 Description
      p.description User has no description
```

Jade还支持`unless`：

```jade
    unless user.isAnonymous
      p You're logged in as #{user.name}
    if !user.isAnonymous
      p You're logged in as #{user.name}
```

### Case

The `case` statement is a shorthand for JavaScript's `switch` statement and takes the following form:

```jade
- var friends = 10
case friends
  when 0
    p you have no friends
  when 1
    p you have a friend
  default
    p you have #{friends} friends
```

You can use fall through just like in a select statement in JavaScript

```jade
- var friends = 0
case friends
  when 0
  when 1
    p you have very few friends
  default
    p you have #{friends} friends
```

Block expansion may also be used:

```jade
- var friends = 1
case friends
  when 0: p you have no friends
  when 1: p you have a friend
  default: p you have #{friends} friends
```

### extend：模板继承

The `extends` keyword allows a template to extend a layout or parent template. It can then override certain pre-defined blocks of content.

```jade
//- layout.jade
doctype html
html
  head
    block title
      title Default title
  body
    block content
```

```jade
//- index.jade
extends ./layout.jade

block title
  title Article Title

block content
  h1 My Article
```

```html
<!doctype html>
<html>
  <head>
    <title>Article Title</title>
  </head>
  <body>
    <h1>My Article</h1>
  </body>
</html>
```

> You can have multiple levels of inheritance, allowing you to create powerful hierarchies of templates.

### 过滤器

Filters let you use other languages within a jade template. They take a block of plain text as an input.

```jade
:markdown
  # Markdown

  I often like including markdown documents.
script
  :coffee
    console.log 'This is coffee script'
```

```html
    <h1>Markdown</h1>
    <p>I often like including markdown documents.</p>
    <script>console.log('This is coffee script')</script>
```

Filters are compile time. This makes them fast but means they cannot support dynamic content.

Built in filters are not available in the browser as they would not all work. Providing you compile your templates server side, they will still work fine though.

### 包含

包含用于将一个jade文件的内容插入另一个。

```jade
//- index.jade
doctype html
html
  include ./includes/head.jade
  body
    h1 My Site
    p Welcome to my super lame site.
    include ./includes/foot.jade
```

```jade
    //- includes/head.jade
    head
      title My Site
      script(src='/javascripts/jquery.js')
      script(src='/javascripts/app.js')
```

```jade
    //- includes/foot.jade
    #footer
      p Copyright (c) foobar
```

**包含纯文本**

Including files that are not jade just includes the raw text.

```jade
//- index.jade
doctype html
html
  head
    style
      include style.css
  body
    h1 My Site
    p Welcome to my super lame site.
    script
      include script.js
```

```css
/* style.css */
h1 { color: red; }
```

```js
// script.js
console.log('You are awesome');
```

**Including Filtered Text**

You can combine filters with includes to filter things as you include them.

```jade
//- index.jade
doctype html
html
  head
    title An Article
  body
    include:markdown article.md
```

### 模板继承

Jade supports template inheritance via the `block` and `extends` keywords. A block is simply a "block" of Jade that may be replaced within a child template, this process is recursive.

Jade blocks can provide default content if desired, however optional as shown below by `block scripts`, `block content`, and `block foot`.

layout.jade

```jade
html
  head
    title My Site - #{title}
    block scripts
      script(src='/jquery.js')
  body
    block content
    block foot
      #footer
        p some footer content
```

Now to extend the layout, simply create a new file and use the `extends` directive as shown below, giving the path (with or without the .jade extension). You may now define one or more blocks that will override the parent block content, note that here the foot block is not redefined and will output "some footer content".

page-a.jade

```jade
extends ./layout.jade

block scripts
  script(src='/jquery.js')
  script(src='/pets.js')

block content
  h1= title
  each pet in pets
    include pet
```

It's also possible to override a block to provide additional blocks, as shown in the following example where `content` now exposes a `sidebar` and `primary` block for overriding, or the child template could override `content` all together.

sub-layout.jade

```jade
extends ./layout.jade

block content
  .sidebar
    block sidebar
      p nothing
  .primary
    block primary
      p nothing
```

page-b.jade

```jade
extends ./sub-layout.jade

block content
  .sidebar
    block sidebar
      p nothing
  .primary
    block primary
      p nothing
```

**Block append / prepend**

Jade allows you to replace (default), prepend, or append blocks. Suppose for example you have default scripts in a "head" block that you wish to utilize on every page, you might do this:

```jade
html
  head
    block head
      script(src='/vendor/jquery.js')
      script(src='/vendor/caustic.js')
  body
    block content
```

Now suppose you have a page of your application for a JavaScript game, you want some game related scripts **as well as these defaults**, you can simply append the block:

```jade
extends layout

block append head
  script(src='/vendor/three.js')
  script(src='/game.js')
```

When using block append or block prepend the block is optional:

```jade
extends layout

append head
  script(src='/vendor/three.js')
  script(src='/game.js')
```
### 插值

**字符串插值，转义**

Consider the placement of the template locals title, author, and theGreat in the following template.

```jade
- var title = "On Dogs: Man's Best Friend";
- var author = "enlore";
- var theGreat = "<span>escape!</span>";

h1= title
p Written with love by #{author}
p This will be safe: #{theGreat}
```

插值内可以含有任何有效的Javascript表达式。

```jade
- var msg = "not my inside voice";
p This is #{msg.toUpperCase()}
```

**字符串插值，不转义**

可以输出不转义的值。

```jade
- var riskyBusiness = "<em>Some of the girls are wearing my mother's clothing.</em>";
.quote
  p Joel: !{riskyBusiness}
```

**Tag Interpolation**

If you take a look at this page's source on GitHub, you'll see several places where the tag interpolation operator is used, like so.

```jade
p.
  If you take a look at this page's source #[a(target="_blank", href="https://github.com/jadejs/jade/blob/master/docs/views/reference/interpolation.jade") on GitHub],
  you'll see several places where the tag interpolation operator is
  used, like so.
```

You could accomplish the same thing by writing an HTML tag inline with your Jade, but then, what's the point of writing the Jade? Wrap an inline Jade tag declaration in `#[` and `]` and it'll be evaluated and buffered into the content of it's containing tag.

### 遍历

Jade支持两种主要的迭代：`each`和`while`。

**each**

遍历数组：

```jade
ul
  each val in [1, 2, 3, 4, 5]
    li= val
```

获取数组下标：

```jade
ul
  each val, index in ['zero', 'one', 'two']
    li= index + ': ' + val
```

遍历对象

```jade
ul
  each val, index in {1:'one',2:'two',3:'three'}
    li= index + ': ' + val
```

迭代的对象或数组是普通的Javascript嗲吗，因此可以是变量、函数调用结果等等。

```jade
- var values = [];
ul
  each val in values.length ? values : ['There are no values']
    li= val
```

`each`可以用关键字`for`替代。

**while**

You can also use while to create a loop:

```jade
- var n = 0
ul
  while n < 4
    li= n++
```

### Mixins

Mixins allow you to create reusable blocks of jade.

```jade
//- Declaration
mixin list
  ul
    li foo
    li bar
    li baz
//- Use
+list
+list
```

它们会被编译为函数，可以带参数：

```jade
mixin pet(name)
  li.pet= name
ul
  +pet('cat')
  +pet('dog')
  +pet('pig')
```

**Mixin Blocks**

Mixins can also take a block of jade to act as the content:

```jade
mixin article(title)
  .article
    .article-wrapper
      h1= title
      if block
        block
      else
        p No content provided

+article('Hello world')

+article('Hello world')
  p This is my
  p Amazing article
```

    <div class="article">
      <div class="article-wrapper">
        <h1>Hello world</h1>
        <p>No content provided</p>
      </div>
    </div>
    <div class="article">
      <div class="article-wrapper">
        <h1>Hello world</h1>
        <p>This is my</p>
        <p>Amazing article</p>
      </div>
    </div>

**Mixin Attributes**

Mixins also get an implicit `attributes` argument taken from the attributes passed to the mixin:

```jade
mixin link(href, name)
  //- attributes == {class: "btn"}
  a(class!=attributes.class, href=href)= name

+link('/foo', 'foo')(class="btn")
```

> The values in attributes are already escaped so you should use != to avoid escaping them a second time (see also unescaped attributes).

You can also use with `&attributes` (see also &attributes)

```jade
mixin link(href, name)
  a(href=href)&attributes(attributes)= name

+link('/foo', 'foo')(class="btn")
```

**剩余参数**

You can write mixins that take an unknown number of arguments using the "rest arguments" syntax. e.g.

```jade
mixin list(id, ...items)
  ul(id=id)
    each item in items
      li= item

+list('my-list', 1, 2, 3, 4)
```

### 纯文本

Jade provides three common ways of getting plain text. They are useful in different situations

**Piped Text**

The simplest way of adding plain text to templates is to prefix the line with a `|` character (pronounced "pipe").

```jade
| Plain text can include <strong>html</strong>
p
  | It must always be on its own line
```

    Plain text can include <strong>html</strong>
    <p>It must always be on its own line</p>

**Inline in a Tag**

Since it's a common use case, you can put text in a tag just by adding it inline after a space.

```jade
	p Plain text can include <strong>html</strong>
```

	<p>Plain text can include <strong>html</strong></p>

**Block in a Tag**

Often you might want large blocks of text within a tag. A good example is with inline scripts or styles. To do this, just add a . after the tag (with no preceding space):

script.
  if (usingJade)
    console.log('you are awesome')
  else
    console.log('use jade')

<script>
  if (usingJade)
    console.log('you are awesome')
  else
    console.log('use jade')
</script>


## API文档

http://jade-lang.com/api/

This page details how to render jade using the JavaScript API in node.js

	npm install jade

### 所有API共有的选项

- `filename`：字符串。Used in exceptions, and required for relative includes and extends
- `doctype`：字符串。If the doctype is not specified as part of the template, you can specify it here. It is sometimes useful to get self-closing tags and remove mirroring of boolean attributes.
- `pretty`：布尔或字符串。Adds whitespace to the resulting html to make it easier for a human to read using `' '` as indentation. If a string is specified, that will be used as indentation instead (e.g. `'\t'`).
- `self`：布尔。Use a `self` namespace to hold the locals (false by default)
- `debug`：布尔。If set to true, the tokens and function body is logged to stdout
- `compileDebug`：布尔。Set this to false to disable debugging instrumentation (recommended in production). Set it to true to include the function source in the compiled template for better error messages (sometimes useful in development).
- `cache`：布尔。If set to true, compiled functions are cached. filename must be set as the cache key.
- `compiler`：类。Override the default compiler
- `parser`：类。Override the default parser
- `globals`：字符串数组。Add a list of global names to make accessible in templates

### jade.compile(source, options)

将一些jade源码编译为一个函数，函数可以被渲染多次，使用不同的locals。

- `source`：字符串。要编译的jade源码
- `options`：一个选项对象，见上一节。
- 返回值：函数。A function to generate the html from an object containing locals

```js
var jade = require('jade');
var fn = jade.compile('string of jade', options); // fn是一个函数
var html = fn(locals); // 渲染
// => '<string>of jade</string>'
```

### jade.compileFile(path, options)

将一个文件中的jade代码编译为一个函数，函数可以被渲染多次，使用不同的locals。

- `source`：jade文件路径。
- `options`：一个选项对象，见上一节。
- 返回值：函数。A function to generate the html from an object containing locals

```js
var jade = require('jade');
var fn = jade.compileFile('path to jade file', options);
var html = fn(locals);
// => '<string>of jade</string>'
```

### jade.compileClient(source, options)

将一些jade代码编译为JavaScript字符串，可以在客户端与jade运行时一起使用。

- `source`：字符串。jade源码。
- `options`：一个选项对象，见上一节。
- 返回：一个字符串

```js
var jade = require('jade');
var fn = jade.compileClient('string of jade', options);
var html = fn(locals);
// => 'function template(locals) { return "<string>of jade</string>"; }'
```

### jade.compileClientWithDependenciesTracked(source, options)

与`jade.compileClient`，只是该方法返回一个对象：

```js
{
  'body': 'function (locals) {...}',
  'dependencies': ['filename.jade']
}
```

You should only use this method if you need to implement something like watching for changes to the jade files.

### jade.compileFileClient(path, options)

编译jade模板文件到一个JavaScript字符串，可以在客户端与jade运行时一起使用。

- `path`：字符串。jade源码文件路径。
- `options`：一个选项对象，见上一节。
- `options.name`：字符串。用作客户端模板函数的函数名。
- 返回：字符串。一个JavaScript函数。

例子，模板文件：

```jade
h1 This is a Jade template
h2 By #{author}
```

编译成函数字符串：

```js
var fs   = require('fs');
var jade = require('jade');
// Compile the template to a function string
var jsFunctionString = jade.compileFileClient('/path/to/jadeFile.jade', {name: "fancyTemplateFun"});

// Maybe you want to compile all of your templates to a templates.js file and serve it to the client
fs.writeFileSync("templates.js", jsFunctionString);
```

下面是输出的函数字符串的样例（已写到templates.js）：

```js
function fancyTemplateFun(locals) {
  var buf = [];
  var jade_mixins = {};
  var jade_interp;

  var locals_for_with = (locals || {});

  (function (author) {
    buf.push("<h1>This is a Jade template</h1><h2>By "
      + (jade.escape((jade_interp = author) == null ? '' : jade_interp))
      + "</h2>");
  }.call(this, "author" in locals_for_with ?
    locals_for_with.author : typeof author !== "undefined" ?
      author : undefined)
  );

  return buf.join("");
}
```

在客户端添加Jade运行时（`node_modules/jade/runtime.js`）和刚才编译的模板。

```html
    <!DOCTYPE html>
    <html>
      <head>
        <script src="/runtime.js"></script>
        <script src="/templates.js"></script>
      </head>

      <body>
        <h1>This is one fancy template.</h1>

        <script type="text/javascript">
          var html = window.fancyTemplateFun({author: "enlore"});
          var div = document.createElement("div");
          div.innerHTML = html;
          document.body.appendChild(div);
        </script>
      </body>
    </html>
```

### jade.render(source, options)

- `source`：字符串。要渲染的jade源码。
- `options`：An options object (see above), also used as the locals object
- 返回：字符串。产生的HTML字符串。

```js
var jade = require('jade');
var html = jade.render('string of jade', options);
// => '<string>of jade</string>'
```

### jade.renderFile(filename, options)

- `filename`：字符串。jade文件路径。
- `options`：An options object (see above), also used as the locals object
- 返回：字符串。产生的HTML字符串。

```js
var jade = require('jade');
var html = jade.renderFile('path/to/file.jade', options);
// ...
```



