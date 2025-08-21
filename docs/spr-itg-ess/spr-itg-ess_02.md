# 第二章：消息摄取

如*序言*中所述，Spring Integration 是*企业集成模式：设计、构建和部署消息解决方案（Addison Wesley 签名系列）*，*Gregor Hohpe*和*Bobby Woolf*，*Addison-Wesley 专业*的实现。**EIP**（即**企业集成模式**的缩写）定义了许多集成挑战的模式，其中之一就是异构系统之间的消息交换。在本章中，我们将探讨与消息交换相关的模式和概念。

异构端点使用消息进行通信。消息交换主要有三个方面：正在交换的消息、参与通信的端点以及消息传递的媒介。在 EIP 范式中，我们将其定义为消息、消息端点和消息通道。让我们逐一讨论每一个，然后我们再讨论模式。

**消息是什么？**用最简单的术语来说，消息可以被理解为一个可以作为不同组件之间互操作和协作的启动物息。它主要由两部分组成：标题和有效载荷。标题包含元数据，通常需要诸如 ID、时间戳等值，但标题的使用也可以扩展为传递其他值，例如，路由器的通道名，文件组件的文件名等等。有效载荷可以是任何类型：标准 Java 对象、XML 或任何自定义或用户定义的值。它也可以是一个简单的信息共享有效载荷（例如，注册模块可以在新用户注册时通知审计模块），或者它可以是一个命令（例如，管理模块可以指示邮件服务通知所有注册课程的用户），或者它可以是一个事件（例如，在发送所有邮件后，邮件服务将事件返回给管理模块，指示所有邮件已发送，可以进行下一步）。

我们注意到了一个模式；两个组件通过这些消息进行通信——在正式术语中，我们称这些组件为消息端点。同样，我们可以观察到消息端点有两种类型：生产者端点和消费者端点。正如他们的名字所暗示的，一个生产者，比如`注册模块`，在给定示例中生成一个消息，而一个消费者则消耗它——比如给定示例中的`审计模块`。一个端点可以同时是生产者和消费者，例如，一个邮件服务。端点通常是智能组件，可以在将消息传递给下一个子系统之前验证消息，或者可以进行路由、过滤、聚合、转换等操作，以便消息能够符合下一环节的预期格式。

# **与消息通道一起工作**

我们定义了消息，并讨论了消息端点如何处理消息，那么消息通道在哪里呢？消息通道是实现企业应用集成（EAI）设计模式的实现，它解耦了端点。端点不需要了解彼此的类型；它们向通道注册，而通道负责安全地在端点之间传递消息。每个通道都有一个逻辑标识符——它可能是一个唯一的名称或 ID，通过它可以引用和注册。根据通道处理消息的方式，它们可以分为两大类：

+   点对点通道

+   发布-订阅通道

# 通道类型

在开始它们的实现之前，让我们首先看看以下类型的通道：

+   **点对点通道**：维护生产者和消费者之间一对一的关系。这些通道将消息发送给唯一的一个接收者。即使注册了多个接收者，消息也只会发送给其中的一个。这种通道类型可用于并行处理场景，允许多个消费者并行监听消息的可用性，但消息的发送只会针对单个消费者进行！

+   **发布-订阅通道**：这些通道将消息发送给在通道上注册的所有订阅者，从而实现生产者和消费者之间的一对多关系。可以将其比作每个订阅者都有自己的私有通道，在该通道上发送消息的副本。一旦被消费，它就会被丢弃。

让我们摆脱成语，窥视一下 Spring Integration 如何为所有这些组件提供支持——毕竟，这是一本关于 Spring Integration 的书，不是吗！

# Spring 通道实现

Spring Integration 为消息通道定义了一个顶级接口，任何具体的通道实现都应该实现这个接口，如下所示：

```java
public interface MessageChannel {
  boolean send(Message<?> message);
  boolean send(Message<?> message, long timeout);
}
```

`MessageChannel` 接口定义了两个版本的 `send` 方法——一个只接受 `Message` 作为参数，而另一个接受一个额外的参数（`timeout`）。`send` 方法如果在成功发送消息则返回 true；否则，如果在超时或由于某种原因发送失败，它返回 false。

此外，Spring Integration 为 `MessageChannel` 接口提供了一个子类型，以支持两种类型的通道：`PollableChannel` 和 `SubscribableChannel`。以下详细解释了这一点：

+   **可轮询通道**：此通道提供了两个版本的接收接口，一个不带任何参数，另一个提供指定 `timeout` 参数的选项。以下代码片段是接口声明：

    ```java
    public interface PollableChannel extends MessageChannel {
      Message<?> receive();
      Message<?> receive(long timeout);
    }
    ```

+   **可订阅通道**：此接口提供了订阅和取消订阅通道的方法。以下是可订阅通道的接口声明：

    ```java
    public interface SubscribableChannel extends MessageChannel {
      boolean subscribe(MessageHandler handler);
      boolean unsubscribe(MessageHandler handler);
    }
    ```

`MessageHandler`接口的实例作为参数传递给`subscribe`和`unsubscribe`方法。`MessageHandler`接口只暴露了一个方法，即`handleMessage`，用于处理消息：

```java
public interface MessageHandler {
  void handleMessage(Message<?> message) throws MessageException;
}
```

无论何时有消息到达通道，框架都会寻找消息处理器的实现，并将消息传递给实现者的`handleMessage`方法。

尽管 Spring Integration 定义了消息通道接口并允许用户提供自己的实现，但通常并不需要。Spring Integration 提供了许多可以即插即用的通道实现。

# 选择通道

让我们讨论一下 Spring Integration 提供的默认实现以及如何利用它们。

## 发布-订阅通道

这是发布-订阅模型通道的唯一实现。这个通道的主要目的是发送消息到注册的端点；这个通道不能被轮询。它可以如下声明：

```java
<int:publish-subscribe-channel id="pubSubChannel"/>
```

让我们讨论一下此行中的每个元素；本章的示例将使用此元素：

+   `int`：这是一个命名空间，声明了所有 Spring Integration 组件。如在第一章，*入门*中讨论的，Spring Integration 的 STS 可视化编辑器可以用来添加不同的 Spring Integration 命名空间。

+   `publish-subscribe-channel`：这是 Spring 暴露的类型。

+   `Id`：这是通道的唯一名称，通过它来引用通道。

要从代码中引用这些元素，我们可以使用：

```java
public class PubSubExample {
  private ApplicationContext ctx = null;
  private MessageChannel pubSubChannel = null;
  public PubSubChannelTest() {
    ctx = new ClassPathXmlApplicationContext("spring-integration-context.xml");
    pubSubChannel = ctx.getBean("pubSubChannel", MessageChannel.class);
  }
}
```

## 队列通道

还记得古老的数据结构中的队列概念吗？`QueueChannel`采用了相同的概念——它强制实施**先进先出**（**FIFO**）顺序，并且一个消息只能被一个端点消费。即使通道有多个消费者，这也是一种严格的一对一关系；只有一个消息将交付给它们中的一个。在 Spring Integration 中，它可以定义如下：

```java
<int:channel id="queueChannel">
  <queue capacity="50"/>
</int:channel>
```

一旦通道上有消息可用，它就会尝试将消息发送给订阅的消费者。元素`capacity`指示队列中保持未交付消息的最大数量。如果队列已满，这是由`capacity`参数确定的，发送者将被阻塞，直到消息被消费并在队列中腾出更多空间。另外，如果发送者已经指定了超时参数，发送者将等待指定的超时间隔——如果在超时间隔内队列中创建了空间，发送者会将消息放在那里，否则它会丢弃该消息并开始另一个。

### 提示

尽管容量参数是可选的，但绝不能省略；否则，队列将变得无界，可能会导致内存溢出错误。

## 优先级通道

队列强制 FIFO，但如果一个消息需要紧急注意并需要从队列中处理怎么办？例如，一个服务器健康监控服务可能会将健康审计发送到一个*审计服务*，但如果它发送了一个服务器下线事件，它需要紧急处理。这就是`PriorityChannel`派上用场的地方；它可以基于消息的优先级而不是到达顺序来选择消息。消息可以根据以下方式进行优先级排序：

+   通过在每条消息中添加一个`priority`头

+   通过向优先级通道的构造函数提供一个`Comparator<Message<?>>`类型的比较器

### 注意

默认是消息中的`priority`头。

让我们以以下优先级通道示例为例，并在其中注入一个比较器，该比较器将用于决定消息的优先级：

```java
<int:channel id="priorityChannel">
  <int:priority-queue capacity="50"/>
</int:channel>
```

比较器可以按照如下方式注入：

```java
<int:channel id="priorityChannel" datatype="com.example.result">
  <int:priority-queue comparator="resultComparator" capacity="50"/>
</int:channel>
```

## 会合通道

通常，我们希望有一个确认消息确实到达端点的回执。`rendezvousChannel`接口是队列通道的一个子类，用于此目的。生产者和消费者以阻塞模式工作。一旦生产者在通道上发送了一条消息，它就会被阻塞，直到那条消息被消费。同样，消费者在队列中到达一条消息之前也会被阻塞。它可以按照以下方式配置：

```java
<int:channel id="rendezvousChannel"/>
  <int:rendezvous-queue/>
</int:channel>
```

`RendezvousChannel`接口实现了一个零容量队列，这意味着在任何给定时刻，队列中只能存在一个消息。难怪没有容量元素。

## 直接通道

直接通道是 Spring Integration 默认使用的通道类型。

### 提示

当使用没有任何子元素的`<channel/>`元素时，它会创建一个`DirectChannel`实例（一个`SubscribableChannel`）处理器。

多个端点可以订阅直接通道的消息处理器；无论何时生产者将一条消息放在通道上，它都会被交付给订阅端点的唯一一个消息处理器。引入多个订阅者，限制将消息交付给唯一一个处理器，带来了新挑战——如何以及哪个处理器将被选择，如果处理器无法处理消息会发生什么？这就是负载均衡器和故障转移进入情景的地方。可以在该通道上定义一个负载均衡器，采用轮询交付策略：

```java
<int:channel id="newQuestions">
  <dispatcher failover="false" load-balancer="round-robin"/>
</int:channel>
```

这将按轮询方式将消息交付给订阅者。这是 Spring 预定义的唯一策略，但可以使用`interface`定义自定义策略：

```java
public interface LoadBalancingStrategy {
  public Iterator<MessageHandler> getHandlerIterator(
  Message<?> message, List<MessageHandler> handlers);
}
```

以下是一个引入自定义负载均衡器的示例：

```java
<int:channel id="lbChannel">
  <int:dispatcher load-balancer-ref="customLb"/>
</int:channel>

<bean id="customLb" class="com.chandan.CustomLoadBalancingImpl"/>
```

### 提示

**下载示例代码**

您可以在[`www.packtpub.com`](http://www.packtpub.com)下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)注册，以便将文件直接通过电子邮件发送给您。代码也可以从[`github.com/cpandey05/siessentials`](https://github.com/cpandey05/siessentials)拉取。

故障转移（Failover）是一个布尔值。如果将其设置为 true，那么如果第一个处理程序无法处理消息，则将尝试所有后续的处理程序。即使其中一个处理程序成功处理了消息，Spring Integration 也不会报告错误。只有当所有处理程序都失败时，它才会抛出异常。

### 提示

故障转移能力在实现事务传播或回退机制时非常有用。例如，如果数据库服务器失败，尝试在下一个处理程序中使用另一个后端服务器。

## 执行器通道

`ExecutorChannel`接口是一个点对点的消息通道。这非常类似于直接通道，不同之处在于可以使用自定义执行器来分派消息。让我们来看看配置：

```java
<int:channel id="results">
<int:dispatcher task-executor="resultExecutor"/></int:channel>
// define the executor
<bean id=" resultExecutor " class="com.example.ResultExecutor"/>
```

`com.example.ResultExecutor`接口是`java.util.concurrent.Executor`的一个实现。

因为生产者线程将消息传递给执行器实例并退却——消息的消费在执行器线程中处理，所以生产者和消费者之间无法建立事务链接。

就像直接通道一样，可以设置负载均衡策略和故障转移。默认值是启用故障转移的轮询策略：

```java
<int:channel id="results">
<int:dispatcher load-balancer="none" failover="false"
  taskexecutor="resultsExecutor"/>
</int:channel>
```

## 作用域通道

```java
thread scope:
```

```java
<int:channel id="threadScopeChannel" scope="thread">
  <int:queue />
</int:channel>
```

也可以定义自定义作用域，如下：

```java
<bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
  <property name="scopes">
    <map>
      <entry key="thread" value="org.springframework.context.support.SimpleThreadScope" />
    </map>
  </property>
</bean>
```

这是一个线程作用域通道的示例。如果我们观察到条目，为作用域定义了一个键值对。对于线程来说，键值对是`org.springframework.context.support.SimpleThreadScope`。它可以是任何 Spring 定义的或用户定义的作用域。

### 注意

以下是一些 Spring 实现的其它作用域：

+   `org.springframework.web.context.request.SessionScope`

+   `org.springframework.web.context.support.ServletContextScope`

+   `org.springframework.web.context.request.RequestScope`

+   `org.springframework.web.portlet.context.PortletContextScope`

## 数据类型通道

通道可以限制只接受具有特定类型有效负载的消息，例如数字、字符串或其他自定义类型。代码如下：

```java
<int:channel id="examMarksChannel" datatype="java.lang.Number"/>
```

也可以提供多种类型，如下：

```java
<int:channel id="stringOrNumberChannel" datatype="java.lang.String,java.lang.Number"/>
```

如果消息以除了前面代码中给出的格式以外的格式到达会发生什么？默认情况下，将抛出异常。然而，如果用例需要，我们可以定义转换器，尝试将传入的消息转换为可接受的格式。一个典型的用例是将字符串转换为整数。为了实现这一点，需要定义一个名为`integrationConversionService`的 bean，它是 Spring 的转换服务的实例，如下所示：

```java
public static class StringToIntegerConverter implements Converter<String, Integer> {
  public Integer convert(String source) {
    return Integer.parseInt(source);
  }
}
<int:converter ref="strToInt"/>

<bean id="strToInt" class="com.chandan.StringToIntegerConverter"/>
```

当解析`converter`元素时，如果尚未定义，它将按需创建`integrationConversionService` bean。有了这个转换器，如果一个字符串消息到达定义为整数的通道，将尝试将其转换为整数。

# 通道上的错误处理

Spring Integration 支持同步以及异步消息处理。在同步处理的情况下，根据返回值或通过捕获抛出的异常来处理错误场景相对容易；对于异步处理，事情会更加复杂。Spring 提供了诸如过滤器和中继器之类的组件，可以用来验证消息的有效性并根据那个采取行动。如果它无效，消息可以被路由到无效通道或重试通道。除此之外，Spring 提供了一个全局错误通道以及定义自定义错误通道的能力。以下几点涵盖了适当的错误通道：

+   需要定义一个错误通道。这可以通过以下方式完成：

    ```java
    <int:channel id="invalidMarksErrorChannel">
      <int:queue capacity="500"/>
    </int:channel>
    ```

+   需要添加一个名为`errorChannel`的头部到消息中。这是处理失败时`ErrorMessage`应该路由到的通道的名称。

+   如果消息处理失败，`ErrorMessage`将被发送到由头部`errorChannel`指定的通道。

+   如果消息不包含`errorChanel`头部，`ErrorMessage`将被路由到由 Spring Integration 定义的全局错误通道，即`errorChannel`。这是一个发布-订阅通道：

    ```java
    <int:gateway default-request-channel="questionChannel" service-interface="com.chandan.processQuestion" 
      error-channel="errorChannel"/>
    ```

# 持久化和恢复通道

我们讨论了各种各样的通道，但如果你注意到了，这些都是内存中的。系统崩溃怎么办？没有人想丢失数据。这就是持久`QueueChannel`发挥作用的地方——消息将被备份在由数据源定义的数据库中。如果系统崩溃，然后在恢复时，它将拉取数据库中的所有消息并将它们排队等待处理。这是使用 Spring 中的`MessageGroupStore`实现的。让我们快速看一下配置：

```java
<int:channel id="resultPersistenceChannel">
  <int:queue message-store="messageStore"/>
</int:channel>

<int-jdbc:message-store id="messageStore" data-source="someDataSource"/>
```

在此，消息存储被映射到由`someDataSource`定义的数据库。当消息到达时，现在将首先添加到`MessageStore`中。成功处理后，将从那里删除。

一旦我们谈论数据库，事务就会进入视野。那么如果轮询器配置了事务会怎样呢？在这种情况下，如果消息处理失败，事务将被回滚，消息将不会从队列中删除。

### 注意

如果支持事务行为，消息将在成功处理后从队列中删除。如果某些消息反复失败，随着时间的推移，队列中可能会积累陈旧的消息。必须仔细考虑这种消息的清理策略。

# 通道拦截器

拦截器模式可用于对从通道发送或接收的消息应用业务规则和验证。以下四种拦截器可用：

```java
public interface ChannelInterceptor {
  Message<?> preSend(Message<?> message, MessageChannel channel);
  void postSend(Message<?> message, MessageChannel channel, boolean sent);
  boolean preReceive(MessageChannel channel);
  Message<?> postReceive(Message<?> message, MessageChannel channel);
}
```

以下是由`ChannelInterceptor`接口暴露的方法：

+   `preSend`: 这是在消息发送之前调用的。如果消息被阻止发送，应返回 null 值。

+   `postSend`: 在尝试发送消息之后调用。它表示消息是否成功发送。这可以用于审计目的。

+   `preReceive`: 仅当通道是轮询的时适用，当组件对通道调用`receive()`时调用，但在实际从该通道读取消息之前。它允许实现者决定通道是否可以向调用者返回消息。

+   `postReceive`: 这与`preReceive`类似，仅适用于轮询通道。在从通道读取消息但在将其返回给调用`receive()`的组件之后调用。如果返回 null 值，则没有接收消息。这允许实现者控制轮询器实际接收了什么（如果有的话）。

# 总结

本章内容相对较长，我们讨论了消息通道模式、不同类型的通道以及 Spring 提供的默认通道实现。我们还介绍了负载均衡、故障转移、在消息通道上的错误处理、消息持久化以及添加拦截器。所有这些概念都是构建可靠和可扩展解决方案的核心，我们将在接下来的章节中看到其实际实现，届时我们将讨论 Spring Integration 组件，如服务激活器、网关、延迟器等，这些组件用于处理消息。
