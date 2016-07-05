[toc]

加载器列表：
http://webpack.github.io/docs/list-of-loaders.html

插件列表：
http://webpack.github.io/docs/list-of-plugins.html

## 动机

http://webpack.github.io/docs/motivation.html

网站朝着 Web APP 进化。

### 各种模块系统

- `<script>`-tag style (without a module system)
- CommonJs
- AMD and some dialects of it
- ES6 modules

#### CommonJs：同步require

This style uses a **synchronous** require method to load a dependency and return an exported interface.

```js
require("module");
require("../file.js");
exports.doStuff = function() {};
module.exports = someValue;
```

node.js使用这种方式。

优点：

- Server-side modules can be reused
- There are already many modules in this style (npm)
- very simple and easy to use.

缺点：

- blocking calls do not apply well on networks. Network requests are asynchronous.
- No parallel require of multiple modules

实现：

- node.js - server-side
- browserify
- modules-webmake - compile to one bundle
- wreq - client-side

#### AMD：异步require

Asynchronous Module Definition

Other module systems (for the browser) had problems with the synchronous require (CommonJs) and introduced an asynchronous version (and a way to define modules and exporting values):

```js
require(["module", "../file"], function(module, file) { /* ... */ });
define("mymodule", ["dep1", "dep2"], function(d1, d2) {
  return someExportedValue;
});
```

优点：

- Fits to the asynchronous request style in networks.
- Parallel loading of multiple modules.

缺点：

- Coding overhead. More difficult to read and write.
- Seems to be some kind of workaround.

实现：

- require.js - client-side
- curl - client-side

Read more about [CommonJs](http://webpack.github.io/docs/commonjs.html) and [AMD](http://webpack.github.io/docs/amd.html).

#### ES6模块

EcmaScript6 adds some language constructs to JavaScript, which form another module system.

```js
import "jquery";
export function doStuff() {}
module "localModule" {}
```

优点：

- Static analysis is easy
- Future-proof as ES standard

缺点：

- Native browser support will take time
- Very few modules in this style

### 传输

传输模块有两个极端：一个模块一个请求；一个请求传所有模块。

**分块传输（Chunked transferring）**

While compiling all modules: Split the set of modules into multiple smaller batches (chunks).

Chunks with modules that are not required initially are only requested on demand. The initial request doesn’t contain your complete code base and is smaller.

分割点（split points）取决于开发者，可选（optional）。

A big code base is possible!

Note: The idea is from Google’s GWT.

Read more about [Code Splitting](http://webpack.github.io/docs/code-splitting.html).

### 其他资源

图片、CSS。

还有方言，如CoffeeScript -> Javascript。Webpack想接收编译。干Grunt、Gulp的事。

```js
require("./style.css");
require("./style.less");
require("./template.jade");
require("./image.png");
```

Read more about [Using loaders](http://webpack.github.io/docs/using-loaders.html) and [Loaders](http://webpack.github.io/docs/loaders.html).

## webpack是什么

http://webpack.github.io/docs/what-is-webpack.html

webpack is a **module bundler**.

webpack takes modules with dependencies and generates static assets representing those modules.

目标：

- Split the dependency tree into **chunks loaded on demand**
- Keep initial loading time low
- Every static asset should be able to be a module
- Ability to integrate 3rd-party libraries as modules
- Ability to customize nearly every part of the module bundler
- Suited for big projects

How is webpack different?

**Code Splitting**

webpack has two types of dependencies in its dependency tree: **sync** and **async**. Async dependencies act as **split points** and form **a new chunk**. After the chunk tree is optimized, **a file is emitted for each chunk**.

Read more about [Code Splitting](http://webpack.github.io/docs/code-splitting.html).

**Loaders**

webpack can only process JavaScript natively, but loaders are used to transform other resources into JavaScript. By doing so, every resource forms a module.

Read more about [Using loaders](http://webpack.github.io/docs/using-loaders.html) and [Loaders](http://webpack.github.io/docs/loaders.html).

**Clever parsing**

webpack has a clever parser that can process nearly every 3rd party library. It even allows expressions in dependencies like so `require("./templates/" + name + ".jade")`. It handles the most common module styles: CommonJs and AMD.

Read more about [expressions in dependencies](http://webpack.github.io/docs/context.html), [CommonJs](http://webpack.github.io/docs/commonjs.html) and [AMD](http://webpack.github.io/docs/amd.html).

**Plugin system**

webpack features a rich plugin system. Most internal features are based on this plugin system. This allows you to customize webpack for your needs and distribute common plugins as open source.

Read more about [Plugins](http://webpack.github.io/docs/plugins.html).

## 安装

http://webpack.github.io/docs/installation.html

	$ npm install webpack -g

下面可以使用`webpack`命令。

## 使用

http://webpack.github.io/docs/usage.html

see CLI for the command line interface.
see node.js API for the node.js interface.
see Configuration for the configuration options.

## 使用加载器

http://webpack.github.io/docs/using-loaders.html

加载器是一种转换（transformations），作用于资源文件。它们是node.js函数，讲源文件转换为目标文件。

例如，用加载器可以加载 CoffeeScript 或 JSX。

加载器功能：

- 加载器可以形成链。They are applied in a pipeline to the resource. 最后一个要反回JavaScript，其他加载器返回值任意。
- 加载器可以是同步的或异步的
- Loaders run in node.js and can do everything that’s possible there.
- 加载器可以接收查询参数。This can be used to pass configuration to the loader.
- Loaders can be bound to extension / RegExps in the configuration.
- Loaders can be published / installed through npm.
- Normal modules can export a loader in addition to the normal main via package.json loader.
- Loaders can access the configuration.
- Plugins can give loaders more features.
- Loaders can emit additional arbitrary files.

If you are interested in some loader examples head of to the [list of loaders](http://webpack.github.io/docs/list-of-loaders.html).

### RESOLVING LOADERS

Loaders are resolved similar to modules. A loader module is expected to export a function and to be written in node.js compatible JavaScript. In the common case you manage loaders with npm, but you can also have loaders as files in your app.

**Referencing loaders**

By convention, though not required, loaders are usually named as `XXX-loader`, where XXX is the context name. For example, `json-loader`.

You may reference loaders by its full (actual) name (e.g. `json-loader`), or by its shorthand name (e.g. json).

The loader name convention and precedence search order is defined by `resolveLoader.moduleTemplates` within the webpack configuration API.

Loader name conventions may be useful, especially when referencing them within `require()` statements; see usage below.

**Installing loaders**

If the loader is available on npm you can install the loader via:

	$ npm install xxx-loader --save
or

	$ npm install xxx-loader --save-dev

### 使用

There are multiple ways to use loaders in your app:

- explicit in the require statement
- configured via configuration
- configured via CLI

#### 在require中使用加载器

> 注意，如果可能尽量不要使用该方法，如果你想让你的脚本适用多种环境（node.js和浏览器）。推荐使用配置方法（下一节介绍）。

It’s possible to specify the loaders in the `require` statement (or `define`, `require.ensure`, etc.). Just separate loaders from resource with `!`. Each part is resolved relative to the current directory.

```js
require("./loader!./dir/file.txt");
// => 使用当前目录的 loader.js 文件，转换 file.txt 文件

require("jade!./template.jade");
// 使用 jade-loader（在node_modules中）转换 template.jade

require("!style!css!less!bootstrap/less/bootstrap.less");
// => bootstrap是一个模块，位于node_modules。less文件先被 less-loader 处理。接着被 css-loader 和 style-loader 处理
```

#### 配置

通过配置，将加载器绑定到一个正则表达式

```js
{
    module: {
        loaders: [
            { test: /\.jade$/, loader: "jade" },
            // => "jade" loader is used for ".jade" files

            { test: /\.css$/, loader: "style!css" },
            // => "style" and "css" loader is used for ".css" files
            // 等价写法：
            { test: /\.css$/, loaders: ["style", "css"] },
        ]
    }
}
```

#### CLI

You can bind loaders to an extension via CLI:

	$ webpack --module-bind jade --module-bind 'css=style!css'

This uses the loader “jade” for “.jade” files and the loaders “style” and “css” for “.css” files.

#### 查询参数

通过查询字符串可以向加载器传递参数。例如`url-loader?mimetype=image/png`。

注意，具体格式取决于加载器。Most loaders accept parameters in the normal query format (`?key=value&key2=value2`) and as JSON object (`?{"key":"value","key2":"value2"}`).

**in require**

```js
	require("url-loader?mimetype=image/png!./file.png");
```

**Configuration**

```js
	{ test: /\.png$/, loader: "url-loader?mimetype=image/png" }
```

or

```js
    {
        test: /\.png$/,
        loader: "url-loader",
        query: { mimetype: "image/png" }
    }
```

**CLI**

	webpack --module-bind "png=url-loader?mimetype=image/png"

## 使用插件

http://webpack.github.io/docs/using-plugins.html

### 内建插件

Plugins are included in your module by using the `plugins` property in the webpack config.

```js
// webpack should be in the node_modules directory, install if not.
var webpack = require("webpack");

module.exports = {
    plugins: [
        new webpack.ResolverPlugin([
            new webpack.ResolverPlugin.DirectoryDescriptionFilePlugin("bower.json", ["main"])
        ], ["normal", "loader"])
    ]
};
```

### 其他插件

Plugins that are not built-in may be installed via npm if published there, or by other means if not:

	npm install component-webpack-plugin

which can then be used as follows:

```js
var ComponentPlugin = require("component-webpack-plugin");
module.exports = {
    plugins: [
        new ComponentPlugin()
    ]
}
```

## 疑难解答

http://webpack.github.io/docs/troubleshooting.html

### RESOLVING

**general resolving issues**

- `--display-error-details` give you more details.
- Read [Configuration](http://webpack.github.io/docs/configuration.html) regarding resolving starting at resolve
- loaders have their own resolving configuration resolveLoader

**npm linked modules doesn’t find their dependencies**

The node.js module resolving algorithm is pretty simple: module dependencies are looked up in node_modules folders in every parent directory of the requiring module. When you npm link modules with peer dependencies that are not in your root directory, modules can no longer be found. (You probably want to consider peerDependencies with npm link as broken by design in node.js.) Note that a dependency to the application (even if this is not the perfect design) is also a kind of peerDependency even if it’s not listed as such in the module’s package.json.

But you can easily workaround that in webpack: Add the node_modules folder of your application to the resolve paths. There are two config options for this: resolve.fallback and resolveLoader.fallback.

Here is a config example:

```js
module.exports = {
  resolve: { fallback: path.join(__dirname, "node_modules") },
  resolveLoader: { fallback: path.join(__dirname, "node_modules") }
};
```

### WATCHING

**webpack doesn’t recompile on change while watching**

**Not enough watchers**

Verify that if you have enough available watchers in your system. If this value is too low, the file watcher in Webpack won’t recognize the changes:

	cat /proc/sys/fs/inotify/max_user_watches

Arch users, add fs.inotify.max_user_watches=524288 to /etc/sysctl.d/99-sysctl.conf and then execute sysctl --system. Ubuntu users (and possibly others): echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p.

**OS-X fsevents bug**

On OS-X folders can get corrupted. See this article:

[OS X FSEvents bug may prevent monitoring of certain folders](http://feedback.livereload.com/knowledgebase/articles/86239-os-x-fsevents-bug-may-prevent-monitoring-of-certai)

**Windows paths**

webpack expects absolute paths for many config options. `__dirname + "/app/folder"` is wrong, because windows uses \ as path separator. This breaks some stuff.

Use the correct separators. I.e. `path.resolve(__dirname, "app/folder")` or `path.join(__dirname, "app", "folder")`.

## CommonJs

http://webpack.github.io/docs/commonjs.html

To achieve this CommonJS give you two tools:

the `require()` function, which allows to import a given module into the current scope.
the `module` object, which allows to export something from the current scope.

例子：

```js
// salute.js
var MySalute = "Hello";
module.exports = MySalute;
// world.js
var MySalute = require("./salute");
var Result = MySalute + "world!";
module.exports = Result;
```

Note that we didn’t use the full filename salute.js but `./salute` when calling require, so you can omit the extension of your scripts. `./` means that the salute module is in the same directory as the `world` module.

## AMD

http://webpack.github.io/docs/amd.html

AMD (Asynchronous Module Definition) was the response to those who thought the CommonJS Module system was not ready for the browser because its nature was synchronous.

AMD specifies a standard for modular JavaScript such that modules can load their dependencies asynchronously, solving the problems associated with synchronous loading.

模块使用 `define` 函数定义。

```
define(id?: String, dependencies?: String[], factory: Function|Object);
```

`id`指定模块名。可选。
`dependencies`指定该模块依赖的其他模块。它是一个数组，包含模块的标示符。It is optional, but if omitted, it defaults to `[“require”, “exports”, “module”]`.
`factory`可以是函数或对象，是模块的实际定义。If the factory is a function, the value returned will be the exported value for the module.

例子：

```js
define('myModule', ['jquery'], function($) {
    // $ is the export of the jquery module.
    $('body').text('hello world');
});
// and use it
define(['myModule'], function(myModule) {});
```

```js
// 多个依赖
define(['jquery', './math.js'], function($, math) {
    // $ and math are the exports of the jquery module.
    $('body').text('hello world');
});
```

```js
// 导出自己
define(['jquery'], function($) {

    var HelloWorldize = function(selector){
        $(selector).text('hello world');
    };

    return HelloWorldize;
});
```

```js
// Using require to load dependencies
define(function(require) {
    var $ = require('jquery');
    $('body').text('hello world');
});
```

## （未）WEBPACK FOR BROWSERIFY USERS

http://webpack.github.io/docs/webpack-for-browserify-users.html

## 代码分割

http://webpack.github.io/docs/code-splitting.html

对于大型Web APP，将所有代码放入一个文件效率不高，特别是部分代码在少数情况下才会用到。webpack能够将其的代码分成大块（chunks），按需加载。这些功能称为代码分割（code splitting）。

你可以在代码中定义分割点。webpack takes care of the dependencies, output files and runtime stuff.

代码分割，不仅仅是把通用代码放入共享的大块（chunk）中。代码分割的更关键功效是能够让代码按需加载。

### 定义一个分割点

AMD和CommonJs按需加载代码的方式不同。Webpack都支持。是它们，形成分割点：

**require.ensure (CommonJs)**

`require.ensure(dependencies, callback)`

The require.ensure method ensures that every dependency in dependencies can be synchronously required when calling the callback. callback is called with the `require` function as parameter.

Example:

```js
require.ensure(["module-a", "module-b"], function(require) {
    var a = require("module-a");
    // ...
});
```

Note: `require.ensure` only loads the modules, it doesn’t evaluate them.

**require (AMD)**

The AMD spec defines an asynchronous `require` method with this definition:
`require(dependencies, callback)`

When called, all dependencies are loaded and the callback is called with the exports of the loaded dependencies.

Example:

```js
require(["module-a", "module-b"], function(a, b) {
    // ...
});
```

Note: AMD require loads and **evaluate** the modules. In webpack modules are evaluated left to right.

Note: It’s allowed to omit the callback.

### Chunk的内容

一个分割点上的所有依赖到进入一个新的大块。依赖会被递归加入。

If you pass a function expression (or bound function expression) as callback to the split point, webpack自动将函数内require的所有依赖放入大块。

### Chunk优化

若两个大块包含的模块相同，它们会合并成一个。This can cause chunks to have multiple parents.

If a module is available in all parents of a chunk, it’s removed from that chunk.

If a chunk contains all modules of another chunk, this is stored. It fulfills multiple chunks.

### Chunk的加载

根据配置选项`target`，一段chunk加载的逻辑被添加到bundle。对于`web`目标，chunks通过**jsonp**架子。A chunk is only loaded once and parallel requests are merged into one. The runtime checks for loaded chunks whether they fulfill multiple chunks.

### Chunk的类型

**Entry chunk**：An entry chunk contains the runtime plus a bunch of modules. If the chunk contains the **module 0** the runtime executes it. If not, it waits for chunks that contains the **module 0** and executes it (every time when there is a chunk with a **module 0**).

**Normal chunk**：普通大块不包含运行时。It only contains a bunch of modules. The structure depends on the chunk loading algorithm. I.e. for jsonp the modules are wrapped in a jsonp callback function. The chunk also contains a list of chunk id that it fulfills.

**Initial chunk (non-entry)**：An initial chunk is a normal chunk. The only difference is that optimization treats it as more important because it counts toward the initial loading time (like entry chunks). That chunk type can occur in combination with the `CommonsChunkPlugin`.

### 分离应用和厂商的代码

To split your app into 2 files, say app.js and vendor.js, you can require the vendor files in vendor.js. Then pass this name to the `CommonsChunkPlugin` as shown below.

```js
var webpack = require("webpack");

module.exports = {
  entry: {
    app: "./app.js",
    vendor: ["jquery", "underscore", ...],
  },
  output: {
    filename: "bundle.js"
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin(/* chunkName= */"vendor", /* filename= */"vendor.bundle.js")
  ]
};
```

This will remove all modules in the vendor chunk from the app chunk. The bundle.js will now contain just your app code, without any of it’s dependencies. These are in `vendor.bundle.js`.

In your HTML page load `vendor.bundle.js` before bundle.js.

```
    <script src="vendor.bundle.js"></script>
    <script src="bundle.js"></script>
```

### Multiple entry chunks

It’s possible to configure multiple entry points that will result in multiple entry chunks. The entry chunk contains the runtime，一个页面只能有一个运行时（有例外）。

With the `CommonsChunkPlugin` the runtime is moved to the commons chunk. The entry points are now in initial chunks. While only one entry chunk can be loaded, multiple initial chunks can be loaded. This exposes the possibility to run multiple entry points in a single page.

Example:

```
    var webpack = require("webpack");
    {
        entry: { a: "./a", b: "./b" },
        output: { filename: "[name].js" },
        plugins: [ new webpack.optimize.CommonsChunkPlugin("init.js") ]
    }
    <script src="init.js"></script>
    <script src="a.js"></script>
    <script src="b.js"></script>
```

### Commons chunk

The `CommonsChunkPlugin` can move modules that occur in multiple entry chunks to a new entry chunk (the commons chunk). The runtime is moved to the commons chunk too. This means the old entry chunks are initial chunks now. See all options in the list of plugins.

### 优化

There are optimizing plugins that can merge chunks depending on specific criteria. See list of plugins.

- LimitChunkCountPlugin
- MinChunkSizePlugin
- AggressiveMergingPlugin

### Named chunks

The `require.ensure` function accepts an additional 3rd parameter. This must be a string. If two split point pass the same string they use the same chunk.

### require.include

`require.include(request)`

`require.include` is a webpack specific function that adds a module to the current chunk, but doesn’t evaluate it (The statement is removed from the bundle).

Example:

```js
require.ensure(["./file"], function(require) {
  require("./file2");
});
```

// is equals to

```js
require.ensure([], function(require) {
  require.include("./file");
  require("./file2");
});
```

`require.include` can be useful if a module is in multiple child chunks. A `require.include` in the parent would include the module and the instances of the modules in the chunk chunks would disappear.

### Examples

- [Simple](https://github.com/webpack/webpack/tree/master/examples/code-splitting)
- [with bundle-loader](https://github.com/webpack/webpack/tree/master/examples/code-splitting-bundle-loader)
- [with context](https://github.com/webpack/webpack/tree/master/examples/code-splitted-require.context)
- [with amd and context](https://github.com/webpack/webpack/tree/master/examples/code-splitted-require.context-amd)
- [with deduplication](https://github.com/webpack/webpack/tree/master/examples/code-splitted-dedupe)
- [named-chunks](https://github.com/webpack/webpack/tree/master/examples/named-chunks)
- [multiple entry chunks](https://github.com/webpack/webpack/tree/master/examples/multiple-entry-points)
- [multiple commons chunks](https://github.com/webpack/webpack/tree/master/examples/multiple-commons-chunks)

For a running demo see the [example-app](http://webpack.github.io/example-app/). Check Network in DevTools.

## 样式表

### 嵌入样式表

通过`style-loader`和`css-loader`，可以将样式表嵌入一个webpack javascript bundle。This way you can modularize your stylesheets with your other modules. This way stylesheets are as easy as `require("./stylesheet.css")`.

安装：

	npm install style-loader css-loader --save-dev

配置。例如：

```js
{
    // ...
    module: {
        loaders: [
            { test: /\.css$/, loader: "style-loader!css-loader" }
        ]
    }
}
```

For compile-to-css languages see the according loaders for configuration examples. You can pipe them…

**Keep in mind that it’s difficult to manage the execution order of modules, so design your stylesheets so that order doesn’t matter. (But you can rely on the order in one css file.)**

使用：

```js
    // in your modules just require the stylesheet
    // This has the side effect that a <style>-tag is added to the DOM.
    require("./stylesheet.css");
```

### 分离 CSS BUNDLE

In combination with the `extract-text-webpack-plugin` it’s possible to generate a native css output file.

With Code Splitting we can use two different modes:

- Create one css file per initial chunk and embed stylesheets into additional chunks. (recommended)
- Create one css file per initial chunk which also contains styles from additional chunks.

The first mode is recommended because it’s optimial in regards to initial page loading time. In small apps with multiple entry points the second mode could be better because of HTTP request overheads and caching.

安装插件：

	npm install extract-text-webpack-plugin --save

To use the plugin you need to flag modules that should be moved into the css file with a special loader. After the compilation in the optimizing phase of webpack the plugin checks which modules are relevant for extraction (in the first mode only these that are in an initial chunk). These modules are compiled for node.js usage and executed to get the content. Additionally the modules are recompiled in the original bundle and replaced with an empty module.

A new asset is created for the extracted modules.

**styles from initial chunks into separate css output file**

This examples shows multiple entry points, but also works with a single entry point.

```js
// webpack.config.js
var ExtractTextPlugin = require("extract-text-webpack-plugin");
module.exports = {
    // The standard entry point and output config
    entry: {
        posts: "./posts",
        post: "./post",
        about: "./about"
    },
    output: {
        filename: "[name].js",
        chunkFilename: "[id].js"
    },
    module: {
        loaders: [
            // Extract css files
            {
                test: /\.css$/,
                loader: ExtractTextPlugin.extract("style-loader", "css-loader")
            },
            // Optionally extract less files
            // or any other compile-to-css language
            {
                test: /\.less$/,
                loader: ExtractTextPlugin.extract("style-loader", "css-loader!less-loader")
            }
            // You could also use other loaders the same way. I. e. the autoprefixer-loader
        ]
    },
    // Use the plugin to specify the resulting filename (and add needed behavior to the compiler)
    plugins: [
        new ExtractTextPlugin("[name].css")
    ]
}
```

You’ll get these output files:

- posts.js posts.css
- post.js post.css
- about.js about.css
- 1.js 2.js (contain embedded styles)

**all styles in separate css output file**

To use the second mode you just need to set the option `allChunks` to true:

```js
// ...
module.exports = {
    // ...
    plugins: [
        new ExtractTextPlugin("style.css", {
            allChunks: true
        })
    ]
}
```

You’ll get these output files:

- posts.js posts.css
- post.js post.css
- about.js about.css
- 1.js 2.js (don’t contain embedded styles)

**styles in commons chunk**

You can use a separate css file in combination with the `CommonsChunkPlugin`. In this case a css file for the commons chunk is emitted too.

```js
// ...
module.exports = {
    // ...
    plugins: [
        new webpack.optimize.CommonsChunkPlugin("commons", "commons.js"),
        new ExtractTextPlugin("[name].css")
    ]
}
```

You’ll get these output files:

- commons.js commons.css
- posts.js posts.css
- post.js post.css
- about.js about.css
- 1.js 2.js (contain embedded styles)

or with `allChunks: true`

- 1.js 2.js (don’t contain embedded styles)

## 优化

### Minimize

To minimize your scripts (and your css, if you use the css-loader) webpack supports a simple option:

`--optimize-minimize` resp. `new webpack.optimize.UglifyJsPlugin()`

That’s a simple but effective way to optimize your web app.

As you already know (if you’ve read the remaining docs) webpack gives your modules and chunks ids to identify them. Webpack can vary the distribution of the ids to get the smallest id length for often used ids with a simple option:

`--optimize-occurence-order` resp. `new webpack.optimize.OccurenceOrderPlugin()`

The entry chunks have higher priority for file size.

### Deduplication

If you use some libraries with cool dependency trees, it may occur that some files are identical. Webpack can find these files and deduplicate them. This prevents the inclusion of duplicate code into your bundle and instead applies a copy of the function at runtime. It doesn’t affect semantics. You can enable it with:

`--optimize-dedupe` resp. `new webpack.optimize.DedupePlugin()`

This feature adds some overhead to the entry chunk.

### Chunks

While writing your code, you may have already added **many** code split points to load stuff on demand. After compiling you might notice that there are **too many chunks that are too small** - creating larger HTTP overhead. Luckily, Webpack can post-process your chunks by merging them. You can provide two options:

- Limit the maximum chunk count with `--optimize-max-chunks 15` `new webpack.optimize.LimitChunkCountPlugin({maxChunks: 15})`
- Limit the minimum chunk size with `--optimize-min-chunk-size 10000` `new webpack.optimize.MinChunkSizePlugin({minChunkSize: 10000})`

Webpack will take care of it by merging chunks (it will prefer merging chunk that have duplicate modules). Nothing will be merged into the entry chunk, so as not to impact initial page loading time.

### Single-Page-App

A Single-Page-App is the type of web app webpack is designed and optimized for.

You may have split the app into multiple chunks, which are loaded at your router. The entry chunk only contains the router and some libraries, but no content. This works great while your user is navigating through your app, but for initial page load you need 2 round trips: One for the router and one for the current content page.

If you use the HTML5 History API to reflect the current content page in the URL, your server can know which content page will be requested by the client code. To save round trips the server can include the content chunk in the response: This is possible by just adding it as script tag. The browser will load both chunks parallel.

    <script src="entry-chunk.js" type="text/javascript" charset="utf-8"></script>
    <script src="3.chunk.js" type="text/javascript" charset="utf-8"></script>

You can extract the chunk filename from the stats.

### 多页面应用

When you compile a (real) multi-page app, you want to share common code between the pages. In fact this is really easy with webpack: Just compile with multiple entry points:

	webpack p1=./page1 p2=./page2 p3=./page3 [name].entry-chunk.js

```js
module.exports = {
    entry: {
        p1: "./page1",
        p2: "./page2",
        p3: "./page3"
    },
    output: {
        filename: "[name].entry.chunk.js"
    }
}
```

This will generate multiple entry chunks: p1.entry.chunk.js, p2.entry.chunk.js and p3.entry.chunk.js. But additional chunks can be shared by them.

If your entry chunks have some modules in common, there is a cool plugin for this. The `CommonsChunkPlugin` identifies common modules and put them into a commons chunk. You need to add two script tags to your page, one for the commons chunk and one for the entry chunk.

```js
var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
module.exports = {
    entry: {
        p1: "./page1",
        p2: "./page2",
        p3: "./page3"
    },
    output: {
        filename: "[name].entry.chunk.js"
    },
    plugins: [
        new CommonsChunkPlugin("commons.chunk.js")
    ]
}
```

This will generate multiple entry chunks: p1.entry.chunk.js, p2.entry.chunk.js and p3.entry.chunk.js, plus one `commons.chunk.js`. First load commons.chunk.js and than one of the xx.entry.chunk.js.

可以产生多个共用chunks，by selecting the entry chunks. And you can nest commons chunks.

```js
var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
module.exports = {
    entry: {
        p1: "./page1",
        p2: "./page2",
        p3: "./page3",
        ap1: "./admin/page1",
        ap2: "./admin/page2"
    },
    output: {
        filename: "[name].js"
    },
    plugins: [
        new CommonsChunkPlugin("admin-commons.js", ["ap1", "ap2"]),
        new CommonsChunkPlugin("commons.js", ["p1", "p2", "admin-commons.js"])
    ]
};
// page1.html: commons.js, p1.js
// page2.html: commons.js, p2.js
// page3.html: p3.js
// admin-page1.html: commons.js, admin-commons.js, ap1.js
// admin-page2.html: commons.js, admin-commons.js, ap2.js
```

Advanced hint: You can run code inside the commons chunk:

```js
var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
module.exports = {
    entry: {
        p1: "./page1",
        p2: "./page2",
        commons: "./entry-for-the-commons-chunk"
    },
    plugins: [
        new CommonsChunkPlugin("commons", "commons.js")
    ]
};
```

See also [multiple-entry-points](https://github.com/webpack/webpack/tree/master/examples/multiple-entry-points) example and [advanced multiple-commons-chunks](https://github.com/webpack/webpack/tree/master/examples/multiple-commons-chunks) example.

## （未）长期缓存

http://webpack.github.io/docs/long-term-caching.html

## （未）如何写加载器

http://webpack.github.io/docs/how-to-write-a-loader.html

## 多个入口点

http://webpack.github.io/docs/multiple-entry-points.html

**Prerequirement: Code Splitting**

If you need multiple bundles for multiple HTML pages you can use the “multiple entry points” feature. It will build multiple bundles at once. Additional chunks can be shared between these entry chunks and modules are only built once.

Hint: **When you want to start an entry chunk from a module, you are doing something wrong. Use Code Splitting instead!**

Every entry chunk contains the webpack runtime, so you can only load one entry chunk per page. (Hint: To bypass this limitation use the CommonsChunkPlugin to move the runtime into a single chunk.)

To use multiple entry points you can pass an object to the `entry` option. Each value is threaded as entry point and the **key** represents the name of the entry point.

When using multiple entry point you must override the default `output.filename` option. Otherwise each entry point would write to the same output file. Use `[name]` to get the name of the entry point.

例子：

```js
{
    entry: {
        a: "./a",
        b: "./b",
        c: ["./c", "./d"]
    },
    output: {
        path: path.join(__dirname, "dist"),
        filename: "[name].entry.js"
    }
}
```

更多例子：

- [multiple-entry-points](https://github.com/webpack/webpack/tree/master/examples/multiple-entry-points)
- [multi-part-library](https://github.com/webpack/webpack/tree/master/examples/multi-part-library)
- [multiple-commons-chunks](https://github.com/webpack/webpack/tree/master/examples/multiple-commons-chunks)

## 库与EXTERNALS

http://webpack.github.io/docs/library-and-externals.html

You developed a library and want to distribute it in compiled/bundled versions (in addition to the modularized version). You want to allow the user to use it in a `<script>`-tag or with a amd loader (i. e. require.js). Or you depend on various precompilations and want to take the pain for the user and distribute it as simple compiled commonjs module.

**configuration options**

webpack has three configuration options that are relevant for these use cases: `output.library`, `output.libraryTarget` and `externals`.

`output.libraryTarget` allows you to specify the kind to the output. I.e. CommonJs, AMD, for usage in a script tag or as UMD module.

`output.library` allows you to optionally specify a name of your library.

`externals` allows you to specify dependencies for your library that are not resolved by webpack, but become dependencies of the output. This means they are imported from the environment during runtime.

例子。

compile library for usage in a `<script>`-tag

depends on "jquery", but jquery should not be included in the bundle.
library should be available at `Foo` in the global context.

```js
var jQuery = require("jquery");
var math = require("math-library");

function Foo() {}

// ...

module.exports = Foo;
```

Recommended configuration (only relevant stuff):

```js
{
    output: {
        // export itself to a global var
        libraryTarget: "var",
        // name of the global var: "Foo"
        library: "Foo"
    },
    externals: {
        // require("jquery") is external and available
        //  on the global var jQuery
        "jquery": "jQuery"
    }
}
```

Resulting bundle:

```js
var Foo = (/* ... webpack bootstrap ... */
({
    0: function(...) {
        var jQuery = require(1);
        /* ... */
    },
    1: function(...) {
        module.exports = jQuery;
    },
    /* ... */
});
```

**Applications and externals**

You can use the `externals` options for applications too, when you want to import an existing API into the bundle. I.e. you want to use jquery from CDN (separate `<script>` tag) and still want to `require("jquery")` in your bundle. Just specify it as external: `{ externals: { jquery: "jQuery" } }`.

**Resolving and externals**

Externals processing happens before resolving the request, which means you need to specify the unresolved request. Loaders are not applied to externals. You can (need to) externalize a request with loader: `require("bundle!jquery") { externals: { "bundle!jquery": "bundledJQuery" } }`

## （未）Shimming modules

http://webpack.github.io/docs/shimming-modules.html

## （未）测试

http://webpack.github.io/docs/testing.html

## （未）构建的性能

http://webpack.github.io/docs/build-performance.html

## （未）Hot Module Replacement with webpack

http://webpack.github.io/docs/hot-module-replacement-with-webpack.html

## （未）比较

http://webpack.github.io/docs/comparison.html

## （未）与Grunt

http://webpack.github.io/docs/usage-with-grunt.html

## 与Gulp

http://webpack.github.io/docs/usage-with-gulp.html

```js
var gulp = require("gulp");
var gutil = require("gulp-util");
var webpack = require("webpack");
var WebpackDevServer = require("webpack-dev-server");
```

**正常编译**

```js
gulp.task("webpack", function(callback) {
    // run webpack
    webpack({
        // configuration
    }, function(err, stats) {
        if(err) throw new gutil.PluginError("webpack", err);
        gutil.log("[webpack]", stats.toString({
            // output options
        }));
        callback();
    });
});
```

**webpack-dev-server**

Don’t be too lazy to integrate the webpack-dev-server into your development process. It’s an important tool for productivity.

```js
gulp.task("webpack-dev-server", function(callback) {
    // Start a webpack-dev-server
    var compiler = webpack({
        // configuration
    });

    new WebpackDevServer(compiler, {
        // server and middleware options
    }).listen(8080, "localhost", function(err) {
        if(err) throw new gutil.PluginError("webpack-dev-server", err);
        // Server listening
        gutil.log("[webpack-dev-server]", "http://localhost:8080/webpack-dev-server/index.html");

        // keep the server alive or continue?
        // callback();
    });
});
```

[样例gulp文件](https://github.com/webpack/webpack-with-common-libs/blob/master/gulpfile.js)

## 与bower

http://webpack.github.io/docs/usage-with-bower.html

To use components from bower you need to add two things to webpack:

Let webpack look in the `bower_components` folder.
Let webpack use the `main` field from the `bower.json` file.

See [configuration](http://webpack.github.io/docs/configuration.html) `resolve.modulesDirectories` and list of plugins `ResolverPlugin`.

```js
var path = require("path");
var webpack = require("webpack");
module.exports = {
    resolve: {
        root: [path.join(__dirname, "bower_components")]
    },
    plugins: [
        new webpack.ResolverPlugin(
            new webpack.ResolverPlugin.DirectoryDescriptionFilePlugin("bower.json", ["main"])
        )
    ]
}
```

**Prefer modules from npm over bower**

In many cases modules from npm are better than the same module from bower. Bower mostly contain only concatenated/bundled files which are:

- More difficult to handle for webpack
- More difficult to optimize for webpack
- Sometimes only useable without a module system

So prefer to use the CommonJs-style module and let webpack build it.

**Example react**

bower package vs. npm package

Note: the bower package is built with browserify and envify (`NODE_ENV = "production"`)

So we compare four configurations:

a) webpack + bower package (DefinePlugin makes no difference here as envify already removed debug code)
b) webpack + bower package + module.noParse for react
c) webpack + npm package
d) webpack + npm package + DefinePlugin with NODE_ENV = "production"




