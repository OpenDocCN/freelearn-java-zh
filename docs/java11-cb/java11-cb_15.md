# 使用 Java 10 和 Java 11 进行编码的新方式

在本章中，我们将介绍以下示例：

+   使用局部变量类型推断

+   使用 lambda 参数的局部变量语法

# 介绍

本章为您快速介绍了影响您编码的新功能。许多其他语言，包括 JavaScript，都具有此功能——使用`var`关键字声明变量（在 Java 中，它实际上是一个**保留类型名称**，而不是关键字）。它有许多优点，但也存在争议。如果过度使用，特别是对于短的非描述性标识符，可能会使代码变得不太可读，而增加的价值可能会被增加的代码模糊所淹没。

这就是为什么在下面的示例中，我们解释了引入保留`var`类型的原因。尽量避免在其他情况下使用`var`。

# 使用局部变量类型推断

在本示例中，您将了解到局部变量类型推断，它是在 Java 10 中引入的，可以在哪里使用以及其限制。

# 准备工作

局部变量类型推断是编译器使用表达式的正确侧来识别局部变量类型的能力。在 Java 中，使用`var`标识符声明具有推断类型的局部变量。例如：

```java
var i = 42;       //int
var s = "42";     //String
var list1 = new ArrayList();          //ArrayList of Objects;
var list2 = new ArrayList<String>();  //ArrayList of Strings
```

前面变量的类型是明确可识别的。我们在注释中捕获了它们的类型。

请注意，`var`不是关键字，而是一个标识符，具有作为局部变量声明类型的特殊含义。

它确实节省了输入，并使代码不再被重复的代码淹没。让我们看看这个例子：

```java
Map<Integer, List<String>> idToNames = new HashMap<>();
//...
for(Map.Entry<Integer, List<String>> e: idToNames.entrySet()){
    List<String> names = e.getValue();
    //...
}
```

这是实现这样的循环的唯一方法。但自 Java 10 以来，可以写成如下形式：

```java
var idToNames = new HashMap<Integer, List<String>>();
//...
for(var e: idToNames.entrySet()){
    var names = e.getValue();
    //...
}
```

正如您所看到的，代码变得更清晰，但使用更具描述性的变量名（如`idToNames`和`names`）是有帮助的。无论如何，这都是有帮助的。但是，如果您不注意变量名，很容易使代码变得难以理解。例如，看看以下代码：

```java
var names = getNames();
```

看看前一行，你不知道`names`变量的类型是什么。将其更改为`idToNames`会更容易猜到。然而，许多程序员不这样做。他们更喜欢使用简短的变量名，并使用 IDE 上下文支持来确定每个变量的类型（在变量名后添加一个点）。但归根结底，这只是一种风格和个人偏好的问题。

另一个潜在的问题来自于新的风格可能违反封装和编码到接口原则，如果不额外注意的话。例如，考虑这个接口及其实现：

```java
interface A {
    void m();
}

static class AImpl implements A {
    public void m(){}
    public void f(){}
}
```

请注意，`AImpl`类具有比其实现的接口更多的公共方法。创建`AImpl`对象的传统方式如下：

```java
A a = new AImpl();
a.m();
//a.f();  //does not compile

```

这样，我们只暴露接口中存在的方法，而新的风格允许访问所有方法：

```java
var a = new AImpl();
a.m();
a.f();

```

为了限制仅引用接口的方法，需要添加类型转换，如下所示：

```java
var a = (A) new AImpl();
a.m();
//a.f();  //does not compile
```

因此，像许多强大的工具一样，新的风格可以使您的代码更易于编写和更易于阅读，或者如果不特别注意，可能会使代码变得不太可读并且更难以调试。

# 如何做...

您可以以以下方式使用局部变量类型：

+   使用右侧初始化器：

```java
var i = 1; 
var a = new int[2];
var l = List.of(1, 2); 
var c = "x".getClass(); 
var o = new Object() {}; 
var x = (CharSequence & Comparable<String>) "x";
```

以下声明和赋值是非法的，不会编译：

```java
var e;                 // no initializer
var g = null;          // null type
var f = { 6 };         // array initializer
var g = (g = 7);       // self reference is not allowed
var b = 2, c = 3.0;    // multiple declarators re not allowed
var d[] = new int[4];  // extra array dimension brackets
var f = () -> "hello"; // lambda requires an explicit target-type
```

通过扩展，在循环中使用初始化器：

```java
for(var i = 0; i < 10; i++){
    //...
}
```

我们已经谈到了这个例子：

```java
var idToNames = new HashMap<Integer, List<String>>();
//...
for(var e: idToNames.entrySet()){
    var names = e.getValue();
    //...
}
```

+   作为匿名类引用：

```java
interface A {
 void m();
}

var aImpl = new A(){
 @Override
 public void m(){
 //...
 }
};
```

+   作为标识符：

```java
var var = 1;
```

+   作为方法名：

```java
public void var(int i){
    //...
}
```

但`var`不能用作类或接口名称。

+   作为包名：

```java
package com.packt.cookbook.var;
```

# 使用 lambda 参数的局部变量语法

在本示例中，您将学习如何使用局部变量语法（在前一个示例中讨论）用于 lambda 参数以及引入此功能的动机。它是在 Java 11 中引入的。

# 准备工作

直到 Java 11 发布之前，有两种声明参数类型的方式——显式和隐式。以下是显式版本：

```java
BiFunction<Double, Integer, Double> f = (Double x, Integer y) -> x / y;
System.out.println(f.apply(3., 2));    //prints: 1.5
```

以下是隐式参数类型定义：

```java
BiFunction<Double, Integer, Double> f = (x, y) -> x / y;
System.out.println(f.apply(3., 2));     //prints: 1.5
```

在上述代码中，编译器通过接口定义来确定参数的类型。

使用 Java 11，引入了另一种参数类型声明的方式——使用`var`标识符。

# 如何做...

1.  在 Java 11 之前的隐式版本的参数声明与以下参数声明完全相同：

```java
BiFunction<Double, Integer, Double> f = (var x, var y) -> x / y;
System.out.println(f.apply(3., 2));       //prints: 1.5

```

1.  新的局部变量样式语法允许我们在不明确定义参数类型的情况下添加注释：

```java
import org.jetbrains.annotations.NotNull;
...
BiFunction<Double, Integer, Double> f = 
 (@NotNull var x, @NotNull var y) -> x / y;
System.out.println(f.apply(3., 2));        //prints: 1.5
```

注释告诉处理代码的工具（例如 IDE）程序员的意图，因此它们可以在编译或执行过程中警告程序员，以防违反声明的意图。例如，我们尝试在 IntelliJ IDEA 中运行以下代码：

```java
BiFunction<Double, Integer, Double> f = (x, y) -> x / y;
System.out.println(f.apply(null, 2));    

```

运行时出现了`NullPointerException`。然后我们运行了以下代码（带有注释）：

```java
BiFunction<Double, Integer, Double> f4 = 
           (@NotNull var x, @NotNull var y) -> x / y;
Double j = 3.;
Integer i = 2;
System.out.println(f4.apply(j, i)); 
```

结果如下：

```java
Exception in thread "main" java.lang.IllegalArgumentException: 
Argument for @NotNull parameter 'x' of com/packt/cookbook/ch17_new_way/b_lambdas/Chapter15Var.lambda$main$4 must not be null
```

lambda 表达式甚至没有被执行。

1.  在 lambda 参数的情况下，局部变量语法的优势变得清晰起来，如果我们需要在参数是一个名字非常长的类的对象时使用注释。在 Java 11 之前，代码可能如下所示：

```java
BiFunction<SomeReallyLongClassName, 
  AnotherReallyLongClassName, Double> f4 = 
    (@NotNull SomeReallyLongClassName x, 
     @NotNull AnotherReallyLongClassName y) -> x.doSomething(y);

```

我们不得不明确声明变量的类型，因为我们想要添加注释，而以下的隐式版本甚至无法编译：

```java
BiFunction<SomeReallyLongClassName, 
   AnotherReallyLongClassName, Double> f4 = 
      (@NotNull x, @NotNull y) -> x.doSomething(y);
```

使用 Java 11，新的语法允许我们使用`var`标识符进行隐式参数类型推断：

```java
BiFunction<SomeReallyLongClassName, 
   AnotherReallyLongClassName, Double> f4 = 
      (@NotNull var x, @NotNull var y) -> x.doSomething(y);
```

这就是引入 lambda 参数声明的局部变量语法的优势和动机。
