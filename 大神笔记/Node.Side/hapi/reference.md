


        - `payload` - validation rules for an incoming request payload (request body). Values allowed:
            - `true` - any payload allowed (no validation performed). This is the default.
            - `false` - no payload allowed.
            - a [Joi](http://github.com/spumko/joi) validation object.
            - a validation function using the signature `function(value, options, next)` where:
                - `value` - the object containing the payload object.
                - `options` - the server validation options.
                - `next(err)` - the callback function called when validation is completed.

        - `path` - validation rules for incoming request path parameters, after matching the path against the route and extracting any
          parameters then stored in `request.params`. Values allowed:
            - `true` - any path parameters allowed (no validation performed).  This is the default.
            - `false` - no path variables allowed.
            - a [Joi](http://github.com/spumko/joi) validation object.
            - a validation function using the signature `function(value, options, next)` where:
                - `value` - the object containing the path parameters.
                - `options` - the server validation options.
                - `next(err)` - the callback function called when validation is completed.

        - `errorFields` - an optional object with error fields copied into every validation error response.
        - `failAction` - determines how to handle invalid requests. Allowed values are:
            - `'error'` - return a Bad Request (400) error response. This is the default value.
            - `'log'` - log the error but continue processing the request.
            - `'ignore'` - take no action.
            - a custom error handler function with the signature `functon(source, error, next)` where:
                - `source` - the source of the invalid field (e.g. 'path', 'query', 'payload').
                - `error` - the error object prepared for the client response (including the validation function error under `error.data`).
                - `next` - the continuation method called to resume route processing or return an error response. The function signature
                  is `function(exit)` where:
                    - `exit` - optional client response. If set to a non-falsy value, the request lifecycle process will jump to the
                      "send response" step, skipping all other steps in between, and using the `exit` value as the new response. `exit` can
                      be any result value accepted by [`reply()`](#replyresult).

    - `payload` - determines how the request payload is processed:
        - `output` - the type of payload representation requested where:
            - `data` - the incoming payload is ready fully into memory. If `parse` is `true`, the payload is parsed (JSON, form-decoded,
              multipart) based on the 'Content-Type' header. If `parse` is false, the raw `Buffer` is returned. This is the default value
              except when a proxy handler is used.
            - `stream` - the incoming payload is made available via a `Stream.Readable` interface. If the payload is 'multipart/form-data' and
              `parse` is `true`, fields values are presented as text while files are provided as streams. File streams from a
              'multipart/form-data' upload will also have a property `.hapi` containing `filename` and `headers` properties.
            - `file` - the incoming payload in written to temporary file in the directory specified by the server's `payload.uploads` settings.
              If the payload is 'multipart/form-data' and `parse` is `true`, fields values are presented as text while files are saved.
        - `parse` - can be `true`, `false`, or `gunzip`; determines if the incoming payload is processed or presented raw. `true` and `gunzip`
          includes gunzipping when the appropriate 'Content-Encoding' is specified on the received request. If parsing is enabled and the
          'Content-Type' is known (for the whole payload as well as parts), the payload is converted into an object when possible. If the
          format is unknown, a Bad Request (400) error response is sent. Defaults to `true`, except when a proxy handler is used. The
          supported mime types are:
            - 'application/json'
            - 'application/x-www-form-urlencoded'
            - 'application/octet-stream'
            - 'text/*'
            - 'multipart/form-data'
        - `allow` - a string or an array of strings with the allowed mime types for the endpoint. Defaults to any of the supported mime types listed
          above. Note that allowing other mime types not listed will not enable them to be parsed, and that if parsing mode is `'parse'`, the request
          will result in an error response.
        - `override` - a mime type string overriding the 'Content-Type' header value received. Defaults to no override.
        - `maxBytes` - overrides the server [default value](#server.config.payload) for this route.
        - `uploads` - overrides the server [default value](#server.config.payload) for this route.
        - `failAction` - determines how to handle payload parsing errors. Allowed values are:
            - `'error'` - return a Bad Request (400) error response. This is the default value.
            - `'log'` - report the error but continue processing the request.
            - `'ignore'` - take no action and continue processing the request.

    - `response` - validation rules for the outgoing response payload (response body). Can only validate [object](#obj) response. Values allowed:
        - `true` - any payload allowed (no validation performed). This is the default.
        - `false` - no payload allowed.
        - an object with the following options:
            - `schema` - the response object validation rules expressed as one of:
                - a [Joi](http://github.com/spumko/joi) validation object.
                - a validation function using the signature `function(value, options, next)` where:
                    - `value` - the object containing the response object.
                    - `options` - the server validation options.
                    - `next(err)` - the callback function called when validation is completed.
            - `sample` - the percent of responses validated (0 - 100). Set to `0` to disable all validation. Defaults to `100` (all responses).
            - `failAction` - defines what to do when a response fails validation. Options are:
                - `error` - return an Internal Server Error (500) error response. This is the default value.
                - `log` - log the error but send the response.

    - `cache` - if the route method is 'GET', the route can be configured to include caching directives in the response using the following options:
        - `privacy` - determines the privacy flag included in client-side caching using the 'Cache-Control' header. Values are:
            - `'default'` - no privacy flag. This is the default setting.
            - `'public'` - mark the response as suitable for public caching.
            - `'private'` - mark the response as suitable only for private caching.
        - `expiresIn` - relative expiration expressed in the number of milliseconds since the item was saved in the cache. Cannot be used
          together with `expiresAt`.
        - `expiresAt` - time of day expressed in 24h notation using the 'MM:HH' format, at which point all cache records for the route
          expire. Cannot be used together with `expiresIn`.

    - `auth` - authentication configuration. Value can be:
        - a string with the name of an authentication strategy registered with `server.auth.strategy()`.
        - a boolean where `false` means no authentication, and `true` sets to the default authentication strategy which is available only
          when a single strategy is configured.
        - an object with:
            - `mode` - the authentication mode. Defaults to `'required'` if a server authentication strategy is configured, otherwise defaults
              to no authentication. Available values:
                - `'required'` - authentication is required.
                - `'optional'` - authentication is optional (must be valid if present).
                - `'try'` - same as `'optional'` but allows for invalid authentication.
            - `strategies` - a string array of strategy names in order they should be attempted. If only one strategy is used, `strategy` can
              be used instead with the single string value. Defaults to the default authentication strategy which is available only when a single
              strategy is configured.
            - `payload` - if set, the payload (in requests other than 'GET' and 'HEAD') is authenticated after it is processed. Requires a strategy
              with payload authentication support (e.g. [Hawk](#hawk-authentication)). Available values:
                - `false` - no payload authentication. This is the default value.
                - `'required'` - payload authentication required.
                - `'optional'` - payload authentication performed only when the client includes payload authentication information (e.g.
                  `hash` attribute in Hawk).
            - `tos` - minimum terms-of-service version required (uses the [semver](https://npmjs.org/package/semver) module). If defined, the
              authentication credentials object must include a `tos` key which satisfies this requirement. Defaults to `false` which means no validation.
            - `scope` - the application scope required to access the route. Value can be a scope string or an array of scope strings. The authenticated
              credentials object `scope` property must contain at least one of the scopes defined to access the route. Defaults to no scope required.
            - `entity` - the required authenticated entity type. If set, must match the `entity` value of the authentication credentials. Available
              values:
                - `any` - the authentication can be on behalf of a user or application. This is the default value.
                - `user` - the authentication must be on behalf of a user.
                - `app` - the authentication must be on behalf of an application.

    - `cors` - when `false`, the server's CORS headers are disabled for the route. Defaults to using the server's settings.

    - `jsonp` - enables JSONP support by setting the value to the query parameter name containing the function name used to wrap the response payload.
      For example, if the value is `'callback'`, a request comes in with `'callback=me'`, and the JSON response is `'{ "a":"b" }'`, the payload will be:
      `'me({ "a":"b" });'`. Does not work with stream responses.

    - `description` - route description used for generating documentation (string).
    - `notes` - route notes used for generating documentation (string or array of strings).
    - `tags` - route tags used for generating documentation (array of strings).

```javascript
var Hapi = require('hapi');
var server = new Hapi.Server();

// Handler in top level

var status = function (request, reply) {

    reply('ok');
};

server.route({ method: 'GET', path: '/status', handler: status });

// Handler in config

var user = {
    cache: { expiresIn: 5000 },
    handler: function (request, reply) {

        reply({ name: 'John' });
    }
};

server.route({ method: 'GET', path: '/user', config: user });
```

##### Path processing

The router iterates through the routing table on each incoming request and executes the first (and only the first) matching route. Route
matching is done on the request path only (excluding the query and other URI components). Requests are matches in a deterministic order where
the order in which routes are added does not matter. The routes are sorted from the most specific to the most generic. For example, the following
path array shows the order in which an incoming request path will be matched against the routes:

```javascript
var paths = [
    '/',
    '/a',
    '/b',
    '/ab',
    '/a{p}b',
    '/a{p}',
    '/{p}b',
    '/{p}',
    '/a/b',
    '/a/{p}',
    '/b/',
    '/a1{p}/a',
    '/xx{p}/b',
    '/x{p}/a',
    '/x{p}/b',
    '/y{p}/b',
    '/{p}xx/b',
    '/{p}x/b',
    '/{p}y/b',
    '/a/b/c',
    '/a/b/{p}',
    '/a/d{p}c/b',
    '/a/d{p}/b',
    '/a/{p}d/b',
    '/a/{p}/b',
    '/a/{p}/c',
    '/a/{p*2}',
    '/a/b/c/d',
    '/a/b/{p*2}',
    '/a/{p}/b/{x}',
    '/{p*5}',
    '/a/b/{p*}',
    '/{a}/b/{p*}',
    '/{p*}'
];
```

##### Path parameters

Parameterized paths are processed by matching the named parameters to the content of the incoming request path at that path segment. For example,
'/book/{id}/cover' will match '/book/123/cover' and `request.params.id` will be set to `'123'`. Each path segment (everything between the opening '/' and
 the closing '/' unless it is the end of the path) can only include one named parameter. A parameter can cover the entire segment ('/{param}') or
 part of the segment ('/file.{ext}').

 An optional '?' suffix following the parameter name indicates an optional parameter (only allowed if the parameter is at the ends of the path or
 only covers part of the segment as in '/a{param?}/b'). For example, the route '/book/{id?}' matches '/book/' with the value of `request.params.id` set
 to an empty string `''`.

```javascript
var getAlbum = function (request, reply) {

    reply('You asked for ' +
          (request.params.song ? request.params.song + ' from ' : '') +
          request.params.album);
};

server.route({
    path: '/{album}/{song?}',
    method: 'GET',
    handler: getAlbum
});
```

In addition to the optional `?` suffix, a parameter name can also specify the number of matching segments using the `*` suffix, followed by a number greater than 1. If the number of expected parts can be anything, then use `*` without a number (matching any number of segments can only be used in the
last path segment).

```javascript
var getPerson = function (request, reply) {

    var nameParts = request.params.name.split('/');
    reply({ first: nameParts[0], last: nameParts[1] });
};

server.route({
    path: '/person/{name*2}',   // Matches '/person/john/doe'
    method: 'GET',
    handler: getPerson
});
```

##### Route handler

When a route is matched against an incoming request, the route handler is called and passed a reference to the [request](#request-object) object.
The handler method must call [`reply()`](#replyresult) or one of its sub-methods to return control back to the router.

```javascript
var handler = function (request, reply) {

    reply('success');
};
```

##### Route prerequisites

It is often necessary to perform prerequisite actions before the handler is called (e.g. load required reference data from a database).
The route `pre` option allows defining such pre-handler methods. The methods are called in order. If the `pre` array contains another array,
those methods are called in parallel. `pre` can be assigned a mixed array of:
- arrays containing the elemets listed below, which are executed in parallel.
- objects with:
    - `method` - the function to call (or short-hand method string as described below). the function signature is identical to a route handler
      as describer in [Route handler](#route-handler).
    - `assign` - key name to assign the result of the function to within `request.pre`.
    - `failAction` - determines how to handle errors returned by the method. Allowed values are:
        - `'error'` - returns the error response back to the client. This is the default value.
        - `'log'` - logs the error but continues processing the request. If `assign` is used, the error will be assigned.
        - `'ignore'` - takes no special action. If `assign` is used, the error will be assigned.
- functions - same as including an object with a single `method` key.
- strings - special short-hand notation for [registered server methods](#servermethodname-fn-options) using the format 'name(args)'
  (e.g. `'user(params.id)'`) where:
    - 'name' - the method name. The name is also used as the default value of `assign`.
    - 'args' - the method arguments (excluding `next`) where each argument is a property of `request`.

```javascript
var Hapi = require('hapi');
var server = new Hapi.Server();

var pre1 = function (request, reply) {

    reply('Hello');
};

var pre2 = function (request, reply) {

    reply('World');
};

var pre3 = function (request, reply) {

    reply(request.pre.m1 + ' ' + request.pre.m2);
};

server.route({
    method: 'GET',
    path: '/',
    config: {
        pre: [
            [
                // m1 and m2 executed in parallel
                { method: pre1, assign: 'm1' },
                { method: pre2, assign: 'm2' }
            ],
            { method: pre3, assign: 'm3' },
        ],
        handler: function (request, reply) {

            reply(request.pre.m3 + '\n');
        }
    }
});
```

##### Route not found

If the application needs to override the default Not Found (404) error response, it can add a catch-all route for a specific
method or all methods. Only one catch-all route can be defined per server instance.

```javascript
var Hapi = require('hapi');
var server = new Hapi.Server();

var handler = function (request, reply) {

    reply('The page was not found').code(404);
};

server.route({ method: '*', path: '/{p*}', handler: handler });
```

#### `server.route(routes)`

Same as [server.route(options)](#serverrouteoptions) where `routes` is an array of route options.

```javascript
server.route([
    { method: 'GET', path: '/status', handler: status },
    { method: 'GET', path: '/user', config: user }
]);
```

#### `server.table([host])`

Returns a copy of the routing table where:
- `host` - optional host to filter routes matching a specific virtual host. Defaults to all virtual hosts.

The return value is an array of routes where each route contains:
- `settings` - the route config with defaults applied.
- `method` - the HTTP method in lower case.
- `path` - the route path.

```javascript
var table = server.table()
console.log(table);

/*  Output:

    [{
        method: 'get',
        path: '/test/{p}/end',
        settings: {
            handler: [Function],
            method: 'get',
            plugins: {},
            app: {},
            validate: {},
            payload: { output: 'stream' },
            auth: undefined,
            cache: [Object] }
    }] */
```

#### `server.log(tags, [data, [timestamp]])`

The `server.log()` method is used for logging server events that cannot be associated with a specific request. When called the server emits a `'log'`
event which can be used by other listeners or plugins to record the information or output to the console. The arguments are:

- `tags` - a string or an array of strings (e.g. `['error', 'database', 'read']`) used to identify the event. Tags are used instead of log levels
  and provide a much more expressive mechanism for describing and filtering events. Any logs generated by the server internally include the `'hapi'`
  tag along with event-specific information.
- `data` - an optional message string or object with the application data being logged.
- `timestamp` - an optional timestamp expressed in milliseconds. Defaults to `Date.now()` (now).

```javascript
var Hapi = require('hapi');
var server = new Hapi.Server();

server.on('log', function (event, tags) {

    if (tags.error) {
        console.log(event);
    }
});

server.log(['test', 'error'], 'Test event');
```

#### `server.state(name, [options])`

[HTTP state management](http://tools.ietf.org/html/rfc6265) uses client cookies to persist a state across multiple requests. Cookie definitions
can be registered with the server using the `server.state()` method, where:

- `name` - is the cookie name.
- `options` - are the optional cookie settings:
    - `ttl` - time-to-live in milliseconds. Defaults to `null` (session time-life - cookies are deleted when the browser is closed).
    - `isSecure` - sets the 'Secure' flag. Defaults to `false`.
    - `isHttpOnly` - sets the 'HttpOnly' flag. Defaults to `false`.
    - `path` - the path scope. Defaults to `null` (no path).
    - `domain` - the domain scope. Defaults to `null` (no domain).
    - `autoValue` - if present and the cookie was not received from the client or explicitly set by the route handler, the cookie is automatically
      added to the response with the provided value. The value can be a function with signature `function(request, next)` where:
        - `request` - the request object.
        - `next` - the continuation function using the `function(err, value)` signature.
    - `encoding` - encoding performs on the provided value before serialization. Options are:
        - `'none'` - no encoding. When used, the cookie value must be a string. This is the default value.
        - `'base64'` - string value is encoded using Base64.
        - `'base64json'` - object value is JSON-stringified than encoded using Base64.
        - `'form'` - object value is encoded using the _x-www-form-urlencoded_ method.
        - `'iron'` - Encrypts and sign the value using [**iron**](https://github.com/hueniverse/iron).
    - `sign` - an object used to calculate an HMAC for cookie integrity validation. This does not provide privacy, only a mean to verify that the cookie value
      was generated by the server. Redundant when `'iron'` encoding is used. Options are:
        - `integrity` - algorithm options. Defaults to [`require('iron').defaults.integrity`](https://github.com/hueniverse/iron#options).
        - `password` - password used for HMAC key generation.
    - `password` - password used for `'iron'` encoding.
    - `iron` - options for `'iron'` encoding. Defaults to [`require('iron').defaults`](https://github.com/hueniverse/iron#options).

```javascript
// Set cookie definition

server.state('session', {
    ttl: 24 * 60 * 60 * 1000,     // One day
    isSecure: true,
    path: '/',
    encoding: 'base64json'
});

// Set state in route handler

var handler = function (request, reply) {

    var session = request.state.session;
    if (!session) {
        session = { user: 'joe' };
    }

    session.last = Date.now();

    reply('Success').state('session', session);
};
```

Registered cookies are automatically parsed when received. Parsing rules depends on the server [`state.cookies`](#server.config.state) configuration.
If an incoming registered cookie fails parsing, it is not included in `request.state`, regardless of the `state.cookies.failAction` setting.
When `state.cookies.failAction` is set to `'log'` and an invalid cookie value is received, the server will emit a `'request'` event. To capture these errors
subscribe to the `'request'` events and filter on `'error'` and `'state'` tags:

```javascript
server.on('request', function (request, event, tags) {

    if (tags.error && tags.state) {
        console.error(event);
    }
});
```

#### `server.views(options)`

Initializes the server views manager programmatically instead of via the server [`views`](#server.config.views) configuration option.
The `options` object is the same as the server [`views`](#server.config.views) config object.

```javascript
server.views({
    engines: {
        html: 'handlebars',
        jade: 'jade'
    },
    path: '/static/templates'
});
```

#### `server.cache(name, options)`

Provisions a server cache segment within the common caching facility where:

- `options` - cache configuration as described in [**catbox** module documentation](https://github.com/spumko/catbox#policy):
    - `expiresIn` - relative expiration expressed in the number of milliseconds since the item was saved in the cache. Cannot be used
      together with `expiresAt`.
    - `expiresAt` - time of day expressed in 24h notation using the 'MM:HH' format, at which point all cache records for the route
      expire. Cannot be used together with `expiresIn`.
    - `staleIn` - number of milliseconds to mark an item stored in cache as stale and reload it. Must be less than `expiresIn`.
    - `staleTimeout` - number of milliseconds to wait before checking if an item is stale.
    - `cache` - the name of the cache connection configured in the ['server.cache` option](#server.config.cache). Defaults to the default cache.

```javascript
var cache = server.cache('countries', { expiresIn: 60 * 60 * 1000 });
```

#### `server.auth.scheme(name, scheme)`

Registers an authentication scheme where:

- `name` - the scheme name.
- `scheme` - the method implementing the scheme with signature `function(server, options)` where:
    - `server` - a reference to the server object the scheme is added to.
    - `options` - optional scheme settings used to instantiate a strategy.

The `scheme` method must return an object with the following keys:

- `authenticate(request, reply)` - required function called on each incoming request configured with the authentication scheme where:
    - `request` - the request object.
    - `reply(err, result)` - the interface the authentication method must call when done where:
        - `err` - if not `null`, indicates failed authentication.
        - `result` - an object containing:
            - `credentials` - the authenticated credentials. Required if `err` is `null`.
            - `artifacts` - optional authentication artifacts.
            - `log` - optional object used to customize the request authentication log which supports:
                - `data` - log data.
                - `tags` - additional tags.
- `payload(request, next)` - optional function called to authenticate the request payload where:
    - `request` - the request object.
    - `next(err)` - the continuation function the method must called when done where:
        - `err` - if `null`, payload successfully authenticated. If `false`, indicates that authentication could not be performed
          (e.g. missing payload hash). If set to any other value, it is used as an error response.
- `response(request, next)` - optional function called to decorate the response with authentication headers before the response
  headers or payload is written where:
    - `request` - the request object.
    - `next(err)` - the continuation function the method must called when done where:
        - `err` - if `null`, successfully applied. If set to any other value, it is used as an error response.

#### `server.auth.strategy(name, scheme, [mode], [options])`

Registers an authentication strategy where:

- `name` - the strategy name.
- `scheme` - the scheme name (must be previously registered using `server.auth.scheme()`).
- `mode` - if `true`, the scheme is automatically assigned as a required strategy to any route without an `auth` config. Can only be
  assigned to a single server strategy. Value must be `true` (which is the same as `'required'`) or a valid authentication mode
  (`'required'`, `'optional'`, `'try'`). Defaults to `false`.
- `options` - scheme options based on the scheme requirements.

#### `server.ext(event, method, [options])`

Registers an extension function in one of the available [extension points](#request-lifecycle) where:

- `event` - the event name.
- `method` - a function or an array of functions to be executed at a specified point during request processing. The required extension function signature
  is `function(request, next)` where:
    - `request` - the incoming `request` object.
    - `next` - the callback function the extension method must call to return control over to the router with signature `function(exit)` where:
        - `exit` - optional request processing exit response. If set to a non-falsy value, the request lifecycle process will jump to the
          "send response" step, skipping all other steps in between, and using the `exit` value as the new response. `exit` can be any result
          value accepted by [`reply()`](#replyresult).
    - `this` - the object provided via `options.bind`.
- `options` - an optional object with the following:
    - `before` - a string or array of strings of plugin names this method must execute before (on the same event). Otherwise, extension methods are executed
      in the order added.
    - `after` - a string or array of strings of plugin names this method must execute after (on the same event). Otherwise, extension methods are executed
      in the order added.
    - `bind` - any value passed back to the provided method (via `this`) when called.

```javascript
var Hapi = require('hapi');
var server = new Hapi.Server();

server.ext('onRequest', function (request, next) {

    // Change all requests to '/test'
    request.setUrl('/test');
    next();
});

var handler = function (request, reply) {

    reply({ status: 'ok' });
};

server.route({ method: 'GET', path: '/test', handler: handler });
server.start();

// All requests will get routed to '/test'
```

##### Request lifecycle

Each incoming request passes through a pre-defined set of steps, along with optional extensions:

- **`'onRequest'`** extension point
    - always called
    - the `request` object passed to the extension functions is decorated with the `request.setUrl(url)` and `request.setMethod(verb)` methods. Calls to these methods
      will impact how the request is routed and can be used for rewrite rules.
    - `request.route` is not yet populated as the router only looks at the request after this point.
- Lookup route using request path
- Parse cookies
- **`'onPreAuth'`** extension point
- Authenticate request
- Read and parse payload
- Authenticate request payload
- **`'onPostAuth'`** extension point
- Validate path parameters
- Process query extensions (e.g. JSONP)
- Validate query
- Validate payload
- **`'onPreHandler'`** extension point
- Route prerequisites
- Route handler
- **`'onPostHandler'`** extension point
    - The response object contained in `request.response` may be modified (but not assigned a new value). To return a different response type
      (for example, replace an error with an HTML response), return a new response via `next(response)`.
- Validate response payload
- **`'onPreResponse'`** extension point
    - always called.
    - The response contained in `request.response` may be modified (but not assigned a new value). To return a different response type (for
      example, replace an error with an HTML response), return a new response via `next(response)`. Note that any errors generated after
      `next(response)` is called will not be passed back to the `'onPreResponse'` extention method to prevent an infinite loop.
- Send response (may emit `'internalError'` event)
- Emits `'response'` event
- Wait for tails
- Emits `'tail'` event

#### `server.method(name, fn, [options])`

Registers a server method function. Server methods are functions registered with the server and used throughout the application as
a common utility. Their advantage is in the ability to configure them to use the built-in cache and shared across multiple request
handlers without having to create a common module.

Methods are registered via `server.method(name, fn, [options])` where:

- `name` - a unique method name used to invoke the method via `server.methods[name]`. When configured with caching enabled,
  `server.methods[name].cache.drop(arg1, arg2, ..., argn, callback)` can be used to clear the cache for a given key. Supports using
  nested names such as `utils.users.get` which will automatically create the missing path under `server.methods` and can be accessed
  for the previous example via `server.methods.utils.users.get`.
- `fn` - the method function with the signature is `function(arg1, arg2, ..., argn, next)` where:
    - `arg1`, `arg2`, etc. - the method function arguments.
    - `next` - the function called when the method is done with the signature `function(err, result, isUncacheable)` where:
        - `err` - error response if the method failed.
        - `result` - the return value.
        - `ttl` - `0` if result is valid but cannot be cached. Defaults to cache policy.
- `options` - optional configuration:
    - `bind` - an object passed back to the provided method function (via `this`) when called. Defaults to `null` unless added via a plugin, in which
      case it defaults to the plugin bind object.
    - `cache` - cache configuration as described in [**catbox** module documentation](https://github.com/spumko/catbox#policy) with a few additions:
        - `expiresIn` - relative expiration expressed in the number of milliseconds since the item was saved in the cache. Cannot be used
          together with `expiresAt`.
        - `expiresAt` - time of day expressed in 24h notation using the 'MM:HH' format, at which point all cache records for the route
          expire. Cannot be used together with `expiresIn`.
        - `staleIn` - number of milliseconds to mark an item stored in cache as stale and reload it. Must be less than `expiresIn`.
        - `staleTimeout` - number of milliseconds to wait before checking if an item is stale.
        - `segment` - optional segment name, used to isolate cached items within the cache partition. Defaults to '#name' where 'name' is the
          method name. When setting segment manually, it must begin with '##'.
        - `cache` - the name of the cache connection configured in the ['server.cache` option](#server.config.cache). Defaults to the default cache.
    - `generateKey` - a function used to generate a unique key (for caching) from the arguments passed to the method function
     (with the exception of the last 'next' argument). The server will automatically generate a unique key if the function's
     arguments are all of types `'string'`, `'number'`, or `'boolean'`. However if the method uses other types of arguments, a
     key generation function must be provided which takes the same arguments as the function and returns a unique string (or
     `null` if no key can be generated). Note that when the `generateKey` method is invoked, the arguments list will include
     the `next` argument which must not be used in calculation of the key.

```javascript
var Hapi = require('hapi');
var server = new Hapi.Server();

// Simple arguments

var add = function (a, b, next) {

    next(null, a + b);
};

server.method('sum', add, { cache: { expiresIn: 2000 } });

server.methods.sum(4, 5, function (err, result) {

    console.log(result);
});

// Object argument

var addArray = function (array, next) {

    var sum = 0;
    array.forEach(function (item) {

        sum += item;
    });

    next(null, sum);
};

server.method('sumObj', addArray, {
    cache: { expiresIn: 2000 },
    generateKey: function (array) {

        return array.join(',');
    }
});

server.methods.sumObj([5, 6], function (err, result) {

    console.log(result);
});
```

#### `server.method(method)`

Registers a server method function as described in [`server.method()`](#servermethodname-fn-options) using a method object or an array
of objects where each has:
- `name` - the method name.
- `fn` - the method function.
- `options` - optional settings.

```javascript
var add = function (a, b, next) {

    next(null, a + b);
};

server.method({ name: 'sum', fn: add, options: { cache: { expiresIn: 2000 } } });

server.method([{ name: 'also', fn: add }]);
```

#### `server.inject(options, callback)`

Injects a request into the server simulating an incoming HTTP request without making an actual socket connection. Injection is useful for
testing purposes as well as for invoking routing logic internally without the overhead or limitations of the network stack. Utilizes the
[**shot**](https://github.com/spumko/shot) module for performing injections, with some additional options and response properties:

- `options` - can be assign a string with the requested URI, or an object with:
    - `method` - the request HTTP method (e.g. `'POST'`). Defaults to `'GET'`.
    - `url` - the request URL. If the URI includes an authority (e.g. `'example.com:8080'`), it is used to automatically set an HTTP 'Host'
      header, unless one was specified in `headers`.
    - `headers` - an object with optional request headers where each key is the header name and the value is the header content. Defaults
      to no additions to the default Shot headers.
    - `payload` - an optional string or buffer containing the request payload (object must be manually converted to a string first).
      Defaults to no payload. Note that payload processing defaults to `'application/json'` if no 'Content-Type' header provided.
    - `credentials` - an optional credentials object containing authentication information. The `credentials` are used to bypass the default
      authentication strategies, and are validated directly as if they were received via an authentication scheme. Defaults to no credentials.
    - `simulate` - an object with options used to simulate client request stream conditions for testing:
        - `error` - if `true`, emits an `'error'` event after payload transmission (if any). Defaults to `false`.
        - `close` - if `true`, emits a `'close'` event after payload transmission (if any). Defaults to `false`.
        - `end` - if `false`, does not end the stream. Defaults to `true`.
- `callback` - the callback function with signature `function(res)` where:
    - `res` - the response object where:
        - `statusCode` - the HTTP status code.
        - `headers` - an array containing the headers set.
        - `payload` - the response payload string.
        - `rawPayload` - the raw response payload buffer.
        - `raw` - an object with the injection request and response objects:
            - `req` - the `request` object.
            - `res` - the response object.
        - `result` - the raw handler response (e.g. when not a stream) before it is serialized for transmission. If not available, set to
          `payload`. Useful for inspection and reuse of the internal objects returned (instead of parsing the response string).

```javascript
var Hapi = require('hapi');
var server = new Hapi.Server();

var get = function (request, reply) {

    reply('Success!');
};

server.route({ method: 'GET', path: '/', handler: get });

server.inject('/', function (res) {

    console.log(res.result);
});
```

#### `server.handler(name, method)`

Registers a new handler type which can then be used in routes. Overriding the built in handler types (`directory`, `file`, `proxy`, and `view`),
or any previously registered types is not allowed.

- `name` - string name for the handler being registered.
- `method` - the function used to generate the route handler using the signature `function(route, options)` where:
    - `route` - the internal route object.
    - `options` - the configuration object provided in the handler config.

```javascript
var Hapi = require('hapi');
var server = Hapi.createServer('localhost', 8000);

// Defines new handler for routes on this server
server.handler('test', function (route, options) {

    return function (request, reply) {

        reply('new handler: ' + options.msg);
    }
});

server.route({
    method: 'GET',
    path: '/',
    handler: { test: { msg: 'test' } }
});

server.start();
```

### `Server` events

The server object inherits from `Events.EventEmitter` and emits the following events:

- `'log'` - events logged with [server.log()](#serverlogtags-data-timestamp).
- `'request'` - events generated by [request.log()](#requestlogtags-data-timestamp) or internally (multiple events per request).
- `'response'` - emitted after a response to a client request is sent back. Single event per request.
- `'tail'` - emitted when a request finished processing, including any registered [tails](#requesttailname). Single event per request.
- `'internalError'` - emitted whenever an Internal Server Error (500) error response is sent. Single event per request.

When provided (as listed below) the `event` object include:

- `timestamp` - the event timestamp.
- `request` - if the event relates to a request, the `request id`.
- `server` - if the event relates to a server, the `server.info.uri`.
- `tags` - an array of tags (e.g. `['error', 'http']`). Includes the `'hapi'` tag is the event was generated internally.
- `data` - optional event-specific information.

The `'log'` event includes the `event` object and a `tags` object (where each tag is a key with the value `true`):

```javascript
server.on('log', function (event, tags) {

    if (tags.error) {
        console.log('Server error: ' + (event.data || 'unspecified'));
    }
});
```

The `'request'` event includes the `request` object, the `event` object, and a `tags` object (where each tag is a key with the value `true`):

```javascript
server.on('request', function (request, event, tags) {

    if (tags.received) {
        console.log('New request: ' + event.id);
    }
});
```

The `'response'` and `'tail'` events include the `request` object:

```javascript
server.on('response', function (request) {

    console.log('Response sent for request: ' + request.id);
});
```

The `'internalError'` event includes the `request` object and the causing error `err` object:

```javascript
server.on('internalError', function (request, err) {

    console.log('Error response (500) sent for request: ' + request.id + ' because: ' + err.message);
});
```

## Request object

The `request` object is created internally for each incoming request. It is **not** the node `request` object received from the HTTP
server callback (which is available in `request.raw.req`). The `request` object methods and properties change through the
[request lifecycle](#request-lifecycle).

### `request` properties

Each request object has the following properties:

- `app` - application-specific state. Provides a safe place to store application data without potential conflicts with **hapi**.
  Should not be used by plugins which should use `plugins[name]`.
- `auth` - authentication information:
    - `isAuthenticated` - `true` is the request has been successfully authenticated, otherwise `false`.
    - `credentials` - the `credential` object received during the authentication process. The presence of an object does not mean
      successful authentication.
    - `artifacts` - an artifact object received from the authentication strategy and used in authentication-related actions.
    - `session` - an object used by the [`'cookie'` authentication scheme](#cookie-authentication).
- `domain` - the node domain object used to protect against exceptions thrown in extentions, handlers and prerequisites. Can be used to
  manually bind callback functions otherwise bound to other domains.
- `headers` - the raw request headers (references `request.raw.headers`).
- `id` - a unique request identifier.
- `info` - request information:
    - `received` - request reception timestamp.
    - `remoteAddress` - remote client IP address.
    - `remotePort` - remote client port.
    - `referrer` - content of the HTTP 'Referrer' (or 'Referer') header.
    - `host` - content of the HTTP 'Host' header.
- `method` - the request method in lower case (e.g. `'get'`, `'post'`).
- `mime` - the parsed content-type header. Only available when payload parsing enabled and no payload error occurred.
- `params` - an object where each key is a path parameter name with matching value as described in [Path parameters](#path-parameters).
- `path` - the request URI's path component.
- `payload` - the request payload based on the route `payload.output` and `payload.parse` settings.
- `plugins` - plugin-specific state. Provides a place to store and pass request-level plugin data. The `plugins` is an object where each
  key is a plugin name and the value is the state.
- `pre` - an object where each key is the name assigned by a [route prerequisites](#route-prerequisites) function. The values are the raw values
  provided to the continuation function as argument. For the wrapped response object, use `responses`.
- `response` - the response object when set. The object can be modified but must not be assigned another object. To replace the response
  with another from within an extension point, use `next(response)` to override with a different response.
- `responses` - same as `pre` but represented as the response object created by the pre method.
- `query` - an object containing the query parameters.
- `raw` - an object containing the Node HTTP server objects. **Direct interaction with these raw objects is not recommended.**
    - `req` - the `request` object.
    - `res` - the response object.
- `route` - the route configuration object after defaults are applied.
- `server` - the server object.
- `session` - Special key reserved for plugins implementing session support. Plugins utilizing this key must check for `null` value
  to ensure there is no conflict with another similar plugin.
- `state` - an object containing parsed HTTP state information (cookies) where each key is the cookie name and value is the matching
  cookie content after processing using any registered cookie definition.
- `url` - the parsed request URI.

### `request` methods

#### `request.setUrl(url)`

_Available only in `'onRequest'` extension methods._

 Changes the request URI before the router begins processing the request where:

 - `url` - the new request path value.

```javascript
var Hapi = require('hapi');
var server = new Hapi.Server();

server.ext('onRequest', function (request, next) {

    // Change all requests to '/test'
    request.setUrl('/test');
    next();
});
```

#### `request.setMethod(method)`

_Available only in `'onRequest'` extension methods._

Changes the request method before the router begins processing the request where:

- `method` - is the request HTTP method (e.g. `'GET'`).

```javascript
var Hapi = require('hapi');
var server = new Hapi.Server();

server.ext('onRequest', function (request, next) {

    // Change all requests to 'GET'
    request.setMethod('GET');
    next();
});
```

#### `request.log(tags, [data, [timestamp]])`

_Always available._

Logs request-specific events. When called, the server emits a `'request'` event which can be used by other listeners or plugins. The
arguments are:

- `tags` - a string or an array of strings (e.g. `['error', 'database', 'read']`) used to identify the event. Tags are used instead of log levels
  and provide a much more expressive mechanism for describing and filtering events. Any logs generated by the server internally include the `'hapi'`
  tag along with event-specific information.
- `data` - an optional message string or object with the application data being logged.
- `timestamp` - an optional timestamp expressed in milliseconds. Defaults to `Date.now()` (now).

```javascript
var Hapi = require('hapi');
var server = new Hapi.Server();

server.on('request', function (request, event, tags) {

    if (tags.error) {
        console.log(event);
    }
});

var handler = function (request, reply) {

    request.log(['test', 'error'], 'Test event');
};
```

#### `request.getLog([tags])`

_Always available._

Returns an array containing the events matching any of the tags specified (logical OR) where:
- `tags` - is a single tag string or array of tag strings. If no `tags` specified, returns all events.

```javascript
request.getLog();
request.getLog('error');
request.getLog(['hapi', 'error']);
```

#### `request.tail([name])`

_Available until immediately after the `'response'` event is emitted._

Adds a request tail which has to complete before the request lifecycle is complete where:

- `name` - an optional tail name used for logging purposes.

Returns a tail function which must be called when the tail activity is completed.

Tails are actions performed throughout the request lifecycle, but which may end after a response is sent back to the client. For example, a
request may trigger a database update which should not delay sending back a response. However, it is still desirable to associate the activity
with the request when logging it (or an error associated with it).

When all tails completed, the server emits a `'tail'` event.

```javascript
var Hapi = require('hapi');
var server = new Hapi.Server();

var get = function (request, reply) {

    var dbTail = request.tail('write to database');

    db.save('key', 'value', function () {

        dbTail();
    });

    reply('Success!');
};

server.route({ method: 'GET', path: '/', handler: get });

server.on('tail', function (request) {

    console.log('Request completed including db activity');
});
```

### Request events

The request object supports the following events:

- `'peek'` - emitted for each chunk of payload data read from the client connection. The event method signature is `function(chunk, encoding)`.
- `'finish'` - emitted when the request payload finished reading. The event method signature is `function ()`.
- `'disconnect'` - emitted when a request errors or aborts unexpectedly.

```javascript
var Crypto = require('crypto');
var Hapi = require('hapi');
var server = new Hapi.Server();

server.ext('onRequest', function (request, reply) {

    var hash = Crypto.createHash('sha1');
    request.on('peek', function (chunk) {

        hash.update(chunk);
    });

    request.once('finish', function () {

        console.log(hash.digest('hex'));
    });

    request.once('disconnect', function () {

        console.error('request aborted');
    });
});
```

## Reply interface

### Flow control

When calling `reply()`, the router waits until `process.nextTick()` to continue processing the request and transmit the response.
This enables making changes to the returned response object before the response is sent. This means the router will resume as soon as the handler
method exists. To suspend this behavior, the returned `response` object includes:

- `response.hold()` - puts the response on hold until `response.send()` is called. Available only after `reply()` is called and until
  `response.hold()` is invoked once.
- `response.send()` - resume the response which will be transmitted in the next tick. Available only after `response.hold()` is called and until
  `response.send()` is invoked once.

```javascript
var handler = function (request, reply) {

    var response = reply('success').hold();

    onTimeout(function () {

        response.send();
    }, 1000);
};
```

When calling `reply()` in a prerequisite, it is sometimes necessary to take over the handler execution and return a non-error response back
to the client. The response object provides the `takeover()` method to indicate the value provided via `reply()` should be used as the final
response and skip any other prerequisites and the handler.

```javascript
var pre = function (request, reply) {

    if (!request.auth.isAuthenticated) {
        return reply('You need to login first!').takeover();
    }

    reply({ account: request.auth.credentials });   // Used in the handler later
};
```

### `reply([result])`

_Available only within the handler method and only before one of `reply()`, `reply.file()`, `reply.view()`,
`reply.close()`, or `reply.proxy()`  is called._

Concludes the handler activity by returning control over to the router where:

- `result` - an optional response payload.

Returns a [`response`](#response-object) object based on the value of `result`:

- `null`, `undefined`, or empty string `''` - [`Empty`](#empty) response.
- string - [`Text`](#text) response.
- `Buffer` object - [`Buffer`](#buffer) response.
- `Error` object (generated via [`error`](#hapierror) or `new Error()`) - [`Boom`](#hapierror) object.
- `Stream` object - [`Stream`](#stream) response.
- any other object - [`Obj`](#obj) response.

```javascript
var handler = function (request, reply) {

    reply('success');
};
```

The returned `response` object provides a set of methods to customize the response (e.g. HTTP status code, custom headers, etc.). The methods
are response-type-specific and listed in [`response`](#response-object).

The [response flow control rules](#flow-control) apply.

```javascript
var handler = function (request, reply) {

    reply('success')
        .type('text/plain')
        .header('X-Custom', 'some-value');
};
```

Note that if `result` is a `Stream` with a `statusCode` property, that status code will be used as the default response code.

### `reply.file(path, [options])`

_Available only within the handler method and only before one of `reply()`, `reply.file()`, `reply.view()`,
`reply.close()`, or `reply.proxy()`  is called._

Transmits a file from the file system. The 'Content-Type' header defaults to the matching mime type based on filename extension.:

- `path` - the file path.
- `options` - optional settings:
    - `filePath` - a relative or absolute file path string (relative paths are resolved based on the server [`files`](#server.config.files) configuration).
    - `options` - optional configuration:
        - `filename` - optional filename to send if the 'Content-Disposition' header is sent. Defaults to basename of `path`.
        - `mode` - value of the HTTP 'Content-Disposition' header. Allowed values:
            - `'attachment'`
            - `'inline'`

No return value.

The [response flow control rules](#flow-control) **do not** apply.

```javascript
var handler = function (request, reply) {

    reply.file('./hello.txt');
};
```

### `reply.view(template, [context, [options]])`

_Available only within the handler method and only before one of `reply()`, `reply.file()`, `reply.view()`,
`reply.close()`, or `reply.proxy()`  is called._

Concludes the handler activity by returning control over to the router with a templatized view response where:

- `template` - the template filename and path, relative to the templates path configured via the server [`views.path`](#server.config.views).
- `context` - optional object used by the template to render context-specific result. Defaults to no context `{}`.
- `options` - optional object used to override the server's [`views`](#server.config.views) configuration for this response.

Returns a response object.

The [response flow control rules](#flow-control) apply.

```javascript
var Hapi = require('hapi');
var server = new Hapi.Server({
    views: {
        engines: { html: 'handlebars' },
        path: __dirname + '/templates'
    }
});

var handler = function (request, reply) {

    var context = {
        title: 'Views Example',
        message: 'Hello, World'
    };

    reply.view('hello', context);
};

server.route({ method: 'GET', path: '/', handler: handler });
```

**templates/hello.html**

```html
<!DOCTYPE html>
<html>
    <head>
        <title>{{title}}</title>
    </head>
    <body>
        <div>
            <h1>{{message}}</h1>
        </div>
    </body>
</html>
```

### `reply.close([options])`

_Available only within the handler method and only before one of `reply()`, `reply.file()`, `reply.view()`,
`reply.close()`, or `reply.proxy()`  is called._

Concludes the handler activity by returning control over to the router and informing the router that a response has already been sent back
directly via `request.raw.res` and that no further response action is needed. Supports the following optional options:
- `end` - if `false`, the router will not call `request.raw.res.end())` to ensure the response was ended. Defaults to `true`.

No return value.

The [response flow control rules](#flow-control) **do not** apply.

### `reply.proxy(options)`

_Available only within the handler method and only before one of `reply()`, `reply.file()`, `reply.view()`,
`reply.close()`, or `reply.proxy()`  is called._

Proxies the request to an upstream endpoint where:
- `options` - an object including the same keys and restrictions defined by the [route `proxy` handler options](#route.config.proxy).

No return value.

The [response flow control rules](#flow-control) **do not** apply.

```javascript
var handler = function (request, reply) {

    reply.proxy({ host: 'example.com', port: 80, protocol: 'http' });
};
```

## Response object

Every response includes the following properties:
- `statusCode` - the HTTP response status code. Defaults to `200` (except for errors).
- `headers` - an object containing the response headers where each key is a header field name. Note that this is an incomplete list of
  headers to be included with the response. Additional headers will be added once the response is prepare for transmission (e.g. 'Location',
  'Cache-Control').
- `source` - the value provided using the `reply()` interface.
- `variety` - a string indicating the type of `source` with available values:
    - `'plain'` - a plain response such as string, number, `null`, or simple object (e.g. not a `Stream`, `Buffer`, or view).
    - `'buffer'` - a `Buffer`.
    - `'view'` - a view generated with `reply.view()`.
    - `'file'` - a file generated with `reply.file()` of via the directory handler.
    - `'stream'` - a `Stream`.
- `app` - application-specific state. Provides a safe place to store application data without potential conflicts with **hapi**.
  Should not be used by plugins which should use `plugins[name]`.
- `plugins` - plugin-specific state. Provides a place to store and pass request-level plugin data. The `plugins` is an object where each
  key is a plugin name and the value is the state.
- `settings` - response handling flags:
    - `encoding` - the string encoding scheme used to serial data into the HTTP payload when `source` is a string or marshalls into a string.
      Defaults to `'utf8'`.
    - `charset` -  the 'Content-Type' HTTP header 'charset' property. Defaults to `'utf-8'`.
    - `location` - the raw value used to set the HTTP 'Location' header (actual value set depends on the server
      [`location`](#server.config.location) configuration option). Defaults to no header.
    - `ttl` -  if set, overrides the route cache expiration milliseconds value set in the route config. Defaults to no override.
    - `stringify`: options used for `source` value requiring stringification. Defaults to no replacer and no space padding.
    - `passThrough`: if `true` and `source` is a `Stream`, copies the `statusCode` and `headers` of the stream to the outbound response.
      Defaults to `true`.

It provides the following methods:

- `code(statusCode)` - sets the HTTP status code where:
    - `statusCode` - the HTTP status code.
- `header(name, value, options)` - sets an HTTP header where:
    - `name` - the header name.
    - `value` - the header value.
    - `options` - optional settings where:
        - `append` - if `true`, the value is appended to any existing header value using `separator`. Defaults to `false`.
        - `separator` - string used as separator when appending to an exiting value. Defaults to `','`.
        - `override` - if `false`, the header value is not set if an existing value present. Defaults to `true`.
- `type(mimeType)` - sets the HTTP 'Content-Type' header where:
    - `value` - is the mime type. Should only be used to override the built-in default for each response type.
- `bytes(length)` - sets the HTTP 'Content-Length' header (to avoid chunked transfer encoding) where:
    - `length` - the header value. Must match the actual payload size.
- `vary(header)` - adds the provided header to the list of inputs affected the response generation via the HTTP 'Vary' header where:
    - `header` - the HTTP request header name.
- `location(location)` - sets the HTTP 'Location' header where:
    - `uri` - an absolute or relative URI used as the 'Location' header value. If a relative URI is provided, the value of the server
      [`location`](#server.config.location) configuration option is used as prefix.
- `created(location)` - sets the HTTP status code to Created (201) and the HTTP 'Location' header where:
    `location` - an absolute or relative URI used as the 'Location' header value. If a relative URI is provided, the value of
      the server [`location`](#server.config.location) configuration option is used as prefix. Not available for methods other than PUT and POST.
- `redirect(location)` - sets an HTTP redirection response (302) and decorates the response with additional methods listed below, where:
    - `location` - an absolute or relative URI used to redirect the client to another resource. If a relative URI is provided, the value of
      the server [`location`](#server.config.location) configuration option is used as prefix.
- `encoding(encoding)` - sets the string encoding scheme used to serial data into the HTTP payload where:
    `encoding` - the encoding property value (see [node Buffer encoding](http://nodejs.org/api/buffer.html#buffer_buffer)).
- `charset(charset)` - sets the 'Content-Type' HTTP header 'charset' property where:
    `charset` - the charset property value.
- `ttl(msec)` - overrides the default route cache expiration rule for this response instance where:
    - `msec` - the time-to-live value in milliseconds.
- `state(name, value, [options])` - sets an HTTP cookie where:
    - `name` - the cookie name.
    - `value` - the cookie value. If no `encoding` is defined, must be a string.
    - `options` - optional configuration. If the state was previously registered with the server using [`server.state()`](#serverstatename-options),
      the specified keys in `options` override those same keys in the server definition (but not others).
- `unstate(name)` - clears the HTTP cookie by setting an expired value where:
    - `name` - the cookie name.

When the value provided by `reply()` requires stringification before transmission, the following methods are provided:

- `replacer(method)` - sets the `JSON.stringify()` `replacer` argument where:
    - `method` - the replacer function or array. Defaults to none.
- `spaces(count)` - sets the `JSON.stringify()` `space` argument where:
    - `count` - the number of spaces to indent nested object keys. Defaults to no indentation.

When using the `redirect()` method, the response object provides these additional methods:

- `temporary(isTemporary)` - sets the status code to `302` or `307` (based on the `rewritable()` setting) where:
    - `isTemporary` - if `false`, sets status to permanent. Defaults to `true`.
- `permanent(isPermanent)` - sets the status code to `301` or `308` (based on the `rewritable()` setting) where:
    - `isPermanent` - if `true`, sets status to temporary. Defaults to `false`.
- `rewritable(isRewritable)` - sets the status code to `301`/`302` for rewritable (allows changing the request method from 'POST' to 'GET') or
  `307`/`308` for non-rewritable (does not allow changing the request method from 'POST' to 'GET'). Exact code based on the `temporary()` or
  `permanent()` setting. Arguments:
    - `isRewritable` - if `false`, sets to non-rewritable. Defaults to `true`.

|                |  Permanent | Temporary |
| -------------- | ---------- | --------- |
| Rewritable     | 301        | **302**(1)|
| Non-rewritable | 308(2)     | 307       |

Notes:
1. Default value.
2. [Proposed code](http://tools.ietf.org/id/draft-reschke-http-status-308-07.txt), not supported by all clients.

### Response events

The response object supports the following events:

- `'peek'` - emitted for each chunk of data written back to the client connection. The event method signature is `function(chunk, encoding)`.
- `'finish'` - emitted when the response finished writing but before the client response connection is ended. The event method signature is
  `function ()`.

```javascript
var Crypto = require('crypto');
var Hapi = require('hapi');
var server = new Hapi.Server();

server.ext('onPreResponse', function (request, reply) {

    var response = request.response;
    if (response.isBoom) {
        return reply();
    }

    var hash = Crypto.createHash('sha1');
    response.on('peek', function (chunk) {

        hash.update(chunk);
    });

    response.once('finish', function () {

        console.log(hash.digest('hex'));
    });
});
```

## `Hapi.error`

Provides a set of utilities for returning HTTP errors. An alias of the [**boom**](https://github.com/spumko/boom) module (can be also accessed
`Hapi.boom`). Each utility returns a `Boom` error response object (instance of `Error`) which includes the following properties:

- `isBoom` - if `true`, indicates this is a `Boom` object instance.
- `message` - the error message.
- `output` - the formatted response. Can be directly manipulated after object construction to return a custom error response. Allowed root keys:
    - `statusCode` - the HTTP status code (typically 4xx or 5xx).
    - `headers` - an object containing any HTTP headers where each key is a header name and value is the header content.
    - `payload` - the formatted object used as the response payload (stringified). Can be directly manipulated but any changes will be lost
      if `reformat()` is called. Any content allowed and by default includes the following content:
        - `statusCode` - the HTTP status code, derived from `error.output.statusCode`.
        - `error` - the HTTP status message (e.g. 'Bad Request', 'Internal Server Error') derived from `statusCode`.
        - `message` - the error message derived from `error.message`.
- inherited `Error` properties.

It also supports the following method:

- `reformat()` - rebuilds `error.output` using the other object properties.

```javascript
var Hapi = require('hapi');

var handler = function (request, reply) {

    var error = Hapi.error.badRequest('Cannot feed after midnight');
    error.output.statusCode = 499;    // Assign a custom error code
    error.reformat();
    
    error.output.payload.custom = 'abc_123'; // Add custom key

    reply(error);
});
```

### Error transformation

Error responses return a JSON object with the `statusCode`, `error`, and `message` keys. When a different error representation is desired, such
as an HTML page or using another format, the `'onPreResponse'` extension point may be used to identify errors and replace them with a different
response object.

```javascript
var Hapi = require('hapi');
var server = new Hapi.Server({ views: { engines: { html: 'handlebars' } } });

server.ext('onPreResponse', function (request, reply) {

    var response = request.response;
    if (!response.isBoom) {
        return reply();
    }

    // Replace error with friendly HTML

      var error = response;
      var ctx = {
          message: (error.output.statusCode === 404 ? 'page not found' : 'something went wrong')
      };

      reply.view('error', ctx);
});
```

#### `badRequest([message])`

Returns an HTTP Bad Request (400) error response object with the provided `message`.

```javascript
var Hapi = require('hapi');
Hapi.error.badRequest('Invalid parameter value');
```

#### `unauthorized(message, [scheme, [attributes]])`

Returns an HTTP Unauthorized (401) error response object where:

- `message` - the error message.
- `scheme` - optional HTTP authentication scheme name (e.g. `'Basic'`, `'Hawk'`). If provided, includes the HTTP 'WWW-Authenticate'
  response header with the scheme and any provided `attributes`.
- `attributes` - an object where each key is an HTTP header attribute and value is the attribute content.

```javascript
var Hapi = require('hapi');
Hapi.error.unauthorized('Stale timestamp', 'Hawk', { ts: fresh, tsm: tsm });
```

#### `unauthorized(message, wwwAuthenticate)`

Returns an HTTP Unauthorized (401) error response object where:

- `message` - the error message.
- `wwwAuthenticate` - an array of HTTP 'WWW-Authenticate' header responses for multiple challenges.

```javascript
var Hapi = require('hapi');
Hapi.error.unauthorized('Missing authentication', ['Hawk', 'Basic']);
```

#### `clientTimeout([message])`

Returns an HTTP Request Timeout (408) error response object with the provided `message`.

```javascript
var Hapi = require('hapi');
Hapi.error.clientTimeout('This is taking too long');
```

#### `serverTimeout([message])`

Returns an HTTP Service Unavailable (503) error response object with the provided `message`.

```javascript
var Hapi = require('hapi');
Hapi.error.serverTimeout('Too busy, come back later');
```

#### `forbidden([message])`

Returns an HTTP Forbidden (403) error response object with the provided `message`.

```javascript
var Hapi = require('hapi');
Hapi.error.forbidden('Missing permissions');
```

#### `notFound([message])`

Returns an HTTP Not Found (404) error response object with the provided `message`.

```javascript
var Hapi = require('hapi');
Hapi.error.notFound('Wrong number');
```

#### `internal([message, [data]])`

Returns an HTTP Internal Server Error (500) error response object where:

- `message` - the error message.
- `data` - optional data used for error logging. If `data` is an `Error`, the returned object is `data` decorated with
  the **boom** properties. Otherwise, the returned `Error` has a `data` property with the provided value.

Note that the `error.output.payload.message` is overridden with `'An internal server error occurred'` to hide any internal details from
the client. `error.message` remains unchanged.

```javascript
var Hapi = require('hapi');

var handler = function (request, reply) {

    var result;
    try {
        result = JSON.parse(request.query.value);
    }
    catch (err) {
        result = Hapi.error.internal('Failed parsing JSON input', err);
    }

    reply(result);
};
```

## `Hapi.Pack`

`Pack` is a collection of servers grouped together to form a single logical unit. The pack's primary purpose is to provide a unified object
interface when working with [plugins](#plugin-interface). Grouping multiple servers into a single pack enables treating them as a single
entity which can start and stop in sync, as well as enable sharing routes and other facilities.

The servers in a pack share the same cache. Every server belongs to a pack, even if created directed via
[`new Server()`](#new-serverhost-port-options), in which case the `server.pack` object is automatically assigned a single-server pack.

#### `new Pack([options])`

Creates a new `Pack` object instance where:

- `options` - optional configuration:
    - `app` - an object used to initialize the application-specific data stored in `pack.app`.
    - `cache` - cache configuration as described in the server [`cache`](#server.config.cache) option.
    - `requirePath` - sets the path from which node module plugins are loaded. Applies only when using [`pack.require()`](#packrequirename-options-callback)
      with module names that do no include a relative or absolute path (e.g. 'lout'). Defaults to the node module behaviour described in
      [node modules](http://nodejs.org/api/modules.html#modules_loading_from_node_modules_folders). Note that if the modules
      are located inside a 'node_modules' sub-directory, `requirePath` must end with `'/node_modules'`.

```javascript
var Hapi = require('hapi');
var pack = new Hapi.Pack();
```

### `Pack` properties

Each `Pack` object instance has the following properties:

- `app` - application-specific state. Provides a safe place to store application data without potential conflicts with **hapi**.
  Initialized via the pack `app` configuration option. Defaults to `{}`.
- `events` - an `Events.EventEmitter` providing a consolidate emitter of all the events emitted from all member pack servers as well as
  the `'start'` and `'stop'` pack events.
- `list` - an object listing all the registered plugins where each key is a plugin name and the value is an object with:
    - `name` - plugin name.
    - `version` - plugin version.
    - `path` - the plugin root path (where 'package.json' is located).
    - `register()` - the [`exports.register()`](#exportsregisterplugin-options-next) function.

### `Pack` methods

#### `pack.server([host], [port], [options])`

Creates a `Server` instance and adds it to the pack, where `host`, `port`, `options` are the same as described in
[`new Server()`](#new-serverhost-port-options) with the exception that the `cache` option is not allowed and must be
configured via the pack `cache` option.

```javascript
var Hapi = require('hapi');
var pack = new Hapi.Pack();

pack.server(8000, { labels: ['web'] });
pack.server(8001, { labels: ['admin'] });
```

#### `pack.start([callback])`

Starts all the servers in the pack and used as described in [`server.start([callback])`](#serverstartcallback).

```javascript
var Hapi = require('hapi');
var pack = new Hapi.Pack();

pack.server(8000, { labels: ['web'] });
pack.server(8001, { labels: ['admin'] });

pack.start(function () {

    console.log('All servers started');
});
```

#### `pack.stop([options], [callback])`

Stops all the servers in the pack and used as described in [`server.stop([options], [callback])`](#serverstopoptions-callback).

```javascript
pack.stop({ timeout: 60 * 1000 }, function () {

    console.log('All servers stopped');
});
```

#### `pack.require(name, [options], callback)`

Registers a plugin where:

- `name` - the node module name as expected by node's [`require()`](http://nodejs.org/api/modules.html#modules_module_require_id). If `name` is a relative
  path it is relative to the location of the file requiring it. If `name` is not a relative or absolute path (e.g. 'furball'), it is prefixed with the
  value of the pack `requirePath` configuration option when present. Note that node's `require()` is invoked by hapi which means, the `'node_modules'` path
  is relative to the location of the hapi module.
- `options` - optional configuration object which is passed to the plugin via the `options` argument in
  [`exports.register()`](#exportsregisterplugin-options-next).
- `callback` - the callback function with signature `function(err)` where:
      - `err` - an error returned from `exports.register()`. Note that incorrect usage, bad configuration, or namespace conflicts
        (e.g. among routes, methods, state) will throw an error and will not return a callback.

```javascript
pack.require('furball', { version: '/v' }, function (err) {

    if (err) {
        console.log('Failed loading plugin: furball');
    }
});
```

#### `pack.require(names, callback)`

Registers a list of plugins where:

- `names` - an array of plugins names as described in [`pack.require()`](#packrequirename-options-callback), or an object in which
  each key is a plugin name, and each value is the `options` object used to register that plugin.
- `callback` - the callback function with signature `function(err)` where:
      - `err` - an error returned from `exports.register()`. Note that incorrect usage, bad configuration, or namespace conflicts
        (e.g. among routes, methods, state) will throw an error and will not return a callback.

Batch registration is required when plugins declare a [dependency](#plugindependencydeps-after), so that all the required dependencies are loaded in
a single transaction (internal order does not matter).

```javascript
pack.require(['furball', 'lout'], function (err) {

    if (err) {
        console.log('Failed loading plugin: furball');
    }
});

pack.require({ furball: null, lout: { endpoint: '/docs' } }, function (err) {

    if (err) {
        console.log('Failed loading plugins');
    }
});
```

#### `pack.register(plugin, options, callback)`

Registers a plugin object (without using `require()`) where:

- `plugin` - the plugin object which requires:
    - `name` - plugin name.
    - `version` - plugin version.
    - `path` - optional plugin path for resolving relative paths used by the plugin. Defaults to current working directory.
    - `register()` - the [`exports.register()`](#exportsregisterplugin-options-next) function.
- `options` - optional configuration object which is passed to the plugin via the `options` argument in
  [`exports.register()`](#exportsregisterplugin-options-next).
- `callback` - the callback function with signature `function(err)` where:
    - `err` - an error returned from `exports.register()`. Note that incorrect usage, bad configuration, or namespace conflicts
      (e.g. among routes, methods, state) will throw an error and will not return a callback.

```javascript
var plug = {
    name: 'test',
    version: '2.0.0',
    register: function (plugin, options, next) {

        plugin.route({ method: 'GET', path: '/special', handler: function (request, reply) { reply(options.message); } } );
        next();
    }
};

server.pack.register(plug, { message: 'hello' }, function (err) {

    if (err) {
        console.log('Failed loading plugin');
    }
});
```

## `Hapi.Composer`

The `Composer` provides a simple way to construct a [`Pack`](#hapipack) from a single configuration object, including configuring servers
and registering plugins.

#### `new Composer(manifest)`

Creates a `Composer` object instance where:

- `manifest` - an object or array or objects where:
    - `pack` - the pack `options` as described in [`new Pack()`](#packserverhost-port-options).
    - `servers` - an array of server configuration objects where:
        - `host`, `port`, `options` - the same as described in [`new Server()`](#new-serverhost-port-options) with the exception that the
          `cache` option is not allowed and must be configured via the pack `cache` option. The `host` and `port` keys can be set to an environment variable by prefixing the variable name with `'$env.'`.
    - `plugin` - an object where each key is a plugin name, and each value is the `options` object used to register that plugin.

```javascript
var Hapi = require('hapi');

var manifest = {
    pack: {
        cache: 'catbox-memory'
    },
    servers: [
        {
            port: 8000,
            options: {
                labels: ['web']
            }
        },
        {
            host: 'localhost',
            port: 8001,
            options: {
                labels: ['admin']
            }
        }
    ],
    plugins: {
        'yar': {
            cookieOptions: {
                password: 'secret'
            }
        }
    }
};

var composer = new Hapi.Composer(manifest);
```

#### `composer.compose(callback)`

Creates the packs described in the manifest construction where:

- `callback` - the callback method, called when all packs and servers have been created and plugins registered has the signature
  `function(err)` where:
    - `err` - an error returned from `exports.register()`. Note that incorrect usage, bad configuration, or namespace conflicts
      (e.g. among routes, methods, state) will throw an error and will not return a callback.

```javascript
composer.compose(function (err) {

    if (err) {
        console.log('Failed composing');
    }
});
```

#### `composer.start([callback])`

Starts all the servers in all the pack composed where:

- `callback` - the callback method called when all the servers have been started.

```javascript
composer.start(function () {

    console.log('All servers started');
});
```

#### `composer.stop([options], [callback])`

Stops all the servers in all the packs and used as described in [`server.stop([options], [callback])`](#serverstopoptions-callback).

```javascript
pack.stop({ timeout: 60 * 1000 }, function () {

    console.log('All servers stopped');
});
```

## Plugin interface

Plugins provide an extensibility platform for both general purpose utilities such as [batch requests](https://github.com/spumko/bassmaster) and for
application business logic. Instead of thinking about a web server as a single entity with a unified routing table, plugins enable developers to
break their application into logical units, assembled together in different combinations to fit the development, testing, and deployment needs.

Constructing a plugin requires the following:

- name - the plugin name is used as a unique key. Public plugins should be published in the [npm registry](https://npmjs.org) and derive their name
  from the registry name. This ensures uniqueness. Private plugin names should be picked carefully to avoid conflicts with both private and public
  names. Typically, private plugin names use a prefix such as the company name or an unusual combination of characters (e.g. `'--'`). When using the
  [`pack.require()`](#packrequirename-options-callback) interface, the name is obtained from the 'package.json' module file. When using the
  [`pack.register()`](#packregisterplugin-options-callback) interface, the name is provided as a required key in `plugin`.
- version - the plugin version is only used informatively within the framework but plays an important role in the plugin echo-system. The plugin
  echo-system relies on the [npm peer dependency](http://blog.nodejs.org/2013/02/07/peer-dependencies/) functionality to ensure that plugins can
  specify their dependency on a specific version of **hapi**, as well as on each other. Dependencies are expressed solely within the 'package.json'
  file, and are enforced by **npm**. When using the [`pack.require()`](#packrequirename-options-callback) interface, the version is obtained from
  the 'package.json' module file. When using the [`pack.register()`](#packregisterplugin-options-callback) interface, the version is provided as
  a required key in `plugin`.
- `exports.register()` - the registration function described in [`exports.register()`](#exportsregisterplugin-options-next) is the plugin's core.
  The function is called when the plugin is registered and it performs all the activities required by the plugin to operate. It is the single entry
  point into the plugin functionality. When using the [`pack.require()`](#packrequirename-options-callback) interface, the function is obtained by
  [`require()`](http://nodejs.org/api/modules.html#modules_module_require_id)'ing the plugin module and invoking the exported `register()` method.
  When using the [`pack.register()`](#packregisterplugin-options-callback) interface, the function is provided as a required key in `plugin`.

**package.json**

```json
{
  "name": "furball",
  "description": "Plugin utilities and endpoints",
  "version": "0.3.0",
  "main": "index",
  "dependencies": {
    "hoek": "0.8.x"
  },
  "peerDependencies": {
    "hapi": "1.x.x"
  }
}
```

**index.js**

```javascript
var Hoek = require('hoek');

var internals = {
    defaults: {
        version: '/version',
        plugins: '/plugins'
    }
};

internals.version = 1.1;

exports.register = function (plugin, options, next) {

    var settings = Hoek.applyToDefaults(internals.defaults, options);

    if (settings.version) {
        plugin.route({
            method: 'GET',
            path: settings.version,
            handler: function (request, reply) {

                reply(internals.version);
            }
        });
    }

    if (settings.plugins) {
        plugin.route({
            method: 'GET',
            path: settings.plugins,
            handler: function (request, reply) {

                reply(listPlugins(request.server));
            }
        });
    }

    var listPlugins = function (server) {

        var plugins = [];
        Object.keys(server.pack.list).forEach(function (name) {

            var item = server.pack.list[name];
            plugins.push({
                name: item.name,
                version: item.version
            });
        });

        return plugins;
    };

    plugin.expose('plugins', listPlugins);
    next();
};
```

#### `exports.register(plugin, options, next)`

Registers the plugin where:

- `plugin` - the registration interface representing the pack the plugin is being registered into. Provides the properties and methods listed below.
- `options` - the `options` object provided by the pack registration methods.
- `next` - the callback function the plugin must call to return control over to the application and complete the registration process. The function
  signature is `function(err)` where:
    - `err` - internal plugin error condition, which is returned back via the registration methods' callback. A plugin registration error is considered
      an unrecoverable event which should terminate the application.

```javascript
exports.register = function (plugin, options, next) {

    plugin.route({ method: 'GET', path: '/', handler: function (request, reply) { reply('hello world') } });
    next();
};
```

### Root methods and properties

The plugin interface root methods and properties are those available only on the `plugin` object received via the
[`exports.register()`](#exportsregisterplugin-options-next) interface. They are not available on the object received by calling
[`plugin.select()`](#pluginselectlabels).

#### `plugin.version`

The plugin version information.

```javascript
exports.register = function (plugin, options, next) {

    console.log(plugin.version);
    next();
};
```

#### `plugin.path`

The plugin root path (where 'package.json' resides).

```javascript
var Fs = require('fs');

exports.register = function (plugin, options, next) {

    var file = Fs.readFileSync(plugin.path + '/resources/image.png');
    next();
};
```

#### `plugin.hapi`

A reference to the **hapi** module used to create the pack and server instances. Removes the need to add a dependency on **hapi** within the plugin.

```javascript
exports.register = function (plugin, options, next) {

    var Hapi = plugin.hapi;

    var handler = function (request, reply) {

        reply(Hapi.error.internal('Not implemented yet'));
    };

    plugin.route({ method: 'GET', path: '/', handler: handler });
    next();
};
```

#### `plugin.app`

Provides access to the [common pack application-specific state](#pack-properties).

```javascript
exports.register = function (plugin, options, next) {

    plugin.app.hapi = 'joi';
    next();
};
```

#### `plugin.events`

The `pack.events' emitter.

```javascript
exports.register = function (plugin, options, next) {

    plugin.events.on('internalError', function (request, err) {

        console.log(err);
    });

    next();
};
```

#### `plugin.plugins`

An object where each key is a plugin name and the value are the exposed properties by that plugin using [`plugin.expose()`](#pluginexposekey-value)
when called at the plugin root level (without calling `plugin.select()`).

```javascript
exports.register = function (plugin, options, next) {

    console.log(plugin.plugins.example.key);
    next();
};
```

#### `plugin.log(tags, [data, [timestamp]])`

Emits a `'log'` event on the `pack.events` emitter using the same interface as [`server.log()`](#serverlogtags-data-timestamp).

```javascript
exports.register = function (plugin, options, next) {

    plugin.log(['plugin', 'info'], 'Plugin registered');
    next();
};
```

#### `plugin.dependency(deps, [after])`

Declares a required dependency upon other plugins where:

- `deps` - a single string or array of strings of plugin names which must be registered in order for this plugin to operate. Plugins listed
  must be registered in the same pack transaction to allow validation of the dependency requirements. Does not provide version dependency which
  should be implemented using [npm peer dependencies](http://blog.nodejs.org/2013/02/07/peer-dependencies/).
- `after` - an optional function called after all the specified dependencies have been registered and before the servers start. The function is only
  called if the pack servers are started. If a circular dependency is created, the call will assert (e.g. two plugins each has an `after` function
  to be called after the other). The function signature is `function(plugin, next)` where:
    - `plugin` - the [plugin interface](#plugin-interface) object.
    - `next` - the callback function the method must call to return control over to the application and complete the registration process. The function
      signature is `function(err)` where:
        - `err` - internal plugin error condition, which is returned back via the [`pack.start(callback)`](#packstartcallback) callback. A plugin
          registration error is considered an unrecoverable event which should terminate the application.

```javascript
exports.register = function (plugin, options, next) {

    plugin.dependency('yar', after);
    next();
};

var after = function (plugin, next) {

    // Additional plugin registration logic
    next();
};
```

#### `plugin.after(method)`

Add a method to be called after all the required plugins have been registered and before the servers start. The function is only
called if the pack servers are started. Arguments:

- `after` - the method with signature `function(plugin, next)` where:
    - `plugin` - the [plugin interface](#plugin-interface) object.
    - `next` - the callback function the method must call to return control over to the application and complete the registration process. The function
      signature is `function(err)` where:
        - `err` - internal plugin error condition, which is returned back via the [`pack.start(callback)`](#packstartcallback) callback. A plugin
          registration error is considered an unrecoverable event which should terminate the application.

```javascript
exports.register = function (plugin, options, next) {

    plugin.after(after);
    next();
};

var after = function (plugin, next) {

    // Additional plugin registration logic
    next();
};
```

#### `plugin.views(options)`

Generates a plugin-specific views manager for rendering templates where:
- `options` - the views configuration as described in the server's [`views`](#server.config.views) option. Note that due to the way node
  `require()` operates, plugins must require rendering engines directly and pass the engine using the `engines.module` option.

Note that relative paths are relative to the plugin root, not the working directory or the application registering the plugin. This allows
plugin the specify their own static resources without having to require external configuration.

```javascript
exports.register = function (plugin, options, next) {

    plugin.views({
        engines: {
            html: { 
              module: require('handlebars') 
            }
        },
        path: './templates'
    });

    next();
};
```

#### `plugin.method(name, fn, [options])`

Registers a server method function with all the pack's servers as described in [`server.method()`](#servermethodname-fn-options)

```javascript
exports.register = function (plugin, options, next) {

    plugin.method('user', function (id, next) {

        next(null, { id: id });
    });

    next();
};
```

#### `plugin.method(method)`

Registers a server method function with all the pack's servers as described in [`server.method()`](#servermethodmethod)

```javascript
exports.register = function (plugin, options, next) {

    plugin.method({
        name: 'user',
        fn: function (id, next) {

            next(null, { id: id });
        }
    });

    next();
};
```

#### `plugin.methods`

Provides access to the method methods registered with [`plugin.method()`](#pluginmethodname-fn-options)

```javascript
exports.register = function (plugin, options, next) {

    plugin.method('user', function (id, next) {

        next(null, { id: id });
    });

    plugin.methods.user(5, function (err, result) {

        // Do something with result

        next();
    });
};
```

#### `plugin.cache(options)`

Provisions a plugin cache segment within the pack's common caching facility where:

- `options` - cache configuration as described in [**catbox** module documentation](https://github.com/spumko/catbox#policy) with a few additions:
    - `expiresIn` - relative expiration expressed in the number of milliseconds since the item was saved in the cache. Cannot be used
      together with `expiresAt`.
    - `expiresAt` - time of day expressed in 24h notation using the 'MM:HH' format, at which point all cache records for the route
      expire. Cannot be used together with `expiresIn`.
    - `staleIn` - number of milliseconds to mark an item stored in cache as stale and reload it. Must be less than `expiresIn`.
    - `staleTimeout` - number of milliseconds to wait before checking if an item is stale.
    - `segment` - optional segment name, used to isolate cached items within the cache partition. Defaults to '!name' where 'name' is the
      plugin name. When setting segment manually, it must begin with '!!'.
    - `cache` - the name of the cache connection configured in the ['server.cache` option](#server.config.cache). Defaults to the default cache.
    - `shared` - if true, allows multiple cache users to share the same segment (e.g. multiple servers in a pack using the same cache. Default
      to not shared.

```javascript
exports.register = function (plugin, options, next) {

    var cache = plugin.cache({ expiresIn: 60 * 60 * 1000 });
    next();
};
```

#### `plugin.require(name, [options], callback)`

Registers a plugin with the same pack as the current plugin following the syntax of [`pack.require()`](#packrequirename-options-callback).

```javascript
exports.register = function (plugin, options, next) {

    plugin.require('furball', { version: '/v' }, function (err) {

        next(err);
    });
};
```

#### `plugin.require(names, callback)`

Registers a list of plugins with the same pack following the syntax of [`pack.require()`](#packrequirename-callback).

```javascript
exports.register = function (plugin, options, next) {

    plugin.require(['furball', 'lout'], function (err) {

        next(err);
    });
};
```

#### `plugin.loader(require)`

Forces using the local `require()` method provided by node when calling `plugin.require()`. This sets the module path relative
to the plugin instead of relative to the hapi framework module location. This is needed to work around the limitations in node's
`require()`.

```javascript
exports.register = function (plugin, options, next) {

    plugin.loader(require);
    plugin.require('furball', { version: '/v' }, function (err) {

        next(err);
    });
};
```

#### `plugin.bind(bind)`

Sets a global plugin bind used as the default bind when adding a route or an extension using the plugin interface (if no
explicit bind is provided as an option). The bind object is made available within the handler and extension methods via `this`.

```javascript
var handler = function (request, reply) {

    request.reply(this.message);
};

exports.register = function (plugin, options, next) {

    var bind = {
        message: 'hello'
    };

    plugin.bind(bind);
    plugin.route({ method: 'GET', path: '/', handler: internals.handler });
    next();
};
```

#### `plugin.handler(name, method)`

Registers a new handler type as describe in [`server.handler(name, method)`](#serverhandlername-method).

```javascript
exports.register = function (plugin, options, next) {

    var handlerFunc = function (route, options) {

        return function (request, reply) {

            reply('Message from plugin handler: ' + options.msg);
        }
    };

    plugin.handler('testHandler', handlerFunc);
    next();
}
```

### Selectable methods and properties

The plugin interface selectable methods and properties are those available both on the `plugin` object received via the
[`exports.register()`](#exportsregisterplugin-options-next) interface and the objects received by calling
[`plugin.select()`](#pluginselectlabels). However, unlike the root methods, they operate only on the selected subset of
servers.

#### `plugin.select(labels)`

Selects a subset of pack servers using the servers' `labels` configuration option where:

- `labels` - a single string or array of strings of labels used as a logical OR statement to select all the servers with matching
  labels in their configuration.

Returns a new `plugin` interface with only access to the [selectable methods and properties](#selectable-methods-and-properties).
Selecting again on a selection operates as a logic AND statement between the individual selections.

```javascript
exports.register = function (plugin, options, next) {

    var selection = plugin.select('web');
    selection.route({ method: 'GET', path: '/', handler: function (request, reply) { reply('ok'); } });
    next();
};
```

#### `plugin.length`

The number of selected servers.

```javascript
exports.register = function (plugin, options, next) {

    var count = plugin.length;
    var selectedCount = plugin.select('web').length;
    next();
};
```

#### `plugin.servers`

The selected servers array.

```javascript
exports.register = function (plugin, options, next) {

    var selection = plugin.select('web');
    selection.servers.forEach(function (server) {

        server.route({ method: 'GET', path: '/', handler: function (request, reply) { reply('ok'); } });
    });

    next();
};
```

#### `plugin.expose(key, value)`

Exposes a property via `plugin.plugins[name]` (if added to the plugin root without first calling `plugin.select()`) and `server.plugins[name]`
('name' of plugin) object of each selected pack server where:

- `key` - the key assigned (`server.plugins[name][key]` or `plugin.plugins[name][key]`).
- `value` - the value assigned.

```javascript
exports.register = function (plugin, options, next) {

    plugin.expose('util', function () { console.log('something'); });
    next();
};
```

#### `plugin.expose(obj)`

Merges a deep copy of an object into to the existing content of `plugin.plugins[name]` (if added to the plugin root without first calling
`plugin.select()`) and `server.plugins[name]` ('name' of plugin) object of each selected pack server where:

- `obj` - the object merged into the exposed properties container.

```javascript
exports.register = function (plugin, options, next) {

    plugin.expose({ util: function () { console.log('something'); } });
    next();
};
```

#### `plugin.route(options)`

Adds a server route to the selected pack's servers as described in [`server.route(options)`](#serverrouteoptions).

```javascript
exports.register = function (plugin, options, next) {

    var selection = plugin.select('web');
    selection.route({ method: 'GET', path: '/', handler: function (request, reply) { reply('ok'); } });
    next();
};
```

#### `plugin.route(routes)`

Adds multiple server routes to the selected pack's servers as described in [`server.route(routes)`](#serverrouteroutes).

```javascript
exports.register = function (plugin, options, next) {

    var selection = plugin.select('admin');
    selection.route([
        { method: 'GET', path: '/1', handler: function (request, reply) { reply('ok'); } },
        { method: 'GET', path: '/2', handler: function (request, reply) { reply('ok'); } }
    ]);

    next();
};
```

#### `plugin.state(name, [options])`

Adds a state definition to the selected pack's servers as described in [`server.state()`](#serverstatename-options).

```javascript
exports.register = function (plugin, options, next) {

    plugin.state('example', { encoding: 'base64' });
    next();
};
```

#### `plugin.auth.scheme(name, scheme)`

Adds an authentication scheme to the selected pack's servers as described in [`server.auth.scheme()`](#serverauthschemename-scheme).

#### `plugin.auth.strategy(name, scheme, [mode], [options])`

Adds an authentication strategy to the selected pack's servers as described in [`server.auth.strategy()`](#serverauthstrategyname-scheme-mode-options).

#### `plugin.ext(event, method, [options])`

Adds an extension point method to the selected pack's servers as described in [`server.ext()`](#serverextevent-method-options).

```javascript
exports.register = function (plugin, options, next) {

    plugin.ext('onRequest', function (request, extNext) {

        console.log('Received request: ' + request.path);
        extNext();
    });

    next();
};
```

## `Hapi.state`

#### `prepareValue(name, value, options, callback)`

Prepares a cookie value manually outside of the normal outgoing cookies processing flow. Used when decisions have to be made about
the use of cookie values when certain conditions are met (e.g. stringified object string too long). Arguments:

- `name` - the cookie name.
- `value` - the cookie value. If no `encoding` is defined, must be a string.
- `options` - configuration override. If the state was previously registered with the server using [`server.state()`](#serverstatename-options),
  the specified keys in `options` override those same keys in the server definition (but not others).
- `callback` - the callback function with signature `function(err, value)` where:
    - `err` - internal error condition.
    - `value` - the prepared cookie value.

Returns the cookie value via callback without making any changes to the response.

```javascript
var Hapi = require('hapi');

var handler = function (request, reply) {

    var maxCookieSize = 512;

    var cookieOptions = {
        encoding: 'iron',
        password: 'secret'
    };

    var content = request.pre.user;

    Hapi.state.prepareValue('user', content, cookieOptions, function (err, value) {

        if (err) {
            return reply(err);
        }

        if (value.length < maxCookieSize) {
            reply.state('user', value, { encoding: 'none' } );   // Already encoded
        }

        reply('success');
    });
};
```

## `Hapi.version`

The **hapi** module version number.

```javascript
var Hapi = require('hapi');
console.log(Hapi.version);
```

## `Hapi CLI`

The **hapi** command line interface allows a pack of servers to be composed and started from a configuration file only from the command line.
When installing **hapi** with the global flag the **hapi** binary script will be installed in the path.  The following arguments are available to the
**hapi** CLI:

- '-c' - the path to configuration json file (required)
- '-p' - the path to the node_modules folder to load plugins from (optional)

In order to help with A/B testing there is [confidence](https://github.com/spumko/confidence).  Confidence is a configuration document format, an API, and a foundation for A/B testing. The configuration format is designed to work with any existing JSON-based configuration, serving values based on object path ('/a/b/c' translates to a.b.c). In addition, confidence defines special $-prefixed keys used to filter values for a given criteria.

