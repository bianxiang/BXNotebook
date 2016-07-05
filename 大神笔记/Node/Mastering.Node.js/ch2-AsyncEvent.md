[toc]

## 2 理解异步事件驱动编程

### 2.1 广播事件

I/O是昂贵的。

| 存储| 消耗CPU周期 |
|--------|--------|
| L1 cache | 3 cycles  |
| L2 cache | 14 cycles |
| RAM      | 250 cycles |
| Disk     | 41,000,000 cycles |
| Network  | 240,000,000 cycles |

### 2.2 监听事件

Node中暴露事件接口的多数对象，如文件、网络流，都将`EventEmitter`作为其原型（prototype）。

但本节的目的是讨论一些更少人知晓的事件源：信号、子进程通信、文件系统事件和延迟（deferred）执行。

#### （未）2.2.1 Signals

#### 2.2.2 Forks

Node的一个设计基础是，当需要并行执行时，创建或fork进程，而不是创建线程。这里主要关注子进程之间如何通讯。

要创建一个子进程，调用`child_process`模块的`fork`方法，传入新进程要执行的文件名：

```js
    var cp = require('child_process');
    var child = cp.fork(__dirname + '/lovechild.js');
```

在多核机器上，子进程一般会被OS分配到不同的核上。将Node进程分布到多核（甚至多个机器上），并通过IPC管理，是分布式Node的实现方式之一。

父进程可以向子进程发送消息，或监听子进程的消息。

```js
    child.on('message', function(msg) {
        console.log('Child said: ', msg);
    });
    child.send("I love you");
```

子进程（程序定义在`lovechild.js`）也可以发送、监听消息：

```js
    // lovechild.js
    process.on('message', function(msg) {
        console.log('Parent said: ', msg);
        process.send("I love you too");
    });
```

通过将服务器实例传给子进程，实现多进程（包括父进程）负载均衡。

例如，下面程序启动一个服务器，fork一个子进程，然后把服务器传给子进程：

```js
    var child = require('child_process').fork('./child.js');
    var server = require('net').createServer();
    server.on('connection', function(socket) {
        socket.end('Parent handled connection');
    });
    server.listen(8080, function() {
        child.send("The parent message", server);
    });
```

```js
    // child.js
    process.on('message', function(msg, server) {
        console.log(msg);
        server.on('connection', function(socket) {
            socket.end('Child handled connection');
        });
    });
```

It should be clear that this technique, when combined with the simple inter-process messaging protocol discussed previously, demonstrates how Ryan Dahl's creation succeeds in providing an easy way to build *scalable* network programs.

> 我们会讨论Node的新模块`cluster`，它扩展和简化了上面讨论的技术。If you are interested in how server handles are shared, visit the cluster documentation at the following link:
http://nodejs.org/api/cluster.html
For those who are truly curious, examine the clustercode itself at:
https://github.com/joyent/node/blob/c668185adde3a474585a11f172b8387e270ec23b/lib/cluster.js#L523-558

#### 2.2.3 文件事件

通过`fs.watch`方法可以监听文件系统通知，包括文件的改变和目录的改变。

`watch`接受三个参数，依次是：

- 被监控的文件或目录的路径。如果文件不存在，将抛出**ENOENT(no entity)**错误。可以先使用`fs.exists`检查。
- 选项（可选）：
 - `persistent`：布尔。设置此选项为false，当此watch是唯一活动时，进程停掉。
- 监听器函数。函数接受两个参数：
 - 事件名（如`rename`或`change`）。
 - 发生改变的文件的名字。某些操作系统不会返回该参数。

例子：监控自己，改变自己的名字然后退出：
```js
    var fs = require('fs');
    fs.watch(__filename, { persistent: false }, function(event, filename) {
        console.log(event);
        console.log(filename);
    })
    setImmediate(function() {
        fs.rename(__filename, __filename + '.new', function() {});
    });
```

关闭监控：
```js
    var w = fs.watch('file', function(){})
    w.close();
```

It should be noted that `fs.watch` depends a great deal on how the host OS handles file events, and according to the Node documentation:

> "The fs.watch API is not 100% consistent across platforms, and is unavailable in some situations."

#### 2.2.4 推迟执行

经常需要推迟一个函数的执行。Javascript传统上使用定时器：`setTimeout`和`setInterval`。

下面介绍两种延迟事件源（deferred event sources），其回调执行分别在**I/O事件**之前和之后执行。

##### `process.nextTick`

Node本地模块`process`的`process.nextTick`方法类似于`setTimeout`，延迟执行回调方法。`nextTick`的回调方法会被放置在事件队列头部，在 I/O 和定时器事件之前，但在*当前脚本之后执行*（JavaScript代码在 V8 线程上同步执行）。

例子：

```javascript
    var events = require('events');
    function getEmitter() {
        var emitter = new events.EventEmitter();
        emitter.emit('start');
        return emitter;
    }
    var myEmitter = getEmitter();
    myEmitter.on("start", function() {
        console.log("Started");
    });
```

然后实际结果与期望不同。发出"start"事件发生在注册监听器之前。

使用`process.nextTick`解决该问题：

```javascript
    var events = require('events');
    function getEmitter() {
        var emitter = new events.EventEmitter();
        process.nextTick(function() {
            emitter.emit('start');
        });
        return emitter;
    }
    var myEmitter = getEmitter();
    myEmitter.on('start', function() {
        console.log('Started');
    })
```

在函数中，`nextTick`的主要用途是，postpone the broadcast of result events to listeners on the current execution stack until the caller has had an opportunity to register event listeners—to give the currently executing program a chance to bind callbacks to `EventEmitter.emit` events. It may be thought of as a pattern used wherever asynchronous behavior should be emulated. For instance, imagine a lookup system that may either fetch from a cache or pull fresh data from a data store. The cache is fast and doesn't need callbacks, while the data I/O call would need them. The need for callbacks in the second case argues for emulation of the callback behavior with `nextTick` in the first case. This allows a consistent API, improving clarity of implementation without burdening the developer with the responsibility of determining whether or not to use a callback.

Because it is possible to recursively call `nextTick`, which might lead to an infinite loop of recursive `nextTick` calls (starving the event loop, preventing I/O), there exists a *failsafe* mechanism in Node which limits the number of recursive `nextTick` calls evaluated prior to yielding the I/O: `process.maxTickDepth`. Set this value (which defaults to 1000) if such a construct becomes necessary—although what you probably want to use in such a case is `setImmediate`.

##### `setImmediate`

`setImmediate`类似于`process.nextTick`，但区别在于：`nextTick`的回调在 I/O 和定时器事件之前调用，但`setImmediate`的回调在 I/O 事件之后调用。

> 这两个方法的名字有歧义：`nextTick` 实际在 `setImmediate` 之前执行。

`setImmediate`方法返回值，可以传入`cancelImmediate`，用于取消。

### 2.3 定时器

JavaScript提供两种异步定时器：`setInterval()`和`setTimeout()`。

执行回调前实际的延迟时间可能比设置的时间略长。执行的次序也是不可保证的。Node的定时器是不可中断的。Timers simply promise to execute as close as possible to the specified time (though never before), beholden, as with every other event source, to event loop scheduling.

> At least one thing you may not know about timers...
We are all familiar with the standard arguments to setTimeout: a callback function and timeout interval. Did you know that many additional arguments are passed to the callback function?
```javascript
setTimeout(callback, time, [passArg1, passArg2…])
```

### 2.3.1 setTimeout

延迟一些毫秒后执行：

```javascript
    setTimeout(a, 1000);
    setTimeout(b, 1001);
```

`a`不一定先于`b`执行。但另一种情况：

```javascript
    setTimeout(a, 1000);
    setTimeout(b, 1000);
```

执行顺序是确定的。Node essentially maintains an object map grouping callbacks with identical timeout lengths. *Isaac Schlueter*, the current leader of the Node project, puts it this way:

    [N]ode uses a single low level timer object for each timeout value. If you attach multiple callbacks for a single timeout value, they'll occur in order, because they're sitting in a queue. However, if they're on different timeout values, then they'll be using timers in different threads, and are thus subject to the vagaries of the [CPU] scheduler.

The ordering of timer callbacks registered within an identical execution scope does not predictably determine the eventual execution order in all cases.

Additionally, there exists a minimum wait time of one millisecond for a timeout. Passing a value of zero, -1, or a non-number will be translated into this minimum value.

### 2.3.2 setInterval

周期性执行一个功能。例如，每100毫秒执行一次：

```javascript
	var intervalId = setInterval(function() { ... }, 100);
```

通过`clearInterval(intervalId)`取消。

与`setTimeout`一样不可靠。Importantly, if a system delay (such as some badly written blocking while loop) occupies the event loop for some period of time, intervals set prior and completing within that interim will have their results queued on the stack. 当事件循环不再阻塞，所有的回调将被立即调用（串行）。

幸运的是，Node中的interval比浏览器中可靠很多。

### 2.3.3 unref 和 ref

A Node program does not stay alive without a reason to do so. 如果有回调尚未被处理，进程将继续运行。完成后，进程没什么可做，于是退出。例如，下面的代码会让Node进程一直运行：

```javascript
	var intervalId = setInterval(function() {}, 1000);
```

There are cases of using a timer to do something interesting with external I/O, or some data structure, or a network interface where once those external event sources stop occurring or disappear, the timer itself stops being necessary. Normally one would trap that irrelevant state of a timer somewhere else in the program and cancel the timer from there. This can become difficult or even clumsy, as an unnecessary tangling of concerns is now necessary, an added level of complexity.

The `unref` method allows the developer to assert the following instructions: when this timer is the only event source remaining for the event loop to process, go ahead and terminate the process.

例如，下面的代码会让进程结束，不再一直运行：

```javascript
    var intervalId = setInterval(function() {}, 1000);
    intervalId.unref();
```

`unref`方法是创建定时器时返回的对象。

现在加入一个外部事件源（一个定时器）。当外部事件源结束后，进程终止。

```javascript
    setTimeout(function() {
        console.log("now stop");
    }, 100);
    var intervalId = setInterval(function() {
        console.log("running")
    }, 1);
    intervalId.unref();
```

利用`ref`方法可以让定时器回到常规的状态，它的作用是取消`unref`方法：

```javascript
    var intervalId = setInterval(function() {}, 1000);
    intervalId.unref();
    intervalId.ref();
```

此时，进程又将一直运行下去。


## 2.4 理解事件循环

Node使用单个线程处理Javascript指令。在你的Javascript中，两句代码不可能同时执行。

这并不意味着Node进程所在的机器只使用一个线程。回调并不产生并发。Recall Chapter 1, Understanding the Node Environment, and our discussion about the `process` object—Node's "single thread" simplicity is in fact an abstraction created for the benefit of developers. 但一定要记得，有大量线程在背后管理I/O（和其他东西），and these threads unpredictably insert instructions, originally packaged as callbacks, into the single JavaScript thread for processing.

Node一条一条的执行的执行直到没有指令可执行，no more input or output to stream, and no further callbacks waiting to be handled.

即使延迟（deferred）时间（如定时器）require an eventual interrupt in the event loop to fulfill their promise.

下面的代码，本期望1秒后改变`stop`的值。但实际while循环将无限执行下去。while循环反复执行，一直占据着事件循环。事件循环无法给定时器回调机会执行。

```javascript
    var stop = false;
    setTimeout(function() {
        stop = true;
    }, 1000);
    while(stop === false) {};
```

写Node就是写事件循环。We've previously discussed the event sources that are queued and otherwise arranged and ordered on this event loop — I/O events, timer events, and so on.

When writing non-deterministic code it is imperative that no assumptions about eventual callback orders are made. The abstraction that is Node masks the complexity of the thread pool on which the straight forward main JavaScript thread floats, leading to some surprising results.

We will now refine this general understanding with more information about how, precisely, the callback execution order for each of these types is determined within Node's event loop.

### Four sources of truth

我们已经学习了四组主要的事件源（deferred event sources），下面总结它们在栈中的位置和优先级：

- 执行代码块（Execution blocks）：Javascript代码，包括表达式、循环、函数。This includes `EventEmitter` events emitted within the current execution context.
- 定时器：Callbacks deferred to sometime in the future specified in milliseconds, such as `setTimeout` and `setInterval`.
- I/O：Prepared callbacks returned to the main thread after being delegated to Node's managed thread pool, such as filesystem calls and network listeners.
- Deferred execution blocks: Mainly the functions slotted on the stack according to the rules of `setImmediate` and `nextTick`.

We have learned how the deferred execution method `setImmediate` slots its callbacks after I/O callbacks in the event queue, and `nextTick` slots its callbacks before I/O and timer callbacks.

> A challenge for the reader
After running the following code, what is the expected order of logged messages?

```javascript
    var fs = require('fs');
    var EventEmitter = require('events').EventEmitter;
    var pos = 0;
    var messenger = new EventEmitter();
    // Listener for EventEmitter
    messenger.on("message", function(msg) {
        console.log(++pos + " MESSAGE: " + msg);
    });
    // (A) FIRST
    console.log(++pos + " FIRST");
    // (B) NEXT
    process.nextTick(function() {
        console.log(++pos + " NEXT")
    })
    // (C) QUICK TIMER
    setTimeout(function() {
        console.log(++pos + " QUICK TIMER")
    }, 0)
    // (D) LONG TIMER
    setTimeout(function() {
        console.log(++pos + " LONG TIMER")
    }, 10)
    // (E) IMMEDIATE
    setImmediate(function() {
        console.log(++pos + " IMMEDIATE")
    })
    // (F) MESSAGE HELLO!
    messenger.emit("message", "Hello!");
    // (G) FIRST STAT
    fs.stat(__filename, function() {
        console.log(++pos + " FIRST STAT");
    });
    // (H) LAST STAT
    fs.stat(__filename, function() {
        console.log(++pos + " LAST STAT");
    });
    // (I) LAST
    console.log(++pos + " LAST");
```

The output of is program is:

1.  FIRST (A).
2.  MESSAGE: Hello! (F).
3.  LAST (I).
4.  NEXT (B).
5.  QUICK TIMER (C).
6.  FIRST STAT (G).
7.  LAST STAT (H).
8.  IMMEDIATE (E).
9.  LONG TIMER (D).

Let's break the preceding code down:

A, F, and I execute in the main program flow and as such they will have the first priority in the main thread (this is obvious; your JavaScript executes its instructions in the order they are written, including the synchronous execution of the `emit` callback).

With the main call stack exhausted, the event loop is now almost reading to process I/O operations. This is the moment when nextTick requests are honored slotting in at the head of the event queue. This is when B is displayed.
The rest of the order should be clear. Timers and I/O operations will be processed next, (C, G, H) followed by the results of the `setImmediate` callback (E), always arriving after any I/O and timer responses are executed.

Finally, the long timeout (D) arrives, being a relatively far-future event.
Notice that re-ordering the expressions in this program will notchange the output order (outside of possible re-ordering of the STAT results, which only implies that they have been returned from the thread pool in different order, remaining as a group in the correct order as relates to the event queue).

## 2.5 回调和错误

### 2.5.1 约定

遵循：

- 回调函数的第一个参数是错误消息，最好是一个错误对象。如果没有错误，应该传`null`。- 向函数传入一个回调，回调应该是函数的最后一个参数。APIs should be consistently designed this way.
- 错误参数和回调参数之间可以有任意数量的参数。

创建错误对象：

```javascript
	new Error("Argument must be a String!")
```
### 2.5.2 了解你的错误

It is excellent that the Node community has automatically adopted a convention that compels developers to be diligent and report errors. 收到错误该如何处理？

集中错误处理常常是一个好主意。常常需要设计一个错误处理系统，负责向客户端发送消息，写日志等。有时更好的做法是抛出错误，终止进程。

Node为错误处理提供了更高级的工具。Node的`domain`系统帮助处理事件系统的问题i：how can a stack trace be generated if the full route of a call has been obliterated as it jumped from callback to callback?

`domain`的目标很简单：fence and label an execution context such that all events that occur within it are identified as such, allowing more informative stack traces. By creating several different domains for each significant segment of your program, a chain of errors can be properly understood.

Additionally, this provides a way to catch errors and handle them, rather than allowing your entire Node process to collapse.

In the following example we're going to create two domains: `appDomain` and `fsDomain`. 目标是追踪应用的哪部分目前正处于错误状态：

```javascript
    var domain = require("domain");
    var fs = require("fs");
    var fsDomain = domain.create();
    fsDomain.on("error", function(err) {
        console.error("FS error", err);
    });
    var appDomain = domain.create();
    appDomain.on('error', function(err) {
        console.log("APP error", err);
    });
```

将主程序包裹进`appDomain`，将文件系统调用包裹进`fsDomain`。We then create an error in `fsDomain` by trying to open a non-existent file:

```javascript
    appDomain.run(function() {
        process.nextTick(function() {
            fsDomain.run(function() {
                fs.open('no_file_here', 'r', function(err, fd) {
                    if(err) {
                        throw err;
                    }
                    appDomain.dispose();
                });
            });
        });
    });
```

When the preceding code executes, something resembling this should be echoed to the terminal:

```
FS error { [Error: ENOENT, open 'non-existent file']
    errno: 34,
    code: 'ENOENT',
    path: 'non-existent file',
    domain:
        { domain: null,
          _events: { error: [Function] },
          _maxListeners: 10,
          members: [] },
    domainThrown: true }
```

Now let's create an error in `appDomain` by adding this code, which will produce a reference error (as no `b` is defined):

```javascript
    appDomain.run(function() {
        a = b;
        process.nextTick(function() {
    ...
```

An error similar to that in the precious code should be generated and reported by appDomain.

Notice the command `appDomain.dispose`. As maintaining these error contexts will consume some memory, it is best to dispose of them when no longer needed—after the code they contain has successfully executed, for example. We'll learn more advanced uses of this tool as we progress into more complex territories.

As an application grows in complexity it will become more and more useful to be able to trap errors and handle them properly, perhaps restarting only one part of an application when it fails rather than the entire system.

### 2.5.3 建造金字塔

Simplifying controlflows has been a concern of the Node community since the very beginning of the project. Indeed, this potential criticism was one of the very first anticipated by Ryan Dahl, who discussed it at length during the talk in which he introduced Node to the JavaScript developer community.

Node代码经常回调嵌套回调，Node代码常常像一个侧放的金字塔。

Accordingly, there are several Node packages available which take the problem on, employing strategies as varied as futures, fibers, even C++ modules exposing system threads directly.

- Async https://github.com/caolan/async
- Tame https://github.com/maxtaco/tamejs
- Fibers https://github.com/laverdet/nodefibers
- Promises https://github.com/kriskowal/q

> Mikeal Rogers, in discussing why Promises were removed from the Node core, makes a strong argument in the following link for why leaving feature development to the community leads to a stronger core product:
http://www.futurealoof.com/posts/broken-promises.html

### （未）2.5.4 思考

## 2.6 监听对文件的改变

实践之前学到的知识。创建一个服务，客户端可以接收Twitter的更新。创建一个进程查询Twitter消息，将消息写入*tweets.txt*文件。创建一个网络服务器将这些消息广播到一个客户端。广播由*tweets.txt*文件的写事件触发。最后创建一个*client.html*页面显示消息。

本例展示了：

- 监听文件系统事件
- 使用数据流事件读写文件
- 响应网络事件
- Using timeouts for polling state
- 将Node服务器作为网络事件的广播者

处理服务器广播，使用Server Sent Events(SSE)协议，该协议是HTML5的一部分。

先创建一个Node服务器，监听文件改变，广播内容到客户端。创建`server.js`文件：

```javascript
    var fs = require("fs");
    var http = require('http');
    var theUser = null;
    var userPos = 0;
    var tweetFile = "tweets.txt";
```

我们只接受单个用户连接，保存在`theUser`。`userPos`保存对`tweetFile`文件上一次的读取位置：

```javascript
    http.createServer(function(request, response) {
        response.writeHead(200, {
            'Content-Type': 'text/event-stream',
            'Cache-Control': 'no-cache',
            'Access-Control-Allow-Origin': '*'
        });
        theUser = response;
        response.write(':' + Array(2049).join(' ') + '\n');
        response.write('retry: 2000\n');
        response.socket.on('close', function() {
            theUser = null;
        });
    }).listen(8080);
```

参数`response`实现了writeable stream接口，允许我们向客户端写数据：

```javascript
    var sendNext = function(fd) {
        var buffer = new Buffer(140);
        fs.read(fd, buffer, 0, 140, userPos * 140, function(err, num) {
            if(!err && num > 0 && theUser) {
                ++userPos;
                theUser.write('data: ' + buffer.toString('utf-8', 0, num) + '\n\n');
                return process.nextTick(function() {
                    sendNext(fd);
                });
            }
        });
    };
```

When done, we queue up a repeat call of the same function using `nextTick`, repeating until we get an error, receive no data, or the client disconnects:

```javascript
    function start() {
        fs.open(tweetFile, 'r', function(err, fd) {
            if(err) {
                return setTimeout(start, 1000);
            }
            fs.watch(tweetFile, function(event, filename) {
                if(event === "change") {
                    sendNext(fd);
                }
            });
        });
    };
    start();
```

启动服务器时，文件可能还不存在，因此使用`setTimeout`轮询直到文件出现。

下面产生文件。We first install the *TWiT* Twitter package for Node, via npm.

创建一个进程，唯一的任务是写数据到文件：

```javascript
    var fs = require("fs");
    var Twit = require('twit');
    var twit = new Twit({
        consumer_key: 'your key',
        consumer_secret: 'your secret',
        access_token: 'your token',
        access_token_secret: 'your secret token'
    })
    var tweetFile = "tweets.txt";
    var writeStream = fs.createWriteStream(tweetFile, {
        flags : "a"
    });

    var cleanBuffer = function(len) {
        var buf = new Buffer(len);
        buf.fill('\0');
        return buf;
    }
```

因为Twitter消息不超过140字节，我们可以简化读写操作，总是写140字节的块，即便可能不到140字节。

```javascript
    var check = function() {
        twit.get('search/tweets', {
            q: '#nodejs since:2013-01-01'
        }, function(err, reply) {
            var buffer = cleanBuffer(reply.statuses.length * 140);
            reply.statuses.forEach(function(obj, idx) {
                buffer.write(obj.text, idx*140, 140);
            });
            writeStream.write(buffer);
        })
        setTimeout(check, 10000);
    };
    check();
```

每10秒检查一次消息。

最后是前端页面。使用 *SSE* 监听本地8080端口。

```html
    <!DOCTYPE html>
    <html>
    <head>
    <title></title>
    </head>
    <script>
    window.onload = function() {
        var list = document.getElementById("list");
        var evtSource = new EventSource("http://localhost:8080/events");
        evtSource.onmessage = function(e) {
            var newElement = document.createElement("li");
            newElement.innerHTML = e.data;
            list.appendChild(newElement);
        }
    }
    </script>
    <body>
    <ul id="list"></ul>
    </body>
    </html>
```

> 更多关于 SSE 的知识，参见第六章，Creating Real-time Application。或：
https://developer.mozilla.org/en-US/docs/Server-sent_events/Using_server-sent_events







