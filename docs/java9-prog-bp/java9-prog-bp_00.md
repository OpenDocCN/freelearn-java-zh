# 前言

世界已经等待 Java 9 很长时间了。更具体地说，我们一直在等待 Java 平台模块系统，而 Java 9 终于要推出了。如果一切顺利，我们最终将拥有真正的隔离，这可能会带来更小的 JDK 和更稳定的应用程序。当然，Java 9 提供的不仅仅是这些；在这个版本中有大量的重大变化，但这无疑是最令人兴奋的。话虽如此，这本书并不是一本关于模块系统的书。有很多优秀的资源可以让您深入了解 Java 平台模块系统及其许多影响。不过，这本书更多地是一个对 Java 9 的实际观察。与其讨论发布的细枝末节，尽管那样也很令人满意，但在接下来的几百页中，我们将看到最近 JDK 发布中的所有重大变化--特别是 Java 9--如何以实际的方式应用。

当我们完成时，您将拥有十个不同的项目，涵盖了许多问题领域，您可以从中获取可用的示例，以解决您自己独特的挑战。

# 本书内容

第一章，《介绍》，快速概述了 Java 9 的新功能，并介绍了 Java 7 和 8 的一些主要功能，为我们后面章节的使用做好铺垫。

第二章，《在 Java 中管理进程》，构建了一个简单的进程管理应用程序（类似于 Unix 的 top 命令），我们将探索 Java 9 中新的操作系统进程管理 API 的变化。

第三章，《重复文件查找器》，演示了在应用程序中使用新的文件 I/O API，包括命令行和 GUI，用于搜索和识别重复文件。大量使用了文件哈希、流和 JavaFX 等技术。

第四章，《日期计算器》，展示了一个库和命令行工具来执行日期计算。我们将大量使用 Java 8 的日期/时间 API。

第五章，《Sunago-社交媒体聚合器》，展示了如何与第三方系统集成以构建一个聚合器。我们将使用 REST API、JavaFX 和可插拔应用程序架构。

第六章，《Sunago-Android 移植》，让我们回到了第五章中的应用程序，《Sunago-社交媒体聚合器》。

第七章，《使用 MailFilter 管理电子邮件和垃圾邮件》，构建了一个邮件过滤应用程序，解释了各种电子邮件协议的工作原理，然后演示了如何使用标准的 Java 电子邮件 API--JavaMail 与电子邮件进行交互。

第八章，《使用 PhotoBeans 管理照片》，当我们使用 NetBeans Rich Client Platform 构建一个照片管理应用程序时，我们将走向完全不同的方向。

第九章，《使用 Monumentum 记笔记》，又开辟了一个新方向。在这一章中，我们构建了一个提供基于 Web 的记笔记的应用程序和微服务，类似于一些流行的商业产品。

第十章，《无服务器 Java》，将我们带入云端，我们将在 Java 中构建一个函数作为服务系统，用于发送基于电子邮件和短信的通知。

第十一章，《DeskDroid-用于 Android 手机的桌面客户端》，演示了一个与 Android 设备交互的桌面客户端的简单方法，我们将构建一个应用程序，从桌面查看并发送短信。

第十二章，*接下来是什么？*，讨论了 Java 的未来可能会带来什么，并且还涉及了 Java 在 JVM 上的两个最近的挑战者-Ceylon 和 Kotlin。

# 您需要为这本书做好准备

您需要 Java 开发工具包（JDK）9、NetBeans 8.2 或更新版本，以及 Maven 3.0 或更新版本。一些章节将需要额外的软件，包括 Gluon 的 Scene Builder 和 Android Studio。

# 这本书是为谁准备的

这本书适用于初学者到中级开发人员，他们有兴趣在实际示例中看到新的和多样化的 API 和编程技术。不需要深入了解 Java，但假定您对语言及其生态系统、构建工具等有基本了解。

# 约定

在本书中，您会发现一些区分不同信息种类的文本样式。以下是这些样式的一些示例及其含义的解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下：“Java 架构师引入了一个新文件，`module-info.java`，类似于现有的 `package-info.java` 文件，位于模块的根目录，例如在 `src/main/java/module-info.java`。”

代码块设置如下：

```java
    module com.steeplesoft.foo.intro {
      requires com.steeplesoft.bar;
      exports com.steeplesoft.foo.intro.model;
      exports com.steeplesoft.foo.intro.api;
    }

```

任何命令行输入或输出都以以下方式编写：

```java
$ mvn -Puber install

```

**新术语**和**重要单词**以粗体显示。例如，屏幕上看到的单词，例如菜单或对话框中的单词，会出现在文本中，如下所示：“在新项目窗口中，我们选择 Maven 然后 NetBeans 应用程序。”

警告或重要说明会出现在这样的地方。

提示和技巧会出现如下。

# 读者反馈

我们始终欢迎读者的反馈。让我们知道您对这本书的看法-您喜欢或不喜欢什么。读者的反馈对我们很重要，因为它有助于我们开发出您真正受益的标题。

要向我们发送一般反馈，只需发送电子邮件至 `feedback@packtpub.com`，并在主题中提及书名。

如果您在某个专题上有专长，并且有兴趣撰写或为一本书做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然您已经是 Packt 图书的自豪所有者，我们有一些东西可以帮助您充分利用您的购买。

# 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户中下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，文件将直接发送到您的电子邮件。

您可以按照以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的“支持”选项卡上。

1.  点击“代码下载和勘误”。

1.  在搜索框中输入书名。

1.  选择您要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书的地点。

1.  点击“代码下载”。

下载文件后，请确保使用以下最新版本的软件解压缩文件夹：

+   WinRAR / 7-Zip 适用于 Windows

+   Zipeg / iZip / UnRarX 适用于 Mac

+   7-Zip / PeaZip 适用于 Linux

该书的代码包也托管在 GitHub 上[`github.com/PacktPublishing/Java-9-Programming-Blueprints`](https://github.com/PacktPublishing/Java-9-Programming-Blueprints)。我们还有其他丰富的图书和视频代码包可供下载[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。快去看看吧！

# 下载本书的彩色图片

我们还为您提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。彩色图像将帮助您更好地理解输出中的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/Java9ProgrammingBlueprints_ColorImages`](https://www.packtpub.com/sites/default/files/downloads/Java9ProgrammingBlueprints_ColorImages)下载此文件。

# 勘误

尽管我们已经非常小心确保内容的准确性，但错误是难免的。如果您在我们的书中发现错误，也许是文本或代码中的错误，我们将不胜感激，如果您能向我们报告。通过这样做，您可以帮助其他读者避免挫折，并帮助我们改进本书的后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)报告，选择您的书，点击勘误提交表链接，并输入您的勘误详情。一旦您的勘误经过验证，您的提交将被接受，并且勘误将被上传到我们的网站或添加到该标题的勘误部分下的任何现有勘误列表中。

查看先前提交的勘误表，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)并在搜索框中输入书名。所需信息将显示在勘误部分下方。

# 问题

盗版

请通过`copyright@packtpub.com`与我们联系，并附上涉嫌盗版材料的链接。

我们感谢您在保护我们的作者和我们为您提供有价值内容的能力方面的帮助。

# 在互联网上盗版受版权保护的材料是一个持续存在的问题，涉及所有媒体。在 Packt，我们非常重视版权和许可的保护。如果您在互联网上以任何形式发现我们作品的非法副本，请立即向我们提供位置地址或网站名称，以便我们采取补救措施。

如果您对本书的任何方面有问题，可以通过`questions@packtpub.com`与我们联系，我们将尽力解决问题。
