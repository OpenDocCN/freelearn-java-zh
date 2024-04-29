# 并发和多线程编程

并发编程一直是一项困难的任务。它是许多难以解决的问题的根源。在本章中，我们将向您展示不同的整合并发的方式和一些最佳实践，例如不可变性，它有助于创建多线程处理。我们还将讨论一些常用模式的实现，例如分而治之和发布-订阅，使用 Java 提供的构造。我们将涵盖以下内容：

+   使用并发的基本元素——线程

+   不同的同步方法

+   不可变性作为实现并发的手段

+   使用并发集合

+   使用执行器服务执行异步任务

+   使用 fork/join 实现分而治之

+   使用流来实现发布-订阅模式

# 介绍

并发——能够并行执行多个过程——随着大数据分析进入现代应用程序的主流，变得越来越重要。拥有 CPU 或一个 CPU 中的多个核心有助于增加吞吐量，但数据量的增长速度总是超过硬件的进步。此外，即使在多 CPU 系统中，仍然需要结构化代码并考虑资源共享，以充分利用可用的计算能力。

在前几章中，我们演示了如何使用函数接口和并行流使 lambda 成为每个 Java 程序员工具包的一部分，从而实现了并发处理。一个人可以很容易地在最少或没有指导的情况下利用这种功能。

在本章中，我们将描述一些其他（Java 9 之前的）和新的 Java 特性和 API，它们允许更多地控制并发。自 Java 5 以来，高级并发 Java API 一直存在。JDK 增强提案（JEP）266，“更多并发更新”（[`openjdk.java.net/jeps/266`](http://openjdk.java.net/jeps/266)），在 Java 9 中引入到`java.util.concurrent`包中。

“可互操作的发布-订阅框架，对 CompletableFuture API 的增强以及各种其他改进”

但在我们深入了解最新添加的细节之前，让我们回顾一下 Java 中并发编程的基础知识，以及如何使用它们。

Java 有两个执行单元——进程和线程。一个进程通常代表整个 JVM，尽管应用程序可以使用`ProcessBuilder`创建另一个进程。但由于多进程情况超出了本书的范围，我们将专注于第二个执行单元，即线程，它类似于进程，但与其他线程隔离程度较低，执行所需资源较少。

一个进程可以有许多运行的线程，至少有一个称为*主*线程。线程可以共享资源，包括内存和打开的文件，这可以提高效率。但这也带来了更高的意外相互干扰和甚至阻塞执行的风险。这就需要编程技能和对并发技术的理解。这也是我们将在本章讨论的内容。

# 使用并发的基本元素——线程

在本章中，我们将研究`java.lang.Thread`类，并了解它对并发和程序性能的一般作用。

# 准备工作

Java 应用程序以主线程（不包括支持进程的系统线程）开始。然后它可以创建其他线程，并让它们并行运行，通过时间切片共享同一个核心，或者为每个线程分配一个专用的 CPU。这可以使用实现了`Runnable`函数接口的`java.lang.Thread`类来实现，该接口只有一个抽象方法`run()`。

创建新线程有两种方法：创建`Thread`的子类，或者实现`Runnable`接口并将实现类的对象传递给`Thread`构造函数。我们可以通过调用`Thread`类的`start()`方法来调用新线程，该方法又调用了实现的`run()`方法。

然后，我们可以让新线程运行直到完成，或者暂停它然后让它继续。如果需要，我们还可以访问它的属性或中间结果。

# 如何做到...

首先，我们创建一个名为`AThread`的类，它扩展了`Thread`并重写了它的`run()`方法：

```java
class AThread extends Thread {
  int i1,i2;
  AThread(int i1, int i2){
    this.i1 = i1;
    this.i2 = i2;
  }
  public void run() {
    IntStream.range(i1, i2)
             .peek(Chapter07Concurrency::doSomething)
             .forEach(System.out::println);
  }
}
```

在这个例子中，我们希望线程在特定范围内生成一个整数流。然后，我们使用`peek()`操作来调用主类的`doSomething()`静态方法，以使线程忙碌一段时间。参考以下代码：

```java
int doSomething(int i){
  IntStream.range(i, 100000).asDoubleStream().map(Math::sqrt).average();
  return i;
}
```

如你所见，`doSomething()`方法生成了一个范围在`i`到`99999`之间的整数流；然后将流转换为双精度流，计算每个流元素的平方根，最后计算流元素的平均值。我们丢弃结果并返回传入的参数，这样可以保持流水线的流畅风格，最后打印出每个元素。使用这个新类，我们可以演示三个线程的并发执行，如下所示：

```java
Thread thr1 = new AThread(1, 4);
thr1.start();

Thread thr2 = new AThread(11, 14);
thr2.start();

IntStream.range(21, 24)
         .peek(Chapter07Concurrency::doSomething)
         .forEach(System.out::println);

```

第一个线程生成整数`1`、`2`和`3`，第二个生成整数`11`、`12`和`13`，第三个线程（主线程）生成`21`、`22`和`23`。

如前所述，我们可以通过创建和使用一个实现`Runnable`接口的类来重写相同的程序：

```java
class ARunnable implements Runnable {
  int i1,i2;
  ARunnable(int i1, int i2){
    this.i1 = i1;
    this.i2 = i2;
  }
  public void run() {
    IntStream.range(i1, i2)
             .peek(Chapter07Concurrency::doSomething)
             .forEach(System.out::println);
  }
}
```

可以像这样运行相同的三个线程：

```java
Thread thr1 = new Thread(new ARunnable(1, 4));
thr1.start();

Thread thr2 = new Thread(new ARunnable(11, 14));
thr2.start();

IntStream.range(21, 24)
         .peek(Chapter07Concurrency::doSomething)
         .forEach(System.out::println);

```

我们还可以利用`Runnable`作为一个函数式接口，通过传递 lambda 表达式来避免创建中间类：

```java
Thread thr1 = new Thread(() -> IntStream.range(1, 4)
                  .peek(Chapter07Concurrency::doSomething)
                  .forEach(System.out::println));
thr1.start();

Thread thr2 = new Thread(() -> IntStream.range(11, 14)
                  .peek(Chapter07Concurrency::doSomething)
                  .forEach(System.out::println));
thr2.start();

IntStream.range(21, 24)
         .peek(Chapter07Concurrency::doSomething)
         .forEach(System.out::println);

```

哪种实现更好取决于你的目标和风格。实现`Runnable`有一个优势（在某些情况下，是唯一可能的选项），允许实现扩展另一个类。当你想要向现有类添加类似线程的行为时，这是特别有帮助的。甚至可以直接调用`run()`方法，而不必将对象传递给`Thread`构造函数。

当只需要`run()`方法的实现时，使用 lambda 表达式胜过`Runnable`实现，无论它有多大。如果它太大，可以将其隔离在一个单独的方法中：

```java
public static void main(String arg[]) {
  Thread thr1 = new Thread(() -> runImpl(1, 4));
  thr1.start();

  Thread thr2 = new Thread(() -> runImpl(11, 14));
  thr2.start();

  runImpl(21, 24);
}

private static void runImpl(int i1, int i2){
  IntStream.range(i1, i2)
           .peek(Chapter07Concurrency::doSomething)
           .forEach(System.out::println);
}
```

很难想出比前述功能更短的实现。

如果运行任何前述版本，将会得到类似以下的输出：

![](img/02e8e127-3a86-4d96-aa56-8c08b69770a4.png)

正如你所看到的，这三个线程同时打印出它们的数字，但顺序取决于特定的 JVM 实现和底层操作系统。因此，你可能会得到不同的输出。此外，它也可能在每次运行时发生变化。

`Thread`类有几个构造函数，允许设置线程的名称和所属的组。将线程分组有助于管理它们，如果有许多线程并行运行。该类还有几个方法，提供有关线程状态和属性的信息，并允许我们控制线程的行为。将这两行添加到前面的示例中：

```java
System.out.println("Id=" + thr1.getId() + ", " + thr1.getName() + ",
                   priority=" + thr1.getPriority() + ",
                   state=" + thr1.getState());
System.out.println("Id=" + thr2.getId() + ", " + thr2.getName() + ",
                   priority=" + thr2.getPriority() + ",
                   state=" + thr2.getState());
```

前述代码的结果将会是这样的：

![](img/ab04ad0e-d622-42c0-a51f-9bac8a1b1837.png)

接下来，假设你给每个线程添加一个名字：

```java
Thread thr1 = new Thread(() -> runImpl(1, 4), "First Thread");
thr1.start();

Thread thr2 = new Thread(() -> runImpl(11, 14), "Second Thread");
thr2.start();

```

在这种情况下，输出将显示如下内容：

![](img/e66d2aab-17fb-4b83-ac92-adff52f6f43e.png)

线程的`id`是自动生成的，不能更改，但在线程终止后可以重新使用。另一方面，可以为多个线程设置相同的名称。执行优先级可以通过编程方式设置，其值介于`Thread.MIN_PRIORITY`和`Thread.MAX_PRIORITY`之间。值越小，线程被允许运行的时间就越长，这意味着它具有更高的优先级。如果没有设置，优先级值默认为`Thread.NORM_PRIORITY`。

线程的状态可以有以下值之一：

+   `NEW`：当一个线程尚未启动时

+   `RUNNABLE`：当一个线程正在执行时

+   `BLOCKED`：当一个线程被阻塞并且正在等待监视器锁时

+   `WAITING`：当一个线程无限期地等待另一个线程执行特定操作时

+   `TIMED_WAITING`：当一个线程等待另一个线程在指定的等待时间内执行操作时

+   `TERMINATED`：当一个线程已经退出时

`sleep()`方法可用于暂停线程执行一段指定的时间（以毫秒为单位）。补充的`interrupt()`方法会向线程发送`InterruptedException`，可以用来唤醒*休眠*的线程。让我们在代码中解决这个问题并创建一个新的类：

```java
class BRunnable implements Runnable {
  int i1, result;
  BRunnable(int i1){ this.i1 = i1; }
  public int getCurrentResult(){ return this.result; }
  public void run() {
    for(int i = i1; i < i1 + 6; i++){
      //Do something useful here
      this.result = i;
      try{ Thread.sleep(1000);
      } catch(InterruptedException ex){}
    }
  }
}
```

前面的代码会产生中间结果，这些结果存储在`result`属性中。每次产生新的结果时，线程都会暂停（休眠）一秒钟。在这个特定的示例中，仅用于演示目的的代码并没有做任何特别有用的事情。它只是迭代一组值，并将每个值视为结果。在实际的代码中，您将根据系统的当前状态进行一些计算，并将计算出的值分配给`result`属性。现在让我们使用这个类：

```java
BRunnable r1 = new BRunnable(1);
Thread thr1 = new Thread(r1);
thr1.start();

IntStream.range(21, 29)
         .peek(i -> thr1.interrupt())
         .filter(i ->  {
           int res = r1.getCurrentResult();
           System.out.print(res + " => ");
           return res % 2 == 0;
         })
         .forEach(System.out::println);

thr1 thread and lets it generate the next result, which is then accessed via the getCurrentResult() method. If the current result is an even number, the filter allows the generated number flow to be printed out. If not, it is skipped. Here is a possible result:
```

![](img/d420d69b-2e20-4865-a348-b973dda270f7.png)

输出在不同的计算机上可能看起来不同，但你明白了：这样，一个线程可以控制另一个线程的输出。

# 还有更多...

还有两个支持线程协作的重要方法。第一个是`join()`方法，它允许当前线程等待另一个线程终止。`join()`的重载版本接受定义线程在可以做其他事情之前必须等待多长时间的参数。

`setDaemon()`方法可用于在所有非守护线程终止后使线程自动终止。通常，守护线程用于后台和支持进程。

# 不同的同步方法

在这个示例中，您将学习 Java 中管理共享资源的并发访问的两种最流行的方法：`synchronized method`和`synchronized block`。

# 做好准备

两个或更多线程同时修改相同的值，而其他线程读取它，这是并发访问问题的最一般描述。更微妙的问题包括线程干扰和内存一致性错误，这两者都会在看似良性的代码片段中产生意外结果。我们将演示这些情况以及避免它们的方法。

乍一看，这似乎很简单：只允许一个线程一次修改/访问资源，就可以了。但是如果访问需要很长时间，就会创建一个可能消除多个线程并行工作优势的瓶颈。或者，如果一个线程在等待访问另一个资源时阻塞了对一个资源的访问，而第二个线程在等待对第一个资源的访问时阻塞了对第二个资源的访问，就会创建一个称为死锁的问题。这些都是程序员在处理多个线程时可能面临的挑战的两个非常简单的例子。

# 如何做...

首先，我们将重现由并发修改相同值引起的问题。让我们创建一个`Calculator`类，其中有`calculate()`方法：

```java
class Calculator {
   private double prop;
   public double calculate(int i){
      DoubleStream.generate(new Random()::nextDouble).limit(50);
      this.prop = 2.0 * i;
      DoubleStream.generate(new Random()::nextDouble).limit(100);
      return Math.sqrt(this.prop);
   }
}
```

这种方法将一个输入值分配给一个属性，然后计算其平方根。我们还插入了两行代码，生成了 50 和 100 个值的流。我们这样做是为了让方法忙一段时间。否则，一切都会很快完成，几乎没有并发的机会。我们添加生成 100 个值的代码给另一个线程一个机会，在当前线程计算当前线程刚刚分配的值的平方根之前，为`prop`字段分配另一个值。

现在我们将在以下代码片段中使用`calculate()`方法：

```java
Calculator c = new Calculator();
Runnable runnable = () -> System.out.println(IntStream.range(1, 40)
                                   .mapToDouble(c::calculate).sum());
Thread thr1 = new Thread(runnable);
thr1.start();
Thread thr2 = new Thread(runnable);
thr2.start();
```

前两个线程同时修改同一个`Calculator`对象的同一个属性。以下是我们从其中一个运行中得到的结果：

```java
231.69407148192175
237.44481627598856
```

如果您在计算机上运行这些示例，并且看不到并发效果，请尝试通过将*slowing*行中的`100`替换为`1000`来增加生成的双倍数量，例如。当线程的结果不同时，这意味着在设置`prop`字段的值并在`calculate()`方法中返回其平方根之间的时间段内，另一个线程设法为`prop`分配了不同的值。这是线程干扰的情况。

保护代码免受这种问题的两种方法：使用`synchronized method`或`synchronized block`——两者都有助于在没有其他线程干扰的情况下执行代码作为原子操作。

制作`synchronized method`很容易和直接：

```java
class Calculator{
  private double prop;
 synchronized public double calculate(int i){
     DoubleStream.generate(new Random()::nextDouble).limit(50);
     this.prop = 2.0 * i;
     DoubleStream.generate(new Random()::nextDouble).limit(100);
     return Math.sqrt(this.prop);
  }
}
```

我们只需在方法定义前面添加`synchronized`关键字。现在，两个线程的结果将始终相同：

```java
233.75710300331153
233.75710300331153
```

这是因为另一个线程在当前线程（已经进入方法的线程）退出之前，无法进入同步方法。如果方法执行时间很长，这种方法可能会导致性能下降。在这种情况下，可以使用`synchronized block`，它不是包装整个方法，而是包装几行代码，使其成为原子操作。在我们的情况下，我们可以将生成 50 个值的*slowing*代码行移出同步块：

```java
class Calculator{
  private double prop;
  public double calculate(int i){
  DoubleStream.generate(new Random()::nextDouble).limit(50);
 synchronized (this) {
       this.prop = 2.0 * i;
       DoubleStream.generate(new Random()::nextDouble).limit(100);
       return Math.sqrt(this.prop);
    }
  }

```

这样，同步部分要小得多，因此它更少地成为瓶颈的机会。

`synchronized block`在对象上获取锁—任何对象，无论如何。在一个庞大的类中，您可能不会注意到当前对象（this）被用作多个块的锁。并且在类上获取的锁更容易出现意外共享。因此，最好使用专用锁：

```java
class Calculator{
  private double prop;
  private Object calculateLock = new Object();
  public double calculate(int i){
    DoubleStream.generate(new Random()::nextDouble).limit(50);
    synchronized (calculateLock) {
       this.prop = 2.0 * i;
       DoubleStream.generate(new Random()::nextDouble).limit(100);
       return Math.sqrt(this.prop);
    }
  }
}
```

专用锁具有更高的保证级别，该锁将用于仅访问特定块。

我们做了所有这些示例，只是为了演示同步方法。如果它们是真正的代码，我们将让每个线程创建自己的`Calculator`对象：

```java
    Runnable runnable = () -> {
        Calculator c = new Calculator();
        System.out.println(IntStream.range(1, 40)
                .mapToDouble(c::calculate).sum());
    };
    Thread thr1 = new Thread(runnable);
    thr1.start();
    Thread thr2 = new Thread(runnable);
    thr2.start();
```

这将符合使 lambda 表达式独立于它们创建的上下文的一般思想。这是因为在多线程环境中，人们永远不知道它们执行期间上下文会是什么样子。每次创建一个新对象的成本是可以忽略不计的，除非必须处理大量数据，并且测试确保对象创建开销是显而易见的。

在多线程环境中，内存一致性错误可能具有许多形式和原因。它们在`java.util.concurrent`包的 Javadoc 中有很好的讨论。在这里，我们只提到最常见的情况，即由于可见性不足而引起的。当一个线程更改属性值时，另一个可能不会立即看到更改，并且您不能为原始类型使用`synchronized`关键字。在这种情况下，考虑为属性使用`volatile`关键字；它保证了不同线程之间的读/写可见性。

# 还有更多...

不同类型的锁用于不同的需求，并具有不同的行为，这些锁被组装在`java.util.concurrent.locks`包中。

`java.util.concurrent.atomic`包提供了对单个变量进行无锁、线程安全编程的支持。

以下类也提供了同步支持：

+   `Semaphore`：这限制了可以访问资源的线程数量

+   `CountDownLatch`：这允许一个或多个线程等待，直到其他线程中执行的一组操作完成

+   `CyclicBarrier`：这允许一组线程等待彼此达到一个共同的屏障点

+   `Phaser`：这提供了一种更灵活的屏障形式，可用于控制多个线程之间的分阶段计算

+   `Exchanger`：这允许两个线程在会合点交换对象，并在几个管道设计中非常有用。

Java 中的每个对象都继承了基本对象的`wait()`、`notify()`和`notifyAll()`方法。这些方法也可以用来控制线程的行为和它们对锁的访问。

`Collections`类有方法来同步各种集合。然而，这意味着只有对集合的修改才能变得线程安全，而对集合成员的更改则不是。此外，通过其迭代器遍历集合时，也必须进行保护，因为迭代器不是线程安全的。以下是一个正确使用同步映射的 Javadoc 示例：

```java
 Map m = Collections.synchronizedMap(new HashMap());
 ...
 Set s = m.keySet(); // Needn't be in synchronized block
 ...
 synchronized (m) { // Synchronizing on m, not s!
   Iterator i = s.iterator(); //Must be synchronized block
   while (i.hasNext())
   foo(i.next());
 }
```

作为程序员，你必须意识到以下代码不是线程安全的：

```java
List<String> l = Collections.synchronizedList(new ArrayList<>());
l.add("first");
//... code that adds more elements to the list
int i = l.size();
//... some other code
l.add(i, "last");

```

这是因为虽然`List l`是同步的，在多线程处理中，很可能会有其他代码向列表添加更多元素或删除元素。

并发问题并不容易解决。这就是为什么越来越多的开发人员现在采取更激进的方法。他们更喜欢在一组无状态操作中处理数据，而不是管理对象状态。我们在第五章中看到了这种代码的例子，*流和管道*。看来 Java 和许多现代语言和计算机系统正在朝着这个方向发展。

# 不可变性作为实现并发的手段

在这个示例中，你将学习如何使用不可变性来解决并发引起的问题。

# 准备就绪

并发问题最常发生在不同的线程修改和读取同一共享资源的数据时。减少修改操作的数量会减少并发问题的风险。这就是不可变性——只读值的条件——进入舞台的地方。

对象的不可变性意味着在创建对象后无法更改其状态。这并不能保证线程安全，但有助于显著增加线程安全性，并在许多实际应用程序中提供足够的保护，以防止并发问题。

创建一个新对象而不是通过设置器和获取器更改其状态来重用现有对象通常被认为是一种昂贵的方法。但是随着现代计算机的强大，必须大量创建对象才能显著影响性能。即使是这种情况，程序员通常也会选择一些性能下降作为获得可预测结果的代价。

# 如何做...

下面是一个产生可变对象的类的示例：

```java
class MutableClass{
  private int prop;
  public MutableClass(int prop){
    this.prop = prop;
  }
  public int getProp(){
    return this.prop;
  }
  public void setProp(int prop){
    this.prop = prop;
  }
}
```

要使其不可变，我们需要删除设置器，并将`final`关键字添加到其唯一属性和类本身：

```java
final class ImmutableClass{
  final private int prop;
  public ImmutableClass(int prop){
    this.prop = prop;
  }
  public int getProp(){
    return this.prop;
  }
}
```

向类添加`final`关键字可以防止其被扩展，因此其方法不能被覆盖。向私有属性添加`final`并不那么明显。动机有些复杂，与编译器在对象构造期间重新排序字段的方式有关。如果字段声明为`final`，编译器会将其视为同步。这就是为什么向私有属性添加`final`是必要的，以使对象完全不可变。

如果类由其他类组成，特别是可变类，挑战就会增加。当这种情况发生时，注入的类可能会带入会影响包含类的代码。此外，通过 getter 引用检索的内部（可变）类可能会被修改并传播更改到包含类内部。关闭这种漏洞的方法是在对象检索的组合期间生成新对象。以下是一个示例：

```java
final class ImmutableClass{
  private final double prop;
  private final MutableClass mutableClass;
  public ImmutableClass(double prop, MutableClass mc){
    this.prop = prop;
    this.mutableClass = new MutableClass(mc.getProp());
  }
  public double getProp(){
    return this.prop;
  }
  public MutableClass getMutableClass(){
    return new MutableClass(mutableClass.getProp());
  }
}
```

# 还有更多...

在我们的示例中，我们使用了非常简单的代码。如果任何方法中添加了更多复杂性，特别是带有参数（尤其是当一些参数是对象时），可能会再次出现并发问题：

```java
int getSomething(AnotherMutableClass amc, String whatever){
  //... code is here that generates a value "whatever" 
  amc.setProperty(whatever);
  //...some other code that generates another value "val"
  amc.setAnotherProperty(val);
  return amc.getIntValue();
}
```

即使此方法属于`ImmutableClass`，并且不影响`ImmutableClass`对象的状态，它仍然是线程竞争的主题，并且必须根据需要进行分析和保护。

`Collections`类具有使各种集合不可修改的方法。这意味着集合本身的修改变为只读，而不是集合成员。

# 使用并发集合

在本教程中，您将了解`java.util.concurrent`包的线程安全集合。

# 准备就绪

如果对集合应用`Collections.synchronizeXYZ()`方法之一，则可以对集合进行同步；在这里，我们使用 XYZ 作为占位符，表示`Set`、`List`、`Map`或几种集合类型之一（请参阅`Collections`类的 API）。我们已经提到，同步适用于集合本身，而不适用于其迭代器或集合成员。

这种同步集合也被称为**包装器**，因为所有功能仍由作为参数传递给`Collections.synchronizeXYZ()`方法的集合提供，而包装器仅提供对它们的线程安全访问。通过在原始集合上获取锁也可以实现相同的效果。显然，在多线程环境中，这种同步会产生性能开销，导致每个线程等待轮流访问集合。

`java.util.concurrent`包提供了性能实现线程安全集合的良好调整的应用程序。

# 如何做...

`java.util.concurrent`包的每个并发集合都实现（或扩展，如果它是一个接口）`java.util`包的四个接口之一：`List`、`Set`、`Map`或`Queue`：

1.  `List`接口只有一个实现：`CopyOnWriteArrayList`类。以下摘自此类的 Javadoc：

“所有改变操作（添加、设置等）都是通过制作基础数组的新副本来实现的...“快照”样式的迭代器方法使用对迭代器创建时数组状态的引用。在迭代器的生命周期内，此数组永远不会更改，因此不可能发生干扰，并且保证迭代器不会抛出`ConcurrentModificationException`。迭代器不会反映自创建迭代器以来对列表的添加、删除或更改。迭代器本身的元素更改操作（删除、设置和添加）不受支持。这些方法会抛出`UnsupportedOperationException`。”

1.  为了演示`CopyOnWriteArrayList`类的行为，让我们将其与`java.util.ArrayList`（不是`List`的线程安全实现）进行比较。以下是在迭代相同列表时向列表添加元素的方法：

```java
        void demoListAdd(List<String> list) {
          System.out.println("list: " + list);
          try {
            for (String e : list) {
              System.out.println(e);
              if (!list.contains("Four")) {
                System.out.println("Calling list.add(Four)...");
                list.add("Four");
              }
            }
          } catch (Exception ex) {
            System.out.println(ex.getClass().getName());
          }
          System.out.println("list: " + list);
        }
```

考虑以下代码：

```java
        System.out.println("***** ArrayList add():");
        demoListAdd(new ArrayList<>(Arrays
                          .asList("One", "Two", "Three")));

        System.out.println();
        System.out.println("***** CopyOnWriteArrayList add():");
        demoListAdd(new CopyOnWriteArrayList<>(Arrays.asList("One", 
                                         "Two", "Three")));

```

如果执行此代码，结果将如下所示：

![](img/4ff7a7b6-d0d6-4594-9007-d4f8fe5e9d31.png)

正如您所看到的，当在迭代列表时修改列表时，`ArrayList`会抛出`ConcurrentModificationException`（我们为简单起见使用了相同的线程，因为它会导致与另一个线程修改列表的情况一样的效果）。尽管规范并不保证会抛出异常或应用列表修改（如我们的情况），因此程序员不应该基于这种行为来构建应用程序逻辑。

另一方面，`CopyOnWriteArrayList`类容忍相同的干预；但请注意，它不会将新元素添加到当前列表中，因为迭代器是从底层数组的新副本快照创建的。

现在让我们尝试使用这种方法在遍历列表时并发地删除列表元素：

```java
       void demoListRemove(List<String> list) {
          System.out.println("list: " + list);
          try {
            for (String e : list) {
              System.out.println(e);
              if (list.contains("Two")) {
                System.out.println("Calling list.remove(Two)...");
                list.remove("Two");
              }
            }
          } catch (Exception ex) {
            System.out.println(ex.getClass().getName());
          }
          System.out.println("list: " + list);
       }
```

考虑以下代码：

```java
        System.out.println("***** ArrayList remove():");
        demoListRemove(new ArrayList<>(Arrays.asList("One", 
                                         "Two", "Three")));
        System.out.println();
        System.out.println("***** CopyOnWriteArrayList remove():");
        demoListRemove(new CopyOnWriteArrayList<>(Arrays
                                .asList("One", "Two", "Three")));

```

如果我们执行这个，我们将得到以下结果：

![](img/4bef3bd5-c517-4ec4-9882-24da9bec2772.png)

行为与前面的例子类似。`CopyOnWriteArrayList`类允许并发访问列表，但不允许修改当前列表的副本。

我们很久以前就知道`ArrayList`不是线程安全的，因此我们使用了不同的技术来在遍历列表时删除列表中的元素。这是在 Java 8 发布之前完成的：

```java
        void demoListIterRemove(List<String> list) {
          System.out.println("list: " + list);
          try {
            Iterator iter = list.iterator();
            while (iter.hasNext()) {
              String e = (String) iter.next();
              System.out.println(e);
              if ("Two".equals(e)) {
                System.out.println("Calling iter.remove()...");
                iter.remove();
              }
            }
          } catch (Exception ex) {
              System.out.println(ex.getClass().getName());
          }
          System.out.println("list: " + list);
        }
```

让我们尝试这样做并运行代码：

```java
        System.out.println("***** ArrayList iter.remove():");
        demoListIterRemove(new ArrayList<>(Arrays
                            .asList("One", "Two", "Three")));
        System.out.println();
        System.out.println("*****" 
                    + " CopyOnWriteArrayList iter.remove():");
        demoListIterRemove(new CopyOnWriteArrayList<>(Arrays
                             .asList("One", "Two", "Three")));

```

结果如下：

![](img/c3cdf125-0dbf-4fec-96b7-379aed0651a6.png)

这正是 Javadoc 所警告的（[`docs.oracle.com/cd/E17802_01/j2se/j2se/1.5.0/jcp/beta2/apidiffs/java/util/concurrent/CopyOnWriteArrayList.html`](https://docs.oracle.com/cd/E17802_01/j2se/j2se/1.5.0/jcp/beta2/apidiffs/java/util/concurrent/CopyOnWriteArrayList.html)）：

"迭代器本身的元素更改操作（删除、设置和添加）不受支持。这些方法会抛出 UnsupportedOperationException 异常。"

当我们将应用程序升级以使其在多线程环境中工作时，我们应该记住这一点——如果我们使用迭代器来删除列表元素，仅仅从`ArrayList()`更改为`CopyOnWriteArrayList`是不够的。

自 Java 8 以来，有一种更好的方法可以使用 lambda 从集合中删除元素，应该使用它，因为它将管道细节留给库代码：

```java
        void demoRemoveIf(Collection<String> collection) {
              System.out.println("collection: " + collection);
              System.out.println("Calling list.removeIf(e ->" 
                                      + " Two.equals(e))...");
              collection.removeIf(e -> "Two".equals(e));
              System.out.println("collection: " + collection);
        }
```

所以让我们这样做：

```java
        System.out.println("***** ArrayList list.removeIf():");
        demoRemoveIf(new ArrayList<>(Arrays
                              .asList("One", "Two", "Three")));
        System.out.println();
        System.out.println("*****" 
                   + " CopyOnWriteArrayList list.removeIf():");
        demoRemoveIf(new CopyOnWriteArrayList<>(Arrays
                              .asList("One", "Two", "Three")));

```

上述代码的结果如下：

![](img/f11bd846-7dc6-4cd3-9d78-701ecafaba2a.png)

它很简短，并且不会出现任何与集合相关的问题，符合使用流、lambda 和函数接口进行无状态并行计算的一般趋势。

此外，当我们将应用程序升级为使用`CopyOnWriteArrayList`类后，我们可以利用一种更简单的方法向列表中添加新元素（无需首先检查它是否已存在）：

```java
CopyOnWriteArrayList<String> list =  
  new CopyOnWriteArrayList<>(Arrays.asList("Five","Six","Seven"));
list.addIfAbsent("One");

```

使用`CopyOnWriteArrayList`，这可以作为原子操作完成，因此不需要同步如果-不存在-则添加代码块。

1.  让我们回顾一下实现`Set`接口的`java.util.concurrent`包的并发集合。有三种这样的实现——`ConcurrentHashMap.KeySetView`、`CopyOnWriteArraySet`和`ConcurrentSkipListSet`。

第一个是`ConcurrentHashMap`键的视图。它由`ConcurrentHashMap`支持（可以通过`getMap()`方法检索）。我们稍后将审查`ConcurrentHashMap`的行为。

`java.util.concurrent`包中`Set`的第二个实现是`CopyOnWriteArraySet`类。其行为类似于`CopyOnWriteArrayList`类。实际上，它在底层使用了`CopyOnWriteArrayList`类的实现。唯一的区别是它不允许集合中有重复元素。

在`java.util.concurrent`包中的第三（也是最后一个）`Set`实现是`ConcurrentSkipListSet`；它实现了`Set`的一个子接口`NavigableSet`。根据`ConcurrentSkipListSet`类的 Javadoc，插入、移除和访问操作可以由多个线程安全并发执行。Javadoc 中也描述了一些限制：

+   +   它不允许使用`null`元素。

+   集合的大小是通过遍历集合动态计算的，因此如果在操作期间修改了此集合，它可能报告不准确的结果。

+   `addAll()`，`removeIf()`和`forEach()`操作不能保证原子执行。例如，如果`forEach()`操作与`addAll()`操作并发进行，可能会“观察到只有一些添加的元素”。

`ConcurrentSkipListSet`类的实现基于`ConcurrentSkipListMap`类，我们将很快讨论。为了演示`ConcurrentSkipListSet`类的行为，让我们将其与`java.util.TreeSet`类（`NavigableSet`的非并发实现）进行比较。我们首先移除一个元素：

```java
        void demoNavigableSetRemove(NavigableSet<Integer> set) {
          System.out.println("set: " + set);
          try {
            for (int i : set) {
              System.out.println(i);
              System.out.println("Calling set.remove(2)...");
              set.remove(2);
            }
          } catch (Exception ex) {
            System.out.println(ex.getClass().getName());
          }
          System.out.println("set: " + set);
        }
```

当然，这段代码并不是很高效；我们多次移除了相同的元素而没有检查其是否存在。我们这样做只是为了演示目的。此外，自 Java 8 以来，相同的`removeIf()`方法对`Set`也可以正常工作。但我们想提出新的`ConcurrentSkipListSet`类的行为，所以让我们执行这段代码：

```java
        System.out.println("***** TreeSet set.remove(2):");
        demoNavigableSetRemove(new TreeSet<>(Arrays
                                     .asList(0, 1, 2, 3)));
        System.out.println();
        System.out.println("*****"
                    + " ConcurrentSkipListSet set.remove(2):");
        demoNavigableSetRemove(new ConcurrentSkipListSet<>(Arrays
                                     .asList(0, 1, 2, 3)));

```

输出将如下所示：

![](img/2bacd443-00a8-45c1-8a5c-14c5fc52e246.png)

正如预期的那样，`ConcurrentSkipListSet`类处理并发性，甚至从当前集合中移除一个元素，这是有帮助的。它还通过迭代器移除元素而不会抛出异常。考虑以下代码：

```java
        void demoNavigableSetIterRemove(NavigableSet<Integer> set){
          System.out.println("set: " + set);
          try {
            Iterator iter = set.iterator();
            while (iter.hasNext()) {
              Integer e = (Integer) iter.next();
              System.out.println(e);
              if (e == 2) {
                System.out.println("Calling iter.remove()...");
                iter.remove();
              }
            }
          } catch (Exception ex) {
            System.out.println(ex.getClass().getName());
          }
          System.out.println("set: " + set);
        }
```

对`TreeSet`和`ConcurrentSkipListSet`运行此操作：

```java
        System.out.println("***** TreeSet iter.remove():");
        demoNavigableSetIterRemove(new TreeSet<>(Arrays
                                           .asList(0, 1, 2, 3)));

        System.out.println();
        System.out.println("*****"
                      + " ConcurrentSkipListSet iter.remove():");
        demoNavigableSetIterRemove(new ConcurrentSkipListSet<>
                                    (Arrays.asList(0, 1, 2, 3)));
```

我们不会得到任何异常：

![](img/179b44f4-2176-4707-bb2a-dfd3d85e9793.png)

这是因为根据 Javadoc，`ConcurrentSkipListSet`的迭代器是弱一致的，这意味着以下内容：

+   +   它们可以与其他操作同时进行

+   它们永远不会抛出`ConcurrentModificationException`

+   它们保证遍历元素时仅在构造时存在一次，并且可能（但不保证）反映构造后的任何修改（来自 Javadoc）

这个“不保证”的部分有些令人失望，但比起`CopyOnWriteArrayList`抛出异常要好。

向`Set`类添加不像向`List`类那样有问题，因为`Set`不允许重复，并且会在内部处理必要的检查：

```java
        void demoNavigableSetAdd(NavigableSet<Integer> set) {
          System.out.println("set: " + set);
          try {
            int m = set.stream().max(Comparator.naturalOrder())
                                .get() + 1;
            for (int i : set) {
              System.out.println(i);
              System.out.println("Calling set.add(" + m + ")");
              set.add(m++);
              if (m > 6) {
                break;
              }
            }
          } catch (Exception ex) {
            System.out.println(ex.getClass().getName());
          }
          System.out.println("set: " + set);
        }
```

考虑以下代码：

```java
        System.out.println("***** TreeSet set.add():");
        demoNavigableSetAdd(new TreeSet<>(Arrays
                                     .asList(0, 1, 2, 3)));

        System.out.println();
        System.out.println("*****" 
                            + " ConcurrentSkipListSet set.add():");
        demoNavigableSetAdd(new ConcurrentSkipListSet<>(Arrays
                                        .asList(0,1,2,3)));

```

如果我们运行这个，我们将得到以下结果：

![](img/6063c61a-3f6d-4680-a137-5b6bc60d1d73.png)

与之前一样，我们观察到并发`Set`版本处理并发性更好。

1.  让我们转向`Map`接口，在`java.util.concurrent`包中有两个实现：`ConcurrentHashMap`和`ConcurrentSkipListMap`。

来自`ConcurrentHashMap`类的 Javadoc。

“支持检索的完全并发性和更新的高并发性”

它是`java.util.HashMap`的线程安全版本，并且在这方面类似于`java.util.Hashtable`。实际上，`ConcurrentHashMap`类满足与`java.util.Hashtable`相同的功能规范要求，尽管其实现在“同步细节上有些不同”（来自 Javadoc）。

与`java.util.HashMap`和`java.util.Hashtable`不同，`ConcurrentHashMap`支持，根据其 Javadoc（[`docs.oracle.com/javase/9/docs/api/java/util/concurrent/ConcurrentHashMap.html`](https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/ConcurrentHashMap.html)），

“一组顺序和并行的大容量操作，与大多数 Stream 方法不同，它们被设计为安全地，通常是明智地，即使在被其他线程并发更新的映射中也可以应用”

+   +   `forEach()`: 这对每个元素执行给定的操作

+   `search()`: 这返回将给定函数应用于每个元素的第一个可用的非空结果

+   `reduce()`: 这累积每个元素（有五个重载版本）

这些大容量操作接受一个`parallelismThreshold`参数，允许推迟并行化，直到映射大小达到指定的阈值。当阈值设置为`Long.MAX_VALUE`时，将根本没有并行性。

类 API 中还有许多其他方法，因此请参考其 Javadoc 以获取概述。

与`java.util.HashMap`不同（类似于`java.util.Hashtable`），`ConcurrentHashMap`和`ConcurrentSkipListMap`都不允许将 null 用作键或值。

第二个`Map`的实现——`ConcurrentSkipListSet`类——基于我们之前提到的`ConcurrentSkipListMap`类，因此我们刚刚描述的`ConcurrentSkipListSet`类的所有限制也适用于`ConcurrentSkipListMap`类。`ConcurrentSkipListSet`类实际上是`java.util.TreeMap`的线程安全版本。`SkipList`是一种排序的数据结构，允许并发快速搜索。所有元素都根据它们的键的自然排序顺序进行排序。我们为`ConcurrentSkipListSet`类演示的`NavigableSet`功能在`ConcurrentSkipListMap`类中也存在。有关类 API 中的许多其他方法，请参考其 Javadoc。

现在让我们演示`java.util.HashMap`、`ConcurrentHashMap`和`ConcurrentSkipListMap`类在响应并发性方面的行为差异。首先，我们将编写生成测试`Map`对象的方法：

```java
        Map createhMap() {
          Map<Integer, String> map = new HashMap<>();
          map.put(0, "Zero");
          map.put(1, "One");
          map.put(2, "Two");
          map.put(3, "Three");
          return map;
       }
```

以下是将元素添加到`Map`对象的代码：

```java
        void demoMapPut(Map<Integer, String> map) {
          System.out.println("map: " + map);
          try {
            Set<Integer> keys = map.keySet();
            for (int i : keys) {
              System.out.println(i);
              System.out.println("Calling map.put(8, Eight)...");
              map.put(8, "Eight");

              System.out.println("map: " + map);
              System.out.println("Calling map.put(8, Eight)...");
              map.put(8, "Eight");

              System.out.println("map: " + map);
              System.out.println("Calling" 
                                 + " map.putIfAbsent(9, Nine)...");
              map.putIfAbsent(9, "Nine");

              System.out.println("map: " + map);
              System.out.println("Calling" 
                                 + " map.putIfAbsent(9, Nine)...");
              map.putIfAbsent(9, "Nine");

              System.out.println("keys.size(): " + keys.size());
              System.out.println("map: " + map);
            }
          } catch (Exception ex) {
            System.out.println(ex.getClass().getName());
          }
        }
```

对`Map`的所有三个实现运行此操作：

```java
        System.out.println("***** HashMap map.put():");
        demoMapPut(createhMap());

        System.out.println();
        System.out.println("***** ConcurrentHashMap map.put():");
        demoMapPut(new ConcurrentHashMap(createhMap()));

        System.out.println();
        System.out.println("*****"
                          + " ConcurrentSkipListMap map.put():");
        demoMapPut(new ConcurrentSkipListMap(createhMap()));

```

如果我们这样做，我们将只为第一个键的`HashMap`获得输出：

![](img/99df2694-2099-4ac5-8475-84cd92f6a363.png)

我们还为所有键（包括新添加的键）的`ConcurrentHashMap`和`ConcurrentSkipListMap`获得输出。以下是`ConcurrentHashMap`输出的最后一部分：

![](img/c935da11-bec2-4daf-b06f-6fc4e2d1aa33.png)

如前所述，不能保证会出现`ConcurrentModificationException`。现在我们看到，它被抛出的时刻（如果被抛出）是代码发现修改已经发生的时刻。在我们的例子中，它发生在下一个迭代中。另一个值得注意的是，即使我们将集合隔离在一个单独的变量中，当前的键集合也会发生变化：

```java
      Set<Integer> keys = map.keySet();
```

这提醒我们不要忽视通过它们的引用传播的对象的更改。

为了节省空间和时间，我们将不展示并发删除的代码，只总结结果。如预期的那样，当以任何以下方式之一删除元素时，`HashMap`会抛出`ConcurrentModificationException`异常：

```java
        String result = map.remove(2);
        boolean success = map.remove(2, "Two");

```

可以使用`Iterator`以以下一种方式之一进行并发删除：

```java
         iter.remove();
         boolean result = map.keySet().remove(2);
         boolean result = map.keySet().removeIf(e -> e == 2);

```

相比之下，这两个并发`Map`实现不仅允许使用`Iterator`进行并发元素删除。

`Queue`接口的所有并发实现也表现出类似的行为：`LinkedTransferQueue`、`LinkedBlockingQueue`、`LinkedBlockingDequeue`、`ArrayBlockingQueue`、`PriorityBlockingQueue`、`DelayQueue`、`SynchronousQueue`、`ConcurrentLinkedQueue`和`ConcurrentLinkedDequeue`，都在`java.util.concurrent`包中。但要演示它们所有将需要一个单独的卷，因此我们将其留给您浏览 Javadoc 并提供`ArrayBlockingQueue`的示例。队列将由`QueueElement`类表示：

```java
         class QueueElement {
           private String value;
           public QueueElement(String value){
             this.value = value;
           }
           public String getValue() {
             return value;
           }
         }
```

队列生产者将如下所示：

```java
        class QueueProducer implements Runnable {
          int intervalMs, consumersCount;
          private BlockingQueue<QueueElement> queue;
          public QueueProducer(int intervalMs, int consumersCount, 
                               BlockingQueue<QueueElement> queue) {
            this.consumersCount = consumersCount;
            this.intervalMs = intervalMs;
            this.queue = queue;
          }
          public void run() {
            List<String> list = 
               List.of("One","Two","Three","Four","Five");
            try {
              for (String e : list) {
                Thread.sleep(intervalMs);
                queue.put(new QueueElement(e));
                System.out.println(e + " produced" );
              }
              for(int i = 0; i < consumersCount; i++){
                queue.put(new QueueElement("Stop"));
              }
            } catch (InterruptedException e) {
              e.printStackTrace();
            }
           }
         }
```

以下将是队列消费者：

```java
        class QueueConsumer implements Runnable{
          private String name;
          private int intervalMs;
          private BlockingQueue<QueueElement> queue;
          public QueueConsumer(String name, int intervalMs, 
                               BlockingQueue<QueueElement> queue){
             this.intervalMs = intervalMs;
             this.queue = queue;
             this.name = name;
          }
          public void run() {
            try {
              while(true){
                String value = queue.take().getValue();
                if("Stop".equals(value)){
                  break;
                }
                System.out.println(value + " consumed by " + name);
                Thread.sleep(intervalMs);
              }
            } catch(InterruptedException e) {
              e.printStackTrace();
            }
          }
        }
```

运行以下代码：

```java
        BlockingQueue<QueueElement> queue = 
                      new ArrayBlockingQueue<>(5);
        QueueProducer producer = new QueueProducer(queue);
        QueueConsumer consumer = new QueueConsumer(queue);
        new Thread(producer).start();
        new Thread(consumer).start();

```

它的结果可能如下所示：

![](img/1f475fea-7d38-4c00-af7e-518ba520b778.png)

# 它是如何工作的...

在选择要使用的集合之前，请阅读 Javadoc，看看集合的限制是否适合您的应用程序。

例如，根据 Javadoc，`CopyOnWriteArrayList`类

“通常成本太高，但当遍历操作远远超过变异操作时可能更有效，并且在您不能或不想同步遍历但需要阻止并发线程之间干扰时非常有用。”

当您不需要在不同位置添加新元素并且不需要排序时，请使用它。否则，请使用`ConcurrentSkipListSet`。

根据 Javadoc，`ConcurrentSkipListSet`和`ConcurrentSkipListMap`类

“为包含、添加和删除操作及其变体提供预期的平均 log(n)时间成本。升序排序的视图及其迭代器比降序排序的视图及其迭代器更快。”

当您需要按特定顺序快速迭代元素时，请使用它们。

当并发要求非常苛刻并且需要在写操作上允许锁定但不需要锁定元素时，请使用`ConcurrentHashMap`。

当许多线程共享对公共集合的访问时，`ConcurrentLinkedQueque`和`ConcurrentLinkedDeque`是一个合适的选择。`ConcurrentLinkedQueque`采用了高效的非阻塞算法。

`PriorityBlockingQueue`是一个更好的选择，当自然顺序是可以接受的，并且您需要快速向队列尾部添加元素和快速从队列头部删除元素时。阻塞意味着队列在检索元素时等待变得非空，并在存储元素时等待队列中有空间可用。

`ArrayBlockingQueue`，`LinkedBlockingQueue`和`LinkedBlockingDeque`具有固定大小（它们是有界的）。其他队列是无界的。

将这些类似特性和建议作为指南，但在实现功能之前进行全面测试和性能测量。

# 使用执行器服务执行异步任务

在本示例中，您将学习如何使用`ExecutorService`来实现可控的线程执行。

# 准备就绪

在早期的示例中，我们演示了如何直接使用`Thread`类创建和执行线程。这是一个适用于少量线程的可接受机制，可以快速运行并产生可预测的结果。对于长时间运行且具有复杂逻辑的大规模应用程序（可能会使它们长时间保持活动状态）和/或线程数量也在不可预测增长的情况下，简单的创建和运行直到退出的方法可能会导致`OutOfMemory`错误，或者需要复杂的自定义线程状态维护和管理系统。对于这种情况，`ExecutorService`和`java.util.concurrent`包的相关类提供了一个开箱即用的解决方案，解除了程序员编写和维护大量基础设施代码的需要。

在 Executor Framework 的基础上，有一个只有一个`void execute(Runnable command)`方法的`Executor`接口，它在将来的某个时间执行给定的命令。

它的子接口`ExecutorService`添加了一些允许您管理执行器的方法：

+   `invokeAny()`，`invokeAll()`和`awaitTermination()`方法以及`submit()`允许您定义线程将如何执行以及它们是否预期返回一些值

+   `shutdown()`和`shutdownNow()`方法允许您关闭执行器

+   `isShutdown()`和`isTerminated()`方法提供了执行器的状态

`ExecutorService`的对象可以使用`java.util.concurrent.Executors`类的静态工厂方法创建：

+   `newSingleThreadExecutor()`: 创建一个使用单个工作线程并在无界队列上运行的`Executor`方法。它有一个带有`ThreadFactory`作为参数的重载版本。

+   `newCachedThreadPool()`: 创建一个线程池，根据需要创建新线程，但在可用时重用先前构造的线程。它有一个带有`ThreadFactory`作为参数的重载版本。

+   `newFixedThreadPool(int nThreads)`: 创建一个线程池，该线程池重用固定数量的线程，这些线程在共享的无界队列上运行。它有一个带有`ThreadFactory`作为参数的重载版本。

`ThreadFactory`实现允许您重写创建新线程的过程，使应用程序能够使用特殊的线程子类、优先级等。其使用示例超出了本书的范围。

# 如何做到这一点...

1.  您需要记住的`Executor`接口行为的一个重要方面是，一旦创建，它会一直运行（等待执行新任务），直到 Java 进程停止。因此，如果要释放内存，必须显式停止`Executor`接口。如果不关闭，被遗忘的执行程序将导致内存泄漏。以下是确保没有执行程序被遗留下来的一种可能方法：

```java
        int shutdownDelaySec = 1;
        ExecutorService execService = 
                       Executors.newSingleThreadExecutor();
        Runnable runnable =  () -> System.out.println("Worker One did
                                                       the job.");
        execService.execute(runnable);
        runnable =   () -> System.out.println("Worker Two did the 
                                               job.");
        Future future = execService.submit(runnable);
        try {
          execService.shutdown();
          execService.awaitTermination(shutdownDelaySec, 
                                       TimeUnit.SECONDS);
        } catch (Exception ex) {
          System.out.println("Caught around" 
                  + " execService.awaitTermination(): " 
                  + ex.getClass().getName());
        } finally {
          if (!execService.isTerminated()) {
            if (future != null && !future.isDone() 
                               && !future.isCancelled()){
              System.out.println("Cancelling the task...");
              future.cancel(true);
            }
          }
          List<Runnable> l = execService.shutdownNow();
          System.out.println(l.size() 
                 + " tasks were waiting to be executed." 
                 + " Service stopped.");
        }
```

您可以以各种方式将工作程序（`Runnable`或`Callable`功能接口的实现）传递给`ExecutorService`进行执行，我们将很快看到。在本例中，我们执行了两个线程：一个使用`execute()`方法，另一个使用`submit()`方法。这两种方法都接受`Runnable`或`Callable`，但在本例中我们只使用了`Runnable`。`submit()`方法返回`Future`，它表示异步计算的结果。

`shutdown()`方法启动先前提交的任务的有序关闭，并阻止接受任何新任务。此方法不等待任务完成执行。`awaitTermination()`方法会等待。但是在`shutdownDelaySec`之后，它停止阻塞，代码流进入`finally`块，在该块中，如果所有任务在关闭后都已完成，则`isTerminated()`方法返回 true。在本例中，我们在两个不同的语句中执行了两个任务。但请注意，`ExecutorService`的其他方法接受任务集合。

在这种情况下，当服务关闭时，我们遍历`Future`对象的集合。我们调用每个任务，如果任务尚未完成，则取消它，可能在取消任务之前执行其他必须完成的任务。等待多长时间（`shutdownDelaySec`的值）必须针对每个应用程序和可能正在运行的任务进行测试。

最后，`shutdownNow()`方法表示

“尝试停止所有正在执行的任务，停止等待任务的处理，并返回等待执行的任务列表”

（根据 Javadoc）。

1.  收集和评估结果。在实际应用中，我们通常不希望经常关闭服务。我们只检查任务的状态，并收集那些从`isDone()`方法返回 true 的任务的结果。在前面的代码示例中，我们只是展示了如何确保当我们停止服务时，我们以一种受控的方式进行，而不会留下任何失控的进程。如果运行该代码示例，我们将得到以下结果：

！[](img/3db89b96-4315-4b22-8ddb-3690ff485551.png)

1.  概括前面的代码并创建一个关闭服务和返回`Future`的任务的方法：

```java
        void shutdownAndCancelTask(ExecutorService execService, 
                  int shutdownDelaySec, String name, Future future) {
          try {
            execService.shutdown();
            System.out.println("Waiting for " + shutdownDelaySec 
                         + " sec before shutting down service...");
            execService.awaitTermination(shutdownDelaySec,
                                         TimeUnit.SECONDS);
          } catch (Exception ex) {
            System.out.println("Caught around" 
                        + " execService.awaitTermination():" 
                        + ex.getClass().getName());
         } finally {
           if (!execService.isTerminated()) {
             System.out.println("Terminating remaining tasks...");
             if (future != null && !future.isDone() 
                                && !future.isCancelled()) {
               System.out.println("Cancelling task " 
                                  + name + "...");
               future.cancel(true);
             }
           }
           System.out.println("Calling execService.shutdownNow(" 
                              + name + ")...");
           List<Runnable> l = execService.shutdownNow();
           System.out.println(l.size() + " tasks were waiting" 
                         + " to be executed. Service stopped.");
         }
       }
```

1.  通过使用 lambda 表达式使`Runnable`休眠一段时间（模拟需要完成的有用工作）来增强示例：

```java
        void executeAndSubmit(ExecutorService execService, 
                    int shutdownDelaySec, int threadSleepsSec) {
          System.out.println("shutdownDelaySec = " 
                          + shutdownDelaySec + ", threadSleepsSec = " 
                          + threadSleepsSec);
          Runnable runnable = () -> {
            try {
              Thread.sleep(threadSleepsSec * 1000);
              System.out.println("Worker One did the job.");
            } catch (Exception ex) {
              System.out.println("Caught around One Thread.sleep(): " 
                                 + ex.getClass().getName());
            }
          };
          execService.execute(runnable);
          runnable = () -> {
            try {
              Thread.sleep(threadSleepsSec * 1000);
              System.out.println("Worker Two did the job.");
            } catch (Exception ex) {
              System.out.println("Caught around Two Thread.sleep(): " 
                                 + ex.getClass().getName());
            }
          };
          Future future = execService.submit(runnable);
          shutdownAndCancelTask(execService, shutdownDelaySec, 
                                "Two", future);
        }
```

注意两个参数，`shutdownDelaySec`（定义服务在继续关闭自身之前等待多长时间，不允许提交新任务）和`threadSleepSec`（定义工作者睡眠的时间，表示模拟过程正在工作）。

1.  运行不同的`ExecutorService`实现和`shutdownDelaySec`和`threadSleepSec`值的新代码：

```java
        System.out.println("Executors.newSingleThreadExecutor():");
        ExecutorService execService = 
                       Executors.newSingleThreadExecutor();
        executeAndSubmit(execService, 3, 1);

        System.out.println();
        System.out.println("Executors.newCachedThreadPool():");
        execService = Executors.newCachedThreadPool();
        executeAndSubmit(execService, 3, 1);

        System.out.println();
        int poolSize = 3;
        System.out.println("Executors.newFixedThreadPool(" 
                                            + poolSize + "):");
        execService = Executors.newFixedThreadPool(poolSize);
        executeAndSubmit(execService, 3, 1);

```

这是输出的样子（在你的电脑上可能略有不同，取决于操作系统控制的事件的确切时间）：

![](img/37890a3f-7807-46ef-b5e7-a0ba414fc995.png)

1.  分析结果。在第一个例子中，我们没有惊喜，因为以下一行：

```java
        execService.awaitTermination(shutdownDelaySec, 
                                     TimeUnit.SECONDS);

```

它会阻塞三秒，而每个工作者只工作一秒。所以即使是单线程执行器，每个工作者都有足够的时间完成工作。

让服务只等待一秒：

![](img/43c2a4e6-d333-437c-a082-47aceb424a33.png)

当你这样做时，你会注意到没有一个任务会被完成。在这种情况下，工作者`One`被中断了（参见输出的最后一行），而任务`Two`被取消了。

让服务等待三秒：

![](img/6b9b8113-e7fe-442f-b0cf-e2fde29a1554.png)

现在我们看到工作者`One`能够完成它的任务，而工作者`Two`被中断了。

由`newCachedThreadPool()`或`newFixedThreadPool()`产生的`ExecutorService`接口在单核计算机上表现类似。唯一的显著区别是，如果`shutdownDelaySec`的值等于`threadSleepSec`的值，那么它们都允许你完成线程：

![](img/e2998059-be37-4833-83bc-be14f7c2ed44.png)

这是使用`newCachedThreadPool()`的结果。使用`newFixedThreadPool()`的例子在单核计算机上看起来完全一样。

1.  为了更好地控制任务，检查`Future`对象的返回值，而不仅仅是提交一个任务并希望它按需要完成。`ExecutorService`接口中还有一个名为`submit()`的方法，允许你不仅返回一个`Future`对象，还可以将结果作为第二个参数传递给该方法并包含在返回对象中。让我们看一个例子：

```java
        Future<Integer> future = execService.submit(() -> 
               System.out.println("Worker 42 did the job."), 42);
        int result = future.get();

```

`result`的值是`42`。当你提交了很多工作者（`nWorkers`）并且需要知道哪一个已经完成时，这个方法会很有帮助：

```java
        Set<Integer> set = new HashSet<>();
        while (set.size() < nWorkers){
          for (Future<Integer> future : futures) {
            if (future.isDone()){
              try {
                String id = future.get(1, TimeUnit.SECONDS);
                if(!set.contains(id)){
                  System.out.println("Task " + id + " is done.");
                  set.add(id);
                }
              } catch (Exception ex) {
                System.out.println("Caught around future.get(): "
                                   + ex.getClass().getName());
              }
            }
          }
        }
```

好吧，问题在于`future.get()`是一个阻塞方法。这就是为什么我们使用`get()`方法的一个版本，允许我们设置`delaySec`超时。否则，`get()`会阻塞迭代。

# 它是如何工作的...

让我们更接近实际代码，创建一个实现`Callable`并允许你将工作者的结果作为`Result`类对象返回的类：

```java
class Result {
  private int sleepSec, result;
  private String workerName;
  public Result(String workerName, int sleptSec, int result) {
    this.workerName = workerName;
    this.sleepSec = sleptSec;
    this.result = result;
  }
  public String getWorkerName() { return this.workerName; }
  public int getSleepSec() { return this.sleepSec; }
  public int getResult() { return this.result; }
}
```

实际的数值结果是由`getResult()`方法返回的。在这里，我们还包括了工作者的名字以及线程预计睡眠的时间（为了方便和更好地说明输出）。

工作者本身将是`CallableWorkerImpl`类的一个实例：

```java
class CallableWorkerImpl implements CallableWorker<Result>{
  private int sleepSec;
  private String name;
  public CallableWorkerImpl(String name, int sleepSec) {
    this.name = name;
    this.sleepSec = sleepSec;
  }
  public String getName() { return this.name; }
  public int getSleepSec() { return this.sleepSec; }
  public Result call() {
    try {
      Thread.sleep(sleepSec * 1000);
    } catch (Exception ex) {
      System.out.println("Caught in CallableWorker: " 
                         + ex.getClass().getName());
    }
    return new Result(name, sleepSec, 42);
  }
}
```

在这里，数字`42`是一个实际的数值结果，一个工作者在睡觉的时候计算出来的。`CallableWorkerImpl`类实现了`CallableWorker`接口：

```java
interface CallableWorker<Result> extends Callable<Result> {
  default String getName() { return "Anonymous"; }
  default int getSleepSec() { return 1; }
}
```

我们必须将方法设置为默认的并返回一些数据（它们无论如何都会被类实现覆盖）以保持其`functional interface`状态。否则，我们将无法在 lambda 表达式中使用它。

我们还将创建一个工厂，用于生成工作者列表：

```java
List<CallableWorker<Result>> createListOfCallables(int nSec){
  return List.of(new CallableWorkerImpl("One", nSec),
                 new CallableWorkerImpl("Two", 2 * nSec),
                 new CallableWorkerImpl("Three", 3 * nSec));
}
```

现在我们可以使用所有这些新的类和方法来演示`invokeAll()`方法：

```java
void invokeAllCallables(ExecutorService execService, 
        int shutdownDelaySec, List<CallableWorker<Result>> callables) {
  List<Future<Result>> futures = new ArrayList<>();
  try {
    futures = execService.invokeAll(callables, shutdownDelaySec, 
                                    TimeUnit.SECONDS);
  } catch (Exception ex) {
    System.out.println("Caught around execService.invokeAll(): " 
                       + ex.getClass().getName());
  }
  try {
    execService.shutdown();
    System.out.println("Waiting for " + shutdownDelaySec 
                       + " sec before terminating all tasks...");
    execService.awaitTermination(shutdownDelaySec,
                                 TimeUnit.SECONDS);
  } catch (Exception ex) {
    System.out.println("Caught around awaitTermination(): " 
                       + ex.getClass().getName());
  } finally {
    if (!execService.isTerminated()) {
      System.out.println("Terminating remaining tasks...");
      for (Future<Result> future : futures) {
        if (!future.isDone() && !future.isCancelled()) {
          try {
            System.out.println("Cancelling task "
                       + future.get(shutdownDelaySec, 
                               TimeUnit.SECONDS).getWorkerName());
            future.cancel(true);
          } catch (Exception ex) {
            System.out.println("Caught at cancelling task: " 
                               + ex.getClass().getName());
          }
        }
      }
    }
    System.out.println("Calling execService.shutdownNow()...");
    execService.shutdownNow();
  }
  printResults(futures, shutdownDelaySec);
}
```

`printResults()`方法输出从工作者那里收到的结果：

```java
void printResults(List<Future<Result>> futures, int timeoutSec) {
  System.out.println("Results from futures:");
  if (futures == null || futures.size() == 0) {
    System.out.println("No results. Futures" 
                       + (futures == null ? " = null" : ".size()=0"));
  } else {
    for (Future<Result> future : futures) {
      try {
        if (future.isCancelled()) {
          System.out.println("Worker is cancelled.");
        } else {
          Result result = future.get(timeoutSec, TimeUnit.SECONDS);
          System.out.println("Worker "+ result.getWorkerName() + 
                             " slept " + result.getSleepSec() + 
                             " sec. Result = " + result.getResult());
        }
      } catch (Exception ex) {
        System.out.println("Caught while getting result: " 
                           + ex.getClass().getName());
      }
    }
  }
}
```

为了获得结果，我们再次使用了带有超时设置的`get()`方法的一个版本。运行以下代码：

```java
List<CallableWorker<Result>> callables = createListOfCallables(1);
System.out.println("Executors.newSingleThreadExecutor():");
ExecutorService execService = Executors.newSingleThreadExecutor();
invokeAllCallables(execService, 1, callables);

```

它的输出将如下：

![](img/19552493-b1f0-4f37-965a-adb41534e80e.png)

值得一提的是，三个工作线程的睡眠时间分别为一秒、两秒和三秒，而服务关闭前的等待时间为一秒。这就是为什么所有工作线程都被取消的原因。

现在，如果我们将等待时间设置为六秒，单线程执行程序的输出将如下所示：

![](img/64677f6d-f5e8-40bc-9fa2-79fde02c30a8.png)

当然，如果我们再次增加等待时间，所有工作线程将能够完成它们的任务。

由`newCachedThreadPool()`或`newFixedThreadPool()`产生的`ExecutorService`接口在单核计算机上表现得更好：

![](img/368e2984-b863-4348-944f-43a6c747de12.png)

正如您所看到的，即使等待时间为三秒，所有线程也能够完成。

作为替代方案，您可以在服务关闭期间设置超时，也可以在`invokeAll()`方法的重载版本上设置超时：

```java
List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                          long timeout, TimeUnit unit)
```

`invokeAll()`方法的一个特定方面经常被忽视，这会给初次使用者带来惊喜：它只有在所有任务完成（正常或通过抛出异常）后才返回。阅读 Javadoc 并进行实验，直到您认识到这种行为对您的应用程序是可以接受的。

相比之下，`invokeAny()`方法只会阻塞，直到至少有一个任务完成

“成功完成（没有抛出异常），如果有的话。在正常或异常返回时，未完成的任务将被取消”

上述引用来自 Javadoc（[`docs.oracle.com/javase/7/docs/api/java/util/concurrent/ExecutorService.html`](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ExecutorService.html)）。以下是执行此操作的代码示例：

```java
void invokeAnyCallables(ExecutorService execService, 
        int shutdownDelaySec, List<CallableWorker<Result>> callables) {
  Result result = null;
  try {
    result = execService.invokeAny(callables, shutdownDelaySec,                                    TimeUnit.SECONDS);
  } catch (Exception ex) {
    System.out.println("Caught around execService.invokeAny(): " 
                       + ex.getClass().getName());
  }
  shutdownAndCancelTasks(execService, shutdownDelaySec,
                         new ArrayList<>());
  if (result == null) {
    System.out.println("No result from execService.invokeAny()");
  } else {
    System.out.println("Worker " + result.getWorkerName() + 
                       " slept " + result.getSleepSec() + 
                       " sec. Result = " + result.getResult());
  }
}
```

您可以尝试设置等待时间（`shutdownDelaySec`）和线程的睡眠时间的不同值，直到您对此方法的行为感到满意为止。正如您所看到的，我们通过传递一个空的`Future`对象列表来重用了`shutdownAndCancelTasks()`方法，因为我们这里没有这些对象。

# 还有更多...

`Executors`类中还有两个静态工厂方法，用于创建`ExecutorService`的实例：

+   `newWorkStealingPool()`: 使用可用处理器的数量作为目标并行级别创建工作窃取线程池。它有一个带有并行级别参数的重载版本。

+   `unconfigurableExecutorService(ExecutorService executor)`: 返回一个对象，该对象将所有已定义的`ExecutorService`方法委托给给定的执行程序，除了可能使用转换访问的那些方法。

此外，`ExecutorService`接口的子接口`ScheduledExecutorService`通过增强 API 的能力来在将来调度线程执行和/或它们的周期性执行。

`ScheduledExecutorService`的对象可以使用`java.util.concurrent.Executors`类的静态工厂方法来创建：

+   `newSingleThreadScheduledExecutor()`: 创建一个可以在给定延迟后调度命令运行或定期执行命令的单线程执行程序。它有一个带有`ThreadFactory`参数的重载版本。

+   `newScheduledThreadPool(int corePoolSize)`: 创建一个可以在给定延迟后调度命令运行或定期执行命令的线程池。它有一个带有`ThreadFactory`参数的重载版本。

+   `unconfigurableScheduledExecutorService( ScheduledExecutorService executor )`: 返回一个对象，该对象将所有已定义的`ScheduledExecutorService`方法委托给给定的执行程序，但不包括可能使用转换访问的其他方法。

`Executors`类还有几个重载方法，接受、执行和返回`Callable`（与`Runnable`相反，它包含结果）。

`java.util.concurrent`包还包括实现`ExecutorService`的类：

+   `ThreadPoolExecutor`：这个类使用几个池化线程中的一个来执行每个提交的任务，通常使用`Executors`工厂方法进行配置。

+   `ScheduledThreadPoolExecutor`：这个类扩展了`ThreadPoolExecutor`类，并实现了`ScheduledExecutorService`接口。

+   `ForkJoinPool`：它使用工作窃取算法管理工作者（`ForkJoinTask`进程）的执行。我们将在下一个示例中讨论它。

这些类的实例可以通过接受更多参数的类构造函数创建，包括保存结果的队列，以提供更精细的线程池管理。

# 使用 fork/join 实现分而治之

在这个示例中，您将学习如何使用 fork/join 框架来进行分而治之的计算模式。

# 准备工作

如前一示例中所述，`ForkJoinPool`类是`ExecutorService`接口的实现，使用工作窃取算法管理工作者（`ForkJoinTask`进程）的执行。如果有多个处理器可用，它会充分利用，并且最适合可以递归地分解为更小任务的任务，这也被称为**分而治之**策略。

池中的每个线程都有一个专用的双端队列（deque）来存储任务，线程在当前任务完成后立即从队列头部获取下一个任务。当另一个线程执行完其队列中的所有任务时，它可以从另一个线程的非空队列尾部获取任务（窃取）。

与任何`ExecutorService`实现一样，fork/join 框架将任务分配给线程池中的工作者线程。这个框架是独特的，因为它使用工作窃取算法。运行完任务的工作者线程可以从仍在忙碌的其他线程中窃取任务。

这样的设计可以平衡负载，并有效利用资源。

为了演示目的，我们将使用第三章中创建的 API，*模块化编程*，`TrafficUnit`，`SpeedModel`和`Vehicle`接口以及`TrafficUnitWrapper`，`FactoryTraffic`，`FactoryVehicle`和`FactorySpeedModel`类。我们还将依赖于第三章中描述的流和流管道，*模块化编程*。

为了提醒您，这是`TrafficUnitWrapper`类：

```java
class TrafficUnitWrapper {
  private double speed;
  private Vehicle vehicle;
  private TrafficUnit trafficUnit;
  public TrafficUnitWrapper(TrafficUnit trafficUnit){
    this.trafficUnit = trafficUnit;
    this.vehicle = FactoryVehicle.build(trafficUnit);
  }
  public TrafficUnitWrapper setSpeedModel(SpeedModel speedModel) {
    this.vehicle.setSpeedModel(speedModel);
    return this;
  }
  TrafficUnit getTrafficUnit(){ return this.trafficUnit;}
  public double getSpeed() { return speed; }

  public TrafficUnitWrapper calcSpeed(double timeSec) {
    double speed = this.vehicle.getSpeedMph(timeSec);
    this.speed = Math.round(speed * this.trafficUnit.getTraction());
    return this;
  }
}
```

我们还将稍微修改现有的 API 接口，并通过引入一个新的`DateLocation`类使其更加紧凑：

```java
class DateLocation {
  private int hour;
  private Month month;
  private DayOfWeek dayOfWeek;
  private String country, city, trafficLight;

  public DateLocation(Month month, DayOfWeek dayOfWeek, 
                      int hour, String country, String city, 
                      String trafficLight) {
    this.hour = hour;
    this.month = month;
    this.dayOfWeek = dayOfWeek;
    this.country = country;
    this.city = city;
    this.trafficLight = trafficLight;
  }
  public int getHour() { return hour; }
  public Month getMonth() { return month; }
  public DayOfWeek getDayOfWeek() { return dayOfWeek; }
  public String getCountry() { return country; }
  public String getCity() { return city; }
  public String getTrafficLight() { return trafficLight;}
}
```

它还将帮助您隐藏细节，并帮助您看到这个示例的重要方面。

# 如何做到...

所有计算都封装在`ForkJoinTask`类的两个子类（`RecursiveAction`或`RecursiveTask<T>`）的子类中。您可以扩展`RecursiveAction`（并实现`void compute()`方法）或`RecursiveTask<T>`（并实现`T compute()`方法）。正如您可能已经注意到的，您可以选择扩展`RecursiveAction`类以处理不返回任何值的任务，并在需要任务返回值时扩展`RecursiveTask<T>`。在我们的示例中，我们将使用后者，因为它稍微复杂一些。

假设我们想要计算特定位置在特定日期和时间以及驾驶条件下的交通平均速度（所有这些参数由`DateLocation`属性对象定义）。其他参数如下：

+   `timeSec`：车辆在交通灯停止后加速的秒数

+   `trafficUnitsNumber`：包括在平均速度计算中的车辆数量

自然地，包括在计算中的车辆数量越多，预测就越准确。但随着这个数字的增加，计算的数量也会增加。这就需要将车辆数量分成更小的组，并与其他组并行计算每组的平均速度。然而，有一定数量的计算是不值得分配给两个线程的。Javadoc（[`docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinTask.html`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinTask.html)）对此有以下说明：

“作为一个非常粗略的经验法则，一个任务应该执行超过 100 个，少于 10000 个基本计算步骤，并且应该避免无限循环。如果任务太大，那么并行性无法提高吞吐量。如果太小，那么内存和内部任务维护开销可能会压倒处理。”

然而，一如既往，确定不在并行线程之间分割计算的最佳数量应该基于测试。这就是为什么我们建议您将其作为参数传递。我们将称此参数为`threshold`。请注意，它还用作递归退出的标准。

我们将称我们的类（任务）为`AverageSpeed`，并扩展`RecursiveTask<Double>`，因为我们希望将平均速度值作为结果的`double`类型：

```java
class AverageSpeed extends RecursiveTask<Double> {
  private double timeSec;
  private DateLocation dateLocation;
  private int threshold, trafficUnitsNumber;
  public AverageSpeed(DateLocation dateLocation, 
                      double timeSec, int trafficUnitsNumber, 
                      int threshold) {
    this.timeSec = timeSec;
    this.threshold = threshold;
    this.dateLocation = dateLocation;
    this.trafficUnitsNumber = trafficUnitsNumber;
  }
  protected Double compute() {
    if (trafficUnitsNumber < threshold) {
      //... write the code here that calculates
      //... average speed trafficUnitsNumber vehicles
      return averageSpeed;
    } else{
      int tun = trafficUnitsNumber / 2;
      //write the code that creates two tasks, each
      //for calculating average speed of tun vehicles 
      //then calculates an average of the two results
      double avrgSpeed1 = ...;
      double avrgSpeed2 = ...;
      return (double) Math.round((avrgSpeed1 + avrgSpeed2) / 2);
    }
  }
}
```

在完成`compute()`方法的编写之前，让我们编写将执行此任务的代码。有几种方法可以做到这一点。例如，我们可以使用`fork()`和`join()`：

```java
void demo1_ForkJoin_fork_join() {
  AverageSpeed averageSpeed = createTask();
  averageSpeed.fork();  
  double result = averageSpeed.join();
  System.out.println("result = " + result);
}
```

这种技术为框架提供了名称。根据 Javadoc，`fork()`方法

“安排异步执行此任务，该任务在池中运行，如果适用，或者如果不在`ForkJoinPool()`中，则使用`ForkJoinPool.commonPool()`。”

在我们的情况下，我们还没有使用任何池，因此`fork()`将默认使用`ForkJoinPool.commonPool()`。它将任务放入池中的线程的队列中。`join()`方法在计算完成时返回计算结果。

`createTask()`方法包含以下内容：

```java
AverageSpeed createTask() {
  DateLocation dateLocation = new DateLocation(Month.APRIL, 
        DayOfWeek.FRIDAY, 17, "USA", "Denver", "Main103S");
  double timeSec = 10d;
  int trafficUnitsNumber = 1001;
  int threshold = 100;
  return new AverageSpeed(dateLocation, timeSec, 
                          trafficUnitsNumber, threshold);
}
```

注意`trafficUnitsNumber`和`threshold`参数的值。这对于分析结果非常重要。

实现这一点的另一种方法是使用`execute()`或`submit()`方法中的任一种——每种方法都提供相同的功能——用于执行任务。执行的结果可以通过`join()`方法检索（与前面的示例相同）：

```java
void demo2_ForkJoin_execute_join() {
  AverageSpeed averageSpeed = createTask();
  ForkJoinPool commonPool = ForkJoinPool.commonPool();
  commonPool.execute(averageSpeed);
  double result = averageSpeed.join();
  System.out.println("result = " + result);
}
```

我们将要审查的最后一种方法是`invoke()`，它相当于调用`fork()`方法，然后调用`join()`方法：

```java
void demo3_ForkJoin_invoke() {
  AverageSpeed averageSpeed = createTask();
  ForkJoinPool commonPool = ForkJoinPool.commonPool();
  double result = commonPool.invoke(averageSpeed);
  System.out.println("result = " + result);
}
```

当然，这是开始分治过程的最流行的方法。

现在让我们回到`compute()`方法，看看它如何实现。首先，让我们实现`if`块（计算少于`threshold`车辆的平均速度）。我们将使用我们在第三章中描述的技术和代码，*模块化编程*：

```java
double speed = 
    FactoryTraffic.getTrafficUnitStream(dateLocation, 
                                                trafficUnitsNumber)
        .map(TrafficUnitWrapper::new)
        .map(tuw -> tuw.setSpeedModel(FactorySpeedModel.
                         generateSpeedModel(tuw.getTrafficUnit())))
        .map(tuw -> tuw.calcSpeed(timeSec))
        .mapToDouble(TrafficUnitWrapper::getSpeed)
        .average()
        .getAsDouble();
System.out.println("speed(" + trafficUnitsNumber + ") = " + speed);
return (double) Math.round(speed);

```

我们从`FactoryTraffic`获取了车辆的`trafficUnitsNumber`。我们为每个发射的元素创建一个`TrafficUnitWrapper`对象，并在其上调用`setSpeedModel()`方法（通过传入基于发射的`TrafficUnit`对象生成的新生成的`SpeedModel`对象）。然后我们计算速度，得到流中所有速度的平均值，并从`Optional`对象（`average()`操作的返回类型）中得到结果作为`double`。然后我们打印出结果并四舍五入以获得更具有代表性的格式。

也可以使用传统的`for`循环来实现相同的结果。但是，如前所述，似乎 Java 遵循了更流畅和类似流的风格的总体趋势，旨在处理大量数据。因此，我们建议您习惯于使用它。

在第十四章中，*测试*，您将看到相同功能的另一个版本，它允许更好地单独测试每个步骤，这再次支持单元测试，以及编写代码，帮助您使代码更具可测试性，并减少以后重写代码的需要。

现在，让我们回顾`else`块实现的选项。前几行总是相同的：

```java
int tun = trafficUnitsNumber / 2;
System.out.println("tun = " + tun);
AverageSpeed as1 = 
   new AverageSpeed(dateLocation, timeSec, tun, threshold);
AverageSpeed as2 = 
   new AverageSpeed(dateLocation, timeSec, tun, threshold);

```

我们将`trafficUnitsNumber`的数字除以 2（我们不担心在大型集合的平均值中可能丢失一个单位），并创建两个任务。

接下来-实际任务执行代码-可以用几种不同的方式编写。这是我们已经熟悉的第一个可能的解决方案，首先想到的：

```java
as1.fork();                //add to the queue
double res1 = as1.join();  //wait until completed
as2.fork();
double res2 = as2.join();
return (double) Math.round((res1 + res2) / 2);

```

运行以下代码：

```java
demo1_ForkJoin_fork_join();
demo2_ForkJoin_execute_join();
demo3_ForkJoin_invoke();

```

如果我们这样做，我们将看到相同的输出（但速度值不同）三次：

![](img/44998580-49bf-4d7f-8e32-13e1151df68b.png)

您可以看到，首先将计算 1,001 个单位（车辆）的平均速度的原始任务分成 2 部分，直到一组的数量（62）降到 100 以下的阈值。然后，计算最后两组的平均速度，并将其与其他组的结果合并。

实现`compute()`方法的`else`块的另一种方法可能如下：

```java
as1.fork();                   //add to the queue
double res1 = as2.compute();  //get the result recursively
double res2 = as1.join();     //wait until the queued task ends
return (double) Math.round((res1 + res2) / 2);

```

结果将如下所示：

![](img/12e0f057-8665-4758-88df-ab8166d67dfd.png)

您可以看到，在这种情况下，`compute()`方法（第二个任务的）被递归调用多次，直到达到元素数量的阈值，然后其结果与对第一个任务调用`fork()`和`join()`方法的结果合并。

如前所述，所有这些复杂性都可以通过调用`invoke()`方法来替换：

```java
double res1 = as1.invoke();
double res2 = as2.invoke();
return (double) Math.round((res1 + res2) / 2);

```

它产生的结果类似于对每个任务调用`fork()`和`join()`产生的结果：

![](img/5b63f37b-25bb-4baf-aa17-144ac7bc60a2.png)

然而，实现`compute()`方法的`else`块的更好的方法是：

```java
return ForkJoinTask.invokeAll(List.of(as1, as2))
        .stream()
        .mapToDouble(ForkJoinTask::join)
        .map(Math::round)
        .average()
        .getAsDouble();

```

如果这对您来说看起来很复杂，只需注意这只是一种类似流的方式来迭代`invokeAll()`的结果：

```java
<T extends ForkJoinTask> Collection<T> invokeAll(Collection<T> tasks)
```

还可以迭代对每个返回任务调用`join()`的结果（并将结果合并为平均值）。优点是我们让框架决定如何优化负载分配。结果如下：

![](img/5cd22de5-19dd-45f9-a868-186d9770a80b.png)

您可以看到它与之前的任何结果都不同，并且可以根据计算机上 CPU 的可用性和负载而改变。

# 使用流来实现发布-订阅模式

在这个示例中，您将了解 Java 9 中引入的新的发布-订阅功能。

# 准备好了

除了许多其他功能，Java 9 还在`java.util.concurrent.Flow`类中引入了这四个接口：

```java
Flow.Publisher<T> - producer of items (messages) of type T
Flow.Subscriber<T> - receiver of messages of type T
Flow.Subscription - links producer and receiver
Flow.Processor<T,R> - acts as both producer and receiver
```

通过这种方式，Java 步入了响应式编程的世界-使用数据流的异步处理编程。

我们在第三章中讨论了流，*模块化编程*，并指出它们不是数据结构，因为它们不会在内存中保存数据。流管道在发出元素之前不会执行任何操作。这种模型允许最小的资源分配，并且只在需要时使用资源。应用程序对其所反应的数据的出现做出*响应*，因此得名。

在发布-订阅模式中，主要的两个角色是`Publisher`，它流式传输数据（发布），以及`Subscriber`，它监听数据（订阅）。

`Flow.Publisher<T>`接口是一个函数式接口。它只有一个抽象方法：

```java
void subscribe(Flow.Subscriber<? super T> subscriber)

```

根据 Javadoc（[`docs.oracle.com/javase/10/docs/api/java/util/concurrent/SubmissionPublisher.html`](https://docs.oracle.com/javase/10/docs/api/java/util/concurrent/SubmissionPublisher.html)），这个方法，

“如果可能，添加给定的`Flow.Subscriber<T>`。如果已经订阅，或者订阅失败，则使用`IllegalStateException`调用`Flow.Subscriber<T>`的`onError()`方法。否则，使用新的`Flow.Subscription`调用`Flow.Subscriber<T>`的`onSubscribe()`方法。订阅者可以通过调用此`Flow.Subscription`的`request()`方法启用接收项目，并可以通过调用其`cancel()`方法取消订阅。”

`Flow.Subscriber<T>`接口有四个方法：

+   `void onSubscribe(Flow.Subscription subscription)`: 在给定`Subscription`的其他`Subscriber`方法之前调用

+   `void onError(Throwable throwable)`: 在`Publisher`或`Subscription`遇到不可恢复的错误后调用，之后`Subscription`不会再调用其他`Subscriber`方法

+   `void onNext(T item)`: 调用`Subscription`的下一个项目

+   `void onComplete()`: 当已知对于`Subscription`不会再发生额外的`Subscriber`方法调用时调用

`Flow.Subscription`接口有两个方法：

+   `void cancel()`: 导致`Subscriber`（最终）停止接收消息

+   `void request(long n)`: 将给定的*n*数量的项目添加到此订阅的当前未满足的需求中

`Flow.Processor<T,R>`接口超出了本书的范围。

# 如何做...

为了节省时间和空间，我们可以使用`java.util.concurrent`包中的`SubmissionPublisher<T>`类，而不是创建自己的`Flow.Publisher<T>`接口的实现。但是，我们将创建自己的`Flow.Subscriber<T>`接口的实现：

```java
class DemoSubscriber<T> implements Flow.Subscriber<T> {
  private String name;
  private Flow.Subscription subscription;
  public DemoSubscriber(String name){ this.name = name; }
  public void onSubscribe(Flow.Subscription subscription) {
    this.subscription = subscription;
    this.subscription.request(0);
  }
  public void onNext(T item) {
    System.out.println(name + " received: " + item);
    this.subscription.request(1);
  }
  public void onError(Throwable ex){ ex.printStackTrace();}
  public void onComplete() { System.out.println("Completed"); }
}
```

我们还将实现`Flow.Subscription`接口：

```java
class DemoSubscription<T> implements Flow.Subscription {
  private final Flow.Subscriber<T> subscriber;
  private final ExecutorService executor;
  private Future<?> future;
  private T item;
  public DemoSubscription(Flow.Subscriber subscriber,
                          ExecutorService executor) {
    this.subscriber = subscriber;
    this.executor = executor;
  }
  public void request(long n) {
    future = executor.submit(() -> {
      this.subscriber.onNext(item );
    });
  }
  public synchronized void cancel() {
    if (future != null && !future.isCancelled()) {
      this.future.cancel(true);
    }
  }
}
```

正如您所看到的，我们只是遵循了 Javadoc 的建议，并期望当订阅者添加到发布者时，将调用订阅者的`onSubscribe()`方法。

还有一个要注意的细节是，`SubmissionPublisher<T>`类具有`submit(T item)`方法，根据 Javadoc（[`docs.oracle.com/javase/10/docs/api/java/util/concurrent/SubmissionPublisher.html`](https://docs.oracle.com/javase/10/docs/api/java/util/concurrent/SubmissionPublisher.html)）：

“通过异步调用其`onNext()`方法将给定项目发布给每个当前订阅者，同时在任何订阅者资源不可用时阻塞不间断地。”

这样，`SubmissionPublisher<T>`类将项目提交给当前订阅者，直到关闭。这允许项目生成器充当反应式流发布者。

为了演示这一点，让我们使用`demoSubscribe()`方法创建几个订阅者和订阅： 

```java
void demoSubscribe(SubmissionPublisher<Integer> publisher, 
        ExecutorService execService, String subscriberName){
  DemoSubscriber<Integer> subscriber = 
                     new DemoSubscriber<>(subscriberName);
  DemoSubscription subscription = 
            new DemoSubscription(subscriber, execService);
  subscriber.onSubscribe(subscription);
  publisher.subscribe(subscriber);
}
```

然后在以下代码中使用它们：

```java
ExecutorService execService =  ForkJoinPool.commonPool();
try (SubmissionPublisher<Integer> publisher = 
                            new SubmissionPublisher<>()){
  demoSubscribe(publisher, execService, "One");
  demoSubscribe(publisher, execService, "Two");
  demoSubscribe(publisher, execService, "Three");
  IntStream.range(1, 5).forEach(publisher::submit);
} finally {
  //...make sure that execService is shut down
}
```

上述代码创建了三个订阅者，连接到具有专用订阅的相同发布者。最后一行生成了一个数字流，1、2、3 和 4，并将每个数字提交给发布者。我们期望每个订阅者都会将生成的每个数字作为`onNext()`方法的参数。

在`finally`块中，我们包含了您已经熟悉的代码，来自上一个示例：

```java
try {
  execService.shutdown();
  int shutdownDelaySec = 1;
  System.out.println("Waiting for " + shutdownDelaySec 
                           + " sec before shutting down service...");
  execService.awaitTermination(shutdownDelaySec, TimeUnit.SECONDS);
} catch (Exception ex) {
  System.out.println("Caught around execService.awaitTermination(): " 
                                          + ex.getClass().getName());
} finally {
  System.out.println("Calling execService.shutdownNow()...");
  List<Runnable> l = execService.shutdownNow();
  System.out.println(l.size() 
            +" tasks were waiting to be executed. Service stopped.");
}
```

如果我们运行上述代码，输出可能如下所示：

![](img/a9ef8e7a-c418-44c6-a0bc-2aa9a68eb56f.png)

正如您所看到的，由于异步处理，控制非常快地到达`finally`块，并在关闭服务之前等待一秒钟。这段等待时间足够生成项目并将其传递给订阅者。我们还确认每个生成的项目都发送给了每个订阅者。每次调用每个订阅者的`onSubscribe()`方法时，都会生成三个`null`值。

可以合理地期望，在未来的 Java 版本中，将会为反应式（异步和非阻塞）功能增加更多支持。
