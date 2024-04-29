# 第六章。UDP 和多播

**用户数据报协议**（**UDP**）位于 IP 之上，提供了 TCP 的不可靠对应。UDP 在网络中的两个节点之间发送单独的数据包。UDP 数据包不知道其他数据包，并且不能保证数据包实际到达其预期目的地。当发送多个数据包时，不能保证到达顺序。UDP 消息只是被发送然后被遗忘，因为没有来自接收方的确认。

UDP 是一种无连接的协议。两个节点之间没有消息交换来促进数据包传输。关于连接的状态信息不会被维护。

UDP 适用于需要高效传递的服务，且不需要传递保证的情况。例如，它用于**域名系统**（**DNS**）服务，**网络时间协议**（**NTP**）服务，**语音传输**（**VOIP**），P2P 网络的网络通信协调，以及视频流媒体。如果视频帧丢失，那么如果丢失不频繁，则观看者可能永远不会注意到。

有几种使用 UDP 的协议，包括：

+   **实时流媒体协议（RTSP）**：该协议用于控制媒体的流媒体

+   **路由信息协议（RIP）**：该协议确定用于传输数据包的路由

+   **域名系统（DNS）**：该协议查找互联网域名并返回其 IP 地址

+   **网络时间协议（NTP）**：该协议在互联网上同步时钟

UDP 数据包由 IP 地址和端口号组成，用于标识其目的地。UDP 数据包具有固定大小，最大可达 65,353 字节。然而，每个数据包使用最少 20 字节的 IP 头和 8 字节的 UDP 头，限制了消息的大小为 65,507 字节。如果消息大于这个大小，那么就需要发送多个数据包。

UDP 数据包也可以进行多播。这意味着数据包被发送到属于 UDP 组的每个节点。这是一种有效的方式，可以将信息发送到多个节点，而无需明确地针对每个节点。相反，数据包被发送到一个负责捕获其组数据包的组。

在本章中，我们将说明 UDP 协议如何被用于：

+   支持传统的客户端/服务器模型

+   使用 NIO 通道执行 UDP 操作

+   多播数据包到组成员

+   向客户端流媒体，如音频或视频

我们将从 Java 对 UDP 的支持概述开始，并提供更多 UDP 协议的细节。

# Java 对 UDP 的支持

Java 使用`DatagramSocket`类在节点之间形成套接字连接。`DatagramPacket`类表示数据包。简单的发送和接收方法将在网络中传输数据包。

UDP 使用 IP 地址和端口号来标识节点。UDP 端口号范围从`0`到`65535`。端口号分为三种类型：

+   知名端口（`0`到`1023`）：这些是用于相对常见服务的端口号。

+   注册端口（`1024`到`49151`）：这些是由 IANA 分配给进程的端口号。

+   动态/私有端口（`49152`到`65535`）：这些在连接初始化时动态分配给客户端。这些通常是临时的，不能由 IANA 分配。

以下表格是 UDP 特定端口分配的简要列表。它们说明了 UDP 被广泛用于支持许多不同的应用和服务。TCP/UDP 端口号的更完整列表可在[`en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers`](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)找到：

| 知名端口（0 到 1023） | 用途 |
| --- | --- |
| `7` | 这是回显协议 |
| `9` | 这意味着远程唤醒 |
| `161` | 这是**简单** **网络管理协议**（**SNMP**） |
| `319` | 这些是**精密时间协议**（**PTP**）事件消息 |
| `320` | 这些是 PTP 通用消息 |
| `513` | 这表示用户是谁 |
| `514` | 这是 syslog—用于系统日志 |
| `520` | 这是**路由信息协议**（**RIP**） |
| `750` | 这是`kerberos-iv`，Kerberos 第四版 |
| `944` | 这是网络文件系统服务 |
| `973` | 这是 IPv6 上的网络文件系统服务 |

以下表格列出了注册端口及其用途：

| 注册端口（1024 到 49151） | 用途 |
| --- | --- |
| `1534` | 用于 Eclipse**目标通信框架**（**TCF**） |
| `1581` | 用于 MIL STD 2045-47001 VMF |
| `1589` | 用于思科**虚拟局域网查询协议**（**VQP**）/ VMPS |
| `2190` | 用于 TiVoConnect Beacon |
| `2302` | 用于 Halo：战斗进化多人游戏 |
| `3000` | 用于 BitTorrent 同步 |
| `4500` | 用于 IPSec NAT 穿透 |
| `5353` | 用于**多播 DNS**（**mDNS**） |
| `9110` | 用于 SSMP 消息协议 |
| `27500`到`27900` | 用于 id Software 的 QuakeWorld |
| `29900`到`29901` | 用于任天堂 Wi-Fi 连接 |
| `36963` | 用于虚幻软件多人游戏 |

# TCP 与 UDP

TCP 和 UDP 之间存在几个区别。这些区别包括以下内容：

+   可靠性：TCP 比 UDP 更可靠

+   **顺序**：TCP 保证数据包传输的顺序将被保留

+   **头部大小**：UDP 头部比 TCP 头部小

+   **速度**：UDP 比 TCP 更快

当使用 TCP 发送数据包时，数据包保证会到达。如果丢失，则会重新发送。UDP 不提供此保证。如果数据包未到达，则不会重新发送。

TCP 保留了发送数据包的顺序，而 UDP 则没有。如果 TCP 数据包到达目的地的顺序与发送时不同，TCP 将重新组装数据包以恢复其原始顺序。而 UDP 则不保留此顺序。

创建数据包时，会附加头信息以帮助传递数据包。使用 UDP 时，头部由 8 个字节组成。TCP 头部的通常大小为 32 个字节。

由于较小的头部大小和缺少确保可靠性的开销，UDP 比 TCP 更有效率。此外，创建连接需要的工作量更少。这种效率使其成为流媒体的更好选择。

让我们从支持传统客户端/服务器架构的 UDP 示例开始。

# UDP 客户端/服务器

UDP 客户端/服务器应用程序的结构与 TCP 客户端/服务器应用程序所使用的结构类似。在服务器端，创建了一个 UDP 服务器套接字，等待客户端请求。客户端将创建相应的 UDP 套接字，并使用它向服务器发送消息。服务器随后可以处理请求并发送回响应。

UDP 客户端/服务器将使用`DatagramSocket`类作为套接字，使用`DatagramPacket`来保存消息。消息的内容类型没有限制。在我们的示例中，我们将使用文本消息。

## UDP 服务器应用程序

接下来定义我们的服务器。构造函数将执行服务器的工作：

```java
public class UDPServer {
    public UDPServer() {
        System.out.println("UDP Server Started");
        ...
        System.out.println("UDP Server Terminating");
    }

    public static void main(String[] args) {
        new UDPServer();
    }
}
```

在构造函数的 try-with-resources 块中，我们创建了`DatagramSocket`类的实例。我们将使用的一些方法可能会抛出`IOException`异常，必要时将被捕获：

```java
        try (DatagramSocket serverSocket = 
                new DatagramSocket(9003)) {
            ...
            }
        } catch (IOException ex) {
            //Handle exceptions
        }
```

创建套接字的另一种方法是使用`bind`方法，如下所示。使用`null`作为参数创建`DatagramSocket`实例。然后使用`bind`方法分配端口：

```java
        DatagramSocket serverSocket = new DatagramSocket(null); 
        serverSocket.bind(new InetSocketAddress(9003)); 
```

两种方法都将使用端口`9003`创建`DatagramSocket`实例。

发送消息的过程包括以下步骤：

+   创建字节数组

+   创建`DatagramPacket`实例

+   使用`DatagramSocket`实例等待消息到达

该过程被包含在一个循环中，如下所示，以允许处理多个请求。接收到的消息将简单地回显到客户端程序。使用字节数组及其长度创建`DatagramPacket`实例。它作为`DatagramSocket`类的`receive`方法的参数。此时数据包不包含任何信息。此方法将阻塞，直到有请求发出，然后数据包将被填充：

```java
        while (true) {
            byte[] receiveMessage = new byte[1024];
            DatagramPacket receivePacket = new DatagramPacket(
                receiveMessage, receiveMessage.length);
            serverSocket.receive(receivePacket);
            ...
        }
```

当方法返回时，数据包将被转换为字符串。如果发送了其他数据类型，则需要其他转换。然后显示发送的消息：

```java
        String message = new String(receivePacket.getData());
        System.out.println("Received from client: [" + message
               + "]\nFrom: " + receivePacket.getAddress());
```

要发送响应，需要客户端的地址和端口号。这些分别使用`getAddress`和`getPort`方法从拥有这些信息的数据包中获取。我们将在讨论客户端时看到这一点。还需要的是表示为字节数组的消息，`getBytes`方法提供了这个消息：

```java
        InetAddress inetAddress = receivePacket.getAddress();
        int port = receivePacket.getPort();
        byte[] sendMessage;
        sendMessage = message.getBytes();
```

使用消息、其长度和客户端的地址和端口号创建一个新的`DatagramPacket`实例。`send`方法将数据包发送到客户端：

```java
        DatagramPacket sendPacket = 
            new DatagramPacket(sendMessage,
                sendMessage.length, inetAddress, port);
        serverSocket.send(sendPacket);
```

定义了服务器，现在让我们来看看客户端。

## UDP 客户端应用程序

客户端应用程序将提示用户输入要发送的消息，然后将消息发送到服务器。它将等待响应，然后显示响应。在这里声明：

```java
class UDPClient {
    public UDPClient() {
        System.out.println("UDP Client Started");
        ...
        }
        System.out.println("UDP Client Terminating ");
    }

    public static void main(String args[]) {
        new UDPClient();
    }
}
```

`Scanner`类支持获取用户输入。try-with-resources 块创建了一个`DatagramSocket`实例并处理异常：

```java
        Scanner scanner = new Scanner(System.in);
        try (DatagramSocket clientSocket = new DatagramSocket()) {
            ...
            }
            clientSocket.close();
        } catch (IOException ex) {
            // Handle exceptions
        }
```

使用`getByName`方法访问客户端的当前地址，并声明一个字节数组的引用。此地址将用于创建数据包：

```java
        InetAddress inetAddress = 
            InetAddress.getByName("localhost");
        byte[] sendMessage;
```

使用无限循环提示用户输入消息。当用户输入“quit”时，应用程序将终止，如下所示：

```java
        while (true) {
            System.out.print("Enter a message: ");
            String message = scanner.nextLine();
            if ("quit".equalsIgnoreCase(message)) {
                 break;
            }
        ...
        }
```

要创建一个包含消息的`DatagramPacket`实例，其构造函数需要一个表示消息的字节数组，其长度以及客户端的地址和端口号。在下面的代码中，服务器的端口是`9003`。`send`方法将数据包发送到服务器：

```java
            sendMessage = message.getBytes();
            DatagramPacket sendPacket = new DatagramPacket(
                sendMessage, sendMessage.length, 
                inetAddress, 9003);
            clientSocket.send(sendPacket);
```

为了接收响应，创建一个接收数据包，并与在服务器中处理方式相同地使用`receive`方法。此方法将阻塞，直到服务器响应，然后显示消息：

```java
            byte[] receiveMessage = new byte[1024];
            DatagramPacket receivePacket = new DatagramPacket(
                    receiveMessage, receiveMessage.length);
            clientSocket.receive(receivePacket);
            String receivedSentence = 
                new String(receivePacket.getData());
            System.out.println("Received from server [" 
                + receivedSentence + "]\nfrom "
                + receivePacket.getSocketAddress());
```

现在，让我们看看这些应用程序是如何工作的。

## UDP 客户端/服务器在运行

首先启动服务器。它将显示以下消息：

**UDP 服务器已启动**

接下来，启动客户端应用程序。它将显示以下消息：

**UDP 客户端已启动**

**输入消息：**

输入一条消息，例如以下消息：

**输入消息：早上好**

服务器将显示已收到消息，如下所示。您将看到几行空白的输出。这是用于保存消息的 1024 字节数组的内容。然后将消息回显到客户端：

从客户端收到：[早上好**

**...**

**]**

**来自：/127.0.0.1**

在客户端端，显示了响应。在这个例子中，用户然后输入“quit”来终止应用程序：

从服务器收到：[早上好**

**...**

**]**

**来自/127.0.0.1:9003**

**输入消息：quit**

**UDP 客户端终止**

由于我们正在发送和接收测试消息，当显示消息时，可以使用`trim`方法简化消息的显示，如下所示。此代码可以在服务器和客户端两侧使用：

```java
        System.out.println("Received from client: [" 
                + message.trim()
                + "]\nFrom: " + receivePacket.getAddress());
```

输出将更容易阅读，如下所示：

**从客户端收到：[早上好]**

**来自：/127.0.0.1**

这个客户端/服务器应用程序可以通过多种方式进行增强，包括使用线程，以使其能够更好地与多个客户端一起工作。此示例说明了在 Java 中开发 UDP 客户端/服务器应用程序的基础知识。在下一节中，我们将看到通道如何支持 UDP。

# UDP 的通道支持

`DatagramChannel`类提供了对 UDP 的额外支持。它可以支持非阻塞交换。`DatagramChannel`类是从`SelectableChannel`类派生的，使多线程应用程序更容易。我们将在第七章中研究它的用法，*网络可扩展性*。

`DatagramSocket`类将通道绑定到端口。使用此类后，将不再直接使用。使用`DatagramChannel`类意味着我们不必直接使用数据报包。相反，数据是使用`ByteBuffer`类的实例进行传输。该类提供了几种方便的方法来访问其数据。

为了演示`DatagramChannel`类的用法，我们将开发一个回显服务器和客户端应用程序。服务器将等待来自客户端的消息，然后将其发送回客户端。

## UDP 回显服务器应用程序

UDP 回显服务器应用程序声明如下，并使用端口`9000`。在`main`方法中，使用 try-with-resources 块打开通道并创建套接字。`DatagramChannel`类没有公共构造函数。要创建通道，我们使用`open`方法，它返回`DatagramChannel`类的实例。通道的`socket`方法为通道创建一个`DatagramSocket`实例：

```java
public class UDPEchoServer {

    public static void main(String[] args) {
        int port = 9000;
        System.out.println("UDP Echo Server Started");
        try (DatagramChannel channel = DatagramChannel.open();
            DatagramSocket socket = channel.socket();){
                ...
            }
        }
        catch (IOException ex) {
            // Handle exceptions
        }
        System.out.println("UDP Echo Server Terminated");
    }
}
```

创建后，我们需要将其与端口关联。首先通过创建`SocketAddress`类的实例来完成，该类表示套接字地址。`InetSocketAddress`类是从`SocketAddress`类派生的，并实现了 IP 地址。在以下代码序列中的使用将其与端口`9000`关联。`DatagramSocket`类的`bind`方法将此地址绑定到套接字：

```java
            SocketAddress address = new InetSocketAddress(port);
            socket.bind(address);
```

`ByteBuffer`类是使用数据报通道的核心。我们在第三章中讨论了它的创建，*NIO 支持网络*。在下一个语句中，使用`allocateDirect`方法创建了该类的一个实例。此方法将尝试直接在缓冲区上使用本机操作系统支持。这可能比使用数据报包方法更有效。在这里，我们创建了一个具有可能的最大大小的缓冲区：

```java
            ByteBuffer buffer = ByteBuffer.allocateDirect(65507);
```

添加以下无限循环，它将接收来自客户端的消息，显示消息，然后将其发送回去：

```java
            while (true) {
                // Get message
                // Display message
                // Return message
            }
```

`receive`方法应用于通道以获取客户端的消息。它将阻塞直到消息被接收。它的单个参数是用于保存传入数据的字节缓冲区。如果消息超过缓冲区的大小，额外的字节将被静默丢弃。

`flip`方法使缓冲区可以被处理。它将缓冲区的限制设置为缓冲区中的当前位置，然后将位置设置为`0`。随后的获取类型方法将从缓冲区的开头开始：

```java
        SocketAddress client = channel.receive(buffer);
        buffer.flip();
```

虽然对于回显服务器来说并非必需，但接收到的消息会显示在服务器上。这样可以验证消息是否已接收，并建议如何修改消息以实现更多功能，而不仅仅是回显消息。

为了显示消息，我们需要使用`get`方法逐个获取每个字节，然后将其转换为适当的类型。回显服务器旨在回显简单的字符串。因此，在显示之前，需要将字节转换为字符。

然而，`get`方法修改了缓冲区中的当前位置。在将消息发送回客户端之前，我们需要将位置恢复到其原始状态。缓冲区的`mark`和`reset`方法用于此目的。

所有这些都在以下代码序列中执行。`mark`方法在当前位置设置标记。使用`StringBuilder`实例重新创建客户端发送的字符串。缓冲区的`hasRemaining`方法控制 while 循环。消息被显示，`reset`方法将位置恢复到先前标记的值：

```java
        buffer.mark();
        System.out.print("Received: [");
        StringBuilder message = new StringBuilder();
        while (buffer.hasRemaining()) {
            message.append((char) buffer.get());
        }
        System.out.println(message + "]");
        buffer.reset();
```

最后一步是将字节缓冲区发送回客户端。`send`方法执行此操作。显示消息指示消息已发送，然后是`clear`方法。因为我们已经完成了缓冲区的使用，所以使用此方法。它将位置设置为 0，将缓冲区的限制设置为其容量，并丢弃标记：

```java
        channel.send(buffer, client);
        System.out.println("Sent: [" + message + "]");
        buffer.clear();
```

当服务器启动时，我们将看到此效果的消息，如下所示：

**UDP 回声服务器已启动**

我们现在准备看看客户端是如何实现的。

## UDP 回显客户端应用程序

UDP 回显客户端的实现简单，并使用以下步骤：

+   与回声服务器建立连接

+   创建一个字节缓冲区来保存消息

+   缓冲区被发送到服务器

+   客户端阻塞，直到消息被发送回来

客户端的实现细节与服务器的类似。我们从应用程序的声明开始，如下所示：

```java
public class UDPEchoClient {

    public static void main(String[] args) {
        System.out.println("UDP Echo Client Started");
        try {
            ...
        }
        catch (IOException ex) {
            // Handle exceptions
        }
        System.out.println("UDP Echo Client Terminated");
    }
}
```

在服务器端，单参数`InetSocketAddress`构造函数将端口`9000`与当前 IP 地址关联。在客户端中，我们需要指定服务器的 IP 地址和端口。否则，它将无法确定要发送消息的位置。这是在以下语句中使用类的两个参数构造函数来实现的。我们使用地址`127.0.0.1`，假设客户端和服务器在同一台机器上：

```java
        SocketAddress remote = 
            new InetSocketAddress("127.0.0.1", 9000);
```

然后使用`open`方法创建通道，并使用`connect`方法连接到套接字地址：

```java
        DatagramChannel channel = DatagramChannel.open();
        channel.connect(remote);
```

在下一个代码序列中，创建消息字符串，并分配字节缓冲区。将缓冲区的大小设置为字符串的长度。然后，`put`方法将消息分配给缓冲区。由于`put`方法需要一个字节数组，我们使用`String`类的`getBytes`方法获取与消息内容对应的字节数组：

```java
        String message = "The message";
        ByteBuffer buffer = ByteBuffer.allocate(message.length());
        buffer.put(message.getBytes());
```

在将缓冲区发送到服务器之前，调用`flip`方法。它将设置限制为当前位置，并将位置设置为 0。因此，当服务器接收时，可以进行处理：

```java
        buffer.flip();
```

要将消息发送到服务器，调用通道的`write`方法，如下所示。这将直接将底层数据包发送到服务器。但是，此方法仅在通道的套接字已连接时才有效，这是之前实现的：

```java
        channel.write(buffer);
        System.out.println("Sent: [" + message + "]");
```

接下来，清除缓冲区，允许我们重用缓冲区。`read`方法将接收缓冲区，并且缓冲区将使用与服务器中使用的相同的过程显示：

```java
        buffer.clear();
        channel.read(buffer);
        buffer.flip();
        System.out.print("Received: [");
        while(buffer.hasRemaining()) {
            System.out.print((char)buffer.get());
        }
        System.out.println("]");
```

我们现在准备与服务器一起使用客户端。

## UDP 回显客户端/服务器正在运行

首先需要启动服务器。我们将看到初始服务器消息，如下所示：

**UDP 回声服务器已启动**

接下来，启动客户端。将显示以下输出，显示客户端发送消息，然后显示返回的消息：

**UDP 回显客户端已启动**

**发送：[消息]**

**接收：[消息]**

**UDP 回显客户端终止**

在服务器端，我们将看到消息被接收，然后被发送回客户端：

**接收：[消息]**

**发送：[消息]**

使用`DatagramChannel`类可以使 UDP 通信更快。

# UDP 多播

多播是将消息同时发送给多个客户端的过程。每个客户端将接收相同的消息。为了参与此过程，客户端需要加入多播组。当发送消息时，其目标地址指示它是多播消息。多播组是动态的，客户端可以随时加入和离开组。

多播是旧的 IPv4 CLASS D 空间，使用地址`224.0.0.0`到`239.255.255.255`。IPv4 多播地址空间注册表列出了多播地址分配，并可在[`www.iana.org/assignments/multicast-addresses/multicast-addresses.xml`](http://www.iana.org/assignments/multicast-addresses/multicast-addresses.xml)找到。*IP 多播主机扩展*文档可在[`tools.ietf.org/html/rfc1112`](http://tools.ietf.org/html/rfc1112)找到。它定义了支持多播的实现要求。

## UDP 多播服务器

接下来声明服务器应用程序。这个服务器是一个时间服务器，每秒广播当前日期和时间。这是多播消息的一个很好的用途，因为可能有几个客户端对相同的信息感兴趣，可靠性不是一个问题。try 块将处理异常：

```java
public class UDPMulticastServer {

    public UDPMulticastServer() {
        System.out.println("UDP Multicast Time Server Started");
        try {
            ...
        } catch (IOException | InterruptedException ex) {
            // Handle exceptions
        }
        System.out.println(
            "UDP Multicast Time Server Terminated");
    }

    public static void main(String args[]) {
        new UDPMulticastServer();
    }
}
```

需要`MulticastSocket`类的一个实例，以及保存多播 IP 地址的`InetAddress`实例。在本例中，地址`228.5.6.7`代表多播组。使用`joinGroup`方法加入此多播组，如下所示：

```java
    MulticastSocket multicastSocket = new MulticastSocket();
    InetAddress inetAddress = InetAddress.getByName("228.5.6.7");
    multicastSocket.joinGroup(inetAddress);
```

为了发送消息，我们需要一个字节数组来保存消息和一个数据包。如下所示声明：

```java
    byte[] data;
    DatagramPacket packet;
```

服务器应用程序将使用无限循环每秒广播一个新的日期和时间。线程暂停一秒，然后使用`Data`类创建一个新的日期和时间。使用此信息创建`DatagramPacket`实例。为此服务器分配端口`9877`，客户端需要知道该端口。`send`方法将数据包发送给感兴趣的客户端：

```java
    while (true) {
        Thread.sleep(1000);
        String message = (new Date()).toString();
        System.out.println("Sending: [" + message + "]");
        data = message.getBytes();
        packet = new DatagramPacket(data, message.length(), 
                inetAddress, 9877);
        multicastSocket.send(packet);
    }
```

接下来讨论客户端应用程序。

## UDP 多播客户端

此应用程序将加入由地址`228.5.6.7`定义的多播组。它将阻塞直到接收到消息，然后显示消息。应用程序定义如下：

```java
public class UDPMulticastClient {

    public UDPMulticastClient() {
        System.out.println("UDP Multicast Time Client Started");
        try {
            ...
        } catch (IOException ex) {
            ex.printStackTrace();
        }

        System.out.println(
            "UDP Multicast Time Client Terminated");
    }

    public static void main(String[] args) {
        new UDPMulticastClient();
    }
}
```

使用端口号`9877`创建`MulticastSocket`类的实例。这是必需的，以便它可以连接到 UDP 多播服务器。使用多播地址`228.5.6.7`创建`InetAddress`实例。然后客户端使用`joinGroup`方法加入多播组。

```java
    MulticastSocket multicastSocket = new MulticastSocket(9877);
    InetAddress inetAddress = InetAddress.getByName("228.5.6.7");
    multicastSocket.joinGroup(inetAddress);
```

需要一个`DatagramPacket`实例来接收发送到客户端的消息。创建一个字节数组并用于实例化此数据包，如下所示：

```java
    byte[] data = new byte[256];
    DatagramPacket packet = new DatagramPacket(data, data.length);
```

然后客户端应用程序进入无限循环，在`receive`方法处阻塞，直到服务器发送消息。一旦消息到达，消息将被显示：

```java
    while (true) {
        multicastSocket.receive(packet);
        String message = new String(
            packet.getData(), 0, packet.getLength());
        System.out.println("Message from: " + packet.getAddress() 
            + " Message: [" + message + "]");
    }
```

接下来，我们将演示客户端和服务器是如何交互的。

## UDP 多播客户端/服务器正在运行

启动服务器。服务器的输出将类似于以下内容，但日期和时间将不同：

**UDP 多播时间服务器已启动**

发送：[2015 年 9 月 19 日周六 13:48:42 CDT]

发送：[2015 年 9 月 19 日周六 13:48:43 CDT]

发送：[2015 年 9 月 19 日周六 13:48:44 CDT]

发送：[2015 年 9 月 19 日周六 13:48:45 CDT]

发送：[2015 年 9 月 19 日周六 13:48:46 CDT]

发送：[2015 年 9 月 19 日周六 13:48:47 CDT]

**...**

接下来启动客户端应用程序。它将开始接收类似以下内容的消息：

**UDP 多播时间客户端已启动**

来自：/192.168.1.7 消息：[2015 年 9 月 19 日周六 13:48:44 CDT]

来自：/192.168.1.7 消息：[2015 年 9 月 19 日周六 13:48:45 CDT]

来自：/192.168.1.7 消息：[2015 年 9 月 19 日周六 13:48:46 CDT]

**...**

### 注意

如果程序在 Mac 上执行，可能会出现套接字异常。如果发生这种情况，请使用`-Djava.net.preferIPv4Stack=true VM`选项。

如果启动后续客户端，每个客户端将接收相同系列的服务器消息。

# 使用通道的 UDP 多播

我们还可以使用通道进行多播。我们将使用 IPv6 来演示这个过程。这个过程类似于我们之前使用`DatagramChannel`类的过程，只是我们需要使用多播组。为此，我们需要知道哪些网络接口是可用的。在我们进入使用通道进行多播的具体细节之前，我们将演示如何获取机器的网络接口列表。

`NetworkInterface`类表示网络接口。这个类在第二章中讨论过，*网络寻址*。以下是该章节中演示的方法的变体。它已经增强，以显示特定接口是否支持多播，如下所示：

```java
        try {
            Enumeration<NetworkInterface> networkInterfaces;
            networkInterfaces = 
                NetworkInterface.getNetworkInterfaces();
            for (NetworkInterface networkInterface : 
                    Collections.list(networkInterfaces)) {
                displayNetworkInterfaceInformation(
                    networkInterface);
            }
        } catch (SocketException ex) {
            // Handle exceptions
        }
```

接下来显示`displayNetworkInterfaceInformation`方法。这种方法是从[`docs.oracle.com/javase/tutorial/networking/nifs/listing.html`](https://docs.oracle.com/javase/tutorial/networking/nifs/listing.html)中改编的：

```java
    static void displayNetworkInterfaceInformation(
            NetworkInterface networkInterface) {
        try {
            System.out.printf("Display name: %s\n", 
                networkInterface.getDisplayName());
            System.out.printf("Name: %s\n", 
                networkInterface.getName());
            System.out.printf("Supports Multicast: %s\n", 
                networkInterface.supportsMulticast());
            Enumeration<InetAddress> inetAddresses = 
                networkInterface.getInetAddresses();
            for (InetAddress inetAddress : 
                    Collections.list(inetAddresses)) {
                System.out.printf("InetAddress: %s\n", 
                    inetAddress);
            }
            System.out.println();
        } catch (SocketException ex) {
            // Handle exceptions
        }
    }
```

当执行此示例时，您将获得类似以下的输出：

**显示名称：软件环回接口 1**

**名称：lo**

**支持多播：true**

**InetAddress：/127.0.0.1**

**InetAddress：/0:0:0:0:0:0:0:1**

**显示名称：Microsoft Kernel 调试网络适配器**

名称：eth0

**支持多播：true**

**显示名称：Realtek PCIe FE Family Controller**

**名称：eth1**

**支持多播：true**

**InetAddress：/fe80:0:0:0:91d0:8e19:31f1:cb2d%eth1**

**显示名称：Realtek RTL8188EE 802.11 b/g/n Wi-Fi 适配器**

**名称：wlan0**

**支持多播：true**

**InetAddress：/192.168.1.7**

**InetAddress：/2002:42be:6659:0:0:0:0:1001**

**InetAddress：/fe80:0:0:0:9cdb:371f:d3e9:4e2e%wlan0**

**...**

对于我们的客户端/服务器，我们将使用`eth0`接口。您需要选择最适合您平台的接口。例如，在 Mac 上，这可能是`en0`或`awdl0`。

## UDP 通道多播服务器

UDP 通道多播服务器将：

+   设置通道和多播组

+   创建包含消息的缓冲区

+   使用无限循环来发送和显示组消息

服务器的定义如下：

```java
public class UDPDatagramMulticastServer {

    public static void main(String[] args) {
        try {
            ...
            }
        } catch (IOException | InterruptedException ex) {
            // Handle exceptions
        }
    }

}
```

第一个任务使用`System`类的`setProperty`方法指定要使用 IPv6。然后创建一个`DatagramChannel`实例，并创建`eth0`网络接口。`setOption`方法将通道与用于标识组的网络接口相关联。该组由一个`InetSocketAddress`实例表示，使用 IPv6 节点本地范围的多播地址，如下所示。有关*IPv6 多播地址空间注册表*文档的更多详细信息，请访问[`www.iana.org/assignments/ipv6-multicast-addresses/ipv6-multicast-addresses.xhtml`](http://www.iana.org/assignments/ipv6-multicast-addresses/ipv6-multicast-addresses.xhtml)：

```java
            System.setProperty(
                "java.net.preferIPv6Stack", "true");
            DatagramChannel channel = DatagramChannel.open();
            NetworkInterface networkInterface = 
                NetworkInterface.getByName("eth0");
            channel.setOption(StandardSocketOptions.
                IP_MULTICAST_IF, 
                networkInterface);
            InetSocketAddress group = 
                new InetSocketAddress("FF01:0:0:0:0:0:0:FC", 
                        9003);
```

然后创建一个基于消息字符串的字节缓冲区。缓冲区的大小设置为字符串的长度，并使用`put`和`getBytes`方法的组合进行分配：

```java
            String message = "The message";
            ByteBuffer buffer = 
                ByteBuffer.allocate(message.length());
            buffer.put(message.getBytes());
```

在 while 循环内，缓冲区被发送到组成员。为了清楚地看到发送了什么，使用了与*UDP 回显服务器应用程序*部分中使用的相同代码来显示缓冲区的内容。缓冲区被重置，以便可以再次使用。应用程序暂停一秒钟，以避免对这个例子产生过多的消息：

```java
            while (true) {
                channel.send(buffer, group);
                System.out.println("Sent the multicast message: " 
                    + message);
                buffer.clear();

                buffer.mark();
                System.out.print("Sent: [");
                StringBuilder msg = new StringBuilder();
                while (buffer.hasRemaining()) {
                    msg.append((char) buffer.get());
                }
                System.out.println(msg + "]");
                buffer.reset();

                Thread.sleep(1000);
        }
```

我们现在准备好客户端应用程序。

## UDP 通道多播客户端

UDP 通道多播客户端将加入组，接收消息，显示消息，然后终止。正如我们将看到的，`MembershipKey`类表示对多播组的成员资格。

应用程序声明如下。首先，我们指定要使用 IPv6。然后声明网络接口，这是服务器使用的相同接口：

```java
public class UDPDatagramMulticastClient {
    public static void main(String[] args) throws Exception {
        System.setProperty("java.net.preferIPv6Stack", "true");
        NetworkInterface networkInterface = 
            NetworkInterface.getByName("eth0");
        ...
    }
}
```

接下来创建`DatagramChannel`实例。该通道绑定到端口`9003`，并与网络接口实例相关联：

```java
        DatagramChannel channel = DatagramChannel.open()
                .bind(new InetSocketAddress(9003))
                .setOption(StandardSocketOptions.IP_MULTICAST_IF, 
                    networkInterface);
```

然后基于服务器使用的相同 IPv6 地址创建组，并使用通道的`join`方法创建一个`MembershipKey`实例，如下所示。为了说明客户端的工作原理，显示密钥和等待消息：

```java
        InetAddress group = 
            InetAddress.getByName("FF01:0:0:0:0:0:0:FC");
        MembershipKey key = channel.join(group, networkInterface);
        System.out.println("Joined Multicast Group: " + key);
        System.out.println("Waiting for a  message...");
```

创建一个大小为`1024`的字节缓冲区。这个大小对于这个例子来说足够了，然后调用`receive`方法，该方法将阻塞直到接收到消息：

```java
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        channel.receive(buffer);
```

为了显示缓冲区的内容，我们需要将其翻转。内容将如之前所做的那样显示：

```java
        buffer.flip();
        System.out.print("Received: [");
        StringBuilder message = new StringBuilder();
        while (buffer.hasRemaining()) {
            message.append((char) buffer.get());
        }
        System.out.println(message + "]");
```

当我们完成一个成员资格密钥时，应该使用`drop`方法指示我们不再对接收组消息感兴趣：

```java
        key.drop();
```

如果有待处理的数据包，消息仍然可能到达。

## UDP 通道组播客户端/服务器正在运行

首先启动服务器。该服务器将每秒显示一系列消息，如下所示：

**发送组播消息：消息**

**发送：[消息]**

**发送组播消息：消息**

**发送：[消息]**

**发送组播消息：消息**

**发送：[消息]**

**...**

接下来，启动客户端应用程序。它将显示组播组，等待消息，然后显示消息，如下所示：

**加入组播组：<ff01:0:0:0:0:0:0:fc,eth1>**

**等待消息...**

**接收：[消息]**

使用通道可以提高 UDP 组播消息的性能。

# UDP 流

使用 UDP 来流式传输音频或视频是常见的。它是高效的，任何数据包的丢失或乱序都会导致最小的问题。我们将通过实时音频流来说明这种技术。UDP 服务器将捕获麦克风的声音并将其发送给客户端。UDP 客户端将接收音频并在系统扬声器上播放。

UDP 流服务器的概念是将流分解为一系列数据包，然后发送给 UDP 客户端。客户端将接收这些数据包并使用它们来重建流。

为了说明流式音频，我们需要了解一些关于 Java 处理音频流的知识。音频由`javax.sound.sampled`包中的一系列类处理。用于捕获和播放音频的主要类包括以下内容：

+   `AudioFormat`：这个类指定所使用的音频格式的特性。由于有几种音频格式可用，系统需要知道使用的是哪一种。

+   `AudioInputStream`：这个类代表正在录制或播放的音频。

+   `AudioSystem`：这个类提供对系统音频设备和资源的访问。

+   `DataLine`：这个接口控制应用于流的操作，比如启动和停止流。

+   `SourceDataLine`：这代表声音的目的地，比如扬声器。

+   `TargetDataLine`：这代表声音的来源，比如麦克风。

`SourceDataLine`和`TargetDataLine`接口的术语可能有点令人困惑。这些术语是从线路和混音器的角度来看的。

## UDP 音频服务器实现

`AudioUDPServer`类的声明如下。它使用`TargetDataLine`实例作为音频的来源。它被声明为实例变量，因为它在多个方法中被使用。构造函数使用`setupAudio`方法来初始化音频，并使用`broadcastAudio`方法将音频发送给客户端：

```java
public class AudioUDPServer {
    private final byte audioBuffer[] = new byte[10000];
    private TargetDataLine targetDataLine;

    public AudioUDPServer() {
        setupAudio();
        broadcastAudio();
    }
    ...
    public static void main(String[] args) {
        new AudioUDPServer();
    }
}
```

以下是`getAudioFormat`方法，它在服务器和客户端中都被用来指定音频流的特性。模拟音频信号每秒采样 1,600 次。每个样本是一个带符号的 16 位数字。`channels`变量被赋值为`1`，意味着音频是单声道。样本中字节的顺序很重要，设置为大端序：

```java
    private AudioFormat getAudioFormat() {
        float sampleRate = 16000F;
        int sampleSizeInBits = 16;
        int channels = 1;
        boolean signed = true;
        boolean bigEndian = false;
        return new AudioFormat(sampleRate, sampleSizeInBits, 
            channels, signed, bigEndian);
    }
```

大端和小端是指字节的顺序。大端意味着一个字的最高有效字节存储在最小的内存地址，最低有效字节存储在最大的内存地址。小端颠倒了这个顺序。不同的计算机架构使用不同的顺序。

`setupAudio`方法初始化音频。`DataLine.Info`类使用音频格式信息创建代表音频的线路。`AudioSystem`类的`getLine`方法返回与麦克风对应的数据线。该线路被打开并启动：

```java
    private void setupAudio() {
        try {
            AudioFormat audioFormat = getAudioFormat();
            DataLine.Info dataLineInfo = 
                new DataLine.Info(TargetDataLine.class, 
                        audioFormat);
            targetDataLine =  (TargetDataLine) 
                AudioSystem.getLine(dataLineInfo);
            targetDataLine.open(audioFormat);
            targetDataLine.start();
        } catch (Exception ex) {
            ex.printStackTrace();
            System.exit(0);
        }
    }
```

`broadcastAudio`方法创建了 UDP 数据包。使用端口`8000`创建了一个套接字，并为当前机器创建了一个`InetAddress`实例：

```java
    private void broadcastAudio() {
        try {
            DatagramSocket socket = new DatagramSocket(8000);
            InetAddress inetAddress = 
                InetAddress.getByName("127.0.0.1");
            ...
        } catch (Exception ex) {
            // Handle exceptions
        }
    }
```

进入一个无限循环，`read`方法填充`audioBuffer`数组并返回读取的字节数。对于大于`0`的计数，使用缓冲区创建一个新的数据包，并发送到监听端口`9786`的客户端：

```java
    while (true) {
        int count = targetDataLine.read(
            audioBuffer, 0, audioBuffer.length);
        if (count > 0) {
            DatagramPacket packet = new DatagramPacket(
            audioBuffer, audioBuffer.length, inetAddress, 9786);
            socket.send(packet);
        }
    }
```

执行时，来自麦克风的声音被作为一系列数据包发送到客户端。

## UDP 音频客户端实现

接下来声明了`AudioUDPClient`应用程序。在构造函数中，调用了一个`initiateAudio`方法来开始从服务器接收数据包的过程：

```java
public class AudioUDPClient {
    AudioInputStream audioInputStream;
    SourceDataLine sourceDataLine;
    ...
    public AudioUDPClient() {
        initiateAudio();
    }

    public static void main(String[] args) {
        new AudioUDPClient();
    }
}
```

`initiateAudio`方法创建一个绑定到端口`9786`的套接字。创建一个字节数组来保存 UDP 数据包中包含的音频数据：

```java
    private void initiateAudio() {
        try {
            DatagramSocket socket = new DatagramSocket(9786);
            byte[] audioBuffer = new byte[10000];
            ...
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

一个无限循环将从服务器接收数据包，创建一个`AudioInputStream`实例，然后调用`playAudio`方法来播放声音。以下代码创建数据包，然后阻塞直到接收到数据包：

```java
    while (true) {
        DatagramPacket packet
            = new DatagramPacket(audioBuffer, audioBuffer.length);
        socket.receive(packet);
        ...
    }
```

接下来，创建音频流。从数据包中提取一个字节数组。它被用作`ByteArrayInputStream`构造函数的参数，该构造函数与音频格式信息一起用于创建实际的音频流。这与`SourceDataLine`实例相关联，该实例被打开并启动。调用`playAudio`方法来播放声音：

```java
        try {
            byte audioData[] = packet.getData();
            InputStream byteInputStream = 
                new ByteArrayInputStream(audioData);
            AudioFormat audioFormat = getAudioFormat();
            audioInputStream =  new AudioInputStream(
                byteInputStream, 
                audioFormat, audioData.length / 
                audioFormat.getFrameSize());
            DataLine.Info dataLineInfo = new DataLine.Info(
                SourceDataLine.class, audioFormat);
            sourceDataLine = (SourceDataLine) 
                AudioSystem.getLine(dataLineInfo);
            sourceDataLine.open(audioFormat);
            sourceDataLine.start();
            playAudio();
        } catch (Exception e) {
            // Handle exceptions
        }
```

使用`getAudioFormat`方法，该方法与`AudioUDPServer`应用程序中声明的方法相同。接下来是`playAudio`方法。`AudioInputStream`的`read`方法填充一个缓冲区，然后写入源数据线。这有效地在系统扬声器上播放声音：

```java
    private void playAudio() {
        byte[] buffer = new byte[10000];
        try {
            int count;
            while ((count = audioInputStream.read(
                   buffer, 0, buffer.length)) != -1) {
                if (count > 0) {
                    sourceDataLine.write(buffer, 0, count);
                }
            }
        } catch (Exception e) {
            // Handle exceptions
        }
    }
```

服务器运行时，启动客户端将播放来自服务器的声音。可以通过在服务器和客户端中使用线程来处理声音的录制和播放来增强播放。为简化示例，这些细节已被省略。

在这个例子中，连续的模拟声音被数字化并分成数据包。然后将这些数据包发送到客户端，在那里它们被转换回声音并播放。

在其他几个框架中还有对 UDP 流的额外支持。**Java 媒体框架**（**JMF**）([`www.oracle.com/technetwork/articles/javase/index-jsp-140239.html`](http://www.oracle.com/technetwork/articles/javase/index-jsp-140239.html))支持音频和视频媒体的处理。**实时传输协议**（**RTP**）([`en.wikipedia.org/wiki/Real-time_Transport_Protocol`](https://en.wikipedia.org/wiki/Real-time_Transport_Protocol))用于在网络上发送音频和视频数据。

# 摘要

在本章中，我们研究了 UDP 协议的性质以及 Java 如何支持它。我们对比了 TCP 和 UDP，以提供一些指导，帮助决定哪种协议对于特定问题最合适。

我们从一个简单的 UDP 客户端/服务器开始，以演示`DatagramPacket`和`DatagramSocket`类的使用方式。我们看到了`InetAddress`类是如何用来获取套接字和数据包使用的地址的。

`DatagramChannel`类支持在 UDP 环境中使用 NIO 技术，这可能比使用`DatagramPacket`和`DatagramSocket`方法更有效。该方法使用字节缓冲区来保存服务器和客户端之间发送的消息。这个例子展示了第三章中开发的许多技术，即*网络的 NIO 支持*。

接下来讨论了 UDP 多播的工作原理。这提供了一种简单的技术，可以向组成员广播消息。演示了`MulticastSocket`、`DatagramChannel`和`MembershipKey`类的使用。当使用`DatagramChannel`类时，后者类用于建立一个组。

最后，我们举了一个 UDP 用于支持音频流的例子。我们详细介绍了`javax.sound.sampled`包中几个类的使用，包括`AudioFormat`和`TargetDataLine`类用于收集和播放音频。我们使用了`DatagramSocket`和`DatagramPacket`类来传输音频。

在下一章中，我们将探讨可用于改善客户端/服务器应用程序可伸缩性的技术。
