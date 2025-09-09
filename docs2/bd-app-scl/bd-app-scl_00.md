# 前言

函数式编程始于学术界，最终在 IT 行业中得到了应用。Scala 语言是一种多范式语言，被大型企业和大型组织使用，可以帮助您获得正确的（在纯函数式编程的意义上）软件，同时，软件既实用又可扩展。

Scala 拥有一个非常丰富的生态系统，包括 Play 框架、Akka、Slick、Gatling、Finable 等。在这本书中，我们将从函数式和 ReactiveX 编程的基本原则和思想开始，并通过实际示例，您将学习如何使用 Scala 生态系统中最重要的框架，如 Play、Akka 和 Slick 进行编码。

您将学习如何使用 SBT 和 Activator 引导 Scala 应用程序，如何逐步构建 Play 和 Akka 应用程序，以及我们如何使用云和 NetflixOSS 堆栈的理论来扩展大规模 Scala 应用程序。这本书将帮助您从基础知识到最先进的知识，以使您成为 Scala 专家。

# 本书涵盖的内容

第一章, *FP、反应式和 Scala 简介*，探讨了如何设置 Scala 开发环境，函数式编程与面向对象编程之间的区别，以及函数式编程的概念。

第二章, *创建您的应用程序架构和使用 SBT 引导*，讨论了整体架构，SBT 基础知识，以及如何创建您自己的应用程序。

第三章, *使用 Play 框架开发 UI*，涵盖了 Scala 中网络开发的原则，创建我们的模型，创建我们的视图，以及添加验证。

第四章, *开发反应式后端服务*，介绍了反应式编程原则，重构我们的控制器，以及将 Rx Scala 添加到我们的服务中。

第五章, *测试您的应用程序*，探讨了 Scala 和 JUnit 中的测试原则，行为驱动开发原则，在我们的测试中使用 ScalaTest 规范和 DSL，以及使用 SBT 运行我们的测试。

第六章, *使用 Slick 进行持久化*，涵盖了使用 Slick 的数据库持久化原则，在您的应用程序中处理函数式关系映射，使用 SQL 支持创建所需的查询，以及通过异步数据库操作改进代码。

第七章, *创建报告*，帮助您了解 Jasper 报告并添加数据库报告到您的应用程序中。

第八章，*使用 Akka 开发聊天程序*，讨论了 actor 模型、actor 系统、actor 路由和调度器。

第九章，*设计您的 REST API*，探讨了 REST 和 API 设计，使用 REST 和 JSON 创建我们的 API，添加验证，添加背压，并创建 Scala 客户端。

第十章，*扩展*，涉及架构原则和扩展 UI、响应式驱动程序和可发现性。它还涵盖了中间层负载均衡器、超时、背压和缓存，并指导你使用 Akka 集群扩展微服务，以及使用 Docker 和 AWS 云扩展基础设施。

# 你需要为这本书准备什么

为这本书，你需要以下内容：

+   Ubuntu Linux 14 或更高版本

+   Java 8 更新 48 或更高版本

+   Scala 2.11.7

+   Typesafe Activator 1.3.9

+   Jasper Reports 设计器

+   Linux 的 Windows 字体

+   Eclipse IDE

# 这本书面向的对象

这本书是为想要学习 Scala、以及函数式和响应式技术的专业人士而写的。本书主要面向软件开发者、工程师和架构师。这是一本实用的书，包含实用的代码；然而，我们也提供了关于函数式和响应式编程的理论。

# 规范

在这本书中，你会找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称将显示如下：“下一步是创建名为`SCALA_HOME`的环境变量，并将 Scala 二进制文件放入`PATH`变量中。”

代码块设置如下：

```java
    package scalabook.javacode.chap1; 

    public class HelloWorld { 
      public static void main(String args[]){ 
        System.out.println("Hellow World"); 
      } 
    }
```

任何命令行输入或输出都写作如下：

```java
export JAVA_HOME=~/jdk1.8.0_77
export PATH=$PATH:$JAVA_HOME/bin

```

新术语和重要词汇以粗体显示。屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下：“**Actor**行为是你将在你的**Actor**内部拥有的代码。”

### 注意

警告或重要提示会以这样的框中出现。

### 提示

技巧和窍门会像这样显示。

# 读者反馈

我们欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢什么或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出你真正能从中获得最大收益的标题。

要向我们发送一般反馈，只需发送电子邮件至 feedback@packtpub.com，并在邮件主题中提及本书的标题。

如果你在某个主题领域有专业知识，并且对撰写或为本书做出贡献感兴趣，请参阅我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，你已经是 Packt 图书的骄傲拥有者，我们有一些东西可以帮助你从购买中获得最大收益。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)上的账户下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的“支持”标签上。

1.  点击“代码下载与勘误表”。

1.  在搜索框中输入书的名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买本书的地方。

1.  点击“代码下载”。

文件下载完成后，请确保使用最新版本解压缩或提取文件夹：

+   Windows 版的 WinRAR / 7-Zip

+   Mac 版的 Zipeg / iZip / UnRarX

+   Linux 版的 7-Zip / PeaZip

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Building-Applications-with-Scala`](https://github.com/PacktPublishing/Building-Applications-with-Scala)。我们还有其他来自我们丰富图书和视频目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

## 下载本书的颜色图像

我们还为您提供了一个包含本书中使用的截图/图表的颜色图像的 PDF 文件。这些颜色图像将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/BuildingApplicationswithScala_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/BuildingApplicationswithScala_ColorImages.pdf)下载此文件。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现了错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书，点击勘误提交表单链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书的名称。所需信息将出现在勘误部分下。

## 盗版

在互联网上，版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过版权@packtpub.com 与我们联系，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和我们提供有价值内容的能力方面所提供的帮助。

## 问题

如果您对本书的任何方面有问题，您可以通过 questions@packtpub.com 与我们联系，我们将尽力解决问题。
