# 第七章：类型类的概念

在上一章中，我们看到了函数式编程对数据表示的观点。在函数式编程中，数据最常见的形式是函数返回的结果。这个结果通常是一个包含函数结果和函数中发生的副作用数据的结构。不同的副作用用不同的数据结构表示。

我们还看到了分析和处理这些数据结构可能会变得繁琐，因此函数式编程产生了诸如 map 和`flatMap`之类的模式。还有许多更多的工作效果类型的模式。`map`和`flatMap`只是特定上下文中使用的实用方法。然而，它们足够通用，可以从一种数据类型重复到另一种数据类型。

在本章中，我们将看到函数式编程如何处理数据结构的行为。我们将看到诸如`map`和`flatMap`之类的操作如何组织成逻辑单元，并展示这些类型如何表示数据结构的行为。

我们将介绍类型类的概念，并解释其背后的推理，以便更好地理解这个模式。

在本章中，我们将涵盖以下主题：

+   丰富包装器模式

+   类型类模式

+   类型类模式的解释

+   不同语言中的类型类

# 丰富包装器模式

在本节中，我们将开始了解类型类模式。我们将从介绍*丰富包装器*模式开始。这个模式是特定于 Scala 的，但它引入了将数据与行为分离的问题，这在类型类模式中变得很重要。

# 动机

考虑以下问题。Scala 是一种建立在 JVM 之上的语言，因此它可以访问 Java 核心库，你可以在 Scala 程序中使用 Java 核心类。你还可以在你的 Scala 程序中使用任何 Java 库。

以这种方式，Scala 的 String 和 Array 数据类型来自 Java 核心。然而，如果你熟悉 Scala，你知道 String 和 Array 更像是 Scala 集合，而不是 Java 字符串和数组。它们之所以这样处理，是因为 Scala 为你提供了一组额外的方法，例如`map`、`flatMap`和`filter`，这些方法在上述类型之上。因此，当与字符串和数组一起工作时，所有普通 Scala 集合的方法也都可用。字符串被视为字符集合，数组被视为元素的索引序列。

在 Scala 中，我们如何在字符串和数组上拥有来自 Java 的集合方法？答案是 Scala 有一个机制来模拟将方法注入到类中。我们可以在 Scala 中有一个来自第三方库的类，并且能够向这个类注入额外的方法，而无需修改其原始实现，也不通过子类型扩展原始类。这种方法注入机制在将数据与其行为分离的函数式世界中非常有用。

这个解决方案被称为**Rich Wrapper**模式。要理解它，你需要了解 Scala 中隐式转换的机制。这种机制提供了一种让编译器执行通常手动完成的工作的方法。理解隐式转换的最简单方式是通过一个例子。

# 隐式转换

假设你拥有同一领域两个不同的模型。某些方法期望一个领域的领域对象，但你希望用另一个领域的领域对象来调用它们。

具体来说，想象一个 Web API，它通过 JSON 响应 HTTP 请求。你可能希望有两个版本的表示用户的对象。一个版本是这个实体的完整版本。它包含密码散列和所有其他数据。以下是完整版本，是实体的内部表示，用于后端，不打算泄露给最终用户：

```java
case class FullUser(name: String, id: Int, passwordHash: String)
```

这个对象的另一个版本应该在用户向 Web API 发起 HTTP 请求时发送给最终用户。我们不希望暴露太多信息，因此我们将返回这个对象的简短版本。这个版本不会暴露任何敏感信息：

```java
case class ShortUser(name: String, id: Int)
```

考虑到你需要从请求处理器返回一个对象。由于后端使用`FullUser`类来表示用户，我们首先需要使用转换方法将其转换为`ShortUser`：

```java
def full2short(u: FullUser): ShortUser =
  ShortUser(u.name, u.id)
```

还要考虑到以下方法必须执行，以便在响应 HTTP 请求时从请求处理器返回一个对象：

```java
def respondWith(user: ShortUser): Unit = ???
```

假设我们有一个`root`用户，并且我们需要能够在请求时返回它：

```java
val rootUser = FullUser("root", 0, "acbd18db4cc2f85cedef654fccc4a4d8")
```

从前面的代码片段中，我们可以想象一个按照以下方式定义的 HTTP 请求处理器：

```java
val handlerExplicit: PartialFunction[String, Unit] = {
  case "/root_user" => respondWith(full2short(rootUser))
}
```

你不希望在每次需要返回这个对象时都显式地执行从后端表示的转换。可能有许多上下文你希望这样做。例如，你可以在请求这些内容时将`User`实体与该用户的论坛帖子或评论关联起来。

隐式转换的概念正是为了这些情况而存在的。在 Scala 中，你可以定义一个方法如下：

```java
implicit def full2short(u: FullUser): ShortUser =
 ShortUser(u.name, u.id)
```

每当我们在一个期望`ShortUser`实例的地方使用`FullUser`实例时，编译器会自动使用作用域内的`implicit`方法将完整对象转换为短对象。这样，你可以隐式地将一个值转换为另一个值，而无需在代码中添加无关的细节。

在作用域内有隐式转换的情况下，我们可以编写如下代码：

```java
val handlerImplicit: PartialFunction[String, Unit] = {
  case "/root_user" => respondWith(rootUser)
}
```

上述代码与原始代码等价，其中转换是显式完成的。

# Rich Wrapper

隐式转换如何与我们需要将方法注入类时的示例相关？我们可以将方法注入视为一个转换问题。我们可以使用包装器模式定义一个类，该类包装目标类（即将其作为构造函数参数接受）并定义我们需要的那些方法。然后，每当我们在包装器中调用任何最初不存在的方法时，我们可以隐式地将原始类转换为包装器。

考虑以下示例。我们正在对一个字符串调用`filter`方法：

```java
println("Foo".filter(_ != 'o'))  // "F"
```

此方法不是`String`类的一个成员，因为这里的`String`类是`java.lang.String`。然而，Scala 集合有这个方法。接下来发生的事情是编译器意识到该对象没有这个方法，但它不会立即失败。相反，编译器开始寻找作用域内的隐式转换，这些转换可以将该对象转换为具有所需方法的另一个对象。这里的机制与我们将用户对象作为参数传递给方法的案例相同。关键是编译器期望一种类型，但接收到的却是另一种类型。

在我们的情况下，编译器期望具有定义了`filter`方法的类型，但接收到的`String`类型没有这个方法。因此，它会尝试将其转换为符合其期望的类型，即类中存在`filter`方法。结果是我们确实在作用域中有一个这样的方法：

```java
implicit def augmentString(x: String): StringOps
```

在 Scala 的`Predef`对象中定义了一个隐式转换，该转换将字符串转换为具有所有集合方法（包括`filter`）的 Rich Wrapper。同样的技术也用于将 Scala 集合方法注入 Java 数组中。

这种技术并不特定于 Scala，尽管其底层机制是。以某种形式或另一种形式，它存在于许多语言中。例如，在 C#中，你有隐式转换的概念，可以隐式地将一种类型转换为另一种类型。在 Haskell 中，我们有这种技术的更强大、功能性的版本。

# 类型类模式

有时，我们打算使用的效果类型事先并不知道。考虑日志记录的问题。日志记录可以记录到列表或文件中。使用 Writer 效果类型可以实现将日志记录到列表中。

Writer 是一个计算生成的结果和日志对的抽象。在其最简单形式中，Writer 可以理解为字符串列表和任意结果的配对。我们可以如下定义 Writer 效果类型：

```java
case class SimpleWriterA {
  def flatMapB: SimpleWriter[B] = {
    val wb: SimpleWriter[B] = f(value)
    SimpleWriter(log ++ wb.log, wb.value)
  }
  def mapB: SimpleWriter[B] =
   SimpleWriter(log, f(value)
}
```

注意，我们也为这种效果类型定义了熟悉的`map`和`flatMap`方法。

有几句话要说关于我们如何实现`flatMap`方法。实际上，我们使用的是 Writer 类型的简化版本。在其简化形式中，它是一个包含结果和字符串列表（即日志条目）的数据结构。

`flatMap`方法回答了如何组合具有`SimpleWriter`效果的顺序计算的问题。所以，给定两个这样的计算，一个作为另一个的延续（即前一个计算的结果参数化了它），问题是——我们如何产生该延续的结果，同时保留前一个计算的日志？

在前面的代码片段中，你可以看到`flatMap`方法是如何为`SimpleWriter`数据结构实现的。所以，首先，我们使用当前数据结构的结果作为输入来运行延续。这次运行在`SimpleWriter`的副作用下产生另一个结果，即带有计算日志的结果。之后，我们产生一个结合了第二个计算的结果和第一、第二个计算的合并日志的 Writer。

我们也可以为这种数据类型定义一个伴随对象，其中包含将任何值提升到效果类型和创建一个包含单个日志消息的空结构的方法：

```java
object SimpleWriter {
  // Wraps a value into SimpleWriter
  def pureA: SimpleWriter[A] =
    SimpleWriter(Nil, value)
  // Wraps a log message into SimpleWriter
  def log(message: String): SimpleWriter[Unit] =
    SimpleWriter(List(message), ())
}
```

使用 Writer 效果类型，我们可以从操作中记录日志如下：

```java
import SimpleWriter.log
def add(a: Double, b: Double): SimpleWriter[Double] =
for {
  _ <- log(s"Adding $a to $b")
  res = a + b
  _ <- log(s"The result of the operation is $res")
} yield res
println(add(1, 2))  // SimpleWriter(List(Adding 1.0 to 2.0, The result
of the operation is 3.0),3.0
```

使用另一个效果`IO`可以实现对文件的日志记录。`IO`类型代表输入输出效果，这意味着计算与某些外部资源交换信息。我们可以定义一个模拟的 IO 版本，它只是暂停计算，如下所示：

```java
case class IOA => A) {
  def flatMapB: IO[B] =
   IO.suspend { f(operation()).operation() }
  def mapB: IO[B] =
   IO.suspend { f(operation()) }
}
object IO {
  def suspendA: IO[A] = IO(() => op)
  def log(str: String): IO[Unit] =
   IO.suspend { println(s"Writing message to log file: $str") }
}
```

前面的定义遵循与`SimpleWriter`类型相同的模式。`log`方法实际上并不写入任何文件，而是通过输出到终端来模拟此操作。借助这种效果类型，我们可以如下使用日志记录：

```java
import IO.log
def addIO(a: Double, b: Double): IO[Double] =
for {
  _ <- log(s"Adding $a to $b")
  res = a + b
  _ <- log(s"The result of the operation is $res")
} yield res
addIO(1, 2).operation()
// Outputs:
// Writing message to log file: Adding 1.0 to 2.0
// Writing message to log file: The result of the operation is 3.0
```

如果我们事先不知道我们将要记录在哪里？如果我们有时需要将日志记录到文件，有时需要记录到列表中呢？如果我们处于不同的环境中，例如预发布、测试或生产环境，上述情况可能发生。问题是：我们如何具体进行代码的泛化，使其与效果无关？这里的问题是，前面两个代码片段仅在它们使用的日志记录效果类型上有所不同。在编程中，每当看到一种模式时，提取它是好主意。

抽象掉效果类型的一种方法如下：

```java
// Does not compile
// def add[F[_]](a: Double, b: Double): F[Double] =
//   for {
//     _ <- log(s"Adding $a to $b")
//     res = a + b
//     _ <- log(s"The result of the operation is $res")
//   } yield res
```

因此，效果类型变成了一个`F`类型参数。函数在类型级别上变得参数化了。然而，当我们尝试实现方法的主体时，我们会迅速遇到困难。前面的代码无法编译，因为编译器对`F`类型参数一无所知。我们在这种类型上调用`map`和`flatMap`方法，编译器无法知道这个类型上实现了哪些方法。

这个问题的解决方案以类型类模式的形式出现。在类型类模式之下，方法看起来如下所示：

```java
import Monad.Ops
def add[F[_]](a: Double, b: Double)(implicit M: Monad[F], L: Logging[F]): F[Double] =
for {
  _ <- L.log(s"Adding $a to $b")
  res = a + b
  _ <- L.log(s"The result of the operation is $res")
} yield res
println(addSimpleWriter)  // SimpleWriter(List(Adding 1.0 to 2.0, The result of the operation is 3.0),3.0)
println(addIO.operation())
// Outputs:
// Writing message to log file: Adding 1.0 to 2.0
// Writing message to log file: The result of the operation is 3.0
// 3.0
```

我们可以在这里使用`map`和`flatMap`方法的原因是，我们现在在方法参数列表中有一个隐式依赖。这个依赖是类型类`Monad`的依赖。`Monad`是函数式编程中最常见的类型类之一。还有一个依赖项是 Logging，它提供了`log`方法，这也是我们感兴趣的效果类型都可用的一种常见方法。

让我们看看类型类是什么，以及它们如何在`Monad`的例子上工作。

在函数的主体中，我们可以使用`map`和`flatMap`函数，编译器可以解析它们。之前，我们看到了使用隐式依赖完成的相同方法注入技巧。在那个案例中，我们有一个将目标类型转换为 Rich Wrapper 的隐式转换。在这种情况下，使用了类似的模式。然而，它更复杂。复杂性在于 Rich Wrapper 封装了具体类，但我们现在针对的是抽象类型变量，在我们的例子中是`F`。

正如 Rich Wrappers 的情况一样，`map`和`flatMap`方法通过隐式转换注入到前面的代码中。让我们看看使这种转换成为可能的方法和类：

```java
trait Monad[F[_]] {
  def pureA: F[A]
  def mapA, B(f: A => B): F[B]
  def flatMapA, B(f: A => F[B]): F[B]
}
object Monad {
  implicit class Ops[F[_], A](fa: F[A])(implicit m: Monad[F]) {
    def mapB: F[B] = m.map(fa)(f)
    def flatMapB: F[B] = m.flatMap(fa)(f)
  }
  implicit val writerMonad: Monad[SimpleWriter] = 
   new Monad[SimpleWriter] {
     def pureA: SimpleWriter[A] =
      SimpleWriter.pure(a)
    def mapA, B(f: A => B): SimpleWriter[B] =
      fa.map(f)
    def flatMapA, B(f: A => SimpleWriter[B]):
      SimpleWriter[B] = fa.flatMap(f)
  }
  implicit val ioMonad: Monad[IO] = new Monad[IO] {
    def pureA: IO[A] =
     IO.suspend(a)
    def mapA, B(f: A => B): IO[B] =
     fa.map(f)
    def flatMapA, B(f: A => IO[B]): IO[B] =
     fa.flatMap(f)
  }
}
```

在前面的代码片段中，你可以看到使所需的转换成为可能的整个代码。这段代码实现了类型类模式。让我们一步一步地看看它：

```java
trait Monad[F[_]] {
  def pureA: F[A]
  def mapA, B(f: A => B): F[B]
  def flatMapA, B(f: A => F[B]): F[B]
}
```

在前面的代码片段中，你可以看到包含所有特定效果类型必须实现的方法的特质的定义。特质的具体实现将特质的类型参数设置为类实现的具体类型。特质由所有应该支持该类型的方法的声明组成。请注意，所有方法都期望调用它们的对象。这意味着这个特质不应该由目标对象实现。相反，这个特质的实例应该是一种工具箱，为特定类型定义某些行为，而不使该类型扩展特质。

接下来，我们有这个特质的伴随对象。这个伴随对象定义了模式中也是一部分的特定方法：

```java
implicit class Ops[F[_], A](fa: F[A])(implicit m: Monad[F]) {
  def mapB: F[B] = m.map(fa)(f)
  def flatMapB: F[B] = m.flatMap(fa)(f)
}
```

首先，有一个富包装器，正如你在前面的代码中看到的那样。这个模式的工作方式与我们之前看到的字符串和数组包装器的方式相同。然而，有一个小小的不同之处。它是在一个`F[A]`抽象类型上定义的。原则上，它可以是对任何效果类型。一开始可能会觉得我们正在为每个可能的类型定义一组方法。然而，对于实现这些方法的类型有一些约束。这些约束是通过富包装器构造函数后面的隐含参数来强制执行的：

```java
implicit m: Monad[F]
```

因此，为了构建包装器，我们需要满足对前面代码片段中定义的类型类的隐含依赖。这意味着为了`F`类型能够使用富包装器模式，我们需要在作用域中隐含地有一个前面代码中定义的特质的实例。当我们说“F 类型的类型类的一个实例”时，我们指的是一个具体对象，它扩展了类型类特质，其中类型参数被设置为`F`。

例如，`Monad for Writer`实例是一个类型符合`Monad[Writer]`的对象。

所有富包装器的方法都模仿类型类，并将其委托给它。

之后，我们对某些常见的类提供了一些类型类的默认实现。例如，我们可以为我们的 Writer 和 IO 类型定义一些：

```java
implicit val writerMonad: Monad[SimpleWriter] = new Monad[SimpleWriter] {
  def pureA: SimpleWriter[A] =
   SimpleWriter.pure(a)
  def mapA, B(f: A => B): SimpleWriter[B] =
   fa.map(f)
  def flatMapA, B(f: A => SimpleWriter[B]):
   SimpleWriter[B] = fa.flatMap(f)
}
implicit val ioMonad: Monad[IO] = new Monad[IO] {
  def pureA: IO[A] =
   IO.suspend(a)
  def mapA, B(f: A => B): IO[B] =
   fa.map(f)
  def flatMapA, B(f: A => IO[B]): IO[B] =
   fa.flatMap(f)
}
```

注意，在前面示例中，我们通过将它们委托给`SimpleWrapper`和 IO 类拥有的实现来实现`map`和`flatMap`方法。这是因为我们已经在这些类中实现了这些方法。在现实世界中，通常类将不会有所需的方法。所以你将编写它们的整个实现，而不是将它们委托给类拥有的方法。

与`Monad`类似，`Logging`类型类封装了两个效果类型共有的`log`方法：

```java
trait Logging[F[_]] {
  def log(msg: String): F[Unit]
}
object Logging {
  implicit val writerLogging: Logging[SimpleWriter] =
  new Logging[SimpleWriter] {
    def log(msg: String) = SimpleWriter.log(msg)
  }
  implicit val ioLogging: Logging[IO] = new Logging[IO] {
    def log(msg: String) = IO.log(msg)
  }
}
```

它遵循与`Monad`类型类相同的模式。首先，特质声明了类型类将拥有的方法。接下来，我们有伴随对象，为我们的效果类型提供了一些默认实现。

让我们看看前面的代码是如何使日志示例能够使用`flatMap`和`map`方法的，以及这里隐含解析的机制是如何工作的。

首先，编译器看到我们正在尝试在`F`类型上调用`flatMap`方法。编译器对`F`类型一无所知——它不知道它是否有这个方法。在普通的编程语言中，在这个点上就会发生编译时错误。然而，在 Scala 中，隐式转换开始发挥作用。编译器会尝试将这个`F`类型转换为具有所需`flatMap`方法的东西。它将开始隐式查找，以找到将任意`F`类型转换为具有所需方法的隐式转换。它会找到这样的转换。这种转换将是之前讨论过的`Monad`类型类的 Rich Wrapper。编译器会看到它可以转换任何`F[A]`效果类型到一个具有所需方法的包装器。然而，它会看到除非它能向 Rich Wrapper 的构造函数提供一个对类型类的隐式依赖，否则它无法做到这一点。这个类型类`Monad`为它所实现的效应类型定义了`map`和`flatMap`方法。换句话说，只有当作用域内有类型类的实现时，效应类型才能转换为这种 Rich Wrapper。如果一个类型没有`Monad`类型类的实现，它将不会被 Monad 的 Rich Wrapper 包装，因此它将不会注入`map`和`flatMap`方法，并且将生成编译时错误。

因此，编译器会看到它可以隐式地注入所需的方法，但前提是它找到了必要的类型类的隐式实现。因此，它会尝试找到这种实现。如果你使用 Writer 或 IO 类型调用它，它将能够找到类型类的实例，因为它们是在 Monad 伴随对象内部定义的。伴随对象会搜索其伴随类的隐式实现。

在这里，我们讨论了一些特定于 Scala 的细节——“**Rich Wrapper**”模式比其他任何模式都更特定于 Scala。然而，**Type Class**模式在许多语言中都有重复出现。接下来，我们将讨论一些关于类型类的原因，以便你知道如何思考这种模式。

# **Type Class**模式的解释

由于类型类的概念非常抽象，有必要了解它是什么以及如何在实践中使用它。

# 可注入接口

考虑**Type Class**模式的一种方式是将其视为将整个接口注入现有类的方法。

在普通的命令式语言中，接口促进了多态。它们允许你统一地对待表现出相似行为的类。例如，如果你有汽车、摩托车和卡车的类，你可以定义一个`vehicle`接口，并将所有这些类视为该接口的实例。你不再关心每个类的实现细节，你只关心所有实体都能驾驶。也就是说，它们表现出所有类都典型的一种行为。接口是一种封装共同行为的方式。当面向接口编程时，你是在基于这样的假设来编写程序：程序中的一组实体表现出在本质上相同的行为，尽管在每种实现中可能有所不同。

然而，在普通的命令式语言中，例如 Java，你必须在定义时声明接口。这意味着一旦类被定义，你就无法让它实现额外的接口。这个事实使你在某些情况下与多态作斗争。例如，如果你有一堆第三方库，并且希望这个库的类实现你程序中定义的特定接口，你就无法做到这一点。

如果你看看带有日志的示例，我们会看到这个示例正是关于多态的。我们随机选择一个`F`效果类型，并基于它具有某些行为——`flatMap`和`map`——的假设来定义示例。尽管这些行为可能因效果类型而异，但它们的本质是相同的——副作用计算的顺序组合。我们唯一关心的是我们使用的效果类型支持这些方法。只要这个条件得到满足，我们就不会关心效果类型的其他细节。

这种技术在函数式编程领域特别有帮助。让我们回顾一下——`map`和`flatMap`的需求最初是如何产生的？从数学的角度来看，它们有一个理论基础。然而，从工程的角度来看，`map`和`flatMap`方法的需求非常实用。函数式程序员需要频繁地分析代码中效果类型的代码结构，以便按顺序组合纯副作用计算，这很快就会变得相当繁琐。因此，为了避免每次分析数据结构时的模板代码，我们将顺序组合的问题抽象成了`map`和`flatMap`方法。

这里的普遍模式是我们需要用功能数据结构来做各种事情。`map`和`flatMap`函数定义了如何进行计算的顺序组合。然而，我们可能想要做更多的事情。普遍的模式是，我们应该能够抽象出我们已有的常见重复操作，而我们可能事先并不知道所有我们可能想要支持的运算。这种情况使得将数据与行为分离成为必要。在现代函数式编程库中，效果类型（包含计算副作用信息的数据结构）与其行为（你可以用它们做什么）是分开的。这意味着效果类型只包含表示副作用的那些数据。每当我们需要对效果类型做些什么时，我们就可以使用之前讨论过的类型类模式将所需的行为注入其中。许多函数式库分为两部分——描述效果类型的部分，以及代表你可以对数据做什么的、其行为的类型类部分。这两部分通过特定于库所写编程语言的类型类机制统一在一起。例如，在 Scala 中，隐式转换机制为类型类模式和注入方法提供动力。Scala 编译器本身没有类型类模式的概念，但你可以使用语言提供的工具有效地表达它。

Haskell 对类型类有语言级别的支持。在 Haskell 中，数据和类型类在语言级别上是分开的。你无法在数据上定义任何行为。Haskell 在语言级别上实现了数据与行为分离的哲学。这一点在 Scala 中并不适用。在 Scala 中，你可以有普通的面向对象类，这些类可以既有数据（变量）也有行为（方法）。

# 工具箱

类型类模式的另一个有用的隐喻是，存在一些工具箱，允许你对数据进行操作。

想象你自己是一名木匠。木匠是这样一种人，他们用木头创造出各种东西。一个人如何从木头中创造出有用的东西呢？他们取来原木，然后去他们的工坊，那里有一堆可以用来加工木头的工具。他们使用锤子、锯子等等，将木头变成桌子、椅子和其他商品。如果木匠技艺高超，他们可能会区分不同种类的木材。例如，某些树木的木材坚固，而其他树木的木材柔软。同样的锯子对一种木材的加工效果比对另一种木材的效果更好。因此，木匠会为不同种类的木材准备不同类型的锯子。然而，无论木材的种类如何，木匠需要锯子来砍伐树木这一事实是恒定的。

在编程世界中，效果类型就像是木材。它们是函数式编程的原材料，你从中组合出你的程序。在原始状态下，没有工具很难处理——手动分析、组合和处理效果类型就像没有锯子和锤子很难从木材中雕刻出成品一样。

因此，存在处理效果类型的工具。类型类对于效果类型来说，就像锯子对于木材一样。它们是允许你处理原材料的工具。

同一把锯子可能不适用于不同类型的木材。同样地，不同的效果类型需要不同类型类的实现。例如，Writer 和 IO 效果类型需要分别实现`Monad`类型类。类型类的目的，即顺序组合，保持不变；不同情况下顺序组合的方式不同。这可以与锯切各种原材料的目的保持一致，即切割木材。然而，具体操作细节各不相同，因此需要为不同类型的原材料准备不同的锯子。

这就是为什么在类型类模式中，我们首先声明在特质中必须展示的行为，然后才为每个类型单独实现这种行为。

正如木匠有一个工具箱来处理原材料木材一样，函数式程序员有一个类型类来处理原材料效果类型。而且正如木匠有一个充满工具的整个车间一样，函数式程序员有充满不同目的类型类的库。我们将在下一章中介绍这些库。

# 不同语言中的类型类

原则上，类型类的想法即使在 Java 中也是存在的。例如，Java 有`Comparator`接口，它定义了如何比较两种任意类型。它定义了一个类型上的顺序关系。与集合一起使用的类型定义了它们排序的顺序。

然而，像 Java 这样的语言缺乏将此类方便地应用于类型的机制。所以，例如，当你对一个集合进行排序时，你需要显式地提供一个类型类的实例给排序方法。这与 Scala 不同，在 Scala 中，可以使用隐式转换和隐式查找，让编译器自己查找类型类的实现，从而不使代码杂乱。

在 Scala 中，编译器比 Java 中的编译器要聪明得多，部分原因是因为存在隐式解析机制。因此，当我们想要将一组特定方法注入一个类时，我们可以借助隐式转换来实现。如果在 Java 中我们需要显式提供所有类型类，那么在 Scala 中我们可以将大部分这项工作留给编译器。

在 Haskell 中，存在类似的机制来执行类型类的隐式查找。此外，Haskell 遵循数据和行为的分离。因此，通常情况下，你无法在数据上声明方法，也无法定义同时包含变量和方法的大类。这是为了强制执行纯函数式编程风格。在 Scala 中，它是一种纯函数式编程和面向对象编程的混合，你可以拥有同时包含变量和方法的大类。

谈到隐式解析机制，我们应该注意到这是一个相对高级的功能，并不是每种编程语言都有。

# 摘要

在本章中，我们介绍了类型类的概念，这是现代函数式编程的核心。我们通过首先介绍 Rich Wrapper 模式来构建这个概念，该模式有助于 Scala 中的类型类。类型类可以被理解为处理原始效果类型的工具箱。对类型类模式的另一种理解是，它是一个可注入的接口，你可以将其注入到你的类中以实现多态。最后，我们探讨了类型类在其他语言中的应用。在下一章中，我们将学习常用类型类及其组织在的库。

# 问题

1.  Scala 中的 *Rich Wrapper* 模式是什么？

1.  Scala 中 Rich Wrapper 的实现方式是什么？Scala 中的隐式转换机制是什么？

1.  解释类型类模式。

1.  类型类模式的动机是什么？

1.  强制性语言有类型类吗？
