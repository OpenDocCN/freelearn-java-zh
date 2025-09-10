# 前言

欢迎使用 *Apache Karaf 烹饪书*. 这本烹饪书是为了提供您在 Apache Karaf 实战中积累的最佳实践和经验教训而创建的。

本书中的食谱不应按顺序阅读。读者应根据需要挑选食谱。我们已经尽最大努力确保可以在适用的情况下参考基础知识。然而，我们确实期望我们的读者对 OSGi、Karaf 以及每个食谱关注的包有一定的背景知识。需要更多关于 Karaf 和 OSGi 背景信息的读者可能需要查找以下书籍：

+   *学习 Apache Karaf*，*Johan Edstrom, Jamie Goodyear, 和 Heath Kesler*，*Packt Publishing*

+   *Instant OSGi Starter*，*Johan Edstrom 和 Jamie Goodyear*，*Packt Publishing*

# 本书涵盖的内容

第一章, *为系统构建者准备的 Apache Karaf*，涵盖了如何使 Apache Karaf 更适用于生产的食谱。探讨了改进日志记录、自定义命令、品牌化、管理和高可用性等主题。

第二章, *使用 Apache Camel 构建智能路由器*，介绍了 Apache Camel 命令，然后讨论了如何使用 Plain Old Java Objects (POJO)、Blueprint、带有配置管理支持的 Blueprint 以及最终使用托管服务工厂来构建 Camel 路由器的食谱。

第三章, *使用 Apache ActiveMQ 部署消息代理*，探讨了如何在嵌入式方式下使用 Apache ActiveMQ，并介绍了用于监控和与嵌入式 ActiveMQ 代理交互的不同命令。

第四章, *使用 Pax Web 托管 Web 服务器*，解释了如何配置和使用 Apache Karaf 与 Pax Web。它从基本的 Http 服务开始，到完整的 Web 应用支持。

第五章, *使用 Apache CXF 托管 Web 服务*，展示了如何在 Karaf 中设置 Apache CXF 端点，以支持 RESTful 和基于 WSDL 的 Web 服务。

第六章, *使用 Apache Karaf Cellar 分发集群容器*，解释了 Cellar 的设置并介绍了其命令的使用。

第七章, *使用 Apache Aries 和 OpenJPA 提供持久层*，探讨了如何将 Java 持久性和事务支持添加到您的 OSGi 环境中。

第八章，*使用 Apache Cassandra 提供大数据集成层*，展示了如何将 Cassandra 客户端包安装到 Karaf 中，设置建模数据，并构建项目以利用由 Cassandra 支持的持久化层。

第九章，*使用 Apache Hadoop 提供大数据集成层*，向您展示了如何将所有 Hadoop 依赖项集成到功能文件中，在管理资源的同时正确部署到 Karaf 中，并使用 HDFS 存储和检索数据。

第十章，*使用 Pax Exam 测试 Apache Karaf*，解释了如何使用 OSGi 进行集成测试，并展示了如何在集成测试中使用 Apache Karaf。

# 您需要为此书准备的内容

作者们非常注意将探索 Apache Karaf 所需的软件数量保持在最低。在尝试本书中包含的食谱之前，您需要获取以下内容：

+   Apache Karaf 3.0

+   Oracle Java SDK 1.7

+   Apache Maven 3.0

+   Git 客户端

+   任何文本编辑器

# 本书面向对象

*Apache Karaf 食谱集*是为寻求在开发和部署应用程序时应用最佳实践的开发商和系统管理员而收集的食谱集。读者将发现许多与面向服务的架构和大数据管理相关的食谱，以及需要一些 OSGi 经验才能充分利用其潜力的几个库。

# 约定

在本书中，您将找到多种文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称如下所示：“我们将`jmxRole`附加到管理员组。”

代码块设置如下：

```java
<resource>
 <directory>
  ${project.basedir}/src/main/resources
 </directory>
 <filtering>true</filtering>
 <includes>
  <include>**/*</include>
 </includes>
</resource>
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
log4j.appender.out.append=true
log4j.appender.out.maxFileSize=10MB
log4j.appender.out.maxBackupIndex=100
```

任何命令行输入或输出都应如下编写：

```java
karaf@root()> feature:repo-add mvn:com.packt/features-file/1.0.0-SNAPSHOT/xml/features

```

**新术语**和**重要词汇**将以粗体显示。屏幕上显示的词汇，例如在菜单或对话框中，在文本中如下显示：“**QueueSize**列显示当前队列中等待消费的消息数量。”

### 注意

警告或重要提示将以如下框中显示。

### 小贴士

小技巧和窍门如下所示。

# 读者反馈

我们欢迎读者的反馈。请告诉我们您对本书的看法——您喜欢什么或可能不喜欢什么。读者反馈对我们开发您真正能从中获得最大价值的标题非常重要。

要向我们发送一般反馈，只需发送电子邮件到`<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 错误清单

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误清单，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**错误提交表单**链接，并输入您的错误详细信息来报告它们。一旦您的错误得到验证，您的提交将被接受，错误将被上传到我们的网站，或添加到该标题的现有错误清单中，在“错误清单”部分下。您可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的标题来查看任何现有错误。

## 盗版

在互联网上盗版版权材料是一个跨所有媒体持续存在的问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以追究补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面的帮助。

## 问题

如果您在本书的任何方面遇到问题，可以通过`<questions@packtpub.com>`与我们联系，我们将尽力解决。
