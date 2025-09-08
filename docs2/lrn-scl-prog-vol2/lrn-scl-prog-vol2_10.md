# 与隐式和异常一起工作

“多么讽刺，当你做生意时，你通过创造异常来创造新的机会，当你编写代码（做工作）时，你处理异常以使其变得干净。”

- Pushkar Saraf

函数式程序是表达式。当我们说我们想要运行一个函数式程序时，我们的意思是我们要评估表达式。当我们评估一个表达式时，我们得到一个值。我们还知道，函数式编程是关于组合和评估表达式。这意味着你写下的函数签名对每次评估都成立。但有些情况下，这种情况不太可能发生。你的代码可能不会按预期工作，并可能导致异常行为。我们如何处理这些情况，如何在函数式编程中处理异常？这些问题是基本的，任何刚开始学习函数式编程的人可能会问同样的问题。所以，在本章中，我们将尝试回答这些问题，然后我们将继续前进，看看另一个在 Scala 中非常重要且广为人知的概念，称为 **隐式**。我们将看看它们是什么，以及我们可能想要使用它们的场景。所以，以下是我们在本章中将要讨论的内容：

+   异常处理 - 旧方法

+   使用选项方式

+   要么是左边，要么是右边

+   隐式 - 什么是以及为什么

+   隐式类

+   隐式参数

让我们从向特定功能引入异常行为并处理它开始。

# 异常处理 – 旧方法

让我们编写一些代码，以便我们可以讨论异常处理。看看下面的代码：

```java
def toInt(str: String): Int = str.toInt 
```

在前面的代码中，`toInt` 是一个接受 `String` 值的函数，理论上它可以转换成相应的 `Int` 值。定义看起来没问题，但作为函数式程序员，我们习惯于尝试函数以查看它是否如定义中所说那样工作。让我们尝试调用这个函数：

```java
println(toInt("121")) 
println(toInt("-199")) 
```

前面的代码给出了以下结果：

```java
121 
-199 
```

对于我们来说，一切都很顺利。我们传递了一个字符串格式的数字，并得到了相应的整数值。但是，如果你尝试以下内容会怎样呢？

```java
println(toInt("+ -199")) 
```

假设我们得到了一些意外的情况，一些异常信息如下：

```java
Exception in thread "main" java.lang.NumberFormatException: For input string: "+ -199" 
   at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65) 
```

我们没有得到整数结果，而是一个异常——这很糟糕。但让我们保持积极；我们可以从这次失败中学到一些东西：

+   我们函数的定义不正确。它告诉我们，“你给我一个字符串，我会给你相应的整数。”但事实并非如此。

+   我们知道，除了理想情况外，可能还有我们的操作可能无法完成的情况。

因此，从这次经历中学习，我们现在有一个想法，我们可能想要处理这些不期望的情况。但怎么办呢？

在某些编程语言中，我们得到一个构造函数，它可以包装可能抛出异常的代码块，并在抛出异常时捕获异常，我们允许通过在捕获块中放入我们期望的行为来引入我们期望的行为。这些不过是`try... catch`块。我们为什么不尝试这些块呢？

```java
import java.lang.Exception 

object Main extends App { 

  def toInt(str: String): Int = 
    try{ 
      str.toInt 
    } catch { 
      case exp: Exception => 
        println("Something unexpected happened, you may want to check the string you passed for conversion.") 

        println("WARN: Overriding the usual behavior, returning Zero!") 
        0 
    } 

  println(toInt("121")) 
  println(toInt("-199")) 
  println(toInt("+ -199")) 
} 
```

以下就是结果：

```java
121 
-199 
```

发生了意外的情况；你可能想检查你传递给转换的字符串：

```java
WARN: Overriding the usual behavior, returning Zero! 
0 
try block and also prepared what should be the behavior in case something went wrong. That change gave us a synthetic result, along with a pretty warning message.
```

这个实现看起来合理吗？在某种程度上，它是一个折衷方案，并且它确实做了函数签名中所说的。但在异常情况下返回零仍然不是一个好的选择。

# 使用 Option 方法

让我们尝试更改函数签名，以便我们可以推理和修改它，使其按其所说的那样工作：

```java
def toInt(str: String): Option[Int] = Try(str.toInt) match { 
  case Success(value) => Some(value) 
  case Failure(_) => None 
}
```

在前面的定义中，我们知道响应是可选的。我们可能或可能不会为传递给我们的函数的每个字符串得到相应的整数值。因此，我们将响应类型设为`Option[Int]`。此外，正如你可能已经注意到的，我们使用了`scala.util`包中可用的另一个构造函数，名为`Try`。我们如何使用`Try`？我们传递一个函数给`Try`块的构造函数/`apply`方法。显然，`Try`块的`apply`方法接受一个函数作为`by-name`参数，尝试评估该函数。根据结果或异常，它响应为`Success(value)`或`Failure(exception)`。

我们使用了`Try`构造函数，并将逻辑作为参数传递。在成功的情况下，我们响应为`Some(value)`，在失败的情况下，我们返回`None`。两者都运行良好，因为它们都是`Option`类型的子类型。我们已经在第九章中看到了`Option[+T]`，*使用强大的函数式构造函数*。让我们简单谈谈`Try[+T]`类型。我们将从签名开始：

```java
sealed abstract class Try[+T] extends Product with Serializable 

object Try { 
  /** Constructs a 'Try' using the by-name parameter.  This 
   * method will ensure any non-fatal exception is caught and a 
   * 'Failure' object is returned. 
   */ 
  def applyT: Try[T] = 
    try Success(r) catch { 
      case NonFatal(e) => Failure(e) 
    } 
} 

final case class Success+T extends Try[T] 

final case class Failure+T extends Try[T] 
```

现在我们已经熟悉了工作与参数化类型，理解`Try`的签名将更容易。要注意的两点是`Success`和`Failure`子类型——这里不需要解释它们在这里的作用。让我们看看`Try`类型的伴随对象，它有一个`apply`方法，正如之前讨论的那样。它期望一个`by-name`参数。我们著名的`try... catch`块正在处理其余的事情。

这是你可能想要更改函数签名以处理异常并按其所说的那样工作的方法之一。让我们谈谈一个我们可能想要管道化几个操作的场景——换句话说，我们想要执行函数式组合。看看以下函数定义：

```java
def getAccountInfo(id: String): Option[AccountInfo] 

def makeTransaction(amt: Double, accountInfo: AccountInfo): Option[Double] 

case class AccountInfo(id: String, balance: Double) 
```

看到这两个函数，它们似乎可以一起流水线化执行逻辑。但是如何实现呢？我们可以向我们的`getAccountInfo`函数传递一个账户 ID，该函数随后返回一个可选的`AccountInfo`。我们可以使用这个账户信息和金额调用`makeTransaction`来执行交易。这两个操作看起来足够好，可以组合在一起，但我们唯一的问题是第一个操作的输出是可选的，因此第二个函数可能被调用也可能不被调用。所以对于这个问题，`flatMap`操作看起来是个不错的选择。那么让我们试试看：

```java
import scala.util.Try 

object BankApp extends App { 

  val accountHolders = Map( 
    "1234" -> AccountInfo("Albert", 1000), 
    "2345" -> AccountInfo("Bob", 3000), 
    "3456" -> AccountInfo("Catherine", 9000), 
    "4567" -> AccountInfo("David", 7000) 
  ) 

  def getAccountInfo(id: String): Option[AccountInfo] = Try(accountHolders(id)).toOption 

  def makeTransaction(amt: Double, accountInfo: AccountInfo): Option[Double] = Try(accountInfo.balance - amt).toOption 

  println(getAccountInfo("1234").flatMap(actInfo => makeTransaction(100, actInfo))) 

  println(getAccountInfo("12345").flatMap(actInfo => makeTransaction(100, actInfo))) 
} 

case class AccountInfo(id: String, balance: Double) 
```

以下就是结果：

```java
Some(900.0) 
None 
```

如果我们看一下前面的代码，我们可以看到我们的`getAccountInfo`和`makeTransaction`函数返回可选值，这两个结果中的任何一个都可能为`None`*.* 由于没有好的错误信息告诉我们出了什么问题，所以很难知道哪个操作出了错。所以总结一下，`Option`是一种处理此类场景的方法，但如果我们能知道出了什么问题会更好。为此，我们可以使用 Scala 的另一个结构，名为`Either`。

# 左或右

Scala 为我们提供了一个`Either[+A, +B]`类型。但在我们谈论`Either`之前，让我们先使用它。我们将使用`Either`类型重构我们的代码：

```java
import java.lang.Exception 
import scala.util.{Failure, Success, Try} 

object Main extends App { 

  def toInt(str: String): Either[String, Int] = Try(str.toInt) match { 
    case Success(value) => Right(value) 
    case Failure(exp) => Left(s"${exp.toString} occurred," + 
      s" You may want to check the string you passed.") 
  } 

  println(toInt("121")) 
  println(toInt("-199")) 
  println(toInt("+ -199")) 
} 
```

以下就是结果：

```java
Right(121) 
Right(-199) 
Left(java.lang.NumberFormatException: For input string: "+ -199" occurred, You may want to check the string you passed.) 
```

在前面的代码中，我们知道从`string`到`int`的转换可能会出错。所以结果可能是一个异常或预期的整数。所以我们尝试做同样的事情：我们使用`Either`类型，当出错时左值是一个`String`消息，而右值是一个`Int`。为什么这样做呢？让我们看一下`Either`类型的签名来理解这一点：

```java
sealed abstract class Either[+A, +B] extends Product with Serializable 
final case class Right+A, +B extends Either[A, B] 
final case class Left+A, +B extends Either[A, B] 
```

从前面的签名中，我们可以看到`Either`类型接受两个类型参数，`A`和`B`；按照惯例，我们认为`Left`值是异常情况值，而右值是预期的结果值。这就是为什么我们声明响应类型如下：

```java
Either[String, Int] 
```

这表示我们期望得到一个`String`或`Int`值。所以用例很清楚。我们知道了我们的操作发生了什么——即从字符串到相应整数的转换。现在，为什么我们不尝试使用`Either`类型进行一些函数组合呢？我们可以用同样的场景来做这件事：

```java
import scala.util.{Failure, Success, Try} 

object BankApp extends App { 

  val accountHolders = Map( 
    "1234" -> AccountInfo("Albert", 1000), 
    "2345" -> AccountInfo("Bob", 3000), 
    "3456" -> AccountInfo("Catherine", 9000), 
    "4567" -> AccountInfo("David", 7000) 
  ) 

  def getAccountInfo(id: String): Either[String, AccountInfo] = Try(accountHolders(id)) match { 
    case Success(value) => Right(value) 
    case Failure(excep) => Left("Couldn't fetch the AccountInfo, Please Check the id passed or try again!") 
  } 

  def makeTransaction(amount: Double, accountInfo: AccountInfo): Either[String, Double] = Try { 
    if(accountInfo.balance < amount) throw new Exception("Not enough account balance!") else accountInfo.balance - amount 
  } match { 

    case Success(value) => Right(value) 
    case Failure(excep) => Left(excep.getMessage) 
  }

  println(getAccountInfo("1234").flatMap(actInfo => makeTransaction(100, actInfo))) 

  println(getAccountInfo("1234").flatMap(actInfo => makeTransaction(10000, actInfo))) 

  println(getAccountInfo("12345").flatMap(actInfo => makeTransaction(100, actInfo))) 
} 

case class AccountInfo(id: String, balance: Double) 
```

以下就是结果：

```java
Right(900.0) 
Left(Not enough account balance!) 
Left(Couldn't fetch the AccountInfo, Please Check the id passed or try again!) 
```

这很有趣。这个新的结构让我们的生活变得更简单，并给我们提供了关于失败的有意义的信息。现在我们也能识别出哪里出了问题，以及何时出了问题。

我们可以看到`Either`帮助我们更好地处理异常。我们也看到了几种处理异常情况的方法。这次讨论的收获是什么？让我们总结一下。

我们已经看到了一些在 Scala 程序中处理异常场景的构造。你可能认为其中一个构造`Try[+T]`除了使用一个*`try... catch`块来处理异常之外，没有做什么。所以我们对这个论点的回应是关于*函数组合*的。你可能会选择使用`scala.util.Try[+T]`而不是普通的`try... catch`块，原因在于函数组合*。**

*这个类型提供了一些函数，例如`map`用于转换和`flatMap`用于组合，这样我们就可以使用`flatMap`操作将两个操作组合在一起。如果你想知道这是什么，让我告诉你，我们已经看到了这个例子。我们想要使用`flatMap`方法将两个函数组合起来得到结果，这之所以可能，仅仅是因为我们的类型`Try`*、*`Option`和`Either`有这个看起来很疯狂的功能`flatMap`*。看看`flatMap`方法的实现是值得的。这个`Option`的`flatMap`函数可能看起来如下：

```java
def flatMapA, B(functionToPerfom: A => Option[B]): Option[B] = 
  if (someValue.isEmpty) None else functionToPerfom(someValue.get) 
```

根据签名，我们将传递`Option[A]`。这里的`A`参数不过是一个类型参数和一个形式为`A => Option[B]`的函数，定义将返回类型`Option[B]`。这很强大，并帮助我们组合这两个函数。这就是你可能想要选择`Option`/`Either`/`Try`构造的原因之一。三个中哪一个将被使用取决于用例。`Either`类型在你出错时提供了返回消息的便利。

所以这就解释了我们在 Scala 程序中如何处理异常。现在让我们继续讨论 Scala 提供的一个概念，让你能够隐式地做事情。让我们来谈谈 Scala 中的隐式参数。

# 隐式参数 - 什么是隐式参数以及为什么

什么是*隐式参数*？当我们谈论隐式参数时，我们指的是隐式参数或隐式转换。隐式参数是与关键字`implicit`一起出现的参数，如果这些参数在作用域内，我们不需要显式传递这些参数的参数。让我们看看它是如何工作的。

让我们举一个例子，并创建一个`Future`值。`Future`不过是我们提供的将在未来某个时间点发生的计算。这意味着一个将在未来发生的计算。当我们讨论第十三章中的并发编程技术时，我们将深入讨论`Future`值，*Scala 中的并发编程*。现在我们先写一个代码片段：

```java
import scala.concurrent.Future 

object FuturesApp extends App { 

  val futureComp = Future { 
     1 + 1 
  } 

  println(s"futureComp: $futureComp") 

  futureComp.map(result => println(s"futureComp: $result")) 
} 
Future block and that we are then printing out this Future instance. After that, we're extracting the computation's result out of the Future value and printing it. Looks like it should work fine. Let's run this. We will get the following result:
```

```java
Error:(7, 27) Cannot find an implicit ExecutionContext. You might pass 
an (implicit ec: ExecutionContext) parameter to your method 
or import scala.concurrent.ExecutionContext.Implicits.global. 
  val futureComp = Future { 

Error:(7, 27) not enough arguments for method apply: (implicit executor: scala.concurrent.ExecutionContext)scala.concurrent.Future[Int] in object Future. 
Unspecified value parameter executor. 
  val futureComp = Future { 
```

Scala 编译器给我们带来了两个编译错误。第一个错误说它找不到`ExecutionContext`类型的隐式值*.* 好的，我们现在还不知道`ExecutionContext`是什么。让我们看看下一个错误。它说`方法 apply 的参数不足：`(implicit executor: ExecutionContext) scala.concurrent.Future[Int]`*.*

现在，我们有一个想法，即有一个需要但对我们代码不可用的参数。让我们看看`Future`块的`apply`方法来解决这个问题：

```java
def applyT(implicit executor: ExecutionContext): Future[T] 
```

好的，这似乎很有趣。我们为参数`ExecutionContext`提供了一个`implicit`关键字。这意味着调用`Future`块的`apply`方法是允许的；我们唯一需要关注的是声明类型的隐式值。所以，如果我们能以某种方式将`ExecutionContext`类型的值引入我们的作用域，事情应该会顺利。我们所说的作用域是什么意思呢？让我们暂时将当前的编译单元（Scala 文件）视为作用域。那么，让我们这样做：

```java
import scala.concurrent.Future 

object FuturesApp extends App { 

  implicit val ctx = scala.concurrent.ExecutionContext.Implicits.global 

  val futureComp = Future { 
     1 + 1 
  } 

  println(s"futureComp: $futureComp") 

  futureComp.map(result => println(s"futureComp: $result")) 
} 
```

以下结果是：

```java
futureComp: Future(Success(2)) 
futureComp: 2 
```

我们声明了一个名为`ctx`的`implicit`值，其类型为`ExecutionContext`，然后再次尝试运行应用程序，神奇的是一切正常。我们没有明确传递任何上下文或做任何特别的事情——我们只是将所需类型的值引入了作用域，事情就顺利了。我们得到了结果。不过，有一点需要注意，那就是我们使用了这个`implicit`关键字；这就是为什么`Future.apply`能够推断出作用域中可用的值。如果我们没有使用`implicit`关键字尝试这样做，我们会得到与之前类似的编译错误。所以，我们的想法是在作用域中获取一个隐式值，现在我们知道什么是隐式了。不过，有一个大问题：你为什么想要这种行为呢？我们将就这个想法进行一次有益的讨论。

让我们从这样的想法开始，即 Scala 中的隐式可以用来自动化将值传递给操作或从一种类型到另一种类型的转换的过程。让我们先谈谈第一个：隐式参数。

# 隐式参数

当我们想要编译器帮助我们找到一个已经为某种类型可用的值时，我们会使用隐式参数。当我们谈论`Future`时，我们已经看到了一个隐式参数的例子。为什么我们不为自己定义一个类似的东西呢？

我们可以想象一个场景，我们需要在我们的应用程序中显示当前日期，并且我们希望避免再次明确传递日期实例。相反，我们可以使`LocalDateTime.now`值对相应的函数是隐式的，让当前日期和时间作为隐式参数传递给它们。让我们为这个写一些代码：

```java
import java.time.{LocalDateTime} 

object ImplicitParameter extends App { 

  implicit val dateNow = LocalDateTime.now() 

  def showDateTime(implicit date: LocalDateTime) = println(date) 

  //Calling functions! 
  showDateTime 
} 
```

以下结果是：

```java
2017-11-17T10:06:12.321 
```

将`showDateTime`函数视为需要日期和时间当前值的函数——因此，我们可以将其作为隐式值提供。这就是我们做的——在`showDateTime`的定义中，我们声明了一个名为`date`的隐式参数，其类型为`LocalDateTime`。我们还有一个名为`dateNow`的隐式值在作用域中。这就是为什么我们不需要在调用点传递参数，事情仍然对我们很顺利。

这似乎是一个很好的用例。你可以使用*隐式*来使你需要的值自动对你自己可用。

# 隐式方法

Scala 的标准库提供了一个创建类型具体实例的实用方法，该方法的名称也是`implicitly`*.* 让我们看看函数签名：

```java
def implicitlyT = e 
```

这个`implicitly`方法只期望一个类型参数，找到作用域中可用的隐式值，并召唤并返回它给我们。这是我们用来判断特定类型的值是否在隐式作用域中可用的一个好选项。让我们看看这个方法的一个应用：

```java
import java.time.{LocalDateTime} 

object ImplicitParameter extends App { 

  implicit val dateNow = LocalDateTime.now() 

  def showDateTime(implicit date: LocalDateTime) = println(date) 

  val ldt = implicitly[LocalDateTime] 

  println(s"ldt value from implicit scope: $ldt") 
} 
```

以下结果是：

```java
ldt value from implicit scope: 2017-12-17T10:47:13.846 
implicitly, along with the type, returned us the value available—as we already know, it's the current date-time value.
```

因此，这就是我们如何在定义中使用`implicit`参数，并使它们在相应的范围内可用。

现在我们对隐式转换有了些了解，让我们来看看*隐式转换*。

# 隐式转换

标准 Scala FAQ 页面将隐式转换描述为：“*如果在对象`o`的类`C`上调用方法`m`，而该类`C`不支持方法`m`，那么 Scala 编译器将寻找从`C`类型到支持`m`方法的类型的隐式转换*”。

理念很清晰：这是一种合成行为（使用方法），我们正在强制应用于特定类型的实例，而这些行为（方法）并不是定义类型的一部分。这就像我们有一个具有某些功能的库，我们希望向库中的某个类型添加一些附加功能。想想看——这是强大的。能够为特定类型添加功能本身就是强大的。这正是隐式转换让我们做到的。我们将尝试做一些类似以下的事情。

首先，考虑一个我们需要创建一些语法方法的场景。我们有一些可用于日期时间库`java.time.LocalDate`的方法，可以帮助我们添加或减去天数/周数/月数/年数，这些方法包括：

```java
scala> import java.time.LocalDate 
import java.time.LocalDate 

scala> val ld = LocalDate.now 
ld: java.time.LocalDate = 2017-12-17 

scala> ld.plusDays(1) 
res0: java.time.LocalDate = 2017-12-18 

scala> ld.plusWeeks(1) 
res1: java.time.LocalDate = 2017-12-24 

scala> ld.plusMonths(1) 
res2: java.time.LocalDate = 2018-01-17 

scala> ld.plusYears(1) 
res3: java.time.LocalDate = 2018-12-17 
```

我们希望有一个简单的加号（+）或减号（-）用于表示天数/周数/月数/年数，以便它们能像`plusXXX`或`minusXXX`方法一样工作。我们有哪些选项可以实现这样的语法？

其中一个选项是在`LocalDate`上创建一个`Wrapper`类，例如`CustomDate(date: LocalDate)`，并为它定义这些方法。在这种情况下，代码可能看起来像这样：

```java
import java.time.LocalDate 

case class CustomDate(date: LocalDate) { 
  def +(days: Day): CustomDate = CustomDate(this.date.plusDays(days.num)) 
  def -(days: Day): CustomDate = CustomDate(this.date.minusDays(days.num)) 

  def +(weeks: Week): CustomDate = CustomDate(this.date.plusWeeks(weeks.num)) 
  def -(weeks: Week): CustomDate = CustomDate(this.date.minusWeeks(weeks.num)) 

  def +(months: Month): CustomDate = CustomDate(this.date.plusMonths(months.num)) 
  def -(months: Month): CustomDate = CustomDate(this.date.minusMonths(months.num)) 

  def +(years: Year): CustomDate = CustomDate(this.date.plusYears(years.num)) 
  def -(years: Year): CustomDate = CustomDate(this.date.minusYears(years.num)) 

  def till(endDate: CustomDate): CustomDateRange = if(this.date isBefore endDate.date) 
    CustomDateRange(this, endDate) 
  else { 
    throw new IllegalArgumentException("Can't create a DateRange with given start and end dates.") 
  } 

  override def toString: String = s"Date: ${this.date}" 
} 

case class Day(num: Int) 
case class Week(num: Int) 
case class Month(num: Int) 
case class Year(num: Int) 

case class CustomDateRange(sd: CustomDate, ed: CustomDate){ 
  override def toString: String = s"$sd till $ed " 
} 
```

如您在前面的代码中所注意到的，我们有一个`CustomDate`类封装了`LocalDate`类型，并使用`LocalDate`类型的这些方法来定义我们自己的期望语法方法。让我们尝试使用它。为此，我们可以创建另一个扩展`App`特质的对象：

```java
import java.time.LocalDate 

object BeautifulDateApp extends App { 

  val today = CustomDate(LocalDate.now()) 
  val tomorrow = today + Day(1) 
  val yesterday = today - Day(1) 

  println(today) 
  println(tomorrow) 
  println(today + Year(1)) 

  val dateRange = today till tomorrow + Day(20) 
  println(dateRange) 

} 
```

以下结果是：

```java
Date: 2017-12-17 
Date: 2017-12-18 
Date: 2018-12-17 
Date: 2017-12-17 till Date: 2018-01-07 
LocalDate gives us the feeling that this syntax isn't a part of the standard library we have. So for this, implicits come into the picture. We're going to do a similar syntax hack using the implicit class.
```

为了做到这一点，我们将创建一个隐式类，它只接受一个`val`类型的`LocalDate`*，然后使用类似的逻辑提供我们所有的语法方法。之后，我们将通过导入它来将这个隐式类引入作用域。让我们写下这个：

```java
case class Day(num: Int) 
case class Week(num: Int) 
case class Month(num: Int) 
case class Year(num: Int) 

case class CustomDateRange(sd: CustomDate, ed:CustomDate){ 
  override def toString: String = s"$sd till $ed " 
} 

object LocalDateOps { 
  implicit class CustomDate(val date: LocalDate) { 

    def +(days: Day): CustomDate = CustomDate(this.date.plusDays(days.num)) 
    def -(days: Day): CustomDate = CustomDate(this.date.minusDays(days.num)) 

    def +(weeks: Week): CustomDate = CustomDate(this.date.plusWeeks(weeks.num)) 
    def -(weeks: Week): CustomDate = CustomDate(this.date.minusWeeks(weeks.num)) 

    def +(months: Month): CustomDate = CustomDate(this.date.plusMonths(months.num)) 
    def -(months: Month): CustomDate = CustomDate(this.date.minusMonths(months.num)) 

    def +(years: Year): CustomDate = CustomDate(this.date.plusYears(years.num)) 
    def -(years: Year): CustomDate = CustomDate(this.date.minusYears(years.num)) 

    def till(endDate: CustomDate): CustomDateRange = if(this.date isBefore endDate.date) 
      CustomDateRange(this, endDate) 
    else { 
      throw new IllegalArgumentException("Can't create a DateRange with given start and end dates.") 
    } 

    override def toString: String = s"Date: ${this.date}" 
  } 
} 
```

现在，是时候在我们的`BeautifulDateApp`类中使用它了*：

```java
import java.time.LocalDate 
import LocalDateOps._ 

object BeautifulDateApp extends App { 

  val today = LocalDate.now() 
  val tomorrow = today + Day(1) 
  val yesterday = today - Day(1) 

  println(today) 
  println(tomorrow) 
  println(today + Year(1)) 

  val dateRange = today till tomorrow + Day(20) 
  println(dateRange) 
} 
```

以下结果是：

```java
2017-12-17 
Date: 2017-12-18 
Date: 2018-12-17 
Date: 2017-12-17 till Date: 2018-01-07 
```

我们可以看到我们采取的两种方法之间的区别。第二种方法似乎更符合本地方法。作为这些语法方法的消费者，我们从未尝试调用`CustomDate`类——相反，我们创建了一个`LocalDate`类型的实例*：*

```java
  val today = LocalDate.now() 
```

我们使用了`+`和`-`，就像在`LocalDate`类中定义的本地方法一样。这就是力量，或者说魔法，在于*隐式转换*。对于那些想知道幕后发生了什么的人来说，让我们更详细地看看代码的工作原理。

Scala 编译器看到了以下内容：

```java
val tomorrow = today + Day(1) 
```

然后，编译器试图在`LocalDate`类中寻找一个名为`+`的方法，该方法接受一个日期作为参数。编译器无法在那里找到这样的方法并不奇怪，因此它试图检查是否在隐式作用域中存在任何其他类，该类期望一个`LocalDate`，并且执行了诸如`+`这样的操作（日期/周/月/年）。然后，编译器找到了我们的`CustomDate`隐式类。最后，发生了隐式转换，这个特定的方法调用对我们有效。然后我们能够使这样的方法语法黑客成为可能。

现在我们已经看到了这样的例子，我们可能想要问自己一个问题：我们所说的*隐式作用域*是什么意思？我们还需要了解 Scala 编译器如何搜索隐式值。让我们尝试找到这个答案。

# 查找隐式值

您的常规 Scala 应用程序代码可能包含一些导入其他类和对象的构造，或者它也可能继承其他类。您编写的方法期望类型作为参数，并声明参数。因此，当 Scala 编译器寻找隐式值时，它应该在何处开始寻找这样的值？编译器开始根据以下标准寻找隐式值：

+   在当前作用域中定义

+   明确导入

+   使用通配符导入

+   类型伴生对象

+   参数类型的隐式作用域

+   类型参数的隐式作用域

+   嵌套类型的外部对象

我们知道，如果我们当前作用域（代码块）中定义了一个隐式值，它将获得最高的优先级。之后，您也可以使用`import`语句导入它，如下面的代码所示：

```java
import scala.concurrent.Future 
import scala.concurrent.ExecutionContext.Implicits.global 

object FuturesApp extends App { 

  val futureComp = Future { 
     1 + 1 
  } 

  println(s"futureComp: $futureComp") 

  futureComp.map(result => println(s"futureComp: $result")) 
} 
```

以下结果是：

```java
futureComp: Future(Success(2)) 
futureComp: 2 
```

通配符导入也可以适用于此：

```java
import scala.concurrent.ExecutionContext.Implicits._ 
```

但是，当编译器看到同一作用域中存在两个适用于相同类型的隐式值时，生活和我们都感到有些不舒服。然后我们看到的是一个编译错误，指出存在`歧义隐式值`*。让我们试试看：

```java
import scala.concurrent.Future 
import scala.concurrent.ExecutionContext.Implicits.global 

object FuturesApp extends App { 

  implicit val ctx = scala.concurrent.ExecutionContext.Implicits.global 

  val futureComp = Future { 
     1 + 1 
  } 

  println(s"futureComp: $futureComp") 

  futureComp.map(result => println(s"futureComp: $result")) 
} 
```

对于前面的代码，我们将面临以下编译错误：

```java
Error:(10, 27) ambiguous implicit values: 
 both lazy value global in object Implicits of type => scala.concurrent.ExecutionContext 
 and value ctx in object FuturesApp of type => scala.concurrent.ExecutionContext 
 match expected type scala.concurrent.ExecutionContext 
  val futureComp = Future { 
```

因此，我们需要注意隐式值的歧义。

如果编译器无法在当前代码块或通过导入中找到隐式值，它将在类型的伴生对象中搜索它。这就是编译器搜索隐式值的方式。标准的 Scala 文档解释了查找隐式的主题，你可以在[`docs.scala-lang.org/tutorials/FAQ/finding-implicits.html`](http://docs.scala-lang.org/tutorials/FAQ/finding-implicits.html)找到它。

通过对隐式的讨论，我们看到了我们可以使用这个概念并让魔法为我们工作的几种方式。当库设计者在定义类型类并通过隐式值提供它们的实例时，这被广泛使用。我们已经涵盖了类型类是什么，并且我们可以自己创建一个。让我们试试：

# 接下来是类型类！

当创建类型类来解决像为特定格式提供编码类型机制这样的问题时，我们必须释放像 Scala 这样的语言的力量。我们想要的编码特定类型值的逗号分隔值（CSV）格式的方法。为此，我们将创建一个名为 `CSVEncoder` 的类型类。在 Scala 中，我们可以通过使用某种类型的 trait 来做这件事：

```java
trait CSVEncoder[T]{ 
  def encode(value: T): List[String] 
} 
```

我们定义的是一个为我们类型提供功能的功能提供者。目前的功能是将特定类型的值编码并返回一个我们可以表示为 CSV 的字符串值列表。现在，你可能想通过调用一些函数来使用这个功能，对吧？对于像 `Person` 这样的简单类型，它可以看起来像这样：

```java
case class Person(name: String) 

CSVEncoder.toCSV(Person("Max")) 
```

一些其他的语法可能看起来像这样：

```java
Person("Caroline").toCSV 
```

要使用类似这些的东西，我们需要的是这个：

+   一种将 `Person` 类型编码为 CSV 格式的方法

+   实用函数 `toCSV`

让我们定义我们的类型类提供的功能可以使用的方法：

```java
object CSVEncoder { 

 def toCSVT(implicit encoder: CSVEncoder[T]): String = 
  list.map(mem => encoder.encode(mem).mkString(", ")).mkString(", ") 

} 
```

在这里，我们为 `CSVEncoder` 定义了一个伴生对象，并定义了一个名为 `toCSV` 的实用函数，它接受一个类型参数和相同类型的值序列，除了它期望一个相同类型的隐式 `CSVEncoder` 实例。它返回的是一个 `List[String]`*.* 我们知道将字符串值序列转换为 CSV 很容易。这就是我们从这个函数中想要的东西。因此，我们简单地调用 `encoder.encode(value)` 并将值转换为逗号分隔的格式。

现在，让我们定义一种编码 `Person` 类型的方法：

```java
implicit val personEncoder: CSVEncoder[Person] = new CSVEncoder[Person] { 
  def encode(person: Person) = List(person.name) 
} 
```

在前面的代码中，我们提供了一种编码我们的 `Person` 类型的方法。现在，让我们来使用它：

```java
object EncoderApp extends App { 
  import CSVEncoder.personEncoder 

  println(CSVEncoder.toCSV(List(Person("Max Black"), Person("Caroline Channing")))) 

} 
```

以下就是结果：

```java
Max Black, Caroline Channing 
```

在我们的 `EncoderApp` 中，我们隐式地导入了 `CSVEncoder[Person]` 并调用一个带有预期值的 `toCSV` 函数。调用这个函数会给我们期望的结果*.* 我们现在可以使用隐式类来修改 `toCSV` 函数的语法，并为我们的类型类的消费者提供另一种使用我们的编码器的方式。让我们这么做：

```java
trait CSVEncoder[T]{ 
  def encode(value: T): List[String] 
} 

object CSVEncoder { 

  def toCSVT(implicit encoder: CSVEncoder[T]): String = 
    list.map(mem => encoder.encode(mem).mkString(", ")).mkString(", ") 

  implicit val personEncoder: CSVEncoder[Person] = new CSVEncoder[Person] {
     def encode(person: Person) = List(person.name) 
  } 

} 

case class Person(name: String) 

object EncoderApp extends App { 
  import CSVEncoder._ 
  import CSVEncoderOps._ 

  println(CSVEncoder.toCSV(List(Person("Max Black"), Person("Caroline Channing")))) 

  println(List(Person("Max Black"), Person("Caroline Channing")).toCSV) 
} 

object CSVEncoderOps { 
  implicit class CSVEncoderExtT { 
    def toCSV(implicit encoder: CSVEncoder[T]) : String = 
      list.map(mem => encoder.encode(mem).mkString(", ")).mkString(", ") 
  } 
} 
```

以下就是结果：

```java
Max Black, Caroline Channing 
Max Black, Caroline Channing 
toCSV function as a method:
```

```java
List(Person("Max Black"), Person("Caroline Channing")).toCSV 
```

我们使用隐式 `CSVEncoderExt` 类实现了这种语法调用，这是我们为 `LocalDate` 的语法方法所采取的方法：

```java
implicit class CSVEncoderExtT { 
    def toCSV(implicit encoder: CSVEncoder[T]) : String = 
      list.map(mem => encoder.encode(mem).mkString(", ")).mkString(", ") 
  } 
```

我们所需要做的就是确保这个特定的类在调用点的作用域内，所以我们导入了它。这就是我们创建和使用我们的第一个类型类的方式。这并不难，对吧？当然，我们已经在本章中足够详细地介绍了类型类。让我们继续总结本章我们所学的知识。

# 摘要

首先，我们讨论了在尝试编程时出现的异常情况。我们看到了在函数式编程中如何处理这些异常情况。我们甚至在函数组合中尝试了异常处理。然后，我们开始看到 Scala 中隐式带来的魔法。我们讨论了隐式参数和*隐式转换*。我们看到了由 Scala 标准库提供的`implicitly`方法。最后，我们谈论了已经讨论得很多的类型类，并定义/使用了我们的第一个类型类。一旦你足够多地练习了我们讨论的概念，详细学习类型类是值得的。Scala 的大多数库框架都大量使用了这个概念。

在下一章，我们将学习 Akka 工具包。我们将涵盖 Akka 提供的一项服务，即*Actor 系统*，以及更多内容。
