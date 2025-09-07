# 前言

这本书是深入使用 Spring 6 和 Spring Boot 3 进行 Web 开发的指南。Spring 是一个强大且广泛使用的框架，用于在 Java 中构建可扩展且可靠的 Web 应用程序。Spring Boot 是框架的一个流行扩展，简化了基于 Spring 的应用程序的设置和配置。本书将教你如何使用这些技术构建现代且健壮的 Web API 和服务。

本书涵盖了 API 开发所必需的广泛主题，例如 REST/GraphQL/gRPC 的基础、Spring 概念以及 API 规范和实现。此外，本书还涵盖了异步 API 设计、安全性、设计用户界面、测试 API 和 Web 服务的部署等主题。本书提供了一个高度情境化的真实世界示例应用程序，读者可以用它作为参考来构建针对真实世界应用程序的不同类型的 API，包括持久化数据库层。本书采用的方法是指导读者通过 API 开发的整个开发周期，包括设计规范、实现、测试和部署。

到这本书的结尾，你将学会如何使用 Spring 6 和 Spring Boot 3 设计、开发、测试和部署可扩展且易于维护的现代 API，同时了解确保应用程序安全性和可靠性的最佳实践，以及提高应用程序功能性的实用想法。

# 本书面向对象

本书面向初学者 Java 程序员、近期计算机科学毕业生、编码训练营校友以及新接触创建真实世界 Web API 和服务的专业人士。对于希望转向 Web 开发并寻求 Web 服务开发全面介绍的 Java 开发者来说，本书也是一个宝贵的资源。理想的读者应具备 Java 基础编程结构、数据结构和算法的知识，但缺乏作为 Web 开发者开始工作的实际 Web 开发经验。

# 本书涵盖内容

*第一章*，*RESTful Web 服务基础*，将引导你了解 RESTful API 的基础知识，或简称为 REST API，以及它们的设计范式。这些基础知识将为你开发 RESTful Web 服务提供一个坚实的基础。你还将了解设计 API 的最佳实践。本章还将介绍本书将使用的示例电子商务应用程序，在学习 API 开发不同方面时将用到它。

*第二章*，*Spring 概念和 REST API*，探讨了 Spring 基本原理和特性，这些是使用 Spring 框架实现 REST、gRPC 和 GraphQL API 所必需的。这将为你开发示例电子商务应用程序提供所需的技术视角。

*第三章*, *API 规范和实现*，利用 OpenAPI 和 Spring 实现 REST API。我们选择了先设计后实现的设计方法。您将使用 OpenAPI 规范首先设计 API，然后实现它们。您还将学习如何处理请求服务过程中发生的错误。在这里，示例电子商务应用的 API 将被设计和实现以供参考。

*第四章*, *为 API 编写业务逻辑*，帮助您在 H2 数据库中实现 API 的代码，以及数据持久化。您将编写服务和存储库以进行实现。您还将向 API 响应添加超媒体和 ETag 头，以实现最佳性能和缓存。

*第五章*, *异步 API 设计*，涵盖了异步或响应式 API 设计，其中调用将是异步和非阻塞的。我们将使用基于 Project Reactor（[`projectreactor.io`](https://projectreactor.io)）的 Spring WebFlux 来开发这些 API。首先，我们将了解响应式编程的基础知识，然后将现有的电子商务 REST API（上一章的代码）迁移到异步（响应式）API，通过关联和比较现有的（命令式）编程方式和响应式编程方式来简化事情。

*第六章*, *使用授权和身份验证保护 REST 端点*，解释了您如何使用 Spring Security 来保护这些 REST 端点。您将为 REST 端点实现基于令牌的认证和授权。成功的认证将提供两种类型的令牌——`Admin`、`User`等。这些角色将用作授权，以确保只有用户拥有特定角色时才能访问 REST 端点。我们还将简要讨论**跨站请求伪造**（**CSRF**）和**跨源资源共享**（**CORS**）。

*第七章*, *设计用户界面*，总结了在线购物应用不同层之间的端到端开发和通信。这个 UI 应用将包括`登录`、`产品列表`、`产品详情`、`购物车`和`订单列表`。到本章结束时，您将了解使用 React 进行 SPA 和 UI 组件开发，以及使用浏览器的内置 Fetch API 消费 REST API。

*第八章*, *测试 API*，介绍了 API 的手动和自动化测试。您将了解单元和集成测试自动化。在本章学习自动化之后，您将能够使这两种测试成为构建的组成部分。您还将设置 Java 代码覆盖率工具来计算不同的代码覆盖率指标。

*第九章*，*Web 服务的部署*，解释了容器化、Docker 和 Kubernetes 的基础知识。然后，你将使用这个概念使用 Docker 容器化示例电子商务应用程序。然后，这个容器将在 Kubernetes 集群中部署。你将使用 minikube 进行 Kubernetes，这使得学习和基于 Kubernetes 的开发更加容易。

*第十章*，*开始使用 gRPC*，介绍了 gRPC 的基础知识。

*第十一章*，*gRPC API 开发与测试*，实现了基于 gRPC 的 API。你将学习如何编写 gRPC 服务器和客户端，以及编写基于 gRPC 的 API。在第章的后半部分，你将介绍微服务以及它们如何帮助你设计现代、可扩展的架构。在这里，你将进行两个服务的实现——一个 gRPC 服务器和一个 gRPC 客户端。

*第十二章*，*向服务添加日志和跟踪*，探讨了名为 **Elasticsearch、Logstash、Kibana** （**ELK**）堆栈和 Zipkin 的日志和监控工具。然后，这些工具将被用于实现 API 调用的请求/响应的分布式日志和跟踪。你将学习如何发布和分析不同请求的日志以及与响应相关的日志。你还将使用 Zipkin 来监控 API 调用的性能。

*第十三章*，*开始使用 GraphQL*，讨论了 GraphQL 的基础知识——**模式定义语言**（**SDL**）、查询、突变和订阅。这些知识将有助于你在下一章中实现基于 GraphQL 的 API。在本章的整个过程中，你将了解 GraphQL 模式的基础知识以及解决 N+1 问题。

*第十四章*，*GraphQL API 开发与测试*，解释了基于 GraphQL 的 API 开发及其测试。在本章中，你将为示例应用程序实现基于 GraphQL 的 API。将基于设计优先的方法开发 GraphQL 服务器实现。

# 为了充分利用本书

确保你拥有以下硬件和软件：

| **本书涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |
| Java 17 | Windows、macOS 或 Linux（任何） |
| 任何 Java IDE，如 Netbeans、IntelliJ 或 Eclipse | 连接到互联网以从 GitHub 克隆代码并下载依赖项和库 |
| Docker |  |
| Kubernetes（minikube） |  |
| cURL 或任何 API 客户端，如 Insomnia |  |
| Node 18.x |  |
| VS Code |  |
| ELK 堆栈和 Zipkin |  |

每章都将包含安装所需工具的特殊说明（如果适用）。

**如果您使用的是本书的数字版，我们建议您亲自输入代码或从书籍的 GitHub 仓库（下一节中提供链接）获取代码。这样做将帮助您避免与代码的复制和粘贴相关的任何潜在错误。**

# 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件，网址为[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3)。如果代码有更新，它将在 GitHub 仓库中更新。

我们还有其他来自我们丰富的书籍和视频目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

# 使用的约定

本书使用了多种文本约定。

`文本中的代码`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“如果我们使用模型中的`Link`，则生成的模型将使用映射的`org.springframework.hateoas.Link`类，而不是 YAML 文件中定义的模型。”

代码块设置为以下格式：

```java
 const Footer = () => {   return (
     <div>
       <footer
         className="text-center p-2 border-t-2 bggray-
           200 border-gray-300 text-sm">
         No &copy; by Ecommerce App.{" "}
         <a href=https://github.com/PacktPublishing/Modern- 
           API-Development-with-Spring-and-Spring-Boot>
           Modern API development with Spring and Spring Boot
         </a>
       </footer>
     </div>
   );
 };
 export default Footer;
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
<Error>  <errorCode>PACKT-0001</errorCode>
  <message>The system is unable to complete the request. 
      Contact system support.</message>
  <status>500</status>
  <url>http://localhost:8080/api/v1/carts/1</url>
  <reqMethod>GET</reqMethod>
</Error>
```

任何命令行输入或输出都按以下方式编写：

```java
$ curl --request POST 'http://localhost:8080/api/v1/carts/1/items' \ --header 'Content-Type: application/json' \ 
 --header 'Accept: application/json' \ 
 --data-raw '{ 
 "id": "1", 
 "quantity": 1, 
 "unitPrice": 2.5 
 }'
[]
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词以**粗体**显示。以下是一个示例：“从**管理**面板中选择**系统信息**。”

小贴士或重要注意事项

看起来是这样的。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请通过 mailto:customercare@packtpub.com 给我们发邮件，并在邮件主题中提及书籍标题。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一点。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表格。

**盗版**：如果您在互联网上以任何形式遇到我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过 mailto:copyright@packt.com 与我们联系，并在邮件主题中提及书籍标题。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

一旦你阅读了《使用 Spring 6 和 Spring Boot 3 的现代 API 开发》，我们很乐意听听你的想法！请[点击此处直接进入亚马逊评论页面](https://packt.link/r/1-804-61327-4)并分享你的反馈。

你的评论对我们和科技社区非常重要，并将帮助我们确保我们提供高质量的内容。

# 下载这本书的免费 PDF 副本。

感谢您购买这本书！

你喜欢在路上阅读，但无法携带你的印刷书籍到处走吗？你的电子书购买是否与你的选择设备不兼容？

别担心，现在，每购买一本 Packt 书籍，你都可以免费获得该书的 DRM 免费 PDF 版本。

在任何地方、任何时间、任何设备上阅读。从你最喜欢的技术书籍中搜索、复制和粘贴代码直接到你的应用程序中。

优惠远不止这些，你还可以获得独家折扣、时事通讯和每日免费内容的每日邮箱访问权限。

按照以下简单步骤获取福利：

1.  扫描二维码或访问以下链接。

![二维码](img/B19349_QR_Free_PDF.jpg)

[`packt.link/free-ebook/9781804613276`](https://packt.link/free-ebook/9781804613276)

1.  提交你的购买证明

1.  就这些了！我们将直接将你的免费 PDF 和其他福利发送到你的邮箱。

# 第一部分 – RESTful Web 服务

在这部分，你将开发和测试由 HATEOAS 和 ETags 支持的、适用于生产的不断发展的基于 REST 的 API。API 规范将使用 OpenAPI 规范（Swagger）编写。你将学习使用 Spring WebFlux 进行反应式 API 开发的基础知识。到这部分结束时，你将了解 REST API 的基础知识、最佳实践以及如何编写不断发展的 API。完成这部分后，你将能够开发同步和异步（反应式）的非阻塞 API。

本部分包含以下章节：

+   *第一章*，*RESTful Web 服务基础*

+   *第二章*，*Spring 概念和 REST API*

+   *第三章*，*API 规范和实现*

+   *第四章*，*为 API 编写业务逻辑*

+   *第五章*，*异步 API 设计*
