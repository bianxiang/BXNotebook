http://www.rabbitmq.com/getstarted.html

node版本：https://github.com/squaremo/amqp.node/tree/master/examples/tutorials


[toc]

## 1 Hello World

队列可以连接多个生产者和消费者。

生产者的代码：

```java
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;

public class Send {

  private final static String QUEUE_NAME = "hello";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.queueDeclare(QUEUE_NAME, false, false, false, null);
    String message = "Hello World!";
    channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
    System.out.println(" [x] Sent '" + message + "'");

    channel.close();
    connection.close();
  }
}
```

声明队列是幂等的——仅当不存在时才会创建。

消费者代码：

```java
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.QueueingConsumer;

public class Recv {

    private final static String QUEUE_NAME = "hello";

    public static void main(String[] argv) throws Exception {

    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.queueDeclare(QUEUE_NAME, false, false, false, null);
    System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

    QueueingConsumer consumer = new QueueingConsumer(channel);
    channel.basicConsume(QUEUE_NAME, true, consumer);

    while (true) {
      QueueingConsumer.Delivery delivery = consumer.nextDelivery();
      String message = new String(delivery.getBody());
      System.out.println(" [x] Received '" + message + "'");
    }
  }
}
```

`QueueingConsumer.nextDelivery()`阻塞知道收到下一条消息。

编译：
```
$ javac -cp rabbitmq-client.jar Send.java Recv.java
```

运行：
```
$ java -cp .:commons-io-1.2.jar:commons-cli-1.1.jar:rabbitmq-client.jar Send
$ java -cp .:commons-io-1.2.jar:commons-cli-1.1.jar:rabbitmq-client.jar Recv
```

## 2 工作队列

将任务封装成消息，发送到队列。

### 消息ACK

在当前模式，RabbitMQ递送消息给消费者后，立即从内存中移除。如果杀死消费者，则消息丢失。

为确保不丢消息，RabbitMQ支持消息ACK。RabbitMQ在收到ACK后删除消息。

连接死掉被认为消息未被处理。消息不会超时，RabbitMQ会一直等待ACK。只有在连接死掉后，RabbitMQ才会重新递送。

消息ACK默认是关闭的。开启：
```java
QueueingConsumer consumer = new QueueingConsumer(channel);
boolean autoAck = false;
channel.basicConsume("hello", autoAck, consumer);

while (true) {
  QueueingConsumer.Delivery delivery = consumer.nextDelivery();
  //...
  channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
}
```

忘记ACK后果是严重的。消息会被重新递送。还可能导致RabbitMQ内存不断上升。

要调试这类错误，可以用`rabbitmqctl`打印`messages_unacknowledged`字段：
```
$ sudo rabbitmqctl list_queues name messages_ready messages_unacknowledged
Listing queues ...
hello    0       0
...done.
```

### 消息durability

令队列duable：

```java
boolean durable = true;
channel.queueDeclare("hello", durable, false, false, null);
```

上面的代码实际会报错。因为已经定义了一个名叫`hello`的队列且它不是durable的。RabbitMQ不允许用不同的参数创建已存在的队列。因此这里需要一个新队列：`task_queue`：
```java
boolean durable = true;
channel.queueDeclare("task_queue", durable, false, false, null);
```

接下来令消息持久化
```java
channel.basicPublish("", "task_queue", MessageProperties.PERSISTENT_TEXT_PLAIN,
	message.getBytes());
```

> Note on message persistence
 Marking messages as persistent doesn't fully guarantee that a message won't be lost. Although it tells RabbitMQ to save the message to disk, there is still a short time window when RabbitMQ has accepted a message and hasn't saved it yet. Also, RabbitMQ doesn't do `fsync(2)` for every message -- it may be just saved to cache and not really written to the disk. The persistence guarantees aren't strong, but it's more than enough for our simple task queue. If you need a stronger guarantee you can wrap the publishing code in a transaction.

### Fair dispatch

一个场景：有两个工作者。奇数号消息比偶数号消息总是需要更多处理时间。于是第一个工作者总是很忙，第二个总是很闲。RabbitMQ对此并不知晓，仍然依次递送消息。

为解决该问题，可以调用`basicQos`方法，设置`prefetchCount = 1`。这使得RabbitMQ一次最多个一个消息，或者说，直到消费者ACK一个消息后再递送下一个。消息会递送给下一个不忙的消费者。

```java
int prefetchCount = 1;
channel.basicQos(prefetchCount);
```

### 最终代码

```java
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.MessageProperties;

public class NewTask {
  private static final String TASK_QUEUE_NAME = "task_queue";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();
    channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);
    String message = getMessage(argv);
    channel.basicPublish("", TASK_QUEUE_NAME,
		MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
    System.out.println(" [x] Sent '" + message + "'");

    channel.close();
    connection.close();
  }

  private static String getMessage(String[] strings) {
    if (strings.length < 1)
      return "Hello World!";
    return joinStrings(strings, " ");
  }

  private static String joinStrings(String[] strings, String delimiter) {
    int length = strings.length;
    if (length == 0) return "";
    StringBuilder words = new StringBuilder(strings[0]);
    for (int i = 1; i < length; i++) {
      words.append(delimiter).append(strings[i]);
    }
    return words.toString();
  }
}
```

```java
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.QueueingConsumer;

public class Worker {
  private static final String TASK_QUEUE_NAME = "task_queue";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();
    channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);
    System.out.println(" [*] Waiting for messages. To exit press CTRL+C");
    channel.basicQos(1);
    QueueingConsumer consumer = new QueueingConsumer(channel);
    channel.basicConsume(TASK_QUEUE_NAME, false, consumer);
    while (true) {
      QueueingConsumer.Delivery delivery = consumer.nextDelivery();
      String message = new String(delivery.getBody());
      System.out.println(" [x] Received '" + message + "'");
      doWork(message);
      System.out.println(" [x] Done");

      channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
    }
  }

  private static void doWork(String task) throws InterruptedException {
    for (char ch: task.toCharArray()) {
      if (ch == '.') Thread.sleep(1000);
    }
  }
}
```

## 3 发布/订阅

之前的教程中，工作队列隐含着，一个消息只会被递送给一个工作者。本节则完全不同——消息会被递送给多个消费者。这种模式称为“发布/订阅”。

To illustrate the pattern, we're going to build a simple logging system. In our logging system every running copy of the receiver program will get the messages.

### Exchanges

生产者从来不直接向队列发消息。生产者根本不知晓消息是否被递送到了队列。生产者只能将消息发给一个exchange。交换机一边从生产者接收消息，一边将消息送给队列。送给哪个队列由交换机类型决定。

交换器类型有：direct, topic, headers, fanout。这里创建最后一种，并将其命名为`logs`：
```java
channel.exchangeDeclare("logs", "fanout");
```

fanout将所有消息播发给所有队列。

列出服务器上的exchanges：

    $ sudo rabbitmqctl list_exchanges
    Listing exchanges ...
            direct
    amq.direct      direct
    amq.fanout      fanout
    amq.headers     headers
    amq.match       headers
    amq.rabbitmq.log        topic
    amq.rabbitmq.trace      topic
    amq.topic       topic
    logs    fanout
    ...done.

`amq.*`开头的exchanges和默认exchange（无名，第一个）是默认创建的。

> 无名exchange
之前的例子中，没有显式使用Exchange，但仍能将消息送到队列。That was possible because we were using a default exchange, which we identify by the empty string ("").
Recall how we published a message before:
```java
channel.basicPublish("", "hello", null, message.getBytes());
```
第一个参数是exchange名。空串即表示默认或无名exchange： messages are routed to the queue with the name specified by routing Key, if it exists.

现在发布消息到一个命名exchange：
```java
channel.basicPublish( "logs", "", null, message.getBytes());
```

### 临时队列

Giving a queue a name is important when you want to share the queue between producers and consumers.

But that's not the case for our logger. 我们想要收到所有日志消息，而不是部分。我们只关心当前喜爱消息，不关心历史的。为此，首先，每次我们连接到RabbitMQ时，都需要一个新的、空的队列。让服务器为我们挑一个随机的名字。第二，消费者断开连接后，队列被自动删除。

In the Java client, when we supply no parameters to `queueDeclare()` we create a non-durable, exclusive, autodelete queue with a generated name:
```java
String queueName = channel.queueDeclare().getQueue();
```

At that point queueName contains a random queue name. For example it may look like `amq.gen-JzTY20BRgKO-HjmUJj0wLg`.

### 绑定

exchange和队列的关系叫做绑定。
```java
channel.queueBind(queueName, "logs", "");
```

> 列出绑定
```java
rabbitmqctl list_bindings.
```

### 最终代码

对于发布者，最重要的改变是，我们想要发布消息到logs exchange。We need to supply a **routingKey** when sending, but its value is ignored for fanout exchanges. Here goes the code for EmitLog.java program:

```java
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;

public class EmitLog {

  private static final String EXCHANGE_NAME = "logs";

  public static void main(String[] argv) throws Exception {

    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

    String message = getMessage(argv);

    channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes());
    System.out.println(" [x] Sent '" + message + "'");

    channel.close();
    connection.close();
  }

  private static String getMessage(String[] strings){
    if (strings.length < 1)
    	    return "info: Hello World!";
    return joinStrings(strings, " ");
  }

  private static String joinStrings(String[] strings, String delimiter) {
    int length = strings.length;
    if (length == 0) return "";
    StringBuilder words = new StringBuilder(strings[0]);
    for (int i = 1; i < length; i++) {
        words.append(delimiter).append(strings[i]);
    }
    return words.toString();
  }
}
```
```java
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.QueueingConsumer;

public class ReceiveLogs {

  private static final String EXCHANGE_NAME = "logs";

  public static void main(String[] argv) throws Exception {

    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
    String queueName = channel.queueDeclare().getQueue();
    channel.queueBind(queueName, EXCHANGE_NAME, "");

    System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

    QueueingConsumer consumer = new QueueingConsumer(channel);
    channel.basicConsume(queueName, true, consumer);

    while (true) {
      QueueingConsumer.Delivery delivery = consumer.nextDelivery();
      String message = new String(delivery.getBody());

      System.out.println(" [x] Received '" + message + "'");
    }
  }
}
```

## 4 路由

本节，实现订阅部分消息。例如，值将错误日志记录在文件，但所有级别的日志仍打印在控制台。

### 绑定

绑定，形如：
```java
channel.queueBind(queueName, EXCHANGE_NAME, "");
```

绑定是exchange与队列之间的关系。
绑定有一个参数`routingKey`.To avoid the confusion with a` basicpublish` parameter we're going to call it a binding key. This is how we could create a binding with a key:
```java
channel.queueBind(queueName, EXCHANGE_NAME, "black");
```

binding key的含义取决于exchange类型。例如对于fanout，会忽略binding key。

### Direct exchange

我们想让记录日志到磁盘的程序只接收错误日志。使用direct exchange，它的路由算法是，消息进入哪个队列，取决于消息的routing key与队列的binding key是否匹配。

例如，下面三个队列的binding key分别是orange, black, green。则routing key为orange的消息到达Q1。

![](direct-exchange.png)

### 多绑定

多个队列可以具有相同的binding key。则匹配的routing key的消息会到达所有这些队列。


### 收发消息

我们打算将日志级别作为routing key。首先，创建一个Exchange：

```java
channel.exchangeDeclare(EXCHANGE_NAME, "direct");
```

发消息：

```java
channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes());
```

To simplify things we will assume that 'severity' can be one of 'info', 'warning', 'error'.

下面，为每一种日志级别创建一个绑定：

```java
String queueName = channel.queueDeclare().getQueue();

for(String severity : argv) {
  channel.queueBind(queueName, EXCHANGE_NAME, severity);
}
```

### 最终代码

```java
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;

public class EmitLogDirect {

  private static final String EXCHANGE_NAME = "direct_logs";

  public static void main(String[] argv) throws Exception {

    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.exchangeDeclare(EXCHANGE_NAME, "direct");

    String severity = getSeverity(argv);
    String message = getMessage(argv);

    channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes());
    System.out.println(" [x] Sent '" + severity + "':'" + message + "'");

    channel.close();
    connection.close();
  }

  private static String getSeverity(String[] strings) {
    if (strings.length < 1)
    	    return "info";
    return strings[0];
  }

  private static String getMessage(String[] strings) {
    if (strings.length < 2)
    	    return "Hello World!";
    return joinStrings(strings, " ", 1);
  }

  private static String joinStrings(String[] strings, String delimiter, int startIndex) {
    int length = strings.length;
    if (length == 0 ) return "";
    if (length < startIndex ) return "";
    StringBuilder words = new StringBuilder(strings[startIndex]);
    for (int i = startIndex + 1; i < length; i++) {
        words.append(delimiter).append(strings[i]);
    }
    return words.toString();
  }
}

```

```java
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.QueueingConsumer;

public class ReceiveLogsDirect {
  private static final String EXCHANGE_NAME = "direct_logs";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.exchangeDeclare(EXCHANGE_NAME, "direct");
    String queueName = channel.queueDeclare().getQueue();

    if (argv.length < 1){
      System.err.println("Usage: ReceiveLogsDirect [info] [warning] [error]");
      System.exit(1);
    }

    for(String severity : argv) {
      channel.queueBind(queueName, EXCHANGE_NAME, severity);
    }

    System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

    QueueingConsumer consumer = new QueueingConsumer(channel);
    channel.basicConsume(queueName, true, consumer);

    while (true) {
      QueueingConsumer.Delivery delivery = consumer.nextDelivery();
      String message = new String(delivery.getBody());
      String routingKey = delivery.getEnvelope().getRoutingKey();

      System.out.println(" [x] Received '" + routingKey + "':'" + message + "'");
    }
  }
}
```

## 5 Topics

我们需要根据多个条件路由。上一节的direct只能根据一个条件（日志级别）。We may want to listen to just critical errors coming from 'cron' but also all logs from 'kern'.

### Topic exchange

发送到topic exchange的消息的`routing_key`必须是以点分隔的多个单词。单词可以任意。A few valid routing key examples: "stock.usd.nyse", "nyse.vmw", "quick.orange.rabbit". There can be as many words in the routing key as you like, **up to the limit of 255 bytes**.

binding key也必须是这种形式。binding keys有两个特殊形式：

- `*` 可以替换一个单词can substitute for exactly one word.
- `#` 可以替换零到多个单词

![](python-five.png)

我们将发送描述动物的消息。消息的routing key包括三个词。第一个是速度，第二个是颜色，第三个是物种："`<speed>.<colour>.<species>`"。

创建三个绑定：Q1 is bound with binding key "`*.orange.*`" and Q2 with "`*.*.rabbit`" and "`lazy.#`".

"quick.orange.rabbit"会被递送到两个队列。"quick.orange.fox"只会被递送到第一个队列。"lazy.pink.rabbit"只会递送到第二个队列一次，尽管它匹配两个绑定。"quick.brown.fox"不匹配任何绑定，因此会被丢弃。

如果消息不带三个单词，如"orange"或"quick.orange.male.rabbit"，消息会被丢弃。但"lazy.orange.male.rabbit"匹配最后一个绑定，因此会被递送到第二个队列。


如果队列的binding key是"#"，则它将接收所有消息，就像fanout exchange一样。
如果不使用任何通配符（"*"、"#"），则topic exchange就像一个direct exchange。

### 最终代码

We'll start off with a working assumption that the routing keys of logs will have two words: "`<facility>.<severity>`".

```java
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;

public class EmitLogTopic {

  private static final String EXCHANGE_NAME = "topic_logs";

  public static void main(String[] argv) {
    Connection connection = null;
    Channel channel = null;
    try {
      ConnectionFactory factory = new ConnectionFactory();
      factory.setHost("localhost");
  
      connection = factory.newConnection();
      channel = connection.createChannel();

      channel.exchangeDeclare(EXCHANGE_NAME, "topic");

      String routingKey = getRouting(argv);
      String message = getMessage(argv);

      channel.basicPublish(EXCHANGE_NAME, routingKey, null, message.getBytes());
      System.out.println(" [x] Sent '" + routingKey + "':'" + message + "'");

    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      if (connection != null) {
        try {
          connection.close();
        } catch (Exception ignore) {}
      }
    }
  }
  
  private static String getRouting(String[] strings){
    if (strings.length < 1)
		return "anonymous.info";
    return strings[0];
  }

  private static String getMessage(String[] strings){ 
    if (strings.length < 2)
		return "Hello World!";
    return joinStrings(strings, " ", 1);
  }
  
  private static String joinStrings(String[] strings, String delimiter, int startIndex) {
    int length = strings.length;
    if (length == 0 ) return "";
    if (length < startIndex ) return "";
    StringBuilder words = new StringBuilder(strings[startIndex]);
    for (int i = startIndex + 1; i < length; i++) {
        words.append(delimiter).append(strings[i]);
    }
    return words.toString();
  }
}
```

```java
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.QueueingConsumer;

public class ReceiveLogsTopic {

  private static final String EXCHANGE_NAME = "topic_logs";

  public static void main(String[] argv) {
    Connection connection = null;
    Channel channel = null;
    try {
      ConnectionFactory factory = new ConnectionFactory();
      factory.setHost("localhost");
  
      connection = factory.newConnection();
      channel = connection.createChannel();

      channel.exchangeDeclare(EXCHANGE_NAME, "topic");
      String queueName = channel.queueDeclare().getQueue();
 
      if (argv.length < 1){
        System.err.println("Usage: ReceiveLogsTopic [binding_key]...");
        System.exit(1);
      }
    
      for(String bindingKey : argv){    
        channel.queueBind(queueName, EXCHANGE_NAME, bindingKey);
      }
    
      System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

      QueueingConsumer consumer = new QueueingConsumer(channel);
      channel.basicConsume(queueName, true, consumer);

      while (true) {
        QueueingConsumer.Delivery delivery = consumer.nextDelivery();
        String message = new String(delivery.getBody());
        String routingKey = delivery.getEnvelope().getRoutingKey();

        System.out.println(" [x] Received '" + routingKey + "':'" + message + "'");   
      }
    } catch  (Exception e) {
      e.printStackTrace();
    } finally {
      if (connection != null) {
        try {
          connection.close();
        } catch (Exception ignore) {}
      }
    }
  }
}
```

## 6 RPC

But what if we need to run a function on a remote computer and wait for the result? Well, that's a different story. This pattern is commonly known as Remote Procedure Call or RPC.

In this tutorial we're going to use RabbitMQ to build an RPC system: a client and a scalable RPC server.

### 客户端接口

暴露一个方法`call`，发出RPC请求，阻塞，直到收到结果：
```java
FibonacciRpcClient fibonacciRpc = new FibonacciRpcClient();
String result = fibonacciRpc.call("4");
System.out.println( "fib(4) is " + result);
```

### 回调队列

In general doing RPC over RabbitMQ is easy. A client sends a request message and a server replies with a response message. 需要在请求中指定回调队列的地址。我们可以使用默认队列（Java客户端独有的）：
```java
callbackQueueName = channel.queueDeclare().getQueue();
BasicProperties props = new BasicProperties
                            .Builder()
                            .replyTo(callbackQueueName)
                            .build();
channel.basicPublish("", "rpc_queue", props, message.getBytes());
// ... then code to read a response message from the callback_queue ...
```

### 消息属性

AMQP协议预定义了消息可以携带的14个属性。多数属性很少会被用到，除了：
- `deliveryMode`：标记消息为persistent（传2）或transient（其他任何值）。
- `contentType`: 例如对于JSON数据，可以传`application/json`。
- `replyTo`：用于命名回调队列。
- `correlationId`：Useful to correlate RPC responses with requests.

### Correlation Id

之前的方法，每个请求需要一个独立的回调队列，这样何不高效。更好的方式是，一个客户端只用一个回调。

使用一个队列，需要给请求唯一标识（通过`correlationId`），在回调队列收到响应时，根据这个标识判断响应的是哪个请求。如果遇到未知`correlationId`值，可以抛弃此消息。

You may ask, why should we ignore unknown messages in the callback queue, rather than failing with an error? It's due to a possibility of a race condition on the server side. Although unlikely, it is possible that the RPC server will die just after sending us the answer, but before sending an acknowledgment message for the request. If that happens, the restarted RPC server will process the request again. That's why on the client we must handle the duplicate responses gracefully, and the RPC should ideally be idempotent.

### 总结

- When the Client starts up, it creates an anonymous exclusive callback queue.
- For an RPC request, the Client sends a message with two properties: `replyTo`, which is set to the callback queue and `correlationId`, which is set to a unique value for every request.
- The request is sent to an `rpc_queue` queue.
- The RPC worker (aka: server) is waiting for requests on that queue. When a request appears, it does the job and sends a message with the result back to the Client, using the queue from the `replyTo` field.
- The client waits for data on the callback queue. When a message appears, it checks the `correlationId` property. If it matches the value from the request it returns the response to the application.

### 完整代码

[RPCClient.java](https://github.com/rabbitmq/rabbitmq-tutorials/blob/master/java/RPCClient.java)

[RPCServer.java](https://github.com/rabbitmq/rabbitmq-tutorials/blob/master/java/RPCServer.java)