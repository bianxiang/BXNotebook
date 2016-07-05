http://webpack.github.io/docs/tutorials/getting-started/

You’ll learn:

How to install webpack
How to use webpack
How to use loaders
How to use the development server

安装：

	$ npm install webpack -g

This makes the `webpack` command available.

**第一发**

add entry.js

```js
document.write("It works.");
```

add index.html

```
<html>
    <head>
        <meta charset="utf-8">
    </head>
    <body>
        <script type="text/javascript" src="bundle.js" charset="utf-8"></script>
    </body>
</html>
```

运行：

	$ webpack ./entry.js bundle.js

It will compile your file and create **a bundle file**.
If successful it displays something like this:

    Version: webpack 1.5.3
    Time: 129ms
        Asset  Size  Chunks             Chunk Names
    bundle.js  1524       0  [emitted]  main
    chunk    {0} bundle.js (main) 28 [rendered]
        [0] ./tutorials/getting-started/setup-compilation/entry.js 28 {0} [built]

Open index.html in your browser. It should display It works.

**第二发**

add content.js

```js
module.exports = "It works from content.js.";
```

update entry.js

```
- - document.write("It works.");
+ document.write(require("./content.js"));
```

And recompile with:

```
$ webpack ./entry.js bundle.js
```

Update your browser window and you should see the text It works from content.js.

webpack会分析entry文件对其他文件的依赖。这些依赖（称为模块）也会加入到 bundle.js。webpack会给每个模块一个唯一的id，bundle.js文件中所有模块都可以通过id方为。Only the entry module is executed on startup. A small **runtime** provides the `require` function and executes the dependencies when required.

**第一个加载器**

We want to add a css file to our application.

webpack自己只能处理js，我们需要 css-loader 处理css文件。需要 style-loader 将css应用到页面。

Run `npm install css-loader style-loader` to install the loaders.

add style.css

```css
body {
    background: yellow;
}
```

update entry.js

```
+ require("!style!css!./style.css");
  document.write(require("./content.js"));
```

**Recompile** and update your browser to see your application with yellow background.

By prefixing loaders to a module request, the module went through a loader pipeline. These loader transform the file content in specific ways. After these transformations are applied, the result is a javascript module.

**绑定加载器**

We don’t want to write such long requires `require("!style!css!./style.css");`.

We can bind file extensions to loaders so we just need to write: `require("./style.css")`.

update entry.js

```
- require("!style!css!./style.css");
+ require("./style.css");
  document.write(require("./content.js"));
```

Run the compilation with:

`webpack ./entry.js bundle.js --module-bind 'css=style!css'`

**配置文件**

将配置选项放入配置文件：add webpack.config.js

```js
module.exports = {
    entry: "./entry.js",
    output: {
        path: __dirname,
        filename: "bundle.js"
    },
    module: {
        loaders: [
            { test: /\.css$/, loader: "style!css" }
        ]
    }
};
```

Now we can just run: `webpack` to compile:

    Version: webpack 1.5.3
    Time: 341ms
        Asset  Size  Chunks             Chunk Names
    bundle.js  2920       0  [emitted]  main
    chunk    {0} bundle.js (main) 1368 [rendered]
        [0] ./tutorials/getting-started/config-file/entry.js 65 {0} [built]
        [1] ./tutorials/getting-started/config-file/content.js 45 {0} [built]
        [2] ./tutorials/getting-started/config-file/style.css 484 {0} [built]
        [3] ../~/css-loader!./tutorials/getting-started/config-file/style.css 57 {0} [built]
        [4] ../~/style-loader/addStyle.js 717 {0} [built]

The webpack command-line will try to load the file `webpack.config.js` in the current directory.

**更好看的输出**

If the project grows the compilation may take a bit longer. So we want to display some kind of progress bar. And we want colors…

We can achieve this with

`webpack --progress --colors`

**WATCH MODE**

We don’t want to manually recompile after every change…

`webpack --progress --colors --watch`

Webpack can cache unchanged modules and output files between compilations.

When using watch mode, webpack installs file watchers to all files, which were used in the compilation process. If any change is detected, it’ll run the compilation again. When caching is enabled, webpack keeps each module in memory and will reuse it if it isn’t changed.

**开发服务器**

Even better is the development server.

```
npm install webpack-dev-server -g
webpack-dev-server --progress --colors
```

It binds a small express server on `localhost:8080` which serves your static assets as well as the bundle (compiled automatically). It automatically updates the browser page when a bundle is recompiled (**socket.io**). Open `http://localhost:8080/webpack-dev-server/bundle` in your browser.

The dev server uses webpack’s watch mode. It also prevents webpack from emitting the resulting files to disk. Instead it keeps and serves the resulting files from memory.