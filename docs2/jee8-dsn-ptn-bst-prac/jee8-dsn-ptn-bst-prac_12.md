# MicroProfile

本章将概述 Eclipse MicroProfile 项目，包括其目标、预期成果、何时使用以及使用它来开发我们的应用程序的好处。本章仅是一个概述，我们不会探讨如何使用 MicroProfile 项目实现应用程序。

以下将是本章将涵盖的中心主题：

+   解释 Eclipse MicroProfile 项目方法

# 解释 Eclipse MicroProfile 项目方法

现在，微服务架构的使用正在迅速增加。出现了新的工具，以促进和开发使用 MicroProfile 模式和最佳实践的微服务应用程序。Eclipse MicroProfile 项目的创建是为了使使用 Java EE 的力量来优化企业 Java 以适应微服务架构成为可能。

Eclipse MicroProfile 项目是一系列规范，用于使用微服务架构开发应用程序。这个伞形项目包含一些 Java EE 规范和一些专有规范（由 Eclipse MicroProfile 创建）。因此，Eclipse MicroProfile 项目允许我们使用 Java EE 规范，如 CDI、JAX-RS 和 JSON-B，来开发微服务应用程序。此外，MicroProfile 项目在多个企业 Java 运行时之间提供可移植的微服务架构，并允许互操作微服务架构，这允许多语言运行时（不仅仅是 Java）之间的通信。到目前为止，Eclipse MicroProfile 项目的当前版本是 2.0，具有以下规范：

+   Eclipse MicroProfile Config 1.3

+   Eclipse MicroProfile 故障转移 1.1

+   Eclipse MicroProfile 健康检查 1.0

+   Eclipse MicroProfile JWT 认证 1.1

+   Eclipse MicroProfile Metrics 1.1

+   Eclipse MicroProfile OpenAPI 1.0

+   Eclipse MicroProfile OpenTracing 1.1

+   Eclipse MicroProfile Rest 客户端 1.1

+   CDI 2.0

+   常见注解 1.3

+   JAX-RS 2.1

+   JSON-B 1.0

+   JSON-P 1.1

MicroProfile 项目被设计为可以在多个运行时之间移植，但只有当我们使用前面的规范来开发它时，它才是可移植的。如果我们使用额外的 API 或框架，则可移植性不能保证。此外，MicroProfile 项目支持 Java 8，但不支持 Java 7 及更早版本。

# Eclipse MicroProfile Config 1.3

应用程序需要从相同的内部或外部位置读取配置。Eclipse MicroProfile Config 1.3 使得从一些内部或外部来源获取配置属性成为可能，这些属性通过依赖注入或查找提供。通过这种方式，我们可以实现外部配置存储模式并消费配置属性。配置的格式可以是系统属性、系统环境变量、`.properties`、`.xml` 或数据源。

# Eclipse MicroProfile 故障转移 1.1

使用微服务架构的应用程序需要具备容错能力。容错是关于利用不同的策略来引导逻辑的执行和结果。Eclipse MicroProfile 容错 1.1 提供了将业务逻辑与执行逻辑解耦的能力，分离这些逻辑。有了这个功能，我们可以处理 API，如 TimeOut、RetryPolicy、Fallback、Bulkhead 和 Circuit Breaker，这些都是与微服务一起使用的最流行概念。

# Eclipse MicroProfile 健康检查 1.0

在微服务架构中，当涉及到检测故障时，提升服务健康检查的能力非常重要。Eclipse MicroProfile 健康检查 1.0 提供了从另一台机器验证计算节点状态的能力。有了这个功能，我们可以使用 MicroProfile 项目功能实现健康端点监控模式，如第十一章[2f8a0a53-0ca6-4f8f-8248-d62db3021f4c.xhtml]中所述的*操作模式*。

# Eclipse MicroProfile JWT 身份验证 1.1

今天，微服务架构最常使用的是 RESTful 架构风格。然而，RESTful 架构风格与无状态服务协同工作，并且不提供自身的安全性。因此，最常见的做法是创建基于令牌的安全服务。Eclipse MicroProfile JWT 身份验证 1.1 提供了**基于角色的访问控制**（**RBAC**），使用**OpenID Connect**（**OIDC**）的微服务端点，以及**JSON Web Tokens**（**JWT**）。

# Eclipse MicroProfile 指标 1.1

指标对于微服务架构非常重要。这是因为它们使我们能够更好地评估我们的服务。Eclipse MicroProfile 指标 1.1 为 MicroProfile 服务器提供了一个统一的出口，将监控数据导出到管理代理。指标还将提供一个通用的 Java API，用于公开它们的遥测数据。这个功能可以用来生成有关应用程序的信息，使得能够实现健康端点监控模式，以便更好地评估我们的服务。

# Eclipse MicroProfile OpenAPI 1.0

当我们开发微服务应用程序时，我们创建相互通信的服务。因此，API 文档对于通过不了解合同来最小化错误非常重要。因此，Eclipse MicroProfile OpenAPI 1.0 提供了一个统一的 Java API，用于 OpenAPI v3 规范，所有应用程序开发者都可以使用它来公开他们的 API 文档。因此，我们可以创建服务之间的合同，并且 API 被公开。

# Eclipse MicroProfile OpenTracing 1.1

在微服务应用程序中，分析或调试操作工作流是困难的。为了便于调试，我们可以使用 OpenTracing。OpenTracing 是一个用于分布式跟踪的微服务仪器化标准 API，它通过检查和记录请求在分布式系统中传播来帮助调试微服务。

Eclipse MicroProfile OpenTracing 1.1 定义了一个 API 和相关的行为，允许服务轻松参与分布式跟踪环境。有了这个，我们可以分析或调试操作的工作流程，而无需添加任何代码以完成跟踪。

# Eclipse MicroProfile REST Client 1.1

在微服务架构中，通信通常是通过 HTTP 协议使用 RESTful 实现的。然后我们需要创建一个客户端，该客户端发送遵守定义合同的 HTTP 请求。在这种情况下，Eclipse MicroProfile REST 客户端提供了一种类型安全的调用 HTTP 上 RESTful 服务的途径。MicroProfile REST 客户端基于 JAX-RS 2.1 API，以确保一致性和易用性。

# CDI 2.0

MicroProfile 项目在其内部嵌入了一个 Java EE 规范。其中之一是 CDI，它为 MicroProfile 2.0 中包含的越来越多的 API 提供了基础。自 MicroProfile 2.0 版本起，允许使用 CDI 2.0 以外的实现，但不是必需的。

# 常见注释 1.3

MicroProfile 项目在其内部嵌入了一些 Java EE 规范。其中之一是常见注释，它为 Java SE 和 Java EE 平台上的各种个别技术提供了跨多种语义概念的注释。自 MicroProfile 2.0 版本起，允许使用 Common annotations 1.3 以外的实现，但不是必需的。

# JAX-RS 2.1

Eclipse MicroProfile 项目在其内部嵌入了一些 Java EE 规范。其中之一是 JAX-RS，它为 MicroProfile 2.0 应用程序提供了标准客户端和服务器 API，用于 RESTful 通信。自 MicroProfile 2.0 版本起，允许使用 JAX-RS 2.1 以外的实现，但不是必需的。

# JSON-B 1.0

MicroProfile 项目中嵌入的另一个 Java EE 规范是 JSON-B，它作为 MicroProfile 2.0 的一部分包含在内，旨在提供将 JSON 文档绑定到 Java 代码的标准 API。自 MicroProfile 2.0 版本起，允许使用 JSON-B 1.0 以外的实现，但不是必需的。

# JSON-P 1.1

MicroProfile 项目中嵌入的另一个 Java EE 规范是 JSON-P，它是 MicroProfile 2.0 的一部分，旨在提供处理 JSON 文档的标准 API。自 MicroProfile 2.0 版本起，允许使用 JSON-P 1.1 以外的实现，但不是必需的。

# 为什么我们应该使用 MicroProfile 项目？

使用 MicroProfile 项目，我们可以利用 Java EE 的特性创建具有微服务架构的应用程序，并在 Java EE 运行时上运行。此外，使用 MicroProfiles 意味着我们能够创建具有指标、容错、文档、应用程序的独立配置以及易于调试等特性的应用程序，包括将业务代码与这些特性的代码解耦。

Java EE 是一套经过广泛测试的规范。使用这些规范来创建具有微服务架构的应用程序的想法非常有益；使用规范使用户免受供应商的约束，同时也使应用程序能够在不同的运行时之间移植。

# 社区

Eclipse MicroProfile 项目由 MicroProfile 社区（[`microprofile.io/`](http://microprofile.io/））维护，这是一个半新的社区。这个群体中的主要参与者包括 IBM、Red Hat、Tomitribe、Payara、**伦敦 Java 社区**（**LJC**）和 SouJava。这个项目的目标是成为一个基于社区的项目。它还旨在在短时间内迭代和创新，获得社区批准，发布新版本，并重复这个过程。随着使用微服务架构开发的应用程序的增长，MicroProfile 的使用将增加，还将引入其他功能，最终增强 MicroProfile 的能力。

# 未来工作

Eclipse MicroProfile 项目支持了一些 Java EE 规范，但目标是扩大这种支持并添加其他 Java EE 规范。预期以下规范将是下一个被纳入 MicroProfile 项目的：

+   JCache

+   持久化

+   Bean 验证

+   WebSockets

# 摘要

在本章中，我们概述了 Eclipse MicroProfile 项目。在这个概述中，我们讨论了 Eclipse MicroProfile 项目是什么，它的规范是什么，为什么我们使用 MicroProfile 项目，以及 Eclipse MicroProfile 项目的未来发展方向。

本章仅作为一个概述，并不旨在教我们如何使用 MicroProfile。如果您想了解更多关于 Eclipse MicroProfile 项目的信息，请访问 [https://projects.eclipse.org/projects/technology.microprofile/releases/microprofile-2.0](https://projects.eclipse.org/projects/technology.microprofile/releases/microprofile-2.0)。
