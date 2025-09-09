# 前言

Java 9 及其新特性丰富了这种语言——构建健壮软件应用中最常用的语言之一。Java 9 特别强调模块化，由 Jigsaw 项目实现。本书是掌握 Java 平台变更的一站式指南。

本书概述并解释了 Java 9 中引入的新特性以及新 API 和增强的重要性。Java 9 的一些新特性具有划时代的意义，如果你是一位经验丰富的程序员，通过实施这些新特性，你将能够使你的企业应用更加精简。你将获得关于如何应用新获得的 Java 9 知识的实用指导，以及关于 Java 平台未来发展的更多信息。本书将提高你的生产力，使你的应用更快。通过学习 Java 的最佳实践，你将成为你组织中 Java 9 的*首选*人员。

在本书结束时，你不仅将了解 Java 9 的重要概念，还将对使用这种伟大语言的编程重要方面有深入的理解。

# 本书涵盖内容

第一章，*Java 9 景观*，探讨了在 Java 9 中引入的最显著特性，包括 Jigsaw 项目、Java Shell、G1 垃圾收集和响应式编程。本章为这些主题提供了介绍，为后续章节的深入探讨奠定了基础。

第二章，*探索 Java 9*，涵盖了 Java 平台的一些变更，包括堆空间效率、内存分配、编译过程改进、类型测试、注解、自动运行时编译器测试和改进的垃圾收集。

第三章，*Java 9 语言增强*，专注于对 Java 语言的修改。这些修改影响了变量处理器、弃用警告、对 Java 7 中实现的 Project Coin 变更的改进以及导入语句处理。

第四章，*使用 Java 9 构建模块化应用*，检查了由 Project Jigsaw 指定的 Java 模块结构以及 Project Jigsaw 如何作为 Java 平台的一部分得到实现。本章还回顾了与新的模块化系统相关的 Java 平台的关键内部变更。

第五章，*迁移应用至 Java 9*，探讨了如何将 Java 8 应用迁移到 Java 9 平台。涵盖了手动和半自动迁移过程。

第六章，*Java Shell 实验*，涵盖了 Java 9 中的新命令行读取-评估-打印循环工具 JShell。内容包括关于该工具、读取-评估-打印循环概念以及与 JShell 一起使用的命令和命令行选项的信息。

第七章，*利用新的默认 G1 垃圾收集器*，深入探讨了垃圾收集以及它在 Java 9 中的处理方式。

第八章，*使用 JMH 进行微基准测试应用*，探讨了如何使用 Java Microbenchmark Harness（JMH）编写性能测试，JMH 是一个用于为 Java 虚拟机（JVM）编写基准测试的 Java 工具库。Maven 与 JMH 一起使用，以帮助说明使用新的 Java 9 平台进行微基准测试的强大功能。

第九章，*利用 ProcessHandle API*，回顾了新的类 API，这些 API 能够管理操作系统进程。

第十章，*细粒度堆栈跟踪*，涵盖了允许有效堆栈跟踪的新 API。本章包括如何访问堆栈跟踪信息的详细信息。

第十一章，*新工具和工具增强*，涵盖了 16 个被纳入 Java 9 平台的 Java 增强提案（JEPs）。这些 JEPs 覆盖了广泛的工具和 API 更新，以使使用 Java 进行开发更加容易，并为我们的 Java 应用程序提供更大的优化可能性。

第十二章，*并发增强*，涵盖了 Java 9 平台引入的并发增强。主要关注对响应式编程的支持，这是由 Flow 类 API 提供的并发增强。Java 9 中引入的其他并发增强也进行了介绍。

第十三章，*安全增强*，涵盖了针对 JDK 进行的一些涉及安全性的小改动。Java 9 平台引入的安全增强为开发者提供了更大的能力，以编写和维护比以前更安全的应用程序。

第十四章，*命令行标志*，探讨了 Java 9 中的命令行标志更改。本章涵盖的概念包括统一的 JVM 日志记录、编译器控制、诊断命令、堆分析代理、JHAT、命令行标志参数验证以及为旧平台版本编译。 

第十五章，*Java 9 最佳实践*，专注于使用 Java 9 平台提供的实用程序，包括 UTF-8 属性文件、Unicode 7.0.0、Linux/AArch64 端口、多分辨率图像和常见区域数据存储库。

第十六章，*未来方向*，概述了 Java 平台未来的发展，超出了 Java 9 的范围。这包括对 Java 10 的计划以及我们预计未来将看到的变化的具体探讨。

# 您需要为此书准备的内容

要使用此文本，您至少需要具备 Java 编程语言的基本知识。

您还需要以下软件组件：

+   Java SE 开发工具包 9 (JDK)

    +   [`www.oracle.com/technetwork/java/javase/downloads/`](http://www.oracle.com/technetwork/java/javase/downloads/)

+   集成开发环境（IDE）用于编码。以下是一些建议：

    +   Eclipse

        +   [`www.eclipse.org`](https://www.eclipse.org)

    +   IntelliJ

        +   [`www.jetbrains.com/idea/`](https://www.jetbrains.com/idea/)

    +   NetBeans

        +   [`netbeans.org`](https://netbeans.org)

# 本书面向的对象

此书面向企业开发人员和现有的 Java 开发人员。Java 基础知识是必要的。

# 规范

在此书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL 和用户输入如下所示：“在`C:\chapter8-benchmark\src\main\java\com\packt`子目录结构下是`MyBenchmark.java`文件。”

代码块设置如下：

```java
    public synchronized void protectedMethod()
    {
      . . . 
    }
```

**新术语**和**重要词汇**以粗体显示。

警告或重要提示如下所示。

技巧和窍门如下所示。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们来说很重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。

要向我们发送一般反馈，请简单地发送电子邮件至`feedback@packtpub.com`，并在邮件主题中提及书的标题。

如果您在某个主题上具有专业知识，并且您有兴趣撰写或为此书做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您充分利用您的购买。

# 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载此书的示例代码文件。如果您在其他地方购买了此书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的 SUPPORT 标签上。

1.  点击代码下载与勘误。

1.  在搜索框中输入书的名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书的来源。

1.  点击代码下载。

文件下载完成后，请确保使用最新版本的以下软件解压或提取文件夹：

+   WinRAR / 7-Zip for Windows

+   Zipeg / iZip / UnRarX for Mac

+   7-Zip / PeaZip for Linux

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Mastering-Java-9`](https://github.com/PacktPublishing/Mastering-Java-9)。我们还有其他来自我们丰富图书和视频目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

# 错误清单

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误清单，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击错误提交表单链接，并输入您的错误详细信息来报告它们。一旦您的错误清单得到验证，您的提交将被接受，错误清单将被上传到我们的网站或添加到该标题的错误清单部分。

要查看之前提交的错误清单，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在错误清单部分。

# 盗版

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过`copyright@packtpub.com`与我们联系，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面的帮助。

# 问题

如果您对本书的任何方面有问题，您可以通过`questions@packtpub.com`与我们联系，我们将尽力解决问题。
