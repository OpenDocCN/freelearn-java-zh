# 第四章：使用函数为程序赋予意义

"面向对象编程通过封装移动部分使代码易于理解；函数式编程通过最小化移动部分使代码易于理解。"

- 迈克尔·费思

作为一种混合范式语言，Scala 鼓励你重用你编写的代码，同时期望你遵循函数式编程背后的动机。这个动机是，你的程序应该被分解成更小的抽象来完成一个定义良好的任务。这可以通过使用函数来实现。函数不过是一个执行特定任务并可以按需重用的逻辑构造。

有一些方法可以将这些函数引入并应用到我们的程序中。有许多原因可以在我们的程序中使用函数。编写得好的函数，具有确切数量的所需参数，具有明确定义的作用域和隐私性，可以使你的代码看起来很好。此外，这些函数给你的程序赋予了意义。根据我们的需求，可以采用一些语法变化来采用特定的评估策略。例如，只有在需要时才评估函数，或者惰性评估，意味着表达式将在首次访问时进行评估。让我们准备好了解 Scala 中的函数。在本章中，我们将熟悉以下内容：

+   函数语法

+   调用函数

+   函数字面量/匿名函数

+   评估策略

+   部分函数

因此，让我们从编写函数所需的内容开始我们的讨论。

# 函数语法

Scala 中的函数可以使用`def`关键字来编写，后面跟函数名，并给函数提供一些作为输入的参数。让我们看看函数的通用语法：

```java
modifiers... 
def function_name(arg1: arg1_type, arg2: arg2_type,...): return_type = ???
```

上述语法显示了 Scala 中通用的函数签名。首先，我们给出函数的修饰符。修饰符可以理解为为函数定义的属性。修饰符有多种形式。其中一些如下：

+   注解

+   覆盖修饰符

+   访问修饰符（`private`等）

+   `final`关键字

建议的做法是按需且按照给定顺序使用前面的修饰符。在指定修饰符后，我们使用`def`关键字来表示函数，然后跟函数名。在给出函数名之后，我们指定参数。参数在括号中指定：首先，参数名然后是其类型。这与 Java 有所不同。在 Java 中，这个顺序正好相反。最后，我们指定函数的返回类型。我们也可以省略返回类型，因为 Scala 可以推断它。除了某些特殊情况外，它将正常工作。为了提高我们程序的可读性，我们可以将返回类型作为函数签名的一部分。在声明函数之后，我们可以给出定义体。让我们看看一些具体的例子：

```java
def compareIntegers(value1: Int, value2: Int): Int = if (value1 == value2) 0 else if (value1 > value2) 1 else -1
```

上述示例简单地定义了一个期望两个整数值、比较它们并返回整数响应的函数。定义也很简单，我们正在检查输入的相等性。如果值相等，则返回`0`；如果第一个值大于第二个值，则返回`1`，否则返回`-1`。在这里，我们没有为我们的函数使用任何修饰符。默认情况下，Scala 将其函数视为`public`，这意味着你可以从任何其他类访问它们并覆盖它们。

如果你仔细观察函数体，它是行内的。我们直接定义了函数，这样做有两个原因：

+   为了使我们的代码简单易读

+   定义足够小，可以放在一行中

定义函数定义行内的一个推荐做法是，当你的函数签名及其定义大约有 30 个字符时。如果它更长但仍然简洁，我们可以在下一行开始定义，如下所示：

```java
def compareIntegersV1(value1: Int, value2: Int): Int = 
  if (value1 == value2) 0 else if (value1 > value2) 1 else -1 
```

所以选择权在你；为了使你的代码更易读，你可以选择在行内定义函数。如果函数体中有多行，你可以选择将它们封装在一对大括号内：

```java
def compareIntegersV2(value1: Int, value2: Int): Int = { 
  println(s" Executing V2") 
  if (value1 == value2) 0 else if (value1 > value2) 1 else -1 
} 
```

在这里，函数定义中有两个语句。第一个是打印，后者是评估比较。因此，我们使用一对大括号将它们封装起来。让我们看看整个程序：

```java
object FunctionSyntax extends App{ 
 /* 
  * Function compare two Integer numbers 
  * @param value1 Int 
  * @param value2 Int 
  * return Int 
  * 1  if value1 > value2 
  * 0  if value1 = value2 
  * -1 if value1 < value2 
  */ 
  def compareIntegers(value1: Int, value2: Int): Int = if (value1 == value2) 0 else if (value1 > value2) 1 else -1 

  def compareIntegersV1(value1: Int, value2: Int): Int = {
    if (value1 == value2) 0 else if (value1 > value2) 1 else -1 
   }

  def compareIntegersV2(value1: Int, value2: Int): Int =
    if (value1 == value2) 0 else if (value1 > value2) 1 else -1 

  println(compareIntegers(1, 2)) 
  println(compareIntegersV1(2, 1)) 
  println(compareIntegersV2(2, 2)) 

} 
```

以下结果是：

```java
-1
1 
0 
```

当我们定义一个函数体时，最后一个表达式作为函数的返回类型。在我们的例子中，`if else`表达式的评估，即一个整数，将是`compareIntegers`函数的返回类型。

# 函数嵌套

无论何时我们有封装逻辑的可能性，我们都将代码片段转换为一个函数。如果我们这样做很多次，可能会污染我们的代码。此外，当我们将函数分解成更小的辅助单元时，我们倾向于给出几乎相同的名字。让我们举一个例子：

```java
object FunctionSyntaxOne extends App { 

  def compareIntegersV4(value1: Int, value2: Int): String = { 
 println("Executing V4") 
    val result = if (value1 == value2) 0 else if (value1 > value2) 1 else -1 
    giveAMeaningFullResult(result, value1, value2) 
  } 

  private def giveAMeaningFullResult(result: Int, value1: Int, value2: Int) = result match { 
    case 0 => "Values are equal" 
    case -1 => s"$value1 is smaller than $value2" 
    case 1 => s"$value1 is greater than $value2" 
    case _ => "Could not perform the operation" 
  } 

  println(compareIntegersV4(2,1)) 
} 
```

以下结果是：

```java
Executing V4 
2 is greater than 1 
```

在前面的程序中，我们定义了`compareIntegersV4`函数，在该函数中，在评估两个整数的比较之后，我们调用了一个名为`giveAMeaningFullResult`的辅助函数，传递了一个结果和两个值。这个函数根据结果返回一个有意义的字符串。代码运行正常，但如果你仔细思考，可能会发现这个私有方法只对`compareIntegersV4`有意义，因此最好将`giveAMeaningFullResult`的定义放在函数内部。让我们重构我们的代码，以嵌套方式在`compareIntegersV5`内部定义辅助函数：

```java
object FunctionSyntaxTwo extends App { 

  def compareIntegersV5(value1: Int, value2: Int): String = { 
 println("Executing V5") 

    def giveAMeaningFullResult(result: Int) = result match { 
      case 0 => "Values are equal" 
      case -1 => s"$value1 is smaller than $value2" 
      case 1 => s"$value1 is greater than $value2" 
      case _ => "Could not perform the operation" 
    } 

    val result = if (value1 == value2) 0 else if (value1 > value2) 1 else -1 
    giveAMeaningFullResult(result) 
  } 

  println(compareIntegersV5(2,1)) 
} 
```

以下结果是：

```java
Executing V5 
2 is greater than 1 
```

如你在上述代码中所见，我们定义了一个嵌套函数`giveAMeaningFullResult`*，*并且也做了一些修改。现在它只期望接收一个整数类型的参数，并返回一个有意义的字符串。我们可以访问外部函数的所有变量；这就是为什么我们没有将`value1`和`value2`传递给我们的嵌套辅助函数*.* 这使得我们的代码看起来更简洁。我们能够直接传递参数调用我们的函数，在我们的例子中是`2`和`1`。我们可以以多种方式调用函数；我们为什么不看看这些方式呢？

# 调用函数

我们可以调用一个函数来执行我们为其定义的任务。在调用时，我们传递函数作为输入参数的参数。这可以通过多种方式实现：我们可以指定可变数量的参数，我们可以指定参数的名称，或者我们可以指定一个默认值，以防在调用函数时未传递参数。让我们考虑一个场景，我们不确定要传递给函数进行评估的参数数量，但我们确定它的类型。

# 传递可变数量的参数

如果你还记得，我们在上一章已经看到了一个关于接受可变数量参数并在其上执行操作的函数的例子：

```java
 /* 
  * Prints pages with given Indexes for doc 
  */ 
  def printPages(doc: Document, indexes: Int*) = for(index <- indexes if index <= doc.numOfPages) print(index) 
```

我们的方法接受索引数字，并打印出作为第一个参数传递的文档中的那些页面。在这里，参数`indexes`被称为**可变参数***.* 这表示我们可以传递任何指定类型的参数数量；在这种情况下，我们指定了`Int`*.* 在调用此函数时，我们可以传递任何类型的`Int`*.* 我们已经尝试过了。现在，让我们考虑一个期望接收多个整数并返回所有数字平均值的数学函数。它应该是什么样子呢？

它可能是一个带有`def`关键字、名称和参数的签名，或者只是一个*可变参数*：

```java
def average(numbers: Int*): Double = ??? 
```

上述代码是`average`函数的签名。函数的主体尚未定义：

```java
object FunctionCalls extends App { 

  def average(numbers: Int*) : Double = numbers.foldLeft(0)((a, c) => a + c) / numbers.length 

  def averageV1(numbers: Int*) : Double = numbers.sum / numbers.length 

  println(average(2,2)) 
  println(average(1,2,3)) 
  println(averageV1(1,2,3)) 

} 
```

以下就是结果：

```java
2.0 
2.0 
2.0 
```

让我们看看第一个 `average` 函数；它期望一个类型为 `Int`* 的可变参数。它已经用参数 `2` 和 `2` 被调用。这里参数的数量是 2。我们可以提供任意数量的参数来执行操作。我们函数的定义使用了 `fold` 操作来对所有传递的数字进行求和。我们将在下一章讨论我们的集合函数时看到 `fold` 的工作细节。现在，只需理解它遍历集合中的每个元素并与提供的参数执行操作即可，在我们的例子中，即 `0`。我们用不同数量的参数调用了函数。同样，我们可以定义我们的函数以支持任意类型的可变数量参数。我们可以相应地调用函数。唯一的要求是 *可变参数* 参数应该在函数签名参数列表的末尾：

```java
def averageV1(numbers: Int*, wrongArgument: Int): Double = numbers.sum / numbers.length 
```

这意味着 `numbers`*，* 即一个可变参数，应该放在最后声明，之后声明 `wrongArgument` 将会导致 *编译时错误*。

# 使用默认参数值调用函数

我们可以在声明函数时提供默认参数值。如果我们这样做，在调用函数时可以避免为该参数传递参数。让我们通过一个例子看看这是如何工作的。我们已经看到了这个例子，我们将比较两个整数。让我们给第二个参数一个默认值 `10`：

```java
def compareIntegersV6(value1: Int, value2: Int = 10): String = { 
 println("Executing V6") 

  def giveAMeaningFullResult(result: Int) = result match { 
    case 0 => "Values are equal" 
    case -1 => s"$value1 is smaller than $value2" 
    case 1 => s"$value1 is greater than $value2" 
    case _ => "Could not perform the operation" 
  } 

  val result = if (value1 == value2) 0 else if (value1 > value2) 1 else -1 
  giveAMeaningFullResult(result) 
} 

println(compareIntegersV6(12)) 
```

以下结果是：

```java
Executing V6 
12 is greater than 10 
```

在这里，当我们声明 `compareIntegersV6` 函数时，我们给参数 `value2`* 提供了一个默认值 `10`。在调用函数的末尾，我们只传递了一个参数：

```java
compareIntegersV6(12) 
```

在调用函数时，我们只传递了一个参数 `12`，这是 `value1`* 的值。在这些情况下，Scala 编译器会寻找绑定到其他参数的值。在我们的例子中，编译器能够推断出对于其他参数，默认值已经是 10，所以函数应用将基于这两个值进行评估。提供默认值并使用它们仅在 Scala 编译器能够推断值的情况下才有效。在存在歧义的情况下，它不允许你调用函数。让我们举一个例子：

```java
def compareIntegersV6(value1: Int = 10, value2: Int) = ??? 
```

对于这个函数，让我们尝试使用以下函数调用方式来调用：

```java
println(compareIntegersV6(12)) // Compiler won't allow 
```

如果我们尝试以这种方式调用函数，Scala 编译器将会抛出一个错误，因为编译器无法将值 `12` 绑定到 `value2`*，* 这是因为参数的顺序问题。如果我们能以某种方式告诉编译器我们传递的参数绑定到了名为 `value2` 的参数上，我们的函数就能正常工作。为了实现这一点，我们通过命名传递参数来调用函数。

# 在传递命名参数时调用函数

是的，在调用函数时，我们可以直接命名参数。这确保了参数传递的正确顺序不受限制。让我们调用我们的函数：

```java
def compareIntegersV6(value1: Int = 10, value2: Int): String = { 
 println("Executing V6") 

  def giveAMeaningFullResult(result: Int) = result match { 
    case 0 => "Values are equal" 
    case -1 => s"$value1 is smaller than $value2" 
    case 1 => s"$value1 is greater than $value2" 
    case _ => "Could not perform the operation" 
  } 

  val result = if (value1 == value2) 0 else if (value1 > value2) 1 else -1 
  giveAMeaningFullResult(result) 
} 

println(compareIntegersV6(value2 = 12)) 
```

以下结果是：

```java
Executing V6 
10 is smaller than 12 
```

原因很简单：唯一要确保的是 Scala 编译器能够推断。这也允许你无论函数签名中出现的顺序如何，都可以传递参数。因此，我们可以这样调用我们的函数：

```java
println(compareIntegersV6(value2 = 12, value1 = 10)) 
```

以下结果是：

```java
Executing V6 
10 is smaller than 12 
```

这给我们提供了定义和调用函数的多种方式。好消息是，你还可以以字面量的形式将函数传递给函数；我们称之为函数字面量。让我们看看函数字面量是什么样的。

# 函数字面量

我们可以将一个函数以字面量的形式传递给另一个函数，让它为我们工作。让我们以相同的 `compareIntegers` 函数为例：

```java
def compareIntegersV6(value1: Int = 10, value2: Int): Int = ??? 
```

我们知道我们的函数应该做什么：接收两个整数作为输入，并返回一个整数响应，告诉我们比较的结果。如果我们看一下我们函数的抽象形式，它将看起来像这样：

```java
(value1: Int, value2: Int) => Int     
```

这意味着该函数期望两个整数，并返回一个整数响应；我们的需求是相同的。这是一个抽象形式，表示左侧的元素是输入，右侧的元素是函数的响应类型。我们可以说这是它的字面量形式，也称为**函数字面量**。因此，它也可以分配给任何变量：

```java
val compareFuncLiteral = (value1: Int, value2: Int) => if (value1 == value2) 0 else if (value1 > value2) 1 else -1 
```

记得在上一章的`PagePrinter`中，我们有一个接受索引并打印该页面的`print`函数：

```java
private def print(index: Int) = println(s"Printing Page $index.") 
```

如果我们看我们的函数的形式，它接受一个整数并打印页面。所以形式将如下所示：

```java
(index: Int) => Unit 
```

这里的`Unit`关键字表示我们的字面量不会返回任何值。现在让我们考虑一个场景，要求告诉打印机以彩色或简单的方式打印一页。我们将重构我们的代码以支持使用函数字面量：

```java
object ColorPrinter extends App { 

  def printPages(doc: Document, lastIndex: Int, print: (Int) => Unit) = if(lastIndex <= doc.numOfPages) for(i <- 1 to lastIndex) print(i) 

  val colorPrint = (index: Int) => println(s"Printing Color Page $index.") 

  val simplePrint = (index: Int) => println(s"Printing Simple Page $index.") 

  println("---------Method V1-----------") 
  printPages(Document(15, "DOCX"), 5, colorPrint) 

   println("---------Method V2-----------") 
   printPages(Document(15, "DOCX"), 2, simplePrint) 
} 

case class Document(numOfPages: Int, typeOfDoc: String) 
```

以下结果是：

```java
---------Method V1----------- 
Printing Color Page 1\. 
Printing Color Page 2\. 
Printing Color Page 3\. 
Printing Color Page 4\. 
Printing Color Page 5\. 
---------Method V2----------- 
Printing Simple Page 1\. 
Printing Simple Page 2\. 
```

我们重构了`printPages`方法，现在它接受一个*函数字面量*。函数字面量代表我们的`print`函数的形式。我们表示了两种`print`函数的形式，第一种打印*彩色*页面，第二种打印*简单*页面。这使得调用相同的`printPages`函数并按需传递一个*函数字面量*变得简单。我们只需要告诉函数这种函数可以被传递，在调用函数时，我们可以传递相同形式的函数字面量。

Scala 在默认构造中也使用*函数字面量*。一个例子是集合的`filter`函数。`filter`函数期望一个谓词，该谓词检查条件并返回一个布尔响应，基于此，我们可以从列表或集合中过滤出元素：

```java
scala> val names = List("Alice","Allen","Bob","Catherine","Alex") 
names: List[String] = List(Alice, Allen, Bob, Catherine, Alex) 

scala> val nameStartsWithA = names.filter((name) => name.startsWith("A")) 
nameStartsWithA: List[String] = List(Alice, Allen, Alex) 
```

我们检查名称是否以`A`开头的那部分是一个*函数字面量*的例子：

```java
 (name) => name.startsWith("A") 
```

Scala 编译器只在需要推断类型信息的地方需要额外信息；有了这个，它允许我们省略只是额外语法的部分，因此可以写成以下语法：

```java
scala> val nameStartsWithA = names.filter(_.startsWith("A")) 
nameStartsWithA: List[String] = List(Alice, Allen, Alex) 
placeholder syntax instead. What if we pass a function literal as an argument and want it to be evaluated only when it's needed, for example, a predicate that gets evaluated only if a certain functionality is active? In that case, we can pass the parameter as a named parameter. Scala does provide this functionality in the form of *call by name* parameters. These parameters get evaluated lazily whenever needed or first called. Let's take a look at some evaluation strategies provided by Scala.
```

# 评估策略

当函数中定义了一些参数时，这些函数调用期望我们在调用时传递参数。正如我们所知，我们可以传递一个在调用或使用时被评估的*函数字面量*。Scala 支持函数的*按值调用*和*按名调用*。让我们详细讨论一下。

# 按名调用

按名调用是一种评估策略，其中我们用调用函数的位置处的字面量来替换。字面量在第一次出现并被调用时进行评估。我们可以用一个简单的例子来理解这一点。首先，让我们以我们的`ColorPrinter`应用程序为例，并传递一个检查打印机是否开启的布尔函数字面量。为此，我们可以重构我们的函数：

```java
def printPages(doc: Document, lastIndex: Int, print: (Int) => Unit, isPrinterOn: () => Boolean) = { 

  if(lastIndex <= doc.numOfPages && isPrinterOn()) for(i <- 1 to lastIndex) print(i) 

} 
```

要调用这个函数，我们可以使用：

```java
printPages(Document(15, "DOCX"), 16, colorPrint, () => !printerSwitch) 
```

这种方法有两个问题。首先，它看起来很奇怪；在这里使用``() => expression`，因为我们已经知道它将是一个布尔函数字面量。其次，我们可能不希望我们的表达式在它被使用之前被评估。为此，我们将在`printPages`函数签名中进行一些小的修改：

```java
object ColorPrinter extends App { 

  val printerSwitch = false 

  def printPages(doc: Document, lastIndex: Int, print: (Int) => Unit, isPrinterOn: => Boolean) = { 

    if(lastIndex <= doc.numOfPages && isPrinterOn) for(i <- 1 to lastIndex) print(i) 

  } 

  val colorPrint = (index: Int) => { 
    println(s"Printing Color Page $index.") 
  } 

  println("---------Method V1-----------") 
  printPages(Document(15, "DOCX"), 2, colorPrint, !printerSwitch) 

} 

case class Document(numOfPages: Int, typeOfDoc: String) 
```

以下为结果：

```java
---------Method V1----------- 
Printing Color Page 1\. 
Printing Color Page 2\. 
```

仔细观察，你会发现我们在函数签名中移除了`()`括号，并添加了`*=>*`。这使得我们的代码理解这是一个*按名*参数，并且只有在调用时才会对其进行评估。这就是我们允许进行这种调用的原因：

```java
printPages(Document(15, "DOCX"), 2, colorPrint, !printerSwitch) 
```

这个调用由一个布尔表达式作为最后一个参数组成。由于我们的函数期望它是一个*按名*类型，它将在实际调用时被评估。

# 按值调用

*按值调用*是一种简单且常见的评估策略，其中表达式被评估，结果被绑定到参数上。在参数被使用的地方，绑定的值简单地被替换。我们已经看到了许多这种策略的例子：

```java
def compareIntegers(value1: Int, value2: Int): Int = 
       if (value1 == value2) 0 else if (value1 > value2) 1 else -1 

compareIntegers(10, 8) 
```

对这个函数的调用是*按值调用*策略的例子。我们简单地给出作为参数的值，这些值在函数中被参数值所替代。

这些策略为我们提供了多种调用函数的方式。此外，仅在需要时评估表达式是函数式语言的特征；这被称为*惰性评估*。我们将在第九章[使用强大的函数式构造](https://cdp.packtpub.com/learning_scala_programming/wp-admin/post.php?post=157&action=edit)中更详细地学习*惰性评估*，我们将讨论*强大的函数式构造*。

函数式编程支持这种编写函数的类比，这些函数对输入值有效且能正常工作，而不是通过错误来失败。为了支持这一点，Scala 有一个定义部分函数的功能。

# 部分函数

部分函数对于给定的每个输入都不够用，这意味着这些函数是为特定的一组输入参数定义的，以实现特定目的。为了更好地理解，让我们首先定义一个部分函数：

```java
scala> val oneToFirst: PartialFunction[Int, String] = { 
     | case 1 => "First" 
     | } 
oneToFirst: PartialFunction[Int, String] = <function1> 

scala> println(oneToFirst(1)) 
First 
```

在前面的代码中，我们定义了一个名为 `oneToFirst` 的部分函数*。我们还为我们的部分函数指定了类型参数；在我们的例子中，我们传递了 `Int` 和 `String`。`PartialFunction` 函数是 Scala 中的一个特质，定义为：

```java
trait PartialFunction[-A, +B] extends (A) => B 
```

该特性显示期望两个参数 `A` 和 `B`，它们将成为我们部分函数的输入和输出类型。我们的 `oneToFirst` 部分函数仅期望 `1` 并返回 1 的字符串表示形式作为第一个。这就是为什么当我们尝试通过传递 1 调用该函数时，它运行良好；但如果我们尝试传递任何其他参数，比如说 `2`，它将抛出一个 `MatchError`：

```java
scala> println(oneToFirst(2)) 
scala.MatchError: 2 (of class java.lang.Integer) 
  at scala.PartialFunction$$anon$1.apply(PartialFunction.scala:254) 
  at scala.PartialFunction$$anon$1.apply(PartialFunction.scala:252) 
  at $anonfun$1.applyOrElse(<console>:12) 
  at $anonfun$1.applyOrElse(<console>:11) 
  at scala.runtime.AbstractPartialFunction.apply(AbstractPartialFunction.scala:34) 
```

这是因为我们的部分函数仅适用于单个值，即 `1`；对于其他值则不适用。为了确保我们的函数不会抛出错误，我们可以使用 `isDefinedAt` 方法检查部分函数是否适用于某个值：

```java
scala> oneToFirst.isDefinedAt(1) 
res3: Boolean = true 

scala> oneToFirst.isDefinedAt(2) 
res4: Boolean = false 
```

对于我们的部分函数支持的值，`isDefinedAt` 返回 `true`；对于其他值，它返回 `false`。这些部分函数也可以组合。为此，`PartialFunction` 特质定义了两个方法：`orElse` 和 `andThen`*：*

```java
object PartialFunctions extends App { 

  val isPrimeEligible: PartialFunction[Item, Boolean] = { 
    case item => item.isPrimeEligible 
  } 

  val amountMoreThan500: PartialFunction[Item, Boolean] = { 
    case item => item.price > 500.0 
  } 

  val freeDeliverable = isPrimeEligible orElse amountMoreThan500 

  def deliveryCharge(item: Item): Double = if(freeDeliverable(item)) 0 else 50 

  println(deliveryCharge(Item("1", "ABC Keyboard", 490.0, false))) 

} 

case class Item(id: String, name: String, price: Double, isPrimeEligible: Boolean)
```

以下为结果：

```java
50.0 
```

在前面的程序中，我们定义了名为 `isPrimeEligible` 和 `amountMoreThan500` 的部分函数*，然后使用 `orElse` 方法组合了另一个部分函数，该方法检查项目是否可以免费交付。因此，部分函数为我们提供了组合和定义函数的方法，以服务于特定值集的特定目的。此外，部分函数还为我们提供了一种根据某些区分定义从给定输入值集合中分离逻辑的方法。重要的是要记住，我们的部分函数仅对一个操作数起作用。因此，这是一种一元函数的形式，程序员有责任检查对于特定值，函数是否已定义。

# 摘要

是时候总结本章内容了。在本章中，我们对 Scala 中重要的 *函数* 概念进行了简要介绍。我们从定义函数的语法开始。重要的是要知道，我们可以嵌套函数并使我们的代码看起来更整洁。我们学习了如何以各种方式调用函数，例如使用可变数量的参数、默认参数值和使用命名参数。然后我们学习了如何在 Scala 中编写函数字面量。之后，我们讨论了 Scala 中函数的几种评估策略，其中我们讨论了 *按名调用* 和 *按值调用*。最后，我们讨论了在 Scala 中定义 *部分函数* 的另一个重要概念。

通过本章，我们完成了旅程的第一部分。学习所有这些概念无疑增加了我们编写和理解成熟 Scala 代码的能力。在后面的部分，我们将继续这样做。第二部分是关于 Scala 丰富的集合层次结构。在下一章，我们将学习 Scala 提供的集合数量以及以各种方式使用集合的各种方法。
