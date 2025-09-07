# 前言

Spring Boot 为 JavaScript 用户提供了一个平台，只需几行代码就能让应用程序启动运行。同时，Angular 是一个基于组件的框架，使得构建 Web 应用程序的前端变得容易。本书解释了 Spring Boot 和 Angular 如何协同工作，帮助您快速有效地创建全栈应用程序。

在本书中，您将首先探索为什么 Spring Boot 和 Angular 是需求旺盛的框架，然后通过专家解决方案和最佳实践来构建您自己的 Web 应用程序。关于后端，您将了解 Spring Boot 如何通过让 Spring 框架和 Spring Boot 扩展承担繁重的工作来提高应用程序的构建效率，同时在使用 Spring Data JPA 和 Postgres 依赖项将数据保存或持久化到数据库中。在前端，您将使用 Angular 构建项目架构，构建响应式表单，并添加身份验证以防止恶意用户从应用程序中窃取数据。

最后，您将了解如何使用 Mockito 测试服务，使用持续集成和持续部署部署应用程序，以及如何将 Spring Boot 和 Angular 集成到一个单一包中，以便在本书结束时，您将能够构建自己的全栈 Web 应用程序。

# 本书面向对象

本书面向忙碌的 Java Web 开发者和 TypeScript 开发者，他们对于开发 Angular 和 Spring Boot 应用程序的经验不多，但希望了解构建全栈 Web 应用程序的最佳实践。

# 本书涵盖内容

*第一章*，*Spring Boot 和 Angular – 大全景*，作为 Spring Boot 和 Angular 当前状态的简要回顾，为您展示 Java Spring Boot 和 Angular Web 开发的未来。您还将看到 Vue.js 作为应用程序的稳定性和可靠性，以及编写和维护 Vue.js 框架的团队。 

*第二章*，*设置开发环境*，教您如何设置计算机的开发环境以构建后端和前端 Web 应用程序。在开始应用程序开发之前，我们将转向不同的 IDE 和文本编辑器来编写代码，并确保一切都已经设置好。

*第三章*，*进入 Spring Boot*，揭示了 Spring Boot 的内部工作原理以及如何使用 Spring Initializr 启动项目。本章还将向您介绍依赖注入和 IoC 容器概念。本章还将探讨 Bean 和注解的工作方式。

*第四章*, *设置数据库和 Spring Data JPA*，帮助你将 Java Spring Boot 连接到数据库。本章将描述 Spring Data JPA 以及如何在项目中添加 Spring Data JPA 和 Postgres 依赖项。本章还将展示如何使用配置文件将 Java Spring Boot 连接到 Postgres 数据库实例。

*第五章*, *使用 Spring 构建 API*，展示了如何启动和运行 Java Spring Boot 应用。本章还将展示如何为应用添加模型并在编写路由器和控制器时使用它们。之后，本章将解释如何使用 Redis 进行缓存以提高应用性能。

*第六章*, *使用 OpenAPI 规范记录 API*，涵盖了 Java Spring Boot 应用 API 的文档部分。本章还将展示如何在应用中包含 Swagger UI 以提供 API 文档的图形界面。

*第七章*, *使用 JWT 添加 Spring Boot 安全*，详细说明了 CORS 是什么以及如何在 Spring Boot 应用中添加 CORS 策略。本章描述了 Spring 安全、认证和授权。本章还将演示 JSON web tokens 的工作原理以及**身份即服务**（**IaaS**）是什么。

*第八章*, *在 Spring Boot 中记录事件*，解释了什么是日志以及实现日志的流行包。本章还将教你日志的保存位置以及如何处理日志。

*第九章*, *在 Spring Boot 中编写测试*，全部关于为 Java Spring Boot 应用编写测试。本章将描述 JUnit 和 AssertJ。本章还将教你如何编写测试，如何测试存储库，以及如何使用 Mockito 测试服务。

*第十章*, *设置我们的 Angular 项目和架构*，主要关注如何组织特性和模块，如何构建组件，以及如何添加 Angular Material。

*第十一章*, *构建响应式表单*，展示了如何构建响应式表单、基本表单控件和分组表单控件。本章还将解释如何使用 FormBuilder 和验证表单输入。

*第十二章*, *使用 NgRx 管理状态*，涵盖了复杂应用中的状态管理。本章还将介绍 NgRx 以及如何在 Angular 应用中设置和使用它。

*第十三章*, *使用 NgRx 保存、删除和更新*，描述了如何使用 NgRx 删除项目，如何使用 NgRx 添加项目，以及如何使用 NgRx 更新项目。

*第十四章*，*在 Angular 中添加身份验证*，探讨了如何添加用户登录和注销、检索用户配置文件信息、保护应用程序路由以及调用具有受保护端点的 API。

*第十五章*，*在 Angular 中编写测试*，说明了如何编写基本的 Cypress 测试以及如何模拟 HTTP 请求进行测试。

*第十六章*，*使用 Maven 打包后端和前端*，展示了如何利用 Maven 前端插件将 Angular 和 Spring Boot 集成到一个包中。

*第十七章*，*部署 Spring Boot 和 Angular 应用程序*，描述了 CI/CD 和 GitHub Actions。本章还将向您展示如何为 Spring Boot 和 Angular 应用程序创建 CI 工作流程或管道。

# 要充分利用本书

你应该确保你对 HTML、CSS、JavaScript、TypeScript、Java 和 REST API 有基本的了解。你不需要对提到的要求有中级或高级知识。

| **本书涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |
| Node.js（LTS 版本） | Windows、macOS 或 Linux |
| Angular | Windows、macOS 或 Linux |
| Java 17 | Windows、macOS 或 Linux |
| Visual Studio Code | Windows、macOS 或 Linux |
| IntelliJ IDEA | Windows、macOS 或 Linux |
| Google Chrome | Windows、macOS 或 Linux |

如果你在开发应用程序时安装运行时、SDK 或任何软件工具时遇到问题，不要失去希望。错误很常见，但在 Google 上搜索错误信息在解决某些问题时对开发者有很大帮助。

**如果您使用的是本书的数字版，我们建议您亲自输入代码或从本书的 GitHub 仓库（下一节中有一个链接）获取代码。这样做将有助于您避免与代码复制和粘贴相关的任何潜在错误** **。**

在 stackblitz.com 或 codesandbox.io 上尝试使用 Angular，以查看 Angular 的外观和感觉，而无需在您的计算机上安装任何东西。

# 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件：[`github.com/PacktPublishing/Spring-Boot-and-Angular`](https://github.com/PacktPublishing/Spring-Boot-and-Angular)。如果代码有更新，它将在 GitHub 仓库中更新。

我们还有其他来自我们丰富的图书和视频目录的代码包，可在 https://github.com/PacktPublishing/找到。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图和图表的彩色图像 PDF 文件。您可以从这里下载：[`packt.link/pIe6D`](https://packt.link/pIe6D)。

# 使用的约定

本书使用了多种文本约定。

`文本中的代码`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“Spring Boot 只需要`spring-boot-starter-web`，这是一个 Spring Starter，以便我们的应用程序运行。”

代码块设置如下：

```java
@Configuration
public class AppConfig
{
   @Bean
   public Student student() {
       return new Student(grades());
    }
   @Bean
   public Grades grades() {
      return new Grades();
    }
}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
dependencies {
   implementation 'org.springframework.boot:spring-boot-
   starter-data-jpa'
   runtimeOnly 'com.h2database:h2'
   runtimeOnly 'org.postgresql:postgresql'
}
```

任何命令行输入或输出都如下所示：

```java
rpm -ivh jdk-17.interim.update.patch_linux-x64_bin.rpm
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会以粗体显示。以下是一个示例：“选择**Spring Initializr**，这将打开具有相同网页界面的表单。”

小贴士或重要注意事项

看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请通过 customercare@packtpub.com 给我们发邮件，并在邮件主题中提及书名。

**勘误表**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，如果您能向我们报告，我们将不胜感激。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表格。

**盗版**：如果您在互联网上以任何形式发现我们作品的非法副本，如果您能向我们提供位置地址或网站名称，我们将不胜感激。请通过版权@packt.com 与我们联系，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

一旦您阅读了《Spring Boot 3 和 Angular》，我们很乐意听听您的想法！请选择[`www.amazon.in/review/create-review/?asin=180324321X`](https://www.amazon.in/review/create-review/?asin=180324321X)为这本书留下您的反馈。

您的评论对我们和科技社区非常重要，并将帮助我们确保我们提供高质量的内容。

# 下载本书的免费 PDF 副本

感谢您购买本书！

您喜欢在路上阅读，但无法携带您的印刷书籍到处走吗？

您的电子书购买是否与您选择的设备不兼容？

不要担心，现在，每购买一本 Packt 书籍，您都可以免费获得该书的 DRM 免费 PDF 版本。

在任何地方、任何设备上阅读。直接从您最喜欢的技术书籍中搜索、复制和粘贴代码到您的应用程序中。

优惠远不止于此，您还可以获得独家折扣、时事通讯以及每天收件箱中的精彩免费内容。

按照以下简单步骤获取好处：

1.  扫描下面的二维码或访问以下链接

![图片](img/B18159_QR_Free_PDF.jpg)

[`packt.link/free-ebook/9781803243214`](https://packt.link/free-ebook/9781803243214)

1.  提交您的购买证明

1.  就这些！我们将直接将您的免费 PDF 和其他优惠发送到您的电子邮件

# 第一部分：Spring Boot 和 Angular 开发概述

本部分包含一个真实场景，介绍如何启动一个 Web 应用程序项目。以下章节包含在本部分中：

+   *第一章*, *Spring Boot 和 Angular – 全景图*

+   *第二章*, *设置开发环境*
