# 前言

Spring 框架是一个成熟、企业级和开源的应用程序开发框架，它提供了一个灵活且现代的替代方案，用于官方 Java EE 标准。

# 这本书面向的对象

这本书旨在帮助那些处于职业生涯的初级或早期中级阶段的开发者，他们希望能够轻松地使用 Java 平台创建健壮的 Web 应用程序或 RESTful 服务。你应该至少具备基本的 Java 知识，并且知道如何使用 Maven 编译带有给定 POM 文件的程序。假设你处于初级或早期中级阶段。你不需要成为 HTML 专家，但应该了解 HTML 的工作原理以及如何保持文件 XML/XHTML 的合规性。

如果你想要使用 Java 创建现代 Web 应用程序或 RESTful 服务，你应该参加这门课程。你还将学习很多关于 Spring 框架本身的知识，这将使你能够在之后独立创建完整的应用程序。

# 这本书涵盖的内容

*第一章*, *Spring 项目和框架*，介绍了 Spring 框架及其原则。然后我们第一次尝试构建和运行项目。在关注 Spring 的主要构建块之后，本章最后展示了如何使用 Project Lombok 库。

*第二章*, *构建 Spring 应用程序*，带我们了解 Spring Bean 的配置类以及它们的各种依赖关系，然后展示如何创建和配置不同的环境。

*第三章*, *测试 Spring 应用程序*，涵盖了创建和分析两种不同类型的测试，即单元测试和集成测试。

*第四章*, *MVC 模式*，讨论了上述模式，详细解释了模型、视图和控制器的概念。本章简要介绍了应用程序的开发和不同控制器的实现。

*第五章*, *使用网页显示信息*，训练学生使用模板引擎 Thymleaf，它的语法；以及元素；然后如何使用 Thymleaf 构建网页。

*第六章*, *在视图和控制之间传递数据*，教我们如何创建表单以及在网页浏览器中输入信息时使用的不同输入字段。

*第七章*, *RESTful API*，涵盖了 REST 的不同方面，如何使用 Postman，以及编写 RESTful API 的过程。

*第八章*, *Web 应用程序安全*，讨论了 Web 应用程序安全方面的内容，以及 Spring 中的不同安全选项。

*第九章*，*使用数据库持久化数据*，涵盖选择数据库管理系统以及与关系数据库和 SQL 一起工作；理解使用 JDBC 和 JdbcTemplate 进行数据库访问，以及使用 Spring 进一步的数据实现。

# 要充分利用本书

要成功完成本书，您将需要至少配备 Intel Core i5 处理器或等效处理器、8 GB RAM 和 5 GB 可用存储空间的计算机系统。此外，您还需要以下软件：

+   操作系统：Windows 7 或更高版本

+   浏览器：安装了最新更新的 Google Chrome 或 Mozilla Firefox

+   IntelliJ Community Edition（或 Ultimate）-最新版本

+   JDK 8+

+   Maven 3.3+

# 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的账户下载本书的示例代码文件。如果您在其他地方购买了此书，您可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择支持选项卡。

1.  点击代码下载和勘误表。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载后，请确保使用最新版本解压缩或提取文件夹：

+   适用于 Windows 的 WinRAR/7-Zip

+   适用于 Mac 的 Zipeg/iZip/UnRarX

+   适用于 Linux 的 7-Zip/PeaZip

书籍的代码包也托管在 GitHub 上，地址为[`github.com/TrainingByPackt/Spring-Boot-2-Fundamentals`](https://github.com/TrainingByPackt/Spring-Boot-2-Fundamentals)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有来自我们丰富的书籍和视频目录中的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

# 使用的约定

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称的显示方式如下：“常规作用域在`ConfigurableBeanFactory`类中定义，而特定于 Web 应用程序的作用域在`WebApplicationContext`中定义。”

代码块设置为如下：

```java
@Repository
public class ExampleBean {
   @Autowired
   private DataSource dataSource;
...
}
```

**粗体**：新术语和重要单词以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下：“默认消息应该是 INPUT+ **问候世界**。”

警告或重要说明如下所示。

技巧和窍门如下所示。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请在邮件主题中提及书籍标题，并通过`customercare@packtpub.com`给我们发邮件。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一点。请访问 [www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何非法副本，无论形式如何，如果您能提供位置地址或网站名称，我们将不胜感激。请通过`copyright@packt.com`与我们联系，并附上材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/).

# 评论

请留下评论。一旦您阅读并使用了这本书，为何不在您购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

想了解更多关于 Packt 的信息，请访问 [packt.com](http://www.packt.com/).
