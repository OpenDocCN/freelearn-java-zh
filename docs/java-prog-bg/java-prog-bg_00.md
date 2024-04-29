# 前言

无论您是第一次接触高级面向对象编程语言，比如 Java，还是已经有一段时间的编程经验，只是想要将 Java 添加到您的技能范围，或者您从未接触过一行代码，本书都旨在满足您的需求。我们将快速前进，不会回避繁重的主题，但我们将从最基础的知识开始，边学习面向对象编程的概念。如果这本书能帮助您理解 Java 编程的重要性，以及如何在 NetBeans 中开始开发 Java 应用程序，我将认为它是成功的。如果 Java 成为您最喜爱的编程语言，我同样会感到高兴！

# 您需要为本书做些什么

对于本书，您需要**Java 开发工具包**（**JDK**）和 NetBeans

# 本书适合谁

本书适用于任何想要开始学习 Java 语言的人，无论您是学生、业余学习者，还是现有程序员想要增加新的语言技能。不需要 Java 或编程的任何先前经验。

# 约定

在本书中，您将找到一些区分不同信息类型的文本样式。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名显示如下："`Source Packages`文件夹是我们将编写代码的地方。"

代码块设置如下：

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目以粗体显示：

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

任何命令行输入或输出都以以下方式编写：

```java
java -jar WritingToFiles.jar
```

新术语和重要单词以粗体显示。您在屏幕上看到的单词，例如菜单或对话框中的单词，会以这种方式出现在文本中：“选择 Java SE 列下方的下载按钮。”

警告或重要提示将显示在这样的框中。

提示和技巧会以这种方式出现。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对本书的看法，您喜欢或不喜欢的地方。读者的反馈对我们开发您真正受益的书籍至关重要。

要向我们发送一般反馈，只需发送电子邮件至`feedback@packtpub.com`，并在主题中提及书名。

如果您在某个专题上有专业知识，并且有兴趣编写或为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的自豪所有者，我们有一些事情可以帮助您充分利用您的购买。

# 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载您购买的所有 Packt Publishing 书籍的示例代码文件。如果您在其他地方购买了本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，文件将直接通过电子邮件发送给您。您可以按照以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册到我们的网站。

1.  将鼠标指针悬停在顶部的“支持”选项卡上。

1.  单击“代码下载和勘误”。

1.  在搜索框中输入书名。

1.  选择您要下载代码文件的书籍。

1.  从下拉菜单中选择您购买本书的地方。

1.  单击“代码下载”。

文件下载完成后，请确保您使用最新版本的解压缩软件解压文件夹：

+   Windows 系统使用 WinRAR / 7-Zip

+   Mac 系统使用 Zipeg / iZip / UnRarX

+   Linux 系统使用 7-Zip / PeaZip

本书的代码包也托管在 GitHub 上，链接为[`github.com/PacktPublishing/Java-Programming-for-Beginners`](https://github.com/PacktPublishing/Java-Programming-for-Beginners)。我们还有其他丰富图书和视频代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。快去看看吧！

# 下载本书的彩色图片

我们还为您提供了一个 PDF 文件，其中包含本书中使用的截图/图表的彩色图片。这些彩色图片将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/JavaProgrammingforBeginners_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/JavaProgrammingforBeginners_ColorImages.pdf)下载此文件。

# 勘误

尽管我们已经尽最大努力确保内容的准确性，但错误还是会发生。如果您在我们的书中发现错误——可能是文字或代码上的错误，我们将不胜感激，如果您能向我们报告。通过这样做，您可以帮助其他读者避免挫折，并帮助我们改进本书的后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)报告，选择您的书，点击“勘误提交表”链接，并输入您的勘误详情。一旦您的勘误经过验证，您的提交将被接受，并且勘误将被上传到我们的网站或添加到该书标题的勘误部分的任何现有勘误列表中。

要查看先前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需信息将出现在“勘误”部分下。

# 盗版

互联网上的版权盗版是所有媒体的持续问题。在 Packt，我们非常重视版权和许可的保护。如果您在互联网上发现我们作品的任何非法副本，请立即向我们提供位置地址或网站名称，以便我们采取补救措施。

请通过`copyright@packtpub.com`与我们联系，并附上涉嫌盗版材料的链接。

感谢您帮助保护我们的作者，以及我们为您提供有价值内容的能力。

# 问题

如果您在阅读本书的过程中遇到任何问题，可以通过`questions@packtpub.com`与我们联系，我们将尽力解决。
