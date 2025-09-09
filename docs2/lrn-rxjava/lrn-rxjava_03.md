# 基本操作符

在上一章中，你学到了很多关于 `Observable` 和 `Observer` 的知识。我们还介绍了一些操作符，特别是 `map()` 和 `filter()`，以了解操作符的作用。但我们可以利用数百个 RxJava 操作符来表达业务逻辑和行为。我们将在这本书的大部分内容中全面介绍操作符，这样你知道何时使用哪些操作符。了解可用的操作符并将它们组合起来对于成功使用 ReactiveX 至关重要。你应该努力使用操作符来表达业务逻辑，以便你的代码尽可能保持反应式。

应该注意的是，操作符本身是它们所调用的 `Observable` 的观察者。如果你在 `Observable` 上调用 `map()`，返回的 `Observable` 将订阅它。然后它将转换每个输出，并依次成为下游观察者（包括其他操作符和终端 `Observer`）的生产者。

你应该努力使用 RxJava 操作符执行尽可能多的逻辑，并使用 `Observer` 来接收准备消费的最终产品输出。尽量别从 `Observable` 链中提取值，或者求助于阻塞过程或命令式编程策略。当你保持算法和过程反应式时，你可以轻松利用反应式编程的好处，如降低内存使用、灵活的并发性和可丢弃性。

在本章中，我们将介绍以下主题：

+   抑制操作符

+   转换操作符

+   减法操作符

+   错误恢复操作符

+   动作操作符

# 抑制操作符

有许多操作符可以抑制不符合特定标准的输出。这些操作符通过简单地不对不合格的输出调用下游的 `onNext()` 函数来实现，因此不会向下传递到 `Observer`。我们已经看到了 `filter()` 操作符，这可能是最常见的抑制操作符。我们将从这个开始。

# filter()

`filter()` 操作符接受 `Predicate<T>` 用于给定的 `Observable<T>`。这意味着你提供一个 lambda 表达式，通过映射每个输出到布尔值来验证每个输出，带有 false 的输出将不会继续。

例如，你可以使用 `filter()` 来只允许长度不是五个字符的字符串输出：

```java
import io.reactivex.Observable;

public class Launcher {
    public static void main(String[] args) {

      Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
      .filter(s -> s.length() != 5)
      subscribe(s -> System.out.println("RECEIVED: " + s));
    }
} 
```

上述代码片段的输出如下：

```java
RECEIVED: Beta
RECEIVED: Epsilon
```

`filter()` 函数可能是最常用的用于抑制输出的操作符。

注意，如果所有输出都无法满足你的标准，返回的 `Observable` 将为空，且在调用 `onComplete()` 之前不会发生任何输出。

# take()

`take()`操作符有两个重载版本。其中一个将获取指定数量的发射，并在捕获所有这些发射后调用`onComplete()`。它还将销毁整个订阅，以便不再发生更多发射。例如，`take(3)`将发射前三个发射，然后调用`onComplete()`事件：

```java
import io.reactivex.Observable;

public class Launcher {
    public static void main(String[] args) {

      Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
      .take(3)
      .subscribe(s -> System.out.println("RECEIVED: " + s));
    }
}
```

上一代码片段的输出如下：

```java
    RECEIVED: Alpha
    RECEIVED: Beta
    RECEIVED: Gamma
```

注意，如果你在`take()`函数中指定的发射少于你得到的，它将简单地发射它所得到的，然后调用`onComplete()`函数。

另一个重载版本将获取特定时间持续时间内的发射，然后调用`onComplete()`。当然，我们这里的冷`Observable`会非常快地发射，这会是一个不好的例子。也许使用`Observable.interval()`函数会更好。让我们以每`300`毫秒发射一次，但在以下代码片段中只对`2`秒内的发射进行`take()`操作：

```java
import io.reactivex.Observable;
import java.util.concurrent.TimeUnit;

public class Launcher {
      public static void main(String[] args) {

        Observable.interval(300, TimeUnit.MILLISECONDS)
          .take(2, TimeUnit.SECONDS)
          .subscribe(i -> System.out.println("RECEIVED: " + i));

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

上一代码片段的输出如下：

```java
RECEIVED: 0
RECEIVED: 1
RECEIVED: 2
RECEIVED: 3
RECEIVED: 4
RECEIVED: 5
```

你可能会得到这里显示的输出（每个打印每`300`毫秒发生一次）。如果它们以`300`毫秒的间隔分散，那么在`2`秒内你只能得到六个发射。

注意，还有一个`takeLast()`操作符，它将在调用`onComplete()`函数之前获取指定数量的最后发射（或时间持续时间）。只需记住，它将内部排队发射，直到其`onComplete()`函数被调用，然后它可以逻辑上识别并发射最后的发射。

# `skip()`

`skip()`操作符与`take()`操作符相反。它将忽略指定数量的发射，然后发射后续的发射。如果我想跳过一个`Observable`的前`90`个发射，我可以使用这个操作符，如下所示：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.range(1,100)
          .skip(90)
          .subscribe(i -> System.out.println("RECEIVED: " + i));
      }
}
```

以下代码片段的输出如下：

```java
RECEIVED: 91
RECEIVED: 92
RECEIVED: 93
RECEIVED: 94
RECEIVED: 95
RECEIVED: 96
RECEIVED: 97
RECEIVED: 98
RECEIVED: 99
RECEIVED: 100
```

就像`take()`操作符一样，还有一个接受时间持续的重载。还有一个`skipLast()`操作符，它将在调用`onComplete()`事件之前跳过指定数量的项目（或时间持续时间）。只需记住，`skipLast()`操作符将排队并延迟发射，直到它确认该范围内的最后发射。

# `takeWhile()` 和 `skipWhile()`

`take()`操作符的另一个变体是`takeWhile()`操作符，它将在从每个发射推导出的条件为真时获取发射。以下示例将在发射小于`5`时持续获取发射。一旦遇到一个不满足条件的发射，它将调用`onComplete()`函数并销毁它：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.range(1,100)
          .takeWhile(i -> i < 5)
          .subscribe(i -> System.out.println("RECEIVED: " + i));
      }
}
```

上一代码片段的输出如下：

```java
RECEIVED: 1
RECEIVED: 2
RECEIVED: 3
RECEIVED: 4
```

就像`takeWhile()`函数一样，还有一个`skipWhile()`函数。它将一直跳过满足条件的排放项。一旦条件不再满足，排放项将开始通过。在下面的代码中，只要排放项小于或等于`95`，就跳过排放项。一旦遇到不满足此条件的排放项，它将允许所有后续的排放项向前传递：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.range(1,100)
          .skipWhile(i -> i <= 95)
          .subscribe(i -> System.out.println("RECEIVED: " + i));
      }
}

```

上述代码片段的输出如下：

```java
RECEIVED: 96
RECEIVED: 97
RECEIVED: 98
RECEIVED: 99
RECEIVED: 100
```

`takeUntil()`操作符与`takeWhile()`类似，但它接受另一个`Observable`作为参数。它将一直接收排放项，直到那个其他`Observable`推送一个排放项。`skipUntil()`操作符具有类似的行为。它也接受另一个`Observable`作为参数，但它将一直跳过，直到其他`Observable`发出一些内容。

# distinct()

`distinct()`操作符会排放每个唯一的排放项，但它会抑制随后出现的任何重复项。等价性基于排放对象的`hashCode()/equals()`实现。如果我们想排放字符串序列的唯一长度，可以这样做：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
          .map(String::length)
          .distinct()
          .subscribe(i -> System.out.println("RECEIVED: " + i));
      }
}

```

上述代码片段的输出如下：

```java
RECEIVED: 5
RECEIVED: 4
RECEIVED: 7
```

请记住，如果您有一系列广泛的、独特的值，`distinct()`可能会使用一些内存。想象一下，每个订阅都会产生一个`HashSet`，用于跟踪之前捕获的唯一值。

您还可以添加一个 lambda 参数，将每个排放项映射到一个用于等价逻辑的键。这允许排放项（而不是键）向前传递，同时使用键进行不同的逻辑。例如，我们可以根据每个字符串的长度进行键控，并使用它来保证唯一性，但排放字符串而不是它们的长度：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
          .distinct(String::length)
          .subscribe(i -> System.out.println("RECEIVED: " + i));
      }
}

```

上述代码片段的输出如下：

```java
RECEIVED: Alpha
RECEIVED: Beta
RECEIVED: Epsilon
```

`Alpha`是五个字符，`Beta`是四个。`Gamma`和`Delta`被忽略，因为`Alpha`已经被排放，并且是 5 个字符。`Epsilon`是七个字符，因为还没有排放过七个字符的字符串，所以它被向前排放。

# distinctUntilChanged()

`distinctUntilChanged()`函数会忽略连续的重复排放项。这是一种忽略重复直到它们改变的有用方法。如果正在重复排放相同的值，所有重复项都将被忽略，直到发射一个新的值。下一个值的重复项将被忽略，直到它再次改变，依此类推。观察以下代码的输出，以查看此行为：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just(1, 1, 1, 2, 2, 3, 3, 2, 1, 1)
          .distinctUntilChanged()
          .subscribe(i -> System.out.println("RECEIVED: " + i));
      }
}

```

上述代码片段的输出如下：

```java
RECEIVED: 1
RECEIVED: 2
RECEIVED: 3
RECEIVED: 2
RECEIVED: 1
```

我们首先接收一个`1`的排放，这是允许的。但接下来的两个`1`被忽略，因为它们是连续的重复项。当它切换到`2`时，那个初始的`2`被排放，但随后的重复项被忽略。排放一个`3`，其随后的重复项也被忽略。最后，我们切换回一个排放的`2`，然后是一个重复项被忽略的`1`。

就像`distinct()`一样，你可以通过一个 lambda 映射提供一个可选的键参数。在以下代码片段中，我们使用基于字符串长度的键执行`distinctUntilChanged()`操作：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Zeta", "Eta", "Gamma", 
"Delta")
          .distinctUntilChanged(String::length)
          .subscribe(i -> System.out.println("RECEIVED: " + i));
      }
}
```

上述代码片段的输出如下：

```java
RECEIVED: Alpha
RECEIVED: Beta
RECEIVED: Eta
RECEIVED: Gamma
```

注意，由于`Zeta`紧随`Beta`之后，而`Beta`也是四个字符，因此跳过了`Zeta`。同样，由于`Delta`紧随`Gamma`之后，而`Gamma`也是五个字符，因此也忽略了`Delta`。

# elementAt()

你可以通过指定一个 Long 类型的索引来获取特定的发射项，索引从`0`开始。找到并发出该项目后，将调用`onComplete()`，并取消订阅。

如果你想从`Observable`中获取第四个发射项，你可以按照以下代码片段所示进行操作：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Zeta", "Eta", "Gamma", 
"Delta")
            .elementAt(3)
            .subscribe(i -> System.out.println("RECEIVED: " + i));
      }
}
```

以下代码片段的输出如下：

```java
RECEIVED: Eta
```

你可能没有注意到，`elementAt()`返回`Maybe<T>`而不是`Observable<T>`。这是因为它将产生一个发射项，但如果发射项少于所求索引，它将是空的。

`elementAt()`还有其他变体，例如`elementAtOrError()`，它返回一个`Single`，如果在该索引处找不到元素，则会发出错误。`singleElement()`将`Observable`转换为`Maybe`，但如果存在超过一个元素，则会产生错误。最后，`firstElement()`和`lastElement()`将分别产生第一个或最后一个发射项。

# 转换操作符

接下来，我们将介绍各种常见的转换发射项的操作符。在`Observable`链中的操作符系列是一系列转换。你已经看到了`map()`，这是这一类别中最明显的操作符。我们将从这个开始。

# map()

对于给定的`Observable<T>`，`map()`操作符将使用提供的`Function<T,R>` lambda 将`T`发射项转换为`R`发射项。我们已经多次使用此操作符，将字符串转换为长度。以下是一个新示例：我们可以将原始日期字符串作为输入，并使用`map()`操作符将每个字符串转换为`LocalDate`发射项，如下所示：

```java
import io.reactivex.Observable;
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

public class Launcher {
      public static void main(String[] args) {

        DateTimeFormatter dtf = DateTimeFormatter.ofPattern("M/d
          /yyyy");

        Observable.just("1/3/2016", "5/9/2016", "10/12/2016")
          .map(s -> LocalDate.parse(s, dtf))
          .subscribe(i -> System.out.println("RECEIVED: " + i));
      }
}
```

上述代码片段的输出如下：

```java
RECEIVED: 2016-01-03
RECEIVED: 2016-05-09
RECEIVED: 2016-10-12
```

我们传递了一个将每个字符串转换为`LocalDate`对象的 lambda 表达式。我们提前创建了一个`DateTimeFormatter`，以便协助进行`LocalDate.parse()`操作，该操作返回一个`LocalDate`。然后，我们将每个`LocalDate`发射项推送到我们的`Observer`以进行打印。

`map()`操作符对每个发射项进行一对一的转换。如果你需要执行一对一转换（将一个发射项转换为多个发射项），你可能希望使用`flatMap()`或`concatMap()`，我们将在下一章中介绍。

# cast()

一个简单的、类似映射的操作符，用于将每个发射项转换为不同类型的是`cast()`。如果我们想将`Observable<String>`转换为对象（并返回一个`Observable<Object>`），我们可以使用`map()`操作符，如下所示：

```java
Observable<Object> items = 
  Observable.just("Alpha", "Beta", "Gamma").map(s -> (Object) s);
```

但我们可以使用的一个简写是`cast()`，我们可以简单地传递我们想要转换到的类类型，如下面的代码片段所示：

```java
Observable<Object> items = 
  Observable.just("Alpha", "Beta", "Gamma").cast(Object.class);
```

如果你发现由于继承或多态类型混合而出现打字问题，这是一个将所有内容强制转换为公共基类型的有效暴力方法。但首先努力正确使用泛型和适当使用类型通配符。

# `startWith()`

对于给定的`Observable<T>`，`startWith()`操作符允许你插入一个在所有其他发射之前发生的`T`发射。例如，如果我们有一个`Observable<String>`，它发射我们想要打印的菜单项，我们可以使用`startWith()`来首先添加一个标题头：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable<String> menu =
          Observable.just("Coffee", "Tea", "Espresso", "Latte");

        //print menu
        menu.startWith("COFFEE SHOP MENU")
          .subscribe(System.out::println);

      }
}
```

上述代码片段的输出如下：

```java
COFFEE SHOP MENU
Coffee
Tea
Espresso
Latte
```

如果你想要开始于多个发射，请使用`startWithArray()`来接受`varargs`参数。如果我们想在标题和菜单项之间添加分隔符，我们可以开始于标题和分隔符作为发射：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable<String> menu =
          Observable.just("Coffee", "Tea", "Espresso", "Latte");

        //print menu
        menu.startWithArray("COFFEE SHOP MENU","----------------")
          .subscribe(System.out::println);

     }
}
```

上述代码片段的输出如下：

```java
COFFEE SHOP MENU
----------------
Coffee
Tea
Espresso
Latte
```

`startWith()`操作符在这种情况下很有用，当我们想要提供一个初始值或在我们自己的发射之前添加一个或多个发射。

如果你想要一个完整的`Observable`发射先于另一个`Observable`的发射，你将想要使用`Observable.concat()`或`concatWith()`，我们将在下一章中介绍。

# `defaultIfEmpty()`

如果我们希望在给定的`Observable`为空时仅使用单个发射，我们可以使用`defaultIfEmpty()`。对于给定的`Observable<T>`，我们可以在调用`onComplete()`时没有发射发生的情况下指定一个默认的`T`发射。

如果我们有一个`Observable<String>`并过滤以查找以`Z`开头的项，但没有项符合此标准，我们可以求助于发射`None`：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable<String> items =
          Observable.just("Alpha","Beta","Gamma","Delta","Epsilon");

        items.filter(s -> s.startsWith("Z"))
          .defaultIfEmpty("None")
          .subscribe(System.out::println);
     }
}
```

上述代码片段的输出如下：

```java
None
```

当然，如果发生发射，我们永远不会看到`None`被发射。这只会在前面的`Observable`为空时发生。

# `switchIfEmpty()`

与`defaultIfEmpty()`类似，`switchIfEmpty()`在源`Observable`为空时指定一个不同的`Observable`来发射值。这允许我们在源为空的情况下指定不同的发射序列，而不是只发射一个值。

我们可以选择发射三个额外的字符串，例如，如果前面的`Observable`由于`filter()`操作为空：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
          .filter(s -> s.startsWith("Z"))
          .switchIfEmpty(Observable.just("Zeta", "Eta", "Theta"))
          .subscribe(i -> System.out.println("RECEIVED: " + i),
          e -> System.out.println("RECEIVED ERROR: " + e)
          );
     }
}

```

上述代码片段的输出如下：

```java
RECEIVED: Zeta
RECEIVED: Eta
RECEIVED: Theta
```

当然，如果前面的`Observable`不为空，那么`switchIfEmpty()`将没有效果，不会使用指定的`Observable`。

# `sorted()`

如果你有一个有限制的`Observable<T>`，它发射实现`Comparable<T>`的项，你可以使用`sorted()`来对发射进行排序。内部，它将收集所有发射，然后按排序顺序重新发射它们。在以下代码片段中，我们按自然顺序对`Observable<Integer>`的发射进行排序：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just(6, 2, 5, 7, 1, 4, 9, 8, 3)
          .sorted()
          .subscribe(System.out::println);
      }
}

```

上述代码片段的输出如下：

```java
1
2
3
4
5
6
7
8
9
```

当然，这可能会对性能产生一些影响，因为它将在再次发射之前在内存中收集所有排放。如果你使用它来对抗无限的`Observable`，可能会得到`OutOfMemory`错误。

你还可以提供`Comparator`作为参数来指定一个显式的排序标准。我们可以提供`Comparator`来反转排序顺序，如下所示：

```java
import io.reactivex.Observable;
import java.util.Comparator;

public class Launcher {
      public static void main(String[] args) {

        Observable.just(6, 2, 5, 7, 1, 4, 9, 8, 3)
          .sorted(Comparator.reverseOrder())
          .subscribe(System.out::println);
      }
}
```

前一个代码片段的输出如下：

```java
9
8
7
6
5
4
3
2
1
```

由于`Comparator`是一个单抽象方法接口，你可以通过 lambda 表达式快速实现它。指定代表两个排放的两个参数，然后将它们映射到它们的比较操作。我们可以用这个来按长度排序字符串排放，例如。这也允许我们排序不实现`Comparable`接口的项目：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma" ,"Delta", "Epsilon")
          .sorted((x,y) -> Integer.compare(x.length(), y.length()))
          .subscribe(System.out::println);
     }
}

```

前一个代码片段的输出如下：

```java
    Beta
    Alpha
    Gamma
    Delta
    Epsilon
```

# delay()

我们可以使用`delay()`运算符来推迟排放。它将保留任何接收到的排放，并将每个排放推迟指定的时长。如果我们想将排放推迟三秒，可以这样做：

```java
import io.reactivex.Observable;
import java.util.concurrent.TimeUnit;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma" ,"Delta", "Epsilon")
          .delay(3, TimeUnit.SECONDS)
          .subscribe(s -> System.out.println("Received: " + s));

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

前一个代码片段的输出如下：

```java
    Received: Alpha
    Received: Beta
    Received: Gamma
    Received: Delta
    Received: Epsilon
```

因为`delay()`在另一个调度器（如`Observable.interval()`）上操作，我们需要利用`sleep()`方法来保持应用程序足够长时间，以便看到这个效果。每个排放将被推迟三秒。你可以传递一个可选的第三个布尔参数，表示你是否想推迟错误通知。

对于更复杂的情况，你可以将另一个`Observable`作为你的`delay()`参数传递，这将推迟排放，直到那个其他`Observable`发射一些内容。

注意，还有一个`delaySubscription()`运算符，它将延迟订阅它前面的`Observable`，而不是延迟每个单独的排放。

# repeat()

`repeat()`运算符将在`onComplete()`之后重复上游订阅指定次数。

例如，我们可以通过将`2`作为`repeat()`的参数传递给给定的`Observable`来重复排放两次，如下面的代码片段所示：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma" ,"Delta", "Epsilon")
          .repeat(2)
          .subscribe(s -> System.out.println("Received: " + s));
      }
}
```

前一个代码片段的输出如下：

```java
    Received: Alpha
    Received: Beta
    Received: Gamma
    Received: Delta
    Received: Epsilon
    Received: Alpha
    Received: Beta
    Received: Gamma
    Received: Delta
    Received: Epsilon
```

如果你没有指定数字，它将无限重复，在每次`onComplete()`之后重新订阅。还有一个`repeatUntil()`运算符，它接受一个布尔供应商 lambda 参数，并将继续重复，直到它产生`true`。

# scan()

`scan()`方法是一个滚动聚合器。对于每个排放，你将其添加到一个累积中。然后，它将发出每个增量累积。

例如，你可以通过将一个 lambda 表达式传递给`scan()`方法来发射每个排放的滚动总和，该 lambda 表达式将每个`next`排放添加到`accumulator`中：

```java
    import io.reactivex.Observable;

    public class Launcher {
      public static void main(String[] args) {

        Observable.just(5, 3, 7, 10, 2, 14)
          .scan((accumulator, next) -> accumulator + next)
          .subscribe(s -> System.out.println("Received: " + s));

      }
    }

```

前一个代码片段的输出如下：

```java
Received: 5
Received: 8
Received: 15
Received: 25
Received: 27
Received: 41
```

它发出了初始值`5`，这是它接收到的第一个值。然后，它接收到了`3`，并将其加到`5`上，发出了`8`。之后，接收到了`7`，将其加到`8`上，发出了`15`，以此类推。这不仅仅可以用于滚动求和。你可以创建许多种类的累积（甚至是非数学的，如字符串连接或布尔缩减）。

注意，`scan()`与`reduce()`非常相似，我们将在稍后了解。小心不要混淆这两个。`scan()`方法为每个发射发出滚动累积，而`reduce()`在调用`onComplete()`后产生一个单一的发射，反映最终的累积。`scan()`可以在无限`Observable`上安全使用，因为它不需要调用`onComplete()`。

你也可以为第一个参数提供一个初始值，并将聚合到与发射不同的类型中。如果我们想发出发射的滚动计数，我们可以提供一个初始值`0`，并为每个发射加`1`。请注意，初始值将首先发出，所以如果你不想有初始发射，请在`scan()`之后使用`skip(1)`：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
          .scan(0, (total, next) -> total + 1)
          .subscribe(s -> System.out.println("Received: " + s));

      }
}

```

上一段代码的输出如下：

```java
    Received: 0
    Received: 1
    Received: 2
    Received: 3
    Received: 4
    Received: 5
```

# 缩减操作符

你可能会遇到想要将一系列发射合并成一个发射（通常通过`Single`发出）的时刻。我们将介绍一些完成此任务的运算符。请注意，几乎所有这些运算符都只适用于有限`Observable`，因为通常我们只能合并有限的数据集。我们将随着介绍这些运算符来探索这种行为。

# count()

将发射合并成一个的最简单操作符是`count()`。它将计算发射的数量，并在调用`onComplete()`后通过`Single`发出，如下所示：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
          .count()
          .subscribe(s -> System.out.println("Received: " + s));

      }
}
```

上一段代码的输出如下：

```java
    Received: 5
```

与大多数缩减操作符一样，这个操作符不应该用在无限`Observable`上。它将挂起并无限期地工作，永远不会发出计数或调用`onComplete()`。你应该考虑使用`scan()`来发出滚动计数。

# reduce()

`reduce()`操作符在语法上与`scan()`相同，但它只在源调用`onComplete()`时才发出最终的累积。根据你使用的重载，它可以产生`Single`或`Maybe`。如果你想发出所有整数发射的总和，你可以将每个值加到滚动总和中。但只有在最终确定后才会发出：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just(5, 3, 7, 10, 2, 14)
          .reduce((total, next) -> total + next)
          .subscribe(s -> System.out.println("Received: " + s));
      }
}

```

上一段代码的输出如下：

```java
    Received: 41
```

与`scan()`类似，你可以提供一个种子参数，它将作为累积的初始值。如果我们想将我们的发射转换为单个逗号分隔的值字符串，我们可以像下面这样使用`reduce()`：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just(5, 3, 7, 10, 2, 14)
          .reduce("", (total, next) -> total + (total.equals("") ? "" :
            ",") +  next)
              .subscribe(s -> System.out.println("Received: " + s));
      }
}

```

上一段代码的输出如下：

```java
    Received: 5,3,7,10,2,14

```

我们提供了一个空字符串作为我们的种子值，并保持一个滚动连接总和，并继续添加到它。我们使用三元运算符来防止前面的逗号，检查`total`是否是种子值，如果是，则返回一个空字符串而不是逗号。

您的种子值应该是不可变的，例如整数或字符串。如果它是可变的，可能会发生不良副作用，您应该使用`collect()`（或`seedWith()`）来处理这些情况，我们将在稍后介绍。例如，如果您想将`T`发射减少到一个集合中，例如`List<T>`，请使用`collect()`而不是`reduce()`。使用`reduce()`将产生一个不期望的副作用，即每次订阅都使用相同的列表，而不是每次都创建一个新的、空的列表。

# all()

`all()`运算符验证每个发射是否符合指定的条件，并返回一个`Single<Boolean>`。如果它们都通过，它将发出`True`。如果它遇到一个失败的，它将立即发出`False`。在以下代码片段中，我们发出对六个整数的测试，验证它们是否都小于`10`：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just(5, 3, 7, 11, 2, 14)
          .all(i -> i < 10)
          .subscribe(s -> System.out.println("Received: " + s));
     }
}
```

上述代码片段的输出如下：

```java
    Received: false
```

当遇到`all()`运算符的`11`时，它立即发出`False`并调用`onComplete()`。它甚至没有到达`2`或`14`，因为这将是多余的工作。它已经找到了一个不符合整个测试条件的元素。

如果您在一个空的`Observable`上调用`all()`，它将由于空值真原则而发出`true`。您可以在维基百科上了解更多关于空值真的信息：[`en.wikipedia.org/wiki/Vacuous_truth`](https://en.wikipedia.org/wiki/Vacuous_truth)。

# any()

`any()`方法将检查至少有一个发射满足特定标准，并返回一个`Single<Boolean>`。一旦它找到一个符合条件的发射，它将发出`true`然后调用`onComplete()`。如果它处理了所有发射并发现它们都是`false`，它将发出`false`并调用`onComplete()`。

在以下代码片段中，我们发出四个日期字符串，将它们转换为`LocalDate`发射，并测试其中是否有在六月或之后的月份：

```java
import io.reactivex.Observable;
import java.time.LocalDate;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("2016-01-01", "2016-05-02", "2016-09-12", 
"2016-04-03")
        .map(LocalDate::parse)
        .any(dt -> dt.getMonthValue() >= 6)
        .subscribe(s -> System.out.println("Received: " + s));
      }
}
```

上述代码片段的输出如下：

```java
    Received: true
```

当它遇到日期`2016-09-12`时，它立即发出`true`并调用`onComplete()`。它没有继续处理`2016-04-03`。

如果您在一个空的`Observable`上调用`any()`，它将由于空值真原则而发出`false`。您可以在维基百科上了解更多关于空值真的信息：[`en.wikipedia.org/wiki/Vacuous_truth`](https://en.wikipedia.org/wiki/Vacuous_truth)。

# contains()

`contains()`运算符将检查一个特定的元素（基于`hashCode()/equals()`实现）是否曾从`Observable`中发出。它将返回一个`Single<Boolean>`，如果找到则发出`true`，如果没有找到则发出`false`。

在以下代码片段中，我们发出整数`1`到`10000`，并使用`contains()`检查是否从中发出了数字`9563`：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.range(1,10000)
          .contains(9563)
          .subscribe(s -> System.out.println("Received: " + s));
      }
}

```

上述代码片段的输出如下：

```java
    Received: true
```

如你或许能猜到的，一旦找到元素，它将发出 true 并调用 `onComplete()` 以及处置操作。如果源调用 `onComplete()` 但元素未找到，它将发出 `false`。

# 集合操作符

集合操作符会将所有发射累积到一个集合，如列表或映射中，然后作为单个发射发出整个集合。集合操作符是减少操作符的另一种形式，因为它们将发射合并成一个。我们将单独介绍它们，因为它们是一个重要的类别。

注意，你应该避免仅仅为了将发射减少到集合中。这可能会削弱反应式编程的好处，在反应式编程中，项目是按顺序、逐个处理的。你只应该在逻辑上将发射以某种方式分组时才将发射合并到集合中。

# `toList()`

一个常见的集合操作符是 `toList()`。对于一个给定的 `Observable<T>`，它将收集传入的发射到 `List<T>` 中，然后将整个 `List<T>` 作为单个发射（通过 `Single<List<T>>`）推送出去。在下面的代码片段中，我们将字符串发射收集到 `List<String>` 中。在先前的 `Observable` 发出 `onComplete()` 之后，该列表被推送到 `observer` 以供打印：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
          .toList()
          .subscribe(s -> System.out.println("Received: " + s));
      }
}
```

上述代码片段的输出如下：

```java
    Received: [Alpha, Beta, Gamma, Delta, Epsilon]
```

默认情况下，`toList()` 将使用标准的 `ArrayList` 实现。你可以选择指定一个整数参数作为 `capacityHint`，这将优化 ArrayList 的初始化，以期望大约有那么多项：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.range(1,1000)
          .toList(1000)
          .subscribe(s -> System.out.println("Received: " + s));
      }
}

```

如果你想要指定除了 `ArrayList` 之外的不同列表实现，你可以提供一个 `Callable` lambda 作为参数来构建一个。在下面的代码片段中，我提供了一个 `CopyOnWriteArrayList` 实例作为我的列表：

```java
import io.reactivex.Observable;
import java.util.concurrent.CopyOnWriteArrayList;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
          .toList(CopyOnWriteArrayList::new)
          .subscribe(s -> System.out.println("Received: " + s));
      }
}
```

如果你想要使用 Google Guava 的不可变列表，这会稍微复杂一些，因为它是不可变的并且使用构建器。我们将在本节稍后通过 `collect()` 展示如何做到这一点。

# `toSortedList()`

`toList()` 的另一种风味是 `toSortedList()`。这将收集发射到根据其 `Comparator` 实现自然排序的列表中。然后，它将排序后的 `List<T>` 推送到 `Observer`：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just(6, 2, 5, 7, 1, 4, 9, 8, 3)
          .toSortedList()
          .subscribe(s -> System.out.println("Received: " + s));
     }
}
```

上述代码片段的输出如下：

```java
Received: [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

与 `sorted()` 一样，你可以提供一个 `Comparator` 作为参数来应用不同的排序逻辑。你还可以指定与 `toList()` 一样的基础 `ArrayList` 的初始容量。

# `toMap()` 和 `toMultiMap()`

对于一个给定的 `Observable<T>`，`toMap()` 操作符将收集发射到 `Map<K,T>` 中，其中 `K` 是从 lambda `Function<T,K>` 参数派生的键类型，为每个发射生成键。

如果我们想要将字符串收集到 `Map<Char,String>` 中，其中每个字符串都根据其第一个字符作为键，我们可以这样做：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
          .toMap(s -> s.charAt(0))
          .subscribe(s -> System.out.println("Received: " + s));
      }
}
```

上述代码片段的输出如下：

```java
    Received: {A=Alpha, B=Beta, D=Delta, E=Epsilon, G=Gamma}
```

`s -> s.charAt(0)` lambda 参数将每个字符串取出来，并从中导出键以与之配对。在这种情况下，我们正在将字符串的第一个字符作为键。

如果我们想要产生与键关联的不同值，而不是发射值，我们可以提供一个第二个 lambda 参数，该参数将每个发射映射到不同的值。例如，我们可以将每个首字母键映射到该字符串的长度：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
          .toMap(s -> s.charAt(0), String::length)
          .subscribe(s -> System.out.println("Received: " + s));
      }
}

```

上述代码片段的输出如下：

```java
    Received: {A=5, B=4, D=5, E=7, G=5}
```

默认情况下，`toMap()` 将使用 `HashMap`。您也可以提供一个第三个 lambda 参数，以提供不同的映射实现。例如，我可以提供 `ConcurrentHashMap` 而不是 `HashMap`：

```java
import io.reactivex.Observable;
import java.util.concurrent.ConcurrentHashMap;

public class Launcher {
      public static void main(String[] args) {
        Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
          .toMap(s -> s.charAt(0), String::length,   
ConcurrentHashMap::new)
          .subscribe(s -> System.out.println("Received: " + s));
      }
}

```

注意，如果我有多个发射映射到键，则该键的最后发射将替换后续的发射。如果我将字符串长度作为每个发射的键，则 `Alpha` 将被 `Gamma` 替换，而 `Gamma` 将被 `Delta` 替换：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
          .toMap(String::length)
          .subscribe(s -> System.out.println("Received: " + s));
      }
}
```

上述代码片段的输出如下：

```java
    Received: {4=Beta, 5=Delta, 7=Epsilon}
```

如果您希望给定的键映射到多个发射，则可以使用 `toMultiMap()`，它将为每个键维护一个相应的值列表。`Alpha`、`Gamma` 和 `Delta` 将被放入一个以长度五为键的列表中：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
          .toMultimap(String::length)
          .subscribe(s -> System.out.println("Received: " + s));
      }
}
```

上述代码片段的输出如下：

```java
    Received: {4=[Beta], 5=[Alpha, Gamma, Delta], 7=[Epsilon]}
```

# collect()

当没有收集操作符满足您的需求时，您始终可以使用 `collect()` 操作符来指定一个不同的类型以收集项目。例如，没有 `toSet()` 操作符来收集到 `Set<T>` 中的排放物，但您可以使用 `collect()` 快速有效地完成此操作。您需要指定两个由 lambda 表达式构建的参数：`initialValueSupplier`，它将为新的 `Observer` 提供一个新的 `HashSet`，以及 `collector`，它指定了如何将每个发射添加到该 `HashSet` 中：

```java
import io.reactivex.Observable;
import java.util.HashSet;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
          .collect(HashSet::new, HashSet::add)
          .subscribe(s -> System.out.println("Received: " + s));
      }
}
```

上述代码片段的输出如下：

```java
    Received: [Gamma, Delta, Alpha, Epsilon, Beta]
```

现在的 `collect()` 操作符将发出一个包含所有发出值的单个 `HashSet<String>`。

当您需要将排放物放入可变对象中，并且每次都需要一个新的可变对象种子时，请使用 `collect()` 而不是 `reduce()`。我们还可以使用 `collect()` 处理更复杂的情况，这些情况不是简单的集合实现。

假设您已将 Google Guava 作为依赖项添加 ([`github.com/google/guava`](https://github.com/google/guava))，并且您想要将排放物收集到一个 `ImmutableList` 中。要创建一个 `ImmutableList`，您必须调用其 `builder()` 工厂方法以产生一个 `ImmutableList.Builder<T>`**。** 然后，您调用其 `add()` 方法将项目放入构建器中，接着调用 `build()`，它返回一个密封的、最终的 `ImmutableList<T>`，该列表不能被修改。

要将发射收集到 `ImmutableList` 中，你可以为第一个 lambda 参数提供一个 `ImmutableList.Builder<T>`，然后在第二个参数中通过其 `add()` 方法添加每个元素。这将在其完全填充后发出 `ImmutableList.Builder<T>`，然后你可以将其 `map()` 到其 `build()` 调用，以发出 `ImmutableList<T>`：

```java
import com.google.common.collect.ImmutableList;
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
          .collect(ImmutableList::builder, ImmutableList.Builder::add)
          .map(ImmutableList.Builder::build)
          .subscribe(s -> System.out.println("Received: " + s));
      }
}
```

上述代码片段的输出如下：

```java
    Received: [Alpha, Beta, Gamma, Delta, Epsilon]
```

再次强调，`collect()` 操作符对于将发射收集到任何 RxJava 不直接提供的任意类型非常有用。

# 错误恢复操作符

在你的 `Observable` 链中，可能会因为你的操作而出现异常。我们已经知道 `onError()` 事件是沿着 `Observable` 链向下传递到 `Observer` 的。在那之后，订阅终止，不会再有更多的发射发生。但有时，我们想在异常到达 `Observer` 之前拦截它们，并尝试某种形式的恢复。我们不一定假装错误从未发生并期望发射继续，但我们可以尝试重新订阅或切换到另一个来源的 `Observable`。

我们仍然可以这样做，只是不是使用 RxJava 操作符，我们很快就会看到。如果你发现错误恢复操作符不符合你的需求，那么很可能会以创造性的方式组合它们。

对于这些示例，让我们将每个整数发射量除以 10，其中一个发射量是 `0`。这将导致向 `Observer` 发射一个 "`/ by zero`" 异常，如下面的代码片段所示：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just(5, 2, 4, 0, 3, 2, 8)
          .map(i -> 10 / i)
          .subscribe(i -> System.out.println("RECEIVED: " + i),
          e -> System.out.println("RECEIVED ERROR: " + e)
          );
      }
}

```

上述代码片段的输出如下：

```java
    RECEIVED: 2
    RECEIVED: 5
    RECEIVED: 2
    RECEIVED ERROR: java.lang.ArithmeticException: / by zero
```

# `onErrorReturn()` 和 `onErrorReturnItem()`

当你想在发生异常时回退到默认值时，你可以使用 `onErrorReturnItem()`。如果我们想在发生异常时发出 `-1`，我们可以这样做：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just(5, 2, 4, 0, 3, 2, 8)
          .map(i -> 10 / i)
          .onErrorReturnItem(-1)
          .subscribe(i -> System.out.println("RECEIVED: " + i),
          e -> System.out.println("RECEIVED ERROR: " + e)
          );
      }
}
```

上述代码片段的输出如下：

```java
    RECEIVED: 2
    RECEIVED: 5
    RECEIVED: 2
    RECEIVED: -1
```

你也可以提供一个 `Function<Throwable,T>` 来动态地使用 lambda 生成值。这让你可以访问 `Throwable`，你可以用它来确定返回的值，如下面的代码片段所示：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just(5, 2, 4, 0, 3, 2, 8)
          .map(i -> 10 / i)
          .onErrorReturn(e -> - 1)
          .subscribe(i -> System.out.println("RECEIVED: " + i),
          e -> System.out.println("RECEIVED ERROR: " + e)
          );
      }
}
```

`onErrorReturn()` 的放置很重要。如果我们把它放在 `map()` 操作符之前，错误就不会被捕获，因为错误发生在 `onErrorReturn()` 之后。为了拦截发出的错误，它必须在错误发生的地方下游。

注意，尽管我们发出了 `-1` 来处理错误，但序列在那之后仍然终止了。我们没有得到应该跟随的 `3`、`2` 或 `8`。如果你想恢复发射，你只需要在错误可能发生的 `map()` 操作符内处理错误。你会这样做，而不是使用 `onErrorReturn()` 或 `onErrorReturnItem()`：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just(5, 2, 4, 0, 3, 2, 8)
          .map(i -> {
            try {
                  return 10 / i;
                } catch (ArithmeticException e) {
                  return -1;
                }
                })
                .subscribe(i -> System.out.println("RECEIVED: " + i),
                e -> System.out.println("RECEIVED ERROR: " + e)
              );
       }
}
```

上述代码片段的输出如下：

```java
    RECEIVED: 2
    RECEIVED: 5
    RECEIVED: 2
    RECEIVED: -1
    RECEIVED: 3
    RECEIVED: 5
    RECEIVED: 1
```

# `onErrorResumeNext()`

与 `onErrorReturn()` 和 `onErrorReturnItem()` 类似，`onErrorResumeNext()` 非常相似。唯一的区别是它接受另一个 `Observable` 作为参数，在异常发生时发出可能多个值，而不是单个值。

这有点牵强，可能没有实际的应用场景，但我们可以错误发生时发出三个 `-1` 的发射项：

```java
    import io.reactivex.Observable;

    public class Launcher {
    public static void main(String[] args) {

      Observable.just(5, 2, 4, 0, 3, 2, 8)
        .map(i -> 10 / i)
        .onErrorResumeNext(Observable.just(-1).repeat(3))
        .subscribe(i -> System.out.println("RECEIVED: " + i),
        e -> System.out.println("RECEIVED ERROR: " + e)
        );
      }
    }

```

上一段代码片段的输出如下：

```java
    RECEIVED: 2
    RECEIVED: 5
    RECEIVED: 2
    RECEIVED: -1
    RECEIVED: -1
    RECEIVED: -1
```

我们也可以传递它 `Observable.empty()` 来在发生错误时安静地停止发射，并优雅地调用 `onComplete()` 函数：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just(5, 2, 4, 0, 3, 2, 8)
          .map(i -> 10 / i)
          .onErrorResumeNext(Observable.empty())
          .subscribe(i -> System.out.println("RECEIVED: " + i),
          e -> System.out.println("RECEIVED ERROR: " + e)
          );
      }
}
```

上一段代码片段的输出如下：

```java
RECEIVED: 2
RECEIVED: 5
RECEIVED: 2
```

与 `onErrorReturn()` 类似，你可以提供一个 `Function<Throwable,Observable<T>>` lambda 来从发出的 `Throwable` 动态生成一个 `Observable`，如下面的代码片段所示：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just(5, 2, 4, 0, 3, 2, 8)
          .map(i -> 10 / i)
          .onErrorResumeNext((Throwable e) ->   
Observable.just(-1).repeat(3))
              .subscribe(i -> System.out.println("RECEIVED: " + i),
            e -> System.out.println("RECEIVED ERROR: " + e)
            );
       }
}
```

上一段代码的输出如下：

```java
RECEIVED: 2
RECEIVED: 5
RECEIVED: 2
RECEIVED: -1
RECEIVED: -1
RECEIVED: -1
```

# retry()

尝试恢复的另一种方法是使用 `retry()` 算子，它有几个参数重载。它将重新订阅前面的 `Observable`，并希望不再出现错误。

如果你不带参数调用 `retry()`，它将为每个错误无限次地重新订阅。你需要小心使用 `retry()`，因为它可能会产生混乱的效果。使用我们的示例将导致它无限次地重复发射这些整数：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just(5, 2, 4, 0, 3, 2, 8)
        .map(i -> 10 / i)
        .retry()
        .subscribe(i -> System.out.println("RECEIVED: " + i),
        e -> System.out.println("RECEIVED ERROR: " + e)
        );
      }
}
```

上一段代码片段的输出如下：

```java
    RECEIVED: 5
    RECEIVED: 2
    RECEIVED: 2
    RECEIVED: 5
    RECEIVED: 2
    RECEIVED: 2
    RECEIVED: 5
    RECEIVED: 2
    ...
```

在放弃并仅向 `Observer` 发出错误之前指定一个固定的重试次数可能更安全。在下面的代码片段中，我们只会重试两次：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just(5, 2, 4, 0, 3, 2, 8)
          .map(i -> 10 / i)
          .retry(2)
          .subscribe(i -> System.out.println("RECEIVED: " + i),
          e -> System.out.println("RECEIVED ERROR: " + e)
          );
      }
}
```

上一段代码片段的输出如下：

```java
    RECEIVED: 2
    RECEIVED: 5
    RECEIVED: 2
    RECEIVED: 2
    RECEIVED: 5
    RECEIVED: 2
    RECEIVED: 2
    RECEIVED: 5
    RECEIVED: 2
    RECEIVED ERROR: java.lang.ArithmeticException: / by zero
```

你还可以提供 `Predicate<Throwable>` 或 `BiPredicate<Integer,Throwable>` 来有条件地控制何时尝试 `retry()`。`retryUntil()` 算子将允许在给定的 `BooleanSupplier` lambda 为 `false` 时进行重试。还有一个高级的 `retryWhen()` 算子，它支持高级的任务组合，例如延迟重试。

# 行动算子

为了结束这一章，我们将介绍一些有助于调试以及了解 `Observable` 链的辅助算子。这些是动作或 `doOn` 算子。

# doOnNext(), doOnComplete(), 和 doOnError()

这三个算子：`doOnNext()`、`doOnComplete()` 和 `doOnError()` 就像在 `Observable` 链的中间放置了一个迷你 `Observer`。

`doOnNext()` 算子允许你查看从算子中发出并进入下一个算子的每个发射项。此算子不会以任何方式影响操作或转换发射项。我们只为链中该点的每个事件创建副作用。例如，我们可以在将字符串映射到其长度之前对每个字符串执行操作。在这种情况下，我们只需通过提供一个 `Consumer<T>` lambda 来打印它们：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
          .doOnNext(s -> System.out.println("Processing: " + s))
          .map(String::length)
          .subscribe(i -> System.out.println("Received: " + i));
      }
}
```

上一段代码的输出如下：

```java
    Processing: Alpha
    Received: 5
    Processing: Beta
    Received: 4
    Processing: Gamma
    Received: 5
    Processing: Delta
    Received: 5
    Processing: Epsilon
    Received: 7
```

你还可以利用 `doAfterNext()`，它在发射传递到下游之后执行操作，而不是在之前。

`onComplete()` 操作符允许你在 `Observable` 链中的该点调用 `onComplete()` 时触发一个操作。这有助于查看 `Observable` 链的哪些点已经完成，如下面的代码片段所示：**

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
          .doOnComplete(() -> System.out.println("Source is done   
            emitting!"))
          .map(String::length)
          .subscribe(i -> System.out.println("Received: " + i));
     }
}

```

以下代码片段的输出如下：

```java
    Received: 5
    Received: 4
    Received: 5
    Received: 5
    Received: 7
    Source is done emitting!
```

当然，`onError()` 会查看正在向上传递的错误，并且你可以用它执行一个操作。这有助于在操作符之间放置，以查看哪个操作符导致了错误：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just(5, 2, 4, 0, 3, 2, 8)
          .doOnError(e -> System.out.println("Source failed!"))
          .map(i -> 10 / i)
          .doOnError(e -> System.out.println("Division failed!"))
          .subscribe(i -> System.out.println("RECEIVED: " + i),
          e -> System.out.println("RECEIVED ERROR: " + e)
          );
     }
}
```

以下代码片段的输出如下：

```java
    RECEIVED: 2
    RECEIVED: 5
    RECEIVED: 2
    Division failed!
    RECEIVED ERROR: java.lang.ArithmeticException: / by zero
```

我们在两个地方使用了 `doOnError()` 来查看错误首次出现的位置。由于我们没有看到打印出 `Source failed!`，而是看到了 `Division failed!`，我们可以推断错误发生在 `map()` 操作符中。

使用这三个操作符一起，可以深入了解你的 `Observable` 操作正在做什么，或者快速创建副作用。

你可以使用 `doOnEach()` 来指定 `onNext()`、`onComplete()` 和 `onError()` 的所有三个操作。`subscribe()` 方法接受这三个操作作为 lambda 参数或整个 `Observer<T>`。这就像在 `Observable` 链的中间放置了 `subscribe()`！还有一个 `doOnTerminate()` 操作符，它在 `onComplete()` 或 `onError()` 事件发生或被下游销毁时触发。

# doOnSubscribe() 和 doOnDispose()

另外两个有用的操作符是 `doOnSubscribe()` 和 `doOnDispose()`。`doOnSubscribe()` 在 `Observable` 链中的订阅发生时立即触发一个特定的 `Consumer<Disposable>`。它提供了对 Disposable 的访问，以便你在该操作中调用 `dispose()`。`doOnDispose()` 操作符将在 `Observable` 链中的该点执行销毁时执行特定操作。

我们使用这两个操作符来打印订阅和销毁发生的时间，如下面的代码片段所示。正如你所预测的，我们首先看到订阅事件被触发。然后，发射通过，最后销毁事件被触发：

```java
import io.reactivex.Observable;

public class Launcher {
      public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
          .doOnSubscribe(d -> System.out.println("Subscribing!"))
          .doOnDispose(() -> System.out.println("Disposing!"))
          .subscribe(i -> System.out.println("RECEIVED: " + i));
      }
}
```

以下代码片段的输出如下：

```java
    Subscribing!
    RECEIVED: Alpha
    RECEIVED: Beta
    RECEIVED: Gamma
    RECEIVED: Delta
    RECEIVED: Epsilon
    Disposing!
```

注意，`doOnDispose()` 可能会因为冗余的销毁请求而多次触发，或者如果没有以某种形式销毁，则根本不会触发。另一个选择是使用 `doFinally()` 操作符，它将在 `onComplete()` 或 `onError()` 被调用或由下游销毁后触发。

# doOnSuccess()

请记住，`Maybe` 和 `Single` 类型没有 `onNext()` 事件，而是有一个 `onSuccess()` 操作符来传递单个发射。因此，这两种类型都没有 `doOnNext()` 操作符，如下面的代码片段所示，而是有一个 `doOnSuccess()` 操作符。它的使用应该感觉就像 `doOnNext()`：

```java
import io.reactivex.Observable;

public class Launcher {

      public static void main(String[] args) {
        Observable.just(5, 3, 7, 10, 2, 14)
          .reduce((total, next) -> total + next)
          .doOnSuccess(i -> System.out.println("Emitting: " + i))
          .subscribe(i -> System.out.println("Received: " + i));
      }
}
```

以下代码片段的输出如下：

```java
    Emitting: 41
    Received: 41
```

# 摘要

在本章中，我们涵盖了大量的内容，并且希望到现在为止，你已经开始看到 RxJava 有很多实际的应用。我们介绍了各种抑制和转换发射以及将它们以某种形式减少到单个发射的操作符。你学习了 RxJava 如何通过操作符提供强大的错误恢复方式以及了解`Observable`链如何操作。

如果你想要了解更多关于 RxJava 操作符的信息，网上有很多资源。Marble 图是 Rx 文档的一种流行形式，可以直观地展示每个操作符的工作方式。*rxmarbles.com*（[`rxmarbles.com`](http://rxmarbles.com)）是一个流行的、交互式的网络应用，允许你拖动 Marble 发射并查看每个操作符影响的行为。还有一个*RxMarbles*安卓应用（[`play.google.com/store/apps/details?id=com.moonfleet.rxmarbles`](https://play.google.com/store/apps/details?id=com.moonfleet.rxmarbles)），你可以在你的安卓设备上使用。当然，你还可以在 ReactiveX 网站上看到操作符的完整列表（[`reactivex.io/documentation/operators.html`](http://reactivex.io/documentation/operators.html)）。

信不信由你，我们才刚刚开始。本章只涵盖了基本操作符。在接下来的章节中，我们将介绍执行强大行为的操作符，例如并发和多播。但在我们这样做之前，让我们继续介绍那些组合 Observables 的操作符。
