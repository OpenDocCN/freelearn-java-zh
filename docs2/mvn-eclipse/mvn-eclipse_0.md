# 前言

*Maven for Eclipse* 是一本不可或缺的指南，帮助您使用 m2eclipse 插件在 Eclipse IDE 中理解和使用 Maven。它绝对不是一本深入和全面的资源。相反，它是一本关于 Maven 项目开发的快速便捷指南。它从 Apache Maven 的基础开始；涵盖核心概念；并展示如何在 Eclipse IDE 中使用 m2eclipse 插件创建、导入、构建、运行、打包和自定义，以生成 Maven 项目的项目工件。

# 本书涵盖内容

第一章，*Apache Maven – 简介和安装*，为用户提供了一个关于 Apache Maven 的快速介绍和安装参考。到本章结束时，用户将能够在他们的系统上运行一个 Maven 项目。

第二章，*安装 m2eclipse*，为用户提供安装 m2eclipse 插件的参考，并提供了 Eclipse 的 Maven 集成。到本章结束时，用户将在他们的系统上安装好 m2eclipse 并准备好使用。

第三章，*创建和导入项目*，从 Maven 项目结构开始，介绍核心方面和概念，并指导您使用 m2eclipse 插件创建和导入 Maven 项目。到本章结束时，用户将熟悉 Maven 项目结构的核心概念，并且能够创建和导入 Maven 项目。

第四章，*构建和运行项目*，向用户介绍不同的构建生命周期，并教授他们如何查看 m2eclipse 控制台以及如何构建和运行项目。到本章结束时，用户将熟悉构建生命周期，并且能够熟练使用 m2eclipse 构建和运行项目。

第五章，*调味 Maven 项目*，教授用户如何创建一个简单的 Web 应用程序，展示如何自定义它，并提供如何编写和运行单元测试的指南。到本章结束时，用户将学会使用 m2eclipse 创建 Web 应用程序，并学习如何更改 POM 文件以针对单元测试生成报告。

第六章，*创建多模块项目*，旨在介绍多模块项目的概念，并教授用户如何创建、构建和运行项目。到本章结束时，用户将了解如何使用 m2eclipse 插件创建和运行多模块 Maven 项目。

第七章，*探索 m2eclipse*，深入探讨了 m2eclipse 插件，并介绍了不同的功能和方面，使生活更加便捷。到本章结束时，用户将熟悉 m2eclipse 的每个方面，并能高效且轻松地使用它。

# 您需要为这本书准备什么

建议您在开发期间使用以下配置的笔记本电脑或台式机以获得最佳性能：

+   4 GB RAM

+   Windows OS 7 / Ubuntu 12.04 / Mac OS Maverick

+   双核 / iSeries 处理器

+   互联网连接

# 本书面向对象

本书面向初学者和现有开发者，他们想学习如何为 Java 项目使用 Maven。假设您有 Java 编程经验，并且您已经使用 IDE 进行开发。

# 习惯用法

在本书中，您将找到许多不同风格的文本，以区分不同类型的信息。以下是一些这些风格的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称将如下所示："可以在`pom`文件中声明性地包含插件和目标，以自定义项目的执行。"

代码块将如下设置：

```java
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.packt.mvneclipse</groupId>
    <artifactId>mvneclipse</artifactId>
    <version>1.2</version>
</project>
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
<!--General project Information -->
 <modelVersion>4.0.0</modelVersion>
 <groupId>com.packt.mvneclipse</groupId>
 <artifactId>hello-project</artifactId>
 <version>0.0.1-SNAPSHOT</version>
 <name>hello-project</name>
 <url>http://maven.apache.org</url>
 <properties>1
 <project.build.sourceEncoding>UTF8</project.build.sourceEncoding>
</properties>

<repositories>
  <repository>
    <snapshots>
      <enabled>false</enabled>
    </snapshots>
    <id>central</id>
    <name>Maven Repository Switchboard</name>
    <url>http://repo1.maven.org/maven2</url>
  </repository>
</repositories>
```

任何命令行输入或输出将如下所示：

```java
set PATH =%PATH%;%M2_HOME%\bin

```

**新术语**和**重要词汇**将以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中将如下所示："要使 m2eclipse 使用外部 Maven，请转到 Eclipse 中的**窗口** | **首选项**，然后出现**首选项**窗口。"

### 注意

警告或重要注意事项将以如下框中显示。

### 小贴士

小贴士和技巧如下所示。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者反馈对我们开发您真正从中受益的标题非常重要。

要向我们发送一般反馈，只需将电子邮件发送到`<feedback@packtpub.com>`，并在邮件主题中提及书籍标题。

如果您在某个主题领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您是 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助您从购买中获得最大收益。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 错误清单

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然会发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误清单，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**错误提交表单**链接，并输入您的错误详细信息来报告它们。一旦您的错误得到验证，您的提交将被接受，错误将被上传到我们的网站，或添加到该标题的现有错误清单中，在“错误清单”部分下。您可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的标题来查看任何现有错误清单。

## 盗版

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过链接到疑似盗版材料的方式，通过`<copyright@packtpub.com>`联系我们。

我们感谢您在保护我们作者以及为我们提供有价值内容的能力方面的帮助。

## 问题

如果您在本书的任何方面遇到问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决。
