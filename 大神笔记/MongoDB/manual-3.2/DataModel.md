[toc]

# Data Model

## 数据模型介绍

https://docs.mongodb.org/manual/core/data-modeling-introduction/

When designing a data model, consider how applications will use your database. For instance, if your application only uses recently inserted documents, consider using [Capped Collections](https://docs.mongodb.org/manual/core/capped-collections/).

**Additional Resources**

[Thinking in Documents Part 1 (Blog Post)](https://www.mongodb.com/blog/post/thinking-documents-part-1?jmp=docs&_ga=1.234485850.515386276.1456988650)

## （未）Document Validation

https://docs.mongodb.org/manual/core/document-validation/

## 数据模型设计

https://docs.mongodb.org/manual/core/data-model-design/

嵌入文档或使用引用。

嵌入文档。文档修改后增长的问题。Furthermore, documents in MongoDB must be smaller than the [maximum BSON document size](https://docs.mongodb.org/manual/reference/limits/#BSON-Document-Size). For bulk binary data, consider [GridFS](https://docs.mongodb.org/manual/core/gridfs/).


**Additional Resources**

[Thinking in Documents (Presentation)](http://www.mongodb.com/presentations/webinar-back-basics-1-thinking-documents?jmp=docs&_ga=1.234551002.515386276.1456988650)

[Schema Design for Time Series Data (Presentation)](http://www.mongodb.com/presentations/webinar-time-series-data-mongodb?jmp=docs&_ga=1.234551002.515386276.1456988650)

[Socialite, the Open Source Status Feed - Storing a Social Graph (Presentation)](http://www.mongodb.com/presentations/socialite-open-source-status-feed-part-2-managing-social-graph?jmp=docs&_ga=1.234551002.515386276.1456988650)

[MongoDB Rapid Start Consultation Services](https://www.mongodb.com/products/consulting?jmp=docs&_ga=1.234551002.515386276.1456988650#rapid_start)

## Operational Factors and Data Models

https://docs.mongodb.org/manual/core/data-model-operations/


**文档增长**

一些更新操作，如向数据添加元素，或添加新的字段，会增加一个文档的大小。

对 MMAPv1 存储引擎，若文档大小超过分配给它的空间，MongoDB 会在磁盘上为文档重新分配空间。With the MMAPv1 storage engine, document growth can impact write performance and lead to data fragmentation.

When using the MMAPv1 storage engine, growth consideration can affect the decision to normalize or denormalize data. 

从 3.0.0 开始，MongoDB 使用 [Power of 2 Sized Allocations](https://docs.mongodb.org/manual/core/mmapv1/#power-of-2-allocation) 作为 MMAPv1 默认的分配策略。该策略减少了需要重新分配的可能，而且能更有效的利用已释放的记录空间。

When using MMAPv1, if your applications require updates that will frequently cause document growth to exceeds the current power of 2 allocation, you may want to refactor your data model to use references between data in distinct documents rather than a denormalized data model.

还可以使用预分配策略。Refer to the [Pre-Aggregated Reports Use Case](https://docs.mongodb.org/ecosystem/use-cases/pre-aggregated-reports) for an example of the pre-allocation approach to handling document growth.

**写操作的原子性问题**

在 MongoDB 中，写操作在文档级别是原子的，一个写操作无法原子地影响超过一个文档或超过一个集合。一些能够修改多个文档的操作，本质也是一次操作一个文档的的。

**Sharding**

MongoDB uses [sharding](https://docs.mongodb.org/manual/reference/glossary/#term-sharding) to provide horizontal scaling. These clusters support deployments with large data sets and high-throughput operations. Sharding allows users to partition a collection within a database to distribute the collection’s documents across a number of mongod instances or shards.

To distribute data and application traffic in a sharded collection, MongoDB uses the [shard key](https://docs.mongodb.org/manual/core/sharding-shard-key/#shard-key). Selecting the proper shard key has significant implications for performance, and can enable or prevent query isolation and increased write capacity. It is important to consider carefully the field or fields to use as the shard key.

See [Sharding Introduction](https://docs.mongodb.org/manual/core/sharding-introduction/) and [Shard Keys](https://docs.mongodb.org/manual/core/sharding-shard-key/) for more information.

**索引**

- 每个索引需要至少8kB空间。
- 索引对鞋操作性能有负面影响。对于写远大于读的集合，索引是昂贵的，因为每次插入需要更新索引。
- When active, each index consumes disk space and memory. This usage can be significant and should be tracked for capacity planning, especially for concerns over working set size.
See [Indexing Strategies](https://docs.mongodb.org/manual/applications/indexes/) for more information on indexes as well as [Analyze Query Performance](https://docs.mongodb.org/manual/tutorial/analyze-query-plan/). Additionally, the MongoDB [database profiler](https://docs.mongodb.org/manual/tutorial/manage-the-database-profiler/) may help identify inefficient queries.

**大量集合**

Generally, having a large number of collections has no significant performance penalty and results in very good performance. Distinct collections are very important for high-throughput batch processing.

**数据生命周期管理**

Data modeling decisions should take data lifecycle management into consideration.

The [Time to Live or TTL feature](https://docs.mongodb.org/manual/tutorial/expire-data/) of collections expires documents after a period of time. Consider using the TTL feature if your application requires some data to persist in the database for a limited period of time.

Additionally, if your application only uses recently inserted documents, consider [Capped Collections](https://docs.mongodb.org/manual/core/capped-collections/). Capped collections provide first-in-first-out (FIFO) management of inserted documents and efficiently support operations that insert and read documents based on insertion order.

## （未）Data Model Examples and Patterns

https://docs.mongodb.org/manual/applications/data-models/

## 数据模型参考

### 文档

https://docs.mongodb.org/manual/core/document/

The mongo JavaScript shell and the MongoDB language drivers translate between BSON and the language-specific document representation.

**字段名**

字段名是字符串。限制如下：

- `_id` 保留用于主键；必须唯一；不可变；不能是数组。
- 字段名不能以 `$` 开头。
- 字段名不能包含 `.` 字符。
- 字段名不能包含 `null` 字符。

BSON documents may have more than one field with the same name. Most MongoDB interfaces, however, represent MongoDB with a structure (e.g. a hash table) that does not support duplicate field names. If you need to manipulate documents that have more than one field with the same name, see the [driver documentation](https://docs.mongodb.org/manual/applications/drivers/) for your driver.

Some documents created by internal MongoDB processes may have duplicate fields, but no MongoDB process will ever add duplicate fields to an existing user document.

**字段值限制**

For indexed collections, the values for the indexed fields have a [Maximum Index Key Length](https://docs.mongodb.org/manual/reference/limits/#Index-Key-Limit) limit. See [Maximum Index Key Length](https://docs.mongodb.org/manual/reference/limits/#Index-Key-Limit) for details.

**文档的限制**

1、文档大小限制。最大 BSON 文档大小是16M。To store documents larger than the maximum size, MongoDB provides the GridFS API. See [mongofiles](https://docs.mongodb.org/manual/reference/program/mongofiles/#bin.mongofiles) and the documentation for your driver for more information about GridFS.

2、文档字段顺序。字段的属性保留写入时的顺序，除了：`_id` 字段总是文档的第一个字段；Updates that include [renaming](https://docs.mongodb.org/manual/reference/operator/update/rename/#up._S_rename) of field names may result in the reordering of fields in the document.

Changed in version 2.6: Starting in version 2.6, MongoDB actively attempts to preserve the field order in a document. Before version 2.6, MongoDB did not actively preserve the order of the fields in a document.

**`_id` 字段**

`_id` 字段有以下行为和限制：

- By default, MongoDB creates a unique index on the `_id` field during the creation of a collection.
- `_id` 字段总是文档的第一个字段。If the server receives a document that does not have the `_id` field first, then the server will move the field to the beginning.
- The `_id` field may contain values of any BSON data type, other than an array.

> To ensure functioning replication, do not store values that are of the BSON regular expression type in the `_id` field.

The following are common options for storing values for `_id`:

- 用 ObjectId。
- 若可能，使用自然的唯一标示符。节省空间，避免额外的索引。
- 产生自增的数字。See [Create an Auto-Incrementing Sequence Field](https://docs.mongodb.org/manual/tutorial/create-an-auto-incrementing-field/).
- 在应用中产生一个 UUID。For a more efficient storage of the UUID values in the collection and in the `_id` index, store the UUID as a value of the BSON `BinData` type. Index keys that are of the `BinData` type are more efficiently stored in the index if:
  - the binary subtype value is in the range of 0-7 or 128-135, and
  - the length of the byte array is: 0, 1, 2, 3, 4, 5, 6, 7, 8, 10, 12, 14, 16, 20, 24, or 32.
- Use your driver’s BSON UUID facility to generate UUIDs. Be aware that driver implementations may implement UUID serialization and deserialization logic differently, which may not be fully compatible with other drivers. See your driver documentation for information concerning UUID interoperability.

Most MongoDB driver clients will include the `_id` field and generate an `ObjectId` before sending the insert operation to MongoDB; however, if the client sends a document without an `_id` field, the mongod will add the `_id` field and generate the `ObjectId`.

### 数据引用

https://docs.mongodb.org/manual/reference/database-references/