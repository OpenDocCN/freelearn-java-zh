# 前言

Java 9 及其新特性丰富了语言——这是构建健壮软件应用程序最常用的语言之一。Java 9 特别强调模块化。Java 9 的一些新特性具有开创性，如果您是经验丰富的程序员，您可以通过实施这些新特性来使您的企业应用程序更加精简。您将获得关于 Java 9 的实际指导，以及有关 Java 平台未来发展的更多信息。您还将通过项目进行工作，从中您可以提取可用的示例，以解决您自己的独特挑战。

# 这条学习路径面向的人群

这条学习路径是为那些希望提升一个层次并学习如何在 Java 最新版本中构建健壮应用程序的 Java 开发者准备的。

# 本学习路径涵盖的内容

*第一部分，精通 Java 9*，概述并解释了 Java 9 中引入的新特性以及新 API 和增强的重要性。本模块将提高您的生产力，使您的应用程序运行更快。通过学习 Java 的最佳实践，您将成为您组织中 Java 9 的首选人员。

*第二部分，Java 9 编程蓝图*，带您了解书中 10 个综合项目，展示 Java 9 的各种特性。这些项目涵盖了各种库和框架，并介绍了一些补充和扩展 Java SDK 的框架。

# 要充分利用这条学习路径

1.  一些基本的 Java 知识将有所帮助。

1.  对更高级主题的熟悉，如网络编程和线程，将有所帮助，但不是必需的。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载此学习路径的示例代码文件。如果您在其他地方购买了此学习路径，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载与勘误表”。

1.  在搜索框中输入学习路径的名称，并遵循屏幕上的说明。

下载文件后，请确保您使用最新版本解压缩或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Java-9-Building-Robust-Modular-Applications/`](https://github.com/PacktPublishing/Java-9-Building-Robust-Modular-Applications/)。我们还有其他来自我们丰富图书和视频目录的代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。查看它们吧！

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账号。以下是一个示例：“将下载的 `WebStorm-10*.dmg` 磁盘映像文件作为系统中的另一个磁盘挂载。”

代码块应如下设置：

```java
 module com.three19.irisScan 
    {
      // modules that com.three19.irisScan depends upon
      requires com.three19.irisCore;
      requires com.three19.irisData;
    }
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将被设置为粗体：

```java
[default]
exten => s,1,Dial(Zap/1|30)
exten => s,2,Voicemail(u100)
exten => s,102,Voicemail(b100)
exten => i,1,Voicemail(s0)
```

任何命令行输入或输出都应如下所示：

```java
$ mkdir css
$ cd css
```

**粗体**：表示新术语、重要词汇或您在屏幕上看到的词汇。例如，菜单或对话框中的词汇在文本中显示如下。以下是一个示例：“从管理面板中选择 System info。”

警告或重要注意事项看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：请将邮件发送至 `feedback@packtpub.com`，并在邮件主题中提及学习路径的标题。如果您对学习路径的任何方面有疑问，请通过 `questions@packtpub.com` 发送邮件给我们。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这个学习路径中发现了错误，我们将不胜感激，如果您能向我们报告这个错误。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的学习路径，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何非法副本，我们将不胜感激，如果您能提供位置地址或网站名称。请通过 `copyright@packtpub.com` 联系我们，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用了这个学习路径，为何不在您购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/)。
