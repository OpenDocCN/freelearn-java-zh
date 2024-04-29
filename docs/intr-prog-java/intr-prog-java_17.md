# 第十七章：Lambda 表达式和函数式编程

本章解释了函数式编程的概念。它提供了 JDK 附带的功能接口的概述，解释了如何在 Lambda 表达式中使用它们，以及如何以最简洁的方式编写 Lambda 表达式。

在本章中，我们将介绍以下主题：

+   函数式编程

+   函数式接口

+   Lambda 表达式

+   方法引用

+   练习——使用方法引用创建一个新对象

# 函数式编程

函数式编程允许我们像处理对象一样处理一段代码（一个函数），将其作为参数传递或作为方法的返回值。这个特性存在于许多编程语言中。它不需要我们管理对象状态。这个函数是无状态的。它的结果只取决于输入数据，不管它被调用多少次。这种风格使结果更可预测，这是函数式编程最具吸引力的方面。

没有函数式编程，Java 中将功能作为参数传递的唯一方式是通过编写一个实现接口的类，创建其对象，然后将其作为参数传递。但即使是最简单的样式——使用匿名类——也需要编写太多的样板代码。使用函数式接口和 Lambda 表达式使代码更短、更清晰、更具表现力。

将其添加到 Java 中增加了并行编程的能力，将并行性的责任从客户端代码转移到库中。在此之前，为了处理 Java 集合的元素，客户端代码必须遍历集合并组织处理。在 Java 8 中，添加了新的（默认）方法，接受一个函数（函数式接口的实现）作为参数，然后根据内部处理算法并行或顺序地将其应用于集合的每个元素。因此，组织并行处理是库的责任。

在本章中，我们将定义和解释这些 Java 特性——函数式接口和 Lambda 表达式，并演示它们在代码示例中的适用性。它们将函数作为语言中与对象同等重要的一等公民。

# 什么是函数式接口？

实际上，您在我们的演示代码中已经看到了函数式编程的元素。一个例子是`forEach(Consumer consumer)`方法，适用于每个`Iterable`，其中`Consumer`是一个函数式接口。另一个例子是`removeIf(Predicate predicate)`方法，适用于每个`Collection`对象。传入的`Predicate`对象是一个函数——函数式接口的实现。类似地，`List`接口中的`sort(Comparator comparator)`和`replaceAll(UnaryOperator uo)`方法以及`Map`中的几个`compute()`方法都是函数式编程的例子。

一个函数接口是一个只有一个抽象方法的接口，包括那些从父接口继承的方法。

为了帮助避免运行时错误，在 Java 8 中引入了`@FunctionalInterface`注解，告诉编译器关于意图，因此编译器可以检查被注解接口中是否真正只有一个抽象方法。让我们一起审查下面的与同一继承线的接口：

```java
@FunctionalInterface
interface A {
  void method1();
  default void method2(){}
  static void method3(){}
}

@FunctionalInterface
interface B extends A {
  default void method4(){}
}

@FunctionalInterface
interface C extends B {
  void method1();
}

//@FunctionalInterface  //compilation error
interface D extends C {
  void method5();
}
```

接口`A`是一个函数接口，因为它只有一个抽象方法：`method1()`。接口`B`也是一个函数接口，因为它也只有一个抽象方法 - 从接口`A`继承的同一个`method1()`。接口`C`是一个函数接口，因为它只有一个抽象方法，`method1()`，它覆盖了父接口`A`的抽象`method1()`方法。接口`D`不能是一个函数接口，因为它有两个抽象方法 - 从父接口`A`继承的`method1()`和`method5()`。

当使用`@FunctionalInterface`注解时，它告诉编译器只检查存在一个抽象方法，并警告程序员读取代码时，这个接口只有一个抽象方法是有意的。否则，程序员可能会浪费时间完善接口，最后发现无法完成。

出于同样的原因，自 Java 早期版本以来存在的`Runnable`和`Callable`接口在 Java 8 中被注释为`@FunctionalInterface`。这明确表明了这种区别，并提醒其用户以及可能尝试添加另一个抽象方法的人：

```java
@FunctionalInterface
interface Runnable { 
  void run(); 
} 
@FunctionalInterface
interface Callable<V> { 
  V call() throws Exception; 
}
```

可以看到，创建一个函数接口很容易。但在这之前，考虑使用`java.util.function`包中提供的 43 个函数接口之一。

# 准备好使用的标准函数接口

`java.util.function`包中提供的大多数接口都是以下四个接口的专业化：`Function`，`Consumer`，`Supplier`和`Predicate`。让我们对它们进行审查，然后简要概述其余 39 个标准函数接口。

# Function<T, R>

这个和其他函数接口的标记包括输入数据类型(`T`)和返回数据类型(`R`)的列举。因此，`Function<T, R>`表示该接口的唯一抽象方法接受类型为`T`的参数并产生类型为`R`的结果。您可以通过阅读在线文档找到该抽象方法的名称。在`Function<T, R>`接口的情况下，它的方法是`R apply(T)`。

在学习所有内容后，我们可以使用匿名类创建该接口的实现：

```java
Function<Integer, Double> multiplyByTen = new Function<Integer, Double>(){
  public Double apply(Integer i){
    return i * 10.0;
  }
};
```

由程序员决定`T`（输入参数）将是哪种实际类型，以及`R`（返回值）将是哪种类型。在我们的示例中，我们已经决定输入参数将是`Integer`类型，结果将是`Double`类型。正如你现在可能已经意识到的那样，类型只能是引用类型，并且原始类型的装箱和拆箱会自动执行。

现在我们可以按照需要使用我们新的`Function<Integer, Double> multiplyByTen`函数。我们可以直接使用它，如下所示：

```java
System.out.println(multiplyByTen.apply(1)); //prints: 10.0
```

或者我们可以创建一个接受这个函数作为参数的方法：

```java
void useFunc(Function<Integer, Double> processingFunc, int input){
  System.out.println(processingFunc.apply(input));
}
```

然后我们可以将我们的函数传递给这个方法，并让方法使用它：

```java
useFunc(multiplyByTen, 10);     //prints: 100.00

```

我们还可以创建一个方法，每当我们需要一个函数时就会生成一个函数：

```java
Function<Integer, Double> createMultiplyBy(double num){
  Function<Integer, Double> func = new Function<Integer, Double>(){
    public Double apply(Integer i){
      return i * num;
    }
  };
  return func;
}
```

使用前述方法，我们可以编写以下代码：

```java
Function<Integer, Double> multiplyByFive = createMultiplyBy(5);
System.out.println(multiplyByFive.apply(1)); //prints: 5.0
useFunc(multiplyByFive, 10);                 //prints: 50.0

```

在下一节中，我们将介绍 lambda 表达式，并展示如何使用它们以更少的代码来表示函数接口实现。

# `Consumer<T>`

通过查看`Consumer<T>`接口的定义，你可以猜到这个接口有一个接受`T`类型参数的抽象方法，而且不返回任何东西。从`Consumer<T>`接口的文档中，我们了解到它的抽象方法是`void accept(T)`，这意味着，例如，我们可以这样实现它：

```java
Consumer<Double> printResult = new Consumer<Double>() {
  public void accept(Double d) {
    System.out.println("Result=" + d);
  }
};
printResult.accept(10.0);         //prints: Result=10.0

```

或者我们可以创建一个生成函数的方法：

```java
Consumer<Double> createPrintingFunc(String prefix, String postfix){
  Consumer<Double> func = new Consumer<Double>() {
    public void accept(Double d) {
      System.out.println(prefix + d + postfix);
    }
  };
  return func;
}
```

现在我们可以像下面这样使用它：

```java
Consumer<Double> printResult = createPrintingFunc("Result=", " Great!");
printResult.accept(10.0);    //prints: Result=10.0 Great!

```

我们还可以创建一个新方法，不仅接受一个处理函数作为参数，还接受一个打印函数：

```java
void processAndConsume(int input, 
                       Function<Integer, Double> processingFunc, 
                                          Consumer<Double> consumer){
  consumer.accept(processingFunc.apply(input));
}
```

然后我们可以编写以下代码：

```java
Function<Integer, Double> multiplyByFive = createMultiplyBy(5);
Consumer<Double> printResult = createPrintingFunc("Result=", " Great!");
processAndConsume(10, multiplyByFive, printResult); //Result=50.0 Great! 
```

正如我们之前提到的，在下一节中，我们将介绍 lambda 表达式，并展示如何使用它们以更少的代码来表示函数接口实现。

# `Supplier<T>`

这是一个诡计问题：猜猜`Supplier<T>`接口的抽象方法的输入和输出类型。答案是：它不接受参数，返回`T`类型。正如你现在理解的那样，区别在于接口本身的名称。它应该给你一个提示：消费者只消耗而不返回任何东西，而供应者只提供而不需要任何输入。`Supplier<T>`接口的抽象方法是`T get()`。

与前面的函数类似，我们可以编写生成供应者的方法：

```java
Supplier<Integer> createSuppplier(int num){
  Supplier<Integer> func = new Supplier<Integer>() {
    public Integer get() { return num; }
  };
  return func;
}
```

现在我们可以编写一个只接受函数的方法：

```java
void supplyProcessAndConsume(Supplier<Integer> input, 
                             Function<Integer, Double> process, 
                                      Consumer<Double> consume){
  consume.accept(processFunc.apply(input.get()));
}
```

注意`input`函数的输出类型与`process`函数的输入类型相同，返回类型与`consume`函数消耗的类型相同。这使得以下代码成为可能：

```java
Supplier<Integer> supply7 = createSuppplier(7);
Function<Integer, Double> multiplyByFive = createMultiplyBy(5);
Consumer<Double> printResult = createPrintingFunc("Result=", " Great!");
supplyProcessAndConsume(supply7, multiplyByFive, printResult); 
                                            //prints: Result=35.0 Great!

```

到此为止，我们希望你开始欣赏函数式编程带来的价值。它允许我们传递功能块，可以插入到算法的中间而不需要创建对象。静态方法也不需要创建对象，但它们由于在 JVM 中是唯一的，所以会被所有应用线程共享。与此同时，每个函数都是一个对象，可以在 JVM 中是唯一的（如果赋值给静态变量），或者为每个处理线程创建一个（这通常是情况）。它几乎没有编码开销，并且在 lambda 表达式中使用时可以更少地使用管道 - 这是我们下一节的主题。

到目前为止，我们已经演示了如何将函数插入现有的控制流表达式中。现在我们将描述最后一个缺失的部分 - 一个表示决策构造的函数，也可以作为对象传递。

# Predicate<T>

这是一个表示具有单个方法`boolean test(T)`的布尔值函数的接口。这里是一个创建`Predicate<Integer>`函数的方法示例：

```java
Predicate<Integer> createTestSmallerThan(int num){
  Predicate<Integer> func = new Predicate<Integer>() {
    public boolean test(Integer d) {
      return d < num;
    }
  };
  return func;
}
```

我们可以使用它来为处理方法添加一些逻辑：

```java
void supplyDecideProcessAndConsume(Supplier<Integer> input, 
                                  Predicate<Integer> test, 
                                   Function<Integer, Double> process, 
                                            Consumer<Double> consume){
  int in = input.get();
  if(test.test(in)){
    consume.accept(process.apply(in));
  } else {
    System.out.println("Input " + in + 
                     " does not pass the test and not processed.");
  }
}
```

下面的代码演示了它的使用方法：

```java
Supplier<Integer> input = createSuppplier(7);
Predicate<Integer> test = createTestSmallerThan(5);
Function<Integer, Double> multiplyByFive = createMultiplyBy(5);
Consumer<Double> printResult = createPrintingFunc("Result=", " Great!");
supplyDecideProcessAndConsume(input, test, multiplyByFive, printResult);
             //prints: Input 7 does not pass the test and not processed.
```

例如，让我们将输入设置为 3：

```java
Supplier<Integer> input = createSuppplier(3)
```

前面的代码将导致以下输出：

```java
Result=15.0 Great!
```

# 其他标准的函数式接口

`java.util.function`包中的其他 39 个函数接口是我们刚刚审查的四个接口的变体。这些变体是为了实现以下目的之一或任意组合：

+   通过明确使用整数、双精度或长整型原始类型来避免自动装箱和拆箱，从而获得更好的性能

+   允许两个输入参数

+   更简短的记法

这里只是一些例子：

+   `IntFunction<R>`提供了更简短的记法（不需要输入参数类型的泛型）并且避免了自动装箱，因为它要求参数为`int`原始类型。

+   `BiFunction<T,U,R>`的`R apply(T,U)`方法允许两个输入参数

+   `BinaryOperator<T>`的`T apply(T,T)`方法允许两个`T`类型的输入参数，并返回相同的`T`类型的值

+   `IntBinaryOperator`的`int applAsInt(int,int)`方法接受两个`int`类型的参数，并返回`int`类型的值

如果你打算使用函数接口，我们鼓励你学习`java.util.functional`包中接口的 API。

# 链接标准函数

`java.util.function`包中的大多数函数接口都有默认方法，允许我们构建一个函数链（也称为管道），将一个函数的结果作为另一个函数的输入参数传递，从而组合成一个新的复杂函数。例如：

```java
Function<Double, Long> f1 = d -> Double.valueOf(d / 2.).longValue();
Function<Long, String> f2 = l -> "Result: " + (l + 1);
Function<Double, String> f3 = f1.andThen(f2);
System.out.println(f3.apply(4.));            //prints: 3

```

如您从前面的代码中所见，我们通过使用 `andThen()` 方法将 `f1` 和 `f2` 函数组合成了一个新的 `f3` 函数。这就是我们将要在本节中探讨的方法的思想。首先，我们将函数表示为匿名类，然后在以下部分中，我们介绍了前面示例中使用的 lambda 表达式。

# 链两个 Function<T,R>

我们可以使用 `Function` 接口的 `andThen(Function after)` 默认方法。我们已经创建了 `Function<Integer, Double> createMultiplyBy()` 方法：

```java
Function<Integer, Double> createMultiplyBy(double num){
  Function<Integer, Double> func = new Function<Integer, Double>(){
    public Double apply(Integer i){
      return i * num;
    }
  };
  return func; 
```

我们还可以编写另一个方法，该方法创建具有 `Double` 输入类型的减法函数，以便我们可以将其链接到乘法函数：

```java
private static Function<Double, Long> createSubtractInt(int num){
  Function<Double, Long> func = new Function<Double, Long>(){
    public Long apply(Double dbl){
      return Math.round(dbl - num);
    }
  };
  return func;
}

```

现在我们可以编写以下代码：

```java
Function<Integer, Double> multiplyByFive = createMultiplyBy(5);
System.out.println(multiplyByFive.apply(2));  //prints: 10.0

Function<Double, Long> subtract7 = createSubtractInt(7);
System.out.println(subtract7.apply(11.0));   //prints: 4

long r = multiplyByFive.andThen(subtract7).apply(2);
System.out.println(r);                          //prints: 3

```

如您所见，`multiplyByFive.andThen(subtract7)` 链有效地作为 `Function<Integer, Long> multiplyByFiveAndSubtractSeven`。

`Function` 接口还有另一个默认方法 `Function<V,R> compose(Function<V,T> before)`，它也允许我们链两个函数。必须先执行的函数可以作为 `before` 参数传递到第二个函数的 `compose()` 方法中：

```java
boolean r = subtract7.compose(multiplyByFive).apply(2);
System.out.println(r);                          //prints: 3         

```

# 链两个 Consumer<T>

`Consumer` 接口也有 `andThen(Consumer after)` 方法。我们已经编写了创建打印函数的方法：

```java
Consumer<Double> createPrintingFunc(String prefix, String postfix){
  Consumer<Double> func = new Consumer<Double>() {
    public void accept(Double d) {
      System.out.println(prefix + d + postfix);
    }
  };
  return func;
}
```

现在我们可以创建和链两个打印函数，如下所示：

```java
Consumer<Double> print21By = createPrintingFunc("21 by ", "");
Consumer<Double> equalsBy21 = createPrintingFunc("equals ", " by 21");
print21By.andThen(equalsBy21).accept(2d);  
//prints: 21 by 2.0 
//        equals 2.0 by 21

```

如您在 `Consumer` 链中所见，两个函数按链定义的顺序消耗相同的值。

# 链两个 Predicate<T>

`Supplier` 接口没有默认方法，而 `Predicate` 接口有一个静态方法 `isEqual(Object targetRef)` 和三个默认方法：`and(Predicate other)`、`negate()` 和 `or(Predicate other)`。为了演示 `and(Predicate other)` 和 `or(Predicate other)` 方法的用法，例如，让我们编写创建两个 `Predicate<Double>` 函数的方法。一个函数检查值是否小于输入：

```java
Predicate<Double> testSmallerThan(double limit){
  Predicate<Double> func = new Predicate<Double>() {
    public boolean test(Double num) {
      System.out.println("Test if " + num + " is smaller than " + limit);
      return num < limit;
    }
  };
  return func;
}
```

另一个函数检查值是否大于输入：

```java
Predicate<Double> testBiggerThan(double limit){
  Predicate<Double> func = new Predicate<Double>() {
    public boolean test(Double num) {
      System.out.println("Test if " + num + " is bigger than " + limit);
      return num > limit;
    }
  };
  return func;
}
```

现在我们可以创建两个 `Predicate<Double>` 函数并将它们链在一起：

```java
Predicate<Double> isSmallerThan20 = testSmallerThan(20d);
System.out.println(isSmallerThan20.test(10d));
     //prints: Test if 10.0 is smaller than 20.0
     //        true

Predicate<Double> isBiggerThan18 = testBiggerThan(18d);
System.out.println(isBiggerThan18.test(10d));
    //prints: Test if 10.0 is bigger than 18.0
    //        false

boolean b = isSmallerThan20.and(isBiggerThan18).test(10.);
System.out.println(b);
    //prints: Test if 10.0 is smaller than 20.0
    //        Test if 10.0 is bigger than 18.0
    //        false

b = isSmallerThan20.or(isBiggerThan18).test(10.);
System.out.println(b);
    //prints: Test if 10.0 is smaller than 20.0
    //        true

```

如您所见，`and()` 方法需要执行每个函数，而 `or()` 方法在链中的第一个函数返回 `true` 后就不执行第二个函数。

# identity() 和其他默认方法

`java.util.function` 包的功能接口有其他有用的默认方法。其中一个显著的是 `identity()` 方法，它返回一个始终返回其输入参数的函数：

```java
Function<Integer, Integer> id = Function.identity();
System.out.println(id.apply(4));          //prints: 4

```

`identity()`方法在某些过程需要提供特定函数，但你不希望提供的函数改变任何东西时非常有用。在这种情况下，你可以创建一个具有必要输出类型的身份函数。例如，在我们之前的代码片段中，我们可能决定`multiplyByFive`函数在`multiplyByFive.andThen(subtract7)`链中不改变任何东西：

```java
Function<Double, Double> multiplyByFive = Function.identity();
System.out.println(multiplyByFive.apply(2.));  //prints: 2.0

Function<Double, Long> subtract7 = createSubtractInt(7);
System.out.println(subtract7.apply(11.0));    //prints: 4

long r = multiplyByFive.andThen(subtract7).apply(2.);
System.out.println(r);                       //prints: -5

```

正如你所看到的，`multiplyByFive`函数未对输入参数`2`做任何操作，因此结果（减去`7`后）是`-5`。

其他默认方法大多涉及转换和装箱和拆箱，但也提取两个参数的最小值和最大值。如果你感兴趣，可以查看`java.util.function`包接口的 API，并了解可能性。

# Lambda 表达式

前一节中的例子（使用匿名类实现函数接口）看起来庞大，并且显得冗长。首先，无需重复接口名称，因为我们已经将其声明为对象引用的类型。其次，在只有一个抽象方法的功能接口的情况下，不需要指定需要实现的方法名称。编译器和 Java 运行时可以自行处理。我们所需做的就是提供新的功能。Lambda 表达式就是为了这个目的而引入的。

# 什么是 Lambda 表达式？

术语 lambda 来自于 lambda 演算——一种通用的计算模型，可用于模拟任何图灵机。它是数学家阿隆佐·丘奇在 20 世纪 30 年代引入的。Lambda 表达式是一个函数，在 Java 中实现为匿名方法，还允许我们省略修饰符、返回类型和参数类型。这使得它具有非常简洁的表示。

Lambda 表达式的语法包括参数列表、箭头符号`->`和主体部分。参数列表可以是空的`()`，没有括号（如果只有一个参数），或者用括号括起来的逗号分隔的参数列表。主体部分可以是单个表达式或语句块。

让我们看几个例子：

+   `() -> 42;` 总是返回`42`

+   `x -> x + 1;` 将变量`x`增加`1`

+   `(x, y) -> x * y;` 将`x`乘以`y`并返回结果

+   `(char x) -> x == '$';` 比较变量`x`和符号`$`的值，并返回布尔值

+   `x -> {  System.out.println("x=" + x); };` 打印带有`x=`前缀的`x`值

# 重新实现函数

我们可以使用 lambda 表达式重新编写前一节中创建的函数，如下所示：

```java
Function<Integer, Double> createMultiplyBy(double num){
  Function<Integer, Double> func = i -> i * num;
  return func;
}
Consumer<Double> createPrintingFunc(String prefix, String postfix){
  Consumer<Double> func = d -> System.out.println(prefix + d + postfix);
  return func;
}
Supplier<Integer> createSuppplier(int num){
  Supplier<Integer> func = () -> num;
  return func;
}
Predicate<Integer> createTestSmallerThan(int num){
  Predicate<Integer> func = d -> d < num;
  return func;
}
```

我们不重复实现接口的名称，因为它在方法签名中指定为返回类型。我们也不指定抽象方法的名称，因为它是唯一必须实现的接口方法。编写这样简洁高效的代码变得可能是因为 lambda 表达式和函数接口的组合。

通过前面的例子，你可能意识到不再需要创建函数的方法了。让我们修改调用`supplyDecideProcessAndConsume()`方法的代码：

```java
void supplyDecideProcessAndConsume(Supplier<Integer> input, 
                                   Predicate<Integer> test, 
                                   Function<Integer, Double> process, 
                                            Consumer<Double> consume){
  int in = input.get();
  if(test.test(in)){
    consume.accept(process.apply(in));
  } else {
    System.out.println("Input " + in + 
                 " does not pass the test and not processed.");
  }
}
```

让我们重新审视以下内容：

```java
Supplier<Integer> input = createSuppplier(7);
Predicate<Integer> test = createTestSmallerThan(5);
Function<Integer, Double> multiplyByFive = createMultiplyBy(5);
Consumer<Double> printResult = createPrintingFunc("Result=", " Great!");
supplyDecideProcessAndConsume(input, test, multiplyByFive, printResult);
```

我们可以将前面的代码更改为以下内容而不改变功能：

```java
Supplier<Integer> input = () -> 7;
Predicate<Integer> test = d -> d < 5.;
Function<Integer, Double> multiplyByFive = i -> i * 5.;;
Consumer<Double> printResult = 
                     d -> System.out.println("Result=" + d + " Great!");
supplyDecideProcessAndConsume(input, test, multiplyByFive, printResult); 

```

我们甚至可以内联前面的函数，并像这样一行写出前面的代码：

```java
supplyDecideProcessAndConsume(() -> 7, d -> d < 5, i -> i * 5., 
                    d -> System.out.println("Result=" + d + " Great!")); 

```

注意定义打印函数的透明度提高了多少。这就是 lambda 表达式与函数接口结合的力量和美丽所在。在第十八章，*流和管道*，你将看到 lambda 表达式实际上是处理流数据的唯一方法。

# Lambda 的限制

有两个我们想指出和澄清的 lambda 表达式方面，它们是：

+   如果 lambda 表达式使用在其外部创建的局部变量，则此局部变量必须是 final 或有效 final（在同一上下文中不可重新赋值）

+   lambda 表达式中的 `this` 关键字引用的是封闭上下文，而不是 lambda 表达式本身

# 有效 final 局部变量

与匿名类一样，创建在 lambda 表达式外部并在内部使用的变量将变为有效 final，并且不能被修改。你可以编写以下内容：

```java
int x = 7;
//x = 3;       //compilation error
int y = 5;
double z = 5.;
supplyDecideProcessAndConsume(() -> x, d -> d < y, i -> i * z,
            d -> { //x = 3;      //compilation error
                   System.out.println("Result=" + d + " Great!"); } );

```

但是，正如你所看到的，我们不能改变 lambda 表达式中使用的局部变量的值。这种限制的原因在于函数可以被传递并在不同的上下文中执行（例如，不同的线程），尝试同步这些上下文会破坏状态无关函数和表达式的独立分布式评估的原始想法。这就是为什么 lambda 表达式中使用的所有局部变量都是有效 final 的原因，这意味着它们可以明确声明为 final，也可以通过它们在 lambda 表达式中的使用变为 final。

这个限制有一个可能的解决方法。如果局部变量是引用类型（但不是 `String` 或原始包装类型），即使该局部变量用于 lambda 表达式中，也可以更改其状态：

```java
class A {
  private int x;
  public int getX(){ return this.x; }
  public void setX(int x){ this.x = x; }
}
void localVariable2(){
  A a = new A();
  a.setX(7);
  a.setX(3);
  int y = 5;
  double z = 5.;
  supplyDecideProcessAndConsume(() -> a.getX(), d -> d < y, i -> i * z,
               d -> { a.setX(5);
    System.out.println("Result=" + d + " Great!"); } );
}
```

但是，只有在真正需要的情况下才应该使用这种解决方法，并且必须谨慎进行，因为存在意外副作用的危险。

# 关于 this 关键字的解释

匿名类和 lambda 表达式之间的一个主要区别是对`this`关键字的解释。在匿名类内部，它引用匿名类的实例。在 lambda 表达式内部，`this`引用包围表达式的类实例，也称为*包围实例*、*包围上下文*或*包围范围*。

让我们编写一个演示区别的`ThisDemo`类：

```java
class ThisDemo {
  private String field = "ThisDemo.field";
  public void useAnonymousClass() {
    Consumer<String> consumer = new Consumer<>() {
      private String field = "AnonymousClassConsumer.field";
      public void accept(String s) {
        System.out.println(this.field);
      }
    };
    consumer.accept(this.field);
  }
  public void useLambdaExpression() {
    Consumer<String> consumer = consumer = s -> {
      System.out.println(this.field);
    };
    consumer.accept(this.field);
  }

}
```

正如您所看到的，匿名类中的`this`指的是匿名类实例，而 lambda 表达式中的`this`指的是包围表达式的类实例。Lambda 表达式确实没有字段，也不能有字段。 如果执行前面的方法，输出将确认我们的假设：

```java
ThisDemo d = new ThisDemo();
d.useAnonymousClass();   //prints: AnonymousClassConsumer.field
d.useLambdaExpression(); //prints: ThisDemo.field

```

Lambda 表达式不是类的实例，不能通过`this`引用。根据 Java 规范，这种方法*通过将[this]与所在上下文中的相同方式来处理，* *允许更多实现的灵活性*。

# 方法引用

让我们再看一下我们对`supplyDecidePprocessAndConsume()`方法的最后一个实现：

```java
supplyDecideProcessAndConsume(() -> 7, d -> d < 5, i -> i * 5., 
                    d -> System.out.println("Result=" + d + " Great!")); 
```

我们使用的功能相当琐碎。在现实代码中，每个都可能需要多行实现。在这种情况下，将代码块内联会使代码几乎不可读。在这种情况下，引用具有必要实现的方法是有帮助的。让我们假设我们有以下的`Helper`类：

```java
public class Helper {
  public double calculateResult(int i){
    // Maybe many lines of code here
    return i* 5;
  }
  public static void printResult(double d){
    // Maybe many lines of code here
    System.out.println("Result=" + d + " Great!");
  }
}
```

`Lambdas`类中的 lambda 表达式可以引用`Helper`和`Lambdas`类的方法，如下所示：

```java
public class Lambdas {
  public void methodReference() {
    Supplier<Integer> input = () -> generateInput();
    Predicate<Integer> test = d -> checkValue(d);
    Function<Integer, Double> multiplyByFive = 
                                  i -> new Helper().calculateResult(i);
    Consumer<Double> printResult = d -> Helper.printResult(d);
    supplyDecideProcessAndConsume(input, test, 
                                           multiplyByFive, printResult);
  }
  private int generateInput(){
    // Maybe many lines of code here
    return 7;
  }
  private static boolean checkValue(double d){
    // Maybe many lines of code here
    return d < 5;
  }
}
```

前面的代码已经更易读了，函数还可以再次内联：

```java
supplyDecideProcessAndConsume(() -> generateInput(), d -> checkValue(d), 
            i -> new Helper().calculateResult(i), Helper.printResult(d));
```

但在这种情况下，表示法可以做得更紧凑。当一个单行 lambda 表达式由对现有方法的引用组成时，可以通过使用不列出参数的方法引用进一步简化表示法。

方法引用的语法为`Location::methodName`，其中`Location`表示`methodName`方法所在的位置（对象或类），两个冒号(`::`)用作位置和方法名之间的分隔符。如果在指定位置有多个同名方法（因为方法重载的原因），则通过 lambda 表达式实现的函数接口抽象方法的签名来标识引用方法。

使用方法引用，`Lambdas`类中`methodReference()`方法下的前面代码可以重写为：

```java
Supplier<Integer> input = this::generateInput;
Predicate<Integer> test = Lambdas::checkValue;
Function<Integer, Double> multiplyByFive = new Helper()::calculateResult;;
Consumer<Double> printResult = Helper::printResult;
supplyDecideProcessAndConsume(input, test, multiplyByFive, printResult);

```

内联这样的函数更有意义：

```java
supplyDecideProcessAndConsume(this::generateInput, Lambdas::checkValue, 
                    new Helper()::calculateResult, Helper::printResult);

```

您可能已经注意到，我们有意地使用了不同的位置和两个实例方法以及两个静态方法，以展示各种可能性。

如果觉得记忆负担过重，好消息是现代 IDE（例如 IntelliJ IDEA）可以为您执行此操作，并将您正在编写的代码转换为最紧凑的形式。

# 练习 - 使用方法引用创建一个新对象

使用方法引用来表示创建一个新对象。假设我们有`class A{}`。用方法引用替换以下的`Supplier`函数声明，以另一个使用方法引用的声明替代：

```java
Supplier<A> supplier = () -> new A();

```

# 答案

答案如下：

```java
Supplier<A> supplier = A::new;

```

# 摘要

本章介绍了函数式编程的概念。它提供了 JDK 提供的函数式接口的概述，并演示了如何使用它们。它还讨论并演示了 lambda 表达式，以及它们如何有效地提高代码可读性。

下一章将使读者熟悉强大的数据流处理概念。它解释了什么是流，如何创建它们和处理它们的元素，以及如何构建处理流水线。它还展示了如何轻松地将流处理组织成并行处理。
