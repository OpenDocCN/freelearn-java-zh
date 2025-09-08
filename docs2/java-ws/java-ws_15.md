# 15. 使用流处理数据

概述

本章讨论了 Java 中的 Stream API，它允许你用更少的代码行有效地编写程序。一旦你掌握了并行流和顺序流之间的区别（在早期章节中定义和概述），你将能够通过首先学习如何创建和关闭这些流来练习使用 Java Stream API 来处理数组和集合。下一步是探索 Java 中可用的不同类型的操作、它们的定义和相应的功能。你将首先遇到的是终端操作和归约器，你将使用它们从元素流中提取数据。然后，你将转向中间操作来过滤、映射以及其他方式修改流结构。最后，在本章的最后练习和活动中，你将学习如何应用不同类型的收集器来将流元素包装在新容器中。

# 简介

Java 8 引入了新的 Stream API。有了流，Java 程序员现在可以使用一种更声明性的编程风格来编写程序，这种风格你之前只在函数式编程语言或函数式编程库中见过。

使用流，你现在可以用更少的代码行写出更具有表现力的程序，并且可以轻松地对大列表上的多个操作进行链式调用。流还使得在列表上并行化操作变得简单——也就是说，如果你有非常大的列表或复杂的操作。关于流，有一件重要的事情需要记住，那就是，尽管它们可能看起来像是一个改进的集合，但实际上并不是。流没有自己的存储；相反，它们使用提供源的数据存储。

在 Java 中，有四种类型的流：`Stream`，用于流式传输对象；`IntStream`，用于流式传输整数；`LongStream`，用于流式传输长整型；最后是`DoubleStream`，当然，用于流式传输双精度浮点数。所有这些流都以完全相同的方式工作，除了它们专门用于处理各自的数据类型。

注意

深入代码，你会发现这些类型只是指向`StreamSupport`类的静态方法的接口。这是任何想要编写特定流库的人的核心 API。然而，在构建应用程序时，通常可以使用四个标准流接口和静态生成函数。

流的来源可以是单个元素、集合、数组，甚至是文件。在流源之后是一系列中间操作，它们构成了核心管道。管道以终端操作结束，通常，它要么遍历剩余的元素以创建副作用，要么将它们减少到特定值——例如，计算最后一个流中剩余元素的数量。

注意

流是延迟构建和执行的。这意味着只有在执行终端操作时，流才会运行。源元素也只有在需要时才会读取；也就是说，只有所需的元素会被传递到下一个操作。

# 创建流

在 Java 中创建流有多种方式；其中最简单的是使用`Stream.of()`函数。这个函数可以接受单个对象或`varargs`中的多个对象：

```java
Stream<Object> objectStream = Stream.of(new Object());
```

如果你流中有多个对象，则使用`varargs`版本：

```java
Stream<Object> objectStream = Stream.of(new Object(), new Object(), new Object());
```

这些流的原始版本以相同的方式工作；只需将`Object`实例替换为整数、长整型或双精度浮点型。

你还可以从不同的集合中创建流——例如，列表和数组。从列表创建流看起来像这样：

```java
List<String> stringList = List.of("string1", "string2", "string3");
Stream<String> stringStream = stringList.stream();
```

要从项目数组创建流，你可以使用`Arrays`类，就像原始版本的流一样：

```java
String[] stringArray = new String[]{"string1", "string2", "string3"};
Stream<String> stringStream = Arrays.stream(stringArray);
```

有一种特殊的流类型可以优雅地处理令人讨厌的 null 类型，具体如下：

```java
Stream<Object> nullableStream = Stream.ofNullable(new Object());
```

这个流将取一个单独的对象，该对象可以是 null。如果对象为 null，则将生成一个空流；或者，如果对象不为 null，则将生成一个包含该单个对象的流。当然，在不确定源状态的情况下，这可以非常方便。

生成元素流的另一种方法是使用`Stream.iterate()`生成函数。这个函数将在你告诉它停止之前，在流中生成无限数量的元素，从种子元素开始：

```java
Stream<Integer> stream = Stream.iterate(0, (i) -> {
    return i + 1;
}).limit(5);
```

在这个例子中，我们正在创建一个包含五个元素的流，从索引`0`开始。这个流将包含元素`0`、`1`、`2`、`3`和`4`：

注意

如果不提供适当的限制，`Stream.iterate()`生成函数可能会非常危险。有几种方法可以创建无限流——通常是通过错误地放置操作顺序或忘记对流应用限制。

此外，还有一个特殊的`Builder`类，它嵌入在`Stream`类型中。这个`Builder`类允许你在创建元素时添加元素；它消除了需要保持`ArrayList`或其他集合作为元素临时缓冲区的需求。

`Builder`类有一个非常简单的 API；你可以将一个元素`accept()`到构建器中，这在你想从循环中生成元素时非常完美：

```java
Stream.Builder<String> streamBuilder = Stream.builder();
for (int i = 0; i < 10; i++) {
    streamBuilder.accept("string" + i);
}
```

你也可以向构建器中`add()`元素。`add()`方法允许链式调用，这在你不希望从循环中生成元素，而是希望在一行中添加它们时非常完美：

```java
Stream.Builder<String> streamBuilder = Stream.builder();
streamBuilder.add("string1").add("string2").add("string3");
```

使用构建器创建流时，当所有方法都已添加后，你可以调用`build()`方法。然而，请注意，如果在调用`build()`方法之后尝试向构建器中添加元素，它将抛出`IllegalStateException`异常：

```java
Stream<String> stream = streamBuilder.build();
```

所有这些创建流的方法都使用相同的底层辅助类，称为`StreamSupport`。此类具有创建具有不同属性的流的许多有用和高级方法。所有这些流的共同特征是`Spliterator`。

## 并行流

在 Java Stream API 中，流要么是顺序的，要么是并行的。顺序流仅使用单个线程来执行任何操作。通常，您会发现这个流足以解决大多数问题；然而，有时您可能需要多个核心上的多个线程运行。

并行流在多个核心上的多个线程上并行操作。它们在 JVM 中使用`ForkJoinPool`来启动多个线程。当您发现自己处于性能热点时，它们可以是一个非常强大的工具。然而，由于并行流使用多个线程，除非需要，否则您应该谨慎使用它们；并行流的开销可能会产生比解决的问题更多的问题。

注意

并行流是一把双刃剑。在某些情况下，它们可能非常有用，然而，同时，它们也可能完全锁定您的程序。因为并行流使用公共的`ForkJoinPool`，它们可能会产生线程，这些线程可能会阻塞您的应用程序和其他系统组件，以至于用户会受到 影响。

要创建并行流，您可以使用`Collections.parallelStream()`方法，它将尝试创建一个并行流：

```java
List.of("string1", "string2", "string3").parallelStream()
```

或者，您可以使用`BaseStream.parallel()`中间操作使流并行化：

```java
List.of(1, 2, 3).stream().parallel()
```

注意，在源和终端操作之间，您可以使用`BaseStream.parallel()`或`BaseStream.sequential()`操作更改流的类型。如果需要更改流的底层状态，这些操作将对流产生影响；如果流已经具有正确的状态，它将简单地返回自身。多次调用`BaseStream.parallel()`对性能没有影响：

```java
List.of(1, 2, 3).stream().parallel().parallel().parallel()
```

## 遭遇顺序

根据流源的类型，它可能具有不同的遭遇顺序。例如，列表具有内置的元素排序——也称为索引。源排序还意味着元素将以该顺序被遇到；然而，您可以使用`BaseStream.unordered()`和`Stream.sorted()`中间操作来更改这种遭遇顺序。

`unordered()`操作不会改变流的排序；相反，它只尝试移除一个特定属性并通知我们流是否已排序。元素仍然具有特定的顺序。无序流的整个目的就是在应用于并行流时使其他操作更高效。将`unordered()`操作应用于顺序流将使其非确定性。

## 关闭流

与之前 Java 版本的流类似，`InputStream` 和 `OutputStream`，Stream API 包含一个 `close()` 操作。然而，在大多数情况下，你实际上根本不需要担心关闭你的流。你应该担心关闭流的情况是当源是一个系统资源——例如文件或套接字——需要关闭以避免占用系统资源。

`close()` 操作返回 void，这意味着在调用 `close()` 之后，流将无法进行任何其他中间或终端操作；尽管可以在流关闭时注册 `close` 处理器来通知。`close` 处理器是一个 `Runnable` 函数式接口；最好使用 lambda 函数来注册它们：

```java
Stream.of(1, 2, 3, 4).onClose(() -> {
    System.out.println("Closed");
}).close();
```

你可以在你的管道中注册任意数量的 `close` 处理器。即使其中任何一个处理器在其代码中抛出异常，`close` 处理器也总是会运行。此外，值得注意的是，它们将始终以它们被添加到管道中的相同顺序被调用，而不管流的遇到顺序如何：

```java
Stream.of(1, 2, 3, 4).onClose(() -> {
    System.out.println("Close handler 1");
}).onClose(() -> {
    System.out.println("Close handler 2");
}).onClose(() -> {
    System.out.println("Close handler 3");
}).close();
```

注意

即使可以在任何流上注册关闭处理器，如果流不需要关闭，它可能实际上不会运行。

自 Java 7 以来，有一个名为 `AutoCloseable` 的接口，它将尝试在 try-with-resources 语句中自动关闭所持有的资源。所有流都继承自的 `BaseStream` 接口扩展了这个 `AutoCloseable` 接口。这意味着任何流如果被包裹在 try-with-resources 语句中，都会尝试自动释放资源：

```java
try (Stream<Integer> stream = Stream.of(6, 3, 8, 12, 3, 9)) {
    boolean matched = stream.onClose(() -> {
        System.out.println("Closed");
    }).anyMatch((e) -> {
        return e > 10;
    });
    System.out.println(matched);
}
```

虽然前面的例子确实可行，但通常没有必要在基本流上包裹 try-with-resources 语句，除非你明确需要在流运行完成后执行逻辑。此示例将首先将 `true` 打印到终端，然后打印 `Closed`。

## 终端操作

每个管道都需要以终端操作结束；没有这个，管道将不会执行。与中间操作不同，终端操作可能有各种返回值，因为它们标志着管道的结束。在终端操作之后，你不能应用另一个操作。

注意

当对流应用终端操作时，你不能再使用该流。因此，在代码中存储流的引用可能会导致混淆，不清楚该引用可能如何使用——不允许“分割”流到两个不同的用例。如果你尝试对一个已经执行了终端操作的流应用操作，那么它将抛出一个带有消息`stream has already been operated upon or closed`的 `IllegalStateException`。

Stream API 中有 16 种不同的终端操作——每个都有其特定的用例。以下是对每个操作的说明：

+   `forEach`：这个终端操作像是一个普通的`for`循环；它将为流中的每个元素运行一些代码。这不是一个线程安全的操作，所以如果你发现自己正在使用共享状态，你需要提供同步：

    ```java
    Stream.of(1, 4, 6, 2, 3, 7).forEach((n) -> { System.out.println(n); });
    ```

    如果在这个并行管道上应用此操作，元素被作用的方式的顺序将无法保证：

    ```java
    Stream.of(1, 4, 6, 2, 3, 7).parallel().forEach((n) -> { System.out.println(n); });
    ```

    如果元素被作用的方式的顺序很重要，你应该使用`forEachOrdered()`终端操作。

+   `forEachOrdered`：与`forEach()`终端操作类似，这将允许你对流中的每个元素执行一个操作。然而，`forEachOrdered()`操作将保证处理元素的顺序，无论它们在多少个线程上被处理：

    ```java
    Stream.of(1, 4, 6, 2, 3, 7).parallel().forEachOrdered((n) -> { System.out.println(n); });
    ```

    这里，你可以看到具有定义的遭遇顺序的并行流。使用`forEachOrdered()`操作，它将始终按自然顺序、索引顺序遇到元素。

+   `toArray`：这两个终端操作将允许你将流中的元素转换为数组。基本版本将生成一个`Object`数组：

    ```java
    Object[] array = Stream.of(1, 4, 6, 2, 3, 7).toArray();
    ```

    如果你需要一个特定类型的数组，你可以提供一个构造器引用来指定所需的数组类型：

    ```java
    Integer[] array = Stream.of(1, 4, 6, 2, 3, 7).toArray(Integer[]::new);
    ```

    第三种选择是为你自己的`toArray()`操作编写自己的生成器：

    ```java
    Integer[] array = Stream.of(1, 4, 6, 2, 3, 7).toArray(elements -> new Integer[elements]);
    ```

+   `reduce`：对一个流进行归约意味着只提取该流元素的有用部分，并将它们归约为一个单一值。有两个通用的`reduce`操作可用。第一个，更简单的一个，接受一个累加函数作为参数。它通常在流上应用了映射操作之后使用：

    ```java
    int sum = Stream.of(1, 7, 4, 3, 9, 6).reduce(0, (a, b) -> a + b);
    ```

    第二个更复杂的版本采用了一个同时作为归约初始值的身份。它还要求一个累加函数，其中归约发生，以及一个组合函数来定义如何将两个元素归约：

    ```java
    int sum = Stream.of(1, 7, 4, 3, 9, 6).reduce(0, (total, i) -> total + i, (a, b) -> a + b );
    ```

    在这个例子中，累加函数将组合函数的结果加到身份值上，在这种情况下，是归约的总和。

+   `sum`：这是一个更具体的归约操作，它将流中的所有元素相加。这个终端操作仅适用于`IntStream`、`LongStream`和`DoubleStream`。要在更通用的流中使用此功能，你需要实现一个使用`reduce()`操作的管道，通常在`map()`操作之前：

    ```java
    int intSum = IntStream.of(1, 7, 4, 3, 9, 6).sum();
    System.out.println(intSum);
    ```

    这将打印出结果为`30`。以下示例说明了`LongStream`的使用：

    ```java
    long longSum = LongStream.of(7L, 4L, 9L, 2L).sum();
    System.out.println(longSum);
    ```

    这将打印出结果为`22`。以下示例说明了`DoubleStream`的使用：

    ```java
    double doubleSum = DoubleStream.of(5.4, 1.9, 7.2, 6.1).sum();
    System.out.println(doubleSum);
    ```

    这将打印出结果为`20.6`。

+   `collect`：收集操作类似于 reduce 操作，因为它接受流中的元素并创建一个新的结果。然而，与 reduce 操作不同，`collect`可以接受元素并生成一个新的容器或集合，该集合包含所有剩余的元素；例如，一个列表。通常，你会使用`Collectors`的`help`类，因为它包含许多现成的收集操作：

    ```java
    List<Integer> items = Stream.of(6, 3, 8, 12, 3, 9).collect(Collectors.toList());
    System.out.println(items);
    ```

    这将在控制台打印`[6, 3, 8, 12, 3, 9]`。你可以在*使用收集器*部分查看`Collectors`的更多用法。另一种选择是为`collect()`操作编写自己的供应商、累加器和组合器：

    ```java
    List<Integer> items = Stream.of(6, 3, 8, 12, 3, 9).collect(
            () -> { return new ArrayList<Integer>(); },
            (list, i) -> { list.add(i); },
            (list, elements) -> { list.addAll(elements); });
    System.out.println(items);
    ```

    当然，在这个例子中，可以通过使用方法引用来简化：

    ```java
    List<Integer> items = Stream.of(6, 3, 8, 12, 3, 9).collect(ArrayList::new, List::add, List::addAll);
    System.out.println(items);
    ```

+   `min`：正如其名所示，这个终端操作将返回流中所有元素的最小值，用`Optional`包装，并按照指定的`Comparator`进行指定。在应用此操作时，大多数情况下你会使用`Comparator.comparingInt()`、`Comparator.comparingLong()`或`Comparator.comparingDouble()`静态辅助函数：

    ```java
    Optional min = Stream.of(6, 3, 8, 12, 3, 9).min((a, b) -> { return a - b;});
    System.out.println(min);
    ```

    这应该写入`Optional[3]`。

+   `max`：与`min()`操作相反，`max()`操作返回具有最大值的元素的值，根据指定的`Comparator`进行包装，并用`Optional`包装：

    ```java
    Optional max = Stream.of(6, 3, 8, 12, 3, 9).max((a, b) -> { return a - b;});
    System.out.println(max);
    ```

    这将在终端打印`Optional[12]`。

+   `average`：这是一个特殊的终端操作，仅在`IntStream`、`LongStream`和`DoubleStream`上可用。它返回一个包含流中所有元素平均值的`OptionalDouble`：

    ```java
    OptionalDouble avg = IntStream.of(6, 3, 8, 12, 3, 9).average();
    System.out.println(avg);
    ```

    这将为你提供一个包含值`6.833333333333333`的`Optional`。

+   `count`：这是一个简单的终端操作，返回流中的元素数量。值得注意的是，有时`count()`终端操作会找到计算流大小的更有效的方法。在这些情况下，管道甚至不会执行：

    ```java
    long count = Stream.of(6, 3, 8, 12, 3, 9).count();
    System.out.println(count);
    ```

+   `anyMatch`：如果流中的任何元素匹配指定的谓词，则`anyMatch()`终端操作将返回`true`：

    ```java
    boolean matched = Stream.of(6, 3, 8, 12, 3, 9).anyMatch((e) -> { return e > 10; });
    System.out.println(matched);
    ```

    由于有一个值大于 10 的元素，这个管道将返回`true`。

+   `allMatch`：如果流中的所有元素都匹配指定的谓词，则`allMatch()`终端操作将返回`true`：

    ```java
    boolean matched = Stream.of(6, 3, 8, 12, 3, 9).allMatch((e) -> { return e > 10; });
    System.out.println(matched);
    ```

    由于这个源有值小于 10 的元素，它应该返回`false`。

+   `noneMatch`：与`allMatch()`相反，如果流中的没有任何元素匹配指定的谓词，则`noneMatch()`终端操作将返回`true`：

    ```java
    boolean matched = Stream.of(6, 3, 8, 12, 3, 9).noneMatch((e) -> { return e > 10; });
    System.out.println(matched);
    ```

    因为流中有值大于 10 的元素，所以这也会返回`false`。

+   `findFirst`：这检索流中的第一个元素，并用`Optional`包装：

    ```java
    Optional firstElement = Stream.of(6, 3, 8, 12, 3, 9).findFirst();
    System.out.println(firstElement);
    ```

    这将在终端打印`Optional[6]`。如果流中没有元素，它将打印`Optional.empty`。

+   `findAny`：与`findFirst()`终端操作类似，`findAny()`操作将返回一个被`Optional`包装的元素。然而，这个操作将返回剩余元素中的任何一个。你实际上永远不应该假设它会返回哪个元素。通常，这个操作会比`findFirst()`操作运行得更快，尤其是在并行流中。当你只需要知道是否还有剩余元素，但并不真正关心哪些元素剩余时，这是理想的：

    ```java
    Optional firstElement = Stream.of(7, 9, 3, 4, 1).findAny();
    System.out.println(firstElement);
    ```

+   `iterator`：这是一个终端操作，它生成一个迭代器，让你遍历元素：

    ```java
    Iterator<Integer> iterator = Stream.of(1, 2, 3, 4, 5, 6)
            .iterator();
    while (iterator.hasNext()) {
        Integer next = iterator.next();
        System.out.println(next);
    }
    ```

+   `summaryStatistics`：这是一个特殊的终端操作，适用于`IntStream`、`LongStream`和`DoubleStream`。它将返回一个特殊类型——例如，`IntSummaryStatistics`——描述流中的元素：

    ```java
    IntSummaryStatistics intStats = IntStream.of(7, 9, 3, 4, 1).summaryStatistics();
    System.out.println(intStats);
    LongSummaryStatistics longStats = LongStream.of(6L, 4L, 1L, 3L, 7L).summaryStatistics();
    System.out.println(longStats);
    DoubleSummaryStatistics doubleStats = DoubleStream.of(4.3, 5.1, 9.4, 1.3, 3.9).summaryStatistics();
    System.out.println(doubleStats);
    ```

    这将打印出三个流的所有摘要到终端，其外观应该如下所示：

    ```java
    IntSummaryStatistics{count=5, sum=24, min=1, average=4,800000, max=9}
    LongSummaryStatistics{count=5, sum=21, min=1, average=4,200000, max=7}
    DoubleSummaryStatistics{count=5, sum=24,000000, min=1,300000, average=4,800000, max=9,400000}
    ```

# 中间操作

流可以接受任意数量的中间操作，这些操作是在创建流之后进行的。中间操作通常是某种类型的过滤器或映射，但还有其他类型。每个中间操作都会返回另一个流；这样，你可以将任意数量的中间操作链接到你的管道中。

中间操作的顺序非常重要，因为从操作返回的流将只引用前一个流中剩余或所需的元素。

有几种不同的中间操作类型。以下是对每种类型的解释：

+   `filter`：正如其名所示，这个中间操作将从流中返回一个子集元素。它在应用匹配模式时使用谓词，这是一个返回`Boolean`的功能接口。实现这个接口最简单和最常见的方式是使用 lambda 函数：

    ```java
    Stream.of(1, 2, 3, 4, 5, 6)
            .filter((i) -> { return i > 3; })
            .forEach(System.out::println);
    ```

    在这个示例中，`filter`方法将过滤掉任何值等于或小于 3 的元素。然后，`forEach()`终端操作将取剩余的元素，并在循环中打印它们。

+   `map`：`map`操作将应用一个特殊函数到流中的每个元素，并返回修改后的元素：

    ```java
    Stream.of("5", "3", "8", "2")
            .map((s) -> { return Integer.parseInt(s); })
            .forEach((i) -> { System.out.println(i > 3); });
    ```

    这个管道将取字符串，使用`map()`操作将它们转换为整数，然后根据解析的字符串值是否大于 3 来打印`true`或`false`。这只是`map`的一个简单示例；这种方法在将流转换为非常不同的东西时非常灵活。

    此外，这个中间操作还有特殊的版本，将返回整数值、长值和双精度值。它们分别称为`mapToInt()`、`mapToLong()`和`mapToDouble()`：

    ```java
    Stream.of("5", "3", "8", "2")
            .mapToInt((i) -> { return Integer.parseInt(i); })
            .forEach((i) -> { System.out.println(i > 3); });
    ```

    注意，这些特殊的`map`操作将返回`IntStream`、`LongStream`或`DoubleStream`，而不是`Stream<Integer>`、`Stream<Long>`或`Stream<Double>`。

+   `flatMap`: 这为你提供了一个将多维数据结构扁平化到一个单一流中的简单方法——例如，一个包含对象或数组的对象的流。使用 `flatMap()`，你可以将这些子元素连接成一个单一的流：

    ```java
    Stream.of(List.of(1, 2, 3), List.of(4, 5, 6), List.of(7, 8, 9))
            .flatMap((l) -> { return l.stream(); })
            .forEach((i) -> { System.out.print(i); });
    ```

    在这个示例管道中，我们从一个多个列表中创建一个流；然后，在 `flatMap` 操作中，我们提取每个列表的流。`flatMap` 操作然后将它们连接成一个单一的流，我们通过 `forEach` 遍历它。终端将打印出完整的流：`123456789`。

    `flatMap` 函数也存在于整数、长和双精度特殊操作中——`flatMapToInt`、`flatMapToLong` 和 `flatMapToDouble`——当然，它们将返回相应的类型流：

+   `distinct`: 这将返回流中的所有唯一元素。如果流中有重复的元素，则将返回第一个项目：

    ```java
    Stream.of(1, 2, 2, 2, 2, 3)
            .distinct()
            .forEach((i) -> { System.out.print(i); });
    ```

    在这里，我们从一个包含六个元素的流开始，然而，其中四个在值上是相同的。`distinct()` 操作将过滤这些元素，剩下的三个将被打印到终端。

+   `sorted`: `sorted` 中间操作存在两个版本。第一个版本没有参数，假设 `map` 的元素可以按自然顺序排序——实现 `Comparable` 接口。如果它们不能排序，则将抛出异常：

    ```java
    Stream.of(1, 3, 6, 4, 5, 2)
            .sorted()
            .forEach((i) -> { System.out.print(i); });
    ```

    `sorted` 操作的第二个版本接受一个 `Comparator` 作为参数，并将相应地返回排序后的元素：

    ```java
    Stream.of(1, 3, 6, 4, 5, 2)
            .sorted((a, b) -> a - b)
            .forEach((i) -> { System.out.print(i); });
    ```

+   `unordered`: 与 `sorted` 相反，`unordered` 中间操作将对流的元素施加无序的遭遇顺序。在并行流上使用此操作有时可以提高性能，因为某些中间和终端状态操作在元素顺序更宽松的情况下表现更好：

    ```java
    Stream.of(1, 2, 3, 4, 5, 6)
            .unordered()
            .forEach((i) -> { System.out.print(i); });
    System.out.println();
    Stream.of(1, 2, 3, 4, 5, 6)
            .parallel()
            .unordered()
            .forEach((i) -> { System.out.print(i); });
    ```

+   `limit`: 这个操作返回一个包含 `n` 个元素的新的流。如果元素的数量少于请求的限制，则没有效果：

    ```java
    Stream.of(1, 2, 3, 4, 5, 6)
            .limit(3)
            .forEach((i) -> { System.out.print(i); });
    ```

    运行此示例的结果将是 `123`，忽略任何超过第三个元素的元素。

+   `skip`: 这个操作将跳过此流的前 `n` 个元素，并返回一个包含剩余元素的新流：

    ```java
    Stream.of(1, 2, 3, 4, 5, 6)
            .skip(3)
            .forEach((i) -> { System.out.print(i); });
    ```

    这将打印 `456` 到终端，跳过前三个元素。

+   `boxed`: 特殊的原始流 `IntStream`、`LongStream` 和 `DoubleStream` 都可以访问 `boxed()` 操作。此操作将“封装”每个原始元素在相应类型的类版本中，并返回该流。`IntStream` 将返回 `Stream<Integer>`，`LongStream` 将返回 `Stream<Long>`，而 `DoubleStream` 将返回 `Stream<Double>`：

    ```java
    IntStream.of(1, 2)
            .boxed()
            .forEach((i) -> { System.out.println(i + i.getClass().getSimpleName()); });
    System.out.println();
    LongStream.of(3, 3)
            .boxed()
            .forEach((l) -> { System.out.println(l + l.getClass().getSimpleName()); });
    System.out.println();
    DoubleStream.of(5, 6)
            .boxed()
            .forEach((d) -> { System.out.println(d + d.getClass().getSimpleName()); });
    ```

    这个示例将取每个原始流，将其封装在相应的对象类型中，然后打印出值以及该类型的类名：

    ```java
    1Integer
    2Integer
    3Long
    4Long
    5.0Double
    6.0Double
    ```

+   `takeWhile`：这是一种特殊类型的操作，根据流是有序的还是无序的，其行为不同。如果流是有序的——也就是说，它有一个定义的遭遇顺序——它将返回一个包含最长匹配元素序列的流，该序列从流中的第一个元素开始。这个始终以第一个元素开始的元素流有时也被称为前缀：

    ```java
    Stream.of(2, 2, 2, 3, 1, 2, 5)
            .takeWhile((i) -> { return i == 2; })
            .forEach((i) -> { System.out.println(i); });
    ```

    此管道将`222`打印到终端。然而，你应该注意，如果第一个元素不匹配谓词，此操作将返回一个空流。这是因为`takeWhile()`的内部工作原理；也就是说，它将从第一个元素开始，直到第一个元素失败匹配——给你一个空流：

    ```java
    Stream.of(1, 2, 2, 3, 1, 2, 5)
            .takeWhile((i) -> { return i == 2; })
            .forEach((i) -> { System.out.println(i); });
    ```

    如果流是无序的——也就是说，它没有定义的遭遇顺序——`takeWhile()`操作可能会返回任何匹配的元素子集，包括空子集。在这种情况下，`filter()`操作可能更合适。

+   `dropWhile`：`dropWhile()`操作与`takeWhile()`相反。就像`takeWhile()`一样，它将根据流是有序的还是无序的不同而有所不同。如果流是有序的，它将丢弃与谓词匹配的最长前缀，而不是像`takeWhile()`那样返回前缀：

    ```java
    Stream.of(2, 2, 2, 3, 1, 2, 5)
            .dropWhile((i) -> { return i == 2; })
            .forEach((i) -> { System.out.print(i); });
    ```

    此管道将`3125`打印到终端，丢弃匹配的前缀，即前三个 2。如果流是无序的，操作可能会丢弃任何元素子集，或者丢弃空子集，实际上返回整个流。在使用此操作于无序流时要小心。

+   `ForkJoinPool`。大多数流都是顺序的，除非特别创建为并行，或者使用此中间操作转换为并行。

+   **顺序**：这返回一个顺序流，是并行的对立面。

+   `peek`：这种中间操作主要用于在应用其他中间操作之后检查流。通常，目标是了解操作如何影响元素。在以下示例中，我们正在打印每个元素如何通过管道中的每个流操作：

    ```java
    long count = Stream.of(6, 5, 3, 8, 1, 9, 2, 4, 7, 0)
            .peek((i) -> { System.out.print(i); })
            .filter((i) -> { return i < 5; })
            .peek((i) -> { System.out.print(i); })
            .map((i) -> { return String.valueOf(i); })
            .peek((p) -> { System.out.print(p); })
            .count();
    System.out.println(count);
    ```

    在这个例子中，终端将读取`653338111922244470005`。我们可以快速推断的是，任何值大于或等于 5 的元素只会打印一次。`Peek`将依次跟随整个流中的每个元素；这就是为什么顺序可能看起来很奇怪。6 和 5 只会打印一次，因为它们在第一个`peek`操作后被过滤掉。然而，3 将在所有三个`peek()`操作中被触发，因此有一连串的三个 3。输出中的最后一个数字 5 只是剩余元素的数量。

    虽然`peek()`操作最常用于在元素遍历管道时检查元素，但也可以使用这些操作来修改流中的元素。考虑以下类定义：

    ```java
    class MyItem {
        int value;
        public MyItem(int value) {
            this.value = value;
        }
    }
    ```

    然后，考虑将这些值中的几个添加到一个应用了修改 `peek` 操作的流中：

    ```java
    long sum = Stream.of(new MyItem(1), new MyItem(2), new MyItem(3))
            .peek((item) -> {
                item.value = 0;
            })
            .mapToInt((item) -> { return item.value; })
            .sum();
    System.out.println(sum);
    ```

如果我们忽略 `peek()` 操作，这些对象的总和应该是 6。然而，`peek` 操作正在将每个对象修改为零值——实际上使总和为零。虽然这是可能的，但它从未被设计成这样使用。使用 `peek()` 来修改是不推荐的，因为它不是线程安全的，访问任何共享状态可能会引发异常。不同的 `map()` 操作通常是更好的选择。

## 练习 1：使用 Stream API

一个允许客户同时收集和保存多个不同购物车的在线杂货店要求你实现他们的多购物车系统的联合结账。结账程序应该连接所有购物车中所有商品的价格，然后向客户展示。为此，执行以下步骤：

1.  如果 IntelliJ 已经启动但没有打开项目，那么请选择 `Create New Project`。如果 IntelliJ 已经打开了项目，那么请从菜单中选择 `File` | `New` | `Project`。

1.  在 `New Project` 对话框中，选择 `Java project`，然后点击 `Next`。

1.  打开复选框以从模板创建项目。选择 `Command Line App`，然后点击 `Next`。

1.  给新项目命名为 `Chapter15`。

1.  IntelliJ 会为你提供一个默认的项目位置。如果你希望选择一个，你可以在这里输入。

1.  将包名设置为 `com.packt.java.chapter15`。

1.  点击 `Finish`。

    IntelliJ 将创建你的项目，名为 `Chapter15`，具有标准的文件夹结构。IntelliJ 还会创建你的应用程序的主入口点，名为 `Main.java`。

1.  将此文件重命名为 `Exercise1.java`。完成后，它应该看起来像这样：

    ```java
    package com.packt.java.chapter15;
    public class Exercise1 {
        public static void main(String[] args) {
        // write your code here
        }
    }
    ```

1.  创建一个新的内部类，称为 `ShoppingArticle`。将其设置为静态，这样我们就可以轻松地从程序的主入口点访问它。这个类应该包含文章的名称和该文章的价格。让 `price` 是一个双精度变量：

    ```java
        private static final class ShoppingArticle {
            final String name;
            final double price;
            public ShoppingArticle(String name, double price) {
                this.name = name;
                this.price = price;
            }
        }
    ```

1.  现在创建一个简单的 `ShoppingCart` 类。在这个版本中，我们将只允许购物车中的每篇文章有一个项目，所以一个列表就足够用来在 `ShoppingCart` 中保存文章了：

    ```java
        private static final class ShoppingCart {
            final List<ShoppingArticle> mArticles;
            public ShoppingCart(List<ShoppingArticle> list) {
                mArticles = List.copyOf(list);
            }
        }
    ```

1.  创建你的第一个购物车，`fruitCart`，并向其中添加三种水果文章——`Orange`、`Apple` 和 `Banana`——每种各一个。设置每单位价格为 `1.5`、`1.7` 和 `2.2` `Java-$`：

    ```java
    public class Exercise1 {
        public static void main(String[] args) {
            ShoppingCart fruitCart = new ShoppingCart(List.of(
                    new ShoppingArticle("Orange", 1.5),
                    new ShoppingArticle("Apple", 1.7),
                    new ShoppingArticle("Banana", 2.2)
            ));
        }
    ```

1.  创建另一个 `ShoppingCart`，但这次是蔬菜——`Cucumber`、`Salad` 和 `Tomatoes`。同样，为它们设置价格，Java-$ 为 `0.8`、`1.2` 和 `2.7`：

    ```java
            ShoppingCart vegetableCart = new ShoppingCart(List.of(
                    new ShoppingArticle("Cucumber", 0.8),
                    new ShoppingArticle("Salad", 1.2),
                    new ShoppingArticle("Tomatoes", 2.7)
            ));
        }
    ```

1.  用第三个和最后的 `shoppingCart` 包裹测试购物车，其中包含一些肉类和鱼类。它们通常比水果和蔬菜贵一些：

    ```java
            ShoppingCart meatAndFishCart = new ShoppingCart(List.of(
                    new ShoppingArticle("Cod", 46.5),
                    new ShoppingArticle("Beef", 29.1),
                    new ShoppingArticle("Salmon", 35.2)
            ));
        }
    ```

1.  现在是时候开始实现一个函数，该函数将计算购物车中所有商品的总价。声明一个新的函数，它接受一个`ShoppingCart` `vararg`作为参数并返回一个 double 类型。让它成为静态的，这样我们就可以在`main`函数中轻松使用它：

    ```java
        private static double calculatePrice(ShoppingCart... carts) {
        }
    ```

1.  从所有购物车流开始构建一个管道：

    ```java
    private static double calculatePrice(ShoppingCart... carts) {
            return Stream.of(carts)
        }
    ```

1.  向所有购物车添加一个`flatMap()`操作以提取单个`ShoppingArticles`流：

    ```java
        private static double calculatePrice(ShoppingCart... carts) {
            return Stream.of(carts)
                .flatMap((cart) -> { return cart.mArticles.stream(); })
        }
    ```

1.  使用`mapToDouble()`操作提取每个`ShoppingArticle`的价格；这将创建一个`DoubleStream`：

    ```java
        private static double calculatePrice(ShoppingCart... carts) {
            return Stream.of(carts)
                .flatMap((cart) -> { return cart.mArticles.stream(); })
                .mapToDouble((item) -> { return item.price; })
        }
    ```

1.  最后，使用`DoubleStream`中可用的`sum()`方法将所有`ShoppingArticle`的价格减少到总和：

    ```java
    private static double calculatePrice(ShoppingCart... carts) {
        return Stream.of(carts)
                .flatMap((cart) -> { return cart.mArticles.stream(); })
                .mapToDouble((item) -> { return item.price; })
                .sum();
    }
    ```

1.  现在你有一个函数，它将把`ShoppingCart`列表减少到 Java-$的统一总和。你现在要做的就是将这个函数应用到你的`ShoppingCart`类中，然后打印出结果总和到终端，并四舍五入到两位小数：

    ```java
        double sum = calculatePrice(fruitCart, vegetableCart, meatAndFishCart);
        System.out.println(String.format("Sum: %.2f", sum));
    }
    ```

    注意

    你可以在以下位置找到完整的代码：[`packt.live/2qzLaHx`](https://packt.live/2qzLaHx)。

现在，你已经使用功能 Java Stream API 创建了你第一段完整的代码。你创建了一个复杂对象的流，对流的元素应用映射操作以转换它们，然后又应用另一个映射操作以再次转换元素，改变了流类型两次。最后，你将整个流减少到一个单一的基本值，并将其呈现给用户。

## 活动一：对商品应用折扣

通过添加一个在计算最终价格之前对购物车中某些商品应用折扣的功能来改进前面的示例。确保价格计算仍然正确。

注意

这个活动的解决方案可以在第 563 页找到。

# 使用收集器

在 Java 中，**收集器**当你需要从大型数据结构中提取某些数据点、描述或元素时是一个非常强大的工具。它们提供了一种非常易于理解的方式来描述你想要对元素流执行的操作，而不需要编写复杂的逻辑。

`Collector`接口有许多有用的默认实现，你可以轻松开始使用。大多数这些收集器不允许 null 值；也就是说，如果它们在你的流中找到一个 null 值，它们将抛出一个`NullPointerException`。在使用收集器将你的元素减少到这些容器中的任何一种之前，你应该小心处理流中的 null 元素。

以下是对所有默认收集器的介绍：

+   `toCollection`: 这个泛型收集器将允许你将你的元素包装在任何已知实现`Collection`接口的类中；例如`ArrayList`、`HashSet`、`LinkedList`、`TreeSet`和其他：

    ```java
    List.of("one", "two", "three", "four", "five")
            .stream()
            .collect(Collectors.toCollection(TreeSet::new));
    ```

+   `toList`: 这将把你的元素减少到一个`ArrayList`实现。如果你需要一个更具体的列表类型，你应该使用`toCollection()`收集器：

    ```java
    List.of("one", "two", "three", "four", "five")
            .stream()
            .collect(Collectors.toList());
    ```

+   `toUnmodifiableList`：这本质上与 `toList()` 收集器相同，唯一的不同之处在于它使用 `List.of()` 生成器函数来创建不可修改的列表：

    ```java
    List.of("one", "two", "three", "four", "five")
            .stream()
            .collect(Collectors.toUnmodifiableList());
    ```

+   `toSet`：这将在 `HashSet` 中包装元素：

    ```java
    List.of("one", "two", "three", "four", "five")
            .stream()
            .collect(Collectors.toSet());
    ```

+   `toUnmodifiableSet`：这与 `toSet()` 收集器类似，区别在于它将使用 `Set.of()` 生成器来创建一个不可修改的集合：

    ```java
    List.of("one", "two", "three", "four", "five")
            .stream()
            .collect(Collectors.toUnmodifiableSet());
    ```

+   `joining`：这个收集器将使用 `StringBuilder` 将流元素连接成一个字符串，不包含任何分隔字符：

    ```java
    String joined = List.of("one", "two", "three", "four", "five")
            .stream()
            .collect(Collectors.joining());
    System.out.println(joined);
    ```

    这将在终端打印 `onetwothreefourfive`。如果你需要元素之间用逗号分隔，例如，使用 `Collectors.joining(",")`：

    ```java
    String joined = List.of("one", "two", "three", "four", "five")
            .stream()
            .collect(Collectors.joining(","));
    System.out.println(joined);
    ```

    在这个示例中，你会在终端得到 `one,two,three,four,five` 的打印。最后，你还有添加前缀和后缀到生成的字符串的选项：

    ```java
    String joined = List.of("one", "two", "three", "four", "five")
            .stream()
            .collect(Collectors.joining(",", "Prefix", "Suffix"));
    System.out.println(joined);
    ```

    前缀和后缀是添加到字符串上的，而不是每个元素。生成的字符串将看起来像：`Prefixone,two,three,four,fiveSuffix`。

+   `mapping`：这是一种特殊的收集器，允许你在应用定义的收集器之前对流中的每个元素应用映射：

    ```java
    Set<String> mapped = List.of("one", "two", "three", "four", "five")
            .stream()
            .collect(Collectors.mapping((s) -> { return s + "-suffix"; }, Collectors.toSet()));
    System.out.println(mapped);
    ```

    在这里，我们从一个 `List<String>` 的源开始，将其收集到一个 `Set<String>` 中。但在收集之前，我们使用 `mapping()` 收集器将 `-suffix` 字符串连接到每个元素上。

+   `flatMapping`。就像 `flatMap()` 中间操作一样，这个收集器将允许你在收集到新容器之前对流元素应用扁平映射。在以下示例中，我们从一个源 `List<Set<String>>` 开始，然后将其展开为 `Stream<Set<String>>` 并应用 `Collector.toList()`——实际上是将所有集合转换成一个单独的列表：

    ```java
    List<String> mapped = List.of(
            Set.of("one", "two", "three"),
            Set.of("four", "five"),
            Set.of("six")
    )
            .stream()
            .collect(Collectors.flatMapping(
                    (set) -> { return set.stream(); },
                    Collectors.toList())
            );
    System.out.println(mapped);
    ```

+   `filter()` 中间操作，在这里，你可以在对流执行操作之前应用过滤。

    ```java
    Set<String> collected = List.of("Andreas", "David", "Eric")
            .stream()
            .collect(Collectors.filtering(
                    (name) -> { return name.length() < 6; },
                    Collectors.toSet())
            );
    System.out.println(collected);
    ```

+   `collectingAndThen`：这个特殊的收集器将允许你使用一个特殊函数来完成收集；例如，将你的集合转换成一个不可变集合：

    ```java
    Set<String> immutableSet = List.of("Andreas", "David", "Eric")
            .stream()
            .collect(Collectors.collectingAndThen(
                    Collectors.toSet(), 
                    (set) -> { return Collections.unmodifiableSet(set); })
            );
    System.out.println(immutableSet);
    ```

+   `counting`：这产生与 `count()` 中间操作相同的结果：

    ```java
    long count = List.of("Andreas", "David", "Eric")
            .stream()
            .collect(Collectors.counting());
    System.out.println(count);
    ```

+   `minBy`：这个收集器等同于使用 `min()` 终端操作符。以下示例将打印 `Optional[1]` 到终端：

    ```java
    Optional<Integer> smallest = Stream.of(1, 2, 3)
            .collect(Collectors.minBy((a, b) -> { return a - b; });
    System.out.println(smallest);
    ```

+   `maxBy`：使用这个收集器可以得到与使用 `max()` 终端操作符相同的结果：

    ```java
    Optional<Integer> biggest = Stream.of(1, 2, 3)
            .collect(Collectors.maxBy((a, b) -> { return a - b; }));
    System.out.println(biggest);
    ```

+   `summingInt`：这是 `reduce()` 中间操作的替代方案，用于计算流中所有元素的总和：

    ```java
    int sum = Stream.of(1d, 2d, 3d)
            .collect(Collectors.summingInt((d) -> { return d.intValue(); }));
    System.out.println(sum);
    ```

+   `summingLong`：这与 `Collector.summingInt()` 相同，但将产生一个 `long` 类型的总和：

    ```java
    long sum = Stream.of(1d, 2d, 3d)
            .collect(Collectors.summingLong((d) -> { return d.longValue(); }));
    System.out.println(sum);
    ```

+   `summingDouble`：这与 `Collector.summingLong()` 相同，但将产生一个 `double` 类型的总和：

    ```java
    double sum = Stream.of(1, 2, 3)
            .collect(Collectors.summingDouble((i) -> { return i.doubleValue(); }));
    System.out.println(sum);
    ```

+   `averagingInt`：返回传入整数的平均值：

    ```java
    double average = Stream.of(1d, 2d, 3d)
            .collect(Collectors.averagingInt((d) -> { return d.intValue(); }));
    System.out.println(average);
    ```

+   `averagingLong`：返回传入的长整数的平均值：

    ```java
    double average = Stream.of(1d, 2d, 3d)
            .collect(Collectors.averagingLong((d) -> { return d.longValue(); }));
    System.out.println(average);
    ```

+   `averagingDouble`：返回传入参数中数字的平均值：

    ```java
    double average = Stream.of(1, 2, 3)
            .collect(Collectors.averagingDouble((i) -> { return i.doubleValue(); }));
    System.out.println(average);x§
    ```

+   `reduce()`终端操作符，这个收集器从它那里继承了名称和操作。

+   `groupingBy`：这个收集器将根据给定的函数对元素进行分组，并根据给定的集合类型收集它们。考虑以下示例类，描述一辆汽车：

    ```java
    private static class Car {
        String brand;
        long enginePower;
        Car(String brand, long enginePower) {
            this.brand = brand;
            this.enginePower = enginePower;
        }
        public String getBrand() {
            return brand;
        }
        @Override
        public String toString() {
            return brand + ": " + enginePower;
        }
    } 
    ```

    如果你想要根据品牌对几辆汽车进行排序并将它们收集到新的容器中，那么使用`groupingBy()`收集器就很简单：

    ```java
    Map<String, List<Car>> grouped = Stream.of(
            new Car("Toyota", 92),
            new Car("Kia", 104),
            new Car("Hyundai", 89),
            new Car("Toyota", 116),
            new Car("Mercedes", 209))
            .collect(Collectors.groupingBy(Car::getBrand));
    System.out.println(grouped);
    ```

    这里，我们有四种不同的汽车。然后，我们根据汽车的品牌应用`groupingBy()`收集器。这将产生一个`Map<String, List<Car>>`集合，其中`String`是汽车的品牌，`List`包含该品牌的所有汽车。这总是返回`Map`；然而，你可以定义收集分组元素时应使用的集合类型。在下面的例子中，我们将它们分组到`Set`而不是默认列表中：

    ```java
    Map<String, Set<Car>> grouped = Stream.of(
            new Car("Toyota", 92),
            new Car("Kia", 104),
            new Car("Hyundai", 89),
            new Car("Toyota", 116),
            new Car("Mercedes", 209))
            .collect(Collectors.groupingBy(Car::getBrand, Collectors.          toSet()));
    System.out.println(grouped);
    ```

    如果将`groupingBy`收集器与另一个收集器结合使用，它将变得更加强大——例如，`reducing`收集器：

    ```java
    Map<String, Optional<Car>> collected = Stream.of(
            new Car("Volvo", 195),
            new Car("Honda", 96),
            new Car("Volvo", 165),
            new Car("Volvo", 165),
            new Car("Honda", 104),
            new Car("Honda", 201),
            new Car("Volvo", 215))
            .collect(Collectors.groupingBy(Car::getBrand, Collectors.          reducing((carA, carB) -> {
                if (carA.enginePower > carB.enginePower) {
                    return carA;
                }
                return carB;
            })));
    System.out.println(collected);
    ```

    在这个例子中，我们根据品牌对汽车进行分组，然后只显示每个品牌中最强大的引擎的汽车。当然，这种组合也可以与其他收集器一起使用，例如过滤、计数等：

+   `groupingBy`收集器，并且具有完全相同的 API。

+   `partitioningBy`：`partitioningBy`收集器的工作方式与`groupingBy`收集器类似，区别在于它将元素分组到两个集合中，这两个集合要么匹配谓词，要么不匹配谓词。它将这两个集合包装到`Map`中，其中`true`关键字将引用匹配谓词的元素集合，而`false`关键字将引用不匹配谓词的元素：

    ```java
    Map<Boolean, List<Car>> partitioned = Stream.of(
            new Car("Toyota", 92),
            new Car("Kia", 104),
            new Car("Hyundai", 89),
            new Car("Toyota", 116),
            new Car("Mercedes", 209))
            .collect(Collectors.partitioningBy((car) -> { return car.          enginePower > 100; }));
    System.out.println(partitioned);
    ```

    你也可以选择将元素包装在哪种类型的集合中，就像`groupingBy`收集器一样：

    ```java
    Map<Boolean, Set<Car>> partitioned = Stream.of(
            new Car("Toyota", 92),
            new Car("Kia", 104),
            new Car("Hyundai", 89),
            new Car("Toyota", 116),
            new Car("Mercedes", 209))
            .collect(Collectors.partitioningBy((car) -> { return car.          enginePower > 100; }, Collectors.toSet()));
    System.out.println(partitioned);
    ```

+   `toMap`：这个收集器将允许你通过定义映射函数从你的流元素创建`map`，其中你提供一个要放入`map`中的键和值。通常，这只是一个元素的唯一标识符和元素本身。

    这可能有点棘手，因为如果你提供了一个重复的元素，那么你的管道将抛出一个`IllegalStateException`，因为`Map`不允许重复键：

    ```java
    Map<String, Integer> mapped = List.of("1", "2", "3", "4", "5")
            .stream()
            .collect(Collectors.toMap((s) -> {
                return s;
            }, (s) -> {
                return Integer.valueOf(s);
            }));
    System.out.println(mapped);
    ```

    这个简单的例子演示了如何将整数的字符串表示映射到实际的整数。如果你知道你可能会有重复的元素，那么你可以提供一个`merge`函数来解决这个问题：

    ```java
    Map<String, Integer> mapped = List.of("1", "2", "3", "4", "5", "1", "2")
            .stream()
            .collect(Collectors.toMap((s) -> {
                return s;
            }, (s) -> {
                return Integer.valueOf(s);
            }, (a, b) -> {
                return Integer.valueOf(b);
            }));
    System.out.println(mapped);
    ```

    你还可以通过在收集器的最后应用一个`factory`函数来生成自己的`Map`类型。在这里，我们告诉收集器为我们生成一个新的`TreeMap`：

    ```java
    TreeMap<String, Integer> mapped = List.of("1", "2", "3", "4", "5", "1", "2")
            .stream()
            .collect(Collectors.toMap((s) -> {
                return s;
            }, (s) -> {
                return Integer.valueOf(s);
            }, (a, b) -> {
                return Integer.valueOf(b);
            }, () -> {
                return new TreeMap<>();
            }));
    System.out.println(mapped);
    ```

+   `toUnmodifiableMap`：这本质上与`toMap`相同，具有相同的 API；然而，它返回不可变的`Map`版本。这在你知道你永远不会在`Map`中更改数据时非常完美。

+   `toConcurrentMap`：由于 `Map` 的实现方式，当在并行流中使用时可能会对性能造成一定风险。在这种情况下，建议使用 `toConcurrentMap()` 收集器。它具有与其他 `toMap` 函数类似的 API，不同之处在于它将返回 `ConcurrentMap` 实例而不是 `Map`。

+   从之前的收集器中的 `Car` 类，你可以生成所有汽车引擎的摘要，如下所示：

    ```java
    LongSummaryStatistics statistics = Stream.of(
            new Car("Volvo", 165),
            new Car("Volvo", 165),
            new Car("Honda", 104),
            new Car("Honda", 201)
    ).collect(Collectors.summarizingLong((e) -> {
        return e.enginePower;
    }));
    System.out.println(statistics);
    ```

## I/O 流

除了集合和其他原始数据类型之外，你还可以在管道中使用文件和 I/O 流作为数据源。这使得针对服务器编写任务变得更加描述性。

由于这些类型的资源通常需要正确关闭，你应该使用 try-with-resources 语句来确保在完成使用后资源被归还给系统。

考虑创建一个名为 `authors.csv` 的 CSV 文件，其内容如下：

```java
Andreas, 42, Sweden
David, 37, Sweden
Eric, 39, USA
```

你可以使用 try-with-resources 语句将此文件放入流中：

```java
String filePath = System.getProperty("user.dir") + File.separator +  "res/authors.csv";
try (Stream<String> authors = Files.lines(Paths.get(filePath))) {
    authors.forEach((author) -> {
        System.out.println(author);
    });
} catch (IOException e) {
    e.printStackTrace();
}
```

在 I/O 流中，你可以添加 `onClose` 处理程序来接收当流关闭时的通知。与其他流不同，当流的资源被关闭时，它将自动关闭。在这个例子中，这是由 try-with-resources 语句自动处理的。在下面的例子中，我们添加了一个 `onClose` 处理程序，当流关闭时将打印单词 `Closed`：

```java
try (Stream<String> authors = Files.lines(Paths.get(filePath))) {
    authors.onClose(() -> {
        System.out.println("Closed");
    }).forEach((author) -> {
        System.out.println(author);
    });
} catch (IOException e) {
    e.printStackTrace();
}
```

下面是使用 `InputStream` 编写的相同示例。请注意，代码现在更加冗长，有三个嵌套对象创建：

```java
try (Stream<String> authors = new BufferedReader(
        new InputStreamReader(new FileInputStream(filePath))).lines()
) {
    ...
} catch (FileNotFoundException e) {
    e.printStackTrace();
}
```

## 练习 2：将 CSV 转换为列表

一家基于标准 Java `List` 集合的在线杂货店已经实现了自己的数据库，并且还实现了一个备份系统，将数据库备份到 CSV 文件中。然而，他们还没有构建从 CSV 文件恢复数据库的方法。他们已经要求你构建一个系统，该系统能够读取这样的 CSV 文件，并将其内容扩展到列表中。

数据库备份 CSV 文件包含一种单一类型的对象：`ShoppingArticle`。每篇文章都有一个 `name`、一个 `price`、一个 `category`，最后还有一个 `unit`。名称、类别和单位应该是 `String` 类型，而价格是 `double` 类型：

1.  如果尚未打开，请打开 IDEA 中的 `Chapter15` 项目。

1.  创建一个新的 Java 类，使用 `File` | `New` | `Java`。

1.  将名称输入为 `Exercise2`，然后选择 `OK`。

    IntelliJ 将创建你的新类；它应该看起来像以下片段：

    ```java
    package com.packt.java.chapter15;
    public class Exercise2 {
    }
    ```

1.  向此类添加一个 `main` 方法。这是你将编写应用程序大部分代码的地方。你的类现在应该看起来像这样：

    ```java
    package com.packt.java.chapter15;
    public class Exercise2 {
        public static void main(String[] args) {
        }
    }
    ```

1.  创建一个 `ShoppingArticle` 内部类，并将其设置为静态，这样你就可以在主方法中轻松使用它。重写 `toString` 方法，以便稍后能够轻松地将文章打印到终端：

    ```java
        private static class ShoppingArticle {
            final String name;
            final String category;
            final double price;
            final String unit;
            private ShoppingArticle(String name, String category, double price,           String unit) {
                this.name = name;
                this.category = category;
                this.price = price;
                this.unit = unit;
            }
            @Override
            public String toString() {
                return name + " (" + category + ")";
            }
        }
    ```

1.  如果项目中尚未存在，请创建一个新的文件夹 `res`。然后，将其放置在根目录中，与 `src` 文件夹相邻。

1.  将`database.csv`文件从 GitHub 复制到你的项目中，并将其放置在`res`文件夹中。

1.  在你的`Exercise2.java`类中，添加一个生成`List<ShoppingArticle>`的函数。这将是我们将数据库加载到列表中的函数。由于该函数将加载文件，它需要抛出一个 I/O 异常（`IOException`）：

    ```java
    private static List<ShoppingArticle> loadDatabaseFile() throws IOException {
        return null;
    }
    ```

1.  从你的`main`方法中调用此函数：

    ```java
    public static void main(String[] args) {
        try {
            List<ShoppingArticle> database = loadDatabaseFile();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    ```

1.  首先使用 try-with-resources 块加载数据库文件。使用`Files.lines`加载`database.csv`文件中的所有行。它应该看起来像这样：

    ```java
    private static List<ShoppingArticle> loadDatabaseFile() throws IOException {
        try (Stream<String> stream = Files.lines(Path.of("res/database.csv"))) {
        }
        return null;
    }
    ```

1.  让我们查看流的状态，看看它现在的状态。中间操作只有在定义了终端操作时才会运行，所以添加一个`count()`在最后，只是为了强制执行整个管道：

    ```java
    private static List<ShoppingArticle> loadDatabaseFile() throws IOException {
        try (Stream<String> stream = Files.lines(Path.of("res/database.csv"))) {
            return stream.peek((line) -> {
                System.out.println(line);
            }).count();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
    ```

    这应该打印出文件中的每一行。注意，它还打印了标题行——当我们将其转换为`ShoppingArticles`时，我们并不关心标题行。

1.  由于我们并不真正对第一行感兴趣，所以在`count()`方法之前添加一个`skip`操作：

    ```java
    private static List<ShoppingArticle> loadDatabaseFile() throws IOException {
        try (Stream<String> stream = Files.lines(Path.of("res/database.csv"))) {
            return stream.peek((line) -> {
                System.out.println(line);
            }).skip(1).count();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
    ```

1.  现在数据库文件的每一行都已经加载为流中的元素，除了标题行。现在是时候从这些行中提取每一条数据了；适合这个操作的选项是`map`。使用`split()`函数将每一行分割成`String`数组：

    ```java
    private static List<ShoppingArticle> loadDatabaseFile() throws IOException {
        try (Stream<String> stream = Files.lines(Path.of("res/database.csv"))) {
            return stream.peek((line) -> {
                System.out.println(line);
            }).skip(1).map((line) -> {
                return line.split(",");
            }).count();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
    ```

1.  添加另一个`peek`操作来找出`map`操作如何改变了流；你的流类型现在应该是`Stream<String[]>`：

    ```java
    private static List<ShoppingArticle> loadDatabaseFile() throws IOException {
        try (Stream<String> stream = Files.lines(Path.of("res/database.csv"))) {
            return stream.peek((line) -> {
                System.out.println(line);
            }).skip(1).map((line) -> {
                return line.split(",");
            }).peek((arr) -> {
                System.out.println(Arrays.toString(arr));
            }).count();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
    ```

1.  添加另一个`map`操作，但这次是将流转换为`Stream<ShoppingArticle>`：

    ```java
    private static List<ShoppingArticle> loadDatabaseFile() throws IOException {
        try (Stream<String> stream = Files.lines(Path.of("res/database.csv"))) {
            return stream.peek((line) -> {
                System.out.println(line);
            }).skip(1).map((line) -> {
                return line.split(",");
            }).peek((arr) -> {
                System.out.println(Arrays.toString(arr));
            }).map((arr) -> {
                return new ShoppingArticle(arr[0], arr[1],               Double.valueOf(arr[2]), arr[3]);
            }).count();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
    ```

1.  现在你可以再次使用`peek`来确保文章被正确创建：

    ```java
    private static List<ShoppingArticle> loadDatabaseFile() throws IOException {
        try (Stream<String> stream = Files.lines(Path.of("res/database.csv"))) {
            return stream.peek((line) -> {
                System.out.println(line);
            }).skip(1).map((line) -> {
                return line.split(",");
            }).peek((arr) -> {
                System.out.println(Arrays.toString(arr));
            }).map((arr) -> {
                return new ShoppingArticle(arr[0], arr[1],               Double.valueOf(arr[2]), arr[3]);
            }).peek((art) -> {
                System.out.println(art);
            }).count();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
    ```

1.  将所有文章收集到一个列表中。使用不可修改的列表来保护数据库免受不想要的修改：

    ```java
    private static List<ShoppingArticle> loadDatabaseFile() throws IOException {
        try (Stream<String> stream = Files.lines(Path.of("res/database.csv"))) {
            return stream.peek((line) -> {
                System.out.println(line);
            }).skip(1).map((line) -> {
                return line.split(",");
            }).peek((arr) -> {
                System.out.println(Arrays.toString(arr));
            }).map((arr) -> {
                return new ShoppingArticle(arr[0], arr[1],               Double.valueOf(arr[2]), arr[3]);
            }).peek((art) -> {
                System.out.println(art);
            }).collect(Collectors.toUnmodifiableList());
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
    ```

这可能看起来有些冗长，因为一些操作可以一起应用来使其更短。然而，保持每个操作都非常小是有意义的，这样可以使整个逻辑非常透明。如果你在管道中发现了问题，你可以简单地移动管道中的一个操作，这样应该就能解决所有问题。

如果在一个操作中组合多个步骤，那么在管道中移动操作或完全替换它会更困难。

## 活动二：搜索特定内容

数据库加载完成后，应用一些搜索逻辑：

1.  编写一个函数，用于从一个`ShoppingArticles`列表中找出最便宜的水果。

1.  编写一个函数，用于从一个`ShoppingArticles`列表中找出最昂贵的蔬菜。

1.  编写一个函数，用于将所有水果收集到一个单独的列表中。

1.  编写一个函数，用于在数据库中找出五个最便宜的文章。

1.  编写一个函数，用于在数据库中找出五个最昂贵的文章。

    注意

    本活动的解决方案可以在第 564 页找到。

# 摘要

在编写程序时，描述性代码始终是一个值得追求的理想。代码越简单，就越容易向同事和其他感兴趣的人传达你的意图。

Java Streams API 允许您构建简单且高度描述性的函数。通常情况下，它们将是纯函数，因为 Streams API 使得避免操作状态变得非常容易。

在下一章中，我们将进一步探讨函数式编程主题，探索可用的不同函数式接口。
