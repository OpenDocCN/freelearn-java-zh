# 前言

本书介绍了使用 Spring Boot 和 Spring Cloud 构建生产就绪的微服务。五年前，当我开始探索微服务时，我一直在寻找这样的书。

在我学会并精通用于开发、测试、部署和管理协作微服务生态的开源软件之后，这本书才得以编写。

本书主要涵盖了 Spring Boot、Spring Cloud、Docker、Kubernetes、Istio、EFK 堆栈、Prometheus 和 Grafana。这些开源工具各自都很好用，但理解如何将它们以有利的方式结合起来可能会有挑战性。在某些领域，它们相互补充，但在其他领域，它们重叠，对于特定情况选择哪一个并不明显。

这是一本实用书籍，详细介绍了如何逐步使用这些开源工具。五年前，当我开始学习微服务时，我一直在寻找这样的书籍，但现在它涵盖了这些开源工具的最新版本。

# 本书面向人群

这本书面向希望学习如何将现有单体拆分为微服务并在本地或云端部署的 Java 和 Spring 开发者及架构师，使用 Kubernetes 作为容器编排器，Istio 作为服务网格。无需对微服务架构有任何了解即可开始本书的学习。

# 为了最大化本书的收益

需要对 Java 8 有深入了解，以及对 Spring Framework 有基本知识。此外，对分布式系统的挑战有一个大致的了解，以及对在生产环境中运行自己代码的一些经验，也将有益于学习。

# 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)您的账户上下载本书的示例代码文件。如果您在其他地方购买了此书，您可以访问[www.packtpub.com/support](https://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  登录或注册[www.packt.com](http://www.packt.com)。

1.  选择“支持”选项卡。

1.  点击“代码下载”。

1.  在搜索框中输入书籍名称，然后按照屏幕上的指示操作。

文件下载完成后，请确保使用最新版本解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，地址为[`github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud`](https://github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud)。如有代码更新，它将在现有的 GitHub 仓库中更新。

我们还有其他丰富的书籍和视频目录中的代码包，托管在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。去看看它们！

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含了本书中使用的屏幕截图/图表的颜色图像。您可以通过以下链接下载： [`static.packt-cdn.com/downloads/9781789613476_ColorImages.pdf`](https://static.packt-cdn.com/downloads/9781789613476_ColorImages.pdf)。

# 代码在行动

若要查看代码的执行情况，请访问以下链接： [`bit.ly/2kn7mSp`](http://bit.ly/2kn7mSp)。

# 本书中使用了一些约定。

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码单词，数据库表名，文件夹名，文件名，文件扩展名，路径名，假 URL，用户输入和 Twitter 处理。这是一个示例："要使用本地文件系统，配置服务器需要启动带有 Spring 配置文件`native`的特性"。

代码块如下所示：

```java
management.endpoint.health.show-details: "ALWAYS"
management.endpoints.web.exposure.include: "*"

logging.level.root: info
```

当我们希望引起你对代码块中的某个特定部分的关注时，相关的行或项目会被设置为粗体：

```java
   backend:
    serviceName: auth-server
    servicePort: 80
 - path: /product-composite
```

任何命令行输入或输出都会如下书写：

```java
brew install kubectl
```

**粗体**：表示一个新术语，一个重要的单词，或者你在屏幕上看到的单词。例如，菜单或对话框中的单词会在文本中像这样出现。这是一个示例："正如前一个屏幕截图所示，Chrome 报告：此证书有效！"

警告或重要注释会像这样出现。

提示和技巧会像这样出现。

# 联系我们

读者反馈始终受欢迎。

**一般反馈**：如果你对本书的任何方面有疑问，请在消息的主题中提到书名，并通过 `customercare@packtpub.com` 向我们发送电子邮件。

**勘误**：虽然我们已经尽一切努力确保我们的内容的准确性，但是错误确实存在。如果您在这本书中发现了错误，我们将非常感激如果您能向我们报告。请访问 [www.packtpub.com/support/errata](https://www.packtpub.com/support/errata)，选择您的书籍，点击勘误表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式遇到我们作品的非法副本，我们将非常感激如果您能提供其位置地址或网站名称。请通过 `copyright@packt.com` 与我们联系，并附上材料的链接。

**如果你想成为作者**：如果你对你的某个主题有专业知识，并且你想写书或者为某个书做贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用了这本书，为什么不在这本书购买的网站上留下评论呢？潜在的读者可以看到并使用您的客观意见来做出购买决策，我们 Pactt 出版社可以了解您对我们产品的看法，我们的作者可以看到您对他们书籍的反馈。谢谢！

关于 Pactt 出版社的更多信息，请访问 [packt.com](http://www.packt.com/)。
