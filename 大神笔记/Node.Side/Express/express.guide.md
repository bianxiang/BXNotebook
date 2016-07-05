http://expressjs.com/guide.html

[toc]

## 入门

package.json

    {
      "name": "hello-world",
      "description": "hello world test app",
      "version": "0.0.1",
      "private": true,
      "dependencies": {
        "express": "3.x"
      }
    }

创建一个`app.js`或`server.js`文件（或其他名字）。调用`express()`创建一个新应用

	var express = require('express');
	var app = express();


利用`app.VERB()`创建路由（如`app.get()`用于*GET*方法，`app.post()`用于*POST*方法）。`req`和`res`是node提供的对象，因此你可以调用`res.pipe()`，`req.on('data', callback)`等方法。Express增强了这些变量。例如`res.send()`可以帮你设置`Content-Type`。

	app.get('/hello.txt', function(req, res) {
	  res.send('Hello World');
	});

`app.listen()`监听特定端口，与node的`net.Server#listen()`接受相同的参数：

    var server = app.listen(3000, function() {
        console.log('Listening on port %d', server.address().port);
    });

## 利用`express(1)`产生应用

The Express team maintains a handy quickstart project generator aptly named express(1). If you install `express-generator` globally with npm you'll have it available from anywhere on your machine:

    $ npm install -g express-generator

If you want to generate an application with Jade and Stylus support you would simply execute:

	$ express --css stylus myapp
	
	create : myapp
	create : myapp/package.json
	create : myapp/app.js
	create : myapp/public
	create : myapp/public/javascripts
	create : myapp/public/images
	create : myapp/public/stylesheets
	create : myapp/public/stylesheets/style.styl
	create : myapp/routes
	create : myapp/routes/index.js
	create : myapp/views
	create : myapp/views/index.jade
	create : myapp/views/layout.jade
	
	install dependencies:
	  $ cd myapp && npm install
	  
	run the app:
	  $ DEBUG=myapp node app

先安装依赖：

	$ cd myapp
	$ npm install

启动
	
	$ npm start

## 错误处理

错误处理中间件与普通中间件一样，只是需要能接受四个参数`(err, req, res, next)`。

	app.use(function(err, req, res, next){
	  console.error(err.stack);
	  res.send(500, 'Something broke!');
	});

错误处理中间件一般定义在最后

	app.use(express.bodyParser());
	app.use(express.methodOverride());
	app.use(app.router);
	app.use(function(err, req, res, next){
	  // logic
	});

错误处理中间件的响应是任意的，可以响应HTML、JSON等任意格式。

For organizational, and higher-level framework purposes you may define several of these error-handling middleware.

	app.use(express.bodyParser());
	app.use(express.methodOverride());
	app.use(app.router);
	app.use(logErrors);
	app.use(clientErrorHandler);
	app.use(errorHandler);

其中，`logErrors`强错误信息写到stderr, loggly等地方。

	function logErrors(err, req, res, next) {
	  console.error(err.stack);
	  next(err);
	}

其中，`clientErrorHandler`。如果不是它关心的错误，就调用`next(err)`交给下一个错误处理中间件。否则结束。

	function clientErrorHandler(err, req, res, next) {
	  if (req.xhr) {
	    res.send(500, { error: 'Something blew up!' });
	  } else {
	    next(err);
	  }
	}

最后`errorHandler`是一个*catch-all*处理器：

	function errorHandler(err, req, res, next) {
	  res.status(500);
	  res.render('error', { error: err });
	}

## 用户在线统计

使用Redis。首先在`package.json`加入对*redis client*的依赖。

	{
	  "name": "app",
	  "version": "0.0.1",
	  "dependencies": {
	    "express": "3.x",
	    "redis": "*"
	  }
	}

创建app：

	var express = require('express');
	var redis = require('redis');
	var db = redis.createClient();
	var app = express();

Next up is the middleware for tracking online users. Here we'll use sorted sets so that we can query redis for the users online within the last N milliseconds. We do this by passing a timestamp as the member's "score". Note that here we're using the User-Agent string in place of what would normally be a user id.

	app.use(function(req, res, next){
	  var ua = req.headers['user-agent'];
	  db.zadd('online', Date.now(), ua, next);
	});

This next middleware is for fetching the users online in the last minute using zrevrangebyscore to fetch with a positive infinite max value so that we're always getting the most recent users, capped with a minimum score of the current timestamp minus 60,000 milliseconds.

	app.use(function(req, res, next){
	  var min = 60 * 1000;
	  var ago = Date.now() - min;
	  db.zrevrangebyscore('online', '+inf', ago, function(err, users){
	    if (err) return next(err);
	    req.online = users;
	    next();
	  });
	});

Then finally we use it, and bind to a port! That's all there is to it, visit the app in a few browsers and you'll see the count increase.

	app.get('/', function(req, res){
	  res.send(req.online.length + ' users online');
	});

	app.listen(3000);

## 代理后面的Express

在Ngnix等反向带来后面使用，需要配置。调用`app.enable('trust proxy')`启用*trust proxy*，告诉Express它在代理后面，and that the `X-Forwarded-*` header fields may be trusted, which otherwise may be easily spoofed.

几个效果。The first of which is that `X-Forwarded-Proto` may be set by the reverse proxy to tell the app that it is https or simply http. This value is reflected by `req.protocol`.

第二，`req.ip`和`req.ips`将填入`X-Forwarded-For`中的一组地址。

## 调试Express

Express使用[debug](https://github.com/visionmedia/debug)模块记录路由和应用模式的日志。要看到这些信息，设置环境变量`DEBUG`为`express:*`，调试信息将显示在控制台：

	$ DEBUG=express:* node ./bin/www

Running this on the hello world example would print the following

	express:application booting in development mode +0ms
	express:router defined get /hello.txt +0ms
	express:router defined get /hello.txt +1ms

Additionally, the app generated by the express executable (generator) also uses the debug module and by default is scoped to the `my-application` debug namespace.

You can enable those debug statements with the following command

	$ DEBUG=my-application node ./bin/www






