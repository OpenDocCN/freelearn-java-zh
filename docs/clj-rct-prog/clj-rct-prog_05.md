# 第五章。使用 core.async 创建您自己的 CES 框架

在上一章中，提到`core.async`与其他框架如 RxClojure 或 RxJava 相比，在抽象级别上更低。

这是因为大多数时候我们必须仔细思考我们创建的通道以及要使用的缓冲区类型和大小，是否需要 pub/sub 功能，等等。

然而，并非所有应用程序都需要这种级别的控制。现在我们已经熟悉了`core.async`的动机和主要抽象，我们可以开始编写一个使用`core.async`作为底层基础的简单 CES 框架。

通过这样做，我们避免了需要考虑线程池管理，因为框架为我们处理了这一点。

在本章中，我们将涵盖以下主题：

+   使用`core.async`作为其底层并发策略构建 CES 框架

+   构建一个使用我们的 CES 框架的应用程序

+   理解到目前为止所提出的不同方法的权衡

# 一个最小化的 CES 框架

在我们深入细节之前，我们应该在较高层面上定义什么是*最小化*。

让我们从我们的框架将提供的两个主要抽象开始：行为和事件流。

如果你还记得第一章中的内容，*什么是响应式编程？*，行为代表连续、随时间变化的值，如时间或鼠标位置行为。另一方面，事件流代表在时间点 T 的离散事件，如按键。

接下来，我们应该考虑我们希望支持哪些类型的操作。行为相当简单，所以至少我们需要：

+   创建新的行为

+   获取行为当前值

+   将行为转换为事件流

事件流有更复杂的逻辑，我们至少应该支持以下操作：

+   将值推送到流中

+   从给定的时间间隔创建流

+   使用`map`和`filter`操作转换流

+   使用`flatmap`组合流

+   订阅到流

这是一个小的子集，但足够大，可以展示 CES 框架的整体架构。一旦完成，我们将用它来构建一个简单的示例。

## Clojure 还是 ClojureScript？

在这里，我们将转换方向，并给我们的小库添加另一个要求：它应该既能在 Clojure 中运行，也能在 ClojureScript 中运行。起初，这可能会听起来像是一个艰巨的要求。然而，记住`core.async`——我们框架的基础——既可以在 JVM 上运行，也可以在 JavaScript 中运行。这意味着我们有很多工作要做来让它实现。

然而，这意味着我们需要能够从我们的库中生成两个工件：一个`jar`文件和一个 JavaScript 文件。幸运的是，有 leiningen 插件可以帮助我们做到这一点，我们将使用其中几个：

+   `lein-cljsbuild`（见 [`github.com/emezeske/lein-cljsbuild`](https://github.com/emezeske/lein-cljsbuild)）：用于简化 ClojureScript 构建的 Leiningen 插件

+   `cljx`（见 [`github.com/lynaghk/cljx`](https://github.com/lynaghk/cljx)）：一个用于编写可移植 Clojure 代码库的预处理器，即编写一个文件并输出 `.clj` 和 `.cljs` 文件

您不需要详细了解这些库。我们只使用它们的基本功能，并在遇到时会解释各个部分。

让我们开始创建一个新的 leiningen 项目。我们将我们的框架命名为 respondent——reactive 这个词的许多同义词之一：

```java
$ lein new respondent

```

我们需要对 `project.clj` 文件进行一些修改，以包含我们将要使用的依赖项和配置。首先，确保项目依赖项看起来如下所示：

```java
:dependencies [[org.clojure/clojure "1.5.1"]
               [org.clojure/core.async "0.1.303.0-886421-alpha"]
               [org.clojure/clojurescript "0.0-2202"]]
```

这里不应该有任何惊喜。仍然在项目文件中，添加必要的插件：

```java
:plugins [[com.keminglabs/cljx "0.3.2"]
          [lein-cljsbuild "1.0.3"]]
```

这些是我们之前提到的插件。它们本身并不做什么，但是需要配置。

对于 `cljx`，请在项目文件中添加以下内容：

```java
:cljx {:builds [{:source-paths ["src/cljx"]
                 :output-path "target/classes"
                 :rules :clj}

                {:source-paths ["src/cljx"]
                 :output-path "target/classes"
                 :rules :cljs}]}
  :hooks [cljx.hooks]
```

```java
cljx allows us to write code that is portable between Clojure and ClojureScript by placing annotations its preprocessor can understand. We will see later what these annotations look like, but this chunk of configuration tells cljx where to find the annotated files and where to output them once they're processed.
```

例如，根据前面的规则，如果我们有一个名为 `src/cljx/core.cljx` 的文件，并且运行预处理器，我们最终会得到 `target/classes/core.clj` 和 `target/classes/core.cljs` 输出文件。另一方面，钩子（hooks）只是自动在启动 REPL 会话时运行 `cljx` 的便捷方式。

配置的下一部分是针对 `cljsbuild` 的：

```java
:cljsbuild
{:builds [{:source-paths ["target/classes"]
           :compiler {:output-to "target/main.js"}}]}
```

`cljsbuild` 提供了 leiningen 任务，用于将 Clojuresript 源代码编译成 JavaScript。根据我们之前的 `cljx` 配置，我们知道 `source.cljs` 文件将位于 `target/classes` 目录下，所以我们只是告诉 `cljsbuild` 编译该目录下的所有 ClojureScript 文件，并将内容输出到 `target/main.js`。这是项目文件中需要的最后一部分。

好的，请继续删除由 leiningen 模板创建的这些文件，因为我们不会使用它们：

```java
$ rm src/respondent/core.clj
$ rm test/respondent/core_test.clj

```

然后，在 `src/cljx/respondent/` 下创建一个新的 `core.cljx` 文件，并添加以下命名空间声明：

```java
(ns respondent.core
  (:refer-clojure :exclude [filter map deliver])

  #+clj
  (:import [clojure.lang IDeref])

  #+clj
  (:require [clojure.core.async :as async
             :refer [go go-loop chan <! >! timeout
                     map> filter> close! mult tap untap]])
  #+cljs
  (:require [cljs.core.async :as async
             :refer [chan <! >! timeout map> filter>
                     close! mult tap untap]])

  #+cljs
  (:require-macros [respondent.core :refer [behavior]]
                   [cljs.core.async.macros :refer [go go-loop]]))
```

在这里，我们开始看到 `cljx` 注释。`cljx` 只是一个文本预处理器，所以当它使用 `clj` 规则处理文件时——如配置中所示——它将在输出文件中保留由注释 `#+clj` 前缀的 `s-` 表达式，同时删除由 `#+cljs` 前缀的。当使用 `cljs` 规则时，发生相反的过程。

这是因为宏需要在 JVM 上编译，所以它们必须通过使用 `:require-macros` 命名空间选项单独包含。不要担心我们之前没有遇到的 `core.async` 函数；我们将在使用它们构建我们的框架时解释它们。

此外，请注意我们如何排除 Clojure 标准 API 中的函数，因为我们希望使用相同的名称，并且不希望有任何不希望的命名冲突。

本节为我们设置了一个新项目以及我们框架所需的插件和配置。我们准备好开始实现了。

## 设计公共 API

我们对行为的要求之一是将给定行为转换为事件流的能力。一种常见的方法是在特定间隔采样行为。如果我们以*鼠标位置*行为为例，通过每*x*秒采样一次，我们得到一个事件流，该事件流将在离散的时间点发出当前鼠标位置。

这导致以下协议：

```java
(defprotocol IBehavior
  (sample [b interval]
    "Turns this Behavior into an EventStream from the sampled values at the given interval"))
```

它有一个名为`sample`的函数，我们在前面的代码中已经描述过。我们还需要对行为做更多的事情，但现在这已经足够了。

我们下一个主要抽象是`EventStream`，根据之前看到的需要，导致以下协议：

```java
(defprotocol IEventStream
  (map        [s f]
    "Returns a new stream containing the result of applying f
    to the values in s")
  (filter     [s pred]
    "Returns a new stream containing the items from s
    for which pred returns true")
  (flatmap    [s f]
    "Takes a function f from values in s to a new EventStream.
    Returns an EventStream containing values from all underlying streams combined.")
  (deliver    [s value]
    "Delivers a value to the stream s")
  (completed? [s]
    "Returns true if this stream has stopped emitting values. False otherwise."))
```

这为我们提供了一些基本函数来转换和查询事件流。但它确实省略了订阅流的能力。不用担心，我没有忘记它！

虽然，订阅事件流是常见的，但协议本身并没有强制要求这样做，这是因为这种操作最适合它自己的协议：

```java
(defprotocol IObservable
  (subscribe [obs f] "Register a callback to be invoked when the underlying source changes.
   Returns a token the subscriber can use to cancel the subscription."))
```

关于订阅，有一个从流中取消订阅的方法是有用的。这可以以几种方式实现，但前面函数的`docstring`暗示了一种特定的方法：一个可以用来取消订阅流的令牌。这导致我们最后的协议：

```java
(defprotocol IToken
  (dispose [tk]
    "Called when the subscriber isn't interested in receiving more items"))
```

## 实现令牌

令牌类型在整个框架中是最简单的，因为它只有一个具有直接实现的函数：

```java
(deftype Token [ch]
  IToken
  (dispose [_]
    (close! ch)))
```

它只是关闭它所给的任何通道，停止事件通过订阅流动。

## 实现事件流

另一方面，事件流实现是我们框架中最复杂的。我们将逐步解决它，在前进的过程中实现和实验。

首先，让我们看看我们的主要构造函数，`event-stream`：

```java
(defn event-stream
  "Creates and returns a new event stream. You can optionally provide an existing
  core.async channel as the source for the new stream"
  ([]
     (event-stream (chan)))
  ([ch]
     (let [multiple  (mult ch)
           completed (atom false)]
       (EventStream. ch multiple completed))))
```

`docstring`应该足以理解公共 API。然而，可能不清楚所有构造函数参数的含义。从左到右，`EventStream`的参数是：

+   `ch`: 这是支持此流的`core.async`通道。

+   `multiple`: 这是一种将信息从一条通道广播到许多其他通道的方式。它是我们将要解释的`core.async`概念之一。

+   `completed`: 一个布尔标志，表示此事件流是否已完成且不会发出任何新值。

从实现中，你可以看到`multiple`是从支持流的通道创建的。`multiple`有点像广播。考虑以下示例：

```java
  (def in (chan))
  (def multiple (mult in))

  (def out-1 (chan))
  (tap multiple out-1)

  (def out-2 (chan))
  (tap multiple out-2)
  (go (>! in "Single put!"))

  (go (prn "Got from out-1 " (<! out-1)))
  (go (prn "Got from out-2 " (<! out-2)))
```

```java
in, and mult of it called multiple. Then, we create two output channels, out-1 and out-2, which are both followed by a call to tap. This essentially means that whatever values are written to in will be taken by multiple and written to any channels tapped into it as the following output shows:
```

```java
"Got from out-1 " "Single put!"
"Got from out-2 " "Single put!"
```

这将使理解`EventStream`实现更容易。

接下来，让我们看看`EventStream`的最小实现是什么样的——确保实现位于前面描述的构造函数之前：

```java
(declare event-stream)

(deftype EventStream [channel multiple completed]
  IEventStream
  (map [_ f]
    (let [out (map> f (chan))]
      (tap multiple out)
      (event-stream out)))

  (deliver [_ value]
    (if (= value ::complete)
      (do (reset! completed true)
          (go (>! channel value)
              (close! channel)))
      (go (>! channel value))))

  IObservable
  (subscribe [this f]
    (let [out (chan)]
      (tap multiple out)
      (go-loop []
        (let [value (<! out)]
          (when (and value (not= value ::complete))
            (f value)
            (recur))))
      (Token. out))))
```

目前，我们选择只实现 `IEventStream` 协议中的 `map` 和 `deliver` 函数。这允许我们将值传递到流中，并转换这些值。

然而，如果我们不能检索到传递的值，这将不会非常有用。这就是为什么我们还实现了 `IObservable` 协议中的 `subscribe` 函数。

简而言之，`map` 需要从输入流中取一个值，对其应用一个函数，并将其发送到新创建的流中。我们通过创建一个连接到当前 `multiple` 的输出通道来完成这个操作。然后我们使用这个通道来支持新的事件流。

`deliver` 函数简单地将值放入支持通道。如果值是命名空间关键字 `::complete`，我们更新 `completed` 原子并关闭支持通道。这确保流不会发出任何其他值。

最后，我们有 `subscribe` 函数。订阅者被通知的方式是通过使用连接到后端 `multiple` 的输出通道。我们无限循环调用订阅函数，每当发出新值时。

我们通过返回一个令牌来完成，这个令牌会在处置后关闭输出通道，导致 `go-loop` 停止。

让我们通过在 REPL 中进行几个示例实验来确保所有这些内容都很有意义：

```java
  (def es1 (event-stream))
  (subscribe es1 #(prn "first event stream emitted: " %))
  (deliver es1 10)
  ;; "first event stream emitted: " 10

  (def es2 (map es1 #(* 2 %)))
  (subscribe es2 #(prn "second event stream emitted: " %))

  (deliver es1 20)
  ;; "first event stream emitted: " 20
  ;; "second event stream emitted: " 40
```

太棒了！我们已经实现了 `IEventStream` 协议的最小、工作版本！

我们接下来要实现的下一个函数是 `filter`，它与 `map` 非常相似：

```java
  (filter [_ pred]
    (let [out (filter> pred (chan))]
      (tap multiple out)
      (event-stream out)))
```

唯一的区别是我们使用 `filter>` 函数，而 `pred` 应该是一个布尔函数：

```java
  (def es1 (event-stream))
  (def es2 (filter es1 even?))
  (subscribe es1 #(prn "first event stream emitted: " %))
  (subscribe es2 #(prn "second event stream emitted: " %))

  (deliver es1 2)
  (deliver es1 3)
  (deliver es1 4)

  ;; "first event stream emitted: " 2
  ;; "second event stream emitted: " 2
  ;; "first event stream emitted: " 3
  ;; "first event stream emitted: " 4
  ;; "second event stream emitted: " 4
```

正如我们所看到的，`es2` 只在值是偶数时发出新值。

### 小贴士

如果你正在逐步跟随示例进行输入，每次我们在任何 `deftype` 定义中添加新函数时，你都需要重新启动你的 REPL。这是因为 `deftype` 在评估时会生成和编译一个 Java 类。因此，仅仅重新加载命名空间是不够的。

或者，你可以使用像 `tools.namespace`（见 [`github.com/clojure/tools.namespace`](https://github.com/clojure/tools.namespace)）这样的工具，它解决了一些这些 REPL 重新加载的限制。

在我们的列表中向下移动，我们现在有 `flatmap`：

```java
(flatmap [_ f]
    (let [es (event-stream)
          out (chan)]
      (tap multiple out)
      (go-loop []
        (when-let [a (<! out)]
          (let [mb (f a)]
            (subscribe mb (fn [b]
                            (deliver es b)))
            (recur))))
      es))
```

我们在调查反应式扩展时已经遇到过这个操作符。这是我们的文档字符串对它的描述：

*将函数 f 从 s 中的值映射到新的 EventStream。*

*返回一个包含所有底层流值的 EventStream。*

这意味着 `flatmap` 需要将所有可能的事件流合并为单个输出事件流。和之前一样，我们向 `multiple` 流中连接一个新的通道，但然后我们在输出通道上循环，对每个输出值应用 `f`。

然而，正如我们所看到的，`f` 本身返回一个新的事件流，所以我们只需订阅它。每当注册在订阅中的函数被调用时，我们将该值传递到输出事件流中，实际上是将所有流合并为一个。

考虑以下示例：

```java
  (defn range-es [n]
    (let [es (event-stream (chan n))]
      (doseq [n (range n)]
        (deliver es n))
      es))

  (def es1 (event-stream))
  (def es2 (flatmap es1 range-es))
  (subscribe es1 #(prn "first event stream emitted: " %))
  (subscribe es2 #(prn "second event stream emitted: " %))

  (deliver es1 2)
  ;; "first event stream emitted: " 2
  ;; "second event stream emitted: " 0
  ;; "second event stream emitted: " 1

  (deliver es1 3)
  ;; "first event stream emitted: " 3
  ;; "second event stream emitted: " 0
  ;; "second event stream emitted: " 1
  ;; "second event stream emitted: " 2
```

我们有一个名为 `range-es` 的函数，它接收一个数字 `n` 并返回一个事件流，该流从 `0` 到 `n` 发射数字。和之前一样，我们有一个起始流 `es1` 和通过 `flatmap` 创建的转换流 `es2`。

从前面的输出中我们可以看到，由 `range-es` 创建的流被扁平化为 `es2`，这使得我们只需订阅一次就能接收所有值。

这就留下了我们从 `IEventStream` 中需要实现的单一函数：

```java
  (completed? [_] @completed)
```

`completed?` 简单地返回 `completed` 原子的当前值。我们现在可以开始实现行为。

## 实现行为

如果你还记得，`IBehavior` 协议有一个名为 `sample` 的单一函数，其文档字符串说明：*将此行为转换为从给定间隔的采样值生成的事件流*。

为了实现 `sample`，我们首先将创建一个有用的辅助函数，我们将称之为 `from-interval`：

```java
(defn from-interval
  "Creates and returns a new event stream which emits values at the given
interval.
  If no other arguments are given, the values start at 0 and increment by
one at each delivery.

  If given seed and succ it emits seed and applies succ to seed to get
the next value. It then applies succ to the previous result and so on."
  ([msecs]
     (from-interval msecs 0 inc))
  ([msecs seed succ]
     (let [es (event-stream)]
       (go-loop [timeout-ch (timeout msecs)
                 value seed]
         (when-not (completed? es)
           (<! timeout-ch)
           (deliver es value)
           (recur (timeout msecs) (succ value))))
       es)))
```

在这个阶段，`docstring` 函数应该足够清晰，但我们希望通过在 REPL 中尝试它来确保我们正确理解其行为：

```java
  (def es1 (from-interval 500))
  (def es1-token (subscribe es1 #(prn "Got: " %)))
  ;; "Got: " 0
  ;; "Got: " 1
  ;; "Got: " 2
  ;; "Got: " 3
  ;; ...
  (dispose es1-token)
```

如预期，`es1` 以 500 毫秒的间隔从零开始发射整数。默认情况下，它将无限期地发射数字；因此，我们保留调用 `subscribe` 返回的令牌的引用。

这样我们就可以在完成时将其丢弃，导致 `es-1` 完成并停止发射项目。

接下来，我们最终可以实现 `Behavior` 类型：

```java
(deftype Behavior [f]
  IBehavior
  (sample [_ interval]
    (from-interval interval (f) (fn [& args] (f))))
  IDeref
  (#+clj deref #+cljs -deref [_]
    (f)))

(defmacro behavior [& body]
  `(Behavior. #(do ~@body)))
```

通过传递一个函数来创建行为。你可以将这个函数视为一个生成器，负责生成此事件流中的下一个值。

这个生成器函数将在我们（1）`deref` `Behavior` 或（2）在 `sample` 给定的间隔时被调用。

`behavior` 宏是为了方便而存在的，它允许我们创建一个新的 `Behavior`，而无需我们自己将主体包装在函数中：

```java
  (def time-behavior (behavior (System/nanoTime)))

  @time-behavior
  ;; 201003153977194

  @time-behavior
  ;; 201005133457949
```

在前面的例子中，我们定义了 `time-behavior`，它始终包含当前系统时间。然后我们可以通过使用 `sample` 函数将此行为转换为离散事件的流：

```java
  (def time-stream (sample time-behavior 1500))
  (def token       (subscribe time-stream #(prn "Time is " %)))
  ;; "Time is " 201668521217402
  ;; "Time is " 201670030219351
  ;; ...

  (dispose token)
```

### 小贴士

总是记得在处理像 `sample` 和 `from-interval` 创建的无穷流时保留订阅令牌的引用，否则你可能会遇到不希望的内存泄漏。

恭喜！我们已经使用 `core.async` 创建了一个工作、最小化的 CES 框架！

然而，我们没有证明它可以用 ClojureScript 实现，这最初是主要要求之一。没关系。我们将通过开发一个简单的 ClojureScript 应用程序来解决它，该应用程序将使用我们新的框架。

为了做到这一点，我们需要将框架部署到我们的本地 Maven 仓库。从项目根目录，输入以下 `lein` 命令：

```java
$ lein install
Rewriting src/cljx to target/classes (clj) with features #{clj} and 0 transformations.
Rewriting src/cljx to target/classes (cljs) with features #{cljs} and 1 transformations.
Created respondent/target/respondent-0.1.0-SNAPSHOT.jar
Wrote respondent/pom.xml

```

# 练习

以下几节有一些练习供你完成。

## 练习 5.1

将我们当前的 `EventStream` 实现扩展以包含一个名为 `take` 的函数。它的工作方式与 Clojure 的核心 `take` 函数对序列的处理非常相似：它将从底层事件流中取出 `n` 个项目，之后将停止发出项目。

这里展示了从原始事件流中发出的前五个项目的示例交互：

```java
(def es1 (from-interval 500))
(def take-es (take es1 5))

(subscribe take-es #(prn "Take values: " %))

;; "Take values: " 0
;; "Take values: " 1
;; "Take values: " 2
;; "Take values: " 3
;; "Take values: " 4
```

### 提示

在这里保留一些状态可能是有用的。原子可以有所帮助。此外，尝试想出一种方法来处理任何由解决方案要求的未使用的订阅。

## 练习 5.2

在这个练习中，我们将添加一个名为 `zip` 的函数，该函数将两个不同事件流发出的项目组合到一个向量中。

与 `zip` 函数的一个示例交互如下：

```java
(def es1 (from-interval 500))
(def es2 (map (from-interval 500) #(* % 2)))
(def zipped (zip es1 es2))

(def token (subscribe zipped #(prn "Zipped values: " %)))

;; "Zipped values: " [0 0]
;; "Zipped values: " [1 2]
;; "Zipped values: " [2 4]
;; "Zipped values: " [3 6]
;; "Zipped values: " [4 8]

(dispose token)
```

### 提示

对于这个练习，我们需要一种方式来知道我们是否有足够的项目可以从两个事件流中发出。最初管理这种内部状态可能很棘手。Clojure 的 `ref` 类型，特别是 `dosync`，可能会有所帮助。

# 一个响应式应用程序

如果我们没有通过在新的应用程序中部署和使用新框架的整个开发生命周期来完善这一章，那么这一章将是不完整的。这正是本节的目的。

我们将要构建的应用程序非常简单。它所做的只是跟踪鼠标的位置，使用我们构建到响应式程序中的反应式原语。

为了达到这个目的，我们将使用由 Mimmo Cosenza 创建的出色的 lein 模板 `cljs-start`（见 [`github.com/magomimmo/cljs-start`](https://github.com/magomimmo/cljs-start)），以帮助开发者开始使用 ClojureScript。

让我们开始吧：

```java
lein new cljs-start respondent-app

```

接下来，让我们修改项目文件以包含以下依赖项：

```java
[clojure-reactive-programming/respondent "0.1.0-SNAPSHOT"]
[prismatic/dommy "0.1.2"]
```

第一个依赖项是显而易见的。它只是我们自己的框架。`dommy` 是一个用于 ClojureScript 的 DOM 操作库。当构建我们的网页时，我们将简要使用它。

接下来，编辑 `dev-resources/public/index.html` 文件以匹配以下内容：

```java
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">

    <title>Example: tracking mouse position</title>
    <!--[if lt IE 9]>
    <script src="img/html5.js"></script>
    <![endif]-->
</head>

<body>
    <div id="test">
        <h1>Mouse (x,y) coordinates:</h1>
    </div>
    <div id="mouse-xy">
      (0,0)
    </div>
    <script src="img/respondent_app.js"></script>
</body>
</html>
```

```java
 created a new div element, which will contain the mouse position. It defaults to (0,0).
```

最后一个拼图是修改 `src/cljs/respondent_app/core.cljs` 以匹配以下内容：

```java
 (ns respondent-app.core
  (:require [respondent.core :as r]
            [dommy.core :as dommy])
  (:use-macros
   [dommy.macros :only [sel1]]))

(def mouse-pos-stream (r/event-stream))
(set! (.-onmousemove js/document)
      (fn [e]
        (r/deliver mouse-pos-stream [(.-pageX e) (.-pageY e)])))

(r/subscribe mouse-pos-stream
             (fn [[x y]]
               (dommy/set-text! (sel1 :#mouse-xy)
                                (str "(" x "," y ")"))))
```

这是我们的主要应用程序逻辑。它创建了一个事件流，我们将从浏览器窗口的 `onmousemove` 事件中传递当前的鼠标位置到这个事件流。

之后，我们只需简单地订阅它，并使用 `dommy` 来选择和设置我们之前添加的 `div` 元素的文本。

我们现在可以使用应用程序了！让我们首先编译 ClojureScript：

```java
$ lein compile

```

这可能需要几秒钟。如果一切顺利，接下来要做的事情是启动一个 REPL 会话并启动服务器：

```java
$ lein repl
user=> (run)

```

现在，将你的浏览器指向 `http://localhost:3000/` 并拖动鼠标以查看其当前位置。

恭喜你成功开发、部署和使用你自己的 CES 框架！

# CES 与 core.async

在这个阶段，你可能想知道何时选择一种方法而不是另一种方法。毕竟，正如本章开头所展示的，我们可以使用 `core.async` 来完成使用 `respondent` 所做的一切。

这一切都归结于使用适合当前任务的正确抽象级别。

`core.async`为我们提供了许多在处理需要相互通信的进程时非常有用的低级原语。`core.async`的通道作为并发阻塞队列工作，在这些场景中是一个出色的同步机制。

然而，它使得其他解决方案的实现更加困难：例如，通道默认是单次取用的，所以如果我们有多个消费者对放入通道中的值感兴趣，我们必须使用`mult`和`tap`等工具自行实现分发。

另一方面，CES 框架在更高的抽象级别上运行，并且默认情况下与多个订阅者一起工作。

此外，`core.async`依赖于副作用，这可以通过`go`块内使用`>!`等函数来看到。例如，RxClojure 这样的框架通过使用纯函数来促进流转换。

这并不是说 CES 框架中没有副作用。当然有。然而，作为库的消费者，这些副作用大部分对我们来说是隐藏的，这让我们认为大部分的代码只是简单的序列转换。

总之，不同的应用领域将或多或少地从这两种方法中受益——有时它们可以从两者中受益。我们应该认真思考我们的应用领域，并分析开发者最有可能设计的解决方案和习惯用法。这将指引我们在开发特定应用时朝向更好的抽象方向。

# 摘要

在本章中，我们开发了我们自己的 CES 框架。通过开发自己的框架，我们巩固了对 CES 以及如何有效使用`core.async`的理解。

将`core.async`用作 CES 框架基础的想法并非是我的。James Reeves（见[`github.com/weavejester`](https://github.com/weavejester)）——路由库**Compojure**（见[`github.com/weavejester/compojure`](https://github.com/weavejester/compojure)）和许多其他有用的 Clojure 库的创造者——也看到了同样的潜力，并着手编写**Reagi**（见[`github.com/weavejester/reagi`](https://github.com/weavejester/reagi)），这是一个建立在`core.async`之上的 CES 库，其精神与我们本章开发的类似。

他投入了更多的努力，使其成为纯 Clojure 框架的一个更稳健的选择。我们将在下一章中探讨它。
