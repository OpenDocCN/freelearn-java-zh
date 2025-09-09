# 第五章：Schema Registry

在上一章中，我们看到了如何以 JSON 格式生产和消费数据。在本章中，我们将了解如何使用 Apache Avro 序列化相同的消息。

本章涵盖了以下主题：

+   Avro 简要介绍

+   定义模式

+   启动 Schema Registry

+   使用 Schema Registry

+   如何构建 Java `AvroProducer`、消费者和处理器

+   如何运行 Java `AvroProducer` 和处理器

# Avro 简要介绍

Apache Avro 是一种二进制序列化格式。该格式基于模式，因此它依赖于 JSON 格式中的模式定义。这些模式定义了哪些字段是必需的以及它们的类型。Avro 还支持数组、枚举和嵌套字段。

Avro 的一个主要优点是它支持模式演进。这样，我们可以拥有几个模式的历史版本。

通常，系统必须适应业务的变化需求。因此，我们可以向我们的实体添加或删除字段，甚至更改数据类型。为了支持向前或向后兼容性，我们必须考虑哪些字段被标记为可选的。

因为 Avro 将数据转换为字节数组（序列化），而 Kafka 的消息也是以二进制数据格式发送的，所以使用 Apache Kafka，我们可以以 Avro 格式发送消息。真正的问题是，我们将在哪里存储 Apache Avro 的工作模式？

回想一下，企业服务总线的主要功能之一是验证它所处理的消息的格式，如果它有这些格式的历史记录会更好。

Kafka Schema Registry 是负责执行重要功能的模块。第一个功能是验证消息是否处于适当的格式，第二个功能是拥有这些模式的存储库，第三个功能是拥有这些模式的历史版本格式。

Schema Registry 是一个与我们的 Kafka 代理在同一位置运行的服务器。它运行并存储模式，包括模式版本。当以 Avro 格式将消息发送到 Kafka 时，消息包含一个存储在 Schema Registry 中的模式标识符。

有一个库允许在 Avro 格式中进行消息序列化和反序列化。这个库与 Schema Registry 透明且自然地协同工作。

当以 Avro 格式发送消息时，序列化器确保已注册模式并获取模式 ID。如果我们发送一个不在 Schema Registry 中的 Avro 消息，当前版本的模式将自动在注册表中注册。如果您不希望 Schema Registry 以这种方式操作，可以通过将 `auto.register.schemas` 标志设置为 `false` 来禁用它。

当接收到 Avro 格式的消息时，反序列化器试图在注册表中找到模式 ID 并获取模式以反序列化 Avro 格式的消息。

模式注册表以及用于 Avro 格式消息序列化和反序列化的库都在 Confluent 平台下。重要的是要提到，当您需要使用模式注册表时，您必须使用 Confluent 平台。

还很重要的一点是，使用模式注册表时，应该使用 Confluent 库进行 Avro 格式的序列化，因为 Apache Avro 库不起作用。

# 定义模式

第一步是定义 Avro 模式。作为提醒，我们的`HealthCheck`类看起来像*列表 5.1*：

```java
public final class HealthCheck {
 private String event;
 private String factory;
 private String serialNumber;
 private String type;
 private String status;
 private Date lastStartedAt;
 private float temperature;
 private String ipAddress;
}
```

列表 5.1: HealthCheck.java

现在，有了这个消息的 Avro 格式表示，所有此类消息的 Avro 模式（即模板）将是*列表 5.2*：

```java
{
 "name": "HealthCheck",
 "namespace": "kioto.avro",
 "type": "record",
 "fields": [
 { "name": "event", "type": "string" },
 { "name": "factory", "type": "string" },
 { "name": "serialNumber", "type": "string" },
 { "name": "type", "type": "string" },
 { "name": "status", "type": "string"},
 { "name": "lastStartedAt", "type": "long", "logicalType": "timestamp-
    millis"},
 { "name": "temperature", "type": "float" },
 { "name": "ipAddress", "type": "string" }
 ]
}
```

列表 5.2: healthcheck.avsc

此文件必须保存在`kioto`项目的`src/main/resources`目录下。

重要的是要注意，有`string`、`float`和`double`这些类型。但是，对于`Date`，它可以存储为`long`或`string`。

对于这个例子，我们将`Date`序列化为`long`。Avro 没有专门的`Date`类型；我们必须在`long`和`string`（通常是 ISO-8601 的`string`）之间选择，但这个例子中的重点是展示如何使用不同的数据类型。

更多关于 Avro 模式和如何映射类型的信息，请查看以下 URL 的 Apache Avro 规范：

[`avro.apache.org/docs/current/spec.html`](http://avro.apache.org/docs/current/spec.html)

# 启动模式注册表

好吧，我们已经有了我们的 Avro 模式；现在，我们需要在模式注册表中注册它。

当我们启动 Confluent 平台时，模式注册表也会启动，如下面的代码所示：

```java
$./bin/confluent start
Starting zookeeper
zookeeper is [UP]
Starting kafka
kafka is [UP]
Starting schema-registry
schema-registry is [UP]
Starting kafka-rest
kafka-rest is [UP]
Starting connect
connect is [UP]
Starting ksql-server
ksql-server is [UP]
Starting control-center
control-center is [UP]
```

如果我们只想启动模式注册表，我们需要运行以下命令：

```java
$./bin/schema-registry-start etc/schema-registry/schema-registry.properties
```

输出类似于这里所示：

```java
...
[2017-03-02 10:01:45,320] INFO Started NetworkTrafficServerConnector@2ee67803{HTTP/1.1,[http/1.1]}{0.0.0.0:8081}
```

# 使用模式注册表

现在，模式注册表正在端口`8081`上运行。要与模式注册表交互，有一个 REST API。我们可以使用`curl`来访问它。第一步是在模式注册表中注册一个模式。为此，我们必须将我们的 JSON 模式嵌入到另一个 JSON 对象中，并且必须转义一些特殊字符并添加一个有效负载：

+   在开始时，我们必须添加`{ \"schema\": \"`

+   所有双引号（`"`）都应该用反斜杠（`\"`）转义

+   最后，我们必须添加`\" }`

是的，正如你可以猜到的，API 有几个命令来查询模式注册表。

# 在一个值主题下注册模式的新版本

要使用`curl`命令注册位于`src/main/resources/`路径中，如*列表 5.2*中列出的`healthcheck.avsc`Avro 模式，我们使用以下命令：

```java
$ curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" \
--data '{ "schema": "{ \"name\": \"HealthCheck\", \"namespace\": \"kioto.avro\", \"type\": \"record\", \"fields\": [ { \"name\": \"event\", \"type\": \"string\" }, { \"name\": \"factory\", \"type\": \"string\" }, { \"name\": \"serialNumber\", \"type\": \"string\" }, { \"name\": \"type\", \"type\": \"string\" }, { \"name\": \"status\", \"type\": \"string\"}, { \"name\": \"lastStartedAt\", \"type\": \"long\", \"logicalType\": \"timestamp-millis\"}, { \"name\": \"temperature\", \"type\": \"float\" }, { \"name\": \"ipAddress\", \"type\": \"string\" } ]} " }' \
http://localhost:8081/subjects/healthchecks-avro-value/versions
```

输出应该是这样的：

```java
{"id":1}
```

这意味着我们已经将`HealthChecks`模式注册为版本`"id":1`（恭喜，这是您的第一个版本）。

注意，该命令在名为`healthchecks-avro-value`的主题上注册模式。Schema Registry 没有关于主题的信息（我们还没有创建`healthchecks-avro`主题）。序列化器/反序列化器遵循一个惯例，即在遵循<主题>-value 格式的名称下注册模式。在这种情况下，由于该模式用于消息值，我们使用后缀-value。如果我们想使用 Avro 来标识我们的消息键，我们将使用<主题>-key 格式。

例如，要获取我们模式的 ID，我们使用以下命令：

```java
$ curl http://localhost:8081/subjects/healthchecks-avro-value/versions/
```

以下输出是模式 ID：

```java
[1]
```

使用模式 ID，为了检查我们模式的值，我们使用以下命令：

```java
$ curl http://localhost:8081/subjects/healthchecks-avro-value/versions/1
```

输出是这里显示的模式值：

```java
{"subject":"healthchecks-avro-value","version":1,"id":1,"schema":"{\"type\":\"record\",\"name\":\"HealthCheck\",\"namespace\":\"kioto.avro\",\"fields\":[{\"name\":\"event\",\"type\":\"string\"},{\"name\":\"factory\",\"type\":\"string\"},{\"name\":\"serialNumber\",\"type\":\"string\"},{\"name\":\"type\",\"type\":\"string\"},{\"name\":\"status\",\"type\":\"string\"},{\"name\":\"lastStartedAt\",\"type\":\"long\",\"logicalType\":\"timestamp-millis\"},{\"name\":\"temperature\",\"type\":\"float\"},{\"name\":\"ipAddress\",\"type\":\"string\"}]}"}
```

# 在–key 主题下注册模式的新的版本

例如，要将我们模式的新的版本注册到`healthchecks-avro-key`主题下，我们将执行以下命令（不要运行它；这只是为了举例）：

```java
curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json"\
--data 'our escaped avro data' \
http://localhost:8081/subjects/healthchecks-avro-key/versions
```

输出应该是这样的：

```java
{"id":1}
```

# 将现有模式注册到新主题

假设有一个名为`healthchecks-value1`的主题上已注册的模式，我们需要这个模式在名为`healthchecks-value2`的主题上可用。

以下命令从`healthchecks-value1`读取现有模式并将其注册到`healthchecks-value2`（假设`jq`工具已经安装）：

```java
curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json"\
--data "{\"schema\": $(curl -s http://localhost:8081/subjects/healthchecks-value1/versions/latest | jq '.schema')}" \
http://localhost:8081/subjects/healthchecks-value2/versions
```

输出应该是这样的：

```java
{"id":1}
```

# 列出所有主题

要列出所有主题，你可以使用以下命令：

```java
curl -X GET http://localhost:8081/subjects
```

输出应该是这样的：

```java
["healthcheck-avro-value","healthchecks-avro-key"]
```

# 通过其全局唯一 ID 获取模式

获取模式，你可以使用以下命令：

```java
curl -X GET http://localhost:8081/schemas/ids/1
```

输出应该是这样的：

```java
{"schema":"{\"type\":\"record\",\"name\":\"HealthCheck\",\"namespace\":\"kioto.avro\",\"fields\":[{\"name\":\"event\",\"type\":\"string\"},{\"name\":\"factory\",\"type\":\"string\"},{\"name\":\"serialNumber\",\"type\":\"string\"},{\"name\":\"type\",\"type\":\"string\"},{\"name\":\"status\",\"type\":\"string\"},{\"name\":\"lastStartedAt\",\"type\":\"long\",\"logicalType\":\"timestamp-millis\"},{\"name\":\"temperature\",\"type\":\"float\"},{\"name\":\"ipAddress\",\"type\":\"string\"}]}"}
```

# 列出在`healthchecks–value`主题下注册的所有模式版本

要列出在`healthchecks-value`主题下注册的所有模式版本，你可以使用以下命令：

```java
curl -X GET http://localhost:8081/subjects/healthchecks-value/versions
```

输出应该是这样的：

```java
[1]
```

# 获取在`healthchecks-value`主题下注册的模式的版本 1

要获取在`healthchecks-value`主题下注册的模式的版本 1，你可以使用以下命令：

```java
curl -X GET http://localhost:8081/subjects/ healthchecks-value/versions/1
```

输出应该是这样的：

```java
{"subject":" healthchecks-value","version":1,"id":1}
```

# 删除在`healthchecks-value`主题下注册的模式的版本 1

要删除在`healthchecks-value`主题下注册的模式的版本 1，你可以使用以下命令：

```java
curl -X DELETE http://localhost:8081/subjects/healthchecks-value/versions/1
```

输出应该是这样的：

```java
1
```

# 删除在`healthchecks-value`主题下最近注册的模式

要删除在`healthchecks-value`主题下最近注册的模式，你可以使用以下命令：

```java
curl -X DELETE http://localhost:8081/subjects/healthchecks-value/versions/latest
```

输出应该是这样的：

```java
2
```

# 删除在`healthchecks–value`主题下注册的所有模式版本

要删除在`healthchecks-value`主题下注册的所有模式版本，你可以使用以下命令：

```java
curl -X DELETE http://localhost:8081/subjects/healthchecks-value
```

输出应该是这样的：

```java
[3]
```

# 检查是否在`healthchecks-key`主题下已注册该模式

要检查是否在`healthchecks-key`主题下已注册该模式，可以使用以下命令：

```java
curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json"\
--data 'our escaped avro data' \
http://localhost:8081/subjects/healthchecks-key
```

输出应该是这样的：

```java
{"subject":"healthchecks-key","version":3,"id":1}
```

# 对`healthchecks-value`主题下的最新模式进行模式兼容性测试

要对`healthchecks-value`主题下的最新模式进行模式兼容性测试，可以使用以下命令：

```java
curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json"\
--data 'our escaped avro data' \
http://localhost:8081/compatibility/subjects/healthchecks-value/versions/latest
```

输出应该是这样的：

```java
{"is_compatible":true}
```

# 获取顶级兼容性配置

要获取顶级兼容性配置，可以使用以下命令：

```java
curl -X GET http://localhost:8081/config
```

输出应该是这样的：

```java
{"compatibilityLevel":"BACKWARD"}
```

# 全局更新兼容性要求

要全局更新兼容性要求，可以使用以下命令：

```java
curl -X PUT -H "Content-Type: application/vnd.schemaregistry.v1+json" \
--data '{"compatibility": "NONE"}' \
http://localhost:8081/config
```

输出应该是这样的：

```java
{"compatibility":"NONE"}
```

# 更新`healthchecks-value`主题下的兼容性要求

要更新`healthchecks-value`主题下的兼容性要求，可以使用以下命令：

```java
curl -X PUT -H "Content-Type: application/vnd.schemaregistry.v1+json" \
--data '{"compatibility": "BACKWARD"}' \
http://localhost:8081/config/healthchecks-value
```

输出应该是这样的：

```java
{"compatibility":"BACKWARD"}
```

# Java AvroProducer

现在，我们应该修改我们的 Java 生产者以发送 Avro 格式的消息。首先，重要的是要提到，在 Avro 中有两种类型的消息：

+   **特定记录**：包含 Avro 模式（avsc）的文件被发送到特定的 Avro 命令以生成相应的 Java 类。

+   **通用记录**：在此方法中，使用类似于映射字典的数据结构。这意味着您可以通过名称设置和获取字段，并且您必须知道它们对应的类型。此选项不是类型安全的，但它比其他选项提供了更多的灵活性，并且在这里，版本随着时间的推移更容易管理。在这个例子中，我们将使用这种方法。

在我们开始编写代码之前，请记住，在上一章中，我们向 Kafka 客户端添加了支持 Avro 的库。如果您还记得，`build.gradle`文件有一个特殊的仓库，其中包含所有这些库。

Confluent 的仓库在以下行指定：

```java
repositories {
 ...
 maven { url 'https://packages.confluent.io/maven/' }
 }
```

在依赖关系部分，我们应该添加特定的 Avro 库：

```java
dependencies {
 ...
 compile 'io.confluent:kafka-avro-serializer:5.0.0'
 }
```

不要使用 Apache Avro 提供的库，因为它们将不起作用。

如我们所知，要构建 Kafka 消息生产者，我们使用 Java 客户端库；特别是生产者 API。如我们所知，所有 Kafka 生产者都应该有两个必备条件：成为一个`KafkaProducer`并设置特定的`Properties`，例如*列表 5.3*：

```java
import io.confluent.kafka.serializers.KafkaAvroSerializer; 
import org.apache.avro.Schema;
import org.apache.avro.Schema.Parser;
import org.apache.avro.generic.GenericRecord;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.common.serialization.StringSerializer;

public final class AvroProducer {
  private final Producer<String, GenericRecord> producer; //1
  private Schema schema;

  public AvroProducer(String brokers, String schemaRegistryUrl) { //2
    Properties props = new Properties();
    props.put("bootstrap.servers", brokers);
    props.put("key.serializer", StringSerializer.class); //3
    props.put("value.serializer", KafkaAvroSerializer.class); //4
    props.put("schema.registry.url", schemaRegistryUrl) //5
    producer = new KafkaProducer<>(props);

    try {
      schema = (new Parser()).parse( new 
      File("src/main/resources/healthcheck.avsc")); //6
    } catch (IOException e) {
      // deal with the Exception
    }
  }
  ...
}
```

列表 5.3：AvroProducer 构造函数

分析`AvroProducer`构造函数显示以下内容：

+   在行`//1`中，现在的值是`org.apache.avro.generic.GenericRecord`类型

+   在行`//2`中，构造函数现在接收模式注册表 URL

+   在行`//3`中，消息键的序列化器类型仍然是`StringSerializer`

+   在行`//4`中，消息值的序列化器类型现在是`KafkaAvroSerializer`

+   在行`//5`中，将 Schema Registry URL 添加到生产者属性中

+   在行`//6`中，使用 Schema Parser 解析包含模式定义的 avsc 文件

由于我们选择了使用通用记录，我们必须加载模式。请注意，我们本可以从 Schema Registry 获取模式，但这并不安全，因为我们不知道哪个版本的模式已注册。相反，将模式与代码一起存储是一种聪明且安全的做法。这样，即使有人更改 Schema Registry 中注册的模式，我们的代码也会始终产生正确的数据类型。

现在，在`src/main/java/kioto/avro`目录下，创建一个名为`AvroProducer.java`的文件，内容为*列表 5.4*：

```java
package kioto.avro;
import ...
public final class AvroProducer {
 /* here the Constructor code in Listing 5.3 */

 public final class AvroProducer {

  private final Producer<String, GenericRecord> producer;
  private Schema schema;

  public AvroProducer(String brokers, String schemaRegistryUrl) {
    Properties props = new Properties();
    props.put("bootstrap.servers", brokers);
    props.put("key.serializer", StringSerializer.class);
    props.put("value.serializer", KafkaAvroSerializer.class);
    props.put("schema.registry.url", schemaRegistryUrl);
    producer = new KafkaProducer<>(props);
    try {
      schema = (new Parser()).parse(new                   
      File("src/main/resources/healthcheck.avsc"));
    } catch (IOException e) {
      e.printStackTrace();
    }
  }

  public final void produce(int ratePerSecond) {
    long waitTimeBetweenIterationsMs = 1000L / (long)ratePerSecond;
    Faker faker = new Faker();

    while(true) {
      HealthCheck fakeHealthCheck =
          new HealthCheck(
              "HEALTH_CHECK",
              faker.address().city(),
              faker.bothify("??##-??##", true),
              Constants.machineType.values()                                                                                                                 
              [faker.number().numberBetween(0,4)].toString(),
              Constants.machineStatus.values()                                        
              [faker.number().numberBetween(0,3)].toString(),
              faker.date().past(100, TimeUnit.DAYS),
              faker.number().numberBetween(100L, 0L),
              faker.internet().ipV4Address());
              GenericRecordBuilder recordBuilder = new                                       
              GenericRecordBuilder(schema);
              recordBuilder.set("event", fakeHealthCheck.getEvent());
              recordBuilder.set("factory", 
              fakeHealthCheck.getFactory());
              recordBuilder.set("serialNumber",                                          
              fakeHealthCheck.getSerialNumber());
              recordBuilder.set("type", fakeHealthCheck.getType());
              recordBuilder.set("status", fakeHealthCheck.getStatus());
              recordBuilder.set("lastStartedAt",                                      
              fakeHealthCheck.getLastStartedAt().getTime());
              recordBuilder.set("temperature",                                          
              fakeHealthCheck.getTemperature());
              recordBuilder.set("ipAddress",   
              fakeHealthCheck.getIpAddress());
              Record avroHealthCheck = recordBuilder.build();
              Future futureResult = producer.send(new ProducerRecord<>               
              (Constants.getHealthChecksAvroTopic(), avroHealthCheck));
      try {
        Thread.sleep(waitTimeBetweenIterationsMs);
        futureResult.get();
      } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
      }
    }
  }

  public static void main( String[] args) {
    new AvroProducer("localhost:9092",                                       
    "http://localhost:8081").produce(2);
  }
}
```

列表 5.4：AvroProducer.java

对`AvroProducer`类的分析显示以下内容：

+   在行`//1`中，`ratePerSecond`是 1 秒内要发送的消息数量

+   在行`//2`中，为了模拟重复，我们使用了一个无限循环（在生产环境中尽量避免这样做）

+   在行`//3`中，现在我们可以使用`GenericRecordBuilder`创建`GenericRecord`对象

+   在行`//4`中，我们使用 Java Future 将记录发送到`healthchecks-avro`主题

+   在行`//5`中，我们这次等待再次发送消息

+   在行`//6`中，我们读取 Future 的结果

+   在行`//7`中，一切都在本地的 9092 端口上的代理上运行，Schema Registry 也在本地的 8081 端口上运行，以 1 秒的间隔发送两条消息

# 运行 AvroProducer

要构建项目，请在`kioto`目录下运行以下命令：

```java
$ gradle jar
```

如果一切正常，输出将类似于这里所示：

```java
BUILD SUCCESSFUL in 3s
 1 actionable task: 1 executed
```

1.  如果它还没有运行，请转到 Confluent 目录并启动它：

```java
$ ./bin/confluent start
```

1.  代理正在 9092 端口上运行。要创建`healthchecks-avro`主题，执行以下命令：

```java
$ ./bin/kafka-topics --zookeeper localhost:2181 --create --topic healthchecks-avro --replication-factor 1 --partitions 4
```

1.  注意，我们只是创建了一个普通主题，没有指示消息的格式。

1.  为`healthchecks-avro`主题运行控制台消费者：

```java
$ ./bin/kafka-console-consumer --bootstrap-server localhost:9092 --topic healthchecks-avro
```

1.  从我们的 IDE 中运行`AvroProducer`的 main 方法。

1.  控制台消费者的输出应该类似于这里所示：

```java
HEALTH_CHECKLake JeromyGE50-GF78HYDROELECTRICRUNNING�����Y,B227.30.250.185
HEALTH_CHECKLockmanlandMW69-LS32GEOTHERMALRUNNING֗���YB72.194.121.48
HEALTH_CHECKEast IsidrofortIH27-WB64NUCLEARSHUTTING_DOWN�̤��YB88.136.134.241
HEALTH_CHECKSipesshireDH05-YR95HYDROELECTRICRUNNING����Y�B254.125.63.235
HEALTH_CHECKPort EmeliaportDJ83-UO93GEOTHERMALRUNNING���Y�A190.160.48.125
```

二进制格式对人类来说难以阅读，不是吗？我们只能读取字符串，但不能读取记录的其余部分。

为了解决我们的可读性问题，我们应该使用`kafka-avro-console-consumer`。这个花哨的消费者将 Avro 记录反序列化并打印成人类可读的 JSON 对象。

从命令行运行一个`healthchecks-avro`主题的 Avro 控制台消费者：

```java
$ ./bin/kafka-avro-console-consumer --bootstrap-server localhost:9092 --topic healthchecks-avro
```

控制台消费者的输出应该类似于以下内容：

```java
{"event":"HEALTH_CHECK","factory":"Lake Jeromy","serialNumber":" GE50-GF78","type":"HYDROELECTRIC","status":"RUNNING","lastStartedAt":1537320719954,"temperature":35.0,"ipAddress":"227.30.250.185"}
{"event":"HEALTH_CHECK","factory":"Lockmanland","serialNumber":" MW69-LS32","type":"GEOTHERMAL","status":"RUNNING","lastStartedAt":1534188452893,"temperature":61.0,"ipAddress":"72.194.121.48"}
{"event":"HEALTH_CHECK","factory":"East Isidrofort","serialNumber":" IH27-WB64","type":"NUCLEAR","status":"SHUTTING_DOWN","lastStartedAt":1539296403179,"temperature":62.0,"ipAddress":"88.136.134.241"}
...
```

现在，我们终于开始以 Avro 格式生产 Kafka 消息。在 Schema Registry 和 Confluent 库的帮助下，这个任务相当简单。正如描述的那样，在生产环境中经过许多挫折后，通用记录模式比特定记录模式更好，因为知道我们正在使用哪个模式来生产数据更好。将模式代码与代码一起保存可以给你这个保证。

如果我们在生产数据之前从 Schema Registry 获取模式会发生什么？正确的答案是这取决于，这取决于

`auto.register.schemas`属性。如果此属性设置为 true，当您请求 Schema Registry 中不存在的模式时，它将自动注册为新的模式（在生产环境中不建议使用此选项，因为它容易出错）。如果属性设置为 false，则不会存储模式，并且由于模式不匹配，我们将得到一个漂亮的异常（不要相信我，读者；去获取这个证明）

# Java AvroConsumer

让我们创建一个 Kafka `AvroConsumer`，我们将使用它来接收输入记录。正如我们已经知道的，所有的 Kafka Consumers 都应该有两个前提条件：成为一个`KafkaConsumer`并设置特定的属性，如*列表 5.5*所示：

```java
import io.confluent.kafka.serializers.KafkaAvroDeserializer;
import org.apache.avro.generic.GenericRecord;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.consumer.Consumer;
import org.apache.kafka.common.serialization.StringSerializer;

public final class AvroConsumer {
  private Consumer<String, GenericRecord> consumer; //1
  public AvroConsumer(String brokers, String schemaRegistryUrl) { //2
     Properties props = new Properties();
     props.put("group.id", "healthcheck-processor");
     props.put("bootstrap.servers", brokers);
     props.put("key.deserializer", StringDeserializer.class); //3
     props.put("value.deserializer", KafkaAvroDeserializer.class); //4
     props.put("schema.registry.url", schemaRegistryUrl); //5
     consumer = new KafkaConsumer<>(props); //6
  }
 ...
}
```

列表 5.5：AvroConsumer 构造函数

分析`AvroConsumer`构造函数的变化，我们可以看到以下内容：

+   在行`//1`中，值现在是`org.apache.avro.generic.GenericRecord`类型

+   在行`//2`中，构造函数现在接收 Schema Registry URL

+   在行`//3`中，消息键的反序列化类型仍然是`StringDeserializer`

+   在行`//4`中，值的反序列化类型现在是`KafkaAvroDeserializer`

+   在行`//5`中，将 Schema Registry URL 添加到消费者属性中

+   在行`//6`中，使用这些`Properties`，我们构建了一个具有字符串键和`GenericRecord`值的`KafkaConsumer`：`<String, GenericRecord>`

重要的是要注意，当为反序列化器定义 Schema Registry URL 以获取模式时，消息只包含模式 ID 而不是模式本身。

# Java AvroProcessor

现在，在`src/main/java/kioto/avro`目录下，创建一个名为`AvroProcessor.java`的文件，其内容为*列表 5.6*：

```java
package kioto.plain;
import ...
public final class AvroProcessor {
  private Consumer<String, GenericRecord> consumer;
  private Producer<String, String> producer;

  public AvroProcessor(String brokers , String schemaRegistryUrl) {
    Properties consumerProps = new Properties();
    consumerProps.put("bootstrap.servers", brokers);
    consumerProps.put("group.id", "healthcheck-processor");
    consumerProps.put("key.deserializer", StringDeserializer.class);
    consumerProps.put("value.deserializer", KafkaAvroDeserializer.class);
    consumerProps.put("schema.registry.url", schemaRegistryUrl);
    consumer = new KafkaConsumer<>(consumerProps);

    Properties producerProps = new Properties();
    producerProps.put("bootstrap.servers", brokers);
    producerProps.put("key.serializer", StringSerializer.class);
    producerProps.put("value.serializer", StringSerializer.class);
    producer = new KafkaProducer<>(producerProps);
 }
```

列表 5.6：AvroProcessor.java（第一部分）

分析`AvroProcessor`类的前一部分，我们可以看到以下内容：

+   在第一部分，我们声明了一个`AvroConsumer`，如*列表 5.5*所示

+   在第二部分，我们声明了一个`AvroProducer`，如*列表 5.4*所示

现在，在`src/main/java/kioto/avro`目录下，让我们用*列表 5.7*的内容来完成`AvroProcessor.java`文件：

```java
public final void process() {
  consumer.subscribe(Collections.singletonList(
    Constants.getHealthChecksAvroTopic())); //1
    while(true) {
      ConsumerRecords records = consumer.poll(Duration.ofSeconds(1L));
      for(Object record : records) {
        ConsumerRecord it = (ConsumerRecord) record;
        GenericRecord healthCheckAvro = (GenericRecord) it.value(); //2
        HealthCheck healthCheck = new HealthCheck ( //3
          healthCheckAvro.get("event").toString(),
          healthCheckAvro.get("factory").toString(),
          healthCheckAvro.get("serialNumber").toString(),
          healthCheckAvro.get("type").toString(),
          healthCheckAvro.get("status").toString(),
          new Date((Long)healthCheckAvro.get("lastStartedAt")),
          Float.parseFloat(healthCheckAvro.get("temperature").toString()),
          healthCheckAvro.get("ipAddress").toString());
          LocalDate startDateLocal= 
          healthCheck.getLastStartedAt().toInstant()
                      .atZone(ZoneId.systemDefault()).toLocalDate(); //4
          int uptime = Period.between(startDateLocal,     
          LocalDate.now()).getDays(); //5
          Future future =
               producer.send(new ProducerRecord<>(
                             Constants.getUptimesTopic(),
                             healthCheck.getSerialNumber(),
                             String.valueOf(uptime))); //6
          try {
            future.get();
          } catch (InterruptedException | ExecutionException e) {
            // deal with the exception
          }
        }
      }
    }

    public static void main(String[] args) {
       new      
  AvroProcessor("localhost:9092","http://localhost:8081").process();//7
    }
}
```

列表 5.7：AvroProcessor.java（第二部分）

分析`AvroProcessor`，我们可以看到以下内容：

+   在行`//1`中，消费者订阅了新的 Avro 主题。

+   在行`//2`中，我们正在消费类型为`GenericRecord`的消息。

+   在行`//3`中，将 Avro 记录反序列化以提取`HealthCheck`对象。

+   在行`//4`中，将开始时间转换为当前时区的格式。

+   在行`//5`中，计算了运行时间。

+   在行`//6`中，将运行时间写入`uptimes`主题，使用序列号作为键，运行时间作为值。两个值都作为普通字符串写入。

+   在行`//7`中，所有操作都在本地的`9092`端口上的代理上运行，并且模式注册表在本地的`8081`端口上运行。

如前所述，代码不是类型安全的；所有类型都在运行时进行检查。因此，请对此格外小心。例如，字符串不是`java.lang.String`；它们是`org.apache.avro.util.Utf8`类型。请注意，我们通过直接在对象上调用`toString()`方法来避免类型转换。其余的代码保持不变。

# 运行 AvroProcessor

要构建项目，请从`kioto`目录中运行以下命令：

```java
$ gradle jar
```

如果一切正常，输出将类似于以下内容：

```java
BUILD SUCCESSFUL in 3s
 1 actionable task: 1 executed
```

运行`uptimes`主题的控制台消费者，如下所示：

```java
$ ./bin/kafka-console-consumer --bootstrap-server localhost:9092 --topic uptimes --property print.key=true
```

1.  从 IDE 中运行`AvroProcessor`的主方法

1.  从 IDE 中运行`AvroProducer`的主方法

1.  控制台消费者对于`uptimes`主题的输出应类似于以下内容：

```java
EW05-HV36 33
BO58-SB28 20
DV03-ZT93 46
...
```

# 摘要

在本章中，我们展示了，而不是以 JSON 格式发送数据，如何使用 AVRO 作为序列化格式。AVRO（例如，相对于 JSON）的主要好处是数据必须符合模式。AVRO 相对于 JSON 的另一个优点是，以二进制格式发送时，消息更加紧凑，尽管 JSON 是可读的。

模式存储在模式注册表中，这样所有用户都可以咨询模式版本历史，即使那些消息的生产者和消费者的代码不再可用。

Apache Avro 还保证了此格式中所有消息的向前和向后兼容性。通过遵循一些基本规则实现向前兼容性，例如，当添加新字段时，将其值声明为可选的。

Apache Kafka 鼓励在 Kafka 系统中使用 Apache Avro 和模式注册表来存储所有数据和模式，而不是仅使用纯文本或 JSON。使用这个成功的组合，你可以保证你的系统可以进化。
