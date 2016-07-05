## Centos

### 安装Elang

1、添加到仓库

    wget http://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
    rpm -Uvh erlang-solutions-1.0-1.noarch.rpm

2、安装

	sudo yum install erlang

参考：https://www.erlang-solutions.com/downloads/download-erlang-otp。

### 安装RabbitMQ

参见：http://www.rabbitmq.com/install-rpm.html。大体分两步。

1、从列表中下载，如http://www.rabbitmq.com/releases/rabbitmq-server/v3.3.5/rabbitmq-server-3.3.5-1.noarch.rpm。

2、执行

    rpm --import http://www.rabbitmq.com/rabbitmq-signing-key-public.asc
    yum install rabbitmq-server-3.3.5-1.noarch.rpm

## 启动

默认不会作为守护进程启动。要随系统启动，执行

	chkconfig rabbitmq-server on

启动停止服务：

    /sbin/service rabbitmq-server stop/start
