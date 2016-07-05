[toc]

## 13. Beyond web servers

### 13.1 Socket.IO

Socket.IO 允许你编写实时Web应用，利用服务器和客户端之间的双向的信道。Socket.IO 的API与 WebSocket API(http://www.websocket.org)非常像。但也支持老的浏览器。Socket.IO also provides convenient APIs for broadcasting, volatile messages, and a lot more.

本节将搭建两个 Socket.IO 应用：

- A minimal Socket.IO application that pushes the server’s time to connected clients
- A Socket.IO application that triggers page refreshes when CSS files are edited

#### 13.1.1 一个最小的 Socket.IO 应用

连续的更新浏览器中显示的服务器的时间。先安装

	npm install socket.io

下面是服务端的代码：

    var app = require('http').createServer(handler);
    // 将普通的HTTP服务器升级为Socket.io服务器
    var io = require('socket.io').listen(app);
    var fs = require('fs');
    var html= fs.readFileSync('index.html','utf8');
    // 传统HTTP服务器功能：提供Html页面
    function handler(req,res){
    	res.setHeader('Content-Type','text/html');
    	res.setHeader('Content-Length', Buffer.byteLength(html,'utf8'));
    	res.end(html);
    }
    function tick(){
    	var now = newDate().toUTCString();
    	io.sockets.send(now);
    }
    setInterval(tick, 1000);
    app.listen(8080);

客户端代码：

    <!DOCTYPE html>
    <html>
    <head>
    	<script type="text/javascript" src="/socket.io/socket.io.js">
    </script>
    <script type="text/javascript">
        var socket= io.connect();
        socket.on('message', function(time) {
        	document.getElementById('time').innerHTML= time;
        });
    </script>
    </head>
    <body>Currentservertimeis:<b><span id="time"></span></b>
    </body>
    </html>

> 其他种类的消息
Sending a message to all the connected sockets is only one way that Socket.IO enables you to interact with connected users. You can also send messages to individual sockets, broadcast to all sockets except one, send volatile (optional) messages, and a lot more. Be sure to check out Socket.IO’s documentation for more information (http://socket.io/#how-to-use).

#### 13.1.2 利用 Socket.IO 实现刷新页面和CSS

All the web browsers that have that page open automatically reload the changes in the CSS sheet.

We’ll use `fs.watchFile()` in  this  example  instead  of  the  newer `fs.watch()` because we’re assured this code will work the same on all platforms, but we’ll cover the behavior of `fs.watch()` in depth later.

> FS.WATCHFILE()  VS.  FS.WATCH()
Node.js提供了两种监控文件的方式：[fs.watchFile()](http://mng.bz/v6dA)在资源方面是相对昂贵的，但是在各个平台下更可靠。[fs.watch()](http://mng.bz/5KSC)为每个平台都高度优化，but it has behavioral differences on certain platforms. We’ll go over these functions in greater detail in section 13.3.2.

本例中，就爱那个组合使用 Express 和 Socket.IO。

服务器端代码：

    var fs = require('fs');
    var url = require('url');
    var http = require('http');
    var path = require('path');
    var express = require('express');
    var app = express();
    var server = http.createServer(app);
    var io = require('socket.io').listen(server);
    var root = __dirname;
    app.use(function(req, res, next){
    	req.on('static', function(){
            var file = url.parse(req.url).pathname;
            var mode = 'stylesheet';
            if(file[file.length - 1] == '/') {
                file += 'index.html';
                mode= 'reload';
            }
            createWatcher(file, mode);
        });
        next();
    });
    app.use(express.static(root));
    var watchers = {};
    function createWatcher(file, event){
        var absolute = path.join(root, file);
        if(watchers[absolute]){
            return;
        }
        fs.watchFile(absolute, function(curr, prev) {
            if(curr.mtime !== prev.mtime){
            	io.sockets.emit(event, file);
            }
        });
        watchers[absolute]= true;
    }
    server.listen(8080);

客户端代码：

    <!DOCTYPE html>
    <html>
    <head>
    	<title>Socket.IO dynamically reloading CSS stylesheets</title>
    	<link rel="stylesheet" type="text/css" href="/header.css" />
    	<link rel="stylesheet" type="text/css" href="/styles.css" />
    	<script type="text/javascript" src="/socket.io/socket.io.js">
    	</script>
    	<script type="text/javascript">
            window.onload = function(){
            	var socket= io.connect();
            	socket.on('reload',function() {
            		window.location.reload();
            	});
            	socket.on('stylesheet', function (sheet) {
                    var link = document.createElement('link');
                    var head = document.getElementsByTagName('head')[0];
                    link.setAttribute('rel','stylesheet');
                    link.setAttribute('type','text/css');
                    link.setAttribute('href',sheet);
                    head.appendChild(link);
                });
            }
        </script>
    </head>
    <body>
        <h1>This is our Awesome Web page!</h1>
        <div id="body">
        <p>If this file(<code>index.html</code>)is edited, then the
        serverwillsenda messageto the browser using Socket.IO telling
        ittorefreshthepage.</p>
        <p>Ifeitherofthestylesheets(<code>header.css</code> or
        <code>styles.css</code>)areedited, then the server will send a
        messagetothebrowserusingSocket.IO telling it to dynamically
        reloadtheCSS,withoutrefreshingthe page.</p>
        </div>
        <div id="event-log"></div>
    </body>
    </html>

上例中，`reload`和`stylesheet`是我们的自定义事件。This demonstrates how the socket object acts like a **bidirectional** EventEmitter, which you can use to emit events that Socket.IO will transfer across the wire for you.


#### 13.1.3 Socket.IO的其他使用

Back in chapter 4 we said that Socket.IO would be great for relaying upload progress events back to the browser for the user to see. Using a custom `progress` event would work well:

    form.on('progress', function(bytesReceived, bytesExpected) {
        var percent = Math.floor(bytesReceived / bytesExpected* 100);
        socket.emit('progress',{ percent:percent });
    });

### 13.2 TCP/IP 深入

一些网络协议要求值以二进制形式读写。但JavaScript没有任何原生二进制类型。有些人用字符串解决该问题。Node的解决方式是实现了自己的`Buffer`数据类型，它是一个固定长度的二进制数据。

#### 13.2.1 Buffer和二进制数据

`Buffer`是固定长度的二进制数据。`Buffer`可以被看做是C函数`malloc()`的等价，或C++的`new`关键字。`Buffer`对象是快速轻量的，整个Node核心API都在使用。例如所有流的`data`事件返回的都是它。`Buffer`构造器是Node全局函数，目的是让它作为Javascript语言的扩展。你可以把`Buffer`对象看做数组，只是它不能被重新设置大小，只能容纳0到255的值。

##### 文本数据 VS. 二进制数据

假设你想把数字`121234869`存放在一个`Buffer`中。默认，Node认为你想把`Buffer`中的数据按文本使用，因此你可以直接把字符串`"121234869"`传给`Buffer`构造器。一个新的`Buffer`对象会被分配，字符串会写入其中：

    var b = new Buffer("121234869");
    console.log(b.length);
    9
    console.log(b);
    <Buffer 313231323334383639>

{{表示成人可读的形式还是机器可读的形式}} 这个例子，将会返回一个9字节的`Buffer`。因为字符串默认使用**人可读的编码**（UTF-8）写入`Buffer`。 Node还提供帮助方法读写二进制整数（**机器可读**）。它们用来实现收发原始数据类型（如int, float, double等）的机器协议。例如这里，更高效的方法是使用函数`writeInt32LE()`将数字`121234869`写成**机器可读的**二进制整数，最终是4字节的`Buffer`。

`Buffer`的帮助方法有一些变体，如：

- `writeInt16LE()` for smaller integer values
- `writeUInt32LE()` for unsigned values
- `writeInt32BE()` for big-endian values

例子：

    var b = new Buffer(4);
    b.writeInt32LE(121234869, 0);
    console.log(b.length);
    4
    console.log(b);
    <Bufferb 5e53907>

二进制整数比字符串有显著的空间效率。

> BYTE ENDIANNESS The term endiannessrefers to the order of the bytes within a multibyte sequence. When bytes are in little-endian order, the least significant byte (LSB) is stored first, and the byte sequence is read from right to left. Conversely, big-endian order is when the first byte stored is the most significant byte (MSB) and the byte sequence is  read from left to right. Node.js offers equivalent helper functions for both little-endian and big-endian data types.

#### 13.2.2 创建 TCP 服务器

Node的http模块基于net模块。

##### 写数据

The `net` module offers a raw TCP/IP socket interface for your applications to use. 可以调用`net.createServer()`创建TCP服务器。传入一个函数，每当有连接来时会调用此函数。函数只有一个参数，一般名为`socket`，是一个`Socket`对象。

Node服务器端和客户端都是用`Socket`类。它是`Stream`的子类，可读可写（双工）。即，有数据来时发出`data`事件；用`write()`和`end()`发送数据。

    var net= require('net');
    net.createServer(function(socket){
    	socket.write('HelloWorld!\r\n');
    	socket.end();
    }).listen(1337);
    console.log('listening on port 1337');

用**netcat**连接：

	$ netcat localhost 1337

##### 读数据

因为Socket与HTTP的`req`对象一样都是流，因此也可以通过`data`事件读取数据：

    socket.on('data', function(data){
    	console.log('got "data"', data);
    });

默认对socket未设编码，因此`data`参数将是一个`Buffer`对象。但如果你设置了`setEncoding()`，则`data`参数是一个解码后的字符串。

通过`end`事件可以获知客户端关闭了连接：

    socket.on('end', function(){
	    console.log('socket has ended');
    });

You can easily write a quick TCPclient that looks up the version string of the given SSH server by simply waiting for the first dataevent:

    var net = require('net');
    var socket = net.connect({host:process.argv[2], port: 22});
    socket.setEncoding('utf8');
    socket.once('data', function(chunk){
    	console.log('SSH server version:%j', chunk.trim());
    	socket.end();
    });

##### 使用SOCKET.PIPE()连接两个流

In fact, if you wanted to write a basic TCP server that echoed everything that was sent to it back to the client, you could do that with a single line of code in your callback function:

	socket.pipe(socket);

##### 不干净的断开连接

如果客户端没有干净的关闭连接，将不会触发`end`事件，只会触发`close`事件。In the case of netcat, this would happen when you press Ctrl-C to kill the process, rather than pressing Ctrl-D to cleanly close the connection. To detect this situation, you listen for the `close` event:

    socket.on('close',function(){
        console.log('client disconnected');
    });

##### 综合例子：一个Echo服务器

    var net= require('net');
    net.createServer(function(socket){
    	console.log('socket connected!');
    	socket.on('data', function(data){
    		console.log('"data"event',data);
    	});
        socket.on('end', function(){
        	console.log('"end"event');
        });
        socket.on('close', function(){
        	console.log('"close"event');
        });
        socket.on('error', function(e){
        	console.log('"error"event',e);
        });
        socket.pipe(socket);
    }).listen(1337);


#### 13.2.3 创建 TCP 客户端

利用`net.connect()`连接到TCP服务器；它返回一个socket对象。要监听到`connect`事件后再工作。

    var net = require('net');
    var socket = net.connect({port:1337, host: 'localhost'});
    socket.on('connect',function(){
        //begin writing your "request"
        socket.write('HELOlocal.domain.name\r\n');
        ...
    });

Once the socket instance is connected to the server, it behaves just like the socket instances you get inside a `net.Server` callback function.

一个简单的类似netcat的例子：

    var net = require('net');
    var host = process.argv[2];
    var port = Number(process.argv[3]);
    var socket = net.connect(port,host);
    socket.on('connect', function(){
        process.stdin.pipe(socket);
        socket.pipe(process.stdout);
        process.stdin.resume();
    });
    socket.on('end', function(){
    	process.stdin.pause();
    });

### （及以下未）13.3 Tools for interacting with the operating system
