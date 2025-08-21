# 前言

Spring Web 服务（Spring-WS）由 SpringSource 社区（[`www.springsource.org/`](http://www.springsource.org/)）介绍，旨在创建基于契约的 SOAP Web 服务，其中主要需要 WSDL 或 XSD 来创建 Web 服务。由于 Spring-WS 是基于 Spring 的产品，它利用了 Spring 的概念，如控制反转（IOC）和依赖注入。Spring-WS 的一些关键特性包括：

+   强大的端点映射：根据负载、SOAP 操作和 XPath 表达式，传入的 XML 请求可以转发到任何处理程序对象

+   丰富的 XML API 支持：可以使用各种 Java 的 XML API（如 DOM、JDOM、dom4j 等）读取传入的 XML 消息

+   由 Maven 构建：Spring-WS 可以轻松集成到您的 Maven 项目中

+   支持编组技术：可以使用多种 OXM 技术，如 JAXB、XMLBean、XStream 和 JiBX，用于将 XML 消息转换为对象/从对象转换为 XML

+   安全支持：安全操作，如加密/解密、签名和认证

覆盖 Spring-WS 2.x 的所有关键特性一直是本书的主要目标。

然而，在最后两章中，详细介绍了使用 REST 风格和使用 Spring 远程调用功能进行基于契约的 Web 服务开发的不同方法。

# 本书内容

第一章，*构建 SOAP Web 服务：* 本章涵盖了在 HTTP、JMS、XMPP 和电子邮件协议上设置 SOAP Web 服务。它还涵盖了使用 DOM、JDOM、XPath 和 Marshaller 等技术来实现 Web 服务端点的不同实现。

第二章，*为 SOAP Web 服务构建客户端：* 本章涵盖了使用 Spring-WS 模板类在 HTTP、JMS、XMPP 和电子邮件协议上构建 SOAP Web 服务客户端。

第三章，*测试和监控 Web 服务：* 本章解释了如何使用 Spring-WS 的最新功能测试 Web 服务，并使用诸如 soapUI 和 TCPMon 之类的工具监控 Web 服务。

第四章，*异常/SOAP 故障处理：* 本章解释了在应用程序/系统故障的情况下的异常处理。

第五章，*SOAP 消息的日志记录和跟踪：* 在本章中，我们将看到如何记录重要事件并跟踪 Web 服务。

第六章，*编组和对象-XML 映射（OXM）：* 我们将在本章中讨论编组/解组技术以及创建自定义编组器。

第七章，*使用 XWSS 库保护 SOAP Web 服务：* 本章涵盖了安全主题，如加密、解密、数字签名认证和使用基于 XWSS 的 Spring-WS 功能进行授权，并介绍了创建密钥库的方法。

第八章，*使用 WSS4J 库保护 SOAP Web 服务：* 在本章中，我们将看到安全主题，如加密、解密、数字签名认证和使用基于 WSS4J 包的 Spring-WS 功能进行授权。

第九章，*RESTful Web 服务：* 本章解释了如何使用 Spring 中的 RESTful 支持开发 REST Web 服务。

第十章，*Spring 远程调用：* 我们将讨论使用 Spring 远程调用功能进行基于契约的 Web 服务开发，以将本地业务服务公开为使用 Hessian/Burlap、JAX-WS、JMS 的 Web 服务，并使用 JAX-WS API 通过 Apache CXF 设置 Web 服务的方法。

# 您需要什么来阅读本书

Java 知识以及基本的 Maven 知识是必备的。具有 Web 服务经验可以使您更容易地在开发环境中使用配方。本书中的基本配方帮助初学者快速学习 Web 服务主题。

# 这本书适合谁

这本书适用于那些具有 Web 服务经验或初学者的 Java/J2EE 开发人员。由于本书涵盖了 Web 服务开发的各种主题，因此那些已经熟悉 Web 服务的人可以将本书作为参考。初学者可以使用本书快速获得 Web 服务开发的实际经验。

# 约定

在本书中，您会发现一些区分不同信息类型的文本样式。以下是一些这些样式的示例，以及它们的含义解释。

文本中的代码词显示如下：“MessageDispatcherServlet 是 Spring-WS 的核心组件。”

代码块设置如下：

```java
<context-param>
<param-name>contextConfigLocation</param-name>
<param-value>/WEB-INF/classes/applicationContext.xml</param-value>
</context-param>

```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目会以粗体显示：

```java
<tns:placeOrderRequest ...> 
<tns:order>
......
</tns:order>
</tns:placeOrderRequest>

```

任何命令行输入或输出都以以下形式书写：

```java
mvn clean package tomcat:run 

```

**新术语**和**重要单词**以粗体显示。例如，屏幕上看到的单词，菜单或对话框中的单词会以这种方式出现在文本中：“您可以单击**JUnit**标签，相邻于**Console**标签，以查看测试用例是否成功”。

### 注意

警告或重要说明会显示在这样的框中。

### 注意

提示和技巧会显示为这样。
