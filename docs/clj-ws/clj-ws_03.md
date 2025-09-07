# 第三章：3. 深入了解函数

概述

在本章中，我们将深入探讨 Clojure 的函数。我们发现了解构技术和高级调用签名。我们更深入地研究了函数的一等特性，了解它如何实现函数组合，以及高级多态技术。本章教授的技术将显著提高您代码的简洁性和可读性。它为您准备本书的第二部分，即操作集合的部分奠定了坚实的基础。

到本章结束时，您将能够在编写函数时实现解构、可变参数和多方法等特性。

# 简介

Clojure 是一种函数式编程语言，对于 Clojure 程序员来说，函数具有根本的重要性。在函数式编程中，我们避免改变程序的状态，正如我们在上一章所看到的，Clojure 的不可变数据结构极大地促进了这一点。我们还倾向于用函数做所有事情，因此我们需要函数能够做几乎所有的事情。我们说 Clojure 函数是**一等公民**，因为我们可以将它们传递给其他函数，将它们存储在变量中，或者从其他函数中返回：我们也将它们称为一等函数。以一个电子商务应用程序为例，用户可以看到一个包含不同搜索过滤器和排序选项的商品列表。以命令式编程方式使用标志和条件开发这样的过滤引擎可能会迅速变得过于复杂；然而，使用函数式编程可以优雅地表达这一点。函数组合是一种简单而有效地实现此类过滤引擎的绝佳方式，对于每个过滤器（例如，商品的价格、颜色、尺寸等），逻辑可以包含在一个函数中，并且这些函数可以简单地组合或组合，当用户与界面交互时。

在本章中，您将学习如何掌握函数。我们将从解构技术开始，这些技术特别适用于函数参数，然后我们将继续探讨高级调用签名，包括具有多个可变参数和可变数量参数的函数。接着，我们将深入研究函数的一等特性，了解它们如何实现函数组合。最后，我们将解释使用多方法和解派函数的高级多态技术。

# 解构

解构允许您从结构中移除数据元素或分解结构。这是一种通过提供更好的工具来改进广泛使用模式的可读性和简洁性的技术。解构数据有两种主要方式：顺序性（使用向量）和关联性（使用映射）。

想象我们需要编写一个函数，根据坐标元组打印格式化的字符串，例如，坐标元组 `[48.9615, 2.4372]`。我们可以编写以下函数：

```java
(defn print-coords [coords]
  (let [lat (first coords)
        lon (last coords)]
    (println (str "Latitude: " lat " - " "Longitude: " lon))))
```

这个 `print-coords` 函数接受一个坐标元组作为参数，并以格式化的字符串形式将坐标打印到控制台，例如，`纬度:` `48.9615` – `经度:` `2.4372`。

当我们将第一个元素绑定到 `lat` 并将第二个元素绑定到 `lon` 时，我们实际上在进行解构：我们正在从它们的顺序数据结构中取出每个元素。这种用法非常常见，Clojure 提供了一种内置语法来解构数据结构并将它们的值绑定到符号。

我们可以用以下方式使用顺序解构技术重写 `print-coords` 函数：

```java
(defn print-coords [coords]
  (let [[lat lon] coords]
    (println (str "Latitude: " lat " - " "Longitude: " lon))))
```

观察前面的例子，它比之前的例子更短、更简洁。我们不需要使用像 `first` 或 `last` 这样的函数，我们只需表达我们想要检索的符号。

这两个函数是等价的。`lat` 被映射到向量的第一个元素，而 `lon` 被映射到第二个元素。这个其他、更简单的例子可能更直观地有帮助：

```java
user=>(let
;;
;;     [1 2 3]
;;      | | |
      [[a b c] [1 2 3]] (println a b c))
1 2 3
nil
```

注意绑定是如何根据向量的顺序和定义在向量 `[a b c]` 中的符号顺序创建的。然后，符号值被打印到控制台。

列表，作为一种顺序数据结构，可以类似地分解：

```java
user=> (let [[a b c] '(1 2 3)] (println a b c))
1 2 3
nil
```

考虑打印机场坐标的相同例子，但这次我们接收到的数据是一个映射而不是元组。数据具有以下形状：`{:lat 48.9615, :lon 2.4372, :code 'LFPB', :name "Paris Le Bourget Airport"}`。

我们可以编写以下函数：

```java
(defn print-coords [airport]
  (let [lat (:lat airport)
        lon (:lon airport)
        name (:name airport)]
    (println (str name " is located at Latitude: " lat " - " "Longitude: " lon))))
```

这个函数通过在 `let` 表达式中使用关键字作为函数来从 `airport` 映射中检索值。我们可以看到在绑定 `lat`、`lon` 和 `name` 时出现的重复模式。同样，当我们想要分解的数据结构是关联的（一个 `map`）时，我们可以使用关联解构技术。函数可以用关联解构重写，如下所示：

```java
(defn print-coords [airport]
  (let [{lat :lat lon :lon airport-name :name} airport]
    (println (str airport-name " is located at Latitude: " lat " - " "Longitude: " lon))))
```

使用这种技术，我们通过将符号映射到映射中的键来创建绑定。现在 `lat` 符号包含机场映射中 `:lat` 键的值，`lon` 被映射到 `:lon` 键，最后，`airport-name` 符号被映射到 `:name` 键。

当键和符号都可以有相同的名称时，有一个更短的语法可用：

```java
(defn print-coords [airport]
  (let [{:keys [lat lon name]} airport]
    (println (str name " is located at Latitude: " lat " - " "Longitude: " lon))))
```

之前提到的解构语法表示在 `airport` 映射中查找 `lat`、`lon` 和 `name` 键，并将它们绑定到具有相同名称的符号。语法可能看起来有点令人惊讶，但这在 Clojure 中是一种广泛使用的技术。我们将在下一个练习中使用它，以便您可以学习如何使用它。

让我们看看我们的最终函数是如何工作的：

```java
user=> (def airport {:lat 48.9615, :lon 2.4372, :code 'LFPB', :name "Paris Le Bourget Airport"})
#'user/airport
(defn print-coords [airport]
  (let [{:keys [lat lon name]} airport]
    (println (str name " is located at Latitude: " lat " - " "Longitude: " lon))))
#'user/print-coords
user=> (print-coords airport)
Paris Le Bourget Airport is located at Latitude: 48.9615 - Longitude: 2.4372
nil
```

在前面的例子中，`print-coords` 函数在 `let` 表达式中解构了机场 `map`，并将值 `48.9615`、`2.4372` 和 `Paris Le Bourget Airport` 分别绑定到符号（分别）`lat`、`lon` 和 `name`。然后，这些值通过 `println` 函数（它返回 `nil`）打印到控制台。

现在我们已经发现了解构的基础及其用途，我们可以继续学习到 REPL，开始练习，并学习更多高级的解构技术。

## 练习 3.01：使用顺序解构解析 Fly Vector 的数据

为了这个练习的目的，假设我们正在构建一个航班预订平台应用程序。对于我们的第一个原型，我们只想解析和打印我们从合作伙伴那里收到的航班数据。我们的第一个合作伙伴 Fly Vector 尚未发现关联数据结构的威力，他们以向量的形式发送给我们所有数据。幸运的是，他们有全面的文档。我已经为你阅读了数百页的数据格式规范，以下是总结：

+   坐标点是由纬度和经度组成的元组，例如：`[48.9615, 2.4372]`。

+   航班是由两个坐标点组成的元组，例如：`[[48.9615, 2.4372], [37.742, -25.6976]]`。

+   预订由一些信息后跟一个或多个航班（最多三个）组成。第一个项目是 Fly Vector 的预订内部 ID，第二个项目是乘客的名字，第三个是 Fly Vector 要求我们不要解析或甚至查看的敏感信息（他们无法更新他们的系统以删除信息）。最后，向量中的其余部分是航班坐标数据，例如：

    ```java
    [
      1425,
      "Bob Smith",
      "Allergic to unsalted peanuts only",
      [[48.9615, 2.4372], [37.742, -25.6976]],
      [[37.742, -25.6976], [48.9615, 2.4372]]
    ]
    ```

这应该足够我们开发原型了，所以让我们开始吧：

1.  打开一个 REPL，并将样本预订数据绑定到 `booking` 符号：

    ```java
    user=> (def booking [1425, "Bob Smith", "Allergic to unsalted peanuts only", [[48.9615, 2.4372], [37.742, -25.6976]], [[37.742, -25.6976], [48.9615, 2.4372]]])
    #'user/booking
    ```

1.  通过实验解构来开始开发我们的解析函数。创建一个 `let` 块，并定义如下绑定，使用 `println` 打印结果：

    ```java
    user=> (let [[id customer-name sensitive-info flight1 flight2 flight3] booking] (println id customer-name flight1 flight2 flight3))
    1425 Bob Smith [[48.9615 2.4372] [37.742 -25.6976]] [[37.742 -25.6976] [48.9615 2.4372]] nil
    nil
    ```

    注意到 `flight3` 被绑定到值 `nil`。这是因为数据比定义的绑定短，能够只绑定存在的值是既有效又有用的。

    类似地，如果预订向量包含额外数据，它将被忽略。

1.  记住 `conj` 函数接受一个集合和一些元素作为参数，并返回一个新集合，其中包含那些添加到集合中的元素。使用 `conj` 在预订向量中添加两个航班，并使用相同的解构表达式解析数据：

    ```java
    user=> (let [big-booking (conj booking [[37.742, -25.6976], [51.1537, 0.1821]] [[51.1537, 0.1821], [48.9615, 2.4372]])
           [id customer-name sensitive-info flight1 flight2 flight3] big-booking]
      (println id customer-name flight1 flight2 flight3))
    1425 Bob Smith [[48.9615 2.4372] [37.742 -25.6976]] [[37.742 -25.6976] [48.9615 2.4372]] [[37.742 -25.6976] [51.1537 0.1821]]
    nil
    ```

    注意到最后一次航班被简单地忽略而没有打印出来。这是解构的另一个有用特性，也是 Clojure 动态性和实用性的另一个标志。

1.  在接收到的数据中，我们并不关心 Fly Vector 的内部 ID，我们也不希望解析敏感信息。这可以通过使用下划线 `_` 而不是符号来简单地忽略：

    ```java
    user=> (let [[_ customer-name _ flight1 flight2 flight3] booking] (println customer-name flight1 flight2 flight3))
    Bob Smith [[48.9615 2.4372] [37.742 -25.6976]] [[37.742 -25.6976] [48.9615 2.4372]] nil
    nil
    ```

    太好了，我们现在理解了如何使用解构忽略数据的一些部分。

    仅打印坐标数组并不是很易读，所以直到我们找到更好的打印航班的方法，我们只想简单地显示预订中的航班数量。当然，我们可以测试`flight1`、`flight2`和`flight3`是否存在值，但解构还有另一个我们可以使用的方面：序列的“剩余”部分。通过使用`&`字符后跟一个符号，我们可以将序列的剩余部分绑定到给定的符号。

1.  使用`&`字符将`flights`序列绑定到`flights`符号，然后按以下方式显示航班数量：

    ```java
    user=> (let [[_ customer-name _ & flights] booking]
      (println (str customer-name " booked " (count flights) " flights.")))
    Bob Smith booked 2 flights.
    nil
    ```

    注意，`flights`现在是一个集合，因此我们可以使用`count`函数与它一起使用。

    解构非常强大，也可以解构嵌套的数据结构。为了解析和打印航班详情，让我们创建一个单独的函数来保持代码的清晰和可读性。

1.  创建一个`print-flight`函数，使用嵌套解构解构航班路径，并打印出一个格式良好的航班行程单：

    ```java
    user=>
    (defn print-flight [flight]
      (let [[[lat1 lon1] [lat2 lon2]] flight]
        (println (str "Flying from: Lat " lat1 " Lon " lon1 " Flying to: Lat " lat2 " Lon " lon2))))
    #'user/print-flight
    user=> (print-flight [[48.9615, 2.4372], [37.742 -25.6976]])
    Flying from: Lat 48.9615 Lon 2.4372 Flying to: Lat 37.742 Lon -25.6976
    ```

    注意我们是如何通过简单地使用嵌套向量字面量表示法来挖掘`flight`中包含的嵌套向量，以检索坐标元组内的坐标值。然而，`let`绑定中的嵌套向量稍微有点难以阅读。

1.  通过分解多个`let`绑定来重写`print-flight`函数：

    ```java
    user=>
    (defn print-flight [flight]
      (let [[departure arrival] flight
            [lat1 lon1] departure
            [lat2 lon2] arrival]
        (println (str "Flying from: Lat " lat1 " Lon " lon1 " Flying to: Lat " lat2 " Lon " lon2))))
    #'user/print-flight
    user=> (print-flight [[48.9615, 2.4372], [37.742 -25.6976]])
    Flying from: Lat 48.9615 Lon 2.4372 Flying to: Lat 37.742 Lon -25.6976
    nil
    ```

    在前面的例子中，我们使用顺序解构创建了两个中间绑定：`departure`和`arrival`。这两个绑定包含坐标元组，我们可以进一步解构以创建纬度和经度绑定`lat1`、`lon1`、`lat2`和`lon2`。

1.  最后，让我们通过组合我们迄今为止编写的代码来编写`print-booking`函数：

    ```java
    (defn print-booking [booking]
      (let [[_ customer-name _ & flights] booking]
        (println (str customer-name " booked " (count flights) " flights."))
        (let [[flight1 flight2 flight3] flights]
          (when flight1 (print-flight flight1))
          (when flight2 (print-flight flight2))
          (when flight3 (print-flight flight3)))))
    #'user/print-booking
    user=> (print-booking booking)
    Bob Smith booked 2 flights.
    Flying from: Lat 48.9615 Lon 2.4372 Flying to: Lat 37.742 Lon -25.6976
    Flying from: Lat 37.742 Lon -25.6976 Flying to: Lat 48.9615 Lon 2.4372
    nil
    ```

干得好！在这个练习中，我们成功地使用了*顺序解构*来解析和检索向量中的数据，并提高了我们代码的可读性和简洁性。现在，让我们继续进行下一个练习，我们将使用*关联解构*。

## 练习 3.02：使用关联解构解析 MapJet 数据

让我们继续我们的航班预订平台应用程序。现在我们想要为另一个名为 MapJet 的合作伙伴开发相同的解析器。你可能已经猜到了，MapJet 发现了关联数据结构的威力，并正在向我们发送既好又易于理解的数据结构，这些数据结构由地图和向量组成。现在，数据是自我解释的，即使 MapJet 提供了非常详细的文档，我们也不会费心去阅读它。

让我们看看一个样本预订的数据形状：

```java
  {
    :id 8773
    :customer-name "Alice Smith"
    :catering-notes "Vegetarian on Sundays"
    :flights [
      {
        :from {:lat 48.9615 :lon 2.4372 :name "Paris Le Bourget Airport"},
        :to {:lat 37.742 :lon -25.6976 :name "Ponta Delgada Airport"}},
      {
        :from {:lat 37.742 :lon -25.6976 :name "Ponta Delgada Airport"},
        :to {:lat 48.9615 :lon 2.4372 :name "Paris Le Bourget Airport"}}
    ]
  }
```

首先，让我们同意地图是交换数据的好方法。对我们人类来说，它们非常易读，并且对我们的程序来说解析起来很简单。现在，让我们回到 REPL，看看关联解构如何帮助我们操作数据：

1.  将样本预订映射绑定到 `mapjet-booking` 符号，如下所示：

    ```java
    user=>
    (def mapjet-booking
      {
        :id 8773
        :customer-name "Alice Smith"
        :catering-notes "Vegetarian on Sundays"
        :flights [
          {
            :from {:lat 48.9615 :lon 2.4372 :name "Paris Le Bourget Airport"},
            :to {:lat 37.742 :lon -25.6976 :name "Ponta Delgada Airport"}},
          {
            :from {:lat 37.742 :lon -25.6976 :name "Ponta Delgada Airport"},
            :to {:lat 48.9615 :lon 2.4372 :name "Paris Le Bourget Airport"}}
        ]
      })
    #'user/mapjet-booking
    ```

1.  使用关联解构，以与 Fly Vector 相同的方式打印预订摘要（客户名称和航班数量）：

    ```java
    user=> (let [{:keys [customer-name flights]} mapjet-booking] (println (str customer-name " booked " (count flights) " flights.")))
    Alice Smith booked 2 flights.
    nil
    ```

    通过使用较短的、非重复的 `:keys` 语法，我们能够获取映射中的键并将它们的值绑定到具有相同名称的符号。

1.  让我们编写一个 `print-mapjet-flight` 函数来打印航班详情：

    ```java
    user=> (defn print-mapjet-flight [flight]
      (let [{:keys [from to]} flight
            {lat1 :lat lon1 :lon} from
            {lat2 :lat lon2 :lon} to]
        (println (str "Flying from: Lat " lat1 " Lon " lon1 " Flying to: Lat " lat2 " Lon " lon2))))
    user=> (print-mapjet-flight (first (:flights mapjet-booking)))
    Flying from: Lat 48.9615 Lon 2.4372 Flying to: Lat 37.742 Lon -25.6976
    nil
    ```

    注意，我们无法使用较短的语法来提取坐标，因为 `lat` 和 `lon` 名称会冲突；因此，我们使用了正常的语法，允许我们显式地声明具有不同名称的新绑定。

    与向量一样，我们可以嵌套解构表达式，甚至可以将两种技术结合起来。

1.  让我们重写 `print-mapjet-flight` 函数，但这次我们将嵌套我们的关联解构表达式：

    ```java
    user=>(defn print-mapjet-flight [flight]
      (let [{{lat1 :lat lon1 :lon} :from,
             {lat2 :lat lon2 :lon} :to} flight]
        (println (str "Flying from: Lat " lat1 " Lon " lon1 " Flying to: Lat " lat2 " Lon " lon2))))
    #'user/print-mapjet-flight
    user=> (print-mapjet-flight (first (:flights mapjet-booking)))
    Flying from: Lat 48.9615 Lon 2.4372 Flying to: Lat 37.742 Lon -25.6976
    nil
    ```

    前面的例子稍微有点复杂，所以如果一开始看起来有点混乱，请不要担心。将解构映射的键视为目标，将源视为值，如下所示：`{target1 source1 target2 source2}`。目标可以是符号，也可以是另一个类似这样的解构映射：`{{target3 source3} source1 {target4 source4} source2}`。注意，在最后一个表达式中，我们是如何仅将值绑定到最内层映射中的符号（`target3` 和 `target4`）。这正是我们在 `print-mapjet-flight` 函数中所做的：我们提取了两个坐标点的纬度和经度的嵌套值。

1.  编写用于打印 MapJet 预订的最终函数，使用代码将预订摘要打印到控制台。它应该产生与 Fly Vector 的 `print-booking` 函数类似的输出，首先打印航班数量，然后逐个打印每个航班，如下所示：

    ```java
    user=>
    (defn print-mapjet-booking [booking]
      (let [{:keys [customer-name flights]} booking]
        (println (str customer-name " booked " (count flights) " flights."))
        (let [[flight1 flight2 flight3] flights]
          (when flight1 (print-mapjet-flight flight1)) flights
          (when flight2 (print-mapjet-flight flight2))
          (when flight3 (print-mapjet-flight flight3)))))
    user=> (print-mapjet-booking mapjet-booking)
    ```

    输出如下：

    ```java
    Alice Smith booked 2 flights.
    Flying from: Lat 48.9615 Lon 2.4372 Flying to: Lat 37.742 Lon -25.6976
    Flying from: Lat 37.742 Lon -25.6976 Flying to: Lat 48.9615 Lon 2.4372
    ```

它工作了！我们现在已经完成了第一个原型。在这个练习中，我们实现了一个 `Map` 解析器，将解构后的数据打印到控制台。做得好！

解构技术是必不可少的，因为它们可以使我们的代码更加简洁和易于阅读。此外，我们的程序必须处理的数据通常来自外部数据源，我们并不总是拥有我们需要处理的数据的形状。拥有一个强大的工具来深入各种数据结构，可以显著提高我们作为程序员的生存质量。

然而，我们在上一个练习中编写的代码感觉有点重复；例如，两个 `print-booking` 函数有很多共同之处。根据我们目前所知，重构此代码会有点困难。但不要担心，我们将在下一主题中学习的技巧将允许你编写更加优雅的代码，更少重复，更少代码，因此更少错误。

# 高级调用签名

到目前为止，我们一直使用单一 arity（只有固定数量的参数）声明函数，并且简单地将传递给函数的参数绑定到一些参数名上。然而，Clojure 有一些技术可以在调用函数时提供更多的灵活性。

## Destructuring Function Parameters

首先，我们刚刚学到的关于解构的知识也适用于函数参数。是的，你读得对——我们可以在函数参数声明中使用解构技术！正如承诺的那样，以下是我们对之前练习中的`print-flight`函数进行重构的第一个尝试。观察以下示例，看看顺序解构是如何直接在函数参数中使用的：

```java
user=>
(defn print-flight
  [[[lat1 lon1] [lat2 lon2]]]
    (println (str "Flying from: Lat " lat1 " Lon " lon1 " Flying to: Lat " lat2 " Lon " lon2)))
#'user/print-flight
user=> (print-flight [[48.9615, 2.4372], [37.742 -25.6976]])
Flying from: Lat 48.9615 Lon 2.4372 Flying to: Lat 37.742 Lon -25.6976
nil
```

注意我们是如何移除了`let`表达式的。同样地，我们也可以对`print-mapjet-flight`做同样的处理，在函数参数中使用关联解构：

```java
user=>
(defn print-mapjet-flight
  [{{lat1 :lat lon1 :lon} :from, {lat2 :lat lon2 :lon} :to}]
    (println (str "Flying from: Lat " lat1 " Lon " lon1 " Flying to: Lat " lat2 " Lon " lon2)))
#'user/print-mapjet-flight
user=> (print-mapjet-flight { :from {:lat 48.9615 :lon 2.4372}, :to {:lat 37.742 :lon -25.6976} })
Flying from: Lat 48.9615 Lon 2.4372 Flying to: Lat 37.742 Lon -25.6976
```

再次，我们移除了`let`表达式，并立即从函数参数中解构参数。太好了——这是定义函数参数和进一步改进我们代码的一种新方法。

## Arity Overloading

第二，Clojure 支持“arity overloading”，这意味着我们可以通过指定新函数的额外参数来用另一个具有相同名称的函数来*overload*该函数。这两个函数具有相同的名称但不同的实现，执行函数体是根据函数调用提供的参数数量来选择的。以下是一个示例：

```java
user=>
(defn no-overloading []
  (println "Same old, same old..."))
#'user/no-overloading
user=>
(defn overloading
  ([] "No argument")
  ([a] (str "One argument: " a))
  ([a b] (str "Two arguments: a: " a " b: " b)))
#'user/overloading
```

注意不同的函数实现是如何定义的。在`no-overloading`函数中，这是我们习惯创建函数的方式，参数声明（紧随函数名之后）周围没有额外的括号。而`overloading`函数中，每个实现都由括号包围，从参数声明开始。

让我们看看多 arity 的`overloading`函数是如何发挥作用的：

```java
user=> (overloading)
"No argument"
```

在前面的代码中，没有传递任何参数；因此，调用`overloading`函数的第一个实现。

考虑以下代码：

```java
user=> (overloading 1)
"One argument: 1"
```

在这种情况下，一个参数被传递给`overloading`函数，因此调用第二个实现。

```java
user=> (overloading 1 2)
"Two arguments: a: 1 b: 2"
user=> (overloading 1 nil)
"Two arguments: a: 1 b: "
```

在前面的代码中，传递了两个参数，因此调用`overloading`函数的第三个实现。

```java
user=> (overloading 1 2 3)
Execution error (ArityException) at user/eval412 (REPL:1).
Wrong number of args (3) passed to: user/overloading
```

最后，传递错误数量的参数会产生通常的 arity 异常。

你可能会（合理地）想知道这有什么用，为什么不直接声明不同的函数呢？实际上，当为同一函数定义多个 arity 时，你是在说这些函数本质上相同，它们在做相似的工作，但执行略有不同，这取决于参数的数量。提供默认值也可能很有用。

考虑以下代码，这是一个名为`strike`的新小型幻想游戏函数，用于计算`enemy`实体的新状态：

```java
user=> (def weapon-damage {:fists 10 :staff 35 :sword 100 :cast-iron-saucepan 150})
#'user/weapon-damage
user=>
(defn strike
  ([enemy] (strike enemy :fists))
  ([enemy weapon]
    (let [damage (weapon weapon-damage)]
      (update enemy :health - damage))))
#'user/strike
```

在前面的例子中，我们首先定义了一个 `HashMap` 并将其绑定到 `weapon-damage` 符号。第二个表达式是 `strike` 函数的定义，它从 `enemy` 实体中减去伤害，并在 `weapon-damage` 映射中检索伤害量。注意 `strike` 函数有两个实现。第一个实现只包含一个参数 `enemy`，第二个实现有两个参数：`enemy` 和 `weapon`。注意第一个实现是如何通过提供一个额外的参数来调用第二个实现的。因此，当只传递一个参数调用 `strike` 函数时，将提供默认值 `:fists`：

```java
user=> (strike {:name "n00b-hunter" :health 100})
{:name "n00b-hunter", :health 90}
```

注意到函数只使用了一个参数（`enemy` 实体），因此它通过了函数的单参数实现，使用 `:fists` 作为默认值，并返回一个剩余 90 点生命值的敌人（因为拳头造成了 10 点伤害）：

```java
user=> (strike {:name "n00b-hunter" :health 100} :sword)
{:name "n00b-hunter", :health 0}
user=> (strike {:name "n00b-hunter" :health 100} :cast-iron-saucepan)
{:name "n00b-hunter", :health -50}
```

在前面的例子中，`strike` 函数直接使用了双参数实现，因为第二个参数 `weapon` 被明确提供。

## 可变参数函数

关于函数参数，还有一个最后的秘密要揭示。根据我们关于函数参数的知识，你将如何定义 `str` 函数的参数，例如？

它似乎可以接受无限数量的参数。你可能记得这样使用 `str`：

```java
user=> (str "Concatenating " "is " "difficult " "to " "spell " "but " "easy " "to " "use!")
"Concatenating is difficult to spell but easy to use!"
```

但当然，它并不是通过这样的重载来实现的 `(defn str ([s] ...) ([s1 s2] ...) ([s1 s2 s3] ...) ([s1 s2 s3 s4] …))` 等等……那么，在这种情况下发生了什么？

这就是解构技术再次发挥作用。记住，我们可以使用 `&` 字符将序列的其余部分绑定到数据结构？这与函数参数的工作方式类似，我们可以使用 `&` 字符从传递给函数的参数中创建数据结构。这就是如何创建可变参数函数（接受可变数量参数的函数），这也是 `str` 函数是如何实现的。

看看文档是如何描述 `str` 函数的：

```java
user=> (doc str)
-------------------------
clojure.core/str
([] [x] [x & ys])
  With no args, returns the empty string. With one arg x, returns
  x.toString().  (str nil) returns the empty string. With more than
  one arg, returns the concatenation of the str values of the args.
nil
```

注意其不同参数的声明。它接受零个元素 `[]`，一个元素 `[x]` 或任意数量的元素 `[x & ys]`。

让我们尝试使用这些新知识来创建一个函数，用于向 `Parenthmazes` 的新玩家打印欢迎信息：

```java
user=>
(defn welcome
  [player & friends]
  (println (str "Welcome to the Parenthmazes " player "!"))
  (when (seq friends)
    (println (str "Sending " (count friends) " friend request(s) to the following players: " (clojure.string/join ", " friends)))))
#'user/welcome
```

观察我们如何在函数参数中直接使用解构技术，将 `player` 之后的所有参数绑定到 `friends` 集合。现在，让我们尝试使用我们的函数，传入一个或多个参数：

```java
user=> (welcome "Jon")
Welcome to the Parenthmazes Jon!
nil
user=> (welcome "Jon" "Arya" "Tyrion" "Petyr")
Welcome to the Parenthmazes Jon!
Sending 3 friend request(s) to the following players: Arya, Tyrion, Petyr
nil
```

注意当向 `welcome` 函数传递多个参数时，`friends` 符号被绑定到一个包含其余参数的序列中。

注意

`seq`函数可以用来从集合中获取序列。在`welcome`函数中，我们使用`seq`函数来测试集合是否包含元素。这是因为`seq`在作为参数传递的集合为空时返回`nil`。`(if (seq (coll)))`是一个常用的模式，你应该用它来代替`(if (not (empty? coll)))`。

我们可以稍微改进这个函数。我们不仅可以测试`friends`是否为空，还可以利用多重参数技术：

```java
user=>
(defn welcome
  ([player] (println (str "Welcome to Parenthmazes (single-player mode), " player "!")))
  ([player & friends]
    (println (str "Welcome to Parenthmazes (multi-player mode), " player "!"))
    (println (str "Sending " (count friends) " friend request(s) to the following players: " (clojure.string/join ", " friends)))))
#'user/welcome
```

注意这次，定义了两个`welcome`函数，一个只有一个`player`参数，另一个有无限数量的参数，这些参数将被绑定到`friends`符号。以这种方式分离函数可以提高代码的清晰度，更明确地表达函数的意图，并移除带有`when`的条件表达式。

让我们最后一次尝试`welcome`函数：

```java
user=> (welcome "Jon")
Welcome to Parenthmazes (single-player mode), Jon!
nil
user=> (welcome "Jon" "Arya" "Tyrion" "Petyr")
Welcome to Parenthmazes (multi-player mode), Jon!
Sending 3 friend request(s) to the following players: Arya, Tyrion, Petyr
nil
```

太好了——根据参数的数量，函数调用被调度到了正确的函数。

## 练习 3.03：使用括号迷宫的多重参数和结构化

在这个练习中，我们将继续通过添加新功能来完善`Parenthmazes`游戏，特别是改进我们的`strike`函数以实现治疗机制。我们还想添加护甲的概念，它可以减少受到的伤害。

准备在这个`Parenthmazes`新版本中，迎接侏儒和巨魔之间伟大战斗的到来。

1.  首先，启动 REPL 和旁边的你最喜欢的代码编辑器，创建`weapon-damage`映射，其中包含每种武器的伤害信息：

    ```java
    user=> (def weapon-damage {:fists 10.0 :staff 35.0 :sword 100.0 :cast-iron-saucepan 150.0})
    #'user/weapon-damage
    ```

    我们需要这个映射来查找玩家打击敌人时所造成的伤害量。

1.  现在，让我们创建`strike`函数，该函数将处理当敌人与我们处于同一阵营时（现在我们假设我们选择了侏儒阵营）的治疗：

    ```java
    user=>
    (defn strike
      ([target weapon]
        (let [points (weapon weapon-damage)]
          (if (= :gnomes (:camp target))
            (update target :health + points)
            (update target :health - points)))))
    #'user/strike
    ```

    在前面的函数中，`strike`函数的新代码是通过查找`target`实体的`:camp`键来检索目标所在的阵营。如果`target`属于侏儒阵营，我们使用`+`函数在`target`实体中增加 x 个`points`的健康值。否则，我们使用`-`来减少`target`实体中的健康点数。

1.  创建一个`enemy`实体，并按照以下方式测试我们新创建的`strike`函数：

    ```java
    user=> (def enemy {:name "Zulkaz", :health 250, :camp :trolls})
    #'user/enemy
    user=> (strike enemy :sword)
    {:name "Zulkaz", :health 150.0, :camp :trolls}
    ```

    健康点数已成功扣除。让我们看看友军玩家会发生什么。

1.  创建一个属于`:gnomes`阵营的`ally`实体，并按照以下方式测试我们新创建的`strike`函数：

    ```java
    user=> (def ally {:name "Carla", :health 80, :camp :gnomes})
    #'user/ally
    user=> (strike ally :staff)
    {:name "Carla", :health 115.0, :camp :gnomes}
    ```

    健康点数已成功添加！

    现在我们已经得到了`strike`函数的框架，让我们修改它以实现护甲功能。`target`实体可以包含一个`:armor`键，该键包含一个用于计算最终伤害量的系数。数字越大，护甲越好。例如，100 点打击的护甲值为 0.8 会导致造成 20 点伤害。护甲值为 0.1 会导致造成 90 点伤害，0 表示没有护甲，1 表示无敌。

1.  通过计算`target`实体中的`:armor`值来改变对目标造成的伤害量。如果目标没有护甲值，将其设置为`0`。为了提高可读性，我们将使用`let`绑定来分解伤害计算：

    ```java
    user=>
    (defn strike
      ([target weapon]
        (let [points (weapon weapon-damage)]
          (if (= :gnomes (:camp target))
            (update target :health + points)
            (let [armor (or (:armor target) 0)
                  damage (* points (- 1 armor))]
              (update target :health - damage))))))
    #'user/strike
    ```

    在`if`表达式的第二个分支中，我们使用`let`表达式通过`or`为`armor`分配一个默认值。如果`(:armor target)`是`nil`，则护甲的值为 0。第二个绑定包含基于护甲值的减少伤害。

1.  测试`strike`函数以查看它是否在没有护甲的情况下仍然工作：

    ```java
    user=> (strike enemy :cast-iron-saucepan)
    {:name "Zulkaz", :health 100.0, :camp :trolls}
    ```

    铸铁锅会造成 150 点伤害，250 减去 150 确实等于`100`。太好了，它起作用了。我们继续吧。

1.  重新定义`enemy`绑定以添加护甲值，并再次测试我们的`strike`函数：

    ```java
    user=> (def enemy {:name "Zulkaz", :health 250, :armor 0.8, :camp :trolls})
    #'user/enemy
    user=> (strike enemy :cast-iron-saucepan)
    {:name "Zulkaz", :health 220.0, :armor 0.8, :camp :trolls}
    ```

    太好了，伤害似乎根据护甲系数减少了。现在我们想使用我们的关联解构技术直接从函数参数中检索`camp`和`armor`值，并减少函数体内的代码量。我们唯一的问题是，我们仍然需要返回一个更新后的`target`实体，但我们如何同时解构`target`实体并保持对`target`参数的引用？Clojure 会支持你——你可以使用特殊键`:as`将解构的映射绑定到特定的名称。

1.  修改`strike`函数以在函数参数中使用关联解构。使用特殊键`:as`将传递给函数的映射绑定到符号`target`：

    ```java
    user=>
    (defn strike
      ([{:keys [camp armor] :as target} weapon]
        (let [points (weapon weapon-damage)]
          (if (= :gnomes camp)
            (update target :health + points)
            (let [damage (* points (- 1 (or armor 0)))]
              (update target :health - damage))))))
    #'user/strike
    ```

    关联解构中还有一个有用的特性我们可以利用：特殊键`:or`。它允许我们在想要提取的键找不到时提供一个默认值（而不是绑定到`nil`）。

1.  在解构映射中添加特殊键`:or`，为目标映射中的护甲键提供一个默认值，增加一个额外的参数来使`weapon`参数可选，最后添加一些文档说明，如下。别忘了用自己的一组括号包裹每个函数定义：

    ```java
    user=>
    (defn strike
      "With one argument, strike a target with a default :fists `weapon`. With two argument, strike a target with `weapon`.
       Strike will heal a target that belongs to the gnomes camp."
      ([target] (strike target :fists))
      ([{:keys [camp armor], :or {armor 0}, :as target} weapon]
        (let [points (weapon weapon-damage)]
          (if (= :gnomes camp)
            (update target :health + points)
            (let [damage (* points (- 1 armor))]
              (update target :health - damage))))))
    #'user/strike
    ```

1.  通过以下方式测试不同的场景以确保你的函数仍然按预期工作：

    ```java
    user=> (strike enemy)
    {:name "Zulkaz", :health 248.0, :armor 0.8, :camp :trolls}
    user=> (strike enemy :cast-iron-saucepan)
    {:name "Zulkaz", :health 220.0, :armor 0.8, :camp :trolls}
    user=> (strike ally :staff)
    {:name "Carla", :health 115.0, :camp :gnomes}
    ```

这就结束了这个练习。如果你再次看看你刚才编写的`strike`函数，它使用了 Clojure 的一些高级功能，包括解构、多态函数以及读取和更新映射。通过在`update`函数中传递一个函数作为参数，我们也使用了高阶函数的概念，我们将在下一节中进一步解释。

# 高阶编程

高阶编程意味着程序，特别是函数，可以操作其他程序或函数，这与一阶编程相反，在一阶编程中，函数操作的是诸如字符串、数字和数据结构这样的数据元素。在实践中，这意味着一个函数可以接受一些编程逻辑（另一个函数）作为参数，并/或返回一些编程逻辑以供最终执行。这是一个强大的功能，它允许我们在程序中组合单个、模块化的逻辑单元，以减少重复并提高代码的可重用性。

编写更简单的函数可以增加它们的模块化。我们希望创建简单的功能单元，可以用作小砖块来构建我们的程序。编写纯函数可以降低这些砖块的复杂性，并允许我们构建更好、更坚固的程序。纯函数是不改变我们程序状态的函数——它们不会产生副作用；纯函数在给出完全相同的参数时也总是返回相同的值。这种组合使得纯函数易于推理、构建和测试。尽管 Clojure 提供了修改我们程序状态的方法，但我们应尽可能多地编写和使用纯函数。

## 首等函数

让我们演示将函数作为参数的使用。我们在*第二章*的*练习 2.01*，*混淆机*中使用了函数作为参数，这是关于*数据类型和不可变性*的章节，使用了`clojure.string/replace`函数，也在前面的练习中使用了`update`函数。例如，要在`HashMap`中除以`2`，你可以传递一个匿名函数作为`update`函数的参数来进行除法：

```java
user=> (update {:item "Tomato" :price 1.0} :price (fn [x] (/ x 2)))
{:item "Tomato", :price 0.5}
```

更好的是，你可以简单地传递除法函数`/`，并带上参数`2`，如下所示：

```java
user=> (update {:item "Tomato" :price 1.0} :price / 2)
{:item "Tomato", :price 0.5}
```

注意，`update`将把旧值作为第一个参数传递给`/`函数（在这里，旧值是`1.0`），以及所有额外的参数（在这里，`2`）。

你可以操作任何类型的值。例如，要反转布尔值的值，使用`not`函数：

```java
user=> (update {:item "Tomato" :fruit false} :fruit not)
{:item "Tomato", :fruit true}
```

正如我们刚才看到的，`update`可以接受一个函数作为参数，但我们也可以定义自己的函数，该函数接受一个函数并将其应用于给定的参数：

```java
user=> (defn operate [f x] (f x))
#'user/operate
user=> (operate inc 2)
3
user=> (operate clojure.string/upper-case "hello.")
"HELLO."
```

在前面的例子中，`operate` 函数接受一个函数 `f` 作为参数，并用第二个参数 `x` 调用它。虽然不是很实用，但它展示了传递和调用作为参数传递的函数是多么简单。如果我们想传递任意数量的参数，我们可以使用我们在之前关于解构的章节中学到的 `&` 字符：

```java
user=> (defn operate [f & args] (f args))
#'user/operate
user=> (operate + 1 2 3)
Execution error (ClassCastException) at java.lang.Class/cast (Class.java:3369).
Cannot cast clojure.lang.ArraySeq to java.lang.Number
```

这次，`operate` 似乎可以接受任意数量的参数，但函数调用失败，因为 `args` 是一个序列。这是因为我们直接将 `f` 函数应用于 `args` 序列，而我们真正想要的是使用序列的每个元素作为参数来应用 `f`。有一个特殊函数可以解包序列并将函数应用于该序列的元素——那就是 `apply` 函数：

```java
user=> (+ [1 2 3])
Execution error (ClassCastException) at java.lang.Class/cast (Class.java:3369).
Cannot cast clojure.lang.PersistentVector to java.lang.Number
user=> (apply + [1 2 3])
6
```

注意到 `+` 在向量上不工作，但通过使用 `apply`，我们调用 `+`，将向量的每个元素作为参数传递给 `+`。

因此，我们可以在 `operate` 函数中使用 `apply` 来拥有一个完全工作的函数，该函数接受一个函数 `f` 作为参数，并用剩余的参数 `args` 调用 `f`：

```java
user=> (defn operate [f & args] (apply f args))
#'user/operate
user=> (operate str "It " "Should " "Concatenate!")
"It Should Concatenate!"
```

它成功了！`str` 函数被应用于传递给 `str` 的参数。

能够将函数作为参数传递是高阶函数的一个方面，但另一个方面是函数能够 *返回* 其他函数。考虑以下代码：

```java
user=> (defn random-fn [] (first (shuffle [+ - * /])))
#'user/random-fn
```

`shuffle` 函数通过随机排序数组元素来打乱数组，然后我们从其中取出第一个元素。换句话说，`random-fn` 函数从 `[+ - * /]` 集合中返回一个随机函数。请注意，`random-fn` 函数不接收任何参数：

```java
user=> (random-fn 2 3)
Execution error (ArityException) at user/eval277 (REPL:1).
Wrong number of args (2) passed to: user/random-fn
```

但 `random-fn` 返回的函数期望参数：

```java
user=> ((random-fn) 2 3)
-1
```

在前面的代码中，`(random-fn)` 返回了 `-`，所以从 `2` 中减去 `3`，结果是 `-1`。

您可以使用 `fn?` 函数来检查传递给参数的值是否是函数：

```java
user=> (fn? random-fn)
true
user=> (fn? (random-fn))
true
```

在这种情况下，观察 `random-fn` 和 `random-fn` 返回的值都是函数。因此，我们可以调用 `random-fn` 返回的函数，甚至将其绑定到一个符号上，如下面的例子所示，我们将函数绑定到 `mysterious-fn` 符号：

```java
user=>
(let [mysterious-fn (random-fn)]
  (mysterious-fn 2 3))
6
user=>
(let [mysterious-fn (random-fn)]
  (mysterious-fn 2 3))
2/3
user=>
(let [mysterious-fn (random-fn)]
  (mysterious-fn 2 3))
6
user=>
(let [mysterious-fn (random-fn)]
  (mysterious-fn 2 3))
5
```

注意到每次调用 `random-fn` 都返回了一个不同的函数。在每一步中，使用相同的参数调用 `mysterious-fn` 都由不同的函数处理。根据返回的值，我们可以猜测这些函数分别是 `*`、`/`、`*` 和 `+`。

虽然可以想象，但并不常见，编写返回其他函数的函数。然而，您将经常使用一些 Clojure 的核心实用函数，这些函数返回其他函数，我们将在下面介绍。这些函数值得探索，因为它们使得函数组合、可重用性和简洁性成为可能。

## 部分函数

这些核心实用函数中的第一个是 `partial`，它接受一个函数 `f` 作为参数和任意数量的参数 `args1`。它返回一个新的函数 `g`，该函数可以接受额外的参数 `args2`。当用 `args2` 调用 `g` 时，`f` 将使用 `args1` + `args2` 被调用。这可能听起来很复杂，但考虑以下示例：

```java
user=> (def marketing-adder (partial + 0.99))
#'user/marketing-adder
user=> (marketing-adder 10 5)
15.99
```

调用 `(partial + 0.99)` 返回一个新的函数，我们将它绑定到 `marketing-adder` 符号。当调用 `marketing-adder` 时，它将使用 `0.99` 和传递给函数的任何额外参数调用 `+`。请注意，我们使用了 `def` 而不是 `defn`，因为我们不需要构建一个新的函数——`partial` 会为我们做这件事。

这里是另一个示例：

```java
user=> (def format-price (partial str "€"))
#'user/format-price
user=> (format-price "100")
"€100"
user=> (format-price 10 50)
"€1050"
```

调用 `format-price` 将调用 `str` 函数的第一个参数 `"€"`，然后是其余的参数。当然，你可以像这样编写相同的函数：`(fn [x] (str "€" x))`，但使用 `partial` 是定义函数作为其他函数的函数的一种优雅且表达性强的方法。

## 函数组合

另一个核心实用函数是 `comp`，即组合。以我们的 `random-fn` 函数为例。要从集合中检索一个随机数，我们调用 `shuffle` 函数然后调用 `first` 函数。如果我们想实现一个 `sample` 函数，它正好做这件事，我们可以编写以下函数：

```java
user=> (defn sample [coll] (first (shuffle coll)))
#'user/sample
user=> (sample [1 2 3 4])
2
```

但更优雅的是，我们可以使用函数组合实用工具 `comp` 来实现 `sample` 函数：

```java
user=> (def sample (comp first shuffle))
#'user/sample
user=> (sample [1 2 3 4])
4
```

`comp` 是一个实用工具，它接受任意数量的函数作为参数，并返回一个新的函数，该函数按顺序调用这些函数，并将每个函数的结果传递给下一个函数。请注意，函数是从右到左组合的，所以在前面的例子中，`shuffle` 将在 `first` 之前被应用。这很重要，因为传递给函数链的参数的数量和类型通常是有意义的。例如，如果你想组合一个乘以数字并将结果增加一的函数，你需要将 `inc` 函数（增加）作为函数的第一个参数传递，如下所示：

```java
user=> ((comp inc *) 2 2)
5
user=> ((comp * inc) 2 2)
Execution error (ArityException) at user/eval405 (REPL:1).
Wrong number of args (2) passed to: clojure.core/inc
```

注意，当将 `inc` 作为 `comp` 函数的最后一个参数提供时，它调用 `(inc 2 2)`，这不起作用，因为 `inc` 只接受一个参数。

现在，让我们看看如何结合使用 `partial` 和 `comp` 来组合一个 `checkout` 函数，该函数使用我们之前定义的 `format-price` 和 `marketing-adder` 函数。`checkout` 函数将首先通过重用 `marketing-adder` 来添加其参数，然后使用 `format-price` 格式化价格，并返回在前面加上 `"Only"` 的字符串连接：

```java
user=> (def checkout (comp (partial str "Only ") format-price marketing-adder))
#'user/checkout
user=> (checkout 10 5 15 6 9)
"Only €45.99"
```

在前面的例子中，我们定义了一个`checkout`函数，作为`marketing-adder`、`format-price`和一个由`partial`返回的匿名函数的组合，在价格前添加文本`"Only"`。这个例子展示了 Clojure 中组合相关函数的卓越动态性和表现力。程序员的意图清晰简洁，跳过了定义函数和命名参数的技术细节。

在我们开始练习之前，让我们介绍一种编写匿名函数的新方法：`#()`字面量。`#()`是编写匿名函数的一种更简短的方式。参数没有命名，因此可以按顺序使用`%1`、`%2`、`%3`等来访问参数值。当只提供一个参数时，你可以简单地使用`%`（省略参数编号）来检索参数值。

例如，以下两个表达式是等价的：

```java
(fn [s] (str "Hello" s))
;; is the same as
#(str "Hello" %)
```

以及以下两个表达式也是等价的：

```java
(fn [x y] (* (+ x 10) (+ y 20)))
;; is the same as
#(* (+ %1 10) (+ %2 20))
```

`#()`字面量函数只是函数，调用方式与其他函数相同：

```java
user=> (#(str %1 " " %2 " " %3) "First" "Second" "Third")
"First Second Third"
```

注意，当提供多个参数时，我们需要使用`%1`和`%2`来引用作为参数传递的值。

注意

`#()`函数的简短字面量表示法很方便，但应该谨慎使用，因为编号参数可能难以阅读。一般来说，只应使用具有单个参数和单个函数调用的简短匿名函数。对于其他任何情况，你应该坚持使用带有命名参数的标准`fn`表示法以提高可读性。

让我们在`Parenthmazes`版本 3 的练习中应用这些新技术。

## 练习 3.04：使用 Parenthmazes 的高阶函数

在这个练习中，我们将展示函数作为一等公民的好处。我们将使用函数作为值并将它们组合在一起。

我们希望进一步提高我们的幻想游戏，Parenthmazes，这次通过改变武器机制，使每种武器都能有不同的行为。例如，我们希望“拳头”只有在敌人虚弱或已被削弱时才造成伤害。而不是在我们的`strike`函数中实现条件分支，我们将使用`weapon-damage` `HashMap`实现一个*调度表*，我们将它重命名为`weapon-fn-map`，因为这次每种武器都将有一个相关的函数（而不是一个数值）。调度表是一个指向函数的指针表。我们可以通过一个`HashMap`来实现它，其中指针是键，函数是值。

为了让我们的武器功能能够很好地组合，它们将接受一个表示健康的数值作为参数，并在扣除伤害后返回一个表示健康的数值。为了简化，这次我们将省略护甲的概念。让我们从`拳头`武器开始：

1.  在您最喜欢的代码编辑器旁边启动一个 REPL。创建一个新的`HashMap`，其中包含`:fists`键及其相关联的函数，该函数仅在敌人的生命值小于`100`时造成`10`点伤害，否则返回`health`参数。将新创建的函数绑定到`weapon-fn-map`符号，如下所示：

    ```java
    user=>
    (def weapon-fn-map
      {:fists (fn [health] (if (< health 100) (- health 10) health))})
    #'user/weapon-fn-map
    ```

    `HashMap`可以作为键或参数具有任何类型的值，因此将函数作为值是完全可以接受的。

1.  通过从`weapon-fn-map`检索该函数并使用`150`和`50`作为参数来尝试该函数：

    ```java
    user=> ((weapon-fn-map :fists) 150)
    150
    user=> ((weapon-fn-map :fists) 50)
    40
    ```

    观察到函数正确地返回了新的生命值。当`health`参数小于`100`时，它减去了`10`。

1.  现在转到`staff`武器。`staff`是唯一可以用来恢复生命值（`35`点生命值）的武器，因此相关联的函数应该简单地调用`+`而不是`-`。这似乎是一个使用`partial`生成这个函数的好机会：

    ```java
    (def weapon-fn-map
      {
        :fists (fn [health] (if (< health 100) (- health 10) health))
        :staff (partial + 35)
      })
    ```

    `:staff`键的值现在是一个函数，它将调用`+`并带有`35`以及任何额外的参数。

1.  尝试使用与`staff`相关联的函数，如下所示：

    ```java
    user=> ((weapon-fn-map :staff) 150)
    185
    ```

    对于`sword`武器，我们需要简单地从传入的参数中减去`100`点生命值。然而，`partial`不会起作用，因为参数的顺序不正确。例如，`((partial - 100) 150)`返回`-50`，因为函数调用相当于`(- 100 150)`，但我们需要`(- 150 100)`。

1.  创建一个匿名函数，从其参数中减去`100`，并将其与`sword`键相关联，如下所示：

    ```java
    (def weapon-fn-map
      {
        :fists (fn [health] (if (< health 100) (- health 10) health))
        :staff (partial + 35)
        :sword #(- % 100)
      })
    ```

    注意，我们使用了`%`来检索传递给匿名函数的参数，因为我们使用了简短的文字语法`#()`，期望只有一个参数。

1.  按照以下步骤测试您新创建的武器函数：

    ```java
    user=> ((weapon-fn-map :sword) 150)
    50
    ```

    它工作得很好！

    下一个要添加的武器是`cast-iron-saucepan`。为了增加趣味性，让我们在混合中加入一点随机性（毕竟锅不是一把非常精确的武器）。

1.  将`cast-iron-saucepan`函数添加到`HashMap`中，从生命值中减去`100`点以及`0`到`50`之间的随机数，如下所示：

    ```java
    (def weapon-fn-map
      {
        :fists (fn [health] (if (< health 100) (- health 10) health))
        :staff (partial + 35)
        :sword #(- % 100)
        :cast-iron-saucepan #(- % 100 (rand-int 50))
      })
    ```

    在前面的例子中，我们使用了`rand-int`函数，它生成一个介于`0`和提供的参数之间的随机整数。

1.  按照以下步骤测试新创建的函数：

    ```java
    user=> ((weapon-fn-map :cast-iron-saucepan) 200)
    77
    user=> ((weapon-fn-map :cast-iron-saucepan) 200)
    90
    ```

    由于`rand-int`函数，两次后续调用可能会返回不同的值。

1.  最后，我们想介绍一种新的武器（给不幸的冒险者），它不会造成任何伤害：红薯。为此，我们需要一个返回其参数（生命值）的函数。我们不需要实现它，因为它已经存在：`identity`。首先，让我们使用`source`函数查看`identity`函数的源代码：

    ```java
    user=> (source identity)
    (defn identity
      "Returns its argument."
      {:added "1.0"
       :static true}
      [x] x)
    Nil
    ```

    观察到`identity`只是返回其参数。`source`函数是 REPL 中交互式使用的一个很有用的工具，因为它将函数定义打印到控制台，这有时比函数的文档更有帮助。

1.  让我们最后一次重新定义我们的 `weapon-fn-map` `HashMap`，通过将函数标识符与 `:sweet-potato` 键关联，如下所示：

    ```java
    (def weapon-fn-map
      {
        :fists (fn [health] (if (< health 100) (- health 10) health))
        :staff (partial + 35)
        :sword #(- % 100)
        :cast-iron-saucepan #(- % 100 (rand-int 50))
        :sweet-potato identity
      })
    ```

    现在我们已经最终确定了 `weapon-fn-map`，我们应该修改 `strike` 函数以处理存储在 `HashMap` 中的武器函数值。记住，`strike` 函数接受一个 `target` 实体作为参数，并返回具有新的健康键值的该实体。因此，更新实体的健康值应该像将 `weapon` 函数传递给 `update` 函数那样简单，因为我们的 `weapon` 函数接受健康值作为参数并返回新的健康值。

1.  将上一练习中的 `strike` 函数重写，使其使用存储在 `weapon-fn-map` 中的武器函数，如下所示：

    ```java
    user=>
    (defn strike
      "With one argument, strike a target with a default :fists `weapon`. With two argument, strike a target with `weapon` and return the target entity"
      ([target] (strike target :fists))
      ([target weapon]
        (let [weapon-fn (weapon weapon-fn-map)]
          (update target :health weapon-fn))))
    #'user/strike
    ```

1.  现在通过传递各种武器作为参数来测试你的 `strike` 函数。为了方便，你可能还想创建一个 `enemy` 实体：

    ```java
    user=> (def enemy {:name "Arnold", :health 250})
    #'user/enemy
    user=> (strike enemy :sweet-potato)
    {:name "Arnold", :health 250}
    user=> (strike enemy :sword)
    {:name "Arnold", :health 150}
    user=> (strike enemy :cast-iron-saucepan)
    {:name "Arnold", :health 108}
    ```

    如果我们想同时使用多个武器进行打击，而不是像这样嵌套调用 `strike`：

    ```java
    user=> (strike (strike enemy :sword) :cast-iron-saucepan)
    {:name "Arnold", :health 42}
    ```

    我们可以简单地组合我们的武器函数，并直接使用核心更新函数。

1.  编写一个 `update` 表达式，使用 `comp` 函数同时使用两个武器进行打击，如下所示：

    ```java
    user=> (update enemy :health (comp (:sword weapon-fn-map) (:cast-iron-saucepan weapon-fn-map)))
    {:name "Arnold", :health 15}
    ```

    因为我们编写的武器函数具有一致的接口（以 `health` 作为参数并返回健康值），因此使用 `comp` 函数组合它们非常简单。为了完成这个练习，让我们创建一个 `mighty-strike` 函数，该函数一次性使用所有武器进行打击，也称为组合所有武器函数。`keys` 和 `vals` 函数可以用于 `HashMaps` 来检索地图的键或值的集合。要检索所有武器函数，我们可以简单地使用 `vals` 函数从 `weapon-fn-map` 中检索所有值。现在我们有了函数集合，我们如何组合它们呢？我们需要将集合中的每个函数传递给 `comp` 函数。记住，为了将集合中的每个元素作为函数的参数传递，我们可以使用 `apply`。

1.  编写一个名为 `mighty-strike` 的新函数，该函数接受一个 `target` 实体作为参数，并使用所有武器对其发动攻击。它应该将 `weapon-fn-map` 的值应用于 `comp` 函数，如下所示：

    ```java
    user=>
    (defn mighty-strike
      "Strike a `target` with all weapons!"
      [target]
      (let [weapon-fn (apply comp (vals weapon-fn-map))]
          (update target :health weapon-fn)))
    #'user/mighty-strike
    user=> (mighty-strike enemy)
    {:name "Arnold", :health 58}
    ```

现在，如果我们停下来反思 `mighty-strike` 函数，并思考在没有高阶函数的情况下如何实现它，我们会意识到函数组合的概念是多么简单和强大。

在本节中，我们学习了如何将函数用作简单的值，以及作为其他函数的参数或返回值，以及如何使用 `#()` 创建更短的匿名函数。我们还解释了如何使用 `partial`、`comp` 和 `apply` 来生成、组合和发现函数的新用法。

# 多方法

Clojure 提供了一种使用多方法实现多态的方法。多态是指代码单元（在我们的例子中是函数）在不同上下文中表现不同的能力，例如，根据代码接收到的数据形状。在 Clojure 中，我们也称其为 *运行时多态*，因为调用方法是在运行时而不是在编译时确定的。多方法是由一个分派函数和一个或多个方法组合而成的。创建这些多方法的两个主要操作符是 `defmulti` 和 `defmethod`。`defmulti` 声明一个多方法并定义了如何通过分派函数选择方法。`defmethod` 创建了将被分派函数选择的不同的实现。分派函数接收函数调用的参数并返回一个分派值。这个分派值用于确定调用哪个由 `defmethod` 定义的函数。这些术语很多，但不用担心，下面的示例将帮助您理解这些新概念。

让我们看看我们如何使用多方法实现 Parenthmazes 的 `strike` 函数。这次，武器是在作为参数传递的 `HashMap` 中：

```java
user=> (defmulti strike (fn [m] (get m :weapon)))
#'user/strike
```

在前面的代码中，我们创建了一个名为 `strike` 的多方法。第二个参数是分派函数，它简单地从作为参数传递的映射中检索武器。记住，关键字可以用作 `HashMap` 的函数，所以我们可以简单地写出 `defmulti` 如下：

```java
user=> (defmulti strike :weapon)
nil
```

注意，这次表达式返回了 `nil`。这是因为多方法已经被定义了。在这种情况下，我们需要从 `user` 命名空间中 `unmap` `strike` 变量，然后再次评估相同的表达式：

```java
user=> (ns-unmap 'user 'strike)
nil
user=> (defmulti strike :weapon)
#'user/strike
```

现在我们已经定义了我们的多方法和我们的分派函数（它简单地是 `:weapon` 关键字），让我们为几种武器创建 `strike` 函数，以演示 `defmethod` 的用法：

```java
user=> (defmethod strike :sword
[{{:keys [:health]} :target}]
(- health 100))
#object[clojure.lang.MultiFn 0xaa549e5 "clojure.lang.MultiFn@aa549e5"]
```

观察我们如何使用名为 `strike` 的函数调用 `defmethod`，`defmethod` 的第二个参数是分派值：`:sword`。当 `strike` 被调用时，包含武器键的映射中的武器值将被从参数中检索，然后由分派函数（`:weapon` 关键字）返回。同样，让我们为 `:cast-iron-saucepan` 分派值创建另一个 `strike` 实现方案：

```java
user=> (defmethod strike :cast-iron-saucepan
[{{:keys [:health]} :target}]
(- health 100 (rand-int 50)))
#object[clojure.lang.MultiFn 0xaa549e5 "clojure.lang.MultiFn@aa549e5"]
```

这次，`strike` 是在包含 `:cast-iron-saucepan` 在 `:weapon` 键的映射中调用的。之前定义的函数将被调用。让我们用两种不同的武器测试我们新创建的多方法：

```java
user=> (strike {:weapon :sword :target {:health 200}})
100
user=> (strike {:weapon :cast-iron-saucepan :target {:health 200}})
77
```

注意，调用 `strike` 时使用不同的参数可以调用两个不同的函数。

当分派值不映射到任何已注册的函数时，会抛出异常：

```java
user=> (strike {:weapon :spoon :target {:health 200}})
Execution error (IllegalArgumentException) at user/eval217 (REPL:1).
No method in multimethod 'strike' for dispatch value: :spoon
```

如果我们需要处理这种情况，我们可以添加一个具有 `:default` 分派值的函数：

```java
user=> (defmethod strike :default [{{:keys [:health]} :target}] health)
#object[clojure.lang.MultiFn 0xaa549e5 "clojure.lang.MultiFn@aa549e5"]
```

在这种情况下，我们决定通过简单地返回未修改的健康值来处理任何其他武器，不造成伤害：

```java
user=> (strike {:weapon :spoon :target {:health 200}})
200
```

注意，这次没有抛出异常，并返回了原始的健康值。调度函数可以更加详细。我们可以想象当敌人的健康值低于 50 点时，无论使用什么武器，都会立即将其消除：

```java
user=> (ns-unmap 'user 'strike)
nil
user=> (defmulti strike (fn
  [{{:keys [:health]} :target weapon :weapon}]
  (if (< health 50) :finisher weapon)))
#'user/strike
user=> (defmethod strike :finisher [_] 0)
#object[clojure.lang.MultiFn 0xf478a81 "clojure.lang.MultiFn@f478a81"]
```

之前的代码首先从`user`命名空间中取消映射`strike`，以便可以重新定义。然后，通过查找参数并当敌人的健康值低于`50`时调度到`:finisher`函数来重新定义调度函数。然后定义`:finisher`函数（具有调度值 finisher 的`strike`函数）以返回，简单地忽略其参数，并返回`0`。

由于我们取消了`strike`，我们必须添加`defmethods`，因为它们也会被移除。让我们重新添加一把剑和默认方法：

```java
user=> (defmethod strike :sword
[{{:keys [:health]} :target}]
(- health 100))
#object[clojure.lang.MultiFn 0xaa549e5 "clojure.lang.MultiFn@aa549e5"]
user=> (defmethod strike :default [{{:keys [:health]} :target}] health)
#object[clojure.lang.MultiFn 0xaa549e5 "clojure.lang.MultiFn@aa549e5"]
```

现在让我们看看我们的多方法是如何工作的：

```java
user=> (strike {:weapon :sword :target {:health 200}})
100
```

太好了——我们的函数仍然按预期工作。现在让我们看看当健康值低于`50`时会发生什么：

```java
user=> (strike {:weapon :spoon :target {:health 30}})
0
```

已经调用了`finisher`函数，并且打击多方法成功返回了`0`。

多方法可以做更多的事情，例如使用向量作为调度值进行多值调度或根据类型和层次结构进行调度。这很有用，但可能现在承担得有点多。让我们继续练习，通过值调度来练习使用多方法。

## 练习 3.05：使用多方法

在这个练习中，我们想要扩展我们的小游戏 Parenthmazes，使其具有移动玩家的能力。游戏板是一个简单的二维空间，具有 x 和 y 坐标。我们目前不会实现任何渲染或维护游戏状态，因为我们只想练习使用多方法，所以你必须发挥想象力。

1.  玩家的实体现在被赋予了一个额外的键`:position`，它包含一个`HashMap`，其中包含 x 和 y 坐标以及`:facing`键，它包含玩家面对的方向。以下是一个玩家实体的示例代码：

    ```java
     {:name "Lea" :health 200 :position {:x 10 :y 10 :facing :north}}
    ```

    向北或向南应该改变 y 坐标，向东或向西应该改变 x 坐标。我们将通过一个新的`move`函数来实现这一点。

1.  启动 REPL 并按照以下方式创建玩家实体：

    ```java
    user=> (def player {:name "Lea" :health 200 :position {:x 10 :y 10 :facing :north}})
    #'user/player
    ```

1.  创建`move`多方法。调度函数应该通过检索玩家实体的`:position`映射中的`:facing`值来确定调度值。`:facing`值可以是以下值之一：`:north`、`:south`、`:west`和`:east`：

    ```java
    user=> (defmulti move #(:facing (:position %)))
    #'user/move
    ```

    你可能已经注意到，连续的两个关键字函数调用可以用函数组合更优雅地表达。

1.  通过首先从`user`命名空间中取消映射 var 并使用`comp`来简化其定义来重新定义`move`多方法：

    ```java
    user=> (ns-unmap 'user 'move)
    nil
    user=> (defmulti move (comp :facing :position))
    #'user/move
    ```

1.  使用`:north`调度值创建`move`函数的第一个实现。它应该在`:position`映射中增加`:y`：

    ```java
    User=> (defmethod move :north
    [entity]
      (update-in entity [:position :y] inc))
    #object[clojure.lang.MultiFn 0x1d0d6318 "clojure.lang.MultiFn@1d0d6318"]
    ```

1.  通过使用 `player` 实体调用 `move` 并观察结果来尝试你新创建的函数：

    ```java
    user=> (move player)
    {:name "Lea", :health 200, :position {:x 10, :y 11, :facing :north}}
    ```

    注意到 `y` 位置的值成功增加了 1。

1.  为其余的分派值 `:south`、`:west` 和 `:east` 创建其他函数：

    ```java
    User=> (defmethod move :south
    [entity]
      (update-in entity [:position :y] dec))
    #object[clojure.lang.MultiFn 0x1d0d6318 "clojure.lang.MultiFn@1d0d6318"]
    User=> (defmethod move :west
    [entity]
      (update-in entity [:position :x] inc))
    #object[clojure.lang.MultiFn 0x1d0d6318 "clojure.lang.MultiFn@1d0d6318"]
    User=> (defmethod move :east
    [entity]
      (update-in entity [:position :x] dec))
    #object[clojure.lang.MultiFn 0x1d0d6318 "clojure.lang.MultiFn@1d0d6318"]
    ```

1.  通过提供面向不同方向的 `player` 实体来测试你新创建的函数：

    ```java
    user=> (move {:position {:x 10 :y 10 :facing :west}})
    {:position {:x 11, :y 10, :facing :west}}
    user=> (move {:position {:x 10 :y 10 :facing :south}})
    {:position {:x 10, :y 9, :facing :south}}
    user=> (move {:position {:x 10 :y 10 :facing :east}})
    {:position {:x 9, :y 10, :facing :east}}
    ```

    观察当将玩家向不同方向移动时，坐标是否正确改变。

1.  为当 `:facing` 的值不同于 `:north`、`:south`、`:west` 和 `:east` 时创建一个额外的函数，使用 `:default` 分派值：

    ```java
    user=> (defmethod move :default [entity] entity)
    #object[clojure.lang.MultiFn 0x1d0d6318 "clojure.lang.MultiFn@1d0d6318"]
    ```

1.  尝试你的函数并确保它能够处理意外值，通过返回原始实体映射来处理：

    ```java
    user=> (move {:position {:x 10 :y 10 :facing :wall}})
    {:position {:x 10, :y 10, :facing :wall}}
    ```

    注意到多方法被分配到了默认函数，并且当玩家面对 `:wall` 时，位置保持不变。

在本节中，我们学习了如何使用 Clojure 的多态特性与多方法。

## 活动 3.01：构建距离和成本计算器

让我们回到我们在 *练习 3.01*、*使用顺序解构解析 Fly Vector 的数据* 和 *练习 3.02*、*使用关联解构解析 MapJet 数据* 中所工作的飞行预订平台应用程序。在你到达本章末尾所花费的时间内，我们现在已经将公司发展成为一个名为 WingIt 的正规初创公司，拥有严肃的投资者、每周的董事会会议和一张乒乓球桌，这意味着我们现在需要构建应用程序的核心服务：两个地点之间的行程和成本计算。然而，在研究了航线、机场费用和复杂的燃料计算后，我们意识到我们需要开发的算法可能对我们这个阶段来说过于复杂。我们决定，对于我们的 **最小可行产品**（**MVP**），我们只是“临时应对”并提供更简单的交通方式，如驾驶甚至步行。然而，我们希望代码易于扩展，因为最终我们需要添加飞行（一些员工甚至无意中听到首席执行官在谈论不久后将在路线图中添加太空旅行！）。

WingIt MVP 的要求如下：

+   对于原型，我们将与 Clojure REPL 交互。界面是一个接受 `HashMap` 作为参数的行程函数。目前，用户必须输入坐标。这可能不是非常用户友好，但用户可以在自己的地球仪或地图上查找坐标！

+   行程函数返回一个包含键 `:distance`、`:cost` 和 `:duration` 的 `HashMap`。`:distance` 以公里为单位表示，`:cost` 以欧元表示，`:duration` 以小时表示。

+   行程函数的唯一参数是一个包含 `:from`、`:to`、`:transport` 和 `:vehicle` 的 `HashMap`。`:from` 和 `:to` 包含一个具有 `:lat` 和 `:lon` 键的 `HashMap`，代表我们星球上某个位置的纬度和经度。

+   `:transport` 可以是 `:walking` 或 `:driving`。

+   `:vehicle` 仅在 `:transport` 为 `:driving` 时有用，可以是 `:sporche`、`:sleta` 或 `:tayato`。

**为了计算距离**，我们将使用“欧几里得距离”，通常用于计算平面上两点之间的距离。对于飞行，我们可能需要使用至少哈弗辛公式，技术上，对于驾驶，我们需要使用路线，但我们只是想要相对较短距离的大致估计，所以现在使用更简单的欧几里得距离应该足够。计算中唯一复杂的部分是经度的长度取决于纬度，因此我们需要将经度乘以纬度的余弦值。计算两点（lat1, lon1）和（lat2, lon2）之间距离的最终方程如下：

![图 3.1：计算欧几里得距离](img/B14502_03_01.jpg)

![图 3.1：计算欧几里得距离](img/B14502_03_01.jpg)

图 3.1：计算欧几里得距离

`:driving`, 我们将查看用户在 `HashMap` 参数中选择的 `:vehicle`。每辆车的成本应该是距离的函数：

+   `:sporche` 平均每公里消耗 0.12 升汽油，每升汽油的价格为 €1.5。

+   `:tayato` 平均每公里消耗 0.07 升汽油，每升汽油的价格为 €1.5。

+   `:sleta` 平均每公里消耗 0.2 千瓦时（kwh）的电力，每千瓦时的价格为 €0.1。

+   当交通方式为 `:walking` 时，成本应为 0。

**为了计算持续时间**，我们考虑平均驾驶速度为每小时 70 公里，平均步行速度为每小时 5 公里。

这里有几个调用行程函数的示例，以及预期的输出：

```java
user=> (def paris {:lat 48.856483 :lon 2.352413})
#'user/paris
user=> (def bordeaux {:lat 44.834999  :lon -0.575490})
#'user/bordeaux
user=> (itinerary {:from paris :to bordeaux :transport :walking})
{:cost 0, :distance 491.61380776549225, :duration 122.90345194137306}
user=> (itinerary {:from paris :to bordeaux :transport :driving :vehicle :tayato})
{:cost 44.7368565066598, :distance 491.61380776549225, :duration 7.023054396649889}
```

这些步骤将帮助您完成这项活动：

1.  首先定义 `walking-speed` 和 `driving-speed` 常量。

1.  创建两个其他常量，代表两个具有坐标的地点，`:lat` 和 `:lon`。您可以使用之前的示例，即巴黎和波尔多，或者查找您自己的地点。您将使用它们来测试您的距离和行程函数。

1.  创建 `distance` 函数。它应该接受两个参数，代表我们需要计算距离的两个地点。您可以在函数参数中直接使用顺序和关联解构来分解两个地点的纬度和经度。您可以在 `let` 表达式中分解计算的步骤，并使用 `Math/cos` 函数来计算一个数字的余弦值，以及 `Math/sqrt` 来计算一个数字的平方根；例如，`(Math/cos 0)`，`(Math/sqrt 9)`。

1.  创建一个名为 `itinerary` 的 *多态方法*。它将提供未来添加更多类型交通的灵活性。它应该使用 `:transport` 中的值作为 *调度值*。

1.  创建用于`:walking`调度值的行程函数。你可以在函数参数中使用关联解构来从`HashMap`参数中检索`:from`和`:to`键。你可以使用`let`表达式来分解距离和持续时间的计算。距离应简单地使用你之前创建的`distance`函数。为了计算持续时间，你应该使用在*步骤 1*中定义的`walking-speed`常量。

1.  对于`:driving`行程函数，你可以使用包含与成本函数关联的车辆的调度表。创建一个`vehicle-cost-fns`调度表。它应该是一个`HashMap`，键是车辆类型，值是基于距离的成本计算函数。

1.  创建用于`:driving`调度值的行程函数。你可以在函数参数中使用关联解构来从`HashMap`参数中检索`:from`、`:to`和`:vehicle`键。行驶距离和持续时间可以类似于步行距离和持续时间进行计算。成本可以通过使用`:vehicle`键从调度表中检索成本函数来计算。

预期输出：

```java
user=> (def london {:lat 51.507351, :lon -0.127758})
#'user/london
user=> (def manchester {:lat 53.480759, :lon -2.242631})
#'user/manchester
user=> (itinerary {:from london :to manchester :transport :walking})
{:cost 0, :distance 318.4448148814284, :duration 79.6112037203571}
user=> (itinerary {:from manchester :to london :transport :driving :vehicle :sleta})
{:cost 4.604730845743489, :distance 230.2365422871744, :duration 3.2890934612453484}
```

注意

本活动的解决方案可以在第 686 页找到。

# 摘要

在本章中，我们更深入地了解了 Clojure 的强大函数。我们学习了如何通过解构技术简化我们的函数，然后发现了高阶函数的巨大好处：模块化、简单性和可组合性。我们还介绍了一个高级概念，即使用 Clojure 的多态机制编写更可扩展的代码：多方法。现在你已经熟悉了 REPL、数据类型和函数，你可以继续学习关于工具和函数式技术来操作集合。

在下一章中，我们将探讨 Clojure 的顺序集合，并查看两个最有用的模式：映射和过滤。
