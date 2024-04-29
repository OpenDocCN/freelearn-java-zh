# 第十七章：函数式编程

你可能知道，Java 不像 Haskell 那样是一种纯函数式编程语言，但从版本 8 开始，Java 添加了一些函数式支持。添加这种支持的努力取得了成功，并且函数式代码被开发人员和公司广泛采用。函数式编程支持更易理解、易维护和易测试的代码。然而，以函数式风格编写 Java 代码需要严肃的了解 lambda、流 API、`Optional`、函数接口等知识。所有这些函数式编程主题也可以是面试的主题，在本章中，我们将涵盖一些必须了解的热门问题，以通过常规的 Java 面试。我们的议程包括以下主题：

+   Java 函数式编程概述

+   问题和编码挑战

让我们开始吧！

# Java 函数式编程概述

像往常一样，本节旨在突出和复习我们主题的主要概念，并为回答技术面试中可能出现的基本问题提供全面的资源。

## 函数式编程的关键概念

因此，函数式编程的关键概念包括以下内容：

+   函数作为一等对象

+   纯函数

+   高阶函数

让我们简要地介绍一下这些概念。

### 函数作为一等对象

说函数是一等对象意味着我们可以创建一个函数的*实例*，并将变量引用该函数实例。这就像引用`String`、`List`或任何其他对象。此外，函数可以作为参数传递给其他函数。然而，Java 方法不是一等对象。我们能做的最好的事情就是依赖于 Java lambda 表达式。

### 纯函数

*纯*函数是一个执行没有*副作用*的函数，返回值仅取决于其输入参数。以下 Java 方法是一个纯函数：

```java
public class Calculator {
  public int sum(int x, int y) {
    return x + y;
  }
}
```

如果一个方法使用成员变量或改变成员变量的状态，那么它就不是一个*纯*函数。

### 高阶函数

高阶函数将一个或多个函数作为参数和/或返回另一个函数作为结果。Java 通过 lambda 表达式模拟高阶函数。换句话说，在 Java 中，高阶函数是一个以一个（或多个）lambda 表达式作为参数和/或返回另一个 lambda 表达式的方法。

例如，`Collections.sort()`方法接受一个`Comparator`作为参数，这是一个高阶函数：

```java
Collections.sort(list, (String x, String y) -> {
  return x.compareTo(y);
});
```

`Collections.sort()`的第一个参数是一个`List`，第二个参数是一个 lambda 表达式。这个 lambda 表达式参数是使`Collections.sort()`成为一个高阶函数的原因。

### 纯函数式编程规则

现在，让我们简要讨论纯函数式编程规则。纯函数式编程也有一套规则要遵循。这些规则如下：

+   没有状态

+   没有副作用

+   不可变变量

+   偏爱递归而不是循环

让我们简要地介绍一下这些规则。

### 没有状态

通过*无状态*，我们并不是指函数式编程消除了状态。通常，无状态意味着函数没有外部状态。换句话说，函数可能使用包含临时状态的局部变量，但不能引用其所属类/对象的任何成员变量。

### 无副作用

通过“无副作用”，我们应该理解一个函数不能改变（突变）函数之外的任何状态（在其功能范围之外）。函数之外的状态包括以下内容：

+   包含该函数的类/对象中的成员变量

+   作为参数传递给函数的成员变量

+   或外部系统中的状态（例如数据库或文件）。

### 不可变变量

函数式编程鼓励并支持不可变变量的使用。依赖不可变变量有助于我们更轻松、更直观地避免*副作用*。

### 更喜欢递归而不是循环

由于递归依赖于重复的函数调用来模拟循环，代码变得更加函数式。这意味着不鼓励使用以下迭代方法来计算阶乘：

```java
static long factorial(long n) {
  long result = 1;
  for (; n > 0; n--) {
    result *= n;
  }
  return result;
}
```

函数式编程鼓励以下递归方法：

```java
static long factorial(long n) {
  return n == 1 ? 1 : n * factorial(n - 1);
}
```

我们使用*尾递归*来改善性能损耗，因为在前面的例子中，每个函数调用都保存为递归堆栈中的一个帧。当存在许多递归调用时，尾递归是首选。在尾递归中，函数执行递归调用作为最后要做的事情，因此编译器不需要将函数调用保存为递归堆栈中的帧。大多数编译器将优化尾递归，从而避免性能损耗：

```java
static long factorialTail(long n) {
  return factorial(1, n);
}
static long factorial(long acc, long v) {
  return v == 1 ? acc : factorial(acc * v, v - 1);
}
```

另外，循环可以通过受 Java Stream API 的启发来实现：

```java
static long factorial(long n) {
  return LongStream.rangeClosed(1, n)
     .reduce(1, (n1, n2) -> n1 * n2);
}
```

现在，是时候练习一些问题和编码挑战了。

# 问题和编码挑战

在本节中，我们将涵盖 21 个在面试中非常流行的问题和编码挑战。让我们开始吧！

## 编码挑战 1- Lambda 部分

**问题**：描述 Java 中 lambda 表达式的部分。此外，什么是 lambda 表达式的特征？

**解决方案**：如下图所示，lambda 有三个主要部分：

![图 17.1- Lambda 部分](img/Figure_17.1_B15403.jpg)

图 17.1- Lambda 部分

lambda 表达式的部分如下：

+   在箭头的左侧，是 lambda 的参数，这些参数在 lambda 主体中被使用。在这个例子中，这些是`FilenameFilter.accept(File folder, String fileName)`方法的参数。

+   在箭头的右侧，是 lambda 的主体。在这个例子中，lambda 的主体检查文件（`fileName`）所在的文件夹（`folder`）是否可读，并且这个文件的名称是否以*.pdf*字符串结尾。

+   箭头位于参数列表和 lambda 主体之间，起到分隔作用。

接下来，让我们谈谈 lambda 表达式的特征。因此，如果我们写出前面图表中 lambda 的匿名类版本，那么它将如下所示：

```java
FilenameFilter filter = new FilenameFilter() {
  @Override
  public boolean accept(File folder, String fileName) {
    return folder.canRead() && fileName.endsWith(".pdf");
  }
};
```

现在，如果我们比较匿名版本和 lambda 表达式，我们会注意到 lambda 表达式是一个简洁的匿名函数，可以作为参数传递给方法或保存在变量中。

下图中显示的四个词表征了 lambda 表达式：

![图 17.2- Lambda 特征](img/Figure_17.2_B15403.jpg)

图 17.2- Lambda 特征

作为一个经验法则，请记住，lambda 支持行为参数化设计模式（行为作为函数的参数传递），并且只能在功能接口的上下文中使用。

## 编码挑战 2-功能接口

**问题**：什么是功能接口？

**解决方案**：在 Java 中，功能接口是一个只包含一个抽象方法的接口。换句话说，功能接口只包含一个未实现的方法。因此，功能接口将函数作为接口进行封装，并且该函数由接口上的单个抽象方法表示。

除了这个抽象方法之外，功能接口还可以有默认和/或静态方法。通常，功能接口会用`@FunctionalInterface`进行注解。这只是一个信息性的注解类型，用于标记功能接口。

这是一个功能接口的例子：

```java
@FunctionalInterface
public interface Callable<V> {
  V call() throws Exception;
}
```

根据经验法则，如果一个接口有更多没有实现的方法（即抽象方法），那么它就不再是一个函数式接口。这意味着这样的接口不能被 Java lambda 表达式实现。

## 编码挑战 3 - 集合与流

**问题**：集合和流之间的主要区别是什么？

**解决方案**：集合和流是非常不同的。一些不同之处如下：

+   `List`、`Set`和`Map`），流旨在对该数据应用操作（例如*过滤*、*映射*和*匹配*）。换句话说，流对存储在集合上的数据表示的视图/源应用复杂的操作。此外，对流进行的任何修改/更改都不会反映在原始集合中。

+   **数据修改**：虽然我们可以向集合中添加/删除元素，但我们不能向流中添加/删除元素。实际上，流消耗视图/源，对其执行操作，并在不修改视图/源的情况下返回结果。

+   **迭代**：流消耗视图/源时，它会自动在内部执行该视图/源的迭代。迭代取决于选择应用于视图/源的操作。另一方面，集合必须在外部进行迭代。

+   **遍历**：集合可以被多次遍历，而流只能被遍历一次。因此，默认情况下，Java 流不能被重用。尝试两次遍历流将导致错误读取*Stream has already been operated on or closed*。

+   **构造**：集合是急切构造的（所有元素从一开始就存在）。另一方面，流是懒惰构造的（所谓的*中间*操作直到调用*终端*操作才被评估）。

## 编码挑战 4 - map()函数

`map()`函数是做什么的，为什么要使用它？

`map()`函数是一个名为*映射*的中间操作，通过`Stream` API 可用。它用于通过简单应用给定函数将一种类型的对象转换为另一种类型。因此，`map()`遍历给定流，并通过应用给定函数将每个元素转换为它的新版本，并在新的`Stream`中累积结果。给定的`Stream`不会被修改。例如，通过`Stream#map()`将`List<String>`转换为`List<Integer>`可以如下进行：

```java
List<String> strList = Arrays.asList("1", "2", "3");
List<Integer> intList = strList.stream()
  .map(Integer::parseInt)
  .collect(Collectors.toList());
```

挑战自己多练习一些例子。尝试应用`map()`将一个数组转换为另一个数组。

## 编码挑战 5 - flatMap()函数

`flatMap()`函数是做什么的，为什么要使用它？

`flatMap()`函数是一个名为*展平*的中间操作，通过`Stream` API 可用。这个函数是`map()`的扩展，意味着除了将给定对象转换为另一种类型的对象之外，它还可以展平它。例如，有一个`List<List<Object>>`，我们可以通过`Stream#flatMap()`将其转换为`List<Object>`，如下所示：

```java
List<List<Object>> list = ...
List<Object> flatList = list.stream()
  .flatMap(List::stream)
  .collect(Collectors.toList());
```

下一个编码挑战与此相关，所以也要考虑这一点。

## 编码挑战 6 - map()与 flatMap()

`map()`和`flatMap()`函数？

`flatMap()`函数还能够将给定对象展平。换句话说，`flatMap()`也可以展平一个`Stream`对象。

为什么这很重要？嗯，`map()`知道如何将一系列元素包装在`Stream`中，对吧？这意味着`map()`可以生成诸如`Stream<String[]>`、`Stream<List<String>>`、`Stream<Set<String>>`甚至`Stream<Stream<R>>`等流。但问题是，这些类型的流不能被流操作成功地操作（即，如我们所期望的那样）`sum()`、`distinct()`和`filter()`。

例如，让我们考虑以下`List`：

```java
List<List<String>> melonLists = Arrays.asList(
  Arrays.asList("Gac", "Cantaloupe"),
  Arrays.asList("Hemi", "Gac", "Apollo"),
  Arrays.asList("Gac", "Hemi", "Cantaloupe"));
```

我们试图从这个列表中获取甜瓜的不同名称。如果将数组包装成流可以通过`Arrays.stream()`来完成，对于集合，我们有`Collection.stream()`。因此，第一次尝试可能如下所示：

```java
melonLists.stream()
  .map(Collection::stream) // Stream<Stream<String>>
  .distinct();
```

但这不起作用，因为`map()`将返回`Stream<Stream<String>>`。解决方案由`flatMap()`提供，如下所示：

```java
List<String> distinctNames = melonLists.stream()
  .flatMap(Collection::stream) // Stream<String>
  .distinct()
  .collect(Collectors.toList());
```

输出如下：`Gac`，`Cantaloupe`，`Hemi`，`Apollo`。

此外，如果您在理解这些函数式编程方法时遇到困难，我强烈建议您阅读我的另一本书，*Java 编码问题*，可从 Packt 获得（[`www.packtpub.com/programming/java-coding-problems`](https://www.packtpub.com/programming/java-coding-problems)）。该书包含两个关于 Java 函数式编程的全面章节，提供了详细的解释、图表和应用，对于深入研究这个主题非常有用。

## 编码挑战 7-过滤器（）函数

`filter()`函数是做什么的，为什么要使用它？

`filter()`函数是通过`Stream` API 提供的一种名为*filtering*的中间操作。它用于过滤满足某种条件的`Stream`元素。条件是通过`java.util.function.Predicate`函数指定的。这个谓词函数只是一个以`Object`作为参数并返回`boolean`的函数。

假设我们有以下整数`List`：

```java
List<Integer> ints
  = Arrays.asList(1, 2, -4, 0, 2, 0, -1, 14, 0, -1);
```

可以通过以下方式对此列表进行流处理并提取非零元素：

```java
List<Integer> result = ints.stream()
  .filter(i -> i != 0)
  .collect(Collectors.toList());
```

结果列表将包含以下元素：`1`，`2`，`-4`，`2`，`-1`，`14`，`-1`。

请注意，对于几个常见操作，Java `Stream` API 已经提供了即用即得的中间操作。例如，无需使用`filter()`和为以下操作定义`Predicate`：

+   `distinct()`: 从流中删除重复项

+   `skip(n)`: 跳过前`n`个元素

+   `limit(s)`: 将流截断为不超过`s`长度

+   `sorted()`: 根据自然顺序对流进行排序

+   `sorted(Comparator<? super T> comparator)`: 根据给定的`Comparator`对流进行排序

所有这些函数都内置在`Stream` API 中。

## 编码挑战 8-中间操作与终端操作

**问题**：中间操作和终端操作之间的主要区别是什么？

`Stream`，而终端操作产生除`Stream`之外的结果（例如，集合或标量值）。换句话说，中间操作允许我们在名为*管道*的查询类型中链接/调用多个操作。

中间操作直到调用终端操作才会执行。这意味着中间操作是懒惰的。主要是在实际需要某个给定处理的结果时执行它们。终端操作触发`Stream`的遍历并执行管道。

在中间操作中，我们有`map()`，`flatMap()`，`filter()`，`limit()`和`skip()`。在终端操作中，我们有`sum()`，`min()`，`max()`，`count()`和`collect()`。

## 编码挑战 9-peek()函数

`peek()`函数是做什么的，为什么要使用它？

`peek()`函数是通过`Stream` API 提供的一种名为*peeking*的中间操作。它允许我们查看`Stream`管道。主要是，`peek()`应该对当前元素执行某个*非干扰*的操作，并将元素转发到管道中的下一个操作。通常，这个操作包括在控制台上打印有意义的消息。换句话说，`peek()`是调试与流和 lambda 表达式处理相关问题的一个很好的选择。例如，想象一下，我们有以下地址列表：

```java
addresses.stream()
  .peek(p -> System.out.println("\tstream(): " + p))
  .filter(s -> s.startsWith("c"))
  .sorted()
  .peek(p -> System.out.println("\tsorted(): " + p))
  .collect(Collectors.toList());
```

重要的是要提到，即使`peek()`可以用于改变状态（修改流的数据源），它代表*看，但不要触摸*。通过`peek()`改变状态可能在并行流管道中成为真正的问题，因为修改操作可能在上游操作提供的任何时间和任何线程中被调用。因此，如果操作修改了共享状态，它负责提供所需的同步。

作为一个经验法则，在使用`peek()`来改变状态之前要三思。此外，要注意这种做法在开发人员中是有争议的，并且可以被归类为不良做法甚至反模式的范畴。

## 编码挑战 10 - 懒惰流

**问题**：说一个流是懒惰的是什么意思？

**解决方案**：说一个流是懒惰的意思是，流定义了一系列中间操作的管道，只有当管道遇到终端操作时才会执行。这个问题与本章的*编码挑战 8*有关。

## 编码挑战 11 - 函数式接口与常规接口

**问题**：函数式接口和常规接口之间的主要区别是什么？

**解决方案**：函数式接口和常规接口之间的主要区别在于，常规接口可以包含任意数量的抽象方法，而函数式接口只能有一个抽象方法。

您可以查阅本书的*编码挑战 2*以深入了解。

## 编码挑战 12 - 供应商与消费者

`Supplier`和`Consumer`？

`Supplier`和`Consumer`是两个内置的函数式接口。`Supplier`充当工厂方法或`new`关键字。换句话说，`Supplier`定义了一个名为`get()`的方法，不带参数并返回类型为`T`的对象。因此，`Supplier`对于*提供*某个值很有用。

另一方面，`Consumer`定义了一个名为`void accept(T t)`的方法。这个方法接受一个参数并返回`void`。`Consumer`接口*消耗*给定的值并对其应用一些操作。与其他函数式接口不同，`Consumer`可能会引起*副作用*。例如，`Consumer`可以用作设置方法。

## 编码挑战 13 - 谓词

`Predicate`？

`Predicate`是一个内置的函数式接口，它包含一个抽象方法，其签名为`boolean test(T object)`：

```java
@FunctionalInterface
public interface Predicate<T> {
  boolean test(T t);
  // default and static methods omitted for brevity
}
```

`test()`方法测试条件，如果满足条件则返回`true`，否则返回`false`。`Predicate`的常见用法是与`Stream<T> filter(Predicate<? super T> predicate)`方法一起过滤流中不需要的元素。

## 编码挑战 14 - findFirst()与 findAny()

`findFirst()`和`findAny()`？

`findFirst()`方法从流中返回第一个元素，特别适用于获取序列中的第一个元素。只要流有定义的顺序，它就会返回流中的第一个元素。如果没有遇到顺序，那么`findFirst()`会返回流中的任何元素。

另一方面，`findAny()`方法从流中返回任何元素。换句话说，它从流中返回一个任意（非确定性）的元素。`findAny()`方法忽略了遇到的顺序，在非并行操作中，它很可能返回第一个元素，但不能保证这一点。为了最大化性能，在并行操作中无法可靠地确定结果。

请注意，根据流的来源和中间操作，流可能有或可能没有定义的遇到顺序。

## 编码挑战 15 - 将数组转换为流

**问题**：如何将数组转换为流？

**解决方案**：将对象数组转换为流可以通过至少三种方式来完成，如下所示：

1.  第一种是通过`Arrays#stream()`：

```java
public static <T> Stream<T> toStream(T[] arr) {
  return Arrays.stream(arr);
}
```

1.  其次，我们可以使用`Stream#of()`：

```java
public static <T> Stream<T> toStream(T[] arr) {        
  return Stream.of(arr);
}
```

1.  最后一种技术是通过`List#stream()`：

```java
public static <T> Stream<T> toStream(T[] arr) {        
  return Arrays.asList(arr).stream();
}
```

将原始数组（例如整数）转换为流可以通过至少两种方式完成，如下：

1.  首先，通过`Arrays#stream()`：

```java
public static IntStream toStream(int[] arr) {       
  return Arrays.stream(arr);
}
```

1.  其次，通过使用`IntStream#of()`：

```java
public static IntStream toStream(int[] arr) {
  return IntStream.of(arr);
}
```

当然，对于长整型，您可以使用`LongStream`，对于双精度浮点数，您可以使用`DoubleStream`。

## 编码挑战 16-并行流

**问题**：什么是并行流？

**解决方案**：并行流是一种可以使用多个线程并行执行的流。例如，您可能需要过滤包含 1000 万个整数的流，以找到小于某个值的整数。您可以使用并行流来代替使用单个线程顺序遍历流。这意味着多个线程将同时在流的不同部分搜索这些整数，然后将结果合并。

## 编码挑战 17-方法引用

**问题**：什么是方法引用？

`::`，然后在其后提供方法的名称。我们有以下引用：

+   对静态方法的方法引用：*Class*::*staticMethod*（例如，`Math::max`等同于`Math.max(`*x*`,` *y*`)`）

+   对构造函数的方法引用：*Class*::*new*（例如，`AtomicInteger::new`等同于`new AtomicInteger(`*x*`)`）

+   对实例方法的方法引用：*object*::*instanceMethod*（`System.out::println`等同于`System.out.println(`*foo*`)`）

+   对类类型的实例方法的方法引用：*Class*::*instanceMethod*（`String::length`等同于`str.length()`）

## 编码挑战 18-默认方法

**问题**：什么是默认方法？

**解决方案**：默认方法主要是在 Java 8 中添加的，以提供对接口的支持，使其可以超越抽象合同（即仅包含抽象方法）。这个功能对于编写库并希望以兼容的方式发展 API 的人非常有用。通过默认方法，接口可以在不破坏现有实现的情况下进行丰富。

默认方法直接在接口中实现，并且通过`default`关键字识别。例如，以下接口定义了一个名为`area()`的抽象方法和一个名为`perimeter()`的默认方法：

```java
public interface Polygon {
  public double area();
  default double perimeter(double... segments) {
    return Arrays.stream(segments)
      .sum();
  }
}
```

由于`Polygon`有一个抽象方法，它也是一个函数接口。因此，它可以用`@FunctionalInterface`注解。

## 编码挑战 19-迭代器与 Spliterator

`Iterator`和`Spliterator`？

`Iterator`是为`Collection`API 创建的，而`Spliterator`是为`Stream`API 创建的。

通过分析它们的名称，我们注意到*Spliterator* = *Splittable Iterator*。因此，`Spliterator`可以分割给定的源并且也可以迭代它。分割是用于并行处理的。换句话说，`Iterator`可以顺序迭代`Collection`中的元素，而`Spliterator`可以并行或顺序地迭代流的元素。

`Iterator`只能通过`hasNext()`/`next()`遍历集合的元素，因为它没有大小。另一方面，`Spliterator`可以通过`estimateSize()`近似地提供集合的大小，也可以通过`getExactSizeIfKnown()`准确地提供集合的大小。

`Spliterator`可以使用多个标志来内部禁用不必要的操作（例如，`CONCURRENT`，`DISTINCT`和`IMMUTABLE`）。`Iterator`没有这样的标志。

最后，您可以按以下方式围绕`Iterator`创建一个`Spliterator`：

```java
Spliterators.spliteratorUnknownSize(
  your_Iterator, your_Properties);
```

在书籍*Java 编码问题*（[`www.amazon.com/gp/product/B07Y9BPV4W/`](https://www.amazon.com/gp/product/B07Y9BPV4W/)）中，您可以找到有关此主题的更多详细信息，包括编写自定义`Spliterator`的完整指南。

## 编码挑战 20-Optional

`Optional`类？

`Optional`类是在 Java 8 中引入的，主要目的是减轻/避免`NullPointerException`。Java 语言架构师 Brian Goetz 的定义如下：

Optional 旨在为库方法的返回类型提供有限的机制，在需要清晰表示没有结果的情况下，使用 null 很可能会导致错误。

简而言之，您可以将`Optional`视为一个单值容器，它可以包含一个值或者为空。例如，一个空的`Optional`看起来像这样：

```java
Optional<User> userOptional = Optional.empty();
```

一个非空的`Optional`看起来像这样：

```java
User user = new User();
Optional<User> userOptional = Optional.of(user);
```

在《Java 编程问题》（[`www.amazon.com/gp/product/B07Y9BPV4W/`](https://www.amazon.com/gp/product/B07Y9BPV4W/)）中，您可以找到一个完整的章节专门讨论了使用`Optional`的最佳实践。这是任何 Java 开发人员必读的章节。

## 编码挑战 21 - String::valueOf

`String::valueOf`的意思是什么？

`String::valueOf`是对`String`类的`valueOf`静态方法的方法引用。考虑阅读《编码挑战 17》以获取更多关于这个的信息。

# 总结

在本章中，我们涵盖了关于 Java 中函数式编程的几个热门话题。虽然这个主题非常广泛，有很多专门的书籍，但在这里涵盖的问题应该足以通过涵盖 Java 8 语言主要特性的常规 Java 面试。

在下一章中，我们将讨论与扩展相关的问题。
