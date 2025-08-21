# 第十六章：使用 OAuth 2 和 JSON Web Tokens 的微服务安全

在本章中，我们将探讨基于微服务的架构，并查看 OAuth 2 与**JSON Web Tokens**（**JWT**）在 Spring 基础应用程序中扮演的安全角色。

以下是在本章中将要覆盖的主题列表：

+   单体应用和微服务之间的通用区别

+   比较**服务导向架构**（**SOA**）与微服务

+   OAuth 2 的概念架构及其如何为您的服务提供可信的客户端访问

+   OAuth 2 访问令牌的类型

+   OAuth 2 的授权类型

+   检查 JWT 及其一般结构

+   实现资源服务器和认证服务器，以授予客户端访问 OAuth 2 资源的权限

+   实现 RESTful 客户端以通过 OAuth 2 授权流程访问资源

我们在这章中要覆盖的内容还有很多，但在我们详细介绍如何开始利用 Spring Security 实现 OAuth 2 和 JWT 之前，我们首先想要创建一个没有 Thymeleaf 或其他基于浏览器的用户界面的日历应用程序的基本线。

在移除所有 Thymeleaf 配置和资源后，各种控制器已转换为**JAX-RS REST**控制器。

你应该从`chapter16.00-calendar`的代码开始。

# 微服务是什么？

微服务是一种允许开发物理上分离的模块化应用程序的架构方法，这些应用程序是自主的，支持敏捷性、快速开发、持续部署和扩展。

应用程序作为一组服务构建，类似于 SOA，这样服务可以通过标准 API 进行通信，例如 JSON 或 XML，这允许聚合语言不可知的服务。基本上，服务可以用最适合创建服务任务的编程语言编写。

每个服务在其自己的进程中运行，且与位置无关，因此它可以在访问网络的任何位置运行。

# 单体应用

微服务方法与传统的单体软件方法相反，后者由紧密集成的模块组成，这些模块不经常发货，必须作为一个单一单元进行扩展。本书中的传统 Java EE 应用程序和日历应用程序就是单体应用的例子。请查看以下图表，它描述了单体架构：

![](img/8a5ed567-edbf-45ad-bada-a2ee8e8d2dd0.png)

尽管单体方法对于某些组织和某些应用来说非常适合，但对于需要在其生态系统中具有更多灵活性和可伸缩性的公司来说，微服务越来越受欢迎。

# 微服务

微服务架构是一系列小型离散服务的集合，每个服务实现特定的业务功能。这些服务运行自己的进程，并通过 HTTP API 进行通信，通常使用 RESTful 服务方法。这些服务是为了只服务于一个特定的业务功能而创建的，比如用户管理、行政角色、电子商务购物车、搜索引擎、社交媒体集成等。请查看以下描述微服务架构的图表：

![](img/4d14f1d3-65aa-41c4-9de5-2117238b8db8.png)

每个**s**服务可以独立于应用程序中的其他服务和企业中的其他系统进行部署、升级、扩展、重启和移除。

因为每个服务都是独立于其他服务创建的，所以它们可以分别用不同的编程语言编写，并使用不同的数据存储。集中式服务管理实际上是不存在的，这些服务使用轻量级的 HTTP、REST 或 Thrift API 进行相互之间的通信。

**Apache Thrift** 软件框架可以从 [`thrift.apache.org`](https://thrift.apache.org/) 下载。它是一个用于开发可扩展的跨语言服务的框架，结合了软件栈和代码生成引擎，以高效、无缝地在 C++、Java、Python、PHP、Ruby、Erlang、Perl、Haskell、C#、Cocoa、JavaScript、Node.js、Smalltalk 和其他语言之间构建服务。

# 面向服务的架构

你可能会问自己，“这不是和 SOA 一样吗？” 不完全是，你可以说是微服务实现了 SOA 最初承诺的东西。

面向服务架构（SOA）是一种软件设计风格，其中服务通过计算机网络上的语言无关的通信协议暴露给其他组件。

面向服务架构（SOA）的基本原则是独立于供应商、产品和技术的。

服务的定义是一个可以远程访问、独立操作和更新的离散功能单元，例如在线获取信用卡账单。

尽管相似，但 SOA 和微服务仍然是不同类型的架构。

典型的 SOA 通常在部署单体内部实现，并且更受平台驱动，而微服务可以独立部署，因此，在所有维度上提供更多的灵活性。

当然，关键区别在于规模；单词“微”说明了一切。微服务通常比传统的 SOA 服务要小得多。正如 Martin Fowler 所说：

“我们应该将 SOA 视为微服务的超集。”

-Martin Fowler

# 微服务安全

微服务可以提供极大的灵活性，但也会引入必须解决的问题。

# 服务通信

单体应用程序使用进程间的内存通信，而微服务通过网络进行通信。向网络通信的转变不仅涉及到速度问题，还有安全性问题。

# 紧密耦合

微服务使用许多数据存储而不是几个。这创造了微服务与紧密集成的服务之间的隐式服务合同的机会。

# 技术复杂性

微服务可能会创建额外的复杂性，这可能会造成安全漏洞。如果团队没有正确的经验，那么管理这些复杂性可能会迅速变得无法管理。

# OAuth 2 规范

有时会有一种误解，认为 OAuth 2 是 OAuth 1 的演变，但它是完全不同的方法。OAuth1 规范要求签名，因此你必须使用加密算法来创建生成和验证那些在 OAuth 2 中不再需要的签名。OAuth 2 的加密现在由 TLS 处理，这是强制性的。

**OAuth 2** *RFC-6749*, *OAuth 2.0 授权框架*([`tools.ietf.org/html/rfc6749`](https://tools.ietf.org/html/rfc6749)):

*OAuth 2.0 授权框架允许第三方应用程序获取对 HTTP 服务的有限访问， either on behalf of a resource owner by orchestrating an approval interaction between the resource owner and the HTTP service, or by allowing the third-party application to obtain access on its own behalf*.

本规范取代并使*RFC 5849, The OAuth 1.0 Protocol*([`tools.ietf.org/html/rfc5849)描述的 OAuth 1.0 协议过时.*`](https://tools.ietf.org/html/rfc5849)

为了正确理解如何使用 OAuth 2，我们需要确定某些角色以及这些角色之间的协作。让我们定义参与 OAuth 2 授权过程的每个角色：

+   **资源所有者：**资源所有者是能够授权位于资源服务器上的受保护资源的实体。

+   **授权服务器：**授权服务器在成功验证资源所有者并获取授权后，向客户端发放访问令牌的一个集中的安全网关。

+   **资源服务器：**资源服务器是托管受保护资源的服务器，并能够使用 OAuth 2 访问令牌来解析和响应受保护资源请求。

+   **微服务客户端：**客户端是代表资源所有者请求受保护资源的应用程序，但需要他们的授权。

# 访问令牌

一个 OAuth 2 访问令牌，在代码示例中通常被称为`access_token`，代表一个客户端可以用来访问 API 的凭据。

# 访问令牌

访问令牌通常具有限定的生命周期，当在每次请求的 HTTP 请求头中包含此令牌时，它被用来允许客户端访问受保护的资源。

# 刷新令牌

刷新令牌具有更长的生命周期，当访问令牌过期时用来获取新的访问令牌，而无需再次向服务器发送凭据。

# 授权类型

授权类型是客户端用来获取代表授权的`access_token`的方法。根据应用程序的不同需求，有不同的授权类型允许不同类型的访问。每种授权类型都可以支持不同的 OAuth 2 流程，而无需担心实现的技术方面。

# 授权码

授权码授权类型，定义在*RFC 6749*的第*4.1*节([`tools.ietf.org/html/rfc6749`](https://tools.ietf.org/html/rfc6749))中，是一种基于重定向的流程，浏览器从授权服务器接收一个授权码，并将其发送给客户端。客户端随后与授权服务器交互，用这个授权码交换`access_token`，可选的还有`id_token`和`refresh_token`。客户端现在可以使用这个`access_token`代表用户调用受保护的资源。

# 隐式

隐式授权类型，定义在*RFC 6749*的第*4.1*节([`tools.ietf.org/html/rfc6749`](https://tools.ietf.org/html/rfc6749))中，与授权码授权类型相似，但客户端应用程序直接接收`access_token`，而无需`authorization_code`。这是因为通常在浏览器内运行、比在服务器上运行的客户端应用程序信任度较低的客户端应用程序，不能信任其拥有`client_secret`（授权码授权类型中需要）。隐式授权类型由于信任限制，不会将刷新令牌发送给应用程序。

# 密码凭证

资源所有者密码授权类型，定义在*RFC 6749*的第*4.3*节([`tools.ietf.org/html/rfc6749`](https://tools.ietf.org/html/rfc6749))中，可以直接作为授权许可来获取`access_token`，可选的还有`refresh_token`。这种许可在用户与客户端之间有高度信任，且其他授权许可流程不可用时使用。这种许可类型通过用长期有效的`access_token`或`refresh_token`交换凭据，消除了客户端存储用户凭据的需要。

# 客户端证书

客户端证书授权，定义在*RFC 6749*的第*4.4*节([`tools.ietf.org/html/rfc6749#section-4.4`](https://tools.ietf.org/html/rfc6749#section-4.4))中，适用于非交互式客户端（CLI）、守护进程或其他服务。客户端可以通过使用提供的凭据（客户端 ID 和客户端密钥）进行身份验证，直接向授权服务器请求`access_token`。

# JSON Web 令牌

JWT 是一个开放标准，*RFC 7519* ([`tools.ietf.org/html/rfc7519`](https://tools.ietf.org/html/rfc7519))，定义了一个紧凑且自包含的格式，用于在 JSON 对象的形式下安全地在各方之间传输信息。由于其是数字签名的，这些信息可以被验证和信任。JWT 可以使用秘密（使用**基于哈希的消息认证码**（**HMAC**）**算法**）或使用**Rivest-Shamir-Adleman**（**RSA**）加密算法的公钥/私钥对进行签名。

JWT *RFC- 7519* ([`tools.ietf.org/html/ rfc7519`](https://tools.ietf.org/html/%20rfc7519)):

*JSON Web Token (JWT)是一个紧凑、URL 安全的方式来表示要在两个方之间转移的主张。JWT 中的主张以 JSON 对象的形式作为 JSON Web 签名(*JWS*)结构的载荷或作为 JSON Web 加密(JWE)结构的明文，使主张可以被数字签名或完整性保护 Message Authentication Code (MAC)和/或加密.*

JWT 用于携带与持有令牌的客户端的身份和特征（声明）相关的信息。JWT 是一个容器，并且由服务器签名，以避免客户端篡改。此令牌在认证过程中创建，并在进行任何处理之前由授权服务器验证。资源服务器使用此令牌允许客户端将其“身份卡”呈现给资源服务器，并允许资源服务器以无状态、安全的方式验证令牌的有效性和完整性。

# 令牌结构

JWT 的结构遵循以下三部分结构，包括头部、载荷和签名：

```java
    [Base64Encoded(HEADER)] . [Base64Encoded (PAYLOAD)] . [encoded(SIGNATURE)]
```

# 编码 JWT

以下代码片段是基于客户端请求返回的完整编码`access_token`：

```java
     eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MDk2MTA2ODks
    InVzZXJfbmFtZSI6InVzZXIxQGV4YW1wbGUuY29tIiwiYXV0aG9yaXRpZXMiOlsi
    Uk9MRV9VU0VSIl0sImp0aSI6Ijc1NTRhZGM4LTBhMjItNDBhYS05YjQ5LTU4MTU2N
    DBhNDUzNyIsImNsaWVudF9pZCI6Im9hdXRoQ2xpZW50MSIsInNjb3BlIjpb
    Im9wZW5pZCJdfQ.iM5BqXj70ET1e5uc5UKgws1QGDv6NNZ4iVEHimsp1Pnx6WXuFwtpHQoerH_F-    
    pTkbldmYWOwLC8NBDHElLeDi1VPFCt7xuf5Wb1VHe-uwslupz3maHsgdQNGcjQwIy7_U-  
    SQr0wmjcc5Mc_1BWOq3-pJ65bFV1v2mjIo3R1TAKgIZ091WG0e8DiZ5AQase
    Yy43ofUWrJEXok7kUWDpnSezV96PDiG56kpyjF3x1VRKPOrm8CZuylC57wclk-    
    BjSdEenN_905sC0UpMNtuk9ENkVMOpa9_Redw356qLrRTYgKA-qpRFUpC-3g5
    CXhCDwDQM3jyPvYXg4ZW3cibG-yRw
```

# 头部

我们的`access_token` JWT 的编码头部是**base64**编码的，如下面的代码所示：

```java
    eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
```

通过解码编码头部，我们得到以下载荷：

```java
    {
      "alg": "RS256",
       "typ": "JWT"
    }
```

# 载荷

我们的`access_token` JWT 的编码载荷是 base64 编码的，如下所示：

```java
    eyJleHAiOjE1MDk2MTA2ODksInVzZXJfbmFtZSI6InVzZXIxQGV4YW1wbGUuY29
    tIiwiYXV0aG9yaXRpZXMiOlsiUk9MRV9VU0VSIl0sImp0aSI6Ijc1NTR
    hZGM4LTBhMjItNDBhYS05YjQ5LTU4MTU2NDBhNDUzNyIsImNsaWVudF9pZCI6I
    m9hdXRoQ2xpZW50MSIsInNjb3BlIjpbIm9wZW5pZCJdfQ
```

通过解码编码载荷，我们得到以下载荷声明：

```java
    {
      "exp": 1509610689,  
      "jti": "7554adc8-0a22-40aa-9b49-5815640a4537",
      "client_id": "oauthClient1",
      "authorities": [
         "ROLE_USER"
        ],
         "scope": [
        "openid"
       ],
      "user_name": "user1@example.com"
    }
```

# 签名

授权服务器使用私钥对我们的`access_token`进行了编码，如下面的代码所示：

```java
    iM5BqXj70ET1e5uc5UKgws1QGDv6NNZ4iVEHimsp1Pnx6WXuFwtpHQoerH_F-          
    pTkbldmYWOwLC8NBDHElLeDi1VPFCt7xuf5Wb1VHe-uwslupz3maHsgdQNGcjQwIy7_U-   
    SQr0wmjcc5Mc_1BWOq3-pJ65bFV1v2mjIo3R1TAKgIZ091WG0e8DiZ5AQaseYy43ofUWrJEXok7kUWDpn
    SezV96PDiG56kpyjF3x1VRKPOrm8CZuylC57wclk-    
    BjSdEenN_905sC0UpMNtuk9ENkVMOpa9_Redw356qLrRTYgKA-qpRFUp
    C-3g5CXhCDwDQM3jyPvYXg4ZW3cibG-yRw
```

以下是创建 JWT 签名的伪代码：

```java
    var encodedString = base64UrlEncode(header) + ".";
    encodedString += base64UrlEncode(payload);
    var privateKey = "[-----PRIVATE KEY-----]";
    var signature = SHA256withRSA(encodedString, privateKey);
    var JWT = encodedString + "." + base64UrlEncode(signature);
```

# Spring Security 中的 OAuth 2 支持

Spring Security OAuth 项目提供了使用 Spring Security 进行 OAuth 2 授权的支持，使用标准的 Spring 框架和 Spring Security 编程模型以及配置习惯。

# 资源所有者

资源所有者可以是一个或多个来源，在 JBCP 日历的上下文中，它将拥有日历应用程序作为资源所有者。JBCP 日历除了配置资源服务器外，不需要有任何特定的配置来表示其所有权。

# 资源服务器

`@EnableResourceServer`注解表示容器应用程序的意图，启用一个 Spring Security 过滤器，该过滤器通过传入的 OAuth2 令牌来验证请求：

```java
    //src/main/java/com/packtpub/springsecurity/configuration/
    OAuth2ResourceServerConfig.java

    @EnableResourceServer
    public class OAuth2ResourceServerConfig
    extends ResourceServerConfigurerAdapter {...}
```

`@EnableResourceServer`注解表示容器应用程序的意图，启用一个`OAuth2AuthenticationProcessingFilter`过滤器，该过滤器通过传入的 OAuth 2 令牌来验证请求。`OAuth2AuthenticationProcessingFilter`过滤器需要使用`@EnableWebSecurity`注解在应用程序中的某个位置启用 web 安全。`@EnableResourceServer`注解注册了一个硬编码`@Order`为`3`的自定义`WebSecurityConfigurerAdapter`类。由于 Spring Framework 的技术限制，目前无法更改这个`WebSecurityConfigurerAdapter`类的顺序。为了解决这个限制，建议不要使用其他顺序为`3`的安全适配器，否则 Spring Security 会在你设置相同顺序的一个时提出抗议：

```java
//o.s.s.OAuth 2.config.annotation.web.configuration.ResourceServerConfiguration.class

    @Configuration
    public class ResourceServerConfiguration
       extends WebSecurityConfigurerAdapter implements Ordered {
 private int order = 3;           ...
        }
```

# 授权服务器

为了启用授权服务器功能，我们在配置中包含了`@EnableAuthorizationServer`注解。添加此注解将在上下文中放入`o.s.s.OAuth 2.provider.endpoint.AuthorizationEndpoint`接口和`o.s.s.OAuth 2.provider.endpoint.TokenEndpoint`接口。开发者需要负责使用`@EnableWebSecurity`配置保护`AuthorizationEndpoint`（`/oauth/authorize`）。`TokenEndpoint`（`/oauth/token`）将基于 OAuth 2 客户端凭据自动使用 HTTP 基本身份验证进行保护：

```java
    //src/main/java/com/packtpub/springsecurity/configuration/
    OAuth2AuthorizationServerConfig.java

    @Configuration
    @EnableAuthorizationServer
    public class OAuth 2AuthorizationServerConfig {...}
```

# RSA JWT 访问令牌转换器密钥对

为了创建一个安全的 JWT 编码签名，我们将创建一个自定义 RSA `keystore`，我们将其用于创建自定义`o.s.s.OAuth 2.provider.token.storeJwtAccessTokenConverter`接口：

```java
$ keytool -genkey -alias jbcpOAuth 2client -keyalg RSA \
-storetype PKCS12 -keystore jwtConverterStore.p12 \
-storepass changeit \
-dname "CN=jwtAdmin1@example.com,OU=JBCP Calendar,O=JBCP,L=Park City,S=Utah,C=US"
```

这将创建一个名为`jwtConverterStore.p12`的`PKCS12`证书，需要将其复制到`./src/main/resources/key`目录中。

# OAuth 2 资源配置属性

我们希望通过提供`keyPair`属性来外部化配置我们的 JWT 资源，包括`keystore`、`alias`和`storePassword`，正如你在我们的`application.yml`文件中看到的，位于`src/main/resources/application.yml`：

```java
    # OAuth 2 Configuration:
    security:
    OAuth 2:
       # Resource Config:
       resource:
         jwt:
 keyPair: keystore: keys/jwtConverterStore.p12 alias: jbcpOAuth 2client storePassword: changeit
```

# OAuth 2 客户端配置属性

我们需要为客户端认证、授权和 OAuth 2 范围配置客户端详细信息，正如你在`application.yml`文件中所看到的，位于`src/main/resources/application.yml`：

```java
# OAuth 2 Configuration:
security:
OAuth 2:
   # Client Config:
   client:
     # Basic Authentication credentials for OAuth 2
 clientId: oauthClient1 clientSecret: oauthClient1Password authorizedGrantTypes: password,refresh_token scope: openid
```

# JWT 访问令牌转换器

创建 JWT 令牌的最后一步是创建一个自定义`JwtAccessTokenConverter`，它将使用生成的 RSA 证书为我们的 JWT 签名。为此，我们需要拉取我们的 keyPair 配置，并配置一个自定义`JwtAccessTokenConverter`，正如在 OAuth2AuthorizationServerConfig.java 文件中所看到的：

```java
    //src/main/java/com/packtpub/springsecurity/configuration/
    OAuth2AuthorizationServerConfig.java

    public class OAuth2AuthorizationServerConfig {
       @Value("${security.OAuth 2.resource.jwt.keyPair.keystore}")
       private String keystore;
       @Value("${security.OAuth 2.resource.jwt.keyPair.alias}")
       private String keyPairAlias;
     @Value("${security.OAuth 2.resource.jwt.keyPair.storePassword}")
       private String keyStorePass;
       @Bean
       public JwtAccessTokenConverter jwtAccessTokenConverter() {
           JwtAccessTokenConverter converter = new
           JwtAccessTokenConverter();
           KeyPair keyPair = new KeyStoreKeyFactory
           (new ClassPathResource(keystore),
           keyStorePass.toCharArray() ).getKeyPair(keyPairAlias);
           converter.setKeyPair(keyPair);
           return converter;
       }
    }
```

# 用户详情服务对象

我们将使用`CalendarUser`凭据为客户端分配一个授权的`GrantedAuthority`。为了做到这一点，我们必须要么配置我们的`CalendarUserDetailsService`类，要么通过在下面的`CalendarUserDetailsService.java`文件中指定名称`userDetailsService`来实现，正如你所看到的：

```java
    //src/main/java/com/packtpub/springsecurity/core/userdetails/
    CalendarUserDetailsService.java
 @Component("userDetailsService")    public class CalendarUserDetailsService
    implements UserDetailsService {...}
```

为我们的`@Component`注解定义自定义名称的另一个替代方案是定义一个`@Bean`声明，我们可以通过在`SecurityConfig.java`文件中使用以下条目来实现：

```java
    //src/main/java/com/packtpub/springsecurity/configuration/SecurityConfig.java

    @Bean
    public CalendarUserDetailsService userDetailsService
    (CalendarUserDao calendarUserDao) {
       return new CalendarUserDetailsService(calendarUserDao);
    }
```

# 运行 OAuth 2 服务器应用程序

此时，我们可以启动应用程序，并准备好发送 OAuth 2 请求。

此时，你的代码应该看起来像这样：`chapter16.01-calendar`。

# 服务器请求

我们可以使用命令行工具，如`cURL`或`HTTPie`，来测试应用程序，或者你也可以使用像 Postman 这样的 REST 客户端插件来向服务器发送请求。

`HTTPie`: 一个像 cURL 的面向人类的 CLI 工具，`HTTPie`（发音为 aitch-tee-tee-pie）是一个命令行 HTTP 客户端。它的目标是使与 Web 服务的 CLI 交互尽可能地人性化。它提供了一个简单的 HTTP 命令，使用简单自然的语法发送任意的 HTTP 请求，并显示彩色输出。`HTTPie`可用于测试、调试和与 HTTP 服务器进行交互（[`httpie.org`](https://httpie.org/)）。

# 令牌请求

当我们初次请求令牌时，我们应该得到一个类似于以下的成功响应：

```java
    $ http -a oauthClient1:oauthClient1Password -f POST
    localhost:8080/oauth/token     
    grant_type=password username=user1@example.com password=user1 
    HTTP/1.1 200
    Cache-Control: no-cache, no-store, max-age=0, must-revalidate
    Cache-Control: no-store
    Content-Type: application/json;charset=UTF-8
    Date: Thu, 09 Nov 2017 20:29:26 GMT
    Expires: 0
    Pragma: no-cache
    Pragma: no-cache
    Transfer-Encoding: chunked
    X-Application-Context: application:default
    X-Content-Type-Options: nosniff
    X-Frame-Options: DENY
    X-XSS-Protection: 1; mode=block 
    {
 "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MT
    AzMDI1NjYsInVzZXJfbmFtZSI6InVzZXIxQGV4YW1wbGUuY29tIiwiYXV0aG9yaXRpZ
    XMiOlsiUk9MRV9VU0VSIl0sImp0aSI6ImYzNzYzMWI4LWI0OGEtNG
    Y1MC1iNGQyLTVlNDk1NTRmYzZjZSIsImNsaWVudF9pZCI6Im9hdXRoQ
    2xpZW50MSIsInNjb3BlIjpbIm9wZW5pZCJdfQ.d5I2ZFX9ia_43eeD5X3JO6i_uF1Zw-    
    SaZ1CWbphQlYI3oCq6Xr9Yna5fvvosOZoWjb8pyo03EPVCig3mobhO6AF
    18802XOlBRx3qb0FGmHZzDoPw3naTDHlhE97ctlIFIcuJVqi34T60cvii
    uXmcE1tJ-H6-7AB04-wZl_WaucoO8-K39GvPyVabWBfSpfv0nbhh_XMNiB
    PnN8u5mqSKI9xGjYhjxXspRyy--    
    zXx50Nqj1aYzxexy8Scawrtt2F87o1IesOodoPEQGTgVVieIilplwkMLhMvJfxhyMOt
    ohR63XOGBSI4dDz58z3zOlk9P3k2Uq5FmkqwNNkduKceSw","expires_in": 43199,
    "jti": "f37631b8-b48a-4f50-b4d2-5e49554fc6ce","refresh_token":    
    "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJ1c2VyM
    UBleGFtcGxlLmNvbSIsInNjb3BlIjpbIm9wZW5pZCJdLCJhdGkiOiJmMzc2MzF
    iOC1iNDhhLTRmNTAtYjRkMi01ZTQ5NTU0ZmM2Y2UiLCJleHAiOjE1MTI4NTEzNjYs
    ImF1dGhvcml0aWVzIjpbIlJPTEVfVVNFUiJdLCJqdGkiOiJjODM2OGI4NS0xNTk5L
    TQ0NTgtODQ2Mi1iNGFhNDg1OGIzY2IiLCJjbGllbnRfaWQiOiJvYXV0aENsaWVudDEifQ.
    RZJ2GbEvcmFbZ3SVHmtFnSF_O2kv- 
    TmN56tddW2GkG0gIRr612nN5DVlfWDKorrftmmm64x8bxuV2CcFx8Rm4SSWuoYv
    j4oxMXZzANqXWLwj6Bei4z5uvuu00g6PtJvy5Twjt7GWCvEF82PBoQL-  
    bTM3RNSKmPnYPBwOGaRFTiSTdKsHCcbrg-   
    H84quRKCjXTl7Q6l8ZUxAf1eqWlOYEhRiGHtoULzdOvL1_W0OoWrQds1EN5g
    AuoTTSI3SFLnEE2MYu6cNznJFgTqmVs1hYmX1hiXUhmCq9nwYpWei-  
    bu0MaXCa9LRjDRl9E6v86vWJiBVzd9qQilwTM2KIvgiG7w", "scope": "openid",
    "token_type": "bearer"
    }
```

具体来说，我们已经获得了一个可以在后续请求中使用的访问令牌。以下是我们将用作持有者的`access_token`：

```java
 eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MTAzMDI1
    NjYsInVzZXJfbmFtZSI6InVzZXIxQGV4YW1wbGUuY29tIiwiYXV0aG9yaXRpZXM
    iOlsiUk9MRV9VU0VSIl0sImp0aSI6ImYzNzYzMWI4LWI0OGEtNGY1MC1iNGQyL
    TVlNDk1NTRmYzZjZSIsImNsaWVudF9pZCI6Im9hdXRoQ2xpZW50MSIsInNjb
    3BlIjpbIm9wZW5pZCJdfQ.d5I2ZFX9ia_43eeD5X3JO6i_uF1Zw-   
    SaZ1CWbphQlYI3oCq6Xr9Yna5fvvosOZoWjb8pyo03EPVCig3mobhO6AF18802XO
    lBRx3qb0FGmHZzDoPw3naTDHlhE97ctlIFIcuJVqi34T60cviiuXmcE1tJ-H6-7AB04-wZl_WaucoO8-   
    K39GvPyVabWBfSpfv0nbhh_XMNiBPnN8u5mqSKI9xGjYhjxXspRyy--   
    zXx50Nqj1aYzxexy8Scawrtt2F87o1IesOodoPEQGTgVVieIilplwkMLhMvJfxhyMOto
    hR63XOGBSI4dDz58z3zOlk9P3k2Uq5FmkqwNNkduKceSw
```

现在我们将使用`access_token`，并使用该令牌以以下格式初始化对服务器的额外请求：

```java
$ http localhost:8080/ "Authorization: Bearer [access_token]"
```

当我们添加第一次请求中收到的`access_token`时，我们应该得到以下请求：

```java
 $ http localhost:8080/ 'Authorization: Bearer    
    eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MTAzMD
    I1NjYsInVzZXJfbmFtZSI6InVzZXIxQGV4YW1wbGUuY29tIiwiYXV0aG9yaXRp
    ZXMiOlsiUk9MRV9VU0VSIl0sImp0aSI6ImYzNzYzMWI4LWI0OGEtNGY1MC1iNGQyLT
    VlNDk1NTRmYzZjZSIsImNsaWVudF9pZCI6Im9hdXRoQ2xpZW50MSIsInNjb3BlIjpb
    Im9wZW5pZCJdfQ.d5I2ZFX9ia_43eeD5X3JO6i_uF1Zw-    
    SaZ1CWbphQlYI3oCq6Xr9Yna5fvvosOZoWjb8pyo03EPVCig3mobhO6AF18802XOl
    BRx3qb0FGmHZzDoPw3naTDHlhE97ctlIFIcuJVqi34T60cviiuXmcE1tJ-H6-7AB04-wZl_WaucoO8-   
    K39GvPyVabWBfSpfv0nbhh_XMNiBPnN8u5mqSKI9xGjYhjxXspRyy--   
    zXx50Nqj1aYzxexy8Scawrtt2F87o1IesOodoPEQGTgVVieIilplwkMLhMvJf  
    xhyMOtohR63XOGBSI4dDz58z3zOlk9P3k2Uq5FmkqwNNkduKceSw'    HTTP/1.1 200
    Cache-Control: no-cache, no-store, max-age=0, must-revalidate
    Content-Length: 55
    Content-Type: text/plain;charset=UTF-8
    Date: Thu, 09 Nov 2017 20:44:00 GMT
    Expires: 0
    Pragma: no-cache
    X-Application-Context: application:default
    X-Content-Type-Options: nosniff
    X-Frame-Options: DENY
    X-XSS-Protection: 1; mode=block
    {'message': 'welcome to the JBCP Calendar Application'}
```

我们可以继续使用相同的`access_token`进行后续请求，例如获取当前用户的日历事件：

```java
    $ http localhost:8080/events/my 'Authorization: Bearer    
    eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1MTAzMDI1NjYsI
    nVzZXJfbmFtZSI6InVzZXIxQGV4YW1wbGUuY29tIiwiYXV0aG9yaXRpZXMiOlsiU
    k9MRV9VU0VSIl0sImp0aSI6ImYzNzYzMWI4LWI0OGEtNGY1MC1iNGQyLTVlNDk1NT
    RmYzZjZSIsImNsaWVudF9pZCI6Im9hdXRoQ2xpZW50MSIsInNjb3BlIjpbIm9wZW5pZ
    CJdfQ.d5I2ZFX9ia_43eeD5X3JO6i_uF1Zw-    
    SaZ1CWbphQlYI3oCq6Xr9Yna5fvvosOZoWjb8pyo03EPVCig3mobhO6AF18802XO
    lBRx3qb0FGmHZzDoPw3naTDHlhE97ctlIFIcuJVqi34T60cviiuXmcE1tJ-H6-7AB04-wZl_WaucoO8-   
    K39GvPyVabWBfSpfv0nbhh_XMNiBPnN8u5mqSKI9xGjYhjxXspRyy--  
    zXx50Nqj1aYzxexy8Scawrtt2F87o1IesOodoPEQGTgVVieIilplwkMLhMvJfxhyMOtohR63
    XOGBSI4dDz58z3zOlk9P3k2Uq5FmkqwNNkduKceSw'
    HTTP/1.1 200
    Cache-Control: no-cache, no-store, max-age=0, must-revalidate
    Content-Type: application/json;charset=UTF-8
    Date: Thu, 09 Nov 2017 20:57:17 GMT
    Expires: 0
    Pragma: no-cache
    Transfer-Encoding: chunked
    X-Application-Context: application:default
    X-Content-Type-Options: nosniff
    X-Frame-Options: DENY
    X-XSS-Protection: 1; mode=block
 { "currentUser": [ { "description": "This is going to be a great birthday", "id": 100, "summary": "Birthday Party", 
```

```java
 "when": 1499135400000 } ] }
```

现在我们已经准备好为客户端发放`access_tokens`的 OAuth 2 服务器，我们可以创建一个微服务客户端来与我们的系统交互。

# 微服务客户端

我们通过添加`@EnableOAuth2Client`注解使我们的新客户端应用程序作为一个 OAuth 2 客户端启动。添加`@EnableOAuth2Client`注解将允许这个应用程序从一台或多台 OAuth2 授权服务器检索和使用授权码授予。使用客户端凭据授予的客户端应用程序不需要`AccessTokenRequest`或受限于范围的`RestOperations`（对于应用程序来说，状态是全局的），但它们仍然应该使用过滤器触发`OAuth2RestOperations`在需要时获取一个令牌。使用密码授予的应用程序在使用`RestOperations`方法之前需要设置`OAuth2ProtectedResourceDetails`中的认证属性，我们稍后会进行配置。让我们来看看以下步骤，看看是如何完成的：

1.  我们需要设置一些将在以下`JavaConfig.java`文件中用于配置客户端的属性：

```java
    //src/main/java/com/packtpub/springsecurity/configuration/JavaConfig.java

    @Configuration
 @EnableOAuth 2Client    public class JavaConfig {
       @Value("${oauth.token.uri}")
       private String tokenUri;
       @Value("${oauth.resource.id}")
       private String resourceId;
       @Value("${oauth.resource.client.id}")
       private String resourceClientId;
       @Value("${oauth.resource.client.secret}")
       private String resourceClientSecret;
      @Value("${oauth.resource.user.id}")
      private String resourceUserId;
      @Value("${oauth.resource.user.password}")
      private String resourceUserPassword;
      @Autowired
      private DataSource dataSource;
     ...
    }
```

1.  除了我们需要执行 OAuth 2 RESTful 操作的几个标准属性外，我们还需要创建一个`dataSource`来保存将在初始请求时检索并在后续操作中使用的给定资源的`oauth_client_token`。现在让我们为管理`oauth_client_token`创建一个`ClientTokenServices`，如以下`JavaConfig.java`文件所示：

```java
    //src/main/java/com/packtpub/springsecurity/configuration/JavaConfig.java

    @Bean
   public ClientTokenServices clientTokenServices() {
     return new JdbcClientTokenServices(dataSource);
    }
```

1.  现在我们创建一个`OAuth2RestTemplate`，它将管理 OAuth2 通信。我们将从创建一个`ResourceOwnerPasswordResourceDetails`来持有资源连接详细信息开始，然后构建一个`OAuth2RestTemplate`作为客户端请求的`OAuth2RestOperations`使用：

```java
//src/main/java/com/packtpub/springsecurity/configuration/JavaConfig.java

@Bean
public OAuth2RestOperationsOAuth2RestOperations() {
   ResourceOwnerPasswordResourceDetails resource =
                     new ResourceOwnerPasswordResourceDetails();
   resource.setAccessTokenUri(tokenUri);
   resource.setId(resourceId);
   resource.setClientId(resourceClientId);
   resource.setClientSecret(resourceClientSecret);
   resource.setGrantType("password");
   resource.setScope(Arrays.asList("openid"));
   resource.setUsername(resourceUserId);
   resource.setPassword(resourceUserPassword);
   return new OAuth 2RestTemplate(resource);
}
```

# 配置 OAuth 2 客户端

自从我们启用了`@EnableOAuth2Client`注解并设置了一个`ResourceOwnerPasswordResourceDetails`对象后，我们需要配置用于连接资源服务器和认证服务器的属性：

```java
    //src/main/resources/application.yml

    oauth:
    url: ${OAUTH_URL:http://localhost:8080}
    token:
       uri: ${OAUTH_URL:http://localhost:8080}/oauth/token
    resource:
       id: microservice-test
       # Client BASIC Authentication for Authentication Server
       client:
         id: ${OAUTH_CLIENT_ID:oauthClient1}
         secret: ${OAUTH_CLIENT_SECRET:oauthClient1Password}
       # Resource Password Credentials
       user:
         id: ${OAUTH_USER_ID:user1@example.com}
         password: ${OAUTH_USER_PASSWORD:user1}
```

现在我们已经有了这些组件，可以开始使用`OAuth2RestOperations`对象发送请求。我们将首先创建一个`RestController`来拉取远程详细信息，并将其作为 RESTful 请求的结果显示，正如我们在`OAuth2EnabledEventsController.java`文件中所展示的那样：

```java
    //src/main/java/com/packtpub/springsecurity/web/controllers/
    OAuth2EnabledEventsController.java

    @RestController
    public class OAuth2EnabledEventsController {
       @Autowired
       private OAuth2RestOperations template;
       @Value("${base.url:http://localhost:8888}")
       private String baseUrl;
       @Value("${oauth.url:http://localhost:8080}")
       private String baseOauthUrl;
       @GetMapping("/events/my")
      public String eventsMy() {
          @SuppressWarnings("unchecked")
          String result = template.getForObject(baseOauthUrl+"/events/my",
          String.class);
          return result;
       }
    }
```

现在我们应为客户端应用拥有相同的代码库。

你的代码应看起来像`chapter16.01-calendar-client`。

我们需要确保`chapter16.01-calendar`应用正在运行，并准备好接收来自客户端的 OAuth 2 请求。然后我们可以启动`chapter16.01-calendar-client`应用，该应用将暴露几个 RESTful 端点，包括一个访问配置用户事件（位于远程资源上的`/events/my`）的端点，并通过运行`http://localhost:8888/events/my`返回以下结果：

```java
    {
    "currentUser": [
   {
     "id": 100,
     "summary": "Birthday Party",
     "description": "This is going to be a great birthday",
     "when": 1499135400000
   }
    ]
    }
```

# 摘要

在本章中，你学习了单体应用和微服务之间的通用区别，并将服务导向架构（SOA）与微服务进行了比较。你还了解了 OAuth 2 的概念性架构以及它是如何为你的服务提供可信的客户端访问的，并学习了 OAuth 2 访问令牌的类型以及 OAuth 2 授权类型的类型。

我们检查了 JWT 以及它们的通用结构，实现了一个资源服务器和认证服务器，用于向客户端授予访问 OAuth 2 资源的权限，并实现了一个 RESTful 客户端，通过 OAuth 2 授权流程来获取资源。
