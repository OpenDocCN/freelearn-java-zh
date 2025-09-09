# 第五章 Scala 类型系统

在上一章中，我们介绍了如何处理列表，这使我们熟悉了整个集合库的一些设计原则。我们还介绍了如何泛化到序列，并覆盖了一些相关的数据结构。最后，我们还介绍了集合与单子之间的关系，以及我们如何利用这些知识在我们的代码中使用一些强大的抽象。

在本章中，我们将介绍`type`系统和多态。我们还将介绍不同类型的方差，它提供了一种约束参数化类型的方法。最后，我们将介绍一些高级`type`，如抽象类型成员、选项等。

Scala 是静态类型语言。这意味着变量的类型在编译时已知。静态类型语言的主要优势是编译器可以执行大量检查，从而增加在早期阶段捕获的简单错误数量。静态类型语言对重构也更友好，因为只要代码能够编译，程序员就可以对他们的更改感到更安全。

然而，Scala 不仅仅是静态类型。在本章中，我们将看到 Scala 的表达式类型系统如何启用并强制执行静态类型安全的抽象。类型推断的能力减少了程序员对程序进行冗余类型信息注解的工作量。本章将建立在下一章所需的基础之上，下一章我们将讨论类型类以及它们所启用的类型多态：临时多态。

到本章结束时，你将能够：

+   识别 Scala 类型层次结构

+   使用 Scala 类型系统提供的特性

+   识别 Scala 类型系统所启用的抽象

# 类型基础和多态

在本节中，我们将探讨不同类型和多态。我们将从 Scala 的统一类型系统开始，以存在类型结束。

## 统一的类型系统

Scala 有一个统一的类型系统。这意味着所有类型，包括“原始”类型，都继承自一个公共类型。`Any`是所有类型的超类型，通常被称为顶级类型，并定义了通用方法，如`equals`、`hashCode`和`toString`。`Nothing`是所有类型的子类型，通常被称为底层类型。没有值具有`Nothing`类型，因此它的常见用例是表示非终止：抛出的异常、程序退出或无限循环。

`Any`有两个直接子类：`AnyVal`和`AnyRef`。值类型由`AnyVal`表示。`AnyRef`表示引用类型。有九个不可为空的预定义值类型：`Double`、`Float`、`Long`、`Boolean`、`Unit`、`Byte`、`Char`、`Short`和`Int`。所有这些类型在其他编程语言中都很相似，除了`Unit`。`Unit`有一个实例，其声明方式为`()`。`Unit`是一个重要的返回类型，因为 Scala 中的所有函数都必须返回某些内容。所有非值类型都定义为引用类型。Scala 中的每个用户定义类型都是`AnyRef`的子类型。将`AnyRef`与 Java 运行时环境比较，`AnyRef`类似于`java.lang.Object`。

`null`是所有引用类型的子类型。它包含一个由字面量`null`标识的单个值。`null`用于与其他编程语言进行操作，但在 Scala 中不建议使用它。Scala 提供了其他更安全的`null`选项，我们将在本章后面介绍。

## 参数多态

**参数多态**是允许你为不同类型的值编写泛型代码而不失去静态类型优势的特性。

没有多态，泛型列表类型结构始终看起来像这样：

```java
scala> 2 :: 1 :: "bar" :: "foo" :: Nil
res0: List[Any] = List(2, 1, bar, foo)
```

在这些情况下必须处理`Any`类型意味着我们无法恢复关于各个成员的任何类型信息：

```java
scala> res0.head
res1: Any = 2
```

没有多态，我们就必须使用类型转换，因此会缺乏类型安全性（因为类型转换是动态的，发生在运行时）。

Scala 通过指定`type`变量来实现多态，这在你实现泛型函数时可能已经遇到过了：

```java
scala> def drop1A = l.tail
drop1: AList[A]

scala> drop1(List(1, 2, 3))
res1: List[Int] = List(2, 3)
```

## 类型推断

静态类型语言的一个常见问题是它们提供了过多的“语法开销”。Scala 通过引入类型接口来解决这个问题。在 Scala 中，类型推断是局部的，并且它一次只考虑一个表达式。

类型推断减少了大多数类型注解的需求。例如，在 Scala 中声明变量的类型是不必要的，因为编译器可以从初始化表达式中识别类型。方法返回类型也由编译器成功识别，因为它们类似于主体类型：

```java
val x = 3 + 4 * 5         // the type of x is Int
val y = x.toString()      // the type of y is String
def succ(x: Int) = x + 2  // method succ returns Int values
```

尽管编译器无法从递归方法中推断出结果类型。以下声明将无法编译：

```java
def fac(n: Int) = if (n == 0) 1 else n * fac(n - 1)
```

错误信息足以识别问题：

```java
<console>:11: error: recursive method fac needs result type
       def fac(n: Int) = if (n == 0) 1 else n * fac(n - 1)
```

## 参数化类型

**参数化类型**与 Java 中的泛型类型相同。泛型类型是参数化类型的泛型类或接口。例如：

```java
class Stack[T] {
  var elems: List[T] = Nil
  def push(x: T) { elems = x :: elems }
  def top: T = elems.head
  def pop() {elems = elems.tail }
}
```

泛型类型可以通过界限或变异性与类型检查进行交互。我们将在下一节中介绍变异性。

## 界限

Scala 允许程序员使用界限来限制多态变量。这些界限表达了子类型（`<:`）或超类型（`:>`)关系。例如，如果我们之前已经定义了以下`drop1`方法：

```java
scala> def drop1A <: AnyRef = l.tail
drop1: A <: AnyRefList[A]
```

以下代码将无法编译：

```java
scala> drop1(List(1, 2, 3))
<console>:13: error: inferred type arguments [Int] do not conform to method drop1's type parameter bounds [A <: AnyRef]
       drop1(List(1, 2, 3))
       ^
<console>:13: error: type mismatch;
 found   : List[Int]
 required: List[A]
       drop1(List(1, 2, 3))
```

## 存在类型

Scala 中的存在类型是包含一些未知部分的类型。例如：

```java
Foo[T] forSome { type T }
```

存在类型包括对已知存在但具体值我们不关心的类型成员的引用。在前面代码中，`T` 是一个我们不知道具体类型的类型，但我们知道它存在。使用存在类型，我们可以让程序的一些部分保持未知，并且仍然可以使用不同实现对这些未知部分进行类型检查。

想象你有一个以下的方法：

```java
scala> def foo(x: Array[Any]) = x.length
foo: (x: Array[Any])Int
```

如果你尝试以下操作，它将无法编译，因为 `Array[String]` 不是 `Array[Any]`（将在下一节中看到原因）：

```java
scala> val a = Array("foo", "bar", "baz")
a: Array[String] = Array(foo, bar, baz)

scala> foo(a)
<console>:14: error: type mismatch;
 found   : Array[String]
 required: Array[Any]
We can fix this by adding a type parameter:
scala> def fooT = x.length
foo: TInt
scala> foo(a)
res0: Int = 3
```

现在，`foo` 被参数化以接受任何 `T`。但现在我们必须携带这个 `type` 参数，而我们只关心 `Array` 上的方法，而不是 `Array` 包含的内容。因此，我们可以使用存在类型来解决这个问题。

```java
scala> def foo(x: Array[T] forSome { type T }) = x.length
foo: (x: Array[_])Int

scala> foo(a)
res0: Int = 3
```

这种模式很常见，因此 Scala 为我们提供了“通配符”，当我们不想命名类型变量时：

```java
scala> def foo(x: Array[_]) = x.length
foo: (x: Array[_])Int

scala> foo(a)
res0: Int = 3
```

## 活动：泛化二叉树的实现

在这个活动中，我们将泛化二叉搜索树的实现。假设你有一个整数二叉搜索树的以下定义。我们希望将我们的二叉搜索树实现从 `IntTree` 泛化到 `Tree[A]`。对代码进行必要的修改以支持新的定义，并使 `insert` 和 `search` 方法在新的定义上工作。你可能需要修改 `insert` 和 `search` 定义以提供一个泛型比较函数。我们希望使用这个新的泛型数据结构来存储访问我们网站的用户信息，这些用户被建模为 `User(username: String, country: String)` 案例类：

```java
trait IntTree
case class IntNode(value: Int, left: IntTree, right: IntTree) extends IntTree
case object IntEmpty extends IntTree
```

之前的定义支持以下方法：

```java
def insert(value: Int, tree: IntTree): IntTree =
  tree match {
    case IntEmpty => IntNode(value, IntEmpty, IntEmpty)
    case IntNode(currentValue, left, right) =>
      if (value < currentValue)
        IntNode(currentValue, insert(value, left), right)
      else
        IntNode(currentValue, left, insert(value, right))
  }

def search(value: Int, tree: IntTree): Boolean =
  tree match {
    case IntEmpty => false
    case IntNode(currentValue, left, right) =>
      value == currentValue ||
        (value < currentValue && search(value, left)) ||
       (value >= currentValue && search(value, right))
  }
```

1.  首先，将树的 ADT 从 `IntTree` 修改为 `Tree[A]`。对 `IntNode`（变为 `Node[A]`）和 `IntEmpty`（变为 `Empty`）进行必要的修改。

    ### 注意

    注意，`IntEmpty` 是一个对象，因此 `IntEmpty` 类型只有一个实例。`Empty` 应该是哪种类型的子类型？现在，将 `Empty` 转换为案例类：`case class Empty[A]() extends Tree[A]`。我们稍后会看看定义此类型更好的方法。

1.  修改 `insert` 定义以接受一个额外的比较函数作为函数参数：

    ```java
    insertA => Boolean).
    ```

1.  相应地修改代码以考虑新的 `comp` 参数。

1.  修改 `search` 定义以接受一个额外的 `comparison` 函数作为函数参数：

    ```java
    searchA => Boolean)
    ```

1.  相应地修改代码以考虑新的 `comp` 参数。

1.  创建一个 `User` 的比较函数，并使用它来填充 `Tree[User]`。

1.  实现 `def usersOfCountry(country: String, tree: Tree[User]): Int` 函数，该函数返回给定国家在 `Tree[User]` 中的用户数量。

在本节中，我们介绍了 Scala 的统一类型系统以及 Scala 如何实现多态。我们还介绍了类型推断及其应用的基本规则。界限也被引入作为一种方便地限制多态类型的方法。

# 协变

协变提供了一种约束参数化类型的方法。它根据其组件类型的子类型关系定义了参数化类型之间的子类型关系。

想象一下，你有一个以下类层次结构：

```java
class Tool
class HandTool extends Tool
class PowerTool extends Tool
class Hammer extends HandTool
class Screwdriver extends HandTool
class Driller extends PowerTool
If we define a generic box:
trait Box[T] {
  def get: T
}
```

工具箱中的`Box`如何相互关联？Scala 提供了三种方式：

+   协变：`Box[Hammer] <: Box[Tool]`当且仅当`Hammer <: Tool`

+   逆变：`Box[Tool] <: Box[Hammer]`当且仅当`Tool <: Hammer`

+   不变：无论`Tool`和`Hammer`的子类型关系如何，`Box[Tool]`和`Box[Hammer]`之间没有子类型关系

## 协变

假设我们想要定义一个名为`isSuitable`的函数，它接受一个`Box[HandTool]`并测试该盒子是否适合容纳它试图装箱的工具：

```java
def isSuitable(box: Box[HandTool]) = ???
```

你能传递一个锤子箱给函数吗？毕竟，锤子是一种`HandTool`，所以如果函数想要根据底层工具确定盒子的适用性，它应该接受`Box[Hammer]`。然而，如果你按原样运行代码，你会得到一个编译错误：

```java
<console>:14: error: type mismatch;
 found   : Box[Hammer]
 required: Box[HandTool]
```

这里的问题是`Box[Hammer]`不是`Box[HandTool]`的子类型，尽管`Hammer`是`HandTool`的子类型。在这种情况下，我们希望如果`B`是`A`的子类型，则`Box[B]`是`Box[A]`的子类型。这就是协变。然后我们可以告诉 Scala 编译器`Box[A]`在`A`上是协变的，如下所示：

```java
trait Box[+T] {
  def get: T
}
```

## 逆变

现在，假设我们有一些专门针对特定工具的操作符，所以你会有类似以下的内容：

```java
trait Operator[A] {
  def operate(t: A)
}
```

你有一个需要操作符能够处理锤子的问题：

```java
def fix(operator: Operator[Hammer]) = ???
```

你能传递一个`HandTool`的操作符来解决这个问题吗？毕竟，锤子是一种`HandTool`，所以如果操作符能够与手工具一起工作，它们也应该能够与锤子一起工作。

然而，如果你尝试运行代码，你会得到一个编译错误：

```java
<console>:14: error: type mismatch;
 found   : Operator[HandTool]
 required: Operator[Hammer]
```

这里的问题是`Operator[HandTool]`不是`Operator[Hammer]`的子类型，尽管`Hammer`是`HandTool`的子类型。在这种情况下，我们希望如果`B`是`A`的子类型，则`Operator[A]`是`Operator[B]`的子类型。这就是逆变。我们可以告诉 Scala 编译器`Operator[A]`在`A`上是逆变的，如下所示：

```java
trait Operator[-A] {
  def operate(t: A)
}
```

## 不变

默认情况下，类型参数是不变的，因为编译器无法猜测你打算用给定的类型来建模什么。另一方面，编译器通过禁止定义可能不合理的类型来帮助你。例如，如果你将`Operator`类声明为协变的，你会得到一个编译错误：

```java
scala> trait Operator[+A] { def operate(t: A) }
<console>:11: error: covariant type A occurs in contravariant position in type A of value t
       trait Operator[+A] { def operate(t: A) }
```

通过将`Operator`定义为协变的，你会说`Operator[Hammer]`可以用作`Operator[HandTool]`的替代。所以，只能使用锤子的操作符能够操作任何`HandTool`。

观察到`Box[+A]`和`Operator[-A]`的定义，注意到类型`A`只出现在`Box[+A]`的方法的返回类型中，并且只出现在`Operator[-A]`的方法的参数中。因此，只产生类型`A`值的类型可以在`A`上变得协变，而消耗类型`A`值的类型可以在`A`上变得逆变。

你可以通过前面的点推断出可变数据类型必然是不变的（它们有`getters`和`setters`，因此它们都产生和消费值）。

实际上，Java 在这方面存在问题，因为 Java 数组是协变的。这意味着一些在编译时有效的代码可能在运行时失败。例如：

```java
String[] strings = new String[1];
Object[] objects = strings;
objects[0] = new Integer(1); // RUN-TIME FAILURE
```

在 Scala 中，大多数集合都是协变的（例如，`List[+A]`）。然而，你可能想知道`::`方法和类似方法是如何实现的，因为它们可能在逆变位置有一个类型：

```java
trait List[+A] {
  def ::(a: A): List[A]
}
```

实际上，像`::`这样的方法是这样实现的：

```java
def ::B >: A: List[B]
```

这实际上允许集合始终在它们能够的更具体类型上参数化。注意以下列表是如何提升到`HandTool`列表的：

```java
scala> val l = List(new Hammer {}, new Hammer {}, new Hammer {})
l: List[Hammer] = List($anon$1@79dd6dfe, $anon$2@2f478dcf, $anon$3@3b88adb0)

scala> val l2 = new Screwdriver {} :: l
l2: List[HandTool] = List($anon$1@7065daac, $anon$1@79dd6dfe, $anon$2@2f478dcf, $anon$3@3b88adb0)
```

## 活动：实现协变和工具数据库

在这个活动中，我们将使我们的`Tree[A]`之前的实现对`A`进行协变。我们还希望开始为我们定义的工具建立一个数据库。我们已经扩展了工具的定义，现在它们具有重量和价格：

```java
trait Tool {
  def weight: Long
  def price: Long
}

trait HandTool extends Tool
trait PowerTool extends Tool
case class Hammer(weight: Long, price: Long) extends HandTool
case class Screwdriver(weight: Long, price: Long) extends HandTool
case class Driller(weight: Long, price: Long) extends PowerTool
```

1.  首先定义`Tree`为`Tree[+A]`。你现在可以定义`Empty`为一个扩展`Tree[Nothing]`的案例对象。

1.  定义一些工具的比较函数。例如，你可以按重量、按价格或两者的组合来比较工具。在创建树时，尝试不同的比较函数。

1.  实现一个`def mergeA => Boolean): Tree[A]`函数，该函数将两个树合并为一个。

在本节中，我们介绍了方差作为在类型上根据其组件类型定义子类型关系的方法。

# 高级类型

如果你来自 Java，这些事情可能不会让你感到惊讶。因此，让我们看看 Scala 类型系统的其他一些特性。

## 抽象类型成员

抽象类型成员是对象或类中留下的抽象类型成员。它们可以在不使用类型参数的情况下提供一些抽象。如果一个类型在大多数情况下打算存在性地使用，我们可以通过使用类型成员而不是参数来减少冗余。

```java
class Operator {
  type ToolOfChoice
}

class Susan extends Operator {
  type ToolOfChoice = Hammer
}

class Operator[ToolOfChoice]
class Susan extends Operator[ToolOfChoice]
```

你可以使用哈希运算符来引用抽象类型变量。

```java
scala> val tool: Susan#ToolOfChoice = new Hammer
tool: Hammer = Hammer@d8756ac
```

## 结构化类型

Scala 支持结构化类型：类型要求是通过接口结构而不是具体类型来表达的。结构化类型提供了一种类似于动态语言在支持鸭子类型时允许你做的事情的功能，但在静态类型实现中，这些功能在编译时进行检查。然而，请注意，Scala 使用反射在结构化类型上调用方法，这会对性能产生成本：

```java
def quacker(duck: { def quack(value: String): String }) {
  println(duck.quack("Quack"))
}

object BigDuck {
  def quack(value: String) = value.toUpperCase
}

object SmallDuck {
  def quack(value: String) = value.toLowerCase
}
…
…
 required: AnyRef{def quack(value: String): String}
       quacker(NotADuck)
```

结构化类型在 Scala 代码库中并不常见。

## Option

我们之前在 Scala 层次结构中讨论了 `Null` 类型，但评论说在 Scala 代码中很少看到 `null`。背后的原因是 Scala 标准库中存在 `Option` 类型。如果你以前使用过 Java，那么你可能在某个时候遇到过 `NullPointerException`。这通常发生在某些方法返回 `null` 时，而程序员没有预料到这一点，也没有在客户端代码中处理这种情况。Scala 通过通过 `Option[A]` 特质使可选类型显式化来尝试解决这个问题。`Option[A]` 是类型为 `A` 的可选值的容器。如果值存在，则 `Option[A]` 是 `Some[A]` 的实例，否则它是 `None` 对象。通过在类型级别上使可选值显式化，就不可能意外地依赖于实际上可选的值的存在。

你可以使用 `Some` 情况类或通过分配 `None` 对象来创建一个 `Option`。当与 Java 库一起工作时，你可以使用 `Option` 伴生对象的工厂方法，如果给定的参数为 `null`，则创建 `None`，否则将参数包装在 `Some` 中：

```java
val intOption1: Option[Int] = Some(2)
val intOption2: Option[Int] = None
val strOption: Option[String] = Option(null)
```

`Option` 特质定义了一个 `get` 方法，在 `Some` 的情况下返回包装的值，在 `None` 的情况下抛出 `NoSuchElementException`。一个更安全的方法是 `getOrElse`，在 `Some` 的情况下返回包装的值，但在 `None` 的情况下返回默认值。请注意，`getOrElse` 方法中的默认值是一个按名传递的参数，因此它只会在 `None` 的情况下进行评估。

使用模式匹配是处理 `Option` 的便捷方式：

```java
def foo(v: Option[Int]) = v match {
  case Some(value) => println(s"I have a value and it's $value.")
  case None => println("I have no value.")
}
```

`Option` 的一个优点是它扩展了 `Traversable`，因此你拥有了我们在上一章中提到的所有 `map`、`flatMap`、`fold`、`reduce` 以及其他方法。

## 高阶类型

Scala 可以抽象化高阶类型。你可以将其视为类型的类型。它的一个常见用例是，如果你想要抽象化多个类型的容器，这些容器用于存储多种类型的数据。你可能想要为这些容器定义一个接口，而不必确定值的类型：

```java
trait Container[M[_]] {
  def putA: M[A]
  def getA: A
}

val listContainer = new Container[List] {
  def putA = List(x)
  def getA = m.head
}

scala> listContainer.put("str")
res0: List[String] = List(str)

scala> listContainer.put(123)
res1: List[Int] = List(123)
```

## 类型擦除

为了不产生运行时开销，Java 虚拟机执行类型擦除。在其他方面，类型擦除将泛型类型中的所有类型参数替换为其边界或 `Object`（如果类型参数未指定边界）。这导致字节码只包含普通类、接口和方法，并确保不会为参数化类型创建新类。这导致我们在尝试对泛型类型参数进行匹配时会出现一些陷阱：

```java
def optMatchA = opt match {
  case opt: Option[Int] => println(s"Got Option[Int]: $opt.")
  case opt: Option[String] => println(s"Got Option[String]: $opt.")
  case other => println(s"Got something else: $other.")
}

scala> optMatch(Some(123))
Got Option[Int]: Some(123).

scala> optMatch(Some("str"))
Got Option[Int]: Some(str).
```

因此，你应该始终避免对泛型类型参数进行匹配。如果无法重构执行模式匹配的方法，尝试通过将具有类型参数的输入装箱并指定类型参数的容器来控制传递给函数的值的类型：

```java
case class IntOption(v: Option[Int])
case class StringOption(v: Option[String])
```

## 活动：基于给定谓词查找元素

在这个活动中，我们希望为我们的`Tree`提供基于给定谓词查找元素的功能。更具体地说，我们希望实现`def findA: Option[A]`函数。如果找不到满足谓词的元素，则该函数应返回`None`，或者返回满足谓词的第一个元素（按顺序）。

1.  我们希望按顺序返回第一个元素，因此我们需要假设该树是一个搜索树，并按顺序遍历它。实现`def inOrderA: Iterator[A]`方法，该方法返回一个包含`Tree`中元素顺序遍历的`Iterator`。

1.  使用之前实现的方法，现在依靠`Iterator`的`find`方法来实现`target`函数。

1.  我们希望找到重量低于 100 的最便宜的工具。实现创建树时应使用的函数，以及`find`方法中应使用的谓词。

# 摘要

在本章中，我们介绍了`type`系统和多态。我们还介绍了不同类型的变异性，它提供了一种约束参数化类型的方法。最后，我们介绍了某些高级`type`，例如抽象类型成员、选项等。

在下一章中，我们将介绍`implicits`，这将使使用外部库的工作更加愉快。我们将介绍隐式转换，并最终通过使用类型类来介绍特设多态。
