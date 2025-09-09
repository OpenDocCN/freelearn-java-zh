# 前言

在过去几年中，我们看到了在 JVM 上创建新语言的普遍趋势。有各种不同范式和工作方式的新语言。Clojure 就是其中之一，我们认为它值得学习。

Java 虚拟机在 Clojure 程序的性能中扮演着至关重要的角色。Clojure 将函数式编程引入 Java 虚拟机（JVM），并通过 ClojureScript 引入到网页浏览器中。

随着并行数据处理和多核架构的最近兴起，函数式编程语言在创建既可证明又高效软件方面变得更加流行。与其他函数式编程语言一样，Clojure 专注于使用函数和不可变数据结构来编写程序。Clojure 还通过使用符号表达式和动态类型系统添加了 Lisp 的痕迹。

# 本学习路径涵盖的内容

模块 1，*面向 Java 开发者的 Clojure*，向您介绍 Clojure 及其观点。您将了解为什么不可变对象不仅可行，而且使用它们是一个好主意。您还将了解函数式编程，并了解它如何适应不可变程序的概念。您将理解将代码表示为相同语言的数据结构的强大理念。此外，我们将找出与您已知的 Java 语言的相似之处和不同之处，以便您理解 Clojure 世界是如何运作的。

模块 2，*《Clojure 高性能编程（第 2 版）*，增加了对性能管理 JVM 工具的关注，并探讨了如何使用这些工具。本模块将为您提供性能测量和性能分析工具，以及分析并调整 Clojure 代码性能特性的知识。

模块 3，*精通 Clojure*，将带您了解 Clojure 语言的有趣特性。我们还将讨论 Clojure 中一些更高级和不太为人所知的编程结构。我们还将描述一些 Clojure 生态系统中的库，这些库可以在我们的程序中实际使用。完成本模块后，您将不再需要被说服 Clojure 语言的优雅和强大。

# 您需要本学习路径的内容

您需要 Java SDK。您应该能够在任何操作系统上运行示例；在我们的示例中，如果环境中有一个 shell，应该更容易理解。（我们主要关注 Mac OS X。）

您应该获取 Java 开发工具包（JDK）版本 8 或更高版本（您可以从[`www.oracle.com/technetwork/java/javase/downloads/`](http://www.oracle.com/technetwork/java/javase/downloads/)为您的操作系统获取，以便处理所有示例）。本书讨论了 Oracle HotSpot JVM，因此如果您可能的话，您可能希望获取 Oracle JDK 或 OpenJDK（或 Zulu）。您还应该从[`leiningen.org/`](http://leiningen.org/)获取最新的 Leiningen 版本，以及从[`jd.benow.ca/`](http://jd.benow.ca/)获取 JD-GUI。

您还需要一个文本编辑器或集成开发环境（IDE）。如果您已经有一个偏好的文本编辑器，您可能可以使用它。前往[`dev.clojure.org/display/doc/Getting+Started`](http://dev.clojure.org/display/doc/Getting+Started)获取适用于特定环境的插件列表，以编写 Clojure 代码。如果您没有偏好，建议您使用 Eclipse 与 Counterclockwise ([`doc.ccw-ide.org/`](http://doc.ccw-ide.org/))或 Light Table ([`lighttable.com/`](http://lighttable.com/))。本书中的一些示例也可能需要网络浏览器，例如 Chrome（42 或以上）、Firefox（38 或以上）或 Microsoft Internet Explorer（9 或以上）。

# 这条学习路径面向的对象

这条学习路径针对的是熟悉构建应用程序的 Java 开发者，他们现在想利用 Clojure 的强大功能。

# 读者反馈

我们欢迎读者的反馈。请告诉我们您对这门课程的看法——您喜欢或不喜欢的地方。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。

要发送给我们一般性的反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在邮件的主题中提及课程的标题。

如果你在某个领域有专业知识，并且对撰写或参与一本书籍感兴趣，请参阅我们的作者指南，网址为[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是 Packt 课程的骄傲所有者，我们有一些事情可以帮助你从购买中获得最大收益。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载此课程的示例代码文件。如果您在其他地方购买了此课程，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部**支持**标签上。

1.  点击**代码下载与勘误**。

1.  在**搜索**框中输入课程的名称。

1.  选择您想要下载代码文件的课程。

1.  从下拉菜单中选择您购买此课程的来源。

1.  点击**代码下载**。

您还可以通过点击 Packt Publishing 网站课程网页上的**代码文件**按钮来下载代码文件。您可以通过在**搜索**框中输入课程名称来访问此页面。请注意，您需要登录到您的 Packt 账户。

文件下载完成后，请确保您使用最新版本解压或提取文件夹：

+   WinRAR / 7-Zip for Windows

+   Zipeg / iZip / UnRarX for Mac

+   7-Zip / PeaZip for Linux

该课程的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Clojure-High-Performance-JVM-Programming`](https://github.com/PacktPublishing/Clojure-High-Performance-JVM-Programming)。我们还有其他来自我们丰富图书、视频和课程目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。请查看它们！

## 勘误

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在我们的课程中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者的挫败感，并帮助我们改进课程的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的课程，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入课程名称。所需信息将出现在**勘误**部分下。

## 侵权

在所有媒体中，互联网上对版权材料的侵权都是一个持续存在的问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过发送电子邮件到`<copyright@packtpub.com>`并附上疑似侵权材料的链接与我们联系。

我们感谢您的帮助，保护我们的作者和提供有价值内容的能力。

## 询问

如果您在课程的任何方面遇到问题，您可以通过发送电子邮件到`<questions@packtpub.com>`与我们联系，我们将尽力解决问题。
