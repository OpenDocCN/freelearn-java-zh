# 17

# 微服务安全性使用 OAuth 2 和 JSON Web Tokens

在本章中，我们将探讨基于微服务的架构，并查看如何使用 **OAuth 2** 和 **JSON Web Tokens** (**JWT**) 在基于 Spring 的应用程序中确保微服务的安全性。

以下是在本章中将要涉及的主题列表：

+   **单体应用** 和 **微服务** 之间的基本区别

+   比较 **面向服务的架构** (**SOA**) 和微服务

+   **OAuth 2** 的概念架构以及它如何为您的服务提供可信的客户端访问

+   **OAuth 2** 访问令牌的类型

+   **OAuth 2** 授权类型的种类

+   检查 JWT 及其一般结构

+   实现资源服务器和认证服务器，用于授予客户端访问 **OAuth** **2** 资源的权利

+   实现一个 **RESTful** 客户端，通过 **OAuth 2** 授权流程访问资源

在本章中，我们有相当多的内容要介绍，但在我们深入探讨如何利用 `Spring Security` 实现 **OAuth 2** 和 **JWT** 的细节之前，我们首先想要创建一个没有 **Thymeleaf** 或任何其他基于浏览器的用户界面的日历应用程序的基础。

在移除所有 **Thymeleaf** 配置和资源后，各种控制器已转换为 **JAX-RS** **REST** 控制器。

本章代码的实际链接在这里：[`packt.link/zEHBU`](https://packt.link/zEHBU)。

重要提示

您应该从 `chapter17.00-calendar` 中的代码开始。

# 微服务是什么？

**微服务** 是一种架构方法，它允许开发物理上分离的模块化应用程序，这些应用程序是自治的，能够实现敏捷性、快速开发、持续部署和扩展。

应用程序作为一个服务集构建，如 `JSON` 或 `XML`，这允许聚合语言无关的服务。基本上，一个服务可以用最适合该服务创建任务的编程语言编写。

每个服务都在自己的进程中运行，并且位置无关，因此可以在访问网络的任何地方定位。

在下一节中，我们将探讨单体架构、微服务和面向服务的架构，并区分它们的差异。然后，我们可以深入研究使用 spring-security 的微服务安全性。

## 单体架构

微服务方法与传统单体软件方法相反，后者由紧密集成的模块组成，不经常发货，并且必须作为一个单一单元进行扩展。本书中的传统 `JBCP 日历` 应用程序是单体应用的例子。请看以下图示，它描述了单体架构：

![图 17.1 – 单体架构](img/B21757_17_1.jpg)

图 17.1 – 单体架构

虽然单体方法适合某些组织和某些应用程序，但对于需要更多灵活性和可扩展性的生态系统的公司来说，微服务正在变得流行。

## 微服务

微服务架构是一组小型离散服务，其中每个服务实现特定的业务能力。这些服务运行自己的进程，并通过**HTTP API**进行通信，通常使用**RESTful**服务方法。这些服务是为了仅服务于一个特定的业务功能而创建的，例如用户管理、管理角色、电子商务购物车、搜索引擎、社交媒体集成等。请看以下图示，它描述了微服务架构：

![图 17.2 – 微服务架构](img/B21757_17_2.jpg)

图 17.2 – 微服务架构

每个服务都可以独立于应用程序中的其他服务和企业中的其他系统进行部署、升级、扩展、重启和移除。

由于每个服务都是独立创建的，因此它们可以分别用不同的编程语言编写并使用不同的数据存储。集中的服务管理几乎不存在，这些服务使用轻量级的**HTTP**、**REST**进行相互通信。

# 面向服务的架构

你可能会问自己，“这不是和 SOA 一样吗？” 不完全一样，你可以这样说**微服务**实现了**SOA**最初承诺的东西。

**SOA** 是一种软件设计风格，其中服务通过计算机网络上的语言无关的通信协议暴露给其他组件。

**SOA** 的基本原则是独立于供应商、产品和技术。

服务的定义是一个离散的功能单元，可以远程访问并独立执行和更新，例如在线检索信用卡对账单。

虽然类似，**SOA** 和微服务仍然是不同类型的架构。

典型的**SOA**通常是在部署单体中实现的，并且更受平台驱动，而微服务可以独立部署，因此在整个维度上提供了更多的灵活性。

关键区别当然是大小；微服务一词就说明了这一点。微服务通常比常规的**SOA**服务小得多。正如 *Martin* *Fowler* 所说：

我们应该将 SOA 视为微服务的超集。

# 微服务安全

微服务可以提供很大的灵活性，但也引入了必须解决的问题。

+   **服务通信**：单体应用程序使用进程之间的内存通信，而微服务通过网络进行通信。转向网络通信不仅提出了速度问题，还提出了安全问题。

+   **紧密耦合**：微服务使用许多数据存储而不是少数几个。这为微服务与紧密耦合的服务之间隐式服务合同创造了机会。

+   **技术复杂性**：微服务可以创建额外的复杂性，这可能导致安全漏洞。如果团队没有正确经验，那么管理这些复杂性可能会迅速变得难以控制。

# OAuth 2 规范

有时存在一种误解，认为**OAuth 2**是**OAuth 1**的演变，但实际上它是一种完全不同的方法。**OAuth 1**规范要求签名，因此您必须使用加密算法来创建、生成和验证这些签名，而这些签名在**OAuth 2**中不再需要。**OAuth 2**的加密现在由**TLS**处理，这是必需的。

重要提示

**OAuth 2 RFC-6749**，*OAuth 2.0 授权框架*([`tools.ietf.org/html/rfc6749`](https://tools.ietf.org/html/rfc6749))：

**OAuth 2.0**授权框架允许第三方应用程序代表资源所有者通过在资源所有者和 HTTP 服务之间协调批准交互，或者允许第三方应用程序代表自己获取对 HTTP 服务的有限访问。

此规范取代并使**RFC 5849**中描述的**OAuth 1.0**协议过时，**OAuth 1.0**协议([`tools.ietf.org/html/rfc5849`](https://tools.ietf.org/html/rfc5849))。

为了正确理解如何利用**OAuth 2**，我们需要确定某些角色以及这些角色之间的协作。让我们定义在**OAuth 2**授权过程中参与的所有角色：

+   **资源所有者**：资源所有者是能够授予对位于资源服务器上的受保护资源访问权限的实体。

+   **授权服务器**：授权服务器是在成功验证资源所有者并获得授权后向客户端颁发访问令牌的集中式安全网关。

+   **资源服务器**：资源服务器是托管受保护资源的服务器，并且能够使用**OAuth 2**访问令牌分解和响应受保护资源请求。

+   **微服务客户端**：客户端是代表资源所有者进行受保护资源请求的应用程序，但具有其授权。

## 访问令牌

在代码示例中，`access_token`代表一个凭证，客户端可以使用它来访问**API**。我们有两种类型的访问令牌：

+   **访问令牌**：访问令牌通常具有有限的生命周期，并在将此令牌包含在每个请求的 HTTP 请求头中时，用于使客户端能够访问受保护资源。

+   **刷新令牌**：刷新令牌的生存周期更长，用于在访问令牌过期后获取新的访问令牌，但无需再次向服务器发送凭证。

## 授权类型

授权类型是客户端可以使用的方法来获取代表授予的权限的访问令牌。根据应用程序的需求，有不同的授权类型允许不同类型的访问。每种授权类型都可以支持不同的 **OAuth 2** 流，而无需担心实现的技術方面。在 **OAuth 2** 中，我们有四种主要的授权类型：

+   `access_token` 和可选的 `id_token` 以及 `refresh_token`。客户端现在可以使用此 `access_token` 代表用户调用受保护的资源。

+   直接使用 `access_token`，无需 `authorization_code`。这是因为客户端应用程序，通常是一个在浏览器中运行的 JavaScript 应用程序，其信任度低于在服务器上运行的应用程序，不能信任 `client_secret`（在授权代码授予类型中需要）。由于信任度有限，隐式授权类型不会向应用程序发送刷新令牌。

+   `access_token` 和可选的 `refresh_token`。当用户和客户端之间有高度信任，并且其他授权授予流不可用时，使用此授权类型。此授权类型消除了客户端通过交换长期生存期的 `access_token` 或 `refresh_token` 来存储用户凭证的需要。

+   使用客户端提供的凭证（客户端 ID 和客户端密钥）通过认证来获取 `access_token`。

在介绍了主要的 OAuth 2 访问令牌和授权类型之后，在下一节中，我们将深入探讨 JSON Web Tokens 规范。

# JSON Web Tokens

**JWT** 是一个开放标准，**RFC 7519** ([`tools.ietf.org/html/rfc7519`](https://tools.ietf.org/html/rfc7519))，它定义了一种紧凑且自包含的格式，用于以 **JSON** 对象的形式在各方之间安全地传输信息。由于信息是数字签名的，因此可以验证和信任。JWT 可以使用秘密（使用基于哈希的消息认证码（**HMAC**）算法）或使用公钥/私钥对（使用 **Rivest–Shamir–Adleman** （**RSA**）加密算法）进行签名。

重要注意事项

**JWT RFC-** **7519** ([`tools.ietf.org/html/rfc7519`](https://tools.ietf.org/html/rfc7519)):

**JWT** 是一种紧凑、URL 安全的表示声明的方法，用于在双方之间传输。JWT 中的声明编码为一个 **JSON** 对象，用作 **JSON Web Signature** （**JWS**）结构的有效负载或 **JSON Web Encryption** （**JWE**）结构的明文，使声明可以通过消息认证码（**MAC**）进行数字签名或完整性保护，以及/或加密。

**JWT** 用于携带与携带令牌的客户端的身份和特征（**声明**）相关的信息。JWT 是一个容器，并由服务器签名以防止客户端篡改。此令牌在认证过程中创建，并在任何处理之前由授权服务器验证。它由资源服务器使用，允许客户端向资源服务器展示代表其 *身份证* 的令牌，并允许资源服务器以无状态、安全的方式验证令牌的有效性和完整性。

## 令牌结构

JWT 的结构遵循以下三部分结构，包括标题、有效载荷和签名：

```java
[Base64Encoded(HEADER)] . [Base64Encoded (PAYLOAD)] . [encoded(SIGNATURE)]
```

### 编码后的 JWT

以下代码片段是基于客户端请求返回的完整编码的 `access_token`：

```java
eyJraWQiOiJlOTllMzEyYS0yMDJmLTRmNDItOWExNi1h ZmE2NDA5Mzg0N2QiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJqYmNwLWNhbGVuZGFyIiwiYXVkIjoiamJjcC1jYWxlbmRhciIsIm5iZiI6MT cwODU0ODUzMCwic2NvcGUiOlsiZXZlbnRzLnJlYWQiXSwiaXNzIjoiaHR0cDovL2xvY2Fs aG9zdDo5MDAwIiwiZXhwIjoxNzA4NTQ4ODMwLCJpYXQiOjE3MDg1NDg1MzAsImp0aSI6I jRhMzVjZmNmLTE5YWItNDZjZC05OWI4LWQxNWM5ZmZlNjQ1MiJ9.WNJTwQwHA4TVE1BYuizQUo88Dnf0K2by0awxVo_mSq_8n5KWkQMuKESFQwQHT32VExn7qHW6JoD6sfxrLK5q2o-KKIYDpL1CACtfjK0mUCWjfpLfpeyXg0FpYPw6s4allS3zUfOSrFf53wP8k4XCNaPxU9yVQ8s2TB064Sanl7W0VwSbxoz4B-VgPQwEob1cxhAXrBBy5WmM8rk7WsvPXYvMLdo ISpkP4n66hCzdmmFiBWFhgsfRsOVG8mNmIWgeJVgLXY BiLrbR2FuFK5KxU7Ls7IMZcWiHd95yAgA6TQ46yBiJErclNVr8Xr5M2SnzFR7HWJY 2OHCNJxnjRpbwEQ
```

### 标题

我们 `access_token` JWT 的编码标题是 `base64` 编码的，如下面的代码所示：

```java
eyJraWQiOiJlOTllMzEyYS0yMDJmLTRmNDItOWExNi1hZmE2NDA5Mzg0N2QiLCJhbGciOi JSUzI1NiJ9
```

通过解码编码后的标题，我们得到以下有效载荷：

```java
{
  "kid": "e99e312a-202f-4f42-9a16-afa64093847d",
  "alg": "RS256"
}
```

### 有效载荷

我们 `access_token` JWT 的编码有效载荷是 `base64` 编码的，如下所示：

```java
eyJzdWIiOiJqYmNwLWNhbGVuZGFyIiwiYXVkIjoia mJjcC1jYWxlbmRhciIsIm5iZiI6MTc wODU0ODUzMCwic2NvcGUiOlsiZXZlbnRzL nJlYWQiXSwiaXNzIjoiaHR0cDovL2xvY2FsaG 9zdDo5MDAwIiwiZXhwIjoxNzA4NTQ4ODMwL CJpYXQiOjE3MDg1NDg1MzAsImp0aSI6IjRhM zVjZmNmLTE5YWItNDZjZC05OWI4LWQxNWM5ZmZlNjQ1MiJ9
```

通过解码编码后的有效载荷，我们得到以下有效载荷声明：

```java
{
  "sub": "jbcp-calendar",
  "aud": "jbcp-calendar",
  "nbf": 1708548530,
  "scope": [
    "events.read"
  ],
  "iss": "http://localhost:9000",
  "exp": 1708548830,
  "iat": 1708548530,
  "jti": "4a35cfcf-19ab-46cd-99b8-d15c9ffe6452"
}
```

### 签名

我们 `access_token` 的编码有效载荷是由授权服务器使用私钥编码的，如下面的代码所示：

```java
WNJTwQwHA4TVE1BYuizQUo88Dnf0K2by0awxVo _mSq_8n5KWkQMuKESFQwQHT32VExn7qHW 6JoD6sfxrLK5q2o-KKIYDpL1CACtfjK0mUCW jfpLfpeyXg0FpYPw6s4allS3zUfOSrFf53 wP8k4XCNaPxU9yVQ8s2TB064Sanl7W0VwSbxoz4B-VgPQwEob1cxhAXrBBy5WmM8rk7Ws vPXYvMLdoISpkP4n66hCzdmmFiBWFhgsfRsOV G8mNmIWgeJVgLXYBiLrbR2FuFK5KxU7Ls 7IMZcWiHd95yAgA6TQ46yBiJErclNVr8Xr5M2SnzFR7HWJY2OHCNJxnjRpbwEQ
```

以下是为创建 JWT 签名编写的伪代码：

```java
var encodedString = base64UrlEncode(header) + "."; encodedString += base64UrlEncode(payload);
var privateKey = "[-----PRIVATE KEY
]";
var signature = SHA256withRSA(encodedString, privateKey); var JWT = encodedString + "." + base64UrlEncode(signature);
```

# Spring Security 中的 JWT 认证

接下来，让我们检查 `Spring Security` 在基于 servlet 的应用程序中实现 **JWT** 认证所使用的架构元素，类似于我们之前讨论的那个。

`JwtAuthenticationProvider` 作为 `AuthenticationProvider` 的实现，利用 `JwtDecoder` 和 `JwtAuthenticationConverter` 在认证过程中验证 **JWT**。

现在，让我们深入了解 `JwtAuthenticationProvider` 在 `Spring Security` 上下文中的工作原理。随附的图解说明了 `AuthenticationManager` 的复杂性，如图中描绘的读取 **Bearer Token** 的过程所示。

![图 17.3 – Spring Security 中的 JWT 认证](img/B21757_17_3.jpg)

图 17.3 – Spring Security 中的 JWT 认证

Spring Security 中的 JWT 认证包括以下步骤：

1.  认证过滤器是读取 `BearerTokenAuthenticationToken` 到 `AuthenticationManager` 的过程的一部分，`AuthenticationManager` 由 `ProviderManager` 实现。

1.  `ProviderManager` 已配置为使用 `JwtAuthenticationProvider` 类型的 `AuthenticationProvider`。

1.  `JwtAuthenticationProvider` 通过 `JwtDecoder` 承担解码、验证和验证 `Jwt` 的任务。

1.  随后，`JwtAuthenticationProvider` 使用 `JwtAuthentication` **转换器**将 **JWT** 转换为授权权限的集合。

1.  在认证成功后，返回的 Authentication 以 `JwtAuthenticationToken` 的形式呈现，其中包含代表 `JwtDecoder` 的主体。最终，`JwtAuthenticationToken` 将由 Authentication Filter 放置在 `SecurityContextHolder` 中。

在覆盖了 **OAuth 2** 和 **JWT** 规范之后，我们将更深入地了解它们在 spring-security 中的实现。

# Spring Security 中的 OAuth 2 支持

`Spring Security` 提供了 `Spring Framework` 编程模型和配置惯例。

在下一节中，我们将确定参与 **OAuth** **2** 流的主要组件。

## 资源所有者

资源所有者可以是一个或多个来源，在 **JBCP 日历** 的上下文中，它将具有日历应用程序作为资源所有者。**JBCP 日历**除了配置资源服务器外，不需要任何特定的配置来表示其所有权。

## 资源服务器

大多数资源服务器支持都集中在 `spring-security-oauth2-resource-server` 中。然而，`spring-security-oauth2-jose` 负责解码和验证。因此，这两个组件对于功能齐全的资源服务器，能够处理 **JWT 编码** **Bearer Tokens** 至关重要。

在 Spring Boot 中，将应用程序设置为资源服务器涉及两个基本步骤：

1.  首先，包含必要的依赖项

1.  其次，指定授权服务器位置。

## 授权服务器

要启用授权服务器功能，我们将使用提供 `Authorization` `Server` 产品实现的 `Spring Authorization Server`。

要启动 `Spring Authorization Server` 的使用，最直接的方法是将 `Spring Authorization Server` 作为依赖项构建，如下所示。

```java
//build.gradle
dependencies {
...
    // Spring Authorization Server
    implementation org.springframework.boot:spring-boot-starter-oauth2-authorization-server'
...
}
```

重要提示

您的代码现在应类似于 `chapter17.00-authorization-server`。

## OAuth 2 资源最小配置属性

注意，在项目 `chapter17.00-calendar` 中，我们已添加 `spring-boot-starter-oauth2-resource-server` 依赖项，除了 spring-security。这对于我们的应用程序作为资源服务器的行为非常重要。

```java
//build.gradle
dependencies {
...
    // Spring Authorization Server
    implementation "org.springframework.boot:spring-boot-starter-oauth2-resource-server"
...
}
```

在 Spring Boot 应用程序中，您可以通过以下步骤轻松指定要使用的授权服务器：

```java
//src/main/resources/application.yml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:9000
```

重要提示

要使 `issuer-uri` 属性生效，以下端点之一必须是授权服务器支持端点：`idp.example.com/issuer/.well-known/openid-configuration`、`idp.example.com/.well-known/openid-configuration/issuer` 或 `idp.example.com/.well-known/oauth-authorization-server/issuer`。此端点通常被称为提供者配置端点或授权服务器元数据端点。

### 创业预期

当使用此属性及其相关依赖项时，资源服务器将自动设置其配置以验证 JWT 格式编码的 Bearer Tokens。

这是通过一个可预测的启动序列完成的：

1.  询问提供者配置或授权服务器元数据端点以获取`jwks_url`属性。

1.  检查`jwks_url`端点以支持算法。

1.  配置验证策略以查询`jwks_url`以获取对应于识别算法的有效**公钥**。

1.  配置验证策略以验证每个`idp.example.com`的`"iss"`声明。

这个过程的含义是，授权服务器必须处于运行状态并且能够接收请求，以便资源服务器能够成功初始化。

### 运行时的期望

在应用程序启动后，`Resource Server`将努力处理任何包含`Authorization:` `Bearer`头的请求。

```java
GET / HTTP/1.1
Authorization: Bearer some-token-value # Resource Server will process this
```

只要指定了这个方案，资源服务器就会努力按照 Bearer Token 规范来处理请求。

对于结构良好的 JWT，资源服务器将：

1.  使用在启动时从`jwks_url`端点获得的公钥验证其签名，确保与 JWT 匹配。

1.  验证 JWT 的`exp`和`nbf`时间戳，以及 JWT 的`iss`声明。

1.  将每个范围与一个权限关联，使用前缀**SCOPE_**。

当授权服务器引入新密钥时，Spring Security 将自动更新和轮换用于验证 JWT 的密钥。

重要提示

你的代码现在应该看起来像`chapter17.01-calendar`。

默认情况下，`Authentication#getPrincipal`返回的是一个`Spring Security Jwt`对象，如果可用，`Authentication#getName`对应于 JWT 的 sub 属性。

在定义`jwk-set-uri`之后。

## 定义授权服务器的 JWK Set URI

如果`jwk-set-uri`如下所示：

```java
//src/main/resources/application.yml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:9000
          jwk-set-uri: http://localhost:9000/.well-known/jwks.json
```

因此，`issuer-uri`确保资源服务器验证传入 JWT 中的`"iss"`声明。

## 提供受众信息

如前所述，`issuer-uri`属性验证`"iss"`声明，识别发送 JWT 的实体。

此外，`audiences`属性用于验证`"aud"`声明，确定 JWT 的预期接收者。

你可以这样指定资源服务器的受众：

```java
//src/main/resources/application.yml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:9000
          audiences: https://my-org.audience
```

结果将是，如果`"aud"`声明的`"iss"`声明列表中不包含`my-resource-server.example.com`，则验证将不会成功。

## 使用 SecurityFilterChain 配置授权

`Authorization Server`通常包含一个`scope`或`scp`属性，表示授予的权限或范围。

在这种情况下，资源服务器将努力将这些范围转换为授权权限列表，并为每个范围添加`SCOPE_`字符串作为前缀。因此，为了使用从 JWT 获得的范围来保护端点或方法，相关的表达式应包含此前缀。例如：

```java
//src/main/java/com/packtpub/springsecurity/configuration/ SecurityConfig.java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
          .securityMatcher("/events/**")
          .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/events/**").hasAuthority("SCOPE_events.read"))
          .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()));
    return http.build();
}
```

重要提示

你的代码现在应该看起来像`chapter17.02-calendar`。

在这一点上，我们可以启动 `chapter17.00-authorization-server` 和 `chapter17.02-calendar`，我们将准备好发送 **OAuth** **2** 请求。

## 令牌请求

当我们发起初始令牌请求时，我们应该得到一个类似于以下的成功响应：

```java
curl -i -X POST \
  http://localhost:9000/oauth2/token \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=client_credentials&client_id=jbcp-calendar&&client_secret=secret&scope=events.read'
```

以下是示例响应：

```java
HTTP/1.1 200
Vary: Origin
Vary: Access-Control-Request-Method
Vary: Access-Control-Request-Headers
X-Content-Type-Options: nosniff
X-XSS-Protection: 0
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Wed, 21 Feb 2024 17:51:02 GMT
{
  "access_token": "eyJraWQiOiJjMzJjNmVlNy0yYTM5LTQ0NDY tOWQzZS02NzA2ZWJjMWM5MGUiLCJhbGciOiJSUzI1NiJ9.eyJzdWI iOiJqYmNwLWNhbGVuZGFyIiwiYXVkIjoiamJjcC1j YWxlbmRhciIsIm5iZiI6MTcwODUzNzg2Miwic2NvcGUiOlsiZX ZlbnRzLnJlYWQiXSwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo5MD AwIiwiZXhwIjoxNzA4NTM4MTYyLCJpYXQiOjE3MDg1Mzc4NjIsImp 0aSI6ImVkNjc1YzcwLTg4MGItNDYxYy1hMDk0LTFmMTA1ZTk3OTk0 NCJ9.OWMHZC_cRqUsshwTCIIdo6oGK_KU39hY25U5YhTUU7QTi-Sm F7wy9QdDxJnl9brIXgjq7NpIeC9zZyi l81S4p7HwFP3_3iCN1NQA54vTZ0-UBfT8q6H1aEQzeEdZUDnhoYK2c oOihbYcNH_Dfn13POMcEwBhFwIsul6tJHN_lLVFBA-CTMxSHoBWBDNq NvU-gIdadOxFPDpWV86No8DfYgDGWKLP18k3KggLC37ebMbNkIMgK24gYxM_5f_g2nR_ueiV6ZQO5fyGq960nYWzePoQtdYVcvHwkQk_FG_B75rcSrITuTTgDrcA8FWrZrOoitvEOnglHmieUguoYVG2BA",
  "scope": "events.read",
  "token_type": "Bearer",
  "expires_in": 299
}
```

具体来说，我们已经获得了一个可以在后续请求中使用的访问令牌。这个 `access_token` 将作为我们的载体。

## 端点请求

现在，我们将使用 `access_token` 并用该令牌以以下格式向服务器发起额外的请求：

```java
curl -k -i http://localhost:8080/events/  \
-H "Authorization: Bearer eyJraWQiOiJjMzJjNmVl Ny0yYTM5LTQ0NDYtOWQzZS02NzA2ZWJjMWM5MGUiLCJhbG ciOiJSUzI1NiJ9.eyJzdWIiOiJqYmNwLWNhbGVuZGFyIiwiYXVkIjoiamJjcC 1jYWxlbmRhciIsIm5iZiI6MTcwODUzNzg2Miwic2NvcGUi OlsiZXZlbnRzLnJlYWQiXSwiaXNzIjoiaHR0cDovL2xvY2 FsaG9zdDo5MDAwIiwiZXhwIjoxNzA4NTM4MTYyLCJpYXQi OjE3MDg1Mzc4NjIsImp0aSI6ImVkNjc1YzcwLTg4MGItNDY xYy1hMDk0LTFmMTA1ZTk3OTk0NCJ9.OWMHZC_cRqUsshwT CIIdo6oGK_KU39hY25U5YhTUU7QTi-SmF7wy9QdDxJnl9br IXgjq7NpIeC9zZyil81S4p7HwFP3_3iCN1NQA54vTZ0-UBf T8q6H1aEQzeEdZUDnhoYK2coOihbYcNH_Dfn13POMcEwBhF wIsul6tJHN_lLVFBA-CTMxSHoBWBDNqNvU-gIdadOxFPDpW V86No8DfYgDGWKLP18k3KggLC37ebMbNkIMgK24gYxM_5f_g2nR_ueiV6ZQO5fyGq960nYWzePoQtdYVcvHwkQk_FG_B75 rcSrITuTTgDrcA8FWrZrOoitvEOnglHmieUguoYVG2BA"
```

我们应该得到以下响应：

```java
HTTP/1.1 200
Vary: Origin
Vary: Access-Control-Request-Method
Vary: Access-Control-Request-Headers
X-Content-Type-Options: nosniff
X-XSS-Protection: 0
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
Strict-Transport-Security: max-age=31536000 ; includeSubDomains
X-Frame-Options: DENY
Content-Type: application/json
Transfer-Encoding: chunked
Date: Wed, 21 Feb 2024 17:55:32 GMT
[
  {
    "id": 100,
    "summary": "Birthday Party",
    "description": "This is going to be a great birthday",
    "dateWhen": "2023-07-03T18:30:00.000+00:00"
  },
  {
    "id": 101,
    "summary": "Conference Call",
    "description": "Call with the client",
    "dateWhen": "2023-12-23T12:00:00.000+00:00"
  },
  {
    "id": 102,
    "summary": "Vacation",
    "description": "Paragliding in Greece",
    "dateWhen": "2023-09-14T09:30:00.000+00:00"
  }
]
```

## 使用 @PreAuthorize 注解配置授权

另一种配置授权的方法是使用 `@PreAuthorize` 注解。

第一步是在 `SecurityConfig.java` 中启用方法安全：

```java
//src/main/java/com/packtpub/springsecurity/configuration/ SecurityConfig.java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {
    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
       http
             .securityMatcher("/events/**")
             .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()));
       return http.build();
    }
}
```

下一步是确保 `/events` 端点调用的 `CalendarService.getEvents()` 的安全性。

```java
//src/main/java/com/packtpub/springsecurity/service/ CalendarService.java
public interface CalendarService {
...omitted for brevity
    @PreAuthorize("hasAuthority('SCOPE_events.read')")
    List<Event> getEvents();
...
}
```

重要提示

您的代码现在应该看起来像 `chapter17.03-calendar`。

在这一点上，我们可以启动 `chapter17.00-authorization-server` 和 `chapter17.03-calendar`，我们将准备好发送 **OAuth** **2** 请求。

您可以尝试再次进行，对于 `/token` 和 `/events` 端点请求使用相同的先前步骤。

现在我们已经准备好了 `OAuth 2 服务器`，可以为客户颁发 `access_tokens`，我们现在可以创建一个微服务客户端来与我们的系统交互。

### 配置 OAuth 2 客户端

现在我们已经配置了资源服务器，您可以创建 REST 客户端来消费 **OAuth2** 受保护的资源。

1.  您可以使用 https://start.spring.io/ 通过选择以下依赖项来初始化您的项目：

```java
//build.gradle
dependencies {
...
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-oauth2-client'
...
}
```

1.  接下来，您需要按照以下配置将应用程序与客户端配置一起配置：

```java
//src/main/resources/application.yml
jbcp-calendar:
  events:
    api: http://localhost:8080/events/
spring:
## Chapter 17 Authorization Server
  security:
    oauth2:
      client:
        registration:
          calendar-client:
            client-id: jbcp-calendar
            client-secret: secret
            scope: events.read
            authorization-grant-type: client_credentials
            client-name: Calendar Client
        provider:
          calendar-client:
            token-uri: http://localhost:9000/oauth2/token
server:
  port: 8888
```

1.  在此示例中，我们将使用 `RestTemplate` 和 `ClientHttpRequestInterceptor` 来绑定我们的 REST 客户端的 `OAuth2AccessToken`。

    在 Spring Security 中向第三方 `OAuth2AuthorizedClient` 类发起请求，并通过在出站请求的授权头中插入 Bearer 令牌来获取受保护资源。

    提供的示例设置了应用程序以作为 **OAuth2** 客户端运行，能够从第三方 **API** 请求受保护的资源。

```java
//src/main/java/com/packtpub/springsecurity/config/SecurityConfig.java
@Configuration
public class SecurityConfig {
    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
       http
             .oauth2Client(withDefaults());
       return http.build();
    }
    @Bean
    public RestTemplate oauth2RestTemplate(OAuth2HttpRequestInterceptor oAuth2HttpRequestInterceptor) {
       RestTemplate restTemplate = new RestTemplate();
       List<ClientHttpRequestInterceptor> interceptors = restTemplate.getInterceptors();
       if (CollectionUtils.isEmpty(interceptors)) {
          interceptors = new ArrayList<>();
       }
       interceptors.add(oAuth2HttpRequestInterceptor);
       restTemplate.setInterceptors(interceptors);
       return restTemplate;
    }
}
```

1.  以下 `OAuth2HttpRequestInterceptor` 可以按照示例代码中的描述进行定义：

```java
//src/main/java/com/packtpub/springsecurity/config/OAuth2HttpRequestInterceptor.java
@Component
public class OAuth2HttpRequestInterceptor implements ClientHttpRequestInterceptor {
    private final OAuth2AuthorizedClientManager authorizedClientManager;
    private final ClientRegistrationRepository clientRegistrationRepository;
    public OAuth2HttpRequestInterceptor(OAuth2AuthorizedClientManager authorizedClientManager, ClientRegistrationRepository clientRegistrationRepository) {
       this.authorizedClientManager = authorizedClientManager;
       this.clientRegistrationRepository = clientRegistrationRepository;
    }
    @Override
    public ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution) throws IOException {
       ClientRegistration clientRegistration = clientRegistrationRepository.findByRegistrationId("calendar-client");
       OAuth2AuthorizeRequest oAuth2AuthorizeRequest = OAuth2AuthorizeRequest
             .withClientRegistrationId(clientRegistration.getRegistrationId())
             .principal(clientRegistration.getClientId())
             .build();
       OAuth2AuthorizedClient client = authorizedClientManager.authorize(oAuth2AuthorizeRequest);
       String accessToken = client.getAccessToken().getTokenValue();
       request.getHeaders().setBearerAuth(accessToken);
       return execution.execute(request, body);
    }
}
```

1.  最后一步是创建 `RestTemplate` Bean：

```java
//src/main/java/com/packtpub/springsecurity/web/controllers/OAuth2RestClient.java
@RestController
public class OAuth2RestClient {
    private final RestTemplate oauth2RestTemplate  ;
    public OAuth2RestClient(RestTemplate oauth2RestTemplate) {
       this.oauth2RestTemplate = oauth2RestTemplate;
    }
    @Value("${jbcp-calendar.events.api}")
    private String eventsApi;
    @GetMapping("/")
    public  String apiCheck() {
       return oauth2RestTemplate.getForObject(eventsApi, String.class);
    }
}
```

我们现在应该有一个客户端应用程序的相同代码库。

重要提示

您的代码现在应该看起来像 `chapter17.03-calendar-client`。

我们需要确保 `chapter17.03-calendar` 和 `chapter17.00-authorization-server` 应用程序正在运行，并准备好接收来自客户端的 **OAuth 2** 请求。

然后，我们可以启动 `chapter17.03-calendar-client` 应用程序，该程序将公开一个 RESTful 端点，该端点将调用我们的资源服务器以访问远程资源上的 `/events` 路径上配置的事件，并通过运行 `http://localhost:8888/` 返回以下结果：

```java
[
  {
    "id":100,
    "summary":"Birthday Party",
    "description":"This is going to be a great birthday",
    "dateWhen":"2023-07-03T18:30:00.000+00:00"
  },
  {
    "id":101,
    "summary":"Conference Call",
    "description":"Call with the client",
    "dateWhen":"2023-12-23T12:00:00.000+00:00"
  },
  {
    "id":102,
    "summary":"Vacation",
    "description":"Paragliding in Greece",
    "dateWhen":"2023-09-14T09:30:00.000+00:00"
  }
]
```

# 摘要

在本章中，你学习了单体应用和**微服务**之间的基本区别，并将**SOA**与**微服务**进行了比较。你还学习了**OAuth 2**的概念架构以及它是如何为你的服务提供可信赖的客户端访问，以及了解了**OAuth 2**访问令牌的类型和**OAuth 2**客户端**凭证**类型。

我们考察了**JWT**及其一般结构，实现了一个**资源服务器**和**授权服务器**，用于授予客户端访问**OAuth 2**资源的权限，并实现了一个**RESTful**客户端，通过**OAuth 2****授权流程**来访问资源。

我们通过演示使用 spring-security 的实用**OAuth 2**示例实现来结束本章。接下来，下一章将探讨与**中央认证服务**（**CAS**）的集成，以实现为你的 Spring Security 启用应用提供**单点登录**（**SSO**）和**单点登出**（**SLO**）功能。
