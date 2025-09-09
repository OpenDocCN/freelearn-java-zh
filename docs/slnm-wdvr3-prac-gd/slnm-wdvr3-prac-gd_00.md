# 前言

本书是关于 Selenium WebDriver 的，即一个浏览器自动化工具，软件开发人员和 QA 工程师使用它来测试他们在不同浏览器上的 Web 应用程序。本书可以用作 WebDriver 日常使用的参考。

Selenium 是一套用于自动化浏览器的工具。它主要用于测试应用程序，但其用途不仅限于测试。它还可以用于屏幕抓取和在浏览器窗口中自动化重复性任务。Selenium 支持在所有主要浏览器上自动化，包括 Firefox、Internet Explorer、Google Chrome、Safari 和 Opera。Selenium WebDriver 现在是 W3C 标准的一部分，并且得到了主要浏览器厂商的支持。

# 本书面向对象

如果你是一名质量保证/测试专业人士、测试工程师、软件开发人员或 Web 应用程序开发人员，希望为您的 Web 应用程序创建自动化测试套件，那么这本书是您的完美指南！作为先决条件，本书假设您对 Java 编程有基本的了解，尽管不需要 WebDriver 或 Selenium 的先验知识。本书结束时，您将获得 WebDriver 的全面知识，这将有助于您编写自动化测试。

# 本书涵盖内容

第一章，*介绍 WebDriver 和 WebElements*，将从 Selenium 和其特性的概述开始。然后，我们快速跳入 WebDriver，描述它是如何感知网页的。我们还将探讨 WebDriver 的 WebElement 是什么。然后，我们将讨论在网页上定位 WebElements 以及对他们执行一些基本操作。

第二章，*与浏览器驱动程序一起工作*，将讨论 WebDriver 的各种实现，如 FirefoxDriver、IEDriver 和 ChromeDriver。我们将配置浏览器选项以在无头模式下运行测试、移动仿真以及使用自定义配置文件。随着 WebDriver 成为 W3C 规范的一部分，现在所有主要浏览器厂商都在浏览器中原生支持 WebDriver。

第三章，*使用 Java 8 特性与 Selenium 结合*，将讨论 Java 8 的突出特性，如 Streams API 和 Lambda 表达式，用于处理 WebElements 列表。Stream API 和 Lambda 表达式有助于应用函数式编程风格，创建可读性和流畅性强的测试。

第四章，*探索 WebDriver 的特性*，将讨论 WebDriver 的一些高级特性，如网页截图、执行 JavaScript、处理 Cookies 以及处理窗口和框架。

第五章，*探索高级交互 API*，将深入探讨 WebDriver 可以在网页的 WebElements 上执行更高级的操作，例如将元素从一个页面的一个框架拖放到另一个框架，以及在 WebElements 上右键单击/上下文单击。我们相信你将发现这一章很有趣。 

第六章，*理解 WebDriver 事件*，将处理 WebDriver 的事件处理方面。例如，事件可以是 WebElement 上的值变化、浏览器后退导航调用、脚本执行完成等。我们将使用这些事件来运行可访问性和性能检查。

第七章，*探索 RemoteWebDriver*，将讨论如何使用 RemoteWebDriver 和 Selenium Standalone Server 从你的机器上执行远程机器上的测试。你可以使用 RemoteWebDriver 类与远程机器上的 Selenium Standalone Server 通信，以在远程机器上运行的所需浏览器上执行命令。其流行的用例之一是浏览器兼容性测试。

第八章，*设置 Selenium Grid*，将讨论 Selenium 的一个重要且有趣的功能——Selenium Grid。使用它，你可以通过 Selenium Grid 在分布式计算机网络上执行自动化测试。我们将配置一个 Hub 和多个 Nodes 进行跨浏览器测试。这也使得并行运行测试和在分布式架构中运行测试成为可能。

第九章，*页面对象模式*，将讨论一个名为页面对象模式（PageObject Pattern）的知名设计模式。这是一个经过验证的模式，将帮助你更好地掌握自动化框架和场景，以实现更好的可维护性。

第十章，*使用 Appium 在 iOS 和 Android 上进行移动测试*，将介绍如何使用 WebDriver 通过 Appium 自动化 iOS 和 Android 平台的测试脚本。

第十一章，*使用 TestNG 进行数据驱动测试*，将讨论使用 TestNG 进行数据驱动测试技术。使用数据驱动测试方法，我们可以使用多组测试数据重用测试，以获得额外的覆盖率。

# 为了充分利用这本书

预期读者对编程有一个基本了解，最好是使用 Java，因为我们将通过代码示例带读者了解 WebDriver 的几个功能。以下软件是本书所需的：

1.  Oracle JDK8

1.  Eclipse IDE

1.  Maven 3

1.  Google Chrome

1.  Mozilla Firefox

1.  Internet Explorer 或 Edge（在 Windows 上）

1.  Apple Safari

1.  Appium

# 安装 Java

在本书中，我们展示的所有代码示例，涵盖 WebDriver 的各种功能，都将使用 Java 编写。为了遵循这些示例并编写您自己的代码，您需要在您的计算机上安装 Java 开发工具包。最新版本的 JDK 可以从以下链接下载：

[`www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html`](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

# 安装 Eclipse

本书是一本实用指南，期望用户编写和执行 WebDriver 示例。为此，安装一个 Java 集成开发环境会很有帮助。Eclipse IDE 是 Java 用户社区中流行的选择。Eclipse IDE 可以从 [`www.eclipse.org/downloads/`](https://www.eclipse.org/downloads/) 下载。

# 下载示例代码文件

您可以从 [www.packtpub.com](http://www.packtpub.com) 的账户下载本书的示例代码文件。如果您在其他地方购买了此书，您可以访问 [www.packtpub.com/support](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在 [www.packtpub.com](http://www.packtpub.com/support) 登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载与勘误”。

1.  在“搜索”框中输入书籍名称，并遵循屏幕上的说明。

一旦文件下载完成，请确保您使用最新版本解压缩或提取文件夹，具体如下：

+   适用于 Windows 的 WinRAR/7-Zip

+   适用于 Mac 的 Zipeg/iZip/UnRarX

+   适用于 Linux 的 7-Zip/PeaZip

本书代码包也托管在 GitHub 上，地址为 [`github.com/PacktPublishing/Selenium-WebDriver-3-Practical-Guide-Second-Edition`](https://github.com/PacktPublishing/Selenium-WebDriver-3-Practical-Guide-Second-Edition)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富的图书和视频目录的代码包可供选择，请访问 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。查看它们！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从 [`www.packtpub.com/sites/default/files/downloads/SeleniumWebDriver3PracticalGuideSecondEdition_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/SeleniumWebDriver3PracticalGuideSecondEdition_ColorImages.pdf) 下载。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账号。以下是一个示例：“`beforeMethod()`”，它带有 `@BeforeMethod` 测试 NG 注解。

代码块设置如下：

```java
<input id="search" type="search" name="q" value="" class="input-text required-entry" maxlength="128" placeholder="Search entire store here..." autocomplete="off">
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
WebElement searchBox = driver.findElement(By.id("q"));
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“要运行测试，在代码编辑器中右键单击并选择“运行 As | TestNG 测试”，如图所示。”

警告或重要提示看起来像这样。

技巧和窍门看起来像这样。

# 联系我们

我们欢迎读者的反馈。

**总体反馈**：请发送电子邮件至`feedback@packtpub.com`，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过电子邮件联系我们的`questions@packtpub.com`。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一错误。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式发现我们作品的非法副本，我们将不胜感激，如果您能向我们提供位置地址或网站名称。请通过电子邮件联系我们的`copyright@packtpub.com`，并附上材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用了这本书，为何不在您购买书籍的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解 Packt 的更多信息，请访问[packtpub.com](https://www.packtpub.com/)。
