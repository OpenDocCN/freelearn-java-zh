# 前言

Vaadin 框架是一个开源的 Java Web 框架，在 Apache 许可证下发布。该框架文档齐全，包括复杂的 UI 组件和主题，已在现实应用程序中得到实战检验，并由一家承诺的公司和一个充满活力的社区支持，他们通过论坛回答和数百个附加组件为框架做出贡献。

Vaadin 框架允许开发者使用在服务器 JVM 上运行的 Java 代码实现 Web 用户界面。UI 在浏览器上以 HTML5 的形式渲染。该框架通过类似于 Swing 或 AWT 的编程模型，提供浏览器和服务器之间完全自动化的通信。这允许开发者/程序员将面向对象技术的优势带到 Web 应用程序的表现层。

*使用 Vaadin* 8 实现以数据为中心的应用程序是一本实用指南，它教您如何在数据管理是核心的 Web 应用程序中实现一些最典型的需求。您将了解国际化、身份验证、授权、数据库连接、CRUD 视图、报告生成和数据懒加载。

本书还将通过向您展示如何在 UX 和代码层面做出良好决策，帮助您锻炼编程和软件设计技能。您将学习如何模块化您的应用程序，以及如何在 UI 组件之上提供 API 以增加可重用性和可维护性。

# 本书面向读者

本书非常适合对 Java 编程语言有良好理解且对 Vaadin 框架有基本知识的开发者，他们希望通过框架提高自己的技能。如果您想学习概念、技术、技术和实践，以帮助您掌握使用 Vaadin 进行 Web 开发，并了解现实应用程序中常见应用程序功能的开发方式，这本书适合您。

# 本书涵盖内容

第一章，*创建新的 Vaadin 项目*，演示了如何从头开始创建新的 Vaadin Maven 项目，并解释了 Vaadin 应用程序的主要架构和组成部分。

第二章，*模块化和主屏幕*，解释了如何设计用于实现主屏幕的 API，并展示了如何创建在运行时注册的功能性应用程序模块。

第三章，*使用国际化实现服务器端组件*，讨论了实现具有国际化支持的定制 UI 组件的实施策略。

第四章，*实现身份验证和授权*，探讨了在 Vaadin 应用程序中实现安全的身份验证和授权机制的不同方法。

第五章，*使用 JDBC 连接 SQL 数据库*，重点介绍了 JDBC、连接池和仓库类，以便连接到 SQL 数据库。

第六章，*使用 ORM 框架连接 SQL 数据库*，概述了如何使用 JPA、MyBatis 和 jOOQ 从 Vaadin 应用程序连接到 SQL 数据库。

第七章，*实现 CRUD 用户界面*，带你了解用户界面设计和 CRUD（创建、读取、更新和删除）视图的实现。

第八章，*添加报告功能*，展示了如何使用 JasperReports 生成和可视化打印预览报告。

第九章，*懒加载*，探讨了如何实现懒加载，以使你的应用程序在处理大数据集时消耗更少的资源。

# 为了充分利用这本书

如果你已经对 Vaadin 框架有一些经验，那么你将能从这本书中获得最大收益。如果你没有，请在继续阅读这本书之前，先通过官方在线教程了解[`vaadin.com/docs/v8/framework/tutorial.html`](https://vaadin.com/docs/v8/framework/tutorial.html)。

为了使用配套代码，你需要 Java SE 开发工具包和 Java EE SDK 版本 8 或更高版本。你还需要 Maven 版本 3 或更高版本。建议使用具有 Maven 支持的 Java IDE，例如 IntelliJ IDEA、Eclipse 或 NetBeans。

# 下载示例代码文件

你可以从[www.packtpub.com](http://www.packtpub.com)的账户下载这本书的示例代码文件。如果你在其他地方购买了这本书，你可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给你。

你可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载完成后，请确保使用最新版本的以下软件解压缩或提取文件夹：

+   Windows 版的 WinRAR/7-Zip

+   Mac 版的 Zipeg/iZip/UnRarX

+   Linux 版的 7-Zip/PeaZip

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Data-Centric-Applications-with-Vaadin-8`](https://github.com/PacktPublishing/Data-Centric-Applications-with-Vaadin-8)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**找到。查看它们吧！

# 下载彩色图片

我们还提供包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/DataCentricApplicationswithVaadin8_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/DataCentricApplicationswithVaadin8_ColorImages.pdf)。

# 代码实战

访问以下链接查看代码运行的视频：

[`goo.gl/qFmc3L`](https://goo.gl/qFmc3L)

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账号。以下是一个示例：“应用程序应在 `http://localhost:8080` 上可用。”

代码块设置如下：

```java
LoginForm loginForm = new LoginForm()
loginForm.addLoginListener(e ->  {
    String password = e.getLoginParameter("password");
    String username = e.getLoginParameter("username");
    ...
});
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
LoginForm loginForm = new LoginForm() {
    @Override
    protected Component createContent(TextField username,
            PasswordField password, Button loginButton) {

        CheckBox rememberMe = new CheckBox();
        rememberMe.setCaption("Remember me");

        return new VerticalLayout(username, password, loginButton,
                rememberMe);
    }
};
```

任何命令行输入或输出应如下编写：

```java
cd Data-centric-Applications-with-Vaadin-8
mvn install
```

**粗体**：表示新术语、重要词汇或您在屏幕上看到的词汇。例如，菜单或对话框中的词汇在文本中显示如下。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要提示如下所示。

技巧和窍门如下所示。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：请发送电子邮件至 `feedback@packtpub.com` 并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过 `questions@packtpub.com` 发送电子邮件给我们。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在本书中发现错误，我们将非常感激您能向我们报告。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何非法副本，我们将非常感激您能提供位置地址或网站名称。请通过 `copyright@packtpub.com` 联系我们，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为本书做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用过本书，为何不在购买该书的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt 的我们能够了解您对我们产品的看法，而我们的作者可以查看他们对本书的反馈。谢谢！

如需更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/)。
