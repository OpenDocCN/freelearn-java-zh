# 前言

Scala 是一种大胆的编程语言，它在 JVM 上融合了面向对象和函数式编程概念。Scala 从相对的默默无闻发展成为一个开发健壮和可维护 JVM 应用程序的首选。然而，没有深入了解语言及其提供的先进特性，编写高性能应用程序仍然是一个挑战。

自 2011 年以来，我们一直使用 Scala 来解决具有严格性能要求的复杂商业挑战。在《Scala 高性能编程》中，我们分享了多年来学到的经验和在编写软件时应用的技巧。本书中，我们探讨了语言及其工具和广泛使用的库生态系统。

我们编写这本书的目标是帮助您理解语言为您提供的选项。我们赋予您收集所需信息的能力，以便在软件系统中做出明智的设计和实现决策。我们不是给您一些 Scala 示例代码然后让您去探索，而是帮助您学习如何钓鱼，并为您提供编写更函数式和更高效软件的工具。在这个过程中，我们通过构建类似现实世界问题的商业问题来激发技术讨论。我们希望通过阅读这本书，您能够欣赏 Scala 的力量，并找到编写更函数式和更高效应用程序的工具。

# 本书涵盖的内容

第一章, *性能之路*，介绍了性能的概念以及这个主题的重要术语。

第二章, *在 JVM 上测量性能*，详细介绍了在 JVM 上可用于测量和评估性能的工具，包括 JMH 和 Flight Recorder。

第三章, *释放 Scala 性能*，提供了一次对各种技术和模式的引导之旅，以利用语言特性并提高程序性能。

第四章, *探索集合 API*，讨论了由标准 Scala 库提供的各种集合抽象。在本章中，我们重点关注即时评估的集合。

第五章, *懒集合和事件源*，是一个高级章节，讨论了两种类型的懒序列：视图和流。我们还简要概述了事件源范式。

第六章, *Scala 中的并发*，讨论了编写健壮并发代码的重要性。我们深入探讨了标准库提供的 Future API，并介绍了来自 Scalaz 库的 Task 抽象。

第七章, *针对性能的架构*，这一最后一章结合了之前涵盖的主题的更深入知识，并探讨了 CRDTs 作为分布式系统的构建块。本章还探讨了使用免费单子进行负载控制策略，以构建在高吞吐量情况下具有有限延迟特性的系统。

# 你需要为此书准备的内容

为了运行所有代码示例，您应该在您的操作系统上安装 Java 开发工具包（JDK）版本 8 或更高版本。本书讨论了 Oracle HotSpot JVM，并展示了包含在 Oracle JDK 中的工具。您还应该从[`www.scala-sbt.org/download.html`](http://www.scala-sbt.org/download.html)获取`sbt`（写作时版本为 0.13.11）的最新版本。

# 本书面向的对象

您应该具备 Scala 编程语言的基本知识，对基本的函数式编程概念有所了解，并具有编写生产级 JVM 软件的经验。我们建议 Scala 和函数式编程新手在阅读本书之前花些时间研究其他资源，以便更好地利用本书。两个优秀的以 Scala 为中心的资源是*《Scala 编程》*，*Artima Press*和*《Scala 函数式编程》*，*Manning Publications*。前者最适合那些希望首先理解语言然后是函数式编程范式的强大面向对象 Java 程序员。后者重点在于函数式编程范式，而对语言特定结构的关注较少。

# 约定

在本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL 和用户输入将如下所示：“`-XX:+FlightRecorderOptions`接受一个名为`settings`的参数，默认情况下，它指向`$JAVA_HOME/jre/lib/jfr/default.jfc`。”

代码块设置如下：

```java
def sum(l: List[Int]): Int = l match { 
  case Nil => 0 
  case x :: xs => x + sum(xs) 
}
```

任何命令行输入或输出将如下所示：

```java
sbt 'project chapter2' 'set javaOptions := Seq("-Xmx1G")' 'runMain highperfscala.benchmarks.ThroughputBenchmark
src/main/resources/historical_data 250000'

```

新术语和重要词汇将以粗体显示。屏幕上看到的单词，例如在菜单或对话框中，将以如下形式出现在文本中：“让我们从**代码**标签组的**概览**标签开始。”

### 注意

警告或重要提示将以如下框内显示。

### 小贴士

小贴士和技巧将以如下形式出现。

# 读者反馈

我们欢迎读者的反馈。请告诉我们您对本书的看法——您喜欢或不喜欢的地方。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中受益的标题。

要向我们发送一般反馈，请简单地发送电子邮件至 feedback@packtpub.com，并在邮件主题中提及本书的标题。

如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](https://www.packtpub.com/books/info/packt/authors)。

# 客户支持

现在您已经是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**支持**选项卡上。

1.  点击**代码下载 & 错误清单**。

1.  在**搜索**框中输入书籍名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买本书的来源。

1.  点击**代码下载**。

您也可以通过点击 Packt Publishing 网站书籍网页上的**代码文件**按钮来下载代码文件。您可以通过在搜索框中输入书籍名称来访问此页面。请注意，您需要登录您的 Packt 账户。

一旦文件下载完成，请确保使用最新版本的以下软件解压或提取文件夹：

+   WinRAR / 7-Zip for Windows

+   Zipeg / iZip / UnRarX for Mac

+   7-Zip / PeaZip for Linux

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Scala-High-Performance-Programming`](https://github.com/PacktPublishing/Scala-High-Performance-Programming)。我们还有其他来自我们丰富图书和视频目录的代码包可供下载，网址为[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。请查看它们！

## 下载本书的颜色图像

我们还为您提供了一个包含本书中使用的截图/图表的颜色图像的 PDF 文件。这些颜色图像将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/ScalaHighPerformanceProgramming_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/ScalaHighPerformanceProgramming_ColorImages.pdf)下载此文件。

## 错误清单

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这个问题，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书，点击勘误提交表单链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误表部分。

要查看之前提交的勘误表，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书名。所需信息将出现在勘误表部分。

## 侵权

互联网上对版权材料的侵权是一个持续存在的问题，涉及所有媒体。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现任何形式的非法复制我们的作品，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过发送邮件至 copyright@packtpub.com 并附上疑似侵权材料的链接与我们联系。

我们感谢您在保护我们作者和我们提供有价值内容的能力方面的帮助。

## 询问

如果您在这本书的任何方面遇到问题，您可以给我们发送邮件至 questions@packtpub.com，我们将尽力解决问题。
