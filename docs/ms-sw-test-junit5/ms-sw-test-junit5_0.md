# 前言

人类并非完美的思考者。在撰写本文时，软件工程师是人类。大多数是。因此，编写高质量、有用的软件是一项非常困难的任务。正如我们将在本书中发现的那样，软件测试是软件工程师（即开发人员、程序员或测试人员）进行的最重要的活动之一，以保证软件的质量和信心水平。

JUnit 是 Java 语言中最常用的测试框架，也是软件工程中最显著的框架之一。如今，JUnit 不仅仅是 Java 的单元测试框架。正如我们将发现的那样，它可以用于实现不同类型的测试（如单元测试、集成测试、端到端测试或验收测试），并使用不同的策略（如黑盒或白盒）。

2017 年 9 月 10 日，JUnit 团队发布了 JUnit 5.0.0。本书主要关注这个 JUnit 的新主要版本。正如我们将发现的那样，JUnit 5 对 JUnit 框架进行了完全的重新设计，改进了重要功能，如模块化（JUnit 5 架构完全模块化）、可组合性（JUnit 5 的扩展模型允许以简单的方式集成第三方框架到 JUnit 5 测试生命周期中）、兼容性（JUnit 5 支持在全新的 JUnit 平台中执行 JUnit 3 和 4 的遗留测试）。所有这些都遵循基于 Java 8 的现代编程模型，并符合 Java 9 的规范。

软件工程涉及一个多学科的知识体系，对变革有着强烈的推动力。本书全面审查了与软件测试相关的许多不同方面，主要是从开源的角度（JUnit 从一开始就是开源的）。在本书中，除了学习 JUnit 外，还可以学习如何在开发过程中使用第三方框架和技术，比如 Spring、Mockito、Selenium、Appium、Cucumber、Docker、Android、REST 服务、Hamcrest、Allure、Jenkins、Travis CI、Codecov 或 SonarCube 等。

# 本书涵盖的内容

第一章*，软件质量和 Java 测试的回顾*，对软件质量和测试进行了详细回顾。本章的目标是以易懂的方式澄清这一领域的术语。此外，本章还总结了 JUnit（版本 3 和 4）的历史，以及一些 JUnit 增强器（例如，可以用来扩展 JUnit 的库）。

第二章*，JUnit 5 的新功能*，首先介绍了创建 JUnit 5 版本的动机。然后，本章描述了 JUnit 5 架构的主要组件，即 Platform、Jupiter 和 Vintage。接下来，我们将了解如何运行 JUnit 测试，例如使用不同的构建工具，如 Maven 或 Gradle。最后，本章介绍了 JUnit 5 的扩展模型，允许任何第三方扩展 JUnit 5 的核心功能。

第三章*，JUnit 5 标准测试*，详细描述了新的 JUnit 5 编程模型的基本特性。这个编程模型，连同扩展模型，被称为 Jupiter。在本章中，您将了解基本的测试生命周期、断言、标记和过滤测试、条件测试执行、嵌套和重复测试，以及如何从 JUnit 4 迁移。

第四章*，使用高级 JUnit 功能简化测试*，详细描述了 JUnit 5 的功能，如依赖注入、动态测试、测试接口、测试模板、参数化测试、与 Java 9 的兼容性，以及 JUnit 5.1 的计划功能（在撰写本文时尚未发布）。

第五章*，JUnit 5 与外部框架的集成*，讨论了 JUnit 5 与现有第三方软件的集成。可以通过不同的方式进行此集成。通常，应使用 Jupiter 扩展模型与外部框架进行交互。这适用于 Mockito（一种流行的模拟框架）、Spring（一个旨在基于依赖注入创建企业应用程序的 Java 框架）、Docker（一个容器平台技术）或 Selenium（用于 Web 应用程序的测试框架）。此外，开发人员可以重用 Jupiter 测试生命周期与其他技术进行交互，例如 Android 或 REST 服务。

第六章*，从需求到测试用例*，提供了一套旨在帮助软件测试人员编写有意义的测试用例的最佳实践。考虑需求作为软件测试的基础，本章提供了一个全面的指南，以编写测试，避免典型的错误（反模式和代码异味）。

*第七章，测试管理*，是本书的最后一章，其目标是指导读者了解软件测试活动在一个活跃的软件项目中是如何管理的。为此，本章回顾了诸如**持续集成**（**CI**）、构建服务器（Jenkins、Travis）、测试报告或缺陷跟踪系统等概念。为了结束本书，还提供了一个完整的示例应用程序，以及不同类型的测试（单元测试、集成测试和端到端测试）。

# 您需要为本书做些什么

为了更好地理解本书中提出的概念，强烈建议 fork GitHub 存储库，其中包含本书中提出的代码示例（[`github.com/bonigarcia/mastering-junit5`](https://github.com/bonigarcia/mastering-junit5)）。在作者看来，触摸和玩弄代码对于快速掌握 JUnit 5 测试框架至关重要。正如前面介绍的，本书的最后一章提供了一个完整的应用程序示例，涵盖了本书中一些最重要的主题。这个应用程序（称为*Rate my cat!*）也可以在 GitHub 上找到，位于存储库[`github.com/bonigarcia/rate-my-cat`](https://github.com/bonigarcia/rate-my-cat)中。

为了运行这些示例，您需要 JDK 8 或更高版本。您可以从 Oracle JDK 的网站下载：[`www.oracle.com/technetwork/java/javase/downloads/index.html`](http://www.oracle.com/technetwork/java/javase/downloads/index.html)。此外，强烈建议使用**集成开发环境**（**IDE**）来简化开发和测试过程。正如我们将在本书中发现的那样，在撰写本文时，有两个完全符合 JUnit 5 的 IDE，即：

+   Eclipse 4.7+（Oxygen）：[`eclipse.org/ide/`](https://eclipse.org/ide/)。

+   IntelliJ IDEA 2016.2+：[`www.jetbrains.com/idea/`](https://www.jetbrains.com/idea/)。

如果您更喜欢从命令行运行 JUnit 5，则可以使用两种可能的构建工具：

+   Maven：[`maven.apache.org/`](https://maven.apache.org/)

+   Gradle：[`gradle.org/`](https://gradle.org/)

# 这本书适合谁

本书面向 Java 软件工程师。因此，这部文学作品试图与读者（即 Java）说同样的语言，因此它是由上述公开的 GitHub 存储库上可用的工作代码示例驱动的。

# 约定

在本书中，您会发现一些文本样式，用于区分不同类型的信息。以下是一些样式的示例及其含义解释。文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下：“`@AfterAll`和`@BeforeAll`方法仅执行一次”。

一块代码设置如下：

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.Assertions.*assertTrue*;

import org.junit.jupiter.api.Test;

class StandardTest {

    @Test
    void verySimpleTest () {
        *assertTrue*(true);
    }

}
```

任何命令行输入或输出都以以下形式编写：

```java
mvn test
```

**新术语**和**重要词汇**显示为粗体，如：“**兼容性**是产品、系统或组件与其他产品交换信息的程度”。

警告或重要提示会以这样的方式出现在框中。

提示和技巧会出现在这样的情况下。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对本书的看法-您喜欢或不喜欢的内容。读者的反馈对我们很重要，因为它有助于我们开发您真正能从中获益的标题。

要向我们发送一般反馈，只需发送电子邮件至`feedback@packtpub.com`，并在消息主题中提及书名。

如果您在某个专业领域有专业知识，并且有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南，网址为[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的自豪所有者，我们有一些东西可以帮助您充分利用您的购买。

# 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便文件直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册到我们的网站。

1.  将鼠标指针悬停在顶部的“支持”选项卡上。

1.  单击“代码下载和勘误”。

1.  在搜索框中输入书名。

1.  选择您要下载代码文件的书籍。

1.  从下拉菜单中选择您购买本书的地点。

1.  单击“代码下载”。

下载文件后，请确保使用以下最新版本的软件解压缩文件夹：

+   WinRAR / 7-Zip for Windows

+   Zipeg / iZip / UnRarX for Mac

+   7-Zip / PeaZip for Linux

该书的代码包也托管在 GitHub 上，网址为[`github.com/bonigarcia/mastering-junit5`](https://github.com/bonigarcia/mastering-junit5)。我们还有其他丰富书籍和视频代码包可供下载，网址为[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。快去看看吧！

# 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误确实会发生。如果您在我们的书籍中发现错误-可能是文本或代码中的错误-我们将不胜感激，如果您能向我们报告。通过这样做，您可以帮助其他读者避免挫折，并帮助我们改进本书的后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)报告，选择您的书籍，单击“勘误提交表格”链接，并输入您的勘误详情。一旦您的勘误经过验证，您的提交将被接受，并且勘误将被上传到我们的网站或添加到该标题的“勘误”部分下的任何现有勘误列表中。

要查看先前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需信息将出现在“勘误”部分下。

# 盗版

互联网上侵犯版权材料的盗版是所有媒体都面临的持续问题。在 Packt，我们非常重视保护我们的版权和许可。如果您在互联网上发现我们作品的任何非法副本，请立即向我们提供位置地址或网站名称，以便我们采取补救措施。

请通过`copyright@packtpub.com`与我们联系，并附上涉嫌盗版材料的链接。

我们感谢您在保护我们的作者和我们为您提供有价值内容的能力方面的帮助。

# 问题

如果您对本书的任何方面有问题，可以通过`questions@packtpub.com`与我们联系，我们将尽力解决问题。
