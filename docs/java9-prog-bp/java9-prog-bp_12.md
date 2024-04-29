# 接下来是什么？

最后，我们终于来到了我们的最后一章。我们构建了许多不同类型的应用程序，试图突出和展示 Java 平台的不同部分，特别是 Java 9 中的新部分。正如我们所讨论的，仅使用 Java 9 中的新技术和 API 编写是不可能的，所以我们还看到了一些有趣的 Java 7 和 8 中的项目。随着 Java 9 终于发布，我们有必要展望 Java 的未来可能会为我们带来什么，但也明智地环顾四周，看看其他语言提供了什么，以便我们可以决定我们的下一个 Java 实际上是否会是 Java。在本章中，我们将做到这一点。

在本章中，我们将涵盖以下主题：

+   回顾我们之前涵盖的主题

+   未来可以期待的内容

# 回顾过去

展望 Java 10 及更高版本之前，让我们快速回顾一下本书中涵盖的一些内容：

+   Java 平台模块系统可能是这个版本中最大、最受期待的新增功能。我们看到了如何创建一个模块，并讨论了它对运行时系统的影响。

+   我们走过了 Java 9 中的新进程管理 API，并学习了如何查看进程，甚至在需要时终止它们。

+   我们看了一些 Java 8 中引入的主要功能接口，讨论了它们的用途，并展示了这些接口支持的 lambda 表达式和不支持的代码可能是什么样子。

+   我们详细讨论了 Java 8 的`Optional<T>`，展示了如何创建该类的实例，它暴露的各种方法，以及如何使用它。

+   我们花了大量时间构建基于 JavaFX 的应用程序，展示了各种技巧，解决了一些问题，等等。

+   使用 Java NIO 文件和路径 API，我们遍历文件系统，寻找重复文件。

+   我们使用 Java 持久性 API 实现了数据持久性，演示了如何在 Java SE 环境中使用 API，如何定义实体等。

+   我们使用 Java 8 的日期/时间 API 构建了一个计算器，将功能暴露为库和命令行实用程序。

+   作为这一努力的一部分，我们简要比较了一些命令行实用程序框架（特别关注 Crest 和 Airline），然后选择了 Crest，并演示了如何创建和使用命令行选项。

+   虽然我们并没有在每一章中都专注于它，但我们确实休息一下，讨论并演示了单元测试。

+   我们了解了**服务提供者接口**（**SPIs**）作为一种提供多个替代实现的接口的手段，可以在运行时动态加载。

+   我们实现了一些 REST 服务，不仅演示了 JAX-RS 的基本功能，如何在 Java SE 环境中部署它和 POJO 映射，还包括一些更高级的功能，包括服务器发送事件和使用`Filter`保护端点。

+   我们构建了一些 Android 应用程序，并讨论和演示了活动、片段、服务、内容提供程序、异步消息传递和后台任务。

+   我们看到了 OAuth2 认证流程的实际操作，包括如何使用 Google OAuth 提供程序设置凭据以及驱动该过程所需的 Java 代码。

+   我们发现了 JSON Web Tokens，这是一种在客户端和服务器之间安全传递数据的加密方式，并看到了它们作为认证系统的基本使用。

+   我们了解了 JavaMail API，学习了一些常见电子邮件协议的历史和工作原理，比如 POP3 和 SMTP。

+   我们学习了使用 Quartz 调度程序库进行作业调度。

+   我们看到了如何以声明方式指定数据的约束，然后如何使用 Bean Validation API 在这些约束的光线下验证数据。

+   彻底改变方向，我们使用功能丰富的 NetBeans Rich Client Platform 构建了一个相当复杂的应用程序。

+   我们简要地看了一下使用 MongoDB 的全球文档数据库。

+   我们还学习了依赖注入以及如何在 CDI 规范中使用它。

这是一个相当长的列表，还没有涵盖所有内容。本书的一个声明目的是讨论和演示 Java 9 的新特性。随着发布，将有近 100 个**Java 增强提案**（**JEPs**），其中一些很难，甚至无法演示，但我们已经尽力了。

# 展望未来

那么，Java 9 完成后，自然的问题是，**接下来是什么？**正如你所期望的，甲骨文、红帽、IBM、Azul Systems 等公司的工程师们在 Java 9 规划和开发期间就一直在思考这个问题。虽然几乎不可能确定 Java 10 会包含什么（记住，需要三个主要版本才能完成模块系统），但我们目前正在讨论和设计一些项目，希望能在下一个版本中发布。在接下来的几页中，我们将探讨其中一些，提前了解一下未来几年作为 Java 开发人员的生活可能会是什么样子。

# 瓦哈拉项目

瓦哈拉项目是一个高级语言-虚拟机共同开发项目的孵化基地。它由甲骨文工程师 Brian Goetz 领导。截至目前，瓦哈拉计划有三个特性。它们是值类型、泛型特化和具体化泛型。

# 值类型

这一努力的目标是更新 Java 虚拟机，如果可能的话，还有 Java 语言，以支持小型、不可变、无标识的值类型。目前，如果你实例化一个新的`Object`，JVM 会为其分配一个标识符，这允许对**变量**实例进行引用。

例如，如果你创建一个新的整数，`new Integer(42)`，一个带有`java.lang.Integer@68f29546`标识的变量，但值为`42`，这个变量的值永远不会改变，这通常是我们作为开发人员关心的。然而，JVM 并不真正知道这一点，所以它必须维护变量的标识，带来了所有的开销。根据 Goetz 的说法，这意味着这个对象的每个实例将需要多达 24 个额外字节来存储实例。例如，如果你有一个大数组，那么这可能是一个相当大的内存管理量，最终需要进行垃圾回收。

然后，JVM 工程师们希望实现的是一种温和地扩展 Java 虚拟机字节码和 Java 语言本身的方式，以支持一个小型、不可变的聚合类型（想象一个具有 0 个或更多属性的类），它缺乏标识，这将带来“内存和局部性高效的编程习惯，而不会牺牲封装”。他们希望 Java 开发人员能够创建这些新类型并将它们视为另一种原始类型。如果他们做得正确，Goetz 说，这个特性可以总结为**像类一样编码，像 int 一样工作！**

截至 2017 年 4 月（[`cr.openjdk.java.net/~jrose/values/shady-values.html`](http://cr.openjdk.java.net/~jrose/values/shady-values.html)），当前的提案提供了以下代码片段作为如何定义值类型的示例：

```java
    @jvm.internal.value.DeriveValueType 
    public final class DoubleComplex { 
      public final double re, im; 
      private DoubleComplex(double re, double im) { 
        this.re = re; this.im = im; 
      } 
      ... // toString/equals/hashCode, accessors,
       math functions, etc. 
    } 

```

当实例化时，这种类型的实例可以在堆栈上创建，而不是在堆上，并且使用的内存要少得多。这是一个非常低级和技术性的讨论，远远超出了本书的范围，但如果你对更多细节感兴趣，我建议阅读之前链接的页面，或者在[`cr.openjdk.java.net/~jrose/values/values-0.html`](http://cr.openjdk.java.net/~jrose/values/values-0.html)上阅读该努力的初始公告。

# 泛型特化

泛型特化可能更容易理解一些。目前，泛型类型变量只能持有引用类型。例如，你可以创建一个`List<Integer>`，但不能创建一个`List<int>`。为什么会这样有一些相当复杂的原因，但能够使用原始类型和值类型将使集合在内存和计算方面更有效率。你可以在这篇文章中了解更多关于这个特性的信息，再次是 Brian Goetz 的文章--[`cr.openjdk.java.net/~briangoetz/valhalla/specialization.html`](http://cr.openjdk.java.net/~briangoetz/valhalla/specialization.html)。Jesper de Jong 在这里也有一篇关于泛型类型变量中原始类型复杂性的很好的文章：

[`www.jesperdj.com/2015/10/12/project-valhalla-generic-specialization/`](http://www.jesperdj.com/2015/10/12/project-valhalla-generic-specialization/)

# 具体化的泛型

泛型化的具体化是一个经常引起非常响亮、生动反应的术语。目前，如果你声明一个变量的类型是`List<Integer>`，生成的字节码实际上并不知道参数化类型，因此在运行时无法发现。如果你在运行时检查变量，你将看不到`Integer`的提及。当然，你可以查看每个元素的类型，但即使这样，你也不能确定`List`的类型，因为没有强制要求**只有**`Integer`可以添加到`List`中。

自从 Java 5 引入泛型以来，Java 开发人员一直在呼吁具体化的泛型，或者简单地说，保留泛型在运行时的类型信息。你可能会猜到，使 Java 的泛型具体化并不是一项微不足道的任务，但最终，我们有了一个正式的努力来看看是否可以做到，如果可以做到，是否可以找到一种向后兼容的方式，不会有负面的性能特征，例如。

# Panama 项目

尽管尚未针对任何特定的 Java 版本，Panama 项目为那些使用或希望使用第三方本地库的人提供了一些希望。目前，将本地库（即，针对操作系统的库，比如 C 或 C++编写的库）暴露给 JVM 的主要方式是通过**Java 本地接口**（**JNI**）。JNI 的问题之一，或者至少是其中之一，是它要求每个想要将本地库暴露给 JVM 的 Java 程序员也成为 C 程序员，这意味着不仅要了解 C 语言本身，还要了解每个支持的平台的相关构建工具。

Panama 项目希望通过提供一种新的方式来暴露本地库，而无需深入了解库语言的生态系统或 JVM，来改善这个问题。Panama 项目的 JEP（[`openjdk.java.net/jeps/191`](http://openjdk.java.net/jeps/191)）列出了这些设计目标。

+   描述本地库调用的元数据系统（调用协议、参数列表结构、参数类型、返回类型）和本地内存结构（大小、布局、类型、生命周期）。

+   发现和加载本地库的机制。这些功能可能由当前的`System.loadLibrary`提供，也可能包括额外的增强功能，用于定位适合主机系统的平台或特定版本的二进制文件。

+   基于元数据的机制，将给定库/函数坐标绑定到 Java 端点，可能通过由管道支持的用户定义接口。

+   基于元数据的机制，将特定的内存结构（布局、字节序、逻辑类型）绑定到 Java 端点，无论是通过用户定义的接口还是用户定义的类，在这两种情况下都由管道支持来管理真实的本地内存块。

+   适当的支持代码，将 Java 数据类型转换为本地数据类型，反之亦然。在某些情况下，这将需要创建特定于 FFI 的类型，以支持 Java 无法表示的位宽和数值符号。

JNI 已经可用了相当长一段时间，现在终于得到了一些早就该得到的关注。

# Project Amber

Project Amber 的目标是**探索和孵化更小、以提高生产力为导向的 Java 语言特性**。目前的列表包括局部变量类型推断、增强枚举和 lambda 遗留问题。

# 局部变量类型推断

正如我们在本书中多次看到的那样，在 Java 中声明变量时，您必须在左侧和右侧各声明一次类型，再加上一个变量名：

```java
    AtomicInteger atomicInt = new AtomicInteger(42); 

```

问题在于这段代码冗长而重复。局部变量类型推断工作希望解决这个问题，使得像这样的东西成为可能：

```java
    var atomicInt = new AtomicInteger(42); 

```

这段代码更加简洁，更易读。请注意`val`关键字的添加。通常，编译器知道代码行是变量声明时，例如`<type> <name> = ...`。由于这项工作将消除声明左侧的类型的需要，我们需要一个提示编译器的线索，这就是这个 JEP 的作者提出的`var`。

还有一些关于简化不可变或`final`变量声明的讨论。提议中包括`final var`以及`val`，就像其他语言（如 Scala）中所见的那样。在撰写本文时，尚未就哪项提议最终确定做出决定。

# 增强枚举

增强枚举将通过允许枚举中的类型变量（通用枚举）和对枚举常量进行更严格的类型检查来增强 Java 语言中枚举结构的表现力。([`openjdk.java.net/jeps/301`](http://openjdk.java.net/jeps/301))。这意味着枚举最终将支持参数化类型，允许像这样的东西（取自先前提到的 JEP 链接）：

```java
    enum Primitive<X> { 
      INT<Integer>(Integer.class, 0) { 
        int mod(int x, int y) { return x % y; } 
        int add(int x, int y) { return x + y; } 
      }, 
      FLOAT<Float>(Float.class, 0f)  { 
        long add(long x, long y) { return x + y; } 
      }, ... ; 

      final Class<X> boxClass; 
      final X defaultValue; 

      Primitive(Class<X> boxClass, X defaultValue) { 
        this.boxClass = boxClass; 
        this.defaultValue = defaultValue; 
      } 
    } 

```

请注意，除了为每个`enum`值指定通用类型之外，我们还可以为每个`enum`类型定义特定于类型的方法。这将大大简化定义一组预定义常量，同时也可以为每个常量定义类型安全和类型感知的方法。

# Lambda 遗留问题

目前有两个项目被标记为 Java 8 中 lambda 工作的`leftovers`。第一个是在 lambda 声明中使用下划线表示未使用的参数。例如，在这个非常牵强的例子中，我们只关心`Map`的值：

```java
    Map<String, Integer> numbers = new HashMap<>(); 
    numbers.forEach((k, v) -> System.out.println(v*2)); 

```

这在 IDE 中会产生这样的结果：

![](img/4e5a6dac-2f9b-4a04-a5b4-cdf4ba81addb.png)

一旦允许使用下划线，这段代码将变成这样：

```java
    numbers.forEach((_, v) -> System.out.println(v*2)); 

```

这允许更好地静态检查未使用的变量，使工具（和开发人员）更容易识别这些参数并进行更正或标记。

另一个遗留问题是允许 lambda 参数遮蔽来自封闭范围的变量。如果您现在尝试这样做，您将得到与尝试在语句块内重新定义变量相同的错误--**变量已经定义**：

```java
    Map<String, Integer> numbers = new HashMap<>(); 
    String key = someMethod(); 
    numbers.forEach((key, value) ->  
      System.out.println(value*2)); // error 

```

有了这个改变，前面的代码将编译并正常运行。

# 四处张望

多年来，JVM 已经支持了替代语言。其中一些较为知名的包括 Groovy 和 Scala。这两种语言多年来以某种方式影响了 Java，但是像任何语言一样，它们也不是没有问题。许多人认为 Groovy 的性能不如 Java（尽管`invokedynamic`字节码指令应该已经解决了这个问题），许多人发现 Groovy 更动态的特性不太吸引人。另一方面，Scala（公平与否取决于你问谁）受到了太过复杂的认知。编译时间也是一个常见的抱怨。此外，许多组织都很乐意同时使用这两种语言，因此绝对值得考虑它们是否适合您的环境和需求。

虽然这些可能是很棒的语言，但我们在这里花点时间来看看接下来会发生什么，至少有两种语言似乎脱颖而出——锡兰语和 Kotlin。我们无法对这些语言进行详尽的介绍，但在接下来的几页中，我们将快速浏览这些语言，看看它们现在为 JVM 开发人员提供了什么，也许还能看到它们如何影响未来对 Java 语言的改变。

# 锡兰

锡兰语是由红帽赞助的一种语言，最早出现在 2011 年左右。由 Hibernate 和 Seam Framework 的知名人物 Gavin King 领导，团队着手解决他们多年来在开发自己的框架和库时所经历的一些痛点，从语言和库的层面上解决这些问题。他们承认自己是 Java 语言的忠实粉丝，但也承认这种语言并不完美，特别是在一些标准库方面，他们希望在锡兰语中修复这些缺陷。该语言的目标包括可读性、可预测性、可工具化、模块化和元编程能力（https://ceylon-lang.org/blog/2012/01/10/goals）。

在开始使用锡兰时，您可能会注意到的最大的区别之一是模块的概念已经融入到了语言中。在许多方面，它看起来与 Java 9 的模块声明非常相似，如下所示：

```java
    module com.example.foo "1.0" { 
      import com.example.bar "2.1"; 
    } 

```

然而，有一个非常明显的区别——锡兰模块确实有版本信息，这允许各种模块依赖于系统中可能已经存在的模块的不同版本。

锡兰和 Java 之间至少还有一个相当重要的区别——锡兰内置了构建工具。例如，虽然有 Maven 插件，但首选方法是使用锡兰的本机工具来构建和运行项目：

```java
$ ceylonb new hello-world 
Enter project folder name [helloworld]: ceylon-helloworld 
Enter module name [com.example.helloworld]: 
Enter module version [1.0.0]: 
Would you like to generate Eclipse project files? (y/n) [y]: n 
Would you like to generate an ant build.xml? (y/n) [y]: n 
$ cd ceylon-helloworld 
$ ceylonb compile 
Note: Created module com.example.helloworld/1.0.0 
$ ceylonb run com.example.helloworld/1.0.0 
Hello, World! 

```

除了模块系统，锡兰可能为 Java 开发人员提供什么？其中一个立即有用和实用的功能是改进的空值处理支持。就像在 Java 中一样，我们仍然必须在锡兰中检查空值，但该语言提供了一种更好的方法，一切都始于类型系统。

关于 Scala 的一个抱怨（无论是否真正有根据）是其类型系统过于复杂。不管你是否同意，似乎很明显，相对于 Java 提供的内容，肯定有改进的空间（即使 Java 语言的设计者们也同意，例如提出的局部变量类型推断提案）。锡兰为类型系统提供了一个非常强大的补充——联合类型和交集类型。

联合类型允许变量具有多种类型，但一次只能有一种。在讨论空值时，这就体现在`String? foo = ...`，它声明了一个可为空的`String`类型的变量，实际上与`String|Null foo = ...`是相同的。

这声明了一个名为 foo 的变量，其类型可以是`String`或`Null`，但不能同时是两者。`?`语法只是对联合类型声明（`A | B`或`A`或`B`）的一种语法糖。如果我们有一个方法，那么它接受这种联合类型；我们知道该变量是可空的，所以我们需要使用以下代码片段进行检查：

```java
    void bar (String? Foo) { 
      if (exists foo) { 
        print (foo); 
      } 
    } 

```

由于这是一个联合类型，我们也可以这样做：

```java
    void bar (String? Foo) { 
      if (is String foo) { 
        print (foo); 
      } 
    } 

```

请注意，一旦我们使用`exists`或`is`进行测试，我们可以假定该变量不为空且为`String`。编译器不会抱怨，我们也不会在运行时遇到意外的`NullPointerException`（它们实际上在锡兰中不存在，因为编译器要求您对可空变量的处理非常明确）。这种对空值和类型检查的编译器感知称为**流敏感**类型。一旦您验证了某个东西的类型，编译器就知道并记住了这个检查的结果，这样说来，对于该范围的剩余部分，您可以编写更清晰、更简洁的代码。

联合类型要么是 A 要么是 B，而交集类型是 A**和**B。举个完全随意的例子，假设您有一个方法，其参数必须是`Serializable`**和**`Closeable`。在 Java 中，您必须手动检查，编写以下代码：

```java
    public void someMethod (Object object) { 
      if (!(object instanceof Serializable) ||  
        !(object instanceof Closeable)) { 
        // throw Exception 
      } 
    } 

```

有了交集类型，Ceylon 可以让我们编写这样的代码：

```java
    void someMethod(Serializable&Closeable object) { 
      // ... 
    } 

```

如果我们尝试使用未实现**两个**接口的内容调用该方法，或者说，扩展一个类并实现其他接口，那么我们将在**编译时**出现错误。这非常强大。

在企业采用新语言或库之前，人们经常会查看谁还在使用它。是否有显著的采用案例？是否有其他公司足够自信地使用该技术构建生产系统？不幸的是，Ceylon 网站（撰写时）在 Red Hat 之外的采用细节上非常匮乏，因此很难回答这个问题。但是，Red Hat 正在花费大量资金设计语言并构建围绕它的工具和社区，因此这应该是一个安全的选择。当然，这是您的企业在经过慎重考虑后必须做出的决定。您可以在[`ceylon-lang.org`](https://ceylon-lang.org)了解更多关于 Ceylon 的信息。

# Kotlin

另一个新兴的语言是 Kotlin。这是来自 JetBrains 的静态类型语言，他们是 IntelliJ IDEA 的制造商，旨在同时针对 JVM 和 Javascript。它甚至具有初步支持，可以通过 LLVM 直接编译为机器代码，用于那些不希望或不允许使用虚拟机的环境，例如 iOS、嵌入式系统等。

Kotlin 于 2010 年开始，并于 2012 年开源，旨在解决 JetBrains 在大规模 Java 开发中面临的一些常见问题。在调查了当时的语言格局后，他们的工程师们认为这些语言都没有充分解决他们的问题。例如，被许多人认为是“下一个 Java”的 Scala，尽管具有可接受的功能集，但编译速度太慢，因此 JetBrains 开始设计他们自己的语言，并于 2016 年 2 月发布了 1.0 版本。

Kotlin 团队的设计目标包括表达性、可扩展性和互操作性。他们的目标是允许开发人员通过语言和库功能以更清晰的方式编写更少的代码，以及使用与 Java 完全互操作的语言。他们添加了诸如协程之类的功能，以使基于 Kotlin 的系统能够快速轻松地扩展。

说了这么多，Kotlin 是什么样的，为什么我们作为 Java 开发人员应该感兴趣呢？让我们从变量开始。

正如您所记得的，Java 既有原始类型（`int`、`double`、`float`、`char`等），也有引用或包装类型（`Integer`、`Double`、`Float`、`String`等）。正如我们在本章中讨论的那样，JVM 工程师正在努力解决这种二分法带来的一些行为和能力差异。Kotlin 完全避免了这一点，因为每个值都是一个对象，所以不必担心`List<int>`与`List<Integer>`之间的区别。

此外，Kotlin 已经支持本地变量类型推断以及不可变性。例如，考虑以下 Java 代码作为示例：

```java
    Integer a = new Integer(1); 
    final String s = "This is a string literal"; 

```

Kotlin

```java
    var a = 1; 
    val s = "This is a string literal"; 

```

请注意`var`和`val`关键字的使用。正如我们之前讨论过的关于未来 Java 语言更改，这些关键字允许我们声明可变和不可变变量（分别）。还要注意，我们不需要声明变量的类型，因为编译器会为我们处理。在某些情况下，我们可能需要明确声明类型，例如在编译器可能猜测错误或者没有足够信息进行猜测的情况下，此时，它将停止编译并显示错误消息。在这些情况下，我们可以这样声明类型：

```java
    var a: Int  = 1; 
    val s: String = "This is a string literal"; 

```

正如我们所见，Java 8 中有`Optional<T>`来帮助处理空值。Kotlin 也有空值支持，但它内置在语言中。默认情况下，Kotlin 中的所有变量都**不**可为空。也就是说，如果编译器能够确定您试图将空值赋给变量，或者无法确定值是否可能为空（例如来自 Java API 的返回值），则会收到编译器错误。要指示值可以为空，您需要在变量声明中添加`?`，如下所示：

```java
    var var1 : String = null; // error 
    var var2 : String? = null; // ok 

```

Kotlin 还在方法调用中提供了改进的空值处理支持。例如，假设您想要获取用户的城市。在 Java 中，您可能会这样做：

```java
    String city = null; 
    User user = getUser(); 
    if (user != null) { 
      Address address = user.getAddress(); 
      if (address != null) { 
        city address.getCity(); 
      } 
    } 

```

在 Kotlin 中，可以用一行代码来表达如下：

```java
    var city : String? = getUser()?.getAddress()?.getCity(); 

```

如果在任何时候，其中一个方法返回 null，方法调用链就会结束，并且 null 会被赋给变量 city。Kotlin 在处理空值方面并不止于此。例如，它提供了`let`函数，可以作为 if-not-null 检查的快捷方式。例如，考虑以下代码行：

```java
    if (city != null) { 
      System.out.println(city.toUpperCase()); 
    } 

```

在 Kotlin 中，前面的代码行变成了这样：

```java
    city?.let { 
      println(city.toUpperCase()) 
    } 

```

当然，这可以写成`city?.toUpperCase()`。然而，这应该证明的是在任意大的复杂代码块中安全使用可空变量的能力。值得注意的是，在`let`块内，编译器知道`city`不为空，因此不需要进一步的空值检查。

也许在前面的例子中隐藏着 Kotlin 对 lambda 的支持，没有 lambda 的话，似乎没有现代语言值得考虑。Kotlin 确实完全支持 lambda、高阶函数、下划线作为 lambda 参数名称等。它的支持和语法非常类似于 Java，因此 Java 开发人员应该对 Kotlin 的 lambda 非常熟悉。

当然，一个重要的问题是，**Kotlin 准备好投入使用了吗？** JetBrains 绝对认为是这样，因为他们在许多内部和外部应用程序中都在使用它。其他知名用户包括 Pinterest、Gradle、Evernote、Uber、Pivotal、Atlassian 和 Basecamp。Kotlin 甚至得到了 Google 的官方支持（在 Android Studio 中）用于 Android 开发，因此它绝对是一个生产级的语言。

当然，这个伟大的新语言还有很多，空间不允许我们讨论所有，但您可以浏览[`kotlinlang.org`](https://kotlinlang.org)了解更多信息，看看 Kotlin 是否适合您的组织。

# 总结

当然，关于 Java 10 和这两种语言，以及围绕 Java 虚拟机发生的众多其他项目，还有很多可以讨论的地方。经过 20 多年的发展，Java——语言和环境——仍然非常强大。在本书的页面中，我试图展示语言中的一些重大进展，为您自己的项目提供各种起点、可供学习和重用的示例代码，以及各种库、API 和技术的解释，这些可能对您的日常工作有所帮助。我希望您喜欢这些示例和解释，就像我喜欢准备它们一样，更重要的是，我希望它们能帮助您构建下一个大事件。

祝你好运！
