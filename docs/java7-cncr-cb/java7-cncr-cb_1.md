# 第一章：线程管理

在本章中，我们将涵盖：

+   创建和运行线程

+   获取和设置线程信息

+   中断线程

+   控制线程的中断

+   休眠和恢复线程

+   等待线程的最终化

+   创建和运行守护线程

+   在线程中处理不受控制的异常

+   使用本地线程变量

+   将线程分组

+   在一组线程中处理不受控制的异常

+   通过工厂创建线程

# 介绍

在计算机世界中，当我们谈论**并发**时，我们谈论的是在计算机中同时运行的一系列任务。如果计算机有多个处理器或多核处理器，这种同时性可以是真实的，或者如果计算机只有一个核心处理器，这种同时性可以是表面的。

所有现代操作系统都允许执行并发任务。您可以在读取电子邮件的同时听音乐和在网页上阅读新闻。我们可以说这种并发是**进程级**的并发。但在一个进程内部，我们也可以有各种同时进行的任务。在进程内部运行的并发任务称为**线程**。

与并发相关的另一个概念是**并行**。并发概念有不同的定义和关系。一些作者在你在单核处理器上使用多个线程执行应用程序时谈论并发，因此同时你可以看到你的程序执行是表面的。此外，当您在多核处理器或具有多个处理器的计算机上使用多个线程执行应用程序时，您也可以谈论并行。其他作者在应用程序的线程在没有预定义顺序的情况下执行时谈论并发，并在使用各种线程简化问题解决方案时谈论并行，其中所有这些线程都以有序的方式执行。

本章介绍了一些示例，展示了如何使用 Java 7 API 执行线程的基本操作。您将看到如何在 Java 程序中创建和运行线程，如何控制它们的执行，以及如何将一些线程分组以将它们作为一个单元进行操作。

# 创建和运行线程

在这个示例中，我们将学习如何在 Java 应用程序中创建和运行线程。与 Java 语言中的每个元素一样，线程都是**对象**。在 Java 中创建线程有两种方式：

+   扩展`Thread`类并重写`run()`方法

+   构建一个实现`Runnable`接口的类，然后创建一个`Thread`类的对象，将`Runnable`对象作为参数传递

在这个示例中，我们将使用第二种方法创建一个简单的程序，创建并运行 10 个线程。每个线程计算并打印 1 到 10 之间的数字的乘法表。

## 准备工作

本示例已使用 Eclipse IDE 实现。如果您使用 Eclipse 或其他 IDE，如 NetBeans，请打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`Calculator`的类，实现`Runnable`接口。

```java
public class Calculator implements Runnable {
```

1.  声明一个名为`number`的`private`整数属性，并实现初始化其值的类的构造函数。

```java
  private int number;
  public Calculator(int number) {
    this.number=number;
  }
```

1.  实现`run()`方法。这个方法将执行我们正在创建的线程的指令，因此这个方法将计算数字的乘法表。

```java
  @Override
  public void run() {
    for (int i=1; i<=10; i++){
      System.out.printf("%s: %d * %d = %d\n",Thread.currentThread().getName(),number,i,i*number);
    }
  }
```

1.  现在，实现应用程序的主类。创建一个名为`Main`的类，其中包含`main()`方法。

```java
public class Main {
  public static void main(String[] args) {
```

1.  在`main()`方法中，创建一个有 10 次迭代的`for`循环。在循环内，创建一个`Calculator`类的对象，一个`Thread`类的对象，将`Calculator`对象作为参数传递，并调用线程对象的`start()`方法。

```java
    for (int i=1; i<=10; i++){
      Calculator calculator=new Calculator(i);
      Thread thread=new Thread(calculator);
      thread.start();
    }
```

1.  运行程序，看看不同的线程如何并行工作。

## 它是如何工作的...

程序的输出部分如下截图所示。我们可以看到，我们创建的所有线程都并行运行以完成它们的工作，如下截图所示：

![工作原理...](img/7881_01_01.jpg)

每个 Java 程序至少有一个执行线程。运行程序时，JVM 会运行调用程序的`main()`方法的执行线程。

当我们调用`Thread`对象的`start()`方法时，我们正在创建另一个执行线程。我们的程序将有多少执行线程，就会调用多少次`start()`方法。

Java 程序在所有线程完成时结束（更具体地说，当所有非守护线程完成时）。如果初始线程（执行`main()`方法的线程）结束，其余线程将继续执行直到完成。如果其中一个线程使用`System.exit()`指令来结束程序的执行，所有线程都将结束执行。

创建`Thread`类的对象并不会创建新的执行线程。调用实现`Runnable`接口的类的`run()`方法也不会创建新的执行线程。只有调用`start()`方法才会创建新的执行线程。

## 还有更多...

正如我们在本示例的介绍中提到的，还有另一种创建新执行线程的方法。您可以实现一个继承`Thread`类并重写这个类的`run()`方法的类。然后，您可以创建这个类的对象并调用`start()`方法来创建一个新的执行线程。

## 另请参阅

+   在第一章的*通过工厂创建线程*示例中，*线程管理*

# 获取和设置线程信息

`Thread`类保存了一些信息属性，可以帮助我们识别线程、了解其状态或控制其优先级。这些属性包括：

+   **ID**：此属性为每个`Thread`存储一个唯一标识符。

+   **名称**：此属性存储`Thread`的名称。

+   **优先级**：此属性存储`Thread`对象的优先级。线程的优先级可以在 1 到 10 之间，其中 1 是最低优先级，10 是最高优先级。不建议更改线程的优先级，但如果需要，可以使用这个选项。

+   **状态**：此属性存储`Thread`的状态。在 Java 中，`Thread`可以处于以下六种状态之一：`new`、`runnable`、`blocked`、`waiting`、`time``waiting`或`terminated`。

在本示例中，我们将开发一个程序，为 10 个线程设置名称和优先级，然后显示它们的状态信息，直到它们完成。这些线程将计算一个数字的乘法表。

## 准备工作

本示例使用 Eclipse IDE 实现。如果您使用 Eclipse 或其他 IDE，如 NetBeans，请打开它并创建一个新的 Java 项目。

## 操作步骤...

按照以下步骤实现示例：

1.  创建一个名为`Calculator`的类，并指定它实现`Runnable`接口。

```java
public class Calculator implements Runnable {
```

1.  声明一个名为`number`的`int`私有属性，并实现初始化该属性的类的构造函数。

```java
  private int number;
  public Calculator(int number) {
    this.number=number;
  }
```

1.  实现`run()`方法。这个方法将执行我们正在创建的线程的指令，因此这个方法将计算并打印一个数字的乘法表。

```java
  @Override
  public void run() {
    for (int i=1; i<=10; i++){
      System.out.printf("%s: %d * %d = %d\n",Thread.currentThread().getName(),number,i,i*number);
    }
  }
```

1.  现在，我们实现这个示例的主类。创建一个名为`Main`的类，并实现`main()`方法。

```java
public class Main {
  public static void main(String[] args) {
```

1.  创建一个包含 10 个`threads`和 10 个`Thread.State`的数组，用于存储我们将要执行的线程及其状态。

```java
    Thread threads[]=new Thread[10];
    Thread.State status[]=new Thread.State[10];
```

1.  创建 10 个`Calculator`类的对象，每个对象都初始化为不同的数字，并创建 10 个`threads`来运行它们。将其中五个的优先级设置为最大值，将其余的优先级设置为最小值。

```java
    for (int i=0; i<10; i++){
      threads[i]=new Thread(new Calculator(i));
      if ((i%2)==0){
        threads[i].setPriority(Thread.MAX_PRIORITY);
      } else {
        threads[i].setPriority(Thread.MIN_PRIORITY);
      }
      threads[i].setName("Thread "+i);
    }
```

1.  创建一个`PrintWriter`对象来写入线程状态的文件。

```java
    try (FileWriter file = new FileWriter(".\\data\\log.txt");
PrintWriter pw = new PrintWriter(file);){
```

1.  在这个文件上写下 10 个“线程”的状态。现在，它变成了`NEW`。

```java
      for (int i=0; i<10; i++){
pw.println("Main : Status of Thread "+i+" : "  +             threads[i].getState());
        status[i]=threads[i].getState();
      }
```

1.  开始执行这 10 个线程。

```java
      for (int i=0; i<10; i++){
        threads[i].start();
      }
```

1.  直到这 10 个线程结束，我们将检查它们的状态。如果我们检测到线程状态的变化，我们就把它们写在文件中。

```java
      boolean finish=false;
      while (!finish) {
        for (int i=0; i<10; i++){
          if (threads[i].getState()!=status[i]) {
            writeThreadInfo(pw, threads[i],status[i]);
            status[i]=threads[i].getState();
          }
        }      
        finish=true;
        for (int i=0; i<10; i++){
finish=finish &&(threads[i].getState()==State.TERMINATED);
        }
      }
```

1.  实现`writeThreadInfo()`方法，该方法写入`Thread`的 ID、名称、优先级、旧状态和新状态。

```java
  private static void writeThreadInfo(PrintWriter pw, Thread thread, State state) {
pw.printf("Main : Id %d - %s\n",thread.getId(),thread.getName());
pw.printf("Main : Priority: %d\n",thread.getPriority());
pw.printf("Main : Old State: %s\n",state);
pw.printf("Main : New State: %s\n",thread.getState());
pw.printf("Main : ************************************\n");
  }
```

1.  运行示例并打开`log.txt`文件，查看这 10 个线程的演变。

## 它是如何工作的...

下面的截图显示了该程序执行过程中`log.txt`文件的一些行。在这个文件中，我们可以看到优先级最高的线程在优先级最低的线程之前结束。我们还可以看到每个线程状态的演变。

![它是如何工作的...](img/7881_01_02.jpg)

在控制台显示的程序是线程计算的乘法表和文件`log.txt`中不同线程状态的演变。通过这种方式，你可以更好地看到线程的演变。

`Thread`类有属性来存储线程的所有信息。JVM 使用线程的优先级来选择在每个时刻使用 CPU 的线程，并根据每个线程的情况更新每个线程的状态。

如果你没有为线程指定名称，JVM 会自动分配一个格式为 Thread-XX 的名称，其中 XX 是一个数字。你不能修改线程的 ID 或状态。`Thread`类没有实现`setId()`和`setStatus()`方法来允许它们的修改。

## 还有更多...

在这个示例中，你学会了如何使用`Thread`对象访问信息属性。但你也可以从`Runnable`接口的实现中访问这些属性。你可以使用`Thread`类的静态方法`currentThread()`来访问运行`Runnable`对象的`Thread`对象。

你必须考虑到，如果你尝试设置一个不在 1 到 10 之间的优先级，`setPriority()`方法可能会抛出`IllegalArgumentException`异常。

## 另请参阅

+   *中断线程*在第一章中的*线程管理*中的示例

# 中断线程

一个具有多个执行线程的 Java 程序只有在所有线程的执行结束时才会结束（更具体地说，当所有非守护线程结束执行或其中一个线程使用`System.exit()`方法时）。有时，你需要结束一个线程，因为你想终止一个程序，或者程序的用户想取消`Thread`对象正在执行的任务。

Java 提供了中断机制来指示线程我们想要结束它。这种机制的一个特点是`Thread`必须检查它是否被中断，它可以决定是否响应最终化请求。`Thread`可以忽略它并继续执行。

在这个示例中，我们将开发一个程序，创建`Thread`，并在 5 秒后使用中断机制强制结束它。

## 准备就绪

本示例使用 Eclipse IDE 实现。如果你使用 Eclipse 或其他 IDE，如 NetBeans，打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`PrimeGenerator`的类，该类扩展了`Thread`类。

```java
public class PrimeGenerator extends Thread{
```

1.  重写`run()`方法，包括一个将无限运行的循环。在这个循环中，我们将处理从 1 开始的连续数字。对于每个数字，我们将计算它是否是一个质数，如果是，我们将把它写入控制台。

```java
  @Override
  public void run() {
    long number=1L;
    while (true) {
      if (isPrime(number)) {
        System.out.printf("Number %d is Prime",number);
      }
```

1.  处理完一个数字后，通过调用`isInterrupted()`方法来检查线程是否被中断。如果这个方法返回`true`，我们就写一条消息并结束线程的执行。

```java
      if (isInterrupted()) {
        System.out.printf("The Prime Generator has been Interrupted");
        return;
      }
      number++;
    }
  }
```

1.  实现`isPrime()`方法。它返回一个`boolean`值，指示接收的参数是否为质数（`true`）还是不是（`false`）。

```java
  private boolean isPrime(long number) {
    if (number <=2) {
      return true;
    }
    for (long i=2; i<number; i++){
      if ((number % i)==0) {
        return false;
      }
    }
    return true;
  }
```

1.  现在，通过实现一个名为`Main`的类并实现`main()`方法来实现示例的主类。

```java
public class Main {
  public static void main(String[] args) {
```

1.  创建并启动`PrimeGenerator`类的对象。

```java
    Thread task=new PrimeGenerator();
    task.start();
```

1.  等待 5 秒并中断`PrimeGenerator`线程。

```java
    try {
      Thread.sleep(5000);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
task.interrupt();
```

1.  运行示例并查看结果。

## 它是如何工作的...

以下屏幕截图显示了上一个示例的执行结果。我们可以看到`PrimeGenerator`线程在检测到被中断时写入消息并结束其执行。请参考以下屏幕截图：

![它是如何工作的...](img/7881_01_03.jpg)

`Thread`类有一个属性，用于存储一个`boolean`值，指示线程是否已被中断。当您调用线程的`interrupt()`方法时，您将该属性设置为`true`。`isInterrupted()`方法只返回该属性的值。

## 还有更多...

`Thread`类还有另一个方法来检查`Thread`是否已被中断。它是静态方法`interrupted()`，用于检查当前执行线程是否已被中断。

### 注意

`isInterrupted()`和`interrupted()`方法之间有一个重要的区别。第一个不会改变`interrupted`属性的值，但第二个会将其设置为`false`。由于`interrupted()`方法是一个静态方法，建议使用`isInterrupted()`方法。

如我之前提到的，`Thread`可以忽略其中断，但这不是预期的行为。

# 控制线程的中断

在上一个示例中，您学习了如何中断线程的执行以及如何控制`Thread`对象中的中断。在上一个示例中展示的机制可以用于可以被中断的简单线程。但是，如果线程实现了分为一些方法的复杂算法，或者它具有具有递归调用的方法，我们可以使用更好的机制来控制线程的中断。Java 为此提供了`InterruptedException`异常。当检测到线程中断时，您可以抛出此异常并在`run()`方法中捕获它。

在本示例中，我们将实现一个`Thread`，它在文件夹及其所有子文件夹中查找具有确定名称的文件，以展示如何使用`InterruptedException`异常来控制线程的中断。

## 准备工作

本示例使用 Eclipse IDE 实现。如果您使用 Eclipse 或其他 IDE（如 NetBeans），请打开它并创建一个新的 Java 项目。

## 操作步骤...

按照以下步骤实现示例：

1.  创建一个名为`FileSearch`的类，并指定它实现`Runnable`接口。

```java
public class FileSearch implements Runnable {
```

1.  声明两个`private`属性，一个用于要搜索的文件名，另一个用于初始文件夹。实现类的构造函数，初始化这些属性。

```java
  private String initPath;
  private String fileName;
  public FileSearch(String initPath, String fileName) {
    this.initPath = initPath;
    this.fileName = fileName;
  }
```

1.  实现`FileSearch`类的`run()`方法。它检查属性`fileName`是否为目录，如果是，则调用`processDirectory()`方法。该方法可能会抛出`InterruptedException`异常，因此我们必须捕获它们。

```java
  @Override
  public void run() {
    File file = new File(initPath);
    if (file.isDirectory()) {
      try {
        directoryProcess(file);
      } catch (InterruptedException e) {
        System.out.printf("%s: The search has been interrupted",Thread.currentThread().getName());
      }
    }
  }
```

1.  实现`directoryProcess()`方法。该方法将获取文件夹中的文件和子文件夹并对它们进行处理。对于每个目录，该方法将使用递归调用并将目录作为参数传递。对于每个文件，该方法将调用`fileProcess()`方法。在处理所有文件和文件夹后，该方法检查`Thread`是否已被中断，如果是，则抛出`InterruptedException`异常。

```java
  private void directoryProcess(File file) throws InterruptedException {
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
    if (Thread.interrupted()) {
      throw new InterruptedException();
    }
  }
```

1.  实现`processFile()`方法。此方法将比较其正在处理的文件的名称与我们正在搜索的名称。如果名称相等，我们将在控制台中写入一条消息。在此比较之后，`Thread`将检查它是否已被中断，如果是，则抛出`InterruptedException`异常。

```java
  private void fileProcess(File file) throws InterruptedException {
    if (file.getName().equals(fileName)) {
      System.out.printf("%s : %s\n",Thread.currentThread().getName() ,file.getAbsolutePath());
    }
    if (Thread.interrupted()) {
      throw new InterruptedException();
    }
  }
```

1.  现在，让我们实现示例的主类。实现一个名为`Main`的类，其中包含`main()`方法。

```java
public class Main {
  public static void main(String[] args) {
```

1.  创建并初始化`FileSearch`类的对象和`Thread`以执行其任务。然后，开始执行`Thread`。

```java
    FileSearch searcher=new FileSearch("C:\\","autoexec.bat");
    Thread thread=new Thread(searcher);
    thread.start();
```

1.  等待 10 秒并中断`Thread`。

```java
    try {
      TimeUnit.SECONDS.sleep(10);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    thread.interrupt();
  }
```

1.  运行示例并查看结果。

## 工作原理...

以下屏幕截图显示了此示例执行的结果。您可以看到`FileSearch`对象在检测到已被中断时结束其执行。请参考以下屏幕截图：

![工作原理...](img/7881_01_04.jpg)

在此示例中，我们使用 Java 异常来控制`Thread`的中断。运行示例时，程序开始通过检查文件夹来检查它们是否有文件。例如，如果您进入文件夹`\b\c\d`，程序将对`processDirectory()`方法进行三次递归调用。当它检测到已被中断时，它会抛出`InterruptedException`异常，并在`run()`方法中继续执行，无论已经进行了多少次递归调用。

## 还有更多...

`InterruptedException`异常由一些与并发 API 相关的 Java 方法抛出，例如`sleep()`。

## 另请参阅

+   第一章中的*中断线程示例*，*线程管理*

# 休眠和恢复线程

有时，您可能会对在一定时间内中断`Thread`的执行感兴趣。例如，程序中的一个线程每分钟检查一次传感器状态。其余时间，线程什么也不做。在此期间，线程不使用计算机的任何资源。此时间结束后，当 JVM 选择执行时，线程将准备好继续执行。您可以使用`Thread`类的`sleep()`方法来实现这一目的。该方法接收一个整数作为参数，表示线程暂停执行的毫秒数。当休眠时间结束时，线程在`sleep()`方法调用后的指令中继续执行，当 JVM 分配给它们 CPU 时间时。

另一种可能性是使用`TimeUnit`枚举的元素的`sleep()`方法。此方法使用`Thread`类的`sleep()`方法将当前线程置于休眠状态，但它以表示的单位接收参数，并将其转换为毫秒。

在本示例中，我们将开发一个程序，使用`sleep()`方法每秒写入实际日期。

## 准备就绪

本示例已使用 Eclipse IDE 实现。如果您使用 Eclipse 或其他 IDE，如 NetBeans，请打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`FileClock`的类，并指定它实现`Runnable`接口。

```java
public class FileClock implements Runnable {
```

1.  实现`run()`方法。

```java
  @Override
  public void run() {
```

1.  编写一个具有 10 次迭代的循环。在每次迭代中，创建一个`Date`对象，将其写入文件，并调用`TimeUnit`类的`SECONDS`属性的`sleep()`方法，以暂停线程的执行一秒钟。使用此值，线程将大约休眠一秒钟。由于`sleep()`方法可能会抛出`InterruptedException`异常，因此我们必须包含捕获它的代码。在线程被中断时，包括释放或关闭线程正在使用的资源的代码是一个良好的实践。

```java
    for (int i = 0; i < 10; i++) {
      System.out.printf("%s\n", new Date());
      try {
        TimeUnit.SECONDS.sleep(1);
      } catch (InterruptedException e) {
        System.out.printf("The FileClock has been interrupted");
      }
    }
  }
```

1.  我们已经实现了线程。现在，让我们实现示例的主类。创建一个名为`FileMain`的类，其中包含`main()`方法。

```java
public class FileMain {
  public static void main(String[] args) {
```

1.  创建一个`FileClock`类的对象和一个线程来执行它。然后，开始执行`Thread`。

```java
    FileClock clock=new FileClock();
    Thread thread=new Thread(clock);
    thread.start();
```

1.  在主`Thread`中调用`TimeUnit`类的 SECONDS 属性的`sleep()`方法，等待 5 秒。

```java
    try {
      TimeUnit.SECONDS.sleep(5);
    } catch (InterruptedException e) {
      e.printStackTrace();
    };
```

1.  中断`FileClock`线程。

```java
    thread.interrupt();
```

1.  运行这个例子并查看结果。

## 它是如何工作的...

当你运行这个例子时，你可以看到程序每秒写入一个`Date`对象，然后显示`FileClock`线程已被中断的消息。

当你调用`sleep()`方法时，`Thread`离开 CPU 并停止执行一段时间。在这段时间内，它不会消耗 CPU 时间，所以 CPU 可以执行其他任务。

当`Thread`正在睡眠并被中断时，该方法会立即抛出`InterruptedException`异常，而不会等到睡眠时间结束。

## 还有更多...

Java 并发 API 还有另一个方法，可以让`Thread`对象离开 CPU。这就是`yield()`方法，它告诉 JVM`Thread`对象可以离开 CPU 去做其他任务。JVM 不能保证会遵守这个请求。通常，它只用于调试目的。

# 等待线程的最终化

在某些情况下，我们需要等待线程的最终化。例如，我们可能有一个程序，在继续执行之前需要开始初始化所需的资源。我们可以将初始化任务作为线程运行，并在继续程序的其余部分之前等待其最终化。

为此，我们可以使用`Thread`类的`join()`方法。当我们使用一个线程对象调用这个方法时，它会暂停调用线程的执行，直到被调用的对象完成执行。

在这个示例中，我们将学习如何在初始化示例中使用这个方法。

## 准备工作

这个示例是使用 Eclipse IDE 实现的。如果你使用 Eclipse 或其他 IDE，比如 NetBeans，打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`DataSourcesLoader`的类，并指定它实现`Runnable`接口。

```java
public class DataSourcesLoader implements Runnable {
```

1.  实现`run()`方法。它写入一个消息表示它开始执行，睡眠 4 秒，然后写入另一个消息表示它结束执行。

```java
  @Override
  public void run() {
    System.out.printf("Beginning data sources loading: %s\n",new Date());
    try {
      TimeUnit.SECONDS.sleep(4);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    System.out.printf("Data sources loading has finished: %s\n",new Date());
  }
```

1.  创建一个名为`NetworkConnectionsLoader`的类，并指定它实现`Runnable`接口。实现`run()`方法。它将与`DataSourcesLoader`类的`run()`方法相同，但这将睡眠 6 秒。

1.  现在，创建一个包含`main()`方法的`Main`类。

```java
public class Main {
  public static void main(String[] args) {
```

1.  创建一个`DataSourcesLoader`类的对象和一个`Thread`来运行它。

```java
    DataSourcesLoader dsLoader = new DataSourcesLoader();
    Thread thread1 = new Thread(dsLoader,"DataSourceThread");
```

1.  创建一个`NetworkConnectionsLoader`类的对象和一个`Thread`来运行它。

```java
    NetworkConnectionsLoader ncLoader = new NetworkConnectionsLoader();
    Thread thread2 = new Thread(ncLoader,"NetworkConnectionLoader");
```

1.  调用两个`Thread`对象的`start()`方法。

```java
    thread1.start();
    thread2.start(); 
```

1.  等待使用`join()`方法来完成两个线程的最终化。这个方法可能会抛出`InterruptedException`异常，所以我们必须包含捕获它的代码。

```java
    try {
      thread1.join();
      thread2.join();
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
```

1.  写一个消息表示程序结束。

```java
    System.out.printf("Main: Configuration has been loaded: %s\n",new Date());
```

1.  运行程序并查看结果。

## 它是如何工作的...

当你运行这个程序时，你可以看到两个`Thread`对象开始执行。首先，`DataSourcesLoader`线程完成执行。然后，`NetworkConnectionsLoader`类完成执行，此时，主`Thread`对象继续执行并写入最终消息。

## 还有更多...

Java 提供了`join()`方法的另外两种形式：

+   join (long milliseconds)

+   join (long milliseconds, long nanos)

在`join()`方法的第一个版本中，调用线程不是无限期地等待被调用的线程的最终化，而是等待方法参数指定的毫秒数。例如，如果对象`thread1`有代码`thread2.join(1000)`，线程`thread1`会暂停执行，直到以下两种情况之一为真：

+   `thread2`完成了它的执行

+   已经过去了 1000 毫秒

当这两个条件中的一个为真时，`join()`方法返回。

`join()`方法的第二个版本与第一个版本类似，但接收毫秒数和纳秒数作为参数。

# 创建和运行守护线程

Java 有一种特殊类型的线程称为**守护**线程。这种类型的线程具有非常低的优先级，通常只有在程序中没有其他线程运行时才会执行。当守护线程是程序中唯一运行的线程时，JVM 会结束程序并完成这些线程。

具有这些特性，守护线程通常用作运行在同一程序中的普通（也称为用户）线程的服务提供者。它们通常有一个无限循环，等待服务请求或执行线程的任务。它们不能执行重要的工作，因为我们不知道它们何时会有 CPU 时间，并且如果没有其他线程运行，它们随时可以结束。这种类型线程的典型例子是 Java 垃圾收集器。

在这个示例中，我们将学习如何创建一个守护线程，开发一个包含两个线程的示例；一个用户线程在队列中写入事件，一个守护线程清理队列，删除超过 10 秒前生成的事件。

## 准备工作

这个示例已经使用 Eclipse IDE 实现。如果你使用 Eclipse 或其他 IDE，比如 NetBeans，打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  创建`Event`类。这个类只存储我们的程序将使用的事件的信息。声明两个私有属性，一个叫做`date`，类型为`java.util.Date`，另一个叫做`event`，类型为`String`。生成方法来写入和读取它们的值。

1.  创建`WriterTask`类并指定它实现`Runnable`接口。

```java
public class WriterTask implements Runnable {
```

1.  声明存储事件的队列并实现类的构造函数，初始化这个队列。

```java
private Deque<Event> deque;
  public WriterTask (Deque<Event> deque){
    this.deque=deque;
  }
```

1.  实现这个任务的`run()`方法。这个方法将有一个循环，循环 100 次。在每次迭代中，我们创建一个新的`Event`，将其保存在队列中，并休眠一秒。

```java
  @Override
  public void run() {
    for (int i=1; i<100; i++) {
      Event event=new Event();
      event.setDate(new Date());
      event.setEvent(String.format("The thread %s has generated an event",Thread.currentThread().getId()));
      deque.addFirst(event);
      try {
        TimeUnit.SECONDS.sleep(1);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
  }
```

1.  创建`CleanerTask`类并指定它扩展`Thread`类。

```java
public class CleanerTask extends Thread {
```

1.  声明存储事件的队列并实现类的构造函数，初始化这个队列。在构造函数中，使用`setDaemon()`方法将这个`Thread`标记为守护线程。

```java
  private Deque<Event> deque;
  public CleanerTask(Deque<Event> deque) {
    this.deque = deque;
    setDaemon(true);
  }
```

1.  实现`run()`方法。它有一个无限循环，获取实际日期并调用`clean()`方法。

```java
  @Override
  public void run() {
    while (true) {
      Date date = new Date();
      clean(date);
    }
  }
```

1.  实现`clean()`方法。获取最后一个事件，如果它是在 10 秒前创建的，就删除它并检查下一个事件。如果删除了一个事件，就写入事件的消息和队列的新大小，这样你就可以看到它的演变。

```java
  private void clean(Date date) {
    long difference;
    boolean delete;

    if (deque.size()==0) {
      return;
    }
    delete=false;
    do {
      Event e = deque.getLast();
      difference = date.getTime() - e.getDate().getTime();
      if (difference > 10000) {
        System.out.printf("Cleaner: %s\n",e.getEvent());
        deque.removeLast();
        delete=true;
      }  
    } while (difference > 10000);
    if (delete){
      System.out.printf("Cleaner: Size of the queue: %d\n",deque.size());
    }
  }
```

1.  现在，实现主类。创建一个名为`Main`的类，其中包含一个`main()`方法。

```java
public class Main {
  public static void main(String[] args) {
```

1.  使用`Deque`类创建队列来存储事件。

```java
    Deque<Event> deque=new ArrayDeque<Event>();
```

1.  创建并启动三个`WriterTask`线程和一个`CleanerTask`。

```java
    WriterTask writer=new WriterTask(deque);
    for (int i=0; i<3; i++){
      Thread thread=new Thread(writer);
      thread.start();
    }
    CleanerTask cleaner=new CleanerTask(deque);
    cleaner.start();
```

1.  运行程序并查看结果。

## 工作原理...

如果分析程序的一次执行输出，可以看到队列开始增长，直到有 30 个事件，然后在执行结束之前，它的大小将在 27 和 30 个事件之间变化。

程序以三个`WriterTask`线程开始。每个`Thread`写入一个事件并休眠一秒。在第一个 10 秒之后，我们在队列中有 30 个线程。在这 10 秒内，`CleanerTasks`一直在执行，而三个`WriterTask`线程在休眠，但它没有删除任何事件，因为它们都是在不到 10 秒前生成的。在执行的其余时间里，`CleanerTask`每秒删除三个事件，而三个`WriterTask`线程写入另外三个事件，所以队列的大小在 27 和 30 个事件之间变化。

您可以调整`WriterTask`线程睡眠的时间。如果使用较小的值，您会发现`CleanerTask`的 CPU 时间较少，并且队列的大小会增加，因为`CleanerTask`不会删除任何事件。

## 还有更多...

在调用`start()`方法之前，您只能调用`setDaemon()`方法。一旦线程正在运行，就无法修改其守护进程状态。

您可以使用`isDaemon()`方法来检查线程是否是守护线程（方法返回`true`）还是用户线程（方法返回`false）。

# 处理线程中的未受控异常

Java 中有两种异常：

+   **已检查的异常**：这些异常必须在方法的`throws`子句中指定或在其中捕获。例如，`IOException`或`ClassNotFoundException`。

+   **未检查的异常**：这些异常不必指定或捕获。例如，`NumberFormatException`。

当在`Thread`对象的`run()`方法中抛出已检查的异常时，我们必须捕获和处理它们，因为`run()`方法不接受`throws`子句。当在`Thread`对象的`run()`方法中抛出未检查的异常时，默认行为是在控制台中写入堆栈跟踪并退出程序。

幸运的是，Java 为我们提供了一种机制来捕获和处理`Thread`对象中抛出的未检查异常，以避免程序结束。

在这个示例中，我们将使用一个示例来学习这个机制。

## 准备工作

这个示例使用 Eclipse IDE 实现。如果您使用 Eclipse 或其他 IDE，如 NetBeans，打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  首先，我们必须实现一个类来处理未检查的异常。这个类必须实现`UncaughtExceptionHandler`接口，并实现该接口中声明的`uncaughtException()`方法。在我们的情况下，将这个类命名为`ExceptionHandler`，并使该方法写入有关抛出异常的`Exception`和`Thread`的信息。以下是代码：

```java
public class ExceptionHandler implements UncaughtExceptionHandler {
  public void uncaughtException(Thread t, Throwable e) {
    System.out.printf("An exception has been captured\n");
    System.out.printf("Thread: %s\n",t.getId());
    System.out.printf("Exception: %s: %s\n",e.getClass().getName(),e.getMessage());
    System.out.printf("Stack Trace: \n");
    e.printStackTrace(System.out);
    System.out.printf("Thread status: %s\n",t.getState());
  }
}
```

1.  现在，实现一个抛出未检查异常的类。将这个类命名为`Task`，指定它实现`Runnable`接口，实现`run()`方法，并强制异常，例如，尝试将`string`值转换为`int`值。

```java
public class Task implements Runnable {
  @Override
  public void run() {
    int numero=Integer.parseInt("TTT");
  }
}
```

1.  现在，实现示例的主类。使用`main()`方法实现一个名为`Main`的类。

```java
public class Main {
  public static void main(String[] args) {
```

1.  创建一个`Task`对象和`Thread`来运行它。使用`setUncaughtExceptionHandler()`方法设置未检查的异常处理程序，并开始执行`Thread`。

```java
    Task task=new Task();
    Thread thread=new Thread(task);
    thread.setUncaughtExceptionHandler(new ExceptionHandler());
    thread.start();
    }
}
```

1.  运行示例并查看结果。

## 它是如何工作的...

在下面的屏幕截图中，您可以看到示例执行的结果。异常被抛出并被处理程序捕获，该处理程序在控制台中写入有关抛出异常的`Exception`和`Thread`的信息。请参考以下屏幕截图：

![它是如何工作的...](img/7881_01_05.jpg)

当线程中抛出异常并且未被捕获（必须是未检查的异常）时，JVM 会检查线程是否有相应方法设置的未捕获异常处理程序。如果有，JVM 将使用`Thread`对象和`Exception`作为参数调用此方法。

如果线程没有未捕获的异常处理程序，JVM 会在控制台中打印堆栈跟踪并退出程序。

## 还有更多...

`Thread`类还有另一个与未捕获异常处理相关的方法。这是静态方法`setDefaultUncaughtExceptionHandler()`，它为应用程序中的所有`Thread`对象建立异常处理程序。

当在`Thread`中抛出未捕获的异常时，JVM 会寻找此异常的三个可能处理程序。

首先，查找`Thread`对象的未捕获异常处理程序，就像我们在这个示例中学到的那样。如果这个处理程序不存在，那么 JVM 将查找`Thread`对象的`ThreadGroup`的未捕获异常处理程序，就像在*在一组线程中处理不受控制的异常*示例中解释的那样。如果这个方法不存在，JVM 将查找默认的未捕获异常处理程序，就像我们在这个示例中学到的那样。

如果没有处理程序退出，JVM 会在控制台中写入异常的堆栈跟踪，并退出程序。

## 另请参阅

+   第一章中的*在一组线程中处理不受控制的异常*示例，*线程管理*

# 使用本地线程变量

并发应用程序中最关键的一个方面是共享数据。这在那些扩展了`Thread`类或实现了`Runnable`接口的对象中尤为重要。

如果你创建了一个实现了`Runnable`接口的类的对象，然后使用相同的`Runnable`对象启动各种`Thread`对象，所有线程都共享相同的属性。这意味着，如果你在一个线程中改变了一个属性，所有线程都会受到这个改变的影响。

有时，你可能会对一个属性感兴趣，这个属性不会在运行相同对象的所有线程之间共享。Java 并发 API 提供了一个称为线程本地变量的清晰机制，性能非常好。

在这个示例中，我们将开发一个程序，其中包含第一段中暴露的问题，以及使用线程本地变量机制解决这个问题的另一个程序。

## 准备工作

这个示例已经使用 Eclipse IDE 实现。如果你使用 Eclipse 或其他 IDE，比如 NetBeans，打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  首先，我们将实现一个程序，其中包含先前暴露的问题。创建一个名为`UnsafeTask`的类，并指定它实现了`Runnable`接口。声明一个`private``java.util.Date`属性。

```java
public class UnsafeTask implements Runnable{
  private Date startDate;
```

1.  实现`UnsafeTask`对象的`run()`方法。这个方法将初始化`startDate`属性，将它的值写入控制台，休眠一段随机时间，然后再次写入`startDate`属性的值。

```java
  @Override
  public void run() {
    startDate=new Date();
    System.out.printf("Starting Thread: %s : %s\n",Thread.currentThread().getId(),startDate);
    try {
      TimeUnit.SECONDS.sleep( (int)Math.rint(Math.random()*10));
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    System.out.printf("Thread Finished: %s : %s\n",Thread.currentThread().getId(),startDate);
  }
```

1.  现在，让我们实现这个有问题的应用程序的主类。创建一个名为`Main`的类，其中包含一个`main()`方法。这个方法将创建一个`UnsafeTask`类的对象，并使用该对象启动三个线程，在每个线程之间休眠 2 秒。

```java
public class Core {
  public static void main(String[] args) {
    UnsafeTask task=new UnsafeTask();
    for (int i=0; i<10; i++){
      Thread thread=new Thread(task);
      thread.start();
      try {
        TimeUnit.SECONDS.sleep(2);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
  }
}
```

1.  在下面的截图中，你可以看到这个程序执行的结果。每个`Thread`有不同的开始时间，但当它们完成时，所有的`startDate`属性都有相同的值。![如何做...](img/7881_01_06.jpg)

1.  如前所述，我们将使用线程本地变量机制来解决这个问题。

1.  创建一个名为`SafeTask`的类，并指定它实现了`Runnable`接口。

```java
public class SafeTask implements Runnable {
```

1.  声明一个`ThreadLocal<Date>`类的对象。这个对象将具有一个包含`initialValue()`方法的隐式实现。这个方法将返回实际的日期。

```java
  private static ThreadLocal<Date> startDate= new ThreadLocal<Date>() {
    protected Date initialValue(){
      return new Date();
    }
  };
```

1.  实现`run()`方法。它具有与`UnsafeClass`的`run()`方法相同的功能，但它改变了访问`startDate`属性的方式。

```java
  @Override
  public void run() {
    System.out.printf("Starting Thread: %s : %s\n",Thread.currentThread().getId(),startDate.get());
    try {
      TimeUnit.SECONDS.sleep((int)Math.rint(Math.random()*10));
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    System.out.printf("Thread Finished: %s : %s\n",Thread.currentThread().getId(),startDate.get());
  }
```

1.  这个示例的主类与不安全的示例相同，只是改变了`Runnable`类的名称。

1.  运行示例并分析差异。

## 它是如何工作的...

在下面的截图中，你可以看到安全示例执行的结果。现在，三个`Thread`对象都有自己的`startDate`属性的值。参考下面的截图：

![它是如何工作的...](img/7881_01_07.jpg)

线程本地变量为使用这些变量的每个`Thread`存储一个属性的值。您可以使用`get()`方法读取该值，并使用`set()`方法更改该值。第一次访问线程本地变量的值时，如果它对于调用它的`Thread`对象没有值，则线程本地变量将调用`initialValue()`方法为该`Thread`分配一个值，并返回初始值。

## 还有更多...

线程本地类还提供了`remove()`方法，用于删除调用它的线程的线程本地变量中存储的值。

Java 并发 API 包括`InheritableThreadLocal`类，它提供了从线程创建的线程继承值的功能。如果线程 A 在线程本地变量中有一个值，并且它创建另一个线程 B，则线程 B 将在线程本地变量中具有与线程 A 相同的值。您可以重写`childValue()`方法，该方法用于初始化线程本地变量中子线程的值。它将父线程在线程本地变量中的值作为参数。

# 将线程分组

Java 并发 API 提供的一个有趣功能是能够对线程进行分组。这使我们能够将组中的线程视为单个单位，并提供对属于组的`Thread`对象的访问，以对它们进行操作。例如，如果有一些线程执行相同的任务，并且您想要控制它们，无论有多少线程正在运行，每个线程的状态都将通过单个调用中断所有线程。

Java 提供了`ThreadGroup`类来处理线程组。`ThreadGroup`对象可以由`Thread`对象和另一个`ThreadGroup`对象组成，生成线程的树形结构。

在这个示例中，我们将学习如何使用`ThreadGroup`对象开发一个简单的示例。我们将有 10 个线程在随机时间段内休眠（例如模拟搜索），当其中一个完成时，我们将中断其余的线程。

## 准备工作

这个示例使用 Eclipse IDE 实现。如果您使用 Eclipse 或其他 IDE，如 NetBeans，打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  首先，创建一个名为`Result`的类。它将存储首先完成的`Thread`的名称。声明一个名为`name`的`private`字符串属性和用于读取和设置该值的方法。

1.  创建一个名为`SearchTask`的类，并指定它实现`Runnable`接口。

```java
public class SearchTask implements Runnable {
```

1.  声明`Result`类的`private`属性并实现该类的构造函数以初始化此属性。

```java
  private Result result;
  public SearchTask(Result result) {
    this.result=result;
  }
```

1.  实现`run()`方法。它将调用`doTask()`方法并等待其完成或出现`InterruptedException`异常。该方法将写入消息以指示此`Thread`的开始、结束或中断。

```java
  @Override
  public void run() {
    String name=Thread.currentThread().getName();
    System.out.printf("Thread %s: Start\n",name);
    try {
      doTask();
      result.setName(name);
    } catch (InterruptedException e) {
      System.out.printf("Thread %s: Interrupted\n",name);
      return;
    }
    System.out.printf("Thread %s: End\n",name);
  }
```

1.  实现`doTask()`方法。它将创建一个`Random`对象来生成一个随机数，并调用`sleep()`方法来休眠该随机数的时间。

```java
  private void doTask() throws InterruptedException {
    Random random=new Random((new Date()).getTime());
    int value=(int)(random.nextDouble()*100);
    System.out.printf("Thread %s: %d\n",Thread.currentThread().getName(),value);
    TimeUnit.SECONDS.sleep(value);
  }
```

1.  现在，通过创建一个名为`Main`的类并实现`main()`方法来创建示例的主类。

```java
public class Main {
  public static void main(String[] args) {
```

1.  首先，创建一个`ThreadGroup`对象并将其命名为`Searcher`。

```java
    ThreadGroup threadGroup = new ThreadGroup("Searcher");
```

1.  然后，创建一个`SearchTask`对象和一个`Result`对象。

```java
    Result result=new Result();     SearchTask searchTask=new SearchTask(result);
```

1.  现在，使用`SearchTask`对象创建 10 个`Thread`对象。当调用`Thread`类的构造函数时，将其作为`ThreadGroup`对象的第一个参数传递。

```java
    for (int i=0; i<5; i++) {
      Thread thread=new Thread(threadGroup, searchTask);
      thread.start();
      try {
        TimeUnit.SECONDS.sleep(1);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
```

1.  使用`list()`方法写入关于`ThreadGroup`对象的信息。

```java
    System.out.printf("Number of Threads: %d\n",threadGroup.activeCount());
    System.out.printf("Information about the Thread Group\n");
    threadGroup.list();
```

1.  使用`activeCount()`和`enumerate()`方法来了解有多少`Thread`对象与`ThreadGroup`对象相关联，并获取它们的列表。我们可以使用此方法来获取每个`Thread`的状态，例如。

```java
    Thread[] threads=new Thread[threadGroup.activeCount()];
    threadGroup.enumerate(threads);
    for (int i=0; i<threadGroup.activeCount(); i++) {
      System.out.printf("Thread %s: %s\n",threads[i].getName(),threads[i].getState());
    }
```

1.  调用`waitFinish()`方法。我们稍后将实现此方法。它将等待直到`ThreadGroup`对象的一个线程结束。

```java
    waitFinish(threadGroup);
```

1.  使用`interrupt()`方法中断组中其余的线程。

```java
    threadGroup.interrupt();
```

1.  实现`waitFinish()`方法。它将使用`activeCount()`方法来控制其中一个线程的结束。

```java
  private static void waitFinish(ThreadGroup threadGroup) {
    while (threadGroup.activeCount()>9) {
      try {
        TimeUnit.SECONDS.sleep(1);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    }
  }
```

1.  运行示例并查看结果。

## 它是如何工作的...

在下面的屏幕截图中，您可以看到`list()`方法的输出以及当我们写入每个`Thread`对象的状态时生成的输出，如下面的屏幕截图所示：

![它是如何工作的...](img/7881_01_08.jpg)

`ThreadGroup`类存储`Thread`对象和与之关联的其他`ThreadGroup`对象，因此它可以访问它们所有的信息（例如状态）并对其所有成员执行操作（例如中断）。

## 还有更多...

`ThreadGroup`类有更多的方法。查看 API 文档以获得所有这些方法的完整解释。

# 在一组线程中处理不受控制的异常

在每种编程语言中，一个非常重要的方面是提供管理应用程序中错误情况的机制。Java 语言，就像几乎所有现代编程语言一样，实现了基于异常的机制来管理错误情况。它提供了许多类来表示不同的错误。当检测到错误情况时，Java 类会抛出这些异常。您也可以使用这些异常，或者实现自己的异常来管理类中产生的错误。

Java 还提供了一种机制来捕获和处理这些异常。有些异常必须使用方法的`throws`子句捕获或重新抛出。这些异常称为已检查异常。有些异常不必指定或捕获。这些是未检查的异常。

在这个示例中，*控制线程的中断*，你学会了如何使用一个通用方法来处理`Thread`对象中抛出的所有未捕获的异常。

另一种可能性是建立一个方法，捕获`ThreadGroup`类的任何`Thread`抛出的所有未捕获的异常。

在这个示例中，我们将学习使用一个例子来设置这个处理程序。

## 准备工作

这个示例使用 Eclipse IDE 实现。如果你使用 Eclipse 或其他 IDE 如 NetBeans，打开它并创建一个新的 Java 项目。

## 操作步骤...

按照以下步骤实现示例：

1.  首先，我们必须通过创建一个名为`MyThreadGroup`的类来扩展`ThreadGroup`类，该类从`ThreadGroup`类扩展。我们必须声明一个带有一个参数的构造函数，因为`ThreadGroup`类没有没有参数的构造函数。

```java
public class MyThreadGroup extends ThreadGroup {
  public MyThreadGroup(String name) {
    super(name);
  }
```

1.  重写`uncaughtException()`方法。当`ThreadGroup`类的一个线程抛出异常时，将调用此方法。在这种情况下，此方法将在控制台中写入有关异常和抛出异常的`Thread`的信息，并中断`ThreadGroup`类中的其余线程。

```java
  @Override
  public void uncaughtException(Thread t, Throwable e) {
    System.out.printf("The thread %s has thrown an Exception\n",t.getId());
    e.printStackTrace(System.out);
    System.out.printf("Terminating the rest of the Threads\n");
    interrupt();
  }
```

1.  创建一个名为`Task`的类，并指定它实现`Runnable`接口。

```java
public class Task implements Runnable {
```

1.  实现`run()`方法。在这种情况下，我们将引发一个`AritmethicException`异常。为此，我们将在随机数之间除以 1000，直到随机生成器生成零并抛出异常。

```java
  @Override
  public void run() {
    int result;
    Random random=new Random(Thread.currentThread().getId());
    while (true) {
      result=1000/((int)(random.nextDouble()*1000));
      System.out.printf("%s : %f\n",Thread.currentThread().getId(),result);
      if (Thread.currentThread().isInterrupted()) {
        System.out.printf("%d : Interrupted\n",Thread.currentThread().getId());
        return;
      }
    }
  }
```

1.  现在，我们将通过创建一个名为`Main`的类并实现`main()`方法来实现示例的主类。

```java
public class Main {
  public static void main(String[] args) {
```

1.  创建一个`MyThreadGroup`类的对象。

```java
    MyThreadGroup threadGroup=new MyThreadGroup("MyThreadGroup");
```

1.  创建一个`Task`类的对象。

```java
    Task task=new Task();
```

1.  创建两个带有这个`Task`的`Thread`对象并启动它们。

```java
    for (int i=0; i<2; i++){
      Thread t=new Thread(threadGroup,task);
      t.start();
    }
```

1.  运行示例并查看结果。

## 它是如何工作的...

当你运行示例时，你会看到其中一个`Thread`对象抛出了异常，另一个被中断了。

当在`Thread`中抛出未捕获的异常时，JVM 会寻找这个异常的三个可能的处理程序。

首先，查找线程的未捕获异常处理程序，就像在*处理线程中的不受控制的异常*配方中所解释的那样。如果这个处理程序不存在，那么 JVM 会查找线程的`ThreadGroup`类的未捕获异常处理程序，就像我们在这个配方中学到的那样。如果这个方法不存在，JVM 会查找默认的未捕获异常处理程序，就像在*处理线程中的不受控制的异常*配方中所解释的那样。

如果没有处理程序退出，JVM 会在控制台中写入异常的堆栈跟踪，并退出程序。

## 另请参阅

+   第一章中的*处理线程中的不受控制的异常*配方，*线程管理*

# 通过工厂创建线程

工厂模式是面向对象编程世界中最常用的设计模式之一。它是一种创建模式，其目标是开发一个使命将是创建一个或多个类的其他对象的对象。然后，当我们想要创建其中一个类的对象时，我们使用工厂而不是使用`new`运算符。

有了这个工厂，我们可以集中创建对象，并获得一些优势：

+   很容易改变创建的对象的类或创建这些对象的方式。

+   很容易限制为有限资源创建对象。例如，我们只能有一个类型的*n*个对象。

+   很容易生成有关对象创建的统计数据。

Java 提供了一个接口，即`ThreadFactory`接口，用于实现`Thread`对象工厂。Java 并发 API 的一些高级工具使用线程工厂来创建线程。

在这个配方中，我们将学习如何实现`ThreadFactory`接口，以创建具有个性化名称的`Thread`对象，同时保存创建的`Thread`对象的统计数据。

## 准备工作

这个配方的示例是使用 Eclipse IDE 实现的。如果您使用 Eclipse 或其他 IDE，如 NetBeans，打开它并创建一个新的 Java 项目。

## 如何做...

按照以下步骤实现示例：

1.  创建一个名为`MyThreadFactory`的类，并指定它实现`ThreadFactory`接口。

```java
public class MyThreadFactory implements ThreadFactory {
```

1.  声明三个属性：一个名为`counter`的整数，我们将用它来存储创建的`Thread`对象的数量，一个名为`name`的`String`，它是每个创建的`Thread`的基本名称，以及一个名为`stats`的`String`对象列表，用于保存有关创建的`Thread`对象的统计数据。我们还实现了初始化这些属性的类的构造函数。

```java
  private int counter;
  private String name;
  private List<String> stats;

  public MyThreadFactory(String name){
    counter=0;
    this.name=name;
    stats=new ArrayList<String>();
  }
```

1.  实现`newThread()`方法。这个方法将接收一个`Runnable`接口，并为这个`Runnable`接口返回一个`Thread`对象。在我们的例子中，我们生成`Thread`对象的名称，创建新的`Thread`对象，并保存统计数据。

```java
  @Override
  public Thread newThread(Runnable r) {
    Thread t=new Thread(r,name+"-Thread_"+counter);
    counter++;
    stats.add(String.format("Created thread %d with name %s on %s\n",t.getId(),t.getName(),new Date()));
    return t;
  }
```

1.  实现`getStatistics()`方法，返回包含所有创建的`Thread`对象的统计数据的`String`对象。

```java
  public String getStats(){
    StringBuffer buffer=new StringBuffer();
    Iterator<String> it=stats.iterator();

    while (it.hasNext()) {
      buffer.append(it.next());
      buffer.append("\n");
    }

    return buffer.toString();
  }
```

1.  创建一个名为`Task`的类，并指定它实现`Runnable`接口。在这个例子中，这些任务除了睡一秒钟之外什么也不做。

```java
public class Task implements Runnable {
  @Override
  public void run() {
    try {
      TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}
```

1.  创建示例的主类。创建一个名为`Main`的类，并实现`main()`方法。

```java
public class Main {
  public static void main(String[] args) {
```

1.  创建一个`MyThreadFactory`对象和一个`Task`对象。

```java
    MyThreadFactory factory=new MyThreadFactory("MyThreadFactory");
    Task task=new Task();
```

1.  使用`MyThreadFactory`对象创建 10 个`Thread`对象并启动它们。

```java
    Thread thread;
    System.out.printf("Starting the Threads\n");
    for (int i=0; i<10; i++){
      thread=factory.newThread(task);
      thread.start();
    }
```

1.  在控制台中写入线程工厂的统计数据。

```java
    System.out.printf("Factory stats:\n");
    System.out.printf("%s\n",factory.getStats());
```

1.  运行示例并查看结果。

## 它是如何工作的...

`ThreadFactory`接口只有一个名为`newThread`的方法。它接收一个`Runnable`对象作为参数，并返回一个`Thread`对象。当您实现`ThreadFactory`接口时，您必须实现该接口并重写此方法。大多数基本的`ThreadFactory`只有一行。

```java
return new Thread(r);
```

您可以通过添加一些变体来改进这个实现：

+   创建个性化的线程，就像示例中使用特殊格式的名称或甚至创建我们自己的`thread`类一样，该类继承了 Java 的`Thread`类。

+   保存线程创建统计信息，如前面的示例所示

+   限制创建的线程数量

+   验证线程的创建

+   以及您可以想象的任何其他内容

使用工厂设计模式是一种良好的编程实践，但是，如果您实现了`ThreadFactory`接口来集中创建线程，您必须审查代码以确保所有线程都是使用该工厂创建的。

## 参见

+   第七章中的*实现 ThreadFactory 接口生成自定义线程*配方，*自定义并发类*

+   第七章中的*在 Executor 对象中使用我们的 ThreadFactory*配方，*自定义并发类*
