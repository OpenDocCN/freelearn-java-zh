# 第十一章：内存管理和调试

在本章中，我们将涵盖以下内容：

+   了解 G1 垃圾收集器

+   JVM 的统一日志记录

+   使用`jcmd`命令进行 JVM

+   使用 try-with-resources 来更好地处理资源

+   为了改进调试，堆栈遍历

+   使用内存感知的编码风格

+   更好地使用内存的最佳实践

+   了解 Epsilon，一种低开销的垃圾收集器

# 介绍

内存管理是程序执行的内存分配过程，以及一些分配的内存不再使用后的内存重用。在 Java 中，这个过程被称为**垃圾收集**（**GC**）。GC 的有效性影响两个主要应用特性——响应性和吞吐量。

响应性是指应用程序对请求的快速响应程度。例如，一个网站返回页面的速度，或者桌面应用程序对事件的快速响应。自然地，响应时间越短，用户体验就越好，这是许多应用程序的目标。

吞吐量表示应用程序在单位时间内可以完成的工作量。例如，一个 Web 应用程序可以提供多少请求，或者一个数据库可以支持多少交易。数字越大，应用程序可能产生的价值就越大，可以容纳的用户数量也越多。

并非每个应用程序都需要具有最小的响应性和最大的可实现吞吐量。一个应用程序可能是异步提交并执行其他操作，不需要太多用户交互。可能也只有少数潜在的应用程序用户，因此低于平均水平的吞吐量可能已经足够了。然而，有些应用程序对这些特性中的一个或两个有很高的要求，并且无法容忍 GC 过程带来的长时间暂停。

另一方面，GC 需要偶尔停止任何应用程序执行，重新评估内存使用情况，并释放不再使用的数据。这些 GC 活动期间被称为停止-世界。它们持续的时间越长，GC 完成工作的速度越快，应用程序冻结的时间就越长，最终可能会足够大到影响应用程序的响应性和吞吐量。如果情况如此，GC 调优和 JVM 优化变得重要，并需要理解 GC 原则及其现代实现。

不幸的是，程序员经常忽略了这一步。试图改进响应性和/或吞吐量，他们只是增加了内存和其他计算能力，从而为最初较小的现有问题提供了增长的空间。扩大的基础设施，除了硬件和软件成本外，还需要更多的人来维护，最终证明需要建立一个专门的组织来维护系统。到那时，问题已经达到了几乎无法解决的规模，并且通过迫使他们为其余的职业生涯做例行的——几乎是琐碎的——工作，滋养了那些创造它的人。

在本章中，我们将重点关注**Garbage-First**（**G1**）垃圾收集器，这是自 Java 9 以来的默认收集器。然而，我们也会提到其他几种可用的 GC 实现，以对比和解释一些设计决策，这些决策使 G1 得以诞生。此外，它们可能比 G1 更适合某些应用程序。

内存组织和管理是 JVM 开发中非常专业和复杂的领域。本书不打算在这个层面上解决实现细节。我们的重点是 GC 的那些方面，可以通过设置 JVM 运行时的相应参数，帮助应用程序开发人员调整应用程序的需求。

GC 使用的两个内存区域是堆和栈。第一个由 JVM 用于分配内存和存储程序创建的对象。当使用`new`关键字创建对象时，它位于堆中，并且对它的引用存储在栈中。栈还存储原始变量和当前方法或线程使用的堆对象的引用。栈以**后进先出**（**LIFO**）的方式操作。栈比堆小得多。

对于任何 GC 的略微简化但足够好的高层次视图是—遍历堆中的对象并删除那些在堆栈中没有引用的对象。

# 理解 G1 垃圾收集器

以前的 GC 实现包括**串行 GC**、**并行 GC**和**并发标记-清除**（**CMS**）收集器。它们将堆分成三个部分—年轻代、老年代或终身代和用于容纳大小为标准区域的 50%或更大的对象的巨大区域。年轻代包含大部分新创建的对象；这是最动态的区域，因为大多数对象的寿命很短，很快（随着它们的年龄）就有资格进行收集。术语年龄指的是对象存活的收集周期数。年轻代有三个收集周期—伊甸空间和两个幸存者空间，如幸存者 0（*S0*）和幸存者 1（*S1*）。对象会根据它们的年龄和其他一些特征移动到这些空间中，直到它们最终被丢弃或放入老年代。

老年代包含比一定年龄更老的对象。这个区域比年轻代大，因此这里的垃圾收集更昂贵，发生的频率也不如年轻代频繁。

永久代包含描述应用程序中使用的类和方法的元数据。它还存储字符串、库类和方法。

JVM 启动时，堆是空的，然后对象被推送到伊甸园。当它填满时，一个小的 GC 过程开始。它移除了未引用和循环引用的对象，并将其他对象移动到*S0*区域。

接下来的小 GC 过程将引用的对象迁移到*S1*，并增加了在上一次小集合中幸存的对象的年龄。在所有幸存的对象（不同年龄的对象）都移动到*S1*后，*S0*和伊甸园都变为空。

在下一次小集合中，*S0*和*S1*交换它们的角色。引用的对象从伊甸园移动到*S1*，从*S1*移动到*S0*。

在每次小集合中，已经达到一定年龄的对象被移动到老年代。正如我们之前提到的，老年代最终会被检查（经过几次小集合后），未引用的对象将从中移除，并且内存将被碎片整理。这种对老年代的清理被认为是一次大集合。

永久代由不同的 GC 算法在不同的时间进行清理。

G1 GC 做法略有不同。它将堆分成相等大小的区域，并为每个区域分配相同的角色—伊甸园、幸存者或老年代—但根据需要动态地改变具有相同角色的区域数量。这使得内存清理过程和内存碎片整理更加可预测。

# 准备就绪

串行 GC 在同一个周期内清理年轻代和老年代（串行地，因此得名）。在执行任务期间，它会停止世界。这就是为什么它适用于只有一个 CPU 和堆大小为几百 MB 的非服务器应用程序。

并行 GC 在所有可用核心上并行工作，尽管线程数量可以进行配置。它也会停止世界，只适用于可以容忍长时间冻结的应用程序。

CMS 收集器旨在解决长时间暂停的问题。它以不对旧一代进行碎片整理和在应用程序执行期间进行一些分析（通常使用 25%的 CPU）为代价。旧一代的收集在其占用空间达到 68%时开始（默认情况下，但此值可以配置）。

G1 GC 算法类似于 CMS 收集器。首先，它并发地识别堆中的所有引用对象并相应地标记它们。然后，它首先收集最空的区域，从而释放大量的空间。这就是为什么它被称为*垃圾优先*。因为它使用许多小的专用区域，它有更好的机会来预测清理一个区域所需的时间，并适应用户定义的暂停时间（G1 偶尔可能超出，但大多数情况下非常接近）。

G1 的主要受益者是需要大堆（6GB 或更多）且不能容忍长时间暂停（0.5 秒或更短）的应用程序。如果应用程序遇到太多或太长时间的暂停问题，可以从 CMS 或并行 GC（特别是旧一代的并行 GC）切换到 G1 GC 获益。如果不是这种情况，在使用 JDK 9 或更高版本时，切换到 G1 收集器不是必需的。

G1 GC 从年轻代开始收集，使用停顿世界暂停进行疏散（将年轻代内部和旧一代之间的对象移动）。当旧一代的占用达到一定阈值后，也会进行收集。旧一代中的一些对象是并发收集的，而一些对象是使用停顿世界暂停进行收集的。步骤包括以下内容：

+   幸存者区域（根区域）的初始标记，可能引用旧一代的对象，使用停顿世界暂停来完成

+   扫描幸存者区域以查找对旧一代的引用，与此同时应用程序继续运行

+   在整个堆上并发标记活动对象，同时应用程序继续运行

+   备注步骤完成了活动对象的标记，使用停顿世界暂停来完成

+   清理过程计算活动对象的年龄，释放区域（使用停顿世界暂停），并将它们返回到空闲列表（并发进行）

前面的序列可能会与年轻代疏散交错，因为大多数对象的生命周期很短，通过更频繁地扫描年轻代来释放大量内存更容易。

还有一个混合阶段，当 G1 收集已标记为大部分垃圾的年轻代和旧一代的区域时，以及巨大分配，当大对象被移动到或从巨大区域疏散时。

有一些情况下会执行完全 GC，使用停顿世界暂停：

+   **并发失败**：如果在标记阶段旧一代占满空间

+   **提升失败**：如果在混合阶段旧一代空间不足时发生

+   **疏散失败**：当收集器无法将对象提升到幸存者空间和旧一代时发生

+   **巨大分配**：当应用程序尝试分配一个非常大的对象时发生

如果正确调整，您的应用程序应该避免完全 GC。

为了帮助 GC 调优，JVM 文档（[`docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/ergonomics.html`](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/ergonomics.html)）描述了人体工程学如下：

“*自适应性是 JVM 和垃圾收集调整的过程，例如基于行为的调整，它提高了应用程序的性能。JVM 为垃圾收集器、堆大小和运行时编译器提供了平台相关的默认选择。这些选择符合不同类型应用程序的需求，同时需要较少的命令行调整。此外，基于行为的调整动态调整堆的大小，以满足应用程序的指定行为。”

# 如何做...

1.  要了解 GC 的工作原理，请编写以下程序：

```java
        public class Chapter11Memory {   
           public static void main(String... args) {
              int max = 99_888_999;
              System.out.println("Chapter11Memory.main() for " 
                                      + max + " is running...");
              List<AnObject> list = new ArrayList<>();
              IntStream.range(0, max)
                       .forEach(i -> list.add(new AnObject(i)));
           }

           private static class AnObject {
              private int prop;
              AnObject(int i){ this.prop = i; }
           }
        }
```

如您所见，它创建了 99,888,999 个对象，并将它们添加到`List<AnObject> list`集合中。您可以通过减少对象的最大数量（`max`）来调整它，以匹配您计算机的配置。

1.  自 Java 9 以来，G1 GC 是默认收集器，因此如果对您的应用程序足够好，您无需设置任何内容。尽管如此，您可以通过在命令行上提供`-XX:+UseG1GC`来显式启用 G1：

```java
 java -XX:+UseG1GC -cp ./cookbook-1.0.jar 
      com.packt.cookbook.ch11_memory.Chapter11Memory
```

请注意，我们假设您可以构建一个可执行的`.jar`文件并理解基本的 Java 执行命令。如果不行，请参考 JVM 文档。

其他可用的 GC 可以通过设置以下选项之一来使用：

+   +   `-XX:+UseSerialGC`用于使用串行收集器。

+   `-XX:+UseParallelGC`用于使用带有并行压缩的并行收集器（这使得并行收集器可以并行执行主要收集）。没有并行压缩，主要收集将使用单个线程执行，这可能会严重限制可伸缩性。通过`-XX:+UseParallelOldGC`选项禁用并行压缩。

+   -XX:+UseConcMarkSweepGC 用于使用 CMS 收集器。

1.  要查看 GC 的日志消息，请设置`-Xlog:gc`。您还可以使用 Unix 实用程序`time`来测量完成作业所需的时间（该实用程序会发布输出的最后三行，因此如果您无法或不想使用它，则无需使用它）：

```java
 time java -Xlog:gc -cp ./cookbook-1.0.jar com.packt.cookbook.ch11_memory.Chapter11Memory
```

1.  运行上述命令。输出可能如下所示（实际值可能在您的计算机上有所不同）：

![](img/559f97f3-d6c4-40f6-8656-ba6b3a7f2834.png)

如您所见，GC 经历了我们描述的大部分步骤。它从收集年轻代开始。然后，当`List<AnObject> list`对象（请参阅前面的代码）变得太大（超过年轻代区域的 50%以上）时，为其分配内存到*巨大*区域。您还可以看到初始标记步骤、随后的重新标记和其他先前描述的步骤。

每行以 JVM 运行的时间（以秒为单位）开头，并以每个步骤花费的时间（以毫秒为单位）结尾。在屏幕截图的底部，我们看到了`time`实用程序打印的三行：

+   +   `real`是花费的挂钟时间量——自命令运行以来经过的所有时间（应与 JVM 正常运行时间值的第一列对齐）

+   `user`是进程中所有 CPU 在用户模式代码（内核外）中花费的时间量；它更大是因为 GC 与应用程序并发工作。

+   `sys` 是 CPU 在进程内核中花费的时间量

+   `user`+`sys`是进程使用的 CPU 时间量

1.  设置`-XX:+PrintGCDetails`选项（或只需在日志选项`-Xlog:gc*`中添加`*`）以查看有关 GC 活动的更多详细信息。在以下屏幕截图中，我们仅提供了与 GC 步骤 0 相关的日志的开头：

![](img/99a97b8a-3f4b-44ef-bff8-1001d7346e0c.png)

现在日志中有超过十几个条目，每个 GC 步骤都以记录`User`、`Sys`和`Real`时间量（由`time`实用程序累积的时间量）结束。您可以通过添加更多的短寿命对象来修改程序，例如，看看 GC 活动如何改变。

1.  使用`-Xlog:gc*=debug`选项获取更多信息。以下仅为输出的一部分：

![](img/2b425d3d-0e6e-4cb2-9900-2a001530b65e.png)

因此，您可以选择需要多少信息进行分析。

我们将在《JVM 统一日志记录》中讨论日志格式和其他日志选项的更多细节。

# 工作原理...

正如我们之前提到的，G1 GC 使用默认的人体工程学值，这些值对于大多数应用程序来说可能已经足够好了。以下是最重要的一些值的列表（`<ergo>`表示实际值是根据环境人体工程学确定的）：

+   -XX:MaxGCPauseMillis=200：保持最大暂停时间的值

+   -XX:GCPauseTimeInterval=<ergo>：保持 GC 步骤之间的最大暂停时间（默认情况下未设置，允许 G1 在需要时连续执行垃圾收集）

+   -XX:ParallelGCThreads=<ergo>：保持在垃圾收集暂停期间用于并行工作的最大线程数（默认情况下，从可用线程数派生；如果可用于进程的 CPU 线程数小于或等于八，它使用这个数字；否则，它将大于八的五分之八的线程添加到最终线程数中）

+   -XX:ConcGCThreads=<ergo>：保持用于并发工作的最大线程数（默认设置为`-XX:ParallelGCThreads`除以四）。

+   -XX:+G1UseAdaptiveIHOP：表示启动堆占用应该是自适应的

+   -XX:InitiatingHeapOccupancyPercent=45：设置了最初的几个收集周期；G1 将使用老年代 45%的占用作为标记开始阈值

+   -XX:G1HeapRegionSize=<ergo>：根据初始和最大堆大小保持堆区域大小（默认情况下，因为堆包含大约 2048 个堆区域，堆区域的大小可以从 1 到 32 MB 不等，并且必须是 2 的幂）

+   -XX:G1NewSizePercent=5 和-XX:XX:G1MaxNewSizePercent=60：定义了年轻代的总大小，它们作为当前 JVM 堆使用百分比在这两个值之间变化

+   -XX:G1HeapWastePercent=5：保持收集集候选对象中允许的未回收空间的百分比（如果收集集候选对象中的空闲空间低于此值，G1 将停止空间回收）

+   -XX:G1MixedGCCountTarget=8：保持空间回收阶段的预期长度（以收集次数计算）

+   -XX:G1MixedGCLiveThresholdPercent=85：保持老年代区域中存活对象占用的百分比，超过这个百分比的区域将不会在空间回收阶段被收集

一般来说，默认配置下 G1 的目标是“在高吞吐量下提供相对较小、均匀的暂停”（来自 G1 文档）。如果这些默认设置不适合您的应用程序，您可以改变暂停时间（使用`-XX:MaxGCPauseMillis`）和最大 Java 堆大小（使用`-Xmx`选项）。但请注意，实际的暂停时间在运行时不会完全匹配，但 G1 会尽力满足目标。

如果您想增加吞吐量，可以减少暂停时间目标或请求更大的堆。要增加响应性，改变暂停时间值。但请注意，限制年轻代大小（使用`-Xmn`，`-XX:NewRatio`或其他选项）可能会妨碍暂停时间控制，因为“年轻代大小是 G1 允许其满足暂停时间的主要手段”（来自 G1 文档）。

性能不佳的一个可能原因是由于老年代堆占用过高而触发了 Full GC。这种情况可以通过日志中出现*Pause Full (Allocation failure)*来检测到。通常发生在对象快速创建过多（无法及时回收）或者许多大型（巨大）对象无法及时分配的情况下。有几种推荐的处理这种情况的方法：

+   在出现过多的巨大对象的情况下，尝试通过增加区域大小，使用`-XX:G1HeapRegionSize`选项来减少它们的数量（当前选择的堆区域大小在日志开头打印出来）。

+   增加堆的大小。

+   通过设置`-XX:ConcGCThreads`增加并发标记线程的数量。

+   通过修改`-XX:G1ReservePercent`增加自适应 IHOP 计算中使用的缓冲区，或者通过`-XX:-G1UseAdaptiveIHOP`和`-XX:InitiatingHeapOccupancyPercent`手动设置禁用 IHOP 的自适应计算，促进更早的标记开始（利用 G1 基于更早应用行为做出决策的事实）。 

只有在解决了完整的 GC 后，才能开始调整 JVM 以获得更好的响应和/或吞吐量。JVM 文档确定了以下情况需要调整响应性：

+   异常系统或实时使用

+   引用处理需要太长时间

+   仅年轻代收集需要太长时间

+   混合集合需要太长时间

+   高更新 RS 和扫描 RS 时间

通过减少总暂停时间和暂停频率来实现更好的吞吐量。请参考 JVM 文档以识别和建议减轻问题。

# JVM 的统一日志记录

JVM 的主要组件包括以下内容：

+   类加载器

+   JVM 内存，运行时数据存储在其中；它分为以下几个区域：

+   堆栈区域

+   方法区域

+   堆区域

+   PC 寄存器

+   本地方法栈

+   执行引擎，包括以下部分：

+   解释器

+   JIT 编译器

+   垃圾收集

+   本地方法接口 JNI

+   本地方法库

现在可以使用统一日志记录所有这些组件的日志消息，并通过`-Xlog`选项打开。

新日志系统的主要特点如下：

+   日志级别的使用——`trace`、`debug`、`info`、`warning`、`error`

+   标识 JVM 组件、操作或特定感兴趣消息的消息标签

+   三种输出类型——`stdout`、`stderr`和`file`

+   强制每行限制一个消息

# 准备工作

要一目了然地查看所有日志可能性，可以运行以下命令：

```java
java -Xlog:help
```

以下是输出：

![](img/788ed104-16ed-496b-90d0-b8d535eca63d.png)

如您所见，`-Xlog`选项的格式定义如下：

```java
-Xlog[:[what][:[output][:[decorators][:output-options]]]]
```

让我们详细解释一下这个选项：

+   `what`是`tag1[+tag2...][*][=level][,...]`形式的标签和级别的组合。我们已经演示了当我们在`-Xlog:gc*=debug`选项中使用`gc`标签时，这个结构是如何工作的。通配符（`*`）表示您想要查看所有具有`gc`标签的消息（可能是其他标签中的一部分）。`-Xlog:gc=debug`中缺少通配符表示您只想看到由一个标签（在本例中为`gc`）标记的消息。如果只使用`-Xlog`，日志将以`info`级别显示所有消息。

+   `output`设置输出类型（默认为`stdout`）。

+   `decorators`指示日志每行的开头将放置什么（在实际日志消息来自组件之前）。默认的`decorators`是`uptime`、`level`和`tags`，每个都包含在方括号中。

+   `output_options`可能包括`filecount=file count`和/或`filesize=file size`，可选的 K、M 或 G 后缀。

总之，默认的日志配置如下：

```java
-Xlog:all=info:stdout:uptime,level,tags
```

# 如何做...

让我们运行一些日志设置：

1.  运行以下命令：

```java
 java -Xlog:cpu -cp ./cookbook-1.0.jar 
                  com.packt.cookbook.ch11_memory.Chapter11Memory
```

没有消息是因为 JVM 不仅使用`cpu`标签记录消息。该标签与其他标签结合使用。

1.  添加`*`号并再次运行命令：

```java
 java -Xlog:cpu* -cp ./cookbook-1.0.jar  
                 com.packt.cookbook.ch11_memory.Chapter11Memory
```

结果如下：

![](img/4566a7ef-010f-4d35-ab73-cdba7dd3f4a9.png)

如您所见，`cpu`标签只会显示垃圾收集执行所需的时间。即使我们将日志级别设置为`trace`或`debug`（例如`-Xlog:cpu*=debug`），也不会显示其他消息。

1.  使用`heap`标签运行命令：

```java
 java -Xlog:heap* -cp ./cookbook-1.0.jar 
                 com.packt.cookbook.ch11_memory.Chapter11Memory
```

您将只收到与堆相关的消息：

![](img/c81bd5b6-448f-48d3-b5d4-af99a70f1f13.png)

但让我们仔细看看第一行。它以三个装饰符开头——`uptime`、`log level`和`tags`——然后是消息本身，它以收集周期编号（在本例中为 0）开头，以及 Eden 区域的数量从 24 下降到 0（现在的数量为 9）的信息。这是因为（正如我们在下一行中看到的那样）幸存者区域的数量从 0 增加到 3，老年代的数量（第三行）增加到 18，而巨大区域的数量（23）没有改变。这些都是第一个收集周期中与堆相关的消息。然后，第二个收集周期开始。

1.  再次添加`cpu`标签并运行：

```java
 java -Xlog:heap*,cpu* -cp ./cookbook-1.0.jar 
                   com.packt.cookbook.ch11_memory.Chapter11Memory
```

如您所见，`cpu`消息显示了每个周期的持续时间：

![](img/f3cfeb83-7d29-427b-a784-3905350fbd00.png)

1.  尝试使用通过`+`符号组合的两个标签（例如`-Xlog:gc+heap`）。它只会显示具有这两个标签的消息（类似于二进制的`AND`操作）。请注意，通配符将无法与`+`符号一起使用（例如，`-Xlog:gc*+heap`不起作用）。

1.  您还可以选择输出类型和装饰符。实际上，装饰符级别似乎并不是非常信息丰富，可以通过明确列出仅需要的装饰符来轻松省略。考虑以下示例：

```java
 java -Xlog:heap*,cpu*::uptime,tags -cp ./cookbook-1.0.jar 
                    com.packt.cookbook.ch11_memory.Chapter11Memory
```

注意如何插入两个冒号（`::`）以保留输出类型的默认设置。我们也可以明确显示它：

```java
 java -Xlog:heap*,cpu*:stdout:uptime,tags -cp ./cookbook-1.0.jar
                    com.packt.cookbook.ch11_memory.Chapter11Memory
```

要删除任何装饰，可以将它们设置为`none`：

```java
 java -Xlog:heap*,cpu*::none -cp ./cookbook-1.0.jar
                     com.packt.cookbook.ch11_memory.Chapter11Memory
```

新日志系统最有用的方面是标签选择。它允许更好地分析每个 JVM 组件及其子系统的内存演变，或者找到性能瓶颈，分析在每个收集阶段花费的时间——这两者对于 JVM 和应用程序调优都至关重要。

# 使用 JVM 的 jcmd 命令

如果打开 Java 安装的`bin`文件夹，您可以在那里找到相当多的命令行实用程序，可用于诊断问题并监视使用**Java Runtime Environment**（**JRE**）部署的应用程序。它们使用不同的机制来获取它们报告的数据。这些机制特定于**虚拟机**（**VM**）实现、操作系统和版本。通常，这些工具的子集仅适用于特定问题。

在本示例中，我们将重点放在 Java 9 中引入的诊断命令，即命令行实用程序`jcmd`。如果`bin`文件夹在路径上，您可以通过在命令行上键入`jcmd`来调用它。否则，您必须转到`bin`目录，或者在我们的示例中在`jcmd`之前加上`bin`文件夹的完整路径或相对路径（相对于您的命令行窗口的位置）。

如果您输入它，而机器上当前没有运行 Java 进程，您将只收到一行，如下所示：

```java
87863 jdk.jcmd/sun.tools.jcmd.JCmd 
```

它显示当前只有一个 Java 进程正在运行（`jcmd`实用程序本身），并且它具有**进程标识符**（**PID**）为 87863（每次运行时都会有所不同）。

让我们运行一个 Java 程序，例如：

```java
java -cp ./cookbook-1.0.jar 
                   com.packt.cookbook.ch11_memory.Chapter11Memory
```

`jcmd`的输出将显示（具有不同 PID）以下内容：

```java
87864 jdk.jcmd/sun.tools.jcmd.JCmd 
87785 com.packt.cookbook.ch11_memory.Chapter11Memory
```

如您所见，如果没有任何选项输入，`jcmd`实用程序将报告所有当前运行的 Java 进程的 PID。获取 PID 后，您可以使用`jcmd`从运行该进程的 JVM 请求数据：

```java
jcmd 88749 VM.version 
```

或者，您可以避免使用 PID（并且不带参数调用`jcmd`）通过引用应用程序的主类来引用该进程：

```java
jcmd Chapter11Memory VM.version
```

您可以阅读 JVM 文档，以获取有关`jcmd`实用程序及其用法的更多详细信息。

# 如何做...

`jcmd`是一个允许我们向指定的 Java 进程发出命令的实用程序：

1.  通过执行以下行，可以获取特定 Java 进程可用的`jcmd`命令的完整列表：

```java
 jcmd PID/main-class-name help
```

在`PID/main-class`的位置，放置进程标识符或主类名称。该列表特定于 JVM，因此每个列出的命令都会从特定进程请求数据。

1.  在 JDK 8 中，以下`jcmd`命令是可用的：

```java
JFR.stop
JFR.start
JFR.dump
JFR.check
VM.native_memory
VM.check_commercial_features
VM.unlock_commercial_features
ManagementAgent.stop
ManagementAgent.start_local
ManagementAgent.start
GC.rotate_log
Thread.print
GC.class_stats
GC.class_histogram
GC.heap_dump
GC.run_finalization
GC.run
VM.uptime
VM.flags
VM.system_properties
VM.command_line
VM.version
```

JDK 9 引入了以下`jcmd`命令（JDK 18.3 和 JDK 18.9 没有添加新命令）：

+   +   `Compiler.queue`: 打印排队等待使用 C1 或 C2 编译的方法（分别排队）

+   `Compiler.codelist`: 打印 n 个（已编译的）方法的完整签名、地址范围和状态（活动、非进入和僵尸），并允许选择打印到`stdout`、文件、XML 或文本输出

+   `Compiler.codecache`: 打印代码缓存的内容，即 JIT 编译器存储生成的本机代码以提高性能的地方

+   `Compiler.directives_add file`: 从文件向指令栈顶部添加编译器指令

+   `Compiler.directives_clear`: 清除编译器指令栈（仅保留默认指令）

+   `Compiler.directives_print`: 从顶部到底部打印编译器指令栈上的所有指令

+   `Compiler.directives_remove`: 从编译器指令栈中移除顶部指令

+   `GC.heap_info`: 打印当前堆参数和状态

+   `GC.finalizer_info`: 显示终结器线程的状态，该线程收集具有终结器（即`finalize()`方法）的对象

+   `JFR.configure`: 允许我们配置 Java Flight Recorder

+   `JVMTI.data_dump`: 打印 Java 虚拟机工具接口数据转储

+   `JVMTI.agent_load`: 加载（附加）Java 虚拟机工具接口代理

+   `ManagementAgent.status`: 打印远程 JMX 代理的状态

+   `Thread.print`: 打印所有带有堆栈跟踪的线程

+   `VM.log [option]`: 允许我们在 JVM 启动后（可以通过使用`VM.log list`查看可用性）在运行时设置 JVM 日志配置（我们在前面的配方中描述了）

+   `VM.info`: 打印统一的 JVM 信息（版本和配置）、所有线程及其状态的列表（不包括线程转储和堆转储）、堆摘要、JVM 内部事件（GC、JIT、安全点等）、加载的本机库的内存映射、VM 参数和环境变量，以及操作系统和硬件的详细信息

+   `VM.dynlibs`: 打印动态库的信息

+   `VM.set_flag`: 允许我们设置 JVM 的*可写*（也称为*可管理*）标志（请参阅 JVM 文档以获取标志列表）

+   `VM.stringtable`和`VM.symboltable`: 打印所有 UTF-8 字符串常量

+   `VM.class_hierarchy [full-class-name]`: 打印所有已加载的类或指定类层次结构

+   `VM.classloader_stats`: 打印有关类加载器的信息

+   `VM.print_touched_methods`: 打印在运行时已被访问（至少已被读取）的所有方法

正如您所看到的，这些新命令属于几个组，由前缀编译器、**垃圾收集器**（**GC**）、**Java Flight Recorder**（**JFR**）、**Java 虚拟机工具接口**（**JVMTI**）、**管理代理**（与远程 JMX 代理相关）、**线程**和**VM**表示。在本书中，我们没有足够的空间来详细介绍每个命令。我们只会演示一些实用命令的用法。

# 工作原理...

1.  要获取`jcmd`实用程序的帮助，请运行以下命令：

```java
jcmd -h 
```

以下是命令的结果：

![](img/9c0c5f23-7cdb-4af3-b2ce-942c651d9803.png)

它告诉我们，命令也可以从`-f`之后指定的文件中读取，并且有一个`PerfCounter.print`命令，它打印进程的所有性能计数器（统计信息）。

1.  运行以下命令：

```java
jcmd Chapter11Memory GC.heap_info
```

输出可能看起来像这个屏幕截图：

![](img/abfd96d8-ab1b-4ff2-8991-f21ef5f8d76a.png)

它显示了总堆大小及其使用量，年轻代中区域的大小和分配的区域数量，以及`Metaspace`和`class space`的参数。

1.  以下命令在您寻找失控线程或想了解幕后发生了什么时非常有帮助：

```java
jcmd Chapter11Memory Thread.print
```

以下是可能输出的片段：

![](img/a43091e9-2642-4bde-9792-76cedc1a469e.png)

1.  这个命令可能是最常用的，因为它提供了关于硬件、整个 JVM 进程以及其组件当前状态的丰富信息：

```java
jcmd Chapter11Memory VM.info
```

它以摘要开始，如下所示：

![](img/74c3872d-716d-4a5e-8479-35a470b2c17f.png)

接下来是一般的过程描述：

![](img/d749730b-0f1f-415e-99cb-20c47419c2fe.png)

然后是堆的详细信息（这只是其中的一小部分）：

![](img/6f0074ff-b4b5-465c-b168-bcd5089fa685.png)

然后打印编译事件、GC 堆历史、去优化事件、内部异常、事件、动态库、日志选项、环境变量、VM 参数以及运行进程的系统的许多参数。

`jcmd`命令深入了解 JVM 进程，有助于调试和调整进程以获得最佳性能和最佳资源使用。

# 使用*try-with-resources*更好地处理资源

管理资源是很重要的。任何资源的错误处理（未释放）——例如保持打开的数据库连接和文件描述符——都可能耗尽系统的操作能力。这就是为什么在 JDK 7 中引入了*try-with-resources*语句。我们在第六章的示例中使用了它，*数据库编程*：

```java
try (Connection conn = getDbConnection();
Statement st = createStatement(conn)) {
  st.execute(sql);
} catch (Exception ex) {
  ex.printStackTrace();
}
```

作为提醒，这是`getDbConnection()`方法：

```java
Connection getDbConnection() {
  PGPoolingDataSource source = new PGPoolingDataSource();
  source.setServerName("localhost");
  source.setDatabaseName("cookbook");
  try {
    return source.getConnection(); 
  } catch(Exception ex) {
    ex.printStackTrace();
    return null;
  }
}
```

这是`createStatement()`方法：

```java
Statement createStatement(Connection conn) {
  try {
    return conn.createStatement();
  } catch(Exception ex) {
    ex.printStackTrace();
    return null;
  }
}
```

这非常有帮助，但在某些情况下，我们仍然需要以旧的方式编写额外的代码，例如，如果有一个接受`Statement`对象作为参数的`execute()`方法，并且我们希望在使用后立即释放（关闭）它。在这种情况下，代码将如下所示：

```java
void execute(Statement st, String sql){
  try {
    st.execute(sql);
  } catch (Exception ex) {
    ex.printStackTrace();
  } finally {
    if(st != null) {
      try{
        st.close();
      } catch (Exception ex) {
        ex.printStackTrace();
      }
    }
  }
}
```

正如您所看到的，其中大部分只是样板复制粘贴代码。

Java 9 引入的新*try-with-resources*语句通过允许有效地最终变量作为资源来解决了这种情况。

# 如何做...

1.  使用新的*try-with-resources*语句重写前面的示例：

```java
        void execute(Statement st, String sql){
          try (st) {
            st.execute(sql);
          } catch (Exception ex) {
            ex.printStackTrace();
          }
        }
```

正如您所看到的，它更加简洁和专注，无需反复编写关闭资源的琐碎代码。不再需要`finally`和额外的`try...catch`。

1.  如果连接也被传递进来，它也可以放在同一个 try 块中，并在不再需要时立即关闭：

```java
        void execute(Connection conn, Statement st, String sql) {
          try (conn; st) {
            st.execute(sql);
          } catch (Exception ex) {
            ex.printStackTrace();
          }
        }
```

它可能适合或不适合您应用程序的连接处理，但通常，这种能力是很方便的。

1.  尝试不同的组合，例如以下：

```java
        Connection conn = getDbConnection();
        Statement st = conn.createStatement();
        try (conn; st) {
          st.execute(sql);
        } catch (Exception ex) {
          ex.printStackTrace();
        }
```

这种组合也是允许的：

```java
        Connection conn = getDbConnection();
        try (conn; Statement st = conn.createStatement()) {
          st.execute(sql);
        } catch (Exception ex) {
          ex.printStackTrace();
        }
```

新语句提供了更灵活的编写代码的方式，以满足需求，而无需编写关闭资源的代码行。

唯一的要求如下：

+   +   `try`语句中包含的变量必须是 final 或有效最终

+   资源必须实现`AutoCloseable`接口，其中只包括一个方法：

```java
        void close() throws Exception;
```

# 它是如何工作的...

为了演示新语句的工作原理，让我们创建自己的资源，实现`AutoCloseable`并以与之前示例中的资源类似的方式使用它们。

这是一个资源：

```java
class MyResource1 implements AutoCloseable {
  public MyResource1(){
    System.out.println("MyResource1 is acquired");
  }
  public void close() throws Exception {
    //Do what has to be done to release this resource
    System.out.println("MyResource1 is closed");
  }
}
```

这是第二个资源：

```java
class MyResource2 implements AutoCloseable {
  public MyResource2(){
    System.out.println("MyResource2 is acquired");
  }
  public void close() throws Exception {
    //Do what has to be done to release this resource
    System.out.println("MyResource2 is closed");
  }
}
```

让我们在代码示例中使用它们：

```java
MyResource1 res1 = new MyResource1();
MyResource2 res2 = new MyResource2();
try (res1; res2) {
  System.out.println("res1 and res2 are used");
} catch (Exception ex) {
  ex.printStackTrace();
}
```

如果我们运行它，结果将如下：

![](img/27c7e219-b1e6-488a-b968-ce460476c73a.png)

请注意，在`try`语句中列出的第一个资源最后关闭。让我们只做一个改变，并在`try`语句中切换引用的顺序：

```java
MyResource1 res1 = new MyResource1();
MyResource2 res2 = new MyResource2();
try (res2; res1) {
  System.out.println("res1 and res2 are used");
} catch (Exception ex) {
  ex.printStackTrace();
}
```

输出确认了引用关闭的顺序也发生了变化：

![](img/b9d13e94-10a6-414f-89ae-61cb50692e48.png)

按照相反顺序关闭资源的规则解决了资源之间可能存在的最重要的依赖问题，但是由程序员定义关闭资源的顺序（通过在`try`语句中按正确顺序列出它们）是程序员的责任。幸运的是，大多数标准资源的关闭都由 JVM 优雅地处理，如果资源按照不正确的顺序列出，代码不会中断。但是，按照创建顺序列出它们是一个好主意。

# 用于改进调试的堆栈遍历

堆栈跟踪在找出问题的根源时非常有帮助。当可能进行自动更正时，需要以编程方式读取它。

自 Java 1.4 以来，可以通过`java.lang.Thread`和`java.lang.Throwable`类访问当前堆栈跟踪。您可以在代码的任何方法中添加以下行：

```java
Thread.currentThread().dumpStack();
```

您还可以添加以下行：

```java
new Throwable().printStackTrace();
```

它将堆栈跟踪打印到标准输出。或者，自 Java 8 以来，您可以使用以下任一行达到相同的效果：

```java
Arrays.stream(Thread.currentThread().getStackTrace())
      .forEach(System.out::println);

Arrays.stream(new Throwable().getStackTrace())
      .forEach(System.out::println);

```

或者您可以使用以下任一行提取调用者类的完全限定名称：

```java
System.out.println("This method is called by " + Thread.currentThread()
                                   .getStackTrace()[1].getClassName());

System.out.println("This method is called by " + new Throwable()
                                   .getStackTrace()[0].getClassName());
```

所有上述解决方案都是可能的，因为`java.lang.StackTraceElement`类代表堆栈跟踪中的堆栈帧。该类提供其他描述由此堆栈跟踪元素表示的执行点的方法，这允许以编程方式访问堆栈跟踪信息。例如，您可以在程序的任何位置运行此代码片段：

```java
Arrays.stream(Thread.currentThread().getStackTrace())
  .forEach(e -> {
    System.out.println();
    System.out.println("e="+e);
    System.out.println("e.getFileName()="+ e.getFileName());
    System.out.println("e.getMethodName()="+ e.getMethodName());
    System.out.println("e.getLineNumber()="+ e.getLineNumber());
});

```

或者您可以在程序的任何位置运行以下内容：

```java
Arrays.stream(new Throwable().getStackTrace())
  .forEach(x -> {
    System.out.println();
    System.out.println("x="+x);
    System.out.println("x.getFileName()="+ x.getFileName());
    System.out.println("x.getMethodName()="+ x.getMethodName());
    System.out.println("x.getLineNumber()="+ x.getLineNumber());
});

```

不幸的是，这些丰富的数据是有代价的。JVM 捕获整个堆栈（除了隐藏的堆栈帧），并且在程序堆栈跟踪的程序化分析嵌入主应用程序流程的情况下，可能会影响应用程序性能。与此同时，您只需要这些数据的一小部分来做出决策。

这就是新的 Java 9 类`java.lang.StackWalker`以及其嵌套的`Option`类和`StackFrame`接口派上用场的地方。

# 准备就绪

`StackWalker`类有四个重载的`getInstance()`静态工厂方法：

+   `StackWalker getInstance()`: 这是配置为跳过所有隐藏帧的实例，并且不保留调用者类引用。隐藏帧包含 JVM 内部实现特定的信息。不保留调用者类引用意味着在`StackWalker`对象上调用`getCallerClass()`方法会抛出`UnsupportedOperationException`。

+   `StackWalker getInstance(StackWalker.Option option)`: 这将创建一个具有给定选项的实例，指定它可以访问的堆栈帧信息。

+   `StackWalker getInstance(Set<StackWalker.Option> options)`: 这将创建一个具有给定选项集的实例，指定它可以访问的堆栈帧信息。如果给定的集合为空，则该实例的配置与`StackWalker getInstance()`创建的实例完全相同。

+   `StackWalker getInstance(Set<StackWalker.Option> options, int estimatedDepth)`: 这将创建一个与前一个实例类似的实例，并接受`estimatedDepth`参数，允许我们估计它可能需要的缓冲区大小。

以下是`enum StackWalker.Option`的值：

+   `StackWalker.Option.RETAIN_CLASS_REFERENCE`: 配置`StackWalker`实例以支持`getCallerClass()`方法，并且`StackFrame`支持`getDeclaringClass()`方法

+   `StackWalker.Option.SHOW_HIDDEN_FRAMES`: 配置`StackWalker`实例以显示所有反射帧和特定实现帧

+   `StackWalker.Option.SHOW_REFLECT_FRAMES`: 配置`StackWalker`实例以显示所有反射帧

`StackWalker`类还有三个方法：

+   `T walk(Function<Stream<StackWalker.StackFrame>, T> function)`: 这将给定的函数应用于当前线程的`StackFrames`流，从堆栈顶部遍历帧。顶部帧包含调用此`walk()`方法的方法。

+   `void forEach(Consumer<StackWalker.StackFrame> action)`: 这对当前线程的`StackFrame`流的每个元素执行给定的操作，从堆栈的顶部帧开始，这是调用`forEach`方法的方法。此方法相当于调用`walk(s -> { s.forEach(action); return null; })`。

+   `Class<?> getCallerClass()`: 这获取调用了调用`getCallerClass()`方法的方法的`Class`对象。如果此`StackWalker`实例未配置`RETAIN_CLASS_REFERENCE`选项，则此方法会抛出`UnsupportedOperationException`。

# 如何做...

创建几个类和方法，它们将相互调用，这样您就可以执行堆栈跟踪处理：

1.  创建一个`Clazz01`类：

```java
        public class Clazz01 {
          public void method(){
            new Clazz03().method("Do something");
            new Clazz02().method();
          }
        }
```

1.  创建一个`Clazz02`类：

```java
        public class Clazz02 {
          public void method(){
            new Clazz03().method(null);
          }
        }
```

1.  创建一个`Clazz03`类：

```java
        public class Clazz03 {
          public void method(String action){
            if(action != null){
              System.out.println(action);
              return;
            }
            System.out.println("Throw the exception:");
            action.toString();
          }
        }
```

1.  编写一个`demo4_StackWalk()`方法：

```java
        private static void demo4_StackWalk(){
          new Clazz01().method();
        }
```

从`Chapter11Memory`类的主方法中调用此方法：

```java
        public class Chapter11Memory {
          public static void main(String... args) {
            demo4_StackWalk();
          }
        }
```

如果我们现在运行`Chapter11Memory`类，结果将如下所示：

![](img/60ec130e-737d-49db-831f-f14f3fd5e717.png)

`Do something`消息从`Clazz01`传递并在`Clazz03`中打印出来。然后`Clazz02`将 null 传递给`Clazz03`，并在`action.toString()`行引起的`NullPointerException`的堆栈跟踪之前打印出`Throw the exception`消息。

# 它是如何工作的...

为了更深入地理解这里的概念，让我们修改`Clazz03`：

```java
public class Clazz03 {
  public void method(String action){
    if(action != null){
      System.out.println(action);
      return;
    }
    System.out.println("Print the stack trace:");
    Thread.currentThread().dumpStack();
  }
}
```

结果将如下所示：

![](img/fb839893-eba0-42a0-9ef6-7f8eccc8e86b.png)

或者，我们可以使用`Throwable`而不是`Thread`来获得类似的输出：

```java
new Throwable().printStackTrace();
```

前一行产生了这个输出：

![](img/cbefaa13-cce7-4a17-b224-fc6f2df44f43.png)

每个以下两行将产生类似的结果：

```java
Arrays.stream(Thread.currentThread().getStackTrace())
                             .forEach(System.out::println);
Arrays.stream(new Throwable().getStackTrace())
                             .forEach(System.out::println);

```

自 Java 9 以来，可以使用`StackWalker`类实现相同的输出。让我们看看如果我们修改`Clazz03`会发生什么：

```java
public class Clazz03 {
  public void method(String action){
    if(action != null){
      System.out.println(action);
      return;
    }
    StackWalker stackWalker = StackWalker.getInstance();
    stackWalker.forEach(System.out::println);
  }
}
```

结果如下：

![](img/0a56bc8a-4f69-4dc3-81a0-8cee5859d2bf.png)

它包含了传统方法产生的所有信息。然而，与在内存中生成和存储完整堆栈跟踪不同，`StackWalker`类只带来了请求的元素。这已经是一个很大的优点。然而，`StackWalker`的最大优势是，当我们只需要调用者类名时，而不是获取整个数组并仅使用一个元素，我们现在可以通过以下两行获取所需的信息：

```java
System.out.println("Print the caller class name:");
System.out.println(StackWalker.getInstance(StackWalker
                        .Option.RETAIN_CLASS_REFERENCE)
                        .getCallerClass().getSimpleName());

```

上述代码片段的结果如下：

![](img/370ad025-999a-46d6-afaf-3f2d7557c969.png)

# 使用内存感知编码风格

在编写代码时，程序员有两个主要目标：

+   实现所需的功能

+   编写易于阅读和理解的代码

然而，在这样做的同时，他们还必须做出许多其他决定，其中之一是使用与标准库类和方法具有类似功能的类。在这个示例中，我们将带您了解一些考虑因素，以帮助避免浪费内存，并使您的代码风格具有内存感知能力：

+   注意在循环内创建的对象

+   使用延迟初始化，在使用之前创建对象，特别是如果有很大的可能性，这种需求根本不会出现

+   不要忘记清理缓存并删除不必要的条目

+   使用`StringBuilder`而不是`+`运算符

+   如果符合您的需求，请使用`ArrayList`，然后再使用`HashSet`（从`ArrayList`到`LinkedList`，`HashTable`，`HashMap`和`HashSet`，内存使用量逐渐增加）

# 如何做...

1.  注意在循环内创建的对象。

这个建议非常明显。在快速连续创建和丢弃许多对象可能在垃圾收集器重新利用空间之前消耗太多内存。考虑重用对象而不是每次都创建一个新对象。这里有一个例子：

```java
class Calculator {
   public  double calculate(int i) {
       return Math.sqrt(2.0 * i);
   }
}

class SomeOtherClass {
   void reuseObject() {
      Calculator calculator = new Calculator();
      for(int i = 0; i < 100; i++ ){
          double r = calculator.calculate(i);
          //use result r
      }
   }
} 
```

前面的代码可以通过使`calculate()`方法静态来改进。另一个解决方案是创建`SomeOtherClass`类的静态属性`Calculator calculator = new Calculator()`。但是静态属性在类第一次加载时就会初始化。如果`calculator`属性没有被使用，那么它的初始化将是不必要的开销。在这种情况下，需要添加延迟初始化。

1.  使用延迟初始化，在使用之前创建对象，特别是如果有很大的可能性某些请求可能永远不会实现这个需求。

在前面的步骤中，我们谈到了`calculator`属性的延迟初始化：

```java
class Calculator {
    public  double calculate(int i) {
        return Math.sqrt(2.0 * i);
    }
}

class SomeOtherClass {
     private static Calculator calculator;
     private static Calculator getCalculator(){
        if(this.calculator == null){
            this.calculator = new Calculator();
        }
        return this.calculator;
     }
     void reuseObject() {
        for(int i = 0; i < 100; i++ ){
           double r = getCalculator().calculate(i);
           //use result r
      }
   }
} 
```

在前面的示例中，`Calculator`对象是一个单例 - 一旦创建，应用程序中就只存在一个实例。如果我们知道`calculator`属性总是会被使用，那么就不需要延迟初始化。在 Java 中，我们可以利用静态属性在任何应用程序线程加载类时的第一次初始化。

```java
class SomeOtherClass {
   private static Calculator calculator = new Calculator();
   void reuseObject() {
      for(int i = 0; i < 100; i++ ){
          double r = calculator.calculate(i);
          //use result r
      }
   }
}
```

但是，如果初始化的对象很可能永远不会被使用，我们又回到了可以在单线程中实现的延迟初始化（使用`getCalculator()`方法）或者当共享对象是无状态的且其初始化不消耗太多资源时。

在多线程应用程序和复杂对象初始化的情况下，需要采取一些额外措施来避免并发访问冲突，并确保只创建一个实例。例如，考虑以下类：

```java
class ExpensiveInitClass {
    private Object data;
    public ExpensiveInitClass() {
        //code that consumes resources
        //and assignes value to this.data
    }

    public Object getData(){
        return this.data;
    }
}
```

如果前面的构造函数需要大量时间来完成对象的创建，那么第二个线程有可能在第一个线程完成对象创建之前进入构造函数。为了避免第二个对象的并发创建，我们需要同步初始化过程：

```java
class LazyInitExample {
  public ExpensiveInitClass expensiveInitClass
  public Object getData(){  //can synchrnonize here
    if(this.expensiveInitClass == null){
      synchronized (LazyInitExample.class) {
        if (this.expensiveInitClass == null) {
          this.expensiveInitClass = new ExpensiveInitClass();
        }
      }
    }
    return expensiveInitClass.getData();
  }
}
```

如您所见，我们可以同步访问`getData()`方法，但在对象创建后不需要此同步，并且可能在高并发多线程环境中造成瓶颈。同样，我们可以只在同步块内部进行一次空值检查，但在对象初始化后不需要此同步，因此我们用另一个空值检查来减少瓶颈的机会。

1.  不要忘记清理缓存并删除不必要的条目。

缓存有助于减少访问数据的时间。但缓存会消耗内存，因此有意义的是尽可能保持它小，同时仍然有用。如何做取决于缓存数据使用的模式。例如，如果你知道一旦使用，存储在缓存中的对象不会再次被使用，你可以在应用程序启动时（或根据使用模式定期）将其放入缓存中，并在使用后从缓存中删除：

```java
static HashMap<String, Object> cache = new HashMap<>();
static {
    //populate the cache here
}
public Object getSomeData(String someKey) {
    Object obj = cache.get(someKey);
    cache.remove(someKey);
    return obj;
}
```

或者，如果您期望每个对象具有很高的可重用性，可以在第一次请求后将其放入缓存中：

```java
static HashMap<String, Object> cache = new HashMap<>();
public Object getSomeData(String someKey) {
    Object obj = cache.get(someKey);
    if(obj == null){
        obj = getDataFromSomeSource();
        cache.put(someKey, obj);
    }
    return obj;
}
```

前面的情况可能导致缓存无法控制地增长，消耗太多内存，并最终导致`OutOfMemoryError`条件。为了防止这种情况，您可以实现一个算法，限制缓存的大小 - 达到一定大小后，每次添加新对象时，都会删除一些其他对象（例如，最常用的对象或最少使用的对象）。以下是将缓存大小限制为 10 的示例，通过删除最常使用的缓存对象：

```java
static HashMap<String, Object> cache = new HashMap<>();
static HashMap<String, Integer> count = new HashMap<>();
public static Object getSomeData(String someKey) {
   Object obj = cache.get(someKey);
   if(obj == null){
       obj = getDataFromSomeSource();
       cache.put(someKey, obj);
       count.put(someKey, 1);
       if(cache.size() > 10){
          Map.Entry<String, Integer> max = 
             count.entrySet().stream()
             .max(Map.Entry.comparingByValue(Integer::compareTo))
             .get();
            cache.remove(max.getKey());
            count.remove(max.getKey());
        }
    } else {
        count.put(someKey, count.get(someKey) + 1);
    } 
    return obj;
}
```

或者，可以使用`java.util.WeakHashMap`类来实现缓存：

```java
private static WeakHashMap<Integer, Double> cache 
                                     = new WeakHashMap<>();
void weakHashMap() {
    int last = 0;
    int cacheSize = 0;
    for(int i = 0; i < 100_000_000; i++) {
        cache.put(i, Double.valueOf(i));
        cacheSize = cache.size();
        if(cacheSize < last){
            System.out.println("Used memory=" + 
              usedMemoryMB()+" MB, cache="  + cacheSize);
        }
        last = cacheSize;
    }
}
```

运行上面的示例，您会看到内存使用和缓存大小首先增加，然后下降，然后再次增加，然后再次下降。以下是输出的摘录：

```java
Used memory=1895 MB, cache=2100931
Used memory=189 MB, cache=95658
Used memory=296 MB, cache=271
Used memory=408 MB, cache=153
Used memory=519 MB, cache=350
Used memory=631 MB, cache=129
Used memory=745 MB, cache=2079710
Used memory=750 MB, cache=69590
Used memory=858 MB, cache=213
```

我们使用的内存使用量计算如下：

```java
long usedMemoryMB() {
   return Math.round(
      Double.valueOf(Runtime.getRuntime().totalMemory() - 
                     Runtime.getRuntime().freeMemory())/1024/1024
   );
}
```

`java.util.WeakHashMap`类是一个具有`java.lang.ref.WeakReference`类型键的 Map 实现。只有通过弱引用引用的对象在垃圾收集器决定需要更多内存时才会被回收。这意味着`WeakHashMap`对象中的条目将在没有对该键的引用时被移除。当垃圾收集器从内存中移除键时，相应的值也会从地图中移除。

在我们之前的示例中，缓存键都没有在地图之外使用，因此垃圾收集器会自行删除它们。即使我们在地图之外添加对键的显式引用，代码的行为也是相同的：

```java
private static WeakHashMap<Integer, Double> cache 
                                     = new WeakHashMap<>();
void weakHashMap() {
    int last = 0;
    int cacheSize = 0;
    for(int i = 0; i < 100_000_000; i++) {
        Integer iObj = i;
        cache.put(iObj, Double.valueOf(i));
        cacheSize = cache.size();
        if(cacheSize < last){
            System.out.println("Used memory=" + 
              usedMemoryMB()+" MB, cache="  + cacheSize);
        }
        last = cacheSize;
    }
}
```

这是因为在之前的代码块中显示的`iObj`引用在每次迭代后都被丢弃并被收集，因此缓存中的相应键也没有外部引用，垃圾收集器也会将其删除。为了证明这一点，让我们再次修改上面的代码：

```java
private static WeakHashMap<Integer, Double> cache 
                                     = new WeakHashMap<>();
void weakHashMap() {
    int last = 0;
    int cacheSize = 0;
    List<Integer> list = new ArrayList<>();
    for(int i = 0; i < 100_000_000; i++) {
        Integer iObj = i;
        cache.put(iObj, Double.valueOf(i));
        list.add(iObj);
        cacheSize = cache.size();
        if(cacheSize < last){
            System.out.println("Used memory=" + 
              usedMemoryMB()+" MB, cache="  + cacheSize);
        }
        last = cacheSize;
    }
}
```

我们创建了一个列表，并将地图的每个键添加到其中。如果我们运行上述代码，最终会得到`OutOfMemoryError`，因为缓存的键在地图之外有强引用。我们也可以减弱外部引用：

```java
private static WeakHashMap<Integer, Double> cache 
                                     = new WeakHashMap<>();
void weakHashMap() {
    int last = 0;
    int cacheSize = 0;
    List<WeakReference<Integer>> list = new ArrayList<>();
    for(int i = 0; i < 100_000_000; i++) {
        Integer iObj = i;
        cache.put(iObj, Double.valueOf(i));
 list.add(new WeakReference(iObj));
        cacheSize = cache.size();
        if(cacheSize < last){
            System.out.println("Used memory=" + 
              usedMemoryMB()+" MB, cache="  + cacheSize +
              ", list size=" + list.size());
        }
        last = cacheSize;
    }
}
```

上面的代码现在运行得好像缓存键没有外部引用一样。使用的内存和缓存大小会增长，然后再次下降。但是列表大小不会下降，因为垃圾收集器不会从列表中删除值。因此，最终应用程序可能会耗尽内存。

然而，无论您限制缓存的大小还是让其无法控制地增长，都可能出现应用程序需要尽可能多的内存的情况。因此，如果有一些对应用程序主要功能不是关键的大对象，有时将它们从内存中移除以使应用程序能够生存并避免出现`OutOfMemoryError`的情况是有意义的。

如果存在缓存，通常是一个很好的候选对象来释放内存，因此我们可以使用`WeakReference`类来包装缓存本身：

```java
private static WeakReference<Map<Integer, Double[]>> cache;
void weakReference() {
   Map<Integer, Double[]> map = new HashMap<>();
   cache = new WeakReference<>(map);
   map = null;
   int cacheSize = 0;
   List<Double[]> list = new ArrayList<>();
   for(int i = 0; i < 10_000_000; i++) {
      Double[] d = new Double[1024];
      list.add(d);
      if (cache.get() != null) {
          cache.get().put(i, d);
          cacheSize = cache.get().size();
          System.out.println("Cache="+cacheSize + 
                  ", used memory=" + usedMemoryMB()+" MB");
      } else {
          System.out.println(i +": cache.get()=="+cache.get()); 
          break;
      }
   }
}
```

在上面的代码中，我们将地图（缓存）包装在`WeakReference`类中，这意味着我们告诉 JVM 只要没有对它的引用，就可以收集此对象。然后，在每次 for 循环迭代中，我们创建一个`new Double[1024]`对象并将其保存在列表中。我们这样做是为了更快地使用完所有可用内存。然后我们将相同的对象放入缓存中。当我们运行此代码时，它会迅速得到以下输出：

```java
Cache=4582, used memory=25 MB
4582: cache.get()==null

```

这意味着垃圾收集器在使用了 25MB 内存后决定收集缓存对象。如果您认为这种方法太过激进，而且您不需要经常更新缓存，您可以将其包装在`java.lang.ref.SoftReference`类中。如果这样做，缓存只有在所有内存用完时才会被收集——就在即将抛出`OutOfMemoryError`的边缘。以下是演示它的代码片段：

```java
private static SoftReference<Map<Integer, Double[]>> cache;
void weakReference() {
   Map<Integer, Double[]> map = new HashMap<>();
   cache = new SoftReference<>(map);
   map = null;
   int cacheSize = 0;
   List<Double[]> list = new ArrayList<>();
   for(int i = 0; i < 10_000_000; i++) {
      Double[] d = new Double[1024];
      list.add(d);
      if (cache.get() != null) {
          cache.get().put(i, d);
          cacheSize = cache.get().size();
          System.out.println("Cache="+cacheSize + 
                      ", used memory=" + usedMemoryMB()+" MB");
      } else {
          System.out.println(i +": cache.get()=="+cache.get()); 
          break;
      }
   }
}
```

如果我们运行它，输出将如下所示：

```java
Cache=1004737, used memory=4096 MB
1004737: cache.get()==null
```

没错，在我们的测试计算机上，有 4GB 的 RAM，因此只有在几乎用完所有内存时才会删除缓存。

1.  使用`StringBuilder`代替`+`运算符。

您可以在互联网上找到许多这样的建议。也有相当多的声明说这个建议已经过时，因为现代 Java 使用`StringBuilder`来实现字符串的`+`运算符。以下是我们实验的结果。首先，我们运行了以下代码：

```java
long um = usedMemoryMB();
String s = "";
for(int i = 1000; i < 10_1000; i++ ){
    s += Integer.toString(i);
    s += " ";
}
System.out.println("Used memory: " 
         + (usedMemoryMB() - um) + " MB");  //prints: 71 MB

```

`usedMemoryMB()`的实现：

```java
long usedMemoryMB() {
   return Math.round(
      Double.valueOf(Runtime.getRuntime().totalMemory() - 
                  Runtime.getRuntime().freeMemory())/1024/1024
   );
}
```

然后我们用`StringBuilder`来达到同样的目的：

```java
long um = usedMemoryMB();
StringBuilder sb = new StringBuilder();
for(int i = 1000; i < 10_1000; i++ ){
    sb.append(Integer.toString(i)).append(" ");
}
System.out.println("Used memory: " 
         + (usedMemoryMB() - um) + " MB");  //prints: 1 MB
```

正如你所看到的，使用`+`运算符消耗了 71MB 的内存，而`StringBuilder`仅在相同任务中使用了 1MB。我们也测试了`StringBuffer`。它也消耗了 1MB，但比`StringBuilder`执行稍慢，因为它是线程安全的，而`StringBuilder`只能在单线程环境中使用。

所有这些都不适用于长字符串值，该值已被拆分为几个子字符串，以提高可读性。编译器将子字符串收集回一个长值。例如，`s1`和`s2`字符串占用相同的内存量：

```java
String s1 = "this " +
            "string " +
            "takes " +
            "as much memory as another one";
String s2 = "this string takes as much memory as another one";
```

1.  如果需要使用集合，如果符合你的需求，选择`ArrayList`。从`ArrayList`到`LinkedList`、`HashTable`、`HashMap`和`HashSet`，内存使用量逐渐增加。

`ArrayList`对象将其元素存储在`Object[]`数组中，并使用一个`int`字段来跟踪列表的大小（除了`array.length`）。由于这样的设计，如果有可能这个容量不会被充分使用，那么在声明时不建议分配一个大容量的`ArrayList`。当新元素添加到列表中时，后端数组的容量会以 10 个元素的块递增，这可能是浪费内存的一个可能来源。如果这对应用程序很重要，可以通过调用`trimToSize()`方法来缩小`ArrayList`的容量到当前使用的容量。请注意，`clear()`和`remove()`方法不会影响`ArrayList`的容量，它们只会改变其大小。

其他集合的开销更大，因为它们提供了更多的服务。`LinkedList`元素不仅携带对前一个和后一个元素的引用，还携带对数据值的引用。大多数基于哈希的集合实现都专注于更好的性能，这往往是以内存占用为代价的。

如果集合的大小很小，那么选择 Java 集合类可能是无关紧要的。然而，程序员通常使用相同的编码模式，通过其风格可以识别代码的作者。因此，长远来看，找出最有效的构造并经常使用它们是值得的。但是，尽量避免使你的代码难以理解；可读性是代码质量的一个重要方面。

# 更好地使用内存的最佳实践

内存管理可能永远不会成为你的问题，它可能会成为你每一个清醒的时刻，或者你可能会发现自己处于这两个极端之间。大多数情况下，对于大多数程序员来说，这都不是问题，尤其是随着不断改进的垃圾回收算法。G1 垃圾收集器（JVM 9 中的默认值）绝对是朝着正确方向迈出的一步。但也有可能你会被要求（或者自己注意到）应用程序性能下降的情况，这时你就会了解你有多少能力来应对挑战。

这个示例是为了帮助你避免这种情况或成功摆脱它而做出的尝试。

# 如何做...

第一道防线是代码本身。在之前的示例中，我们讨论了释放资源的必要性，以及使用`StackWalker`来消耗更少的内存。互联网上有很多建议，但它们可能不适用于你的应用程序。你需要监控内存消耗并测试你的设计决策，特别是如果你的代码处理大量数据，然后才决定在哪里集中你的注意力。

一旦你的代码开始做它应该做的事情，就测试和分析你的代码。你可能需要改变你的设计或一些实现的细节。这也会影响你未来的决策。任何环境都有许多分析器和诊断工具可用。我们在*使用 jcmd 命令进行 JVM*示例中描述了其中的一个，`jcmd`。

了解您的垃圾收集器是如何工作的（参见*了解 G1 垃圾收集器*配方），并且不要忘记使用 JVM 日志记录（在*JVM 的统一日志记录*配方中描述）。

在那之后，您可能需要调整 JVM 和垃圾收集器。以下是一些经常使用的`java`命令行参数（默认情况下，大小以字节指定，但您可以附加字母 k 或 K 表示千字节，m 或 M 表示兆字节，g 或 G 表示千兆字节）：

+   `-Xms size`：此选项允许我们设置初始堆大小（必须大于 1 MB 且是 1024 的倍数）。

+   `-Xmx size`：此选项允许我们设置最大堆大小（必须大于 2 MB 且是 1024 的倍数）。

+   `-Xmn size`或`-XX:NewSize=size`和`-XX:MaxNewSize=size`的组合：此选项允许我们设置年轻代的初始和最大大小。为了有效的 GC，它必须低于`-Xmx size`。Oracle 建议将其设置为堆大小的 25%以上但低于 50%。

+   `-XX:NewRatio=ratio`：此选项允许我们设置年轻代和老年代之间的比率（默认为两个）。

+   `-Xss size`：此选项允许我们设置线程堆栈大小。不同平台的默认值如下：

+   Linux/ARM（32 位）：320 KB

+   Linux/ARM（64 位）：1,024 KB

+   Linux/x64（64 位）：1,024 KB

+   macOS（64 位）：1,024 KB

+   Oracle Solaris/i386（32 位）：320 KB

+   Oracle Solaris/x64（64 位）：1,024 KB

+   Windows：取决于虚拟内存

+   `-XX:MaxMetaspaceSize=size`：此选项允许我们设置类元数据区的上限（默认情况下没有限制）。

内存泄漏的明显迹象是老年代的增长导致完整 GC 更频繁地运行。要进行调查，您可以使用将堆内存转储到文件的 JVM 参数：

+   `-XX:+HeapDumpOnOutOfMemoryError`：允许我们将 JVM 堆内容保存到文件中，但仅当抛出`java.lang.OutOfMemoryError`异常时。默认情况下，堆转储保存在当前目录中，名称为`java_pid<pid>.hprof`，其中`<pid>`是进程 ID。使用`-XX:HeapDumpPath=<path>`选项来自定义转储文件位置。`<path>`值必须包括文件名。

+   `-XX:OnOutOfMemoryError="<cmd args>;<cmd args>"`：允许我们提供一组命令（用分号分隔），当抛出`OutOfMemoryError`异常时将执行这些命令。

+   `-XX:+UseGCOverheadLimit`：调节 GC 占用时间比例的大小，超过这个比例会抛出`OutOfMemoryError`异常。例如，并行 GC 将在 GC 占用时间超过 98%且恢复的堆不到 2%时抛出`OutOfMemoryError`异常。此选项在堆较小时特别有用，因为它可以防止 JVM 在几乎没有进展的情况下运行。默认情况下已启用。要禁用它，请使用`-XX:-UseGCOverheadLimit`。

# 了解 Epsilon，一种低开销的垃圾收集器

一个流行的 Java 面试问题是，*您能强制进行垃圾收集吗？* Java 运行时内存管理仍然不受程序员控制，有时会像一个不可预测的小丑一样打断本来表现良好的应用程序，并启动全内存扫描。它通常发生在*最糟糕的时候*。当您尝试在负载下使用短时间运行来测量应用程序性能时，后来意识到大量时间和资源都花在了垃圾收集过程上，并且在更改代码后，垃圾收集的模式变得与更改代码之前不同，这尤其令人恼火。

在本章中，我们描述了许多编程技巧和解决方案，可以帮助减轻垃圾收集器的压力。然而，它仍然是应用程序性能的独立和不可预测的贡献者（或减少者）。如果垃圾收集器能够更好地受控制，至少在测试目的中，或者可以关闭，那不是很好吗？在 Java 11 中，引入了一个名为 Epsilon 的垃圾收集器，称为无操作垃圾收集器。

乍一看，这看起来很奇怪——一个不收集任何东西的垃圾收集器。但它是可预测的（这是肯定的），因为它什么也不做，这个特性使我们能够在短时间内测试算法，而不用担心不可预测的暂停。此外，还有一整类需要在短时间内尽可能利用所有资源的小型短期应用程序，最好重新启动 JVM 并让负载均衡器执行故障转移，而不是尝试考虑垃圾收集过程中不可预测的 Joker。

它也被设想为一个基准过程，可以让我们估计常规垃圾收集器的开销。

# 如何做...

要调用无操作垃圾收集器，请使用`-XX:+UseEpsilonGC`选项。在撰写本文时，它需要一个`-XX:+UnlockExperimentalVMOptions`选项来访问新功能。

我们将使用以下程序进行演示：

```java
package com.packt.cookbook.ch11_memory;
import java.util.ArrayList;
import java.util.List;
public class Epsilon {
    public static void main(String... args) {
        List<byte[]> list = new ArrayList<>();
        int n = 4 * 1024 * 1024;
        for(int i=0; i < n; i++){
            list.add(new byte[1024]);
            byte[] arr = new byte[1024];
        }
    }
}
```

正如您所看到的，在这个程序中，我们试图通过在每次迭代中向列表添加 1KB 数组来分配 4GB 的内存。与此同时，我们还在每次迭代中创建一个 1K 数组`arr`，但不使用对它的引用，因此传统的垃圾收集器可以收集它。

首先，我们将使用默认的垃圾收集器运行前面的程序：

```java
time java -cp cookbook-1.0.jar -Xms4G -Xmx4G -Xlog:gc com.packt.cookbook.ch11_memory.Epsilon
```

请注意，我们已将 JVM 堆内存限制为 4GB，因为出于演示目的，我们希望程序以`OutOfMemoryError`退出。我们已经使用`time`命令包装了调用以捕获三个值：

+   **实际时间**：程序运行的时间

+   **用户时间**：程序使用 CPU 的时间

+   **系统时间**：操作系统为程序工作的时间

我们使用了 JDK 11：

```java
java -version
java version "11-ea" 2018-09-25
Java(TM) SE Runtime Environment 18.9 (build 11-ea+22)
Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11-ea+22, mixed mode)
```

在您的计算机上，前面的命令的输出可能会有所不同。在我们的测试运行期间，当我们使用指定的`java`命令参数执行前面的程序时，输出以以下四行开头：

```java
Using G1
GC(0) Pause Young (Normal) (G1 Evacuation Pause) 204M->101M(4096M)
GC(1) Pause Young (Normal) (G1 Evacuation Pause) 279M->191M(4096M)
GC(2) Pause Young (Normal) (G1 Evacuation Pause) 371M->280M(4096M)
```

正如您所看到的，G1 垃圾收集器是 JDK 11 中的默认值，并且它立即开始收集未引用的`arr`对象。正如我们所预期的那样，程序在`OutOfMemoryError`后退出：

```java
GC(50) Pause Full (G1 Evacuation Pause) 4090M->4083M(4096M)
GC(51) Concurrent Cycle 401.931ms
GC(52) To-space exhausted
GC(52) Pause Young (Concurrent Start) (G1 Humongous Allocation)
GC(53) Concurrent Cycle
GC(54) Pause Young (Normal) (G1 Humongous Allocation) 4088M->4088M(4096M)
GC(55) Pause Full (G1 Humongous Allocation) 4088M->4085M(4096M)
GC(56) Pause Full (G1 Humongous Allocation) 4085M->4085M(4096M)
GC(53) Concurrent Cycle 875.061ms
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
 at java.base/java.util.Arrays.copyOf(Arrays.java:3720)
 at java.base/java.util.Arrays.copyOf(Arrays.java:3689)
 at java.base/java.util.ArrayList.grow(ArrayList.java:237)
 at java.base/java.util.ArrayList.grow(ArrayList.java:242)
 at java.base/java.util.ArrayList.add(ArrayList.java:485)
 at java.base/java.util.ArrayList.add(ArrayList.java:498)
 at com.packt.cookbook.ch11_memory.Epsilon.main(Epsilon.java:12)
```

时间实用程序产生了以下结果：

```java
real 0m11.549s    //How long the program ran
user 0m35.301s    //How much time the CPU was used by the program
sys 0m19.125s     //How much time the OS worked for the program
```

我们的计算机是多核的，因此 JVM 能够并行利用多个核心，很可能是用于垃圾收集。这就是为什么用户时间比实际时间长，系统时间也因同样的原因比实际时间长。

现在让我们用以下命令运行相同的程序：

```java
time java -cp cookbook-1.0.jar -XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC -Xms4G -Xmx4G -Xlog:gc com.packt.cookbook.ch11_memory.Epsilon
```

请注意，我们已添加了`-XX:+UnlockExperimentalVMOptions -XX:+UseEpsilonGC`选项，这需要 Epsilon 垃圾收集器。结果如下：

```java
Non-resizeable heap; start/max: 4096M
Using TLAB allocation; max: 4096K
Elastic TLABs enabled; elasticity: 1.10x
Elastic TLABs decay enabled; decay time: 1000ms
Using Epsilon
Heap: 4096M reserved, 4096M (100.00%) committed, 205M (5.01%) used
Heap: 4096M reserved, 4096M (100.00%) committed, 410M (10.01%) used
Heap: 4096M reserved, 4096M (100.00%) committed, 614M (15.01%) used
Heap: 4096M reserved, 4096M (100.00%) committed, 820M (20.02%) used
Heap: 4096M reserved, 4096M (100.00%) committed, 1025M (25.02%) used
Heap: 4096M reserved, 4096M (100.00%) committed, 1230M (30.03%) used
Heap: 4096M reserved, 4096M (100.00%) committed, 1435M (35.04%) used
Heap: 4096M reserved, 4096M (100.00%) committed, 1640M (40.04%) used
Heap: 4096M reserved, 4096M (100.00%) committed, 1845M (45.05%) used
Heap: 4096M reserved, 4096M (100.00%) committed, 2050M (50.05%) used
Heap: 4096M reserved, 4096M (100.00%) committed, 2255M (55.06%) used
Heap: 4096M reserved, 4096M (100.00%) committed, 2460M (60.06%) used
Heap: 4096M reserved, 4096M (100.00%) committed, 2665M (65.07%) used
Heap: 4096M reserved, 4096M (100.00%) committed, 2870M (70.07%) used
Heap: 4096M reserved, 4096M (100.00%) committed, 3075M (75.08%) used
Heap: 4096M reserved, 4096M (100.00%) committed, 3280M (80.08%) used
Heap: 4096M reserved, 4096M (100.00%) committed, 3485M (85.09%) used
Heap: 4096M reserved, 4096M (100.00%) committed, 3690M (90.09%) used
Heap: 4096M reserved, 4096M (100.00%) committed, 3895M (95.10%) used
Terminating due to java.lang.OutOfMemoryError: Java heap space
```

正如您所看到的，垃圾收集器甚至没有尝试收集被丢弃的对象。堆空间的使用量稳步增长，直到完全耗尽，并且 JVM 以`OutOfMemoryError`退出。使用`time`实用程序允许我们测量三个时间参数：

```java
real 0m4.239s
user 0m1.861s
sys 0m2.132s
```

自然地，耗尽所有堆内存所需的时间要少得多，用户时间要比实际时间少得多。这就是为什么，正如我们已经提到的那样，无操作的 Epsilon 垃圾收集器对于那些必须尽可能快速但不会消耗所有堆内存或可以随时停止的程序可能是有用的。可能还有其他垃圾收集器不做任何事情可能有帮助的用例。
