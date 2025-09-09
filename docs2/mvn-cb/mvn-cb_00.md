# 前言

Apache Maven 食谱集通过一系列食谱描述了 Apache Maven 的功能。这本书将帮助您了解 Apache Maven 是什么，并允许您通过完整且可工作的示例来使用其功能。

# 本书涵盖内容

第一章, *入门*，介绍了在 Microsoft Windows、Mac OS X 或 Linux 上安装 Apache Maven，以及如何使用它创建和构建您的第一个项目。本章还详细说明了安装 Maven 所需的前提软件的步骤。

第二章, *Maven 与 IDE 集成*，专注于使用 Maven 配置流行的 IDE，并在其中运行 Maven 项目。本章涵盖了 Eclipse、NetBeans 和 IntelliJ IDEA 这三个 IDE。

第三章, *Maven 生命周期*，涵盖了 Apache Maven 的生命周期，并探讨了阶段和目标的概念。还描述了用户如何使用配置文件来自定义构建。

第四章, *基本 Maven 插件*，描述了构建项目所必需的 Maven 插件。对于每个插件，还探讨了各种配置选项。

第五章, *依赖管理*，探讨了 Maven 依赖的各种类型，并深入探讨了下载和获取它们的报告。还讨论了在依赖项下载过程中如何处理网络问题。

第六章, *代码质量插件*，涵盖了各种代码质量工具（如 Checkstyle、PMD、FindBugs 和 Sonar）的支持。还探讨了每个插件的配置选项以及生成报告。

第七章, *报告和文档*，涵盖了 Maven 的报告功能。详细描述了 site 插件及其支持的各项报告。

第八章, *处理典型构建需求*，探讨了 Maven 提供的处理选择性源和包含选定资源的构建的功能。还描述了如何使用 Maven 的命令行和帮助功能，以及与软件配置管理系统的接口。

第九章, *多模块项目*，描述了构建具有多个模块的大型项目所需的支持。这里还描述了 Maven 对聚合构建和定义父子关系的支持。

第十章, *使用 Maven 进行 Java 开发*，描述了构建不同类型的 Java 工件（如 Jar、War 和 Ear）的过程。还描述了 Maven 对在 Jetty 和 Tomcat 中运行项目的支持。

第十一章, *高级 Maven 使用*，探讨了 Maven 的高级功能，如创建发行版和强制执行规则。它还描述了如何制作项目发布。

# 您需要这本书什么

要运行本书中的各种食谱，需要以下内容。除非另有说明，最好使用此处建议的软件的最新版本：

+   拥有三种操作系统之一（如 Microsoft Windows、Mac OS X 或 Linux）的计算机，最好是最新/受支持的版本

+   Java——具体来说是 Java 开发工具包（JDK）

+   Apache Maven

+   Git——有关版本控制系统的示例

+   以下一个或多个 IDE：

    +   Eclipse

    +   NetBeans

    +   IntelliJ IDEA

# 这本书面向谁

Apache Maven 食谱旨在帮助那些寻求了解构建自动化是什么以及如何使用 Apache Maven 实现此目的的人。如果您熟悉 Maven，但想了解其更细微的细微差别以解决特定问题，本书也适合您。如果您正在寻找现成的食谱来解决特定用例，本书也是一本好书。

# 部分

在本书中，您将找到几个频繁出现的标题（准备工作、如何做、它是如何工作的、还有更多、另请参阅）。

为了清楚地说明如何完成食谱，我们使用以下部分如下：

## 准备工作

本节告诉您在食谱中可以期待什么，并描述了如何设置任何软件或任何为食谱所需的初步设置。

## 如何做……

本节包含遵循食谱所需的步骤。

## 它是如何工作的……

本节通常包含对前一个章节发生情况的详细解释。

## 还有更多……

本节包含有关食谱的附加信息，以便让读者对食谱有更深入的了解。

## 另请参阅

本节提供了对其他有用信息的有用链接。

# 惯例

在本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称如下所示：“前面的输出仍然不会告诉您 Java 的安装位置，这是设置`JAVA_HOME`所必需的。”

代码块设置如下：

```java
<reporting>
  <plugins>
    <plugin>
      <artifactId>maven-project-info-reports-plugin</artifactId>
      <version>2.0.1</version>
      <reportSets>
        <reportSet></reportSet>
      </reportSets>
    </plugin>
  </plugins>
</reporting>
```

当我们希望将您的注意力引到代码块的一个特定部分时，相关的行或项目将以粗体显示：

```java
<settings 

 xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                      http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <localRepository>C:/software/maven</localRepository>
</settings>
```

任何命令行输入或输出都如下所示：

```java
brew install maven

```

**新术语**和**重要词汇**以粗体显示。屏幕上看到的单词，例如在菜单或对话框中，在文本中如下所示：“为了持久化此设置，使用**控制面板**选项设置**环境变量...**，如稍后所述的`M2_HOME`变量。”

### 注意

警告或重要注意事项以如下方框的形式出现。

### 小贴士

小贴士和技巧看起来像这样。

# 读者反馈

读者反馈始终欢迎。让我们知道您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。

要向我们发送一般反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中提及书的标题。

如果您在某个主题上具有专业知识，并且您有兴趣撰写或为本书做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户下载示例代码文件，适用于您购买的所有 Packt 出版图书。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 下载本书的颜色图像

我们还为您提供了一个包含本书中使用的截图/图表的颜色图像的 PDF 文件。这些颜色图像将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/ApacheMavenCookbook_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/ApacheMavenCookbook_ColorImages.pdf)下载此文件。

## 勘误

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然会发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误表，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书的名称。所需信息将出现在**勘误**部分下。

## 侵权

在互联网上侵犯版权材料是一个跨所有媒体持续存在的问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以追究补救措施。

请通过链接将疑似盗版材料发送至 `<copyright@packtpub.com>` 与我们联系。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面的帮助。

## 问题

如果您对本书的任何方面有问题，您可以通过 `<questions@packtpub.com>` 与我们联系，我们将尽力解决问题。
