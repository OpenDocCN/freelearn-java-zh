# 前言

今天的 Scala 与其早期版本大相径庭。

语言的第二版已经超过十二年历史，并且经历了与支持特性和库实现相关的多次变更。当时被认为至关重要的许多特性，例如对 XML 字面量的支持、Swing 和 Actors，都被从语言核心移动到外部库或被开源替代方案所取代。Scala 成为我们今天所知的编程语言的功能之一是通过直接添加或通过在发行版中包含另一个开源库来实现的。最显著的例子是在 2.10 版本中采用 Akka。

Scala 2.13，其重点是模块化标准库和简化集合，带来了进一步的变更。然而，这些变更不仅影响 Scala 的技术方面。多年来用它来解决实际问题，帮助我们收集了更多关于结构化函数程序和使用面向对象特性以新方式获得优势的知识。正如以前版本的习惯是使用“没有分号的 Java”一样，现在使用单子转换器和类型级编程技术来构建程序已成为常规。

本书通过提供对重新设计的标准库和集合的全面指南，以及深入探讨类型系统和函数的第一级支持，同时处理技术和架构上的变化。它讨论了隐式作为构建类型类的主要机制，并探讨了测试 Scala 代码的不同方法。它详细介绍了在函数式编程中使用的抽象构建块，提供了足够的知识来选择和使用任何现有的函数式编程库。它通过涵盖 Akka 框架和响应式流来探索响应式编程。最后，它讨论了微服务以及如何使用 Scala 和 Lagom 框架来实现它们。

# 本书面向对象

作为一名软件开发人员，你对某些命令式编程语言有实际的知识，可能是 Java。

你已经具备一些 Scala 基础知识，并在实际项目中使用过它。作为一个 Scala 初学者，你对它的生态系统之丰富、解决问题方式的多样性以及可用库的数量感到惊讶。你希望提高你的 Scala 技能，以便能够充分利用语言及其重构的标准库的潜力，最优地使用其丰富的类型系统来尽可能接近问题域地制定程序，并通过理解底层范式、使用相关语言特性和开源库来利用其功能能力。

# 要充分利用本书

1.  我们期望读者能够舒适地构建和实现简单的 Scala 程序，并熟悉 SBT 和 REPL。不需要具备反应式编程、Akka 或微服务的先验知识，但了解相关概念将有益。

1.  要使用本书中的代码示例，需要安装 Java 1.8+和 SBT 1.2+。建议安装 Git 以简化从 GitHub 检出源代码。对于那些没有准备好先决软件的读者，我们在附录 A 中提供了 Java 和 SBT 的安装说明。

# 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的账户下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择 SUPPORT 标签。

1.  点击代码下载与勘误。

1.  在搜索框中输入本书的名称，并遵循屏幕上的说明。

一旦文件下载完成，请确保您使用最新版本解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Learn-Scala-Programming`](https://github.com/PacktPublishing/Learn-Scala-Programming)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：`www.packtpub.com/sites/default/files/downloads/9781788836302_ColorImages.pdf`。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“在 Scala 2.13 中，`StringOps`已通过字符串字面量解析的返回选项的方法进行了扩展。支持的所有类型包括所有数值类型和`Boolean`。”

代码块设置如下：

```java
object UserDb {
  def getById(id: Long): User = ???
  def update(u: User): User = ???
  def save(u: User): Boolean = ???
}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
scala> val user = User("John", "Doe", "jd@mail.me")
user: User = User(John,Doe,jd@mail.me)
scala> naiveToJsonString(user)
res1: String = { "name": "John", "surname": "Doe", "email": "jd@mail.me" }
```

任何命令行输入或输出都应如下编写：

```java
take
-S--c--a--l--a-- --2--.--1--3-
take
Lazy view constructed: -S-S-c-C-a-A-l-L-a-A- - 
Lazy view forced: -S--c--a--l--a-- -List(S, C, A, L, A, )
Strict: List(S, C, A, L, A, )
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。

警告或重要注意事项看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

欢迎读者提供反馈。

**一般反馈**：如果您对这本书的任何方面有疑问，请在邮件主题中提及书名，并给我们发送邮件至 `customercare@packtpub.com`。

**勘误表**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，如果您能向我们报告这一点，我们将不胜感激。请访问 [www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过发送链接至 `copyright@packt.com` 与我们联系。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评价

请留下您的评价。一旦您阅读并使用过这本书，为何不在购买它的网站上留下评价呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 公司可以了解您对我们产品的看法，并且我们的作者可以查看他们对书籍的反馈。谢谢！

想了解更多关于 Packt 的信息，请访问 [packt.com](http://www.packt.com/)。
