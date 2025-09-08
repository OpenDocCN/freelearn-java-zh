# 第九章：使用强大的函数式构造

"我们不能用创造问题时使用的相同思维方式来解决我们的问题。”

– 阿尔伯特·爱因斯坦

当我们试图通过编写程序来解决问题时，我们的目的是编写更好的代码。更确切地说，我们指的是代码应该是可读的，并且在编译时和运行时都是高效的。可读性和效率是两个主要因素，以及其他重要的概念，如并发、异步任务等。我们可以将前两个视为我们想要的下一种特性的构建块。Scala 作为一种多范式语言，提供了多种结构来确保我们编写的代码是优化的，并在需要时提供了语法糖。许多在 *函数式编程* 中使用的函数式结构或概念使你能够编写 *更好* 的代码，不仅满足前两个要求，而且允许你的代码在 *并发* 和 *分布式* 环境中运行。

我们在本章中的目的是学习我们可以使我们的代码变得更好的方法。为此，我们将探讨一些语法结构。让我们看看本章我们将要探讨的内容：

+   对于表达式

+   模式匹配

+   选项类型

+   懒声明

+   尾调用优化

+   组合子

+   类型参数化

所有这些概念都很简单但非常有用，尤其是在你编写 Scala 代码时。其中一些我们已经讨论过，比如 *for 表达式*。我们的目的是将 for 表达式与可用的更高阶函数，如 `map`、`flatMap` 和 `withFilter` 进行比较。

# 对于表达式

如果我们说 `for` 表达式是 Scala 中的强大构造，那我们并不会错。`for` 表达式允许你遍历任何集合并执行过滤和产生新集合的操作。我们已经在 第三章，“塑造我们的 Scala 程序”中讨论过这个概念。让我们回顾一下我们看到的例子：

```java
object ForExpressions extends App { 

  val person1 = Person("Albert", 21, 'm') 
  val person2 = Person("Bob", 25, 'm') 
  val person3 = Person("Cyril", 19, 'f') 
  val persons = List(person1, person2, person3) 

  val winners = for { 
    person <- persons 
    age = person.age 
    name = person.name 
    if age > 20 
  } yield name 

  winners.foreach(println) 

} 

case class Person(name: String, age: Int, gender: Char) 
```

结果如下：

```java
Albert 
Bob 
```

在前面的例子中，我们有一个 `Person` 对象的集合。我们在这个集合上执行遍历，并基于某些条件生成一个包含所有人员名称的新集合。正如我们所知，为此我们使用了三个结构，或者说表达式：

+   生成器

    +   `person <- persons`

+   定义

    +   `age = person.age`

    +   `name = person.name`

+   过滤器

    +   `age > 20`

通过这三个表达式，我们能够以非常少的语法努力执行稍微复杂的逻辑。我们也可以用高阶函数的形式执行类似的操作。使用 `map` 和 `withFilter` 我们可以执行这样的操作，让我们来看一个例子：

```java
val winners1 = persons withFilter(_.age > 20) map(_.name)
winners1.foreach(println)

case class Person(name: String, age: Int, gender: Char) 
```

结果如下：

```java
Albert 
Bob 
```

在这里，我们使用了高阶函数来实现我们使用 for 表达式实现的相同逻辑。我们已经熟悉了集合中的 `map` 方法。它将提供一个年龄大于 20 岁的人的列表。所以现在，我们以两种不同的方式实现了相同的逻辑。首先，以 *for 表达式* 的形式，其次，以 *高阶函数* 的形式。因此，了解这是如何发生的对我们来说很重要。Scala 编译器所做的是，它将 for 表达式内部分解为高阶函数。程序员倾向于使用 for 表达式以提高可读性，但这只是一个选择问题。既然我们已经了解了内部发生的事情，我们可以开始思考 Scala 对稍微复杂的 for 表达式进行的转换，不是吗？是的，所以让我们试试看。

现在，假设我们有一个汽车品牌列表，每个品牌下有许多汽车（换句话说，每个品牌都有一个汽车列表）。代码看起来可能如下所示：

```java
case class Car(name: String, brandName: String) 
case class Brand(name: String, cars: List[Car]) 

val brands = List( 
Brand("Toyota", List(Car("Corolla", "Toyota"))), 
Brand("Honda",  List(Car("Accord", "Honda"))), 
Brand("Tesla",  List(Car("Model S", "Tesla"), 
                                      Car("Model 3", "Tesla"), 
                                      Car("Model X", "Tesla"), 
                                      Car("New Model", "Tesla")))) 
```

你可能希望为所有以 `Model` 关键字开头的特斯拉汽车生成一个对列表。你将执行如下操作：

```java
val teslaCarsStartsWithModel = for { 
  brand <- brands 
  car <- brand.cars 
  if car.name.startsWith("Model") && brand.name == "Tesla" 
} yield (brand.name, car.name) 

teslaCarsStartsWithModel foreach println 
```

结果如下：

```java
(Tesla,Model S) 
(Tesla,Model 3) 
(Tesla,Model X) 
```

我们使用了 for 表达式来完成这个任务。这个任务有两个生成器表达式，我们也在它上面执行了过滤操作。在将这些类型的 for 表达式转换为高阶函数时，Scala 使用了 `flatMap` 方法。让我们看看如何使用 `flatMap` 实现相同的功能：

```java
val teslaCarsStartsWithModel2 = brands.flatMap(brand =>  
  brand.cars withFilter(_.name.startsWith("Model") && brand.name == "Tesla") map(car => (brand.name, car.name))) 

teslaCarsStartsWithModel2 foreach println 
```

结果如下：

```java
(Tesla,Model S) 
(Tesla,Model 3) 
(Tesla,Model X) 
```

我们得到了类似的结果。所以让我们尝试将这个 `teslaCarsStartsWithModel2` 分解开来，理解我们是如何实现这个功能的。首先，我们有的如下所示：

```java
For(gen1 <- list, gen2 <- gen1.list, filter1) 
```

在有两个生成器的情况下，我们使用 `flatMap` 函数而不是 `map`。让我们一步一步地通过 for 表达式到高阶函数的转换过程：

1.  我们有以下内容：

```java
for { 
  brand <- brands 
  car <- brand.cars 
  if car.name.startsWith("Model") && brand.name == "Tesla" 
} yield (brand.name, car.name) 
```

1.  我们首先使用了 `flatMap`:

```java
brands.flatMap{ brand => 
     for{ 
       car <- brand.cars 
        if car.name.startsWith("Model") && brand.name == "Tesla" 
     } yield (brand.name, car.name) 
} 
```

1.  现在我们已经得到了品牌，我们可以访问汽车列表。我们可以继续使用以下过滤谓词：

```java
brands.flatMap{ brand => 
      brand.cars withFilter{ car =>  
         car.name.startsWith("Model") && brand.name == "Tesla" 
      } map(car => (brand.name, car.name))  
} 
```

这是我们的实现最终版本。我们在这里所做的是，我们从 `cars` 集合中过滤出元素，并最终将我们的集合转换为预期的形式。

因此，这就是 Scala 编译器将我们的 for 表达式转换为提供函数的方式。作为程序员，我们只需要处理实现部分。你可能希望将你的逻辑放在 for 表达式中，而不是编写嵌套的高阶函数，Scala 会为你完成剩下的工作。

在这里，我们详细学习了你可以以不同形式添加你的逻辑。同样，你也会找到我们必须通过并执行任何适用逻辑的案例。在这个过程中匹配不同的案例，我们可以通过匹配不同的模式来增强自己。例如，我们可能想要匹配我们的列表类型与可能的值。选项将是空列表或具有某些值的列表。Scala 不会限制你只以这两种方式匹配，但你将拥有更多匹配的选项。所有这些都是通过一个称为*模式匹配*的概念实现的。好事是，我们已经了解了模式匹配的概念，所以我们现在要做的是进一步理解它。

# 模式匹配

我们使用**模式匹配**根据案例执行代码。看看以下内容：

```java
val somelist = List(1,2,3) 

somelist match { 
  case Nil => Nil 
  case _ => ??? 
} 
```

通过查看我们的模式匹配表达式结构，我们可以看到一些事情。首先，我们执行一个匹配某个值，然后跟随着`match`关键字，然后放置案例。对于每个案例，我们指定一些模式。现在，模式可以是一个常量值、一个变量，甚至是一个构造函数。我们很快就会查看所有这些模式。模式匹配还允许我们以条件的形式在我们的匹配中放置守卫。在这种情况下，只有当条件适用时，模式才会匹配。如果你看一下之前的关于`somelist`的玩具示例，你会看到一个*`*_*`下划线。它被称为**通配符模式**。它将匹配所有值或模式与案例。从逻辑上讲，你不能在通配符之后放置另一个案例。例如，以下没有意义，并会抛出一个警告：

*```java
val somelist = 1 :: 2 :: 3 :: Nil 

val x = somelist match { 
  case Nil => Nil 
  case _ => println("anything") 
  case head :: tail => println("something with a head and a tail") 
} 
Warning:(21, 10) patterns after a variable pattern cannot match (SLS 8.1.1) 
    case _ => println("anything") 
Warning:(22, 33) unreachable code due to variable pattern on line 21 
    case head :: tail => println("something with a head and a tail") 
Warning:(22, 33) unreachable code 
    case head :: tail => println("something with a head and a tail") 
```

这是一个 Scala 中模式匹配的相当基础的例子。我们还有更多可以模式匹配的方式。为什么不看看所有这些方式呢？

# 我们可以以不同的方式模式匹配

Scala 中的模式匹配证明是一个非常重要的概念。我们可以对变量、常量甚至构造函数进行匹配。我们将逐一查看它们。让我们从对变量的匹配开始。

# 匹配变量

有时，当我们需要在模式匹配成功后使用值时，我们想要对带有变量的案例进行匹配。这样做的作用是将值分配给变量，然后我们可以在我们的代码中为特定案例使用它。如果我们看一下以下示例，会更好：

```java
import scala.util.control.NonFatal

def safeToInt(canBeNumber: String): Option[Int] = { 
  try { 
    Some(canBeNumber.toInt) 
  } catch { 
    case NonFatal(e) => None 
  } 
} 

safeToInt("10") match { 
  case None => println("Got nothing") 
  case someValue =>  println(s"Got ${someValue.get}") 
} 
```

结果如下：

```java
Got 10 
```

这里，我们定义了一个方法，该方法尝试将字符串表示的数字转换为整数。然后，我们用参数调用该方法，并尝试使用名为`someValue`的变量进行匹配。这个`someValue`变量将与匹配的值的类型相同。

# 匹配常量

我们还可以对常量进行案例匹配，例如基本的 switch 语句。看看以下内容：

```java
def matchAgainst(i: Int) = i match { 
  case 1 => println("One") 
  case 2 => println("Two") 
  case 3 => println("Three") 
  case 4 => println("Four") 
  case _ => println("Not in Range 1 to 4") 
} 

matchAgainst(1)
 matchAgainst(5)
```

结果如下：

```java
One 
Not in Range 1 to 4 
```

在这里，我们直接将我们的表达式与常量值进行匹配。这可以是任何值，具体取决于你的方法接受的数据类型。你可以匹配布尔值、字符串或任何其他常量值。

# 匹配构造函数

好的，构造函数模式看起来是什么样子？它涉及到将构造函数与一个值进行匹配，或者说，提取我们选择的价值。让我们来看一个例子：

```java
def safeToInt(canBeNumber: String): Option[Int] = { 
  try { 
    Some(canBeNumber.toInt) 
  } catch { 
    case NonFatal(e) => None 
  } 
} 

safeToInt("10") match { 
  case None => println("Got nothing") 
  case Some(value) =>  println(s"Got $value") 
} 
```

结果如下：

```java
Got 10 
```

我们能看到的唯一区别是，我们不是提供一个变量，而是给出了一个构造函数模式。`Some(value)`让你从自身中提取`value`。在这个给定的例子中，`safeToInt`方法返回一个`Option`类型。我们将在后续章节中学习类型。现在，对我们来说有趣的信息是我们有`Option`类型的两个子类型，分别命名为`Some`和`None`。正如名称所暗示的，`Some`表示有值，而`None`表示没有值。`Some`子类型期望一个特定的值作为其构造函数参数。因此，我们可以对它进行匹配。下面的行正是我们刚才提到的：

```java
case Some(value) => println(s"Got $value") 
```

通过这个声明，我们可以提取一个值，在我们的例子中，提取的参数名称也是`value`，因此我们使用了它。这是一个使用构造函数进行模式匹配的例子。我们已经了解了 Scala 中的`case`类，并且提到`case`类为我们提供了一个精确的结构，通过这个结构我们可以直接进行模式匹配。所以让我们来看一个例子：

```java
 trait Employee 
 case class ContractEmp(id: String, name: String) extends Employee 
 case class Developer(id: String, name: String) extends Employee 
 case class Consultant(id: String, name: String) extends Employee 

/* 
 * Process joining bonus if 
 *     :> Developer has ID Starting from "DL"  JB: 1L 
 *     :> Consultant has  ID Starting from "CNL":  1L 
 */ 
 def processJoiningBonus(employee: Employee, amountCTC: Double) = employee match { 
   case ContractEmp(id, _) => amountCTC 
   case Developer(id, _) => if(id.startsWith("DL")) amountCTC + 10000.0 else amountCTC 
   case Consultant(id, _) => if(id.startsWith("CNL")) amountCTC + 10000.0 else amountCTC 
 } 

 val developerEmplEligibleForJB = Developer("DL0001", "Alex") 
 val consultantEmpEligibleForJB = Consultant("CNL0001","Henry") 
 val developer = Developer("DI0002", "Heith") 

 println(processJoiningBonus(developerEmplEligibleForJB, 55000)) 
 println(processJoiningBonus(consultantEmpEligibleForJB, 65000)) 
 println(processJoiningBonus(developer, 66000)) 
```

结果如下：

```java
65000.0 
75000.0 
66000.0 
```

在这个例子中，我们定义了三种员工类别：`Developer`、`Consultant`和`ContractEmp`。我们有一个问题要解决：我们必须根据某些条件处理特定类别中特定员工的入职奖金。整个逻辑在`case`类和模式匹配方面非常容易实现，这正是我们在这里所做的事情。看看以下来自前一个解决方案的行：

```java
case Developer(id, _) => if(id.startsWith("DL")) amountCTC + 10000.0 else amountCTC 
```

在这里，我们与`case`类构造函数进行匹配。我们对所需的参数给出了一些名称，其他则用通配符`_`（下划线）替换。在这里，我们必须对`id`参数设置一个条件，因此在相应的`case`类构造函数中提到了它。你可以看到`case`类和模式匹配如何使一个稍微复杂的领域问题变得非常容易解决。嗯，这还没有结束，还有更多。我们还可以在我们的`case`表达式上设置守卫。让我们看看带有守卫的相同例子：

```java
/* 
 * Process joining bonus if 
 *     :> Developer has ID Starting from "DL"  JB: 1L 
 *     :> Consultant has  ID Starting from "CNL":  1L 
 */ 
 def processJoiningBonus(employee: Employee, amountCTC: Double) = employee match { 
   case ContractEmp(id, _) => amountCTC 
   case Developer(id, _) if id.startsWith("DL") => amountCTC + 10000.0 
   case Consultant(id, _) if id.startsWith("CNL") =>  amountCTC + 10000.0 
   case _ => amountCTC 
 } 
```

结果如下：

```java
65000.0 
75000.0 
66000.0 
```

如果我们看看以下表达式，我们可以看到在我们的`case`模式上有守卫。所以值只有在守卫允许的情况下才会匹配：

```java
case Developer(id, _) if id.startsWith("DL") => amountCTC + 10000.0 
```

因此，在执行块的右侧之前，这个表达式检查`id`是否以`"DL"`开头，并根据这个结果进行匹配。这就是我们可以直接使用构造器提取参数并使用它们的方式。你还可以以更多的方式使用模式。例如，我们可以对序列或元组执行匹配。当我们必须匹配一些嵌套表达式，或者匹配一个包含另一个`case`类的`case`类时，这也是可能的。为了使我们的代码更有意义，并且为了可读性，我们可以使用`@`符号绑定嵌套`case`类并执行模式匹配。让我们举一个例子：

```java
case class Car(name: String, brand: CarBrand) 
case class CarBrand(name: String) 

val car = Car("Model X", CarBrand("Tesla")) 
val anyCar = Car("Model XYZ", CarBrand("XYZ")) 

def matchCar(c: Car) = c match { 
  case Car(_, brand @ CarBrand("Tesla")) => println("It's a Tesla Car!") 
  case _ => println("It's just a Carrr!!") 
} 

matchCar(car) 
matchCar(anyCar) 
```

结果如下：

```java
It's a Tesla Car! 
It's just a Carrr!! 
```

上述示例是自我解释的。我们在`Car`内部有一个嵌套的`case`类，名为`CarBrand`，并对其进行了模式匹配。我们使用`@`符号访问了该特定对象。所以，这些都是我们可以使用模式匹配轻松执行所有这些任务的几种方法。到现在，你一定对模式匹配有了概念；它有多么强大和重要。

在执行所有这些模式匹配的过程中，我们感觉到有一些反例，我们不想执行匹配，并使用通配符，这样我们就可以提供任何返回值。可能在这种情况下没有预期的值，我们只想让我们的代码有意义，同时返回一个有意义的响应。在这些情况下，我们可以使用我们的`Option`类型。正如其名所示，当你将类型定义为`Option`时，你可能得到一些值或没有值。为了使其更清楚，让我们回顾一下我们的`safeToInt`函数：

```java
def safeToInt(canBeNumber: String): Option[Int] = { 
  try { 
    Some(canBeNumber.toInt) 
  } catch { 
    case NonFatal(e) => None 
  } 
} 

safeToInt("10") match { 
  case None => println("Got nothing") 
  case Some(value) =>  println(s"Got $value") 
} 
```

在这里，在我们的`safeToInt`函数的定义中，我们定义了我们的响应类型为`Option`，仅仅因为我们知道它可能或可能不会响应一个有意义的值。现在使用`Option`而不是直接使用任何类型的理由是清晰的，让我们讨论`Option`类型。

# 选项类型

选项是 Scala 提供的一种类型构造器。问题随之而来，什么是类型构造器？答案是简单的；它允许你构造一个类型。我们将考虑两个陈述：

1.  `Option`是一种类型构造器

1.  `Option[Int]`是一种类型

让我们详细讨论这些。当我说`Foo`是一个类型构造器时，我的意思是`Foo`期望你以参数的形式提供一个特定的类型。它看起来像`Foo[T]`，其中`T`是一个实际类型。我们称它们为**类型参数**，我们将在接下来的几节中讨论它们。

在第二个陈述中，我们看到了我们给我们的`Option`类型构造器括号中一个`Int`类型，并形成了一个类型。如果你在 Scala REPL 中尝试这样做，它会告诉你我们讨论的完全相同的事情：

```java
scala> val a: Option = Some(1) 
<console>:11: error: class Option takes type parameters 
       val a: Option = Some(1) 

scala> val a: Option[Int] = Some(1) 
a: Option[Int] = Some(1) 
```

简单来说，`Option[T]`类型表示任何给定类型`T`的可选值。现在`T`可以是您传递的任何类型，在上一个例子中它是`Int`。`Option[T]`类型有两个子类型：

+   `Some(T)`

+   `None`

当有值可用时，我们会得到`Some`值，否则得到`None`。`Option`类型还为你提供了一个`map`方法。你想要使用选项值的方式是调用`map`方法：

```java
scala> a map println 
1 
```

在这里发生的情况是，`map`方法会给你相应的值，如果它可用的话。否则，如果可选值是`None`，它将不会做任何事情。通常，你可能会想将这种类型用作异常处理机制。我们如何做到这一点？我们已经看到了一个例子。回想一下我们的`safeToInt`方法，如果没有`Option`，它可能看起来像这样（也许）：

```java
def safeToInt(canBeNumber: String): Int = { 
  try { 
     canBeNumber.toInt 
  } catch { 
    case NonFatal(e) => throw Exception 
  } 
} 
```

但是，如果你看一下签名，声明告诉你函数将返回一个`Int`，但实际上函数可能会抛出一个`Exception`。这既不是预期的，也不正确。函数应该遵循其自己的声明。因此，我们可以使用我们的`Option`类型，作为救星，做我们声明的事情。`Option`是函数式编程为你提供的构造之一。

这些类型还有很多，它们为你提供了一些现成的构造。其中一些是类型，如`Either`、`Try`以及一些其他的。你可以参考 Scala API 文档（[`www.scala-lang.org/api/2.12.3/scala/util/Either.html`](http://www.scala-lang.org/api/2.12.3/scala/util/Either.html)）来获取更多关于这些类型的信息。

接下来，我们将讨论另一个功能构造。它不仅仅是一个构造，它是一种评估方案。是的，我们正在谈论*懒加载*。Scala 允许你以多种方式使用这种方案。让我们谈谈`lazy`关键字。

# 懒声明

在学习更多关于`lazy`关键字或懒加载之前，让我们先谈谈为什么我们需要它以及它究竟是什么。懒加载的好处可以用几行或几页文字来解释，但为了我们的理解，让我们用一个简单的句子来说明。

**懒加载**让你能够以评估顺序无关的方式编写代码。它还通过只评估所需的表达式来为你节省一些时间。它就像代码中存在许多复杂的评估，但由于某种原因从未被评估。最后一行之所以可能，是因为懒加载的概念。在 Scala 中，你可以声明一个值为`lazy`。让我们举一个例子。在 Scala REPL 中尝试以下操作：

```java
scala> lazy val v = 1 
v: Int = <lazy> 

scala> val z = 1 
z: Int = 1 
```

在这里，当我们将值`1`赋给我们的`val v`时，REPL 给了我们`Int`类型和值`<lazy>`，而对于`val z`，我们得到了`1`。为什么会发生这种情况是因为`lazy`声明。在 Scala 中，当你声明一个值为懒时，编译器只有在第一次使用它时才会评估这个值。有了这个，你就无需担心将`val`声明放在任何顺序。每个`lazy`值在其需要时才会被评估。

当我们在讨论优化我们的代码时，让我们看看另一个概念，*尾递归优化*。我们首次介绍*尾递归优化*是在第三章，“塑造我们的 Scala 程序”中，讨论*递归*时。让我们简要地谈谈它。

# 尾递归优化

我们熟悉递归带来的局限性。我们知道，如果函数调用不是尾递归，每次函数调用都会创建一个新的栈帧。对于必须处理大量函数调用的场景，这可能会导致栈溢出，这是我们不希望的。因此，在这种情况下建议的是，将递归函数调用作为你函数定义中的最后一个语句，然后 Scala 编译器会为你完成剩下的工作。看看下面的例子：

```java
import scala.annotation.tailrec

object TailRecursion { 
  def main(args: Array[String]): Unit = { 
      val list = List("Alex", "Bob", "Chris", "David", "Raven", "Stuart") 
    someRecursiveMethod(list) 

  } 

  /* 
      You have a sorted list of names of employees, within a company. 
      print all names until the name "Raven" comes 
  */ 
  @tailrec 
  def someRecursiveMethod(list: List[String]): Unit = { 
      list match { 
        case Nil => println("Can't continue. Either printed all names or encountered Raven") 
        case head :: tail => if(head != "Raven") { 
          println(s"Name: $head") 
          someRecursiveMethod(tail) 
        } else someRecursiveMethod(Nil) 
      }
   }
 }
```

结果如下：

```java
Name: Alex 
Name: Bob 
Name: Chris 
Name: David 
Can't continue. Either printed all names or encountered Raven 
```

在前面的例子中，如果你仔细观察，你会发现我们每次进行递归调用时，它都是该特定作用域中的最后一个语句。这意味着我们对`someRecursiveMethod`的调用是最后一个调用，之后没有其他调用。如果不是这样，Scala 编译器会通过一条消息提醒你，说递归调用不在尾递归位置：

```java
@tailrec 
def someRecursiveMethod(list: List[String]): Unit = { 
    list match { 
      case Nil => println(s"Can't continue. Either printed all names or encountered Raven") 
      case head :: tail => if(head != "Raven") { 
        println(s"Name: $head") 
        someRecursiveMethod(tail) 
        println("Won't happen") 
      } else someRecursiveMethod(Nil) 

    } 
} 
Error:(21, 30) could not optimize @tailrec annotated method someRecursiveMethod: it contains a recursive call not in tail position 
someRecursiveMethod(tail) 
```

另外，需要注意的一点是，我们通过提供`tailrec`注解来帮助 Scala 编译器。当我们提供这个注解时，编译器会将你的函数视为尾递归函数。这种函数的评估不会在每个调用时创建新的栈帧，而是使用已经创建的栈帧。这就是我们避免栈溢出并使用递归的方式。在我们的例子中，我们尝试匹配一个名字列表。是的，你已经熟悉模式匹配的概念。如果不是`Nil`，我们会在列表中检查名字`Raven`，然后停止进一步的调用。如果你在 Scala 中缺少`break`或`continue`语句，这是你可以实现它们的方式：通过递归和检查条件。

所以这就是关于尾递归优化的一切。我们还看到，当提供注解时，Scala 编译器会帮助我们。嗯，递归可以帮助你避免可变性，同时实现复杂的逻辑，这增加了已经强大的函数式编程。由于我们正在学习函数式编程结构，了解它们根植于*永恒的数学*是非常重要的。数学创造了函数式编程的概念，几乎所有函数式编程的概念都来自某些数学证明或概念。其中之一是组合子。了解组合子或理解它们如何与数学相关超出了本书的范围，但我们将简要介绍并查看一个简单的例子。这将会很有趣。让我们通过*组合子*来了解。

# 组合子

维基百科关于**组合子**的描述如下：

“组合子是一个高阶函数，它只使用函数应用和先前定义的组合子来从其参数定义一个结果。”

除了这个定义，我们还可以说，组合子是一个封闭的 *lambda* 表达式。我们已经在前几个地方看到了 lambda 的应用，并且对它们进行了定义。lambda 仅仅是对任何函数的匿名定义。例如，当你将一个表达式传递给我们的`foreach`方法时，你以 lambda 表达式的形式传递它。看看下面的例子：

```java
val brands = List(Brand("Toyota", List(Car("Corolla", "Toyota"))), 
                  Brand("Honda", List(Car("Accord", "Honda"))), 
                  Brand("Tesla", List(Car("Model S", "Tesla"), 
                                      Car("Model 3", "Tesla"), 
                                      Car("Model X", "Tesla"), 
                                      Car("New Model", "Tesla")))) 

brands.foreach((b: Brand) => { 
  //Take the brand name, and check the number of Cars and print them.
val brandName = b.name 
  println(s"Brand: $brandName || Total Cars:${b.cars.length}") 
  (brandName, b.cars) 
}) 
```

在这里，`foreach` 方法接受一个 lambda 表达式并执行它。更准确地说，括号内的 `(b: Brand)` 是 lambda 的一个例子。现在，让我们提出一些问题。lambda 与组合子有什么关系？或者让我们问，工作（函数式）程序员对组合子的定义是什么？好吧，为了回答这些问题，我们将使用第一个维基百科的定义。如果你仔细观察，有几个需要注意的地方。首先，它是一个高阶函数，其次，它是一个封闭的 lambda。封闭意味着它不包含任何 *自由* 变量。对于那些想知道什么是自由变量的你，请看以下 lambda：

```java
((x) => x * y) 
```

在这里，`y` 是一个自由变量。我们之前已经见过这类没有自由变量的高阶函数：我们的 `map` 和 `filter` 函数。这些被称为组合子。在函数式编程中，我们倾向于大量使用这些。你也可以将这些用作对现有数据的转换。如果我们将这些组合子组合起来，它们对于形成数据流逻辑非常有帮助。这意味着，如果你将 `map`、`filter` 和 `fold` 作为组合子一起使用，你可以从你的程序中创建出领域逻辑。这种编写程序的方式在函数式编程中经常被使用。当你查看编写好的库时，你会找到更多这样的例子。各种组合子被用于各种集合和其他数据结构。使用这些组合子的原因在于它们提供了一种抽象的，或者说，一种通用的方法来执行某些操作。因此，它们被广泛应用于各个领域。实现一些类似以下逻辑的代码既容易又有趣：

```java
creditCards.filter(_.limit < 55000)
               .map(cc => cc.accounts(cc.holder))            .filter(_.isLinkedAccount) 
  .get 
  .info 
```

这确实是一个强大的构造，在函数式编程中被广泛使用。

现在你已经了解了组合子，并且也意识到了高阶函数，你就有能力解决编程问题。那么接下来呢？现在是时候迈出下一步了。我们将深入 Scala 编程的抽象（海洋）层面。如果我们能使我们的解决方案抽象化，这意味着我们提供的解决方案应该满足不止一个问题陈述。让我们从学习关于 t*ype parameterization* 开始。

# 类型参数化

对于**类型参数化**的介绍，我们将参考我们已经看到的两个例子，试图弄清楚它的意义。我知道你很感兴趣地跟随着这一章，你已经阅读了我们讨论的例子和概念，所以让我们做一个练习。想想我们的救星`Option[T]`类型，并尝试思考你为什么想向`Option`传递一个类型（因为它要求`T`是一个类型）。它可以有什么用途？

我认为你提出了一些想法。也许你认为通过传递我们选择的一种类型，我们可以让我们的代码使用`Option`类型在多个场景下工作。如果你这样想，太好了！让我们称它为我们的解决方案的泛化。而且，让我们称这种方法为编程的泛型方法。它看起来怎么样？让我们看看下面的代码：

```java
object TypeParameterization { 

  def main(args: Array[String]): Unit = { 
      val mayBeAnInteger = Some("101") 
      val mayBeADouble = Some("101.0") 
      val mayBeTrue = Some("true") 

    println(s"Calling mapToInt: ${mapToInt(mayBeAnInteger, (x: String) => x.toInt)}") 
    println(s"Calling mapToDouble: ${mapToDouble(mayBeADouble, (x: String) => x.toDouble)}") 
    println(s"Calling mapToBoolean: ${mapToBoolean(mayBeTrue, (x: String) => x.toBoolean)}") 
  } 
  def mapToInt(mayBeInt: Option[String], function: String => Int) = function(mayBeInt.get) 

  def mapToDouble(mayBeDouble: Option[String], function: String => Double) = function(mayBeDouble.get) 

  def mapToBoolean(mayBeBoolean: Option[String], function: String => Boolean) = function(mayBeBoolean.get) 
} 
```

结果如下：

```java
Calling mapToInt: 101 
Calling mapToDouble: 101.0 
Calling mapToBoolean: true 
```

所以正如代码所暗示的，我们有一些可选的字符串，可以是`Int`、`String`或`Boolean`类型*.* 我们的意图是将它们转换为它们各自类型。因此，我们形成了一些函数，它们接受可选的字符串，然后我们将它们转换为它们各自类型，所以为此我们传递了一个函数字面量。如果我们不能想到几个反例，那么这意味着它正在工作。然而，这个解决方案很庞大；它感觉不太好。此外，我们可以看到代码中存在一些重复性。我们在几乎相同的操作中使用了`mapToXXX`，其中我们将`XXX`视为任何类型。看起来我们可以泛化这个解决方案。让我们考虑一下，我们如何做到这一点？

我们如何告诉方法我们将提供的类型？一个解决方案是将类型作为参数传递给方法，然后在声明中使用它们。让我们尝试这个解决方案，看看代码会是什么样子：

```java
object TypeParameterization { 

  def main(args: Array[String]): Unit = { 
      val mayBeAnInteger = Some("101") 
      val mayBeADouble = Some("101.0") 
      val mayBeTrue = Some("true") 

    println(s"Calling mapToValue: ${mapToValue(mayBeAnInteger, x => x.toInt)}") 
    println(s"Calling mapToValue: ${mapToValue(mayBeADouble, x => x.toDouble)}") 
    println(s"Calling mapToValue: ${mapToValue(mayBeTrue, x => x.toBoolean)}") 
  } 

  def mapToValueT = function(mayBeValue.get) 
} 
```

结果如下：

```java
Calling mapToValue: 101 
Calling mapToValue: 101.0 
Calling mapToValue: true 
```

泛化之后，我们只需一个函数就能执行相同的逻辑。所以，让我们看看变化以及我们如何给出类型参数：

```java
def mapToValueT : T = function(mayBeValue.get) 
```

在前面的`mapToValue`函数中，在给出函数名，即`mapToValue`之后，我们在花括号中给出了`T`作为类型参数。有了这个，我们就得到了在函数声明和定义中使用这个类型参数的许可。因此，我们将其用作函数字面量中的类型以及返回类型。根据使用情况，我们可以给出任意数量的类型参数。例如，如果你想让它更通用，函数可能看起来像以下这样：

```java
def mapToValueA, B : B = function(mayBeValue.get) 
```

在这个定义中，我们使用了两个类型参数`A`和`B`，因此使我们的方法更加通用。如果你看看`Option[T]`中的`map`方法，它看起来如下：

```java
def mapB: Option[B] = if (isEmpty) 
     None 
 else 
     Some(f(this.get)) 
```

在这里，我们在`map`函数的定义中使用了类型参数。根据这一点，对于`Option[A]`，我们有一个从`A`类型到`B`类型的函数`map`方法。所以当你从这个`map`方法中给出调用时，编译器会根据上下文推断出`A`和`B`类型。

这只是一个关于类型参数化的简介。在下一章中，我们将看到更多关于它的内容，以及一些高级概念。有了这些，我们可以结束本章，让我们总结一下我们学到了什么。

# 概述

本章让我们了解了如何以不同的实现风格实现逻辑。我们讨论了 for 表达式及其转换为高阶函数。我们看到*模式匹配*如何使复杂的逻辑看起来非常简单。我们还讨论了诸如`Option`和`lazy`关键字这样的结构。这些使我们能够编写有意义的和优化的代码。然后我们讨论了*尾调用优化*。我们面对了*组合子*，最后我们得到了关于*类型参数化*的介绍。

在下一章中，我们将从本章结束的地方开始。我们将更多地讨论类型、参数化类型和变体关系，让我告诉你，那将会很有趣*。
