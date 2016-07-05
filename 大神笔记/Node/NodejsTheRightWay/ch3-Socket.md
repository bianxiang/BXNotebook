[toc]

## 3. 网络和Socket

### 3.1 监听Socket连接

#### 服务器端

绑定一个TCP端口监听连接：

```js
    "use strict";
    const
      net=require('net'),
     server = net.createServer(function(connection){
      // use connection object for data transfer
    });
    server.listen(5432);
```

当有连接请求时，Node调用回调方法。`connection`参数是一个`Socket`对象。

#### 向Socket写数据

Let’s reuse the file changes as a source of information for our example networked  service.  This  will  give  us  something  to  code  against  as  we  dig  into aspects of Node.js development.

```js
    'use strict';
    const
      fs=require('fs'),
      net=require('net'),
      filename=process.argv[2],
    server=net.createServer(function(connection){
      // reporting
      console.log('Subscriber connected.')
      connection.write("Now watching'" + filename + "'for changes...\n");
      // watcher setup
      let watcher = fs.watch(filename, function() {
        connection.write("File'" + filename + "'changed:" + Date.now() + "\n");
      });
      // cleanup
      connection.on('close', function(){
        console.log('Subscriber disconnected.');
        watcher.close();
      });
    });
    if(!filename) {
      throw Error('No target filename wass pecified.')
    }
    server.listen(5432, function() {
      console.log('Listening for subscribers...');
    });
```

#### 利用Telnet连接TCP Socket

	$node --harmony net-watcher.js target.txt

	$telnet localhost 5432

#### 监听Unix Socket

TCP socket适合网络计算机之间的通讯。但对于进程间通讯，Unix socket是更高效的选择。

To see how the net module  uses  Unix  sockets,  let’s  modify  the  net-watcher program to use this kind of communication channel. Keep in mind that Unix sockets work only on Unix-like environments.

Open the net-watcher.js program and change the `server.listen()` section to this:

```js
    server.listen('/tmp/watcher.sock', function(){
    	console.log('Listeningforsubscribers...');
    });
```

To connect a client, we now need `nc` instead of `telnet`. `nc` is short for `netcat`, a TCP/UDP socket utility program that also supports Unix sockets.

	$nc -U /tmp/watcher.sock

Unix  sockets  can  be  faster  than  TCP  sockets  because  they  don’t  require invoking network hardware. However, they’re local to the machine.

### 3.2 实现消息协议

这些我们将创建一个协议，协议基于JSON消息。

#### 使用JSON序列化消息

例子net-watcher使用两种消息。建立连接时的消息形如：

	{"type": "watching", "file": "target.txt"}

文件改变时的消息：

	{"type": "changed", "file": "target.txt", "timestamp": 1358175733785}

我们打算用换行分隔消息。称这种协议为Line Delimited JSON (LDJ)。

#### 改为使用JSON消息

```js
    connection.write(JSON.stringify({
    	type:'watching',
    	file:filename
    }) + '\n');

    connection.write(JSON.stringify({
    	type:'changed',
    	file:filename,
    	timestamp:Date.now()
    })+'\n');
```

### 3.3 创建客户端Socket

一个基本的（有问题的）实现：

```js
    "use strict";
    const
      net = require('net'),
      client = net.connect({port:5432});
    client.on('data', function(data){
      let message=JSON.parse(data);
      if(message.type==='watching') {
        console.log("Nowwatching:"+message.file);
      } elseif(message.type==='changed') {
        let date=newDate(message.timestamp);
        console.log("File'"+message.file+"'changedat"+date);
      } else {
        throw Error("Unrecognized message type:"+message.type);
      }
    });
```

上述代码有一个问题——由消息边界导致。一条消息可能一次性到达。或者分成几个快，触发多个`data`事件。解决见下一节。

### 3.4 测试网络应用

#### 测试

测试客户端：

```js
"use strict";
    const
      net=require('net'),
      server=net.createServer(function(connection){
        console.log('Subscriberconnected');
        // send the first chunk immediately
        connection.write('{"type":"changed","file":"targ');
        // after a one second delay, send the other chunk
        let timer = setTimeout(function(){
          connection.write('et.txt","timestamp":1358175758495}'+"\n");
          connection.end();
        },1000);
        // clear timer when the connection ends
        connection.on('end', function(){
          clearTimeout(timer);
          console.log('Subscriberdisconnected');
        });
      });
    server.listen(5432,function(){
      console.log('Testserverlisteningforsubscribers...');
    });
```

### 3.5 自定义模块和类

#### 继承EventEmitter

To relieve the client program from the danger of split JSON messages, we’ll implement an LDJ buffering client module. Then we’ll incorporate it into the network-watcher client.

首先看如何在Node中实现继承。下面的代码，`LDJClient`继承自`EventEmitter`。

```js
    const
      events=require('events'),
      util=require('util'),
      // client constructor
      LDJClient = function(stream){
        events.EventEmitter.call(this);
      };
    util.inherits(LDJClient, events.EventEmitter);
```

`LDJClient`是一个构造函数，于是实例化的方式是`new LDJClient(stream)`。The `stream` parameter is an object that emits `data` events, such as a Socket connection.

在构造器函数中，调用`EventEmitter`的构造器并传入`this`。最后通过`util.inherits()`，让`EventEmitter`称为`LDJClient`的原型。

Javascript中有其他的继承方式。但上面的方式是Node核心库使用的风格。

`LDJClient`的使用：

```js
    const client=newLDJClient(networkStream);
    client.on('message', function(message){
    	//takeactionforthismessage
    });
```

#### 缓冲

It’s time to use the `stream` parameter in the `LDJClient` to retrieve and buffer input.

```js
    LDJClient = function(stream){
      events.EventEmitter.call(this);
      let self = this, buffer = '';
      stream.on('data', function(data){
        buffer+=data;
        let boundary = buffer.indexOf('\n');
        while(boundary!==-1) {
          let input = buffer.substr(0, boundary);
          buffer = buffer.substr(boundary + 1);
          self.emit('message', JSON.parse(input));
          boundary = buffer.indexOf('\n');
        }
      });
    };
```

Next we need to put this class into a Node module so our upstream client can use it.

#### 导出模块中的功能

```js
    "use strict";
    const
      events=require('events'),
      util=require('util'),
      //client constructor
      LDJClient=function(stream){
      events.EventEmitter.call(this);
      let self=this, buffer='';
      stream.on('data',function(data) {
      	buffer+=data;
    	let boundary=buffer.indexOf('\n');
        while(boundary!==-1) {
	      let input=buffer.substr(0,boundary);
	  	  buffer=buffer.substr(boundary+1);
          self.emit('message',JSON.parse(input));
    	  boundary=buffer.indexOf('\n');
        }
      });
    };
    util.inherits(LDJClient,events.EventEmitter);
    //expose module methods
    exports.LDJClient = LDJClient;
    exports.connect=function(stream){
    	return new LDJClient(stream);
	};
```

Save the file as `ldj.js`. Code to use the LDJ module will look something like this:

```js
    const
      ldj = require('./ldj.js'),
      client = ldj.connect(networkStream);
    client.on('message', function(message){
      // take action for this message
});
```

#### 使用新模块实现最终功能

```js
    "use strict";
    const
      net=require('net'),
      ldj=require('./ldj.js'),
      netClient=net.connect({port:5432}),
      ldjClient=ldj.connect(netClient);
    ldjClient.on('message', function(message){
      if(message.type==='watching'){
        console.log("Nowwatching:"+message.file);
      } elseif(message.type==='changed') {
        console.log(
          "File'"+message.file+"'changedat"+newDate(message.timestamp)
        );
      } else {
        throw Error("Unrecognizedmessagetype:"+message.type);
      }
    });
```