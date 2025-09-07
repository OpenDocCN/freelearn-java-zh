# 12

# 前方的地平线

随着云计算技术以快速的速度不断发展，开发者和组织必须保持领先，为下一波创新做好准备。本章将探讨云计算领域的兴起趋势和进步，特别关注 Java 在塑造这些未来发展中扮演的角色。

我们将首先探讨无服务器 Java 的演变，其中像 Quarkus 和 Micronaut 这样的框架正在重新定义**函数即服务**的边界。这些工具利用创新技术，如原生图像编译，以在无服务器环境中提供前所未有的性能和效率。此外，我们还将深入研究无服务器容器的概念，它允许以无服务器的方式部署整个 Java 应用程序，利用 Kubernetes 和**亚马逊网络服务**（**AWS**）Fargate 等容器编排平台的优势。

接下来，我们将探讨 Java 在新兴的边缘计算范式中的作用。随着数据处理和决策越来越接近源头，Java 的平台独立性、性能和广泛的生态系统使其成为构建边缘应用的理想选择。我们将讨论使 Java 开发者能够利用边缘计算架构的强大功能的键框架和工具。

此外，我们还将调查 Java 在云生态系统内人工智能（**AI**）和机器学习（**ML**）集成中的演变位置。从无服务器 AI/ML 工作流到 Java 与基于云的 AI 服务的无缝集成，我们将探讨这种融合带来的机会和挑战。

最后，我们将深入探讨迷人的**量子计算**领域，这个领域承诺将彻底改变各个行业。虽然仍处于早期阶段，但了解量子计算的基本原理，如量子比特、量子门和算法，可以为开发者准备未来的进步及其与基于 Java 的应用程序的潜在集成。

到本章结束时，你将全面了解云计算的兴起趋势以及 Java 在塑造这些创新中的关键作用。你将具备知识和实际示例，以定位你的应用程序和基础设施，在快速发展的云计算领域中取得成功。

本章将涵盖以下关键主题：

+   云计算的未来趋势和 Java 的角色

+   边缘计算和 Java

+   人工智能和机器学习集成

+   Java 中新兴的并发和并行处理工具

+   准备迎接下一波云计算创新

那么，让我们开始吧！

# 技术要求

为了充分参与*第十二章*的内容和示例，请确保以下内容已安装并配置：

+   **Java 开发工具包**（**JDK**）：

    +   Quarkus 需要 JDK 来运行。如果您没有，请从官方源下载并安装最新版本（推荐使用 JDK 17 或更高版本）：

        +   **AdoptOpenJDK**: [`adoptium.net/`](https://adoptium.net/)

        +   **OpenJDK**: [`openjdk.org/`](https://openjdk.org/)

+   `choco install quarkus`) 或 Scoop (`scoop` `install quarkus`)

+   或者，使用 JBang (`jbang app install --``fresh quarkus@quarkusio`)

+   **Quarkus CLI 安装** **指南**: [`quarkus.io/guides/cli-tooling`](https://quarkus.io/guides/cli-tooling)

+   将`GRAALVM_HOME`环境变量设置为 GraalVM 安装目录。*   将`%GRAALVM_HOME%\bin`添加到您的 PATH 环境变量中。*   **Docker Desktop**：

    1.  从[`www.docker.com/products/docker-desktop/`](https://www.docker.com/products/docker-desktop/)下载并安装 Windows 版的 Docker Desktop。

    1.  按照安装向导进行操作，并根据需要配置 Docker。

本章中的代码可以在 GitHub 上找到：

[`github.com/PacktPublishing/Java-Concurrency-and-Parallelism`](https://github.com/PacktPublishing/Java-Concurrency-and-Parallelism)

# 云计算的未来趋势及 Java 的角色

随着云计算的持续发展，几个新兴趋势正在塑造这一技术领域的未来。边缘计算、AI 和 ML 的集成以及无服务器架构等创新处于前沿，推动新的可能性和效率。Java 凭借其强大的生态系统和持续进步，在这些发展中扮演着关键角色。本节将探讨云计算的最新趋势，Java 如何适应并促进这些变化，并提供 Java 在尖端云计算技术中的实际应用案例。

## 云计算的新兴趋势 – 无服务器 Java 超越函数即服务

云计算的新兴趋势正在重塑无服务器 Java 的格局，超越了传统的函数即服务模型。Quarkus 和 Micronaut 等无服务器 Java 框架的创新正在推动这一演变。

### Quarkus

**Quarkus**，因其微服务方面的优势而闻名，现在正在无服务器环境中产生重大影响。它赋予开发者构建遵循微服务原则的无服务器函数的能力，无缝地融合了这两种架构方法。一个突出的特性是 Quarkus 与 GraalVM 的原生集成，使得可以将 Java 应用程序编译成原生可执行文件。这对于无服务器计算来说是一个变革，因为它解决了长期存在的冷启动延迟问题。通过利用 GraalVM，Quarkus 显著减少了 Java 应用程序的启动时间，通常从秒级减少到仅仅毫秒级，与传统 **Java 虚拟机**（**JVM**）基于的替代方案相比。此外，生成的原生二进制文件更节省内存，有助于在无服务器环境的动态世界中实现优化的扩展和资源利用。这些进步正在改变无服务器 Java，为开发者提供了一套强大的工具集，用于创建高效、响应迅速的高性能云原生应用程序。

### Micronaut

**Micronaut** 是另一个在无服务器 Java 领域取得显著进展的创新框架。它通过几个关键特性设计用于优化微服务和无服务器应用程序的性能：

+   **编译时依赖注入**：与在运行时解决依赖的传统框架不同，Micronaut 在编译时执行这项任务。这种方法消除了运行时反射的需求，从而实现了更快的启动时间和更低的内存消耗。

+   **面向方面编程（AOP**）：AOP 是一种编程范式，通过允许分离横切关注点来提高模块化。在 Micronaut 中，AOP 是在编译时而不是在运行时实现的。这意味着事务管理、安全性和缓存等特性在编译期间被编织到字节码中，消除了运行时代理的需求，并进一步减少了内存使用和启动时间。

这些编译时技术使 Micronaut 成为构建轻量级、快速和高效无服务器应用程序的理想选择。该框架的设计特别适合于快速启动和低资源消耗至关重要的环境。

此外，Micronaut 支持创建 GraalVM 原生镜像。这一特性通过最小化冷启动时间和资源使用，进一步增强了其在无服务器环境中的适用性，因为原生镜像可以几乎瞬间启动，并且相比传统的基于 JVM 的应用程序消耗更少的内存。

### 无服务器容器和 Java 应用程序

服务器无服务器容器代表了无服务器计算的一个新维度，它使得整个 Java 应用的部署成为可能，而不仅仅是单个函数。这种方法利用了 Kubernetes 和 AWS Fargate 等容器编排平台以无服务器的方式运行容器。打包为容器的 Java 应用可以享受与无服务器函数相同的无服务器优势，如自动扩展和按使用付费定价，但与传统无服务器函数相比，对运行时环境有更多的控制。开发者可以通过将应用程序及其依赖项打包在一起来确保不同环境之间的一致性。对运行时环境的完全控制允许包含必要的库和工具，提供传统无服务器函数中有时缺乏的灵活性。此外，无服务器容器可以根据需求自动扩展，提供无服务器计算的好处，同时保持容器化应用的稳健性。

通过结合 Quarkus 和 Micronaut 等无服务器 Java 框架的创新与无服务器容器的灵活性，开发者可以创建高度可扩展、高效且响应迅速的 Java 应用，以满足现代云原生环境的需求。这些进步正在为下一代无服务器 Java 铺平道路，它超越了简单的函数，涵盖了完整的应用和服务。

#### 示例用例 - 使用 Quarkus 和 GraalVM 构建无服务器 REST API

**目标**：创建一个用于产品管理的无服务器 REST API，并使用 Quarkus 在 AWS Lambda 上部署它，展示 Quarkus 的关键特性和与 AWS 服务的集成。

本例涵盖了 Quarkus 的关键概念和元素。完整的应用程序将在 GitHub 仓库中提供。

1.  **设置项目**：使用 Quarkus CLI 或 Maven 创建一个新的项目。在这个例子中，我们将使用 Maven。运行以下 Maven 命令以创建 Quarkus 项目：

    ```java
    mvn io.quarkus:quarkus-maven-plugin:2.7.5.Final:create \
        -DprojectGroupId=com.example \
        -DprojectArtifactId=quarkus-serverless \
        -DclassName="com.example.ProductResource" \
    ProductResource class is a RESTful resource that defines the endpoints for managing products within the application. Using JAX-RS annotations, it provides methods for retrieving all products, fetching the count of products, and getting details of individual products by ID. This class serves as the primary interface for client interactions with the product-related data in the application. It demonstrates Quarkus features such as dependency injection, metrics, and OpenAPI documentation:

    ```

    @Path("/api/products")

    @Produces(MediaType.APPLICATION_JSON)

    @Consumes(MediaType.APPLICATION_JSON)

    @Tag(name = "Product",

    description = "产品管理操作")

    public class ProductResource {

    @Inject

    ProductRepository productRepository;

    @GET

    @Counted(name = "getAllProductsCount",

    description = "getAllProducts 被调用的次数")

    @Timed(name = "getAllProductsTimer",

    description = "衡量执行 getAllProducts 所需时间的指标",

    unit = MetricUnits.MILLISECONDS)

    @Operation(summary = "获取所有产品",

    description = "返回所有产品的列表，带有分页和排序")

    public Response getAllProducts(@QueryParam(

    "page") @DefaultValue("0") int page,

    @QueryParam("size") @DefaultValue("20") int size,

    @QueryParam("sort") @DefaultValue("name") String sort) {

    // 省略实现以节省篇幅

    }

    @POST

    @Operation(summary = "创建一个新的产品",

    description = "创建一个新的产品并返回创建的产品")

    public Response createProduct(Product product) {

    // 省略实现以节省篇幅

    }

    // 省略其他 CRUD 方法以节省篇幅

    }

    ```java

    ```

1.  `ProductRepository`：`ProductRepository` 类充当数据访问层，管理与 *AWS DynamoDB* 的交互以实现产品数据持久化。它展示了 Quarkus 与 AWS `DynamoDbClient` 的无缝集成，展示了 Quarkus 如何简化云服务集成。它实现了 **创建、读取、更新和删除** （**CRUD**） 操作的方法，在 Java 对象和 DynamoDB 项目表示之间进行转换，从而展示了 Quarkus 应用程序如何在云环境中高效地与 NoSQL 数据库协同工作：

    ```java
    @ApplicationScoped
    public class ProductRepository {
        @Inject
        DynamoDbClient dynamoDbClient;
        private static final String TABLE_NAME = "Products";
        public void persist(Product product) {
            Map<String, AttributeValue> item = new HashMap<>();
            item.put("id", AttributeValue.builder().s(
                product.getId()).build());
            item.put("name", AttributeValue.builder().s(
                product.getName()).build());
            // Add other attributes
            PutItemRequest request = PutItemRequest.builder()
                    .tableName(TABLE_NAME)
                    .item(item)
                    .build();
            dynamoDbClient.putItem(request);
        }
        // Additional methods omitted for brevity
    }
    ```

1.  `ImageAnalysisCoordinator`：`ImageAnalysisCoordinator` 类展示了 Quarkus 创建与多个 AWS 服务交互的 AWS Lambda 函数的能力。它展示了处理 **简单存储服务** （**S3**） 事件和触发 **弹性容器服务** （**ECS**） 任务，说明了 Quarkus 可以用于构建复杂的事件驱动架构。该类使用依赖注入来处理 AWS 客户端（ECS 和 S3），展示了 Quarkus 如何简化在单个组件中与多个云服务协同工作。它是使用 Quarkus 为无服务器应用程序编排其他 AWS 服务的优秀示例：

    ```java
    @ApplicationScoped
    public class ImageAnalysisCoordinator implements RequestHandler<S3Event, String> {
        @Inject
        EcsClient ecsClient;
        @Inject
        S3Client s3Client;
        @Override
        public String handleRequest(S3Event s3Event,
            Context context) {
                String bucket = s3Event.getRecords().get(
                    0).getS3().getBucket().getName();
                String key = s3Event.getRecords().get(
                    0).getS3().getObject().getKey();
            RunTaskRequest runTaskRequest = RunTaskRequest.builder()
                .cluster("your-fargate-cluster")
                .taskDefinition("your-task-definition")
                .launchType("FARGATE")
                .overrides(TaskOverride.builder()
                    .containerOverrides(
                        ContainerOverride.builder()
                            .name("your-container-name")
                            .environment(
                                KeyValuePair.builder()
                                    .name("BUCKET")
                                    .value(bucket)
                                    .build(),
                                KeyValuePair.builder()
                                    .name("KEY")
                                    .value(key)
                                    .build())
                                .build())
                            .build())
                    .build();
            // Implementation omitted for brevity
        }
    }
    ```

1.  `ProductHealthCheck`：`ProductHealthCheck` 类实现了 Quarkus 的健康检查机制，这对于在云环境中维护应用程序可靠性至关重要。它展示了 Microprofile Health 的使用，允许应用程序向编排系统（如 Kubernetes）报告其状态。该类检查 DynamoDB 表的可访问性，展示了 Quarkus 应用程序如何提供有关外部依赖的有意义健康信息。该组件对于实现能够自我报告其操作状态的健壮微服务至关重要：

    ```java
    @Readiness
    @ApplicationScoped
    public class ProductHealthCheck implements HealthCheck {
        @Inject
        DynamoDbClient dynamoDbClient;
        private static final String TABLE_NAME = "Products";
        @Override
        public HealthCheckResponse call() {
            HealthCheckResponseBuilder responseBuilder =         HealthCheckResponse.named(
                "Product service health check");
            try {
                dynamoDbClient.describeTable(DescribeTableRequest.            builder()
                        .tableName(TABLE_NAME)
                        .build());
                return responseBuilder.up()
                        .withData("table", TABLE_NAME)
                        .withData("status", "accessible")
                        .build();
            } catch (DynamoDbException e) {
                return responseBuilder.down()
                        .withData("table", TABLE_NAME)
                        .withData("status",
                            "inaccessible")
                        .withData("error", e.getMessage())
                        .build();
            }
        }
    }
    ```

1.  `pom.xml` 文件包含一个用于原生构建的配置文件。这将指定 GraalVM 所需的依赖项和插件：

    ```java
    <profiles>
        <profile>
            <id>native</id>
            <activation>
                <property>
                    <name>native</name>
                </property>
            </activation>
                <properties>
                    <skipITs>false</skipITs>
                    <quarkus.package.type>native</quarkus.package.type>
                    <quarkus.native.enabled>true</quarkus.native.enabled>
                </properties>
            </profile>
        </profiles>
    ```

1.  `Dockerfile.native`：提供的 Dockerfile 对于使用 GraalVM 构建和打包 Quarkus 应用程序以部署到 AWS Lambda 是必需的。它首先使用 GraalVM 镜像将应用程序编译成原生可执行文件，确保最佳性能和最短的启动时间。构建阶段包括复制项目文件和运行 Maven 构建过程。随后，运行时阶段使用最小的基础镜像以保持最终镜像轻量。编译后的原生可执行文件从构建阶段复制到运行时阶段，在那里它被设置为容器的入口点。这种设置确保了在无服务器环境中的部署过程流畅且高效：

    ```java
    # Start with a GraalVM image for native building
    FROM quay.io/quarkus/ubi-quarkus-native-image:21.0.0-java17 AS build
    COPY src /usr/src/app/src
    COPY pom.xml /usr/src/app
    USER root
    RUN chown -R quarkus /usr/src/app
    USER quarkus
    RUN mvn -f /usr/src/app/pom.xml -Pnative clean package
    FROM registry.access.redhat.com/ubi8/ubi-minimal
    WORKDIR /work/
    COPY --from=build /usr/src/app/target/*-runner /work/application
    RUN chmod 775 /work/application
    EXPOSE 8080
    CMD ["./application", "-Dquarkus.http.host=0.0.0.0"]
    ```

此 Dockerfile 描述了一个用于 Quarkus 原生应用程序的两个阶段构建过程：

1.  **构建阶段**:

    +   使用基于 GraalVM 的镜像来编译应用程序

    +   复制项目文件并构建原生可执行文件

1.  **运行时阶段**：

    +   使用最小化的 Red Hat UBI 作为基础镜像

    +   从构建阶段复制原生可执行文件

    +   将可执行文件设置为入口点

多阶段构建有以下优点：

+   **较小的镜像大小**：最终镜像精简，仅包含必要的运行时依赖项

+   **改进安全性**：通过包含更少的工具和包来减少攻击面

+   **清晰的分离**：通过将构建环境与运行时环境分离来简化维护

苹果硅用户注意事项

当在苹果硅（M1 或 M2）设备上构建 Docker 镜像时，您可能会遇到由于默认的 **高级精简指令集架构**（**ARM**）导致的兼容性问题。大多数云环境，包括 AWS、Azure 和 Google Cloud，使用 AMD64（x86_64）架构。为了避免这些问题，在构建 Docker 镜像时指定目标平台，以确保兼容性。

在苹果硅设备上构建 Docker 镜像时指定 `--platform` 参数，以确保与云环境兼容。

例如，使用以下命令构建与 AMD64 架构兼容的镜像：

```java
docker build --platform linux/amd64 -t myapp:latest .
```

虽然 `application.properties` 文件不是直接用于启用原生构建，但您可以包含属性以优化应用程序作为原生镜像运行。以下是一个示例 `application.properties` 文件：

```java
# you can include properties to optimize the application for running as a native image:
# Disable reflection if not needed
quarkus.native.enable-http-url-handler=true
# Native image optimization
quarkus.native.additional-build-args=-H:+ReportExceptionStackTraces
# Example logging configuration for production
%prod.quarkus.log.console.level=INFO
```

**部署到** **AWS Lambda**：

`Template.yaml`：一个 AWS **无服务器应用程序模型**（**SAM**）模板，它定义了基于 Quarkus 的 Lambda 函数的基础设施，指定其运行时环境、处理程序、资源分配和必要的权限：

```java
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: >
    quarkus-serverless
Resources:
    QuarkusFunction:
        Type: AWS::Serverless::Function
        Properties:
            Handler: com.example.LambdaHandler
            Runtime: provided.al2
            CodeUri: s3://your-s3-bucket-name/your-code.zip
            MemorySize: 128
            Timeout: 15
            Policies:
                - AWSLambdaBasicExecutionRole
```

`.``jar` 文件：

```java
mvn clean package -Pnative -Dquarkus.native.container-build=true
```

**使用 SAM CLI 部署**：使用 AWS SAM CLI 打包和部署基于 Quarkus 的 Lambda 函数：第一个命令打包应用程序并将其上传到 Amazon S3 桶，而第二个命令将打包的应用程序部署到 AWS，创建或更新一个包含必要资源和权限的 CloudFormation 堆栈：

```java
sam package --output-template-file packaged.yaml --s3-bucket your-s3-bucket-name
sam deploy --template-file packaged.yaml --stack-name quarkus-serverless --capabilities CAPABILITY_IAM
```

通过遵循这些步骤，您将成功构建一个使用 Quarkus 的无服务器 REST API，将其打包为带有 GraalVM 的原生镜像，并部署到 AWS Lambda。这种设置确保了最佳性能，并减少了无服务器应用程序的冷启动时间。

无服务器范式正在不断发展，Java 框架如 Quarkus 领先于优化云原生、无服务器环境。正如我们所见，现代无服务器 Java 应用程序可以利用快速启动时间、低内存占用和与云服务无缝集成的先进功能。这使开发者能够构建复杂、可扩展的应用程序，这些应用程序远远超出了简单函数执行的范围，涵盖了完整的微服务架构。

随着云计算领域的持续演变，另一个新兴趋势正在获得显著的吸引力：边缘计算。让我们探讨 Java 如何适应边缘计算环境所提出的独特挑战和机遇。

# 边缘计算与 Java

边缘计算代表了数据处理范式的一次转变，计算发生在数据源附近或数据源处，而不是仅仅依赖于集中的云数据中心。这种方法减少了延迟，优化了带宽使用，并提高了关键应用的响应时间。

## Java 在边缘计算架构中的作用

Java，凭借其成熟的生态系统和强大的性能，正日益成为边缘计算架构中的关键玩家。

Java 的通用性和平台独立性使其成为边缘计算环境的理想选择，这些环境通常由异构硬件和**操作系统**（**OSs**）组成。Java 能够在各种设备上运行，从强大的服务器到受限的**物联网**（**IoT**）设备，确保开发者可以在整个边缘到云的连续体上利用一致的编程模型。此外，Java 生态系统中可用的广泛库和框架使得边缘应用的快速开发和部署成为可能。

在边缘计算中使用 Java 的关键优势包括以下内容：

+   **跨平台兼容性**：Java 的“一次编写，到处运行”理念允许边缘应用在多种硬件平台上部署而无需修改

+   **性能和可伸缩性**：Java 强大的性能和高效的内存管理对于处理边缘设备中常见的资源受限环境至关重要

+   **安全性**：Java 提供强大的安全模型，这对于保护边缘处理敏感数据至关重要

这些优势使 Java 成为边缘计算的有力选择。为了进一步赋能开发者，已经开发出几个框架和工具，以简化基于 Java 的边缘应用的开发和部署。

## 用于基于 Java 的边缘应用的框架和工具

为了有效地利用 Java 进行边缘计算，开发者可以利用专门为构建和管理边缘应用而设计的各种框架和工具。以下是一些突出的框架和工具：

+   **Eclipse 基金会**的**物联网倡议**：

    +   **Eclipse Kura**：一个用于构建物联网网关的开源框架。它提供了一套 Java API，用于访问硬件接口、管理网络配置以及与云服务交互。

    +   **Eclipse Kapua**：一个模块化的物联网云平台，与 Eclipse Kura 协同工作，提供端到端的物联网解决方案。它提供设备管理、数据管理和应用集成等功能。

+   **Apache Edgent**：Apache Edgent（之前称为 Quarks）是一个轻量级、可嵌入的编程模型和运行时，适用于边缘设备。它允许开发者创建可以在小尺寸设备上运行并集成到中央数据系统的分析应用程序。

+   **Vert.x**：Vert.x 是一个用于在 JVM 上构建反应式应用程序的工具包。其事件驱动架构和轻量级特性使其非常适合需要低延迟和高并发的边缘计算场景。

+   **AWS IoT Greengrass**：AWS IoT Greengrass 将 AWS 的能力扩展到边缘设备，使它们能够在本地处理它们生成的数据，同时仍然使用云进行管理、分析和持久存储。Java 开发者可以创建 Greengrass Lambda 函数来处理和响应本地事件。

+   **Azure IoT Edge**：Azure IoT Edge 允许开发者将容器化应用程序部署和运行在边缘。Java 应用程序可以打包在 Docker 容器中，并使用 Azure IoT Edge 运行时进行部署，从而实现与 Azure 云服务的无缝集成。

+   **Google Cloud IoT Edge**：Google Cloud IoT Edge 将 Google Cloud 的 ML 和数据处理能力带到边缘设备。Java 开发者可以利用 TensorFlow Lite 和其他 Google Cloud 服务来创建智能边缘应用程序。

Java 的强大生态系统、平台独立性和广泛的库支持使其成为边缘计算的强劲竞争者。通过利用为边缘环境设计的框架和工具，Java 开发者可以构建高效、可扩展且安全的边缘应用程序，充分利用边缘计算架构的潜力。随着边缘计算不断演进，Java 在塑造分布式和去中心化数据处理未来方面处于有利位置。

# AI 和 ML 集成

当我们展望 Java 在云计算的未来时，AI 和 ML 的集成带来了令人兴奋的机会和挑战。虽然第七章专注于 Java 在 ML 工作流程中的并发机制，但本节探讨了 Java 在基于云的 AI/ML 生态系统中的演变角色及其与高级云 AI 服务的集成。

## Java 在基于云的 AI/ML 工作流程中的位置

以下是 Java 在基于云的 AI/ML 生态系统中的演变角色：

+   **无服务器 AI/ML 与 Java**：Java 在基于云的 AI/ML 工作流程中的未来越来越趋向于无服务器。如 AWS Lambda 和 Google Cloud Functions 之类的框架允许开发者将 AI/ML 模型作为无服务器函数部署。这一趋势预计将增长，使 AI/ML 操作更加高效和可扩展，无需管理基础设施。

+   **Java 作为编排器**：Java 正在将自己定位为云中复杂 AI/ML 管道的高效编排器。其稳健性和广泛的生态系统使其非常适合管理涉及多个 AI/ML 服务、数据源和处理步骤的工作流程。预计将出现更多针对云环境中 AI/ML 管道编排设计的基于 Java 的工具和框架。

+   **Java 与边缘 AI**：随着边缘计算的兴起，Java 的“一次编写，到处运行”的理念变得越来越有价值。Java 正在适应边缘 AI 应用程序，允许在云中训练的模型部署并在边缘设备上运行。这一趋势可能会加速，Java 将作为云端训练和边缘端推理之间的桥梁。

接下来，让我们探索 Java 与高级基于云的 AI 服务的集成。

## Java 与云 AI 服务的集成

将 Java 应用程序与基于云的 AI 服务集成，为开发者打开了无限可能，使他们能够创建智能和自适应的软件解决方案。云 AI 服务提供预训练模型、可扩展的基础设施和 API，使得在不需大量内部专业知识的情况下实现高级机器学习和 AI 功能变得更加容易。以下是可以与 Java 应用程序集成的流行云 AI 服务列表：

+   **云 AI 服务的原生 Java SDK**：主要云服务提供商正在投资开发用于其 AI 服务的强大 **Java 开发工具包**（**JDK**）。例如，AWS 已经发布了 AWS SDK for Java 2.0，它提供了对 Amazon SageMaker 等服务的简化访问。Google Cloud 也增强了其 AI 和 ML 服务的 Java 客户端库。这一趋势预计将持续，使 Java 开发者更容易将云 AI 服务集成到他们的应用程序中。

+   **Java 友好的 AutoML 平台**：云服务提供商正在开发越来越友好的 AutoML 平台。例如，Google Cloud AutoML 现在提供 Java 客户端库，允许 Java 应用程序轻松训练和部署定制的 ML 模型，而无需广泛的 ML 专业知识。这一趋势可能会扩展，使高级 AI 功能更容易为 Java 开发者所获取。

+   **容器化的 Java AI/ML 部署**：Java 在云 AI/ML 工作流程中的未来与容器化紧密相连。Kubernetes 等平台正在成为在云中部署和管理 AI/ML 工作负载的事实标准。Java 与容器化技术的兼容性使其非常适合这一趋势。预计将出现更多工具和最佳实践，用于在容器化环境中部署基于 Java 的 AI/ML 应用程序。

+   **联邦学习中的 Java**：联邦学习是一种机器学习技术，它在不交换数据的情况下，在多个持有本地数据样本的分散边缘设备或服务器上训练算法。这种方法通过允许在分布式数据集上训练模型而不集中汇总数据，解决了日益增长的隐私问题。

    随着隐私问题的日益突出，联邦学习正在获得关注。Java 的强大安全特性和其在企业环境中的广泛采用使其成为实现联邦学习系统的有力候选者。云提供商可能会在其联邦学习产品中提供更多对 Java 的支持，使模型能够在不损害隐私的情况下跨分散数据源进行训练。

+   **机器学习操作 (MLOps) 中的 Java**：新兴的 MLOps 领域正在越来越多地采用 Java。其稳定性和广泛的工具使得 Java 非常适合在云中构建健壮的 MLOps 管道。预计将看到更多基于 Java 的 MLOps 工具和与云 CI/CD 服务的集成，这些服务专门为 AI/ML 工作流程设计。

总之，Java 在基于云的 AI/ML 中的作用正在从仅仅是一种实现算法的语言转变为更广泛的 AI/ML 生态系统的一部分。它正成为云中从无服务器部署到边缘计算，从 AutoML 到 MLOps 的关键部分。随着云 AI 服务的不断成熟，Java 与这些服务的集成将更加深入，为开发者提供构建智能、可扩展云应用程序的强大新方法。

## 用例 - 使用 AWS Lambda 和 Fargate 的无服务器 AI 图像分析

本用例展示了使用 AWS Lambda 和 Fargate 的可扩展、无服务器架构，用于基于 AI 的图像分析。AWS Fargate 是 AWS 对无服务器容器的实现。这项技术允许以无服务器方式部署整个 Java 应用程序，利用 Kubernetes 和 AWS Fargate 等容器编排平台。通过将 Java 应用程序打包为容器，开发者可以享受无服务器计算的好处——如自动扩展和按使用付费定价——同时保持对运行时环境的控制。这种方法确保了不同环境之间的一致性，通过包含必要的库和工具提供了灵活性，并提供了强大的可伸缩性。

该系统由两个主要组件组成，每个组件都作为一个独立的 Quarkus 项目构建：

+   `ImageAnalysisCoordinator`：

    +   作为无服务器环境中的原生可执行文件构建，以实现最佳性能

    +   当图像上传到 S3 桶时触发

    +   使用 Amazon Rekognition 进行快速分析

    +   通过启动 AWS Fargate 任务来启动更详细的分析

+   `FargateImageAnalyzer`：

    +   作为基于 JVM 的应用程序构建，并使用 Docker 容器化

    +   当 Lambda 函数触发时，作为 AWS Fargate 中的任务运行

    +   使用先进的 AI 技术进行深入图像处理

    +   将详细分析结果存储回 S3

这种双组件架构允许高效地利用资源：轻量级的 Lambda 函数处理初始处理和编排，而 Fargate 容器管理更密集的计算任务。它们共同构成了一个强大、可扩展的无服务器 AI 图像分析解决方案。

**步骤 1：创建一个** **Fargate 容器**：

`Dockerfile.jvm`: `Dockerfile.jvm`用于构建服务器端 AI 图像分析架构中 Fargate 容器组件的 Docker 镜像。与作为本地可执行文件构建的 Lambda 函数不同，Fargate 容器以基于 JVM 的 Quarkus 应用程序运行`FargateImageAnalyzer`应用程序。这种选择是因为 Fargate 容器负责更计算密集型的图像处理任务，Quarkus 框架的好处可能超过本地可执行文件潜在的性能优势。

此 Dockerfile 定义了构建 Quarkus 应用程序 Docker 镜像的步骤。该镜像设计为在带有 OpenJDK 17 的 Red Hat **通用基础镜像**（**UBI**）环境中运行：

```java
FROM registry.access.redhat.com/ubi8/openjdk-17:1.14
ENV LANGUAGE='en_US:en'
# We make four distinct layers so if there are application changes the library layers can be re-used
COPY --chown=185 target/quarkus-app/lib/ /deployments/lib/
COPY --chown=185 target/quarkus-app/*.jar /deployments/
COPY --chown=185 target/quarkus-app/app/ /deployments/app/
COPY --chown=185 target/quarkus-app/quarkus/ /deployments/quarkus/
EXPOSE 8080
USER 185
ENV JAVA_OPTS="-Dquarkus.http.host=0.0.0.0 -Djava.util.logging.manager=org.jboss.logmanager.LogManager"
ENV JAVA_APP_JAR="/deployments/quarkus-run.jar"
ENTRYPOINT [ "/opt/jboss/container/java/run/run-java.sh" ]
```

`FargateImageAnalyzer.java`执行深入图像处理：

```java
@QuarkusMain
@ApplicationScoped
public class FargateImageAnalyzer implements QuarkusApplication {
    @Inject
    S3Client s3Client;
    @Inject
    RekognitionClient rekognitionClient;
    @Override
    public int run(String... args) throws Exception {
        String bucket = System.getenv("IMAGE_BUCKET");
        String key = System.getenv("IMAGE_KEY");
        try {
            DetectLabelsRequest labelsRequest = DetectLabelsRequest.            builder()
                    .image(Image.builder().s3Object(
                        S3Object.builder().bucket(
                            bucket).name(key).build()
                            ).build())
                    .maxLabels(10)
                    .minConfidence(75F)
                    .build();
            DetectLabelsResponse labelsResult = rekognitionClient.            detectLabels(labelsRequest);
            DetectFacesRequest facesRequest = DetectFacesRequest.            builder()
                    .image(Image.builder().s3Object(
                        S3Object.builder().bucket(
                            bucket).name(key).build()
                            ).build())
                    .attributes(Attribute.ALL)
                    .build();
            DetectFacesResponse facesResult = rekognitionClient.            detectFaces(facesRequest);
            String analysisResult =             generateAnalysisResult(labelsResult, facesResult);
            s3Client.putObject(builder -> builder
                    .bucket(bucket)
                    .key(key + "_detailed_analysis.json")
                    .build(),
                    RequestBody.fromString(analysisResult));
        } catch (Exception e) {
            System.err.println("Error processing image: " +             e.getMessage());
            return 1;
        }
        return 0;
    }
    private String generateAnalysisResult(
        DetectLabelsResponse labelsResult, DetectFacesResponse         facesResult) {
            // Implement result generation logic
            return "Analysis result";
    }
}
```

`FargateImageAnalyzer`类是作为无服务器 AI 图像分析架构一部分在 Fargate 容器内运行的主要应用程序。它被设计为 Quarkus 应用程序并实现了`QuarkusApplication`接口。该类负责提取 S3 存储桶和对象键信息，使用 AWS Rekognition 客户端执行图像分析，生成详细的分析结果，并将其存储回同一 S3 存储桶。它被设计为在 Fargate 任务中作为独立的 Quarkus 应用程序运行，利用容器化环境运行的好处以及 Fargate 提供的部署和扩展的便利性。

**步骤 2：创建一个** **Lambda 函数**：

`Dockerfile.native`: 这个 Dockerfile 用于构建服务器端 AI 图像分析架构中 Lambda 函数组件的本地可执行 Docker 镜像。此 Dockerfile 遵循 Quarkus 构建本地可执行文件的约定，通过使用`quay.io/quarkus/ubi-quarkus-native-image`作为基础镜像并执行必要的构建步骤。使用`Dockerfile.native`可以将 Lambda 函数打包为本地可执行文件，与基于 JVM 的部署相比，这提供了改进的性能和减少的冷启动时间。这对于需要快速响应时间的无服务器应用程序特别有益：

```java
FROM quay.io/quarkus/ubi-quarkus-native-image:21.0.0-java17 AS build
COPY src /usr/src/app/src
COPY pom.xml /usr/src/app
USER root
RUN chown -R quarkus /usr/src/app
USER quarkus
RUN mvn -f /usr/src/app/pom.xml -Pnative clean package
FROM registry.access.redhat.com/ubi8/ubi-minimal
WORKDIR /work/
COPY --from=build /usr/src/app/target/*-runner /work/application
RUN chmod 775 /work/application
EXPOSE 8080
CMD ["./application", "-Dquarkus.http.host=0.0.0.0"]
```

`ImageAnalysisCoordinator.java`: 这是一个 AWS Lambda 函数，当新图像上传到 S3 存储桶时被触发：

```java
@ApplicationScoped
public class ImageAnalysisCoordinator implements RequestHandler<S3Event, String> {
    @Inject
    EcsClient ecsClient;
    @Inject
    S3Client s3Client;
    @Override
    public String handleRequest(S3Event s3Event,
        Context context) {
            String bucket = s3Event.getRecords().get(
                0).getS3().getBucket().getName();
            String key = s3Event.getRecords().get(
                0).getS3().getObject().getKey();
            RunTaskRequest runTaskRequest = RunTaskRequest.builder()
                .cluster("your-fargate-cluster")
                .taskDefinition("your-task-definition")
                .launchType("FARGATE")
                .overrides(TaskOverride.builder()
                    .containerOverrides(
                        ContainerOverride.builder()
                        .name("your-container-name")
             // Replace with your actual container name
                        .environment(KeyValuePair.builder()
                            .name("BUCKET")
                            .value(bucket)
                            .build(),
                        KeyValuePair.builder()
                            .name("KEY")
                            .value(key)
                            .build())
                        .build())
                    .build())
                .build();
                try {
                    ecsClient.runTask(runTaskRequest);
                    return "Fargate task launched for image analysis:                     " + bucket + "/" + key;
                } catch (Exception e) {
                     context.getLogger().log(
                         "Error launching Fargate task: " +                          e.getMessage());
                        return "Error launching Fargate task";
                }
        }
}
```

`ImageAnalysisCoordinator`类是一个 AWS Lambda 函数，作为服务器端 AI 图像分析架构的入口点。其主要职责如下：

+   从触发 Lambda 函数的传入 S3 事件中提取 S3 存储桶和对象键信息。

+   通过启动 ECS 任务并传递必要的环境变量（存储桶和键）来启动 Fargate 任务以执行计算密集型的图像分析。

+   处理在 Fargate 任务启动过程中发生的任何错误，并返回适当的状态消息。

这个 Lambda 函数充当轻量级协调器，负责协调整体图像分析工作流程。它触发由运行 `FargateImageAnalyzer` 应用的 Fargate 容器执行的资源密集型处理。通过这种方式分离责任，该架构实现了高效的资源利用和可扩展性。

**第 3 步：构建** **项目**：

对于 Lambda 函数，运行以下命令来打包函数：

```java
mvn package -Pnative -Dquarkus.native.container-build=true
```

对于 Fargate 容器，运行以下命令来构建 Docker 镜像：

```java
docker build -f src/main/docker/Dockerfile.jvm -t quarkus-ai-image-analysis .
```

**第 4 步：部署**：

为了简化无服务器 AI 基础设施的部署，已准备了一个 AWS CloudFormation 模板。此模板自动化整个部署过程，包括以下步骤：

1.  创建必要的 AWS 资源，例如以下内容：

    +   S3 存储桶用于存储图像和分析结果。

    +   Fargate 容器的 ECS 集群和任务定义。

    +   `ImageAnalysisCoordinator` 类的 Lambda 函数。

1.  将构建的工件（Lambda 函数 `.jar` 文件和 Docker 镜像）上传到适当的位置。

1.  配置 Lambda 函数所需的权限和触发器，以便在图像上传到 S3 存储桶时调用。

1.  部署 Fargate 任务定义并设置必要的网络配置。

要使用 CloudFormation 模板，您可以在本书的配套 GitHub 仓库中找到它，与源代码一起。只需下载模板，填写任何必要的参数，然后使用 AWS CloudFormation 服务部署它。这将为您设置整个无服务器 AI 基础设施，简化部署过程并确保不同环境之间的一致性。

# Java 中出现的并发和并行处理工具。

随着 Java 的不断发展，新的工具和框架正在被开发出来，以解决并发和并行编程不断增长的需求。这些进步旨在简化开发、提高性能并增强现代应用程序的可扩展性。

## Project Loom 简介 – 高效并发的虚拟线程。

**Project Loom** 是 OpenJDK 社区的一项雄心勃勃的倡议，旨在通过引入虚拟线程（也称为 **fibers**）来增强 Java 的并发模型。主要目标是简化编写、维护和观察高吞吐量并发应用程序，使其更加容易。

虚拟线程轻量级，由 Java 运行时管理，而不是操作系统。与传统线程受限于操作系统线程数量不同，虚拟线程可以扩展以处理数百万个并发操作，而不会耗尽系统资源。它们允许开发者以同步风格编写代码，同时实现异步模型的可扩展性。

其关键特性包括以下内容：

+   **轻量级特性**：虚拟线程比传统的操作系统线程轻得多，减少了内存和上下文切换的开销

+   **阻塞调用**：它们高效地处理阻塞调用，仅挂起虚拟线程，同时保持底层操作系统线程可用于其他任务

+   **简单性**：开发者可以使用熟悉的构造，如循环和条件语句编写简单、易读的代码，而无需依赖复杂的异步范式

为了说明 Project Loom 和虚拟线程的实际应用，让我们探索一个代码示例，该示例展示了如何在 AWS 云环境中使用 Project Loom 和 Akka 实现一个高并发微服务。

## 代码示例 - 使用 Project Loom 和 Akka 在 AWS 云环境中实现高并发微服务

在本节中，我们将展示如何使用 Project Loom 和 Akka 实现一个高并发微服务，该服务设计用于在 AWS 云环境中运行。此示例将展示如何利用 Project Loom 的虚拟线程和 Akka 提供的演员模型构建一个可扩展且高效的微服务：

`pom.xml` 依赖项：

```java
<!-- Akka Dependencies -->
    <dependency>
        <groupId>com.typesafe.akka</groupId>
        <artifactId>akka-actor-typed_${
            scala.binary.version}</artifactId>
        <version>${akka.version}</version>
    </dependency>
    <dependency>
        <groupId>com.typesafe.akka</groupId>
        <artifactId>akka-stream_${scala.binary.version}</artifactId>
        <version>${akka.version}</version>
    </dependency>
<!-- AWS SDK -->
    <dependency>
        <groupId>software.amazon.awssdk</groupId>
        <artifactId>s3</artifactId>
        <version>2.17.100</version>
    </dependency>
```

**步骤 2**：**代码实现**：

`HighConcurrencyService.java`：服务的主要入口点，它设置 `ActorSystem` 并使用 `ExecutorService` 管理虚拟线程：

```java
public class HighConcurrencyService {
    public static void main(String[] args) {
        ActorSystem<Void> actorSystem = ActorSystem.create(
            Behaviors.empty(), "high-concurrency-system");
        S3Client s3Client = S3Client.create();
        ExecutorService executorService = Executors.        newCachedThreadPool();
// Use a compatible thread pool
        for (int i = 0; i < 1000; i++) {
            final int index = i;
            executorService.submit(() -> {
                // Create and start the actor
                Behavior<RequestHandlerActor.HandleRequest> behavior =                 RequestHandlerActor.create(s3Client);
                var requestHandlerActor = actorSystem.systemActorOf(
                    behavior, "request-handler-" + index,
                    Props.empty());
                // Send a request to the actor
                requestHandlerActor.tell(
                    new RequestHandlerActor.HandleRequest(
                        "example-bucket",
                        "example-key-" + index,
                        "example-content"));
            });
        }
        // Clean up
        executorService.shutdown();
        actorSystem.terminate();
    }
}
```

`HighConcurrencyService` 类作为高并发微服务应用的入口点，旨在高效处理大量请求。利用 Akka 的演员模型和 Java 的并发特性，此类展示了如何有效地管理数千个并发任务。主函数初始化 `ActorSystem` 以创建和管理演员，设置 S3 客户端以与 AWS S3 服务交互，并使用执行器服务提交多个任务。每个任务都涉及创建一个新的演员实例来处理特定的请求，展示了如何在云环境中利用虚拟线程和演员进行可扩展和并发处理。

`RequestHandlerActor.java`：此演员处理单个请求以处理数据和与 AWS S3 交互：

```java
public class RequestHandlerActor {
    public static Behavior<HandleRequest> create(
        S3Client s3Client) {
            return Behaviors.setup(context ->
                Behaviors.receiveMessage(message -> {
            processRequest(s3Client, message.bucket,
                message.key, message.content);
            return Behaviors.same();
        }));
    }
    private static void processRequest(S3Client s3Client,
        String bucket, String key, String content) {
            PutObjectRequest putObjectRequest = PutObjectRequest.            builder()
                .bucket(bucket)
                .key(key)
                .build();
        PutObjectResponse response = s3Client.putObject(
            putObjectRequest,
            RequestBody.fromString(content));
        System.out.println(
            "PutObjectResponse: " + response);
    }
    public static class HandleRequest {
        public final String bucket;
        public final String key;
        public final String content;
        public HandleRequest(String bucket, String key,
            String content) {
                this.bucket = bucket;
                this.key = key;
                this.content = content;
        }
    }
}
```

`RequestHandlerActor` 类定义了处理高并发微服务中单个请求的演员的行为。它通过使用 S3 客户端处理存储数据到 AWS S3 的请求。`HandleRequest` 内部类封装了请求的详细信息，包括要存储的 S3 存储桶名称、键和内容。演员的行为定义为接收这些 `HandleRequest` 消息，通过与 S3 服务交互处理请求，并记录结果。此类展示了 Akka 的演员模型在高效管理和处理并发任务方面的应用，确保了基于云应用程序的可扩展性和健壮性。

**步骤 3：部署到 AWS**：

**Dockerfile**：Dockerfile 应该创建并保存在您的应用程序项目根目录中。这是 Dockerfile 的标准位置，因为它允许 Docker 构建过程访问所有必要的文件和资源，而无需进行额外的上下文切换：

```java
FROM amazoncorretto:17-alpine as builder
WORKDIR /workspace
COPY pom.xml .
COPY src ./src
RUN ./mvnw package -DskipTests
FROM amazoncorretto:17-alpine
WORKDIR /app
COPY --from=builder /workspace/target/high-concurrency-microservice-1.0.0-SNAPSHOT.jar /app/app.jar
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

关于此 Dockerfile 的要点如下：

+   它使用基于 Alpine Linux 分发的 `amazoncorretto: 17-alpine` 基础镜像，该镜像提供了 Java 17 运行时环境

+   它遵循两阶段构建过程：

    +   *构建器* 阶段编译应用程序并将其打包成 JAR 文件

    +   最后阶段复制打包的 JAR 文件并设置入口点以运行应用程序

**使用** **AWS ECS/Fargate** **部署**：

我们还准备了一个 CloudFormation 模板用于这些流程，可以在代码仓库中找到。按照以下步骤进行部署：

1.  **在 AWS 中创建 ECS 集群和任务定义**：设置您的 ECS 集群并定义将运行您的 Docker 容器的任务。

1.  **将 Docker 镜像上传到 Amazon Elastic Container Registry (ECR)**：将 Docker 镜像推送到 Amazon ECR 以便于部署。

1.  **配置 ECS 服务以使用 Fargate 并运行容器**：配置您的 ECS 服务以使用 AWS Fargate，一个无服务器计算引擎，来运行容器化应用程序。

此简化流程确保您的 高并发微服务在可扩展的云环境中高效部署。

此高并发微服务示例展示了利用 Project Loom 的虚拟线程和 Akka 的演员模型构建可扩展、高效且适用于云的应用程序的力量。通过利用这些高级并发工具，开发者可以简化代码，提高资源利用率，并提升其服务的整体性能和响应速度，尤其是在 AWS 云环境背景下。这为探索下一波云创新奠定了基础，其中新兴技术如 AWS Graviton 处理器和 Google Cloud Spanner 可以进一步增强基于云应用程序的可扩展性和功能。

# 准备迎接下一波云创新

随着云计算技术的快速发展，开发者和组织必须保持领先。预测云计算服务的进步，以下是如何为即将到来的云计算服务进步做准备：

+   **AWS Graviton**：AWS Graviton 是 AWS 设计的基于 ARM 的处理器系列，旨在提供比传统 x86 基础处理器更好的性价比，尤其是对于可以利用 ARM 架构并行处理能力的负载。最新的 **Graviton3** 版本可以提供比上一代基于 Intel 的 EC2 实例高达 25% 的性能提升和 60% 的性价比提升。

+   **Amazon Corretto**：另一方面，Amazon Corretto 是一个无需付费、多平台、生产就绪的 OpenJDK 发行版，是 Java 平台的免费开源实现。Corretto 支持基于 x86 和 ARM（包括 Graviton）的架构，为 AWS 客户提供经过认证、测试和支持的 JDK 版本。基于 ARM 的 Corretto JDK 已针对 AWS Graviton 驱动的实例进行了优化。

考虑使用 Amazon Corretto JDK。以下是一个构建 Docker 镜像的代码片段：

```java
FROM --platform=$BUILDPLATFORM amazoncorretto:17
COPY . /app
WORKDIR /app
RUN ./gradlew build
CMD ["java", "-jar", "app.jar"]
```

构建并推送以下命令：

```java
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest --push .
```

**Google Cloud Spanner**：Cloud Spanner 是一个完全托管、可扩展的关系型数据库服务，提供强一致性和高可用性：

+   **全局分布**：Spanner 支持多区域和全球部署，提供高可用性和低延迟的数据访问

+   **强一致性**：与许多 NoSQL 数据库不同，Spanner 维护强一致性，使其适用于需要事务完整性的应用程序

+   **无缝扩展**：Spanner 自动处理水平扩展，允许应用程序在不影响性能或可用性的情况下增长

**示例**：使用 Java 与 Cloud Spanner：

```java
import com.google.cloud.spanner.*;
try (Spanner spanner = SpannerOptions.newBuilder(
    ).build().getService()) {
    DatabaseClient dbClient = spanner.getDatabaseClient(
        DatabaseId.of(projectId, instanceId, databaseId));
    try (ResultSet resultSet = dbClient
        .singleUse() // Create a single-use read-only //transaction
        .executeQuery(Statement.of("SELECT * FROM Users"))){
    while (resultSet.next()) {
        System.out.printf("User ID: %d, Name: %s\n",
            resultSet.getLong("UserId"),
            resultSet.getString("Name"));
        }
    }}
```

此代码片段展示了使用 Java 客户端库与 Google Cloud Spanner 交互的使用方法。代码首先使用 `SpannerOptions` 构建器创建 Spanner 客户端并检索服务实例。然后获取一个 `DatabaseClient` 实例，用于与由 `projectId`、`instanceId` 和 `databaseId` 参数指定的特定 Spanner 数据库交互。

在 try-with-resources 块中，代码使用 `singleUse()` 方法创建一个单次使用的只读事务，并执行一个 `SQL SELECT` 查询以检索 `Users` 表中的所有记录。然后遍历结果，并为每个用户记录打印 `UserId` 和 `Name` 列。

此示例展示了 Google Cloud Spanner Java 客户端库的基本用法，包括建立数据库连接、执行查询和处理结果，同时确保适当的资源管理和清理。

## 量子计算

**量子计算**，尽管仍处于早期阶段，但通过解决经典计算机无法解决的复杂问题，有望彻底改变各个行业。量子计算机利用量子力学的原理，如叠加和纠缠，以并行的方式执行计算。

虽然对于大多数应用来说并不立即实用，但开始学习量子计算原理以及它们如何应用于您的领域是有益的。需要探索的关键概念包括量子比特、量子门以及如 Shor 算法（用于大数分解）和 Grover 算法（用于搜索问题）等量子算法。

理解这些原理将使您为未来的进步和量子计算与您的工作流程的潜在集成做好准备。通过现在熟悉基础概念，您将更好地定位自己，以便在量子计算变得更加易于获取并适用于现实世界问题时充分利用它。

保持信息灵通并探索这些技术，即使在入门级别，也将帮助确保您的组织准备好适应并在这快速发展的云环境中蓬勃发展。

# 摘要

作为本书的最后一章，我们现在站在未来的边缘，云技术正以惊人的速度持续发展。在本节的结尾部分，我们探讨了即将重塑我们在云中开发和部署应用程序方式的新兴趋势和进步，特别强调了 Java 在塑造这些创新中的关键作用。

我们首先深入研究了无服务器 Java 的演变，我们看到 Quarkus 和 Micronaut 等框架如何重新定义了函数即服务的边界。这些尖端工具利用原生图像编译等技术，在无服务器环境中提供前所未有的性能和效率，同时使完整的 Java 应用程序作为无服务器容器进行部署。这代表了一个重大转变，使开发者能够创建高度可扩展、响应迅速且云本地的应用程序，这些应用程序超越了简单的函数执行。

接下来，我们将注意力转向边缘计算领域，数据处理和决策正越来越接近源头。Java 的平台独立性、性能和广泛的生态系统使其成为构建边缘应用的理想选择。我们介绍了关键框架和工具，这些框架和工具使 Java 开发者能够利用边缘计算的力量，确保他们的应用程序能够无缝集成这一快速发展的范式。

此外，我们还探讨了 Java 在云生态系统内 AI 和 ML 集成中不断发展的作用。从无服务器 AI/ML 工作流到 Java 与基于云的 AI 服务的无缝集成，我们揭示了这种融合带来的机遇和挑战，为你提供了利用这些技术在基于 Java 的应用程序中发挥其力量的知识。

最后，我们勇敢地进入了迷人的量子计算领域，这个领域承诺将彻底改变各个行业。尽管仍处于早期阶段，但理解量子计算的基本原理，如量子比特、量子门和算法，可以为开发者准备未来的进步以及它们与基于 Java 的应用的潜在集成。

随着本书的结束，你现在对云计算的兴起趋势和 Java 在塑造这些创新中的关键作用有了全面的理解。凭借这些知识，你将能够为你的应用程序和基础设施在快速发展的云计算环境中取得成功定位，确保你的组织能够在未来几年中适应并蓬勃发展。

# 问题

1.  使用 Quarkus 和 GraalVM 构建无服务器 Java 应用程序的关键好处是什么？

    1.  改善启动时间和减少内存使用

    1.  更容易与基于云的 AI/ML 服务集成

    1.  在多个云提供商之间无缝部署

    1.  所有上述选项

1.  以下哪项是使用 Java 在边缘计算环境中使用的关键优势？

    1.  平台独立性

    1.  广泛的库支持

    1.  强大的安全模型

    1.  所有上述选项

1.  哪个云人工智能服务允许 Java 开发者轻松训练和部署定制的机器学习模型，而无需广泛的机器学习专业知识？

    1.  AWS SageMaker

    1.  Google Cloud AutoML

    1.  微软 Azure 认知服务

    1.  IBM Watson Studio

1.  在提供的代码示例中，哪个量子计算概念展示了将量子比特置于叠加并测量结果？

    1.  量子纠缠

    1.  量子传输

    1.  量子叠加

    1.  量子隧穿

1.  使用无服务器容器在云中为 Java 应用程序构建的关键好处是什么？

    1.  降低管理基础设施的运营开销

    1.  无服务器函数的冷启动时间增加

    1.  无法包含自定义库和依赖项

    1.  对运行时环境的有限控制

# 附录 A：设置云原生 Java 环境

在*附录 A*中，你将学习如何为 Java 应用程序设置云原生环境。本指南全面涵盖了从构建和打包 Java 应用程序到在流行的云平台（如**亚马逊网络服务**（**AWS**）、**微软 Azure**和**谷歌云平台**（**GCP**））上部署它们的所有内容。关键主题包括：

+   **构建和打包**：使用 Maven 和 Gradle 等构建工具创建和管理 Java 项目的逐步说明。

+   **确保云就绪**：使 Java 应用程序无状态和可配置的最佳实践，以便在云环境中茁壮成长。

+   **容器化**：如何为 Java 应用程序创建 Docker 镜像并使用 Docker 进行部署。

+   **云部署**：在 AWS、Azure 和 GCP 上部署 Java 应用程序的详细步骤，包括设置必要的云环境、创建和管理云资源，以及使用特定的云服务如 Elastic Beanstalk、Kubernetes 和无服务器函数。

到附录结束时，你将牢固地理解如何有效地构建、打包、容器化和部署 Java 应用程序到云原生环境中。

# 通用方法 - 构建和打包 Java 应用程序

本节提供了构建和打包 Java 应用程序的必要步骤的详细指南，确保它们在云环境中部署就绪。

1.  确保你的应用程序已准备好云部署

    1.  通过`application.properties`或`application.yaml`文件，或使用配置管理工具来配置。以下是一个示例：

```java
server.port=8080
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
```

1.  使用构建工具如 Maven 或 Gradle

    +   如果你的项目根目录中还没有`pom.xml`文件，请添加必要的依赖项。以下是一个`pom.xml`的示例：

        ```java
        <project  
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
            <modelVersion>4.0.0</modelVersion>
            <groupId>com.example</groupId>
            <artifactId>myapp</artifactId>
            <version>1.0-SNAPSHOT</version>
            <dependencies>
                <dependency>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-web</artifactId>
                </dependency>
                <!-- Add other dependencies here -->
            </dependencies>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.springframework.boot</groupId>
                        <artifactId>spring-boot-maven-plugin</artifactId>
                    </plugin>
                </plugins>
            </build>
        </project>
        ```

    +   如果你的项目根目录中还没有`build.gradle`文件，请添加必要的依赖项，以下是一个`build.gradle`的示例：

        ```java
        plugins {
            id 'org.springframework.boot' version '2.5.4'
            id 'io.spring.dependency-management' version '1.0.11.RELEASE'
            id 'java'
        }
        group = 'com.example'
        version = '1.0-SNAPSHOT'
        sourceCompatibility = '11'
        repositories {
            mavenCentral()
        }
        dependencies {
            implementation 'org.springframework.boot:spring-boot-starter-web'
            // Add other dependencies here
        }
        test {
            useJUnitPlatform()
        }
        ```

1.  **构建 JAR 文件**：使用 Maven 或 Gradle 构建 JAR 文件。

    +   **Maven**：运行以下命令：

```java
mvn clean package
```

这个命令将在目标目录中生成一个 JAR 文件，通常命名为`myapp-1.0-SNAPSHOT.jar`。

+   **Gradle**：运行以下命令：

```java
gradle clean build
```

这个命令将在 build/libs 目录中生成一个 JAR 文件，通常命名为`myapp-1.0-SNAPSHOT.jar`。

注意：

如果你没有使用容器，你可以在这里停止。位于目标或 build/libs 目录中的 JAR 文件现在可以直接用来运行你的应用程序。

1.  使用 Docker 容器化你的应用程序

    1.  **创建 Dockerfile**：在你的项目根目录中创建一个 Dockerfile，内容如下：

    ```java
    # Use an official OpenJDK runtime as a parent image (Java 21)
    FROM openjdk:21-jre-slim
    # Set the working directory
    WORKDIR /app
    # Copy the executable JAR file to the container
    COPY target/myapp-1.0-SNAPSHOT.jar /app/myapp.jar
    # Expose the port the app runs on
    EXPOSE 8080
    # Run the JAR file
    ENTRYPOINT ["java", "-jar", "myapp.jar"]
    ```

    确保根据你的 JAR 文件所在的不同目录或不同的名称调整 COPY 指令。

    1.  **构建 Docker 镜像**：

    使用 Docker build 命令构建 Docker 镜像。在你的 Dockerfile 所在目录中运行此命令：

```java
docker build -t myapp:1.0 .
```

这个命令将创建一个名为`myapp`的 Docker 镜像，带有标签`1.0`。

1.  **运行 Docker 容器**：使用 docker run 命令运行 Docker 容器：

```java
docker run -p 8080:8080 myapp:1.0
```

这个命令将从`myapp:1.0`镜像启动一个容器，并将容器的 8080 端口映射到宿主机的 8080 端口。

本节提供了构建和打包 Java 应用程序的必要步骤的详细指南，确保它们在云环境中部署就绪。

在学习如何构建和打包您的 Java 应用程序之后，下一步是探索在流行的云平台上部署这些应用程序的具体步骤。

**逐步指南：在流行的云平台上部署 Java 应用程序：**

1.  设置 AWS 环境：

    1.  `aws configure`并输入您的 AWS 访问密钥、秘密密钥、区域和输出格式。

    1.  `trust-policy.json`

    ```java
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": {
                    "Service": "ec2.amazonaws.com"
                },
                "Action": "sts:AssumeRole"
            }
        ]
    ```

1.  使用 WAS CLI 将 Java 应用程序部署到 AWS：Elastic Beanstalk（PaaS）

    1.  创建 IAM 角色：

```java
aws iam create-role --role-name aws-elasticbeanstalk-ec2-role --assume-role-policy-document file://trust-policy.json
```

1.  将所需的策略附加到角色：

```java
aws iam attach-role-policy --role-name aws-elasticbeanstalk-ec2-role-java --policy-arn arn:aws:iam::aws:policy/AWSElasticBeanstalkWebTier
aws iam attach-role-policy --role-name aws-elasticbeanstalk-ec2-role-javaa --policy-arn arn:aws:iam::aws:policy/AWSElasticBeanstalkMulticontainerDocker
aws iam attach-role-policy --role-name aws-elasticbeanstalk-ec2-role-java --policy-arn arn:aws:iam::aws:policy/AWSElasticBeanstalkWorkerTier
```

1.  创建实例配置文件：

```java
aws iam create-instance-profile --instance-profile-name aws-elasticbeanstalk-ec2-role-java
```

1.  将角色添加到实例配置文件：

```java
aws iam add-role-to-instance-profile --instance-profile-name aws-elasticbeanstalk-ec2-role-java --role-name aws-elasticbeanstalk-ec2-role-java
```

1.  部署到 Elastic Beanstalk：

    1.  创建 Elastic Beanstalk 应用程序：

```java
aws elasticbeanstalk create-application --application-name MyJavaApp
```

1.  使用最新的 Corretto 21 版本在 Amazon Linux 2023 上创建一个新的 Elastic Beanstalk 环境。

```java
aws elasticbeanstalk create-environment --application-name MyJavaApp --environment-name my-env --solution-stack-name "64bit Amazon Linux 2023 v4.2.6 running Corretto 21"
```

1.  将`my-application.jar`上传到 my-bucket S3 存储桶中的部署文件夹。根据您的特定用例调整参数。

```java
aws s3 cp target/my-application.jar s3://my-bucket/my-application.jar
```

1.  在 Elastic Beanstalk 中创建新的应用程序版本：

```java
aws elasticbeanstalk create-application-version --application-name MyJavaApp --version-label my-app-v1 --description "First version" --source-bundle S3Bucket= my-bucket,S3Key= my-application.jar
```

1.  更新 Elastic Beanstalk 环境以使用新的应用程序版本

```java
aws elasticbeanstalk update-environment --environment-name my-env --version-label my-app-v1
```

1.  检查环境健康：

```java
aws elasticbeanstalk describe-environment-health --environment-name my-env --attribute-names All
```

1.  部署您的 Java 应用程序：ECS（容器）

    1.  将 Docker 镜像推送到 Amazon **弹性容器注册库**（**ECR**）：首先，创建一个 ECR 存储库：

```java
aws ecr create-repository --repository-name my-application
```

1.  将 Docker 认证到您的 ECR：

```java
aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com
```

1.  标记您的 Docker 镜像：

```java
docker tag my-application:1.0                                                                                         <account-id>.dkr.ecr.<region>.amazonaws.com/my-application:1.0
```

1.  将您的 Docker 镜像推送到 ECR：

```java
docker push <account-id>.dkr.ecr.<region>.amazonaws.com/ my-application:1.0
```

1.  使用任务定义配置设置`task-definition.json`：

```java
{
    "family": "my-application-task",
    "networkMode": "awsvpc",
    "requiresCompatibilities": [
        "FARGATE"
    ],
    "cpu": "256",
    "memory": "512",
    "containerDefinitions": [
        {
            "name": "my-application",
            "image": "<account-id>.dkr.ecr.<
            region>.amazonaws.com/my-application:1.0",
            "portMappings": [
            {
                "containerPort": 8080,
                "protocol": "tcp"
            }
            ],
            "essential": true
        }
    ]
}
```

1.  注册任务定义：

```java
aws ecs register-task-definition --cli-input-json file://task-definition.json
```

1.  创建集群：

```java
aws ecs create-cluster --cluster-name cloudapp-cluster
```

1.  **创建服务**：使用以下命令创建服务：

```java
aws ecs create-service \
    --cluster cloudapp-cluster \
    --service-name cloudapp-service \
    --task-definition cloudapp-task \
    --desired-count 1 \
    --launch-type FARGATE \
    --network-configuration "awsvpcConfiguration={
        subnets=[subnet-XXXXXXXXXXXXXXXXX],securityGroups=[
            sg-XXXXXXXXXXXXXXXXX],assignPublicIp=ENABLED}"
```

重要注意事项：

将 subnet-XXXXXXXXXXXXXXXXX 替换为您要运行任务的子网的实际 ID。

将 sg-XXXXXXXXXXXXXXXXX 替换为您要关联任务的实际安全组 ID。

此命令使用正斜杠（\）进行行续接，这在 Unix-like 环境中（Linux、macOS、Windows 上的 Git Bash）是合适的。

对于 Windows 命令提示符，将反斜杠（\）替换为连字符符号（^）以进行行续接。

对于 PowerShell，在每个行的末尾使用反引号（`）而不是反斜杠进行行续接。

--desired-count 1 参数指定您希望始终运行一个任务。

--launch-type FARGATE 参数指定此服务将使用 AWS Fargate，这意味着您不需要管理底层的 EC2 实例。

1.  部署您的无服务器 Java Lambda 函数

    1.  创建 AWS Lambda 函数角色：

```java
aws iam create-role --role-name lambda-role --assume-role-policy-document file://trust-policy.json
```

1.  将`AWSLambdaBasicExecutionRole`策略附加到角色：

```java
aws iam attach-role-policy --role-name lambda-role --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

1.  创建 Lambda 函数：

```java
aws lambda create-function \
    --function-name java-lambda-example \
    --runtime java17 \
    --role arn:aws:iam::<account-id>:role/lambda-role \
    --handler com.example.LambdaHandler::handleRequest \
    --zip-file fileb://target/lambda-example-1.0-SNAPSHOT.jar
```

1.  **调用 Lambda 函数**：当使用 aws lambda invoke 命令测试您的 AWS Lambda 函数时，更新--payload 参数以匹配您特定 Lambda 函数的预期输入格式非常重要。

```java
aws lambda invoke --function-name java-lambda-example --payload '{"name": "World"}' response.json
```

1.  检查响应：

```java
cat response.json
```

现在，您已经了解了如何设置云原生 Java 环境，并在各种云平台上部署应用程序，您可能想深入了解特定的云服务。以下链接提供了额外的资源和文档，以帮助您进一步了解和掌握在云中部署和管理 Java 应用程序的知识和技能。

## AWS 的有用链接，以获取更多关于 AWS 的信息

+   **Amazon EC2**：Amazon EC2 入门指南（[`docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html`](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)）

+   **AWS Elastic Beanstalk**：AWS Elastic Beanstalk 入门指南（[`docs.aws.amazon.com/elasticbeanstalk/latest/dg/GettingStarted.html`](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/GettingStarted.html)）

+   **Amazon ECS**：Amazon ECS 入门指南（[`docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_GetStarted.html`](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_GetStarted.html)）

+   **AWS Lambda**：AWS Lambda 入门指南（[`docs.aws.amazon.com/lambda/latest/dg/getting-started.html`](https://docs.aws.amazon.com/lambda/latest/dg/getting-started.html)）

+   **管理环境变量**：在 AWS Lambda 中管理环境变量的最佳实践（[`docs.aws.amazon.com/lambda/latest/dg/configuration-envvars.html`](https://docs.aws.amazon.com/lambda/latest/dg/configuration-envvars.html)）

# 微软 Azure

在本节中，您将学习部署 Java 应用程序在 Microsoft Azure 上的步骤。这包括设置 Azure 环境、在虚拟机和容器上部署应用程序，以及利用 **Azure Kubernetes 服务**（**AKS**）为容器化应用程序。此外，您还将了解如何在 Azure Functions 上部署 Java 函数，使您能够利用无服务器计算来部署 Java 应用程序。

1.  设置 Azure 环境：

    1.  **下载并安装 Azure CLI**：请从 Azure CLI 安装指南中根据您的操作系统遵循官方安装说明（[`learn.microsoft.com/en-us/cli/azure/install-azure-cli`](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)）。

    1.  **配置 Azure CLI**：打开您的终端或命令提示符，并运行以下命令以登录您的 Azure 账户：

```java
az login
```

1.  按照说明登录您的 Azure 账户。

接下来，您将学习如何在 Azure 虚拟机上部署常规 Java 应用程序。

**在 Azure 虚拟机上部署常规 Java 应用程序** **虚拟机**

1.  创建资源组：

```java
az group create --name myResourceGroup --location eastus
```

1.  创建虚拟机：

```java
az vm create --resource-group myResourceGroup --name myVM --image UbuntuLTS --admin-username azureuser --generate-ssh-keys
```

1.  打开 8080 端口：

```java
az vm open-port --port 8080 --resource-group myResourceGroup --name myVM
```

1.  连接到 VM：

```java
ssh azureuser@<vm-ip-address>
```

1.  在 VM 上安装 Java：

```java
sudo apt update
sudo apt install openjdk-21-jre -y
```

1.  转移并运行 JAR 文件：

```java
scp target/myapp-1.0-SNAPSHOT.jar azureuser@<vm-ip-address>:/home/azureuser
ssh azureuser@<vm-ip-address>
java -jar myapp-1.0-SNAPSHOT.jar
```

一旦你在 Azure 虚拟机上成功部署了你的 Java 应用程序，你可以使用 Azure 门户和 CLI 工具根据需要管理和扩展你的应用程序。这种方法为在云环境中运行传统的 Java 应用程序提供了坚实的基础。

接下来，你将学习如何使用 AKS 部署 Java 应用程序到容器中，AKS 为容器化应用程序提供了一个更灵活和可扩展的解决方案。

**在 AKS 上部署 Java 应用程序到容器**

1.  创建一个 **Azure 容器注册库**（**ACR**）：

```java
az acr create --resource-group myResourceGroup --name myACR --sku Basic
```

1.  登录到 ACR：

```java
az acr login --name myACR
```

1.  标记并推送 Docker 镜像到 ACR：

```java
docker tag myapp:1.0 myacr.azurecr.io/myapp:1.0
docker push myacr.azurecr.io/myapp:1.0
```

1.  创建 AKS 集群：

```java
az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 1 --enable-addons monitoring --generate-ssh-keys
```

1.  获取 AKS 凭据：

```java
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

1.  将应用程序部署到 AKS：

    1.  创建一个部署 YAML 文件（`deployment.yaml`）：

    ```java
    apiVersion: apps/v1
    kind: Deployment
    metadata:
        name: myapp-deployment
    spec:
        replicas: 1
        selector:
            matchLabels:
                app: myapp
        template:
            metadata:
                labels:
                    app: myapp
            spec:
                containers:
                    - name: myapp
                image: myacr.azurecr.io/myapp:1.0
                ports:
                    - containerPort: 8080
    ```

    1.  应用部署：

```java
kubectl apply -f deployment.yaml
```

1.  暴露部署：

```java
kubectl expose deployment myapp-deployment--type=LoadBalancer --port=8080
```

这完成了将容器化 Java 应用程序部署到 AKS 的过程。然而，对于需要更细粒度控制应用程序执行或想要构建无服务器微服务的场景，Azure Functions 提供了一个出色的替代方案。接下来，你将学习如何在 Azure Functions 上部署 Java 函数，使你能够利用无服务器计算为事件驱动应用程序和微服务提供支持。

**在 Azure Functions 上部署 Java 函数**

1.  安装 Azure Functions 核心工具：

    +   **Windows**：使用 MSI 安装程序（[`learn.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Cisolated-process%2Cnode-v4%2Cpython-v2%2Chttp-trigger%2Ccontainer-apps&pivots=programming-language-csharp#v2`](https://learn.microsoft.com/en-us/azure/azure-functions/functions-run-local?tabs=windows%2Cisolated-process%2Cnode-v4%2Cpython-v2%2Chttp-trigger%2Ccontainer-apps&pivots=programming-language-csharp#v2)）

    +   **macOS**：

```java
brew tap azure/functions
brew install azure-functions-core-tools@4
```

1.  创建一个新的函数应用：

```java
func init MyFunctionApp --java
cd MyFunctionApp
func new
```

1.  构建函数：

```java
mvn clean package
```

1.  部署到 Azure：

```java
func azure functionapp publish <FunctionAppName>
```

注意事项

将如 <vm-ip-address>、<your-region>、<FunctionAppName> 等占位符替换为你的实际值。

关于配置环境变量和管理 Azure 环境特定配置的详细信息，你可以参考官方 Azure 文档：

+   **Azure App Service 配置**：配置 Azure App Service 中的应用程序（[`learn.microsoft.com/en-us/azure/app-service/configure-common?tabs=portal`](https://learn.microsoft.com/en-us/azure/app-service/configure-common?tabs=portal)）

+   **AKS 配置**：AKS 中集群和节点池配置的最佳实践（[`learn.microsoft.com/en-us/azure/aks/operator-best-practices-scheduler`](https://learn.microsoft.com/en-us/azure/aks/operator-best-practices-scheduler)）

+   **Azure Functions 配置**：在 Azure Functions 中配置函数应用设置（[`learn.microsoft.com/en-us/azure/azure-functions/functions-how-to-use-azure-function-app-settings?tabs=azure-portal%2Cto-premium`](https://learn.microsoft.com/en-us/azure/azure-functions/functions-how-to-use-azure-function-app-settings?tabs=azure-portal%2Cto-premium)）

现在您已经学会了如何在各种 Azure 服务上部署 Java 应用程序，包括虚拟机、AKS 和 Azure Functions，让我们探索另一个主要的云服务提供商。接下来的章节将指导您在 GCP 上进行类似的部署过程，让您能够在不同的环境中扩展您的云部署技能。

# Google Cloud Platform

在本节中，您将学习如何在 **Google Cloud Platform** 或 **GCP** 上部署 Java 应用程序，GCP 是领先的云服务提供商之一。GCP 提供了一系列服务，满足各种部署需求，从虚拟机到容器化环境和无服务器函数。我们将介绍 GCP 的设置过程，并指导您使用不同的 GCP 服务部署 Java 应用程序，包括 **Google Compute Engine** (**GCE**), **Google Kubernetes Engine** (**GKE**), 和 Google Cloud Functions。这些知识将使您能够利用 GCP 强大的基础设施和服务来支持您的 Java 应用程序。

## 设置 Google Cloud 环境

1.  如果您还没有，请创建一个 Google Cloud 账户。

1.  安装 Google Cloud SDK。请从官方 Google Cloud SDK 文档中获取您操作系统的说明（[`cloud.google.com/sdk/docs/install`](https://cloud.google.com/sdk/docs/install)）

1.  初始化 Google Cloud SDK：

```java
gcloud init
```

1.  按提示登录并选择您的项目。

1.  设置您的项目 ID：

```java
gcloud config set project YOUR_PROJECT_ID
```

在成功设置您的 Google Cloud 环境之后，您现在可以准备使用 GCP 强大的基础设施部署和管理 Java 应用程序。在接下来的章节中，您将探索在 GCE、GKE 和 Google Cloud Functions 上部署 Java 应用程序的具体方法。

## 将您的 Java 应用程序部署到 Google Cloud

**GCE 用于常规** **Java 应用程序**：

1.  创建虚拟机实例：

```java
gcloud compute instances create my-java-vm --zone=us-central1-a --machine-type=e2-medium --image-family=ubuntu-2004-lts --image-project=ubuntu-os-cloud
```

1.  通过 SSH 登录到虚拟机：

```java
gcloud compute ssh my-java-vm --zone=us-central1-a
```

1.  在虚拟机上安装 Java：

```java
sudo apt update
sudo apt install openjdk-17-jdk -y
```

1.  将您的 JAR 文件传输到虚拟机：

```java
gcloud compute scp your-app.jar my-java-vm:~ --zone=us-central1-a
```

1.  运行您的 Java 应用程序：

```java
java -jar your-app.jar
```

通过遵循这些步骤，您可以在 Google Cloud 上高效地部署和管理您的 Java 应用程序，利用 GCP 提供的各种服务和工具。在您的 Java 应用程序成功部署到 Google Cloud 之后，您现在可以探索使用 GKE 的容器化部署，GKE 提供了强大的编排能力来管理大规模的容器。

## GKE 用于容器化应用程序

在本节中，您将学习如何使用 GKE 在容器中部署 Java 应用程序。GKE 提供了一个管理环境，用于使用 Kubernetes 部署、管理和扩展容器化应用程序。您将指导设置 GKE 集群、部署您的 Docker 镜像以及高效管理您的容器化应用程序。

1.  创建 GKE 集群：

```java
gcloud container clusters create my-cluster --num-nodes=3 --zone=us-central1-a
```

1.  获取集群的凭证：

```java
gcloud container clusters get-credentials my-cluster --zone=us-central1-a
```

1.  将您的 Docker 镜像推送到 **Google Container** **Registry** (**GCR**)：

```java
docker tag your-app:latest gcr.io/YOUR_PROJECT_ID/your-app:latest
docker push gcr.io/YOUR_PROJECT_ID/your-app:latest
```

1.  创建 Kubernetes 部署：创建一个名为 deployment.yaml 的文件：

    ```java
    apiVersion: apps/v1
    kind: Deployment
    metadata:
        name: your-app
    spec:
        replicas: 3
        selector:
            matchLabels:
                app: your-app
        template:
            metadata:
                labels:
                    app: your-app
        spec:
            containers:
                - name: your-app
            image: gcr.io/YOUR_PROJECT_ID/your-app:latest
            ports:
            - containerPort: 8080
    ```

1.  应用部署：

```java
kubectl apply -f deployment.yaml
```

1.  暴露部署：

```java
kubectl expose deployment your-app --type=LoadBalancer --port 80 --target-port 8080
```

通过利用 GKE，您可以充分利用 Kubernetes 的强大功能，确保您的容器化 Java 应用程序具有高可用性、可扩展性和易于维护。

## Google Cloud Functions 用于无服务器 Java 函数

在本节中，您将学习如何使用 Google Cloud Functions 部署 Java 函数，使您能够在完全托管的无服务器环境中运行事件驱动的代码。您将指导设置您的开发环境，创建和部署您的 Java 函数，并使用 Google Cloud 的强大无服务器工具有效地管理它们。

1.  为您的函数创建一个新的目录：

```java
mkdir my-java-function
cd my-java-function
```

1.  初始化一个新的 Maven 项目：

```java
mvn archetype:generate -DgroupId=com.example -DartifactId=my-java-function -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

1.  将必要的依赖项添加到您的 `pom.xml`：

    ```java
    <dependency>
      <groupId>com.google.cloud.functions</groupId>
      <artifactId>functions-framework-api</artifactId>
      <version>1.0.4</version>
      <scope>provided</scope>
    </dependency>
    ```

1.  创建您的函数类：

    ```java
    package com.example;
    import com.google.cloud.functions.HttpFunction;
    import com.google.cloud.functions.HttpRequest;
    import com.google.cloud.functions.HttpResponse;
    import java.io.BufferedWriter;
    public class Function implements HttpFunction {
        @Override
        public void service(HttpRequest request,
            HttpResponse response) throws Exception {
            BufferedWriter writer = response.getWriter();
            writer.write("Hello, World!");
            }
        }
    ```

1.  部署函数：

```java
gcloud functions deploy my-java-function --entry-point com.example.Function --runtime java17 --trigger-http --allow-unauthenticated
```

这些说明提供了将 Java 应用程序部署到 Google Cloud 的基本设置，使用不同的服务。请记住，根据您特定的应用程序需求和 Google Cloud 项目设置调整命令和配置。

# 有用的链接以获取更多信息

+   **Google Compute Engine**：使用计算引擎快速入门指南开始创建和管理 GCP 中的虚拟机：[`cloud.google.com/compute/docs/quickstart`](https://cloud.google.com/compute/docs/quickstart)

+   **Google Kubernetes Engine**：通过 GKE 深入容器编排，使用 GKE 快速入门指南部署您的第一个 Kubernetes 集群：[`cloud.google.com/kubernetes-engine/docs/quickstart`](https://cloud.google.com/kubernetes-engine/docs/quickstart)

+   **Google Cloud Functions**：使用云函数部署指南开发和部署响应事件的函数：[`cloud.google.com/functions/docs/deploying`](https://cloud.google.com/functions/docs/deploying)

+   **管理环境变量** [`cloud.google.com/run/docs/configuring/services/environment-variables`](https://cloud.google.com/run/docs/configuring/services/environment-variables)

# 附录 B：资源和进一步阅读

# 推荐的书籍、文章和在线课程

## 第 1-3 章

### 书籍

+   《*云原生 Java：使用 Spring Boot、Spring Cloud 和 Cloud Foundry 设计弹性系统*》由 Josh Long 和 Kenny Bastani 著。这本综合指南提供了构建适用于云环境的可扩展、弹性 Java 应用程序的实际见解，涵盖了 Spring Boot、Spring Cloud 和 Cloud Foundry 技术。链接：[`www.amazon.com/Cloud-Native-Java-Designing-Resilient/dp/1449374646`](https://www.amazon.com/Cloud-Native-Java-Designing-Resilient/dp/1449374646)

+   《*Java 并发实践*》由 Brian Goetz 等人著。这是一部关于 Java 并发的开创性作品，本书深入介绍了并发编程技术、最佳实践以及开发多线程应用程序时需要避免的陷阱。

+   *《Haskell 中的并行和并发编程》* 由 Simon Marlow 撰写。虽然专注于 Haskell，但本书提供了对并行编程概念的宝贵见解，这些概念可以应用于 Java，为并发和并行应用程序设计提供了更广阔的视角。链接：[`www.oreilly.com/library/view/parallel-and-concurrent/9781449335939/`](https://www.oreilly.com/library/view/parallel-and-concurrent/9781449335939/)

+   *《设计分布式系统：构建可扩展、可靠服务的模式和范式》* 由 Brendan Burns（Microsoft Azure）撰写。本书探讨了构建可扩展和可靠分布式系统的基本模式，提供了来自微软 Azure 在云计算方面的经验见解。链接：[`www.amazon.com/Designing-Distributed-Systems-Patterns-Paradigms/dp/1491983647`](https://www.amazon.com/Designing-Distributed-Systems-Patterns-Paradigms/dp/1491983647)

### 文章

+   *《微服务模式》* 由 Chris Richardson（microservices.io）撰写。这是一本关于微服务架构模式的全面指南，帮助开发者理解和实施有效的基于微服务的系统。链接：[`microservices.io/patterns/index.html`](https://microservices.io/patterns/index.html)

+   *《Java Fork/Join 框架》* 由 Doug Lea 撰写。这是其创造者对 Fork/Join 框架的深入探讨，提供了关于其在 Java 中实现并行处理的设计和实现的宝贵见解。链接：[`gee.cs.oswego.edu/dl/papers/fj.pdf`](http://gee.cs.oswego.edu/dl/papers/fj.pdf)

+   *《多核时代下的 Amdahl 定律》* 由 Mark D. Hill 和 Michael R. Marty 撰写。这篇文章提供了对 Amdahl 定律的现代视角及其对并行计算的影响，帮助开发者理解当代系统中并行处理的局限性和潜力。链接：[`research.cs.wisc.edu/multifacet/papers/ieeecomputer08_amdahl_multicore.pdf`](https://research.cs.wisc.edu/multifacet/papers/ieeecomputer08_amdahl_multicore.pdf)

### 在线课程

+   *《Java 多线程、并发与性能优化》* 由 Udemy 提供。这门全面的课程涵盖了 Java 多线程、并发和性能优化技术，提供了实际示例和动手练习，以掌握高级 Java 编程概念。链接：[`www.udemy.com/course/java-multithreading-concurrency-performance-optimization/`](https://www.udemy.com/course/java-multithreading-concurrency-performance-optimization/)

+   *《Java 并发编程》* 由 Coursera（由 Rice 大学提供）。该课程专注于 Java 并发的基础原则，提供了实际练习来巩固对并发编程概念和技术理解。链接：[`www.coursera.org/learn/concurrent-programming-in-java`](https://www.coursera.org/learn/concurrent-programming-in-java)

+   Udemy 上的 *《使用 Project Reactor 在现代 Java 中进行响应式编程》*。这是一门关于使用 Project Reactor 在 Java 中进行响应式编程的全面课程，教导开发者如何构建适用于现代软件架构中更好可扩展性和弹性的响应式应用。链接：[`www.udemy.com/course/reactive-programming-in-modern-java-using-project-reactor/`](https://www.udemy.com/course/reactive-programming-in-modern-java-using-project-reactor/)

+   *《Java 并行、并发与分布式编程专项课程》*，由莱斯大学在 Coursera 上提供。本专项课程全面覆盖了 Java 中的高级并发主题，包括用于开发高性能应用的并行、并发和分布式编程技术。链接：[`www.coursera.org/specializations/pcdp`](https://www.coursera.org/specializations/pcdp)

### 关键博客和网站

+   *《Baeldung》* 提供了关于 Java、Spring 以及相关技术的全面教程和文章，包括关于并发和并行性的深入内容。他们的并发部分特别有价值，有助于学习高级 Java 线程概念。链接：[`www.baeldung.com/java-concurrency`](https://www.baeldung.com/java-concurrency)

+   *《DZone Java 区》* 是一个社区驱动的平台，提供了大量关于 Java 和云原生开发的文章、教程和指南。Java 区是了解 Java 开发最新趋势和最佳实践的绝佳资源。链接：[`dzone.com/java-jdk-development-tutorials-tools-news`](https://dzone.com/java-jdk-development-tutorials-tools-news)

+   *《InfoQ Java》* 提供关于软件开发新闻、文章和访谈，重点在于 Java、并发和云原生技术。InfoQ 特别有助于深入了解 Java 生态系统中的行业趋势和新兴技术。链接：[`www.infoq.com/java/`](https://www.infoq.com/java/)

## 第 4-6 章

### 书籍

+   Unmesh Joshi（InfoQ）的 *《分布式系统模式》* 

+   本书概述了在分布式系统中使用的常见模式，提供了设计稳健和可扩展架构的实用建议。链接：[`www.amazon.com/Patterns-Distributed-Systems-Addison-Wesley-Signature/dp/0138221987`](https://www.amazon.com/Patterns-Distributed-Systems-Addison-Wesley-Signature/dp/0138221987)

### 文章和博客

+   马丁·福勒关于微服务和分布式系统的博客。这个博客是关于微服务和分布式系统的信息宝库，提供了深入的文章和现代软件架构的见解和领导力。链接：[`martinfowler.com/articles/microservices.html`](https://martinfowler.com/articles/microservices.html)

+   *LMAX Disruptor 文档和性能指南*是 Java 的高性能线程间消息库。该资源提供了实现低延迟、高吞吐量系统的文档和性能指南。链接：[`lmax-exchange.github.io/disruptor/`](https://lmax-exchange.github.io/disruptor/)

### 在线课程

+   *阿尔伯塔大学的微服务架构*。这门课程全面介绍了微服务架构，包括设计原则、实施策略以及构建可扩展和可维护系统的最佳实践。链接：[`www.coursera.org/specializations/software-design-architecture`](https://www.coursera.org/specializations/software-design-architecture)

+   *在 Coursera 上使用 Spring Boot 和 Spring Cloud 构建可扩展的 Java 微服务*，由谷歌云提供。谷歌云提供的这门课程教授如何使用 Spring Boot 和 Spring Cloud 构建可扩展的 Java 微服务，重点在于云原生开发实践。链接：[`www.coursera.org/learn/google-cloud-java-spring`](https://www.coursera.org/learn/google-cloud-java-spring)

## 第 7-9 章

### 书籍

+   *AWS 上的无服务器架构*，作者彼得·斯巴斯基。本书全面覆盖了无服务器概念和 AWS 上的实际应用，为寻求构建可扩展和成本效益应用的开发者提供了宝贵的见解。链接：[`www.amazon.com/Serverless-Architectures-AWS-Peter-Sbarski/dp/1617295426`](https://www.amazon.com/Serverless-Architectures-AWS-Peter-Sbarski/dp/1617295426)

### 文章

+   *无服务器计算：向前一步，退两步*，作者约瑟夫·M·赫勒斯坦等。这篇文章对无服务器计算进行了批判性分析，讨论了其优势和局限性，并对其在现代架构中的地位提供了平衡的观点。链接：[`arxiv.org/abs/1812.03651`](https://arxiv.org/abs/1812.03651)

+   *无服务器架构模式和最佳实践*，由 freeCodeCamp 提供

+   这篇文章概述了关键的无服务器模式，如消息传递、函数重点和事件驱动架构，强调了解耦和可扩展性的好处。链接：[`www.freecodecamp.org/news/serverless-architecture-patterns-and-best-practices/`](https://www.freecodecamp.org/news/serverless-architecture-patterns-and-best-practices/)

### 在线课程

+   *在 AWS 上开发无服务器解决方案*，由 AWS 培训团队提供。这包括对 AWS Lambda 的全面覆盖，最佳实践、框架和动手实验室。链接：[`aws.amazon.com/training/classroom/developing-serverless-solutions-on-aws/`](https://aws.amazon.com/training/classroom/developing-serverless-solutions-on-aws/)

### 技术论文

+   *无服务器计算：当前趋势和开放问题* 由 Ioana Baldini 等人撰写。这篇学术论文对无服务器计算进行了全面审查，讨论了该快速发展的领域的当前趋势、挑战和未来方向。链接：[`arxiv.org/abs/1706.03178`](https://arxiv.org/abs/1706.03178)

### 在线资源

+   AWS Lambda 开发者指南: [`docs.aws.amazon.com/lambda/latest/dg/welcome.html`](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)

+   Azure Functions Java 开发者指南: [`docs.microsoft.com/en-us/azure/azure-functions/functions-reference-java`](https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-java)

+   Google Cloud Functions Java 教程: [`codelabs.developers.google.com/codelabs/cloud-starting-cloudfunctions#`](https://codelabs.developers.google.com/codelabs/cloud-starting-cloudfunctions#)

# 第 10-12 章

### 书籍

+   由 Johan Vos 编著的 *开发者量子计算*。这本开创性的书籍为开发者提供了量子计算的友好介绍，架起了理论概念与实际应用之间的桥梁。它对量子原理进行了清晰的解释，并包括使用基于 Java 的框架的实践示例，为软件开发者准备即将出现的量子计算领域。作者 Johan Vos 精通量子算法、量子门和量子电路，展示了如何利用现有的编程技能在这个前沿领域。链接：[`www.manning.com/books/quantum-computing-for-developers`](https://www.manning.com/books/quantum-computing-for-developers)

### 文章

+   *过渡您的服务或应用程序* 由亚马逊网络服务

+   本文探讨了优化 Java 应用程序以提供更多关于将应用程序过渡到 Graviton2 所涉及的各个步骤的详细信息。链接：[`docs.aws.amazon.com/whitepapers/latest/aws-graviton2-for-isv/transitioning-your-service-or-application.html`](https://docs.aws.amazon.com/whitepapers/latest/aws-graviton2-for-isv/transitioning-your-service-or-application.html)

+   由 Cogent University 编著的 *云计算时代的 Java*。这篇文章重点介绍了 Java 的云原生进步，Spring Boot 和 Quarkus 等框架促进了基于云的开发。它还提到了 Maven、Gradle 和 JUnit 等工具，用于提高生产力和确保代码质量。链接：[`www.cogentuniversity.com/post/java-in-the-era-of-cloud-computing`](https://www.cogentuniversity.com/post/java-in-the-era-of-cloud-computing)

### 在线课程

+   由莱斯大学在 Coursera 提供的 *Java 并行、并发和分布式编程专项课程*。本专项课程涵盖了 Java 中的高级并发主题，这些主题适用于云计算环境和自动扩展场景。链接：[`www.coursera.org/specializations/pcdp`](https://www.coursera.org/specializations/pcdp)

+   *在 Coursera 上由 Google Cloud 提供的*《在 Google Cloud Platform 上使用 Tensorflow 的无服务器机器学习》*。这门课程探讨了无服务器计算和机器学习的交汇点，与云环境中 AI/ML 集成讨论和云计算的未来趋势相一致。链接：[`www.coursera.org/learn/serverless-machine-learning-gcp-br`](https://www.coursera.org/learn/serverless-machine-learning-gcp-br)

本附录提供了一系列精选资源，包括书籍、文章和在线课程，以加深你对并发、并行性和 Java 原生开发的了解。利用这些材料将增强你的知识和技能，使你能够构建健壮、可扩展和高效的 Java 原生应用程序。

# 章节末尾的多项选择题答案

## 第一章：并发、并行性和云：导航云原生景观

1.  B) 更容易扩展和维护单个服务

1.  B) 同步

1.  D) 流式 API

1.  C) 自动扩展和管理资源

1.  B) 数据一致性和同步

## 第二章：Java 并发基础介绍：线程、进程及其他

1.  C) 线程共享一个内存空间，而进程是独立的，并且有自己的内存。

1.  B) 它提供了一套用于高效管理线程和进程的类和接口。

1.  B) 允许多个线程并发读取资源，但写入时需要独占访问。

1.  B) 它允许一组线程等待一系列事件发生。

1.  B) 它允许对单个整数值进行无锁线程安全操作。

## 第三章：掌握 Java 中的并行性

1.  B) 通过递归拆分和执行任务来增强并行处理

1.  A) `RecursiveTask`返回一个值，而`RecursiveAction`不返回

1.  B) 它允许空闲线程接管忙碌线程的任务

1.  B) 平衡任务粒度和并行级别

1.  B) 任务的性质、资源可用性和团队的专业知识

## 第四章：云计算时代的 Java 并发工具和测试

1.  C) 为了有效地管理线程执行和资源分配

1.  B) `CopyOnWriteArrayList`

1.  B) 使异步编程和非阻塞操作成为可能

1.  B) 它们使高效的数据处理成为可能，并减少并发访问场景中的锁定开销

1.  C) 通过提供对锁管理的更多控制并减少锁竞争

## 第五章：掌握云计算中的并发模式

1.  C) 防止一个服务中的故障影响其他服务

1.  B) 使用无锁环形缓冲区以最小化竞争

1.  C) 它将服务隔离开来，防止一个服务的故障级联到其他服务。

1.  B) Scatter-Gather 模式

1.  D) 弹性和数据流管理

## 第六章：Java 与大数据——一次协作之旅

1.  B) 体积、速度和多样性

1.  A) **Hadoop 分布式文件系统**（**HDFS**）

1.  C) Spark 提供了更快的内存数据处理能力。

1.  A) Spark 只能处理结构化数据。

1.  C) 它有助于将大型数据集分解为更小、更易于管理的块进行处理。

## 第七章：Java 机器学习中的并发

1.  C) 为了优化计算效率

1.  C) 并行流

1.  C) 它们提高了可伸缩性并管理大规模计算。

1.  B) 更高效地执行数据预处理和模型训练

1.  B) 将 Java 并发与生成式 AI 结合

## 第八章：云中的微服务和 Java 的并发

1.  C) 独立部署和可伸缩性

1.  C) CompletableFuture

1.  B) 在多个实例之间分配传入的网络流量

1.  C) 断路器模式。

1.  C) 为每个微服务分配一个单独管理的数据库实例

## 第九章：无服务器计算与 Java 的并发能力

1.  C) 自动扩展和减少操作开销。

1.  B) CompletableFuture.

1.  C) 通过将任务划分为更小的子任务来管理递归任务。

1.  B) 优化函数大小并使用预留并发。

1.  B) 通过并发数据处理提高性能。

## 第十章：同步 Java 的并发与云自动扩展动态

1.  C) 根据需求动态分配资源

1.  B) `CompletableFuture`

1.  A) 管理固定数量的线程

1.  C) 实现无状态服务

1.  C) 通过并发数据处理提高性能

## 第十一章：云计算中的高级 Java 并发实践

1.  D) 用户界面设计

1.  C) 并行任务性能提升

1.  B) VisualVM

1.  B) 最小化数据丢失并提高可用性

1.  C) 获取分布式操作的统一视图的困难

## 第十二章：未来的展望

1.  A) 改善启动时间并减少内存使用

1.  D) 以上所有

1.  B) Google Cloud AutoML

1.  C) 量子叠加

1.  A) 减少管理基础设施的操作开销
