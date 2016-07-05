[toc]

## 4 使用Node访问文件系统

常见的文件系统：如 UNIX 的 UFS(Unix File System)，Windows 的 NTFS(New Technology File System)。

Plan 9 操作系统将所有的控制接口实现为文件，所有操作建模为文件操作。将文件作为一等公民也是 Unix 系统的哲学——把命名管道、socket都看做文件。

系统暴露的文件接口必须简单易用、一致、快速。Node的`file`模块就是这样的接口。

两方面处理：文件的流入流出（读写）和文件属性的修改（如文件权限）。

> For those with knowledge of C++ and a healthy curiosity, the filesystem implementation in Node can be browsed here: https://github.com/joyent/node/blob/master/src/node_file.cc.

### 4.1 目录；遍历文件和文件夹

While not all constructive, is a good read if one is interested in how filesystems are accessed by Node and other systems: http://vertxproject.wordpress.com/2012/05/09/vert-x-vs-node-js-simple-http-benchmarks/.

#### 4.1.1 文件类型

UNIX 系统中常见的6种文件：

- 普通文件：字节数组。不能包含其他文件。
- 目录：它们也是文件。
- Socket: 用于跨进程通信，允许进程间交换数据
- 具名pipe：A command such as `ps aux | grep node` creates a pipe, which is destroyed once the operation terminates. Named pipes are persistent and addressable, and can be used variously by multiple processes for IPC indefinitely.
- 设备文件：表示 I/O 设备，processes that accept streams of data. `/dev/null` is commonly an example of a character device file (accepts serial streams of I/O), and `/dev/sda` is an example of a block device file (allowing random access I/O for blocks of data) representing a data drive.
- Links: 指向其他文件，有 hard links 和 symbolic links两种。Hard links directly point to another file, and are indistinguishable from the target file. Symbolic links are indirect pointers, and are distinguishable from normal files.

Studying named pipes will reward the reader interested in understanding how Node was designed to work with streams and pipes. Try this from a terminal: `mkfifo namedpipe`

Following `ls –l` a listing similar to this will be shown:

```
	prw-r--r-- 1 system staff 0 May 01 07:52 namedpipe
```

Note the `p` flag in the file mode. You've created a named pipe. Now enter this into the same terminal, pushing some bytes into the named pipe: `echo "hello" > namedpipe`

看上去进程挂起了。实际没有：管道，像水管一样，必须在两边都打开时才能流通。于是，在另一个终端，运行：`cat namedpipe`。

#### 4.1.2 文件路径

操作文件路径可以用上`path`。

如果路径字符串的来源的是不可信的、不可靠的，利用`path.normalize`规范化：

```js
    var path = require('path');
    path.normalize("../one////two/./three.html");
    // -> ../one/two/three.html
```

利用`path.join`拼接路径：

```js
    path.join("../","one","two","three.html");
    // -> ../one/two/three.html
```

用 `path.dirname` 抽出路径中目录部分：

```javascript
    path.dirname("../one/two/three.html");
    // ../one/two
```

用 `path.basename` 取路径中最后一段：

```javascript
    path.basename("../one/two/three.html");
    // -> three.html
    // Remove file extension from the basename
    path.basename("../one/two/three.html", ".html");
    // -> three
```

利用`path.extname`读扩展名：最后一个(.)之后的部分：

```javascript
    var pstring = "../one/two/three.html";
    path.extname(pstring);
    // -> .html
    //
    // Which is identical to:
    // pstring.slice(pstring.lastIndexOf("."));
```

利用`path.relative`，生成第二个路径相对于第一个路径的相对路径：

```javascript
    path.relative('/one/two/three/four', '/one/two/thumb/war');
    // -> ../../thumb/war
```

利用`path.resolve`，沿多个路径找到最终的（绝对）路径：

```javascript
    path.resolve('/one/two', '/three/four');
    // -> /three/four
    path.resolve('/one/two/three', '../', 'four', '../../five')
    // -> /one/five
```

传入`path.resolve`的参数可以看做一系列的`cd`命令：

```
    cd /one/two/three
    cd ../
    cd four
    cd ../../five
    pwd
    // -> /one/five
```

如果`path.resolve`的参数列表无法推导出一个绝对路径，会利用当前目录。例如，如果当前在`/users/home/john/`：

```javascript
    path.resolve('one','two/three','four');
    // -> /users/home/john/one/two/three/four
```

#### 4.1.3 文件属性

利用`fs.stat`读取文件属性：

```javascript
    fs.stat("file.txt", function(err, stats) {
        console.log(stats);
    });
```

回调方法的第二个参数是`fs.Stats`对象。

    dev: 2051, // id of device containing this file
    mode: 33188, // bitmask, status of the file
    nlink: 1, // number of hard links
    uid: 0, // user id of file owner
    gid: 0, // group id of file owner
    rdev: 0, // device id (if device file)
    blksize: 4096, // I/O block size
    ino: 27396003, // a unique file inode number
    size: 2000736, // size in bytes
    blocks: 3920, // number of blocks allocated
    atime: Fri May 3 2013 15:39:57 GMT-0500 (CDT), // last access
    mtime: Fri May 3 2013 17:22:46 GMT-0500 (CDT), // last modified
    ctime: Fri May 3 2013 17:22:46 GMT-0500 (CDT) // last status change

`fs.Stats`对象有几个方法可以使用：

- `stats.isFile`
- `stats.isDirectory`
- `stats.isBlockDevice` to check for block type device files
- `stats.isCharacterDevice` to check for character type device files
- `stats.isSymbolicLink` after an `fs.lstat` to find symbolic links
- `stats.isFIFO` to identify named pipes
- `stats.isSocket`

还有两个stat方法：

- `fs.fstat(fd, callback)`: 与`fs.stat`类似，只是传入的是文件描述符`fd`，而非一个路径。
- `fs.lstat(path, callback)`：An `fs.stat` on a symbolic link will return an `fs.Stats` object for the target file, while `fs.lstat` will return an `fs.Stats` object for the link file itself

两个方法，可以简化对文件时间戳的操纵：

- `fs.utimes(path, atime, mtime, callback)`：Change the access and modify timestamps on a file at `path`. The access and modify times of a file are stored as instances of the JavaScript `Date` object. `Date.getTime` will, for example, return the number of milliseconds elapsed since midnight (UTC) on January 1, 1970.
- `fs.futimes(fd, atime, mtime, callback)`：Changes the access and modify timestamps on a file descriptor `fd`. Similar to `fs.utimes`.

> More information about manipulating dates and times with JavaScript can be found here: https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Date.


#### 4.1.4 打开和关闭文件

管理Node工程的一个非官方规则是，避免不必要的对OS实现细节的封装。例如，对 POSIX 来说，文件描述符仅是一个（非负）整数，唯一的引用一个文件。Since Node modeled its filesystem methods on POSIX, 因此 Node 中文件描述符是一个整数。

因为设备等在OS中也表示成文件，因此标准 I/O 流（stdin, stdout, stderr）也有文件描述符。

```javascript
    console.log(process.stdin.fd); // 0
    console.log(process.stdout.fd); // 1
    console.log(process.stderr.fd); // 2
    fs.fstat(1, function(err, stat) {
        console.log(stat); // an fs.Stats object
    });
```

##### 4.1.4.1 `fs.open(path, flags, [mode], callback)`

打开特定路径下的文件。回调方法的第一个参数用于接收异常，第二个参数是文件描述符。
例子：打开文件用于读取

```js
    fs.open("path.js", "r", function(err, fileDescriptor) {
        console.log(fileDescriptor); // An integer, like `7` or `23`
    });
```

`flags`是字符串，表示对文件操作的类型。

- `r`：打开文件用于读取。如果文件不存在，抛出异常。
- `r+`：打开文件，可以读写。如果文件不存在，抛出异常。
- `w`：打开文件，写。如果文件不存在，先创建一个。如果文件存在，将文件裁成0字节。
- `wx`：写。但排他模式，及如果文件已存在，操作将失败。
- `w+`：打开文件，读写。如果文件不存在，先创建一个。如果文件存在，将文件裁成0字节。
- `wx+`：类似于`wx`，只是还可以读。
- `a`：打开文件，用于追加。如果文件不存在，先创建一个。
- `ax`：类似于`a`，但用排他模式打开。如果文件已存在，操作将失败。
- `a+`：追加和读。如果文件不存在，先创建一个。
- `ax+`：类似于`ax`，只是还可以读。

如果操作可能创建一个文件，可选指定`mode`，设置文件权限。值是八进制，默认为`0666`（参见`fs.chmod`）：

```javascript
    fs.open("index.html","w", 755, function(err, fd) {
        fs.read(fd, …);
    });
```

##### 4.1.4.2 `fs.close(fd, callback)`

关闭一个文件描述符。回调方法只有一个参数，接收异常。It's a good habit to close all file descriptors that have been opened.

#### 4.1.5 文件操作

##### `fs.rename(oldName, newName, callback)`

重命名。回调方法只有一个参数，接收异常。

##### `fs.truncate(path, len, callback)`

改变指定路径上的文件长度为`len`字节。如果`len`比当前文件长度端，文件被截断。如果`len`更长，文件补null字节（\\x00）。回调方法只有一个参数，接收异常。

##### `fs.ftruncate(fd, len, callback)`

类似于`fs.truncate`，只是使用文件描述符而非路径。

##### `fs.chown(path, uid, gid, callback)`

The fs.chown(path, uid, gid, callback) method changes the ownership of the file at path. Use this to set whether user uid or group gid has access to a file. 回调方法只有一个参数，接收异常。

##### `fs.fchown(fd, uid, gid, callback)`

The fs.fchown(fd, uid, gid, callback) method is like `fs.chown`, except that instead of specifying a file path, a file descriptor is passed as fd.

##### `fs.lchown(path, uid, gid, callback)`

The fs.lchown(path, uid, gid, callback) method is like fs.chown, except that in the case of symbolic links ownership of the link file itself is changed, but not the referenced link.

##### `fs.chmod(path, mode, callback)`

The fs.chmod(path, mode, callback)methodchanges the mode (permissions) on a file at path. You are setting the read(4), write(2), and execute(1) bits for this file, which can be sent in octal digits:

| | [r]ead | [w]rite | E[x]ecute | Total |
|-| --- | --- | --- | --- |
| Owner | 4 | 2 | 1 | 7 |
| Group | 4 | 0 | 1 | 5 |
| Other | 4 | 0 | 1 | 5 |
|       |   |   |   | chmod(755) |

You may also use symbolic representations, such as `g+rw` for group read and write, similar to the arguments we saw for `file.open` earlier. For more information on setting file modes, consult: http://en.wikipedia.org/wiki/Chmod.

回调方法只有一个参数，接收异常。

##### `fs.fchmod(fd, mode, callback)`

The fs.fchmod(fd, mode, callback) method is like fs.chmod, except that instead of specifying a file path, a file descriptor is passed as fd.

##### `fs.lchmod(path, mode, callback)`

The fs.lchmod(path, mode, callback) method is like fs.chmod, except that in the case of symbolic links permissions on the link file itself is changed, but not those of the referenced link.

##### `fs.link(srcPath, dstPath, callback)`

The fs.link(srcPath, dstPath, callback) creates a *hard* link between srcPath
and dstPath. This is a way of creating many different paths to exactly the same file. For example, the following directory contains a file target.txt and two hard links—a.txt and b.txt—which each point to this file:

If the content of the target file is changed, the length of the link files will also be changed.

回调方法只有一个参数，接收异常。

##### `fs.symlink(srcPath, dstPath, [type], callback)`

The fs.symlink(srcPath, dstPath, [type], callback) method creates a symbolic link between srcPath and dstPath. Unlike hard links created with fs.link, symbolic links are simply pointers to other files, and do not themselves respond to changes in the target file. The default link `type` is file. Other options are directory and *junction*, the last being a Windows-specific type that is ignored on other systems. The callback receives one argument, any exception thrown in the call.

Unlike hard links, symbolic links do not change in length when their target file (in this case target.txt) changes length.

##### `fs.readlink(path, callback)`

The given symbolic link at path, returns the filename of the targeted file:
```javascript
fs.readlink('a.txt', function(err, targetFName) {
	console.log(targetFName); // target.txt
});
```

##### `fs.realpath(path, [cache], callback)`

The fs.realpath(path, [cache], callback) method attempts to find the real path to file at path. This is a useful way to find the absolute path to a file, resolve symbolic links, and even clean up extraneous slashes and other malformed paths. For example:
```javascript
fs.realpath('file.txt', function(err, resolvedPath) {
	console.log(resolvedPath); // `/real/path/to/file.txt`
});
```
Or, even:
```javascript
fs.realpath('.////./file.txt', function(err, resolvedPath) {
	// still `/real/path/to/file.txt`
});
```

If some of the path segments to be resolved are already known, one can pass a cache of mapped paths:
```javascript
var cache = {'/etc':'/private/etc'};
fs.realpath('/etc/passwd', cache, function(err, resolvedPath) {
	console.log(resolvedPath); // `/private/etc/passwd`
});
```

##### `fs.unlink(path, callback)`

移除特定路径上的文件，等价于删除文件。回调方法只有一个参数，接收异常。

##### `fs.rmdir(path, callback)`

移除指定文件夹。等价于删除目录。如果目录非空会抛出异常。回调方法只有一个参数，接收异常。

##### `fs.mkdir(path, [mode], callback)`

创建目录。To set the mode of the new directory, use the permission bit map described in `fs.chmod`.

如果目录已存在，抛出异常。回调方法只有一个参数，接收异常。

##### `fs.exists(path, callback)`

检查文件是否存在。回调方法接收一个布尔参数。

##### `fs.fsync(fd, callback)`

Between the instant a request for some data to be written to a file is made and that data fully exists on a storage device, the candidate data exists within core system buffers. This latency isn't normally relevant but, in some extreme cases, such as system crashes, it is necessary to insist that the file reflect a known state on a stable storage device.

fs.fsync copies all in-core data of a file referenced by file descriptor `fd` to disk (or other storage device). 回调方法只有一个参数，接收异常。

#### 4.1.6 同步方法

`file`模块为每个异步方法提供一个对应的同步方法，以`Sync`为后缀。例如`fs.mkdir`的同步版本是`fs.mkdirSync`。同步方法直接方法结果，不需要回调。

```javascript
    key: fs.readFileSync('server-key.pem'),
    cert: fs.readFileSync('server-cert.pem')
```

一个常见的同步操作是`require`指令，如`require('fs')`。

#### 4.1.7 遍历目录

编写一个遍历目录的功能，展示目录的层次结构。

```js
    var walk = function(dir, done) {
        var results = {};
        fs.readdir(dir, function(err, list) {
            var pending = list.length;
            if(err || !pending) {
                return done(err, results);
            }
            list.forEach(function(file) {
                var dfile = dir + "/" + file;
                fs.stat(dfile, function(err, stat) {
                    if(stat.isDirectory()) {
                        return walk(dfile, function(err, res) {
                            results[file] = res;
                            !--pending && done(null, results);
                        });
                    }
                    results[file] = stat;
                    !--pending && done(null, results);
                });
            });
        });
    };
    walk(".", function(err, res) {
        console.log(require('util').inspect(res, {depth: null}));
    });
```

Now let's publish events whenever a directory or file is encountered, giving any future implementation flexibility to construct its own representation of the filesystem. To do this we will use the friendly `EventEmitter` object:

```js
    var walk = function(dir, done, emitter) {
        ...
        emitter = emitter || new (require('events').EventEmitter);
        ...
        if(stat.isDirectory()) {
            emitter.emit('directory', dfile, stat);
            return walk(dfile, function(err, res) {
                results[file] = res;
                !--pending && done(null, results);
            }, emitter);
        }
        emitter.emit('file', dfile, stat);
        results[file] = stat;
        ...
        return emitter;
    }
    walk("/usr/local", function(err, res) {
        ...
    }).on("directory", function(path, stat) {
        console.log("Directory: " + path + " - " + stat.size);
    }).on("file", function(path, stat) {
        console.log("File: " + path + " - " + stat.size);
    });
    // File: index.html – 1024
    // File: readme.txt – 2048
    // Directory: images - 106
    // File images/logo.png - 4096
    // ...
```

### 4.2 读取文件

#### 4.2.1 逐字节读取：`fs.read`

`fs.read`方法是读取文件的最底层方法：`fs.read(fd, buffer, offset, length, position, callback)`


获取到文件描述符`fd`后，从文件位置`position`开始，读取`length`字节数据，插入到`buffer`（`Buffer`类）。`offset`指定在`buffer`中的插入位置。例如，从文件第309个字节，读取8366个字节，到`buffer`，从`buffer`的第100字节开始写入：
`fs.read(fd, buffer, 100, 8366, 309, callback)`。

The following code demonstrates how to open and read a file in 512 byte chunks:

```js
    fs.open('path.js', 'r', function(err, fd) {
        fs.fstat(fd, function(err, stats) {
            var totalBytes = stats.size;
            var buffer = new Buffer(totalBytes);
            var bytesRead  = 0;
            // Each call to read should ensure that chunk size is
            // within proper size ranges (not too small; not too large).
            var read = function(chunkSize) {
                fs.read(fd, buffer, bytesRead, chunkSize, bytesRead,
                    function(err, numBytes, bufRef) {
                        if((bytesRead += numBytes) < totalBytes) {
                            return read(Math.min(512, totalBytes - bytesRead));
                        }
                        fs.close(fd);
                        console.log("File read complete. Total bytes read: " + totalBytes);
                        // Note that the callback receives a reference to the
                        // accumulating buffer
                        console.log(bufRef.toString());
                    });
            }
            read(Math.min(512, totalBytes));
        });
    });
```

产生的buffer可以被重定向到任何地方（包括服务器响应对象）。还可以调用`Buffer`对象的方法如`buffer.toString("utf8")`。

#### 4.2.2 一次获取整个文件

`fs.readFile(path, [options], callback)`

```javascript
    fs.readFile('/etc/passwd', function(err, fileData) {
        if(err) throw err;
        console.log(fileData);
        // <Buffer 48 65 6C 6C 6F … >
    });
```

回调方法收到一个buffer。我们可以指定返回数据的编码和读取模式。利用`options`对象的两个特性：

* `encoding`：字符串，如`utf8`。默认为`null`，不编码
* `flag`：文件模式。默认为`r`。

例子：

```js
    fs.readFile('/etc/passwd', { encoding : "utf8" }, function(err, fileData) {
        ...
        console.log(fileData);
        // "Hello ..."
    });
```

#### 4.2.3 创建可读流

`fs.readFile`需要将整个文件读入内存，然后才回调。例如`fs.createReadStream`则可以以流的方式读取。可以对返回的流进行流操作，如`pipe()`。

`fs.createReadStream(path, [options])`

可用选项：

- `flags`：文件模式。默认为`r`。
- `encoding`：取值`utf8`, `ascii`, `base64`。默认不编码。
- `fd`：如果`path`为`null`，这里可以传文件标识符。
- `mode`：八进制表示的文件模式。默认为`0666`。
- `bufferSize`：内部读取流的chunk大小，单位字节。默认为64 * 1024字节。可以设置任何值，but memory allocation is strictly controlled by the host OS, which may ignore a request. See: https://groups.google.com/forum/?fromgroups#!topic/nodejs/p5FuU1oxbeY.
- `autoClose`：是否自动关闭文件描述符（`fs.close`）。默认为true。You may want to set this to false and close manually if you are sharing a file descriptor across many streams.
- `start`：开始读取位置。默认为0。
- `end`：停止读取位置。默认为文件长度。


#### 4.2.4 逐行读取

文本文件一般适于逐行读。More precisely, any stream can be understood in terms of the chunks of data separated by newline characters, typically "`\r\n`" on UNIX systems. Node provides a native module whose methods simplify access to newline-separated chunks in data streams.

`Readline`模块。The bulk of its interface is designed to make command-line prompting easier, such that interfaces taking user input are easier to design.

Remembering that Node is designed for I/O, that I/O operations normally involve moving data between readable and writable streams, and that `stdout` and `stdin` are stream interfaces identical with the file streams returned by `fs.createReadStream` and `fs.createWriteStream`, we will look at how this module can be similarly used to prompt file streams for a line of text.

> The `Readline` module will be used throughout the book (and later in this chapter), for purposes beyond the present discussion on reading files, such as when designing command-line interfaces. For the impatient, more information on this module can be found at: http://nodejs.org/api/readline.html.

To start working with the `Readline` module one must create an interface defining the input stream and the output stream. The default interface options prioritize usage as a terminal interface. The options we are interested in are:
- input: Required. The readable stream being listened to.
- output: Required. The writable stream being written to.
- terminal: Set this to true if both the input and output streams should be treated like a Unix terminal, or TTY(TeleTYpewriter). For files, you will set this to false.

Reading the lines of a file is made easy through this system. For example, assuming one has a dictionary file listing common words in the English language one might want to read the list into an array for processing:

```javascript
    var fs = require('fs');
    var readline = require('readline');
    var rl = readline.createInterface({
        input: fs.createReadStream("dictionary.txt"),
        terminal: false
    });
    var arr = [];
    rl.on("line", function(ln) {
        arr.push(ln.trim())
    });
    // aardvark
    // abacus
    // abaisance
    // ...
```

Note how we disable TTY behavior, handling the lines ourselves without redirecting to an output stream.

As expected with a Node I/O module, we are working with stream events. The events listeners that may be of interest are:

- line: Receives the most recently read line, as a string
- pause: Called whenever the stream is paused
- resume: Called whenever a stream is resumed
- close: Called whenever a stream is closed

Except for line, these event names reflect Readline methods, pause a stream with `Readline.pause`, resume with `Readline.resume`, and close with `Readline.close`.


### 4.3 写文件

#### 4.3.1 逐字节写

`fs.write(fd, buffer, offset, length, position, callback)`

To write the collection of bytes between positions 309 and 8675 (length 8366) of buffer to the file referenced by file descriptor fd, insertion beginning at position 100:

```javascript
    var buffer = new Buffer(8675);
    fs.open("index.html", "w", function(err, fd) {
        fs.write(fd, buffer, 309, 8366, 100, function(err, writtenBytes, buffer) {
            console.log("Wrote " + writtenBytes + " bytes to file");
            // Wrote 8366 bytes to file
        });
    });
```

> Note that for files opened in append (a) mode some operating systems may ignore positionvalues, always adding data to the end of the file. Additionally, it is unsafe to call fs.write multiple times on the same file without waiting for the callback. Use `fs.createWriteStream` in those cases.

利用精确的控制，我们可以结构化文件。In the following (somewhat contrived) example we create a file-based database containing indexed information for six months of baseball scores for a single team. We want to be able to quickly look up whether this team won or lost (or did not play) on a given day.

Since a month can have at most 31 days, we can (randomly) create a 6 x 31 grid of data in this file, placing one of three values in each grid cell: L (loss), W (win), N (no game). For fun, we also create a simple CLI(Command-Line Interface) to our database with a basic query language. This example should make it clear how `fs.read`, `fs.write` and `Buffer` objects are used to precisely manipulate bytes in files:

```javascript
    var fs = require('fs');
    var readline = require('readline');
    var cells = 186; // 6 x 31
    var buffer = new Buffer(cells);
    while(cells--) {
        // 0, 1 or greater
        var rand = Math.floor(Math.random() * 3);
        // 78 = "N", 87 = "W", 76 = "L"
        buffer[cells] = rand === 0 ? 78 : rand === 1 ? 87 : 76;
    }
    fs.open("scores.txt", "r+", function(err, fd) {
        fs.write(fd, buffer, 0, buffer.length, 0, function(err, writtenBytes, buffer) {
            var rl = readline.createInterface({
                input: process.stdin,
                output: process.stdout
            });
            var quest = function() {
                rl.question("month/day:", function(index) {
                    if(!index) {
                        return rl.close();
                    }
                    var md = index.split('/');
                    var pos = parseInt(md[0] -1) * 31 + parseInt(md[1] -1);
                    fs.read(fd, new Buffer(1), 0, 1, pos, function(err, br, buff) {
                       var v = buff.toString();
                       console.log(v === "W" ? "Win!" : v === "L" ? "Loss..." : "No game");
                       quest();
                    });
                });
            };
        quest();
        });
    });
```

#### 4.3.2 将整块数据写到文件

##### `fs.writeFile(path, data, [options], callback)`

将`data`写入文件。`data`可以是一个`Buffer`或字符串。可用选项：

- `encoding`：默认为`utf8`。如果`data`是buffer，该选项会被忽略。
- `mode`：八进制文件模式，默认是`0666`。
- `flag`：写标志，默认为`w`。

例子
```javascript
    fs.writeFile('test.txt', 'A string or Buffer of data', function(err) {
        if(err) return console.log(err);
        // File has been written
    });
```

##### `fs.appendFile(path, data, [options], callback)`

类似于`fs.writeFile`，只是`data`被追加到文件的尾部。`flag`选项默认为`a`。


#### 4.3.3 创建可写流

If the data being written to a file arrives in chunks (such as occurs with a file upload), streaming that data through a `WritableStream` object interface provides more flexibility and efficiency.

`fs.createWriteStream(path, [options])`

`fs.createWriteStream(path, [options])`方法返回一个可写流。选项：

- flags：文件模式。默认为`w`。
- encoding：One of utf8, ascii, or base64. 默认无编码。
- mode: Octal representation of file mode, defaulting to 0666.
- start: 写入的起始位置。

For example, this little program functions as the world's simplest word processor, writing all terminal input to a file, until the terminal is closed:

```javascript
    var writer = fs.createWriteStream("novel.txt", 'w');
    process.stdin.pipe(writer);
```

#### 4.3.4 警告

写文件时的注意事项：

- 磁盘空间是否足够？
- 是否有另一个进程在同时访问此文件，甚至在擦除它？
- What must be done if a write operation fails or is unnaturally terminated mid-stream?



### 4.4 伺服静态文件

一个基本的文件服务器：

```javascript
    http.createServer(function(request, response) {
        if(request.method !== "GET") {
            return response.end("Simple File Server only does GET");
        }
        fs.createReadStream(__dirname + request.url).pipe(response);
    }).listen(8000);
```



#### 4.4.1 重定向请求

重定向用到两个响应头：

- `Location`: This indicates a redirection to a location where said content body can be found
- `Content-Location`: This is meant to indicate the URL where the requester will find the original location of the entity enclosed in the response body

> There are many possible pairings of `Location` and `Content-Location` headers with HTTP status codes, the *3xx (redirection)* set in particular. In fact, these headers may even appear together in the same response. The user is encouraged to read the relevant sections of the HTTP/1.1 specification, as only a small set of common cases is discussed here.



##### Location

Responding toa *POST* with a *201* status code indicates a new resource has been created its URI assigned to the `Location` header, and that the client may go ahead and use that URI in the future.

例子：

```
	POST /path/addUser HTTP/1.1
	Content-Type: application/x-www-form-urlencoded
	name=John&group=friends
	…
	Status: 201
	Location: http://website.com/users/john.html
```

Similarly,in cases where a resource creation request has been accepted but not yet fulfilled, a server would indicate a status of *202*.

##### Content-Location

When a GET is made to a resource that has multiple representations, and those can be found at distinct resource locations, a `content-location` header for the particular entity should be returned.

For example, content format negotiation is a good candidate for `Content-Location` handling. One might be interested in retrieving all blog posts for a given month, perhaps available at a URL such as `http://example.com/september/`. GET requests with an *Accept* header of `application/json` will receive a response in JSON format. A request for XML will receive that representation.

如果启用了环境，这些资源可能有多个永久地址，如`http://example.com/cache/september.json` or `http://example.com/cache/september.xml`. One would send this additional location information via `Content-Location`, in a response object resembling:

```
    Status: 200
    Content-Type: application/json
    Content-Location: http://blogs.com/cache/allArticles.json
    … JSON entity body
```

In cases where the requested URL has been moved, permanently or temporarily, the *3xx* group of status codes can be used with `Content-Location` to indicate this state. For example, to redirect a request to a URL which has been permanently moved one should send a 301 code:

```js
    function requestHandler(request, response) {
        var newPath = "/thedroids.html";
        response.writeHead(301, {
            'Content-Location': newPath
        });
        response.end();
    }
```

#### （未）4.4.2 实现资源缓存

### （未）4.5 文件上传