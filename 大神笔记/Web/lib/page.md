[toc]

https://visionmedia.github.io/page.js/

Tiny ~1200 byte Express-inspired client-side router.

```js
page('/', index)
page('/user/:user', show)
page('/user/:user/edit', edit)
page('/user/:user/album', album)
page('/user/:user/album/sort', sort)
page('*', notfound)
page()
```

### 安装

```
$ npm install page # for browserify
$ component install visionmedia/page.js
$ bower install visionmedia/page.js
```

### 例子

To run examples do the following to install dev dependencies and run the example server:

```
$ git clone git://github.com/visionmedia/page.js
$ cd page.js
$ npm install
$ node examples
$ open http://localhost:4000
```

Currently we have examples for:

- basic minimal application showing basic routing
- notfound similar to basic with single-page 404 support
- album showing pagination and external links
- profile simple user profiles
- query-string shows how you can integrate plugins using the router
- state illustrates how the history state may be used to cache data
- server illustrates how to use the dispatch option to server initial content
- chrome Google Chrome style administration interface
- transitions Shows off a simple technique for adding transitions between "pages"
- partials using hogan.js to render mustache partials client side

NOTE: keep in mind these examples do not use jQuery or similar, so portions of the examples may be relatively verbose, though they're not directly related to page.js in any way.

### API

#### `page(path, callback[, callback ...])`

定义一个映射，路径到回调（可以多个）。回调方法有两个参数：`context`和`next`。调用`next`，将调用注册的下一个回调。

```js
page('/', user.list)
page('/user/:id', user.load, user.show)
page('/user/:id/edit', user.load, user.edit)
page('*', notfound)
```

Under certain conditions, links will be disregarded and will not be dispatched, such as:

- Links that are not of the same origin
- Links with the download attribute
- Links with the target attribute
- Links with the `rel="external"` attribute

#### `page(callback)`

等价于`page('*', callback)`，即注册通用中间件。

#### 导航到指定路径：`page(path)`、`page.show(path)`

导航到指定路径。

```js
$('.view').click(function(e){
  page('/user/12')
  e.preventDefault()
})
```

#### 重定向：`page(fromPath, toPath)`、`page.redirect(fromPath, toPath)`

从一个路径重定向到另一个。

#### `page.redirect(path)`

Calling `page.redirect` with only a string as the first parameter redirects to another route. Waits for the current route to push state and after **replaces** it with the new one leaving the browser history **clean**.

```js
page('/default', function(){
  // some logic to decide which route to redirect to
  if(admin) {
    page.redirect('/admin');
  } else {
    page.redirect('/guest');
  }
});

page('/default');
```

#### `page([options])`、`page.start([options])`

Register page's popstate / click bindings. If you're doing selective binding you'll like want to pass `{ click: false }` to specify this yourself. The following options are available:

- `click` bind to click events [true]
- `popstate` bind to popstate [true]
- `dispatch` perform initial dispatch [true]
- `hashbang` add `#!` before urls [`false`]
- `decodeURLComponents` remove URL encoding from path components (query string, pathname, hash) [true]

If you wish to load serve initial content from the server you likely will want to set `dispatch` to `false`.

#### `page.stop()`

Unbind both the popstate and click handlers.

#### `page.base([path])`

Get or set the base path. For example if page.js is operating within `/blog/*` set the base path to "/blog".

#### `page.exit(path, callback[, callback ...])`

定义一个退出路由映射。

Exit routes are called when a page changes, using the context from the previous change. For example:

```js
page('/sidebar', function(ctx, next) {
  sidebar.open = true
  next()
})

page.exit('/sidebar', function(next) {
  sidebar.open = false
  next()
})
```

#### `page.exit(callback)`

Equivalent to `page.exit('*', callback)`.

### Context

路由处理器接收一个上下文参数，可以用它共享状态，例如，`ctx.user =`。as well as the history "state" `ctx.state` that the `pushState` API provides.

`Context#save()`

利用`replaceState()`保存上下文状态。For example this is useful for caching HTML or other resources that were loaded for when a user presses "back".

`Context#handled`

If true, marks the context as **handled** to prevent default 404 behaviour. For example this is useful for the routes with interminate quantity of the callbacks.

`Context#canonicalPath`

Pathname including the "base" (if any) and query string "/admin/login?foo=bar".

`Context#path`

Pathname and query string "/login?foo=bar".

`Context#querystring`

Query string void of leading ? such as "foo=bar", defaults to "".

`Context#pathname`

The pathname void of query string "/login".

`Context#state`

The pushState state object.

`Context#title`

The pushState title.

### 路由

The router uses the same string-to-regexp conversion that Express does, so things like ":id", ":id?", and "*" work as you might expect.

另一个与Express类似的敌方是，允许设定多个回调。You can use this to your advantage to flatten nested callbacks, or simply to abstract components.

#### 概念分离

例如，有一个路由编辑用户，一个路由显示用户。二者都要先加载用户。一种实现方式是：

```js
page('/user/:user', load, show)
page('/user/:user/edit', load, edit)
```

利用`*`，我们可以让处理器处理所有以`/user`开头的路由：

```js
page('/user/*', load)
page('/user/:user', show)
page('/user/:user/edit', edit)
```

Likewise * can be used as catch-alls after all routes acting as a 404 handler, before all routes, in-between and so on. For example:

```js
page('/user/:user', load, show)
page('*', function(){
  $('body').text('Not found!')
})
```

#### 默认的404行为

By default when a route is not matched, page.js invokes `page.stop()` to unbind itself, and proceed with redirecting to the location requested. This means you may use page.js with a multi-page application without explicitly binding to certain links.

#### 参数和上下文

利用上下文在多个回调直接传递状态。

To build a load function that will load the user for subsequent routes you'll need to access the ":id" passed. You can do this with ctx.params.NAME much like Express:

```js
function load(ctx, next){
  var id = ctx.params.id
}

page('/user/:id', load, show)
```

#### state

When working with the `pushState` API, and page.js you may optionally provide state objects available when the user navigates the history.

For example if you had a photo application and you performed a relatively expensive search to populate a list of images, normally when a user clicks "back" in the browser the route would be invoked and the query would be made yet-again.

An example implemenation might look as follows:

```js
function show(ctx){
  $.getJSON('/photos', function(images){
    displayImages(images)
  })
}
```

You may utilize the history's state object to cache this result, or any other values you wish. This makes it possible to completely omit the query when a user presses back, providing a much nicer experience.

```js
function show(ctx){
  if (ctx.state.images) {
    displayImages(ctx.state.images)
  } else {
    $.getJSON('/photos', function(images){
      ctx.state.images = images
      ctx.save()
      displayImages(images)
    })
  }
}
```

NOTE: `ctx.save()` must be used if the state changes after the first tick (xhr, setTimeout, etc), otherwise it is optional and the state will be saved after dispatching.

#### 匹配路径

Here are some examples of what's possible with the string to RegExp conversion.

Match an explicit path:

`page('/about', callback)`
Match with required parameter accessed via ctx.params.name:

`page('/user/:name', callback)`
Match with several params, for example /user/tj/edit or /user/tj/view.

`page('/user/:name/:operation', callback)`
Match with one optional and one required, now /user/tj will match the same route as /user/tj/show etc:

`page('/user/:name/:operation?', callback)`
Use the wildcard char `*` to match across segments, available via `ctx.params[N]` where N is the index of `*` since you may use several. For example the following will match /user/12/edit, /user/12/albums/2/admin and so on.

`page('/user/*', loadUser)`
Named wildcard accessed, for example /file/javascripts/jquery.js would provide "/javascripts/jquery.js" as `ctx.params.file`:

`page('/file/:file(*)', loadUser)`
And of course RegExp literals, where the capture groups are available via `ctx.params[N]` where N is the index of the capture group.

`page(/^\/commits\/(\d+)\.\.(\d+)/, loadUser)`

### 插件

An example plugin examples/query-string/query.js demonstrates how to make plugins. It will provide a parsed `ctx.query` object derived from `node-querystring`.

Usage by using `"*"` to match any path in order to parse the query-string:

```js
page('*', parse)
page('/', show)
page()

function parse(ctx, next) {
  ctx.query = qs.parse(location.search.slice(1));
  next();
}

function show(ctx) {
  if (Object.keys(ctx.query).length) {
    document
      .querySelector('pre')
      .textContent = JSON.stringify(ctx.query, null, 2);
  }
}
```

### 支持IE8+

If you want the router to work in older version of Internet Explorer that don't support pushState, you can use the HTML5-History-API polyfill:

```
npm install html5-history-api
```

**How to use a Polyfill together with router (OPTIONAL):**

If your web app is located within a nested basepath, you will need to specify the basepath for the HTML5-History-API polyfill. Before calling `page.base()` use: `history.redirect([prefixType], [basepath])` - Translation link if required.

- `prefixType: [string|null]` - Substitute the string after the anchor (#) by default "/".
- basepath: [string|null] - Set the base path. See `page.base()` by default "/". (Note: Slash after pathname required)