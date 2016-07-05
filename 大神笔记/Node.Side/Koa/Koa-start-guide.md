[toc]

## Koa入门

参考：http://code.tutsplus.com/tutorials/introduction-to-generators-koajs-part-1--cms-21615

本文涉及路由、压缩、日志、用量控制、错误处理，用到MongoDb。

Koa.js利用ES6的generator，消除回调，提高代码的可维护性，降低错误可能。

Koa只有550行代码。Koa自带的功能有content-negotiation、重定向、代理支持等。

Koa引入了定制对象，如`this`、`this.request`和`this.response`。

Koa清理了中间件（in Express, relied on ugly hacks which often modified core objects）。提供更高的流处理。

Koa的中间件是一个产生器函数，that returns one generator function and accepts another。Also, a middleware must yield to the next 'downstream' middleware if it is run by an 'upstream middleware'.

要添加中间件，利用`koa.use()`方法，传入中间件函数。例如，`app.use(koa-logger)`添加了`koa-logger`中间件。

### 安装运行

需要Node 0.11.x及以上版本。

因为用到generators，运行时要加`--harmony`标志，如：

	node --harmony app.js

安装 koa：

	npm install koa --save

第一个例子：

```js
    var koa = require('koa');
    var app = koa();

    app.use(function *(){
        this.body = "Hello World !!!";
    });

    app.listen(3000);
```

### 路由

用到中间件 `koa-router` 和 `koa-mount`。

```js
    var koa = require('koa');
    var app = koa();
    var router = require('koa-router');
    var mount = require('koa-mount');

    var handler = function *(next){
        this.type = 'json';
        this.status = 200;
        this.body = {'Welcome': 'This is a level 2 Hello World Application!!'};
    };

    var APIv1 = new router();
    APIv1.get('/all', handler);

    app.use(mount('/v1', APIv1.middleware()));
    if (!module.parent) app.listen(3000);
    console.log('Hello World is Running on http://localhost:3000/');
```

### 后端逻辑

api.js

```js
    var monk = require('monk');
    var wrap = require('co-monk');
    var db = monk('localhost/mydb');
    var words = wrap(db.get('words'));
    /**
    * GET all the results.
    */
    exports.all = function *() {
        if(this.request.query.word) {
        	var res = yield words.find({ word : this.request.query.word });
        	this.body = res;
        } else {
        	this.response.status = 404;
        }
    };
    /**
    * GET a single result.
    */
    exports.single = function *() {
        if(this.request.query.word) {
        	var res = yield words.findOne({ word : this.request.query.word });
        	this.body = res;
        } else {
        	this.response.status = 404;
        }
    };
```

Here we are using `co-monk`, which acts a wrapper around `monk`, making it very easy for us to query MongoDB using generators in Koa. We call `wrap()` on collections, to make them generator-friendly.

挂载到路由器，在index.js文件中：

```js
    var koa = require('koa');
    var app = koa();
    var router = require('koa-router');
    var mount = require('koa-mount');

    var api = require('./api/api.js');
    var APIv1 = new router();
    APIv1.get('/all', api.all);
    APIv1.get('/single', api.single);

    app.use(mount('/v1', APIv1.middleware()));
    if (!module.parent) app.listen(3000);
    console.log('Dictapi is Running on http://localhost:3000/');
```

### 错误处理

By using cascading middlewares, we can catch errors using the try/catch mechanism, as each middleware can respond while yielding to downstream as well as upstream. 因此如果我们在应用**最前面**加一个*Try and Catch*中间件，我们将能够捕获所有错误。

```js
    app.use(function *(next){
        try{
            yield next; // pass on the execution to downstream middlewares
        } catch (err) {
            // executed only when an error occurs & no other middleware responds to the request
            this.type = 'json'; //optional here
            this.status = err.status || 500;
            this.body = { 'error' : 'The application just went bonkers, hopefully NSA has all the logs ;) '};
            // delegate the error back to application
            this.app.emit('error', err, this);
        }
    });
```

### 日志和用量控制

利用社区贡献的模块`koa-logger`和`koa-better-rate-limiting`。

```js
    var logger = require('koa-logger');
    var limit = require('koa-better-ratelimit');
    // Add the lines below just under error middleware.
    app.use(limit({ duration: 1000*60*3 , // 3 min
                    max: 10, blacklist: []}));
    app.use(logger());
```

### 压缩

gzip压缩请求。

```js
    var compress = require('koa-compress');
    var opts =  {
        filter: function (content_type) { return /text/i.test(content_type) }, // filter requests to be compressed using regex 
        threshold: 2048, //minimum size to compress
        flush: require('zlib').Z_SYNC_FLUSH };
                }
    //use the code below to add the middleware to the application
    app.use(compress(opts));
```

You can even turn off compression in a request by adding the following code to a middleware: `this.compress = true;`.

### 运行

To run our applications in production, we will use **PM2**, which is an useful Node process monitor. We should disable the logger app while in production; it can be automated using environment variables.

To install PM2, enter the following command in terminal

	$ npm install pm2 -g
And our app can be launched using the following command:

	$ pm2 start index.js --node-args="--harmony"

Now, even if our application crashes, it will restart automatically and you can sleep soundly.

