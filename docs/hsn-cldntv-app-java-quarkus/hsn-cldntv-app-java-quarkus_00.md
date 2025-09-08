# 前言

尽管 Java 已经存在多年，但它仍然是开发者中最受欢迎的选择之一，经历了近 20 年的持续改进，并为企业系统开发了一套完整的库。在我们生活在一个快节奏的行业中，我们不能否认，在容器、微服务、反应式应用程序和云平台引入的过去几年里，许多事情都发生了变化。为了在这个不断变化的世界中保持一流的地位，Java 需要一次提升。而我们认为，这次提升可以是 Quarkus！

Quarkus 是从头开始设计的，旨在成为一个 Kubernetes 原生 Java 框架，非常适合创建具有最小内存占用和快速执行速度的微服务应用程序。同时，Quarkus 并没有丢弃那些大多数开发者熟悉的丰富库集，例如 Hibernate 和 REST 服务。相反，它们是更大图景的一部分，包括颠覆性的 MicroProfile API、反应式编程模型如 Vert.x 以及许多其他可以轻松集成到您的服务中的功能。

由于 Quarkus 是针对这些考虑而设计的，因此它为在无服务器、微服务、容器、Kubernetes、**函数即服务**（**FaaS**）和云中运行 Java 提供了一个有效的解决方案。它的以容器优先的方法将命令式和反应式编程范式统一于微服务开发中，并提供了一组可扩展的基于标准的 企业 Java 库和框架，结合了极端的开发者生产力，承诺将彻底改变我们用 Java 开发的方式。

# 本书面向对象

这本书是为对学习构建可靠和健壮应用程序的非常有前途的微服务架构感兴趣的 Java 开发者和软件架构师而编写的。假设读者对 Java、Spring 和 REST API 有一定的了解。

# 本书涵盖内容

第一章，*Quarkus 核心概念简介*，解释了以容器优先（最小内存占用的 Java 应用程序在容器中运行是最优的）、云原生（Quarkus 接受了 Kubernetes 等环境中的 12 因素架构）和微服务优先（Quarkus 为 Java 应用程序带来了闪电般的启动时间和代码周转时间）的方法。我们将检查可用于开发 Quarkus 应用程序的各种工具。为此，我们将安装 IDE 和 GraalVM，这是原生应用程序所必需的。

第二章，*使用 Quarkus 开发您的第一个应用程序*，将带您通过使用 Quarkus 构建您的第一个应用程序。我们将看到如何使用 Maven/Gradle 插件来引导应用程序。您将学习如何将应用程序导入您的 IDE，以及如何运行和测试应用程序。接下来，我们将讨论如何从您的 Java 项目中创建原生应用程序。

第三章，*创建应用程序容器镜像*，探讨了如何构建应用程序的 Docker 镜像，如何在 Docker 上运行应用程序，如何安装 OpenShift 的单节点集群，以及如何在 Minishift 上运行应用程序。

第四章，*Web 应用程序开发*，探讨了将使用 REST 和 CDI 堆栈以及 Web 前端的一个客户商店应用程序的使用案例。我们将看到如何部署应用程序，并查看更改而不需要重启 Quarkus。

第五章，*使用 Quarkus 管理数据持久性*，讨论了 Quarkus 的数据持久性。我们将看到如何将持久性添加到客户商店示例中，以及如何设置数据库（PostgreSQL）以运行示例。然后，我们将把应用程序带到云端。最后，我们将向您展示 Hibernate Panache 如何简化应用程序开发。

第六章，*使用 MicroProfile API 构建应用程序*，教您如何将我们已讨论的企业 API 与 Eclipse MicroProfile 的完整规范栈相补充（[`microprofile.io/`](https://microprofile.io/))。

第七章，*保护应用程序安全*，将探讨如何使用内置的安全 API（如 Elytron 安全堆栈、Keycloak 扩展和 MicroProfile JWT 扩展）保护我们的示例应用程序。

第八章，*高级应用程序开发*，探讨了高级应用程序开发技术，如高级应用程序配置管理、生命周期事件和触发计划任务。

第九章，*统一命令式和响应式*，通过 Quarkus 和 Vert.x 编程模型的一个示例应用程序，带您了解非阻塞编程模型。我们还将探讨如何利用 Vert.x 的响应式 SQL API 构建非阻塞数据库应用程序。

第十章，*使用 Quarkus 进行响应式消息传递*，解释了如何使用 CDI 开发模型和 Kafka 以及 AMQP 作为代理来实现响应式数据流应用程序。我们还将解释如何通过在 OpenShift 上部署我们的应用程序来实现云中的完整流式架构。

# 为了充分利用这本书

《*使用 Java 和 Quarkus 进行实战云原生应用开发*》是一本完整的端到端开发指南，它将为您提供在无服务器环境中构建 Kubernetes 原生应用的实战经验。为了最大限度地利用本书，我们建议您使用集成了 Apache Maven（如 IntelliJ IDEA 或 Eclipse）的开发环境，并将其导入我们的示例代码文件中。这将帮助您逐步跟踪我们的项目，并在需要时进行调试。

# 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的账户下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择支持选项卡。

1.  点击代码下载。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载完成后，请确保使用最新版本的以下软件解压或提取文件夹：

+   Windows 版的 WinRAR/7-Zip

+   Mac 版的 Zipeg/iZip/UnRarX

+   Linux 版的 7-Zip/PeaZip

本书代码包也托管在 GitHub 上，地址为[`github.com/PacktPublishing/Hands-On-Cloud-Native-Applications-with-Java-and-Quarkus`](https://github.com/PacktPublishing/Hands-On-Cloud-Native-Applications-with-Java-and-Quarkus)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还提供来自我们丰富的书籍和视频目录中的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表的彩色图像的 PDF 文件。您可以从这里下载：[`static.packt-cdn.com/downloads/9781838821470_ColorImages.pdf`](https://static.packt-cdn.com/downloads/9781838821470_ColorImages.pdf)。

# 代码实战

请访问以下链接查看代码实战视频：[`bit.ly/2LKFbY1`](http://bit.ly/2LKFbY1)

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“我们的项目中提供了一个`index.html`页面作为标记，如图所示的项目层次结构。”

代码块设置如下：

```java
 // Create new JSON for Order #1
    objOrder = Json.createObjectBuilder()
            .add("id", new Long(1))
            .add("item", "mountain bike")
            .add("price", new Long(100))
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
@POST
@RolesAllowed("admin")
public Response create(Customer customer)
```

任何命令行输入或输出都应如下编写：

```java
$ tree src
```

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“一旦您有一些数据，其他操作（如编辑和删除）将可用。”

警告或重要注意事项看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请在邮件主题中提及书名，并将邮件发送至`customercare@packtpub.com`.

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一点。请访问[www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过`copyright@packt.com`与我们联系，并附上材料的链接。

**如果您想成为一名作者**：如果您在某个领域有专业知识，并且对撰写或参与一本书籍感兴趣，请访问[authors.packtpub.com](http://authors.packtpub.com/).

# 评论

请留下评论。一旦您阅读并使用过这本书，为何不在您购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt 公司可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解 Packt 的更多信息，请访问[packt.com](http://www.packt.com/).
