[toc]

# MongoDB CRUD 操作

## MongoDB CRUD 介绍

## MongDB CRUD 概念

### 读操作

http://docs.mongodb.org/manual/core/read-operations-introduction/

#### 读操作概述

查询通过 `db.collection.find()`。指定查询条件和投影，返回游标。例子：

```js
db.users.find( { age: { $gt: 18 } }, { name: 1, address: 1 } ).limit(5)
```

查询行为：

- 查询只能处理单个集合
- You can modify the query to impose limits, skips, and sort orders.
- 返回文档的顺序不定，除非指定了 `sort()`。

`db.collection.findOne()` 只返回单个文档。

**投影**

查询默认返回所有字段。投影可以减少网络开销和处理量。

投影是 `find()` 的第二个参数，可以指定包含哪些字段，也可以指定要排除哪些字段。

Except for excluding the `_id` field in inclusive projections, you cannot mix exclusive and inclusive projections.

例子：

从结果集中排除一个字段 `history`：

```js
db.records.find( { "user_id": { $lt: 42 } }, { "history": 0 } )
```

只返回两个字段和 `_id`：

```js
db.records.find( { "user_id": { $lt: 42 } }, { "name": 1, "email": 1 } )
```

只返回两个字段，不包括`_id`：

```js
db.records.find( { "user_id": { $lt: 42} }, { "_id": 0, "name": 1 , "email": 1 } )
```

MongoDB 投影的特性：

- 默认 `_id` 包含在结果中。除非通过 `_id: 0` 显式排除。
- 对于数组字段，有以下投影运算符：[`$elemMatch`](https://docs.mongodb.org/manual/reference/operator/projection/elemMatch/#proj._S_elemMatch), [`$slice`](https://docs.mongodb.org/manual/reference/operator/projection/slice/#proj._S_slice), and [`$`](https://docs.mongodb.org/manual/reference/operator/projection/positional/#proj._S_)。

#### 游标

https://docs.mongodb.org/manual/core/cursors/

要遍历游标，得先存到一个变量里。在Shell中，若未分配变量，则游标会自动迭代20次，打印文档的前20个结果。可以通过 `DBQuery.shellBatchSize` 改变默认值 20。See [Executing Queries](http://docs.mongodb.org/manual/tutorial/getting-started-with-the-mongo-shell/#mongo-shell-executing-queries) for more information.

To manually iterate the cursor to access the documents, see [Iterate a Cursor in the mongo Shell](http://docs.mongodb.org/manual/tutorial/iterate-a-cursor/).

**不活动游标的关闭**

默认，当游标10分钟不活动，或客户端已经遍历完游标，服务器将自动关闭游标。若覆盖该默认的行为，可以在查询时指定 `noTimeout` [wire protocol flag](http://docs.mongodb.org/meta-driver/latest/legacy/mongodb-wire-protocol)；无论如何你应该让游标自动关闭或遍历完游标。

In the mongo shell, you can set the `noTimeout` flag:

```js
var myCursor = db.inventory.find().noCursorTimeout();
```

设置 `noCursorTimeout` 选项后，你应该通过 `cursor.close()` 手工关闭它或遍历完。

See your driver documentation for information on setting the noCursorTimeout option.

**Cursor Isolation**

当游标返回文档时，其他操作可能与查询交叠。对于 [MMAPv1](https://docs.mongodb.org/manual/core/mmapv1/) 存储引擎，交叠写和查询可能导致游标返回一个文档多次，如果该文档发生了变化。To handle this situation, see the information on [snapshot mode](https://docs.mongodb.org/manual/faq/developers/#faq-developers-isolate-cursors).

**Cursor Batches**

MongoDB 以批量的形式返回查询结果。批次大小不会超过 [maximum BSON document size](https://docs.mongodb.org/manual/reference/limits/#limit-bson-document-size)。对于多数查询，第一批次返回前101个文档，或略大于1M的。后续批次大小是4M。To override the default size of the batch, see [batchSize()](https://docs.mongodb.org/manual/reference/method/cursor.batchSize/#cursor.batchSize) and [limit()](https://docs.mongodb.org/manual/reference/method/cursor.limit/#cursor.limit).

对于不能使用索引的排序，服务器必须将素有文档加载到内存，才能开始返回结果。

As you iterate through the cursor and reach the end of the returned batch, if there are more results, `cursor.next()` will perform a [getmore operation](https://docs.mongodb.org/manual/reference/method/db.currentOp/#currentOp.op) to retrieve the next batch. To see how many documents remain in the batch as you iterate the cursor, you can use the [objsLeftInBatch()](https://docs.mongodb.org/manual/reference/method/cursor.objsLeftInBatch/#cursor.objsLeftInBatch) method, as in the following example:

```js
var myCursor = db.inventory.find();
var myFirstDocument = myCursor.hasNext() ? myCursor.next() : null;
myCursor.objsLeftInBatch();
```

**游标信息**

[`db.serverStatus()`](https://docs.mongodb.org/manual/reference/method/db.serverStatus/#db.serverStatus) 返回返回一个文档，里面有一个 [`metrics`](https://docs.mongodb.org/manual/reference/command/serverStatus/#serverstatus.metrics) 字段，字段值是一个文档，里面有 [`metrics.cursor`](https://docs.mongodb.org/manual/reference/command/serverStatus/#serverstatus.metrics.cursor) 字段，该字段包含信息：

- 服务器重启以来超时的游标数量。
- 打开的且设置了 `DBQuery.Option.noTimeout` 的游标数量。
- number of “pinned” open cursors
- 已打开的游标总数量

```
db.serverStatus().metrics.cursor
{
   "timedOut" : <number>
   "open" : {
      "noTimeout" : <number>,
      "pinned" : <number>,
      "total" : <number>
   }
}
```

#### 查询优化（索引）

http://docs.mongodb.org/manual/core/query-optimization/

For more information about indexes, see [the complete documentation of indexes in MongoDB](http://docs.mongodb.org/manual/core/indexes/).

##### 创建索引支持读操作

例子。给 `inventory` 集合的 `type` 字段加索引：

```js
db.inventory.ensureIndex( { type: 1 } )
```

To analyze the performance of the query with an index, see [Analyze Query Performance](http://docs.mongodb.org/manual/tutorial/analyze-query-plan/).

除了优化读操作，索引支持排序，以及可以帮助存储优化。See [db.collection.ensureIndex()](http://docs.mongodb.org/manual/reference/method/db.collection.ensureIndex/#db.collection.ensureIndex) and [Indexing Tutorials](http://docs.mongodb.org/manual/administration/indexes/) for more information about index creation.

To index fields in subdocuments, use dot notation.


##### 查询的选择性

查询的选择性表示查询是否能够有效利用索引，或用不用的问题。选择性更高，匹配的文档比例更小。例如，相等匹配 `_id` 字段选择性极高：选出至多一个文档。低选择性的查询不能有效使用索引，或根本用不到。

例如，不等运算符 `$nin`、`$ne` 不是非常有效，因为它们往往匹配索引的大部分。因此，很多情况下，`$nin` 或 `$ne` 使用索引，比全文档扫描性能没什么提高。

The selectivity of [regular expressions](https://docs.mongodb.org/manual/reference/operator/query/regex/#op._S_regex) depends on the expressions themselves. For details, see [regular expression and index use](https://docs.mongodb.org/manual/reference/operator/query/regex/#regex-index-use).

##### 覆盖一个查询

索引覆盖一个查询，需要：

- 查询中所有的字段都在索引中，
- 结果集中返回的所有字段在同一个索引中。

For example, a collection `inventory` has the following index on the `type` and `item` fields:

```js
db.inventory.createIndex( { type: 1, item: 1 } )
```

This index will cover the following operation which queries on the `type` and `item` fields and returns only the `item` field:

```js
db.inventory.find(
   { type: "food", item:/^c/ },
   { item: 1, _id: 0 }
)
```

注意必须从结果集中排除 `_id`，因为索引中没有包含 `_id` 字段。

Querying only the index can be much faster than querying documents outside of the index. Index keys are typically smaller than the documents they catalog, and indexes are typically available in RAM or located sequentially on disk.

##### 限制

索引不能覆盖查询，如果：

- 索引的字段，在任一文档中是一个数组。If an indexed field is an array, the index becomes a multi-key index and cannot support a covered query.
- 查询条件或结果集中的任何索引字段，在嵌入文档中。如若有文档结构：`{ _id: 1, user: { login: "tester" } }`。索引是 `{ "user.login": 1 }`。则该索引不能覆盖以下查询 `db.users.find( { "user.login": "tester" }, { "user.login": 1, _id: 0 } )`。However, the query can use the `{ "user.login": 1 }` index to find matching documents.

**Restrictions on Sharded Collection**

An index cannot cover a query on a sharded collection when run against a [mongos](https://docs.mongodb.org/manual/reference/program/mongos/#bin.mongos) if the index does not contain the shard key, with the following exception for the `_id` index: If a query on a sharded collection only specifies a condition on the `_id` field and returns only the `_id` field, the `_id` index can cover the query when run against a mongos even if the `_id` field is not the shard key.

Changed in version 3.0: In previous versions, an index cannot cover a query on a sharded collection when run against a mongos.

##### explain

To determine whether a query is a covered query, use the `db.collection.explain()` or the [explain()](https://docs.mongodb.org/manual/reference/method/cursor.explain/#cursor.explain) method and review the [results](https://docs.mongodb.org/manual/reference/explain-results/#explain-output-covered-queries).

`db.collection.explain()` provides information on the execution of other operations, such as `db.collection.update()`. See `db.collection.explain()` for details.

For more information see [Measure Index Use](https://docs.mongodb.org/manual/tutorial/measure-index-use/#indexes-measuring-use).

#### （未）Query Plans

https://docs.mongodb.org/manual/core/query-plans/

#### （未）Distributed Queries

https://docs.mongodb.org/manual/core/distributed-queries/

### 写操作



