# 前言

Spring 是一个开源的 Java 应用程序开发框架，用于构建和部署在 JVM 上运行的系统和应用程序。它通过使用模型-视图-控制器范式和依赖注入，使得构建模块化和可测试的 Web 应用程序变得更加高效。它与许多框架（如 Hibernate、MyBatis、Jersey 等）无缝集成，并在使用标准技术（如 JDBC、JPA 和 JMS）时减少样板代码。

本书的目的是教会中级 Spring 开发人员掌握使用高级概念和额外模块来扩展核心框架，从而进行 Java 应用程序开发。这样可以开发更高级、更强大的集成应用程序。

# 本书涵盖的内容

第一章，“Spring 与 Mongo 集成”，演示了 Spring MVC 与 MongoDB 的集成，以及安装 MongoDB 来创建数据库和集合。

第二章，“使用 Spring JMS 进行消息传递”，教你安装 Apache ActiveMQ 和不同类型的消息传递。本章还演示了创建多个队列，并使用 Spring 模板与这些队列进行通信，同时提供了屏幕截图的帮助。

第三章，“使用 Spring Mail 进行邮件发送”，创建了一个邮件服务，并使用 Spring API 进行配置，演示了如何使用 MIME 消息发送带附件的邮件。

第四章，“使用 Spring Batch 进行作业”，说明了如何使用 Spring Batch 读取 XML 文件，以及如何创建基于 Spring 的批处理应用程序来读取 CSV 文件。本章还演示了如何使用 Spring Batch 编写简单的测试用例。

第五章，“Spring 与 FTP 集成”，概述了不同类型的适配器，如入站和出站适配器，以及出站网关及其配置。本章还研究了两个重要的类，FTPSessionFactory 和 FTPsSessionFactory，使用 getter 和 setter。

第六章，“Spring 与 HTTP 集成”，介绍了使用多值映射来填充请求并将映射放入 HTTP 标头的用法。此外，它还提供了关于 HTTP 和 Spring 集成支持的信息，可用于访问 HTTP 方法和请求。

第七章，“Spring 与 Hadoop”，展示了 Spring 如何与 Apache Hadoop 集成，并提供 Map 和 Reduce 过程来搜索和计算数据。本章还讨论了在 Unix 机器上安装 Hadoop 实例以及在 Spring 框架中配置 Hadoop 作业。

第八章，“Spring 与 OSGI”，开发了一个简单的 OSGI 应用程序，并演示了 Spring 动态模块如何支持 OSGI 开发，并减少文件的创建，从而使配置变得更加简单。

第九章，“使用 Spring Boot 引导应用程序”，从设置一个简单的 Spring Boot 项目开始，以及使用 Spring Boot 引导应用程序的过程。本章还介绍了 Spring Boot 如何支持云铁路服务器，并帮助在云上部署应用程序。

第十章，“Spring 缓存”，实现了我们自己的缓存算法，并教你制作一个通用算法。本章还讨论了在 Spring 框架中支持缓存机制的类和接口。

第十一章, *Spring 与 Thymeleaf 集成*，将 Thymeleaf 模板引擎集成到 Spring MVC 应用程序中，并使用 Spring Boot 启动 Spring 与 Thymeleaf 应用程序。

第十二章, *Spring 与 Web 服务集成*，将 JAX_WS 与 Spring Web 服务集成。它演示了如何创建 Spring Web 服务和端点类，通过访问 WSDL URL 来访问 Web 服务。

# 你需要什么来阅读这本书

需要一台安装有 Mac OS、Ubuntu 或 Windows 的计算机。为了构建 Spring 应用程序，你至少需要安装 Java 和 Maven 3。

# 这本书适合谁

如果你是一名有 Spring 应用开发经验的 Java 开发者，那么这本书非常适合你。建议具备良好的 Spring 编程约定和依赖注入的知识，以充分利用本书。

# 约定

在本书中，你会发现许多文本样式，用于区分不同类型的信息。以下是一些样式的示例及其含义的解释。

文本中的代码、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名都显示如下：“我们使用`@Controller`注解来表示`ProductController.java`类是一个控制器类。”

一块代码设置如下：

```java
@Controller
public class ProductController {
  @Autowired
  private ProductRepository respository;
  private List <Product>productList;
  public ProductController() {
    super();
  }
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项将以粗体显示：

```java
public class MailAdvice {
  public void advice (final ProceedingJoinPoint proceedingJoinPoint) {
    new Thread(new Runnable() {
    public void run() {
```

任何命令行的输入或输出都是这样写的：

```java
cd E:\MONGODB\mongo\bin
mongod -dbpath e:\mongodata\db

```

**新术语**和**重要单词**以粗体显示。屏幕上看到的单词，例如菜单或对话框中的单词，会以这种方式出现在文本中：“下一步是创建一个 rest 控制器来发送邮件；为此，请单击**提交**。”

### 注意

警告或重要提示会以这样的方式出现在一个框中。

### 提示

技巧和窍门是这样出现的。
