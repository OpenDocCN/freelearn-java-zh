# 第八章：Flowables 和 背压

在上一章中，我们学习了不同的操作符，它们可以拦截快速发射的发射，并合并或省略它们以减少传递到下游的发射。但对于大多数情况下，如果源头产生的发射比下游处理得快，最好首先让源头减速，并以与下游操作一致的速度发射。这被称为背压或流控制，可以通过使用 `Flowable` 而不是 `Observable` 来启用。这将是本章我们将与之工作的核心类型，我们将学习在应用程序中何时利用它。本章我们将涵盖以下主题：

+   理解背压

+   `Flowable` 和 `Subscriber`

+   使用 `Flowable.create()`

+   Observables 和 Flowables 的互操作性

+   背压操作符

+   使用 `Flowable.generate()`

# 理解背压

在整本书中，我强调了 Observables 的“基于推送”的特性。从源头同步且逐个推送项目到 `Observer` 确实是 `Observable` 链默认的工作方式，没有任何并发。

例如，以下是一个将发出从 1 到 999,999,999 的数字的 `Observable`。它将每个整数映射到一个 `MyItem` 实例，该实例简单地将其作为属性持有。但让我们在 `Observer` 中将每个发射的处理速度减慢 50 毫秒。这表明即使下游正在缓慢处理每个发射，上游也会同步地跟上它。这是因为只有一个线程在做所有的工作：

```java
 import io.reactivex.Observable;

 public class Launcher {

     public static void main(String[] args) {

         Observable.range(1, 999_999_999)
                 .map(MyItem::new)
                 .subscribe(myItem -> {
                     sleep(50);
                     System.out.println("Received MyItem " + myItem.id);
                 });
     }

     static void sleep(long milliseconds) {
         try {
             Thread.sleep(milliseconds);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
     }

     static final class MyItem {

         final int id;

         MyItem(int id) {
             this.id = id;
             System.out.println("Constructing MyItem " + id);
         }
     }
 }
```

输出如下：

```java
 Constructing MyItem 1
 Received MyItem 1
 Constructing MyItem 2
 Received MyItem 2
 Constructing MyItem 3
 Received MyItem 3
 Constructing MyItem 4
 Received MyItem 4
 Constructing MyItem 5
 Received MyItem 5
 Constructing MyItem 6
 Received MyItem 6
 Constructing MyItem 7
 Received MyItem 7
 ...
```

输出的 `Constructing MyItem` 和 `Received MyItem` 之间的交替表明每个发射都是从源头逐个处理到终端 `Observer` 的。这是因为只有一个线程为整个操作做所有的工作，使得一切同步。消费者和生产者以序列化、一致的方式传递发射。

# 需要背压的示例

当你向 `Observable` 链（尤其是 `observeOn()`、并行化以及如 `delay()` 这样的操作符）添加并发操作时，操作变为 **异步**。这意味着在给定时间内，`Observable` 链的多个部分可以处理发射，生产者可以超过消费者，因为它们现在在不同的线程上操作。发射不再严格地从源头逐个传递到 `Observer`，然后再开始下一个。这是因为一旦发射通过 `observeOn()`（或其他并发操作符）击中不同的 `Scheduler`，源头就不再负责将那个发射推送到 `Observer`。因此，源头将开始推送下一个发射，即使前一个发射可能还没有到达 `Observer`。

如果我们将之前的例子添加 `observeOn(Shedulers.io())` 在 `subscribe()` 之前（如下面的代码所示），你会注意到一个非常明显的事实：

```java
 import io.reactivex.Observable;
 import io.reactivex.schedulers.Schedulers;

 public class Launcher {

     public static void main(String[] args) {

         Observable.range(1, 999_999_999)
                 .map(MyItem::new)
                 .observeOn(Schedulers.io())
                 .subscribe(myItem -> {
                     sleep(50);
                     System.out.println("Received MyItem " + myItem.id);
                 });

         sleep(Long.MAX_VALUE);
     }

     static void sleep(long milliseconds) {
         try {
             Thread.sleep(milliseconds);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
     }

     static final class MyItem {

         final int id;

         MyItem(int id) {
             this.id = id;
             System.out.println("Constructing MyItem " + id);
         }
     }
 }
```

输出如下：

```java
 ...
 Constructing MyItem 1001899
 Constructing MyItem 1001900
 Constructing MyItem 1001901
 Constructing MyItem 1001902
 Received MyItem 38
 Constructing MyItem 1001903
 Constructing MyItem 1001904
 Constructing MyItem 1001905
 Constructing MyItem 1001906
 Constructing MyItem 1001907
 ..
```

这只是我控制台输出的一个部分。注意，当创建 `MyItem 1001902` 时，`Observer` 仍在处理 `MyItem 38`。排放被推送的速度比 `Observer` 处理它们的速度快得多，并且由于排放积压在 `observeOn()` 中无限制地排队，这可能导致许多问题，包括 `OutOfMemoryError` 异常。

# 引入 `Flowable`

那我们应该如何减轻这种情况呢？你可以尝试使用原生的 Java 并发工具，例如信号量。但幸运的是，RxJava 为此问题提供了一个简化的解决方案：`Flowable`。`Flowable` 是 `Observable` 的背压变体，它告诉源以下游操作指定的速度发出。

在下面的代码中，将 `Observable.range()` 替换为 `Flowable.range()`，这将使整个链使用 `Flowable` 而不是 `Observable` 来工作。运行代码，你将看到输出有非常大的不同：

```java
 import io.reactivex.Observable;
 import io.reactivex.schedulers.Schedulers;
 import io.reactivex.Flowable;

 public class Launcher {

     public static void main(String[] args) {

         Flowable.range(1, 999_999_999)
                 .map(MyItem::new)
                 .observeOn(Schedulers.io())
                 .subscribe(myItem -> {
                     sleep(50);
                     System.out.println("Received MyItem " + myItem.id);
                 });

         sleep(Long.MAX_VALUE);
     }

     static void sleep(long milliseconds) {
         try {
             Thread.sleep(milliseconds);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
     }

     static final class MyItem {

         final int id;

         MyItem(int id) {
             this.id = id;
             System.out.println("Constructing MyItem " + id);
         }
     }
 }
```

输出如下：

```java
 Constructing MyItem 1
 Constructing MyItem 2
 Constructing MyItem 3
 ...
 Constructing MyItem 127
 Constructing MyItem 128
 Received MyItem 1
 Received MyItem 2
 Received MyItem 3
 ...
 Received MyItem 95
 Received MyItem 96
 Constructing MyItem 129
 Constructing MyItem 130
 Constructing MyItem 131
 ...
 Constructing MyItem 223
 Constructing MyItem 224
 Received MyItem 97
 Received MyItem 98
 Received MyItem 99
 ...
```

注意，`Flowable` 不使用观察者（Observers）订阅，而是使用订阅者（Subscribers），我们将在稍后深入探讨。

当使用 `Flowable` 时，你会注意到输出有非常大的不同。我使用 `...` 省略了前面输出的一部分，以突出一些关键事件。`Flowable.range()` 立即推送了 128 个排放，构建了 128 个 `MyItem` 实例。之后，`observeOn()` 将其中的 96 个推送到了 `Subscriber`。在这 96 个排放被 `Subscriber` 处理之后，又有 96 个从源推送出来。然后又有 96 个传递给了 `Subscriber`。

你看到模式了吗？源开始时推送了 128 个排放，之后，`Flowable` 链以每次 96 个排放的稳定流进行处理。这几乎就像整个 `Flowable` 链努力在任何给定时间不超过 96 个排放在其管道中。实际上，这正是正在发生的事情！这就是我们所说的 **背压**，它有效地引入了拉动态到基于推送的操作中，以限制源发出的频率。

但为什么`Flowable.range()`从 128 个发射开始，为什么`observeOn()`在请求另一个 96 个之前只向下发送 96 个，留下了 32 个未处理发射？初始的发射批次稍微大一些，所以如果有任何空闲时间，会有一些额外的工作被排队。如果在理论上我们的`Flowable`操作从请求 96 个发射开始，并继续每次发射 96 个发射，那么可能会有一些时刻操作可能会空闲等待下一个 96 个。因此，维护一个额外的 32 个发射的滚动缓存，在空闲时刻提供工作，这可以提供更高的吞吐量。这就像一个仓库在等待从工厂得到更多库存的同时，持有少量额外的库存来供应订单。

`Flowable`及其操作符的伟大之处在于它们通常为你做所有工作。除非你需要从头创建自己的`Flowable`或处理（如`Observable`）不实现背压的源，否则你不需要指定任何背压策略或参数。我们将在本章的其余部分讨论这些情况，并希望你不会经常遇到。

否则，`Flowable`就像我们迄今为止学到的几乎所有操作符的`Observable`。你可以从`Observable`转换为`Flowable`，反之亦然，我们将在后面讨论。但首先，让我们讨论我们应该在什么情况下使用`Flowable`而不是`Observable`。

# 何时使用 Flowables 和背压

知道何时使用`Flowable`而不是`Observable`是至关重要的。总的来说，`Flowable`提供的优势是更节省内存的使用（防止`OutOfMemoryError`异常）以及防止`MissingBackpressureException`。后者可能发生在操作对源进行背压，但源在其实现中没有背压协议的情况下。然而，`Flowable`的缺点是它增加了开销，可能不如`Observable`快速。

这里有一些指南可以帮助你选择使用`Observable`还是`Flowable`。

# 如果使用一个可观察对象...

+   你预计在`Observable`订阅的生命周期中发射的数量很少（少于 1000）或者发射是间歇性的且间隔很远。如果你只期望从源处发出少量的发射，一个`Observable`就能很好地完成任务并且开销更小。但是当你处理大量数据并对它们执行复杂操作时，你可能会想使用`Flowable`。

+   你的操作是严格同步的，并且并发使用有限。这包括在`Observable`链的开始简单使用`subscribeOn()`，因为该过程仍在单个线程上操作并同步向下发射项目。然而，当你开始在不同的线程上压缩和组合不同的流、并行化或使用如`observeOn()`、`interval()`和`delay()`这样的操作符时，你的应用程序就不再是同步的，你可能更倾向于使用`Flowable`。

+   你想要在 Android、JavaFX 或 Swing 上发射用户界面事件，如按钮点击、`ListView`选择或其他用户输入。由于用户不能被编程告知减慢速度，所以很少有机会使用`Flowable`。为了应对快速的用户输入，你可能会更倾向于使用第七章中讨论的操作符，*切换、节流、窗口化和缓冲*。

# 在以下情况下使用 Flowable...

+   你正在处理超过 10,000 个元素，并且源有机会以有规律的方式生成排放。当源是异步的并且推送大量数据时，这一点尤其正确。

+   你想要从支持阻塞并返回结果的 IO 操作中发射，这是许多 IO 源的工作方式。迭代记录的数据源，如文件中的行或 JDBC 中的`ResultSet`，特别容易控制，因为迭代可以根据需要暂停和恢复。可以请求一定量返回结果的网络和流式 API 也可以轻松地产生背压。

注意在 RxJava 1.0 中，`Observable`会受到背压，这本质上就是 RxJava 2.0 中的`Flowable`。`Flowable`和`Observable`成为不同类型的原因是它们在不同情况下的优点，如前所述。

你会发现你可以轻松地将 Observables 和 Flowables 一起使用。但你需要小心并意识到它们被使用的上下文以及可能出现的未期望的瓶颈。

# 理解 Flowable 和 Subscriber

几乎所有你到目前为止学到的`Observable`工厂和操作符也适用于 Flowable。在工厂方面，有`Flowable.range()`、`Flowable.just()`、`Flowable.fromIterable()`和`Flowable.interval()`。其中大多数为你实现了背压，并且使用方式通常与`Observable`等价。

然而，考虑一下`Flowable.interval()`，它在固定的时间间隔推动基于时间的排放。这能否在逻辑上产生背压？思考这样一个事实：每次排放都与它排放的时间紧密相关。如果我们减慢`Flowable.interval()`的速度，我们的排放将不再反映时间间隔，从而产生误导。因此，`Flowable.interval()`是标准 API 中少数几个在下游请求背压时可以抛出`MissingBackpressureException`的情况之一。在这里，如果我们每毫秒对`observeOn()`之后的慢速`intenseCalculation()`进行排放，我们将得到这个错误：

```java
 import io.reactivex.Flowable;
 import io.reactivex.schedulers.Schedulers;
 import java.util.concurrent.ThreadLocalRandom;
 import java.util.concurrent.TimeUnit;
 public class Launcher {
    public static void main(String[] args) {
         Flowable.interval(1, TimeUnit.MILLISECONDS)
                 .observeOn(Schedulers.io())
                 .map(i -> intenseCalculation(i))
                 .subscribe(System.out::println, Throwable::printStackTrace);
        sleep(Long.MAX_VALUE);
     }
    public static <T> T intenseCalculation(T value) {
         sleep(ThreadLocalRandom.current().nextInt(3000));
         return value;
     }
     public static void sleep(long millis) {
         try {
             Thread.sleep(millis);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
     }
 }
```

输出如下：

```java
0
io.reactivex.exceptions.MissingBackpressureException: Cant deliver value 128 due to lack of requests
    at io.reactivex.internal.operators.flowable.FlowableInterval
  ...
```

为了克服这个问题，你可以使用`onBackpresureDrop()`或`onBackPressureBuffer()`等操作符，我们将在本章后面学习。`Flowable.interval()`是那些在源头上逻辑上不能进行背压的工厂之一，因此你可以使用它后面的操作符来为你处理背压。否则，你使用的其他大多数`Flowable`工厂都支持背压。稍后，我们需要指出如何创建符合背压的自己的`Flowable`源，我们将在稍后讨论这个问题。但首先，我们将更深入地探讨`Subscriber`。

# 关于`Subscriber`

与`Observer`不同，`Flowable`使用`Subscriber`在`Flowable`链的末尾消费排放量和事件。如果你只传递 lambda 事件参数（而不是整个`Subscriber`对象），`subscribe()`不会返回`Disposable`，而是返回`Subscription`，可以通过调用`cancel()`而不是`dispose()`来取消它。`Subscription`还可以通过其`request()`方法向上游传达想要多少项。`Subscription`还可以在`Subscriber`的`onSubscribe()`方法中利用，以便在准备好接收排放量时立即`request()`元素。

就像`Observer`一样，创建`Subscriber`最快的方法是将 lambda 参数传递给`subscribe()`，正如我们之前所做的那样（以下代码再次展示了这一点）。这个`Subscriber`的默认实现将请求无界的排放量，但任何在其前面的操作员仍然会自动处理背压：

```java
 import io.reactivex.Flowable;
 import io.reactivex.schedulers.Schedulers;
 import java.util.concurrent.ThreadLocalRandom;
public class Launcher {
    public static void main(String[] args) {
       Flowable.range(1,1000)
                .doOnNext(s -> System.out.println("Source pushed " + s))
                .observeOn(Schedulers.io())
                .map(i -> intenseCalculation(i))
                .subscribe(s -> System.out.println("Subscriber received " + s),
                        Throwable::printStackTrace,
                        () -> System.out.println("Done!")
                );
        sleep(20000);
     }
     public static <T> T intenseCalculation(T value) {
         *//sleep up to 200 milliseconds*
         sleep(ThreadLocalRandom.current().nextInt(200));
         return value;
     }
    public static void sleep(long millis) {
         try {
             Thread.sleep(millis);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
     }
 }
```

当然，你也可以实现自己的`Subscriber`，它当然有`onNext()`、`onError()`、`onComplete()`方法以及`onSubscribe()`。这不像实现`Observer`那样直接，因为你需要在适当的时候在`Subscription`上调用`request()`来请求排放量。

实现一个`Subscriber`最快和最简单的方法是在`onSubscribe()`方法中调用`Subscription`上的`request(Long.MAX_VALUE)`，这本质上告诉上游“现在给我所有东西”。尽管前面的操作员将以自己的背压速度请求排放量，但最后操作员和`Subscriber`之间不会存在背压。这通常是正常的，因为上游操作员无论如何都会约束流量。

在这里，我们重新实现了我们之前的例子，但实现了我们自己的`Subscriber`：

```java
 import io.reactivex.Flowable;
 import io.reactivex.schedulers.Schedulers;
 import org.reactivestreams.Subscriber;
 import org.reactivestreams.Subscription;
 import java.util.concurrent.ThreadLocalRandom;
 public class Launcher {
    public static void main(String[] args) {
       Flowable.range(1,1000)
                .doOnNext(s -> System.out.println("Source pushed " + s))
                .observeOn(Schedulers.io())
                .map(i -> intenseCalculation(i))
                .subscribe(new Subscriber<Integer>() {
                    @Override
                    public void onSubscribe(Subscription subscription) {
                        subscription.request(Long.MAX_VALUE);
                    }
                   @Override
                    public void onNext(Integer s) {
                        sleep(50);
                        System.out.println("Subscriber received " + s);
                    }
                   @Override
                    public void onError(Throwable e) {
                        e.printStackTrace();
                    }
                   @Override
                    public void onComplete() {
                        System.out.println("Done!");
                    }
                });
        sleep(20000);
     }
     public static <T> T intenseCalculation(T value) {
         *//sleep up to 200 milliseconds*
         sleep(ThreadLocalRandom.current().nextInt(200));
         return value;
     }
    public static void sleep(long millis) {
         try {
             Thread.sleep(millis);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
     }
 }
```

如果你希望你的`Subscriber`与它前面的操作员建立显式的背压关系，你需要对`request()`调用进行微观管理。比如说，在某种极端情况下，你决定让`Subscriber`最初请求 40 个排放量，然后每次请求 20 个排放量。这是你需要做的：

```java
 import io.reactivex.Flowable;
 import io.reactivex.schedulers.Schedulers;
 import org.reactivestreams.Subscriber;
 import org.reactivestreams.Subscription;
 import java.util.concurrent.ThreadLocalRandom;
 import java.util.concurrent.atomic.AtomicInteger;
public class Launcher {
    public static void main(String[] args) {
       Flowable.range(1,1000)
                .doOnNext(s -> System.out.println("Source pushed " + s))
                .observeOn(Schedulers.io())
                .map(i -> intenseCalculation(i))
                .subscribe(new Subscriber<Integer>() {
                   Subscription subscription;
                    AtomicInteger count = new AtomicInteger(0);
                   @Override
                    public void onSubscribe(Subscription subscription) {
                        this.subscription = subscription;
                        System.out.println("Requesting 40 items!");
                        subscription.request(40);
                    }
                   @Override
                    public void onNext(Integer s) {
                        sleep(50);
                        System.out.println("Subscriber received " + s);
                       if (count.incrementAndGet() % 20 == 0 && count.get() >= 40)
                            System.out.println("Requesting 20 more!");
                            subscription.request(20);
                    }
                   @Override
                    public void onError(Throwable e) {
                        e.printStackTrace();
                    }
                   @Override
                    public void onComplete() {
                        System.out.println("Done!");
                    }
                });
        sleep(20000);
     }
     public static <T> T intenseCalculation(T value) {
         //sleep up to 200 milliseconds
         sleep(ThreadLocalRandom.current().nextInt(200));
         return value;
     }
    public static void sleep(long millis) {
         try {
             Thread.sleep(millis);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
     }
 }
```

输出如下：

```java
 Requesting 40 items!
 Source pushed 1
 Source pushed 2
 ...
 Source pushed 127
 Source pushed 128
 Subscriber received 1
 Subscriber received 2
 ...
 Subscriber received 39
 Subscriber received 40
 Requesting 20 more!
 Subscriber received 41
 Subscriber received 42
 ...
 Subscriber received 59
 Subscriber received 60
 Requesting 20 more!
 Subscriber received 61
 Subscriber received 62
 ...
 Subscriber received 79
 Subscriber received 80
 Requesting 20 more!
 Subscriber received 81
 Subscriber received 82
 ...
```

注意，源仍然最初排放 128 个排放量，然后仍然每次推送 96 个排放量。但是我们的`Subscriber`只接收了 40 个排放量，正如指定的那样，然后持续要求再增加 20 个。我们`Subscriber`中的`request()`调用只与它上游的即时操作员`map()`通信。`map()`操作员可能将那个请求转发到`observeOn()`，它正在缓存项目，并且只根据`Subscriber`的要求输出 40 个和 20 个。当其缓存变低或清空时，它将从上游请求另一个 96 个。

这是一个警告：你不应该依赖于请求排放的确切数字，例如 128 和 96。这些是我们偶然观察到的内部实现，这些数字可能会在未来为了进一步优化实现而改变。

这种自定义实现实际上可能正在降低我们的吞吐量，但它展示了如何通过自己的`Subscriber`实现来管理自定义背压。只需记住，`request()`调用并不一直向上游传递。它们只到达前面的操作员，该操作员决定如何将那个请求转发到上游。

# 创建一个 Flowable

在这本书的早期，我们多次使用`Observable.create()`从头开始创建我们自己的`Observable`，这描述了在订阅时如何输出项目，如下面的代码片段所示：

```java
import io.reactivex.Observable;
import io.reactivex.schedulers.Schedulers;
public class Launcher {
    public static void main(String[] args) {
        Observable<Integer> source = Observable.create(emitter -> {
             for (int i=0; i<=1000; i++) {
                 if (emitter.isDisposed())
                     return;
                emitter.onNext(i);
             }
            emitter.onComplete();
         });
        source.observeOn(Schedulers.io())
                 .subscribe(System.out::println);
        sleep(1000);
     }
 }
```

输出如下：

```java
 0
 1
 2
 3
 4
 ...
```

这个`Observable.create()`将输出从 0 到 1000 的整数，然后调用`onComplete()`。如果对从`subscribe()`返回的`Disposable`调用`dispose()`，它可以被突然停止，for 循环将检查这一点。

然而，思考一下，如果我们执行`Flowable.create()`，即`Observable.create()`的`Flowable`等效，如何实现类似这样的背压。使用像前面的简单 for 循环，没有基于下游`Subscriber`请求的排放*停止*和*恢复*的概念。正确实现背压将增加一些复杂性。有一些更简单的方法来支持背压，但它们通常涉及妥协策略，如缓冲和丢弃，我们将在首先介绍。还有一些工具可以在源处实现背压，我们将在之后介绍。

# 使用 Flowable.create()和 BackpressureStrategy

利用`Flowable.create()`创建一个`Flowable`感觉就像`Observable.create()`，但有一个关键的区别；你必须指定一个`BackpressureStrategy`作为第二个参数。这种可枚举类型并不提供任何形式的背压支持的魔法实现。事实上，这仅仅通过缓存或丢弃排放量或根本不实现背压来支持背压。

在这里，我们使用`Flowable.create()`来创建一个`Flowable`，但我们提供了一个第二个`BackpressureStrategy.BUFFER`参数来在背压之前缓存排放量：

```java
 import io.reactivex.BackpressureStrategy;
 import io.reactivex.Flowable;
 import io.reactivex.schedulers.Schedulers;
public class Launcher {
    public static void main(String[] args) {
        Flowable<Integer> source = Flowable.create(emitter -> {
             for (int i=0; i<=1000; i++) {
                 if (emitter.isCancelled())
                     return;
                emitter.onNext(i);
             }
            emitter.onComplete();
         }, BackpressureStrategy.BUFFER);
        source.observeOn(Schedulers.io())
                 .subscribe(System.out::println);
        sleep(1000);
     }
    public static void sleep(long millis) {
         try {
             Thread.sleep(millis);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
     }
 }
```

输出如下：

```java
 0
 1
 2
 3
 4
 ...
```

这不是最佳方案，因为排放将被保留在一个无界队列中，当`Flowable.create()`推送过多的排放时，你可能会得到一个`OutOfMemoryError`。但至少它防止了`MissingBackpressureException`，并且可以在一定程度上使你的自定义`Flowable`可行。我们将在本章后面学习使用`Flowable.generate()`实现更健壮的反压方法。

目前有五种`BackpressureStrategy`选项可供选择。

| **反压策略** | **描述** |
| --- | --- |
| 缺失 | 实际上没有实现任何反压。下游必须处理反压溢出，当与我们在本章后面将要介绍的`onBackpressureXXX()`操作符一起使用时可能很有帮助。 |
| 错误 | 当下游无法跟上源时，会立即触发`MissingBackpressureException`。 |
| BUFFER | 在无界队列中排队排放，直到下游能够消费它们，但如果队列太大，可能会引起`OutOfMemoryError`。 |
| DROP | 如果下游无法跟上，这将忽略上游排放，并在下游忙碌时不会排队任何内容。 |
| 最新 | 这将只保留最新的排放，直到下游准备好接收它。 |

接下来，我们将看到一些这些策略作为操作符的使用，特别是将 Observables 转换为 Flowables。

# 将 Observable 转换为 Flowable（反之亦然）

你还可以通过将`Observable`转换为`Flowable`来对没有反压概念的数据源实现`BackpressureStrategy`。你可以通过调用其`toFlowable()`操作符并传递一个`BackpressureStrategy`作为参数来轻松地将 Observable 转换为`Flowable`。在下面的代码中，我们使用`BackpressureStrategy.BUFFER`将`Observable.range()`转换为`Flowable`。Observable 没有反压概念，所以它将尽可能快地推送项目，不管下游是否能跟上。但是，带有缓冲策略的`toFlowable()`将在下游无法跟上时作为代理来缓冲排放：

```java
 import io.reactivex.BackpressureStrategy;
 import io.reactivex.Observable;
 import io.reactivex.schedulers.Schedulers;
public class Launcher {
    public static void main(String[] args) {
        Observable<Integer> source = Observable.range(1,1000);
        source.toFlowable(BackpressureStrategy.BUFFER)
                 .observeOn(Schedulers.io())
                 .subscribe(System.out::println);
        sleep(10000);
     }
    public static void sleep(long millis) {
         try {
             Thread.sleep(millis);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
     }
 }
```

再次注意，带有缓冲策略的`toFlowable()`将有一个无界队列，这可能会导致`OutOfMemoryError`。在现实世界中，最好首先使用`Flowable.range()`，但有时你可能只能得到一个`Observable`。

`Flowable`还有一个`toObservable()`操作符，它将`Flowable<T>`转换为`Observable<T>`。这有助于使`Flowable`在`Observable`链中使用，特别是与`flatMap()`等操作符一起使用，如下面的代码所示：

```java
 import io.reactivex.Flowable;
 import io.reactivex.Observable;
 import io.reactivex.schedulers.Schedulers;
public class Launcher {
    public static void main(String[] args) {
        Flowable<Integer> integers =
                 Flowable.range(1, 1000)
                         .subscribeOn(Schedulers.computation());
        Observable.just("Alpha","Beta","Gamma","Delta","Epsilon")
                 .flatMap(s -> integers.map(i -> i + "-" + s).toObservable())
                 .subscribe(System.out::println);
        sleep(5000);
     }
    public static void sleep(long millis) {
         try {
             Thread.sleep(millis);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
     }
 }
```

如果`Observable<String>`有超过五个排放（例如 1,000 或 10,000），那么将其转换为`Flowable`可能更好，而不是将扁平映射的`Flowable`转换为`Observable`。

即使你调用 `toObservable()`，`Flowable` 仍然会在上游利用背压。但到了它成为 `Observable` 的点，下游将不再受到背压，并请求 `Long.MAX_VALUE` 数量的发射。只要下游没有发生更复杂的操作或并发变化，并且上游的 `Flowable` 操作限制了发射的数量，这可能就足够了。

但通常情况下，当你承诺使用一个 `Flowable` 时，你应该努力使你的操作保持 `Flowable`。

# 使用 onBackpressureXXX() 操作符

如果你提供了一个没有背压实现（包括从 `Observable` 派生的）的 `Flowable`，你可以使用 `onBackpressureXXX()` 操作符应用 `BackpressureStrategy`。这些操作符还提供了一些额外的配置选项。如果，例如，你有一个 `Flowable.interval()` 发射速度比消费者处理速度快，这可能会很有用。`Flowable.interval()` 由于是时间驱动的，不能在源处减慢速度，但我们可以使用 `onBackpressureXXX()` 操作符在它和下游之间代理。我们将使用 `Flowable.interval()` 进行这些示例，但这可以应用于任何没有实现背压的 `Flowable`。

有时，`Flowable` 可能只是配置了 `BackpressureStrategy.MISSING`，这样 `onBackpressureXXX()` 操作符就可以稍后指定策略。

# onBackPressureBuffer()

`onBackPressureBuffer()` 将接受一个假设未实现背压的现有 `Flowable`，并在该点向下游应用 `BackpressureStrategy.BUFFER`。由于 `Flowable.interval()` 在源处不能进行背压，将其放在 `onBackPressureBuffer()` 之后将代理一个背压队列到下游：

```java
import io.reactivex.Flowable;
 import io.reactivex.schedulers.Schedulers;
 import java.util.concurrent.TimeUnit;
public class Launcher {
    public static void main(String[] args) {
        Flowable.interval(1, TimeUnit.MILLISECONDS)
                 .onBackpressureBuffer()
                 .observeOn(Schedulers.io())
                 .subscribe(i -> {
                     sleep(5);
                     System.out.println(i);
                 });
        sleep(5000);
     }
    public static void sleep(long millis) {
         try {
             Thread.sleep(millis);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
     }
 }
```

输出如下：

```java
 0
 1
 2
 3
 4
 5
 6
 7
 ...
```

你还可以提供一些重载参数。我们不会详细介绍所有这些参数，你可以参考 JavaDocs 获取更多信息，但我们将突出显示常见的参数。容量参数将为缓冲区创建一个最大阈值，而不是允许其无界。你可以指定一个 `onOverflow` `Action` lambda，当溢出超过容量时触发一个动作。你还可以指定一个 `BackpressureOverflowStrategy` 枚举来指示如何处理超过容量的溢出。

这里是你可以选择的三个 `BackpressureOverflowStrategy` 枚举项：

| **BackpressureOverflowStrategy** | **描述** |
| --- | --- |
| ERROR | 当容量超过时立即抛出错误 |
| DROP_OLDEST | 从缓冲区中丢弃最旧的值，为新值腾出空间 |
| DROP_LATEST | 从缓冲区中丢弃最新的值，以优先处理较旧的未消费值 |

在以下代码中，我们保持最大容量为 10，并在溢出时指定使用 `BackpressureOverflowStrategy.DROP_LATEST`。我们还会在溢出时打印一个通知：

```java
import io.reactivex.BackpressureOverflowStrategy;
 import io.reactivex.Flowable;
 import io.reactivex.schedulers.Schedulers;
 import java.util.concurrent.TimeUnit;
public class Launcher {
    public static void main(String[] args) {
        Flowable.interval(1, TimeUnit.MILLISECONDS)
                 .onBackpressureBuffer(10,
                         () -> System.out.println("overflow!"),
                         BackpressureOverflowStrategy.DROP_LATEST)
                 .observeOn(Schedulers.io())
                 .subscribe(i -> {
                     sleep(5);
                     System.out.println(i);
                 });
        sleep(5000);
     }
    public static void sleep(long millis) {
         try {
             Thread.sleep(millis);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
     }
 }
```

输出如下：

```java
 ...
 overflow!
 overflow!
 135
 overflow!
 overflow!
 overflow!
 overflow!
 overflow!
 136
 overflow!
 overflow!
 overflow!
 overflow!
 overflow!
 492
 overflow!
 overflow!
 overflow!
 ...
```

注意，在我嘈杂的输出这部分，`136`和`492`之间跳过了很大一段数字范围。这是因为由于`BackpressureOverflowStrategy.DROP_LATEST`，这些排放被从队列中丢弃。队列已经充满了等待被消费的排放，所以新的排放被忽略了。

# `onBackPressureLatest()`

`onBackpressureBuffer()`的一个轻微变体是`onBackPressureLatest()`。当下游忙碌时，它将保留源的最新值，一旦下游空闲可以处理更多，它将提供最新值。在此忙碌期间发出的任何先前值都将丢失：

```java
import io.reactivex.Flowable;
 import io.reactivex.schedulers.Schedulers;
import java.util.concurrent.TimeUnit;
public class Launcher {
    public static void main(String[] args) {
        Flowable.interval(1, TimeUnit.MILLISECONDS)
                 .onBackpressureLatest()
                 .observeOn(Schedulers.io())
                 .subscribe(i -> {
                     sleep(5);
                     System.out.println(i);
                 });
        sleep(5000);
     }
    public static void sleep(long millis) {
         try {
             Thread.sleep(millis);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
     }
 }
```

输出结果如下：

```java
 ...
 122
 123
 124
 125
 126
 127
 494
 495
 496
 497
 ...
```

如果你研究我的输出，你会注意到`127`和`494`之间有一个跳跃。这是因为所有介于它们之间的数字最终都被`494`这个最新值击败，当时，下游已经准备好处理更多的排放。它从消费缓存的`494`和其他之前被丢弃的排放开始。

# `onBackPressureDrop()`

`onBackpressureDrop()`将简单地丢弃下游太忙无法处理的排放。当下游已经忙碌（例如，一个"`RUN`"请求被重复发送，尽管结果过程已经运行）时，这很有帮助。你可以选择性地提供一个`onDrop` lambda 参数，指定对每个丢弃项要做什么，我们将简单地按照以下代码打印出来：

```java
import io.reactivex.Flowable;
 import io.reactivex.schedulers.Schedulers;
 import java.util.concurrent.TimeUnit;
public class Launcher {
    public static void main(String[] args) {
        Flowable.interval(1, TimeUnit.MILLISECONDS)
                 .onBackpressureDrop(i -> System.out.println("Dropping " + i))
                 .observeOn(Schedulers.io())
                 .subscribe(i -> {
                     sleep(5);
                     System.out.println(i);
                 });
        sleep(5000);
     }
    public static void sleep(long millis) {
         try {
             Thread.sleep(millis);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
     }
 }
```

输出结果如下：

```java
 ...
 Dropping 653
 Dropping 654
 Dropping 655
 Dropping 656
 127
 Dropping 657
 Dropping 658
 Dropping 659
 Dropping 660
 Dropping 661
 493
 Dropping 662
 Dropping 663
 Dropping 664
 ...
```

在我的输出中，注意`127`和`493`之间有一个很大的跳跃。它们之间的数字被丢弃，因为当它们准备好被处理时，下游已经忙碌，所以它们被丢弃而不是排队。

# 使用`Flowable.generate()`

本章到目前为止我们讨论的很多内容都没有展示出对源进行背压的最优方法。是的，使用`Flowable`以及大多数标准工厂和操作符会自动为你处理背压。然而，如果你正在创建自己的自定义源，`Flowable.create()`或`onBackPressureXXX()`操作符在处理背压请求方面有些妥协。虽然对于某些情况来说既快又有效，但缓存排放或简单地丢弃它们并不总是可取的。最好是首先让源受到背压。

幸运的是，`Flowable.generate()`存在，可以帮助创建背压，在很好地抽象级别上尊重源。它将接受一个`Consumer<Emitter<T>>`，就像`Flowable.create()`一样，但它将使用 lambda 来指定每次从上游请求项目时要传递的`onNext()`、`onComplete()`和`onError()`事件。

在你使用 `Flowable.generate()` 之前，考虑将你的源 `Iterable<T>` 改为 `Iterable<T>` 并将其传递给 `Flowable.fromIterable()`。`Flowable.fromIterable()` 会尊重背压，并且对于许多情况来说可能更容易使用。否则，如果你需要更具体的东西，`Flowable.generate()` 是你的下一个最佳选择。

`Flowable.generate()` 的最简单重载只接受 `Consumer<Emitter<T>>` 并假设在发射之间没有维护状态。这有助于创建一个具有背压感知的随机整数生成器，如下所示。请注意，立即发出 128 个发射，但之后，在从源发送另一个 96 个之前，会向下游推送 96 个：

```java
import io.reactivex.Flowable;
 import io.reactivex.schedulers.Schedulers;
 import java.util.concurrent.ThreadLocalRandom;
public class Launcher {
    public static void main(String[] args) {
        randomGenerator(1,10000)
                 .subscribeOn(Schedulers.computation())
                 .doOnNext(i -> System.out.println("Emitting " + i))
                 .observeOn(Schedulers.io())
                 .subscribe(i -> {
                     sleep(50);
                     System.out.println("Received " + i);
                 });
        sleep(10000);
     }
    static Flowable<Integer> randomGenerator(int min, int max) {
         return Flowable.generate(emitter ->
                 emitter.onNext(ThreadLocalRandom.current().nextInt(min, max))
         );
     }
    public static void sleep(long millis) {
         try {
             Thread.sleep(millis);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
     }
 }
```

输出如下：

```java
 ...
 Emitting 8014
 Emitting 3112
 Emitting 5958
 Emitting 4834 //128th emission
 Received 9563
 Received 4359
 Received 9362
 ...
 Received 4880
 Received 3192
 Received 979 //96th emission
 Emitting 8268
 Emitting 3889
 Emitting 2595
...
```

使用 `Flowable.generate()`，在 `Consumer<Emitter<T>>` 中调用多个 `onNext()` 操作符将导致 `IllegalStateException`。下游只需要调用一次 `onNext()`，因此它可以按照需要重复调用，以保持流动。如果发生异常，它还会为您发出 `onError()`。

您还可以提供一个类似于 `reduce()` 的“种子”状态，以维护从一次发射到下一次发射传递的状态。假设我们想要创建类似于 `Flowable.range()` 的东西，但相反，我们想要在 `upperBound` 和 `lowerBound` 之间反向发出整数。使用 `AtomicInteger` 作为我们的状态，我们可以递减它并将它的值传递给发射器的 `onNext()` 操作符，直到遇到 `lowerBound`。这如下所示：

```java
import io.reactivex.Flowable;
 import io.reactivex.schedulers.Schedulers;
 import java.util.concurrent.atomic.AtomicInteger;
public class Launcher {
    public static void main(String[] args) {
        rangeReverse(100,-100)
                 .subscribeOn(Schedulers.computation())
                 .doOnNext(i -> System.out.println("Emitting " + i))
                 .observeOn(Schedulers.io())
                 .subscribe(i -> {
                     sleep(50);
                     System.out.println("Received " + i);
                 });
        sleep(50000);
     }
    static Flowable<Integer> rangeReverse(int upperBound, int lowerBound) {
         return Flowable.generate(() -> new AtomicInteger(upperBound + 1),
                 (state, emitter) -> {
                     int current = state.decrementAndGet();
                     emitter.onNext(current);
                     if (current == lowerBound)
                         emitter.onComplete();
                 }
         );
     }
    public static void sleep(long millis) {
         try {
             Thread.sleep(millis);
         } catch (InterruptedException e) {
             e.printStackTrace();
         }
     }
 }
```

输出如下：

```java
 Emitting 100
 Emitting 99
 ...
 Emitting -25
 Emitting -26
 Emitting -27 //128th emission
 Received 100
 Received 99
 Received 98
 ...
 Received 7
 Received 6
 Received 5 // 96th emission
 Emitting -28
 Emitting -29
 Emitting -30
```

`Flowable.generator()` 提供了一个很好地抽象化的机制来创建一个尊重背压的源。因此，如果你不希望与缓存或丢弃发射项打交道，你可能会更喜欢使用这个而不是 `Flowable.create()`。

使用 `Flowable.generate()`，您还可以提供一个第三个 `Consumer<? super S> disposeState` 参数，在终止时执行任何清理操作，这对于 IO 源可能很有帮助。

# 摘要

在本章中，你学习了关于 `Flowable` 和背压以及它应该在哪些情况下优先于 `Observable`。当并发进入你的应用程序并且大量数据可以通过它流动时，Flowables 特别受欢迎，因为它调节了在给定时间内从源处来的数据量。一些 Flowables，如 `Flowable.interval()` 或从 `Observable` 派生的那些，没有实现背压。在这些情况下，你可以使用 `onBackpressureXXX()` 操作符来排队或丢弃下游的发射。如果你是从头开始创建自己的 `Flowable` 源，则更喜欢使用现有的 `Flowable` 工厂，如果那失败了，则更喜欢使用 `Flowable.generate()` 而不是 `Flowable.create()`。

如果你已经到达这个阶段并且理解了这本书到目前为止的大部分内容，恭喜你！你已经拥有了 RxJava 的核心概念，而本书的剩余部分对你来说将是一条轻松的道路。下一章将介绍如何创建自己的操作符，这可能会是一项相对复杂的工作。至少，你应该知道如何组合现有的操作符来创建新的操作符，这将是下一个话题之一。
