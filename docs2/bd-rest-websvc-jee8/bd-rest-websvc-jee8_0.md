# 前言

Java 企业版是领先的 Java 企业级开发应用程序编程平台之一。随着 Java EE 8 的最终发布和第一个应用程序服务器的可用，现在是更深入地了解如何使用最新的 API 新增和改进开发现代和轻量级网络服务的时候了。

这本书是一本全面的指南，展示了如何使用最新的 Java EE 8 API 开发最先进的 RESTful 网络服务。我们首先概述 Java EE 8 以及最新的 API 新增和改进。然后，你将实现、构建和打包你的第一个工作网络服务作为本书剩余部分的原型。它深入探讨了使用 JAX-RS 实现同步 RESTful 网络服务和客户端的细节。接下来，你将了解使用 JSON-B 1.0 和 JSON-P 1.1 API 进行数据绑定和内容序列化的具体细节。你还将学习如何利用服务器和客户端异步 API 的强大功能，以及如何使用**服务器端事件**（**SSEs**）进行`PUSH`通信。最后一章涵盖了某些高级网络服务主题，例如验证、JWT 安全和诊断网络服务。

在本书结束时，你将彻底理解用于轻量级网络服务开发的 Java EE 8 API。此外，你将实现几个工作网络服务，为你提供所需的实际经验。

# 本书面向对象

本书旨在为想要学习如何使用最新的 Java EE 8 API 实现网络服务的 Java 开发者编写。不需要 Java EE 8 的先验知识；然而，具有先前 Java EE 版本的实践经验将是有益的。

# 为了充分利用这本书

为了充分利用这本书，你需要以下内容：

+   你需要具备基本的编程技能，并且需要一些 Java 知识

+   你需要一个配备现代操作系统的计算机，例如 Windows 10、macOS 或 Linux

+   你需要一个有效的 Java 8 语言安装；我们将使用 Maven 3.5.x 作为我们的构建工具

+   我们将使用 Payara Server 5.x 作为我们的 Java 8 应用程序服务器

+   你需要 Windows、macOS 或 Linux 上的 Docker

+   你需要一个支持 Java EE 8 的 IDE，例如 IntelliJ IDEA 2017.3，以及一个 REST 客户端，例如 Postman 或 SoapUI

# 下载示例代码文件

你可以从[www.packtpub.com](http://www.packtpub.com)的账户下载这本书的示例代码文件。如果你在其他地方购买了这本书，你可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给你。

你可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择“支持”标签。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

下载文件后，请确保使用最新版本的以下软件解压缩或提取文件夹：

+   Windows 上的 WinRAR/7-Zip

+   Mac 上的 Zipeg/iZip/UnRarX

+   Linux 上的 7-Zip/PeaZip

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Building-RESTful-Web-Services-with-Java-EE-8`](https://github.com/PacktPublishing/Building-RESTful-Web-Services-with-Java-EE-8)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富的书籍和视频目录的代码包可供使用，请访问**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。查看它们吧！

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“在先前的`Dockerfile`中，我们提到正在使用`payara/server-full`。”

代码块应如下设置：

```java
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
    @PUT
    @Path("/{isbn}")
    public Response update(@PathParam("isbn") String isbn, Book book) {
        if (!Objects.equals(isbn, book.getIsbn())) {
```

任何命令行输入或输出都应如下编写：

```java
>docker build -t hello-javaee8:1.0 .
```

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“让我们检查我们的浏览器，你应该看到`Hello World.`消息。”

警告或重要注意事项如下所示。

技巧和窍门如下所示。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：请通过`feedback@packtpub.com`发送电子邮件，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过`questions@packtpub.com`发送电子邮件给我们。

**勘误表**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将非常感激您能向我们报告。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式遇到我们作品的非法副本，我们将非常感激您能提供位置地址或网站名称。请通过`copyright@packtpub.com`与我们联系，并提供材料的链接。

**如果您想成为作者**：如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下您的评价。一旦您阅读并使用了这本书，为何不在购买它的网站上留下评价呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 公司可以了解您对我们产品的看法，并且我们的作者可以查看他们对书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/)。
