# 前言

Cucumber JVM 是增长最快的工具之一，它提供了一个前沿的平台来概念化和实现行为驱动开发（BDD）。Cucumber 中提供的各种功能增强了业务和开发团队实施 BDD 的体验。

这本烹饪书大约有 40 个食谱。它带你进行一次学习之旅，从基本概念如特性文件、步骤定义开始，然后过渡到高级概念如钩子、标签、配置以及与 Jenkins 和测试自动化框架的集成。每一章都有多个食谱，第一个食谱介绍该章节的主要概念；随着你在章节中的进展，每个食谱的复杂度逐渐增加。本书为产品负责人、业务分析师、测试人员、开发人员以及所有希望实施 BDD 的人提供了足够的话题。

这本书的写作假设读者已经对 Cucumber 有所了解。如果你是 Cucumber 的新手，建议你先阅读我的博客：

+   **博客 1**：*如何将 Eclipse 与 Cucumber 插件集成* [`shankargarg.wordpress.com/2015/04/26/how-to-integrate-eclipse-with-cucumber-plugin/`](https://shankargarg.wordpress.com/2015/04/26/how-to-integrate-eclipse-with-cucumber-plugin/)

+   **博客 2**：*通过整合 Maven-Cucumber-Selenium-Eclipse 创建 Cucumber 项目* [`shankargarg.wordpress.com/2015/04/29/create-a-cucumber-project-by-integrating-maven-cucumber-selenium-eclipse/`](https://shankargarg.wordpress.com/2015/04/29/create-a-cucumber-project-by-integrating-maven-cucumber-selenium-eclipse/)

这两个博客将帮助你集成 Cucumber 和 Eclipse，并帮助你创建和运行一个基本项目。

本书中的所有代码都已提交到 GitHub。以下是代码仓库的 URL：[`github.com/ShankarGarg/CucumberBook.git`](https://github.com/ShankarGarg/CucumberBook.git)。

本仓库包含五个项目：

+   **Cucumber-book-blog**：该项目用于前面提到的博客中，以 Cucumber、Maven 和 Eclipse 作为起点

+   **CucumberCookbook**：该项目用于第一章到第五章

+   **CucumberWebAutomation, CucumberMobileAutomation, 和 CucumberRESTAutomation**：该项目用于第六章，*构建 Cucumber 框架*

# 本书涵盖内容

第一章，*编写特性文件*，涵盖了 Cucumber 的独特方面——Gherkin 语言及其用于编写有意义的智能特性文件的使用。本章还将涵盖不同的关键词，如文件场景、步骤、场景概述和数据表。

第二章, *创建步骤定义*，涵盖了 Glue Code/步骤定义的基本概念和用法，以及正则表达式来制定高效和优化的步骤定义。本章还将详细阐述字符串和数据表转换的概念。

第三章, *启用固定装置*，涵盖了通过标签和钩子实现固定装置的高级概念。在这里，不仅解释了标签和钩子的个别概念，还解释了使用标签和钩子组合的实践示例。

第四章, *配置 Cucumber*，涉及 Cucumber 与 JUnit 的集成以及 Cucumber 选项的概念。在这里，你将学习使用 Cucumber 选项的各种实际示例以及 Cucumber 可以生成的不同类型的报告。

第五章, *运行 Cucumber*，涵盖了从终端和从 Jenkins 运行 Cucumber 的主题。你将学习 Cucumber 与 Jenkins 和 GitHub 的集成以实现**持续集成和持续部署**（**CICD**）管道。然后你将学习并行执行以充分利用 Cucumber。

第六章, *构建 Cucumber 框架*，涵盖了创建健壮的测试自动化框架的详细步骤，以自动化 Web 应用程序、移动应用程序和 REST 服务。

# 你需要这本书什么

在开始使用 Cucumber 之前，让我们确保我们已经安装了所有必要的软件。

Cucumber 的先决条件如下：

+   Java（版本 7 或更高）作为编程语言

+   Eclipse 作为 IDE

+   Maven 作为构建工具

+   Firefox 作为浏览器

+   Eclipse-Maven 插件

+   Eclipse-Cucumber 插件

+   Jenkins

+   GIT

+   Appium

+   Android SDK

# 这本书面向谁

本书旨在为希望使用 Cucumber 进行行为驱动开发和测试自动化的商业和开发人员编写。对 Cucumber 有一定了解的读者将发现本书最有益。

由于本书的主要目标是创建测试自动化框架，因此之前的自动化经验将有所帮助。

# 部分

在这本书中，你会发现一些经常出现的标题（准备就绪、如何操作、工作原理、更多内容以及相关内容）。

为了清楚地说明如何完成一个菜谱，我们使用以下部分如下：

## 准备就绪

这一部分告诉你菜谱中可以期待什么，并描述了如何设置任何软件或任何为菜谱所需的初步设置。

## 如何操作…

这一部分包含遵循菜谱所需的步骤。

## 工作原理…

这一部分通常包括对上一部分发生情况的详细解释。

## 更多内容…

本节包含有关食谱的附加信息，以便使读者对食谱有更深入的了解。

## 参见

本节提供了有关食谱的其他有用信息的链接。

# 惯例

在这本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称将如下所示：“我们可以通过使用`include`指令来包含其他上下文。”

代码块设置如下：

```java
@When("^user enters \"(.*?)\" in username field$")
  public void user_enters_in_username_field(String userName) {
    //print the value of data passed from Feature file
    System.out.println(userName);
}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
  Scenario: checking pre-condition, action and results
    Given user is on Application landing page
    When user clicks Sign in button
    Then user is on login screen
```

任何命令行输入或输出都应如下编写：

```java
mvn test -Dcucumber.options="--tags @sanity"

```

**新术语**和**重要词汇**将以粗体显示。屏幕上显示的单词，例如在菜单或对话框中，将以如下方式显示：“点击构建的时间戳。然后点击**控制台输出**。”

### 注意

警告或重要注意事项将以如下框中显示。

### 小贴士

技巧和窍门将以如下方式显示。

# 读者反馈

我们欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中获得最大价值的标题。

要向我们发送一般反馈，请简单地发送电子邮件至 `<feedback@packtpub.com>`，并在邮件主题中提及书籍标题。

如果您在某个主题领域有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大价值。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户下载示例代码文件，适用于您购买的所有 Packt Publishing 书籍。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 错误更正

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**勘误**部分下。

## 盗版

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 `<copyright@packtpub.com>` 与我们联系，并提供疑似盗版材料的链接。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面的帮助。

## 问题

如果您对本书的任何方面有问题，您可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决问题。
