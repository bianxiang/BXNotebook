[toc]

## Why AMD

http://requirejs.org/docs/whyamd.html

This page talks about the design forces and use of the [Asynchronous Module Definition (AMD) API](https://github.com/amdjs/amdjs-api/wiki/AMD) for JavaScript modules, the module API supported by RequireJS.

### CommonJs

```js
var $ = require('jquery');
exports.myExample = function () {};
```

原来的 [CommonJS (CJS) list](http://groups.google.com/group/commonjs) 参与者决定设计的模块格式，不仅限于用户浏览器环境。它们希望先在浏览器中用一些“权宜之计”，希望影响浏览器厂商开发一些机制让这种模块格式能被原生支持。这些权宜之计包括：

- 用一台服务器将CJS模块转换为浏览器可用的形式。
- 或使用 XMLHttpRequest (XHR) 加载模块的文本，然后在浏览器中转换或解析。

CJS模块一个文件只允许包含一个模块，so a "transport format" would be used for bundling more than one module in a file for optimization/bundling purposes.

With this approach, the CommonJS group was able to work out dependency references and how to deal with circular dependencies, and how to get some properties about the current module. However, they did not fully embrace some things in the browser environment that cannot change but still affect module design:

- 网络加载
- 固有的异步性

It also meant they placed more of a burden on web developers to implement the format, and the stop-gap measures meant debugging was worse. eval-based debugging or debugging multiple files that are concatenated into one file have practical weaknesses. Those weaknesses may be addressed in browser tooling some day, but the end result: using CommonJS modules in the most common of JS environments, the browser, is non-optimal today.

### AMD

```js
define(['jquery'] , function ($) {
	return function () {};
});
```

依赖通过字符串形式的id指定。id可以被映射到不同路径。于是你可以替换不同的实现，如为单元测试提供Mock。

它比 CommonJS 模块好的原因是：

- 对浏览器更友好。
- 可以实现一个文件中定义多个模块。
- 可以将函数作为返回值。例如，作为构造函数。Node支持 `module.exports = function () {}`， 但 CommonJS 规范不支持。

### 模块定义

```js
//Calling define with a dependency array and a factory function
define(['dep1', 'dep2'], function (dep1, dep2) {
    //Define the module value by returning a value.
    return function () {};
});
```

### 命名模块

注意，上面的模块并未声明一个名字。这使得模块非常具有便携性（portable）。开发者可以将其移到不同的路径，使其具有不同的ID。AMD加载器将根据它被其他模块的引用方式决定其ID。

However, tools that combine multiple modules together for performance need a way to give names to each module in the optimized file. For that, AMD allows a string as the first argument to `define()`:

```js
//Calling define with module ID, dependency array, and factory function
define('myModule', ['dep1', 'dep2'], function (dep1, dep2) {
    //Define the module value by returning a value.
    return function () {};
});
```

应避免手工给模块命名。在开发时，保持一个模块一个文件。

### 语法糖

当依赖过多时，写法是很丑陋的，而且依赖与函数参数的对应也容易出错：

```js
define([ "require", "jquery", "blade/object", "blade/fn", "rdapi",
         "oauth", "blade/jig", "blade/url", "dispatch", "accounts",
         "storage", "services", "widgets/AccountPanel", "widgets/TabButton",
         "widgets/AddAccount", "less", "osTheme", "jquery-ui-1.8.7.min",
         "jquery.textOverflow"],
function (require,   $,        object,         fn,         rdapi,
          oauth,   jig,         url,         dispatch,   accounts,
          storage,   services,   AccountPanel,           TabButton,
          AddAccount,           less,   osTheme) {

});
```

为解决该问题，同时为可以容易的包装CommonJS模块，出现了下面这种称为“简化的CommonJS包装”形式：

```js
define(function (require) {
    var dependency1 = require('dependency1'),
        dependency2 = require('dependency2');
    return function () {};
});
```

AMD加载器利用 `Function.prototype.toString()` 解析出所有 `require()`调用，然后将它们还是转换为最基本的形式：

```js
define(['require', 'dependency1', 'dependency2'], function (require) {
    var dependency1 = require('dependency1'),
        dependency2 = require('dependency2');
    return function () {};
});
```

一些浏览器的 `Function.prototype.toString()` 是由问题的。因此最好先用一些用户工具做些转换，如 **RequireJS optimizer**。

### CommonJS的兼容性

上一届的语法糖并不难完全兼容 CommonJS 模块。However, the cases that are not supported would likely break in the browser anyway, since they generally assume synchronous loading of dependencies.

例如，一个不兼容的情况是，动态计算依赖，或`require()`中依赖名不是字符串字面量。例如：

```js
//BAD
var mod = require(someCondition ? 'a' : 'b');

//BAD
if (someCondition) {
    var a = require('a');
} else {
    var a = require('a1');
}
```

These cases are handled by the [callback-require](https://github.com/amdjs/amdjs-api/wiki/require), `require([moduleName], function (){})` normally present in AMD loaders.

The AMD execution model is better aligned with how ECMAScript Harmony modules are being specified. The CommonJS modules that would not work in an AMD wrapper will also not work as a Harmony module. AMD的代码执行方式更兼容未来。


