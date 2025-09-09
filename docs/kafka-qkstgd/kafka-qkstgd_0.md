# 前言

自 2011 年以来，Kafka 在增长方面呈爆炸式增长。超过三分之一的《财富》500 强公司使用 Apache Kafka。这些公司包括旅游公司、银行、保险公司和电信公司。

Uber、Twitter、Netflix、Spotify、Blizzard、LinkedIn、Spotify 和 PayPal 每天使用 Apache Kafka 处理他们的消息。

今天，Apache Kafka 被用于收集数据、进行实时数据分析以及执行实时数据流。Kafka 还被用于向**复杂事件处理**（**CEP**）架构提供事件，部署在微服务架构中，并在**物联网**（**IoT**）系统中实现。

在流处理领域，有几个 Kafka Streams 的竞争对手，包括 Apache Spark、Apache Flink、Akka Streams、Apache Pulsar 和 Apache Beam。它们都在竞争以超越 Kafka。然而，Apache Kafka 在所有这些方面都有一个关键优势：其易用性。Kafka 易于实现和维护，其学习曲线并不陡峭。

本书是一本实用的快速入门指南。它专注于展示实际示例，不涉及 Kafka 架构的理论解释或讨论。本书是实际操作食谱的汇编，为实施 Apache Kafka 的人提供日常问题的解决方案。

# 本书面向的对象

本书面向数据工程师、软件开发人员和数据架构师，他们寻找快速上手 Kafka 的指南。

本指南是关于编程的；它是为那些对 Apache Kafka 没有先验知识的人提供的入门介绍。

所有示例均使用 Java 8 编写；对 Java 8 的了解是遵循本指南的唯一要求。

# 本书涵盖的内容

第一章，*配置 Kafka*，解释了开始使用 Apache Kafka 的基本知识。它讨论了如何安装、配置和运行 Kafka。它还讨论了如何使用 Kafka 代理和主题进行基本操作。

第二章，*消息验证*，探讨了如何为您的企业服务总线编程数据验证，包括如何从输入流中过滤消息。

第三章，*消息增强*，探讨了消息增强，这是企业服务总线的重要任务之一。消息增强是将额外信息纳入您流的消息的过程。

第四章，*序列化*，讨论了如何构建序列化和反序列化器，用于以二进制、原始字符串、JSON 或 AVRO 格式编写、读取或转换消息。

第五章，*模式注册表*，涵盖了如何使用 Kafka 模式注册表验证、序列化、反序列化和保留消息版本的历史记录。

第六章，*Kafka Streams*，解释了如何获取关于一组消息（换句话说，消息流）的信息，以及如何使用 Kafka Streams 获取其他信息，例如与消息的聚合和组合有关的信息。

第七章，*KSQL*，讨论了如何使用 SQL 在 Kafka Streams 上不写一行代码来操作事件流。

第八章，*Kafka Connect*，讨论了其他快速数据处理工具以及如何与 Apache Kafka 结合构建数据处理管道。本章涵盖了 Apache Spark 和 Apache Beam 等工具。

# 为了充分利用本书

读者应具备使用 Java 8 进行编程的一些经验。

执行本书中食谱所需的最小配置是一个 Intel ® Core i3 处理器、4 GB 的 RAM 和 128 GB 的磁盘空间。推荐使用 Linux 或 macOS，因为 Windows 不支持完全。

# 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的账户下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)上登录或注册。

1.  选择 SUPPORT 标签。

1.  点击代码下载与勘误。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载后，请确保使用最新版本解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Apache-Kafka-Quick-Start-Guide`](https://github.com/PacktPublishing/Apache-Kafka-Quick-Start-Guide)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富的图书和视频目录的代码包可供选择，请访问**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。查看它们！

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“`--topic`参数设置主题名称；在这种情况下，`amazingTopic`。”

代码块设置如下：

```java
{
   "event": "CUSTOMER_CONSULTS_ETHPRICE",
   "customer": {
         "id": "14862768",
         "name": "Snowden, Edward",
         "ipAddress": "95.31.18.111"
   },
   "currency": {
         "name": "ethereum",
         "price": "RUB"
   },
   "timestamp": "2018-09-28T09:09:09Z"
}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
dependencies {
    compile group: 'org.apache.kafka', name: 'kafka_2.12', version:                                                             
                                                          '2.0.0'
    compile group: 'com.maxmind.geoip', name: 'geoip-api', version: 
                                                          '1.3.1'
    compile group: 'com.fasterxml.jackson.core', name: 'jackson-core', version: '2.9.7'
}
```

任何命令行输入或输出都如下所示：

```java
> <confluent-path>/bin/kafka-topics.sh --list --ZooKeeper localhost:2181
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“为了区分它们，**t1**上的事件有一条条纹，**t2**上的事件有两条条纹，**t3**上的事件有三条条纹。”

警告或重要提示如下所示。

技巧和窍门如下所示。

# 联系我们

读者反馈始终欢迎。

**一般反馈**：如果您对本书的任何方面有疑问，请在邮件主题中提及书名，并通过`customercare@packtpub.com`将邮件发送给我们。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一点。请访问[www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式发现我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过`copyright@packt.com`与我们联系，并附上材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/).

# 评论

请留下评论。一旦您阅读并使用了这本书，为何不在您购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解 Packt 的更多信息，请访问[packt.com](http://www.packt.com/).
