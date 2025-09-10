# 前言

WildFly，作为最新的 JBoss 应用程序服务器版本，为开发者提供了 Java EE 7 平台的完整实现。它建立在 JBoss AS 7 引入的模块化架构的坚实基础之上，但在灵活性和性能方面有所提升。最新版本的 Java 企业版专注于提高开发者的生产力，WildFly 也是如此。

本书将向 Java 开发者介绍企业应用程序的世界。我们将使用在现代实际项目中经过实战检验的现代开发技术和工具。我们还将利用 WildFly 平台提供的功能，如安全、缓存、测试和集群。最后，您将学习如何使用为 WildFly 专门创建的工具来管理您的服务器。

学习过程将集中在票务预订应用程序上，这是一个示例项目，每个章节都会增加更多功能（有时是完全不同的用户界面）。

# 本书涵盖的内容

第一章, *开始使用 WildFly*，是 Java EE 平台和新的 Java EE 7 版本规范的介绍。它还侧重于介绍 WildFly 的新特性、开发者环境设置和基本服务器管理。

第二章, *在 WildFly 上创建您的第一个 Java EE 应用程序*，描述了 WildFly 服务器使用的基础知识，提供了部署您第一个应用程序所需的信息。

第三章, *介绍 Java EE 7 – EJBs*，介绍了 Java EE 中的企业对象，称为企业 Java Bean。在本章中，我们为票务预订应用程序奠定了基础。

第四章, *学习上下文和依赖注入*，涵盖了连接您应用程序构建块的 CDI 技术。

第五章, *将持久性与 CDI 结合*，探讨了数据库世界和 Java EE 中的对象映射。

第六章, *使用 JBoss JMS Provider 开发应用程序*，深入探讨了使用 JCA 的企业系统集成和 HornetQ。

第七章, *将 Web 服务添加到您的应用程序中*，不仅讨论了旧式的 SOAP Web 服务，还介绍了基于 JAX-RS（REST）的现代且流行的方法。我们还将探讨如何将 Java EE 7 后端与 AngularJS 浏览器应用程序集成。

第八章，“添加 WebSocket”，介绍了 Java EE 7 平台的一个全新的功能：WebSocket。我们将在我们的 AngularJS 示例中查看它们。

第九章，“管理应用程序服务器”，讨论了 WildFly 的管理功能。

第十章，“保护 WildFly 应用程序”，专注于服务器和你的应用程序的安全相关方面。

第十一章，“集群 WildFly 应用程序”，讨论了使 Java EE 应用程序具有高可用性和可扩展性的方法。

第十二章，“长期任务执行”，描述了企业 Java 批处理应用程序和服务器上的并发管理的新领域。

第十三章，“测试你的应用程序”，展示了在介绍最重要的 Java EE 技术之后，我们如何使用 Arquillian 为我们的应用程序编写集成测试。

附录，“使用 JBoss Forge 快速开发”，涵盖了 JBoss Forge 工具。它展示了如何使用这个应用程序通过其代码生成功能来加速基于 Java EE 的项目开发。

# 你需要这本书的内容

本书是面向 Java EE 和 WildFly 世界的代码导向指南。为了充分利用本书，你需要访问互联网以下载所有必需的工具和库。需要具备 Java 知识。我们还将使用 Maven 作为构建自动化工具，但由于其冗长性，本书中提供的示例都是自解释的。

尽管我们将使用 AngularJS，但不需要额外的 JS 知识。本书中所有框架的示例都将基于展示，其目的是展示在典型场景中不同方如何与 Java EE 交互。

# 这本书面向的对象

如果你是一名想要学习 Java EE 基础知识的 Java 开发者，或者你已经是一名 Java EE 开发者，想要了解 WildFly 或 Java EE 7 的新特性，这本书就是为你准备的。本书涵盖了所述技术的基础知识，并确保为那些已有一定知识的人提供一些更有趣、更高级的主题。

# 习惯用法

在这本书中，你会找到多种文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名如下所示：“在 MDB 实例的`onMessage()`方法返回后，请求完成，实例被放回空闲池中。”

代码块设置如下：

```java
<jms-destinations>
   <jms-queue name="TicketQueue">
      <entry name="java:jboss/jms/queue/ticketQueue"/>
         <durable>false</durable>
   </jms-queue>
</jms-destinations>
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
@Stateless
public class SampleEJB {

 @Resource(mappedName = "java:/ConnectionFactory")
 private ConnectionFactory cf; 
}
```

任何命令行输入或输出都应如下编写：

```java
CREATE DATABASE ticketsystem;
CREATE USER jboss WITH PASSWORD 'jboss';
GRANT ALL PRIVILEGES ON DATABASE ticketsystem TO jboss;

```

**新术语**和**重要词汇**以粗体显示。您在屏幕上、菜单或对话框中看到的单词，例如，在文本中如下所示：“例如，Eclipse 的**文件**菜单包括一个**从表创建 JPA 实体**的选项，一旦设置了与数据库的连接，就可以将您的 DB 模式（或其部分）反转成 Java 实体。”

### 注意

警告或重要注意事项以如下框的形式出现。

### 提示

技巧和窍门看起来像这样。

# 读者反馈

我们欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者反馈对我们开发您真正能从中获得最大价值的标题非常重要。

要向我们发送一般反馈，只需发送电子邮件到`<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

## 客户支持

现在您是 Packt 书籍的骄傲所有者，我们有多种方法可以帮助您从您的购买中获得最大价值。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户中下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 勘误

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)来报告它们，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误表，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将显示在**勘误**部分。

## 盗版

在互联网上，版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

如果您发现了疑似盗版材料，请通过`<copyright@packtpub.com>`联系我们，并提供链接。

我们感谢您在保护我们作者和我们提供有价值内容的能力方面的帮助。

## 问题和建议

如果您在书籍的任何方面遇到问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决。
