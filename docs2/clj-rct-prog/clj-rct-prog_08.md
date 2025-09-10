# 第八章。未来

向反应式应用迈出的第一步是跳出同步处理。一般来说，应用程序浪费了很多时间等待事情发生。也许我们在等待一个昂贵的计算——比如计算第 1000 个斐波那契数。也许我们在等待某些信息被写入数据库。我们也可以在等待一个网络调用返回，带给我们我们最喜欢的在线商店的最新推荐。

无论我们等待什么，我们都不应该阻塞我们应用程序的客户端。这对于在构建反应式系统时实现我们想要的响应性至关重要。

在处理核心丰富的时代——我的 MacBook Pro 有八个处理器核心——阻塞 API 严重地未充分利用我们可用的资源。

随着我们接近这本书的结尾，适当地退后一步，欣赏到并非所有处理并发、异步计算的类问题都需要像 RxJava 或`core.async`这样的框架机制。

在本章中，我们将探讨另一个有助于我们开发并发、异步应用的抽象：**未来**。我们将了解：

+   Clojure 实现 futures 的问题和局限性

+   Clojure 的 futures 的替代方案，提供异步、可组合的语义

+   如何在面临阻塞 IO 的情况下优化并发

# Clojure futures

解决这个问题的第一步——即防止一个可能长时间运行的任务阻塞我们的应用程序——是创建新的线程，这些线程执行工作并等待其完成。这样，我们保持应用程序的主线程空闲，以便为更多客户端提供服务。

直接与线程工作，然而，却是繁琐且容易出错的，所以 Clojure 的核心库包括了未来（futures），它们的使用极其简单：

```java
(def f (clojure.core/future
         (println "doing some expensive work...")
         (Thread/sleep 5000)
         (println "done")
         10))
(println "You'll see me before the future finishes")
;; doing some expensive work...
;; You'll see me before the future finishes
;; done
```

```java
clojure.core/future macro with a body simulating an expensive computation. In this example, it simply sleeps for 5 seconds before returning the value 10\. As the output demonstrates, this does not block the main thread, which is free to serve more clients, pick work items from a queue, or what have you.
```

当然，最有趣的计算，如昂贵的计算，返回我们关心的结果。这就是 Clojure futures 的第一个局限性变得明显的地方。如果我们尝试在完成之前检索未来的结果——通过解引用它——调用线程将阻塞，直到未来返回一个值。尝试运行以下略微修改的先前代码片段：

```java
(def f (clojure.core/future
         (println "doing some expensive work...")
         (Thread/sleep 5000)
         (println "done")
         10))
(println "You'll see me before the future finishes")
@f
(println "I could be doing something else. Instead I had to wait")

;; doing some expensive work...
;; You'll see me before the future finishes
;; 5 SECONDS LATER
;; done
;; I could be doing something else. Instead, I had to wait
```

现在唯一的区别是我们立即尝试在创建未来后立即*解引用*它。由于未来尚未完成，我们就会在那里等待 5 秒钟，直到它返回其值。只有在这种情况下，我们的程序才被允许继续。

通常，这在构建模块化系统时会导致问题。通常，像前面描述的那样，长时间运行的操作会在特定的模块或函数中启动，然后将其传递给下一个逻辑步骤以进行进一步处理。

Clojure futures 不允许我们在未来完成时安排一个函数执行以进行进一步处理。这是构建反应式系统的一个重要功能。

# 并行获取数据

为了更好地理解上一节中概述的问题，让我们构建一个更复杂的示例，该示例获取关于我喜欢的电影之一《指环王》的数据。

这个想法是，给定一部电影，我们希望检索其演员，并且对于每个演员，检索他们参与过的电影。我们还希望了解每个演员的更多信息，例如他们的配偶。

此外，我们将匹配每个演员的电影与顶级五部电影列表，以突出显示它们。最后，结果将打印到屏幕上。

从问题陈述中，我们确定了以下两个主要特征，我们需要考虑：

+   其中一些任务需要并行执行

+   它们相互建立依赖关系

为了开始，让我们创建一个新的 leiningen 项目：

```java
lein new clj-futures-playground

```

接下来，打开 `src/clj_futures_playground/core.clj` 中的核心命名空间文件，并添加我们将要使用的数据：

```java
(ns clj-futures-playground.core
  (:require [clojure.pprint :refer [pprint]]))

(def movie
  {:name "Lord of The Rings: The Fellowship of The Ring"
   :cast ["Cate Blanchett"
          "Elijah Wood"
          "Liv Tyler"
          "Orlando Bloom"]})

(def actor-movies
  [{:name "Cate Blanchett"
    :movies ["Lord of The Rings: The Fellowship of The Ring"
             "Lord of The Rings: The Return of The King"
             "The Curious Case of Benjamin Button"]}

   {:name "Elijah Wood"
    :movies ["Eternal Sunshine of the Spotless Mind"
             "Green Street Hooligans"
             "The Hobbit: An Unexpected Journey"]}

   {:name "Liv Tyler"
    :movies ["Lord of The Rings: The Fellowship of The Ring"
             "Lord of The Rings: The Return of The King"
             "Armageddon"]}

   {:name "Orlando Bloom"
    :movies ["Lord of The Rings: The Fellowship of The Ring"
             "Lord of The Rings: The Return of The King"
             "Pirates of the Caribbean: The Curse of the Black Pearl"]}])

(def actor-spouse
  [{:name "Cate Blanchett"    :spouse "Andrew Upton"}
   {:name "Elijah Wood"       :spouse "Unknown"}
   {:name "Liv Tyler"         :spouse "Royston Langdon"}
   {:name "Orlando Bloom"     :spouse "Miranda Kerr"}])
(def top-5-movies
  ["Lord of The Rings: The Fellowship of The Ring"
   "The Matrix"
   "The Matrix Reloaded"
   "Pirates of the Caribbean: The Curse of the Black Pearl"
   "Terminator"])
```

命名空间声明很简单，只需要 `pprint` 函数，这将帮助我们以易于阅读的格式打印我们的结果。有了所有数据，我们可以创建模拟远程服务的函数，这些服务负责获取相关数据：

```java
(defn cast-by-movie [name]
  (future (do (Thread/sleep 5000)
              (:cast  movie))))

(defn movies-by-actor [name]
  (do (Thread/sleep 2000)
      (->> actor-movies
           (filter #(= name (:name %)))
           first)))

(defn spouse-of [name]
  (do (Thread/sleep 2000)
      (->> actor-spouse
           (filter #(= name (:name %)))
           first)))

(defn top-5 []
  (future (do (Thread/sleep 5000)
              top-5-movies)))
```

每个 `service` 函数通过给定的时间量暂停当前线程以模拟缓慢的网络。函数 `cast-by-movie` 和 `Top 5` 每个都返回一个 future，表示我们希望在另一个线程上获取这些数据。其余的函数简单地返回实际数据。然而，它们也将在一个不同的线程中执行，正如我们很快将看到的。

下一步，我们需要一个函数来聚合所有获取的数据，将配偶与演员匹配，并突出显示 **Top 5** 列表中的电影。我们将称之为 `aggregate-actor-data` 函数：

```java
(defn aggregate-actor-data [spouses movies top-5]
  (map (fn [{:keys [name spouse]} {:keys [movies]}]
         {:name   name
          :spouse spouse
          :movies (map (fn [m]
                         (if (some #{m} top-5)
                           (str m " - (top 5)")
                           m))
                       movies)})
       spouses
       movies))
```

前面的函数相当直接。它只是将配偶和电影组合在一起，构建一个键为 `:name`、`:spouse` 和 `:movies` 的映射。它进一步将 `movies` 转换为在 `top-5` 列表中的项后面添加 **Top 5** 后缀。

最后一个拼图是 `-main` 函数，它允许我们从命令行运行程序：

```java
(defn -main [& args]
  (time (let [cast    (cast-by-movie "Lord of The Rings: The Fellowship of The Ring")
              movies  (pmap movies-by-actor @cast)
              spouses (pmap spouse-of @cast)
              top-5   (top-5)]
          (prn "Fetching data...")
          (pprint (aggregate-actor-data spouses movies @top-5))
          (shutdown-agents))))
```

```java
First, we wrap the whole body in a call to time, a simple benchmarking function that comes with Clojure. This is just so we know how long the program took to fetch all data—this information will become relevant later.
```

然后，我们设置了一系列 `let` 绑定。第一个，`cast`，是调用 `cast-by-movie` 的结果，它返回一个 future。

下一个绑定，`movies`，使用了一个我们之前没有见过的函数：`pmap`。

`pmap` 函数类似于 `map`，除了函数是并行映射到列表中的项。`pmap` 函数在幕后使用 futures，这就是为什么 `movies-by-actor` 不返回 future——它将这个任务留给 `pmap` 处理。

### 提示

`pmap` 函数实际上是为 CPU 密集型操作设计的，但在这里使用是为了使代码简单。面对阻塞 I/O，`pmap` 不会表现最优。我们将在本章后面更多地讨论阻塞 I/O。

我们通过*deref* `cast`绑定来获取演员列表，正如我们在上一节中看到的，这会阻塞当前线程等待异步获取完成。一旦所有结果都准备好了，我们只需调用`aggregate-actor-data`函数。

最后，我们调用`shutdown-agents`函数，这将关闭 Clojure 中 futures 背后的**线程池**。这对于我们的程序正确终止是必要的，否则它会在终端中简单地挂起。

要运行程序，请在终端中（在项目根目录下）输入以下内容：

```java
lein run -m clj-futures-playground.core

"Fetching data..."
({:name "Cate Blanchett",
 :spouse "Andrew Upton",
 :movies
 ("Lord of The Rings: The Fellowship of The Ring - (top 5)"
 "Lord of The Rings: The Return of The King"
 "The Curious Case of Benjamin Button")}
 {:name "Elijah Wood",
 :spouse "Unknown",
 :movies
 ("Eternal Sunshine of the Spotless Mind"
 "Green Street Hooligans"
 "The Hobbit: An Unexpected Journey")}
 {:name "Liv Tyler",
 :spouse "Royston Langdon",
 :movies
 ("Lord of The Rings: The Fellowship of The Ring - (top 5)"
 "Lord of The Rings: The Return of The King"
 "Armageddon")}
 {:name "Orlando Bloom",
 :spouse "Miranda Kerr",
 :movies
 ("Lord of The Rings: The Fellowship of The Ring - (top 5)"
 "Lord of The Rings: The Return of The King"
 "Pirates of the Caribbean: The Curse of the Black Pearl - (top 5)")})
"Elapsed time: 10120.267 msecs"

```

你会注意到程序需要一段时间才能打印出第一条消息。此外，因为当 futures 被解引用时它们会阻塞，所以程序只有在完全完成《指环王》的演员阵容的获取后才会开始获取前五部电影的列表。

让我们看看为什么是这样的：

```java
  (time (let [cast    (cast-by-movie "Lord of The Rings: The Fellowship of The Ring")
              ;; the following line blocks
              movies  (pmap movies-by-actor @cast)
              spouses (pmap spouse-of @cast)
              top-5   (top-5)]
```

```java
cast-by-movie to finish. As stated previously, Clojure futures don't give us a way to run some piece of code when the future finishes—like a callback—forcing us to block too soon.
```

这防止了`top-5`——一个完全独立的并行数据获取——在我们检索电影的演员阵容之前运行。

当然，这是一个人为的例子，我们可以在调用`top-5`之前解决这个特定的烦恼。问题是解决方案并不总是那么清晰，理想情况下我们不应该担心执行顺序。

正如我们将在下一节中看到的，有更好的方法。

# Imminent – 一个用于 Clojure 的可组合 futures 库

在过去几个月里，我一直在开发一个开源库，旨在解决 Clojure futures 之前的问题。这项工作的结果是称为*imminent*（见[`github.com/leonardoborges/imminent`](https://github.com/leonardoborges/imminent)）的库。

基本的区别在于 imminent futures 默认是异步的，并提供了一些组合子，允许我们声明性地编写程序，而无需担心其执行顺序。

展示库如何工作的最佳方式是将之前的电影示例重写进其中。我们将分两步进行。

首先，我们将单独检查即将到来的 API 的各个部分，这些部分将是我们最终解决方案的一部分。然后，我们将把它们全部整合到一个工作应用中。让我们先创建一个新的项目：

```java
lein new imminent-playground

```

接下来，将 imminent 的依赖项添加到你的`project.clj`中：

```java
:dependencies [[org.clojure/clojure "1.6.0"]
               [com.leonardoborges/imminent "0.1.0"]]
```

然后，创建一个新的文件，`src/imminent_playground/repl.clj`，并添加 imminent 的核心命名空间：

```java
(ns imminent-playground.repl
  (:require [imminent.core :as Ii]))

(def  repl-out *out*)
(defn prn-to-repl [& args]
  (binding [*out* repl-out]
    (apply prn args)))
```

```java
Feel free to type this in the REPL as we go along. Otherwise, you can require the namespace file from a running REPL like so:
```

```java
(require 'imminent-playground.repl)

```

所有的以下示例都应该在这个文件中。

## 创建 futures

在 imminent 中创建未来与在 Clojure 中创建未来并没有太大的区别。它就像以下这样简单：

```java
(def age (i/future 31))

;; #<Future@2ea0ca7d: #<Success@3e4dec75: 31>>
```

然而，看起来非常不同的是返回值。imminent API 中的一个关键决策是将计算的值表示为`Success`或`Failure`类型。正如先前的例子所示，Success 封装了计算的成果。Failure，正如你可能猜到的，将封装未来中发生的任何异常：

```java
(def failed-computation   (i/future (throw (Exception. "Error"))))
;; #<Future@63cd0d58: #<Failure@2b273f98: #<Exception java.lang.Exception: Error>>>

(def failed-computation-1 (i/failed-future :invalid-data))
;; #<Future@a03588f: #<Failure@61ab196b: :invalid-data>>
```

正如你所见，你不仅限于异常。我们可以使用`failed-future`函数创建一个立即完成给定原因的未来，在第二个例子中，这个原因只是一个关键字。

我们可能接下来会问的问题是“我们如何从未来中获取结果？”。与 Clojure 中的未来类似，我们可以按照以下方式解引用它：

```java
@age           ;; #<Success@3e4dec75: 31>
(deref @age)   ;; 31
(i/dderef age) ;; 31
```

使用双重解引用的惯用用法很常见，因此 imminent 提供了这样的便利，即`dderef`，它相当于调用`deref`两次。

然而，与 Clojure 未来不同，这是一个非阻塞操作，所以如果未来还没有完成，你将得到以下结果：

```java
@(i/future (do (Thread/sleep 500)
               "hello"))
;; :imminent.future/unresolved
```

未来的初始状态是`unresolved`，除非你绝对确定未来已经完成，否则解引用可能不是处理计算结果的最佳方式。这就是组合子变得有用的地方。

## 组合子和事件处理器

假设我们想要将年龄未来的值加倍。就像我们对列表做的那样，我们可以简单地映射一个函数到未来上，以完成这个操作：

```java
(def double-age (i/map age #(* % 2)))
;; #<Future@659684cb: #<Success@7ce85f87: 62>>
```

### 提示

虽然`i/future`将主体调度到单独的线程上执行，但值得注意的是，未来的组合子如`map`、`filter`等并不会立即创建一个新的线程。相反，它们在原始未来完成之后，将函数调度到线程池中异步执行。

另一种使用未来值的方法是使用`on-success`事件处理器，它在未来成功时被调用，并带有未来的封装值：

```java
(i/on-success age #(prn-to-repl (str "Age is: " %)))
;; "Age is: 31"
```

类似地，存在一个`on-failure`处理器，它对`Failure`类型做同样的事情。在讨论失败的话题时，imminent 未来理解它们被执行的上下文，如果当前未来产生一个`Failure`，它将简单地短路计算：

```java
(-> failed-computation
    (i/map #(* % 2)))
;; #<Future@7f74297a: #<Failure@2b273f98: #<Exception java.lang.Exception: Error>>>
```

在前面的例子中，我们不会得到一个新的错误，而是`failed-computation`中包含的原始异常。传递给`map`的函数永远不会运行。

决定将未来的结果封装在`Success`或`Failure`这样的类型中可能看起来是随意的，但实际上恰恰相反。这两种类型都实现了`IReturn`协议——以及一些其他协议，它们附带了一系列有用的函数，其中之一就是`map`函数：

```java
(i/map (i/success "hello")
       #(str % " world"))
;; #<Success@714eea92: "hello world">

(i/map (i/failure "error")
       #(str % " world"))
;; #<Failure@6d685b65: "error">
```

我们在这里得到的行为与我们之前的行为相似：将函数映射到失败上只是简单地短路整个计算。然而，如果你确实希望映射到失败上，你可以使用 map 的对应函数`map-failure`，它的工作方式与 map 类似，但它是其逆操作：

```java
(i/map-failure (i/success "hello")
               #(str % " world"))
;; #<Success@779af3f4: "hello">

(i/map-failure (i/failure "Error")
               #(str "We failed: " %))
;; #<Failure@52a02597: "We failed: Error">
```

这与最后提供的事件处理器`on-complete`配合得很好：

```java
(i/on-complete age
               (fn [result]
                 (i/map result #(prn-to-repl "success: " %))
                 (i/map-failure result #(prn-to-repl "error: " %))))

;; "success: " 31
```

与`on-success`和`on-failure`不同，`on-complete`使用结果类型封装调用提供的函数，因此它是处理单个函数中两种情况的一种方便方式。

回到组合子，有时我们需要将函数映射到一个返回未来的未来上：

```java
(defn range-future [n]
  (i/const-future (range n)))

(def age-range (i/map age range-future))

;; #<Future@3d24069e: #<Success@82e8e6e: #<Future@2888dbf4: #<Success@312084f6: (0 1 2...)>>>>
```

`range-future`函数返回一个成功的期货，它产生一个范围`n`。`const-future`函数与`failed-future`类似，但它立即使用`Success`类型完成期货。

然而，我们最终得到一个嵌套的期货，这几乎从来不是你想要的。没关系。这正是你将使用另一个组合子`flatmap`的场景。

你可以把它想象成针对期货的`mapcat`——它为我们简化了计算过程：

```java
(def age-range (i/flatmap age range-future))

;; #<Future@601c1dfc: #<Success@55f4bcaf: (0 1 2 ...)>>
```

另一个非常有用的组合子是用来将多个计算汇集到单个函数中使用的——`sequence`：

```java
(def name (i/future (do (Thread/sleep 500)
                        "Leo")))
(def genres (i/future (do (Thread/sleep 500)
                          ["Heavy Metal" "Black Metal" "Death Metal" "Rock 'n Roll"])))

(->  (i/sequence [name age genres])
     (i/on-success
      (fn [[name age genres]]
        (prn-to-repl (format "%s is %s years old and enjoys %s"
                             name
                             age
                             (clojure.string/join "," genres))))))

;; "Leo is 31 years old and enjoys Heavy Metal,Black Metal,Death Metal,Rock 'n Roll"
```

实际上，`sequence`创建了一个新的期货，它将仅在向量中的所有其他期货都完成或其中任何一个失败时才完成。

这很自然地引出了我们将要查看的最后一个组合子——`map-future`——我们将用它来代替电影示例中使用的`pmap`：

```java
(defn calculate-double [n]
  (i/const-future (* n 2)))

(-> (i/map-future calculate-double [1 2 3 4])
    i/await
    i/dderef)

;; [2 4 6 8]
```

在前面的例子中，`calculate-double`是一个返回值`n`翻倍的期货的函数。`map-future`函数随后将`calculate-double`映射到列表上，实际上是在并行执行计算。最后，`map-future`将所有期货序列化，返回一个单一的期货，它提供了所有计算的结果。

因为我们正在执行多个并行计算，并且不知道它们何时会完成，所以我们调用`await`在期货上，这是一种阻塞当前线程直到其结果准备好的方法。通常，你会使用组合子和事件处理器，但在这个例子中，使用`await`是可以接受的。

Imminent 的 API 提供了许多更多的组合子，这有助于我们以声明式的方式编写异步程序。本节让我们领略了 API 的强大功能，足以让我们使用 imminent 期货编写电影示例。

# 再次审视电影示例

仍然在我们的`imminent-playground`项目中，打开`src/imminent_playground/core.clj`文件并添加适当的定义：

```java
(ns imminent-playground.core
  (:require [clojure.pprint :refer [pprint]]
            [imminent.core :as i]))

(def movie ...)

(def actor-movies ...)

(def actor-spouse ...)

(def top-5-movies ...)
```

```java
The service functions will need small tweaks in this new version:
```

```java
(defn cast-by-movie [name]
  (i/future (do (Thread/sleep 5000)
                (:cast  movie))))

(defn movies-by-actor [name]
  (i/future (do (Thread/sleep 2000)
                (->> actor-movies
                     (filter #(= name (:name %)))
                     first))))

(defn spouse-of [name]
  (i/future (do (Thread/sleep 2000)
                (->> actor-spouse
                     (filter #(= name (:name %)))
                     first))))

(defn top-5 []
  (i/future (do (Thread/sleep 5000)
                top-5-movies)))

(defn aggregate-actor-data [spouses movies top-5]
    ...)
```

主要区别是它们现在都返回一个 imminent 期货。`aggregate-actor-data`函数与之前相同。

这带我们来到了`-main`函数，它被重写为使用 imminent 组合子：

```java
(defn -main [& args]
  (time (let [cast    (cast-by-movie "Lord of The Rings: The Fellowship of The Ring")
              movies  (i/flatmap cast #(i/map-future movies-by-actor %))
              spouses (i/flatmap cast #(i/map-future spouse-of %))
              result  (i/sequence [spouses movies (top-5)])]
          (prn "Fetching data...")
          (pprint (apply aggregate-actor-data
                      (i/dderef (i/await result)))))))
```

函数的起始部分与之前的版本非常相似，甚至第一个绑定`cast`看起来也很熟悉。接下来是`movies`，它是通过并行获取一个演员的电影得到的。这本身返回一个期货，所以我们通过`flatmap`在`cast`期货上，以获得最终结果：

```java
movies  (i/flatmap cast #(i/map-future movies-by-actor %))
```

`spouses`与`movies`的工作方式完全相同，这带我们来到了`result`。这是我们希望将所有异步计算汇集在一起的地方。因此，我们使用`sequence`组合子：

```java
result  (i/sequence [spouses movies (top-5)])
```

最后，我们决定通过使用`await`阻塞在`result`期货上——这样我们就可以打印出最终结果：

```java
(pprint (apply aggregate-actor-data
                      (i/dderef (i/await result)))
```

我们以与之前相同的方式运行程序，所以只需在命令行中输入以下内容，在项目的根目录下：

```java
lein run -m imminent-playground.core
"Fetching data..."
({:name "Cate Blanchett",
 :spouse "Andrew Upton",
 :movies
 ("Lord of The Rings: The Fellowship of The Ring - (top 5)"
 "Lord of The Rings: The Return of The King"
 "The Curious Case of Benjamin Button")}
...
"Elapsed time: 7088.398 msecs"

```

输出结果被裁剪了，因为它与之前完全相同，但有两点不同，值得注意：

+   第一个输出，**获取数据...**，打印到屏幕上的速度比使用 Clojure futures 的示例快得多

+   获取所有这些所需的总时间更短，仅超过 7 秒

这突出了 imminent futures 和组合器的异步性质。我们唯一需要等待的时间是在程序末尾显式调用`await`时。

更具体地说，性能提升来自以下代码段：

```java
(let [...
      result  (i/sequence [spouses movies (top-5)])]
   ...)
```

由于之前的所有绑定都不会阻塞当前线程，所以我们永远不需要等待并行启动`top-5`，从而从总体执行时间中节省了大约 3 秒。我们不需要显式考虑执行顺序——组合器只是做了正确的事情。

最后，还有一个不同之处是我们不再需要像以前那样显式调用`shutdown-agents`。这是因为 imminent 使用了一种不同类型的线程池：一个`ForkJoinPool`（参见[`docs.oracle.com/javase/7/docs/api/java/util/concurrent/ForkJoinPool.html`](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ForkJoinPool.html)）。

这个池子相对于其他线程池有许多优点——每个都有自己的权衡——其中一个特点是，我们不需要显式关闭它——它创建的所有线程都是守护线程。

当 JVM 关闭时，它会挂起等待所有非守护线程完成。只有在这种情况下，它才会退出。这就是为什么如果我们没有调用`shutdown-agents`，使用 Clojure futures 会导致 JVM 挂起的原因。

ForkJoinPool 创建的所有线程默认设置为守护线程：当 JVM 尝试关闭时，如果运行的唯一线程是守护线程，它们将被放弃，JVM 优雅地退出。

如`map`和`flatmap`之类的组合器以及`sequence`和`map-future`函数并不局限于 future。它们遵循许多更基本的原则，这使得它们在多个领域都有用。理解这些原则对于理解本书的内容不是必要的。如果您想了解更多关于这些原则的信息，请参阅附录，*《库设计的代数》*。

# Futures 和阻塞 IO

使用 ForkJoinPool 作为 imminent 的选择是故意的。ForkJoinPool 是在 Java 7 中添加的，非常智能。创建时，你给它一个期望的`parallelism`级别，默认为可用的处理器数量。

ForkJoinPool 会尝试通过动态缩小和扩大池子来满足所需的并行性。当一个任务提交到这个池子时，如果不需要，它不一定创建一个新的线程。这使得池子能够用更少的实际线程服务大量的任务。

然而，面对阻塞 I/O 时，它不能保证这样的优化，因为它无法知道线程是否正在阻塞等待外部资源。尽管如此，ForkJoinPool 提供了一种机制，允许线程在可能阻塞时通知池。

Imminent 通过实现 `ManagedBlocker` 接口（见 [`docs.oracle.com/javase/7/docs/api/java/util/concurrent/ForkJoinPool.ManagedBlocker.html`](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ForkJoinPool.ManagedBlocker.html)）利用了这一机制，并提供了另一种创建未来的方法，如下所示：

```java
  (-> (immi/blocking-future 
       (Thread/sleep 100)
       10)
      (immi/await))
  ;; #<Future@4c8ac77a: #<Success@45525276: 10>>

  (-> (immi/blocking-future-call
       (fn []
         (Thread/sleep 100)
         10))
      (immi/await))
  ;; #<Future@37162438: #<Success@5a13697f: 10>>
```

`blocking-future` 和 `blocking-future-call` 与它们的对应物 `future` 和 `future-call` 具有相同的语义，但应该在要执行的任务具有阻塞性质（即非 CPU 密集型）时使用。这允许 ForkJoinPool 更好地利用其资源，使其成为一个强大且灵活的解决方案。

# 摘要

在本章中，我们了解到 Clojure 的未来还有很多需要改进的地方。更具体地说，Clojure 的未来没有提供表达结果之间依赖关系的方法。但这并不意味着我们应该完全摒弃未来。

它们仍然是一个有用的抽象，并且对于异步计算具有正确的语义和丰富的组合器集——例如 Imminent 提供的——它们可以在构建性能和响应性强的反应式应用程序中成为强大的盟友。有时，这已经足够了。

对于需要模拟随时间变化的数据的情况，我们转向受 **函数式响应式编程**（**FRP**）和 **组合事件系统**（**CES**）启发的更丰富的框架——例如 RxJava——或 **通信顺序进程**（**CSP**）——例如 `core.async`。由于它们提供了更多功能，本书的大部分内容都致力于这些方法。

在下一章中，我们将通过案例研究回顾 FRP/CES。
