# 11

# 保护 API 访问

在本章中，我们将看到如何保护上一章中介绍的边缘服务器暴露的 API 和网页的访问。我们将学习如何使用 HTTPS 保护对外部访问我们 API 的监听，以及如何使用 OAuth 2.0 和 OpenID Connect 认证和授权用户和客户端应用程序访问我们的 API。最后，我们将使用 HTTP Basic 认证来保护对发现服务器 Netflix Eureka 的访问。

本章将涵盖以下主题：

+   OAuth 2.0 和 OpenID Connect 标准简介

+   关于如何保护系统架构的一般讨论

+   使用 HTTPS 保护外部通信

+   保护对发现服务器，Netflix Eureka 的访问

+   将本地授权服务器添加到我们的系统架构中

+   使用 OAuth 2.0 和 OpenID Connect 认证和授权 API 访问

+   使用本地授权服务器进行测试

+   使用外部 OpenID Connect 提供商 Auth0 进行测试

# 技术要求

关于如何安装本书中使用的工具以及如何访问本书源代码的说明，请参阅：

+   *第二十一章*，*macOS 安装说明*

+   *第二十二章*，*使用 WSL 2 和 Ubuntu 的 Microsoft Windows 安装说明*

本章中的代码示例全部来自 `$BOOK_HOME/Chapter11` 中的源代码。

如果你想查看对本章源代码所做的更改，即查看在微服务领域中保护 API 访问所需要的内容，你可以将其与 *第十章*，*使用 Spring Cloud Gateway 在边缘服务器后面隐藏微服务* 的源代码进行比较。你可以使用你喜欢的 `diff` 工具比较两个文件夹，`$BOOK_HOME/Chapter10` 和 `$BOOK_HOME/Chapter11`。

# OAuth 2.0 和 OpenID Connect 简介

在介绍 OAuth 2.0 和 OpenID Connect 之前，让我们先明确一下我们所说的认证和授权的含义。**认证**意味着通过验证用户提供的凭据（如用户名和密码）来识别用户。**授权**是指允许经过认证的用户访问各种部分，在我们的例子中，是访问 API。

**OAuth 2.0** 是一个用于**授权委托**的开放标准，**OpenID Connect** 是 OAuth 2.0 的附加组件，它允许客户端应用程序根据授权服务器执行的认证来验证用户的身份。让我们简要地看看 OAuth 2.0 和 OpenID Connect，以获得它们目的的初步了解！

## 介绍 OAuth 2.0

OAuth 2.0 是一个广泛接受的开放标准，用于授权，它允许用户同意第三方客户端应用程序以用户的名义访问受保护资源。将代表用户执行操作的权利（例如，调用 API）授予第三方客户端应用程序，称为**授权委托**。

那么，这又意味着什么呢？

让我们先理清所使用的概念：

+   **资源所有者**：最终用户。

+   **客户端**：第三方客户端应用程序，例如，一个网络应用程序或原生移动应用程序，它希望代表最终用户调用受保护的 API。

+   **资源服务器**：暴露我们想要保护的 API 的服务器。

+   **授权服务器**：在资源所有者，即最终用户经过身份验证后，授权服务器向客户端发放令牌。用户信息和用户身份验证的管理通常在幕后委托给一个**身份提供者**（**IdP**）。

客户端在授权服务器中注册，并分配一个**客户端 ID**和一个**客户端密钥**。客户端密钥必须由客户端保护，就像密码一样。客户端还注册了一组允许的**重定向 URI**，授权服务器在用户经过身份验证后使用这些 URI 发送**授权码**和**令牌**，并将它们发送回客户端应用程序。

以下是一个示例，以供说明。假设用户访问第三方客户端应用程序，并且客户端应用程序希望调用受保护的 API 来服务用户。为了允许访问这些 API，客户端应用程序需要一种方法来告诉 API 它是在代表用户行事。为了避免用户必须与客户端应用程序共享其凭据以进行身份验证的解决方案，授权服务器会发放一个**访问令牌**，该令牌允许客户端应用程序代表用户有限地访问一组选定的 API。

这意味着用户永远不需要向客户端应用程序透露其凭据。用户还可以同意客户端应用程序代表用户访问特定的 API。访问令牌代表一组时间限制的访问权限，在 OAuth 2.0 术语中表达为**作用域**。授权服务器还可以向客户端应用程序发放一个**刷新令牌**。刷新令牌可以被客户端应用程序用来获取新的访问令牌，而无需涉及用户。

OAuth 2.0 规范定义了四种用于发放访问令牌的授权流程，如下所述：

+   **授权码授权流程**：这是最安全的，但也是最复杂的授权流程。此授权流程要求用户通过网页浏览器与授权服务器进行交互以进行身份验证并同意客户端应用程序，如下面的图示所示：

![图示描述自动生成](img/B19825_11_01.png)

图 11.1：OAuth 2.0 – 授权码授权流程

下面是这个图示中发生的事情：

1.  客户端应用程序通过将用户发送到授权服务器（在网页浏览器中）来启动授权流程。

1.  授权服务器将验证用户并请求用户的同意。

1.  授权服务器将用户重定向回客户端应用程序，并附带一个授权代码。授权服务器将使用客户端在 *步骤 1* 中指定的**重定向 URI**来确定发送授权代码的位置。由于授权代码是通过网页浏览器返回给客户端应用程序的，即在一个可能存在恶意 JavaScript 代码潜在获取授权代码的不安全环境中，因此它只能使用一次，并且只能在短时间内使用。

1.  为了将授权代码交换为访问令牌，客户端应用程序需要再次调用授权服务器。客户端应用程序必须向授权服务器出示其客户端 ID、客户端密钥以及授权代码。由于客户端密钥是敏感信息且必须受到保护，此调用必须从服务器端代码执行。

1.  授权服务器发放访问令牌并将其发送回客户端应用程序。授权服务器还可以选择性地发放并返回一个刷新令牌。

1.  使用访问令牌，客户端可以向资源服务器公开的保护 API 发送请求。

1.  资源服务器验证访问令牌，在验证成功的情况下提供服务。*步骤 6* 和 *步骤 7* 可以在访问令牌有效的情况下重复进行。当访问令牌的生命周期已过期时，客户端可以使用其刷新令牌来获取一个新的访问令牌。

+   **隐式授权流程**：这个流程也是基于网页浏览器的，但旨在用于无法保护客户端密钥的客户端应用程序，例如单页网页应用程序。网页浏览器从授权服务器获取访问令牌，而不是授权代码。由于隐式授权流程比授权代码授权流程安全性较低，客户端无法请求刷新令牌。

+   **资源所有者密码凭证授权流程**：如果客户端应用程序无法与网页浏览器交互，它可以回退到这个授权流程。在这个授权流程中，用户必须与客户端应用程序共享其凭证，客户端应用程序将使用这些凭证来获取访问令牌。

+   **客户端凭证授权流程**：在客户端应用程序需要调用与特定用户无关的 API 的情况下，它可以使用此授权流程，通过其自己的客户端 ID 和客户端密钥来获取访问令牌。

完整规范可在此处找到：[`tools.ietf.org/html/rfc6749`](https://tools.ietf.org/html/rfc6749)。还有许多其他规范详细说明了 OAuth 2.0 的各个方面；欲了解概述，请参阅[`www.oauth.com/oauth2-servers/map-oauth-2-0-specs/`](https://www.oauth.com/oauth2-servers/map-oauth-2-0-specs/)。一个值得额外关注的规定是*RFC 7636 – OAuth 公共客户端的代码交换证明密钥*（简称**PKCE**），[`tools.ietf.org/html/rfc7636`](https://tools.ietf.org/html/rfc7636)。

本规范描述了如何通过添加额外的安全层，以安全的方式利用授权代码授权流程，使原本不安全的公共客户端，如移动原生应用或桌面应用程序，能够安全地使用该授权流程。

OAuth 2.0 规范于 2012 年发布，多年来，人们从 OAuth 2.0 的使用中吸取了许多经验教训。2019 年，开始建立 OAuth 2.1，整合了 OAuth 2.0 使用中的所有最佳实践和经验。草稿版本可在此处找到：[`tools.ietf.org/html/draft-ietf-oauth-v2-1-08`](https://tools.ietf.org/html/draft-ietf-oauth-v2-1-08)。

在我看来，OAuth 2.1 最重要的改进是：

+   PKCE 已集成到授权代码授权流程中。如上所述，公共客户端将需要使用 PKCE 来提高其安全性。对于授权服务器可以验证其凭证的机密客户端，PKCE 的使用不是必需的，但建议使用。

+   由于其安全性较低，隐式授权流程已被弃用，并从规范中省略。

+   资源所有者密码凭证授权流程也被弃用，并从规范中省略，原因相同。

鉴于即将发布的 OAuth 2.1 规范的方向，我们将在本书中仅使用授权代码授权流程和客户端凭证授权流程。

当涉及到对由 OAuth 2.0 保护的 API 进行自动化测试时，客户端凭证授权流程非常方便，因为它不需要使用网络浏览器进行手动交互。我们将在本章后面的测试脚本中使用此授权流程；请参阅*测试脚本中的更改*部分。

## 介绍 OpenID Connect

OpenID Connect（缩写为**OIDC**），正如之前提到的，是 OAuth 2.0 的一个附加组件，它使客户端应用程序能够验证用户的身份。OIDC 增加了一个额外的令牌，即 ID 令牌，客户端应用程序在完成授权流程后从授权服务器获取该令牌。

ID 令牌被编码为**JSON Web Token**（**JWT**），并包含多个声明，例如用户的 ID 和电子邮件地址。ID 令牌使用 JSON Web 签名进行数字签名。这使得客户端应用程序可以通过使用授权服务器的公钥验证其数字签名来信任 ID 令牌中的信息。

可选地，访问令牌也可以像 ID 令牌一样进行编码和签名，但根据规范，这不是强制的。同样重要的是，OIDC 定义了一个**发现端点**，这是一种标准化的方式来建立到重要端点的 URL，例如请求授权代码和令牌或获取公钥以验证数字签名的 JWT。最后，它还定义了一个**用户信息端点**，可以使用该端点获取有关给定用户的访问令牌的额外信息。

有关可用规范的概述，请参阅[`openid.net/developers/specs/`](https://openid.net/developers/specs/)。

在本书中，我们只使用符合 OpenID Connect 规范的授权服务器。这将通过使用它们的发现端点简化资源服务器的配置。我们还将使用对数字签名 JWT 访问令牌的可选支持来简化资源服务器验证访问令牌真实性的方式。请参阅下文中的*边缘服务器和产品组合服务的变化*部分。

这就结束了我们对 OAuth 2.0 和 OpenID Connect 标准的介绍。在本章的稍后部分，我们将了解如何使用这些标准。在下一节中，我们将从高层次了解系统景观将如何得到保护。

# 保护系统景观

为了确保本章引言中描述的系统景观安全，我们将执行以下步骤：

1.  使用 HTTPS 加密对我们外部 API 的外部请求和响应，以防止窃听。

1.  使用 OAuth 2.0 和 OpenID Connect 对访问我们的 API 的用户和客户端应用程序进行身份验证和授权。

1.  使用 HTTP 基本身份验证保护对发现服务器 Netflix Eureka 的访问。

我们将只为与边缘服务器的外部通信应用 HTTPS，而在系统景观内部通信时使用纯 HTTP。

在本书稍后出现的关于服务网格的章节（第十八章，*使用服务网格提高可观察性和管理*）中，我们将看到如何从服务网格产品中获得帮助，以自动配置 HTTPS 来保护系统景观内的通信。

为了测试目的，我们将在系统景观中添加一个本地的 OAuth 2.0 授权服务器。所有与授权服务器的外部通信将通过边缘服务器路由。边缘服务器和`product-composite`服务将作为 OAuth 2.0 资源服务器；也就是说，它们将需要有效的 OAuth 2.0 访问令牌才能允许访问。

为了最小化验证访问令牌的开销，我们假设它们被编码为签名 JWT，并且授权服务器公开了一个端点，资源服务器可以使用该端点来访问公钥，也称为**JSON Web Key Set**，或简称**jwk-set**，这是验证签名所必需的。

系统景观将如下所示：

![图描述自动生成](img/B19825_11_02.png)

图 11.2：将授权服务器添加到系统景观

从前面的图中，我们可以注意到：

+   HTTPS 用于外部通信，而纯文本 HTTP 用于系统景观内部。

+   本地 OAuth 2.0 授权服务器将通过边缘服务器从外部访问。

+   边缘服务器和`product-composite`微服务都将验证访问令牌作为已签名的 JWT。

+   边缘服务器和`product-composite`微服务将从其`jwk-set`端点获取授权服务器的公钥，并使用它们来验证基于 JWT 的访问令牌的签名。

注意，我们将专注于通过 HTTP 保护对 API 的访问，而不是涵盖一般性的最佳实践，例如，管理由**OWASP Top Ten 项目**指出的 Web 应用程序安全风险。有关更多信息，请参阅[`owasp.org/www-project-top-ten/`](https://owasp.org/www-project-top-ten/)。

在了解了系统景观如何被保护之后，让我们看看我们如何可以使用 HTTPS 保护外部通信免受窃听。

# 使用 HTTPS 保护外部通信

在本节中，我们将学习如何防止外部通信（例如，从互联网，通过边缘服务器公开的公共 API）被窃听。我们将使用 HTTPS 来加密通信。要使用 HTTPS，我们需要执行以下操作：

+   **创建证书**：我们将创建自己的自签名证书，这对于开发目的来说是足够的。

+   **配置边缘服务器**：它必须配置为仅接受使用证书的基于 HTTPS 的外部流量。

使用以下命令创建自签名证书：

```java
keytool -genkeypair -alias localhost -keyalg RSA -keysize 2048 -storetype PKCS12 -keystore edge.p12 -validity 3650 
```

源代码附带一个示例证书文件，因此您不需要运行此命令来运行以下示例。

命令将要求输入多个参数。当要求输入密码时，我输入了`password`。对于其余的参数，我简单地输入了一个空值以接受默认值。创建的证书文件`edge.p12`被放置在`gateway`项目的`src/main/resources/keystore`文件夹中。这意味着证书文件将在构建时放置在`.jar`文件中，并在运行时在`keystore/edge.p12`的类路径上可用。

在开发期间，使用类路径提供证书是足够的，但不适用于其他环境，例如，生产环境。下面将说明如何在运行时使用外部证书替换此证书！

要配置边缘服务器使用证书和 HTTPS，以下内容被添加到`gateway`项目的`application.yml`中：

```java
server.port: 8443
server.ssl:
 key-store-type: PKCS12
 key-store: classpath:keystore/edge.p12
 key-store-password: password
 key-alias: localhost 
```

以下是从前面的源代码中的一些注意事项：

+   证书的路径由`server.ssl.key-store`参数指定，并设置为`classpath:keystore/edge.p12`。这意味着证书将从`keystore/edge.p12`的位置在类路径中检索。

+   证书的密码指定在`server.ssl.key-store-password`参数中。

+   为了表明边缘服务器使用 HTTPS 而不是 HTTP，我们还将在`server.port`参数中将端口从`8080`更改为`8443`。

除了在边缘服务器中的这些更改外，还需要在以下文件中进行更改，以反映端口和 HTTP 协议的更改，将`HTTP`替换为`HTTPS`，将`8080`替换为`8443`：

+   三个 Docker Compose 文件，`docker-compose*.yml`

+   测试脚本，`test-em-all.bash`

如前所述，使用类路径提供证书仅适用于开发阶段。让我们看看我们如何在运行时用外部证书替换此证书。

## 在运行时替换自签名证书

在`.jar`文件中放置自签名证书仅对开发有用。对于运行时环境中的工作解决方案，例如测试或生产，必须能够使用由授权**CA**（简称**证书颁发机构**）签名的证书。

必须也能够在运行时指定要使用的证书，而无需重新构建`.jar`文件，当使用 Docker 时，包含`.jar`文件的 Docker 镜像。当使用 Docker Compose 管理 Docker 容器时，我们可以将 Docker 容器中的卷映射到 Docker 主机上的证书。我们还可以为 Docker 容器设置环境变量，指向 Docker 卷中的外部证书。

在*第十五章*，*Kubernetes 简介*中，我们将学习 Kubernetes，我们将看到如何处理秘密，如证书，这些秘密适合在集群中运行 Docker 容器；也就是说，容器是在一组 Docker 主机上而不是在单个 Docker 主机上调度。

本主题中描述的更改尚未应用于书中 GitHub 仓库的源代码；您需要自行进行更改才能看到其效果！

要替换打包在`.jar`文件中的证书，请执行以下步骤：

1.  创建第二个证书，并在被要求时将其密码设置为`testtest`：

    ```java
    cd $BOOK_HOME/Chapter11
    mkdir keystore
    keytool -genkeypair -alias localhost -keyalg RSA -keysize 2048 -storetype PKCS12 -keystore keystore/edge-test.p12 -validity 3650 
    ```

1.  更新 Docker Compose 文件，`docker-compose.yml`，包含位置环境变量、新证书的密码以及映射到放置新证书的文件夹的卷。更改后边缘服务器的配置如下：

    ```java
    gateway:
      environment:
        - SPRING_PROFILES_ACTIVE=docker
        - SERVER_SSL_KEY_STORE=file:/keystore/edge-test.p12
        - SERVER_SSL_KEY_STORE_PASSWORD=testtest
      volumes:
        - $PWD/keystore:/keystore
      build: spring-cloud/gateway
      mem_limit: 512m
      ports:
        - "8443:8443" 
    ```

1.  如果边缘服务器正在运行，则需要使用以下命令重新启动：

    ```java
    docker-compose up -d --scale gateway=0
    docker-compose up -d --scale gateway=1 
    ```

`docker-compose restart gateway`命令可能看起来是重启网关服务的良好候选，但实际上它没有考虑`docker-compose.yml`中的更改。因此，在这种情况下，它不是一个有用的命令。

1.  新的证书现在正在使用中！

这部分关于如何使用 HTTPS 保护外部通信的内容到此结束。在下一部分，我们将学习如何使用 HTTP 基本认证来保护发现服务器，Netflix Eureka 的访问。

# 保护发现服务器访问

之前，我们学习了如何使用 HTTPS 保护外部通信。现在我们将使用 HTTP 基本认证来限制对发现服务器 Netflix Eureka 上的 API 和网页的访问。这意味着我们需要用户提供一个用户名和密码来获取访问权限。需要在 Eureka 服务器和 Eureka 客户端上进行更改，具体如下。

## Eureka 服务器的更改

为了保护 Eureka 服务器，以下更改已应用于源代码：

1.  在`build.gradle`中添加了对 Spring Security 的依赖项：

    ```java
    implementation 'org.springframework.boot:spring-boot-starter-security' 
    ```

1.  已将安全配置添加到`SecurityConfig`类中：

    1.  用户定义如下：

        ```java
        @Bean
        public InMemoryUserDetailsManager userDetailsService() {
          UserDetails user = User.withDefaultPasswordEncoder()
              .username(username)
              .password(password)
              .roles("USER")
              .build();
          return new InMemoryUserDetailsManager(user);
        } 
        ```

    1.  `username`和`password`从配置文件注入到构造函数中：

        ```java
        @Autowired
        public SecurityConfig(
          @Value("${app.eureka-username}") String username,
          @Value("${app.eureka-password}") String password
        ) {
          this.username = username;
          this.password = password;
        } 
        ```

    1.  所有 API 和网页都通过以下定义使用 HTTP 基本认证进行保护：

        ```java
        @Bean
        public SecurityFilterChain configure(HttpSecurity http) throws Exception {
          http
            // Disable CRCF to allow services to register themselves with Eureka
            .csrf()
              .disable()
            .authorizeRequests()
              .anyRequest().authenticated()
              .and()
              .httpBasic();
          return http.build();
        } 
        ```

1.  用户凭证在配置文件`application.yml`中设置：

    ```java
    app:
     eureka-username: u
     eureka-password: p 
    ```

1.  最后，测试类`EurekaServerApplicationTests`在测试 Eureka 服务器的 API 时使用配置文件中的凭证：

    ```java
    @Value("${app.eureka-username}")
    private String username;

    @Value("${app.eureka-password}")
    private String password;

    @Autowired
    public void setTestRestTemplate(TestRestTemplate testRestTemplate) {
       this.testRestTemplate = testRestTemplate.withBasicAuth(username, password);
    } 
    ```

上述是限制对发现服务器 Netflix Eureka 的 API 和网页访问所需的步骤。现在它将使用 HTTP 基本认证，并要求用户提供用户名和密码来获取访问权限。最后一步是配置 Netflix Eureka 客户端，以便在访问 Netflix Eureka 服务器时传递凭证。

## Eureka 客户端的更改

对于 Eureka 客户端，凭证可以指定在 Eureka 服务器的连接 URL 中。这在每个客户端的配置文件`application.yml`中指定，如下所示：

```java
app:
  eureka-username: u
  eureka-password: p

eureka:
  client:
     serviceUrl:
       defaultZone: "http://${app.eureka-username}:${app.eureka-
                     password}@${app.eureka-server}:8761/eureka/" 
```

这部分关于如何限制对 Netflix Eureka 服务器访问的内容到此结束。在*测试受保护的发现服务器*部分，我们将运行测试以验证访问是否受到保护。在下一部分，我们将学习如何将本地授权服务器添加到系统架构中。

# 添加本地授权服务器

为了能够使用 OAuth 2.0 和 OpenID Connect 安全使用的 API 在本地和完全自动化地运行测试，我们将添加一个符合这些规范的授权服务器到我们的系统景观中。历史上，Spring Security 没有提供开箱即用的授权服务器。但在 2020 年 4 月，由 Spring Security 团队领导的一个社区驱动项目，**Spring Authorization Server**，宣布旨在提供授权服务器。到 2021 年 8 月，Spring Authorization Server 项目从实验状态移出，成为 Spring 项目组合的一部分。

更多信息，请参阅[`spring.io/blog/2020/04/15/announcing-the-spring-authorization-server`](https://spring.io/blog/2020/04/15/announcing-the-spring-authorization-server)和[`spring.io/blog/2021/08/17/spring-authorization-server-officially-moves-to-spring-projects`](https://spring.io/blog/2021/08/17/spring-authorization-server-officially-moves-to-spring-projects)。

Spring Authorization Server 支持使用 OpenID Connect 发现端点和访问令牌的数字签名。它还提供了一个端点，可以使用发现信息访问以获取验证令牌数字签名的密钥。有了这些功能的支持，它可以作为本地和自动测试中的授权服务器使用，以验证系统景观按预期工作。

本书中的授权服务器基于 Spring Authorization Server 项目提供的示例授权服务器；请参阅[`github.com/spring-projects/spring-authorization-server/tree/main/samples/default-authorizationserver`](https://github.com/spring-projects/spring-authorization-server/tree/main/samples/default-authorizationserver)。

对示例项目已应用以下更改：

+   构建文件已更新，以遵循本书中其他项目的构建文件结构。

+   端口已设置为`9999`。

+   已添加与本书中其他项目相同的结构的 Dockerfile。

+   授权服务器已与 Eureka 集成，以实现与本书中其他项目相同的服务发现方式。

+   已添加对 actuator 端点的公共访问。

**警告**：正如在*第七章*，*开发响应式微服务*中已警告的那样，允许公共访问 actuator 的端点在开发期间非常有帮助，但在生产系统中透露过多信息可能会成为安全问题。因此，计划最小化生产中 actuator 端点暴露的信息！

+   已添加单元测试，以验证根据 OpenID Connect 规范对最关键端点的访问。

+   单个注册用户的用户名和密码分别设置为`u`和`p`。

+   已注册两个 OAuth 客户端，`reader` 和 `writer`。`reader` 客户端被授予 `product:read` 范围，而 `writer` 客户端被授予 `product:read` 和 `product:write` 范围。客户端被配置为将它们的客户端密钥设置为 `secret-reader` 和 `secret-writer`。

+   客户端的允许重定向 URI 设置为 `https://my.redirect.uri` 和 `https://localhost:8443/webjars/swagger-ui/oauth2-redirect.html`。下面的测试中将使用第一个 URI，Swagger UI 组件将使用第二个 URI。

+   默认情况下，出于安全原因，授权服务器不允许以 `https://localhost` 开头的重定向 URI。

授权服务器已被定制以接受 `https://localhost` 用于开发和测试目的。所应用的定制在此处描述：[`docs.spring.io/spring-authorization-server/docs/1.0.0/reference/html/protocol-endpoints.html#oauth2-authorization-endpoint-customizing-authorization-request-validation`](https://docs.spring.io/spring-authorization-server/docs/1.0.0/reference/html/protocol-endpoints.html#oauth2-authorization-endpoint-customizing-authorization-request-validation)。

授权服务器的源代码可在 `$BOOK_HOME/Chapter11/spring-cloud/authorization-server` 中找到。

要将授权服务器纳入系统架构，以下文件已应用了更改：

+   服务器已被添加到通用构建文件中，`settings.gradle`。

+   服务器已被添加到三个 Docker Compose 文件中，`docker-compose*.yml`。

+   边缘服务器，`spring-cloud/gateway`。

+   在 `HealthCheckConfiguration` 中为授权服务器添加了健康检查。

+   在配置文件 `application.yml` 中已添加指向以 `/oauth`、`/login` 和 `/error` 开头的 URI 的授权服务器路由。这些 URI 用于向客户端颁发令牌、验证用户和显示错误消息。

+   由于这三个 URI 需要由边缘服务器进行保护，因此它们被配置在新类 `SecurityConfig` 中以允许所有请求。

在理解了如何将本地授权服务器添加到系统架构之后，让我们继续看看如何使用 OAuth 2.0 和 OpenID Connect 来验证和授权对 API 的访问。

# 使用 OAuth 2.0 和 OpenID Connect 保护 API

在设置好授权服务器之后，我们可以增强边缘服务器和 `product-composite` 服务以成为 OAuth 2.0 资源服务器，这样它们将需要有效的访问令牌才能允许访问。边缘服务器将被配置为接受授权服务器提供的数字签名验证的任何访问令牌。`product-composite` 服务也将需要访问令牌包含有效的 OAuth 2.0 范围：

+   `product:read` 范围将用于访问只读 API。

+   `product:write` 范围将用于访问创建和删除 API。

`product-composite` 服务也将通过一个配置进行增强，允许其 Swagger UI 组件与授权服务器交互以颁发访问令牌。这将允许 Swagger UI 网页的用户测试受保护的 API。

我们还需要增强测试脚本 `test-em-all.bash`，以便在执行测试时获取访问令牌并使用它们。

## 边缘服务器和产品组合服务的更改

在源代码中，对边缘服务器和 `product-composite` 服务都应用了以下更改：

+   Spring Security 依赖项已添加到 `build.gradle` 文件中，以支持 OAuth 2.0 资源服务器：

    ```java
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.security:spring-security-oauth2-resource-server'
    implementation 'org.springframework.security:spring-security-oauth2-jose' 
    ```

+   在两个项目中都添加了新的 `SecurityConfig` 类的安全配置：

    ```java
    @Configuration
    @EnableWebFluxSecurity
    public class SecurityConfig {

      @Bean
      SecurityWebFilterChain springSecurityFilterChain(
          ServerHttpSecurity http) {
        http
          .authorizeExchange()
            .pathMatchers("/actuator/**").permitAll()
            .anyExchange().authenticated()
            .and()
          .oauth2ResourceServer()
            .jwt();
        return http.build();
      }
    } 
    ```

以下是对前面源代码的解释：

+   `@EnableWebFluxSecurity` 注解启用了基于 Spring WebFlux 的 API 的 Spring Security 支持。

+   `.pathMatchers("/actuator/**").permitAll()` 用于允许对应该不受保护的 URL 的无限制访问，例如，在本例中的 `actuator` 端点。请参阅源代码以了解被视为不受保护的 URL。小心选择哪些 URL 不受保护。例如，在投入生产之前，应保护 `actuator` 端点。

+   `.anyExchange().authenticated()` 确保用户在允许访问所有其他 URL 之前已进行身份验证。

+   `.oauth2ResourceServer().jwt()` 指定授权将基于编码为 JWT 的 OAuth 2.0 访问令牌。

+   授权服务器的 OIDC 发现端点已在配置文件 `application.yml` 中注册：

    ```java
    app.auth-server: localhost
    spring.security.oauth2.resourceserver.jwt.issuer-uri: http://${app.auth-server}:9999
    ---
    spring.config.activate.on-profile: docker
    app.auth-server: auth-server 
    ```

L

在本章的后面部分，当系统景观启动时，您可以测试发现端点。例如，您可以使用以下命令找到返回用于验证令牌数字签名的密钥的端点：

```java
docker-compose exec auth-server curl localhost:9999/.well-known/openid-configuration -s | jq -r .jwks_uri 
```

我们还需要对仅适用于 `product-composite` 服务的某些更改。

## 仅在产品组合服务中进行的更改

除了上一节中应用的常见更改外，以下更改也已应用于 `product-composite` 服务：

+   `SecurityConfig` 类中的安全配置已通过要求访问令牌中的 OAuth 2.0 范围来细化，以便允许访问：

    ```java
    .pathMatchers(POST, "/product-composite/**")
      .hasAuthority("SCOPE_product:write")
    .pathMatchers(DELETE, "/product-composite/**")
      .hasAuthority("SCOPE_product:write")
    .pathMatchers(GET, "/product-composite/**")
      .hasAuthority("SCOPE_product:read") 
    ```

按照惯例，当使用 Spring Security 检查权限时，OAuth 2.0 范围需要以 `SCOPE_` 前缀开头。

+   已添加了一个方法 `logAuthorizationInfo()`，用于在每次调用 API 时记录 JWT 编码的访问令牌的相关部分。可以使用标准的 Spring Security `SecurityContext` 获取访问令牌，在反应式环境中，可以使用静态辅助方法 `ReactiveSecurityContextHolder.getContext()` 获取。有关详细信息，请参阅 `ProductCompositeServiceImpl` 类。

+   在运行基于 Spring 的集成测试时，已禁用 OAuth 的使用。为了防止在运行集成测试时 OAuth 机制启动，我们按以下方式禁用它：

    +   添加了一个安全配置`TestSecurityConfig`，用于测试期间使用。它允许访问所有资源：

        ```java
        http.csrf().disable().authorizeExchange().anyExchange().permitAll(); 
        ```

    +   在每个 Spring 集成测试类中，我们配置`TestSecurityConfig`以覆盖现有的安全配置，如下所示：

        ```java
        @SpringBootTest( 
          classes = {TestSecurityConfig.class},
          properties = {"spring.main.allow-bean-definition-
            overriding=true"}) 
        ```

### 允许 Swagger UI 获取访问令牌的更改

为了允许从 Swagger UI 组件访问受保护的 API，已在`product-composite`服务中应用以下更改：

+   由 Swagger UI 组件暴露的网页已被配置为公开可用。以下行已添加到`SecurityConfig`类中：

    ```java
    .pathMatchers("/openapi/**").permitAll()
    .pathMatchers("/webjars/**").permitAll() 
    ```

+   API 的 OpenAPI 规范已被增强，要求应用安全模式`security_auth`。

以下行已添加到`API`项目中接口`ProductCompositeService`的定义中：

```java
@SecurityRequirement(name = "security_auth") 
```

+   为了定义安全模式`security_auth`的语义，已在`product-composite`项目中添加了`OpenApiConfig`类。它看起来像这样：

    ```java
    @SecurityScheme(
      name = "security_auth", type = SecuritySchemeType.OAUTH2,
      flows = @OAuthFlows(
        authorizationCode = @OAuthFlow(
          authorizationUrl = "${springdoc.oAuthFlow.
            authorizationUrl}",
          tokenUrl = "${springdoc.oAuthFlow.tokenUrl}", 
          scopes = {
            @OAuthScope(name = "product:read", description =
              "read scope"),
            @OAuthScope(name = "product:write", description = 
              "write scope")
          }
    )))
    public class OpenApiConfig {} 
    ```

从前面的类定义中，我们可以看到：

1.  安全模式将基于 OAuth 2.0。

1.  将使用授权代码授予流程。

1.  获取授权代码和访问令牌所需的 URL 将由配置通过参数`springdoc.oAuthFlow.authorizationUrl`和`springdoc.oAuthFlow.tokenUrl`提供。

1.  Swagger UI 将需要的一组作用域（`product:read`和`product:write`），以便能够调用 API。

+   最后，一些配置被添加到`application.yml`中：

    ```java
     swagger-ui:
        oauth2-redirect-url: /swagger-ui/oauth2-redirect.html
        oauth:
          clientId: writer
          clientSecret: secret-writer
          useBasicAuthenticationWithAccessCodeGrant: true
      oAuthFlow:
        authorizationUrl: https://localhost:8443/oauth2/authorize
        tokenUrl: https://localhost:8443/oauth2/token 
    ```

从前面的配置中，我们可以看到：

1.  Swagger UI 将使用该 URL 来获取授权代码的重定向 URL。

1.  其客户端 ID 和客户端密钥。

1.  它将使用 HTTP 基本认证来识别授权服务器。

1.  上文所述的`OpenApiConfig`类使用的`authorizationUrl`和`tokenUrl`参数的值。请注意，这些 URL 是由网络浏览器使用的，而不是由`product-composite`服务本身使用的。因此，它们必须可以从网络浏览器解析。

为了允许对 Swagger UI 网页的无保护访问，边缘服务器也已配置为允许对路由到 Swagger UI 组件的 URL 进行无限制访问。以下行被添加到边缘服务器的`SecurityConfig`类中：

```java
.pathMatchers("/openapi/**").permitAll()
.pathMatchers("/webjars/**").permitAll() 
```

在这些更改到位后，边缘服务器和`product-composite`服务都可以作为 OAuth 2.0 资源服务器，Swagger UI 组件可以作为 OAuth 客户端。引入 OAuth 2.0 和 OpenID Connect 使用的最后一步是更新测试脚本，以便在运行测试时获取访问令牌并使用它们。

## 测试脚本的更改

首先，我们需要在调用任何 API（除了健康 API）之前获取一个访问令牌。这就像之前提到的那样，使用 OAuth 2.0 客户端凭据流来完成。为了能够调用创建和删除 API，我们以`writer`客户端的身份获取一个访问令牌，如下所示：

```java
ACCESS_TOKEN=$(curl -k https://writer:secret-writer@$HOST:$PORT/oauth2/token -d grant_type=client_credentials -d scope="product:read product:write" -s | jq .access_token -r) 
```

从前面的命令中，我们可以看到它使用了 HTTP 基本认证，在主机名之前传递其客户端 ID 和客户端密钥作为`writer:secret-writer@`。

为了验证基于范围的授权是否工作，测试脚本中增加了两个测试：

```java
# Verify that a request without access token fails on 401, Unauthorized
assertCurl 401 "curl -k https://$HOST:$PORT/product-composite/$PROD_ID_REVS_RECS -s"
# Verify that the reader client with only read scope can call the read API but not delete API
READER_ACCESS_TOKEN=$(curl -k https://reader:secret-reader@$HOST:$PORT/oauth2/token -d grant_type=client_credentials -d scope="product:read" -s | jq .access_token -r)
READER_AUTH="-H \"Authorization: Bearer $READER_ACCESS_TOKEN\""
assertCurl 200 "curl -k https://$HOST:$PORT/product-composite/$PROD_ID_REVS_RECS $READER_AUTH -s"
assertCurl 403 "curl -k https://$HOST:$PORT/product-composite/$PROD_ID_REVS_RECS $READER_AUTH -X DELETE -s" 
```

测试脚本使用读取器客户端的凭据来获取访问令牌：

+   第一个测试在未提供访问令牌的情况下调用 API。API 预期将返回`401 Unauthorized` HTTP 状态。

+   第二个测试验证读取器客户端可以调用只读 API。

+   最后一个测试调用了一个使用`reader`客户端的更新 API，该客户端仅被授予`read`权限。向删除 API 发送的请求预期将返回`403 Forbidden` HTTP 状态。

对于完整的源代码，请参阅`test-em-all.bash`。

在测试脚本更新为获取和使用 OAuth 2.0 访问令牌后，我们就可以在下一节尝试它了！

# 使用本地授权服务器进行测试

在本节中，我们将尝试测试受保护的系统环境；也就是说，我们将一起测试所有安全组件。我们将使用本地授权服务器来颁发访问令牌。以下将执行以下测试：

1.  首先，我们从源代码构建并运行测试脚本，以确保一切都能正常工作。

1.  接下来，我们将测试受保护的发现服务器的 API 和网页。

1.  之后，我们将学习如何使用 OAuth 2.0 客户端凭据流和授权码授权流获取访问令牌。

1.  使用颁发的访问令牌，我们将测试受保护的 API。我们还将验证为读取器客户端颁发的访问令牌不能用于调用更新 API。

1.  最后，我们还将验证 Swagger UI 是否可以颁发访问令牌并调用 API。

## 构建和运行自动化测试

要构建和运行自动化测试，我们执行以下步骤：

1.  使用以下命令从源代码构建 Docker 镜像：

    ```java
    cd $BOOK_HOME/Chapter11
    ./gradlew build && docker-compose build 
    ```

1.  使用以下命令在 Docker 中启动系统环境并运行常规测试：

    ```java
    ./test-em-all.bash start 
    ```

注意最后的新的负面测试，验证在未认证时返回`401 Unauthorized`代码，在未授权时返回`403 Forbidden`。

## 测试受保护的发现服务器

在受保护的发现服务器 Eureka 启动并运行后，我们必须提供有效的凭据才能访问其 API 和网页。

例如，可以通过以下`curl`命令请求 Eureka 服务器注册的实例，其中我们直接在 URL 中提供用户名和密码：

```java
curl -H "accept:application/json" https://u:p@localhost:8443/eureka/api/apps -ks | jq -r .applications.application[].instance[].instanceId 
```

一个示例响应如下：

![文本描述自动生成](img/B19825_11_03.png)

图 11.3：使用 API 调用在 Eureka 中注册的服务

访问`https://localhost:8443/eureka/web`上的网页时，我们首先必须接受一个不安全的连接，因为我们的证书是自签名的，然后我们必须提供有效的凭据，如配置文件中指定的（`u`作为用户名，`p`作为密码）：

![图形用户界面  自动生成的描述](img/B19825_11_04.png)

图 11.4：Eureka 需要身份验证

登录成功后，我们将看到来自 Eureka 服务器的熟悉网页：

![表格  自动生成的中等置信度描述](img/B19825_11_05.png)

图 11.5：使用网页注册到 Eureka 的服务

在确保对 Eureka 服务器的访问受保护后，我们将学习如何颁发 OAuth 访问令牌。

## 获取访问令牌

现在我们准备使用 OAuth 2.0 定义的授权流程获取访问令牌。我们首先将尝试客户端凭证授权流程，然后是授权码授权流程。

### 使用客户端凭证授权流程获取访问令牌

要为`writer`客户端获取访问令牌，即具有`product:read`和`product:write`作用域，执行以下命令：

```java
curl -k https://writer:secret-writer@localhost:8443/oauth2/token -d grant_type=client_credentials -d scope="product:read product:write" -s | jq . 
```

客户端使用 HTTP 基本身份验证来识别自己，传递其客户端 ID，`writer`，以及其客户端密钥，`secret`。

以下是一个示例响应：

![文本  自动生成的描述](img/B19825_11_06.png)

图 11.6：示例令牌响应

从截图我们可以看到，在响应中我们得到了以下信息：

+   访问令牌本身。

+   授予令牌的作用域。`writer`客户端被授予`product:write`和`product:read`作用域。它还被授予`openid`作用域，允许访问有关用户 ID 的信息，例如电子邮件地址。

+   我们获得的令牌类型；`Bearer`表示持有此令牌的人应根据授予令牌的作用域获得访问权限。

+   访问令牌的有效秒数，在本例中为`3599`秒。

要为`reader`客户端获取访问令牌，即只有`product:read`作用域，只需在先前的命令中将`writer`替换为`reader`，结果如下：

```java
curl -k https://reader:secret-reader@localhost:8443/oauth2/token -d grant_type=client_credentials -d scope="product:read" -s | jq . 
```

### 使用授权码授权流程获取访问令牌

要使用授权码授权流程获取访问令牌，我们需要涉及一个网络浏览器。这个授权流程在部分不安全的环境（网络浏览器）中要复杂一些，以便使其更安全。

在第一个不安全的步骤中，我们将使用网络浏览器获取一个只能使用一次的授权码，以交换访问令牌。授权码将从网络浏览器传递到一个安全层，例如服务器端代码，它可以向授权服务器发出新的请求以交换授权码为访问令牌。在这个安全交换中，服务器必须提供一个客户端密钥来验证其身份。

执行以下步骤以执行授权码授权流程：

1.  要为 `reader` 客户端获取授权码，请在接受自签名证书使用的网页浏览器中输入以下 URL，例如 Chrome：`https://localhost:8443/oauth2/authorize?response_type=code&client_id=reader&redirect_uri=https://my.redirect.uri&scope=product:read&state=35725`。

1.  当网页浏览器要求登录时，使用授权服务器配置中指定的凭据，即 `u` 和 `p`：

![图形用户界面  自动生成的描述](img/B19825_11_07.png)

图 11.7：尝试授权码授权流程

1.  我们将被要求为 `reader` 客户端同意以我们的名义调用 API：

![图形用户界面，应用程序  自动生成的描述](img/B19825_11_08.png)

图 11.8：授权码授权流程同意页面

1.  点击 **提交同意** 按钮后，我们将得到以下响应：

![图形用户界面，文本，应用程序  自动生成的描述](img/B19825_11_09.png)

图 11.9：授权码授权流程重定向页面

1.  初看可能有点失望。授权服务器发送回网页浏览器的 URL 是基于客户端在初始请求中指定的重定向 URI。将 URL 复制到文本编辑器中，你会找到类似以下的内容：

    `https://my.redirect.uri/?code=7XBs...0mmyk&state=35725`

    太好了！我们可以在重定向 URL 的 `code` 请求参数中找到授权码。从 `code` 参数中提取授权码，并定义一个环境变量 `CODE`，其值为：

    ```java
    CODE=7XBs...0mmyk 
    ```

1.  假设你是后端服务器，使用以下 `curl` 命令交换授权码和访问令牌：

    ```java
    curl -k https://reader:secret-reader@localhost:8443/oauth2/token \
     -d grant_type=authorization_code \
     -d client_id=reader \
     -d redirect_uri=https://my.redirect.uri \
     -d code=$CODE -s | jq . 
    ```

    以下是一个示例响应：

    ![文本  自动生成的描述](img/B19825_11_10.png)

    图 11.10：授权码授权流程访问令牌

    从截图可以看出，我们得到的响应信息与客户端凭据流程中得到的类似，但有以下例外：

    +   由于我们使用了更安全的授权流程，我们还获得了一个 `刷新令牌`

    +   由于我们请求了 `reader` 客户端的访问令牌，所以我们只得到了 `product:read` 范围，没有 `product:write` 范围

1.  要为 `writer` 客户端获取授权码，请使用以下 URL：`https://localhost:8443/oauth2/authorize?response_type=code&client_id=writer&redirect_uri=https://my.redirect.uri&scope=product:read+product:write&state=72489`。

1.  要为 `writer` 客户端交换代码以获取访问令牌，请运行以下命令：

    ```java
    curl -k https://writer:secret-writer@localhost:8443/oauth2/token \
      -d grant_type=authorization_code \
      -d client_id=writer \
      -d redirect_uri=https://my.redirect.uri \
      -d code=$CODE -s | jq . 
    ```

验证响应现在也包含 `product:write` 范围！

## 使用访问令牌调用受保护 API

现在，让我们使用我们获得的访问令牌来调用受保护的 API。

预期 OAuth 2.0 访问令牌将以标准 HTTP `authorization` 标头发送，其中访问令牌以 `Bearer` 为前缀。

运行以下命令以调用受保护的 API：

1.  在没有有效访问令牌的情况下调用 API 以检索复合产品：

    ```java
    ACCESS_TOKEN=an-invalid-token
    curl https://localhost:8443/product-composite/1 -k -H "Authorization: Bearer $ACCESS_TOKEN" -i 
    ```

    它应该返回以下响应：

    ![图形用户界面，文本，应用程序  自动生成的描述](img/B19825_11_11.png)

    图 11.11：无效令牌导致 401 未授权响应

    错误信息清楚地表明访问令牌无效！

1.  尝试使用之前章节中为 `reader` 客户端获取的其中一个访问令牌来检索复合产品：

    ```java
    ACCESS_TOKEN={a-reader-access-token}
    curl https://localhost:8443/product-composite/1 -k -H "Authorization: Bearer $ACCESS_TOKEN" -i 
    ```

    现在我们将获得 `200 OK` 状态码，预期的响应体将被返回：

![图形用户界面，文本  自动生成的描述](img/B19825_11_12.png)

图 11.12：有效的访问令牌导致 200 OK 响应

1.  如果我们尝试使用为 `reader` 客户端获取的访问令牌访问一个正在更新的 API，例如删除 API，调用将失败：

    ```java
    ACCESS_TOKEN={a-reader-access-token}
    curl https://localhost:8443/product-composite/999 -k -H "Authorization: Bearer $ACCESS_TOKEN" -X DELETE -i 
    ```

    它将以类似以下响应失败：

    ![图形用户界面，文本  自动生成的描述](img/B19825_11_13.png)

    图 11.13：权限不足导致 403 禁止结果

    从错误响应中可以看出，我们被禁止调用 API，因为请求需要比我们的访问令牌所授予的更高权限。

1.  如果我们重复调用删除 API，但使用为 `writer` 客户端获取的访问令牌，调用将成功，响应为 `202 已接受`。

即使在底层数据库中不存在指定产品 ID 的产品，`delete` 操作也应返回 `202`，因为 `delete` 操作是幂等的，如第六章 *添加持久性* 中所述。请参阅 *添加新 API* 部分。

如果您使用 `docker-compose logs -f product-composite` 命令查看日志输出，应该能够找到如下授权信息：

![文本  自动生成的描述](img/B19825_11_14.png)

图 11.14：日志输出中的授权信息

此信息是从 JWT 编码的访问令牌中提取的 `product-composite` 服务；`product-composite` 服务无需与授权服务器通信即可获取此信息！

通过这些测试，我们已经看到了如何使用客户端凭据和授权代码授权流程获取访问令牌。我们还看到了如何使用作用域来限制客户端可以使用特定访问令牌执行的操作，例如，仅用于读取操作。

## 使用 OAuth 2.0 测试 Swagger UI

在本节中，我们将学习如何使用 Swagger UI 组件访问受保护的 API。上面 *仅针对 product-composite 服务中的更改* 部分中描述的配置允许我们为 Swagger UI 发出访问令牌，并在从 Swagger UI 调用 API 时使用它。

要尝试它，请执行以下步骤：

1.  通过在网页浏览器中访问以下 URL 来打开 Swagger UI 起始页面：`https://localhost:8443/openapi/swagger-ui.html`。

1.  在起始页面上，我们现在可以看到一个新按钮，位于**服务器**下拉列表旁边，按钮上显示文本**授权**。

    点击**授权**按钮以启动授权码授予流程。

1.  Swagger UI 将显示一个作用域列表，它将请求授权服务器获取访问权限。通过点击带有文本**全选**的链接并然后点击**授权**按钮来选择所有作用域：![图形用户界面，应用程序描述自动生成](img/B19825_11_15.png)

    图 11.15：Swagger UI 请求 OAuth 作用域

    您将被重定向到授权服务器。如果您还没有从所使用的网页浏览器登录，授权服务器将要求您提供凭证，就像在*使用授权码授予流程获取访问令牌*部分中描述的那样。

1.  使用用户名`u`和密码`p`登录。

1.  授权服务器将要求您的同意。选择两个作用域并点击**提交同意**按钮。

1.  Swagger UI 将通过显示完成的授权流程信息来完成授权过程。点击**关闭**按钮返回起始页面：

![图形用户界面，应用程序描述自动生成](img/B19825_11_16.png)

图 11.16：Swagger UI 总结 OAuth 授权流程

1.  现在您可以像在*第五章*，*使用 OpenAPI 添加 API 描述*中描述的那样尝试 API。Swagger UI 将把访问令牌添加到请求中。如果您仔细查看**响应**标题下报告的`curl`命令，您可以找到访问令牌。

这完成了我们将使用本地授权服务器进行的测试。在下一节中，我们将用外部 OpenID Connect 兼容提供者替换它。

# 使用外部 OpenID Connect 提供者进行测试

因此，OAuth 舞蹈与我们所控制的授权服务器配合得很好。但如果我们用认证的 OpenID Connect 提供者替换它，会发生什么？从理论上讲，它应该能够直接工作。让我们来看看，好吗？

要获取 OpenID Connect 的认证实现列表，请参阅[`openid.net/developers/certified/`](https://openid.net/developers/certified/).

我们将使用 Auth0，`https://auth0.com/`，作为我们的外部 OpenID 提供者测试。为了能够使用 Auth0 而不是我们自己的授权服务器，我们将讨论以下主题：

+   在 Auth0 中设置一个具有`reader`和`writer`客户端和用户的账户

+   应用使用 Auth0 作为 OpenID 提供者所需的更改

+   运行测试脚本以验证其是否正常工作

+   使用以下授权流程获取访问令牌：

    +   客户端凭证授予流程

    +   授权码授予流程

+   使用从授权流程中获得的访问令牌调用受保护的 API

+   使用用户信息端点获取有关用户的更多信息

让我们在接下来的部分中逐一介绍它们。

## 设置和配置 Auth0 账户

在 Auth0 中所需的大部分配置将由使用 Auth0 管理 API 的脚本处理。但我们必须执行一些手动步骤，直到 Auth0 创建了一个我们可以用来访问管理 API 的客户端 ID 和客户端密钥。Auth0 的服务是多租户的，允许我们创建自己的 OAuth 对象域，包括客户端、资源所有者和资源服务器。

执行以下手动步骤，在 Auth0 注册免费账户并创建一个我们可以用来访问管理 API 的客户端：

1.  在您的浏览器中打开 URL [`auth0.com`](https://auth0.com)。

1.  点击右上角的**汉堡菜单**（☰）。

1.  点击**注册**按钮：

    1.  使用您选择的电子邮件地址进行注册。

    1.  成功注册后，您将被要求创建一个租户域名。输入您选择的租户名称，以我的情况为例：`dev-ml-3rd.eu.auth0.com`。

    1.  按照要求填写您的账户信息。

    1.  同时，查看您的邮箱，寻找主题为**请验证您的 Auth0 账户**的电子邮件，并使用电子邮件中的说明来验证您的账户。

1.  注册后，您将被引导到一个**入门**页面。

1.  在左侧菜单中，点击**应用程序**以展开它，然后点击**APIs**以找到管理 API，**Auth0 管理 API**。此 API 是在创建租户时为您创建的。我们将使用此 API 在租户中创建所需的定义。

1.  点击**Auth0 管理 API**并选择**测试**标签。

1.  将出现一个带有文本**创建并授权测试**的大按钮。点击它以创建一个可以用来访问管理 API 的客户端。

1.  创建后，将显示一个带有标题**从我的应用程序请求 Auth0 令牌**的页面。作为最后一步，我们需要授予创建的客户端使用管理 API 的权限。

1.  点击**测试**标签旁边的**机器到机器应用程序**标签。

1.  在这里我们可以找到测试客户端，**Auth0 管理 API（测试应用程序**），我们可以看到它被授权使用管理 API。如果我们点击**授权**切换按钮旁边的向下箭头，将揭示大量可用的权限。

1.  点击**全部**选项，然后点击**更新**按钮。屏幕应类似于以下截图：

![图形用户界面，应用程序描述自动生成](img/B19825_11_17.png)

图 11.17：Auth0 管理 API 客户端权限

1.  在理解您现在拥有一个可以访问您租户内所有管理 API 的非常强大的客户端后，点击**继续**按钮。

1.  现在，我们只需要收集创建的客户端的客户端 ID 和客户端密钥。最简单的方法是在左侧菜单中选择**应用程序**（在主菜单选择**应用程序**下）然后选择名为**Auth0 管理 API（测试应用程序**）的应用程序。

    应该显示一个类似于以下屏幕的屏幕：

![图形用户界面，应用程序，团队  自动生成的描述](img/B19825_11_18.png)

图 11.18：Auth0 管理 API 客户端应用程序信息

1.  打开文件`$BOOK_HOME/Chapter11/auth0/env.bash`并从上面的屏幕复制以下值：

    1.  **域名**放入变量`TENANT`的值中

    1.  **客户端 ID**放入变量`MGM_CLIENT_ID`的值中

    1.  **客户端密钥**放入变量`MGM_CLIENT_SECRET`的值中

1.  通过指定电子邮件地址和密码，在变量`USER_EMAIL`和`USER_PASSWORD`中完成`env.bash`文件中所需值的设置，该脚本将为我们创建一个测试用户。

从安全角度考虑，为像这样的用户指定密码不被认为是最佳实践。Auth0 支持注册用户，他们可以自己设置密码，但这需要更复杂的设置。有关更多信息，请参阅[`auth0.com/docs/connections/database/password-change`](https://auth0.com/docs/connections/database/password-change)。由于这仅用于测试目的，指定这样的密码是可以的。

我们现在可以运行脚本，该脚本将为我们创建以下定义：

+   两个应用程序，`reader`和`writer`，或者 OAuth 术语中的客户端

+   `product-composite` API，OAuth 术语中的资源服务器，具有 OAuth 作用域`product:read`和`product:write`

+   一个用户，OAuth 术语中的资源所有者，我们将使用它来测试授权代码流

+   最后，我们将授予`reader`应用程序`product:read`作用域，并将`writer`应用程序的作用域设置为`product:read`和`product:write`

1.  运行以下命令：

    ```java
    cd $BOOK_HOME/Chapter11/auth0
    ./setup-tenant.bash 
    ```

预期以下输出（以下输出中已删除详细信息）:

![文本  自动生成的描述](img/B19825_11_19.png)

图 11.19：第一次执行`setup-tenant.bash`时的输出

1.  保存输出末尾打印的`export`命令的副本；我们将在本章的后面多次使用它们。

1.  在您的邮箱中查找为测试用户指定的测试邮件。您将收到一封主题为**验证您的电子邮件**的邮件。*使用邮件中的说明来验证测试用户的电子邮件地址。

注意，该脚本是无状态的，这意味着它可以多次运行而不会破坏配置。如果再次运行脚本，它应该响应：

![](img/B19825_11_20.png)

图 11.20：第二次执行`setup-tenant.bash`时的输出

能够再次运行脚本非常有用，例如，以获取对`reader`和`writer`的客户端 ID 和客户端密钥的访问。

如果您需要删除由 `setup-tenant.bash` 创建的对象，您可以运行脚本 `reset-tenant.bash`。

在创建并配置了 Auth0 账户后，我们可以继续前进并应用系统景观中必要的配置更改。

## 应用必要的更改以使用 Auth0 作为 OpenID 提供者

在本节中，我们将学习需要哪些配置更改才能用 Auth0 替换本地授权服务器。我们只需更改充当 OAuth 资源服务器的两个服务的配置，即 `product-composite` 和 `gateway` 服务。我们还需要稍微修改我们的测试脚本，以便从 Auth0 而不是从本地授权服务器获取访问令牌。让我们从 OAuth 资源服务器，即 `product-composite` 和 `gateway` 服务开始。

本主题中描述的更改尚未应用于书中 Git 仓库中的源代码；您需要自己进行这些更改才能看到它们的效果！

### 在 OAuth 资源服务器中更改配置

如前所述，当使用 OpenID Connect 提供者时，我们只需配置 OAuth 资源服务器中标准发现端点的基 URI。

在 `product-composite` 和 `gateway` 项目中，更新 OIDC 发现端点以指向 Auth0 而不是指向我们的本地授权服务器。在两个项目的 `application.yml` 文件中做出以下更改：

1.  定位属性 `spring.security.oauth2.resourceserver.jwt.issuer-uri`。

1.  将其值替换为 `https://${TENANT}/`，其中 `${TENANT}` 应替换为您的租户域名；在我的情况下，它是 `dev-ml.eu.auth0.com`。**不要**忘记尾随的 `/!`

在我的情况下，OIDC 发现端点的配置将如下所示：

```java
spring.security.oauth2.resourceserver.jwt.issuer-uri: https://dev-ml.eu.auth0.com/ 
```

如果您好奇，可以通过运行以下命令查看发现文档的内容：

```java
curl https://${TENANT}/.well-known/openid-configuration -s | jq 
```

按以下方式重新构建 `product-composite` 和 `gateway` 服务：

```java
cd $BOOK_HOME/Chapter11
./gradlew build && docker-compose up -d --build product-composite gateway 
```

随着 `product-composite` 和 `gateway` 服务的更新，我们可以继续前进并更新测试脚本。

### 修改测试脚本以从 Auth0 获取访问令牌

我们还需要更新测试脚本，以便从 Auth0 OIDC 提供者获取访问令牌。这通过在 `test-em-all.bash` 中执行以下更改来完成：

1.  找到以下命令：

    ```java
    ACCESS_TOKEN=$(curl -k https://writer:secret-writer@$HOST:$PORT/oauth2/token -d grant_type=client_credentials -d scope="product:read product:write" -s | jq .access_token -r) 
    ```

1.  用以下命令替换：

    ```java
    export TENANT=...
    export WRITER_CLIENT_ID=...
    export WRITER_CLIENT_SECRET=...
    ACCESS_TOKEN=$(curl -X POST https://$TENANT/oauth/token \
      -d grant_type=client_credentials \
      -d audience=https://localhost:8443/product-composite \
      -d scope=product:read+product:write \
      -d client_id=$WRITER_CLIENT_ID \
      -d client_secret=$WRITER_CLIENT_SECRET -s | jq -r .access_token) 
    ```

注意到前一个命令，Auth0 要求我们指定请求访问令牌的预期**受众**，作为额外的安全层。受众是我们计划使用访问令牌调用的 API。鉴于 API 实现验证 `audience` 字段，这将防止有人试图使用为其他目的签发的访问令牌来访问 API。

1.  在前面的命令中设置环境变量 `TENANT`、`WRITER_CLIENT_ID` 和 `WRITER_CLIENT_SECRET` 的值，这些值由 `setup-tenant.bash` 脚本返回。

如上所述，您可以再次运行脚本以获取这些值，而不会产生任何负面影响！

1.  找到以下命令：

    ```java
    READER_ACCESS_TOKEN=$(curl -k https://reader:secret-reader@$HOST:$PORT/oauth2/token -d grant_type=client_credentials -d scope="product:read" -s | jq .access_token -r) 
    ```

1.  用以下命令替换：

    ```java
    export READER_CLIENT_ID=...
    export READER_CLIENT_SECRET=...
    READER_ACCESS_TOKEN=$(curl -X POST https://$TENANT/oauth/token \
      -d grant_type=client_credentials \
      -d audience=https://localhost:8443/product-composite \
      -d scope=product:read \
      -d client_id=$READER_CLIENT_ID \
      -d client_secret=$READER_CLIENT_SECRET -s | jq -r .access_token) 
    ```

注意，我们在这里只请求 `product:read` 范围，而不是 `product:write` 范围。

1.  在前面的命令中，将环境变量 `READER_CLIENT_ID` 和 `READER_CLIENT_SECRET` 的值设置为 `setup-tenant.bash` 脚本返回的值。

现在访问令牌由 Auth0 而不是我们的本地授权服务器颁发，我们的 API 实现可以使用在 `application.yml` 文件中配置的 Auth0 的发现服务的信息来验证访问令牌。API 实现可以像以前一样，使用访问令牌中的范围来授权客户端执行对 API 的调用，或者不授权。

这样，我们就完成了所有必要的更改。让我们运行一些测试来验证我们是否可以从 Auth0 获取访问令牌。

## 使用 Auth0 作为 OpenID Connect 提供者运行测试脚本

现在，我们已经准备好尝试使用 Auth0 了！

运行常规测试，但这次使用 Auth0 作为 OpenID Connect 提供者，以下命令：

```java
./test-em-all.bash 
```

在日志中，您将能够找到来自 Auth0 颁发的访问令牌的授权信息。运行以下命令：

```java
docker-compose logs product-composite | grep "Authorization info" 
```

预期以下命令的输出：

1.  从使用具有 `product:read` 和 `product:write` 范围的访问令牌的调用中，我们将看到以下范围列出：

![文本描述自动生成](img/B19825_11_21.png)

图 11.21：日志输出中来自 Auth0 的写客户端的授权信息

1.  从仅使用 `product:read` 范围的访问令牌的调用中，我们将看到只列出以下范围：

![文本描述自动生成](img/B19825_11_22.png)

图 11.22：日志输出中来自 Auth0 的读客户端的授权信息

如我们从日志输出中看到的那样，我们现在还获得了有关此访问令牌的**预期受众**的信息。为了加强安全性，我们可以在我们的服务中添加一个测试来验证其 URL（在这种情况下为 `https://localhost:8443/product-composite`）是否是受众列表的一部分。正如之前提到的，这将防止有人试图使用为获取对我们的 API 访问权限以外的目的而颁发的访问令牌。

当与 Auth0 的自动化测试一起工作时，我们可以继续学习如何使用不同类型的授权流程获取访问令牌。让我们从客户端凭证授权流程开始。

## 使用客户端凭证授权流程获取访问令牌

如果您想自己从 Auth0 获取访问令牌，可以通过运行以下命令，使用客户端凭证授权流程来实现：

```java
export TENANT=...
export WRITER_CLIENT_ID=...
export WRITER_CLIENT_SECRET=...
curl -X POST https://$TENANT/oauth/token \
  -d grant_type=client_credentials \
  -d audience=https://localhost:8443/product-composite \
  -d scope=product:read+product:write \
  -d client_id=$WRITER_CLIENT_ID \
  -d client_secret=$WRITER_CLIENT_SECRET 
```

在前面的命令中，将环境变量 `TENANT`、`WRITER_CLIENT_ID` 和 `WRITER_CLIENT_SECRET` 的值设置为 `setup-tenant.bash` 脚本返回的值。

按照第*使用访问令牌调用受保护的 API*部分的说明，你应该能够使用获取的访问令牌调用 API。

## 使用授权码授权流程获取访问令牌

在本节中，我们将学习如何使用授权码授权流程从 Auth0 获取访问令牌。如上所述，我们首先需要使用网络浏览器获取授权码。接下来，我们可以使用服务器端代码将授权码交换为访问令牌。

执行以下步骤以执行使用 Auth0 的授权码授权流程：

1.  要为默认应用程序客户端获取授权码，请在网络浏览器中使用以下 URL：

    `https://${TENANT}/authorize?audience=https://localhost:8443/product-composite&scope=openid email product:read product:write&response_type=code&client_id=${WRITER_CLIENT_ID}&redirect_uri=https://my.redirect.uri&state=845361`。

1.  将前面的 URL 中的`${TENANT}`和`${WRITER_CLIENT_ID}`替换为`setup-tenant.bash`脚本返回的租户域名和 writer 客户端 ID。

1.  Auth0 应显示以下登录屏幕：

![图形用户界面，应用程序描述自动生成](img/B19825_11_23.png)

图 11.23：使用 Auth0 的授权码授权流程，登录屏幕

1.  登录成功后，Auth0 会要求你同意客户端应用程序：

![图形用户界面，应用程序描述自动生成](img/B19825_11_24.png)

图 11.24：使用 Auth0 的授权码授权流程，同意屏幕

1.  授权码现在位于浏览器中的 URL 中，就像我们尝试使用本地授权服务器进行授权码授权流程时一样：

![图形用户界面，应用程序描述自动生成](img/B19825_11_25.png)

图 11.25：使用 Auth0 的授权码授权流程，访问令牌

1.  提取代码并运行以下命令以获取访问令牌：

    ```java
    CODE=...
    export TENANT=...
    export WRITER_CLIENT_ID=...
    export WRITER_CLIENT_SECRET=...
    curl -X POST https://$TENANT/oauth/token \
     -d grant_type=authorization_code \
     -d client_id=$WRITER_CLIENT_ID \
     -d client_secret=$WRITER_CLIENT_SECRET  \
     -d code=$CODE \
     -d redirect_uri=https://my.redirect.uri -s | jq . 
    ```

在前面的命令中设置环境变量`TENANT`、`WRITER_CLIENT_ID`和`WRITER_CLIENT_SECRET`的值为`setup-tenant.bash`脚本返回的值。

现在我们已经学会了如何使用两种授权流程获取访问令牌，我们准备在下一节尝试使用从 Auth0 获取的访问令牌调用外部 API。

## 使用 Auth0 访问令牌调用受保护的 API

我们可以使用 Auth0 签发的访问令牌来调用我们的 API，就像我们使用本地授权服务器签发的访问令牌一样。

对于只读 API，执行以下命令：

```java
ACCESS_TOKEN=...
curl https://localhost:8443/product-composite/1 -k -H "Authorization: Bearer $ACCESS_TOKEN" -i 
```

对于更新中的 API，执行以下命令：

```java
ACCESS_TOKEN=...
curl https://localhost:8443/product-composite/999 -k -H "Authorization: Bearer $ACCESS_TOKEN" -X DELETE -i 
```

由于我们请求了两个作用域，`product:read`和`product:write`，因此预期的前一个 API 调用都将返回`200 OK`。

## 获取有关用户的额外信息

从“使用 Auth0 作为 OpenID Connect 提供者的运行测试脚本”部分中的*图 11.21*和*图 11.22*的日志输出中，我们看不到任何关于发起 API 请求的用户的信息。如果您希望您的 API 实现能够了解更多关于用户的信息，它可以调用 Auth0 的`userinfo_endpoint`。`userinfo`端点的 URL 可以在对 OIDC 发现端点的请求的响应中找到，如*在 OAuth 资源服务器中的配置更改*部分所述。要获取与访问令牌相关的用户信息，请执行以下请求：

```java
Export TENANT=...
curl -H "Authorization: Bearer $ACCESS_TOKEN" https://$TENANT/userinfo -s | jq 
```

在前面的命令中设置`TENANT`环境变量的值，使其等于`setup-tenant.bash`脚本返回的值。

注意，此命令仅适用于使用授权码授权流程颁发的访问令牌。使用客户端凭据授权流程颁发的访问令牌不包含任何用户信息，如果尝试使用，将导致错误响应。

一个示例响应如下：

![文本描述自动生成](img/B19825_11_26.png)

图 11.26：从 Auth0 请求额外的用户信息

此端点也可以用来验证用户是否没有在 Auth0 中撤销访问令牌。

通过以下命令关闭系统景观以结束测试：

```java
docker-compose down 
```

这部分内容到此结束，我们学习了如何用外部替代方案替换本地 OAuth 2.0 授权服务器。我们还看到了如何重新配置微服务景观，以使用外部 OIDC 提供者验证访问令牌。

# 摘要

在本章中，我们学习了如何使用 Spring Security 来保护我们的 API。

我们看到，使用 Spring Security 启用 HTTPS 以防止第三方窃听是多么容易。使用 Spring Security，我们还了解到，使用 HTTP 基本身份验证限制对发现服务器 Netflix Eureka 的访问是直接了当的。最后，我们看到了如何使用 Spring Security 简化 OAuth 2.0 和 OpenID Connect 的使用，允许第三方客户端应用程序以用户的名义访问我们的 API，但不需要用户与客户端应用程序共享凭据。我们学习了如何设置基于 Spring Security 的本地 OAuth 2.0 授权服务器，以及如何更改配置，以便可以使用外部 OpenID Connect 提供者 Auth0。

然而，一个担忧是，如何管理所需的配置。每个微服务实例都必须提供其自己的配置，这使得很难获得当前配置的良好概览。更新涉及多个微服务的配置也将具有挑战性。加之分散的配置，我们之前看到的一些配置包含敏感信息，如凭证或证书。这似乎需要一种更好的方式来处理多个协作微服务的配置，以及如何处理配置敏感部分的方法。

在下一章中，我们将探讨 Spring Cloud Config Server，并看看它是如何被用来处理这些类型的问题的。

# 问题

1.  使用自签名证书的好处和缺点是什么？

1.  OAuth 2.0 授权码的目的是什么？

1.  OAuth 2.0 范围的目的是什么？

1.  当一个令牌是 JWT 时，这意味着什么？

1.  我们如何信任存储在 JWT 中的信息？

1.  使用 OAuth 2.0 授权码授权流程与原生移动应用是否合适？

1.  OpenID Connect 为 OAuth 2.0 增加了什么？
