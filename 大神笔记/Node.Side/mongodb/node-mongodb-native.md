node-mongodb-native

[官网链接](https://github.com/mongodb/node-mongodb-native)

安装

	npm install mongodb

# README

## 介绍

This is a node.js driver for MongoDB. It's a port (or close to a port) of the library for ruby at [http://github.com/mongodb/mongo-ruby-driver/](http://github.com/mongodb/mongo-ruby-driver/ "http://github.com/mongodb/mongo-ruby-driver/").

	var MongoClient = require('mongodb').MongoClient
		, format = require('util').format;
	
	MongoClient.connect('mongodb://127.0.0.1:27017/test', function(err, db) {
		if(err) throw err;
	
	    var collection = db.collection('test_insert');
	    collection.insert({a:2}, function(err, docs) {
			collection.count(function(err, count) {
	        	console.log(format("count = %s", count));
	    	});
	
	      	// Locate all the entries using find
	      	collection.find().toArray(function(err, results) {
	        	console.dir(results);
	        	// Let's close the db
	        	db.close();
	      	});
	    });
	})

## 数据类型

如何读取保存JSON没有的、MongoDb类型：[ObjectID](http://www.mongodb.org/display/DOCS/Object+IDs), Long, Binary, [Timestamp](http://www.mongodb.org/display/DOCS/Timestamp+data+type), [DBRef](http://www.mongodb.org/display/DOCS/Database+References#DatabaseReferences-DBRef), Code。

特别的，每个文档都有一个唯一性的`_id`，可以是任何类型。但默认是12-byte ObjectID。ObjectID可以表示成24个数字的十六进制字符串，但在放入数据库前要前转换成ObjectID。例如：

	// Get the objectID type
	var ObjectID = require('mongodb').ObjectID;
	
	var idString = '4e4e1638c85e808431000003';
	collection.findOne({_id: new ObjectID(idString)}, console.log)  // ok
	collection.findOne({_id: idString}, console.log)  // 错! callback gets undefined


下面是Javascript中没有的BSON基本类型的构造器：

	// Fetch the library
	var mongo = require('mongodb');
	// Create new instances of BSON types
	new mongo.Long(numberString)
	new mongo.ObjectID(hexString)
	new mongo.Timestamp()  // the actual unique number is generated on insert.
	new mongo.DBRef(collectionName, id, dbName)
	new mongo.Binary(buffer)  // takes a string or Buffer
	new mongo.Code(code, [context])
	new mongo.Symbol(string)
	new mongo.MinKey()
	new mongo.MaxKey()
	new mongo.Double(number)	// Force double storage


### The C/C++ bson parser/serializer

If you are running a version of this library has the C/C++ parser compiled, to enable the driver to use the C/C++ bson parser pass it the option native_parser:true like below

	// using native_parser:
	MongoClient.connect('mongodb://127.0.0.1:27017/test'
		, {db: {native_parser: true}}, function(err, db) {})


The C++ parser uses the js objects both for serialization and deserialization.

## Replicasets

For more information about how to connect to a replicaset have a look at the extensive documentation [Documentation](http://mongodb.github.com/node-mongodb-native/)

### Primary Key Factories

Defining your own primary key factory allows you to generate your own series of id's
(this could f.ex be to use something like ISBN numbers). The generated the id needs to be a 12 byte long "string".

Simple example below

	var MongoClient = require('mongodb').MongoClient
	, format = require('util').format;    
	
	// Custom factory (need to provide a 12 byte array);
	CustomPKFactory = function() {}
	CustomPKFactory.prototype = new Object();
	CustomPKFactory.createPk = function() {
	return new ObjectID("aaaaaaaaaaaa");
	}
	
	MongoClient.connect('mongodb://127.0.0.1:27017/test', {'pkFactory':CustomPKFactory}, function(err, db) {
	if(err) throw err;
	
	db.dropDatabase(function(err, done) {
	  db.createCollection('test_custom_key', function(err, collection) {
	    collection.insert({'a':1}, function(err, docs) {
	      collection.find({'_id':new ObjectID("aaaaaaaaaaaa")}).toArray(function(err, items) {
	        console.dir(items);
	        // Let's close the db
	        db.close();
	      });
	    });
	  });
	});
	});

## Documentation

If this document doesn't answer your questions, see the source of
[Collection](https://github.com/mongodb/node-mongodb-native/blob/master/lib/mongodb/collection.js)
or [Cursor](https://github.com/mongodb/node-mongodb-native/blob/master/lib/mongodb/cursor.js),
or the documentation at MongoDB for query and update formats.

### 查询

find方法实际是一个创建Cursor对象的工厂方法。A Cursor lazily uses the connection the first time you call `nextObject`, `each`, or `toArray`.

The basic operation on a cursor is the `nextObject` method
that fetches the next matching document from the database. The convenience
methods `each` and `toArray` call `nextObject` until the cursor is exhausted.

签名：

	var cursor = collection.find(query, [fields], options);
	cursor.sort(fields).limit(n).skip(m).
	
	cursor.nextObject(function(err, doc) {});
	cursor.each(function(err, doc) {});
	cursor.toArray(function(err, docs) {});
	
	cursor.rewind()  // reset the cursor to its initial state.

cursor很多方法可以链式调用。这些方法也可以通过`find`方法的选项表达：

  * `.limit(n).skip(m)` 控制分页
  * `.sort(fields)` 排序。几个不同的签名：
  * `.sort({field1: -1, field2: 1})` field1降序，field2升序
  * `.sort([['field1', 'desc'], ['field2', 'asc']])` 同上
  * `.sort([['field1', 'desc'], 'field2'])` 同上
  * `.sort('field1')` field1升序

`find`的其他选项：

* `fields` 字段投影。
* `tailable` if true, makes the cursor [tailable](http://www.mongodb.org/display/DOCS/Tailable+Cursors).
* `batchSize` The number of the subset of results to request the database
to return for every request. 应该待遇1否则数据库会自动关闭游标。The batch size can be set to 1
with `batchSize(n, function(err){})` after performing the initial query to the database.
* `hint` See [Optimization: hint](http://www.mongodb.org/display/DOCS/Optimization#Optimization-Hint).
* `explain` turns this into an explain query. You can also call
`explain()` on any cursor to fetch the explanation.
* `snapshot` prevents documents that are updated while the query is active
from being returned multiple times. See more
[details about query snapshots](http://www.mongodb.org/display/DOCS/How+to+do+Snapshotted+Queries+in+the+Mongo+Database).
* `timeout` if false, asks MongoDb not to time out this cursor after an
inactivity period.

更多信息参见[MongoDB section on querying](http://www.mongodb.org/display/DOCS/Querying).

	var MongoClient = require('mongodb').MongoClient
	, format = require('util').format;    
	
	MongoClient.connect('mongodb://127.0.0.1:27017/test', function(err, db) {
	if(err) throw err;
	
	var collection = db.collection('test')
	  	.find({}).limit(10)
	  	.toArray(function(err, docs) {
	    	console.dir(docs);
		});
	});

### 插入

签名：

	collection.insert(docs, options, [callback]);


其中，`docs`可以是一个文档或文档数组。

可用选项：

* `safe:true` 如果要回调方法必须设置。否则回调方法会被立即调用。

See also: [MongoDB docs for insert](http://www.mongodb.org/display/DOCS/Inserting).


	var MongoClient = require('mongodb').MongoClient
		, format = require('util').format;    
	
	MongoClient.connect('mongodb://127.0.0.1:27017/test', function(err, db) {
		if(err) throw err;
		
		db.collection('test').insert({hello: 'world'}, {w:1}, function(err, objects) {
		  	if (err) console.warn(err.message);
		 	if (err && err.message.indexOf('E11000 ') !== -1) {
		   		// this _id was already inserted in the database
		  	}
		});
	});

### 更新：更新或插入(upsert)

更新操作更新匹配的第一个文档（如果指定了`multi:true`，则更新所有匹配）。
If `safe:true`, `upsert` is not set, and no documents match, your callback will return 0 documents updated.

See the [MongoDB docs](http://www.mongodb.org/display/DOCS/Updating) for
the modifier (`$inc`, `$set`, `$push`, etc.) formats.

签名：

	collection.update(criteria, objNew, options, [callback]);

有用的选项：

* `safe:true` 如果想要回调，一定要设置
* `upsert:true` 如果匹配文档，则插入

Example for `update`:

	var MongoClient = require('mongodb').MongoClient
		, format = require('util').format;    
	
	MongoClient.connect('mongodb://127.0.0.1:27017/test',
		function(err, db) {
		if(err) throw err;
		
		db.collection('test').update({hi: 'here'}, {$set: {hi: 'there'}}, {w:1}, function(err) {
			if (err) console.warn(err.message);
			else console.log('successfully updated');
		});
	});

### Find and modify

`findAndModify` is like `update`, but it also gives the updated document to
your callback. `findAndModify`一次只能更新一个文档。

签名：

	collection.findAndModify(query, sort, update, options, callback)

如果有多个文件端匹配，用`sort`参数排序。它的格式与游标的sort一样。

See the
[MongoDB docs for findAndModify](http://www.mongodb.org/display/DOCS/findAndModify+Command)
for more details.

选项：

* `remove:true` 返回前删除对象
* `new:true` 返回修改后的对象而非之前的的。删除时不使用该选项。
* `upsert:true` 如果文档不存在，自动插入

例子

	var MongoClient = require('mongodb').MongoClient
	, format = require('util').format;    
	
	MongoClient.connect('mongodb://127.0.0.1:27017/test', function(err, db) {
		if(err) throw err;
		db.collection('test').findAndModify({hello: 'world'}, [['_id','asc']], {$set: {hi: 'there'}}, {}, 
			function(err, object) {
			if (err) console.warn(err.message);
			else console.dir(object);  // undefined if no matching object exists.
		});
	});

### Save

The `save` method is a shorthand for upsert if the document contains an
`_id`, or an insert if there is no `_id`.















