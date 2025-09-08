# 前言

随着 Java 以极快的速度发展，程序员必须了解最新的发展，以便在他们的应用程序和库中充分利用其新特性。

本书将带你了解 Java 语言的发展，从 Java 10 到 Java 11，再到 Java 12。本书深入探讨了语言的最新发展。你将学习这些特性如何帮助你用该语言推进开发，并使你的应用程序更加精简和快速。

你还将发现如何配置虚拟机以减少启动时间，从而解决未来在吞吐量和延迟方面的挑战。借助这本书，你将克服迁移到 Java 新版本时遇到的挑战。

# 本书面向对象

如果你是一位负责技术选择或 Java 迁移决策的执行者或解决方案架构师，这本书适合你。如果你是一位对最新和即将推出的 Java 特性感兴趣的计算机科学爱好者，你也将从这本书中受益。"Java 11 和 12 – 新特性"将帮助你将解决方案从 Java 8 或更早版本迁移到最新的 Java 版本。

# 本书涵盖内容

第一章*,* *类型推断*，介绍了 Java 10 中引入的局部变量类型推断。你将学习如何使用`var`关键字以及可能遇到的挑战。

第二章，*AppCDS*，涵盖了**应用程序类数据共享**（**AppCDS**），它扩展了**类数据共享**（**CDS**）。你将了解两者并看到它们在实际中的应用。

第三章*,* *垃圾收集器优化*，讨论了各种垃圾收集器及其高效实现接口。

第四章*,* *JDK 10 的杂项改进*，涵盖了 Java 10 中的特性和改进。

第五章*,* *Lambda 参数的局部变量语法*，解释了 Lambda 参数的局部变量语法，并介绍了与 Lambda 参数一起使用`var`的用法。本章还涵盖了其语法和用法，以及你可能会遇到的挑战。

第六章*,* *Epsilon GC*，探讨了 Java 11 引入的 Epsilon，它降低了垃圾收集的延迟。本章解释了为什么需要它以及其设计考虑。

第七章*,* *HTTP 客户端 API*，讨论了 HTTP 客户端 API，它使你的 Java 代码能够通过网络请求 HTTP 资源。

第八章，*ZGC*，探讨了一种名为 ZGC 的新 GC，它具有低延迟的可扩展性。你将了解其特性和通过示例进行操作。

第九章，*飞行记录仪和任务控制*，讨论了 JFR 分析器，它有助于记录数据，以及 MC 工具，它有助于分析收集到的数据。

第十章，*JDK 11 中的各种改进*，涵盖了 Java 11 中的特性和改进。

第十一章，*切换表达式*，介绍了`switch`表达式，这是 Java 12 中增强的基本语言结构。您将学习如何使用这些表达式使您的代码更高效。

第十二章，*JDK 12 中的各种改进*，涵盖了 Java 12 中的特性和改进。

第十三章*，*Project Amber 中的增强枚举*，展示了枚举如何为常量引入类型安全。本章还涵盖了每个枚举常量可以拥有其独特的状态和行为。

第十四章*，*数据类及其用法*，介绍了 Project Amber 中的数据类如何通过定义数据载体类来推动语言变化。

第十五章*，*原始字符串字面量*，讨论了开发者在将各种类型的多行文本值作为字符串值存储时面临的挑战。原始字符串字面量解决了这些问题，同时也显著提高了多行字符串值的可编写性和可读性。

第十六章*，Lambda Leftovers*，展示了 lambda leftovers 项目如何改进 Java 中的函数式编程语法和体验。

第十七章*，*模式匹配*，通过编码示例帮助您理解模式匹配如何改变您编写日常代码的方式。

# 为了充分利用这本书

一些先前的 Java 知识将有所帮助，并且所有必要的说明都已添加到相应的章节中。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载这本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择 SUPPORT 标签。

1.  点击代码下载与勘误。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

下载文件后，请确保您使用最新版本的软件解压缩或提取文件夹：

+   适用于 Windows 的 WinRAR/7-Zip

+   适用于 Mac 的 Zipeg/iZip/UnRarX

+   适用于 Linux 的 7-Zip/PeaZip

本书代码包也托管在 GitHub 上，网址为 [`github.com/PacktPublishing/Java-11-and-12-New-Features`](https://github.com/PacktPublishing/Java-11-and-12-New-Features)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还提供其他代码包，这些代码包来自我们丰富的书籍和视频目录，可在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 上找到。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表的彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/9781789133271_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/9781789133271_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`: 表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“`PUT`请求用于在服务器上创建或更新实体，使用 URI。”

代码块设置为如下：

```java
class GarmentFactory { 
    void createShirts() { 
        Shirt redShirtS = new Shirt(Size.SMALL, Color.red); 
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
abstract record JVMLanguage(String name, int year); 
record Conference(String name, String venue, DateTime when); 
```

任何命令行输入或输出都写作如下：

```java
  java -Xshare:dump   
```

**粗体**: 表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“如您所注意到的，Lock Instances 选项旁边显示了一个感叹号。”

警告或重要提示看起来像这样。

小技巧和技巧看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**: 通过电子邮件 `feedback@packtpub.com` 发送，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过 `questions@packtpub.com` 发送电子邮件给我们。

**勘误**: 尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，如果您能向我们报告，我们将不胜感激。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**: 如果您在互联网上以任何形式遇到我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过 `copyright@packtpub.com` 联系我们，并提供材料的链接。

**如果您有兴趣成为作者**: 如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下您的评价。一旦您阅读并使用了这本书，为何不在购买它的网站上留下评价呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 可以了解您对我们产品的看法，而我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解 Packt 的更多信息，请访问 [packtpub.com](https://www.packtpub.com/).
