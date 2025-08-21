# 前言

REST 是一种解决构建可扩展 Web 服务挑战的架构风格。在当今互联的世界中，API 在 Web 上扮演着核心角色。API 提供了系统相互交互的框架，而 REST 已经成为 API 的代名词。Spring 的深度、广度和易用性使其成为 Java 生态系统中最具吸引力的框架之一。因此，将这两种技术结合起来是非常自然的选择。

从 REST 背后的哲学基础开始，本书介绍了设计和实现企业级 RESTful Web 服务所需的必要步骤。采用实用的方法，每一章都提供了您可以应用到自己情况的代码示例。这第二版展示了最新的 Spring 5.0 版本的强大功能，使用内置的 MVC，以及前端框架。您将学习如何处理 Spring 中的安全性，并发现如何实现单元测试和集成测试策略。

最后，本书通过指导您构建一个用于 RESTful Web 服务的 Java 客户端，以及使用新的 Spring Reactive 库进行一些扩展技术，来结束。

# 这本书适合谁

本书适用于那些想要学习如何使用最新的 Spring Framework 5.0 构建 RESTful Web 服务的人。为了充分利用本书中包含的代码示例，您应该具备基本的 Java 语言知识。有 Spring Framework 的先前经验也将帮助您快速上手。

# 为了充分利用这本书

以下是测试本书中所有代码所需的要求的描述性列表：

+   硬件：64 位机器，至少 2GB RAM 和至少 5GB 的可用硬盘空间

+   软件：Java 9，Maven 3.3.9，STS（Spring Tool Suite）3.9.2

+   Java 9：所有代码都在 Java 9 上测试

+   SoapUI：REST API 调用使用 SoapUI 5.2.1（免费版本）

+   Postman：用于 REST 客户端测试，使用 Postman 5.0.4

# 下载示例代码文件

您可以从您的帐户在[www.packtpub.com](http://www.packtpub.com)下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便文件直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  登录或注册[www.packtpub.com](http://www.packtpub.com/support)。

1.  选择“支持”选项卡。

1.  点击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

一旦文件下载完成，请确保使用最新版本的解压缩或提取文件夹：

+   WinRAR/7-Zip 适用于 Windows

+   Zipeg/iZip/UnRarX 适用于 Mac

+   7-Zip/PeaZip 适用于 Linux

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Building-RESTful-Web-Services-with-Spring-5-Second-Edition`](https://github.com/PacktPublishing/Building-RESTful-Web-Services-with-Spring-5-Second-Edition)。我们还有其他代码包，来自我们丰富的书籍和视频目录，可以在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。快去看看吧！

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/BuildingRESTfulWebServiceswithSpring5_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/BuildingRESTfulWebServiceswithSpring5_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：指示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这是一个例子：“让我们向类中添加一个`Logger`；在我们的情况下，我们可以使用`UserController`。”

代码块设置如下：

```java
@ResponseBody
  @RequestMapping("/test/aop/with/annotation")
  @TokenRequired
  public Map<String, Object> testAOPAnnotation(){
    Map<String, Object> map = new LinkedHashMap<>();
    map.put("result", "Aloha");   
    return map;
  }
```

当我们希望引起你对代码块的特定部分的注意时，相关的行或项目会以粗体显示：

```java
2018-01-15 16:29:55.951 INFO 17812 --- [nio-8080-exec-1] com.packtpub.restapp.HomeController : {test} info
2018-01-15 16:29:55.951 WARN 17812 --- [nio-8080-exec-1] com.packtpub.restapp.HomeController : {test} warn 
2018-01-15 16:29:55.951 ERROR 17812 --- [nio-8080-exec-1] com.packtpub.restapp.HomeController : {test} error
```

任何命令行输入或输出都以以下方式书写：

```java
mvn dependency:tree
```

**粗体**：表示一个新术语，一个重要的词，或者你在屏幕上看到的词。例如，菜单或对话框中的单词会以这种方式出现在文本中。这是一个例子：“现在你可以通过点击生成项目来生成项目。”

警告或重要提示会显示为这样。

提示和技巧会显示为这样。
