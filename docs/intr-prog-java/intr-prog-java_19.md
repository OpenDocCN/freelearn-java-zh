# 第十九章：响应式系统

在本书的最后一章中，我们将打破连贯叙述的流程，更接近真实的专业编程。随着处理的数据越来越多，服务变得更加复杂，对更具适应性、高度可扩展和分布式应用程序的需求呈指数级增长。这就是我们将在本章中讨论的内容——这样的软件系统在实践中可能是什么样子。

在本章中，我们将涵盖以下主题：

+   如何快速处理大量数据

+   微服务

+   响应式系统

+   练习-创建`io.reactivex.Observable`

# 如何快速处理大量数据

可以应用许多可测量的性能特征到一个应用程序中。要使用哪些取决于应用程序的目的。它们通常被列为非功能性要求。最典型的集合包括以下三个：

+   **吞吐量**：单位时间内处理的请求数。

+   **延迟**：从提交请求到接收响应的*第一个*字节所经过的时间。以秒、毫秒等为单位进行测量。

+   **内存占用**：应用程序消耗的内存量——最小、最大或平均。

实际上，延迟通常被计算为吞吐量的倒数。这些特性随着负载的增长而变化，因此非功能性要求通常包括它们在平均和最大负载下的最大值。

通常，吞吐量和延迟的改进只是以内存为代价，除非增加更快的 CPU 可以改善这三个特性。但这取决于处理的性质。例如，与性能低下的设备进行输入/输出（或其他交互）可能会施加限制，代码的任何更改都无法改善应用程序性能。

对于每个特性的测量也有微妙的细微差别。例如，我们可以使用最快（最少延迟）请求中 99%的最大延迟来代替将延迟作为所有请求的平均值。否则，它看起来像是通过将亿万富翁和收入金字塔底部的人的财富除以二获得的平均财富数字。

在评估应用程序性能时，必须回答以下问题：

+   请求的延迟上限是否可能被超过？如果是，多久一次，超过多少？

+   延迟不良的时间段可以有多长，允许发生多少次？

+   谁/什么在生产中测量延迟？

+   预期的峰值负载是多少，预计会持续多长时间？

只有在回答了所有这些问题（和类似的问题），并且已经确定了非功能性要求之后，我们才能开始设计系统，测试它，微调并再次测试。有许多编程技术证明在以可接受的内存消耗实现所需的吞吐量方面是有效的。

在这种情况下，*异步*、*非阻塞*、*分布式*、*可扩展*、*响应式*、*响应式*、*弹性*和*消息驱动*等术语变得无处不在，只是*高性能*的同义词。我们将讨论每个术语，以便读者可以理解为什么会出现*微服务*和*响应式系统*，这将在本章的下两节中介绍。

# 异步

**异步**意味着请求者*立即*获得响应，但结果还没有。相反，请求者收到一个带有方法的对象，允许我们检查结果是否准备就绪。请求者定期调用此方法，当结果准备就绪时，使用同一对象上的另一个方法检索它。

这种解决方案的优势在于请求者可以在等待时做其他事情。例如，在第十一章中，*JVM 进程和垃圾回收*，我们演示了如何创建一个子线程。因此，主线程可以创建一个子线程，发送一个非异步（也称为阻塞）请求，并等待其返回，什么也不做。与此同时，主线程可以继续执行其他操作，定期调用子线程对象以查看结果是否准备好。

这是最基本的异步调用实现。事实上，当我们处理并行流时，我们已经使用了它。并行流操作在后台创建子线程，将流分成段，并将每个段分配给一个专用线程，然后将每个段的结果聚合到最终结果中。在上一章中，我们编写了执行聚合工作的函数。作为提醒，这些函数被称为组合器。

让我们比较处理顺序和并行流时相同功能的性能。

# 顺序与并行流

为了演示顺序和并行处理之间的差异，让我们想象一个从 10 个物理设备（传感器）收集数据并计算平均值的系统。这样一个系统的接口可能如下所示：

```java
interface MeasuringSystem {
    double get(String id);
}
```

它只有一个方法，`get()`，它接收传感器的 ID 并返回测量结果。使用这个接口，我们可以实现许多不同的系统，这些系统能够调用不同的设备。为了演示目的，我们不打算写很多代码。我们只需要延迟 100 毫秒（模拟从传感器收集测量数据所需的时间）并返回一些数字。我们可以这样实现延迟：

```java
void pauseMs(int ms) {
    try{
        TimeUnit.MILLISECONDS.sleep(ms);
    } catch(InterruptedException ex){
        ex.printStackTrace();
    }
}
```

至于结果数字，我们将使用`Math.random()`来模拟从不同传感器接收到的测量值的差异（这就是为什么我们需要找到一个平均值——以抵消个别设备的误差和其他特性）。因此，我们的演示实现可能如下所示：

```java
class MeasuringSystemImpl implements MeasuringSystem {
    public double get(String id){
         demo.pauseMs(100);
         return 10\. * Math.random();
    }
}
```

现在，我们意识到我们的`MeasuringInterface`是一个函数接口，因为它只有一个方法。这意味着我们可以使用`java.util.function`包中的标准函数接口之一；即`Function<String, Double>`：

```java
Function<String, Double> mSys = id -> {
    demo.pauseMs(100);
    return 10\. + Math.random();
};

```

因此，我们可以放弃我们的`MeasuringSystem`接口和`MeasuringSystemImpl`类。但是我们可以保留`mSys`（*Measuring System*）标识符，它反映了这个函数背后的想法：它代表一个提供对其传感器的访问并允许我们从中收集数据的测量系统。

现在，让我们创建一个传感器 ID 列表：

```java
List<String> ids = IntStream.range(1, 11)
        .mapToObj(i -> "id" + i).collect(Collectors.toList());
```

同样，在现实生活中，我们需要收集真实设备的 ID，但是为了演示目的，我们只是生成它们。

最后，我们将创建`collectData()`方法，它调用所有传感器并计算接收到的所有数据的平均值：

```java
Stream<Double> collectData(Stream<String> stream, 
                         Function<String, Double> mSys){
    return  stream.map(id -> mSys.apply(id));
}
```

正如你所看到的，这个方法接收一个提供 ID 的流和一个使用每个 ID 从传感器获取测量值的函数。

这是我们将如何从`averageDemo()`方法中调用这个方法，使用`getAverage()`方法：

```java
void averageDemo() {
    Function<String, Double> mSys = id -> {
         pauseMs(100);
         return 10\. + Math.random();
    };
    getAverage(() -> collectData(ids.stream(), mSys)); 
}

void getAverage(Supplier<Stream<Double>> collectData) {
    LocalTime start = LocalTime.now();
    double a = collectData.get()
                    .mapToDouble(Double::valueOf).average().orElse(0);
    System.out.println((Math.round(a * 100.) / 100.) + " in " + 
         Duration.between(start, LocalTime.now()).toMillis() + " ms");
}   

```

如您所见，我们创建了代表测量系统的函数，并将其传递给`collectData()`方法，以及 ID 流。然后，我们创建了`SupplierStream<Double>>`函数作为`() -> collectData(ids.stream(), mSys)` lambda 表达式，并将其作为`collectData`参数传递给`getAverage()`方法。在`getAverage()`方法内部，我们调用供应商的`get()`，从而调用`collectData(ids.stream(), mSys)`，它返回`Stream<Double>`。然后我们使用`mapToDouble()`操作将其转换为`DoubleStream`，以便应用`average()`操作。`average()`操作返回一个`Optional<Double>`对象，我们调用它的`orElse(0)`方法，它返回计算出的值或零（例如，如果测量系统无法连接到任何传感器并返回空流）。`getAverage()`方法的最后一行打印了结果和计算所需的时间。在实际代码中，我们会返回结果并将其用于其他计算。但是对于我们的演示，我们只是打印它。

现在，我们可以将顺序流处理的性能与并行流处理进行比较：

```java
List<String> ids = IntStream.range(1, 11)
              .mapToObj(i -> "id" + i).collect(Collectors.toList());
Function<String, Double> mSys = id -> {
        pauseMs(100);
        return 10\. + Math.random();
};
getAverage(() -> collectData(ids.stream(), mSys));    
                                             //prints: 10.46 in 1031 ms
getAverage(() -> collectData(ids.parallelStream(), mSys));  
                                             //prints: 10.49 in 212 ms

```

如您所见，处理并行流比处理顺序流快五倍。

尽管在幕后，并行流使用了`\`异步处理，但这并不是程序员在谈论异步处理请求时所指的。从应用程序的角度来看，它只是并行（也称为并发）处理。它比顺序处理快，但主线程必须等待所有调用完成并检索所有数据。如果每个调用至少需要 100 毫秒（就像我们的情况一样），那么所有调用的处理时间不可能少于这个时间。

当然，我们可以创建一个子线程，让它进行所有调用，并等待它们完成，而主线程则做其他事情。我们甚至可以创建一个执行此操作的服务，因此应用程序只需告诉这样的服务要做什么（在我们的情况下传递传感器 ID）并继续做其他事情。稍后，主线程可以再次调用服务，并获取结果或在约定的地方获取结果。这就是程序员所说的真正的异步处理。

但在编写这样的代码之前，让我们看看位于`java.util.concurrent`包中的`CompletableFuture`类。它可以做我们描述的一切，甚至更多。

# 使用 CompletableFuture 类

使用`CompletableFuture`对象，我们可以将向测量系统发送数据请求（并创建`CompletableFuture`对象）与从`CompletableFuture`对象获取结果分开。这正是我们在解释异步处理时描述的场景。让我们在代码中演示它。类似于我们提交请求到测量系统的方式，我们可以使用`CompletableFuture.supplyAsync()`静态方法来完成：

```java
List<CompletableFuture<Double>> list = ids.stream()
        .map(id -> CompletableFuture.supplyAsync(() -> mSys.apply(id)))
        .collect(Collectors.toList());

```

不同之处在于`supplyAsync()`方法不会等待调用测量系统返回。相反，它立即创建一个`CompletableFuture`对象并返回，以便客户端可以随时使用该对象检索测量系统返回的值。还有一些方法可以让我们检查值是否已经返回，但这不是这个演示的重点，重点是展示`CompletableFuture`类如何用于组织异步处理。

创建的`CompletableFuture`对象列表可以存储在任何地方。我们选择将其存储在一个`Map`中。事实上，我们创建了一个`sendRequests()`方法，可以向任意数量的测量系统发送任意数量的请求：

```java
Map<Integer, List<CompletableFuture<Double>>> 
                  sendRequests(List<List<String>> idLists, 
                               List<Function<String, Double>> mSystems){
   LocalTime start = LocalTime.now();
   Map<Integer, List<CompletableFuture<Double>>> requests 
                                                       = new HashMap<>();
   for(int i = 0; i < idLists.size(); i++){
      for(Function<String, Double> mSys: mSystems){
         List<String> ids = idLists.get(i);
         List<CompletableFuture<Double>> list = ids.stream()
          .map(id -> CompletableFuture.supplyAsync(() -> mSys.apply(id)))
          .collect(Collectors.toList());
         requests.put(i, list);
      }
   }
   long dur = Duration.between(start, LocalTime.now()).toMillis();
   System.out.println("Submitted in " + dur + " ms");
   return requests;
}
```

正如您所看到的，前面的方法接受了两个参数：

+   `List<List<String>> idLists`：传感器 ID 列表的集合（列表），每个列表特定于特定的测量系统。

+   `List<Function<String, Double>> mSystems`：测量系统的列表，每个系统都表示为`Function<String, Double>`，具有一个接受传感器 ID 并返回双精度值（测量结果）的`apply()`方法。此列表中的系统与第一个参数中的传感器 ID 列表的顺序相同，因此我们可以通过它们的位置将 ID 与系统匹配。

然后，我们创建了一个`Map<Integer, List<CompletableFuture<Double>>>`对象来存储`CompletableFuture`对象的列表。我们在`for`循环中生成它们，然后将它们存储在一个带有顺序号的`Map`中。`Map`被返回给客户端，可以存储在任何地方，任意时间段（好吧，有一些可以修改的限制，但我们不打算在这里讨论它们）。稍后，当客户端决定获取请求的结果时，可以使用`getAverage()`方法来检索它们：

```java
void getAverage(Map<Integer, List<CompletableFuture<Double>>> requests){
    for(List<CompletableFuture<Double>> list: requests.values()){
        getAverage(() -> list.stream().map(CompletableFuture::join));
    }
}
```

前面的方法接受了`sendRequests()`方法创建的`Map`对象，并迭代存储在`Map`中的所有值（`CompletableFuture`对象的列表）。对于每个列表，它创建一个流，将每个元素（`CompletableFuture`对象）映射到调用该元素的`join()`方法的结果。此方法检索从相应调用测量系统返回的值。如果值不可用，该方法会等待一段时间（可配置的值），然后要么退出（并返回`null`），要么最终接收来自测量系统的值（如果可用）。同样，我们不打算讨论围绕故障的所有保护措施，以便专注于主要功能。

`()-> list.stream().map(CompletableFuture::join)`函数实际上被传递到`getAverage()`方法中（这对您来说应该是熟悉的），我们在前面的示例中处理流时使用过：

```java
void getAverage(Supplier<Stream<Double>> collectData) {
    LocalTime start = LocalTime.now();
    double a = collectData.get()
                    .mapToDouble(Double::valueOf).average().orElse(0);
    System.out.println((Math.round(a * 100.) / 100.) + " in " + 
         Duration.between(start, LocalTime.now()).toMillis() + " ms");
}
```

这个方法计算传入流发出的所有值的平均值，打印出来，并且还捕获了处理流（和计算平均值）所花费的时间。

现在，让我们使用新的方法，看看性能如何提高：

```java
Function<String, Double> mSys = id -> {
     pauseMs(100);
     return 10\. + Math.random();
 };
 List<Function<String, Double>> mSystems = List.of(mSys, mSys, mSys);
 List<List<String>> idLists = List.of(ids, ids, ids);

 Map<Integer, List<CompletableFuture<Double>>> requestLists = 
        sendRequests(idLists, mSystems);  //prints: Submitted in 13 ms

 pauseMs(2000);  //The main thread can continue doing something else
                 //for any period of time
 getAverage(requestLists);               //prints: 10.49 in 5 ms
                                         //        10.61 in 0 ms
                                         //        10.51 in 0 ms

```

为了简单起见，我们重用了相同的测量系统（及其 ID）来模拟与三个测量系统一起工作。您可以看到所有三个系统的请求在 13 毫秒内提交。`sendRequests()`方法存在，主线程至少有两秒的空闲时间去做其他事情。这是实际发送所有请求并接收响应所需的时间，因为每次调用测量系统都使用`pauseMs(100)`。然后，我们为每个系统计算平均值，几乎不需要时间。这就是程序员在谈论异步处理请求时的意思。

`CompletableFuture`类有许多方法，并且得到了几个其他类和接口的支持。例如，使用线程池可以减少收集所有数据的两秒暂停时间：

```java
Map<Integer, List<CompletableFuture<Double>>> 
                  sendRequests(List<List<String>> idLists, 
                               List<Function<String, Double>> mSystems){
   ExecutorService pool = Executors.newCachedThreadPool();
   LocalTime start = LocalTime.now();
   Map<Integer, List<CompletableFuture<Double>>> requests 
                                                       = new HashMap<>();
   for(int i = 0; i < idLists.size(); i++){
      for(Function<String, Double> mSys: mSystems){
         List<String> ids = idLists.get(i);
         List<CompletableFuture<Double>> list = ids.stream()
          .map(id -> CompletableFuture.supplyAsync(() -> mSys.apply(id), 
 pool))
          .collect(Collectors.toList());
         requests.put(i, list);
      }
   }
   pool.shutdown();
   long dur = Duration.between(start, LocalTime.now()).toMillis();
   System.out.println("Submitted in " + dur + " ms");
   return requests;
}
```

有各种各样的这样的池，用于不同的目的和不同的性能。但所有这些都不会改变整体系统设计，因此我们将忽略这些细节。

所以，异步处理的威力是巨大的。但谁从中受益呢？

如果您创建了一个应用程序，根据需要收集数据并计算每个测量系统的平均值，那么从客户端的角度来看，仍然需要很长时间，因为暂停（两秒，或者如果我们使用线程池则更少）仍然包括在客户端的等待时间中。因此，除非您设计了 API，以便客户端可以提交请求并离开做其他事情，然后稍后获取结果，否则客户端将失去异步处理的优势。

这就是*同步*（或*阻塞）API 和*异步*API 之间的区别，当客户端等待（阻塞）直到结果返回时，以及当客户端提交请求并离开做其他事情，然后稍后获得结果时。

异步 API 的可能性增强了我们对延迟的理解。通常，程序员所说的延迟是指在同一次调用 API 时，从提交请求到接收到响应的第一个字节所花费的时间。但如果 API 是异步的，延迟的定义就会变成“请求提交和结果可供客户端收集的时间”。在这种情况下，每次调用的延迟被假定要比发出请求和收集结果之间的时间要小得多。

还有一个*非阻塞*API 的概念，我们将在下一节讨论。

# 非阻塞

对于应用程序的客户端来说，非阻塞 API 的概念只告诉我们应用程序可能是可扩展的、反应灵敏的、响应快速的、具有弹性的和消息驱动的。在接下来的章节中，我们将讨论所有这些术语，但现在，我们希望您可以从这些名称本身中得出它们各自的含义。

这样的陈述意味着两件事：

+   非阻塞不会影响客户端和应用程序之间的通信协议：它可以是同步（阻塞）或异步的。非阻塞是一个实现细节；它是从应用程序内部的 API 视角来看的。

+   非阻塞是帮助应用程序成为以下所有特性的实现：可扩展、反应灵敏、响应快速、具有弹性和消息驱动。这意味着它是许多现代应用程序基础的一个非常重要的设计概念。

众所周知，阻塞 API 和非阻塞 API 并不是对立的。它们描述了应用程序的不同方面。阻塞 API 描述了客户端与之交互的方式：客户端调用并保持连接，直到提供响应。非阻塞 API 描述了应用程序的实现方式：它不为每个请求分配执行线程，而是提供多个轻量级工作线程，以异步和并发的方式进行处理。

非阻塞这个术语是随着提供对密集输入/输出（I/O）操作支持的`java.nio`（NIO 代表非阻塞输入/输出）包的使用而出现的。

# java.io 与 java.nio 包

向外部存储器（例如硬盘）写入和读取数据比在内存中进行的其他进程要慢得多。`java.io`包中已经存在的类和接口运行良好，但偶尔性能会出现瓶颈。新的`java.nio`包被创建出来以提供更有效的 I/O 支持。

`java.io`的实现是基于流处理的，正如我们在前一节中看到的，即使在幕后进行了某种并发操作，它基本上仍然是一个阻塞操作。为了提高速度，`java.nio`的实现是基于在内存中读取/写入缓冲区。这样的设计使我们能够将填充/清空缓冲区的缓慢过程与从中快速读取/写入的过程分开。在某种程度上，这类似于我们在`CompletableFuture`类使用示例中所做的。拥有缓冲区中的数据的额外优势是可以检查它，来回沿着缓冲区进行操作，而从流中顺序读取时是不可能的。这使得在数据处理过程中更加灵活。

此外，`java.nio`实现引入了另一个中间过程，称为通道，它提供了与缓冲区的批量数据传输。读取线程从通道获取数据，并且只接收当前可用的数据，或者根本没有数据（如果通道中没有数据）。如果数据不可用，线程可以做其他事情，而不是保持阻塞状态，例如读取/写入其他通道。就像我们的`CompletableFuture`示例中的主线程在测量系统从传感器中读取数据时可以自由进行其他操作。这样，与将一个线程专用于一个 I/O 进程不同，几个工作线程可以为多个 I/O 进程提供服务。

这样的解决方案被称为非阻塞 I/O，后来被应用于其他进程，其中最突出的是事件循环中的事件处理，也称为运行循环。

# 事件循环，或运行循环

许多非阻塞处理系统都基于事件（或运行）循环——一个不断执行的线程，接收事件（请求、消息），然后将它们分派给相应的*事件处理程序*。事件处理程序没有什么特别之处。它们只是由程序员专门用于处理特定事件类型的方法（函数）。

这种设计被称为*反应器设计模式*，定义为*用于处理并发传递给服务处理程序的服务请求的事件处理模式*。它还为*反应式编程*和*反应式系统*提供了名称，这些系统对某些事件做出*反应*并相应地处理它们。我们将在专门的部分中稍后讨论反应式系统。

基于事件循环的设计在操作系统和图形用户界面中被广泛使用。它在 Spring 5 的 Spring WebFlux 中可用，并在 JavaScript 及其流行的执行环境 Node.js 中实现。最后一个使用事件循环作为其处理骨干。Vert.x 工具包也是围绕事件循环构建的。我们将在“微服务”部分展示后者的一些示例。

在采用事件循环之前，每个传入请求都分配了一个专用线程，就像我们在流处理演示中所做的那样。每个线程都需要分配一定数量的资源，这些资源与请求无关，因此一些资源（主要是内存分配）被浪费了。然后，随着请求数量的增加，CPU 需要更频繁地切换上下文，以允许更多或更少的并发处理所有请求。在负载下，上下文切换的开销变得足够大，以至于影响应用程序的性能。

实现事件循环解决了这两个问题：

+   它通过避免为每个请求创建一个专用线程并保持线程直到请求被处理，从而消除了资源的浪费。有了事件循环，每个请求只需要一个更小的内存分配来捕获其具体信息。这使得可以在内存中保留更多的请求，以便可以并发处理它们。

+   CPU 上下文切换的开销也变得更小了，因为上下文大小减小了。

非阻塞 API 是如何实现请求处理的。有了它，系统能够处理更大的负载（更具可伸缩性和弹性），同时保持高度的响应和弹性。

# 分布式

随着时间的推移，分布式的概念也发生了变化。它曾经意味着在多台计算机上运行的应用程序，通过网络连接。它甚至有一个同义词叫做并行计算，因为应用程序的每个实例都在做同样的事情。这样的应用程序提高了系统的弹性。一台计算机的故障不会影响整个系统。

然后，又添加了另一层含义：一个应用程序分布在多台计算机上，因此其每个组件都对应用程序整体产生的结果有所贡献。这种设计通常用于需要大量 CPU 计算能力或需要来自许多不同来源的大量数据的计算或数据密集型任务。

当单个 CPU 变得足够强大，可以处理成千上万台旧计算机的计算负载，尤其是云计算，特别是像 AWS Lambda 无服务器计算平台这样的系统，它们完全消除了个人计算机的概念；*分布式*可能意味着一个应用程序或其组件在一个或多台计算机上运行的任何组合。

分布式系统的例子包括大数据处理系统、分布式文件或数据存储系统以及分类帐系统，如区块链或比特币，也可以包括在*智能*数据存储系统的子类别下的数据存储系统组中。

当程序员今天称一个系统为*分布式*时，他们通常指的是以下内容：

+   系统可以容忍其构成组件的一个或甚至多个失败。

+   每个系统组件只能看到系统的有限不完整视图。

+   系统的结构是动态的，并且在执行过程中可能会发生变化。

+   系统是可扩展的。

# 可扩展

可扩展性是在不显著降低延迟/吞吐量的情况下承受不断增加的负载的能力。传统上，这是通过将软件系统分解为层来实现的：前端层、中间层和后端层，例如。每个层由负责特定类型处理的相同组件组的多个部署副本组成。

前端组件负责基于请求和从中间层接收到的数据进行呈现。中间层组件负责基于来自前端层的数据和它们可以从后端层读取的数据进行计算和决策。它们还将数据发送到后端进行存储。后端层存储数据，并将其提供给中间层。

通过添加组件的副本，每个层允许我们跟上不断增加的负载。过去，只能通过向每个层添加更多计算机来实现。否则，新部署的组件副本将没有可用资源。

但是，随着云计算的引入，尤其是 AWS Lambda 服务，可扩展性是通过仅添加软件组件的新副本来实现的。增加了更多计算机到层中（或者没有）对部署者来说是隐藏的。

分布式系统架构中的另一个最近的趋势允许我们通过扩展不仅通过层，而且通过特定的小型功能部分来微调可扩展性，并提供一种或多种特定类型的服务，称为微服务。我们将在*微服务*部分讨论这一点，并展示一些微服务的示例。

在这样的架构下，软件系统变成了许多微服务的组合；每个微服务可以根据需要复制多次，以支持所需的处理能力增加。在这个意义上，我们只能谈论一个微服务的可扩展性。

# 反应式

术语*反应式*通常用于反应式编程和反应式系统的上下文中。反应式编程（也称为 Rx 编程）是基于使用异步数据流（也称为反应式流）进行编程。它在 Java 9 中引入了`java.util.concurrent`包。它允许`Publisher`生成数据流，`Subscriber`可以异步订阅。

正如您所见，即使没有这个新的 API，我们也能够异步处理数据，使用`CompletableFuture`。但是，写了几次这样的代码后，人们会注意到其中大部分只是管道工作，因此人们会觉得一定有更简单、更方便的解决方案。这就是 Reactive Streams 倡议([`www.reactive-streams.org`](http://www.reactive-streams.org))的诞生。该努力的范围定义如下：

Reactive Streams 的范围是找到一组最小的接口、方法和协议，描述必要的操作和实体，以实现异步数据流和非阻塞背压。

术语*非阻塞背压*指的是异步处理的问题之一——协调传入数据的速率与系统处理数据的能力，而无需停止（阻塞）数据输入。解决方案是通知源，消费者在跟上输入的速率方面有困难，但处理应该对传入数据速率的变化做出更灵活的反应，而不仅仅是阻塞流（因此称为反应式）。

除了标准的 Java 库，已经存在几个实现了 Reactive Streams API 的其他库：RxJava、Reactor、Akka Streams 和 Vert.x 是其中最知名的。我们将在我们的示例中使用 RxJava 2.1.13。您可以在[`reactivex.io`](http://reactivex.io)找到 RxJava 2.x API，名称为 ReactiveX，代表 Reactive Extension。

让我们首先比较使用`java.util.stream`包和 RxJava 2.1.13 的`io.reactivex`包实现相同功能的两种方式，可以通过以下依赖项添加到项目中：

```java
<dependency>
    <groupId>io.reactivex.rxjava2</groupId>
    <artifactId>rxjava</artifactId>
    <version>2.1.13</version>
</dependency> 
```

示例程序将非常简单：

+   创建一个整数流：1、2、3、4、5。

+   仅过滤偶数（2 和 4）。

+   计算每个过滤后的数字的平方根。

+   计算所有平方根的和。

以下是使用`java.util.stream`包实现的方式：

```java
double a = IntStream.rangeClosed(1, 5)
        .filter(i -> i % 2 == 0)
        .mapToDouble(Double::valueOf)
        .map(Math::sqrt)
        .sum();
System.out.println(a); //prints: 3.414213562373095

```

使用 RxJava 实现相同功能的方式如下：

```java
Observable.range(1, 5)
        .filter(i -> i % 2 == 0)
        .map(Math::sqrt)
        .reduce((r, d) -> r + d)
        .subscribe(System.out::println); //prints: 3.414213562373095
RxJava is based on the Observable object (which plays the role of Publisher) and Observer that subscribes to the Observable and waits for data to be emitted. 
```

除了`Stream`功能外，`Observable`具有显著不同的功能。例如，流一旦关闭，就无法重新打开，而`Observable`对象可以再次使用。这是一个例子：

```java
Observable<Double> observable = Observable.range(1, 5)
        .filter(i -> i % 2 == 0)
        .doOnNext(System.out::println)    //prints 2 and 4 twice
        .map(Math::sqrt);
observable
        .reduce((r, d) -> r + d)
        .subscribe(System.out::println);  //prints: 3.414213562373095
observable
        .reduce((r, d) -> r + d)a
        .map(r -> r / 2)
        .subscribe(System.out::println);  //prints: 1.7071067811865475
```

在前面的示例中，从注释中可以看出，`doOnNext()`操作被调用了两次，这意味着`observable`对象发出了两次值。但是，如果我们不希望`Observable`运行两次，我们可以通过添加`cache()`操作来缓存其数据：

```java
Observable<Double> observable = Observable.range(1,5)
        .filter(i -> i % 2 == 0)
        .doOnNext(System.out::println)  //prints 2 and 4 only once
        .map(Math::sqrt)
        .cache();
observable
        .reduce((r, d) -> r + d)
        .subscribe(System.out::println); //prints: 3.414213562373095
observable
        .reduce((r, d) -> r + d)
        .map(r -> r / 2)
        .subscribe(System.out::println);  //prints: 1.7071067811865475

```

正如你所看到的，同一个`Observable`的第二次使用利用了缓存数据，从而提高了性能。`Observable`接口和 RxJava 中还有更多功能，但本书的格式不允许我们进行描述。但我们希望你能理解。

使用 RxJava 或其他异步流库编写代码构成了反应式编程。它实现了反应式宣言中所宣布的目标，即构建具有响应性、弹性、弹性和消息驱动的反应式系统。

# 响应式

这个术语似乎是不言自明的。及时响应的能力是每个客户对任何系统的首要要求之一。可以通过许多不同的方法来实现这一点。即使传统的阻塞 API 也可以通过足够的服务器和其他基础设施来支持，以在非常大的负载下提供预期的响应性。反应式编程只是帮助使用更少的硬件来实现这一点。

这是有代价的，因为反应式代码需要改变我们过去的做法，甚至是五年前的做法。但过一段时间，这种新的思维方式就会变得和任何其他已经熟悉的技能一样自然。我们将在接下来的章节中看到更多反应式编程的例子。

# 弹性

失败是不可避免的。硬件崩溃，软件有缺陷，接收到意外数据，或者采取了意外和未经充分测试的执行路径——任何这些事件或它们的组合都可能随时发生。弹性是系统在这种情况下继续提供预期结果的能力。

可以通过部署组件和硬件的冗余、系统各部分的隔离（减少多米诺效应的可能性）、设计系统使得丢失的部分可以自动替换或者引发适当的警报以便合格人员干预等措施来实现。

我们已经谈论过分布式系统。这样的架构通过消除单点故障使系统更具弹性。此外，将系统分解为许多专门的组件，并使用消息相互通信，可以更好地调整最关键部分的复制，并为其隔离和潜在故障容纳创造更多机会。

# 弹性

承受最大负载的能力通常与可伸缩性相关。但在不同负载下保持相同性能特征的能力被称为弹性。弹性系统的客户不应该注意到空闲时期和高峰负载时期之间的任何差异。

非阻塞的反应式实现风格有助于实现这一质量。此外，将程序分解为更小的部分并将其转换为可以独立部署和管理的服务，可以进行资源分配的微调。这些小服务被称为微服务，许多微服务可以组成一个既可扩展又具有弹性的反应式系统。我们将在接下来的章节中更详细地讨论这些解决方案。

# 消息驱动

我们已经确定了组件的隔离和系统分布是保持系统响应、弹性和弹性的两个方面。松散和灵活的连接也是支持这些特性的重要条件。而反应式系统的异步性质简单地不给设计者留下其他选择，只能在组件之间建立消息通信。

它为每个组件创建了一个“呼吸空间”，没有这个空间，系统将成为一个紧密耦合的单体，容易受到各种问题的影响，更不用说维护上的噩梦了。

有了这些，我们将研究可以用来构建应用程序的架构风格，作为提供所需业务功能的松散耦合服务的集合——微服务。

# 微服务

为了使一个可部署的代码单元有资格成为微服务，它必须具备以下特征：

+   一个微服务的源代码大小应该小于传统应用程序的大小。另一个大小标准是一个程序员团队应该能够编写和支持其中的几个。

+   它必须能够独立部署。通常，一个微服务通常会合作并期望其他系统的合作，但这不应该妨碍我们部署它的能力。

+   如果一个微服务使用数据库存储数据，它必须有自己的模式或一组表。这个说法仍在争论中，特别是在几个服务修改相同数据集或相互依赖的数据集的情况下。如果同一个团队拥有所有相关服务，那么更容易实现。否则，有几种可能的策略来确保独立的微服务开发和部署。

+   它必须是无状态的，即其状态不应保存在内存中，除非内存是共享的。如果服务的一个实例失败了，另一个实例应该能够完成服务所期望的工作。

+   它应该提供一种检查其*健康*的方式——即服务是否正常运行并准备好执行工作。

说到这里，让我们来看看微服务实现的工具包领域。一个人肯定可以从头开始编写微服务，但在这之前，值得看看已经存在的东西，即使你发现没有什么符合你特定需求的。

两个最流行的工具包是 Spring Boot（[`projects.spring.io/spring-boot`](https://projects.spring.io/spring-boot)）和原始的 J2EE。J2EE 社区成立了 MicroProfile（[`microprofile.io`](https://microprofile.io)）倡议，旨在优化企业 Java 以适应微服务架构。KumuluzEE（[`ee.kumuluz.com`](https://ee.kumuluz.com)）是一个轻量级的符合 MicroProfile 标准的开源微服务框架。

一些其他框架、库和工具包的列表如下（按字母顺序排列）：

+   **Akka**：用于构建高并发、分布式和具有弹性的 Java 和 Scala 消息驱动应用程序的工具包（[`akka.io/`](https://akka.io/)）。

+   **Bootique**：用于可运行 Java 应用程序的最小化框架（[`bootique.io/`](https://bootique.io/)）。

+   **Dropwizard**：用于开发友好运维、高性能、RESTful Web 服务的 Java 框架（[`www.dropwizard.io/`](https://www.dropwizard.io/)）。

+   **Jodd**：一组 Java 微框架、工具和实用程序，不到 1.7 MB（[`jodd.org/`](https://jodd.org/)）。

+   **Lightbend Lagom**：基于 Akka 和 Play 构建的一种倾向性微服务框架（[`www.lightbend.com/`](https://www.lightbend.com/)）。

+   **Ninja**：用于 Java 的全栈 Web 框架（[`www.ninjaframework.org/`](http://www.ninjaframework.org/)）。

+   **Spotify Apollo**：Spotify 用于编写微服务的一组 Java 库（[`spotify.github.io/apollo/`](http://spotify.github.io/apollo/)）。

+   **Vert.x**：用于在 JVM 上构建反应式应用程序的工具包（[`vertx.io/`](https://vertx.io/)）。

所有列出的框架、库和工具包都支持微服务之间的 HTTP/JSON 通信。其中一些还有额外的消息发送方式。如果没有，可以使用任何轻量级的消息系统。我们在这里提到它，因为你可能还记得，基于消息驱动的异步处理是由微服务组成的反应式系统的弹性、响应性和韧性的基础。

为了演示微服务构建的过程，我们将使用 Vert.x，这是一个事件驱动的非阻塞轻量级多语言工具包（组件可以用 Java、JavaScript、Groovy、Ruby、Scala、Kotlin 或 Ceylon 编写）。它支持异步编程模型和分布式事件总线，可达到浏览器 JavaScript，从而实现实时 Web 应用程序的创建。

# Vert.x 基础知识

在 Vert.x 世界中的构建块是实现`io.vertx.core.Verticle`接口的类：

```java
package io.vertx.core;
public interface Verticle {
  Vertx getVertx();
  void init(Vertx vertx, Context context);
  void start(Future<Void> future) throws Exception;
  void stop(Future<Void> future) throws Exception;
}
```

上述接口的实现称为垂直线。上述接口的大多数方法名称都是不言自明的。`getVertex()`方法提供对`Vertx`对象的访问——这是进入 Vert.x Core API 的入口点，该 API 具有允许我们构建微服务构建所需的以下功能的方法：

+   创建 DNS 客户端

+   创建周期性服务

+   创建数据报套接字

+   部署和取消部署垂直线

+   提供对共享数据 API 的访问

+   创建 TCP 和 HTTP 客户端和服务器

+   提供对事件总线和文件系统的访问

所有部署的垂直线都可以通过标准的 HTTP 协议或使用`io.vertx.core.eventbus.EventBus`相互通信，形成一个微服务系统。我们将展示如何使用垂直线和来自`io.vertx.rxjava`包的 RxJava 实现构建一个响应式微服务系统。

可以通过扩展`io.vertx.rxjava.core.AbstractVerticle`类轻松创建`Verticle`接口实现：

```java
package io.vertx.rxjava.core;
import io.vertx.core.Vertx;
import io.vertx.core.Context;
import io.vertx.core.AbstractVerticle
public class AbstractVerticle extends AbstractVerticle {
   protected io.vertx.rxjava.core.Vertx vertx;
   public void init(Vertx vertx, Context context) {
      super.init(vertx, context);
      this.vertx = new io.vertx.rxjava.core.Vertx(vertx);
   } 
}
```

如您所见，上述类扩展了`io.vertx.core.AbstractVerticle`类：

```java
package io.vertx.core;
import java.util.List;
import io.vertx.core.Verticle;
import io.vertx.core.json.JsonObject;
public abstract class AbstractVerticle implements Verticle {
   protected Vertx vertx;
   protected Context context;
   public void init(Vertx vertx, Context context) {
      this.vertx = vertx;
      this.context = context;
   }
   public Vertx getVertx() { return vertx; }
   public JsonObject config() { return context.config(); }
   public String deploymentID() { return context.deploymentID(); }
   public List<String> processArgs() { return context.processArgs(); }
   public void start(Future<Void> startFuture) throws Exception {
      start();
      startFuture.complete();
   }
   public void stop(Future<Void> stopFuture) throws Exception {
      stop();
      stopFuture.complete();
   }
   public void start() throws Exception {}
   public void stop() throws Exception {}
}
```

如您所见，您只需要扩展`io.vertx.rxjava.core.AbstractVerticle`类并实现`start()`方法。新的垂直线将是可部署的，即使没有实现`start()`方法，但它将不会执行任何有用的操作。`start()`方法中的代码是应用功能的入口点。

要使用 Vert.x 并执行示例，必须将以下依赖项添加到项目中：

```java
<dependency>
    <groupId>io.vertx</groupId>
    <artifactId>vertx-web</artifactId>
    <version>${vertx.version}</version>
</dependency>
<dependency>
    <groupId>io.vertx</groupId>
    <artifactId>vertx-rx-java</artifactId>
    <version>${vertx.version}</version>
</dependency>

```

`vertx.version`属性可以在`pom.xml`文件的`properties`部分中设置：

```java
<properties>
    <vertx.version>3.5.1</vertx.version>
</properties>

```

使垂直反应的是事件循环（线程）的基础实现，它接收事件（请求）并将其传递给处理程序 - 垂直中的方法或另一个专用类，该类正在处理此类型的事件。程序员通常将它们描述为与每种事件类型关联的函数。当处理程序返回时，事件循环调用回调，实现了我们在上一节中讨论的反应器模式。

对于某些天生具有阻塞性质的程序（例如 JDBC 调用或长时间计算），可以通过工作人员垂直异步执行，而不是通过事件循环（因此不会阻塞它），而是通过单独的线程，使用`vertx.executeBlocking()`方法。基于事件循环的应用程序设计的黄金法则是，*不要阻塞事件循环！*违反此规则会使应用程序停滞不前。

# 作为微服务的 HTTP 服务器

例如，这是一个充当 HTTP 服务器的垂直：

```java
package com.packt.javapath.ch18demo.microservices;
import io.vertx.rxjava.core.AbstractVerticle;
import io.vertx.rxjava.core.http.HttpServer;
public class HttpServer1 extends AbstractVerticle{
   private int port;
   public HttpServer1(int port) {
       this.port = port;
   }
   public void start() throws Exception {
      HttpServer server = vertx.createHttpServer();
      server.requestStream().toObservable()
         .subscribe(request -> request.response()
             .end("Hello from " + Thread.currentThread().getName() + 
                                         " on port " + port + "!\n\n"));
      server.rxListen(port).subscribe();
      System.out.println(Thread.currentThread().getName() + 
                                 " is waiting on port " + port + "...");
   }
}
```

在上述代码中，创建了服务器，并将可能请求的数据流包装成`Observable`。由`Observable`发出的数据传递给处理请求并生成必要响应的函数（请求处理程序）。我们还告诉服务器要监听的端口，并且现在可以部署此垂直的多个实例，以侦听不同的端口：

```java
vertx().getDelegate().deployVerticle(new HttpServer1(8082));
vertx().getDelegate().deployVerticle(new HttpServer1(8083));
```

还有一个`io.vertx.rxjava.core.RxHelper`助手类，可用于部署。它处理了一些对当前讨论不重要的细节：

```java
RxHelper.deployVerticle(vertx(), new HttpServer1(8082));
RxHelper.deployVerticle(vertx(), new HttpServer1(8083));

```

无论使用哪种方法，您都将看到以下消息：

```java
vert.x-eventloop-thread-0 is waiting on port 8082...
vert.x-eventloop-thread-0 is waiting on port 8083...
```

这些消息确认了我们的预期：同一事件循环线程正在两个端口上监听。现在，我们可以使用标准的`curl`命令向任何正在运行的服务器发送请求：

```java
curl localhost:8082
```

响应将是我们硬编码的响应：

```java
Hello from vert.x-eventloop-thread-0 on port 8082!
```

# 周期性服务作为微服务

Vert.x 还允许我们创建一个定期服务，该服务会定期执行某些操作。这是一个例子：

```java
package com.packt.javapath.ch18demo.microservices;
import io.vertx.rxjava.core.AbstractVerticle;
import java.time.LocalTime;
import java.time.temporal.ChronoUnit;
public class PeriodicService1 extends AbstractVerticle {
  public void start() throws Exception {
     LocalTime start = LocalTime.now();
     vertx.setPeriodic(1000, v-> {
         System.out.println("Beep!");
         if(ChronoUnit.SECONDS.between(start, LocalTime.now()) > 3 ){
             vertx.undeploy(deploymentID());
         }
     });
     System.out.println("Vertical PeriodicService1 is deployed");
  }
  public void stop() throws Exception {
     System.out.println("Vertical PeriodicService1 is un-deployed");
  }
}
```

如您所见，此垂直一旦部署，就会每秒打印一次“Beep！”消息，并且在三秒后会自动取消部署。如果我们部署此垂直，我们将看到：

```java
Vertical PeriodicService1 is deployed
Beep!
Beep!
Beep!
Beep!
Vertical PeriodicService1 is un-deployed
```

当垂直开始时，第一个“嘟嘟声！”响起，然后每秒钟会有三条消息，然后垂直被卸载，正如预期的那样。

# 作为微服务的 HTTP 客户端

我们可以使用周期性服务垂直向服务器垂直发送消息，使用 HTTP 协议。为了做到这一点，我们需要一个新的依赖项，所以我们可以使用`WebClient`类：

```java
<dependency>
    <groupId>io.vertx</groupId>
    <artifactId>vertx-web-client</artifactId>
    <version>${vertx.version}</version>
</dependency>

```

有了这个，向 HTTP 服务器垂直发送消息的周期性服务看起来是这样的：

```java
package com.packt.javapath.ch18demo.microservices;
import io.vertx.rxjava.core.AbstractVerticle;
import io.vertx.rxjava.core.buffer.Buffer;
import io.vertx.rxjava.ext.web.client.HttpResponse;
import io.vertx.rxjava.ext.web.client.WebClient;
import rx.Single;
import java.time.LocalTime;
import java.time.temporal.ChronoUnit;
public class PeriodicService2 extends AbstractVerticle {
    private int port;
    public PeriodicService2(int port) {
        this.port = port;
    }
    public void start() throws Exception {
        WebClient client = WebClient.create(vertx);
        Single<HttpResponse<Buffer>> single = client
                .get(port, "localhost", "?name=Nick")
                .rxSend();
        LocalTime start = LocalTime.now();
        vertx.setPeriodic(1000, v-> {
           single.subscribe(r-> System.out.println(r.bodyAsString()),
                             Throwable::printStackTrace);
           if(ChronoUnit.SECONDS.between(start, LocalTime.now()) >= 3 ){
              client.close(); 
              vertx.undeploy(deploymentID());
              System.out.println("Vertical PeriodicService2 undeployed");
           }
        });
        System.out.println("Vertical PeriodicService2 deployed");
    }
}
```

正如您所看到的，这个周期性服务接受端口号作为其构造函数的参数，然后每秒向本地主机的此端口发送一条消息，并在三秒后卸载自己。消息是`name`参数的值。默认情况下，它是 GET 请求。

我们还将修改我们的服务器垂直以读取`name`参数的值：

```java
public void start() throws Exception {
    HttpServer server = vertx.createHttpServer();
    server.requestStream().toObservable()
          .subscribe(request -> request.response()
             .end("Hi, " + request.getParam("name") + "! Hello from " + 
          Thread.currentThread().getName() + " on port " + port + "!"));
    server.rxListen(port).subscribe();
    System.out.println(Thread.currentThread().getName()
                               + " is waiting on port " + port + "...");
}
```

我们可以部署两个垂直：

```java
RxHelper.deployVerticle(vertx(), new HttpServer2(8082));
RxHelper.deployVerticle(vertx(), new PeriodicService2(8082));

```

输出将如下所示：

```java
Vertical PeriodicService2 deployed
vert.x-eventloop-thread-0 is waiting on port 8082...
Hi, Nick! Hello from vert.x-eventloop-thread-0 on port 8082!
Hi, Nick! Hello from vert.x-eventloop-thread-0 on port 8082!
Vertical PeriodicService2 undeployed
Hi, Nick! Hello from vert.x-eventloop-thread-0 on port 8082!
```

# 其他微服务

原则上，整个微服务系统可以基于使用 HTTP 协议发送的消息构建，每个微服务都实现为 HTTP 服务器或者将 HTTP 服务器作为消息交换的前端。或者，可以使用任何其他消息系统进行通信。

在 Vert.x 的情况下，它有自己基于事件总线的消息系统。在下一节中，我们将演示它，并将其用作反应式系统可能看起来的一个例子。

我们的示例微服务的大小可能会给人留下微服务必须像对象方法一样细粒度的印象。在某些情况下，值得考虑特定方法是否需要扩展。事实上，这种架构风格足够新颖，可以提供明确的大小建议，并且现有的框架、库和工具包足够灵活，可以支持几乎任何大小的独立部署服务。如果可部署的独立服务与传统应用程序一样大，那么它可能不会被称为微服务，而是*外部系统*或类似的东西。

# 反应式系统

熟悉**事件驱动架构**（**EDA**）概念的人可能已经注意到它与反应式系统的想法非常相似。它们的描述使用非常相似的语言和图表。不同之处在于 EDA 只涉及软件系统的一个方面——架构。另一方面，反应式系统更多地涉及代码风格和执行流程，包括强调使用异步数据流。因此，反应式系统可以具有 EDA，而 EDA 可以实现为反应式系统。

让我们看另一组示例，以了解使用 Vert.x 实现的反应式系统可能是什么样子。请注意，Vert.x API 有两个源树：一个以`io.vertx.core`开头，另一个以`io.vertx.rxjava`开头。由于我们正在讨论反应式编程，我们将使用`io.vertx.rxjava`下的包，称为 rx-fied Vert.x API。

# 消息驱动系统

Vert.x 具有直接支持消息驱动架构和 EDA 的功能。它被称为事件总线。任何 verticle 都可以访问事件总线，并且可以使用`io.vertx.core.eventbus.EventBus`类或其类似物`io.vertx.rxjava.core.eventbus.EventBus`向任何地址（只是一个字符串）发送任何消息。我们只会使用后者，但是`io.vertx.core.eventbus.EventBus`中也提供了类似（非 rx-fied）的功能。一个或多个 verticle 可以注册自己作为某个地址的消息消费者。如果有多个 verticle 是相同地址的消费者，那么`EventBus`的`rxSend()`方法使用循环算法仅将消息传递给这些消费者中的一个，以选择下一条消息的接收者。或者，`publish()`方法会将消息传递给具有相同地址的所有消费者。以下是将消息发送到指定地址的代码：

```java
vertx.eventBus().rxSend(address, msg).subscribe(reply -> 
    System.out.println("Got reply: " + reply.body()), 
    Throwable::printStackTrace );

```

`rxSend()`方法返回表示可以接收的消息的`Single<Message>`对象，并且`subscribe()`方法...嗯...订阅它。`Single<Message>`类实现了单个值响应的反应式模式。`subscribe()`方法接受两个`Consumer`函数：第一个处理回复，第二个处理错误。在前面的代码中，第一个函数只是打印回复：

```java
reply -> System.out.println("Got reply: " + reply.body())
```

第二个操作打印异常的堆栈跟踪，如果发生异常：

```java
Throwable::printStackTrace
```

如您所知，前面的结构称为方法引用。作为 lambda 表达式的相同函数将如下所示：

```java
e -> e.printStackTrace()
```

对`publish()`方法的调用看起来很相似：

```java
vertx.eventBus().publish(address, msg)
```

它将消息发布给许多消费者，因此该方法不会返回`Single`对象或任何其他可用于获取回复的对象。相反，它只返回一个`EventBus`对象；如果需要，可以调用更多的事件总线方法。

# 消息消费者

在 Vert.x 中的消息消费者是一个 verticle，它在事件总线上注册为指定地址发送或发布的消息的潜在接收者：

```java
package com.packt.javapath.ch18demo.reactivesystem;
import io.vertx.rxjava.core.AbstractVerticle;
public class MsgConsumer extends AbstractVerticle {
    private String address, name;
    public MsgConsumer(String id, String address) {
        this.address = address;
        this.name = this.getClass().getSimpleName() + 
                                    "(" + id + "," + address + ")";
    }
    public void start() throws Exception {
        System.out.println(name + " starts...");
        vertx.eventBus().consumer(address).toObservable()
         .subscribe(msg -> {
            String reply = name + " got message: " + msg.body();
            System.out.println(reply);
            if ("undeploy".equals(msg.body())) {
                vertx.undeploy(deploymentID());
                reply = name + " undeployed.";
                System.out.println(reply);
            }
            msg.reply(reply);
        }, Throwable::printStackTrace );
        System.out.println(Thread.currentThread().getName()
                + " is waiting on address " + address + "...");
    }
}
```

`consumer(address)`方法返回一个`io.vertx.rxjava.core.eventbus.MessageConsumer<T>`对象，表示提供的地址的消息流。这意味着可以将流转换为`Observable`并订阅它以接收发送到此地址的所有消息。`Observable`对象的`subscribe()`方法接受两个`Consumer`函数：第一个处理接收到的消息，第二个在发生错误时执行。在第一个函数中，我们包含了`msg.reply(reply)`方法，它将消息发送回消息的来源。您可能还记得，如果原始消息是通过`rxSend()`方法发送的，发送方可以获得此回复。如果使用了`publish()`方法，那么由`msg.reply(reply)`方法发送的回复将无处可去。

还要注意，当接收到`undeploy`消息时，消息消费者会取消部署自身。通常只在自动部署期间使用此方法，当旧版本被新版本替换而不关闭系统时。

因为我们将部署几个具有相同地址的消息消费者进行演示，所以我们添加了`id`参数并将其包含在`name`值中。此值用作所有消息中的前缀，因此我们可以跟踪消息在系统中的传播。

您可能已经意识到，前面的实现只是一个可以用来调用一些有用功能的外壳。接收到的消息可以是执行某些操作的命令，要处理的数据，要存储在数据库中的数据，或者其他任何内容。回复可以是收到消息的确认，或者其他预期的结果。如果是后者，处理应该非常快，以避免阻塞事件循环（记住黄金法则）。如果处理不能很快完成，回复也可以是一个回调令牌，稍后由发送方用来检索结果。

# 消息发送者

我们将演示的消息发送者基于我们在*微服务*部分演示的 HTTP 服务器实现。不一定非要这样做。在实际代码中，垂直通常会自动发送消息，要么获取它需要的数据，要么提供其他垂直需要的数据，要么通知另一个垂直，要么将数据存储在数据库中，或者出于任何其他原因。但是出于演示目的，我们决定发送方将侦听某个端口以接收消息，并且我们将手动（使用`curl`命令）或自动（通过*微服务*部分描述的某个周期性服务）发送消息给它。这就是为什么消息发送者看起来比消息消费者复杂一些：

```java
package com.packt.javapath.ch18demo.reactivesystem;
import io.vertx.rxjava.core.AbstractVerticle;
import io.vertx.rxjava.core.http.HttpServer;
public class EventBusSend extends AbstractVerticle {
    private int port;
    private String address, name;
    public EventBusSend(int port, String address) {
       this.port = port;
       this.address = address;
       this.name = this.getClass().getSimpleName() + 
                      "(port " + port + ", send to " + address + ")";
    }
    public void start() throws Exception {
       System.out.println(name + " starts...");
       HttpServer server = vertx.createHttpServer();
       server.requestStream().toObservable().subscribe(request -> {
         String msg = request.getParam("msg");
         request.response().setStatusCode(200).end();
 vertx.eventBus().rxSend(address, msg).subscribe(reply -> {
            System.out.println(name + " got reply:\n  " + reply.body());
         },
         e -> {
            if(StringUtils.contains(e.toString(), "NO_HANDLERS")){
                vertx.undeploy(deploymentID());
                System.out.println(name + " undeployed.");
            } else {
                e.printStackTrace();
            }
         }); });
       server.rxListen(port).subscribe();
       System.out.println(Thread.currentThread().getName()
                               + " is waiting on port " + port + "...");
    }
}
```

大部分前面的代码与 HTTP 服务器功能相关。发送消息（由 HTTP 服务器接收）的几行是这些：

```java
        vertx.eventBus().rxSend(address, msg).subscribe(reply -> {
            System.out.println(name + " got reply:\n  " + reply.body());
        }, e -> {
            if(StringUtils.contains(e.toString(), "NO_HANDLERS")){
                vertx.undeploy(deploymentID());
                System.out.println(name + " undeployed.");
            } else {
                e.printStackTrace();
            }
        });

```

发送消息后，发送者订阅可能的回复并打印它（如果收到了回复）。如果发生错误（在发送消息期间抛出异常），我们可以检查异常（转换为`String`值）是否包含文字`NO_HANDLERS`，如果是，则取消部署发送者。我们花了一段时间才弄清楚如何识别没有分配给此发送者发送消息的消费者的情况。如果没有消费者（很可能都取消部署了），那么发送者就没有必要了，所以我们取消部署它。

清理和取消部署所有不再需要的 verticle 是一个好习惯。但是，如果在 IDE 中运行 verticle，很有可能一旦停止创建 verticle 的主进程（已在 IDE 中创建 verticle），所有 verticle 都会停止。如果没有，请运行`jcmd`命令，并查看是否仍在运行 Vert.x verticle。列出的每个进程的第一个数字是进程 ID。识别不再需要的 verticle，并使用`kill -9 <process ID>`命令停止它们。

现在，让我们部署两个消息消费者，并通过我们的消息发送者向它们发送消息：

```java
String address = "One";
Vertx vertx = vertx();
RxHelper.deployVerticle(vertx, new MsgConsumer("1",address));
RxHelper.deployVerticle(vertx, new MsgConsumer("2",address));
RxHelper.deployVerticle(vertx, new EventBusSend(8082, address));

```

运行前面的代码后，终端显示以下消息：

```java
MsgConsumer(1,One) starts...
MsgConsumer(2,One) starts...
EventBusSend(port 8082, send to One) starts...
vert.x-eventloop-thread-1 is waiting on address One...
vert.x-eventloop-thread-0 is waiting on address One...
vert.x-eventloop-thread-2 is waiting on port 8082...
```

注意运行以支持每个 verticle 的不同事件循环。

现在，让我们使用终端窗口中的以下命令发送几条消息：

```java
curl localhost:8082?msg=Hello!
curl localhost:8082?msg=Hi!
curl localhost:8082?msg=How+are+you?
curl localhost:8082?msg=Just+saying...
```

加号（`+`）是必需的，因为 URL 不能包含空格，必须*编码*，这意味着，除其他外，用加号`+`或`%20`替换空格。作为对前述命令的响应，我们将看到以下消息：

```java
MsgConsumer(2,One) got message: Hello!
EventBusSend(port 8082, send to One) got reply:
 MsgConsumer(2,One) got message: Hello!
MsgConsumer(1,One) got message: Hi!
EventBusSend(port 8082, send to One) got reply:
 MsgConsumer(1,One) got message: Hi!
MsgConsumer(2,One) got message: How are you?
EventBusSend(port 8082, send to One) got reply:
 MsgConsumer(2,One) got message: How are you?
MsgConsumer(1,One) got message: Just saying...
EventBusSend(port 8082, send to One) got reply:
 MsgConsumer(1,One) got message: Just saying...
```

正如预期的那样，消费者根据循环算法轮流接收消息。现在，让我们部署所有的垂直线：

```java
curl localhost:8082?msg=undeploy
curl localhost:8082?msg=undeploy
curl localhost:8082?msg=undeploy
```

以下是对前述命令的响应中显示的消息：

```java
MsgConsumer(1,One) got message: undeploy
MsgConsumer(1,One) undeployed.
EventBusSend(port 8082, send to One) got reply:
 MsgConsumer(1,One) undeployed.
MsgConsumer(2,One) got message: undeploy
MsgConsumer(2,One) undeployed.
EventBusSend(port 8082, send to One) got reply:
 MsgConsumer(2,One) undeployed.
EventBusSend(port 8082, send to One) undeployed.
```

根据前面的消息，我们所有的垂直线都未部署。如果我们再次提交`undeploy`消息，我们将看到：

```java
curl localhost:8082?msg=undeploy
curl: (7) Failed to connect to localhost port 8082: Connection refused
```

这是因为发送者已被取消部署，并且本地主机的端口`8082`没有监听的 HTTP 服务器。

# 消息发布者

我们实现了消息发布者与消息发送者非常相似：

```java
package com.packt.javapath.ch18demo.reactivesystem;

import io.vertx.rxjava.core.AbstractVerticle;
import io.vertx.rxjava.core.http.HttpServer;

public class EventBusPublish extends AbstractVerticle {
    private int port;
    private String address, name;
    public EventBusPublish(int port, String address) {
        this.port = port;
        this.address = address;
        this.name = this.getClass().getSimpleName() + 
                    "(port " + port + ", publish to " + address + ")";
    }
    public void start() throws Exception {
        System.out.println(name + " starts...");
        HttpServer server = vertx.createHttpServer();
        server.requestStream().toObservable()
                .subscribe(request -> {
                    String msg = request.getParam("msg");
                    request.response().setStatusCode(200).end();
 vertx.eventBus().publish(address, msg);
                    if ("undeploy".equals(msg)) {
 vertx.undeploy(deploymentID());
                        System.out.println(name + " undeployed.");
                    }
                });
        server.rxListen(port).subscribe();
        System.out.println(Thread.currentThread().getName()
                + " is waiting on port " + port + "...");
    }
}
```

发布者与发送者的区别仅在于此部分：

```java
            vertx.eventBus().publish(address, msg);
            if ("undeploy".equals(msg)) {
                vertx.undeploy(deploymentID());
                System.out.println(name + " undeployed.");
            }
```

由于在发布时无法获得回复，因此前面的代码比发送消息的代码简单得多。此外，由于所有消费者同时收到`undeploy`消息，我们可以假设它们都将被取消部署，并且发布者可以取消部署自己。让我们通过运行以下程序来测试它：

```java
String address = "One";
Vertx vertx = vertx();
RxHelper.deployVerticle(vertx, new MsgConsumer("1",address));
RxHelper.deployVerticle(vertx, new MsgConsumer("2",address));
RxHelper.deployVerticle(vertx, new EventBusPublish(8082, address));

```

作为对前面的代码执行的响应，我们得到以下消息：

```java
MsgConsumer(1,One) starts...
MsgConsumer(2,One) starts...
EventBusPublish(port 8082, publish to One) starts...
vert.x-eventloop-thread-2 is waiting on port 8082...
```

现在，我们在另一个终端窗口中发出以下命令：

```java
curl localhost:8082?msg=Hello!
```

在运行垂直线的终端窗口中的消息如下：

```java
MsgConsumer(1,One) got message: Hello!
MsgConsumer(2,One) got message: Hello!
```

如预期的那样，具有相同地址的两个消费者都会收到相同的消息。现在，让我们将它们取消部署：

```java
curl localhost:8082?msg=undeploy
```

垂直线对这些消息做出响应：

```java
MsgConsumer(1,One) got message: undeploy
MsgConsumer(2,One) got message: undeploy
EventBusPublish(port 8082, publish to One) undeployed.
MsgConsumer(1,One) undeployed.
MsgConsumer(2,One) undeployed.
```

如果我们再次提交`undeploy`消息，我们将看到：

```java
curl localhost:8082?msg=undeploy
curl: (7) Failed to connect to localhost port 8082: Connection refused
```

通过这样，我们已经完成了一个由微服务组成的反应式系统的演示。添加能够执行有用操作的方法和类将使其更接近实际系统。但我们将把这留给读者作为练习。

# 现实检查

我们在一个 JVM 进程中运行了所有之前的示例。如果需要，Vert.x 实例可以部署在不同的 JVM 进程中，并通过在`run`命令中添加`-cluster`选项来进行集群化，当垂直部署不是从 IDE，而是从命令行时。集群化的垂直共享事件总线，地址对所有 Vert.x 实例可见。这样，如果某些地址的消费者无法及时处理请求（消息），就可以部署更多的消息消费者。

我们之前提到的其他框架具有类似的功能。它们使微服务的创建变得容易，并可能鼓励将应用程序分解为微小的、单方法的操作，期望组装一个非常具有弹性和响应性的系统。然而，这些并不是良好软件的唯一标准。系统分解增加了部署的复杂性。此外，如果一个开发团队负责许多微服务，那么在不同阶段（开发、测试、集成测试、认证、暂存和生产）对这么多部分进行版本控制的复杂性可能会导致混乱。部署过程可能变得如此复杂，以至于减缓变更速度是必要的，以使系统与市场需求保持同步。

除了开发微服务之外，还必须解决许多其他方面，以支持反应式系统：

+   必须建立监控系统以提供对应用程序状态的洞察，但其开发不应该如此复杂，以至于将开发资源从主要应用程序中分散开来。

+   必须安装警报以及及时警告团队可能和实际问题，以便在影响业务之前解决这些问题。

+   如果可能的话，必须实施自我纠正的自动化流程。例如，必须实施重试逻辑，并在宣布失败之前设定合理的尝试上限。

+   一层断路器必须保护系统免受多米诺效应的影响，当一个组件的故障剥夺了其他组件所需的资源时。

+   嵌入式测试系统应该能够引入干扰并模拟负载增加，以确保应用程序的弹性和响应性不会随着时间的推移而降低。例如，Netflix 团队引入了*混沌猴*——一个能够关闭生产系统的各个部分并测试其恢复能力的系统。他们甚至在生产中使用它，因为生产环境具有特定的配置，而在另一个环境中的测试无法保证找到所有可能的问题。

正如你现在可能已经意识到的那样，在承诺采用反应式系统之前，团队必须权衡所有的利弊，以确切地理解他们为什么需要反应式系统，以及其开发的代价。有一句古老的格言说：“没有免费的午餐”。反应式系统的强大力量伴随着复杂性的相应增长，不仅在开发过程中，而且在系统调优和维护过程中也是如此。

然而，如果传统系统无法解决您面临的处理问题，或者如果您对一切反应式并且热爱这个概念，那么请尽管去做。旅程将充满挑战，但回报将是值得的。正如另一句古老的格言所说，“轻而易举地实现不值得努力”。

# 练习-创建 io.reactivex.Observable

编写代码演示创建`io.reactivex.Observable`的几种方法。在每个示例中，订阅创建的`Observable`对象并打印发出的值。

我们没有讨论这一点，因此您需要学习 RxJava2 API 并在互联网上查找示例。

# 答案

以下是允许您创建`io.reactivex.Observable`的六种方法：

```java
//1
Observable.just("Hi!").subscribe(System.out::println); //prints: Hi!
//2
Observable.fromIterable(List.of("1","2","3"))
          .subscribe(System.out::print); //prints: 123
System.out.println();
//3
String[] arr = {"1","2","3"};
Observable.fromArray(arr).subscribe(System.out::print); //prints: 123
System.out.println();
//4
Observable.fromCallable(()->123)
          .subscribe(System.out::println); //prints: 123
//5
ExecutorService pool = Executors.newSingleThreadExecutor();
Future<String> future = pool
        .submit(() -> {
            Thread.sleep(100);
            return "Hi!";
        });
Observable.fromFuture(future)
          .subscribe(System.out::println); //prints: Hi!
pool.shutdown();
//6
Observable.interval(100, TimeUnit.MILLISECONDS)
          .subscribe(v->System.out.println("100 ms is over")); 
                                     //prints twice "100 ms is over"
try { //this pause gives the above method a chance to print the message
    TimeUnit.MILLISECONDS.sleep(200);
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

# 摘要

在本书的最后一章中，我们向读者提供了一个真实专业编程的一瞥，并简要概述了这一行业的挑战。我们重新审视了许多现代术语，这些术语与使用高度可扩展、响应迅速和具有弹性的反应式系统相关，这些系统能够解决现代时代的具有挑战性的处理问题。我们甚至提供了这些系统的代码示例，这可能是您真实项目的第一步。

我们希望您保持好奇心，继续学习和实验，并最终建立一个能够解决真实问题并为世界带来更多幸福的系统。
