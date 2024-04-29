# 第七章。自定义并发类

在本章中，我们将涵盖：

+   自定义`ThreadPoolExecutor`类

+   实现基于优先级的`Executor`类

+   实现`ThreadFactory`接口以生成自定义线程

+   在`Executor`对象中使用我们的`ThreadFactory`

+   自定义在计划线程池中运行的任务

+   实现`ThreadFactory`接口以为 Fork/Join 框架生成自定义线程

+   自定义在 Fork/Join 框架中运行的任务

+   实现自定义`Lock`类

+   基于优先级实现传输队列

+   实现自己的原子对象

# 介绍

Java 并发 API 提供了许多接口和类来实现并发应用程序。它们提供低级机制，如`Thread`类、`Runnable`或`Callable`接口或`synchronized`关键字，以及高级机制，如 Executor 框架和 Java 7 版本中添加的 Fork/Join 框架。尽管如此，您可能会发现自己正在开发一个程序，其中没有任何 java 类满足您的需求。

在这种情况下，您可能需要基于 Java 提供的工具来实现自己的自定义并发工具。基本上，您可以：

+   实现一个接口以提供该接口定义的功能。例如，`ThreadFactory`接口。

+   重写类的一些方法以使其行为适应您的需求。例如，重写`Thread`类的`run()`方法，默认情况下不执行任何有用的操作，应该重写以提供一些功能。

通过本章的示例，您将学习如何更改一些 Java 并发 API 类的行为，而无需从头设计并发框架。您可以将这些示例作为实现自定义的初始点。

# 自定义 ThreadPoolExecutor 类

Executor 框架是一种允许您将线程创建与其执行分离的机制。它基于`Executor`和`ExecutorService`接口，使用实现了这两个接口的`ThreadPoolExecutor`类。它具有内部线程池，并提供方法，允许您发送两种类型的任务以在池化线程中执行。这些任务是：

+   `Runnable`接口以实现不返回结果的任务

+   `Callable`接口以实现返回结果的任务

在这两种情况下，您只需将任务发送到执行器。执行器使用其池化线程之一或创建一个新线程来执行这些任务。执行器还决定任务执行的时机。

在本示例中，您将学习如何重写`ThreadPoolExecutor`类的一些方法，以计算在执行器中执行的任务的执行时间，并在执行器完成执行时在控制台中写入有关执行器的统计信息。

## 准备工作

本示例的示例是使用 Eclipse IDE 实现的。如果您使用 Eclipse 或其他 IDE（如 NetBeans），请打开它并创建一个新的 Java 项目。

## 如何做…

按照下面描述的步骤实现示例：

1.  创建一个名为`MyExecutor`的类，它扩展了`ThreadPoolExecutor`类。

```java
public class MyExecutor extends ThreadPoolExecutor {
```

1.  声明一个私有的`ConcurrentHashMap`属性，参数化为`String`和`Date`类，命名为`startTimes`。

```java
  private ConcurrentHashMap<String, Date> startTimes;
```

1.  实现该类的构造函数。使用`super`关键字调用父类的构造函数并初始化`startTime`属性。

```java
  public MyExecutor(int corePoolSize, int maximumPoolSize,
      long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
    super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
    startTimes=new ConcurrentHashMap<>();
  }
```

1.  重写`shutdown()`方法。在控制台中写入有关已执行任务、正在运行任务和待处理任务的信息。然后，使用`super`关键字调用父类的`shutdown()`方法。

```java
  @Override
  public void shutdown() {
    System.out.printf("MyExecutor: Going to shutdown.\n");
    System.out.printf("MyExecutor: Executed tasks: %d\n",getCompletedTaskCount());
    System.out.printf("MyExecutor: Running tasks: %d\n",getActiveCount());
    System.out.printf("MyExecutor: Pending tasks: %d\n",getQueue().size());
    super.shutdown();
  }
```

1.  重写`shutdownNow()`方法。在控制台中写入有关已执行任务、正在运行任务和待处理任务的信息。然后，使用`super`关键字调用父类的`shutdownNow()`方法。

```java
  @Override
  public List<Runnable> shutdownNow() {
    System.out.printf("MyExecutor: Going to immediately shutdown.\n");
    System.out.printf("MyExecutor: Executed tasks: %d\n",getCompletedTaskCount());
    System.out.printf("MyExecutor: Running tasks: %d\n",getActiveCount());
    System.out.printf("MyExecutor: Pending tasks: %d\n",getQueue().size());
    return super.shutdownNow();
  }
```

1.  覆盖`beforeExecute()`方法。在控制台中写入将执行任务的线程的名称和任务的哈希码的消息。使用任务的哈希码作为键，将开始日期存储在`HashMap`中。

```java
  @Override
  protected void beforeExecute(Thread t, Runnable r) {
    System.out.printf("MyExecutor: A task is beginning: %s : %s\n",t.getName(),r.hashCode());
    startTimes.put(String.valueOf(r.hashCode()), new Date());
  }
```

1.  覆盖`afterExecute()`方法。在控制台中写入任务的结果，并计算任务的运行时间，减去存储在`HashMap`中的任务的开始日期。

```java
  @Override
  protected void afterExecute(Runnable r, Throwable t) {
    Future<?> result=(Future<?>)r;
    try {
      System.out.printf("*********************************\n");
      System.out.printf("MyExecutor: A task is finishing.\n");
      System.out.printf("MyExecutor: Result: %s\n",result.get());
      Date startDate=startTimes.remove(String.valueOf(r.hashCode()));
      Date finishDate=new Date();
      long diff=finishDate.getTime()-startDate.getTime();
      System.out.printf("MyExecutor: Duration: %d\n",diff);
      System.out.printf("*********************************\n");
    } catch (InterruptedException  | ExecutionException e) {
      e.printStackTrace();
    }
  }
}
```

1.  创建一个名为`SleepTwoSecondsTask`的类，实现带有`String`类参数的`Callable`接口。实现`call()`方法。将当前线程休眠 2 秒，并返回转换为`String`类型的当前日期。

```java
public class SleepTwoSecondsTask implements Callable<String> {

  public String call() throws Exception {
    TimeUnit.SECONDS.sleep(2);
    return new Date().toString();
  }

}
```

1.  通过创建一个名为`Main`的主类来实现示例的主要类，其中包含一个`main()`方法。

```java
public class Main {
  public static void main(String[] args) {
```

1.  创建一个名为`myExecutor`的`MyExecutor`对象。

```java
    MyExecutor myExecutor=new MyExecutor(2, 4, 1000, TimeUnit.MILLISECONDS, new LinkedBlockingDeque<Runnable>());
```

1.  创建一个带有`String`类参数的`Future`对象列表，用于存储您要发送到执行器的任务的结果对象。

```java
    List<Future<String>> results=new ArrayList<>();¡;
```

1.  提交 10 个`Task`对象。

```java
    for (int i=0; i<10; i++) {
      SleepTwoSecondsTask task=new SleepTwoSecondsTask();
      Future<String> result=myExecutor.submit(task);
      results.add(result);
    }
```

1.  使用`get()`方法获取前五个任务的执行结果。在控制台中写入它们。

```java
    for (int i=0; i<5; i++){
      try {
        String result=results.get(i).get();
        System.out.printf("Main: Result for Task %d : %s\n",i,result);
      } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
      }
    }
```

1.  使用`shutdown()`方法结束执行器的执行。

```java
    myExecutor.shutdown();
```

1.  使用`get()`方法获取最后五个任务的执行结果。在控制台中写入它们。

```java
    for (int i=5; i<10; i++){
      try {
        String result=results.get(i).get();
        System.out.printf("Main: Result for Task %d : %s\n",i,result);
      } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
      }
    }
```

1.  使用`awaitTermination()`方法等待执行器的完成。

```java
    try {
      myExecutor.awaitTermination(1, TimeUnit.DAYS);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
```

1.  写一条消息指示程序执行结束。

```java
    System.out.printf("Main: End of the program.\n");
```

## 它是如何工作的...

在这个配方中，我们扩展了`ThreadPoolExecutor`类来实现我们的自定义执行器，并覆盖了它的四个方法。`beforeExecute()`和`afterExecute()`方法用于计算任务的执行时间。`beforeExecute()`方法在任务执行之前执行。在这种情况下，我们使用`HashMap`来存储任务的开始日期。`afterExecute()`方法在任务执行后执行。您可以从`HashMap`中获取已完成任务的`startTime`，然后计算实际日期与该日期之间的差异，以获取任务的执行时间。您还覆盖了`shutdown()`和`shutdownNow()`方法，以将执行器中执行的任务的统计信息写入控制台：

+   使用`getCompletedTaskCount()`方法执行的任务

+   使用`getActiveCount()`方法获取当前正在运行的任务

使用阻塞队列的`size()`方法来存储待处理任务的执行器。实现`Callable`接口的`SleepTwoSecondsTask`类将其执行线程休眠 2 秒，`Main`类中，您向执行器发送 10 个任务，使用它和其他类来演示它们的特性。

执行程序，您将看到程序显示每个正在运行的任务的时间跨度以及在调用`shutdown()`方法时执行器的统计信息。

## 另请参阅

+   第四章中的*创建线程执行器*配方，*线程执行器*

+   第七章中的*在执行器中使用我们的 ThreadFactory*对象配方，*自定义并发类*

# 实现基于优先级的执行器类

在 Java 并发 API 的早期版本中，您必须创建和运行应用程序的所有线程。在 Java 版本 5 中，随着执行器框架的出现，引入了一种新的机制来执行并发任务。

使用执行器框架，您只需实现您的任务并将其发送到执行器。执行器负责创建和执行执行您的任务的线程。

在内部，执行程序使用阻塞队列来存储待处理任务。这些任务按照它们到达执行程序的顺序进行存储。一种可能的替代方案是使用优先级队列来存储新任务。这样，如果具有高优先级的新任务到达执行程序，它将在已经等待线程执行的其他线程之前执行，但具有较低优先级。

在本示例中，您将学习如何实现一个执行程序，该执行程序将使用优先级队列来存储您发送的任务以供执行。

## 准备就绪

本示例已使用 Eclipse IDE 实现。如果您使用 Eclipse 或其他 IDE（如 NetBeans），请打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`MyPriorityTask`的类，该类实现了`Runnable`和`Comparable`接口，参数化为`MyPriorityTask`类接口。

```java
public class MyPriorityTask implements Runnable, Comparable<MyPriorityTask> {
```

1.  声明一个名为`priority`的私有`int`属性。

```java
  private int priority;  
```

1.  声明一个名为`name`的私有`String`属性。

```java
  private String name;
```

1.  通过实现类的构造函数来初始化其属性。

```java
  public MyPriorityTask(String name, int priority) {
    this.name=name;
    this.priority=priority;
  }
```

1.  实现一个方法来返回优先级属性的值。

```java
  public int getPriority(){
    return priority;
  }
```

1.  实现`Comparable`接口中声明的`compareTo()`方法。它接收一个`MyPriorityTask`对象作为参数，并比较两个对象的优先级，当前对象和参数对象。您让具有更高优先级的任务在具有较低优先级的任务之前执行。

```java
  @Override
  public int compareTo(MyPriorityTask o) {
    if (this.getPriority() < o.getPriority()) {
      return 1;
    }
    if (this.getPriority() > o.getPriority()) {
      return -1;
    }
    return 0;
  }
```

1.  实现`run()`方法。将当前线程休眠 2 秒。

```java
   @Override
  public void run() {
    System.out.printf("MyPriorityTask: %s Priority : %d\n",name,priority);
    try {
      TimeUnit.SECONDS.sleep(2);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }  
  }
```

1.  通过创建一个名为`Main`的类并实现一个`main()`方法来实现示例的主类。

```java
public class Main {
  public static void main(String[] args) {
```

1.  创建名为`executor`的`ThreadPoolExecutor`对象。使用`PriorityBlockingQueue`参数化为`Runnable`接口作为此执行程序将用于存储其待处理任务的队列。

```java
    ThreadPoolExecutor executor=new ThreadPoolExecutor(2,2,1,TimeUnit.SECONDS,new PriorityBlockingQueue<Runnable>());
```

1.  使用循环的计数器作为任务的优先级，向执行程序发送四个任务。使用`execute()`方法将任务发送到执行程序。

```java
    for (int i=0; i<4; i++){
      MyPriorityTask task=new MyPriorityTask ("Task "+i,i);
      executor.execute(task);
    }
```

1.  将当前线程休眠 1 秒。

```java
    try {
      TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
```

1.  使用循环的计数器作为任务的优先级，向执行程序发送四个额外的任务。使用`execute()`方法将任务发送到执行程序。

```java
    for (int i=4; i<8; i++) {
      MyPriorityTask task=new MyPriorityTask ("Task "+i,i);
      executor.execute(task);      
    }
```

1.  使用`shutdown()`方法关闭执行程序。

```java
    executor.shutdown();
```

1.  使用`awaitTermination()`方法等待执行程序的完成。

```java
    try {
      executor.awaitTermination(1, TimeUnit.DAYS);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
```

1.  在控制台中写入一条消息，指示程序的完成。

```java
    System.out.printf("Main: End of the program.\n");
```

## 它是如何工作的...

将执行程序转换为基于优先级的执行程序很简单。您只需传递一个使用`Runnable`接口参数化的`PriorityBlockingQueue`对象作为参数。但是对于执行程序，您应该知道存储在优先级队列中的所有对象都必须实现`Comparable`接口。

您已经实现了`MyPriorityTask`类，该类实现了`Runnable`接口，用作任务，并实现了`Comparable`接口，用于存储在优先级队列中。该类具有一个`Priority`属性，用于存储任务的优先级。如果任务的这个属性具有更高的值，它将更早执行。`compareTo()`方法确定了优先级队列中任务的顺序。在`Main`类中，您向执行程序发送了八个具有不同优先级的任务。您发送给执行程序的第一个任务是最先执行的任务。当执行程序空闲等待要执行的任务时，随着第一个任务到达执行程序，它立即执行它们。您使用两个执行线程创建了执行程序，因此前两个任务将是最先执行的任务。然后，其余的任务将根据它们的优先级执行。

以下屏幕截图显示了此示例的一个执行：

![它是如何工作的...](img/7881_07_01.jpg)

## 还有更多...

您可以配置`Executor`以使用`BlockingQueue`接口的任何实现。一个有趣的实现是`DelayQueue`。这个类用于存储延迟激活的元素。它提供了只返回活动对象的方法。您可以使用这个类来实现自己版本的`ScheduledThreadPoolExecutor`类。

## 另请参阅

+   第四章中的*创建线程执行器*配方，*线程执行器*

+   第七章中的*自定义 ThreadPoolExecutor 类*配方，*自定义并发类*

+   第六章中的*使用按优先级排序的阻塞线程安全列表*配方，*并发集合*

# 实现 ThreadFactory 接口以生成自定义线程

**工厂模式**是面向对象编程世界中广泛使用的设计模式。它是一个创建模式，其目标是开发一个类，其任务是创建一个或多个类的对象。然后，当我们想要创建这些类中的一个对象时，我们使用工厂而不是使用`new`运算符。

+   使用这个工厂，我们集中了对象的创建，从而方便地改变创建的对象的类或创建这些对象的方式，从而轻松限制了有限资源的对象创建。例如，我们可以只有*N*个对象，这些对象很容易生成有关对象创建的统计数据。

Java 提供了`ThreadFactory`接口来实现`Thread`对象工厂。Java 并发 API 的一些高级工具，如 Executor 框架或 Fork/Join 框架，使用线程工厂来创建线程。

Java 并发 API 中工厂模式的另一个例子是`Executors`类。它提供了许多方法来创建不同类型的`Executor`对象。

在这个配方中，您将通过添加新功能来扩展`Thread`类，并实现一个线程工厂类来生成该新类的线程。

## 准备工作

这个配方的示例是使用 Eclipse IDE 实现的。如果您使用 Eclipse 或其他 IDE，如 NetBeans，打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`MyThread`的类，它继承`Thread`类。

```java
public class MyThread extends Thread {
```

1.  声明三个私有的`Date`属性，分别命名为`creationDate`、`startDate`和`finishDate`。

```java
  private Date creationDate;
  private Date startDate;
  private Date finishDate;
```

1.  实现类的构造函数。它接收名称和`Runnable`对象作为参数。存储线程的创建日期。

```java
  public MyThread(Runnable target, String name ){
    super(target,name);
    setCreationDate();
  }
```

1.  实现`run()`方法。存储线程的开始日期，调用父类的`run()`方法，并存储执行的完成日期。

```java
  @Override
  public void run() {
    setStartDate();
    super.run();
    setFinishDate();
  }
```

1.  实现一个方法来建立`creationDate`属性的值。

```java
  public void setCreationDate() {
    creationDate=new Date();
  }
```

1.  实现一个方法来建立`startDate`属性的值。

```java
  public void setStartDate() {
    startDate=new Date();
  }
```

1.  实现一个方法来建立`finishDate`属性的值。

```java
  public void setFinishDate() {
    finishDate=new Date();
  }
```

1.  实现一个名为`getExecutionTime()`的方法，它计算线程的执行时间，即开始日期和完成日期之间的差异。

```java
  public long getExecutionTime() {
    return finishDate.getTime()-startDate.getTime();
  }
```

1.  重写`toString()`方法以返回线程的创建日期和执行时间。

```java
  @Override
  public String toString(){
    StringBuilder buffer=new StringBuilder();
    buffer.append(getName());
    buffer.append(": ");
    buffer.append(" Creation Date: ");
    buffer.append(creationDate);
    buffer.append(" : Running time: ");
    buffer.append(getExecutionTime());
    buffer.append(" Milliseconds.");
    return buffer.toString();
  }
```

1.  创建一个名为`MyThreadFactory`的类，它实现`ThreadFactory`接口。

```java
public class MyThreadFactory implements ThreadFactory {
```

1.  声明一个私有的`int`属性，命名为`counter`。

```java
  private int counter;
```

1.  声明一个私有的`String`属性，命名为`prefix`。

```java
  private String prefix;
```

1.  实现类的构造函数以初始化其属性。

```java
  public MyThreadFactory (String prefix) {
    this.prefix=prefix;
    counter=1;
  }
```

1.  实现`newThread()`方法。创建一个`MyThread`对象并增加`counter`属性。

```java
  @Override
  public Thread newThread(Runnable r) {
    MyThread myThread=new MyThread(r,prefix+"-"+counter);
    counter++;
    return myThread;
  }
```

1.  创建一个名为`MyTask`的类，它实现`Runnable`接口。实现`run()`方法。让当前线程休眠 2 秒。

```java
public class MyTask implements Runnable {
  @Override
  public void run() {
    try {
      TimeUnit.SECONDS.sleep(2);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
  }
```

1.  通过创建一个名为`Main`的类并添加一个`main()`方法来实现示例的主类。

```java
public class Main {
  public static void main(String[] args) throws Exception {
```

1.  创建一个`MyThreadFactory`对象。

```java
    MyThreadFactory myFactory=new MyThreadFactory("MyThreadFactory");
```

1.  创建一个`Task`对象。

```java
    MyTask task=new MyTask();
```

1.  创建一个`MyThread`对象，使用工厂的`newThread()`方法来执行任务。

```java
    Thread thread=myFactory.newThread(task);
```

1.  启动线程并等待其完成。

```java
    thread.start();
    thread.join();
```

1.  使用`toString()`方法编写有关线程的信息。

```java
    System.out.printf("Main: Thread information.\n");
    System.out.printf("%s\n",thread);
    System.out.printf("Main: End of the example.\n");
```

## 它是如何工作的...

在本篇文章中，您已经实现了一个自定义的`MyThread`类，该类扩展了`Thread`类。该类有三个属性，用于存储创建日期、执行开始日期和执行结束日期。使用开始日期和结束日期属性，您已经实现了`getExecutionTime()`方法，该方法返回线程执行任务的时间。最后，您已重写了`toString()`方法以生成有关线程的信息。

一旦您拥有自己的线程类，您就实现了一个工厂来创建实现`ThreadFactory`接口的该类的对象。如果您要将工厂用作独立对象，则不一定要使用接口，但是如果您想要将此工厂与 Java 并发 API 的其他类一起使用，则必须通过实现该接口来构建您的工厂。`ThreadFactory`接口只有一个方法，即`newThread()`方法，该方法接收一个`Runnable`对象作为参数，并返回一个`Thread`对象来执行该`Runnable`对象。在您的情况下，您返回一个`MyThread`对象。

要检查这两个类，您已经实现了实现`Runnable`对象的`MyTask`类。这是由`MyThread`对象管理的线程要执行的任务。`MyTask`实例将其执行线程休眠 2 秒。

在示例的主方法中，您使用`MyThreadFactory`工厂创建了一个`MyThread`对象来执行一个`Task`对象。执行程序，您将看到一个带有线程开始日期和执行时间的消息。

以下屏幕截图显示了此示例生成的输出：

![它是如何工作的...](img/7881_07_02.jpg)

## 还有更多...

Java 并发 API 提供了`Executors`类来生成线程执行器，通常是`ThreadPoolExecutor`类的对象。您还可以使用此类来获取`ThreadFactory`接口的最基本实现，使用`defaultThreadFactory()`方法。此方法生成的工厂生成基本的`Thread`对象，它们都属于同一个`ThreadGroup`对象。

您可以在程序中使用`ThreadFactory`接口进行任何目的，不一定与 Executor 框架相关。

# 在 Executor 对象中使用我们的 ThreadFactory

在前一篇文章中，*实现 ThreadFactory 接口以生成自定义线程*，我们介绍了工厂模式，并提供了如何实现实现`ThreadFactory`接口的线程工厂的示例。

Executor 框架是一种允许您分离线程创建和执行的机制。它基于`Executor`和`ExecutorService`接口以及实现这两个接口的`ThreadPoolExecutor`类。它具有内部线程池，并提供方法，允许您将两种类型的任务发送到池化线程中进行执行。这两种类型的任务是：

+   实现`Runnable`接口的类，以实现不返回结果的任务

+   实现`Callable`接口的类，以实现返回结果的任务

在内部，Executor 框架使用`ThreadFactory`接口来创建它用于生成新线程的线程。在本篇文章中，您将学习如何实现自己的线程类、线程工厂来创建该类的线程，以及如何在执行器中使用该工厂，以便执行器将执行您的线程。

## 准备就绪...

阅读前一篇文章，*实现 ThreadFactory 接口以生成自定义线程*，并实现其示例。

此示例已使用 Eclipse IDE 实现。如果您使用 Eclipse 或其他 IDE，如 NetBeans，请打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤来实现示例：

1.  将在*实现 ThreadFactory 接口以生成自定义线程*中实现的`MyThread`、`MyThreadFactory`和`MyTask`类复制到项目中，以便在此示例中使用它们。

1.  通过创建一个名为`Main`的类并实现一个`main()`方法来实现示例的主类。

```java
public class Main {
  public static void main(String[] args) throws Exception {
```

1.  创建一个名为`threadFactory`的新`MyThreadFactory`对象。

```java
    MyThreadFactory threadFactory=new MyThreadFactory("MyThreadFactory");
```

1.  使用`Executors`类的`newCachedThreadPool()`方法创建一个新的`Executor`对象。将之前创建的工厂对象作为参数传递。新的`Executor`对象将使用该工厂来创建必要的线程，因此它将执行`MyThread`线程。

```java
    ExecutorService executor=Executors.newCachedThreadPool(threadFactory);
```

1.  创建一个新的`Task`对象，并使用`submit()`方法将其发送到执行器。

```java
    MyTask task=new MyTask();
    executor.submit(task);
```

1.  使用`shutdown()`方法关闭执行器。

```java
    executor.shutdown();
```

1.  使用`awaitTermination()`方法等待执行器的完成。

```java
    executor.awaitTermination(1, TimeUnit.DAYS);
```

1.  写一条消息来指示程序的结束。

```java
    System.out.printf("Main: End of the program.\n");
```

## 它是如何工作的...

在前一个示例的*How it works...*部分，*实现 ThreadFactory 接口以生成自定义线程*中，您可以阅读有关`MyThread`、`MyThreadFactory`和`MyTask`类如何工作的详细解释。

在示例的`main()`方法中，使用`Executors`类的`newCachedThreadPool()`方法创建了一个`Executor`对象。您已将之前创建的工厂对象作为参数传递，因此创建的`Executor`对象将使用该工厂来创建所需的线程，并执行`MyThread`类的线程。

执行程序，您将看到一个关于线程启动日期和执行时间的信息。以下截图显示了此示例生成的输出：

![它是如何工作的...](img/7881_07_03.jpg)

## 另请参阅

+   在第七章的*自定义并发类*中的*实现 ThreadFactory 接口以生成自定义线程*食谱中

# 自定义在定时线程池中运行的任务

**定时线程池**是 Executor 框架的基本线程池的扩展，允许您安排任务在一段时间后执行。它由`ScheduledThreadPoolExecutor`类实现，并允许执行以下两种类型的任务：

+   **延迟任务**：这种类型的任务在一段时间后只执行一次

+   **周期性任务**：这种类型的任务在延迟后定期执行

延迟任务可以执行`Callable`和`Runnable`对象，但周期性任务只能执行`Runnable`对象。定时池执行的所有任务都是`RunnableScheduledFuture`接口的实现。在此示例中，您将学习如何实现自己的`RunnableScheduledFuture`接口的实现来执行延迟和周期性任务。

## 准备工作

此示例已使用 Eclipse IDE 实现。如果您使用 Eclipse 或其他 IDE，如 NetBeans，请打开它并创建一个新的 Java 项目。

## 如何做...

按照下面描述的步骤来实现示例：

1.  创建一个名为`MyScheduledTask`的类，参数化为一个名为`V`的泛型类型。它扩展了`FutureTask`类并实现了`RunnableScheduledFuture`接口。

```java
public class MyScheduledTask<V> extends FutureTask<V> implements RunnableScheduledFuture<V> {
```

1.  声明一个名为`task`的私有`RunnableScheduledFuture`属性。

```java
  private RunnableScheduledFuture<V> task;
```

1.  声明一个名为`executor`的私有`ScheduledThreadPoolExecutor`。

```java
  private ScheduledThreadPoolExecutor executor;
```

1.  声明一个名为`period`的私有`long`属性。

```java
  private long period;
```

1.  声明一个名为`startDate`的私有`long`属性。

```java
  private long startDate;
```

1.  实现一个类的构造函数。它接收一个将由任务执行的`Runnable`对象，将由此任务返回的结果，将用于创建`MyScheduledTask`对象的`RunnableScheduledFuture`任务，以及将执行任务的`ScheduledThreadPoolExecutor`对象。调用其父类的构造函数并存储任务和`executor`属性。

```java
  public MyScheduledTask(Runnable runnable, V result, RunnableScheduledFuture<V> task, ScheduledThreadPoolExecutor executor) {
    super(runnable, result);
    this.task=task;
    this.executor=executor;
  }
```

1.  实现`getDelay()`方法。如果任务是一个周期性任务，并且`startDate`属性的值不为零，则计算返回值为`startDate`属性和实际日期之间的差值。否则，返回存储在`task`属性中的原始任务的延迟。不要忘记以参数传递的时间单位返回结果。

```java
  @Override
  public long getDelay(TimeUnit unit) {
    if (!isPeriodic()) {
      return task.getDelay(unit);
    } else {
      if (startDate==0){
        return task.getDelay(unit);
      } else {
        Date now=new Date();
        long delay=startDate-now.getTime();
        return unit.convert(delay, TimeUnit.MILLISECONDS);
      }
    }
  }
```

1.  实现`compareTo()`方法。调用原始任务的`compareTo()`方法。

```java
  @Override
  public int compareTo(Delayed o) {
    return task.compareTo(o);
  }
```

1.  实现`isPeriodic()`方法。调用原始任务的`isPeriodic()`方法。

```java
  @Override
  public boolean isPeriodic() {
    return task.isPeriodic();
  }
```

1.  实现`run()`方法。如果是一个周期性任务，你必须更新它的`startDate`属性，以便将来执行任务的开始日期。计算方法是将实际日期和周期相加。然后，再次将任务添加到`ScheduledThreadPoolExecutor`对象的队列中。

```java
  @Override
  public void run() {
    if (isPeriodic() && (!executor.isShutdown())) {
      Date now=new Date();
      startDate=now.getTime()+period;
      executor.getQueue().add(this);
    }
```

1.  在控制台中打印一条带有实际日期的消息，调用`runAndReset()`方法执行任务，然后再次在控制台中打印一条带有实际日期的消息。

```java
    System.out.printf("Pre-MyScheduledTask: %s\n",new Date());
    System.out.printf("MyScheduledTask: Is Periodic: %s\n",isPeriodic());
    super.runAndReset();
    System.out.printf("Post-MyScheduledTask: %s\n",new Date());
  }
```

1.  实现`setPeriod()`方法来设定该任务的周期。

```java
  public void setPeriod(long period) {
    this.period=period;
  }
```

1.  创建一个名为`MyScheduledThreadPoolExecutor`的类，以实现执行`MyScheduledTask`任务的`ScheduledThreadPoolExecutor`对象。指定该类扩展`ScheduledThreadPoolExecutor`类。

```java
public class MyScheduledThreadPoolExecutor extends ScheduledThreadPoolExecutor {
```

1.  实现一个类的构造函数，它仅调用其父类的构造函数。

```java
  public MyScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize);
  }
```

1.  实现`decorateTask()`方法。它接收一个将要执行的`Runnable`对象和将执行该`Runnable`对象的`RunnableScheduledFuture`任务作为参数。使用这些对象创建并返回一个`MyScheduledTask`任务。

```java
  @Override
  protected <V> RunnableScheduledFuture<V> decorateTask(Runnable runnable,
      RunnableScheduledFuture<V> task) {
    MyScheduledTask<V> myTask=new MyScheduledTask<V>(runnable, null, task,this);  
    return myTask;
  }
```

1.  重写`scheduledAtFixedRate()`方法。调用其父类的方法，将返回的对象转换为`MyScheduledTask`对象，并使用`setPeriod()`方法设定该任务的周期。

```java
  @Override
  public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit) {
   ScheduledFuture<?> task= super.scheduleAtFixedRate(command, initialDelay, period, unit);
   MyScheduledTask<?> myTask=(MyScheduledTask<?>)task;
 myTask.setPeriod(TimeUnit.MILLISECONDS.convert(period,unit));
   return task;
 }
```

1.  创建一个名为`Task`的类，实现`Runnable`接口。

```java
public class Task implements Runnable {
```

1.  实现`run()`方法。在任务开始时打印一条消息，让当前线程休眠 2 秒，然后在任务结束时再打印一条消息。

```java
  @Override
  public void run() {
    System.out.printf("Task: Begin.\n");
    try {
      TimeUnit.SECONDS.sleep(2);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    System.out.printf("Task: End.\n");
  }
```

1.  通过创建一个名为`Main`的类和一个`main()`方法来实现示例的主类。

```java
public class Main {

  public static void main(String[] args) throws Exception{
```

1.  创建一个名为`executor`的`MyScheduledThreadPoolExecutor`对象。使用`2`作为参数，在池中有两个线程。

```java
    MyScheduledThreadPoolExecutor executor=new MyScheduledThreadPoolExecutor(2);
```

1.  创建一个名为`task`的`Task`对象。在控制台中写下实际日期。

```java
    Task task=new Task();
    System.out.printf("Main: %s\n",new Date());
```

1.  使用`schedule()`方法向执行器发送一个延迟任务。该任务将在 1 秒延迟后执行。

```java
    executor.schedule(task, 1, TimeUnit.SECONDS);
```

1.  让主线程休眠 3 秒。

```java
    TimeUnit.SECONDS.sleep(3);
```

1.  创建另一个`Task`对象。再次在控制台中打印实际日期。

```java
task=new Task();
    System.out.printf("Main: %s\n",new Date());
```

1.  使用`scheduleAtFixedRate()`方法向执行器发送一个周期性任务。该任务将在 1 秒延迟后执行，然后每 3 秒执行一次。

```java
    executor.scheduleAtFixedRate(task, 1, 3, TimeUnit.SECONDS);
```

1.  让主线程休眠 10 秒。

```java
    TimeUnit.SECONDS.sleep(10);
```

1.  使用`shutdown()`方法关闭执行器。使用`awaitTermination()`方法等待执行器的完成。

```java
    executor.shutdown();
    executor.awaitTermination(1, TimeUnit.DAYS);
```

1.  在控制台中写一条消息，指示程序结束。

```java
    System.out.printf("Main: End of the program.\n");
```

## 工作原理...

在这个示例中，您已经实现了`MyScheduledTask`类，以实现可以在`ScheduledThreadPoolExecutor`执行器上执行的自定义任务。该类扩展了`FutureTask`类并实现了`RunnableScheduledFuture`接口。它实现了`RunnableScheduledFuture`接口，因为在计划的执行器中执行的所有任务都必须实现该接口并扩展`FutureTask`类，因为该类提供了在`RunnableScheduledFuture`接口中声明的方法的有效实现。之前提到的所有接口和类都是参数化类，具有将由任务返回的数据类型。

要在计划的执行器中使用`MyScheduledTask`任务，您已经覆盖了`MyScheduledThreadPoolExecutor`类中的`decorateTask()`方法。该类扩展了`ScheduledThreadPoolExecutor`执行器，该方法提供了一种将`ScheduledThreadPoolExecutor`执行器实现的默认计划任务转换为`MyScheduledTask`任务的机制。因此，当您实现自己的计划任务版本时，必须实现自己的计划执行器的版本。

+   `decorateTask()`方法只是使用参数创建一个新的`MyScheduledTask`对象；一个将在任务中执行的`Runnable`对象。该任务将返回一个结果。在这种情况下，任务不会返回结果，因此使用了`null`值。原始任务用于执行`Runnable`对象。这是新对象将在池中替换的任务；将执行任务的执行器。在这种情况下，您使用`this`关键字引用创建任务的执行器。

`MyScheduledTask`类可以执行延迟和周期性任务。您已经实现了两种任务的所有必要逻辑的两种方法，它们是`getDelay()`和`run()`方法。

`scheduled executor`调用`getDelay()`方法来确定是否执行任务。此方法在延迟和周期性任务中的行为不同。正如我们之前提到的，`MyScheduledClass`类的构造函数接收原始的`ScheduledRunnableFuture`对象，该对象将执行`Runnable`对象，并将其存储为类的属性，以便访问其方法和数据。当要执行延迟任务时，`getDelay()`方法返回原始任务的延迟，但是对于周期性任务，`getDelay()`方法返回`startDate`属性与实际日期之间的差异。

`run()`方法是执行任务的方法。周期性任务的一个特点是，如果要再次执行任务，您必须将任务的下一次执行放入执行器的队列中作为新任务。因此，如果要执行周期性任务，您要确定`startDate`属性的值，将其添加到实际日期和任务执行的周期，并将任务再次存储在执行器的队列中。`startDate`属性存储了任务的下一次执行将开始的日期。然后，您使用`FutureTask`类提供的`runAndReset()`方法执行任务。对于延迟任务，您不必将它们放入执行器的队列中，因为它们只执行一次。

### 注意

您还必须考虑执行器是否已关闭。在这种情况下，您不必再将周期性任务存储到执行器的队列中。

最后，您已经覆盖了`MyScheduledThreadPoolExecutor`类中的`scheduleAtFixedRate()`方法。我们之前提到，对于周期性任务，您要使用任务的周期来确定`startDate`属性的值，但是您还没有初始化该周期。您必须覆盖此方法，该方法接收该周期作为参数，然后将其传递给`MyScheduledTask`类，以便它可以使用它。

该示例包括实现了`Runnable`接口的`Task`类，并且是在计划执行程序中执行的任务。示例的主类创建了一个`MyScheduledThreadPoolExecutor`执行程序，并将以下两个任务发送给它们：

+   一个延迟任务，1 秒后执行。

+   一个周期性任务，首次在实际日期后 1 秒执行，然后每 3 秒执行一次

以下屏幕截图显示了此示例的部分执行。您可以检查两种类型的任务是否被正确执行：

![它是如何工作的...](img/7881_07_04.jpg)

## 还有更多...

`ScheduledThreadPoolExecutor`类提供了`decorateTask()`方法的另一个版本，该方法接收`Callable`对象作为参数，而不是`Runnable`对象。

## 另请参阅

+   *在延迟后在执行者中运行任务*配方第四章, *线程执行者*

+   *在执行者中定期运行任务*配方第四章, *线程执行者*

# 实现 ThreadFactory 接口以为 Fork/Join 框架生成自定义线程

Java 7 最有趣的特性之一是 Fork/Join 框架。它是`Executor`和`ExecutorService`接口的实现，允许您执行`Callable`和`Runnable`任务，而无需管理执行它们的线程。

这个执行器旨在执行可以分成更小部分的任务。其主要组件如下：

+   `ForkJoinTask`类实现的一种特殊类型的任务。

+   将任务分成子任务的两个操作（`fork`操作）和等待这些子任务完成（`join`操作）。

+   一种算法，称为工作窃取算法，它优化了线程池中线程的使用。当一个任务正在等待其子任务时，执行它的线程被用来执行另一个线程。

Fork/Join 框架的主类是`ForkJoinPool`类。在内部，它有以下两个元素：

+   一个等待执行的任务队列

+   执行任务的线程池

在本示例中，您将学习如何实现一个自定义的工作线程，用于`ForkJoinPool`类，并使用工厂来使用它。

## 准备工作

这个示例的实现是使用 Eclipse IDE 完成的。如果您使用 Eclipse 或其他 IDE，如 NetBeans，打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`MyWorkerThread`的类，它扩展了`ForkJoinWorkerThread`类。

```java
public class MyWorkerThread extends ForkJoinWorkerThread {
```

1.  声明并创建一个私有的`ThreadLocal`属性，参数为`Integer`类，命名为`taskCounter`。

```java
  private static ThreadLocal<Integer> taskCounter=new ThreadLocal<Integer>();
```

1.  实现一个类的构造函数。

```java
  protected MyWorkerThread(ForkJoinPool pool) {
    super(pool);
  }
```

1.  重写`onStart()`方法。调用其父类的方法，在控制台中打印一条消息，并将此线程的`taskCounter`属性的值设置为零。

```java
  @Override
  protected void onStart() {
    super.onStart();
    System.out.printf("MyWorkerThread %d: Initializing task counter.\n",getId());
    taskCounter.set(0);
  }
```

1.  重写`onTermination()`方法。在控制台中写入此线程的`taskCounter`属性的值。

```java
  @Override
  protected void onTermination(Throwable exception) {
    System.out.printf("MyWorkerThread %d: %d\n",getId(),taskCounter.get());
    super.onTermination(exception);
  }
```

1.  实现`addTask()`方法。增加`taskCounter`属性的值。

```java
  public void addTask(){
    int counter=taskCounter.get().intValue();
    counter++;
    taskCounter.set(counter);
  }
```

1.  创建一个名为`MyWorkerThreadFactory`的类，它实现了`ForkJoinWorkerThreadFactory`接口。实现`newThread()`方法。创建并返回一个`MyWorkerThread`对象。

```java
public class MyWorkerThreadFactory implements ForkJoinWorkerThreadFactory {

  @Override
  public ForkJoinWorkerThread newThread(ForkJoinPool pool) {
    return new MyWorkerThread(pool);
  }

}
```

1.  创建一个名为`MyRecursiveTask`的类，它扩展了`Integer`类参数化的`RecursiveTask`类。

```java
public class MyRecursiveTask extends RecursiveTask<Integer> {
```

1.  声明一个名为`array`的私有`int`数组。

```java
  private int array[];
```

1.  声明两个私有的`int`属性，命名为`start`和`end`。

```java
  private int start, end;
```

1.  实现初始化其属性的类的构造函数。

```java
  public Task(int array[],int start, int end) {
    this.array=array;
    this.start=start;
    this.end=end;
  }
```

1.  实现`compute()`方法，对数组在开始和结束位置之间的所有元素求和。首先，将执行任务的线程转换为`MyWorkerThread`对象，并使用`addTask()`方法增加该线程的任务计数器。

```java
  @Override
  protected Integer compute() {
    Integer ret;
    MyWorkerThread thread=(MyWorkerThread)Thread.currentThread();
    thread.addTask();
  }
```

1.  实现`addResults()`方法。计算并返回作为参数接收的两个任务结果的总和。

```java
  private Integer addResults(Task task1, Task task2) {
    int value;
    try {
      value = task1.get().intValue()+task2.get().intValue();
    } catch (InterruptedException e) {
      e.printStackTrace();
      value=0;
    } catch (ExecutionException e) {
      e.printStackTrace();
      value=0;
    }
```

1.  让线程休眠 10 毫秒，并返回任务的结果。

```java
    try {
      TimeUnit.MILLISECONDS.sleep(10);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    return value;
  }
```

1.  通过创建一个名为`Main`的类和一个`main()`方法来实现示例的主类。

```java
public class Main {

  public static void main(String[] args) throws Exception {
```

1.  创建一个名为`factory`的`MyWorkerThreadFactory`对象。

```java
    MyWorkerThreadFactory factory=new MyWorkerThreadFactory();
```

1.  创建一个名为`pool`的`ForkJoinPool`对象。将之前创建的工厂对象传递给构造函数。

```java
    ForkJoinPool pool=new ForkJoinPool(4, factory, null, false);
```

1.  创建一个包含 100,000 个整数的数组。将所有元素初始化为`1`。

```java
    int array[]=new int[100000];

    for (int i=0; i<array.length; i++){
      array[i]=1;
    }
```

1.  创建一个新的`Task`对象来对数组的所有元素求和。

```java
    MyRecursiveTask task=new MyRecursiveTask(array,0,array.length);
```

1.  使用`execute()`方法将任务发送到池中。

```java
    pool.execute(task);
```

1.  使用`join()`方法等待任务结束。

```java
    task.join();
```

1.  使用`shutdown()`方法关闭池。

```java
    pool.shutdown();
```

1.  使用`awaitTermination()`方法等待执行器的完成。

```java
    pool.awaitTermination(1, TimeUnit.DAYS);
```

1.  使用`get()`方法在控制台中写入任务的结果。

```java
      System.out.printf("Main: Result: %d\n",task.get());    
```

1.  在控制台中写入一条消息，指示示例的结束。

```java
    System.out.printf("Main: End of the program\n");
```

## 它是如何工作的...

Fork/Join 框架使用的线程称为工作线程。Java 包括`ForkJoinWorkerThread`类，该类扩展了`Thread`类并实现了 Fork/Join 框架使用的工作线程。

在这个配方中，您已经实现了`MyWorkerThread`类，该类扩展了`ForkJoinWorkerThread`类并重写了该类的两个方法。您的目标是在每个工作线程中实现一个任务计数器，以便您知道每个工作线程执行了多少任务。您使用`ThreadLocal`属性实现了计数器。这样，每个线程都将以对程序员透明的方式拥有自己的计数器。

您已经重写了`ForkJoinWorkerThread`类的`onStart()`方法，以初始化任务计数器。当工作线程开始执行时，将调用此方法。您还重写了`onTermination()`方法，以将任务计数器的值打印到控制台。当工作线程完成执行时，将调用此方法。您还在`MyWorkerThread`类中实现了一个方法。`addTask()`方法增加每个线程的任务计数器。

`ForkJoinPool`类，与 Java 并发 API 中的所有执行程序一样，使用工厂创建其线程，因此如果要在`ForkJoinPool`类中使用`MyWorkerThread`线程，必须实现自己的线程工厂。对于 Fork/Join 框架，此工厂必须实现`ForkJoinPool.ForkJoinWorkerThreadFactory`类。您已经为此目的实现了`MyWorkerThreadFactory`类。这个类只有一个方法，用于创建一个新的`MyWorkerThread`对象。

最后，您只需使用您创建的工厂初始化一个`ForkJoinPool`类。您已经在`Main`类中使用`ForkJoinPool`类的构造函数完成了这一点。

以下屏幕截图显示了程序输出的一部分：

![它是如何工作...](img/7881_07_05.jpg)

您可以看到`ForkJoinPool`对象已经执行了四个工作线程，以及每个工作线程执行了多少任务。

## 还有更多...

请注意，`ForkJoinWorkerThread`类提供的`onTermination()`方法在线程正常完成或抛出`Exception`异常时调用。该方法接收一个`Throwable`对象作为参数。如果参数取`null`值，则工作线程正常完成，但如果参数取值，则线程抛出异常。您必须包含必要的代码来处理这种情况。

## 另请参阅

+   在第五章的*创建 Fork/Join 池*配方中，*Fork/Join 框架*

+   在第一章的*通过工厂创建线程*配方中，*线程管理*

# 自定义在 Fork/Join 框架中运行的任务

执行器框架将任务的创建和执行分开。您只需实现`Runnable`对象并使用`Executor`对象。将`Runnable`任务发送到执行器，它将创建、管理和完成执行这些任务所需的线程。

Java 7 提供了 Fork/Join 框架中的一种特殊的执行器。该框架旨在使用分而治之的技术解决可以分解为较小任务的问题。在任务内部，您必须检查要解决的问题的大小，如果大于设定的大小，则将问题分成两个或更多任务，并使用框架执行这些任务。如果问题的大小小于设定的大小，则直接在任务中解决问题，然后可选择地返回结果。Fork/Join 框架实现了改进这类问题整体性能的工作窃取算法。

Fork/Join 框架的主要类是`ForkJoinPool`类。在内部，它具有以下两个元素：

+   等待执行的任务队列

+   执行任务的线程池

默认情况下，由`ForkJoinPool`类执行的任务是`ForkJoinTask`类的对象。您还可以将`Runnable`和`Callable`对象发送到`ForkJoinPool`类，但它们无法充分利用 Fork/Join 框架的所有优势。通常，您将向`ForkJoinPool`对象发送`ForkJoinTask`类的两个子类之一的对象：

+   `RecursiveAction`：如果您的任务不返回结果

+   `RecursiveTask`：如果您的任务返回结果

在本示例中，您将学习如何为 Fork/Join 框架实现自己的任务，实现一个扩展了`ForkJoinTask`类的任务，该任务测量并在控制台中写入其执行时间，以便您可以控制其演变。您还可以实现自己的 Fork/Join 任务来写入日志信息，获取任务中使用的资源，或者对任务的结果进行后处理。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`MyWorkerTask`的类，并指定它扩展了参数为`Void`类型的`ForkJoinTask`类。

```java
public abstract class MyWorkerTask extends ForkJoinTask<Void> {
```

1.  声明一个名为`name`的私有`String`属性，用于存储任务的名称。

```java
  private String name;
```

1.  实现类的构造函数以初始化其属性。

```java
  public MyWorkerTask(String name) {
    this.name=name;
  }
```

1.  实现`getRawResult()`方法。这是`ForkJoinTask`类的抽象方法之一。由于`MyWorkerTask`任务不会返回任何结果，因此此方法必须返回`null`值。

```java
  @Override
  public Void getRawResult() {
    return null;
  }
```

1.  实现`setRawResult()`方法。这是`ForkJoinTask`类的另一个抽象方法。由于`MyWorkerTask`任务不会返回任何结果，因此将此方法的主体留空。

```java
  @Override
  protected void setRawResult(Void value) {

  }
```

1.  实现`exec()`方法。这是任务的主要方法。在这种情况下，将任务的逻辑委托给`compute()`方法。计算该方法的执行时间并将其写入控制台。

```java
  @Override
  protected boolean exec() {
    Date startDate=new Date();
    compute();
    Date finishDate=new Date();
    long diff=finishDate.getTime()-startDate.getTime();
    System.out.printf("MyWorkerTask: %s : %d Milliseconds to complete.\n",name,diff);
    return true;
  }
```

1.  实现`getName()`方法以返回任务的名称。

```java
  public String getName(){
    return name;
  }
```

1.  声明抽象方法`compute()`。如前所述，此方法将实现任务的逻辑，并且必须由`MyWorkerTask`类的子类实现。

```java
  protected abstract void compute();
```

1.  创建一个名为`Task`的类，该类扩展了`MyWorkerTask`类。

```java
public class Task extends MyWorkerTask {
```

1.  声明一个名为`array`的私有`int`值数组。

```java
  private int array[];
```

1.  实现一个初始化其属性的类的构造函数。

```java
  public Task(String name, int array[], int start, int end){
    super(name);
    this.array=array;
    this.start=start;
    this.end=end;
  }
```

1.  实现`compute()`方法。此方法增加由开始和结束属性确定的数组元素块。如果此元素块的元素超过 100 个，则将该块分成两部分，并创建两个`Task`对象来处理每个部分。使用`invokeAll()`方法将这些任务发送到池中。

```java
  protected void compute() {
    if (end-start>100){
      int mid=(end+start)/2;
      Task task1=new Task(this.getName()+"1",array,start,mid);
      Task task2=new Task(this.getName()+"2",array,mid,end);
      invokeAll(task1,task2);
```

1.  如果元素块少于 100 个，则使用`for`循环增加所有元素。

```java
    } else {
      for (int i=start; i<end; i++) {
        array[i]++;
      }
```

1.  最后，让执行任务的线程休眠 50 毫秒。

```java
      try {
        Thread.sleep(50);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
  }
```

1.  通过创建一个名为`Main`的类并实现一个`main()`方法来实现示例的主类。

```java
public class Main {
  public static void main(String[] args) throws Exception {
```

1.  创建一个包含 10,000 个元素的`int`数组。

```java
    int array[]=new int[10000];
```

1.  创建一个名为`pool`的`ForkJoinPool`对象。

```java
    ForkJoinPool pool=new ForkJoinPool();
```

1.  创建一个`Task`对象来增加数组的所有元素。构造函数的参数是`Task`作为任务的名称，数组对象和值`0`和`10000`，以指示该任务必须处理整个数组。

```java
    Task task=new Task("Task",array,0,array.length);
```

1.  使用`execute()`方法将任务发送到池中。

```java
    pool.invoke(task);
```

1.  使用`shutdown()`方法关闭池。

```java
    pool.shutdown();
```

1.  在控制台中写入一条消息，指示程序的结束。

```java
    System.out.printf("Main: End of the program.\n");
```

## 工作原理...

在此配方中，您已实现了`MyWorkerTask`类，该类扩展了`ForkJoinTask`类。这是您自己的基类，用于实现可以在`ForkJoinPool`执行程序中执行并且可以利用该执行程序的所有优势的任务，如工作窃取算法。该类相当于`RecursiveAction`和`RecursiveTask`类。

当您扩展`ForkJoinTask`类时，您必须实现以下三种方法：

+   `setRawResult()`: 此方法用于设置任务的结果。由于您的任务不返回任何结果，因此您将此方法留空。

+   `getRawResult()`: 此方法用于返回任务的结果。由于您的任务不返回任何结果，因此此方法返回`null`值。

+   `exec()`: 此方法实现任务的逻辑。在您的情况下，您已将逻辑委托给抽象方法`compute()`（如`RecursiveAction`和`RecursiveTask`类），并且在`exec()`方法中测量该方法的执行时间，并将其写入控制台。

最后，在示例的主类中，您已创建了一个包含 10,000 个元素的数组，一个`ForkJoinPool`执行程序和一个`Task`对象来处理整个数组。执行程序，您将看到执行的不同任务如何在控制台中写入它们的执行时间。

## 另请参阅

+   第五章中的*创建 Fork/Join 池*配方，*Fork/Join Framework*

+   第七章中的*实现 ThreadFactory 接口以为 Fork/Join 框架生成自定义线程*配方，*自定义并发类*

# 实现自定义锁类

锁是 Java 并发 API 提供的基本同步机制之一。它允许程序员保护代码的临界区，因此只有一个线程可以一次执行该代码块。它提供以下两个操作：

+   `lock()`: 当您想要访问临界区时，调用此操作。如果有另一个线程正在运行该临界区，则其他线程将被阻塞，直到它们被锁唤醒以访问临界区。

+   `unlock()`: 在临界区的末尾调用此操作，以允许其他线程访问临界区。

在 Java 并发 API 中，锁在`Lock`接口中声明，并在一些类中实现，例如`ReentrantLock`类。

在此配方中，您将学习如何实现自己的`Lock`对象，该对象实现了实现`Lock`接口的类，可用于保护临界区。

## 准备工作

此示例已使用 Eclipse IDE 实现。如果您使用 Eclipse 或其他 IDE（如 NetBeans），请打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`MyQueuedSynchronizer`的类，它继承`AbstractQueuedSynchronizer`类。

```java
public class MyAbstractQueuedSynchronizer extends AbstractQueuedSynchronizer {
```

1.  声明一个名为`state`的私有`AtomicInteger`属性。

```java
  private AtomicInteger state;
```

1.  实现类的构造函数以初始化其属性。

```java
  public MyAbstractQueuedSynchronizer() {
    state=new AtomicInteger(0);
  }
```

1.  实现`tryAcquire()`方法。此方法尝试将状态变量的值从零更改为一。如果可以，它返回`true`值，否则返回`false`。

```java
  @Override
  protected boolean tryAcquire(int arg) {
    return state.compareAndSet(0, 1);
  }
```

1.  实现`tryRelease()`方法。此方法尝试将状态变量的值从一更改为零。如果可以，则返回`true`值，否则返回`false`值。

```java
  @Override
  protected boolean tryRelease(int arg) {
    return state.compareAndSet(1, 0);
  }
```

1.  创建一个名为`MyLock`的类，并指定它实现`Lock`接口。

```java
public class MyLock implements Lock{
```

1.  声明一个名为`sync`的私有`AbstractQueuedSynchronizer`属性。

```java
  private AbstractQueuedSynchronizer sync;
```

1.  通过使用新的`MyAbstractQueueSynchronizer`对象初始化`sync`属性来实现类的构造函数。

```java
  public MyLock() {
    sync=new MyAbstractQueuedSynchronizer();
  }
```

1.  实现`lock()`方法。调用`sync`对象的`acquire()`方法。

```java
  @Override
  public void lock() {
    sync.acquire(1);
  }
```

1.  实现`lockInterruptibly()`方法。调用`sync`对象的`acquireInterruptibly()`方法。

```java
  @Override
  public void lockInterruptibly() throws InterruptedException {
    sync.acquireInterruptibly(1);
  }
```

1.  实现`tryLock()`方法。调用`sync`对象的`tryAcquireNanos()`方法。

```java
  @Override
  public boolean tryLock() {
    try {
      return sync.tryAcquireNanos(1, 1000);
    } catch (InterruptedException e) {
      e.printStackTrace();
      return false;
    }
  }
```

1.  使用两个参数实现`tryLock()`方法的另一个版本。一个名为`time`的长参数和一个名为`unit`的`TimeUnit`参数。调用`sync`对象的`tryAcquireNanos()`方法。

```java
  @Override
  public boolean tryLock(long time, TimeUnit unit)
      throws InterruptedException {
    return sync.tryAcquireNanos(1, TimeUnit.NANOSECONDS.convert(time, unit));
  }
```

1.  实现`unlock()`方法。调用`sync`对象的`release()`方法。

```java
  @Override
  public void unlock() {
    sync.release(1);
  }
```

1.  实现`newCondition()`方法。创建`sync`对象的内部类`ConditionObject`的新对象。

```java
  @Override
  public Condition newCondition() {
    return sync.new ConditionObject();
  }
```

1.  创建一个名为`Task`的类，并指定它实现`Runnable`接口。

```java
public class Task implements Runnable {
```

1.  声明一个名为`lock`的私有`MyLock`属性。

```java
  private MyLock lock;
```

1.  声明一个名为`name`的私有`String`属性。

```java
  private String name;
```

1.  实现类的构造函数以初始化其属性。

```java
  public Task(String name, MyLock lock){
    this.lock=lock;
    this.name=name;
  }
```

1.  实现类的`run()`方法。获取锁，使线程休眠 2 秒，然后释放`lock`对象。

```java
  @Override
  public void run() {
    lock.lock();
    System.out.printf("Task: %s: Take the lock\n",name);
    try {
      TimeUnit.SECONDS.sleep(2);
      System.out.printf("Task: %s: Free the lock\n",name);
    } catch (InterruptedException e) {
      e.printStackTrace();
    } finally {
      lock.unlock();
    }
  }
```

1.  通过创建一个名为`Main`的类并实现一个`main()`方法来实现示例的主类。

```java
public class Main {
  public static void main(String[] args) {
```

1.  创建一个名为`lock`的`MyLock`对象。

```java
    MyLock lock=new MyLock();
```

1.  创建并执行 10 个`Task`任务。

```java
    for (int i=0; i<10; i++){
      Task task=new Task("Task-"+i,lock);
      Thread thread=new Thread(task);
      thread.start();
    }
```

1.  尝试使用`tryLock()`方法获取锁。等待一秒钟，如果没有获得锁，则写一条消息并重试。

```java
    boolean value;
    do {
      try {
        value=lock.tryLock(1,TimeUnit.SECONDS);
        if (!value) {
          System.out.printf("Main: Trying to get the Lock\n");
        }
      } catch (InterruptedException e) {
        e.printStackTrace();
        value=false;
      }
    } while (!value);
```

1.  写一条消息指示您已获得锁并释放它。

```java
    System.out.printf("Main: Got the lock\n");
    lock.unlock();
```

1.  写一条消息指示程序结束。

```java
    System.out.printf("Main: End of the program\n");
```

## 它是如何工作的...

Java 并发 API 提供了一个类，可用于实现具有锁或信号量特性的同步机制。它是`AbstractQueuedSynchronizer`，正如其名称所示，它是一个抽象类。它提供了控制对临界区的访问以及管理被阻塞等待对临界区的访问的线程队列的操作。这些操作基于两个抽象方法：

+   `tryAcquire()`:调用此方法尝试访问临界区。如果调用它的线程可以访问临界区，则该方法返回`true`值。否则，该方法返回`false`值。

+   `tryRelease()`:调用此方法尝试释放对临界区的访问。如果调用它的线程可以释放访问，则该方法返回`true`值。否则，该方法返回`false`值。

在这些方法中，您必须实现用于控制对临界区的访问的机制。在您的情况下，您已经实现了`MyQueuedSynchonizer`类，该类扩展了`AbstractQueuedSyncrhonizer`类，并使用`AtomicInteger`变量实现了抽象方法，以控制对临界区的访问。如果锁是空闲的，该变量将存储`0`值，因此线程可以访问临界区，如果锁被阻塞，该变量将存储`1`值，因此线程无法访问临界区。

您已经使用了`AtomicInteger`类提供的`compareAndSet()`方法，该方法尝试将您指定为第一个参数的值更改为您指定为第二个参数的值。要实现`tryAcquire()`方法，您尝试将原子变量的值从零更改为一。同样，要实现`tryRelease()`方法，您尝试将原子变量的值从一更改为零。

你必须实现这个类，因为`AbstractQueuedSynchronizer`类的其他实现（例如`ReentrantLock`类使用的实现）是作为私有类在使用它的类内部实现的，所以你无法访问它。

然后，你已经实现了`MyLock`类。这个类实现了`Lock`接口，并有一个`MyQueuedSynchronizer`对象作为属性。为了实现`Lock`接口的所有方法，你使用了`MyQueuedSynchronizer`对象的方法。

最后，你已经实现了`Task`类，它实现了`Runnable`接口，并使用`MyLock`对象来访问临界区。该临界区使线程休眠 2 秒。主类创建了一个`MyLock`对象，并运行了 10 个共享该锁的`Task`对象。主类还尝试使用`tryLock()`方法来访问锁。

当你执行示例时，你会看到只有一个线程可以访问临界区，当该线程完成时，另一个线程将获得访问权限。

你可以使用自己的`Lock`来编写关于其使用情况的日志消息，控制锁定的时间，或实现高级的同步机制，例如控制对资源的访问，使其只在特定时间可用。

## 还有更多...

`AbstractQueuedSynchronizer`类提供了两个方法来管理锁的状态。它们是`getState()`和`setState()`方法。这些方法接收并返回一个整数值，表示锁的状态。你可以使用这些方法来代替`AtomicInteger`属性来存储锁的状态。

Java 并发 API 提供了另一个类来实现同步机制。它是`AbstractQueuedLongSynchronizer`类，它相当于`AbstractQueuedSynchronizer`类，但使用`long`属性来存储线程的状态。

## 另请参阅

+   在第二章的*Synchronizing a block of code with locks*食谱中，*基本线程同步*

# 基于优先级实现传输队列

Java 7 API 提供了几种数据结构来处理并发应用程序。其中，我们想要强调以下两种数据结构：

+   `LinkedTransferQueue`：这种数据结构应该在具有生产者/消费者结构的程序中使用。在这些应用程序中，你有一个或多个数据的生产者和一个或多个数据的消费者，一个数据结构被所有人共享。生产者将数据放入数据结构，消费者从数据结构中取数据。如果数据结构为空，消费者将被阻塞，直到有数据可供消费。如果数据结构已满，生产者将被阻塞，直到有空间放置他们的数据。

+   `PriorityBlockingQueue`：在这种数据结构中，元素以有序方式存储。元素必须实现带有`compareTo()`方法的`Comparable`接口。当你将一个元素插入结构中时，它会与结构中的元素进行比较，直到找到它的位置。

`LinkedTransferQueue`的元素按照它们到达的顺序存储，所以先到达的元素先被消耗。当你想要开发一个生产者/消费者程序时，数据根据某种优先级而不是到达时间进行消耗时，可能会出现这种情况。在这个示例中，你将学习如何实现一个数据结构，用于解决生产者/消费者问题，其元素将按照它们的优先级进行排序。具有更高优先级的元素将首先被消耗。

## 准备工作

这个示例已经在 Eclipse IDE 中实现。如果你使用 Eclipse 或其他 IDE，比如 NetBeans，打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`MyPriorityTransferQueue`的类，该类扩展了`PriorityBlockingQueue`类并实现了`TransferQueue`接口。

```java
public class MyPriorityTransferQueue<E> extends PriorityBlockingQueue<E> implements
    TransferQueue<E> {
```

1.  声明一个私有的`AtomicInteger`属性，名为`counter`，用于存储等待消费元素的消费者数量。

```java
  private AtomicInteger counter;
```

1.  声明一个私有的`LinkedBlockingQueue`属性，名为`transferred`。

```java
  private LinkedBlockingQueue<E> transfered;
```

1.  声明一个私有的`ReentrantLock`属性，名为`lock`。

```java
  private ReentrantLock lock;
```

1.  实现类的构造函数以初始化其属性。

```java
  public MyPriorityTransferQueue() {
    counter=new AtomicInteger(0);
    lock=new ReentrantLock();
    transfered=new LinkedBlockingQueue<E>();
  }
```

1.  实现`tryTransfer()`方法。该方法尝试立即将元素发送给等待的消费者，如果可能的话。如果没有等待的消费者，该方法返回`false`值。

```java
  @Override
  public boolean tryTransfer(E e) {
    lock.lock();
    boolean value;
    if (counter.get()==0) {
      value=false;
    } else {
      put(e);
      value=true;
    }
    lock.unlock();
    return value;
  }
```

1.  实现`transfer()`方法。该方法尝试立即将元素发送给等待的消费者，如果可能的话。如果没有等待的消费者，该方法将元素存储在一个特殊的队列中，以便发送给尝试获取元素并阻塞线程直到元素被消费的第一个消费者。

```java
  @Override
  public void transfer(E e) throws InterruptedException {
    lock.lock();
    if (counter.get()!=0) {
      put(e);
      lock.unlock();
    } else {
      transfered.add(e);
      lock.unlock();
      synchronized (e) {
        e.wait();
      }
    }
  }
```

1.  实现`tryTransfer()`方法，该方法接收三个参数：元素、等待消费者的时间（如果没有）和用于指定时间的时间单位。如果有等待的消费者，它立即发送元素。否则，将指定的时间转换为毫秒，并使用`wait()`方法使线程进入休眠状态。当消费者取走元素时，如果线程正在`wait()`方法中休眠，你将使用`notify()`方法唤醒它，稍后会看到。

```java
  @Override
  public boolean tryTransfer(E e, long timeout, TimeUnit unit)
      throws InterruptedException {
    lock.lock();
    if (counter.get()!=0) {
      put(e);
      lock.unlock();
      return true;
    } else {
      transfered.add(e);
      long newTimeout= TimeUnit.MILLISECONDS.convert(timeout, unit);
      lock.unlock();
      e.wait(newTimeout);
      lock.lock();
      if (transfered.contains(e)) {
        transfered.remove(e);
        lock.unlock();
        return false;
      } else {
        lock.unlock();
        return true;
      }
    }
  }
```

1.  实现`hasWaitingConsumer()`方法。使用计数属性的值来计算该方法的返回值。如果计数大于零，则返回`true`。否则，返回`false`。

```java
  @Override
  public boolean hasWaitingConsumer() {
    return (counter.get()!=0);
  }
```

1.  实现`getWaitingConsumerCount()`方法。返回`counter`属性的值。

```java
  @Override
  public int getWaitingConsumerCount() {
    return counter.get();
  }
```

1.  实现`take()`方法。消费者想要消费元素时调用此方法。首先获取之前定义的锁，并增加等待消费者的数量。

```java
  @Override
  public E take() throws InterruptedException {
    lock.lock();
    counter.incrementAndGet();
```

1.  如果在转移队列中没有任何元素，则释放锁并尝试从队列中获取元素，使用`take()`方法，并再次获取锁。如果队列中没有任何元素，该方法将使线程进入休眠状态，直到有元素可供消费。

```java
    E value=transfered.poll();
    if (value==null) {
      lock.unlock();
      value=super.take();
      lock.lock();
```

1.  否则，从转移队列中取出元素，并唤醒正在等待消费该元素的线程（如果有的话）。

```java
    } else {
      synchronized (value) {
        value.notify();
      }
    }
```

1.  最后，减少等待消费者的计数并释放锁。

```java
    counter.decrementAndGet();
    lock.unlock();
    return value;
  }
```

1.  实现一个名为`Event`的类，该类实现了参数化为`Event`类的`Comparable`接口。

```java
public class Event implements Comparable<Event> {
```

1.  声明一个私有的`String`属性，名为`thread`，用于存储创建事件的线程的名称。

```java
  private String thread;
```

1.  声明一个私有的`int`属性，名为`priority`，用于存储事件的优先级。

```java
  private int priority;
```

1.  实现类的构造函数以初始化其属性。

```java
  public Event(String thread, int priority){
    this.thread=thread;
    this.priority=priority;
  }
```

1.  实现一个方法来返回`thread`属性的值。

```java
  public String getThread() {
    return thread;
  }
```

1.  实现一个方法来返回`priority`属性的值。

```java
  public int getPriority() {
    return priority;
  }
```

1.  实现`compareTo()`方法。该方法将实际事件与作为参数接收的事件进行比较。如果实际事件的优先级高于参数，则返回`-1`，如果实际事件的优先级低于参数，则返回`1`，如果两个事件具有相同的优先级，则返回`0`。你将按优先级降序获得列表。优先级较高的事件将首先存储在队列中。

```java
  public int compareTo(Event e) {
    if (this.priority>e.getPriority()) {
      return -1;
    } else if (this.priority<e.getPriority()) {
      return 1; 
    } else {
      return 0;
    }
  }
```

1.  实现一个名为`Producer`的类，该类实现了`Runnable`接口。

```java
public class Producer implements Runnable {
```

1.  声明一个私有的`MyPriorityTransferQueue`属性，参数化为`Event`类，名为`buffer`，用于存储生产者生成的事件。

```java
  private MyPriorityTransferQueue<Event> buffer;
```

1.  实现类的构造函数以初始化其属性。

```java
  public Producer(MyPriorityTransferQueue<Event> buffer) {
    this.buffer=buffer;
  }
```

1.  实现类的`run()`方法。使用创建顺序作为优先级创建 100 个`Event`对象（最新的事件将具有最高优先级），并使用`put()`方法将它们插入队列。

```java
  @Override
  public void run() {
    for (int i=0; i<100; i++) {
      Event event=new Event(Thread.currentThread().getName(),i);
      buffer.put(event);
    }
  }
```

1.  实现一个名为`Consumer`的类，该类实现了`Runnable`接口。

```java
public class Consumer implements Runnable {
```

1.  声明一个私有的`MyPriorityTransferQueue`属性，参数化为`Event`类，命名为 buffer，以获取此类消耗的事件。

```java
  private MyPriorityTransferQueue<Event> buffer;
```

1.  实现类的构造函数以初始化其属性。

```java
  public Consumer(MyPriorityTransferQueue<Event> buffer) {
    this.buffer=buffer;
  }
```

1.  实现`run()`方法。它使用`take()`方法消耗 1002 个`Events`（在示例中生成的所有事件），并在控制台中写入生成事件的线程编号和其优先级。

```java
  @Override
  public void run() {
    for (int i=0; i<1002; i++) {
      try {
        Event value=buffer.take();
        System.out.printf("Consumer: %s: %d\n",value.getThread(),value.getPriority());
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
  }
```

1.  通过创建一个名为`Main`的类和一个`main()`方法来实现示例的主类。

```java
public class Main {

  public static void main(String[] args) throws Exception {
```

1.  创建一个名为`buffer`的`MyPriorityTransferQueue`对象。

```java
    MyPriorityTransferQueue<Event> buffer=new MyPriorityTransferQueue<Event>();
```

1.  创建一个`Producer`任务并启动 10 个线程来执行该任务。

```java
    Producer producer=new Producer(buffer);

    Thread producerThreads[]=new Thread[10];
    for (int i=0; i<producerThreads.length; i++) {
      producerThreads[i]=new Thread(producer);
      producerThreads[i].start();
    }
```

1.  创建并启动一个`Consumer`任务。

```java
    Consumer consumer=new Consumer(buffer);
    Thread consumerThread=new Thread(consumer);
    consumerThread.start();
```

1.  在控制台中写入实际消费者计数。

```java
    System.out.printf("Main: Buffer: Consumer count: %d\n",buffer.getWaitingConsumerCount());
```

1.  使用`transfer()`方法向消费者传输事件。

```java
    Event myEvent=new Event("Core Event",0);
    buffer.transfer(myEvent);
    System.out.printf("Main: My Event has ben transfered.\n");
```

1.  使用`join()`方法等待生产者的完成。

```java
    for (int i=0; i<producerThreads.length; i++) {
      try {
        producerThreads[i].join();
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
```

1.  使线程休眠 1 秒。

```java
      TimeUnit.SECONDS.sleep(1);  
```

1.  写入实际消费者计数。

```java
    System.out.printf("Main: Buffer: Consumer count: %d\n",buffer.getWaitingConsumerCount());
```

1.  使用`transfer()`方法传输另一个事件。

```java
    myEvent=new Event("Core Event 2",0);
      buffer.transfer(myEvent);  
```

1.  使用`join()`方法等待消费者的完成。

```java
      consumerThread.join();
```

1.  写一条消息指示程序的结束。

```java
    System.out.printf("Main: End of the program\n");
```

## 工作原理...

在这个示例中，您已经实现了`MyPriorityTransferQueue`数据结构。这是一个用于生产者/消费者问题的数据结构，但其元素按优先级而不是到达顺序排序。由于 Java 不允许多重继承，您的第一个决定是`MyPriorityTransferQueue`类的基类。您扩展了`PriorityBlockingQueue`类，以实现按优先级将元素插入结构的操作。您还实现了`TransferQueue`接口以添加与生产者/消费者相关的方法。

`MyPriortyTransferQueue`类具有以下三个属性：

+   一个名为`counter`的`AtomicInteger`属性：此属性存储等待从数据结构中获取元素的消费者数量。当消费者调用`take()`操作从数据结构中获取元素时，计数器会递增。当消费者完成`take()`操作的执行时，计数器再次递减。此计数器用于实现`hasWaitingConsumer()`和`getWaitingConsumerCount()`方法。

+   名为`lock`的`ReentrantLock`属性：此属性用于控制对实现的操作的访问。只有一个线程可以使用数据结构。

+   最后，创建一个`LinkedBlockingQueue`列表来存储传输的元素。

您已经在`MyPriorityTransferQueue`中实现了一些方法。所有这些方法都在`TransferQueue`接口中声明，并且`take()`方法在`PriorityBlockingQueue`接口中实现。其中两个已在前面描述过。以下是其余方法的描述：

+   `tryTransfer(E``e)`:此方法尝试直接将一个元素发送给消费者。如果有消费者在等待，该方法将元素存储在优先级队列中，以便立即被消费者消费，然后返回`true`值。如果没有消费者在等待，该方法返回`false`值。

+   `transfer(E``e)`:此方法直接将一个元素传输给消费者。如果有消费者在等待，该方法将元素存储在优先级队列中，以便立即被消费者消费。否则，该元素将存储在传输元素的列表中，并且线程将被阻塞，直到元素被消费。当线程处于休眠状态时，您必须释放锁，否则会阻塞队列。

+   `tryTransfer(E``e,``long``timeout,``TimeUnit``unit)`:此方法类似于`transfer()`方法，但线程会阻塞由其参数确定的时间段。当线程处于休眠状态时，您必须释放锁，否则会阻塞队列。

+   `take()`: 该方法返回要消耗的下一个元素。如果传输元素列表中有元素，则要消耗的元素将从该列表中取出。否则，它将从优先级队列中取出。

实现数据结构后，您已经实现了`Event`类。这是您在数据结构中存储的元素的类。`Event`类有两个属性，用于存储生产者的 ID 和事件的优先级，并实现了`Comparable`接口，因为这是数据结构的要求。

然后，您已经实现了`Producer`和`Consumer`类。在示例中，您有 10 个生产者和一个消费者，它们共享相同的缓冲区。每个生产者生成 100 个具有增量优先级的事件，因此具有更高优先级的事件是最后生成的。

示例的主类创建了一个`MyPriorityTransferQueue`对象，10 个生产者和一个消费者，并使用`MyPriorityTransferQueue`缓冲区的`transfer()`方法将两个事件传输到缓冲区。

以下屏幕截图显示了程序执行的部分输出：

![工作原理...](img/7881_07_06.jpg)

您可以看到，具有更高优先级的事件首先被消耗，并且消费者消耗了传输的事件。

## 另请参阅

+   第六章中的*使用按优先级排序的阻塞线程安全列表*配方，*并发集合*

+   第六章中的*使用阻塞线程安全列表*配方，*并发集合*

# 实现自己的原子对象

原子变量是在 Java 版本 5 中引入的，它们对单个变量提供原子操作。当线程对原子变量进行操作时，类的实现包括一个机制来检查操作是否在一步中完成。基本上，该操作获取变量的值，将值更改为本地变量，然后尝试将旧值更改为新值。如果旧值仍然相同，则进行更改。如果不是，则该方法重新开始操作。

在本配方中，您将学习如何扩展原子对象以及如何实现遵循原子对象机制的两个操作，以确保所有操作都在一步中完成。

## 准备就绪

本配方的示例是使用 Eclipse IDE 实现的。如果您使用 Eclipse 或其他 IDE（如 NetBeans），请打开它并创建一个新的 Java 项目。

## 操作步骤...

按照以下步骤实现示例：

1.  创建一个名为`ParkingCounter`的类，并指定它扩展`AtomicInteger`类。

```java
public class ParkingCounter extends AtomicInteger {
```

1.  声明一个名为`maxNumber`的私有`int`属性，以存储停车场中允许的最大汽车数量。

```java
  private int maxNumber;
```

1.  实现类的构造函数以初始化其属性。

```java
  public ParkingCounter(int maxNumber){
    set(0);
    this.maxNumber=maxNumber;
  }
```

1.  实现`carIn()`方法。如果计数器的值小于设定的最大值，则该方法递增汽车的计数器。构建一个无限循环，并使用`get()`方法获取内部计数器的值。

```java
  public boolean carIn() {
    for (;;) {
      int value=get();
```

1.  如果该值等于`maxNumber`属性，则无法递增计数器（停车场已满，汽车无法进入）。该方法返回`false`值。

```java
      if (value==maxNumber) {
        System.out.printf("ParkingCounter: The parking lot is full.\n");
        return false;
```

1.  否则，增加该值并使用`compareAndSet()`方法将旧值更改为新值。该方法返回`false`值；计数器未递增，因此必须重新开始循环。如果返回`true`值，则表示已进行更改，然后返回`true`值。

```java
      } else {
        int newValue=value+1;
        boolean changed=compareAndSet(value,newValue);
        if (changed) {
          System.out.printf("ParkingCounter: A car has entered.\n");
          return true;
        }
      }
    }
  }
```

1.  实现`carOut()`方法。如果计数器的值大于`0`，则该方法递减汽车的计数器。构建一个无限循环，并使用`get()`方法获取内部计数器的值。

```java
  public boolean carOut() {
    for (;;) {
      int value=get();
      if (value==0) {
        System.out.printf("ParkingCounter: The parking lot is empty.\n");
        return false;
      } else {
        int newValue=value-1;
        boolean changed=compareAndSet(value,newValue);
        if (changed) {
        System.out.printf("ParkingCounter: A car has gone out.\n");
          return true;
        }
      }
    }
  }
```

1.  创建一个名为`Sensor1`的类，该类实现`Runnable`接口。

```java
public class Sensor1 implements Runnable {
```

1.  声明一个名为`counter`的私有`ParkingCounter`属性。

```java
  private ParkingCounter counter;
```

1.  实现类的构造函数以初始化其属性。

```java
  public Sensor1(ParkingCounter counter) {
    this.counter=counter;
  }
```

1.  实现`run()`方法。多次调用`carIn()`和`carOut()`操作。

```java
   @Override
  public void run() {
    counter.carIn();
    counter.carIn();
    counter.carIn();
    counter.carIn();
    counter.carOut();
    counter.carOut();
    counter.carOut();
    counter.carIn();
    counter.carIn();
    counter.carIn();
  }
```

1.  创建一个名为`Sensor2`的类，实现`Runnable`接口。

```java
public class Sensor2 implements Runnable {
```

1.  声明一个名为`counter`的私有`ParkingCounter`属性。

```java
  private ParkingCounter counter;
```

1.  实现类的构造函数以初始化其属性。

```java
  public Sensor2(ParkingCounter counter) {
    this.counter=counter;
  }
```

1.  实现`run()`方法。多次调用`carIn()`和`carOut()`操作。

```java
   @Override
  public void run() {
    counter.carIn();
    counter.carOut();
    counter.carOut();
    counter.carIn();
    counter.carIn();
    counter.carIn();
    counter.carIn();
    counter.carIn();
    counter.carIn();
  }
```

1.  通过创建一个名为`Main`的类并实现一个`main()`方法来实现示例的主类。

```java
public class Main {

  public static void main(String[] args) throws Exception {
```

1.  创建名为`counter`的`ParkingCounter`对象。

```java
    ParkingCounter counter=new ParkingCounter(5);
```

1.  创建并启动一个`Sensor1`任务和一个`Sensor2`任务。

```java
    Sensor1 sensor1=new Sensor1(counter);
    Sensor2 sensor2=new Sensor2(counter);

    Thread thread1=new Thread(sensor1);
    Thread thread2=new Thread(sensor2);

    thread1.start();
    thread2.start();
```

1.  等待两个任务的最终确定。

```java
    thread1.join();
    thread2.join();
```

1.  在控制台中写入计数器的实际值。

```java
    System.out.printf("Main: Number of cars: %d\n",counter.get());
```

1.  在控制台中写入指示程序结束的消息。

```java
    System.out.printf("Main: End of the program.\n");
```

## 工作原理...

`ParkingCounter`类扩展了`AtomicInteger`类，具有两个原子操作`carIn()`和`carOut()`。该示例模拟了一个控制停车场内汽车数量的系统。停车场可以容纳一定数量的汽车，由`maxNumber`属性表示。

`carIn()`操作将停车场内的实际汽车数量与最大值进行比较。如果它们相等，则汽车无法进入停车场，该方法返回`false`值。否则，它使用原子操作的以下结构：

1.  将原子对象的值存储在本地变量中。

1.  将新值存储在不同的变量中。

1.  使用`compareAndSet()`方法尝试用新值替换旧值。如果此方法返回`true`值，则您发送的旧值作为参数是变量的值，因此它会更改值。该操作以原子方式执行，因为`carIn()`方法返回`true`值。如果`compareAndSet()`方法返回`false`值，则您发送的旧值不是变量的值（其他线程对其进行了修改），因此该操作无法以原子方式执行。操作将重新开始，直到可以以原子方式执行为止。

`carOut()`方法类似于`carIn()`方法。您还实现了两个使用`carIn()`和`carOut()`方法来模拟停车活动的`Runnable`对象。当您执行程序时，您会发现停车场从未超过停车场内汽车的最大值。

## 另请参阅

+   *在第六章中使用原子变量*中的配方，*并发集合*
