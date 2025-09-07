# 第二章：创建出租车应用程序

在日常对话中，人们互相问候，交换闲聊，然后最终结束对话并继续前行。在 RabbitMQ 的轻量级通道中，低级 TCP 连接以同样的方式工作。将要通过 RabbitMQ 交换消息的应用程序需要与消息代理建立永久连接。当建立此连接时，需要创建一个通道，以便执行面向消息的交互，例如发布和消费消息。

在演示了这些基础知识之后，本章将介绍经纪人如何使用**交易所**来确定每条消息应该发送到何处。交易所就像邮递员一样：它将消息递送到适当的队列（邮箱）供消费者在稍后找到。

基本 RabbitMQ 概念如下所示：

![](img/18e8d28a-a508-4004-8e74-5c3d14f7e03b.png)

图 2.1：基本 RabbitMQ 概念

到本章结束时，你将深入理解**完整汽车**（**CC**）平台背后的应用程序架构以及它们如何通过 RabbitMQ 发送第一条消息。这需要介绍两种不同类型的交易所：直接交换，它将消息发送到单个队列，以及主题交换，它根据模式匹配路由键将消息发送到多个队列。

为了获得最佳开始，以下主题将得到介绍：

+   CC 背后的应用程序架构

+   建立与 RabbitMQ 的连接

+   发送第一条消息

+   添加主题消息

让我们开始吧！

# 技术要求

本章的代码文件可以在 GitHub 上找到：[`github.com/PacktPublishing/RabbitMQ-Essentials-Second-Edition/tree/master/Chapter02`](https://github.com/PacktPublishing/RabbitMQ-Essentials-Second-Edition/tree/master/Chapter02)。

# CC 背后的应用程序架构

CC 需要一款供出租车司机使用的应用程序和一款供客户使用的应用程序。客户应能够通过应用程序请求出租车，出租车司机应能够接受请求（行程）： 

![](img/9ef350c8-bc43-4d03-b857-8019b22db9a2.png)

图 2.2：客户通过 CC 应用程序请求出租车

客户应能够输入关于行程起点和终点的信息。活跃的司机收到请求并能够接受它们。最终，客户应能够在整个行程中跟踪出租车的位置。

下图显示了 CC 想要实现的消息架构：

![](img/74212d1d-1a7a-4c38-9195-fa1a97e55bd1.png)

图 2.3：CC 的主要应用程序架构

如前图所示，此流程可以用 10 个步骤来解释：

1.  客户使用 CC 的移动应用程序预订出租车。现在，请求从移动应用程序发送到应用程序服务。此请求包含客户想要预订的行程信息。

1.  应用程序服务将请求存储在数据库中。

1.  应用程序服务将有关行程的信息消息添加到 RabbitMQ 的一个队列中。

1.  已连接的出租车订阅消息（预订请求）。

1.  一辆出租车通过向 RabbitMQ 发送消息来响应客户。

1.  应用程序服务订阅消息。

1.  再次，应用程序服务将信息存储在数据库中。

1.  应用程序服务将信息转发给客户。

1.  出租车应用程序开始自动以给定间隔将出租车的地理位置发送到 RabbitMQ。

1.  然后，将出租车的位置直接通过 WebSockets 传递给客户的移动应用程序，以便他们知道出租车何时到达。

让我们首先仔细看看前面图中所示的步骤*1*、*2*、*3*和*4*，其中客户请求出租车（消息发布到 RabbitMQ），出租车司机接收请求（从 RabbitMQ 消费消息）。

# 建立到 RabbitMQ 的稳固连接

如第一章“兔子跃然纸上”中所述，应用程序服务器和 RabbitMQ 之间必须建立物理网络连接。**高级消息队列协议**（**AMQP**）连接是客户端和代理之间的链接，执行底层网络任务，包括初始身份验证、IP 解析和网络：

![图片](img/e959981b-8e51-459f-adb4-f27147bf43aa.png)

图 2.4：应用程序和 RabbitMQ 之间的 AMQP 连接

每个 AMQP 连接维护一组底层通道。通道重用连接，无需重新授权和打开新的 TCP 流，使其更有效率。

以下图示说明了应用程序与 RabbitMQ 之间连接中的一个通道：

![图片](img/43ce057b-af5f-4164-95ff-ee965b9a21b7.png)

图 2.5：通道允许您更有效地使用资源

与创建通道不同，创建连接是一个成本较高的操作，非常类似于数据库连接。通常，数据库连接是池化的，其中每个池实例由单个执行线程使用。AMQP 的不同之处在于，单个连接可以通过多个复用通道被许多线程使用。

AMQP 连接的握手过程至少需要七个 TCP 数据包，使用 TLS 时甚至更多。如果需要，通道可以更频繁地打开和关闭：

+   AMQP 连接：7 个 TCP 数据包

+   AMQP 通道：2 个 TCP 数据包

+   AMQP 发布：1 个 TCP 数据包（对于更大的消息可能有更多）

+   AMQP 关闭通道：2 个 TCP 数据包

+   AMQP 关闭连接：2 个 TCP 数据包

+   总计 14-19 个数据包（包括确认包）

以下图示说明了发送到连接和通道的信息概述：

![图片](img/f23a471f-89ff-43ca-a643-e779f4492462.png)

图 2.6：AMQP 连接的握手过程

在应用程序服务和 RabbitMQ 之间建立一个单一的长连接是一个良好的开始。

必须决定使用哪种编程语言和客户端库。本书的前几个示例是用 Ruby 编写的，并使用 Bunny 客户端库（[`github.com/ruby-amqp/bunny`](https://github.com/ruby-amqp/bunny)）来发布和消费消息。Ruby 是一种易于阅读和理解的语言，即使你对它不熟悉。

应用程序必须配置为使用特定的连接端点，通常称为连接字符串；例如，主机和端口。连接字符串包含建立连接所需的信息。AMQP 分配的端口号是`5672`。可以通过 AMQPS 使用 TLS/SSL 加密的 AMQP；它是分配端口号`5671`的 AMQP 协议的安全版本。

该库是打开目标 IP 地址和端口的 TCP 连接的元素。连接参数已被添加为一个名为`RABBITMQ_URI`的环境变量的 URI 字符串。AMQP URI 没有 URI 标准，但此格式被广泛使用：

```java
 RABBITMQ_URI="amqp://user:password@host/vhost"
```

根据 Ruby（Bunny）文档，连接到 RabbitMQ 很简单。这段代码被分为代码块，可以在本章后面找到：

1.  添加在第一章“兔子跃动起来”中设置的用户名、密码和`vhost`，然后将字符串添加到机器上的环境变量中：

```java
RABBITMQ_URI="amqp://cc-dev:taxi123@localhost/cc-dev-vhost"
```

1.  需要引入`bunny`客户端库：

```java
# Require client library
require "bunny"
```

1.  从环境变量中读取连接 URI 并启动连接：

```java
connection = Bunny.new ENV['RABBITMQ_URI']
# Start a session with RabbitMQ 
connection.start
```

到目前为止，这似乎很简单，但 CC 需要能够优雅地处理失败的生成级代码。如果 RabbitMQ 没有运行会怎样？显然，如果整个应用程序都崩溃了，那就很糟糕。如果需要重启 RabbitMQ 呢？CC 希望其应用程序在出现任何问题时都能优雅地恢复。实际上，CC 希望其应用程序无论整个消息子系统是否工作都能继续运行。用户体验必须平滑、易于理解，并且可靠。

总结来说，CC 希望实现的行为如下：

+   如果与 RabbitMQ 的连接丢失，它应该自动重新连接。

+   如果连接断开，发送或获取消息应该优雅地失败。

当应用程序连接到代理时，它需要处理连接失败。没有网络总是可靠的，配置错误和错误也会发生；代理可能已经关闭，等等。虽然不是自动的，但在这个情况下，错误检测应该在过程早期发生。

要处理 Bunny 中的 TCP 连接失败，必须捕获异常：

```java
begin
  connection = Bunny.new ENV['RABBITMQ_URI']
  connection.start
rescue Bunny::TCPConnectionFailed => e
  puts "Connection to server failed"
end
```

如果应用程序无法从网络连接失败中恢复，检测网络连接失败几乎是无用的。恢复是错误处理的重要部分。

一些客户端库提供自动连接恢复功能，包括消费者恢复。对已关闭通道的任何尝试操作都将失败并抛出异常。如果 Bunny 检测到 TCP 连接故障，它将尝试每 5 秒重新连接一次，没有关于重连尝试次数的限制。可以通过在`Bunny.new`中添加`automatic_recovery => false`来禁用自动连接恢复。此设置仅在您以其他方式重新连接或测试连接字符串时使用。

消息可以在不同的语言、平台和操作系统之间发送。您可以从多种不同语言的客户端库中进行选择。客户端库有很多，但以下是一些推荐的：

+   Python：Pika

+   Node.js：amqplib

+   PHP：php-amqplib

+   Java：amqp-client

+   Clojure：Langohr

本节展示了 CC 如何建立与 RabbitMQ 的连接。我们演示了为什么推荐长期连接以及如何处理一些常见错误。现在，是时候在连接中创建一个通道了。

## 与通道一起工作

每个 AMQP 协议相关的操作都在通道上发生。通道实例是由连接实例创建的。如上所述，通道是在(TCP)连接内的虚拟(AMQP)连接。客户端执行的所有操作都在通道上，队列在通道上声明，消息通过通道发送。

通道本身并不存在；它总是在连接的上下文中：

```java
# Declare a channel
channel = connection.create_channel
```

一旦连接关闭或发生通道错误，连接中的通道就会被关闭。客户端库允许我们观察并响应通道异常。

通常在通道级别而不是在连接级别抛出更多**异常**。通道级别的异常通常指示应用程序可以从中恢复的错误，例如，当它没有权限，或者尝试从一个已删除的队列中消费时。对已关闭通道的任何尝试操作都将通过异常失败。

尽管通道实例在技术上是无线程安全的，但强烈建议避免多个线程同时使用同一个通道。

CC 现在能够以线程安全和异常安全的方式连接到 RabbitMQ 代理、打开通道并发出一系列命令。现在是时候在此基础上构建了！

## 构建出租车请求工具

现在，是时候构建消息流了。

首先，客户将从移动应用程序向应用程序服务发送一个简单的 HTTP 请求。此消息将包含元信息，如时间戳、发送者和接收者 ID 以及目的地和请求的出租车 ID。

消息流看起来可能如下所示：

![图片](img/c4c3407a-a481-4a0c-9d1c-6f0b2adb7b4c.png)

图 2.7：CC 主应用程序的前端/后端交互

应用程序服务将信息存储在数据库中，以便所有数据在后续状态中都对数据分析脚本可见。

在这些示例中，没有处理数据在数据库中的存储方式，因为这不是本章所关注的主题。最简单的方法是允许应用程序服务将信息添加到数据库中。另一种选择是将应用程序服务卸载，并将新消息放入数据库和应用程序服务之间的消息队列中，让另一个服务订阅这些消息并处理它们；也就是说，将它们存储在数据库中。

以下图示说明了移动设备、应用程序服务和 RabbitMQ 之间的流程：

![图片](img/09a3d121-718e-4fbc-86de-3f58c18094fa.png)

图 2.8：移动设备、应用程序服务和 RabbitMQ 之间的流程

关于我们主要流程的讨论，第一章，“兔子跃然于生”，详细说明了消息在被路由到队列以供消费后是如何发布到交换的。

路由策略决定了消息将被路由到哪个队列（或多个队列）。路由策略基于路由键（一个自由形式的字符串）以及可能的消息元信息做出决策。将路由键想象成交换用来决定消息如何路由的地址。它还需要在交换和队列之间建立一个绑定，以使消息可以从前者流向后者。

现在，让我们来探讨直接交换。

### 直接交换

直接交换根据消息路由键将消息传递到队列。一条消息将发送到与消息路由键匹配的绑定路由键的队列（或队列）。

CC 只有两辆车，因此它从一个简单的通信系统开始，其中一个客户可以请求一位司机的出租车。在这种情况下，需要将一条消息路由到该司机的收件箱队列。因此，将要使用的交换-路由策略是直接的，将目标队列名称与消息产生时使用的路由键相匹配，如下面的图示所示：

![图片](img/40c44f43-019d-4d0d-8880-d292a31a6a19.jpg)

图 2.9：直接交换将消息路由到特定的队列

直接交换的一个示例用例可能如下所示：

1.  客户订购名为 taxi.1 的出租车。一个 HTTP 请求从客户的移动应用程序发送到应用程序服务。

1.  应用程序服务使用路由键 taxi.1 向 RabbitMQ 发送消息。消息路由键与队列名称匹配，因此消息最终进入 taxi.1 队列。

以下图示演示了直接交换消息路由将如何发生：

![图片](img/71ddd244-042c-4d10-a2a0-af0476d1a2cd.png)

图 2.10：根据路由键将直接交换路由消息到特定的队列

这可能不是扩展效率最高的方法。实际上，一旦 CC 有更多的车辆，就会对其进行审查，但这是快速开始和启动应用程序的最简单方法。

让我们跟随 CC 创建的第一个代码块作为初始应用程序，并在学习不同概念的同时进行。代码块开头的代码是从连接和通道部分取出的：

1.  需要使用`bunny`客户端库。

1.  从环境变量中读取`URI`连接并启动连接。

1.  与 RabbitMQ 启动通信会话。

1.  声明`taxi.1`队列。

1.  声明`taxi.1`直接交换。

1.  将`taxi.1`队列绑定到`taxi-direct`交换，使用`taxi.1`路由键：

```java
# 1\. Require client library
require "bunny"

# 2\. Read RABBITMQ_URI from ENV
connection = Bunny.new ENV["'RABBITMQ_URI"]

# 3\. Start a communication session with RabbitMQ
connection.start
channel = connection.create_channel

def on_start(channel)
 # 4\. Declare a queue for a given taxi
 queue = channel.queue("taxi.1", durable: true)
 # 5\. Declare a direct exchange, taxi-direct
 exchange = channel.direct("taxi-direct", durable: true, auto_delete: true)

 # 6\. Bind the queue to the exchange
 queue.bind(exchange, routing_key: "taxi.1")

 # 7\. Return the exchange
 exchange
end

exchange = on_start(channel)                  
```

对于每个发送的消息都声明队列和交换有些过度，并且是不必要的，因此强烈建议创建一个处理应用程序设置的方法。这个方法应该创建连接并声明队列、交换等。本例中的方法简单地称为`on_start`，它声明队列并将交换绑定到队列。

如果在交换上发布消息时交换不存在，它将引发异常。如果交换已经存在，它将不执行任何操作；否则，它实际上会创建一个。这就是为什么每次应用程序启动或发布消息之前声明队列都是安全的。

通道由异常终止。在 CC 的情况下，向不存在的交换发送消息不仅会引发异常，还会终止发生错误的通道。任何尝试使用已终止通道的后续代码都将失败。

除了使用直接类型外，CC 还配置了交换的`durable`类型、`autoDelete`和`argument`属性。这个交换在 RabbitMQ 重启后不应消失，也不应在未使用时消失，这解释了配置中使用的值。

交换声明仅在交换属性相同时才是幂等的。尝试使用不同属性声明已存在的交换将会失败。在交换声明中始终使用一致的属性。如果你正在更改属性，请在声明新属性之前删除交换。同样的规则也适用于队列声明。

在创建交换之后，创建 taxi 队列并将其绑定到它。

队列的声明方法与交换类似，但属性略有不同，如下所示：

+   `durable`: True – 队列必须在代理重启后仍然声明。

+   `autoDelete`: False – 保持队列，即使它不再被消费。

+   `exclusive`: False – 此队列应该能够被其他连接消费（多个应用服务器可以连接到 RabbitMQ，并通过不同的连接访问）。

+   `arguments`: Null – 无需自定义配置队列。

队列使用其自己的名称作为路由键绑定到交换，以便直接路由策略可以将消息路由到它。当这样做时，向`taxi-direct`交换发布消息实际上会将消息传递到名称与发布路由键匹配的出租车队列。

如果没有队列绑定到交换，或者如果路由策略找不到匹配的目标队列，则发布到交换的消息将被静默丢弃。作为选项，当无法路由的消息被丢弃时，可以通知到，如后续章节所示。

再次强调，当使用相同的属性时，这些操作是幂等的，因此队列可以安全地声明并绑定到交换，一次又一次。

尽管直接交换在本章中已经介绍过，但 AMQP 0-9-1 代理提供了四种不同类型的交换。根据队列和参数之间的绑定设置，这些交换以不同的方式路由消息。接下来的章节将更详细地介绍其他类型的交换。现在，这里简要说明每种交换：

+   **扇出**：消息将被路由到绑定到扇出交换的所有队列。

+   **主题**：通配符必须与绑定特定路由模式的路由键匹配。

+   **头部**：使用消息头部属性进行路由。

现在，是时候向 RabbitMQ 发送我们的第一条消息了！

# 发送第一条消息

基本概念和初始设置已经介绍过了，所以让我们直接发送第一条消息！

首先，让我们看看`order_taxi`方法，它负责发送初始车辆请求的消息：

```java
def order_taxi(taxi, exchange)
  payload = "example-message"
  message_id = rand
 exchange.publish(payload,
    routing_key: taxi,
    content_type: "application/json",
    content_encoding: "UTF-8",
    persistent: true,
    message_id: message_id)
end

exchange = on_start(channel)
order_taxi("taxi.1", exchange)
```

`order_taxi`将在用户想要叫出租车时被调用。无法保证收件人是否曾登录过系统，因此对于发送者来说，无法确定目标队列是否存在。最安全的做法是在每条消息发送时声明队列，记住这个声明操作是幂等的，所以如果队列已经存在，它将不会做任何事情。这乍一看可能有些奇怪，但如果发送者想要确保消息不会丢失，那么确保收件人的队列存在是发送者的责任。

当事件之间没有强烈的**发生之前**关系时，这是 AMQP 中的一种常见模式。**重新声明**是可行的方案。相反，**检查**-**然后**-**行动**模式是不被推荐的；试图检查交换或队列的存在并不能保证在 AMQP 通常使用的典型分布式环境中成功。

发布消息的方法非常简单；向`exchange`调用`publish`。然后，使用队列名称作为路由键（按照**直接**路由），以及表示实际消息有效载荷的字节数组。还可以添加一些可选的消息属性，可能包括以下内容：

+   `content_type`（**字符串**）：消息以字节数组的形式发布和消费，但并没有真正说明这些字节代表什么。在当前情况下，发布者和消费者都在同一个系统中，因此可以假设内容类型是预期的。话虽如此，始终指定内容类型，以便消息是自包含的；无论哪个系统最终接收或检查消息，都将确切知道它包含的字节数组代表什么。

+   `content_encoding`（**字符串**）：在将字符串消息序列化为字节数组以便发布时使用特定的编码（UTF-8）。再次强调，为了使消息自解释，提供所有必要的元信息，以便它们可以被读取。

+   `message_id`（**字符串**）：正如本书后面所展示的，消息标识符是消息和分布式应用程序中可追溯性的重要方面。在示例中是一个随机生成的消息 ID。

+   `persistent`（**布尔值**）：指定消息是否应该持久化到磁盘。

不要混淆交换和队列的持久性与消息的持久性；存储在持久队列中的非持久消息在代理重启后将消失，留下一个空队列。

此外，在非持久队列中的持久消息在代理重启后也将消失，同样会留下一个空队列。

通过将队列声明为**持久**的并将消息投递模式设置为**持久**，确保消息不会丢失。

但如果发送消息失败会怎样，比如当与 RabbitMQ 的连接断开时？

为什么你会使用非持久投递模式呢？难道消息代理如 RabbitMQ 的整个目的不是保证消息不会丢失吗？这是真的，但在某些情况下，这种保证可以放宽。考虑一个场景，其中发布者向代理发送大量非关键消息。在这种情况下使用非持久投递意味着 RabbitMQ 不需要不断访问磁盘，从而在此情况下提供更好的性能。

在继续之前，让我们看看 AMQP 消息的结构。

## AMQP 消息结构

以下截图展示了 AMQP 消息的结构，包括刚刚使用的四个 AMQP 消息属性，以及一些新的属性。请注意，此图使用的是字段的规范名称，并且每种语言的实现都会稍微重命名它们，以便它们可以成为有效的名称。例如，`content-type`在 Java 中变为`contentType`，而在 Ruby 中变为`content_type`：

![图片](img/9b3fe337-567a-459c-97b3-1226398fa7dd.png)

图 2.11：AMQP 消息的属性

除了`reserved`之外，所有这些属性都可以自由使用，除非另有说明，否则 AMQP 代理会忽略它们。在 RabbitMQ 的情况下，代理支持的字段只有`user-id`字段，该字段被验证以确保它与建立连接的代理用户名称匹配。注意`headers`属性如何允许我们在没有标准属性符合要求的情况下添加额外的键值对。

下一节将解释消息是如何被消费的。

## 消费消息

现在，让我们将注意力转向负责检索消息的方法，这是 CC 主架构中的**第 4 步**，可以在“CC 背后的应用架构”部分找到。

在这里，出租车应用程序可以定期检查队列以获取新消息。这是一种所谓的同步方法。这意味着将处理轮询请求的应用程序线程保持，直到所有挂起的消息都从队列中移除，如图所示：

![图 2.13](img/2c2137d3-3784-4d43-90d1-4ce304c5d344.png)

图 2.12：客户端在代理中请求新消息

一个前端定期轮询后端以获取消息，很快就会在负载方面造成影响，这意味着整体解决方案将开始遭受性能下降。

相反，CC 明智地决定采用服务器推送的方法来构建解决方案。想法是从代理服务器将消息推送到客户端。幸运的是，RabbitMQ 提供了两种接收消息的方式：基于轮询的`basic.get`方法和基于推送的`basic.consume`方法。如图所示，消息被推送到消费者：

![图 2.13](img/152412b4-01d6-4e4a-ac95-e9fc397b1f52.png)

图 2.13：消费者从代理订阅消息

`subscribe`方法将消费者添加到队列中，然后订阅接收消息传递。

确保消费者从队列中消费消息，而不是使用基本的 GET 操作。当涉及到资源时，`basic.get`命令相对昂贵。

使用`subscribe`，当有新消息准备好且客户端有可用性时，消息将从代理发送到客户端。这通常允许消息的平稳处理。此外，使用`subscribe`意味着消费者只要声明的通道可用或直到客户端取消它，就会保持连接。

消息处理正在平稳且无障碍地进行，几乎就像什么都没发生一样！当然，直到设置了警报来确认或否定确认某个过程是否按预期或未按计划运行。

### 确认和否定确认

RabbitMQ 需要知道何时可以将消息视为**成功**，即在预期中将消息发送给消费者。一旦代理收到响应，代理应从队列中删除消息；否则，队列会溢出。客户端可以通过在收到消息时或消费者完全处理消息时确认消息来回复代理。在任何情况下，一旦消息被确认，它就会从队列中移除。

因此，消费者是否确认消息取决于它是否已经完成处理，或者它确定如果异步处理则没有丢失消息的风险。

为了避免消息可能永远丢失的情况（例如，工作者崩溃、异常等），消费应用程序应在完全完成消息处理之前不确认消息。

当应用程序向代理指示处理失败或无法在此时完成时，应用程序会拒绝消息。否定确认（Nack）告诉 RabbitMQ 消息没有被按指示处理。默认情况下，否定确认的消息会被送回队列以进行另一次尝试。

确认将在第三章中详细说明，*向多个出租车司机发送消息*。

准备好了吗？设定好了吗？是时候 RUN，Rabbit！

### 运行代码

现在，是时候为消费者设置一些代码了。你将能够从上一节*发送第一条消息*中识别出大部分代码。

1.  需要客户端库。

1.  从`ENV`中读取`RABBITMQ_URI`。

1.  与 RabbitMQ 启动通信会话。

1.  为特定出租车声明一个队列。

1.  声明一个直接交换，`taxi-direct`。

1.  将队列绑定到交换。

1.  订阅队列。

接下来是初始消费者设置所需的代码：

```java
# example_consumer.rb
# 1\. Require client library
require "bunny"

# 2\. Read RABBITMQ_URI from ENV
connection = Bunny.new ENV["RABBITMQ_URI"]

# 3\. Start a communication session with RabbitMQ
connection.start
channel = connection.create_channel

# Method for the processing
def process_order(info)

  puts "Handling taxi order"
  puts info
  sleep 5.0
  puts "Processing done"
end

def taxi_subscribe(channel, taxi)
  # 4\. Declare a queue for a given taxi
  queue = channel.queue(taxi, durable: true)

  # 5\. Declare a direct exchange, taxi-direct
  exchange = channel.direct("taxi-direct", durable: true, auto_delete: true)

  # 6\. Bind the queue to the exchange
  queue.bind(exchange, routing_key: taxi)

  # 7\. Subscribe from the queue
  queue.subscribe(block: true, manual_ack: false) do |delivery_info, properties, payload|
    process_order(payload)
  end
end

taxi = "taxi.1"
taxi_subscribe(channel, taxi)                            
```

在这里，`subscribe`方法中添加了两个需要解释的标志。让我们详细看看它们：

+   `block`（布尔值，默认`false`）：调用是否应该阻塞调用线程？此选项对于保持脚本的主线程活动可能很有用。它与自动连接恢复不兼容，并且通常不推荐使用。

+   `manual_ack`（布尔值，默认`false`）：在 CC 的情况下，由于在此阶段丢失消息的风险是可以接受的，系统不会手动确认消息。相反，它会在获取消息后立即通知代理将其视为已确认（关于手动确认的更多内容将在本书的后面部分介绍）。

就这样！CC 现在有一个可测试的工作订单收件箱。接下来，我们将查看当激活出租车运行时的管理控制台。

## 运行应用程序

当应用程序运行并且服务器连接到 RabbitMQ 时，可以从管理控制台中看到以下已建立的连接：

![图片](img/90a38a53-a625-4f9f-a15f-b3a8c4ff7c1a.png)

图 2.14：管理控制台提供连接信息

注意，上游和下游的网络吞吐量被清楚地表示，而那些打开和关闭非常快的通道在管理控制台中很难看到。因此，让我们看看以下交换：

![](img/91175a73-136f-44c6-b528-217c23549537.png)

图 2.15：在管理控制台中出现的 taxi-direct 直接交换

用户交换和进出消息的速率显示在管理控制台中。它们被以与接收速度相同的速度消耗，这是一个好兆头，表明当前架构足以满足 CC 的需求，并且消息没有堆积。但是，所有这些未由代码创建的交换是什么？它们从哪里来？以（AMQP 默认）表示的无名交换以及所有以 amq.开头的交换名称都是由 AMQP 规范定义的，因此 RabbitMQ 必须默认提供。现在，关于队列呢？让我们看看：

![](img/9e8a32d8-41e2-46fc-a8fb-79fd4cb2acb1.png)

图 2.16：每个客户到出租车收件箱队列在管理控制台中可见

如预期的那样，每辆出租车都有一个队列，还有一些巧妙的用法统计信息。注意，ack 列是空的，这是意料之中的事，因为消息确认的工作方式就是这样。队列正在接收消息，同时通知 RabbitMQ 它不会进行确认，因此没有与确认消息相关的活动。

如果有足够的 RAM，RabbitMQ 可以处理数百个队列和绑定而不会出现问题，所以多个队列不是问题。

对其架构和实施充满信心，CC 推出了客户到出租车订购子系统。客户可以发送请求，出租车可以处理请求。

CC 通过两款新的环保汽车迅速扩大了公司规模。与之前的解决方案一样，客户需要向特定司机发送订单请求消息。现在，客户请求了一个新功能——向一组出租车发送消息的能力。客户应该能够选择普通出租车或环保出租车。让我们看看 CC 将通过 RabbitMQ 的力量如何实现这一新功能。

# 添加主题消息

CC 的应用程序允许其出租车通过注册它们感兴趣的主题来组织成组。即将推出的新功能将允许客户向特定主题内的所有出租车发送订单请求。结果证明，这个功能与一个特定的交换路由规则相匹配，不出所料，被称为 topic！这种类型的交换允许我们将消息路由到所有与路由键匹配消息路由键的队列。因此，与最多将消息路由到一个队列的直接交换不同，主题交换可以将消息路由到多个队列。基于主题的路由还可以应用于其他两个场景，例如特定位置的数据，如交通警告广播，或行程价格更新。

路由模式由几个用点分隔的单词组成。遵循的最佳实践是从最一般元素到最具体元素来构建路由键，例如`news.economy.usa`或`europe.sweden.stockholm`。

主题交换支持严格的路由键匹配，并且还会使用`*`和`#`作为占位符来执行通配符匹配，分别代表恰好一个单词和零个或多个单词。

以下图表展示了在 CC 应用中如何使用主题交换。注意，单个收件箱队列保持不变，只是通过额外的绑定连接到主题交换，每个绑定都反映了用户的一个兴趣点：

![图片](img/3f964e6f-517b-466a-a0eb-9965c3fb05a2.png)

图 2.17：主题交换向生态队列发送主题消息

由于使用相同的收件箱处理所有事务，因此用于检索消息的现有代码无需更改。实际上，这个整个功能只需添加少量代码即可实现。这些添加中的第一个是负责在现有的`on_start`方法中声明主题交换，如下所示：

```java
def on_start(channel)
  # Declare and return the topic exchange, taxi-topic
  channel.topic("taxi-topic", durable: true, auto_delete: true)
end
```

这里并没有什么真正新颖或复杂的地方；主要区别在于这个交换被命名为`taxi-topic`，并且是一个`topic`类型的交换。发送消息甚至比客户端到出租车功能还要简单，因为没有尝试创建收件人的队列。发送者遍历所有用户以创建和绑定他们的队列是没有意义的，因为只有那些在发送消息时已经订阅了目标主题的用户会收到消息，这正是预期的功能。`order_taxi`方法如下所示：

```java
# Publishing an order to the exchange
def order_taxi(type, exchange)
  payload = "example-message"
  message_id = rand
  exchange.publish(payload,
                   routing_key: type,
                   content_type: "application/json",
                   content_encoding: "UTF-8",
                   persistent: true,
                   message_id: message_id)
end

exchange = on_start(channel)
# Order will go to any eco taxi
order_taxi('taxi.eco', exchange) 
# Order will go to any eco taxi
order_taxi('taxi.eco', exchange) 
# Order will go to any taxi
order_taxi('taxi', exchange) 
# Order will go to any taxi
order_taxi('taxi', exchange) 
```

不同之处在于，现在消息是发布到`taxi-topic`交换的。创建和发布消息的其余代码与客户端到出租车消息的代码完全相同。最后，当新的出租车订阅或取消订阅某些主题时，需要添加信息：

```java
# example_consumer.rb

def taxi_topic_subscribe(channel, taxi, type)
  # Declare a queue for a given taxi
  queue = channel.queue(taxi, durable: true)

  # Declare a topic exchange
  exchange = channel.topic('taxi-topic', durable: true, auto_delete: true)

  # Bind the queue to the exchange
  queue.bind(exchange, routing_key: type)

  # Bind the queue to the exchange to make sure the taxi will get any order
  queue.bind(exchange, routing_key: 'taxi')

  # Subscribe from the queue
  queue.subscribe(block:true,manual_ack: false) do |delivery_info, properties, payload|
    process_order(payload)
  end
end

taxi = "taxi.3"
taxi_topic_subscribe(channel, taxi, "taxi.eco.3")
```

`taxi.3`是新的环保出租车，现在准备好接收想要环保车型的客户的订单。

AMQP 规范没有提供任何方法来检查队列的当前绑定，因此无法迭代它们并删除不再需要的绑定，以反映出租车感兴趣的主题的变化。这不是一个严重的问题，因为应用程序无论如何都需要维护这个状态。

RabbitMQ 管理控制台提供了一个 REST API，可以用来执行队列绑定自省，以及其他许多 AMQP 规范未涵盖的功能。更多内容将在后续章节中介绍。

在放置了新的代码之后，一切按预期工作。不需要对代码进行更改来检索新的客户端到出租车订单，因为它们与之前的消息一样，到达同一个收件箱队列。主题消息被出租车正确发送和接收，所有这些都是在最小更改和队列数量没有增加的情况下发生的。当连接到管理控制台时，点击“交换”选项卡；唯一可见的差异是新交换的主题；即`taxi-topic`。

# 摘要

本章介绍了如何连接到 RabbitMQ 以及如何发送和接收订单消息。汽车订单系统已成功创建，并在 CC 的客户端到出租车和客户端到多辆出租车功能背景下启动了直接和主题交换。

随着完整汽车（Complete Car）的成长，对出租车应用中新增功能的需求也在增加。当 CC（Complete Car）满足用户需求时，接下来会是什么？下一章将解释如何使用通道和队列来扩展应用的功能。
