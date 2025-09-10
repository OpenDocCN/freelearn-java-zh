# 前言

欢迎来到 *监控 Elasticsearch*！

有许多书籍和在线教程涵盖了 Elasticsearch API 和如何配置集群。但，直到现在，还没有一个全面、易于获取的资源用于监控和故障排除。我们发现，Elasticsearch 监控工具极大地提高了我们解决集群问题的能力，并极大地提高了集群的可靠性和性能。我们写这本书是为了分享这些用例及其产生的见解。

本书涵盖了如何使用几个流行的开源和商业 Elasticsearch 监控工具，包括 Elasticsearch-head、Bigdesk、Marvel、Kopf 和 Kibana。书中还包括了关于 Elasticsearch cat API 的部分以及如何使用 Nagios 进行通用系统监控。此外，我们还将讨论几个案例研究，通过这些工具解决 Elasticsearch 问题的实际示例。

我们认为，最好的学习方式是实践。在这本书中，我们将介绍如何设置一个示例 Elasticsearch 集群并加载数据。有时，我们会故意在集群中引入问题，以便我们可以看到如何使用我们的各种监控工具来追踪错误。在自己的集群中跟随这些示例将帮助您学习如何使用监控工具以及如何应对可能出现的新问题和未知问题。

在阅读本书后，我们希望您将更好地装备自己来运行和维护 Elasticsearch 集群。您也将更加准备好诊断和解决集群问题，例如节点故障、Elasticsearch 进程死亡、配置错误、分片错误、`OutOfMemoryError` 异常、慢查询和慢索引性能。

# 本书涵盖的内容

第一章, *Elasticsearch 监控简介*，概述了 Elasticsearch 并讨论了在监控集群或故障排除问题时需要注意的一些事项。

第二章, *Elasticsearch 的安装和需求*，涵盖了如何安装 Elasticsearch 和几个 Elasticsearch 监控工具。

第三章, *Elasticsearch-head 和 Bigdesk*，展示了如何配置一个多节点 Elasticsearch 集群以及如何使用监控工具 Elasticsearch-head 和 Bigdesk 来检查集群的健康状况和状态。

第四章, *Marvel 仪表板*，介绍了由 Elasticsearch 制造商创建的商业监控工具 Marvel。

第五章, *系统监控*，涵盖了 Elasticsearch 工具 Kopf、Kibana、Elasticsearch cat API 以及几个 Unix 命令行工具。本章还演示了如何使用 Nagios 进行通用系统监控。

第六章，*处理性能和可靠性问题*，涵盖了在使用 Elasticsearch 时如何处理一些常见的性能和可靠性问题。它还包含了一些带有真实世界故障排除案例研究的案例。

第七章，*节点故障和事后分析*，深入分析了你的集群的历史性能以及如何深入调查和从系统故障中恢复。它还包含了一些带有真实世界案例研究的案例。

第八章，*展望未来*，通过讨论 Elasticsearch 5 的未来以及一些即将发布的监控工具来结束本书。

# 你需要这本书的以下内容

要跟随这本书中的示例，你需要一个真实或虚拟化的三节点 Elasticsearch 集群。你可能还需要两个其他节点来运行 Marvel 和 Nagios，分别在 第四章，*Marvel 仪表板*和 第五章，*系统监控*中介绍。在 Elasticsearch 集群的节点上运行 Marvel 和 Nagios 是可能的，但在生产集群中你不应该这样做。查看 VMWare Player ([`www.vmware.com/products/player`](https://www.vmware.com/products/player)) 和 VirtualBox ([`www.virtualbox.org/wiki/Downloads`](https://www.virtualbox.org/wiki/Downloads)) 以建立自己的虚拟五节点环境，或使用 Amazon EC2 ([`aws.amazon.com/ec2/`](https://aws.amazon.com/ec2/)) 在云中构建集群。

对于你的 Elasticsearch 节点，你需要 64 位版本的 Windows、Mac OS X 或 Linux 以及 Java 运行时环境的最新版本。在这些主机上，CPU 速度并不那么重要，但我们建议每个节点至少有 512 MB 的内存。我们在这本书的所有示例中都使用 Ubuntu 14.04 和 Oracle Java 7，但任何现代操作系统以及 OpenJDK 或 Oracle Java 7 和 8 都可以用于运行示例。唯一的例外是 Nagios，它需要在 Linux 上运行。

你将需要以下软件包：

+   Java 7 或 Java 8 ([`www.oracle.com/technetwork/java/javase/downloads/index.html`](http://www.oracle.com/technetwork/java/javase/downloads/index.html))

+   Elasticsearch 2.3.2 ([`www.elastic.co/downloads/past-releases/elasticsearch-2-3-2`](https://www.elastic.co/downloads/past-releases/elasticsearch-2-3-2))

+   Elasticsearch-head ([`github.com/mobz/elasticsearch-head`](https://github.com/mobz/elasticsearch-head))

+   Bigdesk ([`bigdesk.org/`](http://bigdesk.org/))

+   Marvel—免费用于开发，生产使用需订阅费 ([`www.elastic.co/downloads/marvel`](https://www.elastic.co/downloads/marvel))

+   Kibana ([`www.elastic.co/downloads/kibana`](https://www.elastic.co/downloads/kibana))

+   Kopf ([`github.com/lmenezes/elasticsearch-kopf`](https://github.com/lmenezes/elasticsearch-kopf))

+   Nagios ([`www.nagios.org/downloads/`](https://www.nagios.org/downloads/))

所有这些软件包都是免费和开源的，除了 Marvel，它仅限于开发中使用免费。

最后，本书中的几个示例使用`curl` ([`curl.haxx.se/`](https://curl.haxx.se/)) 命令行工具对 Elasticsearch 进行 REST 调用，并且可选地使用 Python 2.7 来美化打印结果。

# 本书面向对象

这本书是为使用 Elasticsearch 的软件开发人员、DevOps 工程师和系统管理员而编写的。我们将介绍 Elasticsearch 的基础知识，以便安装和配置一个简单的集群，但我们不会深入探讨 Elasticsearch API。因此，对 Elasticsearch API 的基本理解可能有所帮助，尽管不是必需的，以便理解这本书。

# 约定

在这本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称将按以下方式显示：“现在我们将安装 Marvel 到`elasticsearch-marvel-01`。”

代码块设置如下：

```java
cluster.name: my_elasticsearch_cluster
node.name: "elasticsearch-node-01"
discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["elasticsearch-node-02", "elasticsearch-node-03"]
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
cluster.name: my_elasticsearch_cluster
node.name: "elasticsearch-node-01"
discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["elasticsearch-node-02","elasticsearch-node-03"]
```

任何命令行输入或输出都按以下方式编写：

```java
# sudo service elasticsearch start

```

**新术语**和**重要词汇**将以粗体显示。屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下：“点击**下一步**按钮将您带到下一个屏幕。”

### 注意

警告或重要注意事项将以如下方式显示。

### 小贴士

小技巧和技巧看起来像这样。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或不喜欢什么。读者反馈对我们来说非常重要，因为它帮助我们开发出您真正能从中获得最大收益的图书。

要向我们发送一般性反馈，请简单地发送电子邮件至 `<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果您在某个主题上具有专业知识，并且您有兴趣撰写或为图书做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了此书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**支持**标签上。

1.  点击**代码下载 & 错误清单**。

1.  在**搜索**框中输入书籍名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书的来源。

1.  点击**代码下载**。

您还可以通过点击 Packt Publishing 网站上书籍网页上的**代码文件**按钮来下载代码文件。您可以通过在**搜索**框中输入书籍名称来访问此页面。请注意，您需要登录到您的 Packt 账户。

文件下载完成后，请确保您使用最新版本的以下软件解压或提取文件夹：

+   Windows 上的 WinRAR / 7-Zip

+   Mac 上的 Zipeg / iZip / UnRarX

+   Linux 上的 7-Zip / PeaZip

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Monitoring-Elasticsearch`](https://github.com/PacktPublishing/Monitoring-Elasticsearch)。我们还有来自我们丰富的图书和视频目录的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

## 下载本书的彩色图像

我们还为您提供了一个包含本书中使用的截图/图表彩色图像的 PDF 文件。彩色图像将帮助您更好地理解输出中的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/MonitoringElasticsearch_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/MonitoringElasticsearch_ColorImages.pdf)下载此文件。

## 错误清单

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误清单，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**错误提交表单**链接，并输入您的错误详细信息来报告它们。一旦您的错误清单得到验证，您的提交将被接受，错误清单将被上传到我们的网站或添加到该标题的错误部分下的现有错误清单中。

要查看之前提交的勘误表，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将在**勘误**部分显示。

## 盗版

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何形式的非法副本，请立即提供位置地址或网站名称，以便我们可以追究补救措施。

请通过 `<copyright@packtpub.com>` 联系我们，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和我们提供有价值内容的能力方面所提供的帮助。

## 问题

如果您在这本书的任何方面遇到问题，您可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决问题。
