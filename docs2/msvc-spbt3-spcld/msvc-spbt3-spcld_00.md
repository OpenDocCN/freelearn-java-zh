# 前言

本书是关于使用 Spring Boot 3 和 Spring Cloud 构建生产就绪微服务的。十年前，当我开始探索微服务时，我正在寻找这样的书籍。

本书是在我学习并掌握了用于开发、测试、部署和管理协作微服务景观的开源软件之后开发的。

本书主要涵盖 Spring Boot、Spring Cloud、Docker、Kubernetes、Istio、EFK 堆栈、Prometheus 和 Grafana。这些开源工具各自都非常出色，但理解如何以有利的方式将它们结合使用可能具有挑战性。在某些领域，它们相互补充，但在其他领域它们存在重叠，对于特定情况选择哪一个并不明显。

这是一本实践性书籍，逐步描述了如何将这些开源工具结合使用。这是我十年前开始学习微服务时一直在寻找的书籍，但现在它涵盖了开源工具的更新版本。

# 本书面向的对象

本书面向的是希望学习如何从头开始构建微服务景观并在本地或云中部署它们的 Java 和 Spring 开发人员以及架构师，使用 Kubernetes 作为容器编排器，Istio 作为服务网格。开始阅读本书不需要对微服务架构有任何了解。

# 本书涵盖的内容

*第一章*，*微服务简介*，将帮助您理解本书的基本前提——微服务——以及与之相关的必要概念和设计模式。

*第二章*，*Spring Boot 简介*，将向您介绍 Spring Boot 3 以及本书第一部分将使用的其他开源项目：用于开发 RESTful API 的 Spring WebFlux、用于生成基于 OpenAPI 的 API 文档的 springdoc-openapi、用于在 SQL 和 NoSQL 数据库中存储数据的 Spring Data、用于基于消息的微服务的 Spring Cloud Stream 以及用于将微服务作为容器运行的 Docker。

*第三章*，*创建一组协作微服务*，将教会您从头开始创建一组协作微服务。您将使用 Spring Initializr 根据 Spring Framework 6.0 和 Spring Boot 3.0 创建基于 Spring 框架的骨架项目。

目标是创建三个核心服务（将处理自己的资源）和一个复合服务，该服务使用三个核心服务来聚合复合结果。在章节的末尾，您将学习如何添加基于 Spring WebFlux 的非常基本的 RESTful API。在接下来的章节中，将向这些微服务添加更多功能。

*第四章，* *使用 Docker 部署我们的微服务*，将教会您如何使用 Docker 部署微服务。您将学习如何添加 Dockerfile 和 docker-compose 文件，以便使用单个命令启动整个微服务景观。然后，您将学习如何使用多个 Spring 配置文件来处理有和无 Docker 的配置。

*第五章，* *使用 OpenAPI 添加 API 描述*，将使您熟悉使用 OpenAPI 记录微服务暴露的 API。您将使用 springdoc-openapi 工具注解服务，以动态创建基于 OpenAPI 的 API 文档。关键亮点将是如何使用 Swagger UI 在网页浏览器中测试 API。

*第六章，* *添加持久性*，将向您展示如何向微服务的数据中添加持久性。您将使用 Spring Data 为两个核心微服务设置和访问 MongoDB 文档数据库中的数据，并为剩余的微服务访问 MySQL 关系数据库中的数据。在运行集成测试时，将使用 Testcontainers 启动数据库。

*第七章，* *开发响应式微服务*，将向您讲解为什么以及何时响应式方法很重要，以及如何开发端到端响应式服务。您将学习如何开发和测试非阻塞同步 RESTful API 和异步事件驱动服务。您还将学习如何使用 MongoDB 的响应式非阻塞驱动程序，以及如何使用传统的阻塞代码来处理 MySQL。

*第八章，* *Spring Cloud 简介*，将向您介绍 Spring Cloud 以及本书中将使用的 Spring Cloud 组件。

*第九章，* *使用 Netflix Eureka 添加服务发现*，将向您展示如何在 Spring Cloud 中使用 Netflix Eureka 来添加服务发现功能。这将通过向系统景观中添加一个基于 Netflix Eureka 的服务发现服务器来实现。然后，您将配置微服务使用 Spring Cloud LoadBalancer 来查找其他微服务。您将了解微服务是如何自动注册的，以及当新实例可用时，通过 Spring Cloud LoadBalancer 自动负载均衡到新实例的流量。

*第十章，* *使用 Spring Cloud Gateway 在边缘服务器后面隐藏微服务*，将指导您如何使用 Spring Cloud Gateway 在边缘服务器后面隐藏微服务，并且只向外部消费者公开选择 API。您还将学习如何从外部消费者隐藏微服务的内部复杂性。这将通过向系统景观中添加一个基于 Spring Cloud Gateway 的边缘服务器并配置它只公开公共 API 来实现。

*第十一章*，*保护 API 访问安全*，将解释如何使用 OAuth 2.0 和 OpenID Connect 保护暴露的 API。你将学习如何将基于 Spring Authorization Server 的 OAuth 2.0 授权服务器添加到系统架构中，以及如何配置边缘服务器和组合服务以要求由该授权服务器签发的有效访问令牌。

你将学习如何通过边缘服务器暴露授权服务器，并使用 HTTPS 确保其与外部消费者的通信安全。最后，你将学习如何用 Auth0 的外部 OpenID Connect 提供者替换内部的 OAuth 2.0 授权服务器。

*第十二章*，*集中式配置*，将处理如何从所有微服务中收集配置文件到一个中央存储库，并使用配置服务器在运行时将配置分发到微服务。你还将学习如何将 Spring Cloud Config Server 添加到系统架构中，并配置所有微服务使用 Spring Config Server 来获取其配置。

*第十三章*，*使用 Resilience4j 提高弹性*，将解释如何使用 Resilience4j 的能力来防止例如“失败链”反模式。你将学习如何向组合服务添加重试机制和断路器，如何配置断路器在电路打开时快速失败，以及如何利用回退方法创建尽力而为的响应。

*第十四章*，*理解分布式跟踪*，将展示如何使用 Zipkin 收集和可视化跟踪信息。你还将使用 Micrometer Tracing 将跟踪 ID 添加到请求中，以便可视化协作微服务之间的请求链。

*第十五章*，*Kubernetes 简介*，将解释 Kubernetes 的核心概念以及如何执行一个示例部署。你还将学习如何使用 Minikube 在本地设置 Kubernetes 以用于开发和测试目的。

*第十六章*，*将我们的微服务部署到 Kubernetes*，将展示如何在 Kubernetes 上部署微服务。你还将学习如何使用 Helm 打包和配置微服务以便在 Kubernetes 上部署。Helm 将被用于部署针对不同运行环境（如测试和生产环境）的微服务。最后，你将学习如何用 Kubernetes 内置的服务发现支持替换 Netflix Eureka，这基于 Kubernetes 服务对象和 kube-proxy 运行时组件。

*第十七章*，*通过实现 Kubernetes 功能简化系统架构*，将解释如何使用 Kubernetes 功能作为替代上一章中介绍的 Spring Cloud 服务的方案。你将了解为什么以及如何用 Kubernetes Secrets 和 ConfigMaps 替换 Spring Cloud Config Server，以及为什么以及如何用 Kubernetes Ingress 对象替换 Spring Cloud Gateway，并学习如何添加 cert-manager 以自动为外部 HTTPS 端点提供和轮换证书。

*第十八章*，*使用服务网格提高可观察性和管理*，将介绍服务网格的概念，并解释如何使用 Istio 在 Kubernetes 中运行时实现服务网格。你将学习如何使用服务网格进一步改进微服务景观的弹性、安全性、流量管理和可观察性。

*第十九章*，*使用 EFK Stack 进行集中日志记录*，将解释如何使用 Elasticsearch、Fluentd 和 Kibana（EFK Stack）来收集、存储和可视化来自微服务的日志流。你将学习如何在 Minikube 中部署 EFK Stack，以及如何使用它来分析收集到的日志记录并找到涉及跨多个微服务的请求处理的微服务的日志输出。你还将学习如何使用 EFK Stack 进行根本原因分析。

*第二十章*，*监控微服务*，将展示如何使用 Prometheus 和 Grafana 监控在 Kubernetes 中部署的微服务。你将学习如何使用 Grafana 中的现有仪表板来监控不同类型的指标，并学习如何创建自己的仪表板。最后，你将学习如何在 Grafana 中创建警报，当配置的阈值被选定的指标超过时，这些警报将被用来发送带有警报的电子邮件。

*第二十一章*，*macOS 安装说明*，将展示如何在 Mac 上安装本书中使用的工具。涵盖了基于 Intel 和 Apple 硅（ARM64）的 Mac。

*第二十二章*，*使用 WSL 2 和 Ubuntu 的 Microsoft Windows 安装说明*，将展示如何使用 Windows Subsystem for Linux (WSL) v2 在 Windows PC 上安装本书中使用的工具。

*第二十三章*，*原生编译的 Java 微服务*，将展示如何创建编译为原生代码的基于 Spring 的微服务。你将学习如何使用 Spring Framework 6 和 Spring Boot 3 中的新原生映像支持以及底层的 GraalVM Native Image 编译器。与使用常规的 Java 虚拟机相比，这将导致可以几乎立即启动的微服务。

每章结束时，你将找到一些简单的问题，这些问题将帮助你回顾章节中涵盖的一些内容。"评估" 是一个可以在 GitHub 仓库中找到的文件，其中包含这些问题的答案。

# 为了充分利用本书

推荐具备 Java 和 Spring 的基本理解。

要运行本书中的所有内容，您需要拥有一台基于 Mac Intel 或 Apple 硅的电脑或至少 16GB 内存的 PC，尽管建议您至少有 24GB，因为随着本书末尾微服务领域变得更加复杂和资源密集，所需的资源也更多。

要获取软件要求的完整列表以及设置环境以便能够跟随本书的详细说明，请参阅*第二十一章*（针对 macOS）和*第二十二章*（针对 Windows）。

## 下载示例代码文件

本书代码包托管在 GitHub 上，网址为[`github.com/PacktPublishing/Microservices-with-Spring-Boot-and-Spring-Cloud-Third-Edition`](https://github.com/PacktPublishing/Microservices-with-Spring-Boot-and-Spring-Cloud-Third-Edition)。我们还有其他丰富的图书和视频代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。请查看它们！

## 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`packt.link/XHJmq`](https://packt.link/XHJmq)。

## 使用的约定

本书使用了多种文本约定。

`CodeInText`: 表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。例如：“`java`插件将 Java 编译器添加到项目中。”

代码块设置如下：

```java
package se.magnus.microservices.core.product;
@SpringBootApplication
public class ProductServiceApplication { 
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
package se.magnus.microservices.core.product;
**@SpringBootTest**
class ProductServiceApplicationTests { 
```

任何命令行输入或输出都按以下方式编写：

```java
mkdir some-temp-folder
cd some-temp-folder 
```

**粗体**: 表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下：“使用**Spring Initializr**为每个微服务生成骨架项目。”

警告或重要注意事项看起来像这样。

技巧和窍门看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**: 请通过`feedback@packtpub.com`发送电子邮件，并在邮件主题中提及本书的标题。如果您对本书的任何方面有疑问，请通过`questions@packtpub.com`与我们联系。

**勘误**: 尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，如果您能向我们报告，我们将不胜感激。请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，点击**提交勘误**，并填写表格。

**盗版**: 如果您在互联网上以任何形式遇到我们作品的非法副本，如果您能向我们提供位置地址或网站名称，我们将不胜感激。请通过`copyright@packtpub.com`与我们联系，并提供材料的链接。

**如果您想成为一名作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[`authors.packtpub.com`](http://authors.packtpub.com)。

# 分享您的想法

一旦您阅读了《使用 Spring Boot 3 和 Spring Cloud 的微服务，第三版》，我们很乐意听听您的想法！请[点击此处直接进入此书的亚马逊评论页面](https://packt.link/r/1-805-12869-8)并分享您的反馈。

您的评论对我们和科技社区非常重要，并将帮助我们确保我们提供高质量的内容。

# 下载此书的免费 PDF 副本

感谢您购买此书！

您喜欢在路上阅读，但无法携带您的印刷书籍到处走？您的电子书购买是否与您选择的设备不兼容？

别担心，现在每购买一本 Packt 图书，您都可以免费获得该书的 DRM 免费 PDF 版本。

在任何地方、任何设备上阅读。从您最喜欢的技术书籍中直接搜索、复制和粘贴代码到您的应用程序中。

优惠远不止这些，您还可以获得独家折扣、时事通讯和丰富的免费内容，每天直接发送到您的邮箱。

按照以下简单步骤获取好处：

1.  扫描下面的二维码或访问以下链接

![](img/B19825_FreePDF_QR.png)

[`packt.link/free-ebook/9781805128694`](https://packt.link/free-ebook/9781805128694)

1.  提交您的购买证明

1.  那就结束了！我们将直接将您的免费 PDF 和其他好处发送到您的电子邮件。

## 目录

1.  前言

    1.  本书面向的对象

    1.  本书涵盖的内容

    1.  联系我们

    1.  分享您的想法

1.  微服务简介

    1.  技术要求

    1.  我的微服务之路

        1.  自主软件组件的优势

        1.  自主软件组件的挑战

        1.  进入微服务

        1.  一个示例微服务景观

    1.  定义微服务

    1.  微服务面临的挑战

    1.  微服务的设计模式

        1.  服务发现

            1.  问题

            1.  解决方案

            1.  解决方案需求

        1.  边缘服务器

            1.  问题

            1.  解决方案

            1.  解决方案需求

        1.  响应式微服务

            1.  问题

            1.  解决方案

            1.  解决方案需求

        1.  集中配置

            1.  问题

            1.  解决方案

            1.  解决方案需求

        1.  集中日志分析

            1.  问题

            1.  解决方案

            1.  解决方案需求

        1.  分布式追踪

            1.  问题

            1.  解决方案

            1.  解决方案需求

        1.  断路器

            1.  问题

            1.  解决方案

            1.  解决方案需求

        1.  控制循环

            1.  问题

            1.  解决方案

            1.  解决方案需求

        1.  集中监控和警报

            1.  问题

            1.  解决方案

            1.  解决方案需求

    1.  软件使能器

    1.  其他重要注意事项

    1.  总结

1.  Spring Boot 简介

    1.  技术要求

    1.  Spring Boot

        1.  约定优于配置和胖 JAR 文件

        1.  设置 Spring Boot 应用程序的代码示例

            1.  神奇的 @SpringBootApplication 注解

            1.  组件扫描

            1.  基于 Java 的配置

        1.  Spring Boot 3.0 的新特性

        1.  迁移 Spring Boot 2 应用程序

    1.  Spring WebFlux

        1.  设置 REST 服务的代码示例

            1.  启动依赖项

            1.  属性文件

            1.  示例 RestController

    1.  springdoc-openapi

    1.  Spring 数据

        1.  实体

        1.  存储库

    1.  Spring Cloud Stream

        1.  发送和接收消息的代码示例

    1.  Docker

    1.  总结

    1.  问题

1.  创建一组协作的微服务

    1.  技术要求

    1.  介绍微服务景观

        1.  由微服务处理的信息

            1.  产品服务

            1.  审查服务

            1.  推荐服务

            1.  产品组合服务

            1.  与基础设施相关的信息

        1.  临时替换服务发现

    1.  生成骨架微服务

        1.  使用 Spring Initializr 生成骨架代码

        1.  在 Gradle 中设置多项目构建

    1.  添加 RESTful API

        1.  添加 API 和 util 项目

            1.  api 项目

            1.  util 项目

        1.  实现我们的 API

    1.  添加复合微服务

        1.  API 类

        1.  属性

        1.  集成组件

        1.  复合 API 实现

    1.  添加错误处理

        1.  全局 REST 控制器异常处理器

        1.  API 实现中的错误处理

        1.  API 客户端中的错误处理

    1.  手动测试 API

    1.  在隔离中添加自动微服务测试

    1.  添加微服务景观的半自动化测试

        1.  尝试测试脚本

    1.  总结

    1.  问题

1.  使用 Docker 部署我们的微服务

    1.  技术要求

    1.  Docker 简介

        1.  运行我们的第一个 Docker 命令

    1.  在 Docker 中运行 Java

        1.  限制可用的 CPU

        1.  限制可用的内存

    1.  使用 Docker 与单个微服务

        1.  源代码中的更改

        1.  构建 Docker 镜像

        1.  启动服务

        1.  以分离模式运行容器

    1.  使用 Docker Compose 管理微服务景观

        1.  源代码中的更改

        1.  启动微服务架构

    1.  自动化测试协作微服务

        1.  调试测试运行

    1.  总结

    1.  问题

1.  使用 OpenAPI 添加 API 描述

    1.  技术要求

    1.  Springdoc-openapi 使用介绍

    1.  将 springdoc-openapi 添加到源代码中

        1.  向 Gradle 构建文件添加依赖项

        1.  向 ProductCompositeService 添加 OpenAPI 配置和通用 API 文档

        1.  向 ProductCompositeService 接口添加 API 特定文档

    1.  构建和启动微服务架构

    1.  尝试 OpenAPI 文档

    1.  总结

    1.  问题

1.  添加持久性

    1.  技术要求

    1.  章节目标

    1.  向核心微服务添加持久层

        1.  添加依赖项

        1.  使用实体类存储数据

        1.  在 Spring Data 中定义仓库

    1.  编写关注持久性的自动化测试

        1.  使用 Testcontainers

        1.  编写持久性测试

    1.  在服务层中使用持久层

        1.  记录数据库连接 URL

        1.  添加新的 API

        1.  从服务层调用持久层

        1.  声明 Java Bean 映射器

        1.  更新服务测试

    1.  扩展组合服务 API

        1.  向组合服务 API 添加新操作

        1.  向集成层添加方法

        1.  实现新的组合 API 操作

        1.  更新组合服务测试

    1.  向 Docker Compose 架构添加数据库

        1.  Docker Compose 配置

        1.  数据库连接配置

        1.  MongoDB 和 MySQL CLI 工具

    1.  对新 API 和持久层进行手动测试

    1.  更新微服务景观的自动化测试

    1.  总结

    1.  问题

1.  开发反应式微服务

    1.  技术要求

    1.  在非阻塞同步 API 和事件驱动异步服务之间选择

    1.  开发非阻塞同步 REST API

        1.  Project Reactor 简介

        1.  使用 Spring Data for MongoDB 进行非阻塞持久化

            1.  测试代码的变更

        1.  核心服务中的非阻塞 REST API

            1.  API 变更

            1.  服务实现变更

            1.  测试代码的变更

            1.  处理阻塞代码

        1.  组合服务中的非阻塞 REST API

            1.  API 变更

            1.  服务实现变更

            1.  集成层的变更

            1.  测试代码的变更

    1.  开发事件驱动异步服务

        1.  处理消息挑战

            1.  消费者组

            1.  重试和死信队列

            1.  保证顺序和分区

        1.  定义主题和事件

        1.  Gradle 构建文件的变更

        1.  在核心服务中消费事件

            1.  声明消息处理器

            1.  服务实现变更

            1.  为消费事件添加配置

            1.  测试代码的变更

        1.  在组合服务中发布事件

            1.  在集成层发布事件

            1.  为发布事件添加配置

            1.  测试代码的变更

    1.  运行反应式微服务景观的手动测试

        1.  保存事件

        1.  添加健康 API

        1.  不使用分区使用 RabbitMQ

        1.  使用分区与 RabbitMQ

        1.  使用每个主题两个分区的 Kafka

    1.  运行反应式微服务架构的自动化测试

    1.  总结

    1.  问题

1.  Spring Cloud 简介

    1.  技术要求

    1.  Spring Cloud 的演变

    1.  使用 Netflix Eureka 进行服务发现

    1.  使用 Spring Cloud Gateway 作为边缘服务器

    1.  使用 Spring Cloud Config 进行集中配置

    1.  使用 Resilience4j 提高弹性

        1.  Resilience4j 中熔断器的示例用法

    1.  使用 Micrometer Tracing 和 Zipkin 进行分布式跟踪

    1.  总结

    1.  问题

1.  使用 Netflix Eureka 添加服务发现

    1.  技术要求

    1.  介绍服务发现

        1.  基于 DNS 的服务发现的问题

        1.  服务发现挑战

        1.  在 Spring Cloud 中使用 Netflix Eureka 进行服务发现

    1.  设置 Netflix Eureka 服务器

    1.  将微服务连接到 Netflix Eureka 服务器

    1.  设置开发使用的配置

        1.  Eureka 配置参数

        1.  配置 Eureka 服务器

        1.  配置 Eureka 服务器的客户端

    1.  尝试发现服务

        1.  扩大规模

        1.  缩小规模

        1.  使用 Eureka 服务器进行破坏性测试

            1.  停止 Eureka 服务器

            1.  启动产品服务的额外实例

    1.  再次启动 Eureka 服务器

    1.  总结

    1.  问题

1.  使用 Spring Cloud Gateway 在边缘服务器后面隐藏微服务

    1.  技术要求

    1.  将边缘服务器添加到我们的系统架构中

    1.  设置 Spring Cloud Gateway

        1.  添加组合健康检查

        1.  配置 Spring Cloud Gateway

            1.  路由规则

    1.  尝试使用边缘服务器

        1.  检查 Docker 引擎外部暴露的内容

        1.  尝试路由规则

            1.  通过边缘服务器调用产品组合 API

            1.  通过边缘服务器调用 Swagger UI

            1.  通过边缘服务器调用 Eureka

            1.  基于主机头进行路由

    1.  总结

    1.  问题

1.  保护 API 访问

    1.  技术要求

    1.  OAuth 2.0 和 OpenID Connect 简介

        1.  介绍 OAuth 2.0

        1.  介绍 OpenID Connect

    1.  保护系统景观

    1.  使用 HTTPS 保护外部通信

        1.  在运行时替换自签名证书

    1.  保护发现服务器的访问

        1.  Eureka 服务器的变化

        1.  Eureka 客户端的变化

    1.  添加本地授权服务器

    1.  使用 OAuth 2.0 和 OpenID Connect 保护 API

        1.  边缘服务器和产品组合服务的变化

        1.  仅针对产品组合服务的变化

            1.  更改以允许 Swagger UI 获取访问令牌

        1.  测试脚本的变化

    1.  使用本地授权服务器进行测试

        1.  构建和运行自动化测试

        1.  测试受保护发现服务器

        1.  获取访问令牌

            1.  使用客户端凭据授权流程获取访问令牌

            1.  使用授权码授权流程获取访问令牌

        1.  使用访问令牌调用受保护 API

        1.  使用 OAuth 2.0 测试 Swagger UI

    1.  使用外部 OpenID Connect 提供者进行测试

        1.  在 Auth0 中设置和配置账户

        1.  应用必要的更改以使用 Auth0 作为 OpenID 提供者

            1.  更改 OAuth 资源服务器中的配置

            1.  更改测试脚本以从 Auth0 获取访问令牌

        1.  使用 Auth0 作为 OpenID Connect 提供者运行测试脚本

        1.  使用客户端凭据授权流程获取访问令牌

        1.  使用授权代码授权流程获取访问令牌

        1.  使用 Auth0 访问令牌调用受保护的 API

        1.  获取有关用户的额外信息

    1.  摘要

    1.  问题

1.  集中式配置

    1.  技术要求

    1.  Spring Cloud Config Server 简介

        1.  选择配置存储库的存储类型

        1.  决定初始客户端连接

        1.  保护配置

            1.  保护传输中的配置

            1.  保护静态配置

        1.  介绍配置服务器 API

    1.  设置配置服务器

        1.  在边缘服务器上设置路由规则

        1.  为与 Docker 一起使用配置配置服务器

    1.  配置配置服务器的客户端

        1.  配置连接信息

    1.  结构化配置存储库

    1.  尝试使用 Spring Cloud Config Server

        1.  构建和运行自动化测试

        1.  使用配置服务器 API 获取配置

        1.  加密和解密敏感信息

    1.  摘要

    1.  问题

1.  使用 Resilience4j 改进弹性

    1.  技术要求

    1.  介绍 Resilience4j 弹性机制

        1.  介绍断路器

        1.  介绍时间限制器

        1.  介绍重试机制

    1.  将弹性机制添加到源代码中

        1.  添加可编程延迟和随机错误

            1.  API 定义中的更改

            1.  产品组合微服务中的更改

            1.  产品微服务中的更改

        1.  添加断路器和时间限制器

            1.  将依赖项添加到构建文件中

            1.  在源代码中添加注解

            1.  添加快速失败回退逻辑

            1.  添加配置

        1.  添加重试机制

            1.  添加重试注解

            1.  添加配置

        1.  添加自动化测试

    1.  尝试断路器和重试机制

        1.  构建和运行自动化测试

        1.  验证在正常操作下电路是否闭合

        1.  在出错时强制打开断路器

        1.  再次关闭断路器

        1.  尝试由随机错误引起的重试

    1.  总结

    1.  问题

1.  理解分布式跟踪

    1.  技术要求

    1.  使用 Micrometer Tracing 和 Zipkin 介绍分布式跟踪

    1.  将分布式跟踪添加到源代码中

        1.  将依赖项添加到构建文件中

        1.  添加 Micrometer Tracing 和 Zipkin 的配置

        1.  将 Zipkin 添加到 Docker Compose 文件中

        1.  添加对缺乏支持反应式客户端的解决方案

        1.  向现有跨度添加自定义跨度自定义标签

            1.  添加自定义跨度

            1.  向现有跨度添加自定义标签

    1.  尝试分布式跟踪

        1.  启动系统景观

        1.  发送成功的 API 请求

        1.  发送失败的 API 请求

        1.  发送触发异步处理的 API 请求

    1.  总结

    1.  问题

1.  Kubernetes 简介

    1.  技术要求

    1.  介绍 Kubernetes 概念

    1.  介绍 Kubernetes API 对象

    1.  介绍 Kubernetes 运行时组件

    1.  使用 Minikube 创建 Kubernetes 集群

        1.  使用 Minikube 配置文件

        1.  使用 Kubernetes CLI，kubectl

        1.  使用 kubectl 管理上下文

        1.  创建 Kubernetes 集群

    1.  尝试一个示例部署

    1.  管理本地 Kubernetes 集群

        1.  休眠和恢复 Kubernetes 集群

        1.  终止 Kubernetes 集群

    1.  总结

    1.  问题

1.  将我们的微服务部署到 Kubernetes

    1.  技术要求

    1.  用 Kubernetes 服务替换 Netflix Eureka

    1.  介绍 Kubernetes 的使用方法

    1.  使用 Spring Boot 对优雅关闭和存活性/就绪性探测的支持

    1.  介绍 Helm

        1.  运行 Helm 命令

        1.  查看 Helm 图表

        1.  Helm 模板和值

        1.  通用库图表

            1.  ConfigMap 模板

            1.  秘密模板

            1.  服务模板

            1.  部署模板

        1.  组件图表

        1.  环境图表

    1.  将应用部署到 Kubernetes 进行开发和测试

        1.  构建 Docker 镜像

        1.  解决 Helm 图表依赖关系

        1.  将应用部署到 Kubernetes

        1.  用于 Kubernetes 的测试脚本更改

        1.  测试部署

            1.  测试 Spring Boot 对优雅关闭和存活性/就绪性探测的支持

    1.  将应用部署到 Kubernetes 进行预发布和生产

        1.  源代码中的更改

        1.  将应用部署到 Kubernetes

        1.  清理

    1.  总结

    1.  问题

1.  实现 Kubernetes 功能以简化系统架构

    1.  技术要求

    1.  替换 Spring Cloud Config Server

        1.  替换 Spring Cloud Config Server 所需的更改

    1.  替换 Spring Cloud Gateway

        1.  替换 Spring Cloud Gateway 所需的更改

    1.  自动化证书提供

    1.  使用 Kubernetes ConfigMaps、Secrets、Ingress 和 cert-manager 进行测试

        1.  轮换证书

        1.  部署到 Kubernetes 进行预演和生产

    1.  验证微服务在没有 Kubernetes 的情况下工作

        1.  Docker Compose 文件中的更改

        1.  使用 Docker Compose 进行测试

    1.  摘要

    1.  问题

1.  使用服务网格提高可观察性和管理

    1.  技术要求

    1.  使用 Istio 介绍服务网格

        1.  介绍 Istio

        1.  将 Istio 代理注入到微服务中

        1.  介绍 Istio API 对象

    1.  简化微服务景观

        1.  用 Istio 入口网关替换 Kubernetes Ingress 控制器

        1.  用 Istio 的 Jaeger 组件替换 Zipkin 服务器

    1.  在 Kubernetes 集群中部署 Istio

        1.  设置访问 Istio 服务的权限

    1.  创建服务网格

        1.  源代码更改

            1.  在 _istio_base.yaml 模板中的内容

            1.  在 _istio_dr_mutual_tls.yaml 模板中的内容

        1.  运行命令创建服务网格

        1.  记录跟踪和跨度 ID 的传播

    1.  观察服务网格

    1.  保护服务网格

        1.  使用 HTTPS 和证书保护外部端点

        1.  使用 OAuth 2.0/OIDC 访问令牌验证外部请求

        1.  使用双向身份验证（mTLS）保护内部通信

    1.  确保服务网格具有弹性

        1.  通过注入故障测试弹性

        1.  通过注入延迟测试弹性

    1.  执行零停机更新

        1.  源代码更改

            1.  虚拟服务和目标规则

            1.  部署和服务

            1.  在 prod-env Helm 图表中整合所有内容

        1.  部署微服务的 v1 和 v2 版本，并通过路由到 v1 版本

        1.  验证所有流量最初都流向微服务的 v1 版本

        1.  运行金丝雀测试

        1.  运行蓝绿部署

            1.  kubectl patch 命令的简要介绍

            1.  执行蓝绿部署

    1.  使用 Docker Compose 运行测试

    1.  总结

    1.  问题

1.  使用 EFK 堆栈进行集中式日志记录

    1.  技术要求

    1.  介绍 Fluentd

        1.  Fluentd 概述

        1.  配置 Fluentd

    1.  在 Kubernetes 上部署 EFK 堆栈

        1.  构建和部署我们的微服务

        1.  部署 Elasticsearch 和 Kibana

            1.  manifest 文件的概述

            1.  运行部署命令

        1.  部署 Fluentd

            1.  manifest 文件的概述

            1.  运行部署命令

    1.  尝试 EFK 堆栈

        1.  初始化 Kibana

        1.  分析日志记录

        1.  从微服务中发现日志记录

        1.  执行根本原因分析

    1.  总结

    1.  问题

1.  监控微服务

    1.  技术要求

    1.  使用 Prometheus 和 Grafana 进行性能监控简介

    1.  修改源代码以收集应用程序度量

    1.  构建和部署微服务

    1.  使用 Grafana 仪表板监控微服务

        1.  为测试安装本地邮件服务器

        1.  配置 Grafana

        1.  启动负载测试

        1.  使用 Kiali 内置仪表板

        1.  导入现有的 Grafana 仪表板

        1.  开发您自己的 Grafana 仪表板

            1.  检查 Prometheus 指标

            1.  创建仪表板

            1.  尝试新的仪表板

        1.  导出和导入 Grafana 仪表板

    1.  在 Grafana 中设置警报

        1.  设置基于邮件的通知通道

        1.  在断路器上设置警报

        1.  尝试断路器警报

    1.  总结

    1.  问题

1.  macOS 安装说明

    1.  技术要求

    1.  安装工具

        1.  安装 Homebrew

        1.  使用 Homebrew 安装工具

        1.  不使用 Homebrew 安装工具

            1.  在基于 Intel 的 Mac 上安装工具

            1.  在基于 Apple 硅的 Mac 上安装工具

        1.  安装后的操作

        1.  验证安装

    1.  访问源代码

        1.  使用 IDE

        1.  代码结构

    1.  总结

1.  带有 WSL 2 和 Ubuntu 的 Microsoft Windows 安装说明

    1.  技术要求

    1.  安装工具

        1.  在 Windows 上安装工具

            1.  与默认 Ubuntu 服务器一起安装 WSL 2

            1.  在 WSL 2 上安装新的 Ubuntu 22.04 服务器

            1.  安装 Windows Terminal

            1.  安装 Docker Desktop for Windows

            1.  安装 Visual Studio Code 及其远程 WSL 扩展

        1.  在 WSL 2 的 Linux 服务器上安装工具

            1.  使用 apt install 安装工具

            1.  使用 SDKman 安装 Java 和 Spring Boot CLI

            1.  使用 curl 和 install 安装剩余的工具

            1.  验证安装

    1.  访问源代码

        1.  代码结构

    1.  总结

1.  原生编译的 Java 微服务

    1.  技术要求

    1.  何时原生编译 Java 源代码

    1.  介绍 GraalVM 项目

    1.  介绍 Spring 的 AOT 引擎

    1.  处理原生编译问题

    1.  源代码中的更改

        1.  Gradle 构建文件的更新

        1.  提供可达性元数据和自定义提示

        1.  在 application.yml 文件中构建时启用 Spring Bean

        1.  更新运行时属性

        1.  配置 GraalVM 原生镜像跟踪代理

        1.  test-em-all.bash 验证脚本的更新

    1.  测试和编译原生镜像

        1.  安装 GraalVM 及其原生镜像编译器

        1.  运行跟踪代理

        1.  运行原生测试

        1.  为当前操作系统创建原生镜像

        1.  将原生镜像作为 Docker 镜像创建

    1.  使用 Docker Compose 进行测试

        1.  禁用 AOT 模式测试基于 Java VM 的微服务

        1.  启用 AOT 模式测试基于 Java VM 的微服务

        1.  测试原生编译的微服务

    1.  使用 Kubernetes 进行测试

    1.  摘要

    1.  问题

1.  你可能喜欢的其他书籍

1.  索引

# 标记

1.  封面

1.  索引
