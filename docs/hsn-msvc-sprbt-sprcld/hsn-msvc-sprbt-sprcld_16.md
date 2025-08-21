# 第十四章：理解分布式追踪

在本章中，我们将学习如何使用分布式追踪更好地了解我们的微服务如何协作，例如，对外部 API 发送请求。能够利用分布式追踪对于能够管理相互协作的微服务系统架构至关重要。如已在第八章, *Spring Cloud 简介*中提到的*Spring Cloud Sleuth 和 Zipkin 进行分布式追踪*部分所述，Spring Cloud Sleuth 将用于收集追踪信息，而 Zipkin 将用于存储和可视化所述追踪信息。

在本章中，我们将学习以下主题：

+   使用 Spring Cloud Sleuth 和 Zipkin 引入分布式追踪

+   如何将分布式追踪添加到源代码中

+   如何进行分布式追踪：

    +   我们将学习如何使用 Zipkin 可视化追踪信息，并与以下内容相关：

        +   成功和失败的 API 请求

        +   API 请求的同步和异步处理

    +   我们将同时使用 RabbitMQ 和 Kafka 将微服务中的追踪事件发送到 Zipkin 服务器

# 技术要求

本书中描述的所有命令都是在 MacBook Pro 上使用 macOS Mojave 运行的，但应该很容易修改，以便它们可以在其他平台（如 Linux 或 Windows）上运行。

在本章中不需要安装任何新工具。

本章的源代码可以在 GitHub 上找到，地址为[`github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud/tree/master/Chapter14`](https://github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud/tree/master/Chapter14)。

为了能够按照书中描述运行命令，将源代码下载到一个文件夹中，并设置一个环境变量`$BOOK_HOME`，使其指向该文件夹。一些示例命令如下：

```java
export BOOK_HOME=~/Documents/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud
git clone https://github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud $BOOK_HOME
cd $BOOK_HOME/Chapter14
```

该 Java 源代码是为 Java 8 编写的，并在 Java 12 上进行了测试。本章使用 Spring Cloud 2.1.0, SR1（也称为**Greenwich**版本），Spring Boot 2.1.4 和 Spring 5.1.6，即在撰写本章时可用的 Spring 组件的最新版本。

所有 Dockerfile 中均使用基础 Docker 镜像`openjdk:12.0.2`。

本章中的所有示例代码均来自`$BOOK_HOME/Chapter14`的源代码，但在许多情况下，为了删除源代码中不相关部分，例如注释和导入以及日志声明，对其进行了编辑。

如果你想查看本章源代码所做的更改，即了解添加 Spring Cloud Sleuth 和 Zipkin 进行分布式追踪所需的内容，你可以将其与第十三章, *使用 Resilience4j 提高弹性*的源代码进行比较。你可以使用你喜欢的`diff`工具，比较两个文件夹——`$BOOK_HOME/Chapter13`和`$BOOK_HOME/Chapter14`。

# 使用 Spring Cloud Sleuth 和 Zipkin 引入分布式跟踪。

回顾第八章，*Spring Cloud 简介*，在*分布式跟踪的 Spring Cloud Sleuth 和 Zipkin*部分，整个工作流程的跟踪信息称为一个**跟踪**或一个**跟踪树**，树的子部分，例如工作基本单元，称为一个**跨度**。跨度可以包括子跨度，形成跟踪树。Zipkin UI 可以如下可视化跟踪树和其跨度：

![](img/c486eb1c-6ecf-4f66-9783-fdecfe33f7dc.png)

Spring Cloud Sleuth 可以通过 HTTP 同步发送跟踪信息到 Zipkin，或者使用 RabbitMQ 或 Kafka 等消息代理异步发送。为了避免在微服务中创建对 Zipkin 服务器的运行时依赖，最好使用 RabbitMQ 或 Kafka 异步发送跟踪信息到 Zipkin。以下图表说明了这一点：

![](img/0006d41f-9f89-4f39-a1ca-8d3971451602.png)

Zipkin 支持本地存储跟踪信息，存储在内存中，或存储在 Apache Cassandra、Elasticsearch 或 MySQL 中。此外，还有许多扩展可用。具体信息请参考[`zipkin.apache.org/pages/extensions_choices.html`](https://zipkin.apache.org/pages/extensions_choices.html)。在本章中，我们将把跟踪信息存储在内存中。

# 向源代码添加分布式跟踪。

在本节中，我们将学习如何更新源代码，使用 Spring Cloud Sleuth 和 Zipkin 启用分布式跟踪。可以通过以下步骤完成：

1.  向构建文件添加依赖项，以引入 Spring Cloud Sleuth 和将跟踪信息发送到 Zipkin 的能力。

1.  为之前未使用过的项目（即 Spring Cloud 项目的`authorization-server`、`eureka-server`和`gateway`）添加 RabbitMQ 和 Kafka 依赖项。

1.  配置微服务使用 RabbitMQ 或 Kafka 将跟踪信息发送到 Zipkin。

1.  在 Docker Compose 文件中添加一个 Zipkin 服务器。

1.  在`docker-compose-kafka.yml`中为 Spring Cloud 项目的`authorization-server`、`eureka-server`和`gateway`添加`kafka` Spring 配置文件。

添加 Zipkin 服务器将通过使用 Docker Hub 上由 Zipkin 项目发布的 Docker 镜像来实现。具体细节请参考[`hub.docker.com/r/openzipkin/zipkin`](https://hub.docker.com/r/openzipkin/zipkin)。

Zipkin 本身是一个 Spring Boot 应用程序，在撰写本文时，它正在 Apache 软件基金会（ASF）下孵化。更多信息请参考[`zipkin.apache.org/`](https://zipkin.apache.org/)。

# 向构建文件添加依赖项。

为了能够使用 Spring Cloud Sleuth 并发送跟踪信息到 Zipkin，我们需要在 Gradle 项目的构建文件`build.gradle`中添加几个依赖项：

这通过添加以下两行来实现：

```java
implementation('org.springframework.cloud:spring-cloud-starter-sleuth')   implementation('org.springframework.cloud:spring-cloud-starter-zipkin')
```

对于尚未使用过 RabbitMQ 和 Kafka 的 Gradle 项目，即 Spring Cloud 项目`authorization-server`、`eureka-server`和`gateway`，需要添加以下依赖项：

```java
implementation('org.springframework.cloud:spring-cloud-starter-stream-rabbit')
implementation('org.springframework.cloud:spring-cloud-starter-stream-kafka')
```

# 为 Spring Cloud Sleuth 和 Zipkin 添加配置

在公共配置文件`config-repo/application.yml`中添加了使用 Spring Cloud Sleuth 和 Zipkin 的配置。在默认配置文件中，指定跟踪信息应通过 RabbitMQ 发送到 Zipkin：

```java
spring.zipkin.sender.type: rabbit
```

默认情况下，Spring Cloud Sleuth 只将 10%的跟踪信息发送到 Zipkin。为了确保所有跟踪信息都发送到 Zipkin，在默认配置文件中添加了以下属性：

```java
spring.sleuth.sampler.probability: 1.0
```

当使用 Kafka 将跟踪信息发送到 Zipkin 时，将使用`kafka`Spring 配置文件。在前几章中，`kafka`Spring 配置文件是在特定于组合和核心微服务的配置文件中定义的。在本章中，Spring Cloud 服务也将使用 Kafka 将跟踪信息发送到 Zipkin，因此将`kafka`Spring 配置文件移动到公共配置文件`config-repo/application.yml`中。在`kafka`Spring 配置文件中还添加了以下两个属性：

+   `spring.zipkin.sender.type: kafka`告诉 Spring Cloud Sleuth 使用 Kafka 将跟踪信息发送到 Zipkin。

+   `spring.kafka.bootstrap-servers: kafka:9092`指定了 Kafka 服务器的所在位置。

总的来说，`kafka`Spring 配置文件如下所示：

```java
--- 
spring.profiles: kafka

management.health.rabbit.enabled: false
spring.cloud.stream.defaultBinder: kafka
spring.zipkin.sender.type: kafka
spring.kafka.bootstrap-servers: kafka:9092
```

# 将 Zipkin 添加到 Docker Compose 文件中

正如我们之前提到的，Zipkin 服务器是通过使用已经存在的 Docker 镜像`openzipkin/zipkin`添加到 Docker Compose 文件中的，该镜像是由 Zipkin 项目发布的。在`docker-compose.yml`和`docker-compose-partitions.yml`中，其中使用 RabbitMQ 时，Zipkin 服务器的定义如下所示：

```java
zipkin:
  image: openzipkin/zipkin:2.12.9
  networks:
    - my-network
  environment:
    - RABBIT_ADDRESSES=rabbitmq
    - STORAGE_TYPE=mem
  mem_limit: 512m
  ports:
    - 9411:9411
  depends_on:
    rabbitmq:
      condition: service_healthy
```

让我们解释一下前面的源代码：

+   Docker 镜像`openzipkin/zipkin`的版本被指定为`2.12.19`版本。

+   环境变量`RABBIT_ADDRESSES=rabbitmq`用于指定 Zipkin 使用 RabbitMQ 接收跟踪信息，并且 Zipkin 使用主机名`rabbitmq`连接到 RabbitMQ。

+   环境变量`STORAGE_TYPE=mem`用于指定 Zipkin 将所有跟踪信息保存在内存中。

+   Zipkin 的内存限制增加到 512 MB，而其他容器的内存限制为 350 MB。这是因为 Zipkin 被配置为将所有跟踪信息保存在内存中，所以过了一段时间后，它将比其他容器消耗更多的内存。

+   Zipkin 暴露出 HTTP 端口`9411`，供浏览器访问其 Web 用户界面。

+   Docker 将等待启动 Zipkin 服务器，直到 RabbitMQ 服务向 Docker 报告自己运行正常。

虽然这对于将跟踪信息存储在 Zipkin 内存中以进行开发和测试活动来说是可行的，但在生产环境中，Zipkin 应配置为将跟踪信息存储在数据库中，例如 Apache Cassandra、Elasticsearch 或 MySQL。

在`docker-compose-kafka.yml`中，其中使用了 Kafka，Zipkin 服务器的定义如下所示：

```java
zipkin:
  image: openzipkin/zipkin:2.12.9  
  networks: 
    - my-network 
  environment:
    - KAFKA_BOOTSTRAP_SERVERS=kafka:9092
    - STORAGE_TYPE=mem 
  mem_limit: 512m
  ports: 
    - 9411:9411 
  depends_on: 
    - kafka
```

让我们详细解释一下前面的源代码：

+   使用 Zipkin 和 Kafka 的配置与之前使用 Zipkin 和 RabbitMQ 的配置相似。

+   主要区别在于使用`KAFKA_BOOTSTRAP_SERVERS=kafka:9092`环境变量，该变量用于指定 Zipkin 应使用 Kafka 接收跟踪信息，并且 Zipkin 应通过主机名`kafka`和端口`9092`连接到 Kafka。

在`docker-compose-kafka.yml`中，为 Spring Cloud 服务`eureka`、`gateway`和`auth-server`添加了`kafka` Spring 配置文件：

```java
    environment:
      - SPRING_PROFILES_ACTIVE=docker,kafka
```

这就是使用 Spring Cloud Sleuth 和 Zipkin 添加分布式跟踪所需的一切，所以在下一节让我们试试吧！

# 尝试分布式跟踪

在源代码中进行了必要的更改后，我们可以尝试分布式跟踪！我们将通过执行以下步骤来实现：

1.  构建、启动并验证使用 RabbitMQ 作为队列管理器的系统架构。

1.  发送一个成功的 API 请求，看看我们可以找到与这个 API 请求相关的 Zipkin 中的跟踪信息。

1.  发送一个失败的 API 请求，看看 Zipkin 中的跟踪信息是什么样子。

1.  发送一个成功的 API 请求，触发异步处理，并查看其在 Zipkin 中的跟踪信息表示。

1.  调查如何监控通过 RabbitMQ 传递给 Zipkin 的跟踪信息。

1.  将队列管理器切换到 Kafka，并重复前面的步骤。

我们将在接下来的部分详细讨论这些步骤。

# 使用 RabbitMQ 作为队列管理器启动系统架构

让我们启动系统架构。使用以下命令构建 Docker 镜像：

```java
cd $BOOK_HOME/Chapter14
./gradlew build && docker-compose build
```

使用 Docker 启动系统架构，并使用以下命令运行常规测试：

```java
./test-em-all.bash start
```

在我们可以调用 API 之前，我们需要一个访问令牌。运行以下命令以获取访问令牌：

```java
unset ACCESS_TOKEN
ACCESS_TOKEN=$(curl -k https://writer:secret@localhost:8443/oauth/token -d grant_type=password -d username=magnus -d password=password -s | jq -r .access_token)
echo $ACCESS_TOKEN
```

# 发送一个成功的 API 请求

现在，我们准备发送一个正常的 API 请求。运行以下命令：

```java
curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/2 -w "%{http_code}\n" -o /dev/null -s
```

期望命令返回成功的 HTTP 状态码，即 200。

现在我们可以启动 Zipkin UI，查看已经发送到 Zipkin 的跟踪信息：

1.  在您的网络浏览器中打开以下 URL：`http://localhost:9411/zipkin/`。

1.  为了找到我们请求的跟踪信息，请执行以下步骤：

    1.  选择“服务名称”：gateway。

    1.  设置排序顺序：最新优先。

    1.  点击“查找跟踪”按钮。

查找跟踪的响应应如下所示：

![](img/ee509be8-dcc5-41bb-b758-361379f8fe94.png)

我们之前的 API 请求的跟踪信息是列表中的第一个。点击它以查看与跟踪相关的详细信息：

![](img/8fa726b9-d1b5-424a-ade5-c5324f925c2c.png)

在详细的跟踪信息视图中，我们可以观察到以下内容：

1.  请求被网关服务接收。

1.  它将请求的处理委托给了`product-composite`服务。

1.  `product-composite`服务反过来向核心服务发送了三个并行请求：`product`、`recommendation`和`review`。

1.  一旦`product-composite`服务收到了所有三个核心服务的响应，它就创建了一个复合响应。

1.  复合响应通过网关服务返回到调用者。

当使用 Safari 时，我注意到跟踪树并不总是正确渲染。切换到 Chrome 或 Firefox 可以解决此问题。

如果我们点击第一个跨度，网关，我们可以看到更多细节：

![](img/f6f693ce-7cea-43ea-ba6c-8faa8db74269.png)

这里，我们可以看到我们实际发送的请求：`product-composite/2`。这在我们分析例如长时间完成的跟踪时非常有价值！

# 发送一个失败的 API 请求

让我们看看如果我们发起一个失败的 API 请求会怎样，例如，搜索一个不存在的产品：

1.  为产品 ID `12345`发送 API 请求，并验证它返回了未找到的 HTTP 状态码，即 404：

```java
curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/12345 -w "%{http_code}\n" -o /dev/null -s
```

1.  在 Zipkin UI 中，回到搜索页面（在网页浏览器中使用后退按钮）并点击“查找跟踪”按钮。你应该会在返回列表的顶部看到失败的请求，用红色标出：

![](img/3b7bf196-16fe-4277-8e51-3f393696278f.png)

1.  点击标记为红色的顶部跟踪：

![](img/4ca53dfa-35ad-4c3c-b50f-8e3a3c357f61.png)

1.  在详细跟踪视图中，我们可以通过颜色编码看到产品服务在调用`product-composite`时出了错。点击产品跨度以查看出错详情：

![](img/b33855c9-3b3e-4aaf-81ed-8a080c05dec7.png)

这里，我们可以看到导致错误的请求`product/12345`以及返回的错误代码和原因：404 Not Found。这在我们分析故障的根本原因时非常有用！

# 发送一个触发异步处理的 API 请求

在 Zipkin UI 中看到的第三种有趣的请求类型是一个部分处理异步的请求。让我们尝试一个删除请求，其中核心服务中的删除过程是异步完成的。`product-composite`服务向消息代理的每个核心服务发送一个删除事件，并且每个核心服务都会拾取该删除事件并异步处理它。得益于 Spring Cloud Sleuth，发送到消息代理的事件中添加了跟踪信息，从而实现了对删除请求整体处理的连贯视图。

运行以下命令删除具有产品 ID`12345`的产品，并验证它返回成功的 HTTP 状态码，200：

```java
curl -X DELETE -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/12345 -w "%{http_code}\n" -o /dev/null -s
```

记住删除操作是幂等的，即即使产品不存在，它也会成功！

在 Zipkin UI 中，回到搜索页面（在 Web 浏览器中使用后退按钮）并点击`Find Traces`按钮。你应该在返回列表的顶部看到删除请求的跟踪：

![](img/c8304d85-6234-4bf4-9470-e18b2f92a68a.png)

点击第一个跟踪以查看其跟踪信息：

![](img/b06714c3-aec0-4166-a066-2001d3c14c3d.png)

在这里，我们可以看到处理删除请求的跟踪信息：

1.  请求被`gateway`服务接收。

1.  它将请求的处理委托给了`product-composite`服务。

1.  反过来，`product-composite`服务在消息代理（本例中为 RabbitMQ）上发布了三个事件。

1.  `product-composite`服务现在完成并返回一个成功的 HTTP 状态码，200，通过网关服务返回到调用者。

1.  核心服务`product`、`recommendation`和`review`接收到删除事件并开始异步处理它们，即彼此独立处理。

要查看更详细的信息，点击产品跨度：

![](img/7fd269c2-3bc7-4c83-91cd-6a16c6010757.png)

在这里，我们可以看到产品服务被输入通道的事件触发，该事件是从消息代理发送的。

Zipkin UI 包含更多查找感兴趣跟踪的功能！

为了更熟悉 Zipkin UI，尝试使用`Annotation Query`参数；例如，使用`http.path=/product-composite/214`或`error=401`查找因授权失败而失败的请求。注意默认设置为`10`的`Limit`参数，这可能会隐藏感兴趣的结果。还要确保`Lookback`参数不会删除感兴趣的跟踪！

# 监控通过 RabbitMQ 发送到 Zipkin 的跟踪信息

要监控通过 RabbitMQ 发送到 Zipkin 的跟踪信息，我们可以使用 RabbitMQ 管理 Web UI。在 Web 浏览器中打开以下 URL：`http://localhost:15672/#/queues/%2F/zipkin`。如果需要，使用用户名`guest`和密码`guest`登录。期待一个看起来像以下的网页：

![](img/0ebbf762-280a-457c-b52e-7344c666c249.png)

在名为`Message Rates`的图表中，我们可以看到跟踪消息正在以每秒 1.2 条消息的平均速率发送到 Zipkin。

使用以下命令结束 RabbitMQ 的分布式跟踪测试，关闭系统架构：

```java
docker-compose down
```

# 使用 Kafka 作为消息代理

让我们也验证一下我们可以使用 Kafka 而不是 RabbitMQ 向 Zipkin 发送跟踪信息！

使用以下命令启动系统架构：

```java
export COMPOSE_FILE=docker-compose-kafka.yml
./test-em-all.bash start
```

重复我们在前面章节中执行的命令，当时我们使用 RabbitMQ，并验证您可以在使用 Kafka 时在 Zipkin UI 中看到相同的跟踪信息：

Kafka 不提供像 RabbitMQ 那样的管理 Web UI。因此，我们需要运行一些 Kafka 命令来验证跟踪事件实际上是通过 Kafka 发送到 Zipkin 服务器的：

要在 Docker 容器中运行 Kafka 命令，请参阅第七章 *《开发响应式微服务》* 中的“每个主题使用两个分区”部分。

1.  首先，列出 Kafka 中可用的主题：

```java
docker-compose exec kafka /opt/kafka/bin/kafka-topics.sh --zookeeper zookeeper --list
```

1.  问题：

跟踪事件的具体细节并不重要。Zipkin 服务器为我们整理了信息，并在 Zipkin UI 中使其易于查看。这里的关键是我们可以看到通过 Kafka 发送到 Zipkin 服务器的跟踪事件。

1.  接下来，询问发送到`zipkin`话题的跟踪事件：

```java
docker-compose exec kafka /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic zipkin --from-beginning --timeout-ms 1000
```

1.  期待很多与以下类似的长时间运行的请求：

![](img/8fcdde07-be13-497a-8b95-1b56e4fe3a95.png)

在下一章中，我们将学习容器编排器，特别是 Kubernetes。我们将学习如何使用 Kubernetes 部署和管理微服务，同时提高重要的运行时特性，如可伸缩性、高可用性和弹性。

现在，请关闭系统架构并取消设置`COMPOSE_FILE`环境变量：

```java
docker-compose down
unset COMPOSE_FILE
```

这结束了关于分布式跟踪的章节！

# 摘要

期待找到一个名为`zipkin`的话题：

Zipkin UI 使识别复杂工作流中的哪个部分导致意外的长时间响应或错误变得非常容易。无论是同步还是异步工作流，都可以通过 Zipkin UI 进行可视化。

在本章中，我们学习了如何使用分布式跟踪来了解微服务如何协同工作。我们还学习了如何使用 Spring Cloud Sleuth 收集跟踪信息，以及如何使用 Zipkin 存储和可视化跟踪信息。

# `spring.sleuth.sampler.probability`配置参数的目的是什么？

1.  控制跟踪信息发送到 Zipkin 的配置参数是什么？

1.  如何在执行`test-em-all.bash`测试脚本后识别最长的运行请求？

1.  如何推广运行时组件的解耦？我们已经了解到如何在构建文件中添加几个依赖项，并设置一些配置参数。

1.  我们如何在第十三章 *使用 Resilience4j 提高弹性* 中找到被超时中断的请求？

1.  当第十三章中引入的断路器*Improving Resilience Using Resilience4j*打开时，API 请求的跟踪日志是什么样的？

1.  我们如何定位因调用者未获得授权而失败的 API？
