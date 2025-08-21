# 第四章：消息转换器

上一章的启示是消息端点使两个异构组件之间的握手变得透明和无缝。在本章中，我们将深入研究集成中的一个重要问题——消息的转换，以便它们可以在链中被消费。我们将介绍：

+   消息转换器

+   处理 XML 有效载荷

+   丰富器

+   索赔检查

同一组数据可以被不同的系统在不同的上下文中查看，例如，员工记录被报告系统和财务系统使用。然而，对象的使用将不同。报告系统只是丢弃员工记录——所以即使它以单个字符串的形式获取也没关系。另一方面，工资系统可能需要发送邮件通知、根据州和国家计算税款，以及执行其他功能，这些功能需要员工数据以 POJO 的形式呈现，信息在单独的字段中，例如，姓名、州、国家、电子邮件等。同样，可能存在需要将原始消息中添加附加信息的情况，可能需要进行加密/解密或转换为某种专有格式——这些就是消息转换器发挥作用的场景！

# 引入消息转换器

消息转换器是名为**消息转换器**的企业集成模式（**EIP**）的实现，该模式为**企业集成模式**（**EIP**），它处理端点之间的数据格式对等。这是一种优雅的设计，可以解耦消息生产者和消息消费者——它们都不需要知道对方期望的格式。这几乎与核心 Java 设计原则中的适配器模式一样，它充当生产者和消费者之间的启用器。让我们举一个更通用的例子，我们经常在 Windows 和 Linux 之间传输文件，尽管这两个系统所需格式不同，但底层应用程序负责从一种格式转换到另一种格式。

Spring Integration 提供了许多开箱即用的转换器，同时保留了定义和扩展新转换器的灵活性。它为最常用的消息交换格式提供了广泛支持，如 XML、JSON、集合等。其中，总的来说，当涉及到跨语言和跨平台通信时，XML 是使用最广泛的语言。让我们来探讨 Spring Integration 对 XML 的支持，然后再探索消息转换的其他方面。

# 处理 XML 有效载荷

两个不同的系统可能同意通过 XML 格式进行交互。这意味着每当有 outgoing 通信时，系统的数据结构需要转换为 XML；而在 incoming 消息的情况下，它需要转换为系统能理解的数据结构。我们怎么做到这一点呢？Spring 通过其 **OXM** （**对象到 XML**）框架提供了处理 XML 的第一级支持。通过相应的类—`org.springframework.oxm.Marshaller` 和 `org.springframework.oxm.Unmarshaller` 进行 marshalling 和 unmarshalling。**Marshaller** 将一个对象转换为 XML 流，而 **unmarshaller** 将 XML 流转换为对象。Spring 的对象/XML 映射支持提供了几个实现，支持使用 JAXB、Castor 和 JiBX 等进行 marshalling 和 unmarshalling。Spring Integration 进一步抽象了这一点，并提供了许多开箱即用的组件，帮助处理 XML 有效载荷。其中一些是 *marshalling transformer*, *unmarshalling transformer*, 和 *XPath transformer*。还有像 Xslt transformer、XPath 分割器和 XPath 路由器等，但我们只覆盖最常用的几个。

## marshalling transformer

用于将对象图转换为 XML 格式的 marshalling transformer。可以提供一个可选的结果类型，可以是用户定义的类型，或者是 Spring 内置的两种类型之一：`javax.xml.transform.dom.DOMResult` 或 `org.springframework.xml.transform.StringResult`。

以下是一个 marshalling transformer 的示例：

```java
<int-xml:marshalling-transformer 
  input-channel="feedsMessageChannel" 
  output-channel="feedXMLChannel" 
  marshaller="marshaller" 
  result-type="StringResult" />
```

这里使用的不同元素的说明如下：

+   `int-xml:marshalling-transformer`：这是由 Spring Integration 提供的命名空间支持

+   `input-channel`：这是从中读取消息的通道

+   `output-channel`：这是 transformed messages 将被丢弃的通道

+   `marshaller`：这是用于 marshalling 的 marshaller 实例

+   `result-type`：这是结果应该被 marshalled 的类型

需要一个有效的 marshaller 引用，例如：

```java
<bean id="marshaller" 
  class="org.springframework.oxm.castor.CastorMarshaller"/>
```

这个示例使用了一种 Spring 内置类型，`org.springframework.xml.transform.StringResult` 作为结果类型。如果未指定 `result-type`，则使用默认的 `DOMResult`。这里也可以使用自定义结果类型：

```java
<int-xml:marshalling-transformer 
  input-channel="feedsMessageChannel" 
  output-channel="feedXMLChannel" 
  marshaller="marshaller" 
  result-factory="feedsXMLFactory"/>
```

这里，`feedsXMLFactory` 指的是一个类，它实现了 `org.springframework.integration.xml.result.ResultFactor` 并重写了 `createResult` 方法：

```java
public class FeedsXMLFactory implements ResultFactory {
  public Result createResult(Object payload) {
  //Do some stuff and return a type which implements
  //javax.xml.transform.result
  return //instance of javax.xml.transform.Result.
  }
}
```

## unmarshalling transformer

几乎所有元素都与前面提到的 marshaller 相同，除了 `unmarshaller` 元素，它应该指向 Spring 支持的有效 unmarshaller 定义。

## XPath 转换器

Spring 集成中的 `xpath-transformer` 组件可以用 XPath 表达式来解析 XML：

```java
<int-xml:xpath-transformer input-channel="feedsReadChannel"
  output-channel="feedTransformedChannel"
  xpath-expression="/feeds/@category" />
```

可以使用 `xpath-expression` 标签给出要评估的 XPath 表达式。当 XML 有效载荷到达输入通道时，转换器解析 XPATH 值并将结果放入输出通道。

默认情况下，解析的值被转换为一个带有字符串负载的消息，但如果需要，可以进行简单的转换。Spring 支持以下隐式转换：`BOOLEAN`、`DOM_OBJECT_MODEL`、`NODE`、`NODESET`、`NUMBER`和`STRING`。这些都在`javax.xml.xpath.XPathConstants`中定义，如下所示：

```java
<int-xml:xpath-transformer input-channel="feedsReadChannel" 
  xpath-expression="/feeds/@category"
  evaluation-type=" STRING_RESULT" 
  output-channel=" feedTransformedChannel "/>
```

`evaluation-type`标签用于引入所需的转换。

# 验证 XML 消息

当我们讨论 XML 转换时，提及 XML 负载的验证方面是相关的。预验证 XML 将使系统免于进入错误状态，并且可以在源头采取行动。Spring Integration 通过过滤器提供 XML 验证的支持：

```java
<int-xml:validating-filter 
  id="feedXMLValidator" 
  input-channel="feedsReadChannel" 
  output-channel="feedsWriteChannel" 
  discard-channel="invalidFeedReads" 
  schema-location="classpath:xsd/feeds.xsd" />
```

`schema-location`元素定义了用于验证的 XSD。这是可选的，如果没有这样做，将其设置为默认的`xml-schema`，内部转换为`org.springframework.xml.validation.XmlValidatorFactory#SCHEMA_W3C_XML`。

我们讨论了很多内置转换器，主要处理 XML 负载。除了这些，Spring Integration 为最常见的转换提供了许多开箱即用的转换器，例如：

+   `object-to-string-transformer`

+   `payload-serializing-transformer`

+   `payload-deserializing-transformer`

+   `object-to-map-transformer`

+   `map-to-object-transformer`

+   `json-to-object-transformer`

+   `object-to-json-transformer`等

详细介绍每一个超出了本书的范围，但概念与前面提到的相同。

# 超出默认转换器

Spring 并没有限制我们使用框架提供的转换器，我们可以定义自己的转换器，这是相当直接的。我们只需要定义一个 Java 类，它接受特定输入类型，将其转换为期望的格式并将其放入输出通道。让我们举一个例子，我们想要将我们的 feed 转换为可以写入数据库的格式；我们可以定义一个类，它接受类型为`com.sun.syndication.feed.synd.SyndEntry`的*Message*负载并将其转换为`com.cpandey.siexample.pojo.SoFeed`，这是一个 JPA 实体：

```java
import com.cpandey.siexample.pojo.SoFeed;
import com.sun.syndication.feed.synd.SyndEntry;
public class SoFeedDbTransformer {
  publicSoFeedtransformFeed(Message<SyndEntry> message){
    SyndEntry entry = message.getPayload();
    SoFeed soFeed=new SoFeed();
    soFeed.setTitle(entry.getTitle());
    soFeed.setDescription(entry.getDescription().getValue());
    soFeed.setCategories(entry.getCategories());
    soFeed.setLink(entry.getLink());
    soFeed.setAuthor(entry.getAuthor());
    //For DB return soFeed
    returnsoFeed;
  }
```

转换器可以使用以下代码声明：

```java
<int:transformer ref="feedDbTransformerBean" 
  input-channel="filteredFeedChannel"
  method="transformFeed" 
  output-channel="jdbcChannel"/>
```

```java
`int:transformer`: This provides the XML namespace supported by Spring Integration`ref`: This is used to provide a reference of bean definition, which will act as the transformer`input-channel`: This is the channel from which messages will be picked up by the transformer`output-channel`: This is the channel where messages will be dropped after completing required transformations`method`: This is the method of the class that will have the transformation logic
```

让我们定义`ref`标签所引用的 bean：

```java
<bean id="feedDbTransformerBean" class="com.cpandey.siexample.transformer.SoFeedDbTransformer" />
```

如前所述，这个类有转换所需的方法。这个 bean 可以在转换器之间使用，每个方法可以有独立的转换逻辑。

# 内容丰富器

在启用异构系统之间的交互时，可能需要向消息添加附加信息，以便它能够被下一组消费者成功处理。让我们举一个例子，在一个批处理环境中，可能需要向传入任务附加优先级信息。对于放在文件服务器上供外部消费的消息—应添加一个时间戳，指示文件将被保留的最大时间。可能存在许多这样的场景，传入的消息不完整，需要由下一个端点处理。内容增强器是一种特殊的转换器，可以为消息附加附加信息。在 Spring Integration 的上下文中，消息由两部分组成—头部和消息载荷。Spring Integration 暴露了一种丰富这些组件中任何一个的方法。

## 头部增强器

**头部**在 Spring Integration 中是`MessageHeaders`类的实例，该类又扩展了`Map<String,?>`。头部不过是键值对，其目的是提供关于消息的元数据。添加一个附加的头部是直接的。让我们举一个例子，无论何时我们的系统中的饲料通过 XML 验证，我们将添加一个常数，指示饲料已验证：

```java
<int:header-enricher input-channel="validatedFeedsChannel" 
  output-channel="nextChannelForProcess">
  <int:header name="validated" value="true"/>
</int:header-enricher>
```

```java
`int:header-enricher`: This element provides the Spring Integration XML namespace support for the header enricher`input-channel`: The header for each message on this channel will be enriched`output-channel`: Additional header messages will be dropped on this channel`int:header`: This is used to provide the key-value pair for the header name and header value
```

如果我们想添加一些动态值，比如说一个时间戳，以特定的格式呢？我们可以利用头部增强器的 bean 支持，在 bean 中定义自定义增强：

```java
<int:header-enricher input-channel="feedsInputChannel" 
  output-channel=" nextChannelForProcess "> 
  <int:header name="customtimestamp"
    method="getTimeStamp"
    ref="timeStamp"/>
</int:header-enricher>
```

这里提到的`ref`标签引用的 bean 如下：

```java
<bean id="timeStamp " class="com.cpandey.siexample.TimeStamp"/>
```

实际类的定义如下：

```java
public class TimeStamp {
  public String getTimeStamp (String payload){
    return //a custom time stamp
  }
}
```

除了一个标准的 Java Bean，我们还可以使用 Groovy 脚本来定义自定义增强器：

```java
<int:header-enricher input-channel="feedsInputChannel" 
  output-channel=" nextChannelForProcess ">
  <int:header name="customtimestamp" 
  <int-groovy:script location="="siexample 
    /TimeStampGroovyEnricher.groovy"/>
  </int:header>
</int:header-enricher>
```

还有预定义的头部元素也可以使用；最简单、最常用的是 error-channel：

```java
<int:header-enricher input-channel=" feedsInputChannel " output-channel=" nextChannelForProcess ">
  <int:error-channel ref="feedserrorchannel"/>
</int:header-enricher>
```

## 载荷增强器

**头部增强器**方便地添加元数据信息。如果消息本身不完整怎么办？让我们举一个例子，当一个饲料到达时，根据饲料类别，可能需要获取该类别的元数据，订阅该类别的用户等等。可以使用其他组件，如服务激活器和网关，但为了方便使用，Spring Integration 暴露了载荷增强器。**载荷增强器**就像网关—把消息放到一个通道上，然后期待这个消息的回复。返回的消息将是载荷增强的。例如，假设外部饲料对 Spring 有很多类别，如 Spring-mvc、Spring-boot、Spring-roo 和 Spring-data，但我们的系统只有一个类别—Spring。基于外部类别，我们可以增强载荷以使用单个类别：

```java
<int:enricher id="consolidateCategoryEnricher"
  input-channel="findFeedCatoryChannel"
  request-channel="findInternalCategoryChannel">
  <int:property name="categroy" 
    expression="payload.category"/>
  <int:property name="feedProcessed" 
    value="true" type="java.lang.String"/>
</int:enricher>
```

这里，配置元素意味着以下内容：

+   `int:enricher`：这是用作 Spring Integration 命名空间支持以增强器的。

+   `input-channel`：这是用于增强的数据读取通道。

+   `request-channel`：这是用于丰富数据的数据发送通道。

+   `int:property`：这是一种方便的设置目标有效载荷值的方法。所提到的属性必须在目标实例上“可设置”。它可以是一个**SpEL**（**Spring 表达式语言**）表达式，由`expression`表示，或者可以是一个由值表示的值。

# 索赔检查

我们讨论了头部和内容丰富器的使用——它们增加了额外信息。然而，在某些情况下，隐藏数据可能是有效的用例——最简单的是重载荷。在大多数通道可能只使用子集甚至只是传递时，移动整个消息并不是一个好主意！引入了一个*索赔检查模式*，它建议将数据存储在可访问的存储中，然后只传递指针。需要处理数据的组件可以使用指针检索它。Spring 集成提供了两个组件来实现这一点：*入站索赔检查转换器*和*出站索赔检查转换器*。入站索赔检查转换器可用于存储数据，而出站索赔检查转换器可用于检索数据。

## 入站索赔检查转换器

**入站索赔检查转换器**将消息存储在其消息存储标记中，并将其有效载荷转换为实际消息的指针，如下面的代码片段所示：

```java
<int:claim-check-in id="feedpayloadin"
  input-channel="feedInChannel"
  message-store="feedMessageStore"
  output-channel="feedOutChannel"/>
```

一旦消息存储在消息存储中，它就会生成一个 ID 进行索引，该 ID 成为该消息的索赔检查。转换后的消息是索赔检查，即新的有效载荷，并将发送到输出通道。要检索此消息，需要一个出站索赔检查转换器。

## 出站索赔检查转换器

基于索赔检查，此转换器将指针转换回原始有效载荷，并将其放回输出通道。如果我们想限制索赔一次怎么办？我们可以引入一个布尔值`remove-message`，将其值设置为 true 将在索赔后立即从消息存储中删除消息。默认值为 false。更新后的代码如下所示：

```java
<int:claim-check-out id="checkout"
  input-channel="checkoutChannel"
  message-store="testMessageStore"
  output-channel="output"
  remove-message="true"/>
```

# 总结

我们讨论了消息可以如何被丰富和转换，以便使异构系统与彼此的数据格式解耦。我们还讨论了索赔检查概念，这是转换的一个特例，可以用于性能、安全和其他非功能性方面。

在下一章中，我们将探讨 Spring Integration 提供的更多开箱即用的组件，以帮助消息流。
