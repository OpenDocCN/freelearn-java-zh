# *第十三章*：函数式编程

本章将带你进入函数式编程的世界。它解释了什么是函数式接口，概述了随 JDK 一起提供的函数式接口，并定义和演示了 Lambda 表达式以及如何使用函数式接口使用它们，包括使用**方法引用**。

本章将涵盖以下主题：

+   什么是函数式编程？

+   标准函数式接口

+   功能型管道

+   Lambda 表达式限制

+   方法引用

到本章结束时，你将能够编写函数并将它们用于 Lambda 表达式，以便将它们作为方法参数传递。

# 技术要求

要能够执行本章提供的代码示例，你需要以下内容：

+   一台装有 Microsoft Windows、Apple macOS 或 Linux 操作系统的计算机

+   Java SE 版本 17 或更高版本

+   你喜欢的 IDE 或任何代码编辑器

在*第一章*，“开始使用 Java 17”中，提供了如何设置 Java SE 和 IntelliJ IDEA 编辑器的说明。本章的代码示例文件可在 GitHub 上找到，网址为 [`github.com/PacktPublishing/Learn-Java-17-Programming.git`](https://github.com/PacktPublishing/Learn-Java-17-Programming.git)，在 `examples/src/main/java/com/packt/learnjava/ch13_functional` 文件夹中。

# 什么是函数式编程？

在我们提供定义之前，让我们回顾一下在前几章中已经使用过的具有函数式编程元素的代码。所有这些示例都给你一个很好的想法，了解一个函数是如何构建并作为参数传递的。

在*第六章*，“数据结构、泛型和常用工具”中，我们讨论了 `Iterable` 接口及其 `default void forEach (Consumer<T> function)` 方法，并提供了以下示例：

```java
Iterable<String> list = List.of("s1", "s2", "s3");
```

```java
System.out.println(list);                //prints: [s1, s2, s3]
```

```java
list.forEach(e -> System.out.print(e + " "));//prints: s1 s2 s3
```

你可以看到一个 `Consumer e -> System.out.print(e + " ")` 函数是如何传递到 `forEach()` 方法并应用于从列表流入此方法的每个元素的。我们将在稍后讨论 `Consumer` 函数。

我们还提到了 `Collection` 接口接受函数作为参数的两个方法：

+   `default boolean remove(Predicate<E> filter)` 方法尝试从集合中删除所有满足给定谓词的元素；一个 `Predicate` 函数接受集合中的一个元素并返回一个布尔值。

+   `default T[] toArray(IntFunction<T[]> generator)` 方法返回一个包含集合中所有元素的数组，使用提供的 `IntFunction` 生成器函数来分配返回的数组。

在同一章中，我们还提到了 `List` 接口的方法：

+   `default void replaceAll(UnaryOperator<E> operator)`: 这会将列表中的每个元素替换为应用提供的 `UnaryOperator` 到该元素的结果；`UnaryOperator` 是我们将在本章中要审查的函数之一。

我们描述了 `Map` 接口，其 `default V merge(K key, V value, BiFunction<V,V,V> remappingFunction)` 方法，以及它如何用于连接 `String` 值：`map.merge(key, value, String::concat)`。`BiFunction<V,V,V>` 接受两个相同类型的参数，并返回相同类型的值。`String::concat` 构造称为 **方法引用**，将在 *方法引用* 部分进行解释。

我们提供了以下传递 `Comparator` 函数的示例：

```java
  list.sort(Comparator.naturalOrder());
```

```java
  Comparator<String> cmp = 
```

```java
               (s1, s2) -> s1 == null ? -1 : s1.compareTo(s2);
```

```java
  list.sort(cmp);
```

它接受两个 `String` 参数，然后比较第一个参数与 `null`。如果第一个参数是 `null`，则函数返回 `-1`；否则，它使用 `compareTo()` 方法比较第一个参数和第二个参数。

在 *第十一章*，*网络编程* 中，我们查看以下代码：

```java
  HttpClient httpClient = HttpClient.newBuilder().build();
```

```java
  HttpRequest req = HttpRequest.newBuilder()
```

```java
   .uri(URI.create("http://localhost:3333/something")).build();
```

```java
  try {
```

```java
     HttpResponse<String> resp = 
```

```java
                 httpClient.send(req, BodyHandlers.ofString());
```

```java
     System.out.println("Response: " + 
```

```java
                      resp.statusCode() + " : " + resp.body());
```

```java
  } catch (Exception ex) {
```

```java
     ex.printStackTrace();
```

```java
  }
```

`BodyHandler` 对象（一个函数）由 `BodyHandlers.ofString()` 工厂方法生成，并作为参数传递给 `send()` 方法。在方法内部，代码调用其 `apply()` 方法：

```java
BodySubscriber<T> apply(ResponseInfo responseInfo)
```

最后，在 *第十二章*，*Java GUI 编程* 中，我们使用了 `EventHandler` 函数作为以下代码片段的参数：

```java
  btn.setOnAction(e -> { 
```

```java
                     System.out.println("Bye! See you later!");
```

```java
                     Platform.exit();
```

```java
                 });
```

```java
  primaryStage.onCloseRequestProperty()
```

```java
     .setValue(e -> System.out.println("Bye! See you later!"));
```

第一个函数是 `EventHandler<ActionEvent>`。这个函数会打印一条消息并强制应用程序退出。第二个是 `EventHandler<WindowEvent>` 函数。它只是打印一条消息。

将函数作为参数传递的能力构成了函数式编程。这在许多编程语言中都有体现，并且不需要管理对象状态。函数是无状态的。它的结果仅取决于输入数据，无论调用多少次。这种编码使得结果更加可预测，这是函数式编程最具吸引力的方面。

从这种设计中获得最大好处的是并行数据处理。函数式编程允许将并行责任从客户端代码转移到库。在此之前，为了处理 Java 集合的元素，客户端代码必须遍历集合并组织处理。在 Java 8 中，添加了新的（默认）方法，这些方法接受一个函数作为参数，然后将其应用于集合的每个元素，是否并行取决于内部处理算法。因此，组织并行处理的责任在于库。

## 什么是函数式接口？

当我们定义一个函数时，我们提供了一个只包含一个抽象方法的接口实现。这就是 Java 编译器知道将提供的功能放在哪里的方式。编译器查看接口（如前例中的`Consumer`、`Predicate`、`Comparator`、`IntFunction`、`UnaryOperator`、`BiFunction`、`BodyHandler`和`EventHandler`），在那里只看到一个抽象方法，并使用传入的功能作为方法实现。唯一的要求是传入的参数必须与方法签名匹配。否则，将生成编译时错误。

因此，任何只有一个抽象方法的接口都被称为**函数式接口**。请注意，**只有一个抽象方法**的要求包括从父接口继承的方法。例如，考虑以下接口：

```java
@FunctionalInterface
```

```java
interface A {
```

```java
    void method1();
```

```java
    default void method2(){}
```

```java
    static void method3(){}
```

```java
}
```

```java
@FunctionalInterface
```

```java
interface B extends A {
```

```java
    default void method4(){}
```

```java
}
```

```java
@FunctionalInterface
```

```java
interface C extends B {
```

```java
    void method1();
```

```java
}
```

```java
//@FunctionalInterface 
```

```java
interface D extends C {
```

```java
    void method5();
```

```java
}
```

`A`接口是一个函数式接口，因为它只有一个抽象方法`method1()`。`B`接口也是一个函数式接口，因为它只有一个抽象方法——从`A`接口继承的同一个方法。`C`接口是一个函数式接口，因为它只有一个抽象方法`method1()`，它覆盖了父接口`A`的抽象方法。`D`接口不能是一个函数式接口，因为它有两个抽象方法——从父接口`A`继承的`method1()`和`method5()`。

为了帮助避免运行时错误，Java 8 中引入了`@FunctionalInterface`注解。它告诉编译器意图，以便编译器可以检查并查看注解接口中是否确实只有一个抽象方法。此注解还警告阅读代码的程序员，这个接口有意只有一个抽象方法。否则，程序员可能会浪费时间向接口添加另一个抽象方法，结果在运行时发现无法实现。

同样的原因，自 Java 早期版本以来就存在的`Runnable`和`Callable`接口在 Java 8 中被标注为`@FunctionalInterface`。这种区分是明确的，并作为对用户的提醒，这些接口可以用来创建函数：

```java
@FunctionalInterface
```

```java
interface Runnable {
```

```java
    void run(); 
```

```java
}
```

```java
@FunctionalInterface
```

```java
interface Callable<V> {
```

```java
    V call() throws Exception;
```

```java
}
```

与任何其他接口一样，函数式接口可以使用`匿名类`来实现：

```java
Runnable runnable = new Runnable() {
```

```java
    @Override
```

```java
    public void run() {
```

```java
        System.out.println("Hello!");
```

```java
    }
```

```java
};
```

以这种方式创建的对象可以后来按以下方式使用：

```java
runnable.run();   //prints: Hello!
```

如果我们仔细查看前面的代码，我们会注意到存在不必要的开销。首先，没有必要重复接口名称，因为我们已经将其声明为对象引用的类型。其次，对于只有一个抽象方法的函数式接口，没有必要指定必须实现的方法名称。编译器和 Java 运行时可以自己推断出来。我们需要的只是提供新的功能。Lambda 表达式正是为了这个目的而引入的。

## 什么是 Lambda 表达式？

术语 **Lambda** 来自于 *lambda 演算*——一种通用的计算模型，可以用来模拟任何图灵机。它在 20 世纪 30 年代由数学家 Alonzo Church 提出。**Lambda 表达式**是一个函数，在 Java 中实现为匿名方法。它还允许省略修饰符、返回类型和参数类型。这使得符号非常紧凑。

Lambda 表达式的语法包括参数列表、箭头符号 (`->`) 和主体。参数列表可以是空的，例如 `()`，如果没有括号（如果只有一个参数），或者是一个用括号包围的、以逗号分隔的参数列表。主体可以是一个单独的表达式，或者是一个花括号 `{}` 内的语句块。让我们看看几个例子：

+   `() -> 42;` 总是返回 `42`。

+   `x -> x*42 + 42;` 将 `x` 的值乘以 `42`，然后将 `42` 添加到结果中并返回。

+   `(x, y) -> x * y;` 将传入的参数相乘并返回结果。

+   `s -> "abc".equals(s);` 比较 `s` 变量和字面量 `"abc"` 的值；它返回一个布尔结果值。

+   `s -> System.out.println("x=" + s);` 打印带有前缀 `"x="` 的 `s` 值。

+   `(i, s) -> { i++; System.out.println(s + "=" + i); };` 增加输入整数，并使用前缀 `s +=` 打印新的值，其中 `s` 是第二个参数的值。

在没有函数式编程的情况下，Java 中传递某些功能作为参数的唯一方法是通过编写一个实现接口的类，创建其对象，然后将其作为参数传递。但即使是使用匿名类这种最少介入的风格，也需要编写太多的样板代码。使用函数式接口和 Lambda 表达式可以使代码更短、更清晰、更具表现力。

例如，Lambda 表达式允许我们使用 `Runnable` 接口重新实现前面的例子，如下所示：

```java
Runnable runnable = () -> System.out.println("Hello!");
```

正如你所见，创建一个函数式接口很容易，尤其是在使用 Lambda 表达式时。但在这样做之前，考虑使用 `java.util.function` 包中提供的 43 个函数式接口之一。这不仅可以让你的代码更简洁，而且还能帮助熟悉标准接口的其他程序员更好地理解你的代码。

### Lambda 参数的局部变量语法

直到 Java 11 的发布，声明参数类型有两种方式——显式和隐式。以下是一个显式版本：

```java
  BiFunction<Double, Integer, Double> f = 
```

```java
                           (Double x, Integer y) -> x / y;
```

```java
  System.out.println(f.apply(3., 2)); //prints: 1.5
```

以下是一个隐式参数类型定义：

```java
  BiFunction<Double, Integer, Double> f = (x, y) -> x / y;
```

```java
  System.out.println(f.apply(3., 2));   //prints: 1.5
```

在前面的代码中，编译器从接口定义中推断参数的类型。

在 Java 11 中，引入了另一种使用 `var` 类型持有器的参数类型声明方法，这与 Java 10 中引入的 `var` 局部变量类型持有器类似（参见 *第一章*，*Java 17 入门*）。

以下参数声明在 Java 11 之前的语法上与隐式声明完全相同：

```java
  BiFunction<Double, Integer, Double> f = 
```

```java
                                       (var x, var y) -> x / y;
```

```java
  System.out.println(f.apply(3., 2));    //prints: 1.5
```

新的局部变量样式语法允许我们在不显式定义参数类型的情况下添加注解。让我们将以下依赖项添加到`pom.xml`文件中：

```java
<dependency>
```

```java
    <groupId>org.jetbrains</groupId>
```

```java
    <artifactId>annotations</artifactId>
```

```java
    <version>22.0.0</version>
```

```java
</dependency>
```

它允许我们定义传入的变量为非空：

```java
import javax.validation.constraints.NotNull;
```

```java
import java.util.function.BiFunction;
```

```java
import java.util.function.Consumer;
```

```java
BiFunction<Double, Integer, Double> f =
```

```java
                     (@NotNull var x, @NotNull var y) -> x / y;
```

```java
System.out.println(f.apply(3., 2));    //prints: 1.5
```

注解向编译器传达程序员的意图，因此它可以在编译或执行期间警告程序员如果声明的意图被违反。例如，我们尝试运行以下代码：

```java
BiFunction<Double, Integer, Double> f = (x, y) -> x / y;
```

```java
System.out.println(f.apply(null, 2));
```

它在运行时失败，抛出`NullPointerException`。然后，我们按照以下方式添加了注解：

```java
BiFunction<Double, Integer, Double> f =
```

```java
        (@NotNull var x, @NotNull var y) -> x / y;
```

```java
System.out.println(f.apply(null, 2));
```

运行前面代码的结果看起来像这样：

```java
Exception in thread "main" java.lang.IllegalArgumentException: 
```

```java
Argument for @NotNull parameter 'x' of 
```

```java
com/packt/learnjava/ch13_functional/LambdaExpressions
```

```java
.lambda$localVariableSyntax$1 must not be null
```

```java
at com.packt.learnjava.ch13_functional.LambdaExpressions
```

```java
.$$$reportNull$$$0(LambdaExpressions.java)
```

```java
at com.packt.learnjava.ch13_functional.LambdaExpressions
```

```java
.lambda$localVariableSyntax$1(LambdaExpressions.java)
```

```java
at com.packt.learnjava.ch13_functional.LambdaExpressions
```

```java
.localVariableSyntax(LambdaExpressions.java:59)
```

```java
at com.packt.learnjava.ch13_functional.LambdaExpressions
```

```java
.main(LambdaExpressions.java:12)
```

Lambda 表达式甚至没有被执行。

如果参数是具有非常长的类名的对象，我们需要在参数上使用注解时，Lambda 参数的局部变量语法的优势就变得明显了。在 Java 11 之前，代码可能看起来像以下这样：

```java
BiFunction<SomeReallyLongClassName,
```

```java
AnotherReallyLongClassName, Double> f =
```

```java
      (@NotNull SomeReallyLongClassName x,
```

```java
    @NotNull AnotherReallyLongClassName y) -> x.doSomething(y);
```

我们必须显式声明变量的类型，因为我们想添加注解，而以下隐式版本甚至无法编译：

```java
BiFunction<SomeReallyLongClassName,
```

```java
AnotherReallyLongClassName, Double> f =
```

```java
           (@NotNull x, @NotNull y) -> x.doSomething(y);
```

使用 Java 11，新的语法允许我们使用`var`类型持有者进行隐式参数类型推断：

```java
BiFunction<SomeReallyLongClassName,
```

```java
AnotherReallyLongClassName, Double> f =
```

```java
          (@NotNull var x, @NotNull var y) -> x.doSomething(y);
```

这就是引入 Lambda 参数声明的局部变量语法的优势所在。否则，考虑远离使用`var`。如果变量的类型很短，使用其实际类型可以使代码更容易理解。

# 标准函数式接口

`java.util.function`包中提供的接口大多是以下四个接口的特化：`Consumer<T>`、`Predicate<T>`、`Supplier<T>`和`Function<T,R>`。让我们回顾它们，然后简要了解一下其他 39 个标准函数式接口。

## Consumer<T>

通过查看`Consumer<T>`接口定义，`<indexentry content="standard functional interfaces:Consumer">`，你可以猜出这个接口有一个接受类型为`T`的参数且不返回任何内容的抽象方法。嗯，当只列出一个类型时，它可能定义了返回值的类型，例如`Supplier<T>`接口的情况。但接口名称是一个线索：`consumer`名称表明这个接口的方法只是接受值并返回空，而`supplier`返回值。这个线索并不精确，但有助于唤起记忆。

关于任何函数式接口的最佳信息来源是`java.util.function`包的 API 文档([`docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/function/package-summary.html`](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/function/package-summary.html))。如果我们阅读它，我们会了解到`Consumer<T>`接口有一个抽象方法和一个默认方法：

+   `void accept(T t)`: 对给定的参数应用操作

+   `default Consumer<T> andThen(Consumer<T> after)`: 返回一个组合的 `Consumer` 函数，它按顺序执行当前操作后跟 `after` 操作

这意味着，例如，我们可以这样实现并执行：

```java
  Consumer<String> printResult = 
```

```java
                       s -> System.out.println("Result: " + s);
```

```java
  printResult.accept("10.0");   //prints: Result: 10.0
```

我们还可以有一个创建函数的工厂方法，例如：

```java
Consumer<String> printWithPrefixAndPostfix(String pref, String postf){
```

```java
    return s -> System.out.println(pref + s + postf);
```

```java
}
```

现在，我们可以这样使用它：

```java
printWithPrefixAndPostfix("Result: ", 
```

```java
                          " Great!").accept("10.0");            
```

```java
                                  //prints: Result: 10.0 Great!
```

为了演示 `andThen()` 方法，让我们创建 `Person` 类：

```java
public class Person {
```

```java
    private int age;
```

```java
    private String firstName, lastName, record;
```

```java
    public Person(int age, String firstName, String lastName) {
```

```java
        this.age = age;
```

```java
        this.lastName = lastName;
```

```java
        this.firstName = firstName;
```

```java
    }
```

```java
    public int getAge() { return age; }
```

```java
    public String getFirstName() { return firstName; }
```

```java
    public String getLastName() { return lastName; }
```

```java
    public String getRecord() { return record; }
```

```java
    public void setRecord(String fullId) { 
```

```java
                                        this.record = record; }
```

```java
}
```

你可能已经注意到，`record` 是唯一一个可以设置的属性。我们将使用它在消费者函数中设置个人记录：

```java
String externalData = "external data";
```

```java
Consumer<Person> setRecord =
```

```java
    p -> p.setFullId(p.getFirstName() + " " +
```

```java
    p.getLastName() + ", " + p.getAge() + ", " + externalData);
```

`setRecord` 函数接受 `Person` 对象的属性值和一些来自外部源的数据，并将结果值设置为 `record` 属性值。显然，这也可以用其他几种方式完成，但我们这样做是为了演示目的。让我们也创建一个打印 `record` 属性值的函数：

```java
Consumer<Person> printRecord = p -> System.out.println(
```

```java
                                                p.getRecord());
```

这两个函数的组合可以创建并执行如下所示：

```java
Consumer<Person> setRecordThenPrint = setRecord.
```

```java
                                        andThen(printPersonId);
```

```java
setRecordThenPrint.accept(new Person(42, "Nick", "Samoylov")); 
```

```java
                 //prints: Nick Samoylov, age 42, external data
```

这样，就可以创建一个整个处理管道的操作，这些操作会转换通过管道传递的对象的属性。

## Predicate<T>

这个函数式接口，`Predicate<T>`，有一个抽象方法，五个默认方法，和一个静态方法，允许谓词链式连接：

+   `boolean test(T t)`: 评估提供的参数是否满足条件

+   `default Predicate<T> negate()`: 返回当前谓词的否定

+   `static <T> Predicate<T> not(Predicate<T> target)`: 返回提供的谓词的否定

+   `default Predicate<T> or(Predicate<T> other)`: 从这个谓词和提供的谓词构建一个逻辑“或”

+   `default Predicate<T> and(Predicate<T> other)`: 从这个谓词和提供的谓词构建一个逻辑“与”

+   `static <T> Predicate<T> isEqual(Object targetRef)`: 构建一个谓词，用于评估两个参数是否根据 `Objects.equals(Object, Object)` 相等

这个接口的基本用法相当简单：

```java
Predicate<Integer> isLessThan10 = i -> i < 10;
```

```java
System.out.println(isLessThan10.test(7));      //prints: true
```

```java
System.out.println(isLessThan10.test(12));     //prints: false
```

我们还可以将它与之前创建的 `printWithPrefixAndPostfix(String pref, String postf)` 函数结合使用：

```java
int val = 7;
```

```java
Consumer<String> printIsSmallerThan10 = 
```

```java
printWithPrefixAndPostfix("Is " + val + " smaller than 10? ", 
```

```java
                                                  "  Great!");
```

```java
printIsSmallerThan10.accept(String.valueOf(isLessThan10.
```

```java
                                                   test(val))); 
```

```java
                    //prints: Is 7 smaller than 10? true Great!
```

其他方法（也称为**操作**）可用于创建操作链（也称为**管道**），以下是一些示例：

```java
Predicate<Integer> isEqualOrGreaterThan10 = isLessThan10.
```

```java
                                                      negate();
```

```java
System.out.println(isEqualOrGreaterThan10.test(7));  
```

```java
                                                //prints: false
```

```java
System.out.println(isEqualOrGreaterThan10.test(12)); 
```

```java
                                                 //prints: true
```

```java
isEqualOrGreaterThan10 = Predicate.not(isLessThan10);
```

```java
System.out.println(isEqualOrGreaterThan10.test(7));  
```

```java
                                                //prints: false
```

```java
System.out.println(isEqualOrGreaterThan10.test(12)); 
```

```java
                                                 //prints: true
```

```java
Predicate<Integer> isGreaterThan10 = i -> i > 10;
```

```java
Predicate<Integer> is_lessThan10_OR_greaterThan10 = 
```

```java
                              isLessThan10.or(isGreaterThan10);
```

```java
System.out.println(is_lessThan10_OR_greaterThan10.test(20)); 
```

```java
                                                        // true
```

```java
System.out.println(is_lessThan10_OR_greaterThan10.test(10)); 
```

```java
                                                       // false
```

```java
Predicate<Integer> isGreaterThan5 = i -> i > 5;
```

```java
Predicate<Integer> is_lessThan10_AND_greaterThan5 = 
```

```java
                   isLessThan10.and(isGreaterThan5);
```

```java
System.out.println(is_lessThan10_AND_greaterThan5.test(3));  
```

```java
                                                       // false
```

```java
System.out.println(is_lessThan10_AND_greaterThan5.test(7));  
```

```java
                                                        // true
```

```java
Person nick = new Person(42, "Nick", "Samoylov");
```

```java
Predicate<Person> isItNick = Predicate.isEqual(nick);
```

```java
Person john = new Person(42, "John", "Smith");
```

```java
Person person = new Person(42, "Nick", "Samoylov");
```

```java
System.out.println(isItNick.test(john));        
```

```java
                                                //prints: false
```

```java
System.out.println(isItNick.test(person));            
```

```java
                                                 //prints: true
```

`predicate` 对象可以被链入更复杂的逻辑语句，并包含所有必要的外部数据，正如之前所演示的那样。

## Supplier<T>

这个函数式接口，`Supplier<T>`，只有一个抽象方法，`T get()`，它返回一个值。基本用法如下所示：

```java
Supplier<Integer> supply42 = () -> 42;
```

```java
System.out.println(supply42.get());  //prints: 42
```

它可以与前面章节中讨论的函数链式连接：

```java
int input = 7;
```

```java
int limit = 10;
```

```java
Supplier<Integer> supply7 = () -> input;
```

```java
Predicate<Integer> isLessThan10 = i -> i < limit;
```

```java
Consumer<String> printResult = printWithPrefixAndPostfix("Is "
```

```java
         + input + " smaller than " + limit + "? ", " Great!");
```

```java
printResult.accept(String.valueOf(isLessThan10.test(
```

```java
                                              supply7.get())));
```

```java
                    //prints: Is 7 smaller than 10? true Great!
```

`Supplier<T>` 函数通常用作数据处理管道的数据入口点。

## Function<T, R>

这种和其它返回值的函数接口的表示法包括将返回类型作为泛型列表中的最后一个（在这种情况下是 `R`）和在其前面（在这种情况下是一个类型为 `T` 的输入参数）列出输入数据类型。所以，`Function<T, R>` 表示法意味着该接口的唯一抽象方法接受类型为 `T` 的参数并产生类型为 `R` 的结果。让我们看看在线文档（[`docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/function/Function.html`](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/function/Function.html)）。

`Function<T, R>` 接口有一个抽象方法 `R apply(T)` 和两个用于链式操作的方法：

+   `default <V> Function<T,V> andThen(Function<R, V> after)`: 返回一个组合函数，它首先将当前函数应用于其输入，然后将 `after` 函数应用于结果

+   `default <V> Function<V,R> compose(Function<V, T> before)`: 返回一个组合函数，它首先将 `before` 函数应用于其输入，然后将当前函数应用于结果

也有一个 `identity()` 方法：

+   `static <T> Function<T,T> identity()`: 返回一个总是返回其输入参数的函数

让我们回顾所有这些方法以及它们的使用方式。以下是对 `Function<T,R>` 接口基本使用的示例：

```java
Function<Integer, Double> multiplyByTen = i -> i * 10.0;
```

```java
System.out.println(multiplyByTen.apply(1));    //prints: 10.0
```

我们还可以将其与前面章节中讨论的所有函数链式连接：

```java
Supplier<Integer> supply7 = () -> 7;
```

```java
Function<Integer, Double> multiplyByFive = i -> i * 5.0;
```

```java
Consumer<String> printResult = 
```

```java
              printWithPrefixAndPostfix("Result: ", " Great!");
```

```java
printResult.accept(multiplyByFive.
```

```java
     apply(supply7.get()).toString()); 
```

```java
                                  //prints: Result: 35.0 Great!
```

`andThen()` 方法允许从更简单的函数构建复杂函数。注意以下代码中的 `divideByTwo.andThen()` 行：

```java
Function<Double, Long> divideByTwo = 
```

```java
                       d -> Double.valueOf(d / 2.).longValue();
```

```java
Function<Long, String> incrementAndCreateString = 
```

```java
                                    l -> String.valueOf(l + 1);
```

```java
Function<Double, String> divideByTwoIncrementAndCreateString = 
```

```java
                 divideByTwo.andThen(incrementAndCreateString);
```

```java
printResult.accept(divideByTwoIncrementAndCreateString.
```

```java
                                                    apply(4.));
```

```java
                                     //prints: Result: 3 Great!
```

它描述了对输入值应用的运算序列。注意 `divideByTwo()` 函数的返回类型（`Long`）与 `incrementAndCreateString()` 函数的输入类型相匹配。

`compose()` 方法以相反的顺序完成相同的结果：

```java
Function<Double, String> divideByTwoIncrementAndCreateString =  
```

```java
                 incrementAndCreateString.compose(divideByTwo);
```

```java
printResult.accept(divideByTwoIncrementAndCreateString.
```

```java
                                                    apply(4.)); 
```

```java
                                     //prints: Result: 3 Great!
```

现在，复杂函数的组合序列与执行序列不匹配。在 `divideByTwo()` 函数尚未创建且你希望内联创建它的情况下，这可能会非常方便。然后，以下构造将无法编译：

```java
Function<Double, String> divideByTwoIncrementAndCreateString =
```

```java
       (d -> Double.valueOf(d / 2.).longValue())
```

```java
                   .andThen(incrementAndCreateString); 
```

以下行将编译无误：

```java
Function<Double, String> divideByTwoIncrementAndCreateString =
```

```java
      incrementAndCreateString
```

```java
      .compose(d -> Double.valueOf(d / 2.).longValue());
```

它在构建函数管道时提供了更多的灵活性，因此你可以在创建下一个操作时以流畅的风格构建它，而不会打断连续行。

当你需要传入一个与所需函数签名匹配但不起作用的函数时，`identity()` 方法很有用。但是，它只能替换返回类型与输入类型相同的函数，如下例所示：

```java
Function<Double, Double> multiplyByTwo = d -> d * 2.0; 
```

```java
System.out.println(multiplyByTwo.apply(2.));  //prints: 4.0
```

```java
multiplyByTwo = Function.identity();
```

```java
System.out.println(multiplyByTwo.apply(2.));  //prints: 2.0
```

为了展示其可用性，让我们假设我们有一个以下处理管道：

```java
Function<Double, Double> multiplyByTwo = d -> d * 2.0;
```

```java
System.out.println(multiplyByTwo.apply(2.));  //prints: 4.0
```

```java
Function<Double, Long> subtract7 = d -> Math.round(d - 7);
```

```java
System.out.println(subtract7.apply(11.0));   //prints: 4
```

```java
long r = multiplyByTwo.andThen(subtract7).apply(2.);
```

```java
System.out.println(r);                       //prints: -3
```

然后，我们决定在某种情况下，`multiplyByTwo()` 函数应该不执行任何操作。我们可以在其中添加一个条件关闭来打开或关闭它。但是，如果我们想保持函数完整或如果这个函数是从第三方代码传递给我们的，我们只需做以下操作：

```java
Function<Double, Double> multiplyByTwo = d -> d * 2.0;
```

```java
System.out.println(multiplyByTwo.apply(2.));  //prints: 4.0
```

```java
Function<Double, Long> subtract7 = d -> Math.round(d - 7);
```

```java
System.out.println(subtract7.apply(11.0));   //prints: 4
```

```java
long r = multiplyByTwo.andThen(subtract7).apply(2.);
```

```java
System.out.println(r);                       //prints: -3 
```

```java
multiplyByTwo = Function.identity();
```

```java
System.out.println(multiplyByTwo.apply(2.)); //prints: 2.0;
```

```java
r = multiplyByTwo.andThen(subtract7).apply(2.);
```

```java
System.out.println(r);                      //prints: -5
```

如您所见，`multiplyByTwo()` 函数现在不执行任何操作，最终结果也不同。

## 其他标准函数式接口

`java.util.function` 包中的其他 39 个函数式接口是我们刚刚审查的四个接口的变体。这些变体是为了实现以下一个或多个目标而创建的：

+   通过显式使用 `int`、`double` 或 `long` 原始类型来避免自动装箱和拆箱，从而提高性能

+   允许两个输入参数和/或简短的表达方式

这里只是几个例子：

+   `IntFunction<R>` 具有带有 `R apply(int)` 方法的 `R`，提供了一种更简短的表达方式（对于输入参数类型没有泛型），并通过要求原始 `int` 作为参数来避免自动装箱。

+   `BiFunction<T, U, R>` 具有带有 `R apply(T, U)` 方法的 `R` 类型的参数，允许两个输入参数；`BinaryOperator<T>` 具有带有 `T apply(T, T)` 方法的 `T` 类型的参数，允许两个 `T` 类型的输入参数，并返回相同类型的值，`T`。

+   `IntBinaryOperator` 具有带有 `int applAsInt(int, int)` 方法的 `int` 类型的参数，并且也返回 `int` 类型的值。

如果您打算使用函数式接口，我们鼓励您研究 `java.util.functional` 包的接口 API ([`docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/function/package-summary.html`](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/function/package-summary.html)).

# Lambda 表达式限制

有两个关于 Lambda 表达式的方面我们想要指出并澄清：

+   如果一个 Lambda 表达式使用了其外部创建的局部变量，那么这个局部变量必须是 final 或实际上是 final（在同一上下文中不会被重新赋值）。

+   Lambda 表达式中的 `this` 关键字指向封装上下文，而不是 Lambda 表达式本身。

与匿名类一样，在 Lambda 表达式外部创建并在其中使用的变量成为实际上是 final 的，并且不能被修改。以下是一个尝试更改初始化变量值的错误示例：

```java
int x = 7;
```

```java
//x = 3; //compilation error
```

```java
Function<Integer, Integer> multiply = i -> i * x;
```

这种限制的原因是函数可以在不同的上下文中传递和执行（例如不同的线程），而尝试同步这些上下文将违背无状态函数的原始想法和表达式的评估，评估仅依赖于输入参数，而不是上下文变量。这就是为什么 Lambda 表达式中使用的所有局部变量都必须是实际上是 final 的，这意味着它们要么可以显式地声明为 final，要么通过不改变值而成为 final。

尽管存在这种限制，但有一个可能的解决方案。如果局部变量是引用类型（但不是 `String` 或原始包装类型），则可以更改其状态，即使这个局部变量在 Lambda 表达式中使用：

```java
List<Integer> list = new ArrayList();
```

```java
list.add(7);
```

```java
int x = list.get(0);
```

```java
System.out.println(x);  // prints: 7
```

```java
list.set(0, 3);
```

```java
x = list.get(0);
```

```java
System.out.println(x);  // prints: 3
```

```java
Function<Integer, Integer> multiply = i -> i * list.get(0);
```

由于这种 Lambda 在不同上下文中执行可能存在意外副作用的风险，因此应谨慎使用此解决方案。

`anonymous` 类内部的 `this` 关键字指向 `anonymous` 类的实例。相比之下，在 Lambda 表达式中，`this` 关键字指向表达式周围的类实例，也称为 **封装实例**、**封装上下文** 或 **封装作用域**。

让我们创建一个 `ThisDemo` 类来展示这种差异：

```java
class ThisDemo {
```

```java
    private String field = "ThisDemo.field";
```

```java
    public void useAnonymousClass() {
```

```java
        Consumer<String> consumer = new Consumer<>() {
```

```java
            private String field = "Consumer.field";
```

```java
            public void accept(String s) {
```

```java
                System.out.println(this.field);
```

```java
            }
```

```java
        };
```

```java
        consumer.accept(this.field);
```

```java
    }
```

```java
    public void useLambdaExpression() {
```

```java
        Consumer<String> consumer = consumer = s -> {
```

```java
            System.out.println(this.field);
```

```java
        };
```

```java
        consumer.accept(this.field);
```

```java
    }
```

```java
}
```

如果执行前面的方法，输出将如下所示（代码注释）：

```java
ThisDemo d = new ThisDemo();
```

```java
d.useAnonymousClass();      //prints: Consumer.field
```

```java
d.useLambdaExpression();    //prints: ThisDemo.field
```

如您所见，`anonymous` 类内部的 `this` 关键字指向 `anonymous` 类实例，而 Lambda 表达式中的 `this` 指向封装类实例。Lambda 表达式根本不（也不能）有字段。Lambda 表达式不是一个类实例，不能通过 `this` 来引用。根据 Java 的规范，这种做法通过将 `this` 视为周围上下文来提供更多的灵活性。

# 方法引用

到目前为止，我们所有的函数都是简短的单一行。这里有一个例子：

```java
Supplier<Integer> input = () -> 3;
```

```java
Predicate<Integer> checkValue = d -> d < 5;
```

```java
Function<Integer, Double> calculate = i -> i * 5.0;
```

```java
Consumer<Double> printResult = d -> System.out.println(
```

```java
                                               "Result: " + d);
```

```java
if(checkValue.test(input.get())){
```

```java
    printResult.accept(calculate.apply(input.get()));
```

```java
} else {
```

```java
    System.out.println("Input " + input.get() + 
```

```java
                                             " is too small.");
```

```java
} 
```

如果函数由两行或多行组成，我们可以这样实现：

```java
Supplier<Integer> input = () -> {
```

```java
     // as many line of code here as necessary
```

```java
     return 3;
```

```java
};
```

```java
Predicate<Integer> checkValue = d -> {
```

```java
    // as many line of code here as necessary
```

```java
    return d < 5;
```

```java
};
```

```java
Function<Integer, Double> calculate = i -> {
```

```java
    // as many lines of code here as necessary
```

```java
    return i * 5.0;
```

```java
};
```

```java
Consumer<Double> printResult = d -> {
```

```java
    // as many lines of code here as necessary
```

```java
    System.out.println("Result: " + d);
```

```java
};
```

```java
if(checkValue.test(input.get())){
```

```java
    printResult.accept(calculate.apply(input.get()));
```

```java
} else {
```

```java
    System.out.println("Input " + input.get() + 
```

```java
                                             " is too small.");
```

```java
}
```

当函数实现的规模超过几行代码时，这种代码布局可能不易阅读。它可能会掩盖整体代码结构。为了避免这个问题，可以将函数实现移动到方法中，然后在 Lambda 表达式中引用这个方法。例如，让我们向使用 Lambda 表达式的类中添加一个静态方法和一个实例方法：

```java
private int generateInput(){
```

```java
    // Maybe many lines of code here
```

```java
    return 3;
```

```java
}
```

```java
private static boolean checkValue(double d){
```

```java
    // Maybe many lines of code here
```

```java
    return d < 5;
```

```java
}
```

此外，为了展示各种可能性，让我们创建另一个类，包含一个静态方法和一个实例方法：

```java
class Helper {
```

```java
    public double calculate(int i){
```

```java
        // Maybe many lines of code here
```

```java
        return i* 5; 
```

```java
    }
```

```java
    public static void printResult(double d){
```

```java
        // Maybe many lines of code here
```

```java
        System.out.println("Result: " + d);
```

```java
    }
```

```java
}
```

现在，我们可以将最后一个示例重写如下：

```java
Supplier<Integer> input = () -> generateInput();
```

```java
Predicate<Integer> checkValue = d -> checkValue(d);
```

```java
Function<Integer, Double> calculate = i -> new Helper().calculate(i);
```

```java
Consumer<Double> printResult = d -> Helper.printResult(d);
```

```java
if(checkValue.test(input.get())){
```

```java
    printResult.accept(calculate.apply(input.get()));
```

```java
} else {
```

```java
    System.out.println("Input " + input.get() + 
```

```java
                                             " is too small.");
```

```java
}
```

如您所见，即使每个函数由许多行代码组成，这种结构仍然使代码易于阅读。然而，当一行 Lambda 表达式包含对现有方法的引用时，可以通过不列出参数来进一步简化表示法，使用方法引用。

方法引用的语法是 `Location::methodName`，其中 `Location` 表示 `methodName` 方法属于哪个对象或类，而两个冒号 (`::`) 作为位置和方法名之间的分隔符。使用方法引用表示法，前面的示例可以重写如下：

```java
Supplier<Integer> input = this::generateInput;
```

```java
Predicate<Integer> checkValue = MethodReferenceDemo::checkValue;
```

```java
Function<Integer, Double> calculate = new Helper()::calculate;
```

```java
Consumer<Double> printResult = Helper::printResult;
```

```java
if(checkValue.test(input.get())){
```

```java
    printResult.accept(calculate.apply(input.get()));
```

```java
} else {
```

```java
    System.out.println("Input " + input.get() + 
```

```java
                                             " is too small.");
```

```java
}
```

你可能已经注意到，我们有意使用了不同的位置，两个实例方法，和两个静态方法，以展示各种可能性。如果感觉太多难以记住，好消息是现代 IDE（IntelliJ IDEA 就是一个例子）可以为你完成这项工作，并将你正在编写的代码转换为最紧凑的形式。你只需接受 IDE 的建议即可。

# 摘要

本章通过解释和演示函数式接口和 Lambda 表达式的概念，向你介绍了函数式编程。JDK 附带的标准函数式接口概述可以帮助你避免编写自定义代码，而方法引用符号允许你编写结构良好、易于理解和维护的代码。

现在，你能够编写函数并将它们用于 Lambda 表达式，以便将它们作为方法参数传递。

在下一章中，我们将讨论数据流处理。我们将定义什么是数据流，并探讨如何处理它们的数据以及如何在管道中链式操作流操作。具体来说，我们将讨论流的初始化和操作（方法），如何以流畅的方式连接它们，以及如何创建并行流。

# 问答

1.  什么是函数式接口？选择所有适用的：

    1.  函数集合

    1.  只有一个方法的接口

    1.  任何只有一个抽象方法的接口

    1.  任何用 Java 编写的库

1.  什么是 Lambda 表达式？选择所有适用的：

    1.  一个作为匿名方法实现的函数，没有修饰符、返回类型和参数类型

    1.  函数式接口的实现

    1.  任何以 Lambda 计算风格实现的实现

    1.  一种包含参数列表、箭头符号（->）和由单个语句或语句块组成的主体符号的表示法

1.  `Consumer<T>`接口的实现有多少个输入参数？

1.  `Consumer<T>`接口的实现中返回值的类型是什么？

1.  `Predicate<T>`接口的实现有多少个输入参数？

1.  `Predicate<T>`接口的实现中返回值的类型是什么？

1.  `Supplier<T>`接口的实现有多少个输入参数？

1.  `Supplier<T>`接口的实现中返回值的类型是什么？

1.  `Function<T,R>`接口的实现有多少个输入参数？

1.  `Function<T,R>`接口的实现中返回值的类型是什么？

1.  在 Lambda 表达式中，`this`关键字指的是什么？

1.  方法引用语法是什么？
