# 序言

本书将帮助你了解 Eclipse MicroProfile，这是一个始于 2016 年的开源企业 Java 微服务规范，以及它的背景和历史、它对组织和企业的价值主张、它的社区治理、当前的 Eclipse MicroProfile 子项目（随着开源项目的演变，还会有更多的子项目加入）、它的实现以及它的互操作性。它还将为你提供 Eclipse MicroProfile 的未来发展方向的预览，一个在 Red Hat 的 Thorntail 实现中的 Eclipse MicroProfile 示例应用程序，Red Hat Runtimes 提供的一个运行时，以及关于在混合云和多云环境中运行 Eclipse MicroProfile 的指导和建议。本书将采用逐步的方法帮助你了解 Eclipse MicroProfile 项目和市场上的其实现。

# 本书适合哪些人

本书适合希望创建企业微服务的 Java 开发者。为了更好地利用这本书，你需要熟悉 Java EE 和微服务概念。

# 为了最大化地利用这本书

需要对微服务和企业 Java 有基本了解。其他安装和设置说明将在必要时提供。

# 下载示例代码文件

你可以从[www.packt.com](http://www.packt.com)上你的账户下载本书的示例代码文件。如果你在其他地方购买了这本书，你可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给你。

你可以通过以下步骤下载代码文件：

1.  登录或注册[www.packt.com](http://www.packt.com)。

1.  选择支持标签。

1.  点击代码下载。

1.  在搜索框中输入本书的名称，并按照屏幕上的指示操作。

文件下载后，请确保使用最新版本进行解压缩或提取文件夹：

+   Windows 上的 WinRAR/7-Zip

+   Mac 上的 Zipeg/iZip/UnRarX

+   Linux 上的 7-Zip/PeaZip

本书的代码包也托管在 GitHub 上，地址为：[`github.com/PacktPublishing/Hands-On-Enterprise-Java-Microservices-with-Eclipse-MicroProfile ...`](https://github.com/PacktPublishing/Hands-On-Enterprise-Java-Microservices-with-Eclipse-MicroProfile)

# 下载彩色图片

我们还将提供一个包含本书中使用的屏幕快照/图表的彩色图像的 PDF 文件。你可以在这里下载它：`static.packt-cdn.com/downloads/9781838643102_ColorImages.pdf`。

# 使用的约定

本书中使用了多种文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、假 URL、用户输入和 Twitter 处理。例如："`checks`数组对象类型包括一个必需的`name`和`status`字符串，以及一个可选的包含可选的`key`和`value`对的对象。"

代码块如下所示：

```java
package org.eclipse.microprofile.health;@FunctionalInterfacepublic interface HealthCheck {  HealthCheckResponse call();}
```

任何命令行输入或输出如下所示：

```java
Scotts-iMacPro:jwtprop starksm$ curl http://localhost:8080/jwt/secureHello; echoNot authorized
```

**粗体**：表示新的...

# 联系我们

来自读者的反馈总是受欢迎的。

**一般反馈**：如果你对本书的任何方面有问题，请在消息的主题中提及书名，并发送电子邮件至`customercare@packtpub.com`。

**错误报告**：尽管我们已经采取了每一步来确保我们内容的准确性，但错误仍然会发生。如果你在这本书中发现了错误，我们将非常感激你能向我们报告。请访问[www.packtpub.com/submit/errata](http://www.packtpub.com/submit/errata)，选择你的书，点击错误提交表单链接，并输入详情。

**盗版**：如果你在互联网上以任何形式发现我们作品的非法副本，如果你能提供位置地址或网站名称，我们将不胜感激。请通过`copyright@packt.com`联系我们，并附上材料的链接。

**如果您有兴趣成为作者**：如果您对某个主题有专业知识，并且有兴趣撰写或贡献一本书，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦你阅读并使用了这本书，为什么不在这本书购买的网站上留下评论呢？潜在的读者可以看到和使用你的客观意见来做出购买决定，我们 Packt 可以了解你对我们的产品的看法，我们的作者可以看到你对他们书的反馈。谢谢！

关于 Packt 的更多信息，请访问[packt.com](http://www.packt.com/)。
