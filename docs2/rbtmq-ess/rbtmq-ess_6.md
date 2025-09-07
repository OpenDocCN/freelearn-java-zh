# 第六章：将 RabbitMQ 投入生产

到目前为止，**完整汽车**（**CC**）在生产中运行单个 RabbitMQ 实例。现在 CC 还需要确保服务具有高可用性。创建节点集群确保即使系统出现故障，信息仍然可访问。本章将介绍如何设置 RabbitMQ 集群，包括代理集群、经典镜像队列和法定多数队列的介绍。CC 还在寻找一个新的优雅的日志聚合解决方案，其中所有日志都通过联邦插件发布到集中的 RabbitMQ 节点，因此本章也将涵盖这一主题。

为了实现 CC 几乎不间断运行的目标，本章将包括以下内容：

+   向集群添加节点

+   发现 RabbitMQ 队列的类型

+   使用联邦代理和日志聚合

# 技术要求

本章的代码文件可以在 GitHub 上找到，地址为[`github.com/PacktPublishing/RabbitMQ-Essentials-Second-Edition/tree/master/Chapter06`](https://github.com/PacktPublishing/RabbitMQ-Essentials-Second-Edition/tree/master/Chapter06)。

# 向集群添加节点

对于 CC 来说，一切运行顺利，但开发者们希望确保系统能够承受崩溃。即使在使用 RabbitMQ 的情况下，崩溃总是可能的。停电可能发生，突然的数据包丢失可能会损坏更新，管理员可能由于意外错误而错误地配置系统。由于故障或错误，整个实例可能会丢失。必须采取措施解决可能导致数据丢失、负面客户体验，甚至可怕的凌晨 2 点给团队打电话的问题。

好消息是，RabbitMQ 提供了处理潜在崩溃和其他灾难所需的特性，无需额外配置。RabbitMQ 可以配置为在活动-活动部署环境中运行，这意味着两个或多个节点可以同时运行相同类型的服务。几个代理可以参与集群，作为单个高可用的**高级消息队列协议**（**AMQP**）服务。

在使用活动-活动部署时，无需手动故障转移。如果代理发生故障，无需任何操作，从而避免了凌晨 2 点的电话。根据高可用性集群中活动节点的数量，集群可以承受多次故障。

为了避免因无法访问代理而引起的复杂情况，CC 决定首先启动第二个 RabbitMQ 实例（命名为 rmq-prod-2），并将其与生产中已使用的实例进行集群化。

RabbitMQ 集群是由一个或多个节点组成的逻辑分组，每个节点共享用户、虚拟主机、队列、交换机等。系统架构仅在集群内部发生变化，如下所示：

![](img/4f35e1a1-09c1-4582-8569-3d7f1e9425c4.png)

图 6.1：由多个 RabbitMQ 代理组成的高可用性集群

更多的节点被添加到 RabbitMQ 集群中。当第二个 RabbitMQ 实例准备与现有的一个进行集群时，CC 会通知团队。为了实现这一点，将使用 RabbitMQ 的 Erlang 集群功能，以允许多个 Erlang 节点之间的本地或远程通信。Erlang 集群使用安全 cookie 作为跨节点认证的机制。为了避免错误，开发人员已确保 `/var/lib/rabbitmq/.erlang.cookie` 文件在每个实例中的内容相同。

注意，如果防火墙阻止 RabbitMQ 实例相互通信，则集群将无法工作。如果发生这种情况，请打开 AMQP（默认为 `5672`）所使用的特定端口，以便集群可以工作。更多信息请参阅[`www.rabbitmq.com/clustering.html#firewall`](http://www.rabbitmq.com/clustering.html#firewall)。

在第二节点上无需配置任何用户或虚拟主机，如第一章“Rabbit 春天苏醒”中所述。只需加入集群，配置将自动与现有的 RabbitMQ 实例同步，包括用户、虚拟主机、交换机、队列和策略。

请记住，节点在加入集群时会完全重置。RabbitMQ 在与其他节点同步之前会删除所有配置和数据。

要将节点加入集群，首先停止 RabbitMQ，然后加入集群，最后重新启动 RabbitMQ 应用程序：

```java
$ sudo rabbitmqctl stop_app
# => Stopping node rabbit@rmq-prod-2 ...
# => ...done.
$ sudo rabbitmqctl join_cluster rabbit@rmq-prod-1
# => Clustering node rabbit@rmq-prod-2 with rabbit@rmq-prod-1 ...
# => ...done.
$ sudo rabbitmqctl start_app
# => Starting node rabbit@rmq-prod-2 ...
# => ...done.
```

确保所有 RabbitMQ 节点使用相同的 Erlang 主版本，否则 `join_cluster` 命令可能会失败。虽然可以运行包含不同 Erlang 版本的集群，但可能会出现不兼容性，从而影响集群的稳定性。

RabbitMQ 还要求节点之间使用相同的 3.7.x 及以下主/次版本。大多数情况下，可以运行不同的补丁版本（例如，3.7.X 和 3.7.Y），除非发布说明中另有说明。

功能标志是 RabbitMQ 3.8 版本引入的新机制。这些标志定义了 RabbitMQ 节点成为集群一部分的能力。功能标志控制哪些功能在所有集群节点上被视为启用或可用，因此使用子系统的节点必须具有相同的依赖项。更多信息请参阅[`www.rabbitmq.com/feature-flags.html`](https://www.rabbitmq.com/feature-flags.html)。

在运行前面的命令后，通过在任何节点上运行 `cluster_status` 命令来检查集群是否活跃：

```java
$ sudo rabbitmqctl cluster_status 
# => Cluster status of node rabbit@rmq-prod-1 # -> 
# => [{nodes,[{disc,[rabbit@rmq-prod-2,rabbit@rmq-prod-1]}]}, {running_nodes,[rabbit@rmq-prod-2,rabbit@rmq-prod-1]}, {partitions,[]}]
# => ...done.
```

注意状态消息中给出了两个节点列表。在这种情况下，节点是集群中配置的节点列表。名为 `running_nodes` 的列表包含那些实际活跃的节点。配置的节点是持久的，这意味着它们将能够在代理重启后存活，因为每个代理都会自动重新加入集群。

通过连接到另一个节点（rmq-prod-2）上的管理控制台来确认新节点将同步到集群。使用 cc-admin 用户登录并转到队列视图。

配置应同步，如下截图所示：

![图片](img/8cdb6780-a35f-4970-b496-94435efaa7b4.png)

图 6.2：加入集群后，所有配置都同步

要添加更多节点，让每个新节点加入集群中的另一个节点。第一个节点管理控制台中的概览选项卡显示了集群中的所有节点，这些节点是自动发现的，如下截图所示：

![图片](img/41c7dd44-7b0f-46f6-bbe1-df5bea88519e.png)

图 6.3：管理控制台概览显示了所有集群成员

如所示，集群的所有成员都列出了，包括基本统计信息和端口。**信息**列中显示的不同值如下：

+   basic：描述`rates_mode`，它告诉队列如何报告统计信息。这可以是`basic`（默认）、`detailed`或`none`之一。

+   disc: 表示节点将数据持久化到文件系统，这是默认行为。也可以以**RAM**模式启动节点，其中所有消息数据都存储在内存中，如果系统有足够的内存，这可以加快系统速度。

+   7：显示已启用的插件数量。

+   allocated: 描述内存计算策略。

可以通过`rabbitmqctl`（管理 RabbitMQ 服务器节点的命令行工具）从集群中移除节点（[`www.rabbitmq.com/clustering.html#breakup`](http://www.rabbitmq.com/clustering.html#breakup)）。

目前所有 CC 应用程序都连接到单个 RabbitMQ 节点。这需要修改。应用程序应首先尝试连接到一个节点，如果原始尝试失败，则切换到另一个节点。继续阅读以了解如何实现。

## 连接到集群

目前所有 CC 应用程序都连接到单个 RabbitMQ 节点，需要修改以利用集群的优势。所有连接到 RabbitMQ 的应用程序都需要修改。应用程序应首先尝试连接到一个节点，如果原始尝试失败，则切换到另一个节点。这是唯一需要更改的地方；应用程序将与代理交互，就像以前一样。

首先，修改主应用程序连接 Ruby 代码如下：

```java
begin
  connection = Bunny.new(
    hosts: ['rmq-prod-01', 'rmq-prod-02'])
  connection.start
  rescue Bunny::TCPConnectionFailed => e
    puts "Connection to server failed"
end
```

基本上，传递了代理地址列表。有了这个，RabbitMQ Ruby 客户端将连接到地址列表中的第一个响应节点，并尝试提供的每个代理地址，直到建立连接或最终失败。在失败的情况下，已经存在的整体重连机制将启动，并将再次尝试连接地址。

可以使用`rabbitmqctl sync_queue <queue_name>`命令手动同步镜像队列。使用`rabbitmqctl cancel_sync_queue <queue_name>`取消同步。

到目前为止，还有一步要执行以确保队列数据的高可用性：启用一种将数据传播到其他节点的方式。可用的选项是**经典镜像队列**和**仲裁队列**。但首先，需要了解一些分区处理策略。

## 分区处理策略

当然，向集群中添加更多节点是可能的。然而，这会带来一个新的挑战，即网络连接问题。当使用多个节点时，脑裂和早期消息确认是常见问题。在分布式系统中，当网络的一部分无法从另一部分访问时，就会发生脑裂，这会创建网络分区（称为**netsplit**）。为了避免这种情况，设置分区处理策略。在 RabbitMQ 中，这通过配置文件中的`cluster_partition_handling`参数来设置——[`www.rabbitmq.com/partitions.html#automatic-handling`](https://www.rabbitmq.com/partitions.html#automatic-handling)。

**pause-minority**策略终止少数分区中的节点。这是许多分布式网络解决脑裂问题的默认方式。**pause-if-all-down**功能仅在没有任何节点可访问时暂停节点。这不可取，因为它会在每个分区的数据之间造成很大的差异。

一旦在**pause-if-all-down**设置中可用节点，还有两个选项可以指定如何重新连接网络。简单地忽略另一个分区或自动修复集群。系统无法暂停的节点也必须指定。在**pause-minority**策略中，分区在可用时重新连接。

RabbitMQ 确保跨集群的同步。客户端可以通过任何节点访问其交换机和队列；然而，消息本身并不会在节点间传递。下一节将介绍如何实现这一点。

# 发现 RabbitMQ 队列的类型

RabbitMQ 中的队列可以是持久的或临时的。对于临时消息处理，建议使用经典镜像队列，而仲裁队列是持久队列的良好替代方案。

持久队列元数据存储在磁盘上，而临时队列在可能的情况下存储在内存中。另一种队列类型，懒队列，尽可能早地将持久和临时消息的内容写入磁盘。

由于经典镜像队列的技术限制，很难对如何处理故障做出保证。RabbitMQ 文档([`www.rabbitmq.com/ha.html`](https://www.rabbitmq.com/ha.html))建议用户熟悉仲裁队列，并在可能的情况下考虑它们而不是经典镜像队列。

## 队列镜像

在 CC 的情况下，队列中的数据需要高度可用。镜像队列提供了这种安全性。队列镜像使用主镜像设计模式。所有消息入队和出队操作都在主节点上进行，镜像节点定期从主节点接收更新。如果主节点不可用，RabbitMQ 会将一个镜像节点提升为主节点；通常，最老的镜像节点成为新的主节点，只要它保持同步。

还可以通过将数据发送到原始集群之外的不同集群来设置主主系统。这为硬件更新和极端故障情况提供了有用的备份。它还可以帮助加快不同地理区域之间的交互。

在我们的情况下，必须通过`Q_TTL_DLX`策略来告诉集群如何镜像队列，因为在一个队列或交换中一次只能允许一个策略。第一步是清除在第四章，*调整消息投递*中创建的策略，然后应用一个新的策略，该策略结合了`Q_TTL_DLX`策略和为队列镜像创建的策略。

运行以下命令以更改`Q_TTL_DLX`策略并告诉 RabbitMQ 如何镜像队列。首先清除策略：

```java
$ sudo rabbitmqctl clear_policy -p cc-prod-vhost Q_TTL_DLX
# => Clearing policy "Q_TTL_DLX"
# => ......done.
"Specify the new HA_Q_TTL_DLX policy:"
$ sudo rabbitmqctl set_policy -p cc-prod-vhost HA_Q_TTL_DLX "taxi\.\d+" '{"message-ttl":604800000, "dead-letter-exchange":"taxi-dlx", "ha-mode":"all", "ha-sync-mode":"automatic"}' --apply-to queues 
# => Setting policy "HA_Q_TTL_DLX" for pattern "taxi\.\d+" to "{\"ha-mode\":\"all\", \"message-ttl\":604800000, \"dead-letter-exchange\":\"taxi-dlx\"}" with priority "0" 
# => ......done.
```

或者，可以从管理控制台添加策略，如下面的截图所示：

![](img/0c6e86cc-eba3-4ca7-8762-804519431010.png)

图 6.4：通过 RabbitMQ 管理控制台添加的策略

已将高可用性模式添加到现有的 TTL 和 DLX 策略规则中。ha-mode 的 all 值告诉 RabbitMQ 在集群的所有节点上镜像队列，这正是 CC 在他们的双节点集群中想要的。其他选项是 exact 和 nodes，允许开发者在使用 exact 选项时指定节点数，在使用 nodes 选项时通过 ha-params 参数指定节点名称列表。

ha-sync-mode 参数用于指定镜像队列的同步模式。此参数可以设置为手动或自动。在手动模式下，新镜像队列不会接收任何现有消息，但最终会随着消费者检索消息而与主队列保持一致。这减少了开销，但以丢失信息为代价。自动模式将消息发送到每个队列，意味着对系统性能有轻微的影响。

CC 决定使用即时队列同步，以便任何现有消息几乎瞬间在所有节点上可见。CC 对由此产生的初始无响应性表示可以接受，因为性能对于用户消息不是关键。

在运行上述命令后，导航到管理控制台中的队列标签页。观察到的 HA_Q_TTL_DLX 策略已应用于目标队列：

![](img/a7c157a8-f91e-493b-a1cb-8d6574d9db2d.png)

图 6.5：应用了高可用性策略的镜像队列

注意镜像队列旁边有一个**+1**。这表示队列已镜像到集群中的另一个节点。在管理控制台的每个队列的详细信息部分也清楚地定义了主节点（rabbit@rmq-prod-1）和镜像节点（rabbit@rmq-prod-2），如下面的截图所示：

![图片](img/7ecc5ce1-9149-4f2e-afa5-558f7ca32c7a.png)

图 6.6：主节点和镜像节点细节

在这一点上，RabbitMQ 代理已集群化，出租车订单请求队列已镜像。客户端应用程序可以从中受益于这种高可用性部署并连接到不同的节点。

**设置主队列位置**：每个队列都有一个主副本，称为队列主。在同步之前，该队列是第一个接收消息的。可以通过在管理控制台的队列选项卡中使用 x-queue-master-locator 参数或在程序创建队列时来影响其设置。

仲裁队列是一种新型队列，通常比经典镜像队列更受推荐。

## 仲裁队列

作为持久镜像队列的替代方案，仲裁队列通过就队列内容达成一致来确保集群是最新的。在此过程中，仲裁队列避免了数据丢失，这在镜像队列中可能会在消息过早确认时发生。仲裁队列自 RabbitMQ 3.8.0 版本起可用。如 RabbitMQ 文档（[`www.rabbitmq.com/quorum-queues.html`](https://www.rabbitmq.com/quorum-queues.html)）中详细说明，当使用仲裁队列时，某些临时功能不可用。

队列仲裁队列有一个领导者，其作用大致与经典镜像队列主相同。所有通信都路由到队列领导者，这意味着队列领导者的本地性会影响消息的延迟和带宽需求；然而，这种影响应该低于经典镜像队列。

在仲裁队列中，领导者和复制是由共识驱动的，这意味着它们就队列的状态及其内容达成一致。虽然镜像队列可能会过早确认消息并丢失数据，但仲裁队列只有在大多数节点可用时才会确认，从而避免了数据丢失。

使用以下命令声明仲裁队列：

```java
rabbitmqadmin declare queue name=<name> durable=true arguments='{“x-queue-type”: “quorum”}'
```

这些队列必须是持久的，并且通过将`x-queue-type`报头设置为`quorum`来实例化。如果大多数节点就队列内容达成一致，则数据有效。否则，系统会尝试将所有队列更新到最新状态。

仲裁队列支持处理毒消息，这些消息永远不会被完全消费或确认。

可以跟踪并显示在`x-delivery-count`报头中的未成功投递尝试次数。当消息被返回的次数超过配置值时，可以将其作为毒消息进行死信处理。

懒队列是另一种值得探索的队列类型，所以请继续阅读。

## 懒队列

由于消费者维护或大量消息批次的到达等原因，队列可能会变得很长。虽然 RabbitMQ 可以支持数百万条消息，但大多数专家建议尽可能保持队列尽可能短。默认情况下，消息存储在内存中。当队列变得太长以至于底层实例无法处理时，RabbitMQ 会将消息（页面输出）刷新以释放 RAM 使用。在 RAM 中存储消息比将它们存储到磁盘上能更快地将消息传递给消费者。

页面输出功能通常需要时间，并且经常停止队列处理消息，这会降低队列速度。因此，包含大量消息的队列可能会对经纪人的性能产生负面影响。此外，在集群重启后重建索引需要花费大量时间，以及在不同节点之间同步消息也需要花费大量时间。

从 RabbitMQ 版本 3.6 开始，添加了一个名为 lazy queues 的策略，以自动将消息存储到磁盘，从而最小化 RAM 使用。可以通过设置`queue.declare`参数或应用策略到所有队列来启用 lazy queues。

持久消息可以在进入经纪人时写入磁盘，并同时保存在 RAM 中。

已经展示了不同的队列类型，现在是时候看看 CC 应该如何处理来自所有集群的日志聚合了。

# 使用联邦经纪人和日志聚合

创建两个 RabbitMQ 经纪人集群的方式实际上与在创建一个高度可用的关系数据库时通常所做的是非常相似的。数据库仍然是一个集中的资源，提供高可用性保证。然而，当涉及到高可用性时，RabbitMQ 并非只有一项绝技。

要形成一个 RabbitMQ 系统的图像，以下两个插件允许经纪人连接：

+   **Shovel**: 在不同经纪人之间连接队列和交易所

+   **联邦**: 为队列到队列或交易所到交易所形成跨经纪人连接

这两个插件通过按指令路由消息或提供一个安全的地方让它们在可以处理之前保持不变，确保了消息在经纪人之间可靠地传递。它们都不需要经纪人集群，这简化了设置和管理。此外，这两个插件在 WAN 连接上也能正常工作，这在集群场景中是不常见的。

手动配置联邦中的目标节点。上游节点会自动配置。另一方面，铲子必须手动配置每个源节点以发送到目标节点，而目标节点本身不需要任何配置。

CC 团队正在寻求一种处理日志的好方法，他们很快意识到联邦插件非常适合这个过程。

## 处理日志处理

CC 的系统正在不断增长，出租车司机和开发者的团队也在不断扩大。负责分析的那个团队一直在寻找一个优雅的解决方案来聚合来自不同应用的日志，以便推出新的统计数据，供内部和最终用户使用。幸运的是，由于 RabbitMQ 的高性能，它可以用于应用日志处理。

在这个拓扑中，所有应用都将写入本地 RabbitMQ 节点，该节点将充当存储和转发代理，将所有日志推送到中心 RabbitMQ 节点，如下面的图所示：

![图片](img/0e4ea0d7-dc9c-4cd4-bd9d-79f54c14dedd.png)

图 6.7：将日志消息联邦到一个中心代理的拓扑

如果这个中心节点宕机，日志条目将保留在本地累积，直到它恢复。消息通过一个位置（称为**上游**）的交换流动，以复制到其他位置的交换（**下游**），如下面的图所示：

![图片](img/b8e50b47-c62f-43b2-805b-433ad9bcc1ff.png)

图 6.8：交换联邦消息流

显然，这里的假设是本地 RabbitMQ 节点非常稳定。过去几个月运行 RabbitMQ 的经验将有助于这种方法。此外，日志被认为对 CC 来说很重要，但不是关键数据，因此尽力而为的方法是可以接受的。了解这一点后，团队选择使用联邦插件，因为它支持联邦到队列连接（如果使用铲子插件，消息将需要在每个节点的本地队列中累积）。

记住，上一节中镜像的所有队列都是匹配`taxi-inbox\.\d+`正则表达式的队列。现在提到的所有日志队列都被排除在外。这就是 CC 团队想要的，因为他们不希望镜像这样高流量的队列。为了使 CC 能够享受相同的日志聚合保证，可以做什么呢？引入消息拓扑的概念。

更多关于铲子插件的信息可以在[`www.rabbitmq.com/shovel.html`](http://www.rabbitmq.com/shovel.html)找到。

需要在所有将参与拓扑的 RabbitMQ 节点上安装联邦插件，通过在每个节点上运行以下命令：

```java
 $ sudo rabbitmq-plugins enable rabbitmq_federation 
Applying plugin configuration to rabbit@app-prod-1...

$ sudo rabbitmq-plugins enable rabbitmq_federation_management
Applying plugin configuration to rabbit@app-prod-1...
```

此外，与集群不同，每个节点都需要手动设置以配置所需的用户和虚拟主机。如第一章“Rabbit Springs to Life”中讨论的那样，现在是运行必要命令的时候了。接下来，必须配置`apps-log`交换联邦本身。这涉及到多个步骤（稍后详细介绍），所有这些步骤都在中心代理上运行，即所有日志将汇聚的下游代理。

首先，配置上游，这些是向中心代理发送数据的 RabbitMQ 节点。需要三个上游，因为有三个服务器将发送日志，`app-prod-1`、`app-prod-2`和`app-prod-3`；然而，为了简洁起见，以下示例中只显示两个节点。

可以通过`rabbitmqctl`添加上游：

```java
# Adds a federation upstream named "app-prod-logs"
rabbitmqctl -p logs-prod set_parameter federation-upstream app-prod-logs '{"uri":"amqp://cc-prod:******@app-prod-1:5672/cc-prod-vhost"}'
```

或者，可以通过管理控制台添加策略：

![图片](img/492e034a-7261-4b8d-bd87-4ee399fe4e3b.png)

图 6.9：将名为 app-prod-logs 的联盟上游添加到下游代理

一旦在下游指定了上游，就可以将控制联盟的策略添加到下游服务器。app-prod-logs 联盟的添加就像任何其他策略一样（[`www.rabbitmq.com/parameters.html#policies`](https://www.rabbitmq.com/parameters.html#policies)），通过使用终端：

```java
rabbitmqctl set_policy -p logs-prod --apply-to exchanges log-exchange-federation "^app-logs*" '{"federation-upstream-set":"all"}' --apply-to exchanges
```

策略也可以通过管理控制台添加：

![图片](img/1049fad9-ee30-4263-8790-44fd71eee88e.png)

图 6.10：将联盟策略添加到下游服务器

CC 团队通过应用匹配交换名称的策略来完成此操作。模式参数是一个用于匹配队列（或交换）名称的正则表达式。在 CC 的情况下，联盟策略应用于所有以`app-prod`开头的交换。

策略可以应用于上游集、单个交换或队列上游。在这个例子中，`federation-upstream-set`应用于所有上游。

如果确定永远不会存在超过一个逻辑上游组，则可以跳过创建上游集，转而使用名为`all`的隐式集，该集自动包含虚拟主机中的所有上游。

在这种情况下，确保联盟插件将在中心代理中用于与联盟交换进行交互的用户也已配置。

浏览到管理控制台管理部分中的**联盟上游**选项卡，它将显示上游已正确配置，如下面的屏幕截图所示：

![图片](img/36c5915b-6424-4c89-90c1-0f61ed9f410b.png)

图 6.11：在联盟中配置上游节点

切换到**联盟状态**会显示一个空屏幕，因为它尚未激活。为什么会这样？毕竟，拓扑刚刚创建。原因是还没有任何交换或队列在拓扑中活跃。由于其动态特性，联盟处于非活动状态。在返回到联盟状态选项卡之前，下一步是在上游和下游服务器上创建 app-logs 交换并将其绑定到队列。这里需要注意的是，联盟现在正在从配置集的两个上游节点为 app-logs 交换运行链接。请参阅以下屏幕截图：

![图片](img/44088094-165a-49eb-8770-b0ca2dcfb5f4.png)

图 6.12：运行联盟交换的上游链接

可以通过在下游节点上运行 `sudo rabbitmqctl eval rabbit_federation_status:status()` 命令从命令行获取联盟的状态。

管理控制台中的连接和通道选项卡现在显示下游节点通过 AMQP 协议连接到上游节点，如下面的屏幕截图所示：

![图片](img/b5739316-b32d-44c0-88d7-1ce4267bd0c8.png)

图 6.13：连接选项卡中的联盟链接

除了拓扑结构的设置本身，联盟没有神秘之处。它是建立在 AMQP 之上的，因此能够从该协议提供的相同优势中受益。因此，如果 RabbitMQ 实例被防火墙隔离，除了默认情况下 AMQP 使用的端口（`5672`）之外，不需要打开任何特殊端口。

更多关于联盟插件的详细信息，请参阅[`www.rabbitmq.com/federation.html`](http://www.rabbitmq.com/federation.html)和[`www.rabbitmq.com/federation-reference.html`](http://www.rabbitmq.com/federation-reference.html)。

# 摘要

CC 示例提供了有关如何创建基本消息队列架构、添加满足用户需求的有价值功能以及使系统无故障运行的信息。本章介绍了 RabbitMQ 如何通过集群和联盟提供强大的功能，以及这些功能如何提高消息基础设施的可用性和整体弹性。还探讨了法定人数、经典镜像和懒队列。

在此过程中，提供了有关构建可靠、弹性系统的最佳实践的信息和指导。下一章强调了这些建议，并从 CC 通过 RabbitMQ 的旅程中总结了关键要点。它还探讨了 RabbitMQ 的监控。
