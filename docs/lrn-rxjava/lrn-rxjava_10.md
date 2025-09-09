# 第十章：测试和调试

虽然单元测试并不是确保您的代码正确工作的银弹，但追求它是良好的实践。这对于您的逻辑高度确定性和足够模块化以隔离的情况尤其正确。

初看起来，使用 RxJava 进行测试可能并不直接。毕竟，RxJava 声明的是行为而不是状态。那么我们如何测试行为是否正确工作，尤其是在大多数测试框架期望有状态结果的情况下呢？幸运的是，RxJava 提供了几个辅助测试的工具，您可以使用这些工具与您喜欢的测试框架一起使用。市场上有很多可以与 RxJava 一起工作的测试工具，但本章我们将使用 JUnit。

我们还将介绍一些调试 RxJava 程序的技巧。RxJava 的一个缺点是，当出现错误时，传统的调试方法并不总是有效的，尤其是因为堆栈跟踪并不总是有帮助，断点也不容易应用。但是，RxJava 在调试方面提供了一个好处：通过正确的方法，您可以遍历整个响应式链，找到导致问题发生的算子。问题变得非常线性，变成隔离坏链的问题。这可以显著简化调试过程。

本章将涵盖许多测试功能，因此我们将从简单的直观方法开始，以涵盖基本的阻塞算子。然后，我们将升级到更健壮的工具，例如 `TestObserver`、`TestSubscriber` 和 `TestScheduler`，您可能会在您的应用程序中使用这些工具。

在本章中，我们将涵盖以下主题：

+   `blockingSubscribe()`

+   阻塞算子

+   `TestObserver` 和 `TestSubscriber`

+   `TestScheduler`

+   RxJava 调试策略

# 配置 JUnit

在本节中，我们将使用 JUnit 作为我们的测试框架。请将以下依赖项添加到您的 Maven 或 Gradle 项目中。

这里是 Maven 的配置：

```java
<dependency>
     <groupId>junit</groupId>
     <artifactId>junit</artifactId>
     <version>4.12</version>
 </dependency>
```

这里是 Gradle 的配置：

```java
dependencies { 
     compile 'junit:junit:4.12'
}
```

为了节省您的时间，组织您的代码项目以符合 Maven 标准目录布局。您可能希望将测试类放在 `/src/test/java/` 文件夹中，这样 Maven 和 Gradle 将自动将其识别为测试代码文件夹。您还应该在项目的 `/src/main/java/` 文件夹中放置您的生产代码。您可以在 [`maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html`](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html) 上了解更多关于 Maven 标准目录布局的信息。

# 阻塞订阅者

记得有时候我们不得不阻止主线程从不同的线程上跳过`Observable`或`Flowable`操作，并防止它在有机会触发之前退出应用程序吗？我们经常使用`Thread.sleep()`来防止这种情况，尤其是在我们使用`Observable.interval()`、`subscribeOn()`或`observeOn()`时。以下代码展示了我们通常是如何做到这一点的，并使一个`Observable.interval()`应用程序保持活跃五秒钟：

```java
 import io.reactivex.Observable;
 import java.util.concurrent.TimeUnit;

 public class Launcher {

     public static void main(String[] args) {
         Observable.interval(1, TimeUnit.SECONDS)
                 .take(5)
                 .subscribe(System.out::println);

         sleep(5000);
     }

     public static void sleep(int millis) {
         try {
             Thread.sleep(millis);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
     }
 }
```

当涉及到单元测试时，单元测试通常必须在开始下一个测试之前完成。当我们有一个在另一个线程上发生的`Observable`或`Flowable`操作时，这可能会变得相当混乱。当一个`test`方法声明了一个异步的`Observable`或`Flowable`链操作时，我们需要阻塞并等待该操作完成。

在这里，我们创建了一个测试来确保从`Observable.interval()`发出五个发射，并在验证它被增加了五次之前增加`AtomicInteger`：

```java
 import io.reactivex.Observable;
 import org.junit.Test;
 import java.util.concurrent.TimeUnit;
 import java.util.concurrent.atomic.AtomicInteger;
 import static org.junit.Assert.assertTrue;

 public class RxTest {

     @Test
     public void testBlockingSubscribe() {

         AtomicInteger hitCount = new AtomicInteger();

         Observable<Long> source = Observable.interval(1, TimeUnit.SECONDS)
                     .take(5);

         source.subscribe(i -> hitCount.incrementAndGet());

         assertTrue(hitCount.get() == 5);
     }
 }
```

我们使用`@Test`注解来告诉`JUnit`这是一个测试方法。您可以通过在 IntelliJ IDEA 的 gutter 中点击其绿色的三角形*播放*按钮或在 Gradle 或 Maven 中运行测试任务来运行它。

然而，存在一个问题。当你运行这个测试时，断言失败。`Observable.interval()`正在计算线程上运行，而主线程在五个发射触发之前就冲过去了。主线程在五个发射触发之前执行`assertTrue()`，因此发现`hitCount`是`0`而不是`5`。我们需要停止主线程，直到`subscribe()`完成并调用`onComplete()`。

幸运的是，我们不需要使用同步器和其他原生 Java 并发工具来发挥创意。相反，我们可以使用`blockingSubscribe()`，它将阻塞声明的主线程，直到`onComplete()`（或`onError()`）被调用。一旦收集到这五个发射，主线程就可以继续进行，并成功执行断言，就像这里所展示的。然后测试应该通过：

```java
 import io.reactivex.Observable;
 import org.junit.Test;
 import java.util.concurrent.TimeUnit;
 import java.util.concurrent.atomic.AtomicInteger;
 import static org.junit.Assert.assertTrue;

 public class RxTest {

     @Test
     public void testBlockingSubscribe() {

         AtomicInteger hitCount = new AtomicInteger();

         Observable<Long> source = Observable.interval(1, TimeUnit.SECONDS)
                     .take(5);

         source.blockingSubscribe(i -> hitCount.incrementAndGet());

         assertTrue(hitCount.get() == 5);
     }
 }
```

正如我们将在本章中看到的，除了`blockingSubscribe()`之外，还有更好的测试方法。但`blockingSubscribe()`是一个快速而有效的方法来停止声明线程，并在继续之前等待`Observable`或`Flowable`完成，即使它是在不同的线程上。只需确保源在某一点终止，否则测试将永远不会完成。

在测试之外以及在生产环境中使用`blockingSubscribe()`时，要谨慎。确实有在接口非响应式 API 时它是合法解决方案的时候。例如，在生产环境中使用它来无限期地保持应用程序活跃，并且是使用`Thread.sleep()`的有效替代方案。只是要小心确保 RxJava 的异步优势不被削弱。

# 阻塞算子

在 RxJava 中，有一组我们尚未介绍的称为**阻塞操作符**的操作符。这些操作符充当反应式世界和有状态世界之间的即时代理，阻塞并等待结果发出，但以非反应式的方式返回。即使反应式操作在不同的线程上运行，阻塞操作符也会停止声明线程并使其以同步方式等待结果，就像`blockingSubscribe()`一样。

阻塞操作符在使`Observable`或`Flowable`的结果易于评估方面特别有用。然而，你希望在生产环境中避免使用它们，因为它们会鼓励反模式并损害响应式编程的好处。对于测试，你仍然希望优先考虑`TestObserver`和`TestSubscriber`，我们将在后面介绍。但如果你确实需要它们，这里有一些阻塞操作符。

# blockingFirst()

`blockingFirst()`操作符将停止调用线程并使其等待第一个值被发射并返回（即使链在具有`observeOn()`和`subscribeOn()`的不同线程上运行）。假设我们想要测试一个`Observable`链，该链仅过滤长度为四的字符串发射序列。如果我们想要断言第一个通过此操作的发射是`Beta`，我们可以这样测试：

```java
 import io.reactivex.Observable;
 import org.junit.Test;
 import static org.junit.Assert.assertTrue;

 public class RxTest {

     @Test
     public void testFirst() {
         Observable<String> source =
                 Observable.just("Alpha", "Beta", "Gamma", "Delta", "Zeta");

         String firstWithLengthFour = source.filter(s -> s.length() == 4)
                 .blockingFirst();

         assertTrue(firstWithLengthFour.equals("Beta"));
     }
 }
```

在这里，我们的单元测试称为`testFirst()`，它将断言第一个长度为四的字符串是`Beta`。请注意，我们不是使用`subscribe()`或`blockingSubscribe()`来接收发射，而是使用`blockingFirst()`，它将以非反应式的方式返回第一个发射。换句话说，它返回一个简单的字符串，而不是发出字符串的`Observable`。

这将阻塞声明线程，直到返回值并分配给`firstWithLengthFour`。然后我们使用这个保存的值来断言它实际上是`Beta`。

看到 blockingFirst()，你可能想在生产代码中使用它来保存结果状态并稍后引用它。尽量不要这样做！虽然在某些情况下你可能会找到合理的理由（例如，将发射保存到`HashMap`中进行昂贵的计算和查找），但阻塞操作符很容易被滥用。如果你需要持久化值，尽量使用`replay()`和其他反应式缓存策略，这样你就可以轻松地更改其行为和并发策略。阻塞通常会使得你的代码更缺乏灵活性，并损害 Rx 的好处。

注意，如果没有任何发射通过，`blockingFirst()`操作符将抛出错误并使测试失败。但是，你可以提供一个默认值作为`blockingFirst()`的重载，这样它总是有一个回退值。

与`blockingFirst()`类似的阻塞操作符是`blockingSingle()`，它期望只发射一个项目，但如果有多于一个项目，则抛出错误。

# blockingGet()

`Maybe` 和 `Single` 没有提供 `blockingFirst()`，因为最多只能有一个元素。从逻辑上讲，对于 `Single` 和 `Maybe`，这并不是确切的第一元素，而是唯一的元素，所以等效的算子是 `blockingGet()`。

在这里，我们断言长度为四的所有项只包含 `Beta` 和 `Zeta`，并且我们使用 `toList()` 收集它们，这会产生一个 `Single<List<String>>`。我们可以使用 `blockingGet()` 等待这个列表，并断言它等于我们期望的结果：

```java
 import io.reactivex.Observable;
 import org.junit.Test;
 import java.util.Arrays;
 import java.util.List;
 import static org.junit.Assert.assertTrue;

 public class RxTest {

     @Test
     public void testSingle() {
         Observable<String> source =
                 Observable.just("Alpha", "Beta", "Gamma", "Delta", "Zeta");

         List<String> allWithLengthFour = source.filter(s -> s.length() == 4)
                 .toList()
                 .blockingGet();

         assertTrue(allWithLengthFour.equals(Arrays.asList("Beta","Zeta")));
     }
 }
```

# blockingLast()

如果有 `blockingFirst()`，那么只有 `blockingLast()` 才有意义。这将阻塞并返回从 `Observable` 或 `Flowable` 操作中发出的最后一个值。当然，它不会返回任何内容，直到 `onComplete()` 被调用，所以这是你想要避免与无限源一起使用的情况。

在这里，我们断言从我们的操作中发出的最后一个四字符字符串是 `Zeta`：

```java
 import io.reactivex.Observable;
 import org.junit.Test;
 import static org.junit.Assert.assertTrue;

 public class RxTest {

     @Test
     public void testLast() {
         Observable<String> source =
                 Observable.just("Alpha", "Beta", "Gamma", "Delta", "Zeta");

         String lastWithLengthFour = source.filter(s -> s.length() == 4)
                 .blockingLast();

         assertTrue(lastWithLengthFour.equals("Zeta"));
     }
 }
```

就像 `blockingFirst()` 一样，如果没有任何排放发生，`blockingLast()` 将会抛出一个错误，但你可以为默认值指定一个重载。

# blockingIterable()

最有趣的阻塞算子之一是 `blockingIterable()`。与之前的示例不同，它不会返回单个排放，而是会通过 `iterable<T>` 提供排放。`Iterable<T>` 提供的 `Iterator<T>` 将会阻塞迭代线程，直到下一个排放可用，迭代将在 `onComplete()` 被调用时结束。在这里，我们遍历每个返回的字符串值，以确保其实际长度是 `5`：

```java
 import io.reactivex.Observable;
 import org.junit.Test;
 import static org.junit.Assert.assertTrue;

 public class RxTest {

     @Test
     public void testIterable() {
         Observable<String> source =
                 Observable.just("Alpha", "Beta", "Gamma", "Delta", "Zeta");

         Iterable<String> allWithLengthFive = source.filter(s -> s.length() == 5)
                 .blockingIterable();

         for (String s: allWithLengthFive) {
             assertTrue(s.length() == 5);
         }
     }
 }
```

`blockingIterable()` 将会排队等待未消费的值，直到 `Iterator` 能够处理它们。在没有背压的情况下，这可能会出现问题，因为你可能会遇到 `OutOfMemoryException` 错误。

与 C# 不同，请注意 Java 的 for-each 构造不会处理取消、中断或处置。你可以通过在 `try-finally` 中迭代可迭代的 `Iterator` 来解决这个问题。在 `finally` 块中，将 `Iterator` 强制转换为 `disposable`，这样你就可以调用它的 `dispose()` 方法。

`blockingIterable()` 可以帮助快速将 `Observable` 或 `Flowable` 转换为拉取驱动的函数序列类型，例如 Java 8 Stream 或 Kotlin 序列，这些可以基于可迭代对象构建。然而，对于 Java 8 streams，你可能会更倾向于使用 David Karnok 的 RxJava2Jdk8Interop 库 ([`github.com/akarnokd/RxJava2Jdk8Interop`](https://github.com/akarnokd/RxJava2Jdk8Interop))，这样终止处理会更加安全。

# blockingForEach()

我们可以以更流畅的方式为每个任务执行阻塞，即使用`blockingForEach()`操作符而不是`blockingIterable()`。这将阻塞声明线程，等待每个发射被处理后再允许线程继续。我们可以简化之前的示例，其中我们迭代每个发射的字符串并确保其长度为五，并将断言作为 lambda 表达式指定在`forEach()`操作符中：

```java
 import io.reactivex.Observable;
 import org.junit.Test;
 import static org.junit.Assert.assertTrue;

 public class RxTest {

     @Test
     public void testBlockingForEach() {
         Observable<String> source =
                 Observable.just("Alpha", "Beta", "Gamma", "Delta", "Zeta");

         source.filter(s -> s.length() == 5)
                 .blockingForEach(s -> assertTrue(s.length() == 5));
     }
 }
```

`blockingForEach()`的一个变体是`blockingForEachWhile()`，它接受一个谓词，如果谓词对发射评估为假，则优雅地终止序列。如果所有发射都不将被消费，并且你希望优雅地终止，这可能是有用的。

# `blockingNext()`

`blockingNext()`将返回一个可迭代对象，并阻塞每个迭代器的`next()`请求，直到下一个值被提供。在最后一个满足的`next()`请求之后和当前`next()`之前发生的发射将被忽略。在这里，我们有一个每微秒（千分之一毫秒）发射的源。请注意，`blockingNext()`返回的可迭代对象忽略了它之前错过的值：

```java
 import io.reactivex.Observable;
 import org.junit.Test;
 import java.util.concurrent.TimeUnit;

 public class RxTest {

     @Test
     public void testBlockingNext() {
         Observable<Long> source =
                 Observable.interval(1, TimeUnit.MICROSECONDS)
                 .take(1000);

         Iterable<Long> iterable = source.blockingNext();

         for (Long i: iterable) {
             System.out.println(i);
         }
     }
 }
```

输出如下：

```java
0
6
9
11
17
23
26

```

# `blockingLatest()`

与`blockingLatest()`相反，`blockingLatest()`的可迭代对象不会等待下一个值，而是请求最后一个发射的值。在此之前的任何未捕获的值都将被遗忘。如果迭代器的`next()`之前已经消费了最新的值，它将不会重新消费，并且会阻塞，直到下一个值到来：

```java
 import io.reactivex.Observable;
 import org.junit.Test;
 import java.util.concurrent.TimeUnit;

 public class RxTest {

     @Test
     public void testBlockingLatest() {
         Observable<Long> source =
                 Observable.interval(1, TimeUnit.MICROSECONDS)
                 .take(1000);

         Iterable<Long> iterable = source.blockingLatest();

         for (Long i: iterable) {
             System.out.println(i);
         }
     }
 }
```

输出如下：

```java
0
49
51
53
55
56
58
...
```

# `blockingMostRecent()`

`blockingMostRecent()`与`blockingLatest()`类似，但它会在迭代器的每个`next()`调用中重复消费最新的值，即使它已经被消费过。它还需要一个`defaultValue`参数，以便在没有发射值时返回某个值。在这里，我们使用`blockingMostRecent()`对每 10 毫秒发射一次的`Observable`进行操作。默认值是`-1`，并且它会重复消费每个值，直到下一个值被提供：

```java
 import io.reactivex.Observable;
 import org.junit.Test;
 import java.util.concurrent.TimeUnit;

 public class RxTest {

     @Test
     public void testBlockingMostRecent() {
         Observable<Long> source =
                 Observable.interval(10, TimeUnit.MILLISECONDS)
                 .take(5);

         Iterable<Long> iterable = source.blockingMostRecent(-1L);

         for (Long i: iterable) {
             System.out.println(i);
         }
     }
 }
```

输出如下：

```java
-1
-1
-1
...
0
0
0
...
1
1
1
...
```

在我们完成对阻塞操作符的介绍后，应该再次强调，它们可以是一种有效的简单断言方式，并提供阻塞以获取结果的方法，以便它们可以轻松地被测试框架消费。然而，你应尽可能避免在生产中使用阻塞操作符。尽量不要屈服于便利的诱惑，因为你会发现它们可以迅速削弱响应式编程的灵活性和优势。

# 使用 TestObserver 和 TestSubscriber

到目前为止，我们已经在本章中介绍了 `blockingSubscribe()` 和几个阻塞操作符。虽然你可以使用这些阻塞工具来进行简单的断言，但测试反应式代码的更全面的方法不仅仅是阻塞一个或多个值。毕竟，我们应该做的不仅仅是测试 `onNext()` 调用。我们还有 `onComplete()` 和 `onError()` 事件需要考虑！此外，简化测试其他 RxJava 事件，如订阅、处置和取消，也会很棒。

所以，让我们来介绍 `TestObserver` 和 `TestSubscriber`，它们是你在测试 RxJava 应用程序时的两位最佳伙伴。

`TestObserver` 和 `TestSubscriber` 是一个方便的测试方法宝库，其中许多方法断言某些事件已经发生或接收到了特定的值。还有一些阻塞方法，例如 `awaitTerminalEvent()`，它将停止调用线程，直到反应式操作终止。

`TestObserver` 用于 `Observable`、`Single`、`Maybe` 和 `Completable` 源，而 `TestSubscriber` 用于 `Flowable` 源。以下是一个单元测试示例，展示了几个 `TestObserver` 方法，这些方法也存在于 `TestSubscriber` 上，如果你正在处理 `Flowables`。这些方法执行诸如断言某些事件是否发生（或未发生）、等待终止或断言接收到了特定值等任务：

```java
 import io.reactivex.Observable;
 import io.reactivex.observers.TestObserver;
 import org.junit.Test;
 import java.util.concurrent.TimeUnit;

 public class RxTest {

     @Test
     public void usingTestObserver() {

         *//An Observable with 5 one-second emissions*
         Observable<Long> source = Observable.interval(1, TimeUnit.SECONDS)
                 .take(5);

         *//Declare TestObserver*
         TestObserver<Long> testObserver = new TestObserver<>();

         *//Assert no subscription has occurred yet*
         testObserver.assertNotSubscribed();

         *//Subscribe TestObserver to source*
         source.subscribe(testObserver);

         *//Assert TestObserver is subscribed*
         testObserver.assertSubscribed();

         *//Block and wait for Observable to terminate*
         testObserver.awaitTerminalEvent();

         *//Assert TestObserver called onComplete()*
         testObserver.assertComplete();

         *//Assert there were no errors*
         testObserver.assertNoErrors();

         *//Assert 5 values were received*
         testObserver.assertValueCount(5);

         *//Assert the received emissions were 0, 1, 2, 3, 4*
         testObserver.assertValues(0L, 1L, 2L, 3L, 4L);
     }
 }
```

这只是众多测试方法中的一小部分，它们将使你的单元测试更加全面和流畅。大多数 `TestObserver` 方法返回 `TestObserver`，因此你可以流畅地链式调用这些断言（这也适用于 `TestSubscriber`）。

注意，`awaitTerminalEvent()` 操作符可以接受一个超时参数，如果在指定时间之前源没有完成，它将抛出一个错误。

花些时间熟悉所有这些测试方法，以便你了解你做出的不同断言。尽可能优先使用 `TestObserver` 和 `TestSubscriber` 而不是阻塞操作符。这样，你可以花更少的时间维护测试，并确保你覆盖了 `Observable` 或 `Flowable` 操作的生命周期中的所有事件范围。

`TestObserver` 实现了 `Observer`、`MaybeObserver`、`SingleObserver` 和 `CompetableObserver`，以支持所有这些反应式类型。如果你正在测试一个长时间运行的异步源，你可能想使用 `awaitCount()` 来等待至少一定数量的发射以进行断言，而不是等待 `onComplete()` 调用。

# 使用 TestScheduler 操作时间

在我们之前的例子中，您注意到测试一个时间驱动的 `Observable` 或 `Flowable` 需要时间流逝到测试完成吗？在上一个练习中，我们从每秒发射一次的 `Observable.interval()` 中获取了五次发射，因此该测试花费了 5 秒来完成。如果我们有很多处理时间驱动源的单元测试，测试完成可能需要很长时间。如果我们可以模拟时间流逝而不是实际体验它们，那不是很好吗？

`TestScheduler` 正好能做这件事。它是一个调度器实现，允许我们通过特定的已过时间量进行快进，并且我们可以在每次快进后进行任何断言，以查看发生了哪些事件。

在这里，我们创建了一个针对每分钟发射一次的 `Observable.interval()` 的测试，并最终断言在 90 分钟后发生了 90 次发射。我们不必在真实时间中等待整整 90 分钟，而是使用 `TestObserver` 人工地流逝这 90 分钟。这使得测试可以立即运行：

```java
 import io.reactivex.Observable;
 import io.reactivex.observers.TestObserver;
 import io.reactivex.schedulers.TestScheduler;
 import org.junit.Test;

 import java.util.concurrent.TimeUnit;

 public class RxTest {

     @Test
     public void usingTestScheduler() {

         *//Declare TestScheduler*
         TestScheduler testScheduler = new TestScheduler();

         *//Declare TestObserver*
         TestObserver<Long> testObserver = new TestObserver<>();

         *//Declare Observable emitting every 1 minute*
         Observable<Long> minuteTicker =
                 Observable.interval(1, TimeUnit.MINUTES, testScheduler);

         *//Subscribe to TestObserver*
         minuteTicker.subscribe(testObserver);

         *//Fast forward by 30 seconds*
         testScheduler.advanceTimeBy(30, TimeUnit.SECONDS);

         *//Assert no emissions have occurred yet*
         testObserver.assertValueCount(0);

         *//Fast forward to 70 seconds after subscription*
         testScheduler.advanceTimeTo(70, TimeUnit.SECONDS);

         *//Assert the first emission has occurred*
         testObserver.assertValueCount(1);

         *//Fast Forward to 90 minutes after subscription*
         testScheduler.advanceTimeTo(90, TimeUnit.MINUTES);

         *//Assert 90 emissions have occurred*
         testObserver.assertValueCount(90);
     }
 }
```

很酷，对吧？这几乎就像是时间旅行！我们将 `Observable.interval()` 放在 `TestScheduler` 上。这样，`TestScheduler` 控制了 `Observable` 如何解释时间并推动发射。我们使用 `advanceTimeBy()` 快进 30 秒，然后断言还没有发生任何发射。然后我们使用 `advanceTimeTo()` 跳转到订阅发生后的 70 秒，并断言确实发生了一次发射。最后，我们在订阅后快进 90 分钟，并断言确实发生了 90 次发射。

所有这些都在瞬间完成，而不是花费 90 分钟，这表明确实可以在不实际流逝该时间的情况下测试时间驱动的 `Observable`/`Flowable` 操作。请注意，`advanceTimeBy()` 将相对于当前时间快进指定的时间间隔，而 `advanceTimeTo()` 将跳转到自订阅发生以来的确切流逝时间。

总结来说，当您需要虚拟表示时间流逝时，请使用 `TestScheduler`，但请注意，它不是一个线程安全的调度器，不应与实际并发一起使用。一个常见的陷阱是使用许多操作符和调度器的复杂流程，这些调度器不易配置为使用 `TestScheduler`。在这种情况下，您可以使用 `RxJavaPlugins.setComputationScheduler()` 和类似方法来覆盖标准调度器，并在其中注入 `TestScheduler`。

在 `TestScheduler` 中还有两个其他方法需要注意。`now()` 将返回在您指定的单位中虚拟流逝了多少时间。`triggerActions()` 方法将启动任何计划触发但尚未虚拟流逝的动作。

# 调试 RxJava 代码

RxJava 初看起来并不容易调试，主要是因为缺乏调试工具和它可能产生的大量堆栈跟踪。有创建针对 RxJava 的有效调试工具的努力，最著名的是 Android 的 Frodo 库 ([`github.com/android10/frodo`](https://github.com/android10/frodo))。我们不会涵盖任何 RxJava 的调试工具，因为还没有标准化，但我们将了解一种有效的调试反应式代码的方法。

在调试 RxJava 操作中，一个常见的主题是找到导致问题的 `Observable`/`Flowable` 链中的坏链接或操作符。无论是否正在发射错误，`onComplete()` 永远没有被调用，或者 `Observable` 突然为空，你通常必须从链的起始处，即源头开始，然后验证每个下游步骤，直到找到不正常工作的那个。

嘿，我们有一个 `Observable` 正在推送包含数字和由斜杠 "`/`" 分隔的字母词的五个字符串。我们想要在斜杠 "`/`" 上分割这些字符串，只过滤字母词，并将它们捕获在 `TestObserver` 中。然而，运行这个操作，你会看到这个测试失败了：

```java
 import io.reactivex.observers.TestObserver;
 import org.junit.Test;
 import io.reactivex.Observable;

 public class RxTest {

     @Test
     public void debugWalkthrough() {

         *//Declare TestObserver*
         TestObserver<String> testObserver = new TestObserver<>();

         *//Source pushing three strings*
         Observable<String> items =
                 Observable.just("521934/2342/Foxtrot",
                         "Bravo/12112/78886/Tango",
                         "283242/4542/Whiskey/2348562");

         *//Split and concatMap() on "/"*
         items.concatMap(s ->
                 Observable.fromArray(s.split("/"))
         )
          *//filter for only alphabetic Strings using regex*
          .filter(s -> s.matches("[A-Z]+"))

          *//Subscribe the TestObserver*
          .subscribe(testObserver);

         *//Why are no values being emitted?*
         System.out.println(testObserver.values());

         *//This fails due to no values*
         testObserver.assertValues("Foxtrot","Bravo","Tango","Whiskey");
     }
 }
```

输出结果如下：

```java
[]

java.lang.AssertionError: Value count differs; Expected: 4 [Foxtrot, Bravo, Tango, Whiskey],
    Actual: 0 [] (latch = 0, values = 0, errors = 0, completions = 1)

    at io.reactivex.observers.BaseTestConsumer.fail(BaseTestConsumer.java:163)
    at io.reactivex.observers.BaseTestConsumer.assertValues(BaseTestConsumer.java:485)
    at RxTest.debugWalkthrough(RxTest.java:32)
...
```

那么到底出了什么问题？我们如何调试这个失败的测试？嗯，记住 RxJava 操作是一个管道。正确的发射项应该流动并通过，到达 `Observer`。但是没有收到任何发射项。让我们穿上我们的管道工装备，找出管道中的堵塞在哪里。我们将从源头开始。

在源头之后和 `concatMap()` 之前立即放置 `doOnNext()`，并打印每个发射项。这让我们能够看到从源头 `Observable` 中出来的内容。如图所示，我们应该看到所有来自源头的发射项被打印出来，这表明没有发射项被遗漏，并且源头上游运行正常：

```java
*//Split and concatMap() on "/"*
 items.doOnNext(s -> System.out.println("Source pushed: " + s))
         .concatMap(s ->
                 Observable.fromArray(s.split("/"))
         )
```

输出结果如下：

```java
Source pushed: 521934/2342/Foxtrot
Source pushed: Bravo/12112/78886/Tango
Source pushed: 283242/4542/Whiskey/2348562
[]

java.lang.AssertionError: Value count differs; Expected ...
```

让我们继续向下流，接下来看看 `concatMap()`。也许它遗漏了发射项，所以让我们检查一下。在 `concatMap()` 之后放置 `doOnNext()` 并打印每个发射项，以查看是否所有这些都能通过，如图所示：

```java
*//Split and concatMap() on "/"*
 items.concatMap(s ->
                 Observable.fromArray(s.split("/"))
         )
  .doOnNext(s -> System.out.println("concatMap() pushed: " + s))
```

输出结果如下：

```java
concatMap() pushed: 521934
concatMap() pushed: 2342
concatMap() pushed: Foxtrot
concatMap() pushed: Bravo
concatMap() pushed: 12112
concatMap() pushed: 78886
concatMap() pushed: Tango
concatMap() pushed: 283242
concatMap() pushed: 4542
concatMap() pushed: Whiskey
concatMap() pushed: 2348562
[]

java.lang.AssertionError: Value count differs; Expected ...
```

好的，所以 `concatMap()` 运行正常，所有的发射项都通过了。所以 `concatMap()` 内部的分割操作没有问题。让我们继续向下流，在 `filter()` 之后放置 `doOnNext()`。如图所示，打印每个发射项以查看我们想要的项是否从 `filter()` 中出来：

```java
*//filter for only alphabetic Strings using regex*
 .filter(s -> s.matches("[A-Z]+"))
 .doOnNext(s -> System.out.println("filter() pushed: " + s))
```

输出结果如下：

```java
[]

java.lang.AssertionError: Value count differs; Expected ...
```

哎！在`filter()`之后没有打印出任何排放项，这意味着没有任何东西通过它。`filter()`是导致问题的操作符。我们原本打算过滤掉数字字符串，只发出字母词。但不知何故，所有排放项都被过滤掉了。如果你对正则表达式有所了解，请注意我们只对完全大写的字符串进行限定。实际上，我们还需要对小写字母进行限定，所以这里是我们需要的修正：

```java
*//filter for only alphabetic Strings using regex*
 .filter(s -> s.matches("[A-Za-z]+"))
 .doOnNext(s -> System.out.println("filter() pushed: " + s))
```

输出结果如下：

```java
filter() pushed: Foxtrot
filter() pushed: Bravo
filter() pushed: Tango
filter() pushed: Whiskey
[Foxtrot, Bravo, Tango, Whiskey]
```

好的，它已经修复了！我们的单元测试最终通过了，下面是它的全部内容。现在问题已经解决，我们完成了调试，我们可以移除`doOnNext()`和任何打印调用：

```java
 import io.reactivex.observers.TestObserver;
 import org.junit.Test;
 import io.reactivex.Observable;

 public class RxTest {

     @Test
     public void debugWalkthrough() {

         *//Declare TestObserver*
         TestObserver<String> testObserver = new TestObserver<>();

         *//Source pushing three strings*
         Observable<String> items =
                 Observable.just("521934/2342/Foxtrot",
                         "Bravo/12112/78886/Tango",
                         "283242/4542/Whiskey/2348562");

         *//Split and concatMap() on "/"*
         items.concatMap(s ->
                         Observable.fromArray(s.split("/"))
          )
          *//filter for only alphabetic Strings using regex*
          .filter(s -> s.matches("[A-Za-z]+"))

          *//Subscribe the TestObserver*
          .subscribe(testObserver);

         *//This succeeds*
         testObserver.assertValues("Foxtrot","Bravo","Tango","Whiskey");
     }
 }
```

输出结果如下：

```java
[Foxtrot, Bravo, Tango, Whiskey]
```

总结来说，当你有一个正在发出错误、错误项或没有任何项的`Observable`或`Flowable`操作时，从源头开始，逐步向下工作，直到找到导致问题的操作符。你还可以在每个步骤放置`TestObserver`以获取该操作中发生的更全面的报告，但使用`doOnNext()`、`doOnError()`、`doOnComplete()`、`doOnSubscribe()`等操作符是快速且简单的方法，可以深入了解管道该部分正在发生的事情。

可能不是最优的做法是必须使用`doXXX()`操作符修改代码来调试它。如果你使用 Intellij IDEA，你可以尝试在 lambda 中设置断点，尽管我使用这种方法只有混合的成功率。你还可以研究 RxJava 调试库，以获取不修改代码的详细日志。希望随着 RxJava 继续获得关注，将出现更多有用的调试工具，并成为标准。

# 摘要

在本章中，你学习了如何测试和调试 RxJava 代码。当你创建一个基于 RxJava 的应用程序或 API 时，你可能想要围绕它构建单元测试，以确保始终执行健全性检查。你可以使用阻塞操作符来帮助执行断言，但`TestObserver`和`TestSubscriber`将为你提供更全面和流畅的测试体验。你还可以使用`TestScheduler`来模拟时间流逝，以便可以立即测试基于时间的 Observables。最后，我们介绍了 RxJava 的调试策略，这通常涉及找到*损坏的操作符*，从源头开始，向下移动直到找到。

本章结束了我们对 RxJava 库的探索之旅，所以如果你已经到达这里，恭喜你！你现在有了构建响应式 Java 应用程序的坚实基础。在最后两章中，我们将涵盖 RxJava 在两个特定领域中的应用：Android 和 Kotlin。
