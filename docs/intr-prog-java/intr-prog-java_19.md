# 第十九章：流和管道

在前一章描述和演示的 lambda 表达式以及功能接口中，为 Java 增加了强大的函数式编程能力。它允许将行为（函数）作为参数传递给针对数据处理性能进行优化的库。这样，应用程序员可以专注于开发系统的业务方面，将性能方面留给专家：库的作者。这样的一个库的例子是`java.util.stream`包，它将成为本章的重点。

我们将介绍数据流处理的概念，并解释流是什么，如何处理它们以及如何构建处理管道。我们还将展示如何轻松地组织并行流处理。

在本章中，将涵盖以下主题：

+   什么是流？

+   创建流

+   中间操作

+   终端操作

+   流管道

+   并行处理

+   练习 - 将所有流元素相乘

# 什么是流？

理解流的最好方法是将其与集合进行比较。后者是存储在内存中的数据结构。在将元素添加到集合之前，会计算每个集合元素。相反，流发出的元素存在于其他地方（源）并且根据需要进行计算。因此，集合可以是流的源。

在 Java 中，流是`java.util.stream`包的`Stream`、`IntStream`、`LongStream`或`DoubleStream`接口的对象。`Stream`接口中的所有方法也可以在`IntStream`、`LongStream`或`DoubleStream`专门的*数值*流接口中使用（相应类型更改）。一些数值流接口有一些额外的方法，例如`average()`和`sum()`，专门用于数值。

在本章中，我们将主要讨论`Stream`接口及其方法。但是，所介绍的一切同样适用于数值流接口。在本章末尾，我们还将回顾一些在数值流接口中可用但在`Stream`接口中不可用的方法。

流代表一些数据源 - 例如集合、数组或文件 - 并且按顺序生成（产生、发出）一些值（与流相同类型的流元素），一旦先前发出的元素被处理。

`java.util.stream`包允许以声明方式呈现可以应用于发出元素的过程（函数），也可以并行进行。如今，随着机器学习对大规模数据处理的要求以及对操作的微调变得普遍，这一特性加强了 Java 在少数现代编程语言中的地位。

# 流操作

`Stream`接口的许多方法（具有函数接口类型作为参数的方法）被称为操作，因为它们不是作为传统方法实现的。它们的功能作为函数传递到方法中。方法本身只是调用分配为方法参数类型的函数接口的方法的外壳。

例如，让我们看一下`Stream<T> filter (Predicate<T> predicate)`方法。它的实现基于对`Predicate<T>`函数的`boolean test(T)`方法的调用。因此，程序员更喜欢说，“我们应用`filter`操作，允许一些流元素通过，跳过其他元素”，而不是说“我们使用`Stream`对象的`filter()`方法来选择一些流元素并跳过其他元素”。这听起来类似于说“我们应用加法操作”。它描述了动作（操作）的性质，而不是特定的算法，直到方法接收到特定函数为止。

因此，`Stream`接口中有三组方法：

+   创建`Stream`对象的静态工厂方法。

+   中间操作是返回`Stream`对象的实例方法。

+   终端操作是返回`Stream`之外的某种类型的实例方法。

流处理通常以流畅（点连接）的方式组织（参见*流管道*部分）。`Stream`工厂方法或另一个流源开始这样的管道，终端操作产生管道结果或副作用，并结束管道（因此得名）。中间操作可以放置在原始`Stream`对象和终端操作之间。它处理流元素（或在某些情况下不处理），并返回修改的（或未修改的）`Stream`对象，以便应用下一个中间或终端操作。

中间操作的示例如下：

+   `filter()`: 这将选择与条件匹配的元素。

+   `map()`: 这将根据函数转换元素。

+   `distinct()`: 这将删除重复项。

+   `limit()`: 这将限制流的元素数量。

+   `sorted()`: 这将把未排序的流转换为排序的流。

还有一些其他方法，我们将在*中间操作*部分讨论。

流元素的处理实际上只有在开始执行终端操作时才开始。然后，所有中间操作（如果存在）开始处理。流在终端操作完成执行后关闭（并且无法重新打开）。终端操作的示例包括`forEach()`、`findFirst()`、`reduce()`、`collect()`、`sum()`、`max()`和`Stream`接口的其他不返回`Stream`的方法。我们将在*终端操作*部分讨论它们。

所有的 Stream 方法都支持并行处理，这在多核计算机上处理大量数据时特别有帮助。必须确保处理管道不使用可以在不同处理环境中变化的上下文状态。我们将在*并行处理*部分讨论这一点。

# 创建流

有许多创建流的方法——`Stream`类型的对象或任何数字接口。我们已经按照创建 Stream 对象的方法所属的类和接口对它们进行了分组。我们之所以这样做是为了方便读者，提供更好的概览，这样读者在需要时更容易找到它们。

# 流接口

这组`Stream`工厂由属于`Stream`接口的静态方法组成。

# empty(), of(T t), ofNullable(T t)

以下三种方法创建空的或单个元素的`Stream`对象：

+   `Stream<T> empty()`: 创建一个空的顺序`Stream`对象。

+   `Stream<T> of(T t)`: 创建一个顺序的单个元素`Stream`对象。

+   `Stream<T> ofNullable(T t)`: 如果`t`参数非空，则创建一个包含单个元素的顺序`Stream`对象；否则，创建一个空的 Stream。

以下代码演示了前面方法的用法：

```java

Stream.empty().forEach(System.out::println);    //不打印任何内容

Stream.of(1).forEach(System.out::println);      //打印：1

List<String> list = List.of("1 ", "2");

//printList1(null);                             //NullPointerException

printList1(list);                               //打印：1 2

void printList1(List<String> list){

list.stream().forEach(System.out::print);;

}

```

注意，当列表不为空时，第一次调用`printList1()`方法会生成`NullPointerException`并打印`1 2`。为了避免异常，我们可以将`printList1()`方法实现如下：

```java

void printList1(List<String> list){

(list == null ? Stream.empty() : list.stream())

.forEach(System.out::print);

}

```

相反，我们使用了`ofNullable(T t)`方法，如下面的`printList2()`方法的实现所示：

```java

printList2(null);                                //prints nothing

printList2(list);                                //prints: [1 , 2]

void printList2(List<String> list){

Stream.ofNullable(list).forEach(System.out::print);

}

```

这就是激发`ofNullable(T t)`方法创建的用例。但是您可能已经注意到，`ofNullable()`创建的流将列表作为一个对象发出：它被打印为`[1 , 2]`。

在这种情况下处理列表的每个元素，我们需要添加一个中间的`Stream`操作`flatMap()`，将每个元素转换为`Stream`对象：

```java

Stream.ofNullable(list).flatMap(e -> e.stream())

.forEach(System.out::print);      //prints: 1 2

```

我们将在*Intermediate operations*部分进一步讨论`flatMap()`方法。

在前面的代码中传递给`flatMap()`操作的函数也可以表示为方法引用：

```java

Stream.ofNullable(list).flatMap(Collection::stream)

.forEach(System.out::print);      //prints: 1 2

```

# iterate(Object, UnaryOperator)

`Stream`接口的两个静态方法允许我们使用类似传统`for`循环的迭代过程生成值流：

+   `Stream<T> iterate(T seed, UnaryOperator<T> func)`: 根据第一个`seed`参数的迭代应用第二个参数（`func`函数）创建一个**无限**顺序`Stream`对象，生成`seed`、`f(seed)`和`f(f(seed))`值的流。

+   `Stream<T> iterate(T seed, Predicate<T> hasNext, UnaryOperator<T> next)`: 根据第三个参数（`next`函数）对第一个`seed`参数的迭代应用，生成`seed`、`f(seed)`和`f(f(seed))`值的有限顺序`Stream`对象，只要第三个参数（`hasNext`函数）返回`true`。

以下代码演示了这些方法的用法：

```java

Stream.iterate(1, i -> ++i).limit(9)

.forEach(System.out::print);        //prints: 123456789

Stream.iterate(1, i -> i < 10, i -> ++i)

.forEach(System.out::print);        //prints: 123456789

```

请注意，我们被迫在第一个管道中添加一个`limit()`中间操作，以避免生成无限数量的值。

# concat(Stream a, Stream b)

`Stream<T>` concatenate (`Stream<> a`, `Stream<T> b`) `Stream` 接口的静态方法基于传递的两个`Stream`对象`a`和`b`创建一个值流。新创建的流由第一个参数`a`的所有元素组成，后跟第二个参数`b`的所有元素。以下代码演示了`Stream`对象创建的这种方法：

```java

Stream<Integer> stream1 = List.of(1, 2).stream();

Stream<Integer> stream2 = List.of(2, 3).stream();

Stream.concat(stream1, stream2)

.forEach(System.out::print);        //prints: 1223

```

请注意，原始流中存在`2`元素，并且因此在生成的流中出现两次。

# generate(Supplier)

`Stream<T> generate(Supplier<T> supplier)` `Stream` 接口的静态方法创建一个无限流，其中每个元素由提供的`Supplier<T>`函数生成。以下是两个示例：

```java

Stream.generate(() -> 1).limit(5)

.forEach(System.out::print);       //prints: 11111

Stream.generate(() -> new Random().nextDouble()).limit(5)

.forEach(System.out::println);     //prints: 0.38575117472619247

//        0.5055765386778835

//        0.6528038976983277

//        0.4422354489467244

//        0.06770955839148762

```

由于流是无限的，我们已经添加了`limit()`操作。

# of(T... values)

`Stream<T> of(T... values)` 方法接受可变参数或值数组，并使用提供的值作为流元素创建`Stream`对象：

```java

Stream.of("1 ", 2).forEach(System.out::print);      //prints: 1 2

//Stream<String> stringStream = Stream.of("1 ", 2); //compile error

String[] strings = {"1 ", "2"};

Stream.of(strings).forEach(System.out::print);      //输出：1 2

```

请注意，在上述代码的第一行中，如果在`Stream`引用声明的泛型中没有指定类型，则`Stream`对象将接受不同类型的元素。在下一行中，泛型将`Stream`对象的类型定义为`String`，相同的元素类型混合会生成编译错误。泛型绝对有助于程序员避免许多错误，并且应该在可能的地方使用。

`of(T... values)`方法也可用于连接多个流。例如，假设我们有以下四个流，并且我们想要将它们连接成一个：

```java

Stream<Integer> stream1 = Stream.of(1, 2);

Stream<Integer> stream2 = Stream.of(2, 3);

Stream<Integer> stream3 = Stream.of(3, 4);

Stream<Integer> stream4 = Stream.of(4, 5);

```

我们期望新流发出值`1`、`2`、`2`、`3`、`3`、`4`、`4`和`5`。首先，我们尝试以下代码：

```java

Stream.of(stream1, stream2, stream3, stream4)

.forEach(System.out::print);

//输出：java.util.stream.ReferencePipeline$Head@58ceff1j

```

上述代码并没有达到我们的期望。它将每个流都视为`java.util.stream.ReferencePipeline`内部类的对象，该内部类用于`Stream`接口实现。因此，我们添加了一个`flatMap()`操作，将每个流元素转换为流（我们将在*中间操作*部分中描述它）：

```java

Stream.of(stream1, stream2, stream3, stream4)

.flatMap(e -> e).forEach(System.out::print);   //输出：12233445

```java

我们将作为参数传递给`flatMap()`的函数（`e -> e`）可能看起来好像什么都没做，但这是因为流的每个元素已经是一个流，所以我们不需要对其进行转换。通过将元素作为`flatMap()`操作的结果返回，我们已经告诉管道将其视为`Stream`对象。已经完成了这一点，并且显示了预期的结果。

# Stream.Builder 接口

`Stream.Builder<T> builder()`静态方法返回一个内部（位于`Stream`接口中的）`Builder`接口，可用于构造`Stream`对象。`Builder`接口扩展了`Consumer`接口，并具有以下方法：

+   `void accept(T t)`: 将元素添加到流中（此方法来自`Consumer`接口）。

+   `default Stream.Builder<T> add(T t)`: 调用`accept(T)`方法并返回`this`，从而允许以流畅的点连接样式链接`add(T)`方法。

+   `Stream<T> build()`: 将此构建器从构造状态转换为构建状态。调用此方法后，无法向流中添加新元素。

使用`add()`方法很简单：

```java

Stream.<String>builder().add("cat").add(" dog").add(" bear")

.build().forEach(System.out::print);  //输出：cat dog bear

```

只需注意我们在`builder()`方法前面添加的`<String>`泛型。这样，我们告诉构建器我们正在创建的流将具有`String`类型的元素。否则，它将将它们添加为`Object`类型。

当构建器作为`Consumer`对象传递时，或者不需要链接添加元素的方法时，使用`accept()`方法。例如，以下是构建器作为`Consumer`对象传递的方式：

```java

Stream.Builder<String> builder = Stream.builder();

List.of("1", "2", "3").stream().forEach(builder);

builder.build().forEach(System.out::print);        //输出：123

```

还有一些情况不需要在添加流元素时链接方法。以下方法接收`String`对象的列表，并将其中一些对象（包含字符`a`的对象）添加到流中：

```java

Stream<String> buildStream(List<String> values){

Stream.Builder<String> builder = Stream.builder();

for(String s: values){

如果(s.contains("a")){

builder.accept(s);

}

}

return builder.build();

}

```

请注意，出于同样的原因，我们为`Stream.Builder`接口添加了`<String>`泛型，告诉构建器我们添加的元素应该被视为`String`类型。

当调用前面的方法时，它会产生预期的结果：

```java

List<String> list = List.of("cat", " dog", " bear");

buildStream(list).forEach(System.out::print);        //输出：cat bear

```

# 其他类和接口

在 Java 8 中，`java.util.Collection`接口添加了两个默认方法：

+   `Stream<E> stream()`: 返回此集合的元素流。

+   `Stream<E> parallelStream()`: 返回（可能）此集合元素的并行流。这里的可能是因为 JVM 会尝试将流分成几个块并并行处理它们（如果有几个 CPU）或虚拟并行处理（使用 CPU 的时间共享）。这并非总是可能的；这在一定程度上取决于所请求处理的性质。

这意味着扩展此接口的所有集合接口，包括`Set`和`List`，都有这些方法。这是一个例子：

```java

List<Integer> list = List.of(1, 2, 3, 4, 5);

list.stream().forEach(System.out::print);    //输出：12345

```

我们将在*并行处理*部分进一步讨论并行流。

`java.util.Arrays`类还添加了八个静态重载的`stream()`方法。它们从相应的数组或其子集创建不同类型的流：

+   `Stream<T> stream(T[] array)`: 从提供的数组创建`Stream`。

+   `IntStream stream(int[] array)`: 从提供的数组创建`IntStream`。

+   `LongStream stream(long[] array)`: 从提供的数组创建`LongStream`。

+   `DoubleStream stream(double[] array)`: 从提供的数组创建`DoubleStream`。

+   `Stream<T> stream(T[] array, int startInclusive, int endExclusive)`: 从提供的数组的指定范围创建`Stream`。

+   `IntStream stream(int[] array, int startInclusive, int endExclusive)`: 从提供的数组的指定范围创建`IntStream`。

+   `LongStream stream(long[] array, int startInclusive, int endExclusive)`: 从提供的数组的指定范围创建`LongStream`。

+   `DoubleStream stream(double[] array, int startInclusive, int endExclusive)`: 从提供的数组的指定范围创建`DoubleStream`。

这是一个从数组的子集创建流的示例：

```java

int[] arr = {1, 2, 3, 4, 5};

Arrays.stream(arr, 2, 4).forEach(System.out::print);    //输出：34

```

请注意，我们使用了`Stream<T> stream(T[] array, int startInclusive, int endExclusive)`方法，这意味着我们创建了`Stream`而不是`IntStream`，尽管创建的流中的所有元素都是整数，就像`IntStream`一样。不同之处在于，`IntStream`提供了一些数字特定的操作，而`Stream`中没有（请参阅*数字流接口*部分）。

`java.util.Random`类允许我们创建伪随机值的数字流：

+   `IntStream ints()` 和 `LongStream longs()`: 创建相应类型的无限伪随机值流。

+   `DoubleStream doubles()`: 创建一个无限流的伪随机双精度值，每个值都介于零（包括）和一（不包括）之间。

+   `IntStream ints(long streamSize)` 和 `LongStream longs(long streamSize)`: 创建指定数量的相应类型的伪随机值流。

+   `DoubleStream doubles(long streamSize)`: 创建指定数量的伪随机双精度值流，每个值都介于零（包括）和一（不包括）之间。

+   `IntStream ints(int randomNumberOrigin, int randomNumberBound)`, `LongStream longs(long randomNumberOrigin, long randomNumberBound)`, 和 `DoubleStream doubles(long streamSize, double randomNumberOrigin, double randomNumberBound)`: 创建一个无限流，包含对应类型的伪随机值，每个值大于或等于第一个参数，小于第二个参数。

以下是前述方法的示例之一：

```java

new Random().ints(5, 8)

.limit(5)

.forEach(System.out::print);    //打印：56757

```

`java.nio.File`类有六个静态方法，用于创建行和路径流：

+   `Stream<String> lines(Path path)`: 从提供的路径指定的文件创建一行流。

+   `Stream<String> lines(Path path, Charset cs)`: 从提供的路径指定的文件创建一行流。使用提供的字符集将文件的字节解码为字符。

+   `Stream<Path> list(Path dir)`: 创建指定目录中的条目流。

+   `Stream<Path> walk(Path start, FileVisitOption... options)`: 创建以给定起始文件为根的文件树条目流。

+   `Stream<Path> walk(Path start, int maxDepth, FileVisitOption... options)`: 创建以给定起始文件为根的文件树条目流，到指定深度。

+   `Stream<Path> find(Path start, int maxDepth, BiPredicate<Path, BasicFileAttributes> matcher, FileVisitOption... options)`: 创建以给定起始文件为根的文件树条目流，到指定深度匹配提供的谓词。

其他创建流的类和方法包括：

+   `IntStream stream()` of the `java.util.BitSet` class: 创建一个索引流，其中`BitSet`包含设置状态的位。

+   `Stream<String> lines()` of the `java.io.BufferedReader` class: 创建从`BufferedReader`对象读取的行流，通常来自文件。

+   `Stream<JarEntry> stream()` of the `java.util.jar.JarFile` class: 创建 ZIP 文件条目的流。

+   `IntStream chars()` of the `java.lang.CharSequence` interface: 从此序列创建`int`类型的流，零扩展`char`值。

+   `IntStream codePoints()` of the `java.lang.CharSequence` interface: 从此序列创建代码点值的流。

+   `Stream<String> splitAsStream(CharSequence input)` of the `java.util.regex.Pattern` class: 创建一个围绕此模式匹配的提供序列的流。

还有`java.util.stream.StreamSupport`类，其中包含库开发人员的静态低级实用方法。这超出了本书的范围。

# 中间操作

我们已经看到了如何创建代表源并发出元素的`Stream`对象。正如我们已经提到的，`Stream`接口提供的操作（方法）可以分为三组：

+   基于源创建`Stream`对象的方法。

+   接受函数并生成发出相同或修改的值的`Stream`对象的中间操作。

+   终端操作完成流处理，关闭它并生成结果。

在本节中，我们将回顾中间操作，这些操作可以根据其功能进行分组。

# 过滤

此组包括删除重复项、跳过一些元素和限制处理元素数量的操作，仅选择所需的元素：

+   `Stream<T> distinct()`: 使用`Object.equals(Object)`方法比较流元素，并跳过重复项。

+   `Stream<T> skip(long n)`: 忽略前面提供的数量的流元素。

+   `Stream<T> limit(long maxSize)`: 仅允许处理提供的流元素数量。

+   `Stream<T> filter(Predicate<T> predicate)`: 仅允许通过提供的`Predicate`函数处理的结果为`true`的元素。

+   默认`Stream<T> dropWhile(Predicate<T> predicate)`: 跳过流的第一个元素，该元素在通过提供的`Predicate`函数处理时结果为`true`。

+   默认`Stream<T> takeWhile(Predicate<T> predicate)`: 仅允许流的第一个元素在通过提供的`Predicate`函数处理时结果为`true`。

以下代码演示了前面的操作是如何工作的：

```java

Stream.of("3", "2", "3", "4", "2").distinct()

.forEach(System.out::print);  //prints: 324

List<String> list = List.of("1", "2", "3", "4", "5");

list.stream().skip(3).forEach(System.out::print);         //prints: 45

list.stream().limit(3).forEach(System.out::print);        //prints: 123

list.stream().filter(s -> Objects.equals(s, "2"))

.forEach(System.out::print);  //prints: 2

list.stream().dropWhile(s -> Integer.valueOf(s) < 3)

.forEach(System.out::print);  //prints: 345

list.stream().takeWhile(s -> Integer.valueOf(s) < 3)

.forEach(System.out::print);  //prints: 12

```

请注意，我们能够重用`List<String>`源对象，但无法重用`Stream`对象。一旦关闭，就无法重新打开。

# Mapping

这组包括可能是最重要的中间操作。它们是唯一修改流元素的中间操作。它们*map*（转换）原始流元素值为新值：

+   `Stream<R> map(Function<T, R> mapper)`: 将提供的函数应用于此流的`T`类型的每个元素，并生成`R`类型的新元素值。

+   `IntStream mapToInt(ToIntFunction<T> mapper)`: 将此流转换为`Integer`值的`IntStream`。

+   `LongStream mapToLong(ToLongFunction<T> mapper)`: 将此流转换为`Long`值的`LongStream`。

+   `DoubleStream mapToDouble(ToDoubleFunction<T> mapper)`: 将此流转换为`Double`值的`DoubleStream`。

+   `Stream<R> flatMap(Function<T, Stream<R>> mapper)`: 将提供的函数应用于此流的`T`类型的每个元素，并生成一个发出`R`类型元素的`Stream<R>`对象。

+   `IntStream flatMapToInt(Function<T, IntStream> mapper)`: 使用提供的函数将`T`类型的每个元素转换为`Integer`值流。

+   `LongStream flatMapToLong(Function<T, LongStream> mapper)`: 使用提供的函数将`T`类型的每个元素转换为`Long`值流。

+   `DoubleStream flatMapToDouble(Function<T, DoubleStream> mapper)`: 使用提供的函数将`T`类型的每个元素转换为`Double`值流。

以下是这些操作的用法示例：

```java

List<String> list = List.of("1", "2", "3", "4", "5");

list.stream().map(s -> s + s)

.forEach(System.out::print);        //prints: 1122334455

list.stream().mapToInt(Integer::valueOf)

.forEach(System.out::print);             //prints: 12345

list.stream().mapToLong(Long::valueOf)

.forEach(System.out::print);             //prints: 12345

list.stream().mapToDouble(Double::valueOf)

.mapToObj(Double::toString)

.map(s -> s + " ")

.forEach(System.out::print);//prints: 1.0 2.0 3.0 4.0 5.0

list.stream().mapToInt(Integer::valueOf)

.flatMap(n -> IntStream.iterate(1, i -> i < n, i -> ++i))

.forEach(System.out::print);        //prints: 1121231234

list.stream().map(Integer::valueOf)

.flatMapToInt(n ->

IntStream.iterate(1, i -> i < n, i -> ++i))

.forEach(System.out::print);        //prints: 1121231234

list.stream().map(Integer::valueOf)

.flatMapToLong(n ->

LongStream.iterate(1, i -> i < n, i -> ++i))

.forEach(System.out::print);        //prints: 1121231234;

list.stream().map(Integer::valueOf)

.flatMapToDouble(n ->

DoubleStream.iterate(1, i -> i < n, i -> ++i))

.mapToObj(Double::toString)

.map(s -> s + " ")

.forEach(System.out::print);

//prints: 1.0 1.0 2.0 1.0 2.0 3.0 1.0 2.0 3.0 4.0

```

在前面的示例中，对于`Double`值，我们将数值转换为`String`，并添加空格，因此结果将以空格分隔的形式打印出来。这些示例非常简单——只是进行最小处理的转换。但是在现实生活中，每个`map`或`flatMap`操作都可以接受一个（任何复杂程度的函数）来执行真正有用的操作。

# 排序

以下两个中间操作对流元素进行排序。自然地，这样的操作直到所有元素都被发射完毕才能完成，因此会产生大量的开销，降低性能，并且必须用于小型流：

+   `Stream<T> sorted()`: 按照它们的`Comparable`接口实现的自然顺序对流元素进行排序。

+   `Stream<T> sorted(Comparator<T> comparator)`: 按照提供的`Comparator<T>`对象的顺序对流元素进行排序。

以下是演示代码：

```java

List<String> list = List.of("2", "1", "5", "4", "3");

list.stream().sorted().forEach(System.out::print);  //prints: 12345

list.stream().sorted(Comparator.reverseOrder())

.forEach(System.out::print);           //prints: 54321

```

# Peeking

`Stream<T> peek(Consumer<T> action)`中间操作将提供的`Consumer`函数应用于每个流元素，并且不更改此`Stream`（返回它接收到的相同元素值），因为`Consumer`函数返回`void`，并且不能影响值。此操作用于调试。

以下代码显示了它的工作原理：

```java

List<String> list = List.of("1", "2", "3", "4", "5");

list.stream().peek(s-> {

if("3".equals(s)){

System.out.print(3);

}

}).forEach(System.out::print);  //prints: 123345

```

# 终端操作

终端操作是流管道中最重要的操作。不需要任何其他操作就可以轻松完成所有操作。我们已经使用了`forEach(Consumer<T>)`终端操作来打印每个元素。它不返回值；因此，它用于其副作用。但是`Stream`接口还有许多更强大的终端操作，它们会返回值。其中最重要的是`collect()`操作，它有两种形式，`R collect(Collector<T, A, R> collector)`和`R collect(Supplier<R> supplier, BiConsumer<R, T> accumulator, BiConsumer<R, R> combiner)`。这些允许我们组合几乎可以应用于流的任何过程。经典示例如下：

```java

List<String> asList = stringStream.collect(ArrayList::new,

ArrayList::add,

ArrayList::addAll);

```

如您所见，它是为并行处理而实现的。它使用第一个函数基于流元素生成值，使用第二个函数累积结果，然后结合处理流的所有线程累积的结果。

然而，只有一个这样的通用终端操作会迫使程序员重复编写相同的函数。这就是为什么 API 作者添加了`Collectors`类，它可以生成许多专门的`Collector`对象，而无需为每个`collect()`操作创建三个函数。除此之外，API 作者还添加了更多专门的终端操作，这些操作更简单，更容易使用`Stream`接口。

在本节中，我们将回顾`Stream`接口的所有终端操作，并在`Collecting`子部分中查看`Collectors`类生成的大量`Collector`对象的种类。

我们将从最简单的终端操作开始，它允许逐个处理流的每个元素。

# 处理每个元素

这个组中有两个终端操作：

+   `void forEach(Consumer<T> action)`: 对流的每个元素应用提供的操作（处理）。

+   `void forEachOrdered(Consumer<T> action)`: 对流的每个元素应用提供的操作（处理），其顺序由源定义，无论流是顺序的还是并行的。

如果您的应用程序对需要处理的元素的顺序很重要，并且必须按照源中值的排列顺序进行处理，那么使用第二种方法是很重要的，特别是如果您可以预见到您的代码将在具有多个 CPU 的计算机上执行。否则，使用第一种方法，就像我们在所有的例子中所做的那样。

这种操作被用于*任何类型*的流处理是很常见的，特别是当代码是由经验不足的程序员编写时。对于下面的例子，我们创建了`Person`类：

```java

class Person {

private int age;

private String name;

public Person(int age, String name) {

this.name = name;

this.age = age;

}

public String getName() { return this.name; }

public int getAge() {return this.age; }

@Override

public String toString() {

return "Person{" + "name='" + this.name + "'" +

", age=" + age + "}";

}

}

```

我们将在终端操作的讨论中使用这个类。在这个例子中，我们将从文件中读取逗号分隔的值（年龄和姓名），并创建`Person`对象。我们已经将以下`persons.csv`文件（**逗号分隔值（CSV）**）放在`resources`文件夹中：

```java

23 , Ji m

2 5 , Bob

15 , Jill

17 , Bi ll

```

请注意我们在值的外部和内部添加的空格。我们这样做是为了借此机会向您展示一些简单但非常有用的处理现实数据的技巧。以下是一个经验不足的程序员可能编写的代码，用于读取此文件并创建`Person`对象列表：

```java

List<Person> persons = new ArrayList<>();

Path path = Paths.get("src/main/resources/persons.csv");

try (Stream<String> lines = Files.newBufferedReader(path).lines()) {

lines.forEach(s -> {

String[] arr = s.split(",");

int age = Integer.valueOf(StringUtils.remove(arr[0], ' '));

persons.add(new Person(age, StringUtils.remove(arr[1], ' ')));

});

} catch (IOException ex) {

ex.printStackTrace();

}

persons.stream().forEach(System.out::println);

//prints: Person{name='Jim', age=23}

//        Person{name='Bob', age=25}

//        Person{name='Jill', age=15}

//        Person{name='Bill', age=17}

```

您可以看到我们使用了`String`方法`split()`，通过逗号分隔每一行的值，并且我们使用了`org.apache.commons.lang3.StringUtils`类来移除每个值中的空格。前面的代码还提供了`try-with-resources`结构的真实示例，用于自动关闭`BufferedReader`对象。

尽管这段代码在小例子和单核计算机上运行良好，但在长流和并行处理中可能会产生意外的结果。也就是说，lambda 表达式要求所有变量都是 final 的，或者有效地是 final 的，因为相同的函数可以在不同的上下文中执行。

相比之下，这是前面代码的正确实现：

```java

List<Person> persons = new ArrayList<>();

Path path = Paths.get("src/main/resources/persons.csv");

try (Stream<String> lines = Files.newBufferedReader(path).lines()) {

persons = lines.map(s -> s.split(","))

.map(arr -> {

int age = Integer.valueOf(StringUtils.remove(arr[0], ' '));

return new Person(age, StringUtils.remove(arr[1], ' '));

}).collect(Collectors.toList());

} catch (IOException ex) {

ex.printStackTrace();

}

persons.stream().forEach(System.out::println);

```

为了提高可读性，可以创建一个执行映射工作的方法：

```java

public List<Person> createPersons() {

List<Person> persons = new ArrayList<>();

Path path = Paths.get("src/main/resources/persons.csv");

try (Stream<String> lines = Files.newBufferedReader(path).lines()) {

persons = lines.map(s -> s.split(","))

.map(this::createPerson)

.collect(Collectors.toList());

} catch (IOException ex) {

ex.printStackTrace();

}

return persons;

}

private Person createPerson(String[] arr){

int age = Integer.valueOf(StringUtils.remove(arr[0], ' '));

return new Person(age, StringUtils.remove(arr[1], ' '));

}

```

正如你所看到的，我们使用了`collect()`操作和`Collectors.toList()`方法创建的`Collector`函数。我们将在*Collect*子部分中看到更多由`Collectors`类创建的`Collector`函数。

# 计算所有元素

`long count()`终端操作的`Stream`接口看起来很简单，也很温和。它返回这个流中的元素数量。习惯于使用集合和数组的人可能会毫不犹豫地使用`count()`操作。下面是一个例子，证明它可以正常工作：

```java

long count = Stream.of("1", "2", "3", "4", "5")

.peek(System.out::print)

.count();

System.out.print(count);                 //输出：5

```

正如你所看到的，实现计数方法的代码能够确定流的大小，而不需要执行整个管道。元素的值并没有被`peek()`操作打印出来，这证明元素并没有被发出。但是并不总是能够在源头确定流的大小。此外，流可能是无限的。因此，必须谨慎使用`count()`。

既然我们正在讨论计算元素的话，我们想展示另一种可能的确定流大小的方法，使用`collect()`操作：

```java

int count = Stream.of("1", "2", "3", "4", "5")

.peek(System.out::print)         //输出：12345

.collect(Collectors.counting());

System.out.println(count);                //输出：5

```

你可以看到`collect()`操作的实现甚至没有尝试在源头计算流的大小（因为，正如你所看到的，管道已经完全执行，并且每个元素都被`peek()`操作打印出来）。这是因为`collect()`操作不像`count()`操作那样专门化。它只是将传入的收集器应用于流，而收集器则计算由`collect()`操作提供给它的元素。你可以将这看作是官僚近视的一个例子：每个操作符都按预期工作，但整体性能仍然有所欠缺。

# 匹配所有、任意或没有

有三个（看起来非常相似的）终端操作，允许我们评估流中的所有、任意或没有元素是否具有特定值：

+   `boolean allMatch(Predicate<T> predicate)`: 当流中的每个元素返回`true`时，作为提供的`Predicate<T>`函数的参数时返回`true`。

+   `boolean anyMatch(Predicate<T> predicate)`: 当流中的一个元素返回`true`时，作为提供的`Predicate<T>`函数的参数时返回`true`。

+   `boolean noneMatch(Predicate<T> predicate)`: 当流中没有元素返回`true`时，作为提供的`Predicate<T>`函数的参数时返回`true`。

以下是它们的使用示例：

```java

List<String> list = List.of("1", "2", "3", "4", "5");

boolean found = list.stream()

.peek(System.out::print)          //输出：123

.anyMatch(e -> "3".equals(e));

System.out.print(found);                  //输出：true   <= 第 5 行

found = list.stream()

.peek(System.out::print)          //输出：12345

.anyMatch(e -> "0".equals(e));

System.out.print(found);                  //输出：false

boolean noneMatches = list.stream()

.peek(System.out::print)          //输出：123

.noneMatch(e -> "3".equals(e));

System.out.print(noneMatches);            //输出：false

noneMatches = list.stream()

.peek(System.out::print)          //输出：12345

.noneMatch(e -> "0".equals(e));

System.out.print(noneMatches);            //输出：true  <= 第 17 行

boolean allMatch = list.stream()

.peek(System.out::print)          //输出：1

.allMatch(e -> "3".equals(e));

System.out.print(allMatch);               //prints: false

```

让我们更仔细地看一下前面示例的结果。这些操作中的每一个都触发了流管道的执行，每次至少处理流的一个元素。但是看看`anyMatch()`和`noneMatch()`操作。第 5 行说明至少有一个元素等于`3`。结果是在*处理了前三个元素之后*返回的。第 17 行说明在*处理了流的所有元素*之后，没有元素等于`0`。

问题是，当您想要知道流*不包含*`v`值时，这两个操作中的哪一个应该使用？如果使用`noneMatch()`，*所有元素都将被处理*。但是如果使用`anyMatch()`，只有在流中没有`v`*值*时，所有元素才会被处理。似乎`noneMatch()`操作是无用的，因为当`anyMatch()`返回`true`时，它的含义与`noneMatch()`返回`false`相同，而`anyMatch()`操作只需处理更少的元素即可实现。随着流大小的增长和存在`v`值的机会增加，这种差异变得更加重要。似乎`noneMatch()`操作的唯一原因是代码可读性，当处理时间不重要时，因为流大小很小。

`allMatch()`操作没有替代方案，与`anyMatch()`类似，当遇到第一个不匹配的元素时返回，或者需要处理所有流元素。

# 查找任何或第一个

以下终端操作允许我们找到流的任何元素或第一个元素：

+   `Optional<T> findAny()`: 返回流的任何元素的值的`Optional`，如果流为空，则返回一个空的`Optional`。

+   `Optional<T> findFirst()`: 返回流的第一个元素的值的`Optional`，如果流为空，则返回一个空的`Optional`。

以下示例说明了这些操作：

```java

List<String> list = List.of("1", "2", "3", "4", "5");

Optional<String> result = list.stream().findAny();

System.out.println(result.isPresent());    //prints: true

System.out.println(result.get());          //prints: 1

result = list.stream().filter(e -> "42".equals(e)).findAny();

System.out.println(result.isPresent());    //prints: true

//System.out.println(result.get());        //NoSuchElementException

result = list.stream().findFirst();

System.out.println(result.isPresent());    //prints: true

System.out.println(result.get());          //prints: 1

```

如您所见，它们返回相同的结果。这是因为我们在单个线程中执行管道。这两个操作之间的差异在并行处理中更加显著。当流被分成几个部分进行并行处理时，如果流不为空，`findFirst()`操作总是返回流的第一个元素，而`findAny()`操作只在一个处理线程中返回第一个元素。

让我们更详细地讨论`java.util.Optional`类。

# Optional 类

`java.util.Optional`对象用于避免返回`null`，因为它可能会导致`NullPointerException`。相反，`Optional`对象提供了可以用来检查值是否存在并在没有值的情况下替换它的方法。例如：

```java

List<String> list = List.of("1", "2", "3", "4", "5");

String result = list.stream().filter(e -> "42".equals(e))

.findAny().or(() -> Optional.of("Not found")).get();

System.out.println(result);                       //prints: Not found

result = list.stream().filter(e -> "42".equals(e))

.findAny().orElse("Not found");

System.out.println(result);                        //prints: Not found

Supplier<String> trySomethingElse = () -> {

//尝试其他操作的代码

return "43";

};

result = list.stream().filter(e -> "42".equals(e))

.findAny().orElseGet(trySomethingElse);

System.out.println(result);                          //prints: 43

list.stream().filter(e -> "42".equals(e))

.findAny().ifPresentOrElse(System.out::println,

() -> System.out.println("Not found"));  //prints: Not found

```

As you can see, if the `Optional` object is empty, then:

+   The `or()` method of the `Optional` class allows for returning an alternative `Optional` object (with a value).

+   The `orElse()` method allows for returning an alternative value.

+   The `orElseGet()` method allows for providing the `Supplier` function, which returns an alternative value.

+   The `ifPresentOrElse()` method allows for providing two functions: one that consumes the value from the `Optional` object, and another that does something if the `Optional` object is empty.

# Min and max

The following Terminal operations return the minimum or maximum value of the stream elements, if present:

+   `Optional<T> min`(Comparator<T> comparator): Returns the minimum element of this stream, using the provided Comparator object.

+   `Optional<T> max`(Comparator<T> comparator): Returns the maximum element of this stream, using the provided Comparator object.

Here is the demonstration code:

```java

List<String> list = List.of("a", "b", "c", "c", "a");

String min = list.stream().min(Comparator.naturalOrder()).orElse("0");

System.out.println(min);     //prints: a

String max = list.stream().max(Comparator.naturalOrder()).orElse("0");

System.out.println(max);     //prints: c

```

As you can see, in the case of non-numerical values, the minimum element is the one that is first (when ordered from the left to the right), according to the provided comparator; the maximum, accordingly, is the last element. In the case of numeric values, the minimum and maximum are just that—the biggest and the smallest number among the stream elements:

```java

int mn = Stream.of(42, 33, 77).min(Comparator.naturalOrder()).orElse(0);

System.out.println(mn);    //prints: 33

int mx = Stream.of(42, 33, 77).max(Comparator.naturalOrder()).orElse(0);

System.out.println(mx);    //prints: 77

```

Let's look at another example, assuming that there is a `Person` class:

```java

class Person {

private int age;

private String name;

public Person(int age, String name) {

this.age = age;

this.name = name;

}

public int getAge() { return this.age; }

public String getName() { return this.name; }

@Override

public String toString() {

return "Person{name:" + this.name + ",age:" + this.age + "}";

}

}

```

The task is to find the oldest person in the following list:

```java

List<Person> persons = List.of(new Person(23, "Bob"),

new Person(33, "Jim"),

new Person(28, "Jill"),

new Person(27, "Bill"));

```

In order to do that, we can create the following `Compartor<Person>`:

```java

Comparator<Person> perComp = (p1, p2) -> p1.getAge() - p2.getAge();

```

Then, using this comparator, we can find the oldest person:

```java

Person theOldest = persons.stream().max(perComp).orElse(null);

System.out.println(theOldest);  //prints: Person{name:Jim,age:33}

```

# The toArray() operation

These two Terminal operations generate an array that contains the stream elements:

+   `Object[] toArray()` : Creates an array of objects; each object is an element of this stream.

+   `A[] toArray(IntFunction<A[]> generator)`: Creates an array of the stream elements using the provided function.

Let's look at an example:

```java

List<String> list = List.of("a", "b", "c");

Object[] obj = list.stream().toArray();

Arrays.stream(obj).forEach(System.out::print);    //prints: abc

String[] str = list.stream().toArray(String[]::new);

Arrays.stream(str).forEach(System.out::print);    //prints: abc

```

The first example is straightforward. It converts elements to an array of the same type. As for the second example, the representation of `IntFunction` as `String[]::new` is probably not obvious, so let's walk through it.

`String[]::new` is a method reference that represents the following lambda expression:

```java

字符串[] str = list.stream().toArray(i -> new String[i]);

Arrays.stream(str).forEach(System.out::print);    //打印：abc

```

这已经是`IntFunction<String[]>`，根据其文档，它接受一个`int`参数并返回指定类型的结果。可以通过使用匿名类来定义，如下所示：

```java

IntFunction<String[]> intFunction = new IntFunction<String[]>() {

@Override

public String[] apply(int i) {

return new String[i];

}

};

```

您可能还记得（来自第十三章，*Java 集合*）我们如何将集合转换为数组：

```java

str = list.toArray(new String[list.size()]);

Arrays.stream(str).forEach(System.out::print);    //打印：abc

```

您可以看到`Stream`接口的`toArray()`操作具有非常相似的签名，只是它接受一个函数，而不仅仅是一个数组。

# reduce 操作

这个终端操作被称为*reduce*，因为它处理所有流元素并产生一个值。它将所有流元素减少为一个值。但这不是唯一的操作。*collect*操作也将流元素的所有值减少为一个结果。而且，在某种程度上，所有终端操作都会减少。它们在处理所有元素后产生一个值。

因此，您可以将*reduce*和*collect*视为帮助为`Stream`接口中提供的许多操作添加结构和分类的同义词。此外，*reduce*组中的操作可以被视为*collect*操作的专门版本，因为`collect()`也可以被定制以提供相同的功能。

有了这个，让我们看看*reduce*操作组：

+   `Optional<T> reduce(BinaryOperator<T> accumulator)`: 使用提供的定义元素聚合逻辑的可关联函数来减少此流的元素。如果可用，返回带有减少值的`Optional`。

+   `T reduce(T identity, BinaryOperator<T> accumulator)`: 提供与先前`reduce()`版本相同的功能，但使用`identity`参数作为累加器的初始值，或者如果流为空则使用默认值。

+   `U reduce(U identity, BiFunction<U,T,U> accumulator, BinaryOperator<U> combiner)`: 提供与先前`reduce()`版本相同的功能，但另外使用`combiner`函数在应用于并行流时聚合结果。如果流不是并行的，则不使用组合器函数。

为了演示`reduce()`操作，我们将使用之前的`Person`类：

```java

类 Person {

private int age;

private String name;

public Person(int age, String name) {

this.age = age;

this.name = name;

}

public int getAge() { return this.age; }

public String getName() { return this.name; }

@Override

public String toString() {

return "Person{name:" + this.name + ",age:" + this.age + "}";

}

}

```

我们还将使用相同的`Person`对象列表作为我们流示例的来源：

```java

List<Person> list = List.of(new Person(23, "Bob"),

new Person(33, "Jim"),

new Person(28, "Jill"),

new Person(27, "Bill"));

```

现在，使用`reduce()`操作，让我们找到此列表中年龄最大的人：

```java

Person theOldest = list.stream()

.reduce((p1, p2) -> p1.getAge() > p2.getAge() ? p1 : p2).orElse(null);

System.out.println(theOldest);         //打印：Person{name:Jim,age:33}

```

这个实现有点令人惊讶，不是吗？我们在谈论“累加器”，但我们没有累加任何东西。我们只是比较了所有的流元素。显然，累加器保存了比较的结果，并将其作为下一个比较（与下一个元素）的第一个参数提供。可以说，在这种情况下，累加器累积了所有先前比较的结果。无论如何，它完成了我们希望它完成的工作。

现在，让我们明确地累积一些东西。让我们将人员名单中的所有名称组合成一个逗号分隔的列表：

```java

String allNames = list.stream().map(p->p.getName())

.reduce((n1, n2) -> n1 + "，" + n2).orElse(null);

System.out.println(allNames);            //打印：Bob, Jim, Jill, Bill

```

在这种情况下，积累的概念更有意义，不是吗？

现在，让我们使用身份值提供一个初始值：

```java

String allNames = list.stream().map(p->p.getName())

.reduce("所有名称：", (n1，n2) -> n1 + "，" + n2);

System.out.println(allNames);       //所有名称：，Bob, Jim, Jill, Bill

```

请注意，这个版本的`reduce()`操作返回值，而不是`Optional`对象。这是因为通过提供初始值，我们保证该值将出现在结果中，即使流为空。

但是，结果字符串看起来并不像我们希望的那样漂亮。显然，提供的初始值被视为任何其他流元素，并且累加器创建的后面添加了逗号。为了使结果再次看起来漂亮，我们可以再次使用`reduce()`操作的第一个版本，并通过这种方式添加初始值：

```java

String allNames = "所有名称：" + list.stream().map(p->p.getName())

.reduce((n1, n2) -> n1 + "，" + n2).orElse(null);

System.out.println(allNames);         //所有名称：Bob, Jim, Jill, Bill

```

我们决定使用空格作为分隔符，而不是逗号，以进行演示：

```java

String allNames = list.stream().map(p->p.getName())

.reduce("所有名称：", (n1, n2) -> n1 + " " + n2);

System.out.println(allNames);        //所有名称：Bob, Jim, Jill, Bill

```

现在，结果看起来更好了。在下一小节中演示`collect()`操作时，我们将向您展示另一种使用前缀创建逗号分隔值列表的方法。

现在，让我们看看如何使用`reduce()`操作的第三种形式——具有三个参数的形式，最后一个称为组合器。将组合器添加到前面的`reduce()`操作中不会改变结果：

```java

String allNames = list.stream().map(p->p.getName())

.reduce("所有名称：", (n1, n2) -> n1 + " " + n2,

(n1, n2) -> n1 + " " + n2 );

System.out.println(allNames);          //所有名称：Bob, Jim, Jill, Bill

```

这是因为流不是并行的，并且组合器仅与并行流一起使用。

如果我们使流并行，结果会改变：

```java

String allNames = list.parallelStream().map(p->p.getName())

.reduce("所有名称：", (n1, n2) -> n1 + " " + n2,

(n1, n2) -> n1 + " " + n2 );

System.out.println(allNames);

//所有名称：Bob 所有名称：Jim 所有名称：Jill 所有名称：Bill

```

显然，对于并行流，元素序列被分成子序列，每个子序列都是独立处理的；它们的结果由组合器聚合。这样做时，组合器将初始值（身份）添加到每个结果中。即使我们删除组合器，并行流处理的结果仍然是相同的，因为提供了默认的组合器行为：

```java

String allNames = list.parallelStream().map(p->p.getName())

.reduce("所有名称：", (n1, n2) -> n1 + " " + n2);

System.out.println(allNames);

//所有名称：Bob 所有名称：Jim 所有名称：Jill 所有名称：Bill

```

在前两种`reduce()`操作中，标识值被累加器使用。在第三种形式中，使用了`U reduce(U identity, BiFunction<U,T,U> accumulator, BinaryOperator<U> combiner)`签名，标识值被组合器使用（注意，`U`类型是组合器类型）。

为了消除结果中重复的标识值，我们决定从 combiner 的第二个参数中删除它：

```java

allNames = list.parallelStream().map(p->p.getName())

.reduce("所有名称:", (n1, n2) -> n1 + " " + n2，

(n1, n2) -> n1 + " " + StringUtils.remove(n2, "所有名称:"));

System.out.println(allNames); //所有名称：Bob, Jim, Jill, Bill

```

如您所见，结果现在看起来好多了。

到目前为止，我们的例子中，标识不仅起到了初始值的作用，还起到了结果中的标识（标签）的作用。当流的元素是数字时，标识看起来更像是初始值。让我们看下面的例子：

```java

整数列表= List.of(1, 2, 3);

int sum = ints.stream().reduce((i1, i2) -> i1 + i2).orElse(0);

System.out.println(sum); //打印：6

sum = ints.stream().reduce(Integer::sum).orElse(0);

System.out.println(sum); //打印：6

sum = ints.stream().reduce(10, Integer::sum);

System.out.println(sum); //打印：16

sum = ints.stream().reduce(10, Integer::sum, Integer::sum);

System.out.println(sum); //打印：16

```

前两个流管道完全相同，只是第二个管道使用了方法引用而不是 lambda 表达式。第三个和第四个管道也具有相同的功能。它们都使用初始值 10。现在第一个参数作为初始值比标识更有意义，不是吗？在第四个管道中，我们添加了一个组合器，但它没有被使用，因为流不是并行的。

让我们并行处理一下，看看会发生什么：

```java

整数列表= List.of(1, 2, 3);

int sum = ints.parallelStream().reduce(10, Integer::sum, Integer::sum);

System.out.println(sum); //打印：36

```

结果为 36，因为初始值 10 被添加了三次-每次都有部分结果。显然，流被分成了三个子序列。但情况并非总是如此，随着流的增长和计算机上 CPU 数量的增加而发生变化。因此，不能依赖于一定数量的子序列，最好不要在这种情况下使用它，如果需要，可以添加到结果中：

```java

整数列表= List.of(1, 2, 3);

int sum = ints.parallelStream().reduce(0, Integer::sum, Integer::sum);

System.out.println(sum); //打印：6

sum = 10 + ints.parallelStream().reduce(0, Integer::sum, Integer::sum);

System.out.println(sum); //打印：16

```

# 收集操作

`collect()`操作的一些用法非常简单，适合任何初学者，而其他情况可能复杂，即使对于经验丰富的程序员也难以理解。除了已经讨论过的操作之外，我们在本节中介绍的`collect()`的最受欢迎的用法已经足够满足初学者的所有需求。再加上我们将在*数字流接口*部分介绍的数字流操作，覆盖的内容可能很容易是未来主流程序员所需的一切。

正如我们已经提到的，collect 操作非常灵活，允许我们自定义流处理。它有两种形式：

+   `R collect(Collector<T, A, R> collector)`:使用提供的`Collector`处理此`T`类型的流的元素，并通过`A`类型的中间累积产生`R`类型的结果

+   `R collect(Supplier<R> supplier, BiConsumer<R, T> accumulator, BiConsumer<R, R> combiner)`: 使用提供的函数处理`T`类型的流的元素：

+   `Supplier<R>`: 创建一个新的结果容器

+   `BiConsumer<R, T> accumulator`: 一个无状态的函数，将一个元素添加到结果容器中

+   `BiConsumer<R, R> combiner`：一个无状态的函数，将两个部分结果容器合并在一起，将第二个结果容器的元素添加到第一个结果容器中。

让我们看看`collect()`操作的第二种形式。它与`reduce()`操作非常相似，具有我们刚刚演示的三个参数。最大的区别在于`collect()`操作中的第一个参数不是标识或初始值，而是容器——一个对象，将在函数之间传递，并维护处理的状态。对于以下示例，我们将使用`Person1`类作为容器：

```java

class Person1 {

private String name;

private int age;

public Person1(){}

public String getName() { return this.name; }

public void setName(String name) { this.name = name; }

public int getAge() {return this.age; }

public void setAge(int age) { this.age = age;}

@Override

public String toString() {

return "人{name:" + this.name + ",年龄:" + age + "}";

}

}

```

正如你所看到的，容器必须有一个没有参数的构造函数和 setter，因为它应该能够接收和保留部分结果——迄今为止年龄最大的人的姓名和年龄。`collect()`操作将在处理每个元素时使用这个容器，并且在处理完最后一个元素后，将包含年龄最大的人的姓名和年龄。这是人员名单，你应该很熟悉：

```java

List<Person> list = List.of(new Person(23, "Bob"),

new Person(33, "吉姆"),

新的人(28, "吉尔"),

new Person(27, "Bill"));

```

这是应该在列表中找到最年长的人的`collect()`操作：

```java

Person1 theOldest = list.stream().collect(Person1::new,

(p1, p2) -> {

if(p1.getAge() < p2.getAge()){

p1.setAge(p2.getAge());

p1.setName(p2.getName());

}

},

(p1, p2) -> { System.out.println("组合器被调用了!"); });

```

我们尝试在操作调用中内联函数，但看起来有点难以阅读，所以这是相同代码的更好版本：

```java

BiConsumer<Person1, Person> accumulator = (p1, p2) -> {

if(p1.getAge() < p2.getAge()){

p1.setAge(p2.getAge());

p1.setName(p2.getName());

}

};

BiConsumer<Person1, Person1> combiner = (p1, p2) -> {

System.out.println("组合器被调用了!");        //不打印任何内容

};

theOldest = list.stream().collect(Person1::new, accumulator, combiner);

System.out.println(theOldest);        //打印：人{name:吉姆,年龄:33}

```

`Person1`容器对象只创建一次——用于第一个元素的处理（在这个意义上，它类似于`reduce()`操作的初始值）。然后将其传递给比较器，与第一个元素进行比较。容器中的`age`字段被初始化为零的默认值，因此，迄今为止，容器中设置了第一个元素的年龄和姓名作为年龄最大的人的参数。

当流的第二个元素（`Person`对象）被发出时，它的`age`字段与容器（`Person1`对象）中当前存储的`age`值进行比较，依此类推，直到处理完流的所有元素。结果如前面的注释所示。

组合器从未被调用，因为流不是并行的。但是当我们并行时，我们需要实现组合器如下：

```java

BiConsumer<Person1, Person1> combiner = (p1, p2) -> {

System.out.println("组合器被调用了!");   //打印 3 次

if(p1.getAge() < p2.getAge()){

p1.setAge(p2.getAge());

p1.setName(p2.getName());

}

};

theOldest = list.parallelStream()

.collect(Person1::new, accumulator, combiner);

System.out.println(theOldest);  //prints: Person{name:Jim,age:33}

```

组合器比较了所有流子序列的部分结果，并得出最终结果。现在我们看到`Combiner is called!`消息打印了三次。但是，与`reduce()`操作一样，部分结果（流子序列）的数量可能会有所不同。

现在让我们来看一下`collect()`操作的第一种形式。它需要一个实现`java.util.stream.Collector<T,A,R>`接口的类的对象，其中`T`是流类型，`A`是容器类型，`R`是结果类型。可以使用`Collector`接口的`of()`方法来创建必要的`Collector`对象：

+   `static Collector<T,R,R> of(Supplier<R> supplier, BiConsumer<R,T> accumulator, BinaryOperator<R> combiner, Collector.Characteristics... characteristics)`

+   `static Collector<T,A,R> of(Supplier<A> supplier, BiConsumer<A,T> accumulator, BinaryOperator<A> combiner, Function<A,R> finisher, Collector.Characteristics... characteristics)`.

前面方法中必须传递的函数与我们已经演示过的函数类似。但我们不打算这样做有两个原因。首先，这涉及的内容更多，超出了本入门课程的范围，其次，在这之前，必须查看提供了许多现成收集器的`java.util.stream.Collectors`类。正如我们已经提到的，加上本书讨论的操作和我们将在*数字流接口*部分介绍的数字流操作，它们涵盖了主流编程中绝大多数处理需求，很可能你根本不需要创建自定义收集器。

# 类收集器

`java.util.stream.Collectors`类提供了 40 多种方法来创建`Collector`对象。我们将仅演示最简单和最流行的方法：

+   `Collector<T,?,List<T>> toList()`：创建一个收集器，将流元素收集到一个`List`对象中。

+   `Collector<T,?,Set<T>> toSet()`：创建一个收集器，将流元素收集到一个`Set`对象中。

+   `Collector<T,?,Map<K,U>> toMap (Function<T,K> keyMapper, Function<T,U> valueMapper)`：创建一个收集器，将流元素收集到一个`Map`对象中。

+   `Collector<T,?,C> toCollection (Supplier<C> collectionFactory)`：创建一个收集器，将流元素收集到由集合工厂指定类型的`Collection`对象中。

+   `Collector<CharSequence,?,String> joining()`：创建一个收集器，将元素连接成一个`String`值。

+   `Collector<CharSequence,?,String> joining (CharSequence delimiter)`：创建一个收集器，将元素连接成一个以提供的分隔符分隔的`String`值。

+   `Collector<CharSequence,?,String> joining (CharSequence delimiter, CharSequence prefix, CharSequence suffix)`：创建一个收集器，将元素连接成一个以提供的前缀和后缀分隔的`String`值。

+   `Collector<T,?,Integer> summingInt(ToIntFunction<T>)`：创建一个计算由提供的函数应用于每个元素生成的结果的总和的收集器。相同的方法也适用于`long`和`double`类型。

+   `Collector<T,?,IntSummaryStatistics> summarizingInt(ToIntFunction<T>)`：创建一个收集器，计算由提供的函数应用于每个元素生成的结果的总和、最小值、最大值、计数和平均值。相同的方法也适用于`long`和`double`类型。

+   `Collector<T,?,Map<Boolean,List<T>>> partitioningBy (Predicate<? super T> predicate)`：创建一个收集器，根据提供的`Predicate`函数将元素分区。

+   `Collector<T,?,Map<K,List<T>>> groupingBy(Function<T,U>)`：创建一个收集器，将元素分组到由提供的函数生成的`Map`中。

The following demo code shows how to use the collectors created by these methods. First, we demonstrate usage of the  `toList()`, `toSet()`, `toMap()`, and `toCollection()` methods:

```java

List<String> ls = Stream.of("a", "b", "c").collect(Collectors.toList());

System.out.println(ls);                //prints: [a, b, c]

Set<String> set = Stream.of("a", "a", "c").collect(Collectors.toSet());

System.out.println(set);                //prints: [a, c]

List<Person> persons = List.of(new Person(23, "Bob"),

new Person(33, "Jim"),

new Person(28, "Jill"),

new Person(27, "Bill"));

Map<String, Person> map = persons.stream()

.collect(Collectors.toMap(p->p.getName() + "-" + p.getAge(), p->p));

System.out.println(map); //prints: {Bob-23=Person{name:Bob,age:23},

Bill-27=Person{name:Bill,age:27},

Jill-28=Person{name:Jill,age:28},

Jim-33=Person{name:Jim,age:33}}

Set<Person> personSet = persons.stream()

.collect(Collectors.toCollection(HashSet::new));

System.out.println(personSet);  //prints: [Person{name:Bill,age:27},

Person{name:Jim,age:33},

Person{name:Bob,age:23},

Person{name:Jill,age:28}]

```

The `joining()` method allows concatenating the `Character` and `String` values in a delimited list with a prefix and suffix:

```java

List<String> list = List.of("a", "b", "c", "d");

String result = list.stream().collect(Collectors.joining());

System.out.println(result);           //abcd

result = list.stream().collect(Collectors.joining(", "));

System.out.println(result);           //a, b, c, d

result = list.stream()

.collect(Collectors.joining(", ", "The result: ", ""));

System.out.println(result);          //The result: a, b, c, d

result = list.stream()

.collect(Collectors.joining(", ", "The result: ", ". The End."));

System.out.println(result);          //The result: a, b, c, d. The End.

```

The `summingInt()` and `summarizingInt()` methods create collectors that calculate the sum and other statistics of the `int` values produced by the provided function applied to each element:

```java

List<Person> list = List.of(new Person(23, "Bob"),

new Person(33, "Jim"),

new Person(28, "Jill"),

new Person(27, "Bill"));

int sum = list.stream().collect(Collectors.summingInt(Person::getAge));

System.out.println(sum);  //prints: 111

IntSummaryStatistics stats =

list.stream().collect(Collectors.summarizingInt(Person::getAge));

System.out.println(stats);     //IntSummaryStatistics{count=4, sum=111,

//    min=23, average=27.750000, max=33}

System.out.println(stats.getCount());    //4

System.out.println(stats.getSum());      //111

System.out.println(stats.getMin());      //23

System.out.println(stats.getAverage());  //27.750000

System.out.println(stats.getMax());      //33

```

There are also `summingLong()`, `summarizingLong()` , `summingDouble()`, and `summarizingDouble()` methods.

The `partitioningBy()` method creates a collector that groups the elements by the provided criteria and put the groups (lists) in a `Map` object with a `boolean` value as the key:

```java

List<Person> list = List.of(new Person(23, "Bob"),

new Person(33, "Jim"),

new Person(28, "Jill"),

new Person(27, "Bill"));

Map<Boolean, List<Person>> map =

list.stream().collect(Collectors.partitioningBy(p->p.getAge() > 27));

System.out.println(map);

//{false=[Person{name:Bob,age:23}, Person{name:Bill,age:27}],

//  true=[Person{name:Jim,age:33}, Person{name:Jill,age:28}]}

```

As you can see, using the `p.getAge() > 27` criteria, we were able to put all the people in two groups—one is below or equals 27 years of age (the key is `false`), and the other is above 27 (the key is `true`).

And, finally, the `groupingBy()` method allows us to group elements by a value and put the groups (lists) in a `Map` object with this value as a key:

```java

List<Person> list = List.of(new Person(23, "Bob"),

new Person(33, "Jim"),

new Person(23, "Jill"),

new Person(33, "Bill"));

Map<Integer, List<Person>> map =

list.stream().collect(Collectors.groupingBy(Person::getAge));

System.out.println(map);

//{33=[Person{name:Jim,age:33}, Person{name:Bill,age:33}],

// 23=[Person{name:Bob,age:23}, Person{name:Jill,age:23}]}

```

为了演示前面的方法，我们通过将每个人的年龄设置为 23 或 33 来改变了`Person`对象的列表。结果是按年龄分成两组。

还有重载的`toMap()`、`groupingBy()`和`partitioningBy()`方法，以及以下通常也重载的方法，它们创建相应的`Collector`对象：

+   `counting()`

+   `reducing()`

+   `filtering()`

+   `toConcurrentMap()`

+   ``collectingAndThen()``

+   `maxBy()` 和 `minBy()`

+   `mapping()` 和 `flatMapping()`

+   `averagingInt()`, `averagingLong()`, 和 `averagingDouble()`

+   `toUnmodifiableList()`、`toUnmodifiableMap()`和 `toUnmodifiableSet()`

如果在本书中找不到所需的操作，请先搜索`Collectors`API，然后再构建自己的`Collector`对象。

# 数字流接口

正如我们已经提到的，所有三个数字接口，`IntStream`、`LongStream`和`DoubleStream`，都有类似于`Stream`接口的方法，包括`Stream.Builder`接口的方法。这意味着我们在本章中讨论的所有内容同样适用于任何数字流接口。因此，在本节中，我们只会讨论`Stream`接口中不存在的那些方法：

+   `IntStream`和`LongStream`接口中的`range(lower,upper)`和`rangeClosed(lower,upper)`方法。它们允许我们从指定范围内的值创建流。

+   `boxed()`和`mapToObj()`中间操作，将数字流转换为`Stream`。

+   `mapToInt()`、`mapToLong()`和`mapToDouble()`中间操作，将一个类型的数字流转换为另一个类型的数字流。

+   `flatMapToInt()`、`flatMapToLong()`和`flatMapToDouble()`中间操作，将流转换为数字流。

+   `sum()`和`average()`终端操作，计算数字流元素的和和平均值。

# 创建流

除了创建流的`Stream`接口方法外，`IntStream`和`LongStream`接口还允许我们从指定范围内的值创建流。

# range()，rangeClosed()

`range(lower, upper)`方法按顺序生成所有值，从`lower`值开始，以`upper`值之前的值结束：

```java

IntStream.range(1, 3).forEach(System.out::print);  //prints: 12

LongStream.range(1, 3).forEach(System.out::print);  //prints: 12

```

`rangeClosed(lower, upper)` 方法按顺序生成所有值，从`lower`值开始，以`upper`值结束：

```java

IntStream.rangeClosed(1, 3).forEach(System.out::print);  //prints: 123

LongStream.rangeClosed(1, 3).forEach(System.out::print);  //prints: 123

```

# 中间操作

除了`Stream`中间操作外，`IntStream`、`LongStream`和`DoubleStream`接口还具有特定于数字的中间操作：`boxed()`、`mapToObj()`、`mapToInt()`、`mapToLong()`、`mapToDouble()`、`flatMapToInt()`、`flatMapToLong()`和`flatMapToDouble()`。

# boxed()和 mapToObj()

`boxed()` 中间操作将原始数值类型的元素转换（装箱）为相应的包装类型：

```java

//IntStream.range(1, 3).map(Integer::shortValue)        //compile error

//                     .forEach(System.out::print);

IntStream.range(1, 3).boxed().map(Integer::shortValue)

.forEach(System.out::print);  //prints: 12

//LongStream.range(1, 3).map(Long::shortValue)          //compile error

//                      .forEach(System.out::print);

LongStream.range(1, 3).boxed().map(Long::shortValue)

.forEach(System.out::print);  //prints: 12

//DoubleStream.of(1).map(Double::shortValue)            //compile error

//                  .forEach(System.out::print);

DoubleStream.of(1).boxed().map(Double::shortValue)

.forEach(System.out::print);      //打印：1

```

在上述代码中，我们已经注释掉了生成编译错误的行，因为`range()`方法生成的元素是原始类型。通过添加`boxed()`操作，我们将原始值转换为相应的包装类型，然后可以将它们作为引用类型进行处理。

`mapToObj()`中间操作进行了类似的转换，但它不像`boxed()`操作那样专门化，并且允许使用原始类型的元素来生成任何类型的对象：

```java

IntStream.range(1, 3).mapToObj(Integer::valueOf)

.map(Integer::shortValue)

.forEach(System.out::print);       //打印：12

IntStream.range(42, 43).mapToObj(i -> new Person(i, "John"))

.forEach(System.out::print);

//打印：Person{name:John,age:42}

LongStream.range(1, 3).mapToObj(Long::valueOf)

.map(Long::shortValue)

.forEach(System.out::print);      //打印：12

DoubleStream.of(1).mapToObj(Double::valueOf)

.map(Double::shortValue)

.forEach(System.out::print);          //打印：1

```

在上述代码中，我们添加了`map()`操作，只是为了证明`mapToObj()`操作可以按预期执行工作并创建包装类型对象。此外，通过添加生成`Person`对象的流管道，我们演示了如何使用`mapToObj()`操作来创建任何类型的对象。

# mapToInt()、mapToLong()和 mapToDouble()

`mapToInt()`、`mapToLong()`、`mapToDouble()`中间操作允许我们将一个类型的数值流转换为另一种类型的数值流。在演示代码中，我们通过将每个`String`值映射到其长度，将`String`值列表转换为不同类型的数值流：

```java

list.stream().mapToInt(String::length)

.forEach(System.out::print); //打印：335

list.stream().mapToLong(String::length)

.forEach(System.out::print); //打印：335

list.stream().mapToDouble(String::length)

.forEach(d -> System.out.print(d + " "));   //打印：3.0 3.0 5.0

```

创建的数值流的元素是原始类型的：

```java

//list.stream().mapToInt(String::length)

//             .map(Integer::shortValue)   //编译错误

//             .forEach(System.out::print);

```

既然我们在这个话题上，如果您想将元素转换为数值包装类型，`map()`中间操作就是这样做的方法（而不是`mapToInt()`）：

```java

list.stream().map(String::length)

.map(Integer::shortValue)

.forEach(System.out::print);  //打印：335

```

# flatMapToInt()、flatMapToLong()和 flatMapToDouble()

`flatMapToInt()`、`flatMapToLong()`、`flatMapToDouble()`中间操作会生成相应类型的数值流：

```java

List<Integer> list = List.of(1, 2, 3);

list.stream().flatMapToInt(i -> IntStream.rangeClosed(1, i))

.forEach(System.out::print);    //打印：112123

list.stream().flatMapToLong(i -> LongStream.rangeClosed(1, i))

.forEach(System.out::print);    //打印：112123

list.stream().flatMapToDouble(DoubleStream::of)

.forEach(d -> System.out.print(d + " "));  //打印：1.0 2.0 3.0

```

如您所见，在上述代码中，我们在原始流中使用了`int`值。但它可以是任何类型的流：

```java

List<String> str = List.of("one", "two", "three");

str.stream().flatMapToInt(s -> IntStream.rangeClosed(1, s.length()))

.forEach(System.out::print);  //打印：12312312345

```

# 终端操作

数值流的附加终端操作非常简单。它们中有两个：

+   `sum()`: 计算数值流元素的总和

+   `average()`: 计算数值流元素的平均值

# sum()和 average()

如果您需要计算数值流元素的总和或平均值，则流的唯一要求是它不应该是无限的。否则，计算永远不会完成：

```java

int sum = IntStream.empty().sum();

System.out.println(sum);          //打印：0

sum = IntStream.range(1, 3).sum();

System.out.println(sum);          //打印：3

double av = IntStream.empty().average().orElse(0);

System.out.println(av);           //打印：0.0

av = IntStream.range(1, 3).average().orElse(0);

System.out.println(av);           //打印：1.5

long suml = LongStream.range(1, 3).sum();

System.out.println(suml);         //打印：3

double avl = LongStream.range(1, 3).average().orElse(0);

System.out.println(avl);          //打印：1.5

double sumd = DoubleStream.of(1, 2).sum();

System.out.println(sumd);         //打印：3.0

double avd = DoubleStream.of(1, 2).average().orElse(0);

System.out.println(avd);          //打印：1.5

```

正如您所看到的，对空流使用这些操作不是问题。

# 并行处理

我们已经看到，从顺序流切换到并行流可能会导致不正确的结果，如果代码没有为处理并行流而编写和测试。以下是与并行流相关的一些其他考虑。

# 无状态和有状态的操作

有无状态的操作，比如`filter()`、`map()`和`flatMap()`，在从一个流元素的处理转移到下一个流元素的处理时不会保留数据（不维护状态）。还有有状态的操作，比如`distinct()`、`limit()`、`sorted()`、`reduce()`和`collect()`，可能会将先前处理的元素的状态传递给下一个元素的处理。

无状态操作通常在从顺序流切换到并行流时不会造成问题。每个元素都是独立处理的，流可以被分成任意数量的子流进行独立处理。

对于有状态的操作，情况是不同的。首先，对无限流使用它们可能永远无法完成处理。此外，在讨论`reduce()`和`collect()`有状态操作时，我们已经演示了如果初始值（或标识）在没有考虑并行处理的情况下设置，切换到并行流可能会产生不同的结果。

而且还有性能方面的考虑。有状态的操作通常需要使用缓冲区多次处理所有流元素。对于大流，这可能会消耗 JVM 资源并减慢甚至完全关闭应用程序。

这就是为什么程序员不应该轻易从顺序流切换到并行流。如果涉及有状态的操作，代码必须被设计和测试，以便能够在没有负面影响的情况下执行并行流处理。

# 顺序或并行处理？

正如我们在前一节中所指出的，并行处理可能会产生更好的性能，也可能不会。在决定使用之前，必须测试每个用例。并行处理可能会产生更好的性能，但代码必须被设计和可能被优化。每个假设都必须在尽可能接近生产环境的环境中进行测试。

然而，在决定顺序处理和并行处理之间可以考虑一些因素：

+   通常情况下，小流在顺序处理时处理速度更快（对于您的环境来说，“小”是通过测试和测量性能来确定的）

+   如果有状态的操作无法用无状态的操作替换，那么必须仔细设计代码以进行并行处理，或者完全避免它。

+   考虑对需要大量计算的程序进行并行处理，但要考虑将部分结果合并为最终结果

# 练习 - 将所有流元素相乘

使用流来将以下列表的所有值相乘：

```java

List<Integer> list = List.of(2, 3, 4);

```

# 答案

```java

int r = list.stream().reduce(1, (x, y) -> x * y);

System.out.println(r);     //打印：24

```

# 总结

本章介绍了数据流处理的强大概念，并提供了许多函数式编程使用示例。它解释了流是什么，如何处理它们以及如何构建处理管道。它还演示了如何可以并行组织流处理以及一些可能的陷阱。

在下一章中，我们将讨论反应式系统，它们的优势以及可能的实现。您将了解异步非阻塞处理、反应式编程和微服务，所有这些都有代码示例，演示了这些反应式系统所基于的主要原则。
