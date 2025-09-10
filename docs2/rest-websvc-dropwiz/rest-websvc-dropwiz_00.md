# 前言

Dropwizard 是一个用于 RESTful Web 服务的 Java 开发框架。它最初由 Yammer 构建，用作其后端系统的基础。Dropwizard 是生产就绪的；它封装了您进行 RESTful 开发所需的一切。

Jersey、Jackson、jDBI 和 Hibernate 只是 Dropwizard 包含的一些库。基于 Dropwizard 构建的应用程序运行在嵌入的 Jetty 服务器上——您无需担心应用程序的部署位置或是否与您的目标容器兼容。

使用 Dropwizard，您将能够以最少的努力和时间高效地构建快速、安全且可扩展的 Web 服务应用程序。

Dropwizard 是开源的，其所有模块都可通过 Maven 仓库获取。这样，您只需在您的 `pom.xml` 文件中添加适当的依赖项条目，就能集成您想要的每个库——如果它尚未存在。需要具备 Maven 的基本知识和理解。

# 本书涵盖内容

第一章, *Dropwizard 入门*，将引导您了解 Dropwizard 的基础知识，帮助您熟悉其概念并准备开发环境。

第二章, *创建 Dropwizard 应用程序*，将介绍 Maven 及其如何用于创建 Dropwizard 应用程序。这包括基于默认工件生成空应用程序的结构，以及启动构建 Dropwizard 应用程序所需的必要修改。

第三章, *配置应用程序*，介绍了通过启用配置文件和配置类（该类负责获取、验证并使配置值在应用程序中可用）来外部化应用程序配置的方法。

第四章, *创建和添加 REST 资源*，将指导您了解应用程序最重要的方面：资源类。您将学习如何将 URI 路径和 HTTP 动词映射到资源类的方 法，以及如何向 Dropwizard 应用程序添加新资源。

第五章, *表示形式 – RESTful 实体*，讨论了将表示形式建模为实际 Java 类以及 Jackson 如何自动将 POJO 转换为 JSON 表示形式。

第六章, *使用数据库*，展示了 jDBI 的集成和使用方法，如何从接口创建数据访问对象，以及如何使用 jDBI 的 SQL 对象 API 与数据库交互。本章还介绍了所需的附加配置修改。

第七章, *验证 Web 服务请求*，介绍了如何使用 Hibernate Validator 在满足请求之前对来自 Web 服务客户端的请求进行验证。

第八章, *Web 服务客户端*，演示了如何创建一个由 Dropwizard 应用程序使用的托管 Jersey HTTP 客户端，以便通过`WebResource`对象与 Web 服务进行交互。

第九章, *认证*，介绍了 Web 服务认证的基础知识，并指导您实现基本的 HTTP 认证器，以及如何将其适配到您的资源类和应用程序的 HTTP 客户端。

第十章, *用户界面 – 视图*，展示了如何使用 Dropwizard 视图包和 Mustache 模板引擎来为 Web 服务客户端创建 HTML 界面。

附录 A, *测试 Dropwizard 应用程序*，展示了如何使用 Dropwizard 的测试模块创建自动化集成测试。本附录还涉及实现应用程序的运行时测试，这些测试被称为健康检查。您将指导实现一个健康检查，以确保您的 HTTP 客户端确实可以与 Web 服务进行交互。

附录 B, *部署 Dropwizard 应用程序*，解释了您需要采取的必要步骤，以便通过使用单独的配置文件并保护应用程序的管理端口访问来将 Dropwizard 应用程序部署到 Web 服务器。

# 您需要为此书准备的内容

为了跟随书中提供的示例和代码片段，您需要一个安装有 Linux、Windows 或 OS X 操作系统的计算机。一个现代的 Java 代码编辑器/ IDE，如 Eclipse、Netbeans 或 IDEA，将真正帮助您。您还需要 Java 开发工具包（JDK）的版本 7 以及 Maven 和 MySQL 服务器。额外的依赖项将由 Maven 获取，因此您需要一个有效的互联网连接。

# 本书面向的对象

本书的目标读者是至少具备基本 Java 知识和 RESTful Web Services 基本理解的软件工程师和网络开发者。了解 SQL/MySQL 的使用和命令行脚本也可能是有必要的。

# 习惯用法

在本书中，您将找到多种文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称的显示方式如下："在`Contact`类中添加一个名为`#isValidPerson()`的新方法。"。

代码块设置如下：

```java
import java.util.Set;
import javax.validation.ConstraintViolation;
import javax.util.ArrayList;
import javax.validation.Validator;
import javax.ws.rs.core.Response.Status;
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目将以粗体显示：

```java
private final ContactDAO contactDao; private final Validator validator;
  public ContactResource(DBI jdbi, Validator validator) {
 contactDao = jdbi.onDemand(ContactDAO.class); this.validator = validator;
  }
```

任何命令行输入或输出都应如下编写：

```java
$> java -jar target/app.jar server conf.yaml

```

**新术语**和**重要词汇**将以粗体显示。你会在屏幕上看到，例如在菜单或对话框中的文字，将以这种方式显示：“在某个时候，你将被提示提供**MySQL Root 密码**。”

### 注意

警告或重要注意事项将以这种方式显示在框中。

### 小贴士

小技巧和技巧看起来像这样。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们你对这本书的看法——你喜欢什么或者可能不喜欢什么。读者的反馈对我们开发你真正能从中获得最大收益的标题非常重要。

要向我们发送一般反馈，只需发送电子邮件到`<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果你在一个领域有专业知识，并且你对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是 Packt 图书的骄傲拥有者，我们有许多事情可以帮助你从购买中获得最大收益。

## 下载示例代码

你可以从你购买的所有 Packt 书籍的账户中下载示例代码文件。[`www.packtpub.com`](http://www.packtpub.com)。如果你在其他地方购买了这本书，你可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接将文件通过电子邮件发送给你。

## 错误清单

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然会发生。如果你在我们的书中发现错误——可能是文本或代码中的错误——如果你能向我们报告这一点，我们将不胜感激。通过这样做，你可以帮助其他读者避免挫折，并帮助我们改进本书的后续版本。如果你发现任何错误清单，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择你的书，点击**错误清单提交表单**链接，并输入你的错误清单详情。一旦你的错误清单得到验证，你的提交将被接受，错误清单将被上传到我们的网站，或添加到该标题的错误清单部分。任何现有的错误清单都可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)中选择你的标题来查看。

## 盗版

在互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果你在网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以追究补救措施。

请通过 `<copyright@packtpub.com>` 联系我们，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者方面的帮助，以及我们为您提供有价值内容的能力。

## 问题和版权

如果您在本书的任何方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决。
