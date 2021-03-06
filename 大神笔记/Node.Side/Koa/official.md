[toc]

http://koajs.com/

## 介绍

Koa is a new web framework designed by the team behind Express, which aims to be a smaller, more expressive, and more robust foundation for web applications and APIs. Through leveraging **generators** Koa allows you to ditch callbacks and greatly increase error-handling. Koa自身不带任何中间件。

Koa currently requires node 0.11.x for the `--harmony` flag which exposes **generators** to your script.

## 应用

Koa应用是一个对象，包含一组中间件产生器函数。

This includes methods for common tasks like content-negotiation, cache freshness, proxy support, and redirection among others. Despite supplying a reasonably large number of helpful methods Koa maintains a small footprint, as no middleware are bundled.

简单例子：

```js
    var koa = require('koa');
    var app = koa();

    app.use(function *(){
      this.body = 'Hello World';
    });

    app.listen(3000);
```

### 级联

使用回调很难做到友好，但通过产生器，可以实现真正的中间件。Contrasting Connect's implementation which simply passes control through series of functions until one returns, Koa yields "downstream", then control flows back "upstream".

下面的例子，请求先经过 x-response-time 和 logging 中间件处理，然后才被响应中间件处理。当中间件调用`yield next`时，函数暂停，将控制传给`next`中间件。所有中间件执行完后，战从后向前，依次恢复每个中间件，执行上游功能。

```js
    var koa = require('koa');
    var app = koa();

    // x-response-time
    app.use(function *(next){
      var start = new Date;
      yield next;
      var ms = new Date - start;
      this.set('X-Response-Time', ms + 'ms');
    });

    // logger
    app.use(function *(next){
      var start = new Date;
      yield next;
      var ms = new Date - start;
      console.log('%s %s - %s', this.method, this.url, ms);
    });

    // response
    app.use(function *(){
      this.body = 'Hello World';
    });

    app.listen(3000);
```

### 设置

应用的设置通过`app`的属性实现，目前支持：

- `app.name`：可选，应用名
- `app.env`：默认为`NODE_ENV`或`development`
- `app.proxy`：when `true` proxy header fields will be trusted
- `app.subdomainOffset`：offset of `.subdomains` to ignore [2]

### app.listen(...)

一个或多个Koa应用可以挂在到一起形成一个更大的应用，组成一个HTTP服务器。

Create and return an HTTP server, passing the given arguments to `Server#listen()`. These arguments are documented on [nodejs.org](http://nodejs.org/api/http.html#http_server_listen_port_hostname_backlog_callback). The following is a useless Koa application bound to port 3000:

```js
    var koa = require('koa');
    var app = koa();
    app.listen(3000);
```

The `app.listen(...)` method is simply sugar for the following:

```js
    var http = require('http');
    var koa = require('koa');
    var app = koa();
    http.createServer(app.callback()).listen(3000);
```

This means you can spin up the same application as both HTTP and HTTPS or on multiple addresses:

```js
    var http = require('http');
    var koa = require('koa');
    var app = koa();
    http.createServer(app.callback()).listen(3000);
    http.createServer(app.callback()).listen(3001);
```

### app.callback()

Return a callback function suitable for the `http.createServer()` method to handle a request. You may also use this callback function to mount your koa app in a Connect/Express app.

### app.use(function)

Add the given middleware function to this application. See [Middleware](https://github.com/koajs/koa/wiki#middleware) for more information.

### app.keys=

设置签名cookie用的key。These are passed to **KeyGrip**, however you may also pass your own KeyGrip instance. For example the following are acceptable:

```js
    app.keys = ['im a newer secret', 'i like turtle'];
    app.keys = new KeyGrip(['im a newer secret', 'i like turtle'], 'sha256');
```

These keys may be rotated and are used when signing cookies with the `{ signed: true }` option:

```js
    this.cookies.set('name', 'tobi', { signed: true });
```

### 错误处理

默认所有的错误输出到 `stderr`，除非设置`NODE_ENV`为"test"。要执行自定义的错误处理逻辑，可以监听"error"事件：

```js
    app.on('error', function(err) {
      log.error('server error', err);
    });
```

If an error in the req/res cycle and it is not possible to respond to the client, the `Context` instance is also passed:

```js
    app.on('error', function(err, ctx) {
      log.error('server error', err, ctx);
    });
```

When an error occurs and it is still possible to respond to the client, aka no data has been written to the socket, Koa will respond appropriately with a 500 "Internal Server Error". In either case an app-level "error" is emitted for logging purposes.

## 上下文

Koa上下文对象封装了node得`request`和`response`对象。

A Context is created per request, and is referenced in middleware as the receiver, or the `this` identifier, as shown in the following snippet:

```js
    app.use(function *(){
      this; // is the Context
      this.request; // is a koa Request
      this.response; // is a koa Response
    });
```

Many of the context's accessors and methods simply delegate to their `ctx.request` or `ctx.response` equivalents for convenience, and are otherwise identical. For example `ctx.type` and `ctx.length` delegate to the `response` object, and `ctx.path` and `ctx.method` delegate to the `request`.

### ctx.req

Node的`request`对象。

### ctx.res

Node的`response`对象。

绕过Koa的相应处理是不行的。避免使用下面node的属性：

- `res.statusCode`
- `res.writeHead()`
- `res.write()`
- `res.end()`

### ctx.request

koa的请求对象

### ctx.response

koa的相应对象

### ctx.state

将信息传给中间件或前端视图的推荐方式。

```js
	this.state.user = yield User.find(id);
```

### ctx.app

指向应用。

### ctx.cookies.get(name, [options])

Get cookie name with options:

- `signed`：the cookie requested should be signed

koa uses the [cookies](https://github.com/jed/cookies) module where options are simply passed.

### ctx.cookies.set(name, value, [options])

Set cookie name to value with options:

- `signed`：sign the cookie value
- `expires`：a Date for cookie expiration
- `path`：cookie path, `/` by default
- `domain`：cookie domain
- `secure`：secure cookie
- `httpOnly`：server-accessible cookie, true by default

koa uses the [cookies](https://github.com/jed/cookies) module where options are simply passed.

### ctx.throw([msg], [status], [properties])

Helper method to throw an error with a `.status` property defaulting to `500` that will allow Koa to respond appropriately. The following combinations are allowed:

```js
    this.throw(403);
    this.throw('name required', 400);
    this.throw(400, 'name required');
    this.throw('something exploded');
```

For example `this.throw('name required', 400)` is equivalent to:

```js
    var err = new Error('name required');
    err.status = 400;
    throw err;
```

Note that these are user-level errors and are flagged with `err.expose` meaning the messages are appropriate for client responses, which is typically not the case for error messages since you do not want to leak failure details.

You may optionally pass a properties object which is merged into the error as-is, useful for decorating machine-friendly errors which are reported to the requester upstream.

```js
    this.throw(401, 'access_denied', { user: user });
    this.throw('access_denied', { user: user });
```

koa uses [http-errors](https://github.com/jshttp/http-errors) to create errors.

### ctx.assert(value, [msg], [status], [properties])

Helper method to throw an error similar to `.throw()` when `!value`. Similar to node's `assert()` method.

```js
	this.assert(this.user, 401, 'User not found. Please login!');
```

koa uses [http-assert](https://github.com/jshttp/http-assert) for assertions.

### ctx.respond

To bypass Koa's built-in response handling, you may explicitly set `this.respond = false;`. Use this if you want to write to the raw res object instead of letting Koa handle the response for you.

Note that using this is **not** supported by Koa. This may break intended functionality of Koa middleware and Koa itself. Using this property is considered a hack and is only a convenience to those wishing to use traditional `fn(req, res)` functions and middleware within Koa.

### Request aliases

The following accessors and alias Request equivalents:

ctx.header
ctx.headers
ctx.method
ctx.method=
ctx.url
ctx.url=
ctx.originalUrl
ctx.href
ctx.path
ctx.path=
ctx.query
ctx.query=
ctx.querystring
ctx.querystring=
ctx.host
ctx.hostname
ctx.fresh
ctx.stale
ctx.socket
ctx.protocol
ctx.secure
ctx.ip
ctx.ips
ctx.subdomains
ctx.is()
ctx.accepts()
ctx.acceptsEncodings()
ctx.acceptsCharsets()
ctx.acceptsLanguages()
ctx.get()

### Response aliases

The following accessors and alias Response equivalents:

ctx.body
ctx.body=
ctx.status
ctx.status=
ctx.message
ctx.message=
ctx.length=
ctx.length
ctx.type=
ctx.type
ctx.headerSent
ctx.redirect()
ctx.attachment()
ctx.set()
ctx.remove()
ctx.lastModified=
ctx.etag=

## 请求

A Koa Request object is an abstraction on top of node's vanilla `request` object, providing additional functionality that is useful for every day HTTP server development.

### request.header

Request header object.

### request.headers

Request header object. Alias as `request.header`.

### request.method

Request method.

### request.method=

设置请求方法，用于实现某些中间件，如`methodOverride()`。

### request.length

请求的`Content-Length`，返回数字，或`undefined`。

### request.url

Get request URL.

### request.url=

设置请求的URL，用于URL重写。

### request.originalUrl

Get request original URL.

### request.href

Get full request URL, include protocol, host and url.

    this.request.href
    // => http://example.com/foo/bar?q=1

### request.path

Get request pathname.

### request.path=

Set request pathname and retain **query-string** when present.

### request.querystring

Get raw query string void of ?.

### request.querystring=

Set raw query string.

### request.search

Get raw query string with the ?.

### request.search=

Set raw query string.

### request.host

Get host (hostname:port) when present. Supports `X-Forwarded-Host` when `app.proxy` is true, otherwise `Host` is used.

### request.hostname

Get hostname when present. Supports `X-Forwarded-Host` when `app.proxy` is true, otherwise `Host` is used.

### request.type

Get request `Content-Type` void of parameters such as "charset".

```js
    var ct = this.request.type;
    // => "image/png"
    request.charset
```

Get request charset when present, or undefined:

```js
    this.request.charset
    // => "utf-8"
```

### request.query

获取解析好的查询字符串。Note that this getter does not support nested parsing.

### request.query=

Set query-string to the given object. Note that this setter does not support nested objects.

```js
	this.query = { next: '/login' };
```

### request.fresh

Check if a request cache is "fresh", aka the contents have not changed. This method is for cache negotiation between **If-None-Match / ETag**, and **If-Modified-Since** and **Last-Modified**. It should be referenced after setting one or more of these response headers.

```js
        this.set('ETag', '123');

        // cache is ok
        if (this.fresh) {
          this.status = 304;
          return;
        }

        // cache is stale
        // fetch new data
        this.body = yield db.find('something');
```

### request.stale

Inverse of `request.fresh`.

### request.protocol

Return request protocol, "https" or "http". Supports `X-Forwarded-Proto` when `app.proxy` is true.

### request.secure

Shorthand for `this.protocol == "https"` to check if a request was issued via TLS.

### request.ip

Request remote address. Supports `X-Forwarded-For` when `app.proxy` is true.

### request.ips

When `X-Forwarded-For` is present and `app.proxy` is enabled an array of these ips is returned, ordered from upstream -> downstream. When disabled an empty array is returned.

### request.subdomains

Return subdomains as an array.

Subdomains are the dot-separated parts of the host before the main domain of the app. By default, the domain of the app is assumed to be the last two parts of the host. This can be changed by setting `app.subdomainOffset`.

For example, if the domain is `"tobi.ferrets.example.com"`: If `app.subdomainOffset` is not set, `this.subdomains` is `["ferrets", "tobi"]`. If `app.subdomainOffset` is `3`, `this.subdomains` is `["tobi"]`.

### request.is(types...)

Check if the incoming request contains the `"Content-Type"` header field, and it **contains** any of the give mime types. If there is no request body, `undefined` is returned. If there is no content type, or the match fails false is returned. Otherwise, it returns the matching content-type.

```js
    // With Content-Type: text/html; charset=utf-8
    this.is('html'); // => 'html'
    this.is('text/html'); // => 'text/html'
    this.is('text/*', 'text/html'); // => 'text/html'

    // When Content-Type is application/json
    this.is('json', 'urlencoded'); // => 'json'
    this.is('application/json'); // => 'application/json'
    this.is('html', 'application/*'); // => 'application/json'

    this.is('html'); // => false
```

For example if you want to ensure that only images are sent to a given route:

```js
    if (this.is('image/*')) {
      // process
    } else {
      this.throw(415, 'images only!');
    }
```

### Content Negotiation

Koa's `request` object includes helpful content negotiation utilities powered by `accepts` and `negotiator`. These utilities are:

```js
    request.accepts(types)
    request.acceptsEncodings(types)
    request.acceptsCharsets(charsets)
    request.acceptsLanguages(langs)
```

If no types are supplied, all acceptable types are returned.

If multiple types are supplied, the best match will be returned. If no matches are found, a false is returned, and you should send a 406 "Not Acceptable" response to the client.

In the case of missing accept headers where any type is acceptable, the first type will be returned. Thus, the order of types you supply is important.

### request.accepts(types)

Check if the given type(s) is acceptable, returning the best match when true, otherwise false. The type value may be one or more mime type string such as "application/json", the extension name such as "json", or an array ["json", "html", "text/plain"].

```js
    // Accept: text/html
    this.accepts('html');
    // => "html"

    // Accept: text/*, application/json
    this.accepts('html');
    // => "html"
    this.accepts('text/html');
    // => "text/html"
    this.accepts('json', 'text');
    // => "json"
    this.accepts('application/json');
    // => "application/json"

    // Accept: text/*, application/json
    this.accepts('image/png');
    this.accepts('png');
    // => false

    // Accept: text/*;q=.5, application/json
    this.accepts(['html', 'json']);
    this.accepts('html', 'json');
    // => "json"

    // No Accept header
    this.accepts('html', 'json');
    // => "html"
    this.accepts('json', 'html');
    // => "json"
```
You may call `this.accepts()` as many times as you like, or use a switch:

```js
    switch (this.accepts('json', 'html', 'text')) {
      case 'json': break;
      case 'html': break;
      case 'text': break;
      default: this.throw(406, 'json, html, or text only');
    }
```

### request.acceptsEncodings(encodings)

Check if encodings are acceptable, returning the best match when true, otherwise false. Note that you should include identity as one of the encodings!

```js
    // Accept-Encoding: gzip
    this.acceptsEncodings('gzip', 'deflate', 'identity');
    // => "gzip"

    this.acceptsEncodings(['gzip', 'deflate', 'identity']);
    // => "gzip"
```

When no arguments are given all accepted encodings are returned as an array:

```
    // Accept-Encoding: gzip, deflate
    this.acceptsEncodings();
    // => ["gzip", "deflate", "identity"]
```

Note that the identity encoding (which means no encoding) could be unacceptable if the client explicitly sends `identity;q=0`. Although this is an edge case, you should still handle the case where this method returns false.

### request.acceptsCharsets(charsets)

Check if charsets are acceptable, returning the best match when true, otherwise false.

```js
    // Accept-Charset: utf-8, iso-8859-1;q=0.2, utf-7;q=0.5
    this.acceptsCharsets('utf-8', 'utf-7');
    // => "utf-8"

    this.acceptsCharsets(['utf-7', 'utf-8']);
    // => "utf-8"
```

When no arguments are given all accepted charsets are returned as an array:

```js
    // Accept-Charset: utf-8, iso-8859-1;q=0.2, utf-7;q=0.5
    this.acceptsCharsets();
    // => ["utf-8", "utf-7", "iso-8859-1"]
```

### request.acceptsLanguages(langs)

Check if langs are acceptable, returning the best match when true, otherwise false.

```js
    // Accept-Language: en;q=0.8, es, pt
    this.acceptsLanguages('es', 'en');
    // => "es"

    this.acceptsLanguages(['en', 'es']);
    // => "es"
```

When no arguments are given all accepted languages are returned as an array:

```js
    // Accept-Language: en;q=0.8, es, pt
    this.acceptsLanguages();
    // => ["es", "pt", "en"]
```

### request.idempotent

Check if the request is idempotent.

### request.socket

Return the request socket.

### request.get(field)

Return request header.

## 响应

A Koa Response object is an abstraction on top of node's vanilla `response` object, providing additional functionality that is useful for every day HTTP server development.

### response.header

Response header object.

### response.socket

Request socket.

### response.status

Get response status. By default, response.status is not set unlike node's `res.statusCode` which defaults to 200.

### response.status=

Set response status via numeric code:

100 "continue"
101 "switching protocols"
102 "processing"
200 "ok"
201 "created"
202 "accepted"
203 "non-authoritative information"
204 "no content"
205 "reset content"
206 "partial content"
207 "multi-status"
300 "multiple choices"
301 "moved permanently"
302 "moved temporarily"
303 "see other"
304 "not modified"
305 "use proxy"
307 "temporary redirect"
400 "bad request"
401 "unauthorized"
402 "payment required"
403 "forbidden"
404 "not found"
405 "method not allowed"
406 "not acceptable"
407 "proxy authentication required"
408 "request time-out"
409 "conflict"
410 "gone"
411 "length required"
412 "precondition failed"
413 "request entity too large"
414 "request-uri too large"
415 "unsupported media type"
416 "requested range not satisfiable"
417 "expectation failed"
418 "i'm a teapot"
422 "unprocessable entity"
423 "locked"
424 "failed dependency"
425 "unordered collection"
426 "upgrade required"
428 "precondition required"
429 "too many requests"
431 "request header fields too large"
500 "internal server error"
501 "not implemented"
502 "bad gateway"
503 "service unavailable"
504 "gateway time-out"
505 "http version not supported"
506 "variant also negotiates"
507 "insufficient storage"
509 "bandwidth limit exceeded"
510 "not extended"
511 "network authentication required"

NOTE: don't worry too much about memorizing these strings, if you have a typo an error will be thrown, displaying this list so you can make a correction.

### response.message

Get response status message. By default, `response.message` is associated with `response.status`.

### response.message=

Set response status message to the given value.

### response.length=

Set response `Content-Length` to the given value.

### response.length

Return response `Content-Length` as a number when present, or deduce from `this.body` when possible, or `undefined`.

### response.body

Get response body.

### response.body=

Set response body to one of the following:

- `string` written. The `Content-Type` is defaulted to text/html or text/plain, both with a default charset of utf-8. The `Content-Length` field is also set.
- `Buffer` written. The `Content-Type` is defaulted to `application/octet-stream`, and `Content-Length` is also set.
- `Stream` piped. The Content-Type is defaulted to `application/octet-stream`.
- `Object` json-stringified. The Content-Type is defaulted to `application/json`.
- `null` no content response

If `response.status` has not been set, Koa will **automatically** set the status to `200` or `204`.

### response.get(field)

Get a response header field value with case-**insensitive** field.

```js
	var etag = this.get('ETag');
```

### response.set(field, value)

Set response header field to value:

```js
	this.set('Cache-Control', 'no-cache');
```

### response.set(fields)

Set several response header fields with an object:

```js
    this.set({
      'Etag': '1234',
      'Last-Modified': date
    });
```

### response.remove(field)

Remove header field.

### response.type

Get response `Content-Type` void of parameters such as "charset".

```js
    var ct = this.type;
    // => "image/png"
```

### response.type=

设置响应的`Content-Type`，使用mime字符串或文件扩展名。

```js
    this.type = 'text/plain; charset=utf-8';
    this.type = 'image/png';
    this.type = '.png';
    this.type = 'png';
```

注意：自动为你设置字符集是"utf-8"，例如当你设置`response.type = 'html'`时。但如果显式定义完整时，如`response.type = 'text/html'`，不会设置字符集。

### response.is(types...)

Very similar to `this.request.is()`. Check whether the response type is one of the supplied types. This is particularly useful for creating middleware that manipulate responses.

For example, this is a middleware that minifies all HTML responses except for streams.

```js
    var minify = require('html-minifier');

    app.use(function *minifyHTML(next){
      yield next;

      if (!this.response.is('html')) return;

      var body = this.body;
      if (!body || body.pipe) return;

      if (Buffer.isBuffer(body)) body = body.toString();
      this.body = minify(body);
    });
```

### response.redirect(url, [alt])

Perform a [302] redirect to url.

字符串`"back"`会被特殊处理，以支持`Referrer`，when `Referrer` is not present `alt` or "/" is used.

```js
    this.redirect('back');
    this.redirect('back', '/index.html');
    this.redirect('/login');
    this.redirect('http://google.com');
```

To alter the default status of 302, simply assign the status before or after this call. To alter the body, assign it after this call:

```js
    this.status = 301;
    this.redirect('/cart');
    this.body = 'Redirecting to shopping cart';
    response.attachment([filename])
```

Set `Content-Disposition` to "attachment" to signal the client to prompt for download. Optionally specify the filename of the download.

### response.headerSent

Check if a response header has already been sent. Useful for seeing if the client may be notified on error.

### response.lastModified

Return the `Last-Modified` header as a Date, if it exists.

### response.lastModified=

Set the `Last-Modified `header as an appropriate UTC string. You can either set it as a Date or date string.

```js
	this.response.lastModified = new Date();
```

### response.etag=

Set the ETag of a response including the wrapped `"`s. Note that there is no corresponding `response.etag` getter.

```js
	this.response.etag = crypto.createHash('md5').update(this.body).digest('hex');
```

### response.vary(field)

Vary on field.

## 链接

- 例子：https://github.com/koajs/examples
- 中间件、WIKI：https://github.com/koajs/koa/wiki
- Guide：https://github.com/koajs/koa/blob/master/docs/guide.md
- FAQ：https://github.com/koajs/koa/blob/master/docs/faq.md










