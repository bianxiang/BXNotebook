http://expressjs.com/4x/api.html

[toc]

## express()

Create an express application.

    var express = require('express');
    var app = express();

    app.get('/', function(req, res){
      res.send('hello world');
    });

    app.listen(3000);

## 应用

### settings

The following settings are provided to alter how Express will behave:

- `env`：Environment mode, 默认是`process.env.NODE_ENV` (NODE_ENV environment variable) or "development"
- `trust proxy` Enables reverse proxy support, disabled by default
- `subdomain offset`： The number of dot-separated parts of the host to remove to access subdomain, two by default
- `jsonp callback name` Changes the default callback name of `?callback=`
- `json replacer` JSON replacer callback, null by default
- `case sensitive routing` 启用大小写敏感。默认禁用，即"/Foo"与"/foo"等价。
- `strict routing` Enable strict routing, by default "/foo" and "/foo/" are treated the same by the router
- `view cache` Enables view template compilation caching, enabled in production by default
- `view engine` The default engine extension to use when omitted
- `views`：视图的目录，默认是`process.cwd() + '/views'`。
- `x-powered-by` Enables the X-Powered-By: Express HTTP header, enabled by default

### app.set(name, value)

设置配置：

    app.set('title', 'My Site');
    app.get('title');
    // => "My Site"

### app.get(name)

获取配置：

    app.get('title');
    // => undefined

    app.set('title', 'My Site');
    app.get('title');
    // => "My Site"

### app.enable(name)

Set setting name to true.

    app.enable('trust proxy');
    app.get('trust proxy');
    // => true

### app.disable(name)

Set setting name to false.

    app.disable('trust proxy');
    app.get('trust proxy');
    // => false

### app.enabled(name)

Check if setting name is enabled.

    app.enabled('trust proxy');
    // => false

    app.enable('trust proxy');
    app.enabled('trust proxy');
    // => true

### app.disabled(name)

Check if setting name is disabled.

    app.disabled('trust proxy');
    // => true

    app.enable('trust proxy');
    app.disabled('trust proxy');
    // => false

### 中间件：app.use([path], function)

使用给定中间件函数。挂载路径可选，默认为"/"。

    var express = require('express');
    var app = express();

    // simple logger
    app.use(function(req, res, next){
      console.log('%s %s', req.method, req.url);
      next();
    });

    // respond
    app.use(function(req, res, next){
      res.send('Hello World');
    });

    app.listen(3000);

挂载路径会被去掉，对中间件函数不可见。这样，中间件可以不必在意前缀。但后续中间件仍可以看到此前缀。

例子，利用`express.static()`中间件伺服静态文件：

    // GET /javascripts/jquery.js
    // GET /style.css
    // GET /favicon.ico
    app.use(express.static(__dirname + '/public'));

例子，请求时静态资源都在`/static`路径下。在中间件这，这个前缀已被去掉。

    // GET /static/javascripts/jquery.js
    // GET /static/style.css
    // GET /static/favicon.ico
    app.use('/static', express.static(__dirname + '/public'));

中间件定义的顺序（`app.use()`的顺序）非常重要。中间件按定义顺序调用。例如，在所有中间件处理前处理应用日志中间件：

    var logger = require('morgan');

    app.use(logger());
    app.use(express.static(__dirname + '/public'));
    app.use(function(req, res){
      res.send('Hello');
    });

但如果向忽略静态文件的日志，则移动中间件的相对位置：

    app.use(express.static(__dirname + '/public'));
    app.use(logger());
    app.use(function(req, res){
      res.send('Hello');
    });

另一个常见的例子是，从多个目录伺服文件：

    app.use(express.static(__dirname + '/public'));
    app.use(express.static(__dirname + '/files'));
    app.use(express.static(__dirname + '/uploads'));

### 模板引擎：app.engine(ext, callback)

注册特定模板引擎。`callback`是引擎，`ext`是扩展名。默认会根据文件扩展名注册引擎。例如，当你尝试渲染"foo.jade"时，Express会在内部调用以下代码，并缓冲以提高性能：

	app.engine('jade', require('jade').__express);

未提供`.__express`的引擎，或想要让其他扩展名映射到引擎，可以使用该方法。For example mapping the EJS template engine to ".html" files:

	app.engine('html', require('ejs').renderFile);

In this case EJS provides a `.renderFile()` method with the same signature that Express expects: `(path, options, callback)`, though note that it aliases this method as `ejs.__express` internally so if you're using ".ejs" extensions you dont need to do anything.

Some template engines do not follow this convention, the [consolidate.js](https://github.com/visionmedia/consolidate.js) library was created to map all of node's popular template engines to follow this convention, thus allowing them to work seemlessly within Express.

    var engines = require('consolidate');
    app.engine('haml', engines.haml);
    app.engine('html', engines.hogan);

### app.param([name], callback)

Map logic to route parameters. 例如，当`:user`出现在路由路径时，应加载用户，并注入到`req.user`，or perform validations on the parameter input.

下面的diamond展示，回调就像中间件，但提供附加的参数，这里是`id`。

    app.param('user', function(req, res, next, id){
      User.find(id, function(err, user){
        if (err) {
          next(err);
        } else if (user) {
          req.user = user;
          next();
        } else {
          next(new Error('failed to load user'));
        }
      });
    });

Alternatively you may pass only a callback, in which case you have the opportunity to alter the `app.param()` API. For example the [express-params](http://github.com/expressjs/express-params) defines the following callback which allows you to restrict parameters to a given regular expression.

This example is a bit more advanced, checking if the second argument is a regular expression, returning the callback which acts much like the "user" param example.

    app.param(function(name, fn){
      if (fn instanceof RegExp) {
        return function(req, res, next, val){
          var captures;
          if (captures = fn.exec(String(val))) {
            req.params[name] = captures;
            next();
          } else {
            next('route');
          }
        }
      }
    });

The method could now be used to effectively validate parameters, or also parse them to provide capture groups:

    app.param('id', /^\d+$/);

    app.get('/user/:id', function(req, res){
      res.send('user ' + req.params.id);
    });

    app.param('range', /^(\w+)\.\.(\w+)?$/);

    app.get('/range/:range', function(req, res){
      var range = req.params.range;
      res.send('from ' + range[1] + ' to ' + range[2]);
    });

### 路由：app.VERB(path, [callback...], callback)

`app.VERB()`提供路由功能，其中`VERB`是某个HTTP动词，如`app.post()`。回调可以有多个，平等对待，类似中间件。这些回调可以通过`next('route')`绕过剩下的路由回调。This mechanism can be used to perform pre-conditions on a route then pass control to subsequent routes when there is no reason to proceed with the route matched.

The following snippet illustrates the most simple route definition possible. Express translates the path strings to regular expressions, used internally to match incoming requests. 匹配时不考虑查询字符串，for example "GET /" would match the following route, as would "GET /?name=tobi".

    app.get('/', function(req, res){
      res.send('hello world');
    });

还可以直接使用正则表达式，for example the following would match "GET /commits/71dbb9c" as well as "GET /commits/71dbb9c..4c084f9".

    app.get(/^\/commits\/(\w+)(?:\.\.(\w+))?$/, function(req, res){
      var from = req.params[0];
      var to = req.params[1] || 'HEAD';
      res.send('commit range ' + from + '..' + to);
    });

Several callbacks may also be passed, useful for re-using middleware that load resources, perform validations, etc.

    app.get('/user/:id', user.load, function(){
      // ...
    })

If you have multiple common middleware for a route, you can use the route api with `all`.

    var middleware = [loadForum, loadThread];

    app.route('/forum/:fid/thread/:tid')
    .all(loadForum)
    .all(loadThread)
    .get(function() { //... });
    .post(function() { //... });

Both middleware will be run for GET and POST requests.

### app.all(path, [callback...], callback)

This method functions just like the app.VERB() methods, however it matches all HTTP verbs.

This method is extremely useful for mapping "global" logic for specific path prefixes or arbitrary matches. For example if you placed the following route at the top of all other route definitions, it would require that all routes from that point on would require authentication, and automatically load a user. Keep in mind that these callbacks do not have to act as end points, `loadUser` can perform a task, then `next()` to continue matching subsequent routes.

	app.all('*', requireAuthentication, loadUser);

Or the equivalent:

    app.all('*', requireAuthentication)
    app.all('*', loadUser);

Another great example of this is white-listed "global" functionality. Here the example is much like before, however only restricting paths prefixed with "/api":

	app.all('/api/*', requireAuthentication);

### app.route(path)

Returns an instance of a single route which can then be used to handle HTTP verbs with optional middleware. Using `app.route()` is a recommended approach to avoiding duplicate route naming and thus typo errors.

    var app = express();

    app.route('/events')
    .all(function(req, res, next) {
      // runs for all HTTP verbs first
      // think of it as route specific middleware!
    })
    .get(function(req, res, next) {
      res.json(...);
    })
    .post(function(req, res, next) {
      // maybe add a new event...
    })

### app.locals

应用的局部变量，提供给应用内的所有模板。利用此功能可以向模板提供帮助函数，或应用级别的数据。

    app.locals.title = 'My App';
    app.locals.strftime = require('strftime');
    app.locals.email = 'me@myapp.com';

`app.locals`是Javascript对象。The properties added to it will be exposed as local variables within the application.

    app.locals.title
    // => 'My App'

    app.locals.email
    // => 'me@myapp.com'

Express默认值暴露一个应用级别的局部变量：`settings`。

	app.set('title', 'My App');
	// use settings.title in a view

### app.render(view, [options], callback)

Render a view with a callback responding with the rendered string. This is the app-level variant of `res.render()`, and otherwise behaves the same way.

    app.render('email', function(err, html){
      // ...
    });

    app.render('email', { name: 'Tobi' }, function(err, html){
      // ...
    });

### app.listen()

Bind and listen for connections on the given host and port, this method is identical to node's [http.Server#listen()](http://nodejs.org/api/http.html#http_server_listen_port_hostname_backlog_callback).

    var express = require('express');
    var app = express();
    app.listen(3000);

`express()`返回的`app`实际是一个Javascript函数，designed to be passed to node's http servers as a callback to handle requests. This allows you to provide both HTTP and HTTPS versions of your app with the same codebase easily, as the app does not inherit from these, it is simply a callback:

    var express = require('express');
    var https = require('https');
    var http = require('http');
    var app = express();

    http.createServer(app).listen(80);
    https.createServer(options, app).listen(443);

The `app.listen()` method is simply a convenience method defined as, if you wish to use HTTPS or provide both, use the technique above.

    app.listen = function(){
      var server = http.createServer(this);
      return server.listen.apply(server, arguments);
    };

## 请求

### req.params

这个属性是一个对象，包含具名的路由参数。例如，对于路由`/user/:name`，"name"属性可以通过`req.params.name`获得。该对象默认是`{}`。

    // GET /user/tj
    req.params.name
    // => "tj"

若路由定义使用这则表达式，捕获组可以通过`req.params[N]`获取，`N`表示第N个捕获组。This rule is applied to unnamed wild-card matches with string routes such as `/file/*`:

    // GET /file/javascripts/jquery.js
    req.params[0]
    // => "javascripts/jquery.js"

### req.query

此属性是一个对象，包含已解析的查询字符串，默认是`{}`。

    // GET /search?q=tobi+ferret
    req.query.q
    // => "tobi ferret"

    // GET /shoes?order=desc&shoe[color]=blue&shoe[type]=converse
    req.query.order
    // => "desc"

    req.query.shoe.color
    // => "blue"

    req.query.shoe.type
    // => "converse"

注意查询可以被解析为对象。

### req.param(name)

返回参数的值（若存在）：

    // ?name=tobi
    req.param('name')
    // => "tobi"

    // POST name=tobi
    req.param('name')
    // => "tobi"

    // /user/tobi for /user/:name 
    req.param('name')
    // => "tobi"

查找顺序：

- req.params
- req.body
- req.query

建议直接访问`req.body`, `req.params`, `req.query`，这样更清楚。

### req.route

The currently matched Route containing several properties such as the route's original path string, the regexp generated, and so on.

    app.get('/user/:id?', function(req, res){
      console.log(req.route);
    });

Example output from the previous snippet:

    { path: '/user/:id?',
      keys: [ { name: 'id', optional: true } ],
      regexp: /^\/user(?:\/([^\/]+?))?\/?$/i,
      params: [ id: '12' ] }

### req.cookies

When the `cookieParser()` middleware is used this object defaults to `{}`, otherwise contains the cookies sent by the user-agent.

    // Cookie: name=tj
    req.cookies.name
    // => "tj"

Please refer to [cookie-parser](https://github.com/expressjs/cookie-parser) for additional documentation or any issues and concerns.

### req.signedCookies

When the `cookieParser(secret)` middleware is used this object defaults to `{}`, otherwise contains the signed cookies sent by the user-agent, unsigned and ready for use. Signed cookies reside in a different object to show developer intent, otherwise a malicious attack could be placed on `req.cookie` values which are easy to spoof. Note that signing a cookie does not mean it is "hidden" nor encrypted, this simply prevents tampering as the secret used to sign is private.

    // Cookie: user=tobi.CP7AWaXDfAKIRfH49dQzKJx7sKzzSoPq7/AcBBRVwlI3
    req.signedCookies.user
    // => "tobi"

Please refer to [cookie-parser](https://github.com/expressjs/cookie-parser) for additional documentation or any issues and concerns.

### req.get(field)

查询请求头字段。大小写不敏感。The Referrer and Referer fields are interchangeable.

    req.get('Content-Type');
    // => "text/plain"

    req.get('content-type');
    // => "text/plain"

    req.get('Something');
    // => undefined

是`req.header(field)`的别名。

### req.accepts(types)

Check if the given types are acceptable, returning the best match when true, otherwise undefined - in which case you should respond with 406 "Not Acceptable".

The type value may be a single mime type string such as "application/json", the extension name such as "json", a comma-delimited list or an array. When a list or array is given the best match, if any is returned.

    // Accept: text/html
    req.accepts('html');
    // => "html"

    // Accept: text/*, application/json
    req.accepts('html');
    // => "html"
    req.accepts('text/html');
    // => "text/html"
    req.accepts('json, text');
    // => "json"
    req.accepts('application/json');
    // => "application/json"

    // Accept: text/*, application/json
    req.accepts('image/png');
    req.accepts('png');
    // => undefined

    // Accept: text/*;q=.5, application/json
    req.accepts(['html', 'json']);
    req.accepts('html, json');
    // => "json"

Please refer to [accepts](https://github.com/expressjs/accepts) for additional documentation or any issues and concerns.

### req.acceptsCharset(charset)

Check if the given charset are acceptable.

Please refer to [accepts](https://github.com/expressjs/accepts) for additional documentation or any issues and concerns.

### req.acceptsLanguage(lang)

Check if the given lang are acceptable.

Please refer to [accepts](https://github.com/expressjs/accepts) for additional documentation or any issues and concerns.

### req.is(type)

Check if the incoming request contains the "Content-Type" header field, and it matches the give mime type.

    // With Content-Type: text/html; charset=utf-8
    req.is('html');
    req.is('text/html');
    req.is('text/*');
    // => true

    // When Content-Type is application/json
    req.is('json');
    req.is('application/json');
    req.is('application/*');
    // => true

    req.is('html');
    // => false

Please refer to [type-is](https://github.com/expressjs/type-is) for additional documentation or any issues and concerns.

### req.ip

Return the remote address, or when "trust proxy" is enabled - the upstream address.

    req.ip
    // => "127.0.0.1"

### req.ips

When "trust proxy" is `true`, parse the "X-Forwarded-For" ip address list and return an array, otherwise an empty array is returned. For example if the value were "client, proxy1, proxy2" you would receive the array `["client", "proxy1", "proxy2"]` where "proxy2" is the furthest down-stream.

### req.path

Returns the request URL pathname.

    // example.com/users?sort=desc
    req.path
    // => "/users"

### req.host

Returns the hostname from the "Host" header field (void of portno).

    // Host: "example.com:3000"
    req.host
    // => "example.com"

### req.fresh

Check if the request is fresh - aka Last-Modified and/or the ETag still match, indicating that the resource is "fresh".

    req.fresh
    // => true

Please refer to [fresh](https://github.com/visionmedia/node-fresh) for additional documentation or any issues and concerns.

### req.stale

Check if the request is stale - aka Last-Modified and/or the ETag do not match, indicating that the resource is "stale".

    req.stale
    // => true

### req.xhr

Check if the request was issued with the "X-Requested-With" header field set to "XMLHttpRequest" (jQuery etc).

    req.xhr
    // => true

### req.protocol

Return the protocol string "http" or "https" when requested with TLS. When the "trust proxy" setting is enabled the "X-Forwarded-Proto" header field will be trusted. If you're running behind a reverse proxy that supplies https for you this may be enabled.

    req.protocol
    // => "http"

### req.secure

Check if a TLS connection is established. This is a short-hand for:

	'https' == req.protocol;

### req.subdomains

Return subdomains as an array.

    // Host: "tobi.ferrets.example.com"
    req.subdomains
    // => ["ferrets", "tobi"]

### req.originalUrl

This property is much like `req.url`, however it retains the original request url, allowing you to rewrite `req.url` freely for internal routing purposes. For example the "mounting" feature of `app.use()` will rewrite req.url to strip the mount point.

    // GET /search?q=something
    req.originalUrl
    // => "/search?q=something"

## 响应

### res.status(code)

Chainable alias of node's `res.statusCode=`.

	res.status(404).sendFile('path/to/404.png');

如果只想发送返回码，没有响应体，在`status()`后调用`end()`：

	res.status(201).end();

### res.set(field, [value])

Set header field to value, or pass an object to set multiple fields at once.

    res.set('Content-Type', 'text/plain');

    res.set({
      'Content-Type': 'text/plain',
      'Content-Length': '123',
      'ETag': '12345'
    })

Aliased as `res.header(field, [value])`.

### res.get(field)

Get the case-insensitive response header field.

    res.get('Content-Type');
    // => "text/plain"

### res.cookie(name, value, [options])

Set cookie name to value, which may be a string or object converted to JSON. The path option defaults to "/".

    res.cookie('name', 'tobi', { domain: '.example.com', path: '/admin', secure: true });
    res.cookie('rememberme', '1', { expires: new Date(Date.now() + 900000), httpOnly: true });

The `maxAge` option is a convenience option for setting "expires" relative to the current time in milliseconds. The following is equivalent to the previous example.

	res.cookie('rememberme', '1', { maxAge: 900000, httpOnly: true })

An object may be passed which is then serialized as JSON, which is automatically parsed by the bodyParser() middleware.

    res.cookie('cart', { items: [1,2,3] });
    res.cookie('cart', { items: [1,2,3] }, { maxAge: 900000 });

Signed cookies are also supported through this method. Simply pass the signed option. When given `res.cookie()` will use the secret passed to `cookieParser(secret)` to sign the value.

	res.cookie('name', 'tobi', { signed: true });

Later you may access this value through the req.signedCookie object.

### res.clearCookie(name, [options])

Clear cookie name. The path option defaults to "/".

    res.cookie('name', 'tobi', { path: '/admin' });
    res.clearCookie('name', { path: '/admin' });

### res.redirect([status], url)

重定向到给定的URL。状态码可选，默认为302 "Found"。

    res.redirect('/foo/bar');
    res.redirect('http://example.com');
    res.redirect(301, 'http://example.com');
    res.redirect('../login');

Express supports a few forms of redirection, first being a fully qualified URI for redirecting to a different site:

	res.redirect('http://google.com');

The second form is the pathname-relative redirect, for example if you were on `http://example.com/admin/post/new`, the following redirect to `/admin` would land you at h`ttp://example.com/admin`:

	res.redirect('/admin');

This next redirect is relative to the mount point of the application. For example if you have a blog application mounted at `/blog`, ideally it has no knowledge of where it was mounted, so where a redirect of `/admin/post/new` would simply give you `http://example.com/admin/post/new`, the following mount-relative redirect would give you `http://example.com/blog/admin/post/new`:

	res.redirect('admin/post/new');

Pathname relative redirects are also possible. If you were on `http://example.com/admin/post/new`, the following redirect would land you at `http//example.com/admin/post`:

	res.redirect('..');

The final special-case is a back redirect, redirecting back to the Referer (or Referrer), defaulting to `/` when missing.

	res.redirect('back');

### res.location

Set the location header.

    res.location('/foo/bar');
    res.location('foo/bar');
    res.location('http://example.com');
    res.location('../login');
    res.location('back');

You can use the same kind of urls as in `res.redirect()`.

For example, if your application is mounted at `/blog`, the following would set the location header to `/blog/admin`:

	res.location('admin')

### res.send([body|status], [body])

Send a response.

    res.send(new Buffer('whoop'));
    res.send({ some: 'json' });
    res.send('some html');
    res.send(404, 'Sorry, we cannot find that!');
    res.send(500, { error: 'something blew up' });
    res.send(200);

This method performs a myriad of useful tasks for simple non-streaming responses such as automatically assigning the `Content-Length` unless previously defined and providing automatic HEAD and HTTP cache freshness support.

When a Buffer is given the `Content-Type` is set to "application/octet-stream" unless previously defined as shown below:

    res.set('Content-Type', 'text/html');
    res.send(new Buffer('some html'));

When a String is given the `Content-Type` is set defaulted to "text/html":

	res.send('some html');

When an Array or Object is given Express will respond with the JSON representation:

    res.send({ user: 'tobi' })
    res.send([1,2,3])

Finally when a Number is given without any of the previously mentioned bodies, then a response body string is assigned for you. For example 200 will respond will the text "OK", and 404 "Not Found" and so on.

    res.send(200)
    res.send(404)
    res.send(500)

### res.json([status|body], [body])

发送一个JSON响应。This method is identical to `res.send()` when an object or array is passed, however it may be used for explicit JSON conversion of non-objects (null, undefined, etc), though these are technically not valid JSON.

    res.json(null)
    res.json({ user: 'tobi' })
    res.json(500, { error: 'message' })

### res.jsonp([status|body], [body])

Send a JSON response with JSONP support. This method is identical to res.json() however opts-in to JSONP callback support.

    res.jsonp(null)
    // => null

    res.jsonp({ user: 'tobi' })
    // => { "user": "tobi" }

    res.jsonp(500, { error: 'message' })
    // => { "error": "message" }

By default the JSONP callback name is simply `callback`, however you may alter this with the [jsonp callback name](http://expressjs.com/4x/api.html#app-settings) setting. The following are some examples of JSONP responses using the same code:

    // ?callback=foo
    res.jsonp({ user: 'tobi' })
    // => foo({ "user": "tobi" })

    app.set('jsonp callback name', 'cb');

    // ?cb=foo
    res.jsonp(500, { error: 'message' })
    // => foo({ "error": "message" })

### res.type(type)

Sets the `Content-Type` to the mime lookup of type, or when "/" is present the `Content-Type` is simply set to this literal value.

    res.type('.html');
    res.type('html');
    res.type('json');
    res.type('application/json');
    res.type('png');

### res.format(object)

Performs content-negotiation on the request Accept header field when present. This method uses `req.accepted`, an array of acceptable types ordered by their quality values, otherwise the first callback is invoked. When no match is performed the server responds with 406 "Not Acceptable", or invokes the default callback.

The Content-Type is set for you when a callback is selected, however you may alter this within the callback using `res.set()` or `res.type()` etcetera.

The following example would respond with `{ "message": "hey" }` when the Accept header field is set to "application/json" or "*/json", however if "*/*" is given then "hey" will be the response.

    res.format({
      'text/plain': function(){
        res.send('hey');
      },

      'text/html': function(){
        res.send('hey');
      },

      'application/json': function(){
        res.send({ message: 'hey' });
      }
    });

In addition to canonicalized MIME types you may also use extnames mapped to these types, providing a slightly less verbose implementation:

    res.format({
      text: function(){
        res.send('hey');
      },

      html: function(){
        res.send('hey');
      },

      json: function(){
        res.send({ message: 'hey' });
      }
    });

### res.sendFile(path, [options], [fn])

Transfer the file at the given path.

Automatically defaults the Content-Type response header field based on the filename's extension. The callback `fn(err)` is invoked when the transfer is complete or when an error occurs.

Options:

- `maxAge` in milliseconds defaulting to 0
- `root` root directory for relative filenames

This method provides fine-grained support for file serving as illustrated in the following example:

    app.get('/user/:uid/photos/:file', function(req, res){
      var uid = req.params.uid
        , file = req.params.file;

      req.user.mayViewFilesFrom(uid, function(yes){
        if (yes) {
          res.sendFile('/uploads/' + uid + '/' + file);
        } else {
          res.send(403, 'Sorry! you cant see that.');
        }
      });
    });

Please refer to [send](https://github.com/visionmedia/send) for additional documentation or any issues and concerns.

### res.download(path, [filename], [fn])

Transfer the file at path as an "attachment", typically browsers will prompt the user for download. The Content-Disposition "filename=" parameter, aka the one that will appear in the brower dialog is set to path by default, however you may provide an override filename.

When an error has ocurred or transfer is complete the optional callback fn is invoked. This method uses `res.sendFile()` to transfer the file.

    res.download('/report-12345.pdf');
    res.download('/report-12345.pdf', 'report.pdf');
    res.download('/report-12345.pdf', 'report.pdf', function(err){
      if (err) {
        // handle error, keep in mind the response may be partially-sent
        // so check res.headersSent
      } else {
        // decrement a download credit etc
      }
    });

### res.links(links)

Join the given links to populate the "Link" response header field.

    res.links({
      next: 'http://api.example.com/users?page=2',
      last: 'http://api.example.com/users?page=5'
    });

yields:

    Link: <http://api.example.com/users?page=2>; rel="next",
          <http://api.example.com/users?page=5>; rel="last"

### res.locals

Response local variables are scoped to the request, thus only available to the **view**(s) rendered during that request / response cycle, if any. Otherwise this API is identical to app.locals.

This object is useful for exposing request-level information such as the request pathname, authenticated user, user settings etcetera.

    app.use(function(req, res, next){
      res.locals.user = req.user;
      res.locals.authenticated = ! req.user.anonymous;
      next();
    });

### res.render(view, [locals], callback)

Render a view with a callback responding with the rendered string. When an error occurs `next(err)` is invoked internally. When a callback is provided both the possible error and rendered string are passed, and no automated response is performed.

    res.render('index', function(err, html){
      // ...
    });

    res.render('user', { name: 'Tobi' }, function(err, html){
      // ...
    });

## 路由










