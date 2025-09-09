# 第四章：Scala 集合

在上一章中，我们介绍了使用 Scala 的函数式编程以及面向对象和函数式方法如何相互补充。我们还介绍了泛型类，它们通常与模式匹配一起使用。最后，我们讨论了如何创建用户定义的模式匹配以及为什么它是有用的。

在本章中，我们将介绍 Scala 集合库。我们将从学习如何处理列表开始，这将使我们熟悉整个集合库的设计原则。之后，我们将推广到序列，并介绍一些相关的数据结构。最后，我们将探讨集合与单子之间的关系，以及我们如何利用这些知识在代码中创建一些强大的抽象。

Scala 的集合库非常丰富，由适用于非常不同用例和性能考虑的数据结构组成。它在不可变数据结构方面尤其丰富，我们将在本章中更详细地介绍。

Scala 集合库中可用的集合继承自常见的顶级抽象类和特质，因此它们共享一些共同的功能性，这使得一旦熟悉了某些方法和设计原则，使用它们就会变得更容易。

到本章结束时，你将能够：

+   识别标准库中可用的 Scala 集合

+   识别如何使用高阶函数抽象序列

+   实现与 Scala 集合一起工作的关键设计原则

# 与列表一起工作

列表可能是 Scala 程序中最常用的数据结构。学习如何处理列表不仅从数据结构的角度来看很重要，而且也是设计围绕递归数据结构的程序的一个切入点。

## 构建列表

为了能够使用列表，必须学习如何构建它们。`Lists` 是递归的，并且基于两个基本构建块：`Nil`（表示空列表）和 `::`（发音为 cons，来自大多数 Lisp 方言的 `cons` 函数）。

我们现在将在 Scala 中创建列表：

1.  启动 Scala `REPL`，它应该为你提供一个提示符：

    ```java
    $ scala
    ```

1.  使用以下方法创建一个字符串 `list`：

    ```java
    scala> val listOfStrings = "str1" :: ("str2" :: ("str3" :: Nil))
    listOfStrings: List[String] = List(str1, str2, str3)
    ```

1.  通过省略括号并得到相同的结果来展示 `::` 操作是右结合的：

    ```java
    scala> val listOfStrings = "str1" :: "str2" :: "str3" :: Nil
    listOfStrings: List[String] = List(str1, str2, str3)
    ```

1.  创建不同类型的列表。

1.  展示 `List` 伴生对象的 `apply` 方法提供了一个方便的方式来从可变数量的参数创建列表：

    ```java
    scala> val listOfStrings = List("str1", "str2", "str3")
    listOfStrings: List[String] = List(str1, str2, str3)
    ```

### 注意

如果你想知道 `::` 操作符如何是右结合的，请注意操作符的结合性由操作符的最后一个字符决定。以冒号 `:` 结尾的操作符是右结合的。所有其他操作符都是左结合的。由于 `::` 以冒号结尾，因此它是右结合的。

## 列表上的操作

`List` 类提供了 `head`、`tail` 和 `isEmpty` 方法。`head` 返回列表的第一个元素，而 `tail` 方法返回不包含第一个元素的列表。`isEmpty` 方法在列表为空时返回 `true`，否则返回 `false`。`head` 和 `tail` 只在非空列表上定义，并在空列表上抛出异常。

### 注意

在空列表（如 `Nil.head` 和 `Nil.tail`）中调用 `head` 和 `tail` 会抛出异常。

要使用以下签名实现 `evenInts` 方法，请使用以下代码：

```java
def evenInts(l: List[Int]): List[Int]

```

该方法应返回包含列表 l 中所有偶数的列表。使用 `List` 的 `head`、`tail` 和 `isEmpty` 方法。此问题的可能解决方案如下：

```java
def evenInts(l: List[Int]): List[Int] = {
  if (l.isEmpty) l
  else if (l.head % 2 == 0) l.head :: evenInts(l.tail)
  else evenInts(l.tail)
}
```

## 列表上的模式匹配

模式匹配是 Scala 中检查值与模式的一种强大机制，并提供了一种习惯用法来分解列表。您可以在 `::` 上进行模式匹配，它模仿列表结构，或在 `List(...)` 上进行匹配以匹配列表的所有值。

让我们在 Scala 的 `REPL` 中进行模式匹配实验。请确保展示使用 `List(...)` 和 `::` 的模式匹配示例。

下面是一个可能的示例：

```java
val l = List(1, 2, 3, 4, 5)
List(a, b, c, d, e) = l
val h :: t = l
```

使用模式匹配通常比使用 `if` 和 `else` 来结构化程序更符合习惯用法。

现在，我们将再次实现 `evenInts` 方法。这次，我们不会使用 `List` 的 `head`、`tail` 和 `isEmpty` 方法：

1.  打开我们已写入 `evenInts` 方法的文件。

1.  不要使用 `list` 的 `head`、`tail` 和 `isEmpty` 方法。

1.  此问题的可能解决方案如下：

    ```java
    def evenInts(l: List[Int]): List[Int] = l match {
      case h :: t if h % 2 == 0 => h :: evenInts(t)
      case _ :: t => evenInts(t)
      case Nil => Nil
    }
    ```

## 列表上的第一阶方法

`List` 类提供了各种有用的第一阶方法。第一阶方法是不接受函数作为参数的方法。我们将在以下小节中介绍一些最常用的方法。

## 添加和连接

我们已经学习了如何使用 `::` 在列表的头部添加一个元素。如果我们想在列表的末尾添加一个元素，我们可以使用 `:+` 操作符。为了连接两个列表，我们可以使用 `:::` 操作符。请注意，然而，`:+` 操作符的时间复杂度为 `O(n)`，其中 `n` 是列表中元素的数量。`:::` 操作符的时间复杂度也是 `O(n)`，`n` 是第一个列表中的元素数量。请注意，`:::` 操作符也具有右结合性，就像 `::` 操作符一样。

示例代码：

```java
scala> val a = List(1, 2, 3)
a: List[Int] = List(1, 2, 3)

scala> val b = List(4, 5, 6)
b: List[Int] = List(4, 5, 6)
scala> val c = a ::: b
c: List[Int] = List(1, 2, 3, 4, 5, 6)

scala> val d = b :+ 7
d: List[Int] = List(4, 5, 6, 7)
```

## 获取列表的长度

获取列表的长度是一个有用的操作。所有列表都有确定的大小，因此它们提供了返回其大小的 `length` 方法。我们将在另一个主题中介绍可能无限的数据结构。

注意，`length` 在列表上是一个昂贵的操作，因为它需要遍历整个列表以找到其末尾，所需时间与列表中元素的数量成比例。

## 反转列表

如果你需要频繁访问列表的末尾，反转一次并使用结果会更方便。`reverse` 方法创建一个新列表，其元素与原始列表相反。`reverse` 方法的复杂度为线性。

## 前缀和后缀

`List` 的 `take` 和 `drop` 方法返回列表的任意前缀或后缀。它们都接受一个整数作为参数：分别是要取或丢弃的元素数量。

示例代码：

```java
scala> val a = List(1, 2, 3, 4, 5)
a: List[Int] = List(1, 2, 3, 4, 5)

scala> val b = a.take(2)
b: List[Int] = List(1, 2)

scala> val c = a.drop(2)
c: List[Int] = List(3, 4, 5)
```

## 元素选择

即使对于列表来说这不是一个常见的操作，`List` 类通过其 `apply` 方法支持随机元素选择：

```java
scala> val a = List(1, 2, 3, 4, 5)
a: List[Int] = List(1, 2, 3, 4, 5)

scala> a.apply(2)
res0: Int = 3
```

由于在方法调用中对象出现在函数位置时，会隐式插入 `apply`，因此我们也可以这样做：

```java
scala> a(2)
res1: Int = 3
```

## 显示

使用 `toString` 获取列表的规范字符串表示：

```java
scala> val a = List(1, 2, 3, 4, 5)
a: List[Int] = List(1, 2, 3, 4, 5)

scala> a.toString
res0: String = List(1, 2, 3, 4, 5)
```

`mkString` 方法更加灵活，因为它允许你指定打印在所有元素之前的前缀、打印在元素之间的分隔符以及打印在所有元素之后的后缀。`mkString` 方法有两个重载变体，允许你在前缀和后缀参数为空字符串时省略它们。如果你想要一个空字符串作为分隔符，也可以不带参数调用 `mkString`：

```java
scala> a.mkString("[", ", ", "]")
res1: String = [1, 2, 3, 4, 5]
scala> a.mkString(", ")
res2: String = 1, 2, 3, 4, 5

scala> a.mkString
res3: String = 12345
```

### 注意

请参阅 [`www.scala-lang.org/api/current/scala/collection/immutable/List.html`](https://www.scala-lang.org/api/current/scala/collection/immutable/List.html) 上的 Scaladoc，了解 `scala.collection.` `immutable.List` 类。如果你对其他有用的方法感兴趣，可以查看该类提供的内容。

## 活动：使用列表为 Chatbot 创建新模式

在这个活动中，我们将构建一个新的模式，这个模式是我们在这本书的第一天创建的 Chatbot 的。这个新模式将能够保持和更新条目的 `todo` 列表。我们将使用 `lists` 作为主要的数据结构来存储我们的信息，并且我们希望至少支持以下命令：

+   `todo list`：列出机器人当前所知的所有当前项。

+   `todo new <item description>`：插入一个带有提供描述的新 TODO 项。

+   `todo done <item number>`：从列表中删除编号为 `<item number>` 的项。使用 `todo list` 时应显示项的编号。

+   `todo done <item description>`：删除与 `<item description>` 匹配的项。

1.  首先定义一个新的类，该类扩展 `ChatbotMode`。由于我们的 TODO 列表项只需要作为字符串建模，因此我们的新模式可以定义为 `case class TodoList(todos: List[String]) extends ChatbotMode`。

1.  实现所需的 `process` 方法。正则表达式可能有助于解析 `line` 参数。根据提供的输入，我们希望创建一个新的 `TodoList` 实例，其 `todos` 的值可能已修改。在无效输入（不可识别的命令或尝试删除不存在的项等）时返回 `None`。

1.  在之前实现的聊天机器人中实验你新定义的模式。看看它与其他已定义的模式如何协同工作。

在本节中，我们从 Scala 程序的主要工作马视角之一来介绍列表。我们学习了可以在列表上执行的操作，并介绍了 Scala 代码中处理列表的一些习惯用法。

# 在序列上抽象

所有 Scala 集合都源自一个名为`Traversable`的共同特质。Scala 集合采用的设计允许在几乎所有集合中使用类似高阶函数，并在特定实例中具有适当的返回类型。将集合视为序列或元素容器，允许无缝地使用不同的数据结构。

## 可遍历特质

集合层次结构的根部是`Traversable`特质。`Traversable`特质有一个抽象方法：

```java
def foreachU
```

此方法的实现足以使`Traversable`特质提供一系列有用的更高阶方法。

我们希望专注于`map`操作。`map`方法接受一个函数并将其应用于集合的每个元素。

让我们在 Scala 的 REPL 中实验`map`方法，并展示它如何应用于不同类型的集合。现在，创建一个将整数乘以 2 的函数，并将其应用于`List`和`Array`：

```java
scala> def f(i: Int) = i * 2
f: (i: Int)Int

scala> val l = List(1, 2, 3, 4).map(f)
l: List[Int] = List(2, 4, 6, 8)

scala> val a = Array(1, 2, 3, 4).map(f)
a: Array[Int] = Array(2, 4, 6, 8)
```

### 注意

注意`map`方法的返回类型根据其被调用的集合类型而变化。

`flatMap`略有不同。它接受一个从集合元素类型到另一个集合的函数，然后在该返回集合中“扁平化”。

作为`flatMap`方法的示例，考虑一个接受整数并创建一个填充 1 的整数大小列表的函数。看看当该函数通过`map`和`flatMap`应用于`list`时返回值是什么。

```java
scala> def f(v: Int): List[Int] = if (v == 0) Nil else 1 :: f(v - 1)
f: (v: Int)List[Int]

scala> val l = List(1, 2, 3, 4).map(f)
l: List[List[Int]] = List(List(1), List(1, 1), List(1, 1, 1), List(1, 1, 1, 1))

scala> val ll = List(1, 2, 3, 4).flatMap(f)
ll: List[Int] = List(1, 1, 1, 1, 1, 1, 1, 1, 1, 1)
```

### 注意

注意在`flatMap`调用中列表是如何被扁平化的。

这类操作与单子的操作非常相似。

单子是一个包装和操作序列化的机制。它提供了两个基本操作：`identity`，用于将值包装在单子中，以及`bind`，用于转换单子的底层值。`单子`将在第七章功能习惯中更详细地介绍，所以如果你现在还没有完全掌握它们的复杂性，请不要担心。

链接`flatMaps`的单子机制非常常见，以至于 Scala 在 for-comprehensions 中为此提供了特殊语法。

单子操作为程序员提供了一种抽象和链式计算的方法，其中`map`和`flatMap`是粘合剂。事实上，`map`和`flatMap`是高阶函数，换句话说，它们接受其他函数作为参数，这使得程序员可以在他们的代码中重用组件（函数）。

集合 API 提供的其他重要高阶函数是`folds`。一般来说，折叠提供了将容器中的元素与某些二元运算符组合的方法。折叠与减少的不同之处在于，使用折叠时你提供了一个起始值，而使用`reduce`时你只使用容器中的元素。`*Left`和`*Right`变体决定了元素组合的顺序。

我们现在将通过使用`foldLeft`来实现列表上的求和。这个问题的可能解决方案如下：

```java
def add(a: Int, b: Int) = a + b
def sum(l: List[Int]) = l.foldLeft(0)(add)
val res = sum(List(1, 2, 3, 4))
// Returns 10
```

## 迭代器

在 Scala 集合层次结构中，紧接在`Traversable`特质之下的是`Iterable`特质。`Iterable`是一个具有单个抽象方法的特质：

```java
def iterator: Iterator[A]
```

`Iterator`提供了一个方法来逐个遍历集合的元素。需要注意的是，`Iterator`是可变的，因为其大多数操作都会改变其状态。具体来说，在`iterator`上调用`next`会改变其`head`的当前位置。由于`Iterator`只是具有`next`和`hasNext`方法的东西，因此可以创建一个没有任何集合支持的迭代器。由于所有 Scala 集合也源自`Iterable`，因此它们都有一个`iterator`方法来返回其元素的`Iterator`。

## 流

**流**提供了一种惰性列表的实现，其中元素仅在需要时才被评估。流也具有递归结构，类似于`列表`，基于`#::`和`Stream.empty`构建块（类似于`::`和`Nil`）。最大的区别在于`#::`是惰性的，并且只有当需要其中的元素时才会评估尾部。流的一个重要特性是它们被缓存，所以如果值已经被计算过一次，就不会重新计算。这种方法的缺点是，如果你保留了对`Stream`头部的引用，你将保留对到目前为止已计算的`Stream`中所有元素的引用。

## 活动：使用流和迭代器实现斐波那契数列

在数学中，被称为斐波那契的序列是由加在数字之前的两个整数生成的数定义的。根据定义，该系列中的前两个整数应该是 1 和 1，或者 0 和 1。

使用流和迭代器实现斐波那契数的无限序列：

```java
lazy val fibIterator: Iterator[BigInt]
lazy val fibStream: Stream[BigInt]
```

这些实现的可能解决方案如下：

```java
lazy val fibStream: Stream[BigInt] = BigInt(0) #:: BigInt(1) #:: 
fibStream.zip(fibStream.tail).map { n => n._1 + n._2 }
lazy val fibIterator = new Iterator[BigInt] {
  var v1 = 0
  var v2 = 1
  val hasNext = true
  def next = {
    val res = v1
    v1 = v2
    v2 = res + v1
    res
  }
}
```

在本节中，我们介绍了`Traversable`作为在 Scala 中使用和推理集合的抽象方式。我们还介绍了迭代器和流以及它们在实现可能无限序列中的有用性。

# 其他集合

现在我们已经介绍了 Scala 标准库中的`List`和一些相关的`Traversable`，我们也应该访问 Scala 提供的一些其他有用的集合。尽管本节的理论材料较少，这意味着我们将有更多时间用于本章的最终活动。

## 集合

`Sets`是`Iterables`，不包含重复元素。`Set`类提供了检查集合中元素包含的方法，以及合并不同集合的方法。请注意，由于`Set`继承自`Traversable`，你可以将其应用于之前看到的所有高阶函数。由于其`apply`方法的特点，`Set`可以被视为一个类型为`A => Boolean`的函数，如果元素存在于集合中，则返回`true`，否则返回`false`。

## 元组

元组是一个能够包含任意数量不同类型元素的类。元组通过将元素括在括号中创建。元组的类型根据其元素的类型进行定义。

现在让我们按照以下步骤在 REPL 中创建元组并访问它们的元素：

1.  在`REPL`中创建一些元组并访问它们的元素。

1.  观察创建的元组的类型，以及它是如何依赖于封装元素的类型的。

1.  使用模式匹配作为解构元组的方法。

完整的代码如下：

```java
scala> val tup = (1, "str", 2.0)
tup: (Int, String, Double) = (1,str,2.0)

scala> val (a, b, c) = tup
a: Int = 1
b: String = str
c: Double = 2.0

scala> tup._1
res0: Int = 1

scala> tup._2
res1: String = str

scala> tup._3
res2: Double = 2.0

scala> val pair = 1 -> "str"
pair: (Int, String) = (1,str)
```

## 映射

`Map`是一个大小为两的元组序列（键/值对的配对），也称为映射或关联。`Map`不能有重复的键。关于 Scala 中的映射的一个有趣的事实是，`Map[A, B]`扩展了`PartialFunction[A, B]`，因此你可以在需要`PartialFunction`的地方使用`Map`。

### 注意

如需更多信息，请参阅 Map 特质的 Scaladoc，链接如下：[`www.scala-lang.org/api/current/scala/collection/Map.html`](https://www.scala-lang.org/api/current/scala/collection/Map.html)。

## 可变和不可变集合

到目前为止，我们主要介绍的是不可变集合（除了`Iterators`，因为大多数操作都会改变其状态，所以它是固有的可变的——请注意，从 Scala 集合的`iterator`方法获得的迭代器不期望改变底层集合）。然而，值得注意的是，Scala 还提供了`scala.collection.mutable`包中的一组可变集合。可变集合提供在原地更改集合的操作。

在同一位置使用不可变和可变集合的一个有用约定是导入`scala.collection.mutable`包，并在集合声明前使用可变关键字作为前缀，即`Map`与`mutable.Map`。

以下代码显示了 Scala 中不可变和可变映射之间的区别，显示后者有一个`update`方法，该方法会原地更改集合：

```java
scala> import scala.collection.mutable
import scala.collection.mutable

scala> val m = Map(1 -> 2, 3 -> 4)
m: scala.collection.immutable.Map[Int,Int] = Map(1 -> 2, 3 -> 4)

scala> val mm = mutable.Map(1 -> 2, 3 -> 4)
mm: scala.collection.mutable.Map[Int,Int] = Map(1 -> 2, 3 -> 4)

scala> mm.update(3, 5)

scala> mm
res1: scala.collection.mutable.Map[Int,Int] = Map(1 -> 2, 3 -> 5)

scala> m.update(3, 5)
<console>:14: error: value update is not a member of scala.collection.immutable.Map[Int,Int]
       m.update(3, 5)
         ^

scala> m.updated(3, 5)
res3: scala.collection.immutable.Map[Int,Int] = Map(1 -> 2, 3 -> 5)

scala> m
res4: scala.collection.immutable.Map[Int,Int] = Map(1 -> 2, 3 -> 4)
```

## 活动：实现汉诺塔问题

我们想要创建一个解决汉诺塔问题的求解器。如果你不熟悉这个谜题，请访问维基百科页面[`en.wikipedia.org/wiki/Tower_of_Hanoi`](https://en.wikipedia.org/wiki/Tower_of_Hanoi)。这是了解它的一个好起点：

1.  实现以下函数的`paths`内部函数：

    ```java
    def path(from: Int, to: Int, graph: Map[Int, List[Int]]): List[Int] = {
      def paths(from: Int): Stream[List[Int]] = ???
      paths(from).dropWhile(_.head != to).head.reverse
    }
    ```

    `path`函数应该返回`graph`图中从`from`节点到`to`节点的最短路径。`graph`被定义为编码为`Map[Int, List[Int]]`的邻接表。`path`的`inner`函数应该返回按长度递增的路径`Stream`（以广度优先搜索的方式）。

1.  实现以下`nextHanoi`函数：

    ```java
    type HanoiState = (List[Int], List[Int], List[Int])
    def nextHanoi(current: HanoiState): List[HanoiState]
    ```

    `nextHanoi`函数应该返回一个列表，其中包含从当前`HanoiState`可以实现的合法状态。例如：`nextHanoi((List(1, 2, 3), Nil, Nil))`应该返回`List((List(2, 3),List(1),List()), (List(2, 3),List(),List(1)))`。

1.  将之前实现的路径方法泛化，使其参数化为我们正在操作的状态类型：

    ```java
    def genericPathA: List[A] = {
     def paths(current: A): Stream[List[A]] = ???
      paths(from).dropWhile(_.head != to).head.reverse
    }
    ```

1.  使用这个新的实现，你应该能够通过调用，例如，来解决汉诺塔问题：

    ```java
    val start = (List(1, 2, 3), Nil, Nil)
    val end = (Nil, Nil, List(1, 2, 3))
    genericPath(start, end, nextHanoi)
    ```

1.  建议的活动的可能实现如下：

    ```java
    // Does not avoid already visited nodes
    def path(from: Int, to: Int, graph: Map[Int, List[Int]]): List[Int] = {
      def paths(current: Int): Stream[List[Int]] = {
        def bfs(current: Stream[List[Int]]): Stream[List[Int]] = {
          if (current.isEmpty) current
          else current.head #:: bfs(current.tail #::: graph(current.head.head).map(_ :: current.head).toStream)
        }

        bfs(Stream(List(current)))
      }

      paths(from).dropWhile(_.head != to).head.reverse
    }

    type HanoiState = (List[Int], List[Int], List[Int])

    def nextHanoi(current: HanoiState): List[HanoiState] = {
      def setPile(state: HanoiState, i: Int, newPile: List[Int]): HanoiState = i match {
    …
    …
     genericPath(start, end, nextHanoi).size
    ```

# 摘要

在本章中，我们介绍了 Scala 集合库。我们介绍了如何处理列表，这将使我们熟悉整个集合库的一些设计原则。我们还介绍了如何泛化到序列，并介绍了一些相关的数据结构。最后，我们还介绍了集合与单子之间的关系，以及我们如何利用这些知识在我们的代码中使用一些强大的抽象。

在下一章中，我们将介绍`type`系统和多态性。我们还将介绍不同类型的变异性，这提供了一种约束参数化类型的方法。最后，我们还将介绍一些高级`type`，如抽象类型成员、选项等。
