
	https://www.ibm.com/developerworks/cn/web/1108_zhaifeng_websqldb/

[toc]

规范中所使用的 SQL 语言为 SQLite 3.6.19。

本文将介绍 Web SQL Database 规范中定义的三个核心方法：

- openDatabase：这个方法使用现有数据库或新建数据库，来创建数据库对象
- transaction：这个方法允许我们根据情况控制事务提交或回滚
- executeSql：这个方法用于执行真实的 SQL 查询

## 数据库API

### 1.Database

每个域都有一组相关的数据库，每个数据库有名字和当前的版本号。这套 API 并不提供遍历或删除域中的某个数据库的功能。每个数据库一时间只能有一个版本号，不能一时间拥有多个版本号，版本号用来保护数据库不被写入脏数据。

清单 1.Database API

    [Supplemental, NoInterfaceObject]
    interface WindowDatabase {
    	Database openDatabase(in DOMString name,
        	in DOMString version,
            in DOMString displayName,
            in unsigned long estimatedSize,
            in optional DatabaseCallback creationCallback);
    };
    Window implements WindowDatabase;

    [Supplemental, NoInterfaceObject]
    interface WorkerUtilsDatabase {
    	Database openDatabase(in DOMString name,
        	in DOMString version,
            in DOMString displayName,
            in unsigned long estimatedSize,
            in optional DatabaseCallback creationCallback);
        DatabaseSync openDatabaseSync(in DOMString name,
        	in DOMString version,
            in DOMString displayName,
            in unsigned long estimatedSize,
            in optional DatabaseCallback creationCallback);
    };
    WorkerUtils implements WorkerUtilsDatabase;

    [Callback=FunctionOnly, NoInterfaceObject]
    interface DatabaseCallback {
	    void handleEvent(in Database database);
    };

接口 `Window`、`WorkerUtils` 的 `openDatabase()` 方法和接口 `WorkerUtils` 的 `openDatabaseSync()` 方法接受以下参数：一个数据库名字（name），一个数据库版本号（version），一个显示名字（displayName），数据库将要保存数据的大小（estimatedSize，以字节为单位 )，一个可选的回调函数（createionCallback，如果数据库没有被创建，这个函数将会被调用）。如果提供了回调函数，回调函数用以调用 `changeVersion()` 函数（见下面一节），不管给定什么样的版本号，回调函数将把数据库的版本号设置为空；如果没有提供回调函数，则以给定的版本号创建数据库。

包括空字符串在内的所有字符串都可以作为有效地数据库名称，数据库名称区分大小写，且可以比较。

### 2.异步数据库 API

清单 2. 异步数据库 API

    interface Database {
        void transaction(in SQLTransactionCallback callback,
            in optional SQLTransactionErrorCallback errorCallback,
            in optional SQLVoidCallback successCallback);

        void readTransaction(in SQLTransactionCallback callback,
            in optional SQLTransactionErrorCallback errorCallback,
            in optional SQLVoidCallback successCallback);

        readonly attribute DOMString version;

        void changeVersion(in DOMString oldVersion,
            in DOMString newVersion,
            in optional SQLTransactionCallback callback,
            in optional SQLTransactionErrorCallback errorCallback,
            in optional SQLVoidCallback successCallback);
    };

    [Callback=FunctionOnly, NoInterfaceObject]
    interface SQLVoidCallback {
        void handleEvent();
    };

    [Callback=FunctionOnly, NoInterfaceObject]
    interface SQLTransactionCallback {
        void handleEvent(in SQLTransaction transaction);
    };

    [Callback=FunctionOnly, NoInterfaceObject]
    interface SQLTransactionErrorCallback {
        void handleEvent(in SQLError error);
    };

方法 `transaction()` 和 `readTransaction()` 有一个到三个参数，后两个参数为可选的，分别表示错误回调函数和成功回调函数。`transaction()` 方法为 read/write 模式，`readTransaction()` 方法为只读模式；获取属性 version 必须返回数据库的当前版本号；方法 `changeVersion` 允许脚本自动地检查版本号，并修改它。

#### 2.1 执行 SQL 语句

清单 3. 执行 SQL 语句 API

	typedef sequence<any> ObjectArray;

	interface SQLTransaction {
        void executeSql(in DOMString sqlStatement,
            in optional ObjectArray arguments,
            in optional SQLStatementCallback callback,
            in optional SQLStatementErrorCallback errorCallback);
    };

    [Callback=FunctionOnly, NoInterfaceObject]
    interface SQLStatementCallback {
        void handleEvent(in SQLTransaction transaction,
            in SQLResultSet resultSet);
    };

    [Callback=FunctionOnly, NoInterfaceObject]
    interface SQLStatementErrorCallback {
        boolean handleEvent(in SQLTransaction transaction,
        in SQLError error);
    };

这个函数具有四个参数：表示查询的字符串（sqlStatement）；插入到查询语句中问号所在处的字符串数据（arguments)；一个可选的成功时执行函数（callback）；一个可选的失败时执行函数（errorCallback）。

### 3.数据库查询结果（Database Query Result）

清单 4. 查询结果集 API

    interface SQLResultSet {
        readonly attribute long insertId;
        readonly attribute long rowsAffected;
        readonly attribute SQLResultSetRowList rows;
    };

如果插入数据库一行数据，`insertId` 代表这个行号；如果**插入多行数据**，`insertId` 代表插入数据**的最后一行**的行号。

SQL 语句执行后改变的行数用 `rowsAffected` 表示，如果 SQL 语句没有改变任何行，则 `rowsAffected` 为 0，对于“SELECT”语句，rowsAffected 就为 0.

`rows` 为一个 `SQLResultSetRowList` 对象，代表数据库按顺序返回的行。如果没有返回任何行，则这个对象为空。

## 一个入门的例子详解

本例将完整地演示 Web SQL Database API 的使用，建立数据库、建立表格、插入数据、查询数据、将查询结果显示。这个例子只能在最新版本的 Chrome、Safari 或 Opera 浏览器中产生输出结果。

清单 5. 例子源码

	<!DOCTYPE HTML>
	<html>
	<head>
	<script type="text/javascript">
	var db = openDatabase('mydb', '1.0', 'Test DB', 2 * 1024 * 1024);
	var msg;
	db.transaction(function (tx) {
	   tx.executeSql('CREATE TABLE IF NOT EXISTS LOGS (id unique, log)');
	   tx.executeSql('INSERT INTO LOGS (id, log) VALUES (1, "foobar")');
		tx.executeSql('INSERT INTO LOGS (id, log) VALUES (2, "logmsg")');
		msg = '<p>Log message created and row inserted.</p>';
		document.querySelector('#status').innerHTML = msg;
	 });

	db.transaction(function (tx) {
		tx.executeSql('SELECT * FROM LOGS', [], function (tx, results) {
			var len = results.rows.length, i;
			msg = "<p>Found rows: " + len + "</p>";
			document.querySelector('#status').innerHTML +=   msg;
			for (i = 0; i < len; i++){
			   msg = "<p><b>" + results.rows.item(i).log + "</b></p>";
			   document.querySelector('#status').innerHTML += msg;
			}
	    }, null);
	});
	</script>
    </head>
     <body>
	 <div id="status" name="status">Status Message</div>
	 </body>
	 </html>

第五行的 `var db = openDatabase('mydb', '1.0', 'Test DB', 2 * 1024 * 1024);`建立一个名称为 mydb 的数据库，它的版本为 1.0，描述信息为 Test DB，大小为 2M 字节。openDatabase 方法打开一个已经存在的数据库，如果数据库不存在则创建数据库。

最后一个参数创建回调函数，在创建数据库的时候调用，但即使没有这个参数，一样可以运行时创建数据库。

在插入新记录时，我们还可以传递动态值，如：

    var db = openDatabase('mydb', '1.0', 'Test DB', 2 * 1024 * 1024);
    db.transaction(function (tx) {
    	tx.executeSql('CREATE TABLE IF NOT EXISTS LOGS (id unique, log)');
        tx.executeSql('INSERT INTO LOGS (id, log) VALUES (?, ?'), [e_id, e_log];
    });

## 链接

规范： http://dev.w3.org/html5/webdatabase/#introduction








