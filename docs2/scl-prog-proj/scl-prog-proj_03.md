# 处理错误

在本章中，我们将继续在 第二章 中实现的退休计算器上进行工作，即 *开发退休计算器*。只要我们传递了正确的参数，我们的计算器就能正常工作，但如果任何参数错误，它将产生糟糕的堆栈跟踪并严重失败。我们的程序只适用于我们所说的 *快乐路径*。

编写生产软件的现实是，可能会出现各种错误场景。其中一些是可恢复的，一些必须以吸引人的方式呈现给用户，而对于一些与硬件相关的错误，我们可能需要让程序崩溃。

在本章中，我们将介绍异常处理，解释引用透明性是什么，并试图说服您异常不是处理错误的最佳方式。然后，我们将解释如何使用函数式编程结构有效地处理错误的可能性。

在每个部分中，我们将简要介绍一个新概念，然后在 Scala 工作表中使用它，以了解如何使用它。之后，我们将应用这些新知识来改进退休计算器。

在本章中，我们将探讨以下主题：

+   在必要时使用异常

+   理解引用透明性

+   使用 `Option` 表示可选值

+   使用 `Either` 顺序处理错误

+   使用 `Validated` 并行处理错误

# 设置

如果您尚未完成 第二章，*开发退休计算器*，那么您可以在 GitHub 上查看退休计算器项目。如果您还不熟悉 Git，我建议您首先阅读 [`guides.github.com/introduction/git-handbook/`](https://guides.github.com/introduction/git-handbook/) 中的文档。

要开始设置，请按照以下步骤操作：

1.  如果您还没有账户，请在 [`github.com/`](https://github.com/) 上创建一个账户。

1.  前往退休计算器项目 [`github.com/PacktPublishing/Scala-Programming-Projects`](https://github.com/PacktPublishing/Scala-Programming-Projects)。点击右上角的 Fork 将项目分叉到您的账户中。

1.  一旦项目被分叉，点击 Clone 或下载，并将 URL 复制到剪贴板。

1.  在 IntelliJ 中，转到 File | New | Project from Version Control | GitHub 并进行以下编辑：

    +   Git 仓库 URL**:** 粘贴您分叉仓库的 URL

    +   父目录**:** 选择一个位置

    +   目录名称**:** 保持 `retirement_calculator`

    +   点击 Clone

1.  项目应该在 IntelliJ 中导入。点击屏幕右下角的 git: master，然后选择 Remote branches | origin/chapter2 | Checkout as new local branch。将新分支命名为 `chapter3_yourusername` 以区分最终的解决方案，该解决方案位于 `origin/chapter3` 分支中。

1.  使用 *Ctrl* + *F9* 构建 project。一切都应该编译成功。

# 使用异常

异常是我们可以在 Scala 中使用的用于处理错误场景的机制之一。它由两个语句组成：

+   `throw exceptionObject`语句停止当前函数并将异常传递给调用者。

+   `try { myFunc() } catch { case pattern1 => recoverExpr1 }`语句会捕获`myFunc()`抛出的任何异常，如果该异常与`catch`块内部的某个模式匹配：

+   如果`myFunc`抛出异常，但没有模式与该异常匹配，则函数停止，并将异常再次传递给调用者。如果没有`try...catch`块可以在调用链中捕获该异常，则整个程序停止。

+   如果`myFunc`抛出异常，并且`pattern1`模式与该异常匹配，则`try...catch`块将返回箭头右侧的`recoverExpr1`表达式。

+   如果没有抛出异常，则`try...catch`块返回`myFunc()`返回的结果。

这种机制来自 Java，由于 Scala SDK 位于 Java SDK 之上，许多对 SDK 的函数调用可能会抛出异常。如果你熟悉 Java，Scala 的异常机制略有不同。Scala 中的异常总是*未检查的*，这意味着编译器永远不会强制你捕获异常或声明一个函数可以抛出异常。

# 抛出异常

下面的代码片段演示了如何抛出异常。你可以将其粘贴到 Scala 控制台或 Scala 工作表中：

```java
case class Person(name: String, age: Int)
case class AgeNegativeException(message: String) extends Exception(message)

def createPerson(description: String): Person = {
  val split = description.split(" ")
  val age = split(1).toInt
  if (age < 0)
    throw AgeNegativeException(s"age: $age should be > 0")
  else
    Person(split(0), age)
```

`createPerson`函数如果传入的字符串参数正确，则创建`Person`对象，如果不正确，则抛出不同类型的异常。在前面的代码中，我们还实现了自己的`AgeNegativeException`实例，如果字符串中传递的年龄是负数，则会抛出，如下面的代码所示：

```java
scala> createPerson("John 25")
res0: Person = Person(John,25)

scala> createPerson("John25")
java.lang.ArrayIndexOutOfBoundsException: 1
  at .createPerson(<console>:17)
  ... 24 elided

scala> createPerson("John -25")
AgeNegativeException: age: -25 should be > 0
  at .createPerson(<console>:19)
  ... 24 elided

scala> createPerson("John 25.3")
java.lang.NumberFormatException: For input string: "25.3"
  at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
  at java.lang.Integer.parseInt(Integer.java:580)
  at java.lang.Integer.parseInt(Integer.java:615)
  at scala.collection.immutable.StringLike.toInt(StringLike.scala:301)
  at scala.collection.immutable.StringLike.toInt$(StringLike.scala:301)
  at scala.collection.immutable.StringOps.toInt(StringOps.scala:29)
  at .createPerson(<console>:17)
  ... 24 elided
```

由于异常没有被任何`try...catch`块捕获，Scala 控制台显示了**堆栈跟踪**。堆栈跟踪显示了导致异常抛出的所有嵌套函数调用。在最后一个例子中，`createPerson`中的`val age = split(1).toInt`行调用了`scala.collection.immutable.StringOps.toInt`，它又调用了`scala.collection.immutable.StringLike.toInt$`，等等，直到最终在`Integer.java`的第 580 行`java.lang.Integer.parseInt`函数抛出了异常。

# 捕获异常

为了说明异常如何在调用栈中向上冒泡，我们将创建一个新的`averageAge`函数，该函数使用字符串描述计算`Person`实例列表的平均年龄，如下面的代码所示：

```java
def averageAge(descriptions: Vector[String]): Double = {
  val total = descriptions.map(createPerson).map(_.age).sum
  total / descriptions.length
}
```

此函数调用我们之前实现的`createPerson`函数，因此会抛出`createPerson`抛出的任何异常，因为在`averageAge`中没有`try...catch`块。

现在，我们可以在上面实现另一个函数，该函数将解析包含多个`Person`描述的输入，并以字符串形式返回摘要。如果输入无法解析，它将打印错误信息，如下面的代码所示：

```java
import scala.util.control.NonFatal
def personsSummary(personsInput: String): String = {
  val descriptions = personsInput.split("\n").toVector
  val avg = try {
    averageAge(descriptions)
  } catch {
    case e:AgeNegativeException =>
      println(s"one of the persons has a negative age: $e")
      0
    case NonFatal(e) =>
      println(s"something was wrong in the input: $e")
      0
  }
  s"${descriptions.length} persons with an average age of $avg"
}
```

在这个函数中，我们声明了一个`avg`值，如果没有抛出异常，它将获取`averageAge`的返回值。如果其中一个描述包含负年龄，我们的捕获块将打印错误信息，并将`avg`赋值为`0`。如果抛出了另一种类型的异常，并且这个异常是`NonFatal`，那么我们将打印另一条信息，并将`avg`也赋值为`0`。致命异常是无法恢复的异常，例如`OutOfMemoryException`。你可以查看`scala.util.control.NonFatal`的实现以获取更多详细信息。

以下代码展示了`personsSummary`的一些示例调用：

```java
scala> personsSummary(
  """John 25
    |Sharleen 45""".stripMargin)
res1: String = 2 persons with an average age of 35.0

scala> personsSummary(
  """John 25
    |Sharleen45""".stripMargin)
something was wrong in the input: java.lang.ArrayIndexOutOfBoundsException: 1
res2: String = 2 persons with an average age of 0.0

scala> personsSummary(
 """John -25
 |Sharleen 45""".stripMargin)
one of the persons has a negative age: $line5.$read$$iw$$iw$AgeNegativeException: age should be > 0
res3: String = 2 persons with an average age of 0.0
```

如我们所见，一旦任何描述无法解析，就会打印错误信息，并将平均年龄设置为`0`。

# 使用 finally 块

`try...catch`块可以可选地跟一个`finally {}`块。`finally`块中的代码总是被执行，即使`catch`块中没有匹配任何模式。`finally`块通常用于关闭在`try`块中访问的任何资源，例如文件或网络连接。

以下代码展示了如何使用 URL 将网页读入字符串：

```java
import java.io.IOException
import java.net.URL
import scala.annotation.tailrec

val stream = new URL("https://www.packtpub.com/").openStream()
val htmlPage: String =
  try {
    @tailrec
    def loop(builder: StringBuilder): String = {
      val i = stream.read()
      if (i != -1)
        loop(builder.append(i.toChar))
      else
        builder.toString()
    }
    loop(StringBuilder.newBuilder)
  } catch {
    case e: IOException => s"cannot read URL: $e"
  }
  finally {
    stream.close()
  }
```

`finally`块允许我们在页面读取成功与否的情况下关闭`InputStream`。这样，如果存在网络问题或线程中断，我们不会留下悬挂的打开连接。

注意，前面的代码仅用于说明目的。在实际项目中，你应该使用以下代码格式：

```java
val htmlPage2 = scala.io.Source.fromURL("https://www.packtpub.com/").mkString
```

现在你已经知道了如何使用异常，我们将定义引用透明的概念，并展示如何捕获异常可以破坏它。然后我们将探索更好的数据结构，这将使我们能够在不破坏引用透明性的情况下管理错误。

# 确保引用透明性

当一个表达式可以被其值替换而不改变程序的行为时，我们称其为**引用透明**。当一个表达式是一个函数调用时，这意味着我们可以总是用函数的返回值替换这个函数调用。在任何上下文中都能保证这一点的函数被称为**纯函数**。

纯函数就像数学函数一样——返回值只依赖于传递给函数的参数。你不需要考虑任何其他关于它被调用的上下文。

# 定义纯函数

在下面的代码中，`pureSquare`函数是纯函数：

```java
def pureSquare(x: Int): Int = x * x
val pureExpr = pureSquare(4) + pureSquare(3)
// pureExpr: Int = 25

val pureExpr2 = 16 + 9
// pureExpr2: Int = 25
```

被调用的函数`pureSquare(4)`和`pureSquare(3)`是引用透明的——当我们用函数的返回值替换它们时，程序的行为不会改变。

另一方面，以下函数是不纯的：

```java
var globalState = 1
def impure(x: Int): Int = {
  globalState = globalState + x
  globalState
}
val impureExpr = impure(3)
// impureExpr: Int = 4
val impureExpr2 = 4

```

我们不能用 `impure(3)` 的返回值来替换其调用，因为返回值会根据上下文而变化。实际上，任何具有**副作用**的函数都是不纯的。副作用可以是以下任何一种结构：

+   修改全局变量

+   向控制台打印

+   打开网络连接

+   从文件读取/写入数据

+   从数据库读取/写入数据

+   更普遍地，任何与外部世界的交互

下面是另一个不纯函数的例子：

```java
import scala.util.Random
def impureRand(): Int = Random.nextInt()
impureRand()
//res0: Int = -528134321
val impureExprRand = impureRand() + impureRand()
//impureExprRand: Int = 681209667
val impureExprRand2 = -528134321 + -528134321
```

你不能将 `impureRand()` 的结果替换为其值，因为每次调用时值都会变化。如果你这样做，程序的行为会改变。`impureRand()` 的调用不是引用透明的，因此 `impureRand` 是不纯的。实际上，`random()` 函数每次调用都会修改一个全局变量以生成一个新的随机数。我们也可以说这个函数是**非确定性的**——我们仅通过观察其参数无法预测其返回值。

我们可以重写我们的不纯函数，使其成为纯函数，如下面的代码所示：

```java
def pureRand(seed: Int): Int = new Random(seed).nextInt()
pureRand(10)
//res1: Int = -1157793070
val pureExprRand = pureRand(10) + pureRand(10)
//pureExprRand: Int = 1979381156
val pureExprRand2 = -1157793070 + -1157793070
//pureExprRand2: Int = 1979381156
```

你可以用 `pureRand(seed)` 的值替换其调用；给定相同的种子，该函数总是会返回相同的值。`pureRand` 的调用是引用透明的，因此 `pureRand` 是一个纯函数。

下面是另一个不纯函数的例子：

```java
def impurePrint(): Unit = println("Hello impure")
val impureExpr1: Unit = impurePrint()
val impureExpr2: Unit = ()
```

在第二个例子中，`impurePrint` 函数的返回值是 `Unit` 类型。在 Scala SDK 中，这种类型的值只有一个：`()` 值。如果我们用 `()` 替换对 `impurePrint()` 的调用，那么程序的行为会改变——在第一种情况下，会在控制台上打印一些内容，但在第二种情况下则不会。

# 最佳实践

引用透明性是函数式编程中的一个关键概念。如果你的程序大部分使用纯函数，那么以下操作会变得容易得多：

+   **理解程序在做什么**：你知道函数的返回值只依赖于其参数。你不必考虑在这个或那个上下文中这个或那个变量的状态。你只需看看参数。

+   **测试你的函数**：我将重申这一点——函数的返回值只依赖于其参数。测试它非常简单；你可以尝试不同的参数值，并确认返回值是你预期的。

+   **编写多线程程序**：由于纯函数的行为不依赖于全局状态，你可以在多个线程甚至不同的机器上并行执行它。返回值不会改变。随着我们今天构建的 CPU 越来越多核心，这将帮助你编写更快的程序。

然而，在程序中只使用纯函数是不可能的，因为本质上，程序必须与外界交互。它必须打印某些内容，读取一些用户输入，或将某些状态保存到数据库中。在函数式编程中，最佳实践是在大多数代码库中使用纯函数，并将不纯的副作用函数推到程序的边界。

例如，在第二章，“开发退休计算器”，我们实现了一个主要使用纯函数的退休计算器。其中一个副作用函数是`SimulatePlanApp`对象中的`println`调用，它在程序的边界上。`EquityData.fromFile`和`InflationData.fromFile`中也有其他副作用；这些函数用于读取文件。然而，资源文件在程序运行期间永远不会改变。对于给定的文件名，我们总是会得到相同的内容，并且我们可以替换所有调用中的`fromFile`的返回值，而不改变程序的行为。在这种情况下，读取文件的副作用是不可观察的，我们可以将这些文件读取函数视为纯函数。另一个副作用函数是`strMain`，因为它可以抛出异常。在本章的其余部分，我们将看到为什么抛出异常会破坏引用透明性，我们将学习如何用更好的函数式编程结构来替换它。

不纯函数满足以下两个标准：

+   它返回`单位`。

+   它不接受任何参数，但返回一个类型。由于它返回的内容无法通过其参数获得，它必须使用全局状态。

注意，纯函数可以在函数体内使用可变变量或副作用。只要这些效果对调用者来说是*不可观察的*，我们就认为该函数是纯的。在 Scala SDK 中，许多纯函数都是使用可变变量实现的，以提高性能。看看以下`TraversableOnce.foldLeft`的实现：

```java
def foldLeftB(op: (B, A) => B): B = {
  var result = z
  this foreach (x => result = op(result, x))
  result
}
```

# 展示异常如何破坏引用透明性

这可能看起来不明显，但当函数抛出异常时，它会破坏引用透明性。在本节中，我将向您展示原因。首先，创建一个新的 Scala 工作表，并输入以下定义：

```java
case class Rectangle(width: Double, height: Double)

def area(r: Rectangle): Double =
  if (r.width > 5 || r.height > 5)
    throw new IllegalArgumentException("too big")
  else
    r.width * r.height
```

然后，我们将使用以下参数调用`area`：

```java
val area1 = area(3, 2)
val area2 = area(4, 2)

val total = try {
  area1 + area2
} catch {
  case e: IllegalArgumentException => 0
}
```

我们得到`total: Double = 14.0`。在上面的代码中，`area1`和`area2`表达式是引用透明的。我们确实可以*替换*它们而不改变程序的行为。在 IntelliJ 中，选择`try`块内的`area1`变量，然后按*Ctrl* + *Alt* + *N*（内联变量），如下所示。对`area2`也做同样的操作：

```java
val total = try {
  area(3, 2) + area(4, 2)
} catch {
  case e: IllegalArgumentException => 0
}
```

`total`与之前相同。程序的行为没有改变，因此`area1`和`area2`是引用透明的。然而，让我们看看如果我们以以下方式定义`area1`会发生什么：

```java
val area1 = area(6, 2)
val area2 = area(4, 2)

val total = try {
  area1 + area2
} catch {
  case e: IllegalArgumentException => 0
}
```

在这种情况下，我们会得到`java.lang.IllegalArgumentException: too big`，因为我们的`area(...)`函数在宽度大于五时抛出异常。现在让我们看看如果我们像之前一样内联`area1`和`area2`会发生什么，如下面的代码所示：

```java
val total = try {
  area(6, 2) + area(4, 2)
} catch {
  case e: IllegalArgumentException => 0
}
```

在这种情况下，我们得到`total: Double = 0.0`。当用其值替换`area1`时，程序的行为发生了变化，因此`area1`不是引用透明的。

我们已经证明了异常处理会破坏引用透明性，因此抛出异常的函数是不纯的。这使得程序更难以理解，因为你必须考虑变量在哪里定义来了解程序将如何运行。行为将根据变量是在`try`块内还是外定义而改变。这在一个简单的例子中可能不是什么大问题，但当存在多个带有`try`块的链式函数调用，并匹配不同类型的异常时，可能会变得令人望而生畏。

当你使用异常时，另一个缺点是函数的*签名*并没有表明它可以抛出异常。当你调用一个可能抛出异常的函数时，你必须查看其实现来了解它可以抛出什么类型的异常，以及在什么情况下。如果函数调用了其他函数，问题会变得更加复杂。你可以通过添加注释或`@throws`注解来指明可能抛出的异常类型，但这些在代码重构时可能会过时。当我们调用一个函数时，我们只需要考虑其签名。签名有点像一份合同——给定这些参数，我会返回一个结果。如果你必须查看实现来了解抛出了哪些异常，这意味着合同尚未完成：有些信息被隐藏了。

我们现在知道了如何抛出和捕获异常，以及为什么我们应该谨慎使用它们。最佳实践是执行以下操作：

+   尽早捕获可恢复的异常，并使用特定的返回类型来指示失败的可能性。

+   不要捕获无法恢复的异常，例如磁盘空间不足、内存不足或其他灾难性故障。这样，每当发生此类异常时，你的程序都会崩溃，你应该在程序外部有一个手动或自动的恢复过程。

在本章的其余部分，我将向你展示如何使用`Option`、`Either`和`Validated`类来模拟失败的可能性。

# 使用 Option

Scala 的 `Option` 类型是一个 **代数数据类型**（**ADT**），它表示一个可选值。它也可以被视为可以包含一个或零个元素的 `List`。它是您在 Java、C++ 或 C# 编程时可能使用的 `null` 引用的安全替代品。

# 操作 `Option` 实例

以下是对 `Option` ADT 的简化定义：

```java
sealed trait Option[+A]
case class SomeA extends Option[A]
case object None extends Option[Nothing]
```

Scala SDK 提供了一个更精细的实现；前面的定义只是为了说明目的。这个定义意味着 `Option` 可以是以下两种类型之一：

+   `Some(value)`，表示一个可选值，其中值存在

+   `None`，表示一个可选值，其中值不存在。

在 `Option[+A]` 声明中 `A` 类型参数前面的 `+` 符号表示 `Option` 在 `A` 上是协变的。我们将在第四章 高级特性中更详细地探讨逆变。

目前，您只需知道如果 `B` 是 `A` 的子类型，那么 `Option[B]` 就是 `Option[A]` 的子类型。

此外，您可能会注意到 `None` 实际上扩展了 `Option[Nothing]` 而不是 `Option[A]`。这是因为一个案例对象不能接受类型参数。

在 Scala 中，`Nothing` 是最低类型，这意味着它是任何其他类型的子类型。

这意味着对于任何 `A`，`None` 都是 `Option[A]` 的子类型。

以下是一些使用不同类型 `Option` 的示例，您可以将它们粘贴到 Scala 工作表中：

```java
val opt0: Option[Int] = None
// opt0: Option[Int] = None

val opt1: Option[Int] = Some(1)
// opt1: Option[Int] = Some(1)

val list0 = List.empty[String]
list0.headOption
// res0: Option[String] = None
list0.lastOption
// res1: Option[String] = None

val list3 = List("Hello", "World")
list3.headOption
// res2: Option[String] = Some(Hello)
list3.lastOption
// res3: Option[String] = Some(World)
```

上述代码的解释如下：

+   前两个示例展示了我们如何定义一个可选包含 `Int` 的 `Option` 类型。

+   以下示例使用 `List` 中的 `headOption` 和 `lastOption` 方法来展示 SDK 的许多安全函数返回 `Option`。如果 `List` 为空，这些函数总是返回 `None`。请注意，SDK 还提供了等效的 *危险* `head` 和 `last` 方法。如果用空 `List` 调用这些危险方法，它们会抛出异常，如果我们没有捕获异常，这可能会使我们的程序崩溃。

SDK 的许多函数提供了等效的安全（返回 `Option`）和危险（抛出异常）函数。始终使用安全替代方案是一种最佳实践。

由于 `Option` 是一个 ADT，我们可以使用模式匹配来测试 `Option` 是 `None` 还是 `Some`，如下面的代码所示：

```java
def personDescription(name: String, db: Map[String, Int]): String =
  db.get(name) match {
    case Some(age) => s"$name is $age years old"
    case None => s"$name is not present in db"
  }

val db = Map("John" -> 25, "Rob" -> 40)
personDescription("John", db)
// res4: String = John is 25 years old
personDescription("Michael", db)
// res5: String = Michael is not present in db
```

`Map` 中的 `get(key)` 方法返回 `Option`，包含与键关联的值。如果键不在 `Map` 中，它返回 `None`。当您开始使用 `Option` 时，模式匹配是根据 `Option` 的内容触发不同行为的最自然方式。

另一种方法是使用 `map` 和 `getOrElse`，如下面的代码所示：

```java
def personDesc(name: String, db: Map[String, Int]): String = {
  val optString: Option[String] = db.get(name).map(age => s"$name is 
  $age years old")
  optString.getOrElse(s"$name is not present in db")
}
```

我们之前看到如何使用`map`转换向量的元素。这对于`Option`也是完全相同的——我们传递一个匿名函数，如果`Option`不为空，它将被调用。由于我们的匿名函数返回一个字符串，我们得到`Option[String]`。然后我们调用`getOrElse`，它提供了一个值，以防`Option`是`None`。`getOrElse`短语是一种安全提取`Option`内容的好方法。

永远不要在`Option`上使用`.get`方法——始终使用`.getOrElse`。如果`Option`是`None`，`.get`方法会抛出异常，因此它是不安全的。

# 使用 for...yield 组合转换

使用相同的`db: Map[String, Int]`短语，包含不同人的年龄，以下代码是一个返回两个人平均年龄的函数的简单实现：

```java
def averageAgeA(name1: String, name2: String, db: Map[String, Int]): Option[Double] = {
  val optOptAvg: Option[Option[Double]] =
    db.get(name1).map(age1 =>
      db.get(name2).map(age2 =>
        (age1 + age2).toDouble / 2))
  optOptAvg.flatten
}
val db = Map("John" -> 25, "Rob" -> 40)
averageAge("John", "Rob", db)
// res6: Option[Double] = Some(32.5)
averageAge("John", "Michael", db)
// res7: Option[Double] = None
```

函数返回`Option[Double]`。如果`name1`或`name2`在`db`映射中找不到，`averageAge`返回`None`。如果两个名字都找到了，它返回`Some(value)`。实现使用`map`来转换选项中包含的值。我们最终得到一个嵌套的`Option[Option[Double]]`，但我们的函数必须返回`Option[Double]`。幸运的是，我们可以使用`flatten`来移除一层嵌套。

我们成功实现了`averageAge`，但我们可以使用`flatMap`来改进它，如下面的代码所示：

```java
def averageAgeB(name1: String, name2: String, db: Map[String, Int]): Option[Double] =
  db.get(name1).flatMap(age1 =>
    db.get(name2).map(age2 =>
      (age1 + age2).toDouble / 2))
```

如其名所示，`flatMap`相当于组合`flatten`和`map`。在我们的函数中，我们将`map(...).flatten`替换为`flatMap(...)`。

到目前为止，一切顺利，但如果我们想得到三或四人的平均年龄怎么办？我们就必须嵌套多个`flatMap`实例，这不会很漂亮或易于阅读。幸运的是，Scala 提供了一个语法糖，允许我们进一步简化我们的函数，称为`for`推导式，如下面的代码所示：

```java
def averageAgeC(name1: String, name2: String, db: Map[String, Int]): Option[Double] =
  for {
    age1 <- db.get(name1)
    age2 <- db.get(name2)
  } yield (age1 + age2).toDouble / 2
```

当你编译一个`for`推导式，例如`for { ... } yield { ... }`时，Scala 编译器将其转换为`flatMap`/`map`操作的组合。以下是它是如何工作的：

+   在`for`块内部，可以有一个或多个表达式以`variable <- context`的形式表达，这被称为**生成器**。箭头的左侧是绑定到箭头右侧**上下文**内容的变量的名称。

+   除了最后一个生成器之外，每个生成器都被转换为一个`flatMap`表达式。

+   最后一个生成器被转换为一个`map`表达式。

+   所有上下文表达式（箭头的右侧）必须具有相同的上下文类型。

在前面的例子中，我们使用了`Option`作为上下文类型，但`for yield`也可以与任何具有`flatMap`和`map`操作的类一起使用。例如，我们可以使用`for..yield`与`Vector`一起运行嵌套循环，如下面的代码所示：

```java
for {
  i <- Vector("one", "two")
  j <- Vector(1, 2, 3)
} yield (i, j)
// res8: scala.collection.immutable.Vector[(String, Int)] = 
// Vector((one,1), (one,2), (one,3), (two,1), (two,2), (two,3))
```

**语法糖**是编程语言中的语法，它使得阅读或编写更容易。它让程序员感到更甜蜜。

# 将退休计算器重构为使用 Option

现在我们知道了`Option`能为我们做什么，我们将重构我们在第二章，“开发退休计算器”中开发的退休计算器的一个函数，以改进对一些边缘情况的处理。如果你还没有做，请按照第二章，“开发退休计算器”开头的说明来设置项目。

在`RetCalc.scala`中，我们将更改`nbMonthsSaving`的返回类型。在第二章，“开发退休计算器”中，如果`netIncome <= currentExpense`，我们返回`Int.MaxValue`以避免无限循环。这并不十分健壮，因为这个无限的结果可能会被用于另一个计算，从而导致错误的结果。最好是返回`Option[Int]`来表示该函数可能不可计算，并让调用者决定如何处理。如果不可计算，我们将返回`None`，如果可计算，则返回`Some(returnValue)`。

以下代码是`nbMonthsSaving`的新实现，其中更改的部分以粗体突出显示：

```java
def nbOfMonthsSaving(params: RetCalcParams, 
                     returns: Returns): Option[Int] = {
  import params._
  @tailrec
  def loop(months: Int): Int = {
    val (capitalAtRetirement, capitalAfterDeath) = 
      simulatePlan(returns, params, months)

    if (capitalAfterDeath > 0.0)
      months
    else
      loop(months + 1)
  }

  if (netIncome > currentExpenses)
    Some(loop(0))
  else
    None
}
```

现在，尝试编译项目。这个更改破坏了我们项目的许多部分，但 Scala 编译器是一个出色的助手。它将帮助我们识别需要更改的代码部分，以使我们的代码更加健壮。

第一个错误在`RetCalcSpec.scala`中，如下所示：

```java
Error:(65, 14) types Option[Int] and Int do not adhere to the type constraint selected for the === and !== operators; the missing implicit parameter is of type org.scalactic.CanEqual[Option[Int],Int]
      actual should ===(expected)
```

这个错误意味着在`actual should === (expected)`表达式中，类型不匹配：`actual`是`Option[Int]0`类型，而`expected`是`Int`类型。我们需要更改断言，如下所示：

```java
actual should ===(Some(expected))
```

你可以将相同的修复应用于第二个单元测试。对于最后一个单元测试，我们希望断言返回`None`而不是`Int.MaxValue`，如下所示：

```java
"not loop forever if I enter bad parameters" in {
  val actual = RetCalc.nbOfMonthsSaving(params.copy(netIncome = 1000), FixedReturns(0.04))
  actual should ===(None)
}
```

你现在可以编译并运行测试。它应该通过。

现在，你能够安全地模拟一个可选值。然而，有时并不总是很明显知道`None`实际上意味着什么。为什么这个函数返回`None`？是因为传递的参数错误吗？哪个参数错误？什么值才是正确的？确实很希望有一些解释与`None`一起出现，以便理解*为什么*没有值。在下一节中，我们将使用`Either`类型来实现这个目的。

# 使用`Either`

`Either`类型是一个 ADT，表示一个`Left`类型或`Right`类型的值。`Either`的一个简化定义如下：

```java
sealed trait Either[A, B]
case class LeftA, B extends Either[A, B]
case class RightA, B extends Either[A, B]
```

当你实例化一个`Right`类型时，你需要提供一个`B`类型的值，当你实例化一个`Left`类型时，你需要提供一个`A`类型的值。因此，`Either[A, B]`可以持有`A`类型的值或`B`类型的值。

以下代码展示了你可以在一个新的 Scala 工作表中输入的此类用法示例：

```java
def divide(x: Double, y: Double): Either[String, Double] =
  if (y == 0)
    Left(s"$x cannot be divided by zero")
  else
    Right(x / y)

divide(6, 3)
// res0: Either[String,Double] = Right(2.0)
divide(6, 0)
// res1: Either[String,Double] = Left(6.0 cannot be divided by zero) 
```

`divide`函数返回字符串或双精度浮点数：

+   如果函数无法计算值，它将返回一个被`Left`类型包裹的`String`错误。

+   如果函数可以计算正确的值，它将返回被`Right`类型包裹的`Double`值。

按照惯例，我们使用`Right`来表示正确或右侧的值，而使用`Left`来表示错误。

# 操作`Either`

由于`Either`是一个 ADT，我们可以使用模式匹配来决定在得到`Left`或`Right`类型时该做什么。

下面的代码是我们在*使用 Option*部分中展示的`personDescription`函数的修改版本：

```java
def getPersonAge(name: String, db: Map[String, Int]): Either[String, Int] =
  db.get(name).toRight(s"$name is not present in db")

def personDescription(name: String, db: Map[String, Int]): String =
  getPersonAge(name, db) match {
    case Right(age) => s"$name is $age years old"
    case Left(error) => error
  }

val db = Map("John" -> 25, "Rob" -> 40)
personDescription("John", db)
// res4: String = John is 25 years old
personDescription("Michael", db)
// res5: String = Michael is not present in db
```

第一个`getPersonAge`函数如果`name`参数在`db`中存在，则产生`Right(age)`。如果`name`不在`db`中，它将返回一个被`Left`类型包裹的错误信息。为此，我们使用`Option.toRight`方法。我鼓励你查看该方法的文档和实现。

`personDescription`的实现很简单——我们使用`getPersonAge`的结果进行模式匹配，并根据结果是否为`Left`或`Right`类型返回适当的`String`。

与`Option`一样，我们也可以使用`map`和`flatMap`来组合多个`Either`实例，如下面的代码所示：

```java
def averageAge(name1: String, name2: String, db: Map[String, Int]): Either[String, Double] =
  getPersonAge(name1, db).flatMap(age1 =>
    getPersonAge(name2, db).map(age2 =>
      (age1 + age2).toDouble / 2))

averageAge("John", "Rob", db)
// res4: Either[String,Double] = Right(32.5)
averageAge("John", "Michael", db)
// res5: Either[String,Double] = Left(Michael is not present in db)
```

注意函数体几乎与`Option`相同。这是因为`Either`是**右偏的**，意味着`map`和`flatMap`将`Either`的右侧进行转换。

如果你想要转换`Either`的`Left`侧，你需要调用`Either.left`方法，如下面的代码所示：

```java
getPersonAge("bob", db).left.map(err => s"The error was: $err")
// res6: scala.util.Either[String,Int] = Left(The error was: bob is not present in db)
```

由于`Either`实现了`map`和`flatMap`，我们可以重构`averageAge`以使用`for`推导式，如下面的代码所示：

```java
def averageAge2(name1: String, name2: String, db: Map[String, Int]): Either[String, Double] =
  for {
    age1 <- getPersonAge(name1, db)
    age2 <- getPersonAge(name2, db)
  } yield (age1 + age2).toDouble / 2
```

再次，代码看起来和用`Option`时一样。

# 重构退休计算器以使用`Either`

现在我们已经很好地理解了如何操作`Either`，我们将重构我们的退休计算器以利用它。

# 重构`nbOfMonthsSavings`

在前面的部分中，我们将`nbOfMonthsSavings`的返回类型更改为返回`Option[Int]`。如果`expenses`参数大于`income`，函数返回`None`。我们现在将其更改为返回被`Left`包裹的错误信息。

我们可以使用一个简单的字符串作为错误信息，但使用`Either`时的最佳实践是为所有可能的错误信息创建一个 ADT。在`src/main/scala/retcalc`中创建一个新的 Scala 类`RetCalcError`，如下面的代码所示：

```java
package retcalc

sealed abstract class RetCalcError(val message: String)

object RetCalcError {
  case class MoreExpensesThanIncome(income: Double, expenses: Double) 
  extends RetCalcError(
    s"Expenses: $expenses >=  $income. You will never be able to save 
    enough to retire !")
}
```

我们定义一个只有`message`方法的`RetCalcError`特质。此方法将在我们需要将错误信息返回给用户时产生错误信息。在`RetCalcError`对象内部，我们为每种错误信息类型定义一个 case 类。然后我们将需要返回错误的函数更改为返回`Either[RetCalcError, A]`。

与仅使用`String`相比，这种模式具有许多优点，如下面的列表所示：

+   所有的错误消息都位于一个地方。这允许你立即知道所有可能的错误消息，这些错误消息可以返回给用户。如果你的应用程序使用多种语言，你也可以添加不同的翻译。

+   由于`RetCalcError`是一个 ADT（抽象数据类型），你可以使用模式匹配从特定错误中恢复并采取行动。

+   它简化了测试。你可以测试一个函数是否返回特定类型的错误，而无需断言错误消息本身。这样，你可以在不更改任何测试的情况下更改错误消息。

现在，我们可以重新整理我们的`RetCalc.nbOfMonthsSavings`函数，使其返回`Either[RetCalcError, Int]`，如下所示：

```java
def nbOfMonthsSaving(params: RetCalcParams, 
                     returns: Returns): Either[RetCalcError, Int] = {
  import params._
  @tailrec
  def loop(months: Int): Int = {
    val (capitalAtRetirement, capitalAfterDeath) = 
      simulatePlan(returns, params, months)
    if (capitalAfterDeath > 0.0)
      months
    else
      loop(months + 1)
  }

  if (netIncome > currentExpenses)
    Right(loop(0))
  else
    Left(MoreExpensesThanIncome(netIncome, currentExpenses))
}
```

我们还必须更改相应的单元测试。ScalaTest 提供了对`Either`类型进行断言的便利扩展。为了将它们引入作用域，在`RetCalcSpec.scala`中扩展`EitherValues`，如下所示：

```java
class RetCalcSpec extends WordSpec with Matchers with TypeCheckedTripleEquals 
  with EitherValues {
```

如果你有一个`myEither`变量，其类型为`Either[A, B]`，那么`EitherValues`将允许我们使用以下方法：

+   `myEither.left.value`返回类型为`A`的左值，或者如果`myEither`是`Right`则测试失败

+   `myEither.right.value`返回类型为`B`的右值，或者如果`myEither`是`Left`则测试失败

现在我们可以更改`nbOfMonthsSaving`的单元测试，如下所示：

```java
"RetCalc.nbOfMonthsSaving" should {
  "calculate how long I need to save before I can retire" in {
    val actual = RetCalc.nbOfMonthsSaving(params, 
    FixedReturns(0.04)).right.value
    val expected = 23 * 12 + 1
    actual should ===(expected)
  }

  "not crash if the resulting nbOfMonths is very high" in {
    val actual = RetCalc.nbOfMonthsSaving(
      params = RetCalcParams(
        nbOfMonthsInRetirement = 40 * 12,
        netIncome = 3000, currentExpenses = 2999, initialCapital = 0),
      returns = FixedReturns(0.01)).right.value
    val expected = 8280
    actual should ===(expected)
  }

  "not loop forever if I enter bad parameters" in {
    val actual = RetCalc.nbOfMonthsSaving(
      params.copy(netIncome = 1000), FixedReturns(0.04)).left.value
    actual should ===(RetCalcError.MoreExpensesThanIncome(1000, 2000))
  }
}
```

运行单元测试。它应该通过。

# 重新整理 monthlyRate

在第二章，“开发退休计算器”中，我们实现了一个`Returns.monthlyRate(returns: Returns, month: Int): Double`函数，它返回给定月份的月收益率。当我们用超过`VariableReturns`实例大小的月份调用它时，我们使用模运算滚动到第一个月份。

这并不完全令人满意，因为它可以计算不切实际的模拟。假设你的`VariableReturns`实例包含从 1950 年到 2017 年的数据。当你要求 2018 年的月收益率时，`monthlyRate`会给你 1950 年的收益率。与当前的经济前景相比，五十年代的经济前景非常不同，而且 2018 年的收益率不太可能反映 1950 年的收益率。

因此，我们将更改`monthlyRate`，使其在`month`参数超出`VariableReturn`的范围时返回错误。首先，打开`RetCalcError.scala`并添加以下错误类型：

```java
case class ReturnMonthOutOfBounds(month: Int, maximum: Int) extends RetCalcError(
  s"Cannot get the return for month $month. Accepted range: 0 to $maximum")
```

接下来，我们将更改单元测试以指定我们期望它返回的函数。打开`ReturnsSpec.scala`并按以下方式更改测试：

```java
"Returns.monthlyReturn" should {
  "return a fixed rate for a FixedReturn" in {
    Returns.monthlyRate(FixedReturns(0.04), 0).right.value should ===
    (0.04 / 12)
    Returns.monthlyRate(FixedReturns(0.04), 10).right.value should ===
    (0.04 / 12)
  }

  val variableReturns = VariableReturns(
    Vector(VariableReturn("2000.01", 0.1), VariableReturn("2000.02", 
    0.2)))
  "return the nth rate for VariableReturn" in {
    Returns.monthlyRate(variableReturns, 0).right.value should ===(0.1)
    Returns.monthlyRate(variableReturns, 1).right.value should ===(0.2)
  }

  "return None if n > length" in {
    Returns.monthlyRate(variableReturns, 2).left.value should ===(
      RetCalcError.ReturnMonthOutOfBounds(2, 1))
    Returns.monthlyRate(variableReturns, 3).left.value should ===(
      RetCalcError.ReturnMonthOutOfBounds(3, 1))
  }

  "return the n+offset th rate for OffsetReturn" in {
    val returns = OffsetReturns(variableReturns, 1)
    Returns.monthlyRate(returns, 0).right.value should ===(0.2)
  }
}
```

然后，打开`Returns.scala`并按以下方式更改`monthlyRate`：

```java
def monthlyRate(returns: Returns, month: Int): Either[RetCalcError, Double] = returns match {
  case FixedReturns(r) => Right(r / 12)

  case VariableReturns(rs) =>
    if (rs.isDefinedAt(month))
      Right(rs(month).monthlyRate)
    else
      Left(RetCalcError.ReturnMonthOutOfBounds(month, rs.size - 1))

  case OffsetReturns(rs, offset) => monthlyRate(rs, month + offset)
}
```

现在尝试编译项目。由于`monthlyRate`被其他函数调用，我们将得到一些编译错误，这实际上是一件好事。我们只需修复编译错误，使我们的代码能够处理错误的可能性。每个修复都需要思考如何处理这种可能性。

另一方面，如果我们抛出异常而不是返回`Either`，那么一切都会编译，但每当月份超出范围时程序都会崩溃。要实现所需的行为会更困难，因为编译器不会帮助我们。

第一个编译错误在`RetCalc.scala`中的`futureCapital`，如下所示代码：

```java
Error:(55, 26) overloaded method value + with alternatives:
(...)
 cannot be applied to (Either[retcalc.RetCalcError,Double])
        accumulated * (1 + Returns.monthlyRate(returns, month)) + 
        monthlySavings
```

这意味着我们不能在`Either[RetCalcError, Double]`上调用`+`方法。如果`monthlyRate`返回`Left`，我们无法计算累积资本。最好的做法是在这里停止并返回错误。为此，我们需要将`futureCapital`的返回类型也改为`Either[RetCalcError, Double]`。

以下是该函数的修正版本：

```java
def futureCapital(returns: Returns, nbOfMonths: Int, netIncome: Int, currentExpenses: Int,
                  initialCapital: Double): Either[RetCalcError, Double] = {
  val monthlySavings = netIncome - currentExpenses
  (0 until nbOfMonths).foldLeft[Either[RetCalcError, Double]] (Right(initialCapital)) {
    case (accumulated, month) =>
      for {
        acc <- accumulated
        monthlyRate <- Returns.monthlyRate(returns, month)
      } yield acc * (1 + monthlyRate) + monthlySavings
  }
}
```

在第二行，我们更改了传递给`foldLeft`的初始元素。我们现在正在累积`Either[RetCalcError, Double]`。请注意，我们必须显式指定`foldLeft`的类型参数。在函数的先前版本中，当我们使用`Double`时，该类型是自动推断的。

如果我们不指定类型参数，编译器将推断它为初始元素的类型。在我们的情况下，`Right(initialCapital)`是`Right[Nothing, Double]`类型，它是`Either[RetCalcError, Double]`的子类。问题在于，在匿名函数内部，我们返回`Either[RetCalcError, Double]`，而不是`Right[Nothing, Double]`。编译器会抱怨类型不匹配。

在传递给`foldLeft`的匿名函数内部，我们使用一个`for`循环来完成以下操作：

+   如果`acc`和`monthlyRate`都是`Right`，则在`Right`中返回累积的结果

+   如果`acc`或`monthlyRate`是`Left`，则返回`Left`

注意，我们的实现不会在`monthlyRate`返回`Left`时立即停止，这有点低效。当我们得到错误时，没有必要遍历其他月份，因为这个函数应该始终返回它遇到的第一个错误。在第四章，*高级特性*中，我们将看到如何使用`foldr`进行懒计算以提前停止迭代。

再次编译项目。现在我们需要修复`simulatePlan`中的编译错误。

# 重构`simulatePlan`

由于`simulatePlan`调用了`futureCapital`，我们需要更改其实现以考虑新的返回类型，如下所示代码：

```java
def simulatePlan(returns: Returns, params: RetCalcParams, nbOfMonthsSavings: Int,
                 monthOffset: Int = 0): Either[RetCalcError, (Double, 
Double)] = {
  import params._

  for {
    capitalAtRetirement <- futureCapital(
      returns = OffsetReturns(returns, monthOffset),
      nbOfMonths = nbOfMonthsSavings, netIncome = netIncome, 
      currentExpenses = currentExpenses,
      initialCapital = initialCapital)

    capitalAfterDeath <- futureCapital(
      returns = OffsetReturns(returns, monthOffset + 
      nbOfMonthsSavings),
      nbOfMonths = nbOfMonthsInRetirement,
      netIncome = 0, currentExpenses = currentExpenses,
      initialCapital = capitalAtRetirement)
  } yield (capitalAtRetirement, capitalAfterDeath)
}
```

我们将两个对`futureCapital`的调用移到了一个`for`循环中。这样，如果这些调用中的任何一个返回错误，`simulatePlan`将返回它。如果两个调用都成功，`simulatePlan`将返回一个包含两个双精度值的元组。

编译项目。现在我们需要修复`nbOfMonthsSaving`中的编译错误，它使用了`simulatePlan`。以下代码是修复后的版本：

```java
def nbOfMonthsSaving(params: RetCalcParams, returns: Returns): Either[RetCalcError, Int] = {
  import params._
  @tailrec
  def loop(months: Int): Either[RetCalcError, Int] = {
    simulatePlan(returns, params, months) match {
      case Right((capitalAtRetirement, capitalAfterDeath)) =>
        if (capitalAfterDeath > 0.0)
          Right(months)
        else
          loop(months + 1)

      case Left(err) => Left(err)
    }
  }

  if (netIncome > currentExpenses)
    loop(0)
  else
    Left(MoreExpensesThanIncome(netIncome, currentExpenses))
}
```

我们不得不将我们的递归 `loop` 函数更改为返回 `Either[RetCalcError, Int]`。循环将在我们得到错误或 `if (capitalAfterDeath > 0.0)` 时停止。你可能想知道为什么我们没有使用 `flatMap` 而不是使用模式匹配。这确实会更简洁，但 `loop` 函数将不再尾递归，因为对循环的递归调用将位于匿名函数内部。作为一个练习，我鼓励你尝试更改代码以使用 `flatMap` 并观察尾递归编译错误。

编译项目。生产代码中的最后一个编译错误在 `SimulatePlanApp.scala`。

# 重构 SimulatePlanApp

我们 `SimulatePlanApp` 应用程序的入口点调用 `simulatePlan`。我们需要将其更改为返回可能发生的任何错误的文本。

首先，我们需要更改集成测试以添加一个新的测试用例。打开 `SimulatePlanIT.scala` 并添加以下测试用例：

```java
  "SimulatePlanApp.strMain" should {
    "simulate a retirement plan using market returns" in {...}

    "return an error when the period exceeds the returns bounds" in {
      val actualResult = SimulatePlanApp.strMain(
        Array("1952.09,2017.09", "25", "60", "3000", "2000", "10000"))
      val expectedResult = "Cannot get the return for month 780\. 
      Accepted range: 0 to 779"
      actualResult should === (expectedResult)
    }
  }
```

然后，打开 `SimulatePlanApp.scala` 并按如下方式更改 `SimulatePlanApp` 的实现：

```java
object SimulatePlanApp extends App {
  println(strMain(args))

  def strMain(args: Array[String]): String = {
    val (from +: until +: Nil) = args(0).split(",").toList
    val nbOfYearsSaving = args(1).toInt
    val nbOfYearsRetired = args(2).toInt

    val allReturns = Returns.fromEquityAndInflationData(
      equities = EquityData.fromResource("sp500.tsv"),
      inflations = InflationData.fromResource("cpi.tsv"))

    RetCalc.simulatePlan(
      returns = allReturns.fromUntil(from, until),
      params = RetCalcParams(
        nbOfMonthsInRetirement = nbOfYearsRetired * 12,
        netIncome = args(3).toInt,
        currentExpenses = args(4).toInt,
        initialCapital = args(5).toInt),
      nbOfMonthsSavings = nbOfYearsSaving * 12
    ) match {
      case Right((capitalAtRetirement, capitalAfterDeath)) =>
        s"""
           |Capital after $nbOfYearsSaving years of savings:    
            ${capitalAtRetirement.round}
           |Capital after $nbOfYearsRetired years in retirement: 
            ${capitalAfterDeath.round}
        """.stripMargin

      case Left(err) => err.message
    }
  }
}
```

我们只需对 `simulatePlan` 的结果进行模式匹配，如果结果是 `Right` 值，则返回解释计算结果的字符串；如果是 `Left` 值，则返回错误信息。

编译项目。现在所有生产代码都应该可以编译，但在单元测试中仍然有几个编译错误。作为一个练习，我鼓励您尝试修复它们。在大多数情况下，您必须使测试扩展 `EitherValues`，并在 `Either` 类上调用 `.right.value` 以获取其右侧值。一旦修复了剩余的错误，编译并运行项目的所有测试。它们都应该通过。

现在您的代码应该看起来像 Scala 基础 GitHub 项目中的 `Chapter03` 分支，除了我们将改进的 `SimulatePlanApp` 类。有关更多详细信息，请参阅 [`github.com/PacktPublishing/Scala-Programming-Projects`](https://github.com/PacktPublishing/Scala-Programming-Projects)。

# 使用 ValidatedNel

在本章中，我们看到了如何使用 `Option` 模型可选值的可能性，以及使用 `Either` 模型错误的可能性。我们展示了这些类型如何替换异常，同时保证引用透明性。

我们还看到了如何使用 `flatMap` 组合几个 `Option` 或 `Either` 类型。当我们需要按顺序检查可选值或错误时，这效果很好——调用 `function1`；如果没有错误，则调用 `function2`；如果没有错误，则调用 `function3`。如果这些函数中的任何一个返回错误，我们将返回该错误并停止调用链，如下面的代码所示：

```java
def sequentialErrorHandling(x: String): Either[MyError, String] =
  for {
    a <- function1(x)
    b <- function2(a)
    c <- function3(b)
  } yield c
```

然而，在某些情况下，我们可能希望并行调用多个函数并返回可能发生的所有错误。例如，当你输入一些个人详细信息以从在线商店购买产品时，你期望网站在你提交详细信息后突出显示所有字段的错误。在提交详细信息后告诉你说姓氏是必填项，然后在你再次提交详细信息后说你的密码太短，这将是一个糟糕的用户体验。所有字段必须同时验证，并且所有错误必须一次性返回给用户。

可以帮助我们解决此用例的数据结构是 `Validated`。不幸的是，它不是 Scala SDK 的一部分，我们必须使用一个名为 `cats` 的外部库将其引入我们的项目中。

# 添加 cats 依赖项

`cats` 库提供了函数式编程的抽象。其名称来自短语 *category theory* 的缩写。它也是对著名笑话的引用，即管理开发者就像放养猫——事实是，你实际上并没有控制权——猫做它们想做的事情。

在本章中，我们将只关注 `Validated` 和 `NonEmptyList`，但 `cats` 提供了许多更强大的抽象，我们将在本书的后续部分中探讨。

首先，编辑 `built.sbt` 并添加以下行：

```java
libraryDependencies += "org.typelevel" %% "cats-core" % "1.0.1"
scalacOptions += "-Ypartial-unification"
```

这将 `cats` 依赖项引入我们的项目，并启用了一个库所需的编译器标志（`partial unification`），以便正确推断类型。

使用 *Ctrl* + *S* 保存项目。IntelliJ 应该会提供更新项目以反映 `build` 文件中更改的选项。在 `built.sbt` 顶部单击刷新项目。

# 介绍 NonEmptyList

如其名所示，`cats.data.NonEmptyList` 类型代表一个至少包含一个元素的 `List` 实例。换句话说，它是一个不能为空的 `List` 实例。以下是一些你可以在新 Scala 工作表中重新输入的此用法示例：

```java
import cats.data.NonEmptyList

NonEmptyList(1, List(2, 3))
// res0: cats.data.NonEmptyList[Int] = NonEmptyList(1, 2, 3)
NonEmptyList.fromList(List(1, 2, 3))
// res3: Option[cats.data.NonEmptyList[Int]] = Some(NonEmptyList(1, 2, 3))
NonEmptyList.fromList(List.empty[Int])
// res4: Option[cats.data.NonEmptyList[Int]] = None
val nel = NonEmptyList.of(1, 2, 3)
// nel: cats.data.NonEmptyList[Int] = NonEmptyList(1, 2, 3)

nel.head
// res0: Int = 1
nel.tail
// res1: List[Int] = List(2, 3)
nel.map(_ + 1)
// res2: cats.data.NonEmptyList[Int] = NonEmptyList(2, 3, 4)
```

你可以使用以下方式构造 `NonEmptyList`：

+   `apply[A]`：你可以传递一个 `head` 元素和一个作为尾部的 `List`。

+   `fromList[A]`：你可以传递一个 `List`。你将得到一个 `Option[NonEmptyList[A]]`，如果 `List` 参数为空，则它将是 `None`。

+   `of[A]`：你可以传递一个 `head` 元素和一个可变长度的 `List` 参数作为尾部。这是当你知道其组成部分时构建 `NonEmptyList` 最方便的方式。

由于 `NonEmptyList` 总是包含至少一个元素，因此我们可以始终调用 `head` 方法而不会冒着抛出异常的风险。因此，没有 `headOption` 方法。你可以使用所有在 `List` 上使用的常规方法来操作 `NonEmptyList`：`map`、`tail`、`flatMap`、`filter` 和 `foldLeft` 等。

# 介绍 Validated

`cats.data.Validated[E, A]`类型与`Either[E, A]`非常相似。它是一个 ADT，表示一个`Invalid`类型或`Valid`类型的值。简化的定义如下：

```java
sealed trait Validated[+E, +A]
case class Valid+A extends Validated[Nothing, A]
case class Invalid+E extends Validated[E, Nothing]
```

我们将在第四章的协变和逆变部分看到类型参数前面`+`符号的含义，*高级特性*。不过，现在不必担心它。

与`Option`的定义类似，定义使用了逆变和`Nothing`。这样，对于任何`E`，`Valid[A]`是`Validated[E, A]`的子类型；对于任何`A`，`Invalid[E]`是`Validated[E, A]`的子类型。

与`Either`的主要区别是我们可以累积由多个`Validated`实例产生的错误。以下是一些你可以在新 Scala 工作表中重新输入的示例。我建议你取消勾选 IntelliJ 右下角的“类型感知高亮”框；否则，IntelliJ 会用红色下划线标记一些表达式，尽管它们可以正常编译：

```java
import cats.data._
import cats.data.Validated._
import cats.implicits._

val valid1: Validated[NonEmptyList[String], Int] = Valid(1)
// valid1: cats.data.Validated[cats.data.NonEmptyList[String],Int] = Valid(1)

val valid2 = 2.validNel[String]
// valid2: cats.data.ValidatedNel[String,Int] = Valid(2)

(valid1, valid2).mapN { case (i1, i2) => i1 + i2 }
// res1: cats.data.ValidatedNel[String,Int] = Valid(3)

val invalid3: ValidatedNel[String, Int] = Invalid(NonEmptyList.of("error"))
val invalid4 = "another error".invalidNel[Int]
(valid1, valid2, invalid3, invalid4).mapN { case (i1, i2, i3, i4) => i1 + i2 + i3 + i4 }
// res2: cats.data.ValidatedNel[String,Int] = Invalid(NonEmptyList(error, another error))

```

我们首先定义一个值为`1`的`Valid`值，具有`Int`类型的`Valid`参数和`NonEmptyList[String]`类型的`Invalid`参数。每个错误都将是一个`String`类型，`NonEmptyList`实例将强制我们在产生`Invalid`值时至少有一个错误。这种用法非常常见，因此`cats`在`cats.data`包中提供了一个类型别名`ValidatedNel`，如下面的代码所示：

```java
type ValidatedNel[+E, +A] = Validated[NonEmptyList[E], A]
```

回到我们的例子，在第二行，我们使用一个方便的`cats`方法`.validNel`定义了一个值为`2`的`Valid`值。在调用`validNel`时，我们必须传递错误类型，因为在这种情况下，编译器没有任何信息可以推断它。在我们的情况下，错误类型是`String`。`valid2`的结果类型是`ValidatedNel[String, Int]`，它是`Validated[NonEmptyList[String], Int]`的别名。

在第三行，我们通过将两个有效值放入一个元组中并调用`mapN`来**组合**这两个有效值。`mapN`短语接受一个`f`函数，该函数接受与元组中元素数量相同的参数。如果元组的**所有**元素都是`Valid`值，则调用`f`，其结果将被包裹在一个`Valid`值中。如果元组内部的**任何**元素是`Invalid`值，则所有`Invalid`值将被**合并**在一起并包裹在一个`Invalid`值中。

我们可以观察到，当我们组合`valid1`和`valid2`（它们都是`Valid`）时，`mapN`返回一个`Valid`值。当我们组合`valid1`、`valid2`、`invalid3`和`invalid4`时，`mapN`返回一个`Invalid`值。这个`Invalid`值包裹了一个包含`invalid3`和`invalid4`错误的`NonEmptyList`。

我们现在知道了两种表示失败可能性的机制：

+   使用`for...yield`的`Either`可以用于**顺序**验证，在遇到第一个错误时停止。

+   `Validated`与`mapN`可以用于**并行**验证，将所有错误累积在`NonEmptyList`中。

# 重构退休计算器以使用 ValidatedNel

带着这些新知识，我们已准备好进一步改进我们的退休计算器。我们将改进`SimulatePlanApp`，以便在程序传递给用户的参数中有一个或多个错误时提供更多信息。

当许多参数错误时，例如，如果用户传递了一些随机文本而不是可解析的数字，我们希望为每个错误参数报告一个错误。

# 添加单元测试

首先，我们需要更改与`SimulatePlanApp`相关的测试。打开`SimulatePlanAppIT.scala`并按照以下内容更改内容：

```java
package retcalc

import cats.data.Validated.{Invalid, Valid}
import org.scalactic.TypeCheckedTripleEquals
import org.scalatest.{Matchers, WordSpec}

class SimulatePlanAppIT extends WordSpec with Matchers with TypeCheckedTripleEquals {
  "SimulatePlanApp.strMain" should {
    "simulate a retirement plan using market returns" in {
      val actualResult = SimulatePlanApp.strMain(
        Array("1952.09,2017.09", "25", "40", "3000", "2000", "10000"))

      val expectedResult =
        s"""
           |Capital after 25 years of savings:    468925
           |Capital after 40 years in retirement: 2958842
           |""".stripMargin
      actualResult should ===(Valid(expectedResult))
    }

    "return an error when the period exceeds the returns bounds" in {
      val actualResult = SimulatePlanApp.strMain(
        Array("1952.09,2017.09", "25", "60", "3000", "2000", "10000"))
      val expectedResult = "Cannot get the return for month 780\. 
      Accepted range: 0 to 779"
      actualResult should ===(Invalid(expectedResult))
    }

    "return an usage example when the number of arguments is incorrect" 
      in {
      val result = SimulatePlanApp.strMain(
        Array("1952.09:2017.09", "25.0", "60", "3'000", "2000.0"))
      result should ===(Invalid(
        """Usage:
          |simulatePlan from,until nbOfYearsSaving nbOfYearsRetired 
          netIncome currentExpenses initialCapital
          |
          |Example:
          |simulatePlan 1952.09,2017.09 25 40 3000 2000 10000
          |""".stripMargin))
    }

    "return several errors when several arguments are invalid" in {
      val result = SimulatePlanApp.strMain(
        Array("1952.09:2017.09", "25.0", "60", "3'000", "2000.0", 
        "10000"))
      result should ===(Invalid(
        """Invalid format for fromUntil. Expected: from,until, actual: 
          1952.09:2017.09
          |Invalid number for nbOfYearsSaving: 25.0
          |Invalid number for netIncome: 3'000
          |Invalid number for currentExpenses: 2000.0""".stripMargin))
    }
  }
}
```

让我们详细看看前面的代码：

+   前两个测试变化不大——我们只是将期望更改为`Valid(expectedResult)`。我们将改变`SimulatePlanApp.strMain`的返回类型——而不是返回一个字符串，我们将将其更改为返回`Validated[String, String]`。我们期望`strMain`在所有参数正确的情况下返回一个包含结果的`Valid`值。如果某些参数不正确，它应该返回一个包含`String`的`Invalid`值，解释哪些参数不正确。

+   第三个测试是一个新测试。如果我们没有传递正确的参数数量，我们期望`strMain`返回一个包含使用示例的`Invalid`值。

+   第四个测试检查每个错误参数都会报告一个错误。

# 实现解析函数

下一步是添加新的错误类型，当某些参数错误时，这些错误将在`ValidateNel`中返回。我们需要按照以下方式更改`RetCalcError.scala`：

```java
object RetCalcError {
 type RetCalcResult[A] = ValidatedNel[RetCalcError, A]

  case class MoreExpensesThanIncome(income: Double, expenses: Double) 
    extends RetCalcError(...)

  case class ReturnMonthOutOfBounds(month: Int, maximum: Int) extends 
    RetCalcError(...)

  case class InvalidNumber(name: String, value: String) extends 
  RetCalcError(
    s"Invalid number for $name: $value")

  case class InvalidArgument(name: String, 
                             value: String, 
                             expectedFormat: String) extends 
  RetCalcError(
    s"Invalid format for $name. Expected: $expectedFormat, actual: 
    $value")
}
```

在这里，我们引入了一个`InvalidNumber`错误，当字符串无法解析为数字时将返回。另一个错误`InvalidArgument`将在参数错误时返回。我们将使用它来处理`from`和`until`参数错误（参见前面的单元测试）。此外，由于我们将使用许多类型的`ValidatedNel[RetCalcError, A]`形式，我们创建了一个类型别名`RetCalcResult`。它还将帮助 IntelliJ 自动完成`cats`库的函数。

之后，我们需要更改`SimulatePlanApp.strMain`以验证参数。为此，我们首先编写一个小的函数，该函数解析一个字符串参数以生成`Validated Int`。

理想情况下，以下所有解析函数都应该进行单元测试。我们确实在`SimulatePlanAppIT`中为它们提供了间接的测试覆盖率，但这并不充分。在测试驱动开发中，每次你需要编写一个新函数时，你应该先定义其签名，然后在其实现之前编写一个测试。不幸的是，这本书中没有足够的空间来展示你期望在生产应用程序中拥有的所有单元测试。然而，作为一个练习，我鼓励你编写它们。

我们称这个函数为`parseInt`。它接受一个参数的名称和其值，并返回`Validated Int`，如下面的代码所示：

```java
def parseInt(name: String, value: String): RetCalcResult[Int] =
  Validated
    .catchOnlyNumberFormatException
    .leftMap(_ => NonEmptyList.of(InvalidNumber(name, value)))
```

我们首先调用 `Validated.catchOnly` 方法，该方法执行一个代码块（在我们的情况下，`value.toInt`）并捕获特定类型的异常。如果代码块没有抛出任何异常，`catchOnly` 返回一个包含结果的 `Valid` 值。如果代码块抛出了作为参数传递的异常类型（在我们的情况下，`NumberFormatException`），则 `catchOnly` 返回一个包含捕获的异常的 `Invalid` 值。结果表达式类型为 `Validated[NumberFormatException, Int]`。然而，我们的 `parseInt` 函数必须返回 `RetCalcResut[Int]`，它是 `ValidatedNel[RetCalcError, Int]` 的别名。为了转换错误或左类型，我们调用 `Validated.leftMap` 方法来生成 `NonEmptyList[RetCalcError]`。

然后，我们编写另一个函数 `parseFromUntil`——该函数负责解析 `from` 和 `until` 参数。这两个参数由逗号分隔，如下所示：

```java
import cats.implicits._
def parseFromUntil(fromUntil: String): RetCalcResult[(String, String)] = {
  val array = fromUntil.split(",")
  if (array.length != 2)
    InvalidArgument(
      name = "fromUntil", value = fromUntil, 
      expectedFormat = "from,until"
    ).invalidNel
  else
    (array(0), array(1)).validNel
}
```

我们使用 `String.split` 方法创建一个 `Array[String]`。如果数组不恰好有两个元素，我们返回一个包含 `InvalidArgument` 错误的 `Invalid` 值。如果数组有两个元素，则将它们作为 `Valid` 值中的元组返回。

最后，我们编写一个 `parseParams` 函数，该函数接受一个参数数组并生成 `RetCalcResult[RetCalcParams]`。`RetCalcParams` 参数是 `RetCalc.simulatePlan` 所需的参数之一，如下所示：

```java
def parseParams(args: Array[String]): RetCalcResult[RetCalcParams] =
  (
    parseInt("nbOfYearsRetired", args(2)),
    parseInt("netIncome", args(3)),
    parseInt("currentExpenses", args(4)),
    parseInt("initialCapital", args(5))
  ).mapN { case (nbOfYearsRetired, netIncome, currentExpenses, 
    initialCapital) =>
    RetCalcParams(
      nbOfMonthsInRetirement = nbOfYearsRetired * 12,
      netIncome = netIncome,
      currentExpenses = currentExpenses,
      initialCapital = initialCapital)
  }
```

该函数假设 `args` 数组至少有六个元素。我们创建一个包含四个元素的元组，每个元素都是 `parseInt` 的结果，因此它具有 `RetCalcResult[Int]` 类型。然后，我们在 `Tuple4` 上调用 `mapN` 方法，这将累积由 `parseInt` 调用产生的任何错误。如果所有的 `parseInt` 调用都返回一个 `Valid` 值，则调用传递给 `mapN` 的匿名函数。它接受 `Tuple4 (Int, Int, Int, Int)` 并返回一个 `RetCalcParams` 实例。

# 实现 SimulatePlanApp.strSimulatePlan

为了保持 `SimulatePlanApp.strMain` 代码小且易于阅读，我们打算提取负责调用 `RetCalc.simulatePlan` 并返回一个详细描述模拟结果的易读字符串的代码。我们称这个新函数为 `strSimulatePlan`，并在以下代码中展示其用法：

```java
def strSimulatePlan(returns: Returns, nbOfYearsSaving: Int, params: RetCalcParams)
: RetCalcResult[String] = {
  RetCalc.simulatePlan(
    returns = returns,
    params = params,
    nbOfMonthsSavings = nbOfYearsSaving * 12
  ).map {
    case (capitalAtRetirement, capitalAfterDeath) =>
      val nbOfYearsInRetirement = params.nbOfMonthsInRetirement / 12
      s"""
         |Capital after $nbOfYearsSaving years of savings:    
        ${capitalAtRetirement.round}
         |Capital after $nbOfYearsInRetirement years in retirement: 
        ${capitalAfterDeath.round}
         |""".stripMargin
  }.toValidatedNel
}
```

该函数接受解析后的参数，调用 `simulatePlan`，并将结果转换为字符串。为了保持与我们的解析函数相同的类型，我们声明函数的返回类型为 `RetCalcResult[String]`。这是 `ValidatedNel[RetCalcError, String]` 的别名，但 `simulatePlan` 返回 `Either[RetCalcError, String]`。幸运的是，`cats` 提供了 `.toValidatedNel` 方法，可以轻松地将 `Either` 转换为 `ValidatedNel`。

# 重构 SimulatePlanApp.strMain

我们实现了一些用于解析整个参数数组的构建块。现在是时候重构 `SimulatePlanApp.strMain` 以调用它们了。首先，我们需要检查参数数组的大小是否正确，如下面的代码所示：

```java
def strMain(args: Array[String]): Validated[String, String] = {
  if (args.length != 6)
    """Usage:
      |simulatePlan from,until nbOfYearsSaving nbOfYearsRetired 
       netIncome currentExpenses initialCapital
      |
      |Example:
      |simulatePlan 1952.09,2017.09 25 40 3000 2000 10000
      |""".stripMargin.invalid
  else {
    val allReturns = Returns.fromEquityAndInflationData(
      equities = EquityData.fromResource("sp500.tsv"),
      inflations = InflationData.fromResource("cpi.tsv"))

    val vFromUntil = parseFromUntil(args(0))
    val vNbOfYearsSaving = parseInt("nbOfYearsSaving", args(1))
    val vParams = parseParams(args)

    (vFromUntil, vNbOfYearsSaving, vParams)
      .tupled
      .andThen { case ((from, until), nbOfYearsSaving, params) =>
        strSimulatePlan(allReturns.fromUntil(from, until), 
        nbOfYearsSaving, params)
      }
      .leftMap(nel => nel.map(_.message).toList.mkString("\n"))
  }
```

为了匹配我们在 `SimulatePlanAppIT` 集成测试中提出的断言，我们将签名更改为返回 `Validated[String, String]`。如果参数数组的大小不正确，我们返回一个 `Invalid` 值，其中包含解释我们程序正确用法的字符串。否则，当参数数组的大小正确时，我们首先声明 `allReturns` 变量，就像之前一样。

然后，我们调用我们之前实现的三个解析函数，并将它们分配给 `vFromUntil`、`vNbOfYearsSaving` 和 `vParams`。它们的类型分别是 `RetCalcResult[(String, String)]`、`RetCalcResult[Int]` 和 `RetCalcResult[RetCalcParams]`。之后，我们将这三个值放入 `Tuple3` 中，并调用 `cats` 的 `tupled` 函数，该函数将元组的三个元素组合起来产生 `RetCalcResult[((String, String), Int, RetCalcParams)]`。

到目前为止，我们有一个 `ValidatedNel` 实例，它包含调用我们之前实现的 `strSimulatePlan` 函数所需的所有参数。在这种情况下，我们需要按顺序检查错误——首先，我们验证所有参数，*然后* 调用 `strSimulatePlan`。如果我们使用了 `Either` 而不是 `ValidatedNel`，我们会使用 `flatMap` 来做这件事。幸运的是，`ValidatedNel` 提供了一个等效的方法，形式为 `andThen`。

与 `Option` 和 `Either` 不同，`ValidatedNel` 的实例没有 `flatMap` 方法，因为它不是一个 monad，而是一个 applicative functor。我们将在 第四章，*高级特性* 中解释这些术语的含义。如果你想按顺序运行验证，你需要使用 `andThen` 或将其转换为 `Either` 并使用 `flatMap`。

在调用 `.leftMap` 之前，我们有一个 `RetCaclResult[String]` 类型的表达式，它是 `Validated[NonEmptyList[RetCalcError], String]` 的别名。然而，我们的函数必须返回 `Validated[String, String]`。因此，我们使用传递给 `.leftMap` 的匿名函数将左边的 `NonEmptyList[RetCalcError]` 类型转换为字符串。

# 摘要

在本章中，我们看到了如何处理可选值以及如何以纯函数方式处理错误。你现在更有能力编写更安全的程序，这些程序不会抛出异常并意外崩溃。

如果你使用 Java 库或某些非纯函数式 Scala 库，你会注意到它们可以抛出异常。如果你不希望程序在抛出异常时崩溃，我建议你尽早将它们包装在 `Either` 或 `Validated` 中。

我们看到了如何使用`Either`来按顺序处理错误，以及如何使用`Validated`来并行处理错误。由于这两个类型非常相似，我建议你大多数时候使用`Validated`。`Validated`的实例确实可以使用`mapN`并行处理错误，但它们也可以使用`andThen`进行顺序验证。

本章在以函数式方式编写程序方面又前进了一步。在下一章中，我们将探索你将在典型的 Scala 项目中必然会遇到的其他语言特性：惰性、协变和逆变，以及隐式。

# 问题

这里有一些问题来测试你的知识：

+   你可以使用哪种类型来表示可选值？

+   你可以使用哪些类型来表示错误的可能性？

+   什么是引用透明性？

+   抛出异常是良好的实践吗？

这里有一些练习：

+   为`SimulatePlanApp`编写单元测试

+   在`RetCalc.scala`中使用`RetCalcResult`代替`Either[RetCalcError, X]`

+   将`VariableReturns.fromUntil`修改为在`monthIdFrom`或`monthIdUntil`在返回的`Vector`中找不到时返回错误

# 进一步阅读

在以下链接中，`cats`文档关于`Either`和`Validated`提供了其他使用示例，以及它们各自主题的更多详细信息：

+   [`typelevel.org/cats/datatypes/either.html`](https://typelevel.org/cats/datatypes/either.html)

+   [`typelevel.org/cats/datatypes/validated.html`](https://typelevel.org/cats/datatypes/validated.html)
