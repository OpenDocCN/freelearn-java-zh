# 前言

Scala 与 Spark 框架相结合，形成了一个丰富而强大的数据处理生态系统。本书将带您深入探索这个生态系统的奥秘。本书中展示的机器学习（ML）项目能够帮助您创建实用、健壮的数据分析解决方案，重点在于使用 Spark ML 管道 API 自动化数据工作流程。本书展示了 Scala 的函数库和其他结构，精心挑选以帮助读者推出他们自己的可扩展数据处理框架。本书中的项目使所有行业的数据从业者能够深入了解数据，帮助组织获得战略和竞争优势。*现代 Scala 项目*专注于监督学习 ML 技术的应用，这些技术可以分类数据和做出预测。您将从实施一个简单的机器学习模型来预测一类花卉的项目开始。接下来，您将创建一个癌症诊断分类管道，随后是深入股票价格预测、垃圾邮件过滤、欺诈检测和推荐引擎的项目。

在本书结束时，您将能够构建满足您软件要求的高效数据科学项目。

# 本书面向的对象

本书面向希望获得一些有趣的实际项目动手经验的 Scala 开发者。需要具备 Scala 的编程经验。

# 本书涵盖的内容

第一章，*从 Iris 数据集中预测花卉类别*，专注于利用基于回归的经过时间考验的统计方法构建机器学习模型。本章将读者引入数据处理，直至训练和测试一个相对简单的机器学习模型。

第二章，*利用 Spark 和 Scala 的力量构建乳腺癌预后管道*，利用公开可用的乳腺癌数据集。它评估了各种特征选择算法，转换数据，并构建了一个分类模型。

第三章，*股票价格预测*，指出股票价格预测可能是一项不可能的任务。在本章中，我们采取了一种新的方法。因此，我们使用训练数据构建和训练一个神经网络模型来解决股票价格预测的明显难以解决的问题。以 Spark 为核心的数据管道将模型的训练分布在集群中的多台机器上。一个真实的数据集被输入到管道中。在训练模型以拟合数据之前，训练数据会经过预处理和归一化步骤。我们还可以提供一种方法来可视化预测结果并在训练后评估我们的模型。

第四章，*构建垃圾邮件分类管道*，告知读者本章的主要学习目标是实现一个垃圾邮件过滤数据分析管道。我们将依赖 Spark ML 库的机器学习 API 及其支持库来构建垃圾邮件分类管道。

第五章，*构建欺诈检测系统*，将机器学习技术和算法应用于构建一个实用的机器学习管道，帮助发现消费者信用卡上的可疑费用。数据来源于公开可访问的消费者投诉数据库。本章展示了 Spark ML 中用于构建、评估和调整管道的工具。特征提取是 Spark ML 提供的一个功能，本章将进行介绍。

第六章，*构建航班性能预测模型*，使我们能够利用航班起飞和到达数据来预测用户的航班是否会延误或取消。在这里，我们将构建一个基于决策树的模型，以推导出有用的预测因子，例如，在一天中什么时间乘坐航班最有可能最小化延误。

第七章，*构建推荐引擎*，引导读者进入可扩展推荐引擎的实现过程。读者将根据用户过去的偏好，逐步通过一个基于阶段的推荐生成过程。

# 为了充分利用本书

假设读者具备 Scala 语言的基础知识。了解如 Spark ML 等基本概念将是一个附加优势。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

按照以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择“支持”标签页。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载完成后，请确保您使用最新版本的 7-Zip/PeaZip 解压缩或提取文件夹。

+   Windows 系统上的 WinRAR/7-Zip

+   Mac 系统上的 Zipeg/iZip/UnRarX

+   Linux 系统上的 7-Zip/PeaZip

书籍的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Modern-Scala-Projects`](https://github.com/PacktPublishing/Modern-Scala-Projects)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

# 下载彩色图片

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/ModernScalaProjects_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/ModernScalaProjects_ColorImages.pdf)

# 使用的约定

本书使用了多种文本约定。

`CodeInText`: 表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“代表女孩年龄的变量名为 `Huan` (`Age_Huan`)。”

代码块按以下方式设置：

```java
val dataFrame = spark.createDataFrame(result5).toDF(featureVector, speciesLabel)
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```java
sc.getConf.getAll
res4: Array[(String, String)] = Array((spark.repl.class.outputDir,C:\Users\Ilango\AppData\Local\Temp\spark-10e24781-9aa8-495c-a8cc-afe121f8252a\repl-c8ccc3f3-62ee-46c7-a1f8-d458019fa05f), (spark.app.name,Spark shell), (spark.sql.catalogImplementation,hive), (spark.driver.port,58009), (spark.debug.maxToStringFields,150),
```

任何命令行输入或输出都按以下方式编写：

```java
scala> val dataSetPath = "C:\\Users\\Ilango\\Documents\\Packt\\DevProjects\\Chapter2\\"
```

**粗体**: 表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要注意事项看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

欢迎读者反馈。

**一般反馈**: 请通过`feedback@packtpub.com`发送电子邮件，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过`questions@packtpub.com`发送电子邮件给我们。

**勘误**: 尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一点。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**: 如果您在互联网上以任何形式发现我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过`copyright@packtpub.com`与我们联系，并附上材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用过这本书，为何不在您购买它的网站上留下评论？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解 Packt 的更多信息，请访问 [packtpub.com](https://www.packtpub.com/).
