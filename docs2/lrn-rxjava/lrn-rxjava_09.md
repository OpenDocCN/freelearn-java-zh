# Transformers 和自定义操作符

在 RxJava 中，有使用`compose()`和`lift()`方法实现自定义操作符的方法，这两个方法都存在于`Observable`和`Flowable`上。大多数时候，你可能会想要组合现有的 RxJava 操作符来创建一个新的操作符。但有时，你可能需要从头开始构建的操作符。后者工作量更大，但我们将介绍如何完成这两个任务。

在本章中，我们将介绍以下主题：

+   使用`compose()`和 Transformers 组合现有操作符

+   `to()`操作符

+   使用`lift()`从头实现操作符

+   RxJava2-Extras 和 RxJava2Extensions

# Transformers

当使用 RxJava 时，你可能希望重用`Observable`或`Flowable`链的一部分，并以某种方式将这些操作符合并成一个新的操作符。优秀的开发者会寻找重用代码的机会，RxJava 通过`ObservableTransformer`和`FlowableTransformer`提供了这种能力，你可以将这些传递给`compose()`操作符。

# ObservableTransformer

将[*Google Guava*](http://search.maven.org/#artifactdetails%7Ccom.google.guava%7Cguava%7C21.0%7Cbundle)作为依赖项恢复。在第三章“基本操作符”中，我们介绍了`collect()`操作符，并使用它将`Observable<T>`转换为`Single<ImmutableList<T>>`。实际上，我们希望将`T`的发射项收集到 Google Guava 的`ImmutableList<T>`中。假设我们重复进行此操作足够多次，直到它开始显得冗余。在这里，我们使用这个`ImmutableList`操作来处理两个不同的`Observable`订阅：

```java
 import com.google.common.collect.ImmutableList;
 import io.reactivex.Observable;

 public class Launcher {

     public static void main(String[] args) {

         Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
                 .collect(ImmutableList::builder, ImmutableList.Builder::add)
                 .map(ImmutableList.Builder::build)
                 .subscribe(System.out::println);

         Observable.range(1,15)
                 .collect(ImmutableList::builder, ImmutableList.Builder::add)
                 .map(ImmutableList.Builder::build)
                 .subscribe(System.out::println);
     }
 }
```

输出如下：

```java
[Alpha, Beta, Gamma, Delta, Epsilon]
 [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

```

看看上面两个地方使用的 Observable 链的这部分：

```java
collect(ImmutableList::builder, ImmutableList.Builder::add)
     .map(ImmutableList.Builder::build)
```

两次调用有点冗余，那么我们能否将这些操作符组合成一个单独的操作符，该操作符将发射项收集到`ImmutableList`中？实际上，可以！为了针对`Observable<T>`，你可以实现`ObservableTransformer<T,R>`。这个类型有一个`apply()`方法，它接受上游的`Observable<T>`并返回下游的`Observable<R>`。在你的实现中，你可以返回一个添加了任何操作符到上游的`Observable`链，并在这些转换之后返回一个`Observable<R>`。

对于我们的示例，我们将针对给定的`Observable<T>`的任何通用类型`T`，`R`将是通过`Observable<ImmutableList<T>>`发射的`ImmutableList<T>`。我们将把这个所有东西打包在一个`ObservableTransformer<T,ImmutableList<T>>`实现中，如下面的代码片段所示：

```java
public static <T> ObservableTransformer<T, ImmutableList<T>> toImmutableList() {

     return new ObservableTransformer<T, ImmutableList<T>>() {
         @Override
         public ObservableSource<ImmutableList<T>> apply(Observable<T> upstream) {
             return upstream.collect(ImmutableList::<T>builder, ImmutableList.Builder::add)
                     .map(ImmutableList.Builder::build)
                     .toObservable(); *// must turn Single into Observable*
         }
     };
 }
```

由于`collect()`返回一个`Single`，因此我们需要在它上面调用`toObservable()`，因为`ObservableTransformer`期望返回一个`Observable`，而不是`Single`。Transformers 通过静态工厂方法提供的情况并不少见，所以我们在这里就是这样做的。

由于`ObservableTransformer`中只有一个抽象方法，我们可以使用 lambda 表达式来简化这一点。这样读起来更容易，因为它从左到右/从上到下读取，并表达出“对于给定的上游 Observable，返回添加了这些操作符的下游 Observable”：

```java
public static <T> ObservableTransformer<T, ImmutableList<T>> toImmutableList() {

     return  upstream -> upstream.collect(ImmutableList::<T>builder, ImmutableList.Builder::add)
                 .map(ImmutableList.Builder::build)
                 .toObservable(); *// must turn Single into Observable*
 }
```

要将一个 Transformer 加入到`Observable`链中，你需要将它传递给`compose()`操作符。当在`Observable<T>`上调用时，`compose()`操作符接受一个`ObservableTransformer<T,R>`并返回转换后的`Observable<R>`。这允许你在多个地方重用 Rx 逻辑，现在我们可以在两个`Observable`操作上调用`compose(toImmutableList())`：

```java
 import com.google.common.collect.ImmutableList;
 import io.reactivex.Observable;
 import io.reactivex.ObservableTransformer;

 public class Launcher {

     public static void main(String[] args) {

         Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
                 .compose(toImmutableList())
                 .subscribe(System.out::println);

         Observable.range(1,10)
                 .compose(toImmutableList())
                 .subscribe(System.out::println);
     }

     public static <T> ObservableTransformer<T, ImmutableList<T>> toImmutableList() {

         return  upstream -> upstream.collect(ImmutableList::<T>builder, ImmutableList.Builder::add)
                     .map(ImmutableList.Builder::build)
                     .toObservable(); *// must turn Single into Observable*
     }
 }
```

输出如下：

```java
[Alpha, Beta, Gamma, Delta, Epsilon]
 [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

在 API 中，将 Transformers 组织在静态工厂类中是很常见的。在实际应用中，你可能会在`GuavaTransformers`类中存储你的`toImmutableList()` Transformer。然后，你可以在`Observable`操作中通过调用`compose(GuavaTransformers.toImmutableList())`来调用它。

注意：在这个例子中，实际上我们可以将`toImmutableList()`制作成一个可重用的单例，因为它不接受任何参数。

你也可以创建针对特定发射类型并接受参数的 Transformers。例如，你可以创建一个接受分隔符参数并将`String`发射与该分隔符连接的`joinToString()` Transformer。只有当在`Observable<String>`上调用时，此`ObservableTransformer`的使用才会编译：

```java
 import io.reactivex.Observable;
 import io.reactivex.ObservableTransformer;

 public class Launcher {

     public static void main(String[] args) {

         Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
                 .compose(joinToString("/"))
                 .subscribe(System.out::println);
     }

     public static ObservableTransformer<String, String> joinToString(String separator) {
         return  upstream -> upstream
                 .collect(StringBuilder::new, (b,s) ->  {
                     if (b.length() == 0)
                         b.append(s);
                     else
                         b.append(separator).append(s);
                 })
                 .map(StringBuilder::toString)
                 .toObservable();
     }
 }
```

输出如下：

```java
Alpha/Beta/Gamma/Delta/Epsilon
```

Transformers 是重用一系列执行共同任务的操作符的绝佳方式，利用它们可以极大地提高你的 Rx 代码的可重用性。通常，通过实现静态工厂方法，你可以获得最大的灵活性和速度，但你也可以将`ObservableTransformer`扩展到你的类实现中。

正如我们将在第十二章中学习的那样，*使用 Kotlin 与 RxJava*，Kotlin 语言提供了强大的语言特性，可以进一步简化 RxJava。你不需要使用 Transformers，而是可以利用扩展函数将操作符添加到`Observable`和`Flowable`类型，而无需继承。我们将在稍后了解更多关于这一点。

# FlowableTransformer

当你实现自己的`ObservableTransformer`时，你可能还想创建一个`FlowableTransformer`对应物。这样，你就可以在你的操作符上使用 Observables 和 Flowables。

`FlowableTransformer`与`ObservableTransformer`没有太大区别。当然，它将支持背压，因为它与 Flowables 组合。在其他方面，它的使用几乎相同，只是你显然需要在`Flowable`上而不是`Observable`上传递它到`compose()`。

在这里，我们将我们的`toImmutableList()`方法返回一个`ObservableTransformer`，并将其实现为`FlowableTransformer`：

```java
 import com.google.common.collect.ImmutableList;
 import io.reactivex.Flowable;
 import io.reactivex.FlowableTransformer;

 public class Launcher {

     public static void main(String[] args) {

         Flowable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
                 .compose(toImmutableList())
                 .subscribe(System.out::println);

         Flowable.range(1,10)
                 .compose(toImmutableList())
                 .subscribe(System.out::println);
     }

     public static <T> FlowableTransformer<T, ImmutableList<T>> toImmutableList() {

         return  upstream -> upstream.collect(ImmutableList::<T>builder, ImmutableList.Builder::add)
                 .map(ImmutableList.Builder::build)
                 .toFlowable(); *// must turn Single into Flowable*
     }
 }
```

你也应该能够将类似的转换应用到我们的`joinToString()`示例中的`FlowableTransformer`。

你可以考虑创建单独的静态实用类来分别存储你的 `FlowableTransformers` 和 `ObservableTransformers`，以防止名称冲突。我们的 `FlowableTransformer` 和 `ObservableTransformer` 的 `toImmutableList()` 变体不能存在于同一个静态实用类中，除非它们有不同的方法名称。但将它们放在单独的类中可能更干净，例如 `MyObservableTransformers` 和 `MyFlowableTransformers`。你也可以将它们放在具有相同类名 `MyTransformers` 的单独包中，一个用于 Observables，另一个用于 Flowables。

# 避免与 Transformers 共享状态

当你开始创建自己的 Transformers 和自定义操作符（稍后介绍）时，一个容易犯的错误是在多个订阅之间共享状态。这可能会迅速产生不希望出现的副作用和有缺陷的应用程序，这也是你创建自己的操作符时必须谨慎行事的原因之一。

假设你想创建一个 `ObservableTransformer<T,IndexedValue<T>>`，它将每个发射项与其连续的索引（从 0 开始）配对。首先，你创建一个 `IndexedValue<T>` 类来简单地将每个 `T` 值与一个 `int index` 配对：

```java
 static final class IndexedValue<T> {
     final int index;
     final T value;

     IndexedValue(int index, T value) {
         this.index = index;
         this.value = value;
     }

     @Override
     public String toString() {
         return  index + " - " + value;
     }
 }
```

然后，你创建一个 `ObservableTransformer<T,IndexedValue<T>>`，它使用 `AtomicInteger` 来递增并将一个整数附加到每个发射项。但我们的实现中存在一些问题：

```java
static <T> ObservableTransformer<T,IndexedValue<T>> withIndex() {
     final AtomicInteger indexer = new AtomicInteger(-1);
     return upstream -> upstream.map(v -> new IndexedValue<T>(indexer.incrementAndGet(), v));
 }
```

看到什么问题了吗？尝试运行这个具有两个观察者并使用此 `withIndex()` Transformers 的 `Observable` 操作。仔细查看输出：

```java
 import io.reactivex.Observable;
 import io.reactivex.ObservableTransformer;
 import java.util.concurrent.atomic.AtomicInteger;

 public class Launcher {

     public static void main(String[] args) {

         Observable<IndexedValue<String>> indexedStrings =
                 Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
                     .compose(withIndex());

         indexedStrings.subscribe(v -> System.out.println("Subscriber 1: " + v));
         indexedStrings.subscribe(v -> System.out.println("Subscriber 2: " + v));
     }

     static <T> ObservableTransformer<T,IndexedValue<T>> withIndex() {
         final AtomicInteger indexer = new AtomicInteger(-1);
         return upstream -> upstream.map(v -> new IndexedValue<T>(indexer.incrementAndGet(), v));
     }

     static final class IndexedValue<T> {
         final int index;
         final T value;

         IndexedValue(int index, T value) {
             this.index = index;
             this.value = value;
         }

         @Override
         public String toString() {
             return  index + " - " + value;
         }
     }
 }
```

输出如下：

```java
 Subscriber 1: 0 - Alpha
 Subscriber 1: 1 - Beta
 Subscriber 1: 2 - Gamma
 Subscriber 1: 3 - Delta
 Subscriber 1: 4 - Epsilon
 Subscriber 2: 5 - Alpha
 Subscriber 2: 6 - Beta
 Subscriber 2: 7 - Gamma
 Subscriber 2: 8 - Delta
 Subscriber 2: 9 - Epsilon
```

注意，`AtomicInteger` 的单个实例在两个订阅之间共享，这意味着其状态也被共享。在第二个订阅中，它不会从 `0` 开始，而是从上一个订阅留下的索引处开始，由于上一个订阅在 `4` 结束，所以从索引 `5` 开始。

除非你故意实现某些有状态的行为，否则这可能是可能产生令人沮丧的错误的不希望出现的副作用。常量通常没问题，但订阅之间的可变共享状态通常是你要避免的。

为每个订阅创建一个新的资源（如 `AtomicInteger`）的快速简单方法是将一切包裹在 `Observable.defer()` 中，包括 `AtomicInteger` 实例。这样，每次都会创建一个新的 `AtomicInteger`，并返回索引操作：

```java
static <T> ObservableTransformer<T,IndexedValue<T>> withIndex() {
     return upstream -> Observable.defer(() -> {
         AtomicInteger indexer = new AtomicInteger(-1);
         return upstream.map(v -> new IndexedValue<T>(indexer.incrementAndGet(), v));
     });
 }
```

你也可以在 `Observable.fromCallable()` 中创建一个 `AtomicInteger`，并使用 `flatMap()` 在它上面创建一个使用它的 `Observable`。

在这个特定的例子中，你也可以使用 `Observable.zip()` 或 `zipWith()` 与 `Observable.range()`。由于这也是一个纯 Rx 方法，多个订阅者之间不会共享状态，这也会解决我们的问题：

```java
 static <T> ObservableTransformer<T,IndexedValue<T>> withIndex() {
     return upstream ->
             Observable.zip(upstream,
                     Observable.range(0,Integer.MAX_VALUE),
                     (v,i) -> new IndexedValue<T>(i, v)
             );
 }
```

再次强调，在 Rx 中，意外的共享状态和副作用是危险的！无论你使用什么实现来创建你的 Transformer，如果可能的话，最好在你的实现中依赖纯 Rx 工厂和操作符。除非你满足某些奇怪的业务需求，其中明确需要共享状态，否则请避免创建可能被订阅共享的状态和对象。

# 使用 to() 进行流畅转换

在罕见的情况下，你可能发现自己需要将一个 `Observable` 传递给另一个 API，该 API 将其转换为专有类型。这可以通过将 `Observable` 作为参数传递给执行此转换的工厂来完成。然而，这并不总是感觉流畅，这就是 `to()` 操作符发挥作用的地方。

例如，JavaFX 有一个 `Binding<T>` 类型，它包含一个可变值，类型为 `T`，并且当它发生变化时，会通知受影响的用户界面元素进行更新。RxJavaFX 有 `JavaFxObserver.toBinding()` 和 `JavaFxSubscriber.toBinding()` 工厂，可以将 `Observable<T>` 或 `Flowable<T>` 转换为 JavaFX 的 `Binding<T>`。以下是一个简单的 JavaFX `Application`，它使用基于 `Observable<String>` 构建的 `Binding<String>`，该 `Binding<String>` 用于绑定到 `label` 的 `textProperty()` 操作符：

```java
 import io.reactivex.Observable;
 import io.reactivex.rxjavafx.observers.JavaFxObserver;
 import io.reactivex.rxjavafx.schedulers.JavaFxScheduler;
 import javafx.application.Application;
 import javafx.beans.binding.Binding;
 import javafx.scene.Scene;
 import javafx.scene.control.Label;
 import javafx.scene.layout.VBox;
 import javafx.stage.Stage;

 import java.util.concurrent.TimeUnit;

 public final class JavaFxApp extends Application {

     @Override
     public void start(Stage stage) throws Exception {

         VBox root = new VBox();
         Label label = new Label("");

         *// Observable with second timer*
         Observable<String> seconds =
                 Observable.interval(1, TimeUnit.SECONDS)
                         .map(i -> i.toString())
                         .observeOn(JavaFxScheduler.platform());

         *// Turn Observable into Binding*
         Binding<String> binding = JavaFxObserver.toBinding(seconds);

         *//Bind Label to Binding*
         label.textProperty().bind(binding);

         root.setMinSize(200, 100);
         root.getChildren().addAll(label);

         Scene scene = new Scene(root);
         stage.setScene(scene);
         stage.show();
     }
 }
```

由于我们已经习惯了使用 RxJava 进行流畅编程，将 `Observable<String>` 转换为 `Binding<String>` 的过程也变成 `Observable` 链的一部分不是很好吗？这样，我们就不必打破我们的流畅风格，也不必保存中间变量。这可以通过 `to()` 操作符来完成，它简单地接受一个 `Function<Observable<T>,R>` 来将 `Observable<T>` 转换为任何任意的 `R` 类型。在这种情况下，我们可以在 `Observable` 链的末尾使用 `to()` 将我们的 `Observable<String>` 转换为 `Binding<String>`：

```java
 import io.reactivex.Observable;
 import io.reactivex.rxjavafx.observers.JavaFxObserver;
 import io.reactivex.rxjavafx.schedulers.JavaFxScheduler;
 import javafx.application.Application;
 import javafx.beans.binding.Binding;
 import javafx.scene.Scene;
 import javafx.scene.control.Label;
 import javafx.scene.layout.VBox;
 import javafx.stage.Stage;
 import java.util.concurrent.TimeUnit;

 public final class JavaFxApp extends Application {

     @Override
     public void start(Stage stage) throws Exception {

         VBox root = new VBox();
         Label label = new Label("");

         *// Turn Observable into Binding*
         Binding<String> binding =
                 Observable.interval(1, TimeUnit.SECONDS)
                         .map(i -> i.toString())
                         .observeOn(JavaFxScheduler.platform())
                         .to(JavaFxObserver::toBinding);

         *//Bind Label to Binding*
         label.textProperty().bind(binding);

         root.setMinSize(200, 100);
         root.getChildren().addAll(label);

         Scene scene = new Scene(root);
         stage.setScene(scene);
         stage.show();
     }
 }
```

简单但很有帮助，对吧？当你处理基于 Rx Observables 和 Flowables 构建的专有非 Rx 类型时，这是一个方便的实用工具，可以保持流畅的 Rx 风格，尤其是在与绑定框架交互时。

# 操作符

理想情况下，你很少需要从头开始实现 `ObservableOperator` 或 `FlowableOperator` 来构建自己的操作符。`ObservableTransformer` 和 `FlowableTransformer` 希望能满足大多数可以使用现有操作符组合新操作符的情况，这通常是 safest 的途径。但有时，你可能发现自己必须做一些现有操作符无法做到或不方便做到的事情。在你用尽所有其他选项之后，你可能不得不创建一个操作符，该操作符在上游和下游之间操作每个 `onNext()`、`onComplete()` 和 `onError()` 事件。

在你出去创建自己的操作符之前，先尝试使用现有的操作符，通过 `compose()` 和转换器来实现。如果这还失败了，建议你在 StackOverflow 上发帖询问，看看 RxJava 社区是否存在这样的操作符或者是否可以轻松组合。RxJava 社区在 StackOverflow 上非常活跃，他们可能会提供一个解决方案，并且只在需要时增加解决方案的复杂性。

注意，David Karnok 的 [*RxJava2Extensions*](https://github.com/akarnokd/RxJava2Extensions) 和 Dave Moten 的 [*RxJava2-Extras*](https://github.com/davidmoten/rxjava2-extras) 包含许多有用的转换器和操作符，可以增强 RxJava。你应该查看这些库，看看它们是否满足你的需求。

如果确定没有现有的解决方案，那么请谨慎地构建自己的操作符。再次强调，建议你首先在 StackOverflow 上寻求帮助。构建一个原生操作符并不容易，从 Rx 专家那里获得见解和经验是非常有价值的，并且很可能是有必要的。

# 实现 `ObservableOperator`

实现 `ObservableOperator`（以及 `FlowableTransformer`）比创建 `ObservableTransformer` 更复杂。不是通过组合一系列现有的操作符，而是通过实现自己的 `Observer` 来拦截来自上游的 `onNext()`、`onComplete()`、`onError()` 和 `onSubscribe()` 调用。这个 `Observer` 将逻辑上传递 `onNext()`、`onComplete()` 和 `onError()` 事件到下游 `Observer`，以实现所需操作。

假设你想创建自己的 `doOnEmpty()` 操作符，该操作符在调用 `onComplete()` 且没有发生任何发射时执行一个 `Action`。要创建自己的 `ObservableOperator<Downstream,Upstream>`（其中 `Upstream` 是上游发射类型，`Downstream` 是下游发射类型），你需要实现其 `apply()` 方法。这个方法接受一个 `Observer<Downstream>` 参数 `observer` 并返回一个 `Observer<Upstream>`。

你可以通过在 `Observable` 链中的 `lift()` 操作符中调用它来使用这个 `ObservableOperator`，如下所示：

```java
 import io.reactivex.Observable;
 import io.reactivex.ObservableOperator;
 import io.reactivex.Observer;
 import io.reactivex.functions.Action;
 import io.reactivex.observers.DisposableObserver;

 public class Launcher {

     public static void main(String[] args) {

         Observable.range(1, 5)
                 .lift(doOnEmpty(() -> System.out.println("Operation 1 Empty!")))
                 .subscribe(v -> System.out.println("Operation 1: " + v));

         Observable.<Integer>empty()
                 .lift(doOnEmpty(() -> System.out.println("Operation 2 Empty!")))
                 .subscribe(v -> System.out.println("Operation 2: " + v));
     }

     public static <T> ObservableOperator<T,T> doOnEmpty(Action action) {
         return new ObservableOperator<T, T>() {

             @Override
             public Observer<? super T> apply(Observer<? super T> observer) throws Exception {
                 return new DisposableObserver<T>() {
                     boolean isEmpty = true;

                     @Override
                     public void onNext(T value) {
                         isEmpty = false;
                         observer.onNext(value);
                     }

                     @Override
                     public void onError(Throwable t) {
                         observer.onError(t);
                     }

                     @Override
                     public void onComplete() {
                         if (isEmpty) {
                             try {
                                 action.run();
                             } catch (Exception e) {
                                 onError(e);
                                 return;
                             }
                         }
                         observer.onComplete();
                     }
                 };
             }
         };
     }
 }
```

输出如下：

```java
 Operation 1: 1
 Operation 1: 2
 Operation 1: 3
 Operation 1: 4
 Operation 1: 5
 Operation 2 Empty!
```

在 `apply()` 中，你接收传递的 `Observer`，该 `Observer` 接受下游的事件。你创建另一个 `Observer`（在这种情况下，我们应该使用一个 `DisposableObserver`，它可以为我们处理销毁请求）来接收上游的发射和事件，并将它们传递给下游 `Observer`。你可以操纵事件以执行所需的逻辑，以及添加任何副作用。

在这种情况下，我们只是将上游的事件原封不动地传递到下游，但会跟踪是否调用了`onNext()`以标记是否有发射发生。当调用`onComplete()`且没有发射发生时，它将在`onComplete()`中执行用户指定的操作。通常，将可能抛出运行时错误的任何代码包裹在`try-catch`中，并将捕获的错误传递给`onError()`是一个好主意。

使用`ObservableOperator`时，你可能觉得奇怪，你得到的是下游作为输入，而必须为上游生成一个`Observer`作为输出。例如，在使用`map()`操作符时，函数接收上游的值并返回要向下游发射的值。这是因为来自`ObservableOperator`的代码在订阅时执行，此时调用是从末端的`Observer`（下游）向源`Observable`（上游）传递。

由于它是一个单抽象方法类，你也可以将你的`ObservableOperator`实现表达为一个 lambda，如下所示：

```java
 public static <T> ObservableOperator<T,T> doOnEmpty(Action action) {
     return observer -> new DisposableObserver<T>() {
         boolean isEmpty = true;

         @Override
         public void onNext(T value) {
             isEmpty = false;
             observer.onNext(value);
         }

         @Override
         public void onError(Throwable t) {
             observer.onError(t);
         }

         @Override
         public void onComplete() {
             if (isEmpty) {
                 try {
                     action.run();
                 } catch (Exception e) {
                     onError(e);
                     return;
                 }
             }
             observer.onComplete();
         }
     };
 }
```

就像`Transformers`一样，在创建自定义操作符时要小心，除非你确实想要这样做，否则不要在订阅之间共享状态。这是一个相对简单的操作符，因为它是一个简单的响应式构建块，但操作符可以变得极其复杂。这尤其适用于操作符处理并发（例如，`observeOn()`和`subscribeOn()`)或订阅之间共享状态（例如，`replay()`）时。`groupBy()`、`flatMap()`和`window()`的实现同样复杂和错综。

在调用三个事件时，`Observable`契约中有一些规则你必须遵守。在`onError()`发生之后（或反之）永远不要调用`onComplete()`。在调用`onComplete()`或`onError()`之后，不要调用`onNext()`，并且在销毁之后不要调用任何事件。违反这些规则可能会产生意外的后果。

另一点需要指出的是，`onNext()`、`onComplete()`和`onError()`调用可以根据需要被操作和混合。例如，`toList()`不会为从上游接收到的每个`onNext()`调用都向下游传递一个`onNext()`调用。它将在内部列表中持续收集这些发射。当上游调用`onComplete()`时，它将在调用`onComplete()`之前在下游上调用`onNext()`以传递该列表。在这里，我们实现了自己的`myToList()`操作符来理解`toList()`是如何工作的，尽管在正常情况下，我们应该使用`collect()`或`toList()`：

```java
 import io.reactivex.Observable;
 import io.reactivex.ObservableOperator;
 import io.reactivex.observers.DisposableObserver;
 import java.util.ArrayList;
 import java.util.List;

 public class Launcher {

     public static void main(String[] args) {

         Observable.range(1, 5)
                 .lift(myToList())
                 .subscribe(v -> System.out.println("Operation 1: " + v));

         Observable.<Integer>empty()
                 .lift(myToList())
                 .subscribe(v -> System.out.println("Operation 2: " + v));
     }

     public static <T> ObservableOperator<List<T>,T> myToList() {
         return observer -> new DisposableObserver<T>() {

             ArrayList<T> list = new ArrayList<>();

             @Override
             public void onNext(T value) {
                 *//add to List, but don't pass anything downstream*
                 list.add(value);
             }

             @Override
             public void onError(Throwable t) {
                 observer.onError(t);
             }

             @Override
             public void onComplete() {
                 observer.onNext(list); *//push List downstream*
                 observer.onComplete();
             }
         };
     }
 }
```

输出如下：

```java
 Operation 1: [1, 2, 3, 4, 5]
 Operation 2: []
```

在你开始雄心勃勃地创建自己的操作符之前，研究 RxJava 或其他库（如 RxJava2-Extras）的源代码可能是个好主意。操作符的正确实现可能很困难，因为你需要很好地理解如何从命令式模式构建响应式模式。你也会想彻底测试它（我们将在第十章 Testing and Debugging 中介绍），以确保在生产之前它表现正确。

# FlowableOperator

当你创建自己的`ObservableOperator`时，你很可能会同时创建一个`FlowableOperator`的对应物。这样，你的操作符就可以用于 Observables 和 Flowables。幸运的是，`FlowableOperator`的实现方式与`ObservableOperator`类似，如下所示：

```java
 import io.reactivex.Flowable;
 import io.reactivex.FlowableOperator;
 import io.reactivex.functions.Action;
 import io.reactivex.subscribers.DisposableSubscriber;
 import org.reactivestreams.Subscriber;

 public class Launcher {

     public static void main(String[] args) {

         Flowable.range(1, 5)
                 .lift(doOnEmpty(() -> System.out.println("Operation 1 Empty!")))
                 .subscribe(v -> System.out.println("Operation 1: " + v));

         Flowable.<Integer>empty()
                 .lift(doOnEmpty(() -> System.out.println("Operation 2 Empty!")))
                 .subscribe(v -> System.out.println("Operation 2: " + v));
     }

     public static <T> FlowableOperator<T,T> doOnEmpty(Action action) {
         return new FlowableOperator<T, T>() {
             @Override
             public Subscriber<? super T> apply(Subscriber<? super T> subscriber) throws Exception {
                 return new DisposableSubscriber<T>() {
                     boolean isEmpty = true;

                     @Override
                     public void onNext(T value) {
                         isEmpty = false;
                         subscriber.onNext(value);
                     }

                     @Override
                     public void onError(Throwable t) {
                         subscriber.onError(t);
                     }

                     @Override
                     public void onComplete() {
                         if (isEmpty) {
                             try {
                                 action.run();
                             } catch (Exception e) {
                                 onError(e);
                                 return;
                             }
                         }
                         subscriber.onComplete();
                     }
                 };
             }
         };
     }
 }
```

我们没有使用观察者，而是使用了订阅者，希望这一点不会让你感到惊讶。通过`apply()`传递的`Subscriber`接收下游的事件，而实现的`Subscriber`接收上游的事件，并将其转发到下游（就像我们使用了`DisposableObserver`一样，我们使用`DisposableSubscriber`来处理销毁/取消订阅）。就像之前一样，`onComplete()`将验证没有发生发射，并在这种情况下运行指定的操作。

当然，你也可以将你的`FlowableOperator`表达为 lambda 表达式：

```java
 public static <T> FlowableOperator<T,T> doOnEmpty(Action action) {
     return subscriber -> new DisposableSubscriber<T>() {
         boolean isEmpty = true;

         @Override
         public void onNext(T value) {
             isEmpty = false;
             subscriber.onNext(value);
         }

         @Override
         public void onError(Throwable t) {
             subscriber.onError(t);
         }

         @Override
         public void onComplete() {
             if (isEmpty) {
                 try {
                     action.run();
                 } catch (Exception e) {
                     onError(e);
                     return;
                 }
             }
             subscriber.onComplete();
         }
     };
 }
```

再次强调，当你开始实现自己的操作符时，要勤奋并彻底，尤其是在它们达到复杂性的阈值时。努力使用现有的操作符来组合 Transformers，并在 StackOverflow 或 RxJava 社区中寻求帮助，看看其他人是否能首先指出明显的解决方案。实现操作符是一件你应该谨慎对待的事情，只有在所有其他选项都已用尽时才追求。

# 单独为 Singles、Maybes 和 Completables 创建的 Transformer 和操作符

对于`Single`、`Maybe`和`Completable`，都有对应的 Transformer 和操作符。当你想要创建一个产生`Single`的`Observable`或`Flowable`操作符时，你可能发现通过调用其`toObservable()`或`toFlowable()`操作符将其转换回`Observable`/`Flowable`可能更容易。这也适用于`Maybe`。

如果在某些罕见的情况下，你需要创建一个专门用于将`Single`转换为另一个`Single`的`Transformer`或操作符，你将想要使用`SingleTransformer`或`SingleOperator`。`Maybe`和`Completable`将分别有`MaybeTransformer`/`MaybeOperator`和`CompletableTransformer`/`CompletableOperator`的对应物。所有这些`apply()`的实现应该有类似的体验，你将使用`SingleObserver`、`MaybeObserver`和`CompletableObserver`来代理上游和下游。

这里是一个`SingleTransformer`的示例，它接受`Single<Collection<T>>`并将发射的`Collection`映射到一个不可修改的集合：

```java
 import io.reactivex.Observable;
 import io.reactivex.SingleTransformer;
 import java.util.Collection;
 import java.util.Collections;

 public class Launcher {

     public static void main(String[] args) {
         Observable.just("Alpha","Beta","Gamma","Delta","Epsilon")
                 .toList()
                 .compose(toUnmodifiable())
                 .subscribe(System.out::println);
     }

     public static <T>  SingleTransformer<Collection<T>, Collection<T>> toUnmodifiable() {
         return singleObserver -> singleObserver.map(Collections::unmodifiableCollection);
     }
 }
```

输出如下：

```java
[Alpha, Beta, Gamma, Delta, Epsilon]
```

# 使用 RxJava2-Extras 和 RxJava2Extensions

如果你感兴趣想要了解 RxJava 提供的额外操作符，那么探索[*RxJava2-Extras*](https://github.com/davidmoten/rxjava2-extras)和[*RxJava2Extensions*](https://github.com/akarnokd/RxJava2Extensions)库可能是有价值的。虽然这两个库都没有达到 1.0 版本，但有用的操作符、转换器和`Observable`/`Flowable`工厂作为持续项目不断添加。

两个有用的操作符是`toListWhile()`和`collectWhile()`。这些操作符会在满足一定条件时将发射项缓冲到列表或集合中。由于`BiPredicate`将列表/集合和下一个`T`项作为 lambda 输入参数传递，你可以使用这个功能来缓冲项目，但在发射项发生变化时停止缓冲。在这里，我们持续将字符串收集到列表中，但在长度变化时将列表向前推进（类似于`distinctUntilChanged()`）。我们还将检查列表是否为空，因为这是下一个缓冲的开始，以及从列表中采样一个项目来与下一个发射项比较长度：

```java
 import com.github.davidmoten.rx2.flowable.Transformers;
 import io.reactivex.Flowable;

 public class Launcher {

     public static void main(String[] args) {

         Flowable.just("Alpha","Beta","Zeta","Gamma","Delta","Theta","Epsilon")
                 .compose(Transformers.toListWhile((list,next) ->
                     list.size() == 0 || list.get(0).length() == next.length()
                 )).subscribe(System.out::println);
     }
 }
```

输出如下：

```java
 [Alpha]
 [Beta, Zeta]
 [Gamma, Delta, Theta]
 [Epsilon]
```

在 RxJava2-Extras 和 RxJava2Extensions 上花费一些高质量的时间来了解它们的自定义操作符。这样，你就不必重新发明可能已经存在的东西，而且已经有很多强大的工厂和操作符。我个人最喜欢的一个是可重置的`cache()`操作符，它的工作方式类似于我们在第五章中学习的缓存，*多播*，但它可以在任何时间清除并重新订阅到源。它还可以在固定的时间间隔或无活动期间清除缓存，防止过时的缓存持久化。

# 摘要

在本章中，我们通过创建自己的操作符来入门。使用`ObservableTransformer`和`FlowableTransformer`将现有操作符组合在一起以创建新的操作符更为可取，即使如此，引入可能导致不希望副作用的状态资源时也需要谨慎。当所有其他方法都失败时，你可以创建自己的`ObservableOperator`或`FlowableOperator`，并在低级别创建一个拦截和转发每个发射项和事件的操作符。这可能很棘手，你应该尝试所有其他选项，但通过仔细研究和测试，创建操作符可以是一项有价值的先进技能。只是要小心不要重复造轮子，并在开始尝试自定义操作符时寻求 Rx 社区的帮助。

如果你真正对实现自己的操作符（在低级别上，而不是使用 Transformers）感兴趣，那么一定要研究 RxJava 和其他信誉良好的 RxJava 扩展库中的现有操作符。很容易拼凑出一个操作符并相信一切都会顺利进行，但实际上有很多你可能忽略的复杂性。你的操作符需要支持序列化、可取消、并发，并处理重入性（当发射操作在同一个线程上引发请求时发生）。当然，有些操作符比其他操作符简单，但你绝不应该在没有深入研究之前就做出假设。

在下一章中，我们将学习关于针对 RxJava API 和实用工具进行单元测试的不同策略。无论你是否创建了自己的自定义操作符，或者在工作中有一个 Rx 项目，自动化测试都是你想要熟练掌握的技能。我们还将学习如何调试 RxJava 应用程序，这并不总是容易，但可以有效地完成。
