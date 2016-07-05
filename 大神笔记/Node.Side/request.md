[toc]

https://github.com/request/request

### 最简单的使用Super simple to use

支持 HTTPS。自动跟随重定向。

```js
var request = require('request');
request('http://www.google.com', function (error, response, body) {
  if (!error && response.statusCode == 200) {
    console.log(body) // Show the HTML for the Google homepage.
  }
})
```

### 流

可以将响应流入一个文件。

```js
request('http://google.com/doodle.png').pipe(fs.createWriteStream('doodle.png'))
```

还可以将一个流，通过 PUT 或 POST 请求发送出去。This method will also check the file extension against a mapping of file extensions to content-types (in this case `application/json`) and use the proper content-type in the PUT request (if the headers don’t already provide one).

```js
fs.createReadStream('file.json').pipe(request.put('http://mysite.com/obj.json'))
```

Request can also pipe to itself. When doing so, `content-type` and `content-length` are preserved in the PUT headers.

```js
request.get('http://google.com/img.png').pipe(request.put('http://mysite.com/img.png'))
```

Request emits a "response" event when a response is received. The `response` argument will be an instance of `http.IncomingMessage`.

```js
request
  .get('http://google.com/img.png')
  .on('response', function(response) {
    console.log(response.statusCode) // 200
    console.log(response.headers['content-type']) // 'image/png'
  })
  .pipe(request.put('http://mysite.com/img.png'))
```

To easily handle errors when streaming requests, listen to the `error` event before piping:

```js
request
  .get('http://mysite.com/doodle.png')
  .on('error', function(err) {
    console.log(err)
  })
  .pipe(fs.createWriteStream('doodle.png'))
```

Now let’s get fancy.

```js
http.createServer(function (req, resp) {
  if (req.url === '/doodle.png') {
    if (req.method === 'PUT') {
      req.pipe(request.put('http://mysite.com/doodle.png'))
    } else if (req.method === 'GET' || req.method === 'HEAD') {
      request.get('http://mysite.com/doodle.png').pipe(resp)
    }
  }
})
```

You can also `pipe()` from `http.ServerRequest` instances, as well as to `http.ServerResponse` instances. The HTTP method, headers, and entity-body data will be sent. Which means that, if you don't really care about security, you can do:

```js
http.createServer(function (req, resp) {
  if (req.url === '/doodle.png') {
    var x = request('http://mysite.com/doodle.png')
    req.pipe(x)
    x.pipe(resp)
  }
})
```

And since `pipe()` returns the destination stream in ≥ Node 0.5.x you can do one line proxying. :)

```js
req.pipe(request('http://mysite.com/doodle.png')).pipe(resp)
```

Also, none of this new functionality conflicts with requests previous features, it just expands them.

```js
var r = request.defaults({'proxy':'http://localproxy.com'})

http.createServer(function (req, resp) {
  if (req.url === '/doodle.png') {
    r.get('http://google.com/doodle.png').pipe(resp)
  }
})
```

You can still use intermediate proxies, the requests will still follow HTTP forwards, etc.

### 表单

request supports `application/x-www-form-urlencoded` and `multipart/form-data` form uploads. For `multipart/related` refer to the `multipart` API.

**application/x-www-form-urlencoded (URL-Encoded Forms)**

URL-encoded forms are simple.

```js
request.post('http://service.com/upload', {form:{key:'value'}})
// or
request.post('http://service.com/upload').form({key:'value'})
// or
request.post({url:'http://service.com/upload', form: {key:'value'}},
	function(err,httpResponse,body){ /* ... */ })
```

**multipart/form-data (Multipart Form Uploads)**

For multipart/form-data we use the [form-data](https://github.com/felixge/node-form-data) library by @felixge. For the most cases, you can pass your upload form data via the `formData` option.

```js
    var formData = {
      // Pass a simple key-value pair
      my_field: 'my_value',
      // Pass data via Buffers
      my_buffer: new Buffer([1, 2, 3]),
      // Pass data via Streams
      my_file: fs.createReadStream(__dirname + '/unicycle.jpg'),
      // Pass multiple values /w an Array
      attachments: [
        fs.createReadStream(__dirname + '/attachment1.jpg'),
        fs.createReadStream(__dirname + '/attachment2.jpg')
      ],
      // Pass optional meta-data with an 'options' object with style: {value: DATA, options: OPTIONS}
      // Use case: for some types of streams, you'll need to provide "file"-related information manually.
      // See the `form-data` README for more information about options: https://github.com/felixge/node-form-data
      custom_file: {
        value:  fs.createReadStream('/dev/urandom'),
        options: {
          filename: 'topsecret.jpg',
          contentType: 'image/jpg'
        }
      }
    };
    request.post({url:'http://service.com/upload', formData: formData}, function optionalCallback(err, httpResponse, body) {
      if (err) {
        return console.error('upload failed:', err);
      }
      console.log('Upload successful!  Server responded with:', body);
    });
```

For advanced cases, you can access the form-data object itself via `r.form()`. This can be modified until the request is fired on the next cycle of the event-loop. (Note that this calling `form()` will clear the currently set form data for that request.)

```js
// NOTE: Advanced use-case, for normal use see 'formData' usage above
var r = request.post('http://service.com/upload', function optionalCallback(err, httpResponse, body) {...})
var form = r.form();
form.append('my_field', 'my_value');
form.append('my_buffer', new Buffer([1, 2, 3]));
form.append('custom_file', fs.createReadStream(__dirname + '/unicycle.jpg'), {filename: 'unicycle.jpg'});
```

See the [form-data README](https://github.com/felixge/node-form-data) for more information & examples.

**multipart/related**

Some variations in different HTTP implementations require a newline/CRLF before, after, or both before and after the boundary of a multipart/related request (using the multipart option). This has been observed in the .NET WebAPI version 4.0. You can turn on a boundary preambleCRLF or postamble by passing them as true to your request options.

```js
  request({
    method: 'PUT',
    preambleCRLF: true,
    postambleCRLF: true,
    uri: 'http://service.com/upload',
    multipart: [
      {
        'content-type': 'application/json'
        body: JSON.stringify({foo: 'bar', _attachments: {'message.txt': {follows: true, length: 18, 'content_type': 'text/plain' }}})
      },
      { body: 'I am an attachment' },
      { body: fs.createReadStream('image.png') }
    ],
    // alternatively pass an object containing additional options
    multipart: {
      chunked: false,
      data: [
        {
          'content-type': 'application/json',
          body: JSON.stringify({foo: 'bar', _attachments: {'message.txt': {follows: true, length: 18, 'content_type': 'text/plain' }}})
        },
        { body: 'I am an attachment' }
      ]
    }
  },
  function (error, response, body) {
    if (error) {
      return console.error('upload failed:', error);
    }
    console.log('Upload successful!  Server responded with:', body);
  })
```

### （未）HTTP身份验证

### 自定义HTTP头

HTTP Headers, such as `User-Agent`, can be set in the `options` object. In the example below, we call the github API to find out the number of stars and forks for the request repository. This requires a custom `User-Agent` header as well as **https**.

```js
var request = require('request');
var options = {
  url: 'https://api.github.com/repos/request/request',
  headers: {
    'User-Agent': 'request'
  }
};

function callback(error, response, body) {
  if (!error && response.statusCode == 200) {
    var info = JSON.parse(body);
    console.log(info.stargazers_count + " Stars");
    console.log(info.forks_count + " Forks");
  }
}

request(options, callback);
```

### （未）OAuth Signing

### （未）代理

### （未）UNIX Domain Sockets

### （未）TLS/SSL Protocol

### （未）Support for HAR 1.2

### `request(options, callback)`

第一个参数可以是`url`或`options`。唯一必需的选项是`uri`；其他都是可选的。

- `uri || url` - fully qualified uri or a parsed url object from `url.parse()`
- `baseUrl` - fully qualified uri string used as the base url. Most useful with `request.defaults`, for example when you want to do many requests to the same domain. If `baseUrl` is `https://example.com/api/`, then requesting `/end/point?test=tru`e will fetch `https://example.com/api/end/point?test=true`. When `baseUrl` is given, `uri` must also be a string.
- `method` - http method (default: "GET")
- `headers` - http headers (default: `{}`)

- `qs` - object containing querystring values to be appended to the uri
- `qsParseOptions` - object containing options to pass to the `qs.parse` method. Alternatively pass options to the `querystring.parse` method using this format `{sep:';', eq:':', options:{}}`
- `qsStringifyOptions` - object containing options to pass to the `qs.stringify` method. Alternatively pass options to the `querystring.stringify` method using this format `{sep:';', eq:':', options:{}}`. For example, to change the way arrays are converted to query strings using the qs module pass the `arrayFormat` option with one of `indices|brackets|repeat`
- `useQuerystring` - If true, use `querystring` to stringify and parse querystrings, otherwise use `qs` (default: false). Set this option to true if you need arrays to be serialized as foo=bar&foo=baz instead of the default foo[0]=bar&foo[1]=baz.

- `body` - entity body for PATCH, POST and PUT requests. Must be a `Buffer` or `String`, unless `json` is `true`. If json is true, then body must be a JSON-serializable object.
- `form` - when passed an object or a querystring, this sets body to a querystring representation of value, and adds `Content-type: application/x-www-form-urlencoded` header. When passed no options, a `FormData` instance is returned (and is piped to request). See "Forms" section above.
- `formData` - Data to pass for a multipart/form-data request. See Forms section above.
- `multipart` - array of objects which contain their own headers and body attributes. Sends a `multipart/related` request. See Forms section above.
  - Alternatively you can pass in an object {chunked: false, data: []} where chunked is used to specify whether the request is sent in chunked transfer encoding In non-chunked requests, data items with body streams are not allowed.
- `preambleCRLF` - append a newline/CRLF before the boundary of your multipart/form-data request.
- `postambleCRLF` - append a newline/CRLF at the end of the boundary of your multipart/form-data request.
- `json` - sets body but to JSON representation of value and adds `Content-type: application/json` header. Additionally, parses the response body as JSON.
- `jsonReviver` - a reviver function that will be passed to JSON.parse() when parsing a JSON response body.





