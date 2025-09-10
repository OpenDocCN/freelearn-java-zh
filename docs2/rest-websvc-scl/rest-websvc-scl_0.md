# 前言

Scala 是一种出色的语言，它提供了函数式编程和面向对象编程之间的一种很好的组合。使用 Scala，您能够以高度生产力的方式编写美观、简洁且易于维护的代码。Scala 还提供了创建 REST 服务非常有用的语言结构。在 Scala 生态系统中，有大量的 REST 框架，每个框架都允许您以略微不同的方式创建此类服务。本书为您概述了五个最成熟和灵活的 REST 框架，并通过广泛的示例向您展示您如何使用这些框架的各种功能。

# 本书涵盖的内容

第一章, *开始使用 RESTful Web 服务*，向您展示如何获取代码、设置您的 IDE 以及启动并运行一个最小的服务。它还提供了关于 REST 和我们将在整本书中使用的模型的一些背景信息。

第二章, *使用 Finagle 和 Finch 创建函数式风格的 REST 服务*，解释了您如何使用 Finch 库与 Finagle 一起创建遵循函数式编程方法的 REST 服务。

第三章, *使用未过滤的 REST 服务模式匹配*，展示了您如何使用 Unfiltered，一个小的 REST 库，来创建 REST 服务。使用 Unfiltered，您可以完全控制您的 HTTP 请求和响应的处理方式。

第四章, *使用 Scalatra 创建简单的 REST 服务*，使用 Scalatra 框架的一部分来创建一个简单的 REST 服务。Scalatra 是一个出色的直观 Scala 框架，它使得创建 REST 服务变得非常简单。

第五章, *使用 Akka HTTP DSL 定义 REST 服务*，专注于 Akka HTTP 的 DSL 部分，并解释了您如何使用基于知名的 Spray DSL 的 DSL，来创建易于组合的 REST 服务。

第六章, *使用 Play 2 框架创建 REST 服务*，解释了您如何使用知名的 Play 2 Web 框架的一部分来创建 REST 服务。

第七章, *JSON, HATEOAS 和文档*，深入探讨了 REST 的两个重要方面——JSON 和 HATEOAS。本章通过示例向您展示如何处理 JSON 并将 HATEOAS 纳入讨论的框架中。

# 您需要这本书的内容

要使用本书中的示例，您除了 Scala 之外不需要任何特殊软件。第一章解释了如何安装最新的 Scala 版本，从那里，您可以获取本书中解释的所有框架的库。

# 本书面向的对象

如果你是一位有 Scala 经验的 Scala 开发者，并且想要了解 Scala 世界中可用的框架概述，那么这本书非常适合你。你需要具备关于 REST 和 Scala 的一般知识。这本书非常适合寻找与 Scala 一起使用的良好 REST 框架的资深 Scala（或其他语言）开发者。

# 习惯用法

在本书中，您会发现许多不同的文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL 和用户输入应如下所示：“在这种情况下，我们拒绝请求，因为`HttpMethod`（动词）与我们能够处理的任何内容都不匹配。”

代码块应如下设置：

```java
class ScalatraBootstrapStep1 extends LifeCycle {
  override def init(context: ServletContext) {
    context mount (new ScalatraStep1, "/*")
  }
}
```

任何命令行输入或输出都应如下所示：

```java
$ sbt runCH04-runCH04Step1
[info] Loading project definition from /Users/jos/dev/git/rest-with-scala/project
[info] Set current project to rest-with-scala (in build file:/Users/jos/dev/git/rest-with-scala/)
[info] Running org.restwithscala.chapter4.steps.ScalatraRunnerStep1 
10:51:40.313 [run-main-0] INFO  o.e.jetty.server.ServerConnector - Started ServerConnector@538c2499{HTTP/1.1}{0.0.0.0:8080}
10:51:40.315 [run-main-0] INFO  org.eclipse.jetty.server.Server - Started @23816ms

```

**新术语**和**重要词汇**将以粗体显示。你在屏幕上看到的单词，例如在菜单或对话框中，将以如下方式显示：“在 Postman 中，你可以使用**步骤 03 – 获取所有任务**请求来完成此操作。”

### 注意

警告或重要注意事项将以如下框中显示。

### 小贴士

小技巧和窍门将以如下方式显示。

# 读者反馈

我们欢迎读者的反馈。请告诉我们您对本书的看法——您喜欢或不喜欢的地方。读者反馈对我们来说非常重要，因为它帮助我们开发出您真正能从中获得最大价值的标题。

要向我们发送一般反馈，请简单地发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中提及本书的标题。

如果您在某个主题上具有专业知识，并且您对撰写或为本书做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)下载示例代码文件，这是您购买的所有 Packt Publishing 书籍的账户。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 错误清单

尽管我们已经尽一切努力确保内容的准确性，但错误仍然会发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误****提交****表单**链接，并输入您的勘误详情。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误表，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将在**勘误**部分显示。

## 海盗行为

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 `<copyright@packtpub.com>` 与我们联系，并提供疑似盗版材料的链接。

我们感谢您在保护我们的作者和我们为您提供有价值内容的能力方面的帮助。

## 问答

如果您对本书的任何方面有问题，您可以联系我们的 `<questions@packtpub.com>`，我们将尽力解决问题。
