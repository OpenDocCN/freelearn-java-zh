

# 第九章：并发策略和模型

今天的计算世界由分布式系统、基于云的架构以及由多核处理器加速的硬件组成。这些特性需要并发策略。本章提供了关于并发概念的基础信息，并提供了在 Java 程序中实现并发的实际操作机会。其根本目标是利用并发的优势和优点来提高我们的 Java 应用程序的性能。

本章首先回顾了不同的并发模型及其实际应用。概念包括基于线程的内存模型和消息传递模型。然后，我们将从理论角度和实际操作方面探讨多线程。将涵盖线程生命周期、线程池和其他相关主题。同步也将被涵盖，包括如何确保线程安全以及避免常见陷阱的策略。最后，本章介绍了非阻塞算法，这是一种高级并发策略，通过原子变量和特定数据结构来提高应用程序性能。

在本章中，我们将涵盖以下主要主题：

+   并发模型

+   多线程

+   同步

+   非阻塞算法

到本章结束时，您应该对并发及其相关策略和模型有深入的理解。您应该准备好在 Java 应用程序中实现并发，确保它们以高水平运行。

# 技术要求

要遵循本章中的示例和说明，您需要具备加载、编辑和运行 Java 代码的能力。如果您尚未设置您的开发环境，请参阅*第一章*。

本章的完整代码可以在以下位置找到：[`github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter09`](https://github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter09)。

# 并发模型

Java 编程语言最令人兴奋的方面之一是其健壮性。在解决并行执行挑战时，Java 支持多种模型，因此我们采取的方法取决于我们。通常，做事情的方法不止一种，每种可能的解决方案都既有优点也有缺点。我们的目标是创建运行效率高、可扩展且易于维护的 Java 应用程序。为此，我们将使用基于线程的并发方法（本章后面将详细介绍）。我们的选择基于其简单性。

**并发**在计算机科学中是指指令的并行执行。这是通过多线程（想想多任务处理）实现的。这种编程范式包括从多个线程访问 Java 对象和其他资源的能力。让我们看看三种特定的模型（基于线程、消息传递和反应式），然后比较它们，以确定在特定场景下哪个模型可能比其他模型更理想。

## 基于线程的模型

`Thread` 类以及 `Callable` 和 `Runnable` 接口。

让我们看看一个简单的实现示例。我们将实现 `increment` 方法，并用 `synchronized` 关键字标记它。这告诉 Java 在任何给定时间只执行一个线程：

```java
public class MyCounter {
  private int count = 0;
  public synchronized void increment() {
    count++;
  }
  public int getCount() {
    return count;
  }
```

以下代码段包含我们的 `main()` 方法。在这个方法中，我们创建了两个线程；它们都将增加我们的计数器：

```java
  public static void main(String[] args) throws InterruptedException {
    MyCounter counter = new MyCounter();
    Thread t1 = new Thread(() -> {
      for(int i = 0; i < 1000; i++) {
        counter.increment();
      }
    });
    Thread t2 = new Thread(() -> {
      for(int i = 0; i < 1000; i++) {
        counter.increment();
      }
    });
```

下一行代码启动了线程：

```java
    t1.start();
    t2.start();
```

在接下来的两行代码中，我们等待两个线程完成：

```java
    t1.join();
    t2.join();
```

最后，我们输出最终结果：

```java
    System.out.println("Final counter value: " + counter.getCount());
  }
}
```

基于线程的模型实现方式简单直接，这代表了一个巨大的优势。这种方法适用于较小的应用程序。使用此模型存在潜在的不利因素，因为当多个线程尝试访问共享的可变数据时，可能会引入**死锁**和**竞态条件**。

死锁和竞态条件

当两个线程等待对方释放所需资源时发生死锁。当需要线程执行的顺序时发生竞态条件。

在我们的应用程序中，应尽可能避免死锁和竞态条件。

## 消息传递模型

**消息传递模型**是一个有趣的模型，因为它避免了**共享状态**。此模型要求线程通过发送消息进行相互通信。

共享状态

当应用程序中的多个线程可以同时访问数据时存在共享状态。

消息传递模型提供了防止死锁和竞态条件的保证。此模型的一个好处是它促进了可伸缩性。

让我们看看如何实现消息传递模型。我们的示例包括一个简单的发送者和接收者场景。我们首先编写 `import` 语句，然后创建一个 `Message` 类：

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
public class MessagePassingExample {
  static class Message {
    private final String content;
    public Message(String content) {
      this.content = content;
    }
    public String getContent() {
      return content;
    }
  }
```

接下来，我们将让我们的 `Sender` 类实现 `Runnable` 接口：

```java
static class Sender implements Runnable {
  private final BlockingQueue<Message> queue;
  public Sender(BlockingQueue<Message> q) {
    this.queue = q;
  }
  @Override
  public void run() {
    // Sending messages
    String[] messages = {"First message", "Second message", "Third 
    message", "Done"};
    for (String m : messages) {
      try {
        Thread.sleep(1000); // Simulating work
        queue.put(new Message(m));
        System.out.println("Sent: " + m);
      } catch (InterruptedException e) {
          Thread.currentThread().interrupt();
      }
    }
  }
}
```

接下来，我们将让我们的 `Receiver` 类实现 `Runnable` 接口：

```java
static class Receiver implements Runnable {
  private final BlockingQueue<Message> queue;
  public Receiver(BlockingQueue<Message> q) {
    this.queue = q;
  }
  @Override
  public void run() {
    try {
      Message msg;
      // Receiving messages
      while (!((msg = queue.take()).getContent().equals("Done"))) {
        System.out.println("Received: " + msg.getContent());
        Thread.sleep(400); // Simulating work
      }
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
  }
}
```

最后一步是创建我们的 `main()` 方法：

```java
  public static void main(String[] args) {
    BlockingQueue<Message> queue = new ArrayBlockingQueue<>(10);
    Thread senderThread = new Thread(new Sender(queue));
    Thread receiverThread = new Thread(new Receiver(queue));
    senderThread.start();
    receiverThread.start();
  }
}
```

我们的示例将 `Sender` 和 `Receiver` 实现为 `Runnable` 类。它们使用 `BlockingQueue` 进行通信。队列用于 `Sender` 添加消息，`Receiver` 用于接收和处理它们。`Sender` 向队列发送 `Done`，以便 `Receiver` 知道何时停止处理。消息传递模型通常用于分布式系统，因为它支持高度可伸缩的系统。

## 反应式模型

**响应式模型** 比我们之前讨论的最后两个模型更新。它的重点是 **非阻塞**、**事件驱动编程**。此模型通常体现在处理大量输入/输出操作的大规模系统中，尤其是在需要高可伸缩性时。我们可以使用外部库来实现此模型，包括 **Project Reactor** 和 **RxJava**。

让我们来看一个使用 **Project Reactor** 的简单实现示例。我们首先将 Project Reactor **依赖项** 添加到我们的项目中。以下是使用 **Maven** 作为构建工具时的样子：

```java
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.4.0</version>
</dependency>
```

以下示例演示了如何创建响应式流来处理一系列事件：

```java
import reactor.core.publisher.Flux;
public class ReactiveExample {
    public static void main(String[] args) {
        Flux<String> messageFlux = Flux.just("Hello", "Reactive", 
        "World", "with", "Java")
                .map(String::toUpperCase)
                .filter(s -> s.length() > 4);
        messageFlux.subscribe(System.out::println);
    }
}
```

响应式模型提供了高效的资源使用、避免阻塞操作和对异步编程的独特方法。然而，与我们所讨论的其他模型相比，它可能更难实现。

比较分析

这三个并发模型各自提供了不同的好处，了解它们的个性和差异可以帮助您做出明智的决定，选择采用哪种模型。

# 多线程

**多线程**简单地说就是程序中两个或更多部分的**同步执行**，或**并发执行**，它是 Java 并发编程机制的基本方面。我们执行程序的多部分，利用多核**中央处理器**（**CPU**）资源来优化我们应用程序的性能。

在我们深入探讨多线程之前，让我们专注于创建和启动线程的单个 `Thread` 类：

```java
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread executed by extending Thread 
        class.");
    }
}
// This is how we create and start a thread.
MyThread thread = new MyThread();
thread.start();
```

下面的代码片段演示了如何实现 `Runnable` 接口：

```java
class MyRunnable implements Runnable {
    public void run() {
        System.out.println("Thread executed by implementing Runnable 
        interface.");
    }
}
// Here, we create and start a thread.
Thread thread = new Thread(new MyRunnable());
thread.start();
```

现在您已经了解了创建和启动线程是多么容易，让我们来检查它们的生命周期。

## 线程生命周期

Java 线程有一个明确的开始和结束状态。它们还有额外的状态，如下面的图中所示。

![图 9.1 – Java 线程生命周期](img/B21942_09_1.jpg)

图 9.1 – Java 线程生命周期

理解这些状态非常重要，这样我们才能有效地管理我们的线程。让我们简要地看看 Java 线程生命周期中的每个状态：

1.  **新建**：线程处于此状态时，我们创建了它们但尚未启动它们。

1.  **可运行**：当线程正在执行时，存在此状态。已启动并等待 CPU 时间的线程也有此状态。

1.  **阻塞**：线程被阻止访问资源。

1.  **等待/定时等待**：线程可以等待其他线程。有时，可能会有特定的等待时间，而在其他时候，等待可能是无限期的。

1.  **终止**：线程在执行完成后处于此状态。

对于依赖于线程通信和同步的应用程序，理解这些状态非常重要。

## 多线程最佳实践

在处理多线程时，有一些事情我们应该注意，以确保我们的应用程序按预期运行，并且我们的线程是安全的。

首先，我们希望确保每个资源一次只被一个线程访问，使用 `java.util.concurrent` 包，它包括我们可以用于同步的并发数据结构和方法。利用这个包可以帮助我们实现线程安全。

Java 的 `Object` 类包括 `wait()`、`notify()` 和 `notifyAll()` 方法，这些方法可以用来使 Java 线程能够相互通信。以下示例应用程序演示了这些方法如何使用。我们的示例包含一个 `Producer` 线程，它为 `Consumer` 线程创建一个值。我们不希望这两个操作同时进行；事实上，我们希望 `Consumer` 等待 `Producer` 创建值。此外，`Producer` 必须等待 `Consumer` 接收到最后一个值后才能创建新的值。第一部分定义了我们的 `WaitNotifyExample` 类：

```java
Public class WaitNotifyExample {
  private static class SharedResource {
    private String message;
    private boolean empty = true;
    public synchronized String take() {
      // Wait until the message is available.
      while (empty) {
        try {
          wait();
        } catch (InterruptedException e) {}
      }
      // Toggle status to true.
      empty = true;
      // Notify producer that status has changed.
      notifyAll();
      return message;
    }
    public synchronized void put(String message) {
      // Wait until the message has been retrieved.
      while (!empty) {
        try {
          wait();
        } catch (InterruptedException e) {}
      }
      // Toggle the status to false.
      empty = false;
      // Store the message.
      this.message = message;
      // Notify that consumer that the status has changed.
      notifyAll();
    }
  }
  private static class Producer implements Runnable {
    private SharedResource resource;
    public Producer(SharedResource resource) {
      this.resource = resource;
    }
    public void run() {
      String[] messages = {"Hello", "World", "Java", "Concurrency"};
      for (String message : messages) {
        resource.put(message);
        System.out.println("Produced: " + message);
        try {
          Thread.sleep(1000); // Simulate time passing
        } catch (InterruptedException e) {}
      }
      resource.put("DONE");
    }
  }
```

接下来，我们需要创建我们的 `Consumer` 类并实现 `Runnable` 接口：

```java
  private static class Consumer implements Runnable {
    private SharedResource resource;
    public Consumer(SharedResource resource) {
      this.resource = resource;
    }
    public void run() {
      for (String message = resource.take(); !message.equals("DONE"); 
      message = resource.take()) {
        System.out.println("Consumed: " + message);
        try {
          Thread.sleep(1000); // Simulate time passing
        } catch (InterruptedException e) {}
      }
    }
  }
```

我们应用程序的最后一部分是 `main()` 类：

```java
  public static void main(String[] args) {
    SharedResource resource = new SharedResource();
    Thread producerThread = new Thread(new Producer(resource));
    Thread consumerThread = new Thread(new Consumer(resource));
    producerThread.start();
    consumerThread.start();
  }
}
```

我们的应用程序输出如下所示：

```java
Produced: Hello
Consumed: Hello
Produced: World
Consumed: World
Produced: Java
Consumed: Java
Produced: Concurrency
Consumed: Concurrency
```

当我们遵循本节提供的最佳实践时，我们增加了拥有高效多线程的机会，从而有助于构建高性能的 Java 应用程序。

# 同步

**同步** 是我们在寻求完全理解并发时应该掌握的另一个关键 Java 概念。正如我们之前指出的，我们使用同步来避免 **竞态条件**。

竞态条件

当多个线程同时尝试修改共享资源时的条件。这种情况的结果是不可预测的，应该避免。

让我们通过查看几个代码片段来了解如何在我们的 Java 应用程序中实现同步。首先，我们展示了如何在方法声明中添加 `synchronized` 关键字。这样我们就可以确保一次只有一个线程可以执行特定对象上的方法：

```java
public class Counter {
    private int count = 0;
     public synchronized void increment() {
        count++;
    }
     public synchronized int getCount() {
        return count;
    }
}
```

我们还可以实现 **synchronized 块**，它是方法的一个子集。这种粒度级别允许我们在不需要锁定整个方法的情况下同步一个块：

```java
public void increment() {
    synchronized(this) {
        count++;
    }
}
```

Java 还包括一个 `Lock` 接口，可以用于更精细的资源锁定方法。以下是实现方法：

```java
import java.util.concurrent.locks.ReentrantLock;
public class Counter {
    private final ReentrantLock lock = new ReentrantLock();
    private int count = 0;
    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();
        }
    }
}
```

Java 还包括 `volatile` 关键字，我们可以用它来告诉 Java，特定的变量可能被多个线程修改。当我们用这个关键字声明变量时，Java 将变量的值放置在一个所有线程都可以访问的内存位置：

```java
public class Flag {
    private volatile boolean flag = true;
    public boolean isFlag() {
        return flag;
    }
    public void setFlag(boolean flag) {
        this.flag = flag;
    }
}
```

如你无疑将理解的那样，同步对于 Java 中成功的并发编程至关重要。

# 非阻塞算法

作为并发编程的最后一个概念，让我们来看看`同步`方法和同步块。非阻塞算法有三种类型——**无锁**、**无等待**和**无阻塞**。尽管它们的名称具有自我描述性，但让我们更深入地了解一下。

现代 CPU 支持原子操作，Java 包含了一些我们可以在实现非阻塞算法时使用的原子类。

原子操作

这些是现代 CPU 作为单一、有限的步骤执行的操作，确保一致性而无需锁。

这里有一个代码片段，说明了如何使用`AtomicInteger`：

```java
import java.util.concurrent.atomic.AtomicInteger;
public class Counter {
    private AtomicInteger count = new AtomicInteger(0);
    public void increment() {
        count.incrementAndGet();
    }
    public int getCount() {
        return count.get();
    }
}
```

以下示例演示了如何实现一个非阻塞栈。正如你所看到的，我们的栈使用原子引用，这确保了线程安全：

```java
import java.util.concurrent.atomic.AtomicReference;
class Node<T> {
    final T item;
    Node<T> next;
    Node(T item) {
        this.item = item;
    }
}
public class ConcurrentStack<T> {
    AtomicReference<Node<T>> top = new AtomicReference<>();
    public void push(T item) {
        Node<T> newHead = new Node<>(item);
        Node<T> oldHead;
        do {
            oldHead = top.get();
            newHead.next = oldHead;
        } while (!top.compareAndSet(oldHead, newHead));
    }
    public T pop() {
        Node<T> oldHead;
        Node<T> newHead;
        do {
            oldHead = top.get();
            if (oldHead == null) {
                return null;
            }
            newHead = oldHead.next;
        } while (!top.compareAndSet(oldHead, newHead));
        return oldHead.item;
    }
}
```

当我们使用非阻塞算法时，我们可以获得性能优势，尤其是在我们的应用程序处理高并发时。这些优势被代码复杂性所抵消，这可能导致错误和更难维护的代码。

# 摘要

本章重点介绍了并发策略和模型，旨在深入探讨并发概念、不同的模型和策略，以及一些实现示例。我们探讨了理论概念和实践示例。涵盖的概念包括并发模型、同步和非阻塞算法。你现在应该具备足够的知识来开始尝试编写代码。

在下一章中，我们将探讨连接池，具体包括概念、实现和最佳实践。你将有机会学习如何创建和维护数据库连接对象的缓存，以帮助提高你的 Java 应用程序的性能。
