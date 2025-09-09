# 第五章。懒惰集合和事件溯源

在上一章中，我们探索了许多可以立即进行贪婪评估的 Scala 集合。Scala 标准库提供了两个操作懒惰的集合：视图和流。为了激发对这些集合的探索，我们将解决 MVT 围绕为客户端生成性能报告的性能困境。在本章中，我们将涵盖以下主题：

+   视图

+   使用两个真实世界应用的流处理

+   事件溯源

+   马尔可夫链生成

# 提高客户报告生成速度

想要了解更多关于 MVT 客户的信息，你决定参加每周的客户状态会议。当你环顾四周时，你看到自己是这里唯一的工程师，其他人都是销售团队的人。MVT 客户管理团队的负责人约翰尼列出了新签约的客户名单。每次他读出一个名字，就会响起一声响亮的铃声。这对你来说似乎是一种奇怪的习俗，但每当铃声响起，销售团队都会兴奋地欢呼。

在新客户名单公布结束，耳边铃声停止后，销售团队中的一名成员问约翰尼：“性能报告何时能生成得更快？客户每天都在打电话给我，抱怨他们无法在交易日看到自己的持仓和盈亏情况。我们没有这种透明度，这很尴尬，我们可能会因此失去业务。”你意识到这个问题报告是一个可以通过 MVT 向客户公开的私有网络门户下载的 PDF 文件。除非客户足够熟练，能够使用 MVT 的性能 API 设置自己的报告，否则客户将依赖于门户来检查最近的交易表现。

意识到这是一个更好地理解问题的机会，你问道：“嗨，我是工程团队的一员。我想今天来了解一下我们的客户。你能分享更多关于报告性能问题的情况吗？我想帮助解决这个问题。”通过与销售团队的交谈，你了解到 PDF 报告是向实时流式 Web 应用迈出的第一步。PDF 报告允许 MVT 快速向客户提供交易表现洞察。每次客户点击“查看表现”时，都会生成一份报告，通过显示客户在过去一小时、一天和七天内是否实现了盈利或亏损来总结表现趋势。尤其是在市场波动时，你了解到客户更有可能生成报告。销售团队认为这加剧了问题，因为当每个人都试图查看最近的交易表现时，报告生成速度会变得更慢。在一些最糟糕的情况下，性能报告需要大约十几分钟才能生成，这对期望近乎实时结果的客户来说是完全不能接受的。

## 深入报告代码

急于深入探究问题，你找到了负责处理报告数据的存储库。你探索领域模型以了解此范围内表示的关注点：

```java
case class Ticker(value: String) extends AnyVal 
case class Price(value: BigDecimal) extends AnyVal 
case class OrderId(value: Long) extends AnyVal 
case class CreatedTimestamp(value: Instant) extends AnyVal 
case class ClientId(value: Long) extends AnyVal 

sealed trait Order { 
  def created: CreatedTimestamp 
  def id: OrderId 
  def ticker: Ticker 
  def price: Price 
  def clientId: ClientId 
} 
case class BuyOrder(created: CreatedTimestamp, id: OrderId, ticker: Ticker, price:   Price, clientId: ClientId) extends Order 

case class SellOrder(created: CreatedTimestamp, id: OrderId, ticker: Ticker, price: Price,clientId: ClientId) extends Order 

case class Execution(created: CreatedTimestamp, id: OrderId, price: Price) 

```

在报告的上下文中，将订单与执行关联起来对于构建性能趋势报告非常重要，因为这种关联使得 MVT 能够识别出从交易中实现的利润或损失。`ClientId`是一个你在处理订单簿或执行数据分析时未曾接触过的概念。客户 ID 用于识别 MVT 客户的账户。由于交易是代表客户执行的，因此客户 ID 使我们能够将已执行的订单与客户账户关联起来。

在扫描代码库时，你发现了性能趋势报告在转换为 PDF 格式之前的表示：

```java
sealed trait LastHourPnL 
case object LastHourPositive extends LastHourPnL 
case object LastHourNegative extends LastHourPnL 

sealed trait LastDayPnL 
case object LastDayPositive extends LastDayPnL 
case object LastDayNegative extends LastDayPnL 

sealed trait LastSevenDayPnL 
case object LastSevenDayPositive extends LastSevenDayPnL 
case object LastSevenDayNegative extends LastSevenDayPnL 

case class TradingPerformanceTrend( 
  ticker: Ticker, 
  lastHour: LastHourPnL, 
  lastDay: LastDayPnL, 
  lastSevenDay: LastSevenDayPnL) 

```

**利润和损失**（**PnL**）趋势由每个支持的时间段对应的不同的 ADT 表示：最后一个小时、最后一天和最后七天。对于每个股票代码，这三个时间段都包含在`TradingPerformanceTrend`中。在多个股票代码中，你可以推断出客户可以确定 MVT 是否在一段时间内产生利润或损失。检查负责计算`TradingPerformanceTrend`的`trend`方法的签名，你可以证实你的想法：

```java
def trend( 
  now: () => Instant, 
  findOrders: (Interval, Ticker) => List[Order], 
  findExecutions: (Interval, Ticker) => List[Execution], 
  request: GenerateTradingPerformanceTrend): 
List[TradingPerformanceTrend] 

case class GenerateTradingPerformanceTrend( 
  tickers: List[Ticker], clientId: ClientId) 

```

计算性能趋势需要一种确定当前时间的方法，以便确定需要回溯多远来计算每个时间段的趋势。`findOrders`和`findExecutions`参数是查询特定股票代码在特定时间间隔内创建的订单和执行的函数。最后一个参数包含客户的 ID 和要报告的股票代码。每个时间段的趋势是通过一个名为`periodPnL`的通用内部方法计算的，其形式如下：

```java
  def periodPnL( 
    duration: Duration): Map[Ticker, PeriodPnL] = { 
    val currentTime = now() 
    val interval = new Interval(currentTime.minus(duration), currentTime) 
    (for { 
      ticker <- request.tickers 
      orders = findOrders(interval, ticker) 
      executions = findExecutions(interval, ticker) 
      idToExecPrice = executions.groupBy(_.id).mapValues(es => 
        Price.average(es.map(_.price))) 
      signedExecutionPrices = for { 
        o <- orders 
        if o.clientId == request.clientId 
        price <- idToExecPrice.get(o.id).map(p => o match { 
          case _: BuyOrder => Price(p.value * -1) 
          case _: SellOrder => p 
        }).toList 
      } yield price 
      trend = signedExecutionPrices.foldLeft(PnL.zero) { 
        case (pnl, p) => PnL(pnl.value + p.value) 
      } match { 
        case p if p.value >= PnL.zero.value => PeriodPositive 
        case _ => PeriodNegative 
      } 
    } yield ticker -> trend).toMap 
  } 

```

`periodPnL`方法是一个包含多个逻辑步骤的复杂方法。对于每个客户提供的股票代码，检索提供的时间段内的相关订单和执行。为了关联订单与执行，使用`groupBy`构建一个`OrderId`到`Execution`的映射。为了简化后续的计算，计算每个已执行订单的平均执行价格，将单个订单的多个执行减少到一个值。

在构建了`idToExecPrice`查找表之后，下一步的逻辑步骤是过滤掉其他客户的订单。一旦只剩下客户的订单，`idToExecution`用于识别已执行的订单。最后两个步骤通过编制客户的绝对回报（即利润和损失）来计算性能趋势。这些步骤涉及领域模型的两个添加，如下所示：

```java
case class PnL(value: BigDecimal) extends AnyVal 
object PnL { 
  val zero: PnL = PnL(BigDecimal(0)) 
} 

sealed trait PeriodPnL 
case object PeriodPositive extends PeriodPnL 
case object PeriodNegative extends PeriodPnL 

```

`PnL`值是一个用于表示客户美元回报的价值类。`PeriodPnL`类似于之前引入的 ADT，可以应用于任何时间段的数据。这使得`PeriodPnL`可以重复用于最后一个小时、最后一天和最后七天的趋势计算。

当交易代表买入时，执行价格会被取反，因为交易代表的是现金交换股票。当交易代表卖出时，执行价格保持正值，因为交易代表的是用股票交换现金。在计算每个股票的性能趋势后，`Ticker`和`PeriodPnL`元组的`List`被转换为`Map`。

理解这个实现的过程，你可以开始想象生成这个 PDF 为什么耗时。没有缓存结果的迹象，这意味着每次客户端发起请求时，趋势报告都会重新计算。随着请求报告的客户数量增加，计算报告时的等待时间也会增加。重新架构报告基础设施以缓存报告是一个过于庞大的短期改变。相反，你尝试识别可以改进报告生成性能的增量变化。

## 使用视图加快报告生成时间

当我们在订单簿上工作时，我们了解到`List`会急切地评估结果。这个特性意味着在`periodPnL`中，对`orders`执行的脱糖 for-comprehension `filter`和`map`操作会产生新的列表。也就是说，每个转换都会产生一个新的集合。对于订单数量大的客户来说，迭代订单集三次可能会在 CPU 时间上造成成本，并且由于重复创建`List`而产生垃圾收集成本。为了缓解这个问题，Scala 提供了一种方法，可以在下游计算需要元素时才延迟转换元素。

从概念上讲，这是通过在急切评估的集合之上添加一个视图来完成的，该视图允许使用延迟评估语义定义转换。可以通过调用`view`从任何 Scala 集合构建集合的懒加载视图。例如，以下片段从一个整数`List`创建了一个视图：

```java
val listView: SeqView[Int, List[Int]] = List(1, 2, 3).view 

```

从这个片段中，我们了解到 Scala 使用一个不同的`SeqView`类型来表示集合的视图，该类型由两种类型参数化：集合元素和集合类型。看到视图的使用使得更容易理解它与急切评估集合的运行时差异。考虑以下片段，它在`List`及其`List`视图上执行相同的操作：

```java
println("List evaluation:") 
val evens = List(0, 1, 2, 3, 4, 5).map(i => { 
  println(s"Adding one to $i") 
  i + 1 
}).filter(i => { 
  println(s"Filtering $i") 
  i % 2 == 0 
}) 

println("--- Printing first two even elements ---") 
println(evens.take(2)) 

println("View evaluation:") 
val evensView = List(0, 1, 2, 3, 4, 5).view.map(i => { 
  println(s"Adding one to $i") 
  i + 1 
}).filter(i => { 
  println(s"Filtering $i") 
  i % 2 == 0 
}) 

println("--- Printing first two even elements ---") 
println(evensView.take(2).toList) 

```

这个片段执行简单的算术运算，然后过滤以找到偶数元素。为了加深我们的理解，这个片段通过添加`println`副作用打破了函数式范式。列表评估的输出符合预期：

```java
List evaluation: 
Adding one to 0 
Adding one to 1 
Adding one to 2 
Adding one to 3 
Adding one to 4 
Adding one to 5 
Filtering 1 
Filtering 2 
Filtering 3 
Filtering 4 
Filtering 5 
Filtering 6 
--- Printing first two even elements --- 
List(2, 4) 

```

在急切评估中，每个转换在移动到下一个转换之前都会应用于每个元素。现在，考虑以下来自视图评估的输出：

```java
View evaluation: 
--- Printing first two even elements --- 
Adding one to 0 
Filtering 1 
Adding one to 1 
Filtering 2 
Adding one to 2 
Filtering 3 
Adding one to 3 
Filtering 4 
List(2, 4) 

```

如我们之前讨论的，使用惰性评估时，只有在需要元素时才会应用转换。在这个例子中，这意味着加法和过滤操作不会在调用`toList`之前发生。在“视图评估”之后没有输出，这表明没有发生任何转换。有趣的是，我们还看到只有六个元素中的前四个被评估。当一个视图应用转换时，它将对每个元素应用所有转换，而不是对每个元素应用每个转换。通过一步应用所有转换，视图能够返回前两个元素，而无需评估整个集合。在这里，我们看到了由于惰性评估而使用视图的潜在性能提升。在将视图的概念应用到性能趋势报告之前，让我们更深入地了解一下视图的实现。

### 构建自定义视图

视图能够通过返回一个组合了先前转换状态和下一个转换的数据结构来延迟评估。Scala 视图的实现确实复杂，因为它提供了大量的功能，同时仍然支持所有 Scala 集合。为了构建对视图实现的理解，让我们构建我们自己的仅适用于`List`且仅支持`map`操作的惰性评估视图。首先，我们定义我们的`PseudoView`视图实现所支持的运算：

```java
sealed trait PseudoView[A] { 
  def mapB: PseudoView[B] 
  def toList: List[A] 
} 

```

`PseudoView`被定义为一种特质，它支持从`A`到`B`的转换的惰性应用，同时也支持评估所有转换以返回一个`List`。接下来，我们定义两种视图类型，以支持零转换已应用时的初始情况，以及支持将转换应用到先前转换过的视图。签名在以下代码片段中显示：

```java
final class InitialViewA extends PseudoView[A] 
final class ComposedViewA, B extends PseudoView[B] 

```

在这两种情况下，原始的`List`必须保留以支持最终应用转换。在`InitialView`的基本情况下，没有转换，这就是为什么没有额外的状态。`ComposedView`通过携带先前`fa`转换的状态来支持链式计算。

实现`InitialView`是一个简单的委托给`ComposedView`：

```java
final class InitialViewA extends PseudoView[A] { 
  def mapB: PseudoView[B] = new ComposedViewA, B 
  def toList: List[A] = xs 
} 

```

`List`实现展示了如何使用函数组合将转换链在一起：

```java
final class ComposedViewA, B extends PseudoView[B] { 
  def mapC: PseudoView[C] = new ComposedView(xs, f.compose(fa)) 
  def toList: List[B] = xs.map(fa) 
} 

```

让我们构建一个`PseudoView`伴生对象，它提供视图构建，如下所示：

```java
object PseudoView {  
  def viewA, B: PseudoView[A] = new InitialView(xs) 
} 

```

我们现在可以通过一个简单的程序来练习`PseudoView`，以证明它延迟了评估：

```java
println("PseudoView evaluation:") 
val listPseudoView = PseudoView.view(List(0, 1, 2)).map(i => { 
  println(s"Adding one to $i") 
  i + 1 
}).map(i => { 
  println(s"Multiplying $i") 
  i * 2 
}) 

println("--- Converting PseudoView to List ---") 
println(listPseudoView.toList) 

```

运行这个程序，我们看到输出与 Scala 视图实现的用法相当：

```java
PseudoView evaluation: 
--- Converting PseudoView to List --- 
Adding one to 0 
Multiplying 1 
Adding one to 1 
Multiplying 2 
Adding one to 2 
Multiplying 3 
List(2, 4, 6) 

```

`PseudoView`有助于建立对 Scala 如何实现视图的直觉。从这里，你可以开始考虑如何支持其他操作。例如，如何实现`filter`？`filter`很有趣，因为它限制了原始集合。如定义，`PseudoView`不适合支持`filter`操作，这是 Scala 视图处理复杂性的一个例子。Scala 视图通过定义一个名为`Transformed`的特质来应对这一挑战。`Transformed`特质是所有视图操作的基础特质。部分定义如下：

```java
trait Transformed[+B] extends GenTraversableView[B, Coll] { 
  def foreachU: Unit 
  lazy val underlying = self.underlying 
} 

```

`underlying`懒值是原始包装集合的访问方式。这与`PseudoView`将`List`状态传递到`ComposedView`的方式类似。`Transformed`定义了一个副作用`foreach`操作来以懒加载方式支持集合操作。使用`foreach`允许实现此特质的实例修改底层集合。这就是`filter`是如何实现的：

```java
trait Filtered extends Transformed[A] { 
  protected[this] val pred: A => Boolean 
  def foreachU { 
    for (x <- self) 
      if (pred(x)) f(x) 
  } 
} 

```

在视图 API 中，`Transformed`用于维护必要操作的状态，而外部 API 支持与`SeqView`交互。遵循在 Scala 集合中常见的另一种模式，`SeqView`通过混合其他特质继承了一组操作。`SeqView`间接混合了`TraversableViewLike`，这提供了对`Transformed`操作的访问。

### 应用视图来提高报告生成性能

通过我们新开发的视图直觉，我们可能（无意中！）以不同的方式看待性能趋势报告的构建。Scala 对视图的实现使得从急切评估的集合切换到懒加载版本变得非常简单。如果你还记得，一旦构建了订单 ID 到平均执行价格查找表，就会对在特定时间段和股票代码下检索到的订单应用一系列转换。通过将`orders`转换为视图，就有机会避免不必要的转换并提高性能趋势报告的速度。

虽然转换为视图很简单，但确定在哪些条件下懒加载评估优于急切评估则不那么简单。作为一名优秀的性能工程师，你希望对你的提议进行基准测试，但你没有访问历史订单和执行数据来构建基准。相反，你编写了一个微基准测试来模拟你正在建模的问题。你试图回答的问题是：“对于什么大小的集合和多少操作，使用视图而不是`List`是有意义的？”构建视图是有成本的，因为它涉及到保留关于延迟转换的信息，这意味着它不总是性能最佳解决方案。你提出了以下场景来帮助你回答问题：

```java
  @Benchmark 
  def singleTransformList(state: ViewState): List[Int] = 
    state.numbers.map(_ * 2) 

  @Benchmark 
  def singleTransformView(state: ViewState): Vector[Int] = 
    state.numbers.view.map(_ * 2).toVector 

  @Benchmark 
  def twoTransformsList(state: ViewState): List[Int] = 
    state.numbers.map(_ * 2).filter(_ % 3 == 0) 

  @Benchmark 
  def twoTransformsView(state: ViewState): Vector[Int] = 
    state.numbers.view.map(_ * 2).filter(_ % 3 == 0).toVector 

  @Benchmark 
  def threeTransformsList(state: ViewState): List[Int] = 
    state.numbers.map(_ * 2).map(_ + 7).filter(_ % 3 == 0) 

  @Benchmark 
  def threeTransformsView(state: ViewState): Vector[Int] = 
    state.numbers.view.map(_ * 2).map(_ + 7).filter(_ % 3 == 0).toVector 

```

对于每种集合类型，一个`List`和一个`Vector`上的视图，你定义了三个测试，以练习不断增加的转换数量。`Vector`被用来代替`List`，因为视图上的`toList`没有针对`List`进行优化。正如我们之前看到的，`List`操作被编写为利用常数时间和预加性能。`toList`执行线性时间追加操作，这给人一种视图提供较低性能的错觉。切换到`Vector`提供了有效的常数时间追加操作。此基准的状态如下所示：

```java
@State(Scope.Benchmark) 
class ViewState { 

  @Param(Array("10", "1000", "1000000")) 
  var collectionSize: Int = 0 

  var numbers: List[Int] = Nil 

  @Setup 
  def setup(): Unit = { 
    numbers = (for (i <- 1 to collectionSize) yield i).toList 
  } 
} 

```

`ViewState`通过遍历不同的集合大小来帮助识别视图性能对集合大小的敏感性。基准通过以下方式调用：

```java
sbt 'project chapter5' 'jmh:run ViewBenchmarks -foe true'

```

此调用产生以下结果：

| **基准测试** | **集合大小** | **吞吐量（每秒操作数）** | **吞吐量误差百分比** |
| --- | --- | --- | --- |
| `singleTransformList` | 10 | 15,171,067.61 | ± 2.46 |
| `singleTransformView` | 10 | 3,175,242.06 | ± 1.37 |
| `singleTransformList` | 1,000 | 133,818.44 | ± 1.58 |
| `singleTransformView` | 1,000 | 52,688.80 | ± 1.11 |
| `singleTransformList` | 1,000,000 | 30.40 | ± 2.72 |
| `singleTransformView` | 1,000,000 | 86.54 | ± 1.17 |
| `twoTransformsList` | 10 | 5,008,830.88 | ± 1.12 |
| `twoTransformsView` | 10 | 4,564,726.04 | ± 1.05 |
| `twoTransformsList` | 1,000 | 44,252.83 | ± 1.08 |
| `twoTransformsView` | 1,000 | 80,674.76 | ± 1.12 |
| `twoTransformsList` | 1,000,000 | 22.85 | ± 3.78 |
| `twoTransformsView` | 1,000,000 | 77.59 | ± 1.46 |
| `threeTransformsList` | 10 | 3,360,399.58 | ± 1.11 |
| `threeTransformsView` | 10 | 3,438,977.91 | ± 1.27 |
| `threeTransformsList` | 1,000 | 36,226.87 | ± 1.65 |
| `threeTransformsView` | 1,000 | 58,981.24 | ± 1.80 |
| `threeTransformsList` | 1,000,000 | 10.33 | ± 3.58 |
| `threeTransformsView` | 1,000,000 | 49.01 | ± 1.36 |

这些结果为我们提供了对使用视图产生更好性能的案例的有趣见解。对于小集合，例如我们基准中的 10 个元素，`List`的表现更好，无论操作量如何，尽管这个差距在 1,000,000 个元素时缩小。当我们转换一个大集合时，例如我们基准中的 1,000,000 个元素，随着转换数量的增加，视图变得更加高效，差异也在增加。例如，对于 1,000,000 个元素和两次转换，视图提供的吞吐量大约是`List`的三倍。在中等大小集合的情况下，例如本例中的 1,000 个元素，这并不那么明显。在执行单个转换时，急切的`List`表现更好，而在应用多个转换时，视图提供的吞吐量更高。

随着数据量和转换次数的增加，视图提供更好的性能的可能性也更大。在这里，你可以看到避免中间集合的实质性好处。考虑性能的第二个方面是转换的性质。从早期终止中受益的转换（例如，`find`），从懒加载中受益很大。这个基准测试说明，了解你的数据大小和打算执行的转换非常重要。

## 视图注意事项

视图提供了一种简单的方法，通过最小侵入性更改系统来提高性能。易用性是视图吸引力的部分，可能会诱使你比平时更频繁地使用它们。正如我们上一节中的基准测试所显示的，使用视图存在非微不足道的开销，这意味着默认使用视图是一个次优选择。从纯粹的性能角度来看，使用视图时还有其他需要谨慎的理由。

### SeqView 扩展了 Seq

由于视图反映了集合 API，识别何时应用懒加载的转换可能是一个挑战。因此，我们建议为视图的使用设定明确的边界。在处理客户端报告时，我们将视图的使用限制在一个内部函数中，并使用 `List` 急加载集合类型作为返回类型。最小化系统执行懒加载的区域可以减少构建运行时执行心理模型时的认知负荷。

在相关方面，我们认为谨慎地处理视图如何转换为急加载的集合类型非常重要。我们通过调用 `toList` 来展示转换，这使得意图变得明确。`SeqView` 还提供了一个 `force` 方法来强制评估。作为一般规则，我们避免使用 `force`，因为它通常返回 `scala.collection.immutable.Seq`。`SeqView` 保留集合类型作为其第二个泛型参数，这使得在有足够证据的情况下，`force` 可以返回原始集合类型。然而，某些操作，如 `map`，会导致视图失去原始集合类型的证据。当这种情况发生时，`force` 返回更通用的 `Seq` 集合类型。`Seq` 是一个特质，它是集合库中所有序列的超类型，包括视图和我们将要讨论的另一个懒加载数据结构，名为 `scala.collection.immutable.Stream`。这种继承方案允许以下三个语句编译：

```java
val list: Seq[Int] = List(1, 2, 3) 
val view: Seq[Int] = list.view 
val stream: Seq[Int] = list.toStream 

```

我们认为这是不可取的，因为 `Seq` 数据类型隐藏了关于底层实现的临界信息。它使用相同的类型表示懒加载和急加载的集合。考虑以下代码片段示例，以了解为什么这是不可取的：

```java
def shouldGenerateOrder(xs: Seq[Execution]): Boolean =  
  xs.size >= 3 

```

在这个人为的例子中，想象一下`shouldGenerateOrder`方法被一个`Vector`对象调用，但后来`Vector`被`SeqView`替换。使用`Vector`时，识别集合长度是一个常数时间操作。使用`SeqView`时，你无法确定操作的运行时间，只能说它肯定比`Vector.size`更昂贵。由于难以推理运行时行为，因此应避免使用`Seq`，以及因此导致的`force`的使用，因为这可能导致意外的副作用。

在典型的软件系统中，责任区域被划分为离散的模块。使用性能趋势报告的例子，你可以想象一个独立的模块，它包含将`List[TradingPerformanceTrend]`转换为 PDF 报告的转换。你可能想将视图暴露给其他模块以扩展延迟转换的好处。如果基准测试证明进行此类更改是合理的，那么我们鼓励你选择这些选项之一。在这种情况下，我们首选的选择是使用`Stream`，它是`List`的延迟评估版本。我们将在本章后面探讨`Stream`。如果无法使用`Stream`，请在使用`SeqView`数据类型时保持严格，以清楚地界定集合是延迟评估的。

### 视图不是记忆化器

使用视图时，还需要考虑的一点是要意识到何时重复应用转换。例如，考虑以下人为的例子，它关注一个视图作为多个计算基础的用例：

```java
> val xs = List(1,2,3,4,5).view.map(x => {  println(s"multiply $x"); x * 2 }) 
xs: scala.collection.SeqView[Int,Seq[_]] = SeqViewM(...) 
> val evens = xs.filter(_ % 2 == 0).toList 
multiply 1 
multiply 2 
multiply 3 
multiply 4 
multiply 5 
evens: List[Int] = List(2, 4, 6, 8, 10) 

> val odds = xs.filter(_ % 2 != 0).toList 
multiply 1 
multiply 2 
multiply 3 
multiply 4 
multiply 5 
odds: List[Int] = List() 

```

在这个例子中，`xs`是一个整数列表的视图。一个`map`转换被延迟应用于将这些整数乘以 2。然后，视图被用来创建两个`List`实例，一个包含偶数元素，另一个包含奇数元素。我们观察到转换被应用于视图两次，每次我们将视图转换为列表时。这表明转换是延迟应用的，但计算的结果没有被缓存。这是视图的一个需要注意的特性，因为多次应用昂贵的转换可能会导致显著的性能下降。这也是为什么在应用于视图的转换中应避免副作用的原因。如果由于某种原因，引用透明性没有得到保持，由于视图的使用导致的副作用和多次评估的组合可能会导致难以维护的软件。

这个例子很简单，视图的误用很容易被发现。然而，即使是标准库提供的方法，在与视图一起使用时也可能导致不希望的结果。考虑以下片段：

```java
> val (evens, odds) = List(1,2,3,4,5).view.map(x => {  println(s"multiply $x"); x * 2 }).partition(_ % 2 == 0) 
evens: scala.collection.SeqView[Int,Seq[_]] = SeqViewMF(...) 
odds: scala.collection.SeqView[Int,Seq[_]] = SeqViewMF(...) 

> println(evens.toList, odds.toList) 
multiply 1 
multiply 2 
multiply 3 
multiply 4 
multiply 5 
multiply 1 
multiply 2 
multiply 3 
multiply 4 
multiply 5 
(List(2, 4, 6, 8, 10),List()) 

```

这个例子实现了与上一个示例相同的结果，但我们依赖于内置的 `partition` 方法将原始列表分割成两个独立的集合，每个集合都操作原始视图。再次，我们看到 `map` 转换被两次应用于原始视图。这是由于 `TraversableViewLike` 中 `partition` 的底层实现。主要的启示是视图和懒加载可以帮助提高性能，但它们应该谨慎使用。在 REPL 中实验并尝试你的算法是一个好主意，以确认你正确地使用了视图。

在我们的关于报告交易性能趋势的运行示例中，我们看到了一个容易忽视的懒加载示例，当在 `Map` 上操作时。回想一下，有一个使用以下代码构建的查找表：

```java
executions.groupBy(_.id).mapValues(es =>
Price.average(es.map(_.price)))
```

`mapValues` 的返回类型是 `Map[A, B]`，这并不暗示任何评估策略上的差异。让我们在 REPL 中运行一个简单的例子：

```java
> val m = Map("a" -> 1, "b" -> 2)
m: scala.collection.immutable.Map[String,Int] = Map(a -> 1, b -> 2)
> val m_prime = m.mapValues{ v => println(s"Mapping $v"); v * 2}
Mapping 1
Mapping 2
m_prime: scala.collection.immutable.Map[String,Int] = Map(a -> 2, b -> 4)
> m_prime.get("a")
Mapping 1
res0: Option[Int] = Some(2)
> m_prime.get("a")
Mapping 1
res1: Option[Int] = Some(2)
```

注意每次我们在 `m_prime` 上调用 `get` 来检索一个值时，我们都可以观察到转换的应用，即使使用相同的键。`mapValues` 是对映射中每个值进行懒加载转换，类似于在映射的键上操作的视图。涉及到的类型并不提供任何见解，除非你检查 `Map` 的实现或仔细阅读与 `mapValues` 相关的文档，否则你可能会错过这个重要的细节。在处理 `mapValues` 时，考虑视图的注意事项。

## 报告生成中的压缩

在调查 `TradingPerformanceTrend` 的实现时，我们深入研究了视图，并发现了它们如何提高性能。现在我们回到 `trend` 的实现，以完成 `List[TradingPerformanceTrend]` 的生成。以下片段显示了 `trend`，其中 `periodPnL` 的实现被隐藏，因为我们已经彻底审查了它：

```java
def trend( 
  now: () => Instant, 
  findOrders: (Duration, Ticker) => List[Order], 
  findExecutions: (Duration, Ticker) => List[Execution], 
    request: GenerateTradingPerformanceTrend): List[TradingPerformanceTrend] = { 
    def periodPnL( 
      start: Instant => Instant): Map[Ticker, PeriodPnL] = { ... } 

    val tickerToLastHour = periodPnL(now => 
      now.minus(Period.hours(1).getMillis)).mapValues { 
      case PeriodPositive => LastHourPositive 
      case PeriodNegative => LastHourNegative 
    } 
    val tickerToLastDay = periodPnL(now => 
      now.minus(Period.days(1).getMillis)).mapValues { 
      case PeriodPositive => LastDayPositive 
      case PeriodNegative => LastDayNegative 
    } 
    val tickerToLastSevenDays = periodPnL(now => 
      now.minus(Period.days(7).getMillis)).mapValues { 
      case PeriodPositive => LastSevenDayPositive 
      case PeriodNegative => LastSevenDayNegative 
    } 
    tickerToLastHour.zip(tickerToLastDay).zip(tickerToLastSevenDays).map({ 
      case (((t, lastHour), (_, lastDay)), (_, lastSevenDays)) => 
        TradingPerformanceTrend(t, lastHour, lastDay, lastSevenDays) 
    }).toList 
  } 

```

此方法专注于将某个时间段的 PnL 转换到相应时间段的性能趋势。涉及两个 `zip` 调用的最终表达式使从具有 `Ticker` 键和相应时间段 PnL 趋势值的三个映射转换到 `List[TradingPerformanceTrend]` 变得优雅。`zip` 遍历两个集合，为每个集合的每个索引生成一个元组。以下是一个简单的片段，用于说明 `zip` 的用法：

```java
println(List(1, 3, 5, 7).zip(List(2, 4, 6))) 

```

这会产生以下结果：

```java
List((1,2), (3,4), (5,6)) 

```

结果是相应的索引被“压缩”在一起。例如，在索引一，第一个列表的值是三，第二个列表的值是四，得到元组 `(3, 4)`。第一个列表有四个元素，而第二个列表只有三个元素；这在结果集合中被默默地省略了。这种行为有很好的文档记录，但一开始可能会让人感到意外。在我们的报告用例中，我们确信每个键（即每个`Ticker`）都出现在所有三个映射中。在这个用例中，我们确信所有三个映射的长度相等。

然而，在我们的`zip`使用中存在一个微妙的错误。`zip`使用集合的迭代器来遍历元素，这意味着`zip`的使用对排序敏感。这三个映射中的每一个都是通过调用`toMap`构建的，这间接地委托给`scala.collection.immutable.HashMap`的`Map`实现。与`Set`类似，Scala 为小集合大小提供了几个手写的`Map`实现（例如，`Map2`），在构建`HashMap`之前。到现在，你可能已经意识到缺陷，`HashMap`不保证排序。

为了修复这个错误并保留`zip`的使用，我们可以利用我们之前发现的`SortedMap`，它是基于`TreeMap`并具有排序键的特质。用`SortedMap`替换`Map`，并对`Ticker`定义适当的`Ordering`，我们现在有一个无错误的、优雅的解决方案来生成交易性能趋势报告。通过审慎地使用视图，我们发现了一种通过最小侵入性更改实现迭代性能改进的方法。这将给销售团队带来一些值得庆祝的事情！这也给我们提供了更多时间来考虑其他生成报告的方法。

# 重新思考报告架构

在部署了一个包含您视图更改的性能报告的新版本网站门户之后，您开始思考还能做些什么来提高报告生成性能。您突然想到，对于特定的时间间隔，报告是不可变的。对于特定小时的计算 PnL 趋势一旦计算出来就永远不会改变。尽管报告是不可变的，但它每次客户端请求报告时都会被无谓地重新计算。根据这种思考方式，您想知道当新的执行数据可用时，每小时生成一份新报告有多困难。在创建时，订单和执行事件可以即时转换为客户端性能趋势报告所需的输入。有了预先生成的报告，网站门户的性能问题应该完全消失，因为报告生成的责任不再属于网站门户。

这种新的报告生成策略引导我们探索一个新的设计范式，称为事件源。事件源描述了一种设计系统的架构方法，它依赖于处理随时间推移的事件，而不是依赖于当前状态模型来回答不同的问题。我们工作的报告系统为了识别执行订单的子集而进行了大量工作，因为当前状态而不是事件被存储。想象一下，如果我们不是与数据，如`Order`和`Execution`，而是与代表系统随时间发生的事情的事件一起工作。一个相关的报告事件可以是`OrderExecuted`事件，可以按以下方式建模：

```java
case class OrderExecuted(created: CreatedTimestamp, orderId: OrderId, price: Price) 

```

此事件描述了发生的事情，而不是表示当前状态的快照。为了扩展这个例子，想象一下如果`Order`还包括一个可选的`Price`来表示执行价格：

```java
sealed trait Order { 
  def created: CreatedTimestamp 
  def id: OrderId 
  def ticker: Ticker 
  def price: Price 
  def clientId: ClientId 
  def executionPrice: Option[Price] 
} 

```

如果将此数据模型映射到关系数据库，`executionPrice`将是一个可空数据库值，在执行发生时被覆盖。当领域模型仅反映当前状态时，不可变性就会丢失。作为一个函数式程序员，这个声明应该让你感到担忧，因为你理解不可变性提供的推理能力。仅存储数据的当前状态也可能导致过大的对象，难以编程。例如，你将如何表示一个`Order`被取消？按照当前的方法，最快捷的方法是添加一个名为`isCanceled`的布尔标志。随着时间的推移，随着你系统需求的变得更加复杂，`Order`对象将增长，你将跟踪更多关于当前状态的特征。这意味着从数据库中加载一系列`Order`对象到内存中将会变得更加难以控制，因为内存需求不断增长。如果你有丰富的**对象关系映射**（**ORM**）经验，你很可能已经经历过这种困境。

为了避免`Order`膨胀，你可能尝试分解订单的概念以支持多个用例。例如，如果你只对已执行的订单感兴趣，模型可能会将`executionPrice`数据类型从`Option[Price]`改为`Price`，你可能也不再需要取消的布尔标志，因为根据定义，已执行的订单不可能被取消。

识别出曾经认为是一个单一概念的多重定义或表示，是解决我们所经历的不足的重要步骤。扩展这一方法，我们回到了事件源的话题。我们可以回放一系列事件来构建`OrderExecuted`。让我们稍微修改订单簿发出的以下事件：

```java
sealed trait OrderBookEvent 
case class BuyOrderSubmitted(created: CreatedTimestamp,  
  id: OrderId, ticker: Ticker, price: Price, clientId: ClientId) 
  extends OrderBookEvent 
case class SellOrderSubmitted(created: CreatedTimestamp,  
  id: OrderId, ticker: Ticker, price: Price clientId: ClientId) 
  extends OrderBookEvent 
case class OrderCanceled(created: CreatedTimestamp, id: OrderId) 
  extends OrderBookEvent 
case class OrderExecuted(created: CreatedTimestamp,  
  id: OrderId, price: Price) extends OrderBookEvent 

```

如果所有`OrderBookEvents`都被持久化（例如，存储到磁盘），那么就可以编写一个程序来读取所有事件，并通过将`BuyOrderSubmitted`和`SellOrderSubmitted`事件与`OrderExecuted`事件相关联来构建一组`ExecutedOrders`。我们注意到这种方法的一个优点是，随着时间的推移，我们能够提出关于系统发生的新问题，然后通过读取事件轻松回答它们。相比之下，如果一个基于当前状态的模型在最初设计时没有包括执行，那么就 impossible to retroactively answer the question, "Which orders executed last week?"（无法事后回答“上周哪些订单被执行了？”）。

我们的新想法令人兴奋，并且有可能带来巨大的改进。然而，它也带来了一系列挑战。与上一节的主要区别在于，我们新的用例不是从数据存储中加载`Order`和`Execution`集合到内存中。相反，我们计划处理由订单簿生成的`OrderBookEvent`。从概念上讲，这种方法仍然涉及处理数据序列。然而，在先前的方法中，整个数据集在开始任何转换之前就已经存在。即时处理事件需要设计处理尚未生成的数据的软件。显然，无论是急切集合还是视图都不是我们新系统的理想工具。幸运的是，标准的 Scala 库为我们提供了正确的抽象：`Stream`。让我们更仔细地看看这种新的集合类型，以更好地理解`Stream`如何帮助我们实现客户端性能报告架构的事件源方法。

## Stream 概述

Stream 可以看作是列表和视图的混合体。像视图一样，它是延迟评估的，并且只有在访问或收集其元素时才应用转换。像`List`一样，`Stream`的元素只被评估一次。`Stream`有时被描述为未实现的`List`，这意味着它本质上是一个尚未完全评估或实现的`List`。

`List`可以用 cons（`::`）运算符构建，`Stream`也可以用其自己的运算符类似地构建：

```java
> val days = "Monday" :: "Tuesday" :: "Wednesday" :: Nil 
days: List[String] = List(Monday, Tuesday, Wednesday) 

> val months = "January" #:: "February" #:: "March" #:: Stream.empty 
months: scala.collection.immutable.Stream[String] = Stream(January, ?) 

```

创建`Stream`的语法与创建`List`的语法相似。一个区别是返回值。`List`是立即评估的，而`Stream`不是。只有第一个元素（`"January"`）被计算；其余的值仍然是未知的（用`?`字符表示）。

让我们观察当我们访问流的一部分时会发生什么：

```java
scala> println(months.take(2).toList) 
List(January, February) 
scala> months 
res0: scala.collection.immutable.Stream[String] = Stream(January, February, ?) 

```

我们通过将其转换为`List`（见下文侧边栏）来强制评估`Stream`的前两个元素。前两个月的数据被打印出来。然后我们显示`months`的值，发现第二个元素（`"February"`）现在已经被计算。

### 注意

在前面的示例中，`toList` 是强制评估 `Stream` 的调用。`take(2)` 是一个惰性应用的转换器，它也返回一个未评估的 `Stream`：

```java
scala> months.take(2) 
res0: scala.collection.immutable.Stream[String] = Stream(January, ?) 

```

为了突出 `Stream` 的评估特性，我们来看另一个创建 `Stream` 的示例：

```java
def powerOf2: Stream[Int] = { 
  def next(n: Int): Stream[Int] = { 
    println(s"Adding $n") 
    n #:: next(2 * n) 
  } 
  1 #:: next(1) 
} 

```

这段简短的代码定义了一个函数，它创建一个包含 2 的幂的 `Stream`。它是一个无限 `Stream`，以第一个值 1 初始化，尾部定义为另一个 `Stream`。我们添加了一个 `println` 语句，以便我们可以研究元素的评估：

```java
scala> val s = powerOf2 
s: Stream[Int] = Stream(1, ?) 

scala> s.take(8).toList 
Adding 1 
Adding 2 
Adding 4 
Adding 8 
Adding 16 
Adding 32 
Adding 64 
res0: List[Int] = List(1, 1, 2, 4, 8, 16, 32, 64) 

scala> s.take(10).toList 
Adding 128 
Adding 256 
res1: List[Int] = List(1, 1, 2, 4, 8, 16, 32, 64, 128, 256) 

```

注意，前八个元素只有在我们将它们转换为 `List` 的第一次转换时才会被评估。在第二次调用中，只有第 9 和第 10 个元素被计算；前八个已经实现，并且是 `Stream` 的一部分。

### 注意

根据前面的示例，你可能想知道 `Stream` 是否是一个不可变的数据结构。它的完全限定名是 `scala.collection.immutable.Stream`，所以这应该给你一个很好的提示。确实，访问 `Stream` 和实现其一些元素会导致 `Stream` 的修改。然而，这个数据结构仍然被认为是不可变的。它包含的值一旦分配就不会改变；甚至在评估之前，这些值就存在，并在 `Stream` 中有定义。

前面的示例展示了 `Stream` 的一个有趣特性：可以创建一个几乎无限的 `Stream`。由 `powerOf2` 创建的 `Stream` 是无界的，并且由于我们的 `next` 方法，总是可以创建一个额外的元素。另一种有用的技术是递归流的创建。递归 `Stream` 在其定义中引用自身。让我们调整我们之前的示例。我们不会返回完整的 2 的幂序列，而是允许调用者设置一个起始值：

```java
def powerOf2(n: Int): Stream[Int] = math.pow(2, n).toInt #:: powerOf2(n+1) 

```

使用 `math.pow` 来计算 `2^n`。注意，我们计算第一个值，并将其余的 `Stream` 定义为 `powerOf2(n+1)`，即下一个 2 的幂：

```java
scala> powerOf2(3).take(10).toList 
res0: List[Int] = List(8, 16, 32, 64, 128, 256, 512, 1024, 2048, 4096) 

```

`Stream` 的伴随对象提供了几个工厂方法来实例化一个 `Stream`。让我们看看其中的一些：

+   `Stream.apply`: 这允许我们为有限序列的值创建一个 `Stream`：

```java
       scala> Stream(1,2,3,4) 
       res0: scala.collection.immutable.Stream[Int] = Stream(1, ?) 
       scala> Stream(List(1,2,3,4):_*) 
       res1: scala.collection.immutable.Stream[Int] = Stream(1, ?) 

```

+   `Stream.fillA(a: => A)`: 这会产生一个包含元素 `a`，重复 `n` 次的 `Stream`：

```java
       scala> Stream.fill(4)(10) 
       res0: scala.collection.immutable.Stream[Int] = Stream(10, ?) 
       scala> res0.toList 
       res1: List[Int] = List(10, 10, 10, 10) 

```

+   `Stream.from(start: Int)`: 这创建了一个以 `start` 开始的递增整数序列：

```java
       scala> Stream.from(4) 
       res0: scala.collection.immutable.Stream[Int] = Stream(4, ?) 
       scala> res0.take(3).toList 
       res1: List[Int] = List(4, 5, 6) 

```

我们邀请您查看伴随对象上可用的其他方法。注意，`Stream` 也可以直接从 `List` 构建，如下所示：

```java
scala> List(1,2,3,4,5).toStream 
res0: scala.collection.immutable.Stream[Int] = Stream(1, ?) 

```

之前的代码可能具有误导性。将 `List` 转换为 `Stream` 并不会节省在内存中评估整个 `List` 的代价。同样，如果我们要在调用 `toStream` 之前对 `List` 应用转换（如 `map` 或 `filter`），我们将在整个 `List` 上执行这些计算。

就像 `List` 一样，你可以在 `Stream` 上进行模式匹配，如下所示：

```java
scala> val s = Stream(1,2,3,4) 
s: scala.collection.immutable.Stream[Int] = Stream(1, ?) 
scala> s match { 
     | case _ #:: _ #:: i #:: _ => i 
     | } 
res0: Int = 3 

```

此模式匹配从`s`流中提取第三个元素。在流上执行模式匹配会强制实现评估匹配表达式所需的元素。在前面的例子中，前三个项目被计算。

### 注意

要在空流上进行模式匹配，你可以使用`Stream.Empty`对象。它是一个单例实例，用于表示空的`Stream`。它的工作方式与`List`中的`Nil`类似。请注意，`Stream`对象包含一个返回此单例的`empty`方法；然而，模式匹配需要一个稳定的标识符，并且不能使用方法调用作为有效的`case`。

## 事件转换

回到报告系统，我们如何应用事件源原则并利用`Stream`来改变报告的生成方式？为了计算客户的`TradingPerformanceTrend`，我们需要计算三个时间段的 PnL 趋势值：每小时、每天和每七天。我们可以编写一个具有以下签名的函数，这使我们更接近于识别每个趋势的 PnL：

```java
def processPnl(e: OrderBookEvent, s: TradeState): (TradeState, Option[PnlEvent]) 

```

`processPnl`的签名接受一个`OrderBookEvent`和以`TradeState`形式的状态，以生成一个新的`TradeState`和可选的`PnlEvent`。让我们首先检查`PnlEvent`，以了解此方法的结果，然后再检查`TradeState`：

```java
sealed trait PnlEvent 
case class PnlIncreased(created: EventInstant, clientId: ClientId, 
  ticker: Ticker, profit: Pnl) extends PnlEvent 
case class PnlDecreased(created: EventInstant, clientId: ClientId, 
  ticker: Ticker, loss: Pnl)extends PnlEvent 

case class Pnl(value: BigDecimal) extends AnyVal { 
  def isProfit: Boolean = value.signum >= 0 
} 
object Pnl { 
  def fromExecution(buy: Price, sell: Price): Pnl = 
    Pnl(sell.value - buy.value) 

  val zero: Pnl = Pnl(BigDecimal(0)) 
} 

```

我们可以看到`PnlEvent`模型了一个 ADT，它表达了客户的 PnL 何时增加或减少。使用过去时来命名事件（例如，增加了）清楚地表明这是一个事实或已完成某事的记录。我们尚未查看`TradeState`的定义或`processPnl`的实现，但我们可以通过研究发出的事件来推断行为。我们显示`TradeState`的定义，这是将提交的订单与执行相关联所需的，如下所示：

```java
case class PendingOrder(ticker: Ticker, price: Price,  
  clientId: ClientId)  

  case class TradeState( 
    pendingBuys: Map[OrderId, PendingOrder], 
    pendingSells: Map[OrderId, PendingOrder]) { 
    def cancelOrder(id: OrderId): TradeState = copy( 
      pendingBuys = pendingBuys - id, pendingSells = pendingSells - id) 
    def addPendingBuy(o: PendingOrder, id: OrderId): TradeState = 
      copy(pendingBuys = pendingBuys + (id -> o)) 
    def addPendingSell(o: PendingOrder, id: OrderId): TradeState = 
      copy(pendingSells = pendingSells + (id -> o)) 
  } 
object TradeState { 
 val empty: TradeState = TradeState(Map.empty, Map.empty) 
} 

```

接下来，我们检查`processPnl`的实现，以查看`PnlEvents`是如何创建的，如下所示：

```java
  def processPnl( 
    s: TradeState, 
    e: OrderBookEvent): (TradeState, Option[PnlEvent]) = e match { 
    case BuyOrderSubmitted(_, id, t, p, cId) => 
      s.addPendingBuy(PendingOrder(t, p, cId), id) -> None 
    case SellOrderSubmitted(_, id, t, p, cId) => 
      s.addPendingSell(PendingOrder(t, p, cId), id) -> None 
    case OrderCanceled(_, id) => s.cancelOrder(id) -> None 
    case OrderExecuted(ts, id, price) => 
      val (p, o) = (s.pendingBuys.get(id), s.pendingSells.get(id)) match { 
        case (Some(order), None) => 
          Pnl.fromBidExecution(order.price, price) -> order 
        case (None, Some(order)) => 
          Pnl.fromOfferExecution(price, order.price) -> order 
        case error => sys.error( 
          s"Unsupported retrieval of ID = $id returned: $error") 
      } 
      s.cancelOrder(id) -> Some( 
        if (p.isProfit) PnlIncreased(ts, o.clientId, o.ticker, p) 
        else PnlDecreased(ts, o.clientId, o.ticker, p)) 
  } 

```

此实现表明`PnlEvent`被模式匹配以确定事件类型，并相应地处理。当提交订单时，`TradeState`被更新以反映存在一个新挂起的订单，该订单将被取消或执行。当订单被取消时，挂起的订单从`TradeState`中移除。当发生执行时，挂起的订单被移除，并且，在计算交易 PnL 后，还会发出一个`PnlEvent`。交易 PnL 将执行价格与挂起订单的原始价格进行比较。

`PnlEvent` 提供了足够的信息来计算 `TradingPerformanceTrend` 所需的所有三个时间段（小时、天和七天）的盈亏趋势。从 `OrderBookEvent` 到 `PnlEvent` 的转换是无副作用的，创建新事件而不是替换当前状态，导致不可变模型。鉴于这些特性，`processPnl` 很容易进行单元测试，并使意图明确。通过使意图明确，可以与技术利益相关者之外的人沟通系统的工作方式。

使用 `PnlEvent` 作为遵循类似 `(State, InputEvent) => (State, Option[OutputEvent])` 签名的方法的输入，我们现在可以计算小时的盈亏趋势，如下所示：

```java
def processHourlyPnl(e: PnlEvent, s: HourlyState): (HourlyState, Option[HourlyPnlTrendCalculated]) 

```

此签名表明，通过在 `HourlyState` 中维护状态，可以发出 `HourlyPnlTrendCalculated` 事件。发出的事件定义如下：

```java
case class HourlyPnlTrendCalculated( 
      start: HourInstant, 
      clientId: ClientId, 
      ticker: Ticker, 
      pnl: LastHourPnL) 

```

对于特定的小时、客户端 ID 和股票代码，`HourlyPnlTrendCalculated` 记录了上一小时的盈亏（PnL）是正数还是负数。`HourInstant` 类是一个值类，它有一个伴随对象方法，可以将一个瞬间转换为小时的开始：

```java
case class HourInstant(value: Instant) extends AnyVal { 
  def isSameHour(h: HourInstant): Boolean = 
    h.value.toDateTime.getHourOfDay == value.toDateTime.getHourOfDay 
} 
object HourInstant { 
  def create(i: EventInstant): HourInstant = 
    HourInstant(i.value.toDateTime.withMillisOfSecond(0) 
     .withSecondOfMinute(0).withMinuteOfHour(0).toInstant) 
} 

```

让我们看看 `HourlyState` 的定义，以便更好地理解产生 `HourlyPnlTrendCalculated` 所需的状态：

```java
case class HourlyState( 
      keyToHourlyPnl: Map[(ClientId, Ticker), (HourInstant, Pnl)]) 
object HourlyState { 
 val empty: HourlyState = HourlyState(Map.empty) 
} 

```

对于 `ClientId` 和 `Ticker`，当前小时的盈亏存储在 `HourlyState` 中。累积盈亏允许 `processHourlyPnl` 在小时结束时确定盈亏趋势。我们现在检查 `processHourlyPnl` 的实现，以了解 `PnlEvent` 如何转换为 `HourlyPnlTrendCalculated`：

```java
def processHourlyPnl( 
 s: HourlyState, 
 e: PnlEvent): (HourlyState, Option[HourlyPnlTrendCalculated]) = { 
 def processChange( 
   ts: EventInstant, 
   clientId: ClientId, 
   ticker: Ticker, 
   pnl: Pnl): (HourlyState, Option[HourlyPnlTrendCalculated]) = { 
   val (start, p) = s.keyToHourlyPnl.get((clientId, ticker)).fold( 
     (HourInstant.create(ts), Pnl.zero))(identity) 
   start.isSameHour(HourInstant.create(ts)) match { 
     case true => (s.copy(keyToHourlyPnl = s.keyToHourlyPnl + 
       ((clientId, ticker) ->(start, p + pnl))), None) 
     case false => (s.copy(keyToHourlyPnl = 
       s.keyToHourlyPnl + ((clientId, ticker) -> 
         (HourInstant.create(ts), Pnl.zero + pnl))), 
       Some(HourlyPnlTrendCalculated(start, clientId, ticker, 
         p.isProfit match { 
           case true => LastHourPositive 
           case false => LastHourNegative 
         }))) 
   } 
 } 

 e match { 
   case PnlIncreased(ts, clientId, ticker, pnl) => processChange( 
     ts, clientId, ticker, pnl) 
   case PnlDecreased(ts, clientId, ticker, pnl) => processChange( 
     ts, clientId, ticker, pnl) 
 } 
} 

```

处理盈亏的增加和减少遵循相同的流程。内部方法 `processChange` 处理相同的处理步骤。`processChange` 通过比较首次添加到状态时的 `HourInstant` 值和事件提供的时间戳的小时来决定是否发出 `HourlyPnlTrendCalculated`。当比较显示小时已更改时，则表示已计算了小时的盈亏趋势，因为小时已完成。当小时未更改时，提供的盈亏将添加到状态中的盈亏以继续累积小时的盈亏。

### 注意

这种方法的明显缺点是，当客户端或股票代码没有任何已执行订单时，将无法确定小时是否完成。为了简化，我们没有将时间视为一等事件。然而，你可以想象如何将时间的流逝建模为一个事件，它是 `processHourlyPnl` 的第二个输入。例如，该事件可能是以下内容：

`case class HourElapsed(hour: HourInstant)`

要使用此事件，我们可以将`processHourlyPnl`的签名更改为接收一个`Either[HourElapsed, PnlEvent]`类型的参数。在计时器上安排`HourElapsed`使我们能够修改`processHourlyPnl`的实现，以便在小时结束时立即发出`HourlyPnlTrendCalculated`，而不是在下一个小时内发生交易时发出。这个简单的例子展示了当你从事件源的角度考虑系统时，如何将时间作为域的显式部分进行建模。

想象编写类似的方法来发出每日和七日 PnL 趋势事件，然后编写一个等待所有三个 PnL 趋势事件以产生`TradingPerformanceTrendGenerated`事件的函数。最后一步是编写一个产生副作用的方法，将`TradingPerformanceTrend`持久化，以便它可以被网络门户读取。到此为止，我们有一系列执行事件转换的方法，但它们还没有紧密地连接在一起。接下来，我们将探讨如何创建一个转换事件的管道。

### 注意

注意，在本案例研究中，我们实际上并没有计算 PnL。进行真实的 PnL 计算将涉及更复杂的算法，并迫使我们引入更多的领域概念。我们选择了更简单的方法，使用一个更接近敞口报告的报告。这使我们能够专注于我们想要展示的代码和编程实践。

## 构建事件源管道

我们使用“pipeline”一词来指代一组经过安排的转换，这些转换可能需要多个步骤才能产生期望的最终结果。这个术语让人联想到一组管道，它们跨越多个方向，途中还有转弯。我们的目标是编写一个程序，该程序接收`PnlEvents`特性并将`HourlyPnlTrendCalculated`事件打印到标准输出。在一个真正的生产环境中，你可以想象用写入持久数据存储来替换打印到标准输出。在两种情况下，我们都在构建一个执行一系列引用透明转换并最终产生副作用的管道。

管道必须累积每个转换的中间状态，因为处理新事件。在函数式编程范式中，累积通常与`foldLeft`操作相关联。让我们看看一个玩具示例，该示例将整数列表相加，以更好地理解累积：

```java
val sum = Stream(1, 2, 3, 4, 5).foldLeft(0) { case (acc, i) => acc + i } 
println(sum) // prints 15  

```

在这里，我们看到`foldLeft`被应用于通过提供一个初始总和为零并 currying 一个函数来添加当前元素到累积总和来计算整数列表的总和。`acc`值是“累积器”的常用缩写。在这个例子中，累积器和列表元素具有相同的数据类型，整数。这只是一个巧合，并不是`foldLeft`操作的要求。这意味着累积器可以与集合元素具有不同的类型。

我们可以使用 `foldLeft` 作为事件源管道的基础，以支持在累积中间状态的同时处理 `OrderBookEvents` 列表。从两个处理方法的实现中，我们看到了维护 `TradeState` 和 `HourlyState` 的需求。我们定义 `PipelineState` 来封装所需的状态，如下所示：

```java
case class PipelineState(tradeState: TradeState, hourlyState: HourlyState) 
object PipelineState { 
  val empty: PipelineState = PipelineState(TradeState.empty, HourlyState.empty) 
} 

```

`PipelineState` 在折叠 `OrderBookEvent` 时作为累加器使用，使我们能够存储两种转换方法的中间状态。现在，我们准备定义管道的签名：

```java
def pipeline(initial: PipelineState, f: HourlyPnlTrendCalculated => Unit, xs: List[OrderBookEvent]): PipelineState 

```

`pipeline` 接受初始状态，一个在生成 `HourlyPnlTrendCalculated` 事件时被调用的副作用函数，以及一组 `OrderBookEvents` 作为数据源。管道的返回值是事件处理后的管道状态。让我们看看我们如何利用 `foldLeft` 来实现 `pipeline`：

```java
def pipeline( 
    initial: PipelineState, 
    f: HourlyPnlTrendCalculated => Unit, 
    xs: Stream[OrderBookEvent]): PipelineState = xs.foldLeft(initial) { 
    case (PipelineState(ts, hs), e) => 
      val (tss, pnlEvent) = processPnl(ts, e) 
      PipelineState(tss, 
        pnlEvent.map(processHourlyPnl(hs, _)).fold(hs) { 
          case (hss, Some(hourlyEvent)) => 
            f(hourlyEvent) 
            hss 
          case (hss, None) => hss 
        }) 
  } 

```

`pipeline` 的实现基于使用提供的 `PipelineState` 作为累加起始点的提供事件折叠。提供给 `foldLeft` 的柯里化函数是转换连接的地方。将两个转换方法和副作用事件处理器拼接在一起需要处理几个不同的场景。让我们逐一分析每种可能的案例，以更好地理解管道的工作原理。`processPnl` 被调用以生成新的 `TradeState` 并可选地产生一个 `PnlEvent`。如果没有生成 `PnlEvent`，则不调用 `processHourlyPnl` 并返回之前的 `HourlyState`。

如果生成了一个 `PnlEvent`，则 `processHourlyPnl` 被评估以确定是否创建了 `HourlyPnlTrendCalculated`。当 `HourlyPnlTrendCalculated` 被生成时，则调用副作用 `HourlyPnlTrendCalculated` 事件处理器并返回新的 `HourlyState`。如果没有生成 `HourlyPnlTrendCalculated`，则返回现有的 `HourlyState`。

我们构建一个简单的示例来证明管道按预期工作，如下所示：

```java
val now = EventInstant(HourInstant.create(EventInstant( 
      new Instant())).value) 
    val Foo = Ticker("FOO") 

    pipeline(PipelineState.empty, println, Stream( 
      BuyOrderSubmitted(now, OrderId(1), Foo, Price(21.07), ClientId(1)), 
      OrderExecuted(EventInstant(now.value.plus(Duration.standardMinutes(30))), 
        OrderId(1), Price(21.00)), 
      BuyOrderSubmitted(EventInstant(now.value.plus( 
        Duration.standardMinutes(35))), 
        OrderId(2), Foo, Price(24.02), ClientId(1)), 
      OrderExecuted(EventInstant(now.value.plus(Duration.standardHours(1))), 
        OrderId(2), Price(24.02)))) 

```

在小时的开始，提交了一个针对股票 FOO 的买入订单。在小时内，以低于买入价的价格执行了买入订单，这表明交易是盈利的。正如我们所知，当前实现依赖于下一小时的执行来生成 `HourlyPnlTrendCalculated`。为了创建此事件，在第二小时的开始提交了第二个买入订单。运行此代码片段会产生一个写入标准输出的单个 `HourlyPnlTrendCalculated` 事件：

```java
HourlyPnlTrendCalculated(HourInstant(2016-02-15T20:00:00.000Z),ClientId(1),Ticker(FOO),LastHourPositive) 

```

虽然将转换连接起来有些复杂，但我们设法仅使用 Scala 标准库和我们对 Scala 集合的现有知识构建了一个简单的事件源管道。这个例子展示了`foldLeft`在构建事件源管道方面的强大功能。使用此实现，我们可以编写一个功能齐全的程序，能够将预先生成的性能报告写入可以被网络门户读取的持久数据存储。这种新的设计允许我们将报告生成的负担从网络门户的责任中移除，从而使网络门户能够提供响应式的用户体验。这种新方法的好处之一是它将面向领域的语言置于设计的中心。我们所有的事件都使用业务术语，并专注于建模领域概念，这使得开发者和利益相关者之间的沟通更加容易。

### 注意

你可能想知道一个具有`Stream`的一些特征但我们尚未提到的数据结构：`Iterator`。正如其名所示，`Iterator`提供了遍历数据序列的设施。其简化的定义可以归结为以下内容：

```java
trait Iterator[A] { 
  def next: A 
  def hasNext: Boolean 
}
```

与`Stream`类似，`Iterator`能够避免将整个数据集加载到内存中，这使得程序可以以恒定的内存使用量编写。与`Stream`不同，`Iterator`是可变的，并且仅用于对集合进行单次迭代（它扩展了`TraversableOnce`特质）。需要注意的是，根据标准库文档，在调用其方法后不应再使用迭代器。例如，对`Iterator`调用`size`会返回序列的大小，但它也会消耗整个序列，使`Iterator`实例变得无用。此规则的唯一例外是`next`和`hasNext`。这些属性导致软件难以推理，这与我们作为函数式程序员所追求的目标相反。因此，我们省略了对`Iterator`的深入讨论。

我们鼓励您通过阅读[Event Store 数据库的文档](http://docs.geteventstore.com/introduction/event-sourcing-basics/)来进一步探索事件溯源。Event Store 是一个围绕事件溯源概念开发的数据库。Event Store 是由事件溯源领域的知名作家 Greg Young 创建的。在丰富您对事件溯源的理解的同时，思考一下您认为何时应用事件溯源技术是合适的。对于具有简单行为的 CRUD 应用程序，事件溯源可能不是值得投入时间的。当您建模更复杂的行为或考虑涉及严格性能和扩展要求的场景时，事件溯源的时间投入可能变得合理。例如，就像我们在性能趋势报告中看到的那样，从事件溯源范式暴露的性能挑战揭示了一种完全不同的设计方法。

当您继续探索流处理的世界时，您会发现您希望构建比我们的事件溯源管道示例更复杂的转换。为了继续深入研究流处理的话题，我们建议研究两个相关的库：`akka streams`和`functional streams`（以前称为`scalaz-stream`）。这些库提供了使用不同于`Stream`的不同抽象构建更复杂转换管道的工具。结合学习 Event Store，您将更深入地了解事件溯源如何与流处理相结合。

## 流式马尔可夫链

在上一节的末尾的简单程序中，我们展示了我们可以将操作事件的转换管道连接起来。作为一个有良好意图的工程师，您希望开发自动测试来证明管道按预期工作。一种方法是将历史生产数据样本添加到存储库中构建测试。这通常是一个不错的选择，但您担心样本不足以代表广泛的场景。另一种选择是编写一个可以创建类似生产数据的生成器。这种方法需要更多的前期努力，但它提供了一种更动态的方式来测试管道。

与 Dave 关于马尔可夫链的最近午餐时间对话激发了对使用生成数据进行事件溯源管道测试的想法。Dave 描述了马尔可夫链是一个仅依赖于当前状态来确定下一个状态的统计状态转换模型。Dave 将股票市场的状态表示为马尔可夫链，使他能够根据他是否认为股票市场处于上升趋势、下降趋势或稳定状态来构建交易策略。在阅读了马尔可夫链的维基百科页面后，您设想编写一个基于马尔可夫链的事件生成器。

我们的目标是能够生成无限数量的遵循生产模式样式的`OrderBookEvent`。例如，根据以往的经验，取消订单通常比执行订单多，尤其是在波动较大的市场中。事件生成器应该能够表示事件发生的不同概率。由于马尔可夫链只依赖于其当前状态来识别其下一个状态，因此`Stream`是一个自然的选择，因为我们只需要检查当前元素来确定下一个元素。对于我们对马尔可夫链的表示，我们需要确定从当前状态转换到任何其他可能状态的几率。以下表格展示了一组可能的概率集：

| **当前状态** | **购买几率** | **销售几率** | **执行几率** | **取消几率** |
| --- | --- | --- | --- | --- |
| `BuyOrderSubmitted` | 10% | 15% | 40% | 40% |
| `SellOrderSubmitted` | 25% | 10% | 35% | 25% |
| `OrderCanceled` | 60% | 50% | 40% | 10% |
| `OrderExecuted` | 30% | 30% | 55% | 30% |

这个表格定义了在当前`OrderBookEvent`的情况下接收`OrderBookEvent`的可能性。例如，给定一个销售订单已提交，下一个出现第二个销售订单的概率为 10%，下一个发生执行的概率为 35%。我们可以根据我们希望在管道中模拟的市场条件开发状态转换概率。

我们可以使用以下领域来模拟转换：

```java
  sealed trait Step 
  case object GenerateBuy extends Step 
  case object GenerateSell extends Step 
  case object GenerateCancel extends Step 
  case object GenerateExecution extends Step 

  case class Weight(value: Int) extends AnyVal 
  case class GeneratedWeight(value: Int) extends AnyVal 
  case class StepTransitionWeights( 
    buy: Weight, 
    sell: Weight, 
    cancel: Weight, 
    execution: Weight) 

```

在这个领域内，`Step`是一个 ADT，它模拟了可能的状态。对于给定的`Step`，我们将关联`StepTransitionWeights`来定义基于提供的权重转换到不同状态的概率。`GeneratedWeight`是一个值类，它定义了当前`Step`生成的权重。我们将使用`GeneratedWeight`来驱动从`Step`到下一个`Step`的转换。

我们下一步要做的是利用我们的领域，根据我们定义的概率生成事件。为了使用`Step`，我们定义了所需的马尔可夫链状态的表示，如下所示：

```java
  case class State( 
    pendingOrders: Set[OrderId], 
    step: Step) 

```

马尔可夫链需要了解当前状态，这由`step`表示。此外，我们对马尔可夫链进行了一些调整，通过维护在`pendingOrders`中提交的既未取消也未执行的订单集合。这个额外的状态有两个原因。首先，生成取消和执行事件需要链接到一个已知的订单 ID。其次，我们通过要求在创建取消或执行之前至少存在一个挂起的订单来限制我们对马尔可夫链的表示。如果没有挂起的订单，转换到生成`OrderCanceled`或`OrderExecuted`的状态是不合法的。

使用`State`，我们可以编写一个具有以下签名的方法来管理转换：

```java
def nextState( 
      weight: StepTransitionWeights => GeneratedWeight, 
      stepToWeights: Map[Step, StepTransitionWeights], 
      s: State): (State, OrderBookEvent) 

```

给定从当前`StepTransitionWeights`生成权重的方法、`Step`到`StepTransitionWeights`的映射以及当前`State`，我们能够生成一个新的`State`和一个`OrderBookEvent`。为了简洁，我们省略了`nextState`的实现，因为我们想最专注地关注流处理。从签名中，我们有足够的洞察力来应用该方法，但我们鼓励你检查存储库以填补你理解中的任何空白。

`nextState`方法是我们在马尔可夫链表示中状态转换的驱动程序。现在我们可以使用便利的`Stream`方法`iterate`根据转换概率生成无限`Stream`的`OrderBookEvent`。根据 Scala 文档，`iterate`通过重复应用函数到起始值来产生无限流。让我们看看我们如何使用`iterate`：

```java
val stepToWeights = MapStep, StepTransitionWeights, Weight(25), Weight(40), Weight(40)), 
      GenerateSell -> StepTransitionWeights( 
        Weight(25), Weight(10), Weight(40), Weight(25)), 
      GenerateCancel -> StepTransitionWeights( 
        Weight(60), Weight(50), Weight(40), Weight(10)), 
      GenerateExecution -> StepTransitionWeights( 
        Weight(30), Weight(30), Weight(60), Weight(25))) 

    val next = State.nextState( 
      t => GeneratedWeight(Random.nextInt(t.weightSum.value) + 1), 
      stepToWeights, _: State) 

    println("State\tEvent") 
    Stream.iterate(State.initialBuy) { case (s, e) => next(s) } 
      .take(5) 
      .foreach { case (s, e) => println(s"$s\t$e")  } 

```

这个片段创建了一个马尔可夫链，通过提供`Step`到`StepTransitionWeights`的映射作为调用`State.nextState`的基础来生成各种`OrderBookEvent`。`State.nextState`是部分应用的，当前状态未被应用。`next`函数具有`State => (State, OrderBookEvent)`签名。在必要的框架到位后，使用`Stream.iterate`通过调用`next`生成无限序列的多个`OrderBookEvent`。类似于`foldLeft`，我们提供一个初始值以开始`initialBuy`迭代，定义如下：

```java
val initialBuy: (State, OrderBookEvent) = { 
      val e = randomBuySubmitted() 
      State(Set(e.id), GenerateBuy) -> e 
    } 

```

运行此片段会产生类似于以下内容的输出：

```java
 State = State(Set(OrderId(1612147067584751204)),GenerateBuy)
    Event = BuyOrderSubmitted(EventInstant(2016-02-22T23:52:40.662Z),OrderId(1612147067584751204),Ticker(FOO),Price(32),ClientId(28))
    State = State(Set(OrderId(1612147067584751204), OrderId(7606120383704417020)),GenerateBuy)
    Event = BuyOrderSubmitted(EventInstant(2016-02-22T23:52:40.722Z),OrderId(7606120383704417020),Ticker(XYZ),Price(18),ClientId(54))
    State = State(Set(OrderId(1612147067584751204), OrderId(7606120383704417020), OrderId(5522110701609898973)),GenerateBuy)
    Event = BuyOrderSubmitted(EventInstant(2016-02-22T23:52:40.723Z),OrderId(5522110701609898973),Ticker(XYZ),Price(62),ClientId(28))
    State = State(Set(OrderId(7606120383704417020), OrderId(5522110701609898973)),GenerateExecution)
    Event = OrderExecuted(EventInstant(2016-02-22T23:52:40.725Z),OrderId(1612147067584751204),Price(21))
    State = State(Set(OrderId(7606120383704417020), OrderId(5522110701609898973), OrderId(5898687547952369568)),GenerateSell)
    Event = SellOrderSubmitted(EventInstant(2016-02-22T23:52:40.725Z),OrderId(5898687547952369568),Ticker(BAR),Price(76),ClientId(45)) 
```

当然，每次调用都取决于为`GeneratedWeight`创建的随机值，它用于概率性地选择下一个转换。这个片段提供了一个基础来编写更大规模的测试，用于报告基础设施。通过这个例子，我们看到马尔可夫链的一个有趣应用，它支持从各种市场条件生成代表性事件，而无需访问大量生产数据。我们现在能够编写测试来确认报告基础设施是否正确计算了不同市场条件下的 PnL 趋势。

## 流注意事项

尽管它们很好，但使用`Stream`时应谨慎。在本节中，我们提到了`Stream`的一些主要注意事项以及如何避免它们。

### 流是记忆化器

虽然`views`不会缓存计算结果，因此每次访问时都会重新计算并实现每个元素，但`Stream`会保存其元素的最后形式。一个元素只会在第一次访问时实现。虽然这是一个避免多次计算相同结果的优秀特性，但它也可能导致大量内存消耗，以至于你的程序最终可能耗尽内存。

为了避免`Stream`记忆化，一个好的做法是避免将`Stream`存储在`val`中。使用`val`会创建对`Stream`头部的永久引用，确保每个被实例化的元素都会被缓存。如果将`Stream`定义为`def`，它可以在不再需要时立即被垃圾回收。

当调用定义在`Stream`上的某些方法时，可能会发生记忆化。例如，`drop`或`dropWhile`将评估并记忆化所有要删除的中间元素。这些元素在`Stream`实例（`Stream`有自己的头部引用）上定义方法时被记忆化。我们可以实现自己的`drop`函数来避免在内存中缓存中间元素：

```java
@tailrec 
def dropA: Stream[A] = count match { 
  case 0 => s 
  case n if n > 0 => drop(s.tail, count - 1) 
  case n if n < 0 => throw new Exception("cannot drop negative count") 
} 

```

我们通过匹配`count`的值来判断是否可以返回给定的`Stream`，或者需要在尾部进行递归调用。我们的方法是尾递归的。这确保了我们不会保留`Stream`头部的引用，因为尾递归函数每次循环时都会回收其引用。我们的`s`引用将仅指向`Stream`的剩余部分，而不是头部。

另一个有问题的方法示例是`max`。调用`max`将记忆化`Stream`的所有元素以确定哪个是最大的。让我们实现一个安全的`max`版本，如下所示：

```java
def max(s: => Stream[Int]): Option[Int] = { 
 @tailrec 
 def loop(ss: Stream[Int], current: Option[Int]): Option[Int] = ss match { 
   case Stream.Empty => current 
   case h #:: rest if current.exists(_ >= h) => loop(rest, current) 
   case h #:: rest => loop(rest, Some(h)) 
 } 
 loop(s, None) 
} 

```

这次，我们使用了一个内部尾递归函数来能够提供一个友好的 API。我们使用`Option[Int]`来表示当前的最大值，以处理方法在空`Stream`上被调用的情况。请注意，`max`接受`s`作为按名参数。这很重要，因为否则我们会在调用内部尾递归`loop`方法之前保留对`Stream`头部的引用。另一个可能的实现如下：

```java
def max(s: => Stream[Int]): Option[Int] = { 
 @tailrec 
 def loop(ss: Stream[Int], current: Int): Int = ss match { 
   case Stream.Empty => current 
   case h #:: hs if h > current => loop(hs, h) 
   case h #:: hs if h <= current => loop(hs, current) 
 } 

 s match { 
   case Stream.Empty => None 
   case h #:: rest => Some(loop(rest, h)) 
 } 
} 

```

这种实现可以说是更简单。我们在`max`函数中检查`Stream`是否为空；这允许我们立即返回（使用`None`），或者调用`loop`并传递一个有效的默认值（`Stream`中的第一个元素）。`loop`不再需要处理`Option[Int]`。然而，这个例子并没有达到避免记忆化的目标。模式匹配将导致`rest`保留对原始`Stream`整个尾部的引用，这将阻止中间元素的垃圾回收。一个好的做法是在消耗性、尾递归方法内部仅对`Stream`进行模式匹配。

### `Stream`可以是无限的

在我们的概述中，我们看到了定义一个无限`Stream`的可能性。然而，当你与无限`Stream`一起工作时，你需要小心。某些方法可能会导致整个`Stream`的评估，从而导致`OutOfMemoryError`。其中一些很明显，例如`toList`，它将尝试将整个`Stream`存储到一个`List`中，从而导致所有元素的实现。其他的一些则更为微妙。例如，`Stream`有一个`size`方法，它与`List`上定义的方法类似。在无限`Stream`上调用`size`会导致程序耗尽内存。同样，`max`和`sum`将尝试实现整个序列并使你的系统崩溃。这种行为尤其危险，因为`Stream`扩展了`Seq`，这是序列的基本特质。考虑以下代码：

```java
def range(s: Seq[Int]): Int = s.max - s.min 

```

这个简短的方法接受一个`Seq[Int]`作为单个参数，并返回其范围，即最大和最小元素之间的差值。由于`Stream`扩展了`Seq`，以下调用是有效的：

```java
val s: Stream[Int] = ??? 
range(s) 

```

编译器会愉快且迅速地为这个片段生成字节码。然而，`s`可以被定义为一个无限的`Stream`：

```java
val s: Stream[Int] = powerOf2(0) 
range(s) 
java.lang.OutOfMemoryError: GC overhead limit exceeded 
  at .powerOf2(<console>:10) 
  at $anonfun$powerOf2$1.apply(<console>:10) 
  at $anonfun$powerOf2$1.apply(<console>:10) 
  at scala.collection.immutable.Stream$Cons.tail(Stream.scala:1233) 
  at scala.collection.immutable.Stream$Cons.tail(Stream.scala:1223) 
  at scala.collection.immutable.Stream.reduceLeft(Stream.scala:627) 
  at scala.collection.TraversableOnce$class.max(TraversableOnce.scala:229) 
  at scala.collection.AbstractTraversable.max(Traversable.scala:104) 
  at .range(<console>:10) 
  ... 23 elided 

```

由于`max`和`min`的实现，`range`的调用永远不会返回。这个例子说明了我们在本章前面提到的一个良好实践。

# 摘要

在本章中，我们探讨了 Scala 标准库提供的两个懒加载集合：视图和流。我们探讨了它们的特性和实现细节，以及在使用这些抽象时需要注意的限制。利用你新获得的知识，你解决了一个影响 MVT 客户端试图查看其性能趋势的关键性能问题。

在*Stream*部分，我们有机会将流处理的概念与事件源结合。我们简要探讨了事件源范式，并介绍了一个简单的基于事件的转换管道，以改进报告系统的架构并定义一个更强的领域模型。最后，我们构建了一个马尔可夫链事件生成器来练习我们生成报告的新方法。

通过探索急切和懒加载集合，你现在对 Scala 标准库提供的集合有了坚实的实际知识。在下一章中，我们将继续通过函数式范式探索 Scala 概念，深入探讨并发性。
