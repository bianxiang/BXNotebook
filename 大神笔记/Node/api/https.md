https

v4.0.0 Documentation

https://nodejs.org/api/https.html

## HTTPS

Stability: 2 - Stable

HTTPS is the HTTP protocol over TLS/SSL. In Node.js this is implemented as a separate module.

### Class: https.Server

This class is a subclass of `tls.Server` and emits events same as `http.Server`. See `http.Server` for more information.

#### server.setTimeout(msecs, callback)

See [http.Server#setTimeout()](https://nodejs.org/api/http.html#http_server_settimeout_msecs_callback).

#### server.timeout

See [http.Server#timeout](https://nodejs.org/api/http.html#http_server_timeout).

### https.createServer(options[, requestListener])

Returns a new HTTPS web server object. The options is similar to [tls.createServer()](https://nodejs.org/api/tls.html#tls_tls_createserver_options_secureconnectionlistener). The `requestListener` is a function which is automatically added to the 'request' event.

Example:

```js
// curl -k https://localhost:8000/
var https = require('https');
var fs = require('fs');

var options = {
  key: fs.readFileSync('test/fixtures/keys/agent2-key.pem'),
  cert: fs.readFileSync('test/fixtures/keys/agent2-cert.pem')
};

https.createServer(options, function (req, res) {
  res.writeHead(200);
  res.end("hello world\n");
}).listen(8000);
```

Or

```js
var https = require('https');
var fs = require('fs');

var options = {
  pfx: fs.readFileSync('server.pfx')
};

https.createServer(options, function (req, res) {
  res.writeHead(200);
  res.end("hello world\n");
}).listen(8000);
```

#### `server.listen(port[, host][, backlog][, callback])`

#### `server.listen(path[, callback])`

#### `server.listen(handle[, callback])`

See [http.listen()](https://nodejs.org/api/http.html#http_server_listen_port_hostname_backlog_callback) for details.

#### `server.close([callback])`

See [http.close()](https://nodejs.org/api/http.html#http_server_close_callback) for details.

### `https.request(options, callback)`

Makes a request to a secure web server.

`options` can be an object or a string. If `options` is a string, it is automatically parsed with [url.parse()](https://nodejs.org/api/url.html#url_url_parse_urlstr_parsequerystring_slashesdenotehost).

All options from [http.request()](https://nodejs.org/api/http.html#http_http_request_options_callback) are valid.

Example:

```js
var https = require('https');

var options = {
  hostname: 'encrypted.google.com',
  port: 443,
  path: '/',
  method: 'GET'
};

var req = https.request(options, function(res) {
  console.log("statusCode: ", res.statusCode);
  console.log("headers: ", res.headers);

  res.on('data', function(d) {
    process.stdout.write(d);
  });
});
req.end();

req.on('error', function(e) {
  console.error(e);
});
```

The `options` argument has the following options

- `host`: 要连接的域名或IP地址。默认为 `'localhost'`。
- `hostname`：`host`的别名。To support `url.parse()` hostname is preferred over `host`.
- `family`: IP address family to use when resolving `host` and `hostname`. Valid values are `4` or `6`. When unspecified, both IP v4 and v6 will be used.
- `port`: 远程服务器的端口。默认为`443`。
- `localAddress`: Local interface to bind for network connections.
- `socketPath`: Unix Domain Socket (use one of `host:port` or `socketPath`).
- `method`: A string specifying the HTTP request method. Defaults to `'GET'`.
- `path`: Request path. Defaults to `'/'`. Should include query string if any. E.G. `'/index.html?page=12'`. An exception is thrown when the request path contains illegal characters. Currently, only spaces are rejected but that may change in the future.
- `headers`：对象，包含请求头。
- `auth`: Basic authentication i.e. 'user:password' to compute an Authorization header.
- `agent`: Controls [Agent](https://nodejs.org/api/https.html#https_class_https_agent) behavior. When an Agent is used request will default to `Connection: keep-alive`. Possible values:
  - `undefined` (default): use [globalAgent](https://nodejs.org/api/https.html#https_https_globalagent) for this host and port.
  - Agent object: explicitly use the passed in Agent.
  - `false`: opts out of connection pooling with an Agent, defaults request to `Connection: close`.

The following options from [tls.connect()](https://nodejs.org/api/tls.html#tls_tls_connect_options_callback) can also be specified. However, a [globalAgent](https://nodejs.org/api/https.html#https_https_globalagent) silently ignores these.

- `pfx`: Certificate, Private key and CA certificates to use for SSL. Default null.
- `key`: Private key to use for SSL. Default null.
- `passphrase`: A string of passphrase for the private key or pfx. Default null.
- `cert`: Public x509 certificate to use. Default null.
- `ca`: An authority certificate or array of authority certificates to check the remote host against.
- `ciphers`: A string describing the ciphers to use or exclude. Consult http://www.openssl.org/docs/apps/ciphers.html#CIPHER_LIST_FORMAT for details on the format.
- `rejectUnauthorized`: If true, the server certificate is verified against the list of supplied CAs. An 'error' event is emitted if verification fails. Verification happens at the connection level, before the HTTP request is sent. Default true.
- `secureProtocol`: The SSL method to use, e.g. `SSLv3_method` to force SSL version 3. The possible values depend on your installation of OpenSSL and are defined in the constant [SSL_METHODS](http://www.openssl.org/docs/ssl/ssl.html#DEALING_WITH_PROTOCOL_METHODS).

In order to specify these options, use a custom `Agent`.

Example:

```js
var options = {
  hostname: 'encrypted.google.com',
  port: 443,
  path: '/',
  method: 'GET',
  key: fs.readFileSync('test/fixtures/keys/agent2-key.pem'),
  cert: fs.readFileSync('test/fixtures/keys/agent2-cert.pem')
};
options.agent = new https.Agent(options);

var req = https.request(options, function(res) {
  ...
}
```

Or does not use an `Agent`.

Example:

```js
var options = {
  hostname: 'encrypted.google.com',
  port: 443,
  path: '/',
  method: 'GET',
  key: fs.readFileSync('test/fixtures/keys/agent2-key.pem'),
  cert: fs.readFileSync('test/fixtures/keys/agent2-cert.pem'),
  agent: false
};

var req = https.request(options, function(res) {
  ...
}
```

### https.get(options, callback)

Like `http.get()` but for HTTPS.

`options` can be an object or a string. If `options` is a string, it is automatically parsed with `url.parse()`.

Example:

```js
var https = require('https');

https.get('https://encrypted.google.com/', function(res) {
  console.log("statusCode: ", res.statusCode);
  console.log("headers: ", res.headers);

  res.on('data', function(d) {
    process.stdout.write(d);
  });

}).on('error', function(e) {
  console.error(e);
});
```

### Class: https.Agent

An Agent object for HTTPS similar to [http.Agent](https://nodejs.org/api/http.html#http_class_http_agent). See [https.request()](https://nodejs.org/api/http.html#http_class_http_agent) for more information.

### https.globalAgent

Global instance of [https.Agent](https://nodejs.org/api/https.html#https_class_https_agent) for all HTTPS client requests.



