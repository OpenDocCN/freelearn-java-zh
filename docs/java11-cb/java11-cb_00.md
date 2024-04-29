# 前言

本食谱书提供了一系列软件开发示例，这些示例通过简单直接的代码进行了说明，提供了逐步资源和节省时间的方法，帮助您高效解决数据问题。从安装 Java 开始，每个食谱都解决了一个特定的问题，并附有解决方案的讨论，以及解释其工作原理的见解。我们涵盖了关于核心编程语言的主要概念，以及构建各种软件所涉及的常见任务。您将按照食谱了解最新 Java 11 版本的新功能，使您的应用程序模块化、安全和快速。

# 本书的受众包括初学者、有中级经验的程序员，甚至专家；所有人都能够访问这些食谱，这些食谱演示了 Java 11 发布的最新功能。

预期读者包括初学者、有中级经验的程序员，甚至专家；所有人都能够访问这些食谱，这些食谱演示了 Java 11 发布的最新功能。

# 为了充分利用本书

为了充分利用本书，需要一些 Java 知识和运行 Java 程序的能力。此外，最好安装和配置了您喜爱的编辑器或 IDE 以供在食谱中使用。因为本书本质上是一本食谱集，每个食谱都是基于具体示例的，如果读者不执行提供的示例，将会失去本书的好处。读者如果在他们的 IDE 中复制每个提供的示例，执行它，并将其结果与书中显示的结果进行比较，将会从本书中获得更多。

# 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便文件直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择“支持”选项卡。

1.  单击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

下载文件后，请确保使用以下最新版本解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Java-11-Cookbook-Second-Edition`](https://github.com/PacktPublishing/Java-11-Cookbook-Second-Edition)。我们还有其他代码包，来自我们丰富的图书和视频目录，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上获得。去看看吧！

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这是一个例子："使用`ProcessHandle`接口上的`allProcesses()`方法获取当前活动进程的流"

代码块设置如下：

```java
public class Thing {
  private int someInt;
  public Thing(int i) { this.someInt = i; }
  public int getSomeInt() { return someInt; }
  public String getSomeStr() { 
    return Integer.toString(someInt); }
} 
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目以粗体设置：

```java
Object[] os = Stream.of(1,2,3).toArray();
 Arrays.stream(os).forEach(System.out::print);
 System.out.println();
 String[] sts = Stream.of(1,2,3)
                      .map(i -> i.toString())
                      .toArray(String[]::new);
 Arrays.stream(sts).forEach(System.out::print);
```

任何命令行输入或输出都写成如下形式：

```java
jshell> ZoneId.getAvailableZoneIds().stream().count()
$16 ==> 599
```

**粗体**：表示一个新术语、一个重要词或屏幕上看到的词。例如，菜单或对话框中的单词会在文本中以这种方式出现。这是一个例子："右键单击“我的电脑”，然后单击“属性”。您会看到

您的系统信息。"

警告或重要说明看起来像这样。

提示和技巧看起来像这样。

# 部分

本书中，您会发现一些经常出现的标题（*准备工作*、*如何做*、*它是如何工作的*、*还有更多*和*另请参阅*）。为了清晰地说明如何完成一个食谱，使用以下部分：

# 准备工作

本节告诉您在食谱中可以期待什么，并描述了如何设置食谱所需的任何软件或初步设置。

# 如何做...

本节包含了遵循食谱所需的步骤。

# 工作原理...

本节通常包括对前一节发生的事情的详细解释。

# 还有更多...

本节包括有关食谱的其他信息，以使您对食谱更加了解。

# 另请参阅

本节提供了有关食谱的其他有用信息的链接。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**: 如果您对本书的任何方面有疑问，请在邮件主题中提及书名，并发送电子邮件至`customercare@packtpub.com`。

**勘误**: 尽管我们已经尽一切努力确保内容的准确性，但错误确实会发生。如果您在本书中发现错误，我们将不胜感激您向我们报告。请访问[www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书，点击勘误提交表格链接，并输入详细信息。

**盗版**: 如果您在互联网上发现我们作品的任何形式的非法副本，我们将不胜感激您向我们提供位置地址或网站名称。请通过`copyright@packtpub.com`与我们联系，并提供材料链接。

**如果您有兴趣成为作者**: 如果您在某个专题上有专业知识，并且有兴趣撰写或为一本书做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用了本书，为什么不在购买书籍的网站上留下评论呢？潜在读者可以看到并使用您的客观意见来做出购买决定，我们在 Packt 可以了解您对我们产品的看法，我们的作者可以看到您对他们书籍的反馈。谢谢！

有关 Packt 的更多信息，请访问[packt.com](https://www.packt.com/)。
