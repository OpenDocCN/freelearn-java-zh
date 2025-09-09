# 第三章。函数

在上一章中，我们介绍了 Scala 的面向对象方面，例如类、对象、模式匹配、自类型、案例类等。我们还在我们的聊天机器人应用程序中实现了我们学到的面向对象概念。

在本章中，我们将介绍使用 Scala 进行函数式编程以及面向对象和函数式方法如何相互补充。我们还将介绍泛型类，它们通常与模式匹配一起使用。我们还将介绍如何创建用户定义的模式匹配以及为什么它是有用的。

到本章结束时，你将能够：

+   识别函数式编程的基本知识

+   识别 Scala 中泛型类型的基本知识

+   实现用户定义的模式匹配

+   识别和使用函数式组合模式

# 函数

在本节中，我们将介绍函数式编程的基础，例如函数值和高阶函数。

## 函数值

函数是什么？我们熟悉的方法必须在作用域（类或对象）中定义：

```java
  def f1(x:Int, y:Int): Int =  x + y
```

在 Scala 中，我们还可以定义一个函数值：

```java
   val f2: (Int,Int) => Int = (x,y) => (x+y).
```

在这里，我们定义了一个函数值，其类型为 `(Int,Int) => Int`。当然，与所有类型声明一样，如果可以从上下文中推断出类型，则可以省略类型。因此，此的另一种语法可以是：

```java
   val f2 = (x:Int,y:Int) => (x+y).
```

`f1(1,2)` 和 `f2(1,2)` 都将强制评估。`f1` 和 `f2` 之间的区别在于第二个是一个值，它可以存储在变量中或传递给另一个函数。

接受其他函数作为参数的函数称为高阶函数，例如：

```java
def twice(f: Int => Int): Int => Int = x => f(f(x))
```

此函数返回函数，这些函数将参数应用两次。此用法的示例如下：

```java
val incr2 = (x:Int) => x+2val incr4 = twice(incr2)
incr4(2)   //==6
twice(x=>x+2)(3)   // 7
```

通常，函数定义的语法如下：

```java
val fname: (X1 … XN)  => Y = (x1:X1, … x2:XN) => expression
```

如果可以从上下文中推断出类型，则可以省略类型签名。

现在，让我们看看如何定义一个函数变量。

在 IDE 工作表的 REPL 中定义函数变量：

```java
  val f:  Int=>Int = x => x+1
```

## 从面向对象的角度定义函数

当查看 Scala 源代码时，我们将看到以下定义：

```java
trait Function1[-T1, +R] {
def apply(v1: T1): R
  ….
}
```

在这里，`Function1` 是一个具有一个参数的函数的基础特质。`Function1` 有一个抽象方法：`apply`。Scala 为 `apply` 方法的调用提供了语法糖。

`T1` 和 `R` 是 `Function1` 的类型参数。`T1` 是第一个参数的类型，而 `R` 是结果类型。

类型参数前的符号 [ `-T1, +R` ] 表示参数的 `逆变` 和 `协变`；我们将在稍后详细讨论一个。现在，让我们编写一个定义：

```java
 F[T]  covariant,  iff   A <: B =>  F[A]  <:  F[B]
  F[T]   contravariant  iff  A >: B => F[A] <: F[B]
```

一个具有一个参数的函数的值，`f: A => B`，只是一个 `Function1[A,B]` 特质的实例。对于具有两个参数的函数，我们有 `Function2[T1,T2,R]` 等等。我们可以使用面向对象的设施以以下形式重写使用 `twice` 的示例：

```java
//   val twice(f:Int=>Int):Int=>Int = x => f(f(x))
object  Twice extends Function1[Function1[Int,Int],Function1[Int,Int]] 
{
	def apply(f: Function1[Int,Int]): Function1[Int,Int] =new Function1[Int,Int] 
	{
		def apply(x:Int):Int =   f.apply(f.apply(x))
	}
}
```

在这里，我们定义了一个具有之前描述的函数相同行为的 `Twice` 对象：

```java
val incr2 = (x:Int) => x+2
val incr4 = Twice(incr2)
incr4(2)  //=6
Twice(x=>x+2)(5) //=9
```

总结来说，我们可以这样说：

+   函数值是适当函数特质的实例。

+   函数值的调用是函数特质中 `apply()` 方法的调用。

现在，是时候创建一些函数了。

1.  在你的项目中打开一个空白工作表。

1.  定义一个接受二元函数和参数的函数，并返回该函数的应用：

    ```java
    g(f: (Int,Int)=>Int, x: Int, y:Int): Int = f(x,y)
    ```

1.  通过使用柯里化重新表述函数来改善语法：

    ```java
    g(f: (Int,Int)=>Int)( x: Int, y:Int): Int = f(x,y)
    ```

1.  部分应用：编写 `fix` 函数，该函数接受二元函数和参数，并返回一个一元函数，该函数将应用放置的函数和参数。例如，定义 `g` 为：`val g = fix1((x,y)=>x+y,3)`。

1.  `g(2)` 应该被评估为 `5`。

    ```java
    fix(f: (Int,Int)=>Int)( x: Int): Int => Int = y => f(x,y)
    ```

## 转换

我们有方法和函数值，它们可以完成相同的事情。期望它们之间有转换是合乎逻辑的，例如，将方法分配给函数变量或将函数作为面向对象接口传递的能力。

Scala 提供了一种特殊的语法，用于将方法转换为值。我们只需在方法名称后添加下划线（_）即可。

例如：

```java
val printFun = Console.print _
```

此外，一个函数值可以隐式转换为所谓的 SAM（单抽象方法）特质。SAM 特质只有一个抽象方法，我们可以在需要 SAM 特质（并且函数类型符合方法签名）的上下文中传递一个函数：

```java
val th = new Thread(()=> println(s"threadId=${Thread.currentThread().getId}"))
th.start()
```

在这里，我们将零参数的函数传递给 `Thread` 构造函数，该构造函数接受 `Runnable` 接口。

在 Scala 中，我们有三种不同的方式来实现延迟调用功能。

## 定义和测量单元函数的时间

定义一个函数，该函数接受一个运行单元函数并测量纳秒时间的函数。这需要多长时间？以三种不同的方式实现它。

1.  编写一个接受其他函数的函数。运行一个并测量执行时间：

    ```java
    def measure1(f: Unit => Unit): Long = {
    	val startTime = System.nanoTime()
    f()
    	val endTime = System.nanoTime()
    endTime – startTime
    }

    ```

1.  编写一个接受按名参数的函数。运行一个并测量执行时间：

    ```java
    def measure2(f: => Unit): Long = {
    	val startTime = System.nanoTime()
    	f
    	val endTime = System.nanoTime()
    	endTime – startTime
    }

    ```

1.  编写一个扩展 `Function1` 特质的对象，并在 `apply` 方法中做同样的事情：

    ```java
    object Measure3 extends Function1[Unit=>Unit,Long]
    {
    	override def apply(f: Unit=>Unit) = {
    		val startTime = System.nanoTime()
    		f()
    		val endTime = System.nanoTime()
        endTime – startTime
       }
    }
    ```

## 函数定义中的语法糖

有时候，编写如 `x => x+1` 这样的表达式看起来过于冗长。为了解决这个问题，存在语法糖，它允许你以紧凑和惯用的方式编写小的函数表达式。根本不需要写左边的部分，在编写时，使用 _（下划线）代替参数。第一个下划线表示第一个参数，第二个下划线表示第二个参数，依此类推：

```java
_ + 1   is a shortcut for  x =>  x+1,   _ + _  -- for (x,y) => x + y.
```

一些函数，如 `(x,y) => x*x + y`，不能用这种表示法表示。

## 部分函数

部分函数也被称为部分定义函数——一些值存在于函数输入域中，其中该函数未定义。

让我们看看一个简化的定义：

```java
trait PartialFunction[-A, +B] extends (A => B) { 
/** Checks if a value is contained in the function's domain.
*
*@param  x   the value to test
*  @return `'''true'''`, iff `x` is in the domain of this function, `'''false'''` otherwise.*/
def isDefinedAt(x: A): Boolean
}
```

除了 apply 方法外，还有一个 `isDefinedAt` 方法，它返回 `true` 如果我们的函数适用于一个参数，并且有一个特殊的语法：

```java
val pf: PartialFunction[Int,String] = {
case 0 => "zero"
case 1 => "one"
case 2 => "two"
case x:Int if x>0 => "many"}

pf.isDefinedAt(1)  - true
pf.isDefinedAt(-1)  - false

pf(-1)  throws exceptions.
```

我们可以将几个部分函数组合成一组新的标准组合子—`orElse`：

```java
val pf1: PartialFunction[Int,String] = pf orElse { case _ => "other" }
```

注意，类型注解是必需的，以给出正确的上下文以进行类型推导。否则，编译器将无法推导内联函数参数的类型。

一个有用的组合子—`andThen`，它允许构建管道，也是必要的：

```java
pf andThen (_.length)(1)  
```

现在，定义一个接受函数并提供转换函数的函数。例如，让输入函数为`f`：`Int => Int`，然后构建`g(f)`：`g(f)(x) = f(x) + x`。如果`f`在`x`处未定义，`g(f)`也不能定义。

1.  复制以下类：

    ```java
    class g1(f:PartialFunction[Int,Int]) extends 
    PartialFunction[Int,Int] {
    	override def isDefinedAt(x: Int) =
    		f.isDefinedAt(x)
    	override def apply(x: Int) =
    		f(x) + x 
    }
    ```

1.  或者作为一个带有`if`子句的 case 表达式：

    ```java
                   def g2(f:PartialFunction[Int,Int]):PartialFunction[Int,Int] = {
    case x if f.isDefinedAt(x) => f(x)+x
                    }
    ```

我们现在将实现一个部分函数，用于在名称和值之间构建关联。

1.  编写一个函数，作为具有参数（name,value）的类，仅在参数等于 name 时定义：

    ```java
    class NVPair(name: String, value: String) extends 
    PartialFunction[String,String] {
    	override def isDefinedAt(x: String): Boolean = (x==name)
    	override def apply(x: String): String = {
    		if (x==name) value else throw new MatchError()
    	}
    }
    ```

1.  使用`orElse`组合子将这样的对组合成更大的函数：

    ```java
    val f = new NVPair("aa","bb") orElse new NVPair("cc","dd")
    ```

# 探索模式匹配

现在，我们将回到模式匹配，并了解扩展 case 类背后的功能。正如你将从上一章中记住的那样，我们可以对 case 类使用模式匹配，其中类的字段可以绑定到适当 case 子句作用域内的变量。我们能否为我们的非 case 类做这件事，并嵌入我们自己的自定义匹配逻辑？

在本节中，我们将学习如何编写我们自己的模式匹配器，并熟悉一些常用于模式匹配的标准通用类。

现在，让我们从一个最小示例开始。

1.  首先，我们在 IDE 中编写以下代码：

    ```java
    case class Wrapper(x:Int)

    w match {case Wrapper(x) => doSomething(x)}
    ```

1.  在底层，编译器将其编译为下一个中间形式：

    ```java
    val container = Wrapper.unapply(w)
    if (container.isDefined) {
    	val x = container.get
    	doSomething(x)
    }
    ```

    +   伴随对象的`unapply`方法被调用，它必须返回具有 get 和`isDefined`方法的类。

1.  当我们有一个以上的绑定变量时，结果容器应包含一个元组。例如，对于点中间形式，这将产生以下代码：

    ```java
    val container = Point.unapply(p)
    if (container.isDefined) 
    {
      val (x,y) = container.getdoSomething(x,y)
    }
    ```

    +   标准 Scala 库提供了`Option`类型，尽管你可以使用这样的方法（这在某些重优化场景中可能很有用）来定义自己的类型。

1.  定义类：

    ```java
    sealed abstract class Option[+A]  {

     def isEmpty: Boolean
     def isDefined: Boolean = !isEmpty
     def get: A

      // …  other methods

    }

    final case class Some+A extends Option[A] {
      def isEmpty = false
      def get = value
    }

    case object None extends Option[Nothing] {
      def isEmpty = true
      def get = throw new NoSuchElementException("None.get")
    }
    ```

    +   在这里，我们看到带有通用类型参数 A 的代数类型（例如 case 类/对象的层次结构）。你可能听说过在提到这种结构时使用的缩写 GADT（通用代数数据类型）。

    +   `Option[A]`的非正式值——一个包含一个或零个元素或可能存在或可能不存在的元素的容器。Some(a)——当元素存在时，None——对于其不存在。

    +   `None`扩展了`Option[Nothing]`。`Nothing`是 Scala 类型系统中的一个`最小`类型，它是任何类型的子类型。

1.  因此，为了定义自定义模式匹配器，我们需要创建一个具有`unapply`方法的对象，并在其中放置逻辑，该逻辑在选项容器中返回一个绑定变量（或绑定变量的元组）：

    ```java
    case class Point(x:Int, y:Int)
    ```

1.  让我们定义一个模式匹配器`Diagonal`，它将只匹配位于对角线上的点：

    ```java
    object Diagonal {
    def unapply(p:Point): Option[Int] =if (p.x == p.y) Some(p.x) else None
    }
    ```

我们现在将实现`unapply`自定义。

1.  定义对象`Axis`（编号项目符号）

1.  定义方法`unapply`（编号项目符号结束）

1.  确定对象是否在 X 轴上（将 BULLET INSIDE BULLET 应用于所有三个点）

1.  确定对象是否在 Y 轴上

1.  否则，返回`None`。

## 在模式匹配器中绑定变量序列

有时候，我们需要一种特定的模式匹配器，其中绑定变量的数量可以变化。例如，标准 Scala 库中的正则表达式如下：

```java
val r1 = "([\\d]+)".r
val r2 = "([\\d]+)  ([^\\W]*)".r

v match {case r1(x) => "1"case r2(x,y) => "2"
}
```

这里，我们可以看到`r1`与一个变量匹配，但`r2`使用了两个绑定变量。对于这种情况，存在另一个约定：伴生对象应该提供`unapplySeq`方法而不是`unapply`，它返回一个包裹在选项中的序列。

我们将在下一章中学习更多关于序列的知识，但到目前为止，我们可以这样说，`Seq[A]`是一个用于序列的泛型特质。序列中的`apply`操作符作为索引访问工作（例如，`seq(n)`返回序列的第 n 个元素，并且可以使用`Seq`伴生类创建默认序列，例如`Seq(1,2,3)`）。

现在我们来实现自定义的`unapplySeq`方法。这个方法定义在字符串上，并返回一个单词序列。

1.  定义`Words`对象。

1.  定义`unapplySeq`方法。将数组转换为`seq`，使用 Scala Array 中的`.toSeq`方法：

    ```java
    object Words {def unapplySeq(arg: String):Option[Seq[String]] = {
    val array = arg.split("\\W+")
    if (array.size == 1 && array(0).isEmpty ) {
          None} else {
      Some(array.toSeq)
      }
      }
    }
    ```

1.  编写一个测试，比较以下字符串的单词：

    ```java
        "1",    "AA AA",  "AA   AA",   "ABC CBA",  "A B C D E     F G X-L",""    
    "AAA     AAA" match {case Words(x,y) => (x,y)
    }
    ```

    +   有时候，当将变量绑定到序列中时，我们不需要为序列中的每个值使用`var`，而只需要第一个值和序列的其余部分。

1.  我们可以使用模式匹配中的变量函数调用语法，例如：

    ```java
    object AsSeq
    {
    def unapplySeq(x:Array[Int]):Option[Seq[Int]] = {Some(x)
    }
    }Array(1,2,3,6) match {case AsSeq(h, _*) => h 
    }
    ```

# 实践中的部分函数

现在我们已经学到了很多关于函数和模式匹配的知识，让我们将我们的理论知识应用到实际编程中。

让我们获取我们在上一章中开发的聊天机器人，并将模式改为部分函数而不是类。

### 注意

在补充材料中打开`/Lesson 3/5-project`，并将项目导入 IDE。

## 将 ChatbotMode 表示为部分函数

让我们导航到`com.packt.courseware.l4`中的`scala`文件包：

```java
package com.packt.courseware.l4

package object modes {
  type ChatbotMode = PartialFunction[(String,EffectsProvider),Processed]

     …
}
```

这里，我们看到`package`对象，这是我们之前章节中没有提到的。

`package`对象是与包相关联的对象。当你使用通配符导入一个包时，如果存在，则导入包对象的当前作用域内容。

因此，`package`对象是一个存储一些实用定义和函数的好方法，这些函数应该在包中可用。

下一个句子是`ChatbotMode`的类型别名：我们将其定义为一个从（`String`，`EffectsProvider`）到`Processed`的部分函数。

如你所记，`Processed`是一个`LineStepResult`特质，与 Processed 或 Failed 联合。使用部分函数，我们不需要`Failed`变体；相反，我们模式中的`isDefined`将被设置为`false`。

现在我们来看一些简单的模式：

```java
val bye: ChatbotMode = { 
case ("bye", eff) => Processed("bye", bye, true) 
}
```

因此，我们可以像写`vars`一样编写部分函数。

在上一个版本中，我们有`OrMode`，它组合了模式。我们能否用部分函数做同样的事情？

```java
def or(frs:ChatbotMode, snd: ChatbotMode): ChatbotMode = {
val frsPost = frs.andThen(p => p.copy(nextMode = or(p.nextMode,snd)))
val sndPost = snd.andThen(p => p.copy(nextMode = or(p.nextMode,frs)))
frsPost orElse sndPost
}
```

我们使用`andThen`组合子来后处理`frs`和`snd`应用的结果，以便在或链中插入`nextMode`，并通过`orElse`组合子返回这些函数。

因此，正如我们所见，我们可以借助部分函数来描述模式。生成的代码稍微短一些，但我们只失去了组合模式的复杂语法。

主模式现在看起来是这样的：

```java
import modes._
def createInitMode() = otherwise (
or(StoreRemindCommand.empty, or(bye,or(currentDate,currentTime))),
  interestingIgnore)
```

现在我们来实现部分函数。在 l4 中，一些模式已从源代码中删除。你能以部分函数的形式将它们移回来吗？

**完成步骤：**

1.  打开`Lesson 3/5-project`。

1.  实现当前的`currentTime`、`otherwise`和`interestingIgnore`模式。

1.  确保测试正在运行。

## 将提醒存储实现为部分函数的集合

让我们看看`RemindStore`的实现。导航到`com/packt/courseware/l4/RemindCommand.scala`。

在模式中使用正则表达式：

```java
val StorePattern = raw"store ([^\W]+) (.*)".r;
val RemindPattern = raw"remind ([^\W]+)".r;

def process(state:RemindedState): ChatbotMode =
{
  case (StorePattern(n,v),effects) => Processed("ok",process(state.store(n,v)),false)
  case (RemindPattern(n),effects) if state.isDefinedAt(n) => Processed(state(n),process(state),false)
}
```

注意`RemindedState`有一个内存泄漏：当我们要求我们的聊天机器人存储相同的单词几次时，函数的行为会是什么？

### 注意

**注意**

内存泄漏是一种情况，我们分配了一个对象，但在使用后仍然可以访问它。

现在我们来查找并修复`StoreRemindCommand`中的内存泄漏。

1.  打开`Lesson 3/5-project`。

1.  分析我们存储相同工作几次的情况。

1.  考虑如何为这个写单元测试（`***`）？

1.  修复内存泄漏。

正如我们所见，在聊天机器人中构建模式作为部分函数是可能的。

## 使用提升进行全函数和部分函数之间的对话

这样的设计有一些缺点：

第一个缺点是我们的部分函数始终接受一个参数：输入和效果的元组。这可能是一个混淆的来源。

还要注意，在处理输入或拒绝（它将通过组合子传递到下一个链）的决策中，应该写两次：首先在`isDefinedAt`中，然后在`apply`中。在简单的情况下，这通过 case 语法隐藏起来，其中`isDefinedAt`是自动生成的。

它看起来像丢失了二元运算符语法是第三个问题。然而，这并不是一个真正的问题。我们将在第五章中学习如何在第三方类上定义自己的语法。

我们能否有一个决策点并处理一个部分定义的值？

让我们看看标准库中的下一个方法：

```java
trait PartialFunction[-A, +B] extends (A => B) {
 ….

  /** Turns this partial function into a plain function returning an `Option` result.
  *  @see     Function.unlift
  *  @return  a function that takes an argument `x` to `Some(this(x))` if `this`
  *           is defined for `x`, and to `None` otherwise.
  */
  def lift: A => Option[B]

}
```

我们可以将部分函数表示为将结果包装在`Option`中的全函数。对于部分函数的组合器，我们有与`Option`非常类似的方法。

让我们再次改变我们模式的设计。

查看`Lesson 3/6-project`。

`ChatbotMode`是一个特质：

```java
trait ChatbotMode {
def process(line:String,effects:EffectsProvider):Option[Processed]
def or(other: ChatbotMode) = OrMode(this,other)
def otherwise(other: ChatbotMode) = OtherwiseMode(this,other)}
```

但我们可以通过部分函数的帮助定义简单的模式，并使用`helper`构造函数将它们转换到我们的特质中：

```java
object ChatbotMode{
def partialFunction(f:PartialFunction[String,Processed]): ChatbotMode =
{(line,effects) => f.lift(line) }}
```

之后，我们可以这样做：

```java
val bye: ChatbotMode = ChatbotMode.partialFunction(
                       { case "bye" => Processed("bye", bye, true) })
```

还要注意，我们可以从函数初始化`ChatbotMode`，因为`ChatbotMode`是一个 SAM 类型：

```java
val interestingIgnore: ChatbotMode = ( line, effects ) => 
                       Some(Processed("interesting...",interestingIgnore,false))
```

此外，我们可以将`OrMode`的实现与基于部分函数组合器的先前变体进行比较：

```java
case class OrMode(frs:ChatbotMode, snd:ChatbotMode) extends ChatbotMode {
	override def process(line: String, effects: EffectsProvider): Option[Processed] = {
		frs.process(line,effects).map(p => p.copy(nextMode = OrMode(p.nextMode,snd))
)orElse snd.process(line,effects).map(
p => p.copy(nextMode = OrMode(p.nextMode,frs))
		)
	}
}

```

如您所见，该结构非常相似：在部分函数中使用`andThen`代替，Option 也使用了`orElse`。我们可以说，`PartialFunction[A,B]`和`Function[A,Option[B]]`的域是同构的。

从部分函数到选项函数的默认转换器，命名为`lift`。

这是一个部分函数的方法：

```java
{ case "bye" => Processed("bye", bye, true) }.lift

```

这将产生与这个相同的效果：

```java
          x => if (x=="bye") Some(Processed("bye", bye, true)) else None
```

让我们编写一个逆转换器，`unlift`：

```java
def unliftX,Y:PartialFunction[X,Y] = new PartialFunction[X,Y] {
	override def isDefinedAt(x: X) =
	f(x).isDefined
	override def apply(x: X) =
	f(x) match {
		case Some(y) => y
	}
}
```

提供更高效的链式操作是一种好习惯，例如：

```java
override def applyOrElseA1 <: X, B1 >: Y: B1 =
	f(x) match {
		case Some(y) =>y
		case None => default(x)  
}
```

这里，我们调用底层的`f`一次。

现在我们给我们的聊天机器人添加一个简单的 TODO 列表。

我们将通过允许多个模式评估输入来改变我们的评估模型。组合器将选择最佳的评估。

1.  打开`Lesson 3/6-project`。

1.  在`Processed`和`relevance`参数之间添加 0 到 1。

1.  修改`or`组合器，以评估两个子模式并基于其相关性选择答案。

1.  在`test`用例中添加一个测试。

# 摘要

在本章中，我们介绍了 Scala 的函数式编程以及面向对象和函数式方法如何相互补充。我们还介绍了泛型类，它们通常与模式匹配一起使用。最后，我们介绍了如何创建用户定义的模式匹配，并学习了为什么它是有用的。

在下一章中，我们将介绍重要的 Scala 集合，如`Sets`和`Maps`。我们还将讨论可变和不可变集合及其在 Scala 代码中的应用。
