[toc]

## 7. OpenID Connect

OpenID Connect 是 OpenID 的下一代版本。OpenID Connect 的开发考虑了两个概念：

- 授予网站访问用户身份信息（用户标示）的权限，与授予访问用户其他信息（如日历）是类似的。这两种情况不应使用不同协议。
- 协议应该是模块化的The specification should be modular — enabling spec compliance without requiring implementation of automated discovery, associations, and other complex bits included in the previous versions of OpenID.

OpenID Connect 的基本流程如下：

1. The application requests OAuth 2.0 authorization for one or more of the OpenID Connect **scopes** (openid, profile, email, address) by redirecting the user to an identity provider.
2. 用户批准 OAuth 权限请求后，用户的浏览器重定向回到应用（传统 OAuth 流程）。用户应用请求 Check ID Endpoint。此端点返回用户的身份（`user_id`），及其他信息，如 aud 和 state，which must be verified **by the client** to ensure valid authentication.
3. 若客户端请求更多用户信息，如用户的全名，照片，则客户端需要再请求 UserInfo Endpoint。

因为 OpenID Connect 构造在 OAuth 2.0 之上，并且设计成模块化的规范，it’s much easier for you to implement federated authentication for your website in a compliant way. Since this is a Getting Started book, this chapter will primarily discuss the [OpenID Connect Basic Client](http://openid.net/specs/openid-connect-basic-1_0.html) implementation.

### 7.1 ID Token

With OpenID Connect authentication, 还有一种类型的 OAuth token：ID token。ID token 或 `id_token` 标示被校验的用户的身份。它独立于 access token，which is used to retrieve the user’s profile information or other user data requested during the same authorization flow.

ID token 是一个 JSON Web Token (JWT)，which is a digitally signed and/or encrypted representation of the user’s identity asserted by the identity provider. Instead of using cryptographic operations to validate the JSON Web Token, it can be treated as an opaque string and passed to the Check ID Endpoint for interpretation (see below). This flexibility keeps with the spirit of OAuth 2.0 and OpenID Connect being significantly easier to use than their predecessors.

### 7.2 安全属性

Although the end user flow is quite similar, the security precautions necessary for authentication are much different than those for authorization because of the potential for replay attacks. Replay attacks occur when legitimate credentials are sent multiple times for malicious purposes.

我们主要预防两类重放冲击：

- An attacker capturing an user’s OAuth credentials as they log in to a site and using them later on the same site.
- A rogue application developer using the OAuth token a user was issued to login to their malicious app in order to impersonate the user on a different legitimate app.

OAuth 2.0 规范要求 OAuth endpoint 和 APIs 用 SSL/TLS 方法，防止中间人攻击，如第一种情况。

Preventing rogue application developers from replaying legitimate OAuth credentials their app received in order to impersonate one of their users on another app requires a solution specific to OpenID Connect. This solution is the Check ID Endpoint. The Check ID Endpoint is used to verify that the credentials issued by the OAuth provider were issued to the correct application.

It is recommended that all developers use the Check ID Endpoint or decode the JSON Web Token to verify the asserted identity, though this is not strictly necessary in some cases when the application uses the server-side Web Application flow and the UserInfo Endpoint provides all required information.

The server-side Web Application flow, when implemented as per the specification, only issues an authorization code through the user’s web browser. The web application should not ever accept an access token or identity token directly from the browser. The access token and identity token are retrieved by exchanging the authorization code in a server-to-server request. Since this exchange requires the server-to-server call to be authenticated with the client ID and client secret of the app which the authorization code was issued for, the OAuth token service will naturally prevent an app from accidentally using an authorization code issued to another app.

Alternatively, the client-side Web Application flow issues an access token and identity token directly to the app through the browser using a hash fragment. The access token and identity token are often sent to the backend web server using JavaScript in order to authenticate the user. In this case, the web server must either cryptographically verify the ID Token or call the Check ID endpoint to verify it was issued to the correct application. This is called “verifying the audience” of the token. See “Check ID Endpoint” on page 54 for more information.

### 7.3 获得用户授权

获取用户对 OpenID Connect 授权的过程，与获取用户对其他 OAuth 2.0 API 授权是一样的。可以使用客户端隐式流或服务器端web流。

客户端产生一个 URL，指向 OAuth Authorization Endpoint，把用户重定向到这个 URL。需要传以下参数：


- `client_id`
- `redirect_uri`：匹配授权请求后用户返回的地方。
- `scope`：对于基本的 OpenID Connect 请求，传 `openid`。若需要更多信息，可以增加下面几个值，空格分隔：`profile`、`email`、`address`。
- `response_type`：`id_token` to indicate that an `id_token` is required for the application. Additionally, a response type of `token` or `code` must be included, separating the two response types by a space. `token` indicates the client-side Web Application flow, while `code` indicates the server-side Web Application flow.
- `nonce`：A unique value used by your application to protect against replay and cross-site request forgery (CSRF) attacks on your implementation. The value should be a random unique string for this **particular** request, unguessable and kept secret in the client (perhaps in a server-side session). This identical value will be included in the ID token response (see below).

下面是一个完整的 Authorization Endpoint URL 的例子，使用客户端隐式流：

    https://accounts.example.com/oauth2/auth?
    scope=openid+email&
    nonce=53f2495d7b435ac571&
    redirect_uri=https%3A%2F%2Foauth2demo.appspot.com%2Foauthcallback&
    response_type=id_token+token&
    client_id=753560681145-2ik2j3snsvbs80ijdi8.apps.googleusercontent.com

After the user approves the authentication request, they will be redirected back to the `redirect_uri`. Since this request uses the implicit flow, the redirect will include an access token that can be used with the UserInfo Endpoint to obtain profile information about the user. Additionally, and specific to OpenID Connect, the redirect will also include an `id_token`, which can be sent to the **Check ID Endpoint** to get the user’s identity.

Here’s an example redirect:

    https://oauth2demo.appspot.com/oauthcallback#
    access_token=ya29.AHES6ZSzX
    token_type=Bearer&
    expires_in=3600&
    id_token=eyJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJhY2NvdW50cy5nb29nbGUuY29tIiwiY...

The client then needs to parse the appropriate parameters from the hash fragment in the URL and call the **Check ID Endpoint** to **validate** the response.

### 7.4 Check ID Endpoint

The Check ID Endpoint exists to validate the `id_token` returned along with the OAuth 2.0 `access_token` by ensuring that it was intended for the correct client and is used by the client to begin an authenticated session. As described above, this check is required for the implicit flow for client-side applications (described in Chapter 3). 若不能正确执行此检查，客户端容易被重放攻击。

Here’s an example Check ID endpoint request:

    https://accounts.example.com/oauth2/tokeninfo?
    id_token=eyJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJhY2NvdW50cy5nb29nbGUuY29tIiwiY...

And the response:

    {
    "iss" : "https://accounts.example.com",
    "user_id" : "113487456102835830811",
    "aud" : "753560681145-2ik2j3snsvbs80ijdi8.apps.googleusercontent.com",
    "exp" : 1311281970
    "nonce" : 53f2495d7b435ac571
    }

If the response is returned without a standard OAuth 2.0 error, the following checks
need to be performed:

- Verify the `aud` value in the response is identical to the `client_id` used in the Authorization request.
- Verify that the `nonce` value in the response matches the value used in the Authorization request.

If this verification is completed successfully, the `user_id` is known to represent the unique identifier for the authenticated user, within the scope of the issuer (iss). If storing the identifier in a user database table and multiple identity providers are supported by your application, it is recommended that both values be stored upon account creation and queried upon each subsequent authentication request.

### 7.5 UserInfo Endpoint

While the Check ID Endpoint will return a unique identifier for the user authenticating to your application, many applications require additional information, such as the user’s name, email address, profile photo, or birthdate. This profile information can be returned by the UserInfo Endpoint.

The UserInfo Endpoint is a standard OAuth-authorized REST API, with JSON responses.
As when accessing any other API using OAuth, the `access_token` can be passed either as an Authorization header or as a URL query parameter.

Here’s an example UserInfo request:

    GET /v1/userinfo HTTP/1.1
    Host: accounts.example.com
    Authorization: Bearer ya29.AHES6ZSzX

With the response:

    {
    "user_id": "3191142839810811",
    "name": "Example User",
    "given_name": "Example",
    "family_name": "User",
    "email": "user@example.com",
    "verified": true,
    "profile": "http://profiles.example.com/user",
    "picture": "https://photos.profiles.example.com/user/photo.jpg",
    "gender": "female",
    "birthday": "1982-02-11",
    "locale": "en-US"
    }

OpenID Connect does not define any specific profile fields as required and does allow for additional profile fields to be included in the response.

### 7.6 性能提示

The objective of the call to the Check ID Endpoint is to verify the legitimacy of the `id_token`. However, this requires an additional HTTP request to the OpenID Connect identity provider. This additional request can be avoided since the `id_token` is returned as a **signed JSON Web Token (JWT)** instead of as an opaque blob. The JWT includes the same information that is typically returned by the Check ID Endpoint, but the value is also cryptographically signed by the server in a way that can be validated by the client.

This gives the client the option to verify the signature using the JWT (for best performance) or simply call the Check ID Endpoint if the client wants to avoid cryptography.

### （未）7.7 现实中的 OpenID Connect






