# 序言

欢迎来到 Spring Security 4.2 的世界！我们非常高兴您拥有了这本唯一专门针对 Spring Security 4.2 出版的书籍。在您开始阅读本书之前，我们想向您概述一下本书的组织结构以及如何充分利用它。

阅读完这本书后，您应该对关键的安全概念有所了解，并能够解决大多数需要使用 Spring Security 解决的实际问题。在这个过程中，您将深入了解 Spring Security 的架构，这使您能够处理书中未涵盖的任何意外用例。

本书分为以下四个主要部分：

+   第一部分（第一章，*不安全应用程序的剖析*和第二章，*Spring Security 入门*)提供了 Spring Security 的简介，并让您能够快速开始使用 Spring Security。

+   第二部分（第三章，*自定义认证*，第四章，*基于 JDBC 的认证*，第五章，*使用 Spring Data 的认证*，第六章，*LDAP 目录服务*，第七章，*记住我服务*，第八章，*使用 TLS 的客户端证书认证*，和第九章，*开放给 OAuth 2*)提供了与多种不同认证技术集成的高级指导。

+   第三部分（第十章，*使用中央认证服务的单点登录*，第十一章，*细粒度访问控制*，和第十二章，*访问控制列表*)解释了 Spring Security 的授权支持是如何工作的。

+   最后，最后一部分（第十三章，*自定义授权*，第十四章，*会话管理*，第十五章，*Spring Security 的其他功能*，以及第十六章，*迁移到 Spring Security 4.2*，第十七章，*使用 OAuth 2 和 JSON Web Tokens 的微服务安全*)提供了专门主题的信息和指导，帮助您执行特定任务。

安全是一个非常交织的概念，书中也有很多这样的主题。然而，一旦您阅读了前三章，其他章节相对独立。这意味着您可以轻松地跳过章节，但仍能理解正在发生的事情。我们的目标是提供一个食谱式的指南，即使您通读全书，也能帮助您清楚地理解 Spring Security。

本书通过一个简单的基于 Spring Web MVC 的应用程序来阐述如何解决现实世界的问题。这个应用程序被设计得非常简单直接，并且故意包含非常少的功能——这个应用程序的目标是鼓励你专注于 Spring Security 概念，而不是陷入应用程序开发的复杂性中。如果你花时间回顾示例应用程序的源代码并尝试跟随练习，你将更容易地跟随这本书。在附录的*开始使用 JBCP 日历示例代码*部分，有一些关于入门的技巧。

# 本书涵盖内容

第一章，《不安全应用程序的剖析》，涵盖了我们的日历应用程序的一个假设性安全审计，说明了可以通过适当应用 Spring Security 解决的一些常见问题。你将学习一些基本的安全术语，并回顾一些将示例应用程序启动并运行的先决条件。

第二章，《Spring Security 入门》，展示了 Spring Security 的“Hello World”安装。在本章中，读者将了解一些 Spring Security 最常见的自定义操作。

第三章，《自定义认证》，逐步解释了通过自定义认证基础设施的关键部分来解决现实世界问题，从而了解 Spring Security 的认证架构。通过这些自定义操作，你将了解 Spring Security 认证是如何工作的，以及如何与现有的和新型的认证机制集成。

第四章，《基于 JDBC 的认证》，介绍了使用 Spring Security 内置的 JDBC 支持的数据库认证。然后，我们讨论了如何使用 Spring Security 的新加密模块来保护我们的密码。

第五章，《使用 Spring Data 的认证》，介绍了使用 Spring Security 与 Spring Data JPA 和 Spring Data MongoDB 集成的数据库认证。

第六章，《LDAP 目录服务》，提供了一个关于应用程序与 LDAP 目录服务器集成的指南。

第七章，《记住我服务》，展示了 Spring Security 中记住我功能的用法和如何配置它。我们还探讨了使用它时需要考虑的其他一些额外因素。

第八章，《使用 TLS 的客户端证书认证》，将基于 X.509 证书的认证作为一个清晰的替代方案，适用于某些商业场景，其中管理的证书可以为我们的应用程序增加额外的安全层。

第九章，《开放给 OAuth 2.0》，介绍了 OAuth 2.0 启用的登录和用户属性交换，以及 OAuth 2.0 协议的逻辑流程的高级概述，包括 Spring OAuth 2.0 和 Spring 社交集成。

第十章 10.html，*与中央认证服务集成实现单点登录*，介绍了与中央认证服务（CAS）集成如何为您的 Spring Security 启用应用程序提供单点登录和单点登出支持。它还演示了如何使用无状态服务的 CAS 代理票证支持。

第十一章 11.html，*细粒度访问控制*，涵盖了页面内授权检查（部分页面渲染）和利用 Spring Security 的方法安全功能实现业务层安全。

第十二章 12.html，*访问控制列表*，介绍了使用 Spring Security ACL 模块实现业务对象级安全的基本概念和基本实现-一个具有非常灵活适用性的强大模块，适用于挑战性的业务安全问题。

第十三章 13.html，*自定义授权*，解释了 Spring Security 的授权工作原理，通过编写 Spring Security 授权基础设施的关键部分的自定义实现。

第十四章 14.html，*会话管理*，讨论了 Spring Security 如何管理和保护用户会话。这一章首先解释了会话固定攻击以及 Spring Security 如何防御它们。然后讨论了您可以如何管理已登录的用户以及单个用户可以有多少个并发会话。最后，我们描述了 Spring Security 如何将用户与 HttpSession 相关联以及如何自定义这种行为。

第十五章 15.html，*额外的 Spring Security 功能*，涵盖了其他 Spring Security 功能，包括常见的网络安全漏洞，如跨站脚本攻击（XSS）、跨站请求伪造（CSRF）、同步令牌和点击劫持，以及如何防范它们。

第十六章 16.html，*迁移到 Spring Security 4.2*，提供从 Spring Security 3 迁移的路径，包括显著的配置更改、类和包迁移以及重要的新功能。它还突出了在 Spring Security 4.2 中可以找到的新功能，并提供参考书中的功能示例。

第十七章 17.html，*使用 OAuth 2 和 JSON Web Tokens 的微服务安全*，探讨了微服务架构以及 OAuth 2 和 JWT 在 Spring 基础应用程序中保护微服务的作用。

附录，*附加参考资料*，包含一些与 Spring Security 直接相关性不大的参考资料，但与本书涵盖的主题仍然相关。最重要的是，它包含一个协助运行随书提供的示例代码的章节。

# 您需要什么（本书）

以下列表包含运行随书提供的示例应用程序所需的软件。一些章节有如下附加要求，这些要求在相应的章节中概述：

+   Java 开发工具包 1.8 可从 Oracle 网站下载，网址为 [`www.oracle.com/technetwork/java/javase/downloads/index.html`](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

+   IntelliJ IDEA 2017+ 可从 [`www.jetbrains.com/idea/`](https://www.jetbrains.com/idea/) 下载

+   Spring Tool Suite 3.9.1.RELEASE+ 可从 [`spring.io/tools/sts`](https://spring.io/tools/sts) 下载

# 本书适合谁

如果您是 Java Web 和/或 RESTful Web 服务开发者，并且具有创建 Java 8、Java Web 和/或 RESTful Web 服务应用程序、XML 和 Spring Framework 的基本理解，这本书适合您。您不需要具备任何之前的 Spring Security 经验。

# 约定

在本书中，您会找到多种文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义。文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、假 URL、用户输入和 Twitter 处理方式如下所示："下一步涉及对 `web.xml` 文件进行一系列更新"。代码块如下所示：

```java
 //build.gradle:
    dependencies {
        compile "org.springframework.security:spring-security-  
        config:${springSecurityVersion}"
        compile "org.springframework.security:spring-security- 
        core:${springSecurityVersion}"
        compile "org.springframework.security:spring-security- 
        web:${springSecurityVersion}"
        ...
    }
```

当我们需要引起您对代码块中的特定部分注意时，相关的行或项目将被加粗：

```java
 [default]
 exten => s,1,Dial(Zap/1|30)
 exten => s,2,Voicemail(u100)
 exten => s,102,Voicemail(b100)
 exten => i,1,Voicemail(s0)
```

任何命令行输入或输出如下所示：

```java
$ ./gradlew idea
```

**新术语**和**重要词汇**以粗体显示。

您在屏幕上看到的单词，例如在菜单或对话框中，会在文本中以这种方式出现："在 Microsoft Windows 中，您可以通过右键单击文件并查看其安全属性（属性 | 安全）来查看文件的一些 ACL 功能，如下面的屏幕截图所示"。

警告或重要说明以这种方式出现。

技巧和窍门以这种方式出现。

# 读者反馈

我们的读者提供的反馈总是受欢迎的。告诉我们您对这本书的看法——您喜欢或不喜欢的地方。读者反馈对我们很重要，因为它有助于我们开发出您能真正从中受益的标题。

要向我们提供一般性反馈，只需给`feedback@packtpub.com`发封电子邮件，并在邮件主题中提到书籍的标题。

如果您在某个主题上有专业知识，并且有兴趣撰写或为书籍做出贡献，请查看我们的作者指南 [www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经成为 Packt 书籍的自豪拥有者，我们有很多事情可以帮助您充分利用您的购买。

# 下载示例代码

您可以从您在 [`www.packtpub.com`](http://www.packtpub.com/) 的账户上下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的 SUPPORT 标签上。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书籍的名称。

1.  选择您要下载代码文件的书籍。

1.  从您购买本书的下拉菜单中选择。

1.  点击“代码下载”。

文件下载完成后，请确保使用最新版本解压或提取文件夹：

+   适用于 Windows 的 WinRAR / 7-Zip

+   适用于 Mac 的 Zipeg / iZip / UnRarX

+   适用于 Linux 的 7-Zip / PeaZip

本书的代码包也托管在 GitHub 上，地址为[`github.com/PacktPublishing/Spring-Security-Third-Edition`](https://github.com/PacktPublishing/Spring-Security-Third-Edition/)。我们还有其他来自我们丰富书籍和视频目录的代码包，您可以在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。去看看吧！

# 勘误表

虽然我们已经尽一切努力确保内容的准确性，但错误仍然会发生。如果您在我们的书中发现错误 - 可能是文本或代码中的错误 - 我们非常感谢您能向我们报告。这样做可以节省其他读者的挫折感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)报告，选择您的书籍，点击勘误提交表单链接，并输入勘误的详细信息。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站，或添加到该标题的勘误部分现有的勘误列表中。要查看之前提交的勘误，请前往[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，在搜索字段中输入书籍的名称。所需信息将在勘误部分出现。

# 盗版

互联网上的版权材料盗版是一个持续存在的问题，所有媒体都受到影响。 Packt 出版社非常重视我们版权和许可的保护。如果您在互联网上以任何形式发现我们作品的非法副本，请立即提供给我们地址或网站名称，以便我们采取补救措施。请通过`copyright@packtpub.com`联系我们，附上疑似盗版材料的链接。您帮助保护我们的作者和我们提供有价值内容的能力，我们非常感激。

# 问题

如果您在阅读本书的任何方面遇到问题，可以通过`questions@packtpub.com`联系我们，我们会尽力解决问题。
