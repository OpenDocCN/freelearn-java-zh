# 序言

开发、部署和运行云应用应该像本地应用一样简单。这是任何云平台、库或工具背后的主导原则。Spring Cloud 使得在云中开发 JVM 应用变得容易。在这本书中，我们向你介绍 Spring Cloud 并帮助你掌握其功能。

你将学习配置 Spring Cloud 服务器并运行 Eureka 服务器以启用服务注册和发现。然后，你将学习与负载均衡和断路器相关的技术，并利用 Feign 客户端的所有功能。接着，我们将深入探讨高级主题，你将学习为 Spring Cloud 实现分布式跟踪解决方案，并构建基于消息的微服务架构。

# 本书面向对象

本书适合那些希望利用 Spring Cloud 这一开源库快速构建分布式系统的开发者。了解 Java 和 Spring Framework 知识将会有所帮助，但不需要先前的 Spring Cloud 经验。

# 本书涵盖内容

第一章，*微服务简介*，将向你介绍微服务架构、云环境等。你将学习微服务应用与单体应用之间的区别，同时学习如何将单体应用迁移到微服务应用。

第二章，*微服务与 Spring*，将向你介绍 Spring Boot 框架。你将学习如何有效地使用它来创建微服务应用。我们将涵盖诸如使用 Spring MVC 注解创建 REST API、使用 Swagger2 提供 API 文档、以及使用 Spring Boot Actuator 端点暴露健康检查和指标等主题。

第三章，*Spring Cloud 概览*，将简要介绍作为 Spring Cloud 一部分的主要项目。它将重点描述 Spring Cloud 实现的主要模式，并将它们分配给特定的项目。

第四章，*服务发现*，将描述一个使用 Spring Cloud Netflix Eureka 的服务发现模式。你将学习如何以独立模式运行 Eureka 服务器，以及如何运行具有对等复制的多个服务器实例。你还将学习如何在客户端启用发现功能，并在不同区域注册这些客户端。

第五章，*使用 Spring Cloud Config 的分布式配置*，将介绍如何在应用程序中使用 Spring Cloud Config 实现分布式配置。你将学习如何启用不同属性源的后端存储库，并使用 Spring Cloud Bus 推送变更通知。我们将比较发现首先引导和配置首先引导的方法，以说明发现服务与配置服务器之间的集成。

第六章，*微服务之间的通信*，将介绍参与服务间通信的最重要元素：HTTP 客户端和负载均衡器。您将学习如何使用 Spring RestTemplate、Ribbon 和 Feign 客户端，以及如何使用服务发现。

第七章，*高级负载均衡和断路器*，将介绍与微服务之间的服务通信相关的更高级主题。您将学习如何使用 Ribbon 客户端实现不同的负载均衡算法，使用 Hystrix 启用断路器模式，并使用 Hystrix 仪表板来监控通信统计。

第八章，*使用 API 网关的路由和过滤*，将比较两个用作 Spring Cloud 应用程序的 API 网关和代理的项目：Spring Cloud Netlix Zuul 和 Spring Cloud Gateway。您将学习如何将它们与服务发现集成，并创建简单和更高级的路由和过滤规则。

第九章，*分布式日志和跟踪*，将介绍一些用于收集和分析由微服务生成的日志和跟踪信息的热门工具。您将学习如何使用 Spring Cloud Sleuth 附加跟踪信息以及关联的消息。我们将运行一些示例应用程序，这些应用程序与 Elastic Stack 集成以发送日志消息，并与 Zipkin 收集跟踪。

第十章，*附加配置和发现特性*，将介绍两个用于服务发现和分布式配置的流行产品：Consul 和 ZooKeeper。您将学习如何本地运行这些工具，并将您的 Spring Cloud 应用程序与它们集成。

第十一章，*消息驱动的微服务*，将指导您如何为您的微服务提供异步、基于消息的通信。您将学习如何将 RabbitMQ 和 Apache Kafka 消息代理与您的 Spring Cloud 应用程序集成，以实现异步的一对一和发布/订阅通信方式。

第十二章，*保护 API*，将描述保护您的微服务的三种不同方法。我们将实现一个系统，该系统由前面介绍的所有元素组成，通过 SSL 相互通信。您还将学习如何使用 OAuth2 和 JWT 令牌来授权对 API 的请求。

第十三章，*测试 Java 微服务*，将介绍不同的微服务测试策略。它将重点介绍消费者驱动的合同测试，这在微服务环境中特别有用。您将了解如何使用 Hoverfly、Pact、Spring Cloud Contract、Gatling 等框架实现不同类型的自动化测试。

第十四章，*Docker 支持*，将简要介绍 Docker。它将重点介绍在容器化环境中运行和监控微服务最常用的 Docker 命令。您还将学习如何使用流行的持续集成服务器 Jenkins 构建和运行容器，并将它们部署在 Kubernetes 平台上。

第十五章，*云平台上的 Spring 微服务*，将介绍两种支持 Java 应用程序的流行云平台：Pivotal Cloud Foundry 和 Heroku。您将学习如何使用命令行工具或网络控制台在這些平台上部署、启动、扩展和监控您的应用程序。

# 为了充分利用本书

为了成功阅读本书并弄懂所有代码示例，我们期望读者满足以下要求：

+   活动互联网连接

+   Java 8+

+   Docker

+   Maven

+   Git 客户端

# 下载示例代码文件

您可以从 [www.packtpub.com](http://www.packtpub.com) 下载本书的示例代码文件。如果您在其他地方购买了此书，您可以访问 [www.packtpub.com/support](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  登录或注册 [www.packtpub.com](http://www.packtpub.com/support)。

1.  选择“支持”选项卡。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书籍名称，并按照屏幕上的指示操作。

文件下载完成后，请确保使用最新版本的软件解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，地址为 [`github.com/PacktPublishing/Mastering-Spring-Cloud`](https://github.com/PacktPublishing/Mastering-Spring-Cloud)。我们还有其他来自我们丰富目录的书籍和视频的代码包，可在 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)** 找到。去看看吧！

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、假 URL、用户输入和 Twitter 处理。例如：“HTTP API 端点的最后一个可用版本，`http://localhost:8889/client-service-zone3.yml`，返回与输入文件相同的数据。”

代码块如下所示：

```java
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

当我们希望吸引你对代码块的特定部分注意时，相关的行或项目会被设置为粗体：

```java
spring:
 rabbitmq:
  host: 192.168.99.100
  port: 5672
```

任何命令行输入或输出都如下所示：

```java
$ curl -H "X-Vault-Token: client" -X GET http://192.168.99.100:8200/v1/secret/client-service
```

**粗体**：表示新术语、重要词汇或你在屏幕上看到的词汇。例如，菜单或对话框中的词汇在文本中会以这种方式出现。示例：“在谷歌浏览器中，你可以通过访问设置*|*显示高级设置...*|*HTTPS/SSL*|*管理证书来导入一个 PKCS12 密钥库。”

警告或重要说明以这种方式出现。

技巧和窍门以这种方式出现。

# 联系我们

我们总是欢迎读者的反馈。

**一般反馈**：发送电子邮件至`feedback@packtpub.com`，并在消息主题中提及书籍标题。如果你对本书的任何方面有疑问，请通过`questions@packtpub.com`向我们发送电子邮件。

**勘误**：虽然我们已经尽一切努力确保内容的准确性，但错误仍然会发生。如果你在这本书中发现了错误，我们将非常感谢你能向我们报告。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择你的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果你在互联网上以任何形式遇到我们作品的非法副本，我们将非常感谢你能提供位置地址或网站名称。请通过`copyright@packtpub.com`联系我们，并提供材料的链接。

**如果你有兴趣成为作者**：如果你在某个话题上有专业知识，并且有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评审

请留下评论。一旦你阅读并使用了这本书，为什么不在这本书购买的网站上留下评论呢？潜在的读者可以看到和使用你的客观意见来做出购买决策，我们 Pactt 可以了解你对我们的产品的看法，我们的作者可以看到你对他们书籍的反馈。谢谢！

有关 Pactt 的更多信息，请访问[packtpub.com](https://www.packtpub.com/)。
