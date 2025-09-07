# 前言

《精通 Spring Boot 3.0》深入探讨了 Spring Boot 3.0，重点关注其高级特性。这项技术被视为渴望构建复杂和可扩展后端系统的 Java 开发者的必备技术。引言为全面指南奠定了基础，强调了 Spring Boot 3.0 在现代软件开发中的实用性。

# 这本书面向的对象

如果你是一名渴望提升技能的 Java 开发者，那么《精通 Spring Boot 3.0》这本书适合你。对于想要通过高级 Spring Boot 特性构建强大后端系统的微服务架构师、DevOps 工程师和技术负责人，这本书也将非常有用。对微服务架构的基础理解以及一些 RESTful API 的经验将帮助你最大限度地利用这本书。

# 这本书涵盖的内容

*第一章*, *高级 Spring Boot 概念简介*, 介绍了 Spring Boot 3.0 的高级特性，为后续章节提供了基础。

*第二章*, *微服务中的关键架构模式 – DDD、CQRS 和事件溯源*, 探索了 DDD、CQRS 和事件溯源等基本架构模式，提供了理论知识和实践示例。

*第三章*, *反应式 REST 开发和异步系统*, 涵盖了 Spring Boot 中的反应式编程以及异步系统的实现细节。

*第四章*, *Spring Data：SQL、NoSQL、缓存抽象和批处理*, 讨论了使用 Spring Data 管理数据，包括 SQL 和 NoSQL 数据库，并介绍了缓存抽象和批处理。

*第五章*, *保护您的 Spring Boot 应用程序*, 综合探讨了使用 OAuth2、JWT 和 Spring Security 过滤器保护 Spring Boot 应用程序的方法。

*第六章*, *高级测试策略*, 深入探讨了 Spring Boot 应用程序中的测试策略，重点关注单元测试、集成测试和安全测试技术。

*第七章*, *Spring Boot 3.0 的容器化和编排特性*, 专注于 Spring Boot 3.0 的容器化和编排特性，包括 Docker 和 Kubernetes 的集成。

*第八章*, *使用 Kafka 探索事件驱动系统*, 探讨了 Kafka 与 Spring Boot 的集成，用于构建事件驱动系统，并包括监控和故障排除技巧。

*第九章*, *提高生产力和开发简化*, 通过诸如面向方面编程和自定义 Spring Boot 启动器等工具和技术简化开发过程。

# 为了最大限度地利用这本书

在阅读本书之前，您需要具备 Java 17 的基本知识。应在您的计算机上安装**Java 开发工具包**（**JDK 17**）。所有代码示例都已在 macOS 上使用 JDK 17 进行了测试。然而，它们应该在其他操作系统上也能工作。

| **本书涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |
| JDK 17 | Windows, macOS, 或 Linux |
| Gradle 8.7 |  |
| Docker Desktop |  |
| IDE | IntelliJ Community Edition, Eclipse |

**如果您正在使用本书的数字版，我们建议您亲自输入代码或从本书的 GitHub 仓库（下一节中提供链接）获取代码。这样做将帮助您避免与代码的复制和粘贴相关的任何潜在错误。**

# 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件 [`github.com/PacktPublishing/Mastering-Spring-Boot-3.0`](https://github.com/PacktPublishing/Mastering-Spring-Boot-3.0)。如果代码有更新，它将在 GitHub 仓库中更新。

我们还有其他丰富的图书和视频资源中的代码包可供在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 获取。查看它们吧！

# 使用的约定

本书使用了多种文本约定。

`文本中的代码`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“然而，在响应式世界中，我们使用`ReactiveCrudRepository`或`R2dbcRepository`。”

代码块设置如下：

```java
# Enable H2 Console
spring.h2.console.enabled=true
# Database Configuration for H2
spring.r2dbc.url=r2dbc:h2:mem:///testdb
spring.r2dbc.username=sa
spring.r2dbc.password=
# Schema Generation
spring.sql.init.mode=always
spring.sql.init.platform=h2
```

任何命令行输入或输出都应如下编写：

```java
./gradlew bootRun
```

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词以**粗体**显示。以下是一个示例：“一旦您完成所有选择，请点击**生成**按钮以获取可构建的项目。”

小贴士或重要注意事项

看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请通过电子邮件发送至 customercare@packtpub.com，并在邮件主题中提及本书标题。

**勘误表**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在本书中发现错误，我们将不胜感激，如果您能向我们报告这一点。请访问 [www.packtpub.com/support/errata](http://www.packtpub.com/support/errata) 并填写表格。

**盗版**：如果您在互联网上以任何形式发现我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过电子邮件发送至 copyright@packt.com 并附上材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)

# 分享您的想法

一旦您阅读了《Mastering Spring Boot 3.0》，我们很乐意听到您的想法！请[点击此处直接转到该书的亚马逊评论页面](https://packt.link/r/1803230789)并分享您的反馈。

您的评论对我们和科技社区非常重要，并将帮助我们确保我们提供高质量的内容。

# 下载此书的免费 PDF 副本

感谢您购买此书！

您喜欢在路上阅读，但无法携带您的印刷书籍到处走？

您的电子书购买是否与您选择的设备不兼容？

别担心！现在，每本 Packt 书籍都免费提供该书的 DRM 免费 PDF 版本。

在任何地方、任何设备上阅读。从您最喜欢的技术书籍中搜索、复制和粘贴代码到您的应用程序中。

优惠不止于此，您还可以获得独家折扣、时事通讯和每天收件箱中的优质免费内容

按照以下简单步骤获取好处：

1.  扫描二维码或访问以下链接：

![](img/B18400_QR_Free_PDF.jpg)

[`packt.link/free-ebook/9781803230788`](https://packt.link/free-ebook/9781803230788)

1.  提交您的购买证明。

1.  就这些！我们将直接将您的免费 PDF 和其他好处发送到您的电子邮件。

# 第一部分：建筑基础

在本部分中，我们将深入探讨高级 Spring Boot 概念的复杂世界。本部分为更深入地理解如何利用 Spring Boot 构建强大和可扩展的应用程序奠定了基础。

本部分包含以下章节：

+   *第一章*，*高级 Spring Boot 概念简介*
