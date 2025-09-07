# 12

# 集中配置

在本章中，我们将学习如何使用**Spring Cloud Config Server**来集中管理我们的微服务的配置。正如已在*第一章*，*微服务简介*中所述，微服务数量的增加通常伴随着需要管理和更新的配置文件数量的增加。

使用 Spring Cloud Config Server，我们可以将所有微服务的配置文件放置在一个中央配置仓库中，这将使处理它们变得更加容易。我们的微服务将在启动时从配置服务器检索其配置。

本章将涵盖以下主题：

+   Spring Cloud Config Server 简介

+   设置配置服务器

+   配置配置服务器的客户端

+   配置仓库的结构化

+   尝试使用 Spring Cloud Config Server

# 技术要求

关于如何安装本书中使用的工具以及如何访问本书源代码的说明，请参阅：

+   *第二十一章*，*macOS 安装说明*

+   *第二十二章*，*使用 WSL 2 和 Ubuntu 的 Microsoft Windows 安装说明*

本章中的代码示例均来自`$BOOK_HOME/Chapter12`中的源代码。

如果您想查看对本章源代码所做的更改，即查看将配置服务器添加到微服务景观中所需要的内容，您可以将其与*第十一章*，*API 访问保护*的源代码进行比较。您可以使用您喜欢的`diff`工具，并比较两个文件夹`$BOOK_HOME/Chapter11`和`$BOOK_HOME/Chapter12`。

# Spring Cloud Config Server 简介

Spring Cloud Config Server（简称**Config Server**）将被添加到边缘服务器后面的现有微服务景观中，与其他微服务一样：

![](img/B19825_12_01.png)

图 12.1：将配置服务器添加到系统架构中

在设置配置服务器时，有许多选项需要考虑：

+   选择配置仓库的存储类型

+   决定初始客户端连接，无论是到配置服务器还是到发现服务器

+   保护配置，既防止对 API 的无权访问，也避免在配置仓库中存储敏感信息为纯文本

让我们逐一介绍每个选项，并介绍配置服务器公开的 API。

## 选择配置仓库的存储类型

如已在*第八章*，*Spring Cloud 简介*中所述，配置服务器支持在多种不同的后端存储配置文件，例如：

+   Git 仓库

+   本地文件系统

+   HashiCorp Vault

+   JDBC 数据库

在本章中，我们将使用本地文件系统。要使用本地文件系统，配置服务器需要以启用原生 Spring 配置文件的方式启动。配置存储库的位置使用`spring.cloud.config.server.native.searchLocations`属性指定。

## 决定初始客户端连接

默认情况下，客户端首先连接到配置服务器以检索其配置。根据配置，它连接到发现服务器（在我们的例子中是 Netflix Eureka）以注册自己。也可以反过来这样做，即客户端首先连接到发现服务器以找到配置服务器实例，然后连接到配置服务器以获取其配置。这两种方法都有其优缺点。

在本章中，客户端将首先连接到配置服务器。采用这种方法，可以将发现服务器的配置存储在配置服务器中。

要了解更多关于其他替代方案的信息，请参阅[`docs.spring.io/spring-cloud-config/docs/4.0.1/reference/html/#discovery-first-bootstrap`](https://docs.spring.io/spring-cloud-config/docs/4.0.1/reference/html/#discovery-first-bootstrap)。

首先连接到配置服务器的一个担忧是配置服务器可能成为单点故障。如果客户端首先连接到一个发现服务器，例如 Netflix Eureka，则可以有多个配置服务器实例注册，从而避免单点故障。在本书的后面部分，当我们学习 Kubernetes 中的**服务**概念时，从*第十五章*，*Kubernetes 简介*开始，我们将看到如何通过在每个 Kubernetes 服务后面运行多个容器（例如，配置服务器）来避免单点故障。

## 配置安全

通常，配置信息将被视为敏感信息。这意味着我们需要在传输和静止状态下保护配置信息。从运行时角度来看，配置服务器不需要通过边缘服务器暴露给外部。然而，在开发期间，能够访问配置服务器的 API 以检查配置是有用的。在生产环境中，建议限制对配置服务器的外部访问。

### 传输中的配置安全

当微服务或使用配置服务器 API 的任何人请求配置信息时，由于它已经使用 HTTPS，因此它将由边缘服务器保护，防止窃听。

为了确保 API 用户是已知的客户端，我们将使用 HTTP 基本身份验证。我们可以在配置服务器中使用 Spring Security 来设置 HTTP 基本身份验证，并指定环境变量`SPRING_SECURITY_USER_NAME`和`SPRING_SECURITY_USER_PASSWORD`，以使用允许的凭据。

### 静止状态下的配置安全

为了避免有人可以访问配置存储库并窃取敏感信息（如密码）的情况，配置服务器支持在磁盘上存储配置信息的加密。配置服务器支持使用对称密钥和非对称密钥。非对称密钥更安全，但更难管理。

在本章中，我们将使用对称密钥。对称密钥在启动时通过指定环境变量`ENCRYPT_KEY`传递给配置服务器。加密密钥只是一个需要像任何敏感信息一样进行保护的纯文本字符串。

要了解更多关于非对称密钥的使用，请参阅[`docs.spring.io/spring-cloud-config/docs/4.0.1/reference/html/#_key_management`](https://docs.spring.io/spring-cloud-config/docs/4.0.1/reference/html/#_key_management)。

## 介绍配置服务器 API

配置服务器公开了一个 REST API，其客户端可以使用它来检索其配置。在本章中，我们将使用 API 中的以下端点：

+   `/actuator`：所有微服务公开的标准 actuator 端点。像往常一样，应该小心使用。它们在开发期间非常有用，但在用于生产之前必须被锁定。

+   `/encrypt`和`/decrypt`：加密和解密敏感信息的端点。在使用生产环境之前，这些也必须被锁定。

+   `/{microservice}/{profile}`：返回指定微服务和指定 Spring 配置的配置。

当我们尝试配置服务器时，我们将看到 API 的一些示例用法。

# 设置配置服务器

在讨论的基础上设置配置服务器非常简单：

1.  使用 Spring Initializr 创建 Spring Boot 项目，如*第三章*中所述，*创建一组协作微服务*。请参阅*使用 Spring Initializr 生成骨架代码*部分。

1.  将依赖项`spring-cloud-config-server`和`spring-boot-starter-security`添加到 Gradle 构建文件`build.gradle`中。

1.  在应用程序类`ConfigServerApplication`中添加注解`@EnableConfigServer`：

    ```java
    @EnableConfigServer
    @SpringBootApplication
    public class ConfigServerApplication { 
    ```

1.  将配置服务器的配置添加到默认属性文件`application.yml`中：

    ```java
    server.port: 8888
    spring.cloud.config.server.native.searchLocations: file:${PWD}/config-repo
    management.endpoint.health.show-details: "ALWAYS"
    management.endpoints.web.exposure.include: "*"
    logging:
      level:
        root: info
    ---
    spring.config.activate.on-profile: docker
    spring.cloud.config.server.native.searchLocations: file:/config-repo 
    ```

    最重要的配置是指定配置存储库的位置，使用`spring.cloud.config.server.native.searchLocations`属性表示。

1.  向边缘服务器添加路由规则，以便从微服务景观外部访问配置服务器的 API。

1.  将 Dockerfile 和配置服务器的定义添加到三个 Docker Compose 文件中。

1.  将敏感配置参数外部化到标准的 Docker Compose 环境文件`.env`中。这些参数在*配置服务器以 Docker 使用*部分中描述。

1.  将配置服务器添加到通用构建文件`settings.gradle`中：

    ```java
    include ':spring-cloud:config-server' 
    ```

Spring Cloud Config 服务器源代码可在 `$BOOK_HOME/Chapter12/spring-cloud/config-server` 中找到。

现在，让我们看看如何设置第 *5* 步中提到的路由规则，以及如何配置在 Docker Compose 中添加的配置服务器，如第 *6* 步和 *7* 步所述。

## 在边缘服务器上设置路由规则

为了能够从微服务景观外部访问配置服务器的 API，我们在边缘服务器中添加了一个路由规则。所有以 `/config` 开头的对边缘服务器的请求都将通过以下路由规则路由到配置服务器：

```java
 - id: config-server
   uri: http://${app.config-server}:8888
  predicates:
  - Path=/config/**
  filters:
  - RewritePath=/config/(?<segment>.*), /$\{segment} 
```

路由规则中的 `RewritePath` 过滤器将在将其发送到配置服务器之前，从传入的 URL 中删除前面的部分 `/config`。

边缘服务器也被配置为允许所有对配置服务器的请求，将安全检查委托给配置服务器。以下行被添加到边缘服务器的 `SecurityConfig` 类中：

```java
 .pathMatchers("/config/**").permitAll() 
```

在此路由规则到位后，我们可以使用配置服务器的 API；例如，运行以下命令以请求使用 `docker` Spring 配置文件的 `product` 服务的配置：

```java
curl https://dev-usr:dev-pwd@localhost:8443/config/product/docker -ks | jq 
```

当我们稍后尝试配置服务器时，将运行此命令。

## 配置配置服务器以与 Docker 一起使用

配置服务器的 Dockerfile 与其他微服务的 Dockerfile 相同，只是它暴露的是端口 `8888` 而不是端口 `8080`。

当涉及到将配置服务器添加到 Docker Compose 文件中时，它与我们所看到的其他微服务略有不同：

```java
config-server:
  build: spring-cloud/config-server
  mem_limit: 512m
  environment:
    - SPRING_PROFILES_ACTIVE=docker,native
    - ENCRYPT_KEY=${CONFIG_SERVER_ENCRYPT_KEY}
    - SPRING_SECURITY_USER_NAME=${CONFIG_SERVER_USR}
    - SPRING_SECURITY_USER_PASSWORD=${CONFIG_SERVER_PWD}
  volumes:
    - $PWD/config-repo:/config-repo 
```

下面是对前面源代码的解释：

1.  Spring 配置文件 `native` 被添加，以向配置服务器发出信号，表明配置存储库基于本地文件。

1.  环境变量 `ENCRYPT_KEY` 用于指定配置服务器将用于加密和解密敏感配置信息的对称加密密钥。

1.  环境变量 `SPRING_SECURITY_USER_NAME` 和 `SPRING_SECURITY_USER_PASSWORD` 用于指定用于保护使用基本 HTTP 认证的 API 的凭据。

1.  `volumes` 声明将使 `config-repo` 文件夹在 Docker 容器中可通过 `/config-repo` 访问。

三个前面环境变量的值，在 Docker Compose 文件中以 `${...}` 标记，由 Docker Compose 从 `.env` 文件中获取：

```java
CONFIG_SERVER_ENCRYPT_KEY=my-very-secure-encrypt-key
CONFIG_SERVER_USR=dev-usr
CONFIG_SERVER_PWD=dev-pwd 
```

存储在 `.env` 文件中的信息，即用户名、密码和加密密钥，是敏感的，如果用于除开发和测试之外的其他用途，则必须受到保护。此外，请注意，丢失加密密钥将导致配置存储库中的加密信息无法解密的情况发生！

# 配置配置服务器的客户端

为了能够从配置服务器获取它们的配置，我们的微服务需要更新。这可以通过以下步骤完成：

1.  将 `spring-cloud-starter-config` 和 `spring-retry` 依赖项添加到 Gradle 构建文件 `build.gradle` 中。

1.  将配置文件 `application.yml` 移动到配置存储库，并使用 `spring.application.name` 属性指定的客户端名称重命名。

1.  将新的 `application.yml` 文件添加到 `src/main/resources` 文件夹中。此文件将用于存储连接到配置服务器所需的配置。有关其内容的说明，请参阅以下 *配置连接信息* 部分。

1.  将访问配置服务器的凭据添加到 Docker Compose 文件中，例如 `product` 服务：

    ```java
    product:
      environment:
     - CONFIG_SERVER_USR=${CONFIG_SERVER_USR}
     - CONFIG_SERVER_PWD=${CONFIG_SERVER_PWD} 
    ```

1.  禁用运行基于 Spring Boot 的自动化测试时使用配置服务器。这是通过将 `spring.cloud.config.enabled=false` 添加到 `@DataMongoTest`、`@DataJpaTest` 和 `@SpringBootTest` 注解来完成的。它们看起来像这样：

    ```java
    @DataMongoTest(properties = {"spring.cloud.config.enabled=false"})
    @DataJpaTest(properties = {"spring.cloud.config.enabled=false"})
    @SpringBootTest(webEnvironment=RANDOM_PORT, properties = {"eureka.client.enabled=false", "spring.cloud.config.enabled=false"}) 
    ```

## 配置连接信息

如前所述，`src/main/resources/application.yml` 文件现在包含连接到配置服务器所需的客户端配置。此文件对所有配置服务器的客户端具有相同的内容，除了由 `spring.application.name` 属性指定的应用程序名称（在以下示例中设置为 `product`）：

```java
spring.config.import: "configserver:"
spring:
  application.name: product
  cloud.config:
    failFast: true
    retry:
      initialInterval: 3000
      multiplier: 1.3
      maxInterval: 10000
      maxAttempts: 20
    uri: http://localhost:8888
    username: ${CONFIG_SERVER_USR}
    password: ${CONFIG_SERVER_PWD}
---
spring.config.activate.on-profile: docker
spring.cloud.config.uri: http://config-server:8888 
```

此配置将使客户端执行以下操作：

1.  当它运行在 Docker 外部时，使用 `http://localhost:8888` URL 连接到配置服务器，当在 Docker 容器中运行时，使用 `http://config-server:8888` URL

1.  使用基于 `CONFIG_SERVER_USR` 和 `CONFIG_SERVER_PWD` 属性值的 HTTP 基本身份验证作为客户端的用户名和密码

1.  如果需要，在启动期间尝试最多 20 次重新连接到配置服务器

1.  如果连接尝试失败，客户端将最初等待 3 秒钟后尝试重新连接

1.  后续重试的等待时间将增加 1.3 倍

1.  连接尝试之间的最大等待时间为 10 秒

1.  如果客户端在 20 次尝试后仍然无法连接到配置服务器，其启动将失败

此配置通常适用于对配置服务器的临时连接问题具有弹性。当整个微服务及其配置服务器同时启动时，例如使用 `docker-compose up` 命令时，特别有用。在这种情况下，许多客户端将在配置服务器准备好之前尝试连接到它，而 `retry` 逻辑将确保客户端在配置服务器启动并运行后成功连接。

# 结构化配置存储库

在将配置文件从每个客户端的源代码移动到配置存储库后，许多配置文件中都将有一些共同配置，例如，对于 actuator 端点的配置以及如何连接到 Eureka、RabbitMQ 和 Kafka。

常用部分已放置在名为`application.yml`的通用配置文件中。此文件由所有客户端共享。配置存储库包含以下文件：

```java
config-repo/
├── application.yml
├── auth-server.yml
├── eureka-server.yml
├── gateway.yml
├── product-composite.yml
├── product.yml
├── recommendation.yml
└── review.yml 
```

配置存储库位于`$BOOK_HOME/Chapter12/config-repo`。

# 尝试 Spring Cloud Config Server

现在是尝试配置服务器的时候了：

+   首先，我们将从源代码构建并运行测试脚本，以确保一切正常。

+   接下来，我们将尝试使用配置服务器 API 检索我们的微服务的配置。

+   最后，我们将了解如何加密和解密敏感信息，例如密码。

## 构建和运行自动化测试

因此，现在我们构建并运行系统景观的验证测试，如下所示：

1.  使用以下命令构建 Docker 镜像：

    ```java
    cd $BOOK_HOME/Chapter12
    ./gradlew build && docker-compose build 
    ```

1.  在 Docker 中启动系统景观并使用以下命令运行常规测试：

    ```java
    ./test-em-all.bash start 
    ```

## 使用配置服务器 API 获取配置

如前所述，我们可以通过使用 URL 前缀`/config`通过边缘服务器访问配置服务器的 API。我们还需要提供`.env`文件中指定的凭据，以进行 HTTP 基本身份验证。例如，要检索以 Docker 容器形式运行的`product`服务所使用的配置，即激活了 Spring 配置文件`docker`，请运行以下命令：

```java
curl https://dev-usr:dev-pwd@localhost:8443/config/product/docker -ks | jq . 
```

预期以下结构的响应（响应中的许多属性已被`...`替换以提高可读性）：

```java
{
  "name": "product",
  "profiles": [
    "docker"
  ],
  ...
  "propertySources": [
    {
      "name": "...file [/config-repo/product.yml]...",
      "source": {
        "spring.config.activate.on-profile": "docker",
        "server.port": 8080,
        ...
      }
    },
    {
      "name": "...file [/config-repo/product.yml]...",
      "source": {
        "server.port": 7001,
        ...
      }
    },
    {
      "name": "...file [/config-repo/application.yml]...",
      "source": {
        "spring.config.activate.on-profile": "docker",
        ...
      }
    },
    {
      "name": "...file [/config-repo/application.yml]...",
      "source": {
        ...
        "app.eureka-password": "p",
        "spring.rabbitmq.password": "guest"
      }
    }
  ]
} 
```

对于此响应的解释如下：

+   响应包含来自多个**属性源**的属性，每个属性文件和与 API 请求匹配的 Spring 配置文件一个。属性源按优先级返回；如果多个属性源中指定了相同的属性，则响应中的第一个属性具有优先权。前面的示例响应包含以下属性源，按以下优先级顺序：

    +   `/config-repo/product.yml`，对于`docker` Spring 配置文件

    +   `/config-repo/product.yml`，对于`default` Spring 配置文件

    +   `/config-repo/application.yml`，对于`docker` Spring 配置文件

    +   `/config-repo/application.yml`，对于`default` Spring 配置文件

    例如，使用的端口将是`8080`而不是`7001`，因为在先前的响应中`"server.port": 8080`在`"server.port": 7001`之前指定。

+   敏感信息，例如 Eureka 和 RabbitMQ 的密码，以纯文本形式返回，例如`"p"`和`"guest"`，但它们在磁盘上被加密。在配置文件`application.yml`中，它们被指定如下：

    ```java
    app:
      eureka-password:
    '{cipher}bf298f6d5f878b342f9e44bec08cb9ac00b4ce57e98316f030194a225fac89fb'
    spring.rabbitmq:
      password: '{cipher}17fcf0ae5b8c5cf87de6875b699be4a1746dd493a99d926c7a26a68c422117ef' 
    ```

## 加密和解密敏感信息

可以使用配置服务器公开的`/encrypt`和`/decrypt`端点来加密和解密信息。`/encrypt`端点可以用来创建要放置在配置存储库中的`property`文件中的加密值。参考前一个示例，其中 Eureka 和 RabbitMQ 的密码以加密形式存储在磁盘上。`/decrypt`端点可以用来验证存储在配置存储库磁盘上的加密信息。

1.  要加密`hello world`字符串，请运行以下命令：

    ```java
    curl -k https://dev-usr:dev-pwd@localhost:8443/config/encrypt --data-urlencode "hello world" 
    ```

    当使用`curl`调用`/encrypt`端点时，使用`--data-urlencode`标志是很重要的，以确保正确处理特殊字符，如`+`。

    预期响应如下：

![图形用户界面，文本，应用程序  自动生成的描述](img/B19825_12_02.png)

图 12.2：配置参数的加密值

1.  要解密加密值，请运行以下命令：

    ```java
    curl -k https://dev-usr:dev-pwd@localhost:8443/config/decrypt -d d91001603dcdf3eb1392ccbd40ff201cdcf7b9af2fcaab3da39e37919033b206 
    ```

    预期响应为`hello world`字符串：

![图片](img/B19825_12_03.png)

图 12.3：配置参数的解密值

如果你想在配置文件中使用加密值，你需要在其前加上`{cipher}`并在`''`中包裹它。例如，要存储`hello world`的加密版本，在基于 YAML 的配置文件中添加以下行：

```java
my-secret: '{cipher}d91001603dcdf3eb1392ccbd40ff201cdcf7b9af2 fcaab3da39e37919033b206' 
```

当配置服务器检测到格式为`'{cipher}...'`的值时，它在将它们发送到客户端之前会尝试使用其加密密钥来解密它们。

1.  这些测试完成了关于集中配置的章节。通过关闭系统景观来结束：

    ```java
    docker-compose down 
    ```

# 摘要

在本章中，我们看到了如何使用 Spring Cloud Config Server 来集中管理我们的微服务的配置。我们可以将配置文件放置在公共配置存储库中，并在单个配置文件中共享公共配置，同时将特定于微服务的配置保留在特定于微服务的配置文件中。微服务已更新，以便在启动时从配置服务器检索其配置，并且配置为在从配置服务器检索配置时处理暂时中断。

配置服务器可以通过要求使用 HTTP 基本身份验证来认证其 API 的使用，从而保护配置信息，并且可以通过通过使用 HTTPS 的边缘服务器公开其 API 来防止窃听。为了防止入侵者从磁盘上的配置文件中获取访问权限，我们可以使用配置服务器的`/encrypt`端点来加密信息，并将其加密存储在磁盘上。

在开发期间公开配置服务器的 API 是有用的，但在生产使用之前应该将其锁定。

在下一章中，我们将学习如何使用**Resilience4j**来减轻过度使用微服务之间同步通信的潜在缺点。

# 问题

1.  在启动期间，我们可以期望审查服务向配置服务器发出哪些 API 调用以检索其配置？

1.  使用以下命令启动了审查服务：`docker compose up -d`。

    使用以下命令调用配置服务器时，我们应该期望返回哪些配置信息？

    ```java
    curl https://dev-usr:dev-pwd@localhost:8443/config/application/default -ks | jq 
    ```

1.  Spring Cloud Config 支持哪些类型的存储后端？

1.  我们如何使用 Spring Cloud Config Server 在磁盘上加密敏感信息？

1.  我们如何保护配置服务器 API 免受滥用？

1.  提及与首先连接到配置服务器相比，那些首先连接到发现服务器的客户端的一些优缺点。
