# 前言

Reactor 是 Java 9 反应式流规范的实现，这是一个用于异步数据处理的应用程序接口。该规范基于反应式编程范式，使开发者能够以降低复杂性和缩短时间的方式构建企业级、健壮的应用程序。*《Reactors 实践反应式编程》*向您展示了 Reactor 的工作原理，以及如何使用它来开发 Java 中的反应式应用程序。

本书从 Reactor 的基础知识及其在构建有效应用程序中的作用开始。您将学习如何构建完全非阻塞的应用程序，并随后将指导您了解发布者和订阅者 API。前四章将帮助您理解反应式流和 Reactor 框架。接下来的四章将使用 Reactor 构建一个基于 REST 的微服务 SpringWebFlux 扩展来构建 Web 服务。它们演示了流、背压和执行模型的概念。您将了解如何使用两个反应式可组合 API，Flux 和 Mono，这些 API 被广泛用于实现反应式扩展。在最后两章中，您将了解反应式流和 Reactor 框架。

各章节解释了最重要的部分，并构建简单的程序来建立基础。到本书结束时，您将获得足够的信心来构建反应式和可扩展的微服务。

# 本书面向的对象

对于任何对 Java 的基本概念有基本了解，并且能够轻松使用 Reactor 开发事件驱动和数据驱动应用程序的人来说，这本书是为您准备的——这是一本逐步指南，帮助您开始使用反应式流和 Reactor 框架。

# 本书涵盖的内容

第一章，*开始使用反应式流*，解释了反应式流 API，并介绍了反应式范式及其优势。本章还介绍了 Reactor 作为反应式流的实现。

第二章，*Reactors 中的发布者和订阅者 API*，解释了生产者和订阅者 API 以及 Reactor 的相应 Flux 和 Mono 影响。它还讨论了 Flux 和 Mono 的用例以及相应的接收器。我们还将探讨组件的热和冷变体。

第三章，*数据和流处理*，探讨了在发布者生成数据并被订阅者消费之前，我们如何处理这些数据，可用的可能操作，以及如何将它们组合起来构建一个健壮的流处理管道。流处理还涉及转换、旋转和聚合数据，然后生成新的数据。

第四章，*处理器*，介绍了 Reactor 中可用的开箱即用的处理器。处理器是特殊的发布者，也是订阅者，在实践之前理解为什么我们需要它们是非常重要的。

第五章，*SpringWebFlux for Microservices*，介绍了 SpringWebFlux，这是 Reactor 的 Web 扩展。它解释了 RouterFunction、HandlerFunction 和 FilterFunction 的概念。然后我们将使用 Mongo 作为存储来构建基于 REST 的微服务。

第六章，*动态渲染*，将模板引擎集成到我们在上一章中介绍的基于 REST 的微服务中，以渲染动态内容。它还演示了请求过滤器。

第七章，*流控制和背压*，讨论了流控制，这是反应式编程的一个重要方面，对于控制快速发布者的溢出是基本必需的。它还讨论了控制整个管道处理的各种方法。

第八章，*处理错误*，正如其标题所暗示的，解释了如何处理错误。所有 Java 开发者都习惯于错误处理的 try-catch-finally 块。本章将其转换为流处理。它还涵盖了如何从错误中恢复以及如何生成错误。这对于所有企业应用都是基本要求。

第九章，*执行控制*，探讨了 Reactor 中用于处理构建流的多种策略。它可以按某个间隔调度，或分组批量处理，或者所有操作都可以并行执行。

第十章，*测试和调试*，列出了我们可以测试流的方法，因为没有测试的开发是不完整的。我们将构建 JUnit 测试，这些测试将使用 Reactor 提供的某些测试工具来创建健壮的测试。本章还列出了调试在 Reactor 之上构建的异步流的方法。

# 为了充分利用这本书

+   理解基本 Java 概念是至关重要的

+   需要 Java 标准版，JDK 8，以及 IntelliJ IDEA IDE 或更高版本

# 下载示例代码文件

您可以从[www.packt.com](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packt.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packtpub.com/support)上登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载与勘误表”。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载后，请确保使用最新版本解压缩或提取文件夹：

+   Windows 上的 WinRAR/7-Zip

+   Mac 上的 Zipeg/iZip/UnRarX

+   Linux 上的 7-Zip/PeaZip

书籍的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Hands-On-Reactive-Programming-with-Reactor`](https://github.com/PacktPublishing/Hands-On-Reactive-Programming-with-Reactor)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富的图书和视频目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“所有`Subscribe`方法返回`Disposable`类型。此类型也可以用于取消订阅。”

代码块设置为如下所示：

```java
public interface Publisher<T> {
    public void subscribe(Subscriber<? super T> s);
}
```

任何命令行输入或输出都写成如下所示：

```java
gradlew bootrun
```

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。

警告或重要提示看起来是这样的。

小贴士和技巧看起来是这样的。

# 联系我们

我们读者的反馈总是受欢迎的。

**一般反馈**：如果您对本书的任何方面有疑问，请在邮件主题中提及书名，并给我们发送邮件至`customercare@packtpub.com`。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将非常感激您能向我们报告。请访问[www.packt.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上遇到我们作品的任何非法副本，我们将非常感激您能提供位置地址或网站名称。请通过`copyright@packt.com`与我们联系，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用过本书，为何不在您购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问[packt.com](https://www.packtpub.com/)。
