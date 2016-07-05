[toc]

## 前言

Perhaps most exciting is the upcoming standardization of **OpenID Connect**—a protocol built on top of OAuth 2.0 to enable using the same identity to log in (authenticate) to multiple applications.

## 1. 介绍

### 术语

**Authentication**：*Authentication*是一个校验用户身份的过程：判定用户是否是他所声称的那个人。

**Federated Authentication**：尽管一些应用有自己的账户系统，另一些应用依赖其他服务验证用户身份。This is called federated authentication.

In a corporate IT environment, applications may trust an Active Directory server, a LDAP server, or a SAML provider to authenticate users.

On the Web, applications often **trust** OpenID providers (such as Google or Yahoo!) to handle the authentication of users. OpenID is the most common open web protocol for handling federated authentication.

尽管OpenID已经存在很多年了，本书只讨论*OpenID Connect*。后者是前者基于OAuth 2.0的下一代产品。

**Authorization**：Authorization是检查用户是否具有某种操作权限的过程。

**Delegated Authorization**：Delegated authorization is granting access to another person or application to perform actions on your behalf.
OAuth works similarly—a user grants access to an application to perform actions on the user’s behalf and the application can only perform the authorized actions.

### 签名的辩论

OAuth works similarly—a user grants access to an application to perform actions on the user’s behalf and the application can only perform the authorized actions. 很多开发者感觉复杂的加密机制是负担。

OAuth 2去掉了对复杂签名的要求。

One of the primary concerns with the elimination of signatures is that developers will not properly verify SSL/TLS certificate chains when making requests to the authorization and resource servers. This is required by the specification and addressed in the OAuth 2.0 threat model document, but the ease of disabling proper certificate and certificate authority validation in popular libraries, combined with the difficulty of fixing issues associated with it, has resulted in many developers taking shortcuts that threaten the security of their applications.

When implementing OAuth 2.0, calling any APIs, or using a library, you should verify that it properly handles SSL/TLS certificate chain validation by doing the following things:

- Checking that the hostname on the certificate returned by the server matches the hostname in the URL being accessed
- Verifying each certificate in the chain properly chains up to a valid and trusted certificate authority (CA)
- Ensuring that the certificate authority bundle on your server is secure and not able to be modified by potential attackers

#### 对你的OAuth 2.0请求签名

The MAC Access Authentication specification defines how clients can sign their OAuth 2.0 requests when signatures are supported or required by the API provider.

Author’s Note: If you’re thinking MAC only refers to a type of computer, you can understand why signatures are hard for many developers! You might want to find a good book on cryptography.

**Getting the key**

In order to sign requests using MAC authentication, the client must first get a MAC key. This can be issued by the OAuth authorization server. In this case, the key is returned each time an **access_token** is returned by the authorization server. This MAC key must be for use in either the hmac-sha-1 or hmac-sha-256 algorithms. Alternatively, the MAC key can be issued in an out-of-band process, such as when the developer registers their application with the API provider. Regardless of how the key is issued, it must always be issued over a secure SSL/TLS channel and must be kept confidential.

**Making API requests**

When connecting to OAuth-enabled APIs that require signatures, each API request must include a MAC signature in the Authorization header of the request. The process of generating this signature involves creating a normalized request string (nonce, HTTP method, request URI, host, port, optional body hash, etc.) and performing a crypto- graphic signature. It is highly recommended that developers use a prebuilt library to handle OAuth MAC signing if needed. If you need to build your own implementation, please see the specification, as the details are out of scope for this book.

### 客户端类型，Access Tokens，授权流程

#### 客户端类型

- 服务器端Web应用。此时OAuth客户端运行在服务器上。用户不知道客户端秘钥或任何access tokens。
- 客户端应用运行在一个Web浏览器中。此时OAuth客户端运行在用户的浏览器中，where the client has access to the application code and/or API requests. 应用代码一般是Javascript的。The OAuth credentials are not trusted to be kept confidential from the resource owner, so some API providers won’t issue client secrets for ap plications using this profile.
- Native应用。An OAuth client which is very similar to the client-side application, as the credentials are not trusted to be kept confidential. 应用可能不能访问Web浏览器完整的功能。

#### Access Tokens

Although signature-based MAC Access Authentication was mentioned earlier, most OAuth 2.0 authorized APIs require only bearer tokens to make authorized requests. Bearer tokens are a type of access token whereby simple possession of the token values provides access to protected resources. No additional information, such as a cryptographic key, is needed to make API calls.

使用OAuth的最终目标是一致的：要获得一个OAuth access token，凭此，你得应用可以代表用户或应用自己访问API。

获得access token后，你的请求可以**采用多种方式**携带这个token。推荐的方式是放入 HTTP Authorization 头：

	GET /tasks/v1/lists/@default/tasks HTTP/1.1
    Host: www.googleapis.com
    Authorization: Bearer ya29.AHES6ZSzX

之所以推荐这种方式，因为：

- The header is rarely logged by proxy servers and web server access logs.
- The header is almost never cached.
- The header doesn’t get stored in the browser cache when making requests from the client.

其他方法：

1、查询参数。使用请求参数`access_token`：

    https://www.googleapis.com/tasks/v1/lists/@default/tasks?callback=outputTasks&access_token=ya29.AHES6ZTh00gsAn4

2、表单编码的body参数

#### 授权流程

The core OAuth 2.0 protocol defines four primary “grant types” used for obtaining authorization and also defines an extension mechanism for enabling additional grant types.

- Authorization code。这种授予流程最适合服务器端Web应用。在资源拥有者授权访问他们的数据后，他们被重定向到**Web应用**，URL中含有一个**authorization code**。**客户端应用**必须用这个码交换一个**access token**。交换过程是服务器与服务器之间进行的，需要`client_id`和`client_secret`。资源拥有者最终并不能知晓**access token**。这种授予类型允许通过**refresh tokens**获得对API的长期访问。

- 隐式授予，用于基于浏览器的客户端应用。隐式授权是所有方式中最简单的。资源所有者授权应用访问，一个新的**access token**立即传回应用，以URL中#hash片段的形式。应用可以（利用Javascript）从hash fragment中抽取出**access token**，然后调用API。这种授予类型不要中间的**authorization code**，但也不能获得**refresh tokens**。

- 资源拥有者基于密码的授予。This grant type enables a resource owner’s username and password to be exchanged for an OAuth access token. 这种方式只对高度受信的客户端有意义。例如API提供者自己写的手机应用。尽管用户的密码暴露给了客户端，但不需要在设备上保存。在完成身份验证后，只有OAuth token需要存下来。

- Client credentials。The client credentials grant type allows an application to obtain an access token for resources owned by the client or when authorization has been “previously arranged with an authorization server.” This grant type is appropriate for applications that need to access APIs, such as storage services or databases, on behalf of themselves rather than on behalf of a specific user.

These additional flows are defined outside of the core spec:


- Device profile。这种方式用于没有内建的浏览器或输入受限的情况（如只有手柄）。用户在设备上发起流。然后告诉用户使用电脑访问一个网站，输入设备上显示的一个**authorization code**来授权。Facebook has a great example of this flow referenced in its documentation.

- SAML bearer assertion profile。This profile enables exchanging SAML 2.0 assertion for an OAuth access token. This is useful in enterprise environments that already have SAML authorization servers set up to control application and data access.









