# 前言

本书的目标是使您熟悉可用于在云中开发和部署 Java EE 应用程序的工具。您将全程参与应用程序开发过程：创建应用程序、在云中部署、配置持续集成以及创建服务之间的安全性和容错通信。结果，您将获得 Java EE 云开发的实际知识，这可以作为您未来项目的参考。

# 本书面向对象

如果您是一位熟悉 Java EE 技术且希望了解如何使用这些技术在 WildFly 和 OpenShift 的云环境中，那么这本书是为您准备的。

# 本书涵盖内容

第一章，*Java EE 和现代架构方法*，为用户提供 Java EE 当前状态的概述及其与现代架构方法（即微服务和云计算）的相关性。我们将介绍本书中将使用的工具以及我们将要开发的应用程序。

第二章，*熟悉 WildFly Swarm*，涵盖 WildFly 及其与 Java EE 及其主要特性的关系。我们将介绍 WildFly Swarm——WildFly 的侧项目，描述其目的，并展示如何使用它来开发微服务。

第三章，*适当规模您的服务*，专注于 Swarm 如何仅使用对服务必要的依赖来创建服务。您将更详细地了解什么是分数，Swarm 如何检测应使用哪些分数，以及您如何修改分数发现行为。

第四章，*调整您的服务配置*，帮助您学习如何配置 Swarm 服务。我们将向您展示不同配置工具的实际案例，以及您如何使用它们来引导应用程序的行为。

第五章，*使用 Arquillian 测试您的服务*，教您如何测试您的微服务。本章将介绍 Arquillian，即将要使用的测试框架，并展示项目的目的及其主要特性。随后，您将通过实际案例学习如何基于实际案例开发、编写和配置服务的测试。

第六章，*使用 OpenShift 在云中部署应用程序*，讨论如何将服务部署到云中，本章使用 OpenShift 来实现这一点。

第七章，*为您的应用程序配置存储*，首先帮助您学习 OpenShift 存储配置的理论基础。随后，我们将向您展示如何在云中部署数据库，并配置您的云应用程序使用它。

第八章，*扩展和连接您的服务*，更详细地探讨了在 OpenShift 环境中部署、扩展和连接应用程序的过程。

第九章，*使用 Jenkins 配置持续集成*，教您如何将宠物商店应用程序与 Jenkins，一个持续集成服务器集成。我们将介绍 CI 概念，并展示如何使用 Jenkins 实现。

第十章，*使用 Keycloak 提供安全功能*，讨论了基于令牌的分布式安全基础。我们将介绍 Keycloak，一个身份验证服务器，可用于保护分布式云应用程序。作为一个实际示例，我们将确保 Petstore 应用程序的部分 API 安全。

第十一章，*使用 Hystrix 增强容错性*，讨论了如何在分布式环境中处理不可避免的网络故障。为了做到这一点，我们将介绍断路器架构模式，并涵盖何时应该使用它及其优势。我们将查看其 Netflix 实现，Hystrix。我们将介绍其实现方式和如何使用它。

第十二章，*未来方向*，简要描述了 Java EE 开发的未来可能看起来如何，例如，平台演变的计划以及本书中描述的应用程序提供概念的未来标准化。我们还将探讨 MicroProfile 和 Jakarta EE，描述它们的目的，并强调它们如何帮助您以更快的速度推进平台。

# 为了充分利用本书

本书假设您熟悉 Java EE 技术。虽然我们将在示例中简要回顾 Java EE 构造的功能，但不会进行详细解释。

代码存储库包含所有章节的示例。为了帮助您导航，示例从章节内部进行索引。

# 下载示例代码文件

您可以从 [www.packtpub.com](http://www.packtpub.com) 的账户下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问 [www.packtpub.com/support](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在 [www.packtpub.com](http://www.packtpub.com/support) 登录或注册。

1.  选择“支持”标签。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载后，请确保您使用最新版本解压缩或提取文件夹：

+   适用于 Windows 的 WinRAR/7-Zip

+   适用于 Mac 的 Zipeg/iZip/UnRarX

+   适用于 Linux 的 7-Zip/PeaZip

本书代码包托管在 GitHub 上，网址为[`github.com/PacktPublishing/Hands-On-Cloud-Development-with-WildFly`](https://github.com/PacktPublishing/Hands-On-Cloud-Development-with-WildFly)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：

[`www.packtpub.com/sites/default/files/downloads/HandsOnCloudDevelopmentwithWildFly_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/HandsOnCloudDevelopmentwithWildFly_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账号。以下是一个示例：“计算机编程书籍通常从`Hello World`应用程序开始。”

代码块设置如下：

```java
package org.packt.swarm;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;

```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
package org.packt.swarm;

import javax.ws.rs.ApplicationPath;
import javax.ws.rs.core.Application;

@ApplicationPath("/")
public class HelloWorldApplication extends Application {
}
```

任何命令行输入或输出都按以下方式编写：

```java
mvn wildfly-swarm:run
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“我们必须点击网页控制台中的服务菜单中的创建路由。”

警告或重要注意事项看起来是这样的。

小技巧和窍门看起来是这样的。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：请通过`feedback@packtpub.com`发送电子邮件，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过`questions@packtpub.com`与我们联系。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一点。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式遇到我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过`copyright@packtpub.com`与我们联系，并提供材料的链接。

**如果您有兴趣成为一名作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/).

# 评论

请留下评论。一旦您阅读并使用了这本书，为何不在您购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/).
