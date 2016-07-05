[toc]

## How-to

中文版：http://segmentfault.com/a/1190000002552008
原版：https://github.com/petehunt/webpack-howto

这是 Webpack 怎么做事情的烹饪书。其中包含了我们在 Instagram 做的大部分功能，而且都是在用的功能。

我的建议是: 把这个当成是 Webpack 的文档开始学习，然后看官方文档的具体说明。

**预备条件**

你懂 Broserify, RequireJS 或者类似的打包工具

你注重这些东西:

- 代码分包（Bundle splitting）
- 异步加载
- 静态资源（图片, CSS）的打包

**为什么用 webpack?**

他像 Browserify，但是将你的应用打包为多个文件。如果你的单页面应用有多个页面，那么用户只从下载对应页面的代码。当他么访问到另一个页面，他们不需要重新下载**通用的**代码.

他在很多地方能**替代** Grunt 跟 Gulp 。因为他能够编译打包 CSS，做 CSS 预处理，编译 JS 方言，打包图片，还有其他一些。

它支持 AMD 跟 CommonJS，以及其他一些模块系统，(Angular, ES6)。如果你不知道用什么，就用 CommonJS。

### 针对 Browserify 用户

```
browserify main.js > bundle.js
```

等价于：

```
webpack main.js bundle.js
```

Webpack 比 Browserify 更强大，你一般会用 `webpack.config.js` 来组织各个过程：

```js
// webpack.config.js
module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'
  }
};
```

### 如何调用 webpack

切换到有 webpack.config.js 的目录然后运行:

- `webpack` 执行一次开发的编译
- `webpack -p` 针对发布环境编译(压缩代码)
- `webpack --watch` 来进行开发过程持续的增量编译(飞快地!)
- `webpack -d` 来生成 SourceMaps

### JavaScript 方言

Webpack 对应 Browsserify transform 和 RequireJS 插件的工具称为 loader. 下边是 Webpack 加载 CoffeeScript 和 Facebook JSX-ES6 的配置(你需要 `npm install jsx-loader coffee-loader`):

```js
// webpack.config.js
module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      { test: /\.coffee$/, loader: 'coffee-loader' },
      { test: /\.js$/, loader: 'jsx-loader?harmony' } // loaders 可以接受 querystring 格式的参数
    ]
  }
};
```

要开启后缀名的自动补全, 你需要设置 `resolve.extensions` 参数指明那些文件 Webpack 是要搜索的:

```js
// webpack.config.js
module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      { test: /\.coffee$/, loader: 'coffee-loader' },
      { test: /\.js$/, loader: 'jsx-loader?harmony' }
    ]
  },
  resolve: {
    // 现在可以写 require('file') 代替 require('file.coffee')
    extensions: ['', '.js', '.json', '.coffee']
  }
};
```
### 样式表和图片

首先更新你的代码用 `require()` 加载静态资源(就像在 Node 里使用 require()):

```js
require('./bootstrap.css');
require('./myapp.less');

var img = document.createElement('img');
img.src = require('./glyph.png');
```

当你引用 CSS(或者 LESS 吧)，Webpack 会将 CSS **内联到** JavaScript 包当中。`require()` 会在页面当中插入一个 `<style>` 标签。当你引入图片，Webpack 在包当中插入对应图片的 URL，这个 URL 是由 `require()` 返回的.

你需要配置 Webpack（添加 loader）：

```js
// webpack.config.js
module.exports = {
  entry: './main.js',
  output: {
    path: './build', // 图片和 JS 会到这里来
    publicPath: 'http://mycdn.com/', // 这个用来生成URL，比如图片的 URL
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      { test: /\.less$/, loader: 'style-loader!css-loader!less-loader' }, // 用 ! 来连接多个人 loader
      { test: /\.css$/, loader: 'style-loader!css-loader' },
      {test: /\.(png|jpg)$/, loader: 'url-loader?limit=8192'} // 内联 base64 URLs, 限定 <=8k 的图片, 其他的用 URL
    ]
  }
};
```

### 功能开关

有些代码我们只想在开发环境使用(比如 log), 或者 dogfooging 的服务器里边(比如内部员工正在测试的功能). 在你的代码中, 引用全局变量吧：

```js
    if (__DEV__) {
      console.warn('Extra logging');
    }
    // ...
    if (__PRERELEASE__) {
      showSecretFeature();
    }
```
然后配置 Webpack 当中的对应全局变量:

```js
// webpack.config.js

    // definePlugin takes raw strings and inserts them, so you can put strings of JS if you want.
    var definePlugin = new webpack.DefinePlugin({
      __DEV__: JSON.stringify(JSON.parse(process.env.BUILD_DEV || 'true')),
      __PRERELEASE__: JSON.stringify(JSON.parse(process.env.BUILD_PRERELEASE || 'false'))
    });

    module.exports = {
      entry: './main.js',
      output: {
        filename: 'bundle.js'
      },
      plugins: [definePlugin]
    };
```
然后你在控制台里用 `BUILD_DEV=1 BUILD_PRERELEASE=1 webpack` 编译。注意一下因为 `webpack -p` 会删除死代码，所以你不用担心秘密功能泄漏。

### 多个入点（entry points）

比如你有两个页面： profile 页面、 feed 页面。当用户访问 profile, 你不想让他们下载 feed 页面的代码。So make multiple bundles: create one "main module" (called an entrypoint) per page:

```js
// webpack.config.js
module.exports = {
  entry: {
    Profile: './profile.js',
    Feed: './feed.js'
  },
  output: {
    path: 'build',
    filename: '[name].js' // 模版基于上边 entry 的 key
  }
};
```

针对 profile, 在页面当中插入 `<script src="build/Profile.js"></script>`. feed 页面也是一样.

### 优化共用代码

feed 页面跟 profile 页面共用很多代码(比如 React 还有通用的样式和 component). Webpack 可以分析出来他们有多少共用模块, 然后生成一个共享的包用于代码的缓存.

```js
// webpack.config.js

var webpack = require('webpack');

var commonsPlugin =
  new webpack.optimize.CommonsChunkPlugin('common.js');

module.exports = {
  entry: {
    Profile: './profile.js',
    Feed: './feed.js'
  },
  output: {
    path: 'build',
    filename: '[name].js'
  },
  plugins: [commonsPlugin]
};
```

在上一个步骤的 script 标签前面加上 `<script src="build/common.js"></script>`.

### 异步加载

CommonJS是同步的。但webpack提供了一种异步指定依赖的方式。这项功能对客户端路由很有帮助：让你能够按需加载资源。

声明你想要异步加载的那个“分割点”（split point）。比如：

```js
if (window.location.pathname === '/feed') {
  showLoadingState();
  require.ensure([], function() { // 语法奇葩, 但是有用
    hideLoadingState();
    require('./feed').show(); // 函数调用后, 模块保证在同步请求下可用
  });
} else if (window.location.pathname === '/profile') {
  showLoadingState();
  require.ensure([], function() {
    hideLoadingState();
    require('./profile').show();
  });
}
```

Webpack 会完成其余的工作, 生成额外的 chunk 文件帮你加载好.

Webpack 在 HTML script 标签中加载他们时会假设这些文件在你的根路径下。你可以用 `output.publicPath` 来配置。

```js
// webpack.config.js
output: {
    path: "/home/proj/public/assets", // path 指向 Webpack 编译能的资源位置
    publicPath: "/assets/" // 引用你的文件时考虑使用的地址
}
```

### 相关资源

看一看真实环境当中一个成功的团队怎么使用 Webpack: http://www.tudou.com/programs/view/6KB6lNbVzhs/ .
这是 Peter Hunt 在 OSCon 大会关于 Instagram 如何使用 Webpack 的演讲.



