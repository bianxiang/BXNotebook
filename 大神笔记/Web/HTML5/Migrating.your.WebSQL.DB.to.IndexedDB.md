# Migrating your WebSQL DB to IndexedDB

http://www.html5rocks.com/en/tutorials/webdatabase/websql-indexeddb/

[toc]

## 介绍

### WebSQL

一个货真价实的关系型数据库（**SQLite**）。
Lock can happen on databases, tables, or rows on 'readwrite' transactions. Transaction creation is explicit. 除非提交，否则默认回滚。

缺点：

- 规范已被废止
- 不是对象驱动的。需要将Javascript对象转换为关系

### IndexedDB

- 快速索引和搜索对象。
- Works in asynchronous mode with moderately granular locking per transaction.
- 查询使用Cursor APIs, Key Range APIs, and Application Code
- Lock can happen on database 'versionchange' transaction, on an objectStore 'readonly' and 'readwrite' transactions.
- Transaction creation is explicit. Default is to commit unless we call abort or there is an error that is not caught.


## 初始化数据库

### WebSQL：创建数据库

要创建表，在事务中调用`CREATE TABLE`。We have defined a function that create a table in the body onload event. If the table doesn't already exist, a table is created. In our case, let's have 2MB of storage allocated to our todo list.

```js
var db = openDatabase('todos1', '1', 'todo list example db', 2 * 1024 * 1024);
```

### IndexedDB：创建数据库

处理浏览器差异：
```js
window.indexedDB = window.indexedDB || window.webkitIndexedDB || window.mozIndexedDB || window.msIndexedDB;

// Handle the prefix of Chrome to IDBTransaction/IDBKeyRange.
if ('webkitIndexedDB' in window) {
  window.IDBTransaction = window.webkitIDBTransaction;
  window.IDBKeyRange = window.webkitIDBKeyRange;
}

indexedDB.db = null;
// Hook up the errors to the console so we could see it.
// In the future, we need to push these messages to the user.
indexedDB.onerror = function(e) {
  console.log(e);
};
```

### 创建表

在WebSQL中创建表：

```js
if (database) {
  database.transaction(function(tx) {
    tx.executeSql("CREATE TABLE IF NOT EXISTS tasks (id REAL UNIQUE, text TEXT)", []);
  });
}
```

在IndexedDB中，在一个'SetVersion'事务中创建一个object store。要修改数据库结构，只能在`SetVersion`中进行，如创建、删除object stores，创建、删除索引。Object stores contain your JavaScript objects and you can reach your data by key or by setting indexes. A call to `setVersion` returns an `IDBRequest` object where we can attach our callbacks. When successful, we start to create our object stores.

```js
indexedDB.open = function() {
  var v = "2.0 beta"; // yes! you can put strings in the version not just numbers
  var request = indexedDB.open("todos", v);

  request.onupgradeneeded = function(e) {
    var db = request.result;
    var store = db.createObjectStore("todo", {keyPath: "timeStamp"});
  };

  request.onsuccess = function(e) {
    todoDB.indexedDB.db = e.target.result;
    todoDB.indexedDB.getAllTodoItems();
  };

  request.onfailure = todoDB.indexedDB.onerror;
};
```

Object Stores are created with a single call to `createObjectStore()`. The method takes a name of the store and an parameter object. The parameter object is very important as it lets you define important optional properties. In our case, we define a `keyPath` that is the property that makes an individual object in the store unique. That property in this example is "timeStamp".{{相当于主键列}} "timeStamp" must be present on every object that is stored in the objectStore.

> Note: according to latest IndexedDB spec: http://dvcs.w3.org/hg/IndexedDB/raw-file/tip/Overview.html `setVersion()` will be taken out. Thus, our code snippets here is going to change and the version setting will be part of the `open()` function of the database.

## 添加数据

### WebSQL：添加数据

```js
function addTodo() {
  var todo = document.getElementById("todo");
  var task = {
    "id": new Date().getTime(),
    "text": todo.value
  };

  database.transaction(function(tx) {
    tx.executeSql('INSERT INTO tasks (id, text) values (?, ?)', [task.id, task.text]);
  });

  todo.value = ''; // Clean for the next todo.
  showAll();
}
```

### IndexedDB：添加数据

We first get a quick reference to the database object `todoDB.indexedDB.db`, initiate a 'readwrite' transaction and get a reference to our object store. 有三种类型的事务：

- 'readwrite' - Allows records contained in object stores to be added, read, modified, and removed.
- 'readonly' - Allows records contained in object stores to be read.
- 'versionchange' - Used to create or update object store and indexes.

Now that the application has access to the object store, we can issue a simple `put` command with a basic JSON object. Notice that there is a timeStamp property. That is our unique key for the object and is used as the "keyPath". When the call to put is successful, our `onsuccess` event is triggered, and we are able to render the contents on the screen.

```js
indexedDB.addTodo = function() {
  var db = todoDB.indexedDB.db;
  var trans = db.transaction('todo', 'readwrite');
  var store = trans.objectStore('todo');

  var data = {
    "text": todoText, // todoText should be visible here
    "timeStamp": new Date().getTime()
  };

  var request = store.put(data);

  request.oncomplete = function(e) {
    todoDB.indexedDB.getAllTodoItems();
  };

  request.onerror = function(e) {
    console.log("Error Adding: ", e);
  };
};
```

## 查询数据

### WebSQL：查询数据

```java
function showAll() {
  var ourList = document.getElementById('ourList');
  ourList.innerHTML = '';

  database.transaction(function(tx) {
    tx.executeSql('SELECT * FROM tasks', [], function (tx, results) {
      var len = results.rows.length;
      var ul = document.createElement("ul");
      for (var i = 0; i < len; i++) {
        var item = results.rows.item(i);

        var li = document.createElement("li");
        var t = document.createTextNode(i + ") key: " + item.id +
                                        " => Todo text: " + item.text);
        // Have the ability to delete the item using data attributes and a link.
        var a = document.createElement("a");
        a.textContent = " [Delete]";
        a.dataset.key = item.id;
        a.dataset.value = item.text;

        a.addEventListener("click", function() {
          deleteTodo(this.dataset.key, this.dataset.val );
        }, false);

        li.appendChild(t);
        li.appendChild(a);
        ul.appendChild(li);
      }

      // Update the DOM only after we have ALL items in one element (performance baby...)
      ourList.appendChild(li);
    });
  });
}
```

### IndexedDB：查询数据

We open a transaction on our object store. This is set to 'readonly', because we only wish to retrieve data. Next, we open a cursor and iterate with it on our list of todos. All of these commands used in this sample are asynchronous and, as such, the data is not returned from inside the transaction.

```js
function showAll() {
  document.getElementById("ourList").innerHTML = "";

  var request = window.indexedDB.open("todos");
  request.onsuccess = function(event) {
    // Enumerate the entire object store.
    var ul = document.createElement("ul");
    var db = todoDB.indexedDB.db;
    var trans = db.transaction("todo", 'readonly');
    var request = trans.objectStore("todo").openCursor();

    request.onsuccess = function(event) {
      var cursor = request.result;

      // If cursor is null then we've completed the enumeration - so update the DOM
      if (cursor) {
        var li = document.createElement("div");
        li.textContent = "key: " + cursor.key + " => Todo text: " + cursor.value.text;
        ul.appendChild(li);
        cursor.continue();
      } else {
        document.getElementById("ourList").appendChild(ul);
      }
    }
  }
}
```

## 删除数据

### WebSQL：删除数据

```js
function deleteTodo(id, text) {
  if (confirm("Are you sure you want to Delete "+ text +"?")) {
    database.transaction(function(tx) {
      tx.executeSql('DELETE FROM tasks WHERE id=?', [id]);
    });
    showAll();
  }
}
```

### IndexedDB：删除数据

Start a transaction, reference the Object Store with your object in and issue a delete command with the unique ID of your object.

```js
indexedDB.deleteTodo = function(id, text) {
  if (confirm("Are you sure you want to Delete " + text + "?")) {
    var db = todoDB.indexedDB.db;
    var trans = db.transaction("todo", 'readwrite');
    var store = trans.objectStore("todo");

    var request = store.delete(id);

    request.onsuccess = function(e) {
      todoDB.indexedDB.getAllTodoItems();
    };

    request.onerror = function(e) {
      console.log("Error Adding: ", e);
    };
  }
};
```

## 完整代码

you can find the full code on github: https://github.com/greenido/WebSQL-to-IndexedDB-example and here is a [live example](http://ido-green.appspot.com/WebSQL-IndexedDB-example/main.html).

















