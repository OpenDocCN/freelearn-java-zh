# 第十章：并发处理

在本章中，我们将涵盖以下内容：

+   在 Java 7 中使用 join/fork 框架

+   使用可重用的同步障碍 Phaser

+   在多个线程中安全地使用 ConcurrentLinkedDeque 类

+   使用 LinkedTransferQueue 类

+   使用 ThreadLocalRandom 类支持多个线程

# 介绍

Java 7 中改进了并发应用程序的支持。引入了几个新类，支持任务的并行执行。`ForkJoinPool`类用于使用分而治之技术解决问题的应用程序。每个子问题都被分叉（分割）为一个单独的线程，然后在必要时合并以提供解决方案。该类使用的线程通常是`java.util.concurrent.ForkJoinTask`类的子类，是轻量级线程。*在 Java 中使用 join/fork 框架*示例中说明了这种方法的使用。

此外，引入了`java.util.concurrent.Phaser`类，以支持一系列阶段中线程集合的执行。一组线程被同步，以便它们都执行然后等待其他线程的完成。一旦它们都完成了，它们可以重新执行第二阶段或后续阶段。*使用可重用的同步障碍 Phaser*示例说明了在游戏引擎设置中使用此类的情况。

*使用 java.util.concurrent.ConcurrentLinkedDeque 类安全地与多个线程一起使用*和*使用 java.util.concurrent.LinkedTransferQueue 类*示例介绍了两个设计用于安全地与多个线程一起工作的新类。展示了它们在支持生产者/消费者框架的使用示例。

`java.util.concurrent.ThreadLocalRandom`类是新的，并提供更好地支持在多个线程之间使用的随机数生成。在*使用 ThreadLocalRandom 类支持多个线程*示例中进行了讨论。

`java.util.ConcurrentModificationException`类中添加了两个新的构造函数。它们都接受一个`Throwable`对象，用于指定异常的原因。其中一个构造函数还接受一个提供有关异常的详细信息的字符串。

Java 7 通过修改锁定机制改进了类加载器的使用，以避免死锁。在 Java 7 之前的多线程自定义类加载器中，某些自定义类加载器在使用循环委托模型时容易发生死锁。

考虑以下情景。Thread1 尝试使用 ClassLoader1（锁定 ClassLoader1）加载 class1。然后将加载 class2 的委托给 ClassLoader2。与此同时，Thread2 使用 ClassLoader2（锁定 ClassLoader2）加载 class3，然后将加载 class4 的委托给 ClassLoader1。由于两个类加载器都被锁定，而两个线程都需要这两个加载器，因此发生死锁情况。

并发类加载器的期望行为是从同一实例的类加载器并发加载不同的类。这需要以更细粒度的级别进行锁定，例如通过正在加载的类的名称锁定类加载器。

同步不应该在类加载器级别进行。相反，应该在类级别上进行锁定，其中类加载器只允许由该类加载器一次加载一个类的单个实例。

一些类加载器能够并发加载类。这种类型的类加载器称为**并行可用的类加载器**。它们在初始化过程中需要使用`registerAsParallelCapable`方法进行注册。

如果自定义类加载器使用无环层次委托模型，则在 Java 中不需要进行任何更改。在层次委托模型中，首先委托给其父类加载器。不使用层次委托模型的类加载器应该在 Java 中构造为并行可用的类加载器。

为自定义类加载器避免死锁：

+   在类初始化序列中使用`registerAsParallelCapable`方法。这表示类加载器的所有实例都是多线程安全的。

+   确保类加载器代码是多线程安全的。这包括：

+   使用内部锁定方案，例如`java.lang.ClassLoader`使用的类名锁定方案

+   删除类加载器锁上的任何同步

+   确保关键部分是多线程安全的

+   建议类加载器覆盖`findClass(String)`方法

+   如果`defineClass`方法被覆盖，则确保每个类名只调用一次

有关此问题的更多详细信息，请访问[`openjdk.java.net/groups/core-libs/ClassLoaderProposal.html`](http://openjdk.java.net/groups/core-libs/ClassLoaderProposal.html)。

# 在 Java 中使用 join/fork 框架

**join/fork**框架是一种支持将问题分解为更小的部分，以并行方式解决它们，然后将结果合并的方法。新的`java.util.concurrent.ForkJoinPool`类支持这种方法。它旨在与多核系统一起工作，理想情况下有数十个或数百个处理器。目前，很少有桌面平台支持这种并发性，但未来的机器将会支持。少于四个处理器时，性能改进将很小。

`ForkJoinPool`类源自`java.util.concurrent.AbstractExecutorService`，使其成为`ExecutorService`。它旨在与`ForkJoinTasks`一起工作，尽管它也可以与普通线程一起使用。`ForkJoinPool`类与其他执行程序不同，其线程尝试查找并执行其他当前运行任务创建的子任务。这称为**工作窃取**。

`ForkJoinPool`类可用于计算子问题上的计算要么被修改，要么返回一个值。当返回一个值时，使用`java.util.concurrent.RecursiveTask`派生类。否则，使用`java.util.concurrent.RecursiveAction`类。在本教程中，我们将说明使用`RecursiveTask`派生类的用法。

## 准备工作

要为返回每个子任务结果的任务使用分支/合并框架：

1.  创建一个实现所需计算的`RecursiveTask`的子类。

1.  创建`ForkJoinPool`类的实例。

1.  使用`ForkJoinPool`类的`invoke`方法与`RecursiveTask`类的子类的实例。

## 如何做...

该应用程序并非旨在以最有效的方式实现，而是用于说明分支/合并任务。因此，在处理器数量较少的系统上，可能几乎没有性能改进。

1.  创建一个新的控制台应用程序。我们将使用一个派生自`RecursiveTask`的静态内部类来计算`numbers`数组中整数的平方和。首先，声明`numbers`数组如下：

```java
private static int numbers[] = new int[100000];

```

1.  如下添加`SumOfSquaresTask`类。它创建数组元素的子范围，并使用迭代循环计算它们的平方和，或者根据阈值大小将数组分成更小的部分：

```java
private static class SumOfSquaresTask extends RecursiveTask<Long> {
private final int thresholdTHRESHOLD = 1000;
private int from;
private int to;
public SumOfSquaresTask(int from, int to) {
this.from = from;
this.to = to;
}
@Override
protected Long compute() {
long sum = 0L;
int mid = (to + from) >>> 1;
if ((to - from) < thresholdTHRESHOLD) {
for (int i = from; i < to; i++) {
sum += numbers[i] * numbers[i];
}
return sum;
}
else {
List<RecursiveTask<Long>> forks = new ArrayList<>();
SumOfSquaresTask task1 =
new SumOfSquaresTask(from, mid);
SumOfSquaresTask task2 =
new SumOfSquaresTask(mid, to);
forks.add(task1);
task1.fork();
forks.add(task2);
task2.fork();
for (RecursiveTask<Long> task : forks) {
sum += task.join();
}
return sum;
}
}
}

```

1.  添加以下`main`方法。为了比较，使用 for 循环计算平方和，然后使用`ForkJoinPool`类。执行时间如下计算并显示：

```java
public static void main(String[] args) {
for (int i = 0; i < numbers.length; i++) {
numbers[i] = i;
}
long startTime;
long stopTime;
long sum = 0L;
startTime = System.currentTimeMillis();
for (int i = 0; i < numbers.length; i++) {
sum += numbers[i] * numbers[i];
}
System.out.println("Sum of squares: " + sum);
stopTime = System.currentTimeMillis();
System.out.println("Iterative solution time: " + (stopTime - startTime));
ForkJoinPool forkJoinPool = new ForkJoinPool();
startTime = System.currentTimeMillis();
long result = forkJoinPool.invoke(new SumOfSquaresTask(0, numbers.length));
System.out.println("forkJoinPool: " + forkJoinPool.toString());
stopTime = System.currentTimeMillis();
System.out.println("Sum of squares: " + result);
System.out.println("Fork/join solution time: " + (stopTime - startTime));
}

```

1.  执行应用程序。您的输出应该类似于以下内容。但是，根据您的硬件配置，您应该观察到不同的执行时间：

**平方和：18103503627376**

**迭代解决方案时间：5**

**平方和：18103503627376**

**分支/合并解决方案时间：23**

请注意，迭代解决方案比使用分支/合并策略的解决方案更快。如前所述，除非有大量处理器，否则这种方法并不总是更有效。

重复运行应用程序将导致不同的结果。更积极的测试方法是在可能不同的处理器负载条件下重复执行解决方案，然后取结果的平均值。阈值的大小也会影响其性能。

## 工作原理...

`numbers`数组声明为一个包含 100,000 个元素的整数数组。`SumOfSquaresTask`类是从`RecursiveTask`类派生的，使用了泛型类型`Long`。设置了阈值为 1000。任何小于此阈值的子数组都使用迭代解决。否则，该段被分成两半，并创建了两个新任务，每个任务处理一半。

`ArrayList`用于保存两个子任务。这严格来说是不需要的，实际上会减慢计算速度。但是，如果我们决定将数组分成两个以上的段，这将是有用的。它提供了一个方便的方法，在子任务加入时重新组合元素。

`fork`方法用于拆分子任务。它们进入线程池，最终将被执行。`join`方法在子任务完成时返回结果。然后将子任务的总和相加并返回。

在`main`方法中，第一个代码段使用`for`循环计算了平方的和。开始和结束时间基于以毫秒为单位测量的当前时间。第二段创建了`ForkJoinPool`类的一个实例，然后使用其`invoke`方法与`SumOfSquaresTask`对象的新实例。传递给`SumOfSquaresTask`构造函数的参数指示它从数组的第一个元素开始，直到最后一个元素。完成后，显示执行时间。

## 还有更多...

`ForkJoinPool`类有几种报告池状态的方法，包括：

+   `getPoolSize:`该方法返回已启动但尚未完成的线程数

+   `getRunningThreadCount:`该方法返回未被阻塞但正在等待加入其他任务的线程数的估计值

+   `getActiveThreadCount:`该方法返回执行任务的线程数的估计值

`ForkJoinPool`类的`toString`方法返回池的几个方面。在`invoke`方法执行后立即添加以下语句：

```java
out.println("forkJoinPool: " + forkJoinPool);

```

当程序执行时，将获得类似以下的输出：

**forkJoinPool: java.util.concurrent.ForkJoinPool@18fb53f6[Running, parallelism = 4, size = 55, active = 0, running = 0, steals = 171, tasks = 0, submissions = 0]**

## 另请参阅

*使用可重用同步障碍`Phaser`*的方法提供了执行多个线程的不同方法。

# 使用可重用的同步障碍`Phaser`

`java.util.concurrent.Phaser`类涉及协调一起工作的线程在循环类型阶段中的同步。线程将执行，然后等待组中其他线程的完成。当所有线程都完成时，一个阶段就完成了。然后可以使用`Phaser`来协调再次执行相同一组线程。

`java.util.concurrent.CountdownLatch`类提供了一种方法来做到这一点，但需要固定数量的线程，并且默认情况下只执行一次。`java.util.concurrent.CyclicBarrier`，它是在 Java 5 中引入的，也使用了固定数量的线程，但是可重用。但是，不可能进入下一个阶段。当问题以一系列基于某些标准的步骤/阶段进行推进时，这是有用的。

随着 Java 7 中`Phaser`类的引入，我们现在有了一个结合了`CountDownLatch`和`CyclicBarrier`功能并支持动态线程数量的并发抽象。术语“phase”指的是线程可以协调执行不同阶段或步骤的想法。所有线程将执行，然后等待其他线程完成。一旦它们完成，它们将重新开始并完成第二个或后续阶段的操作。

屏障是一种阻止任务继续进行的类型的块，直到满足某些条件。一个常见的条件是当所有相关线程都已完成时。

`Phaser`类提供了几个功能，使其非常有用：

+   可以动态地向线程池中添加和删除参与者

+   每个阶段都有一个唯一的阶段号。

+   `Phaser`可以被终止，导致任何等待的线程立即返回

+   发生的异常不会影响屏障的状态

`register`方法增加了参与的方数量。当内部计数达到零或根据其他条件确定时，屏障终止。

## 准备好了

我们将开发一个模拟游戏引擎操作的应用程序。第一个版本将创建一系列代表游戏中参与者的任务。我们将使用`Phaser`类来协调它们的交互。

使用`Phaser`类来同步一组任务的开始：

1.  创建一个将参与`Phaser`的`Runnable`对象集合。

1.  创建`Phaser`类的一个实例。

1.  对于每个参与者：

+   注册参与者

+   使用参与者的`Runnable`对象创建一个新线程

+   使用`arriveAndAwaitAdvance`方法等待其他任务的创建

+   执行线程

1.  使用`Phaser`对象的`arriveAndDeregister`来启动参与者的执行。

## 如何做...

1.  创建一个名为`GamePhaserExample`的新控制台应用程序类。我们将创建一系列内部类的简单层次结构，这些类代表游戏中的参与者。将`Entity`类添加为基本抽象类，定义如下。虽然不是绝对必要的，但我们将使用继承来简化这些类型应用程序的开发：

```java
private static abstract class Entity implements Runnable {
public abstract void run();
}

```

1.  接下来，我们将创建两个派生类：`Player`和`Zombie`。这些类实现`run`方法和`toString`方法。`run`方法使用`sleep`方法来模拟执行的工作。预期地，僵尸比人类慢：

```java
private static class Player extends Entity {
private final static AtomicInteger idSource = new AtomicInteger();
private final int id = idSource.incrementAndGet();
public void run() {
System.out.println(toString() + " started");
try {
Thread.currentThread().sleep(
ThreadLocalRandom.current().nextInt(200, 600));
}
catch (InterruptedException ex) {
ex.printStackTrace();
}
System.out.println(toString() + " stopped");
}
@Override
public String toString() {
return "Player #" + id;
}
}
private static class Zombie extends Entity {
private final static AtomicInteger idSource = new AtomicInteger();
private final int id = idSource.incrementAndGet();
public void run() {
System.out.println(toString() + " started");
try {
Thread.currentThread().sleep(
ThreadLocalRandom.current().nextInt(400, 800));
}
catch (InterruptedException ex) {
ex.printStackTrace();
}
System.out.println(toString() + " stopped");
}
@Override
public String toString() {
return "Zombie #" + id;
}
}

```

1.  为了使示例更清晰，将以下`main`方法添加到`GamePhaserExample`类中：

```java
public static void main(String[] args) {
new GamePhaserExample().execute();
}

```

1.  接下来，添加以下`execute`方法，我们在其中创建参与者列表，然后调用`gameEngine`方法：

```java
private void execute() {
List<Entity> entities = new ArrayList<>();
entities = new ArrayList<>();
entities.add(new Player());
entities.add(new Zombie());
entities.add(new Zombie());
entities.add(new Zombie());
gameEngine(entities);
}

```

1.  接下来是`gameEngine`方法。`for each`循环为每个参与者创建一个线程：

```java
private void gameEngine(List<Entity> entities) {
final Phaser phaser = new Phaser(1);
for (final Entity entity : entities) {
synchronization barrier Phaserusingfinal String member = entity.toString();
System.out.println(member + " joined the game");
phaser.register();
new Thread() {
@Override
public void run() {
System.out.println(member +
" waiting for the remaining participants");
phaser.arriveAndAwaitAdvance(); // wait for remaining entities
System.out.println(member + " starting run");
entity.run();
}
}.start();
}
phaser.arriveAndDeregister(); //Deregister and continue
System.out.println("Phaser continuing");
}

```

1.  执行应用程序。输出是不确定的，但应该类似于以下内容：

**玩家＃1 加入游戏**

**僵尸＃1 加入游戏**

**僵尸＃2 加入游戏**

**玩家＃1 等待剩余参与者**

**僵尸＃1 等待剩余参与者**

**僵尸＃3 加入游戏**

**Phaser 继续**

**僵尸＃3 等待剩余参与者**

**僵尸＃2 等待剩余参与者**

**僵尸＃1 开始奔跑**

**僵尸＃1 开始**

**僵尸＃3 开始奔跑**

**僵尸＃3 开始**

**僵尸＃2 开始奔跑**

**僵尸＃2 开始**

**玩家＃1 开始奔跑**

**玩家＃1 开始**

**玩家＃1 停止**

**僵尸＃1 停止**

**僵尸＃3 停止**

**僵尸＃2 停止**

注意`Phaser`对象会等待直到所有参与者都加入游戏。

## 它是如何工作的...

`sleep` 方法用于模拟实体所涉及的工作。请注意 `ThreadLocalRandom` 类的使用。其 `nextInt` 方法返回其参数中指定的值之间的随机数。在使用并发线程时，这是生成随机数的首选方式，如*使用 ThreadLocalRandom 类支持多个线程*配方中所述。

`AtomicInteger` 类的一个实例用于为每个创建的对象分配唯一的 ID。这是在线程中生成数字的安全方式。`toString` 方法返回实体的简单字符串表示形式。

在`execute` 方法中，我们创建了一个 `ArrayList` 来保存参与者。请注意在创建 `ArrayList` 时使用了菱形操作符。这是 Java 7 语言改进，在第一章的*使用菱形操作符进行构造类型推断*配方中有解释，*Java 语言改进*。添加了一个玩家和三个僵尸。僵尸似乎总是比人类多。然后调用了 `gameEngine` 方法。

使用参数为一的 `Phaser` 对象创建了一个代表第一个参与者的对象。它不是一个实体，只是作为帮助控制阶段器的机制。

在每个循环中，使用 `register` 方法将阶段器中的方的数量增加一。使用匿名内部类创建了一个新线程。在其 `run` 方法中，直到所有参与者到达之前，实体才会开始。`arriveAndAwaitAdvance` 方法导致通知参与者已到达，并且该方法在所有参与者到达并且阶段完成之前不返回。

在`while`循环的每次迭代开始时，注册参与者的数量比已到达的参与者数量多一个。`register` 方法将内部计数增加一。然后内部计数比已到达的数量多两个。当执行 `arriveAndAwaitAdvance` 方法时，现在等待的参与者数量将比已注册的多一个。

循环结束后，仍然有一个比已到达的参与者多的注册方。但是，当执行 `arriveAndDeregister` 方法时，已到达的参与者数量的内部计数与参与者数量匹配，并且线程开始。此外，注册方的数量减少了一个。当所有线程终止时，应用程序终止。

## 还有更多...

可以使用 `bulkRegister` 方法注册一组方。此方法接受一个整数参数，指定要注册的方的数量。

在某些情况下，可能希望强制终止阶段器。`forceTermination` 方法用于此目的。

在执行阶段器时，有几种方法可以返回有关阶段器状态的信息，如下表所述。如果阶段器已终止，则这些方法将不起作用：

| 方法 | 描述 |
| --- | --- |
| `getRoot` | 返回根阶段器。与阶段器树一起使用 |
| `getParent` | 返回阶段器的父级 |
| `getPhase` | 返回当前阶段编号 |
| `getArrivedParties` | 已到达当前阶段的方的数量 |
| `getRegisteredParties` | 注册方的数量 |
| `getUnarrivedParties` | 尚未到达当前阶段的方的数量 |

可以构建阶段器树，其中阶段器作为任务的分支创建。在这种情况下，`getRoot` 方法非常有用。阶段器构造在[`www.cs.rice.edu/~vs3/PDF/SPSS08-phasers.pdf`](http://www.cs.rice.edu/~vs3/PDF/SPSS08-phasers.pdf)中讨论。

### 使用阶段器重复一系列任务

我们还可以使用`Phaser`类来支持一系列阶段，其中执行任务，执行可能的中间操作，然后再次重复一系列任务。

为了支持这种行为，我们将修改`gameEngine`方法。修改将包括：

+   添加一个`iterations`变量

+   覆盖`Phaser`类的`onAdvance`方法

+   在每个任务的`run`方法中使用`while`循环，由`isTerminated`方法控制

添加一个名为`iterations`的变量，并将其初始化为`3`。这用于指定我们将使用多少个阶段。还要重写如下所示的`onAdvance`方法：

```java
final int iterations = 3;
final Phaser phaser = new Phaser(1) {
protected boolean onAdvance(int phase, int registeredParties) {
System.out.println("Phase number " + phase + " completed\n")
return phase >= iterations-1 || registeredParties == 0;
}
};

```

每个阶段都有唯一的编号，从零开始。调用`onAdvance`传递当前阶段编号和注册到 phaser 的当前参与方数量。当注册方数量变为零时，此方法的默认实现返回`true`。这将导致 phaser 被终止。

该方法的实现导致仅当阶段编号超过`iterations`值（即减 1）或没有使用 phaser 的注册方时，该方法才返回`true`。

根据以下代码中突出显示的内容修改`run`方法：

```java
for (final Entity entity : entities) {
final String member = entity.toString();
System.out.println(member + " joined the game");
phaser.register();
new Thread() {
@Override
public void run() {
do {
System.out.println(member + " starting run");
entity.run();
System.out.println(member +
" waiting for the remaining participants during phase " +
phaser.getPhase());
phaser.arriveAndAwaitAdvance(); // wait for remaining entities
}
while (!phaser.isTerminated());
}
}.start();
}

```

实体被允许先运行，然后等待其他参与者完成和到达。只要通过`isTerminated`方法确定的 phaser 尚未终止，当每个人准备好时，下一阶段将被执行。

最后一步是使用`arriveAndAwaitAdvance`方法将 phaser 推进到下一个阶段。同样，只要 phaser 尚未终止，当每个参与者到达时，phaser 将推进到下一个阶段。使用以下代码序列来完成此操作：

```java
while (!phaser.isTerminated()) {
phaser.arriveAndAwaitAdvance();
}
System.out.println("Phaser continuing");

```

仅使用一个玩家和一个僵尸执行程序。这将减少输出量，并且应与以下内容类似：

**玩家＃1 加入游戏**

**僵尸＃1 加入游戏**

**玩家＃1 开始运行**

**玩家＃1 开始**

**僵尸＃1 开始运行**

**僵尸＃1 开始**

**玩家＃1 停止**

**玩家＃1 在第 0 阶段等待剩余参与者**

**僵尸＃1 停止**

**僵尸＃1 在第 0 阶段等待剩余参与者**

**第 0 阶段完成**

**玩家＃1 开始运行**

**玩家＃1 开始**

**僵尸＃1 开始运行**

**僵尸＃1 开始**

**玩家＃1 停止**

**玩家＃1 在第 1 阶段等待剩余参与者**

**僵尸＃1 停止**

**僵尸＃1 在第 1 阶段等待剩余参与者**

**第 1 阶段完成**

**僵尸＃1 开始运行**

**玩家＃1 开始运行**

**僵尸＃1 开始**

**玩家＃1 开始**

**玩家＃1 停止**

**玩家＃1 在第 2 阶段等待剩余参与者**

**僵尸＃1 停止**

**僵尸＃1 在第 2 阶段等待剩余参与者**

**第 2 阶段完成**

**Phaser 继续**

## 另请参阅

有关为多个线程生成随机数的更多信息，请参阅*使用当前线程隔离的随机数生成器*。

# 安全地使用新的`ConcurrentLinkedDeque`与多个线程

`java.util.concurrent.ConcurrentLinkedDeque`类是 Java 集合框架的成员，它允许多个线程安全地同时访问相同的数据集合。该类实现了一个双端队列，称为**deque**，并允许从 deque 的两端插入和删除元素。它也被称为头尾链接列表，并且与其他并发集合一样，不允许使用空元素。

在本示例中，我们将演示`ConcurrentLinkedDeque`类的基本实现，并说明一些最常用方法的使用。

## 准备好了

在生产者/消费者框架中使用`ConcurrentLinkedDeque`：

1.  创建`ConcurrentLinkedDeque`的实例。

1.  定义要放入双端队列的元素。

1.  实现一个生产者线程来生成要放入双端队列中的元素。

1.  实现一个消费者线程来从双端队列中删除元素。

## 如何做...

1.  创建一个新的控制台应用程序。使用`Item`的泛型类型声明一个私有静态实例的`ConcurrentLinkedDeque`。`Item`类被声明为内部类。包括获取方法和构造函数，如下面的代码所示，使用两个属性`description`和`itemId`：

```java
private static ConcurrentLinkedDeque<Item> deque = new ConcurrentLinkedDeque<>();
static class Item {
privateublic final String description;
privateublic final int itemId;
public Item() {
"this(Default Item";, 0)
}
public Item(String description, int itemId) {
this.description = description;
this.itemId = itemId;
}
}

```

1.  然后创建一个生产者类来生成`Item`类型的元素。为了这个示例的目的，我们只会生成七个项目，然后打印出一个声明来证明该项目已添加到双端队列中。我们使用`ConcurrentLinkedDeque`类的`add`方法来添加元素。每次添加后，线程会短暂休眠：

```java
static class ItemProducer implements Runnable {
@Override
public void run() {
String itemName = "";
int itemId = 0;
try {
for (int x = 1; x < 8; x++) {
itemName = "Item" + x;
itemId = x;
deque.add(new Item(itemName, itemId));
System.out.println("New Item Added:" + itemName + " " + itemId);
Thread.currentThread().sleep(250);
}
}
catch (InterruptedException ex) {
ex.printStackTrace();
}
}
}

```

1.  接下来，创建一个消费者类。为了确保在消费者线程尝试访问它之前，双端队列中将有元素，我们让线程在检索元素之前睡眠一秒钟。然后我们使用`pollFirst`方法来检索双端队列中的第一个元素。如果元素不为空，那么我们将元素传递给`generateOrder`方法。在这个方法中，我们打印有关该项目的信息：

```java
static class ItemConsumer implements Runnable {
@Override
public void run() {
try {
Thread.currentThread().sleep(1000);
}
catch (InterruptedException ex) {
ex.printStackTrace();
}
Item item;
while ((item = deque.pollFirst()) != null) {
{
generateOrder(item);
}
}
private void generateOrder(Item item) {
System.out.println("Part Order");
System.out.println("Item description: " + item.getDescriptiond());
System.out.println("Item ID # " + item.getItemIdi());
System.out.println();
try {
Thread.currentThread().sleep(1000);
}
catch (InterruptedException ex) {
ex.printStackTrace();
}
}
}

```

1.  最后，在我们的`main`方法中，启动两个线程：

```java
public static void main(String[] args) {
new Thread(new ItemProducer());.start()
new Thread(new ItemConsumer());.start()
}

```

1.  当您执行程序时，您应该看到类似以下的输出：

**新项目已添加：Item1 1**

**新项目已添加：Item2 2**

**新项目已添加：Item3 3**

**新项目已添加：Item4 4**

**零件订单**

**项目描述：Item1**

**项目 ID＃1**

**新项目已添加：Item5 5**

**新项目已添加：Item6 6**

**新项目已添加：Item7 7**

**零件订单**

**项目描述：Item2**

**项目 ID＃2**

**零件订单**

**项目描述：Item3**

**项目 ID＃3**

**零件订单**

**项目描述：Item4**

**项目 ID＃4**

**零件订单**

**项目描述：Item5**

**项目 ID＃5**

**零件订单**

**项目描述：Item6**

**项目 ID＃6**

**零件订单**para

**项目描述：Item7**

**项目 ID＃7**

## 它是如何工作的...

当我们启动两个线程时，我们让生产者线程提前一点时间来填充我们的双端队列。一秒钟后，消费者线程开始检索元素。使用`ConcurrentLinkedDeque`类允许两个线程同时安全地访问双端队列的元素。

在我们的示例中，我们使用了`add`和`pollFirst`方法来添加和删除双端队列的元素。有许多可用的方法，其中许多方法基本上以相同的方式运行。*还有更多...*部分提供了有关访问双端队列元素的各种选项的更多详细信息。

## 还有更多...

我们将涵盖几个主题，包括：

+   异步并发线程存在问题

+   向双端队列添加元素

+   从双端队列中检索元素

+   访问双端队列的特定元素

### 异步并发线程存在问题

由于多个线程可能在任何给定时刻访问集合，因此`size`方法并不总是会返回准确的结果。当使用`iterator`或`descendingIterator`方法时，情况也是如此。此外，任何批量数据操作，例如`addAll`或`removeAll`，也不总是会达到预期的结果。如果一个线程正在访问集合中的一个项目，而另一个线程尝试拉取所有项目，则批量操作不能保证以原子方式运行。

有两种`toArray`方法可用于检索双端队列的所有元素并将它们存储在数组中。第一个返回表示双端队列所有元素的对象数组，并且可以转换为适当的数据类型。当双端队列的元素是不同的数据类型时，这是有用的。以下是如何使用`toArray`方法的第一种形式的示例，使用我们之前的线程示例：

```java
Item[] items = (Item[]) deque.toArray();

```

另一个`toArray`方法需要一个特定数据类型的初始化数组作为参数，并返回该数据类型的元素数组。

```java
Item[] items = deque.toArray(new Item[0]);

```

### 向双端队列添加元素

以下表格列出了一些可用于向双端队列中添加元素的方法。在下表中分组在一起的方法本质上执行相同的功能。这种类似方法的多样性是`ConcurrentLinkedDeque`类实现略有不同接口的结果： 

| 方法名 | 添加元素到 |
| --- | --- |
| `add(Element e)``offer(Element e)``offerLast(Element e)``addLast(Element e)` | 双端队列的末尾 |
| `addFirst(Element e)``offerFirst(Element e)``push(Element e)` | 双端队列的前端 |

### 从双端队列中检索元素

以下是一些用于从双端队列中检索元素的方法：

| 方法名 | 错误操作 | 功能 |
| --- | --- | --- |
| `element()` | 如果双端队列为空则抛出异常 | 检索但不移除双端队列的第一个元素 |
| `getFirst()` |   |   |
| `getLast()` |   |   |
| `peek()` | 如果双端队列为空则返回 null |   |
| `peekFirst()` |   |   |
| `peekLast()` |   |   |
| `pop()` | 如果双端队列为空则抛出异常 | 检索并移除双端队列的第一个元素 |
| `removeFirst()` |   |   |
| `poll()` | 如果双端队列为空则返回 null |   |
| `pollFirst()` |   |   |
| `removeLast()` | 如果双端队列为空则抛出异常 | 检索并移除双端队列的最后一个元素 |
| `pollLast()` | 如果双端队列为空则返回 null |   |

### 访问双端队列的特定元素

以下是一些用于访问双端队列特定元素的方法：

| 方法名 | 功能 | 注释 |
| --- | --- | --- |
| `contains(Element e)` | 如果双端队列包含至少一个等于`Element e`的元素则返回`true` |   |
| `remove(Element e)``removeFirstOccurrence(Element e)` | 移除双端队列中第一个等于`Element e`的元素 | 如果元素在双端队列中不存在，则双端队列保持不变。如果`e`为 null 则抛出异常 |
| `removeLastOccurrence(Element e)` | 移除双端队列中最后一个等于`Element e`的元素 |   |

# 使用新的 LinkedTransferQueue 类

`java.util.concurrent.LinkedTransferQueue`类实现了`java.util.concurrent.TransferQueue`接口，是一个无界队列，遵循**先进先出**模型。该类提供了用于检索元素的阻塞方法和非阻塞方法，并且适合于多个线程的并发访问。在本示例中，我们将创建一个`LinkedTransferQueue`的简单实现，并探索该类中的一些可用方法。

## 准备工作

要在生产者/消费者框架中使用`LinkedTransferQueue`：

1.  创建一个`LinkedTransferQueue`的实例。

1.  定义要放入队列的元素类型。

1.  实现一个生产者线程来生成要放入队列的元素。

1.  实现一个消费者线程来从队列中移除元素。

## 如何做...

1.  创建一个新的控制台应用程序。使用`Item`的泛型类型声明一个`LinkedTransferQueue`的私有静态实例。然后创建内部类`Item`，并包括如下代码所示的 get 方法和构造函数，使用`description`和`itemId`这两个属性：

```java
private static LinkedTransferQueue<Item>
linkTransQ = new LinkedTransferQueue<>();
static class Item {
public final String description;
public final int itemId;
public Item() {
this("Default Item", 0) ;
}
public Item(String description, int itemId) {
this.description = description;
this.itemId = itemId;
}
}

```

1.  接下来，创建一个生产者类来生成`Item`类型的元素。为了本示例的目的，我们只会生成七个项目，然后打印一条语句来演示该项目已被添加到队列中。我们将使用`LinkedTransferQueue`类的`offer`方法来添加元素。在每次添加后，线程会短暂休眠，然后我们打印出添加的项目的名称。然后我们使用`hasWaitingConsumer`方法来确定是否有任何消费者线程正在等待可用的项目：

```java
static class ItemProducer implements Runnable {
@Override
public void run() {
try {
for (int x = 1; x < 8; x++) {
String itemName = "Item" + x;
int itemId = x;
linkTransQ.offer(new Item(itemName, itemId));
System.out.println("New Item Added:" + itemName + " " + itemId);
Thread.currentThread().sleep(250);
if (linkTransQ.hasWaitingConsumer()) {
System.out.println("Hurry up!");
}
}
}
catch (InterruptedException ex) {
ex.printStackTrace();
}
}
}

```

1.  接下来，创建一个消费者类。为了演示`hasWaitingConsumer`方法的功能，我们让线程在检索元素之前睡眠一秒钟，以确保一开始没有等待的消费者。然后，在`while`循环内，我们使用`take`方法来移除列表中的第一个项目。我们选择了`take`方法，因为它是一个阻塞方法，会等待直到队列有可用的元素。一旦消费者线程能够取出一个元素，我们将元素传递给`generateOrder`方法，该方法打印有关项目的信息：

```java
static class ItemConsumer implements Runnable {
@Override
public void run() {
try {
Thread.currentThread().sleep(1000);
}
catch (InterruptedException ex) {
ex.printStackTrace();
}
while (true) {
try {
generateOrder(linkTransQ.take());
}
catch (InterruptedException ex) {
ex.printStackTrace();
}
}
}
private void generateOrder(Item item) {
System.out.println();
System.out.println("Part Order");
System.out.println("Item description: " + item.description());
System.out.println("Item ID # " + item.itemId());
}
}

```

1.  最后，在我们的`main`方法中，我们启动了两个线程：

```java
public static void main(String[] args) {
new Thread(new ItemProducer()).start();
new Thread(new ItemConsumer()).start();
}

```

1.  当您执行程序时，您应该看到类似以下的输出：

**新添加的项目：Item1 1**

**新添加的项目：Item2 2**

**新添加的项目：Item3 3**

**新添加的项目：Item4 4**

零件订单

**项目描述：Item1**

**项目编号＃1**

零件订单

**项目描述：Item2**

**项目编号＃2**

零件订单

**项目描述：Item3**

**项目编号＃3**

零件订单

**项目描述：Item4**

**项目编号＃4**

**快点！**

**新添加的项目：Item5 5**

零件订单

**项目描述：Item5**

**项目编号＃5**

**快点！**

零件订单

**项目描述：Item6**

**项目编号＃6**

**新添加的项目：Item6 6**

**快点！**

零件订单

**项目描述：Item7**

项目编号＃7

**新添加的项目：Item7 7**

快点！

## 它是如何工作的...

当我们启动了两个线程时，我们让生产者线程有一个**领先**，通过在`ItemConsumer`类中睡眠一秒钟来填充我们的队列。请注意，`hasWaitingConsumer`方法最初返回`false`，因为消费者线程尚未执行`take`方法。一秒钟后，消费者线程开始检索元素。在每次检索时，`generateOrder`方法打印有关检索到的元素的信息。在检索队列中的所有元素之后，请注意最后的*快点！*语句，表示仍有消费者在等待。在这个例子中，因为消费者在`while`循环中使用了一个阻塞方法，线程永远不会终止。在现实生活中，线程应该以更优雅的方式终止，比如向消费者线程发送终止消息。

在我们的例子中，我们使用了`offer`和`take`方法来添加和移除队列的元素。还有其他可用的方法，这些方法在*还有更多..*部分中讨论。

## 还有更多...

在这里，我们将讨论以下内容：

+   异步并发线程的问题

+   向队列添加元素

+   从双端队列中检索元素

### 异步并发线程的问题

由于多个线程可能在任何给定时刻访问集合，因此`size`方法不总是会返回准确的结果。此外，任何批量数据操作，如`addAll`或`removeAll`，也不总能达到期望的结果。如果一个线程正在访问集合中的一个项目，另一个线程尝试拉取所有项目，则不保证批量操作会以原子方式运行。

### 向队列添加元素

以下是一些可用于向队列添加元素的方法：

| 方法名称 | 添加元素到 | 评论 |
| --- | --- | --- |
| `add（Element e）` | 队列末尾 | 队列是无界的，因此该方法永远不会返回`false`或抛出异常 |
| `offer（Element e）` |  | 队列是无界的，因此该方法永远不会返回`false` |
| `put（Element e）` |  | 队列是无界的，因此该方法永远不会阻塞 |
| `offer（Element``e, Long t`,`TimeUnit u）` | 队列末尾等待 t 个时间单位的类型 u 然后放弃 | 队列是无界的，因此该方法将始终返回`true` |

### 从双端队列中检索元素

以下是一些可用于从双端队列中检索元素的方法：

| 方法名称 | 功能 | 评论 |
| --- | --- | --- |
| `peek()` | 检索队列的第一个元素，但不移除 | 如果队列为空，则返回 null |
| `poll()` | 移除队列的第一个元素 | 如果队列为空，则返回 null |
| `poll(Long t, TimeUnit u)` | 从队列前面移除元素，在时间 t（以单位 u 计）之前放弃 | 如果时间限制在元素可用之前到期，则返回 null |
| `remove(Object e)` | 从队列中移除等于`Object e`的元素 | 如果找到并移除元素，则返回`true` |
| `take()` | 移除队列的第一个元素 | 如果在阻塞时被中断，则抛出异常 |
| `transfer(Element e)` | 将元素传输给消费者线程，必要时等待 | 将元素插入队列末尾，并等待消费者线程检索它 |
| `tryTransfer(Element e)` | 立即将元素传输给消费者 | 如果消费者不可用，则返回`false` |
| `tryTransfer(Element e, Time t, TimeUnit u)` | 立即将元素传输给消费者，或在 t（以单位 u 计）指定的时间内 | 如果消费者在时间限制到期时不可用，则返回`false` |

# 使用 ThreadLocalRandom 类支持多个线程

`java.util.concurrent`包中有一个新的类`ThreadLocalRandom`，它支持类似于`Random`类的功能。然而，使用这个新类与多个线程将导致较少的争用和更好的性能，与`Random`类相比。当多个线程需要使用随机数时，应该使用`ThreadLocalRandom`类。随机数生成器是局部的。本食谱将介绍如何使用这个类。

## 准备就绪

使用这个类的推荐方法是：

1.  使用静态的`current`方法返回`ThreadLocalRandom`类的一个实例。

1.  使用该对象的方法。

## 如何做...

1.  创建一个新的控制台应用程序。将以下代码添加到`main`方法中：

```java
System.out.println("Five random integers");
for(int i = 0; i<5; i++) {
System.out.println(ThreadLocalRandom.current(). nextInt());
}
System.out.println();
System.out.println("Random double number between 0.0 and 35.0");
System.out.println(ThreadLocalRandom.current().nextDouble(35.0));
System.out.println();
System.out.println("Five random Long numbers between 1234567 and 7654321");
for(int i = 0; i<5; i++) {
System.out.println(
ThreadLocalRandom.current().nextLong(1234567L, 7654321L));
}

```

1.  执行程序。您的输出应该类似于以下内容：

**五个随机整数**

**0**

**4232237**

**178803790**

**758674372**

**1565954732**

**0.0 和 35.0 之间的随机双精度数**

**3.196571144914888**

**1234567 和 7654321 之间的五个随机长整数**

**7525440**

**2545475**

**1320305**

**1240628**

**1728476**

## 它是如何工作的...

`nextInt`方法被执行了五次，其返回值被显示出来。注意该方法最初返回 0。`ThreadLocalRandom`类扩展了`Random`类。然而，不支持`setSeed`方法。如果尝试使用它，将抛出`UnsupportedOperationException`。

然后执行了`nextDouble`方法。这个重载方法返回了一个介于 0.0 和 35.0 之间的数字。使用两个参数执行了五次`nextLong`方法，指定了其起始（包括）和结束（不包括）的范围值。

## 还有更多...

该类的方法返回均匀分布的数字。以下表总结了它的方法：

### 提示

当指定范围时，起始值是包含的，结束值是不包含的。

| 方法 | 参数 | 返回 |
| --- | --- | --- |
| `current` | 无 | 线程的当前实例 |
| `next` | 代表返回值位数的整数值 | 位数范围内的整数 |
| `nextDouble` | doubledouble, double | 0.0 和其参数之间的双精度数 0.0 和其参数之间的双精度数 |
| `nextInt` | int, int | 其参数之间的整数 |
| `nextLong` | longlong, long | 0 和其参数之间的长整数 0 和其参数之间的长整数 |
| `setSeed` | long | 抛出 `UnsupportedOperationException` |

## 另请参阅

在*使用可重用同步障碍 Phaser*食谱中找到了它的用法示例。
