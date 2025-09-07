# 前言

Java 语言是与一个非常丰富的平台进行通信的工具，该平台提供了许多为应用程序开发准备就绪的功能。本书通过最有用的设计模式的示例探讨了语言语法的最新发展。本书通过示例实现揭示了功能、模式和平台效率之间的关系。本书探讨了理论基础如何帮助提高源代码的可维护性、效率和可测试性。内容帮助读者解决不同的任务，并提供了使用各种可持续和透明方法解决编程挑战的指导。 

# 本书面向的对象

本书献给所有渴望提高软件设计技能的“饥渴”工程师，他们希望通过新的语言增强和更深入地了解 Java 平台来实现这一目标。

# 本书涵盖的内容

*第一章*, *进入软件设计模式*，介绍了源代码设计结构的初步基础，并概述了应遵循的原则以实现可维护性和可读性。

*第二章*, *探索 Java 平台以应用设计模式*，讨论了 Java 平台，这是一个非常广泛且强大的工具。本章更详细地介绍了 Java 平台的功能、功能和设计，以继续构建理解使用设计模式的目的和价值的必要基础。

*第三章*, *与创建型设计模式一起工作*，探讨了对象实例化，这是任何应用程序的关键部分。本章描述了在考虑需求的同时如何应对这一挑战。

*第四章*, *应用结构型设计模式*，展示了如何创建允许所需对象之间关系清晰的源代码。

*第五章*, *行为设计模式*，探讨了如何创建允许对象进行通信和交换信息的同时保持透明形式的源代码。

*第六章*, *并发设计模式*，讨论了 Java 平台及其本质上是一个并发环境。它展示了如何利用其力量为设计应用程序的目的服务。

*第七章*, *理解常见反模式*，处理在任何应用程序开发周期中可能遇到的反模式。它将帮助您处理根本原因及其识别，并提出可能的反模式补救措施。

# 要充分利用本书

要执行本书中的说明，您需要以下内容：

| **本书涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |
| Java 开发工具包 17+ | Windows、macOS 或 Linux |
| 推荐的 IDE VSCode 1.73.1+ | Windows、macOS 或 Linux |
| 文本编辑器或 IDE | Windows、macOS 或 Linux |

对于本书，需要安装 Java 开发工具包 17+。要验证它是否在您的系统上可用，请执行以下命令：

+   Windows 命令提示符：`java –version`

+   Linux 或 macOS 系统命令行：`java –version`

预期输出：

```java
openjdk version "17" 2021-09-14
OpenJDK Runtime Environment (build 17+35-2724)
OpenJDK 64-Bit Server VM (build 17+35-2724, mixed mode,
sharing)
```

如果您在本地机器上没有安装 JDK，请在[`dev.java/learn/getting-started-with-java/`](https://dev.java/learn/getting-started-with-java/)搜索您平台的相关说明，并在[`jdk.java.net/archive/`](https://jdk.java.net/archive/)找到匹配的 JDK 版本。

要下载和安装 Visual Studio Code，请访问[`code.visualstudio.com/download`](https://code.visualstudio.com/download)。

以下页面描述并指导了 VSCode 终端的使用：[`code.visualstudio.com/docs/terminal/basics`](https://code.visualstudio.com/docs/terminal/basics)。

# 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件：[`github.com/PacktPublishing/Practical-Design-Patterns-for-Java-Developers`](https://github.com/PacktPublishing/Practical-Design-Patterns-for-Java-Developers)。如果代码有更新，它将在 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图和图表的彩色 PDF 文件。您可以从这里下载：[`packt.link/nSLEf`](https://packt.link/nSLEf)。

# 使用的约定

本书使用了多种文本约定。

`文本中的代码`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“让我们检查`Vehicle`类开发中的泛化过程。”

代码块设置如下：

```java
public class Vehicle {
    private boolean moving;
    public void move(){
        this.moving = true;
        System.out.println("moving...");
    }
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
sealed interface Engine permits ElectricEngine,
    PetrolEngine  {
    void run();
    void tank();
}
```

任何命令行输入或输出都应如下所示：

```java
$ mkdir main
$ cd main
```

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词以粗体显示。以下是一个示例：“字节码正在运行一个**Java 虚拟****机器**（**JVM**）。”

小贴士或重要注意事项

看起来像这样。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请通过电子邮件发送给我们 customercare@packtpub.com，并在邮件主题中提及书名。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告，请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表格。

**盗版**：如果您在互联网上发现我们作品的任何形式的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过 mailto:copyright@packt.com 与我们联系，并提供材料的链接。

**如果您想成为一名作者**：如果您在某个领域有专业知识，并且对撰写或参与书籍感兴趣，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

一旦您阅读了*Java 开发者实用设计模式*，我们很乐意听到您的想法！[请点击此处直接进入此书的亚马逊评论页面](https://packt.link/r/1-804-61467-X)并分享您的反馈。

您的评论对我们和科技社区非常重要，并将帮助我们确保我们提供高质量的内容。

# 下载本书的免费 PDF 副本

感谢您购买本书！

您喜欢在路上阅读，但无法携带您的印刷书籍到处走？您的电子书购买是否与您选择的设备不兼容？

不要担心，现在，随着每本 Packt 书籍，您都可以免费获得该书的 DRM 免费 PDF 版本。

在任何地方、任何设备上阅读。直接从您最喜欢的技术书籍中搜索、复制和粘贴代码到您的应用程序中。

优惠远不止于此，您还可以获得独家折扣、时事通讯和每日免费内容的每日电子邮件。

按照以下简单步骤获取好处：

1.  扫描二维码或访问以下链接

![](img/B18884_QR_Free_PDF.jpg)

[`packt.link/free-ebook/9781804614679`](https://packt.link/free-ebook/9781804614679)

1.  提交您的购买证明

1.  就这样！我们将直接将免费 PDF 和其他优惠发送到您的电子邮件。

# 第一部分：设计模式和 Java 平台功能

本部分涵盖了软件设计模式的目的。它概述了面向对象编程 APIE 和 SOLID 设计原则的基本思想，并介绍了 Java 平台，这对于理解如何有效地利用设计模式至关重要。

本部分包含以下章节：

+   *第一章*，*进入软件设计模式*

+   *第二章*，*探索 Java 平台的设计模式*
