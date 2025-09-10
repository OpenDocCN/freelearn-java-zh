# 前言

Maven 是开发者使用的首选构建工具，它已经存在十多年了。Maven 因其基于约定优于配置概念的极其可扩展的架构而脱颖而出，这实际上使 Maven 成为管理和构建 Java 项目的默认工具。它被许多开源 Java 项目广泛使用，包括 Apache 软件基金会、SourceForge、Google Code 等。

本书提供了一步一步的指南，展示了读者如何以最佳方式使用 Apache Maven 来满足企业构建需求。遵循本书，读者将能够深入理解以下关键领域：

+   如何开始使用 Apache Maven，应用 Maven 最佳实践以设计构建系统以提高开发者的生产力

+   如何通过使用适当的 Maven 插件、生命周期和原型来定制构建过程，以完全满足你企业的需求

+   如何更有信心地解决构建问题

+   如何设计构建方式，以避免因适当的依赖管理而产生任何维护噩梦

+   如何优化 Maven 配置设置

+   如何使用 Maven 组装构建自己的分发存档

# 本书涵盖的内容

第一章，*Apache Maven 快速入门*，专注于围绕 Maven 构建基本基础以开始使用。它首先解释了在 Ubuntu、Mac OS X 和 Microsoft Windows 操作系统上安装和配置 Maven 的基本步骤。本章的后半部分涵盖了某些常见的有用 Maven 技巧和窍门。

第二章，*理解项目对象模型（POM）*，专注于 Maven 项目对象模型（POM）以及如何遵守行业广泛接受的最佳实践以避免维护噩梦。POM 文件的关键元素、POM 层次结构和继承、管理依赖项以及相关主题在此处得到涵盖。

第三章，*Maven 原型*，专注于 Maven 原型如何提供一种减少构建 Maven 项目重复工作的方法。有数千个原型可供公开使用，帮助你构建不同类型的项目。本章涵盖了一组常用的原型。

第四章，*Maven 插件*，涵盖了最常用的 Maven 插件，并解释了插件是如何被发现和执行的。Maven 只提供构建框架，而 Maven 插件执行实际的任务。Maven 有一套庞大、丰富的插件，你几乎不需要编写自己的自定义插件。

第五章，*构建生命周期*，解释了三个标准生命周期是如何工作的，以及我们如何自定义它们。在章节的后面部分，我们讨论了如何开发我们自己的生命周期扩展。

第六章, *Maven 组件*，详细介绍了如何使用 Maven 组件插件的实际案例，并最终以一个端到端的 Maven 项目示例结束。

第七章，*最佳实践*，探讨了在大型 Maven 开发项目中应遵循的一些最佳实践。始终建议遵循最佳实践，因为它将极大地提高开发者的生产力，并减少任何维护噩梦。

# 您需要为此书准备的

要跟随本书中的示例，您需要以下软件：

+   Apache Maven 3.3.x，您可以在[`maven.apache.org/download.cgi`](http://maven.apache.org/download.cgi)找到。

+   Java 1.7+ SDK，您可以在[`www.oracle.com/technetwork/java/javase/downloads/index.html`](http://www.oracle.com/technetwork/java/javase/downloads/index.html)找到。

+   操作系统：Windows、Linux 或 Mac OS X。

# 本书面向的对象

本书非常适合已经熟悉构建自动化但想学习如何使用 Maven 并将其概念应用于构建自动化中最困难场景的资深开发者。

# 约定

在本书中，您将找到许多不同的文本样式，以区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称如下所示："当您输入`mvn clean install`时，Maven 将执行默认生命周期中的所有阶段直到`install`阶段（包括`install`阶段）。"

代码块如下设置：

```java
<project>
  [...]
  <build>
    [...]
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.1</version>
        <configuration>
          <source>1.7</source>
          <target>1.7</target>
        </configuration>
      </plugin>
    </plugins>
    [...]
  </build>
  [...]
</project>
```

当我们希望引起您对代码块中特定部分的注意时，相关的行或项目将以粗体显示：

```java
<project>
  [...]
  <build>
    [...]
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.1</version>
        <configuration>
 <source>1.7</source>
 <target>1.7</target>
 </configuration>
      </plugin>
    </plugins>
    [...]
  </build>
  [...]
</project>
```

任何命令行输入或输出都应如下编写：

```java
$ mvn install:install

```

**新术语**和**重要词汇**以粗体显示。

### 注意

警告或重要注意事项以如下框中的形式出现。

### 提示

技巧和窍门如下所示。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们您对本书的看法——您喜欢或不喜欢的地方。读者反馈对我们来说很重要，因为它帮助我们开发出您真正能从中受益的标题。

要向我们发送一般反馈，请简单地发送电子邮件至 `<feedback@packtpub.com>`，并在邮件主题中提及本书的标题。

如果您在某个主题上具有专业知识，并且您对撰写或为本书做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 图书的骄傲拥有者，我们有多种方式帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户下载示例代码文件，适用于您购买的所有 Packt 出版图书。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 勘误

尽管我们已经尽最大努力确保内容的准确性，错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者感到沮丧，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**勘误**部分下。

## 侵权

在互联网上，版权材料侵权是一个持续存在的问题，涉及所有媒体。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现任何形式的非法复制我们的作品，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过发送链接到疑似盗版材料的方式与我们联系 `<copyright@packtpub.com>`。

我们感谢您在保护我们的作者和我们为您提供有价值内容的能力方面的帮助。

## 问题

如果您对本书的任何方面有问题，您可以通过发送电子邮件到 `<questions@packtpub.com>` 来联系我们，我们将尽力解决问题。
