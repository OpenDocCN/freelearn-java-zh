# 第五章：并发

并发是 Clojure 的主要设计目标之一。考虑到 Java 中的并发编程模型（与 Java 的比较是因为它是 JVM 上的主要语言），它不仅太底层，而且如果不严格遵循模式，很容易出错，从而自食其果。锁、同步和无保护变异是并发陷阱的配方，除非极端谨慎地使用。Clojure 的设计选择深刻影响了并发模式以安全且功能的方式实现的方式。在本章中，我们将讨论：

+   硬件和 JVM 级别的底层并发支持

+   Clojure 的并发原语——原子、代理、引用和变量

+   Java 内置的并发特性既安全又实用，与 Clojure 的结合

+   使用 Clojure 特性和 reducers 进行并行化

# 底层并发

没有显式的硬件支持，无法实现并发。我们在前几章讨论了 SMT 和多核处理器。回想一下，每个处理器核心都有自己的 L1 缓存，而多个核心共享 L2 缓存。共享的 L2 缓存为处理器核心提供了一个快速机制来协调它们的缓存访问，消除了相对昂贵的内存访问。此外，处理器将写入内存的操作缓冲到一个称为**脏写缓冲区**的地方。这有助于处理器发布一系列内存更新请求，重新排序指令，并确定写入内存的最终值，这被称为**写吸收**。

## 硬件内存屏障（栅栏）指令

内存访问重排序对于顺序（单线程）程序的性能来说非常好，但对于并发程序来说则是有害的，因为在某个线程中内存访问的顺序可能会破坏另一个线程的预期。处理器需要一种同步访问的手段，以便内存重排序要么被限制在不在乎的代码段中，要么在可能产生不良后果的地方被阻止。硬件通过“内存屏障”（也称为“栅栏”）这一安全措施来支持这种功能。

在不同的架构中可以发现多种内存屏障指令，它们可能具有不同的性能特征。编译器（或者在 JVM 的情况下是 JIT 编译器）通常了解它在上面运行的架构上的 fence 指令。常见的 fence 指令有读、写、获取和释放屏障等。屏障并不保证最新的数据，而是只控制内存访问的相对顺序。屏障会在所有写入发布后、屏障对发出它的处理器可见之前，将写缓冲区刷新。

读写屏障分别控制读和写的顺序。写操作通过写缓冲区进行；但读操作可能发生顺序混乱，或者来自写缓冲区。为了保证正确的顺序，使用获取和释放，块/屏障被使用。获取和释放被认为是 "半屏障"；两者一起（获取和释放）形成一个 "全屏障"。全屏障比半屏障更昂贵。

## Java 支持和 Clojure 等价物

在 Java 中，内存屏障指令是由高级协调原语插入的。尽管栅栏指令的运行成本很高（数百个周期），但它们提供了一个安全网，使得在关键部分内访问共享变量是安全的。在 Java 中，`synchronized` 关键字标记一个 "关键部分"，一次只能由一个线程执行，因此它是一个 "互斥" 工具。在 Clojure 中，Java 的 `synchronized` 的等价物是 `locking` 宏：

```java
// Java example
synchronized (someObject) {
    // do something
}
;; Clojure example
(locking some-object
  ;; do something
  )

```

`locking` 宏建立在两个特殊形式 `monitor-enter` 和 `monitor-exit` 之上。请注意，`locking` 宏是一个低级和命令式的解决方案，就像 Java 的 `synchronized` 一样——它们的使用并不被认为是 Clojure 的惯用用法。特殊形式 `monitor-enter` 和 `monitor-exit` 分别进入和退出锁对象的 "monitor"——它们甚至更低级，不建议直接使用。

测量使用此类锁定代码的性能的人应该意识到其单线程与多线程延迟之间的差异。在单线程中锁定是廉价的。然而，当有两个或更多线程争夺同一对象监视器的锁时，性能惩罚就开始显现。锁是在称为 "内在" 或 "monitor" 锁的对象监视器上获取的。对象等价性（即，当 `=` 函数返回 true）永远不会用于锁定的目的。确保从不同的线程锁定时对象引用是相同的（即，当 `identical?` 返回 true）。

线程通过获取监视器锁涉及一个读屏障，该屏障使线程本地的缓存数据、相应的处理器寄存器和缓存行无效。这迫使从内存中重新读取。另一方面，释放监视器锁会导致写屏障，将所有更改刷新到内存中。这些是昂贵的操作，会影响并行性，但它们确保了所有线程的数据一致性。

Java 支持一个`volatile`关键字，用于类中的数据成员，它保证在同步块之外对属性的读取和写入不会发生重排序。值得注意的是，除非属性被声明为`volatile`，否则它不能保证在所有访问它的线程中都是可见的。Clojure 中 Java 的`volatile`等价于我们在第三章, *依赖 Java*中讨论的元数据`^:volatile-mutable`。以下是在 Java 和 Clojure 中使用`volatile`的示例：

```java
// Java example
public class Person {
    volatile long age;
}
;; Clojure example
(deftype Person [^:volatile-mutable ^long age])
```

读取和写入`volatile`数据需要分别使用读获取或写释放，这意味着我们只需要一个半屏障就可以单独读取或写入值。请注意，由于半屏障，读取后跟写入的操作不保证是原子的。例如，`age++`表达式首先读取值，然后增加并设置它。这使得有两个内存操作，这不再是半屏障。

Clojure 1.7 引入了对`volatile`数据的一级支持，使用一组新的函数：`volatile!`、`vswap!`、`vreset!`和`volatile?`。这些函数定义了可变的（可变的）数据，并与之一起工作。然而，请注意，这些函数不与`deftype`中的可变字段一起工作。以下是如何使用它们的示例：

```java
user=> (def a (volatile! 10))
#'user/a
user=> (vswap! a inc)
11
user=> @a
11
user=> (vreset! a 20)
20
user=> (volatile? a)
true
```

对`volatile`数据的操作不是原子的，这就是为什么即使创建`volatile`（使用`volatile!`）也被认为是潜在不安全的。一般来说，`volatile`可能在读取一致性不是高优先级但写入必须快速的情况下很有用，例如实时趋势分析或其他此类分析报告。`volatile`在编写有状态的转换器（参考第二章, *Clojure 抽象*)时也非常有用，作为非常快速的状态容器。在下一小节中，我们将看到其他比`volatile`更安全（但大多数情况下更慢）的状态抽象。

# 原子更新和状态

读取数据元素、执行一些逻辑，并更新为新值的操作是一个常见的用例。对于单线程程序，这不会产生任何后果；但对于并发场景，整个操作必须作为一个原子操作同步执行。这种情况如此普遍，以至于许多处理器在硬件级别使用特殊的比较并交换（CAS）指令来支持这一功能，这比锁定要便宜得多。在 x86/x64 架构上，这个指令被称为 CompareExchange（CMPXCHG）。

不幸的是，可能存在另一种线程更新了变量，其值与正在执行原子更新的线程将要比较的旧值相同。这被称为“ABA”问题。在某些其他架构中发现的“Load-linked”（LL）和“Store-conditional”（SC）等指令集提供了一种没有 ABA 问题的 CAS 替代方案。在 LL 指令从地址读取值之后，用于更新地址的新值的 SC 指令只有在 LL 指令成功后地址未被更新的情况下才会执行。

## Java 中的原子更新

Java 提供了一组内置的无锁、原子、线程安全的比较和交换抽象，用于状态管理。它们位于`java.util.concurrent.atomic`包中。对于布尔型、整型和长型等原始类型，分别有`AtomicBoolean`、`AtomicInteger`和`AtomicLong`类。后两个类支持额外的原子加减操作。对于原子引用更新，有`AtomicReference`、`AtomicMarkableReference`和`AtomicStampedReference`类用于任意对象。还有对数组的支持，其中数组元素可以原子性地更新——`AtomicIntegerArray`、`AtomicLongArray`和`AtomicReferenceArray`。它们易于使用；以下是一个示例：

```java
(import 'java.util.concurrent.atomic.AtomicReference)
(def ^AtomicReference x (AtomicReference. "foo"))
(.compareAndSet x "foo" "bar")
(import 'java.util.concurrent.atomic.AtomicInteger)
(def ^AtomicInteger y (AtomicInteger. 10))
(.getAndAdd y 5)
```

然而，在哪里以及如何使用它取决于更新点和代码中的逻辑。原子更新不保证是非阻塞的。在 Java 中，原子更新不是锁的替代品，而是一种便利，仅当范围限制为对一个可变变量的比较和交换操作时。

## Clojure 对原子更新的支持

Clojure 的原子更新抽象称为`atom`。它底层使用`AtomicReference`。对`AtomicInteger`或`AtomicLong`的操作可能比 Clojure 的`atom`稍微快一些，因为前者使用原语。但由于它们在 CPU 中使用的比较和交换指令，它们并不便宜。速度实际上取决于突变发生的频率以及 JIT 编译器如何优化代码。速度的好处可能不会在代码运行了几十万次之后显现出来，并且原子频繁突变会增加重试的延迟。在实际（或类似实际）负载下测量延迟可以提供更好的信息。以下是一个使用原子的示例：

```java
user=> (def a (atom 0))
#'user/a
user=> (swap! a inc)
1
user=> @a
1
user=> (compare-and-set! a 1 5)
true
user=> (reset! a 20)
20
```

`swap!` 函数在执行原子更新时提供了一种与 `compareAndSwap(oldval, newval)` 方法明显不同的风格。当 `compareAndSwap()` 比较并设置值时，如果成功则返回 true，如果失败则返回 false，而 `swap!` 则会持续在一个无限循环中尝试更新，直到成功。这种风格是 Java 开发者中流行的模式。然而，与更新循环风格相关的一个潜在陷阱也存在。随着更新者的并发性提高，更新的性能可能会逐渐下降。再次，原子更新的高并发性引发了一个问题：对于使用场景来说，是否真的有必要进行无协调的更新。《compare-and-set!` 和 `reset!` 是相当直接的。

传递给 `swap!` 的函数必须是无副作用的（即纯函数），因为在竞争时该函数会在循环中多次重试。如果函数不是纯函数，副作用可能会在重试的次数内发生。

值得注意的是，原子不是“协调”的，这意味着当原子被不同的线程并发使用时，我们无法预测操作在其上工作的顺序，也无法保证最终结果。围绕原子的代码应该考虑到这种约束来设计。在许多场景中，由于缺乏协调，原子可能并不适合——在设计程序时要注意这一点。原子通过额外的参数支持元数据和基本验证机制。以下示例说明了这些功能：

```java
user=> (def a (atom 0 :meta {:foo :bar}))
user=> (meta a)
{:foo :bar}
user=> (def age (atom 0 :validator (fn [x] (if (> x 200) false true))))
user=> (reset! age 200)
200
user=> (swap! age inc)
IllegalStateException Invalid reference state  clojure.lang.ARef.validate (ARef.java:33)
```

第二个重要的事情是，原子支持在它们上添加和移除监视器。我们将在本章后面讨论监视器。

### 使用原子条带化实现更快的写入

我们知道当多个线程尝试同时更新状态时，原子会表现出竞争。这意味着当写入不频繁时，原子具有很好的性能。有一些用例，例如度量计数器，需要快速且频繁的写入，但读取较少，可以容忍一些不一致性。对于这样的用例，我们不必将所有更新都指向单个原子，而可以维护一组原子，其中每个线程更新不同的原子，从而减少竞争。从这些原子中读取的值不能保证是一致的。让我们开发这样一个计数器的示例：

```java
(def ^:const n-cpu (.availableProcessors (Runtime/getRuntime)))
(def counters (vec (repeatedly n-cpu #(atom 0))))
(defn inc! []
  ;; consider java.util.concurrent.ThreadLocalRandom in Java 7+
  ;; which is faster than Math/random that rand-int is based on
  (let [i (rand-int n-cpu)]
    (swap! (get counters i) inc)))
(defn value []
  (transduce (map deref) + counters))
```

在上一个示例中，我们创建了一个名为`counters`的向量，其大小与计算机中 CPU 核心的数量相同，并将每个元素初始化为初始值为 0 的原子。名为`inc!`的函数通过从`counters`中随机选择一个原子并增加 1 来更新计数器。我们还假设`rand-int`在所有处理器核心上均匀地分布了原子的选择，因此我们几乎没有任何竞争。`value`函数简单地遍历所有原子，并将它们的`deref`返回值相加以返回计数器的值。该示例使用`clojure.core/rand-int`，它依赖于`java.lang.Math/random`（由于 Java 6 支持）来随机找到下一个计数器原子。让我们看看当使用 Java 7 或更高版本时，我们如何优化它：

```java
(import 'java.util.concurrent.ThreadLocalRandom)
(defn inc! []
  (let [i (.nextLong (ThreadLocalRandom/current) n-cpu)]
    (swap! (get counters i) inc)))
```

在这里，我们`导入`了`java.util.concurrent.ThreadLocalRandom`类，并定义了`inc!`函数，使用`ThreadLocalRandom`来选择下一个随机原子。其他一切保持不变。

# 异步代理和状态

当原子是同步的，代理（agents）是 Clojure 中实现状态变化的异步机制。每个代理都与一个可变状态相关联。我们向代理传递一个函数（称为“动作”），并附带可选的额外参数。这个函数由代理排队在另一个线程中处理。所有代理共享两个公共线程池——一个用于低延迟（可能为 CPU 密集型、缓存密集型或内存密集型）的工作，另一个用于阻塞（可能为 I/O 相关或长时间处理）的工作。Clojure 提供了`send`函数用于低延迟动作，`send-off`用于阻塞动作，以及`send-via`来在用户指定的线程池上执行动作，而不是预配置的线程池之一。`send`、`send-off`和`send-via`都会立即返回。以下是我们可以如何使用它们的示例：

```java
(def a (agent 0))
;; invoke (inc 0) in another thread and set state of a to result
(send a inc)
@a  ; returns 1
;; invoke (+ 1 2 3) in another thread and set state of a to result
(send a + 2 3)
@a  ; returns 6

(shutdown-agents)  ; shuts down the thread-pools
;; no execution of action anymore, hence no result update either
(send a inc)
@a  ; returns 6
```

当我们检查 Clojure（截至版本 1.7.0）的源代码时，我们可以发现低延迟动作的线程池被命名为`pooledExecutor`（一个有界线程池，初始化为最大'2 + 硬件处理器数量'个线程），而高延迟动作的线程池被命名为`soloExecutor`（一个无界线程池）。这种默认配置的前提是 CPU/缓存/内存密集型动作在有限线程池上运行最优化，默认线程数。I/O 密集型任务不消耗 CPU 资源。因此，可以同时执行相对较多的此类任务，而不会显著影响 CPU/缓存/内存密集型工作的性能。以下是您可以如何访问和覆盖线程池的示例：

```java
(import 'clojure.lang.Agent)
Agent/pooledExecutor  ; thread-pool for low latency actions
Agent/soloExecutor  ; thread-pool for I/O actions
(import 'java.util.concurrent.Executors)
(def a-pool (Executors/newFixedThreadPool 10))  ; thread-pool with 10 threads
(def b-pool (Executors/newFixedThreadPool 100)) ; 100 threads pool
(def a (agent 0))
(send-via a-pool a inc)  ; use 'a-pool' for the action
(set-agent-send-executor! a-pool)  ; override default thread-pool
(set-agent-send-off-executor! b-pool)  ; override default pool
```

如果一个程序通过代理执行大量 I/O 或阻塞操作，限制用于此类操作的线程数量可能是有意义的。使用`set-agent-send-off-executor!`覆盖`send-off`线程池是限制线程池大小的最简单方法。通过使用`send-via`与适合各种 I/O 和阻塞操作的适当大小的线程池，可以更细致地隔离和限制代理上的 I/O 操作。

## 异步、队列和错误处理

向代理发送操作会立即返回，而不会阻塞。如果代理尚未忙于执行任何操作，它将通过将触发操作执行的队列中的操作入队来“反应”，在相应的线程池中。如果代理正在忙于执行另一个操作，新操作将简单地入队。一旦从操作队列中执行了操作，就会检查队列是否有更多条目，如果找到，则触发下一个操作。这个触发操作的“反应”机制消除了需要消息循环或轮询队列的需要。这之所以可能，是因为控制了指向代理队列的入口点。

动作在代理上异步执行，这引发了如何处理错误的疑问。错误情况需要通过一个显式、预定义的函数来处理。当使用默认的代理构造，如`(agent :foo)`时，代理在没有错误处理程序的情况下创建，并在任何异常发生时挂起。它缓存异常，并拒绝接受更多操作。在代理重启之前，它会在发送任何操作时抛出缓存的异常。可以使用`restart-agent`函数重置挂起的代理。这种挂起的目的是安全和监督。当在代理上执行异步操作时，如果突然发生错误，则需要引起注意。查看以下代码：

```java
(def g (agent 0))
(send g (partial / 10))  ; ArithmeticException due to divide-by-zero
@g  ; returns 0, because the error did not change the old state
(send g inc)  ; throws the cached ArithmeticException
(agent-error g)  ; returns (doesn't throw) the exception object
(restart-agent g @g)  ; clears the suspension of the agent
(agent-error g)  ; returns nil
(send g inc)  ; works now because we cleared the cached error
@g  ; returns 1
(dotimes [_ 1000] (send-off g long-task))
;; block for 100ms or until all actions over (whichever earlier)
(await-for 100 g)
(await g)  ; block until all actions dispatched till now are over
```

有两个可选参数`:error-handler`和`:error-mode`，我们可以在代理上配置这些参数以对错误处理和挂起有更精细的控制，如下面的代码片段所示：

```java
(def g (agent 0 :error-handler (fn [x] (println "Found:" x))))  ; incorrect arity
(send g (partial / 10))  ; no error encountered because error-handler arity is wrong
(def g (agent 0 :error-handler (fn [ag x] (println "Found:" x))))  ; correct arity
(send g (partial / 10))  ; prints the message
(set-error-handler! g (fn [ag x] (println "Found:" x)))  ; equiv of :error-handler arg
(def h (agent 0 :error-mode :continue))
(send h (partial / 10))  ; error encountered, but agent not suspended
(send h inc)
@h  ; returns 1
(set-error-mode! h :continue)  ; equiv of :error-mode arg, other possible value :fail
```

## 为什么你应该使用代理

正如“原子”实现只使用比较和交换而不是锁定一样，底层的“代理”特定实现主要使用比较和交换操作。代理实现仅在事务（将在下一节讨论）中调度操作或重启代理时使用锁定。所有操作都在代理中排队并串行分发，无论并发级别如何。这种串行性质使得可以独立且无争用地执行操作。对于同一个代理，永远不会同时执行多个操作。由于没有锁定，对代理的读取（`deref`或`@`）永远不会因为写入而被阻塞。然而，所有操作都是相互独立的——它们的执行没有重叠。

实现甚至确保动作的执行会阻塞队列中后续的动作。尽管动作是在线程池中执行的，但同一代理的动作永远不会并发执行。这是一个出色的排序保证，也由于其串行性质而扩展了自然的协调机制。然而，请注意，这种排序协调仅限于单个代理。如果一个代理的动作发送给两个其他代理，它们不会自动协调。在这种情况下，您可能希望使用事务（将在下一节中介绍）。

由于代理区分低延迟和阻塞作业，作业将在适当的线程池中执行。不同代理上的动作可以并发执行，从而最大限度地利用线程资源。与原子不同，代理的性能不会因高竞争而受阻。事实上，对于许多情况，由于动作的串行缓冲，代理非常有意义。一般来说，代理非常适合高容量 I/O 任务，或者在操作顺序在高度竞争场景中提供优势的情况下。

## 嵌套

当一个代理的动作发送给同一个代理的另一个动作时，这就是嵌套的情况。如果代理不参与 STM 事务（将在下一节中介绍），这本来可能没有什么特别之处。然而，代理确实参与了 STM 事务，这给代理的实现带来了一定的约束，需要对动作进行第二层缓冲。目前，可以说嵌套发送被排队在代理的线程局部队列中，而不是常规队列中。线程局部队列只对执行动作的线程可见。在执行动作时，除非出现错误，否则代理会隐式调用相当于`release-pending-sends`函数的功能，将动作从第二级线程局部队列转移到正常动作队列。请注意，嵌套只是代理的实现细节，没有其他影响。

# 协调事务引用和状态

在前面的章节中，我们了解到原子提供了原子读-更新操作。如果我们需要在两个或更多原子之间执行原子读-更新操作怎么办？这显然是一个协调问题。某个实体必须监控读取和更新的过程，以确保值不会被破坏。这就是引用的作用——提供一个基于**软件事务内存（STM**）的系统，该系统负责在多个引用之间执行并发原子读-更新操作，以确保所有更新都通过，或者在失败的情况下，没有任何更新。与原子一样，在失败的情况下，引用会从头开始重试整个操作，使用新的值。

Clojure 的 STM 实现是粗粒度的。它工作在应用级别的对象和聚合（即聚合的引用），范围仅限于程序中的所有 refs，构成了“Ref 世界”。对 ref 的任何更新都只能以同步方式发生，在事务中，在 `dosync` 代码块内，在同一线程中。它不能跨越当前线程。实现细节揭示，在事务的生命周期内维护了一个线程本地的交易上下文。一旦控制达到另一个线程，相同的上下文就不再可用。

与 Clojure 中的其他引用类型一样，对 ref 的读取永远不会被更新阻塞，反之亦然。然而，与其他引用类型不同，ref 的实现并不依赖于无锁自旋，而是内部使用锁、低级等待/通知、死锁检测和基于年龄的抢占。

`alter` 函数用于读取和更新一个 ref 的值，而 `ref-set` 用于重置值。大致来说，对于 refs 来说，`alter` 和 `ref-set` 类似于原子操作中的 `swap!` 和 `reset!`。就像 `swap!` 一样，`alter` 接受一个无副作用的函数（和参数），并且可能在竞争时重试多次。然而，与原子不同，不仅 `alter`，而且 `ref-set` 和简单的 `deref` 也可能在竞争时导致事务重试。以下是一个如何使用事务的非常简单的例子：

```java
(def r1 (ref [:a :b :c]))
(def r2 (ref [1 2 3]))
(alter r1 conj :d)  ; IllegalStateException No transaction running...
(dosync (let [v (last @r1)] (alter r1 pop) (alter r2 conj v)))
@r1  ; returns [:a :b]
@r2  ; returns [1 2 3 :c]
(dosync (ref-set r1 (conj @r1 (last @r2))) (ref-set r2 (pop @r2)))
@r1  ; returns [:a :b :c]
@r2  ; returns [1 2 3]
```

## Ref 特性

Clojure 在事务中维护了 **原子性**、**一致性**和**隔离性**（**ACI**）特性。这与许多数据库提供的 ACID 保证中的 A、C 和 I 相重叠。原子性意味着事务中的所有更新要么全部成功完成，要么全部不完成。一致性意味着事务必须保持一般正确性，并应遵守验证设置的约束——任何异常或验证错误都应回滚事务。除非共享状态受到保护，否则对它的并发更新可能导致多步事务在不同步骤中看到不同的值。隔离性意味着事务中的所有步骤都将看到相同的值，无论更新多么并发。

Clojure 引用使用一种称为**多版本并发控制（MVCC**）的技术来为事务提供**快照隔离**。在 MVCC 中，不是通过锁定（这可能会阻塞事务），而是维护队列，以便每个事务都可以使用自己的快照副本进行操作，该副本在其“读点”处获取，独立于其他事务。这种方法的主要好处是，只读事务外的操作可以无冲突地通过。没有引用冲突的事务可以并发进行。在与数据库系统的粗略比较中，Clojure 引用隔离级别在读取事务外的引用时为“读取已提交”，而在事务内部默认为“可重复读”。

## 引用历史和在事务中解除引用操作

我们之前讨论过，对引用的读取和更新操作都可能导致事务重试。事务中的读取可以配置为使用引用历史，这样快照隔离实例就存储在历史队列中，并由事务中的读操作使用。默认情况下，不使用历史队列，这可以节省堆空间，并在事务中提供强一致性（避免数据陈旧）。

使用引用历史可以降低因读冲突导致的事务重试的可能性，从而提供弱一致性。因此，它是一种性能优化的工具，但会牺牲一致性。在许多场景中，程序不需要强一致性——如果我们知道权衡利弊，我们可以适当选择。Clojure 引用实现中的快照隔离机制由自适应历史队列支持。历史队列会动态增长以满足读请求，并且不会超过为引用设置的极限。默认情况下，历史记录是禁用的，因此我们需要在初始化时指定它，或者稍后设置。以下是如何使用历史的示例：

```java
(def r (ref 0 :min-history 5 :max-history 10))
(ref-history-count r)  ; returns 0, because no snapshot instances are queued so far
(ref-min-history r)  ; returns 5
(ref-max-history r)  ; returns 10
(future (dosync (println "Sleeping 20 sec") (Thread/sleep 20000) (ref-set r 10)))
(dosync (alter r inc))  ; enter this within few seconds after the previous expression
;; The message "Sleeping 20 sec" should appear twice due to transaction-retry
(ref-history-count r)  ; returns 2, the number of snapshot history elements
(.trimHistory ^clojure.lang.Ref r)
(ref-history-count r)  ; returns 0 because we wiped the history
(ref-min-history r 10)  ; reset the min history
(ref-max-history r 20)  ; reset the max history count
```

最小/最大历史限制与数据陈旧窗口的长度成比例。它还取决于更新和读操作之间的相对延迟差异，以确定在给定主机系统上最小历史和最大历史的工作范围。可能需要一些尝试和错误才能得到正确的范围。作为一个粗略的估计，读操作只需要足够的 min-history 元素来避免事务重试，因为在一次读操作期间可以有那么多更新。max-history 元素可以是 min-history 的倍数，以覆盖任何历史超限或欠限。如果相对延迟差异不可预测，那么我们必须为最坏情况规划一个 min-history，或者考虑其他方法。

## 事务重试和抢占

一个事务可以在五个不同的状态之一内部运行——运行中、提交中、重试、已杀死和已提交。一个事务可能因为各种原因而被杀死。异常是杀死事务的常见原因。但让我们考虑一个特殊情况，即一个事务被多次重试，但似乎没有成功提交——解决方案是什么？Clojure 支持基于年龄的抢占，其中较老的事务会自动尝试中止较新的事务，以便较新的事务稍后重试。如果抢占仍然不起作用，作为最后的手段，在 10,000 次重试尝试的硬性限制之后，事务将被杀死，然后抛出异常。

## 使用 ensure 提高事务一致性

Clojure 的事务一致性在性能和安全之间取得了良好的平衡。然而，有时我们可能需要**可序列化**的一致性来保持事务的正确性。具体来说，在面临事务重试的情况下，当一个事务的正确性依赖于一个 ref 的状态，在该事务中，ref 被另一个事务同时更新时，我们有一个称为“写偏斜”的条件。关于写偏斜的维基百科条目[`en.wikipedia.org/wiki/Snapshot_isolation`](https://en.wikipedia.org/wiki/Snapshot_isolation)，描述得很好，但让我们看看一个更具体的例子。假设我们想要设计一个具有两个引擎的飞行模拟系统，并且系统级的一个约束是不允许同时关闭两个引擎。如果我们把每个引擎建模为一个 ref，并且某些机动确实需要我们关闭一个引擎，我们必须确保另一个引擎是开启的。我们可以使用`ensure`来实现。通常，当需要在 refs 之间维护一致的关系（不变性）时，就需要`ensure`。这不能通过验证函数来保证，因为它们只有在事务提交时才会起作用。验证函数将看到相同的值，因此无法提供帮助。

写偏斜可以通过同名的`ensure`函数来解决，该函数本质上阻止其他事务修改 ref。它类似于锁定操作，但在实践中，当重试代价高昂时，它提供了比显式的读取和更新操作更好的并发性。使用`ensure`非常简单——`(ensure ref-object)`。然而，由于它在事务期间持有的锁，它可能在性能上代价高昂。使用`ensure`来管理性能涉及到在重试延迟和由于确保状态而丢失的吞吐量之间进行权衡。

## 使用交换操作减少事务重试

交换操作与它们应用的顺序无关。例如，从事务 t1 和 t2 中递增计数器引用 c1 将产生相同的效果，无论 t1 和 t2 提交更改的顺序如何。引用为事务中可交换的更改函数提供了特殊的优化——`commute`函数，它与`alter`（相同的语法）类似，但具有不同的语义。像`alter`一样，`commute`函数在事务提交期间原子性地应用。然而，与`alter`不同，`commute`不会在竞争时导致事务重试，并且没有关于`commute`函数应用顺序的保证。这实际上使得`commute`在作为操作结果返回有意义值时几乎无用。事务中的所有`commute`函数都在事务提交期间使用事务中的最终引用值重新应用。

如我们所见，交换操作减少了竞争，从而优化了整体事务吞吐量的性能。一旦我们知道一个操作是可交换的，并且我们不会以有意义的方式使用它的返回值，那么在决定是否使用它时几乎没有权衡——我们只需继续使用它。实际上，考虑到引用事务，考虑到交换，程序设计不是一个坏主意。

## 代理可以参与事务

在关于代理的上一节中，我们讨论了代理如何与排队更改函数协同工作。代理也可以参与引用事务，从而使得在事务中结合使用引用和代理成为可能。然而，代理不包括在“引用世界”中，因此事务作用域不会扩展到代理中更改函数的执行。相反，事务仅确保发送给代理的更改在事务提交发生前被排队。

*嵌套*子节，在关于代理的早期章节中，讨论了一个第二层线程局部队列。这个线程局部队列在事务期间用于持有发送给代理的更改，直到提交。线程局部队列不会阻塞发送给代理的其他更改。事务外部的更改永远不会在线程局部队列中缓冲；相反，它们被添加到代理中的常规队列。

代理参与事务提供了有趣的设计角度，其中协调的以及独立/顺序操作可以作为工作流程进行流水线处理，以获得更好的吞吐量和性能。

## 嵌套事务

Clojure 事务具有嵌套感知性并且可以很好地组合。但是，为什么需要嵌套事务呢？通常，独立的代码单元可能有自己的低粒度事务，高级代码可以利用这些事务。当高级调用者本身需要将操作包装在事务中时，就会发生嵌套事务。嵌套事务有自己的生命周期和运行状态。然而，外部事务可以在检测到失败时取消内部事务。

“ref 世界”快照`ensure`和`commute`在嵌套事务的所有（即外部和内部）级别之间共享。因此，内部事务被视为在外部事务内部的其他引用更改操作（类似于`alter`、`ref-set`等）。监视和内部锁的实现由各自的嵌套级别处理。内部事务中的竞争检测会导致内部和外部事务的重新启动。所有级别的提交最终在最外层事务提交时作为全局状态生效。监视器，尽管在每个单独的事务级别上跟踪，但在提交时最终生效。仔细观察嵌套事务实现可以看出，嵌套对事务性能的影响很小或没有。

## 性能考虑

Clojure Ref 可能是迄今为止实现的最复杂的引用类型。由于其特性，特别是其事务重试机制，在高度竞争场景下，这样的系统可能会有良好的性能可能并不立即明显。

理解其细微差别和最佳使用方式将有所帮助：

+   我们在事务中不使用具有副作用的变化，除非可能将 I/O 变化发送到代理，其中变化被缓冲直到提交。因此，根据定义，我们在事务中不执行任何昂贵的 I/O 工作。因此，这项工作的重试成本也会很低。

+   事务的更改函数应该尽可能小。这降低了延迟，因此重试的成本也会更低。

+   任何没有与至少另一个引用同时更新的引用不需要是引用——在这种情况下原子就足够好了。现在，由于引用只在组中才有意义，它们的竞争与组大小成正比。在事务中使用的小组引用导致低竞争、低延迟和高吞吐量。

+   交换函数提供了在不产生任何惩罚的情况下提高事务吞吐量的好机会。识别这些情况并考虑交换进行设计可以帮助显著提高性能。

+   引用非常粗粒度——它们在应用聚合级别工作。通常，程序可能需要更细粒度地控制事务资源。这可以通过引用条带化实现，例如 Megaref ([`github.com/cgrand/megaref`](https://github.com/cgrand/megaref))，通过提供关联引用的受限视图，从而允许更高的并发性。

+   在高竞争场景中，事务中引用组的大小不能小，考虑使用代理，因为它们由于序列性质而没有竞争。代理可能不是事务的替代品，但我们可以使用由原子、引用和代理组成的管道，以减轻竞争与延迟的担忧。

引用和事务的实现相当复杂。幸运的是，我们可以检查源代码，并浏览可用的在线和离线资源。

# 动态变量绑定和状态

在 Clojure 的引用类型中，第四种是动态变量。从 Clojure 1.3 开始，所有变量默认都是静态的。必须显式声明变量以使其成为动态的。一旦声明，动态变量就可以在每线程的基础上绑定到新的值。不同线程上的绑定不会相互阻塞。这里有一个示例：

```java
(def ^:dynamic *foo* "bar")
(println *foo*)  ; prints bar
(binding [*foo* "baz"] (println *foo*))  ; prints baz
(binding [*foo* "bar"] (set! *foo* "quux") (println *foo*))  ; prints quux
```

由于动态绑定是线程局部的，所以在多线程场景中使用可能会很棘手。长期以来，动态变量被库和应用滥用，作为传递给多个函数使用的公共参数的手段。然而，这种风格被认为是一种反模式，并受到谴责。通常，在反模式动态变量中，变量会被宏包装，以在词法作用域中包含动态线程局部绑定。这会导致多线程和懒序列出现问题。

那么，如何有效地使用动态变量呢？动态变量的查找成本比静态变量查找更高。即使是传递一个函数参数，在性能上也要比查找动态变量便宜得多。绑定动态变量会带来额外的开销。显然，在性能敏感的代码中，最好根本不使用动态变量。然而，在复杂或递归调用图场景中，动态变量可能非常有用，用于持有临时的线程局部状态，在这些场景中，性能并不重要，而且不会被宣传或泄露到公共 API 中。动态变量绑定可以像栈一样嵌套和展开，这使得它们既吸引人又适合这类任务。

# 验证和监视引用类型

变量（静态和动态）、原子、引用和代理提供了一种验证作为状态设置的价值的方法——一个接受新值作为参数的 `validator` 函数，如果成功则返回逻辑 true，如果出错则抛出异常/返回逻辑 false（错误和 nil 值）。它们都尊重验证函数返回的结果。如果成功，更新将通过，如果出错，则抛出异常。以下是声明验证器并将其与引用类型关联的语法：

```java
(def t (atom 1 :validator pos?))
(def g (agent 1 :validator pos?))
(def r (ref 1 :validator pos?))
(swap! t inc)  ; goes through, because value after increment (2) is positive
(swap! t (constantly -3))  ; throws exception
(def v 10)
(set-validator! (var v) pos?)
(set-validator! t (partial < 10)) ; throws exception
(set-validator! g (partial < 10)) ; throws exception
(set-validator! r #(< % 10)) ; works
```

验证器在更新引用类型时会导致实际失败。对于变量和原子，它们通过抛出异常简单地阻止更新。在代理中，验证失败会导致代理失败，需要代理重新启动。在引用内部，验证失败会导致事务回滚并重新抛出异常。

观察引用类型变化的另一种机制是“观察者”。与验证者不同，观察者是被动的——它在更新发生后才会被通知。因此，观察者无法阻止更新通过，因为它只是一个通知机制。对于事务，观察者仅在事务提交后才会被调用。虽然一个引用类型上只能设置一个验证者，但另一方面，可以将多个观察者与一个引用类型关联。其次，在添加观察者时，我们可以指定一个键，这样通知就可以通过键来识别，并由观察者相应地处理。以下是使用观察者的语法：

```java
(def t (atom 1))
(defn w [key iref oldv newv] (println "Key:" key "Old:" oldv "New:" newv))
(add-watch t :foo w)
(swap! t inc)  ; prints "Key: :foo Old: 1 New: 2"
```

与验证者一样，观察者是在引用类型的线程中同步执行的。对于原子和引用，这可能没问题，因为通知观察者的同时，其他线程可以继续进行它们的更新。然而在代理中，通知发生在更新发生的同一线程中——这使得更新延迟更高，吞吐量可能更低。

# Java 并发数据结构

Java 有许多旨在并发和线程安全的可变数据结构，这意味着多个调用者可以同时安全地访问这些数据结构，而不会相互阻塞。当我们只需要高度并发的访问而不需要状态管理时，这些数据结构可能非常适合。其中一些采用了无锁算法。我们已经在*原子更新和状态部分*讨论了 Java 原子状态类，所以这里不再重复。相反，我们只讨论并发队列和其他集合。

所有这些数据结构都位于 `java.util.concurrent` 包中。这些并发数据结构是为了利用 JSR 133 “Java 内存模型和线程规范修订” ([`gee.cs.oswego.edu/dl/jmm/cookbook.html`](http://gee.cs.oswego.edu/dl/jmm/cookbook.html)) 的实现而量身定制的，该实现首次出现在 Java 5 中。

## 并发映射

Java 有一个可变的并发哈希映射——`java.util.concurrent.ConcurrentHashMap`（简称 CHM）。在实例化类时可以可选地指定并发级别，默认为 16。CHM 实现在内部将映射条目分区到哈希桶中，并使用多个锁来减少每个桶的竞争。读取永远不会被写入阻塞，因此它们可能是过时的或不一致的——这种情况通过内置的检测来应对，并发出锁以再次以同步方式读取数据。这是针对读取数量远多于写入的场景的优化。在 CHM 中，所有单个操作几乎都是常数时间，除非由于锁竞争陷入重试循环。

与 Clojure 的持久映射相比，CHM 不能接受 `null`（`nil`）作为键或值。Clojure 的不可变标量和集合自动适合与 CHM 一起使用。需要注意的是，只有 CHM 中的单个操作是原子的，并表现出强一致性。由于 CHM 操作是并发的，聚合操作提供的致性比真正的操作级致性要弱。以下是我们可以使用 CHM 的方法。在 CHM 中提供更好一致性的单个操作是安全的。聚合操作应保留在我们知道其一致性特征和相关权衡的情况下使用：

```java
(import 'java.util.concurrent.ConcurrentHashMap)
(def ^ConcurrentHashMap m (ConcurrentHashMap.))
(.put m :english "hi")                    ; individual operation
(.get m :english)                           ; individual operation
(.putIfAbsent m :spanish "alo")    ; individual operation
(.replace m :spanish "hola")         ; individual operation
(.replace m :english "hi" "hello")  ; individual compare-and-swap atomic operation
(.remove m :english)                     ; individual operation
(.clear m)    ; aggregate operation
(.size m)      ; aggregate operation
(count m)    ; internally uses the .size() method
;; aggregate operation
(.putAll m {:french "bonjour" :italian "buon giorno"})
(.keySet m)  ; aggregate operation
(keys m)      ; calls CHM.entrySet() and on each pair java.util.Map.Entry.getKey()
(vals m)       ; calls CHM.entrySet() and on each pair java.util.Map.Entry.getValue()
```

`java.util.concurrent.ConcurrentSkipListMap` 类（简称 CSLM）是 Java 中另一种并发可变映射数据结构。CHM 和 CSLM 之间的区别在于，CSLM 在所有时间都提供具有 O(log N) 时间复杂度的排序视图。默认情况下，排序视图具有键的自然顺序，可以通过在实例化 CSLM 时指定 Comparator 实现来覆盖。CSLM 的实现基于跳表，并提供导航操作。

`java.util.concurrent.ConcurrentSkipListSet` 类（简称 CSLS）是一个基于 CSLM 实现的并发可变集合。虽然 CSLM 提供了映射 API，但 CSLS 在行为上类似于集合数据结构，同时借鉴了 CSLM 的特性。

## 并发队列

Java 内置了多种可变和并发内存队列的实现。队列数据结构是用于缓冲、生产者-消费者风格实现以及将这些单元管道化以形成高性能工作流程的有用工具。我们不应将它们与用于批量作业中类似目的的持久队列混淆。Java 的内存队列不是事务性的，但它们只为单个队列操作提供原子性和强一致性保证。聚合操作提供较弱的致性。

`java.util.concurrent.ConcurrentLinkedQueue` (CLQ) 是一个无锁、无阻塞的无界“先进先出” (FIFO) 队列。FIFO 意味着一旦元素被添加到队列中，队列元素的顺序就不会改变。CLQ 的 `size()` 方法不是一个常数时间操作；它取决于并发级别。这里有一些使用 CLQ 的例子：

```java
(import 'java.util.concurrent.ConcurrentLinkedQueue)
(def ^ConcurrentLinkedQueue q (ConcurrentLinkedQueue.))
(.add q :foo)
(.add q :bar)
(.poll q)  ; returns :foo
(.poll q)  ; returns :bar
```

| 队列 | 是否阻塞？ | 是否有界？ | FIFO？ | 公平性？ | 备注 |
| --- | --- | --- | --- | --- | --- |
| CLQ | 否 | 否 | 是 | 否 | 无阻塞，但 size() 不是常数时间操作 |
| ABQ | 是 | 是 | 是 | 可选 | 容量在实例化时固定 |
| DQ | 是 | 否 | 否 | 否 | 元素实现了 Delayed 接口 |
| LBQ | 是 | 可选 | 是 | 否 | 容量灵活，但没有公平性选项 |
| PBQ | 是 | 否 | 否 | 否 | 元素按优先级顺序消费 |
| SQ | 是 | – | – | 可选 | 没有容量；它充当一个通道 |

在 `java.util.concurrent` 包中，`ArrayBlockingQueue` (ABQ)，`DelayQueue` (DQ)，`LinkedBlockingQueue` (LBQ)，`PriorityBlockingQueue` (PBQ) 和 `SynchronousQueue` (SQ) 实现了 `BlockingQueue` (BQ) 接口。它的 Javadoc 描述了其方法调用的特性。ABQ 是一个基于数组的固定容量、FIFO 队列。LBQ 也是一个 FIFO 队列，由链表节点支持，并且是可选的有界（默认 `Integer.MAX_VALUE`）。ABQ 和 LBQ 通过阻塞满容量时的入队操作来生成“背压”。ABQ 支持可选的公平性（有性能开销），按访问它的线程顺序。

DQ 是一个无界队列，接受与延迟相关的元素。队列元素不能为 null，并且必须实现 `java.util.concurrent.Delayed` 接口。元素只有在延迟过期后才能从队列中移除。DQ 对于在不同时间安排元素的处理非常有用。

PBQ 是无界且阻塞的，同时允许按优先级从队列中消费元素。元素默认具有自然排序，可以通过在实例化队列时指定 Comparator 实现来覆盖。

SQ 实际上根本不是一个队列。相反，它只是生产者或消费者线程的一个屏障。生产者会阻塞，直到消费者移除元素，反之亦然。SQ 没有容量。然而，SQ 支持可选的公平性（有性能开销），按线程访问的顺序。

Java 5 之后引入了一些新的并发队列类型。自从 JDK 1.6 以来，在`java.util.concurrent`包中，Java 有**BlockingDeque**（**BD**），其中**LinkedBlockingDeque**（**LBD**）是唯一可用的实现。BD 通过添加**Deque**（双端队列）操作来构建在 BQ 之上，即从队列两端添加元素和消费元素的能力。LBD 可以实例化一个可选的容量（有限制）以阻塞溢出。JDK 1.7 引入了**TransferQueue**（**TQ**），其中**LinkedTransferQueue**（**LTQ**）是唯一实现。TQ 以扩展 SQ 概念的方式，使得生产者和消费者阻塞元素队列。这将通过保持它们忙碌来更好地利用生产者和消费者线程。LTQ 是 TQ 的无限制实现，其中`size()`方法不是常数时间操作。

## Clojure 对并发队列的支持

我们在《Clojure 抽象》的第二章中介绍了持久队列。Clojure 有一个内置的`seque`函数，它基于 BQ 实现（默认为 LBQ）来暴露预写序列。序列可能是惰性的，预写缓冲区控制要实现多少元素。与块大小为 32 的块序列不同，预写缓冲区的大小是可控制的，并且在所有时间都可能被填充，直到源序列耗尽。与块序列不同，32 个元素的块不会突然实现。它是逐渐和平稳地实现的。

在底层，Clojure 的`seque`使用代理在预写缓冲区中填充数据。在`seque`的 2 参数版本中，第一个参数应该是正整数，或者是一个 BQ（ABQ、LBQ 等）的实例，最好是有限制的。

# 使用线程的并发

在 JVM 上，线程是并发的事实上的基本工具。多个线程生活在同一个 JVM 中；它们共享堆空间，并竞争资源。

## JVM 对线程的支持

JVM 线程是操作系统线程。Java 将底层 OS 线程包装为`java.lang.Thread`类的实例，并围绕它构建 API 以与线程一起工作。JVM 上的线程有多个状态：新建、可运行、阻塞、等待、定时等待和终止。线程通过覆盖`Thread`类的`run()`方法或通过将`java.lang.Runnable`接口的实例传递给`Thread`类的构造函数来实例化。

调用`Thread`实例的`start()`方法将在新线程中启动其执行。即使 JVM 中只运行一个线程，JVM 也不会关闭。调用带有参数`true`的线程的`setDaemon(boolean)`方法将线程标记为守护线程，如果没有其他非守护线程正在运行，则可以自动关闭该线程。

所有 Clojure 函数都实现了 `java.lang.Runnable` 接口。因此，在新的线程中调用一个函数非常简单：

```java
(defn foo5 [] (dotimes [_ 5] (println "Foo")))
(defn barN [n] (dotimes [_ n] (println "Bar")))
(.start (Thread. foo5))  ; prints "Foo" 5 times
(.start (Thread. (partial barN 3)))  ; prints "Bar" 3 times
```

`run()` 方法不接受任何参数。我们可以通过创建一个不需要参数的高阶函数来解决这个问题，但内部应用参数 `3`。

## JVM 中的线程池

创建线程会导致操作系统 API 调用，这并不总是便宜的。一般的做法是创建一个线程池，可以回收用于不同任务。Java 有内置的线程池支持。名为 `java.util.concurrent.ExecutorService` 的接口代表了线程池的 API。创建线程池最常见的方式是使用 `java.util.concurrent.Executors` 类中的工厂方法：

```java
(import 'java.util.concurrent.Executors)
(import 'java.util.concurrent.ExecutorService)
(def ^ExecutorService a (Executors/newSingleThreadExecutor))  ; bounded pool
(def ^ExecutorService b (Executors/newCachedThreadPool))  ; unbounded pool
(def ^ExecutorService c (Executors/newFixedThreadPool 5))  ; bounded pool
(.execute b #(dotimes [_ 5] (println "Foo")))  ; prints "Foo" 5 times
```

之前的例子等同于我们在上一小节中看到的原始线程的例子。线程池也有能力帮助跟踪新线程中执行函数的完成情况和返回值。ExecutorService 接受一个 `java.util.concurrent.Callable` 实例作为参数，用于启动任务的几个方法，并返回 `java.util.concurrent.Future` 以跟踪最终结果。

所有 Clojure 函数也实现了 `Callable` 接口，因此我们可以如下使用它们：

```java
(import 'java.util.concurrent.Callable)
(import 'java.util.concurrent.Future)
(def ^ExecutorService e (Executors/newSingleThreadExecutor))
(def ^Future f (.submit e (cast Callable #(reduce + (range 10000000)))))
(.get f)  ; blocks until result is processed, then returns it
```

这里描述的线程池与我们在之前的代理部分中简要看到的相同。当不再需要时，线程池需要通过调用 `shutdown()` 方法来关闭。

## Clojure 并发支持

Clojure 有一些巧妙的内置功能来处理并发。我们已经在之前的小节中讨论了代理，以及它们如何使用线程池。Clojure 中还有一些其他并发功能来处理各种用例。

### Future

在本节中，我们之前已经看到了如何使用 Java API 来启动一个新线程，以执行一个函数。我们还学习了如何获取结果。Clojure 有一个内置的支持称为 "futures"，以更平滑和集成的方式完成这些事情。futures 的基础是 `future-call` 函数（它接受一个无参数函数作为参数），以及基于前者的宏 `future`（它接受代码体）。两者都会立即启动一个线程来执行提供的代码。以下代码片段说明了与 future 一起工作的函数以及如何使用它们：

```java
;; runs body in new thread
(def f (future (println "Calculating") (reduce + (range 1e7))))
(def g (future-call #(do (println "Calculating") (reduce + (range 1e7)))))  ; takes no-arg fn
(future? f)                  ; returns true
(future-cancel g)        ; cancels execution unless already over (can stop mid-way)
(future-cancelled? g) ; returns true if canceled due to request
(future-done? f)         ; returns true if terminated successfully, or canceled
(realized? f)               ; same as future-done? for futures
@f                              ; blocks if computation not yet over (use deref for timeout)
```

`future-cancel` 的一个有趣方面是，它有时不仅能够取消尚未开始的任务，还可能中止那些正在执行一半的任务：

```java
(let [f (future (println "[f] Before sleep")
                (Thread/sleep 2000)
                (println "[f] After sleep")
                2000)]
  (Thread/sleep 1000)
  (future-cancel f)
  (future-cancelled? f))
;; [f] Before sleep  ← printed message (second message is never printed)
;; true  ← returned value (due to future-cancelled?)
```

之前的场景发生是因为 Clojure 的 `future-cancel` 以一种方式取消未来（future），如果执行已经开始，可能会被中断，导致 `InterruptedException`，如果未显式捕获，则简单地终止代码块。注意来自未来中执行代码的异常，因为默认情况下，它们不会被详细报告！Clojure 的未来（futures）使用“solo”线程池（用于执行可能阻塞的操作），这是我们之前在讨论代理时提到的。

### 承诺

一个承诺（promise）是代表一个可能发生也可能不发生的计算结果的占位符。承诺并不直接与任何计算相关联。根据定义，承诺并不暗示计算何时发生，因此实现承诺。

通常，承诺起源于代码的一个地方，由知道何时以及如何实现承诺的代码的其他部分实现。这通常发生在多线程代码中。如果承诺尚未实现，任何尝试读取值的尝试都会阻塞所有调用者。如果承诺已经实现，那么所有调用者都可以读取值而不会被阻塞。与未来（futures）一样，可以使用 `deref` 在超时后读取承诺。

这里有一个非常简单的例子，展示了如何使用承诺：

```java
(def p (promise))
(realized? p)  ; returns false
@p  ; at this point, this will block until another thread delivers the promise
(deliver p :foo)
@p  ; returns :foo (for timeout use deref)
```

承诺是一个非常强大的工具，可以作为函数参数传递。它可以存储在引用类型中，或者简单地用于高级协调。

# Clojure 并行化与 JVM

我们在 第一章，*通过设计实现性能* 中观察到，并行化是硬件的函数，而并发是软件的函数，由硬件支持辅助。除了那些本质上纯粹顺序的算法外，并发是实现并行化和提高性能的首选方法。不可变和无状态数据是并发的催化剂，因为没有可变数据，线程之间没有竞争。

## 摩尔定律

在 1965 年，英特尔联合创始人戈登·摩尔观察到，集成电路每平方英寸的晶体管数量每 24 个月翻一番。他还预测这种趋势将持续 10 年，但实际上，它一直持续到现在，几乎半个世纪。更多的晶体管导致了更强的计算能力。在相同面积内晶体管数量增加，我们需要更高的时钟速度来传输信号到所有的晶体管。其次，晶体管需要变得更小以适应。大约在 2006-2007 年，电路能够工作的时钟速度达到了大约 2.8GHz，这是由于散热问题和物理定律的限制。然后，多核处理器应运而生。

## Amdahl 定律

多核处理器自然需要分割计算以实现并行化。从这里开始，出现了一个冲突——原本设计为顺序运行的程序无法利用多核处理器的并行化特性。程序必须被修改，以便在每一步找到分割计算的机会，同时考虑到协调成本。这导致了一个限制，即程序的速度不能超过其最长的顺序部分（*竞争*，或*串行性*），以及协调开销。这一特性被 Amdahl 定律所描述。

## 通用可扩展性定律

尼尔·冈瑟博士的通用可扩展性定律（USL）是 Amdahl 定律的超集，它将 *竞争（α）* 和 *一致性（β）* 作为量化可扩展性的首要关注点，使其非常接近现实中的并行系统。一致性意味着协调开销（延迟）在使并行化程序的一部分结果对另一部分可用时的协调。虽然 Amdahl 定律表明竞争（串行性）会导致性能水平化，但 USL 表明性能实际上随着过度并行化而下降。USL 用以下公式描述：

C(N) = N / (1 + α ((N – 1) + β N (N – 1)))

在这里，C(N) 表示相对容量或吞吐量，以并发源为依据，例如物理处理器，或驱动软件应用的用户。α 表示由于共享数据或顺序代码而引起的竞争程度，β 表示维护共享数据一致性所造成的惩罚。我鼓励您进一步研究 USL（[`www.perfdynamics.com/Manifesto/USLscalability.html`](http://www.perfdynamics.com/Manifesto/USLscalability.html)），因为这是研究并发对可扩展性和系统性能影响的重要资源。

## Clojure 对并行化的支持

依赖于变动的程序在创建对可变状态的竞争之前无法并行化其部分。它需要协调开销，这使情况变得更糟。Clojure 的不可变特性更适合并行化程序的部分。Clojure 还有一些结构，由于 Clojure 考虑了可用的硬件资源，因此适合并行化。结果是，操作针对某些用例场景进行了优化。

### pmap

`pmap` 函数（类似于 `map`）接受一个函数和一个或多个数据元素集合作为参数。该函数被应用于数据元素集合中的每个元素，这样一些元素可以并行地由该函数处理。并行度因子由 `pmap` 实现在运行时选择，通常大于可用的处理器总数。它仍然以惰性方式处理元素，但实现因子与并行度因子相同。

查看以下代码：

```java
(pmap (partial reduce +)
        [(range 1000000)
         (range 1000001 2000000)
         (range 2000001 3000000)])
```

要有效地使用 `pmap`，我们必须理解它的用途。正如文档所述，它是为计算密集型函数设计的。它针对 CPU 密集型和缓存密集型工作进行了优化。对于高延迟和低 CPU 任务，如阻塞 I/O，`pmap` 是一个不合适的选择。另一个需要注意的陷阱是 `pmap` 中使用的函数是否执行了大量的内存操作。由于相同的函数将在所有线程中应用，所有处理器（或核心）可能会竞争内存互连和子系统带宽。如果并行内存访问成为瓶颈，由于内存访问的竞争，`pmap` 无法真正实现操作的并行化。

另一个关注点是当多个 `pmap` 操作同时运行时会发生什么？Clojure 不会尝试检测同时运行多个 `pmap`。对于每个新的 `pmap` 操作，都会重新启动相同数量的线程。开发者负责确保并发 `pmap` 执行的性能特性和程序的响应时间。通常，当延迟原因是至关重要的，建议限制程序中运行的 `pmap` 并发实例。

### pcalls

`pcalls` 函数是使用 `pmap` 构建的，因此它借鉴了后者的属性。然而，`pcalls` 函数接受零个或多个函数作为参数，并并行执行它们，将调用结果作为列表返回。

### pvalues

`pvalues` 宏是使用 `pcalls` 构建的，因此它间接共享了 `pmap` 的属性。它的行为类似于 `pcalls`，但它接受零个或多个在并行中使用 `pmap` 评估的 S-表达式。

## Java 7 的 fork/join 框架

Java 7 引入了一个名为 "fork/join" 的新并行框架，该框架基于分而治之和工作窃取调度算法。使用 fork/join 框架的基本思路相当简单——如果工作足够小，则直接在同一线程中执行；否则，将工作分成两部分，在 fork/join 线程池中调用它们，并等待结果合并。

这样，工作会递归地分成更小的部分，例如倒置树，直到最小的部分可以仅用单个线程执行。当叶/子树工作返回时，父节点将所有子节点的结果合并，并返回结果。

Fork/Join 框架在 Java 7 中通过一种特殊的线程池实现；请查看 `java.util.concurrent.ForkJoinPool`。这个线程池的特殊之处在于它接受 `java.util.concurrent.ForkJoinTask` 类型的作业，并且每当这些作业阻塞，等待子作业完成时，等待作业使用的线程会被分配给子作业。当子作业完成其工作后，线程会被分配回阻塞的父作业以继续执行。这种动态线程分配的方式被称为“工作窃取”。Fork/Join 框架可以从 Clojure 内部使用。`ForkJoinTask` 接口有两个实现：`java.util.concurrent` 包中的 `RecursiveAction` 和 `RecursiveTask`。具体来说，`RecursiveTask` 在 Clojure 中可能更有用，因为 `RecursiveAction` 是设计用来处理可变数据的，并且其操作不会返回任何值。

使用 Fork/Join 框架意味着选择将作业拆分成批次的批大小，这是并行化长时间作业的一个关键因素。批大小过大可能无法充分利用所有 CPU 核心；另一方面，批大小过小可能会导致更长的开销，协调父/子批次。正如我们将在下一节中看到的，Clojure 与 Fork/join 框架集成以并行化减少器实现。

# 使用减少器实现并行化

减少器是 Clojure 1.5 中引入的新抽象，预计在未来版本中将对 Clojure 的其余实现产生更广泛的影响。它们描绘了在 Clojure 中处理集合的另一种思考方式——关键概念是打破集合只能顺序、惰性或产生序列处理的观念，以及更多。摆脱这种行为保证一方面提高了进行贪婪和并行操作的可能性，另一方面则带来了约束。减少器与现有集合兼容。

例如，对常规的 `map` 函数的敏锐观察揭示，其经典定义与产生结果机制（递归）、顺序（顺序）、惰性（通常）和表示（列表/序列/其他）方面相关联。实际上，这大部分定义了“如何”执行操作，而不是“需要做什么”。在 `map` 的情况下，“需要做什么”是关于对其集合参数的每个元素应用函数。但由于集合类型可以是各种类型（树状结构、序列、迭代器等），操作函数不知道如何遍历集合。减少器将操作的“需要做什么”和“如何做”部分解耦。

## 可约性、减少函数、减少转换

集合种类繁多，因此只有集合本身知道如何导航自己。在归约器模型的基本层面上，每个集合类型内部都有一个“reduce”操作，可以访问其属性和行为，以及它返回的内容。这使得所有集合类型本质上都是“可归约”的。所有与集合一起工作的操作都可以用内部“reduce”操作来建模。这种操作的新建模形式是一个“归约函数”，它通常有两个参数，第一个参数是累加器，第二个是新输入。

当我们需要在集合的元素上叠加多个函数时，它是如何工作的？例如，假设我们首先需要“过滤”、“映射”，然后“归约”。在这种情况下，使用“转换函数”来建模归约函数（例如，对于“过滤”），使其作为另一个归约函数（对于“映射”）出现，这样在转换过程中添加功能。这被称为“归约转换”。

## 实现可归约集合

虽然归约函数保留了抽象的纯度，但它们本身并不具有实用性。在名为`clojure.core.reducers`的命名空间中，归约操作与`map`、`filter`等类似，基本上返回一个包含归约函数的归约集合——这些归约函数嵌入在集合内部。一个可归约集合尚未实现，甚至不是懒实现——而只是一个准备实现的配方。为了实现一个可归约集合，我们必须使用`reduce`或`fold`操作之一。

实现可归约集合的`reduce`操作是严格顺序的，尽管与`clojure.core/reduce`相比，由于堆上对象分配的减少，性能有所提升。实现可归约集合的`fold`操作可能是并行的，并使用“reduce-combine”方法在 fork-join 框架上操作。与传统的“map-reduce”风格不同，使用 fork/join 的 reduce-combine 方法在底层进行归约，然后通过再次归约的方式结合。这使得`fold`实现更加节省资源，性能更优。

## 可折叠集合与并行性

通过`fold`进行的并行化对集合和操作施加了某些限制。基于树的集合类型（持久映射、持久向量和持久集合）适合并行化。同时，序列可能无法通过`fold`进行并行化。其次，`fold`要求单个归约函数应该是“结合律”，即应用于归约函数的输入参数的顺序不应影响结果。原因是，`fold`可以将集合的元素分割成可以并行处理的段，而这些元素可能组合的顺序事先是未知的。

`fold`函数接受一些额外的参数，例如“组合函数”，以及用于并行处理的分区批次大小（默认为 512）。选择最佳分区大小取决于工作负载、主机能力和性能基准测试。某些函数是可折叠的（即可以通过`fold`并行化），而另一些则不是，如下所示。它们位于`clojure.core.reducers`命名空间中：

+   **可折叠**：`map`、`mapcat`、`filter`、`remove`和`flatten`

+   **不可折叠**：`take-while`、`take`和`drop`

+   **组合函数**：`cat`、`foldcat`和`monoid`

Reducers 的一个显著特点是，只有在集合是树类型时，它才能在并行中折叠。这意味着在折叠它们时，整个数据集必须加载到堆内存中。这在系统高负载期间会有内存消耗的缺点。另一方面，对于这种情况，一个懒序列是一个完全合理的解决方案。在处理大量数据时，使用懒序列和 reducers 的组合来提高性能可能是有意义的。

# 摘要

并发和并行性在多核时代对性能至关重要。有效地使用并发需要深入了解其底层原理和细节。幸运的是，Clojure 提供了安全且优雅的方式来处理并发和状态。Clojure 的新特性“reducers”提供了一种实现细粒度并行性的方法。在未来的几年里，我们可能会看到越来越多的处理器核心，以及编写利用这些核心的代码的需求不断增加。Clojure 使我们处于应对这些挑战的正确位置。

在下一章中，我们将探讨性能测量、分析和监控。
