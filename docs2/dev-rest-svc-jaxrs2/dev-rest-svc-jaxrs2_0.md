# 前言

几年来，我们见证了从主机到 x86 农场、从重型方法到轻量级敏捷方法、从桌面和厚客户端到瘦、丰富和高度可用的 Web 应用程序以及无处不在的计算的几次革命和范式转变。

随着技术领域向更小、更便携、更轻便的设备发展，以及这些设备在日常活动中被广泛使用，将计算从客户端机器推送到后端的需求变得更加突出。这也带来了在服务器和客户端之间开发具有近似实时或实时事件和数据传播的应用程序的机会和挑战；这正是 HTML 5 为开发者提供标准、API 和灵活性，以在 Web 应用程序中实现与厚客户端应用程序相同结果的地方。

客户端与服务器之间的通信在数量、内容、互操作性和可扩展性等方面已成为最基本的问题。XML 时代、长时间等待请求、单一浏览器和单一设备兼容性已经结束；取而代之的是设备、多个客户端和浏览器的时代，从小型设备（仅能通过 HTTP 处理文本）到巨型规模机器（几乎可以处理任何类型的内容）都有。因此，生成内容、接受内容以及能够在旧协议和新协议之间切换的能力已成为显而易见的必需。

Java EE 7 更加注重这些新兴（和主导）需求；支持 HTML5、更多异步通信/调用能力组件，以及支持 JSON 作为数据格式之一，这些都有助于开发者解决技术需求，并为开发者提供充足的时间来处理系统业务需求的实现。

本书试图为热衷于技术的读者提供一个概述，介绍 Java EE 在一般意义上以及 Java EE 7 在特定意义上作为技术平台，为开发基于 HTML5 的轻量级、交互式应用程序，并可在任何 Java EE 兼容容器中部署提供支持。

# 本书涵盖的内容

第一章, *使用 JAX-RS 构建 RESTful Web 服务*，从构建 RESTful Web 服务的基本概念开始，涵盖了 JAX-RS 2.0 API，详细介绍了不同的注解、提供者、`MessageBodyReader`、`MessageBodyWriter`、客户端 API 和 JAX-RS 2.0 中的 Bean Validation 支持。

第二章，*WebSockets 和服务器端事件*，讨论了向客户端发送近实时更新的不同编程模型。它还涵盖了 WebSockets 和服务器端事件，即 WebSockets 和服务器端事件的 JavaScript 和 Java API。本章比较和对比了 WebSockets 和服务器端事件，并展示了 WebSockets 在减少不必要的网络流量和提高性能方面的优势。

第三章，*详细理解 WebSockets 和服务器端事件*，涵盖了 Java EE 7 API 用于 WebSockets、编码器和解码器、客户端 API，以及如何使用 blob 和`ArrayBuffers`通过 WebSockets 发送不同类型的消息。它教授如何确保基于 WebSockets 的应用程序的安全性。它概述了基于 WebSockets 和服务器端事件的最佳实践。

第四章，*JSON 和异步处理*，涵盖了 Java EE 7 JSON-P API 用于解析和操作 JSON 数据。它还讨论了 Servlet 3.1 规范中引入的新 NIO API。它教授如何使用 JAX-RS 2.0 API 进行异步请求处理以提高可伸缩性。

第五章，*通过示例学习 RESTful Web 服务*，涵盖了两个实际的 RESTful Web 服务示例。它涵盖了一个基于 Twitter 搜索 API 的事件通知示例，服务器如何根据事件发生的情况将数据推送到客户端。一个库应用程序将上述章节中涵盖的不同技术联系起来。

# 您需要为这本书准备什么

要能够构建和运行本书提供的示例，您将需要：

1.  Apache Maven 3.0 及以上版本。Maven 用于构建示例。您可以从[`maven.apache.org/download.cgi`](http://maven.apache.org/download.cgi)下载 Apache Maven。

1.  GlassFish Server Open Source Edition v4.0 是免费的、社区支持的提供 Java EE 7 规范实现的 Application Server。您可以从[`dlc.sun.com.edgesuite.net/glassfish/4.0/promoted/`](http://dlc.sun.com.edgesuite.net/glassfish/4.0/promoted/)下载 GlassFish Server。

# 这本书适合谁阅读

这本书是针对熟悉 Java EE 并渴望了解 Java EE 7 中引入的新 HTML5 相关功能以提高生产力的应用开发者理想的阅读材料。为了充分利用这本书，您需要熟悉 Java EE 并具备一些使用 GlassFish 应用服务器的基本知识。

# 习惯用法

在这本书中，您将找到多种文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称如下所示：“发送给 JAX-RS 资源的请求是一个以`app/library/book/`为目标 URI 的`POST`请求。”

代码块如下所示：

```java
@GET
@Path("browse")
public List<Book> browseCollection() {
  return bookService.getBooks();    }
```

当我们希望将您的注意力引到代码块的一个特定部分时，相关的行或项目将以粗体显示：

```java
@GET
@Path("browse")
public List<Book> browseCollection() {
  return bookService.getBooks();    }
```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下：“当用户点击 HTML 页面上的**保持**按钮时”。

### 注意

警告或重要注意事项如下所示。

### 提示

技巧和窍门如下所示。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者的反馈对我们开发您真正能从中受益的标题非常重要。

要向我们发送一般反馈，只需发送电子邮件到`<feedback@packtpub.com>`，并在邮件主题中提及书籍标题。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您已经成为 Packt 书籍的骄傲所有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户中下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)来报告它们，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站，或添加到该标题的勘误部分下的现有勘误列表中。您可以通过选择您的标题从[`www.packtpub.com/support`](http://www.packtpub.com/support)查看任何现有勘误。

## 盗版

在互联网上侵犯版权材料是一个跨所有媒体持续存在的问题。在 Packt，我们非常重视我们版权和许可证的保护。如果你在网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供疑似盗版材料的链接。

我们感谢你在保护我们作者和提供有价值内容的能力方面的帮助。

## 问题

如果你在本书的任何方面遇到问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决。
