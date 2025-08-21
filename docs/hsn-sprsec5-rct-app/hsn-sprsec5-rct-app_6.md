# 第六章：REST API 安全

Spring Security 可以用于保护 REST API。本章首先介绍了有关 REST 和 JWT 的一些重要概念。

然后，本章介绍了 OAuth 概念，并通过实际编码示例，解释了在 Spring 框架的 Spring Security 和 Spring Boot 模块中利用简单和高级 REST API 安全。

我们将在示例中使用 OAuth 协议来保护暴露的 REST API，充分利用 Spring Security 功能。我们将使用 JWT 在服务器和客户端之间交换声明。

在本章中，我们将涵盖以下概念：

+   现代应用程序架构

+   响应式 REST API

+   简单的 REST API 安全

+   高级 REST API 安全

+   Spring Security OAuth 项目

+   OAuth2 和 Spring WebFlux

+   Spring Boot 和 OAuth2

# 重要概念

在进行编码之前，我们需要熟悉一些重要概念。本节旨在详细介绍一些这些概念。

# REST

**表述性状态转移**（**REST**）是 Roy Fielding 于 2000 年提出的一种用于开发 Web 服务的架构风格。它建立在著名的**超文本传输协议**（**HTTP**）之上，可以以多种格式传输数据，最常见的是**JavaScript 对象表示法**（**JSON**）和**可扩展标记语言**（**XML**）。在 REST 中，请求的状态使用标准的 HTTP 状态码表示（200：OK，404：页面未找到！等）。基于 HTTP，安全性是通过已熟悉的**安全套接字层**（**SSL**）和**传输层安全性**（**TLS**）来处理的。

在编写此类 Web 服务时，您可以自由选择任何编程语言（Java，.NET 等），只要它能够基于 HTTP 进行 Web 请求（这是每种语言都支持的事实标准）。您可以使用许多知名的框架来开发服务器端的 RESTful API，这样做非常容易和简单。此外，在客户端，有许多框架可以使调用 RESTful API 和处理响应变得简单直接。

由于 REST 是基于互联网协议工作的，通过提供适当的 HTTP 头部（Cache-Control，Expires 等），可以很容易地实现对 Web 服务响应的缓存。`PUT`和`DELETE`方法在任何情况下都不可缓存。以下表格总结了 HTTP 方法的使用：

| **HTTP 方法** | **描述** |
| --- | --- |
| `GET` | 检索资源 |
| `POST` | 创建新资源 |
| `PUT` | 更新现有资源 |
| `DELETE` | 删除现有资源 |
| `PATCH` | 对资源进行部分更新 |

表 1：HTTP 方法使用

REST API 请求/响应（通过网络发送的数据）可以通过指定适当的 HTTP 头部进行压缩，类似于缓存。客户端发送 Accept-Encoding 的 HTTP 头部，让服务器知道它可以理解哪些压缩算法。服务器成功压缩响应并输出另一个 HTTP 头部 Content-Encoding，让客户端知道应该使用哪种算法进行解压缩。

# JSON Web Token（JWT）

<q>"JSON Web Tokens 是一种开放的、行业标准的 RFC 7519 方法，用于在两个当事方之间安全地表示声明。"</q>

*- [`jwt.io/`](https://jwt.io/)*

在过去，HTTP 的无状态性质在 Web 应用程序中被规避（大多数 Web 应用程序的性质是有状态的），方法是将每个请求与在服务器上创建的会话 ID 相关联，然后由客户端使用 cookie 存储。每个请求都以 HTTP 头部的形式发送 cookie（会话 ID），服务器对其进行验证，并将状态（用户会话）与每个请求相关联。在现代应用程序中（我们将在下一节中更详细地介绍），服务器端的会话 ID 被 JWT 替代。以下图表显示了 JWT 的工作原理：

![](img/f181c449-77aa-498b-a77a-2104f1bb75ad.png)

图 1：JWT 在现代应用程序中的工作原理

在这种情况下，Web 服务器不会创建用户会话，并且对于需要有状态应用程序的用户会话管理功能被卸载到其他机制。

在 Spring 框架的世界中，Spring Session 模块可以用于将会话从 Web 服务器外部化到中央持久性存储（Redis、Couchbase 等）。每个包含有效令牌（JWT）的请求都会针对这个外部的真实性和有效性存储进行验证。验证成功后，应用程序可以生成有效令牌并将其作为响应发送给客户端。然后客户端可以将此令牌存储在其使用的任何客户端存储机制中（sessionStorage、localStorage、cookies 等，在浏览器中）。使用 Spring Security，我们可以验证此令牌以确定用户的真实性和有效性，然后执行所需的操作。本章的后续部分（简单 REST API 安全性）中有一个专门的示例，该示例使用基本身份验证机制，并在成功时创建 JWT。随后的请求使用 HTTP 标头中的令牌，在服务器上进行验证以访问其他受保护的资源。

以下几点突出了使用 JWT 的一些优点：

+   **更好的性能**：每个到达服务器的请求都必须检查发送的令牌的真实性。JWT 的真实性可以在本地检查，不需要外部调用（比如到数据库）。这种本地验证性能良好，减少了请求的整体响应时间。

+   **简单性**：JWT 易于实现和简单。此外，它是行业中已经建立的令牌格式。有许多知名的库可以轻松使用 JWT。

# 令牌的结构

与常见的安全机制（如加密、混淆和隐藏）不同，JWT 不会加密或隐藏其中包含的数据。但是，它确实让目标系统检查令牌是否来自真实来源。JWT 的结构包括标头、有效载荷和签名。如前所述，与其加密，JWT 中包含的数据被编码，然后签名。编码的作用是以一种可被各方接受的方式转换数据，签名允许我们检查其真实性，实际上是其来源：

```java
JWT = header.payload.signature
```

让我们更详细地了解构成令牌的每个组件。

# 标头

这是一个 JSON 对象，采用以下格式。它提供有关如何计算签名的信息：

```java
{
 "alg": "HS256",
 "typ": "JWT"
}
```

`typ`的值指定对象的类型，在这种情况下是`JWT`。`alg`的值指定用于创建签名的算法，在这种情况下是`HMAC-SHA256`。

# 有效载荷

有效载荷形成 JWT 中存储的实际数据（也称为**声明**）。根据应用程序的要求，您可以将任意数量的声明放入 JWT 有效载荷组件中。有一些预定义的声明，例如`iss`（发行人）、`sub`（主题）、`exp`（过期时间）、`iat`（发布时间）等，可以使用，但所有这些都是可选的：

```java
{
 "sub": "1234567890",
 "username": "Test User",
 "iat": 1516239022
}
```

# 签名

签名形成如下：

1.  `标头`是`base64`编码的：`base64(标头)`。

1.  `有效载荷`是`base64`编码的：`base64(有效载荷)`。

1.  现在用中间的`"."`连接*步骤 1*和*步骤 2*中的值：

```java
base64UrlEncode(header) + "." +base64UrlEncode(payload)
```

1.  现在，签名是通过使用标头中指定的算法对*步骤 3*中获得的值进行哈希，然后将其与您选择的秘密文本（例如`packtpub`）附加而获得的：

```java
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  packtpub
)
```

最终的 JWT 如下所示：

```java
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IlRlc3QgVXNlciIsImlhdCI6MTUxNjIzOTAyMn0.yzBMVScwv9Ln4vYafpTuaSGa6mUbpwCg84VOhVTQKBg
```

网站[`jwt.io/`](https://jwt.io/)是我在任何 JWT 需求中经常访问的地方。此示例中使用的示例数据来自该网站：

![](img/6e508052-e050-47bd-8b70-db78d22598e2.png)

图 2：来自 https://jwt.io/的屏幕截图

# 现代应用程序架构

大多数现代应用的前端不是使用服务器端 Web 应用程序框架构建的，比如 Spring MVC、Java Server Faces（JSF）等。事实上，许多应用是使用完整的客户端框架构建的，比如 React（要成为完整的框架，必须与其他库结合使用）、Angular 等。前面的陈述并不意味着这些服务器端 Web 应用程序框架没有任何用处。根据您正在构建的应用程序，每个框架都有特定的用途。

在使用客户端框架时，一般来说，客户端代码（HTML、JS、CSS 等）并不安全。然而，渲染这些动态页面所需的数据是安全的，位于 RESTful 端点之后。

为了保护 RESTful 后端，使用 JWT 在服务器和客户端之间交换声明。JWT 实现了两方之间令牌的无状态交换，并通过服务器消除了会话管理的负担（不再需要多个服务器节点之间的粘性会话或会话复制），从而使应用程序能够以成本效益的方式水平扩展：

![](img/841b4165-d61d-41e8-89ec-89647e225641.png)

图 3：基于 API 的现代应用架构

# SOFEA

**面向服务的前端架构**（**SOFEA**）是一种在过去获得了流行的架构风格，当时**面向服务的架构**（**SOA**）在许多企业中变得普遍。在现代，SOA 更多地采用基于微服务的架构，后端被减少为一堆 RESTful 端点。另一方面，客户端变得更厚，并使用客户端 MVC 框架，比如 Angular 和 React，只是举几个例子。然而，SOFEA 的核心概念，即后端只是端点，前端（UI）变得更厚，是现代 Web 应用程序开发中每个人都考虑的事情。

SOFEA 的一些优点如下：

+   这种客户端比我们过去看到的薄客户端 Web 应用程序更厚（类似于厚客户端应用程序）。在页面的初始视图/渲染之后，所有资产都从服务器下载并驻留/缓存在客户端（浏览器）上。此后，只有在客户端通过 XHR（Ajax）调用需要数据时，才会与服务器联系。

+   客户端代码下载后，只有数据从服务器流向客户端，而不是呈现代码（HTML、JavaScript 等），更好地利用带宽。由于传输的数据量较少，响应时间更快，使应用程序性能更好。

+   任意数量的客户端可以利用相同的 RESTful 服务器端点编写，充分重用 API。

+   这些端点可以外部化会话（在 Spring 框架中，有一个称为**Spring Session**的模块，可以用来实现这种技术能力），从而轻松实现服务器的水平扩展。

+   在项目中，通过由一个团队管理的 API 和由另一个团队管理的 UI 代码，更好地分离团队成员的角色。

# 响应式 REST API

在第四章中，*使用 CAS 和 JAAS 进行身份验证*，我们详细介绍了响应式 Spring WebFlux Web 应用程序框架。我们还深入研究了 Spring 框架和其他 Spring 模块提供的许多响应式编程支持。无论是有意还是无意，我们在上一章的示例部分创建了一个响应式 REST API。我们使用了处理程序和路由器机制来创建一个 RESTful 应用程序，并使用了*BASIC*身份验证机制进行了安全保护。

我们看到了`WebClient`（一种调用 REST API 的响应式方式，与使用阻塞的`RestTemplate`相对）和`WebTestClient`（一种编写测试用例的响应式方式）的工作原理。我们还以响应式方式使用 Spring Data，使用 MongoDB 作为持久存储。

我们不会在这里详细介绍这些方面；我们只会提到，如果你愿意，你可以通过阅读第四章中的部分来熟悉这个主题，*使用 CAS 和 JAAS 进行身份验证*。在本章中，我们将继续上一章的内容，介绍使用 JWT 进行 REST API 安全，然后介绍使用 OAuth 进行 REST API 安全（实现自定义提供者，而不是使用公共提供者，如 Google、Facebook 等）。

# 简单的 REST API 安全

我们将使用我们在第五章中创建的示例，*与 Spring WebFlux 集成*（*spring-boot-spring-webflux*），并通过以下方式进行扩展：

+   将 JWT 支持引入到已使用基本身份验证进行安全保护的现有 Spring WebFlux 应用程序中。

+   创建一个新的控制器（`路径/auth/**`），将有新的端点，使用这些端点可以对用户进行身份验证。

+   使用基本身份验证或 auth REST 端点，我们将在服务器上生成 JWT 并将其作为响应发送给客户端。客户端对受保护的 REST API 的后续调用可以通过使用作为 HTTP 头部（授权，令牌）提供的 JWT 来实现。

我们无法详细介绍这个项目的每一个细节（我们在规定的页数内需要涵盖一个更重要的主题）。然而，在浏览示例时，重要的代码片段将被列出并进行详细解释。

# Spring Security 配置

在 Spring Security 配置中，我们调整了`springSecurityFilterChain` bean，如下面的代码片段所示：

```java
@Bean
public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http){
    AuthenticationWebFilter authenticationJWT = new AuthenticationWebFilter(new     
    UserDetailsRepositoryReactiveAuthenticationManager(userDetailsRepository()));
    authenticationJWT.setAuthenticationSuccessHandler(new         
                                                    JWTAuthSuccessHandler());
    http.csrf().disable();
    http
      .authorizeExchange()
      .pathMatchers(WHITELISTED_AUTH_URLS)
      .permitAll()
      .and()
      .addFilterAt(authenticationJWT, SecurityWebFiltersOrder.FIRST)
      .authorizeExchange()
      .pathMatchers(HttpMethod.GET, "/api/movie/**").hasRole("USER")
      .pathMatchers(HttpMethod.POST, "/api/movie/**").hasRole("ADMIN")
      .anyExchange().authenticated()
      .and()
      .addFilterAt(new JWTAuthWebFilter(), SecurityWebFiltersOrder.HTTP_BASIC);
    return http.build();
}
```

正如你所看到的，我们配置了一个新的`AuthenticationWebFilter`和一个`AuthenticationSuccessHandler`。我们还有一个新的`JWTAuthWebFilter`类来处理基于 JWT 的身份验证。

我们将使用`ReactiveUserDetailsService`和硬编码的用户凭据进行测试，如下面的代码片段所示：

```java
@Bean
public MapReactiveUserDetailsService userDetailsRepository() {
    UserDetails user = User.withUsername("user").password("    
        {noop}password").roles("USER").build();
    UserDetails admin = User.withUsername("admin").password("
        {noop}password").roles("USER","ADMIN").build();
    return new MapReactiveUserDetailsService(user, admin);
}
```

# 身份验证成功处理程序

我们在 Spring Security 配置类中设置了自定义的`AuthenticationSuccessHandler`（该类的源代码将在下面显示）。在成功验证后，它将生成 JWT 并设置 HTTP 响应头：

+   **头部名称**：`Authorization`

+   **头部值**：`Bearer JWT`

让我们看一下下面的代码：

```java
public class JWTAuthSuccessHandler implements ServerAuthenticationSuccessHandler{
    @Override
    public Mono<Void> onAuthenticationSuccess(WebFilterExchange     
            webFilterExchange, Authentication authentication) {
        ServerWebExchange exchange = webFilterExchange.getExchange();
        exchange.getResponse()
            .getHeaders()
            .add(HttpHeaders.AUTHORIZATION, 
                    getHttpAuthHeaderValue(authentication));
        return webFilterExchange.getChain().filter(exchange);
    }
    private static String getHttpAuthHeaderValue(Authentication authentication){
        return String.join(" ","Bearer",tokenFromAuthentication(authentication));
    }
    private static String tokenFromAuthentication(Authentication authentication){
        return new JWTUtil().generateToken(
            authentication.getName(),
            authentication.getAuthorities());
    }
}
```

`JWTUtil`类包含许多处理 JWT 的实用方法，例如生成令牌、验证令牌等。`JWTUtil`类中的`generateToken`方法如下所示：

```java
public static String generateToken(String subjectName, Collection<? extends             GrantedAuthority> authorities) {
    JWTClaimsSet claimsSet = new JWTClaimsSet.Builder()
        .subject(subjectName)
        .issuer("javacodebook.com")
        .expirationTime(new Date(new Date().getTime() + 30 * 1000))
        .claim("auths", authorities.parallelStream().map(auth ->                             (GrantedAuthority) auth).map(a ->                                 
            a.getAuthority()).collect(Collectors.joining(",")))
        .build();
    SignedJWT signedJWT = new SignedJWT(new JWSHeader(JWSAlgorithm.HS256),         claimsSet);
    try {
        signedJWT.sign(JWTUtil.getJWTSigner());
    } catch (JOSEException e) {
        e.printStackTrace();
    }
    return signedJWT.serialize();
}
```

# 自定义 WebFilter，即 JWTAuthWebFilter

我们的自定义`WebFilter`，名为`JWTAuthWebFilter`，负责将接收到的 JWT 令牌转换为 Spring Security 理解的适当类。它使用了一个名为`JWTAuthConverter`的转换器，该转换器执行了许多操作，如下所示：

+   获取授权`payload`

+   通过丢弃`Bearer`字符串来提取令牌

+   验证令牌

+   创建一个 Spring Security 理解的`UsernamePasswordAuthenticationToken`类

下面的代码片段显示了`JWTAuthWebFilter`类及其上面列出的操作的重要方法。

```java
public class JWTAuthConverter implements Function<ServerWebExchange,             
        Mono<Authentication>> {
    @Override
    public Mono<Authentication> apply(ServerWebExchange serverWebExchange) {
        return Mono.justOrEmpty(serverWebExchange)
            .map(JWTUtil::getAuthorizationPayload)
            .filter(Objects::nonNull)
            .filter(JWTUtil.matchBearerLength())
            .map(JWTUtil.getBearerValue())
            .filter(token -> !token.isEmpty())
            .map(JWTUtil::verifySignedJWT)
            .map(JWTUtil::getUsernamePasswordAuthenticationToken)
            .filter(Objects::nonNull);
    }
}
```

在此转换之后，使用 Spring Security 进行实际的身份验证，该身份验证在应用程序中设置了`SecurityContext`，如下面的代码片段所示：

```java
@Override
public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
    return this.getAuthMatcher().matches(exchange)
        .filter(matchResult -> matchResult.isMatch())
        .flatMap(matchResult -> this.jwtAuthConverter.apply(exchange))
        .switchIfEmpty(chain.filter(exchange).then(Mono.empty()))
        .flatMap(token -> authenticate(exchange, chain, token));
}
//..more methods
private Mono<Void> authenticate(ServerWebExchange exchange,
                              WebFilterChain chain, Authentication token) {
    WebFilterExchange webFilterExchange = new WebFilterExchange(exchange, chain);
    return this.reactiveAuthManager.authenticate(token)
      .flatMap(authentication -> onAuthSuccess(authentication, 
          webFilterExchange));
}
private Mono<Void> onAuthSuccess(Authentication authentication, WebFilterExchange 
        webFilterExchange) {
    ServerWebExchange exchange = webFilterExchange.getExchange();
    SecurityContextImpl securityContext = new SecurityContextImpl();
    securityContext.setAuthentication(authentication);
    return this.securityContextRepository.save(exchange, securityContext)
        .then(this.authSuccessHandler
        .onAuthenticationSuccess(webFilterExchange, authentication))
        .subscriberContext(ReactiveSecurityContextHolder.withSecurityContext(
            Mono.just(securityContext)));
}
```

`JWTAuthWebFilter`类的过滤器方法进行必要的转换，然后`authenticate`方法进行实际的身份验证，最后调用`onAuthSuccess`方法。

# 新的控制器类

我们有两个控制器，分别是`DefaultController`（映射到`/`和`/login`路径）和`AuthController`（映射到`/auth`主路由和`/token`子路由）。`/auth/token`路径可用于检索令牌，该令牌可用于后续的 API 调用（`Bearer <Token>`）。`AuthController`的代码片段如下所示：

```java
@RestController
@RequestMapping(path = "/auth", produces = { APPLICATION_JSON_UTF8_VALUE })
public class AuthController {

    @Autowired
    private MapReactiveUserDetailsService userDetailsRepository;
        @RequestMapping(method = POST, value = "/token")
        @CrossOrigin("*")
        public Mono<ResponseEntity<JWTAuthResponse>> token(@RequestBody     
                JWTAuthRequest jwtAuthRequest) throws AuthenticationException {
            String username =  jwtAuthRequest.getUsername();
            String password =  jwtAuthRequest.getPassword();
            return userDetailsRepository.findByUsername(username)
               .map(user -> ok().contentType(APPLICATION_JSON_UTF8).body(
                 new JWTAuthResponse(JWTUtil.generateToken(user.getUsername(),                  user.getAuthorities()), user.getUsername())))
                 .defaultIfEmpty(notFound().build());
        }
    }
}
```

# 运行应用程序并进行测试

使用下面显示的 Spring Boot 命令运行应用程序：

```java
mvn spring-boot:run
```

我将使用 Postman 执行 REST 端点。

您可以通过以下两种方法获得令牌，并在随后的调用中包含它：

+   如果使用基本身份验证凭据访问任何路由，在响应头中，您应该获得令牌。我将使用`/login`路径与基本身份验证（授权头）获取令牌，如图所示：

![](img/d417a89e-0a43-4612-964b-8d12a8c02328.png)

图 4：在 Postman 中使用基本身份验证获取令牌

+   使用 JSON 形式的基本身份验证凭据（使用`JWTAuthRequest`类），如图所示，在 Postman 中访问`/auth/token`端点：

![](img/b601600b-0c3b-4c15-8c9a-9414f2a45049.png)

图 5：使用 JSON 中的基本身份验证凭据使用/auth/token 端点获取令牌

使用检索到的令牌，如图所示，在 Postman 中调用电影端点：

![](img/967a0611-eeb5-44d1-91ad-c9373acbc3cb.png)

图 6：在 Postman 中使用 JWT 令牌检索电影列表

这完成了我们正在构建的示例。在这个示例中，我们使用 JWT 保护了 REST API，并使用 Spring Security 进行了验证。如前所述，这是您可以使用 Spring Security 和 JWT 保护 REST API 的基本方法。

# 高级 REST API 安全性

REST API 可以通过您的 Web 应用程序中的另一种机制进行保护，即 OAuth。

OAuth 是一个授权框架，允许其他应用程序使用正确的凭据访问存储在 Google 和 Facebook 等平台上的部分/有限用户配置文件详细信息。认证部分被委托给这些服务，如果成功，适当的授权将被授予调用客户端/应用程序，这可以用来访问受保护的资源（在我们的情况下是 RESTful API）。

我们已经在第三章中看到了使用公共身份验证提供程序的 OAuth 安全性，*使用 CAS 和 JAAS 进行身份验证*（在*OAuth 2 和 OpenID 连接*部分）。但是，我们不需要使用这些公共提供程序；您可以选择使用自己的提供程序。在本章中，我们将涵盖一个这样的示例，我们将使用自己的身份验证提供程序并保护基于 Spring Boot 的响应式 REST 端点。

在进入示例之前，我们需要更多地了解 OAuth，并且需要了解它的各个组件。我们已经在[第三章](https://cdp.packtpub.com/hands_on_spring_security_5_for_reactive_applications/wp-admin/post.php?post=30&action=edit#post_28)中详细介绍了 OAuth 的许多细节，*使用 CAS 和 JAAS 进行身份验证*。我们将在本节中添加这些细节，然后通过代码示例进行讲解。

# OAuth2 角色

OAuth 为用户和应用程序规定了四种角色。这些角色之间的交互如下图所示：

![](img/6d946741-9ac2-4e7f-8504-a46b832ebb27.png)

图 7：OAuth 角色交互

我们将详细了解这些 OAuth 角色中的每一个。

# 资源所有者

这是拥有所需受保护资源的消费客户端应用程序的用户。如果我们以 Facebook 或 Google 作为身份验证提供程序，资源所有者就是在这些平台上保存数据的实际用户。

# 资源服务器

这是以托管 API 的形式拥有受保护资源的服务器。如果以 Google 或 Facebook 为例，它们以 API 的形式保存配置文件信息以及其他信息。如果客户端应用程序成功进行身份验证（使用用户提供的凭据），然后用户授予适当的权限，他们可以通过公开的 API 访问这些信息。

# 客户端

这是用于访问资源服务器上可用的受保护资源的应用程序。如果用户成功验证并且客户端应用程序被用户授权访问正确的信息，客户端应用程序可以检索数据。

# 授权服务器

这是一个验证和授权客户端应用程序访问资源所有者和资源服务器上拥有的受保护资源的服务器。同一个服务器执行这两个角色并不罕见。

要参与 OAuth，您的应用程序必须首先向服务提供商（如 Google、Facebook 等）注册，以便通过提供应用程序名称、应用程序 URL 和回调 URL 进行身份验证。成功注册应用程序与服务提供商后，您将获得两个应用程序唯一的值：`client application_id`和`client_secret`。`client_id`可以公开，但`client_secret`保持隐藏（私有）。每当访问服务提供商时，都需要这两个值。以下图显示了这些角色之间的交互：

![](img/5115c019-94ee-4e31-abfd-1458d4305577.png)

图 8：OAuth 角色交互

前面图中的步骤在这里有详细介绍：

1.  客户端应用程序请求资源所有者授权它们访问受保护资源

1.  如果资源所有者授权，授权授予将发送到客户端应用程序

1.  客户端应用程序请求令牌，使用资源所有者提供的授权以及来自授权服务器的身份验证凭据

1.  如果客户端应用程序的凭据和授权有效，授权服务器将向客户端应用程序发放访问令牌

1.  客户端应用程序使用提供的访问令牌访问资源服务器上的受保护资源

1.  如果客户端应用程序发送的访问令牌有效，资源服务器将允许访问受保护资源

# 授权授予类型

如图所示，为了让客户端开始调用 API，它需要以访问令牌的形式获得授权授予。OAuth 提供了四种授权类型，可以根据不同的应用程序需求使用。关于使用哪种授权授予类型的决定留给了客户端应用程序。

# 授权码流程

这是一种非常常用的授权类型，它在服务器上进行重定向。它非常适用于服务器端应用程序，其中源代码托管在服务器上，客户端上没有任何内容。以下图解释了授权码授权类型的流程：

![](img/54908998-f191-4a15-bc07-7a7469a08aa6.png)

图 9：授权码流程

前面图中的步骤在这里有详细介绍：

1.  受保护资源的资源所有者将在浏览器中呈现一个屏幕，以授权请求。这是一个示例授权链接：`https://<DOMAIN>/oauth/authorize?response_type=code&client_id=<CLIENT_ID>&redirect_uri=<CALLBACK_URL>&scope=<SCOPE>`。

这是上述链接中的重要查询参数：

+   +   `client_id`：我们在向服务提供商注册应用程序时获得的客户端应用程序 ID

+   `redirect_uri`：成功授权后，服务器将重定向到提供的 URL

+   `response_type`：客户端用来向服务器请求授权码的非常重要的参数

+   `scope`：指定所需的访问级别

1.  如果资源所有者（用户）允许，他们点击授权链接，该链接被发送到授权服务器。

1.  如果发送到授权服务器的授权请求经过验证并且成功，客户端将从授权服务器接收授权码授权，附加为回调 URL（`<CALLBACK_URL>?code=<AUTHORIZATION_CODE>`）中的查询参数，指定在`步骤 1`中。

1.  使用授权授予，客户端应用程序从授权服务器请求访问令牌（`https://<DOMAIN>/oauth/token?client_id=<CLIENT_ID>&client_secret=<CLIENT_SECRET>&grant_type=authorization_code&code=<AUTHORIZATION_CODE>&redirect_uri=CALLBACK_URL`）。

在此 URL 中，还必须传递客户端应用程序的`client_secret`，以及声明传递的代码是授权代码的`grant_type`参数。

1.  授权服务器验证凭据和授权授予，并向客户端应用程序发送访问令牌，最好以 JSON 形式。

1.  客户端应用程序使用*步骤 5*中收到的访问令牌调用资源服务器上的受保护资源。

1.  如果*步骤 5*中提供的访问令牌有效，则资源服务器允许访问受保护资源。

# 隐式流

这在移动和 Web 应用程序中通常使用，并且也基于重定向工作。以下图表解释了隐式代码授权类型的流程：

![](img/8da3f363-ff0a-4d4a-9058-9247f0dc33e3.png)

图 10：隐式流

前面图表中的步骤在这里进行了详细解释：

1.  资源所有者被呈现一个屏幕（浏览器）来授权请求。这是一个示例授权链接：`https://<DOMAIN>/oauth/authorize?response_type=token&client_id=CLIENT_ID&redirect_uri=CALLBACK_URL&scope=<SCOPE>`。

重要的是要注意，前面链接中指定的`response_type`是`token`。这表示服务器应该给出访问令牌（这是与前一节讨论的授权代码流授权类型的主要区别之一）。

1.  如果资源所有者（用户）允许此操作，则点击授权链接，该链接将发送到授权服务器。

1.  用户代理（浏览器或移动应用程序）在指定的`CALLBACK_URL`中接收访问令牌（`https://<CALLBACK_URL>#token=<ACCESS_TOKEN>`）。

1.  用户代理转到指定的`CALLBACK_URL`，保留访问令牌。

1.  客户端应用程序打开网页（使用任何机制），从`CALLBACK_URL`中提取访问令牌。

1.  客户端应用程序现在可以访问访问令牌。

1.  客户端应用程序使用访问令牌调用受保护的 API。

# 客户端凭据

这是最简单的授权方式之一。客户端应用程序将凭据（客户端的服务帐户）与`client_ID`和`client_secret`一起发送到授权服务器。如果提供的值有效，授权服务器将发送访问令牌，该令牌可用于访问受保护的资源。

# 资源所有者密码凭据

这是另一种简单易用的类型，但被认为是所有类型中最不安全的。在这种授权类型中，资源所有者（用户）必须直接在客户端应用程序界面中输入他们的凭据（请记住，客户端应用程序可以访问资源所有者的凭据）。然后客户端应用程序使用这些凭据发送到授权服务器以获取访问令牌。只有在资源所有者完全信任他们提供凭据给服务提供者的应用程序时，这种授权类型才有效，因为这些凭据通过客户端应用程序的应用服务器传递（因此如果客户端应用程序决定的话，它们可以被存储）。

# 访问令牌和刷新令牌

客户端应用程序可以使用访问令牌从资源服务器检索信息，该信息在令牌被视为有效的规定时间内。之后，服务器将使用适当的 HTTP 响应错误代码拒绝请求。

OAuth 允许授权服务器在访问令牌的同时发送另一个令牌，即刷新令牌。当访问令牌过期时，客户端应用程序可以使用第二个令牌请求授权服务器提供新的访问令牌。

# Spring Security OAuth 项目

目前在 Spring 生态系统中，OAuth 支持已扩展到许多项目，包括 Spring Security Cloud、Spring Security OAuth、Spring Boot 和 Spring Security（5.x+）的版本。这在社区内造成了很多混乱，没有单一的所有权来源。Spring 团队采取的方法是整合这一切，并开始维护与 Spring Security 有关的所有 OAuth 内容。预计将在 2018 年底之前将 OAuth 的重要组件，即授权服务器、资源服务器以及对 OAuth2 和 OpenID Connect 1.0 的下一级支持，添加到 Spring Security 中。Spring Security 路线图清楚地说明，到 2018 年中期，将添加对资源服务器的支持，并在 2018 年底之前添加对授权服务器的支持。

在撰写本书时，Spring Security OAuth 项目处于维护模式。这意味着将发布用于修复错误/安全性问题的版本，以及一些较小的功能。未来不计划向该项目添加重大功能。

各种 Spring 项目中提供的完整 OAuth2 功能矩阵可以在[`github.com/spring-projects/spring-security/wiki/OAuth-2.0-Features-Matrix`](https://github.com/spring-projects/spring-security/wiki/OAuth-2.0-Features-Matrix)找到。

在撰写本书时，我们需要实现 OAuth 的大多数功能都作为 Spring Security OAuth 项目的一部分可用，该项目目前处于维护模式。

# OAuth2 和 Spring WebFlux

在撰写本书时，Spring Security 中尚未提供 Spring WebFlux 应用程序的全面 OAuth2 支持。但是，社区对此非常紧迫，并且许多功能正在逐渐加入 Spring Security。许多示例也正在 Spring Security 项目中展示使用 Spring WebFlux 的 OAuth2。在第五章中，*与 Spring WebFlux 集成*，我们详细介绍了一个这样的示例。在撰写本书时，Spring Security OAuth2 对 Spring MVC 有严格的依赖。

# Spring Boot 和 OAuth2

在撰写本书时，Spring Boot 宣布不再支持 Spring Security OAuth 模块。相反，它将从现在开始使用 Spring Security 5.x OAuth2 登录功能。

一个名为 Spring Security OAuth Boot 2 Autoconfig 的新模块（其在 pom.xml 中的依赖如下代码片段所示），从 Spring Boot 1.5.x 移植而来，可用于将 Spring Security 与 Spring Boot 集成：

```java
<dependency>
 <groupId>org.springframework.security.oauth.boot</groupId>
 <artifactId>spring-security-oauth2-autoconfigure</artifactId>
</dependency>
```

项目源代码可以在[`github.com/spring-projects/spring-security-oauth2-boot`](https://github.com/spring-projects/spring-security-oauth2-boot)找到。此模块的完整文档可以在[`docs.spring.io/spring-security-oauth2-boot/docs/current-SNAPSHOT/reference/htmlsingle/`](https://docs.spring.io/spring-security-oauth2-boot/docs/current-SNAPSHOT/reference/htmlsingle/)找到。

# 示例项目

在我们的示例项目中，我们将设置自己的授权服务器，我们将对其进行授权，以授权通过我们的资源服务器公开的 API。我们在我们的资源服务器上公开了电影 API，并且客户端应用程序将使用应用程序进行身份验证（客户端应用程序受 Spring Security 保护），然后尝试访问其中一个电影 API，此时 OAuth 流程将启动。成功与授权服务器进行授权检查后，客户端将获得对所请求的电影 API 的访问权限。

我们将有一个包含三个 Spring Boot 项目的父项目：`oauth-authorization-server`、`oauth-resource-server`和`oauth-client-app`：

![](img/0a82e06b-986d-4bde-8d68-05c3fd9afa17.png)

图 11：IntelliJ 中的项目结构

我们现在将在后续章节中查看每个单独的 Spring Boot 项目。完整的源代码可在书的 GitHub 页面上的`spring-boot-spring-security-oauth`项目下找到。

# 授权服务器

这是一个传统的 Spring Boot 项目，实现了授权服务器 OAuth 角色。

# Maven 依赖

要包含在 Spring Boot 项目的`pom.xml`文件中的主要依赖项如下面的代码片段所示：

```java
<!--Spring Boot-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<!--OAuth-->
<dependency>
  <groupId>org.springframework.security.oauth</groupId>
  <artifactId>spring-security-oauth2</artifactId>
  <version>2.3.2.RELEASE</version>
</dependency>
<!--JWT-->
<dependency>
  <groupId>org.springframework.security</groupId>
  <artifactId>spring-security-jwt</artifactId>
  <version>1.0.9.RELEASE</version>
</dependency>
```

# Spring Boot 运行类

这个 Spring Boot 的`run`类并没有什么特别之处，如下面的代码片段所示：

```java
@SpringBootApplication
public class OAuthAuthorizationServerRun extends SpringBootServletInitializer {
  public static void main(String[] args) {
      SpringApplication.run(OAuthAuthorizationServerRun.class, args);
  }
}
```

# Spring 安全配置

Spring 安全配置类扩展了`WebSecurityConfigurerAdapter`。我们将重写三个方法，如下面的代码片段所示：

```java
@Configuration
@EnableWebSecurity
public class SpringSecurityConfig extends WebSecurityConfigurerAdapter {
  @Autowired
  private BCryptPasswordEncoder passwordEncoder;
  @Autowired
  public void globalUserDetails(final AuthenticationManagerBuilder auth) throws 
        Exception {
      auth
          .inMemoryAuthentication()
          .withUser("user").password(passwordEncoder.encode("password"))
          .roles("USER")
          .and()
          .withUser("admin").password(passwordEncoder.encode("password"))
          .roles("USER", "ADMIN");
  }
  //...
}
```

我们`autowire`密码编码器。然后我们重写以下方法：`globalUserDetails`、`authenticationManagerBean`和`configure`。这里没有什么特别要提到的。我们在内存中定义了两个用户（用户和管理员）。

# 授权服务器配置

这是这个 Spring Boot 项目中最重要的部分，我们将在其中设置授权服务器配置。我们将使用一个新的注解`@EnableAuthorizationServer`。我们的配置类将扩展`AuthorizationServerConfigurerAdapter`。我们将使用 JWT 令牌存储，并展示一个令牌增强器，使用它可以在需要时增强 JWT 令牌的声明。这个配置类中最重要的方法被提取为下面的代码片段：

```java
@Override
public void configure(final ClientDetailsServiceConfigurer clients) throws 
        Exception {
  clients.inMemory()
     .withClient("oAuthClientAppID")
     .secret(passwordEncoder().encode("secret"))
     .authorizedGrantTypes("password", "authorization_code", "refresh_token")
     .scopes("movie", "read", "write")
     .accessTokenValiditySeconds(3600)
     .refreshTokenValiditySeconds(2592000)
     .redirectUris("http://localhost:8080/movie/", 
        "http://localhost:8080/movie/index");
}
```

这是我们设置与客户端相关的 OAuth 配置的地方。我们只设置了一个客户端，并使用内存选项使示例更容易理解。在整个应用程序中，我们将使用`BCrypt`作为我们的密码编码器。我们的客户端应用程序的客户端 ID 是`oAuthClientAppID`，客户端密钥是`secret`。我们设置了三种授权类型，访问客户端时需要指定必要的范围（movie、read 和 write）。执行成功后，授权服务器将重定向到指定的 URL（`http://localhost:8080/movie/`或`http://localhost:8080/movie/index`）。如果客户端没有正确指定 URL，服务器将抛出错误。

JWT 令牌存储和增强相关方法如下面的代码片段所示：

```java
@Bean
@Primary
public DefaultTokenServices tokenServices() {
  final DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
  defaultTokenServices.setTokenStore(tokenStore());
  defaultTokenServices.setSupportRefreshToken(true);
  return defaultTokenServices;
}
@Override
public void configure(final AuthorizationServerEndpointsConfigurer endpoints) 
    throws Exception {
  final TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
  tokenEnhancerChain.setTokenEnhancers(Arrays.asList(tokenEnhancer(), 
    accessTokenConverter()));
  endpoints.tokenStore(tokenStore()).tokenEnhancer(tokenEnhancerChain)
    .authenticationManager(authenticationManager);
}
@Bean
public TokenStore tokenStore() {
  return new JwtTokenStore(accessTokenConverter());
}
@Bean
public JwtAccessTokenConverter accessTokenConverter() {
  final JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
  converter.setSigningKey("secret");
  return converter;
}
@Bean
public TokenEnhancer tokenEnhancer() {
  return new CustomTokenEnhancer();
}
```

在这段代码中，我们指定了将在`tokenStore`方法中使用的令牌存储，并声明了一个`tokenEnhancer` bean。为了展示令牌增强器，我们将使用一个名为`CustomTokenEnhancer`的自定义类；该类如下面的代码片段所示：

```java
public class CustomTokenEnhancer implements TokenEnhancer {
  @Override
  public OAuth2AccessToken enhance(OAuth2AccessToken accessToken, 
    OAuth2Authentication authentication) {
      final Map<String, Object> additionalInfo = new HashMap<>();
      additionalInfo.put("principalinfo", 
        authentication.getPrincipal().toString());
      ((DefaultOAuth2AccessToken)accessToken)
        .setAdditionalInformation(additionalInfo);
      return accessToken;
  }
}
```

自定义令牌`enhancer`类实现了`TokenEnhancer`。我们只是将新信息（`principalinfo`）添加到包含`principal`对象的`toString`版本的 JWT 令牌中。

# 应用程序属性

由于我们在本地运行了所有三个服务器，我们必须指定不同的端口。此外，授权服务器运行在不同的上下文路径上是很重要的。下面的代码片段显示了我们在`application.properties`文件中的内容：

```java
server.servlet.context-path=/oauth-server
server.port=8082
```

作为一个 Spring Boot 项目，可以通过执行`mvn spring-boot:run`命令来运行。

# 资源服务器

这是一个传统的 Spring Boot 项目，实现了资源服务器 OAuth 角色。

# Maven 依赖

在我们的`pom.xml`中，我们不会添加任何新的内容。我们在授权服务器项目中使用的相同依赖项也适用于这里。

# Spring Boot 运行类

这是一个典型的 Spring Boot `run`类，我们在其中放置了`@SpringBootApplication`注解，它在幕后完成了所有的魔术。同样，在我们的 Spring Boot 运行类中没有特定于这个项目的内容。

# 资源服务器配置

这是主要的资源服务器配置类，我们在其中使用`@EnableResourceServer`注解，并将其扩展自`ResourceServerConfigurerAdapter`，如下面的代码片段所示：

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
  @Autowired
  private CustomAccessTokenConverter customAccessTokenConverter;
  @Override
  public void configure(final HttpSecurity http) throws Exception {
      http.sessionManagement()
        .sessionCreationPolicy(SessionCreationPolicy.ALWAY)
        .and()
        .authorizeRequests().anyRequest().permitAll();
  }
  @Override
  public void configure(final ResourceServerSecurityConfigurer config) {
      config.tokenServices(tokenServices());
  }
  @Bean
  public TokenStore tokenStore() {
      return new JwtTokenStore(accessTokenConverter());
  }
  @Bean
  public JwtAccessTokenConverter accessTokenConverter() {
      final JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
      converter.setAccessTokenConverter(customAccessTokenConverter);
      converter.setSigningKey("secret");
      converter.setVerifierKey("secret");
      return converter;
  }
  @Bean
  @Primary
  public DefaultTokenServices tokenServices() {
      final DefaultTokenServices defaultTokenServices = 
        new DefaultTokenServices();
      defaultTokenServices.setTokenStore(tokenStore());
      return defaultTokenServices;
  }
}
```

# Spring 安全配置

作为资源服务器，我们启用了全局方法安全，以便每个暴露 API 的方法都受到保护，如下面的代码片段所示：

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SpringSecurityConfig extends GlobalMethodSecurityConfiguration {
  @Override
  protected MethodSecurityExpressionHandler createExpressionHandler() {
      return new OAuth2MethodSecurityExpressionHandler();
  }
}
```

在这里，我们使用`OAuth2MethodSecurityExpressionHandler`作为方法安全异常处理程序，以便我们可以使用注解，如下所示：

```java
@PreAuthorize("#oauth2.hasScope('movie') and #oauth2.hasScope('read')")
```

# Spring MVC 配置类

我们在之前的章节中详细介绍了 Spring MVC 配置。在我们的示例中，这是一个非常基本的 Spring MVC `config`类，其中使用了`@EnableWebMvc`并实现了`WebMvcConfigurer`。

# 控制器类

我们有一个控制器类，只公开一个方法（我们可以进一步扩展以公开更多的 API）。这个方法列出了硬编码的电影列表中的所有电影，位于 URL`/movie`下，如下面的代码片段所示：

```java
@RestController
public class MovieController {
   @RequestMapping(value = "/movie", method = RequestMethod.GET)
   @ResponseBody
   @PreAuthorize("#oauth2.hasScope('movie') and #oauth2.hasScope('read')")
   public Movie[] getMovies() {
      initIt();//Movie list initialization
      return movies;
   }
   //…
}
```

我们使用了一个`Movie`模型类，利用了`lombok`库的所有功能，如下面的代码片段所示：

```java
@Data
@ToString
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Movie {
  private Long id;
  private String title;
  private String genre;
}
```

它有三个属性，注解将完成所有的魔术并保持模型简洁。

# 应用程序属性

与授权服务器类似，`application.properties`只有上下文路径和端口分配。

作为一个 Spring Boot 项目，可以通过执行`mvn spring-boot:run`命令来运行。

# 客户端应用程序

这是一个传统的 Spring Boot 项目，实现了客户端 OAuth 角色。

# Maven 依赖项

在我们的 Spring Boot `pom.xml`文件中，添加了`Thymeleaf`和`lombok`库的新 Maven 依赖项。其余部分都是典型的 Spring Boot `pom.xml`文件，你现在已经熟悉了。

# Spring Boot 类

在我们的示例 Spring Boot `run`类中，没有什么值得一提的。这是一个简单的类，包含了至关重要的`main`方法和`@SpringBootApplication`注解。

# OAuth 客户端配置

这是客户端应用程序中的主配置类，使用了`@EnableOAuth2Client`注解，如下面的代码片段所示：

```java
@Configuration
@EnableOAuth2Client
public class OAuthClientConfig {
  @Autowired
  private OAuth2ClientContext oauth2ClientContext;

  @Autowired
  @Qualifier("movieAppClientDetails")
  private OAuth2ProtectedResourceDetails movieAppClientDetails;

  @ConfigurationProperties(prefix = "security.oauth2.client.movie-app-client")
  @Bean
  public OAuth2ProtectedResourceDetails movieAppClientDetails() {
      return new AuthorizationCodeResourceDetails();
  }
  @Bean
  public BCryptPasswordEncoder passwordEncoder() {
      return new BCryptPasswordEncoder();
  }
  @Bean
  public OAuth2RestTemplate movieAppRestTemplate() {
      return new OAuth2RestTemplate(movieAppClientDetails, oauth2ClientContext);
  }
}
```

在这个类中要注意的重要方面是，我们通过在`application.yml`文件中配置客户端详细信息来初始化 OAuth2 REST 模板。

# Spring 安全配置

在 Spring 安全`config`类中，我们设置了可以用于登录到应用程序并访问受保护资源的用户凭据（内存中）。在`configure`方法中，一些资源被标记为受保护的，一些资源被标记为未受保护的。

# 控制器类

我们有两个控制器类，`SecuredController`和`NonSecuredController`。顾名思义，一个用于声明的受保护路由，另一个用于未受保护的路由。我们感兴趣的受保护控制器中的`main`方法如下面的代码片段所示：

```java
@RequestMapping(value = "/movie/index", method = RequestMethod.GET)
@ResponseBody
public Movie[] index() {
  Movie[] movies = movieAppRestTemplate
    .getForObject(movieApiBaseUri, Movie[].class);
  return movies;
}
```

我们将资源服务器项目中使用的`model`类复制到客户端应用程序项目中。在理想情况下，所有这些共同的东西都将被转换为可重用的 JAR，并设置为两个项目的依赖项。

# 模板

模板非常简单。应用程序的根上下文将用户重定向到一个未安全的页面。我们有自己的自定义登录页面，登录成功后，用户将被导航到一个包含指向受保护的 OAuth 支持的电影列表 API 的链接的受保护页面。

# 应用程序属性

在这个项目中，我们使用`application.yml`文件，代码如下：

```java
server:
  port: 8080
spring:
  thymeleaf:
    cache: false
security:
  oauth2:
    client:
      movie-app-client:
        client-id: oAuthClientAppID
        client-secret: secret
        user-authorization-uri: http://localhost:8082/oauth-server/oauth/authorize
        access-token-uri: http://localhost:8082/oauth-server/oauth/token
        scope: read, write, movie
        pre-established-redirect-uri: http://localhost:8080/movie/index
movie:
  base-uri: http://localhost:8081/oauth-resource/movie
```

这个 YML 文件的非常重要的方面是`movie-app-client`属性设置。同样，作为一个 Spring Boot 项目，可以通过执行`mvn spring-boot:run`命令来运行。

# 运行项目

使用 Spring Boot 的`mvn spring-boot:run`命令分别启动所有项目。我在 IntelliJ 中使用 Spring Dashboard，可以启动所有项目，如下面的截图所示：

![](img/f21f02f6-d40c-4e79-a32c-324886db5c03.png)

图 12：IntelliJ 中的 Spring Dashboard

导航到`http://localhost:8080`，您将被重定向到客户端应用程序的未安全页面，如下所示：

![](img/d35a94e3-cbdb-4023-9afd-1e6a48b46cfc.png)

图 13：客户端应用程序的未安全页面

点击链接，您将被带到自定义登录页面，如下所示：

![](img/88846da5-d718-452c-9193-ea3637bd0b20.png)

图 14：客户端应用程序的自定义登录页面

根据页面上的要求输入用户名/密码；然后，点击登录将带您到安全页面，如下所示：

![](img/e0fe14e3-c9f7-45a3-8341-763f88434956.png)

图 15：客户端应用程序中的安全页面

点击电影 API 链接，您将被带到 OAuth 流程，然后到授权服务器的默认登录页面以输入凭据，如下所示：

![](img/6dea5c50-4ea6-4b8e-9507-b3105e8ac5e3.png)

图 16：授权服务器登录页面

输入用户名/密码（我们将其保留为 user/password），然后点击登录按钮。您将被带到授权页面，如下面的截图所示：

![](img/c0889860-4aef-44de-bb61-3bd099664046.png)

图 17：授权服务器上的授权页面

点击授权，您将被带回客户端应用程序页面，显示来自资源服务器的所有电影，如下所示：

![](img/68d63f0a-c1a8-460b-8739-1fa9ae252fbd.png)

图 18：客户端应用程序中显示资源服务器上暴露的电影 API 的电影列表页面

通过这样，我们完成了我们的示例应用程序，其中我们实现了 OAuth 的所有角色。

# 总结

我们在本章开始时向您介绍了一些需要跟进的重要概念。然后我们介绍了现代 Web 应用程序所需的重要特征。我们迅速介绍了一个称为**SOFEA**的架构，它恰当地介绍了我们想要构建现代应用程序的方式。然后我们通过最简单的方式实现了 REST API 的安全性。

在接下来的部分中，我们介绍了如何以更高级的方式使用 OAuth 来保护 REST API，使用 JWT。我们通过介绍了许多关于 OAuth 的概念开始了本节，最后以使用 OAuth 和 JWT 的完整示例项目结束了本章。

阅读完本章后，您应该对 REST、OAuth 和 JWT 有清晰的理解。您还应该在下一章中，对在应用程序中暴露的 RESTful 端点使用 Spring Security 感到舒适。
