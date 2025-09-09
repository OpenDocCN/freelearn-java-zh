# 前言

Play 框架是一个用 Java 和 Scala 编写的开源 Web 应用程序框架。它遵循模型-视图-控制器（Model-View-Controller）的架构模式。

它使用户能够在保持 Play 框架的关键属性和功能的同时使用 Scala 进行应用程序开发。这导致 Web 应用程序更快且可扩展。此外，它使用更函数式和“Scala 惯用”的编程风格，而不牺牲简单性和开发者友好性。

本书将提供有关使用 Play 框架开发 Scala Web 应用程序的高级信息。这将帮助 Scala Web 开发者掌握 Play 2.0，并用于专业的 Scala Web 应用程序开发。

# 本书涵盖的内容

第一章，*Play 入门*，解释了如何使用 Play 框架构建简单应用程序。我们还探讨了项目结构，以便您了解框架如何通过构建文件插入所需的设置。

第二章，*定义动作*，解释了我们可以如何定义具有默认解析器和结果的特定应用程序动作，以及具有自定义解析器和结果的动作。

第三章，*构建路由*，展示了路由在 Play 应用程序中的重要性。除此之外，我们还检查了 Play 提供的各种默认方法，这些方法可以简化路由过程。

第四章，*探索视图*，解释了如何使用 Twirl 和 Play 提供的各种其他辅助方法创建视图。在本章中，您还将学习如何在 Play 应用程序中支持多种语言，使用内置的 i18n API。

第五章，*数据处理*，展示了如何使我们在使用 Play 框架构建的应用程序中持久化应用程序数据的不同方法。此外，您还可以了解如何使用 Play 缓存 API 以及它是如何工作的。

第六章，*响应式数据流*，讨论了迭代器（Iteratee）、枚举器（Enumerator）和枚举者（Enumeratee）的概念，以及它们如何在 Play 框架中实现并用于内部使用。

第七章，*玩转全局变量*，揭示了通过全局插件为 Play 应用程序提供的功能。我们还讨论了请求-响应生命周期的钩子，使用这些钩子我们可以拦截请求和响应，并在必要时修改它们。

第八章，*WebSockets 和 Actors*，简要介绍了 Actor 模型以及在应用程序中使用 Akka Actors 的用法。我们还使用不同的方法，在 Play 应用程序中定义了一个具有各种约束和要求的 WebSocket 连接。

第九章, *测试*，展示了如何使用 Specs2 和 ScalaTest 测试 Play 应用程序。我们探讨了可用于简化 Play 应用程序测试的不同辅助方法。

第十章, *调试和日志记录*，介绍了如何在 IDE 中配置 Play 应用程序的调试。在本章中，您将学习如何在 Scala 控制台中启动 Play 应用程序。本章还强调了 Play Framework 提供的日志 API 以及自定义日志格式的自定义方法。

第十一章, *Web 服务和认证*，解释了 WS（WebService）插件及其暴露的 API。我们还使用 OpenID 和 OAuth 1.0a 从服务提供商访问用户数据。

第十二章, *Play 在生产环境中的应用*，解释了如何在生产环境中部署 Play 应用程序。在部署应用程序时，我们还会检查默认可用的不同打包选项（RPM、Debian、ZIP、Windows 等）。

第十三章, *编写 Play 插件*，解释了所有插件及其声明、定义和最佳实践。

# 您需要为本书准备的

在开始阅读本书之前，请确保您已安装所有必要的软件。本书的先决条件如下：

+   Java: [`www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html`](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)

+   SBT 或 Activator: [`typesafe.com/community/core-tools/activator-and-sbt`](https://typesafe.com/community/core-tools/activator-and-sbt)

+   MariaDB: [`downloads.mariadb.org/`](https://downloads.mariadb.org/)

+   MongoDB: [`www.mongodb.org/downloads`](http://www.mongodb.org/downloads)

+   Cassandra（可选）: [`cassandra.apache.org/download/`](http://cassandra.apache.org/download/)

# 本书面向的对象

本书旨在帮助那些热衷于掌握 Play Framework 内部工作原理的开发者，以便有效地构建和部署与 Web 相关的应用程序。假设您对核心应用程序开发技术有基本的了解。

# 规范

在本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称如下所示："更新索引模板，以便每个`<li>`元素都有一个按钮，点击该按钮将向服务器发送删除请求。"

代码块设置如下：

```java
def runningT(block: => T): T = {
    synchronized {
      try {
        Play.start(app)
        block
      } finally {
        Play.stop()
      }
    }
  }
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
class WebSocketChannel(out: ActorRef)
  extends Actor with ActorLogging {

  val backend = Akka.system.actorOf(DBActor.props)
  def receive: Actor.Receive = {
    case jsRequest: JsValue =>
      backend ! convertJsonToMsg(jsRequest)
    case x:DBResponse =>
      out ! x.toJson
  }
}
```

任何命令行输入或输出都应如下编写：

```java
> run
[info] Compiling 1 Scala source to /AkkaActorDemo/target/scala-2.10/classes...
[info] Running com.demo.Main
?od u od woH ,olleH
ekops ew ecnis gnoL neeB
Sorry, didn't quite understand that I can only process a String.

```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中如下显示：“当您点击**提交**时，表单不会提交，并且不会使用`globalErrors`显示错误。”

### 注意

警告或重要注意事项以如下框中的形式出现。

### 小贴士

小贴士和技巧如下所示。

# 读者反馈

我们欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。

要向我们发送一般反馈，只需通过电子邮件发送到`<feedback@packtpub.com>`，并在邮件主题中提及书籍的标题。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的骄傲拥有者，我们有多个方面可以帮助您充分利用您的购买。

## 下载示例代码

您可以从您的账户下载示例代码文件，地址为[`www.packtpub.com`](http://www.packtpub.com)，适用于您购买的所有 Packt 出版书籍。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，并注册以将文件直接发送给您。

## 错误清单

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中找到错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进此书的后续版本。如果您发现任何错误清单，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**错误提交表单**链接，并输入您的错误详细信息来报告它们。一旦您的错误清单得到验证，您的提交将被接受，错误清单将被上传到我们的网站或添加到该标题错误清单部分。

要查看之前提交的错误清单，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**错误清单**部分。

## 盗版

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，请立即提供位置地址或网站名称，以便我们可以追究补救措施。

请通过链接将疑似盗版材料发送至 `<copyright@packtpub.com>` 与我们联系。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面提供的帮助。

## 问题

如果您对本书的任何方面有问题，您可以通过 `<questions@packtpub.com>` 与我们联系，我们将尽力解决问题。
