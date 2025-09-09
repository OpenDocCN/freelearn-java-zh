# 第七章. 函数式习语

在上一章中，我们介绍了隐式参数和隐式转换。我们看到了如何为你的代码启用优雅的扩展方法。我们还查看了一下 Scala 编译器如何解析隐式参数。最后，我们讨论了隐式参数的工作原理、如何使用它们以及它们提供的益处。

在本章中，我们将介绍函数式编程的核心概念，如`纯`函数、不可变性和高阶函数。我们将在此基础上建立理解，并介绍一些在大规模函数式程序中普遍存在的、你无疑会在开始使用专注于函数式编程的 Scala 库时遇到的模式。最后，我们将介绍两个流行的函数式编程库，即`Cats`和`Doobie`，并使用它们编写一些有趣的程序。

函数式编程语言已经存在很长时间了，但最近它们获得了更多的关注，因为大多数流行的编程语言都采用了函数式编程的概念。可能的原因是函数式编程很容易解决在命令式语言中难以解决的问题，例如编写可并行化的程序。函数式编程还可以提高你程序的模块化程度，从而使它们更容易测试、重用和推理，希望最终产生更少的错误。

到本章结束时，你将能够：

+   确定函数式编程的核心概念

+   识别和实现流行的函数式编程设计模式

+   在你的 Scala 项目中实现 Cats 和 Doobie

# 函数式编程概念简介

在本节中，我们将介绍函数式编程背后的核心概念，并为你提供理解和编写简单函数式程序所需的知识。

到本节结束时，你应该对函数式编程背后的核心概念有很好的理解，例如：

+   编写和使用纯函数

+   使用不可变类而不是可变类

+   编写和使用高阶函数

## 纯函数

函数式编程的核心是`纯`函数的概念。一个函数是纯的，如果它没有任何副作用，也就是说，该函数仅根据函数的参数计算结果，不做其他任何事情。

副作用示例包括修改变量、在对象上设置字段、执行输入/输出操作，如读取或写入文件、将值打印到控制台等。

让我们看看一些`纯`函数和`非纯`函数的例子，以便更好地理解它们之间的区别。让我们看看两个函数：

```java
case class Person(var name: String, var age: Int)

def birthday(p: Person) = p.age += 1

def getName(p: Person) = {
  println(s"Getting the name of ${p.name}")
  p.name
}

def rename(p: Person, name: String) = 
  Person(name, p.age)
```

在这里，我们定义了一个简单的名为`Person`的 case 类，它有一个`name`和一个`age`。然后，我们定义了两个操作`Person`的函数。你能看出为什么这些函数不是纯函数吗？为了测试这些函数是否是纯函数，我们可以尝试用相同的参数两次调用它们，看看是否得到不同的结果——记住，纯函数是一个仅基于函数参数计算结果的函数，并且不做其他任何事情。

让我们从`birthday:`开始。

```java
val p = Person("Jack", 41)
birthday(p)
println(p) // prints Person(Jack,42)
birthday(p)
println(p) // prints Person(Jack,43)
```

好吧，所以`birthday`不是一个纯函数，因为它正在修改传递给函数的`Person p`的状态。我们也许能够猜测这一点，因为`birthday`的返回类型是`Unit`——由于函数不返回任何值，它必须执行一些副作用，否则函数将完全没有用处。

接下来，让我们看看`getName:`。

```java
val n1 = getName(p) // Getting the name of Jack
val n2 = getName(p) // Getting the name of Jack
println(n1) // Jack
println(n2) // Jack
```

好消息是，函数返回了名称值，并提供了相同的参数。然而，该函数仍然不是纯函数，因为它在每次被调用时都会打印到控制台。

最后，让我们看看`rename:`。

```java
val r1 = rename(p, "John")
val r2 = rename(p, "John")
println(r1) // Person(John,43)
println(r2) // Person(John,43)
```

好吧，所以`rename`是一个纯函数。当提供相同的参数时，它会产生相同的值，并且不会执行任何可观察的副作用。

我们现在已经涵盖了纯函数的概念，并看到了纯函数和不纯函数的例子。你已经看到了定义纯函数的两种方式：

+   当提供相同的参数时，它会产生相同的值，并且不会执行任何可观察的副作用。

+   纯函数是一个只包含引用透明表达式的函数。如果一个表达式是引用透明的，那么它可以被它的值替换，而不会改变程序的行为。

在下一个子节中，我们将探讨不可变性，这是另一个使编写纯函数成为可能的核心概念。

## 不可变性

在理解了什么是纯函数之后，现在是时候介绍另一个核心概念，它使得编写纯函数成为可能：不可变性。如果无法更改某个东西，那么它就是不可变的——这是可变的对立面。

Scala 之所以非常适合函数式编程，其中一个原因就是它提供了保证不可变性的构造。让我们看看一些例子。

如果我们使用`val`关键字（而不是`var`）在类上定义一个变量或字段，那么 Scala 编译器将不允许我们更改该值。

1.  在你的电脑上打开终端。

1.  通过输入`scala.`来启动 Scala REPL。

1.  现在，你可以在`scala>`之后开始粘贴代码。

    ```java
    scala> val x = "test"
    x: String = test

    scala> x = "test 2"
    <console>:12: error: reassignment to val
           x = "test 2"
    ```

1.  预期的输出显示在`scala>`的下一行。

1.  然而，Scala 不能保证你不修改分配给变量的值的州，例如：

    ```java
    scala> case class Person(var name: String)
    defined class Person

    scala> val p = Person("Jack")
    p: Person = Person(Jack)

    scala> p.name
    res0: String = Jack
    scala> p.name = "John"
    p.name: String = John

    scala> p
    res1: Person = Person(John)
    ```

1.  然而，如果我们移除`var`关键字，Scala 将默认使用`val`来声明字段名，因此不允许我们更改它，从而强制执行不可变性。

### 实现标准库

Scala 的标准库在`scala.collection.immutable`包中提供了一整套不可变数据结构。

其中最常用的可能是`scala.collection.immutable.List`，它在`Predef`中导入，因此可以在 Scala 程序中简单地作为`List`访问。

```java
scala> val xs = List(1,2,3)
xs: List[Int] = List(1, 2, 3)

scala> xs.reverse
res0: List[Int] = List(3, 2, 1)
scala> xs
res1: List[Int] = List(1, 2, 3)
```

这里，你可以看到`xs.reverse`返回一个新的`List`，它是反转的，并且`x` `s`保持不变。

Scala 提供了确保不可变性的构造，并且在许多情况下使用不可变性作为默认值，例如在定义案例类或使用标准库提供的某些不可变集合时。在下一小节中，我们将探讨高阶函数，当你使用函数式编程在 Scala 中编写程序时，你将广泛使用这些函数。

## 高阶函数

高阶函数是接受其他函数作为参数的函数。这是在 Scala 中广泛使用的一种技术，当你编写 Scala 程序时，你将经常使用它。高阶函数已经在之前讨论过，但为了完整性，我们在这里简要回顾一下。

这里是使用高阶函数`map`在`List`上的一个例子，它对列表中的每个元素调用一个函数以生成一个新的`List`：

```java
scala> val xs = List(1,2,3,4,5)
xs: List[Int] = List(1, 2, 3, 4, 5)

scala> xs.map(_ * 2)
res0: List[Int] = List(2, 4, 6, 8, 10)
```

注意，这个例子使用了纯函数、不可变性和高阶函数——这是完美的函数式程序。

这里是如何定义一个高阶函数的例子。它接受一个`A => Boolean`类型的函数，并返回一个`A => Boolean`类型的函数，该函数否定原始函数的结果：

```java
def negateA: A => Boolean =
  (a: A) => !f(a)
```

我们将编写一个高阶函数，它根据给定的谓词在列表中找到第二个元素：

```java
def sndWhereA(pred: A => Boolean): Option[A] = ???
```

这里有两个使用该函数的例子：

```java
println(sndWhere(List(1,3,2,4,4))(_ > 2)) // Some(4)
println(sndWhere(List(1,3,2,4,4))(_ > 10)) // None
```

现在，尝试编写你自己的高阶函数，以更好地理解它们的工作方式。

1.  在你的编辑器中创建一个新的 Scala 文件，命名为`HOExample.scala`。

1.  将以下代码粘贴进来：

    ```java
    object HOExample extends App {
      def sndWhereA(pred: A => Boolean): Option[A] = ???
      println(sndWhere(List(1, 3, 2, 4, 4))(_ > 2)) // Some(4)
      println(sndWhere(List(1, 3, 2, 4, 4))(_ > 10)) // None
    }
    ```

1.  这个高阶函数应该根据给定的谓词在列表中找到第二个元素。

1.  当你在编辑器中运行应用程序时，你应该看到以下输出：

    ```java
    Some(4) 
    None
    ```

1.  一种可能的解决方案是：

    ```java
      def sndWhereA(pred: A => Boolean): Option[A] =  

        xs.filter(pred) match { 

          case _ :: snd :: _ => Some(snd) 

          case _             => None 

        }
    ```

我们回顾了高阶函数是什么，并看到了它们如何被用来编写通用函数，其中函数的一些功能由函数的参数提供。

在下一节中，我们将超越函数式编程的基本概念，并查看在使用函数式库时可能会遇到的一些函数式设计模式。

你已经看到了函数式编程的三个基石：

+   **纯函数**：一个函数仅根据函数的参数计算结果，并做其他任何事情。

+   **不可变性**：你已经看到了 Scala 如何支持不可变性，并在许多情况下将其用作默认值。

+   **高阶函数**：接受其他函数作为参数或返回函数的函数称为高阶函数。

# 函数式设计模式

在本节中，我们正在超越函数式编程的基本概念，并查看在使用函数式库时可能会遇到的一些函数式设计模式。你将介绍`Monoids`、`Functors`、`Monads`和其他你可以用来结构化程序的函数式编程模式——这些模式是当你第一次学习面向对象编程时可能熟悉的面向对象设计模式的函数式编程等价物。

你是否听说过范畴论？本节中我们将看到的模式来源于范畴论。每个概念（例如一个`Monoid`）都有一个明确的数学定义和一组相关定律。

### 注意

在本节中，我们不会详细介绍这些定律，但如果你想要进一步研究这个主题，了解这个领域被称为范畴论是有好处的。

这将为你进一步学习函数式设计模式做好准备，并使你能够使用一些最受欢迎的函数式编程库，如`Cats`和`Scalaz`。

以下小节将分别介绍一个抽象结构，展示其在 Scala 中的定义，并展示在编写程序时如何使用它——这些结构一开始可能看起来非常抽象，但请耐心，因为例子将展示这些结构如何非常有用。

### 注意

以下代码将大量使用类型类，所以请参考前面的章节，确保你对它们有很好的理解。

## Monoids

我们将要考察的第一个结构被称为`Monoid`。`Monoid`是一个非常简单的结构，但一旦你学会了识别它，你将经常遇到它。

一个`Monoid`有两个操作，`combine`和`empty`。在 Scala 中，`Monoid`的定义可以像以下这样表达为一个类型类：

```java
trait Monoid[A] {
  def combine(x: A, y: A): A
  def empty: A
}
```

也就是说，`Monoid`类型类的实例支持两种操作：

+   `combine`：这个操作接受两个类型为`A`的参数，并返回一个`A`。

+   `empty`：这个操作不接受任何参数，但它返回一个`A`。

### 注意

在范畴论中，这些操作被称为`乘法`和`单位`。如果你想在以后学习这个主题，这可能是有用的。

这非常抽象，所以让我们通过一些例子来具体化它。

让我们看看`Monoid`类型类的一个实例，以更好地了解它是如何工作的以及你可以如何使用它。

1.  为`String`定义一个`Monoid`类型类实例，即`Monoid[String]`：

    ```java
    implicit val strMonoid = new Monoid[String] {
      def combine(x: String, y: String): String = x + y
     def empty: String = ""
    }
    ```

1.  按如下定义`stringMonoid`：

    ```java
    strMonoid.combine("Monoids are ", "great")
    strMonoid.combine("Hello", strMonoid.empty)
    ```

在大多数情况下，你不会像我们在这里这样明确地引用`Monoid`的具体实例，而是当编写多态函数时使用`Monoid`类型类，正如你在练习之后的例子中将会看到的。

让我们创建用于创建`Monoid`的隐式定义。

1.  为`Int`编写一个`Monoid`实例：

    ```java
    implicit val intMonoid = new Monoid[Int] {
        def combine(x: Int, y: Int): Int = x + y
        def empty: Int = 0
      }
    ```

1.  编写一个`implicit def`，可以为任何`A`创建一个`Monoid[List[A]]`：

    ```java
    implicit def listMonoid[A]: Monoid[List[A]] = 
      new Monoid[List[A]] {
       def combine(x: List[A], y: List[A]): List[A] = ???
        def empty: List[A] = ???
      }
    ```

### 使用 Monoids 编写多态函数

尽管`Monoid`可能看起来很简单，但它非常有用。`Monoid`的力量以及你稍后将介绍的其他结构，在我们定义多态函数时发挥作用，这些函数对其参数一无所知，除了它们类型的`Monoid`实例存在。

让我们编写一个求列表和的函数：

```java
def sumA(implicit m: Monoid[A]): A = xs.foldLeft(m.empty)(m.combine)
```

第一个参数列表很简单——它定义了该函数接受一个不同`A`值的`List`。然而，第二个隐式参数列表要求编译器为`A`找到一个`Monoid`实例，并且只有当存在`Monoid[A]`的实例时，你才能调用`sum[A]`。

使用这个函数，只要作用域中存在适当的`Monoid`实例，你就可以对任何`List`求和：

```java
sum(List("Monoids", " are", " cool")) // "Monoids are cool"
sum(List(1,2,3)) // 6
sum(List(List(1,2),List(3,4)) // List(1,2,3,4)
```

我们看到了第一个结构，即`Monoid`。我们看到尽管`Monoid`有一个非常简单的接口，但它展示了自己是一个有用的结构，它允许我们编写有趣的泛型函数。在下一节中，我们将查看`Functors`。简单来说，`Functor`是你可以在其上映射的东西。

## Functor

我们接下来要看的第二个结构是`Functor`。简单来说，`Functor`是你可以在其上映射的东西。在 Scala 中，你无疑已经多次使用这个操作来操作`Lists`、`Options`等。在 Scala 中，`Functors`的类型类可能看起来像这样：

```java
trait Functor[F[_]] {
  def mapA, B(f: A => B): F[B]
}
```

### 注意

`Functor`抽象了一个类型构造器`F`——抽象类型构造器的类型被称为高阶类型。

你可能会觉得`map`是一个方便的方式来迭代集合，如`List`或`Set`，但在`Functor`的上下文中，有一个更有趣的方式来看待它。你应该把`map`看作是对某种类型上的操作进行序列化的方式，它保留了类型结构，正如类型的具体性所定义的那样。细节将根据类型而变化：

+   **Option**：可能没有值。

+   **List**：可能有零个或多个值。

+   **Either**：可能有一个错误或一个值——不会两者都有。

让我们看看一些具体的例子，以便使这更加清晰。

现在我们将在不同的上下文中评估相同的东西。

1.  编写一个抽象不同`Functors`的多态函数：

    ```java
    def compute[F[_]](fa: F[Int])(implicit f: Functor[F]): F[Int] = {
     val fx = f.map(fa) { _ + 2 }
      f.map(fx) { _ * 2}
    }
    ```

    它定义了一个函数`compute`，该函数首先使用`map`将`2`添加到 functor 内的值，然后使用`map`将结果乘以`2`。

1.  我们现在可以用任何具有`Functor`实例定义的值调用这个方法：

    ```java
    compute(List(1,2,3)) // List(6, 8, 10)compute(Option(2) // Some(8)
    compute(Right(2): Either[String, Int]) // Right(8)
    ```

思考这个练习，你可以看到`Functor`如何让你编写多态函数，这些函数对其参数一无所知，除了它们有一个为它们定义的`map`函数。具体`Functor`定义了在其特定上下文中`map`的含义。

让我们定义并使用`List`和`Option`的`Functor`。

1.  定义`List`的`Functor`：

    ```java
    implicit val listFunctor = new Functor[List] {
      def mapA, B(f: A => B): List[B] = fa.map(f)
    }
    ```

1.  使用它看起来可能像这样：

    ```java
    listFunctor.map(List(1,2,3))(_ * 2)
    ```

**为 Option 定义 Functor**

1.  为 `Option` 编写一个 `Functor` 实例：

    ### 注意

    解决方案可以在 `Examples/src/main/scala/Functor.scala` 中找到，定义为 `optionFunctor.` 

    ```java
     implicit val optionFunctor = new Functor[Option] {
        def mapA, B(f: A => B): Option[B] = fa match {
          case None => None
          case Some(x) => Some(f(x))
        }
    ```

我们已经讨论了第二个结构，Functor，它代表可以映射的事物。你看到了如何为 Functor 定义实例，以及如何将 `map` 视为在某种类型上按顺序执行操作的一种方式，这种方式保留了类型的结构，正如类型的具体性所定义的那样。在下一节中，我们将讨论 Monads – 一种你可能已经熟悉但不知道其名称的结构。

## Monads

我们将要讨论的最后一个结构是 `Monad`。大多数 Scala 程序员对 Monads 都很熟悉，即使他们不知道这个名字，因为这种抽象在编写 `for-comprehension` 时总是被使用。一个 Monad 有两个操作，`pure` 和 `flatMap:` 

```java
trait Monad[F[_]] extends Functor[F] {
  def flatMapA, B(f: A => F[B]): F[B]
  def pureA: F[A]
def mapA, B(f: A => B): F[B] =
    flatMap(fa)(f andThen pure)
}
```

### 注意

注意，范畴论中 `flatMap` 操作的名称是 `bind`，而 `pure` 的名称是 `unit`。

回想一下上一节，你可以将 Functors 视为在某种类型上按顺序执行操作的一种方式，这种方式保留了类型的结构，正如类型的具体性所定义的那样。嗯，对于 Monad 也是如此，只是它们更强大。对于 Functors，复杂性只能发生在序列的开始部分，而对于 Monads，它们可以发生在序列的任何部分。

让我们看看一个例子：

```java
Option(10).map(_ + 1).map(_ * 4)
res3: Option[Int] = Some(44)
```

如果其中一个操作返回了一个 Option 值呢？

```java
def big(i: Int): Option[Int] = 
  if (i > 5) Some(i) 
  else None

big(10).map(_ - 5).map(big)
res3: Option[Option[Int]] = Some(None)
```

现在，我们有一个 `Option[Option[Int]]` 类型的值，这并不太方便。这就是 `Monad` 发挥作用的地方。如果你有一系列操作，你希望在每个步骤上保留某些类型的特定性，那么你将想要使用一个 `Monad`。

### 注意

如前所述的定义所示，Monad 也是一个 Functor，因为 map 可以用 `flatMap` 和 `pure` 来实现。

下面是如何为 Option 定义一个 Monad：

```java
implicit val optionMonad = new Monad[Option] {
  def pureA: Option[A] = Some(x)
  def flatMapA, B(f: A => Option[B]): Option[B] = fa match {
    case Some(x) => f(x)
    case None => None
  }
}
```

`pure` 通过简单地用 `Some` 包装值来定义。`flatMap` 通过对值进行模式匹配并在它是 `Some` 时应用函数来定义，否则返回 `None`。

我们可以避免之前不便利的情况，即有一个 `Option[Option[Int]]`：

```java
flatMap(map(big(10))(_ - 5))(big)
```

我们已经看到了 Monad 的定义以及它如何可以在需要保留依赖于 Monad 实例的特定性的特定上下文中按顺序执行操作。

# 流行库

到目前为止，你应该对函数式编程背后的主要概念有了很好的理解，例如纯函数、不可变性和高阶函数。除此之外，你应该熟悉在编写函数式程序时使用的最流行的抽象。有了所有这些知识，你就可以开始研究 Scala 中一些流行的函数式编程库了。

在本节中，我们将查看 Scala 生态系统中的几个流行的函数式编程库。在本节之后，你应该能够：

+   使用 `Cats` 的 `Validated` 类型类来验证你的数据

+   使用 `Doobie` 与数据库通信

## 使用 Cats 验证数据

在本节中，我们将快速概述 Cats 库，并查看它提供用于 `Validate` 数据的一个数据类型。

### 注意

更多关于 Cats 的信息，请参阅 [`github.com/typelevel/cats`](https://github.com/typelevel/cats)。

到本节结束时，你应该了解 `Cats` 如何融入 Scala 生态系统，并知道如何在你的项目中使用它，特别是用于验证数据。

## 使用 Cats 的先决条件

1.  你需要将 `Cats` 添加为 Scala 项目的依赖项。创建一个新的 SBT 项目，包含以下 `build.sbt` 文件：

    ```java
    name := "cats-example"

    scalaVersion := "2.12.4"
    libraryDependencies += "org.typelevel" %% "cats-core" % "1.0.0"

    scalacOptions ++= Seq(
      "-Xfatal-warnings",
      "-Ypartial-unification"
    )
    ```

1.  由于 `Cats` 严重依赖隐式函数来提供类型类实例和扩展方法，因此当你在文件中使用 `Cats` 时，始终需要以下导入：

    ```java
    import cats._
    import cats.implicits._
    ```

## Cats 简介

Cats 是一个库，它为 Scala 编程语言中的函数式编程提供抽象。具体来说，它为上一节中看到的所有模式（Monoid、Monad 等）提供了定义，并且还有更多。它还包含 Scala 标准库中所有相关类的类型类实例。

Cats 的更广泛目标是提供一个支持 Scala 应用程序中纯函数式编程的生态系统的基础。

## 验证数据

使用 Cats 验证数据有两种不同的方式。第一种是 `Either`，正如你从标准库中了解的那样，另一种是 `Validated`。如果你想快速失败验证，你应该使用 `Either`；如果你想累积错误，你应该使用 `Validated`。你应该使用哪一个取决于你的用例。由于你可能已经熟悉标准库中的 `Either`，我们将在这个子节中重点介绍 `Validated`。

首先，让我们看看一个基本的领域模型和一些我们对该数据的要求。假设你有一个如下所示的 `User`：

```java
final case class User(
  username: String,
  age: Int
)
```

让我们假设你为这个用户有以下规则：

+   用户名必须至少包含三个字符。

+   用户名只能包含字母数字字符。

+   年龄必须是正数。

我们可以使用 Scala 中的 sealed trait 和 case objects 来表示这三个错误，如下所示：

```java
sealed trait Error {
  def message: String
}

case object SpecialCharacters extends Error {
  val message: String = "Value can't contain special characters"
}

case object TooShort extends Error {
  val message: String = "Value is too short"
}

case object ValueTooLow extends Error {
  val message: String = "Value is too low"
}
```

让我们看看如何使用 `Validated` 在 Scala 中编写前面的规则。

## 使用 Validated 验证

`Validated` 数据类型定义如下：

```java
sealed abstract class Validated[+E, +A] extends Product with Serializable
final case class Valid+A extends Validated[Nothing, A]
final case class Invalid+E extends Validated[E, Nothing]
```

在这里，`Valid[A]` 表示通过某种验证的类型 `A` 的值，而 `Invalid[A]` 表示某些验证失败，产生类型为 `E` 的错误。让我们尝试实现上一节中的规则：

```java
def validateAge(age: Int): Validated[NonEmptyList[Error], Int] =
    if (age >= 1) age.validNel
    else ValueTooLow.invalidNel
```

在这里，我们定义了一个方法 `validateAge`，它接受一个 `Int` 并返回 `Validated[NonEmptyList[Error], Int]`，这意味着如果它有效，则返回 `Valid(age)`；如果它无效，则返回 `Invalid(NonEmptyList(ValueTooLow))`。我们使用 `Cats` 为您提供的便利扩展方法 `validNel` 和 `invalidNel`。

接下来，让我们定义一个用于用户名的验证器：

```java
private def checkLength(str: String): Validated[NonEmptyList[Error], String] =
  if (str.length > 3) str.validNel
  else TooShort.invalidNel

private def checkSpecialCharacters(str: String): Validated[NonEmptyList[Error], String] =
  if (str.matches("^[a-zA-Z]+$")) str.validNel
  else SpecialCharacters.invalidNel

def validateUsername(username: String): Validated[NonEmptyList[Error], String] =
  (checkLength(username), checkSpecialCharacters(username)).mapN { 
    case (a, _) => a 
  }
```

在这种情况下，我们定义了两个辅助函数，`checkLength` 和 `checkSpecialCharacters`，它们检查字符串是否超过 3 个字符，并且不包含任何字母数字字符。我们使用 Cats 为元组提供的 `mapN` 扩展方法结合这两个检查。如果两个检查都通过，`mapN` 函数将使用包含两个有效值的元组调用，但我们只对用户名感兴趣，一旦我们简单地返回第一个有效值。

最后，让我们编写一个验证用户名和年龄的方法，并在一切有效的情况下返回一个 `User`：

```java
def validate(username: String, age: Int) =
    (validateUsername(username), validateAge(age)).mapN { User.apply }
```

再次，我们使用 `mapN` 方法并传递一个函数，如果所有检查都通过，则调用该函数。在这种情况下，我们使用 `User` 上的 `apply` 方法来创建 `User` 的实例。

如果你调用这个，你可以看到如果有任何错误，它会累积错误；否则，它返回一个经过验证的 `User`：

```java
User.validate("!!", -1)
// Invalid(NonEmptyList(TooShort, SpecialCharacters, ValueTooLow))

User.validate("jack", 42)
// Valid(User(jack,42))
```

到目前为止，你应该知道如何将 Cats 添加到你的 Scala 项目中，并知道如何使用它们的 `Validated` 数据类型以优雅的函数式方式编写你的领域模型的验证方法。在下一节中，我们将探讨如何使用 `Doobie` 库以函数式类型安全的方式与数据库进行通信。

## 使用 Doobie 与数据库通信

在本小节中，我们将使用 `Doobie` 库以函数式和类型安全的方式与数据库进行通信。你将了解 `Doobie` 的核心概念，并了解如何使用它来查询、更新和删除数据库中的行。

在本小节之后，你应该能够在自己的 Scala 项目中使用 `Doobie` 进行简单的查询和插入，并知道在哪里找到更高级用例的文档。

### Doobie 的先决条件

你需要将 Doobie 添加到你的 Scala 项目中依赖项。创建一个新的 SBT 项目，包含以下 `build.sbt` 文件：

```java
scalaVersion := "2.12.4"

lazy val doobieVersion = "0.5.0-M13"

libraryDependencies ++= Seq(
  "org.tpolecat" %% "doobie-core"   % doobieVersion,
  "org.tpolecat" %% "doobie-h2"     % doobieVersion
)
```

Doobie 可以与许多不同的数据库进行通信，例如 MySQL、Postgres、H2 等。在以下示例中，我们将使用内存数据库 H2 来简化设置。查看文档了解如何使用其他数据库。

在这些示例中，我们将使用的数据库表相当简单。有两个表，`user` 和 `todo`，它们具有以下 SQL 定义：

```java
CREATE TABLE user (
  userId INT AUTO_INCREMENT PRIMARY KEY,
  username VARCHAR(255) NOT NULL,
  age INT NOT NULL
);

CREATE TABLE todo (
  todoId INT AUTO_INCREMENT PRIMARY KEY,
  userId INT NOT NULL,
  title VARCHAR(255) NOT NULL,
  completed BOOL DEFAULT false,
  FOREIGN KEY (userId) references user(userId)
);
```

## Doobie

在 Doobie 中，为了编写大型程序，你需要编写小的程序。创建程序后，你可以直接将其放入主函数中，并用作有效的 monad。

Doobie 有一个高级 API 和一个低级 API。在本讲座中，我们将专注于高级 API。在高级 API 中，只有少数几个重要的类。其中最重要的两个是`ConnectionIO`和`Transactor。`

### ConnectionIO

你将遇到的最常见的类型具有`ConnectionIO[A]`的形式，指定在`java.sql.Connection`可用的上下文中进行的计算，最终产生类型为`A`的值。

`ConnectionIO[A]`是最常见的类型，它指定了在`java.sql.Connection`可用的位置进行的计算，它生成类型为`A`的值。

### 事务处理者

`Transactor`是一个用于数据库连接、连接分配和清理的结构；它可以从`ConnectionIO[A]`接收并产生`IO[A]`，这提供了一段可执行代码。它特别提供了一个`IO`，当执行时，将连接到数据库并在单个事务中运行程序。

下面是一个完整的示例，它创建了一个可以连接到 H2 数据库的`Transactor`。然后我们使用这个`transactor`来执行简单的查询`SELECT 42:`

```java
package com.example

import doobie._
import doobie.implicits._
import cats.effect.IO

object ExampleConnection extends App {

  val transactor =
    Transactor.fromDriverManagerIO
 val program: ConnectionIO[Int] =
    sql"select 42".query[Int].unique

  val task: IO[Int] =
    program.transact(transactor)

  val result: Int =
    task.unsafeRunSync

  println(s"Got result ${result}")

}
```

上述示例使用`Transactor.fromDriverManager`创建了一个`Transactor`，然后创建了一个小程序，当提供`Connection`时，将运行`SELECT 42`。最后，程序通过在`ConnectionIO`上使用 transact 方法转换为`IO`单子，并通过`unsafeRunSync`执行`IO`单子来产生一个`Int`，它是执行`SELECT 42`的结果。重要的是要理解，直到使用`unsafeRunSync`执行`IO`单子，不会发生任何副作用。

接下来，让我们看看如何编写查询。

### 选择行

现在我们已经看到了如何使用`ConnectionIO`和`Transactor`，让我们看看一些更有趣的查询，以执行一个非常简单的查询。

让我们从上一个例子中的以下表达式开始分析：

```java
val fragment: Fragment = sql"select 42"
val query: Query0[Int] = fragment.query[Int]
val program = query.unique
```

我们使用 Doobie 的字符串插值函数`sql`将一个普通的`String`转换为`Fragment`。`Fragment`是 Doobie 对 SQL 查询部分的表示，可能包括插值值。`Fragments`可以通过连接组合，这保持了插值值的正确偏移和映射。在本讲座中，我们不会详细介绍你可以用`Fragments`做什么，但你可以在文档中找到更多信息。

一旦你有了`Fragment`，你可以使用`Fragment`上的`query[A]`方法将其转换为`Query`。`Query[A, B]`是 Doobie 对完整 SQL 查询的表示，该查询接受类型为`A`的一些输入并产生类型为 B 的一些输出。在这个特定例子中，我们的查询不接受任何输入，因此返回特定的类型`Query0`，它表示一个不接受任何参数的查询。

最后，通过在 `Query` 上使用 `unique` 方法生成一个 `ConnectionIO[Int]`。如前所述，`ConnectionIO[A]` 表示在具有 `java.sql.Connection` 的上下文中进行的计算，最终产生一个类型为 `A` 的值。在这种情况下，我们使用 `unique` 因为只期望返回一行。其他有趣的方法是 `list` 和 `opti` `on`，分别返回 `ConnectionIO[List[A]]` 和 `ConnectionIO[Option[A]]`。

### 使用参数进行查询

在 Doobie 中，你将参数传递给查询的方式与任何字符串插值器一样。参数被转换为预处理语句中的 `?` 表达式，以避免 SQL 注入。让我们看看如何使用参数进行查询：

### 注意

你可以在 `lesson-1/doobie-example/src/main/scala/com/example/User.scala` 中找到以下查询。

```java
case class User(userId: Int, username: String, age: Int)

def allManual(limit: Int): ConnectionIO[List[User]] = sql"""
    SELECT userId, username, age
    FROM user
    LIMIT $limit
  """
    .query[(Int, String, Int)]
    .map { case (id, username, age) => User(id, username, age) }
    .list

def withUsername(username: String): ConnectionIO[User] = sql"""
    SELECT userId, username, age
    FROM user
    WHERE username = $username
  """.query[User].unique
```

第一个查询 `allManual` 接收一个 `Int` 并将其用作参数来定义 SQL 查询的 `LIMIT`。第二个查询接收一个 `String` 并在 `WHERE` 子句中使用它来选择具有该特定用户名的用户。`allManual` 查询选择一个 `(Int, String, Int)` 的 `Tuple` 并对其 `maps` 以生成一个 `User`，而 `withUsername` 使用 Doobie 的能力在查询该类型的行时自动使用 case class 的 `apply` 方法。

### 删除、插入和更新行

删除、插入和更新的工作方式类似。首先，让我们看看如何删除行：

```java
def delete(username: String): ConnectionIO[Int] = sql"""
    DELETE FROM user
    WHERE username = $username
  """.update.run
```

再次，我们使用 SQL 字符串插值器来定义我们的查询，并简单地引用作用域内的变量来定义查询的参数。然而，我们不是在 `Fragment` 上使用 `query[A]` 方法来生成一个 `Query`，而是使用 `update[A]` 来生成一个 `Update[A]`。然后我们使用 `Update` 上的 `run` 方法来生成 `ConnectionIO[A]:`

```java
def setAge(userId: Int, age: Int): ConnectionIO[Int] = sql"""
    UPDATE user
    SET age = $age
    WHERE userId = $userId
  """.update.run

def create(username: String, age: Int): ConnectionIO[Int] = sql"""
    INSERT INTO user (username, age)
    VALUES ($username, $age)
  """.update.withUniqueGeneratedKeysInt
```

`setAge` 和 `create` 都很相似，但 `create` 使用 `Update` 上的一个有趣的方法 `withUniqueGeneratedKeys`，该方法返回最后插入行的 `id`。

### 一个完整的示例

让我们看看一个完整的示例。你可以在 `lesson-1/doobie-example/src/main/scala/com/example/Main.scala` 中找到它。我们将逐部分查看每个部分：

```java
val program = for {
    _  <- Tables.create
    userId <- User.create("Jack", 41)
    _ <- User.setAge(userId, 42)
    _ <- Todo.create(userId, "Buy Milk", false)
    _ <- Todo.create(userId, "Read the newspaper", false)
    _ <- Todo.create(userId, "Read the full documentation for Doobie", false)
    uncompleted <- Todo.uncompleted(userId)
  } yield uncompleted
```

在本节中，你可以看到如何通过 Scala 的 `for-comprehensions` 使用 `flatMap` 链接 `ConnectionIO`。这允许我们很好地从执行另一个 `ConnectionIO` 的结果中构建 `ConnectionIO`。在这种情况下，我们首先创建一个 `User`，然后使用后续方法中的 `userId` 来设置用户的年龄并为用户构造三个 `Todo` 方法：

```java
 val all: IO[Unit] = for {
    todos <- program.transact(xa)
    users <- User.all(10).transact(xa)
  } yield {
    todos.foreach(println)
    users.foreach(println)
  }
```

在本节中，我们将通过在 `for` `-comprehension` 中使用 `transact` 方法从两个 `ConnectionIO` 实例生成 `IO`。

```java
 all.unsafeRunSync
```

最后，执行 `IO` 以产生副作用。

## 活动：向待办事项列表添加优先级

想象一个场景，一个客户告诉你他需要在待办事项列表中添加优先级功能。为应用程序设计优先级。

通过在 `lesson-1/doobie-example` 中的示例程序中为每个 Todo 添加优先级，并在查询未完成的 Todo 时使用该优先级，以便首先返回最重要的 Todo。你需要执行以下步骤：

1.  通过添加一个 `priority: Int` 字段来扩展 `Todo case class`。

1.  更新 `Todo.table.` 中的 `Table` 定义。

1.  更新 `Todo.create` 方法，使其接受一个表示优先级的 `Int` 类型的参数。

1.  更新 `Todo.uncompleted` 以按降序排列行。

# 摘要

在本章中，我们介绍了函数式编程的核心概念，如纯函数、不可变性和高阶函数。我们还介绍了一些在大型函数式程序中普遍存在的模式。最后，我们介绍了两个流行的函数式编程库，即 Cats 和 Doobie，并使用它们编写了一些有趣的程序。

在下一章中，我们将介绍 Scala 如何通过提供一些有趣的语言特性，使得编写强大的领域特定语言（DSLs）成为可能。我们将简要地看看一般意义上的 DSLs 是什么。我们还将介绍一个如果你打算专业地使用 Scala 的话，你很可能要使用的 DSL。最后，你将实现你自己的 DSL。
