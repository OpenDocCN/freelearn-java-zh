# 前言

Spring 框架为 Java 开发提供了极大的灵活性，这也导致了繁琐的配置工作。Spring Boot 解决了 Spring 的配置难题，使得创建独立的生产级 Spring 应用程序变得容易。

这本实用指南使现有的开发过程更加高效。*《Spring Boot 烹饪书 2.0，第二版》* 智能地结合了所有技能和专业知识，以高效地使用 Spring Boot 在本地和云中开发、测试、部署和监控应用程序。我们从概述您将学习的 Spring Boot 的重要功能开始，以创建一个用于 RESTful 服务的 Web 应用程序。您还将学习如何通过了解自定义路由和资产路径以及如何修改路由模式来微调 Web 应用程序的行为，同时解决复杂企业应用程序的需求，并理解自定义 Spring Boot 启动器的创建。

本书还包含了创建各种测试的示例，这些测试是在 Spring Boot 1.4 和 2.0 中引入的，以及深入了解 Spring Boot DevTools 的方法。我们将探讨 Spring Boot Cloud 模块的基础以及各种云启动器，以创建原生云应用程序并利用服务发现和断路器。

# 本书面向对象

本书针对对 Spring 和 Java 应用程序开发有良好知识和理解的 Java 开发者，熟悉软件开发生命周期（SDLC）的概念，并理解不同类型测试策略、通用监控和部署关注点的需求。本书将帮助您学习高效的 Spring Boot 开发技术、集成和扩展能力，以便使现有的开发过程更加高效。

# 本书涵盖内容

第一章，*Spring Boot 入门*，概述了框架中包含的重要和有用的 Spring Boot 启动器。您将学习如何使用 [spring.io](https://spring.io/) 资源，如何开始一个简单的项目，以及如何配置构建文件以包含所需的启动器。本章将以创建一个配置为执行一些计划任务的简单命令行应用程序结束。

第二章，*配置 Web 应用程序*，提供了如何创建和添加自定义的 servlet 过滤器、拦截器、转换器、格式化器和属性编辑器到 Spring Boot Web 应用程序的示例。它将从创建一个新的 Web 应用程序开始，并使用它作为基础，使用我们在本章前面讨论的组件进行定制。

第三章，*Web 框架行为调整*，深入探讨了调整 Web 应用程序行为。它将涵盖配置自定义路由规则和模式，添加额外的静态资源路径，以及添加和修改 Servlet 容器连接器和其他属性，例如启用 SSL。

第四章，*编写自定义 Spring Boot 启动器*，展示了如何创建自定义 Spring Boot 启动器以提供可能适用于复杂企业应用程序的额外行为和功能。您将了解底层自动配置机制的工作原理，以及如何使用它们有选择性地启用/禁用默认功能并条件性地加载您自己的。

第五章，*应用测试*，探讨了测试 Spring Boot 应用程序的不同技术。它将从介绍测试 MVC 应用程序开始，然后讨论如何使用预填充数据的内存数据库来模拟测试中的真实数据库交互的一些技巧，并以 Cucumber 和 Spock 等测试工具的 BDD 示例结束。

第六章，*应用打包和部署*，将涵盖配置构建以生成适用于 Linux/OSX 环境的 Docker 镜像和自执行二进制文件的示例。我们将探讨使用 Consul 进行外部应用配置的选项，并深入了解 Spring Boot 环境和配置功能的具体细节。

第七章，*健康监控和数据可视化*，探讨了 Spring Boot 提供的各种机制，帮助我们查看与应用程序健康相关的数据。我们将从学习如何编写和公开自定义健康指标，并使用 HTTP 端点和 JMX 查看数据开始。然后，我们将概述 SSHd 的管理命令的创建，并以使用 Micrometer 度量框架将监控数据与 Graphite 和 Dashing 集成结束。

第八章，*Spring Boot DevTools*，深入探讨了如何在应用开发过程中使用 Spring Boot DevTools 来简化动态代码重新编译/重启和远程代码更新的常见任务。我们将学习如何将 DevTools 添加到项目中，随后探讨 DevTools 如何通过自动重启运行中的应用来加速开发过程。

第九章，*Spring Cloud*，提供了 Spring Boot Cloud 模块中各种功能的示例。您将学习如何使用不同的云模块进行服务发现，例如 Consul 或 Netflix Eureka。稍后，我们将探讨如何结合 Netflix 库，如 Hystrix 断路器和基于 Feign 接口的 REST 客户端。

# 要充分利用本书

对于本书，您需要在您喜欢的操作系统（Linux、Windows 或 OS X）上安装 JDK 1.8。假设读者对 Java 有合理的了解，包括 JDK 1.8 添加的最新功能，以及 Spring 框架及其操作概念的基本知识，如依赖注入、控制反转和 MVC。

其余的软件，如 Gradle 构建工具、所有必要的 Java 库（如 Spring Boot、Spring 框架及其依赖项），以及 Docker、Consul、Graphite、Grafana 和 Dashing，将在整个食谱中安装。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择支持选项卡。

1.  点击代码下载与勘误。

1.  在搜索框中输入书名，并遵循屏幕上的说明。

下载文件后，请确保使用最新版本解压缩或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Spring-Boot-2.0-Cookbook-Second-Edition`](https://github.com/PacktPublishing/Spring-Boot-2.0-Cookbook-Second-Edition)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。查看它们！

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“现在我们将为`LocaleChangeInterceptor`添加一个`@Bean`声明。”

代码块设置如下：

```java
@Override 
public void addInterceptors(InterceptorRegistry registry) { 
  registry.addInterceptor(localeChangeInterceptor()); 
} 
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
[default]
exten => s,1,Dial(Zap/1|30)
exten => s,2,Voicemail(u100)
exten => s,102,Voicemail(b100)
exten => i,1,Voicemail(s0)
```

任何命令行输入或输出都按以下方式编写：

```java
$ ./gradlew clean bootRun
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“在“搜索依赖项”下选择“Actuator”选项。”

警告或重要注意事项如下所示。

技巧和窍门如下所示。

# 章节

在本书中，您将找到一些频繁出现的标题（*准备就绪*、*如何操作...*、*工作原理...*、*还有更多...*和*另请参阅*）。

为了清楚地说明如何完成食谱，请按照以下方式使用这些部分：

# 准备工作

本节告诉您在食谱中可以期待什么，并描述如何设置任何软件或任何为食谱所需的初步设置。

# 如何做...

本节包含遵循食谱所需的步骤。

# 它是如何工作的...

本节通常包含对前一个章节发生事件的详细解释。

# 还有更多...

本节包含有关食谱的附加信息，以便您对食谱有更深入的了解。

# 参考信息

本节提供了对食谱有用的其他信息的链接。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：请发送电子邮件至 `feedback@packtpub.com`，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过 `questions@packtpub.com` 发送电子邮件给我们。

**勘误表**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在本书中发现错误，我们将不胜感激，如果您能向我们报告这一点。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误表提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何形式的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过 `copyright@packtpub.com` 联系我们，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/).

# 评论

请留下评论。一旦您阅读并使用过这本书，为什么不在此购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/).
