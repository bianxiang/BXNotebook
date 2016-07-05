[toc]

{{
文档写的一般般，不好懂。
}}

## Guide

It is particularly well-suited for asynchronous and streaming applications.

A pure JavaScript library, Dust is runs in both browser-side and server-side environments. Dust templates are compiled and then loaded where they are needed along with the runtime library. 库对模板的加载方式不做限定。

### 安装

Node.js环境：

	npm install dust

	var dust = require('dust');

浏览器。引入库和编译好的模板：

    <script src="dust-core-0.3.0.min.js"></script>
    <script src="compiled_templates.js"></script>

如果要在浏览器中编译，需要引入完整的库：

	<script src="dust-full-0.3.0.min.js"></script>

Precompilation is the recommended approach for general use.

### 编译模板

利用`dust.compile`将模板便宜为一串Javascript源代码的字符串：

	var compiled = dust.compile("Hello {name}!", "intro");

结果：

	'(function(){dust.register("intro",body_0) ...'

If you save this source to a file and include the file in your HTML script tags, the compiled template will automatically register itself with the local runtime, under the name "intro". To evaluate a compiled template string manually, use `dust.loadSource`:

	dust.loadSource(compiled);

The template is now available within the `dust.cache` object.

### 渲染模板

渲染引擎提供回调和流两种接口。

#### 回调接口

To render a template, call `dust.render` with the template name, a context object and a callback function:

    dust.render("intro", {name: "Fred"}, function(err, out) {
      console.log(out);
    });

The code above will write the following to the console:

	Hello Fred!

#### 流接口

Templates may also be streamed. `dust.stream` returns a handler very similar to a Node EventEmitter:

    dust.stream("index", context)
        .on("data", function(data) {
          console.log(data);
        })
        .on("end", function() {
          console.log("I'm finished!");
        })
        .on("error", function(err) {
          console.log("Something terrible happened!");
        });

When used with specially crafted context handlers, the streaming interface provides chunked template rendering.


### 语法

Dust模板使用两类标签：keys and sections. Keys reference fields within the current view context. Sections accept template blocks that may be enumerated, filtered or transformed in various ways.

### Keys

要在模板中引用视图上下文的一个键，在键周围添加大括号。例如，假如有模板：

	Hello {name}!

和视图上下文：

	{ name: "Fred" }

结果僵尸：

	Hello Fred!

如果在上下文中找不到键，Dust输出空字符串。

Dust会吧一切值转换未字符串。如果Dust遇到一个handler函数，会调用函数，传入模板的当前状态。

#### 过滤器

默认键的内容会被HTML转义。例如，假如某个键的内容是：

	<script>alert('I am evil!')</script>

将被渲染未：

	&lt;script&gt;alert('I am evil!')&lt;/script&gt;

如果要禁用自动转义，在键后追加一个`'|'`和`'s'`。

	Hello {name|s}

There are several other built-in filters: `h` forces HTML escaping, `j` escapes JavaScript strings, `u` proxies to JavaScript's built-in `encodeURI`, and `uc` proxies to JavaScript's `encodeURIComponent`. 过滤器可被级联使用：

	Hello {name|s|h|u}

过滤器不支持传参。；if you need more sophisticated behavior use a **section** tag instead.

### Sections

假如上下文有一个`friends`字段，是一个数组，元素有`name`和`age`两个字段。用section标签：

    {#friends}
      {name}, {age}{~n}
    {/friends}

section从`{#friends}`开始，结束于`{/friends}`。

    {
      friends: [
        { name: "Moe", age: 37 },
        { name: "Larry", age: 39 },
        { name: "Curly", age: 35 }
      ]
    }

的输入是：

    Moe, 37
    Larry, 39
    Curly, 35

When `friends` resolves to a value or object instead of an array, Dust sets the current context to the value and renders the block one time. If `friends` resolves to a custom handler function, the function is given control over the section's behavior.

若`friends`值为空或不存在，Dust什么也不输出。此时，如有`{:else}`，则使用后续替代的模板：

    {#friends}
      {name}, {age}{~n}
    {:else}
      You have no friends!
    {/friends}

Internally, Dust builds **a stack of contexts** as templates delve deeper into nested sections. 如果在当前上下文找不到指定键，会在上级上下文，上上级……寻找。

Self-closing section tags are allowed, so the template code below is permissible (although in this case it won't render anything):

    {#friends/}

#### Paths

Paths用于相对当前上下文引用键。

`.`引用当前上下文自身。

	{#names}{.} {/names}

上下文：
	{ names: ["Moe", "Larry", "Curly"] }

模板输出：

	Moe Larry Curly 

Paths还可以用于访问嵌套的上下文：

	{foo.bar}

或者强制搜索当前section的上下文｛｛不搜索父上下文｝｝：

	{.foo}

To avoid brittle and confusing references, paths never backtrack up the context stack. If you need to drill into a key available within the parent context, pass the key as a parameter.

#### 内联参数

内联参数出现在section的起始标签。参数之间用单个空格分隔。By default, inline parameters merge values into the section's view context:

    {#profile bar="baz" bing="bong"}
      {name}, {bar}, {bing}
    {/profile}

内联参数可以用于给雨子上下文冲突的键一个别名：

    {name}{~n}
    {#profile root_name=name}
      {name}, {root_name}
    {/profile}

注意等号后面是一个键，不是字符串。

实参还可以是插值的字符串：

	{#snippet id="{name}_id"/}

#### Body Parameters

Unlike inline parameters, which modify the context, body parameters pass **named template blocks** to handler functions. Body parameters are useful for implementing striping or other complex behaviors that might otherwise involve manually assembling strings within your view functions. The only body parameter with default behavior is the `{:else}` tag as seen above.

#### 上下文

Normally, upon encountering a section tag Dust merges the section's context with the parent context. 有时想要手工设置节的上下文。Sections accept a context argument for this purpose:

	{#list:projects}{name}{/list}

Here, we're providing an array of `projects` to the list section, which might be a special helper defined on the view. 如果`list`不是一个函数，是一个值，它的上下文仍会作为`projects`的父上下文。

#### 特殊的Sections

除了标准的`#`节标签，Dust提供几个特殊含义的节标签。`?`是存在标签，`^`是不存在标签，`@`是上下文helpers标签。These tags make it easier to work with plain JSON data without additional helpers.

The exists and notexists sections check for the existence (in the falsy sense) of a key within the current context. They do not alter the current context, making it possible to, for instance, check that an array is non-empty before wrapping it in HTML list tags:

    {?tags}
      <ul>
        {#tags}
          <li>{.}</li>
        {/tags}
      </ul>
    {:else}
      No Tags!
    {/tags}

Unlike regular sections, conditional sections do not evaluate functions defined on the view. In those cases you'll still have to write your own handlers.

The context helpers tag provides a couple of convenience functions to support iteration. The **sep** tag prints the enclosed block for every value except for the last. The **idx** tag passes the numerical index of the current element to the enclosed block.

	{#names}{.}{@idx}{.}{/idx}{@sep}, {/sep}{/names}

The template above might output something like:

	Moe0, Larry1, Curly2

### Partials

Partials, also known as **template includes**, allow you to compose templates at runtime.

	{>profile/}

The block above looks for a template named "profile" and inserts its output into the parent template. Like sections, partials inherit the current context. And like sections, partials accept a context argument:

	{>profile:user/}

Partial tags also accept string literals and interpolated string literals as keys:

	{>"path/to/comments.dust.html"/}
	{>"posts/{type}.dust.html"/}

This is useful when you're retrieving templates from the filesystem and the template names wouldn't otherwise be valid identifiers, or when selecting templates dynamically based on information from the view context.

### Blocks and Inline Partials

Often you'll want to have a template inherit the bulk of its content from a common base template. Dust solves this problem via blocks and inline partials. When placed within a template, blocks allow you to define snippets of template code that may be overriden by any templates that reference this template:

    Start{~n}
    {+title}
      Base Title
    {/title}
    {~n}
    {+main}
      Base Content
    {/main}
    {~n}
    End

Notice the special syntax for blocks: `{+block} ... {/block}`. When this template is rendered standalone, Dust simply renders the content within the blocks:

    Start
    Base Title
    Base Content
    End

But when the template is invoked from another template that contains inline partial tags (`{<snippet} ... {/snippet}`):

    {>base_template/}
    {<title}
      Child Title
    {/title}
    {<main}
      Child Content
    {/main}

Dust overrides the block contents of the base template:

    Start
    Child Title
    Child Content
    End

A block may be self-closing (`{+block/}`), in which case it is not displayed unless a calling template overrides the content of the block. Inline partials never output content themselves, and are always global to the template in which they are defined, so the order of their definition has no significance. They are passed to all templates invoked by the template in which they are defined.

Note that blocks can be used to render inline partials that are defined within the same template. This is useful when you want to use the same template to serve AJAX requests and regular requests:

    {^xhr}
      {>base_template/}
    {:else}
      {+main/}
    {/xhr}
    {<title}
      Child Title
    {/title}
    {<main}
      Child Content
    {/main}

### 静态文本

The Dust parser is finely tuned to minimize the amount of escaping that needs to be done within static text. Any text that does not closely resemble a Dust tag is considered static and will be passed through untouched to the template's output. This makes Dust suitable for use in templating many different formats. In order to be recognized as such, Dust tags should not contain extraneous whitespace and newlines.

#### 特殊字符

Depending on whitespace and delimeter settings, it is sometimes necessary to include escape tags within static text. Escape tags begin with a tilde (`~`), followed by a key identifying the desired escape sequence. Currently newline (n), carriage return (r), space (s), left brace (lb) and right brace (rb) are supported. For example:

	Hello World!{~n}

Inserts a newline after the text. Dust默认会压缩空白符，包括换行和缩进。This behavior can be toggled at compile time.

#### 注释

Comments, which do not appear in template output, begin and end with a bang (!):

    {!
      Multiline
      {#foo}{bar}{/foo}
    !}
    {!before!}Hello{!after!}

The template above would render as follows:

Hello

### 上下文

上下文是一个特殊的对象，控制变量的查找和模板的行为。The context can be visualized as a stack of objects that grows as we descend into **nested sections**:

    global     --> { helper: function() { ... }, ... }
    root       --> { profile: { ... }, ... }
    profile    --> { friends: [ ... ], ... }
    friends[0] --> { name: "Jorge", ... }

When looking up a key, Dust searches the context stack from the bottom up. There is no need to merge helper functions into the template data; instead, create a base context onto which you can push your local template data:

    // Set up a base context with global helpers
    var base = dust.makeBase({
      sayHello: function() { return "Hello!" }
    });

    // Push to the base context at render time
    dust.render("index", base.push({foo: "bar"}), function(err, out) {
      console.log(out);
    });

Dust does not care how your reference objects are built. You may, for example, push prototyped objects onto the stack. The system leaves the **this** keyword intact when calling handler functions on your objects.

### Handlers

When Dust encounters a function in the context, it calls the function, passing in arguments that reflect the current state of the template. In the simplest case, a handler can pass a value back to the template engine:

    {
      name: function() {
        return "Bob";
      }
    }

#### Chunks

But handlers can do much more than return values: they have complete control over the flow of the template, using the same API Dust uses internally. For example, the handler below writes a string directly to the current template chunk:

    {
      name: function(chunk) {
        return chunk.write("Bob");
      }
    }

A Chunk is a Dust primitive for controlling the flow of the template. Depending upon the behaviors defined in the context, templates may output one or more chunks during rendering. A handler that writes to a chunk directly must return the modified chunk.

#### 访问上下文

Handlers have access to the context object:

    {
      wrap: function(chunk, context) {
        return chunk.write(context.get("foo"));
      }
    }

`context.get("foo")` searches for `foo` within the context stack. `context.current()` retrieves the value most recently pushed onto the context stack.

#### Accessing Body Parameters

The `bodies` object provides access to any bodies defined within the calling block.

	{#guide}foo{:else}bar{/guide}

The template above will either render "foo" or "bar" depending on the behavior of the handler below:

    {
      guide: function(chunk, context, bodies) {
        if (secret === 42) {
          return chunk.render(bodies.block, context);
        } else {
          return chunk.render(bodies['else'], context);
        }
      }
    }

`bodies.block` is a special parameter that returns the default (unnamed) block. `chunk.render` renders the chosen block.

#### 访问内联参数

The params object contains any inline parameters passed to a section tag:

    {
      hello: function(chunk, context, bodies, params) {
        if (params.greet === "true") {
          return chunk.write("Hello!");
        }
        return chunk;
      }
    }

#### Asynchronous Handlers

You may define handlers that execute asynchronously and in parallel:

    {
      type: function(chunk) {
        return chunk.map(function(chunk) {
          setTimeout(function() {
            chunk.end("Async");
          });
        });
      }
    }

`chunk.map` tells Dust to manufacture a new chunk, reserving a slot in the output stream before continuing on to render the rest of the template. You must (eventually) call `chunk.end()` on a mapped chunk to weave its content back into the stream.

`chunk.map` provides a convenient way to split up templates rendered via dust.stream. For example, you might wrap the head of an HTML document in a special `{#head} ... {/head}` tag that is flushed to the browser before the rest of the body has finished rendering.

### （未）Reference


