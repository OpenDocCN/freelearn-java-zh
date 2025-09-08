# 查看 Monad 转换器和自由 Monads

在第六章，“探索内置效果”中，我们研究了标准效果，并承诺揭示它们背后的概念真相；我们还讨论了组合它们的话题。从那时起，我们已经讨论了代数结构，如幺半群和群，函子，applicatives 和 Monads，履行了我们的第一个承诺。但组合主题一直未被揭露。

在第八章，“处理效果”中，我们实现了一种通用方法来组合 applicatives——这本身非常有用，但无法帮助我们组合具有 Monadic 性质的标准化效果。

在本章中，我们将最终履行我们的第二个承诺，通过讨论一些将不同的 Monadic 效果结合起来的方法。我们将探讨相关的复杂性以及 Scala 社区用来处理这些障碍的一些解决方案，包括：

+   Monad 转换器

+   Monad 转换器堆栈

+   自由 Monads

# 技术要求

在我们开始之前，请确保你已经安装了以下内容：

+   JDK 1.8+

+   SBT 1.2+

本章的源代码可在[`github.com/PacktPublishing/Learn-Scala-Programming/tree/master/Chapter10`](https://github.com/PacktPublishing/Learn-Scala-Programming/tree/master/Chapter10)找到。

# 组合 Monads

在第六章，“探索内置效果”中，我们讨论了标准效果，如`Option`、`Try`、`Either`和`Future`。在第九章，“熟悉基本 Monads”中，我们继续前进，并为它们都实现了 Monads。在我们的例子中，我们展示了 Scala 如何通过 for-comprehension 提供良好的语法，for-comprehension 是`map`、`flatMap`和可能的`filter`方法的组合的语法糖。在我们的所有例子中，我们使用 for-comprehension 来定义一系列步骤，这些步骤构成了某个过程，其中前一步的计算结果被下一步消耗。

例如，这是我们在第六章，“探索内置效果”中用`Option`定义的钓鱼过程的定义方式：

```java
val buyBait: String => Option[Bait]
val makeBait: String => Option[Bait]
val castLine: Bait => Option[Line]
val hookFish: Line => Option[Fish]

def goFishing(bestBaitForFish: Option[String]): Option[Fish] =
  for {
    baitName <- bestBaitForFish
    bait <- buyBait(baitName).orElse(makeBait(baitName))
    line <- castLine(bait)
    fish <- hookFish(line)
  } yield fish
```

通过我们对 Monads 的新知识，我们可以使这个实现不受效果影响：

```java
def goFishing[M[_]: Monad](bestBaitForFish: M[String]): M[Fish] = {

  val buyBait: String => M[Bait] = ???
  val castLine: Bait => M[Line] = ???
  val hookFish: Line => M[Fish] = ???

  import Monad.lowPriorityImplicits._

  for {
    baitName <- bestBaitForFish
    bait <- buyBait(baitName)
    line <- castLine(bait)
    fish <- hookFish(line)
  } yield fish
}
Ch10.goFishing(Option("Crankbait"))
```

我们使用这种方法无法做到的是使用`Option`特有的`orElse`方法来定义诱饵获取的不愉快路径。

我们在这里进行的另一个简化是假设我们所有的行为都可以用相同的效果来描述。实际上，这几乎肯定不是情况。更具体地说，获取诱饵并等待钓鱼可能比抛线要花更长的时间。因此，我们可能希望用`Future`而不是`Option`来表示这些行为：

```java
val buyBait: String => Future[Bait]
val hookFish: Line => Future[Fish]
```

或者，用通用术语来说，我们会有`N`类型的 effect 而不是`M`：

```java
def goFishing[M[_]: Monad, N[_]: Monad](bestBaitForFish: M[String]): N[Fish] = {

  val buyBait: String => N[Bait] = ???
  val castLine: Bait => M[Line] = ???
  val hookFish: Line => N[Fish] = ???

  // ... the rest goes as before
}
import scala.concurrent.ExecutionContext.Implicits.global
Ch10.goFishingOption, Future)
```

但不幸的是，这不再能编译了。让我们考虑一个更简单的例子来理解为什么：

```java
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.Future

scala> for {
     | o <- Option("str")
     | c <- Future.successful(o)
     | } yield c
           c <- Future.successful(o)
             ^
On line 3: error: type mismatch;
        found : scala.concurrent.Future[String]
        required: Option[?]
```

编译器不再接受`Future`而不是`Option`。让我们将我们的 for-comprehension 去糖化，看看发生了什么：

```java
Option("str").flatMap { o: String =>
  val f: Future[String] = Future(o).map { c: String => c }
  f
}
```

现在问题浮出水面——`Option.flatMap`期望一个返回`Option`的函数作为效果（这特别使用了`Option.flatMapB: Option[B]`的定义，以及一般意义上的`Monad.flatMap`）。但我们的返回值被`Future`包裹，这是应用`Future`的`map`函数的结果。

将这个推理推广，我们可以得出结论，在单个 for-comprehension 中，只能使用相同类型的 effect。

由于这个原因，我们似乎有两种方法来组合所需的效果：

+   将它们放入单独的 for-comprehensions

+   将不同的 effect 提升到某种公共分母类型

我们可以使用我们的钓鱼示例作为游乐场来比较这两种方法。单独的 for-comprehensions 的变化将如下所示：

```java
for {
  baitName <- bestBaitForFish
} yield for {
  bait <- buyBait(baitName)
} yield for {
  line <- castLine(bait)
} yield for {
  fish <- hookFish(line)
} yield fish
```

这看起来比原始版本略差，但除了结果类型从`N[Fish]`变为`M[N[M[N[Fish]]]]`之外，仍然相当不错。在`Future`和`Option`的具体情况下，它将是`Option[Future[Option[Future[Fish]]]]`，没有简单的方法可以提取结果，除非逐层通过。这不是一件好事，我们将把它留给谨慎的读者作为练习。

另一个选择是放弃我们实现的慷慨，使其非多态，如下所示：

```java
def goFishing(bestBaitForFish: Option[String]): Future[Fish] =
  bestBaitForFish match {
    case None => Future.failed(new NoSuchElementException)
    case Some(name) => buyBait(name).flatMap { bait: Bait =>
      castLine(bait) match {
        case None => Future.failed(new IllegalStateException)
        case Some(line) => hookFish(line)
      }
    }
  }
```

除了失去通用适用性之外，这种实现显然的缺点是可读性大大降低。

让我们希望第二种方法，即效果类型的公共分母，比第一种方法更有成效。

首先，我们需要决定我们想要如何组合我们目前拥有的两个 effect。有两种选择：`Future[Option[?]]`和`Option[Future[?]]`。从语义上看，在某个时间点有一个可选的结果感觉比有一个将来完成的操作更合适，所以我们将继续使用第一种选择。

使用这种新的固定类型，我们拥有的函数现在都无效了——它们现在都有错误的结果类型。转换为正确的类型只需调整类型，我们可以在现场完成：

```java
val buyBaitFO: String => Future[Option[Bait]] = (name: String) => buyBait(name).map(Option.apply)
val castLineFO: Bait => Future[Option[Line]] = castLine.andThen(Future.successful)
val hookFishFO: Line => Future[Option[Fish]] = (line: Line) => hookFish(line).map(Option.apply)
```

我们需要做的就是将`Option`包裹进`Future`或`Future`包裹进`Option`，具体取决于原始函数的返回类型。

为了保持一切一致，我们也将`goFishing`函数的参数类型和返回类型以相同的方式更改：

```java
def goFishing(bestBaitForFish: Future[Option[String]]): Future[Option[Fish]] = ???
```

由于我们努力将逻辑本身作为 for-comprehension 来表述，合理地尝试用`flatMap`来表述它是合理的：

```java
bestBaitForFish.flatMap { /* takes Option[?] and returns Future[Option[?]] */ }
```

作为对 `flatMap` 的一个论证，我们必须提供一个函数，该函数接受一个 `Option[String]` 并返回 `Future[Option[Fish]]`。但我们的函数期望“真实”的输入，而不是可选的。我们不能像之前讨论的那样在 `Option` 上使用 `flatMap`，也不能简单地使用 `Option.map`，因为它将我们的结果类型包裹在额外的可选性层中。我们可以使用模式匹配来提取值：

```java
case None => Future.successful(Option.empty[Fish])
case Some(name) => buyBaitFO(name) /* now what ? */
```

在 `None` 的情况下，我们只需简化流程并返回结果。在这种情况下，我们确实有一个 `name`；我们可以调用一个相应的函数，并将这个 `name` 作为参数传递。问题是，我们如何进一步操作？如果我们仔细观察 `buyBaitFO(name)` 的返回类型，我们会看到它与初始参数相同——`Future[Option[?]]`。因此，我们可以尝试再次使用 flatmapping 和模式匹配的方法，经过多次迭代后给出以下实现：

```java
def goFishingA(bestBaitForFish: Future[Option[String]]): Future[Option[Fish]] =
  bestBaitForFish.flatMap {
    case None => Future.successful(Option.empty[Fish])
    case Some(name) => buyBaitFO(name).flatMap {
      case None => Future.successful(Option.empty[Fish])
      case Some(bait) => castLineFO(bait).flatMap {
        case None => Future.successful(Option.empty[Fish])
        case Some(line) => hookFishFO(line)
      }
    }
  }
```

这个片段中有许多重复，但它看起来已经有些结构了。通过提取重复的代码片段，我们可以提高其可读性。首先，我们可以将 *无结果* 的情况做成多态，如下所示：

```java
def noResult[T]: Future[Option[T]] = Future.successful(Option.empty[T])
```

其次，我们可能将关于 `flatMap` 和模式匹配的推理作为一个独立的、多态的函数捕捉：

```java
def continueA, B(f: A => Future[Option[B]]): Future[Option[B]] =
  arg.flatMap {
    case None => noResult[B]
    case Some(a) => f(a)
  }
```

经过这些修改，我们最后的尝试开始看起来更加简洁：

```java
def goFishing(bestBaitForFish: Future[Option[String]]): Future[Option[Fish]] =
  continue(bestBaitForFish) { name =>
    continue(buyBaitFO(name)) { bait =>
      continue(castLineFO(bait)) { line =>
        hookFishFO(line)
      }
    }
  }
```

这可以说是相当好的，我们可以在这一刻停止，但还有一个方面我们可以进一步改进。`continue` 函数调用是嵌套的。这使得业务逻辑流程的制定变得复杂。如果我们能够有一种流畅的接口，并且能够链式调用 `continue` 调用，可能会更有益：

这很容易通过将 `continue` 的第一个参数捕捉为某个类的值来实现。这将改变我们的实现形式如下：

```java
final case class FutureOptionA {
  def continueB: FutureOption[B] = new FutureOption(value.flatMap {
    case None => noResult[B]
    case Some(a) => f(a).value
  })
}
```

有两种方法可以进一步改进。首先，`continue` 的签名表明它是一个 Kleisli 箭头，这是我们之前章节中介绍的。其次，以这种形式，每次我们需要调用 `continue` 方法时，都需要手动将 `value` 包装在 `FutureOption` 中。这将使代码变得冗长，我们可以通过将其作为一个 `implicit` 类来增强我们的实现：

```java
implicit class FutureOptionA {
  def composeB: FutureOption[B] = new FutureOption(value.flatMap {
    case None => noResult[B]
    case Some(a) => f(a).value
  })
}
```

让我们看看在引入这些更改后，我们的主要流程看起来像什么：

```java
def goFishing(bestBaitForFish: Future[Option[String]]): Future[Option[Fish]] = {
  val result = bestBaitForFish.compose { name =>
    buyBaitFO(name).compose { bait =>
      castLineFO(bait).compose { line =>
        hookFishFO(line)
      }
    }
  }
  result.value
}
```

太棒了！你能发现进一步改进的可能性吗？如果我们仔细审查 `FutureOption` 的类型签名，我们会看到我们对包裹的 `value` 所做的一切都是调用在 `Future` 上定义的 `flatMap` 方法。但我们已经知道适当的抽象——这是一个 monad。利用这一知识将允许我们使我们的类多态，并在需要的情况下可能将其用于其他类型的效应：

```java
implicit class FOption[F[_]: Monad, A](val value: F[Option[A]]) {
  def composeB: FOption[F, B] = {
    val result = value.flatMap {
      case None => noResultF[F, B]
      case Some(a) => f(a).value
    }
    new FOption(result)
  }
  def isEmpty: F[Boolean] = Monad[F].map(value)(_.isEmpty)
}
```

为了证明新实现的多态性质不会损害我们根据需要定义辅助函数的灵活性，我们还添加了一个方法来检查我们拥有的 monad 组合是否为空。

不幸的是，如果我们尝试使这种实现对第二个效果的类型多态，我们会发现这是不可能的——我们需要像之前解释的那样将其分解，为此我们需要了解效果实现的细节。

在这一点上，一个敏锐的读者会记得，我们在上一章中开发的所有的 monad 都是基于具有相同签名的`compose`函数实现的。我们能再次尝试同样的技巧，为`FutureOption`类型实现一个 monad 吗？熟悉上一章的读者会知道，这几乎是一个机械的任务，即委托给我们刚刚提出的实现：

```java
implicit def fOptionMonad[F[_] : Monad] = new Monad[FOption[F, ?]] {
  override def unitA: FOption[F, A] = Monad[F].unit(Monad[Option].unit(a))
  override def flatMapA, B(f: A => FOption[F, B]): FOption[F, B] =
    a.compose(f)
}
```

现在，我们还需要将原始函数的返回类型更改为`FOption[Future, ?]`，以匹配我们新 monad 的类型签名。我们不需要接触实现——编译器会自动将`implicit FOption`包装在结果周围：

```java
val buyBaitFO: String => FOption[Future, Bait] = // as before
val castLineFO: Bait => FOption[Future, Line] = // as before
val hookFishFO: Line => FOption[Future, Fish] = // as before
```

现在，我们再次可以制定我们的逻辑，这次是使用 for-comprehension：

```java
def goFishing(bestBaitForFish: FOption[Future, String]): FOption[Future, Fish] = for {
  name <- bestBaitForFish
  bait <- buyBaitFO(name)
  line <- castLineFO(bait)
  fish <- hookFishFO(line)
} yield fish
```

最后，这既简洁又清晰！最后的润色将是处理`FOption`这个临时的名称。这个类型的作用是将`Option`转换成我们选择的更高阶的 monadic 效果，通过将一个`Option`包装成我们选择的 monadic 效果。我们可以将其重命名为`OptionTransformer`或简称`OptionT`。

恭喜！我们刚刚实现了一个 monad 转换器。

# Monad 转换器

让我们稍作停顿，回顾一下我们刚才做了什么。

我们做出了一些小的牺牲，增加了我们原始函数返回类型的复杂性，使其成为某种“公因数”类型。这种牺牲相当小，因为在我们的示例以及现实生活中，这通常只是通过将原始函数提升到它们适当的环境中来实现。

我们提出的签名看起来有点尴尬，但这部分是因为我们开始将它们作为具体的实现来开发。实际上，如果我们以更抽象的方式实现，我们的钓鱼组件的用户界面 API 从一开始就应该类似于以下片段：

```java
abstract class FishingApi[F[_]: Monad] {

  val buyBait: String => F[Bait]
  val castLine: Bait => F[Line]
  val hookFish: Line => F[Fish]

  def goFishing(bestBaitForFish: F[String]): F[Fish] = for {
    name <- bestBaitForFish
    bait <- buyBait(name)
    line <- castLine(bait)
    fish <- hookFish(line)
  } yield fish
}
```

这种方法抽象化了效果类型，为我们作为库作者提供了更多的灵活性，并为 API 的用户提供了更多的结构。

这个 API 可以与任何 monad 一起使用。这是一个示例，说明我们可以如何利用我们目前拥有的函数来实现它——返回混合的`Future`和`Optional`结果：

```java
import Transformers.OptionTMonad
import ch09.Monad.futureMonad
import scala.concurrent.ExecutionContext.Implicits.global

// we need to fix the types first to be able to implement concrete fucntions
object Ch10 {
  type Bait = String
  type Line = String
  type Fish = String
}

object Ch10FutureFishing extends FishingApi[OptionT[Future, ?]] with App {

  val buyBaitImpl: String => Future[Bait] = Future.successful
  val castLineImpl: Bait => Option[Line] = Option.apply
  val hookFishImpl: Line => Future[Fish] = Future.successful

  override val buyBait: String => OptionT[Future, Bait] = 
    (name: String) => buyBaitImpl(name).map(Option.apply)
  override val castLine: Bait => OptionT[Future, Line] = 
    castLineImpl.andThen(Future.successful(_))
  override val hookFish: Line => OptionT[Future, Fish] = 
    (line: Line) => hookFishImpl(line).map(Option.apply)

  goFishing(Transformers.optionTunitFuture, String)
}
```

正如之前一样，我们为原始函数实现了门面，所做的只是将它们提升到适当的效果中。而`goFishing`方法可以像以前一样使用——编译器只需要一个 monad 来使它发生，即`OptoinT[Future]`可用的 monad：

例如，在某个时候，底层函数的实现者可以决定它们应该返回 `Try` 而不是现在的 future。这是可以的，因为要求会变化，我们可以在我们的逻辑中相当容易地纳入这个变化：

```java
import scala.util._
object Ch10OptionTTryFishing extends FishingApi[OptionT[Try, ?]] with App {

  val buyBaitImpl: String => Try[Bait] = Success.apply
  val castLineImpl: Bait => Option[Line] = Option.apply
  val hookFishImpl: Line => Try[Fish] = Success.apply

  override val buyBait: String => OptionT[Try, Bait] = 
    (name: String) => buyBaitImpl(name).map(Option.apply)
  override val castLine: Bait => OptionT[Try, Line] = 
    castLineImpl.andThen(Try.apply(_))
  override val hookFish: Line => OptionT[Try, Fish] = 
    (line: Line) => hookFishImpl(line).map(Option.apply)

  goFishingM(Transformers.optionTunitTry, String)

}
```

假设库中的变化是既定的，我们这边唯一需要改变的是：

+   `castLine` 函数的提升方法；它从 `Future.success` 变为 `Try.apply`

+   我们传递给 `goFishing` 函数初始参数包装器的类型参数

我们完成了。我们根本不需要触及我们的钓鱼“业务”逻辑！

在某种意义上，monad transformer “扁平化”了两个 monad，这样在调用 `map` 和 `flatMap` 方法时就可以一次性穿透所有层——因此也在 for-comprehension 中。

目前，我们无法更改“内部”效果的类型——我们只有可用的 `OptionT` monad transformer。但这只是实施另一个 transformer 一次的问题，就像我们处理 monads 一样。更具体地说，让我们看看将基本函数的返回类型从 `Option` 更改为 `Either` 的效果。假设新版本期望使用 `String` 作为不愉快情况的描述；我们会有以下代码：

```java
object Ch10EitherTFutureFishing extends FishingApi[EitherT[Future, String, ?]] with App {

  val buyBaitImpl: String => Future[Bait] = Future.successful
  val castLineImpl: Bait => Either[String, Line] = Right.apply
  val hookFishImpl: Line => Future[Fish] = Future.successful

  override val buyBait: String => EitherT[Future, String, Bait] =
    (name: String) => buyBaitImpl(name).map(l => Right(l): Either[String, Bait])
  override val castLine: Bait => EitherT[Future, String, Line] =
    castLineImpl.andThen(Future.successful(_))
  override val hookFish: Line => EitherT[Future, String, Fish] =
    (line: Line) => hookFishImpl(line).map(l => Right(l): Either[String, Fish])

  goFishing(Transformers.eitherTunitFuture, String, String).value

}
```

`castLineImpl` 的返回类型现在是 `Either[String, Line]`，因为新的要求规定。我们正在进行的提升稍微复杂一些，因为我们需要将 `Either` 的左右两侧的类型传达给编译器。其余的实现与之前相同。

这依赖于我们有一个 `EitherT` 的实例和相应的 monad。我们已经知道如何实现 monad transformer，并且可以立即想出代码。首先，`EitherT` 类，与 `OptionT` 几乎完全相同，需要携带 `Either` 左侧的类型如下：

```java
implicit class EitherT[F[_]: Monad, L, A](val value: F[Either[L, A]]) {
  def composeB: EitherT[F, L, B] = {
    val result: F[Either[L, B]] = value.flatMap {
      case Left(l) => Monad[F].unit(LeftL, B)
      case Right(a) => f(a).value
    }
    new EitherT(result)
  }
  def isRight: F[Boolean] = Monad[F].map(value)(_.isRight)
}
```

我们不再在 `None` 和 `Some` 上进行模式匹配，而是在 `Either` 的 `Left` 和 `Right` 两侧进行模式匹配。我们还用更合适的 `isRight` 替换了辅助方法 `isEmpty`。

提升函数和 monad 的实现也有相当大的相似性——如果你愿意，就是一些样板代码：

```java
def eitherTunit[F[_]: Monad, L, A](a: => A) = new EitherTF, L, A))

implicit def EitherTMonad[F[_] : Monad, L]: Monad[EitherT[F, L, ?]] = 
  new Monad[EitherT[F, L, ?]] {
    override def unitA: EitherT[F, L, A] =
      Monad[F].unit(ch09.Monad.eitherMonad[L].unit(a))
    override def flatMapA, B(f: A => EitherT[F, L, B]): EitherT[F, L, B] =
      a.compose(f)
}
```

太棒了！我们现在有两个 monad transformer 在我们的工具箱中，之前损坏的 `Ch10EitherTFutureFishing` 定义已经开始编译和运行了！

急切地想要实现 `TryT` 来巩固新获得的知识？我们很高兴把这个练习留给你。

# monad transformer 堆栈

同时，我们将用以下想法自娱自乐：

+   monad transformers 需要一个 monad 的实例作为外层

+   monad transformer 本身有一个 monad

+   如果我们将 monad transformer 作为另一个 monad transformer 的 monad 实例使用，会发生什么不好的事情吗？

让我们试试看。我们已经实现了两个单子转换器，所以让我们将它们放在一起。首先，我们将定义堆栈的类型。它将是`EitherT`包裹在`OptionT`中。这将给我们以下代码的未包装类型：

```java
Future[Either[String, Option[Fish]]]
```

这可以解释为一个耗时操作，可能会在非技术故障的情况下返回错误，并且需要有一个解释（技术故障由失败的`Futures`表示）。`Option`表示一个可以以自然方式返回无结果的操作，无需进一步解释。

使用类型别名，我们可以表示内部转换器的类型，将`String`固定为左侧的类型，如下所示：

```java
type Inner[A] = EitherT[Future, String, A]
```

堆栈中的外层转换器甚至更简单。与内部类型不同，我们固定了效果类型为`Future`，它将一个效果类型构造函数作为类型参数，如下所示：

```java
type Outer[F[_], A] = OptionT[F, A]
```

我们现在可以使用这些别名来定义整个堆栈，如下所示：

```java
type Stack[A] = Outer[Inner, A]
```

为了使情况更现实，我们将只取我们原始钓鱼函数的最后一个版本——即`castLineImpl`返回`Either[String, Line]`的那个版本。我们需要装饰所有原始函数，以便结果类型与我们现在拥有的堆栈类型相匹配。这就是事情开始变得难以控制的地方。编译器不允许连续应用两次隐式转换，因此我们必须手动应用其中之一。对于返回`Future[?]`的两个函数，我们还需要将底层包裹进`Option`中：

```java
override val buyBait: String => Stack[Bait] =
  (name: String) => new EitherT(buyBaitImpl(name).map(l => Right(Option(l)): Either[String, Option[Bait]]))

override val hookFish: Line => Stack[Fish] =
  (line: Line) => new EitherT(hookFishImpl(line).map(l => Right(Option(l)): Either[String, Option[Fish]]))
```

现在，编译器将能够对`OptionT`应用隐式转换。

同样，返回`Either[String, Line]`的函数需要在外部转换为`EitherT`，如下所示：

```java
override val castLine: Bait => Stack[Line] =
  (bait: Bait) => new EitherT(Future.successful(castLineImpl(bait).map(Option.apply)))
```

内部，我们必须将`Either`的内容`map`到一个`Option`中，并将`Future`应用于整个结果。

编译器可以帮助我们通过应用所需的隐式转换来创建适当类型的输入——在这边我们不会看到很多变化，如下所示：

```java
val input = optionTunitInner, String
```

目前我们需要进行一个小调整，因为我们正在使用这个转换器堆栈调用我们的业务逻辑——现在我们有两层转换，所以我们需要调用`value`两次来提取结果，如下所示：

```java
val outerResult: Inner[Option[Fish]] = goFishing(input).value
val innerResult: Future[Either[String, Option[Fish]]] = outerResult.value
```

在每个构成堆栈的单子转换器上反复转向`value`方法可能会很快变得令人厌烦。我们为什么需要这样做呢？因为用特定转换器的类型返回结果可能会很快污染客户端代码。因此，通常有一些关于单子和单子转换器堆栈的建议值得考虑，如下所示：

+   堆叠单子和特别是单子转换器会增加性能和垃圾收集开销。仔细考虑向现有类型添加每个额外效果层的必要性是至关重要的。

+   也有争议说，堆栈中更多的层会增加心理负担并使代码杂乱。这种方法与第一个建议相同——除非绝对需要，否则不要这样做。

+   客户通常不根据单调转换器操作，因此它们（转换器）应被视为实现细节。API 应以通用术语定义。如果需要具体化，则优先考虑效果类型而不是转换器类型。在我们的例子中，返回类型 `Future[Option[?]]` 比返回 `OptionT[Future, ?]` 更好。

考虑到所有这些因素，单调转换器在现实生活中真的有用吗？当然有！然而，就像往常一样，总有一些替代方案，例如自由单调。

# 自由单调

在本章和前几章中，我们使用单调表示了顺序计算。单调的 `flatMap` 方法描述了计算步骤应该如何连接，以及作为参数给出的函数——计算步骤本身。自由单调将顺序计算的概念提升到下一个层次。

首先，我们开始将计算步骤表示为我们选择的某些 **ADT（代数数据类型**） 的实例。其次，我们使用另一个 ADT 的实例表示单调概念。

为了证实这种方法，我们可以再次回到钓鱼的例子。早些时候，我们有三个动作，我们将它们编码为函数。现在，这些动作将被表示为值类。我们还需要给之前使用的类型别名赋予特定的含义，以便以后能够运行示例。

下面是钓鱼模型及其相应的 ADT（代数数据类型）的定义如下：

```java
case class Bait(name: String) extends AnyVal
case class Line(length: Int) extends AnyVal
case class Fish(name: String) extends AnyVal

sealed trait Action[A]
final case class BuyBaitA extends Action[A]
final case class CastLineA extends Action[A]
final case class HookFishA extends Action[A]
```

在模型中，我们表示了诱饵、钓线和鱼的一些属性，以便我们以后可以利用它们。

`Action` 类型有几个值得讨论的方面。首先，`Action` 的实例反映了我们之前的功能通过将此参数声明为类的字段来接受单个参数。其次，所有动作都由 *下一个动作的类型* 类型化，并且这个下一个动作被捕获为类的另一个字段，形式为一个函数，该函数期望包装动作的结果作为参数。第二个字段是我们编码动作顺序的方式。

现在我们需要将单调方法表示为类。

`Done` 以与 `Monad.unit` 相同的方式从值组装 `Free` 实例：

```java
final case class Done[F[_]: Functor, A](a: A) extends Free[F, A]
```

`F[_]` 指的是要包装的动作的类型，`A` 是结果类型。`F` 需要有一个 `Functor`；我们将在稍后看到原因。

`Join` 构建了 `flatMap` 的表示——它应该通过将 `F` 应用到 `Free` 的前一个实例来实现这一点。这给我们以下类型的 `action` 参数如下：

```java
final case class Suspend[F[_]: Functor, A](action: F[Free[F, A]]) extends Free[F, A]
```

现在，正如我们所说的，这是一个单调，因此我们需要提供一个 `flatMap` 的实现。我们将在 `Free` 上这样做，以便可以在 for-comprehensions 中使用 `Done` 和 `Join` 的实例，如下所示：

```java
class Free[F[_]: Functor, A] {
  def flatMapB: Free[F, B] = this match {
    case Done(a) => f(a)
    case Join(a) => Join(implicitly[Functor[F]].map(a)(_.flatMap(f)))
  }
}
```

`flatMap`自然接受 Kleisli 箭头作为参数。类似于其他单子上的`flatMap`定义，例如`Option`，我们区分了短路和退出以及继续计算链。在前一种情况下，我们可以直接应用给定的函数；在后一种情况下，我们必须构建序列。这就是我们使用`Functor[F]`进入`F`并在包装的`Free[F, A]`上应用`flatMap`的地方，基本上是以古老的单子方式执行序列化。

函子在这里提供给我们成功进行计算的可能性，这决定了我们的操作函子应该如何实现——给定的函数应该在下一个操作的结果上被调用。我们的操作可能有相当不同的结构，因此描述这种方法的最简单方式是模式匹配，如下所示：

```java
implicit val actionFunctor: Functor[Action] = new Functor[Action] {
  override def mapA, B(f: A => B): Action[B] = in match {
    case BuyBait(name, a) => BuyBait(name, x => f(a(x)))
    case CastLine(bait, a) => CastLine(bait, x => f(a(x)))
    case HookFish(line, a) => HookFish(line, x => f(a(x)))
  }
}
```

我们 ADT 的值以类似的结构组织，这也是为什么所有操作看起来相似的原因。

我们需要的最后一个准备步骤是有一个用户友好的方式来为每个操作创建自由单子的实例。让我们以下面的方式创建辅助方法：

```java
def buyBait(name: String): Free[Action, Bait] = Join(BuyBait(name, bait => Done(bait)))
def castLine(bait: Bait): Free[Action, Line] = Join(CastLine(bait, line => Done(line)))
def hookFish(line: Line): Free[Action, Fish] = Join(HookFish(line, fish => Done(fish)))
```

这些方法中的每一个都创建了一个自由单子实例，它描述了一个由单个操作组成的计算；`Done(...)`编码了我们已经完成，并且有一些结果的事实。

现在，我们可以使用这些辅助函数来构建一个计算链，就像我们之前做的那样。但这次的计算不会是一个可调用的东西——它只是将自由单子的实例作为单个`Free`实例捕获的实例序列，如下所示：

```java
def catchFish(baitName: String): Free[Action, Fish] = for {
  bait <- buyBait(baitName)
  line <- castLine(bait)
  fish <- hookFish(line)
} yield fish
```

这个单一的实例包含了所有步骤，以`Free`包含操作的形式。以伪代码的形式表示，调用此方法的结果将看起来像嵌套结构，如下所示：

```java
Join(BuyBait("Crankbait", Join(CastLine(bait, Join(HookFish(line, Done(fish)))))))
```

在这个时候，我们已经创建了计算序列，但这个序列是无用的，因为它只是一个数据结构。我们需要一种方法让它变得有用——我们必须为它创建一个解释器。这正是自由单子真正开始发光的地方——如何呈现这些数据取决于我们。我们可以创建尽可能多的解释器，例如，一个用于测试目的，另一个用于生产使用。例如，对于测试，仅仅收集应该在某个日志中发生的所有操作可能是有用的——以事件源的方式（我们将在本书的后面详细探讨事件源）。因为我们只是在测试，所以我们的日志不需要持久化——因此，我们可以使用某种类型的集合；例如，一个`List`就足够了，如下所示：

```java
@tailrec
def goFishingAccA: List[AnyVal] = actions match {
  case Join(BuyBait(name, f)) =>
    val bait = Bait(name)
    goFishingAcc(f(bait), bait :: log)
  case Join(CastLine(bait, f)) =>
    val line = Line(bait.name.length)
    goFishingAcc(f(line), line :: log)
  case Join(HookFish(line, f)) =>
    val fish = Fish(s"CatFish from ($line)")
    goFishingAcc(f(fish), fish :: log)
  case Done(_) => log.reverse
}
```

前面的片段确实是一个程序的解释器，该程序是用`Free`中封装的操作构建的。逻辑是重复的——我们产生操作的结果，并递归地调用这个操作，将带有新增条目的日志作为参数传递。在`Done`的情况下，我们忽略结果；我们的目标是日志，我们通过调用`.reverse`来以相反的方向构建它，并返回反转的形式。

执行的结果看起来如下所示：

```java
scala> import ch10.FreeMonad._
import ch10.FreeMonad._
scala> println(goFishingAcc(catchFish("Crankbait"), Nil))
List(Bait(Crankbait), Line(9), Fish(CatFish from (Line(9))))
```

对于生产环境，我们可以做些其他事情，例如收集执行的动作。我们将通过写入控制台来模拟这种副作用，如下所示：

```java
def logA: Unit = println(a)

@scala.annotation.tailrec
def goFishingLoggingA: A = actions match {
  case Join(BuyBait(name, f)) =>
    goFishingLogging(f(Bait(name)), log(s"Buying bait $name"))
  case Join(CastLine(bait, f)) =>
    goFishingLogging(f(Line(bait.name.length)), log(s"Casting line with ${bait.name}"))
  case Join(HookFish(line, f)) =>
    goFishingLogging(f(Fish("CatFish")), log(s"Hooking fish from ${line.length} feet"))
  case Done(fish) => fish
}
```

这个解释器的结构自然与之前相同。计算的输出类型是`Unit`——我们做的所有事情都有副作用，所以没有必要传递任何东西。我们不是将操作累积到日志中，而是直接将报告写入控制台。`Done`的情况也略有不同——我们返回`fish`，即执行组合操作的结果。

执行的结果如预期的那样发生变化，如下所示：

```java
scala> println(goFishingLogging(catchFish("Crankbait"), ()))
Buying bait Crankbait
Casting line with Crankbait
Hooking fish from 9 feet
Fish(CatFish)
```

我们成功实现了一个非常基本的自由 monad 版本，以及一个小型的钓鱼语言和两个不同的解释器。代码量相当大，所以是时候回答一个明显的问题了：我们为什么投入额外的努力？

自由 monad 具有明显的优势；我们提到了这些，它们如下：

+   将计算作为类粘合在一起发生在堆上，并节省了栈内存。

+   可以将计算传递到代码的不同部分，并且副作用将延迟到显式运行时。

+   有多个解释器允许在不同情况下有不同的行为。

+   本章的范围没有允许我们展示不同的“语言”（ADTs）如何组合成一个代数结构，然后可以使用这个结构同时使用两种语言来定义逻辑。这种可能性为 monad 变换和 monad 变换堆栈提供了替代方案，例如，一种结合业务术语和持久性术语的语言。

它们的缺点与 monads 的缺点处于同一层面。这包括额外的初始实现工作量、垃圾收集器的运行时开销，以及对于新接触这个概念的开发者来说，处理额外的指令和心理负担。

# 摘要

Monads 可以说是函数式编程中最普遍的抽象。不幸的是，它们在一般情况下不能组合——与函数和应用不同。

Monad 变换提供了一种绕过这种限制的方法，通过指定一组总体结构来表示 monads 的组合，每个组合都针对单个内部效应类型。Monad 变换以这种方式组合 monads，使得可以通过一次`flatMap`或`map`调用同时跨越两种效应。

Monad transformer stacks 将 monad transformer 的概念提升了一个层次，利用了每个 monad transformer 同时也是一个 monad 的这一事实。通过堆叠 monad transformer，我们可以像处理单个 monad 一样，以相同的方式在单个堆栈中组合几乎任何数量的效果。

Monad transformer 并非没有缺点。该列表包括由于需要在堆栈中解包和重新打包效果而增加的垃圾收集足迹和处理器利用率。同样的推理也适用于开发者在编写和维护代码时需要在脑海中构建和维护的心理模型。

Free monad 通过明确分离计算的结构和解释，提供了一个合理的替代方案。它是通过将业务逻辑表示为某些 ADT 编码的步骤序列来实现的，并使用合适的解释器执行这些步骤。

本章总结了本书的第二部分。在本部分和第一部分中，我们避免使用第三方库，专注于向读者传授对语言特性和底层理论概念的深入理解。

不言而喻，本部分中的代码示例无疑是简化的，仅适用于学习目的。

特别针对函数式编程方面，有两个特别好的库值得再次提及，并且可用于 Scala：Cats ([`typelevel.org/cats/`](https://typelevel.org/cats/)) 和 Scalaz ([`github.com/scalaz/scalaz`](https://github.com/scalaz/scalaz))。如果我们成功地激发了您使用本书本部分展示的函数式风格编程 Scala 的兴趣，我们强烈推荐您查看这两个库。除了提供我们研究的概念的生产就绪实现外，它们还包含了许多我们没有讨论的抽象。

在本书的第三部分，我们将放宽对第三方依赖的自我施加的限制，并将其致力于使用不同的 Akka 库在 Scala 中进行响应式编程的主题。

# 问题

1.  为什么 monad transformer 的类型反映了堆栈类型的“倒置”，其名称指的是最内层 monad 的类型？

1.  为什么可以在堆栈的顶层重用现有的 monad？

1.  为什么不能在堆栈的底层重用现有的 monad？

1.  实现 `TryT` monad transformer。

1.  在本章的示例函数中使用 `TryT` monad transformer 而不是 `EitherT`。

1.  实现另一种 monad transformer stack 的实现，这次将层放置在上方：`EitherT[OptionT[Future, A], String, A]`。

1.  在本章开发的 free monad 示例中添加一个释放捕获的鱼的动作。

# 进一步阅读

Anatolii Kmetiuk，*精通函数式编程*：学习函数式编程如何帮助你以声明性和纯方式部署网络服务器和与数据库交互。

Atul S. Khot，《Scala 函数式编程模式》：掌握并执行有效的 Scala 函数式编程

Ivan Nikolov，《Scala 设计模式》—— 第二版：学习如何使用 Scala 编写高效、简洁且可重用的代码
