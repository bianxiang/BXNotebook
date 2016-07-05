# （未）4. 构建Node Web应用

* 使用Node API处理HTTP请求
* 构建RESTful web service
* 伺服静态文件和fs filesystem模块
* 接受用户表单输入
* 监控文件上传进度
* 使用HTTPS

Node核心有一个强大的streaming HTTP parser，由1500行优化的C代码构成。Node还将底层的TCP暴露给Javascript。

Node核心的HTTP模块比较简化。高层抽象由三方库完成，如Connect或Express。

## 4.1 HTTP服务器基础

Node的HTTP接口比较底层。

### 4.1.1 如何把HTTP请求封装给开发者

http模块提供HTTP服务器和客户端接口。通过`http.createServer()`创建一个HTTP服务器。它接受一个回调方法，负责处理收到的每一个请求。回调方法接受请求和响应两个对象。

	var http = require('http');
	var server = http.createServer(function(req, res){
		// handle request
	});

每个到达服务器的请求，在HTTP被解析后，传给回调方法。

### 4.1.2 一个基本的HTTP服务器

调用响应对象的`res.write()`方法写响应，用`res.end()`结束响应。

	var http = require('http');
	var server = http.createServer(function(req, res){
		res.write('Hello World');
		res.end();
	});

`http.createServer()`返回的实例是`http.Server`，它继承自`net.Server`。用`server.listen()`方法监听连接，这个方法实际定义在`net`模块，在TCP层面。 该方法接受多种参数，目前我们只使用端口和IP地址。

	var http = require('http');
	var server = http.createServer(function(req, res){
		res.end('Hello World');
	});
	server.listen(3000, '127.0.0.1');

### 4.1.3 读请求头、写响应头

设置、读取、移除响应头：`res.setHeader(field, value)`, `res.getHeader(field)`, `res.removeHeader(field)`。这些方法要在调用`res.write()`和`res.end()`之前调用。开始写响应体后，**Node将刷出HTTP头**。

	var body = 'Hello World';
	res.setHeader('Content-Length', body.length);
	res.setHeader('Content-Type', 'text/plain');
	res.end(body);

### 4.1.4 设置HTTP响应状态码

`res.statusCode`要在`res.write()`和`res.end()`之前调用。但可以在设置响应头之前或之后调用。

	var url = 'http://google.com';
	var body = '<p>Redirecting to <a href="' + url + '">'
		+ url + '</a></p>';
	res.setHeader('Location', url);
	res.setHeader('Content-Length', body.length);
	res.setHeader('Content-Type', 'text/html);
	res.statusCode = 302;
	res.end(body);

## 4.2 构建RESTful Web服务

### 4.2.1 使用POST创建资源

使用`req.method`可以检查请求的HTTP方法（动词）。

Node的`http-parser`流数据时，你能收到数据块。数据块可能是`Buffer`对象或字符串。默认收到的是`Buffer`对象，即Node版的字节数组。如果先调用`req.setEncoding(encoding)`设置内容编码（如utf8），则请求将emit字符串。 

在读到全部请求内容后写响应，为此，可以监听"end"事件。

	var http = require('http');
	var url = require('url');
	var items = [];
	var server = http.createServer(function(req, res){
		switch (req.method) {
		case 'POST':
			var item = '';
			req.setEncoding('utf8');
			req.on('data', function(chunk){
				item += chunk;
			});
			req.on('end', function(){
				items.push(item);
				res.end('OK\n');
			});
			break;
		}
	});

### 4.2.2 通过GET获取资源

	...
	case 'GET':
		items.forEach(function(item, i){
			res.write(i + ') ' + item + '\n');
		});
		res.end();
		break;
	...

#### 设置Content-length头

为加速响应，应为响应设置合适的`Content-Length`。设置`Content-Length`头能禁用Node的"chunked"编码，导致传输数据减少，因而提高性能。

	var body = items.map(function(item, i){
		return i + ') ' + item;
	}).join('\n');
	res.setHeader('Content-Length', Buffer.byteLength(body));
	res.setHeader('Content-Type', 'text/plain; charset="utf-8"');
	res.end(body);

注意`Content-Length`的值应该是字节长度。利用`Buffer.byteLength()`方法。


### 4.2.3 使用DELETE请求删除资源

请求URL通过`req.url`属性获得。例如，如果请求是`DELETE /1?api-key=foobar`，则该属性是`/1?api-key=foobar`。要解析此url，Node提供url模块`.parse()`方法。

	$ node
	> require('url').parse('http://google.com?q=tobi')
	{ protocol: 'http:',
		slashes: true,
		host: 'google.com',
		hostname: 'google.com',
		href: 'http://google.com/?q=tobi',
		search: '?q=tobi',
		query: 'q=tobi',
		pathname: '/' }

代码：

	...
	case 'DELETE':
		var path = url.parse(req.url).pathname;
		var i = parseInt(path.slice(1), 10);
		if (isNaN(i)) {
			res.statusCode = 400;
			res.end('Invalid item id');
		} else if (!items[i]) {
			res.statusCode = 404;
			res.end('Item not found');
		} else {
			items.splice(i, 1);
			res.end('OK\n');
		}
		break;
	...

## 4.3 伺服静态文件（文件系统API）

编写鲁棒的高效的静态文件服务器不是很简单的，Node社区已有鲁棒的实现。本节实现静态文件服务器只是为介绍Node底层的**文件系统API**。

### 4.3.1 创建静态文件服务器

root变量定义一个静态文件的根目录。

	var http = require('http');
	var parse = require('url').parse;
	var join = require('path').join;
	var fs = require('fs');
	var root = __dirname;
	...

`__dirname`是Node定义的目录，表示你的脚本所在的目录。

如果请求路径是"/index.html"，根路径是"/var/www/example.com/public"，可以使用"path"模块的`.join()`方法构建绝对路径"/var/www/example.com/public/index.html"。

	var http = require('http');
	var parse = require('url').parse;
	var join = require('path').join;
	var fs = require('fs');
	var root = __dirname;
	var server = http.createServer(function(req, res){
		var url = parse(req.url);
		var path = join(root, url.pathname);
	});
	server.listen(3000);

利用`fs.ReadStream`读文件内容：

	var http = require('http');
	var parse = require('url').parse;
	var join = require('path').join;
	var fs = require('fs');
	var root = __dirname;
	var server = http.createServer(function(req, res) {
		var url = parse(req.url);
		var path = join(root, url.pathname);
		var stream = fs.createReadStream(path);
		stream.on('data', function(chunk){
			res.write(chunk);
		});
		stream.on('end', function(){
			res.end();
		});
	});
	server.listen(3000);

#### 使用STREAM.PIPE()优化数据传输

`Stream#pipe()`类似于命令行的管道符。它能显著简化代码，也能透明的处理TCP back pressure。

	var server = http.createServer(function(req, res){
		var url = parse(req.url);
		var path = join(root, url.pathname);
		var stream = fs.createReadStream(path);
		stream.pipe(res);
	});

### 4.3.2 处理服务器错误

目前服务器没有处理`fs.ReadStream`可能抛出的错误。错误可能是文件不存在，没有访问权，IO错误。

默认，若无任何监听器，Node将抛出"error"事件——所有的`EventEmitter`都是这样的，`fs.ReadStream`也不例外。如果不处理error事件将挂掉整个服务器。To illustrate this try requesting a file that does not exist such as "/notfound.js". In the terminal session running your server you'll see the stack trace of an exception printed to stderr similar to the following: 

	stream.js:99
	throw arguments[1]; // Unhandled 'error' event.
	^
	Error: ENOENT, No such file or directory
	'/Users/tj/projects/node-in-action/source/notfound.js'

为避免挂掉服务器，为`fs.ReadStream`注册一个`error`事件处理器｛｛使用`pipe`时注意注册额外的是错误处理器｝｝：

	...
	stream.pipe(res);
	stream.on('error', function(err){
		res.statusCode = 500;
		res.end('Internal Server Error');
	});
	...

对于静态文件，可以使用系统调用`stat()`获取文件信息，如修改时间，字节大小。这些信息可以为条件GET提供支持。Connect中间件具备这些支持，见第七章。

下面的代码利用`fs.stat()`先检查文件。根据`err.code`判断错误原因。同时利用`stat()`确定文件大小。

	var server = http.createServer(function(req, res){
		var url = parse(req.url);
		var path = join(root, url.pathname);
		fs.stat(path, function(err, stat){
			if (err) {
				if ('ENOENT' == err.code) {
					res.statusCode = 404;
					res.end('Not Found');
				} else {
					res.statusCode = 500;
					res.end('Internal Server Error');
				}
			} else {
				res.setHeader('Content-Length', stat.size); // 文件大小
				var stream = fs.createReadStream(path);
				stream.pipe(res);
				stream.on('error', function(err){
					res.statusCode = 500;
					res.end('Internal Server Error');
				});
			}
		});
	});
	
	
	





