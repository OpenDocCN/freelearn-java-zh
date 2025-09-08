# 处理效果

在前两章中，我们的视角发生了相当大的转变。在第六章探索内置效果中，我们查看了一些标准库中可用的具体效果的实现。在第七章理解代数结构中，我们从实际跳到了理论，并玩转了抽象代数结构。

现在我们已经熟悉了使用由法律定义的抽象的过程，我们最终可以履行我们在第六章探索内置效果中给出的承诺，并识别出隐藏在我们在那里接触到的标准实现背后的抽象。

我们将定义和实现一个函子，这是一个在处理任何效果时都很有用的抽象概念。此外，我们还将有三种不同的风味，所以请保持关注！

到本章结束时，你将能够识别并实现或使用以下结构之一：

+   函子

+   适用函子

+   可遍历函子

# 技术要求

+   JDK 1.8+

+   SBT 1.2+

本章的源代码可在我们的 GitHub 仓库中找到，网址为[`github.com/PacktPublishing/Learn-Scala-Programming/tree/master/Chapter08`](https://github.com/PacktPublishing/Learn-Scala-Programming/tree/master/Chapter08)。

# 函子

在前一章中，我们讨论了想要在容器内部组合元素的情况。我们发现，例如`Reducible`和`Foldable`这样的抽象可以帮助我们通过接受一个两个参数的函数并将其*带入*容器，以便可以在其中内部的元素对上应用它。作为一个例子，我们展示了这种方法如何使实现一桶鱼的不同的生存策略成为可能。

我们还没有涵盖的情况是我们不想在容器中*组合*元素，而是*逐个*对它们进行操作。这是函数式编程的精髓——将纯函数应用于参数并获取结果，然后重复这个过程，使用结果。通常，连续应用于参数的函数可以组合成一个单一函数，这在某种程度上是将所有中间步骤融合为一个步骤。

回到我们的鱼示例。让我们想象我们有一条鱼，我们想吃它。我们首先检查鱼是否健康且仍然新鲜，然后我们会以某种方式烹饪它，最后我们会消费它。我们可能会用以下模型来表示这个序列，扩展前一章中原始的`Fish`定义：

```java
final case class Fish(volume: Int, weight: Int, teeth: Int, poisonousness: Int)

sealed trait Eatable

final case class FreshFish(fish: Fish)
final case class FriedFish(weight: Int) extends Eatable
final case class CookedFish(goodTaste: Boolean) extends Eatable
final case class Sushi(freshness: Int) extends Eatable
```

我们可能采取的行动将自然地表示为函数：

```java
import ch08.Model._
import ch08.ModelCheck._
val check: Fish => FreshFish = f => FreshFish(f)
val prepare: FreshFish => FriedFish = f => FriedFish(f.fish.weight)
val eat: Eatable => Unit = _ => println("Yum yum...")
```

然后，我们可能想要组合我们的操作，使它们代表从新鲜到被吃掉的鱼的全过程：

```java
def prepareAndEat: Fish => Unit = check andThen prepare andThen eat

```

现在，我们可以通过将组合函数应用于鱼来按需对鱼进行操作：

```java
val fish: Fish = fishGen.sample.get
val freshFish = check(fish)
```

在这个例子中，我们使用了我们在前一章中定义的 `Gen[Fish]` 函数。如果您需要刷新对这个是如何完成的了解，请查阅 GitHub 仓库。

到目前为止一切顺利——我们满意且高兴。但如果我们有一个鱼桶，情况就会改变。突然之间，我们定义的所有函数都变得毫无用处，因为我们不知道如何将这些函数应用于桶内的鱼！我们现在该怎么办？

在桶内“工作”的要求可能听起来很奇怪，但这仅仅是因为我们的例子与实现脱节。在编程中，大多数时候，与集合一起工作意味着我们在应用操作后拥有相同的集合（尽管元素已更改）。此外，如果集合的 *结构* 被保留，那么我们之前提到的范畴论可以提供一些关于组合操作保证的保证，只要这些操作遵守所需的一组定律。我们已经看到了这是如何与抽象代数结构一起工作的，并且这个原则适用于从范畴论派生出的所有抽象。在实践中，保留集合结构的要求意味着操作不能改变集合的类型或其中的元素数量，也不能抛出异常。

事实上，有一个抽象可以帮助我们在这个情况下。

`Functor` 有一个 `map` 方法，它接收一个容器和一个函数，并将此函数应用于容器中的所有元素，最后返回具有相同结构但填充了新元素的容器。这就是我们如何在 Scala 中指定它的方式：

```java
import scala.language.higherKinds
trait Functor[F[_]] {
  def mapA,B(f: A => B): F[B]
}
```

`F[_]` 是容器的类型构造函数。`map` 本身接收一个容器和一个要应用的函数，并返回一个包含新元素的容器。我们也可以将 `map` 定义得稍微不同，以柯里化的形式：

```java
def mapCA,B: F[A] => F[B]
```

在这里，`mapC` 接收一个名为 `A => B` 的函数，并返回一个名为 `F[A] => F[B]` 的函数，然后可以将其应用于容器。

由于这是一个抽象定义，我们自然会期望定义并满足一些定律——就像前一章一样。对于函子，有两个定律：

+   恒等律指出，对恒等函数进行映射不应改变原始集合

+   分配律要求对两个函数的连续映射应始终产生与对这两个函数组合进行映射相同的结果

我们将像前一章那样，将这些要求作为属性来捕捉。

首先，让我们看看恒等律：

```java
def id[A, F[_]](implicit F: Functor[F], arbFA: Arbitrary[F[A]]): Prop =
  forAll { as: F[A] => F.map(as)(identity) == as }
```

在这个属性中，我们使用了来自 第三章 的 `identity` 函数，*深入函数的探讨*，它只是返回其参数。

结合律稍微复杂一些，因为我们需要用随机函数来测试它。这需要大量的隐式函数可用：

```java
import org.scalacheck._
import org.scalacheck.Prop._

def associativity[A, B, C, F[_]](implicit F: Functor[F],
                                 arbFA: Arbitrary[F[A]],
                                 arbB: Arbitrary[B],
                                 arbC: Arbitrary[C],
                                 cogenA: Cogen[A],
                                 cogenB: Cogen[B]): Prop = {
  forAll((as: F[A], f: A => B, g: B => C) => {
    F.map(F.map(as)(f))(g) == F.map(as)(f andThen g)
  })
}
```

在这里，我们正在创建任意的函数`f: A => B`和`g: B => C`，并检查组合函数的效果是否与连续应用这两个函数相同。

现在，我们需要一些函子来应用我们的检查。我们可以通过委托到定义在`Option`上的`map`函数来实现`Functor[Option]`：

```java
implicit val optionFunctor: Functor[Option] = new Functor[Option] {
  override def mapA, B(f: A => B): Option[B] = in.map(f)
  def mapCA, B: Option[A] => Option[B] = (_: Option[A]).map(f)
}
```

实例定义为`implicit`，就像上一章一样，因此它代表一个类型类。

这个实现遵守必要的法则吗？让我们看看。本章中的属性在测试范围内定义，可以使用 SBT 中的`test`命令运行。它们不能作为 REPL 的独立部分粘贴，但只能作为`Properties`定义的一部分：

```java
property("Functor[Option] and Int => String, String => Long") = {
  import Functor.optionFunctor
  functor[Int, String, Long, Option]
}
+ Functor.Functor[Option] and Int => String, String => Long: OK, passed 100 tests.

property("Functor[Option] and String => Int, Int => Boolean") = {
  import Functor.optionFunctor
  functor[String, Int, Boolean, Option]
}
+ Functor.Functor[Option] and String => Int, Int => Boolean: OK, passed 100 tests.
```

我们需要指定函子和函数的类型，以检查使其不可能——在我们的情况下——在一般情况下制定函子属性的法则。函数式编程库 cats 通过也为参数类型定义类型类来解决此问题。我们将坚持显式定义——这对我们的学习目的来说足够了。

我们也可以以相同的方式实现我们在第六章中看到的其他效果的函子，*探索内置效果*。`Try`的函子与效果类型相同。我们将把这个实现留给读者作为练习。

`Either`的情况要复杂一些，因为我们需要将它接受的两个类型参数转换为一个`Functor`类型构造函数期望的单个类型参数。我们通过将左侧的类型固定为`L`并在函子的定义中使用类型 lambda 来实现这一点：

```java
implicit def eitherFunctor[L] = new Functor[({ type T[A] = Either[L, A] })#T] {
  override def mapA, B(f: A => B): Either[L, B] = in.map(f)
  def mapCA, B: Either[L, A] => Either[L, B] = (_: Either[L, A]).map(f)
}
```

有趣的是，实现本身再次相同。结果证明，这正是我们在第六章的*探索内置效果*结束时寻找的抽象。我们在第六章中讨论的所有标准效果都是函子！`map`方法定义中的可见差异来自于，对于标准效果，它是使用面向对象的多态定义的，而在我们的函子代码中，我们通过使用类型类的自定义多态来实现这一点。

让我们回到我们的鱼。因为我们有一个桶，它由`List`类型表示，所以我们需要一个`Functor[Bucket]`：

```java
implicit val bucketFunctor: Functor[List] = new Functor[List] {
  override def mapA, B(f: A => B): List[B] = in.map(f)
  def mapCA, B: List[A] => List[B] = (_: List[A]).map(f)
}
```

定义再次与之前相同。然而，我们现在可以按照需要对桶中的鱼执行操作，重用`bucketOfFishGen`：

```java
type Bucket[S] = List[S]
val bucketOfFishGen: Gen[List[Fish]] = Gen.listOf(fishGen)

val bucketOfFriedFish: Bucket[FriedFish] = ch08.Functor.bucketFunctor.map(bucketOfFishGen.sample.get)(check andThen prepare)
```

在这里，我们正在使用我们刚刚定义的函子来检查和准备桶内的鱼。我们实现的一个优点是桶可以是任何具有函子的类型。为了演示这一点，我们需要一个辅助函数，它将允许我们将函子作为第三个参数传递，连同我们在`Functor.map`定义中的两个参数一起：

```java
def mapFunc[A, B, F[_]](as: F[A])(f: A => B)(implicit functor: ch08.Functor[F]): F[B] = functor.map(as)(f)
```

这个函数接受一个效果和一个函数，并隐式解析适当的函子。调用代码不再进行这种区分，因为我们通过使用不同的函数以相同的方式映射三种不同类型的效果：

```java
import ch08.Functor._
import ch08.ModelCheck._
{
 type Bucket[S] = Option[S]
 mapFunc(optionOfFishGen.sample.get)(check)
}
{
 type Bucket[S] = Either[Exception, S]
 mapFunc(eitherFishGen.sample.get)(check andThen prepare)
}
{
 type Bucket[S] = List[S]
 mapFunc(listOfFishGen.sample.get)(prepareAndEat)
}
```

现在，它开始看起来像是一个有用的抽象——好吧，只要我们的愿望仅限于单参数函数。我们将在下一节中看到原因。

# 应用

通过函子，我们现在有了一种方便的方式来将函数应用到效果的内容中，无论效果本身的类型如何。我们能够通过应用与无效果鱼相同的逻辑来检查鱼并烹饪它。为了更熟悉函子，我们现在将使用新工具制作鱼馅饼。

首先，我们将定义一个从单个鱼制作馅饼的函数：

```java
final case class FishPie(weight: Int)
import ch08.Model._
def bakePie(fish: FreshFish, potatoes: Int, milk: Float): FishPie = FishPie(fish.fish.weight)
```

这很简单——一条鱼，一个馅饼，鱼的大小。现在，我们准备好在桶中烘焙每条鱼：

```java
mapFunc(listOfFishGen.sample.get)(bakePie)
```

哎呀！这不能编译，因为函子只接受单参数的函数，而我们有三个。

我们能做什么？一个可能性是对我们的函数进行重构和部分应用。我们还可以创建一个使用 `mapC` 将鱼桶转换为新鲜鱼桶的函数，以便我们可以进一步简化操作：

```java
val freshFishMaker: List[Fish] => List[FreshFish] = ch08.Functor.bucketFunctor.mapC(check)
```

然后，我们可以使用部分应用函数来实现其余的逻辑：

```java
def bucketOfFish: Bucket[Fish] = listOfFishGen.sample.get

def bakeFishAtOnce(potatoes: Int, milk: Float): FreshFish => FishPie = 
  bakePie(_: FreshFish, potatoes, milk)

val pie: Seq[FishPie] = mapFunc(freshFishMaker(bucketOfFish))(bakeFishAtOnce(20, 0.5f))
```

这是一个有效的方法，并且会起作用，但这种方法将为每条鱼使用相同数量的原料。如果采用这种策略，一些馅饼可能味道不太好。我们能做得更好吗？

好吧，我们可以将我们的原始函数转换为柯里化形式。这将给我们一个接受单个鱼然后是其他参数的函数：

```java
def bakeFish: FreshFish => Int => Float => FishPie = (bakePie _).curried
val pieInProgress: List[Int => Float => FishPie] = 
  mapFunc(freshFishMaker(bucketOfFish))(bakeFish)
```

现在，我们希望使用另一个桶中的原料，以便我们可以添加到 `pieInProgress` 中。不幸的是，这是函子无法帮助我们的事情。如果我们尝试嵌套，对土豆桶和牛奶桶的映射调用，我们就会得到以下类似的结果：

```java
mapFunc(pieInProgress) { (pieFactory: Int => Float => FishPie) =>
  mapFunc(bucketOfPotatoes) { potato =>
    mapFunc(bucketOfMilk) { milk =>
      pieFactory(potato)(milk)
    }
  }
}
```

不幸的是，每个嵌套调用都会将结果留在嵌套的桶中，以至于即使最终能够编译，我们也会有三个嵌套的桶。我们的函子不知道如何从彼此中提取嵌套的桶。

能帮助我们的是 *应用函子*。有时也称为 *应用函子*，这个结构通过两个额外的函数扩展了原始函子：

```java
trait Applicative[F[_]] extends Functor[F] {
  def applyA,B(f: F[A => B]): F[B]
  def unitA: F[A]
}
```

应用方法接受一个效果 `a` 和一个函数 `f`，该函数在相同的效果上下文中定义，并将 `f` 应用到 `a` 上，从而返回被相同效果包装的结果。

`unit` 方法允许我们将一个普通值 `a` 包装成效果。这通常被称为 *提升*，尤其是当 `a` 是一个函数时，因为它“提升”了原始值（或函数）到效果 `F` 的上下文中。

一个敏锐的读者会期待上述函数出现一些定律。你绝对是对的！有几个定律：

1.  同一律指出，应用一个恒等函数应该返回未改变的参数，就像恒等函数所做的那样。这与函子恒等律类似，但这次是为`apply`函数定义的。

1.  同态律指出，将函数应用于值然后提升结果与首先提升这个函数和值然后在应用上下文中应用它们是相同的。

1.  交换律指出，改变`apply`方法参数的顺序不应该改变结果。

1.  组合律指出，函数组合应该得到保留。

现在，这可能会开始听起来很抽象。让我们通过将它们作为属性来明确这些观点。

同一律是最简单的一个。唯一的注意事项是我们不能使用`identity`函数——我们必须明确指定`unit`方法的参数类型，因为没有编译器能为我们推断它：

```java
def identityProp[A, F[_]](implicit A: Applicative[F],
                          arbFA: Arbitrary[F[A]]): Prop =
  forAll { as: F[A] =>
    A(as)(A.unit((a: A) => a)) == as
  }
```

同态也不是非常引人注目——它实际上编码了我们用散文形式陈述的规则。类似于`identityProp`的情况，我们正在利用`apply`语法：

```java
def homomorphism[A, B, F[_]](implicit A: Applicative[F],
                             arbA: Arbitrary[A],
                             arbB: Arbitrary[B],
                             cogenA: Cogen[A]): Prop = {
  forAll((f: A => B, a: A) => {
    A(A.unit(a))(A.unit(f)) == A.unit(f(a))
  })
}
```

交换律是开始变得有趣的地方。我们将分别定义左右两侧以简化定义：

```java
def interchange[A, B, F[_]](implicit A: Applicative[F],
                            arbFA: Arbitrary[F[A]],
                            arbA: Arbitrary[A],
                            arbB: Arbitrary[B],
                            cogenA: Cogen[A]): Prop = {
  forAll((f: A => B, a: A) => {
    val leftSide = A(A.unit(a))(A.unit(f))
    val func = (ff: A => B) => ff(a)
    val rightSide = A(A.unit(f))(A.unit(func))
    leftSide == rightSide
  })
}
```

左侧与同态定义相同——我们将某个随机函数和值提升到应用中。现在，我们需要改变`f`和`a`的顺序。`f`是一个一等值，所以我们在这方面没问题，但`a`不是一个函数。因此，我们定义了一个辅助函数`func`，它接受与`f`相同类型的参数并返回类型`B`。给定`a`，我们只有一种方式来实现这一点。有了这个辅助函数，类型就会对齐。最后，我们定义了`rightSide`，其中参数顺序已改变，并以比较它们的属性结束。

组合属性是最长的，因为我们必须定义我们即将组合的函数。首先，让我们将函数组合定义为函数：

```java
def composeF[A, B, C]: (B => C) => (A => B) => (A => C) = _.compose
```

给定两个类型匹配的函数，`composeF`将通过委托给第一个参数的`compose`方法返回一个函数组合。

我们将再次分别定义属性的左右两侧：

```java
def composition[A, B, C, F[_]](implicit A: Applicative[F],
                               arbFA: Arbitrary[F[A]],
                               arbB: Arbitrary[B],
                               arbC: Arbitrary[C],
                               cogenA: Cogen[A],
                               cogenB: Cogen[B]): Prop = {
  forAll((as: F[A], f: A => B, g: B => C) => {
    val af: F[A => B] = A.unit(f)
    val ag: F[B => C] = A.unit(g)
    val ac: F[(B => C) => (A => B) => (A => C)] = A.unit(composeF)
    val leftSide = A(as)(A(af)(A(ag)(ac)))
    val rightSide = A(A(as)(af))(ag)

    leftSide == rightSide
  })
}
```

右侧很简单——我们连续将提升的函数`f`和`g`应用于某个效果`as`。根据组合律，如果我们在一个应用中应用组合，这必须得到保留。这正是左侧所做的。最好是从右到左阅读它：我们将组合函数的函数提升到应用中，然后连续应用提升的`f`和`g`，但这次是在`A`内部。这给我们一个构建在应用内部的`compose`函数，我们最终将其应用于`as`。

对于一个有效的适用性，所有这些属性都必须成立，以及我们之前定义的函子属性，如下面的代码片段所示（未显示隐含参数）：

```java
identityProp[A, F] && homomorphism[A, B, F] && interchange[A, B, F] && composition[A, B, C, F] && FunctorSpecification.functor[A, B, C, F]
```

通过属性保护，我们可以为标准效果定义几个适用性的实例，就像我们为函子（functor）所做的那样。`Option` 可能是最容易实现的一个。不幸的是，我们无法像 `map` 那样委托给实例方法，所以我们不得不亲自动手：

```java
implicit val optionApplicative: Applicative[Option] = new Applicative[Option] {
  ... // map and mapC are the same as in Functor
  override def applyA, B(f: Option[A => B]): Option[B] = (a,f) match {
    case (Some(a), Some(f)) => Some(f(a))
    case _ => None
  }
  override def unitA: Option[A] = Some(a)
}
```

类型签名决定了实现。我们不能通过应用 `f` 到 `a` 来返回任何其他 `Option[B]`。同样，我们也不能从 `unit` 方法返回 `Option[A]`。请注意，我们在这两种情况下都使用 `Some` 构造函数而不是 `Option` 构造函数，这是为了在 `null` 参数或返回值的情况下保持结构。

`Either` 和 `Try` 的实现与效果类型非常相似。值得注意的是，我们的 `Bucket` 类型，它由 `List` 表示，相当不同：

```java
implicit val bucketApplicative: Applicative[List] = new Applicative[List] {
  ... // map and mapC are the same as in Functor
  override def applyA, B(f: List[A => B]): List[B] = (a, f) match {
    case (Nil, _) => Nil
    case (_, Nil) => Nil
    case (aa :: as, ff :: fs) =>
      val fab: (A => B) => B = f => f(aa)
      ff(aa) :: as.map(ff) ::: fs.map(fab) ::: apply(as)(fs)
  }
  override def unitA: List[A] = List(a)
}
```

因为我们需要将所有函数应用到所有参数上，所以在我们的例子中以递归方式执行，将过程分为四个部分——处理两个第一个元素，第一个元素及其所有函数，所有元素和第一个函数，以及从两个列表中除了第一个元素之外的所有元素的递归调用。（注意：这并不是尾递归！）

使用 `bucketApplicative`，我们最终可以通过首先将其应用到 `potato`，然后应用到 `milk` 来完成我们的柯里 `pieInProgress` 函数：

```java
def bakeFish: FreshFish => Int => Float => FishPie = (bakePie _).curried

val pieInProgress: List[Int => Float => FishPie] = 
  mapFunc(freshFishMaker(bucketOfFish))(bakeFish)

def pie(potato: Bucket[Int], milk: Bucket[Float]) = 
  bucketApplicative(milk)(bucketApplicative(potato)(pieInProgress))

scala> pie(List(10), List(2f))
res0: List[ch08.Model.FishPie] = List(FishPie(21), FishPie(11), FishPie(78))
```

这个定义是有效的，并产生了预期的结果——很好。但实现并没有显示出混合三种成分的意图，这并不那么好。让我们改进它。

实际上，有三种不同的有效方式可以用基本函数来定义一个适用性（applicative）：

1.  我们刚刚实现的，使用 `apply` 和 `unit`。

1.  使用 `unit` 和 `map2` 方法来定义它，使得 `map2A, B, C(f: (A, B) => C): F[C]`。

1.  使用 `unit`、`map` 和 `product` 函数来定义它，使得 `productA, B: F[(A, B)]`。

`apply` 和 `map2` 方法在功能上同样强大，因为我们可以用其中一个来实现另一个。同样的，对于 `product` 方法也适用，尽管它较弱，因为它需要一个已定义的 `map` 函数。

由于这些函数功能相同，我们可以在类型类定义中直接实现它们，这样它们就可在所有类型类实例上使用。`map2` 方法看起来是一个好的开始：

```java
trait Applicative[F[_]] extends Functor[F] {
  // ...
  def map2A,B,C(f: (A, B) => C): F[C] =
    apply(fb)(map(fa)(f.curried))
}
```

实现看起来很简单，有些令人失望——我们只是将转换成柯里形式的给定 `f` 依次应用到 `fa` 和 `fb` 上，这样我们就可以分两步应用它们。

很有趣的是，`map2`方法是如何用`map`实现的，这在某种程度上是一个*低级*的映射。好奇的读者可能会问，是否可能用另一个*低级*的函数实现`map`。结果是我们能这样做！以下是实现方式：

```java
override def mapA,B(f: A => B): F[B] = apply(fa)(unit(f))
```

我们需要做的就是将给定的函数`f`提升到应用式的上下文中，并使用我们已有的`apply`函数。

以其他函数定义函数的方式在函数式编程中很常见。在抽象的底层，有一些提供所有其他定义基础的方法，不能定义为其他方法的组合。这些被称为*原始的*。我们正在讨论的应用式的三种风味通过它们选择的原始函数而不同。结果，我们的最初选择是它们中的第一个，即`unit`和`apply`方法。使用这些原始函数，我们能够用`Applicative`来定义`Functor`！在`map`的基础上定义一个`Functor.mapC`也是有意义的：

```java
def mapCA,B: F[A] => F[B] = fa => map(fa)(f)
```

以这种方式推导实现的好处是，一旦原始函数被正确实现并遵守应用式（或函子）定律，推导出的实现也应该是有法的。

回到应用式的风味——我们仍然需要实现`product`方法，它从两个应用式中创建一个产品应用式：

```java
def product[G[_]](G: Applicative[G]): Applicative[({type f[x] = (F[x], G[x])})#f] = {
  val F = this
  new Applicative[({type f[x] = (F[x], G[x])})#f] {
    def unitA = (F.unit(a), G.unit(a))
    override def applyA,B)(fs: (F[A => B], G[A => B])) =
      (F.apply(p._1)(fs._1), G.apply(p._2)(fs._2))
  }
}
```

这次，我们不得不再次使用类型 lambda 来表示两个类型`F`和`G`的乘积作为一个单一的类型。我们还需要将应用式的当前实例引用存储为`F`，这样我们才能稍后调用其方法。实现本身自然地用`unit`和`apply`原始函数表达。对于结果应用式，`unit`定义为`F`和`G`的单位乘积，而`apply`只是对给定参数使用`apply`方法的乘积。

不幸的是，我们仍然无法以一种非常可读的方式定义我们的`pie`函数。如果我们有`map3`，我们可以这样实现它：

```java
def pie3[F[_]: Applicative](fish: F[FreshFish], potato: F[Int], milk: F[Float]): F[FishPie] =
  implicitly[Applicative[F]].map3(fish, potato, milk)(bakePie)
```

显然，这种实现表达了一个非常清晰的意图：取三个装满成分的容器，对这些成分应用一个函数，然后得到一个装有派饼的容器。这对于任何有`Applicative`类型类实例的容器都适用。

好吧，我们已经知道如何从为抽象定义的原始函数推导函数。为什么我们不再次这样做呢？让我们开始吧：

```java
def map3A,B,C,D(f: (A, B, C) => D): F[D] =
  apply(fc)(apply(fb)(apply(fa)(unit(f.curried))))
```

嗯，这实际上是对`map2`函数的定义，只是增加了一个对第三个参数的`apply`调用！不用说，可以以这种方式为任何这种类型的任意数量实现`mapN`方法。我们也可以通过调用较小数量级的`map`以归纳方式定义它：

```java
def map4A,B,C,D,E(f: (A, B, C, D) => E): F[E] = {
  val ff: (A, B, C) => D => E  = (a,b,c) => d => f(a,b,c,d) 
  apply(fd)(map3(fa, fb, fc)(ff))
}
```

我们只需要将提供的函数转换为可以提供所有但最后一个参数和最后一个参数单独的形式。

现在，既然我们有 `pie3` 的实现，我们必须停下来一会儿。我们需要告诉你一些事情。是的，我们必须承认我们在定义 `check` 函数时有点作弊。当然，我们不能每次有 `Fish` 就返回 `FreshFish`，就像我们之前做的那样：

```java
lazy val check: Fish => FreshFish = f => FreshFish(f)
```

我们故意这样做，以便我们能够专注于 `Applicative`。现在，我们准备改进这一点。我们已经熟悉了可选性的概念，因此我们可以将这个函数改为返回 `Option`：

```java
lazy val check: Fish => Option[FreshFish]
```

但让我们稍后决定它应该是哪种类型的效果。现在我们先叫它 `F`。我们需要两种可能性：

+   如果鱼不新鲜，则返回一个空的 `F`

+   在其他情况下返回一个包含新鲜鱼的 `F`

在抽象方面，我们有一种方法可以将鱼提升到 `F` 中，一旦我们有了它的 `applicative`——`applicative` 会提供这个 `unit`。我们需要的只是一个空的 `F[FreshFish]`，我们将将其作为函数的参数提供。

因此，我们新的检查定义将如下所示：

```java
def checkHonestly[F[_] : Applicative](noFish: F[FreshFish])(fish: Fish): F[FreshFish] =
  if (scala.util.Random.nextInt(3) == 0) noFish else implicitly[Applicative[F]].unit(FreshFish(fish))
```

将空的 `F` 作为单独的参数列表，将允许我们稍后部分应用此函数。前面的实现在大约 30%的情况下返回一个空的 `F`。我们要求编译器检查隐式的 `Applicative` 是否对 `F` 可用，正如所讨论的。如果是这种情况，我们的实现将委托给它来创建一个适当的结果。

好的，我们现在有了一种方法来区分新鲜鱼和其他鱼，但还有一个问题。我们的 `pie3` 函数期望所有原料都被包裹在相同类型的 `applicative` 中。这在函数式编程中很常见，我们将通过将其他参数提升到相同的容器中来克服这个障碍。我们可以像对鱼那样对土豆和牛奶进行新鲜度检查，但为了简单起见，我们假设它们总是新鲜的（抱歉，挑剔的读者）：

```java
def freshPotato(count: Int) = List(Some(count))
def freshMilk(gallons: Float) = List(Some(gallons))

val trueFreshFish: List[Option[FreshFish]] = 
  bucketOfFish.map(checkHonestly(Option.empty[FreshFish]))
```

在检查了所有原料的新鲜度后，我们可以使用我们现有的 `pie3` 函数，几乎就像我们之前做的那样：

```java
import ch08.Applicative._
def freshPie = pie3[({ type T[x] = Bucket[Option[x]]})#T](trueFreshFish, freshPotato(10), freshMilk(0.2f))
```

差别在于我们需要帮助编译器识别正确的类型参数。我们通过使用类型 lambda 显式地定义容器的类型来实现这一点。然而，还有一个缺失的拼图。如果我们尝试编译前面的代码，它将失败，因为我们还没有 `Applicative[Bucket[Option]]` 的实例。

准备卷起袖子来实现它吗？好吧，尽管弄脏手没有什么不对，但我们不希望在每次想要组合它们时都实现一个新的 `applicative`。我们将要做的是定义一个通用的 `applicative` 组合，它本身也是一个 `applicative`。`applicative` 能够 *组合* 是它们最值得称赞的特性。让我们看看它是如何工作的。这是我们可以为我们的 `Applicative[F]` 实现它的方法：

```java
  def compose[G[_]](G: Applicative[G]): Applicative[({type f[x] = F[G[x]]})#f] = {
    val F = this

    def fab[A, B]: G[A => B] => G[A] => G[B] = (gf: G[A => B]) => (ga: G[A]) => G.apply(ga)(gf)

    def fgB, A: F[G[A] => G[B]] = F.map(f)(fab)

    new Applicative[({type f[x] = F[G[x]]})#f] {
      def unitA = F.unit(G.unit(a))
      override def applyA, B(f: F[G[A => B]]): F[G[B]] =
        F.apply(a)(fg(f))
    }
  }
```

再次，我们不得不使用类型 lambda 来告诉编译器这实际上只是一个类型参数，而不是两个。`unit`方法的实现只是将一个应用包裹进另一个应用中。`apply`方法更复杂，我们将其实现为一个局部函数，以便更清楚地了解发生了什么。我们首先做的事情是将类型为`G[A => B]`的内部函数转换为类型`G[A] => G[B]`。我们通过在`f`包裹的“内部”函数上应用应用`G`来实现这一点。现在我们有了这个函数，我们可以调用外部应用的`map`函数，将结果包裹进`F`。最后，我们应用这个包裹好的组合函数到原始函数和结果函数上，即`apply`方法的原始参数。

现在，我们可以按我们的意愿组合这些应用：

```java
implicit val bucketOfFresh: ch08.Applicative[({ type T[x] = Bucket[Option[x]]})#T] = 
  bucketApplicative.compose(optionApplicative)
```

然后使用这种组合来调用我们原始的馅饼制作逻辑：

```java
scala> println(freshPie)
List(Some(FishPie(40)), None, Some(FishPie(36)))
```

这种方法的优点在于，它允许我们像以下人工示例一样，以任意嵌套的方式重用现有的逻辑：

```java
import scala.util._
import ch08.Applicative
def deepX = Success(Right(x))
type DEEP[x] = Bucket[Try[Either[Unit, Option[x]]]]

implicit val deepBucket: Applicative[DEEP] =
 bucketApplicative.compose(tryApplicative.compose(eitherApplicative[Unit].compose(optionApplicative)))

val deeplyPackaged =
  pie3DEEP, freshPotato(10).map(deep), freshMilk(0.2f).map(deep))
```

在容器结构改变的情况下，我们只需要定义一个新的组合应用（以及一些语法辅助，如构造函数的类型别名，但这些不是必需的）。然后，我们就可以像之前一样使用现有的逻辑。这是在 REPL 中的结果：

```java
scala> println(deeplyPackaged)
List(Success(Right(Some(FishPie(46)))), Success(Right(Some(FishPie(54)))), Success(Right(None)))
```

我们可以通过重新布线组合应用来轻松地改变结果的结构：

```java
type DEEP[x] = Try[Either[Unit, Bucket[Option[x]]]]

implicit val deepBucket: Applicative[DEEP] = 
 tryApplicative.compose(eitherApplicative[Unit].compose(bucketApplicative.compose(optionApplicative)))

val deeplyPackaged =
  pie3DEEP, deep(freshPotato(10)), deep(freshMilk(0.2f)))
```

我们改变了组合的顺序，现在结果看起来不同了：

```java
scala> println(deeplyPackaged)
Success(Right(List(Some(FishPie(45)), Some(FishPie(66)), None)))
```

是否感觉结合应用（applicatives）能满足所有需求？嗯，从某种意义上说，确实如此，除非我们想要改变当前结果的结构。为了举例说明，让我们回顾一下我们为新鲜鱼制作的烘焙成果：`List(Some(FishPie(45)), Some(FishPie(66)), None)`。这是一个桶，里面要么有馅饼（如果鱼是新鲜的），要么如果没有，就是空的。但如果我们雇佣了一个新厨师，现在桶里的每条鱼都必须是新鲜的，否则整个桶就会被丢弃呢？在这种情况下，我们的返回类型将是`Option[Bucket[FishPie]]`——如果我们有一个装满新鲜鱼的桶，桶里就会装满馅饼，否则什么也没有。尽管如此，我们还想保留我们的厨房流程！这时，`Traversable`函子就登场了。

# Traversable

`Traversable`函子与我们在上一章中讨论的`Reducible`和`Foldable`类似。区别在于，在`Traversable`上定义的方法在遍历过程中保留了底层结构，而其他抽象则将其折叠成单个结果。`Traversable`定义了两个方法：

```java
import scala.{ Traversable => _ }

trait Traversable[F[_]] extends Functor[F] {
  def sequence[A,G[_]: Applicative](a: F[G[A]]): G[F[A]]
  def traverse[A,B,G[_]: Applicative](a: F[A])(f: A => G[B]): G[F[B]]
}
```

不幸的是，Scala 保留了一个过时的`Traversable`定义，这是从之前的版本遗留下来的，所以我们通过使用导入重命名来消除它。我们的`Traversable`定义了`sequence`和`traverse`方法，这些方法与在单例上定义的`reduce`和`fold`方法松散对应。从`sequence`方法开始，我们可以看到它将其参数“翻转”。这正是我们让新厨师高兴所需要的。让我们暂时跳过实现部分，看看它在实际中是如何工作的：

```java
scala> println(freshPie)
List(None, None, Some(FishPie(38)))

scala>println(ch08.Traversable.bucketTraversable.sequence(freshPie))
None
```

一旦我们在列表中遇到`None`，我们就会得到作为结果的`None`。让我们再试一次：

```java
scala> println(freshPie)
List(Some(FishPie(40)), Some(FishPie(27)), Some(FishPie(62)))

scala> println(ch08.Traversable.bucketTraversable.sequence(freshPie))
Some(List(FishPie(40), FishPie(27), FishPie(62)))

```

如果所有的鱼都是新鲜的，我们会得到一些预期的派，但我们对这种方法仍然不满意。原因是我们首先尽可能多地烤制所有新鲜派，然后在不是所有鱼都新鲜的情况下将它们丢弃。相反，我们希望在遇到第一只腐烂的鱼时立即停止。这正是`traverse`方法的作用。使用它，我们可以这样实现我们的烤制过程：

```java
ch08.Traversable.bucketTraversable.traverse(bucketOfFish) { a: Fish =>
  checkHonestly(Option.empty[FreshFish])(a).map(f => bakePie(f, 10, 0.2f))
}
```

在这里，我们正在遍历`bucketOfFish`。我们为此使用`bucketTraversable`。它期望一个名为`Fish => G[?]`的函数，这样`G`就是可应用的。我们可以通过提供一个名为`Fish => Option[FishPie]`的函数来满足这个要求。我们使用`checkHonestly`将一个`Fish`提升到`Option[FreshFish]`，然后我们需要使用我们的原始`bakePie`方法来`map`它。

`traverse`是如何实现的？不幸的是，这个实现需要知道效果的结构，以便可以保留它。因此，它需要为类型类的每个实例实现，或者委托给另一个抽象，在这个抽象中保留这种知识，比如`Foldable`。

这是如何为`Traversable[List]`实现`traverse`方法的：

```java
override def traverse[A, B, G[_] : Applicative](a: Bucket[A])(f: A => G[B]): G[Bucket[B]] = {
  val G = implicitly[Applicative[G]]
  a.foldRight(G.unit(List[B]()))((aa, fbs) => G.map2(f(aa), fbs)(_ :: _))
}
```

为了保留列表的结构，我们从空列表开始`foldRight`，在每次折叠迭代中使用`map2`来调用提供的函数，将原始列表的下一个元素提升到`G`，并将其附加到结果中。

对于`Option`，我们可以使用与`fold`类似的方法，但由于我们只需要处理两种情况，模式匹配实现可以更好地揭示意图：

```java
implicit val optionTraversable = new Traversable[Option] {
  override def mapA, B(f: A => B): Option[B] =
   Functor.optionFunctor.map(in)(f)
  override def traverse[A, B, G[_] : Applicative](a: Option[A])(f: A => G[B]): G[Option[B]] = {
    val G = implicitly[Applicative[G]]
    a match {
      case Some(s) => G.map(f(s))(Some.apply)
      case None => G.unit(None)
    }
  }
}
```

我们只是通过使用`Option`的不同状态的方法将`Option`提升到`G`的上下文中。值得注意的是，在非空`Option`的情况下，我们直接使用`Some.apply`来保留所需的结构。

好消息是第二种方法`sequence`比`traverse`弱。正因为如此，它可以直接在`Traversable`上根据`traverse`定义：

```java
def sequence[A,G[_]: Applicative](a: F[G[A]]): G[F[A]] = traverse(a)(identity)
```

它只是使用`identity`函数返回`G[A]`的正确值，正如`traverse`所期望的。

作为一种函子，`Traversable`s 也可以组合。`compose`函数将具有以下签名：

```java
trait Traversable[F[_]] extends Functor[F] {
  // ...
  def compose[H[_]](implicit H: Traversable[H]): Traversable[({type f[x] = F[H[x]]})#f]
}
```

我们将把这个实现的任务留给读者。

这就是组合`Traversable`可以使生活变得更轻松的方式。还记得我们有争议的`deeplyPackaged`示例吗？这又是容器类型的模样：

```java
type DEEP[x] = scala.util.Try[Either[Unit, Bucket[Option[x]]]]
```

你能想象遍历它并对它的元素应用一些逻辑吗？使用组合的`Traversable`，这绝对简单直接：

```java
import ch08.Traversable._
val deepTraverse = tryTraversable.compose(eitherTraversable[Unit].compose(bucketTraversable))

val deepYummi = deepTraverse.traverse(deeplyPackaged) { pie: Option[FishPie] =>
  pie.foreach(p => println(s"Yummi $p"))
  pie
}
println(deepYummi)
```

我们首先将`Traversable`组合起来以匹配我们的嵌套类型。然后，我们遍历它，就像我们之前做的那样。请注意，我们省略了底层的`Option`类型，并将其作为遍历函数参数的包装类型。这是前面代码片段的输出：

```java
Yummi FishPie(71)
Yummi FishPie(5)
Yummi FishPie(82)
Some(Success(Right(List(FishPie(82), FishPie(5), FishPie(71)))))
```

你感觉像拥有了超能力吗？如果你仍然没有这种感觉，我们将在下一章提供更多内容！

# 摘要

这是一章内容密集的章节。我们学习了以某种方式处理效果的概念，即效果的结构知识被外包给另一个抽象。我们研究了三种这样的抽象。

`Functor`允许我们将一个参数的函数应用到容器中存储的每个元素上。

`Applicative`（或应用函子）以扩展`Functor`的方式，使得应用一个有两个参数的函数（以及通过归纳，任何数量的参数的函数）成为可能。我们已经看到，可以选择三个同样有效的原始函数集来定义应用函子，并从这些原始函数中推导出所有其他方法。

我们说，定义一组最小原始函数以及将这些原始函数作为其他功能定义的方法是函数式编程中的一种常见方法。

我们最后看到的抽象是`Traversable`（或可遍历函子），它允许我们遍历效果，从而改变其内容，但保留底层结构。

我们特别关注了应用和遍历的组合。在实现了允许我们构建任意函子堆栈并使用这些堆栈直接到达*核心*的通用方法之后，我们能够重用那些以纯无效果类型定义的现有函数。

然而，我们没有展示的是，一个应用函子的数据如何影响堆栈中更深层次的函数调用——我们只是在使用示例中使用了常量参数。我们这样做的原因是应用不支持计算序列化。

在下一章，我们将学习另一种能够真正链式计算抽象——单子。

# 问题

1.  实现`Functor[Try]`。检查你的实现是否通过属性检查，就像在本章中做的那样。

1.  实现`Applicative[Try]`。检查你的实现是否通过属性检查，就像在本章中做的那样。

1.  实现`Applicative[Either]`。检查你的实现是否通过属性检查，就像在本章中做的那样。

1.  实现`Traversable[Try]`。

1.  实现`Traversable[Either]`。

1.  实现 `Traversable.compose`，就像我们在本章末尾讨论的那样。

# 进一步阅读

+   Atul S. Khot，《Scala 函数式编程模式》：掌握和执行有效的 Scala 函数式编程

+   Ivan Nikolov，《Scala 设计模式》第二版：学习如何使用 Scala 编写高效、简洁且可重用的代码
