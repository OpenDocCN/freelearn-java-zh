# 第六章 反应式数据流

在特定情况下，我们的应用程序可能需要处理大量文件上传。这可以通过将这些全部放入内存中、创建一个临时文件或直接在流上操作来实现。在这三种方法中，最后一种对我们来说效果最好，因为它消除了 I/O 流限制（如阻塞、内存和线程），并且消除了缓冲的需要（即在所需的速率上对输入进行操作）。

处理大量文件上传属于不可避免的操作集合，这些操作可能会对资源造成很大压力。属于同一类别的其他任务包括处理实时数据以进行监控、分析、批量数据传输以及处理大型数据集。在本章中，我们将讨论用于处理此类情况的 Iteratee 方法。本章涵盖了使用以下主题的简要说明来处理数据流的基础：

+   Iteratees

+   枚举器

+   Enumeratees

本章在某些时候可能看起来很紧张，但这里讨论的主题将对以下章节有所帮助。

# 数据流处理基础

假设我们将一个移动设备（如平板电脑、手机、MP3 播放器等）连接到其充电器并插入。这可能导致以下后果：

+   设备的电池开始充电，并持续充电，直到发生其他选项之一

+   设备的电池完全充电，并且设备为了继续运行而消耗的最小电力

+   由于设备故障，设备的电池无法充电

在这里，电源是源，设备是汇，而充电器是使能量从源传输到汇的通道。设备执行的处理或任务是为其电池充电。

好吧，这涵盖了 Iteratee 方法的大部分内容，没有使用任何通常的术语。简单来说，电源代表数据源，充电器充当 Enumerator，而设备充当 Iteratee。

哎呀，我们错过了 Enumeratee！假设常规电源的能量与设备不兼容；在这种情况下，充电器通常有一个内部组件执行这种转换。例如，将交流电（A.C.）转换为直流电（D.C.）。在这种情况下，充电器可以被认为是 Enumerator 和 Enumeratee 的组合。从电源收集能量的组件类似于 Enumerator，而转换能量的另一个组件类似于 Enumeratee。

![数据流处理基础](img/3803OS_06_01.jpg)

Iteratee、Enumerator 和 Enumeratee 的概念起源于 Haskell 库 Iteratee I/O，该库由 Oleg Kiselyov 开发，旨在克服懒 I/O 所面临的问题。

如 Oleg 在其[`okmij.org/ftp/Streams.html`](http://okmij.org/ftp/Streams.html)上的话所说：

> *Enumerator 是数据源的一个封装，一个流生产者——它将 iteratee 应用于流。Enumerator 接收一个 iteratee 并将其应用于正在产生的流数据，直到源耗尽或 iteratee 表示它已经足够。在处理完缓冲区和其他源排空资源后，enumerator 返回 iteratee 的最终值。因此，Enumerator 是一个 iteratee 转换器。*

Iteratees 是流消费者，一个 Iteratee 可以处于以下状态之一：

+   *完成或完成*：Iteratee 已完成处理

+   *继续*：当前元素已被处理，但 Iteratee 还未完成，可以接受下一个元素

+   *错误*：Iteratee 遇到了错误

> *Enumeratee 既是消费者又是生产者，它逐步解码外部流并产生解码数据的嵌套流。*

虽然枚举器知道如何获取下一个元素，但它对 Iteratee 将要对该元素执行的处理一无所知，反之亦然。

不同的库根据这些定义以不同的方式实现 Iteratee、Enumerator 和 Enumeratee。在接下来的章节中，我们将看到它们在 Play Framework 中的实现方式以及如何在我们的应用程序中使用它们。让我们从 Iteratee 开始，因为 Enumerator 需要一个 Iteratee。

# Iteratees

Iteratee 被定义为 trait，`Iteratee[E, +A]`，其中 E 是输入类型，A 是结果类型。Iteratee 的状态由 `Step` 的一个实例表示，它被定义为如下：

```java
sealed trait Step[E, +A] {

  def it: Iteratee[E, A] = this match {
    case Step.Done(a, e) => Done(a, e)
    case Step.Cont(k) => Cont(k)
    case Step.Error(msg, e) => Error(msg, e)
  }

}

object Step {

  //done state of an iteratee
  case class Done+A, E extends Step[E, A]

  //continuing state of an iteratee.
  case class ContE, +A extends Step[E, A]

  //error state of an iteratee
  case class ErrorE extends Step[E, Nothing]
}
```

这里使用的输入代表数据流中的一个元素，它可以是一个空元素、一个元素或文件结束指示符。因此，`Input` 被定义为如下：

```java
sealed trait Input[+E] {
  def mapU): Input[U] = this match {
    case Input.El(e) => Input.El(f(e))
    case Input.Empty => Input.Empty
    case Input.EOF => Input.EOF
  }
}

object Input {

  //An input element
  case class El+E extends Input[E]

  // An empty input
  case object Empty extends Input[Nothing]

  // An end of file input
  case object EOF extends Input[Nothing]

}
```

Iteratee 是一个不可变的数据类型，处理输入的每个结果都是一个具有新状态的新的 Iteratee。

处理 Iteratee 的可能状态时，为每个状态都有一个预定义的辅助对象。它们是：

+   Cont

+   完成

+   错误

让我们看看 `readLine` 方法的定义，它利用了这些对象：

```java
def readLine(line: List[Array[Byte]] = Nil): Iteratee[Array[Byte], String] = Cont {
      case Input.El(data) => {
        val s = data.takeWhile(_ != '\n')
        if (s.length == data.length) {
          readLine(s :: line)
        } else {
          Done(new String(Array.concat((s :: line).reverse: _*), "UTF-8").trim(), elOrEmpty(data.drop(s.length + 1)))
        }
      }
      case Input.EOF => {
        Error("EOF found while reading line", Input.Empty)
      }
      case Input.Empty => readLine(line)
    }
```

`readLine` 方法负责读取一行并返回一个 Iteratee。只要还有更多字节要读取，就会递归调用 `readLine` 方法。在完成处理后，返回一个具有完成状态（Done）的 Iteratee，否则返回一个具有连续状态（Cont）的 Iteratee。如果方法遇到 EOF，则返回一个具有错误状态（Error）的 Iteratee。

此外，Play Framework 还公开了一个配套的 Iteratee 对象，它提供了处理 Iteratee 的辅助方法。通过 Iteratee 对象公开的 API 在 [`www.playframework.com/documentation/2.3.x/api/scala/index.html#play.api.libs.iteratee.Iteratee$`](https://www.playframework.com/documentation/2.3.x/api/scala/index.html#play.api.libs.iteratee.Iteratee$) 中进行了文档说明。

Iteratee 对象也在框架内部使用，以提供一些关键特性。例如，考虑请求体解析器。`BodyParser`对象的`apply`方法定义如下：

```java
def applyT(f: RequestHeader => Iteratee[Array[Byte], Either[Result, T]]): BodyParser[T] = new BodyParser[T] {
    def apply(rh: RequestHeader) = f(rh)
    override def toString = "BodyParser(" + debugName + ")"
  }
```

因此，为了定义`BodyParser[T]`，我们需要定义一个接受`RequestHeader`并返回一个输入为`Array[Byte]`、结果为`Either[Result,T]`的`Iteratee`的方法。

让我们看看一些现有的实现，以了解它是如何工作的。

`RawBuffer`解析器定义如下：

```java
def raw(memoryThreshold: Int): BodyParser[RawBuffer] = BodyParser("raw, memoryThreshold=" + memoryThreshold) { request =>
      import play.core.Execution.Implicits.internalContext
      val buffer = RawBuffer(memoryThreshold)
      Iteratee.foreach[Array[Byte]](bytes => buffer.push(bytes)).map { _ =>
 buffer.close()
 Right(buffer)
 }
    }
```

`RawBuffer`解析器使用`Iteratee.forEach`方法并将接收到的输入推送到缓冲区。

文件解析器定义如下：

```java
def file(to: File): BodyParser[File] = BodyParser("file, to=" + to) { request =>
      import play.core.Execution.Implicits.internalContext
      Iteratee.fold[Array[Byte], FileOutputStream](new FileOutputStream(to)) {
 (os, data) =>
 os.write(data)
 os
 }.map { os =>
 os.close()
 Right(to)
 }
    }
```

文件解析器使用`Iteratee.fold`方法创建接收数据的`FileOutputStream`。

现在，让我们看看 Enumerator 的实现以及这两部分是如何结合在一起的。

# Enumerator

与 Iteratee 类似，**Enumerator**也是通过特性和同名的对象定义的：

```java
trait Enumerator[E] {
  parent =>
  def applyA: Future[Iteratee[E, A]]
  ...
}
object Enumerator{
def applyE: Enumerator[E] = in.length match {
    case 0 => Enumerator.empty
    case 1 => new Enumerator[E] {
      def applyA: Future[Iteratee[E, A]] = i.pureFoldNoEC {
        case Step.Cont(k) => k(Input.El(in.head))
        case _ => i
      }
    }
    case _ => new Enumerator[E] {
      def applyA: Future[Iteratee[E, A]] = enumerateSeq(in, i)
    }
  }
...
}
```

注意到特性和其伴生对象的`apply`方法不同。特质的`apply`方法接受`Iteratee[E, A]`并返回`Future[Iteratee[E, A]]`，而伴生对象的`apply`方法接受类型为`E`的序列并返回`Enumerator[E]`。

现在，让我们使用伴生对象的`apply`方法定义一个简单的数据流；首先，获取给定`(Seq[String])`行中的字符计数：

```java
  val line: String = "What we need is not the will to believe, but the wish to find out."
  val words: Seq[String] = line.split(" ")

  val src: Enumerator[String] = Enumerator(words: _*)

  val sink: Iteratee[String, Int] = Iteratee.foldString, Int((x, y) => x + y.length)
  val flow: Future[Iteratee[String, Int]] = src(sink)

  val result: Future[Int] = flow.flatMap(_.run)
```

变量`result`具有`Future[Int]`类型。我们现在可以处理它以获取实际计数。

在前面的代码片段中，我们通过以下步骤获取结果：

1.  使用伴生对象的`apply`方法构建 Enumerator：

    ```java
    val src: Enumerator[String] = Enumerator(words: _*)
    ```

1.  通过将枚举器绑定到 Iteratee 获取`Future[Iteratee[String, Int]]`：

    ```java
    val flow: Future[Iteratee[String, Int]] = src(sink)
    ```

1.  展平`Future[Iteratee[String,Int]]`并处理它：

    ```java
    val result: Future[Int] = flow.flatMap(_.run)
    ```

1.  从`Future[Int]`获取结果：

幸运的是，Play 提供了一个快捷方法，通过合并步骤 2 和 3，这样我们就不必每次都重复相同的过程。该方法由|`>>>`符号表示。使用快捷方法，我们的代码简化为如下：

```java
val src: Enumerator[String] = Enumerator(words: _*)
val sink: Iteratee[String, Int] = Iteratee.foldString, Int((x, y) => x + y.length)
val result: Future[Int] = src |>>> sink
```

当我们可以直接使用数据类型的方法时，为什么还要使用这个方法？在这种情况下，我们是否使用`String`的`length`方法来获取相同的值（通过忽略空白字符）？

在这个例子中，我们获取数据作为一个单独的`String`，但这不会是唯一的情况。我们需要处理连续数据的方法，例如文件上传，或从各种网络站点获取数据，等等。

例如，假设我们的应用程序从连接到它的所有设备（如摄像头、温度计等）以固定间隔接收心跳。我们可以使用`Enumerator.generateM`方法模拟数据流：

```java
val dataStream: Enumerator[String] = Enumerator.generateM {
  Promise.timeout(Some("alive"), 100 millis)
}
```

在前面的代码片段中，每 100 毫秒产生一次 `"alive"` 字符串。传递给 `generateM` 方法的函数在 Iteratee 绑定到 Enumerator 时处于 `Cont` 状态时被调用。此方法用于内部构建枚举器，并在我们想要分析预期数据流的处理时很有用。

可以从文件、`InputStream` 或 `OutputStream` 创建 Enumerator。Enumerators 可以连接或交错。Enumerator API 在 [`www.playframework.com/documentation/2.3.x/api/scala/index.html#play.api.libs.iteratee.Enumerator$`](https://www.playframework.com/documentation/2.3.x/api/scala/index.html#play.api.libs.iteratee.Enumerator$) 中进行了文档说明。

## 使用 Concurrent 对象

`Concurrent` 对象是一个辅助工具，它提供了使用 Iteratees、枚举器和 Enumeratees 并发的实用工具。它的重要方法有两个：

+   **单播**：当需要向单个 iterate 发送数据时很有用。

+   **广播**：它便于将相同的数据并发发送到多个 Iteratee。

### 单播

例如，上一节中的字符计数示例可以如下实现：

```java
  val unicastSrc = Concurrent.unicastString
  )

  val unicastResult: Future[Int] = unicastSrc |>>> sink
```

`unicast` 方法接受 `onStart`、`onError` 和 `onComplete` 处理器。在前面的代码片段中，我们提供了强制性的 `onStart` 方法。unicast 的签名如下：

```java
def unicastE ⇒ Unit,
    onComplete: ⇒ Unit = (),
    onError: (String, Input[E]) ⇒ Unit = (_: String, _: Input[E]) => ())(implicit ec: ExecutionContext): Enumerator[E] {…}
```

因此，为了添加错误日志，我们可以定义 `onError` 处理器如下：

```java
val unicastSrc2 = Concurrent.unicastString,
    onError = { (msg, str) => Logger.error(s"encountered $msg for $str")}
      )
```

现在，让我们看看广播是如何工作的。

### 广播

`broadcast[E]` 方法创建一个枚举器和通道，并返回一个 `(Enumerator[E], Channel[E])` 元组。因此获得的枚举器和通道可以用来向多个 Iteratee 广播数据：

```java
  val (broadcastSrc: Enumerator[String], channel: Concurrent.Channel[String]) = Concurrent.broadcast[String]  
  private val vowels: Seq[Char] = Seq('a', 'e', 'i', 'o', 'u')

  def getVowels(str: String): String = {
    val result = str.filter(c => vowels.contains(c))
    result
  }

  def getConsonants(str: String): String = {
    val result = str.filterNot(c => vowels.contains(c))
    result
  }

  val vowelCount: Iteratee[String, Int] = Iteratee.foldString, Int((x, y) => x + getVowels(y).length)

  val consonantCount: Iteratee[String, Int] = Iteratee.foldString, Int((x, y) => x + getConsonants(y).length)

  val vowelInfo: Future[Int]  = broadcastSrc |>>> vowelCount
  val consonantInfo: Future[Int]  = broadcastSrc |>>> consonantCount

  words.foreach(w => channel.push(w))
  channel.end()

  vowelInfo onSuccess { case count => println(s"vowels:$count")}
  consonantInfo onSuccess { case count => println(s"consonants:$count")}
```

# Enumeratees

**Enumeratee** 也使用具有相同 `Enumeratee` 名称的特性和伴随对象来定义。

它的定义如下：

```java
trait Enumeratee[From, To] {
...
def applyOnA: Iteratee[From, Iteratee[To, A]]

def applyA: Iteratee[From, Iteratee[To, A]] = applyOnA
...
}
```

Enumeratee 将其作为输入给出的 Iteratee 进行转换，并返回一个新的 Iteratee。让我们看看通过实现 `applyOn` 方法定义 Enumeratee 的方法。Enumeratee 的 `flatten` 方法接受 `Future[Enumeratee]` 并返回另一个 Enumeratee，其定义如下：

```java
  def flattenFrom, To = new Enumeratee[From, To] {
    def applyOnA: Iteratee[From, Iteratee[To, A]] =
      Iteratee.flatten(futureOfEnumeratee.map(_.applyOnA)(dec))
  }
```

在前面的代码片段中，`applyOn` 方法被调用在传递了其未来的 Enumeratee 上，而 `dec` 是 `defaultExecutionContext`。

使用伴随对象定义 Enumeratee 要简单得多。伴随对象有许多处理 Enumeratees 的方法，例如 map、transform、collect、take、filter 等。API 在 [`www.playframework.com/documentation/2.3.x/api/scala/index.html#play.api.libs.iteratee.Enumeratee$`](https://www.playframework.com/documentation/2.3.x/api/scala/index.html#play.api.libs.iteratee.Enumeratee$) 中进行了文档说明。

让我们通过解决一个问题来定义一个`Enumeratee`。我们在上一节中使用的示例，用于查找元音和辅音的数量，如果句子中的元音被大写，则不会正确工作，即当`line`变量定义为以下内容时，`src |>>> vowelCount`的结果将是错误的：

```java
val line: String = "What we need is not the will to believe, but the wish to find out.".toUpperCase
```

为了解决这个问题，让我们将数据流中所有字符的大小写更改为小写。我们可以使用`Enumeratee`来更新提供给`Iteratee`的输入。

现在，让我们定义一个`Enumeratee`，它返回给定的小写字符串：

```java
  val toSmallCase: Enumeratee[String, String] = Enumeratee.map[String] {
    s => s.toLowerCase
  }
```

有两种方法可以将`Enumeratee`添加到数据流中。它可以绑定到以下内容：

+   枚举器

+   `Iteratee`

## 将一个`Enumeratee`绑定到一个枚举器上

可以通过枚举器的`through`方法将`Enumeratee`绑定到枚举器上，该方法返回一个新的枚举器，并使用给定的`Enumeratee`进行组合。

更新示例以包括`Enumeratee`，我们得到以下内容：

```java
val line: String = "What we need is not the will to believe, but the wish to find out.".toUpperCase
val words: Seq[String] = line.split(" ")

val src: Enumerator[String] = Enumerator(words: _*)

private val vowels: Seq[Char] = Seq('a', 'e', 'i', 'o', 'u')
def getVowels(str: String): String = {
  val result = str.filter(c => vowels.contains(c))
  result
}

src.through(toSmallCase) |>>> vowelCount

```

`through`方法是对`&>`方法的别名，它是为枚举器定义的，因此最后的语句也可以重写为以下内容：

```java
src &> toSmallCase |>>> vowelCount
```

## 将一个`Enumeratee`绑定到一个`Iteratee`上

现在，让我们通过将`Enumeratee`绑定到`Iteratee`上来实现相同的流程。这可以通过使用`Enumeratee`的`transform`方法来完成。`transform`方法将给定的`Iteratee`转换，并产生一个新的`Iteratee`。根据这一点修改流程，我们得到以下内容：

```java
src |>>> toSmallCase.transform(vowelCount)
```

`Enumeratee`的`transform`方法有一个`&>>`符号别名。使用这个别名，我们可以将流程重写如下：

```java
src |>>> toSmallCase &>> vowelCount
```

除了`Enumeratee`可以绑定到枚举器或`Iteratee`之外，如果其中一个的输出类型与另一个的输入类型相同，不同的`Enumeratee`也可以组合。例如，假设我们有一个`filterVowel` `Enumeratee`，它过滤掉元音，如下面的代码所示：

```java
val filterVowel: Enumeratee[String, String] = Enumeratee.map[String] {
  str => str.filter(c => vowels.contains(c))
}
```

`toSmallCase`和`filterVowel`的组合是可能的，因为`toSmallCase`的输出类型是`String`，而`filterVowel`的输入类型也是`String`。为此，我们使用`Enumeratee`的`compose`方法：

```java
toSmallCase.compose(filterVowel)
```

现在，让我们使用以下方式重写流程：

```java
src |>>> toSmallCase.compose(filterVowel) &>> sink
```

在这里，`sink`被定义为以下内容：

```java
  val sink: Iteratee[String, Int] = Iteratee.foldString, Int((x, y) => x + y.length)
```

与`transform`和`compose`方法一样，这也具有`><>`符号别名。让我们使用所有符号而不是方法名来定义以下流程：

```java
src |>>> toSmallCase ><> filterVowel &>> sink
```

我们可以添加另一个`Enumeratee`，它计算`String`的长度并使用`Iteratee`，它简单地求和长度：

```java
  val toInt: Enumeratee[String, Int] = Enumeratee.map[String] {
    str => str.length
  }
  val sum: Iteratee[Int, Int] = Iteratee.foldInt, Int((x, y) => x + y)
  src |>>> toSmallCase ><> filterVowel ><> toInt &>> sum

```

![将一个`Enumeratee`绑定到一个`Iteratee`上](img/3803OS_06_02.jpg)

在前面的代码片段中，我们必须使用一个接受`Int`类型数据的不同迭代器，因为我们的`toInt` `Enumeratee`将`String`输入转换为`Int`。

本章到此结束。定义几个数据流以熟悉 API。从简单的数据流开始，例如提取给定段落中的所有数字或单词，然后逐步复杂化。

# 摘要

在本章中，我们讨论了迭代器（Iteratees）、枚举器（Enumerators）和枚举者（Enumeratees）的概念。我们还看到了它们如何在 Play 框架中实现并被内部使用。本章还通过一个简单的示例向您展示了如何使用 Play 框架公开的 API 定义数据流。

在下一章中，我们将通过一个全局插件来探索 Play 应用程序提供的功能。
