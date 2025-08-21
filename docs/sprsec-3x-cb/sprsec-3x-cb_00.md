# 前言

# 介绍

Spring Security 是 Spring 框架提供的安全层。Spring 框架是一个活跃的开源项目，使应用程序的进一步开发变得更加容易。它提供了各种层来处理项目设计和实施生命周期中面临的不同场景和挑战。

Spring 框架的 Spring Security 层与 Spring 框架的耦合度非常低，因此可以轻松地集成到其他应用程序中。

在本书中，我们将把 Spring Security 与其他框架集成，并通过编码示例进行演示。

# 本书涵盖的内容

第一章，*基本安全*，介绍了 J2ee 应用程序中安全性的基础知识。它向读者介绍了各种应用安全性的机制，以对用户进行身份验证和授权。它还解释了容器管理安全性。

第二章，*使用 Struts 2 的 Spring 安全性*，提供了在 Struts 2 应用程序中集成 Spring Security 的步骤。它演示了使用 Spring 框架提供的其他安全机制进行数据库身份验证和 LDAP 身份验证和授权。

第三章，*使用 JSF 的 Spring 安全性*，解释了在 JSF 应用程序中使用 Spring Security 的所有方面。它展示了如何使 JSF 应用程序使用监听器与 Spring Security 进行通信。

第四章，*使用 Grails 的 Spring 安全性*，演示了 grails 应用程序如何与 Spring Security 无缝集成。我们还展示了 Spring Security UI 如何提供屏幕来创建用户和角色。我们演示了在 GSP 页面中使用 Spring Security 标签。

第五章，*使用 GWT 的 Spring 安全性*，专注于 GWT 框架。GWT 框架与 GWT 集成，Spring Security 可用于对访问 GWT 应用程序的用户进行身份验证和授权。

第六章，*使用 Vaadin 的 Spring 安全性*，提出了将 Spring Security 与 Vaadin 框架集成的各种选项。我们创建了一个示例产品目录应用程序，以演示 Spring Security 与 Vaadin 框架的集成。

第七章，*使用 Wicket 的 Spring 安全性*，演示了将 wicket 框架与 Spring Security 集成。Wicket 本身具有内置的身份验证和授权框架，但挑战在于使 wicket 使用外部框架进行身份验证和授权。

第八章，*使用 ORM 和 NoSQL DB 的 Spring 安全性*，解释了在使用 Spring Security API 类进行身份验证和授权时，使用 Hibernate 和 MongoDB。

第九章，*使用 Spring Social 的 Spring 安全性*，介绍了 Spring Social，这是由 Spring Source 开发的一个框架，用于提供对社交网络站点的集成。Spring Social 使用 Spring Security 进行身份验证和授权。该章节演示了 Spring Social 和 Spring Security 如何通过演示 Facebook 登录应用程序进行集成。

第十章，*使用 Spring Web Services 的 Spring 安全性*，解释了保护 RESTFUL 和基于 SOAP 的 Web 服务的各种选项。

第十一章，*更多关于 Spring 安全性*，是一个杂项章节。它解释了如何将 Spring Security 与 Kaptcha API 集成，并提供多个输入身份验证。

# 您需要为本书准备什么

为了完成本书中的所有示例，您需要了解以下内容：

+   JBOSS 服务器

+   Netbeans

+   Maven

+   Java

+   Tomcat

+   Open LDAP

+   Apache DS

+   Eclipse IDE

# 这本书是为谁写的

这本书适用于所有基于 Spring 的应用程序开发人员，以及希望使用 Spring Security 将强大的安全机制实施到 Web 应用程序开发中的 Java Web 开发人员。

读者被假定具有 Java Web 应用程序开发的工作知识，对 Spring 框架有基本了解，并且对 Spring Security 框架架构的基本知识有一定了解。

对其他 Web 框架（如 Grails 等）的工作知识将是利用本书提供的全部食谱的额外优势，但这并非强制要求。

# 约定

在本书中，您将找到许多不同类型信息之间的区别的文本样式。以下是一些这些样式的示例，以及它们的含义解释。

文本中的代码单词显示如下：“我们可以通过使用`include`指令包含其他上下文。”

代码块设置如下：

```java
<%@ page contentType="text/html; charset=UTF-8" %>
<%@ page language="java" %>
<html >
  <HEAD>
    <TITLE>PACKT Login Form</TITLE>
    <SCRIPT>
      function submitForm() {
        var frm = document. myform;
        if( frm.j_username.value == "" ) {
          alert("please enter your username, its empty");
          frm.j_username.focus();
          return ;
        }
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```java
<%@ page contentType="text/html; charset=UTF-8" %>
<%@ page language="java" %>
<html >
  <HEAD>
    <TITLE>PACKT Login Form</TITLE>
    <SCRIPT>
      function submitForm() {
        var frm = document. myform;
        if( frm.j_username.value == "" ) {
          alert("please enter your username, its empty");
          frm.j_username.focus();
          return ;
        }
```

任何命令行输入或输出都以以下形式编写：

```java
[INFO] Parameter: groupId, Value: com.packt
[INFO] Parameter: artifactId, Value: spring-security-wicket
[INFO] Parameter: version, Value: 1.0-SNAPSHOT

```

**新术语**和**重要单词**以粗体显示。例如，屏幕上看到的单词，例如菜单或对话框中的单词，会在文本中显示为：“单击**提交**后，我们需要获得经过身份验证的会话。”

### 注意

警告或重要说明会以这样的框出现。

### 提示

提示和技巧会以这种形式出现。
