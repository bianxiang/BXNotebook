# Java Client API Guide

http://www.rabbitmq.com/api-guide.html

[toc]

客户端API与AMQP协议规范紧密对应，提供进一步抽象以简化使用。

## 协议类概述

持有类`AMQP`保存所有从AMQP XML协议定义自动产生的代码。

	import com.rabbitmq.client.AMQP;

The method request and response descriptors (and the `BasicProperties` class) come with `Builder` classes to make constructing protocol objects easier and to allow us to construct them with immutable state.

We illustrate the builder classes by constructing some `AMQP.BasicProperties` objects with its `Builder` class:

    AMQP.BasicProperties.Builder bob = new AMQP.BasicProperties.Builder();
    AMQP.BasicProperties minBasic = bob.build();
    AMQP.BasicProperties minPersistentBasic = bob.deliveryMode(2).build();
    AMQP.BasicProperties persistentBasic
        = bob.priority(0).contentType("application/octet-stream").build();
    AMQP.BasicProperties persistentTextPlain = bob.contentType("text/plain").build();

`bob` (the builder) is constructed first and whenever `build()` is invoked this method returns a new `BasicProperties` object with the properties set in `bob` at that point. The parameter calls (deliveryMode, priority, etc.) update bob and not the BasicProperties object. Each of these methods returns the updated builder (allowing chaining, for example as in `persistentBasic` above). This pattern allows the parameters to be named and set in any order, the consistency of the complete set of parameters to be checked at `build()` time, the built objects to be immutable, and partially initialised builders to be re-used.

For details and exact definitions of the AMQP protocol, please see [the AMQP specification document](http://www.amqp.org/); for details of the API (including the Builder classes) see the [Javadoc documentation](http://www.rabbitmq.com/releases/rabbitmq-java-client/v3.3.1/rabbitmq-java-client-javadoc-3.3.1/).

## 连接与信道

核心API是类是`Connection`和`Channel`，分别表示一个AMQP连接和信道。

    import com.rabbitmq.client.Connection;
    import com.rabbitmq.client.Channel;

### 连接到broker

    ConnectionFactory factory = new ConnectionFactory();
    factory.setUsername(userName);
    factory.setPassword(password);
    factory.setVirtualHost(virtualHost);
    factory.setHost(hostName);
    factory.setPort(portNumber);
	Connection conn = factory.newConnection();

或者使用 [AMQP URIs](http://www.rabbitmq.com/uri-spec.html)：

    ConnectionFactory factory = new ConnectionFactory();
    factory.setUri("amqp://userName:password@hostName:portNumber/virtualHost");
    Connection conn = factory.newConnection();

获得`Connection`后即可获得**信道**：

	Channel channel = conn.createChannel();

关闭：

    channel.close();
    conn.close();

关闭信道是好的实践，但不是必须的；关闭连接将自动关闭信道。

## 高级连接选项

### 消费者线程池

`Consumer`默认在一个`ExecutorService`线程池中分配。如果需要额外控制，向`newConnection()`传入一个`ExecutorService`。

    ExecutorService es = Executors.newFixedThreadPool(20);
    Connection conn = factory.newConnection(es);

连接关闭时，默认的`ExecutorService`将`shutdown()`。但用户提供的`ExecutorService`不会被`shutdown()`。客户端必须自己确保最终关闭（调用`shutdown()`）。

一个`ExecutorService`可以被多个连接共享，or serially re-used on re-connection。但，一旦`shutdown()`便不能使用。

仅当`Consumer`回调出现严重瓶颈时再考虑使用自定义线程池。如果没有`Consumer`回调或很少，默认的分配就够了。The overhead is initially minimal and the total thread resources allocated are bounded, even if a burst of consumer activity may occasionally occur.

### 地址数组

可以向`newConnection()`传入一个`Address`数组。`Address`用于携带地址和端口，如：

    Address[] addrArr = new Address[]{ new Address(hostname1, portnumber1)
                                     , new Address(hostname2, portnumber2)};
    Connection conn = factory.newConnection(addrArr);

将先尝试第一个地址，不行再尝试第二个。此功能等价于，反复设置工厂的主机和端口，调用`factory.newConnection()`，直到成功。

If an `ExecutorService` is provided as well (using the form `factory.newConnection(es, addrArr)`) the thread pool is associated with the (first) successful connection.

### 自定义线程工厂

Environments such as Google App Engine (GAE) can restrict direct thread instantiation. To use RabbitMQ Java client in such environments, it's necessary to configure a custom `ThreadFactory` that uses GAE's ThreadManager to create threads. Below is an example for Google App Engine.

    import com.google.appengine.api.ThreadManager;

    ConnectionFactory cf = new ConnectionFactory();
    cf.setThreadFactory(ThreadManager.backgroundThreadFactory());

## 交换机和队列

交换机和队列使用前要先声明：

    channel.exchangeDeclare(exchangeName, "direct", true);
    String queueName = channel.queueDeclare().getQueue();
    channel.queueBind(queueName, exchangeName, routingKey);

分别声明了：

- 一个durable、不会被自动删除的交换机，类型是direct。
- 一个不durable、所有的、自动删除的队列，名字由系统产生

由系统分配名字的队列，适合只有一个客户端使用队列的情景（队列可以被多个客户端使用）。如果多个客户端想使用同一队列，可以：

    channel.exchangeDeclare(exchangeName, "direct", true);
    channel.queueDeclare(queueName, true, false, false, null);
    channel.queueBind(queueName, exchangeName, routingKey);

将：

- 一个durable、不会被自动删除的交换机，类型是direct。
- 一个durable、非私有的、不自动删除的、指定名字的队列

## 发布消息

向交换机发布消息，利用`Channel.basicPublish`：

    byte[] messageBodyBytes = "Hello, world!".getBytes();
    channel.basicPublish(exchangeName, routingKey, null, messageBodyBytes);

或更精细的控制：递送模式为2（持久化）、优先级0、内容类型"text/plain"。

    channel.basicPublish(exchangeName, routingKey, mandatory,
                         MessageProperties.PERSISTENT_TEXT_PLAIN,
                         messageBodyBytes);

可以利用`Builder`构造其他种类的消息属性对象：

    channel.basicPublish(exchangeName, routingKey,
                         new AMQP.BasicProperties.Builder()
                           .contentType("text/plain").deliveryMode(2)
                           .priority(1).userId("bob")
                           .build()),
                         messageBodyBytes);

## 信道线程安全

`Channel`实例是线程安全的。对`Channel`的请求是串行的，一次只有一个线程能够在信道上运行命令。即使如此，应用仍应该每个线程使用一个单独的`Channel`实例。

## 通过订阅接收消息

    import com.rabbitmq.client.Consumer;
    import com.rabbitmq.client.DefaultConsumer;

接收消息最有效的方式是利用`Consumer`接口订阅。

When calling the API methods relating to `Consumer`s, individual subscriptions are always referred to by their **consumer tags**, which can be either client- or server-generated as explained in the [AMQP specification document](http://www.amqp.org/). 同一信道上的不同`Consumer`必须具有不同的consumer tags。

实现`Consumer`最简单的方式是继承`DefaultConsumer`。然后将`DefaultConsumer`对象传给`basicConsume`实现订阅：

    boolean autoAck = false;
    channel.basicConsume(queueName, autoAck, "myConsumerTag",
         new DefaultConsumer(channel) {
             @Override
             public void handleDelivery(String consumerTag, Envelope envelope,
             	AMQP.BasicProperties properties, byte[] body)
                throws IOException {
                 String routingKey = envelope.getRoutingKey();
                 String contentType = properties.contentType;
                 long deliveryTag = envelope.getDeliveryTag();
                 // (process the message components here ...)
                 channel.basicAck(deliveryTag, false);
             }
         });

因为设置了`autoAck = false`，必须ACK消息，在`handleDelivery`中做是比较合适的。

更复杂的`Consumer`可能需要覆盖更多方法。例如，当信道和连接关闭时，会调用`handleShutdownSignal`。在回调发生前，会先调用`handleConsumeOk`，传入consumer tag。

消费者还可以实现`handleCancelOk`和`handleCancel`方法，得到显式、隐式取消的通知。

可以通过`Channel.basicCancel`显示的取消一个消费者。

	channel.basicCancel(consumerTag);

Callbacks to `Consumer`s are dispatched on a thread separate from the thread managed by the `Connection`. This means that Consumers can safely call blocking methods on the `Connection` or `Channel`, such as `queueDeclare`, `txCommit`, `basicCancel` or `basicPublish`.

Each `Channel` has its own dispatch thread. 多数情况下一个信道只有一个`Consumer`。但如果你的信道有多个`Consumer`，要意识到，长时间运行的`Consumer`可能阻塞同一信道上的其他`Consumer`的回调。

## 查询单条消息

使用`Channel.basicGet`可以显式的获取消息。返回值是`GetResponse`的一个实例，从中可以获取头（属性）和Body：

    boolean autoAck = false;
    GetResponse response = channel.basicGet(queueName, autoAck);
    if (response == null) {
        // No message retrieved.
    } else {
        AMQP.BasicProperties props = response.getProps();
        byte[] body = response.getBody();
        long deliveryTag = response.getEnvelope().getDeliveryTag();

        channel.basicAck(method.deliveryTag, false); // acknowledge receipt of the message
    }

因为使用了`autoAck = false`，必须调用`Channel.basicAck`来ACK。

## 处理不可路由的消息

如果发布消息时设置了"mandatory"标志，但消息不能被路由。则broker会把消息退回发送的客户端（通过`AMQP.Basic.Return`命令）。

如果想得到这种通知，客户端可以实现`ReturnListener`接口，并调用`Channel.setReturnListener`。如果客户端不监听，则返回的消息会被悄悄忽略。

    channel.setReturnListener(new ReturnListener() {
        public void handleBasicReturn(int replyCode,
                                      String replyText,
                                      String exchange,
                                      String routingKey,
                                      AMQP.BasicProperties properties,
                                      byte[] body)
        throws IOException {
            ...
        }
    });

A return listener will be called, for example, if the client publishes a message with the "mandatory" flag set to an exchange of "direct" type which is not bound to a queue.

## 基本的RPC

Java客户端提供一个`RpcClient`，利用一个临时回复队列，提供简单的RPC风格的通讯。

The class doesn’t impose any particular format on the RPC arguments and return values. It simply provides a mechanism for sending a message to a given exchange with a particular routing key, and waiting for a response on a reply queue.

	RpcClient rpc = new RpcClient(channel, exchangeName, routingKey);

> 这个类的实现细节是（如何利用AMQP）：请求消息携带一个`basic.correlation_id`字段，这个值对这个`RpcClient`来说是唯一的；还携带一个`basic.reply_to`字段，设置回复队列的名字）。

进行RPC请求：

    byte[] primitiveCall(byte[] message);
    String stringCall(String message)
    Map mapCall(Map message)
    Map mapCall(Object[] keyValuePairs)

`primitiveCall`的请求和响应都是字节。`stringCall`是对`primitiveCall`的封装，treating the message bodies as String instances in the default character encoding。

The `mapCall` variants are a little more sophisticated: they encode a `java.util.Map` containing ordinary Java values into an AMQP binary table representation, and decode the response in the same way. (Note that there are some restrictions on what value types can be used here - see the javadoc for details.)

All the marshalling/unmarshalling convenience methods use `primitiveCall` as a transport mechanism, and just provide a wrapping layer on top of it.

## 关闭协议

### AMQP客户端关闭概述

AMQP的连接和信道处理网络错误、内部错误或客户端显式关闭的机制相同。

AMQP连接和信道具有以下生命周期状态：

- 打开：对象可用
- 关闭中：the object has been explicitly notified to shut down locally, has issued a shutdown request to any supporting lower-layer objects, and is waiting for their shutdown procedures to complete
- 已关闭：the object has received all shutdown-complete notification(s) from any lower-layer objects, and as a consequence has shut itself down

这些对象的终态总是关闭状态，不管原因是什么。

连接和信道处理关闭的相关方法：

- `addShutdownListener(ShutdownListener listener)`和`removeShutdownListener(ShutdownListener listener)`：监听对象进入关闭状态。如果向一个已关闭的对象添加`ShutdownListener`，则监听器立即触发。
- `getCloseReason()`：获取对象关闭原因
- `isOpen()`：检查状态
- `close(int closeCode, String closeMessage)`：显式要求对象关闭。

例子：

    connection.addShutdownListener(new ShutdownListener() {
        public void shutdownCompleted(ShutdownSignalException cause) {
            ...
        }
    });

### 关闭时的信息

One can retrieve the `ShutdownSignalException`, which contains all the information available about the close reason, either by explictly calling the `getCloseReason()` method or by using the `cause` parameter in the `service(ShutdownSignalException cause)` method of the `ShutdownListener` class.

The `ShutdownSignalException` class provides methods to analyze the reason of the shutdown. By calling the `isHardError()` method we get information whether it was a connection or a channel error, and `getReason()` returns information about the cause, in the form an AMQP method - either `AMQP.Channel.Close` or `AMQP.Connection.Close` (or null if the cause was some exception in the library, such as a network communication failure, in which case that exception can be retrieved with `getCause()`).

    public void shutdownCompleted(ShutdownSignalException cause) {
      if (cause.isHardError()) {
        Connection conn = (Connection)cause.getReference();
        if (!cause.isInitiatedByApplication())  {
          Method reason = cause.getReason();
          ...
        }
        ...
      } else {
        Channel ch = (Channel) cause.getReference();
        ...
      }
    }

### 原子性与`isOpen()`方法

不推荐在产品中使用信道或连接的`isOpen()`方法，因为该方法的返回值取决于关闭的原因。下面的代码演示了可能的竞争条件：

    public void brokenMethod(Channel channel) {
        if (channel.isOpen()) {
            // 在isOpen()和basicQos(1)之间状态可能已经发生改变
            ...
            channel.basicQos(1);
        }
    }

我们应该放弃使用这种检查。直接尝试操作。如果在执行过程中信道或连接被关闭，会抛出`ShutdownSignalException`。We should also catch for `IOException` caused either by `SocketException`, when broker closes the connection unexpectedly, or `ShutdownSignalException`, when broker initiated clean close.｛｛Broker正常关闭与异常关闭抛出的异常不同。｝｝

    public void validMethod(Channel channel) {
        try {
            ...
            channel.basicQos(1);
        } catch (ShutdownSignalException sse) {
            // possibly check if channel was closed
            // by the time we started action and reasons for
            // closing it
            ...
        } catch (IOException ioe) {
            // check why connection was closed
            ...
        }
    }

## 自动从网络错误中恢复

### 连接恢复

Network connection between clients and RabbitMQ nodes can fail. RabbitMQ Java client supports automatic recovery of connections and topology (queues, exchanges, bindings, and consumers). 多数应用的自动恢复包含以下步骤：

- 重连
- 恢复连接监听器
- 重新打开信道
- 重新打开信道监听器
- 恢复信道`basic.qos`设置，恢复publisher confirms and transaction settings

Topology recovery includes the following actions, performed for every channel

- 重新声明交换机（除了预定义的）
- 重新声明队列
- 恢复所有绑定
- 恢复所有消费者

要启用自动连接恢复，使用`factory.setAutomaticRecoveryEnabled(true)`：

    ConnectionFactory factory = new ConnectionFactory();
    factory.setUsername(userName);
    factory.setPassword(password);
    factory.setVirtualHost(virtualHost);
    factory.setHost(hostName);
    factory.setPort(portNumber);
    factory.setAutomaticRecoveryEnabled(true);
    // connection that will recover automatically
    Connection conn = factory.newConnection();

如果因异常不会恢复（如RabbitMQ节点仍不可用），会在固定周期后再次尝试（默认5秒）。周期可配置：

    ConnnectionFactory factory = new ConnectionFactory();
    // attempt recovery every 10 seconds
    factory.setNetworkRecoveryInterval(10000);

When a list of addresses is provided, a random one will be picked during recovery:

    ConnectionFactory factory = new ConnectionFactory();
    Address[] addresses = {new Address("192.168.1.4"), new Address("192.168.1.5")};
    factory.newConnection(addresses);

### 对发布的影响

Messages that are published using `Channel.basicPublish` when connection is down will be lost. The client does not enqueue them for delivery after connection has recovered. To ensure that published messages reach RabbitMQ applications need to use Publisher Confirms and account for connection failures.

### Topology Recovery

Topology recovery involves recovery of exchanges, queues, bindings and consumers. It is enabled by default but can be disabled:

```java
ConnectionFactory factory = new ConnectionFactory();

Connection conn = factory.newConnection();
factory.setAutomaticRecoveryEnabled(true);
factory.setTopologyRecoveryEnabled(false);
```

### Manual Acknowledgements and Automatic Recovery

When manual acknowledgements are used, it is possible that network connection to RabbitMQ node fails between message delivery and acknowledgement. After connection recovery, RabbitMQ will reset delivery tags on all channels. This means that `basic.ack`, `basic.nack`, and `basic.reject` with old delivery tags will cause a channel exception. To avoid this, RabbitMQ Java client keeps track of and updates delivery tags to make them monotonically growing between recoveries. `Channel.basicAck`, `Channel.basicNack`, and `Channel.basicReject` then translate adjusted delivery tags into those used by RabbitMQ. Acknowledgements with stale delivery tags will not be sent. Applications that use manual acknowledgements and automatic recovery must be capable of handling redeliveries.

## RabbitMQ Java Client on Google App Engine

Using RabbitMQ Java client on Google App Engine (GAE) requires using a custom thread factory that instantiates thread using GAE's ThreadManager (see above). In addition, it is necessary to set a **low** heartbeat interval (4-5 seconds) to avoid running into the low InputStream read timeouts on GAE:

```java
ConnectionFactory factory = new ConnectionFactory();
cf.setRequestedHeartbeat(5);
```