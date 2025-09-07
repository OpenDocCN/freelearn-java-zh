

# 第八章：内存泄漏

内存泄漏是由于不当的内存管理而发生的，可以直接影响应用程序的性能。这些泄漏发生在内存被不当释放或分配但变得不可访问时。不当的内存管理不仅会负面影响性能，还会阻碍可扩展性，由于`OutOfMemoryError`而导致系统崩溃，并破坏用户体验。许多开发者隐含地相信 Java 的垃圾收集器（在第*第一章*中介绍）在应用程序运行时管理内存；然而，尽管垃圾收集器具有不可思议的能力，内存泄漏仍然是一个持续存在的问题。

垃圾收集器并没有出错；相反，当垃圾收集器无法回收不再被应用程序需要的对象所存储的内存时，就会发生内存泄漏。不当的引用是主要原因，幸运的是，我们可以避免这种情况。本章提供了避免内存泄漏的技术、设计模式、编码示例和最佳实践。

本章涵盖了以下主要内容：

+   正确引用

+   监听器和加载器

+   缓存和线程

到本章结束时，你应该对可能导致运行时内存泄漏的因素以及它们可能对我们 Java 应用程序造成的潜在破坏有一个全面的理解，并且你会知道如何有目的地和高效地预防它们。通过实验提供的示例代码，你可以对自己的内存泄漏预防策略充满信心。

# 技术要求

要遵循本章中的示例和说明，你需要具备加载、编辑和运行 Java 代码的能力。如果你还没有设置你的开发环境，请参阅*第一章*。

本章的完整代码可以在以下链接找到：[`github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter08`](https://github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter08)。

# 正确引用

不可否认——内存泄漏会导致资源可用性的逐渐下降，导致系统变慢和潜在的系统崩溃。好消息是，有两个主要组件提供了解决方案。一个组件是 Java 虚拟机（**JVM**）的一部分，即垃圾收集器。它功能强大，是 Java 语言的一个亮点。第二个，也是更重要的一点，是开发者。作为 Java 开发者，我们通过在代码中对内存管理采取有目的的方法，有权力最小化甚至消除内存泄漏。

为了支持解决方案中开发者部分的消除内存泄漏，本节将重点介绍如何正确引用对象，以防止它们导致内存泄漏，如何识别内存泄漏，以及避免它们的策略。

## 引用简介

避免内存泄漏最重要的方面可能是使用适当的引用。这应该被鼓励，因为它将控制权交给了开发者。Java 为我们提供了强大的工具箱来帮助我们在这方面的工作。具体来说，有几种引用类型，每种类型都有自己的用途和相关的垃圾回收器行为。

引用

在 Java 中，引用是内存位置的指针，是内存管理和内存泄漏缓解的关键组件。

让我们检查 Java 中各种引用类型，包括强引用、软引用、弱引用和虚引用，以便我们可以确定哪种方法最适合任何给定的用例。记住，我们的总体目的是实现高效的内存管理，避免内存链接，以提高我们 Java 应用程序的整体性能。

### 强引用

**强引用**是我们需要重点关注的最重要的引用类型。它不仅是 Java 中的默认引用类型，而且也是内存泄漏最常见的原因。具有强引用类型的对象不符合垃圾回收的条件。要创建强引用，我们只需使用变量直接引用对象即可。

在以下示例代码中，我们使用`sampleCorgiObject`变量创建了一个对象的强引用。只要该变量包含对`SampleCorgiObject`实例的引用，我们就会在内存中保留该对象，垃圾回收器将无法释放其内存：

```java
public class CH8StrongReferenceExample {
  public static void main(String[] args) {
    SampleCorgiObject sampleCorgiObject = new SampleCorgiObject();
    System.out.println(sampleCorgiObject);
  }
  static class SampleCorgiObject {
    @Override
    public String toString() {
      return "This is a SampleCorigObject instance.";
    }
  }
}
```

这之所以是 Java 中的默认引用类型，是有原因的。典型的用例是我们需要在整个应用程序运行期间保持对象，例如配置属性。我们应该谨慎使用强引用，特别是当对象很大时。一个最佳实践是在不再需要引用时立即将其设置为`null`。这将使垃圾回收器能够释放相关的内存。

### 软引用

`OutOfMemoryError`和系统崩溃。当我们创建软引用时，只有在有足够空间的情况下，这些对象才会保留在内存中。这使得软引用成为缓存大型对象的理想解决方案。

重要的是要理解，JVM 仅在必要时才会收集具有软引用的对象。JVM 的垃圾回收器首先收集所有其他可以收集的对象，然后作为最后的努力，收集具有软引用的对象，并且只有在系统内存不足的情况下才会这样做。

要实现软引用，我们使用`java.lang-ref`包中的`SoftReference`类。让我们通过代码示例来看看。

我们通过导入`SoftReference`开始我们的应用程序。然后，我们创建类头，并使用强引用通过`myBougieObject`初始化`MyBougieObject`。然后我们用`SoftReference<MyBougieObject>`包装它以建立软引用。请注意，我们将`myBougieObject`设置为 null，这确保了我们的`MyBougieObject`实例只能通过我们创建的软引用访问：

```java
import java.lang.ref.SoftReference;
public class CH8SoftReferenceExample {
  public static void main(String[] args) {
    MyBougieObject myBougieObject = new MyBougieObject("Cached 
    Object");
    SoftReference<MyBougieObject> softReference = new 
    SoftReference<>(myBougieObject);
    myBougieObject = null;
```

在下一节代码中，我们尝试从软引用中回顾`myBougieObject`。我们使用`System.gc()`来提供一个在正常条件下观察我们的软件引用行为的方法，然后模拟内存压力：

```java
    MyBougieObject retrievedObject = softReference.get();
    if (retrievedObject != null) {
        System.out.println("Object retrieved from soft reference: " 
          + retrievedObject);
    } else {
      System.out.println("The Object has been garbage collected by the 
      JVM.");
    }
    System.gc();
    retrievedObject = softReference.get();
    if (retrievedObject != null) {
      System.out.println("Object is still available after requesting 
      GC: " + retrievedObject);
    } else {
      System.out.println("The Object has been garbage collected after 
      requesting GC.");
    }
  }
}
```

这最后一部分暗示了我们的`MyBougieObject`类：

```java
class MyBougieObject {
  private String name;
  public MyBougieObject(String name) {
    this.name = name;
  }
  @Override
  public String toString() {
    return name;
  }
}
```

下面是输出可能的样子。当然，结果将取决于您系统可用内存的大小：

```java
Object retrieved from soft reference: Cached Object
Object is still available after requesting GC: Cached Object
```

### 弱引用

到目前为止，我们已经介绍了防止垃圾回收的**强引用**和允许垃圾回收作为最后手段以回收内存的**软引用**。**弱引用**的独特之处在于，只有当弱引用是特定对象的唯一引用时，才允许垃圾回收。这种方法在需要更多灵活性的内存管理解决方案中特别有用。

弱引用的一个常见用途是在缓存中，当我们希望对象保留在内存中，但又不想阻止它们在系统内存不足时被 JVM 的垃圾回收器回收。为了实现弱引用，我们使用`WeakReference`类，它是`java.lang-ref`包的一部分。让我们通过一个代码示例来看看。我们的代码从必要的`import`语句和类声明开始。正如您在下面的代码块中可以看到的，我们将`CacheCorgiObject`包装在`WeakReference`中，它最初可以通过弱引用访问。当我们将强引用（`cacheCorgiObject`）设置为 null 时，我们调用`System.gc()`来调用 JVM 的垃圾回收器。根据您的系统内存，对象可能会被回收，因为它可用。在垃圾回收之后，我们调用`weakCacheCorgiObject.get()`，如果发生了对象回收，则返回`null`：

```java
import java.lang.ref.WeakReference;
public class CH8WeakReferenceExample {
  public static void main(String[] args) {
    CacheCorgiObject cacheCorgiObject = new CacheCorgiObject();
    WeakReference<CacheCorgiObject> weakCacheCorgiObject = new 
    WeakReference<>(cacheCorgiObject);
    System.out.println("Cache corgi object before GC: " + 
    weakCacheCorgiObject.get());
    cacheCorgiObject = null;
    System.gc();
    System.out.println("Cache corgi object after GC: " + 
    weakCacheCorgiObject.get());
  }
}
class CacheCorgiObject {
@Override
  protected void finalize() {
    System.out.println("CacheCorgiObject is being garbage collected");
  }
}
```

下面是程序的一个示例输出。结果将因系统可用内存的不同而有所变化：

```java
Cache corgi object before GC: CacheCorgiObject@7344699f
Cache corgi object after GC: null
CacheCorgiObject is being garbage collected
```

### 幻影引用

我们最后一种引用类型是**幻影引用**。这种引用类型不允许直接检索引用的对象；相反，它提供了一个方法来确定对象是否已经被 JVM 垃圾回收器终结并回收。这发生时不会阻止对象被回收。

实现需要`java.lang.ref`包中的两个类——`PhantomReference`和`ReferenceQueue`。我们的示例代码演示了垃圾收集器确定具有虚引用的对象是否可达。这意味着该对象已被**终结**并准备进行垃圾回收。在这种情况下，引用将被排队，我们的应用程序能够相应地做出反应：

```java
import java.lang.ref.PhantomReference;
import java.lang.ref.ReferenceQueue;
public class CH8PhantomReferenceExample {
  public static void main(String[] args) throws InterruptedException {
    ReferenceQueue<VeryImportantResource> queue = new 
    ReferenceQueue<>();
    VeryImportantResource resource = new VeryImportantResource();
    PhantomReference<VeryImportantResource> phantomResource = new 
    PhantomReference<>(resource, queue);
    resource = null;
    System.gc();
    while (true) {
      if (queue.poll() != null) {
        System.out.println("ImportantResource has been garbage 
        collected and its phantom reference is enqueued");
        break;
      }
    }
  }
}
class VeryImportantResource {
  @Override
  protected void finalize() throws Throwable {
    System.out.println("Finalizing VeryImportantResource");
    super.finalize();
  }
}
```

采用虚引用方法使我们能够根据具有虚引用集合的对象调用我们自己的内存清理操作，这种方法不会干扰正常的垃圾回收操作。

使用`finalize()`方法

`finalize()`方法已被弃用，并计划在未来版本的 Java 中删除。在先前的代码示例中，它仅用于演示虚引用方法，并不建议使用`finalize()`。有关更多信息，请参阅 Java 文档。

现在我们已经审查了正确的引用方法，我们应该对开发者干预解决内存泄漏问题的重要性有所认识。此外，对编码示例的审查应使我们能够自信地在我们的 Java 应用程序中实现正确的引用。

## 内存泄漏识别

在我们的 Java 应用程序中，我们应该实现的最重要的事情之一是识别内存泄漏的能力。我们的目标是创建高性能的 Java 应用程序，而内存泄漏与这一目标相悖。为了识别内存泄漏，我们需要了解哪些症状表明存在内存泄漏。这些症状包括垃圾回收活动增加、`OutOfMemoryError`和性能逐渐下降。让我们看看五种识别内存泄漏的方法。

识别内存泄漏最重要的方法之一是审查我们的代码，以确保我们遵循最佳实践并避免常见陷阱，例如未能清除静态集合、使用后未移除**监听器**以及失去控制的缓存。我们将在本章后面讨论监听器：

+   第二种方法涉及使用一个工具，该工具可以揭示我们应用程序中哪个对象消耗了最多的内存。

+   第三种方法是使用工具生成堆转储，这是当前内存中所有对象的瞬间快照。这可以帮助您分析和检测潜在问题。

在审查对象保留时，寻找异常或不希望的图案。例如，当对象的生命周期比预期更长或存在大量您不期望看到的大量特定类型的对象时。

识别内存泄漏的第五种方法是持续不断地测试和评估您的应用程序。一旦您确定了内存泄漏，您就实施修复，然后应该重新测试。

我们可以使用工具来帮助我们识别内存泄漏，包括**JProfiler**、**YourKit**、**Java Flight Recorder**（**JFR**）、**VisualVM**和 Eclipse 的**内存分析工具**（**MAT**）。如果您想认真对待内存泄漏的识别，您应该研究这些工具，看看您如何利用它们的功能。

## 避免内存泄漏的策略

正确的对象引用，如本章前面所述，是避免 Java 应用程序中内存泄漏的主要策略。识别和修复内存泄漏是另一种策略，尽管它是反应性的。这两种策略都很重要，并且已经讨论过。让我们简要地看看一些额外的策略。

避免内存泄漏的第三种策略是正确管理集合对象。将对象放入集合中然后忽略或遗忘的情况并不少见，这可能导致内存泄漏。因此，为了避免这种情况，我们应该开发应用程序，使其定期删除不再需要的应用程序对象。使用弱引用可以帮助实现这一点。在使用静态集合时，我们也应该小心谨慎。这种类型的集合其生命周期与类加载器相关联。

我们还应该注意我们如何实现缓存。缓存的使用可以显著提高应用程序的性能，但也可能导致内存泄漏。在实现缓存时，我们应该使用软引用，设置有限的缓存大小限制，并持续监控缓存使用情况。

第五种策略是持续使用分析工具并测试您的应用程序。这种策略需要不断致力于检测和删除应用程序中的内存泄漏。这是一个重要的策略，不应被轻视。

当我们实施一系列避免内存泄漏的策略时，我们更有可能确保我们的应用程序具有高性能。

接下来，我们将回顾如何使用监听器和加载器来帮助避免内存泄漏。

# 监听器和加载器

我们 Java 应用程序的几个方面可能会影响性能，更具体地说，会导致内存泄漏。其中两个方面是**监听器**和**加载器**。本节专门探讨**事件监听器**和**类加载器**，并包括减轻使用它们的风险的策略，同时不牺牲它们可以为我们的 Java 应用程序提供的强大功能和效率。

## 事件监听器

事件监听器用于允许对象对事件做出反应。这种方法在交互式应用程序中得到了广泛的应用，并且是事件驱动编程的一个关键组件。这些监听器可以导致高度交互的应用程序；然而，如果管理不当，它们也可能成为内存泄漏的来源。

为了更好地理解这个问题，重要的是要注意，事件监听器必须订阅事件源，以便它们可以接收需要采取行动的通知（例如，按钮点击或游戏中的非玩家角色进入预定义区域）。正如你所期望的，每个事件监听器都维护着它们所订阅的事件源的引用。当事件源不再需要但被一个或多个监听器引用时，就会出现问题；这阻止了垃圾收集器收集事件源。

在处理事件监听器时，以下是一些避免内存泄漏的最佳实践：

+   使用弱引用，正如本章前面详细说明的那样。

+   当适用时，明确从事件源注销监听器。

+   为监听器实现静态嵌套类，因为它们没有对外部类实例的隐式引用。应使用这种方法而不是实现非静态内部类。

+   将事件监听器的生命周期与其相关的事件源对齐。

## 类加载器。

类加载器使我们能够动态加载类，使它们成为**Java 运行时环境**（**JRE**）的关键组件。类加载器通过支持多态性和可扩展性提供了巨大的力量和灵活性。当我们的应用程序动态加载类时，这表明 Java 在编译时不需要了解这些类。这种强大的灵活性带来了内存泄漏的潜在风险，我们需要减轻，如果不是消除。

JVM 有一个涉及多个类加载器类型的类加载委托模型：

+   一个**引导类加载器**，用于加载 Java 的核心 API 类。

+   `java.ext.dirs`属性。

+   `classpath`。

类加载器是必要的，当加载的类在内存中保留的时间超过必要的时间时，它们可以引入内存泄漏。这里的罪魁祸首通常是具有静态字段的类，这些字段持有应该由垃圾收集器收集的对象的引用，被具有长期生命周期的对象引用的加载类对象，以及没有适当管理的缓存保留类实例。以下是一些减轻这些风险的策略：

+   使用弱引用。

+   最小化静态字段的使用。

+   确保当不再需要时，自定义类加载器对垃圾收集器可用。

+   根据需要监控、分析和改进。

当我们对加载器和监听器以及减轻它们相关风险的战略有彻底的了解时，我们提高在 Java 应用程序中最大限度地减少内存泄漏的机会。

# 缓存和线程。

本节探讨了缓存策略、线程管理和 Java 并发工具的有效使用。这些是我们继续开发高性能 Java 应用程序旅程中需要掌握的重要概念。我们将探讨这些概念、相关的最佳实践以及减轻使用它们引入的内存泄漏风险的技巧。

## 缓存策略

在编程中，我们使用**缓存**在内存位置临时存储数据，以便允许快速访问。这允许我们重复访问数据，而不会造成延迟或系统崩溃。缓存的好处包括更负责任的应用程序和减少对长期存储解决方案（如数据库和数据库服务器）的负载。当然，也存在陷阱。如果我们没有妥善管理我们的缓存，我们可能会在我们的应用程序中引入显著的内存泄漏。

我们有多个缓存策略可以考虑；其中两个你应该已经熟悉，因为它们在本章的早期已经介绍过，尽管不是专门针对缓存。第一个熟悉策略是使用弱引用。当我们使用缓存中的弱引用时，允许在内存不足时进行垃圾回收。第二个熟悉策略是使用软引用。这种策略在垃圾回收周期中提供了更高的保留优先级。

另一种缓存策略被称为**最近最少使用**（**LRU**），正如其名所示，我们首先移除最少访问的项目，因为它们再次被我们的应用程序使用的可能性最小。

**生存时间**（**TTL**）是另一种有用的缓存策略。TTL 方法跟踪缓存插入时间，并基于规定的时间自动过期项目。

另一种缓存策略是使用基于**大小**的驱逐。这种策略确保缓存不会超过你设定的最大内存边界。这个边界可以用内存使用量或项目总数来设定。

当我们实现缓存时，我们应该注意由于实现不佳而引入内存泄漏。这种有意的做法要求我们进行容量规划，为 LRU 和 TTL 方法建立驱逐策略，并监控系统。这种监控需要后续的微调和重新测试。

## 线程管理

我们在我们的应用程序中使用**线程**来促进多个并发操作。这使现代 CPU 得到更好的利用，并提高了 Java 应用程序的响应性。当我们手动管理线程时，由于线程管理的复杂性和易出错性，我们可能会引入内存泄漏。

我们可以通过扩展`Thread`类或在 Java 中实现`Runnable`接口来创建线程。这些方法直接且易于使用线程；然而，对于大型系统来说，这不是推荐的方法，因为它们直接创建和利用线程，导致显著的开销。相反，考虑使用 Java 的**Executor**框架来从主应用程序中抽象线程管理。

线程管理的最佳实践包括以下内容：

+   优先使用执行器而非直接创建线程

+   将你使用的线程池数量限制在最小值

+   在你的代码中支持线程中断

+   持续监控和优化

让我们通过一个简单的例子来展示正确的线程使用方法。我们将使用`Executor`框架。正如您将看到的，以下应用程序创建了一个固定线程池来运行一系列任务。每个任务通过暂停一段时间来模拟现实世界的操作。在您浏览代码的过程中，您将看到`Executor`框架被有效地用于管理线程：

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
public class CH8ThreadManagementExample {
  public static void main(String[] args) {
    ExecutorService executor = Executors.newFixedThreadPool(5);
    for (int i = 0; i < 10; i++) {
      final int taskId = i;
      executor.submit(() -> {
        System.out.println("Executing task " + taskId + " Thread: " + 
        Thread.currentThread().getName());
        try {
          TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            System.out.println("Task " + taskId + " was interrupted");
        }
      });
    }
    executor.shutdown();
    try {
      if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
        executor.shutdownNow();
        System.out.println("Executor did not terminate in the 
        specified time.");
        if (!executor.awaitTermination(60, TimeUnit.SECONDS))
          System.err.println("Pool did not terminate");
      }
    } catch (InterruptedException ie) {
        executor.shutdownNow();
        Thread.currentThread().interrupt();
    }
    System.out.println("Finished all threads");
  }
}
```

此示例应用程序提供了一种在 Java 应用程序中管理并发的高效方法。

## Java 并发工具

我们很幸运，`java.util.concurrent`包包含了一组我们可以用于**并发**的实用工具。这些工具使我们能够编写线程安全的应用程序，这些应用程序是可靠和可扩展的。这些工具帮助我们解决并发编程中的常见陷阱和挑战，包括数据一致性、生命周期管理和同步。

使用`java.util.concurrent`包中包含的并发工具的优点包括使我们的应用程序更加可靠，使它们在更高的水平上运行，并简化编程工作。让我们看看五个具体的并发工具。

我们的第一个并发工具是之前提到的`Executor`框架。仔细研究这个框架可以发现，存在多种类型的 Executor 服务，包括`ScheduledExecutorService`接口。此接口可用于引入执行延迟。主要接口`ExecutorService`使我们能够管理线程终止并帮助我们跟踪同步任务。

`CountDownLatch`、`CyclicBarrier`、`Exchanger`、`Phaser`和`Semaphore`。如果您需要提高 Java 应用程序中的线程管理，这些方法在 Java 文档中值得回顾。

三个额外的并发工具是`java.util.concurrent.atomic`包。在`java.util.concurrent.locks`包中可用的锁允许我们锁定线程并等待直到满足特定条件。最后，并发集合提供了具有完整并发支持的线程安全集合。

现在我们已经探讨了缓存策略、线程管理和 Java 并发工具的有效使用，您应该已经准备好继续构建高性能的 Java 应用程序。

# 摘要

本章深入探讨了有效管理内存的复杂性，以帮助防止内存泄漏。我们必须不惜一切代价避免这些泄漏，因为它们可能会降低我们的系统性能，破坏用户体验，甚至导致系统崩溃。我们确定了内存泄漏通常是由于不当引用引起的，这阻碍了垃圾收集器释放内存的能力。我们专注于正确的引用、监听器和加载器，以及缓存和线程。现在，您应该已经准备好并自信地实施有效的内存泄漏避免策略，以应用于您的 Java 应用程序。

在下一章，*并发策略*中，我们将涵盖线程、同步、volatile、原子类、锁等概念。我们将利用本章中涵盖的线程相关内容，为我们深入研究并发提供一个先发优势。通过实践方法，您可以深入了解 Java 中的并发，并采用策略帮助您的 Java 程序性能更佳。

# 第三部分：并发和网络

并发和网络对于现代 Java 应用程序至关重要，尤其是那些需要高吞吐量和低延迟的应用程序。本部分介绍了高级并发策略来高效地管理多个线程。它还涵盖了连接池技术以优化网络性能，并探讨了超文本传输协议的复杂性。通过理解和应用这些概念，您将创建高度响应和可扩展的应用程序。

本部分包含以下章节：

+   *第九章*，*并发策略*和*模型*

+   *第十章*，*连接池*

+   *第十一章*，*超文本传输协议*
