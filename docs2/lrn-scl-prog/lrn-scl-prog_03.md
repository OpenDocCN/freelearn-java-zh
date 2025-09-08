# 第三章：深入探讨函数

Scala 结合了面向对象和函数式编程范式。特别是，函数是一个一等语言概念。它们可以用各种方式定义，分配给变量，作为参数传递，并存储在数据结构中。Scala 在如何执行这些操作方面提供了很大的灵活性。

我们将首先详细研究定义函数的不同方式。然后我们将继续应用上一章关于类型的知识，使我们的函数成为多态和更高阶的。我们将研究递归、尾递归和跳跃作为 JVM 函数式编程的重要方面。最后，我们将评估与 Scala 中的函数以面向对象方式实现相关的特殊性。

本章将涵盖以下主题：

+   定义函数的方法

+   多态函数

+   高阶函数

+   递归

+   跳跃

+   函数的面向对象特性

# 技术要求

+   JDK 1.8+

+   SBT 1.2+

本章的源代码可在[`github.com/PacktPublishing/Learn-Scala-Programming/tree/master/Chapter03`](https://github.com/PacktPublishing/Learn-Scala-Programming/tree/master/Chapter03)找到。

# 定义函数的方法

为了为不同 Scala 知识水平的读者提供一个共同的基础，让我们回顾一下如何定义一个函数。我们将从基本方法开始，例如定义一个方法并将其放置在不同的作用域中以创建一个局部函数。然后我们将探讨更有趣的方面，例如闭包作用域、部分应用、指定函数字面量的不同方式，以及最终的多态。

# 函数作为方法

大多数 Scala 开发者都是从 Java 开始的。因此，最常见的方法是在类、特质或对象内部定义一个方法，如下面的熟悉示例所示：

```java
class MethodDefinition {
  def eq(arg1: String, arg2: Int): Boolean = ! nonEqual(arg1, arg2)
  private def nonEq(a: String, b: Int) = a != b.toString
}
```

按照惯例，我们已明确为公共方法定义了返回类型，就像我们为 Java 中的返回类型做的那样。对于非递归函数，可以省略结果类型。我们已经在私有方法中这样做。

值参数的类型声明是强制性的。

每个值参数可以分配一个默认值：

```java
def defaultValues(a: String = "default")(b: Int = 0, c: String = a)(implicit d: Long = b, e: String = a) = ???
```

前面的代码片段还演示了定义多个值参数组是可能的。最后一个组的参数可以是隐式的，也可以提供默认值。连续组的默认值可以引用先前组中定义的参数，例如`e`引用了`c`的默认值，而`c`的默认值是`a`。

值参数前缀为`=>`的类型表示，在方法调用时不应评估此参数，而是在每次在方法体中引用参数时进行评估。这种参数称为*按名参数*，基本上，它们代表一个零参数方法，具有参数的返回类型：

```java
scala> def byName(int: => Int) = {
     | println(int)
     | println(int)
     | }
byName: (int: => Int)Unit
scala> byName({ println("Calculating"); 10 * 2 })
Calculating
20
Calculating
20
```

在这个例子中，我们可以看到传递的代码块被执行了两次，与在方法体内部的用法数量相匹配。

将`*`（星号）添加到最后一个值参数类型的名称后缀，表示这是一个*重复*的*参数*，它接受一定数量的定义类型的参数。然后，给定的参数作为指定类型的`collection.Seq`集合在方法体中可用：

```java
def variable(a: String, b: Int*): Unit = {
  val bs: collection.Seq[Int] = b
}

variable("vararg", 1, 2, 3)
variable("Seq", Seq(1, 2, 3): _*)
```

直接将`Seq`传递作为重复参数是不合法的。前一个代码块中的最后一行显示了`:_*`语法来标记最后一个参数为*序列参数*。重复参数不能接受默认值。

方法定义中的参数都有名字。这些名字可以用来通过提供任意顺序的参数来调用方法（与在方法定义中指定的顺序相反）：

```java
def named(first: Int, second: String, third: Boolean) = s"$first, $second, $third"

named(third = false, first = 10, second = "Nice")
named(10, third = true, second = "Cool")
```

命名参数和普通参数可以混合使用，如前一个代码块的最后一行所示。在这种情况下，必须首先指定位置参数。

到目前为止，我们定义的示例都是在包含类或对象的范围内。但 Scala 在这方面提供了更多的灵活性。方法可以在任何有效的范围内定义。这使得它局部于包含块，从而限制了其可见性。

# 局部函数

这里是两个局部于包含方法的函数的例子：

```java
def average(in: Int*): Int = {
 def sum(in: Int*): Int = in.sum
 def count(in: Int*): Int = in.size
 sum(in:_*)/count(in:_*)
}
```

在这个例子中，我们在`average`定义内部同时定义了`sum`和`count`，这使得它们从外部无法访问：

```java
scala> :type sum
       ^
       error: not found: value sum

scala> :type count
       ^
       error: not found: value count
```

如前所述，函数不需要嵌套在另一个方法中。包含块可以是任何类型，例如，变量定义。例如，考虑如果上一个示例中的`average`函数只是为了计算单个平均值而定义：

```java
val items = Seq(1,2,3,4,5)
val avg = average(items:_*)
```

我们可以将这两个代码块重写如下：

```java
val items = Seq(1,2,3,4,5)
val avg = {
  def sum(in: Int*): Int = in.sum
  def count(in: Int*): Int = in.size
  sum(items:_*)/count(items:_*)
}
```

范围可见性规则适用于方法，就像适用于其他语言结构一样。因此，`outer`方法的参数对内部函数是可见的，不需要显式传递。我们可以再次使用这个规则重写我们的第一个例子，如下所示：

```java
def averageNoPassing(in: Int*): Int = {
  def sum: Int = in.sum
  def count: Int = in.size
  sum /count
}
```

对于引用包含块定义的函数，有一个特殊的名称，*闭包*。让我们更深入地讨论一下。

# 闭包

在我们的函数定义中，我们提到了两种不同类型的变量：作为参数提供的那些（*绑定变量*）和在其他地方定义的，即在包含块中定义的（*自由变量*）。自由变量之所以被称为自由变量，是因为函数本身对它没有任何意义。

不引用任何自由变量的函数是自足的，编译器可以在任何上下文中将其翻译成字节码。另一种表述方式是说，这个定义是自封闭的。因此，它被称为*封闭项*。另一方面，引用自由变量的函数只能在所有这些变量都已定义的上下文中编译。因此，它被称为*开放项*，并且在编译时封闭这些自由变量，因此得名*闭包（关于自由变量）*。

与变量和其他定义一样，闭包的常规作用域解析规则同样适用，如下一个片段所示：

```java
scala> def outerA = {
     | val free = 5
     | def innerA = {
     | val free = 20
     | def closure(in: Int) = free + in
     | closure(10)
     | }
     | innerA + free
     | }
outerA: Int

scala> outerA
res3: Int = 35
```

`res3`的计算方式为`outerA.free (5) + innerA.free (20) + closure.in(10)`。

自由变量必须在闭包之前定义，否则编译器会报错：

```java
scala> def noReference(in: Int) = {
     | def closure(input: Int) = input + free + in
     | }
           def closure(input: Int) = input + free + in
                                             ^
On line 2: error: not found: value free

scala> def forwardReference(in: Int) = {
     | def closure(input: Int) = input + free + in
     | val free = 30
     | }
           def closure(input: Int) = input + free + in
                                             ^
On line 2: error: forward reference extends over definition of value free
```

第一次尝试失败是因为我们忘记定义一个自由变量。第二次尝试仍然不成功，因为自由变量是在闭包之后定义的。

# 部分应用和函数

到目前为止，编译器以相同的方式处理方法和变量。我们能否进一步利用这些相似性，将一个方法作为另一个方法的结果返回，并将其存储到变量中？让我们试一试：

```java
scala> object Functions {
     |   def method(name: String) = {
     |     def function(in1: Int, in2: String): String = name + in2
     |     function
     |   }
     |   val function = method("name")
     | }
           function
           ^
On line 4: error: missing argument list for method function
       Unapplied methods are only converted to functions when a function type is expected.
       You can make this conversion explicit by writing `function _` or `function(_,_)` instead of `function`.
```

不幸的是，它不起作用。我们试图在方法内部创建并返回一个函数，并将这个函数赋值给一个变量，但编译器不允许这样做。然而，它给了我们一个关于我们做错了什么的有用提示！

结果表明，对于编译器来说，函数和方法是不同的，方法只能以封装类的实例形式传递。这种区别与 JVM 中所有内容都表示为某个类的实例的事实有关。正因为如此，我们定义的方法成为类的成员方法，方法在 JVM 中不是一等公民。Scala 通过具有不同 arity 的函数的类层次结构来绕过这种方法。因此，为了使我们能够从一个方法中返回一个方法，前者必须成为一个函数。编译器也给了我们一个提示，如何实现这一点：通过在期望参数的地方使用`_`（下划线）。

```java
scala> object Functions {
     | def method(name: String) = {
     | def function(in1: Int, in2: String): String = name + in2
     | function _
     | }
     | val function = method("name")
     | }
defined object Functions
```

部分应用可以有两种形式：用一个下划线替换整个参数列表，或者用一个下划线替换每个参数。因此，对于我们刚刚定义的函数的部分应用，`function _`或`function(_,_)`都是合适的。

可以使用部分应用语法来为在其他地方定义的函数创建*快捷方式*，通过同时导入和部分应用它们：

```java
val next = Math.nextAfter _
next(10f, 20f)
val /\ = Math.hypot(_, _)
/\ (10 , 20)
```

通常，对于具有 N 个参数的函数，部分应用意味着指定 0 <= M < N 个参数，其余参数未定义，基本上是将函数应用于参数列表的一部分。这种部分应用产生一个具有(N-M)个参数的函数，其结果类型与原始函数相同。在我们的上一个例子中，我们将 M 定义为零，因此结果函数的签名保持不变。但部分应用的事实已经将我们的方法转换成了函数，这使我们能够像处理值一样进一步处理它。

在这种情况下，如果 0 < M < N，下划线将填充当前未应用的参数位置：

```java
def four(one: String, two: Int, three: Boolean, four: Long) = ()
val applyTwo = four("one", _: Int, true, _: Long)
```

我们应用了第一个和第三个参数，并留下了第二个和第四个未应用。编译器要求我们为缺失的参数提供类型注解，以便在使用结果函数时进行类型推断。

为方法定义的参数名称在部分应用期间丢失，默认值也是如此。重复的参数被转换为`Seq`。

# 函数字面量

我们可以使用 REPL 来检查`applyTwo`函数的类型：

```java
scala> :type Functions.applyTwo
(Int, Long) => Unit
```

这就是一等函数的类型！一般来说，函数类型由`=>`分隔的左右两部分组成。左侧定义了参数的类型，右侧定义了结果类型。实现遵循相同的模式，被称为*函数字面量*。以下是一个具有四个参数的函数的完整定义示例：

```java
val hash: (Int, Boolean, String, Long) => Int = (a, b, c, d) => {
  val ab = 31 * a.hashCode() + b.hashCode()
  val abc = 31 * ab + c.hashCode
  31 * abc + d.hashCode()
}
```

在实现方面，我们有一个由三个表达式组成的代码块，因此被括号包围。请注意，我们定义我们的函数为`val`。

通常，函数字面量可以使用简化的语法定义。例如，类型推断允许省略结果类型的定义。在这种情况下，类型定义完全消失，因为参数的类型定义将像方法定义一样靠近参数名称：

```java
val hashInferred = (a: Int, b: Boolean, c: String, d: Long) =>
  // ... same implementation as before
```

在应用方面，编译器可以帮助我们进一步简化定义。让我们考虑一个例子：

```java
def printHash(hasher: String => Int)(s: String): Unit = 
  println(hasher(s))
```

对于`hasher`函数，我们可以有以下等价的定义。完整的定义如下代码块所示：

```java
val hasher1: String => Int = s => s.hashCode
val hasher2 = (s: String) => s.hashCode
printHash(hasher1)("Full")
printHash(hasher2)("Inferred result type")

```

这个片段展示了四种表示函数字面量的不同方式：

+   内联定义：`printHash((s: String) => s.hashCode)("inline")`

+   使用函数参数的类型推断内联定义：`printHash((s: String) => s.hashCode)("inline")`

+   使用函数参数的类型推断内联定义（这被称为*目标类型*）：`printHash((s) => s.hashCode)("inline")`

+   单个参数周围的括号可以省略：`printHash(s => s.hashCode)("single argument parentheses")`

+   在这种情况下，如果函数的实现中使用了参数，我们最多只能进一步使用占位符语法：`printHash(_.hashCode)("placeholder syntax")`

实际上，占位符语法非常强大，也可以用来定义具有多个参数的函数以及不在目标类型位置上的函数。以下是一个使用占位符语法计算四个 `Int` 实例哈希码的函数示例：

```java
scala> val hashPlaceholder = 
(_: Int) * 31⁴ + (_: Int) * 31³ + (_: Int) * 31² + (_: Int) * 31

scala> :type hashPlaceholder
(Int, Int, Int, Int) => Int

```

这种语法看起来接近部分应用语法，但代表了一个完全不同的语言特性。

# Currying

说到部分应用，我们还没有提到这个特殊案例，*Currying*。在某种意义上，Currying 是一种部分应用，我们取一个 N 个参数的函数，并对每个参数逐个应用部分应用，每次产生一个接受一个更少参数的函数。我们重复这个过程，直到我们剩下 N 个函数，每个函数接受一个参数。如果听起来很复杂，考虑下一个两个参数的函数示例：

```java
def sum(a: Int, b: Int) = a + b
```

使用两个参数列表，我们可以将其重写如下：

```java
def sumAB(a: Int)(b: Int) = a + b
```

这个方法类型是 `(a: Int)(b: Int): Int`，或者表示为一个函数：

```java
:type sumAB _
Int => (Int => Int)
```

这是一个接受一个 `Int` 并返回一个从 `Int` 到 `Int` 的函数的函数！当然，参数的数量并不限于仅仅两个：

```java
scala> val sum6 = (a: Int) => (b: Int) => (c: Int) => (d: Int) => (e: Int) => (f: Int) => a + b + c + d+ e + f
sum6: Int => (Int => (Int => (Int => (Int => (Int => Int)))))
```

占位符语法将给我们相同的功能，但以 *未应用 Currying* 的形式：

```java
scala> val sum6Placeholder = (_: Int) + (_: Int) + (_: Int) + (_: Int) + (_: Int) + (_: Int)
sum6Placeholder: (Int, Int, Int, Int, Int, Int) => Int
```

与一些其他函数式编程语言相比，Currying 在 Scala 中并不是很重要，但了解作为一个有用的函数式编程概念是好的。

# 多态性和高阶函数

到目前为止，我们只玩过操作单一类型数据的函数（*单态函数*）。现在，我们终于将我们的类型系统知识应用到构建适用于多种类型的函数上。接受类型参数的函数被称为 *多态函数*，类似于在面向对象类层次结构中实现的多态方法（*子类型多态*）。对于 Scala 中的函数，这被称为 *参数多态*。

# 多态函数

在上一章中我们玩 `Glass` 示例时，我们已经使用了多态函数：

```java
sealed trait Glass[+Contents]
case class FullContents extends Glass[Contents]
case object EmptyGlass extends Glass[Nothing]
case class Water(purity: Int)

def drink(glass: Glass[Water]): Unit = ???

scala> :type drink _
Glass[Water] => Unit

```

`drink` 方法是单态的，因此只能应用于类型 `Glass[Water]` 的参数，甚至不能用于 `EmptyGlass`。当然，我们不想为每种可能的内容类型实现一个单独的方法。相反，我们以多态的方式实现我们的函数：

```java
def drinkAndRefillC: Glass[C] = glass
drinkAndRefill: CGlass[C]

scala> :type drinkAndRefill _
Glass[Nothing] => Glass[Nothing]

scala> :type drinkAndRefill[Water] _
Glass[Water] => Glass[Water]
```

类型参数在方法体中可用。在这种情况下，我们指定结果应具有与参数相同的内容类型。

当然，我们可以进一步约束类型参数，就像我们之前做的那样：

```java
def drinkAndRefillWaterB >: Water, C >: B: Glass[C] = glass

scala> :type drinkAndRefillWater[Water, Water] _
Glass[Water] => Glass[Water]
```

在这里，我们的方法接受任何玻璃，只要它是装水的玻璃，并允许填充比水更具体的东西。

这两个例子也表明，我们可以在部分应用期间指定一个类型参数，以便有一个特定类型的单态函数。否则，编译器将以与我们在使用函数字面量定义多态函数时相同的方式应用底类型参数：

```java
scala> def drinkFun[B] = (glass: Glass[B]) => glass
drinkFun: [B]=> Glass[B] => Glass[B]

scala> :type drinkFun
Glass[Nothing] => Glass[Nothing]

scala> drinkFun(Full(Water))
res17: Glass[Water.type] = Full(Water)
```

在函数应用的那一刻，推断出的结果类型是正确的。

# 高阶函数

到目前为止，我们讨论了函数字面量，并创建了一个`printHash`函数，我们用它来演示将函数传递给方法的不同的形式：

```java
scala> def printHash(hasher: String => Int)(s: String): Unit = 
  println(hasher(s))
printHash: (hasher: String => Int)(s: String)Unit
```

`printHash`接受两个参数：`hasher`函数和要哈希的字符串。或者，以函数形式：

```java
scala> :type printHash _
(String => Int) => (String => Unit)
```

我们的函数是**curried**的，因为它接受一个参数（一个`String => Int`函数）并返回另一个函数，`String => Unit`。`printHash`接受一个函数作为参数的事实反映在说`printHash`是一个**高阶函数**（**HOF**）。除了一个或多个参数是函数这一事实外，HOFs 没有其他特殊之处。它们的工作方式与普通函数一样，可以被分配和传递，部分应用，并且是多态的：

```java
def printHashA(s: A): Unit = println(hasher(s))
```

事实上，HOFs 通常以创造性的方式将作为参数给出的函数应用于另一个参数，因此几乎总是多态的。

让我们再次看看我们的`printHash`示例。没有特别之处要求`hasher`函数来计算哈希；`hasher`执行的函数与`printHash`的逻辑无关。有趣的是，这种情况比人们预期的更常见，这导致了 HOF 的定义，例如：

```java
def printerA, B, C <: A(a: C): Unit = println(f(a))
```

我们的打印逻辑不需要给定的函数具有任何特定的参数或结果类型。我们唯一需要强制执行的限制是，可以使用给定的参数调用该函数，我们用类型约束`C <: A`来表述这一点。函数和参数的性质也可以是任何东西，因此定义 HOF 时通常使用简短的、中性的名称。这就是我们的新定义如何在实践中使用：

```java
scala> printer((_: String).hashCode)("HaHa")
2240498
scala> printer((_: Int) / 2)(42)
21
```

编译器需要知道函数的类型，因此我们需要将其定义为占位符语法的一部分。我们可以通过改变函数参数的顺序来帮助编译器：

```java
def printerA, B, C <: A(f: A => B): Unit = println(f(a))
```

使用这个定义，将首先推断出`C`的类型，然后使用推断出的类型来强制`f`的类型：

```java
scala> printer("HoHo")(_.length)
4
scala> printer(42)(identity)
42
```

`identity`函数在标准库中定义为`def identityA: A = x`。

# 递归和跳跃

函数调用另一个函数有一个特殊情况——函数调用自身。这样的函数被称为**递归**。递归函数可以是头递归或尾递归。还有一种以面向对象的方式模拟递归调用的方法，称为**跳跃**。递归非常方便，并且在函数式编程中经常使用这种技术，所以让我们仔细看看这些概念。

# 递归

递归用于实现循环逻辑，而不依赖于循环及其相关的内部状态。递归行为由两个属性定义：

+   **基本案例**：最简单的终止情况，其中不再需要任何递归调用

+   **递归案例**：描述如何将任何其他状态减少到基本案例的规则集

递归实现的可能示例之一是反转一个字符串。这两个递归属性将是：

+   基本情况是空字符串或单字符字符串。在这种情况下，反转后的字符串就是给定的相同字符串。

+   对于长度为 N 的字符串的递归案例，可以通过取字符串的第一个字符并将其附加到给定字符串的反转尾部来减少到长度为 N-1 的字符串的案例。

这就是我们在 Scala 中实现这种逻辑的方式：

```java
def reverse(s: String): String = {
  if (s.length < 2) s
  else reverse(s.tail) + s.head
}
```

所以，这比解释起来容易，对吧？

在递归案例中，我们实现的一个重要方面是它首先执行递归调用，然后将剩余的字符串添加到结果中。这类函数在计算中保持递归调用在头部，因此是头递归（但通常只是称为递归）的。这类似于深度优先算法，实现首先深入到终端情况，然后从底部向上构建结果：

```java
scala> println(reverse("Recursive function call"))
llac noitcnuf evisruceR
```

嵌套函数调用在运行时自然保持在栈中。正因为如此，对于较小输入量工作良好的函数可能会因为较大输入量而耗尽栈空间，导致整个应用程序崩溃：

```java
scala> println(reverse("ABC" * 100000))
java.lang.StackOverflowError
  at scala.collection.StringOps$.slice$extension(StringOps.scala:548)
  at scala.collection.StringOps$.tail$extension(StringOps.scala:1026)
  at ch03.Recursion$.reverse(Recursion.scala:7)
  at ch03.Recursion$.reverse(Recursion.scala:7)
  at ch03.Recursion$.reverse(Recursion.scala:7)
  ...
```

有可能增加 JVM 中为栈保留的内存大小，但通常有更好的解决方案——尾递归。

# 尾递归

在尾递归函数中，递归调用是作为最后一个活动完成的。正因为如此，才有可能“完成”所有对调用的“准备”，然后只需用新的参数“跳转”回函数的开始部分。Scala 编译器将尾递归调用重写为循环，因此这种递归调用根本不会消耗栈空间。通常，为了使递归函数成为尾递归，需要引入某种状态或某种局部辅助函数。

让我们以尾递归的方式重写我们的`reverse`函数：

```java
def tailRecReverse(s: String): String = {
  def reverse(s: String, acc: String): String =
    if (s.length < 2) s + acc
    else reverse(s.tail, s.head + acc)
  reverse(s, "")
}
```

在这个实现中，我们定义了一个局部尾递归函数，`reverse`，它遮蔽了参数`s`，这样我们就不会无意中引用它，并且还引入了一个`acc`参数，这是为了携带字符串的剩余部分。现在，在将字符串的头部和`acc`粘合在一起之后调用`reverse`。为了返回结果，我们使用原始参数和一个空的累加器调用辅助函数。

这种实现不消耗栈空间，我们可以通过在基本案例中抛出异常并检查堆栈跟踪来检查：

```java
scala> println(inspectReverse("Recursive function call"))
java.lang.Exception
  at $line19.$read$$iw$$iw$.reverse$1(<console>:3)
  at $line19.$read$$iw$$iw$.inspectReverse(<console>:5)
```

在我们完成字符串反转的时候，我们仍然只有一个递归调用在栈中。有时这会让开发者感到困惑，因为它看起来好像递归调用不会被执行。在这种情况下，可以通过使用 **notailcalls** 编译器选项来禁用尾调用优化。

有时情况相反，一个（可能）尾递归调用在运行时因为开发者忽略了头位置的递归调用而溢出栈。为了消除这种错误的可能性，有一个特殊的尾递归调用注释，`@scala.annotation.tailrec`：

```java
def inspectReverse(s: String): String = {
  @scala.annotation.tailrec
  def reverse(s: String, acc: String): String = ...
}
```

编译器将无法编译带有此注释的头递归函数：

```java
scala> @scala.annotation.tailrec
     | def reverse(s: String): String = {
     | if (s.length < 2) s
     | else reverse(s.tail) + s.head
     | }
           else reverse(s.tail) + s.head
                                ^
On line 4: error: could not optimize @tailrec annotated method reverse: it contains a recursive call not in tail position
```

看起来，如果我们正确注释了尾递归函数，我们就在安全的一边？嗯，不是 100%，因为也存在一些函数根本无法变成尾递归的可能性。

当尾递归无法实现时的一个例子是互递归。如果第一个函数调用第二个，而第二个函数又调用第一个，那么这两个函数就是互递归的。

在数学中，Hofstadter 序列是一系列相关整数序列的成员，这些序列由非线性递归关系定义。你可以在维基百科上了解更多信息，请参阅[`en.wikipedia.org/wiki/Hofstadter_sequence#Hofstadter_Female_and_Male_sequences`](https://en.wikipedia.org/wiki/Hofstadter_sequence#Hofstadter_Female_and_Male_sequences)。

这些函数的一个例子是 `Hofstadter` 女性和男性序列，定义为如下：

```java
def F(n:Int): Int = if (n == 0) 1 else n - M(F(n-1))
def M(n:Int): Int = if (n == 0) 0 else n - F(M(n-1))
```

非尾递归函数的另一个例子是 Ackerman 函数（更多关于它可以在[`en.wikipedia.org/wiki/Ackermann_function`](https://en.wikipedia.org/wiki/Ackermann_function)上找到）的定义如下：

```java
val A: (Long, Long) => Long = (m, n) =>
  if (m == 0) n + 1
  else if (n == 0) A(m - 1, 1)
  else A(m - 1, A(m, n - 1))
```

它很简单，但不是原始递归，它对栈的需求很大，即使 m 和 n 的值适中也会溢出栈：

```java
scala> A(4,2)
java.lang.StackOverflowError
  at .A(<console>:4)
  at .A(<console>:4)
...
```

有一种称为跳跃式递归的特殊技术，可以在 JVM 上实现非尾递归函数。

# 跳跃式递归

从本质上讲，*跳跃式递归*是替换递归函数调用为表示这些调用的对象。这样，递归计算就在堆上而不是栈上构建，并且由于堆的大小更大，因此可以表示更深的递归调用。

Scala 的 `util.control.TailCalls` 实现为跳跃式递归调用提供了一个现成的抽象。记住，我们在递归中有两个一般情况，它们分解为三个具体的情况？这些是：

+   基本情况

+   递归情况，可以是：

    +   头递归

    +   尾递归

表示通过遵循三个受保护的案例类来反映它们：

```java
case class DoneA extends TailRec[A]
case class CallA => TailRec[A]) extends TailRec[A]
case class ContA, B extends TailRec[B]
```

由于这些是受保护的，我们不能直接使用它们，而是预期使用特殊的辅助方法。让我们通过重新实现我们的 Ackerman 函数来看看它们：

```java
import util.control.TailCalls._

def tailA(m: BigInt, n: BigInt): TailRec[BigInt] = {
  if (m == 0) done(n + 1)
  else if (n == 0) tailcall(tailA(m - 1, 1))
  else tailcall(tailA(m, n - 1)).flatMap(tailA(m - 1, _))
}
def A(m: Int, n: Int): BigInt = tailA(m, n).result
```

我们将递归调用封装到 `tailcall` 方法中，该方法创建一个 `Call` 实例。递归调用比基本情况要复杂一些，因为我们首先需要递归地封装内部调用，然后使用 `TailRec` 提供的 `flatMap` 方法将结果传递给外部的递归调用。

`A` 只是一个辅助方法，用于将计算结果从 `TailRec` 中解构出来。我们使用 `BigInt` 来表示结果，因为现在，由于实现是栈安全的，它可以返回相当大的数字：

```java
scala> Trampolined.A(4,2).toString.length

```

现在，我们已经看到了如何将递归函数表示为对象，是时候揭示关于 Scala 函数的另一个真相了。

# 函数的面向对象方面

我们提到 Scala 是面向对象和函数式范式的融合。正因为如此，Scala 将函数作为语言的第一级元素。也因为如此，Scala 中的一切都是对象。这部分与 JVM 中一切都是对象或原始类型的事实有关，但 Scala 更进一步，还隐藏了原始类型在对象后面。

结果表明，函数也是对象！根据参数的数量，它们扩展了特殊特质之一。也因为它们的面向对象性质，可以通过在实现类上定义额外的方 法来实现额外的功能。这就是部分函数的实现方式。利用伴生对象定义函数的公共逻辑以方便重用也是自然的。甚至可以编写一些函数的定制实现，尽管这很少是一个好主意。

这些方面每一个都值得深入研究，但要真正理解它们，我们需要从一些实现细节开始。

# 函数是特质

Scala 中的每个函数都实现了 `FunctionN` 特质，其中 N 是函数的参数数量。零参数函数由编译器转换为 `Function0` 的实现，一个参数转换为 `Function1`，以此类推，直到 `Function22`。这种复杂性是由于语言的静态性质所必需的。这意味着不能定义超过 22 个参数的函数吗？嗯，总是可以通过柯里化或多个参数列表来定义函数，所以这并不是一个真正的限制。

函数字面量只是编译器为了开发者的方便而接受的语法糖。这就是我们之前定义的 Ackerman 函数去糖后的签名看起来像这样：

```java
val A: Function2[Long, Long, Long] = (m, n) =>
  if (m == 0) n + 1
  else if (n == 0) A.apply(m - 1, 1)
  else A.apply(m - 1, A.apply(m, n - 1))
```

标准库中 `Function2` 的（简化）定义如下：

```java
trait Function2[-T1, -T2, +R] extends AnyRef { self =>
  def apply(v1: T1, v2: T2): R
  ...
}
```

记得我们在上一章中讨论的共变和逆变讨论吗？这里就是它的实际应用；参数是逆变的，结果类型是共变的。

结果表明，编译器将我们的定义重写为实现了此特质的匿名类的实例：

```java
val objectOrientedA: Function2[Long, Long, Long] = 
  new Function2[Long, Long, Long] {
    def apply(m: Long, n: Long): Long =
      if (m == 0) n + 1
      else if (n == 0) objectOrientedA(m - 1, 1)
      else objectOrientedA(m - 1, objectOrientedA(m, n - 1))
}
```

然后，这个类的实例可以被传递、赋值给变量、存储到数据结构中等。`FunctionN`特质还定义了一些辅助方法，这些方法在库级别实现与函数相关的功能，而不是在语言语法中。一个例子是将普通函数转换为`curried`形式，对于`Function2[T1,T2,R]`，定义为`def curried: T1 => T2 => R = (x1: T1) => (x2: T2) => apply(x1, x2)`

```java
scala> objectOrientedA.curried
res9: Long => (Long => Long)
```

这个方法适用于任何函数。

# 局部函数

有额外方法的可能性提供了一种定义其他方式难以表述的概念的方法，至少在没有扩展语言本身的情况下是这样。一个这样的例子是*局部函数*。局部函数是一个对于其参数的一些值未定义的函数。经典的例子是除法在除数等于零时未定义。但实际上，可以存在任意的域规则，使得某些函数成为局部函数。例如，我们可以决定我们的字符串反转函数对于空字符串应该是未定义的。

在程序中实现此类约束有几个可能性：

+   对于函数未定义的参数抛出异常

+   限制参数的类型，使其只能传递有效的参数给函数，例如使用精炼类型

+   在返回类型中反映局部性，例如使用`Option`或`Either`

与这些方法中的每一个都有关明显的权衡，在 Scala 中，第一个方法因其最自然而被首选。但是，为了更好地模拟函数的局部性质，标准库中有一个特殊的特质可用：

```java
trait PartialFunction[-A, +B] extends (A => B)
```

与普通函数的关键区别是，有一个额外的方法可用，允许我们检查函数是否对某些参数有定义：

```java
def isDefinedAt(x: A): Boolean
```

这允许函数的使用者对“无效”的输入值执行不同的操作。

例如，让我们假设我们发明了一种非常高效的方法来检查一个字符串是否是回文。然后我们可以将我们的反转函数定义为两个局部函数，一个只对回文有效且不执行任何操作，另一个只对非回文有效并执行实际的反转操作：

```java
val doReverse: PartialFunction[String, String] = {
  case str if !isPalindrome(str) => str.reverse
}
val noReverse: PartialFunction[String, String] = {
  case str if isPalindrome(str) => str
}
def reverse = noReverse orElse doReverse

```

在这里，我们再次使用语法糖来定义我们的局部函数作为模式匹配，编译器为我们创建`isDefinedAt`方法。我们的两个局部函数通过`orElse`方法组合成总函数。

# 函数对象

局部函数的`orElse`方法和普通函数的`curried`方法只是标准库中预定义的与函数相关的方法的例子。

与为每个函数实例定义的`curried`方法类似（除了`Function0`和`Function1`），还有一个`tupled`方法，它将 N 个参数的函数转换为只有一个参数的函数，该参数是一个`TupleN`。

此外，还有一个伴随对象，`scala.Function`，它包含了一些对高阶函数式编程有用的方法，最显著的是一个`const`函数，它总是返回其参数，还有一个`chain`函数，它将一系列函数组合成一个单一函数，如下面的示例所示：

```java
val upper = (_: String).toUpperCase
def fill(c: Char) = c.toString * (_: String).length
def filter(c: Char) = (_: String).filter(_ == c)

val chain = List(upper, filter('L'), fill('*'))
val allAtOnce = Function.chain(chain)

scala> allAtOnce("List(upper, filter('a'), fill('C'))")
res11: String = ****
```

`allAtOnce`是一个函数，它类似于通过`andThen`（在`FunctionN`特质中定义）组合我们的三个原始函数可以构建的函数：

```java
val static = upper andThen filter('a') andThen fill('C')
```

但是`allAtOnce`是以动态方式构建的。

# 扩展函数

开发者不能像处理`PartialFunction`那样扩展`FunctionN`特质，尽管由于引用透明性约束带来的限制，这很少有意义。这意味着这种函数的实现不应该有共享状态，也不应该改变状态。

例如，可能会诱使人们将贷款模式实现为一个函数，这样在函数应用之后，使用的资源就会被自动关闭，但这不会是引用透明的，因此不会满足函数的要求。

下面是可能的实现方式：

```java
class Loan-T <: AutoCloseable, +R extends (T => R) {
  override def apply(t: T): R = try app(t) finally t.close()
}
```

这就是如果我们这样称呼它时会发生的情况：

```java
scala> new Loan((_: java.io.BufferedReader).readLine())(Console.in)
res13: String = Hello

scala> [error] (run-main-0) java.io.IOException: Stream Closed
[error] java.io.IOException: Stream Closed
[error] at java.io.FileInputStream.read0(Native Method)
[error] at java.io.FileInputStream.read(FileInputStream.java:207)
[error] at jline.internal.NonBlockingInputStream.read(NonBlockingInputStream.java:245)
...
```

不幸的是，甚至无法测试第二次调用是否会产生相同的结果（显然不会），因为我们通过关闭`Console.in`破坏了 REPL。

# 摘要

函数代表了 Scala 中面向对象和函数式特征融合的另一方面。它们可以通过多种方式定义，包括方法的偏应用、函数字面量和偏函数。函数可以在任何作用域中定义。如果一个函数封闭了作用域中可用的变量，它就被称为**闭包**。

多态函数实现了一个类似于面向对象中多态的概念，但应用于参数和结果类型。这被称为参数多态。当定义接受其他函数作为参数的函数时，这种思想特别有用，所谓的更高阶函数。

实现递归有两种方式，只有尾递归函数在 JVM 中是栈安全的。对于不能转换为尾递归的函数，有一种方法可以通过将调用链编码为对象来在堆中表示它。这种方法称为跳跃（trampolining），并且它在标准库中得到支持。

函数在 Scala 中是一等值，因为它们作为扩展`FunctionN`特质的匿名类来实现。这不仅使得像处理普通变量一样处理函数成为可能，而且还允许提供具有附加属性的扩展函数实现，例如，一个`PartialFunction`。

# 问题

1.  以下函数在`curried`形式中将会是什么类型：`(Int, String) => (Long, Boolean, Int) => String`？

1.  描述部分应用函数和偏函数之间的区别

1.  定义一个签名并实现一个用于三个参数的`curried`函数的`uncurry`函数 `A => B => C => R`

1.  实现一个头递归函数用于阶乘计算 n! = n * (n-1) * (n-2) * ... * 1

1.  实现一个尾递归函数用于阶乘计算

1.  使用跳跃技术实现一个用于阶乘计算的递归函数

# 进一步阅读

Mads Hartmann 和 Ruslan Shevchenko，*《专业 Scala*>：在一个让你为 JVM、浏览器等构建的环境下，编写简洁且易于表达的类型安全代码。

Vikash Sharma，*《Scala 编程学习*>：学习如何在 Scala 语言中编写可扩展和并发的程序，这是一种与你一起成长的编程语言。
