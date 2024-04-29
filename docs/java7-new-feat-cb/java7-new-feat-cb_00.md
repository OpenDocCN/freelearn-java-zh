# 前言

随着 Java 7 的发布，许多新功能已被添加，显著提高了开发人员创建和维护 Java 应用程序的能力。这些包括语言改进，如更好的异常处理技术，以及 Java 核心库的添加，如新的线程机制。

本书使用一系列配方涵盖了这些新功能。每个配方都涉及一个或多个新功能，并提供了使用这些功能的模板。这应该使读者更容易理解这些功能以及何时以及如何使用它们。提供了逐步说明，以指导读者完成配方，然后解释生成的代码。

本书以讨论新语言增强开始，然后是一系列章节，每个章节都涉及特定领域，如文件和目录管理。假定读者熟悉 Java 6 的功能。本书不需要按顺序阅读，这使读者可以选择感兴趣的章节和配方。但建议读者阅读第一章，因为后续配方中会使用那里的许多功能。如果在配方中使用了其他新的 Java 7 功能，则提供了相关配方的交叉引用。

# 本书涵盖的内容

第一章, *Java 语言改进:* 本章讨论了作为 Coin 项目的一部分引入的各种语言改进。这些功能包括简单的改进，如在文字中使用下划线和在 switch 语句中使用字符串。还有更重要的改进，如 try-with-resources 块和引入的菱形操作符。

第二章, *使用路径定位文件和目录:* 本章介绍了 Path 类。它在本章和其他章节中被使用，并且是 Java 7 中许多新的与文件相关的添加的基础。

第三章, *获取文件和目录信息:* 许多应用程序需要访问特定的文件和目录信息。本章介绍了如何访问文件信息，包括访问基本文件属性、Posix 属性和文件的访问控制列表等信息。

第四章, *管理文件和目录:* 本章涵盖了管理文件和目录的基本机制，包括创建和删除文件等操作。还讨论了临时文件的使用和符号链接的管理。

第五章, *管理文件系统:* 这里介绍了许多有趣的主题，如如何获取文件系统和文件存储信息、用于遍历文件结构的类、如何监视文件和目录事件以及如何使用 ZIP 文件系统。

第六章, *Java 7 中的流 IO:* 引入了 NIO2。详细介绍了执行异步 IO 的新技术，以及执行随机访问 IO 和使用安全目录流的新方法。

第七章, *图形用户界面改进:* Java 7 中增加了几项功能，以解决创建 GUI 界面的问题。现在可以创建不同形状的窗口和透明窗口。此外，还解释了许多增强功能，如使用 JLayer 装饰器，它改善了在窗口上叠加图形的能力。

第八章，*事件处理：* 在本章中，我们将研究处理各种应用程序事件的新方法。Java 7 现在支持额外的鼠标按钮和精确的鼠标滚轮。改进了控制窗口焦点的能力，并引入了辅助循环来模拟模态对话框的行为。

第九章，*数据库、安全和系统增强：* 说明了各种数据库改进，例如引入新的 RowSetFactory 类以及如何利用新的 SSL 支持。此外，还演示了其他系统改进，例如对 MXBeans 的额外支持。

第十章，*并发处理：* 添加了几个新类来支持线程的使用，包括支持 fork/join 范式、phaser 模型、改进的 dequeue 类和 transfer queue 类的类。解释了用于生成随机数的新 ThreadLocalRandom 类。

第十一章，*杂项：* 本章演示了许多其他 Java 7 改进，例如对周、年和货币的新支持。本章还包括了对处理空引用的改进支持。

# 您需要为这本书做什么

本书所需的软件包括 Java 开发工具包（JDK）1.7 或更高版本。任何支持 Java 7 的集成开发环境都可以用于创建和执行示例。本书中的示例是使用 NetBeans 7.0.1 开发的。

# 这本书适合谁

本书旨在让熟悉 Java 的人了解 Java 7 中的新功能。

# 约定

在本书中，您会发现一些文本样式，用于区分不同类型的信息。以下是一些这些样式的示例，以及它们的含义解释。

文本中的代码单词显示如下：“我们可以通过使用`include`指令包含其他上下文。”

代码块设置如下：

```java
private void gameEngine(List<Entity> entities)
{
final Phaser phaser = new Phaser(1);
for (final Entity entity : entities)
{
final String member = entity.toString();
System.out.println(member + " joined the game");
phaser.register();
new Thread()
{
@Override
public void run()
{
System.out.println(member +
" waiting for the remaining participants");
phaser.arriveAndAwaitAdvance(); // wait for remaining entities
System.out.println(member + " starting run");
entity.run();
}
}.start();
}
phaser.arriveAndDeregister(); //Deregister and continue
System.out.println("Phaser continuing");
}
```

当我们希望引起您对代码块的特定部分的注意时，相关的行或项目会以粗体显示：

```java
private void gameEngine(List<Entity> entities)
{
final Phaser phaser = new Phaser(1);
for (final Entity entity : entities)
{
final String member = entity.toString();
System.out.println(member + " joined the game");
phaser.register();
new Thread()
{
@Override
public void run()
{
System.out.println(member +
" waiting for the remaining participants");
phaser.arriveAndAwaitAdvance(); // wait for remaining entities
System.out.println(member + " starting run");
entity.run();
}
}.start();
}
phaser.arriveAndDeregister(); //Deregister and continue
System.out.println("Phaser continuing");
}

```

任何命令行输入或输出都是这样写的：

```java
Paths.get(new URI("file:///C:/home/docs/users.txt")), Charset.defaultCharset()))

```

新术语和重要单词以粗体显示。您在屏幕上看到的单词，例如菜单或对话框中的单词，会在文本中以这种方式出现：“单击**下一步**按钮将您移动到下一个屏幕”。

### 注意

警告或重要说明会以这样的方式出现在框中。

### 提示

提示和技巧看起来像这样。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对本书的看法——您喜欢或不喜欢的地方。读者的反馈对我们开发您真正受益的标题非常重要。

要向我们发送一般反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在消息主题中提及书名。

如果您在某个主题上有专业知识，并且有兴趣撰写或为一本书做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 图书的自豪所有者，我们有一些事情可以帮助您充分利用您的购买。

# 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载您购买的所有 Packt 图书的示例代码文件。如果您在其他地方购买了本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接通过电子邮件接收文件。

## 勘误

尽管我们已经尽最大努力确保内容的准确性，但错误是难免的。如果您在我们的书籍中发现错误，无论是文字还是代码方面的错误，我们将不胜感激地接受您的报告。通过这样做，您可以帮助其他读者避免挫折，并帮助我们改进本书的后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，选择您的书籍，点击**勘误提交表**链接，并输入您的勘误详情。一旦您的勘误经过验证，您的提交将被接受，并且勘误将被上传到我们的网站，或者添加到该书籍的勘误列表中的“勘误”部分。

## 盗版

互联网上的侵犯版权行为是各种媒体持续存在的问题。在 Packt，我们非常重视版权和许可的保护。如果您在互联网上发现我们作品的任何非法副本，请立即向我们提供位置地址或网站名称，以便我们采取补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供涉嫌盗版材料的链接。

我们感谢您帮助保护我们的作者，以及我们为您提供有价值内容的能力。

## 问题

如果您在阅读本书的过程中遇到任何问题，请通过`<questions@packtpub.com>`与我们联系，我们将尽力解决。
