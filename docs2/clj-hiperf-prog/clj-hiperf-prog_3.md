# 第三章。依赖 Java

由于托管在 JVM 上，Clojure 有几个方面真正有助于理解 Java 语言和平台。这种需求不仅是因为与 Java 的互操作性或理解其实现，还因为性能原因。在某些情况下，Clojure 默认可能不会生成优化的 JVM 字节码；在另一些情况下，你可能希望超越 Clojure 数据结构提供的性能——你可以通过 Clojure 使用 Java 替代方案来获得更好的性能。本章将讨论这些 Clojure 的方面。在本章中，我们将讨论：

+   检查从 Clojure 源代码生成的 Java 和字节码

+   数值和原始类型

+   操作数组

+   反射和类型提示

# 检查 Clojure 代码的等效 Java 源代码

检查给定 Clojure 代码的等效 Java 源代码可以提供对其性能可能产生影响的深刻见解。然而，除非我们将命名空间编译到磁盘上，否则 Clojure 仅在运行时生成 Java 字节码。当使用 Leiningen 进行开发时，只有 `project.clj` 文件中 `:aot` 向量下的选定命名空间被输出为包含字节码的编译 `.class` 文件。幸运的是，有一个简单快捷的方法可以知道 Clojure 代码的等效 Java 源代码，那就是 AOT 编译命名空间，然后使用 Java 字节码反编译器将字节码反编译成等效的 Java 源代码。

有多种商业和开源的 Java 字节码反编译器可用。我们将在这里讨论的一个开源反编译器是 **JD-GUI**，你可以从其网站下载（[`jd.benow.ca/#jd-gui`](http://jd.benow.ca/#jd-gui)）。使用适合你操作系统的版本。

## 创建一个新项目

让我们看看如何从 Clojure 代码精确地得到等效的 Java 源代码。使用 Leiningen 创建一个新的项目：`lein new foo`。然后编辑 `src/foo/core.clj` 文件，添加一个 `mul` 函数来找出两个数字的乘积：

```java
(ns foo.core)

(defn mul [x y]
  (* x y))
```

## 将 Clojure 源代码编译成 Java 字节码

现在，要编译 Clojure 源代码成字节码并将它们输出为 `.class` 文件，运行 `lein compile :all` 命令。它将在项目的 `target/classes` 目录下创建 `.class` 文件，如下所示：

```java
target/classes/
`-- foo
    |-- core$fn__18.class
    |-- core__init.class
    |-- core$loading__4910__auto__.class
    `-- core$mul.class
```

你可以看到 `foo.core` 命名空间已经被编译成了四个 `.class` 文件。

## 将 .class 文件反编译成 Java 源代码

假设你已经安装了 JD-GUI，反编译 `.class` 文件就像使用 JD-GUI 应用程序打开它们一样简单。

![将 .class 文件反编译成 Java 源代码](img/B04596_03_01.jpg)

检查 `foo.core/mul` 函数的代码如下：

```java
package foo;

import clojure.lang.AFunction;
import clojure.lang.Numbers;
import clojure.lang.RT;
import clojure.lang.Var;

public final class core$mul extends AFunction
{
  public static final Var const__0 = (Var)RT.var("clojure.core", "*");

  public Object invoke(Object x, Object y) { x = null; y = null; return Numbers.multiply(x, y);
  }
}
```

从反编译的 Java 源代码中很容易理解，foo.core/mul 函数是 foo 包中 core$mul 类的一个实例，该类扩展了 clojure.lang.AFunction 类。我们还可以看到，在方法调用(Object, Object)中，参数类型是 Object 类型，这意味着数字将被装箱。以类似的方式，你可以反编译任何 Clojure 代码的类文件来检查等效的 Java 代码。如果你能结合对 Java 类型以及可能的反射和装箱的了解，你就可以找到代码中的次优位置，并专注于需要改进的地方。

## 在没有本地清除的情况下编译 Clojure 源代码

注意方法调用中的 Java 代码，其中说 `x = null; y = null;` ——代码是如何丢弃参数、将它们设置为 null 并有效地将两个 null 对象相乘的呢？这种误导性的反编译是由于本地清除引起的，这是 Clojure JVM 字节码实现的一个特性，在 Java 语言中没有等效功能。

从 Clojure 1.4 版本开始，编译器支持在动态 `clojure.core/*compiler-options*` 变量中使用 `:disable-locals-clearing` 键，我们无法在 `project.clj` 文件中进行配置。因此，我们无法使用 `lein compile` 命令，但我们可以使用 `lein repl` 命令启动一个 **REPL** 来编译类：

```java
user=> (binding [*compiler-options* {:disable-locals-clearing true}] (compile 'foo.core))
foo.core
```

这将在本节前面看到的相同位置生成类文件，但没有 `x = null; y = null;`，因为省略了本地清除。

# 数值、装箱和原始类型

**数值**是标量。关于数值的讨论被推迟到本章，唯一的原因是 Clojure 中数值的实现有强烈的 Java 基础。从版本 1.3 开始，Clojure 采用了 64 位数值作为默认值。现在，`long` 和 `double` 是惯用的默认数值类型。请注意，这些是原始的 Java 类型，不是对象。Java 中的原始类型导致高性能，并在编译器和运行时级别具有多个优化。局部原始类型是在栈上创建的（因此不会对堆分配和 GC 贡献），可以直接访问而无需任何类型的解引用。在 Java 中，也存在数值原始类型的对象等效物，称为 **装箱数值**——这些是分配在堆上的常规对象。装箱数值也是不可变对象，这意味着 JVM 不仅在读取存储的值时需要解引用，而且在需要创建新值时还需要创建一个新的装箱对象。

显然，boxed 数值比它们的原始等效类型要慢。Oracle HotSpot JVM 在启动时使用`-server`选项，会积极内联那些包含对原始操作调用的函数（在频繁调用时）。Clojure 在多个级别自动使用**原始数值**。在`let`块、`loop`块、数组以及算术操作（`+`、`-`、`*`、`/`、`inc`、`dec`、`<`、`<=`、`>`、`>=`）中，会检测并保留原始数值。以下表格描述了原始数值及其 boxed 等效类型：

| 原始数值类型 | Boxed 等效类型 |
| --- | --- |
| 字节型 (1 字节) | `java.lang.Byte` |
| 短整型 (2 字节) | `java.lang.Short` |
| 整型 (4 字节) | `java.lang.Integer` |
| 浮点型 (4 字节) | `java.lang.Float` |
| 长整型 (8 字节) | `java.lang.Long` |
| 双精度浮点型 (8 字节) | `java.lang.Double` |

在 Clojure 中，有时你可能会发现由于运行时缺少类型信息，数值作为 boxed 对象传递或从函数返回。即使你无法控制此类函数，你也可以强制转换值以将其视为原始值。`byte`、`short`、`int`、`float`、`long`和`double`函数从给定的 boxed 数值值创建原始等效值。

Lisp 的传统之一是提供正确的([数值塔](http://en.wikipedia.org/wiki/Numerical_tower))算术实现。当发生溢出或下溢时，低类型不应该截断值，而应该提升到更高类型以保持正确性。Clojure 遵循这一约束，并通过素数([素数](http://en.wikipedia.org/wiki/Prime_(symbol)))函数提供**自动提升**：`+'`, `-'`, `*'`, `inc'`, 和 `dec'`。自动提升以牺牲一些性能为代价提供了正确性。

在 Clojure 中，也存在任意长度或精度的数值类型，允许我们存储无界数字，但与原始类型相比性能较差。`bigint`和`bigdec`函数允许我们创建任意长度和精度的数字。

如果我们尝试执行可能超出其最大容量的原始数值操作，操作会通过抛出异常来保持正确性。另一方面，当我们使用素数函数时，它们会自动提升以提供正确性。还有另一组称为不检查操作的操作，这些操作不检查溢出或下溢，可能返回不正确的结果。

在某些情况下，它们可能比常规和素数函数更快。这些函数是`unchecked-add`、`unchecked-subtract`、`unchecked-multiply`、`unchecked-divide`、`unchecked-inc`和`unchecked-dec`。我们还可以通过`*unchecked-math*`变量启用常规算术函数的不检查数学行为；只需在源代码文件中包含以下内容：

```java
(set! *unchecked-math* true)
```

在算术运算中，常见的需求之一是用于找出自然数除法后的商和余数的除法。Clojure 的`/`函数提供了一个有理数除法，返回一个比例，而`mod`函数提供了一个真正的模除法。这些函数比计算除法商和余数的`quot`和`rem`函数要慢。

# 数组

除了对象和原始数据类型之外，Java 还有一种特殊的集合存储结构，称为**数组**。一旦创建，数组在不需要复制数据和创建另一个数组来存储结果的情况下不能增长或缩小。数组元素在类型上总是同质的。数组元素类似于可以修改它们以保存新值的位置。与列表和向量等集合不同，数组可以包含原始元素，这使得它们在没有 GC 开销的情况下成为一种非常快速的存储机制。

数组通常是可变数据结构的基础。例如，Java 的`java.lang.ArrayList`实现内部使用数组。在 Clojure 中，数组可用于快速数值存储和处理、高效算法等。与集合不同，数组可以有一个或多个维度。因此，您可以在数组中布局数据，如矩阵或立方体。让我们看看 Clojure 对数组的支持：

| 描述 | 示例 | 备注 |
| --- | --- | --- |
| 创建数组 | `(make-array Integer 20)` | 类型为（boxed）整数的数组 |
|   | `(make-array Integer/TYPE 20)` | 原始类型整数的数组 |
|   | `(make-array Long/TYPE 20 10)` | 原始 long 类型二维数组 |
| 创建原始数组 | `(int-array 20)` | 大小为 20 的原始整数数组 |
|   | `(int-array [10 20 30 40])` | 从向量创建的原始整数数组 |
| 从集合创建数组 | `(to-array [10 20 30 40])` | 从可序列化创建的数组 |
|   | `(to-array-2d [[10 20 30][40 50 60]])` | 从集合创建的二维数组 |
| 克隆数组 | `(aclone (to-array [:a :b :c]))` |   |
| 获取数组元素 | `(aget array-object 0 3)` | 获取 2-D 数组中索引[0][3]的元素 |
| 修改数组元素 | `(aset array-object 0 3 :foo)` | 在 2-D 数组中设置索引[0][3]的值为 obj :foo |
| 修改原始数组元素 | `(aset-int int-array-object 2 6 89)` | 在 2-D 数组中设置索引[2][6]的值为 89 |
| 获取数组长度 | `(alength array-object)` | `alength`比`count`要快得多 |
| 遍历数组 | `(def a (int-array [10 20 30 40 50 60]))``(seq``(amap a idx ret``(do (println idx (seq ret))``(inc (aget a idx)))))` | 与 map 不同，`amap`返回一个非懒加载的数组，在遍历数组元素时速度要快得多。注意，`amap`只有在正确类型提示的情况下才更快。下一节将介绍类型提示。 |
| 对数组进行归约 | `(def a (int-array [10 20 30 40 50 60]))``(areduce a idx ret 0``(do (println idx ret)``(+ ret idx)))` | 与归约不同，`areduce`在数组元素上要快得多。请注意，只有当正确类型提示时，归约才更快。下一节将介绍类型提示。 |
| 转换为原始数组 | `(ints int-array-object)` | 与类型提示一起使用（见下一节） |

与`int-array`和`ints`类似，还有其他类型的函数：

| 数组构造函数 | 原始数组转换函数 | 类型提示（不适用于变量） | 泛型数组类型提示 |
| --- | --- | --- | --- |
| 布尔数组 | booleans | `^booleans` | `^"[Z"` |
| 字节型数组 | bytes | `^bytes` | `^"[B"` |
| 短整型数组 | shorts | `^shorts` | `^"[S"` |
| 字符数组 | chars | `^chars` | `^"[C"` |
| 整型数组 | ints | `^ints` | `^"[I"` |
| 长整型数组 | longs | `^longs` | `^"[J"` |
| 浮点数组 | floats | `^floats` | `^"[F"` |
| 双精度数组 | doubles | `^doubles` | `^"[D"` |
| 对象数组 | –– | `^objects` | `^"[Ljava.lang.Object"` |

数组之所以比其他数据结构更受欢迎，主要是因为性能，有时也因为互操作性。在类型提示数组和使用适当的函数处理它们时，应格外小心。

# 反射和类型提示

有时，由于 Clojure 是动态类型的，Clojure 编译器无法确定要调用的对象类型。在这种情况下，Clojure 使用**反射**，这比直接方法分派要慢得多。Clojure 解决这个问题的方法是所谓的**类型提示**。类型提示是一种用静态类型注解参数和对象的方法，这样 Clojure 编译器就可以生成用于高效分派的字节码。

知道在哪里放置类型提示的最简单方法是打开代码中的反射警告。考虑以下确定字符串长度的代码：

```java
user=> (set! *warn-on-reflection* true)
true
user=> (def s "Hello, there")
#'user/s
user=> (.length s)
Reflection warning, NO_SOURCE_PATH:1 - reference to field length can't be resolved.
12
user=> (defn str-len [^String s] (.length s))
#'user/str-len
user=> (str-len s)
12
user=> (.length ^String s)  ; type hint when passing argument
12
user=> (def ^String t "Hello, there")  ; type hint at var level
#'user/t
user=> (.length t)  ; no more reflection warning
12
user=> (time (dotimes [_ 1000000] (.length s)))
Reflection warning, /private/var/folders/cv/myzdv_vd675g4l7y92jx9bm5lflvxq/T/form-init6904047906685577265.clj:1:28 - reference to field length can't be resolved.
"Elapsed time: 2409.155848 msecs"
nil
user=> (time (dotimes [_ 1000000] (.length t)))
"Elapsed time: 12.991328 msecs"
nil
```

在前面的代码片段中，我们可以清楚地看到，使用反射的代码与不使用反射的代码在性能上有很大的差异。在处理项目时，你可能希望所有文件都打开反射警告。你可以在 Leiningen 中轻松做到这一点。只需在`project.clj`文件中添加以下条目：

```java
:profiles {:dev {:global-vars {*warn-on-reflection* true}}}
```

这将在每次通过 Leiningen 在开发工作流程中开始任何类型的调用时自动打开警告反射，例如 REPL 和测试。

## 原始数组

回想一下前一小节中关于`amap`和`areduce`的例子。如果我们打开反射警告运行它们，我们会收到警告说它们使用了反射。让我们给它们添加类型提示：

```java
(def a (int-array [10 20 30 40 50 60]))
;; amap example
(seq
 (amap ^ints a idx ret
    (do (println idx (seq ret))
      (inc (aget ^ints a idx)))))
;; areduce example
(areduce ^ints a idx ret 0
  (do (println idx ret)
    (+ ret idx)))
```

注意，原始数组提示`^ints`在变量级别不起作用。因此，如果你定义了变量`a`，如下所示，它将不起作用：

```java
(def ^ints a (int-array [10 20 30 40 50 60]))  ; wrong, will complain later
(def ^"[I" a (int-array [10 20 30 40 50 60]))  ; correct
(def ^{:tag 'ints} a (int-array [10 20 30 40 50 60])) ; correct
```

这种表示法用于整数数组。其他原始数组类型有类似类型提示。请参考前一小节中关于各种原始数组类型的类型提示。

## 原始类型

原始局部变量的类型提示既不是必需的，也是不允许的。然而，你可以将函数参数类型提示为原始类型。Clojure 允许函数最多有四个参数可以进行类型提示：

```java
(defn do-something
  [^long a ^long b ^long c ^long d]
  ..)
```

### 注意

封装箱可能会导致某些情况下的结果不是原始类型。在这种情况下，你可以使用相应的原始类型强制转换它们。

## 宏和元数据

在宏中，类型提示的方式与其他代码部分不同。由于宏是关于转换**抽象语法树**（**AST**），我们需要有一个心理图来表示转换，并且我们应该在代码中添加类型提示作为元数据。例如，如果`str-len`是一个用于查找字符串长度的宏，我们使用以下代码：

```java
(defmacro str-len
  [s]
  `(.length ~(with-meta s {:tag String})))
;; below is another way to write the same macro
(defmacro str-len
  [s]
  `(.length ~(vary-meta s assoc :tag `String)))
```

在前面的代码中，我们通过标记类型为`String`来修改符号`s`的元数据，在这个例子中恰好是`java.lang.String`类。对于数组类型，我们可以使用`[Ljava.lang.String`来表示字符串对象的数组，以及其他类似类型。如果你尝试使用之前列出的`str-len`，你可能注意到这仅在将字符串绑定到局部变量或变量时才有效，而不是作为字符串字面量。为了减轻这种情况，我们可以将宏编写如下：

```java
(defmacro str-len
  [s]
  `(let [^String s# ~s] (.length s#)))
```

在这里，我们将参数绑定到一个类型提示的 gensym 局部变量，因此调用`.length`时不会使用反射，并且不会发出此类反射警告。

通过元数据进行的类型提示也适用于函数，尽管符号不同：

```java
(defn foo [] "Hello")
(defn foo ^String [] "Hello")
(defn foo (^String [] "Hello") (^String [x] (str "Hello, " x)))
```

除了前面片段中的第一个示例外，它们都提示返回`java.lang.String`类型。

### 字符串连接

Clojure 中的`str`函数用于连接和转换成字符串标记。在 Java 中，当我们编写`"hello" + e`时，Java 编译器将其转换为使用`StringBuilder`的等效代码，在微基准测试中比`str`函数快得多。为了获得接近 Java 的性能，在 Clojure 中我们可以使用类似的机制，通过宏直接使用 Java 互操作来避免通过`str`函数的间接操作。**Stringer** ([`github.com/kumarshantanu/stringer`](https://github.com/kumarshantanu/stringer))库采用相同的技巧，在 Clojure 中实现快速字符串连接：

```java
(require '[stringer.core :as s])
user=> (time (dotimes [_ 10000000] (str "foo" :bar 707 nil 'baz)))
"Elapsed time: 2044.284333 msecs"
nil
user=> (time (dotimes [_ 10000000] (s/strcat "foo" :bar 707 nil 'baz)))
"Elapsed time: 555.843271 msecs"
nil
```

在这里，Stringer 也在编译阶段积极连接字面量。

## 杂项

在类型（如`deftype`中），可变实例变量可以可选地注解为`^:volatile-mutable`或`^:unsynchronized-mutable`。例如：

```java
(deftype Counter [^:volatile-mutable ^long now]
  ..)
```

与`defprotocol`不同，`definterface`宏允许我们为方法提供返回类型提示：

```java
(definterface Foo
  (^long doSomething [^long a ^double b]))
```

`proxy-super`宏（在`proxy`宏内部使用）是一个特殊情况，你不能直接应用类型提示。原因是它依赖于由`proxy`宏自动创建的隐式`this`对象。在这种情况下，你必须显式地将`this`绑定到一个类型：

```java
(proxy [Object][]
  (equals [other]
    (let [^Object this this]
      (proxy-super equals other))))
```

在 Clojure 中，类型提示对于性能非常重要。幸运的是，我们只需要在需要时进行类型提示，并且很容易找出何时需要。在许多情况下，类型提示带来的收益会超过代码内联带来的收益。

# 使用数组/数值库提高效率

你可能已经注意到在前面的章节中，当处理数值时，性能很大程度上取决于数据是否基于数组和原始类型。为了实现最佳效率，程序员可能需要在计算的各个阶段都非常细致地将数据正确地强制转换为原始类型和数组。幸运的是，Clojure 社区的性能爱好者及早意识到这个问题，并创建了一些专门的开源库来减轻这个问题。

## HipHip

**HipHip** 是一个用于处理原始类型数组的 Clojure 库。它提供了一个安全网，即它严格只接受原始数组参数来工作。因此，静默传递装箱的原始数组作为参数总是会引发异常。HipHip 宏和函数很少需要在操作期间程序员进行类型提示。它支持原始类型的数组，如 `int`、`long`、`float` 和 `double`。

HipHip 项目可在 [`github.com/Prismatic/hiphip`](https://github.com/Prismatic/hiphip) 找到。

到编写本文时，HipHip 的最新版本是 0.2.0，支持 Clojure 1.5.x 或更高版本，并标记为 Alpha 版本。HipHip 为所有四种原始类型的数组提供了一套标准操作：整数数组操作在命名空间 `hiphip.int` 中；双精度数组操作在 `hiphip.double` 中；等等。所有操作都针对相应类型进行了类型提示。在相应命名空间中，`int`、`long`、`float` 和 `double` 的所有操作基本上是相同的，除了数组类型：

| 类别 | 函数/宏 | 描述 |
| --- | --- | --- |
| 核心函数 | `aclone` | 类似于 `clojure.core/aclone`，用于原始类型 |
|   | `alength` | 类似于 `clojure.core/alength`，用于原始类型 |
|   | `aget` | 类似于 `clojure.core/aget`，用于原始类型 |
|   | `aset` | 类似于 `clojure.core/aset`，用于原始类型 |
|   | `ainc` | 将数组元素按指定值增加 |
| 等价于 hiphip.array 操作 | `amake` | 使用表达式计算出的值创建一个新的数组并填充 |
|   | `areduce` | 类似于 `clojure.core/areduce`，带有 HipHip 数组绑定 |
|   | `doarr` | 类似于 `clojure.core/doseq`，带有 HipHip 数组绑定 |
|   | `amap` | 类似于 `clojure.core/for`，创建一个新的数组 |
|   | `afill!` | 类似于前面的 `amap`，但覆盖数组参数 |
| 数学运算 | `asum` | 使用表达式计算数组元素的总和 |
|   | `aproduct` | 使用表达式计算数组元素乘积 |
|   | `amean` | 计算数组元素的平均值 |
|   | `dot-product` | 计算两个数组的点积 |
| 查找最小/最大值、排序 | `amax-index` | 在数组中找到最大值并返回其索引 |
|   | `amax` | 在数组中找到最大值并返回它 |
|   | `amin-index` | 在数组中找到最小值并返回其索引 |
|   | `amin` | 在数组中找到最小值并返回它 |
|   | `apartition!` | 数组的三角划分：小于、等于、大于枢轴 |
|   | `aselect!` | 将最小的`k`个元素聚集到数组的开头 |
|   | `asort!` | 使用 Java 的内置实现原地排序数组 |
|   | `asort-max!` | 部分原地排序，将前`k`个元素聚集到数组末尾 |
|   | `asort-min!` | 部分原地排序，将最小的`k`个元素聚集到数组顶部 |
|   | `apartition-indices!` | 与`apartition!`类似，但修改的是索引数组而不是值 |
|   | `aselect-indices!` | 与`aselect!`类似，但修改的是索引数组而不是值 |
|   | `asort-indices!` | 与`asort!`类似，但修改的是索引数组而不是值 |
|   | `amax-indices` | 获取索引数组；最后`k`个索引指向最大的`k`个值 |
|   | `amin-indices` | 获取索引数组；前`k`个索引指向最小的`k`个值 |

要将 HipHip 作为依赖项包含在你的 Leiningen 项目中，请在`project.clj`中指定它：

```java
:dependencies [;; other dependencies
               [prismatic/hiphip "0.2.0"]]
```

作为使用 HipHip 的一个示例，让我们看看如何计算数组的归一化值：

```java
(require '[hiphip.double :as hd])

(def xs (double-array [12.3 23.4 34.5 45.6 56.7 67.8]))

(let [s (hd/asum xs)] (hd/amap [x xs] (/ x s)))
```

除非我们确保`xs`是一个原始双精度浮点数数组，否则 HipHip 在类型不正确时会抛出`ClassCastException`，在其他情况下会抛出`IllegalArgumentException`。我建议探索 HipHip 项目，以获得更多关于如何有效使用它的见解。

## primitive-math

我们可以将`*warn-on-reflection*`设置为 true，让 Clojure 在调用边界处使用反射时警告我们。然而，当 Clojure 必须隐式使用反射来执行数学运算时，唯一的解决办法是使用分析器或将 Clojure 源代码编译成字节码，并使用反编译器分析装箱和反射。这就是`primitive-math`库发挥作用的地方，它通过产生额外的警告并抛出异常来帮助。

`primitive-math`库可在[`github.com/ztellman/primitive-math`](https://github.com/ztellman/primitive-math)找到。

截至撰写本文时，primitive-math 版本为 0.1.4；你可以通过编辑`project.clj`将其作为依赖项包含在你的 Leiningen 项目中，如下所示：

```java
:dependencies [;; other dependencies
               [primitive-math "0.1.4"]]
```

以下代码展示了如何使用它（回想一下*将.class 文件反编译成 Java 源代码*部分的示例）：

```java
;; must enable reflection warnings for extra warnings from primitive-math
(set! *warn-on-reflection* true)
(require '[primitive-math :as pm])
(defn mul [x y] (pm/* x y))  ; primitive-math produces reflection warning
(mul 10.3 2)                        ; throws exception
(defn mul [^long x ^long y] (pm/* x y))  ; no warning after type hinting
(mul 10.3 2)  ; returns 20
```

虽然`primitive-math`是一个有用的库，但它解决的问题大部分已经被 Clojure 1.7 中的装箱检测功能所处理（参见下一节*检测装箱数学*）。然而，如果你无法使用 Clojure 1.7 或更高版本，这个库仍然很有用。

### 检测装箱数学

**装箱数学**难以检测，是性能问题的来源。Clojure 1.7 引入了一种在发生装箱数学时警告用户的方法。这可以通过以下方式配置：

```java
(set! *unchecked-math* :warn-on-boxed)

(defn sum-till [n] (/ (* n (inc n)) 2))  ; causes warning
Boxed math warning, /private/var/folders/cv/myzdv_vd675g4l7y92jx9bm5lflvxq/T/form-init3701519533014890866.clj:1:28 - call: public static java.lang.Number clojure.lang.Numbers.unchecked_inc(java.lang.Object).
Boxed math warning, /private/var/folders/cv/myzdv_vd675g4l7y92jx9bm5lflvxq/T/form-init3701519533014890866.clj:1:23 - call: public static java.lang.Number clojure.lang.Numbers.unchecked_multiply(java.lang.Object,java.lang.Object).
Boxed math warning, /private/var/folders/cv/myzdv_vd675g4l7y92jx9bm5lflvxq/T/form-init3701519533014890866.clj:1:20 - call: public static java.lang.Number clojure.lang.Numbers.divide(java.lang.Object,long).

;; now we define again with type hint
(defn sum-till [^long n] (/ (* n (inc n)) 2))
```

当使用 Leiningen 时，你可以在 `project.clj` 文件中添加以下条目来启用装箱数学警告：

```java
:global-vars {*unchecked-math* :warn-on-boxed}
```

`primitive-math`（如 HipHip）中的数学运算是通过宏实现的。因此，它们不能用作高阶函数，并且因此可能与其他代码组合不佳。我建议探索该项目，看看什么适合你的程序用例。采用 Clojure 1.7 通过 boxed-warning 功能消除了装箱发现问题。

# 退回到 Java 和原生代码

在一些情况下，由于 Clojure 中缺乏命令式、基于栈的、可变变量，可能会导致代码的性能不如 Java，我们可能需要评估替代方案以提高性能。我建议你考虑直接用 Java 编写此类代码以获得更好的性能。

另一个考虑因素是使用原生操作系统功能，例如内存映射缓冲区 ([`docs.oracle.com/javase/7/docs/api/java/nio/MappedByteBuffer.html`](http://docs.oracle.com/javase/7/docs/api/java/nio/MappedByteBuffer.html)) 或文件和不受保护的操作 ([`highlyscalable.wordpress.com/2012/02/02/direct-memory-access-in-java/`](http://highlyscalable.wordpress.com/2012/02/02/direct-memory-access-in-java/))。请注意，不受保护的操作可能具有潜在风险，通常不建议使用。这些时刻也是考虑在 C 或 C++ 中编写性能关键代码，然后通过 **Java Native Interface**（**JNI**）访问它们的良机。

## Proteus – Clojure 中的可变局部变量

Proteus 是一个开源的 Clojure 库，它允许你将局部变量视为局部变量，从而只允许在局部作用域内进行非同步修改。请注意，这个库依赖于 Clojure 1.5.1 的内部实现结构。**Proteus** 项目可在 [`github.com/ztellman/proteus`](https://github.com/ztellman/proteus) 找到。

你可以通过编辑 `project.clj` 将 Proteus 作为依赖项包含在 Leiningen 项目中：

```java
:dependencies [;;other dependencies
               [proteus "0.1.4"]]
```

在代码中使用 Proteus 非常简单，如下代码片段所示：

```java
(require '[proteus :as p])
(p/let-mutable [a 10]
  (println a)
  (set! a 20)
  (println a))
;; Output below:
;; 10
;; 20
```

由于 Proteus 只允许在局部作用域中进行修改，以下代码会抛出异常：

```java
(p/let-mutable [a 10 add2! (fn [x] (set! x (+ 2 x)))]
  (add2! a)
  (println a))
```

可变局部变量非常快，在紧密循环中可能非常有用。Proteus 在 Clojure 习惯用法上是非传统的，但它可能在不编写 Java 代码的情况下提供所需的性能提升。

# 摘要

由于 Clojure 强大的 Java 互操作性和底层支持，程序员可以利用接近 Java 的性能优势。对于性能关键代码，有时有必要了解 Clojure 如何与 Java 交互以及如何调整正确的旋钮。数值是一个需要 Java 互操作性以获得最佳性能的关键领域。类型提示是另一个重要的性能技巧，通常非常有用。有几个开源的 Clojure 库使程序员更容易进行此类活动。

在下一章中，我们将深入挖掘 Java 之下，看看硬件和 JVM 堆栈如何在提供我们所获得性能方面发挥关键作用，它们的限制是什么，以及如何利用对这些理解的应用来获得更好的性能。
