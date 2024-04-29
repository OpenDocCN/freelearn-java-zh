# 前言

如今的企业发展如此迅速，它们正在利用云的弹性来提供一个构建和部署高度可扩展应用程序的平台。这意味着开发人员现在面临着构建原生于云的应用程序的挑战。为此，他们需要了解他们正在编码的环境、工具和资源。

本书首先解释了云采用的驱动因素，并向您展示了云部署与在标准数据中心上进行常规应用程序部署的不同之处。您将了解到特定于在云中运行的应用程序的设计模式，并了解如何使用 REST API 在 Java Spring 中构建微服务。

然后，您将深入了解构建、测试和部署应用程序的生命周期，并最大程度地自动化以减少部署周期时间。逐渐地，您将开始配置 AWS 和 Azure 平台，并使用它们的 API 来部署您的应用程序。最后，您将了解 API 设计问题及其最佳实践。您还将学习如何将现有的单片应用程序迁移到分布式云原生应用程序。

到最后，您将了解如何构建和监控可扩展、有弹性和健壮的云原生应用程序，始终可用且容错。

# 本书适合对象

希望构建面向云端部署的有弹性、健壮和可扩展应用程序的 Java 开发人员会发现本书很有帮助。对 Java、Spring、Web 编程和公共云提供商（AWS 和 Azure）的一些了解应该足以让您顺利阅读本书。

# 为了充分利用本书

1.  本书从介绍开始，然后逐步构建一个简单的服务，通过各章节逐步展开。因此，读者将受益于按照书的流程进行阅读，除非他们正在寻找特定的主题。

1.  下载代码并运行它总是很诱人的。然而，特别是在最初的章节中，当您输入代码时，您将获得更多的好处。本书是以这样一种方式编写的，重要的概念和代码都在章节中，因此您不需要回头查看源代码。

1.  话虽如此，确实尝试运行代码示例。这样可以使原则更具体，更容易理解。

1.  我希望您已经投资购买了一台良好的台式机/笔记本电脑，因为您将在您的设备上运行容器和虚拟机，这需要资源，所以拥有一台强大的设备是很好的起点。

1.  请参考章节中提到的文档链接，以扩展书中讨论的框架和技术的知识。

1.  云是一个技术发展非常迅速的领域。因此，本书强调概念，并通过代码进行演示。例如，CQRS 作为一个概念很重要，因此我们展示了在 MongoDB 和 Elasticsearch 上的实现。然而，您可以尝试在任何其他一组数据库上使用这种模式。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便直接通过电子邮件接收文件。

您可以按照以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择“支持”选项卡。

1.  单击“代码下载和勘误”。

1.  在搜索框中输入书名，并按照屏幕上的说明操作。

下载文件后，请确保使用最新版本的以下软件解压或提取文件：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Cloud-Native-Applications-in-Java`](https://github.com/PacktPublishing/Cloud-Native-Applications-in-Java)。我们还有其他书籍和视频的代码包，都可以在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。快去看看吧！

# 下载彩色图像

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/CloudNativeApplicationsinJava_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/CloudNativeApplicationsinJava_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。以下是一个例子：“`CrudRepository` 接口带有一组默认方法来实现最常见的操作。”

代码块设置如下：

```java
-- Adding a few initial products
insert into product(id, name, cat_Id) values (1, 'Apples', 1) 
insert into product(id, name, cat_Id) values (2, 'Oranges', 1) 
insert into product(id, name, cat_Id) values (3, 'Bananas', 1) 
insert into product(id, name, cat_Id) values (4, 'Carrot', 2) 
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```java
public class Product implements Serializable {
```

任何命令行输入或输出都以以下形式书写：

```java
mongoimport --db masterdb --collection product --drop --file D:datamongoscriptsproducts.json 
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会以这种形式出现。以下是一个例子：“接下来，我们点击左侧的 Deployment credentials 链接。”

警告或重要说明会以这种形式出现。

技巧和窍门会以这种形式出现。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：发送电子邮件至`feedback@packtpub.com`，并在主题中提及书名。如果您对本书的任何方面有疑问，请发送电子邮件至`questions@packtpub.com`。

**勘误表**：尽管我们已经尽最大努力确保内容的准确性，但错误是难免的。如果您在本书中发现了错误，我们将不胜感激地接受您的报告。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何非法副本，请向我们提供位置地址或网站名称。请通过`copyright@packtpub.com`与我们联系，并提供材料链接。

如果您有兴趣成为作者：如果您在某个专业领域有专长，并且有兴趣撰写或为一本书做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下您的评论。当您阅读并使用了本书之后，为什么不在购买书籍的网站上留下您的评论呢？潜在的读者可以看到并使用您的客观意见来做出购买决定。此外，我们在 Packt 可以了解您对我们产品的看法，我们的作者也可以看到您对他们书籍的反馈。谢谢！

有关 Packt 的更多信息，请访问[packtpub.com](https://www.packtpub.com/)。
