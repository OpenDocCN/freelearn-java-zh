# 前言

Jakarta EE 提供了简化云应用程序以及更多传统 Web 应用程序（包括服务器端企业应用程序）开发的函数。本书涵盖了最受欢迎的 Jakarta EE 规范的最新版本，包括 Jakarta Faces、Jakarta Persistence、Jakarta Enterprise JavaBeans、Contexts and Dependency Injection（CDI）、Jakarta JSON Processing、Jakarta JSON Binding、Jakarta WebSocket、Jakarta Messaging、Jakarta Enterprise Web Services、Jakarta REST，以及通过 Jakarta EE Security 保护 Jakarta EE 应用程序的覆盖。

# 这本书面向的对象

本书是为已经熟悉 Java 语言的读者设计的。其目标受众包括希望学习 Jakarta EE 的现有 Java 开发人员，以及希望将他们的技能更新到 Jakarta EE 的现有 Java EE 开发人员。

# 本书涵盖的内容

*第一章*, *Jakarta EE 简介*，简要介绍了 Jakarta EE，包括它是如何作为一个社区努力开发的，同时也澄清了一些关于 Jakarta EE 的常见误解。

*第二章*, *上下文和依赖注入*，包括 CDI 命名 bean 的覆盖，使用 CDI 进行依赖注入和 CDI 限定符，以及 CDI 事件功能。

*第三章*, *Jakarta RESTful Web Services*，讨论了如何使用 Jakarta REST 开发 RESTful Web 服务，以及如何通过 Jakarta REST 客户端 API 开发 RESTful Web 服务客户端。本章还涵盖了如何通过服务器端事件向 RESTful Web 服务客户端发送自动更新。

*第四章*, *JSON 处理和 JSON 绑定*，介绍了如何使用 Jakarta JSON Processing 和 Jakarta JSON Binding 生成和解析 JavaScript 对象表示法（JSON）数据。

*第五章*, *使用 Jakarta EE 进行微服务开发*，解释了如何利用 Jakarta EE 功能开发基于微服务的架构。

*第六章*, *Jakarta Faces*，涵盖了使用 Jakarta Faces 开发 Web 应用程序的内容。

*第七章*, *额外的 Jakarta Faces 功能*，涵盖了额外的 Jakarta Faces 功能，例如 HTML5 友好的标记、WebSocket 支持和 Faces Flows。

*第八章*, *使用 Jakarta Persistence 进行对象关系映射*，讨论了如何通过 Jakarta Persistence 开发与关系数据库管理系统（RDBMS）如 Oracle 或 MySQL 交互的代码。

*第九章*, *WebSocket*，解释了如何开发具有浏览器和服务器之间全双工通信功能的基于 Web 的应用程序。

*第十章*，*确保 Jakarta EE 应用程序安全*，介绍了如何使用 Jakarta Security 确保 Jakarta EE 应用程序的安全。

*第十一章*，*Servlet 开发和部署*，解释了如何使用 Jakarta Servlet 在 Java EE 应用程序中开发服务器端功能。

*第十二章*，*Jakarta 企业 Bean*，解释了如何使用会话和消息驱动 Bean 开发应用程序。涵盖了企业 Bean 功能，如事务管理、定时服务和安全。

*第十三章*，*Jakarta Messaging*，讨论了如何使用 Jakarta Messaging 开发消息应用程序。

*第十四章*，*使用 Jakarta XML Web Services 开发 Web 服务*，解释了如何使用 Jakarta Enterprise Web Services 开发基于 SOAP 的 Web 服务。

*第十五章*，*整合一切*，解释了如何开发集成多个 Jakarta EE API 的应用程序。

# 为了充分利用本书

为了编译和执行本书中的示例，需要一些必需和推荐的工具。

| **必需或推荐软件** | **操作系统要求** |
| --- | --- |
| 需要 Java 17 或更高版本 | Windows、macOS 或 Linux |
| Apache Maven 3.6 或更高版本必需 | Windows、macOS 或 Linux |
| 推荐使用 Java IDE 如 Eclipse IDE、IntelliJ IDEA 或 NetBeans | Windows、macOS 或 Linux |
| 需要符合 Jakarta EE 10 规范的实现，如 GlassFish、WildFly 或 Apache TomEE | Windows、macOS 或 Linux |

## 技术要求

为了编译和构建本书中的示例，需要以下工具：

+   一个较新的 Java 开发工具包，本书中的示例使用 OpenJDK 17 构建。

+   Apache Maven 3.6 或更高版本

+   推荐使用 Java 集成开发环境（IDE）如 Apache NetBeans、Eclipse IDE 或 IntelliJ IDEA，但不是必需的（本书中的示例使用 Apache NetBeans 开发，但鼓励读者使用他们偏好的 Java IDE）

+   一个符合 Jakarta EE 10 规范的运行时（本书中的示例使用 Eclipse GlassFish 部署，但任何符合 Jakarta EE 10 规范的运行时都将工作）

**如果您使用的是本书的电子版，我们建议您亲自输入代码或从本书的 GitHub 仓库（下一节中提供链接）获取代码。这样做将有助于您避免与代码复制和粘贴相关的任何潜在错误。**

# 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件[`github.com/PacktPublishing/Jakarta-EE-Application-Development`](https://github.com/PacktPublishing/Jakarta-EE-Application-Development)。如果代码有更新，它将在 GitHub 仓库中更新。

我们还有其他来自我们丰富的书籍和视频目录的代码包可供在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上获取。查看它们吧！

# 使用的约定

本书使用了多种文本约定。

`文本中的代码`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“如示例所示，我们通过在`JsonObjectBuilder`实例上调用`add()`方法来生成一个`JsonObject`实例。”

代码块设置如下：

```java
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html;
      charset=UTF-8">
    <title>Login Error</title>
  </head>
  <body>
    There was an error logging in.
    <br />
    <a href="login.html">Try again</a>
  </body>
</html>
```

当我们希望将您的注意力引到代码块的一个特定部分时，相关的行或项目将以粗体显示：

```java
package com.ensode.jakartaeebook.security.basicauthexample;
//imports omitted for brevity
@BasicAuthenticationMechanismDefinition
@WebServlet(name = "SecuredServlet", urlPatterns = {"/securedServlet"})
@ServletSecurity(
        @HttpConstraint(rolesAllowed = "admin"))
public class SecuredServlet extends HttpServlet {
  @Override
  protected void doGet(HttpServletRequest request,
    HttpServletResponse response) throws ServletException, IOException {
    response.getOutputStream().print(
      "Congratulations, login successful.");
  }
}
```

任何命令行输入或输出都按以下方式编写：

```java
appclient -client simplesessionbeanclient.jar
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词以**粗体**显示。以下是一个示例：“现在我们已经创建了一个客户，我们的**客户列表**页面显示了一个数据表，列出了我们刚刚创建的客户。”

提示或重要注意事项

看起来是这样的。

# 联系我们

我们读者的反馈总是受欢迎的。

**一般反馈**：如果您对此书的任何方面有疑问，请通过电子邮件发送至 customercare@packtpub.com，并在邮件主题中提及书名。

**勘误表**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在此书中发现错误，我们将不胜感激，如果您能向我们报告此错误。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表格。

**盗版**：如果您在互联网上以任何形式遇到我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过电子邮件发送至 copyright@packtpub.com，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)

# 分享您的想法

一旦您阅读了*Jakarta EE 应用程序开发*，我们很乐意听到您的想法！请[点击此处直接进入此书的亚马逊评论页面](https://packt.link/r/1835085261)并分享您的反馈。

您的评论对我们和科技社区都很重要，并将帮助我们确保我们提供高质量的内容。

# 下载此书的免费 PDF 副本

感谢您购买此书！

您喜欢在路上阅读，但又无法携带您的印刷书籍到处走吗？

您的电子书购买是否与您选择的设备不兼容？

别担心，现在，随着每本 Packt 书籍，您都可以免费获得该书的 DRM 免费 PDF 版本。

在任何地方、任何设备上阅读。直接从您喜欢的技术书籍中搜索、复制和粘贴代码到您的应用程序中。

福利远不止于此，您还可以获得独家折扣、时事通讯和每日免费内容的每日邮箱访问权限

按照以下简单步骤获取福利：

1.  扫描下面的二维码或访问以下链接

![下载此书的免费 PDF 副本二维码](img/B21231_QR_Free_PDF.jpg)(https://packt.link/free-ebook/978-1-83508-526-4)

[`packt.link/free-ebook/978-1-83508-526-4`](https://packt.link/free-ebook/978-1-83508-526-4)

1.  提交您的购买证明

1.  就这样！我们将直接将您的免费 PDF 和其他福利发送到您的邮箱
