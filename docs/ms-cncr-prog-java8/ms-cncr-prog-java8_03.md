# 第二章：管理大量线程-执行程序

当您实现简单的并发应用程序时，您会为每个并发任务创建和执行一个线程。这种方法可能会有一些重要问题。自**Java 版本 5**以来，Java 并发 API 包括**执行程序框架**，以提高具有大量并发任务的并发应用程序的性能。在本章中，我们将介绍以下内容：

+   执行程序介绍

+   第一个示例- k 最近邻算法

+   第二个示例-客户端/服务器环境中的并发性

# 执行程序介绍

在 Java 中实现并发应用程序的基本机制是：

+   **实现 Runnable 接口的类**：这是您想以并发方式实现的代码

+   **Thread 类的实例**：这是将以并发方式执行代码的线程

通过这种方法，您负责创建和管理`Thread`对象，并实现线程之间的同步机制。但是，它可能会有一些问题，特别是对于具有大量并发任务的应用程序。如果创建了太多的线程，可能会降低应用程序的性能，甚至挂起整个系统。

Java 5 包括执行程序框架，以解决这些问题并提供有效的解决方案，这将比传统的并发机制更容易供程序员使用。

在本章中，我们将通过使用执行程序框架实现以下两个示例来介绍执行程序框架的基本特性：

+   **k 最近邻算法**：这是一种基本的**机器学习**算法，用于分类。它根据训练数据集中*k*个最相似示例的标签确定测试示例的标签。

+   **客户端/服务器环境中的并发性**：为数千或数百万客户端提供信息的应用程序现在至关重要。在最佳方式下实现系统的服务器端是至关重要的。

在第三章中，*从执行程序中获取最大值*，和第四章中，*从任务中获取数据- Callable 和 Future 接口*，我们将介绍执行程序的更高级方面。

## 执行程序的基本特性

执行程序的主要特点是：

+   您不需要创建任何`Thread`对象。如果要执行并发任务，只需创建任务的实例（例如，实现`Runnable`接口的类），并将其发送到执行程序。它将管理执行任务的线程。

+   执行程序通过重用线程来减少线程创建引入的开销。在内部，它管理一个名为**worker-threads**的线程池。如果您将任务发送到执行程序并且有一个空闲的 worker-thread，执行程序将使用该线程来执行任务。

+   很容易控制执行程序使用的资源。您可以限制执行程序的 worker-threads 的最大数量。如果发送的任务多于 worker-threads，执行程序会将它们存储在队列中。当 worker-thread 完成任务的执行时，它会从队列中取出另一个任务。

+   您必须显式完成执行程序的执行。您必须指示执行程序完成其执行并终止创建的线程。如果不这样做，它将无法完成其执行，您的应用程序也将无法结束。

执行程序具有更多有趣的特性，使其非常强大和灵活。

## 执行程序框架的基本组件

执行程序框架具有各种接口和类，实现了执行程序提供的所有功能。框架的基本组件包括：

+   **Executor 接口**：这是执行器框架的基本接口。它只定义了一个允许程序员将`Runnable`对象发送到执行器的方法。

+   **ExecutorService 接口**：这个接口扩展了`Executor`接口，并包括更多的方法来增加框架的功能，例如：

+   执行返回结果的任务：`Runnable`接口提供的`run()`方法不返回结果，但使用执行器，你可以有返回结果的任务。

+   使用单个方法调用执行任务列表

+   完成执行器的执行并等待其终止

+   **ThreadPoolExecutor 类**：这个类实现了`Executor`和`ExecutorService`接口。此外，它包括一些额外的方法来获取执行器的状态（工作线程数、执行任务数等），建立执行器的参数（最小和最大工作线程数、空闲线程等待新任务的时间等），以及允许程序员扩展和调整其功能的方法。

+   **Executors 类**：这个类提供了创建`Executor`对象和其他相关类的实用方法。

# 第一个例子 - k 最近邻算法

k 最近邻算法是一种简单的用于监督分类的机器学习算法。该算法的主要组成部分是：

+   **一个训练数据集**：这个数据集由一个或多个属性定义每个实例以及一个特殊属性组成，该属性确定实例的示例或标签

+   **一个距离度量标准**：这个度量标准用于确定训练数据集的实例与你想要分类的新实例之间的距离（或相似性）

+   **一个测试数据集**：这个数据集用于衡量算法的行为

当它必须对一个实例进行分类时，它会计算与这个实例和训练数据集中所有实例的距离。然后，它会取最近的 k 个实例，并查看这些实例的标签。具有最多实例的标签将被分配给输入实例。

在本章中，我们将使用**UCI 机器学习库**的**银行营销**数据集，你可以从[`archive.ics.uci.edu/ml/datasets/Bank+Marketing`](http://archive.ics.uci.edu/ml/datasets/Bank+Marketing)下载。为了衡量实例之间的距离，我们将使用**欧几里得距离**。使用这个度量标准，我们实例的所有属性必须具有数值。银行营销数据集的一些属性是分类的（也就是说，它们可以取一些预定义的值），所以我们不能直接使用欧几里得距离。可以为每个分类值分配有序数；例如，对于婚姻状况，0 表示*单身*，1 表示*已婚*，2 表示*离婚*。然而，这将意味着*离婚*的人比*已婚*更接近*单身*，这是值得商榷的。为了使所有分类值等距离，我们创建单独的属性，如*已婚*、*单身*和*离婚*，它们只有两个值：0（*否*）和 1（*是*）。

我们的数据集有 66 个属性和两个可能的标签：*是*和*否*。我们还将数据分成了两个子集：

+   **训练数据集**：有 39,129 个实例

+   **测试数据集**：有 2,059 个实例

正如我们在第一章中解释的那样，*第一步 - 并发设计原则*，我们首先实现了算法的串行版本。然后，我们寻找可以并行化的算法部分，并使用执行器框架来执行并发任务。在接下来的章节中，我们将解释 k 最近邻算法的串行实现和两个不同的并发版本。第一个版本具有非常细粒度的并发性，而第二个版本具有粗粒度的并发性。

## K 最近邻 - 串行版本

我们已经在`KnnClassifier`类中实现了算法的串行版本。在内部，这个类存储了训练数据集和数字`k`（我们将用来确定实例标签的示例数量）：

```java
public class KnnClassifier {

  private List <? extends Sample> dataSet;
  private int k;

  public KnnClassifier(List <? extends Sample> dataSet, int k) {
    this.dataSet=dataSet;
    this.k=k;
  }
```

`KnnClassifier`类只实现了一个名为`classify`的方法，该方法接收一个`Sample`对象，其中包含我们要分类的实例，并返回一个分配给该实例的标签的字符串：

```java
  public String classify (Sample example) {
```

这种方法有三个主要部分 - 首先，我们计算输入示例与训练数据集中所有示例之间的距离：

```java
    Distance[] distances=new Distance[dataSet.size()];

    int index=0;

    for (Sample localExample : dataSet) {
      distances[index]=new Distance();
      distances[index].setIndex(index);
      distances[index].setDistance (EuclideanDistanceCalculator.calculate(localExample, example));
      index++;
    }
```

然后，我们使用`Arrays.sort()`方法将示例按距离从低到高排序：

```java
    Arrays.sort(distances);
```

最后，我们统计 k 个最近示例中出现最多的标签：

```java
    Map<String, Integer> results = new HashMap<>();
    for (int i = 0; i < k; i++) {
      Sample localExample = dataSet.get(distances[i].getIndex());
      String tag = localExample.getTag();
      results.merge(tag, 1, (a, b) -> a+b);
    }
    return Collections.max(results.entrySet(), Map.Entry.comparingByValue()).getKey();
  }
```

为了计算两个示例之间的距离，我们可以使用一个辅助类中实现的欧几里得距离。这是该类的代码：

```java
public class EuclideanDistanceCalculator {
  public static double calculate (Sample example1, Sample example2) {
    double ret=0.0d;

    double[] data1=example1.getExample();
    double[] data2=example2.getExample();

    if (data1.length!=data2.length) {
      throw new IllegalArgumentException ("Vector doesn't have the same length");
    }

    for (int i=0; i<data1.length; i++) {
      ret+=Math.pow(data1[i]-data2[i], 2);
    }
    return Math.sqrt(ret);
  }

}
```

我们还使用`Distance`类来存储`Sample`输入和训练数据集实例之间的距离。它只有两个属性：训练数据集示例的索引和输入示例的距离。此外，它实现了`Comparable`接口以使用`Arrays.sort()`方法。最后，`Sample`类存储一个实例。它只有一个双精度数组和一个包含该实例标签的字符串。

## K 最近邻 - 细粒度并发版本

如果你分析 k 最近邻算法的串行版本，你会发现以下两个点可以并行化算法：

+   **距离的计算**：计算输入示例与训练数据集中一个示例之间的距离的每次循环迭代都是独立的

+   **距离的排序**：Java 8 在`Arrays`类中包含了`parallelSort()`方法，以并发方式对数组进行排序。

在算法的第一个并发版本中，我们将为我们要计算的示例之间的每个距离创建一个任务。我们还将使并发排序数组的产生成为可能。我们在一个名为`KnnClassifierParrallelIndividual`的类中实现了这个算法的版本。它存储了训练数据集、`k`参数、`ThreadPoolExecutor`对象来执行并行任务、一个属性来存储我们想要在执行器中拥有的工作线程数量，以及一个属性来存储我们是否想要进行并行排序。

我们将创建一个具有固定线程数的执行器，以便我们可以控制此执行器将使用的系统资源。这个数字将是系统中可用处理器的数量，我们使用`Runtime`类的`availableProcessors()`方法获得，乘以构造函数中名为`factor`的参数的值。它的值将是从处理器获得的线程数。我们将始终使用值`1`，但您可以尝试其他值并比较结果。这是分类的构造函数：

```java
public class KnnClassifierParallelIndividual {

  private List<? extends Sample> dataSet;
  private int k;
  private ThreadPoolExecutor executor;
  private int numThreads;
  private boolean parallelSort;

  public KnnClassifierParallelIndividual(List<? extends Sample> dataSet, int k, int factor, boolean parallelSort) {
    this.dataSet=dataSet;
    this.k=k;
    numThreads=factor* (Runtime.getRuntime().availableProcessors());
    executor=(ThreadPoolExecutor) Executors.newFixedThreadPool(numThreads);
    this.parallelSort=parallelSort;
  }
```

要创建执行程序，我们使用了`Executors`实用类及其`newFixedThreadPool()`方法。此方法接收您希望在执行程序中拥有的工作线程数。执行程序的工作线程数永远不会超过您在构造函数中指定的数量。此方法返回一个`ExecutorService`对象，但我们将其转换为`ThreadPoolExecutor`对象，以便访问类提供的方法，而这些方法不包含在接口中。

该类还实现了`classify()`方法，该方法接收一个示例并返回一个字符串。

首先，我们为需要计算的每个距离创建一个任务并将它们发送到执行程序。然后，主线程必须等待这些任务的执行结束。为了控制最终化，我们使用了 Java 并发 API 提供的同步机制：`CountDownLatch`类。该类允许一个线程等待，直到其他线程到达其代码的确定点。它使用两种方法：

+   `getDown()`:此方法减少您必须等待的线程数。

+   `await()`:此方法挂起调用它的线程，直到计数器达到零

在这种情况下，我们使用任务数初始化`CountDownLatch`类。主线程调用`await()`方法，并在完成计算时为每个任务调用`getDown()`方法：

```java
  public String classify (Sample example) throws Exception {

    Distance[] distances=new Distance[dataSet.size()];
    CountDownLatch endController=new CountDownLatch(dataSet.size());

    int index=0;
    for (Sample localExample : dataSet) {
      IndividualDistanceTask task=new IndividualDistanceTask(distances, index, localExample, example, endController);
      executor.execute(task);
      index++;
    }
    endController.await();
```

然后，根据`parallelSort`属性的值，我们调用`Arrays.sort()`或`Arrays.parallelSort()`方法。

```java
    if (parallelSort) {
      Arrays.parallelSort(distances);
    } else {
      Arrays.sort(distances);
    }
```

最后，我们计算分配给输入示例的标签。此代码与串行版本相同。

`KnnClassifierParallelIndividual`类还包括一个调用其`shutdown()`方法关闭执行程序的方法。如果不调用此方法，您的应用程序将永远不会结束，因为执行程序创建的线程仍然活着，等待执行新任务。先前提交的任务将被执行，并且新提交的任务将被拒绝。该方法不会等待执行程序的完成，它会立即返回：

```java
  public void destroy() {
    executor.shutdown();
  }
```

这个示例的一个关键部分是`IndividualDistanceTask`类。这是一个计算输入示例与训练数据集示例之间距离的类。它存储完整的距离数组（我们将仅为其之一的位置设置值），训练数据集示例的索引，两个示例和用于控制任务结束的`CountDownLatch`对象。它实现了`Runnable`接口，因此可以在执行程序中执行。这是该类的构造函数：

```java
public class IndividualDistanceTask implements Runnable {

  private Distance[] distances;
  private int index;
  private Sample localExample;
  private Sample example;
  private CountDownLatch endController;

  public IndividualDistanceTask(Distance[] distances, int index, Sample localExample,
      Sample example, CountDownLatch endController) {
    this.distances=distances;
    this.index=index;
    this.localExample=localExample;
    this.example=example;
    this.endController=endController;
  }
```

`run()`方法使用之前解释的`EuclideanDistanceCalculator`类计算两个示例之间的距离，并将结果存储在距离的相应位置：

```java
  public void run() {
    distances[index] = new Distance();
    distances[index].setIndex(index);
    distances[index].setDistance (EuclideanDistanceCalculator.calculate(localExample, example));
    endController.countDown();
  }
```

### 提示

请注意，尽管所有任务共享距离数组，但我们不需要使用任何同步机制，因为每个任务将修改数组的不同位置。

## K 最近邻 - 粗粒度并发版本

在上一节中介绍的并发解决方案可能存在问题。您正在执行太多任务。如果停下来想一想，在这种情况下，我们有超过 29,000 个训练示例，因此您将为每个要分类的示例启动 29,000 个任务。另一方面，我们已经创建了一个最大具有`numThreads`工作线程的执行程序，因此另一个选项是仅启动`numThreads`个任务并将训练数据集分成`numThreads`组。我们使用四核处理器执行示例，因此每个任务将计算输入示例与大约 7,000 个训练示例之间的距离。

我们已经在`KnnClassifierParallelGroup`类中实现了这个解决方案。它与`KnnClassifierParallelIndividual`类非常相似，但有两个主要区别。首先是`classify()`方法的第一部分。现在，我们只有`numThreads`个任务，我们必须将训练数据集分成`numThreads`个子集：

```java
  public String classify(Sample example) throws Exception {

    Distance distances[] = new Distance[dataSet.size()];
    CountDownLatch endController = new CountDownLatch(numThreads);

    int length = dataSet.size() / numThreads;
    int startIndex = 0, endIndex = length;

    for (int i = 0; i < numThreads; i++) {
      GroupDistanceTask task = new GroupDistanceTask(distances, startIndex, endIndex, dataSet, example, endController);
      startIndex = endIndex;
      if (i < numThreads - 2) {
       endIndex = endIndex + length;
      } else {
       endIndex = dataSet.size();
      }
      executor.execute(task);

    }
    endController.await();
```

在长度变量中计算每个任务的样本数量。然后，我们为每个线程分配它们需要处理的样本的起始和结束索引。对于除最后一个线程之外的所有线程，我们将长度值添加到起始索引以计算结束索引。对于最后一个线程，最后一个索引是数据集的大小。

其次，这个类使用`GroupDistanceTask`而不是`IndividualDistanceTask`。这两个类之间的主要区别是第一个处理训练数据集的子集，因此它存储了完整的训练数据集以及它需要处理的数据集的第一个和最后一个位置：

```java
public class GroupDistanceTask implements Runnable {
  private Distance[] distances;
  private int startIndex, endIndex;
  private Sample example;
  private List<? extends Sample> dataSet;
  private CountDownLatch endController;

  public GroupDistanceTask(Distance[] distances, int startIndex, int endIndex, List<? extends Sample> dataSet, Sample example, CountDownLatch endController) {
    this.distances = distances;
    this.startIndex = startIndex;
    this.endIndex = endIndex;
    this.example = example;
    this.dataSet = dataSet;
    this.endController = endController;
  }
```

`run()`方法处理一组示例而不仅仅是一个示例：

```java
  public void run() {
    for (int index = startIndex; index < endIndex; index++) {
      Sample localExample=dataSet.get(index);
      distances[index] = new Distance();
      distances[index].setIndex(index);
        distances[index].setDistance(EuclideanDistanceCalculator
            .calculate(localExample, example));
    }
    endController.countDown();
  }
```

## 比较解决方案

让我们比较我们实现的 k 最近邻算法的不同版本。我们有以下五个不同的版本：

+   串行版本

+   具有串行排序的细粒度并发版本

+   具有并发排序的细粒度并发版本

+   具有串行排序的粗粒度并发版本

+   具有并发排序的粗粒度并发版本

为了测试算法，我们使用了 2,059 个测试实例，这些实例来自银行营销数据集。我们使用 k 的值为 10、30 和 50，对所有这些示例使用了算法的五个版本进行分类，并测量它们的执行时间。我们使用了**JMH 框架**（[`openjdk.java.net/projects/code-tools/jmh/`](http://openjdk.java.net/projects/code-tools/jmh/)），它允许您在 Java 中实现微基准测试。使用基准测试框架比简单地使用`currentTimeMillis()`或`nanoTime()`方法来测量时间更好。以下是结果：

| 算法 | K | 执行时间（秒） |
| --- | --- | --- |
| 串行 | 10 | 100.296 |
| 30 | 99.218 |
| 50 | 99.458 |
| 细粒度串行排序 | 10 | 108.150 |
| 30 | 105.196 |
| 50 | 109.797 |
| 细粒度并发排序 | 10 | 84.663 |
| 30 | 85,392 |
| 50 | 83.373 |
| 粗粒度串行排序 | 10 | 78.328 |
| 30 | 77.041 |
| 50 | 76.549 |
| 粗粒度并发排序 | 10 | 54,017 |
| 30 | 53.473 |
| 50 | 53.255 |

我们可以得出以下结论：

+   所选的 K 参数值（10、30 和 50）不影响算法的执行时间。这五个版本对于这三个值呈现出类似的结果。

+   正如预期的那样，使用`Arrays.parallelSort()`方法的并发排序在算法的细粒度和粗粒度并发版本中都大大提高了性能。

+   算法的细粒度版本与串行算法给出了相同或略差的结果。并发任务的创建和管理引入的开销导致了这些结果。我们执行了太多的任务。

+   另一方面，粗粒度版本提供了很大的性能改进，无论是串行还是并行排序。

因此，算法的最佳版本是使用并行排序的粗粒度解决方案。如果我们将其与计算加速度的串行版本进行比较：

![比较解决方案](img/00007.jpeg)

这个例子显示了一个并发解决方案的良好选择如何给我们带来巨大的改进，而糟糕的选择会给我们带来糟糕的性能。

# 第二个例子 - 客户端/服务器环境中的并发性

**客户端/服务器模型**是一种软件架构，将应用程序分为两部分：提供资源（数据、操作、打印机、存储等）的服务器部分和使用服务器提供的资源的客户端部分。传统上，这种架构在企业世界中使用，但随着互联网的兴起，它仍然是一个实际的话题。您可以将 Web 应用程序视为客户端/服务器应用程序，其中服务器部分是在 Web 服务器中执行的应用程序的后端部分，Web 浏览器执行应用程序的客户端部分。**SOA**（**面向服务的架构**的缩写）是客户端/服务器架构的另一个例子，其中公开的 Web 服务是服务器部分，而消费这些服务的不同客户端是客户端部分。

在客户端/服务器环境中，通常有一个服务器和许多客户端使用服务器提供的服务，因此服务器的性能是设计这些系统时的关键方面之一。

在本节中，我们将实现一个简单的客户端/服务器应用程序。它将对**世界银行**的**世界发展指标**进行数据搜索，您可以从这里下载：[`data.worldbank.org/data-catalog/world-development-indicators`](http://data.worldbank.org/data-catalog/world-development-indicators)。这些数据包含了 1960 年至 2014 年间世界各国不同指标的数值。

我们服务器的主要特点将是：

+   客户端和服务器将使用套接字连接

+   客户端将以字符串形式发送其查询，服务器将以另一个字符串形式回复结果

+   服务器可以用三种不同的查询进行回复：

+   **查询**：此查询的格式为 `q;codCountry;codIndicator;year`，其中 `codCountry` 是国家的代码，`codIndicator` 是指标的代码，`year` 是一个可选参数，表示您要查询的年份。服务器将以单个字符串形式回复信息。

+   **报告**：此查询的格式为 `r;codIndicator`，其中 `codIndicator` 是您想要报告的指标的代码。服务器将以单个字符串形式回复所有国家在多年间该指标的平均值。

+   **停止**：此查询的格式为 `z;`。服务器在收到此命令时停止执行。

+   在其他情况下，服务器会返回错误消息。

与之前的示例一样，我们将向您展示如何实现此客户端/服务器应用程序的串行版本。然后，我们将向您展示如何使用执行器实现并发版本。最后，我们将比较这两种解决方案，以查看在这种情况下使用并发的优势。

## 客户端/服务器 - 串行版本

我们的服务器应用程序的串行版本有三个主要部分：

+   **DAO**（**数据访问对象**的缩写）部分，负责访问数据并获取查询结果

+   命令部分，由每种查询类型的命令组成

+   服务器部分，接收查询，调用相应的命令，并将结果返回给客户端

让我们详细看看这些部分。

### DAO 部分

如前所述，服务器将对世界银行的世界发展指标进行数据搜索。这些数据在一个 CSV 文件中。应用程序中的 DAO 组件将整个文件加载到内存中的 `List` 对象中。它实现了一个方法来处理它将处理的每个查询，以便查找数据。

我们不在此处包含此类的代码，因为它很容易实现，而且不是本书的主要目的。

### 命令部分

命令部分是 DAO 和服务器部分之间的中介。我们实现了一个基本的抽象 `Command` 类，作为所有命令的基类：

```java
public abstract class Command {

  protected String[] command;

  public Command (String [] command) {
    this.command=command;
  }

  public abstract String execute ();

}
```

然后，我们为每个查询实现了一个命令。查询在 `QueryCommand` 类中实现。`execute()` 方法如下：

```java
  public String execute() {
    WDIDAO dao=WDIDAO.getDAO();

    if (command.length==3) {
      return dao.query(command[1], command[2]);
    } else if (command.length==4) {
      try {
        return dao.query(command[1], command[2], Short.parseShort(command[3]));
      } catch (Exception e) {
        return "ERROR;Bad Command";
      }
    } else {
      return "ERROR;Bad Command";
    }
  }
```

报告是在`ReportCommand`中实现的。`execute()`方法如下：

```java
  @Override
  public String execute() {

    WDIDAO dao=WDIDAO.getDAO();
    return dao.report(command[1]);
  }
```

停止查询是在`StopCommand`类中实现的。其`execute()`方法如下：

```java
  @Override
  public String execute() {
    return "Server stopped";
  }
```

最后，错误情况由`ErrorCommand`类处理。其`execute()`方法如下：

```java
  @Override
  public String execute() {
    return "Unknown command: "+command[0];
  }
```

### 服务器部分

最后，服务器部分是在`SerialServer`类中实现的。首先，它通过调用`getDAO()`方法来初始化 DAO。主要目标是 DAO 加载所有数据：

```java
public class SerialServer {

  public static void main(String[] args) throws IOException {
    WDIDAO dao = WDIDAO.getDAO();
    boolean stopServer = false;
    System.out.println("Initialization completed.");

    try (ServerSocket serverSocket = new ServerSocket(Constants.SERIAL_PORT)) {
```

之后，我们有一个循环，直到服务器接收到停止查询才会执行。这个循环执行以下四个步骤：

+   接收来自客户端的查询

+   解析和拆分查询的元素

+   调用相应的命令

+   将结果返回给客户端

这四个步骤显示在以下代码片段中：

```java
  do {
    try (Socket clientSocket = serverSocket.accept();
      PrintWriter out = new PrintWriter (clientSocket.getOutputStream(), true);
      BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));) {
    String line = in.readLine();
    Command command;

    String[] commandData = line.split(";");
    System.out.println("Command: " + commandData[0]);
    switch (commandData[0]) {
    case "q":
      System.out.println("Query");
      command = new QueryCommand(commandData);
      break;
    case "r":
      System.out.println("Report");
      command = new ReportCommand(commandData);
      break;
    case "z":
      System.out.println("Stop");
      command = new StopCommand(commandData);
      stopServer = true;
      break;
    default:
      System.out.println("Error");
      command = new ErrorCommand(commandData);
    }
    String response = command.execute();
    System.out.println(response);
    out.println(response);
  } catch (IOException e) {
    e.printStackTrace();
  }
} while (!stopServer);
```

## 客户端/服务器-并行版本

服务器的串行版本有一个非常重要的限制。在处理一个查询时，它无法处理其他查询。如果服务器需要大量时间来响应每个请求，或者某些请求，服务器的性能将非常低。

使用并发可以获得更好的性能。如果服务器在收到请求时创建一个线程，它可以将所有查询的处理委托给线程，并且可以处理新的请求。这种方法也可能存在一些问题。如果我们收到大量查询，我们可能会通过创建太多线程来饱和系统。但是，如果我们使用具有固定线程数的执行程序，我们可以控制服务器使用的资源，并获得比串行版本更好的性能。

要将我们的串行服务器转换为使用执行程序的并发服务器，我们必须修改服务器部分。DAO 部分是相同的，我们已更改实现命令部分的类的名称，但它们的实现几乎相同。只有停止查询发生了变化，因为现在它有更多的责任。让我们看看并发服务器部分的实现细节。

### 服务器部分

并发服务器部分是在`ConcurrentServer`部分实现的。我们添加了两个在串行服务器中未包括的元素：一个缓存系统，实现在`ParallelCache`类中，以及一个日志系统，实现在`Logger`类中。首先，它通过调用`getDAO()`方法来初始化 DAO 部分。主要目标是 DAO 加载所有数据并使用`Executors`类的`newFixedThreadPool()`方法创建`ThreadPoolExecutor`对象。此方法接收我们服务器中要使用的最大工作线程数。执行程序永远不会有超过这些工作线程。要获取工作线程数，我们使用`Runtime`类的`availableProcessors()`方法获取系统的核心数：

```java
public class ConcurrentServer {

  private static ThreadPoolExecutor executor;

  private static ParallelCache cache;

  private static ServerSocket serverSocket;

  private static volatile boolean stopped = false;

  public static void main(String[] args) {

    serverSocket=null;
    WDIDAO dao=WDIDAO.getDAO();
    executor=(ThreadPoolExecutor) Executors.newFixedThreadPool (Runtime.getRuntime().availableProcessors());
    cache=new ParallelCache();
    Logger.initializeLog();

    System.out.println("Initialization completed.");
```

`stopped`布尔变量声明为 volatile，因为它将从另一个线程更改。`volatile`关键字确保当`stopped`变量被另一个线程设置为`true`时，这种更改将在主方法中可见。没有`volatile`关键字，由于 CPU 缓存或编译器优化，更改可能不可见。然后，我们初始化`ServerSocket`以侦听请求：

```java
    serverSocket = new ServerSocket(Constants.CONCURRENT_PORT);
```

我们不能使用 try-with-resources 语句来管理服务器套接字。当我们收到`stop`命令时，我们需要关闭服务器，但服务器正在`serverSocket`对象的`accept()`方法中等待。为了强制服务器离开该方法，我们需要显式关闭服务器（我们将在`shutdown()`方法中执行），因此我们不能让 try-with-resources 语句为我们关闭套接字。

之后，我们有一个循环，直到服务器接收到停止查询才会执行。这个循环有三个步骤，如下所示：

+   接收来自客户端的查询

+   创建一个处理该查询的任务

+   将任务发送给执行程序

这三个步骤显示在以下代码片段中：

```java
  do {
    try {
      Socket clientSocket = serverSocket.accept();
      RequestTask task = new RequestTask(clientSocket);
      executor.execute(task);
    } catch (IOException e) {
      e.printStackTrace();
    }
  } while (!stopped);
```

最后，一旦服务器完成了执行（退出循环），我们必须等待执行器的完成，使用 `awaitTermination()` 方法。这个方法将阻塞主线程，直到执行器完成其 `execution()` 方法。然后，我们关闭缓存系统，并等待一条消息来指示服务器执行的结束，如下所示：

```java
  executor.awaitTermination(1, TimeUnit.DAYS);
  System.out.println("Shutting down cache");
  cache.shutdown();
  System.out.println("Cache ok");

  System.out.println("Main server thread ended");
```

我们添加了两个额外的方法：`getExecutor()` 方法，返回用于执行并发任务的 `ThreadPoolExecutor` 对象，以及 `shutdown()` 方法，用于有序地结束服务器的执行器。它调用执行器的 `shutdown()` 方法，并关闭 `ServerSocket`：

```java
  public static void shutdown() {
    stopped = true;
    System.out.println("Shutting down the server...");
    System.out.println("Shutting down executor");
    executor.shutdown();
    System.out.println("Executor ok");
    System.out.println("Closing socket");
    try {
      serverSocket.close();
      System.out.println("Socket ok");
    } catch (IOException e) {
      e.printStackTrace();
    }
    System.out.println("Shutting down logger");
    Logger.sendMessage("Shutting down the logger");
    Logger.shutdown();
    System.out.println("Logger ok");
  }
```

在并发服务器中，有一个重要的部分：`RequestTask` 类，它处理客户端的每个请求。这个类实现了 `Runnable` 接口，因此可以以并发方式在执行器中执行。它的构造函数接收 `Socket` 参数，用于与客户端通信。

```java
public class RequestTask implements Runnable {

  private Socket clientSocket;

  public RequestTask(Socket clientSocket) {
    this.clientSocket = clientSocket;
  }
```

`run()` 方法做了与串行服务器相同的事情来响应每个请求：

+   接收客户端的查询

+   解析和拆分查询的元素

+   调用相应的命令

+   将结果返回给客户端

以下是它的代码片段：

```java
  public void run() {

    try (PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true);
      BufferedReader in = new BufferedReader(new InputStreamReader( clientSocket.getInputStream()));) {

      String line = in.readLine();

      Logger.sendMessage(line);
      ParallelCache cache = ConcurrentServer.getCache();
      String ret = cache.get(line);

      if (ret == null) {
        Command command;

        String[] commandData = line.split(";");
        System.out.println("Command: " + commandData[0]);
        switch (commandData[0]) {
        case "q":
          System.err.println("Query");
          command = new ConcurrentQueryCommand(commandData);
          break;
        case "r":
          System.err.println("Report");
          command = new ConcurrentReportCommand(commandData);
          break;
        case "s":
          System.err.println("Status");
          command = new ConcurrentStatusCommand(commandData);
          break;
        case "z":
          System.err.println("Stop");
          command = new ConcurrentStopCommand(commandData);
          break;
        default:
          System.err.println("Error");
          command = new ConcurrentErrorCommand(commandData);
          break;
        }
        ret = command.execute();
        if (command.isCacheable()) {
          cache.put(line, ret);
        }
      } else {
        Logger.sendMessage("Command "+line+" was found in the cache");
      }

      System.out.println(ret);
      out.println(ret);
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      try {
        clientSocket.close();
      } catch (IOException e) {
        e.printStackTrace();
      }
    }
  }
```

### 命令部分

在命令部分，我们已经重命名了所有的类，就像你在前面的代码片段中看到的那样。实现是一样的，除了 `ConcurrentStopCommand` 类。现在，它调用 `ConcurrentServer` 类的 `shutdown()` 方法，以有序地终止服务器的执行。这是 `execute()` 方法：

```java
  @Override
  public String execute() {
    ConcurrentServer.shutdown();
    return "Server stopped";
  }
```

此外，现在 `Command` 类包含一个新的 `isCacheable()` 布尔方法，如果命令结果存储在缓存中则返回 `true`，否则返回 `false`。

### 并发服务器的额外组件

我们在并发服务器中实现了一些额外的组件：一个新的命令来返回有关服务器状态的信息，一个缓存系统来存储命令的结果，当请求重复时节省时间，以及一个日志系统来写入错误和调试信息。以下各节描述了这些组件的每个部分。

#### 状态命令

首先，我们有一个新的可能的查询。它的格式是 `s;`，由 `ConcurrentStatusCommand` 类处理。它获取服务器使用的 `ThreadPoolExecutor`，并获取有关执行器状态的信息：

```java
public class ConcurrentStatusCommand extends Command {
  public ConcurrentStatusCommand (String[] command) {
    super(command);
    setCacheable(false);
  }
  @Override
  public String execute() {
    StringBuilder sb=new StringBuilder();
    ThreadPoolExecutor executor=ConcurrentServer.getExecutor();

    sb.append("Server Status;");
    sb.append("Actived Threads: ");
    sb.append(String.valueOf(executor.getActiveCount()));
    sb.append(";");
    sb.append("Maximum Pool Size: ");
    sb.append(String.valueOf(executor.getMaximumPoolSize()));
    sb.append(";");
    sb.append("Core Pool Size: ");
    sb.append(String.valueOf(executor.getCorePoolSize()));
    sb.append(";");
    sb.append("Pool Size: ");
    sb.append(String.valueOf(executor.getPoolSize()));
    sb.append(";");
    sb.append("Largest Pool Size: ");
    sb.append(String.valueOf(executor.getLargestPoolSize()));
    sb.append(";");
    sb.append("Completed Task Count: ");
    sb.append(String.valueOf(executor.getCompletedTaskCount()));
    sb.append(";");
    sb.append("Task Count: ");
    sb.append(String.valueOf(executor.getTaskCount()));
    sb.append(";");
    sb.append("Queue Size: ");
    sb.append(String.valueOf(executor.getQueue().size()));
    sb.append(";");
    sb.append("Cache Size: ");
    sb.append(String.valueOf (ConcurrentServer.getCache().getItemCount()));
    sb.append(";");
    Logger.sendMessage(sb.toString());
    return sb.toString();
  }
}
```

我们从服务器获取的信息是：

+   `getActiveCount()`: 这返回了执行我们并发任务的近似任务数量。池子中可能有更多的线程，但它们可能是空闲的。

+   `getMaximumPoolSize()`: 这返回了执行器可以拥有的最大工作线程数。

+   `getCorePoolSize()`: 这返回了执行器将拥有的核心工作线程数。这个数字决定了池子将拥有的最小线程数。

+   `getPoolSize()`: 这返回了池子中当前的线程数。

+   `getLargestPoolSize()`: 这返回了池子在执行期间的最大线程数。

+   `getCompletedTaskCount()`: 这返回了执行器已执行的任务数量。

+   `getTaskCount()`: 这返回了曾被调度执行的任务的近似数量。

+   `getQueue().size()`: 这返回了等待在任务队列中的任务数量。

由于我们使用 `Executor` 类的 `newFixedThreadPool()` 方法创建了我们的执行器，因此我们的执行器将具有相同的最大和核心工作线程数。

#### 缓存系统

我们在并行服务器中加入了一个缓存系统，以避免最近进行的数据搜索。我们的缓存系统有三个元素：

+   **CacheItem 类**：这个类代表缓存中存储的每个元素。它有四个属性：

+   缓存中存储的命令。我们将把 `query` 和 `report` 命令存储在缓存中。

+   由该命令生成的响应。

+   缓存中该项的创建日期。

+   缓存中该项上次被访问的时间。

+   **CleanCacheTask 类**：如果我们将所有命令存储在缓存中，但从未删除其中存储的元素，缓存的大小将无限增加。为了避免这种情况，我们可以有一个删除缓存中元素的任务。我们将实现这个任务作为一个`Thread`对象。有两个选项：

+   您可以在缓存中设置最大大小。如果缓存中的元素多于最大大小，可以删除最近访问次数较少的元素。

+   您可以从缓存中删除在预定义时间段内未被访问的元素。我们将使用这种方法。

+   **ParallelCache 类**：这个类实现了在缓存中存储和检索元素的操作。为了将数据存储在缓存中，我们使用了`ConcurrentHashMap`数据结构。由于缓存将在服务器的所有任务之间共享，我们必须使用同步机制来保护对缓存的访问，避免数据竞争条件。我们有三个选项：

+   我们可以使用一个非同步的数据结构（例如`HashMap`），并添加必要的代码来同步对这个数据结构的访问类型，例如使用锁。您还可以使用`Collections`类的`synchronizedMap()`方法将`HashMap`转换为同步结构。

+   使用同步数据结构，例如`Hashtable`。在这种情况下，我们没有数据竞争条件，但性能可能会更好。

+   使用并发数据结构，例如`ConcurrentHashMap`类，它消除了数据竞争条件的可能性，并且在高并发环境中进行了优化。这是我们将使用`ConcurrentHashMap`类的对象来实现的选项。

`CleanCacheTask`类的代码如下：

```java
public class CleanCacheTask implements Runnable {

  private ParallelCache cache;

  public CleanCacheTask(ParallelCache cache) {
    this.cache = cache;
  }

  @Override
  public void run() {
    try {
      while (!Thread.currentThread().interrupted()) {
        TimeUnit.SECONDS.sleep(10);
        cache.cleanCache();
      }
    } catch (InterruptedException e) {

    }
  }

}
```

该类有一个`ParallelCache`对象。每隔 10 秒，它执行`ParallelCache`实例的`cleanCache()`方法。

`ParallelCache`类有五种不同的方法。首先是类的构造函数，它初始化缓存的元素。它创建`ConcurrentHashMap`对象并启动一个将执行`CleanCacheTask`类的线程：

```java
public class ParallelCache {

  private ConcurrentHashMap<String, CacheItem> cache;
  private CleanCacheTask task;
  private Thread thread;
  public static int MAX_LIVING_TIME_MILLIS = 600_000;

  public ParallelCache() {
    cache=new ConcurrentHashMap<>();
    task=new CleanCacheTask(this);
    thread=new Thread(task);
    thread.start();
  }
```

然后，有两种方法来存储和检索缓存中的元素。我们使用`put()`方法将元素插入`HashMap`中，使用`get()`方法从`HashMap`中检索元素：

```java
  public void put(String command, String response) {
    CacheItem item = new CacheItem(command, response);
    cache.put(command, item);

  }

  public String get (String command) {
    CacheItem item=cache.get(command);
    if (item==null) {
      return null;
    }
    item.setAccessDate(new Date());
    return item.getResponse();
  }
```

然后，`CleanCacheTask`类使用的清除缓存的方法是：

```java
  public void cleanCache() {
    Date revisionDate = new Date();
    Iterator<CacheItem> iterator = cache.values().iterator();

    while (iterator.hasNext()) {
      CacheItem item = iterator.next();
      if (revisionDate.getTime() - item.getAccessDate().getTime() > MAX_LIVING_TIME_MILLIS) {
        iterator.remove();
      }
    }
  }
```

最后，关闭缓存的方法中断执行`CleanCacheTask`类的线程，并返回缓存中存储的元素数量的方法是：

```java
  public void shutdown() {
    thread.interrupt();
  }

  public int getItemCount() {
    return cache.size();
  }
```

#### 日志系统

在本章的所有示例中，我们使用`System.out.println()`方法在控制台中写入信息。当您实现一个将在生产环境中执行的企业应用程序时，最好使用日志系统来写入调试和错误信息。在 Java 中，`log4j`是最流行的日志系统。在这个例子中，我们将实现我们自己的日志系统，实现生产者/消费者并发设计模式。将使用我们的日志系统的任务将是生产者，而将日志信息写入文件的特殊任务（作为线程执行）将是消费者。这个日志系统的组件有：

+   **LogTask**：这个类实现了日志消费者，每隔 10 秒读取队列中存储的日志消息并将其写入文件。它将由一个`Thread`对象执行。

+   **Logger**：这是我们日志系统的主要类。它有一个队列，生产者将在其中存储信息，消费者将读取信息。它还包括将消息添加到队列中的方法以及获取队列中存储的所有消息并将它们写入磁盘的方法。

为了实现队列，就像缓存系统一样，我们需要一个并发数据结构来避免任何数据不一致的错误。我们有两个选择：

+   使用**阻塞数据结构**，当队列满时会阻塞线程（在我们的情况下，它永远不会满）或为空时

+   使用**非阻塞数据结构**，如果队列满或为空，则返回一个特殊值

我们选择了一个非阻塞数据结构，`ConcurrentLinkedQueue`类，它实现了`Queue`接口。我们使用`offer()`方法向队列中插入元素，使用`poll()`方法从中获取元素。

`LogTask`类的代码非常简单：

```java
public class LogTask implements Runnable {

  @Override
  public void run() {
    try {
      while (Thread.currentThread().interrupted()) {
        TimeUnit.SECONDS.sleep(10);
        Logger.writeLogs();
      }
    } catch (InterruptedException e) {
    }
    Logger.writeLogs();
  }
}
```

该类实现了`Runnable`接口，在`run()`方法中调用`Logger`类的`writeLogs()`方法，每 10 秒执行一次。

`Logger`类有五种不同的静态方法。首先是一个静态代码块，用于初始化和启动执行`LogTask`的线程，并创建用于存储日志数据的`ConcurrentLinkedQueue`类：

```java
public class Logger {

  private static ConcurrentLinkedQueue<String> logQueue = new ConcurrentLinkedQueue<String>();

  private static Thread thread;

  private static final String LOG_FILE = Paths.get("output", "server.log").toString();

  static {
    LogTask task = new LogTask();
    thread = new Thread(task);
  }
```

然后，有一个`sendMessage()`方法，它接收一个字符串作为参数，并将该消息存储在队列中。为了存储消息，它使用`offer()`方法：

```java
  public static void sendMessage(String message) {
    logQueue.offer(new Date()+": "+message);
  }
```

该类的一个关键方法是`writeLogs()`方法。它使用`ConcurrentLinkedQueue`类的`poll()`方法获取并删除队列中存储的所有日志消息，并将它们写入文件：

```java
  public static void writeLogs() {
    String message;
    Path path = Paths.get(LOG_FILE);
    try (BufferedWriter fileWriter = Files.newBufferedWriter(path,StandardOpenOption.CREATE,
        StandardOpenOption.APPEND)) {
      while ((message = logQueue.poll()) != null) {
        fileWriter.write(new Date()+": "+message);
        fileWriter.newLine();
      }
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
```

最后，两种方法：一种是截断日志文件，另一种是完成日志系统的执行器，中断正在执行`LogTask`的线程：

```java
  public static void initializeLog() {
    Path path = Paths.get(LOG_FILE);
    if (Files.exists(path)) {
      try (OutputStream out = Files.newOutputStream(path,
          StandardOpenOption.TRUNCATE_EXISTING)) {

      } catch (IOException e) {
        e.printStackTrace();
      }
    }
    thread.start();
  }
  public static void shutdown() {
    thread.interrupt();
  }
```

# 比较两种解决方案

现在是时候测试串行和并发服务器，看看哪个性能更好了。我们已经实现了四个类来自动化测试，这些类向服务器发出查询。这些类是：

+   `SerialClient`：这个类实现了一个可能的串行服务器客户端。它使用查询消息进行九次请求，并使用报告消息进行一次查询。它重复这个过程 10 次，因此它请求了 90 个查询和 10 个报告。

+   `MultipleSerialClients`：这个类模拟了同时存在多个客户端的情况。为此，我们为每个`SerialClient`创建一个线程，并同时执行它们，以查看服务器的性能。我们已经测试了从一个到五个并发客户端。

+   `ConcurrentClient`：这个类类似于`SerialClient`类，但它调用的是并发服务器，而不是串行服务器。

+   `MultipleConcurrentClients`：这个类类似于`MultipleSerialClients`类，但它调用的是并发服务器，而不是串行服务器。

要测试串行服务器，可以按照以下步骤进行：

1.  启动串行服务器并等待其初始化。

1.  启动`MultipleSerialClients`类，它启动一个、两个、三个、四个，最后是五个`SerialClient`类。

您可以使用类似的过程来测试并发服务器：

1.  启动并等待并发服务器的初始化。

1.  启动`MultipleConcurrentClients`类，它启动一个、两个、三个、四个，最后是五个`ConcurrentClient`类。

为了比较两个版本的执行时间，我们使用 JMH 框架（[`openjdk.java.net/projects/code-tools/jmh/`](http://openjdk.java.net/projects/code-tools/jmh/)）实现了一个微基准测试。我们基于`SerialClient`和`ConcurrentClient`任务实现了两次执行。我们重复这个过程 10 次，计算每个并发客户端的平均时间。我们在一个具有四个核心处理器的计算机上进行了测试，因此它可以同时执行四个并行任务。

这些执行的结果如下：

| 并发客户端 | 串行服务器 | 并发服务器 | 加速比 |
| --- | --- | --- | --- |
| 1 | 7.404 | 5.144 | 1.43 |
| 2 | 9.344 | 4.491 | 2.08 |
| 3 | 19.641 | 9.308 | 2.11 |
| 4 | 29.180 | 12.842 | 2.27 |
| 5 | 30.542 | 16.322 | 1.87 |

单元格的内容是每个客户端的平均时间（以秒为单位）。我们可以得出以下结论：

+   两种类型服务器的性能都受到并发客户端发送请求的数量的影响

+   在所有情况下，并发版本的执行时间远远低于串行版本的执行时间

# 其他有趣的方法

在本章的页面中，我们使用了 Java 并发 API 的一些类来实现执行程序框架的基本功能。这些类还有其他有趣的方法。在本节中，我们整理了其中一些。

`Executors`类提供其他方法来创建`ThreadPoolExecutor`对象。这些方法是：

+   `newCachedThreadPool()`: 此方法创建一个`ThreadPoolExecutor`对象，如果空闲，则重用工作线程，但如果有必要，则创建一个新线程。没有工作线程的最大数量。

+   `newSingleThreadExecutor()`: 此方法创建一个只使用单个工作线程的`ThreadPoolExecutor`对象。您发送到执行程序的任务将存储在队列中，直到工作线程可以执行它们。

+   `CountDownLatch`类提供以下附加方法：

+   `await(long timeout, TimeUnit unit)`: 它等待直到内部计数器到达零或者经过参数中指定的时间。如果时间过去，方法返回`false`值。

+   `getCount()`: 此方法返回内部计数器的实际值。

Java 中有两种类型的并发数据结构：

+   **阻塞数据结构**：当您调用一个方法并且库无法执行该操作（例如，尝试获取一个元素，而数据结构为空），它们会阻塞线程，直到操作可以完成。

+   **非阻塞数据结构**：当您调用一个方法并且库无法执行该操作（因为结构为空或已满）时，该方法会返回一个特殊值或抛出异常。

有些数据结构同时实现了这两种行为，有些数据结构只实现了其中一种。通常，阻塞数据结构也实现了具有非阻塞行为的方法，而非阻塞数据结构不实现阻塞方法。

实现阻塞操作的方法有：

+   `put()`, `putFirst()`, `putLast()`: 这些在数据结构中插入一个元素。如果已满，它会阻塞线程，直到有空间。

+   `take()`, `takeFirst()`, `takeLast()`: 这些返回并移除数据结构的一个元素。如果为空，它会阻塞线程，直到有元素。

实现非阻塞操作的方法有：

+   `add()`, `addFirst()`, `addLast()`: 这些在数据结构中插入一个元素。如果已满，方法会抛出`IllegalStateException`异常。

+   `remove()`, `removeFirst()`, `removeLast()`: 这些方法从数据结构中返回并移除一个元素。如果为空，方法会抛出`IllegalStateException`异常。

+   `element()`, `getFirst()`, `getLast()`: 这些从数据结构中返回但不移除一个元素。如果为空，方法会抛出`IllegalStateException`异常。

+   `offer()`, `offerFirst()`, `offerLast()`: 这些在数据结构中插入一个元素值。如果已满，它们返回`false`布尔值。

+   `poll()`, `pollFirst()`, `pollLast()`: 这些从数据结构中返回并移除一个元素。如果为空，它们返回 null 值。

+   `peek()`, `peekFirst()`, `peekLast()`: 这些从数据结构中返回但不移除一个元素。如果为空，它们返回 null 值。

在第九章中，*深入并发数据结构和同步工具*，我们将更详细地描述并发数据结构。

# 摘要

在简单的并发应用程序中，我们使用`Runnable`接口和`Thread`类执行并发任务。我们创建和管理线程并控制它们的执行。在大型并发应用程序中，我们不能采用这种方法，因为它可能会给我们带来许多问题。对于这些情况，Java 并发 API 引入了执行器框架。在本章中，我们介绍了构成此框架的基本特征和组件。首先是`Executor`接口，它定义了将`Runnable`任务发送到执行器的基本方法。该接口有一个子接口，即`ExecutorService`接口，该接口包括将返回结果的任务发送到执行器的方法（这些任务实现了`Callable`接口，正如我们将在第四章中看到的，*从任务中获取数据-Callable 和 Future 接口*）以及任务列表。

`ThreadPoolExecutor`类是这两个接口的基本实现：添加额外的方法来获取有关执行器状态和正在执行的线程或任务数量的信息。创建此类的对象的最简单方法是使用`Executors`实用程序类，该类包括创建不同类型的执行器的方法。

我们向您展示了如何使用执行器，并使用执行器实现了两个真实世界的例子，将串行算法转换为并发算法。第一个例子是 k 最近邻算法，将其应用于 UCI 机器学习存储库的银行营销数据集。第二个例子是一个客户端/服务器应用程序，用于查询世界银行的世界发展指标。

在这两种情况下，使用执行器都为我们带来了很大的性能改进。

在下一章中，我们将描述如何使用执行器实现高级技术。我们将完成我们的客户端/服务器应用程序，添加取消任务和在低优先级任务之前执行具有更高优先级的任务的可能性。我们还将向您展示如何实现定期执行任务，实现一个 RSS 新闻阅读器。
