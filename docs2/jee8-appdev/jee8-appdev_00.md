# 前言

Java 企业版 8，Java EE 规范的最新版本，在该规范中添加了几个新特性。几个现有的 Java EE API 在本规范版本中得到了重大改进，并且一些全新的 API 被添加到 Java EE 中。本书涵盖了最流行的 Java EE 规范的最新版本，包括 JavaServer Faces（JSF）、Java 持久化 API（JPA）、企业 JavaBeans（EJB）、上下文和依赖注入（CDI）、Java 处理 JSON 的 API（JSON-P）、新的 Java JSON 绑定 API（JSON-B）、Java WebSocket API、Java 消息服务（JMS）API 2.0、Java XML Web 服务 API（JAX-WS）和 Java RESTful Web 服务 API（JAX-RS）。它还涵盖了通过全新的 Java EE 8 安全 API 保护 Java EE 应用程序。

# 本书涵盖内容

第一章，*Java EE 简介*，简要介绍了 Java EE，解释了它是如何作为一个社区努力开发的。它还澄清了一些关于 Java EE 的常见误解。

第二章，*JavaServer Faces*，涵盖了使用 JSF 开发 Web 应用程序，包括 HTML5 友好标记和 Faces Flows 等功能。

第三章，*使用 JPA 进行对象关系映射*，讨论了如何通过 Java 持久化 API（JPA）与关系数据库管理系统（RDBMS）如 Oracle 或 MySQL 交互。

第四章，*企业 JavaBeans*，解释了如何使用会话和消息驱动 bean 开发应用程序。涵盖了主要 EJB 功能，如事务管理、EJB 定时服务和安全性。

第五章，*上下文和依赖注入*，讨论了 CDI 命名的 bean、使用 CDI 的依赖注入和 CDI 限定符，以及 CDI 事件功能。

第六章，*使用 JSON-B 和 JSON-P 进行 JSON 处理*，解释了如何使用 JSON-P API 和新的 JSON-B API 生成和解析 JavaScript 对象表示法（JSON）数据。

第七章，*WebSocket*，解释了如何开发基于浏览器和服务器之间全双工通信的 Web 应用程序，而不是依赖于传统的 HTTP 请求/响应周期。

第八章，*Java 消息服务*，讨论了如何使用完全重写的 JMS 2.0 API 开发消息应用程序。

第九章，*保护 Java EE 应用程序*，涵盖了如何通过新的 Java EE 8 安全 API 保护 Java EE 应用程序。

第十章，*使用 JAX-RS 开发 RESTful Web 服务*，讨论了如何通过 Java API for RESTful Web Services 开发 RESTful Web 服务，以及如何通过全新的标准 JAX-RS 客户端 API 开发 RESTful Web 服务客户端。本章还涵盖了 Java EE 8 中引入的新特性服务器端事件。

第十一章，*使用 Java EE 开发微服务*，解释了如何通过利用 Java EE 8 API 来开发微服务。

第十二章，*使用 JAX-WS 开发 Web 服务*，解释了如何通过 Java API for XML Web Services 开发基于 SOAP 的 Web 服务。

第十三章，*Servlet 开发和部署*，解释了如何通过 Servlet API 在 Java EE 应用程序中开发服务器端功能。

附录*，配置和部署到 GlassFish*，解释了如何配置 GlassFish，以便我们可以使用它来部署我们的应用程序，以及我们可以用于将我们的应用程序部署到 GlassFish 的各种方法。

# 您需要这本书的内容

遵循这本书的材料需要以下软件：

+   Java 开发工具包（JDK）1.8 或更高版本

+   一个符合 Java EE 8 的应用服务器，如 GlassFish 5、Payara 5 或 OpenLiberty

+   需要 Maven 3 或更高版本来构建示例

+   A Java IDE，如 NetBeans、Eclipse 或 IntelliJ IDEA（可选，但推荐）

# 本书面向的对象

本书假设您熟悉 Java 语言。本书的目标市场是希望学习 Java EE 的现有 Java 开发人员，以及希望将他们的技能更新到最新 Java EE 规范的现有 Java EE 开发人员。

# 规范

在本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称将如下所示："`@Named`类注解指定此 Bean 为 CDI 命名 Bean。"

代码块将如下设置：

```java
if (!emailValidator.isValid(email)) {
  FacesMessage facesMessage = 
       new FacesMessage(htmlInputText.getLabel()
      + ": email format is not valid");
  throw new ValidatorException(facesMessage);
      }
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
@FacesValidator(value = "emailValidator")
```

任何命令行输入或输出将如下所示：

```java
/asadmin start-domain domain1
```

**新术语**和**重要词汇**将以粗体显示。屏幕上看到的单词，例如在菜单或对话框中，在文本中如下显示："点击 Next 按钮将您移动到下一屏幕。"

警告或重要注意事项如下所示。

技巧和窍门如下所示。

# 读者反馈

我们欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢或不喜欢的地方。读者反馈对我们很重要，因为它帮助我们开发您真正能从中获得最大收益的标题。

要发送给我们一般反馈，请简单地发送电子邮件至 feedback@packtpub.com，并在邮件主题中提及书籍的标题。

如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您已经是 Packt 图书的骄傲拥有者，我们有多种方式可以帮助您从您的购买中获得最大收益。

# 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载此书的示例代码文件。如果您在其他地方购买了此书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的“SUPPORT”标签上。

1.  点击“代码下载 & 勘误”。

1.  在“搜索”框中输入书籍名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书的来源。

1.  点击“代码下载”。

文件下载完成后，请确保您使用最新版本的以下软件解压或提取文件夹：

+   适用于 Windows 的 WinRAR / 7-Zip

+   适用于 Mac 的 Zipeg / iZip / UnRarX

+   适用于 Linux 的 7-Zip / PeaZip

书籍的代码包也托管在 GitHub 上[`github.com/PacktPublishing/Java-EE-8-Application-Development`](https://github.com/PacktPublishing/Java-EE-8-Application-Development)。我们还有其他来自我们丰富图书和视频目录的代码包可供选择，请访问[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。查看它们吧！

# 勘误

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现了错误——可能是文本或代码中的错误——如果您能向我们报告，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进此书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击“勘误提交表单”链接，并输入您的勘误详情来报告。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的“勘误”部分下的现有勘误列表中。

要查看之前提交的勘误表，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)并在搜索框中输入书籍名称。所需信息将在“勘误”部分显示。

# 盗版

在互联网上侵犯版权材料是一个跨所有媒体持续存在的问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 copyright@packtpub.com 与我们联系，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和我们提供有价值内容的能力方面所提供的帮助。

# 问题

如果您对本书的任何方面有问题，您可以通过 questions@packtpub.com 与我们联系，我们将尽力解决问题。
