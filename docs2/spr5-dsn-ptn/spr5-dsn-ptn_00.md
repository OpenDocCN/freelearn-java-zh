# 前言

《Spring 5 设计模式》适合所有希望学习 Spring 用于企业应用程序的 Java 开发者。因此，企业 Java 开发者将特别发现它在理解 Spring 框架中使用的设计模式和它在企业应用程序中解决常见设计问题方面的有用性，并且他们将完全欣赏本书中提供的示例。在阅读本书之前，读者应具备 Core Java、JSP、Servlet 和 XML 的基本知识。

Spring 5 框架是由 Pivotal 新推出的，引入了响应式编程。Spring 5 引入了许多来自其先前版本的新特性和增强功能。我们将在书中讨论所有这些内容。“Spring 5 设计模式”将为您提供关于 Spring 框架的深入见解。

今天 Spring 框架的伟大之处在于，所有公司都已经将其作为企业应用程序开发的主要框架。对于 Spring 来说，无需外部企业服务器即可开始使用。

编写本书的目标是讨论 Spring 框架背后使用的所有设计模式以及它们如何在 Spring 框架中实现。在这里，作者还向您提供了一些在应用程序设计和开发中必须使用的最佳实践。

本书包含 12 章，涵盖了从基础知识到更复杂的设计模式（如响应式编程）的所有内容。

《Spring 5 设计模式》分为三个部分。第一部分向您介绍设计模式和 Spring 框架的基本知识。第二部分深入到前端之后，展示了 Spring 在应用程序后端中的位置。第三部分通过展示如何使用 Spring 构建 Web 应用程序并介绍 Spring 5 响应式编程的新特性来扩展这一点。这部分还展示了如何在企业应用程序中处理并发。

# 本书涵盖的内容

第一章，《Spring Framework 5.0 及设计模式入门》，概述了 Spring 5 框架及其所有新特性，包括一些 DI 和 AOP 的基本示例。您还将了解 Spring 优秀产品组合的概览。

第二章，《GOF 设计模式概览 - 核心设计模式》，概述了 GOF 设计模式家族的核心设计模式，包括一些适用于应用程序设计的最佳实践。您还将了解使用设计模式解决常见问题的概览。

第三章，《结构模式和行为的考虑》，概述了 GOF 设计模式家族的结构和行为设计模式，包括一些适用于应用程序设计的最佳实践。您还将了解使用设计模式解决常见问题的概览。

第四章, *使用依赖注入模式连接豆芽*，探讨了依赖注入模式以及应用程序中 Spring 配置的细节，展示了您应用程序中各种配置方式。这包括使用 XML、注解、Java 和混合配置。

第五章, *理解 Bean 生命周期和使用的模式*，概述了由 Spring 容器管理的 Spring Bean 生命周期，包括对 Spring 容器和 IoC 的理解。您还将了解 Spring Bean 生命周期回调处理程序和后处理器。

第六章, *使用代理和装饰器模式进行 Spring 面向切面编程*，探讨了如何使用 Spring AOP 将横切关注点从它们所服务的对象中解耦。本章还为后续章节奠定了基础，在这些章节中，您将使用 AOP 提供声明式服务，例如事务、安全和缓存。

第七章, *使用 Spring 和 JDBC 模板模式访问数据库*，探讨了如何使用 Spring 和 JDBC 访问数据；在这里，您将看到如何使用 Spring 的 JDBC 抽象和 JDBC 模板以比原生 JDBC 更简单的方式查询关系数据库。

第八章, *使用 Spring ORM 和事务实现模式访问数据库*，展示了 Spring 如何与 ORM 框架集成，例如 Hibernate 和其他 Java 持久化 API（JPA）的实现，以及 Spring 事务管理。此外，它还包含了 Spring Data JPA 提供的即时查询生成魔法。

第九章, *使用缓存模式提高应用程序性能*，展示了如何通过避免使用数据库（如果所需数据 readily available）来提高应用程序性能。因此，我将向您展示 Spring 如何提供对缓存数据的支持。

第十章, *使用 Spring 在 Web 应用程序中实现 MVC 模式*，简要概述了使用 Spring MVC 开发 Web 应用程序。您将学习 MVC 模式、前端控制器模式、Dispatcher Servlet 以及基于 Spring 框架原则的 Spring MVC 基础知识。您将了解如何编写控制器来处理 Web 请求，并看到如何透明地将请求参数和有效负载绑定到您的业务对象，同时提供验证和错误处理。本章还简要介绍了 Spring MVC 中的视图和视图解析器。

第十一章，*实现响应式设计模式*，探讨了响应式编程模型，即使用异步数据流进行编程。您将看到如何在 Spring Web 模块中实现响应式系统。

第十二章，*实现并发模式*，更深入地探讨了在 Web 服务器内部处理多个连接时的并发性。正如我们在我们的架构模型中所概述的，请求处理与应用程序逻辑解耦。

# 您需要这本书什么

这本书可以在没有电脑或笔记本电脑的情况下阅读，在这种情况下，您需要的只是这本书本身。尽管为了跟随书中的示例，您需要 Java 8，您可以从[`www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html`](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下载。您还需要您喜欢的 IDE 来处理示例，但我使用了软件 Spring Tool Suite；根据您的系统操作系统，从[`spring.io/tools/sts/all`](https://spring.io/tools/sts/all)下载 Spring Tool Suite（STS）的最新版本。Java 8 和 STS 在多种平台上运行——Windows、macOS 和 Linux。

# 这本书面向谁

Spring 5 设计模式是为所有希望学习 Spring 用于企业应用的 Java 开发者而设计的。因此，企业 Java 开发者会发现它在理解 Spring 框架中使用的设计模式以及它如何解决企业应用中的常见设计问题特别有用，并且他们将完全欣赏本书中提供的示例。在阅读此书之前，读者应具备 Core Java、JSP、Servlet 和 XML 的基本知识。

# 规范

在这本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL 和用户输入如下所示：“在我们的代码中，我们有一个`TransferServiceImpl`类，其构造函数接受两个参数：”

代码块设置如下：

```java
    public class JdbcTransferRepository implements TransferRepository{ 
      JdbcTemplate jdbcTemplate; 
      public setDataSource(DataSource dataSource) { 
        this.jdbcTemplate = new JdbcTemplate(dataSource); 
    } 
     // ... 
   } 
```

**新术语**和**重要词汇**以粗体显示。

警告或重要注意事项如下所示。

小技巧和技巧看起来像这样。

# 读者反馈

我们欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或不喜欢什么。读者的反馈对我们来说非常重要，因为它帮助我们开发出您真正能从中受益的标题。

要向我们发送一般反馈，只需发送电子邮件至 feedback@packtpub.com，并在邮件主题中提及书的标题。

如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

# 下载示例代码

您可以从您的账户中下载本书的示例代码文件。[`www.packtpub.com`](http://www.packtpub.com)。如果您在其他地方购买了此书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的“支持”标签上。

1.  点击“代码下载与勘误表”。

1.  在搜索框中输入书籍名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书的来源。

1.  点击“代码下载”。

文件下载完成后，请确保您使用最新版本解压缩或提取文件夹：

+   Windows 下的 WinRAR / 7-Zip

+   Mac 下的 Zipeg / iZip / UnRarX

+   Linux 下的 7-Zip / PeaZip

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Spring5-Design-Patterns`](https://github.com/PacktPublishing/Spring5-Design-Patterns)。我们还有其他丰富的图书和视频代码包可供选择，网址为[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。请查看它们！

# 勘误表

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现了错误——可能是文本或代码中的错误——如果您能向我们报告，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击“勘误提交表单”链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误表中的现有勘误列表。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在“勘误”部分。

# 侵权

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 copyright@packtpub.com 与我们联系，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面的帮助。

# 问题

如果您在这本书的任何方面遇到问题，您可以联系我们的 questions@packtpub.com，我们将尽力解决问题。
