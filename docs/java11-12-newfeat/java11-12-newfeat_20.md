# 第十六章：Lambda 剩余

想象一下在方法或 lambda 表达式中标记未使用参数的便利性，并且不需要传递任何任意值以符合语法。此外，当与 lambda 一起工作时，想象一下声明 lambda 参数名称的能力，而不必关心在封装作用域中是否已经使用了相同的变量名。这就是 **lambda 剩余**（JEP 302）提供的内容，以增强 lambda 和方法引用。除了这些增强功能之外，它还将提供更好的方法中函数表达式的去歧义（这在 **JDK 增强提案**（**JEP**）中被标记为可选，截至目前）。

在本章中，我们将涵盖以下主题：

+   使用下划线标记未使用的方法参数

+   隐藏 lambda 参数

+   函数表达式去歧义

# 技术要求

本章中的代码将使用 lambda 剩余（JEP 302）中定义的功能，该功能尚未针对 JDK 发布版本进行目标化。要实验代码，你可以克隆相关的仓库。

本章中的所有代码都可以在 [`github.com/PacktPublishing/Java-11-and-12-New-Features`](https://github.com/PacktPublishing/Java-11-and-12-New-Features) 上访问。

让我们从了解为什么需要标记未使用的 lambda 参数开始。

# 使用下划线标记未使用参数

假设你在餐厅被提供了四种食物，但你没有吃掉其中的三种。如果餐厅有让顾客标记 **ort**（字面意思是 *已使用*）和非 ort 食物的协议，那么桌上的食物可以以某种方式使用。例如，餐厅可以将非 ort 项目标记为 *可食用*，并考虑与需要的人分享。

类似地，在调用方法或 lambda 表达式时，你可能不需要所有的方法参数。在这种情况下，向编译器传达你的意图（即某些参数没有被使用）是一个好主意。这有两个好处——它节省了编译器检查不必要值的类型检查，并且它节省了你传递任何任意值以匹配代码定义和代码调用语法的麻烦。

# lambda 参数的示例

以下是一个示例，演示了如何使用下划线 `_` 来标记未使用的 lambda 参数。以下代码使用了可以接受两个参数（`T` 和 `U`）并返回类型为 `R` 的值的 `BiFunction<T, U, R>` 函数式接口：

```java
BiFunction<Integer, String, Boolean> calc = (age, _) -> age > 10;
```

在前面的示例中，由于分配给 `BiFunction` 函数式接口的 lambda 表达式只使用了其中一个方法参数（即 `age`），JEP 302 建议使用下划线来标记未使用的参数。

以下代码突出了一些用例，以说明在没有标记未使用的 lambda 参数的便利性下，如何使用相同的代码（注释说明了代码传递给未使用参数的值）：

```java
// Approach 1
// Pass null to the unused parameter
BiFunction<Boolean, Integer, String> calc = (age, null) -> age > 10;

// Approach 2
// Pass empty string to the unused parameter
BiFunction<Boolean, Integer, String> calc = (age, "") -> age > 10;

// Approach 3
// Pass ANY String value to the unused parameter -
// - doesn't matter, since it is not used
BiFunction<Boolean, Integer, String> calc = 
                       (age, "Ban plastic straws") -> age > 10;

// Approach 4
// Pass any variable (of the same type) to the unused parameter -
// - doesn't matter, since it is not used
BiFunction<Boolean, Integer, String> calc = (age, name) -> age > 10;
```

许多其他函数式编程语言使用类似的符号来标记未使用的参数。

# 到达那里的旅程

使用下划线标记未使用参数需要对 Java 语言进行更改。

直到 Java 8，下划线被用作有效的标识符。第一步是拒绝使用下划线作为常规标识符的权限。因此，Java 8 对使用 `_` 作为 lambda 形参名称的用法发出了编译器警告。这是简单的，并且没有向后兼容性问题，因为 lambda 在 Java 8 中被引入。继续前进，Java 9 将使用下划线作为参数名称的编译器警告消息替换为编译错误。

使用 JEP 302，开发者可以使用 `_` 来标记未使用的方法参数，用于以下情况：

+   Lambdas

+   方法

+   捕获处理器

在下一节中，你将看到（在将来）你的 lambda 参数将能够遮蔽其封闭作用域中具有相同名称的变量。

# Lambda 参数的遮蔽

Java 不允许在多个场景下声明具有相同名称的变量。例如，你无法在具有相同名称的类中定义实例和静态变量。同样，你无法在方法中定义具有相同名称的方法参数和局部变量。

然而，你可以定义与类中的实例或 `static` 变量具有相同名称的局部变量。这并不意味着你以后不能访问它们。要从方法中访问实例变量，你可以在变量名称前加上 `this` 关键字。

这些限制允许你访问作用域内的所有变量。

# 现有的 lambda 参数案例

当你编写 lambda 表达式时，你可以定义多个参数。以下是一些定义单个或多个参数的 lambda 示例：

```java
key -> key.uppercase();           // single lambda parameter

(int x, int y) -> x > y? x : y;   // two lambda parameters

(a, b, c, d) -> a + b + c + d;    // four lambda parameters

```

让我们使用前面代码中的一个 lambda 来进行流处理。以下示例将提供 `List` 中的字符串值作为输出，并转换为大写：

```java
List<String> talks = List.of("Kubernetes", "Docker", "Java 11");
talks.stream()
        .map(key -> key.toUpperCase())
        .forEach(System.out::prinltn);
```

到目前为止，一切顺利。但是，如果前面的代码定义在具有局部变量（例如，`key`（在第 3 行的代码上））的方法（例如，`process()`）中，并且该局部变量与 `key` lambda 参数（在第 5 行定义和使用）的名称重叠，会发生什么？请看以下代码：

```java
1\. void process() {
2\.     List<String> talks = List.of("Kubernetes", "Docker", "Java 11");
3\.     String key = "Docker"; // local variable key
4\.     talks.stream()
5\.         .map(key -> key.toUpperCase())       // WON'T compile: 'key' 
            redefined
6\.         .forEach(System.out::prinltn);
7\. }
```

目前，前面的代码无法编译，因为用于 `map()` 方法的 lambda 表达式中的 `key` 变量无法遮蔽在 `process()` 方法中定义的局部 `key` 变量。

# 为什么 lambda 参数应该遮蔽封闭变量？

当你编写 lambda 表达式时，你（通常）定义参数名称作为指示如何处理分配给它们的值的指示符。它们不是用来在封闭块中重用由变量引用的现有值的。

让我们回顾一下前面的代码：

```java
1\. String key = "Docker"; // local variable key
2\. talks.stream()
3\.     .map(key -> key.toUpperCase())         // WON'T compile : 'key' 
                                              // redefined
4\.     .forEach(System.out::println);
```

在前面的代码中，第`3`行（以粗体显示）的 lambda 表达式定义了一个 lambda 参数`key`，指定当传递一个值给它时，Java 应该在该值上调用`toUpperCase()`方法并返回结果值。

从这个例子中可以看出，`key` lambda 参数似乎与在封闭块中定义的局部`key`变量无关。因此，lambda 参数应该允许覆盖封闭块中具有相同名称的变量。

# 一些已知的问题

目前尚不清楚 lambda 表达式是否能够访问被 lambda 参数覆盖的封闭变量的值；如果可以，它是如何做到的？

例如，让我们通过将`toUppercase()`方法的调用替换为`concat()`方法的调用来修改前面的代码（变化以粗体显示）：

```java
1\. String key = "Docker"; // local variable key
2\. talks.stream()
3\.     .map(key -> key.concat(key))            
4\.     .forEach(System.out::prinltn);
```

在前面的代码中，想象一下第`3`行的 lambda 表达式需要访问第`1`行定义的`key`变量的值，因为它想要将其传递给`concat()`方法。到目前为止，尚未最终确定这是否会被允许。

如果允许这样做，Java 将需要找到一种方法来标记并清楚地区分 lambda 参数和封闭块中具有相同名称的其他变量。这将对于代码可读性是必要的——正如您所知，这是很重要的。

被覆盖的封闭变量的可访问性是与 lambda 参数覆盖相关的主要问题。

在下一节中，我们将探讨 Java 如何尝试解决重载方法调用，这些调用将函数式接口作为参数。

# 函数表达式的歧义消除

如果您认为 Java 是通过`var`关键字（Java 10）开始其类型推断之旅的，那么您需要重新考虑。类型推断是在 Java 5 中引入的，并且从那时起一直在增加覆盖范围。

在 Java 8 中，重载方法的解析被重构，以便允许使用类型推断。在引入 lambda 表达式和方法引用之前，对方法的调用是通过检查传递给它的参数类型（不考虑返回类型）来解决的。

在 Java 8 中，隐式 lambda 和隐式方法引用无法检查它们接受的值的类型，这导致了编译器能力的限制，以排除对重载方法的模糊调用。然而，编译器仍然可以通过其参数检查显式 lambda 和方法引用。为了您的信息，明确指定其参数类型的 lambda 被称为**显式 lambda**。

限制编译器的能力和以这种方式放宽规则是有意为之的。这降低了 lambda 类型检查的成本，并避免了脆弱性。

虽然这是一个有趣的功能，但函数表达式的歧义消除被计划为 JEP 302 中的一个可选功能，因为 Oracle 需要评估其对编译器实现的影响。

在深入研究提出的解决方案之前，让我们看看现有的问题，使用代码示例。

# 解决重载方法解析问题——传递 lambda 表达式

让我们讨论一下当 lambda 表达式作为方法参数传递时解决重载方法解析的现有问题。让我们定义两个接口，`Swimmer` 和 `Diver`，如下所示：

```java
interface Swimmer {
    boolean test(String lap);
}
interface Diver {
    String dive(int height);
}
```

在以下代码中，重载的 `evaluate` 方法接受 `Swimmer` 和 `Diver` 接口作为方法参数：

```java
class SwimmingMeet {
    static void evaluate(Swimmer swimmer) {   // code compiles

        System.out.println("evaluate swimmer");
    }
    static void evaluate(Diver diver) {      // code compiles

        System.out.println("evaluate diver");
    }
}
```

让我们在以下代码中调用重载的 `evaluate()` 方法：

```java
class FunctionalDisambiguation {
    public static void main(String args[]) {
        SwimmingMeet.evaluate(a -> false); // This code WON'T compile
    }
}
```

重新审视先前的代码中的 lambda：

```java
a -> false                               // this is an implicit lambda
```

由于先前的 lambda 表达式没有指定其输入参数的类型，它可以是 `String`（`test()` 方法和 `Swimmer` 接口）或 `int`（`dive()` 方法和 `Diver` 接口）。由于调用 `evaluate()` 方法的调用是模糊的，所以无法编译。

让我们在先前的代码中添加方法参数的类型，使其成为一个显式的 lambda：

```java
SwimmingMeet.evaluate((String a) -> false);         // This compiles!!
```

现在先前的调用不再模糊不清；lambda 表达式接受一个 `String` 类型的输入参数，并返回一个 `boolean` 值，这映射到接受 `Swimmer` 作为参数的 `evaluate()` 方法（`Swimmer` 接口中的功能 `test()` 方法接受一个 `String` 类型的参数）。

让我们看看如果将 `Swimmer` 接口修改为将 `lap` 参数的数据类型从 `String` 改为 `int` 会发生什么。为了避免混淆，所有代码将重复，修改的地方用粗体标出：

```java
interface Swimmer {                            // test METHOD IS 
                                               // MODIFIED
    boolean test(int lap);      // String lap changed to int lap

}
interface Diver {
    String dive(int height);
}
class SwimmingMeet {
    static void evaluate(Swimmer swimmer) {          // code compiles

        System.out.println("evaluate swimmer");
    }
    static void evaluate(Diver diver) {               // code compiles
        System.out.println("evaluate diver");
    }
}
```

考虑以下代码，思考哪一行代码可以编译：

```java
1\. SwimmingMeet.evaluate(a -> false);
2\. SwimmingMeet.evaluate((int a) -> false);
```

在先前的例子中，两个行号上的代码都无法编译，原因相同——编译器无法确定对重载的 `evaluate()` 方法的调用。由于两个功能方法（即 `Swimmer` 接口中的 `test()` 和 `Diver` 接口中的 `dive()`）都接受一个 `int` 类型的单个方法参数，编译器无法确定方法调用。

作为一名开发者，你可能会争辩说，由于 `test()` 和 `dive()` 方法的返回类型不同，编译器应该能够推断出正确的调用。只是为了重申，方法的返回类型不参与方法重载。重载方法必须在参数的数量或类型上返回。

方法的返回类型不参与方法重载。重载方法必须在参数的数量或类型上返回。

# 解决重载方法解析问题——传递方法引用

你可以定义具有不同参数类型的重载方法，如下所示：

```java
class Championship {
    static boolean reward(Integer lapTime) {
        return(lapTime < 60);
    }
    static boolean reward(String lap) {
        return(lap.equalsIgnoreCase("final ");
    }
}
```

然而，以下代码无法编译：

```java
someMethod(Chamionship::reward);                     // ambiguous call
```

在先前的代码行中，由于编译器不允许检查方法引用，代码无法编译。这是不幸的，因为重载方法的参数是 `Integer` 和 `String`——没有值可以与两者兼容。

# 提出的解决方案

对于使用 lambda 表达式或方法引用的重载方法，涉及到的意外编译问题可以通过允许编译器将它们的返回类型视为“也”来解决。然后编译器就能够选择正确的重载方法并消除不匹配的选项。

# 摘要

对于使用 lambda 和方法引用的 Java 开发者，本章展示了 Java 在管道中有什么可以帮助缓解问题的计划。

Lambda Leftovers (JEP 302) 提出在 lambda、方法和 catch 处理程序中使用下划线来表示未使用的参数。它计划允许开发者定义可以覆盖其封闭块中同名变量的 lambda 参数。功能表达式的歧义消除是一个重要且强大的功能。它将允许编译器考虑 lambda 的返回类型，以确定正确的重载方法。由于它可能影响编译器的工作方式，因此这个特性在这个 JEP 中被标记为可选。

在下一章，关于模式匹配和 switch 表达式的章节中，你将了解正在添加到 Java 语言中的令人兴奋的功能。

本章没有为读者包含任何编码练习。方法的重载不涉及方法的返回类型。重载的方法必须通过其参数的数量或类型来返回。
