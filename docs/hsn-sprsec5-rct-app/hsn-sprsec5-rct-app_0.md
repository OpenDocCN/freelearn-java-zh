# 前言

安全是创建应用程序最困难和高压的问题之一。当您必须将其与现有代码、新技术和其他框架集成时，正确保护应用程序的复杂性会增加。本书将向读者展示如何使用经过验证的 Spring Security 框架轻松保护他们的 Java 应用程序，这是一个高度可定制和强大的身份验证和授权框架。

Spring Security 是一个著名的、成熟的 Java/JEE 框架，可以为您的应用程序提供企业级的安全功能，而且毫不费力。它还有一些模块，可以让我们集成各种认证机制，我们将在本书中使用实际编码来深入研究每一个认证机制。

许多示例仍将使用 Spring MVC web 应用程序框架来解释，但仍将具有响应式编程的特色。

响应式编程正在受到关注，本书将展示 Spring Security 与 Spring WebFlux web 应用程序框架的集成。除了响应式编程，本书还将详细介绍其他 Spring Security 功能。

最后，我们还将介绍市场上可用的一些产品，这些产品可以与 Spring Security 一起使用，以实现现代应用程序中所需的一些安全功能。这些产品提供了新的/增强的安全功能，并且在各个方面与 Spring Security 协同工作。其中一些产品也得到了 Spring 社区的全力支持。

# 这本书适合谁

这本书适合以下任何人：

+   任何希望将 Spring Security 集成到他们的应用程序中的 Spring Framework 爱好者

+   任何热衷的 Java 开发人员，希望开始使用 Spring Framework 的核心模块之一，即 Spring Security

+   有经验的 Spring Framework 开发人员，希望能够亲自动手使用最新的 Spring Security 模块，并且也想开始使用响应式编程范式编写应用程序的人

# 本书涵盖了什么

[第一章]，*Spring 5 和 Spring Security 5 概述*，向您介绍了新的应用程序要求，然后介绍了响应式编程概念。它涉及应用程序安全以及 Spring Security 在应用程序中解决安全问题的方法。该章节随后更深入地介绍了 Spring Security，最后解释了本书中示例的结构。

[第二章]，*深入研究 Spring Security*，深入探讨了核心 Spring Security 的技术能力，即身份验证和授权。然后，该章节通过一些示例代码让您动手实践，我们将使用 Spring Security 设置一个项目。然后，在适当的时候，向您介绍了本书中将解释代码示例的方法。

[第三章]，*使用 SAML、LDAP 和 OAuth/OIDC 进行身份验证*，向您介绍了三种身份验证机制，即 SAML、LDAP 和 OAuth/OIDC。这是两个主要章节中的第一个，我们将通过实际编码深入研究 Spring Security 支持的各种身份验证机制。我们将使用简单的示例来解释每种身份验证机制，以涵盖主题的要点，并且我们将保持示例简单以便易于理解。

第四章，*使用 CAS 和 JAAS 进行身份验证*，向您介绍了企业中非常普遍的另外两种身份验证机制——CAS 和 JAAS。这是两个主要章节中的第二个，类似于[第三章](https://cdp.packtpub.com/hands_on_spring_security_5_for_reactive_applications/wp-admin/post.php?post=25&action=edit#post_28)，*使用 SAML、LDAP 和 OAuth/OIDC 进行身份验证*，最初将涵盖这些身份验证机制的理论方面。本章通过使用 Spring Security 实现一个完整的示例来结束这个主题。

第五章，*与 Spring WebFlux 集成*，向您介绍了作为 Spring 5 的一部分引入的新模块之一——Spring WebFlux。Spring WebFlux 是 Spring 生态系统中的 Web 应用程序框架，从头开始构建，完全是响应式的。本章将介绍 Spring Security 的响应式部分，并详细介绍 Spring WebFlux 框架本身。首先，我们将通过一个示例向您介绍 Spring WebFlux，然后我们将在基础应用程序上构建额外的技术能力。

第六章，*REST API 安全*，首先介绍了有关 REST 和 JWT 的一些重要概念。然后介绍了 OAuth 的概念，并使用实际编码示例解释了简单和高级的 REST API 安全，重点是利用 Spring Framework 中的 Spring Security 和 Spring Boot 模块。示例将使用 OAuth 协议，并将使用 Spring Security 充分保护 REST API。除此之外，JWT 将用于在服务器和客户端之间交换声明。

第七章，*Spring 安全附加组件*，介绍了许多产品（开源和付费版本），可以考虑与 Spring Security 一起使用。这些产品是强有力的竞争者，可以用来实现您在应用程序中寻找的技术能力，以满足各种安全要求。我们将通过概述应用程序中需要解决的技术能力的要点来向您介绍产品，然后再看一下相关产品，并解释它如何提供您需要的解决方案。

# 为了充分利用本书

1.  本书包含许多示例，全部在 Macintosh 机器上使用 IDE（IntelliJ）编码和执行。因此，为了轻松跟随示例，使用 macOS 和 IntelliJ 将会大有帮助。但是，所有代码都可以在 Macintosh、Windows 和 Linux 系统上执行。

1.  需要具备基本到中级的使用 Java 和 Spring Framework 构建应用程序的经验，才能轻松阅读本书。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便文件直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  登录或注册[www.packtpub.com](http://www.packtpub.com/support)。

1.  选择“支持”选项卡。

1.  单击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

下载文件后，请确保使用最新版本的解压缩软件解压或提取文件夹。

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Hands-On-Spring-Security-5-for-Reactive-Applications`](https://github.com/PacktPublishing/Hands-On-Spring-Security-5-for-Reactive-Applications)。如果代码有更新，将在现有的 GitHub 存储库中进行更新。

我们还有其他代码包，来自我们丰富的图书和视频目录，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。去看看吧！

# 下载彩色图像

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/HandsOnSpringSecurity5forReactiveApplications_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/HandsOnSpringSecurity5forReactiveApplications_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这是一个例子：“`Flux<T>`是一个带有基本流操作并支持*0.*.*n*个元素的`Publisher<T>`。”

代码块设置如下：

```java
public abstract class Flux<T>
    extends Object
    implements Publisher<T>
```

任何命令行输入或输出都是这样写的：

```java
curl http://localhost:8080/api/movie -v -u admin:password
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会在文本中出现。这是一个例子：“输入用户名为`admin`，密码为`password`，然后点击“登录”。”

警告或重要说明看起来像这样。

提示和技巧看起来像这样。
