# 前言

Java 企业版 7，Java EE 的最新版本，在规范中添加了几个新特性。在这次规范的版本中，几个现有的 Java EE API 经历了重大改进；此外，还向 Java EE 添加了一些全新的 API。本书涵盖了最流行的 Java EE 规范的最新版本，包括 JavaServer Faces (JSF)、Java 持久化 API (JPA)、企业 JavaBeans (EJB)、上下文和依赖注入（CDI）、新的 Java JSON 处理 API (JSON-P)、WebSocket、完全重写的 Java 消息服务 API 2.0、Java XML Web 服务 API (JAX-WS)和 Java RESTful Web 服务 API (JAX-RS)，以及 Java EE 应用程序的安全性。

GlassFish 应用服务器是 Java EE 的参考实现；它是市场上第一个支持 Java EE 7 的应用服务器。本书涵盖了最新版本的强大开源应用服务器 GlassFish 4.0。

# 本书涵盖内容

第一章，*开始使用 GlassFish*，解释了如何安装和配置 GlassFish。本书还解释了如何通过 GlassFish Web 控制台部署 Java EE 应用程序。最后，讨论了基本的 GlassFish 管理任务，例如通过添加连接池和数据源来设置域和数据库连接。

第二章，*JavaServer Faces*，涵盖了使用 JSF 开发 Web 应用程序，包括 HTML5 友好的标记和 Faces Flows 等新特性。它还涵盖了如何使用 JSF 的标准验证器以及通过创建自定义验证器或编写验证器方法来验证用户输入。

第三章，*使用 JPA 进行对象关系映射*，讨论了如何通过 Java 持久化 API 与关系数据库管理系统（RDBMS）如 Oracle 或 MySQL 交互开发代码。

第四章，*企业 JavaBeans*，解释了如何使用会话和消息驱动豆来开发应用程序。本章涵盖了主要 EJB 特性，如事务管理、EJB 定时服务和安全性。还涵盖了不同类型的企业 JavaBeans 的生命周期，包括解释如何在生命周期中的特定点自动由 EJB 容器调用 EJB 方法。

第五章，*上下文和依赖注入*，介绍了上下文和依赖注入（CDI）。本章涵盖了 CDI 命名豆、使用 CDI 进行依赖注入以及 CDI 限定符。

第六章, *使用 JSON-P 处理 JSON*，介绍了如何使用新的 JSON-P API 生成和解析 JavaScript 对象表示法（JSON）数据。它还涵盖了处理 JSON 的两种 API：模型 API 和流式 API。

第七章，*WebSockets*，解释了如何开发基于 Web 的应用程序，这些应用程序在浏览器和服务器之间实现全双工通信，而不是依赖于传统的 HTTP 请求/响应周期。

第八章, *Java 消息服务*，介绍了如何在 GlassFish 中使用 GlassFish Web 控制台设置 JMS 连接工厂、JMS 消息队列和 JMS 消息主题。本章还讨论了如何使用完全重写的 JMS 2.0 API 开发消息应用程序。

第九章, *保护 Java EE 应用程序*，介绍了如何通过提供的安全领域保护 Java EE 应用程序，以及如何添加自定义安全领域。

第十章, *使用 JAX-WS 的 Web 服务*，介绍了如何通过 JAX-WS API 开发 Web 服务和 Web 服务客户端。还解释了使用 ANT 或 Maven 作为构建工具生成 Web 服务客户端代码。

第十一章, *使用 JAX-RS 开发 RESTful Web 服务*，讨论了如何通过 Java API for RESTful Web services 开发 RESTful Web 服务，以及如何通过全新的标准 JAX-RS 客户端 API 开发 RESTful Web 服务客户端。它还解释了如何利用 Java API for XML Binding (JAXB)自动在 Java 和 XML 之间转换数据。

# 本书所需

需要安装以下软件才能遵循本书中的内容：

+   需要 Java 开发工具包（JDK）1.7 或更高版本

+   GlassFish 4.0

+   构建示例需要 Maven 3 或更高版本

+   一个 Java IDE，如 NetBeans、Eclipse 或 IntelliJ IDEA（可选，但推荐）。

# 本书面向的对象

本书假设读者熟悉 Java 语言。本书的目标市场是希望学习 Java EE 的现有 Java 开发人员以及希望将技能更新到最新 Java EE 规范的现有 Java EE 开发人员。

# 规范

在本书中，您将找到许多不同风格的文本，以区分不同类型的信息。以下是一些这些风格的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名如下所示："`@Named`类注解指定此 Bean 为 CDI 命名 Bean。"

代码块设置如下：

```java
if (!emailValidator.isValid(email)) {
  FacesMessage facesMessage = new FacesMessage(htmlInputText.
getLabel()
+ ": email format is not valid");
  throw new ValidatorException(facesMessage);
}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
        <ejb>
            <ejb-name>CustomerDaoBean</ejb-name>
            <ior-security-config>
 <as-context>
 <auth-method>username_password</auth-method>
 <realm>file</realm>
 <required>true</required>
 </as-context>
            </ior-security-config>
        </ejb>
```

任何命令行输入或输出都应如下所示：

```java
$ ~/GlassFish/glassfish4/bin $ ./asadmin start-domain
Waiting for domain1 to start ........

```

**新术语**和**重要词汇**将以粗体显示。您在屏幕上看到的，例如在菜单或对话框中的文字，将以如下方式显示：“点击**下一个**按钮将您带到下一屏幕。”

### 注意

警告或重要提示将以如下框显示。

### 小贴士

小技巧和技巧将以如下方式显示。

# 读者反馈

我们始终欢迎读者的反馈。让我们知道您对这本书的看法——您喜欢什么或可能不喜欢什么。读者反馈对我们开发您真正从中受益的标题非常重要。

要向我们发送一般反馈，只需发送一封电子邮件到`<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果您在某个领域有专业知识，并且对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您已成为 Packt 图书的骄傲拥有者，我们有许多事情可以帮助您从购买中获得最大收益。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 错误清单

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然会发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进此书的后续版本。如果您发现任何错误清单，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**错误清单提交表单**链接，并输入您的错误清单详情。一旦您的错误清单得到验证，您的提交将被接受，错误清单将被上传到我们的网站，或添加到该标题的错误清单部分。任何现有的错误清单都可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的标题来查看。

## 侵权

在互联网上，版权材料的侵权问题是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以追究补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供疑似侵权材料的链接。

我们感谢您在保护我们作者方面的帮助，以及我们为您提供有价值内容的能力。

## 问题

如果你在本书的任何方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决。
