# 合并 Observables

我们已经介绍了许多抑制、转换、减少和收集发射的操作符。这些操作符可以做很多工作，但关于将多个 Observable 合并并统一它们呢？如果我们想用 ReactiveX 完成更多的工作，我们需要将多个数据流和事件流结合起来，使它们协同工作，并且有操作符和工厂可以实现这一点。这些组合操作符和工厂也可以安全地与在不同线程上发生的 Observables 一起工作（在第六章[4f59db87-4b1d-47e6-95e3-ae0a43193c5f.xhtml]，*并发与并行化*中讨论）。

这是我们开始从使 RxJava 有用转变为使其强大的地方。我们将介绍以下操作来合并 Observables：

+   合并

+   连接

+   不明确

+   压缩

+   组合最新

+   分组

# 合并

在 ReactiveX 中，一个常见的任务是将两个或多个 `Observable<T>` 实例合并成一个 `Observable<T>`。这个合并的 `Observable<T>` 将会同时订阅其所有合并的源，这使得它对于合并有限和无限 Observable 都非常有效。我们可以通过工厂以及操作符来利用这种合并行为。

# `Observable.merge()` 和 `mergeWith()`

`Observable.merge()` 操作符将接受两个或更多发射相同类型 `T` 的 `Observable<T>` 源，并将它们合并成一个单一的 `Observable<T>`。

如果我们只有两个到四个要合并的 `Observable<T>` 源，你可以将每个源作为参数传递给 `Observable.merge()` 工厂。在下面的代码片段中，我将两个 `Observable<String>` 实例合并成了一个 `Observable<String>`：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable<String> source1 =
          Observable.just("Alpha", "Beta", "Gamma", "Delta", 
"Epsilon");

        Observable<String> source2 =
          Observable.just("Zeta", "Eta", "Theta");

        Observable.merge(source1, source2)
          .subscribe(i -> System.out.println("RECEIVED: " + i));
     }
}
```

前述程序的输出如下：

```java
    RECEIVED: Alpha
    RECEIVED: Beta
    RECEIVED: Gamma
    RECEIVED: Delta
    RECEIVED: Epsilon
    RECEIVED: Zeta
    RECEIVED: Eta
    RECEIVED: Theta
```

或者，你可以使用 `mergeWith()`，它是 `Observable.merge()` 操作符的版本**：**

```java
    source1.mergeWith(source2)
      .subscribe(i -> System.out.println("RECEIVED: " + i));
```

`Observable.merge()` 工厂和 `mergeWith()` 操作符将同时订阅所有指定的源，但如果它们是冷源并且在同一线程上，则可能会按顺序触发发射。这只是一个实现细节，如果你明确想要按顺序触发每个 `Observable` 的元素并保持它们的发射顺序，应使用 `Observable.concat()`。

即使看起来顺序被保留了，在使用合并工厂和操作符时也不应依赖于顺序。话虽如此，每个源 Observable 的发射顺序是保持的。源是如何合并的是实现细节，所以如果你想保证顺序，应使用连接工厂和操作符。

如果你有多于四个 `Observable<T>` 源，你可以使用 `Observable.mergeArray()` 来传递一个包含你想要合并的 `Observable[]` 实例的 varargs，如下面的代码片段所示。由于 RxJava 2.0 是为 JDK 6+ 编写的，并且没有访问 `@SafeVarargs` 注解，你可能会收到一些类型安全警告：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable<String> source1 =
          Observable.just("Alpha", "Beta");

        Observable<String> source2 =
          Observable.just("Gamma", "Delta");

        Observable<String> source3 =
          Observable.just("Epsilon", "Zeta");

        Observable<String> source4 =
          Observable.just("Eta", "Theta");

        Observable<String> source5 =
          Observable.just("Iota", "Kappa");

        Observable.mergeArray(source1, source2, source3, source4, 
source5)
            .subscribe(i -> System.out.println("RECEIVED: " + i));
     }
}

```

前述代码的输出如下：

```java
    RECEIVED: Alpha
    RECEIVED: Beta
    RECEIVED: Gamma
    RECEIVED: Delta
    RECEIVED: Epsilon
    RECEIVED: Zeta
    RECEIVED: Eta
    RECEIVED: Theta
    RECEIVED: Iota
    RECEIVED: Kappa
```

您还可以将 `Iterable<Observable<T>>` 传递给 `Observable.merge()`。它将合并该 `Iterable` 中所有的 `Observable<T>` 实例。我可以通过将这些源放入 `List<Observable<T>>` 并将它们传递给 `Observable.merge()` 来以前更安全的方式实现上述示例：

```java
import io.reactivex.Observable;
import java.util.Arrays;
import java.util.List;

public class Launcher {
      public static void main(String[] args) {

        Observable<String> source1 =
          Observable.just("Alpha", "Beta");

        Observable<String> source2 =
          Observable.just("Gamma", "Delta");

        Observable<String> source3 =
          Observable.just("Epsilon", "Zeta");

        Observable<String> source4 =
          Observable.just("Eta", "Theta");

        Observable<String> source5 =
          Observable.just("Iota", "Kappa");

        List<Observable<String>> sources =
          Arrays.asList(source1, source2, source3, source4, source5);

        Observable.merge(sources)
          .subscribe(i -> System.out.println("RECEIVED: " + i));
      }
}

```

`mergeArray()` 获得自己的方法而不是作为 `merge()` 的重载，原因是为了避免与 Java 8 编译器及其对函数类型的处理产生歧义。这对于所有 `xxxArray()` 操作符都适用。

`Observable.merge()` 与无限可观察对象一起工作。由于它将订阅所有可观察对象并在它们可用时立即触发它们的发射，因此您可以将多个无限源合并成一个单一的流。在这里，我们合并了两个分别以一秒和 300 毫秒间隔发射的 `Observable.interval()` 源。但在合并之前，我们使用发射的索引进行一些数学运算，以确定已经过去的时间，并将其与源名称一起以字符串形式发射。我们让这个过程运行三秒钟：

```java
import io.reactivex.Observable;
import java.util.concurrent.TimeUnit;

public class Launcher {
      public static void main(String[] args) {

        //emit every second
        Observable<String> source1 = Observable.interval(1, 
TimeUnit.SECONDS)
        .map(l -> l + 1) // emit elapsed seconds
        .map(l -> "Source1: " + l + " seconds");

        //emit every 300 milliseconds
        Observable<String> source2 =
          Observable.interval(300, TimeUnit.MILLISECONDS)
        .map(l -> (l + 1) * 300) // emit elapsed milliseconds
        .map(l -> "Source2: " + l + " milliseconds");

        //merge and subscribe
        Observable.merge(source1, source2)
.subscribe(System.out::println);
        //keep alive for 3 seconds
        sleep(3000);
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

上述代码的输出如下：

```java
    Source2: 300 milliseconds
    Source2: 600 milliseconds
    Source2: 900 milliseconds
    Source1: 1 seconds
    Source2: 1200 milliseconds
    Source2: 1500 milliseconds
    Source2: 1800 milliseconds
    Source1: 2 seconds
    Source2: 2100 milliseconds
    Source2: 2400 milliseconds
    Source2: 2700 milliseconds
    Source1: 3 seconds
    Source2: 3000 milliseconds
```

总结来说，`Observable.merge()` 将多个发射相同类型 `T` 的 `Observable<T>` 源合并并整合成一个单一的 `Observable<T>`。它适用于无限可观察对象，并且并不保证发射的顺序。如果您关心发射的严格顺序，并且希望每个 `Observable` 源按顺序触发，那么您可能希望使用 `Observable.concat()`，我们将在稍后介绍。

# `flatMap()`

在 RxJava 中，`flatMap()` 是最强大且至关重要的操作符之一。如果您必须投入时间来理解任何 RxJava 操作符，那么这个就是您需要关注的。它是一个执行动态 `Observable.merge()` 的操作符，通过将每个发射映射到一个 `Observable`。然后，它将结果可观察对象的发射合并成一个单一的流。

`flatMap()` 的最简单应用是将 *一个发射映射到多个发射*。比如说，我们想要发射来自 `Observable<String>` 的每个字符串中的字符。我们可以使用 `flatMap()` 来指定一个 `Function<T,Observable<R>>` lambda，它将每个字符串映射到一个 `Observable<String>`，这将发射字母。请注意，映射的 `Observable<R>` 可以发射任何类型的 `R`，这与源 `T` 的发射不同。在这个例子中，它恰好是 `String`，就像源一样：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable<String> source =
          Observable.just("Alpha", "Beta", "Gamma", "Delta", 
"Epsilon");

        source.flatMap(s -> Observable.fromArray(s.split("")))
          .subscribe(System.out::println);
      }
}
```

上述代码的输出如下：

```java
    A
    l
    p
    h
    a
    B
    e
    t
    a
    G
    a
    m
    m
    ...
```

我们已经将这五个字符串输出映射（通过 `flatMap()`）为每个输出中的字母。我们通过调用每个字符串的 `split()` 方法来实现这一点，并传递一个空的字符串参数 `""`，这将根据每个字符进行分隔。这返回一个包含所有字符的 `String[]` 数组，我们将其传递给 `Observable.fromArray()` 以输出每个字符。`flatMap()` 期望每个输出产生一个 `Observable`，然后它会合并所有生成的 `Observable` 并以单个流输出它们的值。

这里还有一个例子：让我们取一个 `String` 值序列（每个值都是一个由 `"/"` 分隔的值串联），对它们使用 `flatMap()`，然后在将它们转换为 `Integer` 输出之前只过滤出数值：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable<String> source =
          Observable.just("521934/2342/FOXTROT", "21962/12112/78886
            /TANGO",
"283242/4542/WHISKEY/2348562");

        source.flatMap(s -> Observable.fromArray(s.split("/")))
          .filter(s -> s.matches("[0-9]+")) //use regex to filter 
            integers
          .map(Integer::valueOf)
          .subscribe(System.out::println);
      }
}
```

上述代码的输出如下：

```java
    521934
    2342
    21962
    12112
    78886
    283242
    4542
    2348562
```

我们通过 `/` 字符将每个 `String` 分割，这产生了一个数组。我们将这个数组转换为一个 `Observable`，并使用 `flatMap()` 在它上面输出每个 `String`。我们使用正则表达式 `[0-9]+`（消除 `FOXTROT`、`TANGO` 和 `WHISKEY`）只过滤出数值字符串，然后将每个输出转换为 **`Integer`**。

就像 `Observable.merge()` 一样，你还可以将输出映射到无限 `Observable` 并合并它们。例如，我们可以从 `Observable<Integer>` 中输出简单的 `Integer` 值，但使用 `flatMap()` 来驱动 `Observable.interval()`，其中每个值都作为周期参数。在以下代码片段中，我们输出值 `2`、`3`、`10` 和 `7`，这将分别产生在 2 秒、3 秒、10 秒和 7 秒输出的间隔 `Observable`。这四个 `Observable` 将合并成一个单一的流：

```java
import io.reactivex.Observable;
import java.util.concurrent.TimeUnit;

public class Launcher {
      public static void main(String[] args) {

        Observable<Integer> intervalArguments =
          Observable.just(2, 3, 10, 7);

        intervalArguments.flatMap(i ->
Observable.interval(i, TimeUnit.SECONDS)
            .map(i2 -> i + "s interval: " + ((i + 1) * i) + " seconds 
              elapsed")
          ).subscribe(System.out::println);

          sleep(12000);
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

上述代码的输出如下：

```java
    2s interval: 2 seconds elapsed
    3s interval: 3 seconds elapsed
    2s interval: 4 seconds elapsed
    2s interval: 6 seconds elapsed
    3s interval: 6 seconds elapsed
    7s interval: 7 seconds elapsed
    2s interval: 8 seconds elapsed
    3s interval: 9 seconds elapsed
    2s interval: 10 seconds elapsed
    10s interval: 10 seconds elapsed
    2s interval: 12 seconds elapsed
    3s interval: 12 seconds elapsed
```

`Observable.merge()` 操作符将接受固定数量的 `Observable` 源。但 `flatMap()` 会根据每个输入的输出动态地添加新的 `Observable` 源。这意味着你可以随着时间的推移持续合并新的输入 `Observable`。

关于 `flatMap()` 的另一个快速笔记是它可以以许多巧妙的方式使用。时至今日，我仍在不断发现使用它的新方法。但你可以用另一种方式发挥创意，那就是在 `flatMap()` 中评估每个输出，并确定你想要返回哪种类型的 `Observable`。例如，如果我的前一个例子向 `flatMap()` 输出了一个 `0`，这将破坏生成的 `Observable.interval()`。但我可以使用一个 `if` 语句来检查它是否为 `0`，并返回 `Observable.empty()`，就像以下代码片段中所示：

```java
Observable<Integer> secondIntervals =
Observable.just(2, 0, 3, 10, 7);

secondIntervals.flatMap(i -> {
if (i == 0)
return Observable.empty();
else
return Observable.interval(i, TimeUnit.SECONDS)
.map(l -> i + "s interval: " + ((l + 1) * i) + " seconds 
elapsed");
}).subscribe(System.out::println);
```

当然，这可能有点过于巧妙，因为你可以在 `flatMap()` 前面直接使用 `filter()` 来过滤掉等于 `0` 的输出。但重点是，你可以在 `flatMap()` 中评估一个输出，并确定你想要返回哪种 `Observable`。

`flatMap()` 也是将热 Observables UI 事件流（如 JavaFX 或 Android 按钮点击）扁平映射到 `flatMap()` 内部整个过程的绝佳方式。失败和错误恢复可以完全在 `flatMap()` 内部处理，因此每个流程实例都不会干扰未来的按钮点击。

如果你不想快速按钮点击产生多个冗余的流程实例，你可以使用 `doOnNext()` 禁用按钮，或者利用 `switchMap()` 杀死之前的流程，我们将在第七章切换、节流、窗口化和缓冲中讨论。 

注意，`flatMap()` 有许多变体和风味，接受许多重载，为了简洁起见，我们不会深入探讨。我们可以传递一个第二个组合器参数，它是一个 `BiFunction<T,U,R>` lambda，将最初发射的 `T` 值与每个扁平映射的 `U` 值关联，并将两者都转换为 `R` 值。在我们的早期示例中，我们可以将每个字母与它映射的原始字符串发射关联起来：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {
        Observable<String> source =
          Observable.just("Alpha", "Beta", "Gamma", "Delta", 
"Epsilon");

        source.flatMap(s -> Observable.fromArray(s.split("")), (s,r) -> 
s + "-" + r)
            .subscribe(System.out::println);
      }
}
```

上述代码的输出如下：

```java
    Alpha-A
    Alpha-l
    Alpha-p
    Alpha-h
    Alpha-a
    Beta-B
    Beta-e
    Beta-t
    Beta-a
    Gamma-G
    ...

```

我们还可以使用 `flatMapIterable()` 将每个 `T` 发射映射到 `Iterable<R>` 而不是 `Observable<R>`。然后它将为每个 `Iterable<R>` 发射所有的 `R` 值，从而节省我们将其转换为 `Observable` 的步骤和开销。还有将到 Singles `(flatMapSingle())`、Maybes **`(flatMapMaybe())`** 和 Completables `(flatMapCompletable())` 的 `flatMap()` 变体。许多这些重载也适用于 `concatMap()`，我们将在下一节中介绍。

# 连接操作

连接操作与合并操作非常相似，但有一个重要的细微差别：它将按顺序依次发射每个提供的 `Observable` 的元素，并且按照指定的顺序。它将在当前 `Observable` 调用 `onComplete()` 之前不会移动到下一个 `Observable`。这使得它非常适合确保合并的 Observables 以保证的顺序发射它们的发射。然而，对于无限 Observables 来说，这通常是一个较差的选择，因为无限的 `Observable` 将无限期地阻塞队列，并永远让后续的 Observables 等待。

我们将介绍用于连接操作的工厂和算子。你会发现它们与合并操作非常相似，只是它们具有顺序行为。

当你想要确保 Observables 按顺序发射它们的发射时，你应该选择连接操作。如果你不关心顺序，则选择合并操作。

# `Observable.concat()` 和 `concatWith()`

`Observable.concat()` 工厂是 `Observable.merge()` 的连接等价物。它将组合多个 Observables 的发射，但将按顺序依次发射每个，并且只有在调用 `onComplete()` 之后才会移动到下一个。

在以下代码中，我们有两个源可观察对象（Observables）发出字符串。我们可以使用`Observable.concat()`来触发第一个可观察对象的发射，然后触发第二个可观察对象的发射：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable<String> source1 =
          Observable.just("Alpha", "Beta", "Gamma", "Delta", 
"Epsilon");

        Observable<String> source2 =
          Observable.just("Zeta", "Eta", "Theta");

        Observable.concat(source1, source2)
          .subscribe(i -> System.out.println("RECEIVED: " + i));
      }
}
```

上述代码的输出如下：

```java
    RECEIVED: Alpha
    RECEIVED: Beta
    RECEIVED: Gamma
    RECEIVED: Delta
    RECEIVED: Epsilon
    RECEIVED: Zeta
    RECEIVED: Eta
    RECEIVED: Theta
```

这与之前我们的`Observable.merge()`示例的输出相同。但如合并部分所述，我们应该使用`Observable.concat()`来保证发射顺序，因为合并不保证顺序。你还可以使用`concatWith()`操作符来完成相同的事情，如下面的代码行所示：

```java
    source1.concatWith(source2)
      .subscribe(i -> System.out.println("RECEIVED: " + i));
```

如果我们使用`Observable.concat()`与无限可观察对象，它将永远从遇到的第一个可观察对象发出，并阻止任何后续的可观察对象触发。如果我们想在连接操作中任何地方放置一个无限`Observable`，它可能被指定为最后一个。这确保了它不会阻止其后的任何可观察对象，因为后面没有可观察对象。我们还可以使用`take()`操作符将无限可观察对象变为有限。

在这里，我们触发一个每秒发出一次的可观察对象，但只从它那里获取两个发射。之后，它将调用`onComplete()`并销毁它。然后，一个在它之后连接的可观察对象将永远发出（或在这种情况下，在应用在五秒后退出时）。由于这个第二个可观察对象是`Observable.concat()`中指定的最后一个，它不会因为无限而阻止任何后续的可观察对象：

```java
import io.reactivex.Observable;
import java.util.concurrent.TimeUnit;

public class Launcher {
    public static void main(String[] args) {

        //emit every second, but only take 2 emissions
        Observable<String> source1 =
          Observable.interval(1, TimeUnit.SECONDS)
            .take(2)
            .map(l -> l + 1) // emit elapsed seconds
            .map(l -> "Source1: " + l + " seconds");

          //emit every 300 milliseconds
          Observable<String> source2 =
            Observable.interval(300, TimeUnit.MILLISECONDS)
              .map(l -> (l + 1) * 300) // emit elapsed milliseconds
              .map(l -> "Source2: " + l + " milliseconds");

          Observable.concat(source1, source2)
            .subscribe(i -> System.out.println("RECEIVED: " + i));

          //keep application alive for 5 seconds
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

上述代码的输出如下：

```java
RECEIVED: Source1: 1 seconds
RECEIVED: Source1: 2 seconds
RECEIVED: Source2: 300 milliseconds
RECEIVED: Source2: 600 milliseconds
RECEIVED: Source2: 900 milliseconds
RECEIVED: Source2: 1200 milliseconds
RECEIVED: Source2: 1500 milliseconds

```

对于数组和`Iterable<Observable<T>>`输入，也存在与合并相同的连接对应物。`Observable.concatArray()`工厂将按顺序在`Observable[]`数组中触发每个`Observable`。`Observable.concat()`工厂也将接受一个`Iterable<Observable<T>>`，并以相同的方式触发每个`Observable<T**>**`。

注意，`concatMap()`有几个变体。当你想要将每个发射映射到`Iterable<T>`而不是`Observable<T>`时，使用`concatMapIterable()`。它将为每个`Iterable<T>`发出所有的`T`值，从而节省了将每个值转换为`Observable<T>`的步骤和开销。还有一个`concatMapEager()`操作符，它将贪婪地订阅它接收到的所有`Observable`源，并将发射缓存起来，直到轮到它们发射。

# concatMap()

正如存在`flatMap()`，它动态合并从每个发射派生的可观察对象一样，还有一个称为`concatMap()`的连接对应物。如果你关心顺序，并且想要每个从每个发射映射的可观察对象在开始下一个之前完成，你应该优先使用此操作符。更具体地说，`concatMap()`将按顺序合并每个映射的可观察对象，并逐个触发它们。它只有在当前的可观察对象调用`onComplete()`时才会移动到下一个可观察对象。如果源发射的可观察对象比`concatMap()`从它们发出要快，那么这些可观察对象将被排队。

我们之前的`flatMap()`示例如果我们要显式关注发射顺序，将更适合使用`concatMap()`。尽管我们这里的示例与`flatMap()`示例具有相同的输出，但当我们显式关注保持顺序并希望按顺序处理每个映射的`Observable`时，我们应该使用`concatMap()`：

```java
import io.reactivex.Observable;

public class Launcher {
    public static void main(String[] args) {

        Observable<String> source =
                Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon");

        source.concatMap(s -> Observable.fromArray(s.split("")))
                .subscribe(System.out::println);
    }
}

```

输出结果如下：

```java
A
l
p
h
a
B
e
t
a
G
a
m
m
...
```

再次强调，您不太可能想要将`concatMap()`映射到无限`Observable`。正如您所猜测的，这将导致后续的`Observable`永远不会触发。您可能会想使用`flatMap()`，我们将在第六章的并发示例中看到它的使用，*并发与并行化*。

# 模糊

在介绍了合并和连接之后，让我们先处理一个简单的组合操作。`Observable.amb()`工厂（**amb**代表**模糊**）将接受一个`Iterable<Observable<T>>`，并发射第一个发射的`Observable`的发射，而其他`Observable`将被销毁。第一个发射的`Observable`是发射通过的那个。这在您有多个相同数据或事件源且希望最快的一个获胜时很有用。

在这里，我们有两个间隔源，我们使用`Observable.amb()`工厂将它们组合起来。如果一个每秒发射一次，而另一个每 300 毫秒发射一次，那么后者将会获胜，因为它会先发射：

```java
import io.reactivex.Observable;
import java.util.Arrays;
import java.util.concurrent.TimeUnit;

public class Launcher {
    public static void main(String[] args) {

        //emit every second
        Observable<String> source1 =
                Observable.interval(1, TimeUnit.SECONDS)
                        .take(2)
                        .map(l -> l + 1) // emit elapsed seconds
                        .map(l -> "Source1: " + l + " seconds");

        //emit every 300 milliseconds
        Observable<String> source2 =
                Observable.interval(300, TimeUnit.MILLISECONDS)
                        .map(l -> (l + 1) * 300) // emit elapsed milliseconds
                        .map(l -> "Source2: " + l + " milliseconds");

        //emit Observable that emits first
        Observable.amb(Arrays.asList(source1, source2))
                .subscribe(i -> System.out.println("RECEIVED: " + i));

        //keep application alive for 5 seconds
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
RECEIVED: Source2: 300 milliseconds
RECEIVED: Source2: 600 milliseconds
RECEIVED: Source2: 900 milliseconds
RECEIVED: Source2: 1200 milliseconds
RECEIVED: Source2: 1500 milliseconds
RECEIVED: Source2: 1800 milliseconds
RECEIVED: Source2: 2100 milliseconds
...
```

您还可以使用`ambWith()`运算符，它将实现相同的结果：

```java
//emit Observable that emits first
source1.ambWith(source2)
        .subscribe(i -> System.out.println("RECEIVED: " + i));
```

您还可以使用`Observable.ambArray()`来指定一个`varargs`数组而不是`Iterable<Observable<T>>`。

# 压缩

压缩允许您从每个`Observable`源中取出发射，并将其组合成一个单一的发射。每个`Observable`可以发射不同的类型，但您可以将这些不同发射的类型组合成一个单一的发射。以下是一个示例，如果我们有一个`Observable<String>`和一个`Observable<Integer>`，我们可以将每个`String`和`Integer`配对成一个一对一的配对，并用 lambda 函数连接它们：

```java
import io.reactivex.Observable;

public class Launcher {
    public static void main(String[] args) {

        Observable<String> source1 =
                Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon");

        Observable<Integer> source2 = Observable.range(1,6);

        Observable.zip(source1, source2, (s,i) -> s + "-" + i)
                .subscribe(System.out::println);
    }
}
```

输出结果如下：

```java
Alpha-1
Beta-2
Gamma-3
Delta-4
Epsilon-5
```

`zip()`函数接收了`Alpha`和`1`，然后将它们配对成一个由破折号`-`分隔的连接字符串，并将其推向前。然后，它接收`Beta`和`2`，并将它们作为连接发射出去，依此类推。一个`Observable`的发射必须等待与另一个`Observable`的发射配对。如果一个`Observable`调用`onComplete()`而另一个`Observable`仍有等待配对的发射，那么这些发射将简单地丢弃，因为它们没有可以配对的。这就是为什么`6`发射发生了，因为我们只有五个字符串发射。

您也可以使用`zipWith()`运算符来完成此操作，如下所示：

```java
source1.zipWith(source2, (s,i) -> s + "-" + i)
```

你可以向 `Observable.zip()` 工厂传递多达九个 `Observable` 实例。如果你需要更多，你可以传递一个 `Iterable<Observable<T>>` 或使用 `zipArray()` 来提供一个 `Observable[]` 数组。请注意，如果一个或多个源产生的发射速度比另一个快，`zip()` 将在等待较慢的源提供发射时排队快速发射。这可能会导致不希望的性能问题，因为每个源都会在内存中排队。如果你只关心将每个源的最近一次发射压缩在一起，而不是赶上整个队列，你将想要使用 `combineLatest()`，我们将在本节稍后介绍。

使用 `Observable.zipIterable()` 传递一个布尔值 `delayError` 参数以延迟错误，直到所有源终止，并传递一个整型 `bufferSize` 以提示每个源期望的元素数量以优化队列大小。你可以指定后者，在某些场景中通过在压缩之前缓冲发射来提高性能。

使用 `Observable.interval()` 也可以通过压缩来减慢发射。在这里，我们以 1 秒的间隔压缩每个字符串。这将使每个字符串发射延迟一秒，但请注意，五个字符串发射可能会排队等待间隔发射配对：

```java
import io.reactivex.Observable;
import java.time.LocalTime;
import java.util.concurrent.TimeUnit;

public class Launcher {
    public static void main(String[] args) {

        Observable<String> strings =
                Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon");

        Observable<Long> seconds =
                Observable.interval(1, TimeUnit.SECONDS);

        Observable.zip(strings,seconds, (s,l) -> s)
                .subscribe(s -> 
System.out.println("Received " + s + 
                                " at " + LocalTime.now())
                );

        sleep(6000);
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
Received Alpha at 13:28:28.428
Received Beta at 13:28:29.388
Received Gamma at 13:28:30.389
Received Delta at 13:28:31.389
Received Epsilon at 13:28:32.389
```

# 组合最新

`Observable.combineLatest()` 工厂与 `zip()` 有一些相似之处，但对于每个源发射的每个发射，它都会立即与每个其他源的最近一次发射耦合。它不会为每个源排队未配对的发射，而是缓存并配对最新的一个。

在这里，让我们使用两个间隔观察者之间的 `Observable.combineLatest()`，第一个每 300 毫秒发射一次，另一个每秒发射一次：

```java
import io.reactivex.Observable;
import java.util.concurrent.TimeUnit;

public class Launcher {
    public static void main(String[] args) {

        Observable<Long> source1 =
                Observable.interval(300, TimeUnit.MILLISECONDS);

        Observable<Long> source2 =
                Observable.interval(1, TimeUnit.SECONDS);

        Observable.combineLatest(source1, source2,
                (l1,l2) -> "SOURCE 1: " + l1 + "  SOURCE 2: " + l2)
                .subscribe(System.out::println);

        sleep(3000);
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
SOURCE 1: 2  SOURCE 2: 0
SOURCE 1: 3  SOURCE 2: 0
SOURCE 1: 4  SOURCE 2: 0
SOURCE 1: 5  SOURCE 2: 0
SOURCE 1: 5  SOURCE 2: 1
SOURCE 1: 6  SOURCE 2: 1
SOURCE 1: 7  SOURCE 2: 1
SOURCE 1: 8  SOURCE 2: 1
SOURCE 1: 9  SOURCE 2: 1
SOURCE 1: 9  SOURCE 2: 2
```

这里发生了很多事情，但让我们尝试将其分解。`source1` 每 300 毫秒发射一次，但前两次发射还没有与每秒发射一次的 `source2` 中的任何内容配对，并且还没有发生任何发射。最后，经过一秒钟后，`source2` 推出其第一次发射 `0`，并与 `source1` 的最新发射 `2`（第三次发射）配对。请注意，`source1` 的前两次发射 `0` 和 `1` 被完全遗忘，因为第三次发射 `2` 现在是最新发射。然后，`source1` 每 300 毫秒发射 `3`、`4` 和 `5`，但 `0` 仍然是 `source2` 的最新发射，所以所有三个都与它配对。然后，`source2` 发射其第二次发射 `1`，并与 `source2` 的最新发射 `5` 配对。

简单来说，当一个源触发时，它会与来自其他源的最近一次发射耦合。`Observable.combineLatest()` 在组合 UI 输入时特别有用，因为之前的用户输入通常是不相关的，只有最新的输入才是关注的重点。

# withLatestFrom()

与 `Observable.combineLatest()` 类似，但并不完全相同的是 `withLatestfrom()` 操作符。它将映射每个 `T` 发射与其他 Observables 的最新值，并将它们组合起来，但它将只从每个其他 Observables 中取 *一个* 发射：

```java
import io.reactivex.Observable;
import java.util.concurrent.TimeUnit;

public class Launcher {
    public static void main(String[] args) {

        Observable<Long> source1 =
                Observable.interval(300, TimeUnit.MILLISECONDS);

        Observable<Long> source2 =
                Observable.interval(1, TimeUnit.SECONDS);

        source2.withLatestFrom(source1,
                (l1,l2) -> "SOURCE 2: " + l1 + "  SOURCE 1: " + l2
        ) .subscribe(System.out::println);

        sleep(3000);
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
SOURCE 2: 0  SOURCE 1: 2
SOURCE 2: 1  SOURCE 1: 5
SOURCE 2: 2  SOURCE 1: 9
```

如您所见，`source2` 每秒发射一次，而 `source1` 每隔 300 毫秒发射一次。当您在 `source2` 上调用 `withLatestFrom()` 并传递 `source1` 时，它将与 `source1` 的最新发射结合，但它不会关心任何之前的或随后的发射。

您可以向 `withLatestFrom()` 传递最多四个任何类型的 `Observable` 实例。如果您需要更多，您可以传递一个 `Iterable<Observable<T>>`。

# 分组

使用 RxJava 可以实现的一个强大操作是将发射按指定的键分组到单独的 Observables 中。这可以通过调用 `groupBy()` 操作符来实现，它接受一个将每个发射映射到键的 lambda 表达式。然后它将返回一个 `Observable<GroupedObservable<K,T>>`，它发出一种特殊类型的 `Observable`，称为 `GroupedObservable`。`GroupedObservable<K,T>` 就像任何其他 `Observable` 一样，但它有一个可访问的键 `K` 值作为属性。它将发出映射给该特定键的 `T` 发射。

例如，我们可以使用 `groupBy()` 操作符来按每个 `String` 的长度对 `Observable<String>` 的发射进行分组。我们稍后将订阅它，但这里是如何声明它的：

```java
import io.reactivex.Observable;
import io.reactivex.observables.GroupedObservable;

public class Launcher {
    public static void main(String[] args) {

        Observable<String> source = 
Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon");

        Observable<GroupedObservable<Integer,String>> byLengths =
                source.groupBy(s -> s.length());
    }
}
```

我们可能需要在每个 `GroupedObservable` 上使用 `flatMap()`，但在那个 `flatMap()` 操作中，我们可能希望减少或收集那些具有相同键的发射（因为这将返回一个 `Single`，我们需要使用 `flatMapSingle()`）。让我们调用 `toList()` 以便我们可以将发射作为按长度分组的列表发出：

```java
import io.reactivex.Observable;
import io.reactivex.observables.GroupedObservable;

public class Launcher {
    public static void main(String[] args) {

        Observable<String> source =
                Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon");

        Observable<GroupedObservable<Integer,String>> byLengths =
                source.groupBy(s -> s.length());

        byLengths.flatMapSingle(grp -> grp.toList())
                .subscribe(System.out::println);
    }
}
```

输出如下：

```java
[Beta]
[Alpha, Gamma, Delta]
[Epsilon]
```

`Beta` 是唯一长度为四的发射，所以它是该长度键列表中的唯一元素。`Alpha`、`Beta` 和 `Gamma` 都有五个长度，所以它们是从同一个 `GroupedObservable` 发射的，这些发射是为长度五而发射的，并被收集到同一个列表中。`Epsilon` 是唯一长度为七的发射，所以它是其列表中的唯一元素。

请记住，`GroupedObservable` 也有一个 `getKey()` 方法，它返回与该 `GroupedObservable` 相关联的键值。如果我们想简单地连接每个 `GroupedObservable` 的 `String` 发射，然后将 `length` 键以这种形式连接起来，我们可以这样做：

```java
import io.reactivex.Observable;
import io.reactivex.observables.GroupedObservable;

public class Launcher {
    public static void main(String[] args) {

        Observable<String> source =
                Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon");

        Observable<GroupedObservable<Integer,String>> byLengths =
                source.groupBy(s -> s.length());

        byLengths.flatMapSingle(grp ->
grp.reduce("",(x,y) -> x.equals("") ? y : x + ", " + y)
                        .map(s -> grp.getKey() + ": " + s)
        ).subscribe(System.out::println);
    }
}
```

输出如下：

```java
4: Beta
5: Alpha, Gamma, Delta
7: Epsilon
```

仔细注意，`GroupedObservables` 是一种热和冷 `Observable` 的奇怪组合。它们之所以不冷，是因为它们不会将未播放的发射重放到第二个 `Observer`，但它们会缓存发射并将它们刷新到第一个 `Observer`，确保没有丢失。如果您需要重放发射，可以将它们收集到一个列表中，就像我们之前做的那样，然后对该列表执行操作。您还可以使用缓存操作符，我们将在下一章中学习这些操作符。

# 摘要

在本章中，我们介绍了以各种有用的方式组合 `Observable`。合并有助于组合和同时触发多个 `Observable`，并将它们的发射合并成一个单一的数据流。`flatMap()` 操作符特别关键，因为动态合并从发射中派生的 `Observable` 在 RxJava 中打开了大量有用的功能。连接类似于合并，但它按顺序触发源 `Observable`，而不是一次性触发。与模糊组合一起使用，我们可以选择第一个发射并触发其发射的 `Observable`。压缩允许您将多个 `Observable` 的发射组合在一起，而 `combineLatest` 则在每个源触发时将每个源的最新发射合并在一起。最后，分组允许您将一个 `Observable` 分割成几个 `GroupedObservables`，每个 `GroupedObservables` 都有具有公共键的发射。

抽时间探索组合 `Observable` 并进行实验，看看它们是如何工作的。它们对于解锁 RxJava 的功能以及快速表达事件和数据转换至关重要。当我们介绍并发性时，我们将在第六章 `并发和并行化` 中查看一些使用 `flatMap()` 的强大应用，我们还将介绍如何进行多任务处理和并行化。
