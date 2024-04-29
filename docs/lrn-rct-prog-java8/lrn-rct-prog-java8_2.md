# 第二章：使用 Java 8 的函数式构造

函数式编程并不是一个新的想法；实际上，它相当古老。例如，**Lisp**是一种函数式语言，是当今常用编程语言中第二古老的语言。

函数式程序是使用可重用的纯函数（lambda）构建的。程序逻辑由小的声明性步骤组成，而不是复杂的算法。这是因为函数式程序最小化了状态的使用，这使得命令式程序复杂且难以重构/支持。

Java 8 带来了 lambda 表达式和将函数传递给函数的能力。有了它们，我们可以以更函数式的风格编码，并摆脱大量的样板代码。Java 8 带来的另一个新功能是流——与 RxJava 的可观察对象非常相似，但不是异步的。结合这些流和 lambda，我们能够创建更类似函数式的程序。

我们将熟悉这些新的构造，并看看它们如何与 RxJava 的抽象一起使用。通过使用 lambda，我们的程序将更简单，更容易跟踪，并且本章介绍的概念将有助于设计应用程序。

本章涵盖：

+   Java 8 中的 Lambda

+   使用 lambda 语法的第一个 RxJava 示例

+   纯函数和高阶函数是什么

# Java 8 中的 Lambda

Java 8 中最重要的变化是引入了 lambda 表达式。它们使编码更快，更清晰，并且可以使用函数式编程。

Java 是在 90 年代作为面向对象的编程语言创建的，其思想是一切都应该是一个对象。那时，面向对象编程是软件开发的主要范式。但是，最近，函数式编程因其适用于并发和事件驱动编程而变得越来越受欢迎。这并不意味着我们应该停止使用面向对象的语言编写代码。相反，最好的策略是将面向对象和函数式编程的元素混合在一起。将 lambda 添加到 Java 8 符合这个想法——Java 是一种面向对象的语言，但现在它有了 lambda，我们也能够使用函数式风格编码。

让我们详细看看这个新功能。

## 介绍新的语法和语义

为了介绍 lambda 表达式，我们需要看到它们的实际价值。这就是为什么本章将以一个不使用 lambda 表达式实现的示例开始，然后重新使用 lambda 表达式实现相同的示例。

还记得`Observable`类中的`map(Func1)`方法吗？让我们尝试为`java.util.List`集合实现类似的东西。当然，Java 不支持向现有类添加方法，因此实现将是一个接受列表和转换并返回包含转换元素的新列表的静态方法。为了将转换传递给方法，我们将需要一个表示它的方法的接口。

让我们来看看代码：

```java
interface Mapper<V, M> { // (1)
  M map(V value); // (2)
}

// (3)	
public static <V, M> List<M> map(List<V> list, Mapper<V, M> mapper) {
  List<M> mapped = new ArrayList<M>(list.size()); // (4)
  for (V v : list) {
    mapped.add(mapper.map(v)); // (5)
  }
  return mapped; // (6)
}
```

这里发生了什么？

1.  我们定义了一个名为`Mapper`的通用接口。

1.  它只有一个方法，`M map(V)`，它接收一个类型为`V`的值并将其转换为类型为`M`的值。

1.  静态方法`List<M> map(List<V>, Mapper<V, M>)`接受一个类型为`V`的元素列表和一个`Mapper`实现。使用这个`Mapper`实现的`map()`方法对源列表的每个元素进行转换，将列表转换为包含转换元素的新类型为`M`的列表。

1.  该实现创建一个新的空类型为`M`的列表，其大小与源列表相同。

1.  使用传递的`Mapper`实现转换源列表中的每个元素，并将其添加到新列表中。

1.  返回新列表。

在这个实现中，每当我们想通过转换另一个列表创建一个新列表时，我们都必须使用正确的转换来实现`Mapper`接口。直到 Java 8，将自定义逻辑传递给方法的正确方式正是这样——使用匿名类实例，实现给定的方法。

但让我们看看我们如何使用这个`List<M> map(List<V>, Mapper<V, M>)`方法：

```java
List<Integer> mapped = map(numbers, new Mapper<Integer, Integer>() {
  @Override
  public Integer map(Integer value) {
    return value * value; // actual mapping
  }
});
```

为了对列表应用映射，我们需要编写四行样板代码。实际的映射非常简单，只有其中一行。真正的问题在于，我们传递的不是一个操作，而是一个对象。这掩盖了这个程序的真正意图——传递一个从源列表的每个项目产生转换的操作，并在最后得到一个应用了变化的列表。

这是使用 Java 8 的新 lambda 语法进行的调用的样子：

```java
List<Integer> mapped = map(numbers, value -> value * value);
```

相当直接了当，不是吗？它只是起作用。我们不是传递一个对象并实现一个接口，而是传递一块代码，一个无名函数。

发生了什么？我们定义了一个任意的接口和一个任意的方法，但我们可以在接口的实例位置传递这个 lambda。在 Java 8 中，如果您定义了*只有一个抽象方法的接口*，并且创建了一个接收此类型接口参数的方法，那么您可以传递 lambda。如果接口的单个方法接受两个字符串类型的参数并返回整数值，那么 lambda 将必须由`->`之前的两个参数组成，并且为了返回整数，参数将被推断为字符串。

这种类型的接口称为**功能接口。**单个方法是抽象的而不是默认的非常重要。Java 8 中的另一件新事物是接口的默认方法：

```java
interface Program {
  default String fromChapter() {
    return "Two";
  }
}
```

默认方法在更改已经存在的接口时非常有用。当我们向它们添加默认方法时，实现它们的类不会中断。只有一个默认方法的接口不是功能性的；单个方法不应该是默认的。

Lambda 充当功能接口的实现。因此，可以将它们分配给接口类型的变量，如下所示：

```java
Mapper<Integer, Integer> square = (value) -> value * value;
```

我们可以重复使用 square 对象，因为它是`Mapper`接口的实现。

也许您已经注意到了，但在目前为止的例子中，lambda 表达式的参数没有类型。那是因为类型是被推断的。因此，这个表达式与前面的表达式完全相同：

```java
Mapper<Integer, Integer> square = (Integer value) -> value * value;
```

没有类型的 lambda 表达式的参数是如何工作的并不是魔术。Java 是一种静态类型语言，因此功能接口的单个方法的参数用于类型检查。

那么 lambda 表达式的主体呢？任何地方都没有`return`语句。事实证明，这两个例子完全相同：

```java
Mapper<Integer, Integer> square = (value) -> value * value;
// and
Mapper<Integer, Integer> square = (value) -> {
 return value * value;
};

```

第一个表达式只是第二个的简写形式。最好 lambda 只有一行代码。但是如果 lambda 表达式包含多行，定义它的唯一方法是使用第二种方法，就像这样：

```java
Mapper<Integer, Integer> square = (value) -> {
  System.out.println("Calculating the square of " + value);
  return value * value;
};
```

在底层，lambda 表达式不仅仅是匿名内部类的语法糖。它们被实现为在**Java 虚拟机**（**JVM**）内快速执行，因此如果您的代码只设计为与 Java 8+兼容，那么您应该绝对使用它们。它们的主要思想是以与数据传递相同的方式传递行为。这使得您的程序更易读。

与新语法相关的最后一件事是能够传递到方法并分配给已定义的函数和方法。让我们定义一个新的功能接口：

```java
interface Action<V> {
  void act(V value);
}
```

我们可以使用它来对列表中的每个值执行任意操作；例如，记录列表。以下是使用此接口的方法：

```java
public static <V> void act(List<V> list, Action<V> action) {
  for (V v : list) {
    action.act(v);
  }
}
```

这个方法类似于`map()`函数。它遍历列表并在每个元素上调用传递的动作的`act()`方法。让我们使用一个简单记录列表中元素的 lambda 来调用它：

```java
act(list, value -> System.out.println(value));
```

这很简单，但不是必需的，因为`println()`方法本身可以直接传递给`act()`方法。这样做如下：

```java
act(list, System.out::println);
```

### 注意

这些示例的代码可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter02/Java8LambdasSyntaxIntroduction.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter02/Java8LambdasSyntaxIntroduction.java)上查看/下载。

这是 Java 8 中的有效语法——每个方法都可以成为 lambda，并且可以分配给一个变量或传递给一个方法。所有这些都是有效的：

+   Book::makeBook // 类的静态方法

+   book::read // 实例的方法

+   Book::new // 类的构造函数

+   Book::read // 实例方法，但在没有使用实际实例的情况下引用。

现在我们已经揭示了 lambda 语法，我们将在 RxJava 示例中使用它，而不是匿名内部类。

## Java 8 和 RxJava 中的函数接口

Java 8 带有一个特殊的包，其中包含常见情况的函数接口。这个包是`java.util.function`，我们不会在本书中详细介绍它，但会介绍一些值得一提的接口：

+   `Consumer<T>`：这代表接受一个参数并返回空的函数。它的抽象方法是`void accept(T)`。例如，我们可以将`System.out::println`方法分配给一个变量，如下所示：

```java
Consumer<String> print = System.out::println;
```

+   `Function<T,R>`：这代表接受给定类型的一个参数并返回任意类型结果的函数。它的抽象方法是`R accept(T)`，可以用于映射。我们根本不需要`Mapper`接口！让我们看一下以下代码片段：

```java
Function<Integer, String> toStr = (value) -> (value + "!");
List<String> string = map(integers, toStr);
```

+   `Predicate<T>`：这代表只有一个参数并返回布尔结果的函数。它的抽象方法是`boolean test(T)`，可以用于过滤。让我们看一下以下代码：

```java
Predicate<Integer> odd = (value) -> value % 2 != 0;
```

还有许多类似的函数接口；例如，带有两个参数的函数，或者二元运算符。这又是一个带有两个参数的函数，但两个参数类型相同，并返回相同类型的结果。它们有助于在我们的代码中重用 lambda。

好处是 RxJava 与 lambda 兼容。这意味着我们传递给`subscribe`方法的动作实际上是函数接口！

RxJava 的函数接口在`rx.functions`包中。它们都扩展了一个基本的**标记** **接口**（没有方法的接口，用于类型检查），称为`Function`。此外，还有另一个标记接口，扩展了`Function`，称为`Action`。它用于标记消费者（返回空的函数）。

RxJava 有十一个`Action`接口：

```java
Action0 // Action with no parameters
Action1<T1> // Action with one parameter
Action2<T1,T2> // Action with two parameters
Action9<T1,T2,T3,T4,T5,T6,T7,T8,T9> // Action with nine parameters
ActionN // Action with arbitrary number of parameters
```

它们主要用于订阅(`Action1`和`Action0`)。我们在第一章中看到的`Observable.OnSubscribe<T>`参数（用于创建自定义可观察对象）也扩展了`Action`接口。

类似地，有十一个`Function`扩展器代表返回结果的函数。它们是`Func0<R>`，`Func1<T1, R>`... `Func9<T1,T2,T3,T4,T5,T6,T7,T8,T9,R>`和`FuncN<R>`。它们用于映射、过滤、组合和许多其他目的。

RxJava 中的每个操作符和订阅方法都适用于一个或多个这些接口。这意味着我们几乎可以在 RxJava 的任何地方使用 lambda 表达式代替匿名内部类。从这一点开始，我们所有的示例都将使用 lambda，以便更易读和有些函数式。

现在，让我们看一个使用 lambda 实现的大型 RxJava 示例。这是我们熟悉的响应式求和示例！

# 使用 lambda 实现响应式求和示例

因此，这次，我们的主要代码片段将与之前的相似：

```java
ConnectableObservable<String> input = CreateObservable.from(System.in);

Observable<Double> a = varStream("a", input);
Observable<Double> b = varStream("b", input);

reactiveSum(a, b); // The difference

input.connect();
```

唯一的区别是我们将采用更加功能性的方法来计算我们的总和，而不是保持相同的状态。我们不会实现`Observer`接口；相反，我们将传递 lambda 来订阅。这个解决方案更加清晰。

`CreateObservable.from(InputStream)`方法与我们之前使用的非常相似。我们将跳过它，看看`Observable<Double> varStream(String, Observable<String>)`方法，它创建了代表收集器的`Observable`实例：

```java
public static Observable<Double> varStream(
  final String name, Observable<String> input) {
    final Pattern pattern =     Pattern.compile(
      "\\s*" + name + "\\s*[:|=]\\s*(-?\\d+\\.?\\d*)$"
    );
    return input
    .map(pattern::matcher) // (1)
 .filter(m -> m.matches() && m.group(1) != null) // (2)
 .map(matcher -> matcher.group(1)) // (3)
 .map(Double::parseDouble); // (4)
  }
)
```

这个方法比以前使用的要短得多，看起来更简单。但从语义上讲，它是相同的。它创建了一个与源可观察对象连接的`Observable`实例，该源可观察对象产生任意字符串，如果字符串符合它期望的格式，它会提取出一个双精度数并发出这个数字。负责检查输入格式和提取数字的逻辑只有四行，由简单的 lambda 表示。让我们来看一下：

1.  我们映射一个 lambda，使用预期的模式和输入字符串创建一个`matcher`实例。

1.  使用`filter()`方法，只过滤正确格式的输入。

1.  使用`map()`操作符，我们从`matcher`实例中创建一个字符串，其中只包含我们需要的数字数据。

1.  再次使用`map()`操作符，将字符串转换为双精度数。

至于新的`void reactiveSum(Observable<Double>, Observable<Double>)`方法的实现，请使用以下代码：

```java
public static void reactiveSum(
  Observable<Double> a,
  Observable<Double> b) {
    Observable
      .combineLatest(a, b, (x, y) -> x + y) // (1)
 .subscribe( // (2)
 sum -> System.out.println("update : a + b = " + sum),
 error -> {
 System.out.println("Got an error!");
 error.printStackTrace();
 },
 () -> System.out.println("Exiting...")
 );
}
```

让我们看一下以下代码：

1.  再次使用`combineLatest()`方法，但这次第三个参数是一个简单的 lambda，实现了求和。

1.  `subscribe()`方法接受三个 lambda 表达式，当发生以下事件时触发：

+   总和改变了

+   有一个错误

+   程序即将完成

### 注意

此示例的源代码可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter02/ReactiveSumV2.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter02/ReactiveSumV2.java)上查看/下载。

使用 lambda 使一切变得更简单。看看前面的程序，我们可以看到大部分逻辑由小型独立函数组成，使用其他函数链接在一起。这就是我们所说的功能性，使用这样的小型可重用函数来表达我们的程序，这些函数接受其他函数并返回函数和数据抽象，使用函数链转换输入数据以产生所需的结果。但让我们深入研究这些函数。

# 纯函数和高阶函数

您不必记住本章介绍的大部分术语；重要的是要理解它们如何帮助我们编写简单但功能强大的程序。

RxJava 的方法融合了许多功能性思想，因此重要的是我们学会如何以更加功能性的方式思考，以便编写更好的响应式应用程序。

## 纯函数

**纯函数**是一个其返回值仅由其输入决定的函数，没有可观察的**副作用**。如果我们用相同的参数调用它*n*次，每次都会得到相同的结果。例如：

```java
Predicate<Integer> even = (number) -> number % 2 == 0;
int i = 50;
while((i--) > 0) {
  System.out.println("Is five even? - " + even.test(5));
}
```

每次，偶函数返回`False`，因为它*仅依赖于其输入*，而每次输入都是相同的，甚至不是。

纯函数的这个特性称为**幂等性**。幂等函数不依赖于时间，因此它们可以将连续数据视为无限数据流。这就是 RxJava（`Observable`实例）中表示不断变化的数据的方式。

### 注意

请注意，在这里，“幂等性”一词是以计算机科学的意义使用的。在计算中，幂等操作是指如果使用相同的输入参数多次调用它，它不会产生额外的效果；在数学中，幂等操作是指满足这个表达式的操作：*f(f(x)) = f(x)*。

纯函数*不会产生副作用*。例如：

```java
Predicate<Integer> impureEven = (number) -> {
  System.out.println("Printing here is side effect!");
  return number % 2 == 0;
};
```

这个函数不是纯的，因为每次调用它时都会在输出上打印一条消息。所以它做了两件事：它测试数字是否为偶数，并且作为副作用输出一条消息。副作用是函数可以产生的任何可能的可观察输出，例如触发事件、抛出异常和 I/O，与其返回值不同。副作用还会改变共享状态或可变参数。

想想看。如果你的大部分程序由纯函数组成，它将很容易扩展，并且可以并行运行部分，因为纯函数不会相互冲突，也不会改变共享状态。

在本节中值得一提的另一件事是**不可变性**。不可变对象是指不能改变其状态的对象。Java 中的`String`类就是一个很好的例子。`String`实例是不可变的；即使像`substring`这样的方法也会创建一个新的`String`实例，而不会修改调用它的实例。

如果我们将不可变数据传递给纯函数，我们可以确保每次使用这些数据调用它时，它都会返回相同的结果。对于**可变**对象，在编写并行程序时情况就不太一样了，因为一个线程可以改变对象的状态。在这种情况下，如果调用纯函数，它将返回不同的结果，因此不再是幂等的。

如果我们将数据存储在不可变对象中，并使用纯函数对其进行操作，在此过程中创建新的不可变对象，我们将不会受到意外并发问题的影响。不会有全局状态和可变状态；一切都将简单而可预测。

使用不可变对象是棘手的；对它们的每个操作都会创建新的实例，这可能会消耗内存。但有方法可以避免这种情况；例如，尽可能多地重用源不可变对象，或使不可变对象的生命周期尽可能短（因为生命周期短的对象对 GC 或缓存友好）。函数式程序应该设计为使用不可变的无状态数据。

复杂的程序不能只由纯函数组成，但只要可能，最好使用它们。在本章对*The Reactive Sum*的实现中，我们只传递了纯函数给`map()`、`filter()`和`combineLatest()`。

谈到`map()`和`filter()`函数，我们称它们为高阶函数。

## 高阶函数

至少有一个函数类型参数或返回函数的函数被称为**高阶函数**。当然，*高阶函数可以是纯的*。

这是一个接受函数参数的高阶函数的例子：

```java
public static <T, R> int highSum(
  Function<T, Integer> f1,
  Function<R, Integer> f2,
  T data1,
  R data2) {
    return f1.apply(data1) + f2.apply(data2);
  }
)
```

它需要两个类型为`T -> int/R -> int`的函数和一些数据来调用它们并对它们的结果求和。例如，我们可以这样做：

```java
highSum(v -> v * v, v -> v * v * v, 3, 2);
```

这里我们对三的平方和两的立方求和。

但高阶函数的理念是灵活的。例如，我们可以使用`highSum()`函数来完成完全不同的目的，比如对字符串求和，如下所示：

```java
Function<String, Integer> strToInt = s -> Integer.parseInt(s);

highSum(strToInt, strToInt, "4",  "5");
```

因此，高阶函数可以用于将相同的行为应用于不同类型的输入。

如果我们传递给`highSum()`函数的前两个参数是纯函数，那么它也将是一个纯函数。`strToInt`参数是一个纯函数，如果我们调用`highSum(strToInt, strToInt, "4", "5")`方法*n*次，它将返回相同的结果，并且不会产生副作用。

这是另一个高阶函数的例子：

```java
public static Function<String, String> greet(String greeting) {
  return (String name) -> greeting + " " + name + "!";
}
```

这是一个返回另一个函数的函数。它可以这样使用：

```java
System.out.println(greet("Hello").apply("world"));
// Prints 'Hellow world!'

System.out.println(greet("Goodbye").apply("cruel world"));
// Prints 'Goodbye cruel world!'

Function<String, String> howdy = greet("Howdy");

System.out.println(howdy.apply("Tanya"));
System.out.println(howdy.apply("Dali"));
// These two print 'Howdy Tanya!' and 'Howdy Dali'
```

### 注意

此示例的代码可以在[`github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter02/PureAndHigherOrderFunctions.java`](https://github.com/meddle0x53/learning-rxjava/blob/master/src/main/java/com/packtpub/reactive/chapter02/PureAndHigherOrderFunctions.java)找到。

这些函数可以用来实现具有共同点的不同行为。在面向对象编程中，我们定义类然后扩展它们，重载它们的方法。在函数式编程中，我们将高阶函数定义为接口，并使用不同的参数调用它们，从而产生不同的行为。

这些函数是*一等公民*；我们可以仅使用函数编写我们的逻辑，将它们链接在一起，并处理我们的数据，将其转换、过滤或累积成一个结果。

## RxJava 和函数式编程

纯函数和高阶函数等函数式概念对 RxJava 非常重要。RxJava 的`Observable`类是*流畅接口*的一种实现。这意味着它的大多数实例方法都返回一个`Observable`实例。例如：

```java
Observable mapped = observable.map(someFunction);
```

`map()`操作符返回一个新的`Observable`实例，发出经过转换的数据。诸如`map()`之类的操作符显然是高阶函数，我们可以向它们传递其他函数。因此，典型的 RxJava 程序由一系列操作符链接到一个`Observable`实例表示，多个*订阅者*可以订阅它。这些链接在一起的函数可以受益于本章涵盖的主题。我们可以向它们传递 lambda 而不是匿名接口实现（就像我们在*Reactive Sum*的第二个实现中看到的那样），并且我们应该尽可能使用不可变数据和纯函数。这样，我们的代码将会简单且安全。

# 总结

在本章中，我们已经了解了一些函数式编程原则和术语。我们学会了如何编写由小的纯函数动作组成的程序，使用高阶函数链接在一起。

随着函数式编程的日益流行，精通它的开发人员将在不久的将来需求量很大。这是因为它帮助我们轻松实现可伸缩性和并行性。而且，如果我们将响应式思想加入其中，它将变得更加吸引人。

这就是为什么我们将在接下来的章节中深入研究 RxJava 框架，学习如何将其用于我们的利益。我们将从`Observable`实例创建技术开始。这将使我们具备从任何东西创建`Observable`实例的技能，从而将几乎一切转变为函数式响应式程序。
