# 第三章：线程同步工具

在本章中，我们将涵盖：

+   控制对资源的并发访问

+   控制对多个资源副本的并发访问

+   等待多个并发事件

+   在一个公共点同步任务

+   运行并发分阶段任务

+   控制并发分阶段任务中的阶段变化

+   在并发任务之间交换数据

# 介绍

在第二章，*基本线程同步*，我们学习了同步和关键部分的概念。基本上，当多个并发任务共享一个资源时，例如一个对象或对象的属性时，我们谈论同步。访问这个共享资源的代码块被称为关键部分。

如果不使用适当的机制，可能会出现错误的结果、数据不一致或错误条件，因此我们必须采用 Java 语言提供的同步机制之一来避免所有这些问题。

第二章，*基本线程同步*，教会了我们以下基本同步机制：

+   `同步`关键字

+   `Lock`接口及其实现类：`ReentrantLock`、`ReentrantReadWriteLock.ReadLock`和`ReentrantReadWriteLock.WriteLock`

在本章中，我们将学习如何使用高级机制来实现多个线程的同步。这些高级机制如下：

+   **信号量**：信号量是控制对一个或多个共享资源的访问的计数器。这种机制是并发编程的基本工具之一，并且大多数编程语言都提供了它。

+   **CountDownLatch**：`CountDownLatch`类是 Java 语言提供的一种机制，允许线程等待多个操作的完成。

+   **CyclicBarrier**：`CyclicBarrier`类是 Java 语言提供的另一种机制，允许多个线程在一个公共点同步。

+   **Phaser**：`Phaser`类是 Java 语言提供的另一种机制，用于控制分阶段并发任务的执行。所有线程必须在继续下一个阶段之前完成一个阶段。这是 Java 7 API 的一个新特性。

+   **Exchanger**：`Exchanger`类是 Java 语言提供的另一种机制，提供了两个线程之间的数据交换点。

信号量是一种通用的同步机制，您可以用它来保护任何问题中的关键部分。其他机制被认为是用于具有特定特征的应用程序，正如之前所描述的。请根据您的应用程序的特点选择适当的机制。

本章介绍了七个示例，展示了如何使用所描述的机制。

# 控制对资源的并发访问

在这个示例中，您将学习如何使用 Java 语言提供的信号量机制。信号量是保护对一个或多个共享资源的访问的计数器。

### 注

信号量的概念是由 Edsger Dijkstra 于 1965 年引入的，并且首次在 THEOS 操作系统中使用。

当一个线程想要访问其中一个共享资源时，首先必须获取信号量。如果信号量的内部计数器大于`0`，则信号量会减少计数器并允许访问共享资源。计数器大于`0`意味着有空闲资源可以使用，因此线程可以访问并使用其中一个。

否则，如果信号量的计数器为`0`，则信号量将线程置于休眠状态，直到计数器大于`0`。计数器为`0`表示所有共享资源都被其他线程使用，因此想要使用其中一个的线程必须等待直到有一个空闲。

当线程完成对共享资源的使用时，它必须释放信号量，以便其他线程可以访问共享资源。这个操作会增加信号量的内部计数器。

在这个示例中，您将学习如何使用`Semaphore`类来实现特殊类型的信号量，称为**二进制信号量**。这些类型的信号量保护对唯一共享资源的访问，因此信号量的内部计数器只能取值`1`或`0`。为了演示如何使用它，您将实现一个打印队列，可以供并发任务使用来打印它们的作业。这个打印队列将受到二进制信号量的保护，因此一次只能有一个线程打印。

## 准备工作

这个示例使用 Eclipse IDE 实现。如果您使用 Eclipse 或其他 IDE，比如 NetBeans，打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`PrintQueue`的类，它将实现打印队列。

```java
public class PrintQueue {
```

1.  声明一个`Semaphore`对象。将其命名为`semaphore`。

```java
  private final Semaphore semaphore;
```

1.  实现类的构造函数。它初始化了将保护对打印队列的访问的`semaphore`对象。

```java
  public PrintQueue(){
    semaphore=new Semaphore(1);
  }
```

1.  实现`printJob()`方法来模拟打印文档。它接收名为`document`的`Object`作为参数。

```java
  public void printJob (Object document){
```

1.  在方法内部，首先必须调用`acquire()`方法来获取信号量。这个方法可能会抛出`InterruptedException`异常，所以您必须包含一些代码来处理它。

```java
    try {
      semaphore.acquire();
```

1.  然后，实现模拟打印文档并等待随机时间段的行。

```java
  long duration=(long)(Math.random()*10);
      System.out.printf("%s: PrintQueue: Printing a Job during %d seconds\n",Thread.currentThread().getName(),duration);
      Thread.sleep(duration);    
```

1.  最后，通过调用信号量的`release()`方法释放信号量。

```java
    } catch (InterruptedException e) {
      e.printStackTrace();
    } finally {
      semaphore.release();      
    }
```

1.  创建一个名为`Job`的类，并指定它实现`Runnable`接口。这个类实现了向打印机发送文档的作业。

```java
public class Job implements Runnable {
```

1.  声明一个`PrintQueue`对象。将其命名为`printQueue`。

```java
  private PrintQueue printQueue;
```

1.  实现类的构造函数。它初始化了类中声明的`PrintQueue`对象。

```java
  public Job(PrintQueue printQueue){
    this.printQueue=printQueue;
  }
```

1.  实现`run()`方法。

```java
  @Override
   public void run() {
```

1.  首先，该方法向控制台写入一条消息，显示作业已经开始执行。

```java
    System.out.printf("%s: Going to print a job\n",Thread.currentThread().getName());
```

1.  然后，调用`PrintQueue`对象的`printJob()`方法。

```java
    printQueue.printJob(new Object());
```

1.  最后，该方法向控制台写入一条消息，显示它已经完成了执行。

```java
    System.out.printf("%s: The document has been printed\n",Thread.currentThread().getName());        
  }
```

1.  通过创建一个名为`Main`的类并实现`main()`方法来实现示例的主类。

```java
public class Main {

  public static void main (String args[]){
```

1.  创建一个名为`printQueue`的`PrintQueue`对象。

```java
    PrintQueue printQueue=new PrintQueue();
```

1.  创建 10 个线程。这些线程中的每一个都将执行一个`Job`对象，该对象将向打印队列发送一个文档。

```java
    Thread thread[]=new Thread[10];
    for (int i=0; i<10; i++){
      thread[i]=new Thread(new Job(printQueue),"Thread"+i);
    }
```

1.  最后，启动 10 个线程。

```java
    for (int i=0; i<10; i++){
      thread[i].start();
    }
```

## 它是如何工作的...

这个示例的关键在`PrintQueue`类的`printJob()`方法中。这个方法展示了在使用信号量实现临界区并保护对共享资源访问时，您必须遵循的三个步骤：

1.  首先，使用`acquire()`方法获取信号量。

1.  然后，执行与共享资源的必要操作。

1.  最后，通过调用`release()`方法释放信号量。

这个示例中的另一个重要点是`PrintQueue`类的构造函数和`Semaphore`对象的初始化。您将`1`作为这个构造函数的参数传递，因此您正在创建一个二进制信号量。内部计数器的初始值为`1`，因此您将保护对一个共享资源的访问，即打印队列。

当您启动 10 个线程时，第一个线程会获取信号量并获得对临界区的访问。其余线程被信号量阻塞，直到已经获取信号量的线程释放它。当这种情况发生时，信号量会选择一个等待的线程并允许其访问临界区。所有的作业都会打印它们的文档，但是一个接一个地进行。

## 还有更多...

`Semaphore`类有两个额外版本的`acquire()`方法：

+   `acquireUninterruptibly()`: `acquire()`方法；当信号量的内部计数器为`0`时，阻塞线程直到信号量被释放。在此阻塞时间内，线程可能会被中断，然后此方法抛出`InterruptedException`异常。此版本的获取操作忽略线程的中断，并且不会抛出任何异常。

+   `tryAcquire()`: 此方法尝试获取信号量。如果可以，该方法返回`true`值。但是如果不能，该方法返回`false`值，而不是被阻塞并等待信号量的释放。根据`return`值，您有责任采取正确的操作。

### 信号量中的公平性

公平性的概念被 Java 语言用于所有可以有各种线程阻塞等待同步资源释放的类（例如信号量）。默认模式称为**非公平模式**。在这种模式下，当同步资源被释放时，会选择等待的线程中的一个来获取此资源，但是选择是没有任何标准的。**公平模式**改变了这种行为，并强制选择等待时间更长的线程。

与其他类一样，`Semaphore`类在其构造函数中接受第二个参数。此参数必须采用`Boolean`值。如果给定`false`值，则创建一个将以非公平模式工作的信号量。如果不使用此参数，将获得相同的行为。如果给定`true`值，则创建一个将以公平模式工作的信号量。

## 另请参阅

+   第八章中的*监视锁接口*配方，*测试并发应用程序*

+   第二章中的*修改锁公平性*配方，*基本线程同步*

# 控制对资源的多个副本的并发访问

在*控制对资源的并发访问*配方中，您学习了信号量的基础知识。

在那个配方中，您使用了二进制信号量来实现一个示例。这些类型的信号量用于保护对一个共享资源的访问，或者只能由一个线程执行的临界区。但是当您需要保护资源的多个副本时，或者当您有一个可以同时由多个线程执行的临界区时，也可以使用信号量。

在这个配方中，您将学习如何使用信号量来保护多个资源的副本。您将实现一个示例，其中有一个打印队列，可以在三台不同的打印机上打印文档。

## 准备工作

本配方的示例已使用 Eclipse IDE 实现。如果您使用 Eclipse 或其他 IDE（如 NetBeans），请打开它并创建一个新的 Java 项目。

在本章中实现*控制对资源的并发访问*配方中描述的示例。

## 如何做...

按照以下步骤实现示例：

1.  正如我们之前提到的，您将修改使用信号量实现的打印队列示例。打开`PrintQueue`类并声明一个名为`freePrinters`的`boolean`数组。该数组存储可以打印作业的空闲打印机和正在打印文档的打印机。

```java
  private boolean freePrinters[];
```

1.  还要声明一个名为`lockPrinters`的`Lock`对象。您将使用此对象来保护对`freePrinters`数组的访问。

```java
  private Lock lockPrinters;
```

1.  修改类的构造函数以初始化新声明的对象。`freePrinters`数组有三个元素，全部初始化为`true`值。信号量的初始值为`3`。

```java
  public PrintQueue(){
    semaphore=new Semaphore(3);
    freePrinters=new boolean[3];
    for (int i=0; i<3; i++){
      freePrinters[i]=true;
    }
    lockPrinters=new ReentrantLock();
  }
```

1.  还要修改`printJob()`方法。它接收一个名为`document`的`Object`作为唯一参数。

```java
  public void printJob (Object document){
```

1.  首先，该方法调用`acquire()`方法来获取对信号量的访问。由于此方法可能会抛出`InterruptedException`异常，因此必须包含处理它的代码。

```java
    try {
      semaphore.acquire();
```

1.  然后，使用私有方法`getPrinter()`获取分配打印此作业的打印机的编号。

```java
      int assignedPrinter=getPrinter();
```

1.  然后，实现模拟打印文档并等待随机时间段的行。

```java
      long duration=(long)(Math.random()*10);
      System.out.printf("%s: PrintQueue: Printing a Job in Printer%d during %d seconds\n",Thread.currentThread().getName(),assignedPrinter,duration);
      TimeUnit.SECONDS.sleep(duration);
```

1.  最后，调用`release()`方法释放信号量，并将使用的打印机标记为自由，将`true`分配给`freePrinters`数组中的相应索引。

```java
      freePrinters[assignedPrinter]=true;
    } catch (InterruptedException e) {
      e.printStackTrace();
    } finally {
      semaphore.release();      
    }
```

1.  实现`getPrinter()`方法。这是一个返回`int`值的私有方法，没有参数。

```java
  private int getPrinter() {
```

1.  首先，声明一个`int`变量来存储打印机的索引。

```java
    int ret=-1;
```

1.  然后，获取`lockPrinters`对象的访问权限。

```java
    try {
      lockPrinters.lock();
```

1.  然后，在`freePrinters`数组中找到第一个`true`值，并将其索引保存在一个变量中。修改此值为`false`，因为这台打印机将忙碌。

```java
    for (int i=0; i<freePrinters.length; i++) {
      if (freePrinters[i]){
        ret=i;
        freePrinters[i]=false;
        break;
      }
    }
```

1.  最后，释放`lockPrinters`对象并返回`true`值的索引。

```java
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      lockPrinters.unlock();
    }
    return ret;
```

1.  `Job`和`Core`类没有修改。

## 它是如何工作的...

这个例子的关键在于`PrintQueue`类。使用`3`作为构造函数的参数创建`Semaphore`对象。调用`acquire()`方法的前三个线程将获得对这个例子的关键部分的访问，而其余的线程将被阻塞。当一个线程完成关键部分并释放信号量时，另一个线程将获取它。

在这个关键部分，线程获取分配打印此作业的打印机的索引。这个例子的这一部分用于使例子更加真实，但它不使用与信号量相关的任何代码。

以下屏幕截图显示了此示例的执行输出：

![它是如何工作的...](img/7881_03_01.jpg)

每个文档都在其中一个打印机上打印。第一个是空闲的。

## 还有更多...

`acquire()`、`acquireUninterruptibly()`、`tryAcquire()`和`release()`方法有一个额外的版本，它们有一个`int`参数。这个参数表示使用它们的线程想要获取或释放的许可数，也就是说，这个线程想要删除或添加到信号量的内部计数器的单位数。在`acquire()`、`acquireUninterruptibly()`和`tryAcquire()`方法的情况下，如果这个计数器的值小于这个值，线程将被阻塞，直到计数器达到这个值或更大的值。

## 另请参阅

+   第三章中的*控制对资源的并发访问*食谱，*线程同步工具*

+   第八章中的*监视锁接口*食谱，*测试并发应用*

+   第二章中的*修改锁公平性*食谱，*基本线程同步*

# 等待多个并发事件

Java 并发 API 提供了一个类，允许一个或多个线程等待一组操作完成。这就是`CountDownLatch`类。这个类用一个整数数初始化，这个整数是线程要等待的操作数。当一个线程想要等待这些操作的执行时，它使用`await()`方法。这个方法使线程进入睡眠状态，直到操作完成。当其中一个操作完成时，它使用`countDown()`方法来减少`CountDownLatch`类的内部计数器。当计数器到达`0`时，类唤醒所有在`await()`方法中睡眠的线程。

在这个食谱中，您将学习如何使用`CountDownLatch`类实现视频会议系统。视频会议系统将等待所有参与者到达后才开始。

## 准备工作

这个食谱的例子是使用 Eclipse IDE 实现的。如果您使用 Eclipse 或其他 IDE，如 NetBeans，请打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`Videoconference`的类，并指定其实现`Runnable`接口。该类将实现视频会议系统。

```java
public class Videoconference implements Runnable{
```

1.  声明一个名为`controller`的`CountDownLatch`对象。

```java
  private final CountDownLatch controller;
```

1.  实现初始化`CountDownLatch`属性的类的构造函数。`Videoconference`类将等待作为参数接收到的参与者数量的到达。

```java
  public Videoconference(int number) {
    controller=new CountDownLatch(number);
  }
```

1.  实现`arrive()`方法。每次参与者到达视频会议时，将调用此方法。它接收一个名为`name`的`String`类型的参数。

```java
  public void arrive(String name){
```

1.  首先，它使用接收到的参数编写一条消息。

```java
    System.out.printf("%s has arrived.",name);
```

1.  然后，它调用`CountDownLatch`对象的`countDown()`方法。

```java
    controller.countDown();
```

1.  最后，使用`CountDownLatch`对象的`getCount()`方法编写另一条消息，指示到达的参与者数量。

```java
    System.out.printf("VideoConference: Waiting for %d participants.\n",controller.getCount());
```

1.  实现视频会议系统的主方法。这是每个`Runnable`对象必须具有的`run()`方法。

```java
   @Override
  public void run() {
```

1.  首先，使用`getCount()`方法编写一条消息，指示视频会议中的参与者数量。

```java
    System.out.printf("VideoConference: Initialization: %d participants.\n",controller.getCount());
```

1.  然后，使用`await()`方法等待所有参与者。由于此方法可能引发`InterruptedException`异常，因此必须包含处理它的代码。

```java
    try {
      controller.await();
```

1.  最后，编写一条消息，指示所有参与者都已到达。

```java
      System.out.printf("VideoConference: All the participants have come\n");
      System.out.printf("VideoConference: Let's start...\n");
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
```

1.  创建`Participant`类并指定其实现`Runnable`接口。该类代表视频会议中的每个参与者。

```java
public class Participant implements Runnable {
```

1.  声明一个名为`conference`的私有`Videoconference`属性。

```java
  private Videoconference conference;
```

1.  声明一个名为`name`的私有`String`属性。

```java
  private String name;
```

1.  实现初始化两个属性的类的构造函数。

```java
  public Participant(Videoconference conference, String name) {
    this.conference=conference;
    this.name=name;
  }
```

1.  实现参与者的`run()`方法。

```java
   @Override
  public void run() {
```

1.  首先，让线程休眠一段随机时间。

```java
    long duration=(long)(Math.random()*10);
    try {
      TimeUnit.SECONDS.sleep(duration);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
```

1.  然后，使用`Videoconference`对象的`arrive()`方法指示该参与者的到达。

```java
    conference.arrive(name);
```

1.  最后，通过创建一个名为`Main`的类并向其中添加`main()`方法来实现示例的主类。

```java
public class Main {

  public static void main(String[] args) {
```

1.  创建一个名为`conference`的`Videoconference`对象，等待 10 个参与者。

```java
    Videoconference conference=new Videoconference(10);
```

1.  创建`Thread`来运行此`Videoconference`对象并启动它。

```java
    Thread threadConference=new Thread(conference);
    threadConference.start();
```

1.  创建 10 个`Participant`对象，一个`Thread`对象来运行每个参与者，并启动所有线程。

```java
    for (int i=0; i<10; i++){
      Participant p=new Participant(conference, "Participant "+i);
      Thread t=new Thread(p);
      t.start();
    }
```

## 它的工作原理...

`CountDownLatch`类有三个基本元素：

+   确定`CountDownLatch`类等待多少个事件的初始化值

+   `await()`方法由等待所有事件完成的线程调用

+   `countDown()`方法由事件在完成执行时调用

当创建一个`CountDownLatch`对象时，对象使用构造函数的参数来初始化内部计数器。每次线程调用`countDown()`方法时，`CountDownLatch`对象将内部计数器减少一个单位。当内部计数器达到`0`时，`CountDownLatch`对象唤醒所有在`await()`方法中等待的线程。

无法重新初始化`CountDownLatch`对象的内部计数器或修改其值。一旦计数器被初始化，您可以使用的唯一方法来修改其值是前面解释的`countDown()`方法。当计数器达到`0`时，对`await()`方法的所有调用立即返回，并且对`countDown()`方法的所有后续调用都没有效果。

与其他同步方法相比，有一些不同之处，如下所示：

+   `CountDownLatch`机制不用于保护共享资源或临界区。它用于将一个或多个线程与执行各种任务同步。

+   它只允许一次使用。正如我们之前解释的，一旦`CountDownLatch`的计数器达到`0`，对其方法的所有调用都没有效果。如果要再次进行相同的同步，必须创建一个新对象。

以下屏幕截图显示了示例执行的输出：

![它是如何工作的...](img/7881_03_02.jpg)

您可以看到最后的参与者到达，一旦内部计数器到达`0`，`CountDownLatch`对象会唤醒`Videoconference`对象，写入指示视频会议应该开始的消息。

## 还有更多...

`CountDownLatch`类有另一个版本的`await()`方法，如下所示：

+   `await`(`long``time,``TimeUnit``unit`): 线程将睡眠，直到被中断；`CountDownLatch`的内部计数器到达`0`或指定的时间过去。`TimeUnit`类是一个枚举，包含以下常量：`DAYS`, `HOURS`, `MICROSECONDS`, `MILLISECONDS`, `MINUTES`, `NANOSECONDS`, 和 `SECONDS`。

# 在一个共同点同步任务

Java 并发 API 提供了一个同步工具，允许在确定点同步两个或多个线程。这就是`CyclicBarrier`类。这个类类似于本章中*等待多个并发事件*一节中解释的`CountDownLatch`类，但有一些不同之处，使它成为一个更强大的类。

`CyclicBarrier`类用一个整数初始化，这个整数是将在确定点同步的线程数。当其中一个线程到达确定点时，它调用`await()`方法等待其他线程。当线程调用该方法时，`CyclicBarrier`类会阻塞正在睡眠的线程，直到其他线程到达。当最后一个线程调用`CyclicBarrier`类的`await()`方法时，它会唤醒所有等待的线程并继续执行任务。

`CyclicBarrier`类的一个有趣的优势是，您可以将一个额外的`Runnable`对象作为初始化参数传递给它，当所有线程到达共同点时，`CyclicBarrier`类会执行这个对象作为一个线程。这个特性使得这个类适合使用分治编程技术并行化任务。

在这个示例中，您将学习如何使用`CyclicBarrier`类来同步一组线程到一个确定的点。您还将使用一个`Runnable`对象，在所有线程到达该点后执行。在这个示例中，您将在一个数字矩阵中查找一个数字。矩阵将被分成子集（使用分治技术），因此每个线程将在一个子集中查找数字。一旦所有线程完成了它们的工作，最终任务将统一它们的结果。

## 准备开始

这个示例已经在 Eclipse IDE 中实现。如果您使用 Eclipse 或其他 IDE，比如 NetBeans，打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  我们将通过实现两个辅助类来开始这个示例。首先，创建一个名为`MatrixMock`的类。这个类将生成一个随机的数字矩阵，数字在 1 到 10 之间，线程将在其中查找一个数字。

```java
public class MatrixMock {
```

1.  声明一个名为`data`的`private``int`矩阵。

```java
  private int data[][];
```

1.  实现类的构造函数。这个构造函数将接收矩阵的行数、每行的长度和要查找的数字作为参数。所有三个参数的类型都是`int`。

```java
  public MatrixMock(int size, int length, int number){
```

1.  初始化构造函数中使用的变量和对象。

```java
    int counter=0;
    data=new int[size][length];
    Random random=new Random();
```

1.  用随机数填充矩阵。每次生成一个数字，都要与要查找的数字进行比较。如果它们相等，就增加计数器。

```java
    for (int i=0; i<size; i++) {
      for (int j=0; j<length; j++){
        data[i][j]=random.nextInt(10);
        if (data[i][j]==number){
          counter++;
        }
      }
    }
```

1.  最后，在控制台打印一条消息，显示在生成的矩阵中要查找的数字的出现次数。这条消息将用于检查线程是否得到了正确的结果。

```java
    System.out.printf("Mock: There are %d ocurrences of number in generated data.\n",counter,number);
```

1.  实现`getRow()`方法。这个方法接收一个矩阵中的行号，并返回该行（如果存在），如果不存在则返回`null`。

```java
  public int[] getRow(int row){
    if ((row>=0)&&(row<data.length)){
      return data[row];
    }
    return null;
  }
```

1.  现在，实现一个名为`Results`的类。这个类将在一个数组中存储矩阵每一行中搜索到的数字的出现次数。

```java
public class Results {
```

1.  声明一个名为`data`的私有`int`数组。

```java
  private int data[];
```

1.  实现类的构造函数。这个构造函数接收一个整数参数，表示数组的元素个数。

```java
  public Results(int size){
    data=new int[size];
  }
```

1.  实现`setData()`方法。这个方法接收一个数组中的位置和一个值作为参数，并确定数组中该位置的值。

```java
  public void  setData(int position, int value){
    data[position]=value;
  }
```

1.  实现`getData()`方法。这个方法返回结果数组的数组。

```java
  public int[] getData(){
    return data;
  }
```

1.  现在你有了辅助类，是时候实现线程了。首先，实现`Searcher`类。这个类将在随机数字矩阵的确定行中查找一个数字。创建一个名为`Searcher`的类，并指定它实现`Runnable`接口。

```java
public class Searcher implements Runnable {
```

1.  声明两个私有的`int`属性，名为`firstRow`和`lastRow`。这两个属性将确定这个对象将在哪些行中查找。

```java
  private int firstRow;

  private int lastRow;
```

1.  声明一个名为`mock`的私有`MatrixMock`属性。

```java
  private MatrixMock mock;
```

1.  声明一个名为`results`的私有`Results`属性。

```java
  private Results results;
```

1.  声明一个名为`number`的私有`int`属性，将存储我们要查找的数字。

```java
  private int number;
```

1.  声明一个名为`barrier`的`CyclicBarrier`对象。

```java
  private final CyclicBarrier barrier;
```

1.  实现类的构造函数，初始化之前声明的所有属性。

```java
  public Searcher(int firstRow, int lastRow, NumberMock mock, Results results, int number, CyclicBarrier barrier){
    this.firstRow=firstRow;
    this.lastRow=lastRow;
    this.mock=mock;
    this.results=results;
    this.number=number;
    this.barrier=barrier;
  }
```

1.  实现`run()`方法，用于搜索数字。它使用一个名为`counter`的内部变量，用于存储每一行中数字的出现次数。

```java
   @Override
  public void run() {
    int counter;
```

1.  在控制台中打印一个消息，指定给这个对象分配的行。

```java
    System.out.printf("%s: Processing lines from %d to %d.\n",Thread.currentThread().getName(),firstRow,lastRow);
```

1.  处理分配给这个线程的所有行。对于每一行，计算你要搜索的数字出现的次数，并将这个数字存储在`Results`对象的相应位置。

```java
    for (int i=firstRow; i<lastRow; i++){
      int row[]=mock.getRow(i);
      counter=0;
      for (int j=0; j<row.length; j++){
        if (row[j]==number){
          counter++;
        }
      }
      results.setData(i, counter);
    }
```

1.  在控制台中打印一条消息，指示这个对象已经完成了搜索。

```java
    System.out.printf("%s: Lines processed.\n",Thread.currentThread().getName());        
```

1.  调用`CyclicBarrier`对象的`await()`方法，并添加必要的代码来处理这个方法可能抛出的`InterruptedException`和`BrokenBarrierException`异常。

```java
    try {
      barrier.await();
    } catch (InterruptedException e) {
      e.printStackTrace();
    } catch (BrokenBarrierException e) {
      e.printStackTrace();
    }
```

1.  现在，实现计算矩阵中数字总出现次数的类。它使用存储矩阵每一行中数字出现次数的`Results`对象来进行计算。创建一个名为`Grouper`的类，并指定它实现`Runnable`接口。

```java
public class Grouper implements Runnable {
```

1.  声明一个名为`results`的私有`Results`属性。

```java
  private Results results;
```

1.  实现类的构造函数，初始化`Results`属性。

```java
  public Grouper(Results results){
    this.results=results;
  }
```

1.  实现`run()`方法，该方法将计算结果数组中数字的总出现次数。

```java
   @Override
  public void run() {
```

1.  声明一个`int`变量，并在控制台中写入一条消息，指示进程的开始。

```java
    int finalResult=0;
    System.out.printf("Grouper: Processing results...\n");
```

1.  使用`results`对象的`getData()`方法获取每一行中数字的出现次数。然后，处理数组的所有元素，并将它们的值加到`finalResult`变量中。

```java
    int data[]=results.getData();
    for (int number:data){
      finalResult+=number;
    }
```

1.  在控制台中打印结果。

```java
    System.out.printf("Grouper: Total result: %d.\n",finalResult);
```

1.  最后，通过创建一个名为`Main`的类并添加`main()`方法来实现示例的主类。

```java
public class Main {

  public static void main(String[] args) {
```

1.  声明并初始化五个常量，用于存储应用程序的参数。

```java
    final int ROWS=10000;
    final int NUMBERS=1000;
    final int SEARCH=5; 
    final int PARTICIPANTS=5;
    final int LINES_PARTICIPANT=2000;
```

1.  创建一个名为`mock`的`MatrixMock`对象。它将有 10000 行，每行 1000 个元素。现在，你要搜索数字 5。

```java
    MatrixMock mock=new MatrixMock(ROWS, NUMBERS,SEARCH);
```

1.  创建一个名为`results`的`Results`对象。它将有 10000 个元素。

```java
    Results results=new Results(ROWS);
```

1.  创建一个名为`grouper`的`Grouper`对象。

```java
    Grouper grouper=new Grouper(results);
```

1.  创建一个名为`barrier`的`CyclicBarrier`对象。这个对象将等待五个线程。当这个线程完成时，它将执行之前创建的`Grouper`对象。

```java
    CyclicBarrier barrier=new CyclicBarrier(PARTICIPANTS,grouper);
```

1.  创建五个`Searcher`对象，五个线程来执行它们，并启动这五个线程。

```java
    Searcher searchers[]=new Searcher[PARTICIPANTS];
    for (int i=0; i<PARTICIPANTS; i++){
      searchers[i]=new Searcher(i*LINES_PARTICIPANT, (i*LINES_PARTICIPANT)+LINES_PARTICIPANT, mock, results, 5,barrier);
      Thread thread=new Thread(searchers[i]);
      thread.start();
    }
    System.out.printf("Main: The main thread has finished.\n");
```

## 它是如何工作的...

以下截图显示了此示例执行的结果：

![它是如何工作的...](img/7881_03_03.jpg)

在示例中解决的问题很简单。我们有一个随机整数数字的大矩阵，你想知道这个矩阵中某个数字出现的总次数。为了获得更好的性能，我们使用分而治之的技术。我们将矩阵分成五个子集，并使用一个线程在每个子集中查找数字。这些线程是`Searcher`类的对象。

我们使用`CyclicBarrier`对象来同步五个线程的完成，并执行`Grouper`任务来处理部分结果，并计算最终结果。

正如我们之前提到的，`CyclicBarrier`类有一个内部计数器来控制有多少线程必须到达同步点。每当一个线程到达同步点时，它调用`await()`方法来通知`CyclicBarrier`对象已经到达了同步点。`CyclicBarrier`会让线程休眠，直到所有线程都到达同步点。

当所有线程都到达同步点时，`CyclicBarrier`对象会唤醒所有在`await()`方法中等待的线程，并且可以创建一个新的线程来执行在`CyclicBarrier`构造函数中传递的`Runnable`对象（在我们的例子中是`Grouper`对象）来执行额外的任务。

## 还有更多...

`CyclicBarrier`类还有另一个版本的`await()`方法：

+   `await`(`long``time,``TimeUnit``unit`)：线程将休眠，直到被中断；`CyclicBarrier`的内部计数器到达`0`或指定的时间过去。`TimeUnit`类是一个枚举，包含以下常量：`DAYS`、`HOURS`、`MICROSECONDS`、`MILLISECONDS`、`MINUTES`、`NANOSECONDS`和`SECONDS`。

该类还提供了`getNumberWaiting()`方法，返回在`await()`方法中被阻塞的线程数，以及`getParties()`方法，返回将与`CyclicBarrier`同步的任务数。

### 重置 CyclicBarrier 对象

`CyclicBarrier`类与`CountDownLatch`类有一些共同点，但也有一些不同之处。其中最重要的一个区别是`CyclicBarrier`对象可以重置为其初始状态，将其内部计数器分配给其初始化时的值。

可以使用`CyclicBarrier`类的`reset()`方法来执行此重置操作。当发生这种情况时，所有在`await()`方法中等待的线程都会收到`BrokenBarrierException`异常。在本配方中的示例中，这个异常通过打印堆栈跟踪来处理，但在更复杂的应用程序中，它可能执行其他操作，比如重新启动它们的执行或在中断点恢复它们的操作。

### 破碎的 CyclicBarrier 对象

`CyclicBarrier`对象可以处于特殊状态，用**broken**表示。当有多个线程在`await()`方法中等待，其中一个被中断时，这个线程会收到`InterruptedException`异常，但其他等待的线程会收到`BrokenBarrierException`异常，`CyclicBarrier`会处于破碎状态。

`CyclicBarrier`类提供了`isBroken()`方法，如果对象处于破碎状态，则返回`true`；否则返回`false`。

## 另请参阅

+   在第三章的*等待多个并发事件*配方中，*线程同步工具*

# 运行并发分阶段任务

Java 并发 API 提供的最复杂和强大的功能之一是使用`Phaser`类执行并发分阶段任务的能力。当我们有一些并发任务分为步骤时，这种机制非常有用。`Phaser`类为我们提供了在每个步骤结束时同步线程的机制，因此在所有线程完成第一步之前，没有线程开始第二步。

与其他同步工具一样，我们必须使用参与同步操作的任务数量初始化`Phaser`类，但是我们可以通过增加或减少这个数量来动态修改它。

在这个示例中，你将学习如何使用`Phaser`类来同步三个并发任务。这三个任务分别在三个不同的文件夹及其子文件夹中寻找最近 24 小时内修改的扩展名为`.log`的文件。这个任务分为三个步骤：

1.  获取分配文件夹及其子文件夹中扩展名为`.log`的文件列表。

1.  通过删除 24 小时前修改的文件来过滤第一步创建的列表。

1.  在控制台打印结果。

在步骤 1 和 2 结束时，我们检查列表是否有任何元素。如果没有任何元素，线程将结束执行并从`phaser`类中被删除。

## 准备工作

这个示例使用 Eclipse IDE 实现。如果你使用 Eclipse 或其他类似 NetBeans 的 IDE，打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`FileSearch`的类，并指定它实现`Runnable`接口。这个类实现了在文件夹及其子文件夹中搜索特定扩展名的文件，并且这些文件在最近 24 小时内被修改的操作。

```java
public class FileSearch implements Runnable {
```

1.  声明一个私有的`String`属性来存储搜索操作将开始的文件夹。

```java
  private String initPath;
```

1.  声明另一个私有的`String`属性来存储我们要查找的文件的扩展名。

```java
  private String end;
```

1.  声明一个私有的`List`属性来存储具有所需特征的文件的完整路径。

```java
  private List<String> results;
```

1.  最后，声明一个私有的`Phaser`属性来控制任务不同阶段的同步。

```java
  private Phaser phaser;
```

1.  实现类的构造函数，初始化类的属性。它的参数是初始文件夹的完整路径、文件的扩展名和 phaser。

```java
  public FileSearch(String initPath, String end, Phaser phaser) {
    this.initPath = initPath;
    this.end = end;
    this.phaser=phaser;
    results=new ArrayList<>();
  }
```

1.  现在，你需要实现一些辅助方法，这些方法将被`run()`方法使用。第一个是`directoryProcess()`方法。它接收一个`File`对象作为参数，并处理它的所有文件和子文件夹。对于每个文件夹，该方法将通过传递文件夹作为参数进行递归调用。对于每个文件，该方法将调用`fileProcess()`方法：

```java
  private void directoryProcess(File file) {

    File list[] = file.listFiles();
    if (list != null) {
      for (int i = 0; i < list.length; i++) {
        if (list[i].isDirectory()) {
          directoryProcess(list[i]);
        } else {
          fileProcess(list[i]);
        }
      }
    }
  }
```

1.  现在，实现`fileProcess()`方法。它接收一个`File`对象作为参数，并检查它的扩展名是否与我们要查找的扩展名相等。如果相等，这个方法将文件的绝对路径添加到结果列表中。

```java
  private void fileProcess(File file) {
    if (file.getName().endsWith(end)) {
      results.add(file.getAbsolutePath());
    }
  }
```

1.  现在，实现`filterResults()`方法。它不接收任何参数，并过滤在第一阶段获取的文件列表，删除修改时间超过 24 小时的文件。首先，创建一个新的空列表并获取当前日期。

```java
  private void filterResults() {
    List<String> newResults=new ArrayList<>();
    long actualDate=new Date().getTime();
```

1.  然后，遍历结果列表的所有元素。对于结果列表中的每个路径，为该文件创建一个`File`对象，并获取它的最后修改日期。

```java
    for (int i=0; i<results.size(); i++){
      File file=new File(results.get(i));
      long fileDate=file.lastModified();

```

1.  然后，将该日期与当前日期进行比较，如果差值小于一天，则将文件的完整路径添加到新的结果列表中。

```java
      if (actualDate-fileDate< TimeUnit.MILLISECONDS.convert(1,TimeUnit.DAYS)){
        newResults.add(results.get(i));
      }
    }
```

1.  最后，将旧的结果列表更改为新的列表。

```java
    results=newResults;
  }
```

1.  现在，实现`checkResults()`方法。这个方法将在第一阶段和第二阶段结束时被调用，它将检查结果列表是否为空。这个方法没有任何参数。

```java
  private boolean checkResults() {
```

1.  首先，检查结果列表的大小。如果为`0`，对象会向控制台写入一条消息，指示这种情况，然后调用`Phaser`对象的`arriveAndDeregister()`方法，通知它该线程已完成当前阶段，并离开阶段操作。

```java
  if (results.isEmpty()) {
      System.out.printf("%s: Phase %d: 0 results.\n",Thread.currentThread().getName(),phaser.getPhase());
      System.out.printf("%s: Phase %d: End.\n",Thread.currentThread().getName(),phaser.getPhase());
      phaser.arriveAndDeregister();
      return false;
```

1.  否则，如果结果列表有元素，对象会向控制台写入一条消息，指示这种情况，然后调用`Phaser`对象的`arriveAndAwaitAdvance()`方法，通知它该线程已完成当前阶段，并希望被阻塞，直到所有参与的线程完成当前阶段。

```java
    } else {
    System.out.printf("%s: Phase %d: %d results.\n",Thread.currentThread().getName(),phaser.getPhase(),results.size());
      phaser.arriveAndAwaitAdvance();
      return true;
    }    
  }
```

1.  最后一个辅助方法是`showInfo()`方法，它将结果列表的元素打印到控制台。

```java
  private void showInfo() {
    for (int i=0; i<results.size(); i++){
      File file=new File(results.get(i));
      System.out.printf("%s: %s\n",Thread.currentThread().getName(),file.getAbsolutePath());
    }
    phaser.arriveAndAwaitAdvance();
  }
```

1.  现在，是时候实现`run()`方法了，该方法使用前面描述的辅助方法和`Phaser`对象来控制阶段之间的变化。首先，调用`phaser`对象的`arriveAndAwaitAdvance()`方法。在创建所有线程之前，搜索不会开始。

```java
   @Override
  public void run() {

    phaser.arriveAndAwaitAdvance();
```

1.  然后，向控制台写入一条消息，指示搜索任务的开始。

```java
    System.out.printf("%s: Starting.\n",Thread.currentThread().getName());
```

1.  检查`initPath`属性是否存储了一个文件夹的名称，并使用`directoryProcess()`方法在该文件夹及其所有子文件夹中查找指定扩展名的文件。

```java
    File file = new File(initPath);
    if (file.isDirectory()) {
      directoryProcess(file);
    }
```

1.  使用`checkResults()`方法检查是否有任何结果。如果没有结果，则使用`return`关键字结束线程的执行。

```java
    if (!checkResults()){
      return;
    }
```

1.  使用`filterResults()`方法过滤结果列表。

```java
    filterResults();
```

1.  再次使用`checkResults()`方法检查是否有任何结果。如果没有结果，则使用`return`关键字结束线程的执行。

```java
    if (!checkResults()){
      return;
    }
```

1.  使用`showInfo()`方法将最终的结果列表打印到控制台，注销线程，并打印一条指示线程最终化的消息。

```java
    showInfo();
    phaser.arriveAndDeregister();
    System.out.printf("%s: Work completed.\n",Thread.currentThread().getName());
```

1.  现在，通过创建一个名为`Main`的类并向其中添加`main()`方法来实现示例的主类。

```java
public class Main {

  public static void main(String[] args) {
```

1.  创建一个具有三个参与者的`Phaser`对象。

```java
    Phaser phaser=new Phaser(3);
```

1.  为三个不同的初始文件夹创建三个`FileSearch`对象。查找扩展名为`.log`的文件。

```java
    FileSearch system=new FileSearch("C:\\Windows", "log", phaser);
    FileSearch apps=
new FileSearch("C:\\Program Files","log",phaser);
    FileSearch documents=
new FileSearch("C:\\Documents And Settings","log",phaser);
```

1.  创建并启动一个线程来执行第一个`FileSearch`对象。

```java
    Thread systemThread=new Thread(system,"System");
    systemThread.start();
```

1.  创建并启动一个线程来执行第二个`FileSearch`对象。

```java
    Thread appsThread=new Thread(apps,"Apps");
    appsThread.start();
```

1.  创建并启动一个线程来执行第三个`FileSearch`对象。

```java
    Thread documentsThread=new Thread(documents, "Documents");
    documentsThread.start();
```

1.  等待三个线程的最终化。

```java
    try {
      systemThread.join();
      appsThread.join();
      documentsThread.join();
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
```

1.  使用`isFinalized()`方法写入`Phaser`对象的最终化标志的值。

```java
    System.out.println("Terminated: "+ phaser.isTerminated());
```

## 它是如何工作的...

程序开始创建一个`Phaser`对象，该对象将控制每个阶段结束时线程的同步。`Phaser`的构造函数接收参与者的数量作为参数。在我们的例子中，`Phaser`有三个参与者。这个数字告诉`Phaser`在`Phaser`改变阶段并唤醒正在睡眠的线程之前，有多少个线程必须执行`arriveAndAwaitAdvance()`方法。

一旦创建了`Phaser`，我们启动三个线程，执行三个不同的`FileSearch`对象。

### 注意

在这个例子中，我们使用 Windows 操作系统的路径。如果您使用另一个操作系统，请修改路径以适应您环境中现有的路径。

这个`FileSearch`对象的`run()`方法中的第一条指令是调用`Phaser`对象的`arriveAndAwaitAdvance()`方法。正如我们之前提到的，`Phaser`知道我们想要同步的线程数量。当一个线程调用这个方法时，`Phaser`减少了必须完成当前阶段的线程数量，并将该线程置于睡眠状态，直到所有剩余的线程完成此阶段。在`run()`方法的开始调用这个方法，使得`FileSearch`线程中的任何一个都不会开始工作，直到所有线程都被创建。

在第一阶段和第二阶段结束时，我们检查阶段是否生成了结果，结果列表是否有元素，否则阶段没有生成结果，列表为空。在第一种情况下，`checkResults()`方法调用`arriveAndAwaitAdvance()`，如前所述。在第二种情况下，如果列表为空，线程没有继续执行的意义，所以返回。但是你必须通知屏障将会少一个参与者。为此，我们使用了`arriveAndDeregister()`。这通知屏障，这个线程已经完成了当前阶段，但不会参与未来的阶段，所以屏障不需要等待它继续。

在`showInfo()`方法中实现的第三阶段结束时，调用了屏障的`arriveAndAwaitAdvance()`方法。通过这个调用，我们保证所有线程同时结束。当这个方法执行结束时，会调用屏障的`arriveAndDeregister()`方法。通过这个调用，我们取消注册屏障的线程，因此当所有线程结束时，屏障将没有参与者。

最后，`main()`方法等待三个线程的完成，并调用屏障的`isTerminated()`方法。当一个屏障没有参与者时，它进入所谓的终止状态，这个方法返回`true`。由于我们取消注册了屏障的所有线程，它将处于终止状态，这个调用将在控制台上打印`true`。

`Phaser`对象可以处于两种状态：

+   **活跃**：当`Phaser`接受新参与者的注册并在每个阶段结束时进行同步时，`Phaser`进入这个状态。在这个状态下，`Phaser`的工作方式如本文所述。这个状态在 Java 并发 API 中没有提到。

+   **终止**：默认情况下，当所有`Phaser`的参与者都被取消注册时，`Phaser`进入这个状态，所以`Phaser`没有参与者。更详细地说，当`onAdvance()`方法返回`true`值时，`Phaser`处于终止状态。如果你重写了这个方法，你可以改变默认行为。当`Phaser`处于这个状态时，同步方法`arriveAndAwaitAdvance()`会立即返回，不执行任何同步操作。

`Phaser`类的一个显著特点是，你不需要控制与屏障相关的方法中的任何异常。与其他同步工具不同，处于屏障中休眠的线程不会响应中断事件，也不会抛出`InterruptedException`异常。下面的*还有更多*部分中只有一个例外情况。

下面的截图显示了示例执行的结果：

![它是如何工作的...](img/7881_03_04.jpg)

它显示了执行的前两个阶段。你可以看到**Apps**线程在第二阶段结束时结束了执行，因为它的结果列表为空。当你执行示例时，你会看到一些线程在其他线程之前完成了一个阶段，但它们会等待所有线程完成一个阶段后才继续执行。

## 还有更多...

`Phaser`类提供了与阶段变化相关的其他方法。这些方法如下：

+   `arrive()`: 此方法通知屏障，一个参与者已经完成了当前阶段，但不需要等待其他参与者继续执行。要小心使用此方法，因为它不会与其他线程同步。

+   `awaitAdvance(int``phase)`: 此方法将当前线程休眠，直到屏障的所有参与者完成屏障的当前阶段，如果我们传递的参数等于屏障的实际阶段。如果参数和屏障的实际阶段不相等，方法会立即返回。

+   `awaitAdvanceInterruptibly(int phaser)`: 此方法与前面解释的方法相同，但如果在此方法中休眠的线程被中断，则会抛出`InterruptedException`异常。

### 在 Phaser 中注册参与者

当您创建`Phaser`对象时，您会指示该 phaser 将有多少参与者。但是`Phaser`类有两种方法来增加 phaser 的参与者数量。这些方法如下：

+   `register()`: 此方法向`Phaser`添加一个新的参与者。这个新的参与者将被视为未到达当前阶段。

+   `bulkRegister(int Parties)`: 此方法向 phaser 添加指定数量的参与者。这些新参与者将被视为未到达当前阶段。

`Phaser`类提供的唯一方法来减少参与者数量是`arriveAndDeregister()`方法，该方法通知 phaser 线程已完成当前阶段，并且不希望继续进行分阶段操作。

### 强制终止 Phaser

当 phaser 没有参与者时，它进入由**终止**表示的状态。`Phaser`类提供了`forceTermination()`来改变 phaser 的状态，并使其独立于 phaser 中注册的参与者数量进入终止状态。当参与者中有一个出现错误情况时，强制终止 phaser 可能会有用。

当 phaser 处于终止状态时，`awaitAdvance()`和`arriveAndAwaitAdvance()`方法立即返回一个负数，而不是通常返回的正数。如果您知道您的 phaser 可能被终止，您应该验证这些方法的返回值，以了解 phaser 是否已终止。

## 另请参阅

+   *在第八章中的*监视 Phaser*示例，*测试并发应用程序*

# 控制并发分阶段任务中的相位变化

`Phaser`类提供了一个在 phaser 改变相位时执行的方法。这是`onAdvance()`方法。它接收两个参数：当前相位的编号和注册参与者的数量；它返回一个`Boolean`值，如果 phaser 继续执行，则返回`false`，如果 phaser 已完成并且必须进入终止状态，则返回`true`。

此方法的默认实现在注册的参与者数量为零时返回`true`，否则返回`false`。但是，如果您扩展`Phaser`类并覆盖此方法，则可以修改此行为。通常，当您必须在从一个阶段前进到下一个阶段时执行一些操作时，您会对此感兴趣。

在这个示例中，您将学习如何控制实现自己版本的`Phaser`类中的相位变化，该类覆盖了`onAdvance()`方法以在每个相位变化时执行一些操作。您将实现一个考试的模拟，其中将有一些学生需要完成三个练习。所有学生都必须在进行下一个练习之前完成一个练习。

## 准备就绪

此示例已使用 Eclipse IDE 实现。如果您使用 Eclipse 或其他 IDE（如 NetBeans），请打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`MyPhaser`的类，并指定它从`Phaser`类扩展。

```java
public class MyPhaser extends Phaser {
```

1.  覆盖`onAdvance()`方法。根据阶段属性的值，我们调用不同的辅助方法。如果阶段等于零，你必须调用`studentsArrived()`方法。如果阶段等于一，你必须调用`finishFirstExercise()`方法。如果阶段等于二，你必须调用`finishSecondExercise()`方法，如果阶段等于三，你必须调用`finishExam()`方法。否则，我们返回`true`值以指示 phaser 已经终止。

```java
   @Override
  protected boolean onAdvance(int phase, int registeredParties) {
    switch (phase) {
    case 0:
      return studentsArrived();
    case 1:
      return finishFirstExercise();
    case 2:
      return finishSecondExercise();
    case 3:
      return finishExam();
    default:
      return true;
    }
  }
```

1.  实现辅助方法`studentsArrived()`。它在控制台上写入两条日志消息，并返回`false`值以指示 phaser 继续执行。

```java
  private boolean studentsArrived() {
    System.out.printf("Phaser: The exam are going to start. The students are ready.\n");
    System.out.printf("Phaser: We have %d students.\n",getRegisteredParties());
    return false;
  }
```

1.  实现辅助方法`finishFirstExercise()`。它在控制台上写入两条消息，并返回`false`值以指示 phaser 继续执行。

```java
  private boolean finishFirstExercise() {
    System.out.printf("Phaser: All the students have finished the first exercise.\n");
    System.out.printf("Phaser: It's time for the second one.\n");
    return false;
  }
```

1.  实现辅助方法`finishSecondExercise()`。它在控制台上写入两条消息，并返回`false`值以指示 phaser 继续执行。

```java
  private boolean finishSecondExercise() {
    System.out.printf("Phaser: All the students have finished the second exercise.\n");
    System.out.printf("Phaser: It's time for the third one.\n");
    return false;
  }
```

1.  实现辅助方法`finishExam()`。它在控制台上写入两条消息，并返回`true`值以指示 phaser 已经完成了它的工作。

```java
  private boolean finishExam() {
    System.out.printf("Phaser: All the students have finished the exam.\n");
    System.out.printf("Phaser: Thank you for your time.\n");
    return true;
  }
```

1.  创建一个名为`Student`的类，并指定它实现`Runnable`接口。这个类将模拟考试的学生。

```java
public class Student implements Runnable {
```

1.  声明一个名为`phaser`的`Phaser`对象。

```java
  private Phaser phaser;
```

1.  实现初始化`Phaser`对象的类的构造函数。

```java
  public Student(Phaser phaser) {
    this.phaser=phaser;
  }
```

1.  实现将模拟考试的`run()`方法。

```java
   @Override
  public void run() {
```

1.  首先，该方法在控制台中写入一条消息，指示该学生已经到达考试，并调用 phaser 的`arriveAndAwaitAdvance()`方法等待其他线程完成第一个练习。

```java
    System.out.printf("%s: Has arrived to do the exam. %s\n",Thread.currentThread().getName(),new Date());
    phaser.arriveAndAwaitAdvance();
```

1.  然后，在控制台上写一条消息，调用私有的`doExercise1()`方法来模拟考试的第一个练习，再在控制台上写一条消息，并调用 phaser 的`arriveAndAwaitAdvance()`方法等待其他学生完成第一个练习。

```java
    System.out.printf("%s: Is going to do the first exercise. %s\n",Thread.currentThread().getName(),new Date());
    doExercise1();
    System.out.printf("%s: Has done the first exercise. %s\n",Thread.currentThread().getName(),new Date());
    phaser.arriveAndAwaitAdvance();
```

1.  为第二个练习和第三个练习实现相同的代码。

```java
    System.out.printf("%s: Is going to do the second exercise. %s\n",Thread.currentThread().getName(),new Date());
    doExercise2();
    System.out.printf("%s: Has done the second exercise. %s\n",Thread.currentThread().getName(),new Date());
    phaser.arriveAndAwaitAdvance();
    System.out.printf("%s: Is going to do the third exercise. %s\n",Thread.currentThread().getName(),new Date());
    doExercise3();
    System.out.printf("%s: Has finished the exam. %s\n",Thread.currentThread().getName(),new Date());
    phaser.arriveAndAwaitAdvance();
```

1.  实现辅助方法`doExercise1()`。这个方法让线程睡眠一段随机时间。

```java
  private void doExercise1() {
    try {
      long duration=(long)(Math.random()*10);
      TimeUnit.SECONDS.sleep(duration);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
```

1.  实现辅助方法`doExercise2()`。这个方法让线程睡眠一段随机时间。

```java
  private void doExercise2() {
    try {
      long duration=(long)(Math.random()*10);
      TimeUnit.SECONDS.sleep(duration);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
```

1.  实现辅助方法`doExercise3()`。这个方法让线程睡眠一段随机时间。

```java
  private void doExercise3() {
    try {
      long duration=(long)(Math.random()*10);
      TimeUnit.SECONDS.sleep(duration);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
```

1.  通过创建一个名为`Main`的类并向其中添加`main()`方法来实现示例的主类。

```java
public class Main {

  public static void main(String[] args) {
```

1.  创建一个`MyPhaser`对象。

```java
    MyPhaser phaser=new MyPhaser();
```

1.  创建五个`Student`对象，并使用`register()`方法在 phaser 中注册它们。

```java
    Student students[]=new Student[5];
    for (int i=0; i<students.length; i++){
      students[i]=new Student(phaser);
      phaser.register();
    }
```

1.  创建五个线程来运行`students`并启动它们。

```java
    Thread threads[]=new Thread[students.length];
    for (int i=0; i<students.length; i++){
      threads[i]=new Thread(students[i],"Student "+i);
      threads[i].start();
    }
```

1.  等待五个线程的完成。

```java
    for (int i=0; i<threads.length; i++){
      try {
        threads[i].join();
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
```

1.  写一条消息来显示 phaser 处于终止状态，使用`isTerminated()`方法。

```java
    System.out.printf("Main: The phaser has finished: %s.\n",phaser.isTerminated());
```

## 它是如何工作的...

这个练习模拟了一个有三个练习的考试的实现。所有学生都必须在开始下一个练习之前完成一个练习。为了实现这个同步要求，我们使用了`Phaser`类，但你已经实现了自己的 phaser，扩展了原始类以覆盖`onAdvance()`方法。

这个方法在 phaser 在进行阶段改变之前和在唤醒所有在`arriveAndAwaitAdvance()`方法中睡眠的线程之前被 phaser 调用。这个方法接收实际阶段的编号作为参数，其中`0`是第一个阶段的编号，注册参与者的数量。最有用的参数是实际阶段。如果根据实际阶段执行不同的操作，你必须使用一个替代结构（`if`/`else`或`switch`）来选择你想要执行的操作。在这个例子中，我们使用了一个`switch`结构来选择每个阶段变化的不同方法。

`onAdvance()`方法返回一个`Boolean`值，指示 phaser 是否已终止。如果 phaser 返回`false`值，则表示它尚未终止，因此线程将继续执行其他阶段。如果 phaser 返回`true`值，则 phaser 仍然唤醒挂起的线程，但将 phaser 移动到终止状态，因此对 phaser 的任何方法的未来调用都将立即返回，并且`isTerminated()`方法返回`true`值。

在`Core`类中，当您创建`MyPhaser`对象时，您没有指定 phaser 中参与者的数量。您为每个创建的`Student`对象调用`register()`方法来注册 phaser 中的参与者。这种调用并不建立`Student`对象或执行它的线程与 phaser 之间的关系。实际上，phaser 中的参与者数量只是一个数字。phaser 和参与者之间没有关系。

以下屏幕截图显示了此示例的执行结果：

![它是如何工作的...](img/7881_03_05.jpg)

您可以看到学生们在不同时间完成第一个练习。当所有人都完成了那个练习时，phaser 调用`onAdvance()`方法在控制台中写入日志消息，然后所有学生同时开始第二个练习。

## 另请参阅

+   第三章中的*运行并发分阶段任务*食谱，*线程同步实用程序*

+   第八章中的*监视 Phaser*食谱，*测试并发应用程序*

# 在并发任务之间交换数据

Java 并发 API 提供了一个同步实用程序，允许在两个并发任务之间交换数据。更详细地说，`Exchanger`类允许在两个线程之间定义同步点。当两个线程到达此点时，它们交换一个数据结构，因此第一个线程的数据结构传递给第二个线程，第二个线程的数据结构传递给第一个线程。

这个类在类似生产者-消费者问题的情况下可能非常有用。这是一个经典的并发问题，其中有一个共同的数据缓冲区，一个或多个数据生产者和一个或多个数据消费者。由于`Exchanger`类只同步两个线程，所以如果你有一个只有一个生产者和一个消费者的生产者-消费者问题，你可以使用它。

在这个示例中，您将学习如何使用`Exchanger`类来解决只有一个生产者和一个消费者的生产者-消费者问题。

## 准备工作

本示例已使用 Eclipse IDE 实现。如果您使用 Eclipse 或其他类似 NetBeans 的 IDE，请打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  首先，让我们开始实现生产者。创建一个名为`Producer`的类，并指定它实现`Runnable`接口。

```java
public class Producer implements Runnable {
```

1.  声明一个名为`buffer`的`List<String>`对象。这将是生产者与消费者进行交换的数据结构。

```java
  private List<String> buffer;
```

1.  声明一个名为`exchanger`的`Exchanger<List<String>>`对象。这将是用于同步生产者和消费者的交换对象。

```java
  private final Exchanger<List<String>> exchanger;
```

1.  实现初始化两个属性的类的构造函数。

```java
  public Producer (List<String> buffer, Exchanger<List<String>> exchanger){
    this.buffer=buffer;
    this.exchanger=exchanger;
  }
```

1.  实现`run()`方法。在其中，实现 10 个交换周期。

```java
  @Override
  public void run() {
    int cycle=1;

    for (int i=0; i<10; i++){
      System.out.printf("Producer: Cycle %d\n",cycle);
```

1.  在每个循环中，向缓冲区添加 10 个字符串。

```java
      for (int j=0; j<10; j++){
        String message="Event "+((i*10)+j);
        System.out.printf("Producer: %s\n",message);
        buffer.add(message);
      }
```

1.  调用`exchange()`方法与消费者交换数据。由于这个方法可能抛出`InterruptedException`异常，你必须添加处理它的代码。

```java
      try {
        buffer=exchanger.exchange(buffer);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      System.out.println("Producer: "+buffer.size());
      cycle++;
    }
```

1.  现在，让我们实现消费者。创建一个名为`Consumer`的类，并指定它实现`Runnable`接口。

```java
public class Consumer implements Runnable {
```

1.  声明一个名为`buffer`的`List<String>`对象。这将是生产者与消费者进行交换的数据结构。

```java
  private List<String> buffer;
```

1.  声明一个名为`exchanger`的`Exchanger<List<String>>`对象。这将是用于同步生产者和消费者的交换对象。

```java
  private final Exchanger<List<String>> exchanger;
```

1.  实现初始化两个属性的类的构造函数。

```java
  public Consumer(List<String> buffer, Exchanger<List<String>> exchanger){
    this.buffer=buffer;
    this.exchanger=exchanger;
  }
```

1.  实现`run()`方法。在其中，实现 10 个交换周期。

```java
  @Override
  public void run() {
    int cycle=1;

    for (int i=0; i<10; i++){
      System.out.printf("Consumer: Cycle %d\n",cycle);
```

1.  在每个周期中，首先调用`exchange()`方法与生产者同步。消费者需要数据来消费。由于此方法可能抛出`InterruptedException`异常，因此您必须添加处理它的代码。

```java
      try {
        buffer=exchanger.exchange(buffer);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
```

1.  将生产者发送到其缓冲区的 10 个字符串写入控制台并从缓冲区中删除它们，使其保持为空。

```java
      System.out.println("Consumer: "+buffer.size());

      for (int j=0; j<10; j++){
        String message=buffer.get(0);
        System.out.println("Consumer: "+message);
        buffer.remove(0);
      }

      cycle++;
    }
```

1.  现在，通过创建一个名为`Core`的类并为其添加`main()`方法来实现示例的主类。

```java
public class Core {

  public static void main(String[] args) {
```

1.  创建生产者和消费者将使用的两个缓冲区。

```java
    List<String> buffer1=new ArrayList<>();
    List<String> buffer2=new ArrayList<>();
```

1.  创建`Exchanger`对象，用于同步生产者和消费者。

```java
    Exchanger<List<String>> exchanger=new Exchanger<>();
```

1.  创建`Producer`对象和`Consumer`对象。

```java
    Producer producer=new Producer(buffer1, exchanger);
    Consumer consumer=new Consumer(buffer2, exchanger);
```

1.  创建线程来执行生产者和消费者，并启动线程。

```java
    Thread threadProducer=new Thread(producer);
    Thread threadConsumer=new Thread(consumer);

    threadProducer.start();
    threadConsumer.start();
```

## 它是如何工作的...

消费者从一个空缓冲区开始，并调用`Exchanger`与生产者同步。它需要数据来消费。生产者从一个空缓冲区开始执行。它创建 10 个字符串，将其存储在缓冲区中，并使用交换器与消费者同步。

此时，生产者和消费者两个线程都在`Exchanger`中，并且它会更改数据结构，因此当消费者从`exchange()`方法返回时，它将拥有一个包含 10 个字符串的缓冲区。当生产者从`exchange()`方法返回时，它将有一个空的缓冲区再次填充。这个操作将重复 10 次。

如果执行示例，您将看到生产者和消费者如何同时执行其工作，以及两个对象如何在每一步中交换它们的缓冲区。与其他同步工具一样，调用`exchange()`方法的第一个线程将被放到睡眠状态，直到其他线程到达。

## 还有更多...

`Exchanger`类有另一个版本的交换方法：`exchange(V data, long time, TimeUnit unit)`，其中`V`是在`Phaser`声明中使用的类型（在我们的例子中是`List<String>`）。线程将休眠，直到被中断，另一个线程到达，或者指定的时间过去。`TimeUnit`类是一个枚举，具有以下常量：`DAYS`，`HOURS`，`MICROSECONDS`，`MILLISECONDS`，`MINUTES`，`NANOSECONDS`和`SECONDS`。
