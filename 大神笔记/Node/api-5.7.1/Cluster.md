[toc]

# 集群

https://nodejs.org/dist/latest-v5.x/docs/api/cluster.html

Stability: 2 - Stable

一个 Node.js 实例运行在一个线程。要充分利用多个系统，可以用 Node.js 进程集群。

集群模式允许你更加容易的创建子进程，共享服务器端口。

```js
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  // Fork workers.
  for (var i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`worker ${worker.process.pid} died`);
  });
} else {
  // Workers can share any TCP connection
  // In this case it is an HTTP server
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('hello world\n');
  }).listen(8000);
}
```

Running Node.js will now share port 8000 between the workers:

```
$ NODE_DEBUG=cluster node server.js
23521,Master Worker 23524 online
23521,Master Worker 23526 online
23521,Master Worker 23523 online
23521,Master Worker 23528 online
```

Please note that, on Windows, it is not yet possible to set up a named pipe server in a worker.

## How It Works

工作进程通过 `child_process.fork()` 产生，因此他们可以通过 IPC 与主进程通信，and pass server handles back and forth.

`cluster` 模块支持两种分发请求连接的分发。

第一种（默认，Windows下除外），round-robin 方式，主进程监听端口，接受新连接，and distributes them across the workers in a round-robin fashion, with some built-in smarts to avoid overloading a worker process.

第二种方式，where the master process creates the listen socket and sends it to interested workers. The workers then accept incoming connections directly.

第二种分发，理论上，性能更高。现实中，分发一般非常不平衡，由于操作系统调度器（scheduler）的行为。Loads have been observed where over 70% of all connections ended up in just two processes, out of a total of eight.

Because `server.listen()` hands off most of the work to the master process, there are three cases where the behavior between a normal Node.js process and a cluster worker differs:

- `server.listen({fd: 7})` Because the message is passed to the master, file descriptor 7 in the parent will be listened on, and the handle passed to the worker, rather than listening to the worker's idea of what the number 7 file descriptor references.
- `server.listen(handle)` Listening on handles explicitly will cause the worker to use the supplied handle, rather than talk to the master process. If the worker already has the handle, then it's presumed that you know what you are doing.
- `server.listen(0)` Normally, this will cause servers to listen on a random port. However, in a cluster, each worker will receive the same "random" port each time they do listen(0). In essence, the port is random the first time, but predictable thereafter. If you want to listen on a unique port, generate a port number based on the cluster worker ID.

There is no routing logic in Node.js, or in your program, and no shared state between the workers. Therefore, it is important to design your program such that it does not rely too heavily on in-memory data objects for things like sessions and login.

Because workers are all separate processes, they can be killed or re-spawned depending on your program's needs, without affecting other workers. As long as there are some workers still alive, the server will continue to accept connections. If no workers are alive, existing connections will be dropped and new connections will be refused. Node.js does not automatically manage the number of workers for you, however. It is your responsibility to manage the worker pool for your application's needs.

## （及以下未）Class: Worker