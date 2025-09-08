# 前言

依赖注入是一种设计模式，它允许我们移除硬编码的依赖关系，使我们的应用程序松耦合、可扩展和可维护。我们可以通过将依赖解析从编译时移动到运行时来实现依赖注入。

这本书将为您提供一个一站式指南，帮助您使用 Java 9 的最新特性以及 Spring 5 和 Google Guice 等框架编写松耦合的代码。

# 这本书面向的对象

这本书是为希望了解如何在他们的应用程序中实现依赖注入的 Java 开发者准备的。假设您对 Spring 和 Guice 框架以及 Java 编程有一定的了解。

# 这本书涵盖的内容

第一章，*为什么需要依赖注入？*，为您详细介绍了各种概念，如依赖倒置原则（DIP）、控制反转（IoC）和依赖注入（DI）。它还讨论了 DI 在实际应用中的常见用例。

第二章，*Java 9 中的依赖注入*，使您熟悉 Java 9 特性和其模块化框架，并解释了如何使用服务加载器概念实现 DI。

第三章，*使用 Spring 进行依赖注入*，教您如何在 Spring 框架中管理依赖注入。它还描述了使用 Spring 实现 DI 的另一种方法。

第四章，*使用 Google Guice 进行依赖注入*，讨论了 Guice 及其依赖机制，并教我们 Guice 框架的依赖绑定和多种注入方法。

第五章，*Scopes*，介绍了 Spring 和 Guice 框架中定义的不同作用域。

第六章，*面向切面编程和拦截器*，展示了面向切面编程（AOP）的目的，以及它是如何通过将重复代码从应用程序中隔离出来，并使用 Spring 框架动态地将其插入来解决不同的设计问题的。

第七章，*IoC 模式和最佳实践*，概述了各种可用于实现 IoC 的设计模式。除此之外，您还将了解在注入 DI 时应该遵循的最佳实践和反模式。

# 要充分利用这本书

1.  如果您了解 Java、Spring 和 Guice 框架，那将很有帮助。这将有助于您理解依赖注入

1.  在开始之前，我们假设您已经在系统上安装了 Java 9 和 Maven

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择 SUPPORT 标签。

1.  点击代码下载与勘误。

1.  在搜索框中输入书名，并遵循屏幕上的说明。

下载文件后，请确保使用最新版本的以下软件解压缩或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Java-9-Dependency-Injection`](https://github.com/PacktPublishing/Java-9-Dependency-Injection)。我们还有其他来自我们丰富图书和视频目录的代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**找到。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/Java9DependencyInjection_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/Java9DependencyInjection_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“将下载的`WebStorm-10*.dmg`磁盘映像文件作为系统上的另一个磁盘挂载。”

代码块设置如下：

```java
module javaIntroduction {
}
```

任何命令行输入或输出都应如下编写：

```java
$ mkdir css
$ cd css
```

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中如下所示。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要说明如下所示。

技巧和窍门如下所示。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：请发送电子邮件至`feedback@packtpub.com`，并在邮件主题中提及书名。如果您对本书的任何方面有疑问，请通过`questions@packtpub.com`发送电子邮件给我们。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将非常感激您能向我们报告。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书，点击勘误提交表单链接，并输入详细信息。

**盗版**: 如果你在互联网上以任何形式遇到我们作品的非法副本，如果你能提供位置地址或网站名称，我们将不胜感激。请通过`copyright@packtpub.com`与我们联系，并附上材料的链接。

**如果你有兴趣成为作者**：如果你在某个领域有专业知识，并且你对撰写或为书籍做出贡献感兴趣，请访问 [authors.packtpub.com](http://authors.packtpub.com/).

# 评论

请留下评论。一旦你阅读并使用了这本书，为何不在你购买它的网站上留下评论呢？潜在的读者可以查看并使用你的客观意见来做出购买决定，Packt 公司可以了解你对我们的产品有何看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

想了解更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/).
