# 抽象和自类型

在软件工程中设计和编写高质量的代码对于拥有易于扩展和维护的应用程序非常重要。这项活动要求领域知识要被开发者充分了解，并且应用程序的要求要明确定义。如果其中任何一个条件不满足，编写好的程序就会变得相当具有挑战性。

通常，工程师会使用一些抽象来模拟 *世界*。这有助于代码的可扩展性和可维护性，并消除了重复，这在许多情况下可能是错误的原因。好的代码通常由多个小型组件组成，这些组件相互依赖并相互作用。有几种不同的方法可以帮助实现抽象和交互。我们将在本章中探讨以下主题：

+   抽象类型

+   多态

+   自类型

我们在这里将要讨论的主题，当我们开始研究一些具体的设计模式时，将会非常有用。了解它们也将帮助我们理解依赖于它们的设计模式。此外，本章中涵盖的概念本身对于编写好的代码就很有用。

# 抽象类型

使用值来参数化类是最常见的方法之一。这很简单，通过为类的构造函数参数传递不同的值来实现。在下面的示例中，我们可以为 `Person` 类的 `name` 参数传递不同的值，这就是我们创建不同实例的方式：

```java
case class Person(name: String)
```

这样，我们可以创建不同的实例并将它们区分开来，但这既不有趣也不需要火箭科学。进一步来说，我们将关注一些更有趣的参数化方法，这将帮助我们改进我们的代码。

# 泛型

泛型是另一种参数化类的方法。当我们要编写一个在多种类型中应用相同功能的功能时，它们非常有用，并且我们可以简单地推迟选择具体类型直到以后。每个开发者都应该熟悉的一个例子是集合类。例如，`List` 可以存储任何类型的数据，我们可以有整数列表、双精度浮点数列表、字符串列表、自定义类列表等等。然而，列表实现始终是相同的。

我们还可以参数化方法。例如，如果我们想实现加法，它不会在不同数值数据类型之间改变。因此，我们可以使用泛型，只需编写一次方法，而不是重载并试图适应世界上每一个类型。

让我们看看一些例子：

```java
trait Adder {
  def sumT(implicit numeric: Numeric[T]): T = 
   numeric.plus(a, b)
}
```

上一段代码稍微复杂一些，它定义了一个名为 `sum` 的方法，这个方法可以用于所有数值类型。这实际上是对 **ad hoc polymorphism** 的一个表示，我们将在本章后面讨论这个概念。

以下代码展示了如何参数化一个类以包含任何类型的数据：

```java
class ContainerT {
  def compare(other: T) = data.equals(other)
}
```

以下代码片段展示了几个示例用法：

```java
object GenericsExamples extends Adder {
  def main(args: Array[String]): Unit = {
    System.out.println(s"1 + 3 = ${sum(1, 3)}")
    System.out.println(s"1.2 + 6.7 = ${sum(1.2, 6.7)}")
    // System.out.println(s"abc + cde = ${sum("abc", "cde")}") // compilation fails

    val intContainer = new Container(10)
    System.out.println(s"Comparing with int: ${intContainer.compare(11)}")

    val stringContainer = new Container("some text")
    System.out.println(s"Comparing with string:
     ${stringContainer.compare("some text")}")
  }
}
```

这个程序的输出将如下所示：

```java
1 + 3 = 4
1.2 + 6.7 = 7.9
Comparing with int: false
Comparing with string: true
```

# 抽象类型

另一种通过抽象类型参数化类的方法。泛型在其他语言（如 Java）中也有对应物。然而，与它们不同，Java 中没有抽象类型。让我们看看前面的`Container`示例如何通过抽象类型而不是泛型来转换：

```java
trait ContainerAT {
  type T
  val data: T

  def compare(other: T) = data.equals(other)
}
```

我们将在类中使用这个特性，如下所示：

```java
class StringContainer(val data: String) extends ContainerAT {
  override type T = String
}
```

在完成这些之后，我们可以有与之前相同的例子：

```java
object AbstractTypesExamples {
  def main(args: Array[String]): Unit = {
    val stringContainer = new StringContainer("some text")
    System.out.println(s"Comparing with string:
     ${stringContainer.compare("some text")}")
  }
}
```

预期的输出如下：

```java
Comparing with string: true
```

当然，我们可以通过创建特例的实例并指定参数的方式，以类似的方式使用它，就像通用示例那样。这意味着泛型和抽象类型实际上给了我们两种不同的方式来实现相同的事情。

# 泛型与抽象类型比较

那么，为什么 Scala 中既有泛型又有抽象类型？它们之间有什么区别，何时应该使用一个而不是另一个？我们将在这里尝试回答这些问题。

泛型和抽象类型可以互换。我们可能需要做一些额外的工作，但最终，我们可以使用泛型得到抽象类型提供的东西。选择哪一个取决于不同的因素，其中一些是个人偏好，例如是否有人追求可读性或类的不同使用方式。

让我们看看一个例子，并尝试了解泛型和抽象类型何时以及如何被使用。在这个当前例子中，我们将讨论打印机。每个人都知道有不同类型——纸张打印机、3D 打印机等等。这些打印机各自使用不同的材料进行打印，例如碳粉、墨水或塑料，并且它们被用于打印在不同的媒体上，如纸张，或者在周围环境中。我们可以使用抽象类型来表示类似的东西：

```java
abstract class PrintData
abstract class PrintMaterial
abstract class PrintMedia
trait Printer {
  type Data <: PrintData
  type Material <: PrintMaterial
  type Media <: PrintMedia
  def print(data: Data, material: Material, media: Media) =
   s"Printing $data with $material material on $media media."
}
```

为了调用`print`方法，我们需要有不同的媒体、数据类型和材料：

```java
case class Paper() extends PrintMedia
case class Air() extends PrintMedia
case class Text() extends PrintData
case class Model() extends PrintData
case class Toner() extends PrintMaterial
case class Plastic() extends PrintMaterial
```

现在我们来制作两个具体的打印机实现，一个激光打印机和 3D 打印机：

```java
class LaserPrinter extends Printer {
  type Media = Paper
  type Data = Text
  type Material = Toner
}

class ThreeDPrinter extends Printer {
  type Media = Air
  type Data = Model
  type Material = Plastic
}
```

在前面的代码中，我们实际上给出了一些关于这些打印机可以使用的类型、媒体和材料的规范。这样，我们就不能要求我们的 3D 打印机使用碳粉打印东西，或者我们的激光打印机在空中打印。这就是我们将如何使用我们的打印机：

```java
object PrinterExample {
  def main(args: Array[String]): Unit = {
    val laser = new LaserPrinter
    val threeD = new ThreeDPrinter

    System.out.println(laser.print(Text(), Toner(), Paper()))
    System.out.println(threeD.print(Model(), Plastic(), Air()))
  }
}
```

前面的代码非常易于阅读，并且使我们能够轻松指定具体类。它使建模变得更容易。有趣的是看到前面的代码如何转换为泛型：

```java
trait GenericPrinter[Data <: PrintData, Material <: PrintMaterial, Media <: PrintMedia] {
  def print(data: Data, material: Material, media: Media) =
    s"Printing $data with $material material on $media media."
}
```

特性很容易表示，可读性和逻辑正确性在这里都没有受到影响。然而，我们必须以这种方式表示具体类：

```java
class GenericLaserPrinter[Data <: Text, Material <: Toner, Media <: Paper] extends GenericPrinter[Data, Material, Media]
class GenericThreeDPrinter[Data <: Model, Material <: Plastic, Media <: Air] extends GenericPrinter[Data, Material, Media]
```

这会变得相当长，开发者很容易出错。以下片段显示了如何创建实例和使用这些类：

```java
val genericLaser = new GenericLaserPrinter[Text, Toner, Paper]
val genericThreeD = new GenericThreeDPrinter[Model, Plastic, Air]
System.out.println(genericLaser.print(Text(), Toner(), Paper()))
System.out.println(genericThreeD.print(Model(), Plastic(), Air()))
```

在这里，我们可以看到，每次创建实例时都必须指定类型。想象一下，如果我们有超过三个泛型类型，其中一些可能基于泛型，例如集合。这可能会很快变得相当繁琐，并使代码看起来比实际更复杂。

另一方面，使用泛型允许我们重用`GenericPrinter`，而无需为每个不同的打印机表示显式地多次子类化它。然而，存在犯逻辑错误的风险：

```java
class GenericPrinterImpl[Data <: PrintData, Material <: PrintMaterial, Media <: PrintMedia] extends GenericPrinter[Data, Material, Media]
```

如果按照以下方式使用，可能会出错：

```java
val wrongPrinter = new GenericPrinterImpl[Model, Toner, Air]
System.out.println(wrongPrinter.print(Model(), Toner(), Air()))
```

# 使用建议

之前的例子展示了泛型和抽象类型使用之间的相对简单比较。这两个都是有用的概念；然而，了解确切正在做什么对于使用正确的一个来应对情况非常重要。以下是一些可以帮助做出正确决定的提示。

**使用泛型：**

+   如果你只需要类型实例化；一个很好的例子是标准集合类

+   如果你正在创建一系列类型

**使用抽象类型：**

+   如果你希望允许人们使用特性混合类型

+   如果你需要在两种类型可以互换的场景中提高可读性

+   如果你希望从客户端代码中隐藏类型定义

# 多态

多态是每个进行过一些面向对象编程的开发商都知道的东西。

多态帮助我们编写通用的代码，这些代码可以重用并应用于各种类型。

了解存在不同类型的多态很重要，我们将在本节中探讨它们。

# 子类型多态

这是每个开发商都知道的多态，它与在具体类实现中重写方法相关。考虑以下简单的层次结构：

```java
abstract class Item {
  def pack: String
}

class Fruit extends Item {
  override def pack: String = "I'm a fruit and I'm packed in a bag."
}

class Drink extends Item {
  override def pack: String = "I'm a drink and I'm packed in a bottle."
}
```

现在，让我们有一个包含物品的购物篮，并对每个物品调用`pack`：

```java
object SubtypePolymorphismExample {
  def main(args: Array[String]): Unit = {
    val shoppingBasket: List[Item] = List(
      new Fruit,
      new Drink
    )
    shoppingBasket.foreach(i => System.out.println(i.pack))
  }
}
```

如您所见，在这里我们可以使用抽象类型，只需调用`pack`方法，而不必考虑它确切是什么。多态将负责打印正确的值。我们的输出将如下所示：

```java
I'm a fruit and I'm packed in a bag.
I'm a drink and I'm packed in a bottle.
```

子类型多态使用`extends`关键字通过继承来表示。

# 参数多态

函数式编程中的参数多态是我们之前关于泛型的章节中展示的内容。泛型是参数多态，正如我们之前所看到的，它们允许我们在任何类型或给定类型的子集上定义方法或数据结构。然后可以在稍后的阶段指定具体类型。

# 临时多态

临时多态与参数多态相似；然而，在这种情况下，参数的类型很重要，因为具体的实现将依赖于它。它是在编译时解决的，与在运行时进行的子类型多态不同。这有点类似于函数重载。

我们在本章前面看到了一个例子，其中我们创建了可以求和不同类型的`Adder`特质。让我们再举一个例子，但更加精细，一步一步来，我们希望理解事物是如何工作的。我们的目标是拥有一个可以添加许多不同类型的`sum`方法：

```java
trait Adder[T] {
  def sum(a: T, b: T): T
}
```

接下来，我们将创建一个 Scala 对象，它使用这个`sum`方法并将其暴露给外界：

```java
object Adder {
  def sumT: Adder: T = implicitly[Adder[T]].sum(a, b)
}
```

在前面的代码中，我们看到的是 Scala 中的一些语法糖，`implicitly`表示存在从`T`类型到`Adder[T]`的隐式转换。我们现在可以编写以下程序：

```java
object AdhocPolymorphismExample {
  import Adder._
  def main(args: Array[String]): Unit = {
    System.out.println(s"The sum of 1 + 2 is ${sum(1, 2)}")
    System.out.println(s"The sum of abc + def is ${sum("abc", "def")}")
  }
}
```

如果我们尝试编译并运行这个程序，我们将遇到麻烦并得到以下错误：

```java
Error:(15, 51) could not find implicit value for evidence parameter of type com.ivan.nikolov.polymorphism.Adder[Int]
  System.out.println(s"The sum of 1 + 2 is ${sum(1, 2)}")
                                                 ^

Error:(16, 55) could not find implicit value for evidence parameter of type com.ivan.nikolov.polymorphism.Adder[String]
  System.out.println(s"The sum of abc + def is ${sum("abc", "def")}")
                                                      ^
```

这表明我们的代码不知道如何隐式地将整数或字符串转换为`Adder[Int]`或`Adder[String]`。我们必须做的是定义这些转换，并告诉我们的程序`sum`方法将做什么。我们的`Adder`对象将如下所示：

```java
object Adder {
  def sumT: Adder: T = implicitly[Adder[T]].sum(a, b)

  implicit val int2Adder: Adder[Int] = new Adder[Int] {
    override def sum(a: Int, b: Int): Int = a + b
  }

  // same implementation as above, but allowed when the trait has a single method
  implicit val string2Adder: Adder[String] =
    (a: String, b: String) => s"$a concatenated with $b"
}
```

如果我们现在编译并运行我们的应用程序，我们将得到以下输出：

```java
The sum of 1 + 2 is 3
The sum of abc + def is abc concatenated with def
```

此外，如果你记得本章开头的例子，我们无法在字符串上使用`sum`方法。正如你所看到的，我们可以提供不同的实现，只要我们定义了一种将类型转换为`Adder`的方法，使用它就不会有问题。

特设多态性允许我们在不修改基类的情况下扩展我们的代码。如果我们正在使用外部库，或者由于某种原因我们无法更改原始代码，这将非常有用。这非常强大，并且在编译时进行评估，确保我们的程序按预期工作。此外，它还允许我们为无法访问的类型（在我们的例子中是`Int`和`String`）提供函数定义。

# 为多种类型添加函数

如果我们回顾本章开头，我们让`Adder`与数值类型一起工作的地方，我们会看到我们的最后一个`Adder`实现将需要我们为每种不同的数值类型分别定义一个操作。有没有办法在这里实现本章开头展示的内容呢？是的，有，做法如下：

```java
implicit def numeric2Adder[T : Numeric]: Adder[T] = new Adder[T] {
  override def sum(a: T, b: T): T = implicitly[Numeric[T]].plus(a, b)
}
```

我们刚刚定义了另一个隐式转换，它将为我们处理正确的事情。现在，我们也可以编写以下代码：

```java
System.out.println(s"The sum of 1.2 + 6.5 is ${sum(1.2, 6.5)}")
```

特设多态性使用隐式表达式来混入行为。它是**类型类设计模式**的主要构建块，我们将在本书的后面部分探讨。

# 自类型

优秀代码的一个特点是关注点的分离。开发者应该努力使类及其方法只负责一件事情。这有助于测试、维护和更好地理解代码。记住——*简单总是更好*。

然而，在编写真实软件时，我们需要某些类的实例在其他类中，以便实现某些功能。换句话说，一旦我们的构建块被很好地分离，它们将具有依赖关系以执行其功能。我们在这里谈论的实际上归结为依赖注入。自定义类型提供了一种优雅地处理这些依赖关系的方法。在本节中，我们将看到如何使用它们以及它们有什么好处。

# 使用自定义类型

自定义类型允许我们轻松地将代码在我们的应用程序中分离，然后从其他地方调用它。一切都会通过例子变得更加清晰，所以让我们看看一个例子。假设我们想要能够将信息持久化到数据库中：

```java
trait Persister[T] {
  def persist(data: T)
}
```

`persist`方法将对数据进行一些转换，然后将其插入我们的数据库中。当然，我们的代码写得很好，所以数据库实现是分开的。对于我们的数据库，我们有以下内容：

```java
import scala.collection.mutable

trait Database[T] {
  def save(data: T)
}

trait MemoryDatabase[T] extends Database[T] {
  val db: mutable.MutableList[T] = mutable.MutableList.empty

  override def save(data: T): Unit = {
    System.out.println("Saving to in memory database.")
    db.+=:(data)
  }
}

trait FileDatabase[T] extends Database[T] {
  override def save(data: T): Unit = {
    System.out.println("Saving to file.")
  }
}
```

我们有一个基特质和一些具体的数据库实现。那么，我们如何将数据库传递给`Persister`呢？它应该能够调用数据库中定义的`save`方法。我们的可能性包括以下内容：

+   在`Persister`中扩展`Database`。然而，这将使`Persister`也成为`Database`的一个实例，我们不想这样。我们将在稍后展示原因。

+   在`Persister`中有一个`Database`变量的变量并使用它。

+   使用自定义类型。

我们在这里试图了解自定义类型是如何工作的，所以让我们使用这种方法。我们的`Persister`接口将变为以下内容：

```java
trait Persister[T] {
  this: Database[T] =>
  def persist(data: T): Unit = {
    System.out.println("Calling persist.")
    save(data)
  }
}
```

现在，我们可以访问`Database`中的方法，并在`Persister`内部调用`save`方法。

**命名自定义类型** 在前面的代码中，我们使用以下语句包含了我们的自定义类型——`this: Database[T] =>`。这允许我们直接访问包含它们的特质的成员方法，就像它们是特质的成员方法一样。在这里做同样的事情的另一种方式是写`self: Database[T] =>`。有很多例子使用后者方法，这在需要在一些嵌套特质或类定义中引用`this`时可以避免混淆。然而，使用这种方法调用注入依赖项的方法时，开发人员需要使用`self.`来访问所需的方法。

自定义类型要求任何将`Persister`混合进来的类也混合`Database`。否则，我们的编译将失败。让我们创建一些将数据持久化到内存和数据库的类：

```java
class FilePersister[T] extends Persister[T] with FileDatabase[T]
class MemoryPersister[T] extends Persister[T] with MemoryDatabase[T]
```

最后，我们可以在我们的应用程序中使用它们：

```java
object PersisterExample {
  def main(args: Array[String]): Unit = {
    val fileStringPersister = new FilePersister[String]
    val memoryIntPersister = new MemoryPersister[Int]

    fileStringPersister.persist("Something")
    fileStringPersister.persist("Something else")

    memoryIntPersister.persist(100)
    memoryIntPersister.persist(123)
  }
}
```

下面是我们程序的输出：

```java
Calling persist.
Saving to file.
Calling persist.
Saving to file.
Calling persist.
Saving to in memory database.
Calling persist.
Saving to in memory database.
```

自定义类型所做的是与继承不同的。它们需要一些代码的存在，因此允许我们很好地分割功能。这可以在维护、重构和理解程序方面产生巨大的差异。

# 需要多个组件

在实际应用程序中，我们可能需要使用多个使用自身类型的组件。让我们通过一个`History`特性来展示这个例子，该特性可能会在某个时刻跟踪更改以进行回滚。我们的例子只是打印：

```java
trait History {
  def add(): Unit = {
    System.out.println("Action added to history.")
  }
}
```

我们需要在我们的`Persister`特性中使用这个功能，它看起来是这样的：

```java
trait Persister[T] {
  this: Database[T] with History =>
  def persist(data: T): Unit = {
    System.out.println("Calling persist.")
    save(data)
    add()
  }
}
```

使用`with`关键字，我们可以添加我们想要的任何要求。然而，如果我们只是留下我们的代码更改，它将无法编译。原因是现在我们必须在每个使用`Persister`的类中混合`History`：

```java
class FilePersister[T] extends Persister[T] with FileDatabase[T] with History
class MemoryPersister[T] extends Persister[T] with MemoryDatabase[T] with History
```

就这样。如果我们现在运行我们的代码，我们会看到这个：

```java
Calling persist.
Saving to file.
Action added to history.
Calling persist.
Saving to file.
Action added to history.
Calling persist.
Saving to in memory database.
Action added to history.
Calling persist.
Saving to in memory database.
Action added to history.
```

# 冲突的组件

在前面的例子中，我们有一个对`History`特性的要求，它有一个`add()`方法。如果不同组件中的方法具有相同的签名并且它们发生冲突，会发生什么？让我们尝试这样做：

```java
trait Mystery {
  def add(): Unit = {
    System.out.println("Mystery added!")
  }
}
```

我们现在可以在我们的`Persister`特性中使用这个功能：

```java
trait Persister[T] {
  this: Database[T] with History with Mystery =>
  def persist(data: T): Unit = {
    System.out.println("Calling persist.")
    save(data)
    add()
  }
}
```

当然，我们将更改所有混合`Persister`的类：

```java
class FilePersister[T] extends Persister[T] with FileDatabase[T] with History with Mystery
class MemoryPersister[T] extends Persister[T] with MemoryDatabase[T] with History with Mystery
```

如果我们尝试编译我们的应用程序，我们会看到它导致以下错误消息：

```java
Error:(47, 7) class FilePersister inherits conflicting members:
  method add in trait History of type ()Unit and
  method add in trait Mystery of type ()Unit
(Note: this can be resolved by declaring an override in class FilePersister.)
class FilePersister[T] extends Persister[T] with FileDatabase[T] with History with Mystery
      ^

Error:(48, 7) class MemoryPersister inherits conflicting members:
  method add in trait History of type ()Unit and
  method add in trait Mystery of type ()Unit
(Note: this can be resolved by declaring an override in class MemoryPersister.)
class MemoryPersister[T] extends Persister[T] with MemoryDatabase[T] with History with Mystery
      ^
```

幸运的是，错误消息还包含信息，告诉我们如何修复问题。这与我们之前使用特性时看到的情况完全相同，我们可以提供以下修复方案：

```java
class FilePersister[T] extends Persister[T] with FileDatabase[T] with History with Mystery {
  override def add(): Unit ={
    super[History].add()
  }
}

class MemoryPersister[T] extends Persister[T] with MemoryDatabase[T] with History with Mystery {
  override def add(): Unit ={
    super[Mystery].add()
  }
}
```

运行示例后，我们将看到以下输出：

```java
Calling persist.
Saving to file.
Action added to history.
Calling persist.
Saving to file.
Action added to history.
Calling persist.
Saving to in memory database.
Mystery added!
Calling persist.
Saving to in memory database.
Mystery added!
```

# 自身类型和蛋糕设计模式

在我们前面的例子中，我们看到的是一个纯粹的依赖注入示例。我们要求一个组件通过自身类型在另一个组件中可用。

自身类型通常用于依赖注入。它们是**蛋糕设计模式**的主要部分，我们将在本书的后面部分熟悉它。

蛋糕设计模式完全依赖于自身类型。它鼓励工程师编写小型且简单的组件，这些组件声明并使用它们的依赖项。在应用程序中的所有组件都被编程之后，它们可以在一个公共组件注册表中实例化，并使它们对实际应用程序可用。蛋糕设计模式的一个很好的优点是，它实际上在编译时检查所有依赖项是否都能得到满足。我们将在本书的后面部分专门用一整节来介绍蛋糕设计模式，我们将提供更多关于如何实际连接模式、它有哪些优点和缺点等详细信息。

# 自身类型与继承

在前面的章节中，我们说我们不想使用继承来访问`Database`方法。为什么是这样？如果我们让`Persister`扩展`Database`，这意味着它将成为一个数据库本身（*is-a*关系）。然而，这是不正确的。它使用数据库来实现其功能。

继承将子类暴露于其父类的实现细节。然而，这并不总是我们所希望的。根据《设计模式：可复用面向对象软件元素》的作者，开发者应该优先选择对象组合而非类继承。

# 继承泄露功能

如果我们使用继承，我们也会泄露我们不希望子类拥有的功能。让我们看看以下代码：

```java
trait DB {
  def connect(): Unit = {
    System.out.println("Connected.")
  }

  def dropDatabase(): Unit = {
    System.out.println("Dropping!")
  }

  def close(): Unit = {
    System.out.println("Closed.")
  }
}

trait UserDB extends DB {
  def createUser(username: String): Unit = {
    connect()
    try {
      System.out.println(s"Creating a user: $username")
    } finally {
      close()
    }
  }

  def getUser(username: String): Unit = {
    connect()
    try {
      System.out.println(s"Getting a user: $username")
    } finally {
      close()
    }
  }
}

trait UserService extends UserDB {
  def bad(): Unit = {
    dropDatabase()
  }
}
```

这可能是一个现实生活中的场景。因为这就是继承的工作方式，我们会在`UserService`中获得对`dropDatabase`的访问权限。这是我们不想看到的事情，我们可以通过使用自身类型来修复它。DB 特质保持不变。其他所有内容都变为以下内容：

```java
trait UserDB {
  this: DB =>

  def createUser(username: String): Unit = {
    connect()
    try {
      System.out.println(s"Creating a user: $username")
    } finally {
      close()
    }
  }

  def getUser(username: String): Unit = {
    connect()
    try {
      System.out.println(s"Getting a user: $username")
    } finally {
      close()
    }
  }
}

trait UserService {
  this: UserDB =>

  // does not compile
  // def bad(): Unit = {
  // dropDatabase()
  //}
}
```

如代码注释所示，在这最后一个版本的代码中，我们无法访问`DB`特质的函数。我们只能调用所需类型的函数，这正是我们想要实现的目标。

# 摘要

在本章中，我们熟悉了一些有助于我们编写更好、更通用和可扩展的软件的概念。我们专注于 Scala 中的抽象类型、多态和自身类型。

我们探讨了类中泛型和抽象类型值的区别，并附带了一些示例和用法建议。然后，我们介绍了不同类型的多态——子类型、参数化和特定。最后，我们探讨了 Scala 中的自身类型及其使用方法。我们展示了自身类型提供了一种封装功能并编写模块化代码的好方法。

在下一章中，我们将探讨分离软件组件责任的重要性。我们还将介绍面向方面编程。
