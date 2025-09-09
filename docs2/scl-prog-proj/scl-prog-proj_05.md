# 类型类

在本章中，你将了解建立在 Scala 之上的概念。本章中的概念将是抽象的，并且理解它们可能需要一些集中注意力；如果你一开始没有完全理解，请不要感到沮丧。每个单独的部分相对容易理解，但当你把它们全部放在一起时，事情可能会变得复杂。

我们将专注于类型类，并为每个类型类提供一个定义。它们将随后通过一个示例来说明类型类如何在典型程序中发挥作用。由于这些概念可能难以理解，我们还建议一些可选练习，这些练习可以加强你的理解。你不必完成它们就能理解本章的其余部分。练习的解决方案可在 GitHub 上找到。

这里展示的大多数类型类都来自一个名为 Cats 的库，由 Typelevel 创建。

在本章中，我们将介绍以下类型类：

+   scala.math.Ordering

+   org.scalactic.Equality

+   cats.Semigroup

+   cats.Monoid

+   cats.Functor

+   cats.Apply

+   cats.Applicative

+   cats.Monad

# 理解类型类

**类型类**代表了一组具有共同行为的类型。类型类对于类型来说，就像类对于对象一样。与传统的类一样，类型类可以定义方法。这些方法可以在属于类型类的所有类型上调用。

类型类是在 Haskell 编程语言中引入的。然而，多亏了隐式的力量，我们也可以在 Scala 中使用它们。在 Scala 中，类型类不是内置的语言结构（就像在 Haskell 中那样），因此我们需要编写一些样板代码来定义它们。

在 Scala 中，我们通过使用`trait`来声明类型类，它接受一个类型参数。例如，让我们定义一个允许将两个对象合并为一个的`Combine`类型类，如下所示：

```java
trait Combine[A] {
  def combine(x: A, y: A): A
}
```

然后，我们可以为`Combine`定义两个类型类实例，如下所示：

+   一个用于`Int`，它将两个参数相加

+   一个用于`String`，它将它们连接起来

代码定义如下：

```java
object Combine {
  def applyA: Combine[A] = combineA

  implicit val combineInt: Combine[Int] = new Combine[Int] {
    override def combine(x: Int, y: Int): Int = x + y
  }

  implicit val combineString: Combine[String] = new Combine[String] {
    override def combine(x: String, y: String) = x + y
  }
}
```

首先，我们为我们的类型类定义一个`apply`构造函数，它仅返回隐式参数。然后，我们通过使用`implicit val`声明类型类实例。这样，编译器将能够通过使用我们在上一节中看到的隐式解析规则自动发现它们。

现在，我们可以实例化和使用我们的类型类，如下所示：

```java
Combine[Int].combine(1, 2)
// res0: Int = 3
Combine[String].combine("Hello", " type class")
// res1: String = Hello type class

```

当我们调用`Combine[Int]`时，实际上我们调用的是`Combine.apply[Int]`。由于我们的`apply`函数接受类型为`Combine[Int]`的隐式参数，编译器会尝试找到它。隐式解析规则之一是在参数类型的伴生对象中搜索。

如我们在`Combine`的伴生对象中声明了`combineInt`，编译器将其用作`Combine.apply`的参数。

一旦我们获得了`Combine`类型类的实例，我们就可以调用它的方法，`combine`。当我们用两个`Int`调用它时，它会将它们相加，当我们用两个`String`调用它时，它会将它们连接起来。

到目前为止，一切顺利；但是，这有点繁琐。如果我们可以像调用`Int`或`String`上的方法一样调用`combine`，那就更实用了。

正如你在前一节中看到的，我们可以在`Combine`对象内部定义一个隐式类，如下所示：

```java
object Combine {
...
  implicit class CombineOpsA(implicit combineA: Combine[A]) {
    def combine(y: A): A = combineA.combine(x, y)
  }
}
```

这个隐式类允许我们在任何具有类型类实例`Combine[A]`的类型`A`上调用`combine`。因此，我们现在可以像下面这样调用`combine`：

```java
2.combine(3)
// res2: Int = 5
"abc" combine "def"
// res3: String = abcdef
```

它可能看起来并不令人印象深刻；你可能会说我们只是给`+`方法起了另一个名字。使用类型类的关键好处是，我们可以使`combine`方法对任何其他类型都可用，而无需更改它。

在传统的面向对象编程中，你必须更改所有类并使它们扩展一个特质，这并不总是可能的。

类型类允许我们在需要时将一个类型转换为另一个类型（在我们的例子中，从`Int`到`Combine`）。这就是我们所说的**特定多态**。

另一个关键好处是，通过使用`implicit def`，我们可以为参数化类型生成类型类实例，例如`Option`或`Vector`。我们只需将它们添加到`Combine`伴生对象中即可：

```java
object Combine {
...
  implicit def combineOptionA
  : Combine[Option[A]] = new Combine[Option[A]] {
    override def combine(optX: Option[A], optY: Option[A]): Option[A] =
      for {
        x <- optX
        y <- optY
      } yield combineA.combine(x, y)
  }
}
```

只要我们有一个类型参数`A`的隐式`Combine[A]`，我们的函数`combineOption`就可以生成`Combine[Option[A]]`。

这种模式非常强大；它允许我们通过使用其他类型类实例来生成类型类实例！编译器将根据其返回类型自动找到正确的生成器。

这非常常见，以至于 Scala 提供了一些语法糖来简化此类函数的定义。我们可以将`combineOption`重写如下：

```java
implicit def combineOption[A: Combine]: Combine[Option[A]] = new Combine[Option[A]] {
  override def combine(optX: Option[A], optY: Option[A]): Option[A] =
    for {
      x <- optX
      y <- optY
    } yield Combine[A].combine(x, y)
}
```

具有一个声明了类型参数`A: MyTypeClass`的函数，等同于有一个类型`MyTypeClass[A]`的隐式参数。

然而，当我们使用这种语法时，我们没有为那个隐式指定名称；我们只是在当前作用域中拥有它。拥有它就足够调用任何接受类型`MyTypeClass[A]`的隐式参数的其他函数。

正是因为我们可以在前面的例子中调用`Combine.apply[A]`。有了这个`combineOption`定义，我们现在也可以在`Option`上调用`combine`：

```java
Option(3).combine(Option(4))
// res4: Option[Int] = Some(7)
Option(3) combine Option.empty
// res5: Option[Int] = None
Option("Hello ") combine Option(" world")
// res6: Option[String] = Some(Hello  world)
```

练习：为`Combine[Vector[A]]`定义一个类型类实例，该实例可以连接两个向量。

练习：为`Combine[(A, B)]`定义一个类型类实例，该实例将两个元组的第一个和第二个元素组合起来。例如，`(1, "Hello ") combine (2, "World")`应该返回`(3, "Hello World")`。

# 类型类配方

总结一下，如果我们想创建一个类型类，我们必须执行以下步骤：

1.  创建`trait`，`MyTypeClass[A]`，它接受一个参数化类型`A`。它代表类型类接口。

1.  在 `MyTypeClass` 的伴生对象中定义一个 `apply[A]` 函数，以便于类型类实例的实例化。

1.  为所有期望的类型（`Int`、`String`、`Option` 等）提供 `trait` 的隐式实例。

1.  定义一个到 `Ops` 类的隐式转换，这样我们就可以像在目标类型中声明一样调用类型类的函数（就像您之前看到的，使用 `2.combine(3)`）。

这些定义可以像上一节那样手动编写。或者，您可以使用 **simulacrum** 生成其中的一些。这有两个好处：减少样板代码并确保一致性。您可以在这里查看它：[`github.com/mpilquist/simulacrum`](https://github.com/mpilquist/simulacrum)。

练习：使用 simulacrum 定义 `Combine` 类型类。

# 常见类型类

在一个典型的项目中，您不会创建很多自己的类型类。由于类型类捕获了跨多个类型的常见行为，因此很可能有人已经实现了与您所需类似的一个类型类，位于库中。通常，重用 SDK（或第三方库）中定义的类型类比尝试定义自己的类型类更有效率。

通常，这些库为 SDK 类型（`String`、`Int`、`Option` 等）定义了类型类的预定义实例。您通常会重用这些实例来为您自己的类型推导实例。

在本节中，我们将介绍您最有可能遇到的类型类，以及如何使用它们来解决日常编程挑战。

# scala.math.Ordering

`Ordering` 是一个 SDK 类型类，它表示对类型实例进行排序的策略。最常见的用例是按如下方式对集合的元素进行排序：

```java
Vector(1,3,2).sorted
// res0: scala.collection.immutable.Vector[Int] = Vector(1, 2, 3)
```

如果您查看 `sorted` 的声明，您将看到它接受一个隐式参数，即 `Ordering[B]`：

```java
def sortedB >: A: Repr
```

当我们在 `Vector[Int]` 上调用 `sorted` 时，编译器找到了一个类型为 `Ordering[Int]` 的隐式值，并将其传递给函数。这个隐式值在 `Ordering` 的伴生对象中找到，它还定义了 `String`、`Long`、`Option`、`Tuples` 等实例。

我们可以定义 `LocalDate` 类型类的实例，这样我们就可以比较日期，或者更容易地对它们进行排序：

```java
import java.time.LocalDate
implicit val dateOrdering: Ordering[LocalDate] =   
  Ordering.fromLessThanLocalDate
import Ordering.Implicits._
LocalDate.of(2018, 5, 18) < LocalDate.of(2017, 1, 1)
// res1: Boolean = false
Vector(
  LocalDate.of(2018, 5, 18), 
  LocalDate.of(2018, 6, 1)
).sorted(dateOrdering.reverse)
// res2: Vector[LocalDate] = Vector(2018-06-01, 2018-05-18)
```

在作用域内有 `Ordering[LocalDate]` 和 `Ordering.Implicits`，我们可以使用 `<` 操作符来比较日期。"Ordering.Implicits" 还定义了其他有用的改进方法，例如 `<`、`>`、`<=`、`>=`、`max` 和 `min`。

我们也可以通过使用反转的 `Ordering[LocalDate]` 来轻松地对 `Vector[LocalDate]` 进行逆序排序。这比先按升序排序然后反转向量更有效。

练习：定义一个 `Ordering[Person]`，它可以按从老到小的顺序对 `case class Person(name: String, age: Int)` 的实例进行排序。您将需要使用 `Ordering.by`。

# org.scalactic.Equality

`Equality` 类型类被 ScalaTest 单元测试使用，每当你在断言中写入时，例如以下内容：

```java
actual shouldBe expected
actual should === (expected)
```

大多数时候，你不必担心它；默认实例通过使用 `equals` 方法比较类型。然而，每当比较双精度值或包含双精度属性的类时，提供不同的 `Equality` 实例就变得必要了。

考虑以下单元测试：

```java
class EqualitySpec extends WordSpec with Matchers with TypeCheckedTripleEquals{
  "Equality" should {
    "allow to compare two Doubles with a tolerance" in {
      1.6 + 1.8 should === (3.4)
    }
  }
}
```

这个测试应该通过，对吧？错了！尝试运行它；断言失败，如下所示：

```java
3.4000000000000004 did not equal 3.4
```

这是因为 `Double` 是一个 IEEE754 双精度浮点数。

因此，一些十进制数字无法用 `Double` 准确表示。为了使我们的测试通过，我们需要提供一个 `Equality[Double]` 实例，该实例将在两个数字的绝对差小于某个容差时返回 true：

```java
class EqualitySpec extends WordSpec with Matchers with 
 TypeCheckedTripleEquals{
  implicit val doubleEquality: Equality[Double] = 
  TolerantNumerics.tolerantDoubleEquality(0.0001)
  "Equality" should {...}
```

再次运行测试；它将通过。

练习：实现一个类型类推导，`Equality[Vector[A]]`，这样我们就可以使用容差比较两个 `Vector[Double]` 实例。

下面的声明是你必须在练习中实现的等价实例：

```java
class EqualitySpec extends WordSpec with Matchers with TypeCheckedTripleEquals{
  implicit val doubleEquality: Equality[Double] = 
  TolerantNumerics.tolerantDoubleEquality(0.0001)
  implicit def vectorEqualityA:   
  Equality[Vector[A]] = ???
```

一旦你完成了练习，测试应该会通过，如下所示：

```java
"Equality" should {
  (...)
  "allow to compare two Vector[Double] with a tolerance" in {
    Vector(1.6 + 1.8, 0.0051) should === (Vector(3.4, 0.0052))
  }
}
```

关于浮点数编码的更多信息，请参阅 [`www.javaworld.com/article/2077257/learn-java/floating-point-arithmetic.html`](https://www.javaworld.com/article/2077257/learn-java/floating-point-arithmetic.html)。

# cats.Semigroup

在类型类介绍中，我们定义了一个 `Combine` 类型类。结果发现，这个类型类已经在 Cats 库中定义了。它的名字是 `Semigroup`；这个名字来源于这种代数结构的数学表示。

在 IntelliJ 中打开它，看看它是如何定义的：

```java
/**
 * A semigroup is any set `A` with an associative operation (`combine`).
 */
trait Semigroup[@sp(Int, Long, Float, Double) A] extends Any with Serializable {
  /**
   * Associative operation taking which combines two values.
   */
  def combine(x: A, y: A): A
(...)
}
```

`@sp` 注解是一种优化，用于避免原始类型的装箱/拆箱。除此之外，`Semigroup` 的定义与我们的 `Combine` 类型类相同。

# 定律

文档提到一个非常重要的观点：类型类的实例必须实现一个结合函数的关联性。

这意味着它们必须验证以下定律：

```java
a combine (b combine c) = (a combine b) combine c
```

Cats 中的大多数类型类都有自己的特定定律。库保证它定义的类型类实例验证这个定律。然而，如果你实现了类型类的自己的实例，那么验证它没有违反任何定律是你的责任。

类型类实例的用户期望它验证类型类的所有定律；这是类型类合约的一部分。

类型类合约是类型类特质加上定律。

当一个类型类验证了某些定律时，你可以更容易地推理泛型代码，并且可以自信地应用一些变换。

例如，如果你知道结合性定律已经得到验证，你可以并行评估 `a combine b combine c combine d`：一个线程可以评估 `(a combine b)`，而另一个线程可以评估 `(c combine d)`。

Cats 提供了一个`laws`模块来帮助检查你的类型类实例。你可以通过编写单元测试来检查你的类型类是否遵守等价律。本书中不会详细介绍这一点；如果你对更多信息感兴趣，可以访问[`typelevel.org/cats/typeclasses/lawtesting.html`](https://typelevel.org/cats/typeclasses/lawtesting.html)。

# 使用示例

Cats 提供了`Semigroup`类型类的几个推导。它还在`SemigroupOps`中声明了一个`|+|`运算符，它是`combine`的别名。

以下是一些示例：

```java
import cats.implicits._
1 |+| 2
// res0: Int = 3
"Hello " |+| "World !"
// res1: String = Hello World !
(1, 2, "Hello ") |+| (2, 4, "World !")
// res2: (Int, Int, String) = (3,6,Hello World !)
```

多亏了`import cats.implicits._`引入的内置推导，我们可以组合`Int`、`String`、`Tuple2`和`Tuple3`。

我们也可以组合一些参数化类型，例如`Option`和`Vector`：

```java
Vector(1, 2) |+| Vector(3, 4)
// res3: Vector[Int] = Vector(1, 2, 3, 4)
Option(1) |+| Option(2)
// res4: Option[Int] = Some(3)
Option(1) |+|  None |+| Option(2)
// res5: Option[Int] = Some(3)
```

对于`Option`，请注意，当我们将其与非空`Option`组合时，空`Option`将被忽略。

在第三章，“处理错误”中，你看到了`Option`是处理错误的一种方式，并且你了解到每当需要错误消息时，你可以使用`Either`而不是`Option`。

那么，如果我们对`Either`调用`combine`会发生什么？考虑以下代码：

```java
1.asRight |+| 2.asRight
// res6: Either[B,Int] = Right(3)
1.asRight[String] |+| 2.asRight |+| "error".asLeft
// res7: Either[String,Int] = Left(error)
"error1".asLeft[Int] |+| "error2".asLeft
// res8: Either[String,Int] = Left(error1)
```

`|+|`（或`combine`）函数返回类型为`Left`的第一个参数。如果所有组合的值都是类型`Right`，它们的值将组合并放入`Right`中。

在第一行，所有组合的值都是类型`Right`，因此结果是`Right(3)`，因为`3`是`1`和`2`应用`combine`操作的结果。

在第二行，第一个组合值为类型`Left`的`Left("error")`，因此结果也是`Left("error")`。

在第三行，第一个组合值为类型`Left`的`Left("error1")`，因此结果是`Left("error1")`。

练习：使用`|+|`运算符组合`ValidatedNel[String, Int]`的实例。

当你组合几个无效值时会发生什么？

# cats.Monoid

`Monoid`是一个带有额外`empty`函数的`Semigroup`，也称为**单位元素**。

以下是从 Cats 中这个特质的定义中提取的内容：

```java
trait Monoid[@sp(Int, Long, Float, Double) A] extends Any with Semigroup[A] {
  /**
   * Return the identity element for this monoid.
   */
  def empty: A
```

`Monoid`特质扩展了`Semigroup`特质。因此，它具有`Semigroup`的所有方法，加上这个额外的`empty`方法。我们已经看到了`Semigroup`的`combine`操作在不同类型上的几个示例。

让我们看看当我们对相同类型调用`empty`时会发生什么，如下所示：

```java
import cats.implicits._
import cats.kernel.Monoid
Monoid[Int].empty
// res0: Int = 0
Monoid[String].empty
// res1: String = 
Monoid[Option[Double]].empty
// res2: Option[Double] = None
Monoid[Vector[Int]].empty
// res2: Vector[Int] = Vector()
Monoid[Either[String, Int]].empty
// res4: Either[String,Int] = Right(0)
```

对于每种类型，`empty`元素相当自然：对于`Int`是`0`，对于`Option`是`None`，等等。

# 等价律

我们可以理解为什么这个`empty`函数被称为单位元素；如果你将任何对象与单位元素组合，它将返回相同的对象，如下所示：

```java
(3 |+| Monoid[Int].empty) == 3
("Hello identity" |+| Monoid[String].empty) == "Hello identity"
(Option(3) |+| Monoid[Option[Int]].empty) == Option(3)
```

该属性通过以下等价律正式定义：

+   左恒等性：对于所有类型为`A`的`x`，`Monoid[A].empty |+| x == x`

+   右恒等性：对于所有类型为`A`的`x`，`x |+| Monoid[A].empty == x`

# 使用示例

这很有趣——我们如何在日常程序中使用 `Monoid`？最引人注目的用例是折叠数据结构。当你有 `Monoid[A]` 时，将 `Vector[A]` 的所有元素组合起来获得一个 `A` 是非常简单的。例如，我们可以得到 `Vector[Int]` 中所有元素的总和，如下所示：

```java
Vector(1,2,3).combineAll
// res8: Int = 6
```

这相当于在 `Vector` 上调用 `foldLeft`：

```java
Vector(1, 2, 3).foldLeft(0) { case (acc, i) => acc + i }
```

事实上，`foldLeft` 接受两个参数，如下所示：

+   一个起始值，我们可以传递单例的空值

+   一个函数，我们可以传递单例的 `combine` 函数

Cats 还提供了一个 `foldMap` 函数，它允许你在折叠之前将集合的元素转换为 `Monoid`：

```java
Vector("1", "2", "3").foldMap(s => (s, s.toInt))
// res10: (String, Int) = (123,6)
```

练习：实现一个 `Monoid[Int]` 的实例，用于乘以 `Vector[Int]` 中所有元素。

练习：使用 `foldMap` 计算平均值的 `Vector[Double]`。

提示：`foldMap` 调用的返回类型应该是 `(Int, Double)`。

# 高阶类型

在探索其他类型类之前，熟悉 **高阶类型** 和 **arity** 的概念会有所帮助。

你已经熟悉了值和函数。一个值是一个字面量或对象，例如 `1`，`false`，或 `"hello world"`。

# Arity

函数接受一个或多个值作为参数并返回另一个值。

函数的 **arity** 是它接受的参数数量。

例如：

+   一个 **一元**（arity 0）函数不接受任何参数

+   一个 **一元**（arity 1）函数只接受一个参数

+   一个 **二元**（arity 2）函数接受两个参数

一个 **类型构造器** 是接受参数的类型。它被称为类型构造器，因为它在我们传递一个具体类型给它时构建一个具体类型。例如，`Option[A]` 是一个类型构造器。当我们传递一个具体类型 `Int` 给它时，我们获得一个具体类型 `Option[Int]`。

由于类型构造器可以接受 0 到 n 个参数，arity 的概念也适用于这里：

+   一个 **零元**类型不接受任何参数。它是一个具体类型——`Int`，`Boolean` 以及更多

+   一元类型接受一个参数——`Option[A]`，`Vector[A]`

+   二元类型接受两个参数——`Either[L, R]`，`Map[K, V]`

+   三元类型接受三个参数——`Tuple3[A, B, C]`

# 高阶函数

函数的 **order** 是函数箭头的嵌套深度：

+   零级——值，例如，`1`，`false` 或 `"hello"`

+   第一级——函数 `A => B`，例如 `def addOne: Int => Int = x => x + 1`

+   第二级——高阶函数 `A => B => C`，例如：

```java
def map: Vector[Int] => (Int => Int) => Vector[Int] = 
                                        xs => f => xs.map(f)
```

+   第三级——高阶函数 `A => B => C => D`

任何阶数严格大于 `1` 的函数都是高阶函数。

# 高阶类型

结果表明，与类型和 **类型类** 存在类似的概念：

+   普通类型，如 `Int` 或 `Boolean` 的类型是 `*`

+   一元类型构造器的类型是 `* -> *`，例如，`Option` 或 `List`

+   二元类型构造器的类型是 `(*, *) -> *`，例如，`Either` 或 `Map`

类似于函数和级别，我们可以通过类型箭头 `->` 的数量来对类型进行排序：

+   第 0 级（`*`）：普通类型，如 `Int`、`Boolean` 或 `String`

+   第 1 级（`* -> *` 或 `(*, *) -> *`）：类型构造器 `Option`、`Vector`、`Map` 以及更多

+   第 2 级（`(* -> *) -> *`）：高阶类型 `Functor`、`Monad` 以及更多

**高阶**类型是一个具有严格多于一个箭头 `->` 的类型构造器。在下一节中，我们将探讨使用高阶类型定义的类型类。

# cats.Functor

`Functor` 是一个**一元**的**高阶**类型。它接受一个一元类型参数 `F[_]`。换句话说，其类型参数必须是一个具有类型参数的类型；例如，`Option[A]`、`Vector[A]`、`Future[A]` 等等。它声明了一个 `map` 方法，可以转换 `F` 内部的元素。以下是 `cats.Functor` 的简化定义：

```java
trait Functor[F[_]] {
  def mapA, B(f: A => B): F[B]
```

这应该很熟悉。我们已经看到了 SDK 中定义了执行相同操作的 `map` 函数的几个类：`Vector`、`Option` 等等。因此，你可能会想知道为什么你需要使用 `Functor[Option]` 或 `Functor[Vector]` 的实例；它们只会定义一个已经可用的 `map` 函数。

在 Cats 中拥有这种 `Functor` 抽象的一个优点是它让我们能够编写更通用的函数：

```java
import cats.Functor
import cats.implicits._
def addOne[F[_] : Functor](fa: F[Int]): F[Int] = fa.map(_ + 1)
```

此函数为任何具有 `Functor` 类型类实例的 `F[Int]` 添加 `1`。我对 `F` 的唯一了解是它有一个 `map` 操作。因此，此函数将适用于许多参数化类型，例如 `Option` 或 `Vector`：

```java
addOne(Vector(1, 2, 3))
// res0: Vector[Int] = Vector(2, 3, 4)
addOne(Option(1))
// res1: Option[Int] = Some(2)
addOne(1.asRight)
// res2: Either[Nothing,Int] = Right(2)
```

我们的功能函数 `addOne` 应用最小功率原则；在给定解决方案的选择中，它选择能够解决你问题的最弱解决方案。

我们使用一个更通用的参数类型，因此更弱（它只有一个 `map` 函数）。这使得我们的函数更可重用，更易读，更易测试：

+   **更可重用**：相同的函数可以用 `Option`、`Vector`、`List` 或任何具有 `Functor` 类型类实例的东西使用。

+   **更易读**：当你阅读 `addOne` 的签名时，你知道它唯一能做的就是转换 `F` 内部的元素。例如，它不能重新排列元素的顺序，也不能删除某些元素。这是由 `Functor` 法律保证的。因此，你不需要阅读其实现来确保它不会陷入任何麻烦。

+   **更容易测试**：你可以使用具有 `Functor` 实例的最简单类型来测试函数，即 `cats.Id`。代码覆盖率将相同。一个简单的测试可以是，例如，`addOnecats.Id == 2`。

# 法律

为了成为 `Functor`，`Functor` 实例的 `map` 函数必须满足两个法律。在本章的其余部分，我们将使用等式来定义我们的法律：`left_expression == right_expression`。这些等式必须对任何指定的类型和实例都成立。

给定一个具有 `Functor[F]` 实例的类型 `F`，对于任何类型 `A` 和任何实例 `fa: F[A]`，必须满足以下等式：

+   **恒等性保持**：`fa.map(identity) == fa`。恒等函数总是返回其参数。使用此函数进行映射不应改变 `fa`。

+   **组合保持**：对于任何函数 `f` 和 `g`，`fa.map(f).map(g) == fa.map(f andThen g)`。依次映射 `f` 和 `g` 与使用这些函数的组合映射相同。这个定律允许我们优化代码。当我们发现自己多次在大型向量上调用 `map` 时，我们知道我们可以用单个调用替换所有的 `map` 调用。

练习：编写一个违反恒等性保持定律的 `Functor[Vector]` 实例。

练习：编写一个违反组合保持定律的 `Functor[Vector]` 实例。

# 使用示例

每当你有一个 `A => B` 的函数时，只要你有一个作用域内的 `Functor[F]` 实例，你就可以将其提升为一个函数 `F[A] => F[B]`，如下所示：

```java
def square(x: Double): Double = x * x
def squareVector: Vector[Double] => Vector[Double] =
  Functor[Vector].lift(square)
  squareVector(Vector(1, 2, 3))
// res0: Vector[Double] = Vector(1.0, 4.0, 9.0)
def squareOption: Option[Double] => Option[Double] =
  Functor[Option].lift(square)
  squareOption(Some(3))
// res1: Option[Double] = Some(9.0)
```

另一个实用的函数是 `fproduct`，它将值与函数应用的结果组合成元组：

```java
Vector("Functors", "are", "great").fproduct(_.length).toMap 
//res2: Map[String,Int] = Map(Functors -> 8, are -> 3, great -> 5)
```

我们通过使用 `fproduct` 创建了 `Vector[(String, Int)]`，然后将其转换为 `Map`。我们得到了一个以单词为键的 `Map`，其关联的值是该单词的字符数。

# cats.Apply

`Apply` 是 `Functor` 的子类。它声明了一个额外的 `ap` 函数。以下是 `cats.Apply` 的简化定义：

```java
trait Apply[F[_]] extends Functor[F] {
  def apA, B(fa: F[A]): F[B]
  /** Alias for [[ap]]. */
  @inline final def <*>A, B(fa: F[A]): F[B] =
  ap(ff)(fa)
```

这个签名意味着对于给定的上下文 `F`，如果我们有一个在 `F` 内部的 `A => B` 函数，我们可以将其应用于另一个 `F` 内部的 `A` 以获得 `F[B]`。我们还可以使用 `ap` 别名操作符 `<*>`。让我们用不同的 `F` 上下文尝试一下，如下所示：

```java
import cats.implicits._

OptionString => String.ap(Some("Apply"))
// res0: Option[String] = Some(Hello Apply)
OptionString => String <*> None
// res1: Option[String] = None
Option.empty[String => String] <*> Some("Apply")
// res2: Option[String] = None
```

对于 `F = Option`，`ap` 仅在两个参数都不为空时才返回一个非空 `Option`。

对于 `F = Vector`，我们得到以下结果：

```java
def addOne: Int => Int = _ + 1 
def multByTwo: Int => Int = _ * 2 
Vector(addOne, multByTwo) <*> Vector(1, 2, 3) 
// res3: Vector[Int] = Vector(2, 3, 4, 2, 4, 6)
```

在 `Vector` 的情况下，`ap` 对第一个 `Vector` 的每个元素进行操作，并将其应用于第二个 `Vector` 的每个元素。因此，我们获得了将每个函数应用于每个元素的所有组合。

练习：使用 `ap` 与 `Future`。

# 法则

与其他 Cats 类型类一样，`Apply` 实例必须遵守某些定律。

给定一个具有 `Apply[F]` 实例的类型 `F`，对于所有类型 `A`，给定一个实例 `fa: F[A]`，必须验证以下等式：

1.  **乘积结合律**：对于所有 `fb: F[B]` 和 `fc: F[C]`，以下适用：

```java
(fa product (fb product fc)) ==
  ((fa product fb) product fc).map { 
     case ((a, b), c) => (a, (b,c))
    }
```

我们可以改变括号，从而改变求值顺序，而不改变结果。

1.  **`ap` 函数组合**：对于所有类型 `B` 和 `C`，给定实例 `fab: F[A => B]` 和 `fbc: F[B => C]`，以下适用：

```java
(fbc <*> (fab <*> fa)) == ((fbc.map(_.compose[A] _) <*> fab) <*> fa)
```

这与我们在 `Functor` 部分看到的函数组合定律相似：`fa.map(f).map(g) == fa.map(f andThen g)`。

不要在阅读这个定律时迷失方向；`<*>` 函数是从右到左应用的，用于函数的 `andThen` 是 `Functor` 的 `.compose[A]`。

练习：验证 `F = Option` 的 `product` 结合性。你可以为 `fa`、`fc` 和 `fc` 使用特定的值，例如 `val (fa, fb, fc) = (Option(1), Option(2), Option(3))`。

练习：验证 `ap` 函数的 `F = Option` 上的组合。和之前一样，你可以为 `fa`、`fab` 和 `fbc` 使用特定的值。

# 使用示例

这都很好，但在实践中，我很少在上下文中放置函数。我发现 `Apply` 中的 `map2` 函数更有用。它通过使用 `product` 和 `map` 在 `Apply` 中定义。`product` 通过使用 `ap` 和 `map` 来定义自己：

```java
trait Apply[F[_]] extends Functor[F] … {
  (...)
  def map2A, B, Z(f: (A, B) => Z): F[Z] =
    map(product(fa, fb))(f.tupled)
    override def productA, B: F[(A, B)] =
    ap(map(fa)(a => (b: B) => (a, b)))(fb)
```

`map2` 对象允许在 `F` 上下文中应用一个函数到两个值。这可以用来在 `F` 内部组合两个值，如下所示：

```java
def parseIntO(s: String): Option[Int] = Either.catchNonFatal(s.toInt).toOption
parseIntO("6").map2(parseIntO("2"))(_ / _)
// res4: Option[Int] = Some(3)
parseIntO("abc").map2(parseIntO("def"))(_ / _)
// res5: Option[Int] = None
```

在前面的例子中，对于 `F = Option`，`map2` 允许我们在两个值都不为空时调用函数 `/`。

Cats 还为 `F = Either[E, ?]` 提供了一个 `Apply` 实例。因此，我们可以将 `parseIntOpt` 的签名更改为返回 `Either[Throwable, Int]`，其余的代码将保持不变：

```java
def parseIntE(s: String): Either[Throwable, Int] = Either.catchNonFatal(s.toInt)
parseIntE("6").map2(parseIntE("2"))(_ / _)
// res6: Either[Throwable,Int] = Right(3)
parseIntE("abc").map2(parseIntE("3"))(_ / _)
// res7: Either[Throwable,Int] = Left(java.lang.NumberFormatException: For input string: "abc")
```

这个 `map2` 函数对两个元素工作得很好，但如果我们有三个、四个或 *N* 个元素怎么办？`Apply` 没有定义 `map3` 或 `map4` 函数，但幸运的是，Cats 在元组上定义了一个 `mapN` 函数：

```java
(parseIntE("1"), parseIntE("2"), parseIntE("3")).mapN( (a,b,c) => a + b + c)
// res8: Either[Throwable,Int] = Right(6)
```

在 第三章，*处理错误* 中，我们看到当我们要在第一个错误处停止时使用 `Either`。这就是我们在上一个例子中看到的情况：错误提到 `"abc"` 无法解析，但没有提到 `"def"`。

应用我们刚刚学到的知识，如果我们想累积所有错误，我们可以使用 `ValidatedNel`：

```java
import cats.data.ValidatedNel
def parseIntV(s: String): ValidatedNel[Throwable, Int] = Validated.catchNonFatal(s.toInt).toValidatedNel
(parseIntV("abc"), parseIntV("def"), parseIntV("3")).mapN( (a,b,c) => a + b + c)
// res9: ValidatedNel[Throwable,Int] = Invalid(NonEmptyList(
// java.lang.NumberFormatException: For input string: "abc", 
// java.lang.NumberFormatException: For input string: "def")
```

练习：使用 `mapN` 与 `Future[Int]`。这允许你并行运行多个计算，并在它们完成时处理它们的结果。

练习：使用 `mapN` 与 `Vector[Int]`。

# cats.Applicative

`Applicative` 是 `Apply` 的一个子类。它声明了一个额外的函数，称为 `pure`：

```java
@typeclass trait Applicative[F[_]] extends Apply[F] { 
  def pureA: F[A]
}
```

将任何类型的 `A` 放入 `F` 上下文的 `pure` 函数。一个具有 `Applicative[F]` 实例并且遵守相关法则的类型 `F` 被称为 **Applicative Functor**。

让我们尝试使用不同的 `F` 上下文来使用这个新的 `pure` 函数，如下所示：

```java
import cats.Applicative
import cats.data.{Validated, ValidatedNel}
import cats.implicits._

Applicative[Option].pure(1)
// res0: Option[Int] = Some(1)
3.pure[Option]
// res1: Option[Int] = Some(3)
type Result[A] = ValidatedNel[Throwable, A]
Applicative[Result].pure("hi pure")
// res2: Result[String] = Valid(hi pure)
"hi pure".pure[Result]
// res3: Result[String] = Valid(hi pure)
```

在大多数情况下，`pure` 等同于 `apply` 构造函数。我们可以通过使用在 `Applicative` 特质上声明的函数来调用它，或者通过在任意类型上调用 `.pure[F]` 来调用它。

# 法则

如你所预期，`Applicative` 必须遵守某些法则。

给定一个具有 `Applicative[F]` 实例的类型 `F`，对于所有类型 `A`，给定一个实例 `fa: F[A]`，以下等式必须得到验证：

1.  `Applicative` 的恒等式如下：

    ```java
    ((identity[A] _).pure[F] <*> fa)  ==  fa
    ```

当我们使用 `pure` 将 `identity` 函数放入 `F` 上下文并调用 `fa` 上的 `<*>` 时，它不会改变 `fa`。这类似于 `Functor` 中的恒等律。

1.  给定实例 `fab: F[A => B]` 和 `fbc: F[B => C]` 的 `Applicative` 组合如下：

    ```java
    (fbc <*> (fab <*> fa)) == ((fbc.map(_.compose[A] _) <*> fab) <*> fa)
    ```

它与`Functor`中的组合保持相似。通过使用`compose`，我们可以改变`<*>`表达式周围的括号，而不会改变结果。

1.  `Applicative`同态如下所示：

    ```java
    Applicative[F].pure(f) <*> Applicative[F].pure(a) == Applicative[F].pure(f(a))
    ```

当我们调用`pure(f)`然后`<*>`，它等同于应用`f`然后调用`pure`。

+   给定实例`fab: F[A => B]`的`Applicative`交换如下所示：

    ```java
    fab <*> Applicative[F].pure(a) == 
      Applicative[F].pure((f: A => B) => f(a)) <*> fab
    ```

如果我们在等式的左侧包裹`a`，或者在等式的右侧包裹`f(a)`，我们可以翻转`<*>`的`fab`参数。

作为练习，我鼓励你打开 Cats 源代码中的`cats.laws.ApplicativeLaws`类。还有其他一些定律要发现，以及所有测试的实现。

# 使用示例

在关于`Apply`的*cats.Apply*部分，你看到我们可以通过使用`mapN`在`F`上下文中组合许多值。但是，如果我们想要组合的值在一个集合中而不是一个元组中怎么办？

在那种情况下，我们可以使用`Traverse`类型类。Cats 为许多集合类型提供了这个类型类的实例，例如`List`、`Vector`和`SortedMap`。

以下是对`Traverse`的简化定义：

```java
@typeclass trait Traverse[F[_]] extends Functor[F] with Foldable[F] with UnorderedTraverse[F] { self =>
  def traverse[G[_]: Applicative, A, B](fa: F[A])(f: A => G[B]): 
  G[F[B]]
  (...)
}
```

这个签名意味着我可以使用一个集合，`fa: F[A]`（例如，`Vector[String]`），和一个函数来调用`A`并返回`G[B]`，其中`G`是`Applicative Functor`（例如，`Option[Int]`）。它将在`F`中的所有值上运行`f`函数，并以`G`上下文返回`F[B]`。

让我们通过具体的例子来看一下它的实际应用，如下所示：

```java
import cats.implicits._
def parseIntO(s: String): Option[Int] =
  Either.catchNonFatal(s.toInt).toOption
Vector("1", "2" , "3").traverse(parseIntO)
// res5: Option[Vector[Int]] = Some(Vector(1, 2, 3))
Vector("1", "boom" , "3").traverse(parseIntO)
// res6: Option[Vector[Int]] = None
```

我们可以安全地将`Vector[String]`解析为返回`Option[Vector[Int]]`。如果任何值无法解析，结果将是`None`。在这个例子中，我们使用`Traverse`，其中`F = Vector`、`G = Option`、`A = String`和`B = Int`。

如果我们想要保留一些解析错误的详细信息，我们可以使用`G = ValidatedNel`，如下所示：

```java
import cats.data.{Validated, ValidatedNel}
def parseIntV(s: String): ValidatedNel[Throwable, Int] = Validated.catchNonFatal(s.toInt).toValidatedNel

Vector("1", "2" , "3").traverse(parseIntV)
// res7: ValidatedNel[Throwable, Vector[Int]] = Valid(Vector(1, 2, 3))
Vector("1", "boom" , "crash").traverse(parseIntV)
// res8: ValidatedNel[Throwable, Vector[Int]] = 
// Invalid(NonEmptyList(
//   NumberFormatException: For input string: "boom", 
//   NumberFormatException: For input string: "crash"))
```

练习：使用`Traverse`，其中`G = Future`。这将允许你并行运行集合中每个元素的函数。

另一个常见的用例是使用`sequence`将结构`F[G[A]]`翻转成`F[G[A]]`。

以下是在`cats.Traverse`特质中`sequence`的定义：

```java
def sequence[G[_]: Applicative, A](fga: F[G[A]]): G[F[A]] =
  traverse(fga)(ga => ga)
```

我们可以看到`sequence`实际上是通过`traverse`实现的。

以下是一个例子，其中`F = Vector`和`G = Option`：

```java
val vecOpt: Vector[Option[Int]] = Vector(Option(1), Option(2), Option(3))
val optVec: Option[Vector[Int]] = vecOpt.sequence
// optVec: Option[Vector[Int]] = Some(Vector(1, 2, 3))
```

下面是另一个例子，其中`F = List`和`G = Future`：

```java
import scala.concurrent._
import ExecutionContext.Implicits.global
import duration.Duration

val vecFut: Vector[Future[Int]] = Vector(Future(1), Future(2), Future(3))
val futVec: Future[Vector[Int]] = vecFut.sequence

Await.result(futVec, Duration.Inf)
// res9: Vector[Int] = Vector(1, 2, 3)
```

`sequence`的调用返回`Future`，它只会在`vecFut`中的三个 future 完成时完成。

# cats.Monad

`Monad`是`Applicative`的子类。它声明了一个额外的函数，`flatMap`，如下所示：

```java
@typeclass trait Monad[F[_]] extends FlatMap[F] with Applicative[F]@typeclass trait FlatMap[F[_]] extends Apply[F] {
  def flatMapA, B(f: A => F[B]): F[B]
}
```

这个签名告诉我们，为了产生`F[B]`，`flatMap`必须以某种方式从`fa: F[A]`中提取`A`，然后调用函数`f`。

之前，你看到 `Applicative` 和 `mapN` 允许我们并行处理多个 `F[A]` 值，并将它们组合成一个单一的 `F[B]`。`Monad` 添加的是处理 `F[]` 值的顺序能力：`flatMap` 必须首先处理 `F` 效应，然后调用 `f` 函数。

与 `Functor` 和 `map` 类似，SDK 中的许多类已经有一个 `flatMap` 方法，例如 `Option`、`Vector`、`Future` 等。拥有这种 `Monad` 抽象的一个优点是我们可以编写接受 `Monad` 的函数，这样它就可以与不同类型重用。

# 法律

给定一个具有 `Monad[F]` 实例的类型 `F`，对于所有类型 `A` 给定一个实例 `fa: F[A]`，以下等式必须得到验证：

+   **所有来自超特质的法律**：参见 `Applicative`、`Apply` 和 `Functor`。

+   **FlatMap 结合律**：给定两个类型 `B` 和 `C`，以及两个函数 `f: A => F[B]` 和 `g: B => F[C]`，以下适用：

```java
((fa flatMap f) flatMap g) == (fa flatMap(f(_) flatMap g))
```

+   **左单位元**：给定一个类型 `B`，一个值 `a: A`，和一个函数 `f: A => F[B]`，以下适用：

```java
Monad[F].pure(a).flatMap(f) == f(a)
```

在 `F` 上下文中引入一个值并调用 `flatMap f` 应该提供与直接调用函数 `F` 相同的结果。

+   **右单位元**：

```java
fa.flatMap(Monad[F].pure) == fa
```

当我们调用 `flatMap` 和 `pure` 时，`fa` 对象不应该改变。

# 使用示例

假设你正在构建一个用于管理商店库存的程序。以下是一个简化版的 API 来管理项目：

```java
import cats.{Id, Monad}
import cats.implicits._

case class Item(id: Int, label: String, price: Double, category: String)

trait ItemApi[F[_]] {
 def findAllItems: F[Vector[Item]]
 def saveItem(item: Item): F[Unit]
}
```

该 API 使用一个 `F` 上下文进行参数化。这允许你有不同的 API 实现，如下所示：

+   在你的单元测试中，你会使用 `class TestItemApi extends ItemApi[cats.Id]`。如果你寻找 `Id` 的定义，你会找到 `type Id[A] = A`。这意味着这个 `TestItemApi` 可以直接在 `findAllItems` 中返回 `Vector[Item]`，在 `saveItem` 中返回 `Unit`。

+   在你的生产代码中，你需要访问数据库，或者调用远程 REST 服务。这些操作需要时间并且可能会失败；因此，你需要使用类似 `F = Future` 或 `F = cats.effects.IO` 的东西。例如，你可以定义 `class DbItemApi extends ItemApi[Future]`。

SDK 中的 `Future` 类有一些问题并违反了一些法律。我鼓励你使用更好的抽象，例如 `cats.effects.IO` ([`typelevel.org/cats-effect/datatypes/io.html`](https://typelevel.org/cats-effect/datatypes/io.html)) 或 `monix.eval.Task` ([`monix.io/docs/2x/eval/task.html`](https://monix.io/docs/2x/eval/task.html))。

配置了这个 API 后，我们可以实现一些业务逻辑。以下是一个应用折扣到所有项目的函数实现：

```java
def startSalesSeason[F[_] : Monad](api: ItemApi[F]): F[Unit] = {
  for {
    items <- api.findAllItems
    _ <- items.traverse { item =>
      val discount = if (item.category == "shoes") 0.80 else 0.70
      val discountedItem = item.copy(price = item.price * discount)
      api.saveItem(discountedItem)
    }
  } yield ()
}
```

`F[_]: Monad`类型参数约束意味着我们可以用任何`ItemApi[F]`调用`startSalesSeason`，只要`F`有一个`Monad`实例。这个隐式参数的存在允许我们在`F[A]`实例上调用`map`和`flatMap`。由于编译器将`for`推导式转换为`map`/`flatMap`的组合，我们可以使用`for`推导式使我们的函数更易读。在*cats.Applicative*部分关于`Applicative`，你看到我们可以对`Vector`调用`traverse`，只要函数返回具有`Applicative[F]`实例的`F`。由于`Monad`扩展了`Applicative`，我们可以使用`traverse`遍历项目并保存每个项目。

如果你眯着眼睛看这段代码，它看起来非常类似于命令式实现的模样。我们设法编写了一个纯函数，同时保持了可读性。这种技术的优势在于，我们可以轻松地对`startSalesSeason`的逻辑进行单元测试，使用`F = Id`，而无需处理`Futures`。在生产代码中，相同的代码可以使用`F = Future`，甚至`F = Future[Either[Exception, ?]]`，并在多线程中优雅地处理流程。

练习：实现`TestItemApi`，它扩展了`ItemApi[Id]`。你可以使用可变的`Map`来存储项目。之后，为`startSalesSeason`编写单元测试。然后，实现一个生产版本的 API，它扩展了`ItemApi[Future]`。

这种方法被称为**无标签最终编码**。您可以在[`www.beyondthelines.net/programming/introduction-to-tagless-final/`](https://www.beyondthelines.net/programming/introduction-to-tagless-final/)找到更多关于此模式的信息。

# 摘要

我们在本章中介绍了一些具有挑战性的概念。类型类也用于其他函数式编程语言，如 Haskell。

为了方便起见，以下表格总结了我们在本章中列举的类型类：

| 名称 | 方法 | 法律 | 示例 |
| --- | --- | --- | --- |
| `Semigroup` | `def combine(` `x: A, y: A) : A` | 结合律 |

```java
Option(1) &#124;+&#124;  None &#124;+&#124; Option(2)
// res5: Option[Int] = Some(3)
```

|

| `Monoid` | `def empty: A` | 标识符 |
| --- | --- | --- |

```java
Vector(1,2,3).combineAll
// res8: Int = 6
```

```java
Vector("1", "2", "3").foldMap(s => (s,s.toInt))
// res10: (String, Int) = (123,6)
```

|

| `Functor` | `def map[A, B]` `(fa: F[A])`

`(f: A => B): F[B]` | 标识符，可组合性 |

```java
def square(x: Double): Double = x * x
def squareVector: 
  Vector[Double] => Vector[Double] =
    Functor[Vector].lift(square)
squareVector(Vector(1, 2, 3))
// res0: Vector[Double] = Vector(1.0, 4.0, 9.0)
```

```java
Vector("Functors", "are", "great")
  .fproduct(_.length)
  .toMap
// res2: Map[String,Int] = Map(Functors -> 8, //are -> 3, great -> 5)
```

|

| `Apply` | `def ap[A, B]` `(ff: F[A => B])`

`(fa: F[A]): F[B]`

别名 `<*>` | 结合律，可组合性 |

```java
Option[String => String]
    ("Hello " + _).ap(Some("Apply"))
// res0: Option[String] = Some(Hello Apply)
OptionString => String <*> None
// res1: Option[String] = None
```

```java
def addOne: Int => Int = _ + 1
def multByTwo: Int => Int = _ * 2
Vector(addOne, multByTwo) <*> Vector(1, 2, 3)
// res3: Vector[Int] = Vector(2, 3, 4, 2, 4, 6)
```

|

| `Applicative` | `def pure[A]` `(x: A): F[A]` | 标识符，可组合性，

同态，

交换 |

```java
import cats.data.{Validated, ValidatedNel}
def parseIntV(s: String): ValidatedNel[Throwable, Int] = Validated.catchNonFatal(s.toInt).toValidatedNel

Vector("1", "2" , "3").traverse(parseIntV)
// res7: ValidatedNel[Throwable, Vector[Int]] = Valid(Vector(1, 2, 3))
Vector("1", "boom" , "crash")
  .traverse(parseIntV)
// res8: ValidatedNel[Throwable, Vector[Int]] = 
// Invalid(NonEmptyList(
//   NumberFormatException: For input string: "boom", 
//   NumberFormatException: For input string: "crash"))
```

|

| `Monad` | `def flatMap[A, B]` `(fa: F[A])`

`(f: A => F[B]): F[B]` | 标识符，结合律，

可组合性，

同态，

交换 |

```java
import cats.{Id, Monad}
import cats.implicits._
case class Item(id: Int,
  label: String,
  price: Double,
  category: String)
trait ItemApi[F[_]] {
  def findAllItems: F[Vector[Item]]
  def saveItem(item: Item): F[Unit]
}
def startSalesSeason[F[_] : Monad](
  api: ItemApi[F]): F[Unit] = {
  for {
    items <- api.findAllItems
    _ <- items.traverse { item =>
    val discount = if (item.category ==
      "shoes") 0.80 else 0.70
    val discountedItem = item.copy(price =
      item.price * discount)
    api.saveItem(discountedItem)
  }} yield ()
}
```

|

如果你想要更详细地了解类型类，我鼓励你查看 Cats 文档[`typelevel.org/cats`](https://typelevel.org/cats)。我也发现阅读 SDK 中的源代码和测试，或者在 Cats 等库中阅读源代码和测试非常有帮助。

Cats 是 Typelevel 创新计划的主要库，但在这个旗下还有许多更多令人着迷的项目，如[`typelevel.org/projects/`](https://typelevel.org/projects/)所示。

在下一章中，我们将使用在 Scala 社区中流行的框架来实现一个购物网站的购物车。
