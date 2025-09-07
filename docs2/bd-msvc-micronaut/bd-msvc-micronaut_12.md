# *第八章*：部署微服务

*部署*的直译意思是将资源投入有效行动。因此，在微服务环境中，这意味着将微服务投入有效行动。任何服务部署都是一个多步骤的过程，通常涉及构建工件然后将工件推送到运行环境。在微服务世界中，一个有效的微服务部署策略至关重要。本质上，在规划部署过程时，我们需要注意以下几点：

+   继续采用*关注点分离*的模式，并为每个微服务的工件构建过程进行自我隔离。

+   解耦微服务内部的任何连接需求，并让服务发现或接近服务发现的实现来处理微服务绑定。

+   实施一个无缝的部署过程，能够以统一和自动化的方式实例化所有微服务应用程序组件。

在本章中，我们将深入探讨上述关注点，同时涵盖以下主题：

+   构建容器工件

+   部署容器工件

到本章结束时，你将对微服务部署的这些方面有深入的了解。

# 技术要求

本章中所有命令和技术说明均在 Windows 10 和 macOS 上运行。本章涵盖的代码示例可在本书的 GitHub 仓库中找到：

[`github.com/PacktPublishing/Building-Microservices-with-Micronaut/tree/master/Chapter08`](https://github.com/PacktPublishing/Building-Microservices-with-Micronaut/tree/master/Chapter08)

以下工具需要在开发环境中安装和设置：

+   **Java SDK**：版本 13 或以上（我们使用了 Java 14）。

+   **Maven**：这是可选的，仅在你希望使用 Maven 作为构建系统时需要。然而，我们建议在任何开发机器上设置 Maven。下载和安装 Maven 的说明可以在[`maven.apache.org/download.cgi`](https://maven.apache.org/download.cgi)找到。

+   **开发 IDE**：根据你的偏好，可以使用任何基于 Java 的 IDE，但为了编写本章，使用了 IntelliJ。

+   **Git**：下载和安装的说明可以在[`git-scm.com/downloads`](https://git-scm.com/downloads)找到。

+   **PostgreSQL**：下载和安装的说明可以在[`www.postgresql.org/download/`](https://www.postgresql.org/download/)找到。

+   **MongoDB**：MongoDB Atlas 提供了一个免费在线数据库即服务，存储空间高达 512 MB。然而，如果你更喜欢本地数据库，下载和安装的说明可以在[`docs.mongodb.com/manual/administration/install-community/`](https://docs.mongodb.com/manual/administration/install-community/)找到。我们为本章的编写使用了本地安装。

+   **REST 客户端**：任何 HTTP REST 客户端都可以使用。我们使用了 **Advanced REST Client** Chrome 插件。

+   **Docker**：有关下载和安装 Docker 的说明，请参阅[`docs.docker.com/get-docker/`](https://docs.docker.com/get-docker/)。

# 构建容器工件

部署任何应用程序的第一步是构建所有必需的工件。构建一个工件通常涉及检出源代码、编译并创建可部署的工件。在微服务方面，通常，这种可部署的形式是一个 Docker 容器镜像。Docker 镜像优雅地解耦了运行时拓扑需求。Docker 镜像是平台无关的，可以部署到任何运行 Docker 的主机机器。在下一节中，我们将深入了解如何在我们的 pet-clinic 应用程序中为微服务构建 Docker 镜像。

## 使用 Jib 容器化 Micronaut 微服务

Jib 是来自 Google 的一个容器化框架，它可以无缝地与 Java 构建框架（如 Maven 或 Gradle）结合使用来构建容器镜像。Jib 极大地简化了创建容器（Docker）镜像的过程。让我们快速看看没有 Jib 创建 Docker 镜像的工作流程：

![图 8.1 – 无 Jib 的 Docker 容器化工作流程![图 8.1 – 无 Jib 的 Docker 容器化工作流程图 8.1 – 无 Jib 的 Docker 容器化工作流程如前图所示，要构建没有 Jib 的 Docker 镜像，我们需要构建 **源代码** 并创建一个 **Docker 文件**。然后，**Docker 守护进程**使用构建上下文创建一个 **容器镜像** 并将其推送到仓库/注册表。与前面的工作流程相比，Jib 如下图中所示简化了整个过程：![图 8.2 – 使用 Jib 的 Docker 容器化![图 8.2 – 使用 Jib 的 Docker 容器化图 8.2 – 使用 Jib 的 Docker 容器化 Jib 从构建文件（如项目的 `pom` 文件或 Gradle 文件）中获取配置来构建容器镜像并将其上传到配置的仓库/注册表。在以下章节中，我们将使用 Jib 容器化我们的微服务。### 容器化宠物主人微服务在宠物主人微服务中，我们将简化配置管理以使用本地的 `application.properties`。我们仍然可以使用 `consul` 进行配置管理，但为了专注于容器化方面，我们选择使用本地的 `application.properties` 文件。删除或备份 `bootstrap.yml` 文件，并对 `application.properties` 文件进行以下更改：```javamicronaut:  application:    name: pet-owner  server:    port: 32581  router:    static-resources:      swagger:        paths: classpath:META-INF/swagger        mapping: /swagger/**datasources:  default:    url: "jdbc:postgresql://host.docker.internal:5432/postgres"    username: postgres    password: postgres    driverClassName: org.postgresql.Driverconsul:  client:    default-zone: "consul:8500"    registration:      enabled: truejpa:…```在这些应用程序属性更改中需要思考的一些事情如下：+   使用 `host.docker.internal` 而不是 localhost 来指向安装在主机操作系统（Docker 外部）上的 Postgres 实例。+   `consul` 服务。要从宠物主人 Docker 容器中使用 Docker 化的 `consul`，我们需要指定服务名称而不是 localhost。+   **端口**: 我们指定了一个固定的端口来运行宠物主人微服务，因为这个端口将在部署过程中被暴露。实际上，要使用主机机器上的任何内容，我们应该使用`host.docker.internal`，而要使用任何 Docker 服务容器，我们必须指定 Docker 服务名称。在进行前面的应用程序配置更改后，我们可以继续容器化。为了容器化宠物主人微服务，我们将使用 Jib。在项目的`pom`文件构建设置中进行以下更改：```java<build>  <plugins>    ...    <plugin>      <groupId>com.google.cloud.tools</groupId>      <artifactId>jib-maven-plugin</artifactId>      <version>2.8.0</version>      <configuration>        <from>          <image>openjdk:13-jdk-slim</image>        </from>        <to>          <image>pet-owner-0.1-image</image>        </to>        <container>          <creationTime>${maven.build.timestamp}</creationTime>        </container>      </configuration>    </plugin>  </plugins></build>```在前面的`pom`更改中，我们使用`jib-maven-plugin`来构建容器镜像。`<configuration>`部分指定了 Docker 配置，例如`<from>`镜像（指向使用 JDK 13）。为了给创建的镜像命名，我们使用`<to>`结合`<creationTime>`来正确地在镜像上标记时间坐标。构建镜像的步骤如下：1.  打开终端并切换到`pet-owner`根目录。1.  在终端中输入并运行`mvn compile jib:dockerBuild`命令。1.  等待命令完成。这些说明将创建一个本地 Docker 镜像，可以使用终端中的`docker images | grep pet-owner`命令进行验证：![图 8.3 – 验证本地 Docker 注册表中的宠物主人镜像](img/Figure_8.3_B16585_Fixed.jpg)

图 8.3 – 验证本地 Docker 注册表中的宠物主人镜像

在前面的屏幕截图中，我们可以看到`docker images`的输出。一个名为`pet-owner-0.1-image`的镜像存储在本地 Docker 注册表中。

### 容器化宠物诊所微服务

为了使宠物诊所微服务容器就绪，对`application.properties`文件进行以下更改：

```java
micronaut:
  application:
    name: pet-clinic
  server:
    port: 32582
kafka:
  bootstrap:
    servers: kafka:9092
datasources:
  default:
    url: 
     «jdbc:postgresql://host.docker.internal:5432/postgres»
    username: postgres
    password: postgres
    driverClassName: org.postgresql.Driver
consul:
  client:
    default-zone: "consul:8500"
    registration:
      enabled: true
…
```

在应用程序属性更改中需要思考的以下几点：

+   使用`host.docker.internal`而不是 localhost 来指向主机操作系统（Docker 外部）中安装的 Postgres。

+   `consul`服务。要使用来自宠物诊所 Docker 容器的 Docker 化`consul`，我们需要指定服务名称而不是 localhost。

+   **Kafka 服务器**: 在我们的 Docker 服务中，我们将配置一个**Kafka**服务。

+   **端口**: 我们指定了一个固定的端口来运行宠物诊所微服务，因为我们将在部署过程中暴露这个端口。

在进行前面的应用程序配置更改后，我们可以继续容器化。为了容器化宠物诊所微服务，我们将使用 Jib。在项目的`pom`文件构建设置中进行以下更改：

```java
<build>
  <plugins>
    ...
    <plugin>
      <groupId>com.google.cloud.tools</groupId>
      <artifactId>jib-maven-plugin</artifactId>
      <version>2.8.0</version>
      <configuration>
        <from>
          <image>openjdk:13-jdk-slim</image>
        </from>
        <to>
          <image>pet-clinic-0.1-image</image>
        </to>
        <container>
          <creationTime>${maven.build.timestamp}</creationTime>
        </container>
      </configuration>
    </plugin>
  </plugins>
</build>
```

在前面的`pom`更改中，我们使用`jib-maven-plugin`来构建容器镜像。`<configuration>`部分指定了 Docker 配置，例如`<from>`镜像（指向使用 JDK 13）。为了给创建的镜像命名，我们使用`<to>`结合`<creationTime>`来正确地在镜像上标记时间坐标。

构建镜像的步骤如下：

1.  打开终端并切换到`pet-clinic`根目录。

1.  在终端中输入并运行 `mvn compile jib:dockerBuild` 命令。

1.  等待命令完成。

上述指令将在终端中创建一个本地 Docker 镜像，可以使用 `docker images | grep pet-clinic` 命令进行验证。

### 容器化 pet-clinic-reviews 微服务

要使 pet-clinic-reviews 微服务容器准备就绪，请在 `application.properties` 文件中进行以下更改：

```java
micronaut:
  application:
    name: pet-clinic-reviews
  server:
    port: 32583
kafka:
  bootstrap:
    servers: kafka:9092
mongodb:
  uri: mongodb://mongodb:mongodb@host.docker.internal:27017/pet-clinic-reviews
  databaseName: pet-clinic-reviews
  collectionName: vet-reviews
consul:
  client:
    default-zone: «consul:8500»
    registration:
      enabled: true
…
```

在应用属性更改中，以下是一些需要思考的事项：

+   使用 `host.docker.internal` 而不是 localhost 来指向安装在宿主操作系统（Docker 外部）上的 MongoDB 实例。

+   `consul` 服务。要从 pet-clinic-concierge Docker 容器中使用 Docker 化的 `consul`，我们需要指定服务名称而不是 localhost。

+   `kafka` 服务。要从 `pet-clinic-reviews` Docker 容器中使用 Docker 化的 `consul`，我们需要指定服务名称而不是 localhost。

+   **端口**：我们指定了一个固定端口来运行 pet-clinic-reviews 微服务，因为我们将在部署中稍后公开此端口。

在进行上述应用程序配置更改后，我们可以继续容器化。要容器化 `pet-clinic-reviews` 微服务，我们将使用 Jib。在项目的 `pom` 文件构建设置中进行以下更改：

```java
<build>
  <plugins>
    ...
    <plugin>
      <groupId>com.google.cloud.tools</groupId>
      <artifactId>jib-maven-plugin</artifactId>
      <version>2.8.0</version>
      <configuration>
        <from>
          <image>openjdk:13-jdk-slim</image>
        </from>
        <to>
          <image>pet-clinic-reviews-0.1-image</image>
        </to>
        <container>
          <creationTime>${maven.build.timestamp}</creationTime>
        </container>
      </configuration>
    </plugin>
  </plugins>
</build>
```

在前面的 `pom` 更改中，我们使用 `jib-maven-plugin` 来构建容器镜像。`<configuration>` 部分指定了 Docker 配置，如 `<from>` 镜像（指向使用 JDK 13）。为了给创建的镜像命名，我们使用 `<to>` 结合 `<creationTime>` 来正确地在镜像上标记时间坐标。

要构建镜像，请执行以下步骤：

1.  打开终端并切换到 `pet-clinic-reviews` 根目录。

1.  在终端中输入并运行 `mvn compile jib:dockerBuild` 命令。

1.  等待命令完成。

上述指令将在终端中创建一个本地 Docker 镜像，可以使用 `docker images | grep pet-clinic-reviews` 命令进行验证。

### 容器化 pet-clinic-concierge 微服务

要使 pet-clinic-concierge 微服务容器准备就绪，请在 `application.properties` 文件中进行以下更改：

```java
micronaut:
  application:
    name: pet-clinic-concierge
  server:
    port: 32584
  config-client:
    enabled: true
consul:
  client:
    default-zone: "consul:8500"
    registration:
      enabled: true
…
```

在应用属性更改中，以下是一些需要思考的事项：

+   `consul` 服务。要从 pet-clinic-concierge Docker 容器中使用 Docker 化的 `consul`，我们需要指定服务名称而不是 localhost。

+   `pet-clinic-concierge` 微服务，因为我们将在部署中稍后公开此端口。

在进行上述应用程序配置更改后，我们可以继续容器化。要容器化 `pet-clinic-concierge` 微服务（API 网关），我们将使用 `jib`。在项目的 `pom` 文件构建设置中进行以下更改：

```java
<build>
  <plugins>
    ...
    <plugin>
      <groupId>com.google.cloud.tools</groupId>
      <artifactId>jib-maven-plugin</artifactId>
      <version>2.8.0</version>
      <configuration>
        <from>
          <image>openjdk:13-jdk-slim</image>
        </from>
        <to>
          <image>pet-clinic-concierge-0.1-image</image>
        </to>
        <container>
          <creationTime>${maven.build.timestamp}</creationTime>
        </container>
      </configuration>
    </plugin>
  </plugins>
</build>
```

在前面的`pom`更改中，我们使用`jib-maven-plugin`来构建容器镜像。`<configuration>`部分指定了 Docker 配置，如`<from>`镜像（指向使用 JDK 13）。为了正确标记镜像上的时间坐标，我们使用`<to>`结合`<creationTime>`来命名创建的镜像。

要构建镜像，请执行以下步骤：

1.  打开终端并切换到`pet-clinic-concierge`根目录。

1.  在终端中输入并运行`mvn compile jib:dockerBuild`命令。

1.  等待命令完成。

前面的说明将创建一个本地 Docker 镜像，您可以在终端中使用`docker images | grep pet-clinic-concierge`命令进行验证。

现在，我们已经容器化了所有微服务。我们也可以在 Docker 仪表板中验证这些镜像：

![Figure 8.4 – 在 Docker UI 中验证 Docker 镜像![img/Figure_8.4_B16585_Fixed.jpg](img/Figure_8.4_B16585_Fixed.jpg)

图 8.4 – 在 Docker UI 中验证 Docker 镜像

在 Docker UI 中，我们可以简单地转到**Images**，然后在**LOCAL**选项卡下，我们可以过滤所有 pet-clinic 应用程序的镜像。在下一节中，我们将在 pet-clinic 应用程序的部署中使用这些 Docker 镜像。

# 部署容器工件

在前面的章节中，我们探讨了如何使用`Jib`简化微服务容器化。在本节中，我们将深入了解如何使用`docker-compose`与 Docker 编排相结合，使端到端部署无缝且统一。

## 使用 docker-compose 部署 pet-clinic 服务

`docker-compose`是 Docker 生态系统中的一个工具，它在定义和部署多容器应用程序时非常直观。使用简单的 YAML 风格语法，我们可以设置所有服务及其依赖关系，并使用单个命令部署整个应用程序。我们将为 pet-clinic 应用程序创建一个`docker-compose`文件，涵盖所有必要的服务/组件，包括微服务、服务发现和 Apache Kafka 生态系统。

首先，让我们在`docker-compose`中定义以下辅助服务：

```java
version: '3'
services:
  consul:
    image: bitnami/consul:latest
    ports:
      - '8500:8500'

  zookeeper:
    image: bitnami/zookeeper:3-debian-10
    ports:
      - 2181:2181
    volumes:
      - zookeeper_data:/pet-clinic-reviews
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes  
  kafka:
    image: bitnami/kafka:2-debian-10
    ports:
      - 9094:9094
    volumes:
      - kafka_data:/pet-clinic-reviews
    environment:
      - KAFKA_BROKER_ID=1
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_LISTENERS=INTERNAL://kafka:9092,OUTSIDE://
kafka:9094
      - KAFKA_ADVERTISED_LISTENERS=INTERNAL://kafka:9092,OUTSIDE://localhost:9094
      - KAFKA_LISTENER_SECURITY_PROTOCOL_MAP=INTERNAL:PLAINTEXT,OUTSIDE:PLAINTEXT
      - KAFKA_INTER_BROKER_LISTENER_NAME=INTERNAL
    depends_on:
      - zookeeper
  kafdrop:
    image: obsidiandynamics/kafdrop
    ports:
      - 9100:9000
    environment:
      - KAFKA_BROKERCONNECT=kafka:9092
      - JVM_OPTS=-Xms32M -Xmx64M
    depends_on:
      - kafka
…
```

从前面的代码中，我们看到`docker-compose`文件。我们首先定义一个用于`consul`服务发现的`consul`服务。我们将`consul`暴露在端口`8500`上。此外，我们将定义 Apache Kafka 生态系统的服务；即 Zookeeper、Kafka 和 Kafdrop UI。一旦这些服务在`docker-compose`文件中定义，我们就可以继续处理`pet-clinic`微服务。请参考以下代码：

```java
…  
pet-owner:
    image: "pet-owner-0.1-image"
    ports:
        - "32581:32581"
    depends_on:
        - consul

  pet-clinic:
    image: "pet-clinic-0.1-image"
    ports:
        - "32582:32582"
    depends_on:
        - kafka
        - consul

  pet-clinic-reviews:
    image: "pet-clinic-reviews-0.1-image"
    ports:
        - "32583:32583"
    depends_on:
        - kafka
        - consul

  pet-clinic-concierge:
    image: "pet-clinic-concierge-0.1-image"
    ports:
        - "32584:32584"
    depends_on:
        - consul    
…
```

在定义`pet-clinic`微服务的配置时，我们可以使用`depends_on`来指定依赖关系。这将确保 Docker 按照依赖顺序实例化服务。此外，对于部署每个服务，我们将使用`pet-clinic`微服务的 Docker 镜像。

一旦为`pet-clinic`应用程序定义了`docker-compose`文件，请参考以下说明来部署`pet-clinic`应用程序：

1.  打开`bash`终端。

1.  将目录更改为存储`docker-compose`文件的位置。

1.  输入并运行`docker compose up`命令。

1.  等待 Docker 根据`docker-compose`文件指定的容器实例化。

在成功运行`docker-compose`命令后，我们可以在 Docker 仪表板上验证`pet-clinic`应用程序，如下所示：

![图 8.5 – 在 Docker 仪表板上验证宠物诊所的部署](img/Figure_8.5_B16585_Fixed.jpg)

图 8.5 – 在 Docker 仪表板上验证宠物诊所的部署

在前面的截图上，你可以看到宠物诊所应用程序中所有服务的状态。你可以点击一个服务来监控日志和访问 Web 界面（如果有）。此外，我们可以检查`consul`服务发现以了解宠物诊所微服务的健康状况。参看以下截图：

![图 8.6 – 在服务发现中验证微服务健康](img/Figure_8.6_B16585.jpg)

图 8.6 – 在服务发现中验证微服务健康

在`consul`服务发现中，我们可以观察到每个微服务实例的健康状况。在前面的截图上，我们可以看到所有`pet-clinic`微服务都在正常运行。

微服务容器编排是任何部署策略的基础。在本章的范围内，我们涵盖了宠物诊所应用程序的本地 Docker 部署，但构建的容器镜像可以在任何地方部署，无论是本地还是云环境，使用容器编排工具如`docker-compose`。

# 摘要

在本章中，我们通过讨论使 Micronaut 微服务容器就绪来启动我们的理解。随后，我们深入探讨了使用 Jib 为每个微服务创建容器镜像。我们看到了如何使用`docker-compose`定义所有服务容器配置，并使用单个命令无缝部署所有必需的服务组件。

本章通过为您提供容器化和自动化部署的第一手知识，增强了您的 Micronaut 微服务之旅的部署方面。这一技能集在微服务应用程序的开发和维护中非常受欢迎。

在下一章中，我们将探讨在 Micronaut 中监控宠物诊所应用程序不同方面的各种方法。

# 问题

1.  什么是 Jib？

1.  我们如何使用 Jib 在 Micronaut 中创建 Docker 容器？

1.  我们如何从 Micronaut 的 Docker 容器连接到 localhost？

1.  我们如何使用`docker-compose`在 Micronaut 中部署多服务应用程序？

1.  我们如何执行 Micronaut 微服务应用程序的 Docker 容器化？
