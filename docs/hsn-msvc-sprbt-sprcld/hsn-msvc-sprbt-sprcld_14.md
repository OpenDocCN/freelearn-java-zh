# 第十二章：集中式配置

在本章中，我们将学习如何使用 Spring Cloud Configuration 服务器来集中管理我们微服务的配置。正如在第一章“微服务简介”中“集中配置”部分所描述的，越来越多的微服务通常伴随着越来越多的需要管理和更新的配置文件。

使用 Spring Cloud Configuration 服务器，我们可以将所有微服务的配置文件放在一个中心配置存储库中，这将使我们更容易管理它们。我们的微服务将在启动时从配置服务器检索其配置。

本章将涵盖以下主题：

+   介绍 Spring Cloud Configuration 服务器

+   设置配置服务器

+   配置配置服务器的客户端

+   组织配置仓库

+   尝试使用 Spring Cloud Configuration 服务器

# 技术要求

本书中描述的所有命令都是在 MacBook Pro 上使用 macOS Mojave 运行的，但应该足够直观，以便能够修改以在其他平台（如 Linux 或 Windows）上运行。

在本章中不需要安装任何新工具。

本章的源代码可以在 GitHub 上找到，地址为[`github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud/tree/master/Chapter12`](https://github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud/tree/master/Chapter12)。

为了能够按照书中描述的命令运行，将源代码下载到一个文件夹中，并设置一个环境变量，`$BOOK_HOME`，指向那个文件夹。示例命令包括以下内容：

```java
export BOOK_HOME=~/Documents/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud
git clone https://github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud $BOOK_HOME
cd $BOOK_HOME/Chapter12
```

Java 源代码是为 Java 8 编写的，并在 Java 12 上进行了测试。本章使用 Spring Cloud 2.1.0，SR1（也称为**Greenwich**版本），Spring Boot 2.1.4 和 Spring 5.1.6，即在撰写本章时可用的 Spring 组件的最新版本。

所有 Dockerfile 中使用的基础 Docker 镜像，`openjdk:12.0.2`。

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

+   `spring-cloud/config-server`

本章中的所有源代码示例都来自`$BOOK_HOME/Chapter12`的源代码，但在许多情况下，为了删除源代码中不相关部分，例如注释、导入语句和日志语句，进行了编辑。

如果你想看到第十二章《集中配置》中应用到源代码的变化，即添加配置服务器所做的工作，你可以将其与第十一章《保护 API 访问》的源代码进行比较。你可以使用你喜欢的`diff`工具，比较两个文件夹`$BOOK_HOME/Chapter11`和`$BOOK_HOME/Chapter12`。

# 介绍 Spring Cloud Config 服务器

在第十章《使用 Spring Cloud Gateway 隐藏边缘服务器后的微服务》中，Spring Cloud Config 服务器（缩写为*config server*）将添加到边缘服务器后面的现有微服务景观中，就像其他微服务一样：

![](img/f1a6d8fa-dd07-47dd-9f4d-3861729c6a70.png)

当涉及到设置配置服务器时，有许多选项需要考虑：

+   为配置仓库选择存储类型

+   决定初始客户端连接，是连接到配置服务器还是发现服务器

+   保护配置，既防止未经授权访问 API 和

    避免在配置仓库中以明文形式存储敏感信息

让我们逐一浏览每个选项，并介绍配置服务器暴露的 API。

# 选择配置仓库的存储类型

如在第八章《Spring Cloud 简介》中*集中配置的 Spring Cloud Config*部分所述，配置服务器支持将配置文件存储在多种不同的后端中，例如：

+   Git 仓库

+   本地文件系统

+   HashiCorp Vault

+   一个 JDBC 数据库

在本章中，我们将使用本地文件系统。要使用本地文件系统，配置服务器需要启动 Spring 配置文件`native`。配置仓库的位置是通过使用属性`spring.cloud.config.server.native.searchLocations`来指定的。

# 决定初始客户端连接

默认情况下，客户端首先连接到配置服务器以获取其配置。基于配置，它连接到发现服务器，即我们情况下的 Netflix Eureka，以注册自己。也可以反过来做，即客户端首先连接到发现服务器找到一个配置服务器实例，然后连接到配置服务器获取其配置。这两种方法都有利弊。

在本章中，客户端将首先连接到配置服务器。采用这种方法，将能够在配置服务器中存储发现服务器的配置，即 Netflix Eureka。

要了解关于其他替代方案的更多信息，请参阅[`cloud.spring.io/spring-cloud-static/spring-cloud-config/2.1.0.RELEASE/single/spring-cloud-config.html#discovery-first-bootstrap`](https://cloud.spring.io/spring-cloud-static/spring-cloud-config/2.1.0.RELEASE/single/spring-cloud-config.html#_environment_repository)。

首先连接到配置服务器的担忧之一是，配置服务器可能成为一个单点故障。如果客户端首先连接到发现服务，如 Netflix Eureka，可以有多个配置服务器实例注册，从而避免单点故障。当我们后来学习 Kubernetes 中的*服务*概念时，将了解如何通过运行多个容器（例如，配置服务器）来避免单点故障。

# 保护配置

一般来说，配置信息将被视为敏感信息。这意味着我们需要保护配置信息在传输和静态存储时的安全。从运行时角度看，配置服务器不需要通过边缘服务器暴露给外部。然而，在开发过程中，能够访问配置服务器的 API 来检查配置是有用的。在生产环境中，建议锁定对外部访问配置服务器。

# 保护传输中的配置

当微服务请求配置信息，或者任何使用配置服务器 API 的人，它将受到边缘服务器的保护，因为它已经使用 HTTPS 来防止窃听。

为了确保 API 用户是已知的客户端，我们将使用 HTTP 基本认证。我们可以在配置服务器中使用 Spring Security 来设置 HTTP 基本认证，并通过指定环境变量`SPRING_SECURITY_USER_NAME`和`SPRING_SECURITY_USER_PASSWORD`来指定许可凭证。

# 静态保护配置

为了避免任何可以访问配置仓库的人窃取敏感信息，如密码，配置服务器支持在磁盘上存储配置信息时进行加密。配置服务器支持使用对称和非对称密钥。非对称密钥更安全，但更难管理。

在本章中，我们将使用对称密钥。对称密钥在配置服务器启动时通过指定环境变量`ENCRYPT_KEY`赋予配置服务器。加密的密钥只是一个需要像任何敏感信息一样保护的普通文本字符串。

要了解关于非对称密钥的使用，请参阅[`cloud.spring.io/spring-cloud-static/spring-cloud-config/2.1.0.RELEASE/single/spring-cloud-config.html#_key_management`](https://cloud.spring.io/spring-cloud-static/spring-cloud-config/2.1.0.RELEASE/single/spring-cloud-config.html#_key_management)。

# 介绍配置服务器 API

配置服务器暴露了一个 REST API，客户端可以使用它来检索他们的配置。在本章中，我们将使用 API 以下端点：

+   `/actuator`：所有微服务暴露的标准 actuator 端点。

    像往常一样，这些应该小心使用。它们在开发过程中非常有用，但在投入生产前必须被锁定。

+   `/encrypt`和`/decrypt`：用于加密和解密敏感信息的端点。这些在投入生产前也必须被锁定。

+   `/{microservice}/{profile}`：返回指定微服务和指定 Spring 配置文件的营养配置。

当我们尝试配置服务器时，将看到 API 的一些示例使用。

# 设置配置服务器

基于讨论的决定来设置配置服务器是简单的：

1.  使用第三章中描述的 Spring Initializr 创建 Spring Boot 项目*创建一组协作微服务*。参考*使用 Spring Initializr 生成骨架代码*部分。

1.  在 Gradle 构建文件`build.gradle`中添加依赖项`spring-cloud-config-server`和`spring-boot-starter-security`。

1.  在应用类`ConfigServerApplication`上添加注解`@EnableConfigServer`：

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

logging.level.root: info
---
spring.profiles: docker
spring.cloud.config.server.native.searchLocations: file:/config-repo

```

最重要的配置是指定配置仓库的位置，由`spring.cloud.config.server.native.searchLocations`属性指定。

1.  在边缘服务器上添加路由规则，使配置服务器的 API 可以从微服务景观外部访问。

1.  在三个 Docker Compose 文件中添加 Dockerfile 和对配置服务器的定义。

1.  将敏感配置参数外部化到标准 Docker Compose 环境文件`.env`中。

1.  在常用的构建文件`settings.gradle`中添加配置服务器：

```java
include ':spring-cloud:config-server'
```

配置服务器 Spring Cloud Configuration 的源代码可以在`$BOOK_HOME/Chapter12/spring-cloud/config-server`中找到。

现在，让我们更深入地了解一下如何设置路由规则以及如何为在 Docker 中使用配置服务器进行配置。

# 在边缘服务器上设置路由规则

为了能够从微服务景观外部访问配置服务器的 API，我们在边缘服务器上添加一个路由规则。所有以`/config`开头的对边缘服务器的请求将被路由到配置服务器，以下路由规则：

```java
 - id: config-server
   uri: http://${app.config-server}:8888
  predicates:
  - Path=/config/**
  filters:
  - RewritePath=/config/(?<segment>.*), /$\{segment}
```

在前面路由规则中的`RewritePath`过滤器将从传入 URL 中删除前缀`/config`，然后将其发送到配置服务器。

有了这个路由规则，我们可以使用配置服务器的 API；例如，运行以下命令，当产品服务使用 Docker Spring 配置文件时，请求其配置：

```java
curl https://dev-usr:dev-pwd@localhost:8443/config/product/docker -ks | jq
```

当我们尝试配置服务器时，我们将运行前面的命令。

# 配置配置服务器以与 Docker 一起使用

配置服务器的 Dockerfile 与其他微服务相同，不同之处在于它暴露了端口`8888`，而不是端口`8080`。

当涉及到将配置服务器添加到 Docker Compose 文件时，它与我们已经看到的其他微服务有所不同：

```java
config-server:
  environment:
    - SPRING_PROFILES_ACTIVE=docker,native
    - ENCRYPT_KEY=${CONFIG_SERVER_ENCRYPT_KEY}
    - SPRING_SECURITY_USER_NAME=${CONFIG_SERVER_USR}
    - SPRING_SECURITY_USER_PASSWORD=${CONFIG_SERVER_PWD}
  volumes:
    - $PWD/config-repo:/config-repo
  build: spring-cloud/config-server
  mem_limit: 350m
```

以下是对前述源代码的解释：

1.  添加 Spring 配置文件`native`，以向配置服务器表明配置仓库是基于普通文件，换句话说，它不是一个 Git 仓库。

1.  环境变量`ENCRYPT_KEY`用于指定配置服务器用于加密和解密敏感配置信息的对称加密密钥。

1.  环境变量`SPRING_SECURITY_USER_NAME`和`SPRING_SECURITY_USER_PASSWORD`用于指定用于保护使用基本 HTTP 认证的 API 的凭据。

1.  卷声明将使`config-repo`文件夹在 Docker 容器中的`/config-repo`处可用。

`.env`文件中定义的前三个环境变量的值由 Docker Compose 从该文件中获取：

```java
CONFIG_SERVER_ENCRYPT_KEY=my-very-secure-encrypt-key
CONFIG_SERVER_USR=dev-usr
CONFIG_SERVER_PWD=dev-pwd
```

`.env`文件中存储的信息，即用户名、密码和加密密钥，都是敏感信息，如果用于除开发和测试之外的其他目的，必须加以保护。另外，请注意，失去加密密钥将导致配置仓库中的加密信息无法被解密！

# 配置配置服务器的客户端

为了使微服务能够从配置服务器获取其配置，我们需要更新微服务。这可以通过以下步骤完成：

1.  在 Gradle 构建文件`build.gradle`中添加`spring-cloud-starter-config`和`spring-retry`依赖项。

1.  将配置文件`application.yml`移动到配置仓库，并将其重命名为根据`spring.application.name`属性指定的客户端名称。

1.  在`src/main/resources`文件夹中添加一个名为`bootstrap.yml`的文件。该文件包含了连接到配置服务器的配置信息。有关其内容的解释请参考以下内容。

1.  在 Docker Compose 文件中添加访问配置服务器的凭据，例如，`product`服务：

```java
product:
  environment:
 - CONFIG_SERVER_USR=${CONFIG_SERVER_USR}
 - CONFIG_SERVER_PWD=${CONFIG_SERVER_PWD}
```

1.  禁用在基于 Spring Boot 的自动化测试中使用配置服务器。这可以通过在`@DataMongoTest`、`@DataJpaTest`和`@SpringBootTest`注解中添加`spring.cloud.config.enabled=false`来实现。例如，执行以下命令：

```java
@DataMongoTest(properties = {"spring.cloud.config.enabled=false"})

@DataJpaTest(properties = {"spring.cloud.config.enabled=false"})

@SpringBootTest(webEnvironment=RANDOM_PORT, properties = {"eureka.client.enabled=false", "spring.cloud.config.enabled=false"})
```

# 配置连接信息

如前所述，`src/main/resources/bootstrap.yml`文件包含了连接到配置服务器所需的客户端配置。除指定为`spring.application.name`的属性（在以下示例中设置为`product`）之外，该文件对所有配置服务器的客户端具有相同的内容：

```java
app.config-server: localhost

spring:
  application.name: product
  cloud.config:
    failFast: true
    retry:
      initialInterval: 3000
      multiplier: 1.3
      maxInterval: 10000
      maxAttempts: 20
    uri: http://${CONFIG_SERVER_USR}:${CONFIG_SERVER_PWD}@${app.config-server}:8888

---
spring.profiles: docker

app.config-server: config-server
```

此配置将使客户端执行以下操作：

1.  当在 Docker 外部运行时，使用`http://localhost:8888` URL 连接到配置服务器，当在 Docker 容器中运行时，使用`http://config-server:8888` URL 连接到配置服务器。

1.  使用`CONFIG_SERVER_USR`和`CONFIG_SERVER_PWD`属性的值进行 HTTP 基本认证，作为其用户名和密码。

1.  在启动过程中，如果需要，尝试重新连接配置服务器高达 20 次。

1.  如果连接尝试失败，客户端将首先等待 3 秒然后尝试重新连接。

1.  后续重试的等待时间将增加 1.3 倍。

1.  连接尝试之间的最大等待时间为 10 秒。

1.  如果客户端在 20 次尝试后仍无法连接到配置服务器，则启动失败。

此配置通常有助于提高配置服务器临时连接问题的弹性。当整个微服务及其配置服务器一次性启动时，它特别有用，例如，使用`docker-compose up`命令时。在这种情况下，许多客户端将试图在配置服务器准备好之前连接到它，重试逻辑将使客户端在配置服务器启动并运行后成功连接到它。

# 将分区配置从 Docker Compose 文件移动到配置仓库

`docker-compose-partitions.yml`和`docker-compose-kafka.yml`Docker Compose 文件包含一些用于处理消息代理中的分区的额外配置，分别是 RabbitMQ 和 Kafka。具体请参考第七章、*开发反应式微服务*中的*保证顺序和分区*部分。此配置也已移动到集中配置仓库。

例如，在`docker-compose-kafka.yml`中，读取 Kafka 中产品主题第一个分区的产品消费者的配置如下所示：

```java
  product:
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - MANAGEMENT_HEALTH_RABBIT_ENABLED=false
      - SPRING_CLOUD_STREAM_DEFAULTBINDER=kafka
      - SPRING_CLOUD_STREAM_BINDINGS_INPUT_CONSUMER_PARTITIONED=true
      - SPRING_CLOUD_STREAM_BINDINGS_INPUT_CONSUMER_INSTANCECOUNT=2
      - SPRING_CLOUD_STREAM_BINDINGS_INPUT_CONSUMER_INSTANCEINDEX=0
```

此配置已结构化为多个 Spring 配置文件，以提高可重用性，并移动到配置仓库中的相应配置文件中。添加的 Spring 配置文件如下：

+   `streaming_partitioned`包含用于在消息代理中启用分区的属性。

+   `streaming_instance_0`包含从第一个分区消费消息所需的属性。

+   `streaming_instance_1`包含从第二个分区消费消息所需的属性。

+   `kafka`包含特定于使用 Kafka 作为消息代理的属性。

以下配置已添加到消息消费者的配置文件中，即产品、评论和推荐服务：

```java
---
spring.profiles: streaming_partitioned
spring.cloud.stream.bindings.input.consumer:
  partitioned: true
  instanceCount: 2

---
spring.profiles: streaming_instance_0
spring.cloud.stream.bindings.input.consumer.instanceIndex: 0

---
spring.profiles: streaming_instance_1
spring.cloud.stream.bindings.input.consumer.instanceIndex: 1

---
spring.profiles: kafka

management.health.rabbit.enabled: false
spring.cloud.stream.defaultBinder: kafka
```

以下配置已添加到消息生产者（即产品组合服务）的配置文件中：

```java
---
spring.profiles: streaming_partitioned

spring.cloud.stream.bindings.output-products.producer:
  partition-key-expression: payload.key
  partition-count: 2

spring.cloud.stream.bindings.output-recommendations.producer:
  partition-key-expression: payload.key
  partition-count: 2

spring.cloud.stream.bindings.output-reviews.producer:
  partition-key-expression: payload.key
  partition-count: 2

---
spring.profiles: kafka

management.health.rabbit.enabled: false
spring.cloud.stream.defaultBinder: kafka
```

现在 Docker Compose 文件更加简洁，只包含访问配置服务器和激活的 Spring 配置文件的凭据列表。例如，产品消费者从 Kafka 的产品主题的第一分区读取消息的配置现在减少到以下内容：

```java
product:
    environment:
      - SPRING_PROFILES_ACTIVE=docker,streaming_partitioned,streaming_instance_0,kafka
      - CONFIG_SERVER_USR=${CONFIG_SERVER_USR}
      - CONFIG_SERVER_PWD=${CONFIG_SERVER_PWD}
```

完整的源代码请参考以下内容：

+   `docker-compose-partitions.yml`

+   `docker-compose-kafka.yml`

+   `config-repo/product-composite.yml`

+   `config-repo/product.yml`

+   `config-repo/recommendation.yml`

+   `config-repo/review.yml`

# 结构化配置仓库

将每个客户端的配置文件移动到配置仓库后，我们在许多配置文件中会有一定程度的一致性配置，例如，actuator 端点的配置和如何连接到 Eureka、RabbitMQ 和 Kafka。公共部分已放置在一个名为`application.yml`的配置文件中，该文件由所有客户端共享。配置仓库包含以下文件：

+   `application.yml`

+   `eureka-server.yml`

+   `product-composite.yml`

+   `recommendation.yml`

+   `auth-server.yml`

+   `gateway.yml`

+   `product.yml`

+   `review.yml`

配置仓库`configuration repository`可以在`$BOOK_HOME/Chapter12/config-repo`找到。

# 尝试使用 Spring Cloud Configuration 服务器

现在是我们尝试配置服务器的时候了：

1.  首先，我们从源代码开始构建，并运行测试脚本来确保一切都能正常运行。

1.  接下来，我们将尝试使用配置服务器 API 来获取我们微服务的配置。

1.  最后，我们将了解如何加密和解密敏感信息，例如密码。

# 构建和运行自动化测试

现在我们按照以下方式构建和运行：

1.  首先，使用以下命令构建 Docker 镜像：

```java
cd $BOOK_HOME/Chapter12
./gradlew build && docker-compose build
```

1.  接下来，在 Docker 中启动系统架构，并使用以下命令运行常规测试：

```java
./test-em-all.bash start
```

# 使用配置服务器 API 获取配置

如前面所述，我们可以通过边缘服务器使用 URL 前缀`/config`来访问配置服务器的 API。我们还需要提供`/.env`文件中指定的 HTTP 基本认证凭据。例如，要在产品服务作为 Docker 容器运行时获取配置，即激活了 Spring 配置文件`docker`，请运行以下命令：

```java
curl https://dev-usr:dev-pwd@localhost:8443/config/product/docker -ks | jq .
```

预期响应具有以下结构（响应中的许多属性被`...`替换以提高可读性）：

```java
{
  "name": "product",
  "profiles": [
    "docker"
  ],
  ...
  "propertySources": [
    {
      "name": "file:/config-repo/product.yml (document #1)",
      "source": {
        "spring.profiles": "docker",
        "server.port": 8080,
        ...
      }
    },
    {
      "name": "file:/config-repo/application.yml (document #1)",
      "source": {
        "spring.profiles": "docker",
        ...
      }
    },
    {
      "name": "file:/config-repo/product.yml (document #0)",
      "source": {
        "server.port": 7001,
        ...
      }
    },
    {
      "name": "file:/config-repo/application.yml (document #0)",
      "source": {
        ...
        "app.eureka-password": "p",
        "spring.rabbitmq.password": "guest"
      }
    }
  ]
}
```

以下是对前面响应的解释：

+   响应包含来自多个*属性源*的属性，每个属性源对应一个匹配 API 请求的 Spring 配置文件和属性文件。属性源按优先级顺序返回；也就是说，如果一个属性在多个属性源中指定，响应中的第一个属性具有优先权。前面的示例响应包含以下属性源：

    +   `/config-repo/product.yml`，用于`docker` Spring 配置文件

    +   `/config-repo/application.yml`，用于`docker` Spring 配置文件

    +   `/config-repo/product.yml`，用于`default` Spring 配置文件

    +   `/config-repo/application.yml`，用于`default` Spring 配置文件 docker

+   例如，将使用的端口将是`8080`，而不是`7001`，因为在前面的响应中`"server.port": 8080`位于`"server.port": 7001`之前。

+   敏感信息（如 Eureka 和 RabbitMQ 的密码）以明文形式返回，例如`"p"`和`"guest"`，但它们在磁盘上是加密的。在配置文件`application.yml`中，它们如下所示：

```java
app:
  eureka-password: '{cipher}bf298f6d5f878b342f9e44bec08cb9ac00b4ce57e98316f030194a225fac89fb'

spring.rabbitmq:
  password: '{cipher}17fcf0ae5b8c5cf87de6875b699be4a1746dd493a99d926c7a26a68c422117ef'
```

# 加密和解密敏感信息

信息可以使用配置服务器暴露的`/encrypt`和`/decrypt`端点进行加密和解密。`/encrypt`端点可用于创建加密值，以便将其放置在配置仓库中的属性文件中。参考前面的示例，其中 Eureka 和 RabbitMQ 的密码以加密形式存储在磁盘上。`/decrypt`端点可用于验证存储在配置仓库磁盘上的加密信息。

要加密`hello world`字符串，请运行以下命令：

```java
curl -k https://dev-usr:dev-pwd@localhost:8443/config/encrypt --data-urlencode "hello world"
```

使用`curl`调用`/encrypt`端点时，使用`--data-urlencode`标志很重要，以确保正确处理`'+'`等特殊字符。

预期如下形式的响应：

![](img/733638b7-6ddf-4111-b1b8-9ea27d16f42e.png)

要解密加密值，请运行以下命令：

```java
curl -k https://dev-usr:dev-pwd@localhost:8443/config/decrypt -d 9eca39e823957f37f0f0f4d8b2c6c46cd49ef461d1cab20c65710823a8b412ce
```

预期`hello world`字符串作为响应：

![](img/db7066ec-de99-4008-91ad-a347b02ff206.png)

如果您想在配置文件中使用加密值，您需要在其前加上`{cipher}`并将其包裹在`''`中。例如，要存储`hello world`的加密版本，请运行以下命令：

```java
my-secret:'{cipher}9eca39e823957f37f0f0f4d8b2c
6c46cd49ef461d1cab20c65710823a8b412ce'
```

这些测试总结了关于集中配置章节的内容。通过关闭系统架构来结束：

```java
docker-compose down
```

# 总结

在本章中，我们看到了如何使用 Spring Cloud Configuration 服务器来集中管理微服务的配置。我们可以将配置文件放在一个共同的配置仓库中，并在一个配置文件中共享公共配置，同时将微服务特定的配置保存在微服务特定的配置文件中。微服务在启动时已更新，可以从配置服务器检索其配置，并配置为在从配置服务器检索配置时处理临时故障。

配置服务器可以通过要求对其 API 进行基本 HTTP 认证的认证使用来保护配置信息，并通过使用 HTTPS 的边缘服务器对外暴露其 API 来防止窃听。为了防止获取到磁盘上配置文件访问权的入侵者获取密码等敏感信息，我们可以使用配置服务器的`/encrypt`端点来加密信息并将其加密存储在磁盘上。

虽然在外部暴露配置服务器的 API 在开发过程中很有用，但在生产中使用前应该加以限制。

在下一章中，我们将学习如何使用**Resilience4j**来减轻过度使用同步通信可能带来的潜在缺点。例如，如果我们建立一个使用 REST API 同步调用彼此的微服务链，最后一个微服务停止响应，可能会发生一些坏事，影响到很多涉及的微服务。Resilience4j 带有断路器模式的实现，可以用来处理这类问题。

# 问题

1.  启动时，评审服务期望从配置服务器中调用什么 API 来检索其配置？评审服务是使用以下命令启动的：`docker compose up -d`。

1.  使用此命令从配置服务器调用 API 期望返回什么配置信息：[完整命令]

```java
curl https://dev-usr:dev-pwd@localhost:8443/config/application/default -ks | jq 
```

1.  **Spring Cloud Config**支持哪些类型的存储后端？

1.  我们如何使用 Spring Cloud Config 对磁盘上的敏感信息进行加密？

1.  我们如何保护配置服务器的 API 免受滥用？

1.  请列举一些首次连接配置服务器与首次连接发现服务器的客户端的优缺点。
