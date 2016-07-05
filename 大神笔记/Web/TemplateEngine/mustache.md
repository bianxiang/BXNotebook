[toc]

## 相关

https://github.com/twitter/hogan.js
https://github.com/jonnyreeves/jquery-Mustache
https://github.com/hanan198501/grunt-cptpl

## 通用版本

http://mustache.github.io/mustache.5.html

### 概要

A typical Mustache template:

    Hello {{name}}
    You have just won {{value}} dollars!
    {{#in_ca}}
    Well, {{taxed_value}} dollars, after taxes.
    {{/in_ca}}

Given the following hash:

    {
      "name": "Chris",
      "value": 10000,
      "taxed_value": 10000 - (10000 * 0.4),
      "in_ca": true
    }

Will produce the following:

    Hello Chris
    You have just won 10000 dollars!
    Well, 6000.0 dollars, after taxes.

"logic-less"的意思是，没有if语句、for循环，只有标签。

### 标签类型

Tags are indicated by the double mustaches. `{{person}}`和`{{#person}}`都是标签。In both examples, we'd refer to person as the key or tag key. Let's talk about the different types of tags.

#### 变量

最基础的标签类型是变量。`{{name}}`标签，尝试在当前上下文找`name`键。如果当前上下文找不到，找上一级上下文。如果最终还是找不到，什么也不输出。

所有的变量默认都会被HTML转码。If you want to return unescaped HTML, use the triple mustache: `{{{name}}}`. You can also use `&` to unescape a variable: `{{& name}}`. This may be useful when changing delimiters (see "Set Delimiter" below).

By default a variable "miss" returns an empty string. This can usually be configured in your Mustache library. The Ruby version of Mustache supports raising an exception in this situation.

JavaScript的点标记可以访问嵌套的对象：

    * {{name.first}} {{name.last}}
    * {{age}}

#### 节（Sections）

Sections render blocks of text one or more times, depending on the value of the key in the current context.

节有开始和结束标签，以`#`开始，`/`结束。如`{{#person}}`开始"person"节，`{{/person}}`结束。

**节的行为取决于键的值。**

##### 假值遇空列表

如果`person`键存在，但是一个假值或空列表，则节中的HTML不会被显示。

Javascript的表述：如`person`键不存在，或者是`null`、`undefined`、`false`、`0`或`NaN`、或是空字符串，空列表，则块不会被渲染。

    Shown.
    {{#person}}
      Never shown!
    {{/person}}

对象：

    {
      "person": false
    }

输出：

	Shown.

##### 非空列表

如果`person`键存在，值飞假值，则节中的HTML会被渲染。

如果值是非空列表，节中的内容会显示多次。节中内容的上下文会被设为循环的当前元素。

    {{#repo}}
      <b>{{name}}</b>
    {{/repo}}

Hash:

    {
      "repo": [
        { "name": "resque" },
        { "name": "hub" },
        { "name": "rip" }
      ]
    }

Output:

    <b>resque</b>
    <b>hub</b>
    <b>rip</b>

若要遍历字符串数组，`.`可以引用列表中的当前字符串：

    {
      "musketeers": ["Athos", "Aramis", "Porthos", "D'Artagnan"]
    }

Template:

    {{#musketeers}}
    * {{.}}
    {{/musketeers}}

Output:

    * Athos
    * Aramis
    * Porthos
    * D'Artagnan

如果节变量的值是一个函数，it will be called in the context of the current item in the list on each iteration.

    {
      "beatles": [
        { "firstName": "John", "lastName": "Lennon" },
        { "firstName": "Paul", "lastName": "McCartney" },
        { "firstName": "George", "lastName": "Harrison" },
        { "firstName": "Ringo", "lastName": "Starr" }
      ],
      "name": function () {
        return this.firstName + " " + this.lastName;
      }
    }

Template:

    {{#beatles}}
    * {{name}}
    {{/beatles}}

Output:

    * John Lennon
    * Paul McCartney
    * George Harrison
    * Ringo Starr

##### Lambdas

When the value is a callable object, such as a function or lambda, the object will be invoked and passed the block of text. The text passed is the literal block, unrendered. `{{tags}}` will not have been expanded - the lambda should do that on its own. In this way you can implement filters or caching.

Template:

    {{#wrapped}}
      {{name}} is awesome.
    {{/wrapped}}

Hash:

    {
      "name": "Willy",
      "wrapped": function() {
        return function(text, render) {
          return "<b>" + render(text) + "</b>"
        }
      }
    }

Output:

	<b>Willy is awesome.</b>

##### Non-False Values

When the value is non-false but not a list, it will be used as the context for a single rendering of the block.

Template:

    {{#person?}}
      Hi {{name}}!
    {{/person?}}

Hash:

    {
      "person?": { "name": "Jon" }
    }

Output:

	Hi Jon!

#### 反向（Inverted）节

反向节以`^`开头，`/`结束。That is `{{^person}}` begins a "person" inverted section while `{{/person}}` ends it.

While sections can be used to render text one or more times based on the value of the key, inverted sections may render text once based on the inverse value of the key. 即，在值不能存在、未空或为空列表的情况下才渲染。

Template:

    {{#repo}}
      <b>{{name}}</b>
    {{/repo}}
    {{^repo}}
      No repos :(
    {{/repo}}

Hash:

    {
      "repo": []
    }

Output:

	No repos :(

#### 注释

Comments begin with a bang and are ignored. The following template:

	<h1>Today{{! ignore me }}.</h1>

Will render as follows:

	<h1>Today.</h1>

Comments may contain newlines.

#### Partials

Partials以大于号开始，如`{{> box}}`。

Partials are rendered at runtime (as opposed to compile time), so recursive partials are possible. Just avoid infinite loops.

They also inherit the calling context. Whereas in an [ERB](http://en.wikipedia.org/wiki/ERuby) file you may have this:

	<%= partial :next_more, :start => start, :size => size %>

Mustache requires only this:

	{{> next_more}}

Why? Because the next_more.mustache file will inherit the size and start methods from the calling context.

In this way you may want to think of partials as includes, or template expansion, even though it's not literally true.

For example, this template and partial:

base.mustache:

    <h2>Names</h2>
    {{#names}}
      {{> user}}
    {{/names}}

user.mustache:

	<strong>{{name}}</strong>

Can be thought of as a single, expanded template:

    <h2>Names</h2>
    {{#names}}
      <strong>{{name}}</strong>
    {{/names}}

In mustache.js an object of partials may be passed as the third argument to `Mustache.render`. The object should be keyed by the name of the partial, and its value should be the partial text.

    Mustache.render(template, view, {
      user: userTemplate
    });

#### Set Delimiter

Set Delimiter tags start with an equal sign and change the tag delimiters from `{{` and `}}` to custom strings.

Consider the following contrived example:

    * {{default_tags}}
    {{=<% %>=}}
    * <% erb_style_tags %>
    <%={{ }}=%>
    * {{ default_tags_again }}

Here we have a list with three items. The first item uses the default tag style, the second uses erb style as defined by the Set Delimiter tag, and the third returns to the default style after yet another Set Delimiter declaration.

According to [ctemplates](http://google-ctemplate.googlecode.com/svn/trunk/doc/howto.html), this "is useful for languages like TeX, where double-braces may occur in the text and are awkward to use for markup."

Custom delimiters may not contain whitespace or the equals sign.

## Javascript版本

https://github.com/janl/mustache.js

### 使用

Below is quick example how to use mustache.js:

    var view = {
      title: "Joe",
      calc: function () {
        return 2 + 4;
      }
    };

	var output = Mustache.render("{{title}} spends {{calc}}", view);

In this example, the Mustache.render function takes two parameters: 1) the mustache template and 2) a view object that contains the data and code needed to render the template.

### 模板

There are several techniques that can be used to load templates and hand them to mustache.js, here are two of them:

#### 包含

Here's a small example using jQuery:

    <html>
    <body onload="loadUser">
    	<div id="target">Loading...</div>
    	<script id="template" type="x-tmpl-mustache">
    		Hello {{ name }}!
    	</script>
    </body>
    </html>

	function loadUser() {
      var template = $('#template').html();
      Mustache.parse(template);   // optional, speeds up future uses
      var rendered = Mustache.render(template, {name: "Luke"});
      $('#target').html(rendered);
    }

#### 加载外部模板

If your templates reside in individual files, you can load them asynchronously and render them when they arrive. Another example using jQuery:

    function loadUser() {
      $.get('template.mst', function(template) {
        var rendered = Mustache.render(template, {name: "Luke"});
        $('#target').html(rendered);
      });
    }

### 节与函数

If the value of a section **key** is a function, it is called with the section's literal block of text, un-rendered, as its first argument. The second argument is a special rendering function that uses the current view as its view argument. It is called in the context of the current view object.

    {
      "name": "Tater",
      "bold": function () {
        return function (text, render) {
          return "<b>" + render(text) + "</b>";
        }
      }
    }

Template:

	{{#bold}}Hi {{name}}.{{/bold}}

Output:

	<b>Hi Tater.</b>

#### Pre-parsing and Caching Templates

By default, when mustache.js first parses a template it keeps the full parsed token tree in a cache. The next time it sees that same template it skips the parsing step and renders the template much more quickly. If you'd like, you can do this ahead of time using mustache.parse.

	Mustache.parse(template);

    // Then, sometime later.
    Mustache.render(template, view);




