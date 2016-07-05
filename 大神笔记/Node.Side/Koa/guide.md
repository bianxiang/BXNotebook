[toc]

https://github.com/koajs/koa/blob/master/docs/guide.md

## Guide

本文介绍编写中间件的最佳实践，以及对应用结构的建议。

### 编写中间件

Koa中间件是一个函数，返回一个产生器函数，接收另一个产生器。When the middleware is run by an "upstream" middleware, it must manually `yield` to the "downstream" middleware.

For example if you wanted to track how long it takes for a request to propagate through Koa by adding an `X-Response-Time` header field the middleware would look like the following:

```js
    function *responseTime(next) {
      var start = new Date;
      yield next;
      var ms = new Date - start;
      this.set('X-Response-Time', ms + 'ms');
    }

    app.use(responseTime);
```

### 中间件最佳实践

#### 中间件选项

最好将中间件外包围一个函数，函数接受选项，让用户定制中间件。

Here our contrived `logger` middleware accepts a `format` string for customization, and returns the middleware itself:

```js
    function logger(format) {
      format = format || ':method ":url"';

      return function *(next) {
        var str = format
          .replace(':method', this.method)
          .replace(':url', this.url);

        console.log(str);

        yield next;
      }
    }

    app.use(logger());
    app.use(logger(':method :url'));
```

#### 命名中间件

Naming middleware is optional, however it's useful for debugging purposes to assign a name.

```js
    function logger(format) {
      return function *logger(next){

      }
    }
```

#### 组合多个中间件

Sometimes you want to "compose" multiple middleware into a single middleware for easy re-use or exporting. To do so, you may chain them together with `.call(this, next)`s, then return another function that yields the chain.

```js
    function *random(next) {
      if ('/random' == this.path) {
        this.body = Math.floor(Math.random()*10);
      } else {
        yield next;
      }
    };

    function *backwards(next) {
      if ('/backwards' == this.path) {
        this.body = 'sdrawkcab';
      } else {
        yield next;
      }
    }

    function *pi(next) {
      if ('/pi' == this.path) {
        this.body = String(Math.PI);
      } else {
        yield next;
      }
    }

    function *all(next) {
      yield random.call(this, backwards.call(this, pi.call(this, next)));
    }

    app.use(all);
```

This is exactly what [koa-compose](https://github.com/koajs/compose) does, which Koa internally uses to create and dispatch the middleware stack.

#### 响应中间件

Middleware that decide to respond to a request and wish to bypass downstream middleware may simply omit `yield next`. Typically this will be in routing middleware, but this can be performed by any. For example the following will respond with "two", however all three are executed, giving the downstream "three" middleware a chance to manipulate the response.

```js
    app.use(function *(next){
      console.log('>> one');
      yield next;
      console.log('<< one');
    });

    app.use(function *(next){
      console.log('>> two');
      this.body = 'two';
      yield next;
      console.log('<< two');
    });

    app.use(function *(next){
      console.log('>> three');
      yield next;
      console.log('<< three');
    });
```

The following configuration omits `yield next` in the second middleware, and will still respond with "two", however the third (and any other downstream middleware) will be ignored:

```js
    app.use(function *(next){
      console.log('>> one');
      yield next;
      console.log('<< one');
    });

    app.use(function *(next){
      console.log('>> two');
      this.body = 'two';
      console.log('<< two');
    });

    app.use(function *(next){
      console.log('>> three');
      yield next;
      console.log('<< three');
    });
```

When the furthest downstream middleware executes `yield next;` it's really yielding to a noop function, allowing the middleware to compose correctly anywhere in the stack.

### 异步操作

The [Co](https://github.com/visionmedia/co) forms Koa's foundation for generator delegation, allowing you to write non-blocking sequential code. For example this middleware reads the filenames from `./docs`, and then reads the contents of each markdown file in parallel before assigning the body to the joint result.

```js
    var fs = require('co-fs');

    app.use(function *(){
      var paths = yield fs.readdir('docs');
      var files = yield paths.map(function(path) {
        return fs.readFile('docs/' + path, 'utf8');
      });

      this.type = 'markdown';
      this.body = files.join('');
    });
```

### 调试Koa

Koa along with many of the libraries it's built with support the __DEBUG__ environment variable from [debug](https://github.com/visionmedia/debug) which provides simple conditional logging.

For example to see all koa-specific debugging information just pass `DEBUG=koa*` and upon boot you'll see the list of middleware used, among other things.

```
$ DEBUG=koa* node --harmony examples/simple
  koa:application use responseTime +0ms
  koa:application use logger +4ms
  koa:application use contentLength +0ms
  koa:application use notfound +0ms
  koa:application use response +0ms
  koa:application listen +0ms
```

Since JavaScript does not allow defining function names at runtime, you can also set a middleware's name as `._name`. This useful when you don't have control of a middleware's name. For example:

```js
    var path = require('path');
    var static = require('koa-static');

    var publicFiles = static(path.join(__dirname, 'public'));
    publicFiles._name = 'static /public';

    app.use(publicFiles);
```

Now, instead of just seeing "static" when debugging, you will see:

```
  koa:application use static /public +0ms
```