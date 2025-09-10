# 前言

本书是使用 Bukkit API 编写 Minecraft 服务器插件的入门指南。Minecraft 是一个非常通用的沙盒游戏，玩家总是期待着用它做更多的事情。Bukkit 允许程序员做到这一点。本书面向可能没有编程背景的个人。它解释了如何设置 Bukkit 服务器并创建可以在服务器上运行的自己的自定义插件。它从 Bukkit 插件的基礎功能开始，如命令和权限，并继续深入到高级概念，如保存和加载数据。本书将帮助读者创建一个完整的 Bukkit 插件，无论他们是 Java 新手还是 Bukkit 新手。高级主题涵盖了 Bukkit API 的部分内容，甚至可以帮助当前插件开发者扩展他们的知识，以改进他们现有的插件。

# 本书涵盖的内容

第一章, *部署 Spigot 服务器*，指导读者如何下载和设置一个运行在 Spigot 上的 Minecraft 服务器，包括转发端口以允许其他玩家连接。在这一章中，还解释了常见的服务器设置和命令。

第二章, *学习 Bukkit API*，通过教授如何阅读其 API 文档来介绍 Bukkit。在这一章中，讨论了常见的 Java 数据类型和 Bukkit 类。

第三章, *创建您的第一个 Bukkit 插件*，指导读者安装 IDE 并创建一个简单的“Hello World”Bukkit 插件。

第四章, *在 Spigot 服务器上测试*，讨论了如何在 Spigot 服务器上安装插件以及简单的测试和调试技术。

第五章, *插件命令*，指导如何通过创建一个名为 Enchanter 的插件来编程用户命令到服务器插件中。

第六章, *玩家权限*，教授如何在插件中通过修改 Enchanter 来编程权限检查。这一章还指导读者安装一个名为 CodsPerms 的第三方插件。

第七章, *Bukkit 事件系统*，教授如何创建使用事件监听器的复杂 mod。这一章还通过创建两个新的插件，即 NoRain 和 MobEnhancer，帮助读者学习。

第八章，*使您的插件可配置*，通过扩展 MobEnhancer 教授读者程序配置。本章还解释了静态变量和类之间的通信。

第九章，*保存您的数据*，讨论了如何通过 YAML 文件配置保存和加载程序数据。本章还帮助创建了一个名为 Warper 的新插件。

第十章，*Bukkit 调度器*，在创建名为 AlwaysDay 的新插件时探讨了 Bukkit 调度器。在本章中，Warper 也被修改以包含计划编程。

# 您需要本书的内容

为了从本书中受益，您需要一个 Minecraft 账户。Minecraft 游戏客户端可以免费下载，但必须在[`minecraft.net/`](https://minecraft.net/)购买账户。本书中使用的其他软件包括 Spigot 服务器.jar（这与正常的 Minecraft 服务器.jar 不同）和一个 IDE，例如 NetBeans 或 Eclipse。本书将指导您在 Windows PC 上下载和安装服务器和 IDE 的过程。

# 本书面向对象

本书非常适合对自定义 Minecraft 服务器感兴趣的人。即使您可能对编程、Java、Bukkit 或甚至 Minecraft 本身都是新手，本书也能为您提供帮助。您所需的一切就是一个有效的 Minecraft 账户。本书是软件开发的优秀入门书籍。如果您对 Java 或编写软件没有任何先前的知识，您可以在[`codisimus.com/learnjava`](http://codisimus.com/learnjava)上找到一些入门教学，为阅读本书的章节做好准备。如果您对编程作为职业或爱好感兴趣，本书将帮助您入门。如果您只是想和朋友们一起玩 Minecraft，那么这本书将帮助您使这种体验更加愉快。

# 习惯用法

在本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称如下所示：`<java bin path>` 应替换为 Java 安装的位置。Java 路径取决于您电脑上 Java 的版本。

代码块应如下设置：

```java
"<java bin path>\java.exe" -jar BuildTools.jar
```

任何命令行输入或输出都应如下所示：

```java
>op <player>

```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下：“打开**文件**菜单并点击**新建项目...**。”

### 注意

警告或重要提示会以这样的框中出现。

### 小贴士

小技巧和窍门看起来像这样。

# 读者反馈

我们欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们来说很重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。

要发送一般反馈，请简单地发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中提及书籍的标题。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您已经成为 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户中下载示例代码文件，适用于您购买的所有 Packt 出版社的书籍。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 下载本书的颜色图像

我们还为您提供了一个包含本书中使用的截图/图表的颜色图像的 PDF 文件。这些颜色图像将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/BuildingMinecraftServerModificationsServerSecondEdition_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/BuildingMinecraftServerModificationsServerSecondEdition_ColorImages.pdf)下载此文件。

## 错误更正

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误更正，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**错误更正提交表单**链接，并输入您的错误更正详情来报告。一旦您的错误更正得到验证，您的提交将被接受，错误更正将被上传到我们的网站或添加到该标题的错误更正部分下的现有错误更正列表中。

要查看之前提交的错误更正，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍的名称。所需信息将出现在**错误更正**部分下。

## 侵权

在互联网上侵犯版权材料是一个跨所有媒体的持续问题。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过链接将涉嫌盗版材料发送至 `<copyright@packtpub.com>`。

我们感谢您在保护我们的作者和我们为您提供有价值内容的能力方面的帮助。

## 问题

如果您对本书的任何方面有问题，您可以联系 `<questions@packtpub.com>`，我们将尽力解决问题。
