[toc]

**现代前端学习路线图**

本指引针对已有一点Web开发基础的同学：需要写过一点HTML、CSS、JavaScript。

> 下面提到的所有书籍，注意搜索是否有最新版。

# 1 HTML

基础的 HTML 没有多少内容。主要目标是掌握所有 HTML 元素（element）、属性（attribute）以及像实体（如空格 `&nbsp;`）等这些细碎的内容。

接下来学习 HTML5 的新特性。主要学习像 section、video 等新增的元素、placeholder 等新增的属性、及表单方面的增强。由于 HTML5 的诸多新特性都要涉及 JavaScript，如存储等，会在 JavaScript 部分学习。因此其实 HTML5 本身(markup 部分）也没有多少要学习的。

小结：基础的 HTML 和 HTML5，看看诸如 W3C School 之类的网页就可以了。很多 HTML5 的知识会在 CSS、JavaScript 的部分学习。

至于 Canvas、OpenGL 等内容，和游戏紧密相关，单独学习。

# 2 CSS

分 CSS 基础、CSS 2.x 全面掌握、CSS 布局、CSS3 四部分。

**2.1 CSS 基础**

选一些非常简单基础的书，主要目的是对 CSS 建立整体感觉。如经典的 《精通CSS：高级Web标准解决方案》（《CSS Mastery》）。

**2.2 CSS 2.x 全面掌握**

CSS 2.x 定义了目前我们使用的绝大多数特性。

本阶段目标是全面掌握目前兼容性较好的 CSS 2.x。掌握所有属性、取值等知识。如清楚知道哪些属性是继承的、哪些不继承。百分比或相对值相对于什么值。

这个阶段最好的书是《CSS权威指南，第3版》（CSS The Definitive Guide）。此书有点老了，没有更新版本。停留在 CSS 2.x 阶段。但够用了。主要是没有一本书比它讲 CSS 2.x 更全面、细致。

> 另一方面，市面上讲 CSS3 的书多数只讲 CSS3 新增的特性，不讲 CSS 2.x 部分。因此学 CSS 2.x 部分还是要看之前的书。

**2.3 CSS 布局**

全部掌握了 CSS 2&3 后并不意味着就能做好布局。布局指的是尺寸设定、居中、多列等于尺寸位置相关的内容。要做好布局，关键是掌握“模式”。

这方面最好的书是 《Pro HTML5 and CSS3 Design Patterns》。这本书难度较大，但一旦掌握，会大幅提高在布局方面的能力。本书可能需要反复阅读。注意去掉思维定式。

同时本书还更加深入的讲述了 HTML 与 CSS 结合方面的知识。如 HTML 视角与 CSS 视角看 HTML 元素的角度的差异。

**2.4 CSS3**

CSS3 的知识点还是比较多的。其中有常用的特性，如圆角、阴影、外盒布局等。也有用的少的，如多列、图片剪切等。

诸如 CSS3 新增选择符这类特性，要注意[看浏览器兼容器](http://caniuse.com/)。

诸如 CSS3 transform、animation，一般需要看专门的书。（还好其实不复杂。）

CSS3 的书非常多，推荐几本：《CSS3 Foundations》、《The Book of CSS3》、《CSS3 The Missing Manual 3rd Edition》。其中，《CSS3 The Missing Manual 3rd Edition》
不仅讲了 CSS3，还较全面的讲了 CSS 2.x。

# 3 JavaScript

分 JavaScript 语言核心（经典）、ES 2015、服务器端编程（宿主环境是 Node.js）、客户端编程（宿主环境是浏览器）三部分。

**3.1 语言核心**

先看 《JavaScript: The Good Part》。本书可以作为语言入门书，强调了“不是所有 JavaScript 特性都是好用的” 这一观点。但随着时代推移，本书并不是圣经：有点内容讲的不够全面客观，有些内容搞复杂了。

有基础后，前面了解 JavaScript 语言核心。经典 JavaScript 看 《JavaScript 权威指南(第6版) (JavaScript：The Definitive Guide)》，第一部分。注意本书很厚。有些内容觉得烦的可以快速读，有印象就行。

如果要进一步精通 JavaScript，需要看一些设计模式的书。如深入思考探讨原型继承与基于类的继承等。这方面的书有《Effective JavaScript》《JavaScript Patterns》《Secrets of the JavaScript - Ninja》等等。注意不要全信书中的内容。这些书需要写过几十万行 JavaScript 代码后再来体会。

**3.2 ES2015**

指 JavaScript 语言标准最新（可用）版。建议入门从 babel 的网页看起：http://babeljs.io/docs/learn-es2015/。重点学习 Promise、Generator。

Promise 深入学习看 bluebird 这个库。

Generator 较难。看 http://davidwalsh.name/es6-generators。看 `co` 这个库。

**3.3 Node.js**

此话题单独讲。

**3.4 客户端编程**

即与 DOM、HTML5 等浏览器相关的知识。虽然不专门学习这部分知识也能写网页 —— 直接使用 jQuery 等工具。但有些基础知识，如事件的冒泡、元素的定位与大小等，即使用 jQuery，也需要这些基础知识。

另外随着现在移动到 Web 的大量使用，移动 Web 开发用到更多这类基础知识。如触摸事件与点击事件的处理等。

这部分内容入门可以看看 W3C School 的教程。深入看 《JavaScript 权威指南(第6版) (JavaScript：The Definitive Guide)》，第二部分。有些细节需要搜索专门的博客学习。

# 4 CSS 预处理框架

常用的有 LESS、Sass、Stylus。现在推荐精通 Stylus。但 LESS、Sass 也要能看懂、基本掌握。看官网学习。

# 5 JavaScript 方言

如 CoffeeScript、TypeScript。目前 CoffeeScript 在客户端、服务器端开发都很方便。看官网学习。

# 6 js库、框架

## 6.1 jQuery

jQuery 几乎是必学的。学习也分三个阶段：

1. 百度点资料快速上手，对 jQuery 有感觉。
2. 找本书系统学习
3. 看最新的官方文档。

## 6.2 工具库

包括 lodash、moment 等。

## 6.3 前端MVC框架

入门学习 BackBone。关键掌握通过路由实现 Single-Page-Web。至于数据绑定等，了解就好。BackBone不一定是最好最适合的。

## 6.4 动画库

入门学习 velocity.js

# 7 HTML5 boilerplate

https://github.com/h5bp/html5-boilerplate

与其说 HTML5 boilerplate 是一个框架，不如说它交给我们如何为前端开发打下良好的基础。如页面编码、必须的 meta、特性侦测等。即重视 HTML 的 head 部分，CSS 的 reset，JavaScript 的入口处。

# 8 工作流

Gulp、WebPack、require.js、Babel 等。

Chrome 调试控制台及相关插件。

# 9 响应式设计

入门：《Implementing Responsive Design》






