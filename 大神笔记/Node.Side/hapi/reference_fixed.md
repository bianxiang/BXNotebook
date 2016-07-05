# 1. `Hapi.Server`

## 1.1 `new Server([host], [port], [options])`

创建新实例。

- `host` - the hostname, IP address, or path to UNIX domain socket the server is bound to. 默认是`0.0.0.0`， which means any available network interface. Set to `127.0.0.1` or `localhost` to restrict connection to those coming from the same machine. If `host` contains a '/' character, it is used as a UNIX domain socket path and if it starts with '\\.\pipe' as a Windows named pipe.
- `port` - 监听的端口。HTTP默认为`80`，TLS默认为`443`。
	To use an ephemeral port, use `0` and once the server is started, retrieve the port allocation via `server.info.port`.
- `options` - An object with the server configuration as described in [server options](#server-options).

```javascript
var Hapi = require('hapi');
var server = new Hapi.Server('localhost', 8000, { cors: true });
```
## 1.2 `createServer([host], [port], [options])`

创建服务器实例的另一种方法，参数与`new Server()`相同。

```javascript
var Hapi = require('hapi');
var server = Hapi.createServer('localhost', 8000, { cors: true });
```

## 1.3 Server选项

### `app`

应用特定配置，可以通过`server.settings.app`访问。Should not be used by plugins which should use `plugins[name]`. 注意区别：`server.settings.app`用于存储配置，而`server.app`存放运行时状态。

### `cache`

<a name="server.config.cache"></a>

指定服务器端缓存类型。每个服务器都有一个默认缓存，用于存储和服务器状态。默认缓存基于内存。**hapi** uses [**catbox** module documentation](https://github.com/spumko/catbox#client) as its cache implementation which includes support for Redis, MongoDB,  Memcached, and Riak. Caching is only utilized if methods and plugins explicitly store their state in the cache. The server cache configuration only defines the store itself. `cache` 取值：
- 一个字符串，表示缓存引擎模块名(e.g. `'catbox-memory'`, `'catbox-redis'`).
- 一个配置对象，选项如下：
    - `engine` - 缓存选项。The cache options are described in the [**catbox** module documentation](https://github.com/spumko/catbox#client). When an array  of options is provided, multiple cache connections are established and each array item (except one) must include an additional option:
- `name` - an identifier used later when provisioning or configuring caching for routes, methods, or plugins. Each connection name must be unique. A single item may omit the `name` option which defines the default cache. If every connection includes a `name`, a default memory cache is provisions as well as the default.
- `shared` - if `true`, allows multiple cache users to share the same segment (e.g. multiple servers in a pack using the same route and cache. Default to not shared.
- an array of the above types for configuring multiple cache instances, each with a unqiue name.

### `cors`

The [Cross-Origin Resource Sharing](http://www.w3.org/TR/cors/) protocol allows browsers to make cross-origin API calls. CORS is required by web applications running inside a browser which are loaded from a different domain than the API server. CORS headers are disabled by default. To enable, set `cors` to `true`, or to an object with the following options:
- `origin` - a strings array of allowed origin servers ('Access-Control-Allow-Origin'). The array can contain any combination of fully qualified origins along with origin strings containing a wilcard `'*'` character, or a single `'*'` origin string. Defaults to any origin `['*']`.
- `isOriginExposed` - if `false`, prevents the server from returning the full list of non-wildcard `origin` values if the incoming origin header does not match any of the values. Has no impact if `matchOrigin` is set to `false`. Defaults to `true`.
- `matchOrigin` - if `false`, returns the list of `origin` values without attempting to match the incoming origin value. Cannot be used with wildcard `origin` values. Defaults to `true`.
- `maxAge` - number of seconds the browser should cache the CORS response ('`Access-Control-Max-Age`'). The greater the value, the longer it will take before the browser checks for changes in policy. Defaults to `86400` (one day).
- `headers` - a strings array of allowed headers ('`Access-Control-Allow-Headers`'). Defaults to `['Authorization', 'Content-Type', 'If-None-Match']`.
- `additionalHeaders` - a strings array of additional headers to `headers`. Use this to keep the default headers in place.
- `methods` - a strings array of allowed HTTP methods ('`Access-Control-Allow-Methods`'). Defaults to `['GET', 'HEAD', 'POST', 'PUT', 'DELETE', 'OPTIONS']`.
- `additionalMethods` - a strings array of additional methods to `methods`. Use this to keep the default methods in place.
- `exposedHeaders` - a strings array of exposed headers ('`Access-Control-Expose-Headers`'). Defaults to `['WWW-Authenticate', 'Server-Authorization']`.
- `additionalExposedHeaders` - a strings array of additional headers to `exposedHeaders`. Use this to keep the default headers in place.
- `credentials` - if `true`, allows user credentials to be sent ('`Access-Control-Allow-Credentials`'). Defaults to `false`.

### `debug`

控制发送到控制台的错误类型：
- `request` - a string array of request log tags to be displayed via `console.error()` when the events are logged via `request.log()`. Defaults to uncaught errors thrown in external code (these errors are handled automatically and result in an Internal Server Error (500) error response) or runtime errors due to incorrect implementation of the hapi API. For example, to display all errors, change the option to `['error']`. 要关闭所有控制台调试消息，设为`false`。

### `files`

<a name="server.config.files"></a>

定义伺服静态资源的行为，使用内建的路由处理器：
- `relativeTo` - determines the folder relative paths are resolved against when using the file and directory handlers.
- `etagsCacheMaxSize` - sets the maximum number of file etag hash values stored in the cache. Defaults to `10000`.

### `json`

optional arguments passed to `JSON.stringify()` when converting an object or error response to a string payload. Supports the following:
- `replacer` - the replacer function or array. Defaults to no action.
- `space` - number of spaces to indent nested object keys. Defaults to no indentation.

### `labels`

a string array of labels used when registering plugins to [`plugin.select()`](#pluginselectlabels) matching server labels. Defaults to an empty array `[]` (no labels).

### `load`

服务器负载监控和限制。(stored under `server.load` when enabled) where:
- `maxHeapUsedBytes` - maximum V8 heap size over which incoming requests are rejected with an HTTP Server Timeout (503) response. Defaults to `0` (no limit).
- `maxRssBytes` - maximum process RSS size over which incoming requests are rejected with an HTTP Server Timeout (503) response. Defaults to `0` (no limit).
- `maxEventLoopDelay` - maximum event loop delay duration in milliseconds over which incoming requests are rejected with an HTTP Server Timeout (503) response. Defaults to `0` (no limit).
- `sampleInterval` - the frequency of sampling in milliseconds. Defaults to `0` (no sampling).

### `location`

<a name="server.config.location"></a>

Used to convert relative 'Location' header URIs to absolute, by adding this value as prefix. Value must not contain a trailing `'/'`. Defaults to the host received in the request HTTP 'Host' header and if missing, to `server.info.uri`.

### `payload`

<a name="server.config.payload"></a>

控制如何处理请求体：
- `maxBytes` - 限制负载的字节大小。允许过大的复杂将导致服务器内存不足。默认为`1048576` (1MB)。
- `uploads` - 用于上传文件的目录。Defaults to `os.tmpDir()`.

### `plugins`

plugin-specific configuration which can later be accessed by `server.plugins`. Provides a place to store and pass plugin configuration that is at server-level. The `plugins` is an object where each key is a plugin name and the value is the configuration. Note the difference between `server.settings.plugins` which is used to store configuration value and `server.plugins` which is meant for storing run-time state.

### `router`

<a name="server.config.router"></a>

Controls how incoming request URIs are matched against the routing table:
- `isCaseSensitive` - determines whether the paths '/example' and '/EXAMPLE' are considered different resources. Defaults to `true`.

### `state`

<a name="server.config.state"></a>

HTTP state management (cookies) allows the server to store information on the client which is sent back to the server with every request (as defined in [RFC 6265](https://tools.ietf.org/html/rfc6265)).
- `cookies` - 根据下面的选项，服务器将自动解析输入的cookies：
  - `parse` - determines if incoming 'Cookie' headers are parsed and stored in the `request.cookies` object. Defaults to `true`.
  - `failAction` - determines how to handle cookie parsing errors. Allowed values are:
   - `'error'` - return a Bad Request (400) error response. This is the default value.
   - `'log'` - report the error but continue processing the request.
   - `'ignore'` - take no action.
  - `clearInvalid` - if `true`, automatically instruct the client to remove invalid cookies. Defaults to `false`.
  - `strictHeader` - if `false`, allows any cookie value including values in violation of [RFC 6265](https://tools.ietf.org/html/rfc6265). Defaults to `true`.

### `timeout`

define timeouts for processing durations:
- `server` - 响应超时，毫秒。Sets the maximum time allowed for the server to respond to an incoming client request before giving up and responding with a Service Unavailable (503) error response. Disabled by default (`false`).
- `client` - 请求超时，毫秒。Sets the maximum time allowed for the client to transmit the request payload (body) before giving up and responding with a Request Timeout (408) error response. Set to `false` to disable. Defaults to `10000` (10 seconds).
- `socket` - by default, node sockets automatically timeout after 2 minutes. Use this option to override this behavior. Defaults to `undefined` which leaves the node default unchanged. Set to `false` to disable socket timeouts.

### `tls`

Used to create an HTTPS server. The `tls` object is passed unchanged as options to the node.js HTTPS server as described in the [node.js HTTPS documentation](http://nodejs.org/api/https.html#https_https_createserver_options_requestlistener).

### `maxSockets`

Sets the number of sockets available per outgoing proxy host connection. `false` means use node.js default value. Does not affect non-proxy outgoing client connections. Defaults to `Infinity`.

### `validation`

Options to pass to [Joi](http://github.com/spumko/joi). Useful to set global options such as `stripUnknown` or `abortEarly` (the complete list is available [here](https://github.com/spumko/joi#validatevalue-schema-options)). Defaults to `{ modify: true }` which will cast data to the specified types.

### `views`

<a name="server.config.views"></a>

启用视图渲染的支持。默认禁用。若要启用，传入带有以下选项的配置对象：
- `engines` - （必需） an object where each key is a file extension (e.g. 'html', 'jade'), mapped to the npm module name (string) used for rendering the templates. Alternatively, the extension can be mapped to an object with the following options:
  - `module` - the npm module name (string) to require or an object with:
   - `compile()` - the rendering function. The required function signature depends on the `compileMode` settings. If the `compileMode` is `'sync'`, the signature is `compile(template, options)`, the return value is a function with signature `function(context, options)`, and the method is allowed to throw errors. If the `compileMode` is `'async'`, the signature is `compile(template, options, callback)` where `callback` has the signature `function(err, compiled)` where `compiled` is a function with signature `function(context, options, callback)` and `callback` has the signature `function(err, rendered)`.
  - any of the `views` options listed below (except `defaultExtension`) to override the defaults for a specific engine.
- `defaultExtension` - defines the default filename extension to append to template names when multiple engines are configured and not explicit extension is provided for a given template. No default value.
- `path` - the root file path used to resolve and load the templates identified when calling `reply.view()`. Defaults to current working directory.
- `partialsPath` - the root file path where partials are located. Partials are small segments of template code that can be nested and reused throughout other templates. Defaults to no partials support (empty path).
- `helpersPath` - the directory path where helpers are located. Helpers are functions used within templates to perform transformations and other data manipulations using the template context or other inputs. Each '.js' file in the helpers directory is loaded and the file name is used as the helper name. The files must export a single method with the signature `function(context)` and return a string. Sub-folders are not supported and are ignored. Defaults to no helpers support (empty path).
- `basePath` - a base path used as prefix for `path` and `partialsPath`. No default.
- `layout` - if set to `true` or a layout filename, layout support is enabled. A layout is a single template file used as the parent template for other view templates in the same engine. If `true`, the layout template name must be 'layout.ext' where 'ext' is the engine's extension. Otherwise, the provided filename is suffixed with the engine's extension and laoded. Disable `layout` when using Jade as it will handle including any layout files independently. Defaults to `false`.
- `layoutPath` - the root file path where layout templates are located (relative to `basePath` if present). Defaults to `path`.
- `layoutKeyword` - the key used by the template engine to denote where primary template content should go. Defaults to `'content'`.
- `encoding` - the text encoding used by the templates when reading the files and outputting the result. Defaults to `'utf8'`.
- `isCached` - if set to `false`, templates will not be cached (thus will be read from file on every use). Defaults to `true`.
- `allowAbsolutePaths` - if set to `true`, allows absolute template paths passed to `reply.view()`. Defaults to `false`.
- `allowInsecureAccess` - if set to `true`, allows template paths passed to `reply.view()` to contain '../'. Defaults to `false`.
- `compileOptions` - options object passed to the engine's compile function. Defaults to empty options `{}`.
- `runtimeOptions` - options object passed to the returned function from the compile operation. Defaults to empty options `{}`.
- `contentType` - the content type of the engine results. Defaults to `'text/html'`.
- `compileMode` - specify whether the engine `compile()` method is `'sync'` or `'async'`. Defaults to `'sync'`.

## 1.4 `Server`属性

每个`Server`实例都必须有以下属性：
- `app` - 应用特定的状态。Provides a safe place to store application data without potential conflicts with **hapi**. Should not be used by plugins which should use `plugins[name]`.
- `methods` - methods registered with [`server.method()`](#servermethodname-fn-options).
- `info` - 服务器信息：
  - `port` - the port the server was configured to (before `start()`) or bound to (after `start()`).
  - `host` - the hostname the server was configured to (defaults to `'0.0.0.0'` if no host was provided).
  - `protocol` - the protocol used (e.g. `'http'` or `'https'`).
  - `uri` - a string with the following format: 'protocol://host:port' (e.g. 'http://example.com:8080').
- `listener` - the node HTTP server object.
- `load` - 服务器负载度量 (when `server.load.sampleInterval` is enabled):
  - `eventLoopDelay` - event loop delay milliseconds.
  - `heapUsed` - V8 heap usage.
  - `rss` - RSS memory usage.
- `pack` - the [`Pack`](#hapipack) object the server belongs to (automatically assigned when creating a server instance directly).
- `plugins` - an object where each key is a plugin name and the value are the exposed properties by that plugin using [`plugin.expose()`](#pluginexposekey-value).
- `settings` - an object containing the [server options](#server-options) after applying the defaults.

## 1.5 `Server`方法

### 1.5.1 `server.start([callback])`

启动服务器，监听特定端口。If provided, `callback()` is called once the server is ready for new connections. If the server is already started, the `callback()` is called on the next tick.

```javascript
var Hapi = require('hapi');
var server = new Hapi.Server();
server.start(function () {
    console.log('Server started at: ' + server.info.uri);
});
```
### 1.5.2 `server.stop([options], [callback])`

停止服务器，拒绝新连接。已存在的连接将继续知道被关闭或超时（默认5秒）。服务器停止后，所有的连接结束后，进程可以安全退出，回调方法（如果有）将被调用。If the server is already stopped, the `callback()` is called on the next tick.

可选的`options`对象支持：

- `timeout` - overrides the timeout in millisecond before forcefully terminating a connection. Defaults to `5000` (5 seconds).

```javascript
server.stop({ timeout: 60 * 1000 }, function () {
    console.log('Server stopped');
});
```

### 1.5.3 `server.route(options)`

向服务器加新路由。

- `options` - 配置项，见[route options](#route-options)。

#### 1.5.3.1 路由选项

##### `path`

（必需）绝对路径，用于匹配请求（必须以'/'开头）。Incoming requests are compared to the configured paths based on the server [`router`](#server.config.router) configuration option. 路径中可以有命名参数，在`{}`中，参见[Path parameters](#path-parameters)。

##### `method`

（必需） HTTP方法。一般是'GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'。Any HTTP method is allowed, except for 'HEAD'. Use `'*'` to match against any HTTP method (only when an exact match was not found, and any match with a specific method will be given a higher priority over a wildcard match). Can be assigned an array of methods which has the same result as adding the same route with different methods manually.

##### `vhost`

an optional domain string or an array of domain strings for limiting the route to only requests with a matching host header field. Matching is done against the hostname part of the header only (excluding the port). Defaults to all hosts.

##### `handler`
（必需） 在校验和身份验证后，调用响应函数。参见[Route handler](#route-handler)。If set to a string, the value is parsed the same way a prerequisite server method string shortcut is processed. `handler`还可以赋予以下几种对象：

###### `file`

<a name="route.config.file"></a>

Generates a static file endpoint for serving a single file. `file` can be set to:
- 相对或绝对路径字符串(relative paths are resolved based on the server [`files`](#server.config.files) configuration).
- 一个函数，签名是`function(request)`，返回一个相对或绝对的文件路径
- 一个对象，包含以下选项：
  - `path` - 上面讨论的路径字符串或函数
  - `filename` - 如果'`Content-Disposition`'头时指定的名字，默认为`path`的basename。
  - `mode` - 指定响应时是否包含'`Content-Disposition`'头。取值：
 	- `false` - 不发送这个头。只是默认值。
   - `'attachment'`
   - `'inline'`
  - `lookupCompressed` - if `true`, looks for the same filename with the '.gz' suffix for a precompressed version of the file to serve if the request supports content encoding. Defaults to `false`.

###### `directory`

<a name="route.config.directory"></a>

generates a directory endpoint for serving static content from a directory. Routes using the directory handler must include a path parameter at the end of the path string (e.g. '`/path/to/somewhere/{param}`' where the parameter name does not matter). The path parameter can use any of the parameter options (e.g. '`{param}`' for one level files only, '`{param?}`' for one level files or the directory root, '`{param*}`' for any level, or '`{param*3}`' for a specific level). If additional path parameters are present, they are ignored for the purpose of selecting the file system resource. The directory handler is an object with the following options:
- `path` - (required) the directory root path (relative paths are resolved based on the server [`files`](#server.config.files) configuration). Value can be:
  - a single path string used as the prefix for any resources requested by appending the request path parameter to the provided string.
  - an array of path strings. Each path will be attempted in order until a match is found (by following the same process as the single path string).
  - a function with the signature `function(request)` which returns the path string.
- `index` - optional boolean, determines if 'index.html' will be served if found in the folder when requesting a directory. Defaults to `true`.
- `listing` - optional boolean, determines if directory listing is generated when a directory is requested without an index document. Defaults to `false`.
- `showHidden` - optional boolean, determines if hidden files will be shown and served. Defaults to `false`.
- `redirectToSlash` - optional boolean, determines if requests for a directory without a trailing slash are redirected to the same path with the missing slash. Useful for ensuring relative links inside the response are resolved correctly. Defaults to `true`.
- `lookupCompressed` - optional boolean, instructs the file processor to look for the same filename with the '.gz' suffix for a precompressed version of the file to serve if the request supports content encoding. Defaults to `false`.
- `defaultExtension` - optional string, appended to file requests if the requested file is not found. Defaults to no extension.

###### `proxy`

<a name="route.config.proxy"></a>

generates a reverse proxy handler with the following options:
- `host` - the upstream service host to proxy requests to.  The same path on the client request will be used as the path on the host.
- `port` - the upstream service port.
- `protocol` - The protocol to use when making a request to the proxied host:
  - `'http'`
  - `'https'`
- `uri` - an absolute URI used instead of the incoming host, port, protocol, path, and query. Cannot be used with `host`, `port`, `protocol`, or `mapUri`.
- `passThrough` - if `true`, forwards the headers sent from the client to the upstream service being proxied to. Defaults to `false`.
- `rejectUnauthorized` - sets the `rejectUnauthorized` property on the https [agent](http://nodejs.org/api/https.html#https_https_request_options_callback) making the request. This value is only used when the proxied server uses TLS/SSL.  When set it will override the node.js `rejectUnauthorized` property. If `false` then ssl errors will be ignored. When `true` the server certificate is verified and an 500 response will be sent when verification fails. Defaults to the https agent default value of `true`.
- `xforward` - if `true`, sets the 'X-Forwarded-For', 'X-Forwarded-Port', 'X-Forwarded-Proto' headers when making a request to the proxied upstream endpoint. Defaults to `false`.
- `redirects` - the maximum number of HTTP redirections allowed, to be followed automatically by the handler. Set to `false` or `0` to disable all redirections (the response will contain the redirection received from the upstream service). If redirections are enabled, no redirections (301, 302, 307, 308) will be passed along to the client, and reaching the maximum allowed redirections will return an error response. Defaults to `false`.
- `timeout` - number of milliseconds before aborting the upstream request. Defaults to `180000` (3 minutes).
- `mapUri` - a function used to map the request URI to the proxied URI. Cannot be used together with `host`, `port`, `protocol`, or `uri`. The function signature is `function(request, callback)` where:
  - `request` - is the incoming `request` object
  - `callback` - is `function(err, uri, headers)` where:
   - `err` - internal error condition.
   - `uri` - the absolute proxy URI.
   - `headers` - optional object where each key is an HTTP request header and the value is the header content.
- `onResponse` - a custom function for processing the response from the upstream service before sending to the client. Useful for custom error handling of responses from the proxied endpoint or other payload manipulation. Function signature is `function(err, res, request, reply, settings, ttl)` where:
  - `err` - internal or upstream error returned from attempting to contact the upstream proxy.
  - `res` - the node response object received from the upstream service. `res` is a readable stream (use the [**nipple**](https://github.com/spumko/nipple) module `read` method to easily convert it to a Buffer or string).
  - `request` - is the incoming `request` object.
  - `reply()` - the continuation function.
  - `settings` - the proxy handler configuration.
  - `ttl` - the upstream TTL in milliseconds if `proxy.ttl` it set to `'upstream'` and the upstream response included a valid 'Cache-Control' header with 'max-age'.
  - `ttl` - if set to `'upstream'`, applies the upstream response caching policy to the response using the `response.ttl()` method (or passed as an argument to the `postResponse` method if provided).

###### `view`

<a name="route.config.view"></a>

generates a template-based response. The `view` option can be set to one of:
- a string with the template file name.
- an object with the following keys:
  - `template` - a string with the template file name.
  - `context` - an optional template context object. Defaults to an object with the following key:
   - `payload` - maps to `request.payload`.
   - `params` - maps to `request.params`.
   - `query` - maps to `request.query`.
   - `pre` - maps to `request.pre`.

##### `config`

附加路由配置(the `config` options allows splitting the route information from its implementation):

##### `handler`

n alternative location for the route handler function. Same as the `handler` option in the parent level. Can only include one handler per route.

##### `bind`

An object passed back to the provided handler (via `this`) when called.

##### `app`

Application-specific configuration. Provides a safe place to pass application configuration without potential conflicts with **hapi**. Should not be used by plugins which should use `plugins[name]`.

##### `plugins`

plugin-specific configuration. Provides a place to pass route-level plugin configuration. The `plugins` is an object where each key is a plugin name and the value is the state.

##### `pre`

an array with prerequisites methods which are executed in serial or in parallel before the handler is called and are described in [Route prerequisites](#route-prerequisites).

##### `validate`

然后：
- `query` - validation rules for an incoming request URI query component (the key-value part of the URI between '?' and '#'). The query is parsed into its individual key-value pairs (see [Query String](http://nodejs.org/api/querystring.html#querystring_querystring_parse_str_sep_eq_options)) and stored in `request.query` prior to validation. Values allowed:
  - `true` - any query parameters allowed (no validation performed). This is the default.
  - `false` - no query parameters allowed.
  - a [Joi](http://github.com/spumko/joi) validation object.
  - a validation function using the signature `function(value, options, next)` where:
   - `value` - the object containing the query parameters.
   - `options` - the server validation options.
   - `next(err)` - the callback function called when validation is completed.


