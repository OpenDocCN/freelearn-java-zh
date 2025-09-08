# Scala 中的基于属性的测试

单元测试是许多程序员日常活动的一部分。它是为了验证正在开发的软件的行为而进行的。基于属性的测试是单元测试的替代和补充方法。它允许描述软件的预期属性，并使用自动生成数据对这些属性进行验证，如果这些属性保持不变。

在本章中，我们将讨论在哪些情况下基于属性的测试特别有用，并探讨如何制定预期的属性以及生成测试数据。

本章将涵盖以下主题：

+   基于属性的测试概念

+   属性

+   生成器

+   缩减器

+   属性作为法则

# 技术要求

+   JDK 1.8+

+   SBT 1.2+

本章的源代码可在以下链接获取：[`github.com/PacktPublishing/Learn-Scala-Programming/tree/master/Chapter05`](https://github.com/PacktPublishing/Learn-Scala-Programming/tree/master/Chapter05).

# 基于属性的测试简介

单元测试的概念应该为任何专业开发者所熟知。单元测试通常包含多个测试用例。每个测试用例描述了程序一部分的预期行为。描述通常以以下形式制定：*对于这个代码单元在该特定状态下，我们期望给定的输入产生以下输出*。然后，开发者通过在初始状态和/或输入数据以及结果期望中引入一些偏差来复制此类测试用例，以覆盖不同的代码路径。

测试用例的规范以测试代码的形式表示，该代码依赖于测试框架。截至撰写本文时，Scala 项目有两个流行的测试框架，`ScalaTest`和`Specs2`。至少有一个应该为任何 Scala 开发者所熟悉，因此我们不会在本书中介绍它们。

相反，我们将探讨其他制定关于程序行为预期的替代方法。

# 从单元测试到属性

结果表明，测试场景（有时也称为基于示例的测试）只是定义系统预期如何工作的许多方法之一。示例仅描述了软件在特定状态下的某些属性。状态通常会影响输出，以响应提供的输入。

一般而言，除了通过示例描述的属性外，还有其他类型的属性可以表征软件，例如：

+   全称量化属性

+   条件属性

通过它们，我们可以了解关于系统的某些信息，这些信息应该适用于任何有效输入，并且可能适用于所有可能的状态。这种测试形式被称为**基于属性的测试**（**PBT**）。与单元测试案例中的具体场景相比，属性是一个抽象规范。

与单元测试框架提供结构化测试和以单元测试形式制定期望的功能相同，Scala 框架也提供了 PBT（参数化测试）的功能。

# ScalaCheck

ScalaCheck ([`www.scalacheck.org`](http://www.scalacheck.org)) 是 Scala 中自动化 PBT（参数化测试）的框架。它与 SBT 或 IntelliJ IDEA 配合得很好，并且内置了测试运行器，因此可以独立使用。它还很好地与`ScalaTest`和`specs2`集成。

`ScalaCheck`是一个外部依赖项，因此我们需要将其添加到`build.sbt`中：

```java
libraryDependencies += "org.scalacheck" %% "scalacheck" % "1.14.0" % Test
```

为了能够在 REPL（Read-Eval-Print Loop）中玩弄代码，我们需要将其添加到默认作用域中（通过移除`% Test`部分（这已经在章节的代码中完成）并使用 SBT 依赖项启动 REPL。如果您不知道如何做，请参阅附录 A，*准备环境和运行代码示例*，其中我们详细解释了它）。

现在，我们可以定义和验证我们的第一个属性：

```java
scala> import org.scalacheck.Prop.forAll
import org.scalacheck.Prop.forAll

scala> val stringLengthProp = forAll { (_: String).length >= 0 }
stringLength: org.scalacheck.Prop = Prop

scala> stringLengthProp.check
+ OK, passed 100 tests.
```

我们刚刚定义并验证了所有`Strings`都具有非负长度！有点困惑吗？让我们更仔细地看看这是如何完成的。

在第一行，我们导入了一个`forAll`属性工厂。本质上，它的目的是将函数转换为属性。

在我们的情况下，在第二行中，函数的类型是`String => Boolean`。自然，这里有一些隐式魔法在起作用。其中之一是一个隐式转换`Boolean => Property`和一个`Arbitrary[String]`，它提供了测试数据，在我们的例子中是随机字符串。

在第三行，我们调用了一个`check`方法，这是在`Prop`（ScalaCheck 使用这个名字作为“属性”的缩写）以及其他组合和执行方法中可用的，用于使用默认配置执行我们的测试。因此，它使用 100 个随机字符串作为输入数据。

现在我们对 PBT（参数化测试）的一般形式有了感觉，我们将严格地处理它的每个方面，从属性开始。

# 属性

定义属性是 PBT（参数化测试）最重要的方面。如果没有良好的属性定义，就无法正确测试系统。从测试场景到属性的过渡通常是刚开始采用 PBT 的开发者面临的最困难的部分。

因此，拥有某种系统来帮助以系统化的方式定义属性是有用的。通常，系统化某事物的第一步是分类。

# 属性类型

我们已经说过，存在普遍量化的属性和条件属性，这取决于某个属性是否始终成立或仅对所有可能输入的某个子集成立。现在，我们想要从不同的维度分解属性——根据它们的定义方式。让我们看看我们如何用一般术语描述一些操作。

# 交换律

如果操作数的顺序不重要，我们说该操作是交换的。最简单的例子就是加法和乘法。这个属性应该适用于这两个操作。在下面的代码中，我们创建了两个属性，一个用于加法，一个用于乘法，并通过比较运算结果的改变顺序来检查我们的假设是否正确：

```java
scala> forAll((a: Int, b: Int) => a + b == b + a).check
+ OK, passed 100 tests.
scala> forAll((a: Int, b: Int) => a * b == b * a).check
+ OK, passed 100 tests.
```

对于字符串，加法定义为连接，但在一般情况下不是交换的：

```java
scala> forAll((a: String, b: String) => a + b == b + a).check
! Falsified after 1 passed tests.
> ARG_0: "\u0001"
> ARG_0_ORIGINAL 
> ARG_1: "\u0000"
> ARG_1_ORIGINAL: 
```

在这个例子中，我们还可以看到`ScalaCheck`如何生成随机输入并找到一些最小失败案例。如果至少有一个字符串为空，该属性变为交换的，这可以通过以下修改之前的测试来证明，其中`b`被分配一个空字符串：

```java
scala> forAll((a: String) => a + "" == "" + a).check
+ OK, passed 100 tests.
```

这是一个字符串连接条件测试的示例。

# 结合性

结合性与交换性对于操作数相同——如果有多个操作，则只要操作数的顺序不变，操作执行的顺序并不重要。

乘法和加法的结合性属性看起来非常相似，如下例所示，其中我们有三个属性，每个属性比较两种不同运算顺序的计算结果：

```java
scala> forAll((a: Int, b: Int, c: Int) => (a + b) + c == a + (b + c)).check
+ OK, passed 100 tests.
scala> forAll((a: Int, b: Int, c: Int) => (a * b) * c == a * (b * c)).check
+ OK, passed 100 tests.
scala> forAll((a: String, b: String, c: String) => 
   (a + b) + c == a + (b + c)).check
+ OK, passed 100 tests.
```

最后一行证明了字符串连接也是结合的。

# 恒等性

某些操作的恒等性属性表明，如果操作数之一是恒等值，则操作的结果将等于另一个操作数。对于乘法，恒等值是 1；对于加法，它是 0。由于乘法和加法的交换性，恒等值可以出现在任何位置。例如，在下面的代码片段中，恒等元素作为所有这些的第一个和第二个操作数出现：

```java
scala> forAll((a: Int) => a + 0 == a && 0 + a == a).check
+ OK, passed 100 tests.
scala> forAll((a: Int) => a * 1 == a && 1 * a == a).check
+ OK, passed 100 tests.
scala> forAll((a: String) => a + "" == a && "" + a == a).check
+ OK, passed 100 tests.
```

对于字符串连接，恒等性是一个空字符串。结果证明，我们为字符串定义的条件交换性属性只是普遍恒等性属性的一种表现！

# 不变量

不变量属性是指在操作上下文中永远不会改变的属性。例如，对字符串内容进行排序或更改其大小写不应改变其长度。下一个属性证明了它对普通字符串以及大写字符串都成立：

```java
scala> forAll((a: String) => a.sorted.length == a.length).check
+ OK, passed 100 tests.
scala> forAll((a: String) => a.toUpperCase().length == a.length).check
! Falsified after 50 passed tests.
> ARG_0: 
> ARG_0_ORIGINAL: 
```

或者，至少对于`toUpperCase`来说，如果区域设置与字符串内容匹配或字符串只包含 ASCII 符号，它应该可以工作：

```java
scala> forAll(Gen.asciiStr)((a: String) => a.toUpperCase().length == a.length).check
+ OK, passed 100 tests.
```

在这里，我们有点超前，使用了`Gen.asciiStr`来生成只包含 ASCII 字符的字符串。

# 幂等性

幂等操作只改变它们的操作数一次。在初始更改之后，任何后续应用都应该保持操作数不变。对字符串内容进行排序和转换大小写是幂等操作的良例。请注意，在之前的例子中，相同的操作具有长度属性的不变量。

我们可以通过应用不同次数的操作并期望结果与第一次应用相同来证明 `toUpperCase` 和 sorted 操作是幂等的：

```java
scala> forAll((a: String) => 
  a.toUpperCase().toUpperCase() == a.toUpperCase()).check
+ OK, passed 100 tests.
scala> forAll((a: String) => a.sorted.sorted.sorted == a.sorted).check
+ OK, passed 100 tests.
```

对于乘法，自然幂等元素按定义是单位元素。但它也是一个零：

```java
scala> forAll((a: Int) => a * 0 * 0 == a * 0) .check
+ OK, passed 100 tests.
```

逻辑 `AND` 和 `OR` 分别对布尔值 `false` 和 `true` 是幂等的。

# 归纳

归纳属性反映了它们的操作数属性。它们通常用于归纳情况。

例如，任何参数的阶乘函数都应该遵守阶乘定义：

```java
scala> def factorial(n: Long): Long = if (n < 2) n else n * factorial(n-1)
factorial: (n: Long)Long

scala> forAll((a: Byte) => a > 2 ==> 
  (factorial(a) == a * factorial(a - 1))).check
+ OK, passed 100 tests.
```

当然，这是一个对于 `n > 2` 的条件属性，我们使用蕴涵运算符 `==>` 来指定（关于这个运算符的更多内容稍后介绍）。

# 对称性

对称性是一种不变性类型。它表明操作数在应用一些有序操作集之后将保持其原始形式。通常这个集合限于一对操作，甚至限于一个对称操作。

对于我们常用的实验字符串，有一个对称操作 `reverse`；对于数字，我们可以定义一对加法和减法：

```java
scala> forAll((a: String) => a.reverse.reverse == a).check
+ OK, passed 100 tests.
scala> forAll((a: Int, b: Int) => a + b - b == a).check
+ OK, passed 100 tests.
```

有可能定义另一对，以乘法和除法作为操作数（关于除以零、溢出和精度）：

对称属性通常被称为 **往返** 属性。对于单个操作，它必须对任何可逆函数成立。

# 测试预言者

严格来说，测试预言者不属于这个列表，因为它没有指定操作的内禀质量。尽管如此，它是一种有用且方便的方式来指明预期的行为。

原则很简单，特别是在重构或重写现有系统时特别有用。它使用给定的可信实现来验证新代码的行为。回到我们的字符串示例，我们可能使用 Java 的数组作为字符串内容排序的测试预言者，通过期望字符串排序和由其元素组成的数组的排序结果相同：

```java
scala> forAll { a: String =>
     | val chars = a.toCharArray
     | java.util.Arrays.sort(chars)
     | val b = String.valueOf(chars)
     | a.sorted == b
     | }.check
+ OK, passed 100 tests.
```

但是，当然，在真实的重构场景中，在数组的位置上会使用现有的实现。

# 定义属性

我们以相同的方式定义了所有不同类型的属性，使用 `forAll` 构造函数的最简洁版本和 `check` 方法。有一些方法可以自定义它们。

# 检查属性

`check()` 方法接受 `Test.Parameters`，允许配置检查执行的一些方面。最有用的是描述最小成功测试次数、并行运行的工人数、每个测试后执行的测试回调、条件测试中通过和丢弃测试之间的最大丢弃比率，以及一个初始种子，这有助于使属性评估确定性。还可以限制测试允许执行的时间。以下是一个示例，它使用了测试参数和时间限制：

```java
scala> val prop = forAll { a: String => a.nonEmpty ==> (a.reverse.reverse == a) }
prop: org.scalacheck.Prop = Prop

scala> val timed = within(10000)(prop)
timed: org.scalacheck.Prop = Prop

scala> Test.check(timed) {
     | _.withMinSuccessfulTests(100000).withWorkers(4).withMaxDiscardRatio(3)
     | }
res47: org.scalacheck.Test.Result = Result(Failed(List(),Set(Timeout)),0,0,Map(),10011)
```

在这里，我们使用了`Test.check`方法，该方法执行带有给定参数的属性并返回测试统计信息。我们可以看到，我们的测试因为超时而失败了。

除了`within`之外，还在`Prop`上定义了其他包装方法。例如，可以将属性抛出的异常转换为测试失败，可以延迟评估属性，或者收集测试报告的数据：

```java
scala> forAll { a: String =>
     |   classify(a.isEmpty, "empty string", "non-empty string") {
     |     a.sorted.length ?= a.length
     |   }
     | }.check()
+ OK, passed 100 tests.
> Collected test data:
96% non-empty string
4% empty string
```

在之前的代码中使用的`==`和`?=`之间的区别是微妙的——`==`比较两个值并返回一个布尔值，然后隐式转换为`Prop`；`?=`直接创建一个`Prop`，有时在属性组合的情况下可能很有用，正如我们将在后面看到的。

属性也可以被标记，这使得在结果中更容易找到它：

```java
scala> val prop2 = "Division by zero" |: protect(forAll((a: Int) => a / a == 1))
prop2: org.scalacheck.Prop = Prop

scala> prop2.check()
! Exception raised on property evaluation.
> Labels of failing property:
Division by zero
> ARG_0: 0
> Exception: java.lang.ArithmeticException: / by zero
$line74.$read$$iw$$iw$$iw$$iw$$iw$$iw$$iw$$iw$.$anonfun$prop2$2(<console>:2
  3)
...
```

在这里，我们也使用了`protect`方法将异常转换为测试失败。

# 组合属性

到目前为止，我们一直在谈论单个、孤立的属性。有时，确保某些属性的组合成立是有用的，甚至是有必要的。例如，我们可能想要定义一个属性，该属性仅在所有其他属性都成立时成立。或者我们可能想要有一个属性，该属性在属性集中至少有一个属性为真时为真。`Prop`上定义了组合方法，正好用于此类用例。结果是另一个属性，我们可以像之前那样进行检查：

```java
    forAll { (a: Int, b: Int, c: Int, d: String) =>
      val multiplicationLaws = all(
        "Commutativity" |: (a * b ?= b * a),
        "Associativity" |: ((a * b) * c ?= a * (b * c)),
        "Identity" |: all(a * 1 ?= a, 1 * a ?= a)
      ) :| "Multiplication laws"
      val stringProps = atLeastOne(d.isEmpty, d.nonEmpty)
      all(multiplicationLaws, stringProps)
    }.check()

+ OK, passed 100 tests.
```

这是一个属性的嵌套组合。最顶层的属性在`multiplicationLaws`和`stringProps`都成立的情况下成立。`stringProps`验证任何`String`要么为空，要么非空；同时只能有一个这样的属性为真。对于`multiplicationLaws`，所有嵌套属性都必须成立。

还有一些更具体的组合器，例如`someFailing`和`noneFailing`，它们分别在底层属性失败或没有任何属性失败的情况下成立。

# 生成器

我们已经对属性进行了详细的讨论，但还没有提到这些属性的输入数据来自哪里。让我们纠正这个遗漏，并给予生成器应有的关注。

生成器的想法来源于类型的通用概念。从某种意义上说，类型是对符合该类型可能值的规范。换句话说，类型描述了值必须遵守的规则。这些规则为我们提供了为给定类型生成数据值范围的可能性。

对于某些类型，值更多；对于其他类型，值更少。正如我们已知的那样，存在包含单个值的字面类型。对于具有`()`值的`Unit`类型，情况也是如此。对于`Boolean`类型，存在两个值：`true`和`false`。对于想象中的等价关系类型，也存在两个值——相等和非相等。按照同样的原则，完整的排序可以取三个值之一：小于、等于或大于。

定义在具有如此有限可能值集合的类型上的属性被称为 *可证明* 属性。这是因为可以尝试给定类型的所有值（如果有多个参数，则是组合），并证明程序对所有可能的输入都是正确的。

另一种类型的属性是可证伪属性。尝试所有可能的输入参数值是不可能的（或没有意义），因此只能说明正在测试的功能对于所有输入的某个子集是有效的。

为了使可证伪属性更加可信，现有的 `ScalaCheck` 生成器为 `Byte`、`Short`、`Int` 和 `Long` 类型在 `zero`、`+1`、`-1` 以及 `minValue` 和 `maxValue` 上增加了额外的权重。

让我们看看 `ScalaCheck` 中包含哪些生成器，以及我们如何使用它们为特定于我们代码的数据类型创建新的生成器。我们还将简要讨论逐渐减少已知失败案例的测试数据，即缩小。

# 现有生成器

说到现有生成器，ScalaCheck 提供了许多内置生成器，例如 `AnyVal` 的所有子类型、`Unit`、`Function0`、具有不同内容的字符和字符串（`alphaChar`、`alphaLowerChar`、`alphaNumChar`、`alphaStr`、`alphaUpperChar`、`numChar`、`numStr`）、容器、列表和映射（`containerOf`、`containerOf1`、`containerOfN`、`nonEmptyContainerOf`、`listOf`、`listOf1`、`listOfN`、`nonEmptyListOf`、`mapOf`、`mapOfN`、`nonEmptyMap`）、数字（`chooseNum`、`negNum`、`posNum`）、持续时间、`Calendar`、`BitSet`，甚至 `Test.Parameters`！

如果没有适合测试目的的生成器可用，可以通过实现一个 `Gen` 类来创建一个自定义生成器：

```java
sealed abstract class Gen[+T] extends Serializable { self =>
  ...
  def apply(p: Gen.Parameters, seed: Seed): Option[T]
  def sample: Option[T]
  ...
}
```

这是一个抽象类，它基本上就是一个接受测试参数并返回所需类型可选值的函数。

虽然部分实现了它，但手动扩展它仍然有点枯燥。因此，通常通过重用现有的生成器来实现新的生成器。作为一个练习，让我们实现一个字面类型生成器：

```java
def literalGenT <: Singleton: Gen[T] = Gen.const(t)
implicit val myGen: Arbitrary[42] = Arbitrary(literalGen(42))
val literalProp = forAll((_: 42) == 42).check
```

在第一行，我们通过将值生成委托给 `Gen.const` 来创建一个字面类型生成器工厂。这样做是安全的，因为根据定义，字面类型只包含单个值。第二行创建了一个 `implicit Arbitrary[42]`，它期望通过 `forAll` 属性在作用域内。

# 生成器的组合

虽然创建一个自定义生成器并不非常困难，但绝大多数生成器是通过组合现有实现来构建的。`Gen` 提供了一些在这种情况下非常有用的方法。一个经典的例子是使用 `map` 和 `flatMap` 方法为案例类创建生成器。

让我们用一个玩牌的例子来演示这一点：

```java
sealed trait Rank
case class SymRank(s: Char) extends Rank {
  override def toString: String = s.toString
}
case class NumRank(n: Int) extends Rank {
  override def toString: String = n.toString
}
case class Card(suit: Char, rank: Rank) {
  override def toString: String = s"$suit $rank"
}
```

首先，我们需要一些用于花色和点数的生成器，我们可以通过重用现有的 `oneOf` 和 `choose` 构造函数来创建它们：

```java
val suits = Gen.oneOf('♡', '♢', '♤', '♧')
val numbers = Gen.choose(2, 10).map(NumRank)
val symbols = Gen.oneOf('A', 'K', 'Q', 'J').map(SymRank)
```

现在，我们可以使用`for`推导式将我们的生成器组合到牌生成器中：

```java
val full: Gen[Card] = for {
  suit <- suits
  rank <- Gen.frequency((9, numbers), (4, symbols))
} yield Card(suit, rank)
```

我们还使用`Gen.frequency`以确保我们的组合生成器产生的数字和符号有适当的分布。

使用`suchThat`组合器很容易将这个生成器改为只为牌组生成牌：

```java
val piquet: Gen[Card] = full.suchThat {
  case Card(_, _: SymRank) => true
  case Card(_, NumRank(n)) => n > 5
}
```

我们可以通过使用`Prop.collect`方法来检查我们的生成器是否产生可信的值：

```java
scala> forAll(piquet) { card =>
     | Prop.collect(card)(true)
     | }.check
+ OK, passed 100 tests.
> Collected test data:
8% ♡ J
6% ♢ 7
6% ♡ 10
... (couple of lines more)
scala> forAll(full) { card =>
     | Prop.collect(card)(true)
     | }.check
+ OK, passed 100 tests.
> Collected test data:
6% ♡ 3
5% ♢ 3
... (a lot more lines)
```

当然，也可以使用容器生成器方法之一从牌堆中生成一些牌：

```java
val handOfCards: Gen[List[Card]] = Gen.listOfN(6, piquet)
```

然后像以前一样使用它：

```java
scala> forAll(handOfCards) { hand: Seq[Card] =>
     | Prop.collect(hand.mkString(","))(true)
     | }.check
! Gave up after only 58 passed tests. 501 tests were discarded.
> Collected test data:
2% ♤ 8,♤ 10,♤ 8,♤ 7,♡ Q,♢ 8
```

哦，我们的手中有多张重复的牌。结果是，我们需要使用容器生成器的更一般形式，它接受容器类型和元素类型的类型参数：

```java
val handOfCards = Gen.containerOfNSet, Card
scala> forAll(handOfCards) { hand =>
     | Prop.collect(hand.mkString(","))(true)
     | }.check
! Gave up after only 75 passed tests. 501 tests were discarded.
> Collected test data:
1% ♡ A,♤ J,♡ K,♢ 6,♧ K,♧ A
1% ♤ 9,♧ A,♧ 8,♧ 9
```

这样更好，但现在看起来重复的元素似乎消失了，所以我们仍然没有预期的行为。此外，还有一个明显的问题——许多测试被丢弃。这是因为我们的`piquet`生成器是在过滤更通用的`full`生成器的输出定义的。`ScalaCheck`注意到有太多测试不符合有效输入的资格，因此提前放弃。

让我们修复我们的`piquet`生成器和一个缺失牌的问题。对于第一个问题，我们将使用与`full`生成器相同的方法。我们只需更改用于排名的数字：

```java
val piquetNumbers = Gen.choose(6, 10).map(NumRank)

val piquet: Gen[Card] = for {
  suit <- suits
  rank <- Gen.frequency((5, piquetNumbers), (4, symbols))
} yield Card(suit, rank)
```

请注意，频率是如何相对于可能值的变化而变化的。

为了修复第二个问题，我们将使用`retryUntil`组合器反复生成牌组，直到它达到预期的尺寸：

```java
val handOfCards = Gen.containerOfNSet, Card.retryUntil(_.size == 6)

scala> forAll(handOfCards) { hand =>
     | Prop.collect(hand.mkString(","))(true)
     | }.check
+ OK, passed 100 tests.
> Collected test data:
1% ♤ 9,♢ 9,♧ 9,♢ Q,♧ J,♤ 10
...
```

现在，我们的手牌生成正如预期的那样。

当然，还有更多有用的组合方法，可以用来创建其他复杂的生成器。请参阅文档([`github.com/rickynils/scalacheck/blob/master/doc/UserGuide.md`](https://github.com/rickynils/scalacheck/blob/master/doc/UserGuide.md))或源代码以获取更多详细信息。

# 缩减器

我们已经探讨了 PBT 的两个基石——属性和生成器。在我们认为自己完成之前，还有一个方面我们应该看看。

在 PBT 中，测试数据来自生成器，它有点随机。考虑到这个事实，我们可能会预期很难找出为什么测试失败。考虑以下示例：

```java
scala> forAllNoShrink { num: Int =>
     | num < 42
     | }.check
! Falsified after 0 passed tests.
> ARG_0: 2008612603
```

在这里，我们可以看到我们的属性被数字`2008612603`所否定，这个数字可以说是没有太大用处。对于一个`Int`来说，这几乎是显而易见的，但考虑一个包含许多元素的列表和为这些元素制定的属性的情况：

```java
scala> forAllNoShrink(Gen.listOfN(1000, Arbitrary.arbString.arbitrary)) {
     | _.forall(_.length < 10)
     | }.check
! Falsified after 10 passed tests.
> ARG_0: List  

  ",
... // a lot of similar lines
```

显然，在这个测试中找出哪个 1,000 个字符串长度错误几乎是不可能的。

在这个时候，新的组件开始发挥作用：`Shrink`。收缩器的作用是找到使属性不成立的最小测试数据。在前两个例子中，我们使用了`forAllNoShrink`属性构造器，因此没有激活收缩器。如果我们将定义更改为正常的`forAll`，结果将如下所示：

```java
scala> forAll(Gen.listOfN(1000, Arbitrary.arbString.arbitrary)) {
     | _.forall(_.length < 10)
     | }.check
! Falsified after 10 passed tests.
> ARG_0: List("")
> ARG_0_ORIGINAL: // a long list as before
```

在这里，我们可以看到，使我们的属性为假的极小列表是包含一个空字符串的列表。原始的失败输入以`ARG_0_ORIGINAL`的形式展示，其长度和复杂性与我们之前看到的相似。

`Shrink`实例作为隐式参数传递，因此我们可以召唤一个来查看它们是如何工作的。我们将使用我们的`Int`属性失败值来做这件事：

```java
val intShrink: Shrink[Int] = implicitly[Shrink[Int]]
scala> intShrink.shrink(2008612603).toList
res23: List[Int] = List(1004306301, -1004306301, 502153150, -502153150, 251076575, -251076575, 125538287, -125538287, 62769143, -62769143, 31384571, -31384571, 15692285, -15692285, 7846142, -7846142, 3923071, -3923071, 1961535, -1961535, 980767, -980767, 490383, -490383, 245191, -245191, 122595, -122595, 61297, -61297, 30648, -30648, 15324, -15324, 7662, -7662, 3831, -3831, 1915, -1915, 957, -957, 478, -478, 239, -239, 119, -119, 59, -59, 29, -29, 14, -14, 7, -7, 3, -3, 1, -1, 0)
```

`shrink`方法生成一个值流，我们通过将其转换为列表来评估它。很容易看出模式——`Shrink`产生的值相对于`0`（零）的**中心**值对称，从初始的失败值开始，然后每次都除以二，直到它们收敛到零。这基本上就是它对数字的实现方式，包括硬编码的`+-two`、`+-one`和`zero`值。

很容易看出，`Shrink`生成的数字将取决于初始的失败参数。这就是为什么对于第一个属性，返回的值每次都会不同的原因：

```java
scala> forAll { (_: Int) < 42 }.check
! Falsified after 0 passed tests.
> ARG_0: 47
> ARG_0_ORIGINAL: 800692446

scala> forAll { (_: Int) < 42 }.check
! Falsified after 0 passed tests.
> ARG_0: 54
> ARG_0_ORIGINAL: 908148321

scala> forAll { (_: Int) < 42 }.check
! Falsified after 2 passed tests.
> ARG_0: 57
> ARG_0_ORIGINAL: 969910515

scala> forAll { (_: Int) < 42 }.check
! Falsified after 6 passed tests.
> ARG_0: 44
> ARG_0_ORIGINAL: 745869268
```

如我们所见，产生的失败值取决于原始的失败值，永远不会是`43`，但有时它非常接近。

当存在一些不成立的属性时，收缩器是必不可少的，尤其是如果输入数据具有显著的大小。

# 摘要

基于属性的测试是传统单元测试和行为驱动开发的一种补充技术。它允许人们以抽象规范的形式描述程序属性，并以规则的形式描述用于生成测试数据的规则。

正确生成的数据包括边缘情况，这些情况在基于示例的测试中通常被忽略，并允许更高的代码覆盖率。

`ScalaCheck`是一个基于 Scala 的属性测试框架。它有三个主要组件——属性、生成器和收缩器。

全称量化属性必须适用于程序任何状态的任何测试数据。条件属性是为数据的一个子集或系统的特定状态定义的。

`ScalaCheck`提供了大量标准类型的生成器。创建自定义类型的生成器的最佳方式是通过使用它们上定义的合适方法组合现有生成器。

可选收缩器的作用是减少失败属性的测试数据集，帮助识别最小失败测试用例。

有几个扩展库可供使用，允许用户生成任意的案例类和 ADT（[scalacheck-shapeless](https://github.com/alexarchambault/scalacheck-shapeless)），cats 类型类实例（[cats-check](https://github.com/non/cats-check)），以及其他常见情况（[scalacheck-toolbox](https://github.com/47deg/scalacheck-toolbox)）。

现在，我们已经准备好开始我们的功能性编程概念之旅，这部分内容将在本书的下一部分中介绍。我们将从检查标准库中的一些类型开始，这些类型被称为效果，例如`Option`、`Try`、`Either`和`Future`。

# 问题

1.  定义一个用于排序列表的不变量性质

1.  定义一个用于排序列表的幂等性质

1.  定义一个用于排序列表的归纳性质

1.  定义一个生成器，用于`List[Lists[Int]]`，使得嵌套列表的元素都是正数

1.  定义一个生成器，用于`Map[UUID, () => String]`
