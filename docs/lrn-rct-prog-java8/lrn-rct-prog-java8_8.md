# 第八章：资源管理和扩展 RxJava

通过前面的章节，我们已经学会了如何使用 RxJava 的可观察对象。我们已经使用了许多不同的操作符和`工厂`方法。`工厂`方法是各种具有不同行为和发射源的`Observable`实例的来源。另一方面，使用操作符，我们已经围绕这些可观察对象构建了复杂的逻辑。

在本章中，我们将学习如何创建我们自己的`工厂`方法，这些方法将能够管理它们的源资源。为了做到这一点，我们需要一种管理和释放资源的方法。我们已经创建并使用了多种类似的方法，包括源文件、HTTP 请求、文件夹或内存中的数据。但其中一些并没有清理它们的资源。例如，HTTP 请求可观察对象需要一个`CloseableHttpAsyncClient`实例；我们创建了一个接收它并将其管理留给用户的方法。现在是时候学习如何自动管理和清理我们的源数据，封装在我们的`工厂`方法中了。

我们也将学习如何编写我们自己的操作符。Java 不是一种动态语言，这就是为什么我们不会将操作符添加为`Observable`类的方法。有一种方法可以将它们插入到可观察的操作链中，我们将在本章中看到。

本章涵盖的主题有：

+   使用`using()`方法进行资源管理

+   使用*高阶* `lift()` 操作符创建自定义操作符

+   使用`compose`创建操作符的组合

# 资源管理

如果我们回顾一下我们在第六章中使用的 HTTP 请求方法，*使用调度程序进行并发和并行处理*和第五章中使用的 HTTP 请求方法，它的签名是：`Observable<Map> requestJson(HttpAsyncClient client, String url)`。

我们不仅仅是调用一个方法，该方法向 URL 发出请求并将响应作为 JSON 返回，我们创建了一个`HttpAsyncClient`实例，必须启动它并将其传递给`requestJson()`方法。但还有更多：我们需要在读取结果后关闭*客户端*，因为可观察是*异步*的，我们需要等待它的`OnCompleted`通知，然后关闭它。这非常复杂，应该进行更改。从文件中读取的`Observable`需要在所有订阅者*取消订阅*后创建流/读取器/通道并关闭它们。从数据库中发出数据的`Observable`应该在读取完成后设置并关闭所有连接、语句和结果集。对于`HttpAsyncClient`对象也是如此。它是我们用来打开与远程服务器的连接的资源；我们的可观察对象应该在一切都被读取并且所有订阅者不再订阅时清理它。

让我们回答一个问题：为什么`requestJson()`方法需要这个`HttpAsyncClient`对象？答案是我们使用了一个 RxJava 模块进行 HTTP 请求。其代码如下：

```java
ObservableHttp
  .createGet(url, client)
  .toObservable();
```

这段代码创建了请求，代码需要客户端，所以我们需要客户端来创建我们的`Observable`实例。我们不能改变这段代码，因为改变它意味着要自己编写 HTTP 请求，这样不好。已经有一个库可以为我们做这件事。我们需要使用一些东西，在*订阅*时提供`HttpAsyncClient`实例，并在*取消订阅*时释放它。有一个方法可以做到这一点：`using()`工厂方法。

## 介绍`Observable.using`方法

`Observable.using`方法的签名如下：

```java
public final static <T, Resource> Observable<T> using(
  final Func0<Resource> resourceFactory,
  final Func1<? super Resource, ? extends Observable<? extends T>> observableFactory,
  final Action1<? super Resource> disposeAction
)
```

这看起来相当复杂，但仔细看一下就不难理解了。让我们来看一下以下描述：

+   它的第一个参数是 `Func0<Resource> resourceFactory`，一个创建 `Resource` 对象的函数（这里 `Resource` 是一个任意对象；它不是接口或类，而是类型参数的名称）。我们的工作是实现资源的创建。

+   `Func1<? super Resource, ? extends Observable<? extends T>> observableFactory` 参数，第二个参数，是一个接收 `Resource` 对象并返回 `Observable` 实例的函数。这个函数将使用我们已经通过第一个参数创建的 `Resource` 对象进行调用。我们可以使用这个资源来创建我们的 `Observable` 实例。

+   `Action1<? super Resource> disposeAction` 参数在应该处理 `Resource` 对象时被调用。它接收了由 `resourceFactory` 参数创建的 `Resource` 对象（并用于创建 `Observable` 实例），我们的工作是处理它。这在*取消订阅*时被调用。

我们能够创建一个函数，进行 HTTP 请求，而现在不需要传递 `HttpAsyncClient` 对象。我们有工具可以根据需要创建和处理它。让我们来实现这个函数：

```java
// (1)
public Observable<ObservableHttpResponse> request(String url) {
  Func0<CloseableHttpAsyncClient> resourceFactory = () -> {
    CloseableHttpAsyncClient client = HttpAsyncClients.createDefault(); // (2)
 client.start();
    System.out.println(
      Thread.currentThread().getName() +
      " : Created and started the client."
    );
    return client;
  };
  Func1<HttpAsyncClient, Observable<ObservableHttpResponse>> observableFactory = (client) -> { // (3)
    System.out.println(
      Thread.currentThread().getName() + " : About to create Observable."
    );
    return ObservableHttp.createGet(url, client).toObservable();
  };
  Action1<CloseableHttpAsyncClient> disposeAction = (client) -> {
    try { // (4)
      System.out.println(
        Thread.currentThread().getName() + " : Closing the client."
      );
      client.close();
    }
    catch (IOException e) {}
  };
  return Observable.using( // (5)
 resourceFactory,
 observableFactory,
 disposeAction
 );
}
```

这个方法并不难理解。让我们来分解一下：

1.  该方法的签名很简单；它只有一个参数，`URL`。调用该方法的调用者不需要创建和管理 `CloseableHttpAsyncClient` 实例的生命周期。它返回一个能够发出 `ObservableHttpResponse` 响应并*完成*的 `Observable` 实例。`getJson()` 方法可以使用它将 `ObservableHttpResponse` 响应转换为表示 JSON 的 `Map` 实例，而无需传递 *client*。

1.  `resourceFactory` lambda 很简单；它创建了一个默认的 `CloseableHttpAsyncClient` 实例并启动它。当被调用时，它将返回一个初始化的 HTTP *client*，能够请求远程服务器数据。我们输出 *client* 已准备好用于调试目的。

1.  `observableFactory` 函数可以访问由 `resourceFactory` 函数创建的 `CloseableHttpAsyncClient` 实例，因此它使用它和传递的 `URL` 来构造最终的 `Observable` 实例。这是通过 RxJava 的 `rxjava-apache-http` 模块 API（[`github.com/ReactiveX/RxApacheHttp`](https://github.com/ReactiveX/RxApacheHttp)）完成的。我们输出我们正在做的事情。

1.  `disposeAction` 函数接收了用于创建 `Observable` 实例的 `CloseableHttpAsyncClient` 对象并对其进行*关闭*。同样，我们打印一条消息到标准输出，说明我们即将这样做。

1.  借助 `using()` 工厂方法，我们返回我们的 HTTP *request* `Observable` 实例。这不会触发任何三个 lambda 中的任何一个。订阅返回的 `Observable` 实例将调用 `resourceFactory` 函数，然后调用 `observableFactory` 函数。

这就是我们实现了一个能够管理自己资源的 `Observable` 实例。让我们看看它是如何使用的：

```java
String url = "https://api.github.com/orgs/ReactiveX/repos";

Observable<ObservableHttpResponse> response = request(url);

System.out.println("Not yet subscribed.");

Observable<String> stringResponse = response
.<String>flatMap(resp -> resp.getContent()
.map(bytes -> new String(bytes, java.nio.charset.StandardCharsets.UTF_8))
.retry(5)

.map(String::trim);

System.out.println("Subscribe 1:");
System.out.println(stringResponse.toBlocking().first());

System.out.println("Subscribe 2:");
System.out.println(stringResponse.toBlocking().first());
```

我们使用新的 `request()` 方法来列出 *ReactiveX* *组织*的存储库。我们只需将 URL 传递给它，就会得到一个 `Observable` 响应。在我们订阅它之前，不会分配任何资源，也不会执行任何请求，所以我们打印出你还没有订阅。

`stringResponse` 可观察对象包含逻辑并将原始的 `ObservableHttpResponse` 对象转换为 `String`。但是，没有分配任何资源，也没有发送请求。

我们使用 `BlockingObservable` 类的 `first()` 方法订阅 `Observable` 实例并等待其结果。我们将响应作为 `String` 检索并输出它。现在，资源已分配并发出了请求。在获取数据后，`BlockingObservable` 实例封装的 `subscriber` 会自动取消订阅，因此使用的资源（HTTP 客户端）被处理掉。我们进行第二次订阅，以查看接下来会发生什么。

让我们来看一下这个程序的输出：

```java
Not yet subscribed.
Subscribe 1:
main : Created and started the client.
main : About to create Observable.
[{"id":7268616,"name":"Rx.rb","full_name":"ReactiveX/Rx.rb",...
Subscribe 2:
I/O dispatcher 1 : Closing the client.
main : Created and started the client.
main : About to create Observable.
I/O dispatcher 5 : Closing the client.
[{"id":7268616,"name":"Rx.rb","full_name":"ReactiveX/Rx.rb",...

```

因此，当我们订阅网站时，HTTP 客户端和`Observable`实例是使用我们的工厂 lambda 创建的。创建在当前主线程上执行。发出请求并打印（此处裁剪）。客户端在 IO 线程上被处理，当`Observable`实例完成执行时，请求被执行。

第二次订阅时，我们从头开始经历相同的过程；我们分配资源，创建`Observable`实例并处理资源。这是因为`using()`方法的工作方式——它为每个订阅分配一个资源。我们可以使用不同的技术来重用下一次订阅的相同结果，而不是进行新的请求和分配资源。例如，我们可以为多个订阅者重用`CompositeSubscription`方法或`Subject`实例。然而，有一种更简单的方法可以重用下一次订阅的获取响应。

# 使用 Observable.cache 进行数据缓存

我们可以使用缓存将响应缓存在内存中，然后在下一次订阅时，而不是再次请求远程服务器，使用缓存的数据。

让我们将代码更改为如下所示：

```java
String url = "https://api.github.com/orgs/ReactiveX/repos";
Observable<ObservableHttpResponse> response = request(url);

System.out.println("Not yet subscribed.");
Observable<String> stringResponse = response
.flatMap(resp -> resp.getContent()
.map(bytes -> new String(bytes)))
.retry(5)
.cast(String.class)
.map(String::trim)
.cache();

System.out.println("Subscribe 1:");
System.out.println(stringResponse.toBlocking().first());

System.out.println("Subscribe 2:");
System.out.println(stringResponse.toBlocking().first());
```

在`stringResponse`链的末尾调用的`cache()`操作符将为所有后续的`subscribers`缓存由`string`表示的响应。因此，这次的输出将是：

```java
Not yet subscribed.
Subscribe 1:
main : Created and started the client.
main : About to create Observable.
[{"id":7268616,"name":"Rx.rb",...
I/O dispatcher 1 : Closing the client.
Subscribe 2:
[{"id":7268616,"name":"Rx.rb",...

```

现在，我们可以在程序中重用我们的`stringResponse` `Observable`实例，而无需进行额外的资源分配和请求。

### 注意

演示源代码可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter08/ResourceManagement.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter08/ResourceManagement.java)找到。

最后，`requestJson()`方法可以这样实现：

```java
public Observable<Map> requestJson(String url) {
Observable<String> rawResponse = request(url)

....

return Observable.amb(fromCache(url), response);
}
```

更简单，具有资源自动管理（资源，即 http 客户端会自动创建和销毁），该方法还实现了自己的缓存功能（我们在第五章中实现了它，*组合器、条件和错误处理*）。

### 注意

书中开发的所有创建`Observable`实例的方法都可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/common/CreateObservable.java 类`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/common/CreateObservable.java 类)中找到。那里还有一个`requestJson()`方法的文件缓存实现。

有了这个，我们可以扩展 RxJava，创建自己的工厂方法，使`Observable`实例依赖于任意数据源。

本章的下一部分将展示如何将我们自己的逻辑放入`Observable`操作符链中。

# 使用 lift 创建自定义操作符

在学习和使用了许多不同的操作符之后，我们已经准备好编写自己的操作符。`Observable`类有一个名为`lift`的操作符。它接收`Operator`接口的实例。这个接口只是一个空的接口，它扩展了`Func1<Subscriber<? super R>, Subscriber<? super T>>`接口。这意味着我们甚至可以将 lambda 作为操作符传递。

学习如何使用`lift`操作符的最佳方法是编写一个示例。让我们创建一个操作符，为发出的每个项目添加一个顺序索引（当然，这也可以在没有专用操作符的情况下完成）。这样，我们将能够生成带有索引的项目。为此，我们需要一个存储项目及其索引的类。让我们创建一个更通用的称为`Pair`的类：

```java
public class Pair<L, R> {
  final L left;
  final R right;

public Pair(L left, R right) {
    this.left = left;
    this.right = right;
  }

  public L getLeft() {
    return left;
  }

public R getRight() {
    return right;
  }

  @Override
  public String toString() {
    return String.format("%s : %s", this.left, this.right);
  }

// hashCode and equals omitted

}'
```

这个类的实例是非常简单的*不可变*对象，包含两个任意对象。在我们的例子中，*left*字段将是类型为`Long`的索引，*right*字段将是发射的项。`Pair`类，和任何*不可变*类一样，包含了`hashCode()`和`equals()`方法的实现。

以下是运算符的代码：

```java
public class Indexed<T> implements Operator<Pair<Long, T>, T> {
  private final long initialIndex;
  public Indexed() {
    this(0L);
  }
  public Indexed(long initial) {
    this. initialIndex = initial;
  }
  @Override
  public Subscriber<? super T> call(Subscriber<? super Pair<Long, T>> s) {
 return new Subscriber<T>(s) {
      private long index = initialIndex;
 @Override
 public void onCompleted() {
 s.onCompleted();
 }
 @Override
 public void onError(Throwable e) {
 s.onError(e);
 }
 @Override
 public void onNext(T t) {
 s.onNext(new Pair<Long, T>(index++, t));
 }
 };
 }
}
```

`Operator`接口的`call()`方法有一个参数，一个`Subscriber`实例。这个实例将订阅由`lift()`运算符返回的可观察对象。该方法返回一个新的`Subscriber`实例，它将订阅调用了`lift()`运算符的可观察对象。我们可以在其中更改所有通知的数据，这就是我们将编写我们自己运算符逻辑的方式。

`Indexed`类有一个状态——`index`。默认情况下，它的初始值是`0`，但是有一个*构造函数*可以创建一个具有任意初始值的`Indexed`实例。我们的运算符将`OnError`和`OnCompleted`通知无修改地委托给订阅者。有趣的方法是`onNext()`。它通过创建一个`Pair`实例和`index`字段的当前值来修改传入的项。之后，`index`被递增。这样，下一个项将使用递增的`index`并再次递增它。

现在，我们有了我们的第一个运算符。让我们编写一个单元测试来展示它的行为：

```java
@Test
public void testGeneratesSequentialIndexes() {
  Observable<Pair<Long, String>> observable = Observable
    .just("a", "b", "c", "d", "e")
    .lift(new Indexed<String>());
  List<Pair<Long, String>> expected = Arrays.asList(
    new Pair<Long, String>(0L, "a"),
    new Pair<Long, String>(1L, "b"),
    new Pair<Long, String>(2L, "c"),
    new Pair<Long, String>(3L, "d"),
    new Pair<Long, String>(4L, "e")
  );
  List<Pair<Long, String>> actual = observable
    .toList()
    .toBlocking().
    single();
  assertEquals(expected, actual);
  // Assert that it is the same result for a second subscribtion.
  TestSubscriber<Pair<Long, String>> testSubscriber = new TestSubscriber<Pair<Long, String>>();
  observable.subscribe(testSubscriber);
  testSubscriber.assertReceivedOnNext(expected);
}
```

测试发射从`'a'`到`'e'`的字母，并使用`lift()`运算符将我们的`Indexed`运算符实现插入到可观察链中。我们期望得到一个由从零开始的顺序数字——*索引*和字母组成的五个`Pair`实例的列表。我们使用`toList().toBlocking().single()`技术来检索实际发射项的列表，并断言它们是否等于预期的发射。因为`Pair`实例有定义了`hashCode()`和`equals()`方法，我们可以比较`Pair`实例，所以测试通过了。如果我们第二次*订阅*，`Indexed`运算符应该从初始索引`0`开始提供索引。我们使用`TestSubscriber`实例来做到这一点，并断言字母被索引，从`0`开始。

### 注意

`Indexed`运算符的代码可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter08/Lift.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter08/Lift.java)找到，以及测试其行为的单元测试可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/test/java/com/packtpub/reactive/chapter08/IndexedTest.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/test/java/com/packtpub/reactive/chapter08/IndexedTest.java)找到。

使用`lift()`运算符和不同的`Operator`实现，我们可以编写我们自己的运算符，这些运算符作用于发射序列的每个单独项。但在大多数情况下，我们将能够在不创建新运算符的情况下实现我们的逻辑。例如，索引行为可以以许多不同的方式实现，其中一种方式是通过与`Observable.range`方法*合并*，就像这样：

```java
Observable<Pair<Long, String>> indexed = Observable.zip(
  Observable.just("a", "b", "c", "d", "e"),
  Observable.range(0, 100),
  (s, i) -> new Pair<Long, String>((long) i, s)
);
subscribePrint(indexed, "Indexed, no lift");
```

实现新的运算符有许多陷阱，比如链接订阅、支持*背压*和重用变量。如果可能的话，我们应该尝试组合现有的由经验丰富的 RxJava 贡献者编写的运算符。因此，在某些情况下，一个转换`Observable`本身的运算符是一个更好的主意，例如，将多个运算符应用于它作为一个。为此，我们可以使用*组合*运算符`compose()`。

# 使用 Observable.compose 运算符组合多个运算符

`compose()`操作符有一个`Transformer`类型的参数。`Transformer`接口，就像`Operator`一样，是一个*空*接口，它扩展了`Func1`（这种方法隐藏了使用`Func1`所涉及的类型复杂性）。不同之处在于它扩展了`Func1<Observable<T>, Observable<R>>`方法，这样它就可以转换一个`Observable`而不是一个`Subscriber`。这意味着它不是在*源*observable 发出的每个单独项目上操作，而是直接在源上操作。

我们可以通过一个例子来说明这个操作符和`Transformer`接口的使用。首先，我们将创建一个`Transformer`实现：

```java
public class OddFilter<T> implements Transformer<T, T> {
  @Override
  public Observable<T> call(Observable<T> observable) {
    return observable
      .lift(new Indexed<T>(1L))
      .filter(pair -> pair.getLeft() % 2 == 1)
      .map(pair -> pair.getRight());
  }
}
```

这个实现的思想是根据 observable 发出的顺序来过滤它们的发射。它在整个序列上操作，使用我们的`Indexed`操作符为每个项目添加一个索引。然后，它过滤具有奇数索引的`Pair`实例，并从过滤后的`Pair`实例中检索原始项目。这样，只有在奇数位置上的发射序列成员才会到达订阅者。

让我们再次编写一个*单元测试*，确保新的`OddFilter`转换器的行为是正确的：

```java
@Test
public void testFiltersOddOfTheSequence() {
  Observable<String> tested = Observable
    .just("One", "Two", "Three", "Four", "Five", "June", "July")
    .compose(new OddFilter<String>());
  List<String> expected =
    Arrays.asList("One", "Three", "Five", "July");
  List<String> actual = tested
    .toList()
    .toBlocking()
    .single();
  assertEquals(expected, actual);
}
```

正如你所看到的，我们的`OddFilter`类的一个实例被传递给`compose()`操作符，这样，它就被应用到了由`range()`工厂方法创建的 observable 上。这个 observable 发出了七个字符串。如果`OddFilter`的实现正确，它应该过滤掉在奇数位置发出的字符串。

### 注意

`OddFilter`类的源代码可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter08/Compose.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter08/Compose.java)找到。测试它的单元测试可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/test/java/com/packtpub/reactive/chapter08/IndexedTest.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/test/java/com/packtpub/reactive/chapter08/IndexedTest.java)中查看/下载。

关于实现自定义操作符的更多信息可以在这里找到：[`github.com/ReactiveX/RxJava/wiki/Implementing-Your-Own-Operators`](https://github.com/ReactiveX/RxJava/wiki/Implementing-Your-Own-Operators)。如果你在 Groovy 等动态语言中使用 RxJava，你可以扩展`Observable`类以添加新方法，或者你可以使用 Xtend，这是一种灵活的 Java 方言。参考[`mnmlst-dvlpr.blogspot.de/2014/07/rxjava-and-xtend.html`](http://mnmlst-dvlpr.blogspot.de/2014/07/rxjava-and-xtend.html)。

# 总结

创建我们自己的操作符和依赖资源的`Observable`实例给了我们在围绕`Observable`类创建逻辑时无限的可能性。我们能够将每个数据源转换成一个`Observable`实例，并以许多不同的方式转换传入的数据。

我希望这本书涵盖了 RxJava 最有趣和重要的部分。如果我漏掉了重要的内容，[`github.com/ReactiveX/RxJava/wiki`](https://github.com/ReactiveX/RxJava/wiki)上的文档是网络上最好的之一。特别是在这一部分，可以找到更多阅读材料：[`github.com/ReactiveX/RxJava/wiki/Additional-Reading`](https://github.com/ReactiveX/RxJava/wiki/Additional-Reading)。

我试图将代码和想法进行结构化，并在各章节中进行小的迭代。第一章和第二章更具有意识形态性；它们向读者介绍了函数式编程和响应式编程的基本思想，第二章试图建立`Observable`类的起源。第三章为读者提供了创建各种不同`Observable`实例的方法。第四章和第五章教会我们如何围绕这些`Observable`实例编写逻辑，第六章将多线程添加到这个逻辑中。第七章涉及读者学会编写的逻辑的*单元测试*，第八章试图进一步扩展这个逻辑的能力。

希望读者发现这本书有用。不要忘记，RxJava 只是一个工具。重要的是你的知识和思维。
