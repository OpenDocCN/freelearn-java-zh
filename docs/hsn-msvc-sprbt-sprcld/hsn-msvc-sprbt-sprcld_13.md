# 第十一章：保护 API 访问

在本章中，我们将了解如何保护前一章中引入的边缘服务器暴露的 API 和网页。我们将学习使用 HTTPS 来防止对 API 的外部访问进行窃听，并了解如何使用 OAuth 2.0 和 OpenID Connect 对用户和客户端应用程序访问我们的 API 进行认证和授权。最后，我们将研究使用 HTTP 基本认证来保护对 Netflix Eureka 发现服务的访问。

本章将涵盖以下主题：

+   介绍 OAuth 2.0 和 OpenID Connect 标准

+   关于如何保护系统架构的一般讨论

+   在我们的系统架构中添加一个授权服务器

+   使用 HTTPS 保护外部通信

+   保护对 Netflix Eureka 发现服务的访问

+   使用 OAuth 2.0 和 OpenID Connect 对 API 访问进行认证和授权

+   使用本地授权服务器进行测试

+   使用 Auth0 的 OpenID Connect 提供者进行测试

# 技术要求

本书中描述的所有命令都是在 MacBook Pro 上使用 macOS Mojave 运行的，但是修改它们以便在另一个平台如 Linux 或 Windows 上运行应该是很直接的。

在本章中不需要安装任何新工具。

本章的源代码可以在 GitHub 上找到，地址为[`github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud/tree/master/Chapter11`](https://github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud/tree/master/Chapter11)。

为了能够按照书中描述运行命令，将源代码下载到一个文件夹中，并设置一个环境变量`$BOOK_HOME`，使其指向该文件夹。可以使用以下命令来执行这些步骤：

```java
export BOOK_HOME=~/Documents/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud
git clone https://github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud $BOOK_HOME
cd $BOOK_HOME/Chapter11
```

Java 源代码是为 Java 8 编写的，并在 Java 12 上进行了测试。本章使用 Spring Cloud 2.1.0, SR1（也称为**Greenwich**版本），Spring Boot 2.1.3 和 Spring 5.1.5，即在编写时可用的 Spring 组件的最新版本。

源代码包含以下 Gradle 项目：

+   `api`

+   `util`

+   `microservices/product-service`

+   `microservices/review-service`

+   `microservices/recommendation-service`

+   `microservices/product-composite-service`

+   `spring-cloud/eureka-server`

+   `spring-cloud/gateway`

+   `spring-cloud/authorization-server`

本章中的代码示例都来自`$BOOK_HOME/Chapter11`中的源代码，但在许多情况下，都编辑了源代码中与主题无关的部分，例如注释、导入和日志声明。

如果你想要查看源代码中在第十一章，*API 的安全访问*中应用的改动，也就是查看在微服务架构中为 API 安全访问所付出的努力，你可以将其与第十章，*使用 Spring Cloud Gateway 将微服务隐藏在边缘服务器后面*的源代码进行对比。你可以使用你喜欢的`diff`工具，比较两个文件夹，`$BOOK_HOME/Chapter10`和`$BOOK_HOME/Chapter11`。

# 介绍 OAuth 2.0 和 OpenID Connect

在介绍 OAuth 2.0 和 OpenID Connect 之前，让我们先澄清我们所说的认证和授权是什么意思。**认证**意味着通过验证用户提供的凭据来识别用户，例如用户名和密码。**授权**是关于给予认证用户，也就是已识别用户，访问我们这里所说的 API 的不同部分的权限。在我们这里，一个用户将被分配一组基于 OAuth 2.0 范围的特权，如下所述。微服务将基于这些特权来确定用户是否被允许访问一个 API。

**OAuth 2.0**是一个开放标准的授权协议，而**OpenID Connect**是 OAuth 2.0 的一个补充，它允许客户端应用程序基于授权服务器执行的认证来验证用户身份。让我们简要地分别了解 OAuth 2.0 和 OpenID Connect，以获得它们目的的初步理解！

# 介绍 OAuth 2.0

**OAuth** *2.0* 是一个广泛接受的开源标准授权协议，它允许用户授权第三方客户端应用以用户的名义访问受保护的资源。

那么，这意味着什么呢？

让我们先来梳理一下所使用的概念：

+   **资源所有者**：终端用户。

+   **客户端**：第三方客户端应用程序，例如，一个网络应用或一个原生移动应用，它想要以终端用户的名义调用一些受保护的 API。

+   **资源服务器**：暴露我们想要保护的 API 的服务器。

+   **授权服务器**：在资源所有者，即终端用户被认证后，向客户端发放令牌的服务器。用户信息的管理和用户的认证通常是在幕后委托给一个**身份提供者**（**IdP**）。

客户端在授权服务器中注册，并获得一个**客户端 ID**和一个**客户端密钥**。客户端密钥必须像密码一样受到客户端的保护。客户端还会获得一组允许的**重定向 URI**，授权服务器在用户被认证后使用这些 URI 将**授权码**和**令牌**发送回客户端应用程序。

以下是一个说明性的例子。假设一个用户访问了一个第三方客户端应用程序，并且客户端应用程序想要调用一个受保护的 API 来为用户服务。为了允许访问这些 API，客户端应用程序需要一种方式来告诉 API 它是在用户的名义下操作。为了避免用户必须与客户端应用程序共享他们的凭据以进行身份验证的解决方案，授权服务器会发放一个 **访问令牌**，该令牌允许客户端应用程序在用户的名义下访问一组选定的 API。

这意味着用户永远不需要向客户端应用程序透露他们的凭据。用户还可以给予客户端应用程序访问代表用户的具体 API 的权限。访问令牌代表了一组时间受限的访问权限，在 OAuth 2.0 术语中表达为 *scope*。一个 **刷新令牌** 也可以由授权服务器发放给客户端应用程序。客户端应用程序可以使用刷新令牌来获取新的访问令牌，而无需涉及用户。

OAuth 2.0 规格定义了四种授权 grant 流程，用于发放访问令牌，如下解释：

+   **授权码 grant 流程**：这是最安全，也是最复杂的 grant 流程。此流程需要用户通过网络浏览器与授权服务器进行交互，以进行身份验证和对客户端应用程序给予同意，如下面的图示：

![](img/186b42c1-4690-4407-99c1-c0f71c1894c4.png)

此图的解释如下：

1.  1.  客户端应用程序通过在网络浏览器中发送用户到授权服务器来启动授权流程。

    1.  授权服务器将会验证用户并请求用户的同意。

    1.  授权服务器将会把用户通过一个 grant code 重定向回客户端应用程序。授权服务器将会使用客户端在第一步中指定的 redirect-URI 来知道重定向的位置。由于 grant code 是通过网络浏览器，也就是到一个不安全的环境中，恶意 JavaScript 代码可能获取 grant code，因此 grant code 仅允许使用一次，并且只在短时间内有效。

    1.  为了用 grant code 换取访问令牌，客户端应用程序预计需要再次调用授权服务器，使用服务器端代码。客户端应用程序必须向授权服务器提供其客户端 ID 和客户端密钥以及 grant code。

    1.  授权服务器发放一个访问令牌并将其发送回客户端应用程序。授权服务器还可以选择性地发放并返回一个刷新令牌。

    1.  使用访问令牌，客户端可以向资源服务器公开的受保护 API 发送请求。

    1.  资源服务器在验证访问令牌成功后验证访问令牌并处理请求。只要访问令牌有效，步骤 6 和 7 可以重复进行。当访问令牌的寿命过期时，客户端可以使用他们的刷新令牌来获取新的访问令牌。

+   **隐式授权流**：这个流程也是基于网络浏览器的，但旨在为无法保护客户端密钥的客户端应用程序，例如单页网络应用程序。它从授权服务器获取访问令牌，而不是授权代码，但由于它使用的是比代码授权流安全性较低的隐式授权流，因此无法请求刷新令牌。

+   **资源所有者密码凭证授权流**：如果客户端应用程序不能与网络浏览器进行交互，它可以回退到这个授权流。在这个授权流中，用户必须与客户端应用程序共享他们的凭据，并且客户端应用程序将使用这些凭据来获取访问令牌。

+   **客户端凭证授权流**：当客户端应用程序需要调用与特定用户无关的 API 时，它可以使用这个授权流来使用自己的客户端 ID 和客户端密钥获取访问令牌。

当涉及到针对受 OAuth 2.0 保护的 API 进行自动化测试时，**资源所有者密码凭证授权流**非常方便，因为它不需要使用网络浏览器进行手动交互。我们稍后将在本章中使用这个授权流与我们的测试脚本一起使用；请参阅*测试脚本的变化*部分。

完整的规范可以在这里找到：[`tools.ietf.org/html/rfc6749`](https://tools.ietf.org/html/rfc6749)。还有许多其他规范详细说明了 OAuth 2.0 的各种方面；要了解概况，请参阅[`www.oauth.com/oauth2-servers/map-oauth-2-0-specs/`](https://www.oauth.com/oauth2-servers/map-oauth-2-0-specs/)。

一个值得额外关注的标准是*RFC 7636 – OAuth 公共客户端（PKCE）代码交换证明密钥*[https://tools.ietf.org/html/rfc7636]。这个标准描述了如何通过添加一个额外的安全层来使一个通常不安全的公共客户端（如移动原生应用或桌面应用）利用代码授权流。

# 保护发现服务，Netflix Eureka 的安全性

以前，我们学习了用 HTTPS 保护外部通信。现在我们将使用 HTTP 基本认证来限制对发现服务器上 API 和网页的访问，即要求用户提供用户名和密码以获得访问权限。需要在 Eureka 服务器以及下面描述的 Eureka 客户端上进行更改。

# 介绍 OpenID Connect

**OpenID Connect**（缩写为**OIDC**）正如前面已经提到的，是 OAuth 2.0 的一个补充，使客户端应用程序能够验证用户身份。OIDC 添加了一个额外的令牌，即 ID 令牌，客户端应用程序在完成授权流程后从授权服务器获得。

身份令牌编码为**JSON Web Token**（**JWT**），包含用户 ID 和电子邮件地址等声明。身份令牌使用 JSON Web 签名进行数字签名。这使得客户端应用程序可以通过使用授权服务器提供的公钥验证数字签名来信任身份令牌中的信息。

可选地，访问令牌也可以以与 ID 令牌相同的方式进行编码和签名，但根据规范这不是强制性的。最后，OIDC 定义了一个**发现端点**，这是建立重要端点的 URL 的标准方式，例如启动授权流程、获取验证数字签名 JWT 令牌的公钥，以及一个**用户信息端点**，可以使用该端点根据用户的访问令牌获取额外信息。

要查看可用的规范，请参阅[`openid.net/developers/specs/`](https://openid.net/developers/specs/)。

这结束了我们对 OAuth 2.0 和 OpenID Connect 标准的介绍。在下一节中，我们将了解系统景观将如何得到保护的高级视图。

# 保护系统景观

为了确保本章介绍的系统景观的安全，我们将执行以下步骤：

+   使用 HTTPS 对发往我们外部 API 的外部请求和响应进行加密，以防止窃听

+   使用 OAuth 2.0 和 OpenID Connect 对访问我们 API 的用户和客户端应用程序进行身份验证和授权

+   使用 HTTP 基本认证访问发现服务 Netflix Eureka

我们只将为边缘服务器的外部通信应用 HTTPS，而将系统景观内部的通信使用普通 HTTP。

在本书稍后的第十八章（第十八章，*使用服务网格提高可观测性和管理*)中，我们将看到如何借助服务网格产品自动为系统景观内的通信提供 HTTPS 加密。

出于测试目的，我们将在我们的系统景观中添加一个本地 OAuth *2.0* 授权服务器。所有与授权服务器的外部通信将通过边缘服务器路由。边缘服务器和产品组合服务将作为 OAuth 2.0 资源服务器运行；也就是说，它们将要求有效的 OAuth 2.0 访问令牌才能访问。

为了最小化验证访问令牌的开销，我们将假设它们以签名 JWT 令牌的形式编码，并且授权服务器暴露了一个端点，资源服务器可以使用该端点访问验证签名所需的公钥，也称为`jwk-set`。

系统景观将如下所示：

![](img/c45c1b73-3816-4923-aca4-414cef12a865.png)

从前面的图表中，我们可以注意到：

1.  外部通信使用 HTTPS，而系统景观内部使用明文 HTTP。

1.  本地 OAuth 2.0 授权服务器将通过边缘服务器外部访问。

1.  边缘服务器和产品复合微服务都将验证作为签名 JWT 令牌的访问令牌。

1.  边缘服务器和产品复合微服务将从授权服务器的`jwk-set`端点获取其公钥，并使用它们验证基于 JWT 的访问令牌的签名。

请注意，我们将重点关注通过 HTTP 保护 API 的访问，而不是涵盖通用最佳实践来保护网络应用程序，例如，管理由*类别：OWASP 前十名项目*指出的网络应用程序安全风险。有关 OWASP 前十名的更多信息，请参阅[`www.owasp.org/index.php/Category:OWASP_Top_Ten_Project`](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_Project)。

在了解了系统景观如何得到保护的概述之后，让我们首先在系统景观中添加一个本地授权服务器。

# 在我们的系统景观中添加授权服务器

为了能够本地运行并完全自动化地与使用 OAuth 2.0 和 OpenID Connect 保护的 API 进行测试，我们将向我们的系统景观中添加一个基于 OAuth 2.0 的授权服务器。Spring Security 5.1 不幸地没有提供开箱即用的授权服务器。但是有一个遗留项目（目前处于维护模式），Spring Security OAuth，提供了一个我们可以使用的授权服务器。

实际上，在 Spring Security 5.1 提供的示例中，一个使用来自 Spring Security OAuth 的授权服务器的项目是可用的。它被配置为使用 JWT 编码的访问令牌，并且它还暴露了一个**JSON Web 密钥集**（**JWKS**）（OpenID Connect 发现标准的组成部分）的端点，这是一组包含资源服务器可以用来验证由授权服务器发行的 JWT 令牌的公钥的密钥。

因此，即使它不是一个完整的 OpenID Connect 提供者，它也适合与我们可以本地运行并完全自动化的测试一起使用。

有关 Spring Security 中计划支持 OAuth 2.0 的更多详细信息，请参阅[`spring.io/blog/2018/01/30/next-generation-oauth-2-0-support-with-spring-security`](https://spring.io/blog/2018/01/30/next-generation-oauth-2-0-support-with-spring-security)。

在 Spring Security 示例项目中，授权服务器可以在这里找到：[`github.com/spring-projects/spring-security/tree/master/samples/boot/oauth2authorizationserver`](https://github.com/spring-projects/spring-security/tree/master/samples/boot/oauth2authorizationserver)。

Spring Security 示例项目配置了两个 OAuth 客户端，`reader`和`writer`，其中`reader`客户端被授予了读取作用域，而`writer`客户端则被授权了读取和写入作用域。两个客户端都配置为将机密设置为`secret`；参考`sample.AuthorizationServerConfiguration`类中的`configure()`方法。

以下更改已应用于示例项目：

+   已经在与其它微服务相同的方式下添加了一个 Eureka 客户端。参见第九章，*使用 Netflix Eureka 和 Ribbon 添加服务发现；* 参考*将微服务连接到 Netflix Eureka 服务器*部分。

+   添加了 Spring Boot Actuator 以获取对`health`端点的访问。

+   添加了一个 Dockerfile，以便能够以 Docker 容器的形式运行授权服务器。

+   `spring-security-samples-boot-oauth2authorizationserver.gradle`构建文件已更改为与本书源代码中使用的`build.gradle`文件更加相似。

+   `sample/AuthorizationServerConfiguration`类中的配置已经按照以下方式更改：

    +   已经添加了对以下授权类型的支持：`code`、`authorization_code`和`implicit`。

    +   作用域的名称`message:read`和`message:write`已经被改成了`product:read`和`product:write`。

    +   在授权服务器中注册的用户用户名已更改为`magnus`，密码为`password`；参考`UserConfig`类中的`userDetailsService()`方法（在`sample/AuthorizationServerConfiguration.java`文件中找到）。

授权服务器的源代码可以在`$BOOK_HOME/Chapter11/spring-cloud/authorization-server`中找到。

为了将授权服务器整合到系统景观中，已经应用了一系列更改。已将授权服务器添加到以下内容中：

+   常用的构建文件`settings.gradle`

+   三个 Docker Compose 文件`docker-compose*.yml`

+   边缘服务器，`spring-cloud/gateway`：

    +   在`HealthCheckConfiguration`中添加了一个健康检查。

    +   添加了一个以`/oauth/`开头的 URI 路由。

了解了如何在系统景观中添加一个本地授权服务器之后，让我们继续探讨如何使用 HTTPS 来保护外部通信免遭窃听。

# 使用 HTTPS 保护外部通信

在本节中，我们将学习如何防止通过边缘服务器公开的公共 API 窃听外部通信，例如来自互联网的通信。我们将使用 HTTPS 来加密通信。要使用 HTTPS，我们需要执行以下操作：

+   **创建证书**：我们将创建自己的自签名证书，足够用于开发目的。

+   **配置边缘服务器**：必须配置它只接受基于 HTTPS 的外部流量并使用证书。

自签名的证书是通过以下命令创建的：

```java
keytool -genkeypair -alias localhost -keyalg RSA -keysize 2048 -storetype PKCS12 -keystore edge.p12 -validity 3650
```

源代码附带一个示例证书文件，所以你不需要运行这个命令来运行以下示例。

命令将询问多个参数。当询问密码时，我输入了`password`。对于其他参数，我简单地输入了一个空值来接受默认值。创建的证书文件`edge.p12`放在`gateway`项目的`src/main/resources/keystore`文件夹中。这意味着当它被构建时，证书文件将被放在`.jar`文件中，并在运行时类路径上可用的位置：`keystore/edge.p12`。

在开发过程中使用类路径提供证书是足够的，但不适用于其他环境，例如生产环境。以下是如何在运行时用外部证书替换此证书的示例！

为了配置边缘服务器使用证书和 HTTPS，以下内容添加到`gateway`项目的`application.yml`中：

```java
server.port: 8443

server.ssl:
 key-store-type: PKCS12
 key-store: classpath:keystore/edge.p12
 key-store-password: password
 key-alias: localhost
```

以下是对前面源代码的解释：

+   证书的路径在`server.ssl.key-store`参数中指定，设置为`classpath:keystore/edge.p12`的值。这意味着证书将从类路径上的位置`keystore/edge.p12`加载。

+   证书的密码在`server.ssl.key-store-password`参数中指定。

+   为了指示边缘服务器使用 HTTPS 而不是 HTTP，我们还在`server.port`参数中将端口从`8080`更改为`8443`。

除了在边缘服务器中的这些更改外，还需要在以下文件中进行更改，以反映端口和 HTTP 协议的变化：

+   三个 Docker Compose 文件，`docker-compose*.yml`

+   测试脚本，`test-em-all.bash`

如前所述，使用类路径提供证书仅适用于开发；让我们看看我们如何在运行时用外部证书替换这个证书！

# 在运行时替换自签名证书

将自签名证书放在`.jar`文件中仅适用于开发。对于运行时环境的工作解决方案，例如用于测试或生产，必须能够使用由授权**CA**（简称**证书颁发机构**）签名的证书。

还必须能够在不需要重新构建`.jar`文件的情况下指定在运行时使用的证书，并且在使用 Docker 时，能够在包含`.jar`文件的 Docker 镜像中指定证书。当使用 Docker Compose 管理 Docker 容器时，我们可以在 Docker 容器中映射一个卷到位于 Docker 宿主机上的证书。我们还可以为 Docker 容器设置环境变量，指向 Docker 卷中的新证书。

在第十五章，《Kubernetes 简介》中，我们将学习 Kubernetes，在那里我们将看到更适合在集群中运行 Docker 容器的处理秘密（如证书）的更强大解决方案；也就是说，容器调度在 Docker 宿主机的组上，而不是单个 Docker 宿主机上。

本主题描述的更改没有应用到书籍 GitHub 仓库中的源代码中；也就是说，你需要亲自做出这些更改才能看到它们的作用！

要替换`.jar`文件中的证书，请执行以下步骤：

1.  创建第二个证书，并将密码设置为`testtest`：

```java
cd $BOOK_HOME/Chapter11
mkdir keystore 
keytool -genkeypair -alias localhost -keyalg RSA -keysize 2048 -storetype PKCS12 -keystore keystore/edge-test.p12 -validity 3650
```

1.  更新`docker-compose.yml`文件，用新的证书位置和密码以及映射到新证书放置文件夹的卷的环境变量。更改后，边缘服务器的配置将如下所示：

```java
gateway:
  environment:
    - SPRING_PROFILES_ACTIVE=docker
    - SERVER_SSL_KEY_STORE=file:/keystore/edge-test.p12
    - SERVER_SSL_KEY_STORE_PASSWORD=testtest
  volumes:
    - $PWD/keystore:/keystore
  build: spring-cloud/gateway
  mem_limit: 350m
  ports:
    - "8443:8443"
```

1.  如果边缘服务器正在运行，它需要使用以下命令重新启动：

```java
docker-compose up -d --scale gateway=0
docker-compose up -d --scale gateway=1
```

`docker-compose restart gateway`命令看起来像是重启`gateway`服务的不错选择，但实际上并没有考虑`docker-compose.yml`中的更改。因此，在这个情况下，它不是一个有用的命令。

新的证书现在正在使用中！

本节介绍了如何使用 HTTPS 保护外部通信的内容。下一节我们将学习如何使用 HTTP 基本认证保护 Netflix Eureka 发现服务。

# Eureka 服务器的变化

为了保护 Eureka 服务器，已经应用了以下更改：

1.  在`build.gradle`中添加了 Spring Security 依赖项：

```java
implementation 'org.springframework.boot:spring-boot-starter-security'
```

1.  `se.magnus.springcloud.eurekaserver.SecurityConfig`类中添加了安全配置：

+   用户定义如下：

```java
public void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth.inMemoryAuthentication()
   .passwordEncoder(NoOpPasswordEncoder.getInstance())
   .withUser(username).password(password)
   .authorities("USER");
}
```

+   `username`和`password`从配置文件中注入到构造函数中：

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

+   所有 API 和网页都通过以下定义使用 HTTP 基本认证进行保护：

```java
protected void configure(HttpSecurity http) throws Exception {
  http
    .authorizeRequests()
      .anyRequest().authenticated()
      .and()
      .httpBasic();
}
```

1.  用户凭据在配置文件`application.yml`中设置：

```java
app:
 eureka-username: u
 eureka-password: p
```

1.  最后，测试类`se.magnus.springcloud.eurekaserver.EurekaServerApplicationTests`在测试 Eureka 服务器的 API 时使用了配置文件中的凭据：

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

前面的是限制对发现服务器 Netflix Eureka 的 API 和网页访问所需的步骤，现在它将使用 HTTP 基本认证，并要求用户提供用户名和密码以获取访问权限。在下一节中，我们将学习如何配置 Netflix Eureka 客户端，以便在访问 Netflix Eureka 服务器时传递凭据。

# 更改 Eureka 客户端

对于 Eureka 客户端，凭据必须指定在 Eureka 服务器的连接 URL 中。这在每个客户端的配置文件`application.yml`中如下指定：

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

我们将在 Netflix Eureka 客户端测试受保护的系统架构时看到这个配置的使用，在*使用本地授权服务器进行测试*一节中。

在下一节中，我们将学习如何在手动访问 Netflix Eureka 服务器时添加凭据，无论是使用其 API 还是其网页。

# 测试受保护的 Eureka 服务器

一旦受保护的 Eureka 服务器运行起来，我们必须提供有效的凭据，才能访问其 API 和网页。

例如，通过以下`curl`命令向 Eureka 服务器请求注册实例：

```java
curl -H "accept:application/json" https://u:p@localhost:8443/eureka/api/apps -ks | jq -r .applications.application[].instance[].instanceId
```

以下是一个示例响应：

![](img/6d901aa7-4296-49ed-ba1d-eae67342dced.png)

当访问`https://localhost:8443/eureka/web`网页时，我们首先必须接受一个不安全的连接，因为我们的证书是自签名的，接下来我们必须提供有效的凭据，如前面配置文件中所指定：

![](img/e043650f-1b2b-479f-a3c6-57ff2aa0d767.png)

登录成功后，我们将看到 Eureka 服务器的熟悉网页：

![](img/46c6f213-a33d-4599-9fdf-9715968c6537.png)

这一节关于如何限制对 Netflix Eureka 服务器的访问就此结束。在下一节中，我们将学习如何使用 OAuth 2.0 和 OpenID Connect 对 API 的访问进行身份验证和授权。

# 使用 OAuth 2.0 和 OpenID Connect 对 API 访问进行身份验证和授权

在授权服务器就位之后，我们可以增强边缘服务器和`product-composite`服务，使它们成为 OAuth 2.0 资源服务器；也就是说，它们需要一个有效的访问令牌才能允许访问。我们将配置边缘服务器，接受任何可以使用授权服务器提供的签名验证的访问令牌。`product-composite`服务也将要求访问令牌包含一些 OAuth 2.0 范围：

+   `product:read`范围将用于访问只读 API。

+   `product:write`范围将用于访问创建和删除 API。

我们还必须增强我们的测试脚本，`test-em-all.bash`，以便在运行测试之前获取访问令牌。

# 边缘服务器和 product-composite 服务的变化

以下更改已应用：

+   `build.gradle`中已添加 Spring Security 5.1 依赖项，以支持 OAuth 2.0 资源服务器：

```java
implementation('org.springframework.boot:spring-boot-starter-security')
implementation('org.springframework.security:spring-security-oauth2-resource-server')
implementation('org.springframework.security:spring-security-oauth2-jose')
```

+   已经向`se.magnus.springcloud.gateway.SecurityConfig`和`se.magnus.microservices.composite.product.SecurityConfig`类添加了安全配置：

```java
@EnableWebFluxSecurity
public class SecurityConfig {

  @Bean
  SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
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

前面源代码的解释如下：

+   `.pathMatchers("/actuator/**").permitAll()`用于允许访问不应受保护的 URL，例如，此处的`actuator`端点。参阅源代码以了解被视为不受保护的 URL。小心哪些 URL 暴露为不受保护。例如，在上线之前，`actuator`端点应该受到保护：

    +   `.anyExchange().authenticated()`确保在允许访问所有其他 URL 之前，用户已进行身份验证。

    +   `.oauth2ResourceServer().jwt()`指定基于 JWT 编码的 OAuth 2.0 访问令牌进行身份验证和授权。

+   已经在配置文件`application.yml`中注册了授权服务器的`jwk-set`端点的地址：

```java
spring.security.oauth2.resourceserver.jwt.jwk-set-uri: http://${app.auth-server}:9999/.well-known/jwks.json
```

为了使边缘服务器和`product-composite`服务作为 OAuth 2.0 资源服务器行动，我们对它们进行了这些更改，同时还需对仅适用于`product-composite`服务进行一些更改。

# 产品组合服务中的更改

除了前面部分应用的常见更改外，还向`product-composite`服务应用了以下更改：

+   已经通过要求访问令牌中具有 OAuth 2.0 范围来细化安全配置，以允许访问：

```java
.pathMatchers(POST, "/product-composite/**").hasAuthority("SCOPE_product:write")
.pathMatchers(DELETE, "/product-composite/**").hasAuthority("SCOPE_product:write")
.pathMatchers(GET, "/product-composite/**").hasAuthority("SCOPE_product:read")
```

按照约定，在 Spring Security 中检查权限时，OAuth 2.0 范围应前缀为`SCOPE_`。

+   添加了一个方法，`logAuthorizationInfo()`，用于在每次调用 API 时记录 JWT 编码访问令牌的相关部分。可以使用标准 Spring Security，`SecurityContext`获取访问令牌，在反应式环境中，可以使用静态帮助方法`ReactiveSecurityContextHolder.getContext()`获取。有关详细信息，请参阅`se.magnus.microservices.composite.product.services.ProductCompositeServiceImpl`类。

+   在基于 Spring 的集成测试中，已禁用了 OAuth 的使用。为了防止在运行集成测试时 OAuth 机制启动，我们按照如下方式禁用它：

+   添加了一个安全配置`TestSecurityConfig`，用于在测试期间允许访问所有资源：

```java
http.csrf().disable().authorizeExchange().anyExchange().permitAll();
```

+   在每个 Spring 集成测试类中，我们配置了`TestSecurityConfig`以用以下内容覆盖现有的安全配置：

```java
@SpringBootTest( classes = 
{ProductCompositeServiceApplication.class, TestSecurityConfig.class },
 properties = {"spring.main.allow-bean-definition-overriding=true"})
```

有了这些更改，边缘服务器和`product-composite`服务都可以作为 OAuth 2.0 资源服务器。为了引入 OAuth 2.0 和 OpenID Connect 的使用，我们需要采取的最后一步是更新测试脚本，使其在运行测试时获取访问令牌并使用它们。

# 测试脚本中的更改

首先，我们需要在调用任何 API（除了健康 API）之前获取访问令牌，这使用 OAuth 2.0 密码流完成。为了能够调用创建和删除 API，我们以`writer`客户端的身份获取访问令牌，如下所示：

```java
ACCESS_TOKEN=$(curl -k https://writer:secret@$HOST:$PORT/oauth/token -d grant_type=password -d username=magnus -d password=password -s | jq .access_token -r)

```

为了验证基于范围的授权是否有效，在测试脚本中添加了两个测试：

+   第一个测试调用 API 时没有提供访问令牌。API 预期返回 401 未授权 HTTP 状态。

+   另一个测试使用`reader`客户端调用更新 API，该客户端仅授予读取范围。API 预期返回 403 禁止 HTTP 状态。

要查看完整的源代码，请参阅`test-em-all.bash`：

```java
# Verify that a request without access token fails on 401, Unauthorized
assertCurl 401 "curl -k https://$HOST:$PORT/product-composite/$PROD_ID_REVS_RECS -s"

# Verify that the reader - client with only read scope can call the read API but not delete API.
READER_ACCESS_TOKEN=$(curl -k https://reader:secret@$HOST:$PORT/oauth/token -d grant_type=password -d username=magnus -d password=password -s | jq .access_token -r)
 READER_AUTH="-H \"Authorization: Bearer $READER_ACCESS_TOKEN\""

assertCurl 200 "curl -k https://$HOST:$PORT/product-composite/$PROD_ID_REVS_RECS $READER_AUTH -s"
assertCurl 403 "curl -k https://$HOST:$PORT/product-composite/$PROD_ID_REVS_RECS $READER_AUTH -X DELETE -s"

```

更新测试脚本以获取和使用 OAuth 2.0 访问令牌后，我们准备在下一节中尝试它们！

# 使用本地授权服务器进行测试

在本节中，我们将尝试使用受保护的系统环境；也就是说，我们将一起测试所有的安全组件。我们将使用本地授权服务器来发放访问令牌。以下测试将执行：

1.  首先，我们从源代码构建并运行测试脚本，以确保一切都能配合在一起。

1.  接下来，我们将学习如何使用 OAuth 2.0 授权流获取访问令牌：密码、隐式和代码授权流。

1.  最后，我们将使用访问令牌来调用 API。我们还将验证，为读者客户端颁发的访问令牌不能用于调用更新 API。

# 构建和运行自动化测试

要构建和运行自动化测试，我们执行以下步骤：

1.  首先，使用以下命令构建 Docker 镜像：

```java
cd $BOOK_HOME/Chapter11
./gradlew build && docker-compose build
```

1.  接下来，使用以下命令启动 Docker 中的系统环境并运行常规测试：

```java
./test-em-all.bash start
```

注意最后的新增的负面测试，验证当我们没有认证时返回 401 未授权代码，以及当我们没有授权时返回 403 禁止。

# 获取访问令牌

现在我们可以使用 OAuth 2.0 定义的各种授权流来获取访问令牌。我们将尝试以下授权流：密码、隐式和代码授权。

# 使用密码授权流获取访问令牌

为了获取`writer`客户端的访问令牌，即具有`product:read`和`product:write`范围的访问令牌，请发出以下命令：

```java
curl -k https://writer:secret@localhost:8443/oauth/token -d grant_type=password -d username=magnus -d password=password -s | jq .
```

客户端使用 HTTP 基本认证来标识自己，传递其`writer`客户端 ID 和其秘密`secret`。它使用`username`和`password`参数发送资源所有者的凭据，即终端用户。

以下是一个示例响应：

![](img/aabd7ac5-4ce6-4f92-ae33-0393200e9116.png)

在响应中将`access_token`字段的值设置为环境变量中的访问令牌：

```java
ACCESS_TOKEN=eyJ...SyIlQ
```

要为`reader`客户端获取访问令牌，即只有`product:read`范围，只需在前面的命令中将`writer`替换为`reader`：

```java
curl -k https://reader:secret@localhost:8443/oauth/token -d grant_type=password -d username=magnus -d password=password -s | jq .
```

# 使用隐式授权流获取访问令牌

要使用隐式授权流获取访问令牌，我们需要涉及一个网络浏览器。打开接受自我签名的证书的网络浏览器，例如 Chrome。然后执行以下步骤：

1.  为了获取`reader`客户端的访问令牌，请打开以下 URL：`https://localhost:8443/oauth/authorize?response_type=token&client_id=reader&redirect_uri=http://my.redirect.uri&scope=product:read&state=48532`。当网络浏览器要求登录时，请使用授权服务器配置中指定的凭据，例如`magnus`和`password`：

![](img/8faa69da-b3b7-4720-90c3-894d5b351ca0.png)

1.  接下来，您将被要求授权`reader`客户端以您的名义调用 API：

![](img/db2552db-ce24-484a-9eae-48e8a08c4e70.png)

1.  最后，我们将得到以下响应：

![](img/fedc6ade-fe6a-4abb-839d-c654eea7bd1f.png)

这可能乍一看有点令人失望。授权服务器返回给网络浏览器的 URL 基于客户端在初始请求中指定的重定向-URI。将 URL 复制到文本编辑器中，您会发现类似于以下内容的东西：

`http://my.redirect.uri/#access_token=eyJh...C8pw&token_type=bearer&state=48532&expires_in=599999999&jti=8956b38b-36ea-4888-80dc-685b49f20f91`

太好了！我们可以在新 URL 中的`access_token`请求参数中找到访问令牌。

将访问令牌保存到环境变量中，如下所示：

```java
ACCESS_TOKEN=eyJh...C8pw
```

为了获取`writer`客户端的访问令牌，您可以使用以下 URL：`https://localhost:8443/oauth/authorize?response_type=token&client_id=writer&redirect_uri=http://my.redirect.uri&scope=product:read+product:write&state=95372`。

您已经认证，因此不需要再次登录，但您必须给予`writer`客户端同意。

注意，不需要客户端密钥；也就是说，隐式授权流并不非常安全。

任何应用程序都可以声称它是`writer`客户端，并可以要求用户同意使用请求的范围以用户的身份访问 API，所以要小心！

# 使用代码授权流获取访问令牌

最后，让我们尝试 OAuth 2.0 中最安全的授权流——代码授权流！

至于代码授权流，为了使流程更加安全，事情变得有点复杂。在第一个不安全步骤中，我们将使用网络浏览器获取一个只能使用一次的代码，当它与访问令牌交换时。代码应从网络浏览器传递到安全层，例如服务器端代码，它可以再次向授权服务器发起新请求以交换代码和访问令牌。在此交换中，服务器必须提供客户端密钥以验证其来源。

为了获取`reader`客户端的代码，请在网络浏览器中使用以下 URL：`https://localhost:8443/oauth/authorize?response_type=code&client_id=reader&redirect_uri=http://my.redirect.uri&scope=product:read&state=35725`。

这次，你将得到一个更短的 URL，例如，`http://my.redirect.uri/?code=T2pxvW&state=72489`。

从`code`参数中提取授权代码，并定义一个名为`CODE`的环境变量，其值为它的值：

```java
CODE=T2pxvW
```

接下来，假设你是后端服务器，使用以下`curl`命令将授权代码与访问令牌进行交换：

```java
curl -k https://reader:secret@localhost:8443/oauth/token \
 -d grant_type=authorization_code \
 -d client_id=reader \
 -d redirect_uri=http://my.redirect.uri \
 -d code=$CODE -s | jq .
```

示例响应如下：

![](img/b06c2376-bab9-452d-8081-fce44ce0b426.png)

最后，像之前一样将访问令牌保存到环境变量中：

```java
ACCESS_TOKEN=eyJh...KUBA
```

为了获取`writer`客户端的代码，请使用以下 URL：`https://localhost:8443/oauth/authorize?response_type=code&client_id=writer&redirect_uri=http://my.redirect.uri&scope=product:read+product:write&state=72489`[.](https://localhost:8443/oauth/authorize?response_type=code&client_id=writer&redirect_uri=http://my.redirect.uri&scope=message:read+message:write&state=72489)

# 使用访问令牌调用受保护的 API

现在，让我们使用我们已经获取的访问令牌来调用受保护的 API！

1.  首先，调用一个 API 来检索没有有效访问令牌的复合产品：

```java
ACCESS_TOKEN=an-invalid-token
curl https://localhost:8443/product-composite/2 -k -H "Authorization: Bearer $ACCESS_TOKEN" -i  
```

1.  它应该返回以下响应：

![](img/a11b7747-9134-464d-956b-8d9ea3ee632b.png)

错误信息清楚地指出访问令牌无效！

1.  接下来，尝试使用 API 检索复合产品，使用之前为`reader`客户端获取的其中一个访问令牌：

```java
ACCESS_TOKEN={a-reader-access-token}
curl https://localhost:8443/product-composite/2 -k -H "Authorization: Bearer $ACCESS_TOKEN" -i 
```

1.  现在我们将得到`200 OK`状态码，并将返回预期的响应体：

![](img/35945d10-f681-4265-99d7-8c71e352e889.png)

如果我们尝试使用为`reader`客户端获取的访问令牌访问更新 API，例如删除 API，调用将失败：

```java
ACCESS_TOKEN={a-reader-access-token}
curl https://localhost:8443/product-composite/999 -k -H "Authorization: Bearer $ACCESS_TOKEN" -X DELETE -i 
```

它将失败，响应类似于以下内容：

![](img/41ac8596-5ddd-4a60-a2f4-48f0ec0e69e9.png)

如果我们重复调用删除 API，但使用的是为`writer`客户端获取的访问令牌，那么调用将成功，响应为 200 OK。

删除操作即使 underlying database 中不存在具有指定 product ID 的产品，也应该返回`200`，因为删除操作如第六章中所述，是幂等的，*添加持久性*。参考*添加新 API*部分。

如果你使用`docker-compose logs -f product-composite`命令查看日志输出，你应该能够找到如下授权信息：

![](img/fa8061ea-8387-4bda-b2bd-ad5bab5f2230.png)

这些信息是通过`product-composite`服务中的新方法`logAuthorizationInfo()`从 JWT 编码的访问令牌中提取的；也就是说，`product-composite`服务不需要与授权服务器通信就能获取这些信息！

通过这些测试，我们看到了如何使用各种授权流获取访问令牌，即密码、隐式和代码授权流。我们还看到了如何使用范围来限制客户端可以使用特定访问令牌执行的操作，例如只用于阅读操作。

在下一节中，我们将用外部 OpenID Connect 提供商替换本节中使用的本地授权服务器。

# 使用 OpenID Connect 提供商进行测试 - Auth0

所以，与我们自己控制的授权服务器一起，OAuth 舞蹈运行良好。但如果我们用一个认证的 OpenID Connect 提供商来替换它呢？从理论上讲，它应该可以即插即用。让我们来找出答案，好吗？

要查看 OpenID Connect 的认证实现列表，请参阅[`openid.net/developers/certified/`](https://openid.net/developers/certified/)。我们将使用 Auth0 进行 OpenID 提供者的测试。[`auth0.com/`](https://auth0.com/)。为了能够使用 Auth0 而不是我们自己的授权服务器，我们将讨论以下主题：

+   在 Auth0 中设置带有 OAuth 客户端和用户的账户

+   应用所需更改以使用 Auth0 作为 OpenID 提供商并运行测试脚本来验证它是否正常工作

+   使用以下方法获取访问令牌：

    +   密码授权流

    +   隐式授权流

    +   授权代码授权流

+   使用访问令牌调用受保护的 API。

+   使用**用户信息**端点获取有关用户的更多信息。

让我们在下面的章节中理解每一个概念。

# 在 Auth0 中设置账户和 OAuth 2.0 客户端

执行以下步骤以在 Auth0 中注册免费账户，配置 OAuth 2.0 客户端和`product-composite` API，最后注册用户：

1.  在浏览器中打开 URL，[`auth0.com`](https://auth0.com)。

1.  点击 SIGN UP 按钮：

    1.  使用您选择的账户注册。

    1.  注册成功后，您将被要求创建一个租户域。

        输入您选择的租户名称，在我的情况下：`dev-ml.eu.auth0.com`。

    1.  填写有关您账户的信息。

1.  注册后，您将被引导到您的仪表板。在左侧选择应用程序标签，以查看在注册过程中为您创建的默认客户端应用程序。

1.  点击默认应用进行配置：

    1.  复制`Client ID`和`Client Secret`；稍后您需要它们。

    1.  作为应用程序类型，选择机器对机器。

    1.  作为令牌端点认证方法，选择 POST。

    1.  将`http://my.redirect.uri`作为允许的回调 URL 输入。

    1.  点击显示高级设置，转到授权类型标签，取消选中客户端凭据，选择密码框。

    1.  点击 SAVE CHANGES。

1.  现在为我们的 API 定义授权：

    1.  点击左侧的 APIs 标签，然后点击+ CREATE API 按钮。

    1.  将 API 命名为`product-composite`，给它一个标识符`https://localhost:8443/product-composite`，然后单击 CREATE 按钮。

    1.  点击“权限”标签页，为`product:read`和`product:write`创建两个权限（即，OAuth 范围）。

1.  接下来，创建一个用户：

    1.  点击“用户与角色”->“用户”标签页（在左侧），然后点击“创建您的第一个用户”按钮。

    1.  输入您喜欢的`email`和`password`，然后点击“保存”按钮。

    1.  寻找 Auth0 发送到您提供的电子邮件地址的邮箱中的验证邮件。

1.  最后，验证用于密码授予流的默认目录设置：

    1.  点击右上角的上传箭头旁边的租户配置文件，选择设置。

    1.  在名为“常规”的标签页中，滚动到名为“默认目录”的字段，并验证它包含`Username-Password-Authentication`值。如果没有，更新字段并保存更改。

1.  就这么多！请注意，默认应用和 API 都获得一个客户端 ID 和一个秘密。我们将使用默认应用的客户端 ID 和秘密；即 OAuth 客户端。

创建并配置了 Auth0 账户后，我们可以继续在系统架构中应用必要的配置更改。

# 将必要的更改应用于使用 Auth0 作为 OpenID 提供者

在本节中，我们将学习需要进行哪些配置更改，才能用 Auth0 替换本地授权服务器。我们只需要更改作为 OAuth 资源服务器的两个服务的配置，即`product-composite`和`gateway`服务。我们还需要稍微更改测试脚本，以便它从 Auth0 获取访问令牌，而不是从我们的本地授权服务器获取。让我们从 OAuth 资源服务器开始，即`product-composite`和`gateway`服务。

本主题中描述的更改**尚未**应用于书中 Git 仓库中的源代码；也就是说，您需要自己进行这些更改以亲眼看到它们的作用！

# 在 OAuth 资源服务器中更改配置

当使用 OpenID Connect 提供者时，我们只需要在 OAuth 资源服务器中配置基础 URI 到标准的发现端点，即`product-composite`和`gateway`服务。Spring Security 将使用来自发现端点的响应中的信息来配置资源服务器。

在`product-composite`和`gateway`项目中，对`resource/application.yml`文件进行以下更改：

现在找到以下属性设置：

```java
spring.security.oauth2.resourceserver.jwt.jwk-set-uri: http://${app.auth-server}:9999/.well-known/jwks.json
```

用以下内容替换：

```java
spring.security.oauth2.resourceserver.jwt.issuer-uri: https://${TENANT_DOMAIN_NAME}/

```

**注意**：用您的租户域名替换前一个配置中的`${TENANT_DOMAIN_NAME}`；在我的情况下，它是`dev-ml.eu.auth0.com`，不要忘记结尾的`/`！

如果您好奇，可以通过运行以下命令查看发现文档中的内容：

`curl https://${TENANT_DOMAIN_NAME}/.well-known/openid-configuration -s | jq`

按照以下方式重新构建`product-composite`和`gateway`服务：

```java
cd $BOOK_HOME/Chapter11
./gradlew build && docker-compose up -d --build product-composite gateway
```

更新了`product-composite`和`gateway`服务后，我们还可以继续更新测试脚本。

# 更改测试脚本以从 Auth0 获取访问令牌

我们还需要更新测试脚本，使其从 Auth0 OIDC 提供者获取访问令牌。这可以通过在`test-em-all.bash`中执行以下更改来实现。

使用以下命令：

```java
ACCESS_TOKEN=$(curl http://writer:secret@$HOST:$PORT/oauth/token -d grant_type=password -d username=magnus -d password=password -s | jq .access_token -r)
```

用以下命令替换它：

```java
ACCESS_TOKEN=$(curl --request POST \
 --url 'https://${TENANT_DOMAIN_NAME}/oauth/token' \
 --header 'content-type: application/json' \
 --data '{"grant_type":"password", "username":"${USER_EMAIL}", "password":"${USER_PASSWORD}", "audience":"https://localhost:8443/product-composite", "scope":"openid email product:read product:write", "client_id": "${CLIENT_ID}", "client_secret": "${CLIENT_SECRET}"}' -s | jq -r .access_token)
```

现在，用之前在 Auth0 注册过程中收集的值替换`${TENANT_DOMAIN_NAME}`、`${USER_EMAIL}`、`${USER_PASSWORD}`、`${CLIENT_ID}`和`${CLIENT_SECRET}`，然后使用以下命令：

```java
READER_ACCESS_TOKEN=$(curl -k https://reader:secret@$HOST:$PORT/oauth/token -d grant_type=password -d username=magnus -d password=password -s | jq .access_token -r)
```

用以下命令替换它：

```java
READER_ACCESS_TOKEN=$(curl --request POST \
 --url 'https://${TENANT_DOMAIN_NAME}/oauth/token' \
 --header 'content-type: application/json' \
 --data '{"grant_type":"password", "username":"${USER_EMAIL}", "password":"${USER_PASSWORD}", "audience":"https://localhost:8443/product-composite", "scope":"openid email product:read", "client_id": "${CLIENT_ID}", "client_secret": "${CLIENT_SECRET}"}' -s | jq -r .access_token)
```

将前面的更改应用到命令中。同时注意我们只要求`product:read`作用域，而不需要`product:write`作用域。这是为了模拟只具有读取权限的客户端。

现在，访问令牌是由 Auth0 签发，而不是我们的本地授权服务器，我们的 API 实现可以验证访问令牌（是否由 Auth0 正确签名且未过期），使用`application.yml`文件中标记的 Auth0 发现服务提供的信息。与之前一样，API 实现可以使用访问令牌中的作用域来授权客户端调用 API 或不允许调用。

现在我们已经实施了所有必需的更改，让我们运行一些测试来验证我们是否可以从 Auth0 获取访问令牌。

# 使用 Auth0 作为 OpenID Connect 提供者运行测试脚本

现在，我们准备尝试 Auth0！

使用以下命令对 Auth0 进行常规测试：

```java
./test-em-all.bash
```

在日志中（使用`docker-compose logs -f product-composite`命令），你将能够找到 Auth0 签发的访问令牌的授权信息：

使用同时带有`product:read`和`product:write`作用域的访问令牌时，我们会看到两个作用域都按如下列出：

![](img/9c98cb07-9e00-4611-8c25-0db0aa384d1d.png)

仅使用带有`product:read`作用域的访问令牌时，我们会看到只列出该作用域如下：

![](img/568e2fc6-35fa-43e1-8247-1d2b9cad167f.png)

从日志输出中，我们可以看到我们现在得到了关于这个访问令牌打算面向的受众的信息。为了加强安全，我们可以在我们的服务中添加一个测试，验证其 URL，在本例中为`https://localhost:8443/product-composite`，是否是受众列表的一部分。这将防止有人尝试使用为其他目的签发的访问令牌来获取对我们的 API 的访问。

有了与 Auth0 集成的自动化测试，我们可以继续学习如何使用不同类型的授权流获取访问令牌。让我们从密码授权流开始。

# 使用密码授权流获取访问令牌

在本节中，我们将学习如何使用密码授权流从 Auth0 获取访问令牌。

如果你自己想要从 Auth0 获取访问令牌，你可以通过运行以下命令来实现：

```java
curl --request POST \
 --url 'https://${TENANT_DOMAIN_NAME}/oauth/token' \
 --header 'content-type: application/json' \
 --data '{"grant_type":"password", "username":"${USER_EMAIL}", "password":"${USER_PASSWORD}", "audience":"https://localhost:8443/product-composite", "scope":"openid email product:read", "client_id": "${CLIENT_ID}", "client_secret": "${CLIENT_SECRET}"}' -s | jq
```

根据*使用访问令牌调用受保护的 API*节中的说明，你应该能够使用获取的访问令牌调用 API。我们接下来要尝试的授权流程是隐式授权流程。

# 使用隐式授权流程获取访问令牌

在本节中，我们将学习如何使用隐式授权流程从 Auth0 获取访问令牌。

如果你想要尝试更复杂的隐式授权流程，你可以在网页浏览器中打开下面的链接：

```java
https://${TENANT_DOMAIN_NAME}/authorize?response_type=token&scope=openid email product:read product:write&client_id=${CLIENT_ID}&state=98421&&nonce=jxdlsjfi0fa&redirect_uri=http://my.redirect.uri&audience=https://localhost:8443/product-composite
```

将前面 URL 中的`${TENANT_DOMAIN_NAME}`和`${CLIENT_ID}`替换为你在 Auth0 注册过程中收集的租户域名和客户端 ID。

让我们看看以下步骤：

1.  Auth0 应该显示以下登录屏幕：

![](img/51fa7e05-b536-4ae8-8108-403cb016fa7f.png)

1.  登录成功后，Auth0 会要求你给予客户端应用程序你的同意：

![](img/88af32b9-cb29-4cea-afba-1e337b85d14f.png)

访问令牌现在在浏览器中的 URL 里，就像我们在本地授权服务器中尝试隐式流程时一样：

![](img/2124688f-d39d-431b-982b-6f5f786ac8d7.png)

要获取对应于`reader`客户端的访问令牌，需要从前面的 URL 中移除`product:write`范围，这个 URL 是我们用来启动隐式授权流程的。

既然我们已经知道如何使用隐式授权流程获取访问令牌，我们可以继续尝试第三个也是最后一个授权流程，即授权码授权流程。

# 使用授权码授权流程获取访问令牌

最后，我们来到了最安全的授权流程——授权码授权流程。我们将遵循与本地授权服务器相同的程序；也就是说，我们首先获取代码，然后用它来换取访问令牌。通过在网页浏览器中打开下面的链接来获取代码，如下所示：

```java
https://${TENANT_DOMAIN_NAME}/authorize?audience=https://localhost:8443/product-composite&scope=openid email product:read product:write&response_type=code&client_id=${CLIENT_ID}&redirect_uri=http://my.redirect.uri&state=845361
```

将前面 URL 中的`${TENANT_DOMAIN_NAME}`和`${CLIENT_ID}`替换为你在 Auth0 注册过程中收集的租户域名和客户端 ID。

期望在网页浏览器中会尝试重定向到一个如下所示的 URL：

```java
http://my.redirect.uri/?code=6mQ7HK--WyX9fMnv&state=845361
```

提取代码并运行以下命令以获取访问令牌：

```java
curl --request POST \
 --url 'https://${TENANT_DOMAIN_NAME}/oauth/token' \
 --header 'content-type: application/json' \
 --data '{"grant_type":"authorization_code","client_id": "${CLIENT_ID}","client_secret": "${CLIENT_SECRET}","code": "${CODE}","redirect_uri": "http://my.redirect.uri"}' -s | jq .
```

将前面 URL 中的`${TENANT_DOMAIN_NAME}`、`${CLIENT_ID}`、`${CLIENT_SECRET}`和`${CODE}`替换为你在 Auth0 注册过程中收集的租户域名、客户端 ID 和客户端密钥。

既然我们已经学会了使用三种类型的授权流程获取访问令牌，我们准备在下一节中尝试使用从 Auth0 获取的访问令牌调用外部 API。

# 使用 Auth0 访问令牌调用受保护的 API

在本节中，我们将了解如何使用从 Auth0 获取的访问令牌调用外部 API。

我们可以使用 Auth0 发行的访问令牌来调用我们的 API，就像我们之前使用本地授权服务器发行的访问令牌一样。

对于只读 API，执行以下命令：

```java
ACCESS_TOKEN=...
curl https://localhost:8443/product-composite/2 -k -H "Authorization: Bearer $ACCESS_TOKEN" -i  
```

对于一个更新的 API，执行以下命令：

```java
ACCESS_TOKEN=...
curl https://localhost:8443/product-composite/999 -k -H "Authorization: Bearer $ACCESS_TOKEN" -X DELETE -i 
```

因为我们请求了两个范围，`product:read`和`product:write`，所以预计前面的 API 调用都会返回 200 OK。

# 获取关于用户的其他信息

正如你在日志输出中看到的，主题（即用户）的 ID 有点难以理解，例如，`auth0|5ca0b73c97f31e11bc85a5e6`。

如果你想要你的 API 实现了解更多关于用户的信息，它可以调用 Auth0 的`userinfo_endpoint`，如前所述的发现请求的响应：

```java
curl -H "Authorization: Bearer $ACCESS_TOKEN" https://${TENANT_DOMAIN_NAME}/userinfo -s | jq
```

在前面的命令中将`${TENANT_DOMAIN_NAME}`替换为你之前在 Auth0 注册过程中收集的租户域名。

以下是一个示例响应：

![](img/85766cca-2c09-4b6a-9ead-a450d32154af.png)

此端点还可以用于验证用户是否在 Auth0 中撤销了访问令牌。

通过以下命令关闭系统架构，结束测试：

```java
docker-compose down 
```

到这里，我们已经学会了如何用一个外部的替代品替换掉只用于测试的本地 OAuth 2.0 授权服务器。我们还看到了如何重新配置微服务架构，使用外部的 OIDC 提供者来验证访问令牌。

# 总结

在本章中，我们学习了如何使用 Spring Security 保护我们的 API。

我们已经看到了如何通过 Spring Security 启用 HTTPS 来防止第三方监听。通过 Spring Security，我们还了解到，限制访问和发现服务器 Netflix Eureka 非常简单，使用 HTTP 基本认证。最后，我们看到了如何使用 Spring Security 简化 OAuth 2.0 和 OpenID Connect 的使用，允许第三方客户端应用程序以用户的名义访问我们的 API，而无需要求用户与客户端应用程序共享凭据。我们已经学习了如何基于 Spring Security 设置本地 OAuth 2.0 授权服务器，以及如何更改配置，以便使用外部的 OpenID Connect 提供者 Auth0。

然而，一个关注点是如何管理所需的配置。对于涉及其中的微服务，必须设置许多小的配置项，并且这些配置必须同步以匹配。除了分散的配置之外，还有一些配置包含敏感信息，例如凭据或证书。看来我们需要一种更好的方法来处理多个协作微服务的配置，同时也需要一个解决方案来处理配置中的敏感部分。

在下一章中，我们将探索 Spring Cloud Configuration 服务器，并了解如何使用它来处理这些类型的需求。

# 问题

1.  使用自签名证书的好处和缺点是什么？

1.  OAuth 2.0 授权码的目的是什么？

1.  什么是 OAuth 2.0 范围的目的？

1.  当一个令牌是一个 JWT 令牌时，这意味着什么？

1.  我们如何信任存储在 JWT（JSON Web Token）令牌中的信息？

1.  对于原生移动应用，使用 OAuth 2.0 授权码授权流是否合适？

1.  开放 ID 连接（OpenID Connect）为 OAuth 2.0 增加了什么？
