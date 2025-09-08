# 熟悉基本单子

在上一章中，我们了解了 Functors，这是一个抽象，它给`map`方法赋予了标准库中定义的效果。回顾第六章，《探索内置效果》，这里仍然有些缺失——`flatMap`方法的来源，所有标准效果都有这个来源。

在本章中，我们终于将遇到单子的概念，这是定义`flatMap`的结构。为了深入了解这个函数，我们将实现四个不同的单子。

到本章结束时，你将熟悉以下主题：

+   抽象单子及其属性

+   为标准效果实现单子

+   以下基本单子的实现和应用：

    +   Id

    +   状态

    +   读者

    +   作者

# 技术要求

在我们开始之前，请确保你已经安装了以下内容：

+   JDK 1.8+

+   SBT 1.2+

本章的源代码可在我们的 GitHub 仓库中找到，网址为[`github.com/PacktPublishing/Learn-Scala-Programming/tree/master/Chapter09`](https://github.com/PacktPublishing/Learn-Scala-Programming/tree/master/Chapter09)。

# 单子简介

我们用了三章的篇幅才到达这样一个时刻，即我们准备讨论`flatMap`方法的起源，这与我们在第六章，《探索内置效果》中看到的效果有关。之所以需要这么多章节，并不是因为主题的复杂性，而是与之相关的抽象家族的丰富性。

在这个介绍之后，一个可疑的读者可能会失望地想——好吧，现在他们将要使用他们常用的伎俩，说有一个`flatMap`、`flattener`或`flatMapative`的抽象，从空中拉出一些法则，然后认为自己已经完成了。这些骗子！

好吧，从技术上讲，我们并没有作弊，因为我们并没有从任何地方拉取东西。相反，我们从范畴论中获取这些概念，这是我们之前提到的数学分支。我们的抽象必须遵守的规则是由数学家定义的。这种方法的优点在于——一旦我们能够证明我们的实现遵循所需的法则，我们就可以利用范畴论已经证明的一切来为我们所用。一个例子就是将两个应用项合并为一个，就像我们在上一章讨论的例子中那样。

回到我们的`flatMap`方法，这里还有一些神秘之处。抽象的名称是`Monad`，它由两个方法定义，`flatMap`和`unit`：

```java
import scala.language.higherKinds
trait Monad[F[_]] {
  def unitA: F[A]
  def flatMapA, B(f: A => F[B]): F[B]
}
```

单子定义于类型`F`的容器。`unit`方法将它的参数提升到`F`的上下文中；`flatMap`在某种意义上类似于普通的`map`方法，它将`f`应用于`a`。`flatMap`与普通`map`的不同之处以及它之所以特殊的地方在于它能够*折叠*或*扁平化*两层`F`，将其合并为一层。这可以从`a`和`f`的类型签名中轻易看出。

将`F[F[A]]`扁平化为`F[A]`的可能性是为什么通常用一组不同的方法来表示 monads；也就是说，`map`和`flatten`：

```java
trait Monad[F[_]] {
  def unitA: F[A]
  def mapA, B(f: A => B): F[B]
  def flattenA: F[A]
}
```

`flatten`方法正是我们刚才所说的那样——它允许我们将 Fs 的堆栈减少到单个`F`。有时，`flatten`方法也被称为`join`。我们很快就会看到为什么这个名字是有意义的。

显然，我们与 monads 的情况与与 applicatives 的情况相同——我们可以选择原始函数集，并以原始函数的形式实现其余的功能。例如，`flatMap`与`map`和`flatten`的组合一样强大，我们可以选择这些组合中的任何一个。

让我们坚持最初的定义，并以`flatMap`为其他方法实现。这就是`map`将看起来像：

```java
def mapA, B(f: A => B): F[B] = 
  flatMap(a)(a => unit(f(a)))
```

我们在这里所做的是基本上用函数`f`进行映射，并将结果以`F`上下文的形式提升，正如类型要求的那样。

你能记得那个以具有`map`方法的抽象体为特征的名称吗？对，这就是 functor。我们能够仅用`flatMap`来定义每个`Monad`的`map`的能力证明了每个`Monad`都是`Functor`。正因为如此，我们可以声明`Monad extends Functor`。

`flatten`方法的定义同样简单明了：

```java
def flattenA: F[A] = flatMap(a)(identity)
```

使用`identity`函数，我们正在使用`flatMap`的一部分力量将两层`F`转换为一层，而实际上并没有对`a`做任何事情。

我们能否更进一步，将已经存在于`F`上下文中的函数应用到`a`上？结果是我们可以，而且我们知道这个方法——这就是在`Applicative`中定义的`apply`：

```java
def applyA, B(f: F[A => B]): F[B] =
  flatMap(f) { fab: (A => B) => map(a) { a: A => fab(a) }}
```

在这里，我们假装`f`是一个值，所以我们只需要将`a`表示为一个可以将此值应用于其上的函数。`fab`函数接受一个名为`A => B`的函数，我们用它来`map`原始的`a`，返回`B`，由于`map`的应用，它变成了`F[B]`。

`apply`函数也是针对每个 monad 以`flatMap`（以及从`flatMap`派生出的`map`）来定义的。这证明了每个`Monad`都是`Applicative`。因此，我们可以将`Monad`的定义改为以下形式：

```java
trait Monad[F[_]] extends ch08.Applicative[F] {
  def flatMapA, B(f: A => F[B]): F[B]

  def flattenA: F[A] = flatMap(a)(identity)

  override def unitA: F[A]

  override def mapA, B(f: A => B): F[B] = 
    flatMap(a)(a => unit(f(a)))

  override def applyA, B(f: F[A => B]): F[B] =
    flatMap(f) { fab: (A => B) => map(a) { a: A => fab(a) }}
}
```

我们可以看到，`flatMap`方法仅对`Monad`可用，而不是对`Applicative`。这导致了有趣的后果，我们将在本章后面讨论。

现在，在切换到实现特定 monads 的实例之前，让我们首先讨论 monadic laws。

幸运的是，只有两个，而且它们都与我们在上一章中讨论的 functor laws 非常相似；即*恒等性*和*结合性*定律。

标识律指出，应用 `flatMap` 和 `unit` 应该返回原始参数。根据应用的顺序，存在左和右的标识律。我们将像往常一样使用 `ScalaCheck` 属性（以下代码片段未显示隐式参数；请参阅附带的代码以获取完整定义）来正式表示它们：

```java
val leftIdentity = forAll { as: M[A] =>
  M.flatMap(as)(M.unit(_)) == as
}
```

左标识律规定，通过将参数提升到单子的上下文中使用 `flatMap` 的结果应该等于原始参数。

右标识律稍微复杂一些：

```java
val rightIdentity = forAll { (a: A, f: A => M[B]) =>
  M.flatMap(M.unit(a))(f) == f(a)
}
```

基本上，规则是将 `a` 提升到上下文中，然后使用某个函数 `f` 进行扁平映射，应该产生与直接应用此函数到 `a` 相同的结果。

现在，我们只需要将这两个属性组合成一个单一的标识属性。我们需要相当多的不同 `implicit Arbitrary` 参数来生成输入数据，包括 `A, M[A]` 和 `A => M[B]`，但属性本身应该不会令人惊讶：

```java
import org.scalacheck._
import org.scalacheck.Prop._

def id[A, B, M[_]](implicit M: Monad[M],
                   arbFA: Arbitrary[M[A]],
                   arbFB: Arbitrary[M[B]],
                   arbA: Arbitrary[A],
                   cogenA: Cogen[A]): Prop = {
  val leftIdentity = forAll { as: M[A] =>
    M.flatMap(as)(M.unit(_)) == as
  }
  val rightIdentity = forAll { (a: A, f: A => M[B]) =>
    M.flatMap(M.unit(a))(f) == f(a)
  }
  leftIdentity && rightIdentity
}
```

结合律属性表明，连续使用函数进行扁平映射应该与在单子上下文中应用函数相同：

```java
forAll((a: M[A], f: A => M[B], g: B => M[C]) => {
  val leftSide = M.flatMap(M.flatMap(a)(f))(g)
  val rightSide = M.flatMap(a)(a => M.flatMap(f(a))(g))
  leftSide == rightSide
})
```

我们将省略隐式参数的定义以及组合规则：

```java
def monad[A, B, C, M[_]](implicit M: Monad[M], ...): Prop = {
  id[A, B, M] && associativity[A, B, C, M]
}
```

请在 GitHub 上查找源代码以查看这些属性的完整签名。

既然我们已经了解了需要定义哪些方法和它们应该如何表现，让我们来实现一些单子！为了实现 `flatMap`，我们需要了解相应容器的内部结构。在前一章中，我们通过委托给底层容器实现了 `map` 方法。现在，我们将使用一种低级方法来展示确实需要了解结构知识。

正如往常一样，我们将从标准效果中最简单的一个开始，即 `Option`。这是我们实现 `Monad[Option]` 的方法：

```java
implicit val optionMonad = new Monad[Option] {
  override def unitA: Option[A] = Some(a)

  override def flatMapA, B(f: A => Option[B]): Option[B] = a match {
    case Some(value) => f(value)
    case _ => None
  }
}
```

`unit` 的实现应该是显而易见的——将 `A` 转换为 `Option[A]` 的唯一方法是通过包装。就像我们之前做的那样，我们直接使用案例类构造函数来保留在 `a` 为 `null` 的情况下的结构。

`flatMap` 的实现也非常透明——我们不能将给定的函数应用到 `None` 上，因此我们直接返回 `None`。在 `a` 已定义的情况下，我们解包值并应用 `f` 到它上。这种 *解包* 正是我们使用对 `Option` 内部结构的了解来扁平化可能嵌套结果的时刻。

我们可以通过为不同类型的 `a` 和 `f` 定义一些属性来检查我们的实现是否遵守单调律：这些属性需要放置在一个扩展 `org.scalacheck.Properties` 的类中，就像往常一样：

```java
property("Monad[Option] and Int => String, String => Long") = {
  monad[Int, String, Long, Option]
}
property("Monad[Option] and String => Int, Int => Boolean") = {
  monad[String, Int, Boolean, Option]
}
+ Monad.Monad[Option] and Int => String, String => Long: OK, passed 100 tests.
+ Monad.Monad[Option] and String => Int, Int => Boolean: OK, passed 100 tests.
```

既然我们的属性对于两种不同的 `a` 类型以及两种不同的函数类型都成立，我们可以相当确信我们的代码是正确的，并继续处理其他容器。

对于 `Either`，我们遇到了一个小麻烦，就像我们定义它的 `Functor` 时一样——需要两个类型参数而不是 `Monad` 所需要的那个。你准备好以同样的方式处理它了吗——通过修复第二个类型参数并使用类型 lambda 来定义 monad 的最终类型？好消息是，我们不需要这样做！类型 lambda 是类型类编程中非常常见的东西，以至于许多人渴望有一个更简单的方式来完成这个任务。这就是插件被创建的原因。它允许我们在 Scala 中使用简化的语法来处理类型 lambda。

为了开始使用插件，我们只需要在我们的项目配置文件 `build.sbt` 中添加依赖项：

```java
addCompilerPlugin("org.spire-math" %% "kind-projector" % "0.9.8")
```

现在我们有了这个，我们可以将我们的类型 lambda 语法从通常的 `({type T[A] = Either[L, A]})#T` 简化为 `Either[L, ?]`。插件功能丰富，我们在这里不会进一步详细介绍；强烈建议访问文档页面 [`index.scala-lang.org/non/kind-projector/kind-projector/0.9.7`](https://index.scala-lang.org/non/kind-projector/kind-projector/0.9.7)。

使用我们的新工具，`eitherMonad` 的定义很容易阅读：

```java
implicit def eitherMonad[L] = new Monad[Either[L, ?]] {
  override def unitA: Either[L, A] = Right(a)

  override def flatMapA, B(f: A => Either[L, B]): Either[L, B] = a match {
    case Right(r) => f(r)
    case Left(l) => Left(l)
  }
}
```

类型类构造函数接受一个类型参数 `L`，用于 `Either` 的左侧。其余的实现现在应该非常熟悉了。值得提醒的是，`Either` 是右偏的——这就是我们从 `unit` 方法返回 `Right` 的原因。还值得提到的是在 `flatMap` 模式匹配中的最后一个情况，我们将从 `Left[L, A]` 中的 `l` 重新打包到 `Left[L, B]`。这样做是为了帮助编译器推断正确的返回类型。

对于属性定义，我们还需要修复左侧的类型。我们可以通过定义一个类型别名来完成这个任务，这将提高可读性：

```java
type UnitEither[R] = Either[Unit, R]

property("Monad[UnitEither[Int]] and Int => String, String => Long") = {
  monad[Int, String, Long, UnitEither]
}

property("Monad[UnitEither[String]] and String => Int, Int => Boolean") = {
  monad[String, Int, Boolean, UnitEither]
}
```

除了类型别名之外，属性的定义与我们对 `Option` 的定义相同。

`Monad[Try]` 的定义是通过类比完成的，我们将把它留给读者作为练习。

相比之下，`Monad[List]`（或者如果我们使用前一章的术语，是 `Monad[Bucket]`）相当不同，因为列表可以包含多个元素：

```java
implicit val listMonad = new Monad[List] {
  def unitA = List(a)

  def flatMapA,B(f: A => List[B]): List[B] = as match {
    case Nil => Nil
    case a :: as => f(a) ::: flatMap(as)(f)
  }
}
```

`unit` 的实现方式与其他效果相同——只是通过包装它的参数。`flatMap` 是以递归方式定义的。对于 `Nil`，我们返回 `Nil`。这种情况类似于 `Monad[Option]` 中的 `None` 的情况。在非空列表的情况下，我们必须将给定的函数应用于列表的所有元素，并同时展平结果。这是在第二个匹配情况中完成的。

让我们看看我们的属性是否成立：

```java
property("Monad[List] and Int => String, String => Long") = {
  monad[Int, String, Long, List]
}
property("Monad[List] and String => Int, Int => Boolean") = {
  monad[String, Int, Boolean, List]
}
+ Monad.Monad[List] and Int => String, String => Long: OK, passed 100 tests.
+ Monad.Monad[List] and String => Int, Int => Boolean: OK, passed 100 tests.
```

它看起来是这样的，但唯一的原因是 `ScalaCheck` 中的列表生成器不会生成具有显著大小的输入列表。如果它做到了，我们的属性会因为 `StackOverflowError` 而失败，因为这不是尾递归！

让我们通过使用在第三章，“深入函数”中讨论的技术来解决这个问题：

```java
override def flatMapA,B(f: A => List[B]): List[B] = {
  @tailrec
  def fMap(as: List[A], acc: List[B])(f: A => List[B]): List[B] = as match {
    case Nil => acc
    case a :: aas => fMap(aas, acc ::: f(a))(f)
  }
  fMap(as, Nil)(f)
}
```

现在我们通过引入累加器使我们的实现尾递归，我们可以安全地使用任意长度的列表。但是，由于这种方法仍然相当直接且速度较慢，所以这种方法仍然相当直接且速度较慢。在我的笔记本电脑上，这个实现消耗了比 List 的“原生”优化实现`flatMap`方法大约五倍的时间。结果证明，这正是委托有意义的场合：

```java
override def flatMapA,B(f: A => List[B]): List[B] = as.flatMap(f)
```

好吧，所以我们忽略了`Future`，但已经为我们在第六章，“探索内置效果”中讨论的所有容器实现了类型类实例。我们关于单子的任务完成了吗？结果证明，我们还没有——远远没有。就像只要适用性属性保持不变，就可以为不同的类型构造器定义无限数量的适用性一样，也可以用单子做同样的事情。

在接下来的部分，我们将一些广泛使用的单子，如 Id、State、Reader 和 Writer，放入代码中，并讨论它们有什么好处。

# Id 单子

与`Option`编码可选性一样，`Id`代表*没有什么特别之处*。它包装了一个值，但对它不做任何事情。我们为什么需要*不是东西*的东西呢？`Id`是一种*元提升*，可以代表任何东西作为效果，而不改变它。这该如何做到呢？首先，我们必须告诉编译器`Id[A]`与`A`是同一件事。这可以通过一个类型别名轻松完成：

```java
type Id[A] = A

```

这个类型定义将决定单子实现的细节：

```java
implicit val idMonad = new Monad[Id] {
  override def unitA: Id[A] = a
  override def flatMapA, B(f: A => Id[B]): Id[B] = f(a)
}
```

显然，`unit(a)`只是`a`，通过我们刚刚定义的类型别名，我们让编译器相信它不是`A`类型，而是一个`Id[A]`。同样，对于`flatMap`，我们无法做任何花哨的事情，所以我们只是将给定的函数`f`应用到`a`上，利用`Id[A]`实际上只是`A`的事实。

显然，由于我们不做任何事情，单子定律应该成立。但为了 100%确定，我们将它们编码为属性：

```java
property("Monad[Id] and Int => String, String => Long") = {
  monad[Int, String, Long, Id]
}
property("Monad[Id] and String => Int, Int => Boolean") = {
  monad[String, Int, Boolean, Id]
}
+ Monad.Monad[Id] and Int => String, String => Long: OK, passed 100 tests.
+ Monad.Monad[Id] and String => Int, Int => Boolean: OK, passed 100 tests.
```

属性是成立的——你还能期望从什么不做的事情中得到什么呢？但我们为什么一开始就需要这个“东西”呢？

这个问题有多个答案。从抽象的角度来看，`Id`单子承担了（惊喜！）在单子空间中的恒等元素的功能，就像零或一在加法或乘法下的数字空间中的恒等元素一样。正因为如此，它可以作为单子变换的占位符（我们将在下一章中了解它们）。在现有代码期望单子但不需要单子的情况下，它也可能很有用。我们将在本章后面看到这种方法是如何工作的。

现在我们已经用最简单的单子打好了基础，是时候做一些更复杂的事情了——实现 State 单子。

# State 单子

在命令式编程中，我们有全局变量的概念——在程序中的任何地方都可以访问的变量。这种方法被认为是一种不好的做法，但仍然被相当频繁地使用。全局状态的概念通过包括系统资源扩展了全局变量的概念。由于只有一个文件系统或系统时钟，因此从程序代码的任何地方都可以全局和普遍地访问它们，这是完全有道理的，对吧？

在 JVM 中，一些这些全局资源可以通过`java.lang.System`类获得。例如，它包含对“标准”输入、输出和错误流的引用，系统计时器，环境变量和属性。那么，如果 Java 在语言级别上公开全局状态，这绝对是一个好主意！

全局状态的问题在于它破坏了代码的*引用透明性*。本质上，引用透明性意味着应该始终可以在程序中的任何地方用评估的结果替换代码的一部分，例如函数调用，并且这种更改不应该导致程序行为可观察的变化。

引用透明性的概念与纯函数的概念密切相关——一个函数是纯的，如果对于所有引用透明的参数，它都是引用透明的。

我们将在稍后看到它是如何工作的，但首先，请考虑以下示例：

```java
var globalState = 0

def incGlobal(count: Int): Int = {
  globalState += count
  globalState
}

val g1 = incGlobal(10) // g1 == 10
val g2 = incGlobal(10) // g1 == 20
```

在`incGlobal`的情况下，该函数不是纯函数，因为它不是引用透明的（因为我们不能用它的评估结果替换它的调用，因为这些结果每次调用函数时都不同）。这使得在没有知道每次访问或修改全局状态的情况下，无法对程序的可能的输出进行推理。

相比之下，以下函数是引用透明的且是纯函数：

```java
def incLocal(count: Int, global: Int): Int = global + count

val l1 = incLocal(10, 0) // l1 = 10
val l2 = incLocal(10, 0) // l2 = 10
```

在函数式编程中，我们期望只使用纯函数。这使得全局状态作为一个概念不适合函数式编程。

但是，仍然有许多情况下需要积累和修改状态，但我们应该如何处理这种情况呢？

这就是`State`单子发挥作用的地方。状态单子是围绕一个函数构建的，该函数将相关部分的全局状态作为参数，并返回一个结果和修改后的状态（当然，不会在全局意义上改变任何东西）。此类函数的签名看起来像这样：`type StatefulFunction[S, A] = S => (A, S)`。

我们可以将这个定义包装成一个案例类，以简化对其辅助方法的定义。这个`State`类将表示我们的效果：

```java
final case class StateS, A)
```

我们还可以在伴随对象中定义几个构造函数，以便在三种不同情况下创建状态（要在 REPL 中这样做，您需要使用`:paste`命令粘贴案例类和伴随对象，然后按*Ctrl* + *D*：

```java
object State {
  def applyS, A: State[S, A] = State(s => (a, s))
  def get[S]: State[S, S] = State(s => (s, s))
  def setS: State[S, Unit] = State(_ => ((), s))
}
```

默认构造函数通过返回给定的参数作为结果，并传播现有的状态而不做任何改变，将一些值`a: A`提升到`State`的上下文中。获取器创建一个`State`，它包装了一个函数，返回给定的参数既作为状态也作为结果。设置器将`State`包装在函数上，该函数接受一个要包装的状态并产生没有结果。这些语义与读取全局状态（因此结果是相同的状态）和设置它（因此结果是`Unit`）相似，但应用于`s: S`。

目前，`State`只是围绕一些涉及通过（并可能改变）一些状态的计算的一个薄包装。我们希望能够将这个计算与下一个计算组合起来。我们希望以与组合函数类似的方式做到这一点，但现在我们有`State[S, A]`和`State[S, B]`。我们该如何做到这一点？

根据定义，我们的第二次计算接受第一次计算的结果作为其参数，因此我们以`(a: A) =>`开始。我们还指出，由于可能的状态变化和第二个状态的返回类型，我们将有一个`State[S, B]`，这为我们提供了与第一个计算组合的完整签名：`f: A => State[S, B]`。

我们可以将这种组合实现为`State`上的一个方法：

```java
final case class StateS, A) {
  def composeB: State[S, B] = {
    val composedRuns = (s: S) => {
      val (a, nextState) = run(s)
      f(a).run(nextState)
    }
    State(composedRuns)
  }
}
```

我们将组合的计算定义为两次运行的组合。第一次使用提供给第一个状态的输入，我们将它分解为结果和下一个状态。然后我们在结果上调用提供的转换`f`，并使用下一个状态来运行它。这两次连续的运行一开始看起来可能有些奇怪，但它们只是表示我们将来自不同状态的两个`run`函数融合为一个定义在组合状态上的函数。

现在，我们有一个效果，可以为其创建一个单子。你应该已经注意到，我们刚刚定义的`compose`方法的签名与单调的`flatMap`方法的签名相同。

在本例及以下情况中的`compose`并不指代我们在第三章“深入函数”中学习的函数组合，而是指 Kleisli 组合的概念。它通常被称为 Kleisli 箭头，本质上只是`A => F[B]`函数的一个包装，允许对返回单调值的函数进行组合。它通常命名为`>>=`，但在这里我们将坚持使用`compose`。

这允许我们将单调行为委托给我们在`State`中已有的逻辑，就像我们可以为标准效果做的那样：

```java
import ch09._
implicit def stateMonad[S] = new Monad[State[S, ?]] {
  override def unitA: State[S, A] = State(a)
  override def flatMapA, B(f: A => State[S, B]): State[S, B] = a.compose(f)
}
```

幸运的是，我们还可以将`unit`所做的提升委托给默认构造函数！这意味着我们已经完成了单子的定义，可以继续使用严格的测试方法，通过指定对其的属性检查来继续。

除了这种情况，我们不会这么做。

其背后的原因是`State`与到目前为止我们所考虑的其他效果在所包含的值方面相当不同。`State`是第一个完全围绕某个函数构建的效果。技术上，因为函数在 Scala 中是一等值，其他效果如`Option`也可以包含一个函数而不是一个值，但这是一种例外。

这给我们的测试尝试带来了复杂性。以前，我们以不同的方式修改了效果中的值，并通过比较它们来检查结果是否相等，这是符合单调律的要求。现在，我们需要将函数作为效果的值，这就面临了比较两个函数是否相等的问题。在撰写本书时，这是一个活跃的学术研究课题。就我们的实际目的而言，目前还没有其他方法可以证明两个函数相等，除了测试它们对每个可能的输入参数(s)并检查它们是否返回相同的结果——这显然是我们无法承担的。

相反，我们将证明我们的实现是正确的。我们将使用一种称为*替换模型*的方法来做这件事。该方法的核心在于使用引用透明性，用它们返回的值来替换所有的变量和函数调用，直到结果代码不能再简化为止——这非常类似于解代数方程。

让我们看看这是如何工作的。

在证明单调律之前，我们将首先证明一个有用的引理。

该引理表述如下：对于`as: M[A], f: A => M[B]`和`M = State`，使得`as.run = s => (a, s1)`（`run`方法返回一对`a`和`s1`，对于某个输入`s`和`f(b) = (b: A) => State(s1 => (b, s2))`），`M.flatMap(as)(f)`将始终产生`State(s => (b, s2))`。

这就是我们的公式是如何得出来的：

1.  根据定义，`as.run = s => (a, s1)`，这给我们`as = State(s => (a, s1))`。

1.  `flatMap`方法委托给定义在`State`上的`compose`方法，因此对于`M = State`，`M.flatMap(a)(f)`变为`a.compose(f)`。

1.  在`as`和`f`的术语中，`as.compose(f)`可以表示为`State(s => (a, s1)).compose(f)`。

现在，我们将用其定义来替换`compose`方法的调用：

```java
State(s => (a, s1)).compose(f) = State(s => {
  f(a).run(s1) // substituting f(a) with the result of the call
}) = State(s => {
  State(s1 => (b, s2)).run(s1)
}) = State(s => (b, s2))
```

在这里，我们已经证明了我们的假设，即对于`as = State(s => (a, s1))`和`f(a) = (b: A) => State(s1 => (b, s2))`，`Monad[State].flatMap(as)(f) = State(s => (b, s2))`。

现在，我们可以在证明`State`的单调律时使用这个引理。

我们将从恒等律开始，更具体地说，从左恒等律开始。这是我们如何在`ScalaCheck`属性中表述它的：

```java
val leftIdentity = forAll { as: M[A] =>
  M.flatMap(as)(M.unit(_)) == as
}
```

因此，我们想要证明的是，如果我们让`M = State`，那么随后的每个`as: M[A]`总是正确的：

```java
M.flatMap(as)(M.unit(_)) == as
```

让我们先简化等式的左边。根据定义，我们可以用`State`实现来替换`as`：

```java
M.flatMap(State(s => (a, s1)))(M.unit(_))
```

我们必须做的下一步是替换 `unit` 方法的调用及其实现。我们只是在委托给 `State` 的默认构造函数，它定义如下：

```java
 def applyS, A: State[S, A] = State(s => (a, s))
```

因此，我们的定义变成了以下形式：

```java
 M.flatMap(State(s => (a, s1)))(b => State(s1 => (b, s1)))
```

为了替换 `flatMap` 调用，我们必须记住它所做的只是委托给定义在 `State` 上的 `compose` 方法：

```java
State(s => (a, s1)).compose(b => State(s1 => (b, s1)))
```

现在，我们可以使用我们的状态组合引理，这给我们以下简化的形式：

```java
State(s => (a, s1))
```

这不能再简化了，所以我们现在将查看等式的右侧，`as`。同样，根据定义，`as` 可以表示为 `State(s => (a, s1))`。这给我们最终的证明，即 `State(s => (a, s1)) == State(s => (a, s1))`，这对于任何 `a: A` 总是成立的。

右侧的恒等性证明与左侧类似，我们将其留给读者作为练习。

我们需要证明的第二条定律是结合律。让我们回顾一下在 ScalaCheck 术语中它是如何描述的：

```java
forAll((as: M[A], f: A => M[B], g: B => M[C]) => {
  val leftSide = M.flatMap(M.flatMap(as)(f))(g)
  val rightSide = M.flatMap(as)(a => M.flatMap(f(a))(g))
  leftSide == rightSide
})
```

让我们看看我们能用它做什么，从 `leftSide` 开始，`M.flatMap(M.flatMap(as)(f))(g)`。

通过在内部部分将 `M` 替换为 `State`，`M.flatMap(as)(f)` 变成了 `State(s => (a, s1)).compose(f)`，通过应用我们的引理，它变成了 `State(s => (b, s2))`。

现在，我们可以替换外部的 `flatMap`：

`M.flatMap(State(s => (b, s2)))(g)` 等同于 `State(s => (b, s2)).compose(g)` **(1**)。

让我们保持这种形式，并查看`rightSide`：`M.flatMap(as)(a => M.flatMap(f(a))(g))`。

首先，我们将内部 `flatMap` 替换为 `compose`，然后再将 `(a: A) => M.flatMap(f(a))(g)` 转换为 `(a: A) => f(a).compose(g)`。

现在，根据我们用于左侧的 `f` 的定义，我们有 `f(a) = a => State(s1 => (b, s2))`，因此内部 `flatMap` 变成了 `a => State(b, s2).compose(g)`。

将外部的 `flatMap` 替换为 `compose`，结合先前的定义，我们得到 `State(s => (a, s1)).compose(a => State(s1 => (b, s2)).compose(g))`。

我们将再次使用我们的引理来替换第一次应用的 `compose`，这将得到 `State(s => (b, s2)).compose(g)` 作为结果。**(2**)。

**(1**) 和 **(2**) 是相同的，这意味着我们属性的 `leftSide` 和 `rightSide` 总是相等的；我们刚刚证明了结合律。

太好了，我们已经实现了 `State` 和相应的 monad，并且已经证明它是正确的。现在是时候看看它们在实际中的应用了。作为一个例子，让我们想象我们正乘船去钓鱼。这艘船有一个位置和方向，可以在一段时间内前进或改变方向：

```java
final case class Boat(direction: Double, position: (Double, Double)) {
  def go(speed: Float, time: Float): Boat = ??? // please see the accompanying code
  def turn(angle: Double): Boat = ??? // please see the accompanying code
}
```

我们可以通过调用其方法来绕着这条船走：

```java
scala> import ch09._
import ch09._
scala> val boat = Boat(0, (0d, 0d))
boat: Boat = Boat(0.0,(0.0,0.0))
scala> boat.go(10, 5).turn(0.5).go(20, 20).turn(-0.1).go(1,1)
res1: Boat = Boat(0.4,(401.95408575015193,192.15963378398988))
```

然而，这种方法有一个问题——它不包括燃油消耗。不幸的是，在开发船的导航时，这个方面没有被预见，后来作为全局状态添加。现在，我们将使用状态单子重构旧风格。如果燃油量被建模为升数，定义状态的最直接方式如下：

```java
type FuelState = State[Float, Boat]
```

现在，我们可以定义我们的船移动逻辑，该逻辑考虑了燃油消耗。但在做之前，我们将稍微简化一下我们的单子调用语法。目前，我们的单子 `flatMap` 和 `map` 方法接受两个参数——容器和应用于容器的函数。

我们希望创建一个包装器，它将结合效果和单子，这样我们就有了一个效果实例，并且只需要传递转换函数给映射方法。这就是我们表达这种方法的途径：

```java
object lowPriorityImplicits {
  implicit class MonadF[A, F[_] : Monad](val value: F[A]) {
    private val M = implicitly[Monad[F]]
    def unit(a: A) = M.unit(a)
    def flatMapB: F[B] = M.flatMap(value)(fab)
    def mapB: F[B] = M.map(value)(fab)
  }
}
```

当 `F` 有隐式单子定义可用时，隐式转换 `MonadF` 将任何效果 `F[A]` 包装起来。有了 `value`，我们可以将其用作定义在单子上的 `flatMap` 和 `map` 方法的第一个参数——因此，在 `MonadF` 的情况下，它们被简化为接受单个参数的高阶函数。通过导入这个隐式转换，我们现在可以直接在 `State` 上调用 `flatMap` 和 `map`：

```java
StateFloat, Boat.flatMap((boat: Boat) => StateFloat, Boat)
```

我们还需要创建一些纯函数，在移动船时考虑燃油消耗。假设我们无法更改 `Boat` 的原始定义，我们必须将这些函数的 `boat` 作为参数传递：

```java
lazy val consumption = 1f
def consume(speed: Float, time: Float) = consumption * time * speed
def turn(angle: Double)(boat: Boat): FuelState =
  State(boat.turn(angle))
def go(speed: Float, time: Float)(boat: Boat): FuelState = 
  new State(fuel => {
    val newFuel = fuel - consume(speed, time)
    (boat.go(speed, time), newFuel)
  })

```

`consume` 函数根据 `speed` 和 `time` 计算燃油消耗。在 `turn` 函数中，我们接受一个 `boat`，通过委托到默认实现来将其旋转指定的 `angle`，并将结果作为 `FuelState` 实例返回。

在 `go` 方法中也使用了类似的方法——为了计算船的位置，我们委托给船的逻辑。为了计算可用的燃油新总量，我们减少初始燃油量（作为参数传递），并将结果作为状态的一部分返回。

我们最终可以创建与最初定义相同的动作链，但这次是通过跟踪燃油消耗来实现的：

```java
import Monad.lowPriorityImplicits._
def move(boat: Boat) = StateFloat, Boat.
  flatMap(go(10, 5)).
  flatMap(turn(0.5)).
  flatMap(go(20,20)).
  flatMap(turn(-0.1)).
  flatMap{b: Boat => go(1,1)(b)}
```

如果你将这个片段与原始定义进行比较，你会看到船的路径是相同的。然而，幕后发生的事情要多得多。每次调用`flatMap`都会传递状态——这是在 monad 的代码中定义的。在我们的例子中，定义是`State`上定义的`compose`方法。传递给`flatMap`方法的函数描述了结果应该发生什么，以及可能传递的状态。从某种意义上说，使用 monads 给我们带来了责任分离——*monad 描述了计算步骤之间应该发生什么*，作为一步的结果传递给下一步，*我们的逻辑描述了在传递给下一步计算之前结果应该发生什么*。

我们使用部分应用函数定义我们的逻辑，这使真正发生的事情变得有些模糊——为了使这一点明显，最后一步使用显式语法定义。我们也可以通过使用 for-comprehension 使步骤之间传递结果的过程更加明确：

```java
def move(boat: Boat) = for {
  a <- StateFloat, Boat
  b <- go(10,5)(a)
  c <- turn(0.5)(b)
  d <- go(20, 20)(c)
  e <- turn(-0.1)(d)
  f <- go(1,1)(e)
} yield f
```

方法与之前相同，但只是语法有所改变——现在，在步骤之间传递船是显式的，但状态传递在视觉上消失了——for-comprehension 使 monadic 代码看起来像命令式。这是执行这两种方法的结果：

```java
scala> println(move(boat).value.run(1000f))
(Boat(0.4,(401.95408575015193,192.15963378398988)),549.0)
```

我们如何确保状态被正确传递？嗯，这正是 monad 法则保证的。对于那些好奇的人，我们甚至可以使用我们在状态伴生对象中定义的方法来操作状态：

```java
def logFuelState(f: Float) = println(s"Current fuel level is $f")

def loggingMove(boat: Boat) = for {
  a <- StateFloat, Boat
  f1 <- State.get[Float]
  _ = logFuelState(f1)
  _ <- State.set(Math.min(700, f1))
  b <- go(10,5)(a)
  f2 <- State.get[Float]; _ = logFuelState(f2)
  c <- turn(0.5)(b)
  f3 <- State.get[Float]; _ = logFuelState(f3)
  d <- go(20, 20)(c)
  f3 <- State.get[Float]; _ = logFuelState(f3)
  e <- turn(-0.1)(d)
  f3 <- State.get[Float]; _ = logFuelState(f3)
  f <- go(1,1)(e)
} yield f
```

我们通过添加日志语句来增强我们之前的 for-comprehension，以输出每一步后的当前状态——这些是如下形式的语句：

```java
  f1 <- State.get[Float]
  _ = logFuelState(f1)
```

是否感觉我们真的在读取某个全局状态？嗯，实际上，正在发生的事情是我们正在获取当前的`State`作为结果（这就是我们之前定义`State.get`的方式），然后将其传递给下一个计算——日志语句。后续的计算只是显式地使用前一步的结果，就像之前一样。

使用这种技术，我们也在修改状态：

```java
  _ <- State.set(Math.min(700, f1))
```

在这里，我们模拟我们的船有一个最大容量为 700 的油箱。我们通过首先读取当前状态，然后设置较小的值——`run`方法的调用者传递的状态或我们的油箱容量。`State.set`方法返回`Unit`——这就是我们忽略它的原因。

增加日志后的定义输出如下：

```java
scala> println(loggingMove(boat).value.run(1000f))
Current fuel level is 1000.0
Current fuel level is 650.0
Current fuel level is 650.0
Current fuel level is 250.0
Current fuel level is 250.0
```

如我们所见，700 的限制是在船的第一步移动之前应用的。

我们的`move`实现仍然存在问题——它使用硬编码的`go`和`turn`函数，好像我们只能导航一艘特定的船。然而，事实并非如此——我们应该能够使用任何具有`go`和`turn`功能的船，即使它们的实现略有不同。我们可以通过将`go`和`turn`函数作为参数传递给`move`方法来模拟这种情况：

```java
def move(
  go: (Float, Float) => Boat => FuelState, 
  turn: Double => Boat => FuelState
)(boat: Boat): FuelState
```

这个定义将允许我们在不同情况下为`go`和`turn`函数有不同的实现，但仍然沿着给定的硬编码路径引导船只。

如果我们仔细观察，我们会发现创建初始包装器后，`move`方法的定义不再有关于`State`的概念——我们需要它是一个 monad 才能使用 for-comprehension，但这个要求比我们目前拥有的 State 要通用得多。

我们可以通过改进这两个方面来使`move`函数的定义通用——通过传递效果而不是创建它，并使方法多态：

```java
def move[A, M[_]: Monad](
  go: (Float, Float) => A => M[A], 
  turn: Double => A => M[A]
)(boat: M[A]): M[A] = for {
  a <- boat
  b <- go(10,5)(a)
  // the rest of the definition is exactly like before
} yield f
```

现在，我们可以使用任何具有单子以及具有指定签名的`go`和`turn`函数的类型来遵循给定的路径。鉴于这种功能现在是通用的，我们也可以将它与默认船的定义一起移动到`Boat`伴随对象中。

让我们看看这种方法与状态 monad 一起是如何工作的。结果是，我们的`go`和`turn`方法定义根本不需要改变。我们唯一需要做的就是调用新的通用`move`方法：

```java
import Boat.{move, boat}
println(move(go, turn)(State(boat)).run(1000f))
```

它看起来更美观，但仍然有一些改进的空间。特别是，`turn`方法什么也不做，只是传播对默认实现的调用。我们可以像对`move`方法做的那样，使它通用：

```java
def turn[M[_]: Monad]: Double => Boat => M[Boat] =
  angle => boat => Monad[M].unit(boat.turn(angle))
```

我们不能使它关于`Boat`多态，因为我们需要传播对特定类型的调用，但我们仍然有通用的 monad 类型。这个特定的代码使用了`Monad.apply`的隐式定义来召唤特定类型的 monad。

实际上，我们也可以对`go`方法做同样的事情——提供一个默认的伪装实现，并将它们都放入`Boat`的伴随对象中：

```java
object Boat {
  val boat = Boat(0, (0d, 0d))
  import Monad.lowPriorityImplicits._
  def go[M[_]: Monad]: (Float, Float) => Boat => M[Boat] =
    (speed, time) => boat => Monad[M].unit(boat.go(speed, time))
  def turn[M[_]: Monad]: Double => Boat => M[Boat] =
    angle => boat => Monad[M].unit(boat.turn(angle))
  def move[A, M[_]: Monad](go: (Float, Float) => A => M[A], turn: Double => A => M[A])(boat: M[A]): M[A] = // definition as above
}
```

再次，要将这个定义放入 REPL，你需要使用`:paste`命令，然后是`boat`案例类的定义和伴随对象，以及*Ctrl* + *D*的组合。

现在，对于不需要覆盖默认行为的场景，我们可以使用默认实现。例如，对于状态的情况，我们可以去除默认的`turn`实现，并使用默认值调用`move`方法：

```java
import ch09._
import Boat.{move => moveB, turn => turnB, boat}
import StateExample._
type FuelState[B] = State[Float, B]
println(moveBoat(go, turnB[FuelState])(State(boat)).run(1000f))
```

我们必须通过提供类型参数来帮助编译器推断要使用的正确 monad 类型，但我们的状态行为定义现在简化为覆盖的`go`方法定义——其余的代码是通用的。

作为说明，我们可以重用我们迄今为止与`Id`单子一起使用的一切——结果应该与直接在`Boat`上执行调用链相同。这是使用`Id`单子完成的完整实现：

```java
import Monad.Id
import Boat._
println(move(go[Id], turn[Id])(boat))
```

再次强调，我们提供了要使用的单子类型，但这基本上就是全部了。由于`Id[Boat] = Boat`，我们甚至可以直接传递`boat`，而无需将其包装到`Id`中。

这不是很好吗？我们可以使用我们迄今为止定义的任何单子来传递不同的效果到以单子形式表述的主要逻辑中。我们将把使用现有定义的简单部分留给读者作为练习，现在我们将实现两个其他单子，代表`State`的读取和写入方面，即`Reader`和`Writer`单子。

# 读取单子

`State`单子代表一个外部（相对于逻辑定义）的状态，这个状态需要被考虑并可能被修改。《Reader》单子在这方面相似——它接受一个外部上下文并将其不变地传递给队列中的每个计算。在讨论状态单子时提到的全局状态方面，`Reader`将能够访问只读的系统属性。正因为如此，读取单子通常被称为依赖注入的机制——因为它接受一些外部配置（不一定是基本的东西，如字符串或数字，也可能是其他复杂的组件、数据库访问机制、网络套接字或其他资源），并将其提供给它包装的函数。

让我们看看`Reader`是如何定义的。我们已经在`State`和`Reader`之间进行了比较，定义也非常相似——唯一的区别是我们不需要返回更改后的上下文（毕竟它是只读的）。在代码中，它看起来是这样的：

```java
final case class ReaderR, A {
  def composeB: Reader[R, B] = 
    Reader { r: R => 
      f(run(r)).run(r)
    }
}
```

`Reader`类型只是对函数的一个包装，该函数接受一个类型为`R`的上下文并返回一些类型为`A`的结果。《flatMap`将两个`run`函数组合在一起——我们通过使用给定上下文调用`run`，将给定的转换应用于结果，然后对结果调用`run`来实现这一点。第一次调用`run`基本上是为了`this`，而第二次是为了通过应用`f`得到的`Reader`。

我们也可以定义一个构造器，它忽略任何给定的上下文：

```java
object Reader {
  def applyR, A: Reader[R, A] = Reader(_ => a)
}
```

现在我们有了这个模型，我们可以为它创建一个单子，就像我们为状态单子所做的那样——通过使用 kind-projector 语法：

```java
implicit def readerMonad[R] = new Monad[Reader[R, ?]] {
  override def unitA: Reader[R, A] = Reader(a)
  override def flatMapA, B(f: A => Reader[R, B]): Reader[R, B] = a.compose(f)
}
```

毫不奇怪，这个单子只是委托给刚刚定义的构造器和`compose`方法。令人惊讶的是，现在我们做了这件事，我们就完成了读取单子的定义，并且可以使用我们的移动函数定义来使用它！

让我们假设我们有一个规定，它定义了船只的速度限制以及它们一次允许的最大转向角度（听起来很奇怪，但在我们钓鱼的地方有判例法，所以我们就是这样做的）。

由于这是外部规则，我们必须用案例类来建模它：

```java
final case class Limits(speed: Float, angle: Double)
type ReaderLimits[A] = ch09.Reader[Limits, A]
```

我们还将定义一个别名，将 `Reader` 的上下文类型固定为 `Limits`。

现在，我们可以通过应用这些限制来重新定义我们的 `go` 和 `turn` 方法，如下所示：

```java
def go(speed: Float, time: Float)(boat: Boat): ReaderLimits[Boat] =
  ch09.Reader(limits => {
    val lowSpeed = Math.min(speed, limits.speed)
    boat.go(lowSpeed, time)
  })

def turn(angle: Double)(boat: Boat): ReaderLimits[Boat] =
  ch09.Reader(limits => {
    val smallAngle = Math.min(angle, limits.angle)
    boat.turn(smallAngle)
  })
```

实现本身并没有什么特别之处。函数的类型签名由 `move` 方法预定义。在每个动作之后，我们返回 `Reader[Limits, Boat]`。为了计算船的新状态，我们在确定可以应用的最大速度或角度后，委托给其方法。

由于我们以通用方式设计了其余的代码，这就足够了——让我们 `move`：

```java
import Monad.readerMonad
import Boat._
println(move(go, turn)(ch09.Reader(boat)).run(Limits(10f, 0.1)))
Boat(0.0,(250.00083305560517,19.96668332936563))
```

要运行此示例，请使用 SBT 的 `run` 命令。

我们将刚刚定义的 `go` 和 `turn` 函数传递给通用的 `move` 方法，以及正确包装的 `boat`，然后运行它。通过查看结果，我们可以断定速度限制得到了适当的运用。

在仔细审查了状态模态之后，关于读者就没有太多可讨论的了，因此我们可以继续到 Writer 模态。

# Writer 模态

`Writer` 模态是状态和读者模态的兄弟，它侧重于修改状态。其主要目的是通过在计算之间传递日志来提供一个写入某种日志的便利设施。日志的类型没有指定，但通常会选择具有可能低开销的追加操作的结构。举几个合适的例子，你可以使用标准库中的 `Vector` 或 `List`。在 `List` 的情况下，我们需要在最后将日志条目添加到前面，并反转生成的日志。

在我们深入讨论日志类型之前，最好意识到我们可以推迟这个决定。我们只需要知道如何将条目追加到现有日志中。或者换句话说，如何将两个日志结合起来，其中一个只包含单个条目。我们已经知道具有这种功能的结构——它是 `Semigroup`。实际上，我们还需要能够表示一个空日志，因此我们的最终决定是拥有一个 `Monoid`。

让我们把这些放在一起。Writer 类接受两个类型参数，一个用于日志条目，另一个用于结果。我们还需要能够有一个 `Monoid` 用于日志。逻辑本身不依赖外部任何东西；它只返回结果和更新后的日志：

```java
import ch07._
final case class WriterW: Monoid, A)
```

接下来，我们想要将我们的 writer 与另一个模态函数组合起来，就像我们之前做的那样：

```java
final case class WriterW: Monoid, A) {
  def composeB: Writer[W, B] = Writer {
    val (a, w) = run
    val (b, ww) = f(a).run
    val www = implicitly[Monoid[W]].op(w, ww)
    (b, www)
  }
}
```

方法的签名与其他我们在本章中遇到的模态非常相似。在内部，我们将当前 `Writer` 的状态分解为结果 `a` 和日志 `w`。然后，我们将给定的函数应用于结果，收集下一个结果和日志条目。最后，我们通过利用模态操作来组合日志条目，并返回结果和组合后的日志。

我们还可以定义默认构造函数，它只是返回一个带有空日志的给定参数：

```java
object Writer {
  def applyW: Monoid, A: Writer[W, A] = Writer((a, implicitly[Monoid[W]].identity))
}
```

单子的定义现在是对这些方法的机械委托。唯一的小区别是要求`Monoid[W]`可用：

```java
implicit def writerMonad[W : Monoid] = new Monad[Writer[W, ?]] {
  override def unitA: Writer[W, A] = Writer(a)
  override def flatMapA, B(f: A => Writer[W, B]): Writer[W, B] = a.compose(f)
}
```

再次，我们已经完成了，现在我们可以开始使用我们的新抽象了。假设现在规定要求我们将每个机器人的移动记录到日志中。我们很高兴遵守。只要它只涉及移动，我们就不需要修改`turn`函数——我们只需要扩展`go`的定义：

```java
type WriterTracking[A] = Writer[Vector[(Double, Double)], A]

def go(speed: Float, time: Float)(boat: Boat): WriterTracking[Boat] = new WriterTracking((boat.go(speed, time), Vector(boat.position)))
```

我们正在将船的位置写入日志，位置由`Vector`表示。在定义中，我们只是再次调用船，并将移动前的船的位置作为日志条目返回。我们还需要满足单子要求。单子定义的方式与我们在第七章中提到的类似，*理解代数结构*：

```java
implicit def vectorMonoid[A]: Monoid[Vector[A]] = 
  new Monoid[Vector[A]] {
    override def identity: Vector[A] = Vector.empty[A]
    override def op(l: Vector[A], r: Vector[A]): Vector[A] = l ++ r
  }
```

准备就绪后，我们再次使用 SBT 会话中的`run`命令来移动我们的船：

```java
import Monad.writerMonad
import Boat.{move, boat, turn}
println(move(go, turn[WriterTracking])(Writer(boat)).run)

(Boat(0.4,(401.95408575015193,192.15963378398988)),Vector((0.0,0.0), (50.0,0.0), (401.0330247561491,191.77021544168122)))
```

我们将增强的`go`函数和原始的`turn`函数（尽管使用`WriterTracking`类型）作为第一个参数列表，以及包裹在`Writer`中的`boat`作为第二个参数列表传递。输出不言自明——它是原始结果和包含每次移动前船的位置的向量——所有这些都无需触及转向逻辑的定义！

`Writer`单子结束了我们对单子王国的探索。在下一章中，我们将看看如何将它们结合起来。如果你的直觉告诉你这比结合应用性更复杂——毕竟，有一个整章是专门讨论这个主题的——那么你是正确的。它更复杂，但也更有趣。让我们看看吧！

# 概述

在本章中，我们探讨了将单子作为计算序列化的一种方式。我们研究了这种序列化在不同我们实现的单子之间的意义变化。`Id`只是按原样组合计算。`Option`在某个步骤返回无结果时提供了停止并返回无结果的可能性。`Try`和`Either`与`Option`具有类似的语义，但允许你指定`no result`的意义，无论是作为一个`Exception`还是`Either`的`Left`部分。`Writer`在链式计算中提供了一个只读日志。`Reader`为每个计算步骤提供了一些配置。`State`在动作之间携带一个*可变*状态。

我们讨论了定义单子的两个原始方法`unit`和`flatMap`如何允许你实现其他有用的方法，如`map`、`map2`和`apply`，从而证明每个单子都是一个函子和一个应用性。

在`map`和`flatMap`——作为 for-comprehensions——方面，我们定义了一些小的业务逻辑来控制船的移动。然后我们展示了即使底层单子的实现被重塑，这种逻辑也可以无变化地重用。

# 问题

1.  实现`Monad[Try]`。

1.  证明`State`单子的右单位律。

1.  从本章定义的单子中选择一个，实现`go`函数，该函数将以 1%的概率编码船只沉没的概念。

1.  请与第 3 个问题做同样的事情，但在 1%的移动中编码引擎故障的概念，使船只无法移动。

1.  使用以下模板（松散地）描述本章定义的单子本质——状态单子通过链式计算传递状态。计算本身接受前一次计算的结果，并返回结果以及新的状态。

1.  定义一个`go`方法，该方法既跟踪船只的位置，又使用以下类型的结构来采取船只沉没的可能性：

```java
type WriterOption[B] = Writer[Vector[(Double, Double)], Option[Boat]]
```

1.  将答案与第 6 个问题的答案以及我们在上一章中组合`Applicatives`的方式进行比较。

# 进一步阅读

+   阿图尔·S·霍特，《Scala 函数式编程模式：掌握 Scala 中的有效函数式编程》

+   伊万·尼古洛夫，《Scala 设计模式 第二版：学习如何使用 Scala 编写高效、简洁和可重用的代码》
