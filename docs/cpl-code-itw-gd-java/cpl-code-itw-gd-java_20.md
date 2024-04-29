# 第十六章：并发

开发单线程的 Java 应用程序很少可行。因此，大多数项目将是多线程的（即它们将在多线程环境中运行）。这意味着，迟早，您将不得不解决某些多线程问题。换句话说，您将不得不动手编写直接或通过专用 API 操纵 Java 线程的代码。

本章涵盖了关于 Java 并发（多线程）的最常见问题，这些问题在关于 Java 语言的一般面试中经常出现。和往常一样，我们将从简要介绍开始，介绍 Java 并发的主要方面。因此，我们的议程很简单，涵盖以下主题：

+   Java 并发（多线程）简介

+   问题和编码挑战

让我们从我们的主题 Java 并发的基本知识开始。使用以下简介部分提取一些关于并发的基本问题的答案，比如*什么是并发？*，*什么是 Java 线程？*，*什么是多线程？*等。

# 技术要求

本章中使用的代码可以在 GitHub 上找到：[`github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter16`](https://github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter16)

# Java 并发（多线程）简介

我们的计算机可以同时运行多个*程序*或*应用程序*（例如，我们可以同时在媒体播放器上听音乐并浏览互联网）。*进程*是程序或应用程序的执行实例（例如，通过在计算机上双击 NetBeans 图标，您启动将运行 NetBeans 程序的进程）。此外，*线程*是*轻量级子进程*，表示进程的最小可执行工作单元。Java 线程的开销相对较低，并且它与其他线程共享公共内存空间。一个进程可以有多个线程，其中一个是*主线程*。

重要说明

进程和线程之间的主要区别在于线程共享公共内存空间，而进程不共享。通过共享内存，线程减少了大量开销。

*并发*是应用程序处理其工作的多个任务的能力。程序或应用程序可以一次处理一个任务（*顺序处理*）或同时处理多个任务（*并发处理*）。

不要将并发与*并行*混淆。*并行*是应用程序处理每个单独任务的能力。应用程序可以串行处理每个任务，也可以将任务分割成可以并行处理的子任务。

重要说明

并发是关于**处理**（而不是执行）多个事情，而并行是关于**执行**多个事情。

通过*多线程*实现并发。*多线程*是一种技术，使程序或应用程序能够同时处理多个任务，并同步这些任务。这意味着多线程允许通过在同一时间执行两个或更多任务来最大程度地利用 CPU。我们在这里说*在同一时间*是因为这些任务看起来像是同时运行；然而，实质上，它们不能这样做。它们利用操作系统的 CPU *上下文切换*或*时间片*功能。换句话说，CPU 时间被所有运行的任务共享，并且每个任务被安排在一定时间内运行。因此，多线程是*多任务处理*的关键。

重要说明

在单核 CPU 上，我们可以实现并发但*不是*并行。

总之，线程可以产生多任务的错觉；然而，在任何给定的时间点，CPU 只执行一个线程。CPU 在线程之间快速切换控制，从而产生任务并行执行（或推进）的错觉。实际上，它们是并发执行的。然而，随着硬件技术的进步，现在普遍拥有多核机器和计算机。这意味着应用程序可以利用这些架构，并且每个线程都有一个专用的 CPU 在运行。

以下图表通过四个线程（**T1**、**T2**、**T3**和**T4**）澄清了并发和并行之间的混淆：

16.1-并发与并行

](img/Figure_16.1_B15403.jpg)

16.1-并发与并行

因此，一个应用程序可以是以下之一：

+   并发但不是并行：它同时执行多个任务，但没有两个任务同时执行。

+   并行但不是并发：它在多核 CPU 中同时执行一个任务的多个子任务。

+   既不是并行也不是并发：它一次执行所有任务（顺序执行）。

+   并行和并发：它在多核 CPU 中同时并发执行多个任务。

被分配执行任务的一组同质工作线程称为*线程池*。完成任务的工作线程将返回到池中。通常，线程池绑定到任务队列，并且可以调整到它们持有的线程的大小。通常情况下，为了获得最佳性能，线程池的大小应等于 CPU 核心的数量。

在多线程环境中*同步*是通过*锁定*实现的。锁定用于在多线程环境中协调和限制对资源的访问。

如果多个线程可以访问相同的资源而不会导致错误或不可预测的行为/结果，那么我们处于*线程安全的上下文*。可以通过各种同步技术（例如 Java `synchronized`关键字）实现*线程安全*。

接下来，让我们解决一些关于 Java 并发的问题和编码挑战。

# 问题和编码挑战

在本节中，我们将涵盖 20 个关于并发的问题和编码挑战，这在面试中非常流行。

您应该知道，Java 并发是一个广泛而复杂的主题，任何 Java 开发人员都需要详细了解。对 Java 并发的基本见解应该足以通过一般的 Java 语言面试，但对于特定的面试来说还不够（例如，如果您申请一个将涉及开发并发 API 的工作，那么您必须深入研究这个主题并学习高级概念-很可能，面试将以并发为中心）。

## 编码挑战 1-线程生命周期状态

`线程`。

`Thread.State`枚举。Java 线程的可能状态可以在以下图表中看到：

![16.2-Java 线程状态](img/Figure_16.2_B15403.jpg)

16.2-Java 线程状态

Java `Thread`的不同生命周期状态如下：

+   `NEW` `Thread#start()`方法被调用）。

+   `RUNNABLE` `Thread#start()`方法，线程从`NEW`到`RUNNABLE`。在`RUNNABLE`状态下，线程可以运行或准备运行。等待**JVM**（Java 虚拟机）线程调度程序分配必要的资源和时间来运行的线程是准备运行的，但尚未运行。一旦 CPU 可用，线程调度程序将运行线程。

+   `BLOCKED` `BLOCKED`状态。例如，如果一个线程*t1*试图进入另一个线程*t2*已经访问的同步代码块（例如，标记为`synchronized`的代码块），那么*t1*将被保持在`BLOCKED`状态，直到它可以获取所需的锁。

+   `WAITING` `WAITING`状态。

+   `TIMED WAITING` `TIMED_WAITING`状态。

+   `TERMINATED` `TERMINATE`状态。

除了描述 Java 线程的可能状态之外，面试官可能会要求您为每个状态编写一个示例。这就是为什么我强烈建议您花时间分析名为*ThreadLifecycleState*的应用程序（为简洁起见，书中未列出代码）。该应用程序的结构非常直观，主要注释解释了每种情景/状态。

## 编码挑战 2 - 死锁

**问题**：向我们解释一下死锁，我们会雇佣你！

**解决方案**：雇佣我，我会向您解释。

在这里，我们刚刚描述了一个死锁。

死锁可以这样解释：线程*T1*持有锁*P*，并尝试获取锁*Q*。与此同时，有一个线程*T2*持有锁*Q*，并尝试获取锁*P*。这种死锁被称为*循环等待*或*致命拥抱*。

Java 不提供死锁检测和/或解决机制（例如数据库有）。这意味着死锁对应用程序来说可能非常尴尬。死锁可能部分或完全阻塞应用程序。这导致显著的性能惩罚，意外的行为/结果等。通常，死锁很难找到和调试，并且会迫使您重新启动应用程序。

避免竞争死锁的最佳方法是避免使用嵌套锁或不必要的锁。嵌套锁很容易导致死锁。

模拟死锁的常见问题是**哲学家就餐**问题。您可以在*Java 编码问题*书中找到对这个问题的详细解释和实现（[`www.packtpub.com/programming/java-coding-problems`](https://www.packtpub.com/programming/java-coding-problems)）。*Java 编码问题*包含两章专门讨论 Java 并发，并旨在使用特定问题深入探讨这个主题。

在本书的代码包中，您可以找到一个名为*Deadlock*的死锁示例。

## 编码挑战 3 - 竞争条件

**问题**：解释一下*竞争条件*是什么。

**解决方案**：首先，我们必须提到可以由多个线程执行（即并发执行）并公开共享资源（例如共享数据）的代码片段/块被称为*关键部分*。

*竞争条件*发生在线程在没有线程同步的情况下通过这样的关键部分。线程在关键部分中*竞争*尝试读取/写入共享资源。根据线程完成这场竞赛的顺序，应用程序的输出会发生变化（应用程序的两次运行可能会产生不同的输出）。这导致应用程序的行为不一致。

避免竞争条件的最佳方法是通过使用锁、同步块、原子/易失性变量、同步器和/或消息传递来正确同步关键部分。

编码挑战 4 - 可重入锁

**问题**：解释什么是*可重入锁*概念。

**解决方案**：一般来说，*可重入锁*指的是一个进程可以多次获取锁而不会使自身陷入死锁的过程。如果锁不是可重入的，那么进程仍然可以获取它。但是，当进程尝试再次获取锁时，它将被阻塞（死锁）。可重入锁可以被另一个线程获取，或者被同一个线程递归地获取。

可重入锁可以用于不包含可能破坏它的更新的代码片段。如果代码包含可以更新的共享状态，那么再次获取锁将会破坏共享状态，因为在执行代码时调用了该代码。

在 Java 中，可重入锁是通过`ReentrantLock`类实现的。可重入锁的工作方式是：当线程第一次进入锁时，保持计数设置为 1。在解锁之前，线程可以重新进入锁，导致每次进入时保持计数增加一。每个解锁请求将保持计数减少一，当保持计数为零时，锁定的资源被打开。

## 编码挑战 5 - Executor 和 ExecutorService

`Executor`和`ExecutorService`？

在`java.util.concurrent`包中，有许多专用于执行任务的接口。最简单的一个被命名为`Executor`。这个接口公开了一个名为`execute (Runnable command)`的方法。

一个更复杂和全面的接口，提供了许多额外的方法，是`ExecutorService`。这是`Executor`的增强版本。Java 带有一个完整的`ExecutorService`实现，名为`ThreadPoolExecutor`。

在本书的代码包中，您可以找到在名为*ExecutorAndExecutorService*的应用程序中使用`Executor`和`ThreadPoolExecutor`的简单示例。

编码挑战 6 - Runnable 与 Callable 的比较

`Callable`接口和`Runnable`接口？

`Runnable`接口是一个包含一个名为`run()`的方法的函数接口。`run()`方法不接受任何参数，返回`void`。此外，它不能抛出已检查的异常（只能抛出`RuntimeException`）。这些陈述使`Runnable`适用于我们不寻找线程执行结果的情况。`run()`签名如下：

```java
void run()
```

另一方面，`Callable`接口是一个包含一个名为`call()`的方法的函数接口。`call()`方法返回一个通用值，并且可以抛出已检查的异常。通常，`Callable`用于`ExecutorService`实例。它用于启动异步任务，然后调用返回的`Future`实例来获取其值。`Future`接口定义了用于获取`Callable`对象生成的结果和管理其状态的方法。`call()`签名如下：

```java
V call() throws Exception
```

请注意，这两个接口都代表一个任务，该任务旨在由单独的线程并发执行。

在本书的代码包中，您可以找到在名为*RunnableAndCallable*的应用程序中使用`Runnable`和`Callable`的简单示例。

## 编码挑战 7 - 饥饿

**问题**：解释什么是线程*饥饿*。

**解决方案**：一个永远（或很少）得不到 CPU 时间或访问共享资源的线程是经历*饥饿*的线程。由于它无法定期访问共享资源，这个线程无法推进其工作。这是因为其他线程（所谓的*贪婪*线程）在这个线程之前获得访问，并使资源长时间不可用。

避免线程饥饿的最佳方法是使用*公平*锁，比如 Java 的`ReentrantLock`。*公平*锁授予等待时间最长的线程访问权限。通过 Java 的`Semaphore`可以实现多个线程同时运行而避免饥饿。*公平*`Semaphore`使用 FIFO 来保证在争用情况下授予许可。

编码挑战 8 - 活锁

**问题**：解释什么是线程*活锁*。

**解决方案**：当两个线程不断采取行动以响应另一个线程时，就会发生活锁。这些线程不会在自己的工作中取得任何进展。请注意，这些线程没有被阻塞；它们都忙于相互响应而无法恢复工作。

这是一个活锁的例子：想象两个人试图在走廊上互相让对方通过。马克向右移动让奥利弗通过，奥利弗向左移动让马克通过。现在他们互相阻塞。马克看到自己挡住了奥利弗，向左移动，奥利弗看到自己挡住了马克，向右移动。他们永远无法互相通过并一直阻塞对方。

我们可以通过 `ReentrantLock` 避免活锁。这样，我们可以确定哪个线程等待的时间最长，并为其分配一个锁。如果一个线程无法获取锁，它应该释放先前获取的锁，然后稍后再试。

编码挑战 9 – Start() 与 run()

Java `Thread` 中的 `start()` 方法和 `run()` 方法。

`start()` 和 `run()` 的区别在于 `start()` 方法创建一个新的线程，而 `run()` 方法不会。`start()` 方法创建一个新的线程，并调用在这个新线程中写的 `run()` 方法内的代码块。`run()` 方法在同一个线程上执行该代码（即调用线程），而不创建新线程。

另一个区别是在线程对象上两次调用 `start()` 将抛出 `IllegalStateException`。另一方面，两次调用 `run()` 方法不会导致异常。

通常，新手会忽略这些区别，并且，由于 `start()` 方法最终调用 `run()` 方法，他们认为没有理由调用 `start()` 方法。因此，他们直接调用 `run()` 方法。

## 编码挑战 10 – 线程与可运行

`Thread` 或实现 `Runnable`？

通过 `java.lang.Thread` 或实现 `java.lang.Runnable`。首选的方法是实现 `Runnable`。

大多数情况下，我们实现一个线程只是为了让它运行一些东西，而不是覆盖 `Thread` 的行为。只要我们想要给一个线程运行一些东西，我们肯定应该坚持实现 `Runnable`。事实上，使用 `Callable` 或 `FutureTask` 更好。

此外，通过实现 `Runnable`，你仍然可以扩展另一个类。通过扩展 `Thread`，你不能扩展另一个类，因为 Java 不支持多重继承。

最后，通过实现 `Runnable`，我们将任务定义与任务执行分离。

编码挑战 11 – CountDownLatch 与 CyclicBarrier

`CountDownLatch` 和 `CyclicBarrier`。

`CountDownLatch` 和 `CyclicBarrier` 是 Java *同步器* 中的五个之一，另外还有 `Exchanger`、`Semaphore` 和 `Phaser`。

`CountDownLatch` 和 `CyclicBarrier` 之间的主要区别在于 `CountDownLatch` 实例在倒计时达到零后无法重用。另一方面，`CyclicBarrier` 实例是可重用的。`CyclicBarrier` 实例是循环的，因为它可以被重置和重用。要做到这一点，在所有等待在屏障处的线程被释放后调用 `reset()` 方法；否则，将抛出 `BrokenBarrierException`。

## 编码挑战 12 – wait() 与 sleep()

`wait()` 方法和 `sleep()` 方法。

`wait()` 方法和 `sleep()` 方法的区别在于 `wait()` 必须从同步上下文（例如，从 `synchronized` 方法）中调用，而 `sleep()` 方法不需要同步上下文。从非同步上下文调用 `wait()` 将抛出 `IllegalMonitorStateException`。

此外，重要的是要提到 `wait()` 在 `Object` 上工作，而 `sleep()` 在当前线程上工作。实质上，`wait()` 是在 `java.lang.Object` 中定义的非`static`方法，而 `sleep()` 是在 `java.lang.Thread` 中定义的`static`方法。

此外，`wait()` 方法释放锁，而 `sleep()` 方法不释放锁。`sleep()` 方法只是暂停当前线程一段时间。它们都会抛出 `IntrupptedException` 并且可以被中断。

最后，应该在决定何时释放锁的循环中调用`wait()`方法。另一方面，不建议在循环中调用`sleep()`方法。

编码挑战 13 - ConcurrentHashMap 与 Hashtable

`ConcurrentHashMap`比`Hashtable`快吗？

`ConcurrentHashMap`比`Hashtable`更快，因为它具有特殊的内部设计。`ConcurrentHashMap`在内部将映射分成段（或桶），并且在更新操作期间仅锁定特定段。另一方面，`Hashtable`在更新操作期间锁定整个映射。因此，`Hashtable`对整个数据使用单个锁，而`ConcurrentHashMap`对不同段（桶）使用多个锁。

此外，使用`get()`从`ConcurrentHashMap`中读取是无锁的（无锁），而所有`Hashtable`操作都是简单的`synchronized`。

## 编码挑战 14 - ThreadLocal

`ThreadLocal`？

`ThreadLocal`用作分别存储和检索每个线程的值的手段。单个`ThreadLocal`实例可以存储和检索多个线程的值。如果线程*A*存储*x*值，线程*B*在同一个`ThreadLocal`实例中存储*y*值，那么后来线程*A*检索*x*值，线程*B*检索*y*值。Java `ThreadLocal`通常用于以下两种情况：

1.  为每个线程提供实例（线程安全和内存效率）

1.  为每个线程提供上下文

## 编码挑战 15 - submit()与 execute()

`ExecutorService#submit()`和`Executor#execute()`方法。

用于执行的`Runnable`任务，它们并不相同。主要区别可以通过简单检查它们的签名来观察。注意，`submit()`返回一个结果（即代表任务的`Future`对象），而`execute()`返回`void`。返回的`Future`对象可以用于在以后（过早地）以编程方式取消运行的线程。此外，通过使用`Future#get()`方法，我们可以等待任务完成。如果我们提交一个`Callable`，那么`Future#get()`方法将返回调用`Callable#call()`方法的结果。

## 编码挑战 16 - interrupted()和 isInterrupted()

`interrupted()`和`isInterrupted()`方法。

`Thread.interrupt()`方法中断当前线程并将此标志设置为`true`。

`interrupted()`和`isInterrupted()`方法之间的主要区别在于`interrupted()`方法会清除中断状态，而`isInterrupted()`不会。

如果线程被中断，则`Thread.interrupted()`将返回`true`。但是，除了测试当前线程是否被中断外，`Thread.interrupted()`还会清除线程的中断状态（即将其设置为`false`）。

非`static isInterrupted()`方法不会更改中断状态标志。

作为一个经验法则，在捕获`InterruptedException`后，不要忘记通过调用`Thread.currentThread().interrupt()`来恢复中断。这样，我们的代码的调用者将意识到中断。

编码挑战 17 - 取消线程

**问题**：如何停止或取消线程？

`volatile`（也称为轻量级同步机制）。作为`volatile`标志，它不会被线程缓存，并且对它的操作不会在内存中重新排序；因此，线程无法看到旧值。读取`volatile`字段的任何线程都将看到最近写入的值。这正是我们需要的，以便将取消操作通知给所有对此操作感兴趣的运行中的线程。以下图表说明了这一点：

![16.3 - Volatile 标志读/写](img/Figure_16.3_B15403.jpg)

16.3 - Volatile 标志读/写

请注意，`volatile`变量不适合读-修改-写场景。对于这种场景，我们将依赖原子变量（例如`AtomicBoolean`、`AtomicInteger`和`AtomicReference`）。

在本书的代码包中，您可以找到一个取消线程的示例。该应用程序名为*CancelThread*。

## 编码挑战 18 - 在线程之间共享数据

问题：如何在两个线程之间共享数据？

`BlockingQueue`，`LinkedBlockingQueue`和`ConcurrentLinkedDeque`。依赖于这些数据结构在线程之间共享数据非常方便，因为您不必担心线程安全和线程间通信。

编码挑战 19 - ReadWriteLock

`ReadWriteLock`是在 Java 中的。

`ReadWriteLock`用于在并发环境中维护读写操作的效率和线程安全性。它通过*锁分段*的概念实现这一目标。换句话说，`ReadWriteLock`为读和写使用单独的锁。更确切地说，`ReadWriteLock`保持一对锁：一个用于只读操作，一个用于写操作。只要没有写线程，多个读线程可以同时持有读锁（共享悲观锁）。一个写线程可以一次写入（独占/悲观锁）。因此，`ReadWriteLock`可以显著提高应用程序的性能。

除了`ReadWriteLock`，Java 还提供了`ReentrantReadWriteLock`和`StampedLock`。`ReentrantReadWriteLock`类将*可重入锁*概念（参见*编码挑战 4*）添加到`ReadWriteLock`中。另一方面，`StampedLock`比`ReentrantReadWriteLock`表现更好，并支持乐观读取。但它不是*可重入*的；因此，它容易发生死锁。

## 编码挑战 20 - 生产者-消费者

**问题**：为著名的生产者-消费者问题提供一个实现。

注意

这是任何 Java 多线程面试中的一个常见问题！

**解决方案**：生产者-消费者问题是一个可以表示为以下形式的设计模式：

![16.4 - 生产者-消费者设计模式](img/Figure_16.4_B15403.jpg)

16.4 - 生产者-消费者设计模式

在这种模式中，生产者线程和消费者线程通常通过一个队列进行通信（生产者将数据入队，消费者将数据出队），并遵循特定于建模业务的一组规则。这个队列被称为*数据缓冲区*。当然，根据流程设计，其他数据结构也可以扮演数据缓冲区的角色。

现在，让我们假设以下情景（一组规则）：

+   如果数据缓冲区为空，那么生产者会生产一个产品（将其添加到数据缓冲区）。

+   如果数据缓冲区不为空，那么消费者会消费一个产品（从数据缓冲区中移除它）。

+   只要数据缓冲区不为空，生产者就会等待。

+   只要数据缓冲区为空，消费者就会等待。

接下来，让我们通过两种常见的方法解决这种情况。我们将从基于`wait()`和`notify()`方法的解决方案开始。

## 通过`wait()`和`notify()`实现生产者-消费者

一些面试官可能会要求您实现`wait()`和`notify()`方法。换句话说，他们不允许您使用内置的线程安全队列，如`BlockingQueue`。

例如，让我们考虑数据缓冲区（`queue`）由`LinkedList`表示，即非线程安全的数据结构。为了确保生产者和消费者以线程安全的方式访问这个共享的`LinkedList`，我们依赖于`Synchronized`关键字。

### 生产者

如果队列不为空，那么生产者会等待，直到消费者完成。为此，生产者依赖于`wait()`方法，如下所示：

```java
synchronized (queue) {     
  while (!queue.isEmpty()) {
    logger.info("Queue is not empty ...");
    queue.wait();
  }
}
```

另一方面，如果队列为空，那么生产者会将一个产品入队，并通过`notify()`通知消费者线程，如下所示：

```java
synchronized (queue) {
  String product = "product-" + rnd.nextInt(1000);
  // simulate the production time
  Thread.sleep(rnd.nextInt(MAX_PROD_TIME_MS)); 
  queue.add(product);
  logger.info(() -> "Produced: " + product);
  queue.notify();
}
```

在将产品添加到队列后，消费者应该准备好消费它。

### 消费者

如果队列为空，那么消费者会等待，直到生产者完成。为此，生产者依赖于`wait()`方法，如下所示：

```java
synchronized (queue) {
  while (queue.isEmpty()) {
    logger.info("Queue is empty ...");
    queue.wait();
  }
}
```

另一方面，如果队列不为空，则消费者将出列一个产品并通过`notify()`通知生产者线程，如下所示：

```java
synchronized (queue) {
  String product = queue.remove(0);
  if (product != null) {
    // simulate consuming time
    Thread.sleep(rnd.nextInt(MAX_CONS_TIME_MS));                                
    logger.info(() -> "Consumed: " + product);
    queue.notify();
  }
}
```

完整的代码在捆绑代码*ProducerConsumerWaitNotify*中可用。

通过内置的阻塞队列进行生产者-消费者

如果您可以使用内置的阻塞队列，那么您可以选择`BlockingQueue`甚至`TransferQueue`。它们两者都是线程安全的。在下面的代码中，我们使用了`TransferQueue`，更确切地说是`LinkedTransferQueue`。

### 生产者

生产者等待消费者通过`hasWaitingConsumer()`可用：

```java
while (queue.hasWaitingConsumer()) {
  String product = "product-" + rnd.nextInt(1000);
  // simulate the production time
  Thread.sleep(rnd.nextInt(MAX_PROD_TIME_MS)); 
  queue.add(product);
  logger.info(() -> "Produced: " + product);
}
```

在将产品添加到队列后，消费者应准备好消费它。

### 消费者

消费者使用`poll()`方法并设置超时来提取产品：

```java
// MAX_PROD_TIME_MS * 2, just give enough time to the producer
String product = queue.poll(
  MAX_PROD_TIME_MS * 2, TimeUnit.MILLISECONDS);
if (product != null) {
  // simulate consuming time
  Thread.sleep(rnd.nextInt(MAX_CONS_TIME_MS));                         
  logger.info(() -> "Consumed: " + product);
}
```

完整的代码在捆绑代码*ProducerConsumerQueue*中可用

总结

在本章中，我们涵盖了在 Java 多线程面试中经常出现的最受欢迎的问题。然而，Java 并发是一个广泛的主题，深入研究它非常重要。我强烈建议您阅读 Brian Goetz 的*Java 并发实践*。这对于任何 Java 开发人员来说都是必读之书。

在下一章中，我们将涵盖一个热门话题：Java 函数式编程。
