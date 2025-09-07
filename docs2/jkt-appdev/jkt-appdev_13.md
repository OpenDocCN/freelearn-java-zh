# 13

# Jakarta Messaging

**Jakarta Messaging**为 Jakarta EE 应用程序之间发送消息提供了一种机制。Jakarta Messaging 应用程序不直接通信；相反，消息生产者将消息发送到目的地，而消息消费者从目的地接收消息。

当使用**点对点**（**PTP**）消息域时，消息目的地是消息队列，或者当使用**发布/订阅**（**pub/sub**）消息域时，消息目的地是消息主题。

在本章中，我们将介绍以下主题：

+   与消息队列一起工作

+   与消息主题一起工作

注意

本章的示例源代码可以在 GitHub 上找到，地址为[`github.com/PacktPublishing/Jakarta-EE-Application-Development/tree/main/ch13_src`](https://github.com/PacktPublishing/Jakarta-EE-Application-Development/tree/main/ch13_src)。

# 与消息队列一起工作

如我们之前提到的，当我们的 Jakarta Messaging 代码使用 PTP 消息域时，会使用消息队列。对于 PTP 消息域，通常有一个消息生产者和一个消息消费者。消息生产者和消息消费者不需要同时运行以进行通信。消息生产者放入消息队列中的消息将保持在消息队列中，直到消息消费者执行并从队列中请求这些消息。

## 向消息队列发送消息

以下示例说明了如何向消息队列添加消息：

```java
package com.ensode.jakartaeebook.ptpproducer;
//imports omitted for brevity
@JMSDestinationDefinition(
    name = "java:global/queue/JakartaEEBookQueue",
    interfaceName = "jakarta.jms.Queue"
)
@Named
@RequestScoped
public class MessageSender {
  @Resource
  private ConnectionFactory connectionFactory;
  @Resource(mappedName = "java:global/queue/JakartaEEBookQueue")
  private Queue queue;
  public void produceMessages() {
    JMSContext jmsContext = connectionFactory.createContext();
    JMSProducer jmsProducer = jmsContext.createProducer();
    String msg1 = "Testing, 1, 2, 3\. Can you hear me?";
    String msg2 = "Do you copy?";
    String msg3 = "Good bye!";
    jmsProducer.send(queue, msg1);
    jmsProducer.send(queue, msg2);
    jmsProducer.send(queue, msg3);
  }
}
```

注意

本章中的大多数示例都是作为**上下文和依赖注入**（**CDI**）豆实现的。有关 CDI 的解释，请参阅*第二章*。

类级别的`@JMSDestinationDefinition`注解定义了一个 Jakarta Messaging 目的地，我们的消息将被放置在这里。此注解有两个必需的属性，`name`和`interfaceName`。`@JMSDestinationDefinition`的`name`属性定义了一个`interfaceName`，它指定了 Jakarta Messaging 目的地接口；对于 PTP 消息，此值必须始终是`produceMessages()`方法，该方法从使用 Facelets 实现的 Jakarta Faces 页面上的`commandButton`调用前面的类。为了简洁，我们不会显示此页面的 XHTML 标记。本章的代码下载包包含完整的示例。

`MessageSender`类中的`produceMessages()`方法执行所有必要的步骤以将消息发送到消息队列。

此方法首先执行的操作是通过在注入的`jakarta.jms.ConnectionFactory`实例上调用`createContext()`方法来创建一个`jakarta.jms.JMSContext`实例。请注意，装饰连接工厂对象的`@Resource`注解的`mappedName`属性与`@JMSDestinationDefinition`注解的`name`属性相匹配。在幕后，使用此名称进行 JNDI 查找以获取连接工厂对象。

接下来，我们通过在刚刚创建的`JMSContext`实例上调用`createProducer()`方法来创建`jakarta.jms.JMSProducer`实例。

在获取`JMSProducer`实例后，代码通过调用其`send()`方法发送一系列文本消息。此方法将消息目的地作为其第一个参数，将包含消息文本的`String`作为其第二个参数。

在`JMSProducer`中，`send()`方法有几个重载版本。我们在示例中使用的是一种方便的方法，它创建`jakarta.jms.TextMessage`实例并将其文本设置为方法调用中提供的第二个参数的`String`。

虽然前例只向队列发送文本消息，但我们并不局限于这种消息类型。Jakarta Messaging 提供了多种类型的消息，这些消息可以被 Jakarta Messaging 应用程序发送和接收。所有消息类型都在`jakarta.jms`包中定义为接口。*表 13.1*列出了所有可用的消息类型：

| **消息类型** | **描述** |
| --- | --- |
| `BytesMessage` | 允许发送字节数组作为消息。`JMSProducer`有一个方便的`send()`方法，它将字节数组作为其参数之一。此方法在发送消息时即时创建`jakarta.jms.BytesMessage`实例。 |
| `MapMessage` | 允许发送实现`java.util.Map`接口的消息。`JMSProducer`有一个方便的`send()`方法，它将`Map`作为其参数之一。此方法在发送消息时即时创建`jakarta.jms.MapMessage`实例。 |
| `ObjectMessage` | 允许发送实现`java.io.Serializable`接口的任何 Java 对象作为消息。`JMSProducer`有一个方便的`send()`方法，它将实现`java.io.Serializable`接口的类的实例作为其第二个参数。此方法在发送消息时即时创建`jakarta.jms.ObjectMessage`实例。 |
| `StreamMessage` | 允许发送字节数组作为消息。与`BytesMessage`不同，它存储添加到流中的每个原始类型的类型。 |
| `TextMessage` | 允许发送`java.lang.String`作为消息。如前例所示，`JMSProducer`有一个方便的`send()`方法，它将`String`作为其第二个参数。此方法在发送消息时即时创建`jakarta.jms.TextMessage`实例。 |

表 13.1 – Jakarta Messaging 消息类型

注意

关于所有上述消息类型的更多信息，请参阅[`jakarta.ee/specifications/messaging/3.0/apidocs/jakarta/jms/package-summary`](https://jakarta.ee/specifications/messaging/3.0/apidocs/jakarta/jms/package-summary)的`JavaDoc`文档。

## 从消息队列中检索消息

当然，如果没有任何接收者，从队列中发送消息是没有意义的。以下示例说明了如何从消息队列中检索消息：

```java
package com.ensode.jakartaeebook.ptpconsumer;
//imports omitted for brevity
@JMSDestinationDefinition(
  name = "java:global/queue/JakartaEEBookQueue",
  interfaceName = "jakarta.jms.Queue"
)
@Named
@RequestScoped
public class MessageReceiver implements Serializable {
  @Resource
  private ConnectionFactory connectionFactory;
  @Resource(mappedName =
    "java:global/queue/JakartaEEBookQueue")
private Queue queue;
  private static final Logger LOG =
    Logger.getLogger(MessageReceiver.class.getName());
  public void receiveMessages() {
    String message;
    boolean goodByeReceived = false;
    JMSContext jmsContext = connectionFactory.
      createContext();
    JMSConsumer jMSConsumer =
      jmsContext.createConsumer(queue);
    while (!goodByeReceived) {
      message = jMSConsumer.receiveBody(String.class);
      LOG.log(Level.INFO, "Received message: {0}", message);
      if (message.equals("Good bye!")) {
        goodByeReceived = true;
      }
    }
  }
}
```

就像在之前的示例中一样，我们通过`@JMSDestinationDefinition`注解定义了一个目的地，并且通过使用`@Resource`注解注入了`jakarta.jms.ConnectionFactory`和`jakarta.jms.Queue`的实例。

在我们的代码中，我们通过调用`ConnectionFactory`的`createContext()`方法来获取`jakarta.jms.JMSContext`的实例，就像在之前的示例中一样。

在这个示例中，我们通过在我们的`JMSContext`实例上调用`createConsumer()`方法来获取`jakarta.jms.JMSConsumer`的实例。

通过在我们的`JMSConsumer`实例上调用`receiveBody()`方法来接收消息。此方法只接受我们期望的消息类型作为其唯一参数（在我们的示例中为`String.class`）。此方法返回其参数指定的类型的对象（在我们的示例中为`java.lang.String`的实例）。一旦消息被`JMSConsumer.receiveBody()`消费，它就会从队列中移除。

在这个特定的例子中，我们将这个方法调用放在了一个`while`循环中，因为我们期望一个消息会告诉我们没有更多的消息到来。具体来说，我们正在寻找包含文本“再见！”的消息。一旦我们收到这个消息，我们就跳出循环并继续处理。在这个特定的案例中，没有更多的处理要做，因此执行在跳出循环后结束。

执行代码后，我们应该在服务器日志中看到以下输出：

```java
Waiting for messages...
Received the following message: Testing, 1, 2, 3\. Can you hear me?
Received the following message: Do you copy?
Received the following message: Good bye!
```

当然，这假设之前的示例已经执行，并且它已经将消息放入了消息队列。

注意

本节讨论的消息处理的一个缺点是消息处理是同步的。在 Jakarta EE 环境中，我们可以通过使用消息驱动豆（如*第十二章*中讨论的那样）来异步处理消息。

## 浏览消息队列

Jakarta Messaging 提供了一种方法来浏览消息队列，而实际上并不从队列中移除消息。以下示例说明了如何做到这一点：

```java
package com.ensode.jakartaeebook.queuebrowser;
//imports omitted for brevity
//Messaging destination definition annotation omitted
@Named
@RequestScoped
public class MessageQueueBrowser {
  @Resource
  private ConnectionFactory connectionFactory;
  @Resource(mappedName =
    "java:global/queue/JakartaEEBookQueue")
  private Queue queue;
  private static final Logger LOG =
    Logger.getLogger(MessageQueueBrowser.class.getName());
  public void browseMessages() throws JMSException {
    Enumeration messageEnumeration;
    TextMessage textMessage;
    JMSContext jmsContext =
      connectionFactory.createContext();
    QueueBrowser browser = jmsContext.createBrowser(queue);
    messageEnumeration = browser.getEnumeration();
    LOG.log(Level.INFO, "messages in the queue:");
    while (messageEnumeration.hasMoreElements()) {
      textMessage = (TextMessage) messageEnumeration.
        nextElement();
      LOG.log(Level.INFO, textMessage.getText());
    }
  }
}
```

如我们所见，浏览消息队列的流程非常直接。我们以通常的方式获取连接工厂、队列和上下文，然后在上下文对象上调用`createBrowser()`方法。此方法返回`jakarta.jms.QueueBrowser`接口的实现。此接口包含一个`getEnumeration()`方法，我们可以调用它来获取包含队列中所有消息的`Enumeration`。要检查队列中的消息，我们只需遍历这个枚举并逐个获取消息。在我们讨论的示例中，我们简单地调用队列中每个消息的`getText()`方法。

现在我们已经看到了如何使用 PTP 消息域发送和接收队列中的消息，我们将把注意力集中在使用 pub/sub 消息域发送和接收消息主题上的消息。

# 与消息主题一起工作

当我们的 Jakarta Messaging 代码使用 pub/sub 消息域时，会使用消息主题。当使用此消息域时，相同的消息可以发送到主题的所有订阅者。

## 向消息主题发送消息

以下示例说明了如何向消息主题发送消息：

```java
package com.ensode.jakartaeebook.pubsubproducer;
//imports omitted
@JMSDestinationDefinition(
    name = "java:global/topic/JakartaEEBookTopic",
    interfaceName = "jakarta.jms.Topic"
)
@Named
@RequestScoped
public class MessageSender {
  @Resource
  private ConnectionFactory connectionFactory;
  @Resource(mappedName =
    "java:global/topic/JakartaEEBookTopic")
  private Topic topic;
  public void produceMessages() {
    JMSContext jmsContext =
      connectionFactory.createContext();
    JMSProducer jmsProducer = jmsContext.createProducer();
    String msg1 = "Testing, 1, 2, 3\. Can you hear me?";
    String msg2 = "Do you copy?";
    String msg3 = "Good bye!";
    jmsProducer.send(topic, msg1);
    jmsProducer.send(topic, msg2);
    jmsProducer.send(topic, msg3);
  }
}
```

如我们所见，前面的代码几乎与我们在讨论 PTP 消息时看到的`MessageSender`类相同。Jakarta Messaging 被设计成可以使用相同的 API 来处理 PTP 和 pub/sub 域。

由于本例中的代码几乎与*使用消息队列*部分中的相应示例相同，我们只解释两个示例之间的差异。在这种情况下，`@JMSDestinationDefinition`的`name`属性值为`jakarta.jms.Topic`，这是在使用 pub/sub 消息域时所需的。此外，我们不是声明一个实现`jakarta.jms.Queue`类的实例，而是声明一个实现`jakarta.jms.Topic`类的实例。然后，我们将这个`jakarta.jms.Topic`的实例作为`JMSProducer`对象的`send()`方法的第一参数传递，同时传递我们希望发送的消息。

## 从消息主题接收消息

正如向消息主题发送消息几乎与向消息队列发送消息相同一样，从消息主题接收消息几乎与从消息队列接收消息相同：

```java
package com.ensode.jakartaeebook.pubsubconsumer;
//imports omitted
@JMSDestinationDefinition(
    name = "java:global/topic/JakartaEEBookTopic",
    interfaceName = "jakarta.jms.Topic"
)
@Named
@RequestScoped
public class MessageReceiver {
  @Resource
  private ConnectionFactory connectionFactory;
  @Resource(mappedName = "java:global/topic/JakartaEEBookTopic")
  private Topic topic;
  private static final Logger LOG =
    Logger.getLogger(MessageReceiver.class.getName());
  public void receiveMessages() {
    String message;
    boolean goodByeReceived = false;
    JMSContext jmsContext = connectionFactory.createContext();
    JMSConsumer jMSConsumer = jmsContext.createConsumer(topic);
    while (!goodByeReceived) {
      message = jMSConsumer.receiveBody(String.class);
      LOG.log(Level.INFO, "Received message: {0}", message);
      if (message.equals("Good bye!")) {
        goodByeReceived = true;
      }
    }
  }
}
```

再次强调，此代码与 PTP 对应代码之间的差异微乎其微。我们不是声明一个实现`jakarta.jms.Queue`类的实例，而是声明一个实现`jakarta.jms.Topic`类的实例。我们使用`@Resource`注解将此类的实例注入到我们的代码中，使用我们在配置应用程序服务器时使用的 JNDI 名称。然后，我们像以前一样获取`JMSContext`和`JMSConsumer`的实例，然后通过在`JMSConsumer`上调用`receiveBody()`方法来接收来自主题的消息。

如本节所示，使用 pub/sub 消息域的优点是消息可以发送到多个消息消费者。这可以通过同时执行本节中开发的`MessageReceiver`类的两个实例，然后执行上一节中开发的`MessageSender`类来轻松测试。我们应该看到每个实例的控制台输出，表明两个实例都接收到了所有消息。

## 创建持久订阅者

使用 pub/sub 消息域的缺点是，消息消费者必须在消息发送到主题时正在执行。如果消息消费者在那时没有执行，它将不会收到消息，而在 PTP 中，消息将保留在队列中，直到消息消费者执行。幸运的是，Jakarta Messaging 提供了一种方法，可以在 pub/sub 消息域中使用并保留消息在主题中，直到所有已订阅的消息消费者执行并接收消息。这可以通过创建对消息主题的持久订阅者来实现。

为了能够服务持久的订阅者，我们需要设置我们的 Jakarta Messaging 连接工厂的`clientId`属性。每个持久的订阅者都必须有一个唯一的客户端 ID，因此必须为每个潜在的持久订阅者声明一个唯一的连接工厂。

我们可以使用`@JMSConnectionFactoryDefinition`注解设置我们的连接工厂的`clientId`属性，如下面的示例所示：

```java
package com.ensode.jakartaeebook.pubsubdurablesubscriber;
//imports omitted for brevity
@JMSConnectionFactoryDefinition(
    name = "java:global/messaging/DurableConnectionFactory",
    clientId = "DurableConnectionFactoryClientId"
)
//Messaging destination definition annotation omitted
@Named
@ApplicationScoped
public class MessageReceiver {
  @Resource(mappedName =
   "java:global/messaging/DurableConnectionFactory")
  private ConnectionFactory connectionFactory;
  @Resource(mappedName =
    "java:global/topic/JakartaEEBookTopic")
  private Topic topic;
  private static final Logger LOG =
   Logger.getLogger(MessageReceiver.class.getName());
  public void receiveMessages() {
    String message;
    boolean goodByeReceived = false;
    JMSContext jmsContext =
      connectionFactory.createContext();
    JMSConsumer jMSConsumer =
      jmsContext.createDurableConsumer(topic,"Subscriber1");
    while (!goodByeReceived) {
      message = jMSConsumer.receiveBody(String.class);
      LOG.log(Level.INFO, "Received message: {0}", message);
      if (message.equals("Good bye!")) {
        goodByeReceived = true;
      }
    }
  }
}
```

如我们所见，前面的代码与之前用于检索消息的示例没有太大区别：与之前的示例相比，只有几个不同之处：我们注入的`ConnectionFactory`实例是通过`@JMSConnectionFactoryDefinition`定义的，并通过其`clientId`属性分配了一个客户端 ID。注意，我们的连接工厂的`@Resource`注解有一个`mappedName`属性，其值与我们定义在`@JMSConnectionFactoryDefinition`中的名称属性相匹配。

另一个区别是，我们不是在`JMSContext`上调用`createConsumer()`方法，而是在调用`createDurableConsumer()`。`createDurableConsumer()`方法接受两个参数，一个用于检索消息的消息`Topic`对象和一个指定此订阅名称的`String`。这个第二个参数必须在所有持久主题的订阅者之间是唯一的。

# 摘要

在本章中，我们详细讨论了如何使用 Jakarta Messaging 发送消息，既使用了 PTP 消息域，也使用了 pub/sub 消息域。

我们讨论的主题包括以下内容：

+   如何通过`jakarta.jms.JMSProducer`接口向消息队列发送消息

+   如何通过`jakarta.jms.JMSConsumer`接口从消息队列接收消息

+   如何通过实现`jakarta.jms.MessageListener`接口异步从消息队列接收消息

+   如何使用前面的接口发送和接收消息到和从消息主题

+   如何通过`jakarta.jms.QueueBrowser`接口浏览消息队列中的消息而不从队列中移除消息

+   如何设置和交互持久订阅到消息主题

带着本章的知识，我们现在可以实现在 Jakarta Messaging 之间进行异步通信。
