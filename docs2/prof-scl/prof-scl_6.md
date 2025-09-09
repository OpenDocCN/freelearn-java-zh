# 第六章 隐式

在上一章中，我们介绍了`类型`系统和多态性。我们还介绍了不同类型的变异性，它们提供了约束参数化类型的方法。最后，我们介绍了某些高级`类型`，如抽象类型成员、选项等。

在本章中，我们将介绍隐式参数和隐式转换。我们将学习它们是如何工作的，如何使用它们，以及它们提供的哪些好处和风险。

当你在代码中使用第三方库时，通常必须接受其代码原样。这可能会使某些库难以处理。可能是代码风格与你的代码库不同，或者库缺少某些功能，你无法优雅地提供。

一些语言已经提出了缓解这个问题的解决方案。Ruby 有模块，Smalltalk 允许包向彼此的类中添加功能，而 C# 3.0 有静态扩展方法。

Scala 有隐式参数和转换。当以受控的方式使用时，隐式参数可以使与外部库的交互更加愉快，并允许你在自己的代码中使用一些优雅的模式。

到本章结束时，你将能够：

+   描述隐式参数以及 Scala 编译器如何处理它们

+   解释隐式参数启用的设计模式

+   分析过度使用隐式参数可能出现的常见问题

# 隐式参数和隐式转换

Scala 有隐式参数和转换。当以受控的方式使用时，隐式参数可以使与外部库的交互更加愉快，并允许你在自己的代码中使用一些优雅的模式。

## 隐式参数

**隐式参数**是一种让编译器在方法调用缺少某些（或所有）隐式参数时自动填充一些参数的方法。编译器将寻找标记为隐式的所需类型的定义。例如，假设你想编写一个程序，在显示消息后提示用户进行某些操作，你希望自定义消息和提示上出现的字符串。我们可以假设提示字符串的默认值将比消息更默认，因此使用隐式参数实现它的方法之一如下：

```java
case class Prompt(value: String)
def message(msg: String)(implicit prompt: Prompt) = {
  println(msg)
  println(s"${prompt.value}>")
}
```

在之前的实现中，你可以调用消息函数，并显式地提供一个参数给提示参数：

```java
message("Welcome!")(Prompt("action"))
```

然而，如果我们想在不同的消息调用中重用提示，我们可以创建一个默认对象。

`默认值`：

```java
object Defaults {
  implicit val defaultPrompt = Prompt("action")
}
```

我们可以在使用消息方法时引入那个`默认值`，从而避免必须显式提供提示参数：

```java
import Defaults._
message("Welcome!")
message("What do you want to do next?")
```

每个方法只能有一个隐式参数列表，但它可以有多个参数。隐式参数列表必须是函数的最后一个参数列表。

隐式参数的有效参数是可以在方法调用点访问且不带前缀的标识符，它们表示隐式定义或隐式参数，以及隐式参数类型的伴生模块中的标记为隐式的成员。例如，在先前的例子中，如果你将 `defaultPrompt` 隐式放入 `Prompt` 的伴生对象中，就不需要在调用消息时导入 `Prompt` 来将 `defaultPrompt` 放入作用域。

```java
object Prompt {
  implicit val defaultPrompt = Prompt("action")
}
message("Welcome!")
message("What do you want to do next?")
```

## 隐式转换

**隐式** **转换**提供了一种在 `类型` 之间透明转换的方法。当你需要一个你无法控制的 `类型`（例如来自外部库）以符合指定接口时，隐式转换非常有用。例如，假设你想将一个整数作为可遍历的来处理，以便你可以遍历其数字。完成此操作的一种方法是通过提供隐式转换：

```java
implicit def intToIterable(i: Int): Traversable[Int] = 
  new Traversable[Int] {
  override def foreachU: Unit = {
    var value = i
    var l = List.empty[Int]
   do {
      l = value % 10 :: l
      value /= 10
    } while (value != 0)
    l.foreach(f)
  }
}
```

`intToIterable` 隐式转换就像一个普通方法一样工作。特别之处在于定义开头处的隐式关键字。你可以显式地应用转换，或者省略它，得到相同的行为：

```java
scala> intToIterable(123).size
res0: Int = 3
scala> 123.size
res1: Int = 3
scala> 123 ++ 456
res2: Traversable[Int] = List(1, 2, 3, 4, 5, 6)
```

隐式转换的最好之处在于它们支持在代码的某个位置需要的类型的转换。例如，如果你有以下函数，它从一个 `Traversable` 返回一个有序的 `Seq`：

```java
def orderedSeqA: Ordering = t.toSeq.sorted
```

### 注意

你可以将 `Int` 传递给 `orderedSeq`，因为存在从 `Int` 到 `Traversable[Int]` 的隐式转换。

```java
orderedSeq(472).toList
// Returns List(2, 4, 7)
```

当不加区分地使用隐式转换时，它们可能会很危险，因为它们可以在我们更希望编译器不编译代码的位置启用运行时错误。

应该避免在常见类型之间进行隐式转换。Scala 编译器在默认定义隐式转换时发出警告，将其标记为危险。

如前所述，隐式转换使语言能够进行类似语法的扩展。这种模式在标准库和 Scala 生态系统中的库中很常见。这种模式通常被称为“丰富包装器”，因此当你看到名为 `RichFoo` 的类时，它很可能是向 Foo 类型添加类似语法的扩展。

为了提供无分配的扩展方法，你可以使用隐式类与值类结合。例如，如果你有以下 `RichInt` 定义：

```java
implicit class RichInt(val self: Int) extends AnyVal {
  def toHexString: String = java.lang.Integer.toHexString(self)
}
```

例如，调用 `3.toHexString` 将会在一个 `static` 对象（`RichInt$.MODULE$.extension$toHexString(3)`）上调用方法，而不是在新生成的对象上调用方法。

## 隐式解析

了解编译器在哪里查找隐式转换以及它如何在看似模糊的情况下决定使用哪个隐式转换，这一点非常重要。

### 注意

有关隐式解析的更多信息，请参阅：[`docs.scala-lang.org/tutorials/FAQ/finding-implicits.html`](https://docs.scala-lang.org/tutorials/FAQ/finding-implicits.html)。

为了根据静态重载解析规则选择最具体的隐式定义，请参考：[`scala-lang.org/files/archive/spec/2.11/06-expressions.html`](http://scala-lang.org/files/archive/spec/2.11/06-expressions.html)。

隐式解析的规则有点难以记住，所以通过实验它们可以给您关于 Scala 编译器的更多直观感受。

以下列表定义了编译器查找隐式的位置：

+   当前作用域中定义的隐式

+   显式导入

+   通配符导入

+   类型类的伴随对象

+   参数类型的隐式作用域

+   类型参数的隐式作用域

+   嵌套类型的外部对象

## 活动：扩展方法的创建

在这个活动中，我们将通过依赖隐式转换来为`Int`类型创建扩展方法。

1.  首先，定义一个新的类`RichInt`，它将实现您所需的方法。

1.  创建从`Int`到`RichInt`的隐式转换。您可以选择创建一个隐式方法或一个隐式值类。由于避免运行时开销很重要，建议使用隐式值类。

1.  实现方法`square`和`plus`。

1.  确保隐式转换在作用域内，并实验调用`square`和`plus`在类型为`Int`的值上。

本节涵盖了隐式参数和隐式转换。我们看到了如何为您的代码启用优雅的扩展方法。我们还查看了一下 Scala 编译器如何解析隐式参数。

# 特设多态性和类型类

在本节中，我们将通过类型类来探讨特设多态性。

## 多态性的类型

在计算机科学中，多态性是提供单一接口以供不同类型的实体使用。多态性由三种类型组成：子类型、参数多态性和特设多态性。

子类型通过在不同的子类中具有相同方法的不同实现（但保持接口）来启用多态性。参数多态性通过允许编写不提及特定类型的代码来启用多态性。例如，当您操作泛型`List`时，您正在应用参数多态性。特设多态性通过允许根据指定的类型允许不同和异构的实现来启用多态性。方法重载是特设多态性的一个例子。

## 类型类

类型类是一种使特设多态性成为可能的构造。它们最初出现在 Haskell 中，Haskell 原生支持它们，但通过隐式参数的使用过渡到了 Scala。

在其核心，`type`类是一个带有`type`参数的类，旨在连接类型层次。也就是说，我们希望通过参数化我们的`type`类并为具体类型提供特定实现来为类型层次提供行为。类型类提供了一种在不接触现有代码的情况下扩展库的简单方法。

在本节的整个过程中，我们将考虑以下 JSON 的实现：

```java
sealed trait JsValue
case class JsObject(fields: Map[String, JsValue]) extends JsValue
case class JsArray(elements: Vector[JsValue]) extends JsValue
case class JsString(value: String) extends JsValue
case class JsNumber(value: BigDecimal) extends JsValue

sealed trait JsBoolean extends JsValue
case object JsTrue extends JsBoolean
case object JsFalse extends JsBoolean

case object JsNull extends JsValue
```

我们将引入一个名为`JsonWriter[A]`的类型类，其接口有一个单一的方法`write`，它接受一个`A`并返回一个`JsValue`。让我们定义`JsonWriter`并提供两个实现，一个用于`Int`，另一个用于`String`：

```java
trait JsonWriter[A] {
  def write(value: A): JsValue
}

object JsonWriter {
  implicit object IntJsonWriter extends JsonWriter[Int] {
    def write(value: Int): JsValue = JsNumber(value)
  }

  implicit object StringJsonWriter extends JsonWriter[String] {
    def write(value: String): JsValue = JsString(value)
  }
}
```

我们可以使用这些特定的`JsonWriter`实现将`Ints`和字符串转换为 JSON。例如，我们可以调用`IntJsonWriter.write(4)`和`StringJsonWriter.write("Hello World")`。然而，我们不想显式地调用编写器。

我们不是显式地调用`JsonWriters`，而是引入了`toJson`方法，该方法可以将类型转换为 JSON，前提是在作用域中有一个`JsonWriter`：

```java
def toJsonA(implicit jw: JsonWriter[A]) =
  jw.write(value)
```

我们现在已经在`toJson`函数中引入了特设多态。根据提供给`toJson`的值的类型，我们为`toJson`函数提供了不同的行为，这些行为由作用域内可用的`JsonWriters`控制。作用域的问题很重要。回想一下，隐式解析有优先级。因此，库的作者可以为其类型类提供自己的默认实现，但您可以在客户端代码中覆盖它，同时保持相同的接口。

## 上下文边界和隐式

**上下文边界**是一种语法糖，当您需要传递隐式值时可以减少冗余。通过使用上下文边界，您减少了隐式参数列表的需求。然而，当使用上下文边界时，您将失去调用方法时使用的隐式参数的访问权限。为了提供访问权限，您可以使用`implicitly`函数。`implicitly`提供了对作用域中请求类型的隐式的访问。它的实现很简单：

```java
def implicitlyT = e
```

## 标准库中的类型类

类型类模式在 Scala 标准库中被广泛使用。其使用的主要例子是之前引入的`Ordering`类型类和代表 Scala 集合构建器工厂的`CanBuildFrom`类型类。

### 注意

请自行查看`Ordering`和`CanBuildFrom`类型类。关于`CanBuildFrom`类型类的好概述可以从以下指南中获得：[`docs.scala-lang.org/overviews/core/architecture-of-scala-collections.html`](http://docs.scala-lang.org/overviews/core/architecture-of-scala-collections.html)。

### 活动：实现支持转换的类型类

在这个活动中，我们将实现`type`类以支持将常见 Scala 类型转换为`JsValue`。考虑本节开头引入的`JsValue` ADT。

1.  首先，如果您还没有定义，请定义`toJson`方法：

    ```java
    def toJsonA(implicit jw: JsonWriter[A]): JsValue and 
    the JsonWriter trait as trait JsonWriter[A] { def write(value: A): JsValue }
    ```

1.  为`Int`、`String`和`Boolean`实现`JsonWriter`。根据之前引入的隐式解析规则，这些实现的好地方是在`JsonWriter`的伴生对象中。

1.  实现`JsonWriter`用于`List`、`Set`和`Map`。在这些泛型集合中，请注意，如果你有一个`JsonWriter[A]`，例如，你可以提供一个`JsonWriter[List[A]]`。并非所有映射都可以转换为 JSON，因此只提供一个`JsonWriter[Map[String, A]]`。

# 摘要

在本章中，我们介绍了隐式参数和隐式转换。我们看到了如何为你的代码启用优雅的扩展方法。我们还了解到了 Scala 编译器如何解析隐式参数。最后，我们讨论了隐式参数的工作原理、如何使用它们以及它们能提供什么样的好处。

在下一章中，我们将介绍函数式编程的核心概念，如纯函数、不可变性和高阶函数。我们将在此基础上介绍一些在大规模函数式程序中普遍存在的模式，你无疑会在开始使用专注于函数式编程的 Scala 库时遇到这些模式。最后，我们将介绍两个流行的函数式编程库，即`Cats`和`Doobie`，并使用它们编写一些有趣的程序。
