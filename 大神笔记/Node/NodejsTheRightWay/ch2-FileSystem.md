[toc]

## 2. 文件系统

### 2.1 Node.js事件循环编程

Let’s get started by developing a couple of simple programs that watch files for changes and read arguments from the command line.

#### Watching a File for Changes

利用touch命令创建一个文件：

	$touch target.txt

```js
    const fs=require('fs');
    fs.watch('target.txt', function(){
        console.log("File 'target.txt' just changed!");
    });
	console.log("Now watching target.txt for changes...");
```

注意`const`关键字（属于ECMAScript Harmony）。

Node’s module implementation is based on the CommonJS module specification.

执行：

	$node --harmony watcher.js

`--harmony`告诉Node使用最新的ECMAScript Harmony特性。

在另一个控制台，输入

	$touch target.txt

#### 读取命令行参数

本节涉及：全局对象`process`；Node如何处理异常。

```js
    const
      fs=require('fs'),
      filename=process.argv[2];
    if(!filename){
      throw Error("A file to watch must be specified!");
    }
    fs.watch(filename, function(){
      console.log("File" + filename + "just changed!");
    });
    console.log("Now watching" + filename + "forchanges...");
```

运行：

	$node --harmony watcher-argv.js target.txt

`process.argv`的前两个元素分别是`node`和`watcher-argv.js`的全路径。第三个元素是`target.txt`。

### 2.2 子进程

file-system/watcher-spawn.js
```js
    "use strict";
    const
      fs=require('fs'),
      spawn=require('child_process').spawn,
      filename=process.argv[2];
    if(!filename){
       throw Error("A file to watch must be specified!");
    }
    fs.watch(filename, function(){
      let ls = spawn('ls', ['-lh', filename]);
      ls.stdout.pipe(process.stdout);
    });
    console.log("Now watching" + filename + "for changes...");
```

运行：

	$node --harmony watcher-spawn.js target.txt

Strict mode is also **required** to use certain ECMAScript Harmony features in Node, such as the **let** keyword. Like **const**, **let** declares a variable, but a variable declared with **let** can be assigned a value more than once.

`require('child_process')`返回子进程模块。

`spawn()`的第一个参数是要执行的程序的名字，这里是`ls`。第二个参数是命令行参数，以数组的形式传入。`spawn()`返回的对象是`ChildProcess`。

标准输入输出都是`Stream`。

### 2.3 从EventEmitter捕获数据

Node中很多对象都继承自`EventEmitter`。Now let’s modify our previous program to capture the child process’s output by listening for events on the stream. Open an editor to the watcher-spawn.js file from the previous section, then find the call to `fs.watch()`. Replace it with this:

file-system/watcher-spawn-parse.js

```js
    fs.watch(filename, function(){
        let
      		ls=spawn('ls',['-lh',filename]),
        	output='';
        ls.stdout.on('data', function(chunk){
        	output+=chunk.toString();
        });
        ls.on('close',function() {
        	let parts=output.split(/\s+/);
        	console.dir([parts[0], parts[4], parts[8]]);
        });
    });
```

事件可以携带附加信息，这些信息会进入监听器回调。

Calling `toString()` explicitly converts the buffer’s contents to a JavaScript string using Node’s default encoding (UTF-8). This means copying the content into Node’s heap, 可能会较慢。If you can, it’s better to work with buffers directly, but strings are more convenient.

`ChildProcess`也继承自`EventEmitter`。

After a child process has exited and all its streams have been flushed, it emits a `close` event.

### 2.4 异步读写文件

本节涉及`EventEmitter`的`error`事件和`err`回调参数。

读文件：

```js
    const fs=require('fs');
    fs.readFile('target.txt', function(err, data){
        if(err) { throw err; }
        console.log(data.toString());
    });
```

如果没有错误`err`是false，否则是一个`Error`对象。

写文件：

```js
    const fs=require('fs');
    fs.writeFile('target.txt', 'awittymessage', function(err){
    	if(err) { throw err; }
    	console.log("File saved!");
    });
```

#### 创建读写流

用`fs.createReadStream()`和`fs.createWriteStream()`创建可读流和可写流。例如下面的例子，将文件流pipe到标准输出：

file-system/cat.js

```js
	#!/usr/bin/env node --harmony
	require('fs').createReadStream(process.argv[2]).pipe(process.stdout);
```

Use `chmod` to make it executable:

	$chmod +x cat.js
	$./cat.js <file_name>

You can also listen for data events from the file stream instead of calling pipe().

```js
    const
      fs=require('fs'),
      stream=fs.createReadStream(process.argv[2]);
    stream.on('data',function(chunk){
      process.stdout.write(chunk);
    });
    stream.on('error',function(err){
      process.stderr.write("ERROR:"+err.message+"\n");
    });
```

使用某种`EventEmitter`时，处理错误的方式是监听`error`事件。

#### 同步文件访问阻塞事件循环

`fs`模块中的很多方法有同步版本。如`readFileSync`。使用这些方法，Node.js进程将阻塞直到I/O结束。Here’s an example of how to read a file using the `readFileSync()` method:

```js
    const
      fs=require('fs'),
      data=fs.readFileSync('target.txt');
    process.stdout.write(data.toString());
```

`readFileSync()`的返回值是一个Buffer。

#### 其他文件系统操作

Node’s fsmodule has many other methods that map nicely onto POSIX conventions. To name a few examples, you can copy() files and unlink()(delete) them. You can use chmod() to change permissions and mkdir() to create directories.

### 2.5 Node程序的两个阶段
