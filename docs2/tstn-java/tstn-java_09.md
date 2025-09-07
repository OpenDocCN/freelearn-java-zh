# 9

# 在 Java 中使用线程

我最早的一个软件开发合同是为美国肯塔基州纯血马场开发隐形围栏安全系统软件。我们使用的计算机是 Apple II plus。在 6502 CPU 或 ProDOS 操作系统中没有线程的概念。我们所做的是用汇编语言编写所有代码，这些代码是以每个单元所需的周期数来衡量的。一旦我们完成了分配的周期，我们就会在内存的一个定义区域中保存我们的状态，并将控制权交给下一个单元。这工作得相当好，如果马走失了，就会响起警报。即使在警报响起的同时，围栏的监控，也就是可以检测到马走过它的地下电缆，也会继续进行。这是我接触到的线程编程。这是在 1982 年。

我直到 1999 年从 C++迁移到 Java 后，才再次与线程打交道。使 Java 脱颖而出的一个特性，以及我放弃 C++的原因，是 Java 语言对线程的标准支持。此外，Swing 对 GUI 应用程序的支持也让我清楚地认识到，Java 是我项目中的学生需要学习的语言。2002 年，多伦多道格森学院的计算机科学技术专业，我是该专业的负责人，放弃了 COBOL 作为主要教学语言，转而使用 Java。

今天，包括 C++在内的许多语言都原生支持多线程编程。在本章中，我们将探讨如何在 Java 中编写依赖于 Java 虚拟机与计算机操作系统协同工作的线程代码。当线程共享资源时，需要处理一些问题。我们将探讨使用同步来处理这些问题。在本章中，我们将探讨以下内容：

+   创建 Java 本地操作系统线程

+   防止线程中的竞争和死锁条件

+   创建新的虚拟线程

让我们从探讨如何使用本地线程编写线程代码开始。

# 技术要求

下面是运行本章示例所需的工具：

+   安装 Java 17 以仅使用本地线程

+   文本编辑器

+   安装 Maven 3.8.6 或更高版本

本章的示例代码可在[`github.com/PacktPublishing/Transitioning-to-Java/tree/chapter09`](https://github.com/PacktPublishing/Transitioning-to-Java/tree/chapter09)找到。

# 创建 Java 本地操作系统线程

“本地线程”一词指的是由计算机操作系统管理的线程。当我们创建 Java 本地线程时，我们指的是 JVM 使用底层操作系统的线程库 API 管理的线程。这也意味着 JVM 处理不同操作系统上的不同线程库，而我们使用 Java API 来创建线程。在苹果 Mac 上编写的使用线程的程序可以在 Windows 机器上运行，因为 JVM 处理线程的最低级别。

我们将探讨三种创建 Java 本地线程的方法以及一种创建线程池的方法。这些将涉及以下内容：

+   扩展`Thread`类

+   实现`Runnable`接口

+   使用`ExecutorService`创建线程池

+   实现`Callable`接口

+   线程管理

我们将要讨论的最后几个主题是以下内容：

+   守护线程和非守护线程

+   线程优先级

## 扩展`Thread`类

任何扩展`Thread`类的类都可以包含作为线程一部分执行的方法。仅仅创建一个对象并不会创建线程。相反，扩展`Thread`类的类必须重写`Thread`超类的`run`方法。这个方法中的任何内容都成为线程。`run`方法可以执行线程中的所有工作。除非这是一个简单的任务，否则`run`方法就像我在之前的示例中使用的`perform`方法一样。`run`方法中的所有内容都是这个线程将要执行的操作。让我们看看一个扩展`Thread`类的简单类。

我们扩展`Thread`类来表示这个类中的代码将被线程化：

```java
public class ThreadClass extends Thread {
```

在这个例子中，每个线程将要执行的工作是从我们最初分配给名为`actionCounter`的字段的任何值开始倒数：

```java
    private int actionCounter = 25;
```

线程可以被赋予一个名称。我们希望在这个例子中的每个线程都有一个数字作为其名称。因此，它必须是一个静态变量，因为不管我们创建多少个`threadCounter`实例，都只有一个`threadCounter`整数。静态字段被认为是线程安全的，这意味着如果有两个或更多线程想要访问静态字段，则不会发生冲突：

```java
    private static int threadCounter = 0;
```

构造函数将线程的名称分配给超类的构造函数。每次我们创建这个对象的实例时，`threadCounter`的值将与前一个实例设置的值相同。这使得每个线程都有一个唯一的名称：

```java
    public ThreadClass() {
        super("" + ++threadCounter);
    }
```

这个线程唯一要执行的任务是显示其名称和`actionCounter`字段的当前值。我们重写了在对象引用必须作为`String`操作时调用的`toString`方法。它将返回一个字符串，由分配给它的名称组成，通过调用超类的`getName`方法，以及`actionCounter`的当前值：

```java
    @Override
    public String toString() {
        return "#" + getName() + " : " + actionCounter;
    }
```

线程类必须重写超类的`run`方法。线程的工作就在这里发生。在这种情况下，我们使用一个无限`while`循环，在这个循环中我们显示这个对象的线程名称和`actionCounter`的当前值。当`actionCounter`达到零时，我们从`run`方法返回，线程结束。使用无限循环语法`while (true)`意味着结束循环的决定是基于循环中发生的事情，在这种情况下，是递减`actionCounter`直到它达到零。这不是编写`run`方法的唯一方法，但这是最常见的方法：

```java
    @Override
    public void run() {
        System.out.printf("extends Thread%n");
        while (true) {
            System.out.printf("%s%n", this);
            if (--actionCounter == 0) {
                return;
            }
        }
    }
}
```

在我们的线程类就绪后，我们现在可以编写一个类，该类将实例化和运行每个线程：

```java
public class ThreadClassRunner {
```

在这里，在`perform`中，我们创建了`ThreadClass`的五个实例。我们调用`start`而不是`run`。`start`方法是对`Thread`超类`start`方法的覆盖，它负责设置线程并调用`run`方法。自己调用`run`方法不会启动线程：

```java
    public void perform() {
        for (int i = 0; i < 5; i++) {
            new ThreadClass().start();
        }
    }
    public static void main(String[] args) {
        new ThreadClassRunner().perform();
    }
}
```

重要提示

线程是非确定性的。

这是一个始终需要注意的重要点。每次运行这个示例代码时，输出都会不同。线程的执行顺序与它们创建的顺序无关。运行这个示例几次并注意每次结果的顺序都是不同的。

这种方法有一个问题。你无法在`ThreadClass`中扩展任何其他超类。这把我们带到了第二种方法。

## 实现 Runnable 接口

在这种方法中，我们实现了`Runnable`接口。我们执行的任务与上一个示例相同：

```java
public class ThreadRunnableInterface implements Runnable{
    private int actionCounter = 25;
    @Override
    public String toString() {
```

我们调用`Thread.currentThread().getName()`来获取这个线程的名称。当我们扩展`Thread`类时，我们可以调用`getName`。由于我们正在实现`Runnable`接口，我们没有超类方法可以调用。我们现在通过使用`Thread`类的静态方法来获取名称，这些方法将返回调用这些方法时当前线程的信息：

```java
        return "#" + Thread.currentThread().getName() +
                 " : " + actionCounter;
    }
```

`run`方法没有改变：

```java
    @Override
    public void run() {
        while (true) {
            System.out.printf("%s%n", this);
            if (--actionCounter == 0) {
                return;
            }
        }
    }
}
```

在使用`Runnable`接口时，启动线程的类中的`perform`方法有所不同：

```java
    public void perform() {
        System.out.printf("implements Runnable%n");
        for (int i = 0; i < 5; i++) {
```

我们通过实例化一个`Thread`类，将其构造函数传递给`Runnable`线程类的一个实例以及线程的名称来创建这些线程。在这个例子中，`Thread`对象是匿名的；我们没有将其分配给一个变量，并在其上调用`start`：

```java
            new Thread(new ThreadRunnableInterface(), ""
                          + ++i).start();
        }
    }
```

就像上一个示例一样，每次运行都会得到不同的输出。

应该使用哪种技术？当前的最好实践是优先选择`Runnable`接口。这允许你在线程化的同时扩展另一个类。让我们看看线程池。

## 使用 ExecutorService 创建线程池

到目前为止我们所看到的内容要求我们为`Thread`类的每一个实例创建一个线程。一种替代方法是创建一个可以重复使用的线程池。这就是`ExecutorService`方法发挥作用的地方。使用这种方法，我们可以在定义最大并发数的同时创建一个线程池。如果需要的线程数超过了池中允许的数量，那么线程将等待直到一个正在执行的线程结束。让我们改变我们的基本示例以使用这个服务。

我们从一个实现了`Runnable`接口的类开始。`actionCounter`字段是线程中将递减的数字：

```java
public class ExecutorThreadingInterface implements Runnable {
    private int actionCounter = 250;
```

由于我们将创建 `Thread` 类的任务留给 `ExecutorService`，我们不再有接受线程名称 `String` 的构造函数。我们将把名称作为 `int` 传递给构造函数，并在这里存储它。成为单个线程的类的字段将各自拥有自己的 `actionCounter` 和 `threadCount` 实例：

```java
    private final int threadCount;
```

这里是接受我们想要知道的线程名称的构造函数：

```java
    public ExecutorThreadingInterface(int count) {
        threadCount = count;
    }
```

我们重写 `toString` 方法以返回包含当前线程名称的 `String`，这是由 `ExecutorService` 分配的，以及我们分配给 `threadCount` 的名称，后面跟着 `actionCounter` 的当前值，它在线程运行时减少：

```java
    @Override
    public String toString() {
        return "#" + Thread.currentThread().getName()
            + "-" + threadCount + " : " + actionCounter;
    }
```

最后一个方法是 `run`。这保持不变：

```java
    @Override
    public void run() {
        while (true) {
            System.out.printf("%s%n", this);
            if (--actionCounter == 0) {
                return;
            }
        }
    }
}
```

现在，让我们看看我们如何使用 `ExecutorService` 来创建线程：

```java
public class ExecutorServiceRunner {
```

我选择将我们使用的变量作为字段。它们都可以在单个方法中声明为局部变量：

```java
    private final int numOfThreads = 5;
    private final int threadPoolSize = 2;
    private final ExecutorService service;
```

构造函数现在负责实例化 `ExecutorService`，以及一个 `Runnable` 线程数组：

```java
    public ExecutorServiceRunner() {
        service =
             Executors.newFixedThreadPool(threadPoolSize);
    }
    public void perform() {
        for (int i = 0; i < numOfThreads; i++) {
```

我们使用 `execute` 方法将线程添加到 `ExecutorService`。我们不需要访问线程，因此它们是匿名实例化的：

```java
            service.execute(
                      new ExecutorThreadingInterface(i));
        }
```

所有线程完成后，服务将关闭。在此方法调用后，你不能再向服务添加任何线程：

```java
        service.shutdown();
    }
```

我们以通常的 `main` 方法结束这个类：

```java
    public static void main(String[] args) {
        new ExecutorServiceRunner().perform();
    }
}
```

这三种方法允许你轻松地创建线程。它们共同的一个问题是，当线程结束时，它不会返回一个值，因为 `run` 是空值。如果需要，我们可以通过结合 `ExecutorService` 和使用 `Callable` 接口的一种第三类线程类来解决这个问题。

## 实现 `Callable` 接口

在我们看到的每个线程类中，它们都有一个 `run` 方法，当线程结束时返回空值。这引导我们到 `Callable` 接口。使用这个接口，线程的结束会返回一个值。我们只能在使用 `ExecutorService` 的情况下使用这种技术。让我们先看看一个 `Callable` 线程类。

我们从想要线程化的类开始。我们实现 `Callable` 接口，并使用泛型表示法，声明线程结束时返回的值将是一个字符串。字段、构造函数和 `toString` 方法与 `ExecutorThreadingInterface` 相同：

```java
public class ThreadCallableInterface
                          implements Callable<String> {
    private int actionCounter = 250;
    private final int threadCount;
    public ThreadCallableInterface(int count) {
        threadCount = count;
    }
    @Override
    public String toString() {
        return "#" + Thread.currentThread().getName() +
                "-" + threadCount + " : " + actionCounter;
    }
```

在这里，我们将 `run` 替换为 `call` 并显示返回类型。`return` 语句将显示我们分配给每个线程的线程名称，作为一个整数。

```java
    @Override
    public String call() {
        while (true) {
            System.out.printf("%s%n", this);
            if (--actionCounter == 0) {
                return "Thread # " + threadCount +
                                          " is finished";
            }
        }
    }
}
```

现在，让我们看看这个 `Callable` 线程的运行者。

```java
public class ThreadCallableInterfaceRunner {
```

我们声明的第一个变量是`Future`类型的`List`。`Future`是一个接口，就像`List`一样。当我们使用（而不是执行）`ExecutorService`的`submit`方法时，它返回一个实现`Future`接口的对象。实现此接口的对象代表异步任务的结果。当我们在这里几行之后实例化此对象时，它将是线程传递的`Future`字符串的`List`：

```java
    private final List<Future<String>> futureList;
    private final ExecutorService executor;
    private final int numOfThreads = 5;
    private final int threadPoolSize = 2;
```

这个例子中的一个新特性是显示每个线程结束时的当前日期和时间。`DateTimeFormatter`对象将`LocalDateTime`对象转换为可读的字符串：

```java
    private final DateTimeFormatter dtf;
```

构造函数实例化了类的字段：

```java
    public ThreadCallableInterfaceRunner() {
        executor =
            Executors.newFixedThreadPool(threadPoolSize);
```

我们将`futureList`实例化为`ArrayList`。随后我们定义我们想要的日期和时间的格式：

```java
        futureList = new ArrayList<>();
        dtf = DateTimeFormatter.ofPattern(
                                "yyyy/MM/dd HH:mm:ss");
    }
    public void perform(){
```

在这里，我们使用`submit`将线程提交给`ExecutorService`。使用`submit`意味着我们期望返回类型为`Future`。我们还把每个`Future`对象添加到一个`ArrayList`实例中：

```java
        for (int i = 0; i < numOfThreads; i++) {
            Future<String> future = executor.submit(
                           new ThreadCallableInterface(i));
            futureList.add(future);
        }
```

在这里，我们遍历`ArrayList`实例，显示当前日期和时间以及线程返回的值——在这个例子中，是`String`。我们通过调用`get`方法来访问`Future`对象的返回值。这是一个阻塞的方法调用。每个`Future`对象都与一个特定的线程相关联，`get`将在允许下一个`Future`对象的`get`执行之前等待其结果。对`get`的调用可能导致两个检查型异常，因此我们必须将调用放在`try`/`catch`块中。为了这个示例的目的，我们只是打印堆栈跟踪。你永远不应该在没有采取任何适当行动的情况下打印堆栈跟踪：

```java
        for (Future<String> futureResult : futureList) {
            try {
                System.out.println(
                    dtf.format(LocalDateTime.now()) + ":" +
                    futureResult.get());
            } catch (InterruptedException |
                             ExecutionException e) {
                e.printStackTrace();
            }
        }
```

当你不再需要服务时，必须显式关闭`ExecutorService`：

```java
        executor.shutdown();
    }
```

我们以通常的`main`方法结束：

```java
    public static void main(String[] args) {
        new ThreadCallableInterfaceRunner().perform();
    }
}
```

现在我们已经回顾了创建`Threads`最常见的方法，让我们更深入地看看我们如何管理一个线程。

## 线程管理

有三种常用的`Thread`方法用于管理线程。这些如下：

+   `yield()`

+   `join()`

+   `sleep()`

`yield()`方法通知线程调度器它可以放弃当前对处理器的使用，但希望尽快重新调度。这只是一个建议，调度器可以自由地做它想做的事情。这使得它是非确定性的，并且依赖于它运行的平台。只有在可以证明的情况下，通常通过代码分析，它才能提高性能时才应该使用。

当一个线程（我们将称之为`join()`）影响创建了第二个线程的第一个线程时，`join()`方法可能很有用。`join()`有两个额外的重载版本，允许你设置阻塞启动线程的时间长度，可以是毫秒，也可以是毫秒和纳秒。

最后一种方法是`Thread`类的一个静态方法。`sleep()`方法会使正在执行的线程暂停特定的时间长度。时间可以是，就像`join`一样，以毫秒为单位，也可以是毫秒和纳秒。

`join()`和`sleep()`的一个共同特点是它们可以抛出检查异常。它们必须在`try`/`catch`块中编码。以下是一个线程类，它实例化和启动第二个类，但随后与第二个类连接，从而阻塞自身，直到它启动的线程完成。

这就像我们之前看到的第一个`ThreadClass`实例。区别在于它实例化另一个线程类，然后在`run`方法的开头启动该线程：

```java
public class ThreadClass1 extends Thread {
    private int actionCounter = 500;
    private static int threadCounter = 0;
    private final ThreadClass2 tc2;
    public ThreadClass1() {
        super("" + ++threadCounter);
        tc2 = new ThreadClass2();
    }
    @Override
    public String toString() {
        return "#" + getName() + " : " + actionCounter;
    }
    @Override
    public void run() {
```

在这里，我们启动第二个线程，它现在将根据调度器的决定执行：

```java
        tc2.start();
        while (true) {
            System.out.printf("%s%n", this);
```

当第一个线程达到 225 时，我们对第二个线程发出连接请求。结果将是第一个线程被阻塞，第二个线程将一直运行到完成，然后才会解除第一个线程的阻塞：

```java
                if (actionCounter == 225) {
                    try {
                        tc2.join();
                    } catch (InterruptedException ex) {
                        ex.printStackTrace();
                    }
                }
            if (--actionCounter == 0) {
                return;
            }
        }
    }
}
```

## 守护线程和非守护线程

“守护”一词指的是被认为是低优先级的线程。这意味着任何被指定为守护的本地线程都将结束，无论在应用程序的主线程结束时它们正在做什么。

非守护线程，当创建本地线程时的默认设置，将阻止应用程序的主线程结束，直到该线程完成任务。这告诉我们非守护线程必须有一个结束条件。守护线程不需要结束条件，因为它会随着主线程结束。

你可以通过简单的调用方法将本地线程设置为守护线程：

```java
thread.setDaemon(true);
```

你只能在线程实例化后但在它启动之前调用此方法。一旦它开始，就不能更改守护线程的状态。调用此方法会导致抛出异常。

## 线程优先级

正如已经指出的，线程是非确定性的。这意味着我们无法绝对控制线程何时获得其运行的时间片或该时间片将持续多长时间。我们可以提出一个建议，也称为提示，这就是线程优先级的作用所在。

线程优先级的可能值范围是 1 到 10。在大多数情况下，使用三个定义的静态常量而不是数字。如下所示：

+   `Thread.MAX_PRIORITY`

+   `Thread.MIN_PRIORITY`

+   `Thread.NORM_PRIORITY`

正如这所暗示的，你不能依赖最大优先级比最小优先级获得更多的时间片。它应该获得更多的时间片，这是你能期望的最好的结果。

现在我们已经看到了如何管理我们的线程，让我们再看看一个额外的主题，线程安全。

# 防止线程中的竞态和死锁条件

有两个常见问题可能导致线程代码出现问题。第一个是竞态条件。当两个或更多线程共同操作一个改变所有线程共享变量的代码块时，就会发生这种情况。

第二种是死锁条件。为了解决竞态条件，你锁定一段代码。如果多个线程使用相同的锁对象，那么可能会出现这些线程都在等待其他线程完成锁但没有任何一个线程完成的情况。让我们更仔细地看看这两个条件。

## 竞态条件

想象一个场景，你需要在多个线程之间共享一个对象的引用。调用这个共享类中只使用局部变量的方法是线程安全的。在这种情况下，线程安全发生是因为每个线程都维护自己的私有栈用于局部变量。线程之间不可能存在冲突。

如果共享对象的方法访问并修改类字段，情况就不同了。与局部变量不同，类字段是共享对象的唯一属性，并且每个调用此类方法线程都有可能修改字段。在这些字段上的操作可能不会在线程的时间片结束之前完成。现在，想象一下，一个线程期望基于上次访问该字段的时间片，字段具有特定的值。然而，它并不知道，另一个线程已经改变了这个值。这导致所谓的竞态条件。让我们看看一个例子。

这里是一个简单的类，它将传递的值添加到名为`counter`的类字段中，并返回加法的结果。每次我们调用`addUp`时，我们都期望`counter`字段改变值：

```java
public class Adder {
    private long counter = 0;
    public long addUp(long value) {
        counter += value;
```

线程从调度器获得的时间与你的计算机的 CPU 有关。高时钟频率以及多个 CPU 核心有时允许线程在下一个线程接管之前完成其任务。因此，我通过让`addUp`方法休眠半秒钟来减慢了`addUp`方法：

```java
        try {
            Thread.sleep(500);
        } catch (InterruptedException ex) {
            ex.printStackTrace();
        }
        return counter;
    }
}
```

线程类基于我们之前看到的`ThreadClass`：

```java
public class SynchronizedThreadClass extends Thread {
    private int actionCounter = 5;
    private static int threadCounter = 0;
```

我们有一个字段用于存储对`Adder`对象的引用。无论我们创建多少个线程，它们都将共享字段变量：

```java
    private final Adder adder;
```

构造函数接收对`Adder`对象的引用：

```java
    public SynchronizedThreadClass(Adder adder) {
        super("" + ++threadCounter);
        this.adder = adder;
    }
    @Override
    public String toString() {
        return "#" + getName() + " : " + actionCounter;
    }
    @Override
    public void run() {
        while (true) {
            var value = adder.addUp(2);
```

在这里，我们正在打印有关此线程和`addUp`方法当前值的信息：

```java
            System.out.printf(
                 "%s : %d%n", this, adder.addUp(2));
            if (--actionCounter == 0) {
                return;
            }
        }
    }
}
```

这里是`main`类：

```java
public class SynchronizedExample {
```

我们将有两个类型的`SynchronizedThreadClass`线程：

```java
    private final SynchronizedThreadClass tc1;
    private final SynchronizedThreadClass tc2;
    private final Adder sa;
```

在我们实例化每个线程之前，我们创建一个单独的`Adder`对象，并将其与每个线程类共享：

```java
    public SynchronizedExample() {
        sa = new Adder();
        tc1 = new SynchronizedThreadClass(sa);
        tc2 = new SynchronizedThreadClass(sa);
    }
    public void perform() {
        tc1.start();
        tc2.start();
    }
    public static void main(String[] args) {
        new SynchronizedExample().perform();
    }
}
```

此代码尚未同步。以下是访问加法器未同步时的结果表：

| **未同步** |
| --- |
| **线程** | **线程** **类动作计数器** | **Adder** **类计数器** |
| #1 | 5 | 4 |
| #2 | 5 | 4 |
| #1 | 4 | 8 |
| #2 | 4 | 10 |
| #1 | 3 | 12 |
| #2 | 3 | 14 |
| #1 | 2 | 16 |
| #2 | 2 | 18 |
| #1 | 1 | 20 |
| #2 | 1 | 20 |

表 9.1 – 运行未同步代码的结果

预期的是`Adder`类计数器应该从 2 计数到 20。但它没有。第一个线程开始将传递的值 2 加到计数器上。但在它能够显示结果之前，第二个线程出现了，并将 2 加到同一个计数器上，现在值增加到 4。当我们回到第一个线程去显示结果时，它现在是 4，而不是它第一次时间片结束时的 2。如果你多次运行这个程序，结果将会不同，但我们将在输出的其他地方看到这个问题。

现在，让我们同步代码。同步将锁应用于代码的一个部分，通常称为临界区。锁是对一个对象的引用，因为所有对象，凭借它们的`Object`超类，都可以用作锁。我们只需要更改`Adder`类，特别是`addUp`方法：

```java
    public long addUp(long value) {
        synchronized (this) {
            counter += value;
            try {
                Thread.sleep(10);
            } catch (InterruptedException ex) {
                ex.printStackTrace();
            }
            return counter;
        }
    }
```

由于整个方法被视为临界区，我们可以移除同步块并将`synchronized`关键字应用于方法名：

```java
    public synchronized long addUp(long value) {
```

这里是使用`synchronized`版本的`addUp` `Adder`类方法的结果表：

| **同步** |
| --- |
| **线程** | **线程** **类 actionCounter** | **类 Adder** **的 counter** |
| #1 | 5 | 2 |
| #2 | 5 | 4 |
| #1 | 4 | 6 |
| #2 | 4 | 8 |
| #1 | 3 | 10 |
| #2 | 3 | 12 |
| #1 | 2 | 14 |
| #2 | 2 | 16 |
| #1 | 1 | 18 |
| #2 | 1 | 20 |

表 9.2 – 运行代码同步的结果

你可以看到从 2 到 20 的每个值都出现了。

作为一名开发者，你总是在寻找可以并发执行的任务。一旦确定，你将在适当的地方应用线程。任何长时间运行的任务都适合作为线程的候选。用户界面通常在一个或多个线程中运行界面，当从菜单或按钮中选择任务时，它们也在线程中运行。这意味着用户界面即使在执行长时间运行的任务时也能对你做出响应。

现在，让我们看看如果我们不正确地使用相同的锁对象同步代码块可能会出现的问题。

## 死锁条件

当线程锁交织在一起时，尤其是在一个线程嵌套在另一个线程内部时，会发生线程死锁。这导致每个线程都在等待另一个线程结束。当使用`synchronized`时，锁可以是 Java 中任何将保护临界区的对象或类，通常是为了避免竞态条件。你还可以创建`Lock`或`ReentrantLock`类型的对象。无论哪种方法，正如我们将看到的，都可能导致死锁。死锁可能很难识别，因为它不会使程序崩溃或抛出异常。让我们看看一个会导致死锁的代码示例。

我们首先创建锁对象所在的类，然后我们使用它们启动两个线程：

```java
public class Deadlock1 {
```

这里是我们将在`Thread1`和`Thread2`中使用的两个锁对象。Java 中的任何对象，无论是你创建的还是已经存在的，例如`String`，都可以用作锁：

```java
    public final Object lock1 = new Object();
    public final Object lock2 = new Object();
    public void perform() {
        var t1 = new ThreadLock1(lock1, lock2);
        var t2 = new ThreadLock2(lock1, lock2);
        t1.start();
        t2.start();
    }
    public static void main(String args[]) {
        new Deadlock1().perform();
    }
}
```

现在，让我们看看扩展`Thread`类的类。请注意，`Thread1`在使用`lock2`之前使用`lock1`，而`Thread2`在使用`lock1`之前使用`lock2`：

```java
public class ThreadLock1 extends Thread {
    private final Object lock1;
    private final Object lock2;
    public ThreadLock1(Object lock1, Object lock2) {
       this.lock1 = lock1;
       this.lock2 = lock2;
    }
```

在这个`run`方法中，我们有一个使用`lock1`的同步块，然后是一个嵌套的同步块，使用`lock2`：

```java
    @Override
    public void run() {
        synchronized (lock1) {
            System.out.printf(
                      "Thread 1: Holding lock 1%n");
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
            }
            System.out.printf(
                       "Thread 1: Waiting for lock 2%n");
            synchronized (lock2) {
                System.out.printf(
                        "Thread 1: Holding lock 1 & 2%n");
            }
        }
    }
}
public class ThreadLock2 extends Thread {
    private final Object lock1;
    private final Object lock2;
    public ThreadLock2(Object lock1, Object lock2) {
       this.lock1 = lock1;
       this.lock2 = lock2;
    }
```

在这个`run`方法中，我们有一个使用`lock2`的同步块，然后是一个嵌套的同步块，使用`lock1`：

```java
    @Override
    public void run() {
        synchronized (lock2) {
            System.out.printf(
                       "Thread 2: Holding lock 2%n");
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
            }
            System.out.printf(
                     "Thread 2: Waiting for lock 1%n");
            synchronized (lock1) {
                System.out.printf(
                      "Thread 2: Holding lock 1 & 2%n");
            }
        }
    }
}
```

当我们运行此代码时，输出将如下所示：

```java
Thread 1: Holding lock 1
Thread 2: Holding lock 2
Thread 1: Waiting for lock 2
Thread 2: Waiting for lock 1
```

现在这个程序已经死锁，两个线程都在等待另一个线程完成。如果我们改变我们使用的锁的顺序，使得两个都先使用`lock1`然后使用`lock2`，我们将得到以下结果：

```java
Thread 1: Holding lock 1
Thread 1: Waiting for lock 2
Thread 1: Holding lock 1 & 2
Thread 2: Holding lock 2
Thread 2: Waiting for lock 1
Thread 2: Holding lock 1 & 2
```

死锁条件已解决。死锁很少这么明显，你可能甚至没有意识到正在发生死锁。你需要线程转储来确定你的代码中是否存在死锁。

关于这个话题的最后一个要点——与其使用`Object`类的一个实例作为锁，你可以使用`Lock`类。语法略有不同，你可以询问一个`Lock`对象它是否正在被使用。以下代码片段显示了它将是什么样子，但它不能解决死锁。

在`main`类中，锁将使用实现该接口的`ReentrantLock`类的`Lock`接口：

```java
    public final Lock lock1 = new ReentrantLock();
    public final Lock lock2 = new ReentrantLock();
```

在此代码中，我们通过构造函数将`Lock`对象传递给类：

```java
public class ThreadLock1a extends Thread {
    private final Lock lock1;
    private final Lock lock2;
    public ThreadLock1a(Lock lock1, Lock lock2) {
       this.lock1 = lock1;
       this.lock2 = lock2;
    }
    @Override
    public void run() {
```

注意，我们没有使用同步块，而是调用`lock1`上的`lock`，当关键部分完成时，我们在`lock1`上发出`unlock`：

```java
        lock1.lock();
        System.out.printf("Thread 1a: Holding lock 1%n");
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
        }
        System.out.printf(
              "Thread 1a: Waiting for lock 2%n");
        lock2.lock();
        System.out.printf("Thread 1a: Holding lock 1 & 2");
        lock2.unlock();
        lock1.unlock();
    }
}
```

使用`lock`对象而不是同步块的一个优点是，`unlock`调用不需要在同一类中。

多线程是 Java 的一个强大特性，你应该在适当的时候使用它。我们所看到的是基于从 Java 1.0 版本中可用的原生线程。最近，引入了一种新的线程类型。让我们看看它。

# 创建新的虚拟线程

如前一小节开头所指出的，原生 Java 线程是由 JVM 通过直接与操作系统的线程库合作来管理的。原生线程和操作系统线程之间存在一对一的关系。这种新的方法被称为**虚拟线程**。虽然原生线程是由 JVM 与操作系统合作管理的，但虚拟线程完全由 JVM 管理。操作系统线程仍然被使用，但使这种方法变得显著的是，虚拟线程可以共享操作系统线程，并且不再是点对点的关系。

虚拟线程运行速度并不快，可能会遭受竞态和死锁条件。虚拟线程的特殊之处在于，您可以启动的线程数量可能达到数百万。我们使用虚拟线程的方式与本地线程没有太大区别。以下代码片段显示了我们在前面的示例中看到的`perform`方法创建虚拟线程。线程类没有改变，这使得使用虚拟线程而不是本地线程变得非常容易：

```java
     public void perform() {
        for (int i = 0; i < 5; ++i) {
```

这里，我们正在创建一个虚拟线程并启动它：

```java
            Thread.ofVirtual().name("Thread # " + i).
               start(new VirtualThreadRunnableInterface());
        }
        try {
            Thread.sleep(500);
        } catch (InterruptedException ex) {
            ex.printStackTrace();
        }
    }
```

我必须尴尬地承认，我花了 3 天时间才让这段代码工作。为什么？我忽略了虚拟线程的一个重要特性——它们是守护线程。尝试使它们非守护线程没有任何效果。发生在我身上的情况是，程序会在任何输出出现之前结束，或者只出现了一些输出，但没有出现所有预期的线程输出。当`perform`结束并返回到`main()`方法时，主本地线程结束。当这种情况发生时，所有守护线程都会结束。我的电脑执行了`perform`，返回到`main`，并在单个虚拟线程能够显示其输出之前结束。

您可以看到我用来使这段代码工作的解决方案。我调用了`Thread.sleep()`。这会使当前线程休眠指定的时间长度。在这种情况下，500 毫秒足够虚拟线程完成所有任务，而主线程结束。

最后，您不能更改虚拟线程的优先级。它们都运行在`NORM_PRIORITY`。

# 摘要

如本章开头所述，Java 对线程的原生支持是其受欢迎的原因之一。在本章中，我们看到了如何通过扩展`Thread`类和实现`Runnable`或`Callable`接口来创建线程。我们看到了`ExecutorService`如何允许我们池化线程。我们通过查看一个特定问题来结束本章，即两个或多个线程竞争访问共享资源，称为竞态条件，并看到我们如何通过应用同步来解决这个问题。

线程领域即将发生一些变化。在撰写本文时，Project Loom 引入了由 JVM 独家管理的线程以及一个并发框架。一些功能处于预览阶段，而其他功能处于孵化阶段。在几年内，这些新型线程才会变得普遍。我建议关注这个项目的开发。

在我们下一章中，我们将探讨 Java 开发中最常用的设计模式。这些模式将为我们提供组织代码的既定方法。

# 进一步阅读

+   *创建和启动 Java 线程*：[`jenkov.com/tutorials/java-concurrency/creating-and-starting-threads.html`](https://jenkov.com/tutorials/java-concurrency/creating-and-starting-threads.html)

+   *Java 中的线程池简介*: [`www.baeldung.com/thread-pool-java-and-guava`](https://www.baeldung.com/thread-pool-java-and-guava)

+   *Java 多线程中的死锁*: [`www.geeksforgeeks.org/deadlock-in-java-multithreading/`](https://www.geeksforgeeks.org/deadlock-in-java-multithreading/)

+   这里有一本 2006 年的书，仍然是 Java 中线程方面最优秀的参考资料之一：*《Java 并发实践，第 1 版》*，由 Brian Goetz 与 Tim Peierls、Joshua Bloch、Joseph Bowbeer、David Holmes 和 Doug Lea 合著（ISBN-13: 978-0321349606）

+   *JEP 425: 虚拟* *线程*: [`openjdk.org/jeps/425`](https://openjdk.org/jeps/425)
