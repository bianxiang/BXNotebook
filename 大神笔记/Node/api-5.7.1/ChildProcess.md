[toc]

# 子进程

https://nodejs.org/dist/latest-v5.x/docs/api/child_process.html

> Stability: 2 - Stable

`child_process` 模块用于产生（spawn）子进程。方式类似于 [popen(3)](http://linux.die.net/man/3/popen)。This capability is primarily provided by the `child_process.spawn()` function:

```js
const spawn = require('child_process').spawn;
const ls = spawn('ls', ['-lh', '/usr']);

ls.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

ls.stderr.on('data', (data) => {
  console.log(`stderr: ${data}`);
});

ls.on('close', (code) => {
  console.log(`child process exited with code ${code}`);
});
```

By default, pipes for `stdin`, `stdout` and `stderr` are established between the parent Node.js process and the spawned child. It is possible to stream data through these pipes in a non-blocking way. Note, however, that some programs use line-buffered I/O internally. While that does not affect Node.js, it can mean that data sent to the child process may not be immediately consumed.

`child_process.spawn()` 方法产生子进程是异步地。`child_process.spawnSync()` 是等价的同步的方式；会阻塞事件循环，直到产生的进程终止（that blocks the event loop until the spawned process either exits of is terminated）。

除了 `child_process.spawn()` 和 `child_process.spawnSync()`，为了使用方便，`child_process` 模块还提供了几个其他同步或异步的方法。这些方法都是对 `child_process.spawn()` 或 `child_process.spawnSync()` 的封装。

- `child_process.exec()`：spawns a shell and runs a command within that shell, passing the `stdout` and `stderr` to a callback function when complete.
- `child_process.execFile()`: similar to `child_process.exec()` except that it spawns the command directly without first spawning a shell.
- `child_process.fork()`: spawns a new Node.js process and invokes a specified module with an IPC communication channel established that allows sending messages between parent and child.
- `child_process.execSync()`: a synchronous version of `child_process.exec()` that will block the Node.js event loop.
- `child_process.execFileSync()`: a synchronous version of `child_process.execFile()` that will block the Node.js event loop.

For certain use cases, such as automating shell scripts, the synchronous counterparts may be more convenient.

## 异步的进程创建

`child_process.spawn()`, `child_process.fork()`, `child_process.exec()`, and `child_process.execFile()` 方法都是异步的。它们返回一个 `ChildProcess` 对象。该对象实现 Node.js EventEmitter API，使得父进程可以注册监听器函数，监听子进程的生命周期。

The `child_process.exec()` and `child_process.execFile()` methods additionally allow for an optional callback function to be specified that is invoked when the child process terminates.

### （未）Spawning .bat and .cmd files on Windows

### `child_process.exec(command[, options][, callback])`

- `command`：字符串。要运行的命令。可带空格分隔的参数。
- `options`：对象
  - `cwd`：字符串。子进程的当前工作目录。
  - `env`：对象。环境键值对。
  - `encoding`：字符串。默认 'utf8'
  - `shell`：字符串。Shell to execute the command with (Default: '/bin/sh' on UNIX, 'cmd.exe' on Windows, The shell should understand the `-c` switch on UNIX or `/s /c` on Windows. On Windows, command line parsing should be compatible with cmd.exe.)
  - `timeout`：数字。默认0。毫秒。If `timeout` is greater than 0, the parent will send the the signal identified by the `killSignal` property if the child runs longer than `timeout` milliseconds.
  - `maxBuffer`：数字。largest amount of data (in bytes) allowed on stdout or stderr - if exceeded child process is killed (Default: 200*1024)
  - `killSignal`：字符串。默认 'SIGTERM'。
  - `uid`：数字。Sets the user identity of the process. (See [setuid(2)](http://man7.org/linux/man-pages/man2/setuid.2.html).)
  - `gid`：数字。Sets the group identity of the process. (See [setgid(2)](http://man7.org/linux/man-pages/man2/setgid.2.html).)
- `callback`：函数。进程终止后调用。
  - `error`：`Error`。
  - `stdout`：`Buffer`。
  - `stderr`：`Buffer`。
- 返回值：`ChildProcess`

Spawns a shell then executes the command within that shell, buffering any generated output.

```js
const exec = require('child_process').exec;
const child = exec('cat *.js bad_file | wc -l',
  (error, stdout, stderr) => {
    console.log(`stdout: ${stdout}`);
    console.log(`stderr: ${stderr}`);
    if (error !== null) {
      console.log(`exec error: ${error}`);
    }
});
```

若提供了回调函数：成功后，`error` 为 null。错误后，`error` 是一个 `Error` 对象。`error.code` 属性是子进程的退出码，`error.signal` 是让进程终止的信号。非0的退出码一般表示错误。

> Note: Unlike the `exec()` POSIX system call, `child_process.exec()` does not replace the existing process and uses a shell to execute the command.

### `child_process.execFile(file[, args][, options][, callback])`

- `file`：字符串。要运行的可执行文件的名字或路径。
- `args`：数组。字符串参数列表。
- `options`：对象
- `options`：对象
  - `cwd`：字符串。子进程的当前工作目录。
  - `env`：对象。环境键值对。
  - `encoding`：字符串。默认 'utf8'
  - `shell`：字符串。Shell to execute the command with (Default: '/bin/sh' on UNIX, 'cmd.exe' on Windows, The shell should understand the `-c` switch on UNIX or `/s /c` on Windows. On Windows, command line parsing should be compatible with cmd.exe.)
  - `timeout`：数字。默认0。毫秒。If `timeout` is greater than 0, the parent will send the the signal identified by the `killSignal` property if the child runs longer than `timeout` milliseconds.
  - `maxBuffer`：数字。largest amount of data (in bytes) allowed on stdout or stderr - if exceeded child process is killed (Default: 200*1024)
  - `killSignal`：字符串。默认 'SIGTERM'。
  - `uid`：数字。Sets the user identity of the process. (See [setuid(2)](http://man7.org/linux/man-pages/man2/setuid.2.html).)
  - `gid`：数字。Sets the group identity of the process. (See [setgid(2)](http://man7.org/linux/man-pages/man2/setgid.2.html).)
- `callback`：函数。进程终止后调用。
  - `error`：`Error`。
  - `stdout`：`Buffer`。
  - `stderr`：`Buffer`。
- 返回值：`ChildProcess`

The `child_process.execFile()` function is similar to `child_process.exec()` except that it does not spawn a shell. Rather, the specified executable file is spawned directly as a new process，因为它比 `child_process.exec()` 高效一点。

Since a shell is not spawned, behaviors such as I/O redirection and file globbing are not supported.

```js
const execFile = require('child_process').execFile;
const child = execFile('node', ['--version'], (error, stdout, stderr) => {
  if (error) {
    throw error;
  }
  console.log(stdout);
});
```

### `child_process.fork(modulePath[, args][, options])`

- `modulePath`：字符串。The module to run in the child
- `args`：数组。字符串参数列表。
- `options`：对象
  - `cwd`：字符串。子进程的当前工作目录。
  - `env`：对象。环境键值对。
  - `execPath`：字符串。Executable used to create the child process
  - `execArgv`：数组。List of string arguments passed to the executable (Default: `process.execArgv`)
  - `silent`：布尔。If true, stdin, stdout, and stderr of the child will be piped to the parent, otherwise they will be inherited from the parent, see the 'pipe' and 'inherit' options for child_process.spawn()'s stdio for more details (default is false)
  - `uid`：数字。Sets the user identity of the process. (See [setuid(2)](http://man7.org/linux/man-pages/man2/setuid.2.html).)
  - `gid`：数字。Sets the group identity of the process. (See [setgid(2)](http://man7.org/linux/man-pages/man2/setgid.2.html).)
- 返回值：`ChildProcess`

The `child_process.fork()` method is a special case of `child_process.spawn()` used specifically to spawn new Node.js processes. 返回的 `ChildProcess` 有额外的通信信道，允许消息在父子进程直接收、发。See `ChildProcess#send()` for details.

产生的 Node.js 子进程与父进程是独立的，除了二者之间的 IPC 通信信道。每个进程有自己的内存，自己的 V8 实例。因为需要额外的资源分配，因此产生不推荐大量 Node.js 进程。

By default, `child_process.fork()` will spawn new Node.js instances using the `process.execPath` of the parent process. The `execPath` property in the `options` object allows for an alternative execution path to be used.

Node.js processes launched with a custom `execPath` will communicate with the parent process using the file descriptor (fd) identified using the environment variable `NODE_CHANNEL_FD` on the child process. The input and output on this fd is expected to be line delimited JSON objects.

> Note: Unlike the `fork()` POSIX system call, `child_process.fork()` does not clone the current process.

### `child_process.spawn(command[, args][, options])`

- `command`。字符串。要运行的命令。
- `args`：字符串数组。命令行参数列表。
- `options`：对象
  - `cwd`：字符串。子进程的当前工作目录。若不指定，继承。
  - `env`：对象。环境键值对。
  - `stdio`：数组或字符串。Child's stdio configuration. 见下文
  - `detached`：布尔。Prepare child to run independently of its parent process. Specific behavior depends on the platform. 见下文
  - `uid`：数字。Sets the user identity of the process. (See [setuid(2)](http://man7.org/linux/man-pages/man2/setuid.2.html).)
  - `gid`：数字。Sets the group identity of the process. (See [setgid(2)](http://man7.org/linux/man-pages/man2/setgid.2.html).)
  - `shell`：布尔或字符串。If true, runs command inside of a shell. Uses '/bin/sh' on UNIX, and 'cmd.exe' on Windows. A different shell can be specified as a string. The shell should understand the `-c` switch on UNIX, or `/s /c` on Windows. Defaults to `false` (no shell).
- 返回值：`ChildProcess`

The `child_process.spawn()` method spawns a new process using the given command.

`options` 的默认值是：`{cwd: undefined, env: process.env}`。

Example of running `ls -lh /usr`, capturing stdout, stderr, and the exit code:

```js
const spawn = require('child_process').spawn;
const ls = spawn('ls', ['-lh', '/usr']);

ls.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

ls.stderr.on('data', (data) => {
  console.log(`stderr: ${data}`);
});

ls.on('close', (code) => {
  console.log(`child process exited with code ${code}`);
});
```

Example: A very elaborate way to run 'ps ax | grep ssh'

```js
const spawn = require('child_process').spawn;
const ps = spawn('ps', ['ax']);
const grep = spawn('grep', ['ssh']);

ps.stdout.on('data', (data) => {
  grep.stdin.write(data);
});

ps.stderr.on('data', (data) => {
  console.log(`ps stderr: ${data}`);
});

ps.on('close', (code) => {
  if (code !== 0) {
    console.log(`ps process exited with code ${code}`);
  }
  grep.stdin.end();
});

grep.stdout.on('data', (data) => {
  console.log(`${data}`);
});

grep.stderr.on('data', (data) => {
  console.log(`grep stderr: ${data}`);
});

grep.on('close', (code) => {
  if (code !== 0) {
    console.log(`grep process exited with code ${code}`);
  }
});
```

Example of checking for failed exec:

```js
const spawn = require('child_process').spawn;
const child = spawn('bad_command');

child.on('error', (err) => {
  console.log('Failed to start child process.');
});
```

#### `options.detached`

On Windows, setting `options.detached` to true makes it possible for the child process to continue running after the parent exits. The child will have its own console window. Once enabled for a child process, it cannot be disabled.

On non-Windows platforms, if `options.detached` is set to true, the child process will be made the leader of a new process group and session. Note that child processes may continue running after the parent exits regardless of whether they are detached or not. See `setsid(2)` for more information.

父进程默认会等待 detached 子进程退出。To prevent the parent from waiting for a given child, use the `child.unref()` method. Doing so will cause the parent's event loop to not include the child in its reference count, allowing the parent to exit independently of the child, unless there is an established IPC channel between the child and parent.

Example of detaching a long-running process and redirecting its output to a file:

```js
const fs = require('fs');
const spawn = require('child_process').spawn;
const out = fs.openSync('./out.log', 'a');
const err = fs.openSync('./out.log', 'a');

const child = spawn('prg', [], {
 detached: true,
 stdio: [ 'ignore', out, err ]
});

child.unref();
```

利用 `detached` 选项开始一个长时间运行的进程，父进程退出后，进程无法在后台运行，除非子进程配置了 `stdio`，且不与父进程连接。If the parent's `stdio` is inherited, the child will remain attached to the controlling terminal.

#### `options.stdio`

The `options.stdio` option is used to configure the pipes that are established between the parent and child process. By default, the child's stdin, stdout, and stderr are redirected to corresponding child.stdin, child.stdout, and child.stderr streams on the `ChildProcess` object. This is equivalent to setting the `options.stdio` equal to `['pipe', 'pipe', 'pipe']`.

For convenience, `options.stdio` may be one of the following strings:

- `'pipe'` - equivalent to `['pipe', 'pipe', 'pipe']` (the default)
- `'ignore'` - equivalent to `['ignore', 'ignore', 'ignore']`
- `'inherit'` - equivalent to `[process.stdin, process.stdout, process.stderr]` or `[0,1,2]`

Otherwise, the value of `option.stdio` is an array where each index corresponds to an fd in the child. The fds 0, 1, and 2 correspond to stdin, stdout, and stderr, respectively. Additional fds can be specified to create additional pipes between the parent and child. The value is one of the following:

- `'pipe'` - Create a pipe between the child process and the parent process. The parent end of the pipe is exposed to the parent as a property on the `child_process` object as `ChildProcess.stdio[fd]`. Pipes created for fds 0 - 2 are also available as `ChildProcess.stdin`, `ChildProcess.stdout` and `ChildProcess.stderr`, respectively.
- `'ipc'` - Create an IPC channel for passing messages/file descriptors between parent and child. A `ChildProcess` may have at most one IPC stdio file descriptor. Setting this option enables the `ChildProcess.send()` method. If the child writes JSON messages to this file descriptor, the `ChildProcess.on('message')` event handler will be triggered in the parent. If the child is a Node.js process, the presence of an IPC channel will enable `process.send()`, `process.disconnect()`, `process.on('disconnect')`, and `process.on('message')` within the child.
- `'ignore'` - Instructs Node.js to ignore the fd in the child. While Node.js will always open fds 0 - 2 for the processes it spawns, setting the fd to 'ignore' will cause Node.js to open `/dev/null` and attach it to the child's fd.
- Stream object - Share a readable or writable stream that refers to a tty, file, socket, or a pipe with the child process. The stream's underlying file descriptor is duplicated in the child process to the fd that corresponds to the index in the stdio array. Note that the stream must have an underlying descriptor (file streams do not until the 'open' event has occurred).
- Positive integer - The integer value is interpreted as a file descriptor that is is currently open in the parent process. It is shared with the child process, similar to how Stream objects can be shared.
- null, undefined - Use default value. For stdio fds 0, 1 and 2 (in other words, stdin, stdout, and stderr) a pipe is created. For fd 3 and up, the default is 'ignore'.

Example:

```js
const spawn = require('child_process').spawn;

// Child will use parent's stdios
spawn('prg', [], { stdio: 'inherit' });

// Spawn child sharing only stderr
spawn('prg', [], { stdio: ['pipe', 'pipe', process.stderr] });

// Open an extra fd=4, to interact with programs presenting a
// startd-style interface.
spawn('prg', [], { stdio: ['pipe', null, null, null, 'pipe'] });
```

若父子进程之间建立了 IPC 信道，且子进程是一个 Node.js 进程，the child is launched with the IPC channel unreferenced (using` unref()`) until the child registers an event handler for the `process.on('disconnected')` event. This allows the child to exit normally without the process being held open by the open IPC channel.

## （未）Synchronous Process Creation

## Class: `ChildProcess`

`ChildProcess` 类是一个 [EventEmitters](https://nodejs.org/dist/latest-v5.x/docs/api/events.html#events_class_events_eventemitter).

Instances of `ChildProcess` are not intended to be created directly. Rather, use the `child_process.spawn()`, `child_process.exec()`, `child_process.execFile()`, or `child_process.fork()` methods to create instances of `ChildProcess`.

### Event: 'close'

- `code`：数字。退出码，若子进程自己退出了。
- `signal`：字符串。让进程中止的信号。

The 'close' event is emitted when the stdio streams of a child process have been closed. 它与 `'exit'` 事件不同，因为多个进程可能共享同一个 stdio 流。

### Event: 'disconnect'

The `'disconnect'` event is emitted after calling the `ChildProcess.disconnect()` method in the parent or child process. After disconnecting it is no longer possible to send or receive messages, and the `ChildProcess.connected` property is false.

### Event: 'error'

- `err`：`Error`。错误。

The 'error' event is emitted whenever:

- The process could not be spawned, or
- The process could not be killed, or
- 给子进程发消息失败。

注意，错误发生后，不一定会发出 `'exit'` 事件。若同时监听了 `'exit'` 和 `'error'`，要注意保护不让回调函数多次执行。

See also ChildProcess#kill() and ChildProcess#send().

### Event: 'exit'

- `code`：数字。退出码，若子进程自己退出了。
- `signal`：字符串。让进程中止的信号。

The 'exit' event is emitted after the child process ends. If the process exited, code is the final exit code of the process, otherwise null. If the process terminated due to receipt of a signal, signal is the string name of the signal, otherwise null. 二者至少有一个非空。

Note that when the 'exit' event is triggered, child process `stdio` streams might still be open.

Also, note that Node.js establishes signal handlers for `SIGINT` and `SIGTERM` and Node.js processes will not terminate immediately due to receipt of those signals. Rather, Node.js will perform a sequence of cleanup actions and then will re-raise the handled signal.

See `waitpid(2)`.

### Event: 'message'

- `message`：对象，解析好的 JSON 对象，或基本类型值
- `sendHandle`：[Handle](https://nodejs.org/dist/latest-v5.x/docs/api/net.html#net_server_listen_handle_backlog_callback) a [net.Socket](https://nodejs.org/dist/latest-v5.x/docs/api/net.html#net_class_net_socket) or [net.Server](https://nodejs.org/dist/latest-v5.x/docs/api/net.html#net_class_net_server) object, or undefined.

当子进程用 `process.send()` 发消息后触发该事件。

### child.connected

布尔值。Set to false after .disconnect is called

The `child.connected` property indicates whether it is still possible to send and receive messages from a child process. 当 `child.connected` 为 false，不能再收发信息。

### child.disconnect()

关闭父子进程之间的 IPC 信道，allowing the child to exit gracefully once there are no other connections keeping it alive. After calling this method the `child.connected` and `process.connected` properties in both the parent and child (respectively) will be set to false, and it will be no longer possible to pass messages between the processes.

The 'disconnect' event will be emitted when there are no messages in the process of being received. This will most often be triggered immediately after calling `child.disconnect()`.

Note that when the child process is a Node.js instance (e.g. spawned using `child_process.fork()`), 子进程可以通过 `process.disconnect()` 方法关闭 IPC 信道。

### child.kill([signal])

- `signal`：字符串。

向子进程发一个信号。If no argument is given, the process will be sent the `'SIGTERM'` signal. See `signal(7)` for a list of available signals.

```js
const spawn = require('child_process').spawn;
const grep = spawn('grep', ['ssh']);

grep.on('close', (code, signal) => {
  console.log(
    `child process terminated due to receipt of signal ${signal}`);
});

// Send SIGHUP to process
grep.kill('SIGHUP');
```

The `ChildProcess` object may emit an 'error' event if the signal cannot be delivered. Sending a signal to a child process that has already exited is not an error but may have unforeseen consequences. 特别是当PID已经被重新分配给其他进程，the signal will be delivered to that process instead which can have unexpected results.

Note that while the function is called kill, the signal delivered to the child process may not actually terminate the process.

See `kill(2)`

### child.pid

整数。子进程的 PID。

```js
const spawn = require('child_process').spawn;
const grep = spawn('grep', ['ssh']);

console.log(`Spawned child pid: ${grep.pid}`);
grep.stdin.end();
```

### child.send(message[, sendHandle][, callback])

- `message`：对象
- `sendHandle`：[Handle](https://nodejs.org/dist/latest-v5.x/docs/api/net.html#net_server_listen_handle_backlog_callback)
- `callback`：函数。
- 返回：布尔。

When an IPC channel has been established between the parent and child ( i.e. when using child_process.fork()), the child.send() method can be used to send messages to the child process. When the child process is a Node.js instance, these messages can be received via the process.on('message') event.

For example, in the parent script:

```js
const cp = require('child_process');
const n = cp.fork(`${__dirname}/sub.js`);

n.on('message', (m) => {
  console.log('PARENT got message:', m);
});

n.send({ hello: 'world' });
```

And then the child script, 'sub.js' might look like this:

```js
process.on('message', (m) => {
  console.log('CHILD got message:', m);
});

process.send({ foo: 'bar' });
```

Child Node.js processes will have a `process.send()` method of their own that allows the child to send messages back to the parent.

There is a special case when sending a `{cmd: 'NODE_foo'}` message. All messages containing a `NODE_` prefix in its `cmd` property are considered to be reserved for use within Node.js core and will not be emitted in the child's `process.on('message')` event. Rather, such messages are emitted using the `process.on('internalMessage')` event and are consumed internally by Node.js. Applications should avoid using such messages or listening for `'internalMessage'` events as it is subject to change without notice.

可选的 `sendHandle` 参数用于向子进程传入一个 TCP server 或 socket 对象。子进程在 `process.on('message')` 事件的回调函数的第二个参数收到该参数。

The optional `callback` is a function that is invoked after the message is sent but before the child may have received it. The function is called with a single argument: null on success, or an Error object on failure.

若不提供回调函数，消息发送失败，`ChildProcess` 会报 'error' 事件。这种情况是存在的，如子进程已退出。

The `callback` function can be used to implement flow control.

当信号已关闭，或未发送出的消息已满，`child.send()` 返回 false。否则它返回 true。

#### 例子：发送一个 server 对象

利用 `sendHandle` 参数：

```js
const child = require('child_process').fork('child.js');
// Open up the server object and send the handle.
const server = require('net').createServer();
server.on('connection', (socket) => {
  socket.end('handled by parent');
});
server.listen(1337, () => {
  child.send('server', server);
});
```

子进程接受：

```js
process.on('message', (m, server) => {
  if (m === 'server') {
    server.on('connection', (socket) => {
      socket.end('handled by child');
    });
  }
});
```

Once the server is now shared between the parent and child, some connections can be handled by the parent and some by the child.

While the example above uses a server created using the `net` module, `dgram` module servers use exactly the same workflow with the exceptions of listening on a `'message'` event instead of `'connection'` and using `server.bind` instead of `server.listen`. This is, however, currently only supported on UNIX platforms.

#### 例子：发送一个 socket 对象

Similarly, the `sendHandler` argument can be used to pass the handle of a socket to the child process. The example below spawns two children that each handle connections with "normal" or "special" priority:

```js
const normal = require('child_process').fork('child.js', ['normal']);
const special = require('child_process').fork('child.js', ['special']);

// Open up the server and send sockets to child
const server = require('net').createServer();
server.on('connection', (socket) => {

  // If this is special priority
  if (socket.remoteAddress === '74.125.127.100') {
    special.send('socket', socket);
    return;
  }
  // This is normal priority
  normal.send('socket', socket);
});
server.listen(1337);
```

The child.js would receive the socket handle as the second argument passed to the event callback function:

```js
process.on('message', (m, socket) => {
  if (m === 'socket') {
    socket.end(`Request handled with ${process.argv[2]} priority`);
  }
});
```

Once a socket has been passed to a child, the parent is no longer capable of tracking when the socket is destroyed. To indicate this, the `.connections` property becomes `null`. It is recommended not to use `.maxConnections` when this occurs.

### child.stderr

A Readable Stream that represents the child process's stderr.

If the child was spawned with stdio[2] set to anything other than 'pipe', then this will be undefined.

child.stderr is an alias for child.stdio[2]. Both properties will refer to the same value.

### child.stdin

A Writable Stream that represents the child process's stdin.

Note that if a child process waits to read all of its input, the child will not continue until this stream has been closed via `end()`.

If the child was spawned with stdio[0] set to anything other than 'pipe', then this will be undefined.

child.stdin is an alias for child.stdio[0]. Both properties will refer to the same value.

### child.stdout

A Readable Stream that represents the child process's stdout.

If the child was spawned with stdio[1] set to anything other than 'pipe', then this will be undefined.

child.stdout is an alias for child.stdio[1]. Both properties will refer to the same value.

### child.stdio

一个数组。
A sparse array of pipes to the child process, corresponding with positions in the stdio option passed to c`hild_process.spawn()` that have been set to the value 'pipe'. Note that child.stdio[0], child.stdio[1], and child.stdio[2] are also available as child.stdin, child.stdout, and child.stderr, respectively.

In the following example, only the child's fd 1 (stdout) is configured as a pipe, so only the parent's child.stdio[1] is a stream, all other values in the array are null.

```js
const assert = require('assert');
const fs = require('fs');
const child_process = require('child_process');

const child = child_process.spawn('ls', {
    stdio: [
      0, // Use parents stdin for child
      'pipe', // Pipe child's stdout to parent
      fs.openSync('err.out', 'w') // Direct child's stderr to a file
    ]
});

assert.equal(child.stdio[0], null);
assert.equal(child.stdio[0], child.stdin);

assert(child.stdout);
assert.equal(child.stdio[1], child.stdout);

assert.equal(child.stdio[2], null);
assert.equal(child.stdio[2], child.stderr);
```

