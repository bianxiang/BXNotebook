[toc]

## 6. 从移动应用访问用户数据

两类移动应用：使用WEB技术的，使用原生技术的。其中前者可以使用 客户端流 或 Web应用流，但原生应用需要特殊考虑。

### 6.1 原生移动应用为什么该用 OAuth

原生应用使用 OAuth 的两个主要原因是：

- **访问你自己的API**。有些应用需要服务端功能。一般通过 REST-based HTTP API 交互。对于这些应用，OAuth是处理API授权的好方式。而且实现一个接口供本地应用或Web访问。
- **使用其他提供商的API**。一些API提供商需要OAuth做API授权。

### 6.2 原生应用应该使用什么流

有时受限于服务提供商支持什么流。

The same considerations apply: Do you need long-lived “offline” access from your mobile app’s backend server? Use the server-side web app flow. Or do you need short-lived one-time access directly from the native app? Use the client-side implicit flow.

When using the server-side web app flow and passing an authorization code to your
server, the user of the app will still need to be authenticated to the app backend, similar to how a user is authenticated to web application servers using session cookies.

No: If your application does not have a mobile backend web server powering it, you
need to use some type of native application flow. This can be very similar to the serverside web app flow or the client-side implicit flow, but there are two restrictions: you don’t have a web server to use for the redirect_uri, and you should maintain the confidentiality of any `client_secret` values, which are sometimes required for the server-side flow.

Depending on the mobile platform you’re building on and the API provider you’re
using, you can use a custom URI scheme such as my-mobile-app://oauth/callback for
the `redirect_uri` in order to return the authorization code or `access_token` to your application. However, on some platforms, these custom URI schemes can be registered by multiple applications (and their uniqueness is not guaranteed), so there is a risk that the tokens could be intercepted by the wrong app on the device and used maliciously.

It’s also possible that your API provider requires preregistration of these `redirect_uri` values and does not accept values using custom URI schemes. There are also some API providers supporting a native client flow. With the native client flow, a special `redirect_uri` value is used to send the authorization code or access token to a web page hosted by the OAuth authorization server. The user can then copy/paste this value into the application or the application can programmatically grab the value
from the body or window title and close the web browser window.

The currently proposed special `redirect_uri` value for the native client flow is urn:ietf:wg:oauth:2.0:oob. Figure 6-1 shows an example result web page after the user approves access to their data. In most cases, the user would never see this page, however, because the application would grab the access token and close the window before it is visible.

### （未）6.3 The (Ugly) Web Browser

### （未）6.4 Enhanced Mobile App Authorization for Specific Providers
