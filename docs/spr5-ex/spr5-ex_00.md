# 前言

随着需求的增长，组织正在寻找健壮且可扩展的系统。因此，Spring 框架已成为 Java 开发中最受欢迎的框架。它不仅简化了软件开发，还提高了开发者的生产力。本书涵盖了使用 Spring 在 Java 中开发健壮应用程序的有效方法。

# 本书面向的对象

对于刚开始使用 Spring 的开发者来说，本书将介绍 Spring 5.0 的新框架概念，并随后讲解其在 Java 和 Kotlin 中的实现。本书还将帮助经验丰富的 Spring 开发者深入了解 Spring 5.0 中新增的功能。

# 本书涵盖的内容

第一章，《春之之旅》，将引导你了解 Spring 框架的主要概念。在这里，我们将学习通过安装 OpenJDK、Maven、IntelliJ IDEA 和 Docker 来设置环境。最后，我们将创建我们的第一个 Spring 应用程序。

第二章，《春之世界起步——CMS 应用程序》，将首先通过使用 Spring Initializr 来为我们的 CMS 应用程序创建配置来动手实践。然后我们将学习如何创建 REST 资源，添加服务层，并最终与 AngularJS 集成。

第三章，《使用 Spring Data 和响应式模式进行持久化》，将在上一章创建的 CMS 应用程序的基础上进行构建。在这里，我们将通过了解 Spring Data Reactive MongoDB 和 PostgreSQL 来学习如何在真实数据库上持久化数据。我们还将了解 Project Reactor，它将帮助你创建 JVM 生态系统中的非阻塞应用程序。

第四章，《Kotlin 基础和 Spring Data Redis》，将为你提供一个 Kotlin 的基本介绍，同时展示该语言的优势。然后我们将学习如何使用 Redis，它将作为消息代理使用发布/订阅功能。

第五章，《响应式 Web 客户端》，将教你如何使用 Spring Reactive Web 客户端并以响应式的方式执行 HTTP 调用。我们还将介绍 RabbitMQ 和 Spring Actuator。

第六章，《玩转服务器端事件》，将帮助你开发一个能够通过文本内容过滤推文的程序。我们将通过使用服务器端事件（这是一种从服务器向客户端发送数据流的标准方式）来消费推特流来实现这一点。

第七章，《航空票务系统》，将教你如何使用 Spring Messaging、WebFlux 和 Spring Data 组件来构建航空票务系统。在本章中，你还将了解断路器和 OAuth。最后，我们将创建一个包含许多微服务以确保可扩展性的系统。

第八章，*电路断路器和安全*，将帮助您了解如何为我们的业务微服务应用服务发现功能，同时了解电路断路器模式如何帮助我们为应用程序带来弹性。

第九章，*整合一切*，将使整本书的视角更加清晰，同时向您介绍 Turbine 服务器。我们还将查看 Hystrix 仪表板来监控我们的不同微服务，以确保应用程序的可维护性和最佳性能。

# 为了充分利用本书

阅读本书的读者应具备基本的 Java 知识。了解分布式系统将是一个额外的优势。

要执行本书中的代码文件，您需要以下软件/依赖项：

+   IntelliJ IDEA 社区版

+   Docker CE

+   pgAdmin

+   Docker Compose

您将通过本书获得安装过程等方面的帮助。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书名，并遵循屏幕上的说明。

文件下载完成后，请确保使用最新版本解压或提取文件夹：

+   Windows 系统下的 WinRAR/7-Zip

+   Mac 系统下的 Zipeg/iZip/UnRarX

+   Linux 系统下的 7-Zip/PeaZip

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Spring-5.0-By-Example`](https://github.com/PacktPublishing/Spring-5.0-By-Example)。我们还有其他丰富的图书和视频代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。请查看它们！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/Spring50ByExample_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/Spring50ByExample_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“它包括在默认配置文件`application.yaml`中配置的基础设施连接。”

代码块设置如下：

```java
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
management:
  endpoints:
    web:
 expose: "*"
```

任何命令行输入或输出都应如下所示：

```java
docker-compose up -d
```

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“下一个屏幕将显示，我们可以配置环境变量：”

警告或重要提示看起来是这样的。

小贴士和技巧看起来是这样的。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：请通过 `feedback@packtpub.com` 发送电子邮件，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过 `questions@packtpub.com` 发送电子邮件给我们。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一点。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式发现我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过 `copyright@packtpub.com` 联系我们，并附上材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用了这本书，为何不在购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt 公司可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解 Packt 的更多信息，请访问 [packtpub.com](https://www.packtpub.com/)。
