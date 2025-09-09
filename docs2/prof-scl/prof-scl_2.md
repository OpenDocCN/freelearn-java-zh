# 第二章. 基本语言特性

在上一章中，我们学习了设置开发环境的各个方面，其中我们涵盖了 Scala 项目的结构，并确定了使用`sbt`构建和运行项目。我们介绍了 REPL，它是运行 Scala 代码的命令行界面，以及如何在 IDEA IDE 中开发和运行代码。最后，我们实现了与我们的简单`chatbot`应用程序的交互。

在本章中，我们将探索所谓的 Scala 的“OO”部分，它允许我们构建类似于任何主流语言（如 Java 或 C++）的类似结构。Scala 的面向对象部分将涵盖类和对象、特性、模式匹配、案例类等。最后，我们将把所学的面向对象概念应用到我们的聊天机器人应用程序中。

通过观察编程范式的历史，我们会注意到第一代高级编程语言（Fortran、C、Pascal）是过程导向的，没有面向对象或函数式编程功能。然后，面向对象在 20 世纪 80 年代成为编程语言的热门话题。

到本章结束时，你将能够做到以下事情：

+   识别非平凡 Scala 程序的结构

+   识别如何使用主要面向对象功能：对象、类和特性

+   识别函数调用语法和参数传递模式的细节

# 对象、类和特性

Scala 是一种多范式语言，它结合了函数式和面向对象编程。现在，我们将探索 Scala 的传统面向对象编程功能：对象、类和特性。

这些功能在某种意义上是相似的，因为每个都包含一些数据集和方法，但在生命周期和实例管理方面是不同的：

+   当我们需要一个只有一个实例的类型（例如单例）时，会使用对象。

+   当我们需要许多实例，并且可以使用 new 运算符创建时，会使用类。

+   特性用于将混合到其他类中

注意，没有必要在代码中导航，因为这已经在示例中暴露了。

## 对象

在上一章中，我们看到了一个对象。现在让我们浏览我们的代码库，并在`Lesson 2/3-project`目录下打开名为`Main`的文件：

```java
object Chatbot3 {
val effects = DefaultEffects
def main(args: Array[String]): Unit = {
   ….}
   def createInitMode() = (Bye or CurrentDate or CurrentTime) otherwise InterestingIgnore
}
```

这只是一组定义，被组合成一个对象，并且是静态可用的。也就是说，这是单例模式的实现：我们只有一个给定类型的对象实例。

在这里，我们可以看到值的定义（`val effects`）和主要函数。语法或多或少是可见的。一个不那么明显的事情是，所表示的`val`和`var`定义不是普通字段，而是内部字段和函数对：`var-s`的`getter`和`setter`函数。这允许通过`val-s`覆盖`def-s`。

注意，对象定义中的名称是对象的名称，而不是类型的名称。对象的类型`Chatbot3`可以通过`Chatbot3.type`访问。

让我们定义对象并调用一个方法。我们还将尝试将对象分配给一个变量。

### 注意

你应该在 IDEA 中打开 `project-3`。

1.  导航到项目结构并找到 `com.packt.courseware.l3` 包。

1.  右键单击并从上下文菜单中选择 `创建类`。

1.  在名称字段中输入 `ExampleObject`，并在表单的类型字段中选择 `object`。

1.  IDEA 将在对象中生成文件。

1.  在对象定义中插入以下内容：

    ```java
       def hello(): Unit = {
           println("hello")
        }
      - navigate to main object
       Insert before start of main method:
        val example = ExampleObject
       Insert at the beginning of the main method:
         example.hello()
    ```

## 类

类是抽象的下一步。以下是一个类定义的示例：

```java
package com.packt.courseware.l4
import math._
class PolarPoint(phi:Double, radius:Double) extends Point2D
{
require(phi >= - Pi && phi < Pi )
require(radius >= 0)
def this(phi:Double) = this(phi,1.0)
override def length = radius
def x: Double = radius*cos(phi)
def y: Double = radius*sin(phi)
def * (x:Double) = PolarPoint(phi,radius*x)
}
```

这里是一个在类定义中指定了参数（`phi`，`radius`）的类。类方法之外（如 require 语句）的语句构成了主构造函数的主体。

下一个定义是一个次要构造函数，它必须在第一行调用主构造函数。

我们可以使用 `new` 运算符创建对象实例：

```java
val p = new PolarPoint(0)
```

默认情况下，成员访问修饰符是 `public`，因此一旦我们创建了一个对象，我们就可以使用它的方法。当然，也可以将方法定义为 `protected` 或 `private`。

有时，我们希望构造函数参数在类成员的角色中可用。为此存在一种特殊语法：

```java
case class PolarPoint(val phi:Double, val radius:Double) extends Point2D
```

如果我们将 `val` 作为构造函数参数的修饰符（`phi`），那么 `phi` 就成为类的一个成员，并将作为字段可用。

如果你浏览典型 Scala 项目的源代码，你会注意到与类定义一起经常定义一个与类同名的对象。这样的对象被称为类的 `companion` 对象：

```java
object PolarPoint{
def apply(phi:Double, r:Double) = new PolarPoint(phi,r)
}
```

这通常是实用函数的典型位置，在 Java 世界中通常由 `static` 方法表示。

方法名称也存在，这允许你在调用侧使用特殊语法糖。我们稍后会告诉你所有这些方法。现在，我们将讨论 `apply` 方法。

当一个方法命名为 `apply` 时，它可以通过函数调用括号来调用（例如，如果 `apply` 在 `x` 中定义，则 `x(y)` 与 `x.apply(y)` 相同）。

传统上，伴随对象的 `apply` 方法通常用于实例创建，以允许不使用 `new` 运算符的语法。所以，在我们的例子中，`PolarPoint(3.0,5.0)` 将被解耦为 `PolarPoint.apply(3.0,5.0)`。

现在，让我们定义一个带有 `length` 方法的案例类 `CartesianPoint`。

1.  确保在 IDE 中打开了 `Lesson 2/4-project` 项目。

1.  使用名称 `CartesianPoint` 创建一个新的 Scala 类。

1.  代码应该像这样：

    ```java
    case class CartesianPoint(x:Double, y:Double) extends Point2D {
    override def length(): Double = x*x + y*y
    }
    ```

## 平等性和案例类

通常，存在两种类型的平等：

+   **扩展性**，当两个对象的所有外部属性都相等时，它们是相等的。

    +   在 JVM 中，用户可以通过重写对象的 `equals` 和 `hashCode` 方法来实现这种行为。

    +   在 Scala 表达式中，如果 `x` 是引用类型（例如，一个类或对象），则 `x == y` 是 `x.equals(y)` 的快捷方式。

+   **有意**（或引用），其中具有相同属性的两个对象可以不同，因为它们是在不同的时间和环境中创建的。

    +   在 JVM 中，这是引用的比较；（Java 中的`(x == y)`和 Scala 中的`(x eq y)`）。

看看我们的`PolarPoint`，如果我们想`PolarPoint(0,1)`等于`PolarPoint(0,1)`，那么我们必须重写`equals`和`hashCode`。

Scala 语言提供了一种类风味，它将自动执行这项工作（以及其他一些工作）。

让我们看看`case`类：

```java
case class PolarPoint(phi:Double, radius:Double) extends Point2D
```

当我们将一个类标记为案例类时，Scala 编译器将生成以下内容：

+   `equals`和`hashCode`方法，这些方法将通过组件比较类

+   `A toString`方法，它将输出组件

+   `A copy`方法，它将允许你创建类的副本，其中一些字段已更改：

    ```java
    val p1 = PolarPoint(Pi,1)
    val p2 = p1.copy(phi=1)
    ```

+   所有参数构造函数都将成为类值（因此，我们不需要写`val`）

+   具有 apply 方法（用于构造器快捷方式）和`unapply`方法（用于案例模式中的解构）的类的伴生对象

现在，我们将探讨说明值相等和引用相等之间的差异。

1.  在`test/com.packt.courseware.l4`中创建一个工作表。

    ### 注意

    要创建工作表，导航到包，然后右键单击并从下拉菜单中选择创建 Scala 工作表。

1.  在导入之后，在此文件中定义一个非案例类，包含字段：

    ```java
    class NCPoint(val x:Int, val y:Int)
    val ncp1 = new NCPoint(1,1)
    val ncp2 = new NCPoint(1,1)
    ncp1 == ncp2
    ncp1 eq ncp2
    ```

    ### 注意

    注意，结果是`false`。

1.  定义具有相同字段的案例类：

    ```java
    case class CPoint(x:Int, y:Int)
    ```

1.  编写一个类似的测试。注意差异：

    ```java
    val cp1 = CPoint(1,1)val cp2 = CPoint(1,1)cp1 == cp2cp1 eq cp2
    ```

## 模式匹配

**模式匹配**是一种构造，它最初于 1972 年左右引入到 ML 语言家族中（另一种类似的技术也可以被视为模式匹配的前身，这发生在 1968 年的 REFAL 语言中）。在 Scala 之后，大多数新的主流编程语言（如 Rust 和 Swift）也开始包含模式匹配结构。

让我们看看模式匹配的使用：

```java
val p = PolarPoint(0,1)
val r = p match {
case PolarPoint(_,0) => "zero"
case x: PolarPoint if (x.radius == 1) => s"r=1, phi=${x.phi}"
case v@PolarPoint(x,y) => s"(x=${x},y=${y})"
case _ => "not polar point"
}
```

在第二行，我们看到一个 match/case 表达式；我们将`p`与 case-e 子句的序列进行匹配。每个 case 子句包含一个模式和主体，如果匹配的表达式满足适当的模式，则评估主体。

在这个例子中，第一个案例模式将匹配任何半径为`0`的点，即`_`匹配任何。

第二 - 这将满足任何半径为 1 的`PolarPoint`，如可选模式条件中指定的。注意新值（`x`）被引入到主体上下文中。

第三 - 这将匹配任何点；将`x`和`y`绑定到`phi`和相应的`radius`，并将`v`绑定到模式（`v`与原始匹配模式相同，但具有正确的类型）。

最终的案例表达式是一个`default`案例，它匹配`p`的任何值。

注意，模式可以嵌套。

如我们所见，案例类可以参与案例表达式，并提供将匹配值推入主体内容的方法（这是解构）。

现在，是时候使用 match/case 语句了。

1.  在当前项目的测试源中创建一个名为 `Person` 的类文件。

1.  创建一个名为 `Person` 的案例类，包含成员 `firstName` 和 `lastName`: 

    ```java
    case class Person(firstName:String,lastName:String)
    ```

1.  创建一个伴随对象并添加一个接受 `person` 并返回 `String` 的方法：

    ```java
    def classify(p:Person): String = {
    // insert match code here .???
    }
    }
    ```

1.  创建一个 `case` 语句，它将打印：

    +   如果人的名字是 "Joe"，则输出 "`A`"

    +   如果人不满足其他情况，则输出 "`B`"

    +   如果 `lastName` 以小写字母开头，则输出 "`C`"

1.  为此方法创建一个测试用例：

    ```java
    class PersonTest extends FunSuite {
    test("Persin(Joe,_) should return A") {
    assert( Person.classify(Person("Joe","X")) == "A" )
    }
    }
    }
    ```

## 特性

特性用于对方法和值进行分组，这些方法和值可以在其他类中使用。特性的功能与其他特性和类混合，在其他语言中，这种适当的构造被称为 `mixins`。在 Java 8 中，接口类似于特性，因为可以定义默认实现。但这并不完全准确，因为 Java 的默认方法不能完全参与继承。

让我们看看以下代码：

```java
trait Point2D {
def x: Double
def y: Double
def length():Double = x*x + y*y}
```

这里有一个特性，它可以被 `PolarPoint` 类扩展，或者与 `CartesianPoint` 使用以下定义一起使用：

```java
case class CartesianPoint(x:Double, y:Double) extends Point2D
```

特性的实例不能创建，但可以创建扩展特性的匿名类：

```java
val p = new Point2D {override def x: Double = 1
override def y: Double = 0}
assert(p.length() == 1)
```

这里是一个特性的示例：

```java
trait A {
def f = "f.A"
}
trait B {def f = "f.B"def g = "g.B"
}
trait C extends A with B {override def f = "f.C" // won't compile without override.
}
```

正如我们所见，冲突的方法必须被覆盖：

然而，还有一个谜题：

```java
trait D1 extends B1 with C{override def g = super.g}
trait D2 extends C with B1{override def g = super.g}
```

`D1.g` 的结果将是 `g.B`，而 `D2.g` 将是 `g.C`。这是因为特性被线性化为序列，其中每个特性覆盖了前一个特性中的方法。

现在，让我们尝试在特性层次结构中表示菱形。

创建以下实体：

`Component` – 一个具有 `description()` 方法的 `base` 类，该方法输出组件的描述。

`Transmitter` – 一个生成信号并具有名为 `generateParams` 的方法的组件。

`Receiver` – 一个接受信号并具有名为 `receiveParams` 的方法的组件。

无线电 – 一个 `Transmitter` 和 `Receiver`。编写一组特性，其中 `A` 被建模为继承。

这个问题的答案应该是以下内容：

```java
trait Component{
def description(): String
}
trait Transmitter extends Component{
def generateParams(): String
}
trait Receiver extends Component{
def receiverParame(): String
}
trait Radio extends Transmitter with Receiver
```

## 自类型

在 Scale-trait 中，有时可以看到自类型注解，例如：

### 注意

对于完整的代码，请参阅 `Code Snippets/Lesson 2.scala` 文件。

```java
trait Drink
{
 def baseSubstation: String
 def flavour: String
 def description: String
}

trait VanillaFlavour
{
 thisFlavour: Drink =>

 def flavour = "vanilla"
 override def description: String = s"Vanilla ${baseSubstation}"
}

trait SpecieFlavour
{
 thisFlavour: Drink =>

 override def description: String = s"${baseSubstation} with ${flavour}"
}

trait Tee
{
  thisTee: Drink =>

  override def baseSubstation: String = "tee"

  override def description: String = "tee"

    def withSpecies: Boolean = (flavour != "vanilla")
}
```

这里，我们看到 `identifier => {typeName}` 前缀，这通常是一个自类型注解。

如果指定了类型，则该特性只能混合到该类型中。例如，`VanillaTrait` 只能与 `Drink` 混合。如果我们尝试将其与另一个对象混合，我们将收到一个错误。

### 注意

如果 `Flavor` 不是从 `Drink` 扩展，但可以访问 `Drink` 方法，如 `Flavor` 中的外观，那么我们将其置于 `Drink` 内部。

此外，可以使用自注解而不指定类型。这在嵌套特性中很有用，当我们想要调用封装特性的 "this" 时：

```java
trait Out{
thisOut =>

trait Internal{def f(): String = thisOut.g()
  def g(): String = .
  }
def g(): String = ….
}
```

有时，我们可以将一些大型类的组织看作是一组特质，围绕一个“基础”分组。我们可以将这种组织看作是“蛋糕”，它由自注解的特质“Pieces:”组成。我们可以通过更改混入的特质来改变一个部分到另一个部分。这种代码组织方式被称为“蛋糕模式”。请注意，使用蛋糕模式通常是有争议的，因为它很容易创建一个“上帝对象”。另外，请注意，在蛋糕模式内部重构类层次结构更难实现。

现在，让我们探索注解。

1.  创建一个带有`VanillaFlavour`的 Tee 实例，该实例引用`description`：

    ```java
    val tee = new Drink with Tee with VanillaFlavour
    val tee1 = new Drink with VanillaFlavour with Tee
    tee.description
    tee1.description
    ```

1.  尝试在`Tee`类中覆盖描述：

    在`Drinks`文件中取消注释`Tee`: `def description = plain tee`。

    检查是否有错误信息出现。

1.  创建第三个对象，从`Drink`派生，带有`Tee`和`VanillaFlavour`，具有重载的描述：

    ```java
    val tee2 = new Drink with Tee with VanillaFlavour{
    override def description: String ="plain vanilla tee"
    }
    ```

    ### 注意

    对于完整代码，请参考`Code Snippets/Lesson 2.scala`文件。

还要注意，存在特殊的方法语法，必须在覆盖方法之后进行“混合”，例如：

```java
trait Operation
{

  def doOperation(): Unit

}

trait PrintOperation
{
  this: Operation =>

  def doOperation():Unit = Console.println("A")
}

trait LoggedOperation extends Operation
{
  this: Operation =>

  abstract override def doOperation():Unit = {
    Console.print("start")
    super.doOperation()
    Console.print("end")
  }
}
```

我们可以看到标记为`abstract override`的方法可以调用`super`方法，这些方法实际上是在特质中定义的，而不是在这个基类中。这是一种相对罕见的技巧。

## 特殊类

有几个类具有特殊语法，在 Scala 类型系统中起着重要作用。我们将在稍后详细讨论这个问题，但现在让我们只列举一些：

+   **函数**：在 Scala 中，这可以编码为`A => B`。

+   **元组**：在 Scala 中，这可以编码为`(A,B), (A,B,C)`等等，这是`Tuple2[A,B]`、`Tuple3[A,B,C]`等的语法糖。

# 我们聊天机器人的面向对象

现在我们已经了解了理论基础知识，让我们看看这些设施以及它们在我们程序中的使用方式。让我们在我们的 IDE 中打开`Lesson 2/3-project`并扩展我们在上一章中开发的聊天机器人。

## 解耦逻辑和环境

要做到这一点，我们必须解耦环境和逻辑，并在`main`方法中仅集成一个。

让我们打开`EffectsProvider`类：

### 注意

对于完整代码，请参考`Code Snippets/Lesson 2.scala`文件。

```java
trait EffectsProvider extends TimeProvider {

 def input: UserInput

 def output: UserOutput

}

object DefaultEffects extends EffectsProvider
{
 override def input: UserInput = ConsoleInput

 override def output: UserOutput = ConsoleOutput

 override def currentTime(): LocalTime = LocalTime.now()

 override def currentDate(): LocalDate = LocalDate.now()
}
```

在这里，我们将所有效果封装到我们的特质中，这些特质可以有不同实现。

例如，让我们看看`UserOutput`：

对于完整代码，请参考`Code Snippets/Lesson 2.scala`文件。

```java
trait UserOutput {

 def write(message: String): Unit

 def writeln(message: String): Unit = {
  write(message)
  write("\n")
 }

}

object ConsoleOutput extends UserOutput
{

 def write(message: String): Unit = {
  Console.print(message)
 }
}
```

在这里，我们可以看到特性和对象，它们实现了当前特质。这样，当我们需要接受来自标准输入之外的命令，比如来自聊天机器人 API 或 Twitter 的命令时，我们只需要更改`UserOutput`/`ConsoleOutput`接口的实现。

现在是时候实现`ConsoleOutput`和`DefaultTimeProvider`了。

在`main`中将`???`替换为适当的构造函数。

实现`ConsoleOutput`和`DefaultTimeProvider`的步骤如下：

1.  确保在 IDE 中打开`Lesson 2/3-project`。

1.  在`UserOutput`文件中，找到`ConsoleOutput`文件，将`???`改为`write`方法的主体。结果方法应该看起来像这样：

    ```java
    object ConsoleOutput extends UserOutput{
    def write(message: String): Unit = {
    Console.print(message)
    }
    }
    }
    ```

1.  在`TimeProvider`文件中，添加一个扩展自`TimeProvider`并实现`currentTime`和`currentDate`函数的`DefaultTimeProvide`对象。结果代码应该看起来像这样：

    ```java
    object DefaultTimeProvider extends TimeProvider {
    override def currentTime(): LocalTime = LocalTime.now()

      override def currentDate(): LocalDate = LocalDate.now()
      }

     }
    ```

## 密封特性和代数数据类型

让我们处理第二个问题——让我们将聊天机器人模式的逻辑封装到特质中，这个特质将只处理逻辑，不处理其他任何事情。看看以下定义：

```java
trait ChatbotMode {
def process(message: String, effects: EffectsProvider): LineStepResult
def or(other: ChatbotMode): ChatbotMode = Or(this,other)
def otherwise(other: ChatbotMode): ChatbotMode = Otherwise(this,other)
}
```

现在，让我们忽略`or`和`otherwise`组合子，看看`process`方法。它接受输入消息和效果，并返回处理结果，这可能是一个失败或发送给用户的消息，带有模式的下一个状态：

```java
sealed trait LineStepResultcase class Processed(
  answer:String,
  nextMode: ChatbotMode,
 endOfDialog:Boolean) extends LineStepResult
case object Failed extends LineStepResult
```

在这里，我们可以看到一个新修饰符：`sealed`。

当一个特性（或类）被密封时，它只能在定义它的同一个文件中进行扩展。由于这个原因，你可以确信，在你的类家族中，没有人能够在你的项目中添加一个新的类。如果你使用 match/case 表达式进行用例分析，编译器可以进行详尽的检查：所有变体都存在。

从一个`sealed`特质扩展的 case 类/对象家族的构造通常被称为代数数据类型（ADT）。

这个术语来自 1972 年的 HOPE 语言（爱丁堡大学），在那里所有类型都可以通过代数运算从一个初始类型集合中创建：其中之一是一个命名的`product`（在 Scala 中看起来像 case 类）和`distinct union`（由密封特质和子类型建模）。

在领域建模中使用 ADT 是有益的，因为我们可以对领域模型进行明显的用例分析，并且没有弱抽象；我们可以实现各种设计，这些设计可以在未来的模型中添加。

回到我们的`ChatbotMode`。

在`bye`时，我们必须退出程序。

这很简单——只需定义适当的对象：

```java
object Bye extends ChatbotMode {
 override def process(message: String, effects: EffectsProvider): LineStepResult =
  if (message=="bye") {
   Processed("bye",this,true)
  } else Failed
}
```

现在，我们将查看为`CurrentTime`查询创建相同的模式。

### 注意

这个练习的代码可以在`Lesson 2/3-project`中找到。

1.  在`CurrentTime`模式包中创建一个新文件。

1.  在`Main`中的模式链中添加一个（例如，`createInitMode`的 Modify 定义）。

1.  确保通过时间功能检查的`test`通过。

下一步是从几个更简单的模式中创建一个更大的模式。让我们看看这个扩展了两个模式并可以选择能够处理传入消息的模式：

```java
case class Or(frs: ChatbotMode, snd: ChatbotMode) extends ChatbotMode
{
override def process(message: String, effects: EffectsProvider): LineStepResult ={
frs.process(message, effects) match {
case Processed(answer,nextMode,endOfDialog) => Processed(answer, Or(nextMode,snd),endOfDialog)
case Failed => snd.process(message,effects) match {
case Processed(answer,nextMode,endOfDialog) => Processed(answer, Or(nextMode,frs),endOfDialog)
case Failed => Failed}}
 }}
```

在这里，如果`frs`可以处理一条消息，那么处理这条消息的结果将被返回。它将包含一个答案。`NextMode`（它将接受下一个序列）与`frs`中的`nextMode`相同，处理结果和`snd`。

如果`frs`不能回答这个问题，那么我们尝试`snd`。如果`snd's`处理成功，那么，在下一个对话步骤中，第一个消息处理器将是一个`nextStep`，来自`snd`。这允许模式形成自己的对话上下文，就像一个理解你语言的人。这将是下次你问的第一个问题。

我们可以用这样的组合子将简单的模式链接成复杂的模式。Scala 允许我们使用花哨的语法进行链式调用：任何只有一个参数的方法都可以用作二元运算符。所以，如果我们定义`ChatbotMode`中的`or`方法，我们就能组合我们的模式：

```java
def or(other: ChatbotMode): ChatbotMode = Or(this,other)
```

然后在`main`中，我们可以写这个：

```java
 def createInitMode() = (Bye or CurrentDate or CurrentTime) otherwise InterestingIgnore
```

`Otherwise`看起来非常相似，只有一个区别：第二个模式必须始终是第二个。

当我们写一个时，它看起来像这样。

```java
def main(args: Array[String]): Unit = {
val name = StdIn.readLine("Hi! What is your name? ")
println(s" $name, tell me something interesting, say 'bye' to end the talk")
var mode = createInitMode()
var c = Processed("",mode,false)
while(!c.endOfDialog){
c = c.nextMode.process(effects.input.read(),effects) match {
case next@Processed(_,_,_) => next
case Failed => // impossible, but let be here as extra control.
      Processed("Sorry, can't understand you",c.nextMode,false)}
  effects.output.writeln(c.answer)}}
```

我们可以使其更好：让我们首先将交互（程序询问用户姓名的地方）移动到模式。

现在，我们将第一个交互移动到模式

这里，我们将创建一个`mode`，它记得你的名字并为你创建一个。

1.  定义一个新的对象，它实现了`chatbot`特质，当运行第一个单词`my name is`时，接受一个名字并回答`hi`，然后告诉你你的名字：

    ```java
    case class Name(name:String) extends ChatbotMode {
    override def process(message: String, effects: EffectsProvider): LineStepResult = {
    message match {
    case "my name" => if (name.isEmpty) {
    effects.output.write("What is your name?")
    val name = effects.input.read()
    Processed("hi, $name", Name(name), false)
    } else {
    Processed(s"your name is $name",this,false)}case _ =>  Failed
    }
    }
    }
    }
    ```

1.  将此对象添加到`main:`节点序列中

    ```java
    def createInitMode() = (Bye or CurrentDate or CurrentTime or Name("")) otherwise InterestingIgnore
    ```

1.  向 testcase 添加一个具有此功能的测试，注意自定义效果的使用：

    ### 注意

    对于完整的代码，请参阅`Code Snippets/Lesson 2.scala`文件。

    ```java
    test("step of my-name") {
      val mode = Chatbot3.createInitMode()
      val effects = new EffectsProvider {
        override val output: UserOutput = (message: String) => {}

        override def input: UserInput = () => "Joe"

        override def currentDate(): LocalDate = Chatbot3.effects.currentDate()

        override def currentTime(): LocalTime = Chatbot3.effects.currentTime()
      }
      val result1 = mode.process("my name",effects)
      assert(result1.isInstanceOf[Processed])
      val r1 = result1.asInstanceOf[Processed]
      assert(r1.answer == "Hi, Joe")
      val result2 = r1.nextMode.process("my name",effects)
      assert(result2.isInstanceOf[Processed])
      val r2 = result2.asInstanceOf[Processed]
      assert(r2.answer == "Your name is Joe")

    }
    ```

# 函数调用

现在，我们将看看如何在 Scala 中实现函数调用。

## 语法小技巧

Scala 提供了灵活的语法，值得花几分钟时间了解这个概念。

### 命名参数

以下是一个函数，`f(a:Int, b:Int)`。我们可以使用命名参数语法调用此函数：`f(a = 5, b=10)`。如果我们交换参数但保留正确的名称，方法仍然正确。

可以组合位置和命名函数调用——前几个参数可以是位置的。

例如：

```java
def f(x:Int, y:Int) = x*2 + y
f(x=1,y=2) // 4
f(y=1,x=2) // 5
```

### 默认参数

当指定一个函数时，我们可以设置默认参数。然后，稍后当我们调用此函数时，我们可以省略参数，编译器将使用默认值：

```java
def f(x:Int, y:Int=2) = x*2 + y
f(1) // 4
```

有可能通过命名和默认参数的组合创建一个舒适的 API。例如，对于具有 N 个组件的案例类，编译器生成一个具有 N 个参数的复制方法；所有这些都有默认值：

```java
case class Person(firstName: String, lastName: String)
val p1 = Person("Jon","Bull")
val p2 = p1.copy(firstName = "Iyan")
```

现在，让我们将`Or`和`Otherwise`组合器中的代码转换为使用`copy`方法而不是`Processed`构造函数。

1.  将情况表达式更改为类型，检查`(processed:Processed)`或向情况类模式添加`bind`变量（`processed@Processed(… )`）

1.  在情况体中，使用`copy`方法而不是`Processed`构造函数：

    +   生成的代码应如下所示：

    +   如果学生在情况表达式中使用类型检查：

        ```java
           case processed:Processed =>processed.copy(nextMode = Or(processed.nextMode,snd))
        ```

    +   如果学生使用绑定变量：

        ```java
          case processed@Processed(answer,nextMode,endOfDialog) =>
             processed.copy(nextMode = Or(nextMode,snd))
        ```

1.  对第二个匹配语句做同样的转换。

完整的代码如下所示：

```java
case class Or(frs: ChatbotMode, snd: ChatbotMode) extends ChatbotMode{
override def process(message: String, 
           effects: EffectsProvider): LineStepResult = {
   frs.process(message, effects) match {
   case processed@Processed(answer,nextMode,endOfDialog) =>
   processed.copy(nextMode = Or(nextMode,snd))
   case Failed => snd.process(message,effects) match {
   case processed@Processed(answer,nextMode,endOfDialog) =>
   processed.copy(nextMode=Or(nextMode,frs))
   case Failed => Failed
   }
   }
   }
   }
}
```

### 柯里化形式（多个参数列表）

柯里化是一个用于描述将多个参数的函数转换为单个参数的函数的术语。我们将在下一章详细描述这个过程。

对于语法，我们可以使用多个参数列表非常重要：

```java
def f1(x:Int,y:Int) = x + y 
def f2(x:Int)(y:Int) = x + y
```

在这里，`f2`是它的柯里化形式。它与`f1`具有相同的语义，但可以以不同的语法调用。这在需要视觉上分离参数时很有用。

### 特殊魔法方法

以下表格显示了各种魔法方法：

| x.apply(y,z) | x(y,z) |   |
| --- | --- | --- |
| x.update(y,z) | x(y)=z |   |
| x.y_=(z) | x.y=z | 方法 y 也必须被定义。 |
| x.unary- | -x | 同样适用于 +, ~, ! |
| x = x + y | x += y | 同样适用于 -,*,/,&#124;,& |

## 在`CartesianPoint`中实现`+`

从`Lesson2`打开之前的项目并实现`CartesianPoint`中的`+`。

1.  在你的 IDE 中，打开之前的项目（4-project，命名为`coordinates`）。

1.  在`CartesianPoint.scala`文件中，添加以下定义的`+`方法：

    ```java
     def +(v:CartesianPoint) = CartesianPoint(x+v.x,y+v.y)
    ```

## 参数传递模式

在本节中，我们将学习参数传递模式中的参数类型：`by value`、`by name`和`by need`。

## 通过值

在前面的章节中，我们使用了默认的参数传递模式：`by value`，这是大多数编程语言的默认模式。

在这个模型中，函数调用表达式以以下方式评估：

+   首先，所有参数都从左到右进行评估

+   然后，函数被调用，参数被引用为评估过的参数：

    ```java
    def f(x:Int) = x + x + 1
    f({ println("A "); 10 }) // A res: 21
    ```

+   有时，我们听说 Java 参数模式，其中值通过`by value`传递，引用通过`by reference`（例如，如果我们把`reference`作为一个`value`传递给对象）

## 通过名称

`by name`参数传递模式的核心是参数在函数调用之前不会被评估，而是在目标函数中每次使用参数名称时：

```java
def f(x: =>Int) = x + x + 1
f({ println("A "); 10 }) // A A res: 21
```

名称术语来自 Algol68：通过名称传递参数被描述为用参数体替换名称。这对编译器编写者来说是一个多年的挑战。

通过名称参数可以用于定义控制流表达式：

```java
def until(condition: =>Boolean)(body: =>Unit) ={
while(!condition) body
}
```

注意，构造函数参数也可以通过名称传递：

```java
class RunLater(x: =>Unit){
def run(): Unit = x
}
}
```

## 通过需要

`By need`仅在必要时评估参数一次。这可以通过`by name`调用和懒`val`来模拟：

```java
def f(x: =>Int): Int = {lazy val nx = xnx + nx + 1
}
f({ println("A "); 10 }) // A res: 21
```

我们看到`val`的懒修饰符。懒值在第一次使用时进行评估，然后作为值存储在内存中。

懒值可以是特质、类和对象的组成部分：这是定义懒初始化的常用方式。

## 创建可运行的构造

让我们创建一个可运行的构造，其语法与`Scalatest FunSuite`相同，并且`executor`将返回`true`，如果`test`参数内的代码评估没有异常。

1.  定义一个父类，其中包含将要捕获代码的变量。以下是一个可能的示例：

    ```java
    class DSLExample {val name: String = "undefined"var code: ()=>Unit = { () => () }
    }
    ```

1.  使用名称和按名称参数定义函数，该函数将填充此变量：

    ```java
    def test(testName:String)(testCode: =>Unit):Unit = {
    name = testName
     code = () => testCode }
    ```

1.  定义`executor`方法，该方法在 try/catch 块中使用命名参数。

    ```java
    def run(): Boolean = {
    try {
     code()
     true} catch {
    case ex: Exception => 
    ex.printStackTrace()
    false
    	}
    }

    ```

## 将日志参数打印到控制台和文件

让我们创建一个`log`语句，该语句将参数打印到控制台和文件，但仅当在记录器构造函数中设置名为`enabled`的参数为 true 时。

1.  使用参数和类定义`logger`。签名必须类似于以下这样：

    ```java
    class Logger(outputStream:PrintStream, dupToConsole: Boolean, enabled: Boolean) {

         …. Inset method here

    }
    ```

1.  使用按需参数定义方法，该参数仅在启用记录器时使用：

    ```java
    def log(message: => String): Unit = {
    if (enabled) {
    	val evaluatedMessage = message
    	if (dupToConsole) {
    Console.println(evaluatedMessage)
    }
    outputStream.println(evaluatedMessage)
    	}
    }

    ```

让我们创建一个`mode`命令，该命令理解`store name`定义和`remind`定义。

1.  定义一个新的对象，该对象实现了`ChatbotMode`特质，并具有数据结构（一个形成链表的密封特质）作为状态。

1.  在处理`store`时，修改状态并回答`ok.` 在处理时，`remind` – 回答。

1.  向`testcase.`添加测试

# 摘要

我们现在已经到达了本章的结尾。在本章中，我们涵盖了 Scala 的面向对象方面，如类、对象、模式匹配、自类型、案例类等。我们还实现了我们在聊天机器人应用程序中学到的面向对象概念。

在下一章中，我们将介绍 Scala 的函数式编程以及面向对象和函数式方法如何相互补充。我们还将介绍泛型类，这些类通常与模式匹配一起使用。我们还将介绍如何创建用户定义的模式匹配以及为什么它是有用的。
