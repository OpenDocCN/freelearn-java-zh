# 3

# 理解字节码

在 JVM 错综复杂的领域中，字节码作为中介语言，使 Java 程序能够超越特定平台硬件和操作系统的限制。当我们深入 JVM 内部的核心时，本章专注于解码字节码，这是执行 Java 应用程序的基本组件。字节码，作为一组指令的表示，充当了高级 Java 代码和底层硬件特定语言之间的桥梁。通过理解字节码，开发者可以深入了解 JVM 的内部工作原理，从而优化代码性能并解决复杂问题。

字节码的核心是一系列指令，这些指令决定了 JVM 执行的底层操作。本章揭示了算术操作的细微差别，阐明了 JVM 如何处理数学计算。从基本的加法和减法到更复杂的程序，我们探讨了管理这些过程的字节码指令。此外，我们还深入探讨了值转换，揭示了 JVM 如何在不同类型之间转换数据。理解这些底层操作对于寻求优化应用程序性能和效率的开发者至关重要。加入我们，一起探索字节码领域，在这里，算术操作和值转换的复杂性为掌握 JVM 铺平了道路。

在本章中，我们将探讨以下主题：

+   查看字节码

+   算术运算

+   值转换

+   对象操作

+   条件指令

# 技术要求

对于本章，你需要以下内容：

+   Java 21

+   Git

+   Maven

+   任何首选的 IDE

+   本章的 GitHub 仓库位于 - [`github.com/PacktPublishing/Mastering-the-Java-Virtual-Machine/tree/main/chapter-03`](https://github.com/PacktPublishing/Mastering-the-Java-Virtual-Machine/tree/main/chapter-03)

# 查看字节码

字节码，Java 编程中的关键概念，是促进 Java 应用程序在 JVM 上实现跨平台兼容性和执行的中介语言。本次会议旨在揭开字节码的神秘面纱，提供一个对其重要性、目的以及它在 JVM 内允许的操作范围的全面概述。

在其核心，字节码充当了高级 Java 代码和底层硬件特定语言之间的桥梁。当 Java 程序编译时，源代码被转换成字节码，这是一组 JVM 可理解的指令。这种平台无关的字节码允许 Java 应用程序在不同环境中无缝执行，这是 Java“一次编写，到处运行”信条的基本原则。

为什么我们有字节码？答案在于它为 Java 应用程序带来的可移植性和多功能性。通过在高级源代码和机器代码之间引入一个中间步骤，Java 程序可以在任何配备 JVM 的设备上运行，无论其架构或操作系统如何。这种抽象保护开发者免受硬件特定细节的复杂性，促进了一个更通用和易于访问的编程环境。

现在，让我们深入探讨字节码中编码的操作。字节码指令涵盖了众多功能，从基本的加载和保存操作到复杂的算术计算。JVM 的基于栈的架构控制这些操作，其中值被推入和弹出栈，形成数据操作的基础。包括加法、减法、乘法等在内的算术操作通过特定的字节码指令执行，使开发者能够理解和优化代码的数学基础。

**值转换**是字节码操作的另一个方面，涉及在不同类型之间转换数据。无论是将整数转换为浮点数还是管理其他类型转换，字节码指令为这些操作提供了基础。这种灵活性对于开发者编写代码和在 Java 生态系统中无缝处理各种数据类型至关重要。

除了这个之外，字节码负责对象的创建和操作，控制条件语句，并管理方法的调用和返回。每条字节码指令都对 Java 程序的总体执行流程做出贡献，理解这些操作使开发者能够构建高效、性能良好且可靠的应用程序。

事实上，理解字节码行为对于导航 JVM 的复杂性至关重要。字节码指令旨在操作特定类型的值，识别被操作的类型对于编写高效和正确的 Java 代码是基本的。每个字节码助记符的首字母通常是一个宝贵的提示，有助于辨别正在执行的操作类型。

让我们深入探讨这个基于字节码助记符首字母识别操作类型的技巧：

+   **i 用于整数操作**：以**i**开头的字节码，例如**iload**（加载整数）、**iadd**（加整数）或**isub**（减整数），表示涉及整数值的操作。这些字节码指令操作存储为 32 位有符号整数的数据。

+   **l 用于长操作**：**l**前缀，如在**lload**（加载长整型）或**lmul**（乘长整型）中看到的那样，表示对 64 位有符号长整数的操作。

+   **s 用于短操作**：以**s**开头的字节码，例如**sload**（加载短整型），与 16 位有符号短整数的操作相关。

+   **b 用于字节操作**：在**bload**（加载字节）等字节码指令中发现的**b**前缀表示对 8 位有符号字节整数的操作。

+   **c 用于字符操作**：对 16 位 Unicode 字符的操作通过以**c**开头的字节码指令表示，例如**caload**（加载**char**数组）。

+   **f 用于浮点操作**：在**fload**（加载浮点数）或**fadd**（加浮点数）等字节码助记符中看到的**f**前缀表示涉及 32 位单精度浮点数的运算。

+   **d 用于双精度操作**：双精度浮点数（64 位）是字节码指令以**d**开头关注的焦点，例如**dload**（加载双精度）或**dmul**（乘双精度）。

+   **a 用于引用操作**：涉及对象引用的运算通过以**a**开头的字节码指令表示，例如**aload**（加载引用）或**areturn**（返回引用）。

这种系统性的命名约定有助于开发者快速识别字节码指令所操作的数据类型。通过识别初始字母并将其与特定数据类型关联，开发者可以编写更明智和精确的代码，确保字节码操作与 JVM 中预期的数据类型和行为保持一致。这种理解对于掌握字节码和优化 Java 应用程序的性能和可靠性至关重要。

在 Java 字节码中，布尔值通常使用整数表示（`0`表示`false`，`1`表示`true`）。然而，需要注意的是，布尔值没有专门的字节码指令；相反，使用标准的整数算术和逻辑指令。例如：

+   **iadd**、**isub**、**imul**、**idiv**以及类似的指令可以无缝地与布尔值一起工作

+   逻辑运算如*与*（**iand**）、*或*（**ior**）和*异或*（**ixor**）可用于布尔逻辑

关键要点是布尔值在字节码中被视为整数，这使得开发者可以使用相同的算术和逻辑指令进行数值和布尔计算。

字节码为算术运算提供了基础，塑造了 Java 程序数学核心。我们的旅程将在下一节继续，我们将深入探讨字节码中算术运算的复杂世界。我们将剖析控制加法、减法、乘法等运算的指令，揭示定义 Java 应用程序数学本质的字节码序列。

通过理解字节码中编码的算术运算，开发者可以深入了解他们代码的内部工作原理，从而能够优化性能并提高效率。请加入我们下一节，揭开算术运算背后的秘密，为掌握 JVM 的复杂性铺平道路。

# 算术运算

在本节中，我们专注于探索字节码的一个基石方面：算术操作。这些操作是数学基础，为 Java 程序注入活力，塑造了 JVM 中计算的数值景观。

字节码算术操作遵循一个基本原则：它们在`操作数栈`上的前两个值上操作，执行指定的操作，并将结果返回到栈中。本节深入探讨字节码算术的复杂性，揭示其细微差别、行为和程序执行的影响。

字节码中的算术操作分为两大类：涉及浮点数的操作和涉及整数的操作。每一类都表现出不同的行为，理解这些差异对于寻求精确性和可靠性的 Java 开发者来说至关重要。

在我们探索字节码算术领域时，我们研究了控制浮点数和整数加、减、乘、除的指令。我们剖析了封装这些操作的字节码序列，阐明其实现和性能影响。

## 加法、减法、乘法和除法

基本算术操作是 Java 中数值计算的基础。从整数的加法（`iadd`）到双精度的除法（`ddiv`），每条字节码指令都是精心设计的，以处理特定的数据类型。揭示加法、减法、乘法和除法整数、长整数、浮点数和双精度的细微差别：

+   **加法**:

    +   **iadd**: 两个整数相加

    +   **ladd**: 两个长整数相加

    +   **fadd**: 两个浮点数相加

    +   **dadd**: 两个双精度浮点数相加

+   **减法**:

    +   **isub**: 从第一个整数中减去第二个整数

    +   **lsub**: 从第一个长整数中减去第二个长整数

    +   **fsub**: 从第一个浮点数中减去第二个浮点数

    +   **dsub**: 从第一个双精度浮点数中减去第二个双精度浮点数

+   **乘法**:

    +   **imul**: 乘以两个整数

    +   **lmul**: 乘以两个长整数

    +   **fmul**: 乘以两个浮点数

    +   **dmul**: 两个双精度浮点数相乘

+   **除法**:

    +   **idiv**: 第一个整数除以第二个整数

    +   **ldiv**: 第一个长整数除以第二个长整数

    +   **fdiv**: 第一个浮点数除以第二个浮点数

    +   **ddiv**: 第一个双精度浮点数除以第二个双精度浮点数

## 余数和取反

+   **余数（余数）**:

    +   **irem**: 计算第一个整数除以第二个整数的余数

    +   **lrem**: 计算第一个长整数除以第二个长整数的余数

    +   **frem**: 计算第一个浮点数除以第二个浮点数的余数

    +   **drem**: 计算第一个双精度浮点数除以第二个双精度浮点数的余数

+   **取反（取反）**:

    +   **ineg**: 取反（改变整数的符号）

    +   **lneg**: 取反长整数

    +   **fneg**: 取反浮点数

    +   **dneg**: 取反双精度浮点数

## 移位和位运算

深入位运算（`ior`、`iand`、`ixor`、`lor`、`land`、`lxor`）和移位操作（`ishl`、`ishr`、`iushr`、`lshl`、`lshr`、`lushr`）的世界。了解这些操作如何操纵单个位，为高级计算和优化提供强大的工具：

+   **移位** **操作（移位）**:

    +   **ishl**, **ishr**, **iushr**: 将整数的位向左、右（带符号扩展）或右（不带符号扩展）移动

    +   **lshl**, **lshr**, **lushr**: 将长整数的位向左、右（带符号扩展）或右（不带符号扩展）移动

+   **位运算**:

    +   **ior**, **lor**: 整数和长整数的位或

    +   **iand**, **land**: 整数和长整数的位与

    +   **ixor**, **lxor**: 整数和长整数的位异或

## 局部变量增量

解锁 `iinc` 指令的潜力，这是一个微妙但强大的操作，可以通过一个常量值增加局部变量。了解这个字节码指令如何在特定场景中提高代码的可读性和效率：

+   **局部变量增量**（**iinc**）: **iinc** 通过一个常量值增加局部变量

## 比较操作

深入比较值的世界，使用 `cmpg`、`dcmpl`、`fcmpg`、`fcmpl` 和 `lcmp` 等指令。揭示产生 1、-1 或 0 等结果的内幕，这些结果表示双精度浮点数、浮点数和长整数的比较结果：

+   **比较**:

    +   **dcmpg**, **dcmpl**: 比较两个双精度浮点数，返回 1、-1 或 0（表示大于、小于或等于）

    +   **fcmpg**, **fcmpl**: 比较两个浮点数，返回 1、-1 或 0（表示大于、小于或等于）

    +   **lcmp**: 比较两个长整数，返回 1、-1 或 0（表示大于、小于或等于）

在字节码比较的世界中，`dcmpg` 和 `dcmpl` 指令有效地比较双精度浮点数，返回 1、-1 或 0 以表示大于、小于或等于比较。同样，`fcmpg` 和 `fcmpl` 处理单精度浮点数。然而，当涉及到长整数时，`lcmp` 通过提供一个单一的 1、-1 或 0 的结果来简化事情，表示大于、小于或等于比较。这种简化的方法优化了字节码中的长整数比较。

这些字节码指令构成了 Java 程序中算术和逻辑操作的基础。需要注意的是，这些操作的行为可能因整数和浮点数而异，尤其是在处理除以零或溢出条件等边缘情况时。理解这些字节码指令为开发者提供了在 Java 应用程序中构建精确且健壮数值计算的工具。

在解释了字节码的概念并展示了某些算术操作之后，我们将通过检查一个实际示例来深入探讨 JVM 内部的字节码算术世界。我们的重点是通过对一个简单的 Java 代码片段进行分析来理解这个过程，该代码片段执行基本的算术操作，即添加两个整数。这次动手探索旨在揭示字节码算术的复杂运作。

考虑以下 Java 代码片段：

```java
public class ArithmeticExample {    public static void main(String[] args) {
        int a = 5;
        int b = 7;
        int result = a + b;
        System.out.println("Result: " + result);
    }
}
```

将代码保存到名为`ArithmeticExample.java`的文件中，并使用以下命令进行编译：

```java
javac ArithmeticExample.java
```

现在，让我们使用`javap`命令来反汇编字节码：

```java
javap -c ArithmeticExample.class
```

执行命令后，它将生成字节码的输出：

```java
public static void main(java.lang.String[]);
```

代码如下：

```java
     ...     5: iload_1         // Load the value of 'a' onto the stack
     6: iload_2         // Load the value of 'b' onto the stack
     7: iadd            // Add the top two values on the stack                           (a and b) 
     8: istore_3        // Store the result into the local variable 'result'
     ...
```

在这些字节码指令中发生以下操作：

+   **iload_1**：将局部变量**a**的值加载到栈上

+   **iload_2**：将局部变量**b**的值加载到栈上

+   **iadd**：将栈顶的两个值（即**a**和**b**）相加

+   **istore_3**：将加法的结果存储回局部变量 result

这些字节码指令精确地反映了 Java 代码中的算术操作`int result = a + b`。`iadd`指令执行加载值的加法，而`istore_3`指令将结果存储回局部变量以供进一步使用。理解这些字节码提供了对 JVM 在 Java 程序中执行简单算术操作的详细视图。

在我们通过字节码算术的旅程中，我们已经剖析了在 Java 程序中添加两个整数的看似平凡却又深刻影响的过程。字节码指令揭示了隐藏的复杂层，展示了高级操作如何在 Java 虚拟机（JVM）中转化为可执行的机器代码。

当我们结束这一部分时，我们的下一个目的地等待着：值转换的领域。理解不同数据类型在字节码中的交互对于构建健壮和高效的 Java 应用程序至关重要。在下一节中，让我们深入探讨值转换的复杂性，揭示在 JVM 中转换数据的细微差别。旅程继续，每一行字节码指令都让我们更接近掌握 Java 字节码的深度。

# 值转换

在本节中，我们沉浸在 JVM 内部值转换的复杂领域。这些转换是字节码景观中的变色龙，通过允许整型扩展为长整型和浮点型超越为双精度浮点型，而不会损害原始值的忠实度，从而实现变量的类型优雅转换。促进这些变形的字节码指令对于保持精度、防止数据丢失和确保不同数据类型的无缝集成至关重要。加入我们，我们将剖析这些指令，揭示支撑 Java 编程的优雅和精确的交响曲：

+   **整型转换为长整型**（**i2l**）：探索**i2l**指令如何将整型变量提升为长整型，同时保留原始值的精度

+   **整型转换为浮点型**（**i2f**）：深入**i2f**的世界，其中整型可以优雅地转换为浮点型而不牺牲精度

+   **整型转换为双精度浮点型**（**i2d**）：见证通过**i2d**指令从整型到双精度浮点型的精度保持之旅

+   **长整型转换为浮点型**（**l2f**）和**长整型转换为双精度浮点型**（**l2d**）：考察**l2f**和**l2d**的优雅之处，其中长整型值可以无缝地转换为浮点型和双精度浮点型

+   **浮点型转换为双精度浮点型**（**f2d**）：探索**f2d**指令，展示浮点型提升为双精度浮点型的同时保持精度

在我们探索字节码的复杂性时，我们遇到了一个关键部分，专门用于管理缩短——这是一个标记着潜在损失和溢出考虑的微妙过程。在这个探索中，我们深入研究了将变量转换为较短数据类型的字节码指令，承认与精度损失和溢出风险相关的微妙挑战。现在，让我们来探讨这一组指令：

+   **整型转换为字节型**（**i2b**）、**整型转换为短整型**（**i2s**）、**整型转换为字符型**（**i2c**）：研究通过**i2b**、**i2s**和**i2c**指令将整型转换为字节型、短整型和字符型时可能出现的精度损失

+   **长整型转换为整型**（**l2i**）：考察使用**l2i**指令将长整型转换为整型时涉及的考虑因素，承认可能发生溢出的可能性

+   **浮点型转换为整型**（**f2i**）、**浮点型转换为长整型**（**f2l**）：揭示通过**f2i**和**f2l**将浮点型转换为整型和长整型时遇到的挑战，注意精度和溢出问题

+   **双精度浮点型转换为整型**（**d2i**）、**双精度浮点型转换为长整型**（**d2l**）、**双精度浮点型转换为浮点型**（**d2f**）：通过**d2i**、**d2l**和**d2f**指令导航，了解将双精度浮点型转换为整型、长整型和浮点型时精度和潜在溢出的微妙平衡

在字节码的复杂性领域，以下最佳实践作为指南，引导我们通过实际考虑。在这里，我们连接理论和应用，探讨字节码指令对现实世界 Java 编程场景的实质性影响。从在复杂算术操作中保持精度到导航面向对象设计的灵活性，这些实际考虑照亮了理解和掌握字节码在开发领域中的重要性。

+   **保持算术精度：**在值转换和算术操作之间建立联系，确保在复杂计算中保持精度

+   **处理对象引用：**探索值转换如何有助于面向对象编程的灵活性，允许在类和接口之间实现平滑过渡

在解码控制值转换的字节码指令时，前面的观点为您提供了导航变量类型在 JVM 内部转换细微差别的见解。

在以下示例 Java 代码片段中，我们聚焦于值转换，明确关注在 JVM 内部转换变量类型的约定。代码片段展示了提升和考虑精度损失或溢出的微妙舞蹈。随着我们遍历字节码结果，我们的注意力始终集中在使这些约定得以实现的指令上：

```java
public class ValueConversionsExample {    public static void main(String[] args) {
        // Promotion: Enlargement of Types
        int intValue = 42;
        long longValue = intValue; // Promotion: int to long
        float floatValue = 3.14f;
        double doubleValue = floatValue; // Promotion: float to double
        // Shortening: Considerations for Loss and Overflow
        short shortValue = 32767;
        byte byteValue = (byte) shortValue; // Shortening: short to 
                                            // byte 
        double largeDouble = 1.7e308;
        int intFromDouble = (int) largeDouble; // Shortening: double 
                                               // to int 
```

结果如下所示：

```java
        System.out.println("Promotion Results: " + longValue + ", " +           doubleValue);
        System.out.println("Shortening Results: " + byteValue + ", " + 
          intFromDouble);
    }
}
```

将提供的 Java 代码保存为名为`ValueConversionsExample.java`的文件。打开您的终端或命令提示符，导航到文件保存的目录。然后，使用以下命令编译代码：

```java
javac ValueConversionsExample.java
```

编译后，您可以使用`javap`命令反汇编字节码并显示相关部分。在终端或命令提示符中执行以下命令：

```java
javap -c ValueConversionsExample.class
```

在这次分析中，我们专注于字节码的特定段，以探索 Java 代码如何在 JVM 内部转换为可执行指令。我们的注意力集中在选定的字节码部分，揭示了 Java 编程领域中提升、精度考虑和缩短的复杂性。随着我们解读 JVM 的语言，提供了一幅描绘塑造 Java 字节码约定的视觉叙事。

```java
0: bipush        422: istore_1
3: iload_1
4: i2l
5: lstore_2
8: ldc           3.14
10: fstore_4
11: fload         4
13: dstore        5
17: ldc           32767
19: istore        7
21: iload         7
23: i2b
24: istore        8
27: ldc2_w        #2                  // double 1.7e308
34: dstore        9
36: dload         9
38: d2i
39: istore        11
```

在这段 Java 代码中，我们可以看到提升和缩短约定在实际中的应用。字节码片段专门关注与这些约定相关的指令，详细展示了 JVM 如何处理变量类型的扩展和缩短。

在我们探索 JVM 中的值转换时，我们剖析了字节码指令如何编排提升和考虑精度损失或溢出。这些复杂性凸显了 Java 编程中数据类型细微的舞蹈。随着本段的结束，将高级代码无缝转换为字节码的过程变得更加清晰，揭示了 JVM 的细致编排。在下一节中，我们将关注字节码中的对象操作领域，揭示编织 Java 面向对象范式的线索。在即将到来的旅程中，我们将审视塑造和操控对象的字节码指令，深入动态、多变的 Java 编程核心。

# 对象操作

在这次沉浸式体验中，我们开始全面探索 Java 字节码复杂结构中的对象操作。我们的旅程揭示了创建和操作实例、构建数组和访问类静态和实例属性的关键字节码指令。我们审视了从数组中加载值、保存到栈上、查询数组长度以及对实例或数组执行关键检查的指令。从基础的`new`指令到`multianewarray`的动态复杂性，每条字节码指令都推动我们更深入地进入面向对象操作领域。

在 Java 的字节码织锦中，`new`指令是通往对象创建和操作领域的门户。它不仅为对象分配内存，还调用其构造函数，启动动态实体的诞生。加入我们深入字节码复杂性的探索，看似简单的`new`指令揭示了将 Java 对象带入生命的根本步骤。随着我们对这条字节码指令的剖析，内存分配和构造函数调用的底层交响曲变得更加清晰，为在 JVM 中创建实例提供了更深入的理解。

+   **new**：创建一个新的对象，分配内存并调用对象的构造函数。新创建对象的引用放置在栈上。

在 Java 字节码的编排中，创建数组的命令呈现出一种多功能的织锦。在这一部分，我们深入探讨塑造数组的字节码指令，为数据存储提供了一个动态的画布。从为原始类型提供基础的`newarray`指令到为对象引用提供细微的`anewarray`指令，以及为多维数组提供的复杂`multianewarray`指令，每条字节码指令都为 JVM 内部的活跃数组生态系统做出了贡献。随着我们对这些命令的剖析，JVM 内部数组实例化的艺术性逐渐显现，为深入理解 Java 编程中的数据结构动态打开了大门。

+   **newarray**：创建一个新的原始类型数组

+   **anewarray**：创建对象引用的新数组

+   **multianewarray**：创建多维数组

在 Java 字节码的复杂舞蹈中，访问类静态或实例属性的指令——`getfield`、`putfield`、`getstatic` 和 `putstatic`——占据了中心舞台。从优雅地检索实例字段值到动态设置静态字段值，每条字节码指令都为面向对象编程的微妙舞蹈做出了贡献。加入我们，一起揭开字节码访问的优雅之处，其中实例和类属性之间的微妙平衡展开，揭示了在 JVM 中管理数据操作的底层机制。随着我们剖析这些指令，访问类属性的芭蕾舞生动起来，为深入理解 Java 编程中的面向对象复杂性铺平了道路。

+   **getfield**：从对象中检索实例字段的值

+   **putfield**：设置对象中实例字段的值

+   **getstatic**：从类中检索静态字段的值

+   **putstatic**：在类中设置静态字段的值

在 Java 字节码交响曲中，加载指令——`baload`、`caload`、`saload`、`iaload`、`laload`、`faload`、`daload` 和 `aaload`——占据了中心舞台，定义了从数组中检索值的舞蹈。在这一部分，我们沉浸在节奏感强烈的字节码命令中，优雅地将数组元素带到前台。从提取字节和字符到加载整数、长整型、浮点数、双精度数和对象引用，每条指令都在数组与 JVM 之间和谐的交互中扮演着关键角色。这些加载指令揭示了 Java 字节码在数组中无缝导航时展开的精心编排的芭蕾舞，展示了数组元素检索的灵活性和精确性。随着我们探索这些加载命令，从数组中加载值的复杂舞蹈生动起来，为深入了解 Java 编程的流体动力学提供了更深的洞察。

在 Java 字节码杰作中，保存指令——`bastore`、`castore`、`sastore`、`iastore`、`lastore`、`fastore`、`dastore` 和 `aastore`——巧妙地指挥着数组操作的画布。这些指令对于将值存储到不同类型的数组中至关重要。让我们通过示例深入了解它们的重要性：

+   **bastore**：将字节或布尔值存储到字节数组中

+   **castore**：将字符值存储到字符数组中

+   **sastore**：将短值存储到短数组中

+   **iastore**：将整数值存储到整型数组中

+   **lastore**：将长值存储到长数组中

+   **dastore**：将双精度值存储到双精度数组中

这些指令在数组操作中起着基本的作用，允许在数组中精确存储各种数据类型。

`arraylength`指令在 Java 字节码中充当指南针，通过提供数组的长度来引导开发者了解数组的度量：

+   **arraylength**: 获取数组的长度并将其推入栈。

在 Java 字节码的领域内，`instanceof`和`checkcast`指令充当着警惕的守护者，确保对象类型的完整性及其与指定类的对齐。虽然我们之前的探索深入到了数组操作，但现在让我们将焦点转移到这些指令在类型检查中的基本作用。`Instanceof`评估一个对象是否属于特定类，提供了关于对象类型的重要见解。另一方面，`checkcast`仔细检查并转换对象，确保它们与指定的类和谐对齐。这些字节码守护者共同在 JVM 中维护面向对象范式的健壮性和一致性中发挥着关键作用：

+   **instanceof**: 检查一个对象是否是特定类的实例

+   **checkcast**: 检查并将对象转换为指定的类，确保类型兼容性

这些字节码指令为在 Java 中操作对象提供了基础，允许创建、访问和修改实例和数组。无论是实例化新对象、处理数组、访问类属性还是执行动态检查，每条指令都为 Java 字节码中面向对象编程的灵活性和强大功能做出了贡献。理解这些指令是掌握 Java 对象操作复杂性的关键。

```java
Person class, focusing on a single attribute: name. This class encapsulates fundamental principles of object manipulation, featuring methods for accessing and modifying the attribute. As we navigate this example, we’ll delve into the bytecode generated from this code, offering insights into the low-level intricacies of object manipulation within the JVM. These bytecode instructions underpin the dynamic nature of Java programming, and this practical illustration will shed light on their real-world application:
```

```java
public class Person {    private String name;
    public Person(String name) {
        this.name = name;
    }
    public String getName() {
        return name;
    }
    public void setName(String newName) {
        this.name = newName;
    }
    public static void main(String[] args) {
        // Creating an instance of Person
        Person person = new Person("John");
        // Accessing and displaying the name attribute
        System.out.println("Original Name: " + person.getName());
        // Changing the name attribute
        person.setName("Alice");
        // Displaying the updated name
        System.out.println("Updated Name: " + person.getName());
    }
}
```

编译并显示字节码：

```java
javac Person.javajavap -c Person.class
```

让我们关注与对象操作相关的字节码的相关部分，包括对象创建（`new`）、属性访问（`getfield`、`putfield`）和方法调用：

```java
Compiled from "Person.java"public class Person {
  private java.lang.String name;
  public Person(java.lang.String);
```

代码：

```java
       0: aload_0       1: invokespecial #1                  // Method java/lang/
                                            Object."<init>":()V
       4: aload_0
       5: aload_1
       6: putfield      #2                  // Field name:Ljava/lang/
                                            String;
       9: return
  public java.lang.String getName();
```

代码：

```java
       0: aload_0       1: getfield      #2                  // Field name:Ljava/lang/
                                            String;
       4: areturn
  public void setName(java.lang.String);
```

代码：

```java
       0: aload_0       1: aload_1
       2: putfield      #2                  // Field name:Ljava/lang/
                                            String;
       5: return
  public static void main(java.lang.String[]);
```

代码：

```java
       0: new           #3                  // class Person       3: dup
       4: ldc           #4                  // String John
       6: invokespecial #5                  // Method "<init>":(Ljava/
                                            lang/String;)V
       9: astore_1
      10: getstatic     #6                  // Field java/lang/System.
                                            out:Ljava/io/PrintStream;
      13: ldc           #7                  // String Original Name:
      15: invokevirtual #8                  // Method java/io/
                             PrintStream.println:(Ljava/lang/String;)V
      18: getstatic     #6                  // Field java/lang/System.
                                            out:Ljava/io/PrintStream;
      21: aload_1
      22: invokevirtual #9                  // Method getName:()Ljava/
                                            lang/String;
      25: invokevirtual #8                  // Method java/io/
                             PrintStream.println:(Ljava/lang/String;)V
      28: aload_1
      29: ldc           #10                 // String Alice
      31: invokevirtual #11                 // Method setName:(Ljava/
                                            lang/String;)V
      34: getstatic     #6                  // Field java/lang/System.
                                            out:Ljava/io/PrintStream;
      37: ldc           #12                 // String Updated Name:
      39: invokevirtual #8                  // Method java/io/
                             PrintStream.println:(Ljava/lang/String;)V
      42: getstatic     #6                  // Field java/lang/System.
                                            out:Ljava/io/PrintStream;
      45: aload_1
      46: invokevirtual #9                  // Method getName:()Ljava/
                                            lang/String;
      49: invokevirtual #8                  // Method java/io/
                             PrintStream.println:(Ljava/lang/String;)V
      52: return
}
```

让我们分解关键的字节码指令：

+   对象创建（**new**）：

    +   **0: new #3**: 创建一个类型为**Person**的新对象

    +   **3: dup**: 在栈上复制对象引用

    +   **4: ldc #4**: 将常量字符串**"John"**推入栈

    +   **6: invokespecial #5**: 调用构造函数（**<init>**）以初始化对象

+   属性访问（**getfield, putfield**）：

    +   **1: getfield #2**: 获取**name**字段的值

    +   **2: putfield #2**: 设置**name**字段的值

+   方法调用：

    +   **22: invokevirtual #9**: 调用**getName**方法

    +   **31: invokevirtual #11**: 调用**setName**方法

这些字节码片段突出了与对象操作相关的根本指令，揭示了 Java 在底层字节码级别的动态特性。

在导航简化版`Person`类中对象操作的字节码织锦时，我们发现了管理对象创建、属性访问和方法调用的指令编排。随着字节码交响乐的展开，我们无缝过渡到下一个部分，我们将深入探索方法调用和返回的动态领域。加入我们，解读支撑方法调用的本质的字节码指令，揭示定义 JVM 中程序执行流程的复杂性的细节。随着我们前进，对方法调用和返回的探索将丰富我们对字节码交响乐的理解，揭示 Java 编程复杂性的下一层。

## 方法调用和返回

让我们开始一段旅程，深入探索 Java 编程领域中方法调用和值返回的复杂动态。我们将揭示动态调用方法和敏捷调用接口方法的微妙之处，并探讨调用私有或超类方法的独特和弦以及静态方法调用的强大音调。在这段探索中，我们将遇到动态构建的引入，展示了 Java 编程的适应性。记住，值返回的节奏由特定的指令定义。

探索 Java 字节码中方法调用的交响乐，以下指令在方法调用的旋律中演奏各种音调，每个都为语言的动态和多功能性做出了独特的贡献：

+   **invokevirtual:** 启动方法调用的旋律，这条指令从一个实例中调用方法，为 Java 中动态和多态行为提供支撑

+   **invokeinterface:** 添加一个和谐的音符，这条指令从一个接口中调用方法，为 Java 面向对象范式的灵活性和适应性做出贡献

+   **invokespecial:** 引入一个独特的和弦，这条指令调用一个私有或超类方法，封装了特权方法调用

+   **invokestatic:** 以强有力的音调，这条指令调用一个静态方法，强调了对不依赖于实例创建的方法的调用

+   **invokedynamic:** 这条指令演奏一曲多变的旋律，动态地构建一个对象，展示了 Java 方法调用的动态能力

方法执行节奏由返回指令（`ireturn`、`lreturn`、`freturn`、`dreturn` 和 `areturn`）补充，定义了从方法返回值的节奏。在异常中断的意外情况下，`athrow` 调用成为焦点，管理错误处理的编排。

我们进一步深入同步方法的复杂性，其中由 `ACC_SYNCHRONIZED` 标志标记的监视器，编排了一场受控的舞蹈。通过 `monitorenter` 指令，方法进入监视器，确保独占执行，并在完成后优雅地通过 `monitorexit` 退出，在字节码织锦中编织出同步交响曲。现在，让我们深入一个 Java 代码中方法调用和返回的现场演示。以下是一个简单的 Java 程序，通过方法调用执行计算。然后我们将仔细审查字节码，以解码这些方法调用的编排：

```java
public class MethodCallsExample {    public static void main(String[] args) {
        int result = performCalculation(5, 3);
        System.out.println("Result of calculation: " + result);
    }
    private static int performCalculation(int a, int b) {
        int sum = add(a, b);
        int product = multiply(a, b);
        return subtract(sum, product);
    }
    private static int add(int a, int b) {
        return a + b;
    }
    private static int multiply(int a, int b) {
        return a * b;
    }
    private static int subtract(int a, int b) {
        return a - b;
    }
}
```

编译并显示字节码：

```java
javac MethodCallsExample.javajavap -c MethodCallsExample.class
```

在字节码中，我们将关注与方法调用相关的指令（`invokevirtual`、`invokespecial`、`invokestatic`）以及返回指令（`ireturn`）。

以下是从 `MethodCallsExample.java` 编译的简化摘录：

```java
public class MethodCallsExample {  public MethodCallsExample();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/
                                            Object."<init>":()V
       4: return
  public static void main(java.lang.String[]);
```

代码：

```java
       0: iconst_5       1: iconst_3
       2: invokestatic  #2                  // Method 
                                            performCalculation:(II)I
       5: istore_1
       6: getstatic     #3                  // Field java/lang/System.
                                            out:Ljava/io/PrintStream;
       9: new           #4                  // class java/lang/
                                            StringBuilder
      12: dup
      13: ldc           #5                  // String Result of 
                                            calculation:
      15: invokespecial #6                  // Method java/lang/
                          StringBuilder."<init>":(Ljava/lang/String;)V
      18: iload_1
      19: invokevirtual #7                  // Method java/lang/
                     StringBuilder.append:(I)Ljava/lang/StringBuilder;
      22: invokevirtual #8                  // Method java/lang/
                           StringBuilder.toString:()Ljava/lang/String;
      25: invokevirtual #9                  // Method java/io/
                             PrintStream.println:(Ljava/lang/String;)V
      28: return
  private static int performCalculation(int, int);
```

代码：

```java
       0: iload_0       1: iload_1
       2: invokestatic  #2                  // Method 
                                            performCalculation:(II)I
       5: iload_0
       6: iload_1
       7: invokestatic  #11                 // Method multiply:(II)I
      10: invokestatic  #12                 // Method subtract:(II)I
      13: ireturn
  private static int add(int, int);
```

代码：

```java
       0: iload_0       1: iload_1
       2: iadd
       3: ireturn
  private static int multiply(int, int);
```

代码：

```java
       0: iload_0       1: iload_1
       2: imul
       3: ireturn
  private static int subtract(int, int);
```

代码：

```java
       0: iload_0       1: iload_1
       2: isub
       3: ireturn
}
```

这段字节码摘录展示了与方法和返回相关的基本指令，为提供的 Java 代码的字节码交响曲提供了一瞥。

当我们关闭对 Java 字节码中方法和返回探索的帷幕时，我们揭示了指令如何编排程序流程的复杂舞蹈。`invokevirtual`、`invokeinterface`、`invokespecial` 和 `invokestatic` 的交响曲在我们的字节码织锦中回响，展示了方法调用的动态性质和值的规律性返回。当我们转向下一节时，聚光灯转向条件指令，字节码决策塑造了程序执行的路径。加入我们解码条件语句的字节码复杂性，揭示引导 JVM 通过由条件定义的路径的逻辑，并继续我们深入 Java 编程复杂性的旅程。

# 条件指令

在本节中，我们深入探讨条件指令的微妙领域，揭示 JVM 内部决策的复杂性。这些指令构成了条件语句的骨架，引导 JVM 通过由布尔结果指定的路径。随着我们解码条件指令的字节码复杂性，揭示动态塑造 JVM 内部程序流程的逻辑，让我们一起来探索。

探索 Java 字节码的领域，揭示了在 JVM 内部控制条件逻辑的一系列指令。这些指令作为条件语句的建筑师，精确地执行决策并基于布尔结果影响程序流程。这次探索揭开了字节码复杂性的层层面纱，提供了关于 JVM 内部由这些基本指令塑造的动态路径的见解。

让我们探索一组具有独特控制程序流程和决策能力的字节码指令。这些指令包括一系列条件、switch 和跳转，每个都在指导 Java 程序执行路径中扮演着独特的角色：

+   **ifeq:** 如果栈顶的值等于 0，则跳转到目标指令

+   **ifne:** 如果栈顶的值不等于 0，则跳转到目标指令

+   **iflt:** 如果栈顶的值小于 0，则跳转到目标指令

+   **ifle:** 如果栈顶的值小于或等于 0，则跳转到目标指令

+   **ifgt:** 如果栈顶的值大于 0，则跳转到目标指令

+   **ifge:** 如果栈顶的值大于或等于 0，则跳转到目标指令

+   **ifnull:** 如果栈顶的值是 null，则跳转到目标指令

+   **ifnonnull:** 如果栈顶的值不是 null，则跳转到目标指令

+   **if_icmpeq:** 如果栈上的两个整数值相等，则跳转到目标指令

+   **if_icmpne:** 如果栈上的两个整数值不相等，则跳转到目标指令

+   **if_icmplt:** 如果栈上的第二个整数值小于第一个，则跳转到目标指令

+   **if_icmple:** 如果栈上的第二个整数值小于或等于第一个，则跳转到目标指令

+   **if_icmpgt:** 如果栈上的第二个整数值大于第一个，则跳转到目标指令

+   **if_icmpge:** 如果栈上的第二个整数值大于或等于第一个，则跳转到目标指令

+   **if_acmpeq:** 如果栈上的两个对象引用相等，则跳转到目标指令

+   **if_acmpne:** 如果栈上的两个对象引用不相等，则跳转到目标指令

+   **tableswitch:** 提供了一种更有效的方法来实现具有连续整数情况的 switch 语句

+   **lookupswitch:** 与**tableswitch**类似，但支持稀疏的 case 值

+   **goto:** 无条件地跳转到目标指令

+   **goto_w:** 无条件地跳转到目标指令（宽索引）

+   **jsr:** 跳转到子程序，并将返回地址保存在栈上

+   **jsr_w:** 跳转到子程序（宽索引），并将返回地址保存在栈上

+   **ret:** 使用之前**jsr**指令保存的返回地址从子程序返回

这些指令在构建条件语句和控制程序执行流程方面起着至关重要的作用。理解它们的行为对于解析和优化 Java 字节码至关重要。

```java
if_icmpne and if_acmpeq, responsible for steering the program through divergent paths. Join us in this exploration, unraveling the dynamic interplay of Java code and bytecode that shapes the outcomes of logical decisions within the JVM.
```

```java
public class ConditionalExample {    public static void main(String[] args) {
        int a = 5;
        int b = 3;
        if (a == b) {
            System.out.println("a is equal to b");
        } else {
            System.out.println("a is not equal to b");
        }
        String str1 = "Hello";
        String str2 = "Hello";
        if (str1.equals(str2)) {
            System.out.println("Strings are equal");
        } else {
            System.out.println("Strings are not equal");
        }
    }
}
```

编译并显示字节码：

```java
javac ConditionalExample.javajavap -c ConditionalExample.class
```

在字节码中，我们将关注条件指令，如`ifeq`、`ifne`和`if_acmpeq`，它们根据相等条件处理分支。以下是一个简化的摘录：

编译自 `ConditionalExample.java`

```java
public class ConditionalExample {  public ConditionalExample();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/
                                            Object."<init>":()V
       4: return
  public static void main(java.lang.String[]);
```

代码：

```java
       0: iconst_5       1: istore_1
       2: iconst_3
       3: istore_2
       4: iload_1
       5: iload_2
       6: if_icmpne     19
       9: getstatic     #2                  // Field java/lang/System.
                                            out:Ljava/io/PrintStream;
      12: ldc           #3                  // String a is equal to b
      14: invokevirtual #4                  // Method java/io/
                             PrintStream.println:(Ljava/lang/String;)V
      17: goto          32
      20: getstatic     #2                  // Field java/lang/System.
                                            out:Ljava/io/PrintStream;
      23: ldc           #5                  // String a is not equal 
                                            to b
      25: invokevirtual #4                  // Method java/io/
                             PrintStream.println:(Ljava/lang/String;)V
      28: goto          32
      31: astore_3
      32: ldc           #6                  // String Hello
      34: astore_3
      35: ldc           #6                  // String Hello
      37: astore        4
      39: aload_3
      40: aload         4
      42: if_acmpeq     55
      45: getstatic     #2                  // Field java/lang/System.
                                            out:Ljava/io/PrintStream;
      48: ldc           #7                  // String Strings are not 
                                            equal
      50: invokevirtual #4                  // Method java/io/
                             PrintStream.println:(Ljava/lang/String;)V
      53: goto          68
      56: getstatic     #2                  // Field java/lang/System.
                                            out:Ljava/io/PrintStream;
      59: ldc           #8                  // String Strings are equal
      61: invokevirtual #4                  // Method java/io/
                             PrintStream.println:(Ljava/lang/String;)V
      64: goto          68
      67: astore        5
      69: return
    [...]
}
```

这段字节码摘录展示了条件指令（`if_icmpne` 和 `if_acmpeq`）的实际应用，根据 Java 代码中指定的相等条件来指导程序流程。

在这次对 Java 字节码的探索中，我们解码了构成 JVM 内部逻辑的复杂条件指令的舞蹈。从基于相等的分支到无条件跳转，这些字节码命令指导了决策过程。随着我们结束关于 *条件指令* 的这一部分，视野变得更加开阔，引领我们进入旅程的下一阶段。接下来的部分深入到整个类的字节码表示，揭示封装 Java 程序本质的指令层。加入我们在这个过渡中，关注点从孤立的条件扩展到字节码中类的整体视图，照亮 Java 运行时环境的内部工作原理。

# 展示我字节码

随着我们继续探索 Java 字节码，我们现在将目光投向整个类，这是一次对 Java 程序二进制表示的全面深入研究。值得注意的是，我们检查的字节码可能因 JVM 版本和特定 JVM 供应商而异。在本节中，我们揭示编译和检查封装完整 Java 类本质的字节码的复杂性。从类初始化到方法实现，类的每个方面都在字节码中体现出来。让我们共同揭开 Java 程序的整体视图，探索我们的代码如何转换成 JVM 理解的语言。

在我们探索 Java 字节码的旅程中，让我们首先制作一个简单而通用的 `Animal` 类。以下定义该类的 Java 代码片段：

```java
public class Animal {    private String name;
    public String name() {
        return name;
    }
    public int age() {
        return 10;
    }
    public String bark() {
        return "woof";
    }
}
```

现在，让我们导航编译过程并深入了解字节码：

```java
javac Animal.javajavap -verbose Animal
```

有了这个，我们就开始了这次迷人的探索，编译我们的 Java 类，揭示表面下的字节码复杂性。加入我们，解码 JVM 的语言，并照亮我们的 `Animal` 类的字节码表示。

这部分字节码输出提供了关于编译后的类文件的元数据。让我们分析关键信息：

```java
Last modified Nov 16, 2023; size 433 bytes  SHA-256 checksum   0f087fdc313e02a8307d47242cb9021672ca110932ffe9ba89ae313a4f963da7
  Compiled from "Animal.java"
public class Animal
  minor version: 0
  major version: 65
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER
  this_class: #8                          // Animal
  super_class: #2                         // java/lang/Object
  interfaces: 0, fields: 2, methods: 4, attributes: 1
```

+   **最后修改**：表示对类文件的最后修改日期；在本例中是 **2023 年 11 月 16 日**。

+   **大小**：指定类文件的字节数，本例中为 **433** 字节。

+   **SHA-256 校验和**：代表类文件的 SHA-256 校验和。这个 **校验和** 作为文件的唯一标识符，并确保其完整性。

+   **从 "Animal.java" 编译**：告知我们此字节码是从源文件 **Animal.java** 编译的。

+   **类声明**：声明名为 **Animal** 的类。

+   **版本信息**：

    +   **次要版本**：设置为 0。

    +   **主要版本**：设置为 65，表示与 Java 11 兼容。

+   **Flags****:** 显示表示应用于类的访问控制修饰符的十六进制标志。在这种情况下，它是一个公开类（**ACC_PUBLIC**）并具有附加属性（**ACC_SUPER**）。

+   **Class hierarchy:**

    +   **this_class**: 指向常量池索引（**#8**）表示当前类，即**Animal**。

    +   **super_class**: 指向常量池索引（**#2**）表示超类，即**java/lang/Object**。

+   **Interfaces, fields, methods, and attributes****:** 提供了这些元素在类中的计数。

这条元数据提供了类文件属性的快照，包括其版本、访问修饰符和结构细节。

字节码输出的`Constant pool`部分提供了对常量池的洞察，常量池是一个用于存储各种常量的结构表，例如字符串、方法和方法引用、类名等。让我们解析这个常量池中的条目：

```java
Constant pool:   #1 = Methodref          #2.#3            // java/lang/
                                            Object."<init>":()V
   #2 = Class              #4               // java/lang/Object
   #3 = NameAndType        #5:#6            // "<init>":()V
   #4 = UTF-8              java/lang/Object
   #5 = UTF-8              <init>
   #6 = UTF-8              ()V
   #7 = Fieldref           #8.#9            // Animal.name:Ljava/lang/
                                            String;
   #8 = Class              #10              // Animal
   #9 = NameAndType        #11:#12          // name:Ljava/lang/String;
  #10 = UTF-8              Animal
  #11 = UTF-8              name
  #12 = UTF-8              Ljava/lang/String;
  #13 = Fieldref           #8.#14           // Animal.age:I
  #14 = NameAndType        #15:#16          // age:I
  #15 = UTF-8              age
  #16 = UTF-8              I
  #17 = String             #18              // woof
  #18 = UTF-8              woof
  #19 = UTF-8              Code
  #20 = UTF-8              LineNumberTable
  #21 = UTF-8              ()Ljava/lang/String;
  #22 = UTF-8              ()I
  #23 = UTF-8              bark
  #24 = UTF-8              SourceFile
  #25 = UTF-8              Animal.java
```

这将显示字段引用位置：

+   对**Object**类的构造函数的引用：

    +   **#1 = Methodref #2.#3 //** **java/lang/Object."<init>":()V**

        +   此条目引用了**java/lang/Object**类的构造函数，表示为**<init>**。它表示每个类从**Object**类隐式继承的初始化方法。

+   **Object**类的类引用：

    +   **#2 = Class #4 //** **java/lang/Object**

        +   指向**java/lang/Object**类的类引用，表示**Animal**类扩展了**Object**。

+   **Object**类的构造函数的名称和类型：

    +   **#3 = NameAndType #5:#6 // "<****init>":()V**

        +   指定无参数且返回 void 的构造函数（**<init>**）的名称和类型。

+   **Object**类名称的 UTF-8 条目：

    +   **#4 =** **UTF-8 java/lang/Object**

        +   表示**java/lang/Object**类名称的 UTF-8 编码。

+   构造函数和参数类型的 UTF-8 条目：

    +   **#5 =** **UTF-8 <init>**

    +   **#6 =** **UTF-8 ()V**

        +   表示构造函数的名称（**<init>**）及其类型（无参数且返回 void）的 UTF-8 编码。

+   对**Animal**类的**name**字段的字段引用：

    +   **#7 = Fieldref #8.#9 //** **Animal.name:Ljava/lang/String;**

        +   指向**Animal**类中的**name**字段，其类型为**java/lang/String**。

+   **Animal**类的类引用：

    +   **#8 = Class #10 //** **Animal**

        +   指向**Animal**类的类引用。

+   **name**字段的名称和类型：

    +   **#9 = NameAndType #11:#12 //** **name:Ljava/lang/String;**

        +   指定**name**字段的名称和类型：其名称（**name**）和类型（String）。

+   **Animal**类名称的 UTF-8 条目：

    +   **#10 =** **UTF-8 Animal**

        +   表示**Animal**类名称的 UTF-8 编码。

+   **name**字段的 UTF-8 条目：

    +   **#11 =** **UTF-8 name**

    +   **#12 =** **UTF-8 Ljava/lang/String;**

        +   表示**name**字段的名称及其类型（String）的 UTF-8 编码。

对于 `age` 字段和 `bark` 方法，存在类似的条目，引用了字段和方法名称、它们的类型以及常量池中的类名。总的来说，常量池是字节码执行期间解析符号引用的关键组件。

提供的字节码片段代表了 `Animal` 类中的方法。让我们逐一分析每个方法：

+   构造方法（**public Animal();**）:

    +   **描述符**: **()V**（无参数，返回 void）

    +   **标志**: **ACC_PUBLIC**（公共方法）

    +   **代码**:

        ```java
        stack=1, locals=1, args_size=1   0: aload_0   1: invokespecial #1 // Method java/lang/Object."<init>":()V   4: return
        ```

    此构造函数通过调用其超类（`Object`）的构造函数来初始化 `Animal` 对象。`aload_0` 指令将对象引用（`this`）加载到栈上，`invokespecial` 调用超类构造函数。`LineNumberTable` 指示此代码对应源文件中的第 `1` 行。

+   方法 **public** **java.lang.String name();**:

    +   **描述符**: **()Ljava/lang/String;**（无参数，返回 **String**）

    +   **标志**: **ACC_PUBLIC**（公共方法）

    +   **代码**:

        ```java
        stack=1, locals=1, args_size=1   0: aload_0   1: getfield #7 // Field name:Ljava/lang/String;   4: areturn
        ```

    方法 `name()` 获取 `name` 字段的值并返回它。`aload_0` 加载对象引用（`this`），`getfield` 获取 `name` 字段的值。`LineNumberTable` 指示此代码对应源文件中的第 `9` 行。

+   方法 **public** **int age();**:

    +   **描述符**: **()I**（无参数，返回 int）

    +   **标志**: **ACC_PUBLIC**（公共方法）

    +   **代码**:

        ```java
        stack=1, locals=1, args_size=1   0: aload_0   1: getfield #13 // Field age:I   4: ireturn
        ```

    与 `name` 方法类似，它检索 `age` 字段的值并返回它。`getfield` 获取值，`ireturn` 返回它。`LineNumberTable` 指示此代码对应源文件中的第 `13` 行。

+   方法 **public** **java.lang.String bark();**:

    +   **描述符**: **()Ljava/lang/String;**（无参数，返回 **String**）

    +   **标志**: **ACC_PUBLIC**（公共方法）

    +   **代码**:

        ```java
        stack=1, locals=1, args_size=1   0: ldc #17 // String woof   2: areturn
        ```

    `bark()` 方法直接返回字符串 `woof`，而不访问任何字段。`ldc` 加载一个常量字符串，`areturn` 返回它。`LineNumberTable` 指示此代码对应源文件中的第 `17` 行。

这些字节码片段封装了 `Animal` 类中每个方法的逻辑，展示了方法执行期间的低级操作。

在 Java 字节码中，每个变量和方法参数都被分配一个类型描述符，以指示其数据类型。这些描述符是用于传达变量类型或参数信息的紧凑表示。以下是一个详细说明：

+   **B**（**byte**）：表示一个有符号的 8 位整数

+   **C**（**char**）：表示一个 Unicode 字符

+   **D**（**double**）：表示双精度浮点值

+   **F**（**float**）：表示单精度浮点值

+   **I**（**int**）：表示一个 32 位整数

+   **J**（**long**）：表示一个 64 位长整数

+   **L Classname**（**引用**）：指向指定类的实例；完全限定类名跟在 **L** 后面，并以分号结尾

+   **S** (**short**): 表示一个 16 位的短整数

+   **Z** (**Boolean**): 表示一个布尔值（true 或 false）

+   **[** (**数组引用**): 表示一个数组。数组元素的类型由**[**后面的附加字符确定

对于数组引用：

+   **[L Classname**: 表示指定类的对象数组

+   **[[B**: 表示一个字节数组的二维数组

这些类型描述符在检查与 Java 类中的方法声明、字段定义和变量使用相关的字节码指令时至关重要。它们使得在 Java 程序的低级字节码表示中简洁地表示数据类型成为可能。

# 摘要

随着我们结束对 Java 字节码及其复杂条件指令的探索，读者已经对 Java 程序中的细微控制流有了坚实的理解。凭借对字节码决策能力的了解，读者已经为优化代码以实现效率和精度做好了准备。现在，我们的旅程将我们带入 JVM 的核心，重点关注即将到来的章节中的执行引擎。读者可以期待深入了解字节码解释的机制和**即时编译**（**JIT**）的变革领域。这些技能在现实生活中的工作场所中非常有价值，因为在性能上优化 Java 应用程序是一项关键任务。加入我们，揭开 JVM 执行引擎的秘密，在那里，字节码的二进制舞蹈演变为优化的机器指令，赋予读者增强 Java 应用程序运行时魔力的能力。

# 问题

回答以下问题以测试你对本章知识的掌握：

1.  哪个字节码指令用于比较两个整数是否相等，并相应地分支？

    1.  **ifeq**

    1.  **if_icmpeq**

    1.  **if_acmpeq**

    1.  **tableswitch**

1.  字节码指令**ifeq**做什么？

    1.  如果栈顶的值等于 0，则分支

    1.  如果栈上的两个整数相等，则分支

    1.  跳转到子程序

    1.  从数组中加载一个整数

1.  用于无条件分支的字节码指令是什么？

    1.  **goto**

    1.  **ifne**

    1.  **jsr_w**

    1.  **lookupswitch**

1.  在 Java 字节码中，**jsr**指令做什么？

    1.  跳转到子程序

    1.  调用一个静态方法

    1.  比较两个双精度浮点数

    1.  如果栈顶的值是 null，则分支

1.  哪个字节码指令用于检查两个对象引用是否不相等，并跳转到目标指令？

    1.  **if_acmpeq**

    1.  **if_acmpne**

    1.  **ifnull**

    1.  **goto_w**

# 答案

这里是本章问题的答案：

1.  B. **if_icmpeq**

1.  A. 如果栈顶的值等于 0，则分支

1.  A. **goto**

1.  A. 跳转到子程序

1.  B. **if_acmpne**
