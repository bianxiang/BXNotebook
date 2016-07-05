# Part I. 设计目标

1. 主要针对程序客户端，也要让AJAX可以使用。
1. 支持接口版本化。版本化到具体接口。
1. 目前只支持JSON格式的数据。

------------------------------------------------------------------------

# Part II. 对外接口规范

## 0. 概览

1. 所有资源名复数。例如礼品取`/gifts`。多个单词使用驼峰命名，如	`/softwareInfos`。
1. 常见CRUD接口：
	* 列表：`GET /gifts?page=1?pageSize=10`
	* 查询： `GET /gifts/1`
	* 子资源： `GET /gifts/1/items`
	* 新增（服务器决定主键）：`POST /gifts`
	* 删除：`DELETE /gifts/1`
	* 修改：`PUT /gifts/1`
	* 元数据：`HEAD /gifts?page=1?pageSize=10`
1. 目前接口只支持`JSON`格式，`UTF8`编码。
1. 支持GZIP压缩。
1. 接口是版本化了的。一定要设置正确的版本化！例如`application/vnd.xctvstore+json;version=1`。  
	版本号为整数。当出现不兼容性修改时会增加版本号。
	* 发送数据（POST或PUT）时，请设置`Content-Type`头，如`Content-Type=application/vnd.xctvstore+json;version=1`。
	* 接收数据时，请设置`Accept`，如`Accept=application/vnd.xctvstore+json;version=1`。  
	若不设置（如使用`Accept:*/*`），将使用版本为1的数据格式——注意，*我们并不保证在不设置版本号的情况下不会出错！*
	* 响应数据的版本号在响应头`Content-type`中。
1. 客户端要能处理`3xx`、`4xx`、`5xx`系列响应。  
带消息体的响应响应码为`200`，不带响应体的响应码是`204`。  
特别注意处理`301`等重定向请求。不要因响应码不是`2xx`即认为失败。注意处理`304 Not Modified`。
1. 由于JSON不支持日期类型，因此将日期格式化为字符串。接口中所有请求响应数据中，日期均应使用*ISO8601*格式。
格式串：`yyyy-MM-dd'T'HH:mm:ss.SSSZ`。例子：`2013-09-05T07:29:13.000+0000`。
1. 支持缓存控制。条件查询。条件更新。

------------------------------------
	
## 1. 列表查询（分页）

典型请求响应

	GET /tvstoreapi/gifts?page=2&pageSize=5 HTTP/1.1
	Accept: application/vnd.xctvstore+json;version=2

* `page`是页码，从`1`开始。`pageSize`是页面大小。二者均可省略，默认值分别是`1`、`10`。
* 其他请求参数也通过查询参数列出。

响应：

	HTTP/1.1 200 OK
	X-Total-Count: 11
	Link: <gifts?page=1&pageSize=5>; rel="prev"
	Link: <gifts?page=3&pageSize=5>; rel="next"
	Link: <gifts?page=3&pageSize=5>; rel="last"
	Content-Type: application/vnd.xctvstore+json; version=2

* `X-Total-Count`表示资源总数量。
* `Link`中分别给出上一页、下一页、最后一页的页面链接。若没有上一页或下一页，则没有对应链接。 

------------------------------------

## 2. 查询单个记录

典型请求响应
	
	GET /tvstoreapi/gifts/2 HTTP/1.1
	Accept: application/vnd.xctvstore+json; version=2
	
响应：

	HTTP/1.1 200 OK
	Content-Type: application/vnd.xctvstore+json; version=1

	{"name":"马尔代夫","pic":"uploadfile/gift/201396933511000.png", ...}


* 找不到响应`404`。

------------------------------------

## 3. 删除

典型请求

	DELETE /tvstoreapi/gifts/1 HTTP/1.1
	Accept: */*

响应说明

* 删除有两种模式，严格模式和普通模式。严格模式下，若记录找不到（或被并发删除），返回`404`；普通模式下返回`204`。  
默认取普通模式，若要采用严格模式，加请求参数`?strict=true`。
* 返回除了`204`表示成功外，还可能响应`4xx`表示客户端错误，如`400 Bad Request`、`409 conflict`；	  
`5xx`响应，表示服务器内部错误。

------------------------------------

## 4. 新增（插入）

典型请求响应：

	POST /tvstoreapi/gifts HTTP/1.1
	Content-Length: 725
	Content-Type: application/vnd.xctvstore+json;version=1
	
	{
	    "name": "马尔代夫",
	    "pic": "uploadfile/gift/201396933511000.png",
	 	...
	}

响应：

	HTTP/1.1 201 Created
	Location: http://localhost:8080/tvstoreapi/gifts/1

* 成功响应`201`。
* 因为是服务器产生主键，会在`Location`头中给出产生的资源的链接。

------------------------------------
## 5. 更新

典型请求响应

	PUT /tvstoreapi/gifts/2 HTTP/1.1
	Content-Length: 738
	Content-Type: application/vnd.xctvstore+json;version=1
	
	{
	    "name": "马尔代夫",
	    "pic": "uploadfile/gift/201396933511000.png",
	    ...
	}

响应：

	HTTP/1.1 204 No Content


* 虽然根据REST语义，`PUT`是全量更新，`PATCH`是局部更新。由于我们目前不支持`PATCH`，因此目前我们的`PUT`是局部更新。即只更新传入的字段。不在请求中出现的字段不会更新。
* 成功响应`204`。找不到响应`404`。其他`400`错误表示客户端错误。

------------------------------------------------------------------------

# Part III. 安全（身份认证与权限控制）

部分接口不需要任何身份认证就能访问。其他接口需要通过身份认证和权限控制才能访问。

若未通过身份认证，响应`401 Unauthorized`。若通过身份认证但无相关权限，响应`403 Forbidden`。

## 登录方式

RESTful接口是无状态。因此每个请求都需要携带身份信息。有两个关键的请求头需要设置：`X-Auth`和`X-Auth-Timestamp`。

请求样例：

	PUT http://localhost:8080/tvstoreapi/security/users/yaoyuan2/permissions HTTP/1.1
	Cache-Control: no-cache
	X-Auth: someone:1BBFCAB0652785A79014792E64F04931
	X-Auth-Timestamp: 1387001811897
	Content-Type: application/vnd.xctvstore+json; version=1

`X-Auth-Timestamp`表示发出请求的时间。它是自1970年1月1日0点到现在的毫秒数。

> Java中通过`System.currentTimeMillis()`即可获得`X-Auth-Timestamp`的值。   
	在Javascript中，通过`Date`对象的`getTime()`方法可以获得该值。

`X-Auth`提供身份信息。它由两部分构成。冒号前是用户名（如`someone`）。
冒号后是认证信息。它是由用户名、密码和一个混淆因子经MD5加密而成的。
接入系统的用户会获得用户名、密码和一个密位。密位指定将`X-Auth-Timestamp`中哪一位数字作为混淆因子。
例如，加入用户名是someone，密码是abc。`X-Auth-Timestamp`是1387001811897。密位是3。
则取1387001811897右边第3位数，即8，做混淆因子。然后将字符串`someone8abc`做MD5（32位）。
计算结果作为`X-Auth`冒号右边部分（`1BBFCAB0652785A79014792E64F04931`）。

使用混淆因子的目的是，身份验证头的值会经常变动。

------------------------------------------------------------------------

# Part IV. 缓存相关

## 4.1 GET缓存

注意！如果客户端不支持缓存（如不能正确处理`304 Not Modified`响应），请正确设置请求头告诉服务器不要使用缓存。
如`Cache-Control: no-cache`；不要发送`If-None-Match`等请求头。 

---

1. 如果服务器认为内容可缓存且客户端未显式禁用缓存。`Cache-Control`头可能给出缓存时间。如：`Cache-Control:max-age=15, private`。
1. 支持ETag和条件请求。

## 4.2 条件更新

支持的接口会给出显式的说明。


