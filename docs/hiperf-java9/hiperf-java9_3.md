# 第三章：多线程和响应式编程

在本课中，我们将探讨一种通过在多个工作线程之间编程地分割任务来支持应用程序高性能的方法。这就是 4,500 年前建造金字塔的方法，自那时以来，这种方法从未失败。但是，可以参与同一项目的劳动者数量是有限制的。共享资源为工作人员的增加提供了上限，无论资源是以平方英尺和加仑（如金字塔时代的居住区和水）计算，还是以千兆字节和千兆赫（如计算机的内存和处理能力）计算。

生活空间和计算机内存的分配、使用和限制非常相似。但是，我们对人力和 CPU 的处理能力的感知却大不相同。历史学家告诉我们，数千年前的古埃及人同时工作于切割和移动大型石块。即使我们知道这些工人一直在轮换，有些人暂时休息或处理其他事务，然后回来取代完成年度任务的人，其他人死亡或受伤并被新兵取代，我们也不会有任何问题理解他们的意思。

但是，在计算机数据处理的情况下，当我们听到工作线程同时执行时，我们自动假设它们确实并行执行其编程的任务。只有在我们深入了解这样的系统之后，我们才意识到，只有在每个线程由不同的 CPU 执行时，才可能进行这样的并行处理。否则，它们共享相同的处理能力，并且我们认为它们同时工作，只是因为它们使用的时间段非常短——是我们在日常生活中使用的时间单位的一小部分。当线程共享相同的资源时，在计算机科学中，我们说它们是并发执行的。

在本课中，我们将讨论通过使用同时处理数据的工作线程（线程）来增加 Java 应用程序性能的方法。我们将展示如何通过对线程进行池化来有效地使用线程，如何同步同时访问的数据，如何在运行时监视和调整工作线程，以及如何利用响应式编程概念。

但在这之前，让我们重新学习在同一个 Java 进程中创建和运行多个线程的基础知识。

# 先决条件

主要有两种方法可以创建工作线程——通过扩展`java.lang.Thread`类和通过实现`java.lang.Runnable`接口。在扩展`java.lang.Thread`类时，我们不需要实现任何内容：

```java
class MyThread extends Thread {
}
```

我们的`MyThread`类继承了自动生成值的`name`属性和`start（）`方法。我们可以运行此方法并检查`name`：

```java
System.out.print("demo_thread_01(): ");
MyThread t1 = new MyThread();
t1.start();
System.out.println("Thread name=" + t1.getName());
```

如果我们运行此代码，结果将如下所示：

![先决条件](img/03_01.jpg)

如您所见，生成的`name`是`Thread-0`。如果我们在同一个 Java 进程中创建另一个线程，`name`将是`Thread-1`等等。`start（）`方法什么也不做。源代码显示，如果实现了`run（）`方法，它会调用`run（）`方法。

我们可以将任何其他方法添加到`MyThread`类中，如下所示：

```java
class MyThread extends Thread {
    private double result;
    public MyThread(String name){ super(name); }
    public void calculateAverageSqrt(){
        result =  IntStream.rangeClosed(1, 99999)
                           .asDoubleStream()
                           .map(Math::sqrt)
                           .average()
                           .getAsDouble();
    }
    public double getResult(){ return this.result; }
}
```

`calculateAverageSqrt（）`方法计算前 99,999 个整数的平均平方根，并将结果分配给可以随时访问的属性。以下代码演示了我们如何使用它：

```java
System.out.print("demo_thread_02(): ");
MyThread t1 = new MyThread("Thread01");
t1.calculateAverageSqrt();
System.out.println(t1.getName() + ": result=" + t1.getResult());
```

运行此方法将产生以下结果：

![先决条件](img/03_02.jpg)

正如您所期望的，`calculateAverageSqrt（）`方法会阻塞，直到计算完成。它是在主线程中执行的，没有利用多线程。为了做到这一点，我们将功能移动到`run（）`方法中：

```java
class MyThread01 extends Thread {
    private double result;
    public MyThread01(String name){ super(name); }
    public void run(){
        result =  IntStream.rangeClosed(1, 99999)
                           .asDoubleStream()
                           .map(Math::sqrt)
                           .average()
                           .getAsDouble();
    }
    public double getResult(){ return this.result; }
}
```

现在我们再次调用`start（）`方法，就像第一个示例中一样，并期望计算结果：

```java
System.out.print("demo_thread_03(): ");
MyThread01 t1 = new MyThread01("Thread01");
t1.start();
System.out.println(t1.getName() + ": result=" + t1.getResult());
```

然而，这段代码的输出可能会让您感到惊讶：

![先决条件](img/03_03.jpg)

这意味着主线程在新的`t1`线程完成计算之前访问（并打印）了`t1.getResult()`函数。我们可以尝试改变`run()`方法的实现，看看`t1.getResult()`函数是否可以获得部分结果：

```java
public void run() {
    for (int i = 1; i < 100000; i++) {
        double s = Math.sqrt(1\. * i);
        result = result + s;
    }
    result = result / 99999;
}
```

但是，如果我们再次运行`demo_thread_03()`方法，结果仍然是相同的：

![先决条件](img/03_04.jpg)

创建新线程并使其运行需要时间。与此同时，`main`线程立即调用`t1.getResult()`函数，因此还没有得到结果。

为了给新的（子）线程完成计算的时间，我们添加了以下代码：

```java
try {
     t1.join();
 } catch (InterruptedException e) { 
     e.printStackTrace();
 }
```

您已经注意到我们通过 100 毫秒暂停了主线程，并添加了打印当前线程名称，以说明我们所说的`main`线程，这个名称是自动分配给执行`main()`方法的线程。前面代码的输出如下：

![先决条件](img/03_05.jpg)

100 毫秒的延迟足以让`t1`线程完成计算。这是创建多线程计算的两种方式中的第一种。第二种方式是实现`Runnable`接口。如果进行计算的类已经扩展了其他类，并且由于某些原因您不能或不想使用组合，那么可能是唯一的可能方式。`Runnable`接口是一个函数接口（只有一个抽象方法），必须实现`run()`方法：

```java
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     */
    public abstract void run();
```

我们在`MyRunnable`类中实现了这个接口：

```java
class MyRunnable01 implements Runnable {
    private String id;
    private double result;
    public MyRunnable01(int id) {
        this.id = String.valueOf(id);
    }
    public String getId() { return this.id; }
    public double getResult() { return this.result; }
    public void run() {
        result = IntStream.rangeClosed(1, 99999)
                          .asDoubleStream()
                          .map(Math::sqrt)
                          .average()
                          .getAsDouble();
    }
}
```

它具有与之前的`Thread01`类相同的功能，另外我们添加了 id，以便在必要时识别线程，因为`Runnable`接口没有像`Thread`类那样内置的`getName()`方法。

同样，如果我们执行这个类而不暂停`main`线程，就像这样：

```java
System.out.print("demo_runnable_01(): ");
MyRunnable01 myRunnable = new MyRunnable01(1);
Thread t1 = new Thread(myRunnable);
t1.start();
System.out.println("Worker " + myRunnable.getId() 
           + ": result=" + myRunnable.getResult());
```

输出将如下所示：

![先决条件](img/03_06.jpg)

现在我们将添加暂停如下：

```java
System.out.print("demo_runnable_02(): ");
MyRunnable01 myRunnable = new MyRunnable01(1);
Thread t1 = new Thread(myRunnable);
t1.start();
try {
    t1.join();
} catch (InterruptedException e) { 
    e.printStackTrace();
}
System.out.println("Worker " + myRunnable.getId() 
           + ": result=" + myRunnable.getResult());
```

结果与`Thread01`类产生的结果完全相同：

![先决条件](img/3_07.jpg)

所有先前的示例都将生成的结果存储在类属性中。但情况并非总是如此。通常，工作线程要么将其值传递给另一个线程，要么将其存储在数据库或其他外部位置。在这种情况下，可以利用`Runnable`接口作为函数接口，并将必要的处理函数作为 lambda 表达式传递到新线程中：

```java
System.out.print("demo_lambda_01(): ");
String id = "1";
Thread t1 = 
    new Thread(() -> IntStream.rangeClosed(1, 99999)
         .asDoubleStream().map(Math::sqrt).average()
         .ifPresent(d -> System.out.println("Worker " 
                            + id + ": result=" + d)));
t1.start();
try {
    t1.join();
} catch (InterruptedException e) { 
    e.printStackTrace();
}
```

结果将会完全相同，如下所示：

![先决条件](img/03_08.jpg)

根据首选的样式，您可以重新排列代码，并将 lambda 表达式隔离在一个变量中，如下所示：

```java
Runnable r = () -> IntStream.rangeClosed(1, 99999)
       .asDoubleStream().map(Math::sqrt).average()
      .ifPresent(d -> System.out.println("Worker " 
                           + id + ": result=" + d));
Thread t1 = new Thread(r);
```

或者，您可以将 lambda 表达式放在一个单独的方法中：

```java
void calculateAverage(String id) {
    IntStream.rangeClosed(1, 99999)
        .asDoubleStream().map(Math::sqrt).average()
        .ifPresent(d -> System.out.println("Worker " 
                            + id + ": result=" + d));
}
void demo_lambda_03() {
    System.out.print("demo_lambda_03(): ");
    Thread t1 = new Thread(() -> calculateAverage("1"));
    ...
}
```

结果将是相同的，如下所示：

![先决条件](img/03_09.jpg)

有了对线程创建的基本理解，我们现在可以回到讨论如何使用多线程来构建高性能应用程序。换句话说，在我们了解了每个工作线程所需的能力和资源之后，我们现在可以讨论如何为像吉萨金字塔这样的大型项目引入许多工作线程的后勤问题。

编写管理工作线程的生命周期和它们对共享资源的访问的代码是可能的，但在一个应用程序到另一个应用程序中几乎是相同的。这就是为什么在 Java 的几个版本发布之后，线程管理的管道成为标准 JDK 库的一部分，作为`java.util.concurrent`包。这个包有丰富的接口和类，支持多线程和并发。我们将在后续章节中讨论如何使用大部分这些功能，同时讨论线程池、线程监视、线程同步和相关主题。

# 线程池

在本节中，我们将研究`java.util.concurrent`包中提供的`Executor`接口及其实现。它们封装了线程管理，并最大程度地减少了应用程序开发人员在编写与线程生命周期相关的代码上所花费的时间。

`Executor`接口在`java.util.concurrent`包中定义了三个。第一个是基本的`Executor`接口，其中只有一个`void execute(Runnable r)`方法。它基本上替代了以下内容：

```java
Runnable r = ...;
(new Thread(r)).start()
```

但是，我们也可以通过从池中获取线程来避免创建新线程。

第二个是`ExecutorService`接口，它扩展了`Executor`并添加了以下管理工作线程和执行器本身生命周期的方法组：

+   `submit()`: 将对象的执行放入接口`Runnable`或接口`Callable`的队列中（允许工作线程返回值）；返回`Future`接口的对象，可用于访问`Callable`返回的值并管理工作线程的状态

+   `invokeAll()`: 将一组接口`Callable`对象的执行放入队列中，当所有工作线程都完成时返回`Future`对象的列表（还有一个带有超时的重载`invokeAll()`方法）

+   `invokeAny()`: 将一组接口`Callable`对象的执行放入队列中；返回任何已完成的工作线程的一个`Future`对象（还有一个带有超时的重载`invokeAny()`方法）

管理工作线程状态和服务本身的方法：

+   `shutdown()`: 防止新的工作线程被提交到服务

+   `isShutdown()`: 检查执行器是否已启动关闭

+   `awaitTermination(long timeout, TimeUnit timeUnit)`: 在关闭请求后等待，直到所有工作线程完成执行，或超时发生，或当前线程被中断，以先发生的为准

+   `isTerminated()`: 在关闭被启动后检查所有工作线程是否已完成；除非首先调用了`shutdown()`或`shutdownNow()`，否则它永远不会返回`true`

+   `shutdownNow()`: 中断每个未完成的工作线程；工作线程应该定期检查自己的状态（例如使用`Thread.currentThread().isInterrupted()`），并在自己上优雅地关闭；否则，即使调用了`shutdownNow()`，它也会继续运行

第三个接口是`ScheduledExecutorService`，它扩展了`ExecutorService`并添加了允许调度工作线程执行（一次性和周期性）的方法。

可以使用`java.util.concurrent.ThreadPoolExecutor`或`java.util.concurrent.ScheduledThreadPoolExecutor`类创建基于池的`ExecutorService`实现。还有一个`java.util.concurrent.Executors`工厂类，涵盖了大部分实际情况。因此，在编写自定义代码创建工作线程池之前，我们强烈建议查看`java.util.concurrent.Executors`类的以下工厂方法：

+   `newSingleThreadExecutor()`: 创建一个按顺序执行工作线程的`ExecutorService`（池）实例

+   `newFixedThreadPool()`: 创建一个重用固定数量的工作线程的线程池；如果在所有工作线程仍在执行时提交了新任务，它将被放入队列，直到有工作线程可用

+   `newCachedThreadPool()`: 创建一个线程池，根据需要添加新线程，除非之前已创建了空闲线程；空闲了六十秒的线程将从缓存中移除

+   `newScheduledThreadPool()`: 创建一个固定大小的线程池，可以安排命令在给定延迟后运行，或定期执行

+   `newSingleThreadScheduledExecutor()`: 这将创建一个可以在给定延迟后调度命令运行或定期执行的单线程执行程序。

+   `newWorkStealingThreadPool()`: 这将创建一个使用与`ForkJoinPool`相同的工作窃取机制的线程池，对于工作线程生成其他线程的情况特别有用，比如递归算法。

每个方法都有一个重载版本，允许传入一个`ThreadFactory`，在需要时用于创建新线程。让我们看看在代码示例中如何运行。

首先，我们创建一个实现`Runnable`接口的`MyRunnable02`类——我们未来的工作线程：

```java
class MyRunnable02 implements Runnable {
    private String id;
    public MyRunnable02(int id) {
        this.id = String.valueOf(id);
    }
    public String getId(){ return this.id; }
    public void run() {
        double result = IntStream.rangeClosed(1, 100)
           .flatMap(i -> IntStream.rangeClosed(1, 99999))
           .takeWhile(i -> 
                 !Thread.currentThread().isInterrupted())
           .asDoubleStream()
           .map(Math::sqrt)
           .average()
           .getAsDouble();
        if(Thread.currentThread().isInterrupted()){
            System.out.println(" Worker " + getId() 
                       + ": result=ignored: " + result);
        } else {
            System.out.println(" Worker " + getId() 
                                + ": result=" + result);
        }
}
```

请注意，这种实现与之前的示例有一个重要的区别——`takeWhile(i -> !Thread.currentThread().isInterrupted())`操作允许流继续流动，只要线程工作状态未被设置为中断，这在调用`shutdownNow()`方法时会发生。一旦`takeWhile()`的谓词返回`false`（工作线程被中断），线程就会停止产生结果（只是忽略当前的`result`值）。在实际系统中，这相当于跳过将`result`值存储在数据库中，例如。

值得注意的是，在前面的代码中使用`interrupted()`状态方法来检查线程状态可能会导致不一致的结果。由于`interrupted()`方法返回正确的状态值，然后清除线程状态，因此对该方法的第二次调用（或在调用`interrupted()`方法后调用`isInterrupted()`方法）总是返回`false`。

尽管在这段代码中不是这种情况，但我们想在这里提到一些开发人员在实现工作线程的`try/catch`块时常犯的错误。例如，如果工作线程需要暂停并等待中断信号，代码通常如下所示：

```java
try {
    Thread.currentThread().wait();
} catch (InterruptedException e) {}
// Do what has to be done
```

```java
The better implementation is as follows:
```

```java
try {
    Thread.currentThread().wait();
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
}
// Do what has to be done
```

```java
join() method, we did not need to do that because that was the main code (the highest level code) that had to be paused.
```

现在我们可以展示如何使用`ExecutiveService`池的缓存池实现来执行之前的`MyRunnable02`类（其他类型的线程池使用方式类似）。首先，我们创建池，提交三个`MyRunnable02`类的实例进行执行，然后关闭池：

```java
ExecutorService pool = Executors.newCachedThreadPool();
IntStream.rangeClosed(1, 3).
       forEach(i -> pool.execute(new MyRunnable02(i)));
System.out.println("Before shutdown: isShutdown()=" 
          + pool.isShutdown() + ", isTerminated()=" 
                                + pool.isTerminated());
pool.shutdown(); // New threads cannot be submitted
System.out.println("After  shutdown: isShutdown()=" 
          + pool.isShutdown() + ", isTerminated()=" 
                                + pool.isTerminated());
```

如果运行这些代码，将会看到以下输出：

![线程池](img/03_10.jpg)

这里没有什么意外！在调用`shutdown()`方法之前，`isShutdown()`方法返回`false`值，之后返回`true`值。`isTerminated()`方法返回`false`值，因为没有任何工作线程已经完成。

通过在`shutdown()`方法后添加以下代码来测试`shutdown()`方法：

```java
try {
    pool.execute(new MyRunnable02(100));
} catch(RejectedExecutionException ex){
    System.err.println("Cannot add another worker-thread to the service queue:\n" + ex.getMessage());
}
```

现在输出将包含以下消息（如果截图对于此页面来说太大或者在适应时不可读）。

```java
Cannot add another worker-thread to the service queue:
Task com.packt.java9hp.ch09_threads.MyRunnable02@6f7fd0e6 
    rejected from java.util.concurrent.ThreadPoolExecutor
    [Shutting down, pool size = 3, active threads = 3, 
    queued tasks = 0, completed tasks = 0]
```

预期的是，在调用`shutdown()`方法后，将无法再向线程池中添加更多的工作线程。

现在让我们看看在启动关闭之后我们能做些什么：

```java
long timeout = 100;
TimeUnit timeUnit = TimeUnit.MILLISECONDS;
System.out.println("Waiting for all threads completion " 
                     + timeout + " " + timeUnit + "...");
// Blocks until timeout or all threads complete execution
boolean isTerminated = 
                pool.awaitTermination(timeout, timeUnit);
System.out.println("isTerminated()=" + isTerminated);
if (!isTerminated) {
    System.out.println("Calling shutdownNow()...");
    List<Runnable> list = pool.shutdownNow(); 
    printRunningThreadIds(list);
    System.out.println("Waiting for threads completion " 
                     + timeout + " " + timeUnit + "...");
    isTerminated = 
                pool.awaitTermination(timeout, timeUnit);
    if (!isTerminated){
        System.out.println("Some threads are running...");
    }
    System.out.println("Exiting.");
}
```

`printRunningThreadIds()`方法如下所示：

```java
void printRunningThreadIds(List<Runnable> l){
    String list = l.stream()
            .map(r -> (MyRunnable02)r)
            .map(mr -> mr.getId())
            .collect(Collectors.joining(","));
    System.out.println(l.size() + " thread"
       + (l.size() == 1 ? " is" : "s are") + " running"
            + (l.size() > 0 ? ": " + list : "") + ".");
}
```

前面代码的输出将如下所示：

![线程池](img/03_11.jpg)

这意味着每个工作线程完成计算所需的时间为 100 毫秒。（请注意，如果您尝试在您的计算机上重现这些数据，由于性能差异，结果可能会略有不同，因此您需要调整超时时间。）

当我们将等待时间减少到 75 毫秒时，输出如下：

![线程池](img/03_12.jpg)

在我们的计算机上，75 毫秒不足以让所有线程完成，因此它们被`shutdownNow()`中断，并且它们的部分结果被忽略。

现在让我们移除`MyRunnable01`类中对中断状态的检查：

```java
class MyRunnable02 implements Runnable {
    private String id;
    public MyRunnable02(int id) {
        this.id = String.valueOf(id);
    }
    public String getId(){ return this.id; }
    public void run() {
        double result = IntStream.rangeClosed(1, 100)
           .flatMap(i -> IntStream.rangeClosed(1, 99999))
           .asDoubleStream()
           .map(Math::sqrt)
           .average()
           .getAsDouble();
        System.out.println(" Worker " + getId() 
                                + ": result=" + result);
}
```

没有这个检查，即使我们将超时时间减少到 1 毫秒，结果也将如下所示：

![线程池](img/03_13.jpg)

这是因为工作线程从未注意到有人试图中断它们并完成了它们分配的计算。这最后的测试演示了在工作线程中观察中断状态的重要性，以避免许多可能的问题，即数据损坏和内存泄漏。

演示的缓存池在工作线程执行短任务且其数量不会过多增长时运行良好，不会出现问题。如果您需要更多地控制任何时候运行的工作线程的最大数量，请使用固定大小的线程池。我们将在本课程的以下部分讨论如何选择池的大小。

单线程池非常适合按特定顺序执行任务的情况，或者当每个任务需要的资源太多，无法与其他任务并行执行时。使用单线程执行的另一个情况是对修改相同数据的工作线程，但数据无法以其他方式受到并行访问的保护。线程同步也将在本课程的以下部分中更详细地讨论。

在我们的示例代码中，到目前为止，我们只包括了`Executor`接口的`execute()`方法。在接下来的部分中，我们将演示`ExecutorService`池的其他方法，同时讨论线程监控。

在本节的最后一条备注。工作线程不需要是同一个类的对象。它们可以代表完全不同的功能，仍然可以由一个池管理。

# 监控线程

有两种监控线程的方法，即通过编程和使用外部工具。我们已经看到了如何检查工作计算的结果。让我们重新访问一下那段代码。我们还将稍微修改我们的工作实现：

```java
class MyRunnable03 implements Runnable {
  private String name;
  private double result;
  public String getName(){ return this.name; }
  public double getResult() { return this.result; }
  public void run() {
    this.name = Thread.currentThread().getName();
    double result = IntStream.rangeClosed(1, 100)
      .flatMap(i -> IntStream.rangeClosed(1, 99999))
      .takeWhile(i -> !Thread.currentThread().isInterrupted())
      .asDoubleStream().map(Math::sqrt).average().getAsDouble();
    if(!Thread.currentThread().isInterrupted()){
      this.result = result;
    }
  }
}
```

对于工作线程的标识，我们现在使用在执行时自动分配的线程名称，而不是自定义 ID（这就是为什么我们在`run()`方法中分配`name`属性，在线程获取其名称时调用该方法）。新的`MyRunnable03`类可以像这样使用：

```java
void demo_CheckResults() {
    ExecutorService pool = Executors.newCachedThreadPool();
    MyRunnable03 r1 = new MyRunnable03();
    MyRunnable03 r2 = new MyRunnable03();
    pool.execute(r1);
    pool.execute(r2);
    try {
        t1.join();
    } catch (InterruptedException e) { 
        e.printStackTrace();
    }
    System.out.println("Worker " + r1.getName() + ": result=" + r1.getResult());
    System.out.println("Worker " + r2.getName() + ": result=" + r2.getResult());
    shutdown(pool);
}
```

`shutdown()`方法包含以下代码：

```java
void shutdown(ExecutorService pool) {
    pool.shutdown();
    try {
        if(!pool.awaitTermination(1, TimeUnit.SECONDS)){
            pool.shutdownNow();
        }
    } catch (InterruptedException ie) {}
}
```

如果我们运行上述代码，输出将如下所示：

![监控线程](img/03_14.jpg)

如果您的计算机上的结果不同，请尝试增加`sleepMs()`方法的输入值。

获取有关应用程序工作线程的信息的另一种方法是使用`Future`接口。我们可以使用`ExecutorService`池的`submit()`方法访问此接口，而不是`execute()`、`invokeAll()`或`invokeAny()`方法。以下代码显示了如何使用`submit()`方法：

```java
ExecutorService pool = Executors.newCachedThreadPool();
Future f1 = pool.submit(new MyRunnable03());
Future f2 = pool.submit(new MyRunnable03());
printFuture(f1, 1);
printFuture(f2, 2);
shutdown(pool);
```

`printFuture()`方法的实现如下：

```java
void printFuture(Future future, int id) {
    System.out.println("printFuture():");
    while (!future.isCancelled() && !future.isDone()){
        System.out.println("    Waiting for worker " 
                                + id + " to complete...");
        sleepMs(10);
    }
    System.out.println("    Done...");
}
```

`sleepMs()`方法包含以下代码：

```java
void sleepMs(int sleepMs) {
    try {
        TimeUnit.MILLISECONDS.sleep(sleepMs);
    } catch (InterruptedException e) {}
}
```

我们更喜欢这种实现而不是传统的`Thread.sleep()`，因为它明确指定了使用的时间单位。

如果我们执行前面的代码，结果将类似于以下内容：

![监控线程](img/03_15.jpg)

`printFuture()`方法已经阻塞了主线程的执行，直到第一个线程完成。与此同时，第二个线程也已经完成。如果我们在`shutdown()`方法之后调用`printFuture()`方法，那么两个线程在那时已经完成了，因为我们设置了 1 秒的等待时间（参见`pool.awaitTermination()`方法），这足够让它们完成工作。

![监控线程](img/03_16.jpg)

如果您认为这不是从线程监视的角度来看的太多信息，`java.util.concurrent`包通过`Callable`接口提供了更多功能。这是一个允许通过`Future`对象返回任何对象（包含工作线程计算结果的结果）的功能接口，使用`ExecutiveService`方法--`submit()`、`invokeAll()`和`invokeAny()`。例如，我们可以创建一个包含工作线程结果的类：

```java
class Result {
    private double result;
    private String workerName;
    public Result(String workerName, double result) {
        this.result = result;
        this.workerName = workerName;
    }
    public String getWorkerName() { return workerName; }
    public double getResult() { return result;}
}
```

我们还包括了工作线程的名称，以便监视生成的结果。实现`Callable`接口的类可能如下所示：

```java
class MyCallable01<T> implements Callable {
  public Result call() {
    double result = IntStream.rangeClosed(1, 100)
       .flatMap(i -> IntStream.rangeClosed(1, 99999))
       .takeWhile(i -> !Thread.currentThread().isInterrupted())
       .asDoubleStream().map(Math::sqrt).average().getAsDouble();

    String workerName = Thread.currentThread().getName();
    if(Thread.currentThread().isInterrupted()){
        return new Result(workerName, 0);
    } else {
        return new Result(workerName, result);
    }
  }
}
```

以下是使用`MyCallable01`类的代码：

```java
ExecutorService pool = Executors.newCachedThreadPool();
Future f1 = pool.submit(new MyCallable01<Result>());
Future f2 = pool.submit(new MyCallable01<Result>());
printResult(f1, 1);
printResult(f2, 2);
shutdown(pool);
```

`printResult()` 方法包含以下代码：

```java
void printResult(Future<Result> future, int id) {
    System.out.println("printResult():");
    while (!future.isCancelled() && !future.isDone()){
        System.out.println("    Waiting for worker " 
                              + id + " to complete...");
        sleepMs(10);
    }
    try {
        Result result = future.get(1, TimeUnit.SECONDS);
        System.out.println("    Worker " 
                + result.getWorkerName() + ": result = " 
                                   + result.getResult());
    } catch (Exception ex) {
        ex.printStackTrace();
    }
}
```

此代码的输出可能如下所示：

![监视线程](img/03_17.jpg)

先前的输出显示，与之前的示例一样，`printResult()`方法会等待工作线程中的第一个完成，因此第二个线程成功在同一时间完成其工作。正如您所看到的，使用`Callable`的优点是，我们可以从`Future`对象中检索实际结果，如果需要的话。

`invokeAll()`和`invokeAny()`方法的使用看起来很相似：

```java
ExecutorService pool = Executors.newCachedThreadPool();
try {
    List<Callable<Result>> callables = 
              List.of(new MyCallable01<Result>(), 
                           new MyCallable01<Result>());
    List<Future<Result>> futures = 
                             pool.invokeAll(callables);
    printResults(futures);
} catch (InterruptedException e) {
    e.printStackTrace();
}
shutdown(pool);
```

`printResults()`方法使用了您已经了解的`printResult()`方法：

```java
void printResults(List<Future<Result>> futures) {
    System.out.println("printResults():");
    int i = 1;
    for (Future<Result> future : futures) {
        printResult(future, i++);
    }
}
```

如果我们运行上述代码，输出将如下所示：

![监视线程](img/03_18.jpg)

如您所见，不再等待工作线程完成工作。这是因为`invokeAll()`方法在所有作业完成后返回`Future`对象的集合。

`invokeAny()`方法的行为类似。如果我们运行以下代码：

```java
System.out.println("demo_InvokeAny():");
ExecutorService pool = Executors.newCachedThreadPool();
try {
    List<Callable<Result>> callables = 
                   List.of(new MyCallable01<Result>(), 
                            new MyCallable01<Result>());
    Result result = pool.invokeAny(callables);
    System.out.println("    Worker " 
                        + result.getWorkerName()
                  + ": result = " + result.getResult());
} catch (InterruptedException | ExecutionException e) {
    e.printStackTrace();
}
shutdown(pool);
```

以下将是输出：

![监视线程](img/03_19.jpg)

这些是以编程方式监视线程的基本技术，但可以轻松地扩展我们的示例，以涵盖更复杂的情况，以满足特定应用程序的需求。在第 5 课中，*利用新的 API 改进您的代码*，我们还将讨论另一种以编程方式监视工作线程的方法，即 JDK 8 中引入并在 JDK 9 中扩展的`java.util.concurrent.CompletableFuture`类。

如果需要，可以使用`java.lang.Thread`类获取有关 JVM 进程中的应用程序工作线程以及所有其他线程的信息：

```java
void printAllThreads() {
    System.out.println("printAllThreads():");
    Map<Thread, StackTraceElement[]> map = Thread.getAllStackTraces();
    for(Thread t: map.keySet()){
        System.out.println("    " + t);
    }
```

现在，让我们按如下方式调用此方法：

```java
void demo_CheckResults() {
    ExecutorService pool = Executors.newCachedThreadPool();
    MyRunnable03 r1 = new MyRunnable03();
    MyRunnable03 r2 = new MyRunnable03();
    pool.execute(r1);
    pool.execute(r2);
    sleepMs(1000);
    printAllThreads();
    shutdown(pool);
}
```

结果如下所示：

![监视线程](img/03_20.jpg)

我们利用了`Thread`类的`toString()`方法，该方法仅打印线程名称、优先级和线程所属的线程组。我们可以在名称为`pool-1-thread-1`和`pool-1-thread-2`的列表中明确看到我们创建的两个应用程序线程（除了`main`线程）。但是，如果我们在调用`shutdown()`方法后调用`printAllThreads()`方法，输出将如下所示：

![监视线程](img/03_21.jpg)

我们不再在列表中看到`pool-1-thread-1`和`pool-1-thread-2`线程，因为`ExecutorService`池已关闭。

我们可以轻松地添加从相同映射中提取的堆栈跟踪信息：

```java
void printAllThreads() {
    System.out.println("printAllThreads():");
    Map<Thread, StackTraceElement[]> map 
                               = Thread.getAllStackTraces();
    for(Thread t: map.keySet()){
        System.out.println("   " + t);
        for(StackTraceElement ste: map.get(t)){
            System.out.println("        " + ste);
        }
    }
}
```

然而，这将占用书页太多空间。在第 5 课中，*利用新的 API 改进您的代码*，我们将介绍随 JDK 9 一起提供的新的 Java 功能，并讨论通过`java.lang.StackWalker`类更好地访问堆栈跟踪的方法。

`Thread`类对象还有其他几个方法，提供有关线程的信息，如下所示：

+   `dumpStack()`: 这将堆栈跟踪打印到标准错误流

+   `enumerate(Thread[] arr)`: 这将当前线程的线程组中的活动线程及其子组复制到指定的数组`arr`中

+   `getId()`: 这提供了线程的 ID

+   `getState()`：这读取线程的状态；`enum Thread.State`的可能值可以是以下之一：

+   `NEW`：这是尚未启动的线程

+   `RUNNABLE`：这是当前正在执行的线程

+   `BLOCKED`：这是正在等待监视器锁释放的线程

+   `WAITING`：这是正在等待中断信号的线程

+   `TIMED_WAITING`：这是等待中断信号直到指定等待时间的线程

+   `TERMINATED`：这是已退出的线程

+   `holdsLock(Object obj)`：这表示线程是否持有指定对象的监视器锁

+   `interrupted()`或`isInterrupted()`：这表示线程是否已被中断（收到中断信号，意味着中断标志被设置为`true`）

+   `isAlive()`：这表示线程是否存活

+   `isDaemon()`：这表示线程是否为守护线程。

`java.lang.management`包为监视线程提供了类似的功能。例如，让我们运行这段代码片段：

```java
void printThreadsInfo() {
    System.out.println("printThreadsInfo():");
    ThreadMXBean threadBean = 
                      ManagementFactory.getThreadMXBean();
    long ids[] = threadBean.getAllThreadIds();
    Arrays.sort(ids);
    ThreadInfo[] tis = threadBean.getThreadInfo(ids, 0);
    for (ThreadInfo ti : tis) {
        if (ti == null) continue;
        System.out.println("    Id=" + ti.getThreadId() 
                       + ", state=" + ti.getThreadState() 
                          + ", name=" + ti.getThreadName());
    }
}
```

为了更好地呈现，我们利用了列出的线程 ID，并且如您之前所见，已按 ID 对输出进行了排序。如果我们在`shutdown()`方法之前调用`printThreadsInfo()`方法，输出将如下所示：

![监控线程](img/03_22.jpg)

但是，如果我们在`shutdown()`方法之后调用`printThreadsInfo()`方法，输出将不再包括我们的工作线程，就像使用`Thread`类 API 的情况一样：

![监控线程](img/03_23.jpg)

`java.lang.management.ThreadMXBean`接口提供了关于线程的许多其他有用数据。您可以参考 Oracle 网站上关于此接口的官方 API，了解更多信息，请查看此链接：[`docs.oracle.com/javase/8/docs/api/index.html?java/lang/management/ThreadMXBean.html`](https://docs.oracle.com/javase/8/docs/api/index.html?java/lang/management/ThreadMXBean.html)。

在前面提到的线程列表中，您可能已经注意到`Monitor Ctrl-Break`线程。此线程提供了另一种监视 JVM 进程中线程的方法。在 Windows 上按下*Ctrl*和*Break*键会导致 JVM 将线程转储打印到应用程序的标准输出。在 Oracle Solaris 或 Linux 操作系统上，*Ctrl*键和反斜杠*\*的组合具有相同的效果。这将我们带到了用于线程监视的外部工具。

如果您无法访问源代码或更喜欢使用外部工具进行线程监视，则 JDK 安装中提供了几种诊断实用程序。在以下列表中，我们仅提到允许线程监视的工具，并仅描述所列工具的此功能（尽管它们还具有其他广泛的功能）：

+   `jcmd`实用程序使用 JVM 进程 ID 或主类的名称向同一台机器上的 JVM 发送诊断命令请求：`jcmd <process id/main class> <command> [options]`，其中`Thread.print`选项打印进程中所有线程的堆栈跟踪。

+   JConsole 监控工具使用 JVM 中的内置 JMX 工具来提供有关运行应用程序的性能和资源消耗的信息。它有一个线程选项卡窗格，显示随时间变化的线程使用情况，当前活动线程数，自 JVM 启动以来的最高活动线程数。可以选择线程及其名称、状态和堆栈跟踪，以及对于阻塞线程，线程正在等待获取的同步器以及拥有锁的线程。使用**死锁检测**按钮来识别死锁。运行该工具的命令是`jconsole <process id>`或（对于远程应用程序）`jconsole <hostname>:<port>`，其中`port`是使用启用 JMX 代理的 JVM 启动命令指定的端口号。

+   `jdb`实用程序是一个示例命令行调试器。它可以附加到 JVM 进程并允许您检查线程。

+   `jstack`命令行实用程序可以附加到 JVM 进程并打印所有线程的堆栈跟踪，包括 JVM 内部线程，还可以选择打印本地堆栈帧。它还允许您检测死锁。

+   **Java Flight Recorder**（**JFR**）提供有关 Java 进程的信息，包括等待锁的线程，垃圾收集等。它还允许获取线程转储，这类似于使用`Thread.print`诊断命令或使用 jstack 工具生成的线程转储。如果满足条件，可以设置**Java Mission Control**（**JMC**）来转储飞行记录。JMC UI 包含有关线程、锁争用和其他延迟的信息。尽管 JFR 是商业功能，但对于开发人员的台式机/笔记本电脑以及测试、开发和生产环境中的评估目的，它是免费的。

### 注意

您可以在官方 Oracle 文档[`docs.oracle.com/javase/9/troubleshoot/diagnostic-tools.htm`](https://docs.oracle.com/javase/9/troubleshoot/diagnostic-tools.htm)中找到有关这些和其他诊断工具的更多详细信息。

# 线程池执行器的大小

在我们的示例中，我们使用了一个缓存线程池，根据需要创建新线程，或者如果可用，重用已经使用过的线程，但是完成了工作并返回到池中以便新的分配。我们不担心创建太多线程，因为我们的演示应用程序最多只有两个工作线程，并且它们的生命周期非常短。

但是，在应用程序没有固定的工作线程限制或者没有好的方法来预测线程可能占用多少内存或执行多长时间的情况下，设置工作线程计数的上限可以防止应用程序性能意外下降，内存耗尽或工作线程使用的其他任何资源枯竭。如果线程行为非常不可预测，单个线程池可能是唯一的解决方案，并且可以选择使用自定义线程池执行器（稍后将对此最后一个选项进行解释）。但在大多数情况下，固定大小的线程池执行器是应用程序需求和代码复杂性之间的一个很好的实际折衷。根据具体要求，这样的执行器可能是以下三种类型之一：

+   一个直接的、固定大小的`ExecutorService.newFixedThreadPool(int nThreads)`池，不会超出指定的大小，也不会采用其他方式

+   `ExecutorService.newScheduledThreadPool(int nThreads)` 提供了几个允许调度不同线程组的线程池，具有不同的延迟或执行周期

+   `ExecutorService.newWorkStealingPool(int parallelism)`，它适应于指定数量的 CPU，您可以将其设置为高于或低于计算机上实际 CPU 数量

在任何上述池中设置固定大小过低可能会剥夺应用程序有效利用可用资源的机会。因此，在选择池大小之前，建议花一些时间对其进行监视和调整 JVM（请参阅本课程的某个部分中如何执行此操作），以便识别应用程序行为的特殊性。实际上，部署-监视-调整-调整的周期必须在整个应用程序生命周期中重复进行，以适应并利用代码或执行环境中发生的变化。

您考虑的第一个参数是系统中的 CPU 数量，因此线程池大小至少可以与 CPU 数量一样大。然后，您可以监视应用程序，查看每个线程使用 CPU 的时间以及使用其他资源（如 I/O 操作）的时间。如果未使用 CPU 的时间与线程的总执行时间相当，那么可以通过**未使用 CPU 的时间/总执行时间**来增加池大小。但前提是另一个资源（磁盘或数据库）不是线程之间争用的主题。如果是后者的情况，那么可以使用该资源而不是 CPU 作为界定因素。

假设您的应用程序的工作线程不是太大或执行时间不太长，并且属于典型工作线程的主流人口，可以通过增加期望响应时间和线程使用 CPU 或其他最具争议资源的时间的比率（四舍五入）来增加池大小。这意味着，对于相同的期望响应时间，线程使用 CPU 或其他同时访问的资源越少，池大小就应该越大。如果具有改善并发访问能力的争议资源（如数据库中的连接池），请首先考虑利用该功能。

如果在不同情况下运行的同时所需的线程数量在运行时发生变化，可以使池大小动态化，并创建一个新大小的池（在所有线程完成后关闭旧池）。在添加或删除可用资源后，可能还需要重新计算新池的大小。例如，您可以使用`Runtime.getRuntime().availableProcessors()`根据当前可用 CPU 的数量来以编程方式调整池大小。

如果 JDK 提供的现成线程池执行器实现都不适合特定应用程序的需求，在从头开始编写线程管理代码之前，可以尝试首先使用`java.util.concurrent.ThreadPoolExecutor`类。它有几个重载的构造函数。

为了让您了解其功能，这是具有最多选项的构造函数：

```java
ThreadPoolExecutor (int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, RejectedExecutionHandler handler)
```

前面提到的参数是（引用自 JavaDoc）：

+   `corePoolSize`: 这是保留在池中的线程数，即使它们处于空闲状态，除非设置了`allowCoreThreadTimeOut`

+   `maximumPoolSize`: 这是允许在池中的最大线程数

+   `keepAliveTime`: 当线程数大于核心数时，这是多余的空闲线程在终止之前等待新任务的最长时间

+   `unit`: 这是`keepAliveTime`参数的时间单位

+   `workQueue`: 这是在执行任务之前用于保存任务的队列，该队列将仅保存由`execute`方法提交的`Runnable`任务

+   `threadFactory`: 这是执行器创建新线程时要使用的工厂

+   `handler`: 这是在执行受阻时要使用的处理程序，因为已达到线程边界和队列容量

除了`workQueue`参数之外，以前的构造函数参数也可以在创建`ThreadPoolExecutor`对象后通过相应的 setter 进行设置，从而在动态调整现有池特性时提供更大的灵活性。

# 线程同步

我们已经收集了足够的人员和资源，如食物、水和工具，用于金字塔的建造。我们将人员分成团队，并为每个团队分配了一个任务。一个（池）人住在附近的村庄，处于待命状态，随时准备取代在任务中生病或受伤的人。我们调整了劳动力数量，以便只有少数人会在村庄中闲置。我们通过工作-休息周期轮换团队，以保持项目以最大速度进行。我们监控了整个过程，并调整了团队数量和他们所需的供应流量，以确保没有明显的延迟，并且整个项目中有稳定的可测量的进展。然而，整体上有许多活动部分和各种大小的意外事件和问题经常发生。

为了确保工人和团队不会互相干扰，并且有某种交通规则，以便下一个技术步骤在前一个完成之前不会开始，主建筑师派遣他的代表到建筑工地的所有关键点。这些代表确保任务以预期的质量和规定的顺序执行。他们有权力阻止下一个团队开始工作，直到前一个团队尚未完成。他们就像交通警察或可以关闭工作场所的锁，或者在必要时允许它。

这些代表正在做的工作可以用现代语言定义为执行单元的协调或同步。没有它，成千上万的工人的努力结果将是不可预测的。从一万英尺高的大局观看起来平稳和和谐，就像飞机窗户外的农田一样。但是，如果不仔细检查和关注关键细节，这个看起来完美的画面可能会带来一次糟糕的收成，甚至没有收成。

同样，在多线程执行环境的安静电子空间中，如果它们共享对同一工作场所的访问权，工作线程必须进行同步。例如，让我们为一个线程创建以下类-工作者：

```java
class MyRunnable04 implements Runnable {
  private int id;
  public MyRunnable04(int id) { this.id = id; }
  public void run() {
    IntStream.rangeClosed(1, 5)
      .peek(i -> System.out.println("Thread "+id+": "+ i))
      .forEach(i -> Demo04Synchronization.result += i);
    }
}
```

正如你所看到的，它依次将 1、2、3、4、5（因此，预期的总和应该是 15）添加到`Demo04Synchronization`类的静态属性中：

```java
public class Demo04Synchronization {
    public static int result;
    public static void main(String... args) {
        System.out.println();
        demo_ThreadInterference();
    }
    private static void demo_ThreadInterference(){
        System.out.println("demo_ThreadInterference: ");
        MyRunnable04 r1 = new MyRunnable04(1);
        Thread t1 = new Thread(r1);
        MyRunnable04 r2 = new MyRunnable04(2);
        Thread t2 = new Thread(r2);
        t1.start();
        sleepMs(100);
        t2.start();
        sleepMs(100);
        System.out.println("Result=" + result);
    }
    private static void sleepMs(int sleepMs) {
        try {
            TimeUnit.MILLISECONDS.sleep(sleepMs);
        } catch (InterruptedException e) {}
    }
}
```

在早期的代码中，当主线程第一次暂停 100 毫秒时，线程`t1`将变量 result 的值带到 15，然后线程`t2`再添加 15，得到总和 30。以下是输出：

![线程同步](img/03_24.jpg)

如果我们去掉 100 毫秒的第一个暂停，线程将同时工作：

![线程同步](img/03_25.jpg)

最终结果仍然是 30。我们对这段代码感到满意，并将其部署到生产环境中作为经过充分测试的代码。然而，如果我们将添加的次数从 5 增加到 250，例如，结果将变得不稳定，并且每次运行都会发生变化。以下是第一次运行（我们注释掉了每个线程的打印输出，以节省空间）：

![线程同步](img/03_26.jpg)

以下是另一次运行的输出：

![线程同步](img/03_27.jpg)

它证明了`Demo04Synchronization.result += i`操作不是原子的。这意味着它包括几个步骤，从`result`属性中读取值，向其添加值，将得到的总和分配回`result`属性。这允许以下情景，例如：

+   两个线程都读取了`result`的当前值（因此每个线程都有相同原始`result`值的副本）

+   每个线程都向相同的原始整数添加另一个整数

+   第一个线程将总和分配给`result`属性

+   第二个线程将其总和分配给`result`属性

正如你所看到的，第二个线程不知道第一个线程所做的加法，并且覆盖了第一个线程分配给`result`属性的值。但是这种线程交错并不是每次都会发生。这只是一个机会游戏。这就是为什么我们只看到五个数字时没有看到这样的效果。但是随着并发操作数量的增加，这种情况发生的概率会增加。

在建造金字塔的过程中也可能发生类似的情况。第二个团队可能在第一个团队完成任务之前开始做一些事情。我们绝对需要一个**同步器**，它使用`synchronized`关键字。通过使用它，我们可以在`Demo04Synchronization`类中创建一个方法（建筑师代表），控制对`result`属性的访问，并向其添加这个关键字。

```java
private static int result;
public static synchronized void incrementResult(int i){
    result += i;
}
```

现在我们也需要修改工作线程中的`run()`方法：

```java
public void run() {
    IntStream.rangeClosed(1, 250)
       .forEach(Demo04Synchronization::incrementResult);
}
```

现在输出显示每次运行都有相同的最终数字：

![线程同步](img/03_28.jpg)

`synchronized`关键字告诉 JVM 一次只允许一个线程进入这个方法。所有其他线程将等待，直到当前访问者退出该方法。

通过向代码块添加`synchronized`关键字也可以实现相同的效果：

```java
public static void incrementResult(int i){
    synchronized (Demo04Synchronization.class){
        result += i;
    }
}
```

不同之处在于，代码块同步需要一个对象--在静态属性同步的情况下是一个类对象（就像我们的情况一样），或者在实例属性同步的情况下是任何其他对象。每个对象都有一个固有锁或监视器锁，通常简称为监视器。一旦一个线程在对象上获取了锁，直到第一个线程从锁定的代码中正常退出或代码抛出异常后释放锁，其他线程就无法在同一个对象上获取锁。

事实上，在同步方法的情况下，一个对象（方法所属的对象）也被用于锁定。它只是在幕后自动发生，不需要程序员明确地使用对象的锁。

如果您没有访问`main`类代码（就像之前的例子一样），您可以将`result`属性保持为公共，并在工作线程中添加一个同步方法（而不是像我们所做的那样添加到类中）：

```java
class MyRunnable05 implements Runnable {
    public synchronized void incrementResult(int i){
        Demo04Synchronization.result += i;
    }
    public void run() {
        IntStream.rangeClosed(1, 250)
                .forEach(this::incrementResult);
    }
}
```

在这种情况下，`MyRunnable05`工作类的对象默认提供其固有锁。这意味着，您需要为所有线程使用`MyRunnable05`类的相同对象：

```java
void demo_Synchronized(){
    System.out.println("demo_Synchronized: ");
    MyRunnable05 r1 = new MyRunnable05();
    Thread t1 = new Thread(r1);
    Thread t2 = new Thread(r1);
    t1.start();
    t2.start();
    sleepMs(100);
    System.out.println("Result=" + result);
}
```

前面代码的输出与之前相同：

![线程同步](img/03_29.jpg)

有人可能会认为这种最后的实现更可取，因为它将同步的责任分配给了线程（以及其代码的作者），而不是共享资源。这样，随着线程实现的演变，同步的需求也会发生变化，只要客户端代码（使用相同或不同的对象进行线程）也可以根据需要进行更改。

还有另一个可能发生在某些操作系统中的并发问题。根据线程缓存的实现方式，一个线程可能会保留`result`属性的本地副本，并且在另一个线程更改其值后不会更新它。通过向共享（在线程之间）属性添加`volatile`关键字，可以保证其当前值始终从主内存中读取，因此每个线程都将看到其他线程所做的更新。在我们之前的例子中，我们只是将`Demo04Synchronization`类的属性设置为`private static volatile int result`，在同一类或线程中添加一个同步的`incrementResult()`方法，不再担心线程相互干扰。

所描述的线程同步通常对主流应用程序来说已经足够了。但是，更高性能和高并发处理通常需要更仔细地查看线程转储，这通常显示方法同步比块同步更有效。当然，这也取决于方法和块的大小。由于所有其他尝试访问同步方法或块的线程都将停止执行，直到当前访问者退出该方法或块，因此尽管有开销，但小的同步块可能比大的同步方法性能更好。

对于某些应用程序，默认的内部锁的行为可能不太合适，因为它只会在锁被释放之前阻塞。如果是这种情况，请考虑使用`java.util.concurrent.locks`包中的锁。与使用默认的内部锁相比，该包中的锁所基于的访问控制有几个不同之处。这些差异可能对您的应用程序有利，也可能提供不必要的复杂性，但重要的是要了解它们，以便您可以做出明智的决定：

+   代码的同步片段不需要属于一个方法；它可以跨越几个方法，由实现`Lock`接口的对象上调用`lock()`和`unlock()`方法来界定

+   在创建名为`ReentrantLock`的`Lock`接口对象时，可以在构造函数中传递一个`fair`标志，使锁能够首先授予等待时间最长的线程访问，这有助于避免饥饿（低优先级线程永远无法访问锁）

+   允许线程在承诺被阻塞之前测试锁是否可访问

+   允许中断等待锁的线程，以便它不会无限期地保持阻塞

+   您可以根据应用程序需要自己实现`Lock`接口

`Lock`接口的典型使用模式如下：

```java
Lock lock = ...;
...
    lock.lock();
    try {
        // the fragment that is synchronized
    } finally {
        lock.unlock();
    }
...
}
```

注意`finally`块。这是确保最终释放`lock`的方法。否则，在`try-catch`块内的代码可能会抛出异常，而锁却永远不会被释放。

除了`lock()`和`unlock()`方法之外，`Lock`接口还有以下方法：

+   `lockInterruptibly()`: 除非当前线程被中断，否则获取锁。与`lock()`方法类似，此方法在等待锁被获取时阻塞，与`lock()`方法不同的是，如果另一个线程中断等待线程，此方法会抛出`InterruptedException`异常

+   `tryLock()`: 如果在调用时空闲，立即获取锁

+   `tryLock(long time, TimeUnit unit)`: 如果在给定的等待时间内空闲，并且当前线程未被中断，则获取锁

+   `newCondition()`: 返回一个绑定到此`Lock`实例的新`Condition`实例，获取锁后，线程可以释放它（在`Condition`对象上调用`await()`方法），直到其他线程在相同的`Condition`对象上调用`signal()`或`signalAll()`，还可以指定超时期限（使用重载的`await()`方法），因此如果没有收到信号，线程将在超时后恢复，有关更多详细信息，请参阅`Condition` API

本书的范围不允许我们展示`java.util.concurrent.locks`包中提供的所有线程同步可能性。描述所有这些可能需要几节课。但即使从这个简短的描述中，您也可以看到很难找到一个不能使用`java.util.concurrent.locks`包解决的同步问题。

当需要隔离几行代码作为原子（全有或全无）操作时，方法或代码块的同步才有意义。但是在简单的变量赋值或数字的增加/减少的情况下（就像我们之前的例子中一样），有一种更好的方式可以通过使用`java.util.concurrent.atomic`包中支持无锁线程安全编程的类来同步这个操作。各种类涵盖了所有的数字，甚至是数组和引用类型，比如`AtomicBoolean`、`AtomicInteger`、`AtomicIntegerArray`、`AtomicReference`和`AtomicReferenceArray`。

总共有 16 个类。根据值类型的不同，每个类都允许进行全方位的操作，即`set()`、`get()`、`addAndGet()`、`compareAndSet()`、`incrementAndGet()`、`decrementAndGet()`等等。每个操作的实现要比使用`synchronized`关键字实现的同样操作要高效得多。而且不需要`volatile`关键字，因为它在底层使用了它。

如果同时访问的资源是一个集合，`java.util.concurrent`包提供了各种线程安全的实现，其性能优于同步的`HashMap`、`Hashtable`、`HashSet`、`Vector`和`ArrayList`（如果我们比较相应的`ConcurrentHashMap`、`CopyOnWriteArrayList`和`CopyOnWriteHashSet`）。传统的同步集合会锁定整个集合，而并发集合使用诸如锁分离之类的先进技术来实现线程安全。并发集合在更多读取和较少更新时特别出色，并且比同步集合更具可伸缩性。但是，如果您的共享集合的大小较小且写入占主导地位，那么并发集合的优势就不那么明显了。

# 调整 JVM

每座金字塔建筑，就像任何大型项目一样，都经历着设计、规划、执行和交付的相同生命周期。在每个阶段，都在进行持续的调整，一个复杂的项目之所以被称为如此，是有原因的。软件系统在这方面并没有什么不同。我们设计、规划和构建它，然后不断地进行更改和调整。如果我们幸运的话，新的更改不会太大地回到最初的阶段，也不需要改变设计。为了防范这种激烈的步骤，我们使用原型（如果采用瀑布模型）或迭代交付（如果采用敏捷过程）来尽早发现可能的问题。就像年轻的父母一样，我们总是警惕地监视着我们孩子的进展，日夜不停。

正如我们在之前的某个部分中已经提到的，每个 JDK 9 安装都附带了几个诊断工具，或者可以额外使用这些工具来监视您的 Java 应用程序。这些工具的完整列表（以及如何创建自定义工具的建议，如果需要的话）可以在 Oracle 网站的官方 Java SE 文档中找到：[`docs.oracle.com/javase/9/troubleshoot/diagnostic-tools.htm`](https://docs.oracle.com/javase/9/troubleshoot/diagnostic-tools.htm)。

使用这些工具，可以识别应用程序的瓶颈，并通过编程或调整 JVM 本身或两者兼而行之来解决。最大的收益通常来自良好的设计决策以及使用某些编程技术和框架，其中一些我们在其他部分中已经描述过。在本节中，我们将看看在应用所有可能的代码更改后或者当更改代码不是一个选项时可用的选项，因此我们所能做的就是调整 JVM 本身。

努力的目标取决于应用程序的分析结果和非功能性需求：

+   延迟，或者说应用程序对输入的响应速度

+   吞吐量，或者说应用程序在给定时间单位内所做的工作量

+   内存占用，或者说应用程序需要多少内存

其中一个的改进通常只能以牺牲另一个或两者的方式实现。内存消耗的减少可能会降低吞吐量和延迟，而延迟的减少通常只能通过增加内存占用来实现，除非你可以引入更快的 CPU，从而改善这三个特性。

应用程序分析可能会显示，一个特定的操作在循环中不断分配大量内存。如果你可以访问代码，可以尝试优化代码的这一部分，从而减轻 JVM 的压力。另外，它可能会显示涉及 I/O 或其他与低性能设备的交互，并且在代码中无法做任何改进。

定义应用程序和 JVM 调优的目标需要建立指标。例如，已经众所周知，将延迟作为平均响应时间的传统度量隐藏了更多关于性能的信息。更好的延迟指标将是最大响应时间与 99%最佳响应时间的结合。对于吞吐量，一个好的指标将是单位时间内的交易数量。通常，这些指标的倒数（每个交易的时间）会反映延迟。对于内存占用，最大分配的内存（在负载下）允许进行硬件规划，并设置防范可怕的`OutOfMemoryError`异常。避免完整（停止一切）的垃圾收集循环将是理想的。然而，在实践中，如果**Full GC**不经常发生，不明显影响性能，并且在几个周期后最终堆大小大致相同，那就足够了。

不幸的是，这种简单的需求在实践中确实会发生。现实生活中不断出现更多的问题，如下：

+   目标延迟（响应时间）是否会被超过？

+   如果是，频率是多少，幅度是多少？

+   响应时间不佳的时间段可以持续多久？

+   谁/什么在生产中测量延迟？

+   目标性能是峰值性能吗？

+   预期的峰值负载是多少？

+   预期的峰值负载将持续多久？

只有在回答了所有这些类似的问题并建立了反映非功能性需求的指标之后，我们才能开始调整代码，运行它并一遍又一遍地进行分析，然后调整代码并重复这个循环。这项活动必须占用大部分的努力，因为与通过代码更改获得的性能改进相比，调整 JVM 本身只能带来一小部分性能改进。

然而，JVM 调优的几次尝试必须在早期进行，以避免浪费努力并试图将代码强行放入配置不良的环境中。JVM 配置必须尽可能慷慨，以便代码能够充分利用所有可用资源。

首先，从 JVM 9 支持的四种垃圾收集器中选择一个，它们分别是：

+   **串行收集器**：这使用单个线程执行所有垃圾收集工作。

+   **并行收集器**：这使用多个线程加速垃圾收集。

+   **并发标记清除（CMS）收集器**：这使用更短的垃圾收集暂停来换取更多的处理器时间。

+   **垃圾优先（G1）收集器**：这是为多处理器机器和大内存设计的，但在高概率下达到垃圾收集暂停时间目标，同时实现高吞吐量。

官方的 Oracle 文档（[`docs.oracle.com/javase/9/gctuning/available-collectors.htm`](https://docs.oracle.com/javase/9/gctuning/available-collectors.htm)）提供了垃圾收集选择的初始指南：

+   如果应用程序的数据集很小（大约 100MB 以下），则选择带有`-XX:+UseSerialGC`选项的串行收集器。

+   如果应用程序将在单个处理器上运行，并且没有暂停时间要求，则选择带有`-XX:+UseSerialGC`选项的串行收集器。

+   如果（a）峰值应用程序性能是第一优先级，（b）没有暂停时间要求或者接受一秒或更长时间的暂停，那么让虚拟机选择收集器或者选择并行收集器与`-XX:+UseParallelGC`

+   如果响应时间比整体吞吐量更重要，并且垃圾收集暂停必须保持在大约一秒以下，则选择带有`-XX:+UseG1GC`或`-XX:+UseConcMarkSweepGC`的并发收集器。

但是，如果您还没有特定的偏好，让 JVM 选择垃圾收集器，直到您更多地了解您的应用程序的需求。在 JDK 9 中，G1 在某些平台上是默认选择的，如果您使用的硬件资源足够，这是一个很好的开始。

Oracle 还建议使用 G1 的默认设置，然后使用`-XX:MaxGCPauseMillis`选项和`-Xmx`选项来尝试不同的暂停时间目标和最大 Java 堆大小。增加暂停时间目标或堆大小通常会导致更高的吞吐量。延迟也受暂停时间目标的改变影响。

在调整 GC 时，保持`-Xlog:gc*=debug`日志选项是有益的。它提供了许多有关垃圾收集活动的有用细节。JVM 调优的第一个目标是减少完整堆 GC 周期（Full GC）的数量，因为它们非常消耗资源，因此可能会减慢应用程序的速度。这是由老年代区域的占用率过高引起的。在日志中，它被识别为`Pause Full (Allocation Failure)`。以下是减少 Full GC 机会的可能步骤：

+   使用`-Xmx`增加堆的大小。但要确保它不超过物理内存的大小。最好留一些 RAM 空间给其他应用程序。

+   显式增加并发标记线程的数量，使用`-XX:ConcGCThreads`。

+   如果庞大的对象占用了太多堆空间（观察显示在**gc+heap=info**日志中的巨大区域旁边的数字），尝试使用`-XX:G1HeapRegionSize`来增加区域大小。

+   观察 GC 日志，并修改代码，以便您的应用程序创建的几乎所有对象都不会超出年轻代（早夭）。

+   一次添加或更改一个选项，这样您就可以清楚地了解 JVM 行为变化的原因。

这些步骤将帮助您创建一个试错循环，让您更好地了解您正在使用的平台，应用程序的需求，以及 JVM 和所选 GC 对不同选项的敏感性。掌握了这些知识，您将能够通过更改代码、调整 JVM 或重新配置硬件来满足非功能性能要求。

# 响应式编程

经过几次失败的尝试和一些灾难性的中断，然后是英勇的恢复，金字塔建造的过程形成了，古代建筑师们能够完成一些项目。最终的形状有时并不完全如预期（第一座金字塔最终弯曲了），但是，金字塔至今仍然装饰着沙漠。经验代代相传，设计和工艺经过调整，能够在 4000 多年后产生一些宏伟而令人愉悦的东西。

软件实践也随着时间而改变，尽管图灵先生编写了第一个现代程序只有大约 70 年。起初，当世界上只有少数程序员时，计算机程序通常是一连串的指令。函数式编程（像第一类公民一样推动函数）也很早就被引入，但并没有成为主流。相反，**GOTO**指令允许您将代码卷入意大利面条般的混乱中。接着是结构化编程，然后是面向对象编程，函数式编程也在某些领域蓬勃发展。许多程序员已经习惯了异步处理按键生成的事件。JavaScript 试图使用所有最佳实践，并获得了很大的力量，尽管在调试（有趣）阶段程序员会感到沮丧。最后，随着线程池和 lambda 表达式成为 JDK SE 的一部分，将响应式流 API 添加到 JDK 9 中，使 Java 成为允许使用异步数据流进行响应式编程的家庭的一部分。

公平地说，即使没有这个新的 API，我们也能够异步处理数据--通过旋转工作线程和使用线程池和可调用对象（正如我们在前面的部分中所描述的）或通过传递回调（即使偶尔在谁调用谁的迷宫中迷失）。但是，在几次编写这样的代码之后，人们会注意到大多数这样的代码只是一个可以包装在框架中的管道，可以显著简化异步处理。这就是响应式流倡议（[`www.reactive-streams.org`](http://www.reactive-streams.org)）的创建背景和努力的范围定义如下：

响应式流的范围是找到一组最小的接口、方法和协议，描述必要的操作和实体以实现异步数据流和非阻塞背压。

术语**非阻塞背压**是重要的，因为它确定了现有异步处理的问题之一--协调传入数据的速率与系统处理这些数据的能力，而无需停止（阻塞）数据输入。解决方案仍然会包括一些背压，通过通知源消费者在跟不上输入时存在困难，但新框架应该以更灵活的方式对传入数据的速率变化做出反应，而不仅仅是阻止流动，因此称为**响应式**。

响应式流 API 由包含在类中的五个接口组成，它们是`java.util.concurrent.Flow`、`Publisher`、`Subscriber`、`Subscription`和`Processor`：

```java
@FunctionalInterface
public static interface Flow.Publisher<T> {
  public void subscribe(Flow.Subscriber<? super T> subscriber);
}

public static interface Flow.Subscriber<T> {
  public void onSubscribe(Flow.Subscription subscription);
  public void onNext(T item);
  public void onError(Throwable throwable);
  public void onComplete();
}

public static interface Flow.Subscription {
  public void request(long numberOfItems);
  public void cancel();
}

public static interface Flow.Processor<T,R> 
               extends Flow.Subscriber<T>, Flow.Publisher<R> {
}
```

在`Flow.Publisher`对象的`subscribe()`方法中将`Flow.Subscriber`对象作为参数传递后，`Flow.Subscriber`对象成为`Flow.Publisher`对象产生的数据的订阅者。发布者（`Flow.Publisher`对象）调用订阅者的`onSubscribe()`方法，并将`Flow.Subscription`对象作为参数传递。现在，订阅者可以通过调用订阅的`request()`方法从发布者那里请求`numberOffItems`个数据。这是实现拉模型的方式，订阅者决定何时请求另一个项目进行处理。订阅者可以通过调用`cancel()`订阅方法取消订阅发布者的服务。

作为回报（或者如果实现者决定这样做，那将是一种推送模型），发布者可以通过调用订阅者的`onNext()`方法向订阅者传递一个新项目。发布者还可以告诉订阅者，项目生产遇到了问题（通过调用订阅者的`onError()`方法）或者不会再有数据传入（通过调用订阅者的`onComplete()`方法）。

`Flow.Processor`接口描述了一个既可以充当订阅者又可以充当发布者的实体。它允许创建这些处理器的链（管道），因此订阅者可以从发布者那里接收一个项目，对其进行调整，然后将结果传递给下一个订阅者。

这是 Reactive Streams 倡议定义的最小接口集（现在是 JDK 9 的一部分），支持非阻塞背压的异步数据流。正如您所看到的，它允许订阅者和发布者相互交流和协调，如果需要的话，协调传入数据的速率，从而使我们在开始讨论时所讨论的背压问题有可能有各种解决方案。

有许多实现这些接口的方法。目前，在 JDK 9 中，只有一个接口实现的例子——`SubmissionPublisher`类实现了`Flow.Publisher`。但已经存在几个其他库实现了 Reactive Streams API：RxJava、Reactor、Akka Streams 和 Vert.x 是其中最知名的。我们将在我们的示例中使用 RxJava 2.1.3。您可以在[`reactivex.io`](http://reactivex.io)上找到 RxJava 2.x API，名称为 ReactiveX，代表 Reactive Extension。

在这样做的同时，我们也想解释一下`java.util.stream`包和响应式流（例如 RxJava 中实现的）之间的区别。可以使用任何一种流编写非常相似的代码。让我们看一个例子。这是一个程序，它遍历五个整数，只选择偶数（2 和 4），对每个偶数进行转换（对每个选定的数字取平方根），然后计算两个平方根的平均值。它基于传统的`for`循环。

让我们从相似性开始。可以使用任何一种流来实现相同的功能。例如，这是一个方法，它遍历五个整数，只选择偶数（在这种情况下是 2 和 4），对每个偶数进行转换（对每个偶数取平方根），然后计算两个平方根的平均值。它基于传统的`for`循环：

```java
void demo_ForLoop(){
    List<Double> r = new ArrayList<>();
    for(int i = 1; i < 6; i++){
        System.out.println(i);
        if(i%2 == 0){
            System.out.println(i);
            r.add(doSomething(i));
        }
    }
    double sum = 0d;
    for(double d: r){ sum += d; }
    System.out.println(sum / r.size());
}
static double doSomething(int i){
    return Math.sqrt(1.*i);
}
```

如果我们运行这个程序，结果将如下所示：

![响应式编程](img/03_30.jpg)

相同的功能（具有相同的输出）也可以使用`java.util.stream`包来实现，如下所示：

```java
void demo_Stream(){
    double a = IntStream.rangeClosed(1, 5)
        .peek(System.out::println)
        .filter(i -> i%2 == 0)
        .peek(System.out::println)
        .mapToDouble(i -> doSomething(i))
        .average().getAsDouble();
    System.out.println(a);
}
```

相同的功能也可以使用 RxJava 实现：

```java
void demo_Observable1(){
    Observable.just(1,2,3,4,5)
        .doOnNext(System.out::println)
        .filter(i -> i%2 == 0)
        .doOnNext(System.out::println)
        .map(i -> doSomething(i))
        .reduce((r, d) -> r + d)
        .map(r -> r / 2)
        .subscribe(System.out::println);
}
```

RxJava 基于`Observable`对象（扮演`Publisher`的角色）和订阅`Observable`并等待数据被发射的`Observer`。从`Observable`到`Observer`的每个发射数据项都可以通过以流畅的方式链接的操作进行处理（参见之前的代码）。每个操作都采用 lambda 表达式。操作功能从其名称中很明显。

尽管能够表现得与流类似，但`Observable`具有显着不同的功能。例如，流一旦关闭，就无法重新打开，而`Observable`可以被重复使用。这是一个例子：

```java
void demo_Observable2(){
    Observable<Double> observable = Observable
            .just(1,2,3,4,5)
            .doOnNext(System.out::println)
            .filter(i -> i%2 == 0)
            .doOnNext(System.out::println)
            .map(Demo05Reactive::doSomething);

    observable
            .reduce((r, d) -> r + d)
            .map(r -> r / 2)
            .subscribe(System.out::println);

    observable
            .reduce((r, d) -> r + d)
            .subscribe(System.out::println);
}
```

在之前的代码中，我们两次使用了`Observable`——一次用于计算平均值，一次用于对偶数的平方根求和。输出如下截图所示：

![响应式编程](img/03_31.jpg)

如果我们不希望`Observable`运行两次，可以通过添加`.cache()`操作来缓存其数据：

```java
void demo_Observable2(){
    Observable<Double> observable = Observable
            .just(1,2,3,4,5)
            .doOnNext(System.out::println)
            .filter(i -> i%2 == 0)
            .doOnNext(System.out::println)
            .map(Demo05Reactive::doSomething)
            .cache();

    observable
            .reduce((r, d) -> r + d)
            .map(r -> r / 2)
            .subscribe(System.out::println);

    observable
            .reduce((r, d) -> r + d)
            .subscribe(System.out::println);
}
```

之前代码的结果如下：

![响应式编程](img/03_32.jpg)

您可以看到同一个`Observable`的第二次使用利用了缓存的数据，从而实现了更好的性能。

另一个`Observable`的优势是异常可以被`Observer`捕获：

```java
subscribe(v -> System.out.println("Result=" + v),
        e -> {
            System.out.println("Error: " + e.getMessage());
            e.printStackTrace();
        },
        () -> System.out.println("All the data processed"));
```

`subscribe()`方法是重载的，允许传入一个、两个或三个函数：

+   第一个用于成功的情况

+   第二个用于异常情况

+   第三个在所有数据处理完毕后调用

`Observable`模型还允许更多控制多线程处理。在流中使用`.parallel()`不允许您指定要使用的线程池。但是，在 RxJava 中，您可以使用`Observable`中的`subscribeOn()`方法设置您喜欢的池类型：

```java
observable.subscribeOn(Schedulers.io())
        .subscribe(System.out::println);
```

`subscribeOn()`方法告诉`Observable`在哪个线程上放置数据。`Schedulers`类有生成线程池的方法，主要处理 I/O 操作（如我们的示例中），或者处理计算密集型操作（`computation()`方法），或者为每个工作单元创建一个新线程（`newThread()`方法），以及其他几种方法，包括传入自定义线程池（`from(Executor executor)`方法）。

这本书的格式不允许我们描述 RxJava API 和其他响应式流实现的所有丰富性。它们的主要目的反映在响应式宣言（[`www.reactivemanifesto.org/`](http://www.reactivemanifesto.org/)）中，该宣言将响应式系统描述为新一代高性能软件解决方案。建立在异步消息驱动进程和响应式流上，这些系统能够展示响应式宣言中声明的特性：

+   **弹性**：具有根据负载需要扩展和收缩的能力

+   **更好的响应性**：在这里，处理可以使用异步调用进行并行化

+   **弹性**：在这里，系统被分解为多个（通过消息松耦合）组件，从而促进灵活的复制、封装和隔离

使用响应式流来编写响应式系统的代码，以实现先前提到的特性，构成了响应式编程。这种系统今天的典型应用是微服务，下一课将对此进行描述。

# 摘要

在本课中，我们讨论了通过使用多线程来改善 Java 应用程序性能的方法。我们描述了如何通过使用线程池和适用于不同处理需求的各种类型的线程池来减少创建线程的开销。我们还提出了用于选择池大小的考虑因素，以及如何同步线程，使它们不会相互干扰，并产生最佳性能结果。我们指出，对性能改进的每个决定都必须通过直接监视应用程序进行制定和测试，并讨论了通过编程和使用各种外部工具进行此类监视的可能选项。最后一步，JVM 调优，可以通过我们在相应部分列出并评论的 Java 工具标志来完成。采用响应式编程的概念可能会使 Java 应用程序的性能获得更多收益，我们将其作为朝着高度可伸缩和高性能 Java 应用程序的最有效举措之一。

在下一课中，我们将讨论通过将应用程序拆分为多个微服务来添加更多的工作线程，每个微服务都独立部署，并且每个微服务都使用多个线程和响应式编程以获得更好的性能、响应、可伸缩性和容错性。

# 评估

1.  命名一个方法，计算前 99,999 个整数的平均平方根，并将结果分配给可以随时访问的属性。

1.  以下哪种方法创建了一个固定大小的线程池，可以在给定延迟后安排命令运行，或定期执行：

1.  新的调度线程池()

1.  新的工作窃取线程池()

1.  新的单线程调度执行器()

1.  新的固定线程池()

1.  陈述是否正确：可以利用`Runnable`接口是一个函数式接口，并将必要的处理函数作为 lambda 表达式传递到新线程中。

1.  在调用`__________`方法之后，不能再向池中添加更多的工作线程。

1.  shutdownNow()

1.  shutdown()

1.  isShutdown()

1.  isShutdownComplete()

1.  ________ 基于`Observable`对象，它扮演着发布者的角色。
