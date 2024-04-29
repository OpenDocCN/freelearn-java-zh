# 前言

测试驱动开发已经存在一段时间了，但仍然有许多人没有采用它。其原因在于 TDD 很难掌握。尽管理论很容易理解，但要真正熟练掌握它需要大量的实践。本书的作者们已经练习 TDD 多年，并将尝试将他们的经验传授给您。他们是开发人员，并相信学习一些编码实践的最佳方式是通过代码和不断的实践。本书遵循相同的理念。我们将通过练习来解释所有的 TDD 概念。这将是一次通过 Java 开发应用到 TDD 最佳实践的旅程。最终，您将获得 TDD 黑带，并在您的软件工艺工具包中多了一个工具。

# 这本书适合谁

如果您是一名经验丰富的 Java 开发人员，并希望实现更有效的系统和应用程序编程方法，那么这本书适合您。

# 要充分利用本书

本书中的练习需要读者拥有 64 位计算机。本书提供了所有所需软件的安装说明。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便文件直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  登录或注册[www.packtpub.com](http://www.packtpub.com/support)。

1.  选择“支持”选项卡。

1.  点击“代码下载和勘误”。

1.  在搜索框中输入书名，并按照屏幕上的说明进行操作。

文件下载后，请确保使用最新版本的解压缩或提取文件夹：

+   WinRAR/Windows 7-Zip

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Test-Driven-Java-Development-Second-Edition`](https://github.com/PacktPublishing/Test-Driven-Java-Development-Second-Edition)。如果代码有更新，将在现有的 GitHub 存储库上进行更新。

我们还有来自我们丰富书籍和视频目录的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。去看看吧！

# 下载彩色图像

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/TestDrivenJavaDevelopmentSecondEdition_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/TestDrivenJavaDevelopmentSecondEdition_ColorImages.pdf)[.](http://www.packtpub.com/sites/default/files/downloads/Bookname_ColorImages.pdf)

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这是一个例子："在这个测试中，我们定义了当调用`ticTacToe.play(5, 2)`方法时，期望出现`RuntimeException`。"

代码块设置如下：

```java
public class FooTest {
  @Rule
  public ExpectedException exception = ExpectedException.none();
  @Test
  public void whenDoFooThenThrowRuntimeException() {
    Foo foo = new Foo();
    exception.expect(RuntimeException.class);
    foo.doFoo();
  }
}
```

任何命令行输入或输出都是这样写的：

```java
    $ gradle test
```

**粗体**：表示一个新术语、一个重要词或屏幕上看到的词。例如，菜单或对话框中的单词会在文本中出现。这是一个例子："IntelliJ IDEA 提供了一个非常好的 Gradle 任务模型，可以通过点击 View|Tool Windows|Gradle 来访问。"

警告或重要说明会出现在这样的形式中。

提示和技巧会出现在这样的形式中。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：发送电子邮件至`feedback@packtpub.com`，并在主题中提及书名。如果您对本书的任何方面有疑问，请发送电子邮件至`questions@packtpub.com`。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误是难免的。如果您在本书中发现错误，请向我们报告。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表格链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何非法副本，请提供给我们地址或网站名称，我们将不胜感激。请通过`copyright@packtpub.com`与我们联系，并附上材料链接。

**如果您有兴趣成为作者**：如果您在某个专业领域有专长，并且有兴趣撰写或为一本书做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。在阅读并使用本书后，为什么不在购买书籍的网站上留下评论呢？潜在读者可以看到并使用您的客观意见来做出购买决定，我们在 Packt 可以了解您对我们产品的看法，我们的作者也可以看到您对他们书籍的反馈。谢谢！

有关 Packt 的更多信息，请访问[packtpub.com](https://www.packtpub.com/)。
