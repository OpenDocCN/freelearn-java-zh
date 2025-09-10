# 前言

jBPM 是一个领先的开源 BPM 和流程平台，其开发由红帽公司赞助，遵循 Apache 软件许可（ASL）许可。jBPM 产品已经存在近 10 年；其最突出的优点在于灵活性、可扩展性和轻量级，它是一个模块化、跨平台的纯 Java 引擎，符合 BPMN2 规范。

它提供了一个强大的管理控制台和开发工具，支持用户在业务流程生命周期（开发、部署和版本控制）中的使用。它集成了广泛采用的框架和技术（SOAP、REST、Spring、Java EE CDI 和 OSGi），并为 Git 和 Maven 提供现成的支持。

它可以适应不同的系统架构，可以作为完整的 Web 应用程序或作为服务部署；它可以紧密嵌入到传统的桌面应用程序中，也可以松散集成到复杂的事件驱动架构中。在其默认配置下，jBPM 可以由企业级应用程序服务器红帽 EAP 6.x 或前沿的红帽 WildFly 8 服务器托管。

*精通 JBPM6* 带您通过一种实用的方法来使用和扩展 jBPM 6.2。本书提供了详细的 jBPM 6.2 概述；它涵盖了引擎支持的 BPM 符号，并解释了高级引擎和 API 主题，尽可能多地关注几个实际的工作示例。

本书向用户展示了针对常见实时问题（如 BAM，即业务活动监控）和生产场景的解决方案。

# 本书涵盖的内容

第一章，*业务流程建模 – 连接业务与技术*，为用户提供了对 BPM 环境的概述，介绍了 jBPM 世界，并揭示了业务逻辑集成平台的整体图景。

第二章，*构建您的第一个 BPM 应用程序*，首先通过为读者提供实际的产品安装和配置教程，直接将用户带入 jBPM 工具栈，然后处理初学者主题，如业务流程建模和部署。

第三章，*使用流程设计器*，深入探讨了基于 Web 的 jBPM 工具，向用户展示主要的 jBPM Web 设计器功能：用户表单、脚本和流程模拟。

第四章，*运营管理*，介绍了新的 jBPM 工件架构，重点关注 Maven 仓库（模块和部署）、引擎审计和日志分析、作业调度以及一个完整的 BAM 定制示例（包含仪表板集成）。

第五章，*BPMN 结构*，说明了 jBPM 实现的 BPMN2 结构，并通过注释上下文中准备好的源代码示例提供了使用它们的见解和注意事项。

第六章，*核心架构*，通过详细阐述如何利用几个源代码示例来利用引擎功能，涵盖了所有 jBPM 模块（例如，人类任务服务、持久化、审计和配置）。

第七章，*定制和扩展 jBPM*，以实用方法探讨引擎定制区域；它向用户提供有关如何定制持久化、人类任务服务、序列化机制和工作项处理架构的解释。

第八章，*将 jBPM 与企业架构集成*，描述了 jBPM 如何通过 SOAP、REST 或 JMS 作为客户端或服务器与外部应用程序集成。它提供了有关如何在 Java EE 应用程序中利用其服务的见解。

第九章，*生产中的 jBPM*，探讨了在处理服务可用性、可扩展性和安全性时 jBPM 系统的功能；它提供了有关在生产环境中调整引擎性能的技巧和技术。

附录 A，*未来*，简要介绍了业务流程建模的趋势和未来。

附录 B，*jBPM BPMN 结构参考*，是 jBPM 支持的 BPMN 结构的快速参考。

# 您需要为这本书准备以下内容

在运行代码示例之前，您需要安装以下软件：

jBPM 需要 JDK 6 或更高版本。可以从[`www.oracle.com/technetwork/java/javase/downloads/index.html`](http://www.oracle.com/technetwork/java/javase/downloads/index.html)下载 JDK 6 或更新的版本。此页面上也有安装说明。要验证您的安装是否成功，请在命令行中运行`java –version`。

从[`sourceforge.net/projects/jbpm/files/jBPM%206/jbpm-6.2.0.Final/`](http://sourceforge.net/projects/jbpm/files/jBPM%206/jbpm-6.2.0.Final/)下载`jbpm-6.2.0.Final-installer-full.zip`。只需将其解压缩到您选择的文件夹中。用户指南([`docs.jboss.org/jbpm/v6.2/userguide/jBPMInstaller.html`](http://docs.jboss.org/jbpm/v6.2/userguide/jBPMInstaller.html))包括如何简单快速开始使用说明。

jBPM 设置需要 Ant 1.7 或更高版本([`ant.apache.org/srcdownload.cgi`](http://ant.apache.org/srcdownload.cgi))。

额外所需的软件如下：

+   Git 1.9 或更高版本([`git-scm.com/downloads`](http://git-scm.com/downloads))

+   Maven 3.2.3 或更高版本([`maven.apache.org/download.cgi`](http://maven.apache.org/download.cgi))

运行示例的首选开发 IDE 是 Eclipse Kepler 发行版，它可以在 BPMN 安装过程中自动下载并预先配置。

# 本书面向对象

本书主要面向 jBPM 开发者、业务分析师和流程建模师，以及在某种程度上必须接触 jBPM 平台功能的项目经理。本书假设你具备业务分析和建模的先验知识，当然，还需要 Java 知识；对 jBPM 的基本知识也是必需的。

# 习惯用法

在这本书中，你会发现许多不同风格的文本，用于区分不同类型的信息。以下是一些这些风格的示例及其含义的解释。

文本中的代码词汇如下所示：“在`roles.properties`文件中为用户指定一个角色。”

代码块如下设置：

```java
ReleaseId newReleaseId = ks.newReleaseId("com.packt.masterjbpm6", "pizzadelivery", "1.1-SNAPSHOT");
// then create the container to load the existing module
Results result = ks.updateToVersion (newReleaseId);
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目将以粗体显示：

```java
<bpmn2:scriptTask id="_2" name="prepare order" scriptFormat="http://www.java.com/java">
```

任何命令行输入或输出都如下所示：

```java
ant install.demo

```

**新术语**和**重要词汇**将以粗体显示。你在屏幕上看到的，例如在菜单或对话框中的文字，将以如下方式显示：“从左侧导航菜单面板中选择**管理** | **数据提供者**链接。”

### 注意

警告或重要提示将以如下框显示。

### 小贴士

小贴士和技巧如下所示。

# 读者反馈

我们的读者反馈总是受欢迎的。告诉我们你对这本书的看法——你喜欢什么或可能不喜欢什么。读者反馈对我们开发你真正能从中获得最大价值的标题非常重要。

要向我们发送一般反馈，只需发送一封电子邮件到`<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果你在一个领域有专业知识，并且你对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是 Packt 图书的骄傲拥有者，我们有许多事情可以帮助你从购买中获得最大价值。

## 下载示例代码

你可以从[`www.packtpub.com`](http://www.packtpub.com)下载示例代码文件，这些文件存储在你购买的所有 Packt 出版物的账户中。如果你在其他地方购买了这本书，你可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给你。

## 错误清单

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然会发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误表，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将在**勘误**部分显示。

## 盗版

在互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

如果您发现疑似盗版材料，请通过 `<copyright@packtpub.com>` 联系我们，并提供链接。

我们感谢您在保护我们作者以及为我们提供有价值内容方面的帮助。

## 问题

如果您在本书的任何方面遇到问题，请通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决。
