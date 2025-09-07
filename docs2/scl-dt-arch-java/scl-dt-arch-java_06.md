# 6

# 架构实时处理管道

在上一章中，我们学习了如何为高吞吐量的基于批处理的数据工程问题架构大数据解决方案。然后，我们学习了如何使用 Glue DataBrew 对大数据进行概要分析。最后，我们学习了如何逻辑上选择各种技术，在云中构建基于 Spark 的完整大数据解决方案。

在本章中，我们将讨论如何分析、设计和实现一个实时数据分析解决方案来解决业务问题。我们将学习如何借助分布式消息系统（如 Apache Kafka）实现数据的流式传输和处理，从而实现处理速度和可靠性的提升。在这里，我们将讨论如何编写 Kafka Streams 应用程序来处理和分析流式数据，并使用 Kafka 连接器将实时处理引擎的结果存储到 NoSQL 数据库（如 MongoDB、DynamoDB 或 DocumentDB）中。

在本章结束时，你将了解如何使用 Java 和 Kafka 相关技术构建一个实时流式解决方案来预测贷款申请的风险类别。你还将了解实时数据分析问题的设计和架构。在整个过程中，你将学习如何将事件发布到 Kafka，使用 Kafka Streams 分析数据，并将分析结果实时存储到 MongoDB 中。通过这样做，你将了解如何处理实时数据工程问题并构建有效的流式解决方案。

在本章中，我们将涵盖以下主要主题：

+   理解和分析流式问题

+   架构解决方案

+   实施和验证设计

# 技术要求

要完成本章，你需要以下内容：

+   熟悉 Java

+   在本地系统上安装 Java 1.8 或更高版本、Maven、Apache Kafka 和 PostgreSQL

+   在云中拥有 MongoDB Atlas 订阅

+   在本地系统上安装 IntelliJ IDEA Community 或 Ultimate 版本

# 理解和分析流式问题

到目前为止，我们已经探讨了涉及数据摄取、存储或分析的数据工程问题。然而，在当今竞争激烈的市场环境中，在线 Web 应用和移动应用使得消费者更加苛刻且缺乏耐心。因此，企业必须适应并实时做出决策。在本章中，我们将尝试解决这样一个实时决策问题。

## 问题陈述

一家名为 XYZ 的金融公司，提供信用卡服务，其信用卡申请是实时工作并使用各种用户界面，如移动和在线网络应用程序。由于客户有多种选择并且不太有耐心，XYZ 想确保信贷贷款官员可以在瞬间或实时决定信用卡批准。为此，需要分析应用程序并为每个申请生成一个信用风险评分。这个风险评分，连同必要的申请参数，将帮助信贷贷款官员快速做出决定。

## 分析问题

让我们分析给定的问题。首先，让我们从数据的四个维度来分析需求。

首先，我们将尝试回答*数据速度是多少？*这是这个问题的关键因素。正如问题陈述所示，与我们的先前问题不同，源数据是实时接收的，数据分析也需要实时进行。这类问题非常适合实时流解决方案。

现在，我们需要讨论的下一个维度是*体积*。然而，由于我们的问题涉及流数据，讨论数据的总体体积是没有意义的。相反，我们应该回答诸如*平均每分钟或每小时提交多少应用程序？在高峰时段会怎样？这个体积在未来会增加吗？如果会增加，它可能会增加多少次以及频率如何？*等问题。我们应该带着这些问题回到客户那里。通常情况下，如果客户是第一次创建实时管道，这些答案可能并不容易获得。在这种情况下，我们应该要求客户提供最细粒度的平均数据速度信息（在这种情况下，指的是每天、每周或每月提交的申请数量），然后计算每分钟的平均预期体积。此外，为了了解体积的增加，我们可以询问关于一年内销售方面的目标预测，并尝试预测体积增加。

假设客户每天接收一百万个申请，并且他们的目标是未来两年内将销售额增加 50%。考虑到通常的批准率为 50%，我们可以预期申请提交率将增加两倍。这意味着我们可能会在未来每天预期到两百万个申请的体积。

由于我们的解决方案需要是实时的，必须处理超过一百万条记录，并且预计体积在未来可能会增加，因此以下特性对于我们的流解决方案是必不可少的：

+   应该健壮

+   应支持解决方案内部各种系统以及外部源/汇之间的异步通信

+   应确保零数据丢失

+   应该具有容错性，因为我们正在实时处理数据

+   应该可扩展

+   即使数据量增加，也应提供出色的性能

考虑到所有这些因素，我们应该选择一个发布/订阅消息系统，因为这可以确保可伸缩性、容错性、更高的并行性和消息投递保证。分布式消息/流平台，如 Apache Kafka、AWS Kinesis 和 Apache Pulsar，最适合解决我们的问题。

接下来，我们将关注数据的**多样性**。在一个典型的流平台中，我们以事件的形式接收数据。每个事件通常包含一条记录，尽管有时可能包含多条记录。通常，这些事件以平台无关的数据格式（如 JSON 和 Avro）传输。在我们的用例中，我们将以 JSON 格式接收数据。在实际的生产场景中，数据可能是 Avro 格式。

实时流解决方案面临的一个挑战是数据的**真实性**。通常，真实性是基于数据可能出现的各种噪声可能性来确定的。然而，真实性的准确分析是在实时项目实施和用真实数据进行测试时发生的。与许多软件工程解决方案一样，实时数据工程解决方案会随着时间的推移而成熟，以处理噪声和异常。

为了简化问题，我们假设输入主题中发布的数据已经是干净的，因此我们不会在我们的当前用例中讨论真实性。然而，在现实世界中，通过输入主题接收到的数据可能包含异常和噪声，这需要我们注意。在这种情况下，我们可以编写一个 Kafka Streams 应用程序来清理和格式化数据，并将其放入处理主题。此外，从输入主题中移动错误记录到错误主题；它们不会被发送到处理主题。然后，数据分析流应用程序从处理主题（仅包含干净数据）中消费数据。

现在我们已经分析了该问题的数据维度，并得出结论，我们需要构建一个实时流管道，我们的下一个问题将是，*哪个平台？云还是本地？*

为了回答这些问题，让我们看看我们有哪些限制。为了分析流数据，我们必须提取并读取每位客户的信用历史记录。然而，由于客户的信用历史是敏感信息，我们更愿意从本地应用程序中使用这些信息。然而，公司的移动或 Web 后端系统部署在云上。因此，将分析数据存储在云上是有意义的，因为移动或其他 Web 应用程序从云上获取数据比从本地获取数据所需的时间更少。因此，在这种情况下，我们将采用混合方法，其中信用历史数据将存储在本地，数据将在本地进行分析和处理，但结果数据将存储在云上，以便可以轻松地从移动和 Web 后端系统检索。

在本节中，我们分析了数据工程问题，并意识到这是一个实时流处理问题，处理将在本地进行。在下节中，我们将使用这次分析的结果，连接点来设计数据管道和选择正确的技术堆栈。

# 设计解决方案

为了设计解决方案，让我们总结一下上一节中讨论的分析。以下是我们可以得出的结论：

+   这是一个实时数据工程问题。

+   此问题可以使用 Kafka 或 Kinesis 等流平台解决。

+   每天将发布 100 万条事件，事件量有可能随时间增加。

+   解决方案应托管在混合平台上，数据处理和分析在本地进行，结果存储在云中以方便检索。

由于我们的流媒体平台是本地部署的，并且可以在本地服务器上维护，因此 Apache Kafka 是一个很好的选择。它支持分布式、容错、健壮和可靠的架构。它可以通过增加分区数量来轻松扩展，并提供至少一次投递保证（这确保了所有事件至少会投递一次，不会发生事件丢失）。

现在，让我们看看我们将如何确定结果和其他信息将如何存储。在本用例中，个人的信用历史具有结构化格式，应存储在本地。关系数据库管理系统（RDBMS）是此类数据存储的绝佳选择。在这里，我们将使用 PostgreSQL，因为 PostgreSQL 是开源的、企业级的、健壮的、可靠的、高性能的（而且因为我们将其用作 RDBMS 选项在*第四章*，*ETL 数据加载 - 数据仓库中数据摄入的基于批处理解决方案*））。与信用历史不同，应用程序需要由运行在 AWS 上的移动和 Web 后端访问，因此数据存储应在云上。

此外，让我们考虑这些数据将主要被移动和 Web 后端应用程序消费。那么，将数据存储在可以由 Web 和移动后端轻松拉取和使用的文档格式中是否值得？MongoDB Atlas 在 AWS 云上是一个以可扩展方式存储文档的绝佳选择，并且具有按使用付费的模式。我们将使用 AWS 上的 MongoDB Atlas 作为结果数据的汇点。

现在，让我们讨论我们将如何实时处理数据。数据将以事件的形式发送到 Kafka 主题。我们将编写一个流应用程序来处理并将结果事件写入输出主题。生成的记录将包含风险评分。要将数据从 Kafka 导出到任何其他数据存储或数据库，我们可以编写一个消费者应用程序或使用 Kafka Sink 连接器。编写 Kafka 消费者应用程序需要开发和维护工作。然而，如果我们选择使用 Kafka Connect，我们只需配置它即可获得 Kafka 消费者的好处。Kafka Connect 交付更快，维护更简单，并且更健壮，因为所有异常处理和边缘情况都已妥善处理，并有良好的文档记录。因此，我们将使用 Kafka Sink 连接器将输出主题的结果事件保存到 MongoDB Atlas 数据库。

下图描述了解决方案架构：

![图 6.1 – 我们实时信用风险分析器的解决方案架构](img/B17084_06_001.jpg)

图 6.1 – 我们实时信用风险分析器的解决方案架构

如前图所示，我们的解决方案架构如下：

+   一个新的应用程序事件被发布到输入 Kafka 主题

+   Kafka Streams 应用程序（风险计算器应用程序）读取应用程序事件并从信用历史数据库获取申请人的相应信用历史

+   风险计算器应用程序创建并发送一个包含所有必需参数的 HTTP 请求到风险评分生成器应用程序

+   风险评分生成器应用程序使用已经训练好的机器学习模型来计算应用程序的风险评分，并将结果返回给风险计算器应用程序

+   风险计算器应用程序生成丰富的应用程序事件，并将结果事件写入输出主题

+   配置在输出主题上的 Kafka Sink 连接器负责消费并将数据写入 MongoDB Atlas 云数据库

+   如果在 Kafka 流处理过程中出现处理错误，将错误消息以及输入事件写入错误数据库

现在我们已经学习了如何为我们的实时数据分析需求构建解决方案，让我们学习如何实现架构。

# 实施和验证设计

在这种实时实现中的第一步是设置流平台。为了实现我们的架构，我们需要在本地机器上安装 Apache Kafka 并创建必要的主题。

## 在您的本地机器上设置 Apache Kafka

在本节中，您将学习如何设置 Apache Kafka 集群，运行它，以及创建和列出主题。按照以下步骤操作：

1.  从[`archive.apache.org/dist/kafka/2.8.1/kafka_2.12-2.8.1.tgz`](https://archive.apache.org/dist/kafka/2.8.1/kafka_2.12-2.8.1.tgz)下载 Apache Kafka 版本 2.8.1。

1.  提取`kafka_2.12-2.8.1.tgz`存档文件。以下命令将帮助您在 Linux 或 macOS 上执行相同的操作：

    ```java
    $ tar -xzf kafka_2.12-2.8.1.tgz
    $ cd kafka_2.12-2.8.1
    ```

1.  导航到 Kafka 安装根目录，并使用以下命令启动 zookeeper：

    ```java
    $ bin/zookeeper-server-start.sh config/zookeeper.properties
    ```

1.  接下来，使用以下命令运行 Kafka 服务器：

    ```java
    $ bin/kafka-server-start.sh config/server.properties
    ```

1.  接下来，使用以下命令创建主题：

    ```java
    $ bin/kafka-topics.sh --create --topic landingTopic1 --bootstrap-server localhost:9092
    $ bin/kafka-topics.sh --create --topic enrichedTopic1 --bootstrap-server localhost:9092
    ```

为了简化，我们定义了一个分区，并将副本因子设置为 1。但在实际的生产环境中，副本因子应该是三个或更多。分区的数量基于需要处理的数据量及其速度，以及它们应该以何种最佳速度进行处理。

1.  我们可以使用以下命令列出在集群中创建的主题：

    ```java
    $ bin/kafka-topics.sh --describe --bootstrap-server localhost:9092
    ```

现在我们已经安装了 Apache Kafka 并创建了所需的主题，您可以将注意力集中在在本地机器上安装的 PostgreSQL 实例中创建信用记录表和错误表。这些表的 DDL 和 DML 语句可在[`github.com/PacktPublishing/Scalable-Data-Architecture-with-Java/tree/main/Chapter06/SQL`](https://github.com/PacktPublishing/Scalable-Data-Architecture-with-Java/tree/main/Chapter06/SQL)找到。

参考笔记

如果您是 Kafka 的新手，我建议通过阅读官方 Kafka 文档来学习基础知识：[`kafka.apache.org/documentation/#gettingStarted`](https://kafka.apache.org/documentation/#gettingStarted)。或者，您可以参考由*Neha Narkhede*、*Gwen Sharipa*和*Todd Palino*合著的书籍《Kafka，权威指南》。

在本节中，我们设置 Kafka 流平台和信用记录数据库。在下一节中，我们将学习如何实现 Kafka 流应用程序，以实时处理到达*landingTopic1*的应用程序事件。

## 开发 Kafka 流应用程序

在我们实现解决方案之前，让我们探索并理解一些关于 Kafka Streams 的基本概念。Kafka Streams 提供了一个客户端库，用于实时处理和分析数据，并将处理后的结果发送到接收器（最好是输出主题）。

流是一个抽象，表示 Kafka Streams 中无界、持续更新的数据。流处理应用程序是使用 Kafka Streams 库编写的程序，用于处理流中的数据。它使用拓扑定义处理逻辑。Kafka Streams 拓扑是一个由流处理器作为节点和流作为边的图。以下图显示了 Kafka Streams 的一个示例拓扑：

![图 6.2 – Kafka Streams 拓扑示例](img/B17084_06_002.jpg)

图 6.2 – Kafka Streams 拓扑示例

如您所见，一个拓扑由**流处理器**组成——这些节点和边代表流。可以有两种特殊流处理器节点，如下所示：

+   **源处理器**：这是一个特殊的流处理节点，它从消费一个或多个 Kafka 主题的消息中产生数据输入流

+   **汇处理器**：正如其名所示，汇处理器从上游消耗数据并将其写入汇点或目标主题

在 Kafka 流应用程序中，可以使用低级处理器 API 或使用高级**领域特定语言**（**DSL**）API 构建拓扑。当一个事件发布到源 Kafka 主题时，拓扑被触发，使用拓扑定义处理该事件，并将处理后的事件发布到汇点主题。一旦拓扑在源事件上成功调用并完成，事件偏移量将被提交。

在我们的用例中，Kafka Streams 应用程序将执行以下操作：

+   对于接收到的应用程序事件，从信用记录数据库中查找信用历史。

+   使用从 Kafka 接收的数据和从信用记录数据库中提取的数据创建 ML 请求体

+   向风险评分生成器应用程序发出 REST 调用

+   形成最终输出记录

+   使用汇处理器将最终输出记录发送到汇点主题

首先，我们需要创建一个 Spring Boot Maven 项目并添加所需的 Maven 依赖项。以下 Spring Maven 依赖项应添加到`pom.xml`文件中，如下所示：

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-jdbc -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

除了这些之外，由于我们计划开发 Kafka 流应用程序，我们还需要添加 Kafka 相关的 Maven 依赖项，如下所示：

```java
<!-- Kafka dependencies -->
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <version>2.6.2</version>
</dependency>
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka-test</artifactId>
    <version>2.6.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams</artifactId>
    <version>3.0.0</version>
</dependency>
```

首先，让我们编写`main`类，我们将在这里初始化 Kafka Spring Boot 应用程序。然而，在我们的应用程序中，我们必须排除`KafkaAutoConfiguration`（因为我们打算使用自己的属性名来表示与 Kafka 相关的字段，而不是 Spring Boot 的默认 Kafka 属性名），如下所示：

```java
@SpringBootApplication(exclude = KafkaAutoConfiguration.class)
@Configuration
public class CreditRiskCalculatorApp {
    public static void main(String[] args) {
        SpringApplication.run(CreditRiskCalculatorApp.class);
    }
. . .
}
```

在创建`main`类之后，我们将创建主要的`KafkaStreamConfiguration`类，其中将定义和实例化所有流化 bean。这是我们使用 Kafka Streams DSL 构建拓扑的地方。此类必须使用以下代码片段中的`@EnableKafka`和`@EnableKafkaStreams`进行注解：

```java
@Configuration
@EnableKafka
@EnableKafkaStreams
public class KStreamConfiguration {
...
```

接下来，我们将创建`KafkaStreamsConfiguration` bean。以下代码片段显示了`KafkaStreamsConfiguration` bean 的实现：

```java
@Bean(name = KafkaStreamsDefaultConfiguration.DEFAULT_STREAMS_CONFIG_BEAN_NAME)
public KafkaStreamsConfiguration kStreamsConfig(){
    Map<String,Object> props = new HashMap<>();
    props.put(StreamsConfig.APPLICATION_ID_CONFIG,appId);
    props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG,bootstrapServer);
    props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
    props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG,Serdes.String().getClass());
    props.put(StreamsConfig.DEFAULT_DESERIALIZATION_EXCEPTION_HANDLER_CLASS_CONFIG, LogAndContinueExceptionHandler.class);
    return new KafkaStreamsConfiguration(props);
}
```

在创建 `KafkaStreamsConfiguration` 对象时，我们必须传递所有与 Kafka 流相关的属性。在这里，设置 `StreamsConfig.APPLICATION_ID` 和 `StreamsConfig.BOOTSTRAP_SERVERS_CONFIG` 是强制性的。在这种情况下，`StreamsConfig.APPLICATION_ID` 对应于 Kafka Streams 应用程序的消费者组 ID，而 `StreamsConfig.BOOTSTRAP_SERVERS_CONFIG` 对应于 Kafka 代理地址。没有这些值，无法运行或连接到 Kafka 集群的 Kafka 流或消费者应用程序。Kafka Streams 应用程序可以在具有相同消费者组 ID 的多个消费者之间分配来自消费者组内主题的流量。通过使用相同的 ID 增加运行实例的数量，我们可以获得更多的并行性和更好的吞吐量。然而，将实例数量增加到 Kafka 主题分区数量以上将不会对吞吐量产生任何影响。

现在我们已经创建了 `KafkaStreamsConfiguration` 对象，让我们创建 `KStream`。在创建此 `KStream` 对象时，我们必须定义拓扑。以下代码创建了 `KStream` 对象：

```java
@Bean
public KStream<String,String> kStream(StreamsBuilder builder){
    KStream<String,String> kStream = builder.stream(inputTopic);
    kStream.transform(()->new RiskCalculateTransformer (jdbcTemplate,restTemplate,mlRequestUrl)).to(outTopic);
    return kStream;
}
```

Kafka 主题中的每条消息都由一个键和一个值组成。值包含实际的消息，而键有助于在消息发布时确定分区。然而，当我们使用流来消费消息时，我们必须提到我们期望的键和值的类型。在我们的情况下，我们期望键和值都是 `String` 类型。因此，`KStream` 对象被创建为一个 `KStream<String,String>` 的实例。首先，我们必须使用 `StreamsBuilder` 类创建一个流，它是 Kafka Streams API 的一部分。在我们的用例中，拓扑构建如下：

1.  首先，使用 `StreamsBuilder` API，从 `inputTopic` 创建输入流。

1.  使用 `transform()` DSL 函数将转换处理器应用于结果输入流。

1.  一个名为 `RiskCalculatorTransformer` 的自定义 Transformer 用于转换/处理来自输入流的 数据。

1.  处理后的输出事件被写入 `outputTopic`。

现在，让我们学习如何为 Kafka Streams 应用程序编写自定义 Transformer。在我们的场景中，我们创建了 `RiskCalculatorTransformer`。以下讨论解释了如何开发自定义 Transformer：

首先，我们必须创建一个实现 `org.apache.kafka.streams.kstream.Transformer` 接口的类。它有三个方法——`init`、`transform` 和 `close`——需要实现。以下代码显示了 `Transformer` 接口的定义：

```java
public interface Transformer<K, V, R> {
    void init(ProcessorContext var1);
    R transform(K var1, V var2);
    void close();
}
```

如您所见，`Transformer` 接口期望三个泛型类型 – `K`、`V` 和 `R`。`K` 指定消息键的数据类型，`V` 指定消息值的数据类型，`R` 指定消息结果的数据类型。而 `init` 和 `close` 只在需要在进行消息处理之前进行预处理或后处理时使用，`transform` 是一个强制方法，它定义了实际的转换或处理逻辑。

在我们的用例中，我们以 JSON 字符串的形式接收消息的值，处理它，添加风险评分，并以 JSON 字符串的形式发送结果值。键的数据类型保持不变。因此，我们发送一个 `KeyValue` 对象作为结果。我们的最终 `Transformer` 概述如下：

```java
public class RiskCalculateTransformer implements Transformer<String,String, KeyValue<String,String>> {

    @Override
    public void init(ProcessorContext processorContext) {
     ...
    }
    @Override
    public KeyValue<String, String> transform(String key, String value) {
        ...
    }

    @Override
    public void close() {
     ...
    }
}
```

如前述代码所示，我们的 Transformer 期望消息的键和值都是 `String` 类型，并返回一个键值对，其中键和值都是 `String` 类型。

在我们的 Transformer 中，我们不需要任何预处理或后处理。那么，让我们继续讨论如何实现我们的 `Transformer` 的 `transform` 方法。`transform` 方法的代码如下：

```java
@Override
public KeyValue<String, String> transform(String key, String value) {
    try {
        ApplicationEvent event = mapper.readValue(value,ApplicationEvent.class);
        List<CreditRecord> creditRecord = jdbcTemplate.query(String.format("select months_balance,status from chapter6.creditrecord where id='%s'",event.getId()),new BeanPropertyRowMapper<CreditRecord>(CreditRecord.class));
        MLRequest mlRequest = new MLRequest();
        mlRequest.setAmtIncomeTotal(event.getAmtIncomeTotal());
        ...
        HttpEntity<MLRequest> request = new HttpEntity<>(mlRequest);
        ResponseEntity<RiskScoreResponse> response = restTemplate.exchange(mlRequestUrl, HttpMethod.POST, request, RiskScoreResponse.class);
        if(response.getStatusCode()== HttpStatus.OK){
            EnrichedApplication enrichedApplicationEvent = new EnrichedApplication();
            enrichedApplicationEvent.setApplicationforEnrichedApplication(event);
            enrichedApplicationEvent.setRiskScore(response.getBody().getScore());
            return KeyValue.pair(key,mapper.writeValueAsString(enrichedApplicationEvent));
        }else{
            throw new Exception("Unable to generate risk score.Risk REST response - "+ response.getStatusCode());
        }
    } catch (Exception e) {
        ...
    }
    return null;
}
```

下面是实现我们的 `transform` 方法的逐步指南：

1.  首先，我们使用 Jackson 的 `ObjectMapper` 类将传入的值（一个 JSON 字符串）反序列化为一个名为 `ApplicationEvent` 的 POJO。

1.  然后，我们使用 Spring 的 `JdbcTemplate` 初始化一个对信用记录数据库的 JDBC 调用。在构建 SQL 时，我们使用之前步骤中反序列化的应用程序 ID。由于 JDBC 调用，我们获得了一个 `CreditRecord` 对象列表。

1.  接下来，我们为即将进行的 HTTP REST 调用构建请求体。在这里，我们使用之前反序列化的 `ApplicationEvent` 对象和之前步骤中获得的 `CreditRecord` 对象列表填充一个 `MLRequest` 对象。

1.  然后，我们将 `MLRequest` 对象包装在一个 `HTTPEntity` 对象中，并使用 Spring 的 `RestTemplate` API 进行 REST 调用。

1.  我们将 REST 响应反序列化为 `RiskScoreResponse` 对象。`RiskScoreResponse` 对象的模型如下所示：

    ```java
    public class RiskScoreResponse {
        private int score;
        public int getScore() {
            return score;
        }
        public void setScore(int score) {
            this.score = score;
        }
    }
    ```

1.  如果 REST 响应是 `OK`，则使用 `ApplicationEvent` 和 `RiskScoreResponse` 对象构建 `EnrichedApplication` 对象。

1.  最后，我们创建并返回一个新的 `KeyValue` 对象，其中键保持不变，但值是我们创建的 `EnrichedApplication` 对象的序列化字符串，该对象是在 *步骤 6* 中创建的。

1.  对于异常处理，我们将任何错误记录下来，并将错误事件发送到错误数据库以供未来分析、报告和调整。报告和调整过程在此不涉及，通常由某种批量编程完成。

在本节中，我们学习了如何从头开始开发一个 Kafka Streams 应用程序。然而，我们应该能够成功地对流式应用程序进行单元测试，以确保我们的预期功能运行良好。在下一节中，我们将学习如何对 Kafka Streams 应用程序进行单元测试。

## Kafka Streams 应用程序的单元测试

要对 Kafka Streams 应用程序进行单元测试，我们必须将 Kafka Streams 测试实用程序依赖项添加到 `pom.xml` 文件中：

```java
<!-- test dependencies -->
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams-test-utils</artifactId>
    <version>3.0.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest-core</artifactId>
    <version>1.3</version>
    <scope>test</scope>
</dependency>
```

此外，在我们进行 JUnit 测试之前，我们需要对我们的代码进行一点重构。我们必须将 `KStream` bean 的定义拆分为两个方法，如下所示：

```java
@Bean
public KStream<String,String> kStream(StreamsBuilder builder){
    KStream<String, String> kStream = StreamBuilder.INSTANCE.getkStream(builder,inputTopic,outTopic,mlRequestUrl,jdbcTemplate,restTemplate);
    return kStream;
}
...
public enum StreamBuilder {
    INSTANCE;
    public KStream<String, String> getkStream(StreamsBuilder builder, String inputTopic,String outTopic, String mlRequestUrl, JdbcTemplate jdbcTemplate, RestTemplate restTemplate) {
        KStream<String,String> kStream = builder.stream(inputTopic);
        kStream.transform(()->new RiskCalculateTransformer (jdbcTemplate,restTemplate,mlRequestUrl)).to(outTopic);
        return kStream;
    }
}
```

如前述代码所示，我们将 `KStream` 形成代码提取出来，放入一个名为 `StreamBuilder` 的单例类中的工具方法中，并在其上方使用 `Bean` 方法作为包装器。

现在，让我们学习如何编写 JUnit 测试用例。首先，我们的转换需要 JDBC 调用和 REST 调用。要做到这一点，我们需要模拟 JDBC 调用。为此，我们将使用 Mockito 库。

我们可以这样模拟我们的 `JdbcTemplate` 调用：

```java
@Mock
JdbcTemplate jdbcTemplate;
...
public void creditRiskStreams(){
   ...
    List<CreditRecord> creditRecords = new ArrayList<>();
    CreditRecord creditRecord = new CreditRecord();
    . . .
    creditRecords.add(creditRecord);
    Mockito. lenient().when(jdbcTemplate.query("select months_balance,status from chapter6.creditrecord where id='5008804'",new BeanPropertyRowMapper<CreditRecord>(CreditRecord.class)))
            .thenReturn(creditRecords);
      ...
```

首先，我们使用 `@Mock` 注解创建一个模拟的 `JdbcTemplate` 对象。然后，我们使用 Mockito 的 `when().thenReturn()` API 定义一个模拟输出，用于通过模拟的 `JdbcTemplate` 对象进行的调用。

可以使用类似的技术来模拟 `RestTemplate`。模拟 `RestTemplate` 的代码如下：

```java
@Mock
private RestTemplate restTemplate;
public void creditRiskStreams(){
    ...
    RiskScoreResponse riskScoreResponse = new RiskScoreResponse();
    ...
    Mockito
            .when(restTemplate.exchange(Mockito.anyString(), HttpMethod.POST, Mockito.any(), RiskScoreResponse.class))
      .thenReturn(new ResponseEntity(riskScoreResponse, HttpStatus.OK));
```

如您所见，首先，我们使用 `@Mock` 注解模拟 `RestTemplate`。然后，使用 Mockito API 模拟任何返回 `RiskScoreResponse` 对象的 `POST` 调用。

现在，让我们构建拓扑。您可以使用以下代码创建拓扑：

```java
@Test
public void creditRiskStreamsTest() throws JsonProcessingException {
    //test input and outputTopic
    String inputTopicName = "testInputTopic";
    String outputTopicName = "testOutputTopic";
    ...
   StreamsBuilder builder = new StreamsBuilder();
StreamBuilder.INSTANCE.getkStream(builder,inputTopicName,outputTopicName,"any url",jdbcTemplate,restTemplate);
Topology testTopology = builder.build();
  ...
```

在这里，我们创建了一个 `org.apache.kafka.streams.StreamsBuilder` 类的实例。使用我们的 `StreamBuilder` 工具类，我们通过调用 `getkStream` 方法定义了拓扑。最后，我们通过调用 `org.apache.kafka.streams.StreamsBuilder` 类的 `build()` 方法构建了拓扑。

Kafka Streams 的测试工具包含一个名为 `Utility` 的类，称为 `TopologyTestDriver`。`TopologyTestDriver` 通过传递拓扑和配置详细信息来创建。一旦创建了 `TopologyTestDriver`，它有助于创建 `TestInputTopic` 和 `TestOutputTopic`。以下代码描述了如何实例化 `TopologyTestDriver` 并创建 `TestInputTopic` 和 `TestOutputTopic`：

```java
public class CreditRiskCalculatorTests {
    private final Properties config;
public CreditRiskCalculatorTests() {
    config = new Properties();
    config.setProperty(StreamsConfig.APPLICATION_ID_CONFIG, "testApp");
    config.setProperty(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "test:1234");
    config.setProperty(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
    config.setProperty(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
}
. . .
@Test
public void creditRiskStreamsTest() throws JsonProcessingException {
. . .
TopologyTestDriver testDriver = new TopologyTestDriver(testTopology,config);
TestInputTopic<String,String> inputTopic = testDriver.createInputTopic(inputTopicName, Serdes.String().serializer(), Serdes.String().serializer());
TestOutputTopic<String,String> outputTopic = testDriver.createOutputTopic(outputTopicName, Serdes.String().deserializer(), Serdes.String().deserializer());
. . .
```

要创建一个 `TestInputTopic`，我们需要指定主题名称，以及键和值序列化器。同样，`TestOutputTopic` 需要键和值反序列化器，以及输出主题名称。我们可以使用以下代码将测试事件推送到 `TestInputTopic`：

```java
inputTopic.pipeInput(inputPayload);
```

最后，我们可以使用 `org.junit.Assert.assertEquals` 静态方法，通过以下方式对我们的预期结果和实际结果进行断言：

```java
assertEquals(mapper.readTree(outputTopic.readValue()), mapper.readTree("{ \"id\": \"5008804\", \"genderCode\": \"M\", \"flagOwnCar\": \"Y\", \"flagOwnRealty\": \"Y\", \"cntChildren\": 0, \"amtIncomeTotal\": 427500.0, \"nameIncomeType\": \"Working\", \"nameEducationType\": \"Higher education\", \"nameFamilyStatus\": \"Civil marriage\", \"nameHousingType\": \"Rented apartment\", \"daysBirth\": -12005, \"daysEmployed\": -4542, \"flagMobil\": 1, \"flagWorkPhone\": 1, \"flagPhone\": 0, \"flagEmail\": 0, \"occupationType\": \"\", \"cntFamMembers\": 2 , \"riskScore\": 3.0}"));
```

我们可以通过右键单击并运行 `Test` 类来运行此 JUnit 测试，如下面的截图所示：

![图 6.3 – 运行 Kafka Streams JUnit 测试用例](img/B17084_06_003.jpg)

图 6.3 – 运行 Kafka Streams JUnit 测试用例

一旦你运行了 JUnit 测试用例，你将在 IntelliJ IDE 的运行窗口中看到测试结果，如下面的截图所示：

![图 6.4 – 验证 JUnit 测试的结果](img/B17084_06_004.jpg)

图 6.4 – 验证 JUnit 测试的结果

在本节中，我们学习了如何为 Kafka 流式应用程序编写 JUnit 测试用例并对我们的 Streams 应用程序进行单元测试。在下一节中，我们将学习如何配置流式应用程序并在我们的本地系统上运行应用程序。

## 配置和运行应用程序

要运行此应用程序，我们必须配置 `application.yaml` 文件，其中包含以下详细信息：

+   应用程序端口号（因为我们将在本地机器上启动两个 Spring Boot 应用程序）

+   数据源细节

+   Kafka 细节，如引导服务器和主题

+   风险评分生成器应用程序的 REST HTTP URL

我们的示例 `application.yaml` 文件将如下所示：

```java
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/database
    username: postgres
    driverClassName: org.postgresql.Driver
riskcalc:
  bootstrap-servers: localhost:9092
  appId: groupId1
  inputTopic: landingTopic1
  outTopic: enrichedTopic1
  mlRequestUrl: "http://localhost:8081/riskgenerate/score"
```

现在，我们可以通过运行 CreditRiskCalculator 应用程序的 main 类 `CreditRiskCalculatorApp` 来运行应用程序。但在我们启动 CreditRiskCalculator 应用程序之前，我们应该通过运行其 main 类 – 即 `RiskScoreGenerator` 来运行 RiskScoreGenerator 应用程序。这两个应用程序都是 Spring Boot 应用程序；请参阅 *第四章* 的 *实现和单元测试解决方案* 部分，*ETL 数据加载 – 数据仓库中数据摄取的基于批处理解决方案*，以了解如何运行 Spring Boot 应用程序。

故障排除技巧

如果在启动 CreditRiskCalculator 应用程序时，你在日志中注意到类似于 **无法建立到节点 -1 (localhost/127.0.0.1:9092) 的连接。代理可能不可用** 的警告消息，请确保你的 Kafka 服务器是可访问的并且正在运行。

如果你注意到一个异常，例如 `max.poll.interval.ms` 或减少 `max.poll.records` 的值。这通常发生在检索的记录数量处理时间超过配置的最大检索间隔时间时。

如果你遇到一个错误，例如 **java.lang.IllegalArgumentException: 分区已分配**。

`application.id`。将你的 `application.id` 更改以解决这个问题。

在本节中，我们学习了如何创建和单元测试 Kafka Streams 应用程序。该应用程序的源代码可在 GitHub 上找到，地址为 [`github.com/PacktPublishing/Scalable-Data-Architecture-with-Java/tree/main/Chapter06/sourcecode/CreditRiskCalculator`](https://github.com/PacktPublishing/Scalable-Data-Architecture-with-Java/tree/main/Chapter06/sourcecode/CreditRiskCalculator)。

由于实现基于机器学习的风险分数生成器应用程序超出了本书的范围，我们已创建了一个 Spring Boot REST 应用程序，该应用程序生成介于 1 到 100 之间的虚拟风险分数。此虚拟应用程序的代码库可在 GitHub 上找到，地址为 [`github.com/PacktPublishing/Scalable-Data-Architecture-with-Java/tree/main/Chapter06/sourcecode/RiskScoreGenerator`](https://github.com/PacktPublishing/Scalable-Data-Architecture-with-Java/tree/main/Chapter06/sourcecode/RiskScoreGenerator)。

在现实世界的场景中，基于机器学习的应用程序更有可能用 Python 编写而不是 Java，因为 Python 对 AI/ML 库的支持更好。然而，Kafka Streams 应用程序将能够发出 REST 调用并从该应用程序获取之前展示的生成风险分数，如图所示。

到目前为止，我们已经从输入 Kafka 主题接收事件，即时处理它，生成风险分数，并将丰富的事件写入输出 Kafka 主题。在下一节中，我们将学习如何将 Kafka 与 MongoDB 集成，并将事件流式传输到 MongoDB，一旦它们在输出 Kafka 主题中发布。

## 创建 MongoDB Atlas 云实例和数据库

在本节中，我们将使用 MongoDB Atlas 创建一个基于云的实例。按照以下步骤设置 MongoDB 云实例：

1.  如果您尚未注册，请注册 MongoDB Atlas 账户（[`www.mongodb.com/atlas/database`](https://www.mongodb.com/atlas/database)）。在注册过程中，您将被要求选择所需的订阅类型。对于这个练习，您可以选择共享订阅，它是免费的，并选择 AWS 集群作为您首选的云服务。

1.  您将看到以下屏幕。在这里，点击**构建数据库**按钮以创建新的数据库实例：

![图 6.5 – MongoDB Atlas 欢迎屏幕](img/B17084_06_005.png)

图 6.5 – MongoDB Atlas 欢迎屏幕

1.  要配置新的数据库，我们将被要求设置用户名和密码，如下面的截图所示：

![图 6.6 – 配置新的数据库实例](img/B17084_06_006.jpg)

图 6.6 – 配置新的数据库实例

1.  然后，我们将被要求输入所有我们希望授予 MongoDB 实例访问权限的 IP 地址。在这里，由于我们将从我们的本地系统运行应用程序，我们将添加我们的本地 IP 地址到 IP 访问列表。然后，点击**完成并关闭**：

![图 6.7 – 在数据库配置期间设置 IP 访问列表](img/B17084_06_007.jpg)

图 6.7 – 在数据库配置期间设置 IP 访问列表

1.  一旦集群创建完成，我们将在仪表板上看到集群，如下面的截图所示。现在，点击**浏览集合**按钮以查看此数据库实例中的集合和数据：

![图 6.8 – MongoDB Atlas 的集群仪表板](img/B17084_06_008.jpg)

图 6.8 – MongoDB Atlas 的集群仪表板

1.  如以下截图所示，目前没有任何集合或数据。然而，您可以通过点击**添加我的数据**按钮在此界面使用时手动创建集合或数据：

![图 6.9 – 在 MongoDB 数据库实例中探索集合和数据](img/B17084_06_009.jpg)

图 6.9 – 在 MongoDB 数据库实例中探索集合和数据

在本节中，我们学习了如何使用 MongoDB Atlas 的在线界面创建基于云的 MongoDB 实例。在下一节中，我们将学习如何配置和部署我们的 MongoDB Kafka 连接器，以实时将数据从 Kafka 发送到 MongoDB。

## 配置 Kafka Connect 以将结果存储在 MongoDB 中

Kafka Connect 是一个开源的、可插拔的数据集成框架，用于 Kafka。它使得数据源和数据汇能够轻松地连接到 Kafka。无需编写繁琐的代码来从数据源发布消息或从 Kafka 消费消息写入数据汇，Kafka Connect 提供了声明性配置来连接到数据源或汇。

Kafka Connect 集群已经预装了一些类型的连接器，例如`FileSourceConnector`。然而，我们可以通过将它们放置在`plugins`文件夹中来安装任何可用的连接器。对于我们的用例，我们将部署 MongoDB 连接器插件（本章后面将讨论）。

Kafka Connect 实例可以以集群模式或独立模式部署和运行。然而，在生产环境中，它通常以集群模式运行。当我们以集群模式运行时，我们可以使用 Kafka Connect 的 REST API 注册 Kafka 连接器配置。在独立模式下，我们可以在启动 Kafka Connect 实例时注册连接器配置。

由于我们将在本地机器上运行我们的 Kafka 集群，因此我们将以独立模式部署我们的 Kafka Connect 实例进行此实现。但请记住，如果您是为了生产目的而实施，您应该以集群模式运行 Kafka 以及 Kafka Connect（这可以是一个物理集群或虚拟集群，例如虚拟机、AWS ECS 或 Docker 容器）。

首先，让我们设置 Kafka Connect 集群：

1.  首先，在 Kafka 根安装文件夹下创建一个名为`plugins`的新文件夹，如图所示：

![图 6.10 – 创建插件文件夹](img/B17084_06_010.jpg)

图 6.10 – 创建插件文件夹

1.  接下来，导航到`connect-standalone.properties`，它位于`<Kafka-root>/config`文件夹中。将以下属性添加到`connect-standalone.properties`文件中：

    ```java
    .. .
    plugin.path=/<full path of Kafka installation root>/plugins
    ```

1.  然后，从[`www.confluent.io/hub/mongodb/kafka-connect-mongodb`](https://www.confluent.io/hub/mongodb/kafka-connect-mongodb)下载 MongoDB Kafka 连接器插件。

将下载一个 ZIP 文件。在 Kafka 根安装文件夹下创建的`plugin`文件夹中复制并解压该 ZIP 文件。此时，文件夹结构应如下所示：

![图 6.11 – 部署 mongo-kafka-connect 插件后的 Kafka 文件结构](img/B17084_06_011.jpg)

图 6.11 – 部署 mongo-kafka-connect 插件后的 Kafka 文件结构

现在，让我们学习如何创建和部署 Kafka Connect 工作员配置，以在 Kafka 主题和 MongoDB 溢出之间创建管道。

要编写 Kafka Connect 工作员，我们必须了解 Kafka Connect 支持的各种声明性属性类型。以下图表描述了 Kafka Connect 工作员由哪些类型的组件组成：

![图 6.12 – Kafka Connect 组件](img/B17084_06_012.jpg)

图 6.12 – Kafka Connect 组件

Kafka 连接器由三种类型的组件组成。具体如下：

+   **连接器**：此接口将 Kafka 与外部数据源连接。它负责实现数据源和连接器与 Kafka 通信所需的外部协议。

+   **转换器**：转换器用于序列化和反序列化事件。

+   **转换器**：这是一个可选属性。它是一个无状态的函数，用于对数据进行轻微的转换，以便使其适合目标格式。

对于我们的用例，我们不需要转换器，但我们需要设置所有与连接器和转换器相关的属性。以下代码是 Kafka Sink Connect 工作员的代码：

```java
name=mongo-sink
topics=enrichedTopic1
connector.class=com.mongodb.kafka.connect.MongoSinkConnector
tasks.max=1
# converter configs
key.converter=org.apache.kafka.connect.storage.StringConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=false
value.converter.schemas.enable=false
...
```

如前述配置代码所示，配置了连接器属性，如 `connector.class` 以及特定于 MongoDB 的其他属性，并设置了转换器属性，如 `key.converter` 和 `value.converter`。接下来，在连接器配置中，我们定义了所有 MongoDB 连接属性，如下所示：

```java
# Specific global MongoDB Sink Connector configuration
connection.uri=mongodb+srv://username:password@cluster0.ipguv.mongodb.net/CRRD?retryWrites=true&w=majority
database=CRRD
collection=newloanrequest
max.num.retries=1
retries.defer.timeout=5000
```

现在，我们将在连接器配置中设置 `document.id` 和 `writemodel.strategy` 属性，如下所示：

```java
document.id.strategy=com.mongodb.kafka.connect.sink.processor.id.strategy.PartialValueStrategy
document.id.strategy.partial.value.projection.list=id
document.id.strategy.partial.value.projection.type=AllowList
writemodel.strategy=com.mongodb.kafka.connect.sink.writemodel.strategy.ReplaceOneBusinessKeyStrategy
```

将这些配置保存到名为 `connect-riskcalc-mongodb-sink.properties` 的属性文件中，并将其放置在 Kafka Connect 的 `config` 文件夹中。

现在，我们可以以独立模式运行 Kafka Connect 实例，并使用以下命令启动 `mongodb-sink` 连接器：

```java
bin/connect-standalone.sh config/connect-standalone.properties connect-riskcalc-mongodb-sink.properties
```

现在，让我们学习如何排查我们可能遇到的潜在问题。

### Kafka Sink 连接器的故障排除

当源或连接器在 Kafka Connect 集群上运行时，你可能会遇到多个问题。以下列表指定了一些常见问题及其解决方法：

+   如果你遇到以下类似的错误，请检查 JSON 消息是否应该包含模式。如果 JSON 消息不应该包含模式，请确保将 `key.converter.schemas.enable` 和 `value.converter.schemas.enable` 属性设置为 `false`：

    ```java
    org.apache.kafka.connect.errors.DataException: JsonConverter with schemas.enable requires "schema" and "payload" fields and may not contain additional fields.
    ```

+   如果您遇到类似`org.apache.kafka.common.errors.SerializationException: Error deserializing Avro message for id -1`的错误，请检查负载是否为 Avro 或 JSON。如果消息是 JSON 负载而不是 Avro，请将连接器中`value.converter`属性的值更改为`org.apache.kafka.connect.json.JsonConverter`。

+   在向 MongoDB 写入时，您可能会遇到`BulkWriteExceptions`。`BulkWriteExceptions`可能是一个`WriteError`、一个`WriteConcernError`或一个`WriteSkippedError`（由于在有序批量写入中先前的记录失败）。尽管我们无法防止此类错误，但我们可以设置以下参数将拒绝的消息移动到名为`dead-letter-queue`的错误主题：

    ```java
    errors.tolerance=all
    errors.deadletterqueue.topic.name=<name of topic to use as dead letter queue>
    errors.deadletterqueue.context.headers.enable=true
    ```

在本节中，我们成功创建、部署并运行了 MongoDB Kafka Sink 连接器。在下一节中，我们将讨论如何测试端到端解决方案。

## 验证解决方案

要测试端到端管道，我们必须确保所有服务，如 Kafka、PostgreSQL、Kafka Connect 和 MongoDB 实例，都已启动并运行。

此外，Kafka Streams 应用程序和风险评分生成器 REST 应用程序应该已经启动并运行。我们可以通过运行主 Spring Boot 应用程序类来启动这些应用程序。

要测试应用程序，打开一个新的终端并导航到 Kafka 根安装文件夹。要启动 Kafka 控制台生产者实例，请使用以下命令：

```java
bin/kafka-console-producer.sh --topic landingTopic1 --bootstrap-server localhost:9092
```

然后，我们可以使用控制台生产者发布输入消息，如下面的截图所示：

![图 6.13 – 使用 Kafka 控制台生产者发布消息](img/B17084_06_013.jpg)

图 6.13 – 使用 Kafka 控制台生产者发布消息

一旦我们在输入主题中发布消息，它就会被处理，并将结果写入 MongoDB 实例。您可以在 MongoDB 中这样验证结果：

![图 6.14 – 验证结果](img/B17084_06_015.jpg)

图 6.14 – 验证结果

在本节中，我们学习了如何测试实时数据处理问题的端到端解决方案并验证结果。现在，让我们总结本章所学的内容。

# 摘要

在本章中，我们讨论了如何分析实时数据工程问题，确定流平台，并考虑我们的解决方案必须具备的基本特征以成为有效的实时解决方案。首先，我们学习了如何选择一个混合平台以满足法律需求以及性能和成本效益。

然后，我们学习了如何利用我们的问题分析结论来构建一个稳健、可靠和有效的实时数据工程解决方案。之后，我们学习了如何在本地机器上安装和运行 Apache Kafka 并在那个 Kafka 集群中创建主题。我们还学习了如何开发 Kafka Streams 应用程序来进行流处理并将结果写入输出主题。接着，我们学习了如何对 Kafka Streams 应用程序进行单元测试，以使代码更加稳健且无缺陷。之后，我们学习了如何在 AWS 云上设置 MongoDB Atlas 实例。最后，我们了解了 Kafka Connect 以及如何配置和使用 Kafka MongoDB Sink 连接器，将处理后的事件从输出主题发送到 MongoDB 集群。在这个过程中，我们还学习了如何测试和验证我们所开发的实时数据工程解决方案。

通过这样，我们已经学会了如何为基于批处理和实时数据工程问题开发优化且成本效益高的解决方案。在下一章中，我们将学习在数据摄取或分析问题中常用的一些架构模式。
