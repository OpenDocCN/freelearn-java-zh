# 第六章：实践中的效果类型

在前面的章节中，我们看到了抽象副作用的一般模式是使用效果类型。这种模式允许你减轻心理负担。该模式指出，我们首先定义一个效果类型，然后使用此类型表示特定副作用的所有发生。在本章中，我们将看到更多关于现实世界效果类型的示例以及何时使用它们。

更精确地说，我们将涵盖以下主题：

+   `Future`

+   `Either`

+   `Reader`

# 未来

我们将要查看的第一个效果类型是未来。这种效果在广泛的项目中经常遇到，甚至在非功能语言中也是如此。如果你在 Java 中有关编写并发和异步应用程序的丰富经验，你可能已经了解这种类型的效果。

首先，让我们看看效果类型抽象的现象以及为什么可能需要这种效果类型的动机。

# 动机和命令式示例

考虑以下示例。假设你正在开发一个日历应用程序来编写用户的日常日程。此应用程序允许用户将未来的计划写入数据库。例如，如果他们与某人有一个会议，他们可以在数据库中创建一个单独的条目，指定何时以及在哪里举行。

他们还可能希望将天气预报集成到应用程序中。他们希望在用户有户外活动且天气条件不利时提醒他们。例如，在雨天举办户外野餐派对是不受欢迎的。帮助用户避免这种情况的一种方法是通过使应用程序联系天气预报服务器，看看给定日期的天气是否令人满意。

对于任何给定的事件，这个过程可以用以下算法来完成：

1.  根据事件的 ID 从数据库中检索事件

1.  检索事件的时间和地点

1.  联系天气预报服务器，并给它提供我们感兴趣的日期和地点，然后检索天气预报

1.  如果天气不好，我们可以向用户发送通知

上述算法可以如下实现：

```java
def weatherImperative(eventId: Int): Unit = {
  val evt = getEvent(eventId)  // Will block
  val weather = getWeather(evt.time, evt.location)  // Will block
  if (weather == "bad") notifyUser() // Will block
}
```

方法定义如下：

```java
case class Event(time: Long, location: String)
def getEvent(id: Int): Event = {
  Thread.sleep(1000)  // Simulate delay
  Event(System.currentTimeMillis, "New York")
}
def getWeather(time: Long, location: String): String = {
  Thread.sleep(1000) // Simulate delay
  "bad"
}
def notifyUser(): Unit = Thread.sleep(1000) // Simulate delay
```

在前面的例子中，有一个可能引起麻烦的效果。连接到数据库需要时间，而联系天气服务器则需要更多的时间。

如果我们像前一个示例那样从应用程序的主线程顺序执行所有这些操作，我们就有阻塞这个线程的风险。阻塞主应用程序线程意味着应用程序将变得无响应。避免这种体验的一种标准方法是在单独的线程中运行所有这些耗时计算。然而，在异步应用程序中，通常以非阻塞方式指定每个计算。阻塞方法并不常见；相反，每个方法都应该立即返回一个表示计算的异步原语。

在 Java 中，这种想法的最简单实现是在单独的线程中运行每个计算：

```java
// Business logic methods
def notifyThread(weather: String): Thread = thread {
  if (weather == "bad") notifyUser()
}
def weatherThread(evt: Event): Thread = thread {
  val weather = getWeather(evt.time, evt.location)
  runThread(notifyThread(weather))
}
val eventThread: Thread = thread {
  val evt = getEvent(eventId)
  runThread(weatherThread(evt))
}
```

三个业务逻辑方法各自有自己的线程。`thread`和`runThread`方法定义如下：

```java
// Utility methods
def thread(op: => Unit): Thread =
new Thread(new Runnable { def run(): Unit = { op }})
def runThread(t: Thread): Unit = t.start()
```

你可以按照以下方式运行此应用程序：

```java
// Run the app
runThread(eventThread)  // Prints "The user is notified"
```

在这里，每个后续计算都在每个先前计算结束后调用，因为后续计算依赖于先前计算的结果。

代码难以阅读，执行流程难以跟踪。因此，抽象出这些计算的顺序组合是明智的。

# 抽象和函数式示例

让我们看看以函数式方式编写这个示例。在函数式世界中，处理异步计算的一个抽象是 Future。Future 具有以下签名——`Future[A]`。此类型表示在单独的线程中运行的计算，并计算一些结果，在我们的情况下是`A`。

处理`Future`时的一种常见技术是使用回调来指定计算的后续操作。计算的后续操作是在计算完成后要执行的指令。后续操作可以访问它所继续的计算的结果。这是可能的，因为它在计算终止后运行。

在大多数关于`Future`数据类型的用法中，回调模式以某种形式存在。例如，在 Scala 的`Future`实现中，你可以使用`onSuccess`方法将函数的后续操作指定为回调：

```java
def weatherFuture(eventId: Int): Unit = {
  implicit val context =   ExecutionContext.fromExecutorService(Executors.newFixedThreadPool(5))
  Future { getEvent(eventId) }
  .onSuccess { case evt =>
  Future { getWeather(evt.time, evt.location) }
  .onSuccess { case weather => Future { if (weather == "bad") notifyUser } }
}
```

在前一个示例中，我们可以在先前的 Future 终止后使用它们计算的结果启动新的 Future。

此外，请注意我们在运行 Future 之前定义的`implicit val`。它为 Future 引入了一个隐式的执行上下文。Future 是一个在单独的线程中运行的异步计算。它确切地运行在哪个线程上？我们如何控制线程的数量以及是否重用线程？在运行 Future 时，我们需要一个线程策略的规范。

在 Future 类型的 Scala 实现中，我们使用 Scala 的隐式机制将线程上下文引入作用域。然而，在其他语言中，你应该期望存在类似的控制 Future 线程策略的方法。

# 编写未来

需要在异步方式下依次运行多个计算的情况是一个常见的模式。解决这个任务的一种方法是通过回调，正如我们之前所看到的。每个异步计算都是一个独立的实体，并且从一个回调开始，该回调注册在它所依赖的另一个计算上。

另一种构想这种模式的方式是将 Futures 视为可组合的实体。所涉及的概念是将两个 Futures 组合成一个的能力。组合的`Future`的语义是在第一个`Future`之后顺序执行第二个`Future`。

因此，给定一个用于联系数据库的 Future 和一个用于联系天气预报服务器的 Future，我们可以创建一个将两者顺序组合的 Future，第二个 Future 能够使用第一个 Future 的结果。

使用我们已经在上一章中熟悉的`flatMap`方法，我们可以方便地进行顺序组合。因此，我们的示例可以这样实现：

```java
def weatherFutureFlatmap(eventId: Int): Future[Unit] = {
  implicit val context =   ExecutionContext.fromExecutorService(Executors.newFixedThreadPool(5))
  for {
    evt  <- Future { getEvent(eventId) }
    weather <- Future { getWeather(evt.time, evt.location) }
     _  <- Future { if (weather == "bad") notifyUser() }
  } yield ()
}
```

`for`推导式是顺序调用`flatMap`的简写。这种技术被称为**单调流**，存在于一些函数式语言中，包括 Scala 和 Haskell。前面的 Scala 代码是以下代码的语法糖：

```java
def weatherFutureFlatmapDesugared(eventId: Int): Future[Unit] = {
  implicit val context =   ExecutionContext.fromExecutorService(Executors.newFixedThreadPool(5))
  Future { getEvent(eventId) }
  .flatMap { evt => Future { getWeather(evt.time, evt.location) } }
  .flatMap { weather => Future { if (weather == "bad") notifyUser() } }
}
```

# `flatMap`的推广

在上一章中，我们已经看到了在`Try`类型上下文中`flatMap`的使用。它被构想为一个可能产生错误的计算的后继。我们可以将这种构想推广到 Futures 的情况。正如`flatMap`在`Try`的情况下被构想为一个可能产生错误的计算的后继一样，在 Future 的情况下，它是一个异步计算的后继。

`flatMap`函数在处理任何效果类型时的作用大致相同。它是一个产生副作用并带有另一个产生相同副作用但需要第一个计算结果来继续的后继计算。

类似于我们在上一章中在`Try`的情况下使用它的方式，我们也可以为 Futures 定义一个`flatMap`的签名，如下所示—`(A => Future[B]) => (Future[A] => Future[B])`。另一种看待这个`flatMap`函数的方式是，它是一个提升。`flatMap`将产生 Future 副作用并依赖于某些值（如`(A => Future[B])`）的函数提升为一个执行与原始函数相同操作但依赖于`Future[A]`（`Future[A] => Future[B]`）值的函数。也就是说，依赖不再以原始格式存在，而是由另一个产生 Future 副作用的计算来计算。

应该提到的是，Futures 并非仅限于函数式编程。你可以在许多其他语言中遇到它们，例如 Java、JavaScript、Python 以及许多其他语言。异步计算如此普遍，程序员设计出一种原始类型来抽象它们的复杂性是自然而然的。然而，在函数式编程语言，如 Scala 或 Haskell 中，Future 获得了一种我们之前看到的功能性扭曲。

让我们通过一个使用 `Either` 的例子来继续探索副作用以及你可以如何使用它们。

# Either

`Either` 是一种类似于我们在前几章中遇到过的 `Try` 效果。

如果你还记得，`Try` 是一个可以包含两个值之一的结构——一个异常或计算的结果。让我们简要回顾一下前几章中的除以零的例子：

```java
def functionalDivision(n1: Double, n2: Double): Try[Double] =
  if (n2 == 0) Failure(new RuntimeException("Division by zero!"))
  else Success(n1 / n2)
```

在这里，在成功的情况下，我们创建一个 `Success` 数据结构。在失败的情况下，我们需要创建一个带有特定错误信息的异常。

在这里创建异常是必要的吗？毕竟，有用的负载是错误信息。异常在它们被 `throw` 语句抛出时是需要的。然而，正如我们在前几章中讨论的，函数式编程避免这种副作用，而是将其修正为效果类型。如果我们没有抛出异常，那么在 `Failure` 数据结构中显式创建和包装它的意义何在？更有效的方法是返回一个原始的错误信息，例如一个 `String`，而不是带有这个错误信息的异常。然而，当你查看 `Failure` 数据结构的签名时，你会发现它只能包含 `Throwable` 的子类。

为了在错误情况下返回一个字符串而不是异常，我们可以使用另一种数据类型：`Either`。

`Either` 表示两个值之间的一个选择。如果 `Try` 是异常和结果之间的一个选择，那么 `Either` 就是任意两种类型之间的一个选择。它有两个子类。因此，类型为 `Either[A, B]` 的值可以是 `Right[B]` 或 `Left[A]`。传统上，右侧用于成功计算的结果，而左侧用于错误。

让我们看看如何使用这个新的数据结构改进我们的除以零的例子：

```java
def division(n1: Double, n2: Double): Either[String, Double] =
 if (n2 == 0) Left("Division by zero!")
 else Right(n1 / n2)
 println(division(1, 0))  // Left("Division by Zero")
 println(division(2, 2))  // Right(1.0)
```

我们不再需要将错误信息包装在异常中。我们可以直接返回错误信息。函数的结果类型现在是 `Either[String, Double]`，其中 `String` 是我们表示错误的方式，而 `Double` 是结果类型。

应该注意的是，替代的概念可以进一步扩展。`Either` 不是唯一用于抽象替代的数据类型。正如你可能注意到的，`Either` 可以是两个值中的任意一个，但不能同时是两个，也不能是两者都不是。

无论何时你有一个同时拥有两个值的用例，或者当你有一个空的选择时，你可能希望使用其他专门针对此用例的效果类型。为 Scala 或 Haskell 等语言提供的函数式编程库提供此类类型。例如，在 Scala 中，名为`cats`的库提供了可以同时包含两个值的`Ior`数据类型。

我们可能希望同时拥有两个值的用例之一是用于显示警告。如果错误可以理解为导致计算终止而没有产生结果的致命事件，那么警告就是通知你计算中出了问题，但它能够成功终止。在这种情况下，你可能需要一个可以同时包含计算值和生成的警告的数据结构。

错误和异步计算并不是效果类型所解决的唯一领域。现在，让我们看看如何以纯函数式的方式解决依赖注入的问题。让我们来看看`Reader`类型。

# Reader

依赖注入是一种机制，它定义了程序的部分应该如何访问同一程序的其他部分或外部资源。

让我们考虑一个依赖注入变得相关的场景。例如，假设你正在编写一个银行为数据库编写应用程序。该应用程序将包括将你的业务域对象读入和写入数据库的方法。例如，你可能有一个创建新用户的方法和一个为他们创建新账户的方法。这些方法依赖于数据库连接。注入这种依赖的一种方法是将数据库连接对象作为参数传递给这些方法：

```java
def createUser(u: User, c: Connection): Int = ???
def createAccount(a: Account, c: Connection): Int = ???
```

前面的类型定义如下：

```java
class Connection
case class User(id: Option[Int], name: String)
case class Account(id: Option[Int], ownerId: Int, balance: Double)
```

然而，这会使方法的签名变得杂乱。此外，调用数据库的其他方法，如依赖方法，也会变得杂乱，因为它们需要数据库连接对象来满足它们所调用方法的依赖。例如，想象一个业务逻辑方法，它同时为用户创建一个新账户和一个账户：

```java
def registerNewUser(name: String, c: Connection): Int = {
  val uid   = createUser(User(None, name), c)
  val accId = createAccount(Account(None, uid, 0), c)
  accId
}
```

它由两个数据库调用组成，并且由于每个调用都依赖于数据库连接，因此此方法也必须依赖于数据库连接。因此，你必须将数据库连接作为参数提供给业务逻辑方法。将依赖作为参数提供并不方便，因为它将连接对象引入了你的关注点。在业务逻辑层，你希望专注于业务逻辑，而不是数据库连接的工作细节。

# 函数式解决方案

函数式编程为依赖注入问题提供的一种解决方案是，它可以把依赖需求视为一个以依赖项作为参数定义的函数，然后抽象出这个函数。如果我们想这样做，那么首先我们必须定义我们的数据库访问方法如下：

```java
def createUserFunc   (u: User ): Connection => Int = ???
def createAccountFunc(a: Account): Connection => Int = ???
```

这种方法表明，每当有一个依赖于某些外部资源的计算时，我们将这种依赖建模为一个接受此资源作为参数的函数。因此，当我们有一个应该创建用户的函数时，它本身并不执行计算。相反，它返回一个执行计算的函数，前提是你提供了数据库连接。

在这个设置下，如何表达业务逻辑方法如下：

```java
def registerNewUserFunc(name: String): Connection => Int = { c:  Connection =>
  val uid   = createUserFunc(User(None, name))(c)
  val accId = createAccountFunc(Account(None, uid, 0))(c)
  accId
}
```

这种方法与在函数中添加额外参数的方法并没有太大的不同。然而，这是抽象过程的第一个步骤，这个步骤是为了将注意力集中在我们要抽象的效果上。

第二步是抽象出这些函数。实现这一目标的一种方法是将这些函数视为效果。这个效果被用来确保除非你提供其依赖项——函数的参数，否则无法执行由这个函数表示的计算。考虑我们已熟悉的例子，这个例子在`Reader`效果类型的帮助下被重新编写：

```java
def createUserReader   (u: User ): Reader[Connection, Int] = Reader { _ => 0 }  // Dummy implementation, always returns 0
def createAccountReader(a: Account): Reader[Connection, Int] = Reader { _ => 1 }  // Dummy implementation, always returns 1
def registerNewUserReader(name: String): Reader[Connection, Int] =
createUserReader(User(None, name)).flatMap { uid =>
createAccountReader(Account(None, uid, 0)) }
```

`Reader`可以定义为如下：

```java
case class ReaderA, B {
  def apply(a: A): B = f(a)
  def flatMapC: Reader[A, C] =
   Reader { a => f2(f(a))(a) }
}
```

我们可以看到`flatMap`模式和效果类型正在重复出现。之前，我们看到了异步计算和错误的副作用。所有这些都由单独的数据结构表示——`Future`和`Either`（以及`Try`）。现在，我们可以看到依赖的效果。也就是说，这种效果是计算无法执行，除非满足特定的资源需求。这种效果也由其自己的效果类型`Reader`来建模：

正如我们之前所述，我们为`Reader`类提供了`flatMap`方法。这个方法的意义与`Future`和`Try`的情况相同。也就是说，对副作用计算执行后续操作。这个方法可以在依赖于`createUser`和`createAccount`方法的业务逻辑方法设置中使用。

注意到`Readers`本质上就是函数。这意味着你无法在没有提供它们所需的依赖项之前运行它们。为此，你可以调用通常定义在`Reader`数据结构 API 中的一个方法。在我们的例子中，根据前面定义的`Reader`类，可以这样操作：

```java
val reader: Reader[Connection, Int] = registerNewUserReader("John")
val accId = reader(new Connection)
println(s"Success, account id: $accId") // Success, account id: 1
```

# 摘要

在本章中，我们掌握了效果的理论基础，即它们是什么，以及为什么需要它们。我们查看了一些在实践中最常遇到的效果类型示例。我们看到了 `Future` 类型如何抽象处理异步计算。我们还研究了 `Either` 类型，它类似于 `Try`，但允许对错误进行不同的表示。最后，我们介绍了 `Reader` 效果类型，它抽象处理了依赖效果。我们还看到 `flatMap` 是效果类型中的一个典型模式，它抽象处理了副作用计算的顺序组合，并将这些效果修正为效果类型。

在下一章中，我们将探讨如何泛化处理效果类型的工作模式。

# 问题

1.  `Future` 效果类型是如何进行抽象的？

1.  如果我们已经有 `Try` 效果类型，为什么还需要 `Either` 效果类型？

1.  函数式编程如何表示依赖注入？

1.  在我们遇到的所有效果类型中，`flatMap` 函数扮演着什么角色？
