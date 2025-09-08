# 评估

# 第一章

1.  **描述两种使某些资源`R`能够与`scala.util.Using`资源管理实用程序一起使用的方法。**

让`R`扩展`java.lang.AutoCloseable`。这将允许将现有的从`AutoCloseable`到`Resource`的隐式转换应用于`R`。

提供对`Resource[R]`的隐式实现。

1.  **如何比较`Set`和`List`？**

`Set`和`List`之间没有定义等性，因此我们必须以以下两种方式之一使用`sameElements`方法，直接在`List`上或在`Set`的迭代器上，如下面的代码片段所示：

```java
val set = Set(1,2,3)
val list = List(1,2,3)

set == list // false

set.iterator.sameElements(list) // true

list.sameElements(set) // true

```

另一种可能性是利用`corresponds`操作与等性检查函数的组合。这在两个方向上工作方式相似：

```java
scala> set.corresponds(list)(_ == _)
res2: Boolean = true
scala> list.corresponds(set)(_ == _)
res3: Boolean = true
```

1.  **为不可变的`Seq`命名默认的具体实现。**

`scala.collection.immutable.List`

1.  **为不可变的索引`Seq`命名默认的具体实现。**

`scala.collection.immutable.Vector`

1.  **为可变的`Seq`命名默认的具体实现。**

`scala.collection.mutable.ArrayBuffer`

1.  **为可变的`IndexedSeq`命名默认的具体实现。**

`scala.collection.mutable.ArrayBuffer`

1.  **有时人们会说** `List.flatMap` **比预期的更强大。你能尝试解释一下为什么吗？**

`flatMap`是在`IterableOnce`上定义的，因此它的参数是一个返回`IterableOnce`的函数。正因为如此，在`flatMap`操作时可以混合不同的类型。考虑以下示例，其中`List`能够将`Set[Int]`和其元素进行`flatMap`操作：

```java
scala> List(1,2,3,4,5,6).flatMap(i => if (i<3) Set.fill(i)(i) else Seq.fill(i)(i))
res28: List[Int] = List(1, 2, 3, 3, 3, 4, 4, 4, 4, 5, 5, 5, 5, 5, 6, 6, 6, 6, 6, 6)
```

1.  **描述一种使用不同函数多次映射集合的方法，但又不产生中间集合。**

创建一个视图，按需映射视图，并强制将其转换回原始表示类型：

```java
scala> List(1,2,3,4,5).view.flatMap(List.fill(_)("a")).map(_.toUpperCase).toList
res33: List[String] = List(A, A, A, A, A, A, A, A, A, A, A, A, A, A, A)
```

# 第二章

1.  **你能命名哪些类型约束？**

有两个约束：下界或子类型关系和上界或超类型关系。

1.  **如果没有开发者为类型定义类型约束，那么会添加哪些隐式类型约束到该类型中？**

对于缺少上界，编译器添加`Any`作为约束，对于缺少下界则添加`Nothing`。

1.  **哪些运算符可以用来引用某些类型的嵌套类型？**

有两个运算符。`A#B`的概念指的是`A`类型的嵌套类型。`a.B`的概念指的是实例`a`的`B`子类型。

1.  **哪种类型可以用作中缀类型？**

任何恰好由两个类型参数参数化的类型。

1.  **为什么在 Scala 中不建议使用结构化类型？**

结构化类型的用法通常会导致生成的字节码，它通过反射访问方法，这比正常方法调用要慢。

1.  **通过变异性表达了什么？**

参数化类型和参数化类型之间的子类型关系的相关性。

# 第三章

1.  **以下函数在柯里化形式中的类型将是什么：`(Int, String) => (Long, Boolean, Int) => String`？**

`Int => (String => ((Long, Boolean, Int) => String)))` 或简化为 `Int => String => (Long, Boolean, Int) => String`

1.  **描述部分应用函数和部分函数之间的区别**。

部分函数未定义在某些可能的输入值上。部分应用函数将其一些参数固定为特定值。

1.  **定义一个签名并实现一个函数，`uncurry`，用于三个参数的柯里化函数，`A => B => C => R`**。

`def uncurryA,B,C,R: (A,B,C) => R = (a,b,c) => in(a)(b)(c)`

1.  **实现一个用于阶乘计算的头部递归函数（n! = n * (n-1) * (n-2) * ... * 1**）。

`def factorial(n: Long): Long = if (n < 2) n else n * factorial(n-1)`

1.  **实现一个用于阶乘计算的尾递归函数**。

```java
def factorial(n: Long): Long = {
  def f(n: Long, acc: Long): Long = if (n < 2) acc else f(n-1, n * acc)
  f(n,1)
}
```

1.  **实现一个使用跳跃递归的阶乘计算的递归函数**。

```java
import util.control.TailCalls._
def factorial(n: Long): TailRec[Long] = if (n<2) done(n) else tailcall(factorial(n-1)).map(_ * n)
```

# 第四章

1.  **描述一个隐式参数也是隐式转换的情况**。

如果隐式参数是一个函数，则这是这种情况：`def funcA, T(implicit adapter: A => T): T = adapter(a)`。

1.  **将以下使用视图边界的定义替换为使用上下文边界的定义：`def compare[T <% Comparable[T]](x: T, y: T) = x < y`**。

```java
type ComparableContext[T] = T => Comparable[T]
def compareT : ComparableContext = x < y
```

1.  **为什么有时说类型类将行为和数据分开？**

因为类型类实例定义了逻辑以工作，并且数据来自类型类应用到的值。

很容易改变可能的冲突的示例，以便其中一个隐式参数胜过其他隐式参数，并且所有其他隐式参数都可以取消注释而不再有冲突。进行此更改。例如，可以通过将`val`的定义之一更改为`object`的定义来完成：

```java
package object resolution {
  // change // implicit val a: TS = new TS("val in package object") // (1)
  // to     //
  implicit object TSO extends TS("object in package object") // (1)
}
```

然后，由于静态解析规则，TSO 将比其他值更具体，并且将由编译器为应用程序选择。

# 第五章

1.  **定义一个用于排序列表的不变性质**。

```java
def invariant[T: Ordering: Arbitrary]: Prop =
  forAll((l: List[T]) => l.sorted.length == l.length)

scala> invariant[Long].check
+ OK, passed 100 tests.

scala> invariant[String].check
+ OK, passed 100 tests.
```

1.  **定义一个用于排序列表的幂等性质**。

```java
def idempotent[T: Ordering: Arbitrary]: Prop =
  forAll((l: List[T]) => l.sorted.sorted == l.sorted)

scala> idempotent[Long].check
+ OK, passed 100 tests.

scala> idempotent[String].check
+ OK, passed 100 tests.
```

1.  **定义一个用于排序列表的归纳性质**。

```java
def inductive[T: Ordering: Arbitrary]: Prop = {
  def ordered(l: List[T]): Boolean =
    (l.length < 2) || 
      (ordered(l.tail) && implicitly[Ordering[T]].lteq(l.head, l.tail.head))
  forAll((l: List[T]) => ordered(l.sorted))
}

scala> inductive[Int].check
+ OK, passed 100 tests.

scala> inductive[String].check
+ OK, passed 100 tests.
```

1.  **定义一个用于`List[List[Int]]`的生成器，使得嵌套列表的元素都是正数**。

```java
val genListListInt = Gen.listOf(Gen.listOf(Gen.posNum[Int]))

scala> genListListInt.sample
res35: Option[List[List[Int]]] = Some(List(List(60, 99, 5, 68, 52, 98, 31, 29, 30, 3, 91, 54, 88, 49, 97, 2, 92, 28, 75, 100, 100, 38, 16, 2, 86, 41, 4, 7, 43, 70, 21, 72, 90, 59, 69, 43, 88, 35, 57, 67, 88, 37, 4, 97, 51, 76, 69, 79, 33, 53, 18), List(85, 23, 4, 97, 7, 50, 36, 24, 94), List(97, 9, 25, 34, 29, 82, 59, 24, 94, 42, 34, 80, 7, 79, 44, 54, 61, 84, 32, 14, 9, 17, 95, 98), List(4, 70, 13, 18, 42, 74, 63, 21, 58, 4, 32, 61, 52, 77, 57, 40, 37, 54, 11), List(9, 22, 33, 19, 56, 29, 45, 34, 61, 48, 42, 56, 64, 96, 56, 77, 58, 90, 30, 48, 32, 49, 80, 58, 65, 5, 24, 88, 27, 44, 15, 5, 65, 11, 14, 80, 30, 5, 23, 31, 38, 55, 1, 94, 15, 89, 69, 23, 35, 45, 38, 96, 11, 35, 22, 90, 46, 39, 69, 11, 26, 53, 18, 23, 8, 85, 22, 12, 49, 79, 63, 39, 1, 89, 68, 91, 24...
```

1.  **定义一个用于`Map[UUID, () => String]`的生成器**。

```java
val pairGen = for {
  uuid <- Gen.uuid
  function0 <- Gen.function0(Gen.asciiStr)
} yield (uuid, function0)

val mapGen = Gen.mapOf(pairGen)
mapGen: org.scalacheck.Gen[Map[java.util.UUID,() => String]] = org.scalacheck.Gen$$anon$1@16ca4e8d

scala> mapGen.sample
res36: Option[Map[java.util.UUID,() => String]] = Some(Map(31395a9b-78af-4f4a-9bf3-c19b3fb245b6 -> org.scalacheck.Gen$$$Lambda$2361/1400300928@178b18c, ...
```

请注意，`Gen.function0`生成一个零参数的函数，该函数仅返回由提供的生成器生成的随机值。

# 第六章

1.  **表示获取以下每个元素的适当效果是什么：**

+   **某些列表的第一个元素**：`Option[?]`，其中`None`表示空列表没有头元素

+   **推文列表**：`Future[List[Tweet]]`，因为操作可能需要一些时间，因为它通过网络进行

+   **使用数据库中的用户信息** `userId`：`Future[Option[?]]`，其中`Future`表示网络调用，`Option`表示对于给定的`userId`没有用户账户

1.  **以下表达式的可能值范围是什么：** `Option(scala.util.Random.nextInt(10)).fold(9)(_-1)`

一个包含 [-1;9] 的区间

1.  **以下表达式的结果会是什么：**

```java
TryInt).filter(_ > 10).recover {
  case _: OutOfMemoryError => 100
}(20)
```

`Try` 构造函数不会捕获 `OutOfMemoryError`，因此给定的表达式将抛出 `OutOfMemoryError`。

1.  **描述以下表达式的结果：**

```java
FutureInt).filter(_ > 10).recover {
  case _: OutOfMemoryError => 100
}
```

表达式的结果将是 `Future(<未完成>)`，最终会抛出 `OutOfMemoryError`，就像上一个案例一样。

1.  **给定以下函数：**

```java
def either(i: Int): Boolean = 
  Either.cond(i > 10, i * 10, new IllegalArgumentException("Give me more")).forall(_ < 100)
```

1.  **以下调用会有什么结果：** `either(1)`？

结果将是 `true`，因为 `Either.cond` 在 `i == 2` 时评估为 `Left`，而 `Left.forall` 对任何 `Left` 评估为 `true`。

# 第七章

1.  **为什么结合律对于幺半群在分布式设置中有用是本质的？**

在分布式设置中，我们通常讨论的是折叠和重用数据集，其中部分数据由不同的计算机处理。幺半群操作应用于远程机器。无论它们是从主机器发送的顺序如何，网络延迟、不同的负载模式和硬件设置都将影响它们返回的顺序。能够在不等待第一次操作完成的情况下，对已经掌握的中间结果应用操作是很重要的。

1.  **在 `OR` 下实现 `Boolean` 的幺半群。**

实现如下：

```java
implicit val booleanOr: Monoid[Boolean] = new Monoid[Boolean] {
  override def identity: Boolean = false
  override def op(l: Boolean, r: Boolean): Boolean = l || r
}
```

属性如下::

```java
property("boolean under or") = {
  import Assessment.booleanOr
  monoidProp[Boolean]
}
```

1.  **在 `AND` 下实现 `Boolean` 的幺半群。**

实现如下：

```java
implicit val booleanAnd: Monoid[Boolean] = new Monoid[Boolean] {
  override def identity: Boolean = true
  override def op(l: Boolean, r: Boolean): Boolean = l && r
}
```

属性如下：

```java
property("boolean under and") = {
  import Assessment.booleanAnd
  monoidProp[Boolean]
}
```

1.  **给定 `Monoid[A]`，实现 `Monoid[Option[A]].`**

实现如下：

```java
implicit def option[A : Monoid]: Monoid[Option[A]] = new Monoid[Option[A]] {
  override def identity: Option[A] = None
  override def op(l: Option[A], r: Option[A]): Option[A] = (l, r) match {
    case (Some(la), Some(lb)) => Option(implicitly[Monoid[A]].op(la, lb))
    case _ => l orElse r
  }
}
```

属性如下：

```java
property("Option[Int] under addition") = {
  import Monoid.intAddition
  import Assessment.option
  monoidProp[Option[Int]]
}

property("Option[String] under concatenation") = {
  import Monoid.stringConcatenation
  import Assessment.option
  monoidProp[Option[String]]
}
```

1.  **给定 `Monoid[R]`，实现 `Monoid[Either[L, R]].`**

实现如下：

```java
def either[L, R : Monoid]: Monoid[Either[L, R]] = new Monoid[Either[L, R]] {
  private val ma = implicitly[Monoid[R]]
  override def identity: Either[L, R] = Right(ma.identity)
  override def op(l: Either[L, R], r: Either[L, R]): Either[L, R] = (l, r) match {
    case (l @ Left(_), _) => l
    case (_, l @ Left(_)) => l
    case (Right(la), Right(lb)) => Right(ma.op(la, lb))
  }
}
```

属性如下：

```java
property("Either[Int] under multiplication") = {
  import Monoid.intMultiplication
  implicit val monoid: Monoid[Either[Unit, Int]] = Assessment.either[Unit, Int]
  monoidProp[Either[Unit, Int]]
}

property("Either[Boolean] under OR") = {
  import Assessment.booleanOr
  implicit val monoid: Monoid[Either[String, Boolean]] = Assessment.either[String, Boolean]
  monoidProp[Either[String, Boolean]]
}
```

1.  **泛化前两个实现以适用于任何由 `A` 参数化的效果，或描述为什么不可能。**

不幸的是，在一般情况下无法实现这样的幺半群，因为实现将需要两个方面：

+   新幺半群的单位元素

+   检查一个效果是否为空，并在它不为空时检索一个元素的可能性

可以将单位元素作为参数传递给构造函数，但这样就没有办法按照第二点的要求与现有的效果一起工作。

# 第八章

1.  **实现** `Functor[Try]`。

```java
implicit val tryFunctor: Functor[Try] = new Functor[Try] {
  override def mapA, B(f: A => B): Try[B] = in.map(f)
  override def mapCA, B: Try[A] => Try[B] = fa => map(fa)(f)
}
```

1.  **实现** `Applicative[Try]`。

```java
implicit val tryApplicative: Applicative[Try] = new Applicative[Try] {
  override def applyA, B(f: Try[A => B]): Try[B] = (a, f) match {
    case (Success(a), Success(f)) => Try(f(a))
    case (Failure(ex), _) => Failure(ex)
    case (_, Failure(ex)) => Failure(ex)
  }
  override def unitA: Try[A] = Success(a)
}
```

1.  **实现** `Applicative[Either]`。

```java
implicit def eitherApplicative[L] = new Applicative[({ type T[A] = Either[L, A] })#T] {
  override def applyA, B(f: Either[L, A => B]): Either[L, B] = (a, f) match {
    case (Right(a), Right(f)) => Right(f(a))
    case (Left(l), _) => Left(l)
    case (_, Left(l)) => Left(l)
  }
  override def unitA: Either[L, A] = Right(a)
}
```

1.  **实现** `Traversable[Try]`。

```java
implicit val tryTraversable = new Traversable[Try] {
  override def mapA, B(f: A => B): Try[B] = Functor.tryFunctor.map(in)(f)
  override def traverse[A, B, G[_] : Applicative](a: Try[A])(f: A => G[B]): G[Try[B]] = {
    val G = implicitly[Applicative[G]]
    a match {
      case Success(s) => G.map(f(s))(Success.apply)
      case Failure(ex) => G.unit(Failure(ex)) // re-wrap the ex to change the type of Failure
    }
  }
}
```

1.  **实现** `Traversable[Either]`。

```java
implicit def eitherTraversable[L] = new Traversable[({ type T[A] = Either[L, A] })#T] {
  override def mapA, B(f: A => B): Either[L, B] = 
    Functor.eitherFunctor[L].map(in)(f)
  override def traverse[A, B, G[_] : Applicative](a: Either[L, A])(f: A => G[B]): G[Either[L, B]] = {
    val G = implicitly[Applicative[G]]
    a match {
      case Right(s) => G.map(f(s))(Right.apply)
      case Left(l) => G.unit(Left(l)) // re-wrap the l to change the type of Failure
    }
  }
}
```

1.  **实现** `Traversable.compose`。

```java
trait Traversable[F[_]] extends Functor[F] {
  def traverse[A,B,G[_]: Applicative](a: F[A])(f: A => G[B]): G[F[B]]
  def sequence[A,G[_]: Applicative](a: F[G[A]]): G[F[A]] = traverse(a)(identity)

  def compose[H[_]](implicit H: Traversable[H]): Traversable[({type f[x] = F[H[x]]})#f] = {
    val F = this
    new Traversable[({type f[x] = F[H[x]]})#f] {
      override def traverse[A, B, G[_] : Applicative](fa: F[H[A]])(f: A => G[B]) =
        F.traverse(fa)((ga: H[A]) => H.traverse(ga)(f))

      override def mapA, B(f: A => B): F[H[B]] =
        F.map(in)((ga: H[A]) => H.map(ga)(f))
    }
  }
}
```

# 第九章

1.  **实现** `Monad[Try]`。

```java
implicit val tryMonad = new Monad[Try] {
  override def unitA: Try[A] = Success(a)

  override def flatMapA, B(f: A => Try[B]): Try[B] = a match {
    case Success(value) => f(value)
    case Failure(ex) => Failure(ex)
  }
}
```

1.  **证明 `State` 幺半群的右单位律。**

让我们从本章中已有的属性定义开始：

```java
val rightIdentity = forAll { (a: A, f: A => M[B]) =>
  M.flatMap(M.unit(a))(f) == f(a)
}
```

设 `f(a) = a => State(s => (b, s2))`

首先，我们将单位定义的值替换为调用结果。因此，`M.flatMap(M.unit(a))(f)` 变为 `M.flatMap(State(s => (a, s)))(f)`。

接下来，我们将`M.flatMap`替换为`compose`，这给我们`State(s => (a, s)).compose(f).`

接下来，我们将使用本章中证明的引理，用其定义替换`compose`调用：

```java
State(s => { 
  val (a, nextState) = (a, s)
  f(a).run(nextState)
}
```

通过应用`f`，之前的代码可以简化为`State(s => State(s => (b, s2)).run(s)`，进一步简化为`State(s => (b, s2))`。**(1)**

方程式的右侧，`f(a)`，根据定义等于`State(s => (b, s2))`。**(2)**

我们有(1) == (2)，因此证明了状态单子的右单位律。

1.  **在本章定义的单子中选择一个，并实现一个`go`函数，该函数将以 1%的概率编码船沉没的概念。**

`Option`将代表船沉没的概念：

```java
import Monad.optionMonad

def go(speed: Float, time: Float)(boat: Boat): Option[Boat] = 
  if (Random.nextInt(100) == 0) None
  else Option(boat.go(speed, time))

println(move(go, turn[Option])(Option(boat)))
```

1.  **请同样操作，但以 1%的概率编码电机在移动中损坏的概念，使船无法移动。**

`Try`和右偏`Either`都可以用来编码电机损坏的情况。

以下是使用`Try`的实现：

```java
import Monad.tryMonad

def go(speed: Float, time: Float)(boat: Boat): Try[Boat] =
  if (Random.nextInt(100) == 0) Failure(new Exception("Motor malfunction"))
  else Success(boat.go(speed, time))

println(move(go, turn[Try])(Success(boat)))
```

以下是使用`Either`的实现：

```java
import Monad.eitherMonad
type ErrorOr[B] = Either[String, B]

def go(speed: Float, time: Float)(boat: Boat): ErrorOr[Boat] =
  if (Random.nextInt(100) == 0) Left("Motor malfunction")
  else Right(boat.go(speed, time))

println(move(go, turn[ErrorOr])(Right(boat)))
```

1.  **使用以下模板（松散地）描述本章中定义的单子的本质：状态单子传递链式计算之间的状态。计算本身接受前一次计算的结果，并返回结果以及新的状态。**

选项单子允许链式计算，这些计算可能不会返回结果。计算会一直进行，直到最后一个或直到第一个返回无结果。

尝试单子与选项单子做同样的事情，但不是通过一个特殊的*无结果*值来中断整个计算链，而是有一个由`Failure`案例类表示的*失败*概念。

两个单子都有类似于选项和尝试单子的语义，但在这个情况下，中断步骤序列的概念由`Left`类型承担，而继续序列的概念由`Right`类型承担。

1.  **定义一个`go`方法，该方法既跟踪船的位置，又使用以下类型的结构来考虑船沉没的可能性：`type WriterOption[B] = Writer[Vector[(Double, Double)], Option[Boat]]`**。

```java
object WriterOptionExample extends App {
  type WriterOption[B] = Writer[Vector[(Double, Double)], Option[B]]
  import WriterExample.vectorMonoid

  // this implementation delegates to the logic we've implemented in the chapter
  def go(speed: Float, time: Float)(boat: Boat): WriterOption[Boat] = {
    val b: Option[Boat] = OptionExample.go(speed, time)(boat)
    val c: WriterTracking[Boat] = WriterExample.go(speed, time)(boat)
    Writer((b, c.run._2))
  }

  // constructor - basically unit for the combined monad
  private def writerOptionA =
    Writer[Vector[(Double, Double)], Option[A]](Option(a))

  // we need a monad of the appropriate type
  implicit val readerWriterMonad = new Monad[WriterOption] {
    override def flatMapA, B(f: A => WriterOption[B]): WriterOption[B] =
      wr.compose {
        case Some(a) => f(a)
        case None => Writer(Option.empty[B])
      }

    override def unitA: WriterOption[A] = writerOption(a)
  }
  // tracks boat movement until it is done navigating or sank
  println(move(go, turn)(writerOption(boat)).run)
}
```

1.  **比较第 6 题的答案和我们在上一章中组合应用的方式。**

在第八章，*处理效果*中，我们实现了一个通用的组合子用于应用。在这个涉及单子的实现中，我们需要了解如何剖析选项的效果，以便能够实现组合逻辑。请阅读第十章，*单子变换器和自由单子的观察*，以获取更多详细信息。

# 第十章

1.  **为什么单子变换器的类型反映了栈类型的“颠倒”？**

在一般情况下，无法定义单子组合，只有在其堆栈内部效果特定的情况下才能定义。因此，效果的名字被固定在转换器的名字中，外部效果成为类型参数。

1.  **为什么可以在堆栈顶层重用现有的单子？**

Kleisli 箭的返回类型与堆栈的类型很好地匹配。因此，可以通过利用外部单子的 `flatMap` 方法来产生正确类型的正确结果。

1.  **为什么不能在堆栈底层重用现有的单子？**

箭的参数类型期望一个普通参数。因此，我们需要从内部效果上下文中提取无效果的价值。这只能在特定方式下实现，而不能在一般方式下实现。

1.  **实现 `TryT` 单子转换器。**

```java
private def noResultTryT[F[_] : Monad, T](ex: Throwable): F[Try[T]] = 
  Monad[F].unit(FailureT)

implicit class TryT[F[_] : Monad, A](val value: F[Try[A]]) {
  def composeB: TryT[F, B] = {
    val result = value.flatMap {
      case Failure(ex) => noResultTryTF, B
      case Success(a) => f(a).value
    }
    new TryT(result)
  }

  def isSuccess: F[Boolean] = Monad[F].map(value)(_.isSuccess)
}

def tryTunit[F[_] : Monad, A](a: => A) = new TryT(Monad[F].unit(Try(a)))

implicit def TryTMonad[F[_] : Monad]: Monad[TryT[F, ?]] = new Monad[TryT[F, ?]] {
  override def unitA: TryT[F, A] = Monad[F].unit(Monad[Try].unit(a))
  override def flatMapA, B(f: A => TryT[F, B]): TryT[F, B] = a.compose(f)
}
```

1.  **请使用 `TryT` 单子转换器代替本章中的示例函数中的 `EitherT`。**

```java
object Ch10FutureTryFishing extends FishingApi[TryT[Future, ?]] with App {
  val buyBateImpl: String => Future[Bate] = ???
  val castLineImpl: Bate => Try[Line] = ???
  val hookFishImpl: Line => Future[Fish] = ???

  override val buyBate: String => TryT[Future, Bate] = 
    (name: String) => buyBateImpl(name).map(Try(_))
  override val castLine: Bate => TryT[Future, Line] = 
    castLineImpl.andThen(Future.successful(_))
  override val hookFish: Line => TryT[Future, Fish] = 
    (line: Line) => hookFishImpl(line).map(Try(_))

  val result: Future[Try[Fish]] = goFishing(tryTunitFuture, String).value
}
```

1.  **实现另一种单子转换器堆栈，这次将层放置颠倒： **`EitherT[OptionT[Future, A], String, A]`。

```java
type Inner[A] = OptionT[Future, A]
type Outer[F[_], A] = EitherT[F, String, A]
type Stack[A] = Outer[Inner, A]

object Ch10EitherTOptionTFutureFishing extends FishingApi[Stack[?]] with App {

  val buyBateImpl: String => Future[Bate] = ???
  val castLineImpl: Bate => Either[String, Line] = ???
  val hookFishImpl: Line => Future[Fish] = ???

  override val castLine: Bate => Stack[Line] =
    (bate: Bate) => new OptionT(Future.successful(Option(castLineImpl(bate))))

  override val buyBate: String => Stack[Bate] =
    (name: String) => new OptionT(buyBateImpl(name).map(l => Option(Right(l)): Option[Either[String, Bate]]))

  override val hookFish: Line => Stack[Fish] =
    (line: Line) => new OptionT(hookFishImpl(line).map(l => Option(Right(l)): Option[Either[String, Fish]]))

  val input: EitherT[Inner, String, String] = eitherTunitInner, String, String
  val outerResult: Inner[Either[String, Fish]] = goFishing(input).value
  val innerResult: Future[Option[Either[String, Fish]]] = outerResult.value

}
```

1.  **将释放捕获的鱼的动作添加到我们在本章中开发的自由单子示例中。**

这里只显示了示例的更改部分。请参阅附带的代码，以查看包含更改的示例：

```java
final case class ReleaseFishA extends Action[A]

def releaseFish(fish: Fish): Free[Action, Unit] = Join(ReleaseFish(fish, _ => Done(())))

implicit val actionFunctor: Functor[Action] = new Functor[Action] {
  override def mapA, B(f: A => B): Action[B] = in match {
    ... // other actions as before  
    case ReleaseFish(fish, a) => ReleaseFish(fish, x => f(a(x)))
  }
}

def catchFish(bateName: String): Free[Action, _] = for {
    bate <- buyBate(bateName)
    line <- castLine(bate)
    fish <- hookFish(line)
    _ <- releaseFish(fish)
} yield ()

def goFishingLoggingA: A = actions match {
    ... // the rest as in the chapter code
    case Join(ReleaseFish(fish, f)) =>
      goFishingLogging(f(()), log(s"Releasing the fish $fish"))
}

def goFishingAccA: List[AnyVal] = actions match {
    ...
    // the rest as in the chapter code
    case Join(ReleaseFish(fish, f)) =>
      goFishingAcc(f(()), fish.copy(name = fish.name + " released") :: log)
}
```

我们需要扩展动作模型和函数，添加一个辅助提升方法，将额外的步骤添加到进程的定义中，并增强两个解释器以支持新的动作。

# 第十一章

1.  **请列举两种演员可以对其接收到的消息做出反应并改变自己的方式。**

演员可以使用 var 字段来改变其内部状态。这是一种经典的对象导向方法。

另一种方法是使用上下文并围绕某个将成为新状态一部分的值进行成为操作。context.become 也可以用来完全改变演员的行为。这更是一种函数式方法，因为状态和行为实际上都是不可变的。

1.  **`ActorRef` 的目的是什么？**

ActorRef 提供了一种通过演员路径来寻址演员的方法。它还封装了一个演员的邮箱和调度器。Akka 中的演员通过 ActorReference 进行通信。

1.  **在官方文档中查找系统守护者的描述。它的主要目的是什么？**

系统守护者的主要目的是监督系统级演员。它还用于确保适当的关闭顺序，以便在用户守护者终止之前，系统级演员对用户定义的演员可用。

1.  **描述使用 Akka FSM 的优缺点。**

Akka FSM 允许将演员行为建模为状态机，定义这些状态之间的单独状态转换和数据。

Akka FSM 将业务逻辑与特定实现耦合，使得测试和调试变得困难。

1.  **以多少种方式可以访问另一个演员系统中的演员？描述它们。**

在远程系统中访问 actor 有两种方式——远程部署和远程查找。通过远程部署，在远程系统中创建一个新的 actor。远程部署可以在代码中显式执行，或者通过提供部署配置来完成。远程查找允许使用与本地查找相同的方法来选择远程系统中的现有 actor。

1.  **为什么测试 actor 需要特殊的工具集？**

Actors 具有高度的非确定性。actor 的状态是不可访问的。正确测试 actor 的唯一方法是通过向其发送消息并等待其响应。有时需要创建整个 actor 层次结构来完成此目的。

# 第十二章

1.  **`Behavior[Tpe]`定义的意义是什么？**

`Behavior[Tpe]`明确指定了这个 actor 能够处理`Tpe`的子类型消息。通过递归，我们可以得出结论，返回的行为也将是`Behavior[Tpe]`。

1.  **如何在 actor 的行为中获取对调度器的访问？**

调度程序可以通过行为构造函数`Behaviors.withTimers`访问。

1.  **描述 actor 可能被停止的可能方式。**

父 actor 可以使用父 actor 的 actor 上下文来停止 actor：`context.stop(child)`。

actor 也可以通过返回相应的行为来停止自己：`Behaviors.stopped`。

如果 actor 的逻辑抛出了异常，并且为该 actor 定义的`SupervisorStrategy`是`stop`，actor 也可以被停止。

1.  **本地和集群接待员之间的区别是什么？**

虽然有不同的实现方式，但对开发者来说没有明显的区别。

1.  **存在哪些监督可能性，以及它们是如何定义的？**

有三种监督策略：停止、重启和恢复。它们通过使用`Behaviors.supervise`将监督行为包装在 actor 上来定义。

1.  **为什么应该谨慎使用 stashing？**

当前的 stashing 实现将消息缓冲在内存中，可能导致`OutOfMemory`或`StashOverflowException`，具体取决于 stash 的大小。如果消息被移除，actor 将不会产生其他传入的消息，直到所有缓存的位都处理完毕，这可能会使其无响应。

1.  **在独立测试 actor 逻辑方面，有什么推荐的方法？**

使用 BehaviorTestKit 提供的同步测试可以更好地测试 actor 逻辑的独立性。

# 第十三章

1.  **与“经典”流相关联的两个不同模式是什么？为什么它们有问题？**

这两种模式是推送和拉取。在消费者速度较慢的情况下，推送可能会导致流元素丢失或内存溢出。在生产者速度较慢的情况下，拉取可能不理想，因为它可能导致阻塞或大量资源消耗。

1.  **为什么 Reactive Streams 被认为是在动态拉-推模式下工作？**

Reactive Streams 引入了非阻塞背压的概念。消费者报告其需求，生产者根据这个需求批量推送数据。当消费者更快时，需求总是存在，因此生产者总是尽快推送数据。如果有一个比消费者更快的生产者，总是有数据可用，消费者一旦有需求就会立即拉取。流会自动在这些模式之间切换。

1.  **Akka Stream 图的典型构建块是什么？**

流是一个具有一个输入和一个输出的阶段。Fan-In 有多个输入和一个输出。Fan-Out 是相反的，有多个输出和一个输入。BidiFlow 代表双向流，有两个输入和两个输出。

1.  **如何将图转换为可运行的图？**

通过将源和汇连接到图中，可以将一个图连接成一个可运行的图。

1.  **为什么将材料化作为一个独立的显式步骤的主要目标？**

在材料化步骤之前，任何图都可以被视为流的蓝图，因此可以自由共享和重用。

1.  **描述应用不同监督策略的效果。**

有三种不同的监督策略。

停止中断失败的处理阶段的流。失败会向下传播，取消会向上传播。

恢复丢弃当前元素并继续流处理。

重启丢弃当前元素，清理处理阶段的内部状态（通常是通过重新创建它），并继续流处理。

1.  **哪些主要抽象提供了 Akka Streams TestKit？为什么它们是有用的？**

Akka Streams TestKit 提供的两个主要抽象是 TestSink 和 TestSource。它们允许在不同级别上控制和验证关于流流的假设，例如，在高消息级别或低 reactive-streams 级别。它们还使得可以使用一个漂亮的 DSL 来驱动测试，并形成关于结果期望。

# 第十四章

1.  **什么是数据库迁移？**

数据库迁移（或模式迁移）是数据库模式更新的自动管理。模式更改是增量性的，通常是可逆的，并在数据库模式需要更改以反映应用程序代码更改的时刻应用。

1.  **描述在库存不足的情况下，完全丢弃订单的替代方法可能是什么？**

一种可能的替代方案是满足所有有足够库存的文章的订单。这可以通过在每个库存更新中运行单独的事务并组合所有成功的事务的结果来实现。

另一种可能的替代方案是尽可能满足订单。这种方法需要选择更新行，计算新的可能状态，并在同一事务中应用它们。

1.  **描述 http4s 和 Akka HTTP 在定义路由方面的概念差异。**

http4s 将路由定义为部分函数，该函数通过请求进行模式匹配。Akka HTTP 路由定义是由嵌套指令构建的。请求通过匹配指令自顶向下遍历路径。

1.  **你能说出为什么事件源数据存储可以比传统的关系型数据库扩展得更好吗？**

并发更新比追加操作需要更多的锁定和同步。

1.  **使用 http4s 和 doobie 实现`GET /articles/:name`调用。**

1. 添加新的路由定义：

`case GET -> Root / "articles" / name => renderInventory(repo.getArticle(name))`

2. 扩展`getArticle`方法：

```java
def getArticle(name: String): Stream[IO, Inventory] =
  sql"SELECT name, count FROM article where name = $name"
    .query[(String, Int)].stream.transact(transactor)
    .fold(Map.empty[String, Int])(_ + _)
```

在 GitHub 上查看重构版本的源代码，该版本重用了`getInventory`的无参数定义。

1.  **使用 Akka HTTP 和 Akka Persistence 实现`GET /articles/:name`调用。**

1. 添加新的查询定义：

```java
final case class GetArticle(name: String) extends Query
```

2. 在`InventoryActor`中添加查询处理器：

```java
case GetArticle(name) =>
  sender() ! Inventory(inventory.state.filter(_._1 == name))
```

3. 添加路由定义：

```java
  pathPrefix("articles") {
      path(Segment) { name =>
          get {
            complete((inventory ? GetArticle(name)).mapTo[Inventory])
          }
      }
  }
```

GitHub 仓库包含此路由定义，它嵌入在先前定义的`lazy val articlesRoutes: Route`中。

# 第十五章

1.  **如何将带有查询参数的端点映射到 REST 调用？**

```java
def answer(parameter: Int): ServiceCall[NotUsed, Done]

override def descriptor: Descriptor = {
  import Service._
  named("Answer").withCalls(
    restCall(Method.POST, "/answer?parameter", answer _)
  )
}
```

1.  **推荐用于持久化实体的序列化格式是什么？**

Lagom 推荐使用 JSON 作为序列化格式。

1.  **你能解释为什么在 Lagom 中使用持久化需要集群吗？**

Lagom 的持久化是在 Akka 持久化之上实现的。Akka 要求每个持久化 actor 都有一个唯一的持久化 ID。在微服务领域中，每个服务应该同时拥有多个实例。如果没有集群，将会有多个具有相同 ID 的持久化 actor 将事件存储到同一个数据库中，这将损坏数据。通过利用集群和集群分片，Akka 确保在服务的所有实例中集群中只有一个持久化 actor。

1.  **描述一个可能的数据模型，该模型可用于将`Manager`转换为持久化实体。**

```java
trait ManagerCommand
final case class AddCookies(count: Int) extends ManagerCommand with ReplyType[Int]
final case class RemoveCookies(count: Int) extends ManagerCommand with ReplyType[Int]

trait ManagerEvent
final case class NumberOfCookiesChanged(count: Int) extends ManagerEvent with AggregateEvent[NumberOfCookiesChanged] {
  override def aggregateTag: AggregateEventTag[NumberOfCookiesChanged] = AggregateEventTagNumberOfCookiesChanged
}
sealed trait ManagerState {
  def cookies: Int
}
final case class MixingState(cookies: Int) extends ManagerState
```

1.  **概述实现 Baker 服务的另一种方法。**

`Baker`服务也可以实现类似于`Chef`服务的信息传递风格。

1.  **你能识别出 Chef 当前实现中的设计缺陷吗？**

`Chef`在恢复后不会触发不平衡混合事件的混合。

1.  **`Manager`实现将多个 cookie 存储在内存中，而这个数字在服务重启的瞬间将会丢失。你能说出另一个为什么将 cookie 的数量保存在局部变量中不是一个好主意的原因吗？**

在生产环境中，将有多个服务实例运行。每个实例都将有自己的内部状态。
