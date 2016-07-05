[toc]

## 启动、停止

### 启动

一个RabbitMQ服务器实例常称为一个节点。实际上，一个节点实际指的是一个Erlang阶段。Erlang有一个虚拟机，每个实例称为一个节点。与JVM不同的是，一个节点内可以跑多个应用，而且节点间可以值金额通信（不管它们在不在同一个机器上）。若一个应用因故崩溃，Erlang节点将自动尝试重启应用（只要Erlang自身没有崩溃）。

通过一条命令可以同时启动Erlang节点和Rabbit应用：`./rabbitmq-server`。 若想将Rabbit节点作为后台守护，添加`-detached`选项：`./rabbitmq -server -detached`。

### 停止


两种停止的方式：干净的和不干净的。如果你是在控制台运行的RabbitMQ，可以按CTRL-C，然后会看到：

	BREAK:(a)bort (c)ontinue (p)rocinfo (i)nfo (l)oaded
	(v)ersion (k)ill (D)b-tables(d)istribution

这条信息是Erlang打印的。一般选择关闭整个节点，即*abort*。

但更好的方式是，告诉RabbitMQ干净的关闭，保护所有持久化的队列。利用`rabbitmqctl`关闭RabbitMQ：`./sbin/rabbitmqctl stop`。或者通过`-n rabbit@[hostname]`关闭远程节点。在RabbitMQ日志中会记录：

	=INFO REPORT====
		application: rabbit
		exited: stopped
		type: permanent
	=INFO REPORT====
		application: mnesia
		exited: stopped
		type: permanent
	=INFO REPORT====
		application: os_mon
		exited: stopped
		type: permanent

When you see that *rabbit*, *mnesia*, and *os_mon* are stopped, the Rabbit node is completely shut down.

## 管理虚拟机

- 创建虚拟机需要运行：`rabbitmqctl add_vhost [vhost_name]`。
- 删除虚拟机：`rabbitmqctl delete_vhost [vhost_name]`。
- 列出服务器上的虚拟机：`rabbitmqctl list_vhosts`。

## 用户与权限

一个用户被授予的权限可以跨多个虚拟机。

### 管理用户

通过`rabbitmqctl`管理用户。

添加用户，用户名为`cashing-tier`，密码为`cashMe1`：

    $ rabbitmqctl add_user cashing-tier cashMe1
    Creating user "cashing-tier" ...
    ...done.

删除用户`cashing-tier`：

    $ rabbitmqctl delete_user cashing-tier
    Deleting user "cashing-tier" ...
    ...done.

列出用户：

    $ rabbitmqctl list_users
    Listing users ...
    cashing-tier
    guest
    ...done.

改密码：

    $ rabbitmqctl change_password cashing-tier compl3xPassword
    Changing password for user "cashing-tier"...
    ...done.

### Rabbit的权限系统

三个权限：

- Read：消费消息的所有操作，包括清空整个队列（需要先成功绑定）。
- Write：发布消息（需要先成功绑定）。
- Configure：创建或删除队列和Exchange

Table 3.3 AMQP操作与RabbitMQ权限的映射

|AMQP命令         |配置     |写      |读      |
|----------------|--------|--------|--------|
|exchange.declare|exchange|        |        |
|exchange.delete |exchange|        |        |
|queue.declare   |queue   |        |        |
|queue.delete    |queue   |        |        |
|queue.bind      |        |queue   |exchange|
|basic.publish   |        |exchange|        |
|basic.get       |        |        |queue   |
|basic.consume   |        |        |queue   |
|queue.purge     |        |        |queue   |

访问控制项包含四部分：

- 用户是谁
- 在哪个虚拟机
- 分配何种权限：read/write/configure
- 权限范围：whether the permissions apply only to client-named queues/exchanges, server-named queues/exchanges, or both. Client-named means your app set the name of the exchange/queue; server-named means your app didn’t supply a name and let the server assign a random one for you.

例如，向用户 *cashing-tier* 授予访问虚拟机 *sycamore* 的全部权限。利用`rabbitmqctl`的`set_permissions`命令：

    $ rabbitmqctl set_permissions -p sycamore cashing-tier ".*" ".*" ".*"
    Setting permissions for user "cashing-tier" in vhost "sycamore" ...
    ...done.

`set_permissions`命令组成：

- `-p sycamore`：指定作用于哪个虚拟机
- `cashing-tier`：授予哪个用户
- `".*" ".*" ".*"`：权限。分别是configure, write和read权限。

三个权限制都是正则表达式（Perl的语法）。"`.*`"表示匹配所有队列和exchange名。

例子，授予用户*cashing-tier*访问虚拟机*oak*。允许用户执行任何读取。限制写只能对以`checks-`开头的队列或Exchange。禁止所有配置。执行：`set_permissions`：

    $ ./rabbitmqctl set_permissions -p oak \
    -s all cashing-tier "" "checks-.*" ".*"
    Setting permissions for user "cashing-tier"in vhost "oak" ...
    ...done.

用`list_permissions`命令查看权限：

    $ ./rabbitmqctl list_permissions -p oak
    Listing permissions in vhost "oak"...
    cashing-tier checks-.* .* all
    ...done.

利用`clear_permissions`移除用户在特定虚拟机上的权限：

    $ ./rabbitmqctl clear_permissions -p oak cashing-tier
    Clearing permissions for user "cashing-tier"in vhost "oak" ...
    ...done.

利用`list_user_permissions`更新权限：

    $ ./rabbitmqctl list_user_permissions cashing-tier
    Listing permissions for user "cashing-tier"...
    oak checks-.* .* all
    sycamore .* .* .* all
    ...done.

## 管理插件

http://www.rabbitmq.com/management.html

管理插件使得可以通过Web管理和监控服务器，还提供REST API和一个命令行工具`rabbitmqadmin`。特性：

- 声明、列出、删除交换机、队列、绑定、用户、虚拟机和权限
- 管理全局的，或某个信道的队列长度、消息速率等
- 收发消息
- 管理 Erlang 进程、文件描述符、内存等
- 导出、导入对象定义（JSON格式）
- 强制关闭连接、清空队列

### 启用

管理插件包含在 RabbitMQ 分发内。使用[rabbitmq-plugins](http://www.rabbitmq.com/man/rabbitmq-plugins.1.man.html)启用：

	rabbitmq-plugins enable rabbitmq_management

If you wish to build the plugin from source, it can be built like any other. See the [plugin development page](http://www.rabbitmq.com/plugin-development.html) for more information.

- Web界面在：http://server-name:15672/
- HTTP API及其文档在http://server-name:15672/api/。最新的API文档在[这](http://hg.rabbitmq.com/rabbitmq-management/raw-file/rabbitmq_v3_3_5/priv/www/api/index.html)。
- Download [rabbitmqadmin](http://www.rabbitmq.com/management-cli.html) at: http://server-name:15672/cli/

注意：3.0之前的端口在55672。

登录需要RabbitMQ用户（默认用户密码都是guest）。

管理界面是单页应用，背后通过Javascript与HTTP API通信。

### 权限

管理插件扩展了权限模型。RabbitMQ中用户可以被给于任意标签。管理插件使用的标签是"management", "policymaker", "monitoring", "administrator"。

相关命令：[rabbitmqctl add_user](http://www.rabbitmq.com/man/rabbitmqctl.1.man.html#)、[rabbitmqctl set_user_tags](http://www.rabbitmq.com/man/rabbitmqctl.1.man.html#set_user_tags)。

|标签|能力|
|---|----|
|(None)|不能访问管理插件|
|management|用户可以通过AMQP做的任何事，加上：列出它们可以登录的虚拟机；查看它们虚拟机内的所有队列、交换机、绑定；查看和关闭它们的信道与连接；查看覆盖他的虚拟机的统计，包括其他用户在这些虚拟机中的活动|
|policymaker|在"management"的基础上：View, create and delete policies and parameters for virtual hosts to which they can log in via AMQP|
|monitoring|在"management"的基础上：List all virtual hosts, including ones they could not log in to via AMQP; View other users's connections and channels; View node-level data such as memory use and clustering; View truly global statistics for all virtual hosts;|
|administrator|Everything "policymaker" and "monitoring" can plus: Create and delete virtual hosts; View, create and delete users; View, create and delete permissions; Close other users's connections|

Normal RabbitMQ permissions still apply to monitors and administrators; just because a user is a monitor or administrator does not give them full access to exchanges, queues and bindings through either AMQP or the management plugin.

All users can only list objects within a particular virtual host if they have any permissions for that virtual host.

### HTTP API

The management plugin will create an HTTP-based API at http://server-name:15672/api/.

For HTTP API clients in several languages, see [Developer Tools](http://www.rabbitmq.com/devtools.html).

### （及以下未）Configuration
