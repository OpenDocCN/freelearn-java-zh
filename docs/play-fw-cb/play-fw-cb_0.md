# 前言

在过去 5-10 年中，Web 应用已经取得了长足的进步。从 Geocities 和 Friendster 的辉煌时代，到社交媒体网站（如 Facebook 和 Twitter）的兴起，再到更实用的软件即服务（SaaS）应用（如 Salesforce 和 Github），不可否认的是，随着消费者和企业级 Web 软件的这些进步，出现了对构建在稳固的 Web 技术平台之上的需求，这不仅适用于最终用户 Web 客户端，还适用于各种复杂和高级的后端服务，所有这些都可以组成现代 Web 应用。

这就是 Play Framework 2.0 登场的地方。Play 为开发者提供了一个强大且成熟的轻量级无状态网络开发平台，其设计初衷就是考虑开发速度和 Web 应用的扩展性。

本书旨在通过基于非常常见用例和场景的简洁代码示例，让读者对 Play Framework 的不同部分有更深入的理解。您将了解 Play 中使用的基本概念和抽象，我们将深入探讨更高级和相关的主题，例如创建 RESTful API、使用第三方云存储服务存储上传的图片、向外部消息队列发送消息，以及使用 Docker 部署 Play 网络应用。

通过提供相关的食谱，本书希望为开发者提供创建下一个 Facebook 或 Salesforce 所需的必要构建块，使用 Play Framework。

# 本书涵盖的内容

第一章 *Play Framework 基础* 介绍了 Play Framework 及其功能。本章还介绍了 Play 的基本组件，例如控制器和视图模板。最后，我们讨论了如何对 Play Framework 的模型和控制器类进行单元测试。

第二章 *使用控制器* 深入讨论了 Play 控制器。本章演示了如何使用控制器与其他 Web 组件（如请求过滤器、会话和 JSON）一起使用。它还涉及了如何从控制器层面利用 Play 和 Akka。

第三章 *利用模块* 探讨了利用官方 Play 2 模块以及其他第三方模块。这应该有助于开发者通过重用和集成现有模块来加快他们的开发速度。

第四章 *创建和使用 Web API* 讨论了如何使用 Play 创建安全的 RESTful API 端点。本章还讨论了如何使用 Play WS 库消费其他基于 Web 的 API。

第五章 *创建插件和模块* 讨论了如何编写 Play 模块和插件，并告诉我们如何将 Play 模块发布到 Amazon S3 上的私有仓库。

第六章 *实用模块示例* 在前一章关于 Play 模块的基础上，讨论了更多关于集成模块（如消息队列和搜索服务）的实用示例。

第七章，*部署 Play 2 网络应用*，讨论了使用 Docker 和 Dokku 等工具在不同环境中部署 Play 网络应用的部署场景。

第八章，*附加游戏信息*，讨论了与开发者相关的话题，例如与 IDE 集成和使用其他第三方云服务。本章还讨论了如何使用 Vagrant 从零开始构建 Play 开发环境。

# 本书所需

您需要以下软件来使用本书中的食谱：

+   Java 开发工具包 1.7

+   Typesafe Activator

+   Mac OS X 终端

+   Cygwin

+   Homebrew

+   curl

+   MongoDB

+   Redis

+   boot2docker

+   Vagrant

+   Intellij IDEA

# 本书面向的对象

本书旨在帮助高级开发者利用 Play 2.x 的力量。本书对希望深入了解网络开发的专业人士也很有用。Play 2.x 是一个优秀的框架，可以加速您对高级主题的学习。

# 部分

在本书中，您会发现几个频繁出现的标题（准备就绪、如何操作、它是如何工作的、还有更多、相关内容）。

为了清楚地说明如何完成一个食谱，我们使用以下部分如下：

## 准备就绪

本节告诉您在食谱中可以期待什么，并描述了如何设置任何软件或任何为食谱所需的初步设置。

## 如何操作…

本节包含遵循食谱所需的步骤。

## 它是如何工作的…

本节通常包含对上一节发生情况的详细解释。

## 还有更多…

本节包含有关食谱的附加信息，以便让读者对食谱有更深入的了解。

## 相关内容

本节提供了对其他有用信息的链接，这些信息对食谱很有帮助。

# 惯例

在本书中，您会发现许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 处理方式如下所示：“通过在 `app/Global.scala` 文件中声明来使用这个新的过滤器。”

代码块按以下方式设置：

```java
// Java
    return Promise.wrap(ask(fileReaderActor, words, 3000)).map(
      new Function&lt;Object, Result&gt;() {
        public Result apply(Object response) {
          return ok(response.toString());
        }
      }
    );
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
<span class="strong"><strong>GET   /dashboard  controllers.Application.dashboard</strong></span>
GET   /login    controllers.Application.login
```

任何命令行输入或输出都按以下方式编写：

```java
<span class="strong"><strong>$ curl -v http://localhost:9000/admins</strong></span>
```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中如下所示：“再次使用相同的网络浏览器访问这个新的 URL 路由，您会看到文本 **Found userPref: tw**。”

### 注意

警告或重要注意事项以如下框的形式出现。

### 小贴士

小技巧和技巧如下所示。

# 读者反馈

读者反馈始终受到欢迎。让我们知道您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们来说很重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。

要发送给我们一般反馈，只需发送电子邮件至`&lt;<a class="email" href="mailto:feedback@packtpub.com">feedback@packtpub.com</a>&gt;`，并在您的邮件主题中提及书籍的标题。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的骄傲拥有者，我们有多个方面可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)下载您购买的所有 Packt 出版物的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 下载本书的颜色图像

我们还为您提供了一个包含本书中使用的截图/图表的颜色图像的 PDF 文件。颜色图像将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/1234OT_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/1234OT_ColorImages.pdf)下载此文件。

## 勘误

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然会发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍的名称。所需信息将出现在**勘误**部分下。

## 盗版

互联网上对版权材料的盗版是一个横跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供给我们地址或网站名称，以便我们可以寻求补救措施。

请通过链接发送至`&lt;<a class="email" href="mailto:copyright@packtpub.com">copyright@packtpub.com</a>&gt;`与我们联系，以提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面的帮助。

## 问题和建议

如果您对本书的任何方面有问题，您可以通过`&lt;<a class="email" href="mailto:questions@packtpub.com">questions@packtpub.com</a>&gt;`与我们联系，我们将尽力解决问题。
