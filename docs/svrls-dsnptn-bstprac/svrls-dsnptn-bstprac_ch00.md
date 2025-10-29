# 前言

无服务器架构正在改变软件系统的构建和运营方式。与使用物理服务器或虚拟机的系统相比，许多工具、技术和模式保持不变；然而，有几件事情可能或需要发生剧烈变化。为了充分利用无服务器系统的优势，在开始无服务器之旅之前，应该仔细思考工具、模式和最佳实践。

本书介绍了适用于几乎所有类型无服务器应用程序的可重用模式，无论是 Web 系统、数据处理、大数据还是物联网。通过示例和解释，你将了解无服务器环境中的各种模式，例如 RESTful API、GraphQL、代理、扇出、消息传递、Lambda 架构和 MapReduce，以及何时使用这些模式来使你的应用程序可扩展、高性能和容错。本书将带你了解持续集成和持续部署的技术，以及测试、保护和扩展你的无服务器应用程序的设计。学习和应用这些模式将加快你的开发周期，同时提高在所选无服务器平台之上构建的整体应用程序架构。

# 本书的目标读者

这本书的目标读者是软件工程师、架构师以及任何对使用云服务提供商构建无服务器应用程序感兴趣的人。读者应该对学习流行的模式以提高敏捷性、代码质量和性能感兴趣，同时避免新用户在开始使用无服务器系统时可能陷入的一些陷阱。假设读者具备编程知识和基本的无服务器计算概念。

# 本书涵盖的内容

第一章，*介绍*，涵盖了无服务器系统的基础知识，并讨论了无服务器架构可能或可能不适合的情况。介绍了三类无服务器模式，并简要解释了它们。

第二章，*使用 REST 构建的三层 Web 应用程序*，通过一个完整的示例，指导你如何使用 AWS Lambda 驱动的 REST API 构建传统的 Web 应用程序，以及使用无服务器技术托管 HTML、CSS 和 JavaScript 前端代码。

第三章，*使用 GraphQL 的三层 Web 应用程序模式*，介绍了 GraphQL，并解释了将之前的 REST API 转换为 GraphQL API 所需的变化。

第四章，*使用代理模式集成遗留 API*，展示了如何在仅使用 AWS API 网关的情况下，完全不改变 API 合约，同时使用遗留 API 后端。

第五章，*使用 Fan-Out 模式扩展*，教您最基本的无服务器模式之一，其中单个事件触发多个并行无服务器函数，从而在串行实现中实现更快的执行时间。

第六章，*使用消息模式进行异步处理*，解释了不同的消息模式类别，并演示了如何使用无服务器数据生产者将消息放入队列，并使用无服务器数据消费者在下游处理这些消息。

第七章，*使用 Lambda 模式进行数据处理*，解释了如何使用多个子模式创建两个计算平面，这些平面提供了对历史聚合数据和实时数据的视图。

第八章，*MapReduce 模式*，探讨了并行聚合大量数据的示例实现，类似于 Hadoop 等系统的工作方式。

第九章，*部署和 CI/CD 模式*，解释了如何为无服务器项目设置持续集成和持续交付，以及在此过程中需要注意的事项，同时还展示了持续部署的示例。

第十章，*错误处理和最佳实践*，回顾了自动跟踪意外错误的工具和技术，以及创建无服务器应用程序时的几个最佳实践和技巧。

# 为了充分利用本书

1.  本书中的几乎所有示例都使用 Serverless Framework 来管理 AWS 资源和 Lambda 函数。Serverless Framework 的安装说明可以在[`serverless.com/framework/docs/getting-started/`](https://serverless.com/framework/docs/getting-started/)找到。

1.  除了 Serverless Framework 之外，读者还需要一个 AWS 账户来运行示例。对于 AWS 新手，您可以在[`aws.amazon.com`](https://aws.amazon.com)创建一个新账户，该账户在免费层中提供一年的使用时间。

    在本书的学习过程中，您将需要以下工具：

    +   AWS Lambda

    +   AWS RDS

    +   AWS API Gateway

    +   AWS DynamoDB

    +   AWS S3

    +   AWS SQS

    +   AWS Rekognition

    +   AWS Kinesis

    +   AWS SNS

    我们将通过本书的学习过程来学习如何使用这些工具。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户中下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择支持选项卡。

1.  点击代码下载和勘误表。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载后，请确保使用最新版本解压缩或提取文件夹：

+   适用于 Windows 的 WinRAR/7-Zip

+   适用于 Mac 的 Zipeg/iZip/UnRarX

+   适用于 Linux 的 7-Zip/PeaZip

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Serverless-Design-Patterns-and-Best-Practices`](https://github.com/PacktPublishing/Serverless-Design-Patterns-and-Best-Practices)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。查看它们吧！

# 使用的约定

本书使用了多种文本约定。

`CodeInText`: 表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“为了测试这个，我们需要将`divide`函数的超时设置为 4 秒，并在应用程序代码的中间放置`time.sleep(3)`。”

代码块按照以下方式设置：

```java
def divide(event, context):
    params = event.get('queryStringParameters') or {}
    numerator = int(params.get('numerator', 10))
    denominator = int(params.get('denominator', 2)) 
    body = { 
        "message": "Results of %s / %s = %s" % ( 
            numerator, 
            denominator,
            numerator // denominator,
        ) 
    } 

    response = { 
        "statusCode": 200,
        "body": json.dumps(body)
    } 

    return response
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
from raven_python_lambda import RavenLambdaWrapper

@RavenLambdaWrapper()
```

```java
from raven_python_lambda import RavenLambdaWrapper

@RavenLambdaWrapper()
def divide(event, context):
    # Code
```

任何命令行输入或输出都按照以下方式编写：

```java
$ curl "https://5gj9zthyv1.execute-api.us-west-2.amazonaws.com/dev?numerator=12&denominator=3"
{"message": "Results of 12 / 3 = 4"}
```

**粗体**: 表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“以下截图显示了 AWS Lambda 监控页面中`divide`函数的调用错误：”

警告或重要注意事项看起来像这样。

技巧和窍门看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**: 请发送电子邮件至`feedback@packtpub.com`，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过`questions@packtpub.com`发送电子邮件给我们。

**勘误**: 尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一错误。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**: 如果您在互联网上以任何形式遇到我们作品的非法副本，我们将不胜感激，如果您能提供位置地址或网站名称。请通过`copyright@packtpub.com`与我们联系，并提供材料的链接。

**如果您想成为作者**: 如果您在某个领域有专业知识，并且对撰写或参与一本书籍感兴趣，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下您的评价。一旦您阅读并使用了这本书，为何不在购买它的网站上留下评价呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 可以了解您对我们产品的看法，而我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解 Packt 的更多信息，请访问 [packtpub.com](https://www.packtpub.com/).
