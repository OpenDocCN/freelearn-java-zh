# 前言

当我们在 Java 或 Groovy 项目中编写代码时，我们通常依赖于其他项目或库。例如，我们可以在我们的项目中使用 Spring 框架，因此我们依赖于 Spring 框架中找到的类。我们希望能够从我们的构建自动化工具 Gradle 中管理此类依赖关系。

我们将看到如何定义和自定义我们需要的依赖关系。我们不仅学习如何定义依赖关系，还学习如何与存储依赖关系的仓库一起工作。接下来，我们将看到如何自定义 Gradle 解决依赖关系的方式。

除了依赖于其他库之外，我们的项目也可以成为其他项目的依赖。这意味着我们需要知道如何部署我们的项目工件，以便其他开发者可以使用它。我们学习如何定义工件以及如何将它们部署到，例如，Maven 或 Ivy 仓库。

# 本书涵盖的内容

第一章，*定义依赖关系*，介绍了依赖配置作为组织依赖关系的一种方式。您将了解 Gradle 中不同类型的依赖关系。

第二章，*与仓库一起工作*，涵盖了我们可以如何定义存储我们的依赖关系的仓库。我们将看到如何设置位置以及仓库的布局。

第三章，*解决依赖关系*，讲述了 Gradle 如何解决我们的依赖。您将学习如何自定义依赖关系解析以及解决依赖之间的冲突。

第四章，*发布工件*，涵盖了如何为我们的项目定义工件以便将其作为依赖关系发布给他人。我们将看到如何使用配置来定义工件。我们还使用本地目录作为仓库来发布工件。

第五章，*发布到 Maven 仓库*，探讨了如何将我们的工件发布到 Maven 仓库。您将学习如何为类似 Maven 的仓库（如 Artifactory 或 Nexus）定义发布，以及如何使用 Gradle 的新和孵化发布功能。

第六章，*发布到 Bintray*，涵盖了如何将我们的工件部署到 Bintray。Bintray 将自己称为“服务即分发”并提供了将我们的工件发布到世界的一种低级方式。在本章中，我们将探讨如何使用 Bintray Gradle 插件发布我们的工件。

第七章，*发布到 Ivy 仓库*，是关于将我们的工件发布到 Ivy 仓库。我们将探讨将我们的工件发布到 Ivy 仓库的不同选项，这实际上与发布到 Maven 仓库非常相似。

# 为本书所需

为了使用 Gradle 以及本书中的代码示例，我们需要至少 Java 开发工具包（版本 1.6 或更高），Gradle（示例是用 Gradle 2.3 编写的），以及一个好的文本编辑器。

# 本书面向对象

如果你正在从事 Java 或 Groovy 项目，并使用或打算使用 Gradle 来构建你的代码，这本书就是为你准备的。如果你的代码依赖于其他项目或库，你将学习如何定义和自定义这些依赖项。你的代码也可以被其他项目使用，因此你想将你的项目作为依赖项发布给那些想要阅读这本书的人。

# 规范

在本书中，你会发现许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账号如下所示：“我们可以通过使用 `include` 指令包含其他上下文。”

代码块设置如下：

```java
// Define new configurations for build.
configurations {

    // Define configuration vehicles.
    vehicles {
        description = 'Contains vehicle dependencies'
    }

    traffic {
        extendsFrom vehicles
        description = 'Contains traffic dependencies'
    }

}
```

任何命令行输入或输出都如下所示：

```java
$ gradle bintrayUpload
:generatePomFileForSamplePublication
:compileJava
:processResources UP-TO-DATE
:classes
:jar
:publishSamplePublicationToMavenLocal
:bintrayUpload

BUILD SUCCESSFUL

Total time: 9.125 secs

```

**新术语**和**重要词汇**以粗体显示。屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下：“从该屏幕，我们点击**新包**按钮。”

# 读者反馈

我们欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢或不喜欢什么。读者反馈对我们来说很重要，因为它帮助我们开发出你真正能从中获得最大收益的标题。

要向我们发送一般反馈，只需发送电子邮件至 `<feedback@packtpub.com>`，并在邮件主题中提及本书的标题。

如果你在某个主题上有专业知识，并且你对撰写或为本书做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助你从购买中获得最大收益。

## 下载示例代码

你可以从你的账户中下载示例代码文件[`www.packtpub.com`](http://www.packtpub.com)，以获取你购买的所有 Packt 出版物的示例代码。如果你在其他地方购买了这本书，你可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给你。

## 下载本书的颜色图像

我们还为您提供了一个包含本书中使用的截图/图表彩色图像的 PDF 文件。彩色图像将帮助您更好地理解输出的变化。您可以从 [`www.packtpub.com/sites/default/files/downloads/B03462_Coloredimages.pdf`](https://www.packtpub.com/sites/default/files/downloads/B03462_Coloredimages.pdf) 下载此文件。

## 勘误

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**勘误**部分下。

## 盗版

互联网上版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何形式的非法复制我们的作品，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 `<copyright@packtpub.com>` 联系我们，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面所提供的帮助。

## 问题

如果您对本书的任何方面有问题，您可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决问题。
