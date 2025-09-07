# 前言

Spring Boot 是 Java 在 Web 和微服务开发中最受欢迎的框架。它允许你通过遵循“约定优于配置”的方法，以最小的配置创建生产级的应用程序。

Spring Boot 始终在演变和适应最新的技术趋势。其生态系统允许你与任何技术集成，从数据库到 AI，并使用诸如可观测性和安全性等跨度功能。你可以用它来开发几乎任何类型的应用程序。

在本书中，我们将通过实际操作的方式涵盖最常见的场景，并帮助你掌握使用大量功能的基础。

# 本书面向的对象

本书面向希望获得现代 Web 开发专业知识的 Java 开发者、设计复杂系统的架构师、经验丰富的 Spring Boot 开发者和希望跟上最新趋势的技术爱好者，以及需要解决日常挑战的软件工程师。需要具备 Java 的实际操作经验。在云上的开发经验将很有用，但不是必需的。

# 本书涵盖的内容

*第一章*, *构建 RESTful API*，教你如何使用 Spring Boot 3 编写、消费和测试 RESTful API。

*第二章*, *使用 OAuth2 保护 Spring Boot 应用程序*，展示了如何部署授权服务器并使用它来保护 RESTful API 和网站。你将学习如何使用 Google 账户进行用户认证。你还将学习如何使用 Azure AD B2C 保护应用程序。

*第三章*, *可观测性、监控和应用管理*，探讨了如何利用 Spring Boot 中 Actuator 提供的可观测性功能。本章使用 Open Zipkin、Prometheus 和 Grafana，消费由 Spring Boot 应用程序暴露的可观测性数据以进行监控。

*第四章*, *Spring Cloud*，介绍了如何使用 Spring Cloud 开发由多个微服务组成的分布式系统。我们将使用 Eureka Server、Spring Cloud Gateway、Spring Config 和 Spring Boot Admin。

*第五章*, *使用 Spring Data 与关系型数据库进行数据持久化和集成*，深入探讨了如何使用 Spring Data JPA 将应用程序与 PostgreSQL 集成。你将定义仓库并使用**Java 持久化查询语言**（**JPQL**）和原生 SQL。你将学习使用事务、数据库版本控制和 Flyway 以及 Testcontainers 进行集成测试。

*第六章*，*使用 Spring Data 与 NoSQL 数据库进行数据持久性和集成*，解释了使用如 MongoDB 和 Cassandra 等 NoSQL 数据库的优缺点，教你如何应对 NoSQL 数据库的一些常见挑战，例如数据分区或并发管理。你将使用 Testcontainers 对 MongoDB 和 Cassandra 进行集成测试。

*第七章*，*寻找瓶颈并优化你的应用程序*，描述了如何使用 JMeter 对 Spring Boot 应用程序进行负载测试，应用不同的优化，如缓存或构建本地应用程序，并将改进与原始结果进行比较。你还将学习一些有用的技术来准备本地应用程序，例如使用 GraalVM 追踪代理。

*第八章*，*Spring Reactive 和 Spring Cloud Stream*，探讨了如何使用 Spring Reactive 处理高并发场景，创建一个反应式 RESTful API、一个反应式 API 客户端以及 PostgreSQL 的 R2DBC 驱动程序。你将学习如何创建一个使用 Spring Cloud Stream 连接到 RabbitMQ 服务器的基于事件的驱动应用程序。

*第九章*，*从 Spring Boot 2.x 升级到 Spring Boot 3.0*，解释了如何手动将 Spring Boot 2.6 应用程序升级到 Spring Boot 的最新版本。在升级到 Spring Boot 3 之前，你需要准备应用程序，并在升级后逐步修复所有问题。你还将学习如何使用 OpenRewrite 来自动化迁移过程的一部分。

# 为了充分利用这本书

你将需要 JDK 21 来阅读这本书的所有章节。在*第九章*中，你还需要 JDK 11 和 JDK 17。我建议使用 SDKMAN!这样的工具在你的电脑上安装和配置 SDK。如果你使用 Windows，你可以使用 JDK 安装程序。

我为所有示例使用了 Maven 作为依赖和构建系统。你可以选择在你的电脑上安装它，但书中创建的所有项目都使用了 Maven Wrapper，它会在需要时下载所有依赖项。

如果你是一名 Windows 用户，我推荐使用**Windows 子系统 Linux**（**WSL**），因为这本书中使用的某些辅助工具在 Linux 上可用，而且书中 GitHub 仓库中的脚本仅在 Linux 上进行了测试。实际上，我也是一名 Windows 用户，并使用 WSL 为这本书准备的所有示例。

我还推荐安装 Docker，因为它是运行这本书中集成的某些服务（如 PostgreSQL）的最简单方式。Docker 是运行由不同应用程序组成、在您的计算机上相互通信的分布式系统的最佳选择。此外，大多数集成测试都使用 Testcontainers，这需要 Docker。

我试图在本书中解释所有示例，而不需要特定的 IDE 要求。我主要使用 Visual Studio Code，因为它与 WSL 的集成非常出色，但您可以使用任何其他您偏好的 IDE，例如 IntelliJ 或 Eclipse。

| **本书中涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |
| OpenJDK 21 | Windows、macOS 或 Linux |
| OpenJDK 11 和 17 | Windows、macOS 或 Linux |
| Docker | Windows（推荐与 WSL 集成），macOS 或 Linux |
| Prometheus | 在 Docker（推荐）或 Windows、macOS 或 Linux 上原生运行 |
| Grafana | 在 Docker（推荐）或 Windows、macOS 或 Linux 上原生运行 |
| OpenZipkin | 在 Docker（推荐）或 Windows、macOS 或 Linux 上原生运行 Java |
| PostgreSQL | 在 Docker（推荐）或 Windows、macOS 或 Linux 上原生运行。 |
| MongoDB | 在 Docker（推荐）或 Windows、macOS 或 Linux 上原生运行 |
| Apache Cassandra | 在 Docker（推荐）或 Linux 上原生运行 |
| RabbitMQ | 在 Docker（推荐）或 Windows、macOS 或 Linux 上原生运行 |
| JMeter | Windows、macOS 或 Linux |
| 一种 IDE，如 Visual Studio Code/IntelliJ | Windows、macOS 或 Linux |

**如果您使用的是本书的数字版，我们建议您自己输入代码或通过 GitHub 仓库（下一节中提供链接）访问代码。这样做将帮助您避免与代码的复制和粘贴相关的任何潜在错误。**

## 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件：[`github.com/PacktPublishing/Spring-Boot-3.0-Cookbook`](https://github.com/PacktPublishing/Spring-Boot-3.0-Cookbook)。如果代码有更新，它将在现有的 GitHub 仓库中更新。一些食谱使用之前的食谱作为起点。在这些情况下，我在每个食谱的`start`子文件夹中提供一个工作版本，在`end`文件夹中提供一个完整版本。

我们还有来自我们丰富的图书和视频目录的其他代码包，可在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 获取。查看它们！

# 使用约定

本书使用了多种文本约定。

`文本中的代码`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账号。以下是一个示例：“此行为可以使用`@Transactional`注解的传播属性进行配置。”

代码块设置如下：

```java
em.getTransaction().begin();
// do your changes
em.getTransaction().commit();
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
public int getTeamCount() {
    return jdbcTemplate.queryForObject("SELECT COUNT(*) FROM teams", Integer.class);
}
```

任何命令行输入或输出都按以下方式编写：

```java
docker-compose -f docker-compose-redis.yml up
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“如果您现在运行应用程序，您将看到一个名为**match-events-topic.score.dlq**的新队列。”

小贴士或重要注意事项

显示如下。

# 部分

在本书中，您会发现一些频繁出现的标题（*准备工作*、*如何操作...*、*工作原理...*、*更多内容...*和*参见*）。

为了清楚地说明如何完成食谱，请按照以下方式使用这些部分。

## 准备工作

本节告诉您在食谱中可以期待什么，并描述了如何设置任何软件或任何为食谱所需的初步设置。

## 如何操作…

本节包含遵循食谱所需的步骤。

## 工作原理...

本节通常包含对前一个节段发生事件的详细解释。

## 更多内容...

本节包含有关食谱的附加信息，以便您对食谱有更深入的了解。

## 参见

本节提供了对食谱其他有用信息的链接。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请在邮件主题中提及书名，并通过 customercare@packtpub.com 将邮件发送给我们。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在本书中发现错误，我们将不胜感激，如果您能向我们报告，我们将非常感谢。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)，选择您的书籍，点击**勘误提交表单**链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式发现我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过版权@packtpub.com 与我们联系，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

一旦您阅读了*Spring Boot 3.0 烹饪书*，我们非常乐意听到您的想法！请[点击此处直接进入此书的亚马逊评论页面](https://packt.link/r/1835089496)并分享您的反馈。

您的评论对我们和科技社区都非常重要，这将帮助我们确保我们提供高质量的内容。

# 下载本书的免费 PDF 副本

感谢您购买本书！

您喜欢在路上阅读，但又无法携带您的印刷书籍到处走吗？

您的电子书购买是否与您选择的设备不兼容？

请放心，现在，每购买一本 Packt 图书，您都可以免费获得该书的 DRM 免费 PDF 版本。

在任何地方、任何设备上阅读。直接从您最喜欢的技术书籍中搜索、复制和粘贴代码到您的应用程序中。

优惠不仅限于此，您还可以获得独家折扣、时事通讯以及每天收件箱中的优质免费内容。

按照以下简单步骤获取这些好处：

1.  扫描二维码或访问以下链接

![](img/B21646_QR_Free_PDF.jpg)

[`packt.link/free-ebook/9781835089491`](https://packt.link/free-ebook/9781835089491)

1.  提交您的购买证明

1.  就这些！我们将直接将免费 PDF 和其他福利发送到您的邮箱

# 第一部分：Web 应用和微服务

在本部分，我们介绍了 RESTful API、分布式应用和 Spring Cloud 微服务的基础知识，以及跨功能特性，如安全和可观察性。

本部分包含以下章节：

+   *第一章*, *构建 RESTful API*

+   *第二章*, *使用 Oauth2 保护 Spring Boot 应用*

+   *第三章*, *可观察性、监控和应用管理*

+   *第四章*, *Spring Cloud*
