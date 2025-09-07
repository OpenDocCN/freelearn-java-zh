# *第七章*：Java 标准库和外部库

没有使用标准库（也称为 **Java 类库**（**JCL**））就无法编写 Java 程序。这就是为什么对这类库的熟悉程度对于成功的编程来说与了解语言本身一样重要。

此外，还有非标准库，也称为外部库或第三方库，因为它们不包括在 **Java 开发工具包**（**JDK**）的发行版中。其中一些已经成为任何程序员工具箱中的永久性固定装置。

跟踪这些库中所有可用的功能并不容易。这是因为一个 `java.lang`。

本章的目的是为您提供一个关于 JCL 最受欢迎的包以及外部库的功能概述。

本章将涵盖以下主题：

+   **Java 类库**（**JCL**）

+   外部库

# 技术要求

要能够执行本章中的代码示例，您需要以下内容：

+   搭载 Microsoft Windows、Apple macOS 或 Linux 操作系统的计算机

+   Java SE 版本 17 或更高版本

+   您选择的 IDE 或代码编辑器

在*第一章*“Java 17 入门”中提供了如何设置 Java SE 和 IntelliJ IDEA 编辑器的说明。包含本章代码示例的文件可在 GitHub 的 [`github.com/PacktPublishing/Learn-Java-17-Programming.git`](https://github.com/PacktPublishing/Learn-Java-17-Programming.git) 仓库中的 `examples/src/main/java/com/packt/learnjava/ch07_libraries` 文件夹中找到。

# Java 类库（JCL）

JCL 是一组实现语言的包。用更简单的话说，它是一组包含在 JDK 中的 `.class` 文件集合，可供使用。一旦您安装了 Java，您就会作为安装的一部分获得它们，并可以使用 JCL 类作为构建块开始构建您的应用程序代码，这些构建块负责处理大量的底层管道。JCL 的丰富性和易用性在很大程度上促进了 Java 的普及。

要使用 JCL 包，您可以在不向 `pom.xml` 文件添加新依赖项的情况下导入它。Maven 会自动将 JCL 添加到类路径中。这就是标准库和外部库的区别；如果您需要将库（通常是 `.jar` 文件）作为依赖项添加到 Maven `pom.xml` 配置文件中，则该库是外部库。否则，它是一个标准库或 JCL。

一些 JCL 包名以 `java` 开头。传统上，它们被称为 *核心 Java 包*，而以 `javax` 开头的那些曾经被称为 *扩展包*。这样做是因为扩展被认为是可选的，甚至可能独立于 JDK 发布。也曾尝试将前一个扩展库提升为核心包。但这将需要将包名从 `java` 更改为 `javax`，这将破坏已经使用 `javax` 包的应用程序。因此，这个想法被放弃了，所以核心包和扩展包之间的区别逐渐消失了。

因此，如果你查看 Oracle 网站上的官方 Java API，你将看到不仅列出了 `java` 和 `javax` 包，还包括 `jdk`、`com.sun`、`org.xml` 以及其他一些包。这些额外的包主要用于其他专用应用程序的工具。在这本书中，我们将主要关注主流 Java 编程，并只讨论 `java` 和 `javax` 包。

## java.lang

这个包非常基础，因此在使用它时不需要导入。JVM 作者决定自动导入它。它包含了 JCL 中最常用的类：

+   `Object`：任何其他 Java 类的基类。

+   `Class`：在运行时携带每个加载类的元数据。

+   `String`、`StringBuffer` 和 `StringBuilder`：支持 `String` 类型的操作。

+   所有原始类型的包装类：`Byte`、`Boolean`、`Short`、`Character`、`Integer`、`Long`、`Float` 和 `Double`。

+   `Number`：数值原始类型包装类的基类——所有之前列出的类，除了 `Boolean`。

+   `System`：提供对重要系统操作以及标准输入和输出的访问（我们在本书的每个代码示例中都使用了 `System.out` 对象）。

+   `Runtime`：提供对执行环境的访问。

+   `Thread` 和 `Runnable` 接口：对于创建 Java 线程是基本的。

+   `Iterable` 接口：用于迭代语句。

+   `Math`：提供基本数值操作的方法。

+   `Throwable`：所有异常的基类。

+   `Error`：这是一个 `exception` 类，因为所有它的子类都用于传达系统错误，这些错误不能被应用程序捕获。

+   `Exception`：这个类及其直接子类代表受检异常。

+   `RuntimeException`：这个类及其子类代表非受检异常，也称为运行时异常。

+   `ClassLoader`：这个类读取 `.class` 文件并将它们（加载）放入内存；它也可以用来构建定制的类加载器。

+   `Process` 和 `ProcessBuilder`：这些类允许你创建其他 JVM 进程。

还有很多其他有用的类和接口也都可以使用。

## java.util

`java.util` 包的大部分内容都是用来支持 Java 集合的：

+   `Collection` 接口：许多其他集合接口的基接口，它声明了管理集合元素所需的所有基本方法；例如，`size()`、`add()`、`remove()`、`contains()`、`stream()`等。它还扩展了`java.lang.Iterable`接口并继承了其方法，包括`iterator()`和`forEach()`，这意味着`Collection`接口的任何实现或其子接口（`List`、`Set`、`Queue`、`Deque`等）也可以用于迭代语句，例如`ArrayList`、`LinkedList`、`HashSet`、`AbstractQueue`、`ArrayDeque`等。

+   `Map` 接口及其实现类：`HashMap`、`TreeMap`等。

+   `Collections` 类：此类提供了许多静态方法，用于分析、操作和转换集合。

许多其他集合接口、类和相关实用工具也可用。

我们在第六章中讨论了 Java 集合，并展示了它们的使用示例*第六章*，*数据结构、泛型和常用工具*。

`java.util` 包还包括几个其他有用的类：

+   `Objects`：提供各种与对象相关的实用方法，其中一些我们在*第六章*中讨论过，*数据结构、泛型和常用工具*。

+   `Arrays`：包含 160 个静态方法来操作数组，其中一些我们在*第六章*中讨论过，*数据结构、泛型和常用工具*。

+   `Formatter`：这允许你格式化任何原始类型，包括`String`、`Date`和其他类型；我们曾在*第六章*中学习过如何使用它，*数据结构、泛型和常用工具*。

+   `Optional`、`OptionalInt`、`OptionalLong`和`OptionalDouble`：这些类通过包装可能为 null 或非 null 的实际值来帮助避免`NullPointerException`。

+   `Properties`：帮助读取和创建用于应用程序配置和类似目的的键值对。

+   `Random`：通过生成伪随机数流来补充`java.lang.Math.random()`方法。

+   `StringTokenizer`：将`String`对象分割成由指定分隔符分隔的标记。

+   `StringJoiner`: 构建一个由指定分隔符分隔的字符序列。可选地，它被指定的前缀和后缀包围。

许多其他有用的实用类也可用，包括支持国际化、Base64 编码和解码的类。

## java.time

`java.time` 包包含用于管理日期、时间、期间和持续时间的类。该包包括以下内容：

+   `Month` 枚举。

+   `DayOfWeek` 枚举。

+   `Clock` 类，它使用时区返回当前的瞬间、日期和时间。

+   `Duration` 和 `Period` 类表示和比较不同时间单位的时间量。

+   `LocalDate`、`LocalTime` 和 `LocalDateTime` 类表示不带时区的日期和时间。

+   `ZonedDateTime` 类表示带时区的日期和时间。

+   `ZoneId` 类识别一个时区，例如 America/Chicago。

+   `java.time.format.DateTimeFormatter` 类允许您按照 **国际标准化组织** (**ISO**) 格式呈现日期和时间，例如 *YYYY-MM-DD* 模式。

+   一些支持日期和时间操作的其它类。

我们在 *第六章*，*数据结构、泛型和常用工具* 中讨论了这些类中的大多数。

## java.io 和 java.nio

`java.io` 和 `java.nio` 包包含支持使用流、序列化和文件系统读取和写入数据的类和接口。这两个包之间的区别如下：

+   `java.io` 包的类允许您在不缓存数据的情况下读取/写入数据（正如我们在 *第五章*，*字符串、输入/输出和文件*）中讨论的那样），而 `java.nio` 包的类创建缓冲区，允许您在填充的缓冲区中来回移动。

+   `java.io` 包的类在读取或写入所有数据之前会阻塞流，而 `java.nio` 包的类以非阻塞方式实现（我们将在 *第十五章*，*响应式编程*）中讨论非阻塞方式）。

## java.sql 和 javax.sql

这两个包构成了 `javax.sql` 包，它通过提供以下支持来补充 `java.sql` 包：

+   `DataSource` 接口作为 `DriverManager` 类的替代方案

+   连接和语句池

+   分布式事务

+   行集

我们将在 *第十章*，*数据库中的数据管理* 中讨论这些包并展示代码示例。

## java.net

`java.net` 包包含支持以下两个级别的应用程序网络功能的类：

+   **低级网络**，基于以下：

    +   IP 地址

    +   套接字，作为基本的双向数据通信机制

    +   各种网络接口

+   **高级网络**，基于以下：

    +   **通用资源标识符** (**URI**)

    +   **通用资源定位符** (**URL**)

    +   连接到由 URL 指向的资源

我们将在 *第十二章*，*网络编程* 中讨论这个包并展示其代码示例。

## java.lang.math 和 java.math

`java.lang.math` 包包含执行基本数值操作的方法，例如计算两个数值的最小值和最大值、绝对值、基本指数、对数、平方根、三角函数以及许多其他数学运算。

`java.math` 包补充了 `java.lang` 包的原生类型和包装类，因为你可以使用 `BigDecimal` 和 `BigInteger` 类处理更大的数字。

## java.awt, javax.swing, 和 javafx

第一个支持构建 `java.awt` 包的 Java 库。它提供了一个接口，允许你访问执行平台的本地系统，从而创建和管理窗口、布局和事件。它还提供了基本的 GUI 小部件（如文本字段、按钮和菜单），提供了对系统托盘的访问，并允许你从 Java 代码中启动网页浏览器和发送电子邮件。它对本地代码的依赖性使得基于 AWT 的 GUI 在不同的平台上看起来不同。

1997 年，Sun Microsystems 和 Netscape Communications Corporation 引入了 Java 基础类库，后来称为 Swing，并将它们放在了 `javax.swing` 包中。使用 Swing 构建的 GUI 组件能够模拟某些本地平台的样式和感觉，同时也允许你插入一个不依赖于其运行平台的样式和感觉。它通过添加选项卡面板、滚动面板、表格和列表来扩展了 GUI 可以拥有的小部件列表。Swing 组件是轻量级的，因为它们不依赖于本地代码，并且完全用 Java 实现。

2007 年，Sun Microsystems 宣布创建了 JavaFX，它最终成为了一个跨多种不同设备创建和交付桌面应用程序的软件平台。它旨在取代 Swing 成为 Java SE 的标准 GUI 库。JavaFX 框架位于以 `javafx` 开头的包中，支持所有主要的桌面 **操作系统**（**OSs**）和多个移动操作系统，包括 Symbian OS、Windows Mobile 和一些专有实时操作系统。

JavaFX 为 GUI 开发者的工具箱增加了对平滑动画、网页视图、音频和视频播放以及样式的支持，这些都是基于 **层叠样式表**（**CSS**）。然而，Swing 有更多的组件和第三方库，因此使用 JavaFX 可能需要创建在 Swing 中早已实现的定制组件和管道。这就是为什么，尽管 JavaFX 被推荐为桌面 GUI 实现的首选，但 Swing 仍将在可预见的未来成为 Java 的一部分，根据 Oracle 网站的官方回应（[`www.oracle.com/technetwork/java/javafx/overview/faq-1446554.html#6`](http://www.oracle.com/technetwork/java/javafx/overview/faq-1446554.html#6)）。因此，继续使用 Swing 是可能的，但如果可能的话，最好切换到 JavaFX。

我们将在 *第十二章* *Java GUI 编程* 中讨论 JavaFX 并展示其代码示例。

# 外部库

最常用的第三方非 JCL 库列表之间包括 20 到 100 个库。在本节中，我们将讨论那些包含在大多数此类列表中的库。所有这些都是开源项目。

## org.junit

`org.junit` 包是开源测试框架 JUnit 的根包。它可以作为以下 `pom.xml` 依赖项添加到项目中：

```java
<dependency>
```

```java
    <groupId>junit</groupId>
```

```java
    <artifactId>junit</artifactId>
```

```java
    <version>4.13.2</version>
```

```java
    <scope>test</scope>
```

```java
</dependency>
```

前一个依赖项标签中的 `scope` 值告诉 Maven 在测试代码将要运行时包含库 `.jar` 文件，但不在应用程序的生产 `.jar` 文件中。有了这个依赖项，您可以创建一个测试。您可以自己编写代码，或者通过以下操作让 IDE 帮您完成：

1.  右键点击您想要测试的类名。

1.  选择 **转到**。

1.  选择 **测试**。

1.  点击 **创建新测试**。

1.  选择您想要测试的类的相关方法复选框。

1.  使用 `@Test` 注解编写生成的测试方法的代码。

1.  如果需要，添加带有 `@Before` 和 `@After` 注解的方法。

假设我们有一个以下类：

```java
public class Class1 {
```

```java
    public int multiplyByTwo(int i){
```

```java
        return i * 2;
```

```java
    }
```

```java
}
```

如果您遵循前面的步骤，以下测试类将在测试源树下创建：

```java
import org.junit.Test;
```

```java
public class Class1Test {
```

```java
    @Test
```

```java
    public void multiplyByTwo() {
```

```java
    }
```

```java
}
```

现在，您可以按照以下方式实现 `void` `multiplyByTwo()` 方法：

```java
@Test
```

```java
public void multiplyByTwo() {
```

```java
    Class1 class1 = new Class1();
```

```java
    int result = class1.multiplyByTwo(2);
```

```java
    Assert.assertEquals(4, result);
```

```java
}
```

单元是一个最小的可测试的代码片段，因此得名。最佳测试实践将方法视为最小的可测试单元。这就是为什么单元测试通常测试一个方法。

## org.mockito

单元测试经常遇到的一个问题是需要测试一个使用第三方库、数据源或另一个类的方法。在测试时，您想要控制所有输入，以便您可以预测测试代码的预期结果。这就是模拟或模拟测试代码交互的对象的行为技术派上用场的时候。

开源 Mockito 框架（`org.mockito` 根包名）允许您做到这一点——创建模拟对象。使用它相当简单。这里有一个简单的例子。假设我们需要测试另一个 `Class1` 方法：

```java
public class Class1 {
```

```java
    public int multiplyByTwo2(Class2 class2){
```

```java
        return class2.getValue() * 2;
```

```java
    }
```

```java
}
```

要测试此方法，我们需要确保 `getValue()` 方法返回某个特定的值，因此我们将模拟此方法。为此，请按照以下步骤操作：

1.  在 Maven 的 `pom.xml` 配置文件中添加一个依赖项：

    ```java
       <dependency>
           <groupId>org.mockito</groupId>
           <artifactId>mockito-core</artifactId>
           <version>4.2.0</version>
           <scope>test</scope>
       </dependency>
    ```

1.  为您需要模拟的类调用 `Mockito.mock()` 方法：

    ```java
    Class2 class2Mock = Mockito.mock(Class2.class);
    ```

1.  设置您需要从方法返回的值：

    ```java
    Mockito.when(class2Mock.getValue()).thenReturn(5);
    ```

1.  现在，您可以将模拟对象作为参数传递到调用模拟方法的测试方法中：

    ```java
    Class1 class1 = new Class1();
    int result = class1.multiplyByTwo2(class2Mock);
    ```

1.  模拟的方法返回您预定义的结果：

    ```java
    Assert.assertEquals(10, result);
    ```

1.  `@Test` 方法应如下所示：

    ```java
    @Test
    public void multiplyByTwo2() {
        Class2 class2Mock = Mockito.mock(Class2.class);
        Mockito.when(class2Mock.getValue()).thenReturn(5);
        Class1 class1 = new Class1();
        int result = class1.multiplyByTwo2(mo);
        Assert.assertEquals(10, result);
    }
    ```

Mockito 有一些限制。例如，您不能模拟静态方法和私有方法。否则，这是一个通过可靠地预测使用第三方类的结果来隔离您正在测试的代码的绝佳方式。

## org.apache.log4j 和 org.slf4j

在整本书中，我们使用`System.out`来显示结果。在实际应用中，你可以这样做并将输出重定向到文件，例如，用于后续分析。一旦你这样做了一段时间，你会注意到你需要更多关于每个输出的详细信息：每个语句的日期和时间，以及生成日志语句的类名，例如。随着代码库的增长，你会发现将来自不同子系统或包的输出发送到不同的文件或关闭一些消息会很不错，当一切按预期工作的时候，当检测到问题并需要更多关于代码行为的信息时再打开它们。而且你不想日志文件的大小无限制地增长。

可以编写代码来完成所有这些。但有几个框架基于配置文件中的设置来完成这些，你可以每次需要更改日志行为时更改该配置文件。用于此目的的最流行的两个框架被称为 log4j（发音为 LOG-FOUR-JAY）和 slf4j（发音为 S-L-F-FOUR-JAY）。

这两个框架不是竞争对手。slf4j 框架是一个门面，它提供了一个对底层实际日志框架的统一访问；其中之一也可以是 log4j。这样的门面在库开发期间特别有用，当程序员不知道应用程序将使用哪种日志框架时。通过使用 slf4j 编写代码，程序员允许你在以后配置它，以便可以使用任何日志系统。

因此，如果你的代码只将被你团队开发的应用程序使用，仅使用 log4j 就足够了。否则，考虑使用 slf4j。

就像任何第三方库一样，在使用 log4j 框架之前，你必须将相应的依赖项添加到 Maven 的`pom.xml`配置文件中：

```java
<dependency>
```

```java
    <groupId>org.apache.logging.log4j</groupId>
```

```java
    <artifactId>log4j-api</artifactId>
```

```java
    <version>2.17.0</version>
```

```java
</dependency>
```

```java
<dependency>
```

```java
    <groupId>org.apache.logging.log4j</groupId>
```

```java
    <artifactId>log4j-core</artifactId>
```

```java
    <version>2.17.0</version>
```

```java
</dependency>
```

例如，以下是框架的使用方法：

```java
import org.apache.logging.log4j.LogManager;
```

```java
import org.apache.logging.log4j.Logger;
```

```java
public class Class1 {
```

```java
   static final Logger logger = 
```

```java
               LogManager.getLogger(Class1.class.getName());
```

```java
    public static void main(String... args){
```

```java
        new Class1().multiplyByTwo2(null);
```

```java
    }
```

```java
    public int multiplyByTwo2(Class2 class2){
```

```java
        if(class2 == null){
```

```java
            logger.error("The parameter should not be null");
```

```java
            System.exit(1);
```

```java
        }
```

```java
        return class2.getValue() * 2;
```

```java
    }
```

```java
}
```

如果我们运行前面的`main()`方法，我们将得到以下输出：

```java
18:34:07.672 [main] ERROR Class1 - The parameter should not be null
```

```java
Process finished with exit code 1
```

如你所见，如果没有将 log4j 特定的配置文件添加到项目中，log4j 将在`DefaultConfiguration`类中提供默认配置。默认配置如下：

+   日志消息将输出到控制台。

+   消息的格式将是`%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n`。

+   日志级别将是`Level.ERROR`（其他级别包括`OFF`、`FATAL`、`WARN`、`INFO`、`DEBUG`、`TRACE`和`ALL`）。

通过将`log4j2.xml`文件添加到`resources`文件夹（Maven 将其放在类路径上）并包含以下内容，可以达到相同的结果：

```java
<?xml version="1.0" encoding="UTF-8"?>
```

```java
<Configuration status="WARN">
```

```java
    <Appenders>
```

```java
        <Console name="Console" target="SYSTEM_OUT">
```

```java
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] 
```

```java
                              %-5level %logger{36} - %msg%n"/>
```

```java
        </Console>
```

```java
    </Appenders>
```

```java
    <Loggers>
```

```java
        <Root level="error">
```

```java
            <AppenderRef ref="Console"/>
```

```java
        </Root>
```

```java
    </Loggers>
```

```java
</Configuration>
```

如果这对你来说还不够好，你可以更改配置，以便将不同级别的消息记录到不同的文件中，等等。阅读 log4j 文档以了解更多信息：[`logging.apache.org`](https://logging.apache.org)。

## org.apache.commons

`org.apache.commons` 包是另一个由名为 **Apache Commons** 的项目开发的流行库。它由一个名为 **Apache 软件基金会** 的开源程序员社区维护。该组织由 Apache Group 于 1999 年成立。Apache Group 自 1993 年以来一直围绕 Apache HTTP Server 的发展壮大。Apache HTTP Server 是一个开源的跨平台 Web 服务器，自 1996 年 4 月以来一直是最受欢迎的 Web 服务器。

Apache Commons 项目有以下三个组件：

+   **Commons Sandbox**：一个用于 Java 组件开发的工位；你可以在那里为开源工作做出贡献。

+   **Commons Dormant**：一个存放当前不活跃组件的仓库；你可以使用那里的代码，但由于这些组件可能不会很快发布，你必须自己构建这些组件。

+   `org.apache.commons` 库。

我们在第 *第五章*，*字符串、输入/输出和文件* 中讨论了 `org.apache.commons.io` 包。

在以下小节中，我们将讨论 Commons Proper 的三个最受欢迎的包：

+   `org.apache.commons.lang3`

+   `org.apache.commons.collections4`

+   `org.apache.commons.codec.binary`

然而，在 `org.apache.commons` 下还有许多其他包，包含数千个类，可以轻松地用于使你的代码更加优雅和高效。

### lang 和 lang3

`org.apache.commons.lang3` 包是 `org.apache.commons.lang` 包的第三个版本。创建新包的决定是由引入的版本 3 的更改导致的向后不兼容，这意味着使用 `org.apache.commons.lang` 包的先前版本的现有应用程序在升级到版本 3 后可能会停止工作。但在主流编程的大多数情况下，将 `3` 添加到 `import` 语句中（作为迁移到新版本的方式）通常不会破坏任何东西。

根据文档，`org.apache.commons.lang3` 包提供了高度可重用的静态实用方法，主要关注为 `java.lang` 类增加价值。以下是一些显著的例子：

+   `ArrayUtils` 类：允许你搜索和操作数组；我们在 *第六章*，*数据结构、泛型和常用工具* 中讨论并演示了这一点。

+   `ClassUtils` 类：提供有关类的某些元数据。

+   `ObjectUtils` 类：检查对象数组是否为 null，比较对象，并以安全的方式计算对象数组的平均值和最小/最大值；我们在 *第六章*，*数据结构、泛型和常用工具* 中讨论并演示了这一点。

+   `SystemUtils` 类：提供有关执行环境的信息。

+   `ThreadUtils` 类：查找有关当前正在运行的线程的信息。

+   `Validate` 类：验证单个值和集合，比较它们，检查空值和匹配，并执行许多其他验证。

+   `RandomStringUtils` 类：从各种字符集的字符生成 `String` 对象。

+   `StringUtils` 类：我们在*第五章*，*字符串、输入/输出和文件*中讨论了这个类。

### collections4

尽管从表面上看，`org.apache.commons.collections4` 包的内容与 `org.apache.commons.collections` 包的内容（该包的版本 3）非常相似，但迁移到版本 4 可能不会像仅仅在 `import` 语句中添加 `4` 那样顺利。版本 4 删除了已弃用的类，并添加了与先前版本不兼容的泛型和其它特性。

您很难想出一个在这个包或其子包中不存在的集合类型或集合实用工具。以下只是包含的功能和实用工具的高级列表：

+   `Bag` 接口用于具有每个对象多个副本的集合。

+   实现 `Bag` 接口的十几个类。例如，以下是如何使用 `HashBag` 类的示例：

    ```java
      Bag<String> bag = new HashBag<>();
      bag.add("one", 4);
      System.out.println(bag);    //prints: [4:one]
      bag.remove("one", 1);
      System.out.println(bag);    //prints: [3:one]
      System.out.println(bag.getCount("one")); //prints: 3
    ```

+   `BagUtils` 类，用于转换基于 `Bag` 的集合。

+   `BidiMap` 接口用于双向映射，允许您不仅可以通过键获取值，还可以通过值获取键。它有几个实现，以下是一个示例：

    ```java
     BidiMap<Integer, String> bidi = new TreeBidiMap<>();
     bidi.put(2, "two");
     bidi.put(3, "three");
     System.out.println(bidi);  
                                   //prints: {2=two, 3=three}
     System.out.println(bidi.inverseBidiMap()); 
                                   //prints: {three=3, two=2}
     System.out.println(bidi.get(3));         //prints: three
     System.out.println(bidi.getKey("three")); //prints: 3
     bidi.removeValue("three"); 
     System.out.println(bidi);              //prints: {2=two}
    ```

+   `MapIterator` 接口提供对映射的简单快速迭代，如下所示：

    ```java
     IterableMap<Integer, String> map =
                 new HashedMap<>(Map.of(1, "one", 2, "two"));
     MapIterator it = map.mapIterator();
     while (it.hasNext()) {
          Object key = it.next();
          Object value = it.getValue();
          System.out.print(key + ", " + value + ", "); 
                                    //prints: 2, two, 1, one, 
          if(((Integer)key) == 2){
             it.setValue("three");
          }
     }
     System.out.println("\n" + map); 
                                   //prints: {2=three, 1=one}
    ```

+   有序映射和集合，它们保持元素在特定顺序，就像 `List` 一样；例如：

    ```java
     OrderedMap<Integer, String> map = new LinkedMap<>();
     map.put(4, "four");
     map.put(7, "seven");
     map.put(12, "twelve");
     System.out.println(map.firstKey()); //prints: 4
     System.out.println(map.nextKey(2)); //prints: null
     System.out.println(map.nextKey(7)); //prints: 12
     System.out.println(map.nextKey(4)); //prints: 7
    ```

+   引用映射、其键和/或值可以被垃圾收集器移除。

+   `Comparator` 接口的多种实现。

+   `Iterator` 接口的多种实现。

+   将数组枚举转换为集合的类。

+   允许您测试或创建集合的并集、交集或闭包的实用工具。

+   `CollectionUtils`、`ListUtils`、`MapUtils` 和 `MultiMapUtils` 类，以及许多其他特定接口的实用类。

读取该包的文档([`commons.apache.org/proper/commons-collections`](https://commons.apache.org/proper/commons-collections))以获取更多详细信息。

### codec.binary

`org.apache.commons.codec.binary` 包提供了对 Base64、Base32、二进制和十六进制字符串编码和解码的支持。这种编码是必要的，以确保您发送到不同系统的数据在传输过程中不会因为不同协议对字符范围的限制而改变。此外，一些系统将发送的数据解释为控制字符（例如，调制解调器）。

以下代码片段展示了该包中 `Base64` 类的基本编码和解码功能：

```java
String encodedStr = new String(Base64
```

```java
                    .encodeBase64("Hello, World!".getBytes()));
```

```java
System.out.println(encodedStr);  //prints: SGVsbG8sIFdvcmxkIQ==
```

```java
System.out.println(Base64.isBase64(encodedStr)); //prints: true
```

```java
String decodedStr = 
```

```java
  new String(Base64.decodeBase64(encodedStr.getBytes()));
```

```java
System.out.println(decodedStr);  //prints: Hello, World!
```

你可以在 Apache Commons 项目网站上了解更多关于此包的信息：[`commons.apache.org/proper/commons-codec`](https://commons.apache.org/proper/commons-codec)。

# 摘要

在本章中，我们概述了 JCL（Java 类库）中最受欢迎的包的功能——即 `java.lang`、`java.util`、`java.time`、`java.io`、`java.nio`、`java.sql`、`javax.sql`、`java.net`、`java.lang.math`、`java.math`、`java.awt`、`javax.swing` 和 `javafx`。

最受欢迎的外部库由 `org.junit`、`org.mockito`、`org.apache.log4j`、`org.slf4j` 和 `org.apache.commons` 包表示。这些库可以帮助你在功能已经存在且可以直接导入和使用的情况下避免编写自定义代码。

在下一章中，我们将讨论 Java 线程并演示其用法。我们还将解释并行处理和并发处理之间的区别。然后，我们将向您展示如何创建线程以及如何执行、监控和停止它。这对那些将要编写多线程处理代码的人来说将很有用，同时也对那些希望提高对 JVM 工作原理理解的人来说很有用，这将是下一章的主题。

# 测验

回答以下问题以测试你对本章知识的掌握：

1.  Java 类库是什么？选择所有适用的选项：

    1.  编译类的集合

    1.  随 Java 安装一起提供的包

    1.  Maven 自动添加到类路径中的 `.jar` 文件

    1.  任何用 Java 编写的库

1.  Java 外部库是什么？选择所有适用的选项：

    1.  不包含在 Java 安装中的 `.jar` 文件

    1.  必须在 `pom.xml` 中添加为依赖项的 `.jar` 文件，才能使用它

    1.  不是 JVM 作者编写的类

    1.  不属于 JCL 的类

1.  `java.lang` 包中包含哪些功能？选择所有适用的选项：

    1.  它是唯一包含 Java 语言实现的包。

    1.  它包含 JCL 中最常用的类。

    1.  它包含 `Object` 类，这是任何 Java 类的基类。

    1.  它包含 Java 语言规范中列出的所有类型。

1.  `java.util` 包中包含哪些功能？选择所有适用的选项：

    1.  Java 集合接口的所有实现

    1.  Java 集合框架的所有接口

    1.  JCL 的所有实用工具

    1.  类、数组、对象和属性

1.  `java.time` 包中包含哪些功能？选择所有适用的选项：

    1.  管理日期的类。

    1.  它是唯一管理时间的包。

    1.  表示日期和时间的类。

    1.  它是唯一管理日期的包。

1.  `java.io` 包中包含哪些功能？选择所有适用的选项：

    1.  处理二进制数据流

    1.  处理字符流

    1.  处理字节流

    1.  处理数字流

1.  `java.sql` 包中包含哪些功能？选择所有适用的选项：

    1.  支持数据库连接池

    1.  支持执行数据库语句

    1.  提供从数据库读取/写入数据的能力

    1.  支持数据库事务

1.  `java.net`包中包含哪些功能？选择所有适用的：

    1.  支持.NET 编程

    1.  支持套接字通信

    1.  支持基于 URL 的通信

    1.  支持基于 RMI 的通信

1.  `java.math`包中包含哪些功能？选择所有适用的：

    1.  支持最小和最大计算

    1.  支持大数

    1.  支持对数

    1.  支持开方计算

1.  `javafx`包中包含哪些功能？选择所有适用的：

    1.  支持发送传真消息

    1.  支持接收传真消息

    1.  支持 GUI 编程

    1.  支持动画

1.  `org.junit`包中包含哪些功能？选择所有适用的：

    1.  支持测试 Java 类

    1.  支持 Java 度量单位

    1.  支持单元测试

    1.  支持组织统一

1.  `org.mockito`包中包含哪些功能？选择所有适用的：

    1.  支持 Mockito 协议

    1.  允许你模拟方法的行为

    1.  支持静态方法模拟

    1.  生成类似第三方类的对象

1.  `org.apache.log4j`包中包含哪些功能？选择所有适用的：

    1.  支持将消息写入文件

    1.  支持从文件中读取消息

    1.  支持 Java 的 log4j 协议

    1.  支持控制日志文件的数量和大小

1.  `org.apache.commons.lang3`包中包含哪些功能？选择所有适用的：

    1.  支持 Java 语言版本 3

    1.  补充`java.lang`类

    1.  包含`ArrayUtils`、`ObjectUtils`和`StringUtils`类

    1.  包含`SystemUtils`类

1.  `org.apache.commons.collections4`包中包含哪些功能？选择所有适用的：

    1.  Java 集合框架接口的各种实现

    1.  Java 集合框架实现的多种实用工具

    1.  Vault 接口及其实现

    1.  包含`CollectionUtils`类

1.  `org.apache.commons.codec.binary`包中包含哪些功能？选择所有适用的：

    1.  支持在网络中发送二进制数据

    1.  允许你编码和解码数据

    1.  支持数据加密

    1.  包含`StringUtils`类
