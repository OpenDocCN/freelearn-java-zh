# 第七章。测试您的 RxJava 应用程序

在编写软件时，尤其是将被许多用户使用的软件，我们需要确保一切都正常运行。我们可以编写可读性强、结构良好、模块化的代码，这将使更改和维护变得更容易。我们应该编写测试，因为每个功能都存在回归的危险。当我们已经为现有代码编写了测试时，重构它就不会那么困难，因为测试可以针对新的、更改过的代码运行。

几乎一切都需要进行测试和自动化。甚至有一些意识形态，如**测试驱动开发**（**TDD**）和**行为驱动开发**（**BDD**）。如果我们不编写自动化测试，我们不断变化的代码往往会随着时间的推移而变得更加难以测试和维护。

在本章中，我们不会讨论为什么需要测试我们的代码。我们将接受这是强制性的，并且是作为程序员生活的一部分。我们将学习如何测试使用 RxJava 编写的代码。

我们将看到编写它的单元测试并不那么困难，但也有一些难以测试的情况，比如*异步*`Observable`实例。我们将学习一些新的操作符，这些操作符将帮助我们进行测试，以及一种新的`Observable`实例。

说到这里，这一章我们将涵盖以下内容：

+   通过`BlockingObservable`类和*聚合*操作测试`Observable`实例

+   使用`TestSubscriber`实例进行深入测试

+   `TestScheduler`类和测试*异步*`Observable`实例

# 使用简单订阅进行测试

我们可以通过简单订阅*源*`Observable`实例并收集所有传入的通知来测试我们得到的内容。为了演示这一点，我们将开发一个用于创建新`Observable`实例并测试其行为的`factory`方法。

该方法将接收一个`Comparator`实例和多个项目，并将返回`Observable`实例，按排序顺序发出这些项目。项目将根据传递的`Comparator`实例进行排序。

我们可以使用 TDD 来开发这个方法。让我们首先定义测试如下：

```java
public class SortedObservableTest {
  private Observable<String> tested;
  private List<String> expected;
  @Before
  public void before() {
    tested = CreateObservable.<String>sorted(
 (a, b) -> a.compareTo(b),
 "Star", "Bar", "Car", "War", "Far", "Jar");
    expected = Arrays.asList(
      "Bar", "Car", "Far", "Jar", "Star", "War"
    );
  }
  TestData data = new TestData();
  tested.subscribe(
    (v) -> data.getResult().add(v),
    (e) -> data.setError(e),
    () -> data.setCompleted(true)
  );
  Assert.assertTrue(data.isCompleted());
  Assert.assertNull(data.getError());
  Assert.assertEquals(expected, data.getResult());
}
```

### 注意

本章的示例使用**JUnit**框架进行测试。您可以在[`junit.org`](http://junit.org)了解更多信息。

该测试使用两个变量来存储预定义的可重用状态。第一个是我们用作源的`Observable`实例—被测试的。在设置`@Before`方法中，它被分配给我们的方法`CreateObservable.sorted(Comparator, T...)`的结果，该方法尚未实现。我们比较一组`String`实例，并期望它们按照*预期*变量中存储的顺序接收—第二个可重用字段。

测试本身相当冗长。它使用`TestData`类的一个实例来存储来自*被测试*`Observable`实例的通知。

如果有一个`OnCompleted`通知，`data.completed`字段将设置为`True`。我们期望这种情况发生，这就是为什么我们在测试方法的最后进行断言。如果有一个`OnError`通知，`data.error`字段将设置为错误。我们不希望发生这种情况，所以我们断言它为`null`。

由`Observable`实例发出的每个传入项目都将添加到`data.resultList`字段中。最后，它应该等于*预期*的`List`变量，我们对此进行断言。

### 注意

前面测试的源代码可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/test/java/com/packtpub/reactive/chapter07/SortedObservableTest.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/test/java/com/packtpub/reactive/chapter07/SortedObservableTest.java)中查看/下载——这是第一个测试方法。

然而，这个测试当然失败了，因为`CreateObservable.sorted(Comparator, T...)`方法还没有实现。让我们实现它并再次运行测试：

```java
@SafeVarargs
public static <T> Observable<T> sorted(
  Comparator<? super T> comparator,
  T... data) {
    List<T> listData = Arrays.asList(data);
    listData.sort(comparator);
  return Observable.from(listData);
}
```

就是这么简单！它只是将传递的`varargs`数组转换为一个`List`变量，并使用它的`sort()`方法与传递的`Comparator`实例对其进行排序。然后，使用`Observable.from(Iterable)`方法，我们返回所需的`Observable`实例。

### 注意

前面实现的源代码可以在以下位置找到：[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/common/CreateObservable.java#L262`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/common/CreateObservable.java#L262)。

如果现在运行测试，它将通过。这很好！我们有了我们的第一个测试！但是编写类似这样的测试需要大量的样板代码。我们总是需要这三个状态变量，我们总是需要断言相同的事情。那么像`interval()`和`timer()`方法创建的*异步*`Observable`实例呢？

有一些技术可以去除样板变量，稍后，我们将看看如何测试*异步*行为。现在，我们将介绍一种新类型的 observable。

# BlockingObservable 类

每个`Observable`实例都可以用`toBlocking()`方法转换为`BlockingObservable`实例。`BlockingObservable`实例有多个方法，它们会阻塞当前线程，直到*源*`Observable`实例发出`OnCompleted`或`OnError`通知。如果有`OnError`通知，将抛出异常（`RuntimeException`异常直接抛出，检查异常包装在`RuntimeException`实例中）。

`toBlocking()`方法本身不会阻塞，但它返回的`BlockingObservable`实例的方法可能会阻塞。让我们看一些这些方法：

+   我们可以使用`forEach()`方法迭代`BlockingObservable`实例中的所有项目。这是一个使用的例子：

```java
Observable
  .interval(100L, TimeUnit.MILLISECONDS)
  .take(5)
  .toBlocking()
 .forEach(System.out::println);
System.out.println("END");
```

这也是如何使*异步*代码表现*同步*的一个例子。`interval()`方法创建的`Observable`实例不会在后台执行，因为`toBlocking()`方法使当前线程等待直到它完成。这就是为什么我们在这里使用`take(int)`方法，否则，*主*线程将永远被*阻塞*。`forEach()`方法将使用传递的函数打印五个项目，只有在那之后我们才会看到`END`输出。`BlockingObservable`类也有一个`toIterable()`方法。它返回的`Iterable`实例也可以用于迭代源发出的序列。

+   有类似*异步*的*阻塞*方法，比如`first()`、`last()`、`firstOrDefault()`和`lastOrDefault()`方法（我们在第四章中讨论过它们，*转换、过滤和累积您的数据*）。它们在等待所需项目时都会阻塞。让我们看一下以下代码片段：

```java
Integer first = Observable
  .range(3, 13).toBlocking().first();
  System.out.println(first);
  Integer last = Observable
  .range(3, 13).toBlocking().last();
  System.out.println(last);
```

这将打印`'3'`和`'15'`。

+   一个有趣的方法是`single()`方法；当*源*发出一个项目并且*源*完成时，它只返回一个项目。如果没有发出项目，或者*源*发出多个项目，分别抛出`NoSuchElementException`异常或`IllegalArgumentException`异常。

+   有一个`next()`方法，它不会*阻塞*，而是返回一个`Iterable`实例。当从这个`Iterable`实例中检索到一个`Iterator`实例时，它的每个`next()`方法都会*阻塞*，同时等待下一个传入的项目。这可以用于无限的`Observable`实例，因为*当前线程*只会在等待*下一个*项目时*阻塞*，然后它就可以继续了。（请注意，如果没有人及时调用`next()`方法，源元素可能会被跳过）。这是一个使用的例子：

```java
Iterable<Long> next = Observable
  .interval(100L, TimeUnit.MILLISECONDS)
  .toBlocking()
 .next();
Iterator<Long> iterator = next.iterator();
System.out.println(iterator.next());
System.out.println(iterator.next());
System.out.println(iterator.next());
```

*当前线程*将*阻塞*3 次，每次 100 毫秒，然后在每次暂停后打印`0`，`1`和`2`。还有一个类似的方法叫做`latest()`，它返回一个`Iterable`实例。行为不同，因为`latest()`方法产生的`Iterable`实例返回源发出的最后一个项目，或者如果没有，则等待下一个项目。

```java
Iterable<Long> latest = Observable
  .interval(1000L, TimeUnit.MILLISECONDS)
  .toBlocking()
 .latest();
iterator = latest.iterator();
System.out.println(iterator.next());
Thread.sleep(5500L);
System.out.println(iterator.next());
System.out.println(iterator.next());
```

这将打印`0`，然后`5`和`6`。

### 注意

展示所有前述运算符以及聚合运算符的源代码可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter07/BlockingObservablesAndOperators.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter07/BlockingObservablesAndOperators.java)中查看/下载。

使用`BlockingObservable`实例可以帮助我们收集我们的测试数据。但是还有一组称为**聚合运算符**的`Observable`运算符，当与`BlockingObservables`实例结合使用时也很有用。

# 聚合运算符和 BlockingObservable 类

聚合运算符产生的`Observable`实例只发出一个项目并完成。这个项目是由*source* `Observable`实例发出的所有项目组成或计算得出的。在本节中，我们只讨论其中的两个。有关更详细的信息，请参阅[`github.com/ReactiveX/RxJava/wiki/Mathematical-and-Aggregate-Operators`](https://github.com/ReactiveX/RxJava/wiki/Mathematical-and-Aggregate-Operators)。

其中第一个运算符是`count()`或`countLong()`方法。它发出*source* `Observable`实例发出的项目数。例如：

```java
Observable
  .range(10, 100)
  .count()
  .subscribe(System.out::println);
```

这将打印`100`。

另一个是`toList()`或`toSortedList()`方法，它发出一个包含*source* `Observable`实例发出的所有项目的`list`变量（可以排序）并完成。

```java
List<Integer> list = Observable
  .range(5, 15)
  .toList()
  .subscribe(System.out::println);
```

这将输出以下内容：

```java
[5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19]
```

所有这些方法，结合`toBlocking()`方法一起很好地工作。例如，如果我们想要检索由*source* `Observable`实例发出的所有项目的列表，我们可以这样做：

```java
List<Integer> single = Observable
  .range(5, 15)
  .toList()
 .toBlocking().single();
```

我们可以根据需要使用这些项目的集合：例如用于测试。

### 提示

聚合运算符还包括一个`collect()`运算符，它可以用于生成`Observable`实例并发出任意集合，例如`Set()`运算符。

# 使用聚合运算符和 BlockingObservable 类进行测试

使用在前两节中学到的运算符和方法，我们能够重新设计我们编写的测试，使其看起来像这样：

```java
@Test
public void testUsingBlockingObservable() {
  List<String> result = tested
    .toList()
 .toBlocking()
 .single();
  Assert.assertEquals(expected, result);
}
```

这里没有样板代码。我们将所有发出的项目作为列表检索并将它们与预期的列表进行比较。

在大多数情况下，使用`BlockingObsevables`类和聚合运算符非常有用。然而，在测试*异步*`Observable`实例时，它们并不那么有用，因为它们发出长时间的慢序列。长时间阻塞测试用例不是一个好的做法：慢测试是糟糕的测试。

### 注意

前面测试的源代码可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/test/java/com/packtpub/reactive/chapter07/SortedObservableTest.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/test/java/com/packtpub/reactive/chapter07/SortedObservableTest.java)找到-这是第二个测试方法。

另一个这种测试方法不太有用的情况是当我们想要检查*source*发送的`Notification`对象或订阅状态时。

还有一种编写测试的技术，可以更精细地控制*订阅*本身，这是通过一个特殊的`Subscriber`-`TestSubscriber`。

# 使用 TestSubscriber 类进行深入测试

`TestSubscriber`实例是一个特殊的`Subscriber`实例，我们可以将其传递给任何`Observable`实例的`subscribe()`方法。

我们可以从中检索所有接收到的项目和通知。我们还可以查看接收到通知的最后一个`thread`和订阅状态。

让我们使用它来重写我们的测试，以展示它的功能和存储的内容：

```java
@Test
public void testUsingTestSubscriber() {
  TestSubscriber<String> subscriber =
 new TestSubscriber<String>();
  tested.subscribe(subscriber);
  Assert.assertEquals(expected, subscriber.getOnNextEvents());
  Assert.assertSame(1, subscriber.getOnCompletedEvents().size());
  Assert.assertTrue(subscriber.getOnErrorEvents().isEmpty());
  Assert.assertTrue(subscriber.isUnsubscribed());
}
```

测试是非常简单的。我们创建一个`TestSubscriber`实例，并使用它*订阅*了*被测试的*`Observable`实例。在`Observable`实例*完成*后，我们可以访问整个状态。让我们来看一下以下的术语列表：

+   通过`getOnNextEvents()`方法，我们能够检索`Observable`实例发出的所有项目，并将它们与*expected*`List`变量进行比较。

+   通过`getOnCompletedEvents()`方法，我们能够检查*OnCompleted*通知，并检查是否已发送。例如，`Observable.never()`方法不会发送它。

+   通过`getOnErrorEvents()`方法，我们能够检查*OnError*通知是否存在。在这种情况下，我们*assert*没有*errors*。

+   使用`isUnsubscribed()`方法，我们可以*assert*在一切*完成*后，我们的`Subscriber`实例已被*unsubscribed*。

`TestSubscriber`实例也有一些*assertion*方法。因此，还有一种测试的方法：

```java
@Test
public void testUsingTestSubscriberAssertions() {
  TestSubscriber<String> subscriber = new TestSubscriber<String>();
  tested.subscribe(subscriber);
 subscriber.assertReceivedOnNext(expected);
 subscriber.assertTerminalEvent();
 subscriber.assertNoErrors();
 subscriber.assertUnsubscribed();
}
```

这些几乎是相同的*assertions*，但是使用`TestSubscriber`实例自己的`assert*`方法完成。

### 注意

前面测试的源代码可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/test/java/com/packtpub/reactive/chapter07/SortedObservableTest.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/test/java/com/packtpub/reactive/chapter07/SortedObservableTest.java)找到-这是第三和第四个测试方法。

通过这些技术，我们可以测试`RxJava`逻辑的不同行为和状态。在本章中还有一件事要学习-测试*异步*`Observable`实例，例如`Observable.interval()`方法创建的实例。

# 使用 TestScheduler 类测试异步 Observable 实例

在第六章中我们没有提到的最后一种预定义的`scheduler`是`TestScheduler`调度程序，这是一个专为单元测试设计的`scheduler`。在它上面安排的所有操作都被包装在对象中，这些对象包含它们应该执行的时间，并且在调用`Scheduler`实例的`triggerActions()`方法之前不会执行。这个方法执行所有未执行并且计划在`Scheduler`实例的当前时间或之前执行的操作。这个时间是虚拟的。这意味着它是由我们设置的，我们可以使用这个`scheduler`的特殊方法提前到未来的任何时刻。

为了演示它，我们将开发另一种创建新类型的`observable`的方法。该方法的实现本身不会在本章中讨论，但您可以在附带书籍的源代码中找到它。

该方法创建一个在设定时间间隔发出项目的`Observable`实例。但是间隔不是均匀分布的，就像内置的`interval`方法一样。我们可以提供一个不同的多个*间隔*的列表，`Observable`实例将无限循环其中。该方法的签名如下：

```java
Observable<Long> interval(List<Long> gaps, TimeUnit unit, Scheduler scheduler)
```

如果我们传递一个只包含一个时间段值的`List`变量，它的行为应该与`Observable.interval`方法相同。以下是针对这种情况的测试：

```java
@Test
public void testBehavesAsNormalIntervalWithOneGap() {
  TestScheduler testScheduler = Schedulers.test(); // (1)
  Observable<Long> interval = CreateObservable.interval(
 Arrays.asList(100L), TimeUnit.MILLISECONDS, testScheduler
 ); // (2)
  TestSubscriber<Long> subscriber = new TestSubscriber<Long>();
  interval.subscribe(subscriber); // (3)
  assertTrue(subscriber.getOnNextEvents().isEmpty()); // (4)
  testScheduler.advanceTimeBy(101L, TimeUnit.MILLISECONDS); // (5)
  assertEquals(Arrays.asList(0L), subscriber.getOnNextEvents());
  testScheduler.advanceTimeBy(101L, TimeUnit.MILLISECONDS); // (6)
  assertEquals(
    Arrays.asList(0L, 1L),
    subscriber.getOnNextEvents()
  );
  testScheduler.advanceTimeTo(1L, TimeUnit.SECONDS); // (7)
  assertEquals(
    Arrays.asList(0L, 1L, 2L, 3L, 4L, 5L, 6L, 7L, 8L, 9L),
    subscriber.getOnNextEvents()
  );
}
```

让我们来看一下以下的解释：

1.  我们使用`Schedulers.test()`方法创建`TestScheduler`实例。

1.  我们的方法的第三个参数是一个`Scheduler`实例。它将在其上*发出项目*，因此我们传递我们的`TestScheduler`实例。

1.  使用`TestSubscriber`实例，我们*订阅*了`Observable`实例。

1.  订阅后立即，我们不应该有任何通知，因此我们要检查一下。

1.  `TestScheduler`实例有一个`advanceTimeBy(long, TimeUnit)`方法，它控制其`Worker`实例的时间，因此我们可以使用它将时间推进 101 毫秒。101 毫秒后，我们期望收到一个项目——`0`。

1.  使用`advanceTimeBy()`方法，我们将时间推进 101 毫秒，然后我们应该已经收到了`0`和`1`。

1.  `TestScheduler`实例的另一个重要方法是`advanceTimeTo(long, TimeUnit)`方法。它可以用来推进到未来的特定时间点。因此，我们使用它来到达从*订阅*开始过去一秒的时刻。我们期望到那时已经收到了十个通知。

`TestScheduler`实例使用其`advanceTimeBy()`和`advanceTimeTo()`方法来控制时间，因此我们不需要*阻塞**主*`Thread`实例等待某些事件发生。我们可以直接到达它已经发生的时间。使用`TestScheduler`实例，有一个全局事件顺序。因此，如果两个任务被安排在完全相同的时间，它们有一个将执行的顺序，并且可能会导致测试出现问题，因为测试期望特定的全局顺序。如果我们有这样的操作符需要测试，我们应该通过定时到不同的值来避免这种情况——一个是 100 毫秒，另一个是 101 毫秒。使用这种技术，测试*异步*`Observable`实例不再是一个复杂的任务。

### 注意

前面测试的源代码可以在以下链接找到：[`github.com/meddle0x53/learning-rxjava/blob/master/src/test/java/com/packtpub/reactive/chapter07/CreateObservableIntervalTest.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/test/java/com/packtpub/reactive/chapter07/CreateObservableIntervalTest.java)。

# 总结

通过本章，我们不仅了解了如何使用 RxJava 编写程序，还了解了如何测试它们的任何方面。我们还学习了一些新的操作符和`BlockingObservables`类。

RxJava 库有许多在本书中未提及的操作符，但我们已经学习了更重要和有用的操作符。您可以随时参考[`github.com/ReactiveX/RxJava/wiki`](https://github.com/ReactiveX/RxJava/wiki)了解其余部分。关于*订阅*、*背压*和`Observable`实例的*生命周期*还有更多内容，但是凭借您目前的知识，掌握库中的一切不会很难。请记住，这只是一个库，一个编写代码的工具。逻辑才是重要的。这种编程方式与过程式编程有些不同，但一旦您掌握了它，就会觉得自然。

在下一章中，我们将学习如何释放*订阅*分配的资源，如何防止内存泄漏，以及如何创建我们自己的操作符，这些操作符可以在`RxJava`逻辑中链接。
