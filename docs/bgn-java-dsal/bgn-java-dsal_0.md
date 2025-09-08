# 前言

数据结构是一种组织数据的方式，以便可以高效地访问和/或修改。了解数据结构和算法可以帮助你更好地洞察如何解决常见的编程问题。程序员每天面临的许多问题都已经得到解决、尝试和测试。了解这些解决方案的工作原理，确保在面临这些问题时选择正确的工具。

本书教你使用工具来构建高效的应用程序。它从算法和大数据符号的介绍开始，随后解释了冒泡排序、归并排序、快速排序和其他流行的编程模式。你还将了解二叉树、哈希表和图等数据结构。本书逐步深入到高级概念，如算法设计范式和图论。到本书结束时，你将知道如何在应用程序中正确实现常见的算法和数据结构。

# 本书面向对象

如果你希望通过 Java 代码示例更好地理解常见的数据结构和算法，并提高你的应用程序效率，那么这本书适合你。具备基本的 Java、数学和面向对象编程技术知识会有所帮助。

# 本书涵盖内容

第一章，*算法与复杂度*，涵盖了如何定义算法、衡量算法复杂度以及识别不同复杂度的算法。它还涵盖了如何评估具有不同运行时间复杂度的各种示例。

第二章，*排序算法和基本数据结构*，探讨了冒泡排序、快速排序和归并排序。我们还将介绍数据结构，并研究链表、队列和栈的各种实现和使用案例。我们还将看到一些数据结构如何用作构建更复杂结构的基石。

第三章，*哈希表和二叉搜索树*，讨论了实现数据字典操作的数据结构。此外，二叉树还使我们能够执行各种范围查询。我们还将看到这两种数据结构的示例，以及这些操作的实现。

第四章，*算法设计范式*，讨论了三种不同的算法设计范式以及示例问题，并讨论了如何识别问题是否可能由给定的范式之一解决。

第五章，*字符串匹配算法*，介绍了字符串匹配问题。本章还介绍了字符串匹配算法，从简单的搜索算法开始，并使用 Boyer 和 Moore 提出的规则进行改进。我们还将探索一些其他字符串匹配算法，而不会过多地深入细节。

第六章，*图、素数和复杂度类*，介绍了图，形式化它们是什么，并展示了在计算机程序中代表它们的两种不同方式。稍后，我们将探讨遍历图的方法，使用它们作为构建更复杂算法的构建块。我们还将探讨在图中找到最短路径的两种不同算法。

# 为了充分利用本书

为了成功完成本书，您需要一个至少配备 i3 处理器的计算机系统，4 GB RAM，10 GB 硬盘和互联网连接。此外，您还需要以下软件：

+   Java SE 开发工具包，JDK 8（或更高版本）

+   Git 客户端

+   Java IDE（IntelliJ 或 Eclipse）

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择“支持”标签。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书籍名称，并按照屏幕上的说明操作。

文件下载后，请确保使用最新版本的软件解压缩或提取文件夹：

+   Windows 的 WinRAR/7-Zip

+   Mac 的 Zipeg/iZip/UnRarX

+   Linux 的 7-Zip/PeaZip

本书代码包也托管在 GitHub 上，网址为[`github.com/TrainingByPackt/Data-Structures-and-Algorithms-in-Java`](https://github.com/TrainingByPackt/Data-Structures-and-Algorithms-in-Java)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有来自我们丰富图书和视频目录的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/BeginningJavaDataStructuresandAlgorithms_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/BeginningJavaDataStructuresandAlgorithms_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“这是`merge()`函数的任务，该函数位于前一个部分中显示的伪代码的末尾。”

代码块设置为以下格式：

```java
quickSort(array, start, end)
if(start < end)
p = partition(array, start, end)
quickSort(array, start, p - 1)
quickSort(array, p + 1, end)
```

任何命令行输入或输出都按以下方式编写：

```java
gradlew test --tests com.packt.datastructuresandalg.lesson2.activity.selectionsort*
```

**粗体**: 表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。这里有一个例子：“选择动态 Web 项目，然后点击下一步以打开动态 Web 项目向导。”

**活动**: 这些是基于场景的活动，将让您在实际应用中应用您在整章中学到的知识。它们通常是在现实世界问题或情境的背景下。

警告或重要注意事项显示如下。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**: 请通过`feedback@packtpub.com`发送电子邮件，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过`questions@packtpub.com`发送电子邮件给我们。

**勘误**: 尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，如果您能向我们报告，我们将不胜感激。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**: 如果您在互联网上以任何形式遇到我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过`copyright@packtpub.com`与我们联系，并提供材料的链接。

**如果您有兴趣成为作者**: 如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用过这本书，为什么不在您购买它的网站上留下评论呢？潜在读者可以看到并使用您的客观意见来做出购买决定，Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解 Packt 的更多信息，请访问[packtpub.com](https://www.packtpub.com/)。
