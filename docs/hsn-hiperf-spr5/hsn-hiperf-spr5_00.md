# 前言

这本书的使命是向开发人员介绍应用程序监控和性能调优，以创建高性能的应用程序。该书从 Spring Framework 的基本细节开始，包括各种 Spring 模块和项目、Spring bean 和 BeanFactory 实现，以及面向方面的编程。它还探讨了 Spring Framework 作为 IoC bean 容器。我们将讨论 Spring MVC，这是一个常用的 Spring 模块，用于详细构建用户界面，包括 Spring Security 身份验证部分和无状态 API。这本书还强调了构建与关系数据库交互的优化 Spring 应用程序的重要性。然后，我们将通过一些高级的访问数据库的方式，使用对象关系映射（ORM）框架，如 Hibernate。该书继续介绍了 Spring Boot 和反应式编程等新 Spring 功能的细节，并提出了最佳实践建议。该书的一个重要方面是它专注于构建高性能的应用程序。该书的后半部分包括应用程序监控、性能优化、JVM 内部和垃圾收集优化的细节。最后，解释了如何构建微服务，以帮助您了解在该过程中面临的挑战以及如何监视其性能。

# 第五章《理解 Spring 数据库交互》帮助我们了解 Spring Framework 与数据库交互。然后介绍了 Spring 事务管理和最佳连接池配置。最后，介绍了数据库设计的最佳实践。

这本书适合想要构建高性能应用程序并在生产和开发中更多地控制应用程序性能的 Spring 开发人员。这本书要求开发人员对 Java、Maven 和 Eclipse 有一定的了解。

# 这本书涵盖了什么

第一章《探索 Spring 概念》侧重于清晰理解 Spring Framework 的核心特性。它简要概述了 Spring 模块，并探讨了不同 Spring 项目的集成，并清晰解释了 Spring IoC 容器。最后，介绍了 Spring 5.0 的新功能。

第二章《Spring 最佳实践和 Bean 装配配置》探讨了使用 Java、XML 和注解进行不同的 bean 装配配置。该章还帮助我们了解在 bean 装配配置方面的不同最佳实践。它还帮助我们了解不同配置的性能评估，以及依赖注入的陷阱。

第三章《调优面向方面的编程》探讨了 Spring 面向方面的编程（AOP）模块及其各种术语的概念。它还涵盖了代理的概念。最后，介绍了使用 Spring AOP 模块实现质量和性能的最佳实践。

第四章《Spring MVC 优化》首先清楚地介绍了 Spring MVC 模块以及不同的 Spring MVC 配置方法。它还涵盖了 Spring 中的异步处理概念。然后解释了 Spring Security 配置和无状态 API 的身份验证部分。最后，介绍了 Tomcat 与 JMX 的监控部分，以及 Spring MVC 性能改进。

这本书适合谁

第六章《Hibernate 性能调优和缓存》描述了使用 ORM 框架（如 Hibernate）访问数据库的一些高级方式。最后，解释了如何使用 Spring Data 消除实现数据访问对象（DAO）接口的样板代码。

第七章，*优化 Spring 消息传递*，首先探讨了 Spring 消息传递的概念及其优势。然后详细介绍了在 Spring 应用程序中使用 RabbitMQ 进行消息传递的配置。最后，描述了提高性能和可伸缩性以最大化吞吐量的参数。

第八章，*多线程和并发编程*，介绍了 Java 线程的核心概念和高级线程支持。还介绍了 Java 线程池的概念以提高性能。最后，将探讨使用线程进行 Spring 事务管理以及编程线程的各种最佳实践。

第九章，*性能分析和日志记录*，专注于性能分析和日志记录的概念。本章首先定义了性能分析和日志记录以及它们如何有助于评估应用程序的性能。在本章的后半部分，重点将是学习可以用于研究应用程序性能的软件工具。

第十章，*应用性能优化*，专注于优化应用程序性能。还介绍了识别性能问题症状、性能调优生命周期和 Spring 中的 JMX 支持的详细信息。

第十一章，*JVM 内部*，介绍了 JVM 的内部结构和调优 JVM 以实现高性能的内容。还涵盖了与内存泄漏相关的主题以及与垃圾回收相关的常见误解，然后讨论了不同的垃圾回收方法及其重要性。

第十二章，*Spring Boot 微服务性能调优*，介绍了 Spring Boot 微服务及其性能调优的概念。还清楚地描述了如何使用执行器和健康检查来监视 Spring Boot 应用程序。还涵盖了不同的技术，以调优 Spring Boot 应用程序的性能。

# 充分利用本书

本书要求开发人员对 Java、Maven 和 Eclipse 有一定的了解。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，文件将直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  登录或注册[www.packtpub.com](http://www.packtpub.com/support)。

1.  选择“支持”选项卡。

1.  单击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

下载文件后，请确保使用最新版本的解压缩软件解压文件夹：

+   WinRAR/7-Zip 适用于 Windows

+   Zipeg/iZip/UnRarX 适用于 Mac

+   7-Zip/PeaZip 适用于 Linux

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Hands-On-High-Performance-with-Spring-5`](https://github.com/PacktPublishing/Hands-On-High-Performance-with-Spring-5)。如果代码有更新，将在现有的 GitHub 存储库中更新。

我们还有其他代码包，来自我们丰富的图书和视频目录，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。快去看看吧！

# 下载彩色图片

我们还提供了一份 PDF 文件，其中包含本书中使用的截图/图表的彩色图片。您可以从[`www.packtpub.com/sites/default/files/downloads/HandsOnHighPerformancewithSpring5_ColorImages.pdf.`](https://www.packtpub.com/sites/default/files/downloads/HandsOnHighPerformancewithSpring5_ColorImages.pdf)下载。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码单词，数据库表名，文件夹名，文件名，文件扩展名，路径名，虚拟 URL，用户输入和 Twitter 句柄。这是一个例子：“为了避免`LazyInitializationException`，解决方案之一是在视图中打开会话。”

一块代码设置如下：

```java
PreparedStatement st = null;
try {
    st = conn.prepareStatement(INSERT_ACCOUNT_QUERY);
    st.setString(1, bean.getAccountName());
    st.setInt(2, bean.getAccountNumber());
    st.execute();
}
```

当我们希望引起您对代码块的特定部分的注意时，相关的行或项目以粗体显示：

```java
@Configuration
@EnableTransactionManagement
@PropertySource({ "classpath:persistence-hibernate.properties" })
@ComponentScan({ "com.packt.springhighperformance.ch6.bankingapp" })
    @EnableJpaRepositories(basePackages = "com.packt.springhighperformance.ch6.bankingapp.repository")
public class PersistenceJPAConfig {

}
```

任何命令行输入或输出都是这样写的：

```java
curl -sL --connect-timeout 1 -i http://localhost:8080/authentication-cache/secure/login -H "Authorization: Basic Y3VzdDAwMTpUZXN0QDEyMw=="
```

**粗体**：表示新术语，重要单词，或者您在屏幕上看到的单词。例如，菜单或对话框中的单词会以这种方式出现在文本中。这是一个例子：“在应用程序窗口内，我们可以看到一个用于本地节点的菜单。”

警告或重要提示看起来像这样。

提示和技巧看起来像这样。
