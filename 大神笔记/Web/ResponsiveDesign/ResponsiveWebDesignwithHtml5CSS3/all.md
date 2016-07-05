## 1. HTML5、CSS3及响应式设计入门

CSS3是在CSS2基础上按模块构建的，而不是大一统的标准。

## 2. 媒体查询

媒体查询已经被广泛使用，而且也被浏览器广泛支持（如Firefox 3.6+、Safari 4+、Chrome 4+、
Opera 9.5+、iOS Safari 3.2+、Opera Mobile 10+、Android 2.1+和Internet Explorer 9+）。IE6、7、8的支持可以被修复。

创建媒体查询时，最常用的是设备的视口宽度（`width`）和屏幕宽度（`device-width`）。依我的经验，很少需要检测其他特性。

`device-width`是渲染表面的宽度（对我们来说，就是设备屏幕的宽度）。

Respond.js（https://github.com/scottjehl/Respond）是为Internet Explorer 8及更低版本增加媒体查询支持的最快的JavaScript工具。

## 3. 流动布局

固定布局经不起未来考验。

那些仅使用媒体查询来适应不同视口的固定宽度设计，只会从一组CSS媒体查询规则突变到另一组，两者之间没有任何平滑渐变。

两全其美：使用**百分比布局**创建流动的弹性界面，同时使用**媒体查询**来限制元素的变动范围。

### （未）3.5 弹性图片















