[toc]

## 4. 鲁棒的消息服务

为创建我们的消息服务，我们使用跨平台库 ØMQ (pronounced “Zero-M-Q”)。ØMQ provides high-scalability and low-latency messaging.

### 4.1 ØMQ 的优点

ØMQ的目标是暴露高层的消息模式，以in从网络底层细节：

- ØMQ的 endpoints 能自动重连
- ØMQ delivers only whole messages, so you don’t have to create buffers to deal with chunked data.
- ØMQ’s  low-overhead  protocol  takes  care  of  many  routing  details,  like sending responses back to the correct clients.

### 4.2 使用 npm 引入外部模块

#### Installing the ØMQ Base Library

Installing the ØMQ library is fairly straightforward on most platforms. Binary and source packages are available from the project download page.(http://www.zeromq.org/intro:get-the-software)

To test whether ØMQ was installed successfully, you can try to load its man page with `man zmq`.

#### Installing the zmqNode Module

Once you’ve installed the base ØMQ library, you can use npm to pull down the `zmq` Node module.

Create a directory called `messaging` and navigate to this directory on the command line. Then install `zmq`:

	$npm install zmq

Notice the call to `node-gyp` about halfway though. `node-gyp` is a cross-platform tool for compiling native addons.

To  test  that  the module was installed successfully, run this command:

	$node --harmony -p -e 'require("zmq")'

The -e flag tells Node to evaluate the provided string, and the -p flag tells it to print that output to the terminal.

### 4.3 消息发布和订阅

ØMQ  supports  a  number  of  different  message-passing  patterns  that  work
great in Node. We’ll start with the publish/subscribe pattern (PUB/SUB).
Recall the code we wrote in Chapter 3, Networking with Sockets, on page 23,
when we developed a networked file-watching service and a client to connect
to  it.  They  communicated  over  TCP  by  sending  Line-Delimited  JavaScript
Object Notation (JSON) messages. The server would publishinformation in
this format, and any number of client programs could subscribeto it.
We  had  to  work  hard  to  make  our  client  code  safely  handle  the  messageboundary  problem.  We  created  a  separate  module  dedicated  to  buffering
chunked data and emitting messages. Even so, we were left with questions
like how to handle network interrupts or server restarts.

