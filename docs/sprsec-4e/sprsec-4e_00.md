# 前言

知道经验丰富的黑客渴望测试你的技能，这使得安全成为创建应用程序中最困难且压力最大的问题之一。当你必须将这一因素与现有代码、新技术和其他框架集成时，正确保护应用程序的复杂性会进一步增加。使用这本书，你可以轻松地使用经过验证和值得信赖的**Spring Security**框架来保护你的 Java 应用程序，这是一个强大且高度可定制的认证和访问控制框架。

本书首先集成各种认证机制。然后演示如何正确限制对应用程序的访问。它还涵盖了与一些更受欢迎的 Web 框架集成的技巧。还包括 Spring Security 如何防御会话固定攻击的示例，以及如何利用会话管理进行管理功能，以及并发控制。

它以 RESTful Web 服务和微服务的先进安全场景结束，详细说明了围绕无状态认证的问题，并展示了一种简洁的、分步骤的方法来解决这些问题。

# 本书面向对象

如果你是一名 Java Web 和/或 RESTful Web 服务开发者，并且对创建 Java 17/21、Java Web 和/或 RESTful Web 服务应用程序、XML 以及 Spring Security 框架有基本的了解，那么这本书适合你。

# 本书涵盖内容

*第一章*，*不安全应用程序的解剖结构*，涵盖了对我们的日历应用程序的假设性安全审计，说明了通过正确应用 Spring Security 可以解决的常见问题。你将了解一些基本的安全术语，并回顾一些使示例应用程序运行所需的先决条件。

*第二章*，*Spring Security 入门*，演示了 Spring Security 的`"Hello World"`安装。之后，本章将引导你了解 Spring Security 的一些最常见定制。

*第三章*，*自定义认证*，通过自定义认证基础设施的关键部分来逐步解释 Spring Security 的认证架构，以解决现实世界的问题。通过这些定制，你将了解 Spring Security 认证的工作原理以及如何与现有和新认证机制集成。

*第四章*，*基于 JDBC 的认证*，介绍了使用 Spring Security 内置的**Java 数据库连接**（**JDBC**）支持对数据库进行认证。然后我们讨论了如何使用 Spring Security 的新加密模块来保护我们的密码。

*第五章*, *使用 Spring Data 进行认证*，探讨了 Spring Data 项目，以及如何利用**Jakarta Persistence** (**JPA**)在关系型数据库上执行认证。我们还将探讨如何使用 MongoDB 对文档数据库进行认证。

*第六章*, *LDAP 目录服务*，将回顾**轻量级目录访问协议** (**LDAP**)，并了解它如何集成到启用 Spring Security 的应用程序中，为感兴趣的参与者提供认证、授权和用户信息服务。

*第七章*, *记住我服务*，展示了 Spring Security 中记住我功能的用法以及如何配置它。我们还探讨了在使用它时需要考虑的额外因素。我们将添加应用记住用户的功能，即使他们的会话已过期且浏览器已关闭。

*第八章*, *使用 TLS 的客户端证书认证*，展示了尽管用户名和密码认证极为常见，正如我们在*第一章*，*不安全应用程序的解剖结构*，以及*第二章*，*Spring Security 入门*中讨论的那样，存在其他认证形式，允许用户展示不同类型的凭证。Spring Security 也满足这些需求。在本章中，我们将超越基于表单的认证，探索使用受信任的客户端证书进行认证。

*第九章*, *开启 OAuth 2 支持*，解释了 OAuth 2 是一种非常流行的受信任身份管理形式，允许用户通过单个受信任的提供者管理其身份。这个便捷的功能为用户提供将密码和个人信息存储在受信任的 OAuth 2 提供者的安全性，并在请求时可选地披露个人信息。此外，启用 OAuth-2 的网站可以提供信心，即提供 OAuth 2 凭证的用户就是他们所说的那个人。

*第十章*, *SAML 2 支持*，将深入探讨**安全断言标记语言** (**SAML 2.0**)支持的领域，以及它如何无缝集成到 Spring Security 应用程序中。SAML 2.0 是一种基于 XML 的标准，用于在**身份提供者** (**IdP**)和**服务提供者** (**SP**)之间交换认证和授权数据。

*第十一章*，*细粒度访问控制*，将首先探讨两种实现细粒度授权的方法——这种授权可能会影响应用程序页面的一部分。接下来，我们将探讨 Spring Security 通过方法注解和使用基于接口的代理来确保业务层安全的方法，以实现**面向方面编程**（**AOP**）。然后，我们将回顾基于注解的安全性的一个有趣功能，该功能允许对数据集合进行基于角色的过滤。最后，我们将探讨基于类的代理与基于接口的代理之间的区别。

*第十二章*，*访问控制列表*，将讨论复杂的**访问控制列表**（**ACLs**）主题，它可以为域对象实例级授权提供丰富的模型。Spring Security 附带了一个强大但复杂的 ACL 模块，可以合理地满足从小型到中型实施的需求。

*第十三章*，*自定义授权*，将包括一些针对 Spring Security 关键授权 API 的自定义实现。完成这些后，我们将利用对自定义实现的理解来了解 Spring Security 授权架构的工作原理。

*第十四章*，*会话管理*，讨论了 Spring Security 如何管理和保护用户会话。本章首先解释了会话固定攻击以及 Spring Security 如何防御这些攻击。然后讨论了如何管理已登录用户并限制单个用户并发会话的数量。最后，我们描述了 Spring Security 如何将用户与`HttpSession`关联以及如何自定义此行为。

*第十五章*，*额外的 Spring Security 功能*，涵盖了其他 Spring Security 功能，包括常见的安全漏洞，如**跨站脚本**（**XSS**）、**跨站请求伪造**（**CSRF**）、**同步令牌**和**点击劫持**，以及如何防范它们。

*第十六章*，*迁移到 Spring Security 6*，提供了从 Spring Security 5 迁移的路径，包括显著的配置更改、类和包迁移，以及包括 Java 17 支持和新的 OAuth 2.1 认证机制在内的重要新功能。

它还强调了 Spring Security 6.1 中可以找到的新功能，并提供了书中这些功能示例的参考。

*第十七章*，*使用 OAuth 2 和 JSON Web 令牌的微服务安全*，将探讨基于微服务的架构以及 OAuth 2 与**JSON Web 令牌**（**JWT**）如何在基于 Spring 的应用程序中确保微服务的安全。它还突出了 Spring Security 6.1 中的新功能，并提供了书中这些功能示例的参考。

*第十八章*，*使用中央认证服务进行单点登录*，展示了如何通过集成**中央认证服务**（**CAS**）为您的 Spring-Security 启用应用程序提供单点登录和单点注销支持。

*第十九章*，*构建 GraalVM 原生镜像*，探讨了 Spring Security 6 对使用 GraalVM 构建原生图像的支持。这可以是一种提高 Spring Security 应用程序性能和安全的绝佳方式。

# 要充分利用本书

与示例代码集成的首选方法是提供**Gradle**和**Maven**兼容的项目。由于许多**集成开发环境**（**IDE**）与 Gradle 和 Maven 有丰富的集成，用户应该能够将代码导入支持 Gradle 或 Maven 的任何 IDE。由于许多开发者使用 Gradle 和 Maven，我们认为这是打包示例的最直接方法。无论您熟悉哪种开发环境，希望您能找到一种方法来处理本书中的示例。

许多 IDE 提供了 Gradle 或 Maven 工具，可以自动为您下载 Spring 和**Spring Security 6**的 Javadoc 和源代码。然而，有时这可能不可行。在这种情况下，您可能需要下载**Spring 6**和 Spring Security 6 的完整版本。Javadoc 和源代码质量上乘。如果您感到困惑或需要更多信息，示例可以为您提供额外的支持或学习保障。

要运行示例应用程序，您需要一个 IDE，例如**IntelliJ IDEA**或**Eclipse**，并使用**Gradle**或**Maven**构建，这些工具对硬件要求不严格。然而，以下是一些一般性建议，以确保开发体验顺畅：

1.  系统要求：

    +   一台配备至少 4GB RAM 的现代计算机（建议 8GB 或更多）。

    +   多核处理器以加快构建和开发速度。

1.  操作系统：

    +   Spring 应用程序可以在 Windows、macOS 或 Linux 上开发。选择您最舒适的一个。

1.  磁盘空间：

    +   您的项目文件、依赖项以及可能使用的任何数据库都需要磁盘空间。至少建议有 10GB 的空闲磁盘空间。

1.  网络连接：

    +   在项目设置期间下载依赖项、插件和库可能需要稳定的网络连接。

| **本书涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |
| IntelliJ IDEA 和 Eclipse 都是 Spring 开发的流行选择 | Windows、macOS 或 Linux |
| JDK 版本：17 或 21 |  |
| Spring- Security 6. |  |
| Spring- Boot 3. |  |
| Thymeleaf 6. |  |

**如果您正在使用本书的数字版，我们建议您亲自输入代码或从本书的 GitHub 仓库（下一节中提供链接）获取代码。这样做将帮助您避免与代码的复制和粘贴相关的任何潜在错误。**

从*第三章*的*自定义身份验证*开始，本书将重点转向深入探讨 Spring Security，尤其是在与 Spring Boot 框架结合使用时。

# 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件[`github.com/PacktPublishing/Spring-Security-Fourth-Edition/`](https://github.com/PacktPublishing/Spring-Security-Fourth-Edition/)。如果代码有更新，它将在 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包可供选择，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

# 代码实战

本书代码实战视频可在[`packt.link/Om1ow`](https://packt.link/Om1ow)查看。

# 使用的约定

本书使用了多种文本约定。

`文本中的代码`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“我们从`classpath`加载`calendar.ldif`文件，并使用它来填充 LDAP 服务器。”

代码块设置如下：

```java
//src/main/java/com/packtpub/springsecurity/configuration/SecurityConfig.java
@Bean public SecurityFilterChain filterChain(HttpSecurity http,        PersistentTokenRepository persistentTokenRepository, RememberMeServices rememberMeServices) throws Exception {    http.authorizeHttpRequests( authz -> authz                 .requestMatchers("/webjars/**").permitAll()
…
    // Remember Me     http.rememberMe(httpSecurityRememberMeConfigurer -> httpSecurityRememberMeConfigurer           .key("jbcpCalendar")          .rememberMeServices(rememberMeServices)          .tokenRepository(persistentTokenRepository));        return http.build();}
```

第一行`//src/main/java/com/packtpub/springsecurity/configuration/SecurityConfig.java`指示要修改的文件位置。

任何命令行输入或输出都应如下编写：

```java
X-Content-Security-Policy: default-src 'self' X-WebKit-CSP: default-src 'self'
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词以**粗体**显示。以下是一个示例：“右键单击**世界**并选择**搜索**。”

小贴士或重要提示

看起来像这样。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请通过 customercare@packtpub.com 给我们发邮件，并在邮件主题中提及书名。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告，我们将非常感谢。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表格。

**盗版**：如果您在互联网上以任何形式发现我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过版权@packtpub.com 与我们联系，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

一旦您阅读了《Spring Security》，我们很乐意听听您的想法！请[点击此处直接进入本书的亚马逊评论页面](https://packt.link/r/1-835-46050-X)并分享您的反馈。

您的评论对我们和科技社区非常重要，并将帮助我们确保我们提供高质量的内容。

# 下载本书的免费 PDF 副本

感谢您购买本书！

您喜欢随时随地阅读，但无法携带您的印刷书籍到处走？

您的电子书购买是否与您选择的设备不兼容？

别担心，现在每购买一本 Packt 图书，你都可以免费获得该书的 DRM 免费 PDF 版本。

在任何地方、任何地方、任何设备上阅读。从您最喜欢的技术书籍中搜索、复制和粘贴代码到您的应用程序中。

优惠远不止于此，您还可以获得独家折扣、时事通讯和每日收件箱中的精彩免费内容。

按照以下简单步骤获取福利：

1.  扫描二维码或访问以下链接：

![二维码](img/B21757_QR_Free_PDF.jpg)

[`packt.link/free-ebook/9781835460504`](https://packt.link/free-ebook/9781835460504)

1.  提交您的购买证明

1.  就这些！我们将直接将您的免费 PDF 和其他福利发送到您的电子邮件。

# 第一部分：应用安全基础

本部分深入探讨应用安全的基础方面，为理解潜在漏洞奠定基础。我们通过 Spring Security 对应用安全进行全面的探索。本部分向您介绍对假设的日历应用进行安全审计的过程。通过这次审计，我们发现了常见的安全漏洞，并为实施强大的安全措施奠定了基础。

在此基础上，本部分指导您进行 Spring Security 的安装和配置。我们从基本的`"Hello World"`示例开始，逐步定制 Spring Security 以满足我们应用程序的特定需求。

我们还将更深入地探讨 Spring Security 中的身份验证过程。通过定制身份验证基础设施的关键组件，我们解决现实世界的身份验证挑战，并全面了解 Spring Security 的身份验证机制。通过实际示例和动手练习，我们学习如何无缝地将自定义身份验证解决方案集成到我们的应用程序中。

本部分包含以下章节：

+   *第一章*，*不安全应用程序的解剖结构*

+   *第二章*，*Spring Security 入门*

+   *第三章*，*自定义身份验证*
