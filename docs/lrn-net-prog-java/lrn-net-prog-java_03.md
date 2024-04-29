# 第三章。NIO 对网络的支持

在本章中，我们将重点关注 Java **New IO** (**NIO**)包的`Buffer`和`Channels`类。NIO 是早期 Java IO API 和部分网络 API 的替代品。虽然 NIO 是一个广泛而复杂的主题，但我们感兴趣的是它如何为网络应用程序提供支持。

我们将探讨几个主题，包括以下内容：

+   缓冲区，通道和选择器之间的性质和关系

+   使用 NIO 技术构建客户端/服务器

+   处理多个客户端的过程

+   支持异步套接字通道

+   基本缓冲区操作

NIO 包提供了广泛的支持，用于构建高效的网络应用程序。

# Java NIO

Java NIO 使用三个核心类：

+   `Buffer`: 保存读取或写入通道的信息

+   `Channel`: 这是一种类似流的技术，支持对数据源/汇的异步读/写操作

+   `Selector`: 这是处理单个线程中的多个通道的机制

在概念上，缓冲区和通道一起处理数据。如下图所示，数据可以在缓冲区和通道之间的任一方向移动：

![Java NIO](img/B04915_03_01.jpg)

通道连接到某个外部数据源，而缓冲区在内部用于处理数据。有几种类型的通道和缓冲区。以下是其中一些列在下表中。

通道表如下：

| 通道类 | 目的 |
| --- | --- |
| `FileChannel` | 这连接到文件 |
| `DatagramChannel` | 这支持数据报套接字 |
| `SocketChannel` | 这支持流式套接字 |
| `ServerSocketChannel` | 这用于监听套接字请求 |
| `NetworkChannel` | 这支持网络套接字 |
| `AsynchronousSocketChannel` | 这支持异步流式套接字 |

缓冲区表如下：

| 缓冲区类 | 支持的数据类型 |
| --- | --- |
| `ByteBuffer` | `byte` |
| `CharBuffer` | `char` |
| `DoubleBuffer` | `double` |
| `FloatBuffer` | `float` |
| `IntBuffer` | `int` |
| `LongBuffer` | `long` |
| `ShortBuffer` | `short` |

当应用程序使用许多低流量连接时，`Selector`类非常有用，可以使用单个线程处理这些连接。这比为每个连接创建一个线程更有效。这也是一种用于使应用程序更具可伸缩性的技术，这是我们将在第七章中讨论的内容，*网络可伸缩性*。

在本章中，我们将创建客户端/服务器应用程序，以说明通道和缓冲区之间的交互。这包括一个简单的时间服务器，一个聊天服务器来演示可变长度消息，一个部件服务器来说明处理多个客户端的一种技术，以及一个异步服务器。我们还将研究专门的缓冲区技术，包括批量传输和视图。

我们将从缓冲区的概述和它们与通道的工作方式开始讨论。

# 缓冲区介绍

缓冲区临时保存数据，因为数据正在通道之间移动。当创建缓冲区时，它是以固定大小或容量创建的。可以使用缓冲区中的一部分或全部内存，有几个`Buffer`类字段可用于管理缓冲区中的数据。

`Buffer`类是抽象的。但是，它具有用于操作缓冲区的基本方法，包括：

+   `capacity`: 返回缓冲区中的元素数量

+   `limit`: 返回不应被访问的缓冲区的第一个索引

+   `position`: 返回下一个要读取或写入的元素的索引

元素取决于缓冲区类型。

`mark`和`reset`方法还控制缓冲区内的位置。`mark`方法将缓冲区的标记设置为其位置。`reset`方法将标记位置恢复到先前标记的位置。以下代码显示了各种缓冲区术语之间的关系：

```java
0 <= mark <= position <= limit <= capacity
```

缓冲区可以是**直接**或**非直接**。直接缓冲区将尝试在可能的情况下使用本机 IO 方法。直接缓冲区的创建成本较高，但对于驻留在内存中时间较长的较大缓冲区，性能更高。使用`allocateDirect`方法创建直接缓冲区，并接受一个指定缓冲区大小的整数。`allocate`方法也接受一个整数大小参数，但创建一个非直接缓冲区。

对于大多数操作，非直接缓冲区的效率不如直接缓冲区。但是，非直接缓冲区使用的内存将被 JVM 垃圾收集器回收，而直接内存缓冲区可能不受 JVM 控制。这使得非直接缓冲区的内存管理更加可预测。

有几种方法用于在通道和缓冲区之间传输数据。这些可以分类为以下两种：

+   绝对或相对

+   批量传输

+   使用原始数据类型

+   支持视图

+   压缩、复制和切片字节缓冲区

许多`Buffer`类的方法支持调用链接。put 类型的方法将数据传输到缓冲区，而 get 类型的方法从缓冲区检索信息。我们将在我们的示例中广泛使用 get 和 put 方法。这些方法将一次传输一个字节。

这些 get 和 put 方法是相对于缓冲区内位置的当前位置。还有几种绝对方法，它们使用缓冲区中的索引来隔离特定的缓冲区元素。

批量数据传输连续的数据块。这些 get 和 put 方法使用字节数组作为它们的参数之一来保存数据。这些在“批量数据传输”部分讨论。

当`Buffer`类中的所有数据都是相同类型时，可以创建一个**视图**，以便使用特定的数据类型（如`Float`）方便地访问数据。我们将在*使用视图*部分演示这个缓冲区。

支持压缩、复制和切片类型操作。压缩操作将移动缓冲区的内容，以消除已经处理过的数据。复制将复制一个缓冲区，而切片将创建一个基于原始缓冲区的全部或部分的新缓冲区。对任一缓冲区的更改将反映在另一个缓冲区中。但是，每个缓冲区的位置、限制和标记值是独立的。

让我们看一个缓冲区的创建。

# 使用时间服务器的通道

在第一章中介绍的时间服务器和客户端，*开始网络编程*，将在这里实现，以演示缓冲区和通道的使用。这些应用程序很简单，但它们说明了缓冲区和通道如何一起使用。我们将首先创建一个服务器，然后创建一个使用该服务器的客户端。

## 创建时间服务器

以下代码是`ServerSocketChannelTimeServer`类的初始声明，它将成为我们的时间服务器。`ServerSocketChannel`类的`open`方法创建一个`ServerSocketChannel`实例。`socket`方法检索通道的`ServerSocket`实例。`bind`方法将此服务器套接字与端口`5000`关联。虽然`ServerSocketChannel`类有一个`close`方法，但使用 try-with-resources 块更容易：

```java
public class ServerSocketChannelTimeServer {
    public static void main(String[] args) {
        System.out.println("Time Server started");
        try {
            ServerSocketChannel serverSocketChannel = 
                ServerSocketChannel.open();
            serverSocketChannel.socket().bind(
                new InetSocketAddress(5000));
            ...
            }
        } catch (IOException ex) {
            // Handle exceptions
        }
    }
}
```

服务器将进入一个无限循环，其中`accept`方法会一直阻塞，直到从客户端接收到请求。当这种情况发生时，将返回一个`SocketChannel`实例：

```java
    while (true) {
        System.out.println("Waiting for request ...");
        SocketChannel socketChannel = 
            serverSocketChannel.accept();
```

假设此实例不为空，则创建包含当前日期和时间的字符串：

```java
    if (socketChannel != null) {
        String dateAndTimeMessage = "Date: " 
            +  new Date(System.currentTimeMillis());
        ...

    }
```

创建一个大小为 64 字节的`ByteBuffer`实例。这对于大多数消息来说已经足够了。`put`方法将数据移入缓冲区。这是一个批量数据传输操作。如果缓冲区不够大，则会抛出`BufferOverflowException`异常：

```java
    ByteBuffer buf = ByteBuffer.allocate(64);
    buf.put(dateAndTimeMessage.getBytes());
```

我们需要调用`flip`方法，以便可以将其与通道的写操作一起使用。这将设置限制为当前位置，并将位置设置为零。使用 while 循环来写出每个字节，并在`hasRemaining`方法确定没有更多字节可写时终止。最后一个动作是显示发送给客户端的消息：

```java
    buf.flip();
    while (buf.hasRemaining()) {
        socketChannel.write(buf);
    }
    System.out.println("Sent: " + dateAndTimeMessage);
```

当服务器启动时，它将产生类似于以下的输出：

**时间服务器已启动**

**等待请求...**

现在我们准备创建我们的客户端。

## 创建一个时间客户端

客户端在`SocketChannelTimeClient`类中实现，如下所定义。为了简化示例，假定客户端与服务器在同一台机器上。使用 IP 地址`127.0.0.1`创建一个`SocketAddress`实例，并与端口`5000`关联。`SocketChannel`类的`open`方法返回一个`SocketChannel`实例，该实例将在 try-with-resources 块中用于处理来自服务器的响应：

```java
public class SocketChannelTimeClient {
    public static void main(String[] args) {
        SocketAddress address = new InetSocketAddress(
            "127.0.0.1", 5000);
        try (SocketChannel socketChannel = 
                SocketChannel.open(address)) {
            ...
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }
}
```

在 try 块的主体中，创建一个大小为 64 的`ByteBuffer`实例。使用比实际消息更小的大小将使这个示例变得复杂。在*处理可变长度的消息*部分，我们将重新检查缓冲区大小。使用`read`方法从通道中读取消息，并将其放入`ByteBuffer`实例中。然后翻转此缓冲区以准备处理。读取每个字节，然后显示：

```java
            ByteBuffer byteBuffer = ByteBuffer.allocate(64);
            int bytesRead = socketChannel.read(byteBuffer);
            while (bytesRead > 0) {
                byteBuffer.flip();
                while (byteBuffer.hasRemaining()) {
                    System.out.print((char) byteBuffer.get());
                }
                System.out.println();
                bytesRead = socketChannel.read(byteBuffer);
            }
```

当客户端启动时，其输出将类似于以下内容：

**日期：2015 年 8 月 18 日星期二 21:36:25 CDT**

服务器的输出现在将类似于这样：

**时间服务器已启动**

**等待请求...**

**发送：日期：2015 年 8 月 18 日星期二 21:36:25 CDT**

**等待请求...**

我们现在准备检查通道和缓冲区交互的细节。

# 聊天服务器/客户端应用程序

本节的目的是更深入地演示缓冲区和通道如何一起工作。我们将使用客户端和服务器应用程序来来回传递消息。具体来说，我们将创建一个简单版本的聊天服务器。

我们将执行以下操作：

+   创建一个服务器和一个客户端，它们来回发送消息。

+   演示如何处理可变长度的消息

首先，我们将演示使用`sendFixedLengthMessage`和`receiveFixedLengthMessage`方法来处理固定大小的消息。然后我们将使用`sendMessage`和`receiveMessage`方法来处理可变长度的消息。固定长度的消息更容易处理，但如果消息的长度超过缓冲区的大小，则无法工作。可变长度的消息需要比我们在以前的示例中看到的更谨慎的处理。这些方法已放置在名为`HelperMethods`的类中，以便在多个应用程序中使用它们。

## 聊天服务器

让我们从服务器开始。服务器在`ChatServer`类中定义如下。创建一个`ServerSocketChannel`实例并绑定到端口`5000`。它将在 while 循环的主体中使用。`running`变量控制服务器的生命周期。根据需要捕获异常。与之前的服务器一样，服务器将在`accept`方法处阻塞，直到客户端连接到服务器：

```java
public class ChatServer {

    public ChatServer() {
        System.out.println("Chat Server started");
        try {
            ServerSocketChannel serverSocketChannel = 
                ServerSocketChannel.open();
            serverSocketChannel.socket().bind(
                new InetSocketAddress(5000));

            boolean running = true;
            while (running) {
                System.out.println("Waiting for request ...");
                SocketChannel socketChannel
                        = serverSocketChannel.accept();
                ...
            }
        } catch (IOException ex) {
            ex.printStackTrace();
        }        
    }

    public static void main(String[] args) {
        new ChatServer();
    }
}
```

在这个聊天/服务器应用程序中，通信是受限制的。一旦建立连接，服务器将提示用户发送消息给客户端。客户端将等待接收此消息，然后提示用户回复。回复将发送回服务器。这个顺序被限制在简化交互以便专注于通道/缓冲区交互。

当建立连接时，服务器会显示一条相应的消息，然后进入下面显示的循环。用户会被提示输入一条消息。调用`sendFixedLengthMessage`方法。如果用户输入了`quit`，那么会向服务器发送一条终止消息，然后服务器终止。否则，消息会被发送到服务器，然后服务器会在`receiveFixedLengthMessage`方法处阻塞，等待客户端的响应：

```java
    System.out.println("Connected to Client");
    String message;
    Scanner scanner = new Scanner(System.in);
    while (true) {
        System.out.print("> ");
        message = scanner.nextLine();
        if (message.equalsIgnoreCase("quit")) {
            HelperMethods.sendFixedLengthMessage(
                    socketChannel, "Server terminating");
            running = false;
            break;
        } else {
            HelperMethods.sendFixedLengthMessage(
                socketChannel, message);
            System.out.println(
                "Waiting for message from client ...");
            System.out.println("Message: " + HelperMethods
                .receiveFixedLengthMessage(socketChannel));
        }
    }
```

当服务器启动时，它的输出将如下所示：

**聊天服务器已启动**

**等待请求...**

服务器创建好后，让我们来看看客户端应用程序。

## 聊天客户端

客户端应用程序使用下面定义的`ChatClient`类。它的结构与之前的客户端应用程序类似。本地主机（`127.0.0.1`）与端口`5000`一起使用。一旦建立了连接，程序就会进入一个无限循环，并等待服务器发送一条消息：

```java
public class ChatClient {

    public ChatClient() {
        SocketAddress address = 
            new InetSocketAddress("127.0.0.1", 5000);
        try (SocketChannel socketChannel = 
                SocketChannel.open(address)) {
            System.out.println("Connected to Chat Server");
            String message;
            Scanner scanner = new Scanner(System.in);
            while (true) {
                System.out.println(
                    "Waiting for message from the server ...");
                ...
            }
        } catch (IOException ex) {
            ex.printStackTrace();
        }        
    }

    public static void main(String[] args) {
        new ChatClient();
    }
}
```

在循环中，程序会在`receiveFixedLengthMessage`方法处阻塞，直到服务器发送一条消息。然后消息会被显示出来，并提示用户发送一条消息回复给服务器。如果消息是**quit**，那么会使用`sendFixedLengthMessage`方法向服务器发送一条终止消息，并且应用程序终止。否则消息会被发送到服务器，程序会等待另一条消息：

```java
    System.out.println("Waiting for message from the server ...");
    System.out.println("Message: " 
            + HelperMethods.receiveFixedLengthMessage(
                    socketChannel));
    System.out.print("> ");
    message = scanner.nextLine();
    if (message.equalsIgnoreCase("quit")) {
        HelperMethods.sendFixedLengthMessage(
            socketChannel, "Client terminating");
        break;
    }
    HelperMethods.sendFixedLengthMessage(socketChannel, message);
```

客户端和服务器创建好后，让我们来看看它们是如何交互的。

## 服务器/客户端交互

启动服务器后，启动客户端应用程序。客户端的输出将如下所示：

**已连接到聊天服务器**

**等待来自服务器的消息...**

服务器的输出将反映这个连接：

**聊天服务器已启动**

**等待请求...**

**已连接到客户端**

输入消息`你好`。然后你会得到以下输出：

**> 你好**

**发送：你好**

**等待来自客户端的消息...**

客户端现在将显示为：

消息：你好

输入一个回复`嗨！`，客户端的输出将如下所示：

> 嗨！

发送：嗨！

**等待来自服务器的消息...**

服务器将显示为：

消息：嗨！

> 

我们可以继续这个过程，直到任何一方输入`quit`命令。然而，输入超过 64 字节缓冲区限制的消息将导致抛出`BufferOverflowException`异常。用`sendMessage`方法替换`sendFixedLengthMessage`方法，用`receiveMessage`方法替换`receiveFixedLengthMessage`方法将避免这个问题。

让我们来看看这些发送和接收方法是如何工作的。

## `HelperMethods`类

接下来定义了`HelperMethods`类。它拥有之前使用过的发送和接收方法。这些方法被声明为静态的，以便能够轻松访问它们：

```java
public class HelperMethods {
    ...
}
```

接下来显示了固定长度消息的方法。它们的执行方式基本与*使用通道与时间服务器*部分中使用的方法相同：

```java
    public static void sendFixedLengthMessage(
            SocketChannel socketChannel, String message) {
        try {
            ByteBuffer buffer = ByteBuffer.allocate(64);
            buffer.put(message.getBytes());
            buffer.flip();
            while (buffer.hasRemaining()) {
                socketChannel.write(buffer);
            }
            System.out.println("Sent: " + message);
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    public static String receiveFixedLengthMessage
            (SocketChannel socketChannel) {
        String message = "";
        try {
            ByteBuffer byteBuffer = ByteBuffer.allocate(64);
            socketChannel.read(byteBuffer);
            byteBuffer.flip();
            while (byteBuffer.hasRemaining()) {
                message += (char) byteBuffer.get();
            }
        } catch (IOException ex) {
            ex.printStackTrace();
        }
        return message;
    }
```

### 处理可变长度消息

本节讨论了处理可变长度消息的技术。可变长度消息的问题在于我们不知道它们的长度。当缓冲区没有完全填满时，我们不能假设消息的结束已经到达。虽然对于大多数消息来说这可能是正确的，但如果消息长度与消息缓冲区的大小相同，那么我们可能会错过消息的结尾。

确定何时已经到达消息的结尾的另一种方法是要么发送消息的长度作为消息的前缀，要么在消息的末尾附加一个特殊的终止字符。我们选择后者的方法。

### 注意

这个示例适用于 ASCII 字符。如果使用 Unicode 字符，将生成`BufferOverflowException`异常。`CharBuffer`类用于字符数据，并提供与`ByteBuffer`类类似的功能。`CharBuffer`类的详细信息请参阅[`docs.oracle.com/javase/8/docs/api/java/nio/CharBuffer.html`](http://docs.oracle.com/javase/8/docs/api/java/nio/CharBuffer.html)。

使用`0x00`的值来标记消息的结束。我们选择这个值是因为用户不太可能意外输入它，因为它不可打印，并且恰好对应于字符串在内部通常是如何终止的，比如 C 语言。

在接下来的`sendMessage`方法中，`put`方法在发送消息之前将终止字节添加到消息的末尾。缓冲区大小为消息长度加一。否则，代码与用于发送固定长度消息的代码类似：

```java
    public static void sendMessage(
        SocketChannel socketChannel, String message) {
        try {
            ByteBuffer buffer = 
                ByteBuffer.allocate(message.length() + 1);
            buffer.put(message.getBytes());
            buffer.put((byte) 0x00);
            buffer.flip();
            while (buffer.hasRemaining()) {
                socketChannel.write(buffer);
            }
            System.out.println("Sent: " + message);
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }
```

在`receiveMessage`方法中，每个接收到的字节都会被检查是否是终止字节。如果是，则返回消息。在我们提取消息的一部分之后，`byteBuffer`变量会应用`clear`方法。这个方法是必需的；否则，读取方法将返回`0`。该方法将缓冲区的位置设置回`0`，限制为容量：

```java
    public static String receiveMessage(SocketChannel socketChannel) {
        try {
            ByteBuffer byteBuffer = ByteBuffer.allocate(16);
            String message = "";
            while (socketChannel.read(byteBuffer) > 0) {
                char byteRead = 0x00;
                byteBuffer.flip();
                while (byteBuffer.hasRemaining()) {
                    byteRead = (char) byteBuffer.get();
                    if (byteRead == 0x00) {
                        break;
                    }
                    message += byteRead;
                }
                if (byteRead == 0x00) {
                    break;
                }
                byteBuffer.clear();
            }
            return message;
        } catch (IOException ex) {
            ex.printStackTrace();
        }
        return "";
    }
```

现在我们准备演示应用程序。

## 运行聊天服务器/客户端应用程序

首先启动服务器。输出将如下所示：

**聊天服务器已启动**

**等待请求...**

接下来，启动客户端，将得到以下输出：

**连接到聊天服务器**

**等待来自服务器的消息...**

这些用户在服务器和客户端之间的交换受到当前实现的限制。当两个应用程序都启动后，客户端将等待来自服务器的消息。这在服务器窗口中反映出来，如下所示：

**聊天服务器已启动**

**等待请求...**

**连接到客户端**

**>**

输入消息后，将发送给客户端。输入消息**你好**。客户端窗口现在将显示消息，如下所示：

**连接到聊天服务器**

**等待来自服务器的消息...**

**消息：你好**

**>**

在服务器端，将出现以下输出：

**已发送：你好**

**等待来自客户端的消息...**

现在我们可以从客户端向服务器发送消息。可以以这种方式交换消息，直到从任一应用程序发送`quit`消息为止。

# 处理多个客户端

使用线程可以实现处理多个客户端。在本节中，我们将开发一个简单的零件服务器和客户端应用程序。服务器将使用单独的线程来处理每个客户端。这种技术简单易行，但并不总是适用于更苛刻的应用程序。我们将介绍在第七章*网络可扩展性*中多任务处理的替代技术。

部件服务器实现在`PartsServer`类中，客户端实现在`PartsClient`类中。每个客户端将创建一个`ClientHandler`类的新实例。此处理程序将接受有关零件价格的请求。客户端将向处理程序发送零件的名称。处理程序将使用`PartsServer`的`getPrice`方法查找零件的价格。然后将价格返回给客户端。

## 部件服务器

部件服务器使用`HashMap`变量来保存有关零件的信息。零件的名称用作键，值存储为`Float`对象。`PartsServer`类在此处声明：

```java
public class PartsServer {
    private static final HashMap<String,Float> parts = 
            new HashMap<>();

    public PartsServer() {
        System.out.println("Part Server Started");
        ...
    }

    public static void main(String[] args) {
        new PartsServer();
    }
}
```

一旦服务器启动，将调用`initializeParts`方法：

```java
        initializeParts();
```

接下来是这个方法：

```java
    private void initializeParts() {
        parts.put("Hammer", 12.55f);
        parts.put("Nail", 1.35f);
        parts.put("Pliers", 4.65f);
        parts.put("Saw", 8.45f);
    }
```

处理程序将使用`getPrice`方法来检索零件的价格，如下所示：

```java
    public static Float getPrice(String partName) {
        return parts.get(partName);
    }
```

在调用`initializeParts`方法后，使用 try 块打开与客户端的连接，如下所示：

```java
        try {
            ServerSocketChannel serverSocketChannel = 
                ServerSocketChannel.open();
            serverSocketChannel.socket().bind(
                new InetSocketAddress(5000));
             ...
        } catch (IOException ex) {
            ex.printStackTrace();
        }
```

接下来，无限循环将为每个客户端创建一个新的处理程序。虽然在 Java 中有几种创建线程的方法，但下面使用的方法会创建`ClientHandler`类的新实例，并将客户端的套接字传递给类的构造函数。这种方法不限制应用程序创建的线程数量，这使得它容易受到拒绝服务攻击的影响。在第七章“网络可扩展性”中，我们将研究几种替代的线程方法。

`ClientHandler`实例用作`Thread`类的参数。该类将创建一个新线程，该线程将执行`ClientHandler`类的`run`方法。但是，`run`方法不应直接调用，而应调用`start`方法。此方法将创建线程所需的程序堆栈：

```java
            while (true) {
                System.out.println("Waiting for client ...");
                SocketChannel socketChannel
                        = serverSocketChannel.accept();
                new Thread(
                    new ClientHandler(socketChannel)).start();
            }
```

当服务器启动时，将显示以下输出：

零件服务器已启动

等待客户端...

让我们看看处理程序是如何工作的。

## 零件客户端处理程序

`ClientHandler`类在以下代码中定义。使用`socketChannel`实例变量连接到客户端。在`run`方法中，将显示处理程序开始的消息。虽然不是必需的，但它将帮助我们查看服务器、客户端和处理程序之间的交互。

进入无限循环，使用在“HelperMethods 类”部分中开发的`receiveMessage`方法获取零件的名称。`quit`消息将终止处理程序。否则，将调用`getPrice`方法，该方法使用`sendMessage`方法将结果返回给客户端：

```java
public class ClientHandler implements Runnable{
    private final SocketChannel socketChannel;

    public ClientHandler(SocketChannel socketChannel) {
        this.socketChannel = socketChannel;
    }

    public void run() {
        System.out.println("ClientHandler Started for " 
            + this.socketChannel);
        String partName;
        while (true) {
            partName = 
                HelperMethods.receiveMessage(socketChannel);
            if (partName.equalsIgnoreCase("quit")) {
                break;
            } else {
                Float price = PartsServer.getPrice(partName);
                HelperMethods.sendMessage(socketChannel, "" + 
                    price);
            }
        }
        System.out.println("ClientHandler Terminated for " 
            + this.socketChannel);
    }
}
```

当我们演示客户端时，我们将观察`run`方法的输出。

## 零件客户端

`PartsClient`类在下一个代码序列中定义。与服务器建立连接。显示消息以指示客户端何时启动，并且与服务器建立了连接。在 while 循环中使用`Scanner`类从用户那里获取输入：

```java
public class PartsClient {

    public PartsClient() {
        System.out.println("PartsClient Started");
        SocketAddress address = 
            new InetSocketAddress("127.0.0.1", 5000);
        try (SocketChannel socketChannel = 
                SocketChannel.open(address)) {
            System.out.println("Connected to Parts Server");
            Scanner scanner = new Scanner(System.in);
            while (true) {
            ...
            }
            System.out.println("PartsClient Terminated");
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    public static void main(String[] args) {
        new PartsClient();
    }
}
```

循环的主体将提示用户输入零件名称。如果名称是退出，则客户端将终止。否则，`sendMessage`方法将发送名称给处理程序进行处理。客户端将在`receiveMessage`方法调用时阻塞，直到服务器响应。然后将显示此零件的价格：

```java
    System.out.print("Enter part name: ");
    String partName = scanner.nextLine();
    if (partName.equalsIgnoreCase("quit")) {
        HelperMethods.sendMessage(socketChannel, "quit");
        break;
    } else {
        HelperMethods.sendMessage(socketChannel, partName);
        System.out.println("The price is " 
            + HelperMethods.receiveMessage(socketChannel));
    }
```

现在，让我们看看它们如何一起工作。

## 运行零件客户端/服务器

首先启动服务器。服务器启动时将产生以下输出：

零件服务器已启动

等待客户端...

现在，启动客户端应用程序。您将获得此输出：

PartsClient 已启动

已连接到零件服务器

输入零件名称：

输入零件名称，例如“锤子”。客户端输出现在将显示如下。如果需要，可以通过修改`sendMessage`方法来删除“Sent: Hammer”输出：

PartsClient 已启动

已连接到零件服务器

输入零件名称：锤子

已发送：锤子

价格是 12.55

输入零件名称：

在服务器端，您将获得类似以下输出。每当启动新客户端时，都会看到显示有关处理程序的信息的消息：

零件服务器已启动

等待客户端...

已为 java.nio.channels.SocketChannel[connected local=/127.0.0.1:5000 remote=/127.0.0.1:51132]启动 ClientHandler

等待客户端...

已发送：12.55

从客户端方面，我们可以继续检查价格，直到输入`quit`命令。此命令将终止客户端。一种可能的请求序列如下：

PartsClient 已启动

已连接到零件服务器

输入零件名称：锤子

已发送：锤子

价格是 12.55

输入零件名称：钳子

已发送：钳子

价格是 4.65

输入零件名称：锯子

已发送：锯子

价格为空

输入零件名称：电锯

已发送：电锯

价格是 8.45

**输入部分名称：退出**

**发送：退出**

**PartsClient 终止**

由于可能有其他客户端正在寻找价格信息，服务器将继续运行。当客户端处理程序终止时，服务器将显示类似以下的输出：

**为 java.nio.channels.SocketChannel[connected local=/127.0.0.1:5000 remote=/127.0.0.1: 51132]终止 ClientHandler**

启动两个或更多客户端，并观察它们如何与服务器交互。我们将在第七章*网络可扩展性*中研究更复杂的应用程序扩展方式。

# 异步套接字通道

异步通信涉及发出请求，然后继续进行其他操作，而无需等待请求完成。这被称为非阻塞。

有三个类用于支持异步通道操作：

+   `AsynchronousSocketChannel`：这是一个简单的到套接字的异步通道

+   `AsynchronousServerSocketChannel`：这是一个到服务器套接字的异步通道

+   `AsynchronousDatagramChannel`：这是一个用于数据报套接字的通道

`AsynchronousSocketChannel`类的读/写方法是异步的。`AsynchronousServerSocketChannel`类具有一个`accept`方法，该方法返回一个`AsynchronousSocketChannel`实例。这个方法也是异步的。我们将在第六章*UDP 和组播*中讨论`AsynchronousDatagramChannel`类。

处理异步 I/O 操作有两种方法：

+   在`java.util.concurrent`包中找到`Future`接口

+   使用`CompletionHandler`接口

`Future`接口表示一个未决结果。这通过允许应用程序继续执行而不阻塞来支持异步操作。使用这个对象，您可以使用以下方法之一：

+   `isDone`方法

+   `get`方法，它会阻塞直到完成

`get`方法有一个支持超时的重载版本。当操作完成时，将调用`CompletionHandler`实例。这本质上是一个回调。我们将不在这里说明这种方法。

我们将开发一个名为`AsynchronousServerSocketChannelServer`和`AsynchronousSocketChannelClient`的异步服务器和客户端。客户端/服务器应用程序是有限的，只允许从客户端发送消息到服务器。这将使我们能够专注于应用程序的异步方面。

## 创建异步服务器套接字通道服务器

`AsynchronousServerSocketChannelServer`类在下一个代码序列中定义。显示了服务器已启动的消息，并进入了一个 try-with-resources 块，在这个块中创建了`AsynchronousServerSocketChannel`类的一个实例，并进行了实际的工作：

```java
public class AsynchronousServerSocketChannelServer {

    public AsynchronousServerSocketChannelServer() {
        System.out.println("Asynchronous Server Started");
        try (AsynchronousServerSocketChannel serverChannel
                = AsynchronousServerSocketChannel.open()) {
        ...
        } catch (IOException | InterruptedException 
               | ExecutionException ex) {
            ex.printStackTrace();
        }        

    }

    public static void main(String[] args) {
        new AsynchronousServerSocketChannelServer();
    }

}
```

`bind`方法用于将代表`AsynchronousServerSocketChannel`实例的`serverChannel`变量与本地主机和端口`5000`关联起来：

```java
    InetSocketAddress hostAddress
        = new InetSocketAddress("localhost", 5000);
    serverChannel.bind(hostAddress);
```

服务器然后等待客户端连接。`Future`实例由`acceptResult`变量引用：

```java
    System.out.println("Waiting for client to connect... ");
    Future acceptResult = serverChannel.accept();
```

另一个 try 块用于处理客户端请求。它创建了一个`AsynchronousSocketChannel`类的实例，该实例连接到客户端。`get`方法将阻塞，直到通道被创建：

```java
    try (AsynchronousSocketChannel clientChannel
             = (AsynchronousSocketChannel) acceptResult.get()) {
        ...
    }
```

try 块的主体将分配一个缓冲区，然后从通道中读取以填充缓冲区。当缓冲区被填充时，将应用`flip`方法到缓冲区，并处理和显示消息：

```java
    System.out.println("Messages from client: ");
    while ((clientChannel != null) && (clientChannel.isOpen())) {
        ByteBuffer buffer = ByteBuffer.allocate(32);
        Future result = clientChannel.read(buffer);
        // Wait until buffer is ready using 
        // one of three techniques to be discussed
        buffer.flip();
        String message = new String(buffer.array()).trim();
        System.out.println(message);
        if (message.equals("quit")) {
            break;
        }
    }
```

有三种确定缓冲区是否准备就绪的方法。第一种技术使用`isDone`方法轮询`Future`对象，由结果变量表示，直到缓冲区准备就绪，如下所示：

```java
    while (!result.isDone()) {
        // do nothing   
    }
```

第二种技术使用`get`方法，该方法会阻塞直到缓冲区准备就绪：

```java
    result.get();
```

第三种技术也使用`get`方法，但使用超时来确定等待的时间。在这个例子中，它在超时之前等待 10 秒：

```java
    result.get(10, TimeUnit.SECONDS);
```

当使用`get`方法的这个版本时，需要在封闭的 try 块中添加一个 catch 块来处理`TimeoutException`异常。

当服务器启动时，我们得到以下输出：

**异步服务器已启动**

**等待客户端连接...**

现在，让我们来看看客户端。

## 创建异步套接字通道客户端

客户端是使用下一个代码片段中的`AsynchronousSocketChannelClient`类实现的。首先显示客户端已启动的消息，然后是创建`AsynchronousSocketChannel`实例的 try 块：

```java
public class AsynchronousSocketChannelClient {

    public static void main(String[] args) {
        System.out.println("Asynchronous Client Started");
        try (AsynchronousSocketChannel client = 
                AsynchronousSocketChannel.open()) {
            ...
        } catch (IOException | InterruptedException 
                             | ExecutionException ex) {
            // Handle exception
        }
    }

}
```

创建一个`InetSocketAddress`实例，指定服务器使用的地址和端口号。然后创建表示连接的`Future`对象。`get`方法将阻塞，直到连接建立：

```java
            InetSocketAddress hostAddress = 
                    new InetSocketAddress("localhost", 5000);
            Future future = client.connect(hostAddress);
            future.get();
```

一旦建立连接，就会显示一条消息。进入一个无限循环，提示用户输入消息。`wrap`方法将使用消息填充缓冲区。`write`方法将开始将消息写入`AsynchronousSocketChannel`实例，并返回一个`Future`对象。使用`isDone`方法等待写入完成。如果消息是**退出**，客户端应用程序将终止：

```java
    System.out.println("Client is started: " + client.isOpen());
    System.out.println("Sending messages to server: ");

    Scanner scanner = new Scanner(System.in);
    String message;
    while (true) {
        System.out.print("> ");
        message = scanner.nextLine();
        ByteBuffer buffer = ByteBuffer.wrap(message.getBytes());
        Future result = client.write(buffer);
        while (!result.isDone()) {
            // Wait
        }                
        if (message.equalsIgnoreCase("quit")) {
            break;
        }
    }
```

让我们看看异步客户端/服务器的运行情况。

在服务器运行时，启动客户端应用程序。这将产生以下输出：

**异步客户端已启动**

**客户端已启动：true**

**向服务器发送消息：**

**>**

现在服务器的输出如下：

**异步服务器已启动**

**等待客户端连接...**

**来自客户端的消息：**

使用客户端，我们可以输入以下消息：

**> 你好**

**> 这条消息来自异步客户端，发送到服务器**

**> 退出**

这些将逐一发送到服务器。从服务器端，我们将得到以下响应：

**你好**

**这条消息来自异步**

**异步客户端发送到**

**服务器**

**退出**

请注意，较长的消息已分成多行。这是使用服务器缓冲区大小仅为 32 字节的结果。更大的缓冲区将避免此问题。但是，除非我们知道将发送的最大消息的大小，否则我们需要开发一种处理长消息的方法。这留给读者作为练习。

# 其他缓冲区操作

我们将通过检查其他几种有用的缓冲区操作来结束。这些包括使用视图在缓冲区和数组之间进行批量数据传输，以及只读缓冲区。

## 批量数据传输

批量传输是在缓冲区和数组之间传输数据的一种方式。有几种支持批量数据传输的 get 和 put 类型方法。它们通常有两个版本。第一个版本使用一个参数，即传输数组。第二个版本也使用数组，但有两个额外的参数：数组中的起始索引和要传输的元素数。

为了演示这些技术，我们将使用`IntBuffer`缓冲区。我们将使用以下`displayBuffer`方法来帮助我们理解数据传输的工作原理：

```java
    public void displayBuffer(IntBuffer buffer) {
        for (int i = 0; i < buffer.position(); i++) {
            System.out.print(buffer.get(i) + " ");
        }
        System.out.println();
    }
```

我们将首先声明一个数组，并将其内容传输到缓冲区。数组在以下语句中声明并初始化：

```java
        int[] arr = {12, 51, 79, 54};
```

分配了一个比数组更大的缓冲区，如下所示。数组大小与缓冲区中可用数据之间的差异很重要。如果处理不当，将抛出异常：

```java
        IntBuffer buffer = IntBuffer.allocate(6);
```

接下来，我们将使用批量`put`方法将数组的内容传输到缓冲区：

```java
        buffer.put(arr);
```

然后使用以下语句显示缓冲区：

```java
        System.out.println(buffer);
        displayBuffer(buffer);
```

输出如下。整个数组已经传输，位置设置为下一个可用的索引：

**java.nio.HeapIntBuffer[pos=4 lim=6 cap=6]**

**12 51 79 54**

由于缓冲区仍有空间，我们可以将更多数据传输到其中。但是，我们必须小心，不要尝试传输太多数据，否则将抛出异常。第一步是确定缓冲区中剩余多少空间。如下所示，`remaining`方法可以做到这一点。然后，批量`put`语句将数组的前两个元素转移到缓冲区的最后两个位置，如下所示：

```java
        int length = buffer.remaining();
        buffer.put(arr, 0, length);
```

如果我们再次显示缓冲区及其内容，我们将得到以下输出：

**java.nio.HeapIntBuffer[pos=6 lim=6 cap=6]**

**12 51 79 54 12 51**

`get`方法被重载以支持批量数据传输。我们可以修改`displayBuffer`方法来说明这是如何工作的，如下所示。创建了一个与缓冲区内容大小相同的整数数组。`rewind`方法将缓冲区的位置移回零。然后批量`get`方法执行传输，然后是一个 for-each 循环来实际显示其内容：

```java
    public void displayBuffer(IntBuffer buffer) {
        int arr[] = new int[buffer.position()];
        buffer.rewind();
        buffer.get(arr);
        for(int element : arr) {
            System.out.print(element + " ");
        }
    }
```

## 使用视图

视图反映了另一个缓冲区中的数据。对任一缓冲区的修改都会影响另一个缓冲区。但是，位置和限制是独立的。可以使用多种方法创建视图，包括`duplicate`方法。在下面的示例中，使用批量`getBytes`方法针对字符串创建了一个缓冲区的视图。然后创建了一个视图：

```java
    String contents = "Book";
    ByteBuffer buffer = ByteBuffer.allocate(32);
    buffer.put(contents.getBytes());
    ByteBuffer duplicateBuffer = buffer.duplicate();
```

为了证明修改一个缓冲区将影响另一个缓冲区，将复制的第一个字符更改为字母“L”。然后显示每个缓冲区的第一个字节以确认更改：

```java
    duplicateBuffer.put(0,(byte)0x4c); // 'L'
    System.out.println("buffer: " + buffer.get(0));
    System.out.println("duplicateBuffer: " +
        duplicateBuffer.get(0));
```

输出将显示字母已在两个缓冲区中更改。`slice`方法还将创建一个视图，但它仅使用原始缓冲区的一部分。

## 使用只读缓冲区

默认情况下，缓冲区是读写的。但是，它可以是只读的或读写的。要创建只读缓冲区，请使用缓冲区类的`asReadOnlyBuffer`方法。在下一个序列中，创建了一个只读缓冲区：

```java
    ByteBuffer buffer = ByteBuffer.allocate(32);
    ByteBuffer readOnlyBuffer = buffer.asReadOnlyBuffer();
```

`isReadOnly`方法将确定缓冲区是否为只读，如下所示：

```java
    System.out.println("Read-only: " + 
        readOnlyBuffer.isReadOnly());
```

只读缓冲区是原始缓冲区的不同视图。对缓冲区的任何修改都会反映在另一个缓冲区中。

# 控制套接字选项

套接字类的底层套接字实现可以进行配置。可用的选项取决于套接字类型。通常，支持选项的实际机制是特定于操作系统的。而且，有时选项只是对底层实现的提示。

下面显示的每个套接字类的可用选项是从 Java API 文档中获取的：

| 类 | 选项名称 | 描述 |
| --- | --- | --- |
| `SocketChannel` | `SO_SNDBUF` | 这是套接字发送缓冲区的大小 |
| `SO_RCVBUF` | 这是套接字接收缓冲区的大小 |
| `SO_KEEPALIVE` | 这将保持连接活动 |
| `SO_REUSEADDR` | 这将重新使用地址 |
| `SO_LINGER` | 如果存在数据，则在关闭时等待（仅在配置为阻塞模式时） |
| `TCP_NODELAY` | 这将禁用 Nagle 算法 |
| `ServerSocketChannel` | `SO_RCVBUF` | 这是套接字接收缓冲区的大小 |
| `SO_REUSEADDR` | 这将重新使用地址 |
| `AsynchronousSocketChannel` | `SO_SNDBUF` | 这是套接字发送缓冲区的大小 |
| `SO_RCVBUF` | 这是套接字接收缓冲区的大小 |
| `SO_KEEPALIVE` | 这将保持连接活动 |
| `SO_REUSEADDR` | 这将重新使用地址 |
| `TCP_NODELAY` | 这将禁用 Nagle 算法 |

套接字选项是使用`setOption`方法进行配置的。以下代码示例了在“零件服务器”部分中使用的服务器套接字通道的此方法：

```java
    serverSocketChannel.setOption(SO_RCVBUF, 64);
```

第一个参数是`SocketOption<T>`接口的实例。该接口为选项定义了名称和类型方法。`StandardSocketOptions`类定义了一系列选项，这些选项实现了该接口。例如，`SO_RCVBUF`实例定义如下：

```java
    public static final SocketOption<Integer> SO_RCVBUF;
```

可能还有其他特定于实现的选项可用。

# 总结

在本章中，我们研究了 NIO 的通道和缓冲区类的使用。通道连接到外部源，并在缓冲区之间传输数据。我们举例说明了通道套接字，它们连接到网络中的另一个套接字。

缓冲区是数据的临时存储库。使用缓冲区可以顺序或随机访问数据。有许多缓冲区操作，这使得它成为许多应用程序的良好选择。

我们研究了几种类型的通道套接字，包括`SocketChannel`、`ServerSocketChannel`和`AsynchronousSocketChannel`类。`ServerSocketChannel`类支持服务器，并使用`accept`方法阻塞直到客户端请求连接。该方法将返回一个`SocketChannel`实例，该实例将连接到客户端的`SocketChannel`。`AsynchronousSocketChannel`和`AsynchronousSocketChannel`类支持异步通信，实现两个应用程序之间的非阻塞通信。还支持`DatagramChannel`，我们将在第六章中进行调查，*UDP 和组播*。

我们解释了缓冲区和通道类如何协同工作，并举例说明了它们在几个客户端/服务器应用程序中的使用。我们还研究了使用线程处理多个客户端的简单方法。

我们演示了如何在数组和缓冲区之间执行大容量数据传输。还研究了视图和只读缓冲区的使用。最后介绍了如何配置底层操作系统套接字支持。

在下一章中，我们将使用许多这些类和技术来支持其他客户端/服务器应用程序。
