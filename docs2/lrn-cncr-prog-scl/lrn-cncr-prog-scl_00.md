# 前言

并发无处不在。随着消费市场多核处理器的兴起，对并发编程的需求已经压倒了开发者世界。它曾经用于在程序和计算机系统中表达异步性，主要是一个学术学科，但现在并发编程已成为软件开发中的一种普遍方法。因此，高级并发框架和库正以惊人的速度涌现。近年来，并发计算领域经历了复兴。

随着现代语言和并发框架抽象级别的提高，了解何时以及如何使用它们变得越来越重要。仅仅掌握线程、锁和监视器等经典并发和同步原语已经不再足够。解决许多传统并发问题并针对特定任务定制的先进并发框架正在逐渐主导并发编程的世界。

本书描述了 Scala 中的高级并发编程。它详细解释了各种并发主题，并涵盖了并发编程的基本理论。同时，它描述了现代并发框架，展示了它们的详细语义，并教你如何使用它们。其目标是介绍重要的并发抽象，同时展示它们在实际代码中的工作方式。

我们相信，通过阅读本书，你将获得对并发编程的扎实理论理解，并培养出一套编写正确和高效并发程序所需的有用实用技能。这些技能是成为现代并发专家的第一步。

我们希望你在阅读本书时能像我们写作时一样享受乐趣。

# 本书涵盖的内容

本书分为一系列章节，涵盖了并发编程的多个主题。本书涵盖了 Scala 运行时的一部分基本并发 API，介绍了更复杂的并发原语，并对高级并发抽象提供了广泛的概述。

第一章，*简介*，解释了并发编程的需求，并提供了哲学背景。同时，它涵盖了理解本书其余部分所需的 Scala 编程语言的基础知识。

第二章，*JVM 和 Java 内存模型上的并发*，教你并发编程的基础。这一章将教你如何使用线程，如何保护对共享内存的访问，并介绍 Java 内存模型。

第三章, *并发的传统构建块*，介绍了经典的并发工具，例如线程池、原子变量和并发集合，特别关注与 Scala 语言特性的交互。本书的重点在于现代、高级的并发编程框架。因此，本章概述了传统的并发编程技术，但并不旨在详尽。

第四章, *使用未来和承诺进行异步编程*，是第一章节处理 Scala 特定的并发框架。本章介绍了未来和承诺 API，并展示了在实现异步程序时如何正确使用它们。

第五章, *数据并行集合*，描述了 Scala 并行集合框架。在本章中，你将学习如何并行化集合操作，何时可以并行化它们，以及如何评估这样做带来的性能收益。

第六章, *使用响应式扩展进行并发编程*，教你如何使用响应式扩展框架进行基于事件和异步编程。你将看到事件流上的操作如何对应于集合操作，如何将事件从一个线程传递到另一个线程，以及如何使用事件流设计响应式用户界面。

第七章, *软件事务内存*，介绍了 ScalaSTM 库，用于事务编程，旨在提供一个更安全、更直观的共享内存编程模型。在本章中，你将学习如何使用可伸缩的内存事务保护对共享数据的访问，同时降低死锁和竞态条件的风险。

第八章, *演员*，介绍了演员编程模型和 Akka 框架。在本章中，你将学习如何透明地构建在多台机器上运行的基于消息传递的分布式程序。

第九章, *实践中的并发*，总结了前面章节中介绍的不同并发库。在本章中，你将学习如何选择正确的并发抽象来解决给定的问题，以及如何在设计更大的并发应用程序时将不同的并发抽象组合在一起。

第十章，即 **反应器**，介绍了反应器编程模型，其重点是提高并发和分布式程序中的组合性。这种新兴模型使并发和分布式编程模式能够分离成称为协议的模块化组件。

虽然我们建议您按照章节出现的顺序阅读，但这并非绝对必要。如果您对 第二章 中的内容非常熟悉，即 **JVM 和 Java 内存模型上的并发**，您可以直接学习大多数其他章节。唯一依赖于所有前述章节内容的章节是 第九章，即 **实践中的并发**，其中我们展示了本书主题的实践概述，以及 第十章，即 **反应器**，了解演员和事件流的工作方式对理解它是有帮助的。

# 您需要为本书准备的东西

在本节中，我们描述了阅读和理解本书所必需的一些要求。我们解释了如何安装 Java 开发工具包，这是运行 Scala 程序所必需的，并展示了如何使用 Simple Build Tool 运行各种示例。

本书将不会要求使用 IDE。您用来编写代码的程序完全由您决定，您可以选择任何东西，例如 Vim、Emacs、Sublime Text、Eclipse、IntelliJ IDEA、Notepad++ 或其他文本编辑器。

## 安装 JDK

Scala 程序不是直接编译成原生机器代码，因此不能在各种硬件平台上作为可执行文件运行。相反，Scala 编译器生成一种中间代码格式，称为 Java 字节码。要运行这种中间代码，您的计算机必须安装 Java 虚拟机软件。在本节中，我们解释了如何下载和安装包含 Java 虚拟机和其他有用工具的 Java 开发工具包。

可从不同的软件供应商处获取多个 JDK 实现。我们建议您使用 Oracle JDK 发行版。要下载和安装 Java 开发工具包，请按照以下步骤操作：

1.  在您的网络浏览器中打开以下 URL：[www.oracle.com/technetwork/java/javase/downloads/index.html](http://www.oracle.com/technetwork/java/javase/downloads/index.html)。

1.  如果您无法打开指定的 URL，请访问您的搜索引擎并输入关键词 **JDK 下载**。

1.  一旦您在 Oracle 网站上找到 Java SE 的链接，请下载适合您操作系统的 JDK 7 版本：Windows、Linux 或 Mac OS X；32 位或 64 位。

1.  如果你使用的是 Windows 操作系统，只需运行安装程序。如果你使用的是 Mac OS X，打开 dmg 存档以安装 JDK。最后，如果你使用的是 Linux，将存档解压到 XYZ 目录，并将 bin 子目录添加到 PATH 变量中：

    ```java
     export PATH=XYZ/bin:$PATH

    ```

1.  你现在应该能够在终端中运行 java 和 javac 命令。输入  **`javac`**  命令以查看它是否可用（在这本书中你永远不会直接调用此命令，但运行它可以验证它是否可用）。

你的操作系统可能已经安装了 JDK。为了验证这一点，只需运行 **`javac`** 命令，就像我们在前面的描述中上一步所做的那样。

## 安装和使用 SBT

**简单构建工具**（**SBT**）是一个用于 Scala 项目的命令行构建工具。它的目的是编译 Scala 代码、管理依赖项、持续编译和测试、部署以及许多其他用途。在这本书中，我们将使用 SBT 来管理我们的项目依赖项并运行示例代码。

要安装 SBT，请按照以下说明操作：

1.  前往 [`www.scala-sbt.org/`](http://www.scala-sbt.org/) 网址。

1.  下载适用于你的平台的安装文件。如果你在 Windows 上运行，这是 `msi` 安装程序文件。如果你在 Linux 或 OS X 上运行，这是 `zip` 或 `tgz` 存档文件。

1.  安装 SBT。如果你在 Windows 上运行，只需运行安装程序文件。如果你在 Linux 或 OS X 上运行，将存档内容解压缩到你的家目录中。

你现在可以使用 SBT 了。在以下步骤中，我们将创建一个新的 SBT 项目：

1.  如果你使用的是 Windows，打开命令提示符；如果你使用的是 Linux 或 OS X，打开终端窗口。

1.  创建一个名为 scala-concurrency-examples 的空目录：

    ```java
     $ mkdir scala-concurrency-examples

    ```

1.  将你的路径更改为 scala-concurrency-examples 目录：

    ```java
     $ cd scala-concurrency-examples

    ```

1.  为我们的示例创建一个单独的源代码目录：

    ```java
     $ mkdir src/main/scala/org/learningconcurrency/

    ```

1.  现在，使用你的编辑器创建一个名为 build.sbt 的构建定义文件。此文件定义了各种项目属性。在项目的根目录（scala-concurrency-examples）中创建它。将以下内容添加到构建定义文件中（注意，空行是必需的）：

    ```java
            name := "concurrency-examples"

            version := "1.0"

            scalaVersion := "2.11.1"
    ```

1.  最后，回到终端并从项目的根目录运行 SBT：

    ```java
     $ sbt

    ```

1.  SBT 将启动一个交互式 shell，我们将使用它来向 SBT 提供各种构建命令。

现在，你可以开始编写 Scala 程序了。打开你的编辑器，在 `src/main/scala/org/learningconcurrency` 目录中创建一个名为 `HelloWorld.scala` 的源代码文件。将以下内容添加到 `HelloWorld.scala` 文件中：

```java
    package org.learningconcurrency

    object HelloWorld extends App {
      println("Hello, world!")
    }
```

现在，回到带有 SBT 交互式 shell 的终端窗口，并使用以下命令运行程序：

```java
> run

```

运行此程序应给出以下输出：

```java
Hello, world!

```

这些步骤足以运行本书中的大多数示例。偶尔，在运行示例时，我们将依赖外部库。这些库将由 SBT 从标准软件仓库自动解析。对于某些库，我们需要指定额外的软件仓库，因此我们在 `build.sbt` 文件中添加以下行：

```java
    resolvers ++= Seq(
      "Sonatype OSS Snapshots" at
        "https://oss.sonatype.org/content/repositories/snapshots",
      "Sonatype OSS Releases" at
        "https://oss.sonatype.org/content/repositories/releases",
      "Typesafe Repository" at
        "http://repo.typesafe.com/typesafe/releases/"
    )
```

现在我们已经添加了所有必要的软件仓库，我们可以添加一些具体的库。通过在 `build.sbt` 文件中添加以下行，我们可以获得 Apache Commons IO 库的访问权限：

```java
    libraryDependencies += "commons-io" % "commons-io" % "2.4"
```

在更改 `build.sbt` 文件后，必须重新加载任何正在运行的 SBT 实例。在 SBT 交互式 shell 中，我们需要输入以下命令：

```java
> reload

```

这使得 SBT 能够检测构建定义文件中的任何更改，并在必要时下载额外的软件包。

不同的 Scala 库位于不同的命名空间中，称为包。为了访问特定包的内容，我们使用 `import` 语句。当我们第一次在示例中使用特定的并发库时，我们总是会显示必要的 `import` 语句集合。在随后的相同库的使用中，我们不会重复相同的 `import` 语句。

同样，我们避免在代码示例中添加包声明以保持其简短。相反，我们假设特定章节中的代码位于同名包中。例如，属于第二章，*JVM 和 Java 内存模型上的并发*的所有代码都位于 `org.learningconcurrency.ch2` 包中。该章节中展示的示例的源代码文件以以下代码开始：

```java
    package org.learningconcurrency
    package ch2
```

最后，本书讨论了并发和异步执行。许多示例在主执行停止后继续执行并发计算。为了确保这些并发计算始终完成，我们将大多数示例运行在与 SBT 本身相同的 JVM 实例中。我们在 `build.sbt` 文件中添加以下行：

```java
    fork := false
```

在示例中，当需要在一个单独的 JVM 进程中运行时，我们会指出这一点并给出明确的说明。

## 使用 Eclipse、IntelliJ IDEA 或其他 IDE

使用像 Eclipse 或 IntelliJ IDEA 这样的**集成开发环境**（**IDE**）的一个优点是，您可以自动编写、编译和运行您的 Scala 程序。在这种情况下，您不需要安装 SBT，如前所述。虽然我们建议您使用 SBT 运行示例，但您也可以使用 IDE。

在使用 IDE 运行本书中的示例时，有一个重要的注意事项：例如 Eclipse 和 IntelliJ IDEA 这样的编辑器在单独的 JVM 进程中运行程序。如前所述，某些并发计算在主执行停止后仍然继续执行。为了确保它们始终完成，你有时需要在主执行的末尾添加`sleep`语句，这会减慢主执行的速率。在本书的大多数示例中，已经为你添加了`sleep`语句，但在某些程序中，你可能需要自己添加它们。

# 本书面向的对象

本书主要面向已经学会如何编写顺序 Scala 程序的开发者，并希望学习如何编写正确的并发程序。本书假设你具备 Scala 编程语言的基本知识。在本书的整个过程中，我们努力使用 Scala 的简单特性来展示如何编写并发程序。即使你对 Scala 只有基础的了解，你也应该能够理解各种并发主题。

这并不是说这本书仅限于 Scala 开发者。无论你是否有 Java 经验，来自.NET 背景，还是一般编程语言爱好者，你都有可能在这本书中找到有见地的内容。对面向对象或函数式编程的基本理解应该是一个足够的先决条件。

最后，本书是现代并发编程（广义上）的良好入门。即使你对多线程计算或 JVM 并发模型有基本知识，你也会学到很多关于现代高级并发工具的知识。本书中的一些并发库才刚刚开始进入主流编程语言，其中一些确实是尖端技术。

# 惯例

在本书中，你会找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名如下所示：“接下来的代码行读取链接并将其分配给`BeautifulSoup`函数。”

代码块设置如下：

```java
    package org
    package object learningconcurrency {
      def log(msg: String): Unit =   
        println(s"${Thread.currentThread.getName}: $msg")
    }
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目将以粗体显示：

```java
    object ThreadsMain extends App {
      val t: Thread = Thread.currentThread      val name = t.getName
      println(s"I am the thread $name")
    }
```

任何命令行输入或输出都按照以下方式编写：

```java
$ mkdir scala-concurrency-examples

```

**新术语**和**重要词汇**以粗体显示。屏幕上看到的词，例如在菜单或对话框中，在文本中如下所示：“为了下载新模块，我们将转到**文件** | **设置** | **项目名称** | **项目解释器**。”

### 注意

警告或重要注意事项以如下方式显示：

### 小贴士

小技巧和技巧看起来像这样。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢或不喜欢的地方。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中获得最大价值的标题。要发送一般反馈，请简单地发送电子邮件至 feedback@packtpub.com，并在邮件主题中提及书籍的标题。如果您在某个主题领域有专业知识，并且有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您已经成为 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大价值。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**支持**标签上。

1.  点击**代码下载与勘误**。

1.  在**搜索**框中输入书籍的名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书的来源。

1.  点击**代码下载**。

下载文件后，请确保您使用最新版本的软件解压缩或提取文件夹：

+   WinRAR / 7-Zip for Windows

+   Zipeg / iZip / UnRarX for Mac

+   7-Zip / PeaZip for Linux

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/LearningConcurrentProgrammingInScalaSecondEdition`](https://github.com/PacktPublishing/LearningConcurrentProgrammingInScalaSecondEdition)。我们还有其他来自我们丰富图书和视频目录的代码包可供选择，网址为[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。请查看它们！

## 下载本书的彩色图像

我们还为您提供了一个包含本书中使用的截图/图表彩色图像的 PDF 文件。这些彩色图像将帮助您更好地理解输出中的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/LearningConcurrentProgrammingInScalaSecondEdition_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/LearningConcurrentProgrammingInScalaSecondEdition_ColorImages.pdf)下载此文件。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误表，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将在**勘误**部分显示。

## 盗版

在互联网上对版权材料的盗版是所有媒体中持续存在的问题。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现任何形式的非法复制我们的作品，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过版权@packtpub.com 与我们联系，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面的帮助。

## 问题

如果您在这本书的任何方面遇到问题，您可以通过 questions@packtpub.com 联系我们，我们将尽力解决问题。
