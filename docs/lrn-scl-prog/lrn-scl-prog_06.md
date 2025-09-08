# 第六章：探索内置效果

有时，计算机的行为与开发者的预期不同。有时，一个函数可能无法为给定的一组参数返回值，设备在运行时不可用，或者调用外部系统所需的时间比预期长得多。

函数式方法努力捕捉这些方面，并用类型来表示它们。这允许对程序进行精确推理，并有助于在运行时避免意外。

在本章中，我们将研究这些方面是如何被 Scala 标准库所涵盖的。我们将查看以下内容：

+   使用类型编码运行时方面的基础

+   选项

+   或者

+   尝试

+   未来

# 技术要求

在我们开始之前，请确保您已安装以下内容：

+   JDK 1.8+

+   SBT 1.2+

本章的源代码可在我们的 GitHub 仓库中找到：[`github.com/PacktPublishing/Learn-Scala-Programming/tree/master/Chapter06`](https://github.com/PacktPublishing/Learn-Scala-Programming/tree/master/Chapter06).

# 效果简介

Scala 代码编译成 Java 字节码，并在 JVM（Java 虚拟机）上运行。正如其名所示，JVM 并非专门为 Scala 构建。因此，Scala 语言能够表达的内容与 JVM 支持的内容之间存在不匹配。后果是双重的：

+   编译器将 Scala 中 JVM 不支持的功能转换为适当的字节码，主要是通过创建包装类。因此，一个简单的 Scala 程序的编译可能会导致创建数十或数百个类，这反过来又会导致性能下降和更高的垃圾回收足迹。本质上，这些负面后果只是实现细节。随着 JVM 的改进，有可能优化编译器为 Java 的新版本生成的字节码，而无需应用程序开发者的任何努力。

+   从相反的方向来看，平台有一些特性与 Scala 并不特别一致。尽管如此，对这些特性的支持是必要的，部分原因是出于 Scala 与 Java 互操作性的原因，部分原因是因为如果底层运行时发生某些情况，它需要能够在语言中表达出来。

对于我们这些语言使用者来说，第一类差异更多的是一种理论上的兴趣，因为我们很舒服地假设编译器开发者正在尽最大努力生成最适合当前 JVM 版本的字节码，因此我们可以依赖他们。第二类差异对我们影响更直接，因为它们可能会影响我们编写代码的方式，并且它们肯定会影响我们与现有库（尤其是 Java 库）交互时编写的代码。

在这里，我们讨论的是可能会对我们的代码推理能力产生负面影响的功能，特别是那些破坏其引用透明性的功能。

为了说明最后一点，让我们看一下以下代码：

```java
scala> import java.util
import java.util
scala> val map = new util.HashMap[String, Int] { put("zero", 0) }
map: java.util.HashMap[String,Int] = {zero=0}
scala> val list = new util.ArrayList[String] { add("zero") }
list: java.util.ArrayList[String] = [zero]
scala> println(map.get("zero"))
0
scala> println(list.get(0))
zero
```

我们定义了一个 Java `HashMap`，一个`ArrayList`，在其中放入了一些项目，并如预期地得到了这些项目。到目前为止，一切顺利。

让我们进一步探讨：

```java
scala> println(map.get("one"))
null
scala> :type null
Null
```

对于在`map`中找不到的元素，我们得到了一个`null: Null`的返回值，这有点出乎意料。这真的那么糟糕吗？嗯，可能确实如此：

```java
scala> println(map.get("one").toString)
java.lang.NullPointerException
  ... 40 elided
```

我们遇到了一个`NullPointerException`，如果不捕获它，程序在运行时将会崩溃！

好吧，但我们*可以*检查返回的元素是否为`null`。我们只需要记住每次调用可能返回`null`的函数时都要这样做。让我们用`list`来做这个操作：

```java
scala> list.get(1) match {
     | case null => println("Not found")
     | case notNull => println(notNull)
     | }
java.lang.IndexOutOfBoundsException: Index: 1, Size: 1
  at java.util.ArrayList.rangeCheck(ArrayList.java:653)
  at java.util.ArrayList.get(ArrayList.java:429)
  ... 49 elided
```

哦，列表对于缺失的元素不会返回`null`，它只是直接抛出`IndexOutOfBoundsException`！看起来我们需要在我们的调用中添加一个`catch`子句，以便使其更安全...

到目前为止，我们的观点已经很明确了——在没有查看 JavaDocs，最终查看实现源代码的情况下，很难或不可能推理出以这种风格编写的某些代码的执行可能的结果。原因在于我们调用的函数可以以它们类型中未编码的方式返回结果。在第一个例子中，函数在集合中没有元素的情况下返回一个特殊的`null`值。但这个特殊值也可能是其他东西！另一个例子是`ArrayList`上定义的`indexOf`方法的`-1`。还有另一种情况是指出无法执行操作，这是通过抛出异常来完成的。

从某种意义上说，我们调用的函数改变了它们执行的环境。在异常的情况下，程序的执行路径改变以传播异常，而在返回`null`的情况下，调用者的期望发生了变化，不幸的是，不是在编译时，而是在运行时。

在函数式编程中，我们称这种行为为*效果*，并努力在类型级别上表达这种效果。函数式编程（FP）中的效果与*副作用*重叠，但代表了一个更广泛的概念。例如，结果的可选性（返回`null`）并不是副作用。

效果的优势在于它们不需要在语言级别上实现。正因为如此，根据库作者的目标和架构考虑，可以以不同的方式设计相同的效果。最重要的是，它们可以被扩展和组合，这允许我们以结构化和类型安全的方式表示复杂的效果集。

在本章的后续部分，我们将探讨标准库中可用的四种不同类型的效果，从`Option`开始。

# 可选

`Option`可能是新手 Scala 开发者首先熟悉的效果。它编码了函数可能不返回结果的情况。简化来说，它在`stdlib`中以代数数据类型的形式表示，如下面的代码所示：

```java
sealed abstract class Option[+A]
case object None extends Option[Nothing]
final case class Some+A extends Option[A]
```

`None`表示不存在结果的情况，而`Some(value)`表示存在结果的情况。接下来，我们将探讨一种三步法来更深入地理解如何使用`Option`——如何创建它、从它（如果有的话）读取值以及从`Option`是一个效果的事实中产生的可能性。

# 创建一个 Option

`Option`可以通过多种方式创建。最直接的方法，尽管不推荐，是使用情况类的构造器或直接返回`None`：

```java
val opt1 = None
val opt2 = Some("Option")
```

这是不推荐的，因为绝对有可能再次返回被`Option`包裹的`null`，从而违背了它的目的：

```java
val opt3 = Some(null)
```

因此，我们需要首先检查构造器参数是否为`null`：

```java
def opt4A: Option[A] = if (a == null) None else Some(a)
```

实际上，这种模式如此常见，以至于`Option`伴随对象提供了相应的构造器：

```java
def opt5A: Option[A] = Option(a)
```

伴随对象定义了几个更多的构造器，允许你完全避免直接使用`Some`或`None`：

```java
val empty = Option.empty[String]
val temperature: Int = 26
def readExactTemperature: Int = 26.3425 // slow and expensive method call
val temp1 = Option.when(temperature > 45)(readExactTemperature)
val temp2 = Option.unless(temperature < 45)(readExactTemperature)
```

第一个构造器创建类型`None`，第二个和第三个返回`Some`，但只有当条件分别为`true`或`false`时。第二个参数是一个按名传递的参数，并且只有在条件成立时才会计算。

# 从 Option 中读取

现在我们有一个`Option`，需要从中取出值。最明显的方法是在“空值检查”风格中这样做：

```java
if (opt1.isDefined) println(opt1.get)
if (opt1.isEmpty) println("Ooops, option is empty") else println(opt1.get)
```

在这里，我们使用了两种空值检查中的一种，在这种情况下，如果`Option`非空，我们调用`.get`来检索其值。除了相当冗长之外，这种方法的主要缺点是很容易忘记检查`Option`是否已定义。如果调用`None.get`，它将抛出`NoSuchElementException`：

```java
scala> None.get
java.util.NoSuchElementException: None.get
  at scala.None$.get(Option.scala:378)
  ... 40 elided
```

还有几个方法允许你检查`Option`的内容是否满足给定的条件，但它们都以相同的方式存在问题：

```java
if (option.contains("boo")) println("non-empty and contains 'boo'")
if (option.exists(_ > 10)) println("non-empty and greater then 10")
if (option.forall(_ > 10)) println("empty or greater then 10")
```

`contains`方法在`Option`被定义的情况下将其参数与`Option`的内容进行比较。`exists`接受一个谓词，如果`Option`非空，则应用于`Option`的值。与其它方法相比，`forall`是特殊的，因为它在应用于参数的谓词对非空`Option`成立或`Option`为空时返回`true`。

从`Option`中获取值的另一种方式是解构它：

```java
if (opt.isDefined) { val Some(value) = opt }
```

你也可以使用模式匹配来完全避免检查非空性：

```java
opt match {
  case Some(value) => println(value)
  case None => println("there is nothing inside")
}
```

有时，对于调用者来说，如果`Option`为空，只需要一个“默认”值。为此有一个特殊的方法叫做`getOrElse`：

```java
val resultOrDefault = opt.getOrElse("No value")
```

另一个类似的方法是`orNull`。在 Scala 代码中，它并不很有用，但对于 Java 互操作性来说非常方便，并且对`Option[AnyRef]`可用。在空`Option`的情况下，它返回`null`，否则返回`Option`的值：

```java
scala> None.orNull
res8: Null = null
```

`foreach`方法与之前我们所见的不同，因为它在`Option`的值被定义的情况下执行一个函数：

```java
scala> val opt2 = Some("I'm a non-empty option")
opt2: Some[String] = Some(I'm a non-empty option)
scala> opt2.foreach(println)
I'm a non-empty option
```

与我们之前看到的其他方法相比，它之所以特殊，是因为它不将选项视为特殊值。相反，它在语义上被表述为一个回调——“如果这个效果（已经）发生，就在它上面执行以下代码。”

这种对 `Option` 的看法提供了另一种可能性，以便我们可以与之交互——提供在空或非空选项的情况下将执行的更高阶函数。让我们详细看看这是如何工作的。

# 选项作为效果

上述方法的第一种后果是，可以在不检查其内容的情况下约束 `Option` 的可能值（如果条件不成立，则将其转换为 `None`）。以下是一个进一步过滤选项的例子，包含一个大于或小于 `10` 的数字：

```java
val moreThen10: Option[Int] = opt.filter(_ > 10)
val lessOrEqual10: Option[Int] = opt.filterNot(_ > 10)
```

还可以使用部分函数作为过滤器。这允许你在过滤和转换值的同时进行操作。例如，你可以过滤大于 `10` 的数字并将它们转换为 `String`：

```java
val moreThen20: Option[String] = opt.collect {
  case i if i > 20 => s"More then 20: $i"
}
```

从功能上讲，`collect` 方法可以看作是 `filter` 和 `map` 的组合，其中后者可以单独使用来转换非空选项的内容。例如，让我们想象一系列我们需要调用的函数来捕捉鱼：

```java
val buyBait: String => Bait = ???
val castLine: Bait => Line = ???
val hookFish: Line => Fish = ???

def goFishing(bestBaitForFish: Option[String]): Option[Fish] =
  bestBaitForFish.map(buyBait).map(castLine).map(hookFish)
```

在这里，我们是在购买一些诱饵，抛出鱼线，并在适当的时机钓到鱼。我们实现中的参数是可选的，因为我们可能不知道鱼的最佳咬钩时机是什么。

然而，这个实现存在一个问题。我们忽略了我们的函数对于给定的参数将没有结果的事实。鱼店可能已经关闭，抛出的鱼线可能断裂，鱼也可能滑走。结果是我们违反了我们之前定义的关于使用类型表达效果的规则！

让我们通过使我们的函数返回 `Option` 来修复这个问题。我们首先从 `hookFish` 开始：

```java
val hookFish: Line => Option[Fish]

def goFishingOld(bestBaitForFish: Option[String]): Option[Option[Fish]] =
  bestBaitForFish.map(buyBait).map(castLine).map(hookFish)

```

但现在我们的函数返回一个嵌套的 `Option`，这很难处理。我们可以通过使用相应的方法来展平结果来解决这个问题：

```java
def goFishingOld(bestBaitForFish: Option[String]): Option[Fish] =
  bestBaitForFish.map(buyBait).map(castLine).map(hookFish).flatten
```

现在，我们也可以让 `castLine` 返回 `Option`：

```java
val castLine: Bait => Option[Line]
val hookFish: Line => Option[Fish]

def goFishingOld(bestBaitForFish: Option[String]): Option[Fish] =
  bestBaitForFish.map(buyBait).map(castLine).map(hookFish).flatten
```

不幸的是，这个实现无法编译：

```java
error: type mismatch;
 found : FishingOptionExample.this.Line => Option[FishingOptionExample.this.Fish]
 required: Option[UserExample.this.Line] => ?
```

为了处理链式非空选项，有一个 `flatMap` 方法，它接受一个返回 `Option` 的函数，并在返回之前展平结果。使用 `flatMap`，我们可以实现我们的调用链，而无需在最后调用 `flatten`：

```java
val buyBait: String => Option[Bait]
val makeBait: String => Option[Bait]
val castLine: Bait => Option[Line]
val hookFish: Line => Option[Fish]

def goFishingOld(bestBaitForFish: Option[String]): Option[Fish]   bestBaitForFish.flatMap(buyBait).flatMap(castLine).flatMap(hookFish)
```

有 `map` 和 `flatMap` 也允许我们在 `for` 简化表达式中使用 `Option`。例如，我们可以这样重写前面的例子：

```java
def goFishing(bestBaitForFish: Option[String]): Option[Fish] =
  for {
    baitName <- bestBaitForFish
    bait <- buyBait(baitName).orElse(makeBait(baitName))
    line <- castLine(bait)
    fish <- hookFish(line)
  } yield fish
```

在这里，我们还添加了一个后备情况，以应对鱼店关闭的情况，以及当你需要手工制作诱饵时。这表明空选项也可以链式使用。`orElse` 方法解决一系列选项，直到找到第一个定义的选项，或者返回链中的最后一个 `Option`，无论其内容如何：

```java
val opt5 = opt0 orElse opt2 orElse opt3 orElse opt4
```

有一种方法可以对`Option`进行映射，并为空情况提供一个默认值。这是通过`fold`方法完成的，它接受默认值作为第一个参数列表，映射函数作为第二个参数：

```java
opt.fold("Value for an empty case")((i: Int) => s"The value is $i")
```

对于`Option`，最后两个可用方法是`toRight`和`toLeft`。它们返回下一个我们想要查看的效果的实例，即`Either`。`toRight`返回`Left`，其中包含其参数的`None`，或者返回包含`Some`值的`Right`：

```java
opt.toRight("If opt is empty, I'll be Left[String]")
```

`toLeft`做同样的事情，但返回`Either`的不同侧面：

```java
scala> val opt = Option.empty[String]
opt: Option[String] = None

scala> opt.toLeft("Nonempty opt will be Left, empty - Right[String]")
res8: Either[String,String] = Right(Nonempty opt will be Left, empty - Right[String])
```

但我们所说的这些`Left`和`Right`选项是什么？

# Either

`Either`表示一个函数可能有两种不同的结果，这些结果不能由单一类型表示的可能性。

例如，让我们想象我们有一个新的模拟系统，它取代了旧的一个。新系统非常受欢迎，因此它始终处于负载状态，因此并不总是可用。出于这个原因，旧系统被保留作为后备。不幸的是，两个系统的模拟结果格式非常不同。因此，将它们表示为`Either`是有意义的：

```java
type OldFormat
type NewFormat

def runSimulation(): Either[OldFormat, NewFormat]
```

如果这个例子让你觉得替代项的类型必须相关，那么你就有误解了。通常，结果类型会完全不相关。为了说明这一点，让我们考虑另一个例子。

在我们钓鱼的时候，有可能钓到非常不同种类的鱼。还有一种可能是拉出完全不同的东西——一个两年前游客丢失的旧靴子，或者被罪犯隐藏的潜在证据：

```java
def catchFish(): Either[Boot, Fish]
```

传统上，右边被用来表示更理想的、*正确*的结果，而左边则表示不那么理想的结果。

`Either`在 Scala 库中的简化定义如下：

```java
sealed abstract class Either[+A, +B]
final case class Left+A, +B extends Either[A, B]
final case class Right+A, +B extends Either[A, B]
```

它为左边和右边接受两个类型参数，并且有两个案例类代表这些侧面。让我们深入一点，使用与`Option`相同的方法——创建一个效果，从效果中读取，并对其抽象：

# 创建 Either

同样，在`Option`的情况下，创建`Either`实例的一个明显方法是使用相应的案例类的构造函数：

```java
scala> Right(10)
res1: scala.util.Right[Nothing,Int] = Right(10)
```

注意事项是，前面的定义让我们得到了一个左边的类型为`Nothing`的`Either`，这很可能不是我们的初衷。因此，为两边提供类型参数是很有必要的：

```java
scala> LeftString, Int
res2: scala.util.Left[String,Int] = Left(I'm left)
```

这可能有点繁琐。

再次强调，与`Option`类似，`Either`伴随对象提供了一个有用的构造函数，它接受一个谓词和两个按名称指定的构造函数，用于右边和左边：

```java
scala> val i = 100
i: Int = 100
scala> val either = Either.cond(i > 10, i, "i is greater then 10")
either: scala.util.Either[String,Int] = Right(100)
```

如果条件成立，则构建一个带有给定参数的`Right`，否则创建一个`Left`。由于两边都已定义，编译器可以正确地推断出`Either`的结果类型。

有两个辅助方法定义在 `Left` 和 `Right` 上，帮助将先前定义的侧升级为完整的 `Either`：

```java
scala> val right = Right(10)
right: scala.util.Right[Nothing,Int] = Right(10)
scala> right.withLeft[String]
res11: scala.util.Either[String,Int] = Right(10)
scala> Left(new StringBuilder).withRight[BigDecimal]
res12: scala.util.Either[StringBuilder,BigDecimal] = Left()
```

在这里，我们将 `Right[Nothing,Int]` 升级为 `Either[String,Int]`，并将 `Left` 也做同样的处理，这会产生类型为 `Either[StringBuilder,BigDecimal]` 的结果值。

# 从 Either 读取值

`Either` 与 `Option` 不同之处在于它表示两个可能的值而不是一个。因此，我们无法仅仅检查 `Either` 是否包含值。我们必须指定我们谈论的是哪一侧：

```java
if (either.isRight) println("Got right")
if (either.isLeft) println("Got left")
```

与 Option 的方法相比，它们用处不大，因为 `Either` 不提供从其中提取值的方法。在 `Either` 的情况下，模式匹配是一种可行的方法：

```java
either match {
  case Left(value) => println(s"Got Left value $value")
  case Right(value) => println(s"Got Right value $value")
}
```

断言函数也提供了与 `Option` 类似的语义，其中 `None` 由 `Left` 表示，而 `Some` 由 `Right` 表示：

```java
if (either.contains("boo")) println("Is Right and contains 'boo'")
if (either.exists(_ > 10)) println("Is Right and > 10")
if (either.forall(_ > 10)) println("Is Left or > 10")
```

将 `Right` 作为默认侧的特殊处理使得 `Either` 具有向右偏好的特性。另一个例子是 `getOrElse` 函数，它也会在 `Left` 的情况下返回提供的默认值：

```java
scala> val left = Left(new StringBuilder).withRight[BigDecimal]
res14: scala.util.Either[StringBuilder,BigDecimal] = Left()

scala> .getOrElse("Default value for the left side")
res15: String = Default value for the left side
```

向右偏好在转换为 `Option` 时表现得非常好，其中 `Some` 表示右侧，而 `None` 表示左侧，无论 `Left` 的 `value` 如何：

```java
scala> left.toOption
res17: Option[BigDecimal] = None
```

类似地，`toSeq` 将 `Right` 表示为一个只有一个元素的 `Seq`，而将 `Left` 表示为一个空 `Seq`：

```java
scala> left.toSeq
res18: Seq[BigDecimal] = List()
```

如果我们希望将 `Left` 转换为 `Right` 或反之亦然，有一个 `swap` 方法，它的唯一目的是改变两侧：

```java
scala> left.swap
res19: scala.util.Either[BigDecimal,StringBuilder] = Right()
```

这可以帮助在需要应用值的左侧应用向右偏好的方法。

# Either 作为 Effect

自然地，以效应定义的方法在 `Either` 上也是向右偏好的。例如，回调方法 `foreach` 也是这样，我们已经在 `Option` 中知道了它：

```java
scala> val left = Left("HoHoHo").withRight[BigDecimal]
left: scala.util.Either[String,BigDecimal] = Left(HoHoHo)
scala> left.foreach(println)
scala> left.swap.foreach(println)
HoHoHo
```

在前面的例子中，回调在 `left` 上没有执行，而是在我们对它调用 `swap` 后立即变为 `Right` 时被调用。过滤的定义略有不同，因为它接受一个用于过滤右侧的谓词，以及一个在谓词不成立时作为 `Left` 返回的值：

```java
scala> left.swap.filterOrElse(_.length > 10, "Sorry, too short")
res27: ... = Left(Sorry, too short)
```

`map` 和 `flatMap` 允许你在提供适当的函数时转换右侧。`flatMap` 期望函数的结果类型也是 `Either`。为了演示这一点，我们将重用我们的 `Option` 示例：

```java
val buyBait: String => Bait = ???
val makeBait: String => Bait = ???
val castLine: Bait => Line = ???
val hookFish: Line => Fish = ???
```

但这次我们将从 `bestBaitForFish` 开始，这是我们询问另一位渔夫的结果。渔夫可能心情不好，我们可能会听到他们咒骂而不是我们期望得到的提示。这两者都是 `String` 类型，但我们绝对想要区分它们：

```java
def goFishing(bestBaitForFishOrCurse: Either[String, String]): Either[String, Fish] =
  bestBaitForFishOrCurse.map(buyBait).map(castLine).map(hookFish)
```

再次，我们没有达到自己设定的标准。我们可能从商店的卖家那里得到解释，说明为什么我们不能买到我们想要的诱饵。如果我们未能制作诱饵，抛出鱼线或钓到鱼，我们也可以用一些文字来表达，这些文字我们不会放在这本书的例子中。表达函数在出错时返回这种口头反馈的可能性是有意义的：

```java
val buyBait: String => Either[String, Bait]
val makeBait: String => Either[String, Bait]
val castLine: Bait => Either[String, Line]
val hookFish: Line => Either[String, Fish]
```

现在，我们可以用`flatMap`重写使用`map`的代码。将其写成`for`推导式是有意义的：

```java
def goFishing(bestBaitForFishOrCurse: Either[String, String]): Either[String, Fish] = for {
  baitName <- bestBaitForFishOrCurse
  bait <- buyBait(baitName).fold(_ => makeBait(baitName), Right(_))
  line <- castLine(bait)
  fish <- hookFish(line)
} yield fish
```

调用将一直进行，直到最后一个成功或其中一个产生`Left`。在第二种情况下，我们遇到的第一`Left`将被作为函数调用的结果返回。

在先前的例子中，我们使用了`fold`方法，它允许我们将给定的函数应用于`Either`的一侧。在我们的用例中，我们这样做是为了忽略卖家在商店返回的任何可能的错误消息，并自己制作诱饵。如果我们成功了，我们在返回之前将诱饵包裹在`Right`中，以便我们有正确的类型对齐。

`fold`方法是无偏的，因为它平等地对待`Either`的左右两侧。

在我们之前看到的最后一个例子中，我们用`Either`表示的模型，其左侧专门用于描述其操作期间发生的失败。拥有比`String`更具体的错误类型总是有用的。特别是在涉及与 Java 集成的案例中，最合适的选择可能是将错误表示为`Exception`的子类型。事实上，这在 Scala 中是如此普遍，以至于有一个特殊的效果叫做`Try`。一个左侧类型继承自`Throwable`的`Either`可以转换为`Try`，使用相应的方法：

```java
def toTry(implicit ev: A <:< Throwable): Try[B] = this match {
  case Right(b) => Success(b)
  case Left(a)  => Failure(a)
}
```

让我们考察一下在哪些情况下`Try`比`Either`更好，并学习如何使用它。

# Try

就像`Either`代表可能结果的效应一样，`Try`表示函数抛出`Exception`的效应。在某种意义上，它只是`Either`的一个子集，但它如此常见，以至于它有自己的实现。不出所料，它的简化表示看起来相当熟悉：

```java
sealed abstract class Try[+T]
final case class Success+T extends Try[T]
final case class Failure+T extends Try[T]
```

显然，`Success`代表操作的快乐路径结果，而`Failure`用于异常条件。`Failure`的内容类型被固定为`Throwable`的子类，因此我们回到了整个 ADT 的单个类型参数，这与`Option`类似。

我们将以与`Option`和`Either`相同的方式研究`Try`——通过创建、读取和抽象其效应。

# 创建一个 Try

由于与`Either`的相似性，`Try`的定义已经熟悉，创建其实例的方法也是如此。首先，我们可以使用案例类的构造函数直接创建实例：

```java
scala> import scala.util._
import scala.util._
scala> Success("Well")
res1: scala.util.Success[String] = Success(Well)
scala> Failure(new Exception("Not so well"))
res2: scala.util.Failure[Nothing] = Failure(java.lang.Exception: Not so well)
```

`Try`背后的理念是它可以在通常抛出异常的场景中使用。因此，我们刚才提到的构造函数通常会形成以下模式：

```java
try Success(System.console().readLine()) catch {
  case err: IOError => Failure(err)
}
```

这将以`try`块的结果被包裹在`Success`中，以及`catch`异常被包裹在`Failure`中的方式结束。再次强调，`stdlib`已经在`Try`的伴生对象中实现了这种模式。`apply`方法接受一个单参数的 by-name 参数，用于`try`块，如下所示：

```java
Try(System.console().readLine())
```

然后捕获所有`NonFatal`异常。

`NonFatal`代表一类开发者能够处理的异常。它不包括像`OutOfMemoryError`、`StackOverflowError`、`LinkageError`或`InterruptedException`这样的致命错误。这些错误在程序上处理没有意义。与`NonFatal`不匹配的另一组`Throwables`是`scala.util.control.ControlThrowable`，它用于内部控制程序流程，因此也不应该在捕获异常时使用。

通常会将多行块用花括号括起来作为`Try`构造函数的参数，使其看起来像是一种语言特性：

```java
scala>val line = Try {
  val line = System.console().readLine()
  println(s"Got $line from console")
  line
}
```

这个构造函数非常常见，它涵盖了绝大多数的使用场景。

现在，让我们看看如何从一个`Try`实例中获取值。

# 从`Try`中读取值

有多种方法可以处理这个任务。可以使用类似于`Option`的`isDefined`和`isEmpty`的方法，这些方法允许进行空指针检查风格的检查：

```java
if (line.isSuccess) println(s"The line was ${line.get}")
if (line.isFailure) println(s"There was a failure")
```

显然，这种方法存在与`Option`相同的问题——如果我们忘记在提取之前检查结果是否为`Success`，调用`.get`将抛出异常：

```java
scala> Try { throw new Exception("No way") }.get
java.lang.Exception: No way
 at .$anonfun$res34$1(<console>:1)
 at scala.util.Try$.apply(Try.scala:209)
 ... 40 elided
```

为了避免在捕获异常后立即抛出异常，有一个`get`版本允许我们为`Try`是`Failure`的情况提供一个默认参数：

```java
scala> Try { throw new Exception("No way") }.getOrElse("There is a way")
res35: String = There is a way
```

不幸的是，没有像`Option`那样的接受谓词的方法。这是因为`Try`是从 Twitter 的实现中采纳的，并且首次在 Scala 2.10 版本的标准库中添加。

尽管如此，`foreach`回调仍然可用，并允许我们定义一个函数，该函数将在`Success`的值上执行：

```java
scala> line.foreach(println)
Hi, I'm the success!
```

`foreach`方法将我们的讨论带到了`Try`的效果方面。

# 将`Try`作为效果

`Try`在用谓词过滤其结果方面提供了与`Option`相同的功能。如果谓词不成立，结果将表示为`Failure[NoSuchElementException]`。以我们之前的例子中的`line`定义为例，如下所示：

```java
scala> line.filter(_.nonEmpty)
res38: scala.util.Try[String] = Success(Hi, I'm the success!)
scala> line.filter(_.isEmpty)
res39: scala.util.Try[String] = Failure(java.util.NoSuchElementException: Predicate does not hold for Hi, I'm the success!)
```

`collect`的工作方式相同，但它接受一个部分函数，并允许我们在过滤和转换`Try`的内容的同时进行：

```java
scala> line.collect { case s: String => s * 2 }
res40: scala.util.Try[String] = Success(Hi, I'm the success!Hi, I'm the success!)
scala> line.collect { case s: "Other input" => s * 10 }
res41: scala.util.Try[String] = Failure(java.util.NoSuchElementException: Predicate does not hold for Hi, I'm the success!)
```

`filter` 和 `collect` 函数是 `Success` 有偏的，`map` 和 `flatMap` 也是如此。让我们在这个情况下重新实现钓鱼示例，其中我们的参数是 `Try[String]` 类型，异常正在替换字符串作为我们 `Either` 示例中的问题描述：

```java
def goFishing(bestBaitForFishOrCurse: Try[String]): Try[Fish] =
  bestBaitForFishOrCurse.map(buyBait).map(castLine).map(hookFish)
```

操作是在 `Success` 上链式的。再次强调，我们必须修复我们函数的签名，以便它们在结果的类型中编码每个步骤的错误可能性：

```java
val buyBait: String => Try[Bait]
val makeBait: String => Try[Bait]
val castLine: Bait => Try[Line]
val hookFish: Line => Try[Fish]
```

现在，我们必须使用 `flatMap` 而不是 `map` 来对齐类型。再次强调，如果以 `for` 循环的形式表示，则更易于阅读：

```java
def goFishing(bestBaitForFishOrCurse: Try[String]): Try[Fish] = for {
  baitName <- bestBaitForFishOrCurse
  bait <- buyBait(baitName).fold(_ => makeBait(baitName), Success(_))
  line <- castLine(bait)
  fish <- hookFish(line)
} yield fish
```

这个实现几乎与我们的 `Either` 实现相同，唯一的区别在于我们现在必须将成功的调用包装到 `Success` 中，而不是 `Right` 中（我们必须使用不同的构造函数来表示效果）。

`fold` 是对 `Try` 无偏的方法之一。它接受传递 `Success` 和 `Failure` 的参数，如前述代码所示。另一个无偏方法是 `transform`，它与 `fold` 类似，但接受返回 `Try` 的函数作为参数。在某种意义上，`transform` 可以被称为 `flatFold`：

```java
scala> line.transform((l: String) => Try(println(l)), (ex: Throwable) => Try(throw ex))
Hi, I'm the success!
res45: scala.util.Try[Unit] = Success(())
```

还有一些函数是 `Failure` 有偏的。

`recover` 和 `recoverWith` 将给定的部分函数应用于 `Failure`。它们基本上是 `map` 和 `flatMaps` 的对偶，但针对异常方面：

```java
line.recover {
  case ex: NumberFormatException => Math.PI
}
line.recoverWith {
  case ex: NoSuchElementException => Try(retryAfterDelay)
}
```

`orElse` 方法允许我们以与 `None` 相同的方式链式连接失败：

```java
val result = firstTry orElse secondTry orElse failure orElse success
```

如我们所见，`Try` 与 `Option` 和 `Either` 类似，因此它能够转换为 `Option` 和 `Either[Throwable, _]` 并不令人惊讶：

```java
scala> line.toOption
res51: Option[String] = Some("Hi, I'm the success!")
scala> line.toEither
res53: scala.util.Either[Throwable,String] = Right("Hi, I'm the success!")
```

标准库中还有一个效果与前面我们看到的三个略有不同，因为它考虑了调用函数的一个更微妙方面——返回结果所需的时间。

# 未来

有时，我们调用的函数需要时间来返回计算的结果。通常，原因是有副作用，如从磁盘读取或调用慢速的远程 API。有时，操作本身需要大量的 CPU 时间才能完成。在这两种情况下，程序的主要流程都会停止，直到函数返回结果。在后一种情况下，如果计算后立即需要结果，等待结果可能是可以接受的（尽管在这种情况下也是次优的，因为它使系统无响应），但在前一种情况下是不希望的，因为这意味着我们的程序在什么也不做（好吧，等待计算机的其他子系统返回结果，但仍然与 CPU 无关）时消耗 CPU。通常，这样的长时间运行的操作是在单独的执行线程中执行的。

作为一名函数式程序员，我们希望将这些——即两个方面的内容，即调用的持续时间和代码在单独的线程中执行的事实——表达为一个效果。这正是`Future`所做的事情。更具体地说，它并不明确表示调用的持续时间，而是将其编码为二进制形式——一个操作要么运行时间较长且可能在单独的线程中运行，要么不运行。

`Future`是一个非常有趣的概念，值得单独用一整章来介绍。在这里，我们只是简要地看看它的一些方面。我们强烈建议您参考官方文档以获取更多详细信息。让我们再次应用我们无处不在的三步法，这次是为了`Future`。

# 创建一个 Future

`Future`不是编码为一个 ADT，因此我们必须使用由伴生对象提供的构造函数来构建它。因为我们将提供的代码将在单独的线程中执行，所以`Future`必须有一种方法来获取这个`Thread`。这是通过隐式地有一个`ExecutionContext`在作用域内来完成的，我们通过两个步骤导入它。首先，我们将导入作用域内的`scala.concurrent`包及其中的`ExecutionContext`：

```java
scala> import scala.concurrent._
import scala.concurrent._
scala> import ExecutionContext.Implicits.global
import ExecutionContext.Implicits.global
```

`ExecutionContext`基本上是一个`Thread`的工厂。它可以根据特定的用例进行配置。为了演示目的，我们使用全局上下文，但通常不推荐这样做。请参阅[`www.scala-lang.org/api/current/scala/concurrent/ExecutionContext.html`](https://www.scala-lang.org/api/current/scala/concurrent/ExecutionContext.html)下的 ScalaDocs 以获取更多详细信息。

在作用域内有这个上下文的情况下，我们可以通过向其构造函数提供一个按名参数来构建一个`Future`：

```java
val success = Future("Well")
val runningForever = Future {
  while (true) Thread.sleep(1000)
}
```

`Future`在创建后立即开始执行，相对于从执行器获取线程所需的时间。

有时，我们只想将手头的值包装到`Future`中，以便代码期望一个`Future`作为参数。在这种情况下，我们不需要执行上下文，因为我们不需要进行任何计算。我们可以使用帮助成功创建它的特殊构造函数之一：`failed`和从`Try`创建的`Future`：

```java
scala> val success = Future.successful("Well")
success: scala.concurrent.Future[String] = Future(Success(Well))
scala> val failure = Future.failed(new IllegalArgumentException)
failure: scala.concurrent.Future[Nothing] = Future(Failure(java.lang.IllegalArgumentException))
scala> val fromTry = Future.fromTry(Try(10 / 0))
fromTry: scala.concurrent.Future[Int] = Future(Failure(java.lang.ArithmeticException: / by zero))
```

此外，还有一个预定义的`Future[Unit]`，可以通过映射作为间接构造函数使用：

```java
scala> val runningLong = Future.unit.map { _ =>
 | while (math.random() > 0.001) Thread.sleep(1000)
 | }
runningLong: scala.concurrent.Future[Unit] = Future(<not completed>)
```

现在，既然我们在`Future`内部有一个值，让我们看看获取它的可能方法。

# 从 Future 读取值

由于`Future`不是作为 ADT 实现的，我们不能像在本章中查看的其他效果那样直接在它上进行模式匹配。

相反，我们可以使用空值检查风格：

```java
scala> if (runningLong.isCompleted) runningLong.value
res54: Any = Some(Success(()))
```

幸运的是，`value`方法返回一个`Option`，在`Future`完成之前将是`None`，因此我们可以在模式匹配中使用它：

```java
scala> runningForever.value match {
 | case Some(Success(value)) => println(s"Finished successfully with $value")
 | case Some(Failure(exception)) => println(s"Failed with $exception")
 | case None => println("Still running")
 | }
Still running
```

当然，最有用的方法不是与`Future`的值相关，而是与`Future`作为效果相关。

# 作为效果的未来

`Future`具有到目前为止我们已经从本章中了解到的所有常见功能。`foreach`允许我们定义在`Future`成功完成后执行的回调：

```java
scala> runningLong.foreach(_ => println("First callback"))
scala> runningLong.foreach(_ => println("Second callback"))
scala> Second callback
First callback
```

执行顺序没有保证，如前一个示例所示。

还有一个回调会在任何完成的功能上被调用，无论其成功与否。它接受一个函数作为参数，该函数接受`Try`作为参数：

```java
scala> runningLong.onComplete {
     | case Success(value) => println(s"Success with $value")
     | case Failure(ex) => println(s"Failure with $ex")
     | }
scala> Success with ()
```

`transform`方法也在这两种情况下应用。它有两种形式。一种接受两个函数，分别对应`Success`和`Failure`，另一种接受一个函数`Try => Try`：

```java
stringFuture.transform(_.length, ex => new Exception(ex))
stringFuture.transform {
  case Success(value) => Success(value.length)
  case Failure(ex) => Failure(new Exception(ex))
}
```

在这两种情况下，我们都会在成功的情况下将字符串转换为它的长度，在失败的情况下包装一个异常。然而，第二种变体更加灵活，因为它允许我们将成功转换为失败，反之亦然。

这种过滤也是以与其他效果相同的方式进行，即使用`filter`和`collect`方法：

```java
stringFuture.filter(_.length > 10)
stringFuture.collect {
  case s if s.length > 10 => s.toUpperCase
}
```

前者如果谓词不成立，则将`Success`转换为`Failure(NoSuchElementException)`（或者保留现有的`Failure`不变）。后者也会将`Success`的内容修改为大写。

当然，`map`和`flatMap`也是可用的。我们将让我们的用户服务使用`Future`作为效果——这次是为了表示每个动作，包括我们为鱼寻找最佳咬合名称的研究，都需要一些时间来完成：

```java
val buyBait: String => Future[Bait]
val makeBait: String => Future[Bait]
val castLine: Bait => Future[Line]
val hookFish: Line => Future[Fish]
```

这使我们来到了以下已经熟悉的实现：

```java
def goFishing(bestBaitForFish: Future[String]): Future[Fish] = for {
  baitName <- bestBaitForFish
  bait <- buyBait(baitName).fallbackTo(makeBait(baitName))
  line <- castLine(bait)
  fish <- hookFish(line)
} yield fish
```

很容易看出，除了效果类型的变化外，与之前的实现相比，唯一的区别是使用了回退方法来在`buyBait`方法调用失败的情况下提供替代方案。

关于`Future`及其双胞胎`Promise`还有很多内容要介绍。我们鼓励您查看官方文档和相关博客文章（例如 [`viktorklang.com/blog/Futures-in-Scala-2.12-part-9.html`](https://viktorklang.com/blog/Futures-in-Scala-2.12-part-9.html)），以了解一些高级用法的示例。

# 摘要

在本章中，我们讨论了标准库中定义的效果。首先是一个`Option`，它表示函数可能无法返回结果的情况。然后是`Try`，它通过在失败情况下返回错误描述来扩展可选性。接下来是`Either`，它通过允许提供任意类型作为失败路径的描述来进一步扩展`Try`的概念。最后是`Future`，它在列表中稍微独立一些，代表长时间可能在不同上下文中执行的计算。

我们注意到，这些效果有不同的构造函数，针对需要创建相应实例的情况进行了定制。相应地，它们提供了稍微不同的方式来访问容器内部存储的值。

我们注意到，将效果作为一个一等概念可以让我们不仅根据包含的值来定义方法，还可以根据效果本身来定义，这通常会导致更具有表现力的代码。

最重要的是，我们意识到许多方法，如`filter`、`collect`、`map`、`flatMap`等，从用户的角度来看是相同的，并为不同类型的效果诱导出相同的高级实现。我们通过实现四个统一示例来展示这一点，这些示例涉及在不同效果中编码的几个步骤来捕鱼。

在本书的后面部分，我们将确定导致这些相似性的基本概念。

我们还将探讨结合不同类型效果的话题，目前我们将其排除在讨论范围之外。

# 问题

1.  如何表示获取列表的第一个元素的效果，例如获取推文列表？对于给定`userId`的用户信息从数据库中获取呢？

1.  以下表达式的可能值范围是什么：`Option(scala.util.Random.nextInt(10)).fold(9)(_-1)`？

1.  以下表达式的结果会是什么？

```java
TryInt).filter(_ > 10).recover {
  case _: OutOfMemoryError => 100
}
```

1.  描述以下表达式的结果：

```java
FutureInt).filter(_ > 10).recover {
  case _: OutOfMemoryError => 100
}(20)
```

1.  给定以下函数，以下调用`either(1)`的结果会是什么？

```java
def either(i: Int): Boolean = 
  Either.cond(i > 10, i * 10, new IllegalArgumentException("Give me more")).forall(_ < 100)
```
