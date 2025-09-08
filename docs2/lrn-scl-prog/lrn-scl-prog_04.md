# 了解隐式和类型类

我们已经熟悉 Scala 的两个基石——其类型系统和一等函数。隐式是第三个。隐式使得优雅的设计成为可能，而且没有隐式，可能没有哪个 Scala 库能达到当前的技术水平。

在本章中，我们将从不同类型的隐式的系统概述开始，并回顾隐式作用域解析规则。在简要查看上下文边界后，我们将继续讨论类型类，这是现代函数式编程库中使用的核心实现机制。

本章将涵盖以下主题：

+   隐式类型的种类

+   上下文边界

+   类型类

+   类型类和递归解析

+   类型类变异性

+   隐式作用域解析规则

# 技术要求

在我们开始之前，请确保您已安装以下内容：

+   JDK 1.8+

+   SBT 1.2+

本章的源代码可在我们的 GitHub 仓库中找到：[`github.com/PacktPublishing/Learn-Scala-Programming/tree/master/Chapter04`](https://github.com/PacktPublishing/Learn-Scala-Programming/tree/master/Chapter04)。

# 隐式类型的种类

在 Scala 中，关键字 `implicit` 后面隐藏着几种不同的机制。这个列表包括隐式参数、隐式转换和隐式类。它们具有略微不同的语义，了解在哪些情况下哪种最适合非常重要。这三种类型每一种都值得简要概述。

# 隐式转换

我们列表中的第一种隐式类型是隐式转换。它们允许您自动将一种类型的值转换为另一种类型的值。这种隐式转换被定义为标记有 `implicit` 关键字的单参数方法。隐式转换被认为是一种有争议的语言特性（我们稍后会探讨原因），因此我们需要通过编译器标志或导入相应的语言特性来显式启用它们：

```java
import scala.language.implicitConversions
```

`Predef` 包含了针对 Java 特定类和原语的一组隐式转换。例如，这是 Scala 的 `Int` 和 Java 的 `Integer` 中 *自动装箱* 和 *自动拆箱* 的定义方式：

```java
// part of Predef in Scala
implicit def int2Integer(x: Int): java.lang.Integer = x.asInstanceOf[java.lang.Integer]
implicit def Integer2int(x: java.lang.Integer): Int = x.asInstanceOf[Int]

```

这两种方法在编译器期望 `Int` 类型的值，但提供了 `java.lang.Integer` 类型的值（反之亦然）的情况下被使用。假设我们有一个返回随机 `Integer` 的 Java 方法，我们将在以下场景中应用隐式转换：

```java
val integer: Integer = RandomInt.randomInt()
val int: Int = math.abs(integer)
```

`math.abs` 期望 `Int` 类型，但提供了 `Integer`，因此编译器应用了隐式转换 `Integer2int`。

相同的原则也适用于返回类型，就像它们适用于参数一样。如果编译器在一个没有此方法的类型上找到方法调用，它将寻找隐式转换，以便原始返回类型可以转换为适合此方法的数据类型。这允许你实现一个名为**扩展方法**的模式。Scala 中的`String`类型是一个完美的例子。它被定义为 Java 的`String`类型的类型别名：

```java
type String = java.lang.String
```

但是，你可以调用诸如`map`、`flatMap`、`append`、`prepend`以及许多其他未在原始`String`中定义的方法。这是通过每次调用此类方法时将`String`转换为`StringOps`来实现的：

```java
@inline implicit def augmentString(x: String): StringOps = new StringOps(x)

scala> "I'm a string".flatMap(_.toString * 2) ++ ", look what I can do"
res1: String = II''mm aa ssttrriinngg, look what I can do
```

隐式转换可以是类型参数化的，但不能嵌套或直接链式调用。编译器一次只会应用一个隐式转换：

```java
case class AT
case class BT

implicit def a2AT: A[T] = A(a)
implicit def a2BT: B[T] = B(a)

def abC: Unit = println(a)
```

由于隐式转换`t2B`在作用域内，编译器将接受带有`A`的调用，但会拒绝既不是`A`也不是`B`的所有内容：

```java
scala> ab(A("A"))
B(A(A))

scala> ab("A")
          ^
       error: type mismatch;
        found : String("A")
        required: B[A[?]]
```

有时，可以强制执行其中一个转换，以便编译器可以应用另一个。在这里，我们通过提供类型注解来告诉编译器应用从`String`到`A[String]`的转换。然后，从`A`到`B[A]`的转换就像之前一样发生：

```java
scala> ab("A" : A[String])
B(A(A))
```

非常方便，不是吗？

那么，为什么隐式转换被认为是可疑的呢？因为有时它们可以在开发者不知道的情况下应用，并以意想不到的方式改变语义。在存在两种类型转换（如我们的 Int/Integer 示例）或涉及现有类型的情况下，这可能会特别糟糕。这个经典例子是基于作用域内存在一些隐式转换和后续的类型强制转换：

```java
scala> implicit val directions: List[String] = List("North", "West", "South", "East")
directions: List[String] = List(north, west, south, east)
scala> implicit val grades: Map[Char, String] = Map('A' -> "90%", 'B' -> "80%", 'C' -> "70%", 'D' -> "60%", 'F' -> "0%")
grades: Map[Char,String] = ChampHashMap(F -> 0%, A -> 90%, B -> 80%, C -> 70%, D -> 60%)
scala> println("B" + 42: String)
B42
scala> println(("B" + 42): String)
B42
scala> println("B" + (42: String))
java.lang.IndexOutOfBoundsException: 42
  at scala.collection.LinearSeqOps.apply(LinearSeq.scala:74)
  at scala.collection.LinearSeqOps.apply$(LinearSeq.scala:71)
  at scala.collection.immutable.List.apply(List.scala:72)
  ... 38 elided
scala> "B" + 'C'
res3: String = BC
scala> "B" + ('C': String)
res4: String = B70%
scala> "B" + (2: String)
res5: String = BSouth
```

在这里，我们可以看到这种行为的两个例子：一个是语义上相似的`String`加`Int`连接产生不同的结果，另一个是以相同的方式制作，但用于`String`和`Char`。

奇怪的结果和`IndexOutOfBoundsException`的原因是`Map`和`List`都实现了`PartialFunction`，因此只是`Function1`。在我们的例子中，对于`List`是`Int => String`，对于`Map`是`Char => String`。两者都被定义为隐式的，并且当需要其中一个类型转换时，相应的函数会被应用。

由于这种不可预测性，现代 Scala 中不建议使用隐式转换，尽管它们没有被从语言中移除或弃用，因为许多现有实现依赖于它们。它们主要用于向现有类添加方法或为新特质添加特质实现。

# 隐式参数

隐式参数使用与隐式转换相同的语法，但提供不同的功能。它们允许你自动将参数传递到函数中。隐式参数的定义是在函数定义中作为单独的参数列表完成的，前面有一个`implicit`关键字。只允许一个隐式参数列表：

```java
case class AT
case class BT

def abC(a: A[C])(implicit b: B[C]): Unit =
  println(s"$name$a$b")
```

隐式参数不需要任何特殊的导入或编译器选项来激活。前面的例子显示，它们也可以是类型参数化的。如果在调用方法时没有为隐式参数提供值，编译器将报告错误：

```java
scala> ab("1")(A("A"))
                ^
    error: could not find implicit value for parameter b: B[String]
```

这个错误可以通过提供所需的隐式值来修复：

```java
scala> implicit val b = B("[Implicit]")
b: B[String] = B([Implicit])

scala> ab("1")(A("A"))
1A(A)B([Implicit])
```

如果作用域中有多个隐式值，编译器将返回错误：

```java
scala> implicit val c = B("[Another Implicit]")
c: B[String] = B([Another Implicit])

scala> ab("1")(A("A"))
                ^
       error: ambiguous implicit values:
        both value b of type => B[String]
        and value c of type => B[String]
        match expected type B[String]
```

解决这个问题的方法是移除所有除一个之外的模糊隐式值，或者使其中一个值更具体。我们稍后会看看如何做到这一点。另一种方法是显式地提供隐式值：

```java
scala> ab("1")(A("A"))(b)
1A(A)B([Implicit])

scala> ab("1")(A("A"))(c)
1A(A)B([Another Implicit])
```

隐式参数不需要是一个值——它可以定义为一种方法。拥有不纯的隐式方法可能会导致*随机*行为，尤其是在隐式参数类型相对通用的情况下：

```java
scala> implicit def randomLong: Long = scala.util.Random.nextLong()
randomLong: Long

scala> def withTimestamp(s: String)(implicit time: Long): Unit = println(s"$time: $s")
withTimestamp: (s: String)(implicit time: Long)Unit

scala> withTimestamp("First")
-3416379805929107640: First

scala> withTimestamp("Second")
8464636473881709888: Second
```

由于这个原因，有一个普遍的规则，即隐式参数必须具有可能的具体类型。遵循这个规则还可以帮助你避免使用像以下这样的递归隐式参数来混淆编译器：

```java
scala> implicit def recursiveLong(implicit seed: Long): Long = scala.util.Random.nextLong(seed)
recursiveLong: (implicit seed: Long)Long

scala> withTimestamp("Third")
                    ^
       error: diverging implicit expansion for type Long
       starting with method recursiveLong
```

如果做得正确，隐式参数可以非常有用，可以为实现提供配置参数。通常，它是自顶向下进行的，并影响程序的各个层：

```java
object Application {
  case class Configuration(name: String)
  implicit val cfg: Configuration = Configuration("test")
  class Persistence(implicit cfg: Configuration) {
    class Database(implicit cfg: Configuration) {
      def query(id: Long)(implicit cfg: Configuration) = ???
      def update(id: Long, name: String)(implicit cfg: Configuration) = ???
    }
    new Database().query(1L)
  }
}
```

在这个例子中，配置在顶层定义一次，并自动传递到最低层的函数中。因此，函数的调用变得更加易读。

这些配置设置只是更通用用例的一个特例——上下文传递。与普通参数相比，上下文通常更稳定，这就是为什么隐式传递它是有意义的。这个经典的例子是`ExecutionContext`，这对于大多数`Future`方法都是必需的（我们将在第六章，*探索内置效果*)中详细探讨）：

```java
def filter(p: T => Boolean)(implicit executor: ExecutionContext): Future[T] = ...
```

执行上下文通常不会改变，与过滤逻辑相反，因此它是隐式传递的。

另一个用例是验证类型。我们已经在第二章，*理解 Scala 中的类型*中看到了一个例子，当时我们讨论了泛化类型约束。

# 隐式类

到目前为止，我们已经讨论了隐式转换和扩展方法模式。实现通常是这样的，即旧类型被包装在一个新类型的实例中，然后提供额外的功能。我们以`StringOps`为例，但让我们尝试自己实现这个模式。我们将有一个类型`A`，我们希望它能够执行某些操作`b`：

```java
case class AT { def doA(): T = a }
A("I'm an A").doB() // does not compile
```

我们可以通过定义一个包含所需操作的类，并提供从`A`到`B`的隐式转换来修复编译错误：

```java
case class BT { def doB(): T = b }

import scala.language.implicitConversions
implicit def a2bT: B[T] = B(a.a)

A("I'm an A").doB() // works
```

这种方法如此常见，以至于 Scala 为此提供了一个特殊的语法，称为**隐式类**。它将定义一个类和一个隐式转换合并为一个类的定义。扩展类型成为新类构造函数的参数，就像在前面的代码和以下示例中一样：

```java
implicit class CT { def doC(): T = a.a }
A("I'm an A").doC()
```

这样更简洁，并且不需要`scala.language.implicitConversions`导入。

这种情况的原因在于，普通隐式转换和隐式类之间存在微妙但重要的区别。虽然隐式转换可以表示任何类型的改变，包括已经存在的和/或原始类型，但隐式类是一种在考虑类型转换的情况下创建的东西。它接受初始类型作为构造函数参数的事实使得它以这种方式参数化——从某种意义上说。总的来说，使用隐式类比使用隐式转换更安全。

# 视图和上下文边界

我们之前讨论过的隐式转换和隐式参数，它们无处不在，以至于有专门的编程语言语法来表示它们，即视图和上下文边界。自从 Scala 2.11 以来，视图边界已经被弃用，但我们相信了解它们将有助于你理解上下文边界，因此我们将讨论两者，尽管详细程度不同。

# 视图边界

*视图边界*是隐式参数的语法糖，它表示两种类型之间的转换。它允许你以略短的形式编写带有这种隐式参数的方法签名。我们可以通过开发一个方法来比较两种不相关的类型，如果两者都存在转换到第三个特定类型的话，来看到这两种方法之间的区别：

```java
case class CanEqual(hash: Int)

def equalCA, CB(implicit ca: CA => CanEqual, cb: CB => CanEqual): Boolean = ca(a).hash == ca(a).hash
```

带有视图边界的版本（类似于我们在第二章，*理解 Scala 中的类型*)中讨论的上界和下界，有一个更简短的定义：

```java
def equalsWithBoundsCA <% CanEqual, CB <% CanEqual: Boolean = {
  val hashA = implicitly[CA => CanEqual].apply(a).hash
  val hashB = implicitly[CB => CanEqual].apply(b).hash
  hashA == hashB 
}
```

我们在这里使用的隐式方法是`helper`方法，它在`Predef`中定义为以下内容：

```java
@inline def implicitlyT = e
```

这允许我们召唤一个类型为`T`的隐式值。我们没有明确提供这个隐式值，因此我们需要通过在召唤的转换上使用`apply`方法来帮助编译器确定调用序列。

如果实现比原始版本更复杂，我们为什么要使用它呢？答案是——如果隐式参数只是传递给某个内部函数，它就会变得更好：

```java
def equalsWithPassingCA <% CanEqual, CB <% CanEqual: Boolean = equal(a, b)
```

正如我们之前所说的，自从 Scala 2.11 以来，视图边界已经被弃用，所以我们不会进一步讨论。相反，我们将关注上下文边界。

# 上下文边界

在隐式参数以正常参数的类型进行参数化的另一个特殊情况下，我们之前的示例可以重写如下：

```java
trait CanEqual[T] { def hash(t: T): Int }

def equalCA, CB(implicit ca: CanEqual[CA], cb: CanEqual[CB]): Boolean =
  ca.hash(a) == cb.hash(b)
```

正如我们之前提到的，为此情况提供了一些语法糖，称为 *上下文边界*。有了上下文边界，我们的示例可以简化如下：

```java
def equalBoundsCA: CanEqual, CB: CanEqual: Boolean = {
  val hashA = implicitly[CanEqual[CA]].hash(a) 
  val hashB = implicitly[CanEqual[CB]].hash(b)
  hashA == hashB
}
```

正如前一个例子，当隐式参数传递给内部函数时，这种语法变得简洁：

```java
def equalDelegateCA: CanEqual, CB: CanEqual: Boolean = equal(a, b)
```

现在，这既简短又易于阅读！

缺失的是为不同的 `CA` 和 `CB` 实现隐式参数。对于 `String` 类型，可能实现如下：

```java
implicit val stringEqual: CanEqual[String] = new CanEqual[String] {
  def hash(in: String): Int = in.hashCode()
}
```

`Int` 的实现以非常相似的方式进行。使用单抽象方法语法，我们可以用函数替换类定义：

```java
implicit val intEqual: CanEqual[Int] = (in: Int) => in
```

我们可以通过使用柯里化的恒等函数来用更短的代码实现这一点：

```java
implicit val intEqual: CanEqual[Int] = identity _
```

现在，我们可以使用我们的隐式值来调用具有上下文边界的函数：

```java
scala> equal(10, 20)
res5: Boolean = false
scala> equalBounds("10", "20")
res6: Boolean = false
scala> equalDelegate(10, "20")
res7: Boolean = false
scala> equalDelegate(1598, "20")
res8: Boolean = true
```

在前面的代码片段中，编译器为不同类型的参数解析不同的隐式参数，这些隐式参数用于比较函数的参数。

# 类型类

前面的示例表明，为了使上下文边界工作，我们需要三个部分：

1.  被定义为将要调用的函数的隐式参数的参数化类型 `T`

1.  在 `T` 上定义的一个或多个操作（方法），在转换后可用

1.  实现 `T` 的隐式实例

如果方法定义中引用的类型是抽象的，并且该方法在实例中以不同的方式实现，那么我们谈论的是 *特殊参数多态*（与函数的参数多态和子类多态相对）。在这里，我们将探讨如何使用类型类实现这个概念，如果需要，编译器如何找到合适的实例，以及如何在特殊参数多态的情况下应用变异性。

# 类型类

针对非面向对象的语言，特别是那些不能有子类型多态性的语言，如 Haskell，这种特殊的参数多态非常有用。我们讨论的模式在 Haskell 中被称为 *类型类*，这个名字也传到了 Scala 中。类型类在 `stdlib` 和开源库中广泛使用，并且对于 Scala 的函数式编程至关重要。

对于面向对象开发者来说，由于类这个概念，*类型类*这个名字听起来非常熟悉。不幸的是，它与面向对象的类没有关系，反而让人困惑。为了重新调整我的大脑以适应这个模式，我帮助自己将类型类视为*类型的一个类*。

让我们将其与传统面向对象方法以及用于定义一组 USB 电缆的类型类进行比较。在面向对象中，我们会得到以下定义：

```java
trait Cable {
  def connect(): Boolean
}
case class Usb(orientation: Boolean) extends Cable {
  override def connect(): Boolean = orientation
}
case class Lightning(length: Int) extends Cable {
  override def connect(): Boolean = length > 100
}
case class UsbC(kind: String) extends Cable {
  override def connect(): Boolean = kind.contains("USB 3.1")
}
def connectCable(c: Cable): Boolean = c.connect()
```

每个子类通过重写基特质的 `connect` 方法来实现 `connect` 方法。`connectCable` 只是将调用委托给实例，并通过动态分派调用适当的实现：

```java
scala> connectCable(Usb(false))
res9: Boolean = false
scala> connectCable(Lightning(150))
res10: Boolean = true
```

类型类版本看起来略有不同。类不再需要扩展 `Cable`（因此可以成为不同类层次结构的一部分）。我们还为了好玩，将 `UsbC` 类型泛型化：

```java
case class Usb(orientation: Boolean)
case class Lightning(length: Int)
case class UsbCKind
```

连接逻辑已经移动到了由电缆类型参数化的类型类中：

```java
trait Cable[C] {
  def connect(c: C): Boolean
}
```

它是在相应的类型类实例中实现的：

```java
implicit val UsbCable: Cable[Usb] = new Cable[Usb] {
  override def connect(c: Usb): Boolean = c.orientation
}
```

或者使用相同的方法，使用单个抽象方法语法：

```java
implicit val LightningCable: Cable[Lightning] = (_: Lightning).length > 100
```

我们不能为最近参数化的 `UsbC` 定义一个隐式实例，因为我们不能为任何类型参数提供一个通用实现。`UsbC[String]` 的实例（与面向对象版本相同）可以通过以下方式轻松实现：

```java
implicit val UsbCCableString: Cable[UsbC[String]] = 
  (_: UsbC[String]).kind.contains("USB 3.1")
```

`connectCable` 是通过上下文绑定实现的，并使用临时多态来选择合适的委托方法：

```java
def connectCableC : Cable: Boolean = implicitly[Cable[C]].connect(c)
```

这个方法可以像调用它的面向对象兄弟一样调用：

```java
scala> connectCable(Usb(false))
res11: Boolean = false
scala> connectCable(Lightning(150))
res12: Boolean = true
scala> connectCable(UsbC("USB 3.1"))
res13: Boolean = true
```

在调用端，语法相同，但实现不同。它是完全解耦的——我们的案例类对连接逻辑一无所知。实际上，我们可以在另一个封闭源代码库中为定义的类实现这个逻辑！

# 类型类递归解析

在我们之前的例子中，我们没有为参数化的 `UsbC` 类型实现连接功能，我们的解决方案仅限于 `UsbC[String]`。

我们可以通过进一步委托连接逻辑来改进我们的解决方案。比如说，我们有一个隐式函数 `T => Boolean` 可用——我们可以说这是用户想要用来描述连接方法的逻辑。

这是一个**不良**使用隐式的例子。这不仅包括**原始**的 `Boolean` 类型；在定义隐式转换的时刻，它很可能引用另一个预定义的类型。我们提供这个例子正是如它所提及的那样——作为一个避免不良设计的示例！

这就是我们的委托方法可能的样子：

```java
implicit def usbCCableDelegateT: Cable[UsbC[T]] = (c: UsbC[T]) => conn(c.kind)
```

它直接反映了我们对委托函数的直觉——如果存在 `T => Boolean` 的隐式转换，编译器将创建一个 `Cable[UsbC[T]]` 的实例。

这就是它的用法：

```java
implicit val symbolConnect: Symbol => Boolean = 
  (_: Symbol).name.toLowerCase.contains("cable")

scala> connectCable(UsbC('NonameCable))
res18: Boolean = true
scala> connectCable(UsbC('FakeKable))
res19: Boolean = false
```

但然后，我们必须处理我们委托的隐式转换的所有危险。例如，存在以下无关的转换：

```java
implicit val isEven: Int => Boolean = i => i % 2 == 0
implicit val hexChar: Char => Boolean = c => c >= 'A' && c <='F'

```

将突然允许我们以意想不到的方式连接电缆：

```java
scala> connectCable(UsbC(10))
res23: Boolean = true
scala> connectCable(UsbC(11))
res24: Boolean = false
scala> connectCable(UsbC('D'))
res25: Boolean = true
```

这可能看起来像是一种危险的方法，即依赖于另一个隐式定义的存在来产生所需的隐式值，但这正是类型类获得其力量的原因。

为了演示这一点，让我们想象我们想要实现一个 USB 适配器，该适配器应该连接具有不同标准的两个 USB 设备。我们可以通过将适配器表示为连接电缆的两个电缆端来轻松实现这一点，并将实际连接委托给电缆的相应端：

```java
implicit def adaptA, B: Cable[(A, B)] = new Cable[(A, B)] {
  def connect(ab: (A, B)): Boolean = 
    ev1.connect(ab._1) && ev2.connect(ab._2) 
}
```

或者，我们可以使用上下文界限和 SAM 语法：

```java
implicit def adapt[A: Cable, B: Cable]: Cable[(A, B)] =
  (ab: (A, B)) => 
    implicitly[Cable[A]].connect(ab._1) &&
    implicitly[Cable[B]].connect(ab._2)
```

现在，我们可以使用这个隐式定义来调用我们现有的`connectCable`方法，但带有适配器逻辑：

```java
scala> val usb2usbC = (Usb(false), UsbC('NonameCable))
usb2usbC: (Usb, UsbC[Symbol]) = (Usb(false),UsbC('NonameCable))

scala> connectCable(usb2usbC)
res33: Boolean = false

scala> val lightning2usbC = (Lightning(150), UsbC('NonameCable))
lightning2usbC: (Lightning, UsbC[Symbol]) = (Lightning(150),UsbC('NonameCable))

scala> connectCable(lightning2usbC)
res34: Boolean = true
```

非常令人印象深刻，不是吗？想象一下，要为 OO 版本添加这个功能需要多少努力！

乐趣还没有结束！由于上下文界限解析的递归性质，我们现在可以构建任意长度的链，编译器将递归地检查在编译时是否可以构建所需的适配器：

```java
scala> val usbC2usb2lightning2usbC = ((UsbC('NonameCable), Usb(false)), (Lightning(150), UsbC("USB 3.1")))
usbC2usb2lightning2usbC: ((UsbC[Symbol], Usb), (Lightning, UsbC[String])) = ((UsbC('NonameCable),Usb(false)),(Lightning(150),UsbC(USB 3.1)))

scala> connectCable(usbC2usb2lightning2usbC)
res35: Boolean = false

scala> val noUsbC_Long_Cable = (UsbC('NonameCable), (Lightning(150), UsbC(10L)))
noUsbC_Long_Cable: (UsbC[Symbol], (Lightning, UsbC[Long])) = (UsbC('NonameCable),(Lightning(150),UsbC(10)))

scala> connectCable(noUsbC_Long_Cable)
                     ^
       error: could not find implicit value for evidence parameter of type Cable[(UsbC[Symbol], (Lightning, UsbC[Long]))]
```

我们可以通过在我们的类型类定义上应用特殊注解来稍微改进错误信息：

```java
@scala.annotation.implicitNotFound("Cannot connect cable of type ${C}")
trait Cable[C] {
  def connect(c: C): Boolean
}
```

那么，我们的最后一次失败的尝试将更好地解释失败的原因：

```java
scala> connectCable(noUsbC_Long_Cable)
                     ^
       error: Cannot connect cable of type (UsbC[Symbol], (Lightning, UsbC[Long]))
```

很遗憾，在这个案例中我们只能做到这一步。编译器目前无法确定失败的真实原因仅仅是`UsbC[Long]`而不是整个类型。

编译器将始终尝试根据子类型和方差推断最具体的隐式值。这就是为什么可以结合子类型多态和特设多态。

# 类型类方差

为了了解这种组合是如何工作的，让我们想象我们的 USB 电缆代表一个具有共同祖先的层次结构：

```java
abstract class UsbConnector
case class Usb(orientation: Boolean) extends UsbConnector
case class Lightning(length: Int) extends UsbConnector
case class UsbCKind extends UsbConnector
```

这将如何影响我们的类型类定义？当然，我们之前的每个子类型都单独实现的版本将正常工作。但如果我们想为整个`UsbConnector`层次结构提供一个泛型类型类实例，就像以下示例中所示，会怎样呢？

```java
implicit val usbCable: Cable[UsbConnector] = new Cable[UsbConnector] {
  override def connect(c: UsbConnector): Boolean = {
    println(s"Connecting $c")
    true
  }
}
```

我们将无法再连接我们的电缆：

```java
scala> connectCable(UsbC("3.1"))
                   ^
       error: could not find implicit value for evidence parameter of type Cable[UsbC[String]]
```

这是因为我们的类型类定义是不变的——因此，我们预计要提供一个`Cable[T]`的实例，其中`T <:< UsbC[String]`。`usbCable`是一个合适的匹配吗？结果证明它不是，因为它的返回类型是`Cable[UsbConnector]`，而我们期望提供一个`UsbC[String]`。

我们可以通过两种方式解决这个问题，具体取决于我们是否希望我们的类型类对任何类层次结构都以相同的方式工作，或者是否每个需要一般处理的类层次结构都必须单独定义它。

在第一种情况下，我们需要确保编译器理解以下内容：

```java
Cable[UsbConnector] <:< Cable[UsbC[String]]
```

我们可以在 REPL 中检查这目前不是这种情况：

```java
implicitly[Cable[UsbConnector] <:< Cable[UsbC[String]]]
                 ^
        error: Cannot prove that Cable[UsbConnector] <:< Cable[UsbC[String]]
```

但我们已经知道我们需要做什么才能让它通过——我们的`Cable`应该成为协变类型：

```java
trait Cable[-C] {
  def connect(c: C): Boolean
}
```

一旦我们在`Cable`的定义中引入适当的变体，所有问题都会迎刃而解，编译器可以解决所有必需的隐式类型：

```java
scala> implicitly[Cable[UsbConnector] <:< Cable[UsbC[String]]]
res1: TypeClassVariance.Cable[TypeClassVariance.UsbConnector] <:< TypeClassVariance.Cable[TypeClassVariance.UsbC[String]] = generalized constraint

scala> connectCable(UsbC("3.1"))
Connecting UsbC(3.1)
```

不幸的是，如果我们决定只为我们的类层次结构中的某些类进行特殊处理，我们就无法通过定义一个*更具体*的类型类实例来重用我们的实现：

```java
implicit val usbCCable: Cable[UsbC[String]] = new Cable[UsbC[String]] {
  override def connect(c: UsbC[String]): Boolean = {
    println(s"Connecting USB C ${c.kind}")
    true
  }
}

scala> connectCable(UsbC("3.1"))
Connecting UsbC(3.1)
```

这个测试表明，泛型实例仍然被使用，而特定实例被忽略。

幸运的是，我们还有另一个选择，在我们可以承担每个层次结构自己处理子类型解析的情况下是可行的。在这种情况下，我们保持类型类的不变性，但将类型类实例改为特定类型而不是通用类型：

```java
implicit def usbPolyCable[T <: UsbConnector]: Cable[T] = new Cable[T] {
  override def connect(c: T): Boolean = {
    println(s"Poly-Connecting $c")
    true
  }
}
```

我们需要将`val`改为`def`以便能够对其进行参数化。我们的泛化约束再次开始对不变类型类失效：

```java
scala> implicitly[Cable[UsbConnector] <:< Cable[UsbC[String]]]
 ^
 error: Cannot prove that Cable[UsbConnector] <:< Cable[UsbC[String]].
```

尽管如此，我们可以连接电缆：

```java
scala> connectCable(UsbC("3.1"))
Poly-Connecting UsbC(3.1)
```

现在，编译器能够为我们类型类选择最具体的可用实例！如果我们将`implicit val usbCCable`的定义重新引入作用域，我们会看到输出发生变化：

```java
scala> connectCable(UsbC("3.1"))
Connecting USB C 3.1
```

这显示了*静态重载解析*是如何工作的。但这只是部分情况。让我们澄清编译器在需要隐式类型时如何以及在哪里查找它们。

# 隐式作用域解析

为了将隐式类型放在它们所需的位置，编译器首先必须找到它们。这个过程称为**隐式作用域解析**，并具有明确的规则，以确保隐式类型按照语言规范和开发者使用它们的方式被确定。隐式作用域解析是一个三步过程。

或者如果我们把隐式参数作为方法参数显式提供的情况算作第四步。我们将考虑这种情况为“零”，因为它具有最高的优先级，并且不涉及隐式查找。

我们将简要概述这些步骤，以便我们有一个地方可以方便地参考，然后我们将详细介绍列表中的每个细节：

+   当前调用（或词法）作用域。它具有优先级，并包含可以直接通过其名称（无需前缀）访问的隐式类型，如下所示：

    +   局部声明

    +   外部作用域声明

    +   包对象

    +   继承链

    +   导入语句

+   隐式作用域。它是递归查找的，包括以下内容：

    +   参数的伴生对象

    +   超类型伴生对象

    +   混合类型（超特性）的伴生对象

    +   类的伴生对象

    +   类型参数的伴生对象

    +   类型构造器的伴生对象

+   在一个作用域中找到多个隐式类型时的静态重载规则。

# 词法作用域

让我们从*词法作用域*开始。词法作用域定义了如何在嵌套语言结构（如方法、函数和其他结构化块）中解析变量。一般来说，外部块的定义在内部块内部是可见的（除非它们被遮蔽）。

以下列表显示了在此作用域中隐式解析期间的所有可能的冲突：

```java

package object resolution {
  implicit val a: TS = new TS("val in package object") // (1)
}

package resolution {
  class TS(override val toString: String)
  class Parent {
    // implicit val c: TS = new TS("val in parent class") // (2)
  }
  trait Mixin {
    // implicit val d: TS = new TS("val in mixin") // (3)
  }
  // import Outer._ // (4)
  class Outer {
    // implicit val e: TS = new TS("val in outer class") // (5)
    // import Inner._ // (6)

    class Inner(/*implicit (7) */ val arg: TS = implicitly[TS]) extends Parent with Mixin {
      // implicit val f: TS = new TS("val in inner class") (8)
      private val resolve = implicitly[TS]
    }
    object Inner {
      implicit val g: TS = new TS("val in companion object")
    }
  }
  object Outer {
    implicit val h: TS = new TS("val in parent companion object")
  }
}
```

可能的冲突已用下划线标出。很容易看出，包对象中的隐式值、外部和内部作用域，以及那些被引入内部或外部作用域的值，具有相同的权重。如果类构造函数的参数（7）被声明为隐式，它也会导致冲突。

# 隐式作用域

现在，让我们来看一个关于*隐式作用域*的例子，它的优先级低于词法作用域。隐式作用域通常包括（如果适用）类型的伴随对象、参数类型的隐式作用域、类型参数的隐式作用域，以及对于嵌套类型，外部对象。

以下示例演示了前三个情况的实际操作：

```java
import scala.language.implicitConversions

trait ParentA { def name: String }
trait ParentB
class ChildA(val name: String) extends ParentA with ParentB

object ParentB {
  implicit def a2Char(a: ParentA): Char = a.name.head

}
object ParentA {
  implicit def a2Int(a: ParentA): Int = a.hashCode()
  implicit val ordering = new Ordering[ChildA] {
    override def compare(a: ChildA, b: ChildA): Int =
      implicitly[Ordering[String]].compare(a.name, b.name)
  }
}
object ChildA {
  implicit def a2String(a: ParentA): String = a.name
}

trait Test {
  def test(a: ChildA) = {
    val _: Int = a // companion object of ParentA
    val _: String = a // companion object of ChildA
    val _: Char = a // companion object of ParentB
  }
  def constructorT: Ordering: List[T] = in.toList.sorted // companion object of type constructor
  constructor(new ChildA("A"), new ChildA("B")).sorted // companion object of type parameters
}
```

在这里，我们在类层次结构中散布了一些隐式转换，以展示查找是如何在参数及其超类型（包括超特型）的伴随对象上进行的。最后两行演示了隐式作用域包括构造函数和`sorted`方法参数类型的类型参数。

与第一个例子不同，我们在这个例子中定义的所有隐式内容都是明确的。如果不是这样，编译器将应用静态解析规则来尝试确定最具体的隐式内容。

# 静态重载规则

静态重载规则的定义相当长且复杂（可以在官方文档[`www.scala-lang.org/files/archive/spec/2.13/06-expressions.html#overloading-resolution`](https://www.scala-lang.org/files/archive/spec/2.13/06-expressions.html#overloading-resolution)中找到）。它指定了编译器用来决定选择哪个替代隐式内容的一组规则。这个决定基于替代方案的相对权重。权重越高意味着替代方案`A`比`B`更具体，`A`获胜。

`A`相对于`B`的相对权重是两个数字的总和：

+   如果`A`定义在从定义`B`的类或对象派生的类或对象中（简化地说，如果`A`是`B`的子类或子类伴随对象，或者`B`是`A`的超类伴随对象`A`的子类`B`的伴随对象）

+   如果`A`与`B`一样具体（简化地说，这意味着如果`A`是一个方法，它可以与`B`使用相同的参数调用；对于多态方法，这也意味着更具体的类型约束）

这两个规则允许你计算两个隐式转换或参数之间的相对权重，在权重不同的情况下选择更合适的替代方案。如果权重相等，编译器将报告模糊的隐式值。

# 摘要

在本章中，我们讨论了三种类型的隐式表达式。这包括隐式转换、隐式类和隐式参数。

我们还讨论了语言提供的语法糖，即视图边界和上下文边界。我们已经看到，前者以某种简洁的方式允许定义隐式转换，而后者对类型类做同样的事情。

我们比较了面向对象和基于类型类的多态行为方法。根据我们对这个主题的了解，我们通过案例类的递归解析进行了工作，并展示了类型类变异性示例。

总之，我们研究了三个级别的隐式作用域解析的工作方式。我们已经表明，所有在词法作用域中的隐式表达式具有相同的优先级。只有当在词法作用域中找不到合适的隐式表达式时，编译器才会查看隐式作用域。如果作用域中有多个隐式表达式，则使用静态重载规则来解决可能的冲突。

本章总结了本书中关于 Scala 语言结构的部分。在接下来的章节中，我们将转向更复杂的概念。但在这样做之前，在下一章中，我们将简要地探讨基于属性的测试，以了解我们将用于验证本书第二部分所写代码假设的一些技术。

# 问题

1.  描述一个隐式参数也是隐式转换的情况。

1.  将以下使用视图边界的定义替换为使用上下文边界的定义：`def compare[T <% Comparable[T]](x: T, y: T) = x < y`？

1.  为什么有时人们说类型类将行为和数据分开？

1.  很容易改变词法作用域中可能冲突的示例，以便其中一个隐式表达式胜过其他所有隐式表达式，并且所有其他隐式表达式都可以在不产生冲突的情况下取消注释。你能改变这个吗？

# 进一步阅读

Mads Hartmann, Ruslan Shevchenko, *《专业 Scala》*: 在一个让你为 JVM、浏览器等更多环境构建的环境中使用简洁和表达性强的、类型安全的代码。

Vikash Sharma, *《学习 Scala 编程》*: 学习如何在 Scala 中编写可扩展和并发程序，这是一种随着你成长的语言。
