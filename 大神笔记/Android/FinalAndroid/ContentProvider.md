[toc]

## Content Provider

### 基础

Content Provider 是连接一个进程中的数据与另一个进程中运行的代码的标准接口。

一个 content provider 提供的数据集称为一个 **authority**，类似于数据库的库。一个数据集中可以有多张“表”。一个表有多个“列”。表中可以有多“行”数据。底层数据不一定是数据库。可以是任何数据源。

客户端访问提供者要通过一个内容 URI。内容 URI 标示提供者的部分数据。一个内容 URI 的形式是 `content://<authority>/<path>` 或 `content://<authority>/<path>/<id>`。分别表示一个表或具体一行记录。

通过内容 URI 的第一部分 `<authority>`，系统决定分发到哪个内容提供者。`<path>` 部分，由内容提供者自己解析，判定用户访问哪个“表”。

内容提供者可以设置权限，也可以不设。若设置了权限，访问者需要在安装时申请权限。提供者可以决定权限粒度。可以只设一个权限项，或读写分离，或按表设置权限等等。

客户端应用通过 `ContentResolver` 访问提供者。可以进行增删改查操作。`ContentResolver` 会自动处理跨进程通信。

### 访问者角度：使用提供者

例子，读写系统提供的 User Dictionary Provider。

> 实际项目，避免在UI线程中执行 `ContentResolver.query()` 等方法。可以使用后台线程，或 `CursorLoader` 机制。

#### 第一步申请权限

必须在 manifest 声明你需要的权限。
User Dictionary Provider 定义了权限 `android.permission.READ_USER_DICTIONARY`，表示读取权限。因此需要：

```xml
	<uses-permission android:name="android.permission.READ_USER_DICTIONARY">
```
如果要修改数据，还需要 `android.permission.WRITE_USER_DICTIONARY` 权限。

#### URI相关

Many providers allow you to access a single row in a table by appending an ID value to the end of the URI. For example, to retrieve a row whose `_ID` is 4 from user dictionary, you can use this content URI:

```java
Uri singleUri = ContentUris.withAppendedId(UserDictionary.Words.CONTENT_URI, 4);
```

You often use id values when you've retrieved a set of rows and then want to update or delete one of them.

Note: The `Uri` and `Uri.Builder` classes contain convenience methods for constructing well-formed `Uri` objects from strings. The `ContentUris` contains convenience methods for appending id values to a URI. The previous snippet uses `withAppendedId()` to append an id to the `UserDictionary` content URI.

#### 读取

例子，读取 User Dictionary Provider 的 word 表。根据用户输入的关键词，查询某个单词。用户不输入，返回全部单词。

查询条件分查询子句和参数。查询子句中出现的 `?` 占位符对应查询参数。

```java
// 投影，决定返回哪些列
String[] mProjection =
{
    UserDictionary.Words._ID, // _ID column name
    UserDictionary.Words.WORD, // word column name
    UserDictionary.Words.LOCALE // locale column name
};

// 查询子句
String mSelectionClause = null;

// 查询参数
String[] mSelectionArgs = {""};

// 从界面获取输入
mSearchString = mSearchWord.getText().toString();
// If the word is the empty string, gets everything
if (TextUtils.isEmpty(mSearchString)) {
    // 选择子句为 Null 表示返回所有
    mSelectionClause = null;
    mSelectionArgs[0] = "";
} else {
    // Constructs a selection clause that matches the word that the user entered.
    mSelectionClause = UserDictionary.Words.WORD + " = ?";
    // Moves the user's input string to the selection arguments.
    mSelectionArgs[0] = mSearchString;
}

// 查询，返回一个 Cursor 对象
mCursor = getContentResolver().query(
    UserDictionary.Words.CONTENT_URI, // words 表的内容URI
    mProjection,
    mSelectionClause,
    mSelectionArgs,
    mSortOrder);

// TODO query 方法可能抛出异常
// 返回的游标是可能为 null，表示错误。
if (null == mCursor) {
	// If the Cursor is empty, the provider found no matches
} else if (mCursor.getCount() < 1) {
} else {
    // Insert code here to do something with the results
}
```

#### 显示查询结果

`ContentResolver.query()` 总是返回一个 `Cursor`。

Some `Cursor` implementations automatically update the object when the provider's data changes, or trigger methods in an observer object when the `Cursor` changes, or both.

`Cursor.getCount()` 返回数据量。

查询发生内部错误，取决于提供者，`query` 可能返回 null，或抛异常。

Since a Cursor is a "list" of rows, a good way to display the contents of a Cursor is to link it to a `ListView` via a `SimpleCursorAdapter`.

```java
String[] mWordListColumns =
{
    UserDictionary.Words.WORD,
    UserDictionary.Words.LOCALE
};

// Defines a list of View IDs that will receive the Cursor columns for each row
int[] mWordListItems = { R.id.dictWord, R.id.locale};

// Creates a new SimpleCursorAdapter
mCursorAdapter = new SimpleCursorAdapter(
    getApplicationContext(), // The application's Context object
    R.layout.wordlistrow, // A layout in XML for one row in the ListView
    mCursor, // The result from the query
    mWordListColumns, // A string array of column names in the cursor
    mWordListItems, // An integer array of view IDs in the row layout
    0); // Flags (usually none are needed)

// Sets the adapter for the ListView
mWordList.setAdapter(mCursorAdapter);
```

Note: To back a `ListView` with a `Cursor`, the cursor must contain a column named `_ID`. Because of this, the query shown previously retrieves the `_ID` column for the "words" table, even though the `ListView` doesn't display it. This restriction also explains why most providers have a `_ID` column for each of their tables.

#### 手工遍历游标

```java
// Determine the column index of the column named "word"
int index = mCursor.getColumnIndex(UserDictionary.Words.WORD);
if (mCursor != null) {
    while (mCursor.moveToNext()) {
        // Gets the value from the column.
        newWord = mCursor.getString(index);
        // Insert code here to process the retrieved word.
        ...
        // end of while loop
    }
}
```

`Cursor` implementations contain several "get" methods for retrieving different types of data from the object. For example, the previous snippet uses `getString()`. They also have a `getType()` method that returns a value indicating the data type of the column.

#### 插入数据

调用 `ContentResolver.insert()` 方法。

```java
Uri mNewUri; // 用来接受表示新插入的记录的 URI
...

ContentValues mNewValues = new ContentValues();
mNewValues.put(UserDictionary.Words.APP_ID, "example.user");
mNewValues.put(UserDictionary.Words.LOCALE, "en_US");
mNewValues.put(UserDictionary.Words.WORD, "insert");
mNewValues.put(UserDictionary.Words.FREQUENCY, "100");

mNewUri = getContentResolver().insert(
    UserDictionary.Word.CONTENT_URI,
    mNewValues
);
```

If you don't want to specify a value at all, you can set a column to null using `ContentValues.putNull()`.

上面并未插入 `_ID` 列，因为它是自动维护的。提供者会为插入的每一行分配一个唯一的 `_ID`。

`insert()` 方法返回新插入的记录的 URI。这类它的结构是：
`content://user_dictionary/words/<id_value>`

To get the value of `_ID` from the returned Uri, call `ContentUris.parseId()`.

#### 更新数据

只需要把需要更新的列放入 `ContentValues` 对象。如果将把某列的值清空，设 `null`。

The following snippet changes all the rows whose locale has the language "en" to a have a locale of null. The return value is the number of rows that were updated:

```java
ContentValues mUpdateValues = new ContentValues();

String mSelectionClause = UserDictionary.Words.LOCALE +  "LIKE ?";
String[] mSelectionArgs = {"en_%"};

int mRowsUpdated = 0;
...
mUpdateValues.putNull(UserDictionary.Words.LOCALE);
mRowsUpdated = getContentResolver().update(
    UserDictionary.Words.CONTENT_URI,
    mUpdateValues
    mSelectionClause
    mSelectionArgs
);
```

#### 删除数据

The following snippet deletes rows whose appid matches "user". The method returns the number of deleted rows.

```java
String mSelectionClause = UserDictionary.Words.APP_ID + " LIKE ?";
String[] mSelectionArgs = {"user"};

int mRowsDeleted = 0;
...
// Deletes the words that match the selection criteria
mRowsDeleted = getContentResolver().delete(
    UserDictionary.Words.CONTENT_URI,
    mSelectionClause
    mSelectionArgs
);
```

#### Contract 类

Contract 类是辅助类。定义常量，用于内容 URI、类名、 intent actions等。

例如 User Dictionary Provider 的 Contract 类是 `UserDictionary`。The content URI for the "words" table is defined in the constant `UserDictionary.Words.CONTENT_URI`. The `UserDictionary.Words` class also contains column name constants, which are used in the example snippets in this guide. For example, a query projection can be defined as:

```java
String[] mProjection =
{
    UserDictionary.Words._ID,
    UserDictionary.Words.WORD,
    UserDictionary.Words.LOCALE
};
```

Another contract class is `ContactsContract` for the Contacts Provider. The reference documentation for this class includes example code snippets. One of its subclasses, `ContactsContract.Intents.Insert`, is a contract class that contains constants for intents and intent data.


### 数据类型

内容提供者可以提供多种数据类型。除了文本，还可以是：

- integer
- long integer (long)
- floating point
- long floating point (double)

另一种形式是 **Binary Large OBject (BLOB)**（implemented as a 64KB byte array）。

You can see the available data types by looking at the `Cursor` class "get" methods.

You can also determine the data type by calling `Cursor.getType()`.

Providers also maintain MIME data type information for each content URI they define. You can use the MIME type information to find out if your application can handle data that the provider offers, or to choose a type of handling based on the MIME type. You usually need the MIME type when you are working with a provider that contains complex data structures or files. For example, the `ContactsContract.Data` table in the Contacts Provider uses MIME types to label the type of contact data stored in each row. To get the MIME type corresponding to a content URI, call `ContentResolver.getType()`.

The section [MIME Type Reference](http://developer.android.com/guide/topics/providers/content-provider-basics.html#MIMETypeReference) describes the syntax of both standard and custom MIME types.

### 访问提供者的其他方式

- **批量访问**: You can create a batch of access calls with methods in the `ContentProviderOperation` class, and then apply them with `ContentResolver.applyBatch()`.
- **异步查询**: You should do queries in a separate thread. One way to do this is to use a `CursorLoader` object. The examples in the Loaders guide demonstrate how to do this.
- **Data access via intents**: Although you can't send an intent directly to a provider, you can send an intent to the provider's application, which is usually the best-equipped to modify the provider's data.

#### 批量访问

批量访问适用于插入大量数据，若向多个表插入数据，或将多个操作包含进一个事务。

To access a provider in "batch mode", you create an array of `ContentProviderOperation` objects and then dispatch them to a content provider with `ContentResolver.applyBatch()`. You pass the content provider's **authority** to this method, rather than a particular content URI. This allows each `ContentProviderOperation` object in the array to work against a different table. A call to `ContentResolver.applyBatch()` returns an array of results.

The description of the `ContactsContract.RawContacts` contract class includes a code snippet that demonstrates batch insertion. The Contact Manager sample application contains an example of batch access in its **ContactAdder.java** source file.

#### 通过 Intent 访问数据

Intents can provide indirect access to a content provider. You allow the user to access data in a provider even if your application doesn't have access permissions, either by getting a result intent back from an application that has permissions, or by activating an application that has permissions and letting the user do work in it.

**Getting access with temporary permissions**

You can access data in a content provider, even if you don't have the proper access permissions, by sending an intent to an application that does have the permissions and receiving back a result intent containing "URI" permissions. These are permissions for a specific content URI that last until the activity that receives them is finished. The application that has permanent permissions grants temporary permissions by setting a flag in the result intent:

Read permission: `FLAG_GRANT_READ_URI_PERMISSION`
Write permission: `FLAG_GRANT_WRITE_URI_PERMISSION`

Note: These flags don't give general read or write access to the provider whose authority is contained in the content URI. The access is only for the URI itself.

A provider defines URI permissions for content URIs in its manifest, using the `android:grantUriPermission` attribute of the `<provider>` element, as well as the `<grant-uri-permission>` child element of the `<provider>` element. The URI permissions mechanism is explained in more detail in the Security and Permissions guide, in the section "URI Permissions".

For example, you can retrieve data for a contact in the Contacts Provider, even if you don't have the `READ_CONTACTS` permission. You might want to do this in an application that sends e-greetings to a contact on his or her birthday. Instead of requesting `READ_CONTACTS`, which gives you access to all of the user's contacts and all of their information, you prefer to let the user control which contacts are used by your application. To do this, you use the following process:

1. Your application sends an intent containing the action `ACTION_PICK` and the "contacts" MIME type `CONTENT_ITEM_TYPE`, using the method `startActivityForResult()`.
2. Because this intent matches the intent filter for the People app's "selection" activity, the activity will come to the foreground.
3. In the selection activity, the user selects a contact to update. When this happens, the selection activity calls `setResult(resultcode, intent)` to set up a intent to give back to your application. The intent contains the content URI of the contact the user selected, and the "extras" flags `FLAG_GRANT_READ_URI_PERMISSION`. These flags grant URI permission to your app to read data for the contact pointed to by the content URI. The selection activity then calls `finish()` to return control to your application.
4. Your activity returns to the foreground, and the system calls your activity's `onActivityResult()` method. This method receives the result intent created by the selection activity in the People app.
5. With the content URI from the result intent, you can read the contact's data from the Contacts Provider, even though you didn't request permanent read access permission to the provider in your manifest. You can then get the contact's birthday information or his or her email address and then send the e-greeting.

**Using another application**

A simple way to allow the user to modify data to which you don't have access permissions is to activate an application that has permissions and let the user do the work there.

For example, the Calendar application accepts an `ACTION_INSERT` intent, which allows you to activate the application's insert UI. You can pass "extras" data in this intent, which the application uses to pre-populate the UI. Because recurring events have a complex syntax, the preferred way of inserting events into the Calendar Provider is to activate the Calendar app with an `ACTION_INSERT` and then let the user insert the event there.

### MIME Type Reference

Content providers can return standard MIME media types, or custom MIME type strings, or both.

MIME types have the format
`type/subtype`

For example, the well-known MIME type `text/html` has the text type and the html subtype. If the provider returns this type for a URI, it means that a query using that URI will return text containing HTML tags.

Custom MIME type strings, also called "vendor-specific" MIME types, have more complex type and subtype values. The type value is always `vnd.android.cursor.dir` for multiple rows, or `vnd.android.cursor.item` for a single row.

The subtype is provider-specific. The Android built-in providers usually have a simple subtype. For example, when the Contacts application creates a row for a telephone number, it sets the following MIME type in the row:
`vnd.android.cursor.item/phone_v2`

Notice that the subtype value is simply phone_v2.

Other provider developers may create their own pattern of subtypes based on the provider's authority and table names. For example, consider a provider that contains train timetables. The provider's authority is com.example.trains, and it contains the tables Line1, Line2, and Line3. In response to the content URI
content://com.example.trains/Line1

for table Line1, the provider returns the MIME type
vnd.android.cursor.dir/vnd.example.line1

In response to the content URI
content://com.example.trains/Line2/5

for row 5 in table Line2, the provider returns the MIME type
vnd.android.cursor.item/vnd.example.line2

Most content providers define contract class constants for the MIME types they use. The Contacts Provider contract class `ContactsContract.RawContacts`, for example, defines the constant `CONTENT_ITEM_TYPE` for the MIME type of a single raw contact row.

Content URIs for single rows are described in the section Content URIs.

### 设计我们自己的 Content Provider

为什么要用 Content Provider？

- 向其他应用提供复杂的数据或文件
- 允许用户拷贝复杂的数据到其他应用
- 想提供定制化的搜索建议，使用搜索框架

实现提供者的步骤：

1. 设置数据的原始存储。内容提供者有两种提供数据的方式：
  - 文件数据：图像、视频等一般是文件。Store the files in your application's private space. In response to a request for a file from another application, your provider can offer a handle to the file.
  - 结构化的数据：Data that normally goes into a database, array, or similar structure.
2. 实现 ContentProvider 类和方法。
3. 定义提供者的 **authority** 字符串，它的 URIs、列名等。If you want the provider's application to handle intents, also define intent actions, extras data, and flags. Also define the **permissions** that you will require for applications that want to access your data. You should consider defining all of these values as constants in a separate contract class; later, you can expose this class to other developers.
4. Add other optional pieces, such as sample data or an implementation of `AbstractThreadedSyncAdapter` that can synchronize data between the provider and cloud-based data.

#### 设计数据存储

> For working with network-based data, use classes in java.net and android.net. You can also synchronize network-based data to a local data store such as a database, and then offer the data as tables or files. The Sample Sync Adapter sample application demonstrates this type of synchronization.

一些建议：

- 表应该有主键列。类名一般取 `BaseColumns._ID` 的值。
- If you want to provide bitmap images or other very large pieces of file-oriented data, store the data in a file and then provide it indirectly rather than storing it directly in a table. If you do this, you need to tell users of your provider that they need to use a `ContentResolver` **file method** to access the data.
- 利用 Binary Large OBject (BLOB) 数据类型存储大小可变或结构可变的数据。例如，可以用它存储二进制或JSON。
- 利用 BLOB 实现 schema-independent 表。In this type of table, you define a primary key column, a MIME type column, and one or more generic columns as BLOB. The meaning of the data in the BLOB columns is indicated by the value in the MIME type column. This allows you to store different row types in the same table. The Contacts Provider's "data" table `ContactsContract.Data` is an example of a schema-independent table.

#### 设计内容 URIs

一个内容 URI 的形式是 `content://<authority>/<path>` 或 `content://<authority>/<path>/<id>`。分别表示一个表或具体一行记录。

**设计authority**

一个提供者一般只有一个 authority。为避免冲突，要加限定。例如，若你的应用包名是 `com.example.<appname>`，则 authority 可以是 `com.example.<appname>.provider`。

**设计路径结构**

URI 中 authority 后的路径表示一个“表”。例如 `com.example.<appname>.provider/table1`。路径可以有多段，到表的映射可以由提供者自由决定。

**处理内容 URI IDs**

一般来说，若提供者要提供到具体某一数据行的访问，在内容 URI 最后添加行的ID。

**内容 URI 模式**

为帮助提供者根据请求的 URI 判定要请求的内容，`UriMatcher` 可以帮助将内容 URI 模式处理成整数值。

内容 URI 模式提供两个通配符：

- `*`: Matches a string of any valid characters of any length.
- `#`: Matches a string of numeric characters of any length.

As an example of designing and coding content URI handling, consider a provider with the authority `com.example.app.provider` that recognizes the following content URIs pointing to tables:

- content://com.example.app.provider/table1: A table called table1.
- content://com.example.app.provider/table2/dataset1: A table called dataset1.
- content://com.example.app.provider/table2/dataset2: A table called dataset2.
- content://com.example.app.provider/table3: A table called table3.

The provider also recognizes these content URIs if they have a row ID appended to them, as for example `content://com.example.app.provider/table3/1` for the row identified by 1 in table3.

The following content URI patterns would be possible:

- `content://com.example.app.provider/*` Matches any content URI in the provider.
- `content://com.example.app.provider/table2/*` Matches a content URI for the tables dataset1 and dataset2, but doesn't match content URIs for table1 or table3.
- `content://com.example.app.provider/table3/#`: Matches a content URI for single rows in table3, such as content://com.example.app.provider/table3/6 for the row identified by 6.

The following code snippet shows how the methods in `UriMatcher` work. This code handles URIs for an entire table differently from URIs for a single row, by using the content URI pattern `content://<authority>/<path>` for tables, and `content://<authority>/<path>/<id>` for single rows.

The method `addURI()` maps an authority and path to an integer value. The method `match()` returns the integer value for a URI. A switch statement chooses between querying the entire table, and querying for a single record:

```java
public class ExampleProvider extends ContentProvider {

    private static final UriMatcher sUriMatcher;
...
    // The calls to addURI() go here, for all of the content URI patterns that the provider should recognize. For this snippet, only the calls for table 3 are shown.
    sUriMatcher.addURI("com.example.app.provider", "table3", 1);
    sUriMatcher.addURI("com.example.app.provider", "table3/#", 2);
...
    // Implements ContentProvider.query()
    public Cursor query(
        Uri uri,
        String[] projection,
        String selection,
        String[] selectionArgs,
        String sortOrder) {
...
        switch (sUriMatcher.match(uri)) {
            // If the incoming URI was for all of table3
            case 1:
                if (TextUtils.isEmpty(sortOrder)) sortOrder = "_ID ASC";
                break;
            // If the incoming URI was for a single row
            case 2:
                selection = selection + "_ID = " uri.getLastPathSegment();
                break;
            default:
            ...
                // If the URI is not recognized, you should do some error handling here.
        }
        // call the code to actually do the query
    }
```

Another class, `ContentUris`, provides convenience methods for working with the id part of content URIs. The classes `Uri` and `Uri.Builder` include convenience methods for parsing existing `Uri` objects and building new ones.

#### 实现 ContentProvider 类

##### 必须实现的方法

抽象类 `ContentProvider` 定义了6个抽象方法。除 `onCreate()` 外，都会被客户端调用。

- `query()`：Retrieve data from your provider. Use the arguments to select the table to query, the rows and columns to return, and the sort order of the result. Return the data as a `Cursor` object.
- `insert()`：Insert a new row into your provider. Use the arguments to select the destination table and to get the column values to use. Return a content URI for the newly-inserted row.
- `update()`：Update existing rows in your provider. Use the arguments to select the table and rows to update and to get the updated column values. Return the number of rows updated.
- `delete()`：Delete rows from your provider. Use the arguments to select the table and the rows to delete. Return the number of rows deleted.
- `getType()`：Return the MIME type corresponding to a content URI. This method is described in more detail in the section Implementing Content Provider MIME Types.
- `onCreate()`：Initialize your provider. The Android system calls this method immediately after it creates your provider. Notice that your provider is not created until a `ContentResolver` object tries to access it.

Notice that these methods have the same signature as the identically-named `ContentResolver` methods.

Your implementation of these methods should account for the following:

- All of these methods except `onCreate()` can be called by multiple threads at once, so they must be thread-safe.
- Avoid doing lengthy operations in `onCreate()`.

##### 实现 query() 方法

The `ContentProvider.query()` method must return a `Cursor` object, or if it fails, throw an Exception. If you are using an SQLite database as your data storage, you can simply return the `Cursor` returned by one of the `query()` methods of the `SQLiteDatabase` class. If the query does not match any rows, you should return a `Cursor` instance whose `getCount()` method returns 0. You should return `null` only if an internal error occurred during the query process.

If you aren't using an SQLite database as your data storage, use one of the concrete subclasses of `Cursor`. For example, the `MatrixCursor` class implements a cursor in which each row is an array of Object. With this class, use `addRow()` to add a new row.

Remember that the Android system must be able to communicate the Exception across process boundaries. Android can do this for the following exceptions that may be useful in handling query errors:

- IllegalArgumentException (You may choose to throw this if your provider receives an invalid content URI)
- NullPointerException


##### 实现 insert() 方法

The `insert()` method adds a new row to the appropriate table, using the values in the ContentValues argument. If a column name is not in the ContentValues argument, you may want to provide a default value for it either in your provider code or in your database schema.

This method should return the content URI for the new row. To construct this, append the new row's `_ID` (or other primary key) value to the table's content URI, using `withAppendedId()`.

##### 实现 delete() 方法

The `delete()` method does not have to physically delete rows from your data storage. If you are using a sync adapter with your provider, you should consider marking a deleted row with a "delete" flag rather than removing the row entirely. The sync adapter can check for deleted rows and remove them from the server before deleting them from the provider.

##### 实现 update() 方法

The `update()` method takes the same ContentValues argument used by insert(), and the same selection and selectionArgs arguments used by delete() and ContentProvider.query(). This may allow you to re-use code between these methods.

##### 实现 onCreate() 方法

The Android system calls` onCreate()` when it starts up the provider. You should perform only fast-running initialization tasks in this method, and defer database creation and data loading until the provider actually receives a request for the data. If you do lengthy tasks in `onCreate()`, you will slow down your provider's startup. In turn, this will slow down the response from the provider to other applications.

For example, if you are using an SQLite database you can create a new SQLiteOpenHelper object in ContentProvider.onCreate(), and then create the SQL tables the first time you open the database. To facilitate this, the first time you call getWritableDatabase(), it automatically calls the SQLiteOpenHelper.onCreate() method.

The following two snippets demonstrate the interaction between ContentProvider.onCreate() and SQLiteOpenHelper.onCreate(). The first snippet is the implementation of ContentProvider.onCreate():

```java
public class ExampleProvider extends ContentProvider
    // Defines a handle to the database helper object.
    private MainDatabaseHelper mOpenHelper;
    // 数据库名
    private static final String DBNAME = "mydb";
    // 数据库对象
    private SQLiteDatabase db;

    public boolean onCreate() {
        /*
         * Creates a new helper object. This method always returns quickly.
         * Notice that the database itself isn't created or opened
         * until SQLiteOpenHelper.getWritableDatabase is called
         */
        mOpenHelper = new MainDatabaseHelper(
            getContext(),        // the application context
            DBNAME,              // the name of the database)
            null,                // uses the default SQLite cursor
            1                    // the version number
        );
        return true;
    }
    ...
    // Implements the provider's insert method
    public Cursor insert(Uri uri, ContentValues values) {
        // Insert code here to determine which table to open, handle error-checking, and so forth
        ...
        // Gets a writeable database. This will trigger its creation if it doesn't already exist.
        db = mOpenHelper.getWritableDatabase();
    }
}
```

The next snippet is the implementation of `SQLiteOpenHelper.onCreate()`, including a helper class:

```java
...
// A string that defines the SQL statement for creating a table
private static final String SQL_CREATE_MAIN = "CREATE TABLE " +
    "main " +                       // Table's name
    "(" +                           // The columns in the table
    " _ID INTEGER PRIMARY KEY, " +
    " WORD TEXT"
    " FREQUENCY INTEGER " +
    " LOCALE TEXT )";
...
// Helper class that actually creates and manages the provider's underlying data repository.
protected static final class MainDatabaseHelper extends SQLiteOpenHelper {
    /*
     * Instantiates an open helper for the provider's SQLite data repository
     * Do not do database creation and upgrade here.
     */
    MainDatabaseHelper(Context context) {
        super(context, DBNAME, null, 1);
    }

    // Creates the data repository. This is called when the provider attempts to open the repository and SQLite reports that it doesn't exist.
    public void onCreate(SQLiteDatabase db) {

        // Creates the main table
        db.execSQL(SQL_CREATE_MAIN);
    }
}
```

#### 声明权限

Permissions and access for all aspects of the Android system are described in detail in the topic Security and Permissions. The topic Data Storage also described the security and permissions in effect for various types of storage.

##### 权限种类和粒度

默认你的提供者没有设置权限，所有应用可以读写你的提供者。要设置权限，需要通过设置 `<provider>` 元素的属性和子元素。

但首先要在 manifest 中利用 `<permission>` 把要创造的权限定义出来。权限名，`android:name`，要全限。例如一个读权限：`com.example.app.provider.permission.READ_PROVIDER`。

下面描述提供者权限的范围和粒度。粒度更小的权限优先级更高：

- **整个提供者级别的、读写合一的权限**：一个权限控制整个提供者的读和写访问，specified with the `android:permission` attribute of the `<provider>` element.
- **整个提供者级别的、读写分离的权限**：A read permission and a write permission for the entire provider. You specify them with the `android:readPermission` and `android:writePermission` attributes of the `<provider> `element. They take precedence over the permission required by `android:permission`.
- Path-level permission：Read, write, or read/write permission for a content URI in your provider. You specify each URI you want to control with a `<path-permission>` child element of the `<provider>` element. For each content URI you specify, you can specify a read/write permission, a read permission, or a write permission, or all three. The read and write permissions take precedence over the read/write permission. Also, path-level permission takes precedence over provider-level permissions.
- 临时权限：A permission level that grants temporary access to an application, even if the application doesn't have the permissions that are normally required. The temporary access feature reduces the number of permissions an application has to request in its manifest. When you turn on temporary permissions, the only applications that need "permanent" permissions for your provider are ones that continually access all your data.
  Consider the permissions you need to implement an email provider and app, when you want to allow an outside image viewer application to display photo attachments from your provider. To give the image viewer the necessary access without requiring permissions, set up temporary permissions for content URIs for photos. Design your email app so that when the user wants to display a photo, the app sends an intent containing the photo's content URI and permission flags to the image viewer. The image viewer can then query your email provider to retrieve the photo, even though the viewer doesn't have the normal read permission for your provider.
  To turn on temporary permissions, either set the android:grantUriPermissions attribute of the `<provider>` element, or add one or more `<grant-uri-permission>` child elements to your `<provider>` element. If you use temporary permissions, you have to call Context.revokeUriPermission() whenever you remove support for a content URI from your provider, and the content URI is associated with a temporary permission.
  The attribute's value determines how much of your provider is made accessible. If the attribute is set to true, then the system will grant temporary permission to your entire provider, overriding any other permissions that are required by your provider-level or path-level permissions.
  If this flag is set to false, then you must add `<grant-uri-permission>` child elements to your `<provider>` element. Each child element specifies the content URI or URIs for which temporary access is granted.
  To delegate temporary access to an application, an intent must contain the `FLAG_GRANT_READ_URI_PERMISSION` or the `FLAG_GRANT_WRITE_URI_PERMISSION` flags, or both. These are set with the `setFlags()` method.
  If the `android:grantUriPermissions` attribute is not present, it's assumed to be false.

##### `provider` 元素

Like Activity and Service components, a subclass of `ContentProvider` must be defined in the manifest file for its application, using the `<provider>` element. The Android system gets the following information from the element:

**Authority (`android:authorities`)**
Symbolic names that identify the entire provider within the system. This attribute is described in more detail in the section Designing Content URIs.

**Provider class name ( `android:name` )**
The class that implements ContentProvider. This class is described in more detail in the section Implementing the ContentProvider Class.

**Permissions**
该属性指明其他应用若想访问提供者的数据需要具备的权限：

- `android:grantUriPermssions`: Temporary permission flag.
- `android:permission`: Single provider-wide read/write permission.
- `android:readPermission`: Provider-wide read permission.
- `android:writePermission`: Provider-wide write permission.

Permissions and their corresponding attributes are described in more detail in the section Implementing Content Provider Permissions.

**启动与控制属性**
These attributes determine how and when the Android system starts the provider, the process characteristics of the provider, and other run-time settings:

- `android:enabled`: Flag allowing the system to start the provider.
- `android:exported`: Flag allowing other applications to use this provider.
- `android:initOrder`: The order in which this provider should be started, relative to other providers in the same process.
- `android:multiProcess`: Flag allowing the system to start the provider in the same process as the calling client.
- `android:process`: The name of the process in which the provider should run.
- `android:syncable`: Flag indicating that the provider's data is to be sync'ed with data on a server.

The attributes are fully documented in the dev guide topic for the `<provider>` element.

**Informational attributes**
An optional icon and label for the provider:

- `android:icon`: A drawable resource containing an icon for the provider. The icon appears next to the provider's label in the list of apps in Settings > Apps > All.
- `android:label`: An informational label describing the provider or its data, or both. The label appears in the list of apps in Settings > Apps > All.

The attributes are fully documented in the dev guide topic for the `<provider>` element.

### Intents and Data Access

Applications can access a content provider indirectly with an Intent. The application does not call any of the methods of ContentResolver or ContentProvider. Instead, it sends an intent that starts an activity, which is often part of the provider's own application. The destination activity is in charge of retrieving and displaying the data in its UI. Depending on the action in the intent, the destination activity may also prompt the user to make modifications to the provider's data. An intent may also contain "extras" data that the destination activity displays in the UI; the user then has the option of changing this data before using it to modify the data in the provider.

You may want to use intent access to help ensure data integrity. Your provider may depend on having data inserted, updated, and deleted according to strictly defined business logic. If this is the case, allowing other applications to directly modify your data may lead to invalid data. If you want developers to use intent access, be sure to document it thoroughly. Explain to them why intent access using your own application's UI is better than trying to modify the data with their code.

Handling an incoming intent that wishes to modify your provider's data is no different from handling other intents. You can learn more about using intents by reading the topic Intents and Intent Filters.
