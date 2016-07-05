参考：http://segmentfault.com/a/1190000002551952

Webpack 的报错挺不友好的, 最初的时候我看着模块找不到没法搞明白
这种时候把中间过程打印出来看是不错的选择:

```
webpack --display-error-details
```

### 加载不同的资源

对应各种不同文件类型的资源，Webpack 有对应的模块 loader。比如 CoffeeScript 用的是 `coffee-loader`，其他还有很多：http://webpack.github.io/docs/list-of-loaders.html。
大致的写法也就这样子:

```js
  module: {
    loaders: [
      { test: /\.coffee$/, loader: 'coffee-loader' },
      { test: /\.js$/, loader: 'jsx-loader?harmony' } // loaders can take parameters as a querystring
    ]
  },
```

### CommonJS 与 AMD 支持

Webpack 对 CommonJS 的 AMD 的语法做了兼容，方便迁移代码。不过实际上，引用模块的规则是依据 CommonJS 来的：

```js
require('lodash') // 从模块目录查找
require('./file') // 按相对路径查找
```

AMD 语法中, 也要注意, 是按 CommonJS 的方案查找的

```js
define (require, exports, module) ->
  require('lodash') # commonjs 当中这样是查找模块的
  require('./file')
```

### 查找依赖

Webpack 是类似 Browserify 那样在本地按目录对依赖进行查找的。可以构造一个例子, 用 `--display-error-details` 查看查找过程。例子当中 `resolve.extensions` 用于指明程序自动补全识别哪些后缀,
注意一下, extensions 第一个是空字符串! 对应不需要后缀的情况.

```js
// webpack.config.js
module.exports = {
  entry: './a.js',
  output: {
    filename: 'b.js'
  },
  resolve: {
    extensions: ['', '.coffee', '.js']
  }
}
// a.js
require('./c')
```

    ➤➤ webpack --display-error-details
    Hash: e38f7089c39a1cf34032
    Version: webpack 1.5.3
    Time: 54ms
    Asset  Size  Chunks             Chunk Names
     b.js  1646       0  [emitted]  main
       [0] ./a.js 15 {0} [built] [1 error]

    ERROR in ./a.js
    Module not found: Error: Cannot resolve 'file' or 'directory' ./c in /Users/chen/Drafts/webpack/details
    resolve file
      /Users/chen/Drafts/webpack/details/c doesn't exist
      /Users/chen/Drafts/webpack/details/c.coffee doesn't exist
      /Users/chen/Drafts/webpack/details/c.js doesn't exist
    resolve directory
      /Users/chen/Drafts/webpack/details/c doesn't exist (directory default file)
      /Users/chen/Drafts/webpack/details/c/package.json doesn't exist (directory description file)
    [/Users/chen/Drafts/webpack/details/c]
    [/Users/chen/Drafts/webpack/details/c.coffee]
    [/Users/chen/Drafts/webpack/details/c.js]
     @ ./a.js 2:0-14

`./c` 是不存在, 从这个错误信息当中我们大致能了解 Webpack 是怎样查找的。大概就是会尝试各种文件名, 会尝试作为模块, 等等。一般模块就是查找 `node_modules`, 但这个也是能被配置的:
http://webpack.github.io/docs/configuration.html#resolve-modulesdirectories

### url-loader

稍微啰嗦一下这个 loader, 这个 loader 实际上是对 file-loader 的封装：https://github.com/webpack/url-loader。比如 CSS 文件当中有这样的引用:

```css
.demo {
  background-image: url('a.png');
}
```

那么对应这样的 loader 配置就能把 a.png 抓出来, 并且按照文件大小, 或者转化为 base64, 或者单独作为文件:

```js
module: {
  loaders: [
    {test: /\.(png|jpg)$/, loader: 'url-loader?limit=8192'} // inline base64 URLs for <=8k images, direct URLs for the rest
  ]
}
```

上边 ? 后边的 query 有两种写法, 可以看下文档: http://webpack.github.io/docs/using-loaders.html#query-parameters

由于 url-loader 是对 file-loader 的一个封装, 以因此带有后者一些功能: https://github.com/webpack/file-loader. 比如说, file-loader 有不弱的定义文件名的功能

```js
require("file?name=[path][name].[ext]?[hash]!./dir/file.png")
```

对应 url-loader 当中如果文件超出体积, 就给一个这样的文件名..

### 打成多个包

有时考虑类库代码的缓存, 我们会考虑打成多个包, 这样不难。比如下边的配置, 首先 entry 有多个属性, 对应多个 JavaScript 包, 然后 commonsPlugin 可以用于分析模块的共用代码, 单独打一个包出来:
https://github.com/petehunt/webpack-howto#8-optimizing-common-code
https://github.com/webpack/docs/wiki/optimization#multi-page-app

```js
// webpack.config.js
var webpack = require('webpack');
var commonsPlugin = new webpack.optimize.CommonsChunkPlugin('common.js');

module.exports = {
  entry: {
    Profile: './profile.js',
    Feed: './feed.js'
  },
  output: {
    path: 'build',
    filename: '[name].js' // Template based on keys in entry above
  },
  plugins: [commonsPlugin]
};
```

### 对文件做 revision

这个在文档上做了说明, 可以自动生成 js 文件的 Hash: http://webpack.github.io/docs/long-term-caching.html

```js
output: { chunkFilename: "[chunkhash].bundle.js" }
plugins: [
  function() {
    this.plugin("done", function(stats) {
      require("fs").writeFileSync(
        path.join(__dirname, "...", "stats.json"),
        JSON.stringify(stats.toJson()));
    });
  }
]
```

同时, 可以注册事件, 拿到生成的带 Hash 的文件的一个表
但是拿到那个表之后, 就需要自己写代码进行替换了.. 这有点麻烦
官网的 Issue 里提到个办法是生成 HTML 时引用 stats.json 的数据,
我此前的方案是生成 HTML 之后再进行替换, 相对赖上生成时写入更好一些

### 上线

另一份配置文件

用 webpack --config webpack.min.js 指定另一个名字的配置文件。这个文件当中可以写不一样配置, 专门用于代码上线时的操作。

### 压缩 JavaScript

因为代码都是 JavaScript, 所以压缩就很简单了, 加上一行 plugin 就好了：http://webpack.github.io/docs/list-of-plugins.html#uglifyjsplugin

```js
plugins: [
   new webpack.optimize.MinChunkSizePlugin(minSize)
]
```

### 压缩 React

React 官方提供的代码是已经合并的, 这个是 Webpack 不推荐的用法,
在合并话的代码上进行定制有点麻烦, Webpack 提供了设置环境变量来优化代码的方案:

```js
new webpack.DefinePlugin({
  "process.env": {
    NODE_ENV: JSON.stringify("production")
  }
})
```

https://github.com/webpack/webpack/issues/292#issuecomment-44804366

### 代码热替换

虽然文档上写得挺复杂的, 但如果只是简单的功能还是很容易的

第一步, 把 'webpack/hot/dev-server' 加入到打包的代码当中,
这个是对应 node_modules/webpack/ 目录当中的文件的:

```js
  entry: {
    main: ['webpack/hot/dev-server', './main'],
    vendor: ['lodash', './styles']
  },
```

启动服务器, 比如我是这样子的

webpack-dev-server --hot --quiet

正常可以看到提示说服务器已经起来了

http://localhost:8080/webpack-dev-server/

如果有 index.html 的话, 直接访问网址应该就能开始调试了

### 单独打包 CSS

因为公司里有这个需要求, 强制把 CSS 从 js 文件当中独立出来.
官方文档是以插件的形式做的: http://webpack.github.io/docs/stylesheets.html#separate-css-bundle

参考文档但是注意一下函数参数, 第一第二个参数是有区别的, 比如这样用:

```js
ExtractTextPlugin.extract('style-loader', 'css!less')
```

第一个参数是编译好以后的代码用的, 第二个参数是编译到源代码用的.. 有点难懂..




