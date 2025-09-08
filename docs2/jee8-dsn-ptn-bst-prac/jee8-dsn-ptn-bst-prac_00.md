# 前言

随着时间的推移，企业界在优化流程和帮助企业增加利润以及改善服务或产品方面投入了越来越多的技术和应用。企业环境面临需要面对的挑战，以实施良好的解决方案，例如服务的高可用性、在需要时改变的能力、服务扩展的能力以及处理大量数据的能力。因此，创建了新的应用来优化流程和增加利润。Java 语言和 Java EE 是创建企业环境应用的优秀工具，因为 Java 语言是跨平台的、开源的、经过广泛测试的，并且拥有强大的社区和生态系统。此外，Java 语言拥有 Java EE，这是一个允许我们开发者开发企业应用而不依赖于供应商的规范集合。企业应用的开发有一些众所周知的问题会反复出现。这些问题涉及服务的集成、应用程序的高可用性和弹性。

这本书将解释 Java EE 8 的概念、其层级结构以及如何使用 Java EE 8 最佳实践来开发企业应用。此外，这本书将展示我们如何使用 Java EE 8 与设计模式和企业模式相结合，以及我们如何使用面向方面编程、响应式编程和 Java EE 8 的微服务来优化我们的解决方案。在这本书的整个过程中，我们将了解集成模式、响应式模式、安全模式、部署模式和操作模式。这本书的结尾，我们将对 MicroProfile 有一个概述，以及它如何帮助我们使用微服务架构开发应用。

# 这本书面向的对象

这本书是为想要学习使用设计模式、企业模式和 Java 最佳实践来开发和交付企业应用的 Java 开发者而写的。读者需要了解 Java 语言和 Java EE 的基本概念。

# 为了充分利用这本书

1.  在阅读这本书之前，读者需要了解面向对象的概念、Java 语言以及 Java EE 的基本概念。在这本书中，我们假设读者已经了解 Java EE 伞形规范的一些规格，例如 EJB、JPA 和 CDI 等。

1.  要测试这本书的代码，您需要一个支持 Java EE 8 的应用服务器，例如 GlassFish 5.0。此外，您需要使用 IntelliJ、Eclipse、NetBeans 或其他支持 Java 语言的任何 IDE。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载这本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在 [www.packtpub.com](http://www.packtpub.com/support) 登录或注册。

1.  选择“支持”标签。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载后，请确保使用最新版本的软件解压或提取文件夹：

+   Windows 下的 WinRAR/7-Zip

+   Mac 下的 Zipeg/iZip/UnRarX

+   Linux 下的 7-Zip/PeaZip

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Java-EE-8-Design-Patterns-and-Best-Practices`](https://github.com/PacktPublishing/Java-EE-8-Design-Patterns-and-Best-Practices)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包可供使用，网址为**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。请查看它们！

# 下载彩色图像

我们还提供包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/JavaEE8DesignPatternsandBestPractices_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/JavaEE8DesignPatternsandBestPractices_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账号。以下是一个示例：“还重要的是要注意，`@Electronic`修饰符标识了被修饰的对象。”

代码块设置如下：

```java
public interface Engineering {
  List<String> getDisciplines ();
}
public class BasicEngineering implements Engineering {

  @Override
  public List<String> getDisciplines() {
    return Arrays.asList("d7", "d3");
  }
}
@Electronic
public class ElectronicEngineering extends BasicEngineering {
   ...  
}
@Mechanical
public class MechanicalEngineering extends BasicEngineering {
   ...
}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
@Loggable
@Interceptor
public class LoggedInterceptor implements Serializable {

  @AroundInvoke
  public Object logMethod (InvocationContext invocationContext) throws 
  Exception{
    System.out.println("Entering method : "
        + invocationContext.getMethod().getName() + " " 
        + invocationContext.getMethod().getDeclaringClass()
        );
    return invocationContext.proceed();
  }
}
```

任何命令行输入或输出都如下所示：

```java
creating bean.
intercepting post construct of bean.
post construct of bean
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“用户登录后，当他们访问**应用程序 1**、**应用程序 2**或**应用程序 3**时，他们无需再次登录。”

警告或重要提示如下所示。

技巧和窍门如下所示。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：请通过电子邮件`feedback@packtpub.com`发送反馈，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过电子邮件`questions@packtpub.com`联系我们。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将非常感激您能向我们报告。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上遇到任何形式的我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过发送链接至`copyright@packtpub.com`与我们联系。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且对撰写或参与一本书籍感兴趣，请访问 [authors.packtpub.com](http://authors.packtpub.com/).

# 评论

请留下评论。一旦您阅读并使用了这本书，为何不在您购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

想了解更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/).
