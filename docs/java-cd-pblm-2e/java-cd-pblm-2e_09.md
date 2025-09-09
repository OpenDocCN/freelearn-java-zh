

# 第九章：函数式风格编程 – 扩展 API

本章包括涵盖广泛函数式编程主题的 24 个问题。我们将首先介绍 JDK 16 的 `mapMulti()` 操作，然后继续讨论一些使用谓词（`Predicate`）、函数和收集器的练习问题。

如果你没有 Java 函数式编程的背景，那么我强烈建议你推迟学习本章，直到你花了一些时间熟悉它。你可以考虑阅读《Java 编程问题》第一版中的第八章和第九章。

在本章结束时，你将精通 Java 中的函数式编程。

# 问题

使用以下问题来测试你在 Java 函数式编程中的编程能力。我强烈建议你在查看解决方案并下载示例程序之前尝试每个问题：

1.  **使用 mapMulti()**：解释并举例说明 JDK 16 的 `mapMulti()`。提供简要介绍，解释它与 `flatMap()` 的工作方式，并指出何时 `mapMulti()` 是一个好的选择。

1.  **将自定义代码流式传输到 map 中**：想象一个类，它处理一些博客文章。每篇文章都有一个唯一的整数 ID，文章有几个属性，包括其标签。每篇文章的标签实际上是以标签字符串的形式表示的，标签之间用井号（`#`）分隔。每当我们需要给定文章的标签列表时，我们都可以调用 `allTags()` 辅助方法。我们的目标是编写一个流管道，从这个标签列表中提取一个 `Map<String, List<Integer>>`，其中每个标签（键）对应文章列表（值）。

1.  **展示方法引用与 lambda 的区别**：编写一段相关代码片段，以突出方法引用与等效 lambda 表达式之间的行为差异。

1.  **通过 Supplier/Consumer 捕获 lambda 的惰性**：编写一个 Java 程序，突出 `Supplier`/`Consumer` 的工作方式。在此上下文中，指出 lambda 的惰性特征。

1.  **重构代码以添加 lambda 惰性**：提供一个简单的示例，通过函数式代码重构一段命令式代码。

1.  **编写一个 Function<String, T> 来解析数据**：想象一个给定的文本（`test, a, 1, 4, 5, 0xf5, 0x5, 4.5d, 6, 5.6, 50000, 345, 4.0f, 6$3, 2$1.1, 5.5, 6.7, 8, a11, 3e+1, -11199, 55`）。编写一个应用程序，公开一个能够解析此文本并提取双精度浮点数、整数、长整数等的 `Function<String, T>`。

1.  **在 Stream 的过滤器中组合谓词**：编写几个示例，突出复合谓词在过滤器中的使用。

1.  **使用 Streams 过滤嵌套集合**：想象你有两个嵌套集合。提供几个流管道示例来从内部集合中过滤数据。

1.  **使用 BiPredicate**：举例说明 `BiPredicate` 的用法。

1.  **为自定义模型构建动态谓词**：编写一个应用程序，能够根据一些简单的输入动态生成谓词（`Predicate`）。

1.  **从自定义条件映射构建动态谓词**：考虑有一个条件映射（映射的键是一个字段，映射的值是该字段的预期值）。在这种情况下，编写一个应用程序，动态生成适当的谓词。

1.  **在谓词中使用日志记录**：编写一个自定义解决方案，允许我们在谓词中记录失败。

1.  **通过 containsAll()和 containsAny()扩展 Stream**：提供一个解决方案，通过名为`containsAll()`和`containsAny()`的两个最终操作扩展 Java Stream API。

1.  **通过 removeAll()和 retainAll()扩展 Stream**：提供一个解决方案，通过名为`removeAll()`和`retainAll()`的两个最终操作扩展 Java Stream API。

1.  **介绍流比较器**：提供使用流比较器的详细说明（包括示例）。

1.  **排序映射**：编写几个代码片段来突出显示排序映射的不同用例。

1.  **过滤映射**：编写几个代码片段来突出显示过滤映射的不同用例。

1.  **通过 Collector.of()创建自定义收集器**：通过`Collector.of()` API 编写一组任意选择的自定义收集器。

1.  **在 lambda 表达式中抛出检查型异常**：提供一个允许我们从 lambda 表达式中抛出检查型异常的技巧。

1.  **为 Stream API 实现 distinctBy()**：编写一个 Java 应用程序，实现`distinctBy()`流中间操作。这类似于内置的`distinct()`，但它允许我们通过给定的属性/字段过滤不同的元素。

1.  **编写一个自定义收集器，该收集器接受/跳过给定数量的元素**：提供一个自定义收集器，允许我们仅收集前*n*个元素。此外，提供一个自定义收集器，跳过前*n*个元素并收集其余元素。

1.  **实现一个接受五个（或任何其他任意数量）参数的函数**：编写并使用一个代表`java.util.function.Function`特殊化的五个参数的功能接口。

1.  **实现一个接受五个（或任何其他任意数量）参数的消费者**：编写并使用一个代表`java.util.function.Consumer`特殊化的五个参数的功能接口。

1.  **部分应用一个函数**：编写一个代表`java.util.function.Function`特殊化的*n*-元功能接口。此外，此接口应提供支持（即提供必要的`default`方法），以便仅应用*n*-1，*n*-2，*n*-3，…，1 个参数。

以下几节描述了先前问题的解决方案。请记住，通常没有解决特定问题的唯一正确方法。此外，请记住，这里所示的解释仅包括解决这些问题所需的最有趣和最重要的细节。下载示例解决方案以查看更多细节，并实验程序，请访问 [`github.com/PacktPublishing/Java-Coding-Problems-Second-Edition/tree/main/Chapter09`](https://github.com/PacktPublishing/Java-Coding-Problems-Second-Edition/tree/main/Chapter09)。

# 185. 使用 mapMulti()

从 JDK 16 开始，Stream API 通过一个新的中间操作 `mapMulti()` 得到了增强。这个操作在 `Stream` 接口中由以下 `default` 方法表示：

```java
default <R> Stream<R> mapMulti (
  BiConsumer<? super T, ? super Consumer<R>> mapper) 
```

让我们采用以例学例的方法，考虑下一个经典示例，它使用 `filter()` 和 `map()` 的组合来过滤偶数整数并加倍它们的值：

```java
List<Integer> integers = List.of(3, 2, 5, 6, 7, 8);
List<Integer> evenDoubledClassic = integers.stream()
  .filter(i -> i % 2 == 0)
  .map(i -> i * 2)
  .collect(toList()); 
```

可以通过以下方式使用 `mapMulti()` 获得相同的结果：

```java
List<Integer> evenDoubledMM = integers.stream()
  .<Integer>mapMulti((i, consumer) -> {
     if (i % 2 == 0) {
       consumer.accept(i * 2);
     }
  })
  .collect(toList()); 
```

因此，我们不是使用两个中间操作，而是只使用了一个，即 `mapMulti()`。`filter()` 的作用被一个 `if` 语句所取代，而 `map()` 的作用则在 `accept()` 方法中完成。这次，我们通过 `mapper` 过滤了偶数并加倍了它们的值，其中 `mapper` 是一个 `BiConsumer<? super T, ? super Consumer<R>>`。这个双函数应用于每个整数（每个流元素），并且只有偶数整数被传递给消费者。这个消费者充当一个缓冲区，简单地向下传递（在流管道中）接收到的元素。`mapper.accept(R r)` 可以被调用任意次数，这意味着对于给定的流元素，我们可以产生我们需要的任意数量的输出元素。在先前的例子中，我们有一个一对一映射（当 `i % 2 == 0` 被评估为 `false` 时）和一个一对应多映射（当 `i % 2 == 0` 被评估为 `true` 时）。

**重要提示**

更精确地说，`mapMulti()` 接收一个元素输入流，并输出另一个包含零个、较少、相同或更多元素（这些元素可以是未更改的或被其他元素替换）的流。这意味着输入流中的每个元素都可以通过一对一、一对应零或一对应多映射。

你注意到返回值上应用了 `<Integer>mapMulti(…)` 类型见证了吗？没有这个类型见证，代码将无法编译，因为编译器无法确定 `R` 的正确类型。这是使用 `mapMulti()` 的不足之处，因此我们必须付出这个代价。

对于原始类型（`double`、`long` 和 `int`），我们有 `mapMultiToDouble()`、`mapMultiToLong()` 和 `mapMultiToInt()`，它们分别返回 `DoubleStream`、`LongStream` 和 `IntStream`。例如，如果我们计划求和偶数整数，那么使用 `mapMultiToInt()` 比使用 `mapMulti()` 更好，因为我们可以跳过类型见证，只使用原始的 `int`：

```java
int evenDoubledAndSumMM = integers.stream()
  .mapMultiToInt((i, consumer) -> {
     if (i % 2 == 0) {
       consumer.accept(i * 2);
     }
  })
  .sum(); 
```

另一方面，无论何时你需要 `Stream<T>` 而不是 `Double`/`Long`/`IntStream`，你仍然需要依赖于 `mapToObj()` 或 `boxed()`：

```java
List<Integer> evenDoubledMM = integers.stream()
  .mapMultiToInt((i, consumer) -> {
    if (i % 2 == 0) {
      consumer.accept(i * 2);
    }
  })
  .mapToObj(i -> i) // or, .boxed()
  .collect(toList()); 
```

一旦熟悉了 `mapMulti()`，你就会开始意识到它与众所周知的 `flatMap()` 非常相似，后者用于展开嵌套的 `Stream<Stream<R>>` 模型。让我们考虑以下一对一关系：

```java
public class Author {
  private final String name;
  private final List<Book> books;
  ...
}
public class Book {

  private final String title;
  private final LocalDate published;
  ...
} 
```

每个 `Author` 都有一系列书籍。因此，一个 `List<Author>`（可能成为 `Stream<Author>` 的候选）将为每个 `Author` 嵌套一个 `List<Book>`（可能成为嵌套 `Stream<Book>` 的候选）。此外，我们还有以下简单的映射 `author` 和单个 `book` 的模型：

```java
public class Bookshelf {
  private final String author;
  private final String book;
  ...
} 
```

在函数式编程中，将一对一多模型映射到扁平的 `Bookshelf` 模型是使用 `flatMap()` 的经典场景，如下所示：

```java
List<Bookshelf> bookshelfClassic = authors.stream()
  .flatMap(
    author -> author.getBooks()
                    .stream()
                    .map(book -> new Bookshelf(
                       author.getName(), book.getTitle()))
  ).collect(Collectors.toList()); 
```

`flatMap()` 的问题在于我们需要为每位作者创建一个新的中间流（对于大量作者，这可能会成为性能惩罚），然后我们才能应用 `map()` 操作。使用 `mapMulti()`，我们不需要这些中间流，映射过程非常直接：

```java
List<Bookshelf> bookshelfMM = authors.stream()
  .<Bookshelf>mapMulti((author, consumer) -> {
     for (Book book : author.getBooks()) {
       consumer.accept(new Bookshelf(
         author.getName(), book.getTitle()));
     }
  })
  .collect(Collectors.toList()); 
```

这是一个一对一的映射。对于每位作者，消费者缓冲与作者书籍数量相等的 `Bookshelf` 实例。这些实例在下游中展开，最终通过 `toList()` 收集器收集到一个 `List<Bookshelf>` 中。

而且这条路径带我们来到了关于 `mapMulti()` 的另一个重要提示。

**重要提示**

当我们必须替换流中的少量元素时，`mapMulti()` 中间操作非常有用。这一陈述在官方文档中表述如下：“*当用少量（可能为零）的元素替换每个流元素时。”

接下来，查看基于 `flatMap()` 的这个示例：

```java
List<Bookshelf> bookshelfGt2005Classic = authors.stream()
  .flatMap(
    author -> author.getBooks()
      .stream()
      .filter(book -> book.getPublished().getYear() > 2005)
      .map(book -> new Bookshelf(
         author.getName(), book.getTitle()))
  ).collect(Collectors.toList()); 
```

这个例子非常适合使用 `mapMulti()`。一位作者有相对较少的书籍，我们对它们进行过滤。所以基本上，我们将每个流元素替换为少量（可能为零）的元素：

```java
List<Bookshelf> bookshelfGt2005MM = authors.stream()
  .<Bookshelf>mapMulti((author, consumer) -> {
    for (Book book : author.getBooks()) {
      if (book.getPublished().getYear() > 2005) {
        consumer.accept(new Bookshelf(
          author.getName(), book.getTitle()));
      }
    }
  })
  .collect(Collectors.toList()); 
```

这比使用 `flatMap()` 更好，因为我们减少了中间操作的数量（不再有 `filter()` 调用），并且避免了中间流。这也更易于阅读。

`mapMulti()` 的另一个用例如下。

**重要提示**

当使用命令式方法生成结果元素比以 `Stream` 形式返回它们更容易时，`mapMulti()` 操作也非常有用。这一陈述在官方文档中表述如下：“*当使用命令式方法生成结果元素比以* `Stream` “*形式返回它们更容易时。”

想象一下，我们已经在 `Author` 类中添加了以下方法：

```java
public void bookshelfGt2005(Consumer<Bookshelf> consumer) {
  for (Book book : this.getBooks()) {
    if (book.getPublished().getYear() > 2005) {
      consumer.accept(new Bookshelf(
        this.getName(), book.getTitle()));
    }
  }
} 
```

现在，我们可以简单地使用 `mapMulti()` 来获取 `List<Bookshelf>`，如下所示：

```java
List<Bookshelf> bookshelfGt2005MM = authors.stream()
  .<Bookshelf>mapMulti(Author::bookshelfGt2005)
  .collect(Collectors.toList()); 
```

这有多酷？！在下一个问题中，我们将在另一个场景中使用 `mapMulti()`。

# 186. 将自定义代码流式传输以映射

假设我们有一个以下遗留类：

```java
public class Post {

  private final int id;
  private final String title;
  private final String tags;
  public Post(int id, String title, String tags) {
    this.id = id;
    this.title = title;
    this.tags = tags;
  }
  ...
  public static List<String> allTags(Post post) {

    return Arrays.asList(post.getTags().split("#"));
  }
} 
```

因此，我们有一个类，它塑造了一些博客文章。每篇文章都有几个属性，包括其标签。每篇文章的标签实际上是以哈希标签（`#`）分隔的标签字符串表示的。每当我们需要给定文章的标签列表时，我们可以调用`allTags()`辅助函数。例如，以下是一系列文章及其标签：

```java
List<Post> posts = List.of(
  new Post(1, "Running jOOQ", "#database #sql #rdbms"),
  new Post(2, "I/O files in Java", "#io #storage #rdbms"),
  new Post(3, "Hibernate Course", "#jpa #database #rdbms"),
  new Post(4, "Hooking Java Sockets", "#io #network"),
  new Post(5, "Analysing JDBC transactions", "#jdbc #rdbms")
); 
```

我们的目标是从这个列表中提取一个`Map<String, List<Integer>>`，其中包含每个标签（键）的帖子列表（值）。例如，对于标签`#database`，我们有文章 1 和 3；对于标签`#rdbms`，我们有文章 1、2、3 和 5，等等。

在函数式编程中完成这项任务可以通过`flatMap()`和`groupingBy()`来实现。简而言之，`flatMap()`对于展开嵌套的`Stream<Stream<R>>`模型很有用，而`groupingBy()`是一个用于按某些逻辑或属性在 map 中分组数据的收集器。

我们需要`flatMap()`，因为我们有一个`List<Post>`，对于每个`Post`，通过`allTags()`嵌套一个`List<String>`（如果我们简单地调用`stream()`，那么我们得到的是一个`Stream<Stream<R>>`）。在展开后，我们将每个标签包装在`Map.Entry<String, Integer>`中。最后，我们将这些条目按标签分组到一个`Map`中，如下所示：

```java
Map<String, List<Integer>> result = posts.stream()
  .flatMap(post -> Post.allTags(post).stream()
  .map(t -> entry(t, post.getId())))
  .collect(groupingBy(Entry::getKey,
              mapping(Entry::getValue, toList()))); 
```

然而，根据前面的问题，我们知道从 JDK 16 开始，我们可以使用`mapMulti()`。因此，我们可以将之前的代码片段重写如下：

```java
Map<String, List<Integer>> resultMulti = posts.stream()
  .<Map.Entry<String, Integer>>mapMulti((post, consumer) -> {
      for (String tag : Post.allTags(post)) {
             consumer.accept(entry(tag, post.getId()));
      }
  })
  .collect(groupingBy(Entry::getKey,
              mapping(Entry::getValue, toList()))); 
```

这次，我们保存了`map()`中间操作和中间流。

# 187. 演示方法引用与 lambda 的区别

你是否曾经编写过一个 lambda 表达式，而你的 IDE 建议你用方法引用替换它？你很可能已经这样做了！我相信你更喜欢遵循替换，因为*名称很重要*，方法引用通常比 lambda 表达式更易读。虽然这是一个主观问题，但我很确信你会同意，在方法中提取长 lambda 表达式并通过方法引用使用/重用它们是一种普遍接受的良好实践。

然而，除了某些神秘的 JVM 内部表示之外，它们的行为是否相同？lambda 表达式和方法引用之间是否有任何差异可能会影响代码的行为？

好吧，让我们假设我们有一个以下简单的类：

```java
public class Printer {

  Printer() {
    System.out.println("Reset printer ...");
  }

  public static void printNoReset() {
    System.out.println(
      "Printing (no reset) ..." + Printer.class.hashCode());
  }

  public void printReset() {
    System.out.println("Printing (with reset) ..." 
      + Printer.class.hashCode());
  }
} 
```

如果我们假设`p1`是一个方法引用，而`p2`是对应的 lambda 表达式，那么我们可以执行以下调用：

```java
System.out.print("p1:");p1.run();
System.out.print("p1:");p1.run();
System.out.print("p2:");p2.run();
System.out.print("p2:");p2.run();
System.out.print("p1:");p1.run();
System.out.print("p2:");p2.run(); 
```

接下来，让我们看看使用`p1`和`p2`的两个场景。

## 场景 1：调用 printReset()

在第一种情况下，我们通过`p1`和`p2`调用`printReset()`，如下所示：

```java
Runnable p1 = new Printer()::printReset;
Runnable p2 = () -> new Printer().printReset(); 
```

如果我们现在运行代码，那么我们会得到以下输出（由`Printer`构造函数生成的信息）：

```java
Reset printer ... 
```

这种输出是由方法引用`p1`引起的。即使我们没有调用`run()`方法，`Printer`构造函数也会立即被调用。因为`p2`（lambda 表达式）是惰性的，所以只有在调用`run()`方法时才会调用`Printer`构造函数。

进一步，我们为`p1`和`p2`调用`run()`调用链。输出将是：

```java
p1:Printing (with reset) ...1159190947
p1:Printing (with reset) ...1159190947
p2:Reset printer ...
Printing (with reset) ...1159190947
p2:Reset printer ...
Printing (with reset) ...1159190947
p1:Printing (with reset) ...1159190947
p2:Reset printer ...
Printing (with reset) ...1159190947 
```

如果我们分析这个输出，我们可以看到每次 lambda (`p2.run()`) 执行时都会调用 `Printer` 构造函数。另一方面，对于方法引用（`p1.run()`），`Printer` 构造函数不会被调用。它只在一个地方被调用，即在 `p1` 声明时。所以 `p1` 打印时不会重置打印机。

## 场景 2：调用静态 printNoReset()

接下来，让我们调用静态方法 `printNoReset()`：

```java
Runnable p1 = Printer::printNoReset;
Runnable p2 = () -> Printer.printNoReset(); 
```

如果我们立即运行代码，那么什么也不会发生（没有输出）。接下来，我们启动 `run()` 调用，我们得到以下输出：

```java
p1:Printing (no reset) ...149928006
p1:Printing (no reset) ...149928006
p2:Printing (no reset) ...149928006
p2:Printing (no reset) ...149928006
p1:Printing (no reset) ...149928006
p2:Printing (no reset) ...149928006 
```

`printNoReset()` 是一个静态方法，所以不会调用 `Printer` 构造函数。我们可以互换使用 `p1` 或 `p2` 而不会有任何行为上的差异。所以，在这种情况下，这只是一种偏好。

## 结论

当调用非静态方法时，方法引用和 lambda 之间有一个主要区别。方法引用立即且仅调用一次构造函数（在方法调用（`run()`）时，构造函数不会被调用）。另一方面，lambda 是懒加载的。它们仅在方法调用时调用构造函数，并且在每个这样的调用（`run()`）中。

# 188. 通过 Supplier/Consumer 捕获 lambda 懒加载

`java.util.function.Supplier` 是一个可以通过其 `get()` 方法提供结果的函数式接口。`java.util.function.Consumer` 是另一个可以通过其 `accept()` 方法消耗通过其提供的参数的函数式接口。它不返回任何结果（`void`）。这两个函数式接口都是懒加载的，因此分析和使用它们的代码并不容易，尤其是在代码片段同时使用这两个接口时。让我们试一试！

考虑以下简单的类：

```java
static class Counter {
  static int c;
  public static int count() {
    System.out.println("Incrementing c from " 
      + c + " to " + (c + 1));
    return c++; 
  }
} 
```

然后让我们编写以下 `Supplier` 和 `Consumer`：

```java
Supplier<Integer> supplier = () -> Counter.count();
Consumer<Integer> consumer = c -> {
  c = c + Counter.count(); 
  System.out.println("Consumer: " + c ); 
}; 
```

那么，到目前为止，`Counter.c` 的值是多少？

```java
System.out.println("Counter: " + Counter.c); // 0 
```

正确答案是，`Counter.c` 是 0。供应商和消费者都是懒加载的，所以在它们的声明中都没有调用 `get()` 或 `accept()` 方法。`Counter.count()` 没有被调用，所以 `Counter.c` 没有增加。

这里有一个棘手的问题……现在怎么样？

```java
System.out.println("Supplier: " + supplier.get()); // 0 
```

我们知道通过调用 `supplier.get()`，我们触发了 `Counter.count()` 的执行，`Counter.c` 应该增加并变为 1。然而，`supplier.get()` 将返回 0。

解释位于 `count()` 方法的第 `return c++;` 行。当我们写 `c++` 时，我们使用后增量操作，因此我们在我们的语句中使用 `c` 的当前值（在这种情况下，`return`），然后我们将其增加 1。这意味着 `supplier.get()` 返回 `c` 的值为 0，而增量操作发生在 `return` 之后，此时 `Counter.c` 为 1：

```java
System.out.println("Counter: " + Counter.c); // 1 
```

如果我们从后增量（`c++`）切换到前增量（`++c`），那么 `supplier.get()` 将返回 1，这将与 `Counter.c` 保持同步。这是因为增量操作在我们使用值之前发生（在这里，`return`）。

好的，到目前为止，我们知道`Counter.c`等于 1。接下来，让我们调用消费者并传入`Counter.c`的值：

```java
consumer.accept(Counter.c); 
```

通过这个调用，我们在以下计算和显示中推入`Counter.c`（它是 1）：

```java
c -> {
  c = c + Counter.count(); 
  System.out.println("Consumer: " + c ); 
} // Consumer: 2 
```

因此`c = c + Counter.count()`可以看作`Counter.c = Counter.c + Counter.count()`，这相当于 1 = 1 + `Counter.count()`，所以 1 = 1 + 1。输出将是`Consumer: 2`。这次，`Counter.c`也是 2（记住后增量效应）：

```java
System.out.println("Counter: " + Counter.c); // 2 
```

接下来，让我们调用供应商：

```java
System.out.println("Supplier: " + supplier.get()); // 2 
```

我们知道`get()`将接收`c`的当前值，它是 2。之后，`Counter.c`变为 3：

```java
System.out.println("Counter: " + Counter.c); // 3 
```

我们可以永远这样继续下去，但我认为你已经了解了`Supplier`和`Consumer`函数式接口的工作方式。

# 189. 重新整理代码以添加 lambda 惰性

在这个问题中，让我们进行一次重构会话，将非功能代码转换为功能代码。我们从以下给定的代码开始——关于应用程序依赖项的简单类映射信息：

```java
public class ApplicationDependency {

  private final long id;
  private final String name;
  private String dependencies;
  public ApplicationDependency(long id, String name) {
    this.id = id;
    this.name = name;
  }
  public long getId() {
    return id;
  }
  public String getName() {
    return name;
  }   

**public** **String** **getDependencies****()** **{**
**return** **dependencies;**
 **}**  

  private void downloadDependencies() {

    dependencies = "list of dependencies 
      downloaded from repository " + Math.random();
  }    
} 
```

为什么我们强调了`getDependencies()`方法？因为这是应用程序中存在功能障碍的点。更准确地说，以下类需要应用程序的依赖项以便相应地处理它们：

```java
public class DependencyManager {

  private Map<Long,String> apps = new HashMap<>();

  public void processDependencies(ApplicationDependency appd){

    System.out.println();
    System.out.println("Processing app: " + appd.getName());
    System.out.println("Dependencies: " 
      + **appd.getDependencies()**);

    apps.put(appd.getId(),**appd.getDependencies()**); 
  }    
} 
```

这个类依赖于`ApplicationDependency.getDependecies()`方法，它只返回`null`（`dependencies`字段的默认值）。由于没有调用`downloadDependecies()`方法，预期的应用程序依赖项没有被下载。很可能会有一位代码审查员指出这个问题并创建一个工单来修复它。

## 以命令式方式修复

一个可能的修复方法如下（在`ApplicationDependency`中）：

```java
public class ApplicationDependency {

  **private****String****dependencies****=** **downloadDependencies();**
  ...
  public String getDependencies() {

    return dependencies;
  }
  ...
  private String downloadDependencies() {

    return "list of dependencies downloaded from repository " 
      + Math.random();
  }  
} 
```

在`dependencies`初始化时调用`downloadDependencies()`肯定可以修复加载依赖项的问题。当`DependencyManager`调用`getDependencies()`时，它将能够访问已下载的依赖项。然而，这是一个好方法吗？我的意思是，下载依赖项是一个昂贵的操作，我们每次创建`ApplicationDependency`实例时都会这样做。如果`getDependencies()`方法从未被调用，那么这个昂贵的操作就没有得到回报。

因此，一个更好的方法是在`getDependencies()`实际调用之前推迟应用程序依赖项的下载：

```java
public class ApplicationDependency {
  private String dependencies;
  ...
  public String getDependencies() {

    **downloadDependencies();** 

    return dependencies;
  }  
  ...
  private void downloadDependencies() {

    dependencies = "list of dependencies 
      downloaded from repository " + Math.random();
  }    
} 
```

这比之前好，但还不是最佳方法！这次，每次调用`getDependencies()`方法时都会下载应用程序的依赖项。幸运的是，有一个快速的修复方法。我们只需要在执行下载之前添加一个`null`检查：

```java
public String getDependencies() {

**if** **(dependencies ==** **null****) {**
 **downloadDependencies();**
 **}**

  return dependencies;
} 
```

完成！现在，应用程序的依赖项只在第一次调用`getDependencies()`方法时下载。这个命令式解决方案效果很好，并且通过了代码审查。

## 以函数式方式修复

关于以函数式编程的方式提供这个修复方案怎么样？实际上，我们想要的只是惰性下载应用程序的依赖项。由于惰性是函数式编程的专长，我们现在已经熟悉了`Supplier`（参见前一个问题），我们可以从以下开始：

```java
public class ApplicationDependency {

**private****final** **Supplier<String> dependencies** 
 **=** **this****::downloadDependencies;** 
 **...**
  public String getDependencies() {
    **return** **dependencies.get();**
  }   
  ...
  private **String** downloadDependencies() {

    return "list of dependencies downloaded from repository " 
     + Math.random();
  } 
} 
```

首先，我们定义了一个调用`downloadDependencies()`方法的`Supplier`。我们知道`Supplier`是惰性的，所以直到其`get()`方法被明确调用之前，不会发生任何事情。

其次，我们已修改`getDependencies()`方法，使其返回`dependencies.get()`。因此，我们将应用程序依赖项的下载延迟到它们被明确需要时。

第三，我们将`downloadDependencies()`方法的返回类型从`void`修改为`String`。这是为了`Supplier.get()`。

这是一个很好的修复方案，但它有一个严重的缺点。我们失去了缓存！现在，依赖项将在每次`getDependencies()`调用时下载。

我们可以通过*记忆化*（[`en.wikipedia.org/wiki/Memoization`](https://en.wikipedia.org/wiki/Memoization)）来避免这个问题。这个概念在*《Java 完整编码面试指南》*的第*第八章*中也有详细说明。简而言之，记忆化是一种通过缓存可重用结果来避免重复工作的技术。

记忆化是一种在动态规划中常用到的技术，但没有任何限制或限制。例如，我们可以在函数式编程中应用它。在我们的特定情况下，我们首先定义了一个扩展`Supplier`接口的功能接口（或者，如果你觉得更简单，可以直接使用`Supplier`）：

```java
@FunctionalInterface
public interface FSupplier<R> extends Supplier<R> {} 
```

接下来，我们提供了一个`FSupplier`的实现，它基本上缓存了未查看的结果，并从缓存中提供已查看的结果：

```java
public class Memoize {
  private final static Object UNDEFINED = new Object();
  public static <T> FSupplier<T> supplier(
    final Supplier<T> supplier) {

    AtomicReference cache = new AtomicReference<>(UNDEFINED);

    return () -> { 

      Object value = cache.get(); 

      if (value == UNDEFINED) { 

        synchronized (cache) {

          if (cache.get() == UNDEFINED) {

            System.out.println("Caching: " + supplier.get());
            value = supplier.get();
            cache.set(value);
          }
        }
      }

      return (T) value;
    };
  }
} 
```

最后，我们将我们的初始`Supplier`替换为`FSupplier`，如下所示：

```java
private final Supplier<String> dependencies 
  = Memoize.supplier(this::downloadDependencies); 
```

完成！我们的函数式方法利用了`Supplier`的惰性并可以缓存结果。

# 190. 编写一个`Function<String, T>`来解析数据

假设我们有以下文本：

```java
String text = """
  test, a, 1, 4, 5, 0xf5, 0x5, 4.5d, 6, 5.6, 50000, 345, 
  4.0f, 6$3, 2$1.1, 5.5, 6.7, 8, a11, 3e+1, -11199, 55 
  """; 
```

目标是从这段文本中提取出数字。根据给定的场景，我们可能只需要整数，或者只需要双精度浮点数，等等。有时，我们可能需要在提取之前进行一些文本替换（例如，我们可能想要将`xf`字符替换为点，`0xf5 = 0.5`）。

解决这个问题的可能方法之一是编写一个方法（让我们称它为`parseText()`），它接受一个`Function<String, T>`作为参数。`Function<String, T>`给我们提供了灵活性，可以塑造以下任何一种：

```java
List<Integer> integerValues 
  = parseText(text, Integer::valueOf);
List<Double> doubleValues 
  = parseText(text, Double::valueOf);
...
List<Double> moreDoubleValues 
  = parseText(text, t -> Double.valueOf(t.replaceAll(
      "\\$", "").replaceAll("xf", ".").replaceAll("x", "."))); 
```

`parseText()`应该执行几个步骤，直到达到最终结果。它的签名可以是以下这样：

```java
public static <T> List<T> parseText(
    String text, Function<String, T> func) {
  ...
} 
```

首先，我们必须通过逗号分隔符拆分接收到的文本，并从`String[]`中提取项目。这样，我们就能够访问文本中的每个项目。

其次，我们可以流式传输`String[]`并过滤掉任何空项。

第三，我们可以调用`Function.apply()`将给定的函数应用于每个项目（例如，应用`Double::valueOf`）。这可以通过中间操作`map()`来完成。由于一些项目可能是无效的数字，我们必须捕获并忽略任何`Exception`（吞咽这样的异常是不良的做法，但在这个情况下，实际上没有其他事情可做）。对于任何无效的数字，我们简单地返回`null`。

第四，我们过滤掉所有`null`值。这意味着剩余的流只包含通过`Function.apply()`过滤的数字。

第五，我们将流收集到一个`List`中并返回它。

将这五个步骤组合起来，将得到以下代码：

```java
public static <T> List<T> parseText(
    String text, Function<String, T> func) {
  return Arrays.stream(text.split(",")) // step 1 and 2
    .filter(s -> !s.isEmpty())
    .map(s -> {
       try {
         return func.apply(s.trim());   // step 3
       } catch (Exception e) {}
       return null;
    })
    .filter(Objects::nonNull)           // step 4
    .collect(Collectors.toList());      // step 5
} 
```

完成！您可以用这个例子解决一系列类似的问题。

# 191. 在 Stream 的过滤器中组合谓词

一个谓词（基本上，一个条件）可以通过`java.util.function.Predicate`函数式接口建模为一个布尔值函数。它的函数方法是名为`test(T t)`并返回一个`boolean`。

在流管道中应用谓词可以通过几个流中间操作来完成，但我们这里只对`filter(Predicate p)`操作感兴趣。例如，让我们考虑以下类：

```java
public class Car {
  private final String brand;
  private final String fuel;
  private final int horsepower;
  public Car(String brand, String fuel, int horsepower) {
    this.brand = brand;
    this.fuel = fuel;
    this.horsepower = horsepower;
  }

  // getters, equals(), hashCode(), toString()
} 
```

如果我们有一个`List<Car>`并且我们想要表达一个过滤条件，产生所有雪佛兰汽车，那么我们可以先定义适当的`Predicate`：

```java
Predicate<Car> pChevrolets 
  = car -> car.getBrand().equals("Chevrolet"); 
```

接下来，我们可以在流管道中使用这个`Predicate`，如下所示：

```java
List<Car> chevrolets = cars.stream() 
  .filter(pChevrolets)
  .collect(Collectors.toList()); 
```

一个`Predicate`可以通过至少三种方式取反。我们可以通过逻辑非(`!`)运算符取反条件：

```java
Predicate<Car> pNotChevrolets 
  = car -> !car.getBrand().equals("Chevrolet"); 
```

我们可以调用`Predicate.negate()`方法：

```java
Predicate<Car> pNotChevrolets = pChevrolets.negate(); 
```

或者我们可以调用`Predicate.not()`方法：

```java
Predicate<Car> pNotChevrolets = Predicate.not(pChevrolets); 
```

无论您更喜欢哪种方法，以下过滤器将产生所有不是雪佛兰的汽车：

```java
List<Car> notChevrolets = cars.stream() 
  .filter(pNotChevrolets) 
  .collect(Collectors.toList()); 
```

在前面的例子中，我们在流管道中应用了一个谓词。然而，我们也可以应用多个谓词。例如，我们可能想要表达一个过滤条件，产生所有不是雪佛兰且至少有 150 马力的汽车。对于这个复合谓词的第一部分，我们可以任意使用`pChevrolets.negate()`，而对于第二部分，我们需要以下`Predicate`：

```java
Predicate<Car> pHorsepower 
  = car -> car.getHorsepower() >= 150; 
```

我们可以通过链式调用`filter()`来获得一个复合谓词，如下所示：

```java
List<Car> notChevrolets150 = cars.stream() 
  .filter(pChevrolets.negate())
  .filter(pHorsepower)
  .collect(Collectors.toList()); 
```

依赖于`Predicate.and(Predicate<? super T> other)`可以使代码更简洁、更易于表达，它会在两个谓词之间应用短路逻辑与。所以前面的例子可以这样表达：

```java
List<Car> notChevrolets150 = cars.stream()
  .filter(pChevrolets.negate().and(pHorsepower))
  .collect(Collectors.toList()); 
```

如果我们需要在两个谓词之间应用短路逻辑或，那么依赖于`Predicate.or(Predicate<? super T> other)`是正确的选择。例如，如果我们想要表达一个过滤条件，产生所有雪佛兰或电动汽车，那么我们可以这样做：

```java
Predicate<Car> pElectric 
  = car -> car.getFuel().equals("electric");

List<Car> chevroletsOrElectric = cars.stream() 
  .filter(pChevrolets.or(pElectric))
  .collect(Collectors.toList()); 
```

如果我们处于一个高度依赖复合谓词的场景中，那么我们可以先创建两个辅助函数，使我们的工作更容易：

```java
@SuppressWarnings("unchecked")
public final class Predicates {

  private Predicates() {
    throw new AssertionError("Cannot be instantiated");
  }
  public static <T> Predicate<T> asOneAnd(
      Predicate<T>... predicates) {
    Predicate<T> theOneAnd = Stream.of(predicates)
      .reduce(p -> true, Predicate::and);

    return theOneAnd;
  }

  public static <T> Predicate<T> asOneOr(
      Predicate<T>... predicates) {
    Predicate<T> theOneOr = Stream.of(predicates)
      .reduce(p -> false, Predicate::or);

    return theOneOr; 
  }
} 
```

这些辅助函数的目标是将几个谓词粘合在一起，通过短路逻辑与和或形成一个单一的组合谓词。

假设我们想要表达一个通过短路逻辑与应用以下三个谓词的过滤条件：

```java
Predicate<Car> pLexus = car -> car.getBrand().equals("Lexus");
Predicate<Car> pDiesel = car -> car.getFuel().equals("diesel"); 
Predicate<Car> p250 = car -> car.getHorsepower() > 250; 
```

首先，我们将这些谓词合并为一个单一的谓词：

```java
Predicate<Car> predicateAnd = Predicates
  .asOneAnd(pLexus, pDiesel, p250); 
```

然后，我们表达过滤条件：

```java
List<Car> lexusDiesel250And = cars.stream() 
  .filter(predicateAnd) 
  .collect(Collectors.toList()); 
```

那么表达一个产生包含所有马力在 100 到 200 或 300 到 400 之间的汽车的流的过滤条件怎么样？谓词如下：

```java
Predicate<Car> p100 = car -> car.getHorsepower() >= 100;
Predicate<Car> p200 = car -> car.getHorsepower() <= 200;

Predicate<Car> p300 = car -> car.getHorsepower() >= 300;
Predicate<Car> p400 = car -> car.getHorsepower() <= 400; 
```

组合谓词可以按照以下方式获得：

```java
Predicate<Car> pCombo = Predicates.asOneOr(
  Predicates.asOneAnd(p100, p200), 
  Predicates.asOneAnd(p300, p400)
); 
```

表达过滤条件很简单：

```java
List<Car> comboAndOr = cars.stream() 
  .filter(pCombo) 
  .collect(Collectors.toList()); 
```

你可以在捆绑的代码中找到所有这些示例。

# 192. 使用 Streams 过滤嵌套集合

这是一个面试中的经典问题，通常从一个模型开始，如下（我们假设集合是一个`List`）：

```java
public class Author {
  private final String name;
  private final List<Book> books;
  ...
}
public class Book {

  private final String title;
  private final LocalDate published;
  ...
} 
```

将`List<Author>`表示为`authors`，编写一个流管道，返回在 2002 年出版的`List<Book>`。你应该已经认识到这是一个典型的`flatMap()`问题，所以无需进一步细节，我们可以写出如下代码：

```java
List<Book> book2002fm = authors.stream()
  .flatMap(author -> author.getBooks().stream())
  .filter(book -> book.getPublished().getYear() == 2002)
  .collect(Collectors.toList()); 
mapMulti():
```

```java
List<Book> book2002mm = authors.stream()
  .<Book>mapMulti((author, consumer) -> {
     for (Book book : author.getBooks()) {
       if (book.getPublished().getYear() == 2002) {
         consumer.accept(book);
       }
     }
   })
   .collect(Collectors.toList()); 
```

好的，这已经很清晰了！那么，我们如何找到在 2002 年出版的`List<Author>`呢？当然，`mapMulti()`可以再次帮助我们。我们只需要遍历书籍，当我们找到一个在 2002 年出版的书籍时，我们只需将`author`传递给`consumer`而不是书籍。此外，在将`author`传递给`consumer`之后，我们可以中断当前作者的循环，并取下一个作者：

```java
List<Author> author2002mm = authors.stream()
  .<Author>mapMulti((author, consumer) -> {
     for (Book book : author.getBooks()) {
       if (book.getPublished().getYear() == 2002) {
         consumer.accept(author);
         break;
       }
     }
   })
   .collect(Collectors.toList()); 
```

另一种方法可以依赖于`anyMatch()`和一个产生 2002 年出版书籍流的谓词，如下所示：

```java
List<Author> authors2002am = authors.stream()
  .filter(
     author -> author.getBooks()
                     .stream()
                     .anyMatch(book -> book.getPublished()
                       .getYear() == 2002)
  )
 .collect(Collectors.toList()); 
```

通常，我们不想修改给定的列表，但如果这不是问题（或者这正是我们想要的），那么我们可以依赖`removeIf()`直接在`List<Author>`上完成相同的结果：

```java
authors.removeIf(author -> author.getBooks().stream()
  .noneMatch(book -> book.getPublished().getYear() == 2002)); 
```

完成！现在，如果你在面试中遇到类似的问题，你应该不会有任何问题。

# 193. 使用 BiPredicate

让我们考虑`Car`模型和一个表示为`cars`的`List<Car>`：

```java
public class Car {
  private final String brand;
  private final String fuel;
  private final int horsepower;
  ...
} 
```

我们的目标是查看以下`Car`是否包含在`cars`中：

```java
Car car = new Car("Ford", "electric", 80); 
```

我们知道`List` API 公开了一个名为`contains(Object o)`的方法。此方法在给定的`Object`存在于给定的`List`中时返回`true`。因此，我们可以轻松地编写一个`Predicate`，如下所示：

```java
Predicate<Car> predicate = cars::contains; 
```

然后，我们调用`test()`方法，我们应该得到预期的结果：

```java
System.out.println(predicate.test(car)); // true 
```

我们可以通过`filter()`、`anyMatch()`等在流管道中获得相同的结果。这里是通过`anyMatch()`实现的：

```java
System.out.println(
  cars.stream().anyMatch(p -> p.equals(car))
); 
```

或者，我们可以依赖`BiPredicate`。这是一个表示已知`Predicate`的两个参数特殊化的函数式接口。它的`test(Object o1, Object o2)`方法接受两个参数，因此它非常适合我们的情况：

```java
BiPredicate<List<Car>, Car> biPredicate = List::contains; 
```

我们可以按照以下方式执行测试：

```java
System.out.println(biPredicate.test(cars, car)); // true 
```

在下一个问题中，你将看到一个使用`BiPredicate`的更实际的例子。

# 194. 为自定义模型构建动态谓词

让我们考虑`Car`模型和一个表示为`cars`的`List<Car>`：

```java
public class Car {
  private final String brand;
  private final String fuel;
  private final int horsepower;
  ...
} 
```

此外，假设我们需要动态生成一系列谓词，这些谓词将`<`、`>`、`<=`、`>=`、`!=`和`==`运算符应用于`horsepower`字段。直接硬编码这些谓词会很麻烦，因此我们必须想出一个解决方案，可以即时构建任何涉及此字段和此处列出的比较运算符的谓词。

有几种方法可以实现这个目标，其中之一是使用 Java `enum`。我们有一个固定的运算符列表，可以编码为`enum`元素，如下所示：

```java
enum PredicateBuilder {
  GT((t, u) -> t > u),
  LT((t, u) -> t < u),
  GE((t, u) -> t >= u),
  LE((t, u) -> t <= u),
  EQ((t, u) -> t.intValue() == u.intValue()),
  NOT_EQ((t, u) -> t.intValue() != u.intValue());
  ... 
```

为了应用这些`(t, u)`lambda 表达式之一，我们需要一个`BiPredicate`构造函数（参见*问题 193*），如下所示：

```java
 private final BiPredicate<Integer, Integer> predicate;
  private PredicateBuilder(
      BiPredicate<Integer, Integer> predicate) {
    this.predicate = predicate;
  }
  ... 
```

现在我们能够定义一个`BiPredicate`，我们可以编写包含实际测试并返回`Predicate<T>`的方法：

```java
 public <T> Predicate<T> toPredicate(
      Function<T, Integer> getter, int u) {
    return obj -> this.predicate.test(getter.apply(obj), u);
  }
  ... 
```

最后，我们必须提供这里的`Function<T, Integer>`，这是对应于`horsepower`的 getter。我们可以通过 Java 反射来完成此操作，如下所示：

```java
public static <T> Function<T, Integer> getFieldByName(
    Class<T> cls, String field) {
  return object -> {
    try {
      Field f = cls.getDeclaredField(field);
      f.setAccessible(true);
      return (Integer) f.get(object);
    } catch (IllegalAccessException | IllegalArgumentException
           | NoSuchFieldException | SecurityException e) { 
      throw new RuntimeException(e);
    }
  };
} 
```

当然，这也可以是任何其他类和整数字段，而不仅仅是`Car`类和`horsepower`字段。基于此代码，我们可以动态创建一个谓词，如下所示：

```java
Predicate<Car> gtPredicate 
  = PredicateBuilder.GT.toPredicate(
      PredicateBuilder.getFieldByName(
        Car.class, "horsepower"), 300); 
```

使用此谓词很简单：

```java
cars.stream()
    .filter(gtPredicate)
    .forEach(System.out::println); 
```

您可以使用这个问题作为实现更多类型动态谓词的灵感来源。例如，在下一个问题中，我们在另一个场景中使用了相同的逻辑。

# 195. 从自定义条件映射构建动态谓词

让我们考虑`Car`模型和表示为`cars`的`List<Car>`：

```java
public class Car {
  private final String brand;
  private final String fuel;
  private final int horsepower;
  ...
} 
```

此外，假设我们收到一个类型为*field : value*的`Map`条件，这可以用来构建动态`Predicate`。此类`Map`的示例如下：

```java
Map<String, String> filtersMap = Map.of(
  "brand", "Chevrolet",
  "fuel", "diesel"
); 
```

如您所见，我们有一个`Map<String, String>`，因此我们对`equals()`比较感兴趣。这有助于我们通过以下 Java `enum`（我们遵循*问题 194*中的逻辑）开始开发：

```java
enum PredicateBuilder {
  EQUALS(String::equals);
  ... 
```

当然，我们可以添加更多运算符，例如`startsWith()`、`endsWith()`、`contains()`等等。接下来，基于在*问题 193*和*194*中获得的经验，我们需要添加一个`BiPredicate`构造函数、`toPredicate()`方法和 Java 反射代码来获取给定字段（此处为`brand`和`fuel`）对应的 getter：

```java
 private final BiPredicate<String, String> predicate;
  private PredicateBuilder(
      BiPredicate<String, String> predicate) {
    this.predicate = predicate;
  }
  public <T> Predicate<T> toPredicate(
      Function<T, String> getter, String u) {
    return obj -> this.predicate.test(getter.apply(obj), u);
  }
  public static <T> Function<T, String> 
      getFieldByName(Class<T> cls, String field) {
    return object -> {
      try {
        Field f = cls.getDeclaredField(field);
        f.setAccessible(true);
        return (String) f.get(object);
      } catch (
          IllegalAccessException | IllegalArgumentException
          | NoSuchFieldException | SecurityException e) { 
        throw new RuntimeException(e);
      }
    };
  }        
} 
```

接下来，我们必须为每个映射条目定义一个谓词，并通过短路 AND 运算符将它们链接起来。这可以通过以下循环完成：

```java
Predicate<Car> filterPredicate = t -> true;
for(String key : filtersMap.keySet()) {
  filterPredicate 
    = filterPredicate.and(PredicateBuilder.EQUALS
      .toPredicate(PredicateBuilder.getFieldByName(
        Car.class, key), filtersMap.get(key))); 
} 
```

最后，我们可以使用生成的谓词来过滤汽车：

```java
cars.stream()
    .filter(filterPredicate)
    .forEach(System.out::println); 
```

完成！

# 196. 谓词的登录

我们已经知道`Predicate`函数式接口依赖于其`test()`方法来执行给定的检查，并返回一个布尔值。假设我们想要修改`test()`方法以记录失败案例（导致返回`false`值的案例）。

一种快速的方法是编写一个辅助方法，偷偷地包含日志部分，如下所示：

```java
public final class Predicates {
  private static final Logger logger 
    = LoggerFactory.getLogger(LogPredicate.class);
  private Predicates() {
    throw new AssertionError("Cannot be instantiated");
  }
  public static <T> Predicate<T> testAndLog(
      Predicate<? super T> predicate, String val) {
    return t -> {
      boolean result = predicate.test(t);
      if (!result) {
        logger.warn(predicate + " don't match '" + val + "'");
      }
      return result;
    };
  }
} 
```

另一种方法是通过扩展 `Predicate` 接口，并提供一个 `default` 方法来测试和记录失败案例，如下所示：

```java
@FunctionalInterface
public interface LogPredicate<T> extends Predicate<T> {
  Logger logger = LoggerFactory.getLogger(LogPredicate.class);

  default boolean testAndLog(T t, String val) {
    boolean result = this.test(t);
    if (!result) {
      logger.warn(t + " don't match '" + val + "'");
    }
    return result;
  }
} 
```

您可以在捆绑的代码中练习这些示例。

# 197. 使用 containsAll() 和 containsAny() 扩展 Stream

假设我们有以下代码：

```java
List<Car> cars = Arrays.asList(
  new Car("Dacia", "diesel", 100),
  new Car("Lexus", "gasoline", 300), 
  ...
  new Car("Ford", "electric", 200)
);

Car car1 = new Car("Lexus", "diesel", 300);
Car car2 = new Car("Ford", "electric", 80);
Car car3 = new Car("Chevrolet", "electric", 150);
List<Car> cars123 = List.of(car1, car2, car3); 
```

接下来，在流管道的上下文中，我们想检查 `cars` 是否包含 `car1`、`car2`、`car3` 或 `cars123` 的所有/任何项。

Stream API 提供了一组丰富的中间和最终操作，但它没有内置的 `containsAll()`/`containsAny()`。因此，我们的任务是提供以下最终操作：

```java
boolean contains(T item);
boolean containsAll(T... items);
boolean containsAll(List<? extends T> items);
**boolean****containsAll****(Stream<? extends T> items)****;**
boolean containsAny(T... items);
boolean containsAny(List<? extends T> items);
**boolean****containsAny****(Stream<? extends T> items)****;** 
```

我们突出了获取 `Stream` 参数的方法，因为这些方法提供了主要逻辑，而其余的方法只是在将它们的参数转换为 `Stream` 后调用这些方法。

## 通过自定义接口来暴露 containsAll/Any()

`containsAll(Stream<? extends T> items)` 依赖于一个 `Set` 来完成其任务，如下所示（作为一个挑战，尝试找到另一种实现方法）：

```java
default boolean containsAll(Stream<? extends T> items) {
  Set<? extends T> set = toSet(items);
  if (set.isEmpty()) {
    return true;
  }
  return stream().filter(item -> set.remove(item))
                 .anyMatch(any -> set.isEmpty());
} 
```

`containsAny(Stream<? extends T> items)` 方法也依赖于一个 `Set`：

```java
default boolean containsAny(Stream<? extends T> items) {
  Set<? extends T> set = toSet(items);
  if (set.isEmpty()) {
    return false;
  }
  return stream().anyMatch(set::contains);
} 
```

`toSet()` 方法只是一个辅助工具，它将 `Stream` 项收集到一个 `Set` 中：

```java
static <T> Set<T> toSet(Stream<? extends T> stream) {
  return stream.collect(Collectors.toSet());
} 
```

接下来，让我们偷偷地将这段代码放入其最终位置，即一个自定义界面。

如您所见，`containsAll(Stream<? extends T> items)` 方法和 `containsAny(Stream<? extends T> items)` 被声明为 `default`，这意味着它们是接口的一部分。此外，它们都调用了 `stream()` 方法，这也是接口的一部分，并连接了常规的 `Stream`。

基本上，解决这个问题的快速方法（特别是在面试中非常有用）是编写这个自定义接口（让我们随意命名为 `Streams`），它能够访问原始的内置 `Stream` 接口，如下所示：

```java
@SuppressWarnings("unchecked")
public interface Streams<T> {
  Stream<T> stream();
  static <T> Streams<T> from(Stream<T> stream) {
    return () -> stream;
  }
  ... 
```

接下来，该接口公开了一组 `default` 方法，代表 `containsAll()`/`containsAny()` 的风味，如下所示：

```java
 default boolean contains(T item) {
    return stream().anyMatch(isEqual(item));
  }
  default boolean containsAll(T... items) {
    return containsAll(Stream.of(items));
  }
  default boolean containsAll(List<? extends T> items) {
    return containsAll(items.stream());
  }
  default boolean containsAll(Stream<? extends T> items) {
    ...   
  }
  default boolean containsAny(T... items) {
    return containsAny(Stream.of(items));
  }
  default boolean containsAny(List<? extends T> items) {
    return containsAny(items.stream());
  }
  default boolean containsAny(Stream<? extends T> items) {
    ...
  }
  static <T> Set<T> toSet(Stream<? extends T> stream) {
    ...
  }
} 
```

完成！现在，我们可以编写使用全新的 `containsAll`/`Any()` 操作的不同流管道。例如，如果我们想检查 `cars` 是否包含 `cars123` 中的所有项，我们可以将流管道表达如下：

```java
boolean result = Streams.from(cars.stream())
  .containsAll(cars123); 
```

这里有一些更多的例子：

```java
boolean result = Streams.from(cars.stream())
  .containsAll(car1, car2, car3);
boolean result = Streams.from(cars.stream())
  .containsAny(car1, car2, car3); 
```

如以下示例所示，可以涉及更多操作：

```java
Car car4 = new Car("Mercedes", "electric", 200); 
boolean result = Streams.from(cars.stream()
    .filter(car -> car.getBrand().equals("Mercedes"))
    .distinct()
    .dropWhile(car -> car.getFuel().equals("gasoline"))
  ).contains(car4); 
```

解决这个问题的更简洁和完整的解决方案是扩展 `Stream` 接口。让我们来做吧！

## 通过扩展 Stream 来暴露 containsAll/Any()

之前的解决方案更像是一种黑客行为。一个更合理和现实的解决方案是扩展内置的 Stream API，并将我们的 `containsAll`/`Any()` 方法作为团队成员添加到 `Stream` 操作旁边。因此，实现开始如下：

```java
@SuppressWarnings("unchecked")
public interface Streams<T> extends Stream<T> { 
  ...
} 
```

在实现 `containsAll`/`Any()` 方法之前，我们需要处理一些由扩展 `Stream` 接口产生的问题。首先，我们需要在 `Streams` 中覆盖每个 `Stream` 方法。由于 `Stream` 接口有很多方法，我们这里只列出其中一些：

```java
@Override
public Streams<T> filter(Predicate<? super T> predicate);
@Override
public <R> Streams<R> map(
  Function<? super T, ? extends R> mapper);
...
@Override
public T reduce(T identity, BinaryOperator<T> accumulator);
...
@Override
default boolean isParallel() {
  return false;
}
...
@Override
default Streams<T> parallel() {
  throw new UnsupportedOperationException(
    "Not supported yet."); // or, return this
}
@Override
default Streams<T> unordered() {
  throw new UnsupportedOperationException(
    "Not supported yet."); // or, return this
}
...
@Override
default Streams<T> sequential() {
  return this;
} 
```

由于 `Streams` 只能处理顺序流（不支持并行），我们可以直接在 `Streams` 中实现 `isParallel()`、`parallel()`、`unordered()` 和 `sequential()` 方法作为 `default` 方法。

接下来，为了使用 `Streams`，我们需要一个 `from(Stream s)` 方法，它能够包装给定的 `Stream`，如下所示：

```java
static <T> Streams<T> from(Stream<? extends T> stream) {
  if (stream == null) {
    return from(Stream.empty());
  }
  if (stream instanceof Streams) {
    return (Streams<T>) stream;
  }
  return new StreamsWrapper<>(stream);
} 
```

`StreamsWrapper` 是一个将当前 `Stream` 包装成顺序 `Streams` 的类。`StreamsWrapper` 类实现了 `Streams`，因此它必须覆盖所有 `Streams` 方法，并正确地将 `Stream` 包装成 `Streams`。由于 `Streams` 有很多方法（这是扩展 `Stream` 的结果），我们这里只列出其中一些（其余的可以在捆绑的代码中找到）：

```java
@SuppressWarnings("unchecked")
public class StreamsWrapper<T> implements Streams<T> {
  private final Stream<? extends T> delegator;
  public StreamsWrapper(Stream<? extends T> delegator) {
    this.delegator = delegator.sequential();
  }       
  @Override
  public Streams<T> filter(Predicate<? super T> predicate) { 
    return Streams.from(delegator.filter(predicate));
  }
  @Override
  public <R> Streams<R> map(
      Function<? super T, ? extends R> mapper) {
    return Streams.from(delegator.map(mapper));
  } 
  ...
  @Override
  public T reduce(T identity, BinaryOperator<T> accumulator) {
    return ((Stream<T>) delegator)
      .reduce(identity, accumulator);
  }
  ...
} 
```

最后，我们将 `Streams` 添加到 `containsAll`/`Any()` 方法中，这些方法相当直接（由于 `Streams` 扩展了 `Stream`，我们无需编写 `stream()` 演技，就可以访问所有 `Stream` 的优点，就像在先前的解决方案中那样）。首先，我们添加 `containsAll()` 方法：

```java
default boolean contains(T item) {
  return anyMatch(isEqual(item));
}
default boolean containsAll(T... items) {
  return containsAll(Stream.of(items));
}
default boolean containsAll(List<? extends T> items) {
  return containsAll(items.stream());
}
default boolean containsAll(Stream<? extends T> items) {
  Set<? extends T> set = toSet(items);
  if (set.isEmpty()) {
    return true;
  }
  return filter(item -> set.remove(item))
    .anyMatch(any -> set.isEmpty());
} 
```

第二，我们添加 `containsAny()` 方法：

```java
default boolean containsAny(T... items) {
  return containsAny(Stream.of(items));
}
default boolean containsAny(List<? extends T> items) {
  return containsAny(items.stream());
}
default boolean containsAny(Stream<? extends T> items) {
  Set<? extends T> set = toSet(items);
  if (set.isEmpty()) {
    return false;
  }
  return anyMatch(set::contains);
} 
```

最后，我们添加了 `toSet()` 方法，您已经知道了：

```java
static <T> Set<T> toSet(Stream<? extends T> stream) {
  return stream.collect(Collectors.toSet());
} 
```

任务完成！现在，让我们写一些示例：

```java
boolean result = Streams.from(cars.stream())
  .filter(car -> car.getBrand().equals("Mercedes"))
  .contains(car1);
boolean result = Streams.from(cars.stream())
  .containsAll(cars123);
boolean result = Streams.from(cars123.stream())
  .containsAny(cars.stream()); 
```

您可以在捆绑的代码中找到更多示例。

# 198\. 通过 `removeAll()` 和 `retainAll()` 扩展 Stream

在阅读这个问题之前，我强烈建议您阅读 *问题 197*。

在 *问题 197* 中，我们通过自定义接口扩展了 Stream API，添加了两个名为 `containsAll()` 和 `containsAny()` 的最终操作。在两种情况下，生成的接口都命名为 `Streams`。在这个问题中，我们遵循相同的逻辑来实现两个名为 `removeAll()` 和 `retainAll()` 的中间操作，其签名如下：

```java
Streams<T> remove(T item);
Streams<T> removeAll(T... items);
Streams<T> removeAll(List<? extends T> items);
**Streams<T>** **removeAll****(Stream<? extends T> items)****;**
Streams<T> retainAll(T... items);
Streams<T> retainAll(List<? extends T> items);
**Streams<T>** **retainAll****(Stream<? extends T> items)****;** 
```

由于 `removeAll()` 和 `retainAll()` 是中间操作，它们必须返回 `Stream`。更确切地说，它们必须返回 `Streams`，这是我们基于自定义接口或扩展 `Stream` 的接口的实现。

## 通过自定义接口公开 `removeAll()`/`retainAll()`

`removeAll(Stream<? extends T> items)` 方法依赖于 `Set` 来完成其任务，如下所示（作为一个挑战，尝试找到另一种实现）：

```java
default Streams<T> removeAll(Stream<? extends T> items) {
  Set<? extends T> set = toSet(items);
  if (set.isEmpty()) {
    return this;
  }
  return from(stream().filter(item -> !set.contains(item)));
} 
```

`retainAll(Stream<? extends T> items)` 方法也依赖于 `Set`：

```java
default Streams<T> retainAll(Stream<? extends T> items) {
  Set<? extends T> set = toSet(items);
  if (set.isEmpty()) {
    return from(Stream.empty());
  }
  return from(stream().filter(item -> set.contains(item)));
} 
```

`toSet()` 方法只是一个收集 `Stream` 项到 `Set` 的辅助工具：

```java
static <T> Set<T> toSet(Stream<? extends T> stream) {
  return stream.collect(Collectors.toSet());
} 
```

接下来，我们可以将这些 `default` 方法悄悄地放入一个名为 `Streams` 的自定义接口中，就像我们在 *问题 197* 中做的那样：

```java
@SuppressWarnings("unchecked")
public interface Streams<T> {
  Stream<T> stream();
  static <T> Streams<T> from(Stream<T> stream) {
    return () -> stream;
  }
  // removeAll()/retainAll() default methods and toSet()
} 
```

这个实现有一个大问题。当我们尝试在流管道中链式调用`removeAll()`/`retainAll()`旁边其他`Stream`操作时，问题变得明显。因为这两个方法返回`Streams`（而不是`Stream`），我们无法在它们之后链式调用`Stream`操作，而必须首先调用 Java 内置的`stream()`。这是从`Streams`切换到`Stream`所需要的。以下是一个示例（使用在*问题 197*中引入的`cars`，`car1`，`car2`，`car3`和`car123`）：

```java
Streams.from(cars.stream())
  .retainAll(cars123)
  .removeAll(car1, car3)
  **.stream()**
  .forEach(System.out::println); 
```

如果我们必须在`Streams`和`Stream`之间多次交替，问题会变得更加严重。查看这个僵尸：

```java
Streams.from(Streams.from(cars.stream().distinct())
  .retainAll(car1, car2, car3)
  .stream()
  .filter(car -> car.getFuel().equals("electric")))
  .removeAll(car2)
  .stream()
  .forEach(System.out::println); 
```

这个技巧并不是一个令人愉快的选项来丰富 Stream API 的中间操作。然而，它对于终端操作工作得相当好。因此，正确的方法是扩展`Stream`接口。

## 通过扩展 Stream 暴露 removeAll/retainAll()

我们已经从*问题 197*中了解到如何扩展`Stream`接口。`removeAll()`的实现也是直接的：

```java
@SuppressWarnings("unchecked")
public interface Streams<T> extends Stream<T> {   

  default Streams<T> remove(T item) {
    return removeAll(item);
  }

  default Streams<T> removeAll(T... items) {
    return removeAll(Stream.of(items));
  }
  default Streams<T> removeAll(List<? extends T> items) {
    return removeAll(items.stream());
  }
  default Streams<T> removeAll(Stream<? extends T> items) {
    Set<? extends T> set = toSet(items);
    if (set.isEmpty()) {
      return this;
    }
    return filter(item -> !set.contains(item))
      .onClose(items::close);
  }       
  ... 
```

然后，以同样的方式跟随`retainAll()`：

```java
 default Streams<T> retainAll(T... items) {
    return retainAll(Stream.of(items));
  }
  default Streams<T> retainAll(List<? extends T> items) {
    return retainAll(items.stream());
  }
  default Streams<T> retainAll(Stream<? extends T> items) {
    Set<? extends T> set = toSet(items);
    if (set.isEmpty()) {
      return from(Stream.empty());
    }
    return filter(item -> set.contains(item))
      .onClose(items::close);
  }
  ...
} 
```

如您从*问题 197*中知道，接下来，我们必须重写所有`Stream`方法以返回`Streams`。虽然这部分代码在捆绑代码中可用，以下是如何使用`removeAll()`/`retainAll()`的示例：

```java
Streams.from(cars.stream())
  .distinct() 
  .retainAll(car1, car2, car3)
  .filter(car -> car.getFuel().equals("electric")) 
  .removeAll(car2) 
  .forEach(System.out::println); 
```

如您所见，这次，流管道看起来相当好。没有必要通过`stream()`调用在`Streams`和`Stream`之间进行切换。所以，任务完成了！

# 199. 引入流比较器

假设我们有以下三个列表（一个数字列表，一个字符串列表和一个`Car`对象列表）：

```java
List<Integer> nrs = new ArrayList<>();
List<String> strs = new ArrayList<>();
List<Car> cars = List.of(...);
public class Car {
  private final String brand;
  private final String fuel;
  private final int horsepower;
  ...
} 
```

接下来，我们想在流管道中对这些列表进行排序。

## 通过自然顺序排序

通过自然顺序排序非常简单。我们只需要调用内置的中间操作`sorted()`：

```java
nrs.stream()
   .sorted()
   .forEach(System.out::println);
strs.stream()
    .sorted()
    .forEach(System.out::println); 
```

如果`nrs`包含 1，6，3，8，2，3 和 0，那么`sorted()`将产生 0，1，2，3，3，6 和 8。因此，对于数字，自然顺序是按值升序排列。

如果`strs`包含“book”，“old”，“new”，“quiz”，“around”和“tick”，那么`sorted()`将产生“around”，“book”，“new”，“old”，“quiz”和“tick”。因此，对于字符串，自然顺序是字母顺序。

如果我们显式地通过`sorted(Comparator<? super T> comparator)`调用`Integer.compareTo()`和`String.compareTo()`，可以得到相同的结果：

```java
nrs.stream()
   .sorted((n1, n2) -> n1.compareTo(n2))
   .forEach(System.out::println);
strs.stream()
    .sorted((s1, s2) -> s1.compareTo(s2))
    .forEach(System.out::println); 
```

或者，我们可以使用`java.util.Comparator`函数式接口，如下所示：

```java
nrs.stream()
   .sorted(Comparator.naturalOrder())
   .forEach(System.out::println);
strs.stream()
    .sorted(Comparator.naturalOrder())
    .forEach(System.out::println); 
```

这三种方法返回相同的结果。

## 反转自然顺序

可以通过`Comparator.reverseOrder()`反转自然顺序，如下所示：

```java
nrs.stream()
   .sorted(Comparator.reverseOrder())
   .forEach(System.out::println);
strs.stream()
    .sorted(Comparator.reverseOrder())
    .forEach(System.out::println); 
```

如果`nrs`包含 1，6，3，8，2，3 和 0，那么`sorted()`将产生 8，6，3，3，2，1 和 0。反转数字的自然顺序将按值降序排列。

如果`strs`包含“book”，“old”，“new”，“quiz”，“around”和“tick”，那么`sorted()`将产生“tick”，“quiz”，“old”，“new”，“book”和“around”。所以对于字符串，反转自然顺序会导致反转字母顺序。

## 排序和 null 值

如果`nrs`/`strs`包含`null`值，那么所有之前的示例都将抛出`NullPointerException`。然而，`java.util.Comparator`提供了两个方法，允许我们首先（`nullsFirst(Comparator<? super T> comparator)`)或最后（`nullsLast(Comparator<? super T> comparator)`）对`null`值进行排序。它们的使用方法如下面的示例所示：

```java
nrs.stream()
   .sorted(Comparator.nullsFirst(Comparator.naturalOrder()))
   .forEach(System.out::println);

nrs.stream()
   .sorted(Comparator.nullsLast(Comparator.naturalOrder()))
   .forEach(System.out::println);
nrs.stream()
   .sorted(Comparator.nullsFirst(Comparator.reverseOrder()))
   .forEach(System.out::println); 
```

第三个示例首先对`null`值进行排序，然后按逆序排序数字。

## 编写自定义比较器

有时，我们需要一个自定义比较器。例如，如果我们想按最后一个字符对`strs`进行升序排序，那么我们可以编写一个自定义比较器，如下所示：

```java
strs.stream()
    .sorted((s1, s2) -> 
       Character.compare(s1.charAt(s1.length() - 1), 
                         s2.charAt(s2.length() - 1)))
    .forEach(System.out::println); 
```

如果`strs`包含“book”，“old”，“new”，“quiz”，“around”和“tick”，那么`sorted()`将产生“old”，“around”，“book”，“tick”，“new”和“quiz”。

然而，自定义比较器通常用于对模型进行排序。例如，如果我们需要排序`cars`列表，那么我们需要定义一个比较器。我们不能只是说：

```java
cars.stream()
    .sorted()
    .forEach(System.out::println); 
```

这将无法编译，因为没有为`Car`对象提供比较器。一种方法是实现`Comparable`接口并重写`compareTo(Car c)`方法。例如，如果我们想按`horsepower`对`cars`进行升序排序，那么我们首先实现`Comparable`，如下所示：

```java
public class Car implements Comparable<Car> {
  ...
  @Override
  public int compareTo(Car c) {
    return this.getHorsepower() > c.getHorsepower()
      ? 1 : this.getHorsepower() < c.getHorsepower() ? -1 : 0;
  }
} 
```

现在，我们可以成功编写这个：

```java
cars.stream()
    .sorted()
    .forEach(System.out::println); 
```

或者，如果我们不能修改`Car`代码，我们可以尝试使用现有的`Comparator`方法之一，这些方法允许我们传递一个包含排序键的函数，并返回一个自动按该键比较的`Comparator`。由于`horsepower`是整数，我们可以使用`comparingInt(ToIntFunction<? super T> keyExtractor)`，如下所示：

```java
cars.stream() 
    .sorted(Comparator.comparingInt(Car::getHorsepower))
    .forEach(System.out::println); 
```

这里是反转顺序：

```java
cars.stream()
    .sorted(Comparator.comparingInt(
            Car::getHorsepower).reversed())
    .forEach(System.out::println); 
```

你可能还对`comparingLong(ToLongFunction)`和`comparingDouble(ToDoubleFunction)`感兴趣。

`ToIntFunction`，`ToLongFunction`和`ToDoubleFunction`是`Function`方法的特殊化。在这个上下文中，我们可以说`comparingInt()`，`comparingLong()`和`comparingDouble()`是`comparing()`的特殊化，`comparing()`有两种风味：`comparing(Function<? super T,? extends U> keyExtractor)`和`comparing(Function<? super T,? extends U> keyExtractor, Comparator<? super U> keyComparator)`。

这里是使用`comparing()`的第二种风味按`fuel`类型（自然顺序）对`cars`进行升序排序的示例，将`null`值放在末尾：

```java
cars.stream()
    .sorted(Comparator.comparing(Car::getFuel, 
            Comparator.nullsLast(Comparator.naturalOrder())))
    .forEach(System.out::println); 
```

此外，这里还有一个按`fuel`类型的最后一个字符对`cars`进行升序排序的示例，将`null`值放在末尾：

```java
cars.stream()
    .sorted(Comparator.comparing(Car::getFuel, 
            Comparator.nullsLast((s1, s2) -> 
              Character.compare(s1.charAt(s1.length() - 1), 
                                s2.charAt(s2.length() - 1)))))
    .forEach(System.out::println); 
```

通常，在函数表达式中链式多个比较器会导致代码可读性降低。在这种情况下，您可以通过导入静态并分配以“by”开头的变量来保持代码的可读性，如下例所示（此代码的结果与上一个示例相同，但更易于阅读）：

```java
import static java.util.Comparator.comparing;
import static java.util.Comparator.nullsLast;
...
Comparator<String> byCharAt = nullsLast(
   (s1, s2) -> Character.compare(s1.charAt(s1.length() - 1), 
    s2.charAt(s2.length() - 1)));
Comparator<Car> byFuelAndCharAt = comparing(
  Car::getFuel, byCharAt);
cars.stream()
    .sorted(byFuelAndCharAt)
    .forEach(System.out::println); 
```

完成！在下一个问题中，我们将对映射进行排序。

# 200. 对映射进行排序

假设我们有以下映射：

```java
public class Car {
  private final String brand;
  private final String fuel;
  private final int horsepower;
  ...
}
Map<Integer, Car> cars = Map.of(
  1, new Car("Dacia", "diesel", 350),
  2, new Car("Lexus", "gasoline", 350),
  3, new Car("Chevrolet", "electric", 150),
  4, new Car("Mercedes", "gasoline", 150),
  5, new Car("Chevrolet", "diesel", 250),
  6, new Car("Ford", "electric", 80),
  7, new Car("Chevrolet", "diesel", 450),
  8, new Car("Mercedes", "electric", 200),
  9, new Car("Chevrolet", "gasoline", 350),
  10, new Car("Lexus", "diesel", 300)
); 
```

接下来，我们希望将此映射排序到 `List<String>` 中，如下所示：

+   如果马力值不同，则按马力降序排序

+   如果马力值相等，则按映射键的升序排序

+   结果 `List<String>` 应包含类型为 *键(马力)* 的项

在这些语句下，对 `cars` 映射进行排序将得到：

```java
[7(450), 1(350), 2(350), 9(350), 10(300), 5(250), 
8(200), 3(150), 4(150), 6(80)] 
```

显然，这个问题需要一个自定义的比较器。有两个映射条目 `(c1, c2)`，我们详细阐述以下逻辑：

1.  检查 `c2` 的马力是否等于 `c1` 的马力

1.  如果它们相等，则比较 `c1` 的键与 `c2` 的键

1.  否则，比较 `c2` 的马力与 `c1` 的马力

1.  将结果收集到 `List` 中

在代码行中，这可以表示如下：

```java
List<String> result = cars.entrySet().stream()
  .sorted((c1, c2) -> c2.getValue().getHorsepower() 
        == c1.getValue().getHorsepower()
     ? c1.getKey().compareTo(c2.getKey())
     : Integer.valueOf(c2.getValue().getHorsepower())
        .compareTo(c1.getValue().getHorsepower()))
  .map(c -> c.getKey() + "(" 
                       + c.getValue().getHorsepower() + ")")
  .toList(); 
```

或者，如果我们依赖于 `Map.Entry.comparingByValue()`、`comparingByKey()` 和 `java.util.Comparator`，则可以写成如下：

```java
List<String> result = cars.entrySet().stream()
  .sorted(Entry.<Integer, Car>comparingByValue(
            Comparator.comparingInt(
              Car::getHorsepower).reversed())
  .thenComparing(Entry.comparingByKey()))
  .map(c -> c.getKey() + "(" 
    + c.getValue().getHorsepower() + ")")
  .toList(); 
```

这种方法更易于阅读和表达。

# 201. 过滤映射

让我们考虑以下映射：

```java
public class Car {
  private final String brand;
  private final String fuel;
  private final int horsepower;
  ...
}
Map<Integer, Car> cars = Map.of(
  1, new Car("Dacia", "diesel", 100),
  ...
  10, new Car("Lexus", "diesel", 300)
); 
```

为了流映射，我们可以从 `Map` 的 `entrySet()`、`values()` 或 `keyset()` 开始，然后调用 `stream()`。例如，如果我们想表达一个表示为 *Map* -> *Stream* -> *Filter* -> *String* 的管道，它返回包含所有电动汽车品牌的 `List<String>`，则可以依赖 `entrySet()` 如下：

```java
String electricBrands = cars.entrySet().stream()
  .filter(c -> "electric".equals(c.getValue().getFuel()))
  .map(c -> c.getValue().getBrand())
  .collect(Collectors.joining(", ")); 
```

然而，正如你所看到的，这个流管道没有使用映射的键。这意味着我们可以通过 `values()` 而不是 `entrySet()` 更好地表达它，如下所示：

```java
String electricBrands = cars.values().stream()
  .filter(c -> "electric".equals(c.getFuel()))
  .map(c -> c.getBrand())
  .collect(Collectors.joining(", ")); 
```

这更易于阅读，并且清楚地表达了其意图。

这里有一个例子，你应该能够理解而不需要进一步细节：

```java
Car newCar = new Car("No name", "gasoline", 350);
String carsAsNewCar1 = cars.entrySet().stream()
 .filter(c -> (c.getValue().getFuel().equals(newCar.getFuel())
   && c.getValue().getHorsepower() == newCar.getHorsepower()))
 .map(map -> map.getValue().getBrand())
 .collect(Collectors.joining(", "));

String carsAsNewCar2 = cars.values().stream()
 .filter(c -> (c.getFuel().equals(newCar.getFuel())
   && c.getHorsepower() == newCar.getHorsepower()))
 .map(map -> map.getBrand())
 .collect(Collectors.joining(", ")); 
```

因此，当流管道只需要映射的值时，我们可以从 `values()` 开始；当它只需要键时，我们可以从 `keyset()` 开始；当它需要两者（值和键）时，我们可以从 `entrySet()` 开始。

例如，一个表示为 *Map* -> *Stream* -> *Filter* -> *Map* 的流管道，它通过键过滤前五辆汽车并将它们收集到结果映射中，需要从 `entrySet()` 开始，如下所示：

```java
Map<Integer, Car> carsTop5a = cars.entrySet().stream()
  .filter(c -> c.getKey() <= 5)
  .collect(Collectors.toMap(
     Map.Entry::getKey, Map.Entry::getValue));
  //or, .collect(Collectors.toMap(
  //      c -> c.getKey(), c -> c.getValue())); 
```

这里有一个返回具有超过 100 马力的前五辆汽车的映射的例子：

```java
Map<Integer, Car> hp100Top5a = cars.entrySet().stream()
  .filter(c -> c.getValue().getHorsepower() > 100)
  .sorted(Entry.comparingByValue(
          Comparator.comparingInt(Car::getHorsepower)))
  .collect(Collectors.toMap(
     Map.Entry::getKey, Map.Entry::getValue, 
       (c1, c2) -> c2, LinkedHashMap::new));
  //or, .collect(Collectors.toMap(
  //      c -> c.getKey(), c -> c.getValue(), 
  //      (c1, c2) -> c2, LinkedHashMap::new)); 
```

如果我们需要经常表达这样的管道，那么我们可能更喜欢编写一些辅助函数。以下是一组用于按键过滤和排序 `Map<K, V>` 的四个通用辅助函数：

```java
public final class Filters {
  private Filters() {
    throw new AssertionError("Cannot be instantiated");
  }
  public static <K, V> Map<K, V> byKey(
        Map<K, V> map, Predicate<K> predicate) {
  return map.entrySet()
    .stream()
    .filter(item -> predicate.test(item.getKey()))
    .collect(Collectors.toMap(
       Map.Entry::getKey, Map.Entry::getValue));
  }

  public static <K, V> Map<K, V> sortedByKey(
    Map<K, V> map, Predicate<K> predicate, Comparator<K> c) {
    return map.entrySet()
      .stream()
      .filter(item -> predicate.test(item.getKey()))
      .sorted(Map.Entry.comparingByKey(c))
      .collect(Collectors.toMap(
         Map.Entry::getKey, Map.Entry::getValue,
              (c1, c2) -> c2, LinkedHashMap::new));
  }
  ... 
```

用于按值过滤和排序 `Map` 的集合：

```java
 public static <K, V> Map<K, V> byValue(
      Map<K, V> map, Predicate<V> predicate) {
    return map.entrySet()
      .stream()
      .filter(item -> predicate.test(item.getValue()))
      .collect(Collectors.toMap(
         Map.Entry::getKey, Map.Entry::getValue));
  }
  public static <K, V> Map<K, V> sortedbyValue(Map<K, V> map, 
      Predicate<V> predicate, Comparator<V> c) {
  return map.entrySet()
    .stream()
    .filter(item -> predicate.test(item.getValue()))
    .sorted(Map.Entry.comparingByValue(c))
    .collect(Collectors.toMap(
       Map.Entry::getKey, Map.Entry::getValue,
           (c1, c2) -> c2, LinkedHashMap::new));
  }
} 
```

现在，我们的代码已经变得非常简短。例如，我们可以通过键过滤前五辆汽车并将它们收集到结果映射中，如下所示：

```java
Map<Integer, Car> carsTop5s 
  = Filters.byKey(cars, c -> c <= 5); 
```

或者，我们可以按照以下方式过滤出马力超过 100 的前五辆汽车：

```java
Map<Integer, Car> hp100Top5s 
  = Filters.byValue(cars, c -> c.getHorsepower() > 100); 
Map<Integer, Car> hp100Top5d 
  = Filters.sortedbyValue(cars, c -> c.getHorsepower() > 100,
      Comparator.comparingInt(Car::getHorsepower)); 
```

很酷，对吧？！请随意扩展 `Filters` 以包含更多通用辅助函数，以处理流管道中的 `Map` 处理。

# 202. 通过 Collector.of() 创建自定义收集器

在 *Java Coding Problem*，第一版，第 *9* 章，*问题 193* 中，我们详细介绍了创建自定义收集器这个主题。更确切地说，在那个问题中，您看到了如何通过实现 `java.util.stream.Collector` 接口来编写自定义收集器。

如果您还没有阅读那本书/问题，请不要担心；您仍然可以跟随这个问题。首先，我们将创建几个自定义收集器。这次，我们将依赖于两个具有以下签名的 `Collector.of()` 方法：

```java
static <T,R> Collector<T,R,R> of(
  Supplier<R> supplier, 
  BiConsumer<R,T> accumulator, 
  BinaryOperator<R> combiner, 
  Collector.Characteristics... characteristics)
static <T,A,R> Collector<T,A,R> of(
  Supplier<A> supplier, 
  BiConsumer<A,T> accumulator, 
  BinaryOperator<A> combiner, 
  Function<A,R> finisher, 
  Collector.Characteristics... characteristics) 
```

在这个上下文中，`T`、`A` 和 `R` 代表以下内容：

+   `T` 代表 `Stream` 的元素类型（将被收集的元素）

+   `A` 代表在集合过程中使用的对象类型，称为累加器，它用于在可变结果容器中累积流元素

+   `R` 代表集合过程之后对象的类型（最终结果）

此外，一个 `Collector` 由四个函数和一个枚举来表征。以下是 *Java Coding Problems*，第一版中的一段简短笔记：

“*这些函数协同工作，将条目累积到可变结果容器中，并可选择对结果执行最终转换。它们如下：*

+   *创建一个新的空可变结果容器（提供者参数）*

+   *将新的数据元素合并到可变结果容器中（累加器参数）*

+   *将两个可变结果容器合并为一个（组合器参数）*

+   *对可变结果容器执行可选的最终转换以获得最终结果（完成器参数）*

此外，我们还有 `Collector.Characteristics...` 枚举，它定义了收集器的行为。可能的值有 `UNORDERED`（无顺序）、`CONCURRENT`（更多线程累积元素）和 `IDENTITY_FINISH`（完成器是恒等函数，因此不会进行进一步转换）。

在这个上下文中，让我们尝试运行几个示例。但首先，让我们假设我们有以下模型：

```java
public interface Vehicle {}
public class Car implements Vehicle {
  private final String brand;
  private final String fuel;
  private final int horsepower;
  ...
}
public class Submersible implements Vehicle {

  private final String type;
  private final double maxdepth;
  ...
} 
```

此外，还有一些数据：

```java
Map<Integer, Car> cars = Map.of(
  1, new Car("Dacia", "diesel", 100),
  ...
  10, new Car("Lexus", "diesel", 300)
); 
```

接下来，让我们在名为 `MyCollectors` 的辅助类中创建一些收集器。

## 编写一个将元素收集到 TreeSet 的自定义收集器

在一个将元素收集到 `TreeSet` 并以 `TreeSet::new` 为提供者的自定义收集器中，累加器是 `TreeSet.add()`，组合器依赖于 `TreeSet.addAll()`，完成器是恒等函数：

```java
public static <T> 
    Collector<T, TreeSet<T>, TreeSet<T>> toTreeSet() {
  return Collector.of(TreeSet::new, TreeSet::add,
    (left, right) -> {
       left.addAll(right);
       return left;
    }, Collector.Characteristics.IDENTITY_FINISH);
} 
```

在以下示例中，我们使用这个收集器收集所有电品牌到 `TreeSet<String>` 中：

```java
TreeSet<String> electricBrands = cars.values().stream()
  .filter(c -> "electric".equals(c.getFuel()))
  .map(c -> c.getBrand())
  .collect(MyCollectors.toTreeSet()); 
```

非常简单！

## 编写一个将元素收集到 LinkedHashSet 的自定义收集器

在一个将收集器收集到`LinkedHashSet`的自定义收集器中，其中供应商是`LinkedHashSet::new`，累加器是`HashSet::add`，组合器依赖于`HashSet.addAll()`，而完成器是恒等函数：

```java
public static <T> Collector<T, LinkedHashSet<T>, 
    LinkedHashSet<T>> toLinkedHashSet() {
  return Collector.of(LinkedHashSet::new, HashSet::add,
    (left, right) -> {
       left.addAll(right);
       return left;
    }, Collector.Characteristics.IDENTITY_FINISH);
} 
```

在以下示例中，我们使用这个收集器来收集排序后的汽车马力：

```java
LinkedHashSet<Integer> hpSorted = cars.values().stream()
  .map(c -> c.getHorsepower())
  .sorted()
  .collect(MyCollectors.toLinkedHashSet()); 
```

完成！`LinkedHashSet<Integer>`包含按升序排列的马力值。

## 编写一个排除另一个收集器元素的定制收集器

本节的目标是提供一个自定义收集器，它接受一个`Predicate`和一个`Collector`作为参数。它将给定的`predicate`应用于要收集的元素，以排除给定`collector`中的失败项：

```java
public static <T, A, R> Collector<T, A, R> exclude(
    Predicate<T> predicate, Collector<T, A, R> collector) {
  return Collector.of(
    collector.supplier(),
    (l, r) -> {
       if (predicate.negate().test(r)) {
         collector.accumulator().accept(l, r);
       }
    },
    collector.combiner(),
    collector.finisher(),
    collector.characteristics()
     .toArray(Collector.Characteristics[]::new)
  );
} 
```

自定义收集器使用给定的供应商、组合器、完成器和特性。它只影响给定收集器的累加器。基本上，它只显式调用给定收集器的累加器，对于通过给定谓词的元素。

例如，如果我们想通过这个自定义收集器获取小于 200 的排序马力，那么我们可以这样调用它（谓词指定了应该排除的内容）：

```java
LinkedHashSet<Integer> excludeHp200 = cars.values().stream()
  .map(c -> c.getHorsepower())
  .sorted()
  .collect(MyCollectors.exclude(c -> c > 200, 
           MyCollectors.toLinkedHashSet())); 
```

在这里，我们使用了两个自定义收集器，但我们可以轻松地将`toLinkedHashSet()`替换为一个内置收集器。挑战自己编写这个自定义收集器的对应版本。编写一个收集通过给定谓词通过的元素的收集器。

## 编写一个按类型收集元素的定制收集器

假设我们有一个以下`List<Vehicle>`：

```java
Vehicle mazda = new Car("Mazda", "diesel", 155);
Vehicle ferrari = new Car("Ferrari", "gasoline", 500);

Vehicle hov = new Submersible("HOV", 3000);
Vehicle rov = new Submersible("ROV", 7000);

List<Vehicle> vehicles = List.of(mazda, hov, ferrari, rov); 
```

我们的目标是只收集汽车或潜水艇，而不是两者。为此，我们可以编写一个自定义收集器，通过`type`收集到给定的供应商中，如下所示：

```java
public static 
  <T, A extends T, R extends Collection<A>> Collector<T, ?, R> 
    toType(Class<A> type, Supplier<R> supplier) {
  return Collector.of(supplier,
      (R r, T t) -> {
         if (type.isInstance(t)) {
           r.add(type.cast(t));
         }
      },
      (R left, R right) -> {
         left.addAll(right);
         return left;
      },
      Collector.Characteristics.IDENTITY_FINISH
  );
} 
```

现在，我们可以只将`List<Vehicle>`中的汽车收集到`ArrayList`中，如下所示：

```java
List<Car> onlyCars = vehicles.stream()
  .collect(MyCollectors.toType(
    Car.class, ArrayList::new)); 
```

此外，我们只能将潜水艇收集到`HashSet`中，如下所示：

```java
Set<Submersible> onlySubmersible = vehicles.stream()
  .collect(MyCollectors.toType(
    Submersible.class, HashSet::new)); 
```

最后，让我们编写一个用于自定义数据结构的定制收集器。

## 编写一个用于 SplayTree 的定制收集器

在*第五章*，*问题 127*中，我们实现了 SplayTree 数据结构。现在，让我们编写一个能够将元素收集到 SplayTree 中的定制收集器。显然，供应商是`SplayTree::new`。此外，累加器是`SplayTree.insert()`，而组合器是`SplayTree.insertAll()`：

```java
public static 
    Collector<Integer, SplayTree, SplayTree> toSplayTree() {
  return Collector.of(SplayTree::new, SplayTree::insert,
    (left, right) -> {
       left.insertAll(right);
       return left;
    }, 
    Collector.Characteristics.IDENTITY_FINISH);
} 
```

这里有一个示例，它将汽车的马力收集到一个 SplayTree 中：

```java
SplayTree st = cars.values().stream()
  .map(c -> c.getHorsepower())
  .collect(MyCollectors.toSplayTree()); 
```

完成！挑战自己实现一个自定义收集器。

# 203. 从 lambda 表达式抛出检查型异常

假设我们有一个以下 lambda 表达式：

```java
static void readFiles(List<Path> paths) {
  paths.forEach(p -> {
    try {
      readFile(p);
    } catch (IOException e) {
      **...** **// what can we throw here?**
    }
  });
} 
```

我们在`catch`块中可以抛出什么？你们大多数人都会知道答案；我们可以抛出一个未检查的异常，例如`RuntimeException`：

```java
static void readFiles(List<Path> paths) {
  paths.forEach(p -> {
    try {
      readFile(p);
    } catch (IOException e) {
      **throw****new****RuntimeException****(e);**
    }
  });
} 
```

此外，大多数人知道我们不能抛出一个检查型异常，例如`IOException`。以下代码片段将无法编译：

```java
static void readFiles(List<Path> paths) {
  paths.forEach(p -> {
    try {
      readFile(p);
    } catch (IOException e) {
      **throw****new****IOException****(e);**
    }
  });
} 
```

我们能否改变这个规则？我们能否想出一个允许从 lambda 表达式抛出检查型异常的技巧？简短的回答是：当然可以！

长答案：当然可以，*如果*我们简单地隐藏编译器对已检查异常的检查，如下所示：

```java
public final class Exceptions {
  private Exceptions() {
    throw new AssertionError("Cannot be instantiated");
  }
  public static void throwChecked(Throwable t) {
    Exceptions.<RuntimeException>throwIt(t);
  }
  @SuppressWarnings({"unchecked"})
  private static <X extends Throwable> void throwIt(
      Throwable t) throws X {
    throw (X) t;
  }
} 
```

没有其他了！现在，我们可以抛出任何已检查的异常。这里，我们抛出一个`IOException`：

```java
static void readFiles(List<Path> paths) throws IOException {
  paths.forEach(p -> {
    try {
      readFile(p);
    } catch (IOException e) { 
      Exceptions.throwChecked(new IOException(
        "Some files are corrupted", e));
    }
  });
} 
```

此外，我们可以这样捕获它：

```java
List<Path> paths = List.of(...);
try {
  readFiles(paths);
} catch (IOException e) {
  System.out.println(e + " \n " + e.getCause());
} 
```

如果某个路径未找到，则报告的错误信息将是：

```java
java.io.IOException: Some files are corrupted 
java.io.FileNotFoundException: ...
(The system cannot find the path specified) 
```

很酷，对吧？！

# 204. 为 Stream API 实现 distinctBy()

假设我们有以下模型和数据：

```java
public class Car {
  private final String brand;
  private final String fuel;
  private final int horsepower;
  ...
}
List<Car> cars = List.of(
  new Car("Chevrolet", "diesel", 350),
  ...
  new Car("Lexus", "diesel", 300)
); 
```

我们知道 Stream API 包含一个名为`distinct()`的中间操作，它能够根据`equals()`方法只保留不同的元素：

```java
cars.stream()
    .distinct()
    .forEach(System.out::println); 
```

当前的代码打印出不同的汽车，但我们可能希望有一个`distinctBy()`中间操作，它能够根据给定的属性/键只保留不同的元素。例如，我们可能需要所有品牌不同的汽车。为此，我们可以依赖`toMap()`收集器和恒等函数，如下所示：

```java
cars.stream()
    .collect(Collectors.toMap(Car::getBrand, 
             Function.identity(), (c1, c2) -> c1))
    .values()
    .forEach(System.out::println); 
```

我们可以将这个想法提取到一个辅助方法中，如下所示：

```java
public static <K, T> Collector<T, ?, Map<K, T>> 
  distinctByKey(Function<? super T, ? extends K> function) {
  return Collectors.toMap(
    function, Function.identity(), (t1, t2) -> t1);
} 
```

此外，我们还可以像下面这样使用它：

```java
cars.stream()
  .collect(Streams.distinctByKey(Car::getBrand))
  .values()
  .forEach(System.out::println); 
```

虽然这是一个很好的工作，也适用于`null`值，但我们还可以想出其他不适用于`null`值的方法。例如，我们可以依赖`ConcurrentHashMap`和`putIfAbsent()`，如下（再次，这不适用于`null`值）：

```java
public static <T> Predicate<T> distinctByKey(
    Function<? super T, ?> function) {
  Map<Object, Boolean> seen = new ConcurrentHashMap<>();
  return t -> seen.putIfAbsent(function.apply(t), 
    Boolean.TRUE) == null;
} 
```

或者，我们可以稍微优化这种方法并使用一个`Set`：

```java
public static <T> Predicate<T> distinctByKey(
    Function<? super T, ?> function) {
  Set<Object> seen = ConcurrentHashMap.newKeySet();
  return t -> seen.add(function.apply(t));
} 
```

我们可以使用以下示例中展示的这两种方法：

```java
cars.stream()
    .filter(Streams.distinctByKey(Car::getBrand))
    .forEach(System.out::println);
cars.stream()
    .filter(Streams.distinctByKey(Car::getFuel))
    .forEach(System.out::println); 
```

作为挑战，使用多个键实现一个`distinctByKeys()`操作。

# 205. 编写一个自定义收集器，它获取/跳过给定数量的元素

在*问题 202*中，我们在`MyCollectors`类中编写了一些自定义收集器。现在，让我们继续我们的旅程，并尝试在这里添加两个更多的自定义收集器，以从当前流中获取和/或保留给定数量的元素。

假设以下模型和数据：

```java
public class Car {
  private final String brand;
  private final String fuel;
  private final int horsepower;
  ...
}
List<Car> cars = List.of(
  new Car("Chevrolet", "diesel", 350),
  ... // 10 more
  new Car("Lexus", "diesel", 300)
); 
```

Stream API 提供了一个名为`limit(long n)`的中间操作，它可以用来截断流到`n`个元素。所以，如果这正是我们想要的，那么我们可以直接使用它。例如，我们可以将结果流限制在前五辆汽车，如下所示：

```java
List<Car> first5CarsLimit = cars.stream()
  .limit(5)
  .collect(Collectors.toList()); 
```

此外，Stream API 提供了一个名为`skip(long n)`的中间操作，它可以用来跳过流管道中的前`n`个元素。例如，我们可以跳过前五辆汽车，如下所示：

```java
List<Car> last5CarsSkip = cars.stream()
  .skip(5)
  .collect(Collectors.toList()); 
```

然而，有些情况下我们需要计算不同的事情，并且只收集前五个/最后一个五个结果。在这种情况下，自定义收集器是受欢迎的。

通过依赖`Collector.of()`方法（如*问题 202*中详细说明），我们可以编写一个自定义收集器，它保留/收集前`n`个元素，如下（只是为了好玩，让我们在不修改的列表中收集这些*n*个元素）：

```java
public static <T> Collector<T, List<T>, List<T>>    
    toUnmodifiableListKeep(int max) {
  return Collector.of(ArrayList::new,
    (list, value) -> {
       if (list.size() < max) {
         list.add(value);
       }
    },
    (left, right) -> {
       left.addAll(right);
       return left;
    },
    Collections::unmodifiableList);
} 
```

因此，供应商是`ArrayList::new`，累加器是`List.add()`，组合器是`List.addAll()`，而最终化器是`Collections::unmodifiableList`。基本上，累加器的任务是在达到给定的`max`值之前只累积元素。从那个点开始，不再累积任何元素。这样，我们就可以只保留前五辆车，如下所示：

```java
List<Car> first5Cars = cars.stream()
  .collect(MyCollectors.toUnmodifiableListKeep(5)); 
```

另一方面，如果我们想跳过前`n`个元素并收集剩余的元素，那么我们可以尝试累积`null`元素，直到达到给定的`index`。从那个点开始，我们开始累积真实元素。最后，最终化器移除包含`null`值的列表部分（从 0 到给定的`index`），并从剩余元素（从给定的`index`到末尾）返回一个不可修改的列表：

```java
public static <T> Collector<T, List<T>, List<T>> 
    toUnmodifiableListSkip(int index) {
  return Collector.of(ArrayList::new,
    (list, value) -> {
       if (list.size() >= index) {
         list.add(value);
       } else {
         list.add(null);
       }
    },
    (left, right) -> {
       left.addAll(right);

       return left;
    },
    list -> Collections.unmodifiableList(
      list.subList(index, list.size())));
} 
```

或者，我们可以通过使用包含结果列表和计数器的供应商类来优化这种方法。在达到给定的`index`之前，我们只需简单地增加计数器。一旦达到给定的`index`，我们就开始累积元素：

```java
public static <T> Collector<T, ?, List<T>> 
    toUnmodifiableListSkip(int index) {
  class Sublist {
    int index;
    List<T> list = new ArrayList<>(); 
  }
  return Collector.of(Sublist::new,
    (sublist, value) -> {
       if (sublist.index >= index) {
         sublist.list.add(value);
       } else {
         sublist.index++;
       }
     },
     (left, right) -> {
        left.list.addAll(right.list);
        left.index = left.index + right.index;

       return left;
     },
     sublist -> Collections.unmodifiableList(sublist.list));
} 
```

这两种方法都可以使用，如下面的示例所示：

```java
List<Car> last5Cars = cars.stream()
  .collect(MyCollectors.toUnmodifiableListSkip(5)); 
```

挑战自己实现一个在给定范围内收集的自定义收集器。

# 206. 实现一个接受五个（或任何其他任意数量）参数的函数

我们知道 Java 已经有了`java.util.function.Function`及其特殊化`java.util.function.BiFunction`。`Function`接口定义了`apply(T, t)`方法，而`BiFunction`有`apply(T t, U u)`。

在这个上下文中，我们可以定义一个`TriFunction`、`FourFunction`或（为什么不呢？）一个`FiveFunction`函数式接口，如下所示（这些都是`Function`的特殊化）：

```java
@FunctionalInterface
public interface FiveFunction <T1, T2, T3, T4, T5, R> {

  R apply(T1 t1, T2 t2, T3 t3, T4 t4, T5 t5);
} 
```

如其名所示，这个函数式接口接受五个参数。

现在，让我们来使用它！假设我们有以下模型：

```java
public class PL4 {

  private final double a;
  private final double b;
  private final double c;
  private final double d;
  private final double x;
  public PL4(double a, double b, 
             double c, double d, double x) { 
    this.a = a;
    this.b = b;
    this.c = c;
    this.d = d;
    this.x = x;
  }
  // getters
  public double compute() {
    return d + ((a - d) / (1 + (Math.pow(x / c, b))));
  }

  // equals(), hashCode(), toString()
} 
```

`compute()`方法塑造了一个称为四参数逻辑（4PL - [`www.myassays.com/four-parameter-logistic-regression.html`](https://www.myassays.com/four-parameter-logistic-regression.html)）的公式。不涉及无关细节，我们传递四个变量（`a`、`b`、`c`和`d`）作为输入，并且对于不同的`x`坐标值，我们计算`y`坐标。坐标对（`x`，`y`）描述了一条曲线（线性图形）。

我们需要为每个`x`坐标创建一个`PL4`实例，并且对于每个这样的实例，我们调用`compute()`方法。这意味着我们可以通过以下辅助方法在`Logistics`中使用`FiveFunction`接口：

```java
public final class Logistics {
  ...
  public static <T1, T2, T3, T4, X, R> R create(
      T1 t1, T2 t2, T3 t3, T4 t4, X x,
      FiveFunction<T1, T2, T3, T4, X, R> f) {

    return f.apply(t1, t2, t3, t4, x);
  }
  ...
} 
```

这充当了`PL4`的工厂：

```java
PL4 pl4_1 = Logistics.create(
    4.19, -1.10, 12.65, 0.03, 40.3, PL4::new);
PL4 pl4_2 = Logistics.create(
    4.19, -1.10, 12.65, 0.03, 100.0, PL4::new);
...
PL4 pl4_8 = Logistics.create(
    4.19, -1.10, 12.65, 0.03, 1400.6, PL4::new);
System.out.println(pl4_1.compute());
System.out.println(pl4_2.compute());
...
System.out.println(pl4_8.compute()); 
```

然而，如果我们只需要`y`坐标的列表，那么我们可以在`Logistics`中编写一个辅助方法，如下所示：

```java
public final class Logistics {
  ...
  public static <T1, T2, T3, T4, X, R> List<R> compute(
      T1 t1, T2 t2, T3 t3, T4 t4, List<X> allX,
      FiveFunction<T1, T2, T3, T4, X, R> f) {
    List<R> allY = new ArrayList<>();
    for (X x : allX) {
      allY.add(f.apply(t1, t2, t3, t4, x));
    }
    return allY;
  }
  ...
} 
```

我们可以像下面这样调用这个方法（这里我们传递了 4PL 公式，但它可以是任何具有五个`double`参数的其他公式）：

```java
FiveFunction<Double, Double, Double, Double, Double, Double> 
    pl4 = (a, b, c, d, x) -> d + ((a - d) / 
                            (1 + (Math.pow(x / c, b)))); 
List<Double> allX = List.of(40.3, 100.0, 250.2, 400.1, 
                            600.6, 800.4, 1150.4, 1400.6); 
List<Double> allY = Logistics.compute(4.19, -1.10, 12.65,
                                      0.03, allX, pl4); 
```

你可以在捆绑的代码中找到完整的示例。

# 207. 实现一个接受五个（或任何其他任意数量）参数的消费者

在继续这个问题之前，我强烈建议你阅读 *问题 206*。

编写一个接受五个参数的自定义 `Consumer` 可以这样做：

```java
@FunctionalInterface
public interface FiveConsumer <T1, T2, T3, T4, T5> {

  void accept (T1 t1, T2 t2, T3 t3, T4 t4, T5 t5);
} 
```

这是对 Java `Consumer` 的五参数特殊化，正如内置的 `BiConsumer` 是 Java `Consumer` 的双参数特殊化。

我们可以将 `FiveConsumer` 与 PL4 公式结合使用，如下（这里，我们计算 `y` 对 `x` = 40.3）：

```java
FiveConsumer<Double, Double, Double, Double, Double> 
  pl4c = (a, b, c, d, x) -> Logistics.pl4(a, b, c, d, x);

pl4c.accept(4.19, -1.10, 12.65, 0.03, 40.3); 
```

`Logistics.pl4()` 是包含公式并显示结果的方法：

```java
public static void pl4(Double a, Double b, 
                       Double c, Double d, Double x) {

  System.out.println(d + ((a - d) / (1 
                       + (Math.pow(x / c, b)))));
} 
```

接下来，让我们看看我们如何部分应用一个 `Function`。

# 208. 部分应用一个 Function

部分应用的 `Function` 是只应用其部分参数的 `Function`，返回另一个 `Function`。例如，这里有一个包含 `apply()` 方法的 `TriFunction`（一个具有三个参数的函数式函数），旁边有两个部分应用此函数的 `default` 方法：

```java
@FunctionalInterface
public interface TriFunction <T1, T2, T3, R> {

  R apply(T1 t1, T2 t2, T3 t3);

  default BiFunction<T2, T3, R> applyOnly(T1 t1) {
    return (t2, t3) -> apply(t1, t2, t3);
  }

  default Function<T3, R> applyOnly(T1 t1, T2 t2) {
    return (t3) -> apply(t1, t2, t3);
  }
} 
```

如你所见，`applyOnly(T1 t1)` 只应用 `t1` 参数并返回一个 `BiFunction`。另一方面，`applyOnly(T1 t1, T2 t2)` 只应用 `t1` 和 `t2`，返回一个 `Function`。

让我们看看我们如何使用这些方法。例如，让我们考虑公式 (a+b+c)² = a²+b²+c²+2ab+2bc+2ca，它可以通过 `TriFunction` 来实现，如下所示：

```java
TriFunction<Double, Double, Double, Double> abc2 = (a, b, c)
  -> Math.pow(a, 2) + Math.pow(b, 2) + Math.pow(c, 2) 
     + 2.0*a*b + 2*b*c + 2*c*a;        
System.out.println("abc2 (1): " + abc2.apply(1.0, 2.0, 1.0));
System.out.println("abc2 (2): " + abc2.apply(1.0, 2.0, 2.0));
System.out.println("abc2 (3): " + abc2.apply(1.0, 2.0, 3.0)); 
```

在这里，我们调用了 `apply(T1 t1, T2 t2, T3 t3)` 三次。正如你所见，每次调用中只有 `c` 项有不同的值，而 `a` 和 `b` 分别始终等于 1.0 和 2.0。这意味着我们可以为 `a` 和 `b` 使用 `apply(T1 t1, T2 t2)`，为 `c` 使用 `apply(T1 t1)`，如下所示：

```java
Function<Double, Double> abc2Only1 = abc2.applyOnly(1.0, 2.0);

System.out.println("abc2Only1 (1): " + abc2Only1.apply(1.0));
System.out.println("abc2Only1 (2): " + abc2Only1.apply(2.0));
System.out.println("abc2Only1 (3): " + abc2Only1.apply(3.0)); 
```

如果我们假设只有 `a` 是常数（1.0），而 `b` 和 `c` 每次调用有不同的值，那么我们可以为 `a` 使用 `apply(T1 t1)`，为 `b` 和 `c` 使用 `apply(T1 t1, T2 t2)`，如下所示：

```java
BiFunction<Double, Double, Double> abc2Only2 
  = abc2.applyOnly(1.0);

System.out.println("abc2Only2 (1): " 
  + abc2Only2.apply(2.0, 3.0));
System.out.println("abc2Only2 (2): " 
  + abc2Only2.apply(1.0, 2.0));
System.out.println("abc2Only2 (3): " 
  + abc2Only2.apply(3.0, 2.0)); 
```

任务完成！

# 摘要

本章涵盖了 24 个问题。大多数问题集中在使用谓词、函数和收集器，但我们还涵盖了 JDK 16 的 `mapMulti()` 操作，将命令式代码重构为函数式代码，等等。

# 加入我们的 Discord 社区

加入我们社区的 Discord 空间，与作者和其他读者进行讨论：

[`discord.gg/8mgytp5DGQ`](https://discord.gg/8mgytp5DGQ)

![](img/QR_Code1139613064111216156.png)
