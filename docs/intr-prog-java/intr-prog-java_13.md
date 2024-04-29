# 第十三章：Java 标准库和外部库

即使我们在本书中编写的第一个程序也使用了 JDK 中包含的库，称为标准库。不使用标准库无法编写非平凡程序。这就是为什么对这些库的熟悉程度对于成功编程与语言本身的知识一样重要。

还有非标准库，被称为外部库或第三方库，因为它们不包含在 JDK 发行版中，但它们几乎和标准库一样经常被使用。它们早已成为任何程序员工具包的固定成员。与 Java 本身保持同步并不容易跟踪这些库中所有可用的功能。这是因为 IDE 可以提示您有关语言功能，但无法提供有关尚未导入的包的功能的建议。唯一自动导入且无需导入的包是`java.lang`，这将是我们在本章中首先要概述的内容。

本章讨论的主题有：

+   什么是标准库和外部库？

+   Java 标准库概述

+   包`java.lang`

+   包`java.util`

+   包`java.io`和`java.nio`

+   包`java.sql`和`javax.sql`

+   包`java.net`

+   包`java.math`

+   包`java.awt`，`javax.swing`和`javafx`

+   Java 外部库概述

+   库`org.junit`

+   库`org.mockito`

+   库`org.log4j`和`org.slf4j`

+   库`org.apache.commons`

+   练习-使用`java.time.LocalDateTime`

# 什么是标准库和外部库？

标准库（也称为类标准库）是一组对语言的所有实现都可用的类和接口。简单来说，这意味着它是包含在 JDK 中的`.class`文件的集合，并且可以立即使用。一旦安装了 Java，您就可以将它们作为安装的一部分，并且可以开始使用标准库的类作为构建块来构建应用程序代码，这些类可以处理许多低级管道。标准库的丰富性和易用性大大促进了 Java 的流行。

这些集合是按包组织的。这就是为什么程序员称它们为 Java 标准库，因为为了使用它们，您必须根据需要导入库包，因此它们被视为许多库。

它们也是标准库，因为 Maven 会自动将它们添加到类路径中，因此我们不需要在`pom.xml`文件中列出它们作为依赖项。这就是标准库和外部库的区别；如果您需要将库（通常是`.jar`文件）作为依赖项添加到 Maven 配置文件`pom.xml`中，这个库就是外部库，也称为第三方库。否则，它就是标准库。

在接下来的章节中，我们将为每个类别提供概述，并更仔细地查看一些最受欢迎的标准和外部库。

# Java 标准库

如果在互联网上搜索“Java API”，您将找到 JDK 中包含的所有包的在线描述。一些包名称以`java`开头。它们传统上被称为核心 Java 包，而以`javax`开头的包曾被称为扩展。这样做可能是因为扩展被认为是可选的，甚至可能独立于 JDK 发布。还有一次尝试将以前的扩展库提升为核心包，但这将需要将包的名称从 Java 更改为 Javax，这将破坏已经存在的应用程序。因此，这个想法被放弃了，扩展成为 JDK 的标准部分，核心和扩展之间的区别逐渐消失。

这就是为什么如果你在 Oracle 网站上查看官方 Java API，你会看到标准不仅列出了`java`和`javax`包，还列出了`jdk`、`com.sun`、`org.xml`和其他一些包。这些额外的包主要被工具或其他专门的应用程序使用。在我们的书中，我们将主要集中讨论主流的 Java 编程，并且只谈论`java`和`javax`包。

# java.lang

这个包对于所有 Java 类库来说是如此基础，以至于不仅不需要在 Maven 配置的`pom.xml`文件中列出它作为依赖项（Java 标准库的所有其他包也不需要列出作为依赖项），而且其成员甚至不需要被导入才能使用。任何包的任何成员，无论是标准的还是非标准的，都必须被导入或者使用其完全限定名，除了`java.lang`包的类和接口。原因是它包含了 Java 中最重要和最常用的两个类：

+   `Object`：任何其他 Java 类的基类（参见第二章，*Java 语言基础*）

+   `Class`：其实例在运行时携带每个加载类的元数据（参见第十一章，*JVM 进程和垃圾回收*）

此外，`java.lang`包包括：

+   `String`、`StringBuffer`和`StringBuilders`类，支持`String`类型的操作（有关更多详细信息和用法示例，请参见第十五章，*管理对象、字符串、时间和随机数*）

+   所有基本类型的包装类：`Byte`、`Boolean`、`Short`、`Character`、`Integer`、`Long`、`Float`和`Double`（有关包装类及其用法的更多详细信息，请参见第九章，*运算符、表达式和语句*）

+   `Number`类，前面列出的数字包装类的基类

+   `System`类，提供对重要系统操作和标准输入输出的访问（我们在本书的每个代码示例中都使用了`System.out`对象）

+   `Runtime`类，提供对执行环境的访问

+   `Thread`类和`Runnable`接口，用于创建 Java 线程的基础

+   `Iterable`接口，用于迭代语句（参见第九章，*运算符、表达式和语句*）

+   `Math`类，提供基本数值操作的方法

+   `Throwable`类 - 所有异常的基类

+   异常类`Error`及其所有子类，用于传达不应被应用程序捕获的系统错误

+   `Exception`类及其许多子类，代表已检查的异常（参见第十章，*控制流语句*）

+   `RuntimeException`类及其许多子类，代表未经检查的异常，也称为运行时异常（参见第十章，*控制流语句*）

+   `ClassLoader`类，允许加载类并可用于构建自定义类加载器

+   `Process`和`ProcessBuilder`类，允许创建外部进程

+   许多其他有用的类和接口

# java.util

这是另一个非常常用的包。它的大部分内容都是用于支持集合：

+   `Collection`接口 - 许多集合接口的基本接口。它包含管理集合元素所需的所有基本方法：`size()`、`add()`、`remove()`、`contains()`、`iterator()`、`stream()`等。请注意，`Collection`接口扩展了`Iterable`接口，并从中继承了`iterator()`方法。这意味着`Collection`接口的任何实现都可以在迭代语句中使用。

+   扩展`Collection`接口的接口：`List`、`Set`、`Queue`、`Deque`等。

+   实现上述接口的许多类：`ArrayList`、`LinkedList`、`HashSet`、`AbstractQueue`、`ArrayDeque`等。

+   `Map`接口及其实现它的类：`HashMap`、`TreeMap`等。

+   `Collections`类提供了许多用于操作和转换集合的静态方法。

+   许多其他集合接口、类和相关实用程序。

我们将在《第十三章》《Java 集合》中更多地讨论集合，并查看它们的使用示例。

`java.util`包还包括其他几个有用的类：

+   `Objects`类提供了各种与对象相关的实用方法，包括两个对象的空安全`equals()`方法

+   `Arrays`类包含 200 多个静态方法来操作数组

+   `Formatter`类允许格式化任何原始类型，如`String`、`Date`和其他类型

+   类`Optional`、`OptionalInt`、`OptionalLong`和`OptionalDouble`通过包装实际值（可空或非空）来帮助避免`NullPointerException`

+   `Properties`类有助于读取和创建用于配置和类似目的的键值对

+   `Random`类通过生成伪随机数流来补充`Math.random()`方法

+   `Stack`类允许创建对象的**后进先出**（**LIFO**）堆栈

+   `StringTokeneizer`类将`String`对象分解为由指定分隔符分隔的标记

+   `StringJoiner`类构造由指定分隔符分隔并可选地由指定前缀和后缀包围的字符序列

+   许多其他有用的实用程序类，包括国际化支持类和 base64 编码和解码

# java.time

这是管理日期、时间、瞬间和持续时间的主要 Java API。该包包括：

+   枚举`Month`

+   枚举`DayOfWeek`

+   `Clock`类可以立即返回使用时区的当前日期和时间

+   `Duration`和`Period`类表示和比较不同时间单位的时间量

+   `LocalDate`、`LocalTime`、`LocalDateTime`类表示没有时区的日期和时间

+   `ZonedDateTime`类表示带有时区的日期时间

+   `ZoneId`类标识诸如 America/Chicago 之类的时区

+   支持日期和时间处理的其他一些类

`java.time.format.DateTimeFormatter`类允许您按照**国际标准化组织**（**ISO**）格式呈现日期和时间，并基于诸如 YYYY-MM-DD 等模式。

我们将在《第十五章》《管理对象、字符串、时间和随机数》中更多地讨论日期和时间处理，并查看代码示例。

# java.io 和 java.nio

`java.io`和`java.nio`包含支持使用流、序列化和文件系统读写数据的类。这两个包之间的区别如下：

+   `java.io`允许我们在读写数据时不进行缓存

+   `java.nio`创建一个缓冲区，并允许程序在缓冲区中来回移动

+   `java.io`的类方法阻塞流，直到所有数据都被读取或写入

+   `java.nio`代表一种非阻塞式的数据读写方式

# java.sql 和 javax.sql

这两个包组成了**Java 数据库连接**（**JDBC**）API，它允许访问和处理存储在数据源中的数据，通常是关系数据库。包`javax.sql`通过提供对以下内容的支持来补充包`java.sql`：

+   `DataSource`接口作为`DriverManager`的替代方案

+   连接池和语句池

+   分布式事务

+   行集

我们将在《第十六章》《数据库编程》中更多地讨论使用这些包，并查看代码示例。

# java.net

`java.net`包含支持两个级别的应用程序网络的类：

+   基于低级网络：

+   IP 地址

+   套接字，这是基本的双向数据通信机制

+   各种网络接口

+   基于高级网络：

+   **统一资源标识符**（**URI**）

+   **统一资源定位符**（**URL**）

+   与 URL 指向的资源的连接

# java.math

这个包通过允许使用`BigDecimal`和`BigInteger`类来处理更大的数字，来补充 Java 原始类型和`java.lang`包的包装类。

# java.awt、javax.swing 和 javafx

支持为桌面应用程序构建**图形用户界面**（**GUI**）的第一个 Java 库是`java.awt`包中的**抽象窗口工具包**（**AWT**）。它提供了一个接口到执行平台的本地系统，允许创建和管理窗口、布局和事件。它还具有基本的 GUI 小部件（如文本字段、按钮和菜单），提供对系统托盘的访问，并允许用户从 Java 代码中启动 Web 浏览器和电子邮件客户端。它对本地代码的重度依赖使得基于 AWT 的 GUI 在不同平台上看起来不同。

1997 年，Sun Microsystems 和 Netscape Communication Corporation 推出了 Java 基础类，后来称为 Swing，并放在`javax.swing`包中。使用 Swing 构建的 GUI 组件可以模拟一些本地平台的外观和感觉，但也允许用户插入不依赖于其运行的平台的外观和感觉。它通过添加选项卡面板、滚动窗格、表格和列表扩展了 GUI 可以拥有的小部件列表。Swing 组件被称为轻量级，因为它们不依赖于本地代码，完全由 Java 实现。

2007 年，Sun Microsystems 宣布了 JavaFX，它最终成为一个用于在许多不同设备上创建和交付桌面应用程序的软件平台，旨在取代 Swing 成为 Java SE 的标准 GUI 库。它位于以`javafx`开头的包中，支持所有主要的桌面操作系统和多个移动操作系统，包括塞班操作系统、Windows Mobile 和一些专有的实时操作系统。

JavaFX 为 GUI 开发人员增加了对平滑动画、Web 视图、音频和视频播放以及基于**层叠样式表**（**CSS**）的样式的支持。然而，Swing 具有更多的组件和第三方库，因此使用 JavaFX 可能需要创建自定义组件和在 Swing 中长时间前已实现的管道。这就是为什么，尽管 JavaFX 被推荐为桌面 GUI 实现的首选，但根据 Oracle 网站上的官方回应（[`www.oracle.com/technetwork/java/javafx/overview/faq-1446554.html#6`](http://www.oracle.com/technetwork/java/javafx/overview/faq-1446554.html#6)），Swing 将在可预见的未来仍然是 Java 的一部分。因此，可以继续使用 Swing，但如果可能的话，最好切换到 JavaFX。

# Java 外部库

各种统计数据在 20 或 100 个最常用的第三方库的列表中包含不同的名称。在本节中，我们将讨论其中大多数都包含在这些列表中的库。所有这些都是开源项目。

# org.junit

JUnit 是一个开源的测试框架，其根包名称为`org.junit`。它在本书中的多个代码示例中都有使用。正如你所看到的，它非常容易设置和使用（我们在第四章 *你的第一个 Java 项目*中描述了步骤）。

+   向 Maven 配置文件`pom.xml`添加依赖

+   手动创建一个测试，或右键单击您想要测试的类名，选择 Go To，然后选择 Test，然后选择 Create New Test，然后检查您想要测试的类的方法

+   为生成的测试方法编写带有注解`@Test`的代码

+   根据需要添加带有注解`@Before`和`@After`的方法

“单元”是可以进行测试的最小代码片段，因此得名。最佳的测试实践将方法视为最小可测试单元。这就是为什么单元测试通常是测试方法。

# org.mockito

单元测试经常面临的问题之一是需要测试使用第三方库、数据源或另一个类的方法。在测试时，您希望控制所有输入，以便可以准确预测所测试代码的预期结果。这就是模拟或模拟所测试代码与之交互的对象的行为技术派上用场的地方。

开源框架 Mockito（根包名称为`org.mockito`）允许您正是这样做 - 创建模拟对象。它非常容易和直接。以下是一个简单的案例：

+   在 Maven 配置文件`pom.xml`中添加依赖项

+   调用`mock()`方法以模拟您需要模拟的类：`SomeClass mo = Mockito.mock(SomeClass.class)`

+   设置您需要从方法返回的值：`Mockito.when(mo.doSomething(10)).thenReturn(20)`

+   现在，将模拟对象作为参数传递到您正在测试的方法中，该方法调用了模拟的方法

+   模拟的方法返回您预定义的结果

Mockito 有一些限制。例如，您不能模拟静态方法和私有方法。否则，这是一种可靠地预测所使用方法的结果来隔离您正在测试的代码的绝佳方式。该框架的名称和标志基于单词*mojitos* - 一种饮料。

# org.apache.log4j 和 org.slf4j

在本书中，我们使用`System.out`对象来显示中间和最终结果的输出。在实际应用程序中，也可以这样做，并将输出重定向到文件，例如，以供以后分析。做了一段时间后，您会注意到您需要更多关于每个输出的细节 - 每个语句的日期和时间，或生成此语句的类名，例如。随着代码库的增长，您会发现希望将来自不同子系统或包的输出发送到不同的文件，或在一切正常工作时关闭一些消息，并在检测到问题并需要更详细的代码行为信息时重新打开它们。

可以编写自己的程序来完成所有这些，但是有几个框架可以根据配置文件中的设置来实现，您可以在需要更改消息行为时随时更改配置文件。这些消息称为应用程序日志消息，或应用程序日志，或日志消息，用于此目的最流行的两个框架称为`log4j`（发音为*LOG-FOUR-JAY*）和`slf4j`（发音为*S-L-F-FOUR-JAY*）。

实际上，这两个框架并不是竞争对手。`slf4j`是一个外观，提供对底层实际日志框架的统一访问 - 其中之一也可以是`log4j`。在库开发期间，这样的外观特别有帮助，因为程序员事先不知道使用库的应用程序将使用什么样的日志框架。通过使用`slf4j`编写代码，程序员允许用户以后配置它以使用任何日志系统。

因此，如果您的代码只会被您的团队开发的应用程序使用，并且将在生产中得到支持，那么只使用`log4j`就足够了。否则，请考虑使用`slf4j`。

并且，与任何第三方库一样，在您可以使用任何日志框架之前，您必须向 Maven 配置文件`pom.xml`添加相应的依赖项。

# org.apache.commons

在前一节中，我们谈到了一个带有`org.apache`根名称的包 - 包`org.apache.log4j`。

`org.apache.commons`包是另一个流行的库，代表了一个名为 Apache Commons 的项目，由名为 Apache Software Foundation 的开源程序员社区维护。该组织于 1999 年从 Apache Group 成立。Apache Group 自 1993 年以来一直围绕 Apache HTTP 服务器的开发而成长。Apache HTTP 服务器是一个开源跨平台的网络服务器，自 1996 年 4 月以来一直保持最受欢迎的地位。来自维基百科的一篇文章：

“截至 2016 年 7 月，据估计，它为所有活跃网站的 46%和前 100 万个网站的 43%提供服务。名称“Apache”是出于对美洲印第安纳州阿帕奇族的尊重，他们以卓越的战争策略和不竭的耐力而闻名。它也对“一个补丁式的网络服务器”进行了双关语——一个由一系列补丁组成的服务器——但这并不是它的起源”

Apache Commons 项目有三个部分：

+   **Commons Sandbox**：Java 组件开发的工作空间；您可以在那里为开源做出贡献

+   **Commons Dormant**：一个存储当前不活跃组件的仓库；您可以使用那里的代码，但必须自己构建组件，因为这些组件可能在不久的将来不会发布

+   **Commons Proper**：可重用的 Java 组件，构成了实际的`org.apache.commons`库

在接下来的小节中，我们将只讨论 Commons Proper 最受欢迎的四个包：

+   `org.apache.commons.io`

+   `org.apache.commons.lang`

+   `org.apache.commons.lang3`

+   `org.apache.commons.codec.binary`

然而，在`org.apache.commons`下还有许多包，其中包含了成千上万个有用的类，可以轻松使用，并且可以帮助使您的代码更加优雅和高效。

# org.apache.commons.io

`org.apache.commons.io`包的所有类都包含在根包和五个子包中：

+   根包`org.apache.commons.io`包含了一些实用类，其中包含了执行常见任务的静态方法，比如一个叫做`FileUtils`的流行类，它允许执行所有可能需要的文件操作：

+   写入文件

+   从文件中读取

+   创建目录，包括父目录

+   复制文件和目录

+   删除文件和目录

+   转换为 URL 和从 URL 转换

+   通过过滤器和扩展名列出文件和目录

+   比较文件内容

+   文件最后更改日期

+   计算校验和

+   `org.apache.commons.io.input`包包含支持基于`InputStream`和`Reader`实现的数据输入的类，例如`XmlStreamReader`或`ReversedLinesFileReader`

+   `org.apache.commons.io.output`包包含支持基于`OutputStream`和`Writer`实现的数据输出的类，例如`XmlStreamWriter`或`StringBuilderWriter`

+   `org.apache.commons.io.filefilter`包包含作为文件过滤器的类，例如`DirectoryFileFilter`或`RegexFileFilter`

+   `org.apache.commons.io.comparato`包包含`java.util.Comparator`的各种实现，例如`NameFileComparator`

+   `org.apache.commons.io.monitor`包提供了一个用于监视文件系统事件（目录和文件创建、更新和删除事件）的组件，例如`FileAlterationMonitor`，它实现了`Runnable`并生成一个监视线程，在指定的间隔触发任何注册的`FileAlterationObserver`

# org.apache.commons.lang 和 lang3

`org.apache.commons.lang3`包实际上是`org.apache.commons.lang`包的第 3 个版本。创建新包的决定是由于第 3 版引入的更改是不向后兼容的。这意味着使用先前版本的`org.apache.commons.lang`包的现有应用程序在升级到第 3 版后可能会停止工作。但是，在大多数主流编程中，将 3 添加到导入语句（作为迁移到新版本的方式）可能不会破坏任何东西。

根据文档 <q>"the package org.apache.commons.lang3 provides highly reusable static utility methods, chiefly concerned with adding value to the java.lang classes</q>.<q>"</q> 这里有一些值得注意的例子：

+   `ArrayUtils`类允许搜索和操作数组。

+   `ClassUtils`类提供有关类的一些元数据。

+   `ObjectUtils`类在数组对象中检查`null`，比较对象，并以空安全的方式计算数组对象的中位数和最小/最大值。

+   `SystemUtils`类提供有关执行环境的信息。

+   `ThreadUtils`类查找有关当前运行线程的信息。

+   `Validate`类验证单个值和集合：比较它们，检查`null`，匹配，并执行许多其他验证。

+   `RandomStringUtils`类从各种字符集的字符生成`String`对象。

+   `StringUtils`类是许多程序员的最爱。以下是它提供的空安全操作列表：

+   `isEmpty`/`isBlank`：检查`String`值是否包含文本

+   `trim`/`strip`：删除前导和尾随空格

+   `equals`/`compare`：空安全地比较两个字符串

+   `startsWith`：空安全地检查`String`值是否以特定前缀开头

+   `endsWith`：空安全地检查`String`值是否以特定后缀结尾

+   `indexOf`/`lastIndexOf`/`contains`：提供空安全的索引检查

+   `indexOfAny`/`lastIndexOfAny`/`indexOfAnyBut`/`lastIndexOfAnyBut`：提供一组`String`值中任何一个的索引

+   `containsOnly`/`containsNone`/`containsAny`：检查`String`值是否仅包含/不包含/包含任何特定字符

+   `substring`/`left`/`right`/`mid`：支持空安全的子字符串提取

+   `substringBefore`/`substringAfter`/`substringBetween`：相对于其他字符串执行子字符串提取

+   `split`/`join`：将`String`值按特定分隔符拆分为子字符串数组，反之亦然

+   `remove`/`delete`：删除`String`值的一部分

+   `replace`/`overlay`：搜索`String`值并用另一个`String`值替换

+   `chomp`/`chop`：删除`String`值的最后一部分

+   `appendIfMissing`：如果不存在，则将后缀附加到`String`值的末尾

+   `prependIfMissing`：如果不存在，则将前缀添加到`String`值的开头

+   `leftPad`/`rightPad`/`center`/`repeat`：填充`String`值

+   `upperCase`/`lowerCase`/`swapCase`/`capitalize`/`uncapitalize`：更改`String`值的大小写

+   `countMatches`：计算另一个`String`值在另一个中出现的次数

+   `isAlpha`/`isNumeric`/`isWhitespace`/`isAsciiPrintable`：检查`String`值中的字符

+   `defaultString`：保护免受`null`输入的`String`值

+   `rotate`：旋转（循环移位）`String`值中的字符

+   `reverse`/`reverseDelimited`：反转`String`值中的字符或分隔的字符组

+   `abbreviate`：使用省略号或另一个给定的`String`值缩写`String`值

+   `difference`：比较`String`值并报告它们的差异

+   `levenshteinDistance`：将一个`String`值更改为另一个所需的更改次数

# org.apache.commons.codec.binary

此库的内容超出了本入门课程的范围。因此，我们只会提到该库提供对 Base64、Base32、二进制和十六进制字符串编码和解码的支持。

编码是必要的，以确保您在不同系统之间发送的数据不会因不同协议中字符范围的限制而在传输过程中发生更改。此外，一些系统将发送的数据解释为控制字符（例如调制解调器）。

# 练习-比较 String.indexOf()和 StringUtils.indexOf()

`String`类的`indexOf()`方法和`StringUtils`类的`indexOf()`方法有什么区别？

# 答案

`String`类的`indexOf()`方法不处理`null`。这是一些演示代码：

```java

String s = null;

int i = StringUtils.indexOf(s, "abc");     //返回-1

s.indexOf("abc");                          //抛出 NullPointerException

```

# 总结

在本章中，读者已经了解了 JDK 中包含的 Java 标准库的内容，以及一些最受欢迎的外部库或第三方库。特别是，我们仔细研究了标准包`java.lang`和`java.util`；比较了包`java.io`和`java.nio`，`java.sql`和`javax.sql`，`java.awt`，`javax.swing`和`javafx`；并回顾了包`java.net`和`java.math`。

我们还概述了一些流行的外部库，如`org.junit`，`org.mockito`，`org.apache.log4j`，`org.slf4j`，以及 Apache Commons 项目的几个包：`org.apache.commons.io`，`org.apache.commons.lang`和`org.apache.commons.lang3`，以及`org.apache.commons.codec.binary`。

下一章将帮助读者更详细地了解最常用的 Java 类。代码示例将说明对集合类的功能进行讨论：`List`和`ArrayList`，`Set`和`HashSet`，以及`Map`和`HashMap`。我们还将讨论类`Arrays`和`ArrayUtils`，`Objects`和`ObjectUtils`，`StringBuilder`和`StringBuffer`，`LocalDate`，`LocalTime`和`LocalDateTime`。
