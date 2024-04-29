# 前言

# 本书涵盖内容

本书深入探讨了 lambda 及其支持功能的特点；例如函数接口和类型推断。

阅读完本书后，您将：

+   了解现代 Java 中的新功能概述

+   深入了解 lambda，它们的背景、语法、实现细节以及如何以及何时使用它们

+   理解函数与类之间的区别，以及这对 lambda 的相关性

+   理解 lambda 和闭包之间的区别

+   欣赏推动许多新功能的类型推断的改进

+   能够使用方法引用并理解作用域和“有效最终”

+   理解使用 lambda 时产生的字节码的差异

+   能够推理异常和异常处理最佳实践在使用 lambda 时

# 您需要为本书做好准备

最新版本的 JDK 和文本编辑器或 IDE。

# 这本书是为谁准备的

无论您是将传统的 Java 程序迁移到更现代的 Java 风格，还是从头开始构建应用程序，本书都将帮助您开始利用 Java 平台上的函数式编程的力量。

# 约定

在本书中，您将找到一些区分不同信息类型的文本样式。以下是这些样式的一些示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下：“这个 LISP 表达式评估为一个函数，当应用时将接受一个参数，将其绑定到`arg`，然后将`1`添加到它。”

代码块设置如下：

```java
void anonymousClass() {
    final Server server = new HttpServer();
    waitFor(new Condition() {
        @Override
        public Boolean isSatisfied() {
            return !server.isRunning();
        }
    });
}
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```java
void anonymousClass() {
    final Server server = new HttpServer();
    waitFor(new Condition() {
        @Override
        public Boolean isSatisfied() {
            return !server.isRunning();
        }
    });
}
```

**新术语**和**重要单词**以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，会以这样的方式出现在文本中：“为了下载新模块，我们将转到**文件** | **设置** | **项目名称** | **项目解释器**。”

### 注意

警告或重要说明会以这样的方式出现在一个框中。

### 提示

提示和技巧会以这种方式出现。

# 读者反馈

我们始终欢迎读者的反馈。让我们知道您对本书的看法-您喜欢或不喜欢的地方。读者的反馈对我们很重要，因为它可以帮助我们开发出您真正受益的标题。要向我们发送一般反馈，只需发送电子邮件至 feedback@packtpub.com，并在主题中提及书名。如果您在某个专题上有专业知识，并且有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的自豪所有者，我们有很多事情可以帮助您充分利用您的购买。

## 下载本书的彩色图像

我们还为您提供了一份 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。彩色图像将帮助您更好地理解输出中的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/LearningJavaLambdas_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/LearningJavaLambdas_ColorImages.pdf)下载此文件。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误确实会发生。如果您在我们的书籍中发现错误-也许是文本或代码中的错误-如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以帮助其他读者避免挫折，并帮助我们改进本书的后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表格**链接，并输入您的勘误详情。一旦您的勘误被验证，您的提交将被接受，并且勘误将被上传到我们的网站或添加到该标题的勘误部分下的任何现有勘误列表中。

要查看先前提交的勘误表，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support) 并在搜索字段中输入书名。所需信息将出现在**勘误表**部分下。

## 盗版

互联网上盗版受版权保护的材料是所有媒体的持续问题。在 Packt，我们非常重视版权和许可的保护。如果您在互联网上发现我们作品的任何非法副本，请立即向我们提供位置地址或网站名称，以便我们采取补救措施。

请通过链接到 suspected pirated material 发送电子邮件至 copyright@packtpub.com 与我们联系。

我们感谢您在保护我们的作者和为您带来有价值内容的能力方面的帮助。

## 问题

如果您对本书的任何方面有问题，可以通过 questions@packtpub.com 与我们联系，我们将尽力解决问题。
