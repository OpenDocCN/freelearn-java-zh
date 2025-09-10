# 第一章. Java EE 7 的新特性

由于其使用，分布式应用程序需要一些非功能性服务，例如远程访问、安全性、事务管理、并发性和健壮性等。除非你有提供这些类型服务的 API，否则你需要从头开始实现它们，因此会增加错误数量，降低软件质量，并增加生产成本和时间。Java EE 平台旨在让开发者摆脱这些担忧。它由一组促进分布式、健壮、可扩展和互操作应用程序的开发和部署的 API 组成。

自 1999 年首次发布以来，Java EE 平台通过提供比之前版本更新、更丰富、更简单的版本而不断发展。为了让你对 Java EE 7 的改进有一个概述，本章讨论以下主题：

+   Java EE 简史

+   Java EE 7 的主要目标

+   Java EE 7 的创新之处

# Java EE 简史

以前称为 J2EE，Java EE 平台的第一版于 1999 年 12 月正式发布，包含 10 个规范。在这些规范中，有用于数据展示的 Servlets 和 JavaServer Pages (JSP)，用于持久数据管理的 **Enterprise JavaBeans** (**EJB**)，通过 RMI-IIOP (远程方法调用跨 Internet Inter-ORB 协议) 远程访问业务服务，以及用于发送消息的 JMS (Java 消息服务) 规范。

尽管有努力和许多贡献，Java EE 的早期版本过于复杂且难以实现。这导致了大量批评，并引发了像 Spring 框架这样的竞争框架的兴起。

从之前的失败中吸取了教训，该平台在时间上经历了相当大的演变，直到 Java EE 5 的推出，这使得平台能够恢复其失去的声誉。从这个版本开始，Java EE 继续提供更简单、更丰富、更强大的平台版本。

![Java EE 简史](img/9235OT_01_01(a).jpg)

以下图表概述了自 1999 年 12 月发布第一个版本以来对 Java EE 平台所做的重大更改。此图表突出了每个版本的发布日期、更新和主要改进。它还让我们对每个版本背后的中心主题有一个了解。

# Java EE 7 的主要目标

自 2006 年 5 月以来，Java EE 平台在实现方面经历了显著的演变。首先，在 Java EE 5 中，它通过允许将简单的 Java 类（POJO 类）通过注解或 XML 描述转换为业务对象，极大地简化了应用程序的开发。在简化的道路上，Java EE 6 帮助丰富了注解，并引入了诸如修剪、RESTful Web 服务、CDI、EJB Lite、异常配置和 Web 配置文件等新概念。这使得平台能够提供许多易于部署和消费的服务。在 Java EE 6 的成功之后，JCP（Java 社区进程）设想通过提供云支持的基础设施，将平台转变为一个服务。但是，由于相关规范缺乏显著进展，它修订了其目标。正是从为 Java EE 平台向云迁移做准备的角度，Java EE 7 专注于生产力和 HTML5 支持。由于错过了大目标（即迁移到云），它将通过完成 Java EE 6 特性和添加一些新规范来实现其新目标。

## 生产力

Java EE 7 在许多方面提高了生产力。通过简化诸如 JMS 和 JAX-RS 之类的某些 API，Java EE 7 平台显著减少了样板代码。正如您将在接下来的章节中注意到的那样，发送一个 JMS 消息可以只占用一行，并且不再需要创建多个对象，正如 JMS 1.1 的情况，那时首先需要创建一个 `Connection`、`Session`、`MessageProducer` 和 `TextMessage`。

Java EE 7 集成了新的 API，以更好地满足企业应用程序在处理大量数据方面的需求。例如，我们有 **并发实用工具**，它允许在容器内创建托管线程，并使开发者能够将大型过程分解为可以并发计算的小单元。同样，还有一个用于批处理的 Java API，用于管理大量和长时间运行的任务。

最后，Java EE 7 在注解方面得到了丰富，并侧重于异常配置。无论是数据源还是批处理，兼容的 Java EE 7 容器都提供了一系列默认对象。甚至可以仅通过少量配置就生成复杂的应用程序。

简而言之，新平台使开发者免于执行许多任务和创建多种类型的对象，这些对象对于设置应用程序是必需的。

## HTML5 支持

有些人可能会 wonder 为什么对 HTML5 的支持如此重要。答案是简单的：HTML5 是 HTML 标准的最新版本。更重要的是，它提供了新的功能，简化了更强大和适合的 Web 应用程序的构建。例如，通过 HTML5 的`<audio>`和`<video>`元素，您可以在不使用第三方插件（如 Flash）的情况下播放、暂停和恢复音频和视频媒体内容。通过画布元素和 WebGL 库（OpenGL 的一个子集），您可以在网站上轻松集成 2D 和 3D 图形。在客户端和服务器之间的通信方面，HTML5 中 WebSocket 协议的完美集成使我们能够构建具有全双工 P2P 通信的 Web 应用程序，并克服 HTTP 在实时通信中的某些限制。使用此协议，您将不会在实现需要客户端和服务器之间实时通信的聊天应用程序或其他 Web 应用程序时遇到困难，例如交易平台和电子商务平台。在数据交换方面，HTML5 对 JSON 格式的原生支持简化了信息处理并减少了文档的大小。许多其他领域也得到了改进，但在此我们只提及这些。

在所有这些创新的基础上，JSF（JavaServer Faces）中增加了对 HTML5 特性的支持，Java EE 7 平台添加了一个新的 API 来构建 WebSocket 驱动的应用程序，并添加了另一个 API 来处理 JSON 数据格式。

# Java EE 7 的新特性

Java EE 7 是作为一个 Java 规范请求（JSR 342）开发的。它总共有 31 个规范，包括 4 个新规范、10 个主要版本和 9 个 MR（维护版本）。所有这些规范都由 GlassFish Server 4.0（可通过地址[`glassfish.java.net/download.html`](https://glassfish.java.net/download.html)访问）考虑在内，它是 Java EE 7 的参考实现。

Java EE 中引入的新规范如下：

+   Java EE 1.0 并发实用工具([`jcp.org/en/jsr/detail?id=236`](http://jcp.org/en/jsr/detail?id=236))，用于 Java EE 应用程序组件中的异步处理和多线程任务。

+   Java 平台批处理应用程序 1.0 ([`jcp.org/en/jsr/detail?id=352`](http://jcp.org/en/jsr/detail?id=352))，用于执行长时间运行的任务和批量操作。

+   Java API for JSON Processing 1.0 ([`jcp.org/en/jsr/detail?id=353`](http://jcp.org/en/jsr/detail?id=353))，它为 JSON 处理提供支持。它为 Java EE 组件提供了解析、生成、转换和查询 JSON 格式的功能。

+   Java API for WebSocket 1.0 ([`jcp.org/en/jsr/detail?id=356`](http://jcp.org/en/jsr/detail?id=356))，用于构建 WebSocket 应用程序。

经历了重大变更的从 Java EE 6 平台继承的 API 如下：

+   Java 平台，企业版 7 (Java EE 7) 规范 ([`jcp.org/en/jsr/detail?id=342`](http://jcp.org/en/jsr/detail?id=342))，与 Java EE 6 相比，进一步简化了开发，增加了对 HTML5 的支持，并为平台迁移到云做好了准备

+   Java Servlet 3.1 规范 ([`jcp.org/en/jsr/detail?id=340`](http://jcp.org/en/jsr/detail?id=340)) 引入了一些特性，例如非阻塞 I/O API 和协议升级处理

+   表达式语言 3.0 ([`jcp.org/en/jsr/detail?id=341`](http://jcp.org/en/jsr/detail?id=341)) 已从 JSP 规范请求中分离出来，并带来了许多变化，包括用于独立环境的 API、lambda 表达式和集合对象支持

+   JavaServer Faces 2.2 ([`jcp.org/en/jsr/detail?id=344`](http://jcp.org/en/jsr/detail?id=344)) 集成了对 HTML5 标准的支持，并带来了资源库合同、Faces Flow 和无状态视图等特性

+   Java 持久化 2.1 ([`jcp.org/en/jsr/detail?id=338`](http://jcp.org/en/jsr/detail?id=338)) 为我们带来了执行存储过程、运行时创建命名查询、通过 Criteria API 构建批量更新/删除、运行时覆盖或更改获取设置以及像 SQL 一样进行显式连接的机会

+   企业 JavaBeans 3.2 ([`jcp.org/en/jsr/detail?id=345`](http://jcp.org/en/jsr/detail?id=345)) 引入了手动禁用有状态会话 Bean 消息化的能力，并且放宽了定义默认本地或远程业务接口的规则

+   Java 消息服务 2.0 ([`jcp.org/en/jsr/detail?id=343`](http://jcp.org/en/jsr/detail?id=343)) 简化了 API

+   JAX-RS 2.0：Java RESTful Web 服务 API ([`jcp.org/en/jsr/detail?id=339`](http://jcp.org/en/jsr/detail?id=339)) 简化了 RESTful Web 服务的实现，并引入了包括客户端 API、异步处理、过滤器和中继器等新特性

+   Java EE 1.1 的上下文和依赖注入 ([`jcp.org/en/jsr/detail?id=346`](http://jcp.org/en/jsr/detail?id=346)) 引入了许多变化，其中一些是访问当前 CDI 容器、访问 Bean 的非上下文实例以及显式销毁 Bean 实例的能力

+   Bean 验证 1.1 ([`jcp.org/en/jsr/detail?id=349`](http://jcp.org/en/jsr/detail?id=349)) 引入了方法构造函数验证、分组转换和使用表达式语言的消息插值支持

只有以下 API 受维护版本的影响：

+   Java EE 1.4 中的 Web 服务 ([`jcp.org/en/jsr/detail?id=109`](http://jcp.org/en/jsr/detail?id=109))

+   Java 容器授权服务提供者合同 1.5 (JACC 1.5) ([`jcp.org/en/jsr/detail?id=115`](http://jcp.org/en/jsr/detail?id=115))

+   Java 容器认证服务提供者接口 1.1 (JASPIC 1.1) ([`jcp.org/en/jsr/detail?id=196`](http://jcp.org/en/jsr/detail?id=196)) 标准化了规范的一些方面的使用

+   JavaServer Pages 2.3 ([`jcp.org/en/jsr/detail?id=245`](http://jcp.org/en/jsr/detail?id=245))

+   Java 平台通用注解 1.2 ([`jcp.org/en/jsr/detail?id=250`](http://jcp.org/en/jsr/detail?id=250)) 添加了一个用于管理优先级的新的注解

+   拦截器 1.2 ([`jcp.org/en/jsr/detail?id=318`](http://jcp.org/en/jsr/detail?id=318)) 添加了用于管理拦截器执行顺序的标准注解

+   Java EE 连接器架构 1.7 ([`jcp.org/en/jsr/detail?id=322`](http://jcp.org/en/jsr/detail?id=322)) 为定义和配置资源适配器的资源添加了两个注解

+   Java 事务 API 1.2 ([`jcp.org/en/jsr/detail?id=907`](http://jcp.org/en/jsr/detail?id=907)) 提供了声明性标记事务的可能性和定义与当前事务生命周期相同的 bean

+   JavaMail 1.5 ([`jcp.org/en/jsr/detail?id=919`](http://jcp.org/en/jsr/detail?id=919)) 通过添加注解和方法略微简化了发送电子邮件的开发

# 摘要

在简要介绍 Java EE 的演变并分析最新平台的目标后，我们列出了在 Java EE 7 中改进或添加的所有规范。在下一章中，我们将重点关注新规范，以突出其有用性并展示它们如何实现。
