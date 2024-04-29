# 安装和预览 Java 11

在本章中，我们将介绍以下内容：

+   在 Windows 上安装 JDK 18.9 并设置 PATH 变量

+   在 Linux（Ubuntu，x64）上安装 JDK 18.9 并配置 PATH 变量

+   编译和运行 Java 应用程序

+   JDK 18.9 的新功能

+   使用应用程序类数据共享

# 介绍

学习编程语言的每一个探索都始于设置环境以进行学习实验。为了与这一理念保持一致，在本章中，我们将向您展示如何设置开发环境，然后运行一个简单的模块化应用程序来测试我们的安装。之后，我们将向您介绍 JDK 18.9 中的新功能和工具。然后，我们将比较 JDK 9、18.3 和 18.9。我们将以 JDK 18.3 中引入的允许应用程序类数据共享的新功能结束本章。

# 在 Windows 上安装 JDK 18.9 并设置 PATH 变量

在本教程中，我们将介绍如何在 Windows 上安装 JDK 以及如何设置`PATH`变量，以便能够在命令行中的任何位置访问 Java 可执行文件（如`javac`、`java`和`jar`）。

# 如何做...

1.  访问[`jdk.java.net/11/`](http://jdk.java.net/11/)并接受早期采用者许可协议，它看起来像这样：

![](img/7f7dc193-5003-47d7-bd7e-fb5fcab5076c.png)

1.  接受许可协议后，您将获得一个基于操作系统和架构（32/64 位）的可用 JDK 捆绑包的网格。单击下载适用于您的 Windows 平台的相关 JDK 可执行文件（.exe）。

1.  运行 JDK 可执行文件（.exe）并按照屏幕上的说明在系统上安装 JDK。

1.  如果您在安装过程中选择了所有默认设置，您将在 64 位的`C:/Program Files/Java`和 32 位的`C:/Program Files (x86)/Java`找到安装的 JDK。

既然我们已经安装了 JDK，让我们看看如何设置`PATH`变量。

JDK 提供的工具，即`javac`、`java`、`jconsole`和`jlink`，都位于 JDK 安装的 bin 目录中。您可以通过两种方式从命令提示符中运行这些工具：

1.  导航到安装工具的目录并运行它们，如下所示：

```java
 cd "C:\Program Files\Java\jdk-11\bin"
      javac -version
```

1.  导出路径到目录，以便在命令提示符中的任何目录中都可以使用工具。为了实现这一点，我们必须将 JDK 工具的路径添加到`PATH`环境变量中。命令提示符将在`PATH`环境变量中声明的所有位置中搜索相关工具。

让我们看看如何将 JDK 的 bin 目录添加到`PATH`变量中：

1.  右键单击“我的电脑”，然后单击“属性”。您将看到系统信息。搜索“高级系统设置”，单击它以获得一个窗口，如下面的屏幕截图所示：

![](img/761194d9-a941-4080-864e-860966d3d79e.png)

1.  单击“环境变量”以查看系统中定义的变量。您会看到已经定义了相当多的环境变量，如下面的屏幕截图所示（变量将在不同系统之间有所不同；在下面的屏幕截图中，有一些预定义的变量和一些我添加的变量）：

![](img/ef3fbf36-9efa-4da2-9d6f-2d6952ce7015.png)

在“系统变量”下定义的变量可供系统的所有用户使用，而在“用户变量”下定义的变量仅供特定用户使用。

1.  一个新变量，名为`JAVA_HOME`，其值为 JDK 9 安装的位置。例如，它将是`C:\Program Files\Java\jdk-11`（64 位）或`C:\Program Files (x86)\Java\jdk-11`（32 位）：

![](img/82b23ee9-7eb9-45d6-9893-92849060dbd1.png)

1.  使用 JDK 安装的 bin 目录的位置（在`JAVA_HOME`环境变量中定义）更新`PATH`环境变量。如果您已经在列表中看到了`PATH`变量定义，那么您需要选择该变量并点击编辑。如果没有看到`PATH`变量，则点击新建。

1.  在上一步中的任何操作都会弹出一个窗口，如下面的屏幕截图所示（在 Windows 10 上）：

![](img/2e7ae014-01ed-45e7-84aa-09cb0ec41743.png)

下面的屏幕截图显示了其他 Windows 版本：

![](img/477d1f7a-76ac-485e-911b-a73cc7fd0e37.png)

1.  您可以在第一个屏幕截图中单击新建并插入`%JAVA_HOME%\bin`值，或者通过在变量值字段中添加`; %JAVA_HOME%\bin`来追加值。在 Windows 中，分号(`;`)用于分隔给定变量名的多个值。

1.  设置完值后，打开命令提示符并运行`javac -version`。您应该能够看到`javac 11-ea`作为输出。如果您没有看到它，这意味着您的 JDK 安装的 bin 目录没有正确添加到`PATH`变量中。

# 在 Linux（Ubuntu，x64）上安装 JDK 18.9 并配置 PATH 变量

在这个示例中，我们将看看如何在 Linux（Ubuntu，x64）上安装 JDK，并如何配置`PATH`变量以使终端中的 JDK 工具（如`javac`、`java`和`jar`）可以从任何位置使用。

# 如何做...

1.  按照*在 Windows 上安装 JDK 18.9 并设置 PATH 变量*的步骤 1 和 2 来到达下载页面。

1.  从下载页面上复制 Linux x64 平台的 JDK 的下载链接（`tar.gz`）。

1.  使用`$> wget <copied link>`下载 JDK，例如，`$> wget https://download.java.net/java/early_access/jdk11/26/BCL/jdk-11-ea+26_linux-x64_bin.tar.gz`。

1.  下载完成后，您应该有相关的 JDK 可用，例如，`jdk-11-ea+26_linux-x64_bin.tar.gz`。您可以使用`$> tar -tf jdk-11-ea+26_linux-x64_bin.tar.gz`列出内容。您甚至可以使用`$> tar -tf jdk-11-ea+26_linux-x64_bin.tar.gz | more`将其传输到`more`来分页输出。

1.  使用`$> tar -xvzf jdk-11-ea+26_linux-x64_bin.tar.gz -C /usr/lib`在`/usr/lib`下提取`tar.gz`文件的内容。这将把内容提取到一个目录`/usr/lib/jdk-11`中。然后，您可以使用`$> ls /usr/lib/jdk-11`列出 JDK 11 的内容。

1.  通过编辑 Linux 主目录中的`.bash_aliases`文件来更新`JAVA_HOME`和`PATH`变量：

```java
 $> vim ~/.bash_aliases
      export JAVA_HOME=/usr/lib/jdk-11
      export PATH=$PATH:$JAVA_HOME/bin
```

源`.bashrc`文件以应用新的别名：

```java
 $> source ~/.bashrc
      $> echo $JAVA_HOME
      /usr/lib/jdk-11
      $>javac -version
      javac 11-ea
      $> java -version
      java version "11-ea" 2018-09-25
 Java(TM) SE Runtime Environment 18.9 (build 11-ea+22)
 Java HotSpot(TM) 64-Bit Server VM 18.9 (build 11-ea+22, mixed 
      mode)
```

本书中的所有示例都是在 Linux（Ubuntu，x64）上安装的 JDK 上运行的，除非我们特别提到这些是在 Windows 上运行的地方。我们已经尝试为两个平台提供运行脚本。

# 编译和运行 Java 应用程序

在这个示例中，我们将编写一个非常简单的模块化`Hello world`程序来测试我们的 JDK 安装。这个简单的示例在 XML 中打印`Hello world`；毕竟，这是 Web 服务的世界。

# 准备工作

您应该已经安装了 JDK 并更新了`PATH`变量以指向 JDK 安装位置。

# 如何做...

1.  让我们使用相关属性和注释定义模型对象，这些属性和注释将被序列化为 XML：

```java
        @XmlRootElement
        @XmlAccessorType(XmlAccessType.FIELD) 
        class Messages{     
          @XmlElement 
          public final String message = "Hello World in XML"; 
        }
```

在上面的代码中，`@XmlRootElement`用于定义根标签，`@XmlAccessorType`用于定义标签名称和标签值的来源类型，`@XmlElement`用于标识成为 XML 中标签名称和标签值的来源。

1.  让我们使用 JAXB 将`Message`类的一个实例序列化为 XML：

```java
public class HelloWorldXml{
  public static void main(String[] args) throws JAXBException{
    JAXBContext jaxb = JAXBContext.newInstance(Messages.class);
    Marshaller marshaller = jaxb.createMarshaller();
    marshaller.setProperty(Marshaller.JAXB_FRAGMENT,Boolean.TRUE);
    StringWriter writer = new StringWriter();
    marshaller.marshal(new Messages(), writer);
    System.out.println(writer.toString());
  } 
}
```

1.  我们现在将创建一个名为`com.packt`的模块。要创建一个模块，我们需要创建一个名为`module-info.java`的文件，其中包含模块定义。模块定义包含模块的依赖关系和模块向其他模块导出的包：

```java
    module com.packt{
      //depends on the java.xml.bind module
      requires java.xml.bind;
      //need this for Messages class to be available to java.xml.bind
      exports  com.packt to java.xml.bind;
    }
```

我们将在第三章中详细解释模块化。但这个例子只是为了让您体验模块化编程并测试您的 JDK 安装。

具有上述文件的目录结构如下：

![](img/c74606e0-ea0b-4bb3-b8ed-a8f2cc8cf30f.png)

1.  让我们编译并运行代码。从`hellowordxml`目录中，创建一个新目录，用于放置编译后的类文件：

```java
      mkdir -p mods/com.packt
```

将源代码`HelloWorldXml.java`和`module-info.java`编译成`mods/com.packt`目录：

```java
 javac -d mods/com.packt/ src/com.packt/module-info.java
      src/com.packt/com/packt/HelloWorldXml.java
```

1.  使用`java --module-path mods -m com.packt/com.packt.HelloWorldXml`运行编译后的代码。您将看到以下输出：

```java
<messages><message>Hello World in XML</message></messages>
```

如果您无法理解`java`或`javac`命令中传递的选项，请不要担心。您将在第三章中了解它们，*模块化编程*。

# Java 11 中有什么新功能？

Java 9 的发布是 Java 生态系统的一个里程碑。在项目 Jigsaw 下开发的模块化框架成为了 Java SE 发布的一部分。另一个主要功能是 JShell 工具，这是一个用于 Java 的 REPL 工具。Java 9 引入的许多其他新功能在发布说明中列出：[`www.oracle.com/technetwork/java/javase/9all-relnotes-3704433.html`](http://www.oracle.com/technetwork/java/javase/9all-relnotes-3704433.html)。

在本教程中，我们将列举并讨论 JDK 18.3 和 18.9（Java 10 和 11）引入的一些新功能。

# 准备就绪

Java 10 发布（JDK 18.3）开始了一个为期六个月的发布周期——每年三月和九月——以及一个新的发布编号系统。它还引入了许多新功能，其中最重要的（对于应用程序开发人员）是以下内容：

+   允许使用保留的`var`类型声明变量的本地变量类型推断（见第十五章，*使用 Java 10 和 Java 11 进行编码的新方法*）

+   G1 垃圾收集器的并行完整垃圾收集，改善了最坏情况下的延迟

+   一个新的方法`Optional.orElseThrow()`，现在是现有`get()`方法的首选替代方法

+   用于创建不可修改集合的新 API：`java.util`包的`List.copyOf()`、`Set.copyOf()`和`Map.copyOf()`方法，以及`java.util.stream.Collectors`类的新方法：`toUnmodifiableList()`、`toUnmodifiableSet()`和`toUnmodifiableMap()`（参见第五章，*流和管道*）

+   一组默认的根证书颁发机构，使 OpenJDK 构建更受开发人员欢迎

+   新的 Javadoc 命令行选项`--add-stylesheet`支持在生成的文档中使用多个样式表

+   扩展现有的类数据共享功能，允许将应用程序类放置在共享存档中，从而提高启动时间并减少占用空间（参见*使用应用程序类数据共享*教程）

+   一种实验性的即时编译器 Graal，可以在 Linux/x64 平台上使用

+   一个干净的垃圾收集器（GC）接口，使得可以更简单地向 HotSpot 添加新的 GC，而不会干扰当前的代码库，并且更容易地从 JDK 构建中排除 GC

+   使 HotSpot 能够在用户指定的替代内存设备上分配对象堆，例如 NVDIMM 内存模块

+   线程本地握手，用于在执行全局 VM 安全点的情况下在线程上执行回调

+   Docker 意识：JVM 将知道它是否在 Linux 系统上的 Docker 容器中运行，并且可以提取容器特定的配置信息，而不是查询操作系统

+   三个新的 JVM 选项，为 Docker 容器用户提供对系统内存的更大控制

在发布说明中查看 Java 10 的新功能的完整列表：[`www.oracle.com/technetwork/java/javase/10-relnote-issues-4108729.html`](https://www.oracle.com/technetwork/java/javase/10-relnote-issues-4108729.html)。

我们将在下一节更详细地讨论 JDK 18.9 的新功能。

# 如何做到...

我们挑选了一些我们认为对应用程序开发人员最重要和有用的功能。

# JEP 318 – Epsilon

Epsilon 是一种所谓的无操作垃圾收集器，基本上什么都不做。它的用例包括性能测试、内存压力和虚拟机接口。它还可以用于短暂的作业或不消耗太多内存且不需要垃圾收集的作业。

我们在第十一章的*内存管理和调试*中的食谱*理解 Epsilon，一种低开销的垃圾收集器*中更详细地讨论了此功能。

# JEP 321 – HTTP 客户端（标准）

JDK 18.9 标准化了 JDK 9 中引入并在 JDK 10 中更新的孵化 HTTP API 客户端。基于`CompleteableFuture`，它支持非阻塞请求和响应。新的实现是异步的，并提供了更好的可追踪的数据流。

第十章的*网络*中，通过几个食谱更详细地解释了此功能。

# JEP 323 – 用于 Lambda 参数的本地变量语法

lambda 参数的本地变量语法与 Java 11 中引入的保留`var`类型的本地变量声明具有相同的语法。有关更多详细信息，请参阅第十五章的*使用 lambda 参数的本地变量语法*食谱。

# JEP 333 – ZGC

**Z 垃圾收集器**（**ZGC**）是一种实验性的低延迟垃圾收集器。其暂停时间不应超过 10 毫秒，与使用 G1 收集器相比，应用吞吐量不应降低超过 15%。ZGC 还为未来的功能和优化奠定了基础。Linux/x64 将是第一个获得 ZGC 支持的平台。

# 新 API

标准 Java API 有几个新增内容：

+   `Character.toString(int codePoint)`: 返回表示由提供的 Unicode 代码点指定的字符的`String`对象：

```java
var s = Character.toString(50);
System.out.println(s);  //prints: 2

```

+   `CharSequence.compare(CharSequence s1, CharSequence s2)`: 按字典顺序比较两个`CharSequence`实例。返回有序列表中第二个参数的位置与第一个参数位置的差异：

```java
var i = CharSequence.compare("a", "b");
System.out.println(i);   //prints: -1

i = CharSequence.compare("b", "a");
System.out.println(i);   //prints: 1

i = CharSequence.compare("this", "that");
System.out.println(i);   //prints: 8

i = CharSequence.compare("that", "this");
System.out.println(i);   //prints: -8

```

+   `String`类的`repeat(int count)`方法：返回由`count`次重复组成的`String`值：

```java
String s1 = "a";
String s2 = s1.repeat(3); //prints: aaa
System.out.println(s2);

String s3 = "bar".repeat(3);
System.out.println(s3); //prints: barbarbar

```

+   `String`类的`isBlank()`方法：如果`String`值为空或仅包含空格，则返回`true`，否则返回`false`。在我们的示例中，我们将其与`isEmpty()`方法进行了对比，后者仅在`length()`为零时返回`true`：

```java
String s1 = "a";
System.out.println(s1.isBlank());  //false
System.out.println(s1.isEmpty());  //false

String s2 = "";
System.out.println(s2.isBlank());  //true
System.out.println(s2.isEmpty());  //true

String s3 = "  ";
System.out.println(s3.isBlank());  //true
System.out.println(s3.isEmpty());  //false
```

+   `String`类的`lines()`方法：返回一个`Stream`对象，该对象从源`String`值中提取行，行之间由行终止符`\n`、`\r`或`\r\n`分隔：

```java
String s = "l1 \nl2 \rl3 \r\nl4 ";
s.lines().forEach(System.out::print); //prints: l1 l2 l3 l4 

```

+   `String`类的三个方法，用于从源`String`值中移除前导空格、尾随空格或两者：

```java
String s = " a b ";
System.out.println("'" + s.strip() + "'");        // 'a b'
System.out.println("'" + s.stripLeading() + "'"); // 'a b '
System.out.println("'" + s.stripTrailing() + "'");// ' a b'

```

+   两个构造`java.nio.file.Path`对象的`Path.of()`方法：

```java
Path filePath = Path.of("a", "b", "c.txt");
System.out.println(filePath);     //prints: a/b/c.txt

try {
    filePath = Path.of(new URI("file:/a/b/c.txt"));
    System.out.println(filePath);  //prints: /a/b/c.txt
} catch (URISyntaxException e) {
    e.printStackTrace();
}
```

+   `java.util.regex.Pattern`类的`asMatchPredicate()`方法，它创建了一个`java.util.function.Predicate`函数接口的对象，然后允许我们测试`String`值是否与编译后的模式匹配。在下面的示例中，我们测试`String`值是否以`a`字符开头并以`b`字符结尾：

```java
Pattern pattern = Pattern.compile("^a.*z$");
Predicate<String> predicate = pattern.asMatchPredicate();
System.out.println(predicate.test("abbbbz")); // true
System.out.println(predicate.test("babbbz")); // false
System.out.println(predicate.test("abbbbx")); // false

```

# 还有更多...

JDK 18.9 中引入了相当多的其他更改：

+   移除了 Java EE 和 CORBA 模块

+   JavaFX 已从 Java 标准库中分离并移除

+   `util.jar`中的 Pack200 和 Unpack200 工具以及 Pack200 API 已被弃用

+   Nashorn JavaScript 引擎以及 JJS 工具已被弃用，并打算在将来删除它们

+   Java 类文件格式被扩展以支持新的常量池形式`CONSTANT_Dynamic`

+   Aarch64 内在函数得到改进，为 Aarch64 处理器实现了`java.lang.Math` sin、cos 和 log 函数的新内在函数 JEP 309—动态类文件常量

+   Flight Recorder 为故障排除 Java 应用程序和 HotSpot JVM 提供了低开销的数据收集框架

+   Java 启动器现在可以运行作为 Java 源代码单个文件提供的程序，因此这些程序可以直接从源代码运行

+   低开销的堆分析，提供了一种对 Java 堆分配进行采样的方式，可以通过 JVM 工具接口访问

+   **传输层安全性**（**TLS**）1.3 增加了安全性并提高了性能

+   在`java.lang.Character`，`java.lang.String`，`java.awt.font.NumericShaper`，`java.text.Bidi,java.text.BreakIterator`和`java.text.Normalizer`类中支持 Unicode 版本 10.0

阅读 Java 11（JDK 18.9）发行说明以获取更多详细信息和其他更改。

# 使用应用程序类数据共享

这个功能自 Java 5 以来就存在。它在 Java 9 中作为商业功能得到了扩展，不仅允许引导类，还允许将应用程序类放置在 JVM 共享的存档中。在 Java 10 中，这个功能成为了 open JDK 的一部分。它减少了启动时间，并且当同一台机器上运行多个 JVM 并部署相同的应用程序时，减少了内存消耗。

# 做好准备

从共享存档加载类的优势有两个原因：

+   存档中存储的类是经过预处理的，这意味着 JVM 内存映射也存储在存档中。这减少了 JVM 实例启动时类加载的开销。

+   内存区域甚至可以在同一台计算机上运行的 JVM 实例之间共享，这通过消除每个实例中复制相同信息的需要来减少总体内存消耗。

新的 JVM 功能允许我们创建一个要共享的类列表，然后使用这个列表创建一个共享存档，并使用共享存档快速加载存档类到内存中。

# 如何做…

1.  默认情况下，JVM 可以使用随 JDK 提供的类列表创建一个存档。例如，运行以下命令：

```java
java -Xshare:dump
```

它将创建一个名为`classes.jsa`的共享存档文件。在 Linux 系统上，该文件放置在以下文件夹中：

```java
/Library/Java/JavaVirtualMachines/jdk-11.jdk/Contents/Home/lib/server
```

在 Windows 系统上，它放置在以下文件夹中：

```java
C:\Program Files\Java\jdk-11\bin\server
```

如果这个文件夹只能被系统管理员访问，以管理员身份运行命令。

请注意，并非所有类都可以共享。例如，位于类路径上的目录中的`.class`文件和由自定义类加载器加载的类不能添加到共享存档中。

1.  告诉 JVM 使用默认的共享存档，使用以下命令：

```java
java -Xshare:on -jar app.jar
```

上述命令将存档的内容映射到固定地址。当所需的地址空间不可用时，这种内存映射操作有时会失败。如果在使用`-Xshare:on`选项时发生这种情况，JVM 将以错误退出。或者，可以使用`-Xshare:auto`选项，它只是禁用该功能，并且如果由于任何原因无法使用共享存档，则从类路径加载类。

1.  创建加载的应用程序类列表的最简单方法是使用以下命令：

```java
java -XX:+UseAppCDS -XX:DumpLoadedClassList=classes.txt -jar app.jar
```

上述命令记录了`classes.txt`文件中加载的所有类。如果要使应用程序加载更快，请在应用程序启动后立即停止 JVM。如果需要更快地加载某些类，但这些类不会在应用程序启动时自动加载，请确保执行需要这些类的用例。

1.  或者，您可以手动编辑`classes.txt`文件，并添加/删除任何需要放入共享存档的类。首次自动创建此文件并查看格式。这是一个简单的文本文件，每行列出一个类。

1.  创建列表后，使用以下命令生成共享存档：

```java
java -XX:+UseAppCDS -Xshare:dump -XX:SharedClassListFile=classes.txt -XX:SharedArchiveFile=app-shared.jsa --class-path app.jar
```

请注意，共享存档文件的名称不是`classes.jsa`，因此不会覆盖默认共享存档。

1.  通过执行以下命令使用创建的存档：

```java
java -XX:+UseAppCDS -Xshare:on -XX:SharedArchiveFile=app-shared.jsa -jar app.jar
```

同样，您可以使用`-Xshare:auto`选项，以避免 JVM 意外退出。

共享存档的使用效果取决于其中的类数量和应用程序的其他细节。因此，我们建议您在承诺使用特定类列表之前进行实验和测试各种配置。
