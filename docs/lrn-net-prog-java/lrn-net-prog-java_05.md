# 第五章：点对点网络

**点对点**（**P2P**）计算机网络指的是一种架构，其节点经常充当服务器和客户端。P2P 系统的主要目标是消除需要单独的服务器来管理系统的需求。P2P 网络的配置将随着节点以不可预测的方式加入和离开网络而动态变化。节点可能在处理速度、带宽支持和存储能力等因素上有所不同。对等方这个术语意味着节点之间的平等性。

对 P2P 网络有各种定义和解释。它们可以被描述为分散的、不断变化的、自我调节的架构。服务器倾向于提供服务，而客户端请求服务。P2P 节点通常两者兼而有之。纯 P2P 网络不会有被指定为客户端或服务器的节点。实际上，这些网络很少见。大多数 P2P 网络依赖于中央设施，如 DNS 服务器，提供支持。

某些网络可能是客户端/服务器架构和更纯粹的 P2P 架构之间的混合体，在这种情况下，从不会有特定的节点充当“主”服务器。例如，文件共享 P2P 可能使用网络的节点来下载文件，而服务器可能提供额外的支持信息。

P2P 可以以多种方式分类。我们将使用一些常见的分类类别，有助于理解 P2P 网络的性质。一个分类是基于**索引**的执行过程，即找到一个节点的过程：

+   **集中式**：这是指一个中央服务器跟踪对等方之间数据位置的过程

+   **本地**：这是指每个对等方跟踪自己的数据的情况

+   **分布式**：这是指数据引用由多个对等方维护的情况

混合 P2P 网络使用集中式索引方案。纯 P2P 网络使用本地或分布式索引。

算法用于确定系统中信息的位置。系统是分散的，没有执行算法的主服务器。该算法支持一个自组织的系统，随着节点的加入和移除而动态重新配置自身。此外，这些系统理想情况下会在网络成员变化时平衡负载和资源。

在本章中，我们将涵盖：

+   P2P 的概念和术语

+   Java 对 P2P 网络的支持

+   分布式哈希表的性质

+   FreePastry 如何支持 P2P 应用程序

### 注意

P2P 应用程序提供了传统客户端/服务器架构的灵活替代方案。

# P2P 功能/特征

理解 P2P 网络的一种方式是审视其特征。这些特征包括以下内容：

+   向系统提供资源的节点，包括：

+   数据存储

+   计算资源

+   它们提供一系列服务的支持

+   它们非常可扩展和容错

+   它们支持资源的负载平衡

+   它们可能支持有限的匿名性

P2P 系统的性质是用户可能无法访问特定节点以使用服务或资源。随着节点随机加入和离开系统，特定节点可能不可用。算法将决定系统如何响应请求。

P2P 系统的基本功能包括：

+   对等方在网络中的注册

+   对等方发现-确定哪个对等方拥有感兴趣的信息的过程

+   对等方之间的消息传递

并非所有对等方都执行所有这些功能。

P2P 系统的资源使用**全局唯一标识符**（**GUID**）进行标识，通常使用安全哈希函数生成，我们将在 DHT 组件中进行讨论。GUID 不打算供人类阅读。它是一个随机生成的值，几乎没有冲突的机会。

P2P 的节点使用**路由** **覆盖**进行组织。这是一种将请求路由到适当节点的**中间件**类型。覆盖指的是位于物理网络之上的网络，由 IP 地址标识资源。我们可以将网络构想为由一系列基于 IP 的节点组成。然而，覆盖是这些节点的一个子集，通常专注于单一任务。

路由覆盖将考虑因素，例如用户和资源之间的节点数量，以及连接的带宽，以确定哪个节点应该满足请求。资源经常会被复制或甚至分布在多个节点之间。路由覆盖将尝试提供到资源的最佳路径。

随着节点加入和离开系统，路由覆盖需要考虑这些变化。当一个节点加入系统时，可能会被要求承担一些责任。当一个节点离开时，系统的其他部分可能需要承担一些离开节点的责任。

在本章中，我们将解释通常作为系统一部分嵌入的各种概念。我们将简要概述不同的 P2P 应用程序，然后讨论 Java 对这种架构的支持。我们演示了分布式哈希表的使用，并深入研究了 FreePastry，这将使我们了解许多 P2P 框架的工作原理。

在适当的情况下，我们将说明如何手动实现一些这些概念。虽然不需要使用这些实现来使用系统，但它们将提供对这些基本概念的更深入的理解。

# 基于应用程序的 P2P 网络

有许多基于 P2P 网络的应用程序。它们可以用于以下用途：

+   **内容分发**：这是文件共享（文件、音乐、视频、图像）

+   **分布式计算**：这是将问题分解为较小任务并以并行方式执行的情况

+   **Collaboration**：这是用户共同解决共同问题时的情况

+   **平台**：这些是构建 P2P 应用程序的系统，如 JXTA 和 Pastry

分布式计算利用更多数量的较小计算机的能力来执行任务。这种方法适用的问题需要被分解成较小的单元，然后并行在多台机器上执行。然后需要将这些较小任务的结果组合起来产生最终结果。

P2P 网络支持许多应用程序，例如以下应用程序：

+   **Skype**：这是一个视频会议应用程序

+   **Freecast**：这是一个点对点的流媒体音频程序

+   **BitTorrent**：这是一个流行的点对点文件共享系统

+   **Tor**：这个程序可以保护用户的身份

+   **Haihaisoft**：用于分发预先录制的电视节目

+   **WoW**：这使用 P2P 进行游戏更新

+   **YaCy**：这是一个搜索引擎和网络爬虫

+   **Octoshape**：支持实时电视

在[`p2peducation.pbworks.com/w/page/8897427/FrontPage`](http://p2peducation.pbworks.com/w/page/8897427/FrontPage)上可以找到对 P2P 应用程序的很好概述。

# Java 对 P2P 应用程序的支持

Java 支持超出了在早期章节中详细介绍的低级套接字支持，包括各种框架。这些框架从众所周知的框架（如 JXTA）到小型的功能有限的协议都有。这些框架为更专业的应用程序提供了基础。

下表列出了几个这些框架：

| P2P framework | URL |
| --- | --- |
| TomP2P | [`tomp2p.net/`](http://tomp2p.net/) |
| JXTA | [`jxta.kenai.com/`](https://jxta.kenai.com/) |
| Hive2Hive | [`github.com/Hive2Hive/Hive2Hive`](https://github.com/Hive2Hive/Hive2Hive) |
| jnmp2p | [`code.google.com/p/jnmp2p/`](https://code.google.com/p/jnmp2p/) |
| FlexGP | [`flexgp.github.io/flexgp/javalibrary.html`](http://flexgp.github.io/flexgp/javalibrary.html) |
| JMaay | [`sourceforge.net/projects/jmaay/`](http://sourceforge.net/projects/jmaay/) |
| P2P-MPI | [`grid.u-strasbg.fr/p2pmpi/`](http://grid.u-strasbg.fr/p2pmpi/) |
| Pastry | [`www.freepastry.org/`](http://www.freepastry.org/) |

这些框架使用算法在对等方之间路由消息。哈希表经常构成这些框架的基础，如下所讨论的。

# 分布式哈希表

**分布式哈希表**（**DHT**）使用键/值对在网络中定位资源。这个映射函数分布在对等方之间，使其分布式。这种架构使得 P2P 网络可以轻松扩展到大量节点，并且可以处理对等方随机加入和离开网络。DHT 是支持核心 P2P 服务的基础。许多应用使用 DHT，包括 BitTorrent、Freenet 和 YaCy。

以下图示了将键映射到值。键通常是包含资源标识的字符串，比如书名；值是用来表示资源的数字。这个数字可以用来在网络中定位资源，并且可以对应于节点的标识符。

![分布式哈希表](img/B04915_05_01.jpg)

P2P 网络已经使用了一段时间。这些网络的演变反映在资源如何被映射，如 Napster、Gnutella 和 Freenet 所体现的那样。

+   Napster ([`en.wikipedia.org/wiki/Napster`](https://en.wikipedia.org/wiki/Napster))是第一个大规模 P2P 内容传送系统。它使用服务器来跟踪网络中的节点。节点保存实际数据。当客户端需要这些数据时，服务器将查找保存数据的当前节点集，并将该节点的位置发送回客户端。然后客户端将联系保存数据的节点。这使得它容易受到攻击，并最终通过诉讼导致了它的消亡。

+   Gnutella ([`web.archive.org/web/20080525005017`](https://web.archive.org/web/20080525005017), [`www.gnutella.com/`](http://www.gnutella.com/))不使用中央服务器，而是向网络中的每个节点广播。这导致网络被消息淹没，并且这种方法在后来的版本中被修改。

+   Freenet ([`freenetproject.org/`](https://freenetproject.org/))使用启发式基于键的路由方案，专注于审查和匿名问题。然而，DHS 使用更结构化的基于键的路由方法，导致以下结果：

+   去中心化

+   容错性

+   可伸缩性

+   效率

然而，DHT 不支持精确匹配搜索。如果需要这种类型的搜索，就必须添加。

## DHT 组件

**键空间**是一组用于标识元素的 160 位字符串（键）。**键空间分区**是将键空间分割成网络节点的过程。覆盖网络连接节点。

常用的哈希算法是**安全哈希算法**（**SHA-1**）([`en.wikipedia.org/wiki/SHA-1`](https://en.wikipedia.org/wiki/SHA-1))。SHA-1 是由 NSA 设计的，生成一个称为消息摘要的 160 位哈希值。大多数 P2P 不需要开发人员显式执行哈希函数。但是，了解如何执行是有益的。以下是使用 Java 创建摘要的示例。

`MessageDigest`类的`getInstance`方法接受一个指定要使用的算法的字符串，并返回一个`MessageDigest`实例。它的`update`方法需要一个包含要哈希的键的字节数组。在这个例子中，使用了一个字符串。`digest`方法返回一个包含哈希值的字节数组。然后将字节数组显示为十六进制数：

```java
        String message = "String to be hashed";
        try {
            MessageDigest messageDigest = 
                MessageDigest.getInstance("SHA-1");
            messageDigest.update(message.getBytes());
            byte[] digest = messageDigest.digest();

            StringBuffer buffer = new StringBuffer();
            for (byte element : digest) {
                buffer.append(Integer
                    .toString((element & 0xff) + 0x100, 16)
                    .substring(1));
            }
            System.out.println("Hex format : " + 
                buffer.toString());

        } catch (NoSuchAlgorithmException ex) {
            // Handle exceptions
        }
```

执行这个序列将产生以下输出：

**十六进制格式：434d902b6098ac050e4ed79b83ad93155b161d72**

要存储数据，比如文件，我们可以使用文件名创建一个键。然后使用 put 类型函数来存储数据：

```java
put(key, data) 
```

要检索与密钥对应的数据，使用 get 类型函数：

```java
data = get(key)
```

覆盖中的每个节点都包含由密钥表示的数据，或者它是更接近包含数据的节点的节点。路由算法确定了前往包含数据的节点的下一个节点。

## DHT 实现

有几种 Java 实现的 DHT，如下表所示：

| 实现 | URL |
| --- | --- |
| openkad | [`code.google.com/p/openkad/`](https://code.google.com/p/openkad/) |
| Open Chord | [`open-chord.sourceforge.net/`](http://open-chord.sourceforge.net/) |
| TomP2P | [`tomp2p.net/`](http://tomp2p.net/) |
| JDHT | [`dks.sics.se/jdht/`](http://dks.sics.se/jdht/) |

我们将使用**Java 分布式哈希表**（**JDHT**）来说明 DHT 的使用。

## 使用 JDHT

为了使用 JDHT，您需要以下表中列出的 JAR 文件。`dks.jar`文件是主要的 jar 文件。但是，JDHT 还使用其他两个 JAR 文件。`dks.jar`文件的备用来源如下所示：

| JAR | 网站 |
| --- | --- |
| `dks.jar` |

+   [`dks.sics.se/jdht/`](http://dks.sics.se/jdht/)

+   [`www.ac.upc.edu/projects/cms/browser/cms/trunk/lib/dks.jar?rev=2`](https://www.ac.upc.edu/projects/cms/browser/cms/trunk/lib/dks.jar?rev=2)

|

| `xercesImpl.jar` | [`www.java2s.com/Code/Jar/x/DownloadxercesImpljar.htm`](http://www.java2s.com/Code/Jar/x/DownloadxercesImpljar.htm) |
| --- | --- |
| Apache log4j 1.2.17 | [`logging.apache.org/log4j/1.2/download.html`](https://logging.apache.org/log4j/1.2/download.html) |

以下示例已经改编自网站上的示例。首先，我们创建一个`JDHT`实例。JDHT 使用端口`4440`作为其默认端口。有了这个实例，我们可以使用它的`put`方法将键/值对添加到表中：

```java
    try {
        JDHT DHTExample = new JDHT();
        DHTExample.put("Java SE API", 
           "http://docs.oracle.com/javase/8/docs/api/");
        ...
    } catch (IOException ex) {
        // Handle exceptions
    }
```

为了使客户端能够连接到此实例，我们需要获取对此节点的引用。如下所示实现：

```java
    System.out.println(((JDHT) DHTExample).getReference());
```

以下代码将使程序保持运行，直到用户终止它。然后使用`close`方法关闭表：

```java
    Scanner scanner = new Scanner(System.in);
    System.out.println("Press Enter to terminate application: ");
    scanner.next();
    DHTExample.close();
```

当程序执行时，您将获得类似以下的输出：

**dksref://192.168.1.9:4440/0/2179157225/0/1952355557247862269**

**按 Enter 键终止应用程序：**

客户端应用程序描述如下。使用不同的端口创建一个新的 JDHT 实例。第二个参数是对第一个应用程序的引用。您需要复制引用并将其粘贴到客户端中。每次执行第一个应用程序时，都会生成一个不同的引用：

```java
    try {
        JDHT myDHT = new JDHT(5550, "dksref://192.168.1.9:4440" 
            + "/0/2179157225/0/1952355557247862269");
        ...
    } catch (IOException | DKSTooManyRestartJoins | 
             DKSIdentifierAlreadyTaken | DKSRefNoResponse ex) {
        // Handle exceptions
    }
```

接下来，我们使用`get`方法检索与密钥关联的值。然后显示该值并关闭应用程序：

```java
    String value = (String) myDHT.get("Java SE API");
    System.out.println(value);
    myDHT.close();
```

输出如下：

**http://docs.oracle.com/javase/8/docs/api/**

这个简单的演示说明了分布式哈希表的基础知识。

# 使用 FreePastry

Pastry（[`www.freepastry.org/`](http://www.freepastry.org/)）是一个 P2P 路由覆盖系统。FreePastry（[`www.freepastry.org/FreePastry/`](http://www.freepastry.org/FreePastry/)）是 Pastry 的开源实现，足够简单，可以用来说明 P2P 系统的许多特性。Pastry 将在*O(log n)*步骤中路由具有*n*节点网络的消息。也就是说，给定一个节点网络，最多需要*log2 n*步骤才能到达该节点。这是一种高效的路由方法。但是，虽然只需要遍历三个节点就可以获得资源，但可能需要大量的 IP 跳数才能到达它。

Pastry 在路由过程中使用**叶集**的概念。每个节点都有一个叶集。叶集是此节点数字上最接近的节点的 GUIDS 和 IP 地址的集合。节点在逻辑上排列成一个圆圈，如下所示。

在下图中，每个点代表一个带有标识符的节点。这里使用的地址范围从`0`到`FFFFFF`。真实地址范围从`0`到`2128`。如果代表请求的消息起源于地址`9341A2`并且需要发送到地址`E24C12`，那么基于数字地址，覆盖路由器可能通过中间节点路由消息，如箭头所示：

![使用 FreePastry](img/B04915_05_02.jpg)

其他应用程序已构建在 FreePastry 之上，包括：

+   **SCRIBE**：这是一个支持发布者/订阅者范式的组通信和事件通知系统

+   **PAST**：这是一个存档存储实用程序系统

+   **SplitStream**：该程序支持内容流和分发

+   **Pastiche**：这是备份系统

每个应用程序都使用 API 来支持它们的使用。

## FreePastry 演示

为了演示 FreePastry 如何支持 P2P 应用程序，我们将创建一个基于[`trac.freepastry.org/wiki/FreePastryTutorial`](https://trac.freepastry.org/wiki/FreePastryTutorial)中找到的 FreePastry 教程的应用程序。在这个演示中，我们将创建两个节点，并演示它们如何发送和接收消息。演示使用三个类：

+   `FreePastryExample`：这用于引导网络

+   `FreePastryApplication`：这执行节点的功能

+   `PastryMessage`：这是在节点之间发送的消息

让我们从引导应用程序开始。

### 了解 FreePastryExample 类

有几个组件与 FreePastry 应用程序一起使用。这些包括：

+   **环境**：这个类代表应用程序的环境

+   **绑定端口**：这代表应用程序将绑定到的本地端口

+   **引导端口**：这是用于节点的`InetAddress`的引导端口

+   **引导地址**：这是引导节点的 IP 地址

接下来定义`FreePastryExample`类。它包含一个主方法和一个构造函数：

```java
public class FreePastryExample {
    ...
}
```

我们将从`main`方法开始。首先创建`Environment`类的实例。这个类保存节点的参数设置。接下来，将 NAT 搜索策略设置为从不，这样我们就可以在本地 LAN 中使用程序而不会有困难：

```java
    public static void main(String[] args) throws Exception {
        Environment environment = new Environment();
        environment.getParameters()
            .setString("nat_search_policy", "never");
        ...
    }
```

端口和`InetSocketAddress`实例被初始化。我们将此时两个端口设置为相同的数字。我们使用 IP 地址`192.168.1.14`来实例化`InetAddress`对象。您需要使用您的机器的地址。这是一个本地 LAN 地址。不要使用`127.0.0.1`，因为它将无法正常工作。`InetAddress`对象以及`bootPort`值用于创建`InetSocketAddress`实例。所有这些都放在 try 块中来处理异常：

```java
    try {
        int bindPort = 9001;
        int bootPort = 9001;
        InetAddress bootInetAddress = 
            InetAddress.getByName("192.168.1.14"); 
        InetSocketAddress bootAddress = 
                new InetSocketAddress(bootInetAddress, bootPort);
        System.out.println("InetAddress: " + bootInetAddress);
        ...
    } catch (Exception e) {
        // Handle exceptions
    }
```

最后一个任务是通过调用构造函数创建`FreePastryExample`类的实例：

```java
    FreePastryExample freePastryExample = 
        new FreePastryExample(bindPort, bootAddress, environment);
```

构造函数将创建并启动节点的应用程序。为了实现这一点，我们需要创建一个`PastryNode`实例，并将应用程序附加到它上面。为了创建节点，我们将使用一个工厂。

每个节点都需要一个唯一的 ID。`RandomNodeIdFactory`类根据当前环境生成 ID。使用此对象与绑定端口和环境，创建`SocketPastryNodeFactory`的实例。使用此工厂调用`newNode`方法来创建我们的`PastryNode`实例：

```java
    public FreePastryExample(int bindPort, 
            InetSocketAddress bootAddress, 
            Environment environment) throws Exception {
        NodeIdFactory nidFactory = 
            new RandomNodeIdFactory(environment);
        PastryNodeFactory factory = 
            new SocketPastryNodeFactory(
                nidFactory, bindPort, environment);
        PastryNode node = factory.newNode();
        ...
    }
```

接下来，创建`FreePastryApplication`类的实例，并使用`boot`方法启动节点：

```java
    FreePastryApplication application = 
        new FreePastryApplication(node);
    node.boot(bootAddress);
    ...
```

然后显示节点的 ID，如下一个代码序列所示。由于网络中会有多个节点，我们暂停 10 秒钟，以便其他节点启动。我们使用 FreePastry 计时器来实现这种延迟。创建一个随机节点 ID，并调用应用程序的`routeMessage`消息将消息发送到该节点：

```java
    System.out.println("Node " + node.getId().toString() + " created");
    environment.getTimeSource().sleep(10000);
    Id randomId = nidFactory.generateNodeId();
    application.routeMessage (randomId);
```

在执行程序之前，我们需要开发应用程序类。

### 了解 FreePastryApplication 类

`FreePastryApplication`类实现了`Application`接口，并实现了节点的功能。构造函数创建并注册了一个`Endpoint`实例，并初始化了一个消息。节点使用`Endpoint`实例来发送消息。以下是类和构造函数的示例：

```java
public class FreePastryApplication implements Application {
    protected Endpoint endpoint;
    private final String message;
    private final String instance = " Instance ID";

    public FreePastryApplication(Node node) {
        this.endpoint = node.buildEndpoint(this, instance);
        this.message = "Hello there! from Instance: "
                + instance + " Sent at: [" + getCurrentTime() 
                + "]";
        this.endpoint.register();
    }

    ...
}
```

当这段代码被编译时，您可能会收到“在构造函数中泄漏 this”的警告。这是由于使用`this`关键字将构造函数的对象引用作为参数传递给`buildEndpoint`方法。这是一个潜在的不良实践，因为在传递时对象可能尚未完全构造。另一个线程可能会在对象准备好之前尝试对其进行操作。如果它被传递给执行常见初始化的包私有方法，那么这不会是太大的问题。在这种情况下，它不太可能引起问题。

`Application`接口要求实现三种方法：

+   `deliver`：当接收到消息时调用

+   `forward`：用于转发消息

+   `update`：通知应用程序一个节点已加入或离开了一组本地节点

我们只对这个应用程序中的`deliver`方法感兴趣。此外，我们将添加`getCurrentTime`和`routeMessage`方法到应用程序中。我们将使用`getCurrentTime`方法来显示我们发送和接收消息的时间。`routeMessage`方法将向另一个节点发送消息。

`getCurrentTime`方法如下。它使用`EndPoint`对象来访问节点的环境，然后获取时间：

```java
    private long getCurrentTime() {
        return this.endpoint
                .getEnvironment()
                .getTimeSource()
                .currentTimeMillis();
    }
```

`routeMessage`方法传递了目标节点的标识符。消息文本是通过添加端点和时间信息来构造的。使用端点标识符和消息文本创建了一个`PastryMessage`实例。然后调用`route`方法来发送这条消息：

```java
    public void routeMessage(Id id) {
        System.out.println(
                "Message Sent\n\tCurrent Node: " +
                   this.endpoint.getId()
                + "\n\tDestination: " + id
                + "\n\tTime: " + getCurrentTime());
        Message msg = new PastryMessage(endpoint.getId(), 
            id, message);
        endpoint.route(id, msg, null);
    }
```

当节点接收到消息时，将调用`deliver`方法。该方法的实现如下。显示了端点标识符、消息和到达时间。这将帮助我们理解消息是如何发送和接收的：

```java
    public void deliver(Id id, Message message) {
        System.out.println("Message Received\n\tCurrent Node: " 
            + this.endpoint.getId() + "\n\tMessage: " 
            + message + "\n\tTime: " + getCurrentTime());
    }
```

`PastryMessage`类实现了`Message`接口，如下所示。构造函数接受目标、源和消息：

```java
public class PastryMessage implements Message {
  private final Id from;
  private final Id to;
  private final String messageBody;

  public PastryMessage(Id from, Id to, String messageBody) {
    this.from = from;
    this.to = to;
    this.messageBody = messageBody;
  }

    ...
}
```

`Message`接口拥有一个需要被重写的`getPriority`方法。在这里，我们返回一个低优先级，以便它不会干扰底层的 P2P 维护流量：

```java
  public int getPriority() {
    return Message.LOW_PRIORITY;
  }
```

`toString`方法被重写以提供消息的更详细描述：

```java
  public String toString() {
    return "From: " + this.from 
            + " To: " + this.to 
            + " [" + this.messageBody + "]";
  }
```

现在，我们准备执行示例。执行`FreePastryExample`类。初始输出将包括以下输出。显示了缩写的节点标识符，本例中为`<0xB36864..>`。您得到的标识符将会不同：

**InetAddress：/192.168.1.14 节点<0xB36864..>已创建**

之后，发送一个暂停消息，随后当前节点接收到该消息。这条消息是在`FreePastryExample`类中创建的，以下是重复的代码以供您参考：

```java
    Id randomId = nidFactory.generateNodeId();
    application.routeMessage(randomId);
```

使用了一个随机标识符，因为我们没有特定的节点来发送消息。当消息被发送时，将生成以下输出。本次运行的随机标识符是`<0x83C7CD..>`：

**消息已发送**

**当前节点：<0xB36864..>**

**目标：<0x83C7CD..>**

**时间：1441844742906**

**消息已接收**

**当前节点：<0xB36864..>**

**消息：从：<0xB36864..> 到：<0x83C7CD..> [你好！来自实例：实例 ID 发送于：[1441844732905]]**

**时间：1441844742915**

发送和接收消息之间的时间是最短的。如果 P2P 网络由更大的节点集合组成，将会出现更显著的延迟。

在先前的输出中，节点地址被截断了。我们可以使用`toStringFull`方法，如下所示，来获取完整的地址：

```java
    System.out.println("Node " + node.getId().toStringFull() 
       + " created");
```

这将产生类似以下的输出：

**节点 B36864DE0C4F9E9C1572CBCC095D585EA943B1B4 已创建**

我们没有为消息提供特定的地址。相反，我们随机生成了地址。这个应用程序演示了 FreePastry 应用程序的基本元素。其他层用于促进节点之间的通信，比如 Scribe 支持的发布者/提供者范式。

我们可以使用相同的程序启动第二个节点，但我们需要使用不同的绑定端口以避免绑定冲突。任一节点发送的消息不一定会被另一个节点接收。这是 FreePastry 生成的路由的结果。

### 向特定节点发送消息

要直接向节点发送消息，我们需要其标识符。要获取远程节点的标识符，我们需要使用叶集。这个集合不严格地是一个集合，因为对于小型网络（比如我们正在使用的网络），同一个节点可能会出现两次。

`LeafSet`类代表这个集合，并且有一个`get`方法，将为每个节点返回一个`NodeHandle`实例。如果我们有这个节点句柄，我们可以向节点发送消息。

为了演示这种方法，将以下方法添加到`FreePastryApplication`类中。这类似于`routeMessage`方法，但它使用节点句柄作为`route`方法的参数：

```java
    public void routeMessageDirect(NodeHandle nh) {
        System.out.println("Message Sent Direct\n\tCurrent Node: "
                + this.endpoint.getId() + " Destination: " + nh
                + "\n\tTime: " + getCurrentTime());
        Message msg = 
            new PastryMessage(endpoint.getId(), nh.getId(),
                "DIRECT-" + message);
        endpoint.route(null, msg, nh);
    }
```

将以下代码序列添加到`FreePastryExample`构造函数的末尾。可选择注释掉使用`routeMessage`方法的先前代码。首先，我们暂停 10 秒，以便其他节点加入网络：

```java
    environment.getTimeSource().sleep(10000);

```

接下来，我们创建`LeafSet`类的一个实例。`getUniqueSet`方法返回叶集，不包括当前节点。然后，for-each 语句将使用`routeMessageDirect`变量将消息发送到集合的节点：

```java
    LeafSet leafSet = node.getLeafSet();
    Collection<NodeHandle> collection = leafSet.getUniqueSet();
    for (NodeHandle nodeHandle : collection) {
        application.routeMessageDirect(nodeHandle);
        environment.getTimeSource().sleep(1000);
    }

```

使用绑定端口`9001`启动`FreePastryExample`类。然后，将绑定端口更改为`9002`，并再次启动该类。几秒钟后，您将看到类似以下输出。第一组输出对应应用程序的第一个实例，而第二组对应第二个实例。每个实例将向另一个实例发送一条消息。请注意消息发送和接收时使用的时间戳：

```java
InetAddress: /192.168.1.9
Node <0xA5BFDA..> created
Message Sent Direct
 Current Node: <0xA5BFDA..> Destination: [SNH: <0x2C6D18..>//192.168.1.9:9002]
 Time: 1441849240310
Message Received
 Current Node: <0xA5BFDA..>
 Message: From: <0x2C6D18..> To: <0xA5BFDA..> [DIRECT-Hello there! from Instance: Instance ID Sent at: [1441849224879]]
 Time: 1441849245038

InetAddress: /192.168.1.9
Node <0x2C6D18..> created
Message Received
 Current Node: <0x2C6D18..>
 Message: From: <0xA5BFDA..> To: <0x2C6D18..> [DIRECT-Hello there! from Instance: Instance ID Sent at: [1441849220308]]
 Time: 1441849240349
Message Sent Direct
 Current Node: <0x2C6D18..> Destination: [SNH: <0xA5BFDA..>//192.168.1.9:9001]
 Time: 1441849245020

```

FreePastry 还有很多内容，我们无法在这里详细说明。然而，这些示例提供了 P2P 应用程序开发性质的感觉。其他 P2P 框架以类似的方式工作。

# 总结

在本章中，我们探讨了 P2P 网络的性质和用途。这种架构将所有节点视为平等，避免使用中央服务器。节点使用覆盖网络进行映射，有效地在 IP 地址空间中创建了一个节点的子网络。这些节点的能力各不相同，会以随机的方式加入和离开网络。

我们看到了分布式哈希表如何支持在网络中识别和定位节点。路由算法使用这个表来通过节点之间的消息传递来满足请求。我们演示了 Java 分布式哈希表来说明 DHT 的使用。

有几个基于 Java 的开源 P2P 框架可用。我们使用 FreePastry 来演示 P2P 网络的工作原理。具体来说，我们向您展示了节点如何加入网络以及消息如何在节点之间发送。这提供了对这些框架如何运作的更好理解。

在下一章中，我们将探讨 UDP 协议的性质以及它如何支持多播。
