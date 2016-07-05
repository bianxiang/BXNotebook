[toc]

### 数据库

数据库存放在`/data/data/<package_name>/databases`。数据是安全的，因为默认不能被其他应用访问。

#### SQLite

SQLite本地API由C写成。JDBC对内存受限的设备来说过于昂贵。因此得使用新API。

By being implemented as a library, rather than running as a separate ongoing process, each SQLite database is an integrated part of the application that created it. This reduces external dependencies, minimizes latency, 简化了事务锁和同步。

整个SQLite都被包含进了Android。

SQLite has a reputation for being extremely reliable and is the database system of choice for many consumer electronic devices, including many MP3 players and smartphones.

SQLite列没有数据类型，可以存放任意值。尽管在CREATE TABLE语句中可以指定列类型，但对SQLite来说只是个示意。SQLite refers to this as manifest typing, as described in the documentation:

> In manifest typing, 数据类型是值自身的一个属性，not of the column in which the value is stored. SQLite thus allows the user to store any value of any datatype into any column regardless of the declared type of that column.

##### SQLite和Android版本

The underlying SQLite library included with Android has evolved as new versions of both continue to be released. The initial release of Android shipped with SQLite 3.5.9. Android 2.2 Froyo updated the SQLite library to 3.6.22. This was a relatively minor upgrade, dealing with bug fixes and the like. Android 3.0 Honeycomb again upgraded the SQLite library to 3.7.4, and this is still the version in use with Android 4.0 Ice Cream Sandwich. While you can treat this upgrade as just another point release fixing bugs and providing incremental improvements, the 3.7 release of SQLite includes a quite radical set of enhanced features around concurrency, logging, and locking.

You might never need to worry about these changes, particularly as your application is likely to be the only one concurrently accessing your SQLite database. There are however a few subtleties introduced.

First, the major internal version number of the SQLite database format was incremented for new databases created with SQLite version 3.7 and later, and older databases could be upgraded to this new format. If you plan on packaging your own SQLite database as part of your application (rather than creating it via onCreate()), you should consider which older devices and versions of Android you will support, and ensure you use the older SQLite database format. It can still beread and manipulated by SQLite 3.7.4 without any form of upgrade.

Second, some new features of SQLite are obviously only made available in later versions. This will mainly affect some of the more advanced queries you execute using rawQuery(), such as using SQL standard foreign key creation commands.

#### Contract和表格结构

定义一个伙伴类，称为contract类，显式定义表结构。Contract类是常量的容器，定义URI、表和列。为的是改变列名后，能传播到所有代码。

组织Contract类的一个好的方式是，对于整个数据库全局起效的定义放在类的顶级。为每个表创建一个子类。

> 内部类可以实现`BaseColumns`接口，继承主键字段`_ID`。一些Android类（如cursor adaptors）期望有这个字段。

例子，下面是表明和列名的定义。

```java
    public final class FeedReaderContract {
        public FeedReaderContract() {}

        /* 内部类定义表的内容 */
        public static abstract class FeedEntry implements BaseColumns {
            public static final String TABLE_NAME = "entry";
            public static final String COLUMN_NAME_ENTRY_ID = "entryid";
            public static final String COLUMN_NAME_TITLE = "title";
            public static final String COLUMN_NAME_SUBTITLE = "subtitle";
            ...
        }
    }
```

#### 设计

- 文件不要存放在数据库表中。可以存放它们的路径。
- 每个表最好都有一个自增的主键。If you plan to share your table using a Content Provider, a unique ID field is required.

#### SQLiteOpenHelper

> 注意创建的是数据库，不是表！！！

`SQLiteOpenHelper`是一个抽象类，实现创建、打开、升级数据库的最佳实践。利用`SQLiteOpenHelper`，隐藏了在打开数据库前是否需要先创建或升级的逻辑，并确保每个操作都高效的完成。

`SQLiteOpenHelper`中创建、更新数据库会被延迟到首次使用时，并不一定在应用启动时。创建、更新、打开数据库可能花费较长时间。

数据库被成功打开后，`SQLiteOpenHelper`会缓存数据库实例。

处于效率原因，一般不需要手工关闭数据库，除非你不再用它了（如活动或服务被停止时）。

你的子类需要实现三个方法：

- 构造器。参数有Context（如一个Activity）、数据库名、可选的游标工厂（一般传null）、数据库schema版本（整数）。
- `onCreate()`，传给你一个SQLiteDatabase对象，可以填入初始化数据。
- `onUpgrade()`，传给你一个SQLiteDatabase对象、老版本和新版本。利用此做数据库schema升级。如果原始数据不重要，最简单的方式是删除原来表，创建新的。

例子：

```java
    public class DatabaseHelper extends SQLiteOpenHelper {
        private static final String DATABASE_NAME="db";
        static final String TITLE="title";
        static final String VALUE="value";

        public DatabaseHelper(Context context) {
	        super(context, DATABASE_NAME, null, 1);
        }

		@Override
        public void onCreate(SQLiteDatabase db) {
            db.execSQL("create table constants (_id integer primary key autoincrement, title text, value real);");
            ContentValues cv=new ContentValues();
            cv.put(TITLE, "Gravity, Death Star I");
            cv.put(VALUE, SensorManager.GRAVITY_DEATH_STAR_I);
            db.insert("constants", TITLE, cv);

            cv.put(TITLE, "Gravity, Earth");
            cv.put(VALUE, SensorManager.GRAVITY_EARTH);
            db.insert("constants", TITLE, cv);

            ...
            cv.put(TITLE, "Gravity, Venus");
            cv.put(VALUE, SensorManager.GRAVITY_VENUS);
            db.insert("constants", TITLE, cv);
        }

        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
            Log.w("Constants", "Upgrading database, which will destroy all old data");
            db.execSQL("drop table if exists constants");
            onCreate(db);
        }
    }
```

要访问数据库，实例化你的SQLiteOpenHelper：

```java
	FeedReaderDbHelper mDbHelper = new FeedReaderDbHelper(getContext());
```

调用SQLiteOpenHelper的`getReadableDatabase()`或`getWriteableDatabase()`，获取SQLiteDatabase对象，然后利用它增删改查。

注意**应在后台线程中**调用`getWritableDatabase()`或`getReadableDatabase()`，如`AsyncTask`或`IntentService`。

```java
	constantsCursor=db.getReadableDatabase()
		.rawQuery("select _id, title, value from constants order by title", null);
```

如果数据库不存在，帮助类会执行`onCreate`。如果数据库版本发生变化，执行`onUpgrade`。

数据库被成功打开后，SQLiteOpenHelper会缓存它，每次需要数据实例时，应该调用一次上述方法{{getXXXDatabase()}}，而不是将打开的数据库缓存在你的应用。

由于磁盘空间和权限问题，`getWritableDatabase`可能失败。因此读的时候用`getReadableDatabase`就好。多数情况下`getReadableDatabase`会返回之前`getWritableDatabase`缓存的可写的数据库实例，除非由于磁盘空间或权限问题，这个可写的实例尚未存在。

若要创建和更新数据库，必须使用可写的数据库。因此，最好先获取可写的数据库，不行再获取只读的。

最后调用`SQLiteOpenHelper.close()`释放连接。

SQLiteOpenHelper中还有两个方法，可以选择性的覆盖：

- `onOpen()`: You can override this to get control when somebody opens this database. 一般不需要。
- `onDowngrade()`: Introduced in Android 3.0, this method will be called if the code requests a schema that is older than what is in the database presently. This is the converse of onUpgrade(). If your version numbers differ, one of these two methods will be invoked. Since normally you are moving forward with updates, you can usually skip onDowngrade().

#### 打开和创建数据库（不使用SQLite Open Helper）

如果不想使用SQLite Open Helper，向直接关联创建、打开、版本。可以使用应用上下文的openOrCreateDatabase方法：

```java
	SQLiteDatabase db = context.openOrCreateDatabase(DATABASE_NAME, Context.MODE_PRIVATE, null);
```

创建好数据库，利用数据库的execSQL方法增删表。

It’s good practice to defer creating and opening databases until they’re needed, and to cache database instances after they’re successfully opened to limit the associated efficiency costs.

#### 建表和索引

要创建数据库和索引，需要调用`SQLiteDatabase.execSQL()`。除非数据库错误，否则该方法什么也不返回。

创建表：

```cpp
	db.execSQL("create table constants (_id integer primary key autoincrement, title text, value real);");
```

SQLite会自动为主键列见索引。显式索引需要通过CREATE INDEX创建。

`execSQL()`也可以用于调用DROP INDEX和DROP TABLE。

#### 增删改

有两种方式处理数据：

- 使用`execSQL()`执行SQL。
- 使用`insert()`, `update()`, `delete()`。

##### 插入

`ContentValues`用于向表中插入新行。一个`ContentValues`对象表示一个数据库行。`ContentValues`是一个类似Map的接口，但能够处理SQL类型。例如，除了get()外，还有`getAsInteger()`, `getAsString()`等。

例子：

```cpp
    private void processAdd(DialogWrapper wrapper) {
        ContentValues values=new ContentValues(2);
        values.put(DatabaseHelper.TITLE, wrapper.getTitle());
        values.put(DatabaseHelper.VALUE, wrapper.getValue());

        db.getWritableDatabase().insert("constants", DatabaseHelper.TITLE, values);
        constantsCursor.requery();
    }
```

向数据库插入行时，必须显式指定至少一列或相应的值（可以为null）。如果`ContentValues`是空的（即没有指定任何列），且insert的第二个参数（即null column hack参数）为null，SQLite将抛出异常。因此，如果想向SQLite插入空行（此时ContentValues是空的）。你还需要传入一个列，该列的值可以被显式设置为null。This is required due to a quirk in SQLite’s support for the SQL INSERT statement.

不插入空的Content Values是一个新的实践。

##### 更新

`update()`方法的参数是表名，一个`ContentValues`表示替换值，一个可选的WHERE子句，一个可选的WHERE子句参数。

```cpp
    SQLiteDatabase db = mDbHelper.getReadableDatabase();

    // New value for one column
    ContentValues values = new ContentValues();
    values.put(FeedEntry.COLUMN_NAME_TITLE, title);

    // Which row to update, based on the ID
    String selection = FeedEntry.COLUMN_NAME_ENTRY_ID + " LIKE ?";
    String[] selectionArgs = { String.valueOf(rowId) };

    int count = db.update(
        FeedReaderDbHelper.FeedEntry.TABLE_NAME,
        values,
        selection,
        selectionArgs);
```

`update()`只能将列设为一个固定值。如果想计算值（set a=a+1），用`execSQL()`。

##### 删除

The `delete()` method works akin to `update()`, taking the name of the table, the optional WHERE clause, and the corresponding parameters to fill into the WHERE clause. For example, here we `delete()` a row from our constants table, given its `_ID`:

```cpp
    private void processDelete(long rowId) {
        String[] args = {String.valueOf(rowId)};
        db.getWritableDatabase().delete("constants", "_ID=?", args);
        constantsCursor.requery();
    }
```

#### 查询

从数据库中查询数据有两种方式：

- 使用`rawQuery()`直接调用SELECT语句
- 用`query()`构造查询

Confounding matters further is the `SQLiteQueryBuilder` class and the issue of cursors and cursor factories. Let’s take all of this one piece at a time.

##### rawQuery()

SELECT语句可以有位置参数。实际参数数组是第二个参数；或null，如果没有参数。

```cpp
    constantsCursor=db.getReadableDatabase().rawQuery("SELECT _ID, title, value " + "FROM constants ORDER BY title", null);
```

返回值是一个`Cursor`。

##### query()

query()需要你传入SELECT语句的各个部分，依次为：

- 要查询的表名
- 投影：要查询的列名
- WHERE子句，可以带位置参数?。
- 一组值，替换位置参数
- GROUP BY子句
- HAVING子句
- ORDER BY子句
- 可选布尔值，指定结果集是否只包含唯一值。
- 最大返回多少行

上面任何一部分，若不需要，传入null。

```java
    // 结果投影
    String[] result_columns = new String[] {
	    KEY_ID, KEY_GOLD_HOARD_ACCESSIBLE_COLUMN, KEY_GOLD_HOARDED_COLUMN };
    // Where子句
    String where = KEY_GOLD_HOARD_ACCESSIBLE_COLUMN + “=” + 1;
    // Replace these with valid SQL statements as necessary.
    String whereArgs[] = null;
    String groupBy = null;
    String having = null;
    String order = null;
    SQLiteDatabase db = hoardDBOpenHelper.getWritableDatabase();
    Cursor cursor = db.query(HoardDBOpenHelper.DATABASE_TABLE,
	    result_columns, where,whereArgs, groupBy, having, order);
```

这个方法不能做连表查询。

##### Cursor

不管使用何种查询方式，得到的都是Cursor。这使得Android能更高效的关系资源，按需查询和释放行和列。

下面是Cursor类常用的导航函数：

- `moveToFirst`：将游标移到结果集第一行
- 通过`moveToFirst()`、`moveToNext()`、`isAfterLast()`遍历行
- `getCount`：结果集行数（将导致隐式查询结果集中所有数据）
- `getColumnIndexOrThrow`：给定列名的索引（0开始），不存在抛异常
- `getColumnName`：给定列索引的名字
- `getColumnNames`：当前游标中的所有列名，字符串数组
- `moveToPosition`：将游标移至指定行
- `getPosition`：放回当前游标位置
- 获取当前行给定列的值：`getString()`, `getInt()`等。
- 重新执行查询`requery()`
- 释放游标资源：`close()`

Android provides a convenient mechanism to ensure queries are performed asynchronously. The `CursorLoader` class and associated Loader Manager (described later in this chapter) were introduced in Android 3.0 (API level 11) and are now also available as part of the support library.

例子。遍历：

```java
    Cursor result = db.rawQuery("select id, name, inventory from widgets", null);
    while (!result.moveToNext()) {
	    int id=result.getInt(0);
    	String name=result.getString(1);
    	int inventory=result.getInt(2);
    	// do something useful with these
    }
    result.close();
```

It’s good practice to use `getColumnIndexOrThrow` when you expect the column to exist in all cases. Using `getColumnIndex` and checking for a –1 result, as shown in the following snippet, is a more efficient technique than catching exceptions when the column might not exist in every case.

```java
    int columnIndex = cursor.getColumnIndex(KEY_COLUMN_1_NAME);
    if (columnIndex > -1) {
    	String columnValue = cursor.getString(columnIndex);
    	// Do something with the column value.
    } else {
    	// Do something else if the column doesn’t exist.
    }
```

使用完Cursor后，记得关闭它{{是否需要try-finally}}，防止内存泄漏，减少应用的资源加载：

```java
	cursor.close();
```

##### CursorAdapter

还可以将Cursor包裹进`SimpleCursorAdapter`或其他实现，然后将适配器放入`ListView`等。注意使用CursorAdapter（及其子类），要求查询结果必须包含`_ID`列，唯一标识结果。This “id” value is then supplied to methods such as `onListItemClick()`, to identify which item the user clicked on in the `AdapterView`.

For example, after retrieving the sorted list of constants, we pop those into the `ListView` for the `ConstantsBrowser` activity in just a few lines of code:

```java
    ListAdapter adapter =new SimpleCursorAdapter(this, R.layout.row, constantsCursor,
    	new String[] {DatabaseHelper.TITLE, DatabaseHelper.VALUE}, new int[] {R.id.title, R.id.value});
```

##### 定制CursorAdapters

之前我们通过覆盖`ArrayAdapter`的`getView()`方法，定制数据行的显示方式。但`CursorAdapter`及其子类默认实现了`getView()`，其实现尝试循环使用视图。如果视图为null,则`getView()`调用`newView()`，然后调用`bindView()`。如果不为null，`getView()`直接调用`bindView()`。如果你想扩展`CursorAdapter`，应该覆盖`newView()`和`bindView()`，而不是`getView()`。

All this does is remove your `if()` test you would have had in `getView()` and puts each branch of that test in an independent method, akin to the following:

```java
    public View newView(Context context, Cursor cursor, ViewGroup parent) {
        LayoutInflater inflater=getLayoutInflater();
        View row=inflater.inflate(R.layout.row, null);
        ViewWrapper wrapper=new ViewWrapper(row);
        row.setTag(wrapper);
        return(row);
        }
    public void bindView(View row, Context context, Cursor cursor) {
        ViewWrapper wrapper = (ViewWrapper)row.getTag();
        // actual logic to populate row from Cursor goes here
    }
```

##### 自定义游标

There may be circumstances in which you want to use your own Cursor subclass, rather than the stock implementation provided by Android. In those cases, you can use `queryWithFactory()` and `rawQueryWithFactory()`, which take a `SQLiteDatabase.CursorFactory` instance as a parameter. The factory, as you might expect, is responsible for creating new cursors via its `newCursor()` implementation.

Finding and implementing a valid use for this facility is left as an exercise for you. Suffice it to say that you should not need to create your own cursor classes much, if at all, in ordinary Android development.

#### 读写性能、可靠性

你的数据库放在flash上。读取一般很快。

但写数据就不一定了。有时很快，几毫秒。但有时要几百毫秒，即使写入数据很少。Flash越满时越慢。
因此考虑在非主线程中写数据库，如AsyncTask。

There are also situations in which writing to flash-based storage can be a risky move. When battery power is low, believing that writing to flash will complete before the battery is drained can be a bit too trusting on your part as a developer. Similarly, relying on the ability to write to flash during power-cycling of the device is not a good move. In these situations, you can add an Intent receiver to your application to watch out for ACTION_BATTERY_CHANGED broadcasts, and then examine the data provided to see what’s happening to the battery, its current charge level, and so on.

Note that the emulator behaves differently, because it is typically using a file on your hard drive for storing data, rather than flash. While the emulator tends to be much slower than hardware for CPU and GPU operations, the emulator will tend to be much faster for writing data to flash. Hence, just because you are not seeing any UI slowdowns due to database I/O in the emulator, do not assume that will be the same when your code is running on a real Android device.

#### 随应用分发数据库

Many applications are shipped with an existing database inplace, to support all manner of uses from a handy reference list, through to a complete offline cache. You can incorporate a database you’ve created elsewhere in your project for packaging with your compiled application. 

First, include your SQLite database file in the assets/ folder of your project. To use your bundled database in your code, you can pass its location and file name to the openDatabase() method. Calling openDatabase() can take as its first parameter the full path and file name. In practice, this full path and file name is constructed by concatenating the following: 
	The path used to refer toall database assets, /data/data/your.application.package/databases/
	Then your desired database file name; e.g., your-db-name 

