

# 第一章：Jakarta EE 简介

Jakarta EE 由一组用于开发服务器端企业 Java 应用程序的 **应用程序编程接口 (API**) 规范组成。本书的大部分章节将涵盖单个 Jakarta EE 规范，例如用于集成应用程序不同部分的 **上下文和依赖注入 (CDI**)，或用于开发 RESTful Web 服务的 Jakarta RESTful Web 服务。我们还涵盖了用于处理 JSON 格式数据的 Jakarta EE API，以及用于开发基于 Web 的用户界面的 Jakarta Faces。我们还深入探讨了如何与关系数据库交互，实现 Web 应用程序中客户端和服务器之间的双向通信，安全性和消息传递。

在本章中，我们将涵盖以下主题：

+   Jakarta EE 简介

+   Jakarta EE、Java EE、J2EE 和 Spring 框架

# Jakarta EE 简介

Jakarta EE 是一组 API 规范的集合，旨在在开发服务器端企业 Java 应用程序时协同工作。Jakarta EE 是一个有多种实现的标准。这一事实防止了供应商锁定，因为针对 Jakarta EE 规范开发的代码可以在任何符合 Jakarta EE 规范的实现中部署。

Jakarta EE 是一个 **Eclipse 软件基金会** 项目。由于 Jakarta EE 规范是开源的，任何希望贡献的组织或个人都可以自由地这样做。

## 贡献给 Jakarta EE

有许多贡献的方式，包括参与讨论并为 Jakarta EE 的未来版本提供建议。为此，只需订阅适当的邮件列表即可，这可以通过访问 [`jakarta.ee/connect/mailing-lists/`](https://jakarta.ee/connect/mailing-lists/) 来完成。

为了订阅邮件列表，您需要在 [`accounts.eclipse.org/user/register`](https://accounts.eclipse.org/user/register) 创建一个免费的 Eclipse.org 账户。

要超越参与讨论并真正贡献代码或文档，必须签署 Eclipse 贡献者协议。Eclipse 贡献者协议可以在 [`www.eclipse.org/legal/ECA.php`](https://www.eclipse.org/legal/ECA.php) 找到。

## Jakarta EE API

如前所述，Jakarta EE 是一组 API 规范的集合，旨在在开发服务器端企业 Java 应用程序时协同工作。

除了完整的 Jakarta EE 平台外，还有两个 Jakarta EE 配置文件，它们包含完整平台中包含的部分规范和 API。**Jakarta EE 网络配置文件**包含适合开发 Web 应用的部分规范和 API。**Jakarta EE 核心配置文件**包含更小的部分规范和 API，更适合开发微服务。

Jakarta EE 核心配置文件 API 包括以下内容：

+   Jakarta Context 和依赖注入轻量级 (CDI Lite)

+   Jakarta RESTful Web 服务

+   Jakarta JSON 处理

+   Jakarta JSON 绑定

核心配置文件中包含的 Contexts and Dependency Injection API 版本是完整规范的子集。Jakarta EE Web Profile API 包括完整的 CDI 规范而不是 CDI Lite，以及核心配置文件中的所有其他规范和 API，以及一些额外的 API：

+   Jakarta 上下文和依赖注入

+   Jakarta Faces

+   Jakarta Persistence

+   Jakarta WebSocket

+   Jakarta 安全性

+   Jakarta Servlet

+   Jakarta 企业 Bean Lite

Web Profile 中包含的 Jakarta 企业 Bean 版本是完整企业 Bean 规范的子集。

完整的 Jakarta EE 平台包括完整的企业 Bean 规范，以及 Web Profile 中包含的所有其他规范和 API，以及一些额外的 API：

+   Jakarta 企业 Bean

+   Jakarta 消息传递

+   Jakarta 企业 Web 服务

Jakarta EE API 的完整列表

前面的列表并不全面，仅列出了一些最流行的 Jakarta EE API。要获取 Jakarta EE API 的完整列表，请参阅[`jakarta.ee/specifications/`](https://jakarta.ee/specifications/)。

应用服务器供应商或开源社区需要为前面列表中的每个 Jakarta EE API 规范提供兼容的实现。

## 一个标准，多个实现

在其核心，Jakarta EE 是一个规范，如果你愿意，可以说是一张纸。Jakarta EE 规范的实现需要开发，以便应用开发者可以针对 Jakarta EE 标准开发服务器端企业 Java 应用程序。

每个 Jakarta EE API 可以有多个实现。例如，流行的 Hibernate 对象关系映射工具是 Jakarta Persistence 的实现，但绝不是唯一的。其他 Jakarta Persistence 实现包括 EclipseLink 和 Open JPA。同样，每个 Jakarta EE 规范都有多个实现。

Jakarta EE 应用程序通常部署到应用服务器。一些流行的应用服务器包括 JBoss、Websphere、Weblogic 和 GlassFish。每个应用服务器都被视为一个 Jakarta EE 实现。应用服务器供应商要么开发自己的一些 Jakarta EE 规范的实现，要么选择包含现有的实现。

应用开发者通过不受特定 Jakarta EE 实现的限制而受益。只要应用程序是针对标准 Jakarta EE API 开发的，它应该非常易于在不同应用服务器供应商之间移植。

应用服务器供应商然后将一组 Jakarta EE API 实现捆绑在一起，作为其应用服务器产品的一部分。由于每个实现都符合相应的 Jakarta EE 规范，针对一个实现开发的代码可以在任何其他实现上无修改地运行，从而避免供应商锁定。

以下表格列出了一些流行的 Jakarta EE 实现：

| **Jakarta** **EE 实现** | **供应商** | **许可** | **URL** |
| --- | --- | --- | --- |
| Apache Tomee | Tomitribe | Apache License, Version 2.0 | [`tomee.apache.org/`](https://tomee.apache.org/) |
| Eclipse GlassFish | Eclipse Foundation | Eclipse Public License - v 2.0 | [`glassfish.org/`](https://glassfish.org/) |
| IBM Websphere Liberty | IBM | 商业 | [`www.ibm.com/products/websphere-liberty`](https://www.ibm.com/products/websphere-liberty) |
| JBoss Enterprise Application Platform | Red Hat | 商业 | [`www.redhat.com/en/technologies/jboss-middleware/application-platform`](https://www.redhat.com/en/technologies/jboss-middleware/application-platform) |
| Open Liberty | IBM | Eclipse Public License 2.0 | [`openliberty.io/`](https://openliberty.io/) |
| Payara Server Community | Payara Services Ltd | 双许可：CDDL 1.1 / GPL v2 + Classpath Exception | [`www.payara.fish/products/payara-platform-community/`](https://www.payara.fish/products/payara-platform-community/) |
| Payara Server Enterprise | Payara Services Ltd | 商业 | [`www.payara.fish/products/payara-platform-community/`](https://www.payara.fish/products/payara-platform-community/) |
| Wildfly | Red Hat | LGPL v2.1 | [`www.wildfly.org/`](https://www.wildfly.org/) |

表 1.1 – 流行的 Jakarta EE 实现

注意

对于完整的 Jakarta EE 兼容实现列表，请参阅 [`jakarta.ee/compatibility/`](https://jakarta.ee/compatibility/)。

在本书的大部分示例中，我们将使用 **GlassFish** 作为我们的 Jakarta EE 运行时。这是因为它是一个高质量、最新的开源实现，不受任何特定供应商的约束；所有示例都应可部署到任何符合 Jakarta EE 的实现。

# Jakarta EE、Java EE、J2EE 和 Spring 框架

2017 年，Oracle 将 **Java EE** 捐赠给了 Eclipse Foundation，作为该过程的一部分，Java EE 被更名为 Jakarta EE。向 Eclipse Foundation 的捐赠意味着 Jakarta EE 规范真正实现了供应商中立，没有任何单一供应商对规范拥有控制权。

Java EE 是由 Sun Microsystems 在 2006 年引入的。Java EE 的第一个版本是 Java EE 5。Java EE 取代了 **J2EE**；J2EE 的最后一个版本是 J2EE 1.4，于 2003 年发布。尽管 J2EE 可以被认为是过时的技术，因为它几年前已被 Java EE 取代，并更名为 Jakarta EE，但术语 *J2EE* 仍然没有消失。时至今日，许多人仍然将 Jakarta EE 称为 J2EE，许多公司在他们的网站和招聘公告上宣称正在寻找“J2EE 开发者”，似乎没有意识到他们所指的技术已经过时了好几年。当前该技术的正确术语是 Jakarta EE。

此外，术语 J2EE 已经成为任何服务器端 Java 技术的“万能”术语；通常 Spring 应用程序被称为 J2EE 应用程序。**Spring**并不是，也从未是 J2EE。事实上，Spring 是由 Rod Johnson 在 2002 年作为 J2EE 的替代品创建的。就像 Jakarta EE 一样，Spring 应用程序经常被错误地称为 J2EE 应用程序。

# 摘要

在本章中，我们介绍了 Jakarta EE，概述了包含在 Jakarta EE 中的几个技术和 API 列表：

+   我们介绍了 Jakarta EE 是如何通过 Eclipse 软件基金会，由软件供应商和整个 Java 社区公开开发的

+   我们解释了 Jakarta EE 有多个实现，这一事实避免了供应商锁定，并允许我们轻松地将我们的 Jakarta EE 应用程序从一个实现迁移到另一个实现

+   我们澄清了 Jakarta EE、Java EE、J2EE 和 Spring 之间的混淆，解释了尽管 J2EE 已经过时多年，但 Jakarta EE 和 Spring 应用程序经常被错误地称为 J2EE 应用程序。

现在我们已经对 Jakarta EE 有了总体了解，我们准备开始学习如何使用 Jakarta EE 来开发我们的应用程序。
