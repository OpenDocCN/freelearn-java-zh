# 前言

这是一本关于 Java EE 7 平台和 Web 数字开发的书籍，它是第一本书《Java EE 7 开发者手册》的延续。这本书的整个焦点是前端技术和业务逻辑层之间的空间和软件架构。虽然在我的第一本书中，这个主题在印刷空间和生活与时间平衡方面有所欠缺，但在本书《数字 Java EE 7》中，我们投入了大量的努力和决心，专门来写关于 Java *表现层*的内容。这本书是为那些希望成为在 JVM 上使用标准 Java EE 7 平台构建 Web 应用的优秀和熟练的开发者而写的。

这本书主要从 Java 标准的视角来介绍表现层。因此，有整整几章是专门介绍 JavaServer Faces 的，因为它是 Java EE 7 平台上最重要且最古老的专用 Web 框架。尽管这项技术自 2004 年以来就已经存在，但全球有商业组织和公司依赖 JSF。这些组织从蓝筹公司到备受尊敬的投资银行都有。然而，随着 Java EE 7 的发布，JSF 2.2 有几个关键特性，Web 开发者会非常喜欢并发现它们非常有帮助，例如 HTML5 友好的标记支持以及 Faces Flow。

作为读者，我希望能让你在构建能够让你攀登当代 Java Web 技术高峰的软件的道路上得到启发，并且你能在心中获得一位熟练的大师（或女大师）的资格。

因此，从 JSF 开始，我们将通过对其概念的全面介绍来学习这个框架。我们将继续构建 JSF 输入表单，并学习如何以多种方式验证它们的输入。对于 JSF Web 应用开发来说，最重要的任务是开发创建、检索、更新和删除（CRUD）功能，我们将直接解决这个问题。之后，我们将为 JSF 应用增添更多风格和精致。在这个过程中，我们将编写使用 AJAX 进行验证的应用程序，以实现即时效果。我们将继续我们的冒险，进入优雅的对话作用域后端控制器世界。我们会发现这些小巧的实用工具可以一起映射，并捕捉我们的利益相关者的客户旅程。最后，我们将学习关于 Faces Flows 的内容，这是 JSF 2.2 中的一个亮点。

没有一本 Java 网络技术书籍会不提及 JavaScript 编程语言和新兴技术。许多资深 Java 工程师会同意，Java 在 Web 上在一定程度上已经向 JavaScript 客户端框架让出了表现层。构建 REST/UI 前端应用程序现在如此普遍，以至于所谓的数字 Java 工程师很难忽视 jQuery、RequireJS 等的影响。在野外有几种已知的 JavaScript 框架。在这本书中，我们将介绍 AngularJS。我们将进入 Java、JVM 和 JavaScript 这两个主要景观之间的狂风暴雨的桥梁中间。我不能保证这不会让你感到害怕，但你可能会发现自己能够舒适地站在那里，在 JAX-RS 服务和 AngularJS 控制器之间谈判过梁和扶手。

在本书的远端，我们为您准备了一个特别的即时发布。我们专门用一整章的篇幅来介绍即将到来的 Java EE 8 模型-视图-控制器（Model-View-Controller），这可能会成为我们在构建未来 REST/UI 应用程序方式中的一个备选的炽热翡翠。在本书的终点线之外，我们还汇集了三个重要的附录，希望它们能作为优秀的参考资料。

在每一章的结尾，我们都专门设置了一个特殊部分用于教育练习，希望您觉得这些练习相关且合适，并在学习的过程中享受乐趣，同时您的思维过程也能得到方便的拓展。这是为您写的，为了创新而奋斗的 Java 网络开发者，祝您享受阅读！

您可以在[`www.xenonique.co.uk/blog/`](http://www.xenonique.co.uk/blog/)找到我的博客。您可以在 Twitter 上关注我，账号为`@peter_pilgrim`。

本书源代码可在 GitHub 上找到，链接为[`github.com/peterpilgrim/digital-javaee7`](https://github.com/peterpilgrim/digital-javaee7)。

# 本书涵盖内容

第一章，*数字 Java EE 7*，从网络技术的角度介绍了企业 Java 平台。我们将看到一个简短的 JSF 示例，研究 JavaScript 模块模式，并检查 Java EE 现代网络架构。

第二章，*JavaServer Faces 生命周期*，从 JSF 框架的基本元素开始。我们将了解 JSF 阶段和生命周期、自定义标签、常见属性和表达式语言。

第三章，*构建 JSF 表单*，将引导我们了解如何构建 JSF 的创建-更新-检索-删除应用程序表单。我们将使用 Bootstrap HTML5 框架和 JSF 自定义标签，以现代网络方法构建这些表单。

第四章，*JSF 验证和 AJAX*，深入探讨了从输入表单验证客户数据。我们将研究从后端和持久层到前端使用客户端 AJAX 检查数据的各种方法。

第五章，*对话与旅行*，将我们的 JSF 知识扩展到对话作用域的 Bean 中。我们将学习如何将数字客户的旅程映射到控制器，并将其他 CDI 作用域应用于我们的工作。

第六章，*优雅的 JSF 流程*，涵盖了 JSF 2.2 版本的关键亮点——流程作用域 Bean。我们将掌握 Faces 流程和对话作用域 Bean 之间的区别，并在过程中为我们的应用程序添加用户友好的功能。

第七章，*渐进式 JavaScript 框架和模块*，从 Java 工程师的角度提供了一个现代 JavaScript 编程的快速概述。我们将熟悉 jQuery 和其他相关框架，如 RequireJS 和 UnderscoreJS。

第八章，*AngularJS 和 Java RESTful 服务*，基于我们新的 JavaScript 知识。我们将使用流行的 AngularJS 框架来编写单页架构应用程序。我们还将获得编写 JAX-RS 服务端点的经验。

第九章，*Java EE MVC 框架*，探讨了即将推出的 Java EE 8 模型-视图-控制器框架的内部结构。我们将利用 Handlebars 模板框架在 Java 中的移植。

附录 A，*JSF 与 HTML5、资源与 JSF 流程*，提供了使用 HTML5 支持在 JSF、资源库合约和程序化 JSF 流程中的参考。它还包括有关使用消息和资源包进行国际化的重要信息。

附录 B，*从请求到响应*，提供了关于现代 Java 企业应用程序架构的密集参考材料。它回答了当收到 Web 请求时会发生什么，以及最终将响应发送回客户端的问题。

附录 C，*敏捷性能 – 数字团队内部工作*，涵盖了现代数字和敏捷软件开发团队中各种性格和角色的范围。

附录 D，*精选参考文献*，是一组特别精选的参考文献、资源和链接，用于进一步学习。

# 你需要这本书的什么

对于这本书，你需要在笔记本电脑或台式电脑上安装以下软件列表：

+   Java SE 8 (Java 开发工具包) [`java.oracle.com/`](http://java.oracle.com/)

+   GlassFish 4.1 ([`glassfish.java.net/`](https://glassfish.java.net/))

+   一个不错的 Java 编辑器或 IDE 用于编码，例如 IntelliJ 14 或更高版本 ([`www.jetbrains.com/idea/`](https://www.jetbrains.com/idea/))、Eclipse Kepler 或更高版本 ([`www.eclipse.org/kepler`](http://www.eclipse.org/kepler)/) 或 NetBeans 8.1 或更高版本 ([`netbeans.org/`](https://netbeans.org/)))

+   用于构建软件的 Gradle 2.6 或更高版本，这是本书的一部分 ([`gradle.org/`](http://gradle.org/))

+   带有开发者工具的 Chrome 网络浏览器 ([`developer.chrome.com/devtools`](https://developer.chrome.com/devtools))

+   Firefox 开发者工具 ([`developer.mozilla.org/en/docs/Tools`](https://developer.mozilla.org/en/docs/Tools))

+   Chris Pederick 的 Web 开发者和使用代理切换器扩展 ([`chrispederick.com/work/web-developer/`](http://chrispederick.com/work/web-developer/))

# 这本书面向的对象

你应该是一位熟练掌握编程语言的 Java 开发者。你应该已经了解类、继承和 Java 集合。因此，这本书是为中级 Java 开发者准备的。你可能拥有 1-2 年的 Java SE 核心开发经验。你应该对核心 Java EE 平台有所了解，尽管深入的知识不是严格必需的。你应该熟悉 Java 持久化、Java 服务器端组件和将 WAR 文件部署到应用服务器，如 GlassFish 或 WildFly 或等效服务器。

这本书的目标读者是那些想学习 JavaServer Faces 或更新现有知识的人。你可能或可能没有 JavaScript 编程经验；然而，本书中有一个专门的入门主题。这主要是一本 Java EE 网络开发书，但要涵盖 AngularJS，你需要学习或重新应用 JavaScript 编码技能。

无论你是来自数字环境，如代理机构或软件公司，还是刚刚开始考虑以网络开发为职业的专业工作，如果你需要在团队中与其他成员合作，你会发现这本书非常有帮助。你将看到行业术语，但我已经将它们的提及保持在最低限度，以便你可以专注于手头的科技并实现你的学习目标。然而，专家可能会在每一章末尾的问题中识别出某些行业想法的渗透。

# 术语约定

在这本书中，你会发现许多不同风格的文本，用于区分不同类型的信息。以下是一些这些风格的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称将如下所示：“我们可以通过使用`include`指令来包含其他上下文。”

代码块将如下设置：

```java
[default]
exten => s,1,Dial(Zap/1|30)
exten => s,2,Voicemail(u100)
exten => s,102,Voicemail(b100)
exten => i,1,Voicemail(s0)
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
[default]
exten => s,1,Dial(Zap/1|30)
exten => s,2,Voicemail(u100)
exten => s,102,Voicemail(b100)
exten => i,1,Voicemail(s0)
```

任何命令行输入或输出将如下所示：

```java
# cp /usr/src/asterisk-addons/configs/cdr_mysql.conf.sample
 /etc/asterisk/cdr_mysql.conf

```

**新术语**和**重要词汇**将以粗体显示。您在屏幕上看到的，例如在菜单或对话框中的单词，在文本中显示如下：“点击**下一个**按钮将您带到下一屏幕”。

### 注意

警告或重要注意事项将以如下所示的框显示。

### 小贴士

小技巧和技巧将如下所示。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者反馈对我们开发您真正能从中获得最大收益的标题非常重要。

要向我们发送一般反馈，只需发送电子邮件到`<feedback@packtpub.com>`，并在邮件主题中提及书籍标题。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您已经成为 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 错误

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)来报告它们，选择您的书籍，点击**错误提交表单**链接，并输入您的错误详情。一旦您的错误得到验证，您的提交将被接受，错误将被上传到我们的网站，或添加到该标题的“错误”部分下的现有错误列表中。任何现有错误都可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的标题来查看。

## 侵权

在互联网上，版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以追究补救措施。

请通过链接版权问题联系我们涉嫌盗版的内容。

我们感谢您在保护我们作者和提供有价值内容方面的帮助。

## 问题

如果你在本书的任何方面遇到问题，可以通过链接问题咨询我们，我们将尽力解决。
