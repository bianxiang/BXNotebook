[toc]

## Ajax

### jQuery.ajax()

方法签名：`jQuery.ajax( url [, settings ] )`或`jQuery.ajax( [settings ] )`。

其中，`url`是字符串。`settings`是一个普通的Javascript对象。在下文中，里面的键值称为“选项”。所有的选项都是可选的。所有的选项都可以通过`$.ajaxSetup()`设置默认值。

url可以作为`jQuery.ajax()`的第一个参数参数，也可以通过选项`url`传入。

快速例子：

```js
$.ajax({
    type: "POST",
    url: "some.php",
    data: { name: "John", location: "Boston" }
}).done(function( msg ) {
    alert( "Data Saved: " + msg );
});
```

#### `jQuery.ajax()`返回`jqXHR`

新版jQuery的Ajax机制是Promise风格的。`jQuery.ajax()`返回 [jqXHR](http://api.jquery.com/Types/#jqXHR)。它是浏览器原生`XMLHttpRequest`对象的一个超集。例如，它增加了`responseText`和`responseXML`属性，`getResponseHeader()`方法。When the transport mechanism is something other than `XMLHttpRequest` (for example, a script tag for a JSONP request) the `jqXHR` object simulates native XHR functionality where possible. 

`jqXHR`对象实现了Promise接口。参见[Deferred object](http://api.jquery.com/category/deferred-object/)。


#### 请求URL、HTTP方法、超时

默认Ajax请求通过GET发送。可以通过`type`选项指定用POST等方法。POST传送数据只会使用UTF-8字符集（W3C XMLHTTPRequest标准）。

`url`选项。取值字符串。默认当前页面。A string containing the URL to which the request is sent.

`type`选项。取值字符串。默认值是'GET'。可用POST、PUT、DELETE等方法。

`timeout`选项：取值数字，单位毫秒。请求的超时。覆盖`$.ajaxSetup()`的全局设置。超时从`$.ajax`被调用时开始计算；如果有同时并发请求且浏览器已无可用连接，请求可能尚未发出就已超时。In jQuery 1.4.x and below, the XMLHttpRequest object will be in an invalid state if the request times out; accessing any object members may throw an exception. In Firefox 3.0+ only, script and JSONP requests cannot be cancelled by a timeout; the script will run even if it arrives after the timeout period.

#### 请求的内容、格式、编码

请求内容通过选项`data`设置。内容取一个查询字符串，如`key1=value1&key2=value2`，或键值对对象`{key1: 'value1', key2: 'value2'}`。后者将被`jQuery.param()`转换为查询字符串。

对于GET请求会追加到URL后。`processData`选项可以禁止这种自动转换。The processing might be undesirable if you wish to send an XML object to the server; in this case, change the `contentType` option from `application/x-www-form-urlencoded` to a more appropriate MIME type.

`processData`选项。默认值true。传给`data`选项的对象（非字符串），默认会被转换为一个查询字符串。按照默认的ContentType——"`application/x-www-form-urlencoded`"。If you want to send a DOMDocument, or other non-processed data, set this option to false.

对象必须以键值对形式传入。如果值是数组，jQuery serializes multiple values with same key based on the value of the `traditional` setting (described below).

`traditional`选项。取值布尔。Set this to true if you wish to use the traditional style of [param serialization.](http://api.jquery.com/jQuery.param/)

`contentType`选项。取值一个字符串。默认值是`application/x-www-form-urlencoded; charset=UTF-8`。When sending data to the server, use this content type. If you explicitly pass in a content-type to $.ajax(), then it is always sent to the server (even if no data is sent). W3C XMLHttpRequest规范指定字符集总是UTF-8；指向其他字符集并不会导致浏览器改变编码。

#### 拦截、修改请求

`beforeSend`选项。取值一个函数（`( jqXHR jqXHR, PlainObject settings )`）。可在发送前修改`jqXHR`。例如，用此功能实现定制请求头。`jqXHR`和`settings`会被传入参数。This is an Ajax Event. 函数返回false将取消请求。

`jqXHR.overrideMimeType()`可以在`beforeSend()`方法中，改变响应的**content-type**头：
```js
$.ajax({
    url: "http://fiddle.jshell.net/favicon.png",
    beforeSend: function( xhr ) {
        xhr.overrideMimeType( "text/plain; charset=x-user-defined" );
    }
}).done(function( data ) {
    if ( console && console.log ) {
        console.log( "Sample of data:", data.slice( 0, 100 ) );
    }
});
```

#### 设置请求头

`headers`选项。取值Javascript对象。默认值`{}`。An object of additional header key/value pairs to send along with requests using the XMLHttpRequest transport. 头`X-Requested-With: XMLHttpRequest`总是被添加，but its default XMLHttpRequest value can be changed here. Values in the headers setting can also be overwritten from within the `beforeSend` function. (version added: 1.5)

另一种方式，通过`beforeSend`修改：

```js
$.ajax({
    type:"POST",
    beforeSend: function (request) {
        request.setRequestHeader("Authority", authorizationToken);
    },
    ...
});
```

一些问题：http://stackoverflow.com/questions/7686827/how-can-i-add-a-custom-http-header-to-ajax-request-with-js-or-jquery。

#### 响应的格式、编码

响应在传给`success`回调前，根据其类型走不同的预处理环节。默认取决于响应的`Content-Type`。可以通过`dataType`选项显式设置，此时`Content-Type`会被忽略。

`dataType`指定期望服务器返回的类型。取一个字符串，包括：text, html, xml, json, jsonp, script。

- 若选定text或html，将不会有预处理。数据直接传给`success`回调，或可以访问`jqXHR.responseText`对象。当选定"html"时，HTML以纯文本的形式返回；把它插入到DOM时，内含的脚本标签会被求值。
- 若选定`xml`，响应先通过`jQuery.parseXML`转换为`XMLDocument`，然后再送给`success`回调。The XML document is made available through the responseXML property of the jqXHR object.
- 若选定`json`。响应被`jQuery.parseJSON`解析后送给`success`回调。The parsed JSON object is made available through the `responseJSON` property of the `jqXHR` object. The JSON data is parsed in a strict manner; any malformed JSON is rejected and a parse error is thrown. jQuery 1.9开始，空响应也会被拒绝；服务器应该返回`null`或`{}`。
- 若选定`script`，`$.ajax()`将先其作为脚本执行，然后再以纯文本的形式传给success回调。Disables caching by appending a query string parameter, "`_=[TIMESTAMP]`", to the URL unless the `cache` option is set to true. Note: This will turn POSTs into GETs for remote-domain requests.
- If `jsonp` is specified, `$.ajax()` will automatically append a query string parameter of (by default) `callback=?` to the URL. The jsonp and jsonpCallback properties of the settings passed to `$.ajax()` can be used to specify, respectively, the name of the query string parameter and the name of the JSONP callback function. The server should return valid JavaScript that passes the JSON response into the callback function. `$.ajax()` will execute the returned JavaScript, calling the JSONP callback function, before passing the JSON object contained in the response to the `$.ajax()` success handler. Adds an extra "`?callback=?`" to the end of your URL to specify the callback. Disables caching by appending a query string parameter, "`_=[TIMESTAMP]`", to the URL unless the `cache` option is set to true.
- 多个，逗号分隔的值：As of jQuery 1.5, jQuery can convert a dataType from what it received in the Content-Type header to what you require. 例如若想将响应文本当作XML，将`dataType`指定为"`text xml`"。You can also make a JSONP request, have it received as text, and interpreted by jQuery as XML: "`jsonp text xml`." Similarly, a shorthand string such as "jsonp xml" will first attempt to convert from jsonp to xml, and, failing that, convert from jsonp to text, and then from text to xml.

#### 响应的回调

有两种获取响应的回调的方式：通过选项指定回调函数，通过`jqXHR`的Promise方法。
选项包括`success`、`statusCode`、`error`、`complete`。
`jqXHR`的Promise方法包括两组。第一组包括，`jqXHR.success()`、`jqXHR.error()`、`jqXHR.complete()`。第二组包括`jqXHR.done()`、`jqXHR.fail()`、`jqXHR.always()`、`jqXHR.then()`。第一组已被废弃。第二组替换第一组的功能。

> 在请求完成后分配回调也是有效的，它们将被立即执行。

选项可以接收一个函数，或多个函数组成的数组数组，里面的函数会被依次调用。下面是选项的详解：

`success`选项。指定请求成功后被调用的函数。回调函数签名为`( PlainObject data, String textStatus, jqXHR jqXHR )`。第一个参数会按照`dataType`被格式化；This is an Ajax Event.

`error`选项。指定请求失败后被调用的函数。回调函数签名为`( jqXHR jqXHR, String textStatus, String errorThrown )`。第二个描述错误类型，取值（若非空）："timeout", "error", "abort", "parsererror", "No Transport"等。When an HTTP error occurs, `errorThrown` receives the textual portion of the HTTP status, such as "Not Found" or "Internal Server Error." 注意：跨域脚本和JSONP请求不会调用此函数。This is an Ajax Event.

`complete`选项。请求结束后调用此函数，不论成功失败。回调函数签名为`( jqXHR jqXHR, String textStatus )`。第二个参数取值为"success", "notmodified", "error", "timeout", "abort", "parsererror"。This is an Ajax Event.

`statusCode`选项。当出现特定的响应状态时调用的函数。取值一个Javascript对象。默认值`{}`。对于成功的状态的回调函数，参数与`success`回调参数相同。对于错误状态（包括3xx）的回调函数，参数与`error`回调的参数相同。：

```js
$.ajax({
    statusCode: {
    	404: function() {
    		alert( "page not found" );
    	}
    }
});
```

`jqXHR`可用的Promise方法包括：

`jqXHR.done(function( data, textStatus, jqXHR ) {})`：替代`success`回调选项，替换已废弃的`jqXHR.success()`方法。参见[deferred.done()](http://api.jquery.com/deferred.done/)。

`jqXHR.fail(function( jqXHR, textStatus, errorThrown ) {})`：替代`error`回调选项，替代已废弃的`jqXHR.error()`方法。参见[deferred.fail()](http://api.jquery.com/deferred.fail/)。

`jqXHR.always(function( data|jqXHR, textStatus, jqXHR|errorThrown ) { })`：替代`complete`回调选项。替换已废弃的`.complete()`方法。若请求成功，方法参数与`.done()`相同。若请求失败，方法参数与`.fail()`相同。参见[deferred.always()](http://api.jquery.com/deferred.always/)。

`jqXHR.then(function( data, textStatus, jqXHR ) {}, function( jqXHR, textStatus, errorThrown ) {})`：组合`.done()`和`.fail()`的功能，允许操纵底层Promise（1.8开始）。参见[deferred.then()](http://api.jquery.com/deferred.then/)。

`fail`、`done`、`always`回调允许注册多个回调。先进先出。参见[Deferred object methods](http://api.jquery.com/category/deferred-object/)。

```js
var jqxhr = $.ajax( "example.php" )
    .done(function() {
        alert( "success" );
    })
    .fail(function() {
        alert( "error" );
    })
    .always(function() {
        alert( "complete" );
    });

// Set another completion function for the request above
jqxhr.always(function() {
	alert( "second complete" );
});
```

#### 缓存

默认，请求总会被发出，但浏览器可能从缓存中返回结果。要禁用缓存，设置`cache`为false。To cause the request to report failure if the asset has not been modified since the last request, set `ifModified` to true.

- `cache`：（默认为true，dataType为'script'和'jsonp'时为false）。取布尔值。If set to false, it will force requested pages not to be cached by the browser. 注意：设置`cache`为false目前只对HEAD和GET请求有效。It works by appending "`_={timestamp}`" to the GET parameters. The parameter is not needed for other types of requests, except in IE8 when a POST is made to a URL that has already been requested by a GET.

#### 回调方法

所有回调方法的`this`指向`context`选项设置的对象。若未指定，`this`指向Ajax settings自己。

`context`选项。接受一个普通的对象。该对象将作为所有Ajax回调的上下文。By default, the context is an object that represents the ajax settings used in the call (`$.ajaxSettings` merged with the settings passed to `$.ajax`). For example, specifying a DOM element as the context will make that the context for the complete callback of a request, like so:

```js
$.ajax({
  url: "test.html",
  context: document.body
}).done(function() {
  $( this ).addClass( "done" );
});
```

`$.ajax()`提供以下回调：

- `beforeSend`回调选项；
- `dataFilter`回调选项；is invoked immediately upon successful receipt of response data. It receives the returned data and the value of dataType, and must return the (possibly altered) data to pass on to success.
- 回调选项：`success`、`error`、`complete`
- Promise回调：`.done()`, `.fail()`, `.always()`, `.then()`。

#### `jqXHR`

为了向后兼容`XMLHttpRequest`，`jqXHR`对象暴露以下属性和方法：

- `readyState`
- `status`
- `statusText`
- `responseXML` and/or `responseText` when the underlying request responded with xml and/or text, respectively
- `setRequestHeader(name, value)` which departs from the standard by replacing the old value with the new one rather than concatenating the new value to the old one
- `getAllResponseHeaders()`
- `getResponseHeader()`
- `statusCode()`
- `abort()`

No `onreadystatechange` mechanism is provided, however, since `done`, `fail`, `always`, and `statusCode` cover all conceivable requirements.

#### 安全

If the server performs HTTP authentication before providing a response, the user name and password pair can be sent via the `username` and `password` options.

- `username`：A username to be used with XMLHttpRequest in response to an HTTP access authentication request.
- `password`。取值字符串。A password to be used with XMLHttpRequest in response to an HTTP access authentication request.

#### 其他选项值

- `accepts`：默认值取决于`DataType`。值是一个普通对象。The content type sent in the request header that tells the server what kind of response it will accept in return.
- `async`：默认值true。取布尔值。默认所有请求都是异步的。If you need synchronous requests, set this option to `false`. 跨域和`dataType: "jsonp"`不支持同步请求。Note that synchronous requests may temporarily lock the browser, disabling any actions while the request is active. As of jQuery 1.8, the use of async: false with jqXHR ($.Deferred) is deprecated; you must use the success/error/complete callback options instead of the corresponding methods of the jqXHR object such as jqXHR.done() or the deprecated jqXHR.success().

- `contents`。Type: PlainObject。An object of string/regular-expression pairs that determine how jQuery will parse the response, given its content type. (version added: 1.5)
- `converters`。取值一个对象。默认值是`{"* text": window.String, "text html": true, "text json": jQuery.parseJSON, "text xml": jQuery.parseXML})`。数据类型到转换函数的映射。
- `dataFilter`：一个函数，用于处理`XMLHttpRequest`原始响应。类型：函数`( String data, String type ) => Object`。This is a pre-filtering function to sanitize the response. You should return the sanitized data. The function accepts two arguments: The raw data returned from the server and the '`dataType`' parameter.

- `crossDomain`。取值布尔。对于同域请求默认值为false。对于跨域请求默认值是true。If you wish to force a crossDomain request (such as JSONP) on the same domain, set the value of crossDomain to true. This allows, for example, server-side redirection to another domain. (version added: 1.5)

- `global`。是否触发全局对的Ajax事件处理器（如`ajaxStart`或`ajaxStop`）。

- `ifModified`。默认值false。Allow the request to be successful only if the response has changed since the last request. This is done by checking the `Last-Modified` header. Default value is false, ignoring the header. In jQuery 1.4 this technique also checks the '`etag`' specified by the server to catch unmodified data.
- `isLocal` ：取值布尔。(default: depends on current location protocol)。Allow the current environment to be recognized as "local," (e.g. the filesystem), even if jQuery does not recognize it as such by default. The following protocols are currently recognized as local: `file`, `*-extension`, and `widget`. If the `isLocal` setting needs modification, it is recommended to do so once in the `$.ajaxSetup()` method. (version added: 1.5.1)
- `jsonp`。类型字符串。Override the callback function name in a jsonp request. This value will be used instead of '`callback`' in the '`callback=?`' part of the query string in the url. So `{jsonp:'onJSONPLoad'}` would result in '`onJSONPLoad=?`' passed to the server. As of jQuery 1.5, setting the jsonp option to false prevents jQuery from adding the "?callback" string to the URL or attempting to use "=?" for transformation. In this case, you should also explicitly set the jsonpCallback setting. For example, { jsonp: false, jsonpCallback: "callbackName" }
- `jsonpCallback`。类型字符串或函数。Specify the callback function name for a JSONP request. This value will be used instead of the random name automatically generated by jQuery. It is preferable to let jQuery generate a unique name as it'll make it easier to manage the requests and provide callbacks and error handling. You may want to specify the callback when you want to enable better browser caching of GET requests. As of jQuery 1.5, you can also use a function for this setting, in which case the value of jsonpCallback is set to the return value of that function.
- `mimeType`。取值字符串。A mime type to override the XHR mime type. (version added: 1.5.1)

- `scriptCharset`：取值字符串。Only applies when the "script" transport is used (e.g., cross-domain requests with "jsonp" or "script" dataType and "GET" type). Sets the charset attribute on the script tag used in the request. Used when the character set on the local page is not the same as the one on the remote script.

- `xhr`： (default: ActiveXObject when available (IE), the XMLHttpRequest otherwise)。取值一个函数。函数用于创建`XMLHttpRequest`对象。Defaults to the ActiveXObject when available (IE), the XMLHttpRequest otherwise. Override to provide your own implementation for XMLHttpRequest or enhancements to the factory.
- `xhrFields`：取值普通Javascript对象。An object of fieldName-fieldValue pairs to set on the native XHR object. For example, you can use it to set `withCredentials` to true for cross-domain requests if needed.
	```
    $.ajax({
       url: a_cross_domain_url,
       xhrFields: {
          withCredentials: true
       }
    });
    ```
	In jQuery 1.5, the withCredentials property was not propagated to the native XHR and thus CORS requests requiring it would ignore this flag. For this reason, we recommend using jQuery 1.5.1+ should you require the use of it.

#### 高级

The global option prevents handlers registered using .ajaxSend(), .ajaxError(), and similar methods from firing when this request would trigger them. This can be useful to, for example, suppress a loading indicator that was implemented with .ajaxSend() if the requests are frequent and brief. With cross-domain script and JSONP requests, the global option is automatically set to false. See the descriptions of these methods below for more details. See the descriptions of these methods below for more details.

The `scriptCharset` allows the character set to be explicitly specified for requests that use a `<script>` tag (that is, a type of script or jsonp). This is useful if the script and host page have differing character sets.

The first letter in Ajax stands for "asynchronous," meaning that the operation occurs in parallel and the order of completion is not guaranteed. The `async` option to $.ajax() defaults to true, indicating that code execution can continue after the request is made. Setting this option to false (and thus making the call no longer asynchronous) is strongly discouraged, as it can cause the browser to become unresponsive.

The $.ajax() function returns the `XMLHttpRequest` object that it creates. Normally jQuery handles the creation of this object internally, but a custom function for manufacturing one can be specified using the xhr option. The returned object can generally be discarded, but does provide a lower-level interface for observing and manipulating the request. In particular, calling `.abort()` on the object will halt the request before it completes.

#### 扩展

As of jQuery 1.5, jQuery's Ajax implementation includes [prefilters](http://api.jquery.com/jQuery.ajaxPrefilter/), [transports](http://api.jquery.com/jQuery.ajaxTransport/), and converters that allow you to extend Ajax with a great deal of flexibility.

$.ajax() converters support mapping data types to other data types. If, however, you want to map a custom data type to a known type (e.g json), you must add a correspondance between the response Content-Type and the actual data type using the `contents` option:

```js
$.ajaxSetup({
  contents: {
    mycustomtype: /mycustomtype/
  },
  converters: {
    "mycustomtype json": function( result ) {
      // Do stuff
      return newresult;
    }
  }
});
```

This extra object is necessary because the response Content-Types and data types never have a strict one-to-one correspondance (hence the regular expression).

To convert from a supported type (e.g text, json) to a custom data type and back again, use another pass-through converter:

```js
$.ajaxSetup({
  contents: {
    mycustomtype: /mycustomtype/
  },
  converters: {
    "text mycustomtype": true,
    "mycustomtype json": function( result ) {
      // Do stuff
      return newresult;
    }
  }
});
```

The above now allows passing from text to mycustomtype and then mycustomtype to json.

#### 例子

```js
    $.ajax({
    	type: "POST",
    	url: "some.php",
    	data: { name: "John", location: "Boston" }
    }).done(function( msg ) {
    	alert( "Data Saved: " + msg );
    });

    $.ajax({
    	url: "test.html",
    	cache: false
    }).done(function( html ) {
    	$( "#results" ).append( html );
    });

    // Send an xml document as data to the server. By setting the processData option to false, the automatic conversion of data to strings is prevented.
    var xmlDocument = [create xml document];
    var xmlRequest = $.ajax({
    	url: "page.php",
    	processData: false,
    	data: xmlDocument
    });

    xmlRequest.done( handleResponse );
```

```js
    var menuId = $( "ul.nav" ).first().attr( "id" );
    var request = $.ajax({
        url: "script.php",
        type: "POST",
        data: { id : menuId },
        dataType: "html"
    });
    request.done(function( msg ) {
    	$( "#log" ).html( msg );
    });
    request.fail(function( jqXHR, textStatus ) {
    	alert( "Request failed: " + textStatus );
    });
```

Load and execute a JavaScript file.

```js
    $.ajax({
        type: "GET",
        url: "test.js",
        dataType: "script"
    });
```





























