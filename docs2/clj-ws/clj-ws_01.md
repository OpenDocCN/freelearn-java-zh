# 第一章：1. Hello REPL！

概述

本章中，我们解释了创建 Clojure 程序的基础。我们首先让你熟悉 **读取求值打印循环**（**REPL**），在编写代码时，大部分实验都是在 REPL 中进行的。REPL 还允许你自己探索代码和文档，因此它是一个极佳的起点。在快速了解 REPL 之后，我们将更详细地描述如何阅读和理解简单的 Lisp 和 Clojure 代码，其语法有时可能显得令人不安。然后，我们探索 Clojure 中的基本运算符和函数，这些运算符和函数使你能够编写和运行简单的 Clojure 程序或脚本。

在本章结束时，你将能够使用 REPL 并在 Clojure 中处理函数。

# 简介

你是否曾经陷入面向对象应用程序的“意大利面代码”中？许多经验丰富的程序员会说是的，在他们的人生旅程或职业生涯的某个时刻，他们可能会重新考虑他们程序的基础。他们可能会寻找一个更简单、更好的面向对象编程的替代方案，而 Clojure 就是一个吸引人的选择。它是 Lisp 家族中的一种功能性强、简洁优雅的语言。它的核心很小，语法也很简单。它因其简单性而闪耀，这种简单性需要经过训练的眼睛才能注意到，最终才能理解。使用 Clojure 更为复杂的构建块将允许你设计和构建更坚固的应用程序。

不论你是经验丰富的程序员还是新手，爱好者还是专业人士，C# 大师还是 Haskell 高手，学习一门新的编程语言都是一项挑战。然而，这却是一种高度有益的经历，它将使你成为一个更好的程序员。在这本书中，你将通过实践来学习，并且能够快速提升你的技能。

Clojure 是今天学习编程语言的一个极佳选择。它将允许你使用一种旨在持久的技术来高效工作。Clojure 可以用来编写几乎任何东西：从完整的客户端-服务器应用程序到简单的脚本或大数据处理任务。到这本书结束时，你将使用 Clojure 和 ClojureScript 编写了一个现代网络应用程序，并将拥有所有启动你自己的应用程序所需的牌面！

# REPL 基础

欢迎来到 Clojure 的 **读取求值打印循环**（**REPL**），这是一个命令行界面，我们可以用它与运行的 Clojure 程序进行交互。REPL 在这个意义上是 **读取**用户的输入（用户就是你，程序员），通过即时编译和执行代码来**评估**输入，并将结果**打印**（即显示）给用户。读取-评估-打印的三步过程会不断重复（**循环**），直到你退出程序。

REPL 提供的动态性允许你发现和实验一个紧密的反馈循环：你的代码被即时评估，你可以调整它直到正确为止。许多其他编程语言提供了交互式外壳（特别是其他动态语言，如 Ruby 或 Python），但在 Clojure 中，REPL 在开发者的生活中扮演着非凡和至关重要的角色。它通常与代码编辑器集成，编辑、浏览和执行代码之间的界限模糊，类似于 Smalltalk 的可塑开发环境。但让我们从基础知识开始。

在这些练习中，你可能会注意到一些关于 Java 的提及（例如，在第二个练习的堆栈跟踪中）。这是因为 Clojure 是用 Java 实现的，并在 **Java 虚拟机**（**JVM**）上运行。因此，Clojure 可以从成熟的生态系统（经过实战检验、广泛部署的执行平台和大量的库）中受益，同时仍然是一个前沿技术。Clojure 被设计为托管语言，另一个名为 ClojureScript 的实现允许你在任何 JavaScript 运行时（例如，网页浏览器或 Node.js）上执行 Clojure 代码。这种托管语言实现的选择允许较小的函数式程序员社区在由 Java、.NET Core 和 JavaScript 技术主导的行业中努力。欢迎来到 Clojure 舞会，在这里我们都在享受我们的蛋糕并吃掉它。

## 练习 1.01：你的第一次舞蹈

在这个练习中，我们将在 REPL 中执行一些基本操作。让我们开始吧：

1.  打开终端并输入 `clj`。这将启动一个 Clojure REPL：

    ```java
    $ clj
    ```

    输出如下：

    ```java
    Clojure 1.10.1
    user=>
    ```

    第一行是 Clojure 的版本，在这个例子中是 `1.10.1`。如果你的版本不同，请不要担心——我们将一起进行的练习应该与任何版本的 Clojure 兼容。

    第二行显示了当前所在的命名空间（`user`）并提示输入。命名空间是一组属于一起的事物（例如函数）。在这里创建的任何内容都将默认位于 `user` 命名空间中。`user` 命名空间可以被视为你的游乐场。

    你的 REPL 已经准备好**读取**。

1.  让我们尝试评估一个表达式：

    ```java
    user=> "Hello REPL!"
    ```

    输出如下：

    ```java
    "Hello REPL!"
    ```

    在 Clojure 中，*字面量*字符串使用双引号创建，`""`。字面量是在源代码中表示固定值的一种表示法。

1.  让我们看看如果我们输入多个字符串会发生什么：

    ```java
    user=> "Hello" "Again"
    ```

    输出如下：

    ```java
    "Hello"
    "Again"
    ```

    我们已经连续评估了两个表达式，并且每个结果都打印到了单独的行上。

1.  现在，让我们尝试一些算术运算，例如 `1 + 2`：

    ```java
    user=> 1 + 2
    ```

    输出如下：

    ```java
    1
    #object[clojure.core$_PLUS_ 0xe8df99a "clojure.core$_PLUS_@e8df99a"]
    2
    ```

    输出并不完全符合我们的预期。Clojure 对三个组件进行了评估，即 `1`、`+` 和 `2`，*分别* 进行评估。评估 `+` 看起来很奇怪，因为 `+` 符号绑定了一个函数。

    注意

    函数是执行特定任务的代码单元。我们现在不需要了解更多，除了函数可以被调用（或调用）并且可以接受一些参数。函数的参数是一个术语，用于设计参数的值，但这些术语通常可以互换使用。

    要添加这些数字，我们需要调用`+`函数，并传入参数`1`和`2`。

1.  按如下方式调用`+`函数，并传入参数`1`和`2`：

    ```java
    user=> (+ 1 2)
    ```

    输出如下：

    ```java
    3
    ```

    你很快就会发现，许多通常是编程语言语法一部分的基本操作，比如加法、乘法、比较等，在 Clojure 中只是简单的函数。

1.  让我们尝试更多基本算术的示例。你甚至可以尝试向以下函数传递超过两个参数，所以将 1 + 2 + 3 相加将看起来像`(+ 1 2 3)`：

    ```java
    user=> (+ 1 2 3)
    6
    ```

1.  其他基本算术运算符的使用方式类似。尝试输入以下表达式：

    ```java
    user=> (- 3 2)
    1
    user=> (* 3 4 1)
    12
    user=> (/ 9 3)
    3
    ```

    在输入前面的示例之后，你应该尝试更多自己输入的内容——REPL 就是为了实验而存在的。

1.  你现在应该足够熟悉 REPL，可以提出以下问题：

    ```java
    user=> (println "Would you like to dance?")
    Would you like to dance?
    nil
    ```

    不要个人化——`nil`是`println`函数返回的值。该函数打印的文本仅仅是这个函数的*副作用*。

    `nil`是 Clojure 中“null”或“nothing”的等价物；也就是说，没有有意义的价值。`print`（不带换行符）和`println`（带换行符）用于将对象打印到标准输出，并在完成后返回`nil`。

1.  现在，我们可以组合这些操作并打印简单加法的结果：

    ```java
    user=> (println (+ 1 2))
    3
    nil
    ```

    该表达式打印了一个值为`3`，并返回了`nil`值。

    注意我们是如何嵌套那些*形式*（或*表达式*）。这就是我们在 Clojure 中链式调用函数的方式：

    ```java
    user=> (* 2 (+ 1 2))
    6
    ```

1.  通过按*Ctrl* + *D*退出 REPL。退出函数是`System/exit`，它接受退出码作为参数。因此，你也可以输入以下内容：

    ```java
    user=> (System/exit 0)
    ```

在这个练习中，我们发现了 REPL 并调用了 Clojure 函数来打印和执行基本的算术操作。

## 练习 1.02：在 REPL 中导航

在这个练习中，我们将介绍一些导航快捷键和命令，帮助你使用并生存于 REPL 中。让我们开始吧：

1.  首先再次打开 REPL。

1.  注意你可以通过按*Ctrl* + *P*（或*UP*箭头）和*Ctrl* + *N*（或*DOWN*箭头）来导航之前输入的内容和之前的会话历史。

1.  你也可以搜索（区分大小写）你输入的命令历史：按 *Ctrl* + *R* 然后输入 `Hello`，这应该会带回到我们之前输入的 `Hello Again` 表达式。如果你再次按 *Ctrl* + *R*，它将遍历搜索的匹配项并带回到第一个命令：`Hello REPL!`。如果你按 *Enter*，它将表达式带回到当前提示符。再次按 *Enter* 将评估它。

1.  现在，评估以下表达式，该表达式将数字 10 *增加*（加 1）：

    ```java
    user=> (inc 10)
    11
    ```

    返回的值是 11，这确实是 10 + 1。

1.  `*1` 是一个特殊变量，它绑定到在 REPL 中评估的最后表达式的结果。你可以通过简单地像这样输入它来评估它的值：

    ```java
    user=> *1
    11
    ```

    类似地，`*2` 和 `*3` 分别绑定到该 REPL 会话中第二和第三最近的价值。

1.  你也可以在其他表达式中重用这些特殊的变量值。看看你是否能跟随并输入以下命令序列：

    ```java
    user=> (inc 10)
    11
    user=> *1
    11
    user=> (inc *1)
    12
    user=> (inc *1)
    13
    user=> (inc *2)
    13
    user=> (inc *1)
    14
    ```

    注意 `*1` 和 `*2` 的值是如何随着新表达式的评估而变化的。当 REPL 中文本很多时，按 *Ctrl* + *L* 清屏。

1.  在 REPL 中还可用的一个有用变量是 `*e`，它包含最后一个异常的结果。目前，它应该是 `nil`，除非你之前生成了错误。让我们通过除以零来自愿触发一个异常：

    ```java
    user=> (/ 1 0)
    Execution error (ArithmeticException) at user/eval71 (REPL:1).
    Divide by zero
    ```

    评估 `*e` 应该包含有关异常的详细信息，包括堆栈跟踪：

    ```java
    user=> *e
    #error {
     :cause "Divide by zero"
     :via
     [{:type java.lang.ArithmeticException
       :message "Divide by zero"
       :at [clojure.lang.Numbers divide "Numbers.java" 188]}]
     :trace
     [[clojure.lang.Numbers divide "Numbers.java" 188]
      [clojure.lang.Numbers divide "Numbers.java" 3901]
      [user$eval1 invokeStatic "NO_SOURCE_FILE" 1]
      [user$eval1 invoke "NO_SOURCE_FILE" 1]
      [clojure.lang.Compiler eval "Compiler.java" 7177]
      [clojure.lang.Compiler eval "Compiler.java" 7132]
      [clojure.core$eval invokeStatic "core.clj" 3214]
      [clojure.core$eval invoke "core.clj" 3210]
      [clojure.main$repl$read_eval_print__9086$fn__9089 invoke "main.clj" 437]
      [clojure.main$repl$read_eval_print__9086 invoke "main.clj" 437]
      [clojure.main$repl$fn__9095 invoke "main.clj" 458]
      [clojure.main$repl invokeStatic "main.clj" 458]
      [clojure.main$repl_opt invokeStatic "main.clj" 522]
      [clojure.main$main invokeStatic "main.clj" 667]
      [clojure.main$main doInvoke "main.clj" 616]
      [clojure.lang.RestFn invoke "RestFn.java" 397]
      [clojure.lang.AFn applyToHelper "AFn.java" 152]
      [clojure.lang.RestFn applyTo "RestFn.java" 132]
      [clojure.lang.Var applyTo "Var.java" 705]
      [clojure.main main "main.java" 40]]}
    ```

    注意

    不同的 Clojure 实现可能会有略微不同的行为。例如，如果你在一个 ClojureScript REPL 中尝试除以 0，它不会抛出异常，而是返回“无穷大”值：

    `cljs.user=> (/ 1 0)`

    `##Inf`

    这是为了保持与主机平台的一致性：字面数字 0 在 Java（和 Clojure）中实现为整数，但在 JavaScript（和 ClojureScript）中实现为浮点数。IEEE 浮点算术标准（IEEE 754）指定除以 0 应该返回 +/- 无穷大。

1.  `doc`、`find-doc` 和 `apropos` 函数是浏览文档的必要 REPL 工具。既然你知道你想使用的函数的名称，你可以使用 `doc` 来阅读它的文档。让我们看看它在实际中的应用。首先，输入 `(doc str)` 来了解更多关于 `str` 函数的信息：

    ```java
    user=> (doc str)
    -------------------------
    clojure.core/str
    ([] [x] [x & ys])
      With no args, returns the empty string. With one arg x, returns
      x.toString().  (str nil) returns the empty string. With more than
      one arg, returns the concatenation of the str values of the args.
    nil
    ```

    `doc` 在第一行打印函数的完全限定名称（包括命名空间），在下一行打印可能的参数集（或“arity”），最后是描述。

    这个函数的完全限定名称是 `clojure.core/str`，这意味着它在 `clojure.core` 命名空间中。在 `clojure.core` 中定义的东西默认情况下就可用在你的当前命名空间中，无需你显式地导入它们。这是因为它们是构建程序的基本组件，每次都使用它们的完整名称将会很繁琐。

1.  让我们尝试使用 `str` 函数。正如文档所解释的，我们可以传递多个参数：

    ```java
    user=> (str "I" "will" "be" "concatenated") (clojure.core/str "This" " works " "too")
    "Iwillbeconcatenated"
    "This works too"
    ```

1.  让我们检查 `doc` 函数的文档：

    ```java
    user=> (doc doc)
    -------------------------
    clojure.repl/doc
    ([name])
    Macro
      Prints documentation for a var or special form given its name,
       or for a spec if given a keyword
    nil
    ```

    此函数位于 `clojure.repl` 命名空间中，它也在你的 REPL 环境中默认可用。

1.  你还可以查看命名空间的文档。正如其文档所建议的，你的最终程序通常不会使用 `clojure.repl` 命名空间中的辅助工具（例如，`doc`、`find-doc` 和 `apropos`）：

    ```java
    user=> (doc clojure.repl)
    -------------------------
    clojure.repl
      Utilities meant to be used interactively at the REPL
    nil
    ```

1.  当你不知道函数的名称，但你对描述或名称可能包含的内容有一个想法时，你可以使用 `find-doc` 辅助工具来搜索它。让我们尝试搜索 `modulus` 运算符：

    ```java
    user=> (find-doc "modulus")
    nil
    ```

1.  没有成功，但有一个转折：`find-doc` 是区分大小写的，但好消息是我们可以使用带有 `i` 修饰符的正则表达式来忽略大小写：

    ```java
    user=> (find-doc #"(?i)modulus")
    -------------------------
    clojure.core/mod
    ([num div])
      Modulus of num and div. Truncates toward negative infinity.
    nil
    ```

    目前你不需要了解更多关于正则表达式的知识——你甚至不需要使用它们，但忽略大小写来搜索函数可能很有用。你可以用 `#"(?i)text"` 语法来写它们，其中 `text` 是你想要搜索的任何内容。

    我们要找的函数是 `clojure.core/mod`。

1.  让我们确保它按照其文档工作：

    ```java
    user=> (mod 7 3)
    1
    ```

1.  使用 `apropos` 函数通过名称搜索函数，从而产生更简洁的输出。比如说，我们正在寻找一个转换给定字符字符串的案例的函数：

    ```java
    user=> (apropos "case")
    (clojure.core/case clojure.string/lower-case clojure.string/upper-case) 
    user=> (clojure.string/upper-case "Shout, shout, let it all out")
    "SHOUT, SHOUT, LET IT ALL OUT"
    ```

    请注意，此函数位于 `clojure.string` 命名空间中，默认情况下并未引用。在我们学习如何从其他命名空间导入和引用符号之前，您需要使用其全名。

## 活动一.01：执行基本操作

在这个活动中，我们将在 Clojure REPL 中打印消息并执行一些基本的算术运算。

这些步骤将帮助您完成此活动：

1.  打开 REPL。

1.  打印消息 "`我不怕括号`" 来激励自己。

1.  将 1、2 和 3 相加，然后将结果乘以 10 减 3，这对应于以下 `中缀` 表示法：(1 + 2 + 3) * (10 - 3)。你应该得到以下结果：

    ```java
    42
    ```

1.  打印消息 "`做得好`!" 来祝贺自己。

1.  退出 REPL。

    备注

    此活动的解决方案可以在第 678 页找到。

# Clojure 代码的评估

Clojure 是 Lisp 的一种方言，一种由 John McCarthy 设计的高级编程语言，首次出现在 1958 年。Lisp 及其衍生品或“方言”的最显著特征之一是使用数据结构来编写程序的源代码。我们 Clojure 程序中不寻常的括号数量就是这种特征的体现，因为括号用于创建列表。

在这里，我们将关注 Clojure 程序的构建块，即 *形式和表达式*，并简要地看看表达式是如何被评估的。

备注

“表达式”和“形式”这两个术语经常可以互换使用；然而，根据 Clojure 文档，表达式是一种形式类型：“每个没有被特殊形式或宏特别处理的表单都被编译器视为表达式，它被评估以产生一个值。”

我们已经看到字面量是有效的语法，并且评估为自身，例如：

```java
user=> "Hello"
"Hello"
user=> 1 2 3
1
2
3
```

我们还学习了如何通过使用括号来调用函数：

```java
user=> (+ 1 2 3)
6
```

值得注意的是，在此点，可以在一行的开头使用 "`;`" 来编写注释。任何以 "`;`" 开头的行将不会被评估：

```java
user=> ; This is a comment
user=> ; This line is not evaluated
```

函数的调用遵循以下结构：

```java
; (operator operand-1 operand-2 operand-3 …)
; for example:
user=> (* 2 3 4)
24
```

注意以下来自前一个示例的内容：

+   列表，由开括号和闭括号 `()` 表示，被评估为函数调用（或调用）。

+   当进行评估时，`*` 符号解析为执行乘法的函数。

+   `2`、`3` 和 `4` 被评估为自身，并将作为参数传递给函数。

考虑你在 *活动 1.01* 中编写的表达式，*执行基本操作*：`(* (+ 1 2 3) (- 10 3))`。将其视为树形结构也有助于可视化该表达式：

![图 1.1：表达式 (* (+ 1 2 3) (- 10 3)) 的树形表示]

](img/B14502_01_01.jpg)

图 1.1：表达式 (* (+ 1 2 3) (- 10 3)) 的树形表示

评估这个表达式包括减少树，从分支（最内层的列表）开始：`(* (+ 1 2 3) (- 10 3))` 变为 `(* 6 7)`，然后变为 `42`。

术语 **s-expression**（或符号表达式）常用来指代这类表达式。你可能会再次遇到它，所以了解 s-expression 是一个使用列表编写数据结构和代码的数据表示法，正如我们之前所展示的那样。

到目前为止，我们只使用字面量标量类型作为运算符的操作数，这些类型只包含一个值，例如数字、字符串、布尔值等。我们只使用列表来调用函数，而不是表示数据。让我们尝试创建一个表示数据但不表示“代码”的列表：

```java
user=> (1 2 3)
Execution error (ClassCastException) at user/eval255 (REPL:1).
java.lang.Long cannot be cast to clojure.lang.IFn
```

抛出了一个异常，因为列表的第一个元素（运算符）不是一个函数。

有一种特殊的语法可以防止列表被视为函数的调用：引用。创建字面量列表是通过在其前面添加引用 `'` 来完成的，所以让我们再试一次：

```java
user=> '(1 2 3)
(1 2 3)
user=> '("a" "b" "c" "d")
("a" "b" "c" "d")
```

太好了！通过防止表单的评估，我们现在可以编写列表的字面量表示。

这个概念将帮助我们为接下来要介绍的内容做好准备。然而，在这个时候注意到 Clojure 代码由数据结构组成，我们的程序可以生成那些相同的数据结构，这是非常迷人的。"代码是数据" 是 Lisp 世界中的一句名言，这是一个强大的概念，允许你的程序生成代码（称为 **元编程**）。如果你对这个概念还不熟悉，那么花一分钟时间思考和欣赏它的纯粹之美是值得的。我们将在解释 *第十一章* 中的 *宏* 时详细解释元编程技术。

# 基本特殊形式

到目前为止，我们一直在编写符合 Clojure 代码最简单规则的代码，但有一些行为不能简单地用正常函数来编码。例如，传递给函数的参数将始终被解析或评估，但如果我们不想评估运算符的所有操作数呢？这就是特殊形式发挥作用的时候。它们可以在 Clojure 读取源代码时为函数提供不同的评估规则。例如，特殊形式 `if` 可能不会评估其参数之一，这取决于第一个参数的结果。

在本节中，我们还将介绍几个其他特殊形式：

+   `when` 可以在只对条件为 *真值*（在布尔表达式的上下文中，值被认为是 *真值*）的情况感兴趣时使用。

+   `do` 可以用来执行一系列表达式并返回最后一个表达式的值。

+   `def` 和 `let`，这些是用于创建全局和局部绑定的特殊形式。

+   `fn` 和 `defn`，这些是用于创建函数的特殊形式。

所有这些特殊形式都有特殊的评估规则，所有这些规则我们都会通过以下三个练习来发现。

## 练习 1.03：使用 if、do 和 when

在这个练习中，我们将使用 `if`、`do` 和 `when` 形式来评估表达式。让我们开始吧：

1.  启动你的 REPL 并输入以下表达式：

    ```java
    user=> (if true "Yes" "No")
    "Yes"
    ```

1.  特殊形式 `if`，评估其第一个参数。如果其值为真，它将评估参数 2，否则（`else`），它将评估参数 3。它永远不会同时评估参数 2 和 3。

1.  我们可以嵌套表达式，并开始做一些更有趣的事情：

    ```java
    user=> (if false (+ 3 4) (rand))
    0.4833142431072903
    ```

    在这种情况下，计算 `(+ 3 4)` 将不会执行，`rand` 函数只会返回一个随机数（介于 0 和 1 之间）。

1.  但如果我们想在条件的分支中做更多的事情呢？我们可以用 `do` 来包裹我们的操作。让我们看看 `do` 是如何工作的：

    ```java
    user=> (doc do)
    -------------------------
    do
      (do exprs*)
    Special Form
      Evaluates the expressions in order and returns the value of
      the last. If no expressions are supplied, returns nil.
      Please see http://clojure.org/special_forms#do
      Evaluates the expressions in order and returns the value of
      the last. If no expressions are supplied, returns nil.
    nil
    ```

1.  要使用特殊形式，请输入以下表达式：

    ```java
    user=> (do (* 3 4) (/ 8 4) (+ 1 1))
    2
    ```

    在最后的`(+ 1 1)`表达式之前的所有表达式都被评估了，但只有最后一个表达式的值被返回。对于不改变世界状态的表达式来说，这看起来并不很有用，因此它通常用于副作用，如日志记录或任何其他类型的 I/O（文件系统访问、数据库查询、网络请求等）。

    你不必相信我的话，所以让我们通过在终端打印副作用来实验：

    ```java
    user=> (do (println "A proof that this is executed") (println "And this   too"))
    A proof that this is executed
    And this too
    nil
    ```

1.  最后，我们可以结合使用`if`和`do`来在条件分支中执行多个操作：

    ```java
    user=> (if true (do (println "Calculating a random number...") (rand)) (+ 1   2))
    Calculating a random number...
    0.8340057877906916
    ```

1.  技术上，你也可以省略第三个参数。在 REPL 中恢复之前的表达式，并移除最后一个表达式，即`(+ 1 2)`：

    ```java
    user=> (if true (do (println "Calculating a random number...") (rand)))
    Calculating a random number...
    0.5451384920081613
    user=> (if false (println "Not going to happen"))
    nil
    ```

    对于这种情况，我们有更好的构造：`when`运算符。当你只对条件执行的一个分支中的工作感兴趣时，使用`when`而不是组合`if`和`do`：

1.  输入以下表达式来使用`when`而不是`if`和`do`的组合：

    ```java
    user=> (when true (println "First argument") (println "Second argument")   "And the last is returned")
    First argument
    Second argument
    "And the last is returned"
    ```

通过完成这个练习，我们已经展示了特殊形式`if`、`do`和`when`的用法。现在我们可以编写包含多个语句以及条件表达式的表达式。

## 绑定

在 Clojure 中，我们使用术语*绑定*而不是*变量*和*赋值*，因为我们倾向于只将一个值绑定到一个符号上。在底层，Clojure 创建*变量*，因此你可能会遇到这个术语，但如果你不把它们视为经典的*变量*或可以改变的值，那就更好了。在本章中，我们不再使用术语变量，因为它可能会造成混淆。你可以使用`def`来定义全局绑定，使用`let`来定义局部绑定。

## 练习 1.04：使用 def 和 let

在这个练习中，我们将展示`def`和`let`关键字的使用，这些关键字用于创建绑定。让我们开始吧：

1.  特殊形式`def`允许你将值绑定到符号。在 REPL 中，输入以下表达式将值`10`绑定到`x`符号：

    ```java
    user=> (def x 10)
    #'user/x
    ```

    注意

    当 REPL 返回`#'user/x`时，它正在返回你刚刚创建的变量的引用。`user`部分表示变量定义的命名空间。`#'`前缀是一种引用变量的方式，这样我们就能看到符号而不是符号的值。

1.  评估表达式`x`，这将解析`x`符号到其值：

    ```java
    user=> x
    10
    ```

1.  技术上，你可以更改绑定，这在 REPL 中实验时是可行的：

    ```java
    user=> (def x 20)
    #'user/x
    user=> x
    20
    ```

    然而，在你的程序中并不推荐这样做，因为它可能会使代码难以阅读并复杂化其维护。现在，最好是将这样的绑定视为一个*常量*。

1.  你可以在另一个表达式中使用`x`符号：

    ```java
    user=> (inc x)
    21
    user=> x
    20
    ```

1.  无论`def`在哪里被调用，它都会将值绑定到当前命名空间中的符号。我们可以在`do`块中尝试定义一个局部绑定，看看会发生什么：

    ```java
    user=> x
    20
    user=> (do (def x 42))
    #'user/x
    user=> x
    42
    ```

    由 `def` 创建的绑定具有不确定的作用域（或动态作用域），可以视为“全局”。它们会自动命名空间，这是一个有用的特性，可以避免与现有名称冲突。

1.  如果我们只想让绑定在局部作用域或词法作用域内可用，我们可以使用特殊形式 `let`。输入以下表达式以创建 `y` 符号的词法绑定：

    ```java
    user=> (let [y 3] (println y) (* 10 y))
    3
    30
    ```

    `let` 以一个“向量”作为参数来创建局部绑定，然后是一系列表达式，这些表达式将被评估，就像它们在一个 `do` 块中一样。

    注意

    向量与列表类似，因为它们都是值的顺序集合。它们的底层数据结构不同，我们将在*第二章*，*数据类型和不可变性*中详细说明。现在，你只需要知道向量可以用方括号创建，例如，`[1 2 3 4]`。

1.  评估 `y` 符号：

    ```java
    user=> y
    Syntax error compiling at (REPL:0:0).
    Unable to resolve symbol: y in this context
    ```

    会抛出一个错误，即“无法在当前上下文中解析符号：y”，因为我们现在处于 `let` 块之外。

1.  输入以下表达式以创建 `x` 到值 `3` 的词法绑定，并看看它如何影响我们在*步骤 4*中创建的不确定（全局）绑定 `x`：

    ```java
    user=> (let [x 3] (println x))
    3
    nil
    user=> x
    42
    ```

    打印 `x` 得到值 `3`，这意味着“全局”的 `x` 符号被在 `println` 调用时的词法上下文临时覆盖或“阴影”。

1.  你可以使用 `let` 一次创建多个局部绑定，通过在向量中传递偶数个项。输入以下表达式以将 `x` 绑定到 `10`，将 `y` 绑定到 `20`：

    ```java
    user=> (let [x 10 y 20]  (str "x is " x " and y is " y))
    "x is 10 and y is 20"
    ```

1.  结合本节的概念，编写以下表达式：

    ```java
    user=> (def message "Let's add them all!")
    #'user/message
    user=> (let [x (* 10 3)
                 y 20
                 z 100]
                  (println message)
                  (+ x y z))
    Let's add them all!
    150
    ```

表达式跨越多行以提高可读性。

## 练习 1.05：使用 fn 和 defn 创建简单函数

定义函数的特殊形式是 `fn`。让我们直接进入正题，通过创建我们的第一个函数来开始：

1.  在你的 REPL 中输入以下表达式：

    ```java
    user=> (fn [])
    #object[user$eval196$fn__197 0x3f0846c6 "user$eval196$fn__197@3f0846c6"]
    ```

    我们刚刚创建了最简单的匿名函数，它不接受任何参数也不做任何事情，并返回了一个对象，即我们的无名称函数。

1.  创建一个接受名为 `x` 的参数并返回其平方值的函数（乘以自身）：

    ```java
    user=> (fn [x] (* x x))
    #object[user$eval227$fn__228 0x68b6f0d6 "user$eval227$fn__228@68b6f0d6"]
    ```

1.  记住，在 Clojure 中，表达式的第一个项将被调用，因此我们可以通过将匿名函数用括号括起来并提供作为表达式第二个项的参数来调用我们的匿名函数：

    ```java
    user=> ((fn [x] (* x x)) 2)
    4
    ```

    现在这是很好的，但不是很方便。如果我们想让函数可重用或可测试，给它一个名字会更好。我们可以在命名空间中创建一个符号并将其绑定到函数上。

1.  使用 `def` 将特殊形式 `fn` 返回的函数绑定到 `square` 符号：

    ```java
    user=> (def square (fn [x] (* x x)))
    #'user/square
    ```

1.  调用你新创建的函数以确保它正常工作：

    ```java
    user=> (square 2)
    4
    user=> (square *1)
    16
    user=> (square *1)
    256
    ```

1.  这种将 `def` 和 `fn` 结合的模式非常常见，以至于出于必要性产生了内置的 *宏*：`defn`。用 `defn` 而不是 `def` 和 `fn` 重新创建平方函数：

    ```java
    user=> (defn square [x] (* x x))
    #'user/square
    user=> (square 10)
    100
    ```

    你注意到`x`参数是以向量形式传递的吗？我们已经了解到向量是集合，因此我们可以将多个符号添加到参数的向量中。在调用函数时传递的值将在函数定义期间绑定到向量中提供的符号。

1.  函数可以接受多个参数，它们的主体可以由多个表达式组成（如隐式`do`块）。创建一个名为`meditate`的函数，它接受两个参数：一个字符串`s`和一个布尔值`calm`。该函数将打印一个介绍性消息并根据`calm`返回`s`的转换：

    ```java
    user=>
    (defn meditate [s calm]
      (println "Clojure Meditate v1.0")
      (if calm
        (clojure.string/capitalize s)
        (str (clojure.string/upper-case s) "!")))
    ```

    注意

    在 REPL 中编辑多行表达式可能会很麻烦。当我们开始创建更长的函数和跨越多行的表达式时，最好在 REPL 窗口旁边打开你最喜欢的编辑器窗口。保持这些窗口并排打开，在编辑器中编辑代码，将其复制到剪贴板，然后将其粘贴到 REPL 中。

    函数体包含两个主要表达式，第一个是一个带有`println`的副作用，第二个是一个`if`块，它将确定返回值。如果`calm`为`true`，它将礼貌地返回大写字母开头的字符串（第一个字符转换为大写），否则它将大声喊叫并返回所有字符都大写的字符串，并以感叹号结尾。

1.  让我们尝试确保我们的函数按预期工作：

    ```java
    user=> (meditate "in calmness lies true pleasure" true)
    Clojure Meditate v1.0
    "In calmness lies true pleasure"
    user=> (meditate "in calmness lies true pleasure" false)
    Clojure Meditate v1.0
    "IN CALMNESS LIES TRUE PLEASURE!"
    ```

1.  如果我们只使用第一个参数调用函数，它将抛出异常。这是因为我们定义的参数是必需的：

    ```java
    user=> (meditate "in calmness lies true pleasure")
    Execution error (ArityException) at user/eval365 (REPL:1).
    Wrong number of args (1) passed to: user/meditate
    ```

    在结束对这些函数的初步浏览之前，我们再来谈谈`doc-string`参数。当提供给`defn`时，它将允许你添加函数的描述。

1.  通过在函数参数之前添加文档字符串来为你的`square`函数添加文档：

    ```java
    user=>
    (defn square
      "Returns the product of the number `x` with itself"
      [x]
      (* x x))
    #'user/square
    ```

    文档字符串不仅在使用项目源代码时很有用——它还使`doc`函数能够访问它。

1.  使用`doc`查找你的`square`函数的文档：

    ```java
    user=> (doc square)
    -------------------------
    user/square
    ([x])
      Returns the product of the number `x` with itself
    nil
    ```

    重要的是要记住，文档字符串需要放在函数参数之前。如果它放在后面，字符串将被逐行评估作为函数体的一部分，而不会抛出错误。这是一个有效的语法，但它将无法在`doc`辅助程序和其他开发工具中使用。

    使用反引号` `` ` ` ``记录参数是一个好习惯，就像我们用` `` `x` ` ``做的那样，这样开发工具（如 IDE）就可以识别它们。

我们将在第三章“深入函数”中更深入地探讨函数，但这几个基本原理将使你在编写函数方面走得很远。

## 活动 1.02：预测大气二氧化碳水平

二氧化碳（CO2）是一种重要的温室气体，目前正在上升，威胁着我们星球上我们所知的生活。我们希望根据**国家海洋和大气管理局**（**NOAA**）提供的历史数据预测大气中未来 CO2 的水平：

![图 1.2：过去年份的 CO2 百万分之一（ppm）]

![图片](img/B14502_01_02.jpg)

图 1.2：过去年份的 CO2 百万分之一（ppm）

注意

前面的图表是从[`packt.live/35kUI7L`](https://packt.live/35kUI7L)获取的，数据来自 NOAA。

我们将以 2006 年为起点，CO2 水平为 382 ppm，使用简化的（且乐观的）线性函数来计算估计值，如下所示：*估计值 = 382 + ((年份 - 2006) * 2)*。

创建一个名为`co2-estimate`的函数，它接受一个名为`year`的整数参数，并返回该年份的估计 CO2 ppm 水平。

这些步骤将帮助你完成这项活动：

1.  打开你最喜欢的编辑器和旁边的 REPL 窗口。

1.  在你的编辑器中，定义两个常量，`base-co2`和`base-year`，分别赋值为 382 和 2006。

1.  在你的编辑器中，编写代码来定义`co2-estimate`函数，不要忘记使用 doc-string 参数对其进行文档化。

1.  你可能会想将函数体写在一行中，但嵌套过多的函数调用会降低代码的可读性。通过在`let`块中分解它们，也更容易推理过程的每一步。使用`let`来定义局部绑定`year-diff`，它是从`year`参数中减去 2006 的结果，来编写函数体。

1.  通过评估`(co2-estimate 2050)`测试你的函数。你应该得到`470`作为结果。

1.  使用`doc`查找你函数的文档，并确保它已被正确定义。

以下为预期输出：

```java
user=> (doc co2-estimate)
user/co2-estimate
([year])
  Returns a (conservative) year's estimate of carbon dioxide parts per million in     the atmosphere
nil
user=> (co2-estimate 2006)
382
user=> (co2-estimate 2020)
410
user=> (co2-estimate 2050)
470
```

注意

本活动的解决方案可以在第 679 页找到。

# 真实感、空值和相等性

到目前为止，我们一直在直观地使用条件表达式，可能基于它们在其他编程语言中通常的工作方式。在本节的最后，我们将详细回顾和解释布尔表达式以及相关的比较函数，从 Clojure 中的`nil`和真值开始。

`nil`是一个表示值缺失的值。在其他编程语言中，它也常被称为`NULL`。表示值的缺失是有用的，因为它意味着某些东西缺失了。

在 Clojure 中，`nil`是“假值”，这意味着在布尔表达式中评估时，`nil`的行为就像`false`。

`false`和`nil`是 Clojure 中唯一被视为*假值*的值；其他所有值都是真值。这个简单的规则是一种祝福（尤其是如果你来自像 JavaScript 这样的语言），它使我们的代码更易于阅读且更少出错。也许这只是因为当奥斯卡·王尔德写下“真理很少是纯粹的，也永远不会简单”时，Clojure 还没有出现。

## 练习 1.06：真理很简单

在这个练习中，我们将演示如何在条件表达式中处理布尔值。我们还将看到如何在条件表达式中玩转逻辑运算符。让我们开始吧：

1.  让我们先验证`nil`和`false`确实是*假值*：

    ```java
    user=> (if nil "Truthy" "Falsey")
    "Falsey"
    user=> (if false "Truthy" "Falsey")
    "Falsey"
    ```

1.  在其他编程语言中，在布尔表达式中，更多的值解析为`false`是常见的。但在 Clojure 中，请记住，只有`nil`和`false`是*假值*。让我们尝试几个例子：

    ```java
    user=> (if 0 "Truthy" "Falsey")
    "Truthy"
    user=> (if -1 "Truthy" "Falsey")
    "Truthy"
    user=> (if '() "Truthy" "Falsey")
    "Truthy"
    user=> (if [] "Truthy" "Falsey")
    "Truthy"
    user=> (if "false" "Truthy" "Falsey")
    "Truthy"
    user=> (if "" "Truthy" "Falsey")
    "Truthy"
    user=> (if "The truth might not be pure but is simple" "Truthy" "Falsey")
    "Truthy"
    ```

1.  如果我们想知道某物是否确实是`true`或`false`，而不仅仅是`真值`或`假值`，我们可以使用`true?`和`false?`函数：

    ```java
    user=> (true? 1)
    false
    user=> (if (true? 1) "Yes" "No")
    "No"
    user=> (true? "true")
    false
    user=> (true? true)
    true
    user=> (false? nil)
    false
    user=> (false? false)
    true
    ```

    `?`字符没有特殊的行为——它只是为返回布尔值的函数命名的一种约定。

1.  同样，如果我们想知道某物是`nil`而不是仅仅是*假值*，我们可以使用`nil?`函数：

    ```java
    user=> (nil? false)
    false
    user=> (nil? nil)
    true
    user=> (nil? (println "Hello"))
    Hello
    true
    ```

    记住`println`返回`nil`，因此前述代码中的最后一行输出是`true`。

    当布尔表达式组合在一起时，它们变得有趣。Clojure 提供了常用的运算符，即`and`和`or`。在这个阶段，我们只对*逻辑*`and`和*逻辑*`or`感兴趣。如果你想要玩转位运算符，你可以通过`(find-doc "bit-")`命令轻松找到它们。

    `and`返回它遇到的第一个*假值*（从左到右），并且当这种情况发生时，它不会评估表达式的其余部分。当传递给`and`的所有值都是*真值*时，`and`将返回最后一个值。

1.  通过传递*真值*和*假值*的混合值来实验`and`函数，观察生成的返回值：

    ```java
    user=> (and "Hello")
    "Hello"
    user=> (and "Hello" "Then" "Goodbye")
    "Goodbye"
    user=> (and false "Hello" "Goodbye")
    false
    ```

1.  让我们使用`println`并确保不是所有的表达式都被评估：

    ```java
    user=> (and (println "Hello") (println "Goodbye"))
    Hello
    nil
    ```

    `and`评估了第一个表达式，它打印了`Hello`并返回`nil`，这是*假值*。因此，第二个表达式没有被评估，`Goodbye`没有被打印。

    `or`的工作方式类似：它会返回遇到的第一个*真值*，并且当这种情况发生时，它不会评估表达式的其余部分。当传递给`or`的所有值都是*假值*时，`or`将返回最后一个值。

1.  通过传递*真值*和*假值*的混合值来实验`or`函数，观察生成的返回值：

    ```java
    user=> (or "Hello")
    "Hello"
    user=> (or "Hello" "Then" "Goodbye")
    "Hello"
    user=> (or false "Then" "Goodbye")
    "Then"
    ```

1.  再次使用`println`来确保表达式没有被全部评估：

    ```java
    user=> (or true (println "Hello"))
    true
    ```

    `or`评估了第一个表达式`true`并返回它。因此，第二个表达式没有被评估，`Hello`没有被打印。

## 相等和比较

在大多数命令式编程语言中，`=` 符号用于变量赋值。正如我们之前看到的，在 Clojure 中，我们有 `def` 和 `let` 来绑定名称和值。`=` 符号是一个用于相等的函数，如果所有参数都相等，它将返回 `true`。正如你现在可能已经猜到的，其他常见的比较函数也是作为函数实现的。`>`, `>=`, `<`, `<=`, 和 `=` 不是特殊语法，你可能已经对这些用法有了直觉。

## 练习 1.07：比较值

在这个最后的练习中，我们将探讨在 Clojure 中比较值的不同方法。让我们开始吧：

1.  首先，如果你的 REPL 还没有运行，请启动它。

1.  输入以下表达式来比较两个数字：

    ```java
    user=> (= 1 1)
    true
    user=> (= 1 2)
    false
    ```

1.  你可以向 `=` 操作符传递多个参数：

    ```java
    user=> (= 1 1 1)
    true
    user=> (= 1 1 1 -1)
    false
    ```

    在那种情况下，即使前三个参数相等，最后一个参数不相等，所以 `=` 函数返回 `false`。

1.  `=` 操作符不仅用于比较数字，还用于比较其他类型。评估以下一些表达式：

    ```java
    user=> (= nil nil)
    true
    user=> (= false nil)
    false
    user=> (= "hello" "hello" (clojure.string/reverse "olleh"))
    true
    user=> (= [1 2 3] [1 2 3])
    true
    ```

    注意

    在 Java 或其他面向对象的编程语言中，比较事物通常检查它们是否是存储在内存中的对象的相同实例，即它们的**身份**。然而，Clojure 中的比较是通过相等性而不是身份来进行的。比较值通常更有用，Clojure 也使其变得方便，但如果你想要比较身份，你可以通过使用 `identical?` 函数来实现。

1.  更令人惊讶的是，不同类型的序列也可以被认为是相等的：

    ```java
    user=> (= '(1 2 3) [1 2 3])
    true
    ```

    列表 `1` `2` `3` 等价于向量 `1` `2` `3`。集合和序列是 Clojure 的强大抽象，将在**第二章**，**数据类型和不可变性**中介绍。

1.  值得注意的是，`=` 函数也可以接受一个参数，在这种情况下，它将始终返回 `true`：

    ```java
    user=> (= 1)
    true
    user=> (= "I will not reason and compare: my business is to create.")
    true
    ```

    其他比较运算符，即 `>`, `>=`, `<`, 和 `<=`，只能用于数字。让我们从 `<` 和 `>` 开始。

1.  `<` 如果所有参数都是严格递增的，则返回 `true`。尝试评估以下表达式：

    ```java
    user=> (< 1 2)
    true
    user=> (< 1 10 100 1000)
    true
    user=> (< 1 10 10 100)
    false
    user=> (< 3 2 3)
    false
    user=> (< -1 0 1)
    true
    ```

    注意，`10` 后面跟着 `10` 并不是严格递增的。

1.  `<=` 类似，但相邻参数可以相等：

    ```java
    user=> (<= 1 10 10 100)
    true
    user=> (<= 1 1 1)
    true
    user=> (<= 1 2 3)
    true
    ```

1.  `>` 和 `>=` 有类似的行为，当它们的参数按递减顺序排列时返回 `true`。`>=` 允许相邻参数相等：

    ```java
    user=> (> 3 2 1)
    true
    user=> (> 3 2 2)
    false
    user=> (>= 3 2 2)
    true
    ```

1.  最后，`not` 操作符是一个有用的函数，当其参数是**假值**（`nil` 或 `false`）时返回 `true`，否则返回 `false`。让我们试一个例子：

    ```java
    user=> (not true)
    false
    user=> (not nil)
    true
    user=> (not (< 1 2))
    false
    user=> (not (= 1 1))
    false
    ```

    为了将这些内容综合起来，让我们考虑以下 JavaScript 代码：

    ```java
    let x = 50;
    if (x >= 1 && x <= 100 || x % 100 == 0) {
      console.log("Valid");
    } else {
      console.log("Invalid");
    }
    ```

    这段代码片段当数字 `x` 在 1 到 100 之间或 `x` 是 100 的倍数时打印 `Valid`，否则打印 `Invalid`。

    如果我们要将此转换为 Clojure 代码，我们会写出以下内容：

    ```java
    (let [x 50]
      (if (or (<= 1 x 100) (= 0 (mod x 100)))
        (println "Valid")
        (println "Invalid")))
    ```

    Clojure 代码中可能有一些额外的括号，但你可以争论 Clojure 比命令式 JavaScript 代码更易读。它包含更少的特定语法，我们不需要考虑运算符优先级。

    如果我们想使用“内联 if”转换 JavaScript 代码，我们会引入带有`?`和`:`的新语法，如下所示：

    ```java
    let x = 50;
    console.log(x >= 0 && x <= 100 || x % 100 == 0 ? "Valid" : "Invalid");
    ```

    Clojure 代码将变成以下样子：

    ```java
    (let [x 50]
      (println (if (or (<= 1 x 100) (= 0 (mod x 100))) "Valid" "Invalid")))
    ```

注意，这里没有新的语法，也没有什么新的东西要学习。你已经知道如何读取列表，而且几乎永远不需要做其他的事情。

这个简单的例子展示了`lists`（列表）的巨大灵活性：Clojure 和其他 Lisp 语言的基本构建块。

## 活动 1.03：meditate 函数 v2.0

在这个活动中，我们将通过用`calmness-level`替换`calm`布尔参数来改进我们在*练习 1.05*，*使用 fn 和 defn 创建简单函数*中编写的`meditate`函数。该函数将根据平静度打印作为第二个参数传递的字符串的转换。函数的规范如下：

+   `calmness-level`是一个介于`1`和`10`之间的数字，但我们将不会检查输入错误。

+   如果平静度严格小于`5`，我们认为用户很生气。函数应该将`s`字符串转换为大写并连接字符串"`, I TELL YA!`"。

+   如果平静度在`5`和`9`之间，我们认为用户处于平静和放松的状态。函数应该只将`s`字符串的第一个字母大写后返回。

+   如果平静度是`10`，用户已经达到了涅槃，被 Clojure 神灵附体。在它的迷幻状态中，用户传达了那些神圣实体的难以理解的语言。函数应该返回反转的`s`字符串。

    提示

    使用`str`函数连接字符串和`clojure.string/reverse`来反转字符串。如果你不确定如何使用它们，可以使用`doc`（例如，`(doc clojure.string/reverse)`）查找它们的文档。

这些步骤将帮助你完成这个活动：

1.  打开你最喜欢的编辑器和旁边的 REPL 窗口。

1.  在你的编辑器中，定义一个名为`meditate`的函数，它接受两个参数`calmness-level`和`s`，不要忘记编写它的文档。

1.  在函数体中，首先编写一个打印字符串`Clojure Meditate v2.0`的表达式。

1.  按照规范，编写第一个条件来测试平静度是否严格小于`5`。编写条件表达式的第一个分支（即`then`）。

1.  编写第二个条件，它应该嵌套在第一个条件的第二个分支（即`else`）中。

1.  编写第三个条件，它应该嵌套在第二个条件的第二个分支中。它将检查`calmness-level`是否正好是`10`，并在这种情况下返回`s`字符串的反转。

1.  通过传递具有不同平静程度的字符串来测试你的函数。输出应该类似于以下内容：

    ```java
    user=> (meditate "what we do now echoes in eternity" 1)
    Clojure Meditate v2.0
    "WHAT WE DO NOW ECHOES IN ETERNITY, I TELL YA!"
    user=> (meditate "what we do now echoes in eternity" 6)
    Clojure Meditate v2.0
    "What we do now echoes in eternity"
    user=> (meditate "what we do now echoes in eternity" 10)
    Clojure Meditate v2.0
    "ytinrete ni seohce won od ew tahw"
    user=> (meditate "what we do now echoes in eternity" 50)
    Clojure Meditate v2.0
    nil
    ```

1.  如果你一直使用 `and` 操作符来查找一个数字是否介于两个其他数字之间，请将你的函数重写，移除它并仅使用 `<=` 操作符。记住，`<=` 可以接受超过两个参数。

1.  在文档中查找 `cond` 操作符，并将你的函数重写，用 `cond` 替换嵌套条件。

    注意

    这个活动的解决方案可以在第 680 页找到。

# 摘要

在本章中，我们发现了如何使用 REPL 及其辅助工具。你现在能够搜索和发现新的函数，并在 REPL 中交互式地查找它们的文档。我们学习了 Clojure 代码是如何评估的，以及如何使用和创建函数、绑定、条件和比较。这些允许你创建简单的程序和脚本。

在下一章中，我们将探讨数据类型，包括集合，以及不可变性的概念。
