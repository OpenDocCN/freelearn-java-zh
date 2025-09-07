

# 第五章：使用授权和认证保护 REST 端点

在前面的章节中，我们使用命令式和响应式编程风格开发了一个 RESTful Web 服务。现在，您将学习如何使用 Spring Security 保护这些 REST 端点。您将为 REST 端点实现基于令牌的认证和授权。成功的认证会提供两种类型的令牌——一个作为访问令牌的 **JavaScript 对象表示法**（**JSON**）**Web 令牌**（**JWT**），以及一个响应中的刷新令牌。这个基于 JWT 的访问令牌随后用于访问受保护的 **统一资源定位符**（**URL**）。刷新令牌用于在现有 JWT 过期时请求新的 JWT，一个有效的请求令牌会提供一个新 JWT 以供使用。

您将把用户关联到如 *admin* 和 *user* 这样的角色。这些角色将用作授权，以确保只有当用户拥有某些角色时才能访问 REST 端点。我们还将简要讨论 **跨站请求伪造**（**CSRF**）和 **跨源资源共享**（**CORS**）。

本章的主题分为以下几节：

+   使用 Spring Security 和 JWT 实现认证

+   使用 JWT 保护 REST API

+   配置 CORS 和 CSRF

+   理解授权

+   测试安全性

到本章结束时，您将了解如何使用 Spring Security 实现认证和授权，并保护您的 Web 服务免受 CORS 和 CSRF 攻击。

# 技术要求

本章的代码可在 [`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter06`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter06) 找到。

# 使用 Spring Security 和 JWT 实现认证

Spring Security 是一个由一系列库组成的框架，它允许您在不担心编写样板代码的情况下实现企业级应用的安全性。在本章中，我们将使用 Spring Security 框架来实现基于令牌（JWT）的认证和授权。在本章的整个过程中，您还将了解 CORS 和 CSRF 配置。

了解 Spring Security 也为不透明令牌提供支持，就像它为 JWT 提供支持一样。它们之间的主要区别在于从令牌中读取信息的方式。您不能像从 JWT 中那样读取不透明令牌的信息——只有发行者知道如何这样做。

注意

令牌是一系列字符，例如

`5rm1tc1obfshrm2354lu9dlt5reqm1ddjchqh81 7rbk37q95b768bib0j``f44df6suk1638sf78cef7` `hfolg4ap3bkighbnk7inr68ke780744fpej0gtd 9qflm999o8q`。

它允许您通过使用各种授权流程来调用受保护的无状态 HTTP 端点或资源。

`You learned about `DispatcherServlet` in *Chapter 2*, *Spring Concepts and REST APIs*. This is an interface between a client request and the REST controller. Therefore, if you want to place logic for token-based authentication and authorization, you will have to do this before a request reaches `DispatcherServlet`. Spring Security libraries provide the servlet with pre-filters (as a part of the filter chain), which are processed before the request reaches `DispatcherServlet`. A `DispatcherServlet`. Similarly, **post-filters** get processed after a request has been processed by the servlet/controller.

`There are two ways you can implement token-based (JWT) authentication – by using either `spring-boot-starter-security` or `spring-boot-starter-oauth2-resource-server`. We will use the latter because it does the boilerplate configuration for us.`

前者包含以下库：

+   ``spring-security-core```

+   ``spring-security-config```

+   ``spring-security-web```

``spring-boot-starter-oauth2-resource-server``` 提供以下内容，以及所有三个先前的 **Java ARchive** (**JAR**) 文件：

+   ``spring-security-oauth2-core```

+   ``spring-security-oauth2-jose```

+   ``spring-security-oauth2-resource-server```

`When you start this chapter’s code, you will find the following log. You can see that, by default, `DefaultSecurityFilterChain` is `auto-configured`. The `log` statement lists the configured filters in `DefaultSecurityFilterChain`, as shown in the following log block:`

```java
INFO [Chapter06,,,] [null] [null] [null] [null] 31975 --- [           main] o.s.s.web.DefaultSecurityFilterChain:

1.  `WebAsyncManagerIntegrationFilter`
2.  `SecurityContextPersistenceFilter`
3.  `HeaderWriterFilter`
4.  `CorsFilter`
5.  `CsrfFilter`
6.  `LogoutFilter`
7.  `BearerTokenAuthenticationFilter`
8.  `RequestCacheAwareFilter`
9.  `SecurityContextHolderAwareRequestFilter`
10.  `AnonymousAuthenticationFilter`
11.  `SessionManagementFilter`
12.  `ExceptionTranslationFilter`
13.  `FilterSecurityInterceptor`

This filter chain may change in future releases. Also, the security filter chain will be different if you just used `spring-boot-starter-security` or changed the configuration. You can find all the filters available in `springSecurityFilterChain` at [`docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-security-filters`](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-security-filters).
Now, you know about the different filters and their order in the default security chain. Next, let’s add the required dependencies, making use of the Spring OAuth 2.0 resource server for authentication in the following subsections.
Adding the required Gradle dependencies
Let’s add the following dependencies to the `build.gradle` file, as shown next:

```

`implementation 'org.springframework.boot:spring-boot-starter-oauth2-resource-server' implementation 'com.auth0:java-jwt:4.3.0'`

```java

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/build.gradle`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/build.gradle)
The Spring Boot Starter OAuth 2 resource server dependency will add the following JARs:

*   `spring-security-core`
*   `spring-security-config`
*   `spring-security-web`
*   `spring-security-cropto`
*   `spring-security-oauth2-core`
*   `spring-security-oauth2-jose`
*   `spring-security-oauth2-resource-server`

For JWT implementation, we will use the `java-jwt` library from [auth0.com](http://auth0.com).
We will now explore how to code these two filters – through login and token-based authentication.
Authentication using the OAuth 2.0 resource server
The Spring Security OAuth 2.0 resource server allows you to implement authentication and authorization using `BearerTokenAuthenticationFilter`. This contains the bearer token authentication logic. However, you still need to write the REST endpoint to generate the token. Let’s explore how the authentication flow works in the OAuth2.0 resource server. Take a look at the following diagram:
![Figure 6.1 – A token authentication flow using the OAuth 2.0 resource server](img/Figure_06.1_B19349.jpg)

Figure 6.1 – A token authentication flow using the OAuth 2.0 resource server
Let’s understand the flow depicted in *Figure 6**.1*:

1.  The client sends a `GET` HTTP request to `/api/v1/addresses`.
2.  `BearerTokenAuthenticationFilter` comes into action. If the request doesn’t contain the `Authorization` header, then `BearerTokenAuthenticationFilter` does not authenticate the request since it did not find the bearer token. It passes the call to `FilterSecurityInterceptor`, which does the authorization. It throws an `AccessDeniedException` exception (marked as `ExceptionTranslationFilter` springs into action. Control is moved to `BearerTokenAuthenticationEntryPoint`, which responds with a `401 Unauthorized` status and a `WWW-Authenticate` header, with a `Bearer` value. If the client receives a `WWW-Authenticate` header with a `Bearer` value in response, it means it must retry with the `Authorization` header that holds the valid bearer token. At this stage, the request cache is `NullRequestCache` (that is, empty) due to security reasons – the client can replay the request.
3.  Let’s assume the HTTP request contains an `Authorization` header. It extracts the `Authorization` header from the HTTP request and, apparently, the token from the `Authorization` header. It creates an instance of `BearerTokenAuthentication`` Token` using the token value. `BearerTokenAuthenticationToken` is a type of `AbstractAuthenticationToken` class that implements an `Authentication` interface, representing the token/principal for the authenticated request.
4.  The HTTP request is passed to `AuthenticationManagerResolver`, which provides the `AuthenticationManager` based on the configuration. `AuthenticationManager` verifies the `BearerTokenAuthenticationToken` token.
5.  If authentication is successful, then `Authentication` is set on the `SecurityContext` instance. This instance is then passed to `SecurityContextHolder.setContext()`. The request is passed to the remaining filters for processing, then routes to `DispatcherServlet`, and then, finally, to `AddressController`.
6.  If authentication fails, then `SecurityContextHolder.clearContext()` is called to clear the context value. `ExceptionTranslationFilter` springs into action. Control is moved to `BearerTokenAuthenticationEntryPoint`, which responds with a `401 Unauthorized` status and a `WWW-Authenticate` header, with a value that contains the appropriate error message, such as the following:

    ```

    `Bearer error="invalid_token", error_description="An error occurred while attempting to decode the Jwt: Jwt expired at 2022-12-14T17:23:30Z", error_uri="https://tools.ietf. org/html/rfc6750#section-3.1".`

    ```java

Now that you have learned about the complete authentication flow using the OAuth 2.0 resource server, let’s learn the fundamentals of JWT.
Exploring the structure of JWT
You need authority in the form of permissions or rights to carry out any activity or access any information in general. This authority is known as a claim with respect to JWT. A claim is represented as a key-value pair. The key contains the claim name and the value contains the claim, which can be a valid JSON value. A claim can also be metadata about the JWT.
How to pronounce JWT
As per [`tools.ietf.org/html/rfc7519`](https://tools.ietf.org/html/rfc7519), the suggested pronunciation of *JWT* is the same as the English word *jot*.
A JWT is an encoded string that contains a set of claims. These claims are either digitally signed by a **JSON Web Signature** (**JWS**) or encrypted by **JSON Web Encryption** (**JWE**). *JWT is a self-contained way to transmit claims securely between parties*. The links to these **Request for Comments** (**RFC**)-proposed standards are provided in the *Further reading* section of this chapter.
A JWT is an encoded string such as `aaa.bbb.ccc`, consisting of the following three parts, separated by dots (`.`):

*   Header
*   Payload
*   Signature

A few websites, such as [`jwt.io`](https://jwt.io), allow you to view the content of a JWT and generate one.
Let’s have a look at the following sample JWT string. You can paste it into one of the aforementioned websites to decode the content:

```

`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c`

```java

 This sample token demonstrates how a JWT is formed and divided into three parts using dots.
Header
A `typ` key) and a signing algorithm (with an `alg` key).
This sample JWT string contains the following header:

```

`{  "alg": "HS256",`

`"typ": "JWT"`

`}`

```java

The preceding header contains the `typ` and `alg` fields, representing the type and algorithm, respectively.
Payload
A **payload** is the second part of the JWT, which contains the claims and also comprises a Base64URL-encoded JSON string. There are three types of claims – registered, public, and private. These are outlined as follows:

*   `iss` key): This claim identifies the principal who issued a token*   `sub` key): This should be a unique value that represents the subject of the JWT*   `exp` key): This is a numeric value representing the expiration time on or after which a JWT should be rejected*   `iat` key): This claim identifies the time at which a JWT is issued*   `jti` key): This claim represents the unique identifier for a JWT*   `aud` key): This claim identifies the recipients, which JWT is intended for*   `nbf` key): This represents the time before which a JWT must be rejected
**Public claims**: These are defined by JWT issuers and must not collide with registered claims. Therefore, these should either be registered with the IANA JWT Claims registry or defined as a URI with a collision-resistant namespace.
**Private claims**: These are custom claims defined and used by the issuer and audience. They are neither registered nor public.

Here is a sample JWT string containing a payload:

```

`{  "sub": "scott2",`

`"roles": [`

`"ADMIN"`

`],`

`"iss": "Modern API Development with Spring and`

`Spring Boot`

`"exp": 1676526792,`

`"iat": 1676525892`

`}`

```java

The preceding payload contains `sub` (subject), `iss` (issuer), `roles` (custom claim roles), `exp` (expires), `iat` (issued at), and `jti` (JWT ID) fields.
Signature
A **signature** is also a Base64-encoded string and makes up the third part of a JWT-encoded string. A signature is there to protect the content of the JWT. The content is visible but cannot be modified if the token is signed. A Base64-encoded header and payload are passed to the signature’s algorithm, along with either a secret or a public key to make the token a signed token. If you wish to include any sensitive or secret information in the payload, then it’s better to encrypt it before assigning it to the payload.
A signature makes sure that the content is not modified once it is received. The use of a public/private key enhances the security step by verifying the sender.
You can use a combination of both a JWT and JWE. However, the recommended way is to first encrypt the payload using JWE and then sign it.
We’ll use public/private keys to sign the token in the next section. Let’s jump into coding in the next section.
Securing REST APIs with JWT
In this section, you’ll secure the REST endpoints exposed in *Chapter 4*, *Writing Business Logic for APIs*. Therefore, we’ll use the code from *Chapter 4* and enhance it to secure the APIs.
The REST APIs should be protected using the following techniques:

*   No secure API should be accessed without a JWT.
*   A JWT can be generated using sign-in/sign-up or a refresh token.
*   A JWT and a refresh token should only be provided for a valid user’s username/password combination or a valid user sign-up.
*   The password should be stored in an encoded format using a `bcrypt` strong hashing function.
*   The JWT should be signed with **Rivest-Shamir-Adleman** (**RSA**) keys with a strong algorithm.

RSA
RSA is an algorithm approved by the **Federal Information Processing Standards**(**FIPS**) (FIPS 186) for digital signatures and in **Special Publication** (**SP**) (SP800-56B) for key establishment.

*   Claims in the payload should not store sensitive or secured information. If they do, then they should be encrypted.
*   You should be able to authorize API access for certain roles.

We need to include new APIs for the authorization flow. Let’s add them first.
Adding new APIs
You will enhance the existing APIs by adding four new APIs – sign-up, sign-in, sign-out, and a refresh token. The sign-up, sign-in, and sign-out operations are self-explanatory.
The refresh token provides a new access token (JWT) once the existing token expires. This is the reason why the sign-up/sign-in API provides two types of tokens – an access token and a refresh token as a part of its response. The JWT access token self-expires; therefore, a sign-out operation would only remove the refresh token.
Let’s add these APIs to the `openapi.yaml` document next.
Apart from adding the new APIs, you also need to add a new user tag for these APIs that will expose all these APIs through the `UserApi` interface. Let’s first add a sign-up endpoint.
Sign-up endpoint
Add the following specification for the sign-up endpoint in `openapi.yaml`:

```

`/api/v1/users:  post:`

`tags:`

`- user`

`summary: Signup the a new customer (user)`

`description: Creates a new customer (user)`

`operationId: signUp`

`requestBody:`

`content:`

`application/xml:`

`schema:`

`$ref: '#/components/schemas/User'`

`application/json:`

`schema:`

`$ref: '#/components/schemas/User'`

`responses:`

`201:`

`description: For successful user creation.`

`content:`

`application/xml:`

`schema:`

`$ref: '#/components/schemas/SignedInUser'`

`application/json:`

`schema:`

`$ref: '#/components/schemas/SignedInUser'`

```java

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/resources/api/openapi.yaml`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/resources/api/openapi.yaml)
The sign-up API call returns the new `SignedInUser` model. This contains `accessToken`, `refreshToken`, `username`, and user ID fields. The code to add the model is shown in the following snippet:

```

`SignedInUser:  description: Signed-in user information`

`type: object`

`properties:`

`refreshToken:`

`description: Refresh Token`

`type: string`

`accessToken:`

`description: JWT Token aka access token`

`type: string`

`username:`

`description: User Name`

`type: string`

`userId:`

`description: User Identifier`

`type: string`

```java

Now, let’s add the sign-in endpoint.
Sign-in endpoint definition
Add the following specification for the sign-in endpoint to `openapi.yaml`:

```

`/api/v1/auth/token:  post:`

`tags:`

`- user`

summary: Signin the customer (user)

description: Generates the JWT and refresh token

operationId: signIn

requestBody:

content:

application/xml:

schema:

$ref: '#/components/schemas/SignInReq'

application/json:

schema:

$ref: '#/components/schemas/SignInReq'

responses:

200:

description: Returns the access and refresh token.

content:

application/xml:

schema:

$ref: '#/components/schemas/SignedInUser'

application/json:

schema:

$ref: '#/components/schemas/SignedInUser'

```java

The sign-in API uses the new request payload object – `SignInReq`. The object just contains the username and password fields. Let’s add it, as follows:

```

SignInReq:  description: Request body for Sign-in

type: object

properties:

username:

description: username of the User

type: string

password:

description: password of the User

type: string

```java

Now, let’s add the sign-out endpoint.
Sign-out endpoint
Add the following specification for the sign-out endpoint to `openapi.yaml`:

```

# Under the /api/v1/auth/tokendelete:

tags:

- user

summary: Signouts the customer (user)

description: Signouts the customer (user).

operationId: signOut

requestBody:

content:

application/xml:

schema:

$ref: '#/components/schemas/RefreshToken'

application/json:

schema:

$ref: '#/components/schemas/RefreshToken'

responses:

202:

description: Accepts the request for logout.

```java

In an ideal scenario, you should remove the refresh token of a user received from the request. You can fetch the user ID from the token itself and then use that ID to remove the refresh token from the `USER_TOKEN` table. This endpoint requires you to send a valid access token.
We have opted for an easy way to remove the token, which is for it to be sent by the user as a payload. Therefore, this endpoint needs the following new model, `RefreshToken`. Here is the code to add the model:

```

RefreshToken:  description: Contains the refresh token

type: object

properties:

refreshToken:

description: Refresh Token

type: string

```java

Finally, let’s add an endpoint to refresh the access token.
Refresh token endpoint
Add the following specification for the refresh token endpoint to `openapi.yaml`:

```

/api/v1/auth/token/refresh:  post:

tags:

- user

summary: Provides new JWT based on valid refresh token.

description: Provides JWT based on valid refresh token.

operationId: getAccessToken

requestBody:

content:

application/json:

schema:

$ref: '#/components/schemas/RefreshToken'

responses:

200:

description: For successful operation.

content:

application/json:

schema:

$ref: '#/components/schemas/SignedInUser'

```java

Here, we have used an exception by defining the refresh endpoint, in terms of forming a URI that represents the refresh token resources. Ideally, a `POST` call generates the new resource defined in URI. However, this endpoint generates the access token in place of the refresh token inside the response object, `SignedInUser`.
In the existing code, we don’t have a table to store the refresh token. Therefore, let’s add one.
Storing the refresh token using a database table
You can modify the Flyway database script to add a new table, as shown in the following code snippet:

```

create TABLE IF NOT EXISTS ecomm.user_token (   id uuid NOT NULL DEFAULT random_uuid(),

refresh_token varchar(128),

user_id uuid NOT NULL,

PRIMARY KEY(id),

FOREIGN KEY (user_id)

REFERENCES ecomm."user"(id)

);

```java

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/resources/db/migration/V1.0.0__Init.sql`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/resources/db/migration/V1.0.0__Init.sql)
Here, the table contains three fields – `id`, `refresh_token`, and `user_id` – for storing the row identifier (primary key), the refresh token, and the user’s ID, respectively. Also, we have put the table name `user` in double quotation marks because the H2 database also makes use of the term `"user"`.
Now, you have completed the API specification modification for authentication and authorization. Next, let’s start writing the implementation code for JWT-based authentication.
Implementing the JWT manager
Let’s add a constant class that contains all the constants related to the security functionality before we implement the JWT manager class, as shown in the following code snippet:

```

public class Constants {  public static final String ENCODER_ID = "bcrypt";

public static final String API_URL_PREFIX = "/api/v1/**";

public static final String H2_URL_PREFIX = "/h2-console/**";

public static final String SIGNUP_URL = "/api/v1/users";

public static final String TOKEN_URL = "/api/v1/auth/token";

public static final String REFRESH_URL =

"/api/v1/auth/token/refresh";

public static final String PRODUCTS_URL =

"/api/v1/products/**";

public static final String AUTHORIZATION =

"Authorization";

public static final String TOKEN_PREFIX = "Bearer ";

public static final String SECRET_KEY = "SECRET_KEY";

public static final long EXPIRATION_TIME = 900_000;

public static final String ROLE_CLAIM = "roles";

public static final String AUTHORITY_PREFIX = "ROLE_";

}

```java

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/security/Constants.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/security/Constants.java)
These constants are self-explanatory, except the `EXPIRATION_TIME` long value (`900_000`), which represents 15 minutes as a time unit.
Now, we can define the JWT manager class – `JwtManager`. `java-jwt` library from [auth0.com](http://auth0.com). We will use public/private keys to sign the token. Let’s define this class, as follows:

```

@Componentpublic class JwtManager {

private final RSAPrivateKey privateKey;

private final RSAPublicKey publicKey;

public JwtManager(@Lazy RSAPrivateKey privateKey,

@Lazy RSAPublicKey publicKey) {

this.privateKey = privateKey;

this.publicKey = publicKey;

}

public String create(UserDetails principal) {

final long now = System.currentTimeMillis();

return JWT.create()

.withIssuer("Modern API Development with Spring…")

.withSubject(principal.getUsername())

.withClaim(

ROLE_CLAIM,

principal.getAuthorities().stream()

.map(GrantedAuthority::getAuthority)

.collect(toList()))

.withIssuedAt(new Date(now))

.withExpiresAt(new Date(now + EXPIRATION_TIME))

.sign(Algorithm.RSA256(publicKey, privateKey));

}

}

```java

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/security/JwtManager.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/security/JwtManager.java)
Here, `JWT` is a class from the `java-jwt` library that provides a fluent API to generate the token. It adds issuer (`iss`), subject (`sub`), issued at (`iat`), and expired at (`exp`) claims.
It also adds a custom claim, `ROLE_CLAIM` (`roles`), which is populated using authorities from `UserDetails`. This is an interface provided by Spring Security. You can use the `org.springframework.security. core.userdetails.User.builder()` method to create a `UserBuilder` class. `UserBuilder` is a final builder class that allows you to build an instance of `UserDetails`.
Finally, this method (`JwtManager.create()`) signs the JWT, using `SHA256` with the RSA algorithm by calling the `sign` operation, which uses the provided public and private RSA keys. The JWT header specifies an `HS256` value for the algorithm (`alg`) claim.
Signing is done using the public and private RSA keys. Let’s add the code for RSA key management in our sample e-commerce application.
Generating the public/private keys
You can use JDK’s keytool to create a key store and generate public/private keys, as shown in the following code snippet:

```

$ keytool -genkey -alias "jwt-sign-key" -keyalg RSA -keystore jwt-keystore.jks -keysize 4096 输入 keystore 密码：

重新输入新密码：

请输入您的姓名和姓氏？

[Unknown]: 现代 API 开发

请输入您组织单位的名称？

[Unknown]: 组织单位

请输入您组织的名称？

[Unknown]: Packt

请输入您城市或地区的名称？

[Unknown]: 城市

请输入您州或省的名称？

[Unknown]: 州

请输入此单位的两位字母国家代码？

[Unknown]: 进口

使用 JWT 保护 REST API 191

CN=现代 API 开发, OU=组织单位, O=Packt, L=城市, ST=州, C=IN 是否正确？

[no]: 是

生成 4,096 位 RSA 密钥对和自签名证书（SHA384withRSA），有效期 90 天

for: CN=现代 API 开发, OU=组织单位, O=Packt, L=城市, ST=州, C=IN

```java

The generated key store should be placed under the `src/main/resources` directory.
Important note
Public/private keys are valid only for 90 days from the time they are generated. Therefore, make sure that you create a new set of public/private keys before you run this chapter’s code.
Required values used in the `keytool` command should also be configured in the `application.properties` file, as shown here:

```

app.security.jwt.keystore-location=jwt-keystore.jksapp.security.jwt.keystore-password=password

app.security.jwt.key-alias=jwt-sign-key

app.security.jwt.private-key-passphrase=password

```java

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/resources/application.properties`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/resources/application.properties)
Now, we can configure the key store and public/private keys in the security configuration class.
Configuring the key store and keys
Let’s add a `SecurityConfig` configuration class to configure the security relation configurations. This class extends the `WebSecurityConfigurerAdapter` class. Here’s the code to do this:

```

@Configuration@EnableWebSecurity

@EnableGlobalMethodSecurity(prePostEnabled = true)

public class SecurityConfig {

@Value("${app.security.jwt.keystore-location}")

private String keyStorePath;

@Value("${app.security.jwt.keystore-password}")

private String keyStorePassword;

@Value("${app.security.jwt.key-alias}")

private String keyAlias;

@Value("${app.security.jwt.private-key-passphrase}")

private String privateKeyPassphrase;

…

…

}

```java

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/security/SecurityConfig.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/security/SecurityConfig.java)
Here, we have added all the properties defined in `application.properties`.
Now, we can make use of the properties defined in `application.properties` to configure the `KeyStore`, `RSAPrivateKey`, and `RSAPublicKey` beans in the security configuration class, as shown in the following few subsections.
KeyStore bean
You can create a new bean for KeyStore by adding the following method and annotating it with `@Bean` in `SecurityConfig.java`:

```

@Beanpublic KeyStore keyStore() {

try {

KeyStore keyStore =

KeyStore.getInstance(KeyStore.getDefaultType());

InputStream resStream = Thread.currentThread().

getContextClassLoader().getResourceAsStream

(keyStorePath);

keyStore.load(resStream, keyStorePassword.

toCharArray());

return keyStore;

} catch (IOException | CertificateException |

NoSuchAlgorithmException | KeyStoreException e) {

LOG.error("无法加载 keystore: {}", keyStorePath, e);

keyStorePath, e);

}

throw new IllegalArgumentException

("无法加载 keystore");

}

```java

This creates a `KeyStore` instance, using the `KeyStore` class from the `java.security` package. It loads the key store from the `src/main/resources` package and uses the password configuration in the `application.properties` file.
Let’s define the `RSAPrivateKey` bean next.
RSAPrivateKey bean
You can create a new bean for `RSAPrivateKey` by adding the following method and annotating it with `@Bean` in `SecurityConfig.java`:

```

@Beanpublic RSAPrivateKey jwtSigningKey(KeyStore keyStore) {

try {

Key key = keyStore.getKey(keyAlias,

privateKeyPassphrase.toCharArray());

if (key instanceof RSAPrivateKey) {

return (RSAPrivateKey) key;

}

} catch (UnrecoverableKeyException |

NoSuchAlgorithmException | KeyStoreException e) {

LOG.error("keystore 中的密钥: {}", keyStorePath, e);

}

throw new IllegalArgumentException("无法加载

private key");

}

```java

This method uses a key alias and a private key password to retrieve the private key, which is used to return the `RSAPrivateKey` bean.
Let’s define the `RSAPublicKey` bean next.
RSAPublicKey bean
You can create a new bean for `RSAPublicKey` by adding the following method and annotating it with `@Bean` in `SecurityConfig.java`:

```

@Beanpublic RSAPublicKey jwtValidationKey(KeyStore keyStore) {

try {

Certificate certificate = keyStore.getCertificate

(keyAlias);

PublicKey publicKey = certificate.getPublicKey();

if (publicKey instanceof RSAPublicKey) {

return (RSAPublicKey) publicKey;

}

} catch (KeyStoreException e) {

LOG.error("keystore 中的密钥: {}", keyStorePath, e);

}

throw new IllegalArgumentException("无法加载公钥");

}

```java

Again, a key alias is used to retrieve the certificate from the key store. Then, the public key is retrieved from the certificate and returned.
As you know, `JwtManager` uses these public and private RSA keys to sign the JWT; therefore, a JWT decoder should use the same public key to decode the token. The Spring OAuth 2.0 resource server uses the `org.springframework.security.oauth2.jwt. JwtDecoder` interface to decode the token. Therefore, we need to create an instance of the `JwtDecoder` implementation and set the same public key in it to decode the token.
The Spring OAuth 2.0 resource server provides a `NimbusJwtDecoder` implementation class of `JwtDecoder`. Let’s now create a bean of it with the public key.
JwtDecoder bean
You can create a new bean for `JwtDecoder` by adding the following method and annotating it with `@Bean` in `SecurityConfig.java`:

```

@Beanpublic JwtDecoder jwtDecoder(RSAPublicKey rsaPublicKey) {

return NimbusJwtDecoder.withPublicKey(rsaPublicKey).build();

}

```java

You have defined all the beans required to sign the JWT token. Now, you can implement the newly added REST APIs.
Implementing new APIs
Let’s implement the APIs exposed using `UserApi`. This is a code part that was autogenerated using OpenAPI Codegen. First, you need to define a new entity mapped to the `user_token` table.
Coding user token functionality
You can create `UserTokenEntity` based on the `user_token` table, as shown in the following code snippet:

```

@Entity@Table(name = "user_token")

public class UserTokenEntity {

@Id

@GeneratedValue

@Column(name = "ID", updatable = false, nullable = false)

private UUID id;

@NotNull(message = "刷新令牌是必需的。")

@Basic(optional = false)

@Column(name = "refresh_token")

private String refreshToken;

@ManyToOne(fetch = FetchType.LAZY)

private UserEntity user;

…

…

}

```java

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/entity/UserTokenEntity.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/entity/UserTokenEntity.java)
Similarly, we can expose the following CRUD repository for `UserTokenEntity` with the following two methods – `deleteByUserId()`, which will remove the `UserToken` table record based on a given user ID, and `findByRefreshToken()`, which will find the `UserToken` table record based on a given refresh token. The code is illustrated in the following snippet:

```

public interface UserTokenRepository extends   CrudRepository<UserTokenEntity, UUID> {

Optional<UserTokenEntity> findByRefreshToken

(StringrefreshToken);

Optional<UserTokenEntity> deleteByUserId(UUID userId);

}

```java

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/repository/UserTokenRepository.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/repository/UserTokenRepository.java)
You have defined both the entity and its repository. Now, you will add new operations in `UserService` that will consume these new classes.
Enhancing the UserService class
We also need to add new methods to `UserService` for the `UserApi` interface. Let’s add new methods to the service, as follows:

```

UserEntity findUserByUsername(String username);Optional<SignedInUser> createUser(User user);

SignedInUser getSignedInUser(UserEntity userEntity); Optional<SignedInUser> getAccessToken(RefreshToken refToken);

void removeRefreshToken(RefreshToken refreshToken);

```java

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/service/UserService.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/service/UserService.java)
Here, each method performs a specific operation, as outlined here:

*   `findUserByUsername()`: This finds and returns a user based on a given username.
*   `createUser()`: This adds a new signed-up user to the database.
*   `getSignedInUser()`: This creates a new model instance of `SignedInUser` that holds the refresh token, access token (JWT), user ID, and username.
*   `getAccessToken()`: This generates and returns a new access token (JWT) for a given valid refresh token.
*   `removeRefreshToken()`: This removes the refresh token from the database. It is called when the user wants to sign out.

Let’s implement each of these methods in the `UserServiceImpl` class.
Implementing findUserByUsername()
First, you can add the implementation for `findUserByUsername()` in `UserServiceImpl` class, as shown in the following code snippet:

```

public UserEntity findUserByUsername(String username) {  if (Strings.isBlank(username)) {

throw new UsernameNotFoundException("无效用户。");

}

final String uname = username.trim();

Optional<UserEntity> oUserEntity =

repository.findByUsername(uname);

UserEntity userEntity = oUserEntity.orElseThrow

(() -> new UsernameNotFoundException(String.format(

"给定用户(%s)未找到。", uname)));

return userEntity;

}

```java

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/service/UserServiceImpl.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/service/UserServiceImpl.java)
This is a straightforward operation. You query the database based on a given username. If the username is found, then it returns the user; otherwise, it throws a `UsernameNotFoundException` exception.
createUser() implementation
Next, you can add the implementation for the `createUser()` method to the `UserServiceImpl` class, as shown in the following code snippet:

```

@Transactionalpublic Optional<SignedInUser> createUser(User user) {

Integer count = repository.findByUsernameOrEmail(

user.getUsername(), user.getEmail());

if (count > 0) {

throw new GenericAlreadyExistsException

("使用不同的用户名和电子邮件。");

}

UserEntity userEntity = repository.save(toEntity(user));

return Optional.of(createSignedUserWithRefreshToken(

userEntity));

}

```java

Here, we first check whether an existing user was assigned the same username or email in the sign-up request. If there was, an exception is simply raised; otherwise, a new user is created in the database and a `SignedInUser` instance is returned with refresh and access tokens, using the `createSignedUserWithRefreshToken()` method.
First, we can add a private `createSignedUserWithRefreshToken()` method in the `UserServiceImpl` class, as shown in the following code snippet:

```

private SignedInUser createSignedUserWithRefreshToken(   UserEntity userEntity) {

return createSignedInUser(userEntity)

.refreshToken(createRefreshToken(userEntity));

}

```java

This also uses another private method, `createSignedInUser()`, which returns `SignedInUser`; then, it adds the refresh token by calling the `createRefreshToken()` method.
Let’s define the two `createSignedInUser()` and `createRefreshToken()` private methods in the `UserServiceImpl` class, as shown in the following code snippet:

```

private SignedInUser createSignedInUser(UserEntity uEntity) {  String token = tokenManager.create(

org.springframework.security.core.userdetails

.User.builder()

.username(userEntity.getUsername())

.password(userEntity.getPassword())

.authorities(Objects.nonNull

(userEntity.getRole()) ?

userEntity.getRole().name() : "")

.build());

return new SignedInUser()

.username(userEntity.getUsername())

.accessToken(token)

.userId(userEntity.getId().toString());

}

private String createRefreshToken(UserEntity user) {

String token = RandomHolder.randomKey(128);

userTokenRepository.save(new UserTokenEntity().

setRefreshToken(token).setUser(user));

return token;

}

```java

Here, `tokenManager` is used in the `createSignedIn()` method to create the JWT. `tokenManager` is an instance of `JwtManager`. The `User.builder()` method is used to create a `UserBuilder` class. `UserBuilder`, which is a final builder class, is used to create an instance of `UserDetails`. The `JwtManager.create()` method uses this `UserDetails` instance to create a token.
The `createRefreshToken()` method uses the `RandomHolder` private static class to generate a refresh token. This token is not a JWT; we can use a longer-lasting valid token, such as one valid for a day, as a refresh token. Saving a JWT as a refresh token in the database removes the sole purpose of using the JWT because it expires by the configured time, and it should not be stored in the database, as it automatically becomes invalid. Therefore, we should think carefully before using a JWT as a refresh token and then saving it in the database.
Let’s add the `RandomHolder` private static class to the `UserServiceImpl` class, as shown in the following code snippet:

```

// https://stackoverflow.com/a/31214709/109354private static class RandomHolder {

static final Random random = new SecureRandom();

public static String randomKey(int length) {

return String.format("%"+length+"s",new BigInteger

(length * 5 /*base32, 2⁵*/,random).toString(32)).

replace('\u0020', '0');

}

}

```java

This class uses a `SecureRandom` instance to generate a random `BigInteger` instance. Then, this random `BigInteger` value is converted into a string with a radix size of `32`. Finally, the space is replaced with `0` if found in a converted string.
You can also use the `org.apache.commons.lang3.RandomStringUtils. randomAlphanumeric()` method, or use any other secured random key generator, to generate a refresh token.
We also need to modify the `UserRepository` class to add a new method that returns the count of users with a given username or email.
getSignedInUser() implementation
The implementation of the `getSignedInUser()` method is straightforward. Add it to the `UserServiceImpl` class, as shown in the following code snippet:

```

@Transactionalpublic SignedInUser getSignedInUser(UserEntity userEntity) {

userTokenRepository.deleteByUserId(userEntity.getId());

return createSignedUserWithRefreshToken(userEntity);

}

```java

Here, this method first removes the existing token from the database associated with the given user, and then it returns the new instance of `SignedInUser` that was created using `createSignedUserWithRefreshToken()`, defined previously in the *createUser()* *implementation* subsection.
getAccessToken() implementation
The implementation of the `getAccessToken()` method is, again, straightforward. Add it to the `UserServiceImpl` class, as shown in the following code snippet:

```

public Optional<SignedInUser> getAccessToken(   RefreshToken refreshToken) {

return userTokenRepository

.findByRefreshToken(refreshToken.getRefreshToken())

.map(ut ->

Optional.of(createSignedInUser(ut.getUser())

.refreshToken(refreshToken.getRefreshToken())))

.orElseThrow(() ->

new InvalidRefreshTokenException

("Invalid token."));

}

```java

First, the `getAccessToke()` method finds the user’s token entity using the `UserTokenRepository` instance. Then, it populates the `SignedInUser` POJO using the retrieved `UserToken` entity. The `createSignedInUser()` method does not populate the refresh token; therefore, we assign the same refresh token back. If it does find the user token entry in the database based on the refresh token, it throws an exception.
Also, you can add a validation for time that will remove/invalidate the refresh token, which has not been added here for simplicity.
You can also add a time validation logic for the refresh token – for example, storing the refresh token creation time in the database and using the configured valid time for refresh token validation, which is a kind of expiration logic for JWTs.
removeRefreshToken() implementation
You can add the `removeRefreshToken()` method to the `UserServiceImpl` class, as shown in the following code snippet:

```

public void removeRefreshToken(RefreshToken refreshToken) { userTokenRepository

.findByRefreshToken(refreshToken.getRefreshToken())

.ifPresentOrElse(

userTokenRepository::delete,

() -> {

throw new InvalidRefreshTokenException

("Invalid token.");

});

}

```java

First, the method finds the given refresh token in the database. If this is not found, then it throws an exception. If the given refresh token is found in the database, then it deletes it.
You have implemented all the extra methods added to the `UserService` class. Now, you will add additional methods in `UserRespository` too in the following section.
Enhancing the UserRepository class
Let’s add the `findByUsername()` and `findByUsernameOrEmail()` methods to `UserRepository`, as follows:

```

public interface UserRepository extends   CrudRepository<UserEntity, UUID> {

Optional<UserEntity> findByUsername(String username);

@Query( value = "select count(u.*) from ecomm.\"user\" u

where u.username = :username or u.email = :email",

nativeQuery = true)

Integer findByUsernameOrEmail(String username,

String email);

}

```java

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/repository/UserRepository.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/repository/UserRepository.java)
The `findByUsernameOrEmail` method returns a count of the records matching the given username or email.
You are now ready to implement the new APIs added to the `UserApi` interface to write the REST controllers. Let’s do that next.
Implementing the REST controllers
In the previous section, you developed and enhanced the services and repositories required to implement the APIs defined in the `UserApi` interface, generated by OpenAPI Codegen. The only pending dependency is `PasswordEncoder`. `PasswordEncoder` is required to encode the password before storing and matching the given password in the sign-in request.
Adding a bean for PasswordEncoder
You should expose the `PasswordEncoder` bean because Spring Security needs to know which encoding you want to use for password encoding, as well as for decoding the passwords. Let’s add a `PasswordEncoder` bean to `AppConfig`, as follows:

```

@Beanpublic PasswordEncoder passwordEncoder() {

Map<String, PasswordEncoder> encoders =

Map.of(

ENCODER_ID, new BCryptPasswordEncoder(),

"pbkdf2", Pbkdf2PasswordEncoder.

defaultsForSpringSecurity_v5_8(),

"scrypt", ScryptPasswordEncoder

.defaultsForSpringSecurity_v5_8());

return new DelegatingPasswordEncoder

(ENCODER_ID, encoders);

}

```java

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/AppConfig.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/AppConfig.java)
Here, you can directly create a new instance of `BcryptPasswordEncoder` and return it for `bcrypt` encoding. However, the use of `DelegatingPasswordEncoder` not only allows you to support existing passwords but also facilitates migration to a new, better encoder if one is available in the future. This code uses `Bcrypt` as a default password encoder, which is the best among the currently available encoders.
For `DelegatingPasswordEncoder` to work, you need to add a hashing algorithm prefix such as `{bcrypt}` to encoded passwords – for example, add `{bcrypt}$2a$10$neR0EcYY5./tLVp4litNyuBy/ kfrTsqEv8hiyqEKX0TXIQQwC/5Rm` to the persistent store if you already have a hashed password in the database, or if you’re adding any seed/test users to the database script. The new password will store the password with a prefix anyway, as configured in the `DelegatingPasswordEncoder` constructor. You have passed `bcrypt` into the constructor; therefore, all new passwords will be stored with a `{``bcrypt}` prefix.
`PasswordEncoder` reads the password from the persistence store and removes the prefix before matching. It uses the same prefix to find out which encoder it needs to use for matching. Now, you can start implementing the new APIs based on `UserApi`.
Implementing the Controller class
First, create a new `AuthController` class, as shown in the following code snippet:

```

@RestControllerpublic class AuthController implements UserApi {

private final UserService service;

private final PasswordEncoder passwordEncoder;

public AuthController(UserService s, PasswordEncoder e) {

this.service = s;

this.passwordEncoder = e;

}

…

…

}

```java

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/controller/AuthController.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/controller/AuthController.java)
The `AuthController` class is annotated with `@RestController` to mark it as a REST controller. Then, it uses two beans, `UserService` and `PasswordEncoder`, which will be injected at the time of the `AuthController` construction.
First, let’s add the sign-in operation, as follows:

```

public ResponseEntity<SignedInUser> signIn(@Valid SignInReq signInReq) {  UserEntity userEntity = service

.findUserByUsername(signInReq.getUsername());

if (passwordEncoder.matches(signInReq.getPassword(),

userEntity.getPassword())) {

return ok(service.getSignedInUser(userEntity));

}

throw new InsufficientAuthenticationException

("Unauthrzed");

}

```java

The operation first finds the user and matches the password using the `PasswordEncoder` instance. If everything goes through successfully, it returns the `SignedInUser` instance with refresh and access tokens; otherwise, it throws an exception.
Let’s add other operations to `AuthController`, as follows:

```

public ResponseEntity<Void> signOut(  @Valid RefreshToken refreshToken) {

service.removeRefreshToken(refreshToken);

return accepted().build();

}

public ResponseEntity<SignedInUser> signUp

(@Valid User user) {

return status(HttpStatus.CREATED)

.body(service.createUser(user).get());

}

public ResponseEntity<SignedInUser> getAccessToken(

@Valid RefreshToken refreshToken) {

return ok(service.getAccessToken(refreshToken)

.orElseThrow(InvalidRefreshTokenException::new));

}

```java

All operations such as `signOut()`, `signUp()`, and `getAccessToken()` are straightforward, as outlined here:

*   `signOut()` uses the user service to remove the given refresh token. Ideally, you would like to get the user ID from the logged-in user’s request and remove the refresh token, based on the retrieved user ID from the request.
*   `signUp()` creates a valid new user and returns the `SignedInUser` instance as a response. Here, we haven’t added the validation of the payload for simplicity. In a real-world application, you must validate the request payload.
*   `getAccessToken()` returns `SignedInUser` with a new access token if the given refresh token is valid.

We have finished coding the controllers. Let’s configure security in the next subsection.
Configuring web-based security
This is the last puzzle to sort out the authentication and authorization piece. The `SecurityConfig` class is also annotated with `@EnableWebSecurity`. With the new version of Spring Security, you now don’t need to extend `WebSecurityConfigurerAdapter` and override the `configure()` method, as we did in the last edition of this book. Instead, you now create a bean that returns the configured instance of the `SecurityFilterChain` class.
The method (`filterChain`) that returns `SecurityFilterChain` takes `HttpSecurity` as a parameter. `HttpSecurity` contains DSL (fluent methods). You can make use of these methods to configure web-based security, such as which web paths to allow and which method to allow. Let’s make the following configurations using these fluent methods to return the `SecurityFilterChain` instance, as shown in the following code snippet from `SecurityConfig.java`:

```

@Beanprotected SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

http.httpBasic().disable()

.formLogin().disable()

.csrf().ignoringRequestMatchers(API_URL_PREFIX)

.ignoringRequestMatchers(toH2Console())

.and()

.headers().frameOptions().sameOrigin()

.and()

.cors()

.and()

.authorizeHttpRequests(req ->

req.requestMatchers(toH2Console()).permitAll()

.requestMatchers(new AntPathRequestMatcher(

TOKEN_URL, HttpMethod.POST.name()))

.permitAll()

.requestMatchers(new AntPathRequestMatcher(

TOKEN_URL, HttpMethod.DELETE.name())).

permitAll()

.requestMatchers(new AntPathRequestMatcher(

SIGNUP_URL, HttpMethod.POST.name()))

.permitAll()

.requestMatchers(new AntPathRequestMatcher(

REFRESH_URL, HttpMethod.POST.name())).

permitAll()

.requestMatchers(new AntPathRequestMatcher(

PRODUCTS_URL, HttpMethod.GET.name())).

permitAll()

.requestMatchers("/api/v1/addresses/**")

.hasAuthority(RoleEnum.ADMIN.getAuthority())

.anyRequest().authenticated())

.oauth2ResourceServer(oauth2ResourceServer ->

oauth2ResourceServer.jwt(jwt ->

jwt.jwtAuthenticationConverter(

getJwtAuthenticationConverter())))

.sessionManagement().sessionCreationPolicy(

SessionCreationPolicy.STATELESS);

return http.build();

}

```java

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/security/SecurityConfig.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/security/SecurityConfig.java)
Here, you configure the following security settings:

1.  First of all, you disable the basic authentication and form login using the `disable()` method.
2.  Then, you ignore the CSRF configuration for the API base path and H2 console URLs.
3.  Then, you set the headers setting for frame options that have the same origin to allow the H2 console application to work fine in the browser. The H2 console UI is based on HTML frames. The H2 console UI won’t display in browsers because, by default, the security header (`X-Frame-Options`) is not sent with permission to allow frames with the same origin. Therefore, you need to configure `headers().frameOptions().sameOrigin()`.
4.  Then, you enable the CORS setting. You’ll learn more about this in the next section.
5.  Then, you configure the authorization of the request, which takes the request object as a parameter. You use this request object to restrict access, based on URL patterns, by using the `requestMatchers()` method and an instance of the `AntPathRequestMatcher` class:
    *   Configure URL patterns and respective HTTP methods using `mvcMatchers()`, which uses the same pattern-matching style as a Spring `Static method toH2Console()` is a utility that provides a matcher that includes the H2 console location.
    *   The `/api/v1/addresses/** ` pattern has been configured to be accessed only by the user who has the `ADMIN` role, by calling `hasAuthority()` and passing the admin authority in it. You’ll learn more about it in the *Understanding authorization* section of this chapter.
6.  All URLs, except those configured explicitly by `authorizeHttpRequests()`, should be allowed by any authenticated user (by using `anyRequest(). authenticated()`).
    *   Enable JWT bearer token support for the OAuth 2.0 resource server (`oauth2ResourceServer.jwt()`).
    *   Enable the `STATELESS` session creation policy (that is, `sessionManagement().sessionCreationPolicy` won’t create any `HTTPSession`)
7.  Finally, the `filterChain` method returns `SecurityFilterChain` by building the instance from the configured `HttpSecurity` instance (the `http.build()` call).

In this section, you learned how to configure Spring security for authentication and authorization. Next, we will learn about the CORS and CSRF.
Configuring CORS and CSRF
Browsers restrict cross-origin requests from scripts for security reasons. For example, a call from `http://mydomain.com` to `http://mydomain-2.com` can’t be made using a script. Also, an origin not only indicates a domain but also includes a scheme and a port.
Before hitting any endpoint, the browser sends a pre-flight request using the HTTP method option to check whether the server will permit the actual request. This request contains the following headers:

*   The actual request’s headers (`Access-Control-Request-Headers`).
*   A header containing the actual request’s HTTP method (`Access-Control- Request-Method`).
*   An `Origin` header that contains the requesting origin (scheme, domain, and port).
*   If the response from the server is successful, then only the browser allows the actual request to fire. The server responds with other headers, such as `Access- Control-Allow-Origin`, which contains the allowed origins (an asterisk `*` value means any origin), `Access-Control-Allow-Methods` (allowed methods), `Access-Control-Allow-Headers` (allowed headers), and `Access-Control-Max-Age` (allowed time in seconds).

You can configure CORS to take care of cross-origin requests. For that, you need to make the following two changes:

*   Add a `CorsConfigurationSource` bean that takes care of the CORS configuration using a `CorsConfiguration` instance.
*   Add the `cors()` method to `HTTPSecurity` in the `configure()` method. The `cors()` method uses `CorsFilter` if a `corsFilter` bean is added; otherwise, it uses `CorsConfigurationSource`. If neither is configured, then it uses the Spring MVC pattern inspector handler.

Let’s now add the `CorsConfigurationSource` bean to the `SecurityConfig` class.
The default permitted values (`new CorsConfiguraton(). applyPermitDefaultValues()`) configure CORS for any origin (`*`), all headers, and simple methods (`GET`, `HEAD`, and `POST`), which have an allowed max age of `30` minutes.
You need to allow mostly all of the HTTP methods, including the `DELETE` method, and you need more custom configuration; therefore, we will use the following bean definition in `SecurityConfig.java`:

```

@BeanCorsConfigurationSource corsConfigurationSource() {

CorsConfiguration configuration = new CorsConfiguration();

configuration.setAllowedOrigins(List.of("*"));

configuration.setAllowedMethods(Arrays.asList("HEAD",

"GET", "PUT", "POST", "DELETE", "PATCH"));

// For CORS response headers

configuration.addAllowedOrigin("*");

configuration.addAllowedHeader("*");

configuration.addAllowedMethod("*");

UrlBasedCorsConfigurationSource source = new

UrlBasedCorsConfigurationSource();

source.registerCorsConfiguration("/**", configuration);

return source;

}

```java

Here, you create a `CorsConfiguration` instance using the default constructor and then set the allowed origins, allowed methods, and response headers. Finally, you pass it as an argument while registering it to the `UrlBasedCorsConfigurationSource` instance and returning it.
In the previous section, inside the `SecurityChainFilter` method annotated with `@Bean`, you have configured CSRF using the `csrf()` DSL. We have applied CSRF protection to all URLs, except URLs starting with `/api/v1` and the `/h2-console` H2 database console URLs. You can change the configuration based on your requirement.
Let’s first understand what CSRF/XSRF is. **CSRF** or **XSRF** stands for **cross-site request forgery**, which is a web security vulnerability. To understand how this vulnerability comes into effect, let’s assume you are a bank customer and are currently signed in to your account online. While you are logged in, you may receive an email and click on a link in it, or on any other malicious website’s link, that contains a malicious script. This script can then send a request to your bank for a fund transfer. The bank then transfers the funds to a perpetrator’s account because the bank thinks that the request has been sent by you, as you are signed in. Hackers can use this vulnerability similarly for different hacking activities.
To prevent such attacks, the application sends new unique CSRF tokens associated with the signed-in user for each new request. These tokens are stored in hidden form fields. When a user submits a form, the same token should be sent back with the request. The application then verifies the CSRF token and only processes the request if the verification is successful. This works because malicious scripts can’t read the token due to the same origin policy.
However, if a perpetrator also tricks you into revealing the CSRF token, then it is very difficult to prevent such attacks. You can disable CSRF protection for this web service by using `csrf().disable()` because we only expose REST endpoints.
Now, let’s move on to the final section, where you will configure the authorization based on the user’s role.
Understanding authorization
Your valid username/password or access token for authentication gives you access to secure resources, such as URLs, web resources, or secure web pages. Authorization is one step ahead; it allows you to configure access security further with scopes such as read, write, or roles such as Admin, User, and Manager. Spring Security allows you to configure any custom authority.
We will configure three types of roles for our sample e-commerce app – namely, Customer (user), Admin, and Customer Support Representative (CSR). Obviously, each user will have their own specific authority. For example, a user can place an order and buy stuff online but should not be able to access the CSR or admin resources. Similarly, a CSR should not be able to have access to admin-only resources. A security configuration that allows authority or role-based access to resources is known as authorization. A failed authentication should return an HTTP `401` status (unauthorized), and a failed authorization should return an HTTP `403` status (forbidden), which means the user is authenticated but does not have the required authority/role to access the resource.
Let’s introduce these three roles in a sample e-commerce app, as shown in the following code snippet:

```

public enum RoleEnum implements GrantedAuthority {  USER(Const.USER), ADMIN(Const.ADMIN), CSR(Const.CSR);

private String authority;

RoleEnum(String authority) {

this.authority = authority;

}

@JsonCreator

public static RoleEnum fromAuthority(String authority) {

for (RoleEnum b : RoleEnum.values()) {

if (b.authority.equals(authority)) {

return b;

}

}

}

@Override

public String toString() {

return String.valueOf(authority);

}

@Override

@JsonValue

public String getAuthority() {

return authority;

}

public class Const {

public static final String ADMIN = "ROLE_ADMIN";

public static final String USER = "ROLE_USER";

public static final String CSR = "ROLE_CSR";

}

}

```java

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/entity/RoleEnum.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/entity/RoleEnum.java)
Here, we declared an `enum` that implements Spring Security’s `GrantedAuthority` interface to override the `getAuthority()` method. `GrantedAuthority` is an authority granted to an `Authentication` (interface) object. As you know, `BearerTokenAuthenticationToken` is a type of `AbstractAuthenticationToken` class that implements the authentication interface, which represents the token/principal for an authenticated request. We have used the string constants for the user’s roles in this `enum`, as we need these when we configure the role-based restriction at a method level.
Let’s discuss the role and authority in detail.
Role and authority
An authority can be assigned for fine-grained control, whereas roles should be applied to large sets of permissions. A role is an authority that has the `ROLE_` prefix. This prefix is configurable in Spring Security.
Spring Security provides the `hasRole()` and `hasAuthority()` methods to apply role- and authority-based restrictions. `hasRole()` and `hasAuthority()` are almost identical, but the `hasRole()` method maps with `Authority` without the `ROLE_` prefix. If you use `hasRole` (`'ADMIN'`), your `Admin` enum must be `ROLE_ADMIN` instead of `ADMIN` because a role is an authority and should have a `ROLE_` prefix, whereas if you use `hasAuthority` (`'ADMIN'`), your `Admin` enum must be only `ADMIN`.
The OAuth 2.0 resource server, by default, populates authorities based on the scope (`scp`) claim. If you provide access to a user’s resources, such as order history for integration with another application, then you can limit an application’s access to a user’s account before granting access to other applications for third-party integration. Third-party applications can request one or more scopes; this information is then presented to the user on the consent screen, and the access token issued to the application will be limited to the scopes granted. However, in this chapter, we haven't provided OAuth 2.0 authorization flows and will limit security access to REST endpoints.
If the JWT contains a claim with the name *scope* (`scp`), then Spring Security will use the value in that claim to construct the authorities by prefixing each value with `SCOPE_`. For example, if a payload contains a `scp=["READ","WRITE"]` claim, this means that an `Authority` list will consist of `SCOPE_READ` and `SCOPE_WRITE`.
We need to change the default authority mapping behavior because a scope (`scp`) claim is the default authority for the OAuth2.0 resource server in Spring. We can do that by adding a custom authentication converter to `JwtConfigurer` in `OAuth2ResourceServer` in your security configuration. Let’s add a method that returns the converter, as follows:

```

private Converter<Jwt, AbstractAuthenticationToken>   getJwtAuthenticationConverter() {

JwtGrantedAuthoritiesConverter authorityConverter =

new JwtGrantedAuthoritiesConverter();

authorityConverter.setAuthorityPrefix(AUTHORITY_PREFIX);

authorityConverter.setAuthoritiesClaimName(ROLE_CLAIM);

JwtAuthenticationConverter converter =

new JwtAuthenticationConverter();

converter.setJwtGrantedAuthoritiesConverter(authorityConverter);

return converter;

}

```java

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/security/SecurityConfig.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/src/main/java/com/packt/modern/api/security/SecurityConfig.java)
Here, we first create a new instance of `JwtGrantedAuthorityConverter` and then assign an authority prefix (`ROLE_`) and authority claim name (the key of the claim in JWT) as `roles`.
Now, we can use this private method to configure the OAuth 2.0 resource server. You can now modify the existing configuration with the following code. We can also add configuration to add role-based restrictions to the `POST` `/api/v1/addresses` API call, in the following code snippet in `SecurityConfig.java`:

```

.requestMatchers("/api/v1/addresses/**")     .hasAuthority(RoleEnum.ADMIN.getAuthority())

.anyRequest().authenticated())

.oauth2ResourceServer(oauth2ResourceServer ->

oauth2ResourceServer.jwt(jwt ->

jwt.jwtAuthenticationConverter(

getJwtAuthenticationConverter())))

```java

After setting this configuration to add an address (`POST /api/v1/addresses`), it now requires both authentication and authorization. This means the logged-in user must have the `ADMIN` role to call this endpoint successfully. Also, we changed the default claim from scope to role.
Now, we can proceed further with method-level, role-based restrictions. Spring Security provides a feature that allows you to place authority- and role-based restrictions on public methods of Spring beans, using a set of annotations such as `@PreAuthorize`, `@Secured`, and `@RolesAllowed`. By default, these are disabled; therefore, you need to enable them explicitly.
Let’s enable these by adding the `@EnableGlobalMethodSecurity(prePostEnabled = true)` annotation to the Spring Security configuration class, as follows:

```

@Configuration@EnableWebSecurity

@EnableGlobalMethodSecurity(prePostEnabled = true)

public class SecurityConfig {

```java

Now, you can use the `@PreAuthorize` (the given access-control expression would be evaluated before the method invocation) and `@PostAuthorize` (the given access-control expression would be evaluated after the method invocation) annotations to place restrictions on public methods of Spring beans because you have set the `prePostEnabled` property to `true` when enabling the global method-level security.
`@EnableGlobalMethodSecurity` also supports the following properties:

*   `securedEnabled`: This allows you to use `@Secured` annotation on public methods.
*   `jsr250Enabled`: This allows you to use JSR-250 annotations such as `@RolesAllowed`, which can be applied to both public classes and methods. As the name suggests, you can use a list of roles for access restrictions.

`@PreAuthorize` and `@PostAuthorize` are more powerful than the other security annotations because not only can they be configured for authorities/roles but also for any valid **Spring Expression Language** (**SpEL**) expression:

```

为了演示目的，让我们向 deleteAddressesById()方法添加@PreAuthorize 注解，该方法与 AddressController 中的 DELETE /v1/auth/addresses/{id}相关联，如下代码片段所示:@PreAuthorize("hasRole('" + Const.ADMIN + "')")

@Override

public ResponseEntity<Void> deleteAddressesById(String id) {

service.deleteAddressesById(id);

return accepted().build();

}

```java

Let’s break down the preceding code snippet:

*   `hasRole()` is a built-in `SpEL` expression. We need to pass a valid `SpEL` expression, and it should be a string. Any variable used to form this `SpEL` expression should be final. Therefore, we have declared the final string constants in the `RoleEnum` enum (for example, `Const.ADMIN`).
*   Now, the `DELETE /api/v1/addresses/{id}` REST API can only be invoked if the user has the `ADMIN` role.
*   Spring Security provides various built-in `SpEL` expressions, such as `hasRole()`. Here are some others:
    *   `hasAnyRole(String… roles)`: This returns `true` if the principal’s role matches any of the given roles.
    *   `hasAuthority(String authority)`: This returns `true` if the principal has given authority. Similarly, you can also use `hasAnyAuthority(String… authorities)`.
    *   `permitAll`: This returns `true`.
    *   `denyAll`: This returns `false`.
*   `isAnonymous()`: This returns `true` if the current user is anonymous.
*   `isAuthenticated()`: This returns `true` if the current user is not anonymous.

A full list of these expressions is available at [`docs.spring.io/spring-security/site/docs/3.0.x/reference/el-access.html`](https://docs.spring.io/spring-security/site/docs/3.0.x/reference/el-access.html).
Similarly, you can apply access restrictions for other APIs. Let’s test security in the next section.
Testing security
By now, you must be looking forward to testing. You can find the API client collection at the following location. You can import it and then test the APIs, using any API client that supports the HAR type file import: [`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/Chapter06-API-Collection.har`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/Chapter06-API-Collection.har).
Important note
Make sure to generate the keys again, as keys generated by the JDK keytool are only valid for 90 days.
Building and running the Chapter 06 code
You can build the code by running `gradlew clean build` from the root of the project, and you can run the service using `java -jar build/libs/Chapter06-0.0.1-SNAPSHOT.jar`. Make sure to use Java 17 in the path.
Now, let’s test our first use case.
Let’s fire the `GET /api/vi/addresses` API without the `Authorization` header, as shown in the following command:

```

$ curl -v 'http://localhost:8080/api/v1/addresses' -H 'Content- Type: application/json' -H 'Accept: application/json'< HTTP/1.1 401

< Vary: Origin

< Vary: Access-Control-Request-Method

< Vary: Access-Control-Request-Headers

< WWW-Authenticate: Bearer

< X-Content-Type-Options: nosniff

< X-XSS-Protection: 1; mode=block

< Cache-Control: no-cache, no-store, max-age=0, must

revalidate

< 其他信息因简洁性而被删除

```java

This returns the HTTP `401` status (`unauthorized`) and a `WWW-Authenticate: Bearer` response header, which suggests the request should be sent with an `Authorization` header.
Let’s send the request again with an invalid token, as shown in the following command:

```

$ curl -v 'http://localhost:8080/api/v1/addresses' -H 'Content-Type: application/json' -H 'Accept: application/json' -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9…rest of the JWT string removed for brevity'

< HTTP/1.1 401

< Vary: Origin

< Vary: Access-Control-Request-Method

< Vary: Access-Control-Request-Headers

< WWW-Authenticate: Bearer

< 其他信息因简洁性而被删除

```java

Again, it returns the `401` response.
We have created two users using a Flyway database migration script – `scott/tiger` and `scott2/tiger`. Now, let’s perform a sign-in with the username `scott` to get the valid JWT, as follows:

```

$ curl -X POST http://localhost:8080/api/v1/auth/token -H  'Accept: application/json' -H  'Content-Type: application/json' -d '{    "username": "scott",

"password": "tiger"

}'

{"refreshToken":"9rdk5b35faafkneqg9519s6p4tbbqcdt412t7h5st9savkonqnnd 5e8ntr334m8rffgo6bio1vglng1hqqse3igdonoabc971lpdgt3bjoc3je3m81ldp2au vimts8p6","accessToken":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJzdWI iOiJzY290dCIsInJvbGVzIjpbIlVTRVIiXSwiaXNzIjoiTW9kZXJuIEFQSSBEZXZlbG9 wbWVudCB3aXRoIFNwcmluZyBhbmQgU3ByaW5nIEJvb3QiLCJleHAiOjE2Nzc0OTYzMzk sImlhdCI6MTY3NzQ5NTQzOX0.a77O7ZbSAOw5v6Tb3w-MtBwotMEUvc1H1y2W0IU2QJh0m lSJxSBCfdrNBl0mVk1HnwX4kOpj4grbNasBjpIpHtyOLXdp-gngxdvVfaKSPuptrW4YzA3 ikxbUMWDdEtij_y2DRxJXQ6CrPTjA40L7yB_SXswnHT988Qq6ZALeGW-Lmz-vzAZiRcZUe 6bPPn7F-p4lK_qi1nsUJ1rnWmmffLWCH37ztllcgh6bB1UJuOn9Hw2A1nQExfUutRKgFK0 -LxBUOKOKdRESOnJR9hwOL6v10IFl9xNm53LVMIcuJrndCxvmv7mv0fUOxY63UwhO9kOT RCXViGKCa3H8RxUFwG52q2nZelle_4I8CUSeDDdmD2Rlax2NyQNe-HHEJb9c91JSzhFm0 K0-c34-kiNGqaB3jljndHoGXCBLM5prphlSdlV4U9PYhmL8ZCaDv8q6rCPSAEcRoiOBPPn dxEApHKulj9vrO_p7K1T9dLamJSFJKw9Yz8M3_ngiE3qtEBQ3tEUFkZsJpGop5HIxrkB0O e7L_oETir_wUe1qs8AIZcKSwP9X6fpUuOlONKDpDc-f-n5PjEAvts3BcxuM8Jrw80F6z6T OJrcikrMt8DGaIXs2WHNP7C605l-JgwCVuZz_8S4LLtaCFnqq4xLU1Gy2qj5CBbALVoFcB fjoVLN2fq4","username":"scott","userId":"a1b9b31d-e73c-4112-af7c-b68530f38222"}

```java

This returns with both a refresh and an access token. Let’s use this access token to call the `GET /api/v1/addresses` API again (please note that the `Bearer` token value in the `Authorization` header is taken from the response of the previous `POST /api/v1/ auth/token` API call). The command is shown in the following block:

```

$ curl -v 'http://localhost:8080/api/v1/addresses' -H 'Content-Type: application/json' -H 'Accept: application/json' -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGci…rest of the JWT string removed for brevity'

< HTTP/1.1 403

< Vary: Origin

< Vary: Access-Control-Request-Method

< Vary: Access-Control-Request-Headers

< WWW-Authenticate: Bearer error="insufficient_scope", error_description="The request requires higher privileges than provided by the access token.", error_uri="https://tools.ietf.org/html/rfc6750#section-3.1"

```java

This command execution returns `403`. It means that the user was authenticated successfully. However, the user doesn’t contain the required role to access the endpoint.
Let’s try again, with user `scott2` this time, who has an ADMIN role:

```

$ curl -X POST http://localhost:8080/api/v1/auth/token -H 'Accept: application/json' -H 'Content-Type: application/json' -d '{    "username": "scott2",

"password": "tiger"

}'

{"refreshToken":"a6hidhaeb8scj3p6kei61g4a649dghcf5jit1v6rba2mn92o0dm0 g6gs6qfh7suiv68p2em0t0nnue8unm10bg079f39590jig0sccisecim5ep3ipuiu29ce aoc793h","accessToken":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9…","username":"scott2","userId":"a1b9b31d-e73c-4112-af7c-b68530f38223"}

## Some of the output removed for brevity

$ curl -v 'http://localhost:8080/api/v1/addresses' -H 'Content-Type: application/json' -H 'Accept: application/json' -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9… rest of the token truncated for brevity'

[{"links":[{"rel":"self","href":"http://localhost:8080/a731fda1-aaad-42ea-bdbc-a27eeebe2cc0"},{"rel":"self","href":"http://localhost:8080/api/v1/addresses/a731fda1-aaad-42ea-bdbc-a27eeebe2cc0"}],"id":"a731fda1-aaad-42ea-bdbc-a27eeebe2cc0","number":"9I-999","residency":"Fraser Suites Le Claridge","street":"Champs-Elysees","city":"Paris","state":"Île-de-France","country":"France","pincode":"75008"}]

```java

This time, the call is successful. Now, let’s use the refresh token to get a new access token, as follows:

```

$  curl -X POST 'http://localhost:8080/api/v1/auth/token/refresh' -H 'Content-Type: application/json' -H 'Accept: application/json' -d '{"refreshToken": "a6hidhaeb8scj3p6kei61g4a649dghcf5jit1v6rba2mn92o0dm0 g6gs6qfh7suiv68p2em0t0nnue8unm10bg079f39590jig0sccisecim5ep3ipuiu29ce aoc793h"'

}'

{"refreshToken":"a6hidhaeb8scj3p6kei61g4a649dghcf5jit1v6rba2mn92o0dm0g 6gs6qfh7suiv68p2em0t0nnue8unm10bg079f39590jig0sccisecim5ep3ipuiu29ceao c793h","accessToken":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9… rest of the token truncated for brevity","username":"scott2","userId":"a1b9b31d-e73c-4112-af7c-b68530f38223"}

```java

This time, it returns a new access token with the same refresh token given in the payload.
If you pass an invalid refresh token while calling the refresh token API, it will provide the following response:

```

{  "errorCode":"PACKT-0010",

"message":"Requested resource not found. Invalidtoken.",

"status":404,

"url":"http://localhost:8080/api/v1/auth/token/refresh",

"reqMethod":"POST","timestamp":"2023-02-27T11:13:27.183172Z"

}

```java

Similarly, you can call other API endpoints. Alternatively, you can import the following HAR file in an API client, such as Insomnia, and then test the remaining APIs: [`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/Chapter06-API-Collection.har`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter06/Chapter06-API-Collection.har).
Summary
In this chapter, you learned about JWTs, Spring Security, authentication using filters, and JWT token validation, using filters and authentication with the Spring OAuth 2.0 resource server. You also learned how you can add CORS and CSRF protection and why these are necessary.
You also learned about access protection based on roles and authorities. You have now the skills to implement JWTs, Spring Security, and the Spring Security OAuth 2.0 resource server to protect your web resources.
In the next chapter, you will develop a sample e-commerce app’s UI using the Spring Security framework and APIs used in this chapter. This integration will allow you to understand the UI flows and how to consume REST APIs using JavaScript.
Questions

1.  What is a security context and a principal?
2.  Which is the preferred way to secure a JWT – signing or encrypting a token?
3.  What are the best practices to use a JWT?

Answers

1.  The security context stores the principal using `SecurityContextHolder` and is always available in the same thread of execution. The security context allows you to extract the principal during the flow execution and use it wherever you want. This is where a security annotation such as `@PreAuthorize` makes use of it for validation. The principal is the currently logged-in user. It can either be an instance of `UserDetails` or a string carrying a username. You can use the following code to extract it:

    ```

    Object principal = SecurityContextHolder                   .getContext().getAuthentication()                   .getPrincipal();if (principal instanceof UserDetails) {  String username =              ((UserDetails)principal).getUsername();} else {  String username = principal.toString();}

    ```java

     2.  This is a subjective question. However, it is recommended to use the signing of tokens (JWS) if a JWT doesn’t contain sensitive and private information, such as date of birth or credit card information. In such cases, you should make use of JWE to encrypt the information. If you want to use both together, then the preferred way is to use encryption for information carried by the token and then sign it with keys.
3.  You can follow the following guidelines and add to them if you discover any new ones:
    *   Make sure that JWT always has issuer and audience validations.
    *   Make sure that a JWT validation does not allow a `none` algorithm (i.e., no algorithm mentioned in JWT or when the `alg` field in the JWT header contains a `none` value). Instead, make sure that you have verification in place that checks the specific algorithm (whatever you configured) and a key.
    *   Keep an eye on the **National Vulnerability** **Database** (**NVD**).
    *   Don’t use a weak key (secret). Instead, use the asymmetric private/public keys with SHA 256, SHA 384, and SHA 512.
    *   Use a minimum key size of 2,048 for normal cases and 3,072 for business cases.
    *   A private key should be used for authentication, and the verification server should use a public key.
    *   Make sure clients use the security guidelines to store the tokens, and web applications should use HTTPS for communication with servers.
    *   Make sure the web application is tested thoroughly for **cross-site scripting** (**XSS**) attacks. It is always best to use a **content security** **policy** (**CSP**).
    *   Keep a short expiration time, and use a refresh token to refresh an access token.
    *   Keep an eye on OWASP security guidelines and new threats.

Further reading

*   *Hands-On Spring Security 5.x* (video course): [`www.packtpub.com/product/hands-on-spring-security-5-x-video/9781789802931`](https://www.packtpub.com/product/hands-on-spring-security-5-x-video/9781789802931)
*   Spring Security documentation: [`docs.spring.io/spring-security/site/docs/current/reference/html5/`](https://docs.spring.io/spring-security/site/docs/current/reference/html5/)
*   JWT: [`tools.ietf.org/html/rfc7519`](https://tools.ietf.org/html/rfc7519)
*   JWS: [`www.rfc-editor.org/info/rfc7515`](https://www.rfc-editor.org/info/rfc7515)
*   JWE: [`www.rfc-editor.org/info/rfc7516`](https://www.rfc-editor.org/info/rfc7516)
*   Spring Security in-built `SpEL` expressions: [`docs.spring.io/spring-security/site/docs/3.0.x/refsseerence/el-access.html`](https://docs.spring.io/spring-security/site/docs/3.0.x/refsseerence/el-access.html)

```
