# 第十八章：Lambda 表达式和函数式编程

本章解释了函数式编程的概念。它概述了 JDK 提供的函数式接口，解释了如何在 Lambda 表达式中使用它们，以及如何以最简洁的方式编写 Lambda 表达式。

在本章中，我们将涵盖以下主题：

+   函数式编程

+   函数式接口

+   Lambda 表达式

+   方法引用

+   练习-使用方法引用创建一个新对象

# 函数式编程

函数式编程允许我们将一块代码（一个函数）视为对象，将其作为参数或方法的返回值。这个特性存在于许多编程语言中。它不要求我们管理对象状态。函数是无状态的。它的结果只取决于输入数据，无论调用多少次。这种风格使结果更可预测，这是函数式编程最吸引人的方面。

没有函数式编程，将功能作为参数传递的唯一方法是通过编写实现接口的类，创建其对象，然后将其作为参数传递。但即使是最不涉及的样式-使用匿名类-也需要编写太多的样板代码。使用函数式接口和 Lambda 表达式使代码更短，更清晰，更具表现力。

将其添加到 Java 中可以通过将并行编程能力从客户端代码转移到库来增加。在此之前，为了处理 Java 集合的元素，客户端代码必须迭代集合并组织处理。在 Java 8 中，添加了接受函数（函数式接口的实现）作为参数然后将其应用于集合的每个元素的新（默认）方法，具体取决于内部处理算法是并行还是串行。因此，组织并行处理是库的责任。

在本章中，我们将定义和解释这些 Java 特性-函数式接口和 Lambda 表达式-并演示它们在代码示例中的适用性。它们使函数成为语言中与对象同等重要的一等公民。

# 什么是函数式接口？

实际上，您已经在我们的演示代码中看到了函数式编程的元素。一个例子是`forEach(Consumer consumer)`方法，对每个`Iterable`都可用，其中`Consumer`是一个函数式接口。另一个例子是`removeIf(Predicate predicate)`方法，对每个`Collection`对象都可用。传入的`Predicate`对象是一个函数-函数式接口的实现。类似地，`List`接口中的`sort(Comparator comparator)`和`replaceAll(UnaryOperator uo)`方法以及`Map`中的几个`compute()`方法都是函数式编程的例子。

函数式接口是一个只有一个抽象方法的接口，包括从父接口继承的方法。

为了避免运行时错误，Java 8 引入了`@FunctionalInterface`注解，告诉编译器意图，因此编译器可以检查注解接口中是否真的只有一个抽象方法。让我们回顾一下相同继承线的以下接口：

```java

@FunctionalInterface

接口 A {

void method1();

default void method2(){}

static void method3(){}

}

@FunctionalInterface

接口 B 扩展自 A {

default void method4(){}

}

@FunctionalInterface

接口 C 扩展自 B {

void method1();

}

//@FunctionalInterface  //编译错误

接口 D 扩展自 C {

void method5();

}

```

接口`A`是一个功能接口，因为它只有一个抽象方法：`method1()`。接口`B`也是一个功能接口，因为它也只有一个抽象方法-与从接口`A`继承的相同的`method1()`。接口`C`是一个功能接口，因为它只有一个抽象方法，`method1()`，它覆盖了父接口`A`的抽象`method1()`方法。接口`D`不能是一个功能接口，因为它有两个抽象方法-从父接口`A`继承的`method1()`和`method5()`。

当使用`@FunctionalInterface`注释时，它告诉编译器仅检查一个抽象方法的存在，并警告程序员，读取代码时，此接口只有一个抽象方法是有意的。否则，程序员可能会浪费时间增强接口，只是后来发现无法完成。

出于同样的原因，自 Java 早期版本以来就存在的`Runnable`和`Callable`接口在 Java 8 中被注释为`@FunctionalInterface`。它使这种区别变得明确，并提醒其用户和那些可能尝试添加另一个抽象方法的人：

```java

@FunctionalInterface

接口 Runnable {

void run();

}

@FunctionalInterface

接口 Callable<V> {

V call() throws Exception;

}

```

如您所见，创建功能接口很容易。但在这之前，考虑使用`java.util.function`包中提供的 43 个功能接口之一。

# 标准功能接口即用即用

`java.util.function`包中提供的大多数接口都是以下四个接口的特殊化：`Function`、`Consumer`、`Supplier`和`Predicate`。让我们先来回顾一下它们，然后简要概述其他 39 个标准功能接口。

# Function<T, R>

此及其他功能`<indexentry content="standard functional interfaces:function">`接口的表示法包括输入数据（`T`）和返回数据（`R`）类型的列表。因此，`Function<T, R>`表示此接口的唯一抽象方法接受`T`类型的参数并产生`R`类型的结果。您可以通过阅读在线文档找到该抽象方法的名称。在`Function<T, R>`接口的情况下，其方法是`R apply(T)`。

学习了所有这些之后，我们可以使用匿名类创建此接口的实现：

```java

Function<Integer, Double> multiplyByTen = new Function<Integer, Double>(){

public Double apply(Integer i){

return i * 10.0;

}

};

```

由程序员决定`T`（输入参数）将是哪种实际类型，`R`（返回值）将是哪种类型。在我们的例子中，我们已经决定输入参数将是`Integer`类型，结果将是`Double`类型。正如您现在可能已经意识到的那样，类型只能是引用类型，并且原始类型的装箱和拆箱是自动执行的。

现在我们可以根据需要使用我们的新`Function<Integer, Double> multiplyByTen`函数。我们可以直接使用它，如下所示：

```java

System.out.println(multiplyByTen.apply(1)); //prints: 10.0

```

或者我们可以创建一个接受此函数作为参数的方法：

```java

void useFunc(Function<Integer, Double> processingFunc, int input){

System.out.println(processingFunc.apply(input));

}

```

然后我们可以将我们的函数传递到这个方法中，并让方法使用它：

```java

useFunc(multiplyByTen, 10);     //prints: 100.00

```

我们还可以创建一个方法，每当需要时就会生成一个函数：

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

使用上述方法，我们可以编写以下代码：

```java

Function<Integer, Double> multiplyByFive = createMultiplyBy(5);

System.out.println(multiplyByFive.apply(1)); //prints: 5.0

useFunc(multiplyByFive, 10);                 //prints: 50.0

```

在下一节中，我们将介绍 lambda 表达式，并展示它们如何用更少的代码来表达函数接口的实现。

# 消费者<T>

通过查看`Consumer<T>`接口的定义，你已经猜到这个接口有一个接受`T`类型参数的抽象方法，并且不返回任何东西。从`Consumer<T>`接口的文档中，我们了解到它的抽象方法是`void accept(T)`，这意味着，例如，我们可以这样实现它：

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

消费者<Double> createPrintingFunc(String prefix, String postfix){

Consumer<Double> func = new Consumer<Double>() {

public void accept(Double d) {

System.out.println(prefix + d + postfix);

}

};

return func;

}

```

现在我们可以这样使用它：

```java

消费者<Double> printResult = createPrintingFunc("Result=", " Great!");

printResult.accept(10.0);    //prints: Result=10.0 Great!

```

我们还可以创建一个新的方法，不仅接受处理函数作为参数，还接受打印函数：

```java

void processAndConsume(int input,

功能<Integer, Double> processingFunc,

Consumer<Double> consumer){

consumer.accept(processingFunc.apply(input));

}

```

然后我们可以写下面的代码：

```java

功能<Integer, Double> multiplyByFive = createMultiplyBy(5);

消费者<Double> printResult = createPrintingFunc("Result=", " Great!");

processAndConsume(10, multiplyByFive, printResult); //Result=50.0 Great!

```

正如我们之前提到的，在下一节中，我们将介绍 lambda 表达式，并展示它们如何用更少的代码来表达函数接口的实现。

# 供应商<T>

这是一个技巧问题：猜猜`Supplier<T>`接口的抽象方法的输入和输出类型是什么。答案是：它不接受任何参数，返回`T`类型。现在你明白了，区别在于接口本身的名称。它应该给你一个提示：消费者只消费而不返回任何东西，而供应商只提供而不需要任何输入。`Supplier<T>`接口的抽象方法是`T get()`。

与之前的函数类似，我们可以编写生成供应商的方法：

```java

供应商<Integer> createSuppplier(int num){

供应商<Integer> func = new 供应商<Integer>() {

public Integer get() { return num; }

};

return func;

}

```

现在我们可以编写一个只接受函数的方法：

```java

void supplyProcessAndConsume(供应商<Integer> input,

功能<Integer, Double> process,

Consumer<Double> consume){

consume.accept(processFunc.apply(input.get()));

}

```

注意`input`函数的输出类型与`process`函数的输入类型相同，它返回的类型与`consume`函数消耗的类型相同。这使得下面的代码成为可能：```

```java

供应商<Integer> supply7 = createSuppplier(7);

功能<Integer, Double> multiplyByFive = createMultiplyBy(5);

消费者<Double> printResult = createPrintingFunc("Result=", " Great!");

supplyProcessAndConsume(supply7, multiplyByFive, printResult);

//prints: Result=35.0 Great!

```

此时，我们希望您开始欣赏函数式编程为我们带来的价值。它允许我们传递功能块，这些功能块可以插入到算法的中间，而无需创建对象。静态方法也不需要创建对象，但它们由 JVM 中的所有应用程序线程共享，因为它们在 JVM 中是唯一的。与此同时，每个函数都是一个对象，可以在 JVM 中是唯一的（如果分配给静态变量）或为每个处理线程创建（这通常是情况）。它的编码开销很小，在 lambda 表达式中使用时甚至可以更少——这是我们下一节的主题。

到目前为止，我们已经演示了如何将函数插入现有的控制流表达式中。现在我们将描述最后一个缺失的部分——代表决策构造的函数，它也可以作为对象传递。

# Predicate<T>

这是一个表示布尔值函数的接口，它有一个方法：`boolean test(T)`。以下是一个创建`Predicate<Integer>`函数的方法示例：

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

我们可以使用它来向处理方法添加一些逻辑：

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

以下代码演示了它的用法：

```java

Supplier<Integer> input = createSuppplier(7);

Predicate<Integer> test = createTestSmallerThan(5);

Function<Integer, Double> multiplyByFive = createMultiplyBy(5);

Consumer<Double> printResult = createPrintingFunc("Result=", " Great!");

supplyDecideProcessAndConsume(input, test, multiplyByFive, printResult);

//打印：Input 7 does not pass the test and not processed.

```

让我们以 3 为例设置输入：

```java

Supplier<Integer> input = createSuppplier(3)

```

前面的代码将产生以下输出：

```java

Result=15.0 Great!

```

# 其他标准函数接口

`java.util.function`包中的其他 39 个函数接口是我们刚刚审查的四个接口的变体。这些变体是为了实现以下一个或多个组合：

+   通过显式使用整数、双精度或长整型原始类型来避免自动装箱和拆箱，从而获得更好的性能

+   允许两个输入参数

+   更短的符号

以下是一些示例：

+   `IntFunction<R>`具有`R apply(int)`方法，提供了更短的符号（无需为输入参数类型使用泛型），并通过要求`int`原始类型作为参数来避免自动装箱

+   `BiFunction<T,U,R>`具有`R apply(T,U)`方法，允许两个输入参数

+   `BinaryOperator<T>`具有`T apply(T,T)`方法，允许`T`类型的两个输入参数，并返回相同`T`类型的值

+   `IntBinaryOperator`具有`int applAsInt(int,int)`方法，接受两个`int`类型的参数，并返回`int`类型的值

如果您要使用函数接口，我们鼓励您研究`java.util.functional`包的接口 API。

# 链接标准函数

`java.util.function`包中的大多数函数接口都有默认方法，允许我们构建一个函数链（也称为管道），将一个函数的结果作为输入参数传递给另一个函数，从而组成一个新的复杂函数。例如：

```java

Function<Double, Long> f1 = d -> Double.valueOf(d / 2.).longValue();

Function<Long, String> f2 = l -> "Result: " + (l + 1);

Function<Double, String> f3 = f1.andThen(f2);

System.out.println(f3.apply(4.));            //打印：3

```

从上面的代码中可以看出，我们通过使用`andThen()`方法组合了`f1`和`f2`函数，创建了一个新的`f3`函数。这就是我们将要在本节中探讨的方法的思想。首先，我们将函数表示为匿名类，然后在下一节中，我们将介绍在前面的示例中使用的 lambda 表达式。

# 链接两个 Function<T,R>

我们可以使用`Function`接口的`andThen(Function after)`默认方法。我们已经创建了`Function<Integer, Double> createMultiplyBy()`方法：

```java

Function<Integer, Double> createMultiplyBy(double num){

Function<Integer, Double> func = new Function<Integer, Double>(){

public Double apply(Integer i){

返回 i * num;

}

};

return func;

```

我们还可以编写另一个方法，创建一个带有`Double`输入类型的减法函数，这样我们就可以将其链接到乘法函数：

```java

private static Function<Double, Long> createSubtractInt(int num){

Function<Double, Long> func = new Function<Double, Long>(){

public Long apply(Double dbl){

返回 Math.round(dbl - num);

}

};

返回 func;

}

```

现在我们可以编写以下代码：

```java

Function<Integer, Double> multiplyByFive = createMultiplyBy(5);

System.out.println(multiplyByFive.apply(2));  //打印：10.0

Function<Double, Long> subtract7 = createSubtractInt(7);

System.out.println(subtract7.apply(11.0));   //打印：4

long r = multiplyByFive.andThen(subtract7).apply(2);

System.out.println(r);                          //打印：3

```

正如你所看到的，`multiplyByFive.andThen(subtract7)`链实际上是`Function<Integer, Long> multiplyByFiveAndSubtractSeven`。

`Function`接口还有另一个默认方法，`Function<V,R> compose(Function<V,T> before)`，它也允许我们链接两个函数。必须首先执行的函数可以作为`before`参数传递到第二个函数的`compose()`方法中：

```java

boolean r = subtract7.compose(multiplyByFive).apply(2);

System.out.println(r);                          //打印：3

```

# 链接两个 Consumer<T>

`Consumer`接口也有`andThen(Consumer after)`方法。我们已经编写了创建打印函数的方法：

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

现在我们可以创建并链接两个打印函数，如下所示：

```java

Consumer<Double> print21By = createPrintingFunc("21 by ", "");

Consumer<Double> equalsBy21 = createPrintingFunc("equals ", " by 21");

print21By.andThen(equalsBy21).accept(2d);

//打印：21 by 2.0

// 由 21 等于 2.0

```

正如您在`Consumer`链中所看到的，两个函数按照链条定义的顺序消耗相同的值。

# 链接两个 Predicate<T>

`Supplier`接口没有默认方法，而`Predicate`接口有一个静态方法`isEqual(Object targetRef)`和三个默认方法：`and(Predicate other)`、`negate()`和`or(Predicate other)`。为了演示`and(Predicate other)`和`or(Predicate other)`方法的使用，例如，让我们编写创建两个`Predicate<Double>`函数的方法。一个函数检查值是否小于输入：

```java

Predicate<Double> testSmallerThan(double limit){

Predicate<Double> func = new Predicate<Double>() {

public boolean test(Double num) {

System.out.println("Test if " + num + " is smaller than " + limit);

返回 num < limit;

}

};

返回 func;

}

```

另一个函数检查值是否大于输入：

```java

Predicate<Double> testBiggerThan(double limit){

Predicate<Double> func = new Predicate<Double>() {

public boolean test(Double num) {

System.out.println("Test if " + num + " is bigger than " + limit);

返回 num > limit;

}

};

return func;

}

```

现在我们可以创建两个 `Predicate<Double>` 函数并将它们链接起来：

```java

Predicate<Double> isSmallerThan20 = testSmallerThan(20d);

System.out.println(isSmallerThan20.test(10d));

//打印：测试 10.0 是否小于 20.0

//        true

Predicate<Double> isBiggerThan18 = testBiggerThan(18d);

System.out.println(isBiggerThan18.test(10d));

//打印：测试 10.0 是否大于 18.0

//        false

boolean b = isSmallerThan20.and(isBiggerThan18).test(10.);

System.out.println(b);

//打印：测试 10.0 是否小于 20.0

//        测试 10.0 是否大于 18.0

//        false

b = isSmallerThan20.or(isBiggerThan18).test(10.);

System.out.println(b);

//打印：测试 10.0 是否小于 20.0

//        true

```

正如你所看到的，`and()` 方法需要执行每个函数，而 `or()` 方法在链中的第一个函数返回 `true` 时不执行第二个函数。

# identity() 和其他默认方法

`java.util.function` 包的函数接口还有其他有用的默认方法。其中最突出的是 `identity()` 方法，它返回一个始终返回其输入参数的函数：

```java

Function<Integer, Integer> id = Function.identity();

System.out.println(id.apply(4));          //打印：4

```

`identity()` 方法在某些程序需要提供某个函数，但你不希望提供的函数改变任何东西时非常有用。在这种情况下，你可以创建一个具有必要输出类型的恒等函数。例如，在我们之前的代码片段中，我们可能决定 `multiplyByFive` 函数不应该在 `multiplyByFive.andThen(subtract7)` 链中改变任何东西：

```java

Function<Double, Double> multiplyByFive = Function.identity();

System.out.println(multiplyByFive.apply(2.));  //打印：2.0

Function<Double, Long> subtract7 = createSubtractInt(7);

System.out.println(subtract7.apply(11.0));    //打印：4

long r = multiplyByFive.andThen(subtract7).apply(2.);

System.out.println(r);                       //打印：-5

```

正如你所看到的，`multiplyByFive` 函数没有对输入参数 `2` 做任何处理，所以结果（减去 `7` 后）是 `-5`。

其他默认方法大多与转换和装箱和拆箱有关，但也包括提取两个参数的最小值和最大值。如果你感兴趣，可以查看一下 `java.util.function` 包接口的 API，了解一下可能性。

# Lambda 表达式

在上一节中的示例（使用匿名类实现函数接口）看起来笨重且过于冗长。首先，没有必要重复接口名称，因为我们已经将其声明为对象引用的类型。其次，在只有一个抽象方法的函数接口的情况下，没有必要指定必须实现的方法名称。编译器和 Java 运行时可以弄清楚。我们只需要提供新的功能。Lambda 表达式就是为了这个目的而引入的。

# 什么是 lambda 表达式？

术语 lambda 来自于 lambda 演算——一种通用的计算模型，可以用来模拟任何图灵机。它是由数学家阿隆佐·邱奇在 20 世纪 30 年代引入的。lambda 表达式是一个函数，在 Java 中实现为匿名方法，它还允许我们省略修饰符、返回类型和参数类型。这使得它的表示非常紧凑。

lambda 表达式的语法包括参数列表、箭头标记 `->` 和主体。参数列表可以是空的 `()`，如果只有一个参数，则不需要括号，或者由括号括起来的逗号分隔的参数列表。主体可以是单个表达式或语句块。

让我们看一些例子：

+   `() -> 42;` 总是返回 `42`

+   `x -> x + 1;` 将 `x` 变量增加 `1`

+   `(x, y) -> x * y;` multiplies `x` by `y` and returns the result

+   `(char x) -> x == '$';` compares the value of the `x` variable and the `$` symbol, and returns a Boolean value

+   `x -> {  System.out.println("x=" + x); };` prints the `x` value with the `x=` prefix

# Re-implementing functions

We can rewrite our functions, created in the previous section, using lambda expressions, as follows:

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

We don't repeat the name of the implemented interface because it is specified as the return type in the method signature. And we do not specify the name of the abstract method either because it is the only method of the interface that has to be implemented. Writing such a compact and efficient code became possible because of the combination of the lambda expression and functional interface.

Looking at the preceding examples, you probably realize that there is no need to have methods that create a function anymore. Let's change the code that calls the `supplyDecideProcessAndConsume()` method:

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

Let's revisit the following lines:

```java

Supplier<Integer> input = createSuppplier(7);

Predicate<Integer> test = createTestSmallerThan(5);

Function<Integer, Double> multiplyByFive = createMultiplyBy(5);

Consumer<Double> printResult = createPrintingFunc("Result=", " Great!");

supplyDecideProcessAndConsume(input, test, multiplyByFive, printResult);

```

We can change the preceding code to the following without changing the functionality:

```java

Supplier<Integer> input = () -> 7;

Predicate<Integer> test = d -> d < 5.;

Function<Integer, Double> multiplyByFive = i -> i * 5.;;

Consumer<Double> printResult =

d -> System.out.println("Result=" + d + " Great!");

supplyDecideProcessAndConsume(input, test, multiplyByFive, printResult);

```

We can even inline the preceding functions and write the preceding code in one line like this:

```java

supplyDecideProcessAndConsume(() -> 7, d -> d < 5, i -> i * 5.,

d -> System.out.println("Result=" + d + " Great!"));

```

Notice how much more transparent the definition of the printing function has become. That is the power and the beauty of lambda expressions in combination with functional interfaces. In Chapter 18, *Streams and Pipelines*, you will see that lambda expressions are, in fact, the only way to process streamed data.

# Lambda limitations

There are two aspects of a lambda expression that we would like to point out and clarify, which are:

+   If a lambda expression uses a local variable created outside it, this local variable has to be final or effectively final (not re-assigned in the same context)

+   The `this` keyword in a lambda expression refers to the enclosing context, and not the lambda expression itself

# Effectively final local variable

As in the anonymous class, the variable, created outside and used inside the lambda expression, becomes effectively final and cannot be modified. You can write the following:

```java

int x = 7;

//x = 3;       //compilation error

int y = 5;

double z = 5.;

supplyDecideProcessAndConsume(() -> x, d -> d < y, i -> i * z,

d -> { //x = 3;      //compilation error

System.out.println("Result=" + d + " Great!"); } );

```

但是，正如你所看到的，我们无法改变 lambda 表达式中使用的局部变量的值。这种限制的原因是函数可以在不同的上下文（例如不同的线程）中传递并执行，尝试同步这些上下文将破坏*无状态函数*和表达式的独立分布式评估的原始想法。这就是为什么 lambda 表达式中使用的所有局部变量实际上都是 final 的原因，这意味着它们可以显式地声明为 final，也可以通过在 lambda 表达式中使用而变为 final。

这种限制有一个可能的变通方法。如果局部变量是引用类型（但不是`String`或原始包装类型），即使该局部变量在 lambda 表达式中使用，也可以改变其状态：

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

但是这种变通方法只有在真正需要时才应该使用，并且必须小心操作，因为存在意外副作用的危险。

# this 关键字的解释

匿名类和 lambda 表达式之间的一个主要区别是对`this`关键字的解释。在匿名类中，它指的是匿名类的实例。在 lambda 表达式中，`this`指的是包围表达式的类实例，也称为*包围实例*、*包围上下文*或*包围范围*。

让我们编写一个`ThisDemo`类来说明这种差异：

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

正如你所看到的，匿名类中的`this`指的是匿名类实例，而 lambda 表达式中的`this`指的是包围表达式的类实例。Lambda 表达式根本没有字段，也不能有字段。如果我们执行前面的方法，输出将确认我们的假设：

```java

ThisDemo d = new ThisDemo();

d.useAnonymousClass();   //输出：AnonymousClassConsumer.field

d.useLambdaExpression(); //输出：ThisDemo.field

```

lambda 表达式不是类实例，也不能被`this`引用。根据 Java 规范，这种方法*允许更多的实现灵活性*，*将[this]视为与周围上下文中的相同*。

# 方法引用

让我们来看看我们对`supplyDecidePprocessAndConsume()`方法的最后一个实现：

```java

supplyDecideProcessAndConsume(() -> 7, d -> d < 5, i -> i * 5.,

d -> System.out.println("Result=" + d + " Great!"));

```

我们使用的函数都相当简单。在实际代码中，每个函数可能需要多行实现。在这种情况下，将代码块内联会使代码几乎无法阅读。在这种情况下，引用具有必要实现的方法是有帮助的。假设我们有以下`Helper`类：

```java

public class Helper {

public double calculateResult(int i){

// 这里可能有很多行代码

return i* 5;

}

public static void printResult(double d){

// 这里可能有很多行代码

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

下一章将使读者熟悉数据流处理的强大概念。它解释了流是什么，如何创建它们并处理它们的元素，以及如何构建处理管道。它还展示了如何轻松地将流处理组织成并行处理。

Consumer<Double> printResult = d -> Helper.printResult(d);

使用方法引用来表示创建一个新对象。假设我们有`class A{}`。用另一个使用方法引用的`Supplier`函数声明替换以下内容：

supplyDecideProcessAndConsume(input, test, multiplyByFive, printResult);

}

private int generateInput(){

// 这里可能有很多行代码

return 7;

}

private static boolean checkValue(double d){

答案

return d < 5;

Supplier<A> supplier = A::new;

// 这里可能有很多行代码

```

前述代码已经读起来更好了，函数可以再次内联：

}

supplyDecideProcessAndConsume(() -> generateInput(), d -> checkValue(d),

i -> new Helper().calculateResult(i), Helper.printResult(d));

supplyDecideProcessAndConsume(input, test,

但在这种情况下，表示法甚至可以更加简洁。当一行 lambda 表达式由对现有方法的引用组成时，可以通过使用方法引用进一步简化表示法，而无需列出参数。

方法引用的语法是`Location::methodName`，其中`Location`表示可以找到`methodName`方法的位置（在哪个对象或类中），两个冒号（`::`）作为位置和方法名之间的分隔符。如果在指定位置有多个同名方法（因为方法重载），则引用方法由 lambda 表达式实现的函数式接口的抽象方法的签名来确定。

使用方法引用，`Lambdas`类中`methodReference()`方法下的前述代码可以重写如下：

```java

Function<Integer, Double> multiplyByFive = new Helper()::calculateResult;;

```java

将上述文本按行翻译成中文，不要输出原文：

```

}

multiplyByFive, printResult);

要内联这样的函数更有意义：

```java

supplyDecideProcessAndConsume(this::generateInput, Lambdas::checkValue,

new Helper()::calculateResult, Helper::printResult);

Consumer<Double> printResult = Helper::printResult;

您可能已经注意到，我们故意在不同位置使用了两个实例方法和两个静态方法，以展示各种可能性。

如果感觉记住太多，好消息是现代 IDE（例如 IntelliJ IDEA）可以为您完成，并将您正在编写的代码转换为最紧凑的形式。

# 练习 - 使用方法引用创建一个新对象

Predicate<Integer> test = Lambdas::checkValue;

```java

Supplier<A> supplier = () -> new A();

本章介绍了函数式编程的概念。它概述了 JDK 提供的函数式接口，并演示了如何使用它们。它还讨论并演示了 lambda 表达式以及它们如何有效地提高代码的可读性。

# i -> new Helper().calculateResult(i);

答案是：

```java

Supplier<Integer> input = this::generateInput;

```

# 总结

```

```
