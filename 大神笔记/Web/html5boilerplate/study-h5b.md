# HTML5 boilerplate研究

[toc]

## 文件列表

基本结构：

    .
    ├── css
    │   ├── main.css
    │   └── normalize.css
    ├── doc
    ├── img
    ├── js
    │   ├── main.js
    │   ├── plugins.js
    │   └── vendor
    │       ├── jquery.min.js
    │       └── modernizr.min.js
    ├── .htaccess
    ├── 404.html
    ├── apple-touch-icon-precomposed.png
    ├── index.html
    ├── humans.txt
    ├── robots.txt
    ├── crossdomain.xml
    └── favicon.ico

**.htaccess**

Apache默认配置。For more information, please refer to the [Apache Server Configs repository](https://github.com/h5bp/server-configs-apache).

其他服务器的配置参见：[Server Configs](https://github.com/h5bp/server-configs/blob/master/README.md)。

**404.html**

A helpful custom 404 to get you started.

**index.html**

首页模板。
If you are using Google Universal Analytics, make sure that you edit the corresponding snippet at the bottom to include your analytics ID.

**humans.txt**

列出团队成员，使用的技术等。

**robots.txt**

列出不想让搜索引擎抓取的文件。

**crossdomain.xml**

A template for working with cross-domain requests. [About crossdomain.xml](misc.md#crossdomainxml).

**Icons**

替换默认的`favicon.ico`和Apple Touch Icon。If you want to use different Apple Touch Icons for different resolutions please refer to the [according documentation](extend.md#apple-touch-icons).

## HTML

### The `no-js` class

The `no-js` class is provided in order to allow you to more easily and
explicitly add custom styles based on whether JavaScript is disabled
(`.no-js`) or enabled (`.js`). Using this technique also helps [avoid the FOUC](http://paulirish.com/2009/avoiding-the-fouc-v3/).

### Language attribute

Please consider specifying the language of your content by adding the `lang`
attribute to `<html>` as in this example:

	<html class="no-js" lang="en">

### title和meta的顺序

它们的顺序是重要的，因为：

1) 字符集声明（`<meta charset="utf-8">`）：必须放在[前1024个字节内](http://www.whatwg.org/specs/web-apps/current-work/multipage/semantics.html#charset)。2，尽量早的指定(before any content that could be controlled by an **attacker**, such as a `<title>` element) in order to avoid a potential [encoding-related security issue](http://code.google.com/p/doctype-mirror/wiki/ArticleUtf7) in Internet Explorer

2) the meta tag for compatibility mode(`<meta http-equiv="X-UA-Compatible" content="IE=edge">`): needs to be included before all other tags except for the `<title>` and the other `<meta>` tags. 参见：http://msdn.microsoft.com/en-us/library/cc288325.aspx


### `X-UA-Compatible`

在有多个渲染引擎的IE中确保使用最新年的。即使用户使用IE8或IE9，也有可能它们使用的不是最新的渲染引擎。解决此问题需要加上：

	<meta http-equiv="X-UA-Compatible" content="IE=edge">

The `meta` tag tells the IE rendering engine it should use the latest, or edge,
version of the IE rendering environment.

This line breaks validation. To avoid this edge case issue it is recommended
that you **remove this line and use the [`.htaccess`](https://github.com/h5bp/server-configs-apache)** (or [other server config](https://github.com/h5bp/server-configs)) to send these headers instead.

You also might want to read [Validating: X-UA-Compatible](http://groups.google.com/group/html5boilerplate/browse_thread/thread/6d1b6b152aca8ed2).

如果你的网站不在标准端口，需要在服务器端设置这个头。因为IE的偏好选项'Display intranet sites in Compatibility View'默认是勾选的。

### Mobile viewport

There are a few different options that you can use with the [`viewport` meta tag](https://docs.google.com/present/view?id=dkx3qtm_22dxsrgcf4 "Viewport and Media Queries - The Complete Idiot's Guide"). You can find out more in [the Apple developer docs](http://j.mp/mobileviewport). HTML5 Boilerplate comes with a simple setup that strikes a good balance for general use cases.

```html
	<meta name="viewport" content="width=device-width, initial-scale=1">
```

### Modernizr

HTML5 Boilerplate uses a custom build of Modernizr.

[Modernizr](http://modernizr.com) is a JavaScript library which adds classes to
the `html` element based on the results of feature test and which ensures that
all browsers can make use of HTML5 elements (as it includes the HTML5 Shiv).
This allows you to target parts of your CSS and JavaScript based on the
features supported by a browser.

In general, in order to keep page load times to a minimum, it's best to call
any JavaScript at the end of the page because if a script is slow to load
from an external server it may cause the whole page to hang. That said, the
Modernizr script *needs* to run *before* the browser begins rendering the page,
so that browsers lacking support for some of the new HTML5 elements are able to
handle them properly. 因此Modernizr脚本是唯一需要在文档顶部同步加载的脚本。

### BrowseHappy Prompt

The main content area of the boilerplate includes a prompt to install an up to
date browser for users of IE 6/7.

### Google CDN for jQuery

The Google CDN version of the jQuery JavaScript library is referenced towards
the bottom of the page using a protocol-independent path (read more about this
in the [FAQ](faq.md)). A local fallback of jQuery is included for rare instances when the CDN version might not be available, and to facilitate offline
development.

The Google CDN version is chosen over other potential candidates (like the
[jQuery CDN](http://jquery.com/download/#jquery-39-s-cdn-provided-by-maxcdn))
because it's fast in absolute terms and it has the best overall
[penetration](http://httparchive.org/trends.php#perGlibs) which increases the
odds of having a copy of the library in your user's browser cache.

While the Google CDN is a strong default solution your site or application may
require a different configuration. Testing your site with services like [WebPageTest](http://www.webpagetest.org/) and browser tools like [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/) or [YSlow](http://developer.yahoo.com/yslow/) will help you examine the real world performance of your site and can show where you can optimize your specific site or application.

### Google Univeral Analytics Tracking Code

Finally, an optimized version of the Google Univeral Analytics tracking code is
included. Google recommends that this script be placed at the top of the page.
Factors to consider: if you place this script at the top of the page, you’ll
be able to count users who don’t fully load the page, and you’ll incur the max
number of simultaneous connections of the browser.

Further information:

* [Optimizing the Google Universal Analytics Snippet](http://mathiasbynens.be/notes/async-analytics-snippet#universal-analytics)
* [Introduction to Analytics.js](https://developers.google.com/analytics/devguides/collection/analyticsjs/)

**N.B.** The Google Universal Analytics snippet is included by default mainly because Google Analytics is [currently one of the most popular tracking solutions](http://trends.builtwith.com/analytics/Google-Analytics) out there. However, its usage isn't set in stone, and you SHOULD consider exploring the [alternatives](https://en.wikipedia.org/wiki/List_of_web_analytics_software) and use whatever suits your needs best!

[HTML5 Boilerplate homepage](http://html5boilerplate.com) | [Documentation
table of contents](TOC.md)

## CSS

HTML5 Boilerplate's CSS includes:

* Normalize.css
* Useful defaults
* Common helpers
* Placeholder media queries
* Print styles

This starting CSS does not rely on the presence of conditional class names,
conditional style sheets, or Modernizr, and it is ready to use no matter what
your development preferences happen to be.

### Normalize.css

We include [Normalize.css](https://necolas.github.io/normalize.css/) — a modern, HTML5-ready alternative to CSS resets.

As opposed to CSS resets, Normalize.css:

* 只针对需要被归一的样式
* 保留有用的浏览器默认，而不是移除它们
* 更正BUG或浏览器的不一致
* improves usability with subtle improvements
* doesn't clutter the debugging tools
* has better documentation

For more information about Normalize.css, please refer to its [project page](https://necolas.github.com/normalize.css/), as well as this [blog post](http://nicolasgallagher.com/about-normalize-css/).

### Useful defaults

在`Normalize.css`的基础上引入了一些基础样式：

* provide basic typography settings that improve text readability
* protect against unwanted `text-shadow` during text highlighting
* tweak the default alignment of some elements (e.g.: `img`, `video`,
  `fieldset`, `textarea`)
* style the prompt that is displayed to users using an outdated browser

You are free and even encouraged to modify or add to these base styles as your
project requires.

### Common helpers

Along with the base styles, we also provide some commonly used helper classes.

`.hidden`

The `hidden` class can be added to any element that you want to hide visually
and from screen readers. It could be an element that will be populated and
displayed later, or an element you will hide with JavaScript.

`.visuallyhidden`

The `visuallyhidden` class can be added to any element that you want to hide
visually, while still have its content accessible to screen readers.

See also:

* [CSS in Action: Invisible Content Just for Screen Reader Users](http://www.webaim.org/techniques/css/invisiblecontent/)
* [Hiding content for accessibility](http://snook.ca/archives/html_and_css/hiding-content-for-accessibility)
* [HTML5 Boilerplate - Issue #194](https://github.com/h5bp/html5-boilerplate/issues/194/).

`.invisible`

The `invisible` class can be added to any element that you want to hide
visually and from screen readers, but without affecting the layout.

As opposed to the `hidden` class that effectively removes the element from the
layout, the `invisible` class will simply make the element invisible while
keeping it in the flow and not affecting the positioning of the surrounding
content.

__N.B.__ Try to stay away from, and don't use the classes specified above for
[keyword stuffing](https://en.wikipedia.org/wiki/Keyword_stuffing) as you will
harm your site's ranking!

`.clearfix`

The `clearfix` class can be added to any element to ensure that it always fully
contains its floated children.

Over the years there have been many variants of the clearfix hack, but currently, we use the [micro clearfix](http://nicolasgallagher.com/micro-clearfix-hack/).

### Media Queries

HTML5 Boilerplate makes it easy for you to get started with a [_mobile first_](http://www.lukew.com/presos/preso.asp?26) and [_responsive web design_](http://www.alistapart.com/articles/responsive-web-design/) approach to development. But it's worth remembering that there are [no silver bullets](http://www.cloudfour.com/css-media-query-for-mobile-is-fools-gold/).

We include placeholder media queries to help you build up your mobile styles for wider viewports and high-resolution displays. It's recommended that you adapt
these media queries based on the content of your site rather than mirroring the
fixed dimensions of specific devices.

If you do not want to take the _mobile first_ approach, you can simply edit or
remove these placeholder media queries. One possibility would be to work from
wide viewports down, and use `max-width` media queries instead (e.g.:
`@media only screen and (max-width: 480px)`).

For more features that can help you in your mobile web development, take a look
into our [Mobile Boilerplate](https://github.com/h5bp/mobile-boilerplate).

### Print styles

Lastly, we provide some useful print styles that will optimize the printing process, as well as make the printed pages easier to read.

At printing time, these styles will:

* strip all background colors, change the font color to black, and remove the `text-shadow` — done in order to [help save printer ink and speed up the printing process](http://www.sanbeiji.com/archives/953)
* underline and expand links to include the URL — done in order to allow users to know where to refer to (exceptions to this are: the links that are   [fragment identifiers](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a#attr-href), or use the [`javascript:` pseudo protocol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/void#JavaScript_URIs))
* expand abbreviations to include the full description — done in order to allow users to know what the abbreviations stands for
* provide instructions on how browsers should break the content into pages and on [orphans/widows](https://en.wikipedia.org/wiki/Widows_and_orphans), namely, we instruct [supporting browsers](https://en.wikipedia.org/wiki/Comparison_of_layout_engines_%28Cascading_Style_Sheets%29#Grammar_and_rules) that they should:

  * ensure the table header (`<thead>`) is [printed on each page spanned by the table](http://css-discuss.incutio.com/wiki/Printing_Tables)
  * prevent block quotations, preformatted text, images and table rows from being split onto two different pages
  * ensure that headings never appear on a different page than the text they
    are associated with
  * ensure that [orphans and widows](https://en.wikipedia.org/wiki/Widows_and_orphans) do [not appear on printed pages](http://css-tricks.com/almanac/properties/o/orphans/)

The print styles are included along with the other `css` to [avoid the additional HTTP request](http://www.phpied.com/delay-loading-your-print-css/). Also, they should always be included last, so that the other styles can be overwritten.

[HTML5 Boilerplate homepage](http://html5boilerplate.com) | [Documentation
table of contents](TOC.md)

## JavaScript

**main.js**

This file can be used to contain or reference your site/app JavaScript code. For larger projects, you can make use of a JavaScript module loader, like [Require.js](http://requirejs.org/), to load any other scripts you need to run.

**plugins.js**

This file can be used to contain all your plugins, such as **jQuery plugins** and other 3rd party scripts.

One approach is to put jQuery plugins inside of a `(function($){ ...})(jQuery);` closure to make sure they're in the jQuery namespace safety blanket. Read more about [jQuery plugin authoring](http://docs.jquery.com/Plugins/Authoring#Getting_Started)

By default the `plugins.js` file contains a small script to avoid `console` errors in browsers that lack a `console`. The script will make sure that, if a console method isn't available, that method will have the value of empty function, thus, preventing the browser from throwing an error.

**vendor**

This directory can be used to contain all 3rd party library code.

Minified versions of the latest jQuery and Modernizr libraries are included by
default. You may wish to create your own [custom Modernizr build](http://www.modernizr.com/download/).

## crossdomain.xml

The cross-domain policy file is an XML document that gives **a web client** —
such as Adobe Flash Player, Adobe Reader, etc. — permission to handle data
across multiple domains, by:

 * granting read access to data
 * permitting the client to include custom headers in cross-domain requests
 * granting permissions for socket-based connections

__e.g.__ If a client hosts content from a particular source domain and that
content makes requests directed towards a domain other than its own, the remote
domain would need to host a cross-domain policy file in order to grant access
to the source domain and allow the client to continue with the transaction.

For more in-depth information, please see Adobe's [cross-domain policy file specification](https://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html).


## robots.txt

The `robots.txt` file is used to give instructions to web robots on what can
be crawled from the website.

By default, the file provided by this project includes the next two lines:

 * `User-agent: *` -  the following rules apply to all web robots
 * `Disallow:` - everything on the website is allowed to be crawled

If you want to disallow certain pages you will need to specify the path in a
`Disallow` directive (e.g.: `Disallow: /path`) or, if you want to disallow
crawling of all content, use `Disallow: /`.

The `/robots.txt` file is not intended for access control, so don't try to
use it as such. Think of it as a "No Entry" sign, rather than a locked door.
URLs disallowed by the `robots.txt` file might still be indexed without being
crawled, and the content from within the `robots.txt` file can be viewed by
anyone, potentially disclosing the location of your private content! So, if
you want to block access to private content, use proper authentication instead.

For more information about `robots.txt`, please see:

  * [robotstxt.org](http://www.robotstxt.org/)
  * [How Google handles the `robots.txt` file](https://developers.google.com/webmasters/control-crawl-index/docs/robots_txt)

## 扩展

https://github.com/h5bp/html5-boilerplate/blob/master/dist/doc/extend.md




