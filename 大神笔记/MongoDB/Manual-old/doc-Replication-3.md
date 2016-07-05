[toc]

3.0

## Replication

### 介绍

http://docs.mongodb.org/manual/core/replication-introduction/

复制（Replication）是在多个服务器之间同步数据的过程。

**复制的目的**

复制通过冗余增加数据的可用性（availability）。复制还能让我们从硬件故障或服务中断中恢复。

有时可以通过复制提供读取能力（read capacity）。客户端可以将读写操作发送到不同服务器。哈可以将数据拷贝放在不同的数据中心，增加分布式应用的本地性和可用性。

**MongoDB的复制**

A replica set is a group of mongod instances that host the same data set. 一个mongod，作为主（primary），接收所有的写操作。其他的实力，都是从（secondaries），从主接收数据以保证数据相同。

[primary](http://docs.mongodb.org/manual/core/replica-set-primary/)接收客户端所有的写操作。一个复制集（replica set）只能有一个主机。为支持复制，主机将数据集的所有变更记录在它的 [oplog](http://docs.mongodb.org/manual/core/replica-set-oplog/)。For more information on primary node operation, see [Replica Set Primary](http://docs.mongodb.org/manual/core/replica-set-primary/).

[secondaries](http://docs.mongodb.org/manual/core/replica-set-secondary/)复制主机的 oplog，应用到它们的数据集。如果主机失联，复制集会从从机种选择一个作为主机。For more information on secondary members, see [Replica Set Secondary Members](http://docs.mongodb.org/manual/core/replica-set-secondary/).

You may add an extra mongod instance to a replica set as an [arbiter](http://docs.mongodb.org/manual/core/replica-set-arbiter/). 仲裁者（Arbiters）没有数据集。它负责维护数据集的法定人数（quorum），通过相应其他数据集成员的心跳和选举（election）请求。因为它没有数据集，因此它的开销可以低得多。若数据集有偶数个成员，添加一个仲裁者可以在主机选举中获得多数票。For more information on arbiters, see [Replica Set Arbiter](http://docs.mongodb.org/manual/core/replica-set-arbiter/).

仲裁者不能改变角色。但主机和从机之间可能转换。

**异步复制**

从机从主机复制操作是异步的。By applying operations after the primary, sets can continue to function despite the failure of one or more members. For more information on replication mechanics, see [Replica Set Oplog](http://docs.mongodb.org/manual/core/replica-set-oplog/#replica-set-oplog) and [Replica Set Data Synchronization](http://docs.mongodb.org/manual/core/replica-set-sync/#replica-set-sync).

**自动错误恢复**

当主机超过10秒未能与复制集其他成员联系时，复制集将尝试选择另一个成员作为主机。第一个收到多数投票的从机变成主机。

See [Replica Set Elections](http://docs.mongodb.org/manual/core/replica-set-elections/#replica-set-elections) and [Rollbacks During Replica Set Failover](http://docs.mongodb.org/manual/core/replica-set-rollbacks/#replica-set-rollbacks) for more information.

**读操作**

若复制集有且只有一个主机时，从主机读取保证严格的一致性。

客户端默认从主机读取；但客户端可以指定一个[读取偏好](http://docs.mongodb.org/manual/core/read-preference/)，将读取请求发送到从机。异步复制意味着凑够从机读取到的可能与主机不同。

In MongoDB, clients can see the results of writes before they are made durable:

- 不管[write concern](http://docs.mongodb.org/manual/reference/write-concern/)是什么，在写操作ACK请求的客户端前，其他客户端可以看到写操作的结果。
- Clients can read data which may be subsequently [rolled back](http://docs.mongodb.org/manual/core/replica-set-rollbacks/).

**其他特性**

Replica sets provide a number of options to support application needs. For example, you may deploy a replica set with [members in multiple data centers](http://docs.mongodb.org/manual/core/replica-set-architecture-geographically-distributed/), or control the outcome of elections by adjusting the [priority](http://docs.mongodb.org/manual/reference/command/replSetGetConfig/#replSetGetConfig.members[n].priority) of some members. Replica sets also support dedicated members for reporting, disaster recovery, or backup functions.

See [Priority 0 Replica Set Members](http://docs.mongodb.org/manual/core/replica-set-priority-0-member/#replica-set-secondary-only-members), [Hidden Replica Set Members](http://docs.mongodb.org/manual/core/replica-set-hidden-member/#replica-set-hidden-members) and [Delayed Replica Set Members](http://docs.mongodb.org/manual/core/replica-set-delayed-member/#replica-set-delayed-members) for more information.

> In some circumstances, two nodes in a replica set may transiently believe that they are the primary, but at most, one of them will be able to complete writes with `{w: majority}` write concern. The node that can complete `{w: majority}` writes is the current primary, and the other node is a former primary that has not yet recognized its demotion, typically due to a [network partition](http://docs.mongodb.org/manual/reference/glossary/#term-network-partition). When this occurs, clients that connect to the former primary may observe stale data despite having requested read preference primary.

**附加资源**

- [Quick Reference Cards](https://www.mongodb.com/lp/misc/quick-reference-cards?jmp=docs&_ga=1.258757897.2037076241.1438306699)
- [Webinar: Managing Your Mission Critical App - Ensuring Zero Downtime](http://www.mongodb.com/webinar/managing-mission-critical-app-downtime?jmp=docs&_ga=1.258757897.2037076241.1438306699)

### 复制的概念

#### 复制集成员

http://docs.mongodb.org/manual/core/replica-set-members/



