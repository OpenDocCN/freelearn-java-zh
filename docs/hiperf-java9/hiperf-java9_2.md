# 第二章。提高生产力和加快应用程序的工具

```java
/list), and /-<n> allow re-running of the snippets that have been run previously.
JShell was able to provide the suggestion because the JAR file with the compiled Pair class was on the classpath (set there by default as part of JDK libraries). You can also add to the classpath any other JAR file with the compiled classes you need for your coding. You can do it by setting it at JShell startup by the option --class-path (can be also used with one dash -class-path):
```

![创建 JShell 会话和设置上下文](img/02_07.jpg)

```java
Shift + *Tab* and then *I* as described earlier.
`<name or id>`: This is the name or ID of a specific snippet or method or type or variable (we will see examples later)`-start`: This shows snippets or methods or types or variables loaded at the JShell start (we will see later how to do it)`-all`: This shows snippets or methods or types or variables loaded at the JShell start and entered later during the session
```

默认情况下，在启动时导入了几个常见的包。您可以通过键入`/l -start`或`/l -all`命令来查看它们：

![JShell 命令](img/02_12.jpg)

```java
/l s5command, for example, it will retrieve the snippet with ID s5:
```

![JShell 命令](img/02_13.jpg)

```java
pair), saved the session entries in the file mysession.jsh (in the home directory), and closed the session. Let's look in the file mysession.jsh now:
```

![JShell 命令](img/02_15.jpg)

```java
7:
```

![JShell 命令](img/02_29.jpg)

```java
/o <file> that opens the file as the source input.
```

命令`/en`、`/res`和`/rel`具有重叠的功能：

+   `/en [options]`：这允许查看或更改评估上下文

+   `/res [options]`：这将丢弃所有输入的片段并重新启动会话

+   `/rel[options]`：这将重新加载会话，与命令`/en`的方式相同

有关更多详细信息和可能的选项，请参阅官方 Oracle 文档（[`docs.oracle.com/javase/9/tools/jshell.htm`](http://docs.oracle.com/javase/9/tools/jshell.htm)）。

命令`[/se [setting]`设置配置信息，包括外部编辑器、启动设置和反馈模式。此命令还用于创建具有自定义提示、格式和截断值的自定义反馈模式。如果未输入任何设置，则显示编辑器、启动设置和反馈模式的当前设置。前面提到的文档详细描述了所有可能的设置。

JShell 在 IDE 内部集成后将变得更加有用，这样程序员就可以实时评估表达式，甚至更好的是，它们可以像编译器今天评估语法一样自动评估。

# 提前（AOT）

Java 的一个重要宣称是一次编写，到处运行。这是通过为几乎所有平台创建**Java Runtime Environment**（**JRE**）的实现来实现的，因此通过 Java 编译器（`javac`工具）从源代码生成的字节码可以在安装了 JRE 的任何地方执行，前提是编译器`javac`的版本与 JRE 的版本兼容。

JRE 的最初版本主要是字节码的解释器，性能比一些其他语言和它们的编译器（如 C 和 C++）要慢。然而，随着时间的推移，JRE 得到了大幅改进，现在产生的结果相当不错，与许多其他流行的系统一样。在很大程度上，这要归功于 JIT 动态编译器，它将最常用的方法的字节码转换为本机代码。一旦生成，编译后的方法（特定于平台的机器代码）将根据需要执行，而无需任何解释，从而减少执行时间。

为了利用这种方法，JRE 需要一些时间来找出应用程序中最常用的方法。在这个编程领域工作的人称之为热方法。这种发现期直到达到最佳性能通常被称为 JVM 的预热时间。对于更大更复杂的 Java 应用程序，这个时间更长，对于较小的应用程序可能只有几秒钟。然而，即使在达到最佳性能之后，由于特定输入的原因，应用程序可能会开始利用以前从未使用过的执行路径，并调用尚未编译的方法，从而突然降低性能。当代码尚未编译的部分属于在某些罕见的关键情况下调用的复杂过程时，这可能尤为重要，这正是需要最佳性能的时候。

自然的解决方案是允许程序员决定应用程序的哪些组件必须预编译成本机机器代码--那些经常使用的（从而减少应用程序的预热时间），以及那些不经常使用但必须尽快执行的（以支持关键情况和整体稳定性能）。这就是**Java Enhancement ProposalJEP 295: Ahead-of-Time Compilation**的动机：

JIT 编译器速度快，但 Java 程序可能变得非常庞大，以至于 JIT 完全预热需要很长时间。很少使用的 Java 方法可能根本不会被编译，可能因为重复的解释调用而导致性能下降。

值得注意的是，即使在 JIT 编译器中，也可以通过设置编译阈值来减少预热时间--一个方法被调用多少次后才将其编译成本机代码。默认情况下，这个数字是 1500。因此，如果我们将其设置为小于这个值，预热时间将会更短。可以使用`java`工具的`-XX:CompileThreshold`选项来实现。例如，我们可以将阈值设置为 500，如下所示（其中`Test`是具有`main()`方法的编译过的 Java 类）：

```java
java -XX:CompileThreshold=500 -XX:-TieredCompilation Test
```

添加`-XX:-TieredCompilation`选项以禁用分层编译，因为它默认启用并且不遵守编译阈值。可能的缺点是 500 的阈值可能太低，太多的方法将被编译，从而降低性能并增加预热时间。这个选项的最佳值将因应用程序而异，并且甚至可能取决于相同应用程序的特定数据输入。

## 静态与动态编译

许多高级编程语言，如 C 或 C++，从一开始就使用 AOT 编译。它们也被称为**静态编译**语言。由于 AOT（或静态）编译器不受性能要求的限制（至少不像运行时的解释器，也称为**动态编译器**），它们可以花费时间产生复杂的代码优化。另一方面，静态编译器没有运行时（分析）数据，这在动态类型语言的情况下尤其受限，Java 就是其中之一。由于 Java 中的动态类型能力--向子类型进行下转换，查询对象的类型以及其他类型操作--是面向对象编程的支柱之一（多态原则），Java 的 AOT 编译变得更加受限。Lambda 表达式对静态编译提出了另一个挑战，目前还不支持。

动态编译器的另一个优点是它可以做出假设并相应地优化代码。如果假设被证明是错误的，编译器可以尝试另一个假设，直到达到性能目标。这样的过程可能会减慢应用程序的速度和/或增加预热时间，但从长远来看，可能会导致更好的性能。基于配置文件的优化也可以帮助静态编译器沿着这条道路前进，但与动态编译器相比，它在优化的机会上始终受到限制。

尽管如此，我们不应该感到惊讶，JDK 9 中当前的 AOT 实现是实验性的且受限的，目前仅适用于 64 位 Linux 系统，支持并行或 G1 垃圾回收，并且唯一支持的模块是`java.base`。此外，AOT 编译应该在执行生成的机器代码的相同系统或具有相同配置的系统上执行。然而，尽管如此，JEP 295 指出：

性能测试显示，一些应用程序受益于 AOT 编译的代码，而其他一些明显显示出退化。

值得注意的是，AOT 编译在**Java Micro Edition**（**ME**）中长期得到支持，但在**Java Standard Edition**（**SE**）中 AOT 的更多用例尚待确定，这是实验性 AOT 实现随 JDK 9 发布的原因之一--以便促进社区尝试并反馈实际需求。

## AOT 命令和程序

JDK 9 中的底层 AOT 编译基于 Oracle 项目`Graal`，这是一个在 JDK 8 中引入的开源编译器，旨在改进 Java 动态编译器的性能。 AOT 组不得不对其进行修改，主要是围绕常量处理和优化。他们还添加了概率性分析和特殊的内联策略，从而使 Grall 更适合静态编译。

除了现有的编译工具`javac`之外，JDK 9 安装中还包括一个新的`jaotc`工具。使用`libelf`库生成 AOT 共享库`.so`，这是将来版本中将要删除的依赖项。

要开始 AOT 编译，用户必须启动`jaotc`并指定要编译的类、JAR 文件或模块。还可以将输出库的名称（保存生成的机器代码）作为`jaotc`参数传递。如果未指定，默认输出的名称将为`unnamed.so`。例如，让我们看看 AOT 编译器如何与类`HelloWorld`一起工作：

```java
public class HelloWorld {
   public static void main(String... args) {
       System.out.println("Hello, World!");
   }
}
```

首先，我们将使用`javac`生成字节码并生成`HelloWorld.class`：

```java
javac HelloWorld.java
```

然后，我们将使用文件`HelloWorld.class`中的字节码生成库`libHelloWorld.so`中的机器代码：

```java
jaotc --output libHelloWorld.so HelloWorld.class
```

现在，我们可以使用`java`工具执行生成的库（在与执行`jaotc`的平台规格相同的平台上），并使用`-XX:AOTLibrary`选项：

```java
java -XX:AOTLibrary=./libHelloWorld.so HelloWorld
```

选项`-XX:AOTLibrary`允许我们列出用逗号分隔的多个 AOT 库。

请注意，`java`工具除了一些组件的本机代码外，还需要所有应用程序的字节码。这一事实减少了一些 AOT 爱好者声称的静态编译的所谓优势，即它更好地保护代码免受反编译。如果相同的类或方法已经在 AOT 库中，未来当字节码在运行时不再需要时，这可能是真的。但是，就目前而言，情况并非如此。

要查看是否使用了 AOT 编译的方法，可以添加一个`-XX:+PrintAOT`选项：

```java
java -XX:AOTLibrary=./libHelloWorld.so -XX:+PrintAOT HelloWorld
```

它将允许您在输出中看到加载的行`./libHelloWorld.so` AOT 库。

如果类的源代码已更改但未通过`jaotc`工具推送到 AOT 库中，JVM 将在运行时注意到，因为每个编译类的指纹都与其在 AOT 库中的本机代码一起存储。 JIT 然后将忽略 AOT 库中的代码，而使用字节码。

JDK 9 中的`java`工具支持与 AOT 相关的其他几个标志和选项：

+   `-XX:+/-UseAOT`告诉 JVM 使用或忽略 AOT 编译的文件（默认情况下，设置为使用 AOT）

+   `-XX:+/-UseAOTStrictLoading`打开/关闭 AOT 严格加载；如果打开，它指示 JVM 在任何 AOT 库是在与当前运行时配置不同的平台上生成的时退出

JEP 295 描述了`jaotc`工具的命令格式如下：

```java
jaotc <options> <name or list>
```

`name`是类名或 JAR 文件。`list`是一个以冒号`:`分隔的类名、模块、JAR 文件或包含类文件的目录列表。`options`是以下列表中的一个或多个标志：

+   `--output <file>`：这是输出文件名（默认情况下为`unnamed.so`）

+   `--class-name <class names>`：这是要编译的 Java 类列表

+   --jar <jar files>：这是要编译的 JAR 文件列表

+   `--module <modules>`：这是要编译的 Java 模块列表

+   `--directory <dirs>`：这是您可以搜索要编译的文件的目录列表

+   `--search-path <dirs>`：这是要搜索指定文件的目录列表

+   `--compile-commands <file>`：这是带有编译命令的文件名；这是一个例子：

```java
exclude sun.util.resources..*.TimeZoneNames_.*.getContents\(\)\[\[Ljava/lang/Object;
exclude sun.security.ssl.*
compileOnly java.lang.String.*

```

AOT 目前识别两个编译命令：

+   `exclude`：这将排除指定方法的编译

+   `compileOnly`：这只编译指定的方法

正则表达式用于指定这里提到的类和方法：

+   --compile-for-tiered：这为分层编译生成了分析代码（默认情况下，不会生成分析代码）

+   --compile-with-assertions：这生成带有 Java 断言的代码（默认情况下，不会生成断言代码）

+   --compile-threads <number>：这是要使用的编译线程数（默认情况下，使用较小值 16 和可用 CPU 的数量）

+   --ignore-errors：这忽略在类加载期间抛出的所有异常（默认情况下，如果类加载抛出异常，则在编译时退出）

+   --exit-on-error：这在编译错误时退出（默认情况下，跳过编译失败，而其他方法的编译继续）

+   --info：这打印有关编译阶段的信息

+   --verbose：这打印有关编译阶段的更多细节

+   --debug：这打印更多细节

+   --help：这打印帮助信息

+   --version：这打印版本信息

+   -J<flag>：这将一个标志直接传递给 JVM 运行时系统

正如我们已经提到的，一些应用程序可以通过 AOT 来提高性能，而其他一些可能会变慢。只有测试才能对每个应用程序的 AOT 的有用性问题提供明确的答案。无论如何，改善性能的一种方法是编译和使用`java.base`模块的 AOT 库：

```java
jaotc --output libjava.base.so --module java.base
```

在运行时，AOT 初始化代码在`$JAVA_HOME/lib`目录中查找共享库，或者在`-XX:AOTLibrary`选项列出的库中查找。如果找到共享库，则会被选中并使用。如果找不到共享库，则 AOT 将被关闭。

# 总结

在本课程中，我们描述了两个新工具，可以帮助开发人员更加高效（JShell 工具）并帮助改善 Java 应用程序的性能（`jaotc`工具）。使用它们的示例和步骤将帮助您了解其使用的好处，并在您决定尝试它们的情况下帮助您入门。

在下一课中，我们将讨论如何使用命令行工具以编程方式监视 Java 应用程序。我们还将探讨如何通过多线程来改善应用程序性能，以及在通过监视了解瓶颈后如何调整 JVM 本身。

# 评估

1.  ______ 编译器接受 Java 字节码并生成本机机器代码，以便生成的二进制文件可以在本机上执行。

1.  以下哪个命令丢弃了一个由名称或 ID 引用的片段？

1.  /d <name or id>

1.  /drop <name or id>

1.  /dr <name or id>

1.  /dp <name or id>

1.  判断真假：Shell 是一种著名的 Ahead-of-Time 工具，适用于那些使用 Scala、Ruby 编程的人。它接受用户输入，对其进行评估，并在一段时间后返回结果。

1.  以下哪个命令用于列出您在 JShell 中键入的源代码？

1.  /l [<name or id>|-all|-start]

1.  /m [<name or id>|-all|-start]L

1.  /t [<name or id>|-all|-start]

1.  /v [<name or id>|-all|-start]

1.  以下哪个正则表达式忽略在类加载期间抛出的所有异常？

1.  --exit-on-error

1.  –ignores-errors

1.  --ignore-errors

1.  --exits-on-error
