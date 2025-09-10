# 第四章 揭秘构建脚本

在前三章中，我们看到了 Gradle 通过在构建文件中添加几行代码就能为我们的构建添加许多有趣的功能。然而，这只是冰山一角。我们所探索的多数是 Gradle 随附的插件添加的任务。从我们的经验来看，我们知道项目构建永远不会这么简单。无论我们多么努力避免，它们都会有定制化。这就是为什么对于构建工具来说，添加自定义逻辑的能力极其重要。

此外，Gradle 的美也恰恰在于此。当我们决定扩展现有功能或完全偏离传统，想要做一些非常规的事情时，Gradle 不会妨碍我们。如果我们想在构建中添加一些逻辑，我们不需要编写 XML 汤或一大堆 Java 代码。我们可以创建自己的任务或扩展现有任务以做更多的事情。

这种灵活性伴随着学习 Groovy DSL 的非常温和的学习曲线。在本章中，我们将了解 Gradle 构建脚本的语法以及 Gradle 的一些关键概念。我们将涵盖以下主题：

+   一本 Groovy 入门指南，将帮助我们理解 Gradle 构建脚本的语法

+   在我们的构建中可用的两个重要对象，即 `project` 对象和 `task` 对象（们）

+   构建阶段和生命周期回调

+   任务的一些细节（任务执行和任务依赖）

# Groovy 用于 Gradle 构建脚本

要熟练使用 Gradle 并编写有效的构建脚本，我们需要了解一些 Groovy 的基础知识，Groovy 本身是一种出色的动态语言。如果我们对动态语言如 Ruby 或 Python 有任何经验，再加上 Java，我们将对 Groovy 感到非常熟悉。如果没有，只要知道大多数 Java 语法也是有效的 Groovy 语法，就足以让我们对 Groovy 感到高兴，因为我们可以从第一天开始编写 Groovy 代码并变得高效，而不需要学习任何新东西。

对于没有准备的人来说，Gradle 脚本可能一开始看起来有点难以理解。Gradle 构建脚本不仅使用 Groovy 语法，还使用一个丰富且具有表达性的 DSL，它提供了高级抽象来表示常见的构建相关逻辑。让我们快速看一下是什么让 Groovy 成为编写构建文件的一个很好的选择。

### 注意

使用 Groovy 编写构建逻辑并不新鲜。Gant 和 GMaven 已经使用 Groovy 来编写构建逻辑，以利用 Groovy 的简洁和表达性。GMavenPlus 是 GMaven 的继任者。它们所基于的工具，即 Ant 和 Maven，分别限制了 Gant 和 GMaven。

与其依赖现有工具仅为了添加语法增强，Gradle 是通过利用从过去工具中学到的经验来设计的。

## 为什么选择 Groovy？

Gradle 的核心主要用 Java 编写（见下文信息）。Java 是一种伟大的语言，但并不是编写脚本的最好选择。想象一下用 Java 编写脚本，我们可能需要为我们的主项目定义构建编写另一个项目，因为 Java 的冗长和仪式感。在上一代构建工具（Ant 和 Maven）中广泛使用的 XML，对于声明性部分来说是可以的，但不是编写逻辑的最佳选择。

### 注意

我们可以从 GitHub 下载 Gradle 的源代码，网址为 [`github.com/gradle/gradle`](https://github.com/gradle/gradle)。

Groovy 是 Java 的动态化身。如前所述，大多数 Java 语法也是有效的 Groovy 语法。如果我们知道 Java，我们就可以编写 Groovy 代码。考虑到现在能够编写 Java 的人数众多，这是一个很大的优势。

Groovy 的语法简洁、表达性强且功能强大。Groovy 是动态风味和类型使用的完美结合。它是少数几种具有可选类型特性的语言之一，即如果我们想提供类型信息，就可以提供；如果我们不想提供，就可以不提供。由于 Groovy 支持一等 lambda 和元编程功能，它是一种构建内部 DSL 的优秀语言。所有这些因素使它成为编写构建脚本的最佳候选人之一。

## Groovy 入门

虽然我们可以在 Groovy 中编写 Java 风格的代码，但如果我们在学习语言动态特性和 Groovy 提供的一些语法增强上投入一些时间，我们将能够编写更好的 Gradle 构建脚本和插件。如果我们还不了解 Groovy，这将是一件很有趣的事情。

让我们学习足够的 Groovy，以便能够正确理解 Gradle 脚本。我们将快速浏览 Groovy 的几个语言特性。

强烈建议尝试执行以下子节中的代码。此外，自己编写和尝试更多代码以探索 Groovy 将有助于我们加强语言基础的理解。本指南绝非详尽无遗，只是为了让 Groovy 的发展迈出第一步。

### 运行 Groovy 代码

最简单且推荐的方式是在本地安装最新的 Groovy SDK。Groovy 代码片段可以使用以下任何一种选项执行：

+   将代码片段保存到 `.groovy` 脚本中，然后使用以下代码在命令行中运行：

    ```java
    groovy scriptname.groovy
    ```

+   我们可以使用随 Groovy 安装包一起提供的 Groovy 控制台 GUI 来编辑和运行脚本。

+   我们还可以使用 Groovy shell，这是一个用于执行或评估 Groovy 语句和表达式的交互式 shell。

如果我们不希望在本地上安装 Groovy，那么：

+   我们可以使用 Groovy 控制台在浏览器中在线运行 Groovy 代码，网址为 [`groovyconsole.appspot.com`](http://groovyconsole.appspot.com)。

+   我们还可以通过创建任务并将代码片段放入其中（我们也可以将它们放在任何任务之外，它仍然会在配置阶段运行它们）在构建脚本中运行 Groovy 代码。

### 变量

在 Groovy 脚本中，`def` 关键字可以定义一个变量（取决于上下文）：

```java
def a = 10
```

然而，`a` 的类型是在运行时根据它指向的对象类型来决定的。大致来说，声明为 `def` 的引用可以指向任何 `Object` 或其子类。

声明一个更具体的类型同样有效，并且应该在我们想要有类型安全时使用：

```java
Integer b = 10
```

我们也可以使用 Java 原始数据类型，但请注意，在 Groovy 中它们实际上不是原始类型。它们仍然是第一类对象，实际上是针对相应数据类型的 Java 包装类。让我们通过以下示例来确认：

```java
int c = 10
println c.getClass()
```

它打印以下输出：

```java
class java.lang.Integer
```

这表明 `c` 是一个对象，因为我们可以在它上面调用方法，并且 `c` 的类型是 `Integer`。

我们建议尽可能使用特定类型，因为这增加了可读性，并帮助 Groovy 编译器通过捕获无效赋值来早期检测错误。这也帮助 IDE 进行代码补全。

#### 字符串

与 Java 不同，单引号（`''`）是字符串字面量，而不是 `char`：

```java
String s = 'hello'
```

当然，Java 的常规字符串字面量（`""`）也可以使用，但在 Groovy 中它们被称为 GStrings。它们具有字符串插值或变量或表达式的内联展开的附加功能：

```java
def name = "Gradle"
println "$name is an awesome build tool"
```

这将打印以下输出：

```java
Gradle is an awesome build tool

```

``${var}` 和 `$var` 都是有效的，但包装（`${}`）更适合，并且在复杂或较长的表达式中是必需的。例如：

```java
def number = 4
println "number is even ? ${number % 2 == 0 }"
```

它将打印以下内容：

```java
number is even ? true

```

我们所有人都记得在 Java 中，为了产生多行字符串，需要在每一行的末尾添加 `+ "\\n"`。那些日子已经过去了，因为 Groovy 支持多行字符串字面量。多行字面量以三个单引号或双引号开始（与 GString 功能相同），并以三个单引号或双引号结束：

```java
def multilineString = '''\
    Hello
    World
'''
println multilineString
```

它将打印以下内容：

```java
 Hello
 World

```

第 1 行上的正斜杠是可选的，用于排除第一个新行。如果我们不放置正斜杠，输出开头将会有一个额外的空行。

此外，查看 `stripMargin` 和 `stripIndent` 方法以了解对前导空白的特殊处理。

如果我们的字面量包含大量的转义字符（例如，正则表达式），那么我们最好使用“斜杠”字符串字面量，它以单个正斜杠（`/`）开始和结束：

```java
def r = /(\d)+/
println r.class
```

它将打印以下内容：

```java
class java.lang.String

```

在上面的例子中，如果我们必须使用普通字符串，那么我们不得不在字符类 `d` 前面的反斜杠进行转义。它看起来如下所示：

```java
"(\\d)+"
```

#### 正则表达式

Groovy 支持一个模式操作符（`~`），当应用于字符串时，会给出一个模式对象：

```java
def pattern = ~/(\d)+/
println pattern.class
```

它打印以下内容：

```java
class java.util.regex.Pattern

```

我们还可以使用查找操作符直接将字符串与模式匹配：

```java
if ("groovy" ==~ /gr(.*)/)
  println "regex support rocks"
```

它将打印以下内容：

```java
regex support rocks
```

#### 闭包

Groovy 中的**闭包**是一段代码块，它可以被赋值给引用或像任何其他变量一样传递。这个概念在许多其他语言中被称为**lambda**，包括 Java 8 或函数指针。

### 注意

Lambda 自 Java 8 以来一直被支持，但其语法与 Groovy 闭包的语法略有不同。你不需要使用 Java 8 就可以在 Groovy 中使用闭包。

如果我们没有接触过上述任何一种，那么为了很好地理解这个概念，就需要进行一些详细的阅读，因为它为后续许多高级主题奠定了基础。闭包本身就是一个很大的主题，深入的讨论超出了本书的范围。

闭包几乎就像一个常规方法或函数，但它也可以被赋值给变量。由于它可以被赋值给变量，它也必须是一个对象；因此，它将有自己的方法：

```java
def cl1 = {
    println "hello world!"
}
```

在这里，代码块被赋值给一个名为`cl1`的变量。现在，代码块可以在未来的调用方法中执行，或者`cl1`变量可以在以后传递并执行：

```java
cl1.call()
```

毫不奇怪，它打印了以下内容：

```java
hello world!

```

由于闭包就像方法一样，它们也可以接受参数：

```java
def cl2 = { n ->
    println "value of param : $n"
}
cl2.call(101)
```

它打印了以下内容：

```java
value of param : 101

```

就像方法一样，它们也可以返回值。如果没有显式声明`return`语句，闭包的最后一个表达式将自动返回。

当我们有接受闭包的方法时，闭包开始发光。例如，`times`方法在整数上可用，它接受一个闭包并执行它，次数与整数的值相同；每次调用时，它将当前值作为如果我们是从`0`循环到该值一样传递：

```java
3.times(cl2)
```

它打印了以下内容：

```java
value of param : 0
value of param : 1
value of param : 2

```

我们还可以将代码块内联并直接传递给方法：

```java
3.times { println it * it }
```

它打印了以下内容：

```java
0
1
4

```

有一个特殊的变量叫做`it`，如果闭包没有定义其参数，它将在代码块的作用域内可用。在上面的例子中，我们使用`it`访问传递给代码块的数量，并将其自身相乘以获得其平方。

闭包在诸如回调处理等情况下非常有用，而在 Java 7 及以下版本中，我们不得不使用匿名接口实现来达到相同的结果。

### 数据结构

Groovy 支持常用数据结构的字面量声明，这使得代码更加简洁，同时又不牺牲可读性。

#### 列表

Groovy 依赖于经过充分测试的 Java Collection API，并在底层使用相同的类，但增加了一些额外的方法和语法糖：

```java
def aList = []
println aList.getClass()
```

它打印了以下内容：

```java
class java.util.ArrayList

```

### 注意

在 Groovy 中，`[]` 实际上是 Java 的 `List` 实例，而不是数组。

让我们创建另一个带有一些初始内容的列表：

```java
def anotherList = ['a','b','c']
```

多亏了运算符重载，我们可以在列表上直观地使用许多运算符。例如，使用`anotherList[1]`将给我们`b`。

下面是一些更多有用的运算符的例子。这个例子将两个列表相加，并将结果赋值给列表变量：

```java
def list = [10, 20, 30] + [40, 50]
```

这会将 `60` 添加到列表中：

```java
list  <<  60 
```

以下两个示例简单地从一个列表中减去另一个列表：

```java
list = list – [20, 30, 40] 
list  -= [20,30,40]
```

遍历列表同样简单直观：

```java
list.each {println it}
```

它将打印以下内容

```java
10
50
60

```

传递给 `each` 的闭包为列表中的每个元素执行，元素作为闭包的参数。因此，前面的代码遍历列表并打印每个元素的值。注意 `it` 的使用，它是列表当前元素的句柄。

#### 集合

定义一个集合与列表类似，但除此之外，我们必须使用 `as Set`：

```java
def aSet = [1,2,3] as Set
println aSet.class
```

这将打印以下内容：

```java
class java.util.LinkedHashSet

```

由于选择的实现类是 `LinkedHashSet`，`aSet` 将保持插入顺序。

或者，声明变量的类型以获取正确的实现：

```java
TreeSet anotherSet = [1,2,3]
println anotherSet.class
```

这将打印以下内容：

```java
class java.util.TreeSet

```

向集合中添加元素就像使用间接操作符的列表一样。其他集合接口方法也是可用的：

```java
aSet << 4
aSet << 3
println aSet
```

这将打印以下内容：

```java
[1, 2, 3, 4]

```

由于集合是集合实现，它按定义消除了重复，所以我们看不到条目 `4` 两次。

#### 映射

映射是任何动态语言最重要的数据结构之一。因此，它在 Groovy 的语法中得到了应有的位置。映射可以使用映射字面量 `[:]` 来声明：

```java
def a = [:]
```

默认选择的实现是 `java.util.LinkedHashMap`，它保留了插入顺序：

```java
def tool = [version:'2.8', name:'Gradle', platform:'all']
```

注意，键不是字符串字面量，但它们会自动转换为字符串：

```java
println tool.name
println tool["version"]
println tool.get("platform")
```

我们可以通过索引操作符和点操作符以及普通的 `get()` 方法来访问值。

我们可以使用索引操作符、点操作符以及当然的 `put()` 方法在映射中添加和更新数据：

```java
tool.version = "2.9"
tool["releaseDate"] = "2015-11-17"
tool.put("platform", "ALL")
```

### 方法

以下更像是 Java 类型的方法，当然这也是一个有效的 Groovy 方法：

```java
int sum(int a, int b) {
  return a + b;
}
```

前面的方法可以简洁地重写如下：

```java
def sum(a, b) {
  a + b
}
```

我们没有指定返回类型，只是声明了 `def`，这意味着方法可以返回任何 `Object` 或子类引用。然后，我们省略了形式参数的类型，因为对于方法的形式参数来说，声明 `def` 是可选的。在第 2 行，我们省略了 `return` 语句，因为方法的最后一个表达式的评估会自动返回。我们还省略了分号，因为它也是可选的。

这两个示例都是有效的 Groovy 方法声明。然而，建议读者明智地选择类型，因为它们提供了类型安全，并作为方法的活文档。如果我们没有声明参数的类型，就像前面的方法一样，求和 (1,"2") 也将成为一个有效的方法调用，而且更糟糕的是，它返回了没有异常的意外结果。 

#### 调用方法

在 Groovy 中，在许多情况下可以省略方法调用中的括号。以下两种情况都是有效的方法调用。

```java
sum(1,2)  
sum 1, 2  
```

#### 参数的默认值

有很多次，我们想要通过提供一个默认值来使参数可选，这样如果调用者没有提供值，则使用默认值。看看以下示例：

```java
def divide(number, by=2) {
    number/by
}

println divide (10, 5)
println divide (10)
```

它打印以下内容：

```java
2
5

```

如果我们提供了将要使用的 `by` 参数的值，则默认值 `2` 将被假定为该参数。

#### 带有 map 参数/命名参数的方法

Groovy 不支持像 Python 那样的命名参数，但 Map 提供了对相同功能的非常接近的近似：

```java
def method(Map options) {
    def a = options.a ?: 10
    def b = options.b ?: 20
}
```

在前面的代码中，我们期望 map 包含键 `a` 和 `b`。

### 注意

在第 2 行和第 3 行，注意 elvis 运算符 `?:`，如果存在值且是 *truthy*，则返回左侧值；否则返回右侧（默认）值。它基本上是以下代码的简写：

```java
options.a ? options.a : 10
```

现在，这个方法可以这样调用：

```java
method([a:10,b:20])
```

我们可以省略方括号 (`[]`)，因为 maps 在方法调用中有特殊支持：

```java
method(a:10, b:20)
```

现在，它看起来明显像是命名参数。参数的顺序并不重要，并且不需要传递所有参数。此外，括号包裹是可选的，就像任何方法调用一样：

```java
method b:30, a:40
method b:30
```

#### 带有 varags 的方法

如同 Java 一样，varags 用 `...` 表示，但提供类型是可选的：

```java
def sumSquares(...numbers) {
    numbers.collect{ it * it }.sum()
}
sumSquares 1, 2, 3
```

在前面的示例中，数字是数组，它具有接受闭包并按顺序转换集合中每个元素的 `collect` 方法，以产生一个新的集合。在这种情况下，我们转换集合中的平方数。最后，我们使用内置的 sum 方法来求所有平方的和。

#### 带闭包参数的方法

闭包很重要，因此 Groovy 如果闭包是方法签名中的最后一个参数，则具有特殊的闭包语法：

```java
def myMethod (param, cls) {
    ...
}
```

然后，这个方法可以这样调用：

```java
myMethod(1,{ ... })
myMethod 2, {... }
myMethod(3) {...}
```

在这些中，第三个是特殊的语法支持，其中括号仅包裹其他参数，而闭包则写在括号外，就像是一个方法体。

### 类

Groovy 中的类声明与 Java 类类似，但仪式较少。类默认是公共的。它们可以使用 `extends` 从其他类继承，或者使用 `implmenets` 实现接口。

以下是一个非常简单的类 `Person` 的定义，它有两个属性，`name` 和 `age`：

```java
class Person {
  def name, age
}
```

我们可以使用更具体的类型来代替 `def` 用于属性。

#### 构造函数

除了默认构造函数之外，Groovy 中的类还有一个特殊的构造函数，它接受类的属性映射。以下是它的用法：

```java
def person = new Person(name:"John Doe", age:35)
```

在前面的代码中，我们使用特殊构造函数创建了 `person` 对象。参数是键值对，其中键是类中属性的名称。为键提供的值将设置对应的属性。

#### 属性

Groovy 在语言级别上支持属性。在前面的类中，`name` 和 `age`，与 Java 不同，不仅仅是字段，而且还是具有其 getter 和 setter 的类的属性。字段默认是私有的，它们的公共访问器和修改器（getter 和 setter）是自动生成的。

我们可以在上面创建的`person`对象上调用`getAge()`/`setAge()`和`getName()`/`setName()`方法。然而，有一个更简洁的方法可以做到这一点。我们可以像访问公共字段一样访问属性，但幕后 Groovy 会通过 getter 和 setter 进行路由。让我们试试：

```java
println person.age
person.age = 36
println person.age
```

它打印以下内容：

```java
35
36

```

在前面的代码中，在第 1 行，`person.age`实际上是调用`person.getAge()`，因此它返回了人的年龄。然后，我们使用`person.age`和右侧的赋值运算符及值更新了年龄。我们没有更新字段，但它内部通过 setter `setAge()`进行传递。这之所以可能，是因为 Groovy 提供了对属性的语法支持。

我们可以为所需的字段提供自己的 getter 和/或 setter，这将优先于生成的那个，但只有在我们需要在这些地方编写一些逻辑时才是必要的。例如，如果我们想设置一个年龄的正值，那么我们可以提供自己的`setAge()`实现，并且每次更新属性时都会使用它：

```java
  void setAge(age){
    if (age < 0) 
      throw new IllegalArgumentException("age must be a positive number")
    else
      this.age = age
  }
```

属性的支持导致类定义中的样板代码显著减少，并增强了可读性。

### 小贴士

属性在 Groovy 中是一等公民。从现在开始，每次我们提到属性时，不要在属性和字段之间混淆。

#### 实例方法

我们可以向类中添加实例方法和静态方法，就像我们在 Java 中做的那样：

```java
def speak(){
  println "${this.name} speaking"
}
```

```java
static def now(){
  new Date().format("yyyy-MM-dd HH:mm:ss")
}
```

如我们上面所讨论的，方法部分没有使用类，而是直接应用于类内部的方法。

### 注意

**脚本即类**

实际上，我们上面讨论的方法都在类内部，它们不是自由浮动的函数。由于脚本被透明地转换为类，所以我们感觉就像在使用函数。

我相信到目前为止你已经享受了 Groovy。Groovy 还有很多内容可以介绍，但我们不得不将焦点转回到 Gradle 上。然而，我希望我已经激发了你足够的对 Groovy 的好奇心，这样你就可以欣赏它作为一种语言，并自己探索更多。参考资料部分包含了一些很好的资源。

## 再次看看应用插件

现在我们已经了解了基本的 Groovy，让我们将其应用于 Gradle 构建脚本的环境中。在早期章节中，我们已经看到了应用插件的语法。它看起来如下所示：

```java
apply plugin: 'java'
```

如果我们仔细观察，`apply`是一个方法调用。我们可以将参数括在括号中：

```java
apply(plugin: 'java')
```

一个接收映射的方法可以像命名参数一样传递键值。然而，为了更清晰地表示映射，我们可以将参数用`[]`括起来：

```java
apply([plugin: 'java'])
```

最后，`apply`方法在`project`对象上隐式应用（我们将在本章接下来的部分中看到这一点）。因此，我们也可以在`project`对象的引用上调用它：

```java
project.apply([plugin: 'java'])
```

因此，从前面的例子中，我们可以看到将插件应用于项目的声明仅仅是一个语法糖，它实际上是对 `project` 对象的方法调用。我们只是在使用 Gradle API 编写 Groovy 代码。一旦我们意识到这一点，我们对理解构建脚本语法的看法就会发生根本性的改变。

# Gradle – 一个面向对象的构建工具

如果我们以面向对象的方式思考构建系统，以下类会立刻浮现在我们的脑海中：

+   一个代表正在构建的系统的 `project`

+   一个封装需要执行的部分构建逻辑的 `task`

好吧，我们很幸运。正如我们所预期的那样，Gradle 创建了 `project` 和 `task` 类型的对象。这些对象在我们的构建脚本中是可访问的，以便我们进行自定义。当然，其底层实现并不简单，API 非常复杂。

`project` 对象是暴露给构建脚本并配置的 API 的核心部分。在脚本中，`project` 对象是可用的，这样就可以在 `project` 对象上智能地调用没有对象引用的方法。我们已经在上一节中看到了一个例子。大多数构建脚本语法都可以通过阅读项目 API 来理解。

`task` 对象是为在构建文件中直接声明的每个任务以及插件而创建的。我们已经在 第一章 中创建了一个非常简单的任务，即 *运行您的第一个 Gradle 任务*，并在 第二章 和 第三章 中使用了来自插件的任务，即 *构建 Java 项目* 和 *构建 Web 应用程序*。

### 注意

正如我们所见，一些任务在我们的构建文件中已经可用，我们无需在构建文件中添加任何一行代码（例如 `help` 任务和 `tasks` 任务等）。即使是这些任务，我们也会有任务对象。

我们很快就会看到这些对象是如何以及何时被创建的。

# 构建阶段

每次调用时，Gradle 构建都会遵循一个非常简单的生命周期。构建过程会经过三个阶段：初始化、配置和执行。当调用 `gradle` 命令时，并非我们构建文件中编写的所有代码都会按顺序从上到下依次执行。只有与当前构建阶段相关的代码块会被执行。此外，构建阶段的顺序决定了代码块何时执行。例如，任务配置与任务执行。理解这些阶段对于正确配置我们的构建非常重要。

## 初始化

Gradle 首先确定当前项目是否有子项目，或者它是否是构建中的唯一项目。对于多项目构建，Gradle 确定哪些项目（或子模块，许多人更喜欢这样称呼它们）需要包含在构建中。我们将在下一章中看到多项目构建。然后，Gradle 为根项目和每个项目的子项目创建一个`Project`实例。对于到目前为止我们所看到的单模块项目，在这个阶段没有太多可以配置的。

## 配置

在这个阶段，参与项目的构建脚本将与初始化阶段创建的相应项目对象进行评估。在多模块项目中，评估是按广度进行的，也就是说，在评估和配置子项目之前，所有兄弟项目都将被评估和配置。然而，这种行为是可以配置的。

注意，执行脚本并不意味着任务也被执行。为了快速验证这一点，我们只需在`build.gradle`文件中添加一个`println`语句，并创建一个打印消息的任务：

```java
task myTask << {
  println "My task is executed"
}
// The following statement will execute before any task 
println "build script is evaluated"
```

如果我们执行以下代码：

```java
$ gradle -q myTask

```

我们将看到以下输出：

```java
build script is evaluated
My task is executed

```

实际上，选择任何内置任务也可以，例如`help`：

```java
$ gradle -q help

```

在任何任务执行之前，我们仍然会看到我们的`build script is evaluated`消息。为什么是这样？

当一个脚本被评估时，脚本中的所有语句都会按顺序执行。这就是为什么根级别的`println`语句会被执行。如果你注意到，任务动作实际上是一个闭包；因此，它仅在语句执行期间附加到任务上。然而，闭包本身尚未执行。动作闭包内的语句只有在任务执行时才会执行，这发生在下一个阶段。

任务仅在此时进行配置。无论将要调用哪些任务，所有任务都将被配置。Gradle 为任务准备了一个**有向无环图**（**DAG**）表示，以确定任务的依赖关系和执行顺序。

## 执行

在这个阶段，Gradle 根据诸如通过命令行参数传递的任务名称和当前目录等参数确定需要运行的任务。这就是任务动作将被执行的地方。因此，在这里，如果任务要运行，动作闭包实际上会执行。

### 注意

在后续调用中，Gradle 智能地确定哪些任务实际上需要运行，哪些可以跳过。例如，对于编译任务，如果源文件在最后一次构建后没有变化，再次编译就没有意义了。在这种情况下，执行可能会被跳过。我们可以在输出中看到这样的任务，标记为`UP-TO-DATE`：

```java
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:compileTestJava UP-TO-DATE
:processTestResources UP-TO-DATE
:testClasses UP-TO-DATE
:test UP-TO-DATE
```

在前面的输出中，由于与前一次构建没有变化，Gradle 实际上跳过了每个任务。然而，这不会发生在我们编写的自定义任务上，除非我们告诉 Gradle 确定任务是否需要执行的逻辑。

## 生命周期回调

Gradle 在生命周期事件的不同阶段提供了各种钩子来执行代码。我们可以在构建脚本中实现回调接口或提供回调闭包。例如，我们可以使用`project`上的`beforeEvaluate`和`afterEvaluate`方法来监听项目评估前后的事件。我们不会逐一查看它们，但`Project`和`Gradle`（接口名称不要与工具本身的名称混淆）API 以及 DSL 文档是检查可用回调的正确地方，如果我们觉得需要实现生命周期回调的话。

# Gradle 项目 API

如前所述，Gradle 在初始化阶段为我们为每个`build.gradle`创建了一个`project`对象。这个对象可以通过`project`引用在我们的构建脚本中使用。作为一个 API 的核心部分，这个对象上有许多方法和属性。

## 项目方法

我们甚至没有意识到，我们已经在调用`project`对象的`methods`了，尽管我们一直在使用项目 API。根据一些管理规则，如果未提供明确的引用，构建脚本中的所有顶级方法调用都是在项目对象上进行的。

让我们重写第一章中非常简单的构建文件，即*运行您的第一个 Gradle 任务*，以使用项目引用进行方法调用：

```java
project.apply plugin: 'java'

project.repositories {
    mavenCentral()
}

project.dependencies {
    testCompile 'junit:junit:4.11'
}
```

正如我们在本章前面看到的，`apply`是`project`上的一个方法。所谓的`dependencies`块实际上是在`project`上接受闭包的`dependencies()`方法。对于`repositories`部分也是如此。我们可以将闭包块用括号括起来，使其看起来像是一个普通的方法调用：

```java
project.repositories({...})
project.dependencies({...})
```

在这个对象上还有许多其他有趣的方法，我们将在接下来的章节中看到，这些方法要么有，要么没有对`project`对象的明确引用。

### 项目属性

在`project`对象上有几个属性可用。一些属性是只读属性，如`name`、`path`、`parent`等，而其他属性则是可读可写的。

例如，我们可以设置`project.description`来提供项目的描述。我们可以使用`project.version`属性来设置项目的版本。这个版本将被其他任务，如`Jar`使用，以在生成的工件中包含版本号。

### 注意

我们不能从`build.gradle`文件中更改`project.name`，但我们可以使用同一项目中的`settings.gradle`来设置项目名称。当我们学习多项目构建时，我们将更详细地了解这个文件。

除了直接通过名称访问属性外，我们还可以使用以下在`project`对象上的方法来访问属性。

要检查属性是否存在，请使用以下方法：

```java
boolean hasProperty(String propertyName)
```

要获取给定属性名的属性值，请使用以下方法：

```java
Object property(String propertyName)
```

要设置给定属性名的属性值，请使用以下方法：

```java
void setProperty(String name, Object value)
```

例如，让我们创建一个包含以下内容的`build.gradle`文件：

```java
description = "a sample project"
version = "1.0"

task printProperties << {
    println project.version
    println project.property("description")
}
```

执行以下任务：

```java
$ gradle -q printProperties
1.0
a sample project

```

如前所述，在 Groovy 中，我们可以使用`property = value`语法来调用 setter。我们在`project`对象上设置了`description`和`version`属性。然后，我们添加了一个任务，该任务使用`project`引用和`description`使用`project`对象上的`property()`方法来打印版本。

我们在上面看到的属性必须在项目上存在，否则构建将失败，并显示`Could not find property …`消息。

### 项目上的额外属性

Gradle 使得在项目上存储用户定义的属性变得非常容易，同时仍然能够享受项目属性语法的便利。我们只需要使用`ext`命名空间将值分配给自定义属性。然后，这个属性可以在项目上像常规项目属性一样访问。以下是一个示例：

```java
ext.abc = "123"
task printExtraProperties << {
    println project.abc
    println project.property("abc")
    println project.ext.abc
}
```

执行以下任务：

```java
$ gradle -q printExtraProperties
123
123
123

```

在前面的示例中，我们声明了一个名为`abc`的自定义属性，并将其赋值为`123`。我们没有使用`project`引用，因为它在脚本根级别上是隐式可用的。在任务操作中，我们首先使用项目引用直接打印它，就像它是在`Project`上的一个属性一样。然后，我们使用`property()`方法和`project.ext`引用来访问它。请注意，在任务的动作闭包内部，我们应该使用`project`引用以避免任何歧义。

额外的属性将在子项目（模块）中可用。也可以在其他对象上设置额外的属性。

### 注意

我们本可以使用局部变量，通过使用`def`声明它。然而，这样的变量在词法作用域之外不可访问。此外，它们也不可查询。

尽管我们已经查看了一些方法和属性，但在这里涵盖所有这些是不切实际的；因此，花些时间阅读`project`接口的 API 和 DSL 文档是值得的。

# 任务

如我们所见，`task`是一个执行某些构建逻辑的命名操作。它是一个构建工作的单元。例如，`clean`、`compile`、`dist`等，如果我们必须为我们自己的项目编写任务，这些是典型的构建任务，很容易浮现在我们的脑海中。任务在某种程度上类似于 Ant 的目标。

创建任务的简单方法如下：

```java
task someTask
```

在我们进一步讨论任务之前，让我们花点时间思考一下任务创建。

我们使用了语句的`taskName`任务形式。

如果我们将其重写为`task (taskName)`，它将立即看起来像方法调用。

我们可能已经猜到的，前面的方法在项目对象上是可用的。

因此，我们也可以写出以下内容之一：

+   `project.task "myTask"`

+   `project.task("myTask")`

注意，在后面的示例中，我们必须传递任务名称作为字符串。`task taskName` 是一种特殊形式，我们可以使用 `taskName` 作为字面量而不是字符串。这是通过 Groovy AST 转换魔法实现的。

项目有几种创建任务对象的 `task` 方法：

```java
Task task(String name)

Task task(String name, Closure configureClosure)

Task task(Map<String, ?> args, String name)

Task task(Map<String, ?> args, String name, Closure configureClosure)
```

然而，本质上，我们可以在创建任务时传递一些键值作为命名参数，并传递一个配置闭包来配置任务。

我们实际上创建了一个 `Task` 类型（确切类名现在并不重要）的对象。我们可以查询属性并在此对象上调用方法。Gradle 优雅地使这个 `task` 对象可用于使用。在优雅的 DSL 之下，我们实际上正在编写一个脚本，以优雅的面向对象方式创建构建逻辑。

## 将操作附加到任务

一个 `Task` 对象，例如上面创建的，实际上并没有做什么。事实上，它没有附加任何操作。我们需要将操作附加到 `Task` 对象上，以便 Gradle 在运行任务时执行这些操作。

一个 `Task` 对象有一个名为 `doLast` 的方法，它接受一个闭包。Gradle 确保传递给此方法的闭包按它们传递的顺序执行：

```java
someTask.doLast({
    println "this should be printed when the task is run"
})
```

我们现在可以再次调用 `doLast`：

```java
someTask.doLast({
    println "this should ALSO be printed when the task is run"
})
```

此外，在另一种语法中：

```java
someTask {
    doLast {
        println "third line that should be printed"
    }
}
```

有多种方法可以向任务添加 `doLast` 逻辑，但最符合习惯用法，也许是最简洁的方法如下：

```java
someTask << {
    println "the action of someTask"
}
```

就像 `Project` 对象一样，我们有一个 `Task` 对象，其中方法和属性是可访问的。然而，与 `Project` 对象不同，它不是在脚本的顶层隐式可用，而仅在任务的配置范围内。直观地说，我们可以认为每个 `build.gradle` 将会有多个 `Task` 对象。我们将在稍后看到访问 `Task` 对象的各种方法。

## 任务流程控制

项目内的任务可能相互依赖。在本节中，我们将看到项目任务中可能存在不同类型的关系。

### dependsOn

有一些任务的执行依赖于其他任务的成功完成。例如，为了创建可分发的 JAR 文件，代码应该首先被编译，并且 "class" 文件应该已经存在。在这种情况下，我们不希望用户明确指定所有任务及其顺序，如下所示：

```java
$ gradle compile dist

```

这可能会导致错误。我们可能会忘记包含一个任务，或者如果有多个任务依赖于前一个任务的完成，顺序可能会变得复杂。我们希望能够指定是否：

```java
task compile << {
    println 'compling the source'
}

task dist(dependsOn: compile) << {
    println "preparing a jar dist"
}
```

### finalizedBy

我们还可以声明，如果调用一个任务，它应该跟随另一个任务，即使没有明确调用另一个任务。这与 `dependsOn` 相反，其中另一个任务在调用任务之前执行。在 `finalizedBy` 的情况下，另一个任务在调用任务的执行之后执行：

```java
task distUsingTemp << {
  println ("preapring dist using a temp dir")
}

task cleanup << {
  println("removing tmp dir")
}

distUsingTemp.finalizedBy cleanup
```

### onlyIf

我们可以指定一个条件，如果满足该条件，则任务将被执行：

```java
cleanup.onlyIf { file("/tmp").exists()}
```

### mustRunAfter 和 shouldRunAfter

有时候，如果我们想要以特定的顺序排列任务，而这种关系并不完全等同于`dependsOn`。例如，如果我们执行以下命令：

```java
$ gradle build clean

```

然后，无关的任务将按照在命令行上指定的顺序执行，在这种情况下这没有意义。

在这种情况下，我们可能需要添加以下代码行：

```java
build.mustRunAfter clean
```

这告诉 Gradle，如果任务图中有这两个任务，那么`build`必须在`clean`运行之后运行。在这里，`build`不依赖于`clean`。

`shouldRunAfter`和`mustRunAfter`之间的区别在于前者对 Gradle 更具暗示性，但并不强制 Gradle 始终遵循这种顺序。在以下两种情况下，Gradle 可能不会遵守`shouldRunAfter`：

+   当它引入循环排序时的情况。

+   在并行执行的情况下，当只有`shouldRunAfter`任务尚未成功完成且其他依赖项已满足时，则`shouldRunAfter`将被忽略。

## 动态创建任务

Gradle 的一个优点是我们可以动态创建任务。这意味着在编写构建时，任务的名称和逻辑并不完全已知，但根据某些变量参数，任务将自动添加到我们的 Gradle 项目中。

让我们通过一个示例来尝试理解：

```java
10.times { number ->
  task "dynamicTask$number" << {
    println "this is dynamic task number # $number "
  }
}
```

在前面的虚构示例中，我们正在动态创建并添加十个任务到我们的构建中。尽管它们只是打印任务编号，但能够动态创建和添加任务到我们的项目中是非常强大的。

## 设置默认任务

到目前为止，我们一直使用带有任务名称（s）的`gradle`命令行界面。这在本质上有点重复，尤其是在开发期间，Gradle 这样的工具为我们提供了覆盖：

```java
defaultTasks "myTaskName", "myOtherTask"
```

设置默认任务是个明智的选择，这样如果我们没有指定任何任务名称，默认的任务就会被执行。

在前面的示例中，不带任何参数从命令行运行`gradle`会依次运行`defaultTasks`中指定的默认任务。

## 任务类型

我们之前看到的任务都是临时的。我们必须编写任务动作的代码，每当任务执行时都需要执行。然而，无论我们构建哪个项目，总有许多任务的动作逻辑不需要改变，如果我们有能力对现有逻辑进行一些配置更改。例如，当你复制文件时，只有源、目标和包含/排除模式会改变，但如何从一处复制文件到另一处并遵守包含/排除模式的实际逻辑保持不变。所以，如果一个项目中需要两个类似复制任务的，比如说`copyDocumentation`和`deployWar`，我们真的需要编写整个逻辑来复制选定的文件两次吗？

对于非常小的构建（例如我们章节中的示例），这可能是可行的，但这种方法扩展性不好。如果我们继续编写任务操作来执行这些日常操作，那么我们的构建脚本将很快膨胀到一个难以管理的状态。

自定义任务类型是 Gradle 将可重用的构建逻辑抽象到自定义任务类中的解决方案，这些类在任务对象上公开输入/输出配置变量。这有助于我们调整类型化任务以满足我们的特定需求。这有助于我们保持常见的构建逻辑可重用和可测试。

另一个与临时代码任务操作相关的问题是它本质上是命令式的。为了工具的灵活性，Gradle 允许我们在构建脚本中以命令式方式编写自定义逻辑。然而，在构建脚本中过度使用命令式代码会使构建脚本难以维护。Gradle 应尽可能以声明式方式使用。命令式逻辑应封装在自定义任务类中，同时向用户公开任务配置以便配置。在 Gradle 的术语中，自定义任务类被称为 **增强任务**。

自定义任务类型充当一个模板，为常见的构建逻辑提供一些合理的默认值。我们仍然需要在构建中声明一个任务，但我们只需告诉 Gradle 这个任务类型以及配置这个任务类型的设置，而不是再次编写整个任务操作块。Gradle 已经提供了许多自定义任务类型；例如，`Copy`、`Exec`、`Delete`、`Jar`、`Sync`、`Test`、`JavaCompile`、`Zip` 等等。我们也可以轻松编写我们自己的增强任务。我们将简要地查看这两种场景。

### 使用任务类型

我们可以使用以下语法配置类型为 `Copy` 的任务：

```java
task copyDocumentation(type:Copy) {
from file("src/docs/html")
into file("$buildDir/docs")
}
```

在前面的示例中，第一个重要的区别是我们传递了一个带有自定义任务类名（在这种情况下为 `Copy`）作为值的 `type` 键。此外，请注意没有 `doLast` 或间接（`<<`）操作符。我们传递给这个任务的闭包实际上在构建配置阶段执行。闭包内的方法调用被委派给隐式可用的 `task` 对象，该对象正在被配置。我们没有在这里编写任何逻辑，只是为类型为 `Copy` 的任务提供了配置。在继续编写临时代码任务操作之前，查看可用的自定义任务总是值得的。

### 创建任务类型

如果我们现在回顾一下，我们为示例任务编写的任务操作代码主要是打印给定消息到 `System.out` 的 `println` 语句。现在，想象一下，如果我们发现 `System.out` 不符合我们的要求，我们更应该使用文本文件从任务中打印消息。我们需要遍历所有任务并更改实现以将消息写入文件而不是 `println`。

处理此类变化需求有更好的方法。我们可以通过提供自己的任务类型来利用任务类型的功能。让我们在`build.gradle`中放入以下代码：

```java
class Print extends DefaultTask {
  @Input
  String message = "Welcome to Gradle"

  @TaskAction
  def print() {
    println "$message"
  }
}

task welcome(type: Print)

task thanks(type: Print) {
  message = "Thanks for trying custom tasks"
}

task bye(type: Print)
bye.message = "See you again"

thanks.dependsOn welcome
thanks.finalizedBy bye
```

在前面的代码示例中：

+   我们首先创建了一个类（它将成为我们的任务类型），该类扩展了`DefaultTask`，这是在 Gradle 中已经定义好的。

+   接下来，我们使用`@Input`在名为`message`的属性上声明了一个可配置的任务输入。我们的任务消费者可以配置这个属性。

+   然后，我们在`print`方法上使用了`@TaskAction`注解。当我们的任务被调用时，这个方法会被执行。它只是使用`println`来打印`message`。

+   然后，我们声明了三个任务；所有任务都使用不同的方式来配置我们的任务。注意没有任务动作。

+   最后，我们应用了任务流程控制技术来声明任务依赖关系。

如果我们现在运行`thanks`任务，我们可以看到预期的输出，如下所示：

```java
$ gradle -q thanks
Welcome to Gradle
Thanks for trying custom tasks
See you again

```

在这里需要注意的几点如下：

+   如果我们想要更改打印逻辑的实现，我们只需要在一个地方进行更改，那就是我们自定义任务类的`print`方法。

+   使用任务类型的任务被使用，并且它们的工作方式与任何其他任务一样。它们也可以使用`doLast {}`、`<< {}`来执行任务动作闭包，但通常不是必需的。

# 参考文献

下几节提到了一些 Groovy 的有用参考资料。

## Groovy

对于 Groovy，有大量的在线参考资料可用。我们可以从这里开始：

+   对于进一步阅读，请参考 Groovy 的在线文档[`www.groovy-lang.org/documentation.html`](http://www.groovy-lang.org/documentation.html)

+   更多 Groovy 资源的参考信息可在[`github.com/kdabir/awesome-groovy`](https://github.com/kdabir/awesome-groovy)找到。

这里有一份关于 Groovy 的书籍列表：

+   《Groovy in Action》一书可在[`www.manning.com/books/groovy-in-action-second-edition`](https://www.manning.com/books/groovy-in-action-second-edition)找到。

+   《Groovy Cookbook》一书可在[`www.packtpub.com/application-development/groovy-2-cookbook`](https://www.packtpub.com/application-development/groovy-2-cookbook)找到。

+   《Programming Groovy 2》一书可在[`pragprog.com/book/vslg2/programming-groovy-2`](https://pragprog.com/book/vslg2/programming-groovy-2)找到。

## 本章中使用的 Gradle API 和 DSL

Gradle 的官方 API 和 DSL 文档是探索和学习本章中讨论的各种类的好地方。这些 API 和 DSL 非常丰富，值得我们花时间阅读。

+   `Project`：

    +   API 文档：[`gradle.org/docs/current/javadoc/org/gradle/api/Project.html`](http://gradle.org/docs/current/javadoc/org/gradle/api/Project.html)

    +   DSL 文档：[`gradle.org/docs/current/dsl/org.gradle.api.Project.html`](http://gradle.org/docs/current/dsl/org.gradle.api.Project.html)

+   `Gradle`（接口）：

    +   API 文档：[`gradle.org/docs/current/javadoc/org/gradle/api/invocation/Gradle.html`](http://gradle.org/docs/current/javadoc/org/gradle/api/invocation/Gradle.html)

    +   DSL 文档：[`gradle.org/docs/current/dsl/org.gradle.api.invocation.Gradle.html`](http://gradle.org/docs/current/dsl/org.gradle.api.invocation.Gradle.html)

+   `任务`：

    +   API 文档：[`www.gradle.org/docs/current/javadoc/org/gradle/api/Task.html`](http://www.gradle.org/docs/current/javadoc/org/gradle/api/Task.html)

    +   DSL 文档：[`www.gradle.org/docs/current/dsl/org.gradle.api.Task.html`](http://www.gradle.org/docs/current/dsl/org.gradle.api.Task.html)

# 摘要

我们本章从 Groovy 语言的快速功能概述开始，涵盖了一些有助于我们理解 Gradle 语法和编写更好的构建脚本的议题。然后，我们研究了 Gradle 向我们的构建脚本公开的 API 以及如何通过 DSL 消费 API。我们还涵盖了 Gradle 构建阶段。然后，我们探讨了任务可以被创建、配置、相互依赖以及默认运行的方式。

在阅读完本章后，我们应该能够理解 Gradle DSL，而不仅仅是试图记住语法。我们现在能够阅读并理解任何给定的 Gradle 构建文件，并且现在能够轻松地编写自定义任务。

本章可能感觉有点长且复杂。我们应该抽出一些时间来练习和重新阅读那些不清楚的部分，并且查阅本章中给出的在线参考资料。接下来的章节将会更加顺畅。
