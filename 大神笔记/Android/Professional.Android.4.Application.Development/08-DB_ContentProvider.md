[toc]

## （未）8 Databases and Content Providers

### 8.1 介绍

#### （未）8.1.1 SQLite Databases

#### 8.1.2 Content Providers

内容提供者提供一个接口，用于发布和消费数据，基于一个简单的 URI 地址模型，`content://`。它使得应用层和数据层解耦，使得应用不需要关心底层数据。

内容提供者可以分享给其他应用。只要具有相应权限，便可以查询结果、更新或添加数据。

三方应用可以访问部分原生的内容提供者，包括联系人、媒体中心、日历。

### 8.5 创建内容提供者（Content Providers）

Content Providers provide an interface for publishing data that will be consumed using a Content Resolver. They allow you to decouple the application components that consume data from their underlying data sources, providing a generic mechanism through which applications can share their data or consume data provided by others.

要创建一个新的 Content Provider，实现抽象类 `ContentProvider`：

```java
public class MyContentProvider extends ContentProvider
```

Like the database contract class described in the previous section, it’s good practice to include static database constants — particularly column names and the Content Provider authority — that will be required for transacting with, and querying, the database.

You will also need to override the `onCreate` handler to initialize the underlying data source, as well as the `query`, `update`, `delete`, `insert`, and `getType` methods to implement the interface used by the Content Resolver to interact with the data, as described in the following sections.

#### 8.5.1 注册 Content Providers

与活动与服务一样，Content Providers 要注册到 application manifest。This is done using a `provider` tag that includes a `name` attribute describing the Provider’s class name and an `authorities` tag.

Use the `authorities` tag to define the base URI of the Provider’s authority. A Content Provider’s authority is used by the **Content Resolver** as an address and used to find the database you want to interact with.

Each Content Provider authority must be unique, so it’s good practice to base the URI path on your package name. The general form for defi ning a Content Provider’s authority is as follows: `com.<CompanyName>.provider.<ApplicationName>`

The completed provider tag should follow the format show in the following XML snippet:

```xml
	<provider android:name=”.MyContentProvider”
    	android:authorities=”com.paad.skeletondatabaseprovider”/>
```

#### 8.1.2 发布内容提供者的 URI 地址

Each Content Provider should expose its authority using a public static `CONTENT_URI` property to make it more easily discoverable. This should include a data path to the primary content — for example:

```java
public static final Uri CONTENT_URI = Uri.parse(“content://com.paad.skeletondatabaseprovider/elements”);
```

These content URIs will be used when accessing your Content Provider using a Content Resolver. A query made using this form represents a request for all rows, whereas an appended trailing `/<rownumber>`, as shown in the following snippet, represents a request for a single record:

```java
content://com.paad.skeletondatabaseprovider/elements/5
```

It’s good practice to support access to your provider for both of these forms. The simplest way to do this is to use a `UriMatcher`, a useful class that parses URIs and determines their forms.

Listing 8-8 shows the implementation pattern for defining a URI Matcher that analyzes the form of a URI — specifically determining if a URI is a request for all data or for a single row.

LISTING 8-8: Defining a UriMatcher to determine if a request is for all elements or a single row

```java
// Create the constants used to differentiate between the different URI
// requests.
private static final int ALLROWS = 1;
private static final int SINGLE_ROW = 2;
private static final UriMatcher uriMatcher;
// Populate the UriMatcher object, where a URI ending in
// ‘elements’ will correspond to a request for all items,
// and ‘elements/[rowID]’ represents a single row.
static {
    uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
    uriMatcher.addURI(“com.paad.skeletondatabaseprovider”, “elements”, ALLROWS);
    uriMatcher.addURI(“com.paad.skeletondatabaseprovider”, “elements/#”, SINGLE_ROW);
}
```

You can use the same technique to expose alternative URIs within the same Content Provider that represent different subsets of data, or different tables within your database.

Having distinguished between full table and single row queries, you can use the
`SQLiteQueryBuilder` class to easily apply the additional selection condition to a query, as shown in the following snippet:

```java
SQLiteQueryBuilder queryBuilder = new SQLiteQueryBuilder();
// If this is a row query, limit the result set to the passed in row.
switch (uriMatcher.match(uri)) {
case SINGLE_ROW :
    String rowID = uri.getPathSegments().get(1);
    queryBuilder.appendWhere(KEY_ID + “=” + rowID);
default: break;
}
```

You’ll learn how to perform a query using the SQLite Query Builder later in the “Implementing Content Provider Queries” section.

#### 8.1.3 创建内容提供者的数据

To initialize the data source you plan to access through the Content Provider, override the onCreate
method, as shown in Listing 8-9. This is typically handled using an SQLite Open Helper implementation,
of the type described in the previous section, allowing you to effectively defer creating and
opening the database until it’s required.


