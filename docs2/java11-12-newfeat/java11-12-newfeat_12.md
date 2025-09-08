# JDK 11 中的其他改进

Java 11 包含了许多我们无法在各个章节中单独涵盖的有趣变化。然而，这并不意味着它们不重要或不重要到足以忽略，而是因为它们的细节超出了本书的范围。例如，基于嵌套的访问控制包括对 **Java 虚拟机**（**JVM**）规范的修改，动态类文件常量扩展了现有的类文件常量，加密层和 **传输层安全性**（**TLS**）的改进，以及更多。

本章包括对与 SE、JDK 和 Java 11 功能实现相关的剩余 JDK 11 功能的概述。

在本章中，我们将涵盖以下主题：

+   基于嵌套的访问控制

+   动态类文件常量

+   改进 AArch64 内联函数

+   移除 Java EE 和 CORBA 模块

+   基于 Curve25519 和 Curve448 的密钥协商

+   Unicode 10

+   ChaCha20 和 Poly1305 密码学算法

+   启动单个文件源代码程序

+   TLS 1.3

+   废弃 Nashorn JavaScript 引擎

+   废弃 pack200 工具和 API

# 技术要求

要使用本章中包含的代码，您需要在您的系统上安装 JDK 11 或更高版本。

由于本章涵盖了 Java 11 的多个功能，让我们快速将功能与其 **JDK 增强提案**（**JEP**）编号和范围进行匹配。

# 列出本章中使用的 JEP

下表列出了本章中涵盖的 JDK 11 功能，它们的对应 JEP 编号和范围：

| **JEP** | **范围** | **描述** |
| --- | --- | --- |
| 181 | SE | 基于嵌套的访问控制 |
| 309 | SE | 动态类文件常量 |
| 315 | 实现 | 改进 AArch64 内联函数 |
| 320 | SE | 移除 Java EE 和 CORBA 模块 |
| 324 | SE | 基于 Curve25519 和 Curve448 的密钥协商 |
| 327 | SE | Unicode 10 |
| 329 | SE | ChaCha20 和 Poly1305 密码学算法 |
| 330 | JDK | 启动单个文件源代码程序 |
| 332 | SE | TLS 1.3 |
| 335 | JDK | 废弃 Nashorn JavaScript 引擎 |
| 336 | SE | 废弃 pack200 工具和 API |

让我们从第一个功能开始。

# 基于嵌套的访问控制

想象一下当你定义嵌套类或接口时会发生什么？例如，如果你定义一个 *两层类*，比如说 `Outer`，和一个 *内部类*，比如说 `Inner`，`Inner` 能否访问 `Outer` 的 `private` 实例变量？以下是一些示例代码：

```java
public class Outer { 
    private int outerInt = 20; 
    public class Inner { 
        int innerInt = outerInt;     // Can Inner access outerInt? 
    } 
} 
```

是的，它可以。由于您在同一个源代码文件中定义了这些类，您可能会认为这是显而易见的。然而，事实并非如此。编译器为 `Outer` 和 `Inner` 类生成单独的字节码文件（`.class`）。对于前面的示例，编译器创建了两个字节码文件：`Outer.class` 和 `Outer$Inner.class`。为了您的快速参考，内部类的字节码文件以外部类的名称和一个美元符号开头。

为了使这些类之间的变量访问成为可能，并保留程序员的期望，编译器将私有成员的访问范围扩展到包中，或者在每一个这些类中添加桥接变量或方法。以下是使用**JAD Java Decompiler**反编译的`Outer`和`Inner`类的反编译版本：

```java
// Decompiled class Outer 
public class Outer { 
    public class Inner { 
        int innerInt; 
        final Outer this$0; 

        public Inner() { 
            this$0 = Outer.this; 
            super(); 
            innerInt = outerInt; 
        } 
    } 
    public Outer() { 
        outerInt = 20; 
    } 
    private int outerInt; 
} 
```

以下是`Outer$Inner`反编译类的如下所示：

```java
public class Outer$Inner { 
    int innerInt; 
    final Outer this$0; 

    public Outer$Inner() { 
        this$0 = Outer.this; 
        super(); 
        innerInt = outerInt; 
    } 
}
```

正如您将注意到的，反编译版本验证了编译器定义了一个桥接变量，即在`Inner`类中的`Outer`类型的`this$0`，以便从`Inner`访问`Outer`的成员。

这些桥接变量和方法可能会破坏封装性，并增加部署应用程序的大小。它们也可能使开发人员和工具感到困惑。

# 什么是基于巢的访问？

JEP 181 引入了基于巢的访问控制。正如您所知，定义在另一个类或接口内部的类和接口被编译为单独的类文件。为了访问对方的私有成员，编译器要么扩展它们的访问级别，要么插入桥接方法。

基于巢的访问控制允许这样的类和接口相互访问对方的私有成员，而不需要编译器进行任何工作（或桥接代码）。

基于巢的访问控制还导致 JVM 规范的变化。您可以参考以下链接来访问这些变化（删除的内容以红色字体背景突出显示，添加的内容以绿色背景突出显示）：[`cr.openjdk.java.net/~dlsmith/nestmates.html`](https://cr.openjdk.java.net/~dlsmith/nestmates.html)。

# 基于巢的控制的 影响

虽然它可能看起来很简单，但基于巢的访问控制会影响所有涉及访问控制或方法调用的规范和 API——无论是隐式还是显式。如*什么是基于巢的访问？*部分所述，它包括 JVM 规范的变化。它还影响类文件属性、访问控制规则、字节码调用规则、反射、方法调用和字段访问规则。

它还添加了新的类文件属性，修改了`MethodHandle`查找规则，导致类转换/重新定义——JVM TI 和`java.lang.instrument` API、JDWP 和 JDI（`com.sun.jdi.VirtualMachine`）。

# 动态类文件常量

JEP 309 扩展了现有的 Java 类文件格式，创建`CONSTANT_Dynamic`。这是一个 JVM 功能，不依赖于更高层的软件。`CONSTANT_Dynamic`的加载将创建引导方法的创建委托给其他地方。当与**invokedynamic**调用进行比较时，您将看到它是如何将链接委托给引导方法的。

动态类文件常量的一个主要目标是使其能够轻松创建可物化的类文件常量的新形式，这为语言设计者和编译器实现者提供了更广泛的表达性和性能选择。这是通过创建一个新的常量池形式来实现的，该形式可以使用具有静态参数的自举方法进行参数化。

# 改进 AArch64 内联函数

JEP 315 通过改进 AArch64 处理器上的内联函数来工作。当前的字符串和数组内联函数得到了改进。同时，在 `java.lang.Math` 中实现了新的内联函数，用于正弦、余弦和对数函数。

为了提高应用程序性能，内联函数使用针对 CPU 架构特定的汇编代码。它不会执行通用的 Java 代码。

注意，您将看到 AArch64 处理器实现了大多数内联函数。然而，JEP 315 在 `java.lang.Math` 类中实现了以下方法的优化内联函数，但并未达到预期：

+   `sin()`

+   `cos()`

+   `log()`

还值得注意的是，在 AArch64 端口之前实现的一些内联函数可能并不完全优化。这样的内联函数可以利用诸如内存地址对齐或软件预取指令等特性。以下列出了一些这些方法：

+   `String::compareTo`

+   `String::indexOf`

+   `StringCoding::hasNegatives`

+   `` `Arrays::equals` ``

+   `StringUTF16::compress`

+   `StringLatin1::inflate`

# 移除 Java EE 和 CORBA 模块

Java EE 在 Java 9 中迁移到了 Eclipse 基金会，并采用了新的名称——Jakarta EE（有趣的是，仍然是 JEE）。在 Java 9 中，特定于 Java EE 的模块和类已被弃用。在 Java 11 中，这些弃用的 API 和模块已被从 Java SE 平台和 JDK 中删除。CORBA 的 API 也在 Java 9 中被弃用，并在 Java 11 中最终被删除。

在 Java SE 6（核心 Java）中，您可以使用以下技术来开发 Web 服务：

+   **JAX-WS**（Java API for XML-Based Web Services 的缩写）

+   **JAXB**（Java Architecture for XML Binding 的缩写）

+   **JAF**（JavaBeans Activation Framework 的缩写）

+   **常用注解**

当将上述技术堆栈的代码添加到核心 Java 中时，它与 **Java 企业版**（**JEE**）的版本相同。然而，随着时间的推移，JEE 版本发生了演变，导致 Java SE 和 JEE 中相同 API 提供的功能不匹配。

JEP 320 在 Java 11 中删除了以下模块，从 OpenJDK 仓库和运行时 JDK 图像中删除了它们的源代码：

+   `java.xml.ws` (JAX-WS)

+   `java.xml.bind` (JAXB)

+   `java.activation` (JAF)

+   `java.xml.ws.annotation` (常用注解)

+   `java.corba` (CORBA)

+   `java.transaction` (JTA)

+   `java.se.ee`（前六个模块的聚合模块）

+   `jdk.xml.ws` (JAX-WS 工具)

+   `jdk.xml.bind` (JAXB 工具)

除了在 Java 9 中将前面的模块标记为已弃用外，JDK 在编译或执行使用这些模块的代码时并没有解决这些问题。这迫使开发者必须在类路径上使用 Java EE 或 CORBA 的独立版本。

在它们被移除后，`wsgen`和`wsimport`（来自`jdk.xml.ws`）、`schemagen`和`xjc`（来自`jdk.xml.bind`）、`idlj`、`orbd`、`servertool`和`tnamesrv`（来自`java.corba`）等工具不再可用。开发者无法通过运行时命令行标志来启用它们，如下所示：

```java
    --add-modules
```

CORBA，一个 ORB 实现，于 1998 年被包含在 Java SE 中。随着时间的推移，对 CORBA 的支持超过了其好处。首先，随着更好的技术的可用性，现在几乎没有人使用 CORBA。CORBA 在**Java 社区进程**（**JCP**）之外发展，维护 JDK 的 CORBA 实现变得越来越困难。由于 JEE 现在正转向 Eclipse 基金会，将 JDK 中的 ORB 与 Jakarta EE 的 ORB 同步就没有意义了。更不用说，JEE 8 将其指定为**建议可选**，这实际上意味着 JEE 可能在未来的某个版本中放弃对 CORBA 的支持（将其标记为已弃用）。

正在从 Java 8 迁移到 Java 11 的企业，如果它们的应用程序使用 JEE 或 CORBA API，将面临更高的风险。然而，Oracle 建议使用一个替代 API，这有助于简化迁移过程。

# 与 Curve25519 和 Curve448 曲线的关键协议

通过 JEP 324，Java SE 在密码学方面取得了进一步进展，提供了安全和性能。此功能实现了使用 Curve25519 和 Curve448 的关键协议。其他密码学库，如 OpenSSL 和 BoringSSL，已经支持使用 Curve25519 和 Curve448 进行密钥交换。

你可以在[`tools.ietf.org/html/rfc7748`](https://tools.ietf.org/html/rfc7748)上找到更多关于 Curve25519 和 Curve448 的信息。

# Unicode 10

JEP 327 将现有平台 API 升级以支持 Unicode 10 标准([`www.unicode.org/standard/standard.html`](http://www.unicode.org/standard/standard.html))，主要在以下类中：

+   `Character`和`String`（位于`java.lang`包）

+   `NumericShaper`（位于`java.awt.font`包）

+   `Bidi`、`BreakIterator`和`Normalizer`（位于`java.text`包）

# ChaCha20 和 Poly1305 加密算法

Java 11 在密码工具包和 TLS 实现中包含了多个新增和增强功能。JEP 329 实现了 ChaCha20 和 ChaCha20-Poly1305 密码。作为一个相对较新的流密码，ChaCha20 能够取代 RC4 流密码。

目前，广泛采用的 RC4 流密码并不那么安全。行业正在转向采用更安全的 ChaCha20-Poly1305。这也已被广泛采用在 TLS 实现以及其他加密协议中。

# 启动单个文件源代码程序

想象一下能够在不进行编译的情况下执行 Java 应用程序；例如，如果你在 `HelloNoCompilation.java` 文件中定义以下 Java 类：

```java
class HelloNoCompilation { 
    public static void main(String[] args) { 
        System.out.println("No compilation! Are you kidding me?"); 
    } 
} 
```

使用 Java 11，你可以使用以下命令执行它（不进行编译）：

```java
    > java HelloNoCompilation.java
```

注意，上述命令使用 `java` 启动 JVM，它传递了一个具有 `.java` 扩展名的源文件名。在这种情况下，类是在 JVM 执行之前在内存中编译的。这适用于在同一源文件中定义的多个类或接口。以下是一个示例（假设它定义在同一个 `HelloNoCompilation.java` 源文件中）：

```java
class HelloNoCompilation { 
    public static void main(String[] args) { 
        System.out.println("No compilation! Are you kidding me?"); 
        EstablishedOrg org = new EstablishedOrg(); 
        org.invite(); 
        System.out.println(new Startup().name); 
    } 
} 
class Startup { 
    String name = "CoolAndExciting"; 
} 
interface Employs { 
    default void invite() { 
        System.out.println("Want to work with us?"); 
    } 
} 
class EstablishedOrg implements Employs { 
    String name = "OldButStable"; 
} 
```

在执行时，你会看到以下命令：

```java
    > java HelloNoCompilation.java
```

上述代码将输出如下：

```java
    No compilation! Are you kidding me?
    Want to work with us?
    CoolAndExciting  
```

使用 JEP 330，你可以减少编译代码的步骤，直接进入执行 Java 应用程序。然而，这仅适用于单源文件的应用程序。源文件可以定义多个类或接口，如前例代码所示。

启动单文件源代码程序有助于减少简单代码执行所需的仪式。这对刚开始学习 Java 的学生或专业人士来说非常有帮助。然而，当他们开始使用多个源文件工作时，他们需要在执行之前编译他们的代码。

然而，如果你使用 `javac` 命令编译源类，然后尝试将其作为单文件源代码启动，它将无法执行。例如，按照以下方式编译 `HelloNoCompilation` 源文件：

```java
    > javac HelloNoCompilation.java
```

然后，尝试执行以下命令：

```java
    > java HelloNoCompilation.java  
```

你将收到以下错误：

```java
    error: class found on application class path: HelloNoCompilation  
```

# TLS 1.3

这里是 Java 中 TLS 实现的另一个补充。JEP 332 实现了 TLS 协议的 1.3 版本。

TLS 1.3 版本取代并过时了其之前的版本，包括 1.2 版本（即 RFC 5246，可以在 [`tools.ietf.org/html/rfc5246`](https://tools.ietf.org/html/rfc5246) 找到）。它还过时或更改了其他 TLS 功能，例如 **OCSP**（即 **在线证书状态协议**）的粘贴扩展（即 RFC 6066，可以在 [`tools.ietf.org/html/rfc6066`](https://tools.ietf.org/html/rfc6066) 找到；以及 RFC 6961，可以在 [`tools.ietf.org/html/rfc6961`](https://tools.ietf.org/html/rfc6961) 找到) 和会话哈希以及扩展主密钥扩展（即 RFC 7627；更多信息，请访问 [`tools.ietf.org/html/rfc7627`](https://tools.ietf.org/html/rfc7627)）。

# 弃用 Nashorn JavaScript 引擎

使用 JEP 335，Java 11 弃用了 Nashorn JavaScript 脚本引擎、其 API 和其 `jjs` 工具。这些将在未来的 Java 版本中删除。

Nashorn JavaScript 引擎首次包含在 JDK 的最新版本中——JDK 8。这样做的原因是为了替换 Rhino 脚本引擎。然而，Java 无法跟上基于 ECMAScript 的 Nashorn JavaScript 引擎的演变步伐。

维护 Nashorn JavaScript 引擎的挑战超过了它所提供的优势，因此为其废弃铺平了道路。

# JEP 336 – 废弃 pack200 工具和 API

Java 5 中引入的 pack200 是一种用于 JAR 文件的压缩方案。它用于在打包、传输或交付 Java 程序时减少磁盘空间和带宽需求。开发者使用 pack200 和 unpack200 来压缩和解压缩 Java JAR 文件。

然而，随着今天现代存储和传输技术的改进，这些变得不再相关。JEP 336 废弃了 pack200 和 unpack200 工具，以及相应的 pack200 API。

# 摘要

在本章中，我们介绍了 Java 11 的各种特性。我们看到了在不同版本中引入了众多变化。

在下一章中，我们将发现 Java 语言在 **Project Amber** 项目中的新增功能和修改——该项目旨在调整 Java 语言的规模。
