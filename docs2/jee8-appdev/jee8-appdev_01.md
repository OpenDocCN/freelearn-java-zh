# 第一章：Java EE 简介

**Java Platform, Enterprise Edition** (**Java EE**) 由一组用于开发服务器端、企业 Java 应用程序的 **应用程序编程接口** (**API**) 规范组成。在本章中，我们将提供一个 Java EE 的高级概述。

我们将在本章中介绍以下主题：

+   Java EE 简介

+   一个标准，多个实现

+   Java EE、J2EE 和 Spring 框架

# Java EE 简介

Java 平台，企业版 (Java EE) 是一组 API 规范的集合，旨在在开发服务器端、企业 Java 应用程序时协同工作。Java EE 是一个标准；Java EE 规范有多个实现。这一事实防止了供应商锁定，因为针对 Java EE 规范开发的代码可以部署到任何 Java EE 兼容的应用服务器，只需进行最小或没有修改。

Java EE 在 **Java Community Process** (**JCP**) 下开发，这是一个负责 Java 技术发展的组织。JCP 成员包括 Oracle（Java 平台的当前监护人）以及整个 Java 社区。

# Java community process

**Java Community Process** (**JCP**) 允许感兴趣的相关方协助开发 Java 技术的标准技术规范。公司和个人都可以成为 JCP 的成员，并为任何他们感兴趣的技术规范做出贡献。每个 Java EE API 规范都是作为 **Java Specification Request** (**JSR**) 的一部分开发的。每个 JSR 都分配了一个唯一的编号。例如，**JavaServer Faces** (**JSF**) 2.3 是作为 JSR 372 开发的。

由于 Java EE 在 JCP 下开发，没有一家公司能够完全控制 Java EE 规范，因为，如前所述，JCP 对整个 Java 社区开放，包括软件供应商和感兴趣的个人。

不同的 JCP 成员有不同的兴趣，并为不同的 Java EE 规范做出贡献；结果是 Java EE 由 Java 社区的各个成员共同开发。

# Java EE API

如前所述，Java EE 是一组 API 规范的集合，旨在在开发服务器端企业 Java 应用程序时协同工作。Java EE 8 API 包括：

+   JavaServer Faces (JSF) 2.3

+   Java 持久性 API (JPA) 2.2

+   企业 JavaBeans (EJB) 3.2

+   Java EE 平台上下文和依赖注入 (CDI) 2.0

+   Java API for JSON Processing (JSON-P) 1.1

+   Java API for JSON Binding (JSON-B) 1.0

+   Java API for WebSocket 1.0

+   Java 消息服务 (JMS) 2.0

+   Java EE 安全 API 1.0

+   Java API for RESTful Web Services (JAX-RS) 2.1

+   Java API for XML Web Services (JAX-WS) 2.2

+   Servlet 4.0

+   表达式语言 (EL) 3.0

+   JavaServer Pages (JSP) 2.3

+   Java 命名和目录接口 (JNDI) 1.2

+   Java 事务 API (JTA) 1.2

+   Java 事务服务 (JTS) 1.0

+   JavaMail 1.5

+   Java EE 连接器架构 (JCA) 1.7

+   Java Architecture for XML Binding (JAXB) 2.2

+   Java Management Extensions (JMX) 1.2

+   JavaServer Pages (JSTL) 标准标签库 1.2

+   Bean Validation 2.0

+   管理 Bean 1.0

+   拦截器 1.2

+   Java EE 并发实用工具 1.0

+   Java 平台批处理应用程序 1.0

上述列表是一个规范列表，应用服务器供应商或开源社区需要为每个 Java EE API 规范提供实现。应用服务器供应商然后将一组 Java EE API 实现捆绑在一起，作为其应用服务器产品的一部分。由于每个实现都符合相应的 Java EE JSR，针对一个实现开发的代码可以在任何其他实现上无修改地运行，从而避免供应商锁定。|

由于时间和空间限制，本书不会涵盖每个 Java EE API 规范，而是专注于最流行的 Java EE API。以下表格总结了我们将要覆盖的 API：

| **Java EE API** | **描述** |
| --- | --- |
| **JavaServer Faces** (**JSF**) 2.3 | JSF 是一个组件库，极大地简化了 Web 应用程序的开发。 |
| **Java Persistence API** (**JPA**) 2.2 | JPA 是 Java EE 标准的**对象关系映射** (**ORM**) API。它使得与关系数据库交互变得容易。 |
| **Enterprise JavaBeans** (**EJB**) 3.2 | EJB 允许我们轻松地为 Java EE 应用程序添加企业功能，如事务和可伸缩性。 |
| **Contexts and Dependency Injection** (**CDI**) 2.0 | CDI 允许我们轻松定义 Java 对象的生命周期，并提供将依赖项轻松注入 Java 对象的能力；它还提供了一个强大的事件机制。 |
| Java API for **JSON Processing** (**JSON-P**) 1.1 | JSON-P 是一个允许在 Java 中处理 JSON 字符串的 API。 |
| Java API for **JSON Binding** (**JSON-B**) 1.0 | JSON-B 提供了从 JSON 流中轻松填充 Java 对象以及反向操作的能力。 |
| Java API for WebSocket 1.0 | WebSocket 是一个标准的 Java EE 实现，实现了**互联网工程任务组** (**IETF**)的 WebSocket 协议，它允许通过单个 TCP 连接进行全双工通信。 |
| **Java Message Service** (**JMS**) 2.0 | JMS 是一个标准 API，允许 Java EE 开发者与**面向消息的中间件** (**MOM**)交互。 |
| Java EE Security API 1.0 | Java EE 安全 API 旨在标准化和简化保护 Java EE 应用程序的任务。 |
| **Java API for RESTful Web Services** (**JAX-RS**) 2.1 | JAX-RS 是一个用于创建 RESTful 网络服务端点和客户端的 API。 |
| **Java API for XML Web Services** (**JAX-WS**) 2.2 | JAX-WS 是一个允许创建**简单对象访问协议** (**SOAP**)网络服务的 API。 |
| Servlet 4.0 | Servlet API 是一个用于在 Web 应用程序中实现服务器端逻辑的低级 API。 |

我们还将介绍如何利用标准 Java EE API 开发微服务。微服务是一种现代、流行的架构风格，其中应用程序被分割成独立部署的小模块，通过网络相互交互，通常通过利用 RESTful 网络服务来实现。

我们还应该注意，除了关于微服务章节外，本书的每个章节都是独立的；您可以随意按任何顺序阅读这些章节。

现在我们已经介绍了 Java EE 提供的不同 API，值得重申的是，Java EE 是一个单一标准，有多种实现，其中一些是商业的，一些是开源的。

# 一个标准，多种实现

在其核心，Java EE 是一个规范——如果你愿意的话，可以说是一张纸。Java EE 规范的实现需要被开发，这样应用开发者才能根据 Java EE 标准实际开发服务器端的企业 Java 应用。每个 Java EE API 都有多种实现；例如，流行的 Hibernate 对象关系映射工具就是 Java EE 的 Java 持久化 API (JPA) 的一个实现。然而，它绝不是唯一的 JPA 实现；其他 JPA 实现包括 EclipseLink 和 OpenJPA。同样，每个 Java EE API 规范都有多种实现。

Java EE 应用通常部署到应用服务器；一些流行的应用服务器包括 JBoss、Websphere、Weblogic 和 GlassFish。每个应用服务器都被视为一个 Java EE 实现。应用服务器供应商要么开发自己的一些 Java EE API 规范的实现，要么选择包含现有的实现。

应用开发者通过不受特定 Java EE 实现的约束而受益。只要应用是针对标准 Java EE API 开发的，它应该非常易于跨应用服务器供应商移植。

# Java EE、J2EE 和 Spring 框架

Java EE 早在 2006 年就被引入；Java EE 的第一个版本是 Java EE 5。Java EE 取代了 J2EE；J2EE 的最后一个版本是 J2EE 1.4，于 2003 年发布。尽管 J2EE 可以被认为是一种已死的技术，在 11 年前就被 Java EE 取代，但 J2EE 这个术语却拒绝消失。时至今日，许多人仍然将 Java EE 称为 J2EE；许多公司在他们的网站和招聘板上宣传他们正在寻找“J2EE 开发者”，似乎没有意识到他们所指的已经是一种存在了几年的过时技术。正确的术语，并且长期以来一直是，Java EE。

此外，术语 J2EE 已经成为任何服务器端 Java 技术的“万能”术语；通常 Spring 应用程序也被称作 J2EE 应用程序。Spring 并非 J2EE，也从未是 J2EE；事实上，Spring 是由 Rod Johnson 在 2002 年作为 J2EE 的替代品而创建的。就像 Java EE 一样，Spring 应用程序也经常被错误地称为 J2EE 应用程序。

# 摘要

在本章中，我们提供了 Java EE 的介绍，并列出了 Java EE 中包含的几个技术和应用程序编程接口（API）。

我们还介绍了 Java EE 是如何通过 Java 社区过程由软件供应商和整个 Java 社区共同开发的。

此外，我们解释了 Java EE 标准有多个实现，这一事实避免了供应商锁定，并使我们能够轻松地将我们的 Java EE 代码从一个应用程序服务器迁移到另一个。

最后，我们澄清了 Java EE、J2EE 和 Spring 之间的混淆，解释了尽管 J2EE 已经是一种过时的技术好几年了，Java EE 和 Spring 应用程序通常仍被称为 J2EE 应用程序。
