

# 第一章：探秘 Java 虚拟机

我将向您介绍一项革命性的技术，它帮助改变了软件行业。认识一下**Java 虚拟机**（**JVM**）。好吧，你可能已经对 JVM 很熟悉了，了解并欣赏它在编译 Java 字节码和几乎无限数量的硬件平台之间的中间件所具有的巨大价值是非常重要的。Java 和 JVM 的巧妙设计是它广受欢迎和用户、开发者价值的证明。

自从 20 世纪 90 年代 Java 的第一个版本发布以来，JVM 一直是 Java 编程语言真正的成功因素。有了 Java 和 JVM，“一次编写，到处运行”的概念应运而生。Java 开发者可以编写一次程序，并允许 JVM 确保代码在安装了 JVM 的设备上运行。JVM 还使 Java 成为一种平台无关的语言。本章的主要目标是更深入地了解 Java 的默默无闻的英雄——JVM。

在本章中，我们将深入探讨 JVM，以便我们能够学会如何充分利用它，从而提高我们的 Java 应用程序的性能。

在本章中，我们将涵盖以下主题：

+   JVM 的工作原理

+   垃圾回收

+   **即时编译器**（**JIT**）优化

到本章结束时，您应该了解如何充分利用 JVM 来提高您的 Java 应用程序的性能。

# 技术要求

要遵循本章中的说明，您需要以下内容：

+   安装了 Windows、macOS 或 Linux 的计算机

+   当前版本的 Java SDK

+   建议使用代码编辑器或**集成开发环境**（**IDE**）（如 Visual Studio Code、NetBeans、Eclipse 或 IntelliJ IDEA）

本章的完整代码可以在以下位置找到：[`github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter01`](https://github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter01)。

重要提示

本书基于 Java 21 和**Java 开发工具包**（**JDK**）21.0.1。它还使用了在 macOS 上运行的 IntelliJ IDEA Community Edition IDE。

本章包含您可以用来跟随和实验的代码示例。因此，您需要确保您的系统已经正确准备。

首先，下载并安装您选择的 IDE。这里有一些选项：

+   Visual Studio Code ([`code.visualstudio.com/download`](https://code.visualstudio.com/download))

+   NetBeans ([`netbeans.apache.org/download/`](https://netbeans.apache.org/download/))

+   Eclipse ([`www.eclipse.org/downloads/`](https://www.eclipse.org/downloads/))

+   IntelliJ IDEA Community Edition ([`www.jetbrains.com/idea/download/`](https://www.jetbrains.com/idea/download/))

在你的 IDE 设置完成后，你需要确保它已配置为 Java 开发。大多数现代 IDE 都具备为你下载和安装 Java SDK 的功能。如果你不是这种情况，Java SDK 可以从这里获取：[`www.oracle.com/java/technologies/downloads/`](https://www.oracle.com/java/technologies/downloads/).

确保你的 IDE 已准备好 Java 开发非常重要。以下是一些 IDE 特定的链接，以防你需要一些帮助：

+   Visual Studio Code 中的 Java ([`code.visualstudio.com/docs/languages/java`](https://code.visualstudio.com/docs/languages/java))

+   NetBeans 的 Java 快速入门教程 ([`netbeans.apache.org/tutorial/main/kb/docs/java/quickstart/`](https://netbeans.apache.org/tutorial/main/kb/docs/java/quickstart/))

+   准备 Eclipse ([`help.eclipse.org/latest/index.jsp?topic=%2Forg.eclipse.jdt.doc.user%2FgettingStarted%2Fqs-3.htm`](https://help.eclipse.org/latest/index.jsp?topic=%2Forg.eclipse.jdt.doc.user%2FgettingStarted%2Fqs-3.htm))

+   使用 IntelliJ IDEA Community Edition 创建你的第一个 Java 应用程序 ([`www.jetbrains.com/help/idea/creating-and-running-your-first-java-application.html`](https://www.jetbrains.com/help/idea/creating-and-running-your-first-java-application.html))

一旦你在电脑上安装并配置了 IDE 和 Java SDK，你就可以进入下一部分了。

# JVM 的工作原理

在核心上，JVM 位于你的 Java 字节码和电脑之间。如图所示，我们在 IDE 中开发 Java 源代码，我们的工作以`.java`文件保存。我们使用 Java 编译器将 Java 源代码转换为字节码；生成的文件是`.class`文件。然后我们使用 JVM 在我们的电脑上运行字节码：

![图片](img/B21942_01_1.jpg)

图 1.1 – Java 应用程序工作流程

重要的是要认识到 Java 与典型的编译和执行语言不同，在这些语言中，源代码被输入到编译器中，然后产生一个`.exe`文件（Windows），一个`.app`文件（macOS），或者`ELF`文件（Linux）。这个过程比看起来要复杂得多。

让我们用一个基本示例来更详细地看看 JVM 的工作原理。在下面的代码中，我们实现了一个简单的循环，它会打印到控制台。这样呈现是为了让我们看到 JVM 是如何处理 Java 代码的：

```java
// Chapter1
// Example 1
public class CH1EX1 {
    public static void main(String[] args) {
        System.out.println("Basic Java loop.");
        // Basic loop example
        for (int i = 1; i <= 5; i++) {
            System.out.println("1 x " + i + " = " + i);
        }
    }
}
```

我们可以使用 Java 编译器将我们的代码编译成`.class`文件，这些文件将以字节码格式排列。让我们打开一个终端窗口，导航到我们的项目`src`文件夹，如以下所示，并使用`ls`命令来显示我们的`.java`文件：

```java
$ cd src
$ ls
CH1EX1.java
$
```

现在我们已经进入了正确的文件夹，我们可以使用`javac`将我们的源代码转换为字节码。正如你接下来可以看到的，这创建了一个`.class`文件：

```java
$ javac CH1EX1.java
$ ls
CH1EX1.class    CH1EX1.java
$
```

接下来，我们可以使用`javap`来打印我们的字节码的反编译版本。正如你接下来可以看到的，我们可以使用`javap`命令而不带任何参数：

```java
javap CH1EX1.class
Compiled from "CH1EX1.java"
public class CH1EX1 {
  public CH1EX1();
  public static void main(java.lang.String[]);
}
```

如您所见，我们使用`javap`命令简化了反编译字节码的打印。现在，让我们使用`-c`参数来揭示字节码：

```java
$ javap -c CH1EX1.class
```

这里是使用`-c`参数运行`javap`命令并使用我们的示例代码的输出：

```java
Compiled from "CH1EX1.java"
public class CH1EX1 {
  public CH1EX1();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/
                                            Object."<init>":()V
       4: return
  public static void main(java.lang.String[]);
    Code:
       0: getstatic     #7                  // Field java/lang/System.
                                            out:Ljava/io/PrintStream;
       3: ldc           #13                 // String Basic Java loop.
       5: invokevirtual #15                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
       8: iconst_1
       9: istore_1
      10: iload_1
      11: iconst_5
      12: if_icmpgt     34
      15: getstatic     #7                  // Field java/lang/System.
                                            out:Ljava/io/PrintStream;
      18: iload_1
      19: iload_1
      20: invokedynamic #21,  0             // InvokeDynamic #0:makeConcatWithConstants:(II)Ljava/lang/String;
      25: invokevirtual #15                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      28: iinc          1, 1
      31: goto          10
      34: return
}
```

您现在应该对 JVM 所做奇迹般的工作有了更好的认识。我们才刚刚开始！

## 使用 javap 的更多选项

到目前为止，我们已经以两种形式使用了`javap`命令：不带参数和带`-c`参数。还有一些其他参数我们可以使用，以进一步了解我们的字节码。为了回顾这些参数，我们将使用一个简化的示例，如下所示：

```java
public class CH1EX2 {
    public static void main(String[] args) {
        System.out.println("Simple Example");
    }
}
```

您现在可以先使用`javac`编译类，然后运行它。让我们使用`-sysinfo`参数输出系统信息。如您所见，打印了文件路径、文件大小、日期和`SHA-256`哈希值：

```java
$ javap -sysinfo CH1EX2.class
```

这里是使用`-sysinfo`参数的输出：

```java
Classfile /HPWJ/Code/Chapter1/CH1Example2/src/CH1EX2.class
  Last modified Oct 22, 2023; size 420 bytes
  SHA-256 checksum 9328e8bab7fcd970f73fc9eec3a856a809b7a45e7b743f8c8b3b7ae0a7fbe0da
  Compiled from "CH1EX2.java"
public class CH1EX2 {
  public CH1EX2();
  public static void main(java.lang.String[]);
}
```

让我们再看一个`javap`命令的例子，这次使用`-verbose`参数。以下是使用终端命令行使用该参数的方法：

```java
$ javap -verbose CH1EX2.class
```

使用`-verbose`参数的输出如下。如您所见，提供了大量附加信息：

```java
Classfile /HPWJ/Code/Chapter1/CH1Example2/src/CH1EX2.class
  Last modified Oct 22, 2023; size 420 bytes
  SHA-256 checksum 9328e8bab7fcd970f73fc9eec3a856a809b7a45e7b743f8c8b3b7ae0a7fbe0da
  Compiled from "CH1EX2.java"
```

剩余的输出可以在此处获得：[`github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter01/java-output`](https://github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter01/java-output)。

我们可以使用`javap`命令彻底检查我们的字节码。以下是一些常见的参数，除了我们已使用的`-c`、`-sysinfo`和`-verbose`之外：

| **javap 参数** | **打印内容** |
| --- | --- |
| `-constants` | 常量（静态 final） |
| `-help (或 -?)` | 帮助 |
| `-l` | 行变量和局部变量 |
| `-private (或 -p)` | 所有类 |
| `-protected` | 受保护和公共类（非私有） |
| `-public` | 公共类（不打印私有类） |
| `-s` | 内部签名类型 |
| `-sysinfo` | Java 发布版本 |

表 1.1 – javap 参数

现在您已经看到了 JVM 的内部工作原理，让我们看看它在垃圾收集方面所做的不凡工作。

# 垃圾收集

Java 开发者长期以来一直享受 JVM 管理内存的能力，包括分配和释放。内存管理的分配部分很简单，没有固有的问题。最重要的领域是释放不再由应用程序需要的先前分配的内存。这被称为释放或垃圾收集。虽然并非 Java 编程语言独有，但其 JVM 在垃圾收集方面做得非常出色。本节将详细探讨 JVM 的垃圾收集。

## 垃圾收集过程

以下示例是创建两个对象，使它们相互引用，然后使它们都为 null。一旦它们被置为 null，尽管它们相互引用，但它们就不再可达。这使得它们有资格进行垃圾收集：

```java
public class Main {
    public static void main(String[] args) {
        // Creating two objects
        SampleClass object1 = new SampleClass("Object 1");
        SampleClass object2 = new SampleClass("Object 2");
        // Making the objects reference each other
        object1.reference = object2;
        object2.reference = object1;
        // References are nullified
        object1 = null;
        object2 = null;
    }
}
class SampleClass {
    String name;
    SampleClass reference;
    public SampleClass(String name) {
        this.name = name;
    }
    // Overriding finalize() method to see the garbage collection 
    // process
    @Override
    protected void finalize() throws Throwable {
        System.out.println(name + " is being garbage collected!");
        super.finalize();
    }
}
```

在前面的示例中，我们可以显式地调用垃圾收集器。这通常不建议这样做，因为 JVM 已经在这方面做得很好，额外的调用可能会影响性能。如果您确实想进行显式调用，以下是这样做的方法：

```java
System.gc();
```

关于`finalize()`方法的说明

JVM 仅在启用的情况下才会调用`finalize()`方法。启用后，它可能会在不确定的时间延迟后由垃圾收集器调用。该方法已被弃用，并且仅应用于测试，而不应在生产系统中使用。

如果你想进行额外的测试，你可以添加尝试访问不可达对象的代码。以下是这样做的方法：

```java
try {
  object1.display();
  } catch (NullPointerException e) {
    System.out.println("Unreachable object!");
}
```

前面的`try`-`catch`块调用了`object1`。由于该对象不可达，它将抛出`NullPointerException`异常。

## 垃圾收集算法

JVM 有几种垃圾收集算法可供选择，具体使用哪种算法取决于 Java 的版本和特定的使用场景。以下是一些值得注意的垃圾收集算法：

+   **串行收集器**

+   **并行收集器**

+   **并发标记清除** **收集器**（**CMS**）

+   **垃圾-第一**（**G1**）**收集器**

+   **Z 垃圾** **收集器**（**ZGC**）

以下将逐一介绍这些垃圾收集算法，以便您更好地理解 JVM 为我们分配内存所做的大量工作。

**串行收集器**

串行垃圾收集器是 JVM 最基础的算法。它用于小型堆和单线程应用程序。这类应用程序的特征是顺序执行，而不是并发执行。由于只有一个线程需要考虑，内存管理要容易得多。另一个特征是，单线程应用程序通常只使用主机 CPU 能力的一部分。最后，由于执行是顺序的，预测垃圾收集的需要很简单。

**并行收集器**

并行垃圾收集器，也称为吞吐量垃圾收集器，在 Java 8 之前的早期 Java 版本中是默认的。它的能力超过了串行收集器，因为它是为中等到大型堆和多线程应用程序设计的。

**CMS**

CMS 垃圾收集器在 Java 9 中被弃用，并在 Java 14 中从 Java 平台中移除。其目的是最小化应用程序因垃圾收集而暂停的时间。弃用和移除的原因包括资源消耗高、由于延迟导致的失败、维护代码库困难，以及有更好的替代方案。

**G1 收集器**

G1 收集器在 Java 9 中成为默认的垃圾收集器，取代了并行收集器。G1 是为了提供更快、更可预测的响应时间和极高的吞吐量而设计的。G1 收集器可以被认为是主要的和默认的垃圾收集器。

**ZGC**

Java 11 发布时，ZGC 垃圾收集器作为一个实验性功能。ZGC 的目标是提供一个低延迟、可扩展的垃圾收集器。目标是让 ZGC 能够处理从非常小到非常大的各种堆大小（例如，太字节）。ZGC 实现了这些目标，而没有牺牲暂停时间。这一成功导致 ZGC 作为 Java 15 的一个功能被发布。

## 垃圾收集优化

精明的 Java 开发者专注于为他们的应用程序从 JVM 中获取最佳性能。为此，可以通过垃圾收集来采取一些措施，以帮助提高 Java 应用程序的性能：

1.  根据你的用例、堆大小和线程数选择最合适的垃圾收集算法。

1.  初始化、监控和管理你的堆大小。它们不应超过绝对需要的大小。

1.  限制对象创建。我们将在*第六章*中讨论这个问题的替代方案。

1.  为你的应用程序使用适当的数据结构。这是*第二章*的主题。

1.  如本章前面所述，为了避免或至少显著限制对 `finalize` 方法的调用，以减少垃圾收集延迟和处理开销。

1.  优化字符串的使用，特别关注最小化重复字符串。我们将在*第七章*中更深入地探讨这个问题。

1.  注意不要在 JVM 的垃圾收集范围之外分配内存（例如，本地代码）。

1.  最后，使用工具来帮助你在实时监控垃圾收集。我们将在*第十四章*中回顾一些这些工具。

希望你已经对 JVM 的垃圾收集和优化方法有了新的认识。现在，我们将探讨具体的编译器优化。

# JIT 编译器优化

JVM 有三个关键组件：一个初始化和链接类和接口的类加载器；运行时数据，包括内存分配；以及执行引擎。本节的重点是这个执行引擎组件。

执行引擎的核心责任是将字节码转换为可以在主机 **中央处理单元**（**CPU**）上执行的形式。这个过程有三个主要实现：

+   解释

+   **提前编译**（**AOT**）

+   JIT 编译

在深入研究 JIT 编译优化之前，我们将简要地看看解释和 AOT 编译。

## 解释

解释是 JVM 可以用来读取和执行字节码而不将其转换为宿主 CPU 的本地机器码的技术。我们可以通过在终端窗口中使用`java`命令来调用此模式。以下是一个示例：

```java
$ java Main.java
```

使用解释模式的唯一真正优势是通过避免编译过程来节省时间。这对于测试可能是有用的，但不建议用于生产系统。与 AOT 和 JIT（接下来将描述）相比，使用解释通常会出现性能问题。

## AOT 编译

我们可以在应用程序执行之前将我们的字节码编译成本地机器码。这种方法称为 AOT，可以用于性能提升。这种方法的具体优势包括以下内容：

+   你可以避免与传统的 JIT 编译过程相关的正常应用程序启动延迟

+   启动执行速度保持一致

+   启动时的 CPU 负载通常较低

+   通过避免其他编译方法来减轻安全风险

+   你可以在启动代码中构建优化

使用 AOT 编译方法也有一些缺点：

+   当你在编译时，你将失去在任何设备上部署的能力

+   可能你的应用程序会变得臃肿，增加存储和相关成本

+   你的应用程序将无法利用与 JIT 编译关联的 JVM 优化

+   代码维护的复杂性增加

了解其优势和劣势可以帮助你决定何时以及何时不使用 AOT 编译过程。

## JIT 编译

JIT 编译过程可能是我们最熟悉的。我们调用 JVM，并让它将我们的字节码转换为针对当前主机机器的特定机器码。这种机器码被称为本地机器码，因为它对本地 CPU 来说是本地的。这种编译是在即时发生的，或者是在执行之前。这意味着不是所有的字节码都会一次性编译。

JIT 编译的优点是相对于解释方法，性能提高，能够在任何设备（平台无关）上部署你的应用程序，以及能够进行优化。JIT 编译器过程能够优化代码（例如，删除死代码、循环展开等）。缺点是初始启动时的开销以及由于需要存储本地机器码翻译而导致的内存使用。

# 摘要

到现在为止，你应该已经对 JVM 的复杂性和其工作原理有了认识。本章涵盖了`javac`和`javap`命令行工具，用于创建和分析字节码。JVM 的垃圾回收功能也通过应用程序性能的角度进行了考察。最后，还介绍了 JIT 编译的优化。

到本书出版之日，JVM 已经有 29 年的历史，自从它的首次发布以来已经取得了长足的进步。除了持续的改进和优化之外，JVM 还能支持额外的语言（例如 Kotlin 和 Scala）。对持续提高其 Java 应用性能感兴趣的 Java 开发者应该关注 JVM 的更新。

对于 JVM 的扎实理解，我们在下一章将关注数据结构。我们的重点将放在如何最优地使用数据结构作为我们高性能策略的一部分。
