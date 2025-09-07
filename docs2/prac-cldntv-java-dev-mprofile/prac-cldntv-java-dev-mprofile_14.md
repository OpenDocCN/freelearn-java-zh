# *第十章*：响应式云原生应用程序

到目前为止，我们主要讨论了采用具有明确定义的输入和输出的**命令式编程**的传统云原生应用程序。命令式编程是最古老的编程范式。使用这种范式的应用程序是通过一个明确定义的指令序列构建的，这使得它更容易理解。其架构要求连接服务是预定义的。

然而，有时，云原生应用程序不知道它应该调用哪些服务。它的目的可能只是发送或接收消息或事件，保持响应性和反应性。因此，命令式编程不再适用于这些类型的应用程序。在这种情况下，你需要依赖**响应式编程**并使用事件驱动架构来实现响应式、响应性和消息驱动的应用程序。我们将在本章中讨论响应式云原生应用程序。

首先，你将学习命令式和响应式应用程序之间的区别。然后我们将讨论如何使用**MicroProfile Reactive Messaging 2.0**创建响应式云原生应用程序。

我们将涵盖以下主题：

+   区分命令式和响应式应用程序

+   使用**MicroProfile Context Propagation**来改进异步编程

+   使用**MicroProfile Reactive Messaging**开发响应式云原生应用程序

为了完全理解本章，你应该具备 Java 多线程、`CompletableFuture`类和 Java 8 的`CompletionStage`接口的知识。

到本章结束时，你应该能够理解什么是响应式云原生应用程序，如何创建避免阻塞 I/O 问题的响应式云原生应用程序，以及如何利用像 Apache Kafka 这样的消息库。

# 区分命令式和响应式应用程序

在开发命令式应用程序时，应用程序开发者定义如何执行任务。你可能一开始设计一个同步应用程序。然而，为了处理重负载和提高性能，你可能会考虑从同步编程切换到异步编程，通过并行执行多个任务来加速。在使用同步编程时，一旦遇到阻塞 I/O，线程必须等待，并且在该线程上无法执行其他任务。然而，在异步编程的情况下，如果有一个线程被阻塞，可以调度多个线程来执行其他任务。

异步编程可以调度多个线程，但它并不能解决阻塞 I/O 问题。如果有阻塞，最终应用程序将消耗所有线程。因此，应用程序将耗尽资源。命令式编程的一个特点是，一个应用程序需要知道要与之交互的服务。在某些情况下，它可能不知道也不关心下游服务。因此，命令式编程不适用于这种情况。在这里，反应式编程就派上用场了。

反应式编程是一种关注数据流和变化传播的范式。这种范式用于构建一个消息驱动、弹性且响应式的云原生应用程序。

反应式应用程序采用了**反应式宣言**的设计原则。反应式宣言([`www.reactivemanifesto.org/`](https://www.reactivemanifesto.org/))概述了以下四个特性：

+   **响应性**：应用程序在所有条件下都能及时响应。

+   **弹性**：系统在各种负载下保持响应性，可以根据需求进行扩展或缩减。

+   **弹性**：系统在所有情况下都具有弹性。

+   **消息驱动**：该系统依赖于异步消息作为组件之间的通信渠道。

反应式应用程序使用异步编程来实现时间解耦。正如我们之前提到的，异步编程涉及调度更多线程。每个线程通常需要一个`java.util.concurrent.ForkJoinPool`类来创建新线程，新调度线程不会关联任何上下文。因此，为了在不同的新线程上继续一个未完成的任务，你需要从先前的线程将一些上下文推送到新线程以继续任务执行。MicroProfile 上下文传播可以用来实现这一点，我们将在下一节中讨论。

# 使用 MicroProfile 上下文传播来管理上下文

MicroProfile 上下文传播([`download.eclipse.org/microprofile/microprofile-context-propagation-1.2/`](https://download.eclipse.org/microprofile/microprofile-context-propagation-1.2/))定义了一种从当前线程传播上下文到新线程的机制。上下文类型包括以下几种：

+   `java:comp`、`java:module`和`java:app`。

+   `SessionScoped`和`ConversationScoped`，在新工作单元中（如新的`CompeletionStage`）仍然有效。

+   **安全性**：这包括与当前线程关联的凭证。

+   **事务**：这是与当前线程关联的活跃事务作用域。通常不期望传播此上下文，而是清除它。

除了上述上下文之外，如果需要，应用程序可以引入自定义上下文。

为了传播上述上下文，本规范引入了两个接口：

+   `ManagedExecutor`：此接口提供了一种异步执行机制，用于定义线程上下文的传播。

+   `ThreadContext`：此接口允许更精细地控制线程上下文的捕获和传播。您可以将此接口与特定的函数关联。

在以下小节中，我们将简要讨论如何使用上述两个接口将当前线程关联的上下文传播到各种工作单元，例如`CompletionStage`、`CompletableFuture`、`Function`和`Runnable`。

## 使用`ManagedExecutor`传播上下文

`ManagedExecutor`与其他已知执行器（如`ForkJoinPool`）不同，后者没有传播上下文的设施。要使用`ManagedExecutor`传播上下文，您需要创建一个`ManagedExecutor`实例，这可以通过构建器模式实现：

```java
ManagedExecutor executor = ManagedExecutor.builder()
        .cleared(ThreadContext.TRANSACTION)
        .propagated(ThreadContext.ALL_REMAINING)
        .maxAsync(10)
        .build();
```

上述代码片段用于创建一个`ManagedExecutor`的执行器对象，该对象清除`Transaction`上下文并传播所有其他剩余的上下文。此执行器支持最多`10`个并发执行。然后，您可以调用执行器的一些方法来创建一个`CompletableFuture`对象，用于异步编程，如下所示：

```java
CompletableFuture<Long> stage1 =   executor.newIncompleteFuture();
stage1.thenApply(function1).thenApply(function2);
stage1.completeAsync(supplier); 
```

上述代码片段演示了使用`executor`对象创建一个不完整的`CompletableFuture` `stage1`，然后执行`function1`。`function1`完成后，将执行`function2`。最后，将使用给定的`supplier`函数完成未来的`stage1`。

除了`ManagedExecutor`之外，另一种传播上下文的方式是使用`ThreadContext`接口。让我们在下一节中更详细地讨论它。

## 使用`ThreadContext`传播上下文

`ThreadContext`提供了对捕获和恢复上下文的精细控制。要使用`ThreadContext`，您需要创建一个`ThreadContext`实例，该实例可以通过以下方式通过构建器模式创建：

```java
ThreadContextthreadContext = ThreadContext.builder()  .propagated(ThreadContext.APPLICATION, ThreadContext     .SECURITY).unchanged().cleared(ThreadContext       .ALL_REMAINING).build();
```

上述代码片段创建了一个`ThreadContext`实例，它从当前线程传播应用程序和安全性上下文，并清除其他上下文。之后，您可以通过调用`threadContext.withContextCapture()`方法创建一个`CompletionStage`实例：

```java
CompletionStage stage =   threadContext.withContextCapture(someMethod);
stage.thenApply(function1).thenAccept(aConsumer);
```

在上述代码片段中，`function1`和`aConsumer`函数都将从当前线程继承应用程序和安全性上下文。

或者，您可以从一个`threadContext`对象创建一个上下文函数，然后将此上下文函数提供给`CompletableFuture`，如下所示：

```java
Function<String, String> aFunction =   threadContext.contextualFunction(function1);
CompletableFuture.thenApply(aFunction)    .thenAccept(aConsumer);
```

在前面的代码片段中，当`aFunction`执行时，运行此`aFunction`的线程将从其父线程继承应用程序和安全上下文，这意味着该线程将能够执行与创建`aFunction`对象的线程类似的功能，而`aConsumer`函数将不会从其父线程继承应用程序和安全上下文。为了使用上下文传播，你需要使 API 对你的应用程序可用。

### 使 MicroProfile 上下文传播 API 可用

MicroProfile 上下文传播 API JAR 可以为 Maven 和 Gradle 项目提供。如果你创建了一个 Maven 项目，你可以直接将以下内容添加到你的`pom.xml`文件中：

```java
<dependency>
    <groupId>org.eclipse.microprofile.context-      propagation</groupId>
    <artifactId>microprofile-context-propagation-      api</artifactId>
    <version>1.2</version>
</dependency>
```

另外，如果你创建了一个 Gradle 项目，你需要添加以下依赖项：

```java
dependencies {
  providedCompile org.eclipse.microprofile.context-    propagation: microprofile-context-propagation-api:1.2
}
```

我们简要讨论了如何捕获和恢复上下文作为异步编程的一部分。如前所述，异步编程并不解决阻塞 I/O 问题，而是通过调度新线程来绕过这个问题。为了解决阻塞 I/O 问题，你需要考虑构建一个响应式应用程序。在下一节中，我们将讨论使用 MicroProfile 响应式消息传递来帮助你构建响应式应用程序。

# 使用 MicroProfile 响应式消息传递构建响应式应用程序

`@Outgoing`注解用于发布消息，`@Incoming`用于消费消息。以下图示说明了消息如何从发布者（**方法 A**）传输到消费者（**方法 B**）。消息可以被发送到消息存储，例如 Apache Kafka、MQ 等，然后将被传递到消费者，如**方法 B**：

![图 10.1 – 消息流](img/Figure 10.1 – Messaging flow)

![图 10.1 – 消息流](img/B17377_10_01.jpg)

图 10.1 – 消息流

在响应式消息传递中，CDI 豆用于产生、处理和消费消息。这些消息可以通过远程代理或 Apache Kafka、MQ 等各种消息传输层进行发送和接收。让我们讨论 MicroProfile 响应式消息传递的一些关键元素：消息、消息确认、通道、消息消费、消息生产、消息处理和`Emitter`。

## 消息

**消息**是包裹在信封中的信息片段。此外，此信息片段可以包括确认逻辑，可以是正面的或负面的。以下是一些产生消息的方法：

+   `Message.of(P p)`: 此方法包装给定的有效负载`p`，不提供任何确认逻辑。

+   `Message.of(P p, Supplier<CompletionStage<Void>> ack)`: 此方法包装给定的有效负载`p`并提供`ack`确认逻辑。

+   `Message.of(P p, Supplier<CompletionStage<Void>> ack, Function<Throwable, CompletionSTage<Void>>> nack)`: 此方法包装给定的有效负载`p`，提供`ack`确认逻辑和`nack`否定确认逻辑。

或者，如果你有一个`Message`对象，你可以通过获取其有效载荷并可选地提供新的积极或消极确认来从它创建一个新的`Message`对象，如下所示：

```java
Message<T> newMessage =   aMessage.withPayload(payload).withAck(...).withNack(...) 
```

上述代码片段从`aMessage`创建`newMessage`，并提供了新的有效载荷、积极确认和消极确认逻辑。

你可能想知道如何执行消息确认和消极确认。我们将在下一节中更详细地讨论它们。

## 消息确认

所有消息都必须进行积极或消极的确认，可以使用 MicroProfile Reactive Messaging 实现显式或隐式地进行确认。

有两种不同的确认类型：**积极确认**和**消极确认**。积极确认表示消息已成功处理，而消极确认表示消息处理失败。

可以通过`@Acknowledgment`注解显式指定确认。此注解与`@Incoming`注解一起使用。你可以指定以下三种确认策略之一：

+   `Message#ack()`用于确认接收到的消息。

+   **@Acknowledgment(PRE_PROCESSING)**：Reactive Messaging 实现，在执行注解方法之前确认消息。

+   **@Acknowledgment(POST_PROCESSING)**：Reactive Messaging 实现，在方法完成且方法不发出数据或发出的数据被确认后确认消息。

以下是一个手动确认的示例。`consume()`方法通过在`msg`对象上调用`msg.ack()`方法手动确认消息消费：

```java
@Incoming("channel-c")
@Acknowledgment(Acknowledgment.Strategy.MANUAL)
public CompletionStage<Void> consume(Message<I> msg) {
  System.out.println("Received the message: " +     msg.getPayload());
  return msg.ack();
}
```

如果缺少`@Acknowledgment`注解，你认为应该使用哪种确认策略？这个答案取决于应用`@Incoming`注解的方法签名。默认的确认策略如下：

+   如果方法参数或返回类型包含`message`类型，则默认确认策略为`MANUAL`。

+   否则，如果方法仅注解了`@Incoming`，则默认确认策略为`POST_PROCESSING`。

+   最后，如果方法同时注解了`@Incoming`和`@Outgoing`，则默认确认策略为`PRE_PROCESSING`。

现在我们已经涵盖了消息及其确认策略的必要概念。你可能想知道消息将被发送到何处或从何处消费，即其目的地或源。这些被称为通道，我们将在下一节中讨论。

## 通道

**通道**是一个表示消息源或目的地的字符串。MicroProfile Reactive Messaging 有两种类型的通道：

+   **内部通道**：这些通道是应用本地的，允许在消息源和消息目的地之间进行多步处理。

+   **外部通道**：这些通道连接到远程代理或各种消息传输层，如 Apache Kafka、AMQP 代理或其他消息传递技术。

消息从上游通道流向下游通道，直到达到消费者进行消息消费。接下来，我们将讨论这些消息是如何被消费的。

## 使用`@Incoming`进行消息消费

一个带有`@Incoming`注解的 CDI bean 上的方法可以消费消息。以下示例从`channel-a`通道消费消息：

```java
@Incoming("channel-a")
CompletionStage<Void> method(Message<I> msg) {
    return message.ack();
}
```

当消息发送到`channel-a`通道时，将调用此方法。此方法确认接收到的消息。支持的数据消费方法签名如下：

+   `Subscriber<Message<I>> consume(); Subscriber<I> consume();`

+   `SubscriberBuilder<Message<I>, Void> consume(); SubscriberBuilder<I, Void> consume();`

+   `void consum(I payload);`

+   `CompletionStage<Void> consume(Message<I> msg); CompletionStage<?> consume(I payload);`

在上述列表中，`I`是传入的有效负载类型。

接收消息的另一种方式是注入`org.reactivestreams.Publisher`或`org.eclipse.microprofile.reactive.streams.operators.PublisherBuilder`，并使用`@Channel`注解，如下所示：

```java
@Inject
@Channel("channel-d")
private Publisher<String> publisher;
```

上述代码片段意味着一个`publisher`实例将被连接到`channel-d`通道。然后消费者可以直接从发布者接收消息。

我们已经讨论了消息流和消息消费。接下来，我们将看看消息是如何生成的。

## 使用`@Outgoing`进行消息生产

带有`@Outgoing`注解的 CDI bean 上的方法是一个消息生产者。以下代码片段演示了消息生产：

```java
@Outgoing("channel-b")
public Message<Integer> publish() {
    return Message.of(123);
}
```

在上述代码片段中，对于每个消费者请求都会调用`publish()`方法，并将`123`消息发布到`channel-b`通道。每个应用程序中只能有一个发布者使用指定通道名的`@Outgoing`，这意味着只能有一个发布者可以向特定通道发布消息。否则，在应用程序部署期间将发生错误。消息发布到通道后，消费者可以从中消费消息。支持的数据生产方法签名如下：

+   `Publisher<Message<O>> produce(); Publisher <O> produce();`

+   `PublisherBuilder<Message<O>> produce (); Publisher Builder<O> produce();`

+   `Message<O> produce(); O produce();`

+   `CompletionStage<Message<O>> produce(); CompletionStage<O> produce();`

在上述列表中，`O`是传出有效负载类类型。

一个方法可以作为消息消费者和消息生产者。这种类型的方法被称为**消息处理器**。

## 使用`@Incoming`和`@Outgoing`进行消息处理

消息处理器既是消息生产者也是消费者，这意味着它具有`@Incoming`和`@Outgoing`注解。让我们看看这个方法：

```java
@Incoming("channel-a")
@Outgoing("channel-b")
public Message<Integer> process(Message<Integer> message) {
    return message.withPayload(message.getPayload() + 100);
}
```

`process()`方法从`channel-a`通道接收整数消息，然后加上`100`。之后，它将新的整数发布到`channel-b`通道。处理数据支持的方法签名如下：

+   `Processor<Message<I>, Message<O>> process(); Processor<I, O> process();`

+   `ProcessorBuilder<Message<I>, Message<O>> process(); ProcessorBuilder<I, O> process();`

+   `PublisherBuilder<Message<O>> process(Message<I> msg); PublisherBuilder<O> process(I payload);`

+   `PublisherBuilder<Message<O>> process(PublisherBuilder<Message<I>> publisherBuilder); PublisherBuilder<O> process(PublisherBuilder<I> publisherBuilder);`

+   `Publisher<Message<O>> method(Publisher<Message<I>> publisher); Publisher<O> method(Publisher<I> publisher);`

+   `Message<O> process(Message<I> msg); O process(I payload);`

+   `CompletionStage<Message<O>> process(Message<I> msg); CompletionStage<O> process(I payload);`

在前面的列表中，`I`是传入负载类类型，`O`是传出负载类类型。

到目前为止，消息消费和生产是 CDI 豆上的方法。这些 CDI 豆可以是`Dependent`或`ApplicationScoped`。我们已经在*第四章**，开发云原生应用*中介绍了 CDI。

我们已经介绍了通过 CDI 豆上的方法进行消息发布和消息消费，当你的应用程序启动并运行时，这些方法将由 Reactive Messaging 实现自动触发。你可能想知道，如果你想在端点被触发时发布一些消息，应该怎么做。我们将在下一节讨论如何做到这一点。

## 使用 Emitter 发布消息

为了从 JAX-RS 资源发布消息，你可以注入一个`Emitter`对象，然后调用`send()`方法。让我们看看以下示例：

```java
@Inject @Channel("channel-c")
private Emitter<String> emitter;
public void publishMessage() {
    emitter.send("m");
    emitter.send("n");
    emitter.complete();
}
```

在上述代码片段中，首先你可以注入一个带有目标通道的`Emitter`。然后，你可以通过调用`emitter.send()`方法发送消息。本例直接发送一个消息负载。你可以通过包装负载来发送消息，具体细节如下：

```java
@Inject @Channel("channel-e")
private Emitter<String> emitter;
public void publishMessage() {
    emitter.send(Message.of("m");
    emitter.send(Message.of("n");
    emitter.send(Message.of("q")
}
```

通常，消息发布的速度可能不会与消息消费的速度相同。使用`Emitter`发布消息时，`@OnOverflow`注解用于处理反压。以下是一个示例，演示如何使用缓冲策略来处理反压：

```java
@Inject @Channel("channel-d")
@OnOverflow(value=OnOverflow.Strategy.BUFFER,   bufferSize=100)
private Emitter<String> emitter;
public void publishMessage() {
    emitter.send("message1");
    emitter.send("message2");
    emitter.complete();
}
```

在前面的代码片段中，一个`Emitter`对象`emitter`使用容量为`100`个元素的缓冲区反压策略连接到`channel-d`。`emitter`对象发送了两个消息并完成了它们。`@OnOverflow`注解支持以下配置，如下表所示：

表 10.1 – 反压策略

我们已经学习了如何使用`@Outgoing`和`@Incoming`注解进行消息的生产和消费。MicroProfile Reactive Messaging 通过反应式消息连接器将出站和入站通道连接到外部技术，如 Apache Kafka、WebSocket、AMQP、JMS 和 MQTT。连接是通过反应式消息连接器实现的。我们将在下一节详细讨论连接器。

## 使用连接器桥接到外部消息技术

连接器可以作为发布者、消费者或处理器。它是一个 CDI bean，实现了 MicroProfile Reactive Messaging 的两个接口`IncomingConnectorFactory`和`OutgoingConnectorFactory`，分别用于接收消息和分发消息。Reactive Messaging 实现为支持的消息技术（如 Kafka、MQTT、MQ 等）提供开箱即用的连接器。但是，如果您需要的连接器未由实现者提供，您可以自己创建一个连接器。以下是一个连接到 Apache Kafka 的连接器示例：

```java
@ApplicationScoped
@Connector("my.kafka")
public class KafkaConnector implements   IncomingConnectorFactory, OutgoingConnectorFactory {
    // ...
}
```

一旦定义了连接器，接下来显示的进一步配置是必需的，以便将您的云原生应用程序中的通道与连接器桥接的外部消息技术相匹配。在以下配置中，`channel-name`必须与`@Incoming`或`@Outgoing`注解中的值相匹配，而`attribute`可以是任何类型的字符串：

+   `mp.messaging.incoming.[channel-name].[attribute]`: 该属性用于将带有`@Incoming`注解的通道映射到由相应消息技术提供的外部目标。

+   `mp.messaging.outgoing.[channel-name].[attribute]`: 这个属性用于将带有`@Outgoing`注解的通道映射到由相应消息技术提供的外部目标。

+   `mp.messaging.connector.[connector-name].[attribute]`: 这个属性用于指定连接器的详细信息。

如果您的云原生应用程序连接到 Apache Kafka，您可能需要为以下消费者方法提供以下配置：

```java
@Incoming("order")
CompletionStage<Void> method(Message<I> msg) {
    return message.ack();
}
```

在以下配置中，`mp.messaging.incoming.order.connector`属性指定了连接器名称为`liberty-kafka`，然后使用`mp.messaging.connector.liberty-kafkabootstrap.servers`属性进一步指定该连接器的配置。然后通过`mp.messaging.incoming.order.topic`属性将`topic-order` Kafka 主题映射到通道`order`：

```java
mp.messaging.incoming.order.connector=liberty-kafka
mp.messaging.connector.liberty-kafka.bootstrap.  servers=localhost:9092
mp.messaging.incoming.order.topic=topic-order
```

我们已经介绍了 MicroProfile Reactive Messaging。现在让我们将其全部整合起来。如果您需要创建一个从事件流系统（如 Apache Kafka）消费消息的消费者，您只需创建一个 CDI 实例，并编写一个带有 `@Incoming` 注解的方法来连接特定的通道。同样，如果您需要发送消息，您将需要创建一个 CDI 实例，并编写一个带有 `@Outging` 注解的方法来连接到生产者通道。最后，您配置通道，如前所述，以声明通道连接到 Apache Kafka 连接器。Open Liberty 提供了 `liberty-kafka` Kafka 连接器。本 Open Liberty 指南 ([`openliberty.io/guides/microprofile-reactive-messaging.html`](https://openliberty.io/guides/microprofile-reactive-messaging.html)) 展示了如何创建 Java 微服务。

要使用 MicroProfile Reactive Messaging 的 API，您需要指定如后所述的 Maven 或 Gradle 依赖项。

### 使 MicroProfile Reactive Messaging API 可用

MicroProfile Reactive Messaging API JAR 可以用于 Maven 和 Gradle 项目。如果您创建了一个 Maven 项目，您可以直接将以下内容添加到您的 `pom.xml` 文件中：

```java
<dependency>
  <groupId>
    org.eclipse.microprofile.reactive.messaging
  </groupId>
  <artifactId>
    microprofile-reactive-messaging-api
  </artifactId>
  <version>2.0</version>
</dependency>
```

另外，如果您创建了一个 Gradle 项目，您需要添加以下依赖项：

```java
dependencies {
  providedCompile org.eclipse.microprofile.reactive     .messaging: microprofile-reactive-messaging-api:2.0
}
```

您现在已经学会了如何在需要与消息传递技术交互并需要消息驱动架构时创建响应式云原生应用程序。

# 摘要

在本章中，我们学习了命令式和响应式应用程序之间的区别。我们简要讨论了如何使用 MicroProfile Context Propagation 来传播异步编程的上下文，然后介绍了 MicroProfile Reactive Messaging 概念，讨论了如何使用 Reactive Messaging 来创建一个响应式云原生应用程序。通过本章，你现在将能够将构建的应用程序与您选择的如 Apache Kafka 之类的消息传递技术连接起来。你现在也理解了，每当您需要创建一个消息驱动应用程序时，您应该考虑使用 MicroProfile Reactive Messaging。

在下一章中，我们将介绍 MicroProfile GraphQL，学习如何在您的云原生应用程序中使用 GraphQL 来提高性能，如果您需要频繁执行查询。

![表 10.1 – 反压策略

![img/Table_01.png]
