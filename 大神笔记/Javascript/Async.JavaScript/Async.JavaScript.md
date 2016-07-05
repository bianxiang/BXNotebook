# 异步Javascript

[toc]

## 介绍

如果事件不多，事件驱动的代码比多线程代码要简单。不必处理线程安全。当事件增多时，可能导致 *Pyramid of Doom* 问题。
```javascript
step1(function(result1){
	step2(function(result2){
		step3(function(result3){
			//andsoon...
		});
	});
});
```

“I love async, but I can’t code like this,”one developer famously complained on the Node.js Google Group. But the problem isn’t with the language itself; it’s with the way programmers use the language. Dealing with complex sets of events in an elegant way is still frontier territory in JavaScript.

----

一组可以编译成JavaScript的语言：http://altjs.org。Some altJS languages are aimed specifically at “taming”async callbacks by allowing them to be written in a more synchronous style.

## 1 理解 JavaScript 事件

### 1.1 事件调度

线程的阻塞：你可能本期望下面的代码输出500。但实际上输出至少有1000。因为必须等while执行完回调才有机会执行。
```javascript
var start=newDate;
setTimeout(function(){
  var end = new Date;
  console.log('Timeelapsed:', end - start, 'ms');
}, 500);
while(newDate - start < 1000) {};
```

### 1.2 异步函数的类型

不同的Javascript环境（浏览器、服务器）提供不同的异步函数。它们一般分为两类：I/O 和时间类。


When you manipulate the DOM in a modern browser, the changes are immediate from your script’s perspective but aren’t rendered until you return to the event queue. That prevents the DOM from being rendered in an inconsistent state. You can see a simple demonstration of this at http://jsfiddle.net/TrevorBurnham/SNBYV/.

书中说，在Webkit中，`console.log`是异步的。下面的代码将打印`{foo:bar}`：
```javascript
var obj = {};
console.log(obj);
obj.foo = 'bar';
```

但在Node.js中，`console.log`是同步的。

但我测试的情况，Chrome中，并未打印`{foo:bar}`，而是打印`Object {}`。说明方法是同步的。

下面的代码，本该以尽可能快的速度重复执行。
```javascript
var fireCount = 0;
var start = newDate;
var timer = setInterval(function(){
	if(new Date - start > 1000){
		clearInterval(timer);
		console.log(fireCount);
		return;
	}
	fireCount++;
}, 0);
```

但在Chrome, Safari, and Firefox，差不多每秒200次。Node能达到每秒1000次。而改成等价的while循环，浏览器和Node分别可以达到 4,000,000 和 5,000,000 次每秒。

In fact, the HTML spec (which all major browsers respect) mandates a minimum timeout/interval of 4ms!

如果向更详细的控制时间：
- In Node, `process.nextTick` lets you schedule an event to fire ASAP. On my system, `process.nextTick` events fire at a rate of over 100,000/sec.
- Modern browsers (including IE9+) have a `requestAnimationFrame` function, which serves a dual purpose: it allows JavaScript animations to run at 60+ frames/sec, and it conserves CPU cycles by preventing those animations from running in background tabs.  In the latest Chrome, you can even get submillisecond precision.

### 1.3 编写异步函数

JavaScript中，异步函数要构建在其他异步函数之上。

使用异步函数的函数也必须异步提供结果。在JavaScript中，无法阻塞函数返回直到某个异步完成——事实上，只有函数方法，异步事件才有机会执行。

有时回调函数是间接指定的。如利用 *Promise* 或 *PubSub*。


### 1.4 处理异步错误

In this section, we’ll see why `throw` is rarely the right tool for handling errors in callbacks and how async APIs are designed around this limitation.

从一个异步回调方法中抛出一个异常：
```javascript
// EventModel/nestedErrors.js
setTimeout(functionA(){
	setTimeout(functionB(){
		setTimeout(functionC(){
			throw new Error('Something terrible has happened!');
		},0);
	},0);
},0);
```

结果：
```
Error:Somethingterriblehashappened!
	atTimer.C(/AsyncJS/nestedErrors.js:4:13)
```

注意到回调 A 和 B 未在异常栈中。这三个回调方法都直接从事件队列中运行。

同样的原因，无法捕获异步回调中抛出的异常：
```javascript
try{
	setTimeout(function(){
		throw new Error('Catchmeifyoucan!');
	}, 0);
} catch(e) {
	console.error(e);
}
```

这就是为什么，Node.js 中，错误总是回调的第一个参数。例如：
```javascript
var fs = require('fs');
	fs.readFile('fhgwgdz.txt', function(err, data){
		if(err){
			return console.error(err);
		};
	console.log(data.toString('utf8'));
});
```

对于客户端JavaScript库，更常见的模式是分开指定成功和失败回调：
```javascript
$.get('/data',{
	success:successHandler,
	failure:failureHandler
});
```

#### 未捕获异常

When we throw an exception from a callback, it’s up to whomever calls the callback to catch it. But what if the exception is never caught? At that point, different JavaScript environments play by different rules….

在浏览器中，未捕获异常会被打到控制台，然后事件循环继续执行。但如果指定了`window.onerror`且方法返回true，可以阻止浏览器默认的异常处理：
```javascript
window.onerror = function(err){
	return true; //ignore all errors completely
};
```

In production, you might want to consider a JavaScript error-handling service, such as [Errorception](https://errorception.com/). Errorception provides  a ready-made `window.onerror` handler that reports all uncaught exceptions to their server, which can then send you notifications.


Node世界中的`window.onerror`是`process`对象的`uncaughtException`事件。未捕获异常会导致 Node 应用立即退出。但如果提供了`uncaughtException`处理器，应用会继续执行事件队列。
```javascript
process.on('uncaughtException', function(err){
	console.error(err);//shutdownaverted!
});
```

但从 Node 0.8.4 开始，`uncaughtException`已不推荐使用。
> uncaughtException is a very crude mechanism for exception handling and may be removed in the future…
Don’t use it, use domains instead.

Domains are *evented objects* that convert throws into 'error' events. 例如：

```javascript
var myDomain = require('domain').create();
myDomain.run(function(){
	setTimeout(function(){
		throw new Error('Listentome!')
	},50);
});
myDomain.on('error', function(err){
	console.log('Error ignored!');
});
```

Whether  you’re  in  the  browser  or  on  the  server,  global  exception  handlers should be seen as a measure of last resort. Use them only for debugging.

### 1.5 过度嵌套的回调

下面的代码功能上没有问题，但却有三层回调：`db.query`、`hash`和`callback`:
```javascript
function checkPassword(username, passwordGuess, callback){
	var queryStr = 'SELECT * FROM user WHERE username = ?';
	db.query(selectUser, username, function(err, result){
		if(err) throw err;
		hash(passwordGuess, function(passwordGuessHash){
			callback(passwordGuessHash === result['password_hash']);
		});
	});
}
```

可以改写为始终只有一层回调：
```javascript
function checkPassword(username, passwordGuess, callback){
	var passwordHash;
	var queryStr = 'SELECT * FROM user WHERE username = ?';
	db.query(selectUser, username, queryCallback);

	function queryCallback(err, result){
		if(err) throw err;
		passwordHash = result['password_hash'];
		hash(passwordGuess, hashCallback);
	}

	function hashCallback(passwordGuessHash){
		callback(passwordHash === passwordGuessHash);
	}
}
```

｛｛副作用，无法使用闭包｝｝

## 2 分发事件

如何处理事件？最直观的理解：一个事件，一个处理器。但有时事件会触发大量操作，如Gmail中的按键事件，可能触发：屏幕显示这个字符，移动光标，拼写检查，自动保存等等。

我们需要能够分发事件（use distributed events）。

本章介绍发布/订阅模式。介绍实际中的表现：Node 的 `EventEmitter`，Backbone 的事件模型，jQuery 的自定义事件。


### 2.1 PubSub

最早的方式：`link.onclick = clickHandler;`。
如果想指定多个处理器，需要手工包装：
```javascript
link.onclick = function(){
	clickHandler1.apply(this,arguments);
	clickHandler2.apply(this,arguments);
};
```

W3C引入`addEventListener`，消除了手工包装的需要。在jQuery中，对应`bind`和`on`。

Node.js中则是`EventEmitter`：
```javascript
['room','moon','cowjumpingoverthemoon']
	.forEach(function(name){
		process.on('exit', function(){
		console.log('Goodnight,' + name);
	});
});
```

发布订阅有多种实现。当jQuery团队意识到库中发布订阅实现有多种时，they decided to abstract them with `$.Callbacks` in jQuery 1.7. Instead of using an array to store the handlers corresponding to an event type, you could use a `$.Callbacks` instance.

Many PubSub implementations parse the event string to provide special features. For example, you may be familiar with namespaced events in jQuery: if I bind events named "`click.tbb`" and "`hover.tbb`", I can unbind them both by simply calling `unbind(".tbb")`. Backbone.js lets you bind handlers to the "all" event type, causing them to go off whenever anything happens. Both jQuery and Backbone let you bind or emit multiple event types simultaneously by separating them with spaces, e.g., "keypress mousemove".

注意，上述的事件回调/处理器都是同步的。

### （未）2.2 Evented Models

### （未）2.3 Custom jQuery Events




