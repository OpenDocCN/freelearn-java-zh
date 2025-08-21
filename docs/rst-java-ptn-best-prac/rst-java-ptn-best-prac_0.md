# 前言

社交网络、云计算和移动应用程序时代的融合，创造了一代新兴技术，使不同的网络设备能够通过互联网相互通信。过去，构建解决方案有传统和专有的方法，涵盖了不同的设备和组件在不可靠的网络或通过互联网相互通信。一些方法，如 RPC CORBA 和基于 SOAP 的 Web 服务，作为面向服务的体系结构（SOA）的不同实现而演变，需要组件之间更紧密的耦合以及更大的集成复杂性。

随着技术格局的演变，今天的应用程序建立在生产和消费 API 的概念上，而不是使用调用服务并生成网页的 Web 框架。这种基于 API 的架构实现了敏捷开发、更容易的采用和普及，以及与企业内外应用程序的规模和集成。

REST 和 JSON 的广泛采用打开了应用程序吸收和利用其他应用程序功能的可能性。REST 的流行主要是因为它能够构建轻量级、简单和成本效益的模块化接口，可以被各种客户端使用。

移动应用程序的出现要求更严格的客户端-服务器模型。在 iOS 和 Android 平台上构建应用程序的公司可以使用基于 REST 的 API，并通过结合来自多个平台的数据来扩展和加深其影响，因为 REST 基于 API 的架构。

REST 具有无状态的额外好处，有助于扩展性、可见性和可靠性，同时也是平台和语言无关的。许多公司正在采用 OAuth 2.0 进行安全和令牌管理。

本书旨在为热心读者提供 REST 架构风格的概述，重点介绍所有提到的主题，然后深入探讨构建轻量级、可扩展、可靠和高可用的 RESTful 服务的最佳实践和常用模式。

# 本书涵盖的内容

《第一章》*REST - 起源*，从 REST 的基本概念开始，介绍了如何设计 RESTful 服务以及围绕设计 REST 资源的最佳实践。它涵盖了 JAX-RS 2.0 API 在 Java 中构建 RESTful 服务。

《第二章》*资源设计*，讨论了不同的请求响应模式；涵盖了内容协商、资源版本控制以及 REST 中的响应代码等主题。

《第三章》*安全和可追溯性*，涵盖了关于 REST API 的安全和可追溯性的高级细节。其中包括访问控制、OAuth 身份验证、异常处理以及审计和验证模式等主题。

《第四章》*性能设计*，涵盖了性能所需的设计原则。它讨论了 REST 中的缓存原则、异步和长时间运行的作业，以及如何使用部分更新。

《第五章》*高级设计原则*，涵盖了高级主题，如速率限制、响应分页以及国际化和本地化原则，并提供了详细的示例。它涵盖了可扩展性、HATEOAS 以及测试和文档化 REST 服务等主题。

第六章*新兴标准和 REST 的未来*，涵盖了使用 WebHooks、WebSockets、PuSH 和服务器发送事件服务的实时 API，并在各个领域进行了比较和对比。此外，本章还涵盖了案例研究，展示了新兴技术如 WebSockets 和 WebHooks 在实时应用中的使用。它还概述了 REST 在微服务中的作用。

附录涵盖了来自 GitHub、Twitter 和 Facebook 的不同 REST API，以及它们如何与第二章*资源设计*中讨论的原则联系起来，一直到第五章*高级设计原则*。

# 您需要什么来阅读这本书

为了能够构建和运行本书提供的示例，您需要以下内容：

+   Apache Maven 3.0 及更高版本：Maven 用于构建示例。您可以从[`maven.apache.org/download.cgi`](http://maven.apache.org/download.cgi)下载 Apache Maven。

+   GlassFish Server Open Source Edition v4.0：这是一个免费的社区支持的应用服务器，提供了 Java EE 7 规范的实现。您可以从[`dlc.sun.com.edgesuite.net/glassfish/4.0/promoted/`](http://dlc.sun.com.edgesuite.net/glassfish/4.0/promoted/)下载 GlassFish 服务器。

# 这本书是为谁准备的

这本书是应用程序开发人员熟悉 REST 的完美阅读来源。它深入探讨了细节、最佳实践和常用的 REST 模式，以及 Facebook、Twitter、PayPal、GitHub、Stripe 和其他公司如何使用 RESTful 服务实现解决方案的见解。

# 约定

在这本书中，您会发现许多不同类型信息的文本样式。以下是一些这些样式的例子，以及它们的含义解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下："`GET`和`HEAD`是安全方法。"

代码块设置如下：

```java
    @GET
    @Path("orders")
    public List<Coffee> getOrders() {
        return coffeeService.getOrders();    }
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目以粗体设置：

```java
@Path("v1/coffees")
public class CoffeesResource {
    @GET
    @Path("orders")
    @Produces(MediaType.APPLICATION_JSON)
    public List<Coffee> getCoffeeList( ){
      //Implementation goes here

    }
```

任何命令行输入或输出都是这样写的：

```java
#  curl -X GET http://api.test.com/baristashop/v1.1/coffees

```

**新术语**和**重要单词**以粗体显示。

### 注意

警告或重要说明出现在这样的框中。

### 提示

提示和技巧看起来像这样。
