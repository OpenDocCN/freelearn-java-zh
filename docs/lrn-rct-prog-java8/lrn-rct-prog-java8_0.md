# 前言

响应式编程已经存在几十年了。自 Smalltalk 语言年轻时起，就有一些响应式编程的实现。然而，它最近才变得流行，并且现在成为一种趋势。你会问为什么现在？因为它适合编写快速、实时应用程序，当前的技术和 Web 需求如此。

我是在 2008 年参与其中的，当时我所在的团队正在开发一个名为 Sophie 2 的多媒体图书创建器。它必须快速响应，因此我们创建了一个名为 Prolib 的框架，它提供了可以相互依赖的对象属性（换句话说，我们为 Swing 实现了绑定等等）。将模型数据与 GUI 连接起来就像这样自然而然。

当然，这远非 RX 所具有的函数式方法。2010 年，微软发布了 RX，之后 Netflix 将其移植到 Java—RxJava。然而，Netflix 将 RxJava 发布给开源社区，该项目取得了巨大成功。许多其他语言都有其 RX 端口以及许多替代方案。现在，您可以在 Java 后端上使用响应式编程进行编码，并将其连接到 RxJava 的前端。

这本书试图向您解释响应式编程的全部内容以及如何在 RxJava 中使用它。它有许多小例子，并以小步骤解释概念和 API 细节。阅读本书后，您将对 RxJava、函数式编程和响应式范式有所了解。

# 本书涵盖的内容

第一章，响应式编程简介，将向您介绍响应式编程的概念，并告诉您为什么应该了解它。本章包含演示 RxJava 如何融合响应式编程概念的示例。

第二章，使用 Java 8 的函数式构造，将教您如何使用 Java 8 的新 lambda 构造。它将解释一些函数式编程概念，并向您展示如何在响应式程序中与 RxJava 一起使用它们。

第三章，创建和连接 Observables、Observers 和 Subjects，将向您展示 RxJava 库的基本构建模块，称为 Observables。您将学习“热”和“冷”Observables 之间的区别，以及如何使用订阅实例订阅和取消订阅它们。

第四章，转换、过滤和累积您的数据，将引导您了解基本的响应式操作符，您将学习如何使用它们来实现逐步计算。本章将让您了解如何转换 Observables 发出的事件，如何仅筛选出我们需要的数据，以及如何对其进行分组、累积和处理。

第五章，组合器、条件和错误处理，将向您介绍更复杂的响应式操作符，这将使您能够掌握可观察链。您将了解组合和条件操作符以及 Observables 如何相互交互。本章演示了不同的错误处理方法。

第六章，使用调度程序进行并发和并行处理，将指导您通过 RxJava 编写并发和并行程序的过程。这将通过 RxJava 调度程序来实现。将介绍调度程序的类型，您将了解何时以及为什么要使用每种调度程序。本章将向您介绍一种机制，向您展示如何避免和应用背压。

第七章，《测试您的 RxJava 应用程序》，将向您展示如何对 RxJava 应用程序进行单元测试。

第八章，《资源管理和扩展 RxJava》，将教您如何管理 RxJava 应用程序使用的数据源资源。我们将在这里编写自己的 Observable 操作符。

# 您需要为本书做好准备

为了运行示例，您需要：

+   已安装 Java 8，您可以从 Oracle 网站[`www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html`](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下载

+   Gradle 构建项目—2.x，您可以从[`gradle.org/downloads`](https://gradle.org/downloads)下载

+   Eclipse 打开项目。您还需要 Eclipse 的 Gradle 插件，可以从 Eclipse MarketPlace 下载。当然，您也可以使用命令行和 Vim 或任何其他任意文本编辑器查看代码。

# 本书适合对象

如果您是一名懂得如何编写软件并希望学习如何将现有技能应用于响应式编程的 Java 开发人员，那么这本书适合您。

这本书对任何人都有帮助，无论是初学者、高级程序员，甚至是专家。您不需要具有 Java 8 的 lambda 和 stream 或 RxJava 的任何经验。

# 约定

在本书中，您将找到一些区分不同信息类型的文本样式。以下是一些样式的示例及其含义的解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下："我们可以通过使用`include`指令包含其他上下文。"

代码块设置如下：

```java
Observable
  .just('R', 'x', 'J', 'a', 'v', 'a')
  .subscribe(
    System.out::print,
    System.err::println,
    System.out::println
  );
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```java
Observable<Object> obs = Observable
 .interval(40L, TimeUnit.MILLISECONDS)
 .switchMap(v ->
 Observable
 .timer(0L, 10L, TimeUnit.MILLISECONDS)
 .map(u -> "Observable <" + (v + 1) + "> : " + (v + u)))
 );
subscribePrint(obs, "switchMap");

```

**新术语**和**重要单词**以粗体显示。您在屏幕上看到的单词，例如菜单或对话框中的单词，会以这样的方式出现在文本中："这种类型的接口称为**函数接口**。"

### 注意

警告或重要说明会出现在这样的框中。

### 提示

提示和技巧会以这种方式出现。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对本书的看法—您喜欢或不喜欢的地方。读者的反馈对我们很重要，因为它有助于我们开发出您真正能从中获益的标题。

要向我们发送一般反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在主题中提及书名。

如果您在某个专题上有专业知识，并且有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的自豪所有者，我们有一些事情可以帮助您充分利用您的购买。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的帐户中为您购买的所有 Packt Publishing 图书下载示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接将文件发送到您的电子邮件。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误是难免的。如果您在我们的书中发现错误——可能是文本或代码中的错误——我们将不胜感激，如果您能向我们报告。通过这样做，您可以帮助其他读者避免挫折，并帮助我们改进本书的后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书，点击**勘误提交表**链接，并输入您的勘误详情。一旦您的勘误经过验证，您的提交将被接受，并且勘误将被上传到我们的网站或添加到该书籍的勘误列表中的勘误部分。

要查看先前提交的勘误，请转到[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需信息将出现在**勘误**部分下。

## 盗版

互联网上盗版受版权保护的材料是所有媒体的持续问题。在 Packt，我们非常重视版权和许可的保护。如果您在互联网上发现我们作品的任何非法副本，请立即向我们提供位置地址或网站名称，以便我们采取补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供涉嫌盗版材料的链接。

我们感谢您帮助保护我们的作者和我们为您提供有价值内容的能力。

## 问题

如果您对本书的任何方面有问题，可以通过`<questions@packtpub.com>`与我们联系，我们将尽力解决问题。
