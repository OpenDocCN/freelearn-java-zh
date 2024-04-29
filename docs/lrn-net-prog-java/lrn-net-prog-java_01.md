# 第一章：网络编程入门

访问网络（特别是互联网）正在成为应用程序的一个重要且经常必要的特性。应用程序经常需要访问和提供服务。随着物联网连接越来越多的设备，了解如何访问网络变得至关重要。

推动更多网络应用程序的重要因素包括更快速的网络和更大带宽的可用性。这使得传输更广泛范围的数据成为可能，比如视频流。近年来，我们看到了连接性的增加，无论是用于新服务、更广泛的社交互动还是游戏。知道如何开发网络应用程序是一项重要的发展技能。

在本章中，我们将介绍网络编程的基础知识：

+   为什么网络是重要的

+   Java 提供的支持

+   解决基本网络操作的简单程序

+   基本网络术语

+   一个简单的服务器/客户端应用程序

+   使用线程支持服务器

在本书中，您将接触到许多网络概念、思想、模式和使用旧版和新版 Java 技术的实现策略。网络连接在低级别使用套接字，而在更高级别使用多种协议。通信可以是同步的，需要仔细协调请求和响应，也可以是异步的，在提交响应之前执行其他活动。

这些以及其他概念通过一系列章节进行讨论，每个章节都专注于特定主题。这些章节通过详细阐述先前介绍的概念来互补彼此。尽可能使用大量的代码示例来进一步加深您对主题的理解。

访问服务的核心是知道或发现其地址。这个地址可能是人类可读的，比如[www.packtpub.com](http://www.packtpub.com)，或者是 IP 地址，比如`83.166.169.231`。Internet Protocol（IP）是一种低级寻址方案，用于访问互联网上的信息。寻址长期以来一直使用 IPv4 来访问资源。然而，这些地址几乎用光了。新的 IPv6 可提供更大范围的地址。网络寻址的基础知识以及如何在 Java 中管理它们是第二章的重点，*网络寻址*。

网络通信的目的是将信息传输到其他应用程序并从其他应用程序中传输信息。这是通过缓冲区和通道来实现的。缓冲区临时保存信息，直到应用程序可以处理它。通道是一种简化应用程序之间通信的抽象。NIO 和 NIO.2 包提供了大部分缓冲区和通道的支持。我们将探讨这些技术以及其他技术，如阻塞和非阻塞 IO，在第三章中，*NIO 支持网络*。

服务是由服务器提供的。一个例子是简单的回显服务器，它重新传输它收到的内容。更复杂的服务器，如 HTTP 服务器，可以支持广泛的服务，以满足各种需求。客户端/服务器模型及其 Java 支持在第三章中有所涵盖，*NIO 支持网络*。

另一个服务模型是**点对点**（**P2P**）模型。在这种架构中，没有中央服务器，而是一组通信以提供服务的应用程序网络。这种模型由应用程序表示，例如 BitTorrent、Skype 和 BBC 的 iPlayer。虽然这些类型应用程序开发所需的大部分支持超出了本书的范围，但是第四章 *客户端/服务器开发* 探讨了 P2P 问题以及 Java 和 JXTA 提供的支持。

IP 在低级别用于在网络上发送和接收信息包。我们还将演示**用户数据报协议**（**UDP**）和**传输控制协议**（**TCP**）通信协议的使用。这些协议是在 IP 之上分层的。UDP 用于广播短数据包或消息，没有可靠交付的保证。TCP 更常用，提供比 UDP 更高级别的服务。我们将在第五章 *点对点网络* 中介绍这些相关技术的使用。

由于许多因素的影响，服务通常会面临不同程度的需求。它的负载可能会随着一天中的时间而变化。随着它变得更受欢迎，其整体需求也会增加。服务器需要扩展以满足负载的增加和减少。线程和线程池已被用于支持这一努力。这些以及其他技术是第六章 *UDP 和多播* 的重点。

应用程序越来越需要防范黑客的攻击。当它连接到网络时，这种威胁会增加。在第七章 *网络可扩展性* 中，我们将探讨许多可用于支持安全 Java 应用程序的技术。其中包括**安全套接字层**（**SSL**）以及 Java 对其的支持。

应用程序很少是孤立运行的。因此，它们需要使用网络来访问其他应用程序。但并非所有应用程序都是用 Java 编写的。与这些应用程序进行网络通信可能会带来特殊问题，从数据类型的字节组织方式到应用程序支持的接口。通常需要使用专门的协议，例如 HTTP 和 WSDL。本书的最后一章从 Java 的角度探讨了这些问题。

我们将演示较旧和较新的 Java 技术。了解较旧的技术可能是必要的，以便维护较旧的代码，并且可以洞察为什么开发了较新的技术。我们还将使用许多 Java 8 函数式编程技术来补充我们的示例。使用 Java 8 示例以及 Java 8 之前的实现，我们可以学习如何使用 Java 8，并更好地了解何时可以和应该使用它。

本书的目的不是完全解释较新的 Java 8 技术，例如 lambda 表达式和流。然而，使用 Java 8 示例将提供关于如何使用它们来支持网络应用程序的见解。

本章的其余部分涉及本书中探讨的许多网络技术。您将介绍这些技术的基础知识，并且应该会发现它们很容易理解。但是，有一些地方我们没有时间充分探讨和解释这些概念。这些问题将在随后的章节中解决。因此，让我们从网络寻址开始探索。

# 使用 InetAddress 类进行网络寻址

IP 地址由`InetAddress`类表示。地址可以是单播，其中它标识特定地址，也可以是多播，其中消息发送到多个地址。

`InetAddress`类没有公共构造函数。要获得一个实例，使用其中几个静态的获取类型方法。例如，`getByName`方法接受一个表示地址的字符串，如下所示。在这种情况下，字符串是一个**统一资源定位符**（**URL**）：

```java
    InetAddress address = 
        InetAddress.getByName("www.packtpub.com");
    System.out.println(address);
```

### 提示

下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载您购买的所有 Packt 图书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，直接将文件发送到您的电子邮件。

这将显示以下结果：

**www.packtpub.com/83.166.169.231**

附加到名称末尾的数字是 IP 地址。这个地址唯一标识互联网上的实体。

如果我们需要有关地址的其他信息，可以使用几种方法，如下所示：

```java
    System.out.println("CanonicalHostName: " 
        + address.getCanonicalHostName());
    System.out.println("HostAddress: " + 
        address.getHostAddress());
    System.out.println("HostName: " + address.getHostName());
```

执行时会产生以下输出：

**规范主机名：83.166.169.231**

**主机地址：83.166.169.231**

**主机名：www.packtpub.com**

要测试这个地址是否可达，使用`isReachable`方法，如下所示。它的参数指定在决定地址是否不可达之前等待多长时间。参数是等待的毫秒数：

```java
    address.isReachable(10000);
```

还有`Inet4Address`和`Inet6Address`类分别支持 IPv4 和 IPv6 地址。我们将在第二章*网络寻址*中解释它们的用法。

一旦我们获得了一个地址，我们可以用它来支持网络访问，比如与服务器一起。在我们演示它在这个上下文中的使用之前，让我们看看如何获得和处理来自连接的数据。

# NIO 支持

`java.io`、`java.nio`和`java.nio`子包提供了大部分 Java 对 IO 处理的支持。我们将在第三章*NIO 支持网络*中检查这些包为网络访问提供的支持。在这里，我们将专注于`java.nio`包的基本方面。

NIO 包中使用了三个关键概念：

+   **通道**：这代表应用程序之间的数据流

+   **缓冲区**：这与通道一起处理数据

+   **选择器**：这是一种允许单个线程处理多个通道的技术

通道和缓冲区通常是相互关联的。数据可以从通道传输到缓冲区，也可以从缓冲区传输到通道。缓冲区，顾名思义，是信息的临时存储库。选择器对支持应用程序的可伸缩性很有用，这将在第七章*网络可伸缩性*中讨论。

有四个主要通道：

+   `FileChannel`：这与文件一起使用

+   `DatagramChannel`：这支持 UDP 通信

+   `SocketChannel`：这与 TCP 客户端一起使用

+   `ServerSocketChannel`：这与 TCP 服务器一起使用

有几个缓冲区类支持原始数据类型，如字符、整数和浮点数。

## 使用 URLConnection 类

访问服务器的一种简单方式是使用`URLConnection`类。这个类表示应用程序和`URL`实例之间的连接。`URL`实例代表互联网上的资源。

在下一个示例中，为 Google 网站创建了一个 URL 实例。使用`URL`类的`openConnection`方法，创建了一个`URLConnection`实例。使用`BufferedReader`实例从连接中读取行，然后显示：

```java
    try {
        URL url = new URL("http://www.google.com");
        URLConnection urlConnection = url.openConnection();
        BufferedReader br = new BufferedReader(
                new InputStreamReader(
                    urlConnection.getInputStream()));
        String line;
        while ((line = br.readLine()) != null) {
            System.out.println(line);
        }
        br.close();
    } catch (IOException ex) {
        // Handle exceptions
    }
```

输出相当冗长，因此这里只显示了第一行的一部分：

**<!doctype html><html itemscope="" itemtype="http://schema.org/WebPage" ...**

`URLConnection`类隐藏了访问 HTTP 服务器的一些复杂性。

## 使用 URLConnection 类与缓冲区和通道

我们可以重新调整前面的例子，以说明通道和缓冲区的使用。`URLConnection`实例与以前一样创建。我们将创建一个`ReadableByteChannel`实例，然后创建一个`ByteBuffer`实例，如下例所示。`ReadableByteChannel`实例允许我们使用其`read`方法从站点读取数据。`ByteBuffer`实例从通道接收数据，并用作`read`方法的参数。创建的缓冲区每次可容纳 64 个字节。

`read`方法返回读取的字节数。`ByteBuffer`类的`array`方法返回一个字节数组，该数组用作`String`类的构造函数的参数。这用于显示读取的数据。`clear`方法用于重置缓冲区，以便可以再次使用：

```java
    try {
        URL url = new URL("http://www.google.com");
        URLConnection urlConnection = url.openConnection();
        InputStream inputStream = urlConnection.getInputStream();
        ReadableByteChannel channel = 
            Channels.newChannel(inputStream);
        ByteBuffer buffer = ByteBuffer.allocate(64);
        String line = null;
        while (channel.read(buffer) > 0) {
            System.out.println(new String(buffer.array()));
            buffer.clear();
        }
        channel.close();
    } catch (IOException ex) {
        // Handle exceptions
    }
```

下面显示了输出的第一行。这产生了与以前相同的输出，但限制为每次显示 64 个字节：

**<!doctype html><html itemscope="" itemtype="http://schema.org/We**

`Channel`类及其派生类提供了一种改进的技术，用于访问网络上的数据，而不是使用旧技术提供的数据。我们将看到更多关于这个类的内容。

# 客户端/服务器架构

有几种使用 Java 创建服务器的方法。我们将说明一些简单方法，并推迟对这些技术的详细讨论，直到第四章，“客户端/服务器开发”。将创建客户端和服务器。

服务器安装在具有 IP 地址的机器上。在任何给定时间，一台机器上可能运行多个服务器。当操作系统接收到对机器上某项服务的请求时，它还将接收到一个端口号。端口号将标识应将请求转发到的服务器。因此，服务器由其 IP 地址和端口号的组合来标识。

通常，客户端会向服务器发出请求。服务器将接收请求并发送回响应。请求/响应的性质和通信所使用的协议取决于客户端/服务器。有时会使用一种良好记录的协议，例如**超文本传输协议**（**HTTP**）。对于更简单的架构，会来回发送一系列文本消息。

为了使服务器与发出请求的应用程序进行通信，需要使用专门的软件来发送和接收消息。这种软件称为套接字。一个套接字位于客户端，另一个套接字位于服务器端。当它们连接时，通信就成为可能。有几种不同类型的套接字。这些包括数据报套接字；经常使用 TCP 的流套接字；以及通常在 IP 级别工作的原始套接字。我们将专注于 TCP 套接字用于我们的客户端/服务器应用程序。

具体来说，我们将创建一个简单的回显服务器。该服务器将接收来自客户端的文本消息，并立即将其发送回该客户端。这个服务器的简单性使我们能够专注于客户端-服务器的基础知识。

# 创建一个简单的回显服务器

我们将从下面显示的`SimpleEchoServer`类的定义开始。在`main`方法中，将显示初始服务器消息：

```java
public class SimpleEchoServer {
    public static void main(String[] args) {
        System.out.println("Simple Echo Server");
        ...
    }
}
```

方法的其余部分由一系列 try 块组成，用于处理异常。在第一个 try 块中，使用`6000`作为参数创建了一个`ServerSocket`实例。`ServerSocket`类是服务器用于监听客户端请求的专用套接字。它的参数是端口号。服务器所在机器的 IP 地址对服务器来说并不一定重要，但客户端最终需要知道这个 IP 地址。

在下一个代码序列中，创建了`ServerSocket`类的一个实例，并调用了它的`accept`方法。`ServerSocket`将阻塞此调用，直到它收到来自客户端的请求。阻塞意味着程序暂停，直到方法返回。收到请求时，`accept`方法将返回一个`Socket`类实例，表示客户端和服务器之间的连接。他们现在可以发送和接收消息：

```java
    try (ServerSocket serverSocket = new ServerSocket(6000)){
        System.out.println("Waiting for connection.....");
        Socket clientSocket = serverSocket.accept();
        System.out.println("Connected to client");
         ...
    } catch (IOException ex) {
        // Handle exceptions
    }
```

创建了客户端套接字后，我们可以处理发送到服务器的消息。由于我们处理的是文本，我们将使用`BufferedReader`实例从客户端读取消息。这是使用客户端套接字的`getInputStream`方法创建的。我们将使用`PrintWriter`实例回复客户端。这是使用客户端套接字的`getOutputStream`方法创建的，如下所示：

```java
    try (BufferedReader br = new BufferedReader(
                new InputStreamReader(
                clientSocket.getInputStream()));
            PrintWriter out = new PrintWriter(
                clientSocket.getOutputStream(), true)) {
        ...
        }
    }
```

`PrintWriter`构造函数的第二个参数设置为`true`。这意味着使用`out`对象发送的文本将在每次使用后自动刷新。

当文本被写入套接字时，它将停留在缓冲区中，直到缓冲区满或调用刷新方法。执行自动刷新可以避免我们记得刷新缓冲区，但可能导致过度刷新，而在最后一次写入后发出单个刷新也可以。

下一个代码段完成了服务器。`readLine`方法一次从客户端读取一行。这个文本被显示，然后使用`out`对象发送回客户端：

```java
    String inputLine;
    while ((inputLine = br.readLine()) != null) {
        System.out.println("Server: " + inputLine);
        out.println(inputLine);
    }
```

在演示服务器运行之前，我们需要创建一个客户端应用程序来使用它。

## 创建一个简单的回声客户端

我们从声明一个`SimpleEchoClient`类开始，在`main`方法中，显示一条消息指示应用程序的启动，如下所示：

```java
public class SimpleEchoClient {
    public static void main(String args[]) {
        System.out.println("Simple Echo Client");
        ...
    }
}
```

需要创建一个`Socket`实例来连接到服务器。在下面的示例中，假设服务器和客户端在同一台机器上运行。`InetAddress`类的静态`getLocalHost`方法返回这个地址，然后在`Socket`类的构造函数中与端口`6000`一起使用。如果它们位于不同的机器上，则需要使用服务器的地址。与服务器一样，创建`PrintWriter`和`BufferedReader`类的实例，以允许文本被发送到服务器和从服务器接收：

```java
    try {
        System.out.println("Waiting for connection.....");
        InetAddress localAddress = InetAddress.getLocalHost();

        try (Socket clientSocket = new Socket(localAddress, 6000);
                    PrintWriter out = new PrintWriter(
                        clientSocket.getOutputStream(), true);
                    BufferedReader br = new BufferedReader(
                        new InputStreamReader(
                        clientSocket.getInputStream()))) {
            ...
        }
    } catch (IOException ex) {
        // Handle exceptions
    }
```

### 注意

本地主机指的是当前机器。它有一个特定的 IP 地址：`127.0.0.1`。虽然一台机器可能关联有额外的 IP 地址，但每台机器都可以使用这个本地主机地址来访问自己。

然后提示用户输入文本。如果文本是退出命令，则终止无限循环，并关闭应用程序。否则，使用`out`对象将文本发送到服务器。当回复返回时，它将如下所示显示：

```java
    System.out.println("Connected to server");
    Scanner scanner = new Scanner(System.in);
    while (true) {
        System.out.print("Enter text: ");
        String inputLine = scanner.nextLine();
        if ("quit".equalsIgnoreCase(inputLine)) {
            break;
        }
        out.println(inputLine);
        String response = br.readLine();
        System.out.println("Server response: " + response);
    }
```

这些程序可以作为两个单独的项目或一个单一的项目来实现。无论哪种方式，都先启动服务器，然后再启动客户端。当服务器启动时，您将看到以下内容显示：

**简单回声服务器**

**等待连接.....**

当客户端启动时，您将看到以下内容：

**简单回声客户端**

**等待连接.....**

**连接到服务器**

**输入文本：**

输入一条消息，观察客户端和服务器的交互。以下是客户端的可能输入序列：

**输入文本：你好服务器**

**服务器响应：你好服务器**

**输入文本：回显这个！**

**服务器响应：回显这个！**

**输入文本：quit**

客户端输入`quit`命令后，服务器的输出如下所示：

**简单回声服务器**

**等待连接.....**

**连接到客户端**

**客户端请求：你好服务器**

**客户端请求：回显这个！**

这是一种实现客户端和服务器的方法。我们将在后面的章节中增强这个实现。

## 使用 Java 8 支持回声服务器和客户端

我们将提供许多新的 Java 8 功能的使用示例。在这里，我们将向您展示先前回声服务器和客户端应用程序的替代实现。

服务器使用 while 循环来处理客户端的请求，如下所示：

```java
    String inputLine;
    while ((inputLine = br.readLine()) != null) {
        System.out.println("Client request: " + inputLine);
        out.println(inputLine);
    }
```

我们可以使用`Supplier`接口与`Stream`对象结合使用来执行相同的操作。下一条语句使用 lambda 表达式从客户端返回一个字符串：

```java
    Supplier<String> socketInput = () -> {
        try {
            return br.readLine();
        } catch (IOException ex) {
            return null;
        }
    };
```

从`Supplier`实例生成一个无限流。以下`map`方法从用户那里获取输入，然后将其发送到服务器。当输入`quit`时，流将终止。`allMatch`方法是一个短路方法，当其参数求值为`false`时，流将终止：

```java
    Stream<String> stream = Stream.generate(socketInput);
    stream
            .map(s -> {
                System.out.println("Client request: " + s);
                out.println(s);
                return s;
            })
            .allMatch(s -> s != null);
```

虽然这种实现比传统实现更长，但它可以提供更简洁和简单的解决更复杂的问题。

在客户端上，我们可以用功能性实现替换 while 循环，如下所示：

```java
    while (true) {
        System.out.print("Enter text: ");
        String inputLine = scanner.nextLine();
        if ("quit".equalsIgnoreCase(inputLine)) {
            break;
        }
        out.println(inputLine);

        String response = br.readLine();
        System.out.println("Server response: " + response);
    }
```

功能性解决方案还使用`Supplier`实例来捕获控制台输入，如下所示：

```java
    Supplier<String> scannerInput = () -> scanner.next();
```

生成一个无限流，如下所示，`map`方法提供了等效的功能：

```java
    System.out.print("Enter text: ");
    Stream.generate(scannerInput)
        .map(s -> {
            out.println(s);
            System.out.println("Server response: " + s);
            System.out.print("Enter text: ");
            return s;
        })
        .allMatch(s -> !"quit".equalsIgnoreCase(s));
```

对于许多问题，功能性方法通常是更好的解决方案。

请注意，在输入`quit`命令后，客户端会显示一个额外的提示**Enter text:**。如果输入了`quit`命令，则可以通过不显示提示来轻松纠正此问题。这个纠正留给读者作为练习。

# UDP 和多播

如果您需要定期向一组发送消息，则多播是一种有用的技术。它使用 UDP 服务器和一个或多个 UDP 客户端。为了说明这种能力，我们将创建一个简单的时间服务器。服务器将每秒向客户端发送一个日期和时间字符串。

多播将向组中的每个成员发送相同的消息。组由多播地址标识。多播地址必须使用以下 IP 地址范围：`224.0.0.0`到`239.255.255.255`。服务器将发送一个带有此地址的消息。客户端必须在接收任何多播消息之前加入该组。

## 创建多播服务器

接下来声明一个`MulticastServer`类，其中创建一个`DatagramSocket`实例。try-catch 块将处理异常：

```java
public class MulticastServer {
    public static void main(String args[]) {
        System.out.println("Multicast  Time Server");
        DatagramSocket serverSocket = null;
        try {
            serverSocket = new DatagramSocket();
            ...
            }
        } catch (SocketException ex) {
            // Handle exception
        } catch (IOException ex) {
            // Handle exception
        }
    }
}
```

try 块的主体使用无限循环来创建一个字节数组来保存当前的日期和时间。接下来，创建一个表示多播组的`InetAddress`实例。使用数组和组地址，实例化一个`DatagramPacket`，并将其用作`DatagramSocket`类的`send`方法的参数。然后显示发送的日期和时间。然后服务器暂停一秒：

```java
    while (true) {
        String dateText = new Date().toString();
        byte[] buffer = new byte[256];
        buffer = dateText.getBytes();

        InetAddress group = InetAddress.getByName("224.0.0.0");
        DatagramPacket packet;
        packet = new DatagramPacket(buffer, buffer.length, 
            group, 8888);
        serverSocket.send(packet);
        System.out.println("Time sent: " + dateText);

        try {
            Thread.sleep(1000);
        } catch (InterruptedException ex) {
            // Handle exception
        }
    }
```

此服务器只广播消息。它从不接收来自客户端的消息。

## 创建多播客户端

使用以下`MulticastClient`类创建客户端。为了接收消息，客户端必须使用相同的组地址和端口号。在接收消息之前，它必须使用`joinGroup`方法加入组。在此实现中，它接收 5 条日期和时间消息，显示它们，然后终止。`trim`方法从字符串中删除前导和尾随的空格。否则，将显示消息的所有 256 个字节：

```java
public class MulticastClient {
    public static void main(String args[]) {
        System.out.println("Multicast  Time Client");
        try (MulticastSocket socket = new MulticastSocket(8888)) {
            InetAddress group = 
                InetAddress.getByName("224.0.0.0");
            socket.joinGroup(group);
            System.out.println("Multicast  Group Joined");

            byte[] buffer = new byte[256];
            DatagramPacket packet = 
                new DatagramPacket(buffer, buffer.length);

            for (int i = 0; i < 5; i++) {
                socket.receive(packet);
                String received = new String(packet.getData());
                System.out.println(received.trim());
            }

            socket.leaveGroup(group);
        } catch (IOException ex) {
            // Handle exception
        }
        System.out.println("Multicast  Time Client Terminated");
    }
}
```

当服务器启动时，发送的消息将显示如下：

**多播时间服务器**

**Time sent: Thu Jul 09 13:19:49 CDT 2015**

**Time sent: Thu Jul 09 13:19:50 CDT 2015**

**Time sent: Thu Jul 09 13:19:51 CDT 2015**

**Time sent: Thu Jul 09 13:19:52 CDT 2015**

**Time sent: Thu Jul 09 13:19:53 CDT 2015**

**Time sent: Thu Jul 09 13:19:54 CDT 2015**

**Time sent: Thu Jul 09 13:19:55 CDT 2015**

**...**

客户端输出将类似于以下内容：

**多播时间客户端**

**加入多播组**

**Thu Jul 09 13:19:50 CDT 2015**

**Thu Jul 09 13:19:51 CDT 2015**

**周四 2015 年 7 月 9 日 13:19:52 CDT**

**周四 2015 年 7 月 9 日 13:19:53 CDT**

**周四 2015 年 7 月 9 日 13:19:54 CDT**

**多播时间客户端已终止**

### 注意

如果在 Mac 上执行示例，可能会收到一个异常，指示它无法分配所请求的地址。这可以通过使用 JVM 选项`-Djava.net.preferIPv4Stack=true`来解决。

还有许多其他的多播功能，将在第六章中探讨，*UDP 和多播*。

# 可扩展性

当服务器的需求增加和减少时，改变专门用于服务器的资源是可取的。可用的选项范围从使用手动线程以允许并发行为到嵌入在专门的类中处理线程池和 NIO 通道。

## 创建一个多线程服务器

在本节中，我们将使用线程来增强我们的简单回显服务器。`ThreadedEchoServer`类的定义如下。它实现了`Runnable`接口，为每个连接创建一个新线程。私有的`Socket`变量将保存特定线程的客户端套接字：

```java
public class ThreadedEchoServer implements Runnable {
    private static Socket clientSocket;

    public ThreadedEchoServer(Socket clientSocket) {
        this.clientSocket = clientSocket;
    }
    ...
}
```

### 注意

线程是应用程序中与其他代码块并发执行的代码块。`Thread`类支持 Java 中的线程。创建线程的几种方法之一是将实现`Runnable`接口的对象传递给其构造函数。当调用`Thread`类的`start`方法时，线程被创建，并且`Runnable`接口的`run`方法被执行。当`run`方法终止时，线程也终止。

添加线程的另一种方法是使用一个单独的类来处理线程。这可以与`ThreadedEchoServer`类分开声明，也可以作为`ThreadedEchoServer`类的内部类。使用单独的类更好地分割了应用程序的功能。

`main`方法创建了服务器套接字，但是当创建客户端套接字时，客户端套接字被用来创建一个线程，如下所示：

```java
    public static void main(String[] args) {
        System.out.println("Threaded Echo Server");
        try (ServerSocket serverSocket = new ServerSocket(6000)) {
            while (true) {
                System.out.println("Waiting for connection.....");
                clientSocket = serverSocket.accept();
                ThreadedEchoServer tes = 
                    new ThreadedEchoServer(clientSocket);
                new Thread(tes).start();
            }

        } catch (IOException ex) {
            // Handle exceptions
        }
        System.out.println("Threaded Echo Server Terminating");
    }
```

实际的工作是在`run`方法中执行的，如下所示。它本质上与原始的回显服务器的实现相同，只是当前线程被显示以澄清使用了哪些线程：

```java
    @Override
    public void run() {
        System.out.println("Connected to client using [" 
            + Thread.currentThread() + "]");
        try (BufferedReader br = new BufferedReader(
                new InputStreamReader(
                    clientSocket.getInputStream()));
                PrintWriter out = new PrintWriter(
                        clientSocket.getOutputStream(), true)) {
            String inputLine;
            while ((inputLine = br.readLine()) != null) {
                System.out.println("Client request [" 
                    + Thread.currentThread() + "]: " + inputLine);
                out.println(inputLine);
            }
            System.out.println("Client [" + Thread.currentThread() 
                + " connection terminated");
        } catch (IOException ex) {
            // Handle exceptions
        }
    }
```

## 使用多线程服务器

以下输出显示了服务器与两个客户端之间的互动。原始的回显客户端被启动了两次。正如你所看到的，每个客户端的互动都是使用不同的线程执行的：

**多线程回显服务器**

**等待连接.....**

**等待连接.....**

**使用[Thread[Thread-0,5,main]]连接到客户端**

**客户端请求[Thread[Thread-0,5,main]]：来自客户端 1 的问候**

**客户端请求[Thread[Thread-0,5,main]]：这边很好**

**等待连接.....**

**使用[Thread[Thread-1,5,main]]连接到客户端**

**客户端请求[Thread[Thread-1,5,main]]：来自客户端 2 的问候**

**客户端请求[Thread[Thread-1,5,main]]：美好的一天！**

**客户端请求[Thread[Thread-1,5,main]]：退出**

**客户端[Thread[Thread-1,5,main]连接已终止**

**客户端请求[Thread[Thread-0,5,main]]：再见**

**客户端请求[Thread[Thread-0,5,main]]：退出**

以下互动来自第一个客户端的角度：

**简单回显客户端**

**等待连接.....**

**连接到服务器**

**输入文本：来自客户端 1 的问候**

**服务器响应：来自客户端 1 的问候**

**输入文本：这边很好**

**服务器响应：这边很好**

**输入文本：再见**

**服务器响应：再见**

**输入文本：退出**

**服务器响应：退出**

以下互动来自第二个客户端的角度：

**简单回显客户端**

**等待连接.....**

**连接到服务器**

**输入文本：来自客户端 2 的问候**

**服务器响应：来自客户端 2 的问候**

**输入文本：美好的一天！**

**服务器响应：美好的一天！**

**输入文本：退出**

**服务器响应：退出**

此实现允许同时处理多个客户端。客户端不会因为另一个客户端正在使用服务器而被阻塞。但是，它也允许创建大量线程。如果存在太多线程，则服务器性能可能会下降。我们将在第七章*网络可扩展性*中解决这些问题。

# 安全

安全是一个复杂的话题。在本节中，我们将演示与网络相关的这个话题的一些简单方面。具体来说，我们将创建一个安全的回声服务器。创建安全的回声服务器与我们之前开发的非安全回声服务器并没有太大的不同。但是，背后有很多工作正在进行。现在我们可以忽略其中许多细节，但是我们将在第八章*网络安全*中更深入地探讨。

我们将使用`SSLServerSocketFactory`类来实例化安全服务器套接字。此外，还需要创建底层 SSL 机制可以使用的密钥来加密通信。

## 创建 SSL 服务器

在以下示例中声明了一个`SSLServerSocket`类作为回声服务器。由于它与之前的回声服务器类似，我们不会解释其实现，除非它与`SSLServerSocketFactory`类的使用有关。它的静态`getDefault`方法返回一个`ServerSocketFactory`实例。其`createServerSocket`方法返回一个绑定到端口`8000`的`ServerSocket`实例，能够支持安全通信。否则，它与之前的回声服务器组织和功能类似：

```java
public class SSLServerSocket {

    public static void main(String[] args) {
        try {
            SSLServerSocketFactory ssf =  (SSLServerSocketFactory) 
                SSLServerSocketFactory.getDefault();
            ServerSocket serverSocket = 
                ssf.createServerSocket(8000);
            System.out.println("SSLServerSocket Started");
            try (Socket socket = serverSocket.accept();
                    PrintWriter out = new PrintWriter(
                            socket.getOutputStream(), true);
                    BufferedReader br = new BufferedReader(
                        new InputStreamReader(
                        socket.getInputStream()))) {
                System.out.println("Client socket created");
                String line = null;
                while (((line = br.readLine()) != null)) {
                    System.out.println(line);
                    out.println(line);
                }
                br.close();
                System.out.println("SSLServerSocket Terminated");
            } catch (IOException ex) {
                // Handle exceptions
            }
        } catch (IOException ex) {
            // Handle exceptions
        }
    }
}
```

## 创建 SSL 客户端

安全回声客户端也类似于之前的非安全回声客户端。`SSLSocketFactory`类的`getDefault`返回一个`SSLSocketFactory`实例，其`createSocket`创建一个连接到安全回声服务器的套接字。应用程序如下：

```java
public class SSLClientSocket {

    public static void main(String[] args) throws Exception {
        System.out.println("SSLClientSocket Started");
        SSLSocketFactory sf = 
            (SSLSocketFactory) SSLSocketFactory.getDefault();
        try (Socket socket = sf.createSocket("localhost", 8000);
                PrintWriter out = new PrintWriter(
                       socket.getOutputStream(), true);
                BufferedReader br = new BufferedReader(
                       new InputStreamReader(
                       socket.getInputStream()))) {
            Scanner scanner = new Scanner(System.in);
            while (true) {
                System.out.print("Enter text: ");
                String inputLine = scanner.nextLine();
                if ("quit".equalsIgnoreCase(inputLine)) {
                    break;
                }
                out.println(inputLine);
                System.out.println("Server response: " + 
                    br.readLine());
            }
            System.out.println("SSLServerSocket Terminated");
        }
    }
}
```

如果我们先执行这个服务器，然后是客户端，它们将因连接错误而中止。这是因为我们没有提供一组应用程序可以共享和用于保护它们之间传递的数据的密钥。

## 生成安全密钥

为了提供必要的密钥，我们需要创建一个密钥库来保存密钥。当应用程序执行时，密钥库必须对应用程序可用。首先，我们将演示如何创建一个密钥库，然后我们将向您展示必须提供哪些运行时参数。

在 Java SE SDK 的`bin`目录中有一个名为`keytool`的程序。这是一个命令级程序，将生成必要的密钥并将它们存储在密钥文件中。在 Windows 中，您需要打开一个命令窗口并导航到源文件的根目录。该目录将包含包含应用程序包的目录。

### 注意

在 Mac 上，您可能会遇到生成密钥对的问题。有关在 Mac 上使用此工具的更多信息，请访问[`developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/keytool.1.html`](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/keytool.1.html)。

您还需要使用类似以下命令的命令设置到`bin`目录的路径。这个命令是为了找到并执行`keytool`应用程序：

```java
 set path= C:\Program Files\Java\jdk1.8.0_25\bin;%path%

```

接下来，输入`keytool`命令。您将被提示输入密码和其他用于创建密钥的信息。这个过程如下所示，其中使用密码`123456`，尽管它输入时没有显示：

```java
Enter keystore password:
Re-enter new password:
What is your first and last name?
 [Unknown]:  First Last
What is the name of your organizational unit?
 [Unknown]:  packt
What is the name of your organization?
 [Unknown]:  publishing
What is the name of your City or Locality?
 [Unknown]:  home
What is the name of your State or Province?
 [Unknown]:  calm
What is the two-letter country code for this unit?
 [Unknown]:  me
Is CN=First Last, OU=packt, O=publishing, L=home, ST=calm, C=me correct?
 [no]:  y

Enter key password for <mykey>
 (RETURN if same as keystore password):

```

创建了密钥库后，您可以运行服务器和客户端应用程序。这些应用程序的启动方式取决于您的项目是如何创建的。您可以从 IDE 中执行它，也可以从命令窗口启动它们。

接下来是可以从命令窗口使用的命令。`java`命令的两个参数是密钥库的位置和密码。它们需要从包目录的根目录执行：

```java
java -Djavax.net.ssl.keyStore=keystore.jks -Djavax.net.ssl.keyStorePassword=123456 packt.SSLServerSocket
java -Djavax.net.ssl.trustStore=keystore.jks -Djavax.net.ssl.trustStorePassword=123456 packt.SSLClientSocket
```

如果您想使用 IDE，则请使用相应的运行时命令参数设置。以下是客户端和服务器之间可能的一种交互。首先显示服务器窗口的输出，然后是客户端的输出：

**SSLServerSocket 已启动**

**客户端套接字已创建**

**你好回声服务器**

**安全可靠**

**SSLServerSocket 已终止**

**SSLClientSocket 已启动**

**输入文本：你好回声服务器**

**服务器响应：你好回声服务器**

**输入文本：安全可靠**

**服务器响应：安全可靠**

**输入文本：退出**

**SSLServerSocket 已终止**

关于 SSL 还有更多内容需要学习。然而，这提供了一个过程概述，更多细节在第八章 *网络安全*中呈现。

# 总结

网络应用程序在当今社会中扮演着越来越重要的角色。随着越来越多的设备连接到互联网，了解如何构建可以与其他应用程序通信的应用程序变得至关重要。

我们简要介绍并解释了 Java 用于连接到网络的几种技术。我们演示了`InetAddress`类如何表示 IP 地址，并在几个示例中使用了这个类。使用 UDP、TCP 和 SSL 技术演示了客户端/服务器架构的基本要素。它们提供不同类型的支持。UDP 速度快，但不像 TCP 那样可靠或功能强大。TCP 是一种可靠和方便的通信方式，但如果不与 SSL 一起使用，则不安全。

我们演示了缓冲区和通道的 NIO 支持。这些技术可以实现更高效的通信。对于许多应用程序来说，应用程序的可扩展性至关重要，特别是客户端/服务器模型。我们还看到了线程如何支持可扩展性。

这些主题将在后续章节中得到更详细的讨论。这包括 NIO 对可扩展性的支持，P2P 应用程序的工作原理，以及可用于与 Java 一起使用的各种互操作技术。

我们将在下一章中开始对网络和网络寻址进行详细的研究。
