# 前言

今天我们依赖于可以应用于不同场景的不同软件架构风格。在本书中，我们将回顾最常见的软件架构风格以及它们如何使用 Spring Framework 来实现，这是 Java 生态系统中最广泛采用的框架之一。

一开始，我们将回顾一些与软件架构相关的关键概念，以便在深入技术细节之前理解基本理论。

# 本书适合人群

本书旨在面向有经验的 Spring 开发人员，他们希望成为企业级应用程序的架构师，以及希望利用 Spring 创建有效应用蓝图的软件架构师。

# 本书涵盖内容

第一章，*今日软件架构*，概述了如何管理当今的软件架构以及为什么它们仍然重要。它讨论了软件行业最新需求如何通过新兴的架构模型来处理，以及它们如何帮助您解决这些新挑战。

第二章，*软件架构维度*，回顾了与软件架构相关的维度以及它们如何影响应用程序构建过程。我们还将介绍用于记录软件架构的 C4 模型。

第三章，*Spring 项目*，介绍了一些最有用的 Spring 项目。了解您的工具箱中有哪些工具很重要，因为 Spring 提供了各种工具，可以满足您的需求，并可用于提升您的开发过程。

第四章，*客户端-服务器架构*，涵盖了客户端-服务器架构的工作原理以及可以应用此架构风格的最常见场景。我们将介绍各种实现，从简单的客户端，如桌面应用程序，到现代和更复杂的用途，如连接到互联网的设备。

第五章，*MVC 架构*，介绍了 MVC，这是最流行和广为人知的架构风格之一。在本章中，您将深入了解 MVC 架构的工作原理。

第六章，*事件驱动架构*，解释了与事件驱动架构相关的基本概念，以及它们如何使用实践方法处理问题。

第七章，*管道和过滤器架构*，重点介绍了 Spring Batch。它解释了如何构建管道，这些管道封装了一个独立的任务链，旨在过滤和处理大量数据。

第八章，*微服务*，概述了如何使用 Spring Cloud 堆栈实现微服务架构。它详细介绍了每个组件以及它们如何相互交互，以提供完全功能的微服务架构。

第九章，*无服务器架构*，介绍了互联网上许多现成可用的服务，可以作为软件系统的一部分使用，使公司可以专注于他们自己的业务核心问题。本章展示了一种围绕一系列第三方服务构建应用程序的新方法，以解决身份验证、文件存储和基础设施等常见问题。我们还将回顾什么是 FaaS 方法以及如何使用 Spring 实现它。

《第十章》*容器化您的应用程序*解释了容器是近年来最方便的技术之一。它们帮助我们摆脱手动服务器配置，并允许我们忘记与构建生产环境和服务器维护任务相关的头痛。本章展示了如何生成一个准备好用于生产的构件，可以轻松替换、升级和交换，消除了常见的配置问题。通过本章，我们还将介绍容器编排以及如何使用 Kubernetes 处理它。

《第十一章》*DevOps 和发布管理*解释了敏捷是组织团队和使他们一起更快地构建产品的最常见方法之一。DevOps 是这些团队的固有技术，它帮助他们打破不必要的隔离和乏味的流程，让团队有机会负责从编写代码到在生产环境中部署应用程序的整个软件开发过程。本章展示了如何通过采用自动化来实现这一目标，以减少手动任务并使用自动化管道部署应用程序，负责验证编写的代码、提供基础设施，并在生产环境中部署所需的构件。

《第十二章》*监控*解释了一旦应用程序发布，出现意外行为并不罕见，因此及时发现并尽快修复是至关重要的。本章提供了一些建议，涉及可以用来监控应用程序性能的技术和工具，考虑到技术和业务指标。

《第十三章》*安全*解释了通常安全是团队在开发产品时不太关注的领域之一。开发人员在编写代码时应该牢记一些关键考虑因素。其中大部分都是相当明显的，而其他一些则不是，因此我们将在这里讨论所有这些问题。

《第十四章》*高性能*解释了在应用程序表现出意外行为时，处理生产中的问题比什么都更令人失望。在本章中，我们将讨论一些简单的技术，可以通过每天应用简单的建议来摆脱这些烦人的问题。

# 充分利用本书

在阅读本书之前，需要对 Java、Git 和 Spring Framework 有很好的理解。深入了解面向对象编程是必要的，尽管一些关键概念在前两章中进行了复习。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便直接将文件发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择“支持”选项卡。

1.  单击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

下载文件后，请确保使用以下最新版本解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Software-Architecture-with-Spring-5.0`](https://github.com/PacktPublishing/Software-Architecture-with-Spring-5.0)。我们还有其他代码包，来自我们丰富的图书和视频目录，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。快去看看吧！

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/SoftwareArchitecturewithSpring5_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/SoftwareArchitecturewithSpring5_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这是一个例子：“这个对象由`Servlet`接口表示。”

代码块设置如下：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class ContextAwareTest {

    @Autowired
    ClassUnderTest classUnderTest;

    @Test
    public void validateAutowireWorks() throws Exception {
        Assert.assertNotNull(classUnderTest);
    }
}
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```java
@Service
public class MyCustomUsersDetailService implements UserDetailsService {

    @Override
    public UserDetails loadUserByUsername(String username) 
       throws UsernameNotFoundException {
        Optional<Customer> customerFound = findByUsername(username);
        ...
    }
}
```

任何命令行输入或输出都以以下方式编写：

```java
$ curl -X POST http://your-api-url:8080/events/<EVENT_ID
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会以这种方式出现在文本中。这是一个例子：“从管理面板中选择系统信息。”

警告或重要说明会以这种方式出现。

提示和技巧会以这种方式出现。

# 联系我们
