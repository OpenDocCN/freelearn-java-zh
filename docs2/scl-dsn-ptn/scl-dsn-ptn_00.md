# 前言

软件工程和设计已经存在了许多年。我们在生活的方方面面都使用软件，这使得程序在解决问题的方面具有独特性。

无论可以用编程做什么，仍然有一些特定的特性会反复出现。随着时间的推移，人们已经提出了一些最佳实践，有助于解决程序中出现的特定模式。这些被称为设计模式。

设计模式不仅解决了常见问题，还处理了语言限制。无论具体的设计模式是什么以及它们解决的单个问题是什么，最终它们的目标都是生产出更好的软件。这包括提高可读性、简单性、易于维护性、可测试性、可扩展性和效率。如今，设计模式是每位优秀软件工程师工具箱中的重要组成部分。

除了我们用编程解决的问题的大量问题之外，还有许多我们可以使用的语言。每种语言都不同，有其优势和劣势，因此我们在做事情时也需要考虑这一点。在这本书中，我们将从 Scala 的角度来探讨设计模式。

在过去几年中，Scala 变得极为流行，使用它的人数也在不断增长。许多公司出于各种目的在生产中使用它——大数据处理、编写 API、机器学习等等。从像 Java 这样的流行语言切换到 Scala 实际上非常简单，因为它是一种面向对象语言和函数式编程语言的混合体。然而，要充分发挥 Scala 的潜力，我们需要熟悉不仅面向对象的特性，还要熟悉函数式特性。Scala 的使用可以提高性能和实现特性的时间。其中一个原因是 Scala 的高度可表达性。

Scala 接近面向对象语言的事实意味着许多面向对象编程的设计模式在这里仍然适用。它也是函数式编程的事实意味着一些其他的设计模式也适用，一些面向对象的模式可能需要修改以更好地适应 Scala 的范式。在这本书中，我们将关注所有这些——我们将探讨 Scala 的一些特定特性，然后从 Scala 的角度来看待流行的四人组设计模式。我们还将熟悉 Scala 特有的设计模式，并理解不同的函数式编程概念，包括幺半群和单子。有意义的例子总是使学习和理解更容易。我们将尝试提供你可以轻松映射到你可能会解决的实际问题的例子。我们还将介绍一些对编写现实世界应用程序的人有用的库。

# 这本书面向的对象是谁

本书面向已经对 Scala 有一定了解，但希望更深入地了解如何在实际应用开发中应用它的人。本书在应用设计时作为参考也很有用。理解使用最佳实践和编写良好代码的重要性是好的；然而，即使你没有，希望你在阅读完这本书后能被说服。不需要具备设计模式的知识，但如果你熟悉一些，这本书将很有用，因为我们将从 Scala 的角度来探讨它们。

# 为了充分利用本书

本书假设读者已经熟悉 Scala。我们为每个章节提供了使用 Maven 和 SBT 的项目示例。你应该对这两种工具中的任何一种有所了解，并且已经将其安装在您的机器上。我们还建议您在计算机上安装一个现代且最新的 IDE，例如 IntelliJ。我们鼓励您打开实际的项目，因为本书的示例专注于设计模式，在某些情况下，为了节省空间，省略了导入。

书中的示例是在基于 Unix 的操作系统上编写和测试的；然而，它们也应该能够在 Windows 上成功编译和运行。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择“支持”标签。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载完成后，请确保使用最新版本的以下软件解压或提取文件夹：

+   Windows 上的 WinRAR/7-Zip

+   Mac 上的 Zipeg/iZip/UnRarX

+   Linux 上的 7-Zip/PeaZip

本书代码包也托管在 GitHub 上，地址为[`github.com/PacktPublishing/Scala-Design-Patterns-Second-Edition`](https://github.com/PacktPublishing/Scala-Design-Patterns-Second-Edition)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富的书籍和视频目录的代码包可供选择，请访问 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。查看它们吧！

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“使用`extends`指定基类，然后使用`with`关键字添加所有特质。”

代码块设置如下：

```java
class MultiplierIdentity {
  def identity: Int = 1
}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
Error:(11, 8) object Clashing inherits conflicting members:
  method hello in trait A of type ()String and
  method hello in trait B of type ()String
(Note: this can be resolved by declaring an override in object Clashing.)
object Clashing extends A with B {
 ^
```

任何命令行输入或输出都应如下所示：

```java
Result 1: 6
Result 2: 2
Result 3: 6
Result 4: 6
Result 5: 6
Result 6: 3
```

**粗体**：表示新术语、重要单词或屏幕上出现的单词。

警告或重要注意事项看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：请将邮件发送至 `feedback@packtpub.com`，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过 `questions@packtpub.com` 发送邮件给我们。

**勘误表**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一错误。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式发现我们作品的非法副本，我们将不胜感激，如果您能提供位置地址或网站名称，请通过 `copyright@packtpub.com` 与我们联系，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且有兴趣撰写或参与一本书籍，请访问 [authors.packtpub.com](http://authors.packtpub.com/).

# 评论

请留下评论。一旦您阅读并使用了这本书，为何不在购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt 公司可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解 Packt 的更多信息，请访问 [packtpub.com](https://www.packtpub.com/).
