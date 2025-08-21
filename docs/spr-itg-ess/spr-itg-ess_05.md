# 第五章．消息流

我们在上章讨论了消息转换。在转换得到处理之后，在传递到链中的下一个环节之前可能还需要执行其他任务。例如，消息可能需要进行一些分割，或者它们可能是不完整的，需要一些临时存储或排序。在本章中，我们将探讨 Spring Integration 框架为在不同组件之间无缝传递消息提供的开箱即用的功能。我们将在本章涵盖以下主题：

+   路由器

+   过滤器

+   分割器

+   聚合器

+   重新排序器

+   链式处理器

# 路由器

**路由器**是选择消息并将其根据一组预定义的规则传递到不同通道的组件。路由器从不改变消息——它们只是将消息路由/重新路由到下一个目的地。Spring Integration 提供了以下内置路由器：

+   负载类型路由器

+   头部值路由器

+   收件人列表路由器

+   XPath 路由器（XML 模块的一部分）

+   错误消息异常类型路由器

## 负载类型路由器

从前面的代码片段可以看出，根据负载类型，消息被路由到不同的通道。`java.lang.String` 类已被配置为路由到 `jmsChannel`，而 `org.springframework.messaging.Message` 已被配置为路由到 `mailChannel`。以下两个元素已使用：

+   `int:payload-type-router`：这用于为负载类型路由器提供命名空间

+   `int:mapping`：此标签用于提供 Java 对象和通道之间的映射

## 头部值路由器

路由器**不基于消息负载的类型**，而是尝试读取已经设置在负载上的头部信息：

```java
<int:header-value-router 
  input-channel="feedsChannel" 
  header-name="feedtype">
  <int:mapping value="java" channel="javachannel" />
  <int:mapping value="spring" channel="springchannel" />
</int:header-value-router>
```

```java
mapping has not been provided and hence the next channel will be javachannel, indicated by the header-name tag:
```

```java
<int:header-value-router 
  input-channel="feedsChannel" 
  header-name="javachannel"/>
```

## 收件人列表路由器

不要将收件人误认为是用户！在这里，收件人列表指的是可以接收消息的通道列表。它可以与发布-订阅通道用例进行比较，其中预定义的一组通道与路由器“订阅”：

```java
<int:recipient-list-router input-channel="feedsChannel">
  <int:recipient channel="transformFeedChannel"/>
  <int:recipient channel="auditFeedChannel"/>
</int:recipient-list-router>
```

所有在 feeds 通道上传递的消息都将同时在 `transformFeedChannel` 和 `auditFeedChannel` 上传递。使用的元素很简单：

+   `int:recipient-list-router`：这用于为收件人列表路由器提供命名空间

+   `int:recipient`：这用于提供应接收消息的通道名称

## XPath 路由器

在第四章，*消息转换器*中，我们详细讨论了处理 XML 负载的问题，并讨论了基于*XPath*的转换器的示例。XPath 路由器类似——不是基于 XPath 值转换消息，而是将其路由到其中一个通道：

```java
<int-xml:xpath-router input-channel="feedChannel">
  <int-xml:xpath-expression expression="/feed/type"/>
</int-xml:xpath-router>
```

这可以将消息发送到通道或一组通道——表达式的值将决定消息应路由到哪些通道。有一种根据表达式的值将消息路由到特定通道的方法：

```java
<int-xml:xpath-router input-channel="feedChannel">
  <int-xml:xpath-expression expression="/feed/type"/>
  <int-xml:mapping value="java" channel="channelforjava"/>
  <int-xml:mapping value="spring" channel="channelforspring"/>
</int-xml:xpath-router>
```

## 错误消息异常类型路由器

```java
invalidFeedChannel, while for a NullPointerException, it will route to npeFeedChannel:
```

```java
<int:exception-type-router 
  input-channel="feedChannel"
  default-output-channel="defaultChannel">
<int:mapping 
  exception-type="java.lang.IllegalArgumentException" 
  channel="invalidFeedChannel"/>
 <int:mapping
    exception-type="java.lang.NullPointerException"
    channel="npeFeedChannel"/>
</int:exception-type-router>
<int:channel id=" illegalFeedChannel " />
<int:channel id=" npeFeedChannel " />
```

下面是对此代码片段中使用的标签的解释：

+   `int:exception-type-router`：这为异常类型路由器提供了命名空间。

+   `default-output-channel`：如果无法为消息解决任何映射的通道，则指定消息应该被投递到的默认通道。这将在后面详细定义。

+   `int:mapping exception-type`：用于将异常映射到通道名称。

## 默认输出通道

可能存在这样的情况，路由器无法决定消息应该被投递到哪个通道——在这种情况下该怎么办？以下有两种可用选项：

+   **抛出异常**：根据用例，这可以是一个已经被映射到通道的异常，或者异常可以被抛出，在链中向上传播。

+   **定义一个默认输出通道**：正如其名称所示，这是所有无法决定通道投递的消息将被投递的通道。

例如，在前面的代码片段中，默认通道已被指定为：

```java
default-output-channel="defaultChannel"
```

如果异常无法映射到定义的列表，消息将被放入默认通道。

# 使用注解

Spring 的威力在于将简单的 Java 类转换为具体的组件，而不需要扩展或实现外部类。为了定义路由器，我们可以利用框架的`@Router`注解。我们可以在任何方法上注解`@Router`，并可以使用其引用。让我们举一个例子，我们想要根据作者来路由我们的饲料：

```java
@Component
public class AnnotatedFeedsAuthorRouter {
  @Router
  public String feedAuthor(Message<SoFeed > message) {
    SoFeed sf = message.getPayload();
    return sf.getAuthor() ;
  }
}
```

返回值是一个字符串，是作者的名字——必须存在一个同名的通道。或者，我们可以直接返回`MessageChannel`或`MessageChannel`引用的列表。

# 过滤器

消息过滤器是 Spring Integration 组件，作为拦截器并决定是否将消息传递给下一个通道/组件或丢弃它。与决定消息下一个通道的路由器不同，过滤器只做一个*布尔*决定——是否传递。在 Spring Integration 中定义消息过滤器有两种方法：

+   编写一个简单的 Java 类，并指定其方法，决定是否传递消息

+   配置它作为一个消息端点，委托给`MessageSelector`接口的实现

这可以在 XML 中配置，也可以使用注解。

## 使用 Java 类作为过滤器

让我们以使用一个简单的 Java 类作为过滤器为例——这是我们关于饲料的例子的一部分。当饲料进来时，我们尝试验证载荷是否为空——只有通过验证后才将其传递进行进一步处理：

```java
<int:filter 
  input-channel="fetchedFeedChannel" 
  output-channel="filteredFeedChannel" 
  ref="filterSoFeedBean" 
  method="filterFeed"/>
```

标签的解释尽可能简单直观：

+   `int:filter`：用于指定 Spring 框架命名空间的过滤器

+   `input-channel`：消息将从这个通道中选择

+   `output-channel`：如果它们通过过滤条件，消息将被传递到的通道：

+   `ref`：这是对作为过滤器的 Java bean 的引用：

+   `method`：这是作为过滤器的 Java bean 的方法

作为过滤器的 bean 的声明如下：

```java
<bean id="filterSoFeedBean" 
class="com.cpandey.siexample.filter.SoFeedFilter"/>
```

以下代码片段显示了一个具有消息过滤方法的实际 Java 类：

```java
public class SoFeedFilter {
public boolean filterFeed(Message<SyndEntry> message){
  SyndEntry entry = message.getPayload();
  if(entry.getDescription()!=null&&entry.getTitle()!=null){
    return true;
  }
  return false;
}
```

我们还可以决定如果有效载荷不符合过滤条件该怎么办，例如，如果有效载荷为空。在这种情况下，我们可以采取以下两个选项之一：

+   可以抛出一个异常：

+   它可以路由到特定的通道，在那里可以对其采取行动—例如，只需记录失败的 occurrence：

要抛出异常，我们可以使用以下代码片段：

```java
<int:filter 
  input-channel="fetchedFeedChannel" 
  output-channel="filteredFeedChannel" 
  ref="filterSoFeedBean" 
  method="filterFeed" 
  throw-exception-on-rejection="true"/>
```

要记录异常，我们可以使用以下代码片段：

```java
<int:filter 
  input-channel="fetchedFeedChannel" 
  output-channel="filteredFeedChannel" 
  ref="filterSoFeedBean" 
  method="filterFeed" 
  discard-channel="rejectedFeeds"/>
```

在这里，我们在直接通道上使用了一个过滤器并验证了有效载荷。如果验证成功，我们传递了消息；否则，我们通过抛出异常或记录其发生来拒绝消息。过滤器的另一个用例可能是发布-订阅通道—许多端点可以监听一个通道并过滤出他们感兴趣的消息。

我们还可以使用*注解*来定义过滤器。只需在 Java 类的某个方法上使用`@Filter`注解，Spring Integration 就会将其转换为过滤器组件—无需扩展或实现任何额外的引用：

```java
@Component
public class SoFeedFilter {
  @Filter
  //Only process feeds which have value in its title and description
  public boolean filterFeed(Message<SyndEntry> message){
    SyndEntry entry = message.getPayload();
    if(entry.getDescription()!=null&&entry.getTitle()!=null){
      return true;
    }
  return false;
}
```

XML 中的过滤器声明需要更改，无需使用`method`参数：

```java
<int:filter 
  input-channel="fetchedFeedChannel" 
  output-channel="filteredFeedChannel" 
  ref="filterSoFeedBean" />
```

## 将过滤器配置为消息端点：

定义过滤器的另一个选项是使用框架（`MessageSelector`）。Java 类需要实现此接口并重写`accept`方法。每当传递有效载荷时，都会调用`accept`方法，并返回是否传递消息或丢弃它的决定。以下代码片段使用`MessageSelector`修改了前面的示例：

```java
public class SoFeedFilter implements MessageSelector{
public boolean accept(Message<?> message) {
      …
      return true;
    }
  return false;
}
```

在此定义之后，过滤器可以如下声明和使用：

```java
<int:filter 
  input-channel="fetchedFeedChannel" 
  outputchannel="filteredFeedChannel"> 
  <bean class=" com.cpandey.siexample.filter.SoFeedFilter "/>
</int:filter>
```

由于已经声明了 bean 类，因此无需引用标签。

# 分隔符：

分隔符，顾名思义，是用来将消息分成更小的块，然后将这些块发送进行独立处理。分割的原因可能有多个—有效载荷的大小大于下一个端点可接受的尺寸，或者可以并行处理或沿链处理的消息负载部分。有一个聚合器，在聚合之前需要进行一些处理。Spring Integration 提供了一个`splitter`标签。与过滤器一样，分隔符也可以通过扩展框架接口或编写自定义的 POJO 来编写。

让我们先从更简单的一个开始，利用一个简单的 Java 类作为分隔符：

```java
<int:splitter
  ref="splitterSoFeedBean" 
  method="splitAndPublish" 
  input-channel="filteredFeedChannel" 
  output-channel="splitFeedOutputChannel" />

<bean id="splitterSoFeedBean" 
  class="com.cpandey.siexample.splitter.SoFeedSplitter"/>
```

这些元素相当直观：

+   `int:splitter`：这是用于指定过滤器 Spring 框架命名空间的：

+   `ref`: 这是用来提供作为分隔器的 bean 的引用。

+   `method`: 这是在 bean 中指定消息分隔实现的方法。

+   `input-channel`: 这是消息将被读取的通道。

+   `output-channel`: 这是消息将被写入的通道。

充当分隔器的 Java 类：

```java
public class SoFeedSplitter {
  public List<SyndCategoryImpl> plitAndPublish(Message<SyndEntry> message) {
    SyndEntry syndEntry=message.getPayload();
    List<SyndCategoryImpl> categories= syndEntry.getCategories();
    return categories;
  }
}
```

分隔器必须返回一个集合类型，然后从这个集合中一次交付每个项目到下一个端点。如果返回的值不是消息类型，那么在交付之前每个元素都将被包裹在消息类型中。让我们为这个分隔器定义一个服务激活器：

```java
<int:service-activator 
  id="splitChannelSA"
  ref="commonServiceActivator"
  method="printSplitMessage"
  input-channel="splitFeedOutputChannel"/>
```

`printSplitMessage`方法在以下代码片段中定义：

```java
public void printSplitMessage(Message<SyndCategoryImpl> message) {
  if(message!=null){
    System.out.println(message.getPayload());
  }else{
    System.out.println("Message is null");
  }

}
```

我们可以通过使用注解来避免使用`method`标签：

```java
@Splitter
public List<SyndCategoryImpl> splitAndPublish(Message<SyndEntry> message) {
    SyndEntry syndEntry=message.getPayload();
    List<SyndCategoryImpl> categories= syndEntry.getCategories();
    return categories;
  }
```

与过滤器一样，我们也可以使用框架支持来编写我们的分隔器。任何 Java 类都可以扩展`AbstractMessageSplitter`并重写`splitMessage`。之前的示例已经通过以下代码片段中的框架支持进行了修改：

```java
public class SoFeedSplitter extends AbstractMessageSplitter {
  @Override
  protected Object splitMessage(Message<?> message) {
    SyndEntry syndEntry=(SyndEntry)message.getPayload();
    List<SyndCategoryImpl> categories= syndEntry.getCategories();
    return categories;
  }
```

# 聚合器

聚合器是分隔器的对立面——它们合并多个消息并将它们呈现给下一个端点作为一个单一的消息。这是一个非常复杂的操作，所以让我们从一个真实的生活场景开始。一个新闻频道可能有许多记者可以上传文章和相关图片。可能会发生文章的文字比相关的图片更早到达——但文章只有在所有相关的图片也到达后才能发送出版。这个场景提出了很多挑战；部分文章应该存储在某个地方，应该有一种方式将传入的组件与现有的组件相关联，还应该有一种方式来识别消息的完成。聚合器就是用来处理所有这些方面的——其中一些相关概念包括`MessageStore`、`CorrelationStrategy`和`ReleaseStrategy`。让我们从一个代码示例开始，然后深入探讨这些概念的每个方面：

```java
<int:aggregator 
  input-channel="fetchedFeedChannelForAggregatior"
  output-channel="aggregatedFeedChannel"
  ref="aggregatorSoFeedBean"
  method="aggregateAndPublish"
  release-strategy="sofeedCompletionStrategyBean"
  release-strategy-method="checkCompleteness"
  correlation-strategy="soFeedCorrelationStrategyBean"
  correlation-strategy-method="groupFeedsBasedOnCategory"
  message-store="feedsMySqlStore "
  expire-groups-upon-completion="true">
  <int:poller fixed-rate="1000"></int:poller>
</int:aggregator>
```

嗯，一个相当大的声明！为什么不呢——很多东西结合在一起充当聚合器。让我们快速浏览一下所有使用的标签：

+   `int:aggregator`: 这是用来指定 Spring 框架命名空间的聚合器。

+   `input-channel`: 这是消息将被消费的通道。

+   `output-channel`: 这是一个通道，在聚合后，消息将被丢弃。

+   `ref`: 这是用来指定在消息发布时调用的 bean 的方法。

+   `method`: 这是当消息被释放时调用的方法。

+   `release-strategy`: 这是用来指定决定聚合是否完成的方法的 bean。

+   `release-strategy-method`: 这是检查消息完整性的逻辑的方法。

+   `correlation-strategy`: 这是用来指定有消息相关联的方法的 bean。

+   `correlation-strategy-method`：这个方法包含了实际的消息关联逻辑。

+   `message-store`：用于指定消息存储，消息在它们被关联并准备好发布之前暂时存储在这里。这可以是内存（默认）或持久化存储。如果配置了持久化存储，消息传递将在服务器崩溃后继续进行。

可以定义 Java 类作为聚合器，如前所述，`method`和`ref`参数决定当根据`CorrelationStrategy`聚合消息并满足`ReleaseStrategy`后释放时，应调用 bean（由`ref`引用）的哪个方法。在以下示例中，我们只是在将消息传递到链中的下一个消费者之前打印消息：

```java
public class SoFeedAggregator {
  public List<SyndEntry> aggregateAndPublish(List<SyndEntry> messages) {
    //Do some pre-processing before passing on to next channel
    return messages;
  }
}
```

让我们来详细了解一下完成聚合器的三个最重要的组件。

## 关联策略

聚合器需要对消息进行分组——但它将如何决定这些组呢？简单来说，`CorrelationStrategy`决定了如何关联消息。默认情况下，是根据名为`CORRELATION_ID`的头部。所有`CORRELATION_ID`头部值相同的消息将被放在一个括号中。另外，我们可以指定任何 Java 类及其方法来定义自定义关联策略，或者可以扩展 Spring Integration 框架的`CorrelationStrategy`接口来定义它。如果实现了`CorrelationStrategy`接口，那么应该实现`getCorrelationKey()`方法。让我们看看在 feeds 示例中的我们的关联策略：

```java
public class CorrelationStrategy {
  public Object groupFeedsBasedOnCategory(Message<?> message) {
    if(message!=null){
      SyndEntry entry = (SyndEntry)message.getPayload();
      List<SyndCategoryImpl> categories=entry.getCategories();
      if(categories!=null&&categories.size()>0){
        for (SyndCategoryImpl category: categories) {
          //for simplicity, lets consider the first category
          return category.getName();
        }
      }
    }
    return null;
  }
}
```

那么我们是如何关联我们的消息的呢？我们是根据类别名称关联 feeds 的。方法必须返回一个可用于关联消息的对象。如果返回一个用户定义的对象，它必须满足作为映射键的要求，例如定义`hashcode()`和`equals()`。返回值不能为空。

另外，如果我们希望通过扩展框架支持来实现它，那么它看起来就像这样：

```java
public class CorrelationStrategy implements CorrelationStrategy {
  public Object getCorrelationKey(Message<?> message) {
    if(message!=null){
      …
            return category.getName();
          }
        }
      }
      return null;
    }
  }
}
```

## 发布策略

我们一直根据关联策略对消息进行分组——但我们什么时候为下一个组件发布它呢？这由发布策略决定。与关联策略类似，任何 Java POJO 都可以定义发布策略，或者我们可以扩展框架支持。以下是使用 Java POJO 类的示例：

```java
public class CompletionStrategy {
  public boolean checkCompleteness(List<SyndEntry> messages) {
    if(messages!=null){
      if(messages.size()>2){
        return true;
      }
    }
    return false;
  }
}
```

消息的参数必须是集合类型，并且必须返回一个布尔值，指示是否发布累积的消息。为了简单起见，我们只是检查了来自同一类别的消息数量——如果它大于两个，我们就发布消息。

## 消息存储

直到一个聚合消息满足发布条件，聚合器需要暂时存储它们。这就是消息存储发挥作用的地方。消息存储可以分为两种类型：内存存储和持久化存储。默认是内存存储，如果使用这种存储，那么根本不需要声明这个属性。如果需要使用持久化消息存储，那么必须声明，并且将其引用给予`message-store`属性。例如，可以声明一个 mysql 消息存储并如下引用：

```java
<bean id=" feedsMySqlStore " 
  class="org.springframework.integration.jdbc.JdbcMessageStore">
  <property name="dataSource" ref="feedsSqlDataSource"/>
</bean>
```

数据源是 Spring 框架的标准 JDBC 数据源。使用持久化存储的最大优势是可恢复性——如果系统从崩溃中恢复，所有内存中的聚合消息都不会丢失。另一个优势是容量——内存是有限的，它只能容纳有限数量的消息进行聚合，但数据库可以有更大的空间。

# 重排序器

**重排序器**可用于强制对下一个子系统进行有序交付。它会持有消息，直到所有在它之前的编号消息已经被传递。例如，如果消息被编号为 1 到 10，如果编号为 8 的消息比编号为 1 到 7 的消息更早到达，它会将其保存在临时存储中，并且只有在编号为 1 到 7 的消息传递完成后才会交付。消息的`SEQUENCE_NUMBER`头由重排序器用来跟踪序列。它可以被认为是聚合器的一个特例，它基于头值持有消息但不对消息进行任何处理：

```java
<int:resequencer input-channel="fetchedFeedChannelForAggregatior" 
  output-channel="cahinedInputFeedChannel" 
  release-strategy="sofeedResCompletionStrategyBean" 
  release-strategy-method="checkCompleteness" 
  correlation-strategy="soFeedResCorrelationStrategyBean" 
  correlation-strategy-method="groupFeedsBasedOnPublishDate" 
  message-store="messageStore"> 
  <int:poller fixed-rate="1000"></int:poller>
</int:resequencer >
```

正如我们提到的，重排序器可以被认为是聚合器的一个特例——几乎所有标签的意义都相同，除了命名空间声明。

# 链接处理器

我们已经讨论了 Spring Integration 提供的许多处理器，如过滤器、转换器、服务激活器等，这些处理器可以独立地应用于消息——Spring Integration 进一步提供了一种机制来链接这些处理器。`MessageHandler`的一个特殊实现是`MessageHandlerChain`，可以配置为一个单一的消息端点。它是由其他处理器组成的链，接收到的消息简单地按照预定义的顺序委托给配置的处理器。让我们来看一个例子：

```java
<int:chain 
  input-channel="cahinedInputFeedChannel" 
  output-channel="logChannel"> 
  input-channel="cahinedInputFeedChannel" 
  output-channel="logChannel"> 
  <int:filter ref="filterSoFeedBean" 
    method="filterFeed" 
    throw-exception-on-rejection="true"/> 
  <int:header-enricher> 
    <int:header name="test" value="value"/>
  </int:header-enricher>
  <int:service-activator 
    ref="commonServiceActivator" 
    method="chainedFeed"/>
</int:chain>
```

让我们快速创建一个链并验证它。从使用一个过滤器开始，这个过滤器仅仅传递所有消息，下一步在消息中添加一个头，最后在服务激活器中打印出这些头。如果我们能在第二步确认已经添加了头，那么我们就没问题——链已执行！

# 总结

深呼吸一下…这一章内容颇长，我们讨论了 Spring Integration 框架提供的许多创新组件，比如路由器、过滤器和分割器。这些组件都有助于消息在不同端点之间的流动。在下一章中，我们将继续探索 Spring Integration 框架的这些内置功能，但重点将放在适配器上，以与外部系统进行交互，比如连接数据库、从 Twitter 获取推文、向 JMS 队列写入、与 FTP 服务器交互等等——有很多有趣的内容，请继续关注！
