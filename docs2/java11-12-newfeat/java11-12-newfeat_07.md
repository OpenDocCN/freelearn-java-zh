# 第五章：Lambda 参数的局部变量语法

Java 通过扩展保留类型`var`在 lambda 参数中的使用来增强其语言。这种增强的唯一目的是使 lambda 参数的语法与使用`var`声明局部变量的语法保持一致。自 lambda 引入以来，Java 8 的编译器已经推断出隐式类型化 lambda 表达式的参数类型。

在本章中，我们将涵盖以下主题：

+   隐式和显式类型化的 lambda 参数

+   如何使用`var`与 lambda 参数一起使用

+   使用 lambda 参数的注解

# 技术要求

要执行本章中的代码，你必须在你的系统上安装 JDK 11（或更高版本）。本章中的所有代码都可以在[`github.com/PacktPublishing/Java-11-and-12-New-Features`](https://github.com/PacktPublishing/Java-11-and-12-New-Features)找到[。](https://github.com/PacktPublishing/Java-11-and-12-New-Features)

# Lambda 表达式

**lambda 表达式**是一个匿名函数，可以接受输入参数并返回一个值。lambda 表达式可以指定所有（或没有）输入参数的类型；lambda 表达式可以是显式类型化的或隐式类型化的。

# 显式类型化的 lambda 表达式

明确指定所有输入参数类型的 lambda 表达式被称为显式类型化的 lambda 表达式。以下代码展示了几个示例（输入参数以粗体显示）：

```java
(Integer age) -> age > 10;                      // input Integer, 
                                                // return Boolean 
(Integer age) -> age > 10? "Kid" : "Not a Kid"; // input Integer, 
                                                // return String 
(Integer age) -> {System.out.println();};       // input Integer, 
                                                // return void 
() -> {return Math.random() + "Number";};       // input none,
                                                // return String 
(String name, List<Person> list) -> { 
                               return ( 
                                   list.stream() 
                                       .filter(e ->             
                                        e.getName().startsWith(name)) 
                                       .map(Person::getAge) 
                                       .findFirst() 
                                   ); 
                               };               // input name, 
                                                // List<person> 
                                                // return 
                                                // Optional<Integer> 
```

在所有前面的示例中，代码明确地定义了传递给它的所有参数的类型。如果一个 lambda 表达式不接受任何参数，它使用一对空圆括号（`()`）。

如果这让你想知道 lambda 表达式将被分配给哪些变量的类型，以下是完整的代码供你参考：

```java
Predicate<Integer> predicate = (Integer age) -> age > 10; 
Function<Integer, String> function = (Integer age) -> age > 10? "Kid" : 
                                    "Not a Kid"; 
Consumer<Integer> consumer =   (Integer age) -> {  
                                                System.out.println();
                                                }; 
Supplier<String> supplier =    () -> {
                                         return Math.random() + 
                                         "Number";
                                     }; 

BiFunction<String, List<Person>,
 Optional<Integer>> firstElement = (String name, List<Person> list) ->                                                                 { 
                                   return ( 
                                       list.stream() 
                                           .filter(e -> 
                                            e.getName().
                                            startsWith(name)) 
                                           .map(Person::getAge) 
                                          .findFirst() 
                                       ); 
                                   }; 
class Person { 
    int age; 
    String name; 
    String getName() { 
        return name; 
    } 
    Integer getAge() { 
        return age; 
    } 
} 
```

# 隐式类型化的 lambda 表达式

没有指定任何输入参数类型的 lambda 表达式被称为**隐式类型化的 lambda 表达式**。在这种情况下，编译器推断方法参数的类型并将其添加到字节码中。

让我们修改上一节中的 lambda 表达式，去掉输入参数的类型（修改后的代码以粗体显示）：

```java
(age) -> age > 10; 
(age) -> age > 10? "Kid" : "Not a Kid"; 
age -> {System.out.println();}; 
() -> {return Math.random() + "Number";}; 

(name, list) -> { 
                    return ( 
                        list.stream() 
                            .filter(e -> e.getName().startsWith(name)) 
                            .map(Person::getAge) 
                            .findFirst() 
                    ); 
                }; 
```

你不能在 lambda 表达式中混合隐式类型化和显式类型化的参数。例如，以下代码无法编译，因为它明确指定了`x`的类型，但没有指定`y`的类型：

```java
(Integer x, y) -> x + y;                 // won't compile
```

# Lambda 参数和 var 的类型推断

在 JDK 11 中，你将能够使用 `var` 与 lambda 参数一起使用。然而，这仅仅是一种语法糖。保留类型名 `var` 在 JDK 10 中被引入，以便开发者可以在不使用显式数据类型的情况下声明局部变量（让编译器在编译期间推断数据类型）。但是，隐式类型 lambda 表达式已经通过仅使用变量名作为它们的参数（而不使用它们的类型）来实现这一点（示例包括在上一节中）。 

# 向 lambda 参数添加 var

Java 允许使用保留词 `var` 与 lambda 参数一起使用，以使其语法与局部变量的声明对齐，现在这些局部变量可以使用 `var`。

让我们修改上一节中的示例，向 lambda 参数添加 `var`：

```java
(var age) -> age > 10; 
(var age) -> age > 10? "Kid" : "Not a Kid"; 
(var age) -> {System.out.println();}; 
() -> {return Math.random() + "Number";}; 

(var name, var list) -> { 
                             return ( 
                                 list.stream() 
                                     .filter(e -> 
                                      e.getName().startsWith(name)) 
                                     .map(Person::getAge) 
                                     .findFirst() 
                                 ); 
                             }; 
```

允许将 `var` 添加到 lambda 参数中的主要原因是为了使其使用与使用 `var` 声明的局部变量的语法对齐。

如果你使用 `var` 与 lambda 参数，你必须与所有 lambda 参数一起使用它。你不能将隐式类型或显式类型参数与使用 `var` 的参数混合。以下代码示例无法编译：

```java
(var x, y) -> x + y;                         // won't compile 
(var x, Integer y) -> x + y;                 // won't compile 
```

如果你只使用一个方法参数，你不能使用圆括号（`()`）来包围 lambda 表达式的参数。但是，如果你在 lambda 参数中使用 `var`，你不能省略 `()`。以下是一些示例代码，以进一步说明这一点：

```java
(int x) -> x > 10;                         // compiles 
(x) -> x > 10;                             // compiles 
x -> x > 10;                               // compiles 
(var x) -> x > 10;                         // compiles 
var x -> x > 10;                           // Won't compile 
```

你不能将隐式类型或显式类型 lambda 参数与使用 `var` 的参数混合。

# 向 lambda 参数添加注解

如果你使用显式数据类型或使用保留类型 `var` 定义 lambda 参数，你可以使用注解。注解可以用来标记 null 或非 null 的 lambda 参数。以下是一个示例：

```java
(@Nullable var x, @Nonnull Integer y) -> x + y;
```

# 摘要

在本章中，我们介绍了使用保留类型 `var` 与隐式类型 lambda 表达式。我们首先确定了显式类型和隐式类型 lambda 表达式的语法差异。

通过示例，你看到了如何将 `var` 添加到 lambda 参数中只是语法糖，因为自从它们在 Java 8 中引入以来，你一直能够使用类型推断来使用隐式类型 lambda 参数。使用 `var` 与 lambda 参数使它们的语法与使用 `var` 声明的局部变量对齐。使用 `var` 还使开发者能够使用注解与 lambda 参数。

在下一章中，我们将处理 HTTP 客户端 API，该 API 将作为 Java 11 的核心 API 之一被添加。
