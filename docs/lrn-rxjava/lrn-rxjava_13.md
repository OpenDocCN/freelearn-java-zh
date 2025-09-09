# 附录

本附录将带你了解 lambda 表达式、函数式类型、面向对象和响应式编程的混合以及调度器的工作原理。

# 引入 lambda 表达式

Java 在 2014 年发布的 Java 8 中正式支持 lambda 表达式。*Lambda 表达式* 是 **单抽象方法** (**SAM**) 类的简写实现。换句话说，它们是传递函数式参数而不是匿名类的快捷方式。

# 将 Runnable 转换为 lambda

在 Java 8 之前，你可能已经利用匿名类即时实现接口，例如 `Runnable`，如下所示：

```java
public class Launcher {

    public static void main(String[] args) {

        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("run() was called!");
            }
        };

        runnable.run();
    }
}
```

输出如下：

```java
run() was called!
```

要实现 `Runnable` 而不声明一个显式的类，你必须立即在构造函数之后在一个块中实现其 `run()` 抽象方法。这产生了大量的样板代码，并成为 Java 开发的一个主要痛点，也是使用 Java 进行函数式编程的障碍。幸运的是，Java 8 正式将 lambda 引入到 Java 语言中。有了 lambda 表达式，你可以用更简洁的方式表达这一点：

```java
public class Launcher {

    public static void main(String[] args) {

        Runnable runnable = () -> System.out.println("run() was
        called!");

        runnable.run();
    }
}
```

太棒了，对吧？这减少了大量的代码和样板，我们将深入探讨它是如何工作的。Lambda 表达式可以针对任何只有一个抽象方法的接口或抽象类，这些类型被称为 *单抽象方法* 类型。在上面的代码中，`Runnable` 接口有一个名为 `run()` 的单抽象方法。如果你传递一个与该抽象方法的参数和返回类型匹配的 lambda，编译器将使用该 lambda 来实现该方法。

`->` 箭头左侧是参数。`Runnable` 的 `run()` 方法不接收任何参数，因此 lambda 使用空括号 `()` 提供没有参数。箭头 `->` 的右侧是要执行的操作。在这个例子中，我们调用一个单条语句并使用 `System.out.println("run() was called!");` 打印一个简单的消息。

Java 8 的 lambda 表达式可以在主体中支持多个语句。比如说，我们有一个包含多个语句的 `Runnable` 匿名内部类，其 `run()` 方法的实现如以下代码片段所示：

```java
public class Launcher {

    public static void main(String[] args) {

        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("Message 1");
                System.out.println("Message 2");
            }
        };

        runnable.run();
    }
}
```

你可以将两个 `System.out.println()` 语句移动到 lambda 中，通过在箭头 `->` 右侧的 `{ }` 多行块中包裹它们。请注意，你需要在 lambda 中的每一行使用分号来终止，如下所示：

```java
public class Launcher {

    public static void main(String[] args) {

        Runnable runnable = () -> {
            System.out.println("Message 1");
            System.out.println("Message 2");
        };

        runnable.run();
    }
}
```

# 将 Supplier 转换为 lambda

Lambda 表达式还可以实现返回项的方法。例如，Java 8 中引入的 `Supplier` 类（最初由 Google Guava 引入）有一个抽象的 `get()` 方法，它为给定的 `Supplier<T>` 返回一个 `T` 项。如果我们有一个 `Supplier<List<String>>`，其 `get()` 返回 `List<String>`，我们可以使用传统的匿名类来实现它：

```java
import java.util.ArrayList;
import java.util.List;
import java.util.function.Supplier;

public class Launcher {

    public static void main(String[] args) {

        Supplier<List<String>> listGenerator = new
Supplier<List<String>>() {
            @Override
            public List<String> get() {
                return new ArrayList<>();
            }
        };

        List<String> myList = listGenerator.get();
    }
}
```

但我们也可以使用 lambda，它可以更简洁地实现 `get()` 并返回 `List<String>`，如下所示：

```java
import java.util.ArrayList;
import java.util.List;
import java.util.function.Supplier;

public class Launcher {

    public static void main(String[] args) {

        Supplier<List<String>> listGenerator = () -> new ArrayList<>
        ();

        List<String> myList = listGenerator.get();
    }
}
```

当你的 lambda 简单地使用 `new` 关键字在类型上调用构造函数时，你可以使用双冒号 `::` lambda 语法来调用该类上的构造函数。这样，你可以省略 `()` 和 `->` 符号，如下所示：

```java
import java.util.ArrayList;
import java.util.List;
import java.util.function.Supplier;

public class Launcher {

    public static void main(String[] args) {

        Supplier<List<String>> listGenerator = ArrayList::new;

        List<String> myList = listGenerator.get();
    }
}
```

RxJava 没有 Java 8 的 Supplier，而是有一个 Callable，它实现了相同的目的。

# 将 Consumer 实现为 lambda

`Consumer<T>` 接受一个 `T` 参数并使用它执行一个操作，但不返回任何值。使用匿名类，我们可以创建一个 `Consumer<String>`，它简单地打印字符串，如下面的代码片段所示：

```java
import java.util.function.Consumer;

public class Launcher {

    public static void main(String[] args) {

        Consumer<String> printConsumer = new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        };

        printConsumer.accept("Hello World");
    }
}
```

输出如下：

```java
Hello World
```

你可以将其实现为一个 lambda。我们可以在 lambda 箭头 `->` 的左侧选择调用 `String` 参数 `s`，然后在右侧打印它：

```java
import java.util.function.Consumer;

public class Launcher {

    public static void main(String[] args) {

        Consumer<String> printConsumer = (String s) ->
        System.out.println(s);

        printConsumer.accept("Hello World");
    }
}
```

编译器实际上可以根据你正在针对的 `Consumer<String>` 推断出 `s` 是一个 `String` 类型。因此，你可以省略那个显式的类型声明，如下面的代码所示：

```java
import java.util.function.Consumer;

public class Launcher {

    public static void main(String[] args) {

        Consumer<String> printConsumer = s -> System.out.println(s);

        printConsumer.accept("Hello World");
    }
}
```

对于简单的单方法调用，实际上你可以使用另一种语法来使用双冒号 `::` 声明 lambda。在双冒号的左侧声明你正在针对的类型，然后在双冒号的右侧调用其方法。编译器足够智能，可以推断出你试图将 `String` 参数传递给 `System.out::println`：

```java
import java.util.function.Consumer;

public class Launcher {

    public static void main(String[] args) {

        Consumer<String> printConsumer = System.out::println;

        printConsumer.accept("Hello World");
    }
}

```

# 将 Function 实现为 lambda

Lambda 也可以实现接受参数并返回项的单个抽象方法。例如，RxJava 2.0（以及 Java 8）有一个 `Function<T,R>` 类型，它接受 `T` 类型并返回 `R` 类型。例如，你可以声明一个 `Function<String,Integer>`，其 `apply()` 方法将接受一个 `String` 并返回一个 `Integer`。在这里，我们通过在匿名类中返回字符串的长度来实现 `apply()`，如下所示：

```java
import java.util.function.Function;

public class Launcher {

    public static void main(String[] args) {

        Function<String,Integer> lengthMapper = new Function<String,
Integer>() {
            @Override
            public Integer apply(String s) {
                return s.length();
            }
        };

        Integer length = lengthMapper.apply("Alpha");

        System.out.println(length);
    }
}
```

你可以通过使用 lambda 来使这个实现更加简洁，如下所示：

```java
import java.util.function.Function;

public class Launcher {

    public static void main(String[] args) {

        Function<String,Integer> lengthMapper = (String s) ->
s.length();

        Integer length = lengthMapper.apply("Alpha");

        System.out.println(length);
    }
}
```

我们有几个可选的语法可以用来实现 `Function<String,Integer>`。

Java 8 的编译器足够智能，可以根据我们分配给它的 `Function<String,Integer>` 类型推断出我们的参数 `s` 是一个 `String` 类型。因此，我们不需要显式声明 `s` 为 `String` 类型，因为它可以推断出来：

```java
   Function<String,Integer> lengthMapper = (s) -> s.length();
```

我们也不需要将 `s` 括在括号 `(s)` 中，因为对于单个参数来说，这些括号不是必需的（但如我们稍后看到的，对于多个参数来说，这些括号是必需的）：

```java
   Function<String,Integer> lengthMapper = s -> s.length();
```

如果我们只是在传入的项目上调用方法或属性，我们可以使用双冒号 `::` 语法来调用该类型上的方法：

```java
   Function<String,Integer> lengthMapper = String::length;

```

`Function<T,R>` 在 RxJava 中被广泛用作 `Observable` 操作符，通常用于转换发射的数据。最常用的例子是 `map()` 操作符，它将每个 `T` 发射转换为 `R` 发射，并从 `Observable<T>` 导出 `Observable<R>`：

```java
import io.reactivex.Observable;

public class Launcher {
    public static void main(String[] args) {
        Observable.just("Alpha","Beta","Gamma")
                .map(String::length) //accepts a Function<T,R>
                .subscribe(s -> System.out.println(s));
    }
}
```

注意，还有其他 `Function` 的变体，例如 `Predicate` 和 `BiFunction`，它们接受两个参数，而不是一个。`reduce()` 操作符接受一个 `BiFunction<T,T,T>`，其中第一个 `T` 参数是滚动聚合，第二个 `T` 是要放入聚合中的下一个项，第三个 `T` 是合并两个的结果。在这种情况下，我们使用 `reduce()` 来通过滚动总数添加所有项：

```java
import io.reactivex.Observable;

public class Launcher {
    public static void main(String[] args) {
        Observable.just("Alpha","Beta","Gamma")
                .map(String::length)
                .reduce((total,next) -> total + next) //accepts a
                 BiFunction<T,T,T>
                .subscribe(s -> System.out.println(s));
    }
}
```

# 功能类型

在撰写本文时，以下是 RxJava 2.0 中所有可用的功能类型，您可以在 `io.reactivex.functions` 包中找到它们。您可能会认出其中许多功能类型几乎与 Java 8（在 `java.util.function` 中）或 Google Guava 中的类型相同。然而，它们在 RxJava 2.0 中被部分复制，以便在 Java 6 和 7 中使用。一个细微的区别是 RxJava 的实现会抛出检查异常。这消除了 RxJava 1.0 中的一个痛点，即检查异常必须在返回它们的 lambda 表达式中处理。

以下列出了 RxJava 1.0 的等效项，但请注意，单抽象方法（SAM）列对应于 RxJava 2.0 类型。RxJava 1.0 函数实现 `call()` 并不支持原始类型。RxJava 2.0 实现了一些带有原始类型的功能类型，以尽可能减少装箱开销：

| **RxJava 2.0** | **RxJava 1.0** | **SAM** | **描述** |
| --- | --- | --- | --- |
| `Action` | `Action0` | `run()` | 执行操作，类似于 `Runnable` |
| `Callable<T>` | `Func0<T>` | `get()` | 返回单个类型为 `T` 的项 |
| `Consumer<T>` | `Action1<T>` | `accept()` | 在给定的 `T` 项上执行操作，但不返回任何内容 |
| `Function<T,R>` | `Func1<T,R>` | `apply()` | 接受类型 `T` 并返回类型 `R` |
| `Predicate<T>` | `Func1<T,Boolean>` | `test()` | 接受 `T` 项并返回原始 `boolean` |
| `BiConsumer<T1,T2>` | `Action2<T1,T2>` | `accept()` | 在 `T1` 和 `T2` 项上执行操作，但不返回任何内容 |
| `BiFunction<T1,T2,R>` | `Func2<T1,T2,R>` | `apply()` | 接受 `T1` 和 `T2` 并返回类型 `R` |
| `BiPredicate<T1,T2>` | `Func2<T1,T2,Boolean>` | `test()` | 接受 `T1` 和 `T2` 并返回原始 `boolean` |
| `Function3<T1,T2,T3,R>` | `Func3<T1,T2,T3,R>` | `apply()` | 接受三个参数并返回 `R` |
| `BooleanSupplier` | `Func0<Boolean>` | `getAsBoolean()` | 返回单个原始 `boolean` 值 |
| `LongConsumer` | `Action1<Long>` | `accept()` | 在给定的 `Long` 上执行操作，但不返回任何内容 |
| `IntFunction` | `Func1<T>` | `apply()` | 接受原始 `int` 并返回类型为 `T` 的项 |

在 RxJava 2.0 中，并非所有功能类型的原始等效项都已实现。例如，目前还没有像 Java 8 标准库中那样的 `IntSupplier`。这是因为 RxJava 2.0 不需要它来实现其任何操作符。

# 混合面向对象和响应式编程

当你开始将 RxJava 知识应用于现实世界的问题时，可能不会立即清楚的是如何将其与面向对象编程相结合。利用面向对象和函数式编程等多个范式正变得越来越普遍。在 Java 环境中，响应式编程和面向对象编程肯定可以为了更大的利益而协同工作。

显然，你可以从 `Observable` 或其他响应式类型中发射任何类型 `T`。发射基于你自己的类构建的对象是面向对象和响应式编程协同工作的一种方式。我们在本书中看到了许多例子。例如，Java 8 的 `LocalDate` 是一个复杂的面向对象类型，但你可以将它通过一个 `Observable<LocalDate>` 推送，如下面的代码所示：

```java
import io.reactivex.Observable;
import java.time.LocalDate;

public class Launcher {

    public static void main(String[] args) {

       Observable<LocalDate> dates = Observable.just(
               LocalDate.of(2017,11,3),
               LocalDate.of(2017,10,4),
               LocalDate.of(2017,7,5),
               LocalDate.of(2017,10,3)
       );

       // get distinct months
       dates.map(LocalDate::getMonth)
       .distinct()
       .subscribe(System.out::println);
    }
}
```

输出如下：

```java
NOVEMBER
OCTOBER
JULY
```

正如我们在本书中的几个例子中所看到的，许多 RxJava 操作符提供了适配器，可以将有状态的、面向对象的项目转换为响应式流。例如，有 `generate()` 工厂用于 `Flowable` 和 `Observable`，可以从一个可变对象构建一系列发射，该对象是逐步更新的。在下面的代码中，我们发射了一个无限连续的 Java 8 `LocalDate` 序列，但只取前 60 个发射。由于 `LocalDate` 是不可变的，我们将 `2017-1-1` 的种子 `LocalDate` 包裹在一个 `AtomicReference` 中，以便它可以被可变地替换为每次增量：

```java
import io.reactivex.Emitter;
import io.reactivex.Flowable;
import java.time.LocalDate;
import java.util.concurrent.atomic.AtomicReference;

public class Launcher {

    public static void main(String[] args) {

       Flowable<LocalDate> dates =
               Flowable.generate(() -> new AtomicReference<>
(LocalDate.of(2017,1,1)),
               (AtomicReference<LocalDate> next, Emitter<LocalDate>
emitter) ->
               emitter.onNext(next.getAndUpdate(dt -> 
dt.plusDays(1)))
                       );

       dates.take(60)
               .subscribe(System.out::println);
    }
}
```

输出如下：

```java
2017-01-01
2017-01-02
2017-01-03
2017-01-04
2017-01-05
2017-01-06
...
```

因此，RxJava 有许多工厂和工具来适应你的面向对象、命令式操作，并将它们变为响应式。其中许多在本书中都有涉及。

但是，是否存在一个类从属性或方法返回 `Observable`、`Flowable`、`Single` 或 `Maybe` 的情况？当然有！当你的对象具有结果动态且随时间变化，并代表事件或大量数据序列的属性或方法时，它们是作为响应式类型返回的候选者。

这里有一个抽象的例子：比如说，你有一个表示飞行无人机的 `DroneBot` 类型。你可以有一个名为 `getLocation()` 的属性，它返回一个 `Observable<Point>` 而不是 `Point`。这样，你就可以获得一个实时流，每次无人机位置改变时都会推送一个新的 `Point` 发射：

```java
import io.reactivex.Observable;

public class Launcher {

    public static void main(String[] args) {
        DroneBot droneBot = null; // create DroneBot

        droneBot.getLocation()
          .subscribe(loc ->
          System.out.println("Drone moved to " + loc.x + "," +
loc.y));
    }

    interface DroneBot {
        int getId();
        String getModel();
        Observable<Location> getLocation();
    }

    static final class Location {
        private final double x;
        private final double y;

        Location(double x, double y) {
            this.x = x;
            this.y = y;
        }
    }
}
```

这个`DroneBot`示例展示了另一种有效混合面向对象和反应式编程的方式。你可以通过返回一个`Observable`来轻松获取该无人机的实时运动数据。这种模式有众多用例：股票行情、车辆位置、气象站数据、社交网络等等。然而，如果属性是无限的，就要小心了。如果你想要管理 100 个无人机的位置数据流，将它们所有的无限位置数据流扁平映射到一个单独的流中可能不会产生任何有意义的输出，除了一个没有上下文的位置噪声序列。你可能会分别订阅每一个，在一个 UI 中填充一个显示所有无人机的`Location`字段的表格，或者使用`Observable.combineLatest()`来发出所有无人机的最新位置快照。后者在实时显示地理地图上的点时可能很有帮助。

当类属性有限时，拥有反应式类属性也是有用的。比如说，你有一份仓库列表，并且想要计算所有仓库的总库存。每个`Warehouse`对象包含一个`Observable<ProductStock>`，它返回当前可用的产品库存的有限序列。`ProductStock`的`getQuantity()`方法返回该产品的可用数量。我们可以对`getQuantity()`值使用`reduce()`操作来获取所有可用库存的总和，如下所示：

```java
import io.reactivex.Observable;
import java.util.List;

public class Launcher {

    public static void main(String[] args) {
        List<Warehouse> warehouses = null; // get warehouses

        Observable.fromIterable(warehouses)
            .flatMap(Warehouse::getProducts)
            .map(ProductStock::getQuantity)
            .reduce(0,(total,next) -> total + next)
            .subscribe(i -> System.out.println("There are " + i + "
            units in inventory"));
    }

    interface Warehouse {
        Observable<ProductStock> getProducts();
    }
    interface ProductStock {
        int getId();
        String getDescription();
        int getQuantity();
    }
}
```

因此，像从`Warehouse`的`getProducts()`返回的有限`Observable`也有帮助，并且对于分析任务尤其有帮助。但请注意，这个特定的业务案例决定`getProducts()`将返回当时可用的产品，而不是每次库存变化时都广播库存的无限数据流。这是一个设计决策，有时，以冷数据的方式表示快照数据比热无限数据流更好。无限数据流将需要返回`Observable<List<ProductStock>>`（或`Observable<Observable<ProductStock>>`）以发出逻辑快照。你总是可以添加一个单独的`Observable`来发出更改通知，然后使用`flatMap()`在你的`getProducts()`上创建库存更改的热流。这样，你就在代码模型中创建了基本构建块，然后通过反应式地组合它们来完成更复杂的任务。

注意，你可以有返回反应式类型的方法并接受参数。这是一种创建针对特定任务定制的`Observable`或`Flowable`的强大方式。例如，我们可以在`warehouse`中添加一个`getProductsOnDate()`方法，该方法返回从给定日期发出的产品库存的`Observable`，如下面的代码所示：

```java
interface Warehouse {
    Observable<ProductStock> getProducts();
    Observable<ProductStock> getProductsOnDate(LocalDate date);
}
```

总结来说，混合响应式和面向对象编程不仅有益，而且是必要的。在设计领域类时，仔细考虑哪些属性和方法应该做成响应式的，以及它们应该是冷、热和/或无限。想象一下你将如何使用你的类，以及你的候选设计是否容易或难以操作。确保不要为了响应式而将每个属性和方法都做成响应式的。只有当有可用性或性能优势时才将其做成响应式的。例如，你不应该将领域类型的 `getId()` 属性做成响应式的。这个类实例上的 ID 很可能不会改变，它只是一个单一值，不是一个值序列。

# 物质化和去物质化

我们没有涵盖的两个有趣的操作符是 `materialize()` 和 `dematerialize()`。我们没有在 第三章，“基本操作符”，中涵盖它们，因为那时在学习曲线上可能会造成混淆。但希望你在阅读这段内容时，已经足够了解 `onNext()`、`onComplete()` 和 `onError()` 事件，足以使用一个以不同方式抽象封装它们的操作符。

`materialize()` 操作符将这三个事件，`onNext()`、`onComplete()` 和 `onError()`，将它们全部转换为包装在 `Notification<T>` 中的发射项。所以如果您的源发出五个发射项，您将得到六个发射项，其中最后一个将是 `onComplete()` 或 `onError()`。在下面的代码中，我们使用五个字符串的 `Observable` 进行 materialize，这些字符串被转换成六个 `Notification` 发射项：

```java
import io.reactivex.Observable;

public class Launcher {

    public static void main(String[] args) {

        Observable<String> source =
                Observable.just("Alpha", "Beta", "Gamma", "Delta",
"Epsilon");

        source.materialize()
                .subscribe(System.out::println);

    }
}
```

输出结果如下：

```java
OnNextNotification[Alpha]
OnNextNotification[Beta]
OnNextNotification[Gamma]
OnNextNotification[Delta]
OnNextNotification[Epsilon]
OnCompleteNotification
```

每个 `Notification` 有三个方法，`isOnNext()`、`isOnComplete()` 和 `isOnError()`，用于确定 `Notification` 是哪种类型的事件。还有一个 `getValue()` 方法，它将为 `onNext()` 返回发射值，但对于 `onComplete()` 或 `onError()` 将返回 null。我们利用 `Notification` 上的这些方法，如下面的代码所示，来过滤出三个事件到三个不同的观察者：

```java
import io.reactivex.Notification;
import io.reactivex.Observable;

public class Launcher {

    public static void main(String[] args) {

        Observable<Notification<String>> source =
                Observable.just("Alpha", "Beta", "Gamma", "Delta",
"Epsilon")
                        .materialize()
                        .publish()
                        .autoConnect(3);

        source.filter(Notification::isOnNext)
                .subscribe(n -> System.out.println("onNext=" +
n.getValue()));

        source.filter(Notification::isOnComplete)
                .subscribe(n -> System.out.println("onComplete"));

        source.filter(Notification::isOnError)
                .subscribe(n -> System.out.println("onError"));
    }
}
```

输出结果如下：

```java
onNext=Alpha
onNext=Beta
onNext=Gamma
onNext=Delta
onNext=Epsilon
onComplete
```

您也可以使用 `dematerialize()` 将发出通知的 `Observable` 或 `Flowable` 转换回正常的 `Observable` 或 `Flowable`。如果任何发射项不是 `Notification`，它将产生错误。不幸的是，在编译时，Java 无法强制执行应用于发出特定类型（如 Kotlin）的 Observables/Flowables 的操作符：

```java
import io.reactivex.Observable;

public class Launcher {

    public static void main(String[] args) {

        Observable.just("Alpha", "Beta", "Gamma", "Delta",
"Epsilon")
                .materialize()
                .doOnNext(System.out::println)
                .dematerialize()
                .subscribe(System.out::println);
    }
}
```

输出结果如下：

```java
OnNextNotification[Alpha]
Alpha
OnNextNotification[Beta]
Beta
OnNextNotification[Gamma]
Gamma
OnNextNotification[Delta]
Delta
OnNextNotification[Epsilon]
Epsilon
OnCompleteNotification
```

那么，你到底会用 `materialize()` 和 `dematerialize()` 做什么呢？你可能不会经常使用它们，这也是为什么它们被放在附录中的另一个原因。但它们在组合更复杂的操作符和扩展转换器以执行更多操作而不从头创建低级操作符时非常有用。例如，RxJava2-Extras 使用 `materialize()` 来实现其操作符中的许多操作，包括 `collectWhile()`。通过将 `onComplete()` 视为一个发射事件，`collectWhile()` 可以将其映射到推动收集缓冲区到下游并开始下一个缓冲区。

否则，你可能不会经常使用它。但如果你需要用它来构建更复杂的转换器，了解它的存在是好的。

# 理解调度器

你可能不会单独使用这样的调度器，就像我们即将在本节中做的那样。你更有可能使用 `observeOn()` 和 `subscribeOn()`。但这里是如何在 Rx 上下文之外独立工作的。

调度器是 RxJava 对线程池和调度任务以由它们执行的概念抽象。这些任务可能根据其调用的执行方法立即执行、延迟执行或周期性重复执行。这些执行方法是 `scheduleDirect()` 和 `schedulePeriodicallyDirect()`，它们有几个重载。下面，我们使用计算调度器来执行一个立即任务、一个延迟任务和一个重复任务，如下所示：

```java
import io.reactivex.Scheduler;
import io.reactivex.schedulers.Schedulers;

import java.util.concurrent.TimeUnit;

public class Launcher {

    public static void main(String[] args) {

        Scheduler scheduler = Schedulers.computation();

        //run task now
        scheduler.scheduleDirect(() -> System.out.println("Now!"));

        //delay task by 1 second
        scheduler.scheduleDirect(() -> 
System.out.println("Delayed!"), 1, TimeUnit.SECONDS);

        //repeat task every second
        scheduler.schedulePeriodicallyDirect(() -> 
System.out.println("Repeat!"), 0, 1, TimeUnit.SECONDS);

        //keep alive for 5 seconds
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

你的输出可能如下所示：

```java
Now!
Repeat!
Delayed!
Repeat!
Repeat!
Repeat!
Repeat!
Repeat!
```

`scheduleDirect()` 将只执行一次任务，并接受可选的重载来指定时间延迟。`schedulePeriodicallyDirect()` 将无限重复。有趣的是，所有这些方法都返回一个 `Disposable`，以便取消正在执行或等待执行的任务。

这三种方法会自动将任务传递给一个 `Worker`，这是一个抽象，它围绕一个单线程进行工作，该线程按顺序执行分配给它的任务。你实际上可以调用调度器的 `createWorker()` 方法来显式获取一个 Worker 并直接将任务委托给它。它的 `schedule()` 和 `schedulePeriodically()` 方法与调度器的 `scheduleDirect()` 和 `schedulePeriodicallyDirect()` 方法操作类似（并且也返回可取消的），但它们是由指定的 Worker 执行的。当你完成一个 Worker 时，你应该取消它，以便它可以被丢弃或返回到 `Scheduler`。以下是我们之前示例使用 `Worker` 的等效示例：

```java
import io.reactivex.Scheduler;
import io.reactivex.schedulers.Schedulers;

import java.util.concurrent.TimeUnit;

public class Launcher {

    public static void main(String[] args) {

        Scheduler scheduler = Schedulers.computation();
        Scheduler.Worker worker = scheduler.createWorker();

        //run task now
        worker.schedule(() -> System.out.println("Now!"));

        //delay task by 1 second
        worker.schedule(() -> System.out.println("Delayed!"), 1,
TimeUnit.SECONDS);

        //repeat task every second
        worker.schedulePeriodically(() ->
System.out.println("Repeat!"), 0, 1, TimeUnit.SECONDS);

        //keep alive for 5 seconds, then dispose Worker
        sleep(5000);
        worker.dispose();
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

你可能会得到以下输出：

```java
Now!
Repeat!
Repeat!
Delayed!
Repeat!
Repeat!
Repeat!
Repeat!
```

当然，每个调度器的实现方式都不同。一个调度器可能使用一个线程或多个线程。它可能缓存和重用线程，也可能完全不重用它们。它可能使用 Android 线程或 JavaFX 线程（正如我们在本书中看到的 RxAndroid 和 RxJavaFX）。但这就是调度器的基本工作方式，也许你可以看到为什么它们在实现 RxJava 操作符时很有用。
