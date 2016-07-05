# 使用何种库

[toc]

参考：

* [Android’s HTTP Clients](http://android-developers.blogspot.com/2011/09/androids-http-clients.html#uds-search-results)

Android自带三个类可用：

* [AndroidHttpClient](http://developer.android.com/reference/android/net/http/AndroidHttpClient.html) ： Apache DefaultHttpClient的实现，配置了一些适用于Android的默认设置。
* [DefaultHttpClient](http://developer.android.com/reference/org/apache/http/impl/client/DefaultHttpClient.html)
* [HttpURLConnection](http://developer.android.com/reference/java/net/HttpURLConnection.html)

该使用哪个？Android官方博客总结道：

* 对于2.0和2.2，Apache HTTP client是最好选择。（Apache HTTP client has fewer bugs）。
* 对于2.3及以后，`HttpURLConnection`是最好的选择。它的API简单、不繁多，适合Android。透明的压缩和响应缓存减少了网络访问，改善了性能，减少了电池使用。新的App应该使用`HttpURLConnection`。**这个类我们还在改善中**。

官方博客称：
> DefaultHttpClient和AndroidHttpClient用于大量的、灵活的API。它们的实现稳定、Bug少。  
> 但由于API太多，我们很难在不破坏兼容性的情况下改进它。Android团队目前不再维护Apache HTTP Client。  
> 由于`HttpURLConnection` API的简单性，使得我们可以持续提高它。

# HttpURLConnection

2.2之前，`HttpURLConnection`有一些Bug。例如calling close() on a readable InputStream could poison the connection pool. Work around this by disabling connection pooling:

	private void disableConnectionReuseIfNecessary() {
	    // HTTP connection reuse which was buggy pre-froyo
	    if (Integer.parseInt(Build.VERSION.SDK) < Build.VERSION_CODES.FROYO) {
	        System.setProperty("http.keepAlive", "false");
	    }
	}

在2.3，添加了透明的响应压缩（response compression）。`HttpURLConnection`将自动向请求添加以下头，并处理响应响应：

	Accept-Encoding: gzip

但如果响应压缩造成了问题，可以禁用它。

由于`Content-Length`头返回的是压缩的大小，根据`getContentLength()`为解压后的数据设置缓存是不对的、你应该一直读知道`InputStream.read()`返回-1。

We also made several improvements to HTTPS in Gingerbread. HttpsURLConnection attempts to connect with Server Name Indication (SNI) which allows multiple HTTPS hosts to share an IP address. It also enables compression and session tickets. Should the connection fail, it is automatically retried without these features. This makes HttpsURLConnection efficient when connecting to up-to-date servers, without breaking compatibility with older ones.

在4.0，引入了response cache。若安装了缓存，可能有以下三种响应情况：

* Fully cached responses are served directly from local storage. Because no network connection needs to be made such responses are available immediately.
* Conditionally cached responses must have their freshness validated by the webserver. The client sends a request like “Give me /foo.png if it changed since yesterday” and the server replies with either the updated content or a 304 Not Modified status. If the content is unchanged it will not be downloaded!
* Uncached responses are served from the web. 然后这些响应会被缓存到响应缓存。

利用反射启用HTTP响应缓存。下面的代码将在4.0上启用响应缓存，同时不影响之前的版本。

	private void enableHttpResponseCache() {
	    try {
	        long httpCacheSize = 10 * 1024 * 1024; // 10 MiB
	        File httpCacheDir = new File(getCacheDir(), "http");
	        Class.forName("android.net.http.HttpResponseCache")
	            .getMethod("install", File.class, long.class)
	            .invoke(null, httpCacheDir, httpCacheSize);
	    } catch (Exception httpResponseCacheNotAvailable) {
	    }
	}

You should also configure your Web server to set cache headers on its HTTP responses.

## 使用方式

1. 调用`URL.openConnection()`，获取一个`HttpURLConnection`实例，注意这里需要一个强制类型转换。
2. 准备请求。equest headers may also include metadata such as credentials, preferred content types, and session cookies.
3. （可选）上传请求头。如果有请求体，必须调用`setDoOutput(true)`。向`getOutputStream()`返回的流写数据。
4. 读响应。从`getInputStream()`返回的流中读响应。如果没有响应，该方法返回一个empty stream。
5. 断开连接。读完响应，调用`disconnect()`。断开释放连接占用的资源。于是它可以被关闭或**重用**。

例子，访问页面http://www.android.com/：

	URL url = new URL("http://www.android.com/");
	HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
	try {
		InputStream in = new BufferedInputStream(urlConnection.getInputStream());
		readStream(in);
	finally {
		urlConnection.disconnect();
	}

## POST内容

先调用`setDoOutput(true)`。

为了提供性能，如果提前知道长度，设置`setFixedLengthStreamingMode(int)`；否则设置`setChunkedStreamingMode(int)`。否则，在传输开始前`HttpURLConnection`要求将整个请求体缓存在内存中。

	HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
	try {
		urlConnection.setDoOutput(true);
		urlConnection.setChunkedStreamingMode(0);

		OutputStream out = new BufferedOutputStream(urlConnection.getOutputStream());
		writeStream(out);

		InputStream in = new BufferedInputStream(urlConnection.getInputStream());
		readStream(in);
	finally {
		urlConnection.disconnect();
	}


## 重定向

`HttpURLConnection`将最多支持五次HTTP重定向。包括重定向到不同服务器。但不支持HTTPS和HTTP之间的重定向。

## 响应

If the HTTP response indicates that an error occurred, getInputStream() will throw an IOException. Use getErrorStream() to read the error response. The headers can be read in the normal way using getHeaderFields(),

## 性能

`HttpURLConnection`返回的输入和输出流都是不缓存的。多数情况下，应该在返回的流外包裹一层`BufferedInputStream`或`BufferedOutputStream`。 Callers that do only bulk reads or writes may omit buffering.

When transferring large amounts of data to or from a server, use streams to limit how much data is in memory at once. Unless you need the entire body to be in memory at once, process it as a stream (rather than storing the complete body as a single byte array or string).

为了减少延迟，多个请求会重用底层的Socket。As a result, HTTP connections may be held open longer than necessary. 调用`disconnect()`将Socket交还给已a pool of connected sockets。发送请求前，设置系统属性`http.keepAlive`为false可以禁用该特性。The http.maxConnections property may be used to control how many idle connections to each server will be held.

默认`HttpURLConnection`请求时使用gzip压缩。Since getContentLength() returns the number of bytes transmitted, you cannot use that method to predict how many bytes can be read from getInputStream(). Instead, read that stream until it is exhausted: when read() returns -1. 可以禁用Gzip压缩：

	urlConnection.setRequestProperty("Accept-Encoding", "identity");

## 其他HTTP方法

HttpURLConnection uses the GET method by default. It will use POST if setDoOutput(true) has been called. Other HTTP methods (OPTIONS, HEAD, PUT, DELETE and TRACE) can be used with `setRequestMethod(String)`.