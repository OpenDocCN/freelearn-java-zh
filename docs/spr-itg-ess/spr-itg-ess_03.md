# 第三章：消息处理

在第一章，*入门 *，我们讨论了企业集成需求是为了解决异构系统之间互操作通信的问题：它们将如何共享数据，它们将如何理解其他系统的数据，如何处理跨应用程序的交叉问题等等。在第二章中，我们讨论了一个方面，即系统将如何交换数据。通道为数据提供了一个逻辑单位，可以将其投放给其他感兴趣的应用程序。然而，它引入了下一组挑战：如果数据格式其他模块无法理解，或者消息生成与消息消费的速率不同怎么办？让我们以一个例子来看；需要从互联网获取 RSS 源并将其放入数据库以进行报告，以及在邮件系统中发送有关新条目可用性的邮件。它会带来什么挑战？

+   RSS 源是 XML 格式的，而数据库和邮件需要转换成 Java 实体和 Java `MailMessage`格式（假设使用 JPA 和 java 邮件）。这意味着 XML 负载需要转换为下一组端点期望的格式。

+   发送邮件时可能会出现延迟；因此，淹没邮件服务器可能会导致消息丢失，表明需要节流。

+   在消息可以被交给数据库之前，需要增加一些审计信息，如时间戳、登录用户等。

+   可能存在一些无效或不完整的 XML 负载。我们希望丢弃这些负载并重新尝试！

+   邮件服务器可能在源到达时不可用——那时该怎么办？

这些点提供了一个快速了解在两个系统尝试通信时需要照顾到的几个方面。肯定不希望用这么多重逻辑来负载系统，并引入它们之间的紧密耦合。那么，谁来照顾所有这些方面呢？让我们欢迎消息端点。在本章中，我们将介绍以下主题：

+   消息端点

+   网关

+   服务激活器

+   延迟器

+   事务

# 消息端点

在最简单的类比中，**消息端点**是促进两个系统之间交互的启用器——无论是消息的转换、节流、中间业务处理，还是消息成功且无缝地处理所需的任何其他任务。为了满足不同的需求，提供了不同类型的消息端点，例如，*增强器*、*延迟器*、*服务激活器*等。然而，在深入讨论每个具体细节之前，让我们讨论一下端点的广泛分类：

+   **接收器或发送器**：端点可以从信道中接收消息，或者将消息放入信道进行进一步处理。

+   **轮询端点或事件驱动端点**：端点可以从信道中拉取消息，或者可以订阅它。每当有消息可用时，注册的回调方法就会被调用。

+   **单向或双向端点**：单向端点发送或接收消息，但不期望或接收任何确认。Spring Integration 为这类交互提供了信道适配器。双向适配器可以发送、接收和确认消息。Spring Integration 提供了与同步双向通信相同的网关。

+   **入站或出站端点**：出站端点与社交网络、邮件服务器、企业 JMS 等外部系统交互，而入站端点则监听来自外部实体（如邮件连接器、FTP 连接器等）的事件。

Spring Integration 为这些类型提供了所有实现的；让我们探索它们。

# 网关

总是希望实现抽象和松耦合。**消息网关**是一种机制，用于发布可以被系统使用而不暴露底层消息实现的合同。例如，邮件子系统的网关可以暴露发送和接收邮件的方法。内部实现可以使用原始 Java 邮件 API，或可以是 Spring Integration 的适配器，或可能是自定义实现。只要合同不改变，实现可以很容易地切换或增强，而不会影响其他模块。它是更一般的*网关*模式的一种实现。网关可以分为两种类型：*同步*和*异步*。

## 同步网关

让我们快速看看在 Spring Integration 中一个网关的声明看起来像什么，然后再进一步解析以建立我们的理解：

```java
<int:gateway id="feedService" 
  service-interface="com.cpandey.siexample.service.FeedService" 
  default-request-channel="requestChannel" 
  default-reply-channel="replyChannel"/>
```

这段基本代码定义了 Spring 中的一个网关。让我们理解一下前面的声明：

+   `int:gateway`：这是 Spring 框架的网关命名空间。

+   `service-interface`：这是一个由网关发布的接口合同。

+   `default-request-channel`：这是网关放置消息进行处理的信道。

+   `default-reply-channel`：这是网关期望回复的信道。

接口是一个简单的 Java 接口声明：

```java
public interface FeedService {
  FeedEntitycreateFeed(FeedEntity feed);
  List<FeedEntity>readAllFeed();
}
```

我们定义了一个接口，然后定义了通过网关发送和读取消息的通道——但组件用来处理消息并确认它的实现类在哪里？在这里，涉及到一些 Spring Integration 的魔法——当解析这个 XML 时，框架的`GatewayProxyFactoryBean`类会创建这个接口的代理。如果有对声明的网关的服务请求，代理会将消息转发到`default-request-channel`，并会阻塞调用到`default-reply-channel`上有确认可用。前面的声明可以进一步扩展，以每个网关方法调用的通道为单位：

```java
<int:gateway id="feedService" 
  service-interface="com.cpandey.siexample.service.FeedService" 
  <int:method name="createFeed" 
    request-channel="createFeedRequestChannel"/>
  <int:method name="readAllFeed" 
    request-channel="readFeedRequestChannel"/>
</int:gateway>
```

现在当调用`createFeed`方法时，消息将被放入`createFeedRequestChannel`，而对于网关的`readAllFeed`方法，消息将被转发到`readFeedRequestChannel`。等一下——`default-reply-channel`在哪里？回复通道是一个可选参数，如果没有声明，网关会创建一个匿名的点对点回复通道，并将其添加到消息头中，名为`replyChannel`。如果我们需要一个发布-订阅通道，多个端点可以监听，显式声明将会有所帮助。

我们可以很容易地利用 Spring Integration 的注解支持，而不是使用 XML 声明：

```java
public interface FeedService{
  @Gateway(requestChannel="createFeedRequestChannel")
    FeedEntitycreateFeed(FeedEntity feed);
  @Gateway(requestChannel="readFeedRequestChannel")
    List<FeedEntity>readAllFeed();
}
```

## 异步网关

异步网关不期望确认。在将消息放入请求通道后，它们会在回复通道上阻塞等待回复的情况下，转而进行其他处理。Java 语言的`java.util.concurrent.Future`类提供了一种实现这种行为的方法；我们可以定义一个返回`Future`值的网关服务。让我们修改`FeedService`：

```java
public interface FeedService {
  Future<FeedEntity>createFeed(FeedEntity feed);
  Future<List<FeedEntity>>readAllFeed();
}
```

其他一切保持不变，所有的 XML 声明都保持一样。当返回类型更改为`Future`时，Spring 框架的`GatewayProxyFactoryBean`类通过利用`AsyncTaskExecutor`来处理切换到异步模式。

# 服务激活器

**服务激活器**是最简单且最实用的端点之一——一个普通的 Java 类，其方法可以被调用在通道上接收的消息。服务激活器可以选择终止消息处理，或者将其传递到下一个通道进行进一步处理。让我们看一下以下的示例。我们希望在将消息传递到下一个通道之前进行一些验证或业务逻辑。我们可以定义一个 Java 类，并按照如下方式注解它：

```java
@MessageEndpoint
public class PrintFeed {
  @ServiceActivator
  public String upperCase(String input) {
    //Do some business processing before passing the message
    return "Processed Message";
  }
}
```

在我们的 XML 中，我们可以将类附加到一个通道上，以便处理其中的每一个消息：

```java
<int:service-activator input-channel="printFeedChannel" ref="printFeed" output-channel="printFeedChannel" />
```

让我们快速浏览一下前面声明中使用的元素：

+   `@MessageEndpoint`：这个注解告诉 Spring 将一个类作为特定的 Spring bean——一个消息端点。由于我们用`MessageEndpoint`注解了这个调用，所以在 XML 中不需要声明这个 bean。它将在 Spring 的组件扫描中被发现。

+   `@ServiceActivator`：这个注解将一个应该在消息到达通道时调用的方法映射起来。这个消息作为一个参数传递。

+   `int:service-activator`：这是声明 Spring 端点类型的 XML 命名空间。

+   `input-channel`：这是服务激活器将要读取消息的通道。

+   `output-channel`：这是激活器将要倾倒处理过的消息的通道。

+   `ref`：这是执行处理的 bean 的引用。

前面的示例限制了一个类中的单个方法作为`@ServiceActivator`。然而，如果我们想要委派到一个明确的方法——也许根据负载？我们在以下代码中定义服务激活器的方法元素：

```java
<int:service-activator ref="feedDaoService"
  method="printFeed" input-channel="printAllFeedChannel"/>

<int:service-activator ref="feedService" method="readFeed" input-channel="printAllFeedChannel"/>
```

在这两个声明中，服务激活器的引用是相同的，也就是说，作为服务的类是`feedDaoService`，但在不同的场景中调用其不同的方法。

如我们之前提到的，输出通道是可选的。如果方法返回类型是 void，那么它表示消息流已经终止，Spring Integration 对此没有问题。然而，如果消息类型不为 null，输出通道也省略了怎么办？Spring Integration 将尝试一个后备机制——它将尝试在消息中查找名为`replyChannel`的头部。如果`replyChannel`头部的值是`MessageChannel`类型，那么消息将被发送到那个通道。但如果它是一个字符串，那么它将尝试查找具有该名称的通道。如果两者都失败，它将抛出一个`DestinationResolutionException`异常。

服务激活器可以处理哪种类型的消息？方法参数可以是`Message`类型或 Java `Object`类型。如果是`Message`，那么我们可以读取载荷并对其进行处理——但这引入了对 Spring `Message`类型的依赖。一个更好的方法是在前一个示例中声明 Java 类型。Spring Integration 将负责提取载荷并将其转换为声明的对象类型，然后调用服务激活器上的方法。如果类型转换失败，将抛出异常。同样，方法返回的数据将被包裹在一个`Message`对象中，并传递到下一个通道。

有没有没有参数的激活方法呢？有的！这在只关心是否执行了某个操作的场景中非常有用，例如，或许用于审计或报告目的。

# 延迟者

正如我们在介绍部分已经讨论过的，消息的生产率和消费率可能会有所不同——如果消费者变慢了怎么办？由于涉及到外部系统，我们可能无法控制生产消息的速率。这时就需要使用延时器。**延时器**是一个简单的端点，在消息被传递到下一个端点之前引入一个延迟。最值得注意的是，原始发送者既不会被阻塞也不会被减慢；而是，延时器将从通道中选择一个消息，并使用`org.springframework.scheduling.TaskScheduler`实例来安排其在配置间隔后发送到输出通道。让我们写一个简单的延时器：

```java
<int:delayer id="feedDelayer" 
  input-channel="feedInput"
  default-delay="10000" 
  output-channel="feedOutput"/>
```

这个简单的配置将会延迟输入通道向输出通道发送消息 10 秒。

如果我们想延迟每个消息以不同的时间间隔——比如说根据负载大小我们想增加或减少延迟时间，`expression`属性就派上用场了。之前的例子可以修改如下：

```java
<int:delayer id="feedDelayer" 
  input-channel="feedInput"
  default-delay="10000"
  output-channel="feedOutput"
  expression="headers['DELAY_MESSAGE_BY']"/>
```

```java
`int:delayer`: This is the Spring Integration namespace support for the delayer`input-channel`: This is the channel from which messages have to be delayed`default-delay`: This is the default delay duration in milliseconds`output-channel`: This is the channel where messages should be dropped after the delay is over`expression`: This is the expression that is evaluated to get the delay interval for each of the messages based on a set header value
```

延时器通过一定的间隔来延迟消息——如果系统在还有尚未在输出通道上交付的延迟消息时宕机怎么办？我们可以利用`MessageStore`，特别是持久性`MessageStore`接口，如`JdbcMessageStore`。如果使用它，那么系统一旦宕机，所有消息都会被立即持久化。当系统恢复时，所有延迟间隔已到的消息都将立即在输出通道上交付。

# 事务

我们一直在讨论消息端点如何使不同子系统之间能够进行通信。这引发了一个非常关键的问题——那么关于事务呢？它们如何在链上处理？Spring Integration 在事务方面提供了哪些功能？

Spring Integration 本身并不提供对事务的额外支持；相反，它是建立在 Spring 提供的事务支持的基础之上。它只是提供了一些可以用来插入事务行为的钩子。给服务激活器或网关注解上事务注解将支持消息流的事务边界。假设一个用户流程在事务性传播的上下文中启动，并且链中的所有 Spring Integration 组件都注解为事务性的，那么链中任何阶段的失败都将导致回滚。然而，这只有在事务边界没有被破坏的情况下才会发生——简单来说，一切都在一个线程中进行。单线程执行可能会断裂，例如，使用任务执行器创建新线程用例，持有消息的聚合器，以及可能发生超时。以下是一个快速示例，使轮询器具有事务性：

```java
<int-jpa:inbound-channel-adapter 
  channel="readFeedInfo" 
  entity-manager="entityManager"
  auto-startup="true" 
  jpa-query="select f from FeedDetailsf" 
  <int:poller fixed-rate="2000" >
    <int:transactional propagation="REQUIRED" 
      transaction-manager="transactionManager"/> 
  </int:poller>
</int-jpa:inbound-channel-adapter>
```

在这里，`"entity-manager"`、`"transaction-manager"` 等都是标准的 Spring 组件——只是这里使用了来自 Spring Integration 的命名空间，比如 `int-jpa` 和 `int:transactional`，来将它们集成进来。目前，适配器对我们来说并不重要；我们将在后续章节中涵盖所有其他的标签。

那么，有没有一种用例，进程没有在事务中启动，但后来我们想在子系统中引入事务呢？例如，一个批处理作业或轮询器，它在通道上轮询并选择一个文件上传到 FTP 服务器。这里没有事务的传播，但我们希望使这一部分具有事务性，以便在失败时可以重试。Spring Integration 为轮询器提供了事务支持，可以帮助启动事务，以便在轮询器之后的进程可以作为一个单一的工作单元来处理！下面是一个快速示例：

```java
<int:poller max-messages-per-poll="1" fixed-rate="1000"> 
  <int:transactional transaction-manager="transactionManager"
    isolation="DEFAULT"
    propagation="REQUIRED"
    read-only="true"
    timeout="1000"/>
</poller>
```

总结一下，Spring Integration 整合了 Spring 事务支持，并且凭借一点直觉和创造力，它甚至可以扩展到本质上非事务性的系统！

# 总结

在本章中，我们理解了为什么需要消息端点的原因，并发现了一些 Spring Integration 提供的端点。我们介绍了网关如何抽象底层消息实现，使开发者的工作更加简单，服务激活器如何在系统中对消息进行中间处理，以及延时器如何用来调节消息处理速率以匹配生产者和消费者的速度！我们简要提到了事务支持——我们只讨论它是因为它不提供任何新的实现，并且依赖于 Spring 框架的事务支持。

在下一章中，我们将更深入地探讨一个最重要的端点——消息转换器。
