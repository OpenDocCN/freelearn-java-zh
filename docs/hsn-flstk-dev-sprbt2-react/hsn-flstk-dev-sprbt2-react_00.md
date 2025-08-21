# 前言

在本书中，我们将使用 Spring Boot 2.0 和 React 创建一个现代 Web 应用程序。我们将从后端开始，使用 Spring Boot 和 MariaDB 开发 RESTful Web 服务。我们还将保护后端并为其创建单元测试。前端将使用 React JavaScript 库开发。将使用不同的第三方 React 组件使前端更加用户友好。最后，应用程序将部署到 Heroku。该书还演示了如何将后端 Docker 化。

# 本书适合谁

这本书是为：

+   想要学习全栈开发的前端开发人员

+   想要学习全栈开发的后端开发人员

+   使用其他技术的全栈开发人员

+   熟悉 Spring 但从未构建过全栈应用程序的 Java 开发人员。

# 充分利用本书

读者应具备以下知识：

+   基本的使用一些终端，如 PowerShell 的知识

+   基本的 Java 和 JavaScript 编程知识

+   基本的 SQL 数据库知识

+   基本的 HTML 和 CSS 知识

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便直接通过电子邮件接收文件。

您可以按照以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明进行操作。

文件下载后，请确保使用最新版本的解压缩或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Hands-On-Full-Stack-Development-with-Spring-Boot-2.0-and-React`](https://github.com/PacktPublishing/Hands-On-Full-Stack-Development-with-Spring-Boot-2.0-and-React)。如果代码有更新，将在现有的 GitHub 存储库上进行更新。

我们还有其他代码包，来自我们丰富的书籍和视频目录，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。去看看吧！

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：指示文本中的代码词，数据库表名，文件夹名，文件名，文件扩展名，路径名，虚拟 URL，用户输入和 Twitter 句柄。这是一个例子：“在`domain`包中创建一个名为`CarRepository`的新类。”

代码块设置如下：

```java
@Entity
public class Car {

}
```

任何命令行输入或输出都是这样写的：

```java
mvn clean install
```

**粗体**：指示一个新术语，一个重要的词，或者您在屏幕上看到的词。例如，菜单或对话框中的单词会以这种形式出现在文本中。这是一个例子：“在 Eclipse 的 Project Explorer 中激活根包，右键单击。从菜单中选择 New | Package。”

警告或重要说明会显示在这样的形式下。

提示和技巧会显示在这样的形式下。
