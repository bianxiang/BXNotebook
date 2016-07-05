**小心看准是1.0还是2.0的文档！**

http://mongodb.github.io/node-mongodb-native/2.0/

[toc]

Node.js Native是MongoDB官方的Node.js驱动。The Node.js driver handles the connections to a single MongoDB server, a replicaset or a set of Mongos proxies in a sharded system.

## 介绍

http://mongodb.github.io/node-mongodb-native/2.0/overview/introduction/

## 安装

http://mongodb.github.io/node-mongodb-native/2.0/overview/installing/

	npm install mongodb

## 快速入门

http://mongodb.github.io/node-mongodb-native/2.0/overview/quickstart/

For more inn depth coverage we encourage reading the tutorials.

**启动服务器**

	mongod --dbpath=/data --port 27017

**链接到服务器**

Let’s create a new app.js file that we will use to show the basic CRUD operations using the MongoDB driver.

链接到数据库`myproject`。

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');

    // Connection URL
    var url = 'mongodb://localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");
      db.close();
    });

**插入一个文档**

    var insertDocuments = function(db, callback) {
      // 先获取文档集合
      var collection = db.collection('documents');
      collection.insert([
        {a : 1}, {a : 2}, {a : 3}
      ], function(err, result) {
        assert.equal(err, null);
        assert.equal(3, result.result.n);
        assert.equal(3, result.ops.length);
        console.log("Inserted 3 document into the document collection");
        callback(result);
      });
    }

`insert`命令将返回一个结果对象，包含：

- `result`：Contains the result document from MongoDB
- `ops`：包含已插入的文档，包括`_id`字段
- `connection` Contains the connection used to perform the insert

Let’s add call the `insertDocuments` command to the `MongoClient.connect` method callback.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');

    // Connection URL
    var url = 'mongodb://localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");
      insertDocuments(db, function() {
        db.close();
      });
    });

**更新文档**

    var updateDocument = function(db, callback) {
      var collection = db.collection('documents');
      collection.update({ a : 2 }
        , { $set: { b : 1 } }, function(err, result) {
        assert.equal(err, null);
        assert.equal(1, result.result.n);
        console.log("Updated the document with the field a equal to 2");
        callback(result);
      });
    }

只会更新匹配的第一个文档。

**删除一个文档**

    var removeDocument = function(db, callback) {
      // Get the documents collection
      var collection = db.collection('documents');
      // Insert some documents
      collection.remove({ a : 3 }, function(err, result) {
        assert.equal(err, null);
        assert.equal(1, result.result.n);
        console.log("Removed the document with the field a equal to 3");
        callback(result);
      });
    }

只会移除匹配的第一个文档。

**Find All Documents**

We will finish up the Quickstart CRUD methods by performing a simple query that returns all the documents matching the query.

    var findDocuments = function(db, callback) {
      // Get the documents collection
      var collection = db.collection('documents');
      // Insert some documents
      collection.find({}).toArray(function(err, docs) {
        assert.equal(err, null);
        assert.equal(2, docs.length);
        console.log("Found the following records");
        console.dir(docs)
        callback(docs);
      });
    }

## （未）教程

### 连接到服务器

主要方法是利用`MongoClient.connect`和一个URI。

#### 单服务器连接

We have a single MongoDB server instance running on the port 27017 Let’s connect using the driver and MongoClient.connect

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');

    // Connection URL
    var url = 'mongodb://localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");
      db.close();
    });

其中`myproject`是数据库名。

#### Replicaset Server Connection

We wish to connect to a ReplicaSet consisting of one primary and 1 or more secondaries. To Do this we need to supply the driver with a seedlist of servers and the name of the ReplicaSet we wish to connect to. Let’s take a look at a code example.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');

    // Connection URL
    var url = 'mongodb://localhost:27017,localhost:27018/myproject?replicaSet=foo';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");
      db.close();
    });

`replicaSet=foo` is the name of the ReplicaSet we are connecting to. This ensures we are connecting to the correct Replicaset. This is a required parameter when using the 2.0 driver

#### Mongos Proxy Connection

We wish to connect to a set of mongos proxies. Just as in the case of connecting to a ReplicaSet we can provide a seed list of mongos proxies. This allows the driver to perform failover between proxies automatically in case of a proxy process having been shut down. Let’s look at an example of code connecting to a set of proxies.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');

    // Connection URL
    var url = 'mongodb://localhost:50000,localhost:50001/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");
      db.close();
    });

#### 身份验证

##### Against The Specified Database

`MongoClient.connect` also allows us to specify authentication credentials as part of the URI. Let’s assume there is a user dave with the password password on the database protected. To correctly authenticate we will do the following.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');

    // Connection URL
    var url = 'mongodb://dave:password@localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");
      db.close();
    });

`dave:password` is the user name and password for the database.

The password and username must be URI **encoded** to allow for all any possible illegal characters

##### Indirectly Against Another Database

In some cases you might have to authenticate against another database than the one you intend to connect to. This is referred to as delegated authentication. Say you wish to connect to the `myproject` database but the user is defined in the `admin` database. Let’s look at how we would accomplish this.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');

    // Connection URL
    var url = 'mongodb://dave:password@localhost:27017/myproject?authSource=admin';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");
      db.close();
    });

`authSource=admin` is the database we wish to authenticate against.

#### MongoClient.connect其他参数

The driver has many more options for tweaking than what’s available through the URI specification. These can be passed to the driver using an optional parameters object. 下面是选项对象的顶层字段：

- `db`, 影响`MongoClient.connect`返回的数据库实例
- `replSet`, Options that modify the Replicaset topology connection behavior. This is a **required** parameter when using the 2.0 driver
- `mongos`, Options that modify the Mongos topology connection behavior.
- `server`, Options that modify the Server topology connection behavior.

例子。连接到单台服务器。设置查询返回原生的BSON buffers。调整连接池为10个连接。

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');
    var url = 'mongodb://dave:password@localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, {
        db: {
          raw: true
        },
        server: {
          poolSize: 10
        }
      }, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");
      db.close();
    });

下面是详细的选项：

##### Data base level options

- `w`, {Number/String, > -1 || ‘majority’} the write concern for the operation where < 1 is no acknowledgment of write and w >= 1 or w = ‘majority’ acknowledges the write
- `wtimeout`, {Number, 0} set the timeout for waiting for write concern to finish (combines with w option)
- `fsync`, (Boolean, default:false) write waits for fsync before returning
- `j`, (Boolean, default:false) write waits for journal sync before returning
- `readPreference` {String}, the preferred read preference (ReadPreference.PRIMARY, ReadPreference.PRIMARY_PREFERRED, ReadPreference.SECONDARY, ReadPreference.SECONDARY_PREFERRED, ReadPreference.NEAREST).
- `readPreferenceTags` {Object, default:null}, the tags object {‘loc’:‘ny’} used with the readPreference.
- `native_parser` {Boolean, default:false}, 使用c++ bson parser。
- `forceServerObjectId` {Boolean, default:false}, force server to create `_id` fields instead of client.
- `pkFactory` {Object}, object overriding the basic ObjectID primary key generation.
- `serializeFunctions` {Boolean, default:false}, serialize functions.
- `raw` {Boolean, default:false}, perform operations using raw bson buffers.
- `retryMiliSeconds` {Number, default:5000}, number of milliseconds between retries.
- `numberOfRetries` {Number, default:5}, number of retries off connection.
- `bufferMaxEntries` {Number, default: -1}, sets a cap on how many operations the driver will buffer up before giving up on getting a working connection, default is -1 which is unlimited.

##### Individual Server Level Options

- `poolSize`, {Number, default: 5} Number of connections in the connection pool for each server instance, set to **5** as default for legacy reasons.
- `ssl`, {Boolean, default: false} Number of connections in the connection pool for each server instance, set to 5 as default for legacy reasons.
- `sslValidate`, {Boolean, default: false} Validate mongod server certificate against ca (needs to have a mongod server with ssl support, 2.4 or higher).
- `sslCA`, {Buffer[]|string[], default: null} Array of valid certificates either as Buffers or Strings (needs to have a mongod server with ssl support, 2.4 or higher).
- `sslCert`, {Buffer|string, default: null} String or buffer containing the certificate we wish to present (needs to have a mongod server with ssl support, 2.4 or higher).
- `sslKey`, {Buffer|string, default: null} String or buffer containing the certificate private key we wish to present (needs to have a mongod server with ssl support, 2.4 or higher).
- `sslPass`, {Buffer|string, default: null} String or buffer containing the certificate password (needs to have a mongod server with ssl support, 2.4 or higher).
- `autoReconnect`, {Boolean, default: true} Reconnect on error.
- `socketOptions.noDelay`, {Boolean, default: true} TCP Socket NoDelay option.
- `socketOptions.keepAlive`, {Number, default: 0} TCP KeepAlive on the socket with a X ms delay before start.
- `socketOptions.connectTimeoutMS`, {Number, default: 0} TCP Connection timeout setting.
- `socketOptions.socketTimeoutMS`, {Number, default: 0} TCP Socket timeout setting.

##### Replicaset Level Options

ha {Boolean, default:true}, turn on high availability.
haInterval {Number, default:5000}, time between each replicaset status check.
replicaSet {String}, the name of the replicaset to connect to. This is a required parameter when using the 2.0 driver
secondaryAcceptableLatencyMS {Number, default:15}, sets the range of servers to pick when using NEAREST (lowest ping ms + the latency fence, ex: range of 1 to (1 + 15) ms)
connectWithNoPrimary {Boolean, default:false}, Sets if the driver should connect even if no primary is available.
poolSize, {Number, default: 5} Number of connections in the connection pool for each server instance, set to 5 as default for legacy reasons.
ssl, {Boolean, default: false} Number of connections in the connection pool for each server instance, set to 5 as default for legacy reasons.
sslValidate, {Boolean, default: false} Validate mongod server certificate against ca (needs to have a mongod server with ssl support, 2.4 or higher).
sslCA, {Buffer[]|string[], default: null} Array of valid certificates either as Buffers or Strings (needs to have a mongod server with ssl support, 2.4 or higher).
sslCert, {Buffer|string, default: null} String or buffer containing the certificate we wish to present (needs to have a mongod server with ssl support, 2.4 or higher).
sslKey, {Buffer|string, default: null} String or buffer containing the certificate private key we wish to present (needs to have a mongod server with ssl support, 2.4 or higher).
sslPass, {Buffer|string, default: null} String or buffer containing the certificate password (needs to have a mongod server with ssl support, 2.4 or higher).
socketOptions.noDelay, {Boolean, default: true} TCP Socket NoDelay option.
socketOptions.keepAlive, {Number, default: 0} TCP KeepAlive on the socket with a X ms delay before start.
socketOptions.connectTimeoutMS, {Number, default: 0} TCP Connection timeout setting.
socketOptions.socketTimeoutMS, {Number, default: 0} TCP Socket timeout setting.

##### Mongos Proxy Level Options

ha {Boolean, default:true}, turn on high availability.
haInterval {Number, default:5000}, time between each replicaset status check.
secondaryAcceptableLatencyMS {Number, default:15}, sets the range of servers to pick when using NEAREST (lowest ping ms + the latency fence, ex: range of 1 to (1 + 15) ms)
poolSize, {Number, default: 5} Number of connections in the connection pool for each server instance, set to 5 as default for legacy reasons.
ssl, {Boolean, default: false} Number of connections in the connection pool for each server instance, set to 5 as default for legacy reasons.
sslValidate, {Boolean, default: false} Validate mongod server certificate against ca (needs to have a mongod server with ssl support, 2.4 or higher).
sslCA, {Buffer[]|string[], default: null} Array of valid certificates either as Buffers or Strings (needs to have a mongod server with ssl support, 2.4 or higher).
sslCert, {Buffer|string, default: null} String or buffer containing the certificate we wish to present (needs to have a mongod server with ssl support, 2.4 or higher).
sslKey, {Buffer|string, default: null} String or buffer containing the certificate private key we wish to present (needs to have a mongod server with ssl support, 2.4 or higher).
sslPass, {Buffer|string, default: null} String or buffer containing the certificate password (needs to have a mongod server with ssl support, 2.4 or higher).
socketOptions.noDelay, {Boolean, default: true} TCP Socket NoDelay option.
socketOptions.keepAlive, {Number, default: 0} TCP KeepAlive on the socket with a X ms delay before start.
socketOptions.connectTimeoutMS, {Number, default: 0} TCP Connection timeout setting.
socketOptions.socketTimeoutMS, {Number, default: 0} TCP Socket timeout setting.

### 连接失败与重试

http://mongodb.github.io/node-mongodb-native/2.0/tutorials/connection_failures/

This comes up a lot because there is some confusion about how the driver works when it comes to Socket timeouts and retries. This Tutorial attempts to clarify the driver’s behavior and explains why, for some legacy reasons as well as for some design reasons, the driver works the way it does.

先以单服务器为例。

    var MongoClient = require('mongodb').MongoClient
      , f = require('util').format;

    MongoClient.connect('mongodb://localhost:27017/test', function(err, db) {
      var col = db.collection('t');
      setInterval(function() {
        col.insert({a:1}, function(err, r) {
          console.log("insert")
          console.log(err)
          col.findOne({}, function(err, doc) {
            console.log("findOne")
            console.log(err)
          });
        })
      }, 1000)
    });

启动脚本，控制台每隔一秒打印日志。现在关闭mongod进程，控制台停止打印。在背后，服务器缓存了操作，知道mongod恢复。这个行为受两个参数的默认值影响：

- `autoReconnect`。默认值true。驱动会尝试自动重连。
- `bufferMaxEntries`。默认值-1。重连成功前缓存的最大操作数量。Driver will error out all operations if the number of buffered operations goes over the limit set

为了向后兼容，启动会自动重连，缓存所有未完成操作。

Now let’s try to disable the bufferMaxEntries by setting it to 0 and see what happens.

    var MongoClient = require('mongodb').MongoClient
      , f = require('util').format;

    MongoClient.connect('mongodb://localhost:27017/test', {
        db: { bufferMaxEntries: 0 }
      }, function(err, db) {
      var col = db.collection('t');

      setInterval(function() {
        col.insert({a:1}, function(err, r) {
          console.log("insert")
          console.log(err)

          col.findOne({}, function(err, doc) {
            console.log("findOne")
            console.log(err)
          });
        })
      }, 1000)
    });

Start the script running and then shut down the mongod process. Notice how all operations are now erroring out instead of just being buffered? Now restart the mongod service and you will see the the operations once again correctly being executed.

So what happens if we disable `autoReconnect` by setting it to false? Let’s take a look.

    var MongoClient = require('mongodb').MongoClient
      , f = require('util').format;

    MongoClient.connect('mongodb://localhost:27017/test?autoReconnect=false', function(err, db) {
      var col = db.collection('t');

      setInterval(function() {
        col.insert({a:1}, function(err, r) {
          console.log("insert")
          console.log(err)

          col.findOne({}, function(err, doc) {
            console.log("findOne")
            console.log(err)
          });
        })
      }, 1000)
    });

When you shut down the mongod process, the driver stops processing operations and keeps buffering them due to `bufferMaxEntries` being -1 by default meaning buffer all operations. When you bring the mongod process back up you will notice how it does not change the fact that we are buffering. This is a legacy behavior and less than ideal. 因此，在关闭`autoReconnect`，最好将`bufferMaxEntries`设为0或其他小的值。

So why is this the case? Well, the main reason is a combination of the asynchronous behavior of node.js as well as Replicasets. When you are using a single server the behavior might be a bit mystifying, but it makes sense in the context of the Replicaset.

Say you have a Replicaset where a new primary is elected. If the driver does not buffer the operations, it will have to error out all operations until there is a new primary available in the set. This complicates people’s code as every operation could potentially fail and thus the driver a long time ago took the decision to make this transparent to the user by buffering operations until the new primary is available and then replaying them. `bufferMaxEntries` was added later to allow developers to control this behavior themselves if they wished to be instantly notified about write errors f.ex instead of letting the driver handle it.

结论。

A lot of the confusion comes from mistaking `socketTimeoutMS` with how the async driver works. `socketTimeoutMS` only applies to sockets if they have successfully connected to the server, but have not been in use and they reach the `socketTimeoutMS`. In contrast, `connectionTimeoutMS` applies only to the initial server connection process timeout. The `connectionTimeoutMS` is independent of the `socketTimeoutMS`.

However, some people set `socketTimeoutMS` expecting it to influence timeouts for operations. But as we have seen above the autoReconnect and bufferMaxEntries are the two settings that control that behavior.

It’s worth noting that you should ensure you have a reasonable `socketTimeoutMS`. A lot of people set it way way too low and find themselves with timeouts happening all the time as operations are infrequent enough to cause constant connection closing and reconnect events.

The rule of thumb I always impart is: Set `socketTimeoutMS` to at least 2-3x the longest running operation in your application or the interval between operations, too ensure you don’t timeout long running operations or servers where there are big gaps of time between operations.

多数改变`socketTimeoutMS`的人其实应改变`maxTimeMS`属性：限制查询运行时间。

    var MongoClient = require('mongodb').MongoClient
      , f = require('util').format;

    MongoClient.connect('mongodb://localhost:27017/test', function(err, db) {
      var cursor = db.collection('t').find({}).maxTimeMS(1000);
      cursor.toArray(function(err, docs) {
        console.dir(docs)
        db.close();
      });
    });

### （未）连接URI格式

http://mongodb.github.io/node-mongodb-native/2.0/tutorials/urls/

	mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]

### CRUD 操作

#### 插入

The `insertOne` and `insertMany` methods exists on the `Collection` class and is used to insert documents into MongoDB.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');
    // Connection URL
    var url = 'mongodb://localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");
      // Insert a single document
      db.collection('inserts').insertOne({a:1}, function(err, r) {
        assert.equal(null, err);
        assert.equal(1, r.insertedCount);
        // Insert multiple documents
        db.collection('inserts').insertMany([{a:2}, {a:3}], function(err, r) {
          assert.equal(null, err);
          assert.equal(2, r.insertedCount);
          db.close();
        });
      });
    });

The first insert inserts a single document into the inserts collection. Notice that we are not explicitly creating a new inserts collection as the server will create it implicitly when we insert the first document. The method `Db.createIndex` only really needs to be used when creating non standard collections such as capped collections or where other parameters than the default collections need to be applied.

`insertOne`和`insertMany`方法还接受第二个参数，传入选项对象。这个对象可以包含以下对象。

- `w`, {Number/String, > -1 || ‘majority’} the write concern for the operation where < 1 is no acknowledgment of write and w >= 1 or w = ‘majority’ acknowledges the write.
- `wtimeout`, {Number, 0} set the timeout for waiting for write concern to finish (combines with w option).
- `j`, (Boolean, default:false) write waits for journal sync.
- `serializeFunctions`, (Boolean, default:false) serialize functions on an object to mongodb, by default the driver does not serialize any functions on the passed in documents.
- `forceServerObjectId`, (Boolean, default:false) Force server to assign _id values instead of driver.

Let’s look at a simple example where we are writing to a replicaset and we wish to ensure that we serialize a passed in function as well as have the server assign the `_id` for each document.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');
    // Connection URL
    var url = 'mongodb://localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");
      // Insert a single document
      db.collection('inserts').insertOne({
            a:1
          , b: function() { return 'hello'; }
        }, {
            w: 'majority'
          , wtimeout: 10000
          , serializeFunctions: true
          , forceServerObjectId: true
        }, function(err, r) {
        assert.equal(null, err);
        assert.equal(1, r.insertedCount);
        db.close();
      });
    });

#### 更新文档

The `updateOne` and `updateMany` methods exists on the `Collection` class and is used to update and upsert documents into MongoDB.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');
    // Connection URL
    var url = 'mongodb://localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");
      var col = db.collection('updates');
      // Insert a single document
      col.insertMany([{a:1}, {a:2}, {a:2}], function(err, r) {
        assert.equal(null, err);
        assert.equal(3, r.insertedCount);

        // Update a single document
        col.updateOne({a:1}, {$set: {b: 1}}, function(err, r) {
          assert.equal(null, err);
          assert.equal(1, r.matchedCount);
          assert.equal(1, r.modifiedCount);

          // Update multiple documents
          col.updateMany({a:2}, {$set: {b: 1}}, function(err, r) {
            assert.equal(null, err);
            assert.equal(2, r.matchedCount);
            assert.equal(2, r.modifiedCount);

            // Upsert a single document
            col.updateOne({a:3}, {$set: {b: 1}}, {
              upsert: true
            }, function(err, r) {
              assert.equal(null, err);
              assert.equal(0, r.matchedCount);
              assert.equal(1, r.upsertedCount);
              db.close();
            });
          });
        });
      });
    });

The `update` method also accepts an second argument that can be an options object. This object can have the following fields.

- `w`, {Number/String, > -1 || ‘majority’} the write concern for the operation where < 1 is no acknowledgment of write and w >= 1 or w = ‘majority’ acknowledges the write.
- `wtimeout`, {Number, 0} set the timeout for waiting for write concern to finish (combines with w option).
- `j`, (Boolean, default:false) write waits for journal sync.
- `multi`, (Boolean, default:false) Update one/all documents with operation.
- `upsert`, (Boolean, default:false) Update operation is an upsert.

Just as for insert the update method allows you to specify a per operation write concern using the `w`, `wtimeout` and `fsync` parameters

#### 删除文档

The `deleteOne` and `deleteMany` methods exist on the `Collection` class and is used to remove documents from MongoDB. Let’s look at a couple of usage examples.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');

    // Connection URL
    var url = 'mongodb://localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");
      var col = db.collection('removes');
      // Insert a single document
      col.insertMany([{a:1}, {a:2}, {a:2}], function(err, r) {
        assert.equal(null, err);
        assert.equal(3, r.insertedCount);
        // Update a single document
        col.deleteOne({a:1}, {$set: {b: 1}}, function(err, r) {
          assert.equal(null, err);
          assert.equal(1, r.deletedCount);
          // Update multiple documents
          col.deleteMany({a:2}, {$set: {b: 1}}, function(err, r) {
            assert.equal(null, err);
            assert.equal(2, r.deletedCount);
            db.close();
          });
        });
      });
    });

The `removeOne` and `deleteMany` methods also accepts an second argument that can be an options object. This object can have the following fields.

- `w`, {Number/String, > -1 || ‘majority’} the write concern for the operation where < 1 is no acknowledgment of write and w >= 1 or w = ‘majority’ acknowledges the write.
- `wtimeout`, {Number, 0} set the timeout for waiting for write concern to finish (combines with w option).
- `j`, (Boolean, default:false) write waits for journal sync.
- `single`, (Boolean, default:false) Removes the first document found.

Just as for updateOne/updateMany and insertOne/insertMany the deleteOne/deleteMany method allows you to specify a per operation write concern using the `w`, `wtimeout` and `fsync` parameters.

#### findAndModify和findAndDelete

The two methods `findOneAndUpdate`, `findOneAndDelete` and `findOneAndReplace` are special commands that allows the user to update or upsert a document and have the modified or existing document returned. It comes at a cost as the operation takes a write **lock** for the duration of the operation as it needs to ensure the modification is atomic. Let’s look at `findOneAndUpdate` first using an example.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');
    // Connection URL
    var url = 'mongodb://localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");
      var col = db.collection('findAndModify');
      // Insert a single document
      col.insert([{a:1}, {a:2}, {a:2}], function(err, r) {
        assert.equal(null, err);
        assert.equal(3, r.result.n);
        // Modify and return the modified document
        col.findOneAndUpdate({a:1}, {$set: {b: 1}}, {
            returnOriginal: false
          , sort: [[a,1]]
          , upsert: true
        }, function(err, doc) {
          assert.equal(null, err);
          assert.equal(1, r.value.b);

          // Remove and return a document
          col.findOneAndDelete({a:2}, function(err, r) {
            assert.equal(null, err);
            assert.ok(r.value.b == null);
            db.close();
          });
        });
      });
    });

The `findOneAndUpdate` method also accepts an second argument that can be an options object. This object can have the following fields.

- `w`, {Number/String, > -1 || ‘majority’} the write concern for the operation where < 1 is no acknowledgment of write and w >= 1 or w = ‘majority’ acknowledges the write.
- `wtimeout`, {Number, 0} set the timeout for waiting for write concern to finish (combines with w option).
- `j`, (Boolean, default:false) write waits for journal sync.
- `upsert`, (Boolean, default:false) Perform an upsert operation.
- `sort`, (Object, default:null) Sort for find operation.
- `projection`, (Object, default:null) Projection for returned result
- `returnOriginal`, (Boolean, default:true) Set to false if you want to return the modified object rather than the original. Ignored for remove.

The `findAndDelete` function is a function especially defined to help remove a document. Let’s look at an example of usage.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');

    // Connection URL
    var url = 'mongodb://localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");

      var col = db.collection('findAndModify');
      // Insert a single document
      col.insert([{a:1}, {a:2}, {a:2}], function(err, r) {
        assert.equal(null, err);
        assert.equal(3, r.result.n);

        // Remove a document from MongoDB and return it
        col.findOneAndRemove({a:1}, {
            sort: [[a,1]]
          }
          , function(err, doc) {
            assert.equal(null, err);
            assert.ok(r.value.b == null);
            db.close();
        });
      });
    });

Just as for `findOneAndUpdate` it allows for an object of options to be passed in that can have the following fields.

- `w`, {Number/String, > -1 || ‘majority’} the write concern for the operation where < 1 is no acknowledgment of write and w >= 1 or w = ‘majority’ acknowledges the write.
- `wtimeout`, {Number, 0} set the timeout for waiting for write concern to finish (combines with w option).
- `j`, (Boolean, default:false) write waits for journal sync.
- `sort`, (Object, default:null) Sort for find operation.

#### BulkWrite

The `bulkWrite` function allows for a simple set of bulk operations to be done in a **non fluent** way as in comparison to the bulk API discussed next. Let’s look at an example.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');
    // Connection URL
    var url = 'mongodb://localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");
      // Get the collection
      var col = db.collection('bulk_write');
      col.bulkWrite([
          { insertOne: { document: { a: 1 } } }
        , { updateOne: { filter: {a:2}, update: {$set: {a:2}}, upsert:true } }
        , { updateMany: { filter: {a:2}, update: {$set: {a:2}}, upsert:true } }
        , { deleteOne: { filter: {c:1} } }
        , { deleteMany: { filter: {c:1} } }
        , { replaceOne: { filter: {c:3}, replacement: {c:4}, upsert:true}}]
      , {ordered:true, w:1}, function(err, r) {
        assert.equal(null, err);
        assert.equal(1, r.insertedCount);
        assert.equal(1, Object.keys(r.insertedIds).length);
        assert.equal(1, r.matchedCount);
        assert.equal(0, r.modifiedCount);
        assert.equal(0, r.deletedCount);
        assert.equal(2, r.upsertedCount);
        assert.equal(2, Object.keys(r.upsertedIds).length);

        // Ordered bulk operation
        db.close();
      });
    });

As we can see the bulkWrite function takes an array of operation that can be objects of either insertOne, insertMany, updateOne, updateMany, deleteOne or deleteMany. It also takes a second parameter that takes the following options.

- `ordered`, (Boolean, default:true) Execute in order or out of order.
- `w`, {Number/String, > -1 || ‘majority’} the write concern for the operation where < 1 is no acknowledgment of write and w >= 1 or w = ‘majority’ acknowledges the write.
- `wtimeout`, {Number, 0} set the timeout for waiting for write concern to finish (combines with w option).
- `j`, (Boolean, default:false) write waits for journal sync.

This covers the basic write operations. Let’s have a look at the Bulk write operations next.

#### Bulk Write Operations

The bulk write operations make it easy to write groups of operations together to MongoDB. There are some caveats and to get the best performance you need to be running against MongoDB 2.6 or higher that support the new write commands. Bulk operations are split into ordered and unordered bulk operations. An ordered bulk operation guarantees the order of execution of writes while the unordered bulk operation makes no assumptions about the order of execution. In the Node.js driver the unordered bulk operations will group operations according to type and write them in parallel. Let’s have a look at how to build an ordered bulk operation.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');

    // Connection URL
    var url = 'mongodb://localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");

      var col = db.collection('bulkops');
      // Create ordered bulk, for unordered initializeUnorderedBulkOp()
      var bulk = col.initializeOrderedBulkOp();
      // Insert 10 documents
      for(var i = 0; i < 10; i++) {
        bulk.insert({a: i});
      }

      // Next perform some upserts
      for(var i = 0; i < 10; i++) {
        bulk.find({b:i}).upsert().updateOne({b:1});
      }

      // Finally perform a remove operation
      bulk.find({b:1}).deleteOne();

      // Execute the bulk with a journal write concern
      bulk.execute(function(err, result) {
        assert.equal(null, err);
        db.close();
      });
    });

We will not cover the results object here as it’s documented in the driver API. The Bulk API handles all the splitting of operations into multiple writes and also emulates 2.6 and higher write commands for 2.4 and earlier servers.

There is are some important things to keep in mind when using the bulk API and especially the ordered bulk API mode. The write commands are single operation type. That means they can only do insert/update and remove. If you f.ex do the following combination of operations.

    Insert {a:1}
    Update {a:1} to {a:1, b:1}
    Insert {a:2}
    Remove {b:1}
    Insert {a:3}

This will result in the driver issuing 4 write commands to the server.

    Insert Command with {a:1}
    Update Command {a:1} to {a:1, b:1}
    Insert Command with {a:2}
    Remove Command with {b:1}
    Insert Command with {a:3}

If you instead organize the your ordered in the following manner.

    Insert {a:1}
    Insert {a:2}
    Insert {a:3}
    Update {a:1} to {a:1, b:1}
    Remove {b:1}

The number of write commands issued by the driver will be.

    Insert Command with {a:1}, {a:2}, {a:3}
    Update Command {a:1} to {a:1, b:1}
    Remove Command with {b:1}

Allowing for more efficient and faster bulk write operation.

For **unordered** bulk operations this is not important as the driver sorts operations by type and executes them in parallel.

This covers write operations for MongoDB. Let’s look at querying for documents next.

#### 读操作

The main method for querying the database are the find and the aggregate method. In this CRUD tutorial we will focus on find only as aggregate has it’s own Aggregation Tutorial.

The method return a cursor that allows us to operate on the data. The cursor also implements the Node.js 0.10.x or higher stream interface allowing us to pipe the results to other streams. We will not cover streams here as they are covered in the Streams Tutorial.

Let’s look at a simple find example that materializes all the documents from a query using the `toArray` but limits the number of returned results to 2 documents.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');
    // Connection URL
    var url = 'mongodb://localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");
      var col = db.collection('find');
      // Insert a single document
      col.insertMany([{a:1}, {a:1}, {a:1}], function(err, r) {
        assert.equal(null, err);
        assert.equal(3, r.insertedCount);

        // Get first two documents that match the query
        col.find({a:1}).limit(2).toArray(function(err, docs) {
          assert.equal(null, err);
          assert.equal(2, docs.length);
          db.close();
        });
      });
    });

The cursor returned by the `find` method has a lot of methods that allow for chaining of options for a query. Once the query is ready to be executed you can retrieve the documents using the `next`, `each` and `toArray` methods. If the query returns a lot of documents it’s preferable to use the `next` or `each` methods as the `toArray` method will materialize all the documents into memory before calling the callback function potentially using a lot of memory if the query returns a lot of documents.

We won’t look at the options we can set on the cursor as they can be viewed in the [Cursor API documentation](http://mongodb.github.io/api-docs).

We already looked at `toArray` method above. Let’s take a look at the next method.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');

    // Connection URL
    var url = 'mongodb://localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");

      var col = db.collection('find');
      // Insert a single document
      col.insertMany([{a:1}, {a:1}, {a:1}], function(err, r) {
        assert.equal(null, err);
        assert.equal(3, r.insertedCount);

        // Get first documents from cursor
        col.find({a:1}).limit(2).next(function(err, doc) {
          assert.equal(null, err);
          assert.ok(doc != null);
          db.close();
        });
      });
    });

The `next` method allows the application to read one document at a time using callbacks. Let’s look at the `each` method next.

    var MongoClient = require('mongodb').MongoClient
      , assert = require('assert');
    // Connection URL
    var url = 'mongodb://localhost:27017/myproject';
    // Use connect method to connect to the Server
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");
      var col = db.collection('find');
      // Insert a single document
      col.insertMany([{a:1}, {a:1}, {a:1}], function(err, r) {
        assert.equal(null, err);
        assert.equal(3, r.insertedCount);
        // Get first documents from cursor using each
        col.find({a:1}).limit(2).each(function(err, doc) {
          if(doc) {
            db.close();
            // Got a document, terminate the each
            return false;
          }
        });
      });
    });

The `each` method will call the supplied callback until there are no more documents available that satisfy the query. Once the available documents is exhausted it will return `null` for the second parameter in the callback. If you wish to terminate the each early you should `return false` in your each callback. This will stop the cursor from returning documents.

This covers the basic crud operations in the Node.js MongoDB driver.

### (未) Aggregation

### (未) Streams

### (未) GridFS Support

### 日志

The driver lets you log at 3 different levels. These are **debug**, **info** and **error**. 默认日志级别是error。You can change the level, only allow specific classes to log and provide your own logger implementation. Let’s look at how we control the log level.

#### 设置日志级别

Setting the log level is pretty easy. Let’s look at example of adjusting it for our application only logging the Db class.

    var MongoClient = require('mongodb').MongoClient
      , Logger = require('mongodb').Logger
      , assert = require('assert');
    var url = 'mongodb://localhost:27017/myproject';
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");

      // Set debug level
      Logger.setLevel('debug');

      // Insert a single document
      db.command({ismaster:true}, function(err, d) {
        assert.equal(null, err);
        db.close();
      });
    });

Setting the level is as easy as calling the method setLevel with the string value debug, info or error. Log level is set globally.

#### Filtering On specific classes

Say you are only interested in logging a specific class. You can tell the Logger to only log specific class names. Let’s take an example Where we only log the Db class.

    var MongoClient = require('mongodb').MongoClient
      , Logger = require('mongodb').Logger
      , assert = require('assert');
    var url = 'mongodb://localhost:27017/myproject';
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");

      // Set debug level
      Logger.setLevel('debug');
      Logger.filter('class', ['Db']);

      // Insert a single document
      db.command({ismaster:true}, function(err, d) {
        assert.equal(null, err);
        db.close();
      });
    });

This will only log statements on the `Db` class. The available classes in the driver are.

- `Db`: The Db instance log statements
- `Server`: A server instance (either standalone, a mongos or replicaset member)
- `ReplSet`: Replicaset related log statements
- `Mongos`: Mongos related log statements
- `Cursor`: Cursor log statements
- `Pool`: Connection Pool specific log statements
- `Connection`: Singular connection specific log statements
- `Ping`: Replicaset ping inquiry log statements

You can add your own classes to the logger if you wish by creating your own logger instances. Let’s look at a simple example on how to add our custom class to the Logger.

    var Logger = require('mongodb').Logger
      , assert = require('assert');

    var A = function() {
      var logger = Logger('A', options);
      this.do = function() {
        if(logger.isInfo()) logger.info('logging A', {});
      }
    }

    // Execute A
    var a = new A();
    a.do();

Pretty simple and straightforward.

#### Custom logger

Let’s say you don’t want the log statements to go to `console.log` but want to send them to a new location or maybe transform them before you send them on. Let’s define our custom logger.

    var MongoClient = require('mongodb').MongoClient
      , Logger = require('mongodb').Logger
      , assert = require('assert');
    var url = 'mongodb://localhost:27017/myproject';
    MongoClient.connect(url, function(err, db) {
      assert.equal(null, err);
      console.log("Connected correctly to server");

      // Set debug level
      Logger.setLevel('debug');

      // Set our own logger
      Logger.setCurrentLogger(function(msg, context) {
        console.log(msg, context);
      });

      // Insert a single document
      db.command({ismaster:true}, function(err, d) {
        assert.equal(null, err);
        db.close();
      });
    });

That wraps up the Logging support in the driver.



