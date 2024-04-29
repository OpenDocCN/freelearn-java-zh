# 前言

世界正在以前所未有的规模相互连接，互联网上提供的服务越来越多。从业务交易到嵌入式应用程序（如冰箱中的应用程序），各种应用程序都在连接到互联网。随着孤立的应用程序不再是常态，应用程序的网络功能变得越来越重要。

本书的目标是为读者提供开发 Java 应用程序的必要技能，使其能够连接并与网络上的其他应用程序和服务一起工作。您将了解使用 Java 提供的各种网络选项，从而能够开发出适合当前任务的应用程序。

# 本书涵盖的内容

第一章, *开始网络编程*，介绍了基本的网络术语和概念。演示了 Java 提供的网络支持，并提供了简短的示例。介绍了一个简单的客户端/服务器应用程序，以及服务器的线程化版本。

第二章, *网络寻址*，解释了网络上的节点如何使用地址。介绍了 Java 如何表示这些地址，以及对 IPv4 和 IPv6 的支持。本章还涵盖了 Java 如何配置各种网络属性。

第三章, *NIO 支持网络*，解释了 NIO 包如何支持使用缓冲区和通道进行通信。演示了这些技术，并演示了 NIO 为异步通信提供的支持。

第四章, *客户端/服务器开发*，介绍了 HTTP 是一种重要且广泛使用的协议。Java 以多种方式支持该协议。演示了这些技术，并演示了 Java 如何处理 cookie。

第五章, *点对点网络*，讨论了点对点网络提供了传统客户端/服务器架构的灵活替代方案。介绍了基本的点对点概念，并演示了 Java 如何支持这种架构。使用 FreePastry 演示了一个开源的点对点解决方案框架。

第六章, *UDP 和多播*，解释了 UDP 是 TCP 的一种替代方案。它为应用程序在互联网上进行通信提供了一种不太可靠但更高效的方式。演示了 Java 对该协议的广泛支持，包括 NIO 支持，以及 UDP 如何支持流媒体。

第七章, *网络可扩展性*，解释了随着服务器承担更多需求，系统需要扩展以满足这些需求。演示了支持这一需求的几种线程技术，包括线程池、futures 和 NIO 的选择器。

第八章, *网络安全*，讨论了应用程序需要防范各种威胁。Java 支持使用加密和安全哈希技术来实现这一点。对称和非对称加密技术得到了说明。此外，还演示了 TLS/SSL 的使用。

第九章, *网络互操作性*，介绍了 Java 应用程序可能需要与用不同语言编写的其他应用程序交换信息的情况。探讨了影响应用程序互操作性的问题，包括字节顺序。使用套接字和中间件演示了不同实现之间的通信。

# 本书所需内容

本书中遇到的网络编程示例需要 Java SDK 1.8。建议使用 NetBeans 或 Eclipse 等集成开发环境。本书使用 NetBeans IDE 8.0.2 EE 版来说明 Web 服务的开发。

# 本书适合谁

这本书是为那些已经精通 Java 并希望学习如何开发网络应用的开发人员准备的。只需要熟悉基本的 Java 和面向对象编程概念。您将学习网络编程的基础知识，以及如何使用多种不同的套接字来创建安全和可扩展的应用程序。

# 约定

在本书中，您将找到一些区分不同信息类型的文本样式。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名显示如下："`SSLSocketFactory`类的`getDefault`返回一个`SSLSocketFactory`实例，其`createSocket`创建一个连接到安全回显服务器的套接字。"

代码块设置如下：

```java
public class ThreadedEchoServer implements Runnable {
    private static Socket clientSocket;

    public ThreadedEchoServer(Socket clientSocket) {
        this.clientSocket = clientSocket;
    }
    ...
}
```

任何命令行输入或输出都以以下方式编写：

```java
Enter keystore password:
Re-enter new password:
What is your first and last name?
 [Unknown]:  First Last
What is the name of your organizational unit?
 [Unknown]:  packt
What is the name of your organization?
 [Unknown]:  publishing
What is the name of your City or Locality?
 [Unknown]:  home
What is the name of your State or Province?
 [Unknown]:  calm
What is the two-letter country code for this unit?
 [Unknown]:  me
Is CN=First Last, OU=packt, O=publishing, L=home, ST=calm, C=me correct?
 [no]:  y

Enter key password for <mykey>
 (RETURN if same as keystore password):

```

**新术语**和**重要单词**以粗体显示。例如，屏幕上看到的单词，比如菜单或对话框中的单词，会以这样的方式出现在文本中："一旦安装了 NetBeans，启动它，然后从**文件** | **新建项目…**菜单项创建一个新项目。"

### 注意

警告或重要提示会以这样的方式显示在一个框中。

### 提示

提示和技巧会以这样的方式出现。

# 读者反馈

我们的读者反馈总是受欢迎的。让我们知道你对这本书的看法——你喜欢或不喜欢什么。读者的反馈对我们很重要，因为它帮助我们开发你真正能充分利用的标题。

要向我们发送一般反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在主题中提及书名。

如果您在某个专题上有专业知识，并且有兴趣撰写或为一本书做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的自豪所有者，我们有一些事情可以帮助您充分利用您的购买。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载示例代码文件，这适用于您购买的所有 Packt Publishing 图书。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便文件直接通过电子邮件发送给您。

## 下载本书的彩色图像

我们还为您提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。彩色图像将帮助您更好地理解输出中的变化。您可以从以下网址下载此文件：[`www.packtpub.com/sites/default/files/downloads/LearningNetworkProgrammingwithJava_Graphics.pdf`](https://www.packtpub.com/sites/default/files/downloads/LearningNetworkProgrammingwithJava_Graphics.pdf)。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误确实会发生。如果您在我们的书中发现错误——也许是文本或代码中的错误——我们将不胜感激，如果您能向我们报告。通过这样做，您可以帮助其他读者避免挫折，并帮助我们改进本书的后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)报告，选择您的书，点击**勘误提交表**链接，并输入您的勘误详情。一旦您的勘误被验证，您的提交将被接受，并且勘误将被上传到我们的网站或添加到该标题的勘误部分的任何现有勘误列表中。

要查看先前提交的勘误表，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需信息将显示在**勘误表**部分下。

## 盗版

互联网上盗版受版权保护的材料是所有媒体持续存在的问题。在 Packt，我们非常重视版权和许可的保护。如果您在互联网上发现我们作品的任何非法副本，请立即向我们提供位置地址或网站名称，以便我们采取补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并附上涉嫌盗版材料的链接。

感谢您帮助保护我们的作者和我们提供有价值内容的能力。

## 问题

如果您对本书的任何方面有问题，可以通过`<questions@packtpub.com>`与我们联系，我们将尽力解决问题。
