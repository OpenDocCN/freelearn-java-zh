# 第三章.释放 Scala 性能

在本章中，我们将探讨 Scala 特定的构造和语言特性，并检查它们如何帮助或损害性能。凭借我们新获得的性能测量知识，我们将分析如何更好地使用 Scala 编程语言提供的丰富语言特性。对于每个特性，我们将介绍它，展示它如何编译成字节码，然后确定在使用此特性时的注意事项和其他考虑因素。

在本章中，我们将展示 Scala 源代码和由 Scala 编译器生成的字节码。检查这些工件对于丰富你对 Scala 如何与 JVM 交互的理解是必要的，这样你就可以对你的软件的运行时性能有一个直观的认识。我们将在编译命令后通过调用`javap`Java 反汇编器来检查字节码，如下所示：

```java
javap -c <PATH_TO_CLASS_FILE>

```

减号`c`开关打印反汇编的代码。另一个有用的选项是`-private`，它打印私有定义的方法的字节码。有关`javap`的更多信息，请参阅手册页。我们将涵盖的示例不需要深入了解 JVM 字节码知识，但如果你希望了解更多关于字节码操作的信息，请参阅 Oracle 的 JVM 规范[`docs.oracle.com/javase/specs/jvms/se7/html/jvms-3.html#jvms-3.4`](http://docs.oracle.com/javase/specs/jvms/se7/html/jvms-3.html#jvms-3.4)。

定期，我们还将通过运行以下命令来检查移除 Scala 特定功能的 Scala 源代码版本：

```java
scalac -print <PATH>

```

这是一种有用的方式，可以了解 Scala 编译器如何将便捷的语法转换为 JVM 可以执行的构造。在本章中，我们将探讨以下主题：

+   值类和标记类型

+   专业化

+   元组

+   模式匹配

+   尾递归

+   `Option`数据类型

+   `Option`的替代方案

# 值类

在第二章《在 JVM 上测量性能》中，我们介绍了订单簿应用程序的领域模型。这个领域模型包括两个类，`Price`和`OrderId`。我们指出，我们为`Price`和`OrderId`创建了领域类，以提供对包装的`BigDecimal`和`Long`的上下文意义。虽然这为我们提供了可读的代码和编译时安全性，但这种做法也增加了我们的应用程序创建的实例数量。分配内存和生成类实例通过增加收集频率和可能引入额外的长生命期对象，为垃圾收集器增加了更多的工作。垃圾收集器将不得不更加努力地收集它们，这个过程可能会严重影响我们的延迟。

幸运的是，从 Scala 2.10 开始，`AnyVal`抽象类可供开发者为解决此问题定义自己的值类。`AnyVal`类在 Scala 文档中定义（[`www.scala-lang.org/api/current/#scala.AnyVal`](http://www.scala-lang.org/api/current/#scala.AnyVal)）为，“所有值类型的根类，描述了在底层宿主系统中未实现为对象的值。”`AnyVal`类可用于定义值类，该类会接受编译器的特殊处理。值类在编译时优化，以避免分配实例，而是使用包装类型。

## 字节码表示

例如，为了提高我们的订单簿的性能，我们可以将`Price`和`OrderId`定义为值类：

```java
case class Price(value: BigDecimal) extends AnyVal 
case class OrderId(value: Long) extends AnyVal 

```

为了说明值类的特殊处理，我们定义了一个假方法，该方法接受一个`Price`值类和一个`OrderId`值类作为参数：

```java
def printInfo(p: Price, oId: OrderId): Unit = 
  println(s"Price: ${p.value}, ID: ${oId.value}") 

```

从这个定义中，编译器生成以下方法签名：

```java
public void printInfo(scala.math.BigDecimal, long); 

```

我们看到生成的签名接受一个`BigDecimal`对象和一个`long`对象，尽管 Scala 代码允许我们利用模型中定义的类型。这意味着我们不能在调用`printInfo`时使用`BigDecimal`或`Long`的实例，因为编译器会抛出一个错误。

### 注意

一个有趣的现象是，`printInfo`的第二个参数不是编译为`Long`（一个对象），而是`long`（一个原始类型，注意小写的'l`）。`Long`和其他与原始类型匹配的对象，如`Int`、`Float`或`Short`，会被编译器特别处理，在运行时以它们的原始类型表示。

值类也可以定义方法。让我们丰富我们的`Price`类，如下所示：

```java
case class Price(value: BigDecimal) extends AnyVal { 
  def lowerThan(p: Price): Boolean = this.value < p.value 
} 

// Example usage 
val p1 = Price(BigDecimal(1.23)) 
val p2 = Price(BigDecimal(2.03)) 
p1.lowerThan(p2) // returns true 

```

我们的新方法使我们能够比较两个`Price`实例。在编译时，为`Price`创建了一个伴随对象。这个伴随对象定义了一个`lowerThan`方法，该方法接受两个`BigDecimal`对象作为参数。实际上，当我们对一个`Price`实例调用`lowerThan`时，代码被编译器从实例方法调用转换为在伴随对象中定义的静态方法调用：

```java
public final boolean lowerThan$extension(scala.math.BigDecimal, scala.math.BigDecimal); 
    Code: 
       0: aload_1 
       1: aload_2 
       2: invokevirtual #56  // Method scala/math/BigDecimal.$less:(Lscala/math/BigDecimal;)Z 
       5: ireturn 

```

如果我们将前面的 Scala 代码写成伪代码，它看起来可能如下所示：

```java
val p1 = BigDecimal(1.23) 
val p2 = BigDecimal(2.03) 
Price.lowerThan(p1, p2)  // returns true 

```

## 性能考虑

值类是我们开发者工具箱中的一个很好的补充。它们帮助我们减少实例数量，并为垃圾收集器节省一些工作，同时允许我们依赖有意义的类型，这些类型反映了我们的业务抽象。然而，扩展 `AnyVal` 需要满足一系列条件，这些条件类必须满足。例如，值类可能只有一个主构造函数，该构造函数接受一个公共 `val` 作为单个参数。此外，此参数不能是值类。我们看到了值类可以通过 `def` 定义方法。值类内部不允许使用 `val` 或 `var`。嵌套类或对象定义也是不可能的。另一个限制条件阻止值类扩展任何不是通用特质的类型，即扩展 `Any` 的特质，只有 `defs` 作为成员，并且不执行初始化。如果这些条件中的任何一个没有得到满足，编译器将生成错误。除了前面列出的限制条件之外，还有一些特殊情况，其中值类必须由 JVM 实例化。这些情况包括执行模式匹配或运行时类型测试，或将值类赋值给数组。后者的一个例子如下所示：

```java
def newPriceArray(count: Int): Array[Price] = { 
  val a = new ArrayPrice 
  for(i <- 0 until count){ 
    a(i) = Price(BigDecimal(Random.nextInt())) 
  } 
  a 
} 

```

生成的字节码如下：

```java
public highperfscala.anyval.ValueClasses$$anonfun$newPriceArray$1(highperfscala.anyval.ValueClasses$Price[]); 
    Code: 
       0: aload_0 
       1: aload_1 
       2: putfield      #29  // Field a$1:[Lhighperfscala/anyval/ValueClasses$Price; 
       5: aload_0 
       6: invokespecial #80  // Method scala/runtime/AbstractFunction1$mcVI$sp."<init>":()V 
       9: return 

public void apply$mcVI$sp(int); 
    Code: 
       0: aload_0 
       1: getfield      #29  // Field a$1:[Lhighperfscala/anyval/ValueClasses$Price; 
       4: iload_1 
       5: new           #31  // class highperfscala/anyval/ValueClasses$Price 
       // omitted for brevity 
      21: invokevirtual #55  // Method scala/math/BigDecimal$.apply:(I)Lscala/math/BigDecimal; 
      24: invokespecial #59  // Method highperfscala/anyval/ValueClasses$Price."<init>":(Lscala/math/BigDecimal;)V 
      27: aastore 
      28: return 

```

注意 `mcVI$sp` 是如何从 `newPriceArray` 中调用的，并在第 `5` 条指令处创建了一个新的 `ValueClasses$Price` 实例。

由于将单个字段案例类转换为值类与扩展 `AnyVal` 特质一样简单，我们建议您尽可能使用 `AnyVal`。开销相当低，并且它在垃圾收集性能方面带来了高收益。要了解更多关于值类、它们的限制和使用案例，您可以在[`docs.scala-lang.org/overviews/core/value-classes.html`](http://docs.scala-lang.org/overviews/core/value-classes.html)找到详细的描述。

## 标签类型 - 值类的替代方案

值类是一个易于使用的工具，并且它们可以在性能方面带来显著的提升。然而，它们附带一系列限制条件，这可能会在某些情况下使它们无法使用。我们将通过查看 `Scalaz` 库实现的标签类型功能来结束本节，该功能提供了一个有趣的替代方案（[`github.com/scalaz/scalaz`](https://github.com/scalaz/scalaz)）。

### 注意

`Scalaz` 对标签类型的实现受到了另一个 Scala 库的启发，该库名为 `shapeless`。`shapeless` 库提供了编写类型安全、泛型代码的工具，且具有最少的样板代码。虽然我们不会深入探讨 `shapeless`，但我们鼓励您了解更多关于这个项目的信息，请访问[`github.com/milessabin/shapeless`](https://github.com/milessabin/shapeless)。

标记类型是强制编译时类型检查而不产生实例实例化成本的另一种方式。它们依赖于在`Scalaz`库中定义的`Tagged`结构类型和`@@`类型别名，如下所示：

```java
type Tagged[U] = { type Tag = U } 
type @@[T, U] = T with Tagged[U] 

```

让我们重写我们代码的一部分，以利用`Price`对象进行标记类型：

```java
object TaggedTypes { 

  sealed trait PriceTag 
  type Price = BigDecimal @@ PriceTag 

  object Price { 
    def newPrice(p: BigDecimal): Price = 
      TagBigDecimal, PriceTag 

    def lowerThan(a: Price, b: Price): Boolean = 
      Tag.unwrap(a) < Tag.unwrap(b) 
  } 
} 

```

让我们简要地浏览一下代码片段。我们将定义一个`PriceTag`密封特质，我们将用它来标记我们的实例，创建并定义一个`Price`类型别名，它是一个标记为`PriceTag`的`BigDecimal`对象。`Price`对象定义了有用的方法，包括用于标记给定的`BigDecimal`对象并返回一个`Price`对象（即标记的`BigDecimal`对象）的`newPrice`工厂函数。我们还将实现一个与`lowerThan`方法等效的函数。这个函数接受两个`Price`对象（即两个标记的`BigDecimal`对象），提取两个`BigDecimal`对象标签的内容，并比较它们。

使用我们新的`Price`类型，我们重写了之前查看过的相同`newPriceArray`方法（为了简洁，代码被省略，但你可以参考附带的源代码），并打印以下生成的字节码：

```java
public void apply$mcVI$sp(int); 
    Code: 
       0: aload_0 
       1: getfield      #29  // Field a$1:[Ljava/lang/Object; 
       4: iload_1 
       5: getstatic     #35  // Field highperfscala/anyval/TaggedTypes$Price$.MODULE$:Lhighperfscala/anyval/TaggedTypes$Price$; 
       8: getstatic     #40  // Field scala/package$.MODULE$:Lscala/package$; 
      11: invokevirtual #44  // Method scala/package$.BigDecimal:()Lscala/math/BigDecimal$; 
      14: getstatic     #49  // Field scala/util/Random$.MODULE$:Lscala/util/Random$; 
      17: invokevirtual #53  // Method scala/util/Random$.nextInt:()I 
      20: invokevirtual #58  // Method scala/math/BigDecimal$.apply:(I)Lscala/math/BigDecimal; 
      23: invokevirtual #62  // Method highperfscala/anyval/TaggedTypes$Price$.newPrice:(Lscala/math/BigDecimal;)Ljava/lang/Object; 
      26: aastore 
      27: return 

```

在这个版本中，我们不再看到`Price`的实例化，尽管我们将其分配给一个数组。标记的`Price`实现涉及到运行时转换，但我们预计这个转换的成本将低于之前值类`Price`策略中观察到的实例分配（以及垃圾收集）。我们将在本章后面再次查看标记类型，并使用它们来替换标准库中的一个知名工具：`Option`。

# 专业化

要理解专业化的重要性，首先掌握对象装箱的概念是至关重要的。JVM 定义了原始类型（`boolean`、`byte`、`char`、`float`、`int`、`long`、`short`和`double`），这些类型是在栈上分配而不是在堆上分配的。当引入泛型类型时，例如`scala.collection.immutable.List`，JVM 引用的是对象等价物，而不是原始类型。在这个例子中，一个整数列表的实例化将是堆分配的对象，而不是整数原语。将原始类型转换为对象等价物的过程称为装箱，而相反的过程称为拆箱。装箱对于性能敏感的编程来说是一个相关的问题，因为装箱涉及到堆分配。在执行数值计算的性能敏感代码中，装箱和拆箱的成本可能会造成显著的性能下降。以下是一个示例，用于说明装箱的开销：

```java
List.fill(10000)(2).map(_* 2) 

```

通过`fill`创建列表会产生 10,000 个整数对象的堆分配。在`map`中进行乘法操作需要 10,000 次拆箱以执行乘法，然后需要 10,000 次装箱将乘法结果添加到新列表中。从这个简单的例子中，你可以想象到由于装箱或拆箱操作，临界区算术将如何被减慢。

如 Oracle 在[`docs.oracle.com/javase/tutorial/java/data/autoboxing.html`](https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html)教程中所示，Java 和 Scala 中的装箱是透明的。这意味着，如果没有仔细的剖析或字节码分析，很难确定你在哪里支付了对象装箱的成本。为了改善这个问题，Scala 提供了一个名为特殊化的功能。专业化指的是在编译时生成泛型特质的或类的重复版本，这些版本直接引用原始类型而不是相关的对象包装器。在运行时，编译器生成的泛型类（或通常称为类的专业化版本）被实例化。这个过程消除了装箱原始类型的运行时成本，这意味着你可以在保持手写专业化实现性能的同时定义泛型抽象。

## 字节码表示

让我们通过一个具体的例子来更好地理解专业化过程是如何工作的。考虑一个关于购买股票数量的简单、通用的表示，如下所示：

```java
case class ShareCountT 

```

对于这个例子，让我们假设预期的用法是在`ShareCount`的整数或 long 表示之间进行交换。根据这个定义，基于 long 的`ShareCount`实例化将产生装箱的成本，如下所示：

```java
def newShareCount(l: Long): ShareCount[Long] = ShareCount(l) 

```

这个定义转换成以下字节码：

```java
  public highperfscala.specialization.Specialization$ShareCount<java.lang.Object> newShareCount(long); 
    Code: 
       0: new           #21  // class orderbook/Specialization$ShareCount 
       3: dup 
       4: lload_1 
       5: invokestatic  #27  // Method scala/runtime/BoxesRunTime.boxToLong:(J)Ljava/lang/Long; 
       8: invokespecial #30  // Method orderbook/Specialization$ShareCount."<init>":(Ljava/lang/Object;)V 
      11: areturn 

```

在前面的字节码中，在指令`5`处很明显，在实例化`ShareCount`实例之前，原始的 long 值被装箱了。通过引入`@specialized`注解，我们能够通过让编译器提供一个与原始 long 值一起工作的`ShareCount`实现来消除装箱。你可以通过提供一组类型来指定你希望专业化的类型。如`Specializables`特质([`www.scala-lang.org/api/current/index.html#scala.Specializable`](http://www.scala-lang.org/api/current/index.html#scala.Specializable))定义的，你可以为所有 JVM 原始类型、`Unit`和`AnyRef`进行专业化。对于我们的例子，让我们将`ShareCount`专业化为整数和 long，如下所示：

```java
case class ShareCount@specialized(Long, Int) T 

```

根据这个定义，字节码现在变成以下形式：

```java
  public highperfscala.specialization.Specialization$ShareCount<java.lang.Object> newShareCount(long); 
    Code: 
       0: new           #21  // class highperfscala.specialization/Specialization$ShareCount$mcJ$sp 
       3: dup 
       4: lload_1 
       5: invokespecial #24  // Method highperfscala.specialization/Specialization$ShareCount$mcJ$sp."<init>":(J)V 
       8: areturn 

```

装箱消失了，奇怪地被不同的类名`ShareCount $mcJ$sp`所取代。这是因为我们正在调用为长值特殊化的编译器生成的`ShareCount`版本。通过检查`javap`的输出，我们看到编译器生成的特殊化类是`ShareCount`的子类：

```java
 public class highperfscala.specialization.Specialization$ShareCount$mcI$sp extends highperfscala.specialization.Specialization$ShareCount<java .lang.Object> 

```

在我们转向*性能考虑*部分时，请记住这个特殊化实现细节。使用继承在更复杂的使用案例中会迫使做出权衡。

## 性能考虑

初看起来，特殊化似乎是 JVM 装箱问题的简单万能药。然而，在使用特殊化时需要考虑几个注意事项。大量使用特殊化会导致编译时间显著增加和代码大小的增加。考虑对`Function3`进行特殊化，它接受三个参数作为输入并产生一个结果。对所有类型（即`Byte`、`Short`、`Int`、`Long`、`Char`、`Float`、`Double`、`Boolean`、`Unit`和`AnyRef`）进行四个参数的特殊化会产生 10⁴ 或 10,000 种可能的排列组合。因此，标准库在应用特殊化时非常谨慎。在你的使用案例中，仔细考虑你希望特殊化的类型。如果我们只为`Int`和`Long`对`Function3`进行特殊化，生成的类数量将减少到 2⁴ 或 16。涉及继承的特殊化需要特别注意，因为在扩展泛型类时很容易丢失特殊化。考虑以下示例：

```java
  class ParentFoo@specialized T 
  class ChildFooT extends ParentFooT 

  def newChildFoo(i: Int): ChildFoo[Int] = new ChildFooInt 

```

在这个场景中，你可能会期望`ChildFoo`是用原始整数定义的。然而，由于`ChildFoo`没有用`@specialized`注解标记其类型，因此没有创建任何特殊化的类。以下是证明这一点的字节码：

```java
  public highperfscala.specialization.Inheritance$ChildFoo<java.lang.Object> newChildFoo(int); 
    Code: 
       0: new           #16  // class highperfscala/specialization/Inheritance$ChildFoo 
       3: dup 
       4: iload_1 
       5: invokestatic  #22  // Method scala/runtime/BoxesRunTime.boxToInteger:(I)Ljava/lang/Integer; 
       8: invokespecial #25  // Method highperfscala/specialization/Inheritance$ChildFoo."<init>":(Ljava/lang/Object;)V 
      11: areturn 

```

下一个逻辑步骤是将`@specialized`注解添加到`ChildFoo`的定义中。在这个过程中，我们会遇到编译器警告关于特殊化的使用，如下所示：

```java
class ParentFoo must be a trait. Specialized version of class ChildFoo will inherit generic highperfscala.specialization.Inheritance.ParentFoo[Boolean] 
class ChildFoo@specialized T extends ParentFooT 

```

编译器指出，你创建了一个菱形继承问题，其中特殊化的`ChildFoo`版本同时扩展了`ChildFoo`及其关联的特殊化版本`ParentFoo`。这个问题可以通过以下方式使用特质来建模：

```java
  trait ParentBar[@specialized T] { 
    def t(): T 
  } 

  class ChildBar@specialized T extends ParentBar[T] 

  def newChildBar(i: Int): ChildBar[Int] = new ChildBar(i) 

```

这个定义使用特殊化的`ChildBar`版本编译，正如我们最初所希望的，如下所示：

```java
  public highperfscala.specialization.Inheritance$ChildBar<java.lang.Object> newChildBar(int); 
    Code: 
       0: new           #32  // class highperfscala/specialization/Inheritance$ChildBar$mcI$sp 
       3: dup 
       4: iload_1 
       5: invokespecial #35  // Method highperfscala/specialization/Inheritance$ChildBar$mcI$sp."<init>":(I)V 
       8: areturn 

```

类似且同样容易出错的场景是当在特殊化类型周围定义泛型方法时。考虑以下定义：

```java
  class FooT 

  object Foo { 
    def createT: Foo[T] = new Foo(t) 
  } 

  def boxed: Foo[Int] = Foo.create(1) 

```

在这里，`create`的定义与继承示例中的子类类似。从`create`方法实例化的包含原始值的`Foo`实例将被装箱。以下字节码演示了`boxed`如何导致堆分配：

```java
  public highperfscala.specialization.MethodReturnTypes$Foo<java.lang.Object> boxed(); 
    Code: 
       0: getstatic     #19  // Field highperfscala/specialization/MethodReturnTypes$Foo$.MODULE$:Lhighperfscala/specialization/MethodReturnTypes$Foo$; 
       3: iconst_1 
       4: invokestatic  #25  // Method scala/runtime/BoxesRunTime.boxToInteger:(I)Ljava/lang/Integer; 
       7: invokevirtual #29  // Method highperfscala/specialization/MethodReturnTypes$Foo$.create:(Ljava/lang/Object;)Lhighperfscala/specialization/MethodReturnTypes$Foo; 
      10: areturn 

```

解决方案是在调用点应用`@specialized`注解，如下所示：

```java
def createSpecialized@specialized T: Foo[T] = new Foo(t) 

```

最后一个有趣的场景是当使用多个类型进行特殊化，其中一个类型扩展 `AnyRef` 或是值类时。为了说明这个场景，考虑以下示例：

```java
  case class ShareCount(value: Int) extends AnyVal 
  case class ExecutionCount(value: Int) 

  class Container2@specialized X, @specialized Y 

  def shareCount = new Container2(ShareCount(1), 1) 

  def executionCount = new Container2(ExecutionCount(1), 1) 

  def ints = new Container2(1, 1) 

```

在这个例子中，您期望使用哪些方法将 `Container2` 的第二个参数装箱？为了简洁，我们省略了字节码，但您可以轻松地自行检查。结果发现，`shareCount` 和 `executionCount` 会将整数装箱。编译器不会为 `Container2` 生成一个接受原始整数和扩展 `AnyVal`（例如，`ExecutionCount`）的值的专用版本。`shareCount` 方法也由于编译器从源代码中移除值类类型信息的顺序而导致装箱。在这两种情况下，解决方案是定义一个特定于一组类型的案例类（例如，`ShareCount` 和 `Int`）。移除泛型允许编译器选择原始类型。

从这些例子中可以得出的结论是，为了在整个应用程序中避免装箱，特殊化需要额外的关注。由于编译器无法推断出您意外忘记应用 `@specialized` 注解的场景，它无法发出警告。这使您必须对性能分析和检查字节码保持警惕，以检测特殊化意外丢失的场景。

### 注意

为了克服特殊化带来的某些缺点，正在积极开发一个名为 miniboxing 的编译器插件，可以在 [`scala-miniboxing.org/`](http://scala-miniboxing.org/) 上找到。这个编译器插件采用了一种不同的策略，涉及将所有原始类型编码为一个长值，并携带元数据以回忆原始类型。例如，`boolean` 可以使用一个位来表示真或假，在 `long` 中表示。这种方法下，性能在定性上与特殊化相似，但为大量排列产生了数量级的更少类。此外，miniboxing 能够更稳健地处理继承场景，并在装箱将要发生时发出警告。虽然特殊化和 miniboxing 的实现不同，但最终用户的使用方式相当相似。像特殊化一样，您必须添加适当的注解来激活 miniboxing 插件。要了解更多关于插件的信息，您可以查看 miniboxing 项目网站上的教程。

确保特殊化生成无堆分配代码的额外关注是值得的，因为在性能敏感的代码中，它带来了性能上的优势。为了强调特殊化的价值，考虑以下微基准测试，该测试通过将份额计数与执行价格相乘来计算贸易的成本。为了简单起见，直接使用原始类型而不是值类。当然，在生产代码中这种情况永远不会发生：

```java
@BenchmarkMode(Array(Throughput)) 
@OutputTimeUnit(TimeUnit.SECONDS) 
@Warmup(iterations = 3, time = 5, timeUnit = TimeUnit.SECONDS) 
@Measurement(iterations = 30, time = 10, timeUnit = TimeUnit.SECONDS) 
@Fork(value = 1, warmups = 1, jvmArgs = Array("-Xms1G", "-Xmx1G")) 
class SpecializationBenchmark { 

  @Benchmark 
  def specialized(): Double = 
    specializedExecution.shareCount.toDouble * specializedExecution.price 

  @Benchmark 
  def boxed(): Double = 
    boxedExecution.shareCount.toDouble * boxedExecution.price 
} 

object SpecializationBenchmark { 
  class SpecializedExecution@specialized(Int) T1, @specialized(Double) T2 
  class BoxingExecutionT1, T2 

  val specializedExecution: SpecializedExecution[Int, Double] = 
    new SpecializedExecution(10l, 2d) 
  val boxedExecution: BoxingExecution[Long, Double] = new BoxingExecution(10l, 2d) 
} 

```

在这个基准测试中，定义了通用执行类的两个版本。`SpecializedExecution` 由于专业化而无需装箱即可计算总成本，而 `BoxingExecution` 需要对象装箱和拆箱来执行算术。微基准测试使用以下参数化调用：

```java
sbt 'project chapter3' 'jmh:run SpecializationBenchmark -foe true'

```

### 注意

我们通过在代码中类级别放置的注解来配置这个 JMH 基准测试。这与我们在 第二章 中看到的不同，*在 JVM 上测量性能*，在那里我们使用了命令行参数。注解的优势在于为你的基准测试设置适当的默认值，并简化命令行调用。仍然可以通过命令行参数覆盖注解中的值。我们使用 `-foe` 命令行参数来启用错误时的失败，因为没有注解可以控制这种行为。在这本书的其余部分，我们将使用注解参数化 JMH，并在代码示例中省略注解，因为我们总是使用相同的值。

结果总结如下表：

| **基准测试** | **吞吐量（每秒操作数）** | **错误作为吞吐量的百分比** |
| --- | --- | --- |
| `boxed` | 251,534,293.11 | ±2.23 |
| `specialized` | 302,371,879.84 | ±0.87 |

这个微基准测试表明专业化实现提供了大约 17% 的更高吞吐量。通过在代码的关键部分消除装箱，可以通过谨慎地使用专业化获得一个数量级的性能提升。对于性能敏感的算术，这个基准测试为确保专业化正确应用所需的额外努力提供了依据。

# 元组

Scala 中的第一类元组支持简化了需要将多个值组合在一起的使用场景。使用元组，你可以使用简洁的语法优雅地返回多个值，而无需定义案例类。以下部分展示了编译器如何转换 Scala 元组。

## 字节码表示

让我们看看 JVM 如何处理创建元组，以更好地理解 JVM 如何支持元组。为了培养我们的直觉，考虑创建一个长度为二的元组，如下所示：

```java
def tuple2: (Int, Double) = (1, 2.0) 

```

此方法的对应字节码如下：

```java
  public scala.Tuple2<java.lang.Object, java.lang.Object> tuple2(); 
    Code: 
       0: new           #36  // class scala/Tuple2$mcID$sp 
       3: dup 
       4: iconst_1 
       5: ldc2_w        #37  // double 2.0d 
       8: invokespecial #41  // Method scala/Tuple2$mcID$sp."<init>":(ID)V 
      11: areturn 

```

这个字节码显示编译器将括号元组定义语法反编译为名为 `Tuple2` 的类的分配。为每个支持的元组长度（例如，`Tuple5` 支持五个成员）定义了一个元组类，直到 `Tuple22`。字节码还显示在第 `4` 和 `5` 条指令中使用了 `Int` 和 `Double` 的原始版本来分配这个 `tuple` 实例。

## 性能考虑

在前面的示例中，`Tuple2` 由于对两个泛型类型的特殊化处理，避免了原始数据类型的装箱。由于 Scala 的表达式元组语法，将多个值组合在一起通常很方便。然而，这会导致过度的内存分配，因为大于两个参数的元组没有进行特殊化。以下是一个说明这一问题的示例：

```java
def tuple3: (Int, Double, Int) = (1, 2.0, 3) 

```

这个定义与我们所审查的第一个元组定义类似，但现在有一个三个参数的参数数量。这个定义产生了以下字节码：

```java
  public scala.Tuple3<java.lang.Object, java.lang.Object, java.lang.Object> tuple3(); 
    Code: 
       0: new           #45  // class scala/Tuple3 
       3: dup 
       4: iconst_1 
       5: invokestatic  #24  // Method scala/runtime/BoxesRunTime.boxToInteger:(I)Ljava/lang/Integer; 
       8: ldc2_w        #37  // double 2.0d 
      11: invokestatic  #49  // Method scala/runtime/BoxesRunTime.boxToDouble:(D)Ljava/lang/Double; 
      14: iconst_3 
      15: invokestatic  #24  // Method scala/runtime/BoxesRunTime.boxToInteger:(I)Ljava/lang/Integer; 
      18: invokespecial #52  // Method scala/Tuple3."<init>":(Ljava/lang/Object;Ljava/lang/Object;Ljava/lang/Object;)V 
      21: areturn 

```

在这个字节码中，由于整数和双精度浮点数的装箱存在，可以清楚地看到没有特殊化。如果你正在处理应用程序中性能敏感的区域，并且发现存在三个或更多参数的元组，你应该考虑定义一个案例类以避免装箱开销。你的案例类的定义将不包含任何泛型。这使得 JVM 能够使用原始数据类型而不是在堆上为原始元组成员分配对象。

即使使用 `Tuple2`，仍然可能你正在承担装箱的成本。考虑以下代码片段：

```java
case class Bar(value: Int) extends AnyVal 
def tuple2Boxed: (Int, Bar) = (1, Bar(2)) 

```

根据我们对 Tuple2 和值类字节码表示的了解，我们预计这个方法的字节码应该是两个栈分配的整数。不幸的是，在这种情况下，生成的字节码如下：

```java
  public scala.Tuple2<java.lang.Object, highperfscala.patternmatch.PatternMatching$Bar> tuple2Boxed(); 
    Code: 
       0: new           #18  // class scala/Tuple2 
       3: dup 
       4: iconst_1 
       5: invokestatic  #24  // Method scala/runtime/BoxesRunTime.boxToInteger:(I)Ljava/lang/Integer; 
       8: new           #26  // class highperfscala.patternmatch/PatternMatching$Bar 
      11: dup 
      12: iconst_2 
      13: invokespecial #29  // Method highperfscala.patternmatch/PatternMatching$Bar."<init>":(I)V 
      16: invokespecial #32  // Method scala/Tuple2."<init>":(Ljava/lang/Object;Ljava/lang/Object;)V 
      19: areturn 

```

在前面的字节码中，我们看到整数被装箱，并且实例化了 Bar。这个例子与我们在 `Container2` 中调查的最终特殊化示例类似。回顾那个例子，应该很明显，`Container2` 是 `Tuple2` 的一个紧密的类似物。与之前一样，由于编译器实现特殊化的方式，编译器无法避免在这个场景中的装箱。如果你面临性能敏感的代码，解决方案仍然是定义一个案例类。以下是一个证明定义案例类可以消除不希望的值类实例化和原始数据装箱的例子：

```java
 case class IntBar(i: Int, b: Bar) 
 def intBar: IntBar = IntBar(1, Bar(2)) 

```

这个定义产生了以下字节码：

```java
  public highperfscala.patternmatch.PatternMatching$IntBar intBar(); 
    Code: 
       0: new           #18  // class highperfscala.patternmatch/PatternMatching$IntBar 
       3: dup 
       4: iconst_1 
       5: iconst_2 
       6: invokespecial #21  // Method highperfscala.patternmatch/PatternMatching$IntBar."<init>":(II)V 
       9: areturn 

```

注意，`IntBar` 不是一个值类，因为它有两个参数。与元组定义不同，这里既没有装箱也没有对 `Bar` 值类的引用。在这种情况下，定义一个案例类对于性能敏感的代码来说是一个性能上的胜利。

# 模式匹配

对于 Scala 新手程序员来说，模式匹配通常是语言特性中最容易理解的之一，但它也开启了以新的方式思考编写软件的方法。这个强大的机制允许你使用优雅的语法在编译时安全地对不同类型进行匹配。鉴于这种技术对于函数式编程范式中的 Scala 编写至关重要，考虑其运行时开销是很重要的。

## 字节码表示

让我们考虑一个涉及使用表示订单可能方向的代数数据类型进行订单处理的示例：

```java
  sealed trait Side 
  case object Buy extends Side 
  case object Sell extends Side 
  def handleOrder(s: Side): Boolean = s match { 
    case Buy => true 
    case Sell => false 
  } 

```

### 注意

术语**代数数据类型**（**ADT**）是更正式地指代一个密封特性和其情况的方式。例如，`Side`、`Buy`和`Sell`形成一个 ADT。就我们的目的而言，一个 ADT 定义了一个封闭的情况集。对于`Side`，封装的情况是`Buy`和`Sell`。密封修饰符提供了封闭集语义，因为它禁止在单独的源文件中扩展`Side`。ADT 隐含的封闭集语义是允许编译器推断模式匹配语句是否完备的原因。如果您想研究 ADT 的另一个示例，请查看第二章中定义的订单簿命令，*测量 JVM 上的性能*。

如以下字节码所示，模式匹配被转换为一系列 if 语句：

```java
 public boolean handleOrder(highperfscala.patternmatch.PatternMatching$Side); 
    Code: 
       0: aload_1 
       1: astore_2 
       2: getstatic     #148  // Field highperfscala.patternmatch/PatternMatching$Buy$.MODULE$:Lhighperfscala.patternmatch/PatternMatching$Buy$; 
       5: aload_2 
       6: invokevirtual #152  // Method java/lang/Object.equals:(Ljava/lang/Object;)Z 
       9: ifeq          17 
      12: iconst_1 
      13: istore_3 
      14: goto          29 
      17: getstatic     #157  // Field highperfscala.patternmatch/PatternMatching$Sell$.MODULE$:Lhighperfscala.patternmatch/PatternMatching$Sell$; 
      20: aload_2 
      21: invokevirtual #152  // Method java/lang/Object.equals:(Ljava/lang/Object;)Z 
      24: ifeq          31 
      27: iconst_0 
      28: istore_3 
      29: iload_3 
      30: ireturn 
      31: new           #159  // class scala/MatchError 
      34: dup 
      35: aload_2 
      36: invokespecial #160  // Method scala/MatchError."<init>":(Ljava/lang/Object;)V 
      39: athrow 

```

检查字节码可以展示 Scala 编译器如何将模式匹配表达式转换为一系列高效的 if 语句，其中包含`ifeq`指令，位于`9`和`24`索引处。这是一个说明 Scala 如何提供表达性和优雅的一等语言特性，同时保留高效字节码等价的示例。

## 性能考虑

对包含状态（例如，一个 case 类）的值进行模式匹配会带来额外的运行时成本，这在查看 Scala 源代码时并不立即明显。考虑以下对先前示例的扩展，它引入了状态：

```java
  sealed trait Order 
  case class BuyOrder(price: Double) extends Order 
  case class SellOrder(price: Double) extends Order 
  def handleOrder(o: Order): Boolean = o match { 
    case BuyOrder(price) if price > 2.0 => true 
    case BuyOrder(_) => false 
    case SellOrder(_) => false 
  } 

```

在这里，示例更加复杂，因为必须为所有三个情况识别实例类型，并在第一种情况中增加了对`BuyOrder`价格的谓词复杂性。在下面，我们将查看移除了所有 Scala 特定特征的`scalac`输出片段：

```java
        case10(){ 
          if (x1.$isInstanceOf[highperfscala.patternmatch.PatternMatching$BuyOrder]()) 
            { 
              rc8 = true; 
              x2 = (x1.$asInstanceOf[highperfscala.patternmatch.PatternMatching$BuyOrder](): highperfscala.patternmatch.PatternMatching$BuyOrder); 
              { 
                val price: Double = x2.price(); 
                if (price.>(2.0)) 
                  matchEnd9(true) 
                else 
                  case11() 
              } 
            } 
          else 
            case11() 
        }; 
        case11(){ 
          if (rc8) 
            matchEnd9(false) 
          else 
            case12() 
        }; 

```

这种转换说明了关于 Scala 编译器的几个有趣点。识别`Order`的类型利用了`java.lang.Object`中的`isInstanceOf`，它映射到`instanceOf`字节码指令。通过`asInstanceOf`进行类型转换将`Order`强制转换为`BuyOrder`价格或`SellOrder`。第一个启示是，携带状态的模式匹配类型会带来类型检查和转换的运行时成本。

另一个洞察是，Scala 编译器能够通过创建一个名为`rc8`的布尔变量来优化第二个模式匹配的实例检查，以确定是否发现了`BuyOrder`。这种巧妙的优化简单易写，但它去除了模式匹配的优雅和简单性。这是编译器能够从表达性、高级代码中产生高效字节码的另一个示例。

从前面的例子中，现在可以清楚地看出模式匹配被编译成 if 语句。对于关键路径代码的一个性能考虑是模式匹配语句的顺序。如果你的代码有五个模式匹配语句，并且第五个模式是最频繁访问的，那么你的代码正在为始终评估其他四个分支付出代价。让我们设计一个 JMH 微基准测试来估计模式匹配的线性访问成本。每个基准定义了十个使用不同值（例如，值类、整数字面量、case 类等）的模式匹配。对于每个基准，匹配的索引被遍历以显示访问第一个、第五个和第十个模式匹配语句的成本。以下是基准定义：

```java
class PatternMatchingBenchmarks { 

  @Benchmark 
  def matchIntLiterals(i: PatternMatchState): Int = i.matchIndex match { 
    case 1 => 1 
    case 2 => 2 
    case 3 => 3 
    case 4 => 4 
    case 5 => 5 
    case 6 => 6 
    case 7 => 7 
    case 8 => 8 
    case 9 => 9 
    case 10 => 10 
  } 

  @Benchmark 
  def matchIntVariables(ii: PatternMatchState): Int = ii.matchIndex match { 
    case `a` => 1 
    case `b` => 2 
    case `c` => 3 
    case `d` => 4 
    case `e` => 5 
    case `f` => 6 
    case `g` => 7 
    case `h` => 8 
    case `i` => 9 
    case `j` => 10 
  } 

  @Benchmark 
  def matchAnyVal(i: PatternMatchState): Int = CheapFoo(i.matchIndex) match { 
    case CheapFoo(1) => 1 
    case CheapFoo(2) => 2 
    case CheapFoo(3) => 3 
    case CheapFoo(4) => 4 
    case CheapFoo(5) => 5 
    case CheapFoo(6) => 6 
    case CheapFoo(7) => 7 
    case CheapFoo(8) => 8 
    case CheapFoo(9) => 9 
    case CheapFoo(10) => 10 
  } 

  @Benchmark 
  def matchCaseClass(i: PatternMatchState): Int = 
    ExpensiveFoo(i.matchIndex) match { 
      case ExpensiveFoo(1) => 1 
      case ExpensiveFoo(2) => 2 
      case ExpensiveFoo(3) => 3 
      case ExpensiveFoo(4) => 4 
      case ExpensiveFoo(5) => 5 
      case ExpensiveFoo(6) => 6 
      case ExpensiveFoo(7) => 7 
      case ExpensiveFoo(8) => 8 
      case ExpensiveFoo(9) => 9 
      case ExpensiveFoo(10) => 10 
    } 
} 

object PatternMatchingBenchmarks { 

  case class CheapFoo(value: Int) extends AnyVal 
  case class ExpensiveFoo(value: Int) 

  private val (a, b, c, d, e, f, g, h, i, j) = (1, 2, 3, 4, 5, 6, 7, 8, 9, 10) 

  @State(Scope.Benchmark) 
  class PatternMatchState { 
    @Param(Array("1", "5", "10")) 
    var matchIndex: Int = 0 
  } 
} 

```

通过运行 30 次试验，每次持续 10 秒，并包含三个预热试验，每个试验持续 5 秒来评估性能。以下是基准调用：

```java
sbt 'project chapter3' 'jmh:run PatternMatchingBenchmarks -foe true'

```

结果总结在下表中：

| **基准测试** | **匹配的索引** | **吞吐量（每秒操作数）** | **吞吐量误差（吞吐量的百分比）** | **吞吐量变化（基准运行的百分比）** |
| --- | --- | --- | --- | --- |
| `matchAnyVal` | 1 | 350,568,900.12 | ±3.02 | 0 |
| `matchAnyVal` | 5 | 291,126,287.45 | ±2.63 | -17 |
| `matchAnyVal` | 10 | 238,326,567.59 | ±2.95 | -32 |
| `matchCaseClass` | 1 | 356,567,498.69 | ±3.66 | 0 |
| `matchCaseClass` | 5 | 287,597,483.22 | ±3.50 | -19 |
| `matchCaseClass` | 10 | 234,989,504.60 | ±2.60 | -34 |
| `matchIntLiterals` | 1 | 304,242,630.15 | ±2.95 | 0 |
| `matchIntLiterals` | 5 | 314,588,776.07 | ±3.70 | 3 |
| `matchIntLiterals` | 10 | 285,227,574.79 | ±4.33 | -6 |
| `matchIntVariables` | 1 | 332,377,617.36 | ±3.28 | 0 |
| `matchIntVariables` | 5 | 263,835,356.53 | ±6.53 | -21 |
| `matchIntVariables` | 10 | 170,460,049.63 | ±4.20 | -49 |

最后的列取每个基准测试在匹配第一个索引时的第一次尝试作为基准情况。对于匹配第五和第十个索引的尝试，显示相对性能下降。在所有情况下，除了匹配字面整数的第五个索引外，随着匹配更深层的索引，吞吐量几乎线性下降。唯一违反这一模式的是匹配字面整数的尝试。在这个尝试中，当访问第五个索引时，性能相对于第一个索引有所提高。通过检查字节码，我们发现这种情况产生了一个跳转表而不是一系列 if 语句。以下是生成的字节码片段：

```java
       6: tableswitch   { // 1 to 10 
                     1: 113 
                     2: 109 
                     3: 105 
                     4: 101 
                     5: 97 
                     6: 92 
                     7: 87 
                     8: 82 
                     9: 77 
                    10: 72 
               default: 60 
          } 

```

这个字节码片段展示了 JVM 如何使用 `tableswitch` 指令将整数字面量的模式匹配转换为跳转表。这是一个常数时间操作，而不是 if 语句的线性遍历。鉴于观察到的错误是几个百分点，并且三次试验中的观察差异大致也是几个百分点，我们可以推断线性访问成本不适用于这种情况。相反，在 N^(th) 索引处匹配字面整数由于生成的跳转表而具有常数访问成本。相比之下，在第十个索引处匹配整数变量证明要贵近一倍。从这个实验中可以清楚地看出，对于任何生成一系列 if 语句的模式匹配，访问 N^(th) 模式匹配语句的成本是线性的。如果你在性能敏感的代码中至少匹配了三个案例，请考虑审查代码以确定语句顺序是否与访问频率相匹配。

### 注意

你有只包含两个模式的模式匹配的例子吗？在只涉及两个直接匹配值的模式匹配语句的场景中，编译器能够生成一个高效的跳转表。当匹配原始字面量（例如，字符串字面量或整数字面量）时，编译器能够为更大的模式匹配生成跳转表。类似于 `@tailrec` 注解，Scala 定义了一个 `@switch` 注解，以便你告知编译器你期望这个模式匹配语句被编译成跳转表。如果编译器无法生成跳转表，而是生成一系列 if 语句，那么将发出警告。类似于 `@tailrec` 注解，编译器将应用跳转表启发式方法，无论你是否提供了 `@switch` 注解。在实践中，我们很少使用这个注解，因为它适用性有限，但了解其存在是有价值的。以下是一个编译成跳转表的注解模式匹配的例子：

```java
def processShareCount(sc: ShareCount): Boolean = 
 (sc: @switch) match { 
 case ShareCount(1) => true 
 case _ => false 
}
```

# 尾递归

当一个函数调用自身时，我们称其为递归。递归是一个强大的工具，它通常用于函数式编程。它允许你将复杂问题分解成更小的子问题，使它们更容易推理和解决。递归也与不可变性概念很好地结合。递归函数为我们提供了一种管理状态变化的好方法，而不需要使用可变结构或可重新分配的变量。在本节中，我们关注在 JVM 上使用递归的不同缺点，尤其是在 Scala 中。

让我们看看一个递归方法的简单例子。以下片段显示了一个 `sum` 方法，用于计算整数列表的总和：

```java
def sum(l: List[Int]): Int = l match { 
  case Nil => 0 
  case x :: xs => x + sum(xs) 
} 

```

前面的代码片段中展示的`sum`方法执行的是所谓的头递归。`sum(xs)`递归调用不是函数中的最后一条指令。这个方法需要递归调用的结果来计算自己的结果。考虑以下调用：

```java
sum(List(1,2,3,4,5))
```

它可以表示为：

```java
1 + (sum(List(2,3,4,5))) 
1 + (2 + (sum(List(3,4,5)))) 
1 + (2 + (3 + (sum(List(4,5))))) 
1 + (2 + (3 + (4 + (sum(List(5)))))) 
1 + (2 + (3 + (4 + (5)))) 
1 + (2 + (3 + (9))) 
1 + (2 + (12)) 
1 + (14) 
15 

```

注意每次我们执行递归调用时，我们的函数都会挂起，等待计算的正确部分完成以便返回。由于调用函数需要在收到递归调用的结果后完成自己的计算，因此每个调用都会在栈上添加一个新的条目。栈的大小是有限的，没有任何东西可以阻止我们用一个非常长的列表调用`sum`。如果列表足够长，对`sum`的调用将导致`StackOverflowError`：

```java
 $ sbt 'project chapter3' console
    scala> highperfscala.tailrec.TailRecursion.sum((1 to 1000000).toList)
    java.lang.StackOverflowError
      at scala.collection.immutable.Nil$.equals(List.scala:424)
      at highperfscala.tailrec.TailRecursion$.sum(TailRecursion.scala:12)
      at highperfscala.tailrec.TailRecursion$.sum(TailRecursion.scala:13)
      at highperfscala.tailrec.TailRecursion$.sum(TailRecursion.scala:13)
      at highperfscala.tailrec.TailRecursion$.sum(TailRecursion.scala:13)
    ...omitted for brevity 
```

栈跟踪显示了所有递归调用堆积在栈上，等待从后续步骤的结果。这证明了在完成递归调用之前，对`sum`的所有调用都无法完成。在最后一个调用能够执行之前，我们的栈空间已经耗尽。

为了避免这个问题，我们需要重构我们的方法使其成为尾递归。如果一个递归调用是最后一条执行的指令，那么这个递归方法被称为尾递归。尾递归方法可以被优化，将一系列递归调用转换为类似于`while`循环的东西。这意味着只有第一个调用被添加到栈上：

```java
def tailrecSum(l: List[Int]): Int = { 
  def loop(list: List[Int], acc: Int): Int = list match { 
    case Nil => acc 
    case x :: xs => loop(xs, acc + x) 
  } 
  loop(l, 0) 
} 

```

这个`sum`的新版本是尾递归的。注意我们创建了一个内部`loop`方法，它接受要加和的列表以及一个累加器来计算当前结果的状态。`loop`方法是因为递归`loop(xs, acc+x)`调用是最后一条指令，所以是尾递归的。通过在迭代过程中计算累加器，我们避免了递归调用的堆栈。初始累加器值如下所示：

```java
 scala> highperfscala.tailrec.TailRecursion.tailrecSum((1 to 1000000).toList)
    res0: Int = 1784293664 
```

### 注意

我们提到递归是函数式编程的一个重要方面。然而，在实践中，你很少需要自己编写递归方法，尤其是在处理`List`等集合时。标准 API 已经提供了优化的方法，应该优先使用。例如，计算整数列表的总和可以写成如下：

`list.foldLeft(0)((acc, x) => acc + x)` 或者当利用 Scala 糖语法时，我们可以使用以下代码：

`list.foldLeft(0)(+)` 内部实现的`foldLeft`函数使用了一个`while`循环，不会引发`StackOverflowError`异常。

实际上，`List`有一个`sum`方法，这使得计算整数列表的总和变得更加容易。`sum`方法是用`foldLeft`实现的，与前面的代码类似。

## 字节码表示

实际上，JVM 不支持尾递归优化。为了让这可行，Scala 编译器在编译时优化尾递归方法，并将它们转换为`while`循环。让我们比较每个实现生成的字节码。

我们原始的、头递归的`sum`方法编译成了以下字节码：

```java
public int sum(scala.collection.immutable.List<java.lang.Object>); 
    Code: 
       0: aload_1 
// omitted for brevity        
      52: invokevirtual #41  // Method sum:(Lscala/collection/immutable/List;)I 
      55: iadd 
      56: istore_3 
      57: iload_3 
      58: ireturn 
// omitted for brevity 

```

而尾递归的`loop`方法产生了以下结果：

```java
  private int loop(scala.collection.immutable.List<java.lang.Object>, int); 
    Code: 
       0: aload_1 
    // omitted for brevity 
      60: goto          0 
   // omitted for brevity 

```

注意`sum`方法如何在`52`索引处使用`invokevirtual`指令调用自身，并且仍然需要执行一些与返回值相关的指令。相反，`loop`方法在`60`索引处使用`goto`指令跳回到其块的开始，从而避免了多次递归调用自身。

## 性能考虑

编译器只能优化简单的尾递归情况。具体来说，只有那些递归调用是最后一条指令的自调用函数。有许多边缘情况可以描述为尾递归，但它们对于编译器来说过于复杂，无法优化。为了避免无意中编写一个不可优化的递归方法，你应该始终使用`@tailrec`来注释你的尾递归方法。`@tailrec`注释是一种告诉编译器的方式：“我相信你将能够优化这个递归方法；然而，如果你不能，请在编译时给我一个错误。”需要记住的一点是，`@tailrec`并不是要求编译器优化方法，如果可能，它仍然会这样做。这个注释是为了让开发者确保编译器可以优化递归。

### 注意

到目前为止，你应该意识到所有`while`循环都可以在不损失性能的情况下替换为尾递归方法。如果你在 Scala 中使用过`while`循环结构，你可以思考如何用尾递归实现来替换它们。尾递归消除了对可变变量的使用。

这里是带有`@tailrec`注释的相同`tailrecSum`方法：

```java
def tailrecSum(l: List[Int]): Int = { 
  @tailrec 
  def loop(list: List[Int], acc: Int): Int = list match { 
    case Nil => acc 
    case x :: xs => loop(xs, acc + x) 
  } 
  loop(l, 0) 
} 

```

如果我们尝试注释我们的第一个、头递归的实现，我们会在编译时看到以下错误：

```java
 [error] chapter3/src/main/scala/highperfscala/tailrec/TailRecursion.scala:12: could not optimize @tailrec annotated method sum: it contains a recursive call not in tail position
    [error]   def sum(l: List[Int]): Int = l match {
    [error]                                ^
    [error] one error found
    [error] (chapter3/compile:compileIncremental) Compilation failed 
```

我们建议始终使用`@tailrec`来确保你的方法可以被编译器优化。因为编译器只能优化尾递归的简单情况，所以在编译时确保你没有无意中编写一个可能引起`StackOverflowError`异常的非可优化函数是很重要的。我们现在来看几个编译器无法优化递归方法的案例：

```java
def sum2(l: List[Int]): Int = { 

 def loop(list: List[Int], acc: Int): Int = list match { 
   case Nil => acc 
   case x :: xs => info(xs, acc + x) 
 } 
 def info(list: List[Int], acc: Int): Int = { 
   println(s"${list.size} elements to examine. sum so far: $acc") 
   loop(list, acc) 
 } 
 loop(l, 0) 
} 

```

`sum2`中的`loop`方法无法优化，因为递归涉及到两个不同的方法相互调用。如果我们用`info`的实际实现替换调用，那么优化将是可能的，如下所示：

```java
def tailrecSum2(l: List[Int]): Int = { 
  @tailrec 
  def loop(list: List[Int], acc: Int): Int = list match { 
    case Nil => acc 
    case x :: xs => 
     println(s"${list.size} elements to examine. sum so far: $acc") 
     loop(list, acc) 
 } 

 loop(l, 0) 
} 

```

有一种类似的使用案例涉及到编译器无法考虑按名传递的参数：

```java
def sumFromReader(br: BufferedReader): Int = { 
 def read(acc: Int, reader: BufferedReader): Int = { 
   Option(reader.readLine().toInt) 
     .fold(acc)(i => read(acc + i, reader)) 
 } 
 read(0, br) 
} 

```

`read`方法无法被编译器优化，因为它无法使用`Option.fold`的定义来理解递归调用实际上是尾位置的。如果我们用其确切实现替换对 fold 的调用，我们可以按如下方式标注该方法：

```java
def tailrecSumFromReader(br: BufferedReader): Int = { 
  @tailrec 
  def read(acc: Int, reader: BufferedReader): Int = { 
    val opt = Option(reader.readLine().toInt) 
    if (opt.isEmpty) acc else read(acc + opt.get, reader) 
  } 
  read(0, br) 
} 

```

编译器也会拒绝优化非 final 的公共方法。这是为了防止子类用非尾递归版本覆盖方法的风险。从超类发出的递归调用可能通过子类的实现，并破坏尾递归：

```java
class Printer(msg: String) { 
 def printMessageNTimes(n: Int): Unit = { 
   if(n > 0){ 
     println(msg) 
     printMessageNTimes(n - 1) 
   } 
 } 
} 

```

尝试将`printMessageNTimes`方法标记为尾递归会导致以下错误：

```java
 [error] chapter3/src/main/scala/highperfscala/tailrec/TailRecursion.scala:74: could not optimize @tailrec annotated method printMessageNTimes: it is neither private nor final so can be overridden
    [error]     def printMessageNTimes(n: Int): Unit = {
    [error]         ^
    [error] one error found
    [error] (chapter3/compile:compileIncremental) Compilation failed 
```

递归方法无法优化的另一个案例是当递归调用是 try/catch 块的一部分时：

```java
def tryCatchBlock(l: List[Int]): Int = { 
 def loop(list: List[Int], acc: Int): Int = list match { 
   case Nil => acc 
   case x :: xs => 
     try { 
       loop(xs, acc + x) 
     } catch { 
       case e: IOException => 
         println(s"Recursion got interrupted by exception") 
         acc 
     } 
 } 

 loop(l, 0) 
} 

```

与先前的示例相反，在这个例子中，编译器不应受到责备。递归调用不在尾位置。由于它被 try/catch 包围，该方法需要准备好接收潜在的异常并执行更多计算来处理它。作为证明，我们可以查看生成的字节码并观察到最后几条指令与 try/catch 相关：

```java
private final int loop$4(scala.collection.immutable.List, int); 
    Code: 
       0: aload_1 
      // omitted for brevity       
      61: new           #43  // class scala/MatchError 
      64: dup 
      65: aload_3 
      66: invokespecial #46  // Method scala/MatchError."<init>":(Ljava/lang/Object;)V 
      69: athrow 
      // omitted for brevity 
      114: ireturn 
    Exception table: 
       from    to  target type 
          48    61    70   Class java/io/IOException 

```

我们希望这些少数示例已经说服你，编写非尾递归方法是容易犯的一个错误。你最好的防御方法是始终使用`@tailrec`注解来验证你的直觉，即你的方法可以被优化。

# `Option`数据类型

`Option`数据类型在 Scala 标准库中被广泛使用。像模式匹配一样，它是 Scala 初学者早期经常采用的语言特性。`Option`数据类型提供了一种优雅的方式来转换和处理不需要的值，从而消除了 Java 代码中常见的 null 检查。我们假设你理解并欣赏`Option`为在函数式范式编写 Scala 带来的价值，因此我们不会进一步重申其好处。相反，我们专注于分析其字节码表示，以获得性能洞察。

## 字节码表示

检查 Scala 源代码，我们看到`Option`被实现为一个抽象类，其中包含可能的输出结果`Some`和`None`，它们扩展了`Option`以编码这种关系。以下代码片段中显示了移除实现后的类定义，以方便查看：

```java
sealed abstract class Option[+A] extends Product with Serializable 
final case class Some+A extends Option[A] 
case object None extends Option[Nothing] 

```

研究定义后，我们可以推断出关于字节码表示的几个要点。关注`Some`，我们注意到它没有扩展`AnyVal`。由于`Option`是通过继承实现的，因此`Some`不能是一个值类，这是我们在值类部分提到的限制。这种限制意味着每个作为`Some`实例包装的值都有一个分配。此外，我们观察到`Some`没有被专门化。从我们对专门化的考察中，我们意识到作为`Some`实例包装的原始数据将被装箱。以下是一个简单示例，用于说明这两个问题：

```java
def optionalInt(i: Int): Option[Int] = Some(i) 

```

在这个简单的例子中，一个整数被编码为一个`Some`实例，用作`Option`数据类型。以下字节码被生成：

```java
  public scala.Option<java.lang.Object> optionalInt(int); 
    Code: 
       0: new           #16  // class scala/Some 
       3: dup 
       4: iload_1 
       5: invokestatic  #22  // Method scala/runtime/BoxesRunTime.boxToInteger:(I)Ljava/lang/Integer; 
       8: invokespecial #25  // Method scala/Some."<init>":(Ljava/lang/Object;)V 
      11: areturn 

```

如我们所预期，有一个对象分配来创建一个`Some`实例，然后是将提供的整数装箱以构建`Some`实例。

`None`实例是一个从字节码角度更容易理解的简单案例。因为`None`被定义为 Scala 对象，所以创建`None`实例没有实例化成本。这很有意义，因为`None`代表没有状态需要维护的场景。

### 注意

你是否曾经考虑过单个值`None`如何代表所有类型的无值？答案在于理解`Nothing`类型。`Nothing`类型扩展了所有其他类型，这使得`None`可以成为任何`A`类型的子类型。要了解更多关于 Scala 类型层次结构的见解，请查看这个有用的 Scala 语言教程：[`docs.scala-lang.org/tutorials/tour/unified-types.html`](http://docs.scala-lang.org/tutorials/tour/unified-types.html)。

## 性能考虑

在任何非性能敏感的环境中，默认使用`Option`来表示不需要的值是合理的。在性能敏感的代码区域，选择变得更加具有挑战性且不那么明确。特别是在性能敏感的代码中，你必须首先优化正确性，然后才是性能。我们建议始终以最地道的方式实现你正在建模的问题的第一个版本，也就是说，使用`Option`。从`Some`的字节码表示中获得的意识，下一步的逻辑是进行性能分析，以确定是否`Option`的使用是瓶颈。特别是，你关注的是内存分配模式和垃圾回收成本。根据我们的经验，代码中通常存在其他开销来源，这些开销比`Option`的使用更昂贵。例如，不高效的算法实现、构建不良的领域模型或系统资源使用不当。如果你的情况下已经消除了其他效率低下的来源，并且确信`Option`是导致你性能问题的原因，那么你需要采取进一步的步骤。

向提高性能的增量步骤可能包括移除使用`Option`高阶函数。在关键路径上，通过用内联等价物替换高阶函数可以实现显著的成本节约。考虑以下简单的例子，它将`Option`数据类型转换为`String`数据类型：

```java
Option(10).fold("no value")(i => s"value is $i") 

```

在关键路径上，以下变更可能会带来实质性的改进：

```java
val o = Option(10) 
if (o.isDefined) s"value is ${o.get} else "no value" 

```

将`fold`操作替换为 if 语句可以节省创建匿名函数的成本。需要重复强调的是，这种类型的更改只有在经过广泛的性能分析并发现`Option`使用是瓶颈之后才应考虑。虽然这种代码更改可能会提高你的性能，但由于使用了`o.get`，它既冗长又不安全。当这种技术被谨慎使用时，你可能会在关键路径代码中保留对`Option`数据类型的使用。

如果用内联和不安全的等效函数替换高阶`Option`函数使用未能充分提高性能，那么你需要考虑更激进的措施。此时，性能分析应该揭示`Option`内存分配是瓶颈，阻止你达到性能目标。面对这种场景，你有两种选择（这并非巧合！）进行探索，这两种选择都涉及实施时间的高成本。

一种进行的方式是承认，对于关键路径，`Option`是不合适的，必须从类型签名中移除，并用 null 检查替换。这是最有效的方法，但它带来了显著的维护成本，因为你和所有其他在关键路径上工作的团队成员都必须意识到这个建模决策。如果你选择这种方式进行，为关键路径定义清晰的边界，将 null 检查限制在代码的最小可能区域。在下一节中，我们将探讨第二种方法，该方法涉及构建一个新的数据类型，该类型利用了我们在本章中获得的知识。

# 案例研究 – 更高性能的选项

如果你还没有准备好丢失由`Option`数据类型编码的信息，那么你可能希望探索更符合垃圾回收友好的`Option`的替代实现。在本节中，我们介绍了一种替代方法，它也提供了类型安全，同时避免了`Some`实例的装箱和实例化。我们利用标记类型和特化，并禁止`Some`作为有效值，从而得出以下实现：

```java
sealed trait Opt 

object OptOps { 

  def some@specialized A: A @@ Opt = Tag(x) 
  def nullCheckingSome@specialized A: A @@ Opt = 
    if (x == null) sys.error("Null values disallowed") else Tag(x) 
  def none[A]: A @@ Opt = Tag(null.asInstanceOf[A]) 

  def isSomeA: Boolean = o != null 
  def isEmptyA: Boolean = !isSome(o) 

  def unsafeGetA: A = 
    if (isSome(o)) o.asInstanceOf[A] else sys.error("Cannot get None") 

  def foldA, B(ifEmpty: => B)(f: A => B): B = 
    if (o == null) ifEmpty else f(o.asInstanceOf[A]) 
} 

```

此实现定义了用于构建可选类型的工厂方法（即`some`、`nullCheckingSome`和`none`）。与 Scala 的`Option`相比，此实现使用标记类型向值添加类型信息，而不是创建一个新值来编码可选性。`none`的实现利用了 Scala 中将`null`视为值而不是语言关键字的事实。记住，除非性能要求需要这种极端措施，否则我们不会默认采用这些更神秘的方法。每个工厂方法返回的标记类型保留了类型安全，并且需要显式解包才能访问底层类型。

### 注意

如果你想了解更多关于 Scala 对`null`值表示的信息，我们鼓励你查看这两个 StackOverflow 帖子：[`stackoverflow.com/questions/8285916/why-doesnt-null-asinstanceofint-throw-a-nullpointerexception`](http://stackoverflow.com/questions/8285916/why-doesnt-null-asinstanceofint-throw-a-nullpointerexception) 和 [`stackoverflow.com/questions/10749010/if-an-int-cant-be-null-what-does-null-asinstanceofint-mean`](http://stackoverflow.com/questions/10749010/if-an-int-cant-be-null-what-does-null-asinstanceofint-mean)。在这两个帖子中，多位响应者提供了优秀的回答，这将帮助你深化理解。

`OptOps`中剩余的方法定义了你在 Scala 的`Option`实现中会发现的方法。由于没有通过工厂方法分配新实例，我们看到这些方法是静态的，而不是实例方法。我们有可能定义一个隐式类，它将提供模拟实例方法调用的语法，但我们避免这样做，因为我们假设极端的性能敏感性。从语义上讲，在`OptOps`中定义的操作与 Scala 的`Option`类似。我们不是匹配表示无值的值（即`None`），而是再次利用将`null`作为值的引用能力。

使用此实现，运行时开销包括实例检查和对`scalaz.Tag`的调用。我们失去了模式匹配的能力，而必须要么折叠，或者在极端情况下使用`isSome`和`unsafeGet`。为了更好地理解运行时差异，我们使用 Scala 的`Option`和前面的标记类型实现进行了微基准测试。微基准测试让你对语法的变化有所体会。我们鼓励你运行`javap`来反汇编字节码，以证明这个实现避免了装箱和对象创建：

```java
 class OptionCreationBenchmarks { 

  @Benchmark 
  def scalaSome(): Option[ShareCount] = Some(ShareCount(1)) 

  @Benchmark 
  def scalaNone(): Option[ShareCount] = None 

  @Benchmark 
  def optSome(): ShareCount @@ Opt = OptOps.some(ShareCount(1)) 

  @Benchmark 
  def optSomeWithNullChecking(): ShareCount @@ Opt = 
    OptOps.nullCheckingSome(ShareCount(1)) 

  @Benchmark 
  def optNone(): ShareCount @@ Opt = OptOps.none 

  @Benchmark 
  def optNoneReuse(): ShareCount @@ Opt = noShares 
} 

object OptionCreationBenchmarks { 
  case class ShareCount(value: Long) extends AnyVal 
  val noShares: ShareCount @@ Opt = OptOps.none 
} 

```

我们使用以下熟悉的参数运行测试：

```java
sbt 'project chapter3' 'jmh:run OptionCreationBenchmarks  -foe true'

```

结果总结在下表中：

| **基准** | **吞吐量（每秒操作数）** | **错误率（吞吐量的百分比）** |
| --- | --- | --- |
| `optNone` | 351,536,523.84 | ±0.75 |
| `optNoneReuse` | 344,201,145.90 | ±0.23 |
| `optSome` | 232,684,849.83 | ±0.37 |
| `optSomeWithNullChecking` | 233,432,224.39 | ±0.28 |
| `scalaNone` | 345,826,731.05 | ±0.35 |
| `scalaSome` | 133,583,718.28 | ±0.24 |

在这里最令人印象深刻的结果可能是，当使用`Some`的标记类型实现而不是 Scala 提供的实现时，吞吐量大约提高了 57%。这很可能是由于减少了内存分配压力。我们发现`None`创建的吞吐量在质量上相似。我们还观察到，在标记`Some`选项的构造中添加空检查似乎没有成本。如果你信任你的团队能够避免传递空值，那么这个检查就是多余的。我们还创建了一系列基准测试来评估折叠性能，以了解使用这种替代`Option`实现的相对成本。以下是简单折叠基准测试的源代码：

```java
class OptionFoldingBenchmarks { 

  @Benchmark 
  def scalaOption(): ShareCount = 
    scalaSome.fold(ShareCount(0))(c => ShareCount(c.value * 2)) 

  @Benchmark 
  def optOption(): ShareCount = 
    OptOps.fold(optSome)(ShareCount(0))(c => ShareCount(c.value * 2)) 

} 

object OptionFoldingBenchmarks { 

  case class ShareCount(value: Long) extends AnyVal 

  val scalaSome: Option[ShareCount] = Some(ShareCount(7)) 
  val optSome: ShareCount @@ Opt = OptOps.some(ShareCount(7)) 
} 

```

这个基准测试使用了之前相同的参数集：

```java
jmh:run OptionFoldingBenchmarks  -foe true

```

以下是测试结果的总结：

| **基准测试** | **吞吐量（每秒操作数）** | **误差作为吞吐量的百分比** |
| --- | --- | --- |
| `optOption` | 346,208,759.51 | ±1.07 |
| `scalaOption` | 306,325,098.74 | ±0.41 |

在这个基准测试中，我们希望证明在使用替代的标记类型启发的实现而不是 Scala `Option`时，没有显著的吞吐量下降。性能的显著下降将危及我们在创建基准测试中找到的性能提升。幸运的是，这个基准测试表明折叠吞吐量实际上比 Scala `Option`折叠实现提高了大约 13%。

### 注意

看到基准测试的结果证实了你的假设，这让人感到欣慰。然而，同样重要的是要理解为什么会产生有利的结果，并且能够解释这一点。如果没有理解这些结果是如何产生的，你很可能无法重现这些结果。你将如何解释标记类型启发的实现相对于 Scala `Option`实现的折叠吞吐量改进？考虑我们讨论的实现和内存分配差异。

基准测试表明，受标记类型启发的`Option`实现相对于 Scala 的`Option`实现带来了定性的性能提升。如果你面临性能问题，并且分析显示 Scala 的`Option`是瓶颈，那么探索这种替代实现可能是有意义的。虽然性能有所提升，但要注意存在权衡。在使用替代实现时，你将失去模式匹配的能力。这看起来是一个微不足道的代价，因为你能够使用折叠操作。更高的代价是集成标准库和第三方库。如果你的关键路径代码与 Scala 标准库或第三方库有大量交互，你将被迫重写大量代码以使用替代的`Option`实现。在这种情况下，如果你处于时间压力之下，重新考虑是否用`null`建模域的部分部分是有意义的。如果你的关键路径代码避免与 Scala 标准库或第三方库有重大交互，那么使用替代的`Option`实现可能是一个更容易的决定。

我们的案例研究受到了 Alexandre Bertails 在他的博客文章[`bertails.org/2015/02/15/abstract-algebraic-data-type/`](https://bertails.org/2015/02/15/abstract-algebraic-data-type/)中探索的一种新颖方法的启发。他通过定义一种他称之为抽象代数数据类型的方法来解决我们处理过的相同性能问题。这两种方法都依赖于使用类型约束来建模`Option`而不进行实例分配。通过抽象`Option`代数数据类型及其操作，他能够设计出一种无分配和无装箱的实现。我们鼓励你探索这种方法，因为它又是如何实现安全性的同时仍然提供优秀性能的一个很好的例子。

# 摘要

在本章中，我们深入探讨了常用 Scala 语言特征的字节码表示和性能考虑。在我们的案例研究中，你亲眼看到了如何结合关于 Scala 语言特征的几个领域的知识，以及出色的 Scalaz 库，以产生更适合高性能需求的`Option`实现。

在我们所有的示例中，一个一致的宗旨是在考虑性能权衡的同时，促进类型安全和正确性。作为函数式程序员，我们重视编译时正确性和引用透明性。即使在标记类型`Option`实现中使用`null`，我们也保留了正确性，因为`null`值是一个内部实现细节。当你反思我们所讨论的策略时，考虑每个策略是如何在保持引用透明（即无副作用）的代码的同时，仍然帮助你达到性能目标的。

在这个阶段，你应该对 Scala 优雅的语言特性带来的权衡更加自信。通过我们的分析，你学习了如何将简洁的 Scala 语法转换为 JVM 字节码。这是一项宝贵的技能，可以帮助调试性能问题。随着你通过研究更多示例来提高自己的意识，你将培养出更强的直觉，了解潜在问题的所在。随着时间的推移，你可以回顾这一章节，以复习常见的补救策略，平衡优雅与安全与性能之间的权衡。在下一章中，我们将通过深入研究集合，继续提升利用 Scala 编写高效、函数式代码的能力。
