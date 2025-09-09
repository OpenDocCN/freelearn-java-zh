# 前言

Java 已经存在多年，如果您把所有年份都加起来，应该得到一个数字——20 多年。这并不意味着 Java 已经过时或消亡；相反，Java 比以往任何时候都充满活力，新的 Java 8 语言规范就是证明。

由于 Java 并未消亡，WildFly 希望给您带来前所未有的更多功能。得益于其小巧的内存占用，WildFly 可以在 Raspberry Pi 上运行（口袋大小的设备），也可以通过 Docker（微服务架构的基础）在 Linux 容器中扩展，或者通过 OpenShift Online 平台进入云端。

此外，WildFly 的新模块化特性允许您根据需要自定义其系统。您可以通过提供自己的扩展和子系统来扩展 WildFly。

WildFly 的模块化类加载让您能够精细控制应用程序加载所需的库和 Java 类，这使您能够在持续集成和持续交付实践中正确使用 WildFly。

此外，WildFly 提供了一套管理 API，可用于管理整个平台。您可以通过命令行界面（CLI）与 WildFly 交互，这是一个强大的工具，用于管理整个系统。如果您更习惯于 UI，您可以使用精心设计的 Web 控制台。

选择 WildFly 的另一个原因是其充满活力的社区和整个 Java EE 环境。别忘了，WildFly 是唯一由其社区支持的开放源代码 Java EE 7 应用服务器！

# 本书涵盖的内容

第一章，*欢迎来到 WildFly！*，介绍了 WildFly Java 应用服务器及其与 Java EE 7 平台相关的核心特性。

第二章，*以独立模式运行 WildFly*，解释了独立操作模式以及您如何以这种方式管理实例。

第三章，*以域模式运行 WildFly*，解释了域操作模式及其包含的所有内容，例如域控制器和主机控制器。

第四章，*使用 CLI 管理日志子系统*，描述了如何配置和管理日志子系统以跟踪 WildFly 和应用程序的操作。

第五章，*使用 CLI 管理数据源子系统*，描述了如何配置和管理数据源子系统。

第六章，*在集群中运行 WildFly*，介绍了如何在两种操作模式下运行 WildFly，并解释了如何进行；展示了 TCP 和 UDP 网络配置。

第七章, *WildFly 的负载均衡*，涵盖了如何使用 Apache HTTP 服务器和 mod_cluster 平衡 WildFly 实例——使用 HTTP 和 AJP 协议。

第八章, *使用 CLI 进行命令操作*，解释了如何使用 CLI 检索配置和运行时信息；两种操作模式都被使用。

第九章, *征服 CLI*，讨论了如何使用 CLI 在两种操作模式下改变 WildFly 的状态，例如部署、取消部署、停止服务器、停止服务器组等。

第十章, *强化 WildFly 通信*，解释了您如何强化 WildFly 通信，例如 Web 控制台通过 HTTPS 在安全通道上进行通信、域控制器和主机控制器。

第十一章, *强化 WildFly 配置*，描述了强化 WildFly 配置的技术，例如哈希密码和使用保险库。

第十二章, *使用 WildFly 进行基于角色的访问控制*，介绍了 RBAC 提供者以访问 WildFly Web 控制台，并展示了如何对其进行自定义。

第十三章, *使用 WildFly 进行消息传递*，描述了您如何配置和管理消息子系统（嵌入式 HornetQ）及其组件，例如队列和主题。

第十四章, *使用 OpenShift 将 WildFly 带入云端*，介绍了 OpenShift Online 平台，以及您如何直接在云上部署 WildFly 应用程序。

第十五章, *使用 Docker 与 WildFly*，介绍了使用 Docker 的 Linux 容器，以及如何在上面运行 WildFly。

*附录，WildFly 域和独立模式*，是一个附加章节，带您了解 WildFly 的域和独立模式。您可以从[`www.packtpub.com/sites/default/files/downloads/2413OS_Appendix.pdf`](https://www.packtpub.com/sites/default/files/downloads/2413OS_Appendix.pdf)下载它。

# 您需要为这本书准备什么

要充分利用这本书，您首先需要一个 4GB RAM 的 PC，以及大约 50GB 的空闲磁盘空间。此外，互联网连接是必需的。

从软件角度来看，如果您想跟随这本书，您需要一个 Fedora 21 操作系统，以及 JDK 8 和 WildFly 9。

我们还将使用其他工具，例如 Maven、Git、Apache JMeter 和 MySQL。

# 本书面向的对象

本书旨在面向中间件系统管理员和 Java 开发者，实际上是对架构设计和实现有要求的优秀 Java 开发者。无论你是 WildFly 的初学者，还是来自之前的版本，如 JBoss AS 5、6 和 7，或者是对其有经验的专家，你都将能够掌握 WildFly 的基本和高级功能。

顺便说一句，WildFly 的核心组件大部分都是全新的，例如它的管理工具，即 CLI；它的操作模式，包括独立模式和域模式；以及它提供的由 Undertow 实现的 Web 服务器。即使你对 JBoss 和 WildFly 完全没有经验，你也可以从这本书中受益。

# 部分

在本书中，你会发现一些经常出现的标题（准备就绪、如何做、如何工作、还有更多、参见等）。

为了清楚地说明如何完成食谱，我们使用以下这些部分：

## 准备就绪

本节将告诉你可以在食谱中期待什么，并描述如何设置任何软件或任何为食谱所需的初步设置。

## 如何做…

本节包含遵循食谱所需的步骤。

## 如何工作…

本节通常包含对前一个章节发生情况的详细解释。

## 还有更多…

本节包含有关食谱的附加信息，以便使读者对食谱有更多的了解。

## 参见

本节提供了对其他有用信息的有帮助链接。

# 规范

在本书中，你会发现许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 处理方式如下所示：“`WFC` 文件夹仅用于不干扰你的当前环境。”

代码块设置如下：

```java
<VirtualHost 10.0.0.1:6666>
    <Directory />
      Order deny,allow
      Deny from all
      Allow from 10.0.0.1
    </Directory>
      ServerAdvertise off
      EnableMCPMReceive
</VirtualHost>
```

当我们希望引起你对命令行块中特定部分的注意时，相关的行或项目将以粗体显示：

```java
$ cd $WILDFLY_HOME
$ ./bin/standalone.sh -Djboss.bind.address=10.0.0.1
...
22:56:05,531 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-3) WFLYUT0006: Undertow HTTP listener default listening on /10.0.0.1:8080
```

任何命令行输入或输出都按如下方式编写：

```java
[disconnected /] connect
[standalone@localhost:9990 /] /socket-binding-group=standard-sockets/socket-binding=http:read-attribute(name=port)
{
    "outcome" => "success",
    "result" => expression "${jboss.http.port:8080}"
}
```

**新术语**和**重要词汇**以粗体显示。你在屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下：“你首先需要标记**接受许可协议**选项以启用链接。”

### 注意

警告或重要注意事项以如下方式显示。

### 小贴士

小贴士和技巧看起来像这样。

# 读者反馈

我们读者的反馈总是受欢迎的。告诉我们你对这本书的看法——你喜欢什么或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出你真正能从中获得最大收益的标题。

要向我们发送一般反馈，只需发送电子邮件至 `<feedback@packtpub.com>`，并在邮件主题中提及本书的标题。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误****提交****表**链接，并输入您的勘误详情来报告。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误表，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将在**勘误**部分显示。

## 侵权

互联网上版权材料的侵权是一个持续存在的问题，涉及所有媒体。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现任何形式的非法复制我们的作品，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供涉嫌侵权材料的链接。

我们感谢您在保护我们的作者以及为我们提供有价值内容的能力方面提供的帮助。

## 问题

如果您在这本书的任何方面遇到问题，您可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决问题。
