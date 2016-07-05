[toc]

## 快速入门

Next install Mongoose from the command line using npm:

	$ npm install mongoose

第一步

	// getting-started.js
    var mongoose = require('mongoose');
    mongoose.connect('mongodb://localhost/test');

We now need to get notified if we connect successfully or if a connection error occurs:

    var db = mongoose.connection;
    db.on('error', console.error.bind(console, 'connection error:'));
    db.once('open', function (callback) {
      // yay!
    });

Once our connection opens, our callback will be called. For brevity, let's assume that all following code is within this callback.

With Mongoose, everything is derived from a Schema.

    var kittySchema = mongoose.Schema({
        name: String
    })

定义了一个属性`name`，字符串类型。The next step is compiling our schema into a Model.

	var Kitten = mongoose.model('Kitten', kittySchema)

Let's create a kitten document representing the little guy we just met on the sidewalk outside:

    var silence = new Kitten({ name: 'Silence' })
    console.log(silence.name) // 'Silence'

Kittens can meow, so let's take a look at how to add "speak" functionality to our documents:

    // NOTE: methods must be added to the schema before compiling it with mongoose.model()
    kittySchema.methods.speak = function () {
      var greeting = this.name
        ? "Meow name is " + this.name
        : "I don't have a name"
      console.log(greeting);
    }

    var Kitten = mongoose.model('Kitten', kittySchema)

Functions added to the `methods` property of a schema get compiled into the Model prototype and exposed on each document instance:

    var fluffy = new Kitten({ name: 'fluffy' });
    fluffy.speak() // "Meow name is fluffy"

We have talking kittens! But we still haven't saved anything to MongoDB. Each document can be saved to the database by calling its `save` method. The first argument to the callback will be an error if any occured.

    fluffy.save(function (err, fluffy) {
      if (err) return console.error(err);
      fluffy.speak();
    });

Say time goes by and we want to display all the kittens we've seen. We can access all of the kitten documents through our Kitten model.

    Kitten.find(function (err, kittens) {
      if (err) return console.error(err);
      console.log(kittens)
    })

We just logged all of the kittens in our db to the console. If we want to filter our kittens by name, Mongoose supports MongoDBs rich querying syntax.

	Kitten.find({ name: /^Fluff/ }, callback)

This performs a search for all documents with a name property that begins with "Fluff" and returns the results to the callback.

## Schemas

http://mongoosejs.com/docs/guide.html

### 定义schema

Mongoose中一切从Schema开始。每个schema映射到一个MongoDB集合，定义集合中文档的样式。

	var mongoose = require('mongoose');
	var Schema = mongoose.Schema;
	var blogSchema = new Schema({
	  title:  String,
	  author: String,
	  body:   String,
	  comments: [{ body: String, date: Date }],
	  date: { type: Date, default: Date.now },
	  hidden: Boolean,
	  meta: {
	    votes: Number,
	    favs:  Number
	  }
	});

If you want to add additional keys later, use the `Schema#add` method.

blogSchema中的每个键定义了文档中的一个属性，属性值会被类型转换到指定的`SchemaType`。例如`title`会被转换为`String`（`SchemaType`的一种）。可以嵌套对象，如`meta`属性。

可用的`SchemaType`有：String、Number、Date、Buffer、Boolean、Mixed、ObjectId、Array。

Schema不仅定义文档结构，也定义文档的实例方法，静态Model方法，索引，文档生命周期钩子（称为中间件）。

### 创建模型

要使用schema定义，需要将blogSchema转换为一个Model。

	var Blog = mongoose.model('Blog', blogSchema);
	// ready to go!

### 实例方法

`Model`的实例是文档。文档有很多内建的实例方法。也可以定义自己的文档实例方法。

	// define a schema
	var animalSchema = new Schema({ name: String, type: String });
	// assign a function to the "methods" object of our animalSchema
	animalSchema.methods.findSimilarTypes = function (cb) {
	  return this.model('Animal').find({ type: this.type }, cb);
	}

现在所有的animal对象都将有`findSimilarTypes`方法：

	var Animal = mongoose.model('Animal', animalSchema);
	var dog = new Animal({ type: 'dog' });
	dog.findSimilarTypes(function (err, dogs) {
	  console.log(dogs); // woof
	});

> 覆盖默认的mongoose文档方法可能导致意外的结果。

### 静态

向Model添加静态方法：

	// assign a function to the "statics" object of our animalSchema
	animalSchema.statics.findByName = function (name, cb) {
	  this.find({ name: new RegExp(name, 'i') }, cb);
	}
	var Animal = mongoose.model('Animal', animalSchema);
	Animal.findByName('fido', function (err, animals) {
	  console.log(animals);
	});

### 索引

MongoDB supports secondary indexes. With mongoose, we define these indexes within our `Schema` at the **path** level or the `schema` level. 创建[compound indexes](http://www.mongodb.org/display/DOCS/Indexes#Indexes-CompoundKeys)需要在schema层面。

	var animalSchema = new Schema({
	  name: String,
	  type: String,
	  tags: { type: [String], index: true } // field level
	});
	animalSchema.index({ name: 1, type: -1 }); // schema level

应用启动后，Mongoose会自动调用`ensureIndex`。While nice for development, it is recommended this behavior be disabled in production since index creation can cause a significant performance impact. Disable the behavior by setting the `autoIndex` option of your `schema` to `false`.

	animalSchema.set('autoIndex', false);
	// or
	new Schema({..}, { autoIndex: false });

See also the `Model#ensureIndexes` method.

### Virtuals

Virtuals指不会被持久化的文档属性。getters用于格式化或组合字段，settings用于把单个值分解成多个值存储。

	// define a schema
	var personSchema = new Schema({
	  name: {
	    first: String,
	    last: String
	  }
	});
	// compile our model
	var Person = mongoose.model('Person', personSchema);
	// create a document
	var bad = new Person({
	    name: { first: 'Walter', last: 'White' }
	});

Suppose we want to log the full name of bad. We could do this manually like so:

	console.log(bad.name.first + ' ' + bad.name.last); // Walter White

或者可以定义一个virtual property getter：

	personSchema.virtual('name.full').get(function () {
	  return this.name.first + ' ' + this.name.last;
	});

Now, when we access our virtual "name.full" property, our getter function will be invoked and the value returned:

	console.log('%s is insane', bad.name.full); // Walter White is insane

注意，当记录被转化为对象或JSON时，virtuals默认不会包含进去。Pass `virtuals : true` to either `toObject()` or `to toJSON()` to have them returned.

It would also be nice to be able to set `this.name.first` and `this.name.last` by setting `this.name.full`. For example, if we wanted to change bad's `name.first` and `name.last` to 'Breaking' and 'Bad' respectively, it'd be nice to just:

	bad.name.full = 'Breaking Bad';

Mongoose lets you do this as well through its virtual property **setters**:

	personSchema.virtual('name.full').set(function (name) {
	  var split = name.split(' ');
	  this.name.first = split[0];
	  this.name.last = split[1];
	});
	...
	mad.name.full = 'Breaking Bad';
	console.log(mad.name.first); // Breaking
	console.log(mad.name.last);  // Bad

Virtual property setters are applied before other validation. So the example above would still work even if the first and last name fields were required.

Only non-virtual properties work as part of queries and for field selection.

### 选项

Schema有一些可配置的项，可以通过构造器或set设置：

	new Schema({..}, options);
	// or
	var schema = new Schema({..});
	schema.set(option, value);

可用选项：

	autoIndex
	capped
	collection
	id
	_id
	read
	safe
	shardKey
	strict
	toJSON
	toObject
	versionKey

#### option: `autoIndex`

应用启动时，Mongoose会为每个定义在Schema中得索引发送一条`ensureIndex`命令。从Mongoose v3开始，索引默认在后台创建。若想禁用自动创建，手工创建，设置`autoIndex`为`false`，and use the `ensureIndexes` method on your model.

	var schema = new Schema({..}, { autoIndex: false });
	var Clock = mongoose.model('Clock', schema);
	Clock.ensureIndexes(callback);

#### option: `bufferCommands`

当连接断掉时，若驱动的`autoReconnect`选项被禁用，且连接到得时一个单台的mongod（不是复制集），mongoose会缓冲命令，直到你手工重连。若要禁止缓存，设置改选项为false。

	var schema = new Schema({..}, { bufferCommands: false });

#### option: `capped`

Mongoose supports MongoDBs [capped](http://www.mongodb.org/display/DOCS/Capped+Collections) collections. To specify the underlying MongoDB collection be capped, set the capped option to the maximum size of the collection in [bytes](http://www.mongodb.org/display/DOCS/Capped+Collections#CappedCollections-size).

	new Schema({..}, { capped: 1024 });

The `capped` option may also be set to an object if you want to pass additional options like max or autoIndexId. In this case you must explicitly pass the size option which is required.

	new Schema({..}, { capped: { size: 1024, max: 1000, autoIndexId: true } });

#### option: collection

Mongoose by default produces a collection name by passing the model name to the `utils.toCollectionName` method. 这个方法会使用名字的复数。 Set this option if you need a different name for your collection.

	var dataSchema = new Schema({..}, { collection: 'data' });

#### option: id

Mongoose为schemas创建一个虚拟getter：id。它返回文档的`_id`字段，且会转为字符串类型（如ObjectId转换为十六进制字符串）。If you don't want an id getter added to your schema, you may disable it passing this option at schema construction time.

    // 默认行为
    var schema = new Schema({ name: String });
    var Page = mongoose.model('Page', schema);
    var p = new Page({ name: 'mongodb.org' });
    console.log(p.id); // '50341373e894ad16347efe01'

    // 禁用id
    var schema = new Schema({ name: String }, { id: false });
    var Page = mongoose.model('Page', schema);
    var p = new Page({ name: 'mongodb.org' });
    console.log(p.id); // undefined

#### option: _id

Mongoose assigns each of your schemas an _id field by default if one is not passed into the Schema constructor. The type assiged is an `ObjectId` to coincide with MongoDBs default behavior. 如果你不想让你的schema含有`_id`，可以通过改选项禁掉。

Pass this option during schema construction to prevent documents from getting an `_id` created by Mongoose (parent documents will still have an `_id` created by MongoDB when inserted). Passing the option later using `Schema.set('_id', false)` will not work. See issue #1512.

    // 默认行为
    var schema = new Schema({ name: String });
    var Page = mongoose.model('Page', schema);
    var p = new Page({ name: 'mongodb.org' });
    console.log(p); // { _id: '50341373e894ad16347efe01', name: 'mongodb.org' }

    // 禁用_id
    var schema = new Schema({ name: String }, { _id: false });

    // Don't set _id to false after schema construction as in
    // var schema = new Schema({ name: String });
    // schema.set('_id', false);

    var Page = mongoose.model('Page', schema);
    var p = new Page({ name: 'mongodb.org' });
    console.log(p); // { name: 'mongodb.org' }

    // MongoDB will create the _id when inserted
    p.save(function (err) {
      if (err) return handleError(err);
      Page.findById(p, function (err, doc) {
        if (err) return handleError(err);
        console.log(doc); // { name: 'mongodb.org', _id: '50341373e894ad16347efe12' }
      })
    })

Note that currently you must disable the _id

#### option: read

Allows setting `query#read` options at the schema level, providing us a way to apply default [ReadPreferences](http://docs.mongodb.org/manual/applications/replication/#replica-set-read-preference) to all queries derived from a model.

    var schema = new Schema({..}, { read: 'primary' });            // also aliased as 'p'
    var schema = new Schema({..}, { read: 'primaryPreferred' });   // aliased as 'pp'
    var schema = new Schema({..}, { read: 'secondary' });          // aliased as 's'
    var schema = new Schema({..}, { read: 'secondaryPreferred' }); // aliased as 'sp'
    var schema = new Schema({..}, { read: 'nearest' });            // aliased as 'n'

The alias of each pref is also permitted so instead of having to type out 'secondaryPreferred' and getting the spelling wrong, we can simply pass 'sp'.

The read option also allows us to specify tag sets. These tell the driver from which members of the replica-set it should attempt to read. Read more about tag sets here and here.

NOTE: you may also specify the driver read pref strategy option when connecting:

    // pings the replset members periodically to track network latency
    var options = { replset: { strategy: 'ping' }};
    mongoose.connect(uri, options);

    var schema = new Schema({..}, { read: ['nearest', { disk: 'ssd' }] });
    mongoose.model('JellyBean', schema);

#### option: safe

This option is passed to MongoDB with all operations and specifies if errors should be returned to our callbacks as well as tune write behavior.

    var safe = true;
    new Schema({ .. }, { safe: safe });

By default this is set to true for all schemas which guarentees that any occurring error gets passed back to our callback. By setting safe to something else like `{ j: 1, w: 2, wtimeout: 10000 }` we can guarantee the write was committed to the MongoDB journal (j: 1), at least 2 replicas (w: 2), and that the write will timeout if it takes longer than 10 seconds (wtimeout: 10000). Errors will still be passed to our callback.

NOTE: In 3.6.x, you also need to turn versioning off. In 3.7.x and above, versioning will automatically be disabled when safe is set to false

> NOTE: this setting overrides any setting specified by passing db options while creating a connection.

There are other write concerns like `{ w: "majority" }` too. See the MongoDB docs for more details.

    var safe = { w: "majority", wtimeout: 10000 };
    new Schema({ .. }, { safe: safe });

#### option: shardKey

The shardKey option is used when we have a [sharded MongoDB architecture](http://www.mongodb.org/display/DOCS/Sharding+Introduction). Each sharded collection is given a shard key which must be present in all insert/update operations. We just need to set this schema option to the same shard key and we’ll be all set.

	new Schema({ .. }, { shardKey: { tag: 1, name: 1 }})

Note that Mongoose does not send the shardcollection command for you. You must configure your shards yourself.

#### option: strict

`strict`选项（默认开启），意为，传入模型构造器的值，如果不在schema中，不会存入数据库。

    var thingSchema = new Schema({..})
    var Thing = mongoose.model('Thing', thingSchema);
    var thing = new Thing({ iAmNotInTheSchema: true });
    thing.save(); // iAmNotInTheSchema is not saved to the db

    // set to false..
    var thingSchema = new Schema({..}, { strict: false });
    var thing = new Thing({ iAmNotInTheSchema: true });
    thing.save(); // iAmNotInTheSchema is now saved to the db!!
    This also affects the use of doc.set() to set a property value.

    var thingSchema = new Schema({..})
    var Thing = mongoose.model('Thing', thingSchema);
    var thing = new Thing;
    thing.set('iAmNotInTheSchema', true);
    thing.save(); // iAmNotInTheSchema is not saved to the db

This value can be overridden at the model instance level by passing a second boolean argument:

    var Thing = mongoose.model('Thing');
    var thing = new Thing(doc, true);  // enables strict mode
    var thing = new Thing(doc, false); // disables strict mode

The strict option may also be set to `"throw"` which will cause errors to be produced instead of dropping the bad data.

> NOTE: do not set to false unless you have good reason.

> NOTE: in mongoose v2 the default was false.

> NOTE: Any key/val set on the instance that does not exist in your schema is always ignored, regardless of schema option.

    var thingSchema = new Schema({..})
    var Thing = mongoose.model('Thing', thingSchema);
    var thing = new Thing;
    thing.iAmNotInTheSchema = true;
    thing.save(); // iAmNotInTheSchema is never saved to the db

#### option: toJSON

Exactly the same as the `toObject` option but only applies when the documents toJSON method is called.

    var schema = new Schema({ name: String });
    schema.path('name').get(function (v) {
      return v + ' is my name';
    });
    schema.set('toJSON', { getters: true, virtuals: false });
    var M = mongoose.model('Person', schema);
    var m = new M({ name: 'Max Headroom' });
    console.log(m.toObject()); // { _id: 504e0cd7dd992d9be2f20b6f, name: 'Max Headroom' }
    console.log(m.toJSON()); // { _id: 504e0cd7dd992d9be2f20b6f, name: 'Max Headroom is my name' }
    // since we know toJSON is called whenever a js object is stringified:
    console.log(JSON.stringify(m)); // { "_id": "504e0cd7dd992d9be2f20b6f", "name": "Max Headroom is my name" }

To see all available toJSON/toObject options, read [this](http://mongoosejs.com/docs/api.html#document_Document-toObject).

#### option: toObject

Documents have a toObject method which converts the mongoose document into a plain javascript object. This method accepts a few options. Instead of applying these options on a per-document basis we may declare the options here and have it applied to all of this schemas documents by default.

    To have all virtuals show up in your console.log output, set the toObject option to `{ getters: true }`:

    var schema = new Schema({ name: String });
    schema.path('name').get(function (v) {
      return v + ' is my name';
    });
    schema.set('toObject', { getters: true });
    var M = mongoose.model('Person', schema);
    var m = new M({ name: 'Max Headroom' });
    console.log(m); // { _id: 504e0cd7dd992d9be2f20b6f, name: 'Max Headroom is my name' }

To see all available toObject options, read [this](http://mongoosejs.com/docs/api.html#document_Document-toObject).

#### option: versionKey

The versionKey is a property set on each document when first created by Mongoose. This keys value contains the internal revision of the document. The name of this document property is configurable. The default is `__v`. If this conflicts with your application you can configure as such:

    var schema = new Schema({ name: 'string' });
    var Thing = mongoose.model('Thing', schema);
    var thing = new Thing({ name: 'mongoose v3' });
    thing.save(); // { __v: 0, name: 'mongoose v3' }

    // customized versionKey
    new Schema({..}, { versionKey: '_somethingElse' })
    var Thing = mongoose.model('Thing', schema);
    var thing = new Thing({ name: 'mongoose v3' });
    thing.save(); // { _somethingElse: 0, name: 'mongoose v3' }

Document versioning can also be disabled by setting the `versionKey` to `false`. DO NOT disable versioning unless you know what you are doing.

    new Schema({..}, { versionKey: false });
    var Thing = mongoose.model('Thing', schema);
    var thing = new Thing({ name: 'no versioning please' });
    thing.save(); // { name: 'no versioning please' }

### Pluggable

Schemas are also [pluggable](http://mongoosejs.com/docs/plugins.html) which allows us to package up reusable features into [plugins](http://plugins.mongoosejs.com/) that can be shared with the community or just between your projects.

## （未）SchemaTypes

http://mongoosejs.com/docs/schematypes.html

## Models

http://mongoosejs.com/docs/models.html

Models are fancy constructors compiled from our Schema definitions. Instances of these models represent documents which can be saved and retreived from our database. All document creation and retreival from the database is handled by these models.

Compiling your first model:
	var schema = new mongoose.Schema({ name: 'string', size: 'string' });
	var Tank = mongoose.model('Tank', schema);

### 构建文档

文档是模型的实例。

创建文档并保存：

    var Tank = mongoose.model('Tank', yourSchema);
    var small = new Tank({ size: 'small' });
    small.save(function (err) {
      if (err) return handleError(err);
      // saved!
    })
    // or
    Tank.create({ size: 'small' }, function (err, small) {
      if (err) return handleError(err);
      // saved!
    })

注意，实际的创建和删除要等待模型使用的链接打开后发生。In this case we are using mongoose.model() so let's open the default mongoose connection:

	mongoose.connect('localhost', 'gettingstarted');

### 查询

Documents can be retreived using each models [find](http://mongoosejs.com/docs/api.html#model_Model.find), [findById](http://mongoosejs.com/docs/api.html#model_Model.findById), [findOne](http://mongoosejs.com/docs/api.html#model_Model.findOne), or [where](http://mongoosejs.com/docs/api.html#model_Model.where) static methods.

    Tank.find({ size: 'small' }).where('createdDate').gt(oneYearAgo).exec(callback);

### 删除

模型有一个静态的`remove`方法用于删除匹配的文档。

    Tank.remove({ size: 'large' }, function (err) {
      if (err) return handleError(err);
      // removed!
    });

### 更新

Each model has its own `update` method for modifying documents in the database without returning them to your application. See the [API](http://mongoosejs.com/docs/api.html#model_Model.update) docs for more detail.

If you want to update a single document in the db and return it to your application, use [findOneAndUpdate](http://mongoosejs.com/docs/api.html#model_Model.findOneAndUpdate) instead.

## Documents

http://mongoosejs.com/docs/documents.html

Mongoose documents represent a one-to-one mapping to documents as stored in MongoDB. Each document is an instance of its Model.

### Updating

There are a number of ways to update documents. We'll first look at a traditional approach using findById:

    Tank.findById(id, function (err, tank) {
      if (err) return handleError(err);
      tank.size = 'large';
      tank.save(function (err) {
        if (err) return handleError(err);
        res.send(tank);
      });
    });

This approach involves first retreiving the document from Mongo, then issuing an update command (triggered by calling save). However, if we don't need the document returned in our application and merely want to update a property in the database directly, `Model#update` is right for us:

	Tank.update({ _id: id }, { $set: { size: 'large' }}, callback);

If we do need the document returned in our application there is another, often better, option:

    Tank.findByIdAndUpdate(id, { $set: { size: 'large' }}, function (err, tank) {
      if (err) return handleError(err);
      res.send(tank);
    });

The findAndUpdate/Remove static methods all make a change to at most one document, and return it with just one call to the database. There are several variations on the findAndModify theme. Read the API docs for more detail. Note that findAndUpdate/Remove do not execute any hooks or validation before making the change in the database. If you need hooks and validation, first query for the document and then save it.

## Sub Docs

http://mongoosejs.com/docs/subdocs.html

Sub-documents are docs with schemas of their own which are elements of a parents document array:

    var childSchema = new Schema({ name: 'string' });
    var parentSchema = new Schema({
      children: [childSchema]
    })

Sub-documents enjoy all the same features as normal documents. The only difference is that they are not saved individually, they are saved whenever their top-level parent document is saved.

    var Parent = mongoose.model('Parent', parentSchema);
    var parent = new Parent({ children: [{ name: 'Matt' }, { name: 'Sarah' }] })
    parent.children[0].name = 'Matthew';
    parent.save(callback);

If an error occurs in a sub-documents' middleware, it is bubbled up to the `save()` callback of the parent, so error handling is a snap!

    childSchema.pre('save', function (next) {
      if ('invalid' == this.name) return next(new Error('#sadpanda'));
      next();
    });

    var parent = new Parent({ children: [{ name: 'invalid' }] });
    parent.save(function (err) {
      console.log(err.message) // #sadpanda
    })

### Finding a sub-document

Each document has an `_id`. DocumentArrays have a special `id` method for looking up a document by its _id.

	var doc = parent.children.id(id);

### Adding sub-docs

MongooseArray methods such as `push`, `unshift`, `addToSet`, and others cast arguments to their proper types transparently:

    var Parent = mongoose.model('Parent');
    var parent = new Parent;
    // create a comment
    parent.children.push({ name: 'Liesl' });
    var subdoc = parent.children[0];
    console.log(subdoc) // { _id: '501d86090d371bab2c0341c5', name: 'Liesl' }
    subdoc.isNew; // true

    parent.save(function (err) {
      if (err) return handleError(err)
      console.log('Success!');
    });

Sub-docs may also be created without adding them to the array by using the create method of MongooseArrays.

	var newdoc = parent.children.create({ name: 'Aaron' });

### Removing docs

Each sub-document has it's own remove method.

    var doc = parent.children.id(id).remove();
    parent.save(function (err) {
      if (err) return handleError(err);
      console.log('the sub-doc was removed')
    });

### Alternate declaration syntax

New in v3 If you don't need access to the sub-document **schema** instance, you may also declare sub-docs by simply passing an object literal:

    var parentSchema = new Schema({
      children: [{ name: 'string' }]
    })

## 查询

http://mongoosejs.com/docs/queries.html
