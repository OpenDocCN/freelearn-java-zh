# 第二章。网络寻址

要使程序与另一个程序通信，它必须有一个地址。在本章中，将研究地址的使用，包括 Internet 地址。我们将在本章的第一部分介绍许多基本概念。这包括网络的架构和用于节点间通信的协议。

我们将讨论几个主题，包括：

+   **网络基础知识**：这是介绍基本概念和术语的地方

+   **使用 NetworkInterface 类**：这提供了对系统设备的访问

+   **URL/UII/URN**：我们将讨论这些术语之间的关系

+   **Inet4Address 和 Inet6Address 类**：我们将讨论它们的使用

+   **网络属性**：我们将考虑在 Java 中可配置的属性

这将为您提供更深入地学习网络的基础。

# 网络基础知识

网络是一个广泛而复杂的主题。特别是像寻址这样的子主题是非常复杂的。我们将介绍从 Java 角度常见和有用的术语和概念。

大部分讨论将集中在 Java 对互联网的支持上。**统一资源定位符**（**URL**）被大多数互联网用户所认可。然而，**统一资源标识符**（**URI**）和**统一资源名称**（**URN**）这些术语并不像 URL 那样被认可或理解。我们将区分这些术语并研究 Java 支持的类。

浏览器用户通常会输入他们想访问的网站的 URL。这个 URL 需要映射到一个 IP 地址。IP 地址是标识该网站的唯一数字。使用**域名系统**（**DNS**）服务器将 URL 映射到 IP 地址。这样用户就不必记住每个网站的数字。Java 使用`InetAddress`类来访问 IP 地址和资源。

UDP 和 TCP 被许多应用程序使用。IP 支持这两种协议。IP 协议在网络节点之间传输信息包。Java 支持 IPv4 和 IPv6 协议版本。

UDP 和 TCP 都是在 IP 之上的分层。还有一些其他协议是在 TCP 之上的，比如 HTTP。这些关系在下图中显示：

![网络基础知识](img/B04915_02_01.jpg)

当不同网络上的不同计算机和操作系统之间进行通信时，可能会出现硬件或软件层面的差异导致的问题。其中一个问题是 URL 中使用的字符。`URLEncoder`和`URLDecoder`类可以帮助解决这个问题，并且它们在第九章 *网络互操作性*中进行了讨论。

分配给设备的 IP 地址可以是**静态**或**动态**的。如果是静态的，每次设备重启时它都不会改变。对于动态地址，每次设备重启或网络连接重置时，地址可能会改变。

静态地址通常由管理员手动分配。动态地址通常使用运行在 DHCP 服务器上的**动态主机配置协议**（**DHCP**）进行分配。对于 IPv6，由于 IPv6 地址空间较大，DHCP 并不那么有用。但是，DHCP 对于支持生成随机地址等任务是有用的，这在从网络外部查看时可以增加网络内部的隐私。

**互联网编号分配机构**（**IANA**）负责分配 IP 地址空间。五个**区域互联网注册机构**（**RIRs**）向本地互联网实体分配 IP 地址块，这些实体通常被称为**互联网服务提供商**（**ISP**）。

有几本出版物详细介绍了 IP 协议：

+   RFC 790—分配的数字：这个规范涉及网络数字的格式。例如，IPv4 的 A、B 和 C 类在这个规范中被定义（[`tools.ietf.org/html/rfc790`](https://tools.ietf.org/html/rfc790)）。

+   RFC 1918—私人互联网地址分配：这个规范涉及私人地址的分配方式。这允许将多个私人地址关联到一个公共地址上（[`tools.ietf.org/html/rfc1918`](https://tools.ietf.org/html/rfc1918)）。

+   RFC 2365—管理范围的 IP 多播：这个规范定义了多播地址空间以及它的实现方式。它定义了 IPv4 和 IPv6 多播地址空间之间的映射关系（[`tools.ietf.org/html/rfc2365`](https://tools.ietf.org/html/rfc2365)）。

+   RFC 2373—IPv6 寻址架构：这个规范研究了 IPv6 协议、它的格式以及 IPv6 支持的各种地址类型（[`www.ietf.org/rfc/rfc2373.txt`](http://www.ietf.org/rfc/rfc2373.txt)）。

在可能的情况下，我们将用 Java 代码来说明这里介绍的许多概念。所以让我们从理解网络开始。

## 理解网络基础知识

网络由节点和链接组成，它们组合在一起创建网络架构。连接到互联网的设备称为节点。计算机节点称为主机。节点之间的通信是通过这些链接使用协议进行的，比如 HTTP 或 UDP。

链接可以是有线的，比如同轴电缆、双绞线和光纤，也可以是无线的，比如微波、蜂窝、Wi-Fi 或卫星通信。这些不同的链接支持不同的带宽和吞吐量，以满足特定的通信需求。

节点包括设备，如网络接口控制器（NIC）、桥接器、交换机、集线器和路由器。它们都涉及在计算机之间传输各种形式的数据。

NIC 有一个 IP 地址，并且是计算机的一部分。桥接器连接两个网络段，允许将一个较大的网络分解成较小的网络。中继器和集线器主要用于重新传输信号并增强其强度。

集线器、交换机和路由器在某种程度上相似，但在复杂性上有所不同。集线器处理多个端口，并简单地将数据转发到所有连接的端口。交换机将根据流量学习数据的发送位置。路由器可以被编程来操作和路由消息。在许多网络中，路由器更有用，大多数家庭网络使用路由器。

当一条消息从家用计算机发送到互联网时，会发生几件事。计算机的地址不是全局唯一的。这要求任何发送到计算机的消息都必须由一个**网络地址转换**（NAT）设备处理，将地址更改为可以在互联网上使用的地址。它允许一个 IP 地址用于网络上的多个设备，比如家庭局域网。

计算机还可以使用代理服务器，它充当其他网络的网关。Java 提供了对代理的支持，使用`Proxy`和`ProxySelector`类。我们将在第九章中研究它们的使用，*网络互操作性*。

消息通常会通过防火墙路由。防火墙保护计算机免受恶意意图。

### 网络架构和协议

常见的网络架构包括总线、星型和树型网络。这些物理网络通常用于支持覆盖网络，即虚拟网络。这样的网络将底层网络抽象出来，以创建支持应用程序的网络架构，比如点对点应用程序。

当两台计算机进行通信时，它们使用协议。网络的各个层面上使用了许多不同的协议，我们将主要关注 HTTP、TCP、UDP 和 IP。

有几种模型描述了如何分层支持不同的任务和协议。一个常见的模型是**开放系统互连**（**OSI**）模型，它定义了七层。网络模型的每一层都可以支持一个或多个协议。各种协议的关系在下表中描述：

| 层 | 示例协议 | 目的 |
| --- | --- | --- |
| 应用 | HTTP，FTP，SNMP | 支持专门操作的高级协议 |
| 表示 | 传输层安全 | 支持应用层数据的传递和处理 |
| 会话 | 网络文件系统 | 管理会话 |
| 传输 | TCP，UDP | 管理数据包 |
| 网络 | IP | 传输数据包 |
| 数据链路 | 以太网，帧中继 | 在网络段之间传输数据 |
| 物理 | DSL，蓝牙 | 处理原始数据 |

OSI 层的更完整的协议列表可以在[`en.wikipedia.org/wiki/List_of_network_protocols_(OSI_model)`](https://en.wikipedia.org/wiki/List_of_network_protocols_(OSI_model))找到。我们无法涵盖所有这些协议，将重点放在 Java SDK 支持的更重要的协议上。

考虑从服务器向客户端传输网页。当数据发送到客户端时，数据将被封装在 HTTP 消息中，进一步封装在 TCP，IP 和链路级协议消息中，每个消息通常包含标题和页脚。这些封装的标题集被发送到互联网上的目标客户端，其中数据被提取以解析每个封装的标题，直到原始 HTML 文件被显示。

幸运的是，我们不需要熟悉这个过程的细节。许多类隐藏了这是如何发生的，使我们能够专注于数据。

我们感兴趣的传输层协议是 TCP 和 UDP。TCP 提供比 UDP 更可靠的通信协议。但是，UDP 更适合在传递不需要稳健性的短消息时使用。流数据通常使用 UDP。

UDP 和 TCP 之间的区别在下表中概述：

| 特征 | TCP | UDP |
| --- | --- | --- |
| 连接 | 面向连接 | 无连接 |
| 可靠性 | 更高 | 更低 |
| 数据包的顺序 | 恢复顺序 | 可能丢失顺序 |
| 数据边界 | 数据包被合并 | 数据包是独立的 |
| 传输时间 | 比 UDP 慢 | 比 TCP 快 |
| 错误检查 | 是 | 是，但没有恢复选项 |
| 确认 | 是 | 否 |
| 权重 | 需要更多支持的重量级 | 需要更少支持的轻量级 |

TCP 用于许多协议，如 HTTP，**简单邮件传输** **协议**（**SMTP**）和**文件传输协议**（**FTP**）。UDP 被 DNS 用于流媒体，如电影，以及**语音传输** **IP**（**VOIP**）。

# 使用`NetworkInterface`类

`NetworkInterface`类提供了访问充当网络节点的设备的方法。该类还提供了获取低级设备地址的方法。许多系统同时连接到多个网络。这些可能是有线的，比如网络卡，也可能是无线的，比如用于无线局域网或蓝牙连接。

`NetworkInterface`类表示一个 IP 地址，并提供有关此 IP 地址的信息。**网络接口**是计算机与网络之间的连接点。这通常使用某种类型的 NIC。它不一定要有物理表现，但可以在软件中执行，就像使用环回连接（IPv4 的`127.0.0.1`和 IPv6 的`::1`）一样。

`NetworkInterface`类没有任何公共构造函数。提供了三个静态方法来返回`NetworkInterface`类的实例：

+   `getByInetAddress`：如果知道 IP 地址，则使用此选项

+   `getByName`：如果知道接口名称，则使用此选项

+   `getNetworkInterfaces`：这提供了可用接口的枚举

以下代码演示了如何使用`getNetworkInterfaces`方法来获取并显示当前计算机的网络接口的枚举：

```java
    try {
        Enumeration<NetworkInterface> interfaceEnum = 
            NetworkInterface.getNetworkInterfaces();
        System.out.printf("Name      Display name\n");
        for(NetworkInterface element : 
                Collections.list(interfaceEnum)) {
            System.out.printf("%-8s  %-32s\n",
                    element.getName(), element.getDisplayName());
    } catch (SocketException ex) {
        // Handle exceptions
    }
```

一个可能的输出如下，但已经被截断以节省空间：

**名称显示名称**

**lo 软件环回接口 1**

**eth0 Microsoft 内核调试网络适配器**

**eth1 Realtek PCIe FE Family Controller**

**wlan0 Realtek RTL8188EE 802.11 b/g/n 无线适配器**

**wlan1 Microsoft Wi-Fi 直接虚拟适配器**

**net0 Microsoft 6to4 适配器**

**net1 Teredo 隧道伪接口**

**...**

如果存在子接口，`getSubInterfaces`方法将返回一个子接口的枚举，如下所示。当单个物理网络接口被划分为逻辑接口以进行路由目的时，就会出现子接口：

```java
    Enumeration<NetworkInterface> interfaceEnumeration = 
        element.getSubInterfaces();
```

每个网络接口将具有一个或多个与其关联的 IP 地址。`getInetAddresses`方法将返回这些地址的`Enumeration`。如下所示，初始的网络接口列表已经被扩充以显示与它们关联的 IP 地址：

```java
    Enumeration<NetworkInterface> interfaceEnum = 
        NetworkInterface.getNetworkInterfaces();
    System.out.printf("Name      Display name\n");
    for (NetworkInterface element : 
            Collections.list(interfaceEnum)) {
        System.out.printf("%-8s  %-32s\n",
                element.getName(), element.getDisplayName());
        Enumeration<InetAddress> addresses = 
            element.getInetAddresses();
        for (InetAddress inetAddress : 
                Collections.list(addresses)) {
            System.out.printf("    InetAddress: %s\n", 
                inetAddress);
        }
```

一个可能的输出如下：

**名称显示名称**

**lo 软件环回接口 1**

**InetAddress: /127.0.0.1**

**InetAddress: /0:0:0:0:0:0:0:1**

**eth0 Microsoft 内核调试网络适配器**

**eth1 Realtek PCIe FE Family Controller**

**InetAddress: /fe80:0:0:0:91d0:8e19:31f1:cb2d%eth1**

**wlan0 Realtek RTL8188EE 802.11 b/g/n 无线适配器**

**InetAddress: /192.168.1.5**

**InetAddress: /2002:6028:2252:0:0:0:0:1000**

**InetAddress: /fe80:0:0:0:9cdb:371f:d3e9:4e2e%wlan0**

**wlan1 Microsoft Wi-Fi 直接虚拟适配器**

**InetAddress: /fe80:0:0:0:f8f6:9c75:d86d:8a22%wlan1**

**net0 Microsoft 6to4 适配器**

**net1 Teredo 隧道伪接口**

**InetAddress: /2001:0:9d38:6abd:6a:37:3f57:fefa**

**...**

我们还可以使用以下 Java 8 技术。使用流和 lambda 表达式来显示 IP 地址以生成相同的输出：

```java
        addresses = element.getInetAddresses();
        Collections
                .list(addresses)
                .stream()
                .forEach((inetAddress) -> {
                    System.out.printf("    InetAddress: %s\n", 
                        inetAddress);
                });
```

有许多`InetworkAddress`方法，可以显示有关网络连接的更多细节。我们在遇到它们时将进行讨论。

## 获取 MAC 地址

**媒体访问控制**（**MAC**）地址用于标识 NIC。MAC 地址通常由 NIC 的制造商分配，并且是其硬件的一部分。节点上的每个 NIC 必须具有唯一的 MAC 地址。理论上，无论其位置如何，所有 NIC 都将具有唯一的 MAC 地址。MAC 地址由 48 位组成，通常以六对十六进制数字的形式编写。这些组之间用破折号或冒号分隔。

### 获取特定的 MAC 地址

通常，普通的 Java 程序员不需要 MAC 地址。但是，它们可以在需要时检索。以下方法返回一个包含`NetworkInterface`实例的 IP 地址及其 MAC 地址的字符串。`getHardwareAddress`方法返回一个包含数字的字节数组。然后将此数组显示为 MAC 地址。大部分代码段逻辑用于格式化输出，其中三元运算符确定是否应显示破折号：

```java
    public String getMACIdentifier(NetworkInterface network) {
        StringBuilder identifier = new StringBuilder();
        try {
            byte[] macBuffer = network.getHardwareAddress();
            if (macBuffer != null) {
                for (int i = 0; i < macBuffer.length; i++) {
                       identifier.append(
                       String.format("%02X%s",macBuffer[i], 
                       (i < macBuffer.length - 1) ? "-" : ""));
                }
            } else {
                return "---";
            }
        } catch (SocketException ex) {
            ex.printStackTrace();
        }
        return identifier.toString();
    }
```

该方法在以下示例中进行了演示，我们使用本地主机：

```java
    InetAddress address = InetAddress.getLocalHost();
    System.out.println("IP address: " + address.getHostAddress());
    NetworkInterface network = 
        NetworkInterface.getByInetAddress(address);
    System.out.println("MAC address: " + 
        getMACIdentifier(network));
```

输出将根据所使用的计算机而有所不同。一个可能的输出如下：

**IP 地址：192.168.1.5**

**MAC 地址：EC-0E-C4-37-BB-72**

### 注意

`getHardwareAddress`方法只允许您访问本地主机的 MAC 地址。您不能使用它来访问远程 MAC 地址。

### 获取多个 MAC 地址

并非所有网络接口都将具有 MAC 地址。这在这里得到了证明，使用`getNetworkInterfaces`方法创建一个枚举，然后显示每个网络接口：

```java
    Enumeration<NetworkInterface> interfaceEnum = 
        NetworkInterface.getNetworkInterfaces();
    System.out.println("Name    MAC Address");
    for (NetworkInterface element : 
            Collections.list(interfaceEnum)) {
        System.out.printf("%-6s  %s\n",
            element.getName(), getMACIdentifier(element));
```

一个可能的输出如下。输出已经被截断以节省空间：

**名称 MAC 地址**

**lo ---**

**eth0 ---**

**eth1 8C-DC-D4-86-B1-05**

**wlan0 EC-0E-C4-37-BB-72**

**wlan1 EC-0E-C4-37-BB-72**

**net0 ---**

**net1 00-00-00-00-00-00-00-E0**

**net2 00-00-00-00-00-00-00-E0**

**...**

或者，我们可以使用以下的 Java 实现。它将枚举转换为流，然后处理流中的每个元素：

```java
    interfaceEnum = NetworkInterface.getNetworkInterfaces();
    Collections
            .list(interfaceEnum)
            .stream()
            .forEach((inetAddress) -> {
                System.out.printf("%-6s  %s\n",
                    inetAddress.getName(), 
                    getMACIdentifier(inetAddress));
            });
```

流的强大之处在于当我们需要执行额外的处理时，比如过滤掉某些接口，或者将接口转换为不同的数据类型时。

# 网络寻址概念

有不同类型的网络地址。地址用于标识网络中的节点。例如，**Internetwork Packet Exchange**（**IPX**）协议是早期用于访问网络节点的协议。X.25 是用于**广域网**（**WAN**）分组交换的协议套件。MAC 地址为物理网络级别的网络接口提供了唯一标识符。然而，我们的主要兴趣是 IP 地址。

## URL/URI/URN

这些术语用于引用互联网资源的名称和位置。URI 标识资源的名称，如网站或互联网上的文件。它可能包含资源的名称和位置。

URL 指定资源的位置以及如何检索它。协议构成 URL 的第一部分，并指定如何检索数据。URL 始终包含协议，如 HTTP 或 FTP。例如，以下两个 URL 使用不同的协议。第一个使用 HTTPS 协议，第二个使用 FTP 协议：

**https://www.packtpub.com/**

**ftp://speedtest.tele2.net/**

Java 提供了支持 URI 和 URL 的类。这些类的讨论将在下一节开始。在这里，我们将更深入地讨论 URN。

URN 标识资源但不标识其位置。URN 类似于城市的名称，而 URL 类似于城市的纬度和经度。当资源（如网页或文件）被移动时，资源的 URL 不再正确。URL 需要在使用的任何地方进行更新。URN 指定了资源的名称，但没有指定其位置。提供 URN 的其他实体将返回其位置。URN 的使用并不那么广泛。

URN 的语法如下所示。`<NID>`元素是命名空间标识符，`<NSS>`是命名空间特定字符串：

**<URN> ::= "urn:" <NID> ":" <NSS>**

例如，以下是一个 URN，作为 SOAP 消息的一部分来指定其命名空间：

```java
<?xml version='1.0'?>
<SOAP:Envelope
  xmlns:SOAP='urn:schemas-xmlsoap-org:soap.v1'>
 <SOAP:Body>
  ...
    xmlns:i='urn:gargantuan-com:IShop'>
   ...
 </SOAP:Body>
</SOAP:Envelope>
```

它也用于其他地方，比如使用它们的 ISBN 来识别书籍。在浏览器中输入以下 URL 将会引用一本 EJB 书籍：

**https://books.google.com/books?isbn=9781849682381**

![URL/URI/URN](img/B04915_02_02.jpg)

URN 的语法取决于命名空间。IANA 负责分配许多互联网资源，包括 URN 命名空间。URN 仍然是一个活跃的研究领域。URL 和 URN 都是 URI。

## 使用 URI 类

URI 的一般语法由方案和特定于方案的部分组成：

**[scheme:] scheme-specific-part**

有许多与 URI 一起使用的方案，包括：

+   **文件**：用于文件系统

+   **FTP**：这是文件传输协议

+   **HTTP**：这通常用于网站

+   **mailto**：这是作为邮件服务的一部分使用的

+   **urn**：用于通过名称标识资源

scheme-specific-part 因使用的方案而异。URI 可以被分类为绝对或相对，或者是不透明或分层的。这些区别对我们来说并不是立即感兴趣，尽管 Java 提供了方法来确定 URI 是否属于这些类别之一。

### 创建 URI 实例

可以使用几种构造函数变体为不同的方案创建 URI。创建 URI 的最简单方法是使用指定 URI 的字符串参数，如下所示：

```java
    URI uri = new 
        URI("https://www.packtpub.com/books/content/support");
```

下一个 URI 使用一个片段来访问维基百科文章的一个子部分，该文章涉及 URL 的规范化：

```java
    uri = new URI("https://en.wikipedia.org/wiki/"
        + "URL_normalization#Normalization_process");
```

我们还可以使用构造函数的以下版本来指定 URI 的方案、主机、路径和片段：

```java
    uri = new 
        URI("https","en.wikipedia.org","/wiki/URL_normalization",
        "Normalization_process");
```

这两个后者的 URI 是相同的。

### 拆分 URI

Java 使用`URI`类来表示 URI，并且具有几种方法来提取 URI 的部分。更有用的方法列在下表中：

| 方法 | 目的 |
| --- | --- |
| `getAuthority` | 这是负责解析 URI 的实体 |
| `getScheme` | 使用的方案 |
| `getSchemeSpecificPart` | URI 的方案特定部分 |
| `getHost` | 主机 |
| `getPath` | URI 路径 |
| `getQuery` | 查询，如果有的话 |
| `getFragment` | 如果使用，这是正在访问的子元素 |
| `getUserInfo` | 用户信息，如果可用 |
| `normalize` | 从路径中删除不必要的“.”和“..” |

还有一些“原始”方法，如`getRawPath`或`getRawFragment`，分别返回路径或片段的版本。这包括特殊字符，如问号，或以星号开头的字符序列。有几个字符类别定义了这些字符及其用法，如[`docs.oracle.com/javase/8/docs/api/java/net/URI.html`](http://docs.oracle.com/javase/8/docs/api/java/net/URI.html)中所述。

我们开发了以下辅助方法，用于显示 URI 特征：

```java
    private static void displayURI(URI uri) {
        System.out.println("getAuthority: " + uri.getAuthority());
        System.out.println("getScheme: " + uri.getScheme());
        System.out.println("getSchemeSpecificPart: " 
            + uri.getSchemeSpecificPart());
        System.out.println("getHost: " + uri.getHost());
        System.out.println("getPath: " + uri.getPath());
        System.out.println("getQuery: " + uri.getQuery());
        System.out.println("getFragment: " + uri.getFragment());
        System.out.println("getUserInfo: " + uri.getUserInfo());
        System.out.println("normalize: " + uri.normalize());
    }
```

下一个代码序列为 Packtpub 网站创建了一个`URI`实例，然后调用了`displayURI`方法：

```java
    try {
        URI uri = new 
            URI("https://www.packtpub.com/books/content/support");
        displayURI(uri);
    } catch (URISyntaxException ex) {
        // Handle exceptions
    }
```

这个序列的输出如下：

**getAuthority: www.packtpub.com**

**getScheme: https**

**getSchemeSpecificPart: //www.packtpub.com/books/content/support**

**getHost: www.packtpub.com**

**getPath: /books/content/support**

**getQuery: null**

**getFragment: null**

**getUserInfo: null**

**normalize: https://www.packtpub.com/books/content/support**

**http://www.packtpub.com**

更常见的是，这些方法用于提取相关信息以进行额外处理。

## 使用 URL 类

连接到站点并检索数据的最简单方法之一是通过`URL`类。您只需要提供站点的 URL 和协议的详细信息。`InetAddress`类的实例将保存 IP 地址和可能的地址的主机名。

`URLConnection`类是在第一章中引入的，*网络编程入门*。它也可以用于提供对 URL 表示的互联网资源的访问。我们将在第四章中讨论这个类及其用法，*客户端/服务器开发*。

### 创建 URL 实例

有几种创建 URL 实例的方法。最简单的方法是简单地将站点的 URL 作为类构造函数的参数提供。这在这里得到了说明，创建了 Packtpub 网站的`URL`实例：

```java
    URL url = new URL("http://www.packtpub.com");
```

URL 需要指定一个协议。例如，尝试创建 URL 将导致**java.net.MalformedURLException: no protocol: www.packtpub.com**错误消息：

```java
    url = new URL("www.packtpub.com"); 
```

有几种构造函数变体。以下两种变体将创建相同的 URL。第二种使用协议、主机、端口号和文件的参数：

```java
    url = new URL("http://pluto.jhuapl.edu/");
    url = new URL("http", "pluto.jhuapl.edu", 80, 
        "News-Center/index.php");
```

### 拆分 URL

了解 URL 的更多信息可能很有用。如果用户输入了我们需要处理的 URL，我们甚至可能不知道正在使用的 URL 是什么。有几种方法支持将 URL 拆分为其组件，如下表所总结的：

| 方法 | 目的 |
| --- | --- |
| `getProtocol` | 这是协议的名称。 |
| `getHost` | 这是主机名。 |
| `getPort` | 这是端口号。 |
| `getDefaultPort` | 这是协议的默认端口号。 |
| `getFile` | 这返回`getPath`连接到`getQuery`的结果。 |
| `getPath` | 如果有的话，这将检索 URL 的路径。 |
| `getRef` | 这是 URL 引用的返回名称。 |
| `getQuery` | 如果存在，这将返回 URL 的查询部分。 |
| `getUserInfo` | 这将返回与 URL 关联的任何用户信息。 |
| `getAuthority` | 权威通常由服务器主机名或 IP 地址组成。它可能包括端口号。 |

我们将使用以下方法来说明上表中的方法：

```java
    private static void displayURL(URL url) {
        System.out.println("URL: " + url);
        System.out.printf("  Protocol: %-32s  Host: %-32s\n",
            url.getProtocol(),url.getHost());
        System.out.printf("      Port: %-32d  Path: %-32s\n", 
            url.getPort(),url.getPath());
        System.out.printf(" Reference: %-32s  File: %-32s\n",
            url.getRef(),url.getFile());
        System.out.printf(" Authority: %-32s Query: %-32s\n", 
            url.getAuthority(),url.getQuery());
        System.out.println(" User Info: " + url.getUserInfo());
    }
```

以下输出演示了将多个 URL 用作此方法的参数时的输出。

```java
URL: http://www.packpub.com
  Protocol: http                              Host: www.packpub.com                 
      Port: -1                                Path:                                 
 Reference: null                              File:                                 
 Authority: www.packpub.com                  Query: null                            
 User Info: null

URL: http://pluto.jhuapl.edu/
  Protocol: http                              Host: pluto.jhuapl.edu                
      Port: -1                                Path: /                               
 Reference: null                              File: /                               
 Authority: pluto.jhuapl.edu                 Query: null                            
 User Info: null

URL: http://pluto.jhuapl.edu:80News-Center/index.php
  Protocol: http                              Host: pluto.jhuapl.edu                
      Port: 80                                Path: News-Center/index.php           
 Reference: null                              File: News-Center/index.php           
 Authority: pluto.jhuapl.edu:80              Query: null                            
 User Info: null

URL: https://en.wikipedia.org/wiki/Uniform_resource_locator#Syntax
  Protocol: https                             Host: en.wikipedia.org                
      Port: -1                                Path: /wiki/Uniform_resource_locator  
 Reference: Syntax                            File: /wiki/Uniform_resource_locator  
 Authority: en.wikipedia.org                 Query: null                            
 User Info: null

URL: https://www.google.com/webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8#q=url+syntax
  Protocol: https                             Host: www.google.com                  
      Port: -1                                Path: /webhp                          
 Reference: q=url+syntax                      File: /webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8
 Authority: www.google.com                   Query: sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8
 User Info: null

URL: https://www.packtpub.com/books/content/support
  Protocol: https                             Host: www.packtpub.com                
      Port: -1                                Path: /books/content/support          
 Reference: null                              File: /books/content/support          
 Authority: www.packtpub.com                 Query: null                            
 User Info: null

```

URL 类还支持打开连接和 IO 流。我们在第一章中演示了`openConnection`方法，*网络编程入门*。`getContent`方法返回 URL 引用的数据。例如，以下是针对 Packtpub URL 应用该方法的示例：

```java
    url = new URL("http://www.packtpub.com");
    System.out.println("getContent: " + url.getContent());
```

输出如下：

**sun.net.www.protocol.http.HttpURLConnection$HttpInputStream@5c647e05**

这表明我们需要使用输入流来处理资源。数据类型取决于 URL。这个主题是通过`URLConnection`类来探讨的，该类在第四章中讨论，*客户端/服务器开发*。

## IP 地址和 InetAddress 类

IP 地址是用于标识节点（如计算机、打印机、扫描仪或类似设备）的数值。它用于网络接口寻址和位置寻址。在其上下文中是唯一的地址，用于标识设备。同时它构成了网络中的位置。名称指定一个实体，比如[www.packtpub.com](http://www.packtpub.com)。它的地址`83.166.169.231`告诉我们它的位置。如果我们想要从一个站点发送或接收消息，消息通常会通过一个或多个节点路由。

### 获取有关地址的信息

`InetAddress`类表示 IP 地址。IP 协议是 UDP 和 TCP 协议使用的低级协议。IP 地址是分配给设备的 32 位或 128 位无符号数。

IP 地址有着悠久的历史，并使用两个主要版本：IPv4 和 IPv6。数字 5 被分配给**Internet Stream Protocol**。这是一个实验性协议，但实际上从未被称为 IPv5 版本，也不打算用于一般用途。

`InetAddress`类的`getAllByName`方法将返回给定 URL 的 IP 地址。在以下示例中，显示了与[www.google.com](http://www.google.com)相关联的地址：

```java
    InetAddress names[] = 
        InetAddress.getAllByName("www.google.com");
    for(InetAddress element : names) {
        System.out.println(element);
    }
```

一个可能的输出如下。输出将根据位置和时间而变化，因为许多网站分配给它们多个 IP 地址。在这种情况下，它同时使用 IPv4 和 IPv6 地址：

**www.google.com/74.125.21.105**

**www.google.com/74.125.21.103**

**www.google.com/74.125.21.147**

**www.google.com/74.125.21.104**

**www.google.com/74.125.21.99**

**www.google.com/74.125.21.106**

**www.google.com/2607:f8b0:4002:c06:0:0:0:69**

`InetAddress`类拥有多个方法来提供对 IP 地址的访问。我们将在相关时候介绍它们。我们从返回其规范主机名、主机名和主机地址的方法开始。它们在以下辅助方法中使用：

```java
    private static void displayInetAddressInformation(
            InetAddress address) {
        System.out.println(address);
        System.out.println("CanonicalHostName: " +
            address.getCanonicalHostName());
        System.out.println("HostName: " + address.getHostName());
        System.out.println("HostAddress: " + 
            address.getHostAddress());
    }
```

规范主机名是**完全限定域名**（**FQDN**）。正如该术语所暗示的那样，它是主机的完整名称，包括顶级域。这些方法返回的值取决于多种因素，包括 DNS 服务器。系统提供有关网络上实体的信息。

以下序列使用 Packtpub 网站的显示方法：

```java
    InetAddress address = 
        InetAddress.getByName("www.packtpub.com");
    displayInetAddressInformation(address);
```

您将获得类似以下内容的输出：

**www.packtpub.com/83.166.169.231**

**CanonicalHostName: 83.166.169.231**

**HostAddress: 83.166.169.231**

**HostName: www.packtpub.com**

`InetAddress`类的`toString`方法返回主机名，后跟斜杠，然后是主机地址。在这种情况下，`getCanonicalHostName`方法返回主机地址，这不是 FQDN。该方法将尽力返回名称，但根据机器的配置可能无法做到。

### 地址范围问题

IP 地址的范围指的是 IP 地址的唯一性。在本地网络中，比如许多家庭和办公室使用的网络中，该地址可能是本地的。有三种范围类型：

+   **链路本地**：这在单个本地子网中使用，不连接到互联网。没有路由器存在。当计算机没有静态 IP 地址并且找不到 DHCP 服务器时，链路本地地址的分配是自动完成的。

+   **站点本地**：当地址不需要全局前缀并且在站点内是唯一的时使用。它无法直接从互联网访问，并且需要诸如 NAT 之类的映射服务。

+   **全局**：顾名思义，该地址在整个互联网中是唯一的。

还有一些私有地址，这些在*IPv4 中的私有地址*和*IPv6 中的私有地址*部分有讨论。`InetAddress`类支持几种方法来识别正在使用的地址类型。大多数这些方法都是不言自明的，如下表所示，其中 MC 是多播的缩写：

| 方法 | 范围 | 描述 |
| --- | --- | --- |
| `isAnyLocalAddress` | 任意 | 这是与任何本地地址匹配的地址。它是一个通配地址。 |
| `isLoopbackAddress` | 回环 | 这是一个回环地址。对于 IPv4，它是`127.0.0.1`，对于 IPv6，它是`0:0:0:0:0:0:0:1`。 |
| `isLinkLocalAddress` | 链路本地 | 这是一个链路本地地址。 |
| `isSiteLocalAddress` | 站点本地 | 这是站点本地的。它们可以被不同网络上的其他节点访问，但在同一站点内。 |
| `isMulticastAddress` | MC | 这是一个多播地址。 |
| `isMCLinkLocal` | MC 链路本地 | 这是一个链路本地的多播地址。 |
| `isMCNodeLocal` | MC 节点本地 | 这是一个节点本地的多播地址。 |
| `isMCSiteLocal` | MC 站点本地 | 这是一个站点本地的多播地址。 |
| `isMCOrgLocal` | MC 组织本地 | 这是一个组织本地的多播地址。 |
| `isMCGlobal` | MC 全局 | 这是一个全局多播地址。 |

所使用的地址类型和范围在以下表格中总结了 IPv4 和 IPv6：

| 地址类型 | IPv4 | IPv6 |
| --- | --- | --- |
| 多播 | `224.0.0.0` 到 `239.255.255.25` | 以字节`FF`开头 |
| MC 全局 | `224.0.1.0` 到 `238.255.255.255` | `FF0E` 或 `FF1E` |
| Org MC | `239.192.0.0/14` | `FF08` 或 `FF18` |
| MC 站点本地 | N/A | `FF05` 或 `FF15` |
| MC 链路本地 | `224.0.0.0` | `FF02` 或 `FF12` |
| MC 节点本地 | `127.0.0.0` | `FF01` 或 `FF11` |
| 私有 | `10.0.0.0` 到 `10.255.255.255``172.16.0.0` 到 `172.31.255.255``192.168.0.0` 到 `192.168.255.255` | `fd00::/8` |

### 测试可达性

`InetAddress`类的`isReachable`方法将尝试确定是否可以找到地址。如果可以，该方法将返回`true`。以下示例演示了这种方法。`getAllByName`方法返回一个可用于 URL 的`InetAddress`实例数组。`isReachable`方法使用整数参数指定在决定地址不可访问之前最长等待时间（以毫秒为单位）：

```java
    String URLAddress = "www. packtpub.com";
    InetAddress[] addresses = 
        InetAddress.getAllByName(URLAddress);
    for (InetAddress inetAddress : addresses) {
        try {
            if (inetAddress.isReachable(10000)) {
                System.out.println(inetAddress + " is reachable");
            } else {
                System.out.println(inetAddress + 
                    " is not reachable");
            }
        } catch (IOException ex) {
            // Handle exceptions
        }
    }
```

URL [www.packtpub.com](http://www.packtpub.com) 是可访问的，如下所示：

**www.packtpub.com/83.166.169.231 可访问**

然而，[www.google.com](http://www.google.com)不是：

**www.google.com/173.194.121.52 无法访问**

**www.google.com/173.194.121.51 无法访问**

**www.google.com/2607:f8b0:4004:809:0:0:0:1014 无法访问**

您的结果可能会有所不同。`isReachable`方法将尽最大努力确定地址是否可达。但是，它的成功取决于更多的因素，而不仅仅是地址是否存在。失败的原因可能包括：服务器可能已关闭，网络响应时间过长，或者防火墙可能正在阻止某个站点。操作系统和 JVM 设置也会影响该方法的运行效果。

这种方法的替代方法是使用`RunTime`类的`exec`方法来执行针对 URL 的`ping`命令。然而，这并不是可移植的，可能仍然受到影响`isReachable`方法的一些因素的影响。

## 介绍 Inet4Address

IPv4 地址由 32 位组成，允许最多 4,294,967,296（232）个地址。地址的人类可读形式由四个十进制数（8 位）组成，每个数的范围是 0 到 255。一些地址已被保留用于私人网络和多播地址。

在 IPv4 的早期使用中，第一个**八位组**代表网络号（也称为网络前缀或网络块），其余的位代表**剩余**字段（主机标识符）。后来，使用三个类来分配地址：A、B 和 C。这些系统已经大部分不再使用，并已被**无类域间路由**（**CIDR**）所取代。这种路由方法在位边界上分配地址，提供更大的灵活性。这种方案被称为无类，与早期的有类系统相对。在 IPv6 中，使用 64 位网络标识符。

### IPv4 中的私有地址

私人网络不一定需要全局访问互联网。这导致一系列地址被分配给这些私人网络。

| 范围 | 位数 | 地址数量 |
| --- | --- | --- |
| `10.0.0.0`到`10.255.255.255` | 24 位 | 16,777,216 |
| `172.16.0.0`到`172.31.255.255` | 20 位 | 1,048,576 |
| `192.168.0.0`到`192.168.255.255` | 16 位 | 65,536 |

您可能会注意到，最后一组地址是由家庭网络使用的。私人网络通常使用 NAT 与互联网进行接口。这种技术将本地 IP 地址映射到互联网上可访问的 IP 地址。最初是为了缓解 IPv4 地址短缺而引入的。

### IPv4 地址类型

IPv4 中支持的三种地址类型：

+   **单播**：这个地址用于标识网络中的单个节点。

+   **多播**：这个地址对应于一组网络接口。成员将加入一个组，消息将发送给组的所有成员。

+   **广播**：这将向子网上的所有网络接口发送消息。

`Inet4Address`类支持 IPv4 协议。我们将在接下来更深入地研究这个类。

### Inet4Address 类

`Inet4Address`类是从`InetAddress`类派生出来的。作为一个派生类，它并没有覆盖`InetAddress`类的许多方法。例如，要获取一个`InetAddress`实例，我们可以使用任一类的`getByName`方法，如下所示：

```java
    Inet4Address address;
    address = (Inet4Address) 
       InetAddress.getByName("www.google.com");
    address = (Inet4Address) 
        Inet4Address.getByName("www.google.com");
```

在任一情况下，地址都需要转换，因为基类方法在任一情况下都被使用。`Inet4Address`类没有添加任何超出`InetAddress`类的新方法。

### 特殊的 IPv4 地址

有几个特殊的 IPv4 地址，包括以下两个：

+   **0.0.0.0**：这被称为未指定的 IPv4 地址（通配符地址），通常在网络接口没有 IP 地址并且正在尝试使用 DHCP 获取 IP 地址时使用。

+   **127.0.0.1**：这被称为环回地址。它提供了一种方便的方式来给自己发送消息，通常用于测试目的。

如果地址是通配符地址，`isAnyLocalAddress`方法将返回`true`。该方法在这里进行了演示，它返回`true`：

```java
    address = (Inet4Address) Inet4Address.getByName("0.0.0.0");
    System.out.println(address.isAnyLocalAddress());
```

接下来显示`isLoopbackAddress`方法，并将返回`true`：

```java
    address = (Inet4Address) Inet4Address.getByName("127.0.0.1");
    System.out.println(address.isLoopbackAddress());
```

我们将经常使用这个来测试后续章节中的服务器。

除此之外，其他特殊地址还包括用于协议分配、IPv6 到 IPv4 中继和测试目的的地址。有关这些和其他特殊地址的更多详细信息，请参阅[`en.wikipedia.org/wiki/IPv4#Special-use_addresses`](https://en.wikipedia.org/wiki/IPv4#Special-use_addresses)。

## 介绍 Inet6Address 类

IPv6 地址使用 128 位（16 个八位字节）。这允许最多 2128 个地址。IPv6 地址写为一系列由冒号分隔的八个组，每个组有 4 个十六进制数字。数字大小写不敏感。例如，[www.google.com](http://www.google.com)的 IPv6 地址如下：

**2607:f8b0:4002:0c08:0000:0000:0000:0067**

IPv6 地址可以通过几种方式简化。组中的前导零可以被移除。前面的例子可以重写为：

**2607:f8b0:4002:c08:0:0:0:67**

连续的零组可以用`::`替换，如下所示：

**2607:f8b0:4002:c08::67**

IPv6 支持三种寻址类型：

+   **单播**：这指定一个单一的网络接口。

+   **Anycast**：这种类型的地址分配给一组接口。当数据包发送到该组时，只有组中的一个成员接收数据包，通常是最近的成员。

+   **Multicast**：这将数据包发送到组中的所有成员。

该协议不支持广播寻址。IPv6 比网络规模的增加要复杂得多。它包括几个改进，如更容易的管理、更有效的路由能力、简单的报头格式以及消除了对 NAT 的需求。

### IPv6 中的私有地址

IPv6 中有私有地址空间。最初，它使用具有 fec0::/10 前缀的站点本地地址。然而，由于其定义存在问题，这已被取消，并用具有地址块`fc00::/7`的**唯一本地**（**UL**）地址替换。

这些地址可以由任何人生成，无需协调。但它们不一定是全局唯一的。其他私人网络可以使用相同的地址。它们不能使用全局 DNS 服务器分配，并且只能在本地地址空间中进行路由。

### Inet6Address 类

一般来说，除非您正在开发一个仅支持 IPv6 的应用程序，否则不需要使用`Inet6Address`类。大多数网络操作都是透明处理的。`Inet6Address`类是从`InetAddress`类派生的。`Inet6Address`类的`getByName`方法使用其基类`InetAddrress`类的`getAllByName`方法，返回它找到的第一个地址，如下所示。这可能不是一个 IPv6 地址：

```java
    public static InetAddress getByName(String host)
        throws UnknownHostException {
        return InetAddress.getAllByName(host)[0];
    }
```

### 注意

为了使这些示例中的一些正常工作，您的路由器可能需要配置为支持 IPv6 互联网连接。

`Inet6Address`类仅添加了一个方法，超出了`InetAddress`类的方法。这就是`isIPv4CompatibleAddress`方法，它在*使用 IPv4 兼容 IPv6 地址*部分中讨论。

### 特殊的 IPv6 地址

有一个由 64 个网络前缀组成的地址块：`2001:0000::/29`到`2001:01f8::/29`。这些用于特殊需求。其中三个已被 IANA 分配：

+   `2001::/32`：这是 teredo 隧道技术，是从 IPv4 过渡的一种技术

+   `2001:2::/48`：这用于基准测试

+   `2001:20::/28`：这用于加密哈希标识符

大多数开发人员不需要处理这些地址。

## 测试 IP 地址类型

通常，我们不关心 IP 地址是 IPv4 还是 IPv6。这两者之间的差异隐藏在各种协议级别之下。当您确实需要知道区别时，您可以使用这两种方法之一。`getAddress`方法返回一个字节数组。您可以检查字节数组的大小来确定它是 IPv4 还是 IPv6。或者您可以使用`instanceOf`方法。这两种方法如下所示：

```java
    byte buffer[] = address.getAddress();
    if(buffer.length <= 4) {
        System.out.println("IPv4 Address");
    } else {
        System.out.println("IPv6 Address");
    }
    if(address instanceof Inet4Address) {
        System.out.println("IPv4 Address");
    } else {
        System.out.println("IPv6 Address");
    }
```

### 使用 IPv4 兼容 IPv6 地址

点分十进制表示法是一种使用 IPv6 表示 IPv4 地址的方法。`::ffff:`前缀放在 IPv4 地址或其十六进制等价物的前面。例如，IPv4 地址`74.125.21.105`的十六进制等价物是`4a7d1569`。两者都代表 32 位数量。因此，以下三个地址中的任何一个都代表相同的网站：

```java
    address = InetAddress.getByName("74.125.21.105");
    address = InetAddress.getByName("::ffff:74.125.21.105");
    address = InetAddress.getByName("::ffff:4a7d:1569");
```

如果我们使用`displayInetAddressInformation`方法来处理这些地址，输出将是相同的，如下所示：

**/74.125.21.105**

**规范主机名：yv-in-f105.1e100.net**

主机名：yv-in-f105.1e100.net

**主机地址：74.125.21.105**

**规范主机名：83.166.169.231**

这些被称为 IPv4 兼容 IPv6 地址。

`Inet6Address`类具有`isIPv4CompatibleAddress`方法。如果地址仅仅是放置在 IPv6 地址内的 IPv4 地址，则该方法返回`true`。当这种情况发生时，除了最后四个字节外，其他都是零。

以下示例说明了如何使用此方法。测试与[www.google.com](http://www.google.com)相关的每个地址，以确定它是 IPv4 地址还是 IPv6 地址。如果它是 IPv6 地址，则对其应用该方法：

```java
    try {
        InetAddress names[] = 
            InetAddress.getAllByName("www.google.com");
        for (InetAddress address : names) {
            if ((address instanceof Inet6Address) && 
                       ((Inet6Address) address)
                           .isIPv4CompatibleAddress()) {
                System.out.println(address
                        + " is IPv4 Compatible Address");
            } else {
                System.out.println(address
                        + " is not a IPv4 Compatible Address");
            }
        }
    } catch (UnknownHostException ex) {
        // Handle exceptions
    }
```

输出取决于可用的服务器。以下是一个可能的输出：

**www.google.com/173.194.46.48 不是 IPv4 兼容地址**

**www.google.com/173.194.46.51 不是 IPv4 兼容地址**

**www.google.com/173.194.46.49 不是 IPv4 兼容地址**

**www.google.com/173.194.46.52 不是 IPv4 兼容地址**

**www.google.com/173.194.46.50 不是 IPv4 兼容地址**

**www.google.com/2607:f8b0:4009:80b:0:0:0:2004 不是 IPv4 兼容地址**

另一个 Java 8 解决方案如下：

```java
    names = InetAddress.getAllByName("www.google.com");
    Arrays.stream(names)
            .map(address -> {
                if ((address instanceof Inet6Address) && 
                        ((Inet6Address) address)
                            .isIPv4CompatibleAddress()) {
                    return address + 
                        " is IPv4 Compatible Address";
                } else {
                    return address + 
                        " is not IPv4 Compatible Address";
                }
            })
            .forEach(result -> System.out.println(result));
```

# 控制网络属性

在许多操作系统上，默认行为是使用 IPv4 而不是 IPv6。在执行 Java 应用程序时，可以使用以下 JVM 选项来控制此行为。第一个设置如下：

```java
-Djava.net.preferIPv4Stack=false
```

这是默认设置。如果 IPv6 可用，则应用程序可以使用 IPv4 或 IPv6 主机。如果设置为`true`，它将使用 IPv4 主机。将不使用 IPv6 主机。

第二个设置涉及使用的地址类型：

```java
-Djava.net.preferIPv6Addresses=false
```

这是默认设置。如果 IPv6 可用，它将优先使用 IPv4 地址而不是 IPv6 地址。这是首选的，因为它允许 IPv4 服务的向后兼容性。如果设置为`true`，它将在可能的情况下使用 IPv6 地址。

# 总结

本章概述了基本网络术语和概念。网络是一个庞大而复杂的主题。在本章中，我们重点关注了与 Java 中的网络相关的概念。

引入了`NetworkInterface`类。该类提供对连接到支持网络的计算机的设备的低级访问。我们还学习了如何获取设备的 MAC 地址。

我们关注 Java 提供的访问互联网的支持。详细介绍了基础 IP 协议。该协议由`InetAddress`类支持。Java 使用`Inet4Address`和`Inet6Address`类分别支持 IPv4 和 IPv6 地址。

我们还说明了`URI`和`URL`类的用法。这些类具有几种方法，允许我们获取有关特定实例的更多信息。我们可以使用这些方法将 URI 或 URL 分割成部分以进行进一步处理。

我们还讨论了如何控制一些网络连接属性。我们将在后面的章节中更详细地介绍这个主题。

有了这个基础，我们现在可以继续讨论使用 NIO 包来支持网络。NIO 是面向缓冲区的，支持非阻塞 IO。此外，它提供了更好的性能，适用于许多 IO 操作。
