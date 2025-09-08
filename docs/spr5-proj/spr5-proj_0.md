# 前言

Spring 使得创建 RESTful 应用、与社交服务交互、与现代化数据库通信、保护系统安全以及使代码模块化和易于测试变得简单。这本书将向您展示如何使用 Spring 5.0 的各个特性和第三方工具构建各种项目。

# 本书面向对象

这本书是为希望了解如何使用 Spring 开发复杂且灵活的应用的合格 Spring 开发者所写。您必须具备良好的 Java 编程知识，并熟悉 Spring 的基础知识。

# 本书涵盖内容

第一章，*创建一个列出世界国家和它们 GDP 的应用*，是关于启动基于 Spring 的 Web 应用开发的。我们将专注于使用 Spring MVC、Spring Data 和世界银行 API 为不同国家的一些统计数据以及 MySQL 数据库创建一个 Web 应用。应用的核心数据将来自 MySQL 附带的世界数据库样本。这个应用的 UI 将由 Angular.js 提供。我们将遵循 WAR 模型进行应用部署，并在最新版本的 Tomcat 上部署。

第二章，*构建响应式 Web 应用*，是关于纯使用 Spring 的新 WebFlux 框架构建 RESTful Web 服务应用。Spring WebFlux 是一个新的框架，它以函数式方式帮助创建响应式应用。

第三章，*Blogpress – 一个简单的博客管理系统*，是关于创建一个简单的基于 Spring Boot 的博客管理系统，该系统使用 Elasticsearch 作为数据存储。我们还将使用 Spring Security 实现用户角色管理、认证和授权。

第四章，*构建中央认证服务器*，是关于构建一个将作为认证和授权服务器的应用。我们将使用 OAuth2 协议和 LDAP 构建一个支持认证和授权的中央应用。

第五章，*使用 JHipster 查看国家和它们的 GDP 应用*，回顾了我们在第一章中开发的应用，*创建一个列出世界国家和它们 GDP 的应用*，我们将使用 JHipster 开发相同的应用。JHipster 帮助开发 Spring Boot 和 Angular.js 的生产级应用，我们将探索这个平台并了解其特性和功能。

第六章，*创建在线书店*，是关于通过以分层方式开发 Web 应用来创建一个销售书籍的在线商店。

第七章，*使用 Spring 和 Kotlin 的任务管理系统*，探讨了如何使用 Spring 框架和 Kotlin 构建任务管理系统。

# 要充分利用本书

在阅读本书之前，需要具备 Java、Git 和 Spring 框架的良好理解。虽然希望有深入的对象导向编程知识，但前两章中回顾了一些关键概念。

# 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的账户下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书名，并遵循屏幕上的说明。

文件下载完成后，请确保使用最新版本的以下软件解压或提取文件夹：

+   Windows 的 WinRAR/7-Zip

+   Mac 的 Zipeg/iZip/UnRarX

+   Linux 的 7-Zip/PeaZip

本书代码包也托管在 GitHub 上，地址为**[`github.com/PacktPublishing/Spring 5.0 Projects`](https://github.com/PacktPublishing/Spring-5.0-Projects)**。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`www.`](https://www.packtpub.com/sites/default/files/downloads/9781788390415_ColorImages.pdf)[packtpub.com/sites/default/files/downloads/9781788390415_ColorImages.pdf](https://www.packtpub.com/sites/default/files/downloads/9781788390415_ColorImages.pdf)。

# 代码实战

访问以下链接查看代码运行的视频：[`bit.ly/2ED57Ss`](http://bit.ly/2ED57Ss)

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“上一行必须在`<Host></Host>`标签之间添加。”

代码块设置如下：

```java
<depedency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
<depedency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

任何命令行输入或输出应如下编写：

```java
$ mvn package 
```

**粗体**: 表示新术语、重要单词或你在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“下载 STS，将其解压缩到你的本地文件夹中，然后打开`.exe`文件以启动 STS。启动后，创建一个具有以下属性的 Spring Starter Project，类型为 Spring Boot。”

警告或重要提示看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**: 如果你对此书的任何方面有疑问，请在邮件主题中提及书名，并通过`customercare@packtpub.com`给我们发送邮件。

**勘误**: 尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果你在这本书中发现了错误，我们将不胜感激，如果你能向我们报告这个错误。请访问[www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择你的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**: 如果你在互联网上以任何形式遇到我们作品的非法副本，如果你能提供位置地址或网站名称，我们将不胜感激。请通过`copyright@packt.com`与我们联系，并附上材料的链接。

**如果你有兴趣成为作者**: 如果你有一个你擅长的主题，并且你对撰写或为书籍做出贡献感兴趣，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦你阅读并使用了这本书，为什么不在你购买它的网站上留下评论呢？潜在的读者可以看到并使用你的无偏见意见来做出购买决定，Packt 公司可以了解你对我们的产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问[packt.com](http://www.packt.com/)。
