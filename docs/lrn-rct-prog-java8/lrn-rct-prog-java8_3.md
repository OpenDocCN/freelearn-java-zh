# 第三章。创建和连接 Observables、Observers 和 Subjects

RxJava 的 `Observable` 实例是响应式应用程序的构建模块，这是 RxJava 的优势。如果我们有一个源 `Observable` 实例，我们可以将逻辑链接到它并订阅结果。我们只需要这个初始的 `Observable` 实例。

在浏览器或桌面应用程序中，用户输入已经被表示为我们可以处理并通过 `Observable` 实例转发的事件。但是将所有数据更改或操作转换为 `Observable` 实例会很好，而不仅仅是用户输入。例如，当我们从文件中读取数据时，将每一行读取或每个字节序列视为可以通过 `Observable` 实例发出的消息将会很好。

我们将详细了解如何将不同的数据源转换为 `Observable` 实例；无论它们是外部的（文件或用户输入）还是内部的（集合或标量）都无关紧要。此外，我们将了解各种类型的 `Observable` 实例，取决于它们的行为。另一个重要的是我们将学习如何何时取消订阅 `Observable` 实例以及如何使用订阅和 `Observer` 实例。此外，我们还将介绍 Subject 类型及其用法。

在本章中，我们将学习以下内容：

+   `Observable` 工厂方法——`just`、`from`、`create` 等

+   观察者和订阅者

+   热和冷 Observable；可连接的 Observable

+   主题是什么以及何时使用它们

+   `Observable` 创建

有很多种方法可以从不同的来源创建 `Observable` 实例。原则上，可以使用 `Observable.create(OnSubscribe<T>)` 方法创建 `Observable` 实例，但是有许多简单的方法，旨在让我们的生活更美好。让我们来看看其中一些。

# Observable.from 方法

`Observable.from` 方法可以从不同的 Java 结构创建 `Observable` 实例。例如：

```java
List<String> list = Arrays.asList(
  "blue", "red", "green", "yellow", "orange", "cyan", "purple"
);
Observable<String> listObservable = Observable.from(list);
listObservable.subscribe(System.out::println);
```

这段代码从 `List` 实例创建了一个 `Observable` 实例。当在 `Observable` 实例上调用 `subscribe` 方法时，源列表中包含的所有元素都将被发射到订阅方法中。对于每次调用 `subscribe()` 方法，整个集合都会从头开始逐个元素发射：

```java
listObservable.subscribe(
  color -> System.out.print(color + "|"),
  System.out::println,
  System.out::println
);
listObservable.subscribe(color -> System.out.print(color + "/"));
```

这将以不同的格式两次打印颜色。

这个版本的 `from` 方法的真正签名是 `final static <T> Observable<T> from(Iterable<? extends T> iterable)`。这意味着可以将实现 `Iterable` 接口的任何类的实例传递给这个方法。这些包括任何 Java 集合，例如：

```java
Path resources = Paths.get("src", "main", "resources");
try (DirectoryStream<Path> dStream =Files.newDirectoryStream(resources)) {
  Observable<Path> dirObservable = Observable.from(dStream);
  dirObservable.subscribe(System.out::println);
}
catch (IOException e) {
  e.printStackTrace();
}
```

这将把文件夹的内容转换为我们可以订阅的事件。这是可能的，因为 `DirectoryStream` 参数是一个 `Iterable` 实例。请注意，对于此 `Observable` 实例的每次调用 `subscribe` 方法，它的 `Iterable` 源的 `iterator()` 方法都会被调用以获取一个新的 `Iterator` 实例，用于从头开始遍历数据。使用此示例，第二次调用 `subscribe()` 方法时将抛出 `java.lang.IllegalStateException` 异常，因为 `DirectoryStream` 参数的 `iterator()` 方法只能被调用一次。

用于从数组创建 `Observable` 实例的 `from` 方法的另一个重载是 `public final static <T> Observable<T> from(T[] array)`，使用 `Observable` 实例的示例如下：

```java
Observable<Integer> arrayObservable = Observable.from(new Integer[] {3, 5, 8});
  arrayObservable.subscribe(System.out::println);
```

`Observable.from()` 方法非常有用，可以从集合或数组创建 `Observable` 实例。但是有些情况下，我们需要从单个对象创建 `Observable` 实例；对于这些情况，可以使用 `Observable.just()` 方法。

### 注意

使用`Observable.from()`方法的示例的源代码可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter03/CreatingObservablesWithFrom.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter03/CreatingObservablesWithFrom.java)中查看和下载。

# Observable.just 方法

`just()`方法将其参数作为`OnNext`通知发出，然后发出`OnCompleted`通知。

例如，一个字母：

```java
Observable.just('S').subscribe(System.out::println);
```

或者一系列字母：

```java
Observable
  .just('R', 'x', 'J', 'a', 'v', 'a')
  .subscribe(
    System.out::print,
    System.err::println,
    System.out::println
  );
```

第一段代码打印`S`和一个换行，第二段代码打印字母并在完成时添加一个换行。该方法允许通过响应式手段观察最多九个任意值（相同类型的对象）。例如，假设我们有这个简单的`User`类：

```java
public static class User {
  private final String forename;
  private final String lastname;
  public User(String forename, String lastname) {
    this.forename = forename;
    this.lastname = lastname;
  }
  public String getForename() {
    return this.forename;
  }
  public String getLastname() {
    return this.lastname;
  }
}
```

我们可以这样打印`User`实例的全名：

```java
Observable
  .just(new User("Dali", "Bali"))
  .map(u -> u.getForename() + " " + u.getLastname())
  .subscribe(System.out::println);
```

这并不是非常实用，但展示了将数据放入`Observable`实例上下文并利用`map()`方法的方法。一切都可以成为一个事件。

还有一些更方便的工厂方法，可在各种情况下使用。让我们在下一节中看看它们。

### 注意

`Observable.just()`方法示例的源代码可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter03/CreatingObservablesUsingJust.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter03/CreatingObservablesUsingJust.java)中查看/下载。

# 其他 Observable 工厂方法

在这里，我们将检查一些可以与转换操作符（如 flatMap）或组合操作符（如`.zip`文件）结合使用的方法（有关更多信息，请参见下一章）。

为了检查它们的结果，我们将使用以下方法创建订阅：

```java
void subscribePrint(Observable<T> observable, String name) {
  observable.subscribe(
    (v) -> System.out.println(name + " : " + v),
    (e) -> {
      System.err.println("Error from " + name + ":");
      System.err.println(e.getMessage());
    },
    () -> System.out.println(name + " ended!")
  );
}
```

前面方法的想法是*订阅*一个`Observable`实例并用名称标记它。在*OnNext*时，它打印带有名称前缀的值；在*OnError*时，它与名称一起打印错误；在*OnCompleted*时，它打印带有名称前缀的`'ended!'`。这有助于我们调试结果。

### 注意

前面方法的源代码可以在[`github.com/meddle0x53/learning-rxjava/blob/4a2598aa0835235e6ef3bc3371a3c19896161628/src/main/java/com/packtpub/reactive/common/Helpers.java#L25`](https://github.com/meddle0x53/learning-rxjava/blob/4a2598aa0835235e6ef3bc3371a3c19896161628/src/main/java/com/packtpub/reactive/common/Helpers.java#L25)找到。

以下是介绍新工厂方法的代码：

```java
subscribePrint(
  Observable.interval(500L, TimeUnit.MILLISECONDS),
  "Interval Observable"
);
subscribePrint(
  Observable.timer(0L, 1L, TimeUnit.SECONDS),
  "Timed Interval Observable"
);
subscribePrint(
  Observable.timer(1L, TimeUnit.SECONDS),
  "Timer Observable"
);

subscribePrint(
  Observable.error(new Exception("Test Error!")),
  "Error Observable"
);
subscribePrint(Observable.empty(), "Empty Observable");
subscribePrint(Observable.never(), "Never Observable");
subscribePrint(Observable.range(1, 3), "Range Observable");
Thread.sleep(2000L);
```

以下是代码中发生的情况：

+   `Observable<Long> Observable.interval(long, TimeUnit, [Scheduler])`：此方法创建一个`Observable`实例，将以给定间隔发出顺序数字。它可用于实现周期性轮询，或通过仅忽略发出的数字并发出有用消息来实现连续状态记录。该方法的特殊之处在于，默认情况下在*计算线程*上运行。我们可以通过向方法传递第三个参数——`Scheduler`实例（有关`Scheduler`实例的更多信息，请参见第六章, *使用调度程序进行并发和并行处理*）来更改这一点。

+   `Observable<Long> Observable.timer(long, long, TimeUnit, [Scheduler])`：`interval()`方法仅在等待指定时间间隔后开始发出数字。如果我们想要告诉它在何时开始工作，可以使用此`timer()`方法。它的第一个参数是开始时间，第二个和第三个是间隔设置。同样，默认情况下在*计算线程*上执行，同样，这是可配置的。

+   `Observable<Long> Observable.timer(long, TimeUnit, [Scheduler])`：这个在*计算线程*（默认情况下）上在一定时间后只发出输出`'0'`。之后，它发出一个*completed*通知。

+   `<T> Observable<T> Observable.error(Throwable)`：这只会将传递给它的错误作为*OnError*通知发出。这类似于经典的命令式 Java 世界中的`throw`关键字。

+   `<T> Observable<T> Observable.empty()`：这个不发出任何项目，但立即发出一个`OnCompleted`通知。

+   `<T> Observable<T> Observable.never()`：这个什么都不做。它不向其`Observer`实例发送任何通知，甚至`OnCompleted`通知也不发送。

+   `Observable<Integer>` `Observable.range(int, int, [Scheduler])`：此方法从传递的第一个参数开始发送顺序数字。第二个参数是发射的数量。

这个程序将打印以下输出：

```java
Timed Interval Observable : 0
Error from Error Observable:
Test Error!
Range Observable : 1
Range Observable : 2
Range Observable : 3
Range Observable ended!
Empty Observable ended!
Interval Observable : 0
Interval Observable : 1
Timed Interval Observable : 1
Timer Observable : 0
Timer Observable ended!
Interval Observable : 2
Interval Observable : 3
Timed Interval Observable : 2

```

正如你所看到的，`interval Observable`实例不会发送*OnCompleted*通知。程序在两秒后结束，`interval Observable`实例在 500 毫秒后开始发出，每 500 毫秒发出一次；因此，它发出了三个*OnNext*通知。`timed interval Observable`实例在创建后立即开始发出，每秒发出一次；因此，我们从中得到了两个通知。

### 注意

前面示例的源代码可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter03/CreatingObservablesUsingVariousFactoryMethods.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter03/CreatingObservablesUsingVariousFactoryMethods.java)上查看/下载。

所有这些方法都是使用`Observable.create()`方法实现的。

# Observable.create 方法

让我们首先看一下该方法的签名：

```java
public final static <T> Observable<T> create(OnSubscribe<T>)
```

它接受一个`OnSubscribe`类型的参数。这个接口扩展了`Action1<Subscriber<? super T>>`接口；换句话说，这种类型只有一个方法，接受一个`Subscriber<T>`类型的参数并返回空。每次调用`Observable.subscribe()`方法时，都会调用此函数。它的参数，`Subscriber`类的一个实例，实际上是观察者，订阅`Observable`实例（这里，`Subscriber`类和 Observer 接口扮演相同的角色）。我们将在本章后面讨论它们。我们可以在其上调用`onNext()`、`onError()`和`onCompleted()`方法，实现我们自己的自定义行为。

通过一个例子更容易理解。让我们实现`Observable.from(Iterabale<T>)`方法的一个简单版本：

```java
<T> Observable<T> fromIterable(final Iterable<T> iterable) {
  return Observable.create(new OnSubscribe<T>() {
    @Override
    public void call(Subscriber<? super T> subscriber) {
      try {
        Iterator<T> iterator = iterable.iterator(); // (1)
        while (iterator.hasNext()) { // (2)
          subscriber.onNext(iterator.next());
        }
        subscriber.onCompleted(); // (3)
      }
      catch (Exception e) {
        subscriber.onError(e); // (4)
      }
    }
  });
}
```

该方法以一个`Iterable<T>`参数作为参数，并返回一个`Observable<T>`参数。行为如下：

1.  当一个`Observer/Subscriber`实例订阅生成的`Observable`实例时，会从`Iterable`源中检索一个`Iterator`实例。`Subscriber`类实际上实现了`Observer`接口。它是一个抽象类，`on*`方法不是由它实现的。

1.  当有元素时，它们作为`OnNext`通知被发送。

1.  当所有元素都被发出时，将发送一个`OnCompleted`通知。

1.  如果在任何时候发生错误，将会发送一个`OnError`通知与错误。

这是`Observable.from(Iterable<T>)`方法行为的一个非常简单和天真的实现。第一章和第二章中描述的 Reactive Sum 是`Observable.create`方法的另一个例子（由`CreateObservable.from()`使用）。

但正如我们所看到的，传递给`create()`方法的逻辑是在`Observable.subscribe()`方法在`Observable`实例上被调用时触发的。到目前为止，我们一直在创建`Observable`实例并使用这种方法*订阅*它们。现在是时候仔细看一下了。

# 订阅和取消订阅

Observable.subscribe()方法有许多重载，如下所示：

+   `subscribe()`: 这个方法忽略来自 Observable 实例的所有发射，并且如果有 OnError 通知，则抛出一个 OnErrorNotImplementedException 异常。这可以用来触发`OnSubscribe.call`行为。

+   `subscribe(Action1<? super T>)`: 这只订阅`onNext()`方法触发的更新。它忽略`OnCompleted`通知，并且如果有`OnError`通知，则抛出一个 OnErrorNotImplementedException 异常。这不是真正的生产代码的好选择，因为很难保证不会抛出错误。

+   `subscribe(Action1<? super T>, Action1<Throwable>)`: 这与前一个方法相同，但如果有`OnError`通知，则调用第二个参数。

+   `subscribe(Action1<? super T>,Action1<Throwable>, Action0)`: 这与前一个方法相同，但第三个参数在`OnCompleted`通知时被调用。

+   `subscribe(Observer<? super T>)`: 这使用其 Observer 参数的`onNext/onError/onCompleted`方法来观察 Observable 实例发出的通知。我们在第一章中实现"响应式求和"时使用了这个方法。

+   `subscribe(Subscriber<? super T>)`: 这与前一个方法相同，但使用 Observer 接口的 Subscriber 实现来观察通知。Subscriber 类提供了高级功能，如取消订阅（取消）和背压（流量控制）。实际上，所有前面的方法都调用这个方法；这就是为什么我们从现在开始谈论`Observable.subscribe`时将引用它。该方法确保传递的 Subscriber 实例看到一个 Observable 实例，符合以下**Rx contract**：

> *"发送到 Observer 接口实例的消息遵循以下语法：*
> 
> *onNext* (onCompleted | onError)?*
> 
> *这种语法允许可观察序列向 Subscriber 发送任意数量（0 个或更多个）的`OnNext()`方法消息，可选地跟随单个成功（`onCompleted`）或失败（`onError`）消息。指示可观察序列已完成的单个消息确保可观察序列的消费者可以确定地建立安全执行清理操作。单个失败进一步确保可以维护对多个可观察序列进行操作的操作符的中止语义。*

- RxJava 的 JavaDoc 的一部分。

这是通过在传递的 Subscriber 实例周围使用一个包装器——SafeSubscriber 来内部完成的。

+   `unsafeSubscribe(Subscriber<? super T>)`: 这与前一个方法相同，但没有**Rx contract**保护。它旨在帮助实现自定义操作符（参见第八章，“资源管理和扩展 RxJava”），而不会增加`subscribe()`方法的额外开销；在一般代码中使用这种方法观察 Observable 实例是不鼓励的。

所有这些方法返回 Subscription 类型的结果，可以用于从 Observable 实例发出的通知中*取消订阅*。取消订阅通常会清理与订阅相关的内部资源；例如，如果我们使用`Observable.create()`方法实现一个 HTTP 请求，并希望在特定时间取消它，或者我们有一个发射无限序列的数字/单词/任意数据的 Observable 实例，并希望停止它。

Subscription 接口有两个方法：

+   `void unsubscribe()`: 这用于*取消订阅*。

+   `boolean isUnsubscribed()`: 这用于检查 Subscription 实例是否已经*取消订阅*。

传递给`Observable.create()`方法的`OnSubscribe()`方法的`Subscriber`类的实例实现了`Subscription`接口。因此，在编写`Observable`实例的行为时，可以进行*取消订阅*和检查`Subscriber`是否已订阅。让我们更新我们的`Observable<T> fromIterable(Iterable<T>)`方法的实现以对*取消订阅*做出反应：

```java
<T> Observable<T> fromIterable(final Iterable<T> iterable) {
  return Observable.create(new OnSubscribe<T>() {
    @Override
    public void call(Subscriber<? super T> subscriber) {
      try {
        Iterator<T> iterator = iterable.iterator();
        while (iterator.hasNext()) {
          if (subscriber.isUnsubscribed()) {
 return;
 }
          subscriber.onNext(iterator.next());
        }
        if (!subscriber.isUnsubscribed()) {
 subscriber.onCompleted();
 }
 }
 catch (Exception e) {
 if (!subscriber.isUnsubscribed()) {
 subscriber.onError(e);
 }
 }
    }
  });
}
```

新的地方在于`Subscription.isUnsubscribed()`方法用于确定是否应终止数据发射。我们在每次迭代时检查`Subscriber`是否已*取消订阅*，因为它可以随时*取消订阅*，之后我们将不需要再发出任何内容。在发出所有内容之后，如果 Subscriber 已经*取消订阅*，则会跳过`onCompleted()`方法。如果有异常，则只有在`Subscriber`实例仍然*订阅*时才会作为`OnError`通知发出。

让我们看看*取消订阅*是如何工作的：

```java
Path path = Paths.get("src", "main", "resources", "lorem_big.txt"); // (1)
List<String> data = Files.readAllLines(path);
Observable<String> observable = fromIterable(data).subscribeOn(Schedulers.computation()); // (2)
Subscription subscription = subscribePrint(observable, "File");// (3)
System.out.println("Before unsubscribe!");
System.out.println("-------------------");
subscription.unsubscribe(); // (4)
System.out.println("-------------------");
System.out.println("After unsubscribe!");
```

以下是这个例子中发生的事情：

1.  数据源是一个巨大的文件，因为我们需要一些需要一些时间来迭代的东西。

1.  `Observable`实例的所有订阅将在另一个*线程*上进行，因为我们希望在主线程上*取消订阅*。

1.  在本章中定义的`subscribePrint()`方法被使用，但已修改为返回`Subscription`。

1.  订阅用于从`Observable`实例*取消订阅*，因此整个文件不会被打印，并且会显示*取消订阅*执行的标记。

输出将类似于这样：

```java
File : Donec facilisis sollicitudin est non molestie.
File : Integer nec magna ac ex rhoncus imperdiet.
Before unsubscribe!
-------------------
File : Nullam pharetra iaculis sem.
-------------------
After unsubscribe!

```

大部分文件内容被跳过。请注意，可能会在*取消订阅*后立即发出某些内容；例如，如果`Subscriber`实例在检查*取消订阅*后立即*取消订阅*，并且程序已经执行`if`语句的主体，则会发出内容。

### 注意

前面示例的源代码可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter03/ObservableCreateExample.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter03/ObservableCreateExample.java)中下载/查看。

还要注意的一点是，`Subscriber`实例有一个`void add(Subscription s)`方法。当`Subscriber`*取消订阅*时，传递给它的每个订阅将自动*取消订阅*。这样，我们可以向`Subscriber`实例添加额外的操作；例如，在*取消订阅*时应执行的操作（类似于 Java 中的 try-finally 结构）。这就是*取消订阅*的工作原理。在第八章中，我们将处理资源管理。我们将学习如何通过`Subscription`包装器将`Observable`实例附加到`Subscriber`实例，并且调用*取消订阅*将释放任何分配的资源。

在本章中，我们将讨论与订阅行为相关的下一个主题。我们将谈论热和冷的`Observable`实例。

# 热和冷的 Observable 实例

查看使用`Observable.create()`、`Observable.just()`和`Observable.from()`方法实现的先前示例时，我们可以说在有人订阅它们之前，它们是不活动的，不会发出任何内容。但是，每次有人订阅时，它们就开始发出它们的通知。例如，如果我们对`Observable.from(Iterable)`对象进行三次订阅，`Iterable`实例将被迭代*三*次。像这样行为的`Observable`实例被称为冷的 Observable 实例。

在本章中我们一直在使用的所有工厂方法返回冷的 Observables。冷的 Observables 按需产生通知，并且对于每个 Subscriber，它们产生*独立*的通知。

有些`Observable`实例在开始发出通知时，无论是否有订阅，都会继续发出通知，直到完成。所有订阅者都会收到相同的通知，默认情况下，当一个订阅者*订阅*时，它不会收到之前发出的通知。这些是热 Observable 实例。

我们可以说，冷 Observables 为每个订阅者生成通知，而热 Observables 始终在运行，向所有订阅者广播通知。把热 Observable 想象成一个广播电台。此刻收听它的所有听众都在听同一首歌。冷 Observable 就像一张音乐 CD。许多人可以购买并独立听取它。

正如我们提到的，本书中有很多使用冷 Observables 的例子。那么热 Observable 实例呢？如果你还记得我们在第一章中实现'响应式求和'时，我们有一个`Observable`实例，它会发出用户在标准输入流中输入的每一行。这个是热的，并且我们从中派生了两个`Observable`实例，一个用于收集器`a`，一个用于`b`。它们接收相同的输入行，并且只过滤出它们感兴趣的行。这个输入`Observable`实例是使用一种特殊类型的`Observable`实现的，称为`ConnectableObservable`。

## ConnectableObservable 类

这些`Observable`实例在调用它们的`connect()`方法之前是不活跃的。之后，它们就变成了热 Observables。可以通过调用其`publish()`方法从任何`Observable`实例创建`ConnectableObservable`实例。换句话说，`publish()`方法可以将任何冷 Observable 转换为热 Observable。让我们看一个例子：

```java
Observable<Long> interval = Observable.interval(100L, TimeUnit.MILLISECONDS);
ConnectableObservable<Long> published = interval.publish();
Subscription sub1 = subscribePrint(published, "First");
Subscription sub2 = subscribePrint(published, "Second");
published.connect();
Subscription sub3 = null;
try {
  Thread.sleep(500L);
  sub3 = subscribePrint(published, "Third");
  Thread.sleep(500L);
}
catch (InterruptedException e) {}
sub1.unsubscribe();
sub2.unsubscribe();
sub3.unsubscribe();
```

在调用`connect()`方法之前什么都不会发生。之后，我们将看到相同的顺序数字输出两次——每个订阅者一次。第三个订阅者将加入其他两个，打印在第一个 500 毫秒后发出的数字，但它不会打印其订阅之前发出的数字。

如果我们想要在我们的订阅之前接收*所有*已发出的通知，然后继续接收即将到来的通知，可以通过调用`replay()`方法而不是`publish()`方法来实现。它从源`Observable`实例创建一个`ConnectableObservable`实例，有一个小变化：所有订阅者在订阅时都会收到*所有*通知（之前的通知将按顺序同步到达）。

有一种方法可以激活`Observable`实例，使其在不调用`connect()`方法的情况下变为热 Observable。它可以在*第一次订阅*时激活，并在每个`Subscriber`实例*取消订阅*时停用。可以通过在`ConnectableObservable`实例上调用`refCount()`方法（方法的名称来自'引用计数'；它计算订阅到由它创建的`Observable`实例的`Subscriber`实例数量）从`ConnectableObservable`实例创建这样的`Observable`实例。以下是使用`refCount()`方法实现的前面的例子：

```java
Observable<Long> refCount = interval.publish().refCount();
Subscription sub1 = subscribePrint(refCount, "First");
Subscription sub2 = subscribePrint(refCount, "Second");
try {
  Thread.sleep(300L);
}
catch (InterruptedException e) {}
sub1.unsubscribe();
sub2.unsubscribe();
Subscription sub3 = subscribePrint(refCount, "Third");
try {
  Thread.sleep(300L);
}
catch (InterruptedException e) { }
sub3.unsubscribe();
```

`sub2` *取消订阅*后，`Observable`实例将停用。如果此后有人*订阅*它，它将从头开始发出序列。这就是`sub3`的情况。还有一个`share()`方法，它是`publish().refCount()`调用的别名。

### 注意

前面例子的源代码可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter03/UsingConnectableObservables.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter03/UsingConnectableObservables.java)上查看/下载。

还有一种创建热 Observable 的方法：使用`Subject`实例。我们将在本章的下一节和最后一节介绍它们。

# Subject 实例

`Subject`实例既是`Observable`实例又是`Observer`实例。与`Observable`实例一样，它们可以有多个`Observer`实例，接收相同的通知。这就是为什么它们可以用来将冷的`Observable`实例转换为热的实例。与`Observer`实例一样，它们让我们访问它们的`onNext()`、`onError()`或`onCompleted()`方法。

让我们看一下使用`Subject`实例实现前面的热间隔示例：

```java
Observable<Long> interval = Observable.interval(100L, TimeUnit.MILLISECONDS); // (1)
Subject<Long, Long> publishSubject = PublishSubject.create(); // (2)
interval.subscribe(publishSubject);
// (3)
Subscription sub1 = subscribePrint(publishSubject, "First");
Subscription sub2 = subscribePrint(publishSubject, "Second");
Subscription sub3 = null;
try {
  Thread.sleep(300L);
  publishSubject.onNext(555L); // (4)
  sub3 = subscribePrint(publishSubject, "Third"); // (5)
  Thread.sleep(500L);
}
catch (InterruptedException e) {}
sub1.unsubscribe(); // (6)
sub2.unsubscribe();
sub3.unsubscribe();
```

现在示例略有不同：

1.  间隔`Observable`实例的创建方式与以前相同。

1.  在这里，我们创建了一个`PublishSubject`实例 - 一个`Subject`实例，只向订阅后由源`Observable`实例发出的项目发出。这种行为类似于使用`publish()`方法创建的`ConnectableObservable`实例。新的`Subject`实例订阅了由间隔工厂方法创建的间隔`Observable`实例，这是可能的，因为`Subject`类实现了`Observer`接口。还要注意，`Subject`签名有两种泛型类型 - 一种是`Subject`实例将接收的通知类型，另一种是它将发出的通知类型。`PublishSubject`类的输入和输出通知类型相同。

请注意，可以创建一个`PublishSubject`实例而不订阅源`Observable`实例。它只会发出传递给其`onNext()`和`onError()`方法的通知，并在调用其`onCompleted()`方法时完成。

1.  我们可以订阅`Subject`实例；毕竟它是一个`Observable`实例。

1.  我们可以随时发出自定义通知。它将广播给主题的所有订阅者。我们甚至可以调用`onCompleted()`方法并关闭通知流。

1.  第三个订阅者只会收到订阅后发出的通知。

1.  当一切都取消订阅时，`Subject`实例将继续发出。

### 注意

此示例的源代码可在[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter03/SubjectsDemonstration.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter03/SubjectsDemonstration.java)上查看/下载。

RxJava 有四种类型的主题：

+   `PublishSubject`：这是我们在前面的示例中看到的，行为类似于使用`publish()`方法创建的`ConnectableObservable`。

+   `ReplaySubject`：这会向任何观察者发出源`Observable`实例发出的所有项目，无论观察者何时订阅。因此，它的行为类似于使用`replay()`方法创建的`ConnectableObservable`。`ReplaySubject`类有许多工厂方法。默认的工厂方法会缓存所有内容；请记住这一点，因为它可能会占用内存。有用于使用大小限制和/或时间限制缓冲区创建它的工厂方法。与`PublishSubject`类一样，这个可以在没有源`Observable`实例的情况下使用。使用其`onNext()`、`onError()`和`onCompleted()`方法发出的所有通知都将发送给每个订阅者，即使在调用`on*`方法后订阅。

+   `BehaviorSubject`：当观察者订阅它时，它会发出源`Observable`实例最近发出的项目（如果尚未发出任何项目，则发出种子/默认值），然后继续发出源`Observable`实例后来发出的任何其他项目。`BehaviorSubject`类几乎与具有缓冲区大小为一的`ReplaySubjects`类相似。`BehaviorSubject`类可用于实现有状态的响应实例 - 一个响应属性。再次强调，不需要源`Observable`实例。

+   `AsyncSubject`：这会发出源`Observable`实例发出的最后一个值（仅此一个），并且只有在源`Observable`实例完成后才会发出。如果源`Observable`实例没有发出任何值，`AsyncSubject`实例也会在不发出任何值的情况下完成。这在 RxJava 的世界中有点像*promise*。不需要源`Observable`实例；可以通过调用`on*`方法将值、错误或`OnCompleted`通知传递给它。

使用主题可能看起来是解决各种问题的一种很酷的方式，但你应该避免使用它们。或者，至少要在返回`Observable`类型的结果的方法中实现它们和它们的行为。

`Subject`实例的危险在于它们提供了`onNext()`，`onError()`和`onCompleted()`方法的访问权限，你的逻辑可能会变得混乱（它们需要遵循本章前面引用的 Rx 合同）。它们很容易被滥用。

在需要从冷 Observable 创建热 Observable 时，选择使用`ConnecatableObservable`实例（即通过`publish()`方法）而不是`Subject`。

但让我们看一个`Subject`实例的一个很好的用法——前面提到的*反应性属性*。同样，我们将实现*'The Reactive Sum'*，但这次会有很大不同。以下是定义它的类：

```java
public class ReactiveSum { // (1)
  private BehaviorSubject<Double> a = BehaviorSubject.create(0.0);
 private BehaviorSubject<Double> b = BehaviorSubject.create(0.0);
 private BehaviorSubject<Double> c = BehaviorSubject.create(0.0);
  public ReactiveSum() { // (2)
    Observable.combineLatest(a, b, (x, y) -> x + y).subscribe(c);
  }
  public double getA() { // (3)
    return a.getValue();
  }
  public void setA(double a) {
    this.a.onNext(a);
  }
  public double getB() {
    return b.getValue();
  }
  public void setB(double b) {
    this.b.onNext(b);
  }
  public double getC() { // (4)
    return c.getValue();
  }
  public Observable<Double> obsC() {
    return c.asObservable();
  }
}
```

这个类有三个双精度属性：两个可设置的属性`a`和`b`，以及它们的*和*，`c`。当`a`或`b`改变时，`c`会*自动更新*为它们的和。我们可以使用一种特殊的方法来跟踪`c`的变化。那它是如何工作的呢？

1.  `ReactiveSum`是一个普通的 Java 类，定义了三个`BehaviorSubject<Double>`类型的私有字段，表示变量`a`，`b`和`c`，默认值为零。

1.  在构造函数中，我们订阅`c`依赖于`a`和`b`，并且等于它们的和，再次使用`combineLatest()`方法。

1.  属性`a`和`b`有 getter 和 setter。getter 返回它们当前的值——最后接收到的值。setter 将传递的值*发出*到它们的`Subject`实例，使其成为最后一个。

### 注意

`BehaviorSubject`参数的`getValue()`方法用于检索它。它在 RxJava 1.0.5 中可用。

1.  属性`c`是只读的，所以它只有一个 getter，但可以被监听。这可以通过`obsC()`方法来实现，它将其作为`Observable`实例返回。记住，当你使用主题时，要始终将它们封装在类型或方法中，并将可观察对象返回给外部世界。

这个`ReactiveSum`类可以这样使用：

```java
ReactiveSum sum = new ReactiveSum();
subscribePrint(sum.obsC(), "Sum");
sum.setA(5);
sum.setB(4);
```

这将输出以下内容：

```java
Sum : 0.0
Sum : 5.0
Sum : 9.0

```

第一个值在`subscribe` `()`方法上*发出*（记住`BehaviorSubject`实例总是在订阅时*发出*它们的最后一个值），其他两个将在设置`a`或`b`时自动*发出*。

### 注意

前面示例的源代码可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter03/ReactiveSumV3.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter03/ReactiveSumV3.java)上查看/下载。

*反应性属性*可用于实现绑定和计数器，因此它们对于桌面或浏览器应用程序非常有用。但这个例子远离了任何功能范式。

# 总结

在本章中，我们学习了许多创建不同类型的`Observable`实例和其他相关实例（`Observer`，`Subscriber`，`Subscription`和`Subject`）的方法。我们已经从计时器，值，集合和文件等外部来源创建了它们。利用这些知识作为基础，我们可以开始通过对它们进行操作来构建逻辑。这里介绍的许多工厂方法将在接下来的章节中再次出现。例如，我们将使用`Observable.create`方法构建不同的行为。

在下一章中，我们将介绍各种**操作符**，这将赋予我们使用`Observable`实例编写真正逻辑的能力。我们已经提到了其中一些，比如`map()`和`filter()`，但现在是时候深入研究它们了。
