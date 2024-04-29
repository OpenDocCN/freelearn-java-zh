# 第六章。使用调度程序进行并发和并行处理

现代处理器具有多个核心，并且能够同时更快地处理许多耗时操作。Java 并发 API（包括线程等）使这成为可能。

RxJava 的`Observable`链似乎很适合线程。如果我们可以在后台*订阅*我们的源并进行所有的转换、组合和过滤，然后在一切都完成时将结果传递给主线程，那将是很棒的。是的，这听起来很美好，但是 RxJava 默认是单线程的。这意味着，在大多数情况下，当在`Observable`实例上调用`subscribe`方法时，当前线程会阻塞直到所有内容被发出。（这对于由`interval`或`timer`工厂方法创建的`Observable`实例并不成立，例如。）这是一件好事，因为处理线程并不那么容易。它们很强大，但它们需要彼此同步；例如，当一个依赖于另一个的结果时。

在多线程环境中最难管理的事情之一是线程之间的共享数据。一个线程可以从数据源中读取，而另一个线程正在修改它，这导致不同版本的相同数据被不同的线程使用。如果`Observable`链构造得当，就没有共享状态。这意味着同步并不那么复杂。

在本章中，我们将讨论并行执行事务，并了解并发意味着什么。此外，我们将学习一些处理我们的`Observable`实例发出太多项目的情况的技术（这在多线程环境中并不罕见）。本章涵盖的主题如下：

+   使用`Scheduler`实例实现*并发*

+   使用`Observable`实例的**缓冲**、**节流**和**去抖动**

# RxJava 的调度程序

调度程序是 RxJava 实现并发的方式。它们负责为我们创建和管理线程（在内部依赖于 Java 的线程池设施）。我们不会涉及 Java 的并发 API 及其怪癖和复杂性。我们一直在使用调度程序，隐式地使用定时器和间隔，但是现在是掌握它们的时候了。

让我们回顾一下我们在第三章中介绍的`Observable.interval`工厂方法，*创建和连接 Observables、Observers 和 Subjects*。正如我们之前看到的，RxJava 默认情况下是*单线程*的，所以在大多数情况下，在`Observable`实例上调用`subscribe`方法会阻塞当前线程。但是`interval Observable`实例并非如此。如果我们查看`Observable<Long> interval(long interval, TimeUnit unit)`方法的 JavaDoc，我们会看到它说，由它创建的`Observable`实例在一个叫做“计算调度程序”的东西上运行。

为了检查`interval`方法的行为（以及本章中的其他内容），我们将需要一个强大的调试工具。这就是为什么我们在本章中要做的第一件事。

## 调试 Observables 和它们的调度程序

在上一章中，我们介绍了`doOnNext()`操作符，它可以用于直接从`Observable`链中记录发出的项目。我们提到了`doOnError()`和`doOnCompleted()`操作符。但是有一个结合了所有三者的操作符——`doOnEach()`操作符。我们可以从中记录所有内容，因为它接收所有发出的通知，而不管它们的类型。我们可以将它放在操作符链的中间，并使用它来记录状态。它接受一个`Notification -> void`函数。

这是一个返回`lambda`结果的高阶*debug*函数的源代码，它能够记录使用传递的描述标记的`Observable`实例的发射：

```java
<T> Action1<Notification<? super T>> debug(
  String description, String offset
) {
  AtomicReference<String> nextOffset = new AtomicReference<String>(">");
  return (Notification<? super T> notification) -> {
    switch (notification.getKind()) {
    case OnNext:
      System.out.println(
        Thread.currentThread().getName() +
        "|" + description + ": " + offset +
        nextOffset.get() + notification.getValue()
      );
      break;
    case OnError:
      System.err.println(
        Thread.currentThread().getName() +
        "|" + description + ": " + offset +
        nextOffset.get() + " X " + notification.getThrowable()
      );
      break;
    case OnCompleted:
      System.out.println(
        Thread.currentThread().getName() +
        "|" + description + ": " + offset +
        nextOffset.get() + "|"
      );
    default:
      break;
    }
    nextOffset.getAndUpdate(p -> "-" + p);
  };
}
```

根据传递的*description*和*offset*，返回的方法记录每个通知。然而，重要的是，在一切之前记录当前活动线程的名称。`<value>`标记*OnNext 通知*；`X`标记*OnError 通知*；`|`标记*OnCompleted 通知*，`nextOffset`变量用于显示时间上的值。

这是使用这个新方法的一个例子：

```java
Observable
  .range(5, 5)
  .doOnEach(debug("Test", ""))
  .subscribe();
```

这个例子将生成五个连续的数字，从数字五开始。我们通过调用我们的`debug(String, String)`方法传递给`doOnEach()`操作符来记录`range()`方法调用之后的一切。通过不带参数的订阅调用，这个小链将被触发。输出如下：

```java
main|Test: >5
main|Test: ->6
main|Test: -->7
main|Test: --->8
main|Test: ---->9
main|Test: ----->|

```

首先记录的是当前线程的名称（主线程），然后是传递给`debug()`方法的`Observable`实例的描述，之后是一个冒号和破折号形成的箭头，表示时间。最后是通知类型的符号——对于值本身是值，对于完成是`|`。

让我们定义`debug()`辅助方法的一个重载，这样我们就不需要传递第二个参数给它，如果不需要额外的偏移量：

```java
<T> Action1<Notification<? super T>> debug(String description) {
  return debug(description, "");
}
```

### 注意

前面方法的代码可以在以下链接查看/下载：[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/common/Helpers.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/common/Helpers.java)。

现在我们准备调试由间隔方法创建的`Observable`实例发生了什么！

## 间隔 Observable 及其默认调度程序

让我们来看下面的例子：

```java
Observable
  .take(5)
  .interval(500L, TimeUnit.MILLISECONDS)
  .doOnEach(debug("Default interval"))
  .subscribe();
```

这创建了一个`interval Observable`实例，每隔半秒发出一次。我们使用`take()`方法只获取前五个*通知*并完成。我们将使用我们的`debug()`辅助方法记录由间隔方法创建的`Observable`实例发出的值，并使用`subscribe()`调用来触发逻辑。输出应该如下所示：

```java
RxComputationThreadPool-1|Default interval: >0
RxComputationThreadPool-1|Default interval: ->1
RxComputationThreadPool-1|Default interval: -->2
RxComputationThreadPool-1|Default interval: --->3
RxComputationThreadPool-1|Default interval: ---->4

```

这里应该都很熟悉，除了`Observable`实例执行的线程！这个线程不是*主*线程。看起来它是由 RxJava 管理的可重用`Thread`实例池创建的，根据它的名称（`RxComputationThreadPool-1`）。

如果你还记得，`Observable.interval`工厂方法有以下重载：

```java
Observable<Long> interval(long, TimeUnit, Scheduler)
```

这意味着我们可以指定它将在哪个调度程序上运行。之前提到过，只有两个参数的重载在*computation*调度程序上运行。所以，现在让我们尝试传递另一个调度程序，看看会发生什么：

```java
Observable
  .take(5)
  .interval(500L, TimeUnit.MILLISECONDS, Schedulers.immediate())
  .doOnEach(debug("Imediate interval"))
  .subscribe();
```

这与以前相同，但有一点不同。我们传递了一个名为*immediate*的调度程序。这样做的想法是立即在当前运行的线程上执行工作。结果如下：

```java
main|Imediate interval: >0
main|Imediate interval: ->1
main|Imediate interval: -->2
main|Imediate interval: --->3
main|Imediate interval: ---->4

```

通过指定这个调度程序，我们使`interval Observable`实例在当前*主*线程上运行。

### 注意

前面例子的源代码可以在以下链接找到：[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter06/IntervalAndSchedulers.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter06/IntervalAndSchedulers.java)。

借助调度程序的帮助，我们可以指示我们的操作符在特定线程上运行或使用特定的线程池。

我们刚刚讨论的一切都导致了这样的结论：调度程序会生成新的线程，或者重用已经生成的线程，*操作*是`Observable`实例链的一部分，会在这些线程上执行。因此，我们可以通过仅使用它们来实现并发（操作同时进行）。

为了拥有*多线程*逻辑，我们只需要学习这两件事：

+   我们可以选择的调度程序类型

+   如何在任意`Observable`链的*操作*中使用这些调度程序

## 调度程序的类型

有几种专门用于某种类型操作的`schedulers`。为了更多地了解它们，让我们看一下`Scheduler`类。

事实证明这个类非常简单。它只有两个方法，如下所示：

+   `long now()`

+   `abstract Worker createWorker()`

第一个返回当前时间的毫秒数，第二个创建一个`Worker`实例。这些`Worker`实例用于在单个线程或事件循环上执行操作（取决于实现）。使用`Worker`的`schedule*`方法来安排执行操作。`Worker`类实现了`Subscription`接口，因此它有一个`unsubscribe()`方法。*取消订阅*`Worker`会*取消排队*的所有未完成工作，并允许资源清理。

我们可以使用工作线程在`Observable`上下文之外执行调度。对于每种`Scheduler`类型，我们可以做到以下几点：

```java
scheduler.createWorker().schedule(Action0);
```

这将安排传递的操作并执行它。在大多数情况下，这个方法不应该直接用于调度工作，我们只需选择正确的调度程序并在其上安排操作即可。为了了解它们的作用，我们可以使用这个方法来检查各种可用的调度程序类型。

让我们定义一个测试方法：

```java
void schedule(Scheduler scheduler, int numberOfSubTasks, boolean onTheSameWorker) {
  List<Integer> list = new ArrayList<>(0);
  AtomicInteger current = new AtomicInteger(0);
  Random random = new Random();
  Worker worker = scheduler.createWorker();
  Action0 addWork = () -> {
    synchronized (current) {
      System.out.println("  Add : " + Thread.currentThread().getName() + " " + current.get());
      list.add(random.nextInt(current.get()));
      System.out.println("  End add : " + Thread.currentThread().getName() + " " + current.get());
    }
  };
  Action0 removeWork = () -> {
    synchronized (current) {
      if (!list.isEmpty()) {
        System.out.println("  Remove : " + Thread.currentThread().getName());
        list.remove(0);
        System.out.println("  End remove : " + Thread.currentThread().getName());
      }
    }
  };
  Action0 work = () -> {
    System.out.println(Thread.currentThread().getName());
    for (int i = 1; i <= numberOfSubTasks; i++) {
      current.set(i);
      System.out.println("Begin add!");
      if (onTheSameWorker) {
        worker.schedule(addWork);
      }
      else {
 scheduler.createWorker().schedule(addWork);
      }
      System.out.println("End add!");
    }
    while (!list.isEmpty()) {
      System.out.println("Begin remove!");
    if (onTheSameWorker) {
 worker.schedule(removeWork);
    }
    else {
 scheduler.createWorker().schedule(removeWork);
    }
    System.out.println("End remove!");
  };
  worker.schedule(work);
}
```

该方法使用传递的`Scheduler`实例来执行一些工作。有一个选项可以指定它是否应该为每个任务使用相同的`Worker`实例，或者为每个子任务生成一个新的`Worker`实例。基本上，虚拟工作包括用随机数填充列表，然后逐个删除这些数字。每个*添加操作*和*删除操作*都是通过传递的`Scheduler`实例创建的工作线程作为子任务进行调度的。在每个子任务之前和之后，当前线程和一些额外信息都被记录下来。

### 提示

在现实世界的场景中，一旦所有工作都完成了，我们应该始终调用`worker.unsubscribe()`方法。

转向预定义的`Scheduler`实例。它们可以通过`Schedulers`类中包含的一组静态方法来获取。我们将使用之前定义的调试方法来检查它们的行为，以了解它们的差异和用处。

### `Schedulers.immediate`调度程序

`Schedulers.immediate`调度程序在此时此刻执行工作。当一个操作传递给它的工作线程的`schedule(Action0)`方法时，它就会被调用。假设我们用它来运行我们的测试方法，就像这样：

```java
schedule(Schedulers.immediate(), 2, false);
schedule(Schedulers.immediate(), 2, true);
```

在这两种情况下，结果看起来都是这样的：

```java
main
Begin add!
 Add : main 1
 End add : main 1
End add!
Begin add!
 Add : main 2
 End add : main 2
End add!
Begin remove!
 Remove : main
 End remove : main
End remove!
Begin remove!
 Remove : main
 End remove : main
End remove!

```

换句话说，一切都在调用线程上执行——主线程上，没有任何并行操作。

这个调度程序可以用来在前台执行`interval()`和`timer()`等方法。

### `Schedulers.trampoline`调度程序

通过`Schedulers.trampoline`方法检索到的调度程序会在当前`线程`上*排队*子任务。排队的工作会在当前正在进行的工作完成后执行。假设我们要运行这个：

```java
schedule(Schedulers.trampoline(), 2, false);
schedule(Schedulers.trampoline(), 2, true);
```

在第一种情况下，结果将与立即调度程序相同，因为所有任务都是在它们自己的`Worker`实例中执行的，因此每个工作线程只有一个任务要排队执行。但是当我们使用相同的`Worker`实例来调度每个子任务时，我们会得到这样的结果：

```java
main
Begin add!
End add!
Begin add!
End add!
 Add : main 2
 End add : main 2
 Add : main 2
 End add : main 2

```

换句话说，它将首先执行整个主要操作，然后执行子任务；因此，`List`实例将被填充（子任务已入队），但永远不会被清空。这是因为在执行主任务时，`List`实例仍然为空，并且`while`循环没有被触发。

### 注意

*trampoline*调度程序可用于在递归运行多个任务时避免`StackOverflowError`异常。例如，假设一个任务完成后调用自身执行一些新工作。在单线程环境中，这将导致由于递归而导致堆栈溢出；但是，如果我们使用*trampoline*调度程序，它将序列化所有已安排的活动，并且堆栈深度将保持正常。但是，*trampoline*调度程序通常比*immediate*调度程序慢。因此，使用正确的调度程序取决于用例。

### Schedulers.newThread 调度程序

此调度程序为每个新的`Worker`实例创建一个*new* `Thread`实例（确切地说是单线程的`ScheduledThreadPoolExecutor`实例）。此外，每个工作人员通过其`schedule()`方法排队接收到的操作，就像*trampoline*调度程序一样。让我们看看以下代码：

```java
schedule(Schedulers.newThread(), 2, true);
```

它将具有与*trampoline*相同的行为，但将在新的`thread:`中运行：

```java
RxNewThreadScheduler-1
Begin add!
End add!
Begin add!
End add!
  Add : RxNewThreadScheduler-1 2
  End add : RxNewThreadScheduler-1 2
  Add : RxNewThreadScheduler-1 2
  End add : RxNewThreadScheduler-1 2
```

相反，如果我们像这样调用测试方法：

```java
schedule(Schedulers.newThread(), 2, false);
```

这将为每个*子任务*生成一个新的`Thread`实例，其输出类似于这样：

```java
RxNewThreadScheduler-1
Begin add!
End add!
Begin add!
  Add : RxNewThreadScheduler-2 1
  End add : RxNewThreadScheduler-2 2
End add!
Begin remove!
  Add : RxNewThreadScheduler-3 2
  End add : RxNewThreadScheduler-3 2
End remove!
Begin remove!
End remove!
Begin remove!
  Remove : RxNewThreadScheduler-5
  End remove : RxNewThreadScheduler-5
  Remove : RxNewThreadScheduler-4
  End remove : RxNewThreadScheduler-4
End remove!
```

通过使用*new thread* `Scheduler`实例，您可以执行后台任务。

### 注意

这里非常重要的要求是，其工作人员需要*取消订阅*以避免泄漏线程和操作系统资源。请注意，每次创建新线程都是昂贵的，因此在大多数情况下，应使用*computation*和*IO* `Scheduler`实例。

### Schedulers.computation 调度程序

计算调度程序与*new thread*调度程序非常相似，但它考虑了运行它的机器的处理器/核心数量，并使用可以重用有限数量线程的线程池。每个新的`Worker`实例在其中一个`Thread`实例上安排顺序操作。如果线程当前未被使用，并且它是活动的，则它们将被排队以便稍后执行。

如果我们使用相同的`Worker`实例，我们将只是将所有操作排队到其线程上，并且结果将与使用一个`Worker`实例调度，使用*new thread* `Scheduler`实例相同。

我的机器有四个核心。假设我像这样调用测试方法：

```java
schedule(Schedulers.computation(), 5, false);
```

我会得到类似于这样的输出：

```java
RxComputationThreadPool-1
Begin add!
  Add : RxComputationThreadPool-2 1
  End add : RxComputationThreadPool-2 1
End add!
Begin add!
End add!
Begin add!
  Add : RxComputationThreadPool-3 3
  End add : RxComputationThreadPool-3 3
End add!
Begin add!
  Add : RxComputationThreadPool-4 4
End add!
Begin add!
  End add : RxComputationThreadPool-4 4
End add!
Begin remove!
End remove!
Begin remove!
  Add : RxComputationThreadPool-2 5
  End add : RxComputationThreadPool-2 5
End remove!
Begin remove!
End remove!
Begin remove!
End remove!
Begin remove!
End remove!
Begin remove!
End remove!
Begin remove!
End remove!
Begin remove!
End remove!
Begin remove!
  Remove : RxComputationThreadPool-3
End remove!
Begin remove!
  End remove : RxComputationThreadPool-3
  Remove : RxComputationThreadPool-2
End remove!
Begin remove!
  End remove : RxComputationThreadPool-2
End remove!
Begin remove!
  Remove : RxComputationThreadPool-2
End remove!
Begin remove!
End remove!
Begin remove!
End remove!
Begin remove!
End remove!
Begin remove!
  End remove : RxComputationThreadPool-2
End remove!
  Remove : RxComputationThreadPool-2
Begin remove!
  End remove : RxComputationThreadPool-2
End remove!
  Add : RxComputationThreadPool-1 5
  End add : RxComputationThreadPool-1 5
  Remove : RxComputationThreadPool-1
  End remove : RxComputationThreadPool-1
```

所有内容都是使用来自池中的四个`Thread`实例执行的（请注意，有一种方法可以将`Thread`实例的数量限制为少于可用处理器数量）。

*computation* `Scheduler`实例是执行后台工作 - 计算或处理的最佳选择，因此它的名称。您可以将其用于应该在后台运行且不是*IO*相关或阻塞操作的所有内容。

### Schedulers.io 调度程序

输入输出（IO）调度程序使用`ScheduledExecutorService`实例从*线程池*中检索线程以供其工作人员使用。未使用的线程将被缓存并根据需要重用。如果需要，它可以生成任意数量的线程。

同样，如果我们只使用一个`Worker`实例运行我们的示例，操作将被排队到其线程上，并且其行为将与*computation*和*new thread*调度程序相同。

假设我们使用多个`Worker`实例运行它，如下所示：

```java
schedule(Schedulers.io(), 2, false);
```

它将根据需要从其*池*生成`Thread`实例。结果如下：

```java
RxCachedThreadScheduler-1
Begin add!
End add!
Begin add!
 Add : RxCachedThreadScheduler-2 2
 End add : RxCachedThreadScheduler-2 2
End add!
Begin remove!
 Add : RxCachedThreadScheduler-3 2
 End add : RxCachedThreadScheduler-3 2
End remove!
Begin remove!
 Remove : RxCachedThreadScheduler-4
 End remove : RxCachedThreadScheduler-4
End remove!
Begin remove!
End remove!
Begin remove!
 Remove : RxCachedThreadScheduler-6
 End remove : RxCachedThreadScheduler-6
End remove!

```

*IO*调度程序专用于阻塞*IO 操作*。用于向服务器发出请求，从文件和套接字读取以及其他类似的阻塞任务。请注意，其线程池是无界的；如果其工作人员未取消订阅，则池将无限增长。

### 注意

所有前述代码的源代码位于[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter06/SchedulersTypes.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter06/SchedulersTypes.java)。

### Schedulers.from(Executor)方法

这可以用来创建一个自定义的`Scheduler`实例。如果没有预定义的调度程序适合您，可以使用这个方法，将它传递给`java.util.concurrent.Executor`实例，以实现您需要的行为。

现在我们已经了解了预定义的`Scheduler`实例应该如何使用，是时候看看如何将它们与我们的`Observable`序列集成了。

## 组合 Observable 和调度程序

为了在其他线程上执行我们的可观察逻辑，我们可以使用调度程序。有两个特殊的操作符，它们接收`Scheduler`作为参数，并生成`Observable`实例，能够在与当前线程不同的`Thread`实例上执行操作。

### Observable<T> subscribeOn(Scheduler)方法

`subscribeOn()`方法创建一个`Observable`实例，其`subscribe`方法会导致订阅在从传递的调度程序中检索到的线程上发生。例如，我们有这样的：

```java
Observable<Integer> range = Observable
  .range(20, 4)
  .doOnEach(debug("Source"));
range.subscribe();

System.out.println("Hey!");
```

我们将得到以下输出：

```java
main|Source: >20
main|Source: ->21
main|Source: -->22
main|Source: --->23
main|Source: -------->|
Hey!

```

这是正常的；调用`subscribe`方法会在主线程上执行可观察逻辑，只有在所有这些都完成之后，我们才会看到`'Hey!'`。

让我们修改代码看起来像这样：

```java
CountDownLatch latch = new CountDownLatch(1);
Observable<Integer> range = Observable
  .range(20, 4)
  .doOnEach(debug("Source"))
  .subscribeOn(Schedulers.computation())
  .finallyDo(() -> latch.countDown());
range.subscribe();
System.out.println("Hey!");
latch.await();
```

输出变成了以下内容：

```java
Hey!
RxComputationThreadPool-1|Source: >20
RxComputationThreadPool-1|Source: ->21
RxComputationThreadPool-1|Source: -->22
RxComputationThreadPool-1|Source: --->23
RxComputationThreadPool-1|Source:--------->|

```

这意味着*调用者*线程不会阻塞首先打印`'Hey!'`或在数字之间，所有`Observable`实例的可观察逻辑都在*计算*线程上执行。这样，您可以使用任何您喜欢的调度程序来决定在哪里执行工作。

在这里，我们需要提到`subscribeOn()`方法的一些重要内容。如果您在整个链中多次调用它，就像这样：

```java
CountDownLatch latch = new CountDownLatch(1);
Observable<Integer> range = Observable
  .range(20, 3)
  .doOnEach(debug("Source"))
  .subscribeOn(Schedulers.computation());
Observable<Character> chars = range
  .map(n -> n + 48)
  .map(n -> Character.toChars(n))
  .subscribeOn(Schedulers.io())
  .map(c -> c[0])
  .subscribeOn(Schedulers.newThread())
  .doOnEach(debug("Chars ", "    "))
  .finallyDo(() -> latch.countDown());
chars.subscribe();
latch.await();
```

调用它时*最接近*链的开头很重要。在这里，我们首先在*计算*调度程序上*订阅*，然后在*IO*调度程序上，然后在*新线程*调度程序上，但我们的代码将在*计算*调度程序上执行，因为这在链中*首先*指定。

```java
RxComputationThreadPool-1|Source: >20
RxComputationThreadPool-1|Chars :     >D
RxComputationThreadPool-1|Source: ->21
RxComputationThreadPool-1|Chars :     ->E
RxComputationThreadPool-1|Source: -->22
RxComputationThreadPool-1|Chars :     -->F
RxComputationThreadPool-1|Source: --->|
RxComputationThreadPool-1|Chars :     --->|

```

总之，在生成`Observable`实例的方法中不要指定调度程序；将这个选择留给方法的调用者。或者，使您的方法接收`Scheduler`实例作为参数；例如`Observable.interval`方法。

### 注意

`subscribeOn()`操作符可用于在订阅时阻塞调用者线程的`Observable`实例。在这些源上使用`subscribeOn()`方法让调用者线程与`Observable`实例逻辑并发进行。

那么另一个操作符呢，它帮助我们在其他线程上执行工作呢？

### Observable<T> observeOn(Scheduler)操作符

`observeOn()`操作符类似于`subscribeOn()`操作符，但它不是在传递的`Scheduler`实例上执行整个链，而是从其在其中的位置开始执行链的一部分。通过一个例子最容易理解这一点。让我们使用稍微修改过的前一个例子：

```java
CountDownLatch latch = new CountDownLatch(1);
Observable<Integer> range = Observable
  .range(20, 3)
  .doOnEach(debug("Source"));
Observable<Character> chars = range
  .map(n -> n + 48)
  .doOnEach(debug("+48 ", "    "))
  .map(n -> Character.toChars(n))
  .map(c -> c[0])
  .observeOn(Schedulers.computation())
  .doOnEach(debug("Chars ", "    "))
  .finallyDo(() -> latch.countDown());
chars.subscribe();
System.out.println("Hey!");
latch.await();
```

在这里，我们告诉`Observable`链在订阅后在*主*线程上执行，直到它到达`observeOn()`操作符。在这一点上，它被移动到*计算*调度程序上。这样的输出类似于以下内容：

```java
main|Source: >20
main|+48 :     >68
main|Source: ->21
main|+48 :     ->69
main|Source: -->22
main|+48 :     -->70
RxComputationThreadPool-3|Chars :     >D
RxComputationThreadPool-3|Chars :     ->E
RxComputationThreadPool-3|Chars :     -->F
main|Source: --->|
main|+48 :    --->|
Hey!
RxComputationThreadPool-3|Chars :    --->|

```

正如我们所看到的，调用操作符之前的链部分会阻塞*主*线程，阻止打印`Hey!`。然而，在所有通知通过`observeOn()`操作符之后，`'Hey!'`被打印出来，执行继续在*计算*线程上进行。

如果我们将`observeOn()`操作符移到`Observable`链上，更大部分的逻辑将使用*计算*调度程序执行。

当然，`observeOn()`操作符可以与`subscribeOn()`操作符一起使用。这样，链的一部分可以在一个线程上执行，而其余部分可以在另一个线程上执行（在大多数情况下）。如果你编写客户端应用程序，这是特别有用的，因为通常这些应用程序在一个*事件排队*线程上运行。你可以使用`subscribeOn()`/`observeOn()`操作符使用*IO*调度程序从文件/服务器读取数据，然后在*事件*线程上观察结果。

### 提示

有一个 RxJava 的 Android 模块没有在本书中涵盖，但它受到了很多关注。你可以在这里了解更多信息：[`github.com/ReactiveX/RxJava/wiki/The-RxJava-Android-Module`](https://github.com/ReactiveX/RxJava/wiki/The-RxJava-Android-Module)。

如果你是 Android 开发人员，不要错过它！

**Swing**和**JavaFx**也有类似的模块。

让我们看一个使用`subscribeOn()`和`observeOn()`操作符的示例：

```java
CountDownLatch latch = new CountDownLatch(1);
Observable<Integer> range = Observable
  .range(20, 3)
  .subscribeOn(Schedulers.newThread())
  .doOnEach(debug("Source"));
Observable<Character> chars = range
  .observeOn(Schedulers.io())
  .map(n -> n + 48)
  .doOnEach(debug("+48 ", "    "))
  .observeOn(Schedulers.computation())
  .map(n -> Character.toChars(n))
  .map(c -> c[0])
  .doOnEach(debug("Chars ", "    "))
  .finallyDo(() -> latch.countDown());
chars.subscribe();
latch.await();
```

在这里，我们在链的开头使用了一个`subsribeOn()`操作符的调用（实际上，放在哪里都无所谓，因为它是对该操作符的唯一调用），以及两个`observeOn()`操作符的调用。执行此代码的结果如下：

```java
RxNewThreadScheduler-1|Source: >20
RxNewThreadScheduler-1|Source: ->21
RxNewThreadScheduler-1|Source: -->22
RxNewThreadScheduler-1|Source: --->|
RxCachedThreadScheduler-1|+48 :     >68
RxCachedThreadScheduler-1|+48 :     ->69
RxCachedThreadScheduler-1|+48 :     -->70
RxComputationThreadPool-3|Chars :     >D
RxCachedThreadScheduler-1|+48 :     --->|
RxComputationThreadPool-3|Chars :     ->E
RxComputationThreadPool-3|Chars :     -->F
RxComputationThreadPool-3|Chars :     --->|

```

我们可以看到链通过了三个线程。如果我们使用更多元素，一些代码将看起来是*并行*执行的。结论是，使用`observeOn()`操作符，我们可以多次更改线程；使用`subscribeOn()`操作符，我们可以一次性进行此操作—*订阅*。

### 注意

使用`observeOn()`/`subscribeOn()`操作符的上述示例的源代码可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter06/SubscribeOnAndObserveOn.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter06/SubscribeOnAndObserveOn.java)找到。

使用这两个操作符，我们可以让`Observable`实例和*多线程*一起工作。但是*并发*并不真正意味着我们可以*并行*执行任务。它意味着我们的程序有多个线程，可以独立地取得一些进展。真正的*并行*是当我们的程序以最大限度利用主机机器的 CPU（核心）并且其线程实际上同时运行时。

到目前为止，我们的所有示例都只是将链逻辑移动到其他线程上。尽管有些示例确实在*并行*中执行了部分操作，但真正的*并行*示例看起来是不同的。

## 并行

我们只能通过使用我们已经知道的操作符来实现*并行*。想想`flatMap()`操作符；它为源发出的每个项目创建一个`Observable`实例。如果我们在这些`Observable`实例上使用`subscribeOn()`操作符和`Scheduler`实例，每个实例将在新的`Worker`实例上*调度*，并且它们将*并行*工作（如果主机机器允许）。这是一个例子：

```java
Observable<Integer> range = Observable
  .range(20, 5)
  .flatMap(n -> Observable
    .range(n, 3)
    .subscribeOn(Schedulers.computation())
    .doOnEach(debug("Source"))
  );
range.subscribe();
```

这段代码的输出如下：

```java
RxComputationThreadPool-3|Source: >23
RxComputationThreadPool-4|Source: >20
RxComputationThreadPool-2|Source: >22
RxComputationThreadPool-3|Source: ->24
RxComputationThreadPool-1|Source: >21
RxComputationThreadPool-2|Source: ->23
RxComputationThreadPool-3|Source: -->25
RxComputationThreadPool-3|Source: --->|
RxComputationThreadPool-4|Source: ->21
RxComputationThreadPool-4|Source: -->22
RxComputationThreadPool-4|Source: --->|
RxComputationThreadPool-2|Source: -->24
RxComputationThreadPool-2|Source: --->|
RxComputationThreadPool-1|Source: ->22
RxComputationThreadPool-1|Source: -->23
RxComputationThreadPool-1|Source: --->|
RxComputationThreadPool-4|Source: >24
RxComputationThreadPool-4|Source: ->25
RxComputationThreadPool-4|Source: -->26
RxComputationThreadPool-4|Source: --->|

```

我们可以通过线程的名称看出，通过`flatMap()`操作符定义的`Observable`实例是在*并行*中执行的。这确实是这种情况——四个线程正在使用我的处理器的四个核心。

我将提供另一个示例，这次是对远程服务器进行*并行*请求。我们将使用前一章中定义的`requestJson()`方法。思路是这样的：

1.  我们将检索 GitHub 用户的关注者信息（在本例中，我们将使用我的帐户）。

1.  对于每个关注者，我们将得到其个人资料的 URL。

1.  我们将以*并行*方式请求关注者的个人资料。

1.  我们将打印关注者的数量以及他们的关注者数量。

让我们看看这是如何实现的：

```java
Observable<Map> response = CreateObservable.requestJson(
  client,
  "https://api.github.com/users/meddle0x53/followers"
); // (1)
response
  .map(followerJson -> followerJson.get("url")) // (2)
  .cast(String.class)
  .flatMap(profileUrl -> CreateObservable
    .requestJson(client, profileUrl)
    .subscribeOn(Schedulers.io()) // (3)
    .filter(res -> res.containsKey("followers"))
    .map(json ->  // (4)
      json.get("login") +  " : " +
      json.get("followers"))
  )
  .doOnNext(follower -> System.out.println(follower)) // (5)
  .count() // (6)
  .subscribe(sum -> System.out.println("meddle0x53 : " + sum));
```

在上述代码中发生了什么：

1.  首先，我们对我的用户的关注者数据进行请求。

1.  请求以*JSON*字符串形式返回关注者，这些字符串被转换为`Map`对象（请参阅`requestJson`方法的实现）。从每个*JSON*文件中，读取表示关注者个人资料的 URL。

1.  对每个 URL 执行一个新的请求。请求在*IO*线程上*并行*运行，因为我们使用了与前面示例相同的技术。值得一提的是，`flatMap()`运算符有一个重载，它接受一个`maxConcurrent`整数参数。我们可以使用它来限制并发请求。

1.  获取关注者的用户数据后，生成他/她的关注者的信息。

1.  这些信息作为副作用打印出来。

1.  使用`count()`运算符来计算我的关注者数量（这与`scan(0.0, (sum, element) -> sum + 1).last()`调用相同）。然后我们打印它们。打印的数据顺序不能保证与遍历关注者的顺序相同。

### 注意

前面示例的源代码可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter06/ParallelRequestsExample.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter06/ParallelRequestsExample.java)找到。

这就是*并发*和*并行*的全部内容。一切都很简单，但功能强大。有一些规则（例如使用`Subscribers.io`实例进行阻塞操作，使用*计算*实例进行后台任务等），您必须遵循以确保没有任何问题，即使是*多线程*的可观察链操作。

使用这种*并行*技术很可能会使`Observable`实例链中涌入大量数据，这是一个问题。这就是为什么我们必须处理它。在本章的其余部分，我们将学习如何处理来自*上游*可观察链操作的太多元素。

# 缓冲、节流和去抖动

这里有一个有趣的例子：

```java
Path path = Paths.get("src", "main", "resources");
Observable<String> data = CreateObservable
  .listFolder(path, "*")
  .flatMap(file -> {
    if (!Files.isDirectory(file)) {
      return CreateObservable
    .from(file)
    .subscribeOn(Schedulers.io());
  }
  return Observable.empty();
});
subscribePrint(data, "Too many lines");
```

这将遍历文件夹中的所有文件，并且如果它们本身不是文件夹，则会并行读取它们。例如，当我运行它时，文件夹中有五个文本文件，其中一个文件相当大。在使用我们的`subscribePrint()`方法打印这些文件的内容时，我们得到了类似于这样的内容：

```java
Too many lines : Morbi nec nulla ipsum.
Too many lines : Proin eu tellus tortor.
Too many lines : Lorem ipsum dolor sit am
Error from Too many lines:
rx.exceptions.MissingBackpressureException
Too many lines : Vivamus non vulputate tellus, at faucibus nunc.
Too many lines : Ut tristique, orci eu
Too many lines : Aliquam egestas malesuada mi vitae semper.
Too many lines : Nam vitae consectetur risus, vitae congue risus.
Too many lines : Donec facilisis sollicitudin est non molestie.
 rx.internal.util.RxRingBuffer.onNext(RxRingBuffer.java:349)
 rx.internal.operators.OperatorMerge$InnerSubscriber.enqueue(OperatorMerge.java:721)
 rx.internal.operators.OperatorMerge$InnerSubscriber.emit(OperatorMerge.java:698)
 rx.internal.operators.OperatorMerge$InnerSubscriber.onNext(OperatorMerge.java:586)
 rx.internal.operators.OperatorSubscribeOn$1$1$1.onNext(OperatorSubscribeOn.java:76)

```

输出被裁剪了，但重要的是我们得到了`MissingBackpressureException`异常。

读取每个文件的线程正在尝试将它们的数据推送到`merge()`运算符（`flatMap()`运算符实现为`merge(map(func))`）。该运算符正在努力处理大量数据，因此它将尝试通知过度生产的`Observable`实例减速（通知上游无法处理数据量的能力称为*背压*）。问题在于它们没有实现这样的机制（*背压*），因此会遇到`MissingBackpressureException`异常。

通过在上游可观察对象中实现*背压*，使用其中一个特殊的`onBackpressure*`方法或尝试通过将大量传入的项目打包成更小的发射集来避免它。这种打包是通过*缓冲*、*丢弃*一些传入的项目、*节流*（使用时间间隔或事件进行缓冲）和*去抖动*（使用项目发射之间的间隔进行缓冲）来完成的。

让我们检查其中一些。

## 节流

使用这种机制，我们可以调节`Observable`实例的发射速率。我们可以指定时间间隔或另一个流控制`Observable`实例来实现这一点。

使用`sample()`运算符，我们可以使用另一个`Observable`实例或时间间隔来控制`Observable`实例的发射。

```java
data = data
  .sample(
 Observable
 .interval(100L, TimeUnit.MILLISECONDS)
 .take(10)
 .concatWith(
 Observable
 .interval(200L, TimeUnit.MILLISECONDS)
 )
 );
subscribePrint(data, "Too many lines");
```

*采样* `Observable` 实例在前两秒每 100 毫秒发出一次，然后开始每 200 毫秒发出一次。*data* `Observable` 实例放弃了所有项目，直到 *sampling* 发出。当这种情况发生时，*data* `Observable` 实例发出的最后一个项目被传递。因此，我们有很大的数据丢失，但更难遇到 `MissingBackpressureException` 异常（尽管有可能遇到）。

`sample()` 操作符有两个额外的重载，可以传递时间间隔、`TimeUnit` 度量和可选的 `Scheduler` 实例：

```java
data = data.sample(
 100L,
 TimeUnit.MILLISECONDS
);
```

使用 `sample()` 操作符与 `Observable` 实例可以更详细地控制数据流。`throttleLast()` 操作符只是 `sample()` 操作符的不同版本的别名，它接收时间间隔。`throttleFirst()` 操作符与 `throttleLast()` 操作符相同，但 *source* `Observable` 实例将在间隔开始时发出它发出的第一个项目，而不是最后一个。这些操作符默认在 *computation* 调度程序上运行。

这些技术在有多个相似事件时非常有用（以及本节中的大多数其他技术）。例如，如果您想捕获并对 *鼠标移动事件* 做出反应，您不需要包含所有像素位置的所有事件；您只需要其中一些。

## 防抖动

在我们之前的例子中，*防抖动* 不起作用。它的想法是仅发出在给定时间间隔内没有后续项目的项目。因此，必须在发射之间经过一些时间才能传播一些东西。因为我们 *data* `Observable` 实例中的所有项目似乎一次性发出，它们之间没有可用的间隔。因此，我们需要稍微改变示例以演示这一点。

```java
Observable<Object> sampler = Observable.create(subscriber -> {
  try {
    subscriber.onNext(0);
    Thread.sleep(100L);
    subscriber.onNext(10);
    Thread.sleep(200L);
    subscriber.onNext(20);
    Thread.sleep(150L);
    subscriber.onCompleted();
  }
  catch (Exception e) {
    subscriber.onError(e);
  }
}).repeat()
  .subscribeOn(Schedulers.computation());
data = data
  .sample(sampler)
  .debounce(150L, TimeUnit.MILLISECONDS);
```

在这里，我们使用 `sample()` 操作符与特殊的 *sampling* `Observable` 实例，以便将发射减少到发生在 100、200 和 150 毫秒的时间间隔上。通过使用 `repeat()` 操作符，我们创建了一个重复源的 *无限* `Observable` 实例，并将其设置为在 *computation* 调度程序上执行。现在我们可以使用 `debounce()` 操作符，只发出这组项目，并在它们的发出之间有 150 毫秒或更长的时间间隔。

*防抖动*，像 *节流* 一样，可以用于过滤来自过度生产的源的相似事件。一个很好的例子是自动完成搜索。我们不希望在用户输入每个字母时触发搜索；我们需要等待他/她停止输入，然后触发搜索。我们可以使用 `debounce()` 操作符，并设置一个合理的 *时间间隔*。`debounce()` 操作符有一个重载，它将 `Scheduler` 实例作为其第三个参数。此外，还有一个带有选择器返回 `Observable` 实例的重载，以更精细地控制 *数据流*。

## 缓冲和窗口操作符

这两组操作符与 `map()` 或 `flatMap()` 操作符一样是 *transforming* 操作符。它们将一系列元素转换为一个集合，这些元素的序列将作为一个元素发出。

本书不会详细介绍这些操作符，但值得一提的是，`buffer()` 操作符具有能够基于 *时间间隔*、*选择器* 和其他 `Observable` 实例收集发射的重载。它还可以配置为跳过项目。以下是使用 `buffer(int count, int skip)` 方法的示例，这是 `buffer()` 操作符的一个版本，它收集 *count* 个项目并跳过 *skip* 个项目：

```java
data = data.buffer(2, 3000);
Helpers.subscribePrint(data, "Too many lines");
```

这将输出类似于以下内容：

```java
Too many lines : ["Lorem ipsum dolor sit amet, consectetur adipiscing elit.", "Donec facilisis sollicitudin est non molestie."]
Too many lines : ["Integer nec magna ac ex rhoncus imperdiet.", "Nullam pharetra iaculis sem."]
Too many lines : ["Integer nec magna ac ex rhoncus imperdiet.", "Nullam pharetra iaculis sem."]
Too many lines : ["Nam vitae consectetur risus, vitae congue risus.", "Donec facilisis sollicitudin est non molestie."]
Too many lines : ["Sed mollis facilisis rutrum.", "Proin enim risus, congue id eros at, pharetra consectetur ex."]
Too many lines ended!

```

`window()` 操作符与 `buffer()` 操作符具有完全相同的重载集。不同之处在于，`window()` 操作符创建的 `Observable` 实例发出发出收集的元素的 `Observable` 实例，而不是缓冲元素的数组。

为了演示不同的重载，我们将使用`window(long timespan, long timeshift, TimeUnit units)`方法来举例。该操作符会收集在*timespan*时间间隔内发出的元素，并跳过在*timeshift*时间间隔内发出的所有元素。这将重复，直到源`Observable`实例完成。

```java
data = data
  .window(3L, 200L, TimeUnit.MILLISECONDS)
  .flatMap(o -> o);
subscribePrint(data, "Too many lines");
```

我们使用`flatMap()`操作符来展平`Observable`实例。结果包括在*订阅*的前三毫秒内发出的所有项，以及在 200 毫秒间隔后的三毫秒内发出的项，这将在源发出时重复。

### 注意

在前一节介绍的所有示例都可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter06/BackpressureExamples.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter06/BackpressureExamples.java)找到。

## 背压操作符

最后一组操作符可以防止`MissingBackpressureException`异常，当有一个过度生产的*源*`Observable`实例时，它们会自动激活。

`onBackpressureBuffer()`操作符会对由快于其`Observer`实例的*源*`Observable`发出的项进行缓冲。然后以订阅者可以处理的方式发出缓冲的项。例如：

```java
Helpers.subscribePrint(
  data.onBackpressureBuffer(10000),
  "onBackpressureBuffer(int)"
);
```

在这里，我们使用了一个大容量的缓冲区，因为元素数量很大，但请注意，溢出此缓冲区将导致`MissingBackpressureException`异常。

`onBackpressureDrop()`操作符会丢弃所有来自*源*`Observable`实例的无法被订阅者处理的传入项。

有一种方法可以通过实现智能的 Observables 或 Subscribers 来建立*背压*，但这个话题超出了本书的范围。在 RxJava 维基页面上有一篇关于*背压*和 observable 的优秀文章—[`github.com/ReactiveX/RxJava/wiki/Backpressure`](https://github.com/ReactiveX/RxJava/wiki/Backpressure)。本节中提到的许多操作符在那里都有详细描述，并且有大理石图可用于帮助您理解更复杂的操作符。

# 总结

在本章中，我们学习了如何在与*主*线程不同的其他线程上执行我们的 observable 逻辑。有一些简单的规则和技术可以做到这一点，如果一切都按照规定进行，就不应该有危险。使用这些技术，我们能够编写*并发*程序。我们还学习了如何使用调度程序和`flatMap()`操作符实现*并行*执行，并且看到了一个真实世界的例子。

我们还研究了如何处理*过度生产*的数据源。有许多操作符可以通过不同的方式来做到这一点，我们介绍了其中一些，并讨论了它们的有用性。

有了这些知识，我们可以编写任意的 RxJava 程序，能够处理来自不同源的数据。我们知道如何使用多个线程来做到这一点。使用 RxJava、它的操作符和*构造*几乎就像使用一种新语言编码。它有自己的规则和流程控制方法。

为了编写稳定的应用程序，我们必须学会如何对它们进行*单元测试*。测试*异步*代码并不是一件容易的事情。好消息是，RxJava 提供了一些操作符和类来帮助我们做到这一点。您可以在下一章中了解更多信息。
