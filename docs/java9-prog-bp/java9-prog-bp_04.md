# 日期计算器

如果你在 Java 中有任何严肃的开发经验，你会知道一件事是真实的——处理日期是糟糕的。`java.util.Date`类及其相关类是在 1.0 版中发布的，`Calendar`及其相关类是在 1.1 版中发布的。甚至在早期，问题就已经显现出来。例如，`Date`的 Javadoc 上说——*不幸的是，这些函数的 API 不适合国际化*。因此，`Calendar`在 1.1 版中被引入。当然，多年来还有其他的增强，但考虑到 Java 对向后兼容性的严格遵守，语言架构师们能做的只有那么多。尽管他们可能想要修复这些 API，但他们的手是被捆绑的。

幸运的是，**Java 规范请求**（**JSR 310**）已经提交。由 Stephen Colebourne 领导，开始了一个努力创建一个新的 API，基于非常流行的开源库 Joda-Time。在本章中，我们将深入研究这个新的 API，然后构建一个简单的命令行实用程序来执行日期和时间计算，这将让我们有机会看到这个 API 的一些实际应用。

因此，本章将涵盖以下主题：

+   Java 8 日期/时间 API

+   重新审视命令行实用程序

+   文本解析

# 入门

就像第二章中的项目，*在 Java 中管理进程*，这个项目在概念上是相当简单的。最终目标是创建一个命令行实用程序来执行各种日期和时间计算。然而，在此过程中，如果实际的日期/时间工作能够被放入一个可重用的库中，那将是非常好的，所以我们将这样做。这给我们留下了两个项目，我们将像上次一样设置为多模块 Maven 项目。

父 POM 将看起来像这样：

```java
    <?xml version="1.0" encoding="UTF-8"?> 
    <project 

      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
      http://maven.apache.org/xsd/maven-4.0.0.xsd"> 
      <modelVersion>4.0.0</modelVersion> 

      <artifactId>datecalc-master</artifactId> 
      <version>1.0-SNAPSHOT</version> 
      <packaging>pom</packaging> 
      <modules> 
        <module>datecalc-lib</module> 
        <module>datecalc-cli</module> 
      </modules> 
    </project> 

```

如果你读过第二章，*在 Java 中管理进程*，或者之前有过多模块 Maven 构建的经验，这里没有什么新的。这只是为了完整性而包含在内。如果这对你来说是陌生的，请花点时间在继续之前回顾第二章的前几页。

# 构建库

由于我们希望能够在其他项目中重用这个工具，我们将首先构建一个公开其功能的库。我们需要的所有功能都内置在平台中，因此我们的 POM 文件非常简单：

```java
    <?xml version="1.0" encoding="UTF-8"?> 
    <project 

      xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
      http://maven.apache.org/xsd/maven-4.0.0.xsd"> 
      <modelVersion>4.0.0</modelVersion> 
      <parent> 
        <groupId>com.steeplesoft</groupId> 
          <artifactId>datecalc-master</artifactId> 
          <version>1.0-SNAPSHOT</version> 
      </parent> 
      <artifactId>datecalc-lib</artifactId> 
      <packaging>jar</packaging> 
      <dependencies> 
        <dependency> 
          <groupId>org.testng</groupId> 
          <artifactId>testng</artifactId> 
          <version>6.9.9</version> 
          <scope>test</scope> 
        </dependency> 
      </dependencies> 
    </project> 

```

几乎没有外部依赖。唯一列出的依赖是测试库 TestNG。在上一章中我们没有谈论太多关于测试（请放心，项目中有测试）。在本章中，我们将介绍测试的主题并展示一些例子。

现在我们需要定义我们的模块。请记住，这些是 Java 9 项目，所以我们希望利用模块功能来帮助保护我们的内部类不被意外公开。我们的模块非常简单。我们需要给它一个名称，然后导出我们的公共 API 包，如下所示：

```java
    module datecalc.lib { 
      exports com.steeplesoft.datecalc; 
    } 

```

由于我们需要的一切都已经在 JDK 中，我们没有什么需要声明的，除了我们导出的内容。

有了我们的项目设置，让我们快速看一下功能需求。我们这个项目的目的是构建一个系统，允许用户提供一个表示日期或时间计算表达式的任意字符串，并得到一个响应。这个字符串可能看起来像`"today + 2 weeks"`，以找出从今天开始的 2 周后的日期，`"now + 3 hours 15 minutes"`，以找出 3 小时 15 分钟后的时间，或者`"2016/07/04 - 1776/07/04"`，以找出两个日期之间的年、月和天数。这些表达式的处理将一次处理一行，因此明确排除了传入多个表达式的文本文档并得到多个结果的能力。当然，任何消费应用程序或库都可以很容易地实现这一点。

现在，我们已经设置好了一个项目，并且准备好了，我们对其相当简单的功能需求有一个大致的草图。我们准备开始编码。在这之前，让我们快速浏览一下新的`java.time`包，以更好地了解我们将在这个项目中看到的内容，以及我们在这个简单项目中**不会**使用的一些功能。

# 及时的插曲

在 Java 8 之前，两个主要的日期相关类是`Date`和`Calendar`（当然还有`GregorianCalendar`）。新的`java.time`包提供了几个新类，如`Duration`、`Period`、`Clock`、`Instant`、`LocalDate`、`LocalTime`、`LocalDateTime`和`ZonedDateTime`。还有大量的支持类，但这些是主要的起点。让我们快速看一下每一个。

# 持续时间

`Duration`是一个**基于时间的时间单位**。虽然用这种方式来表达可能听起来有点奇怪，但选择这种措辞是为了区分它与基于日期的时间单位，我们将在下面看到。简单来说，它是时间的度量，比如**10 秒**、**1 小时**或**100 纳秒**。`Duration`以秒为单位，但有许多方法可以以其他度量单位表示持续时间，如下所示：

+   `getNano()`: 这是以纳秒为单位的`Duration`

+   `getSeconds()`: 这是以秒为单位的`Duration`

+   `get(TemporalUnit)`: 这是以指定的度量单位为单位的`Duration`

还有各种算术方法，如下所述：

+   `add`/`minus (int amount, TemporalUnit unit)`

+   `add`/`minus (Duration)`

+   `addDays`/`minusDays(long)`

+   `addHours`/`minusHours(long)`

+   `addMillis`/`minusMillis(long)`

+   `addMinutes`/`minusMinutes(long)`

+   `addNanos`/`minusNanos(long)`

+   `addSeconds`/`minusSeconds(long)`

+   `dividedBy`/`multipliedBy`

我们还有许多方便的工厂和提取方法，如下所示：

+   `ofDays(long)`/`toDays()`

+   `ofHours(long)`/`toHours()`

+   `ofMinutes(long)`/`toMinutes()`

+   `ofSeconds(long)`/`toSeconds()`

还提供了一个`parse()`方法。不幸的是，对于一些人来说，这个方法的输入可能不是你所期望的。因为我们通常处理的是以小时和分钟为单位的持续时间，你可能期望这个方法接受类似于"1:37"的输入，表示 1 小时 37 分钟。然而，这将导致系统抛出`DateTimeParseException`。这个方法期望接收的是一个 ISO-8601 格式的字符串，看起来像这样--`PnDTnHnMn.nS`。这很棒，不是吗？虽然一开始可能会有点困惑，但一旦你理解了它，就不会太糟糕：

+   第一个字符是可选的`+`（加号）或`-`（减号）符号。

+   下一个字符是`P`，可以是大写或小写。

+   接下来是至少四个部分中的一个，表示天（`D`）、小时（`H`）、分钟（`M`）和秒（`S`）。再次强调，大小写不重要。

+   它们必须按照这个顺序声明。

+   每个部分都有一个数字部分，包括一个可选的`+`或`-`符号，一个或多个 ASCII 数字，以及度量单位指示符。秒数可能是分数（表示为浮点数），可以使用句点或逗号。

+   字母`T`必须出现在小时、分钟或秒的第一个实例之前。

简单吧？对于非技术人员来说可能不太友好，但它支持以字符串编码持续时间，允许无歧义地解析，这是一个巨大的进步。

# 期间

`Period`是一个基于日期的时间单位。而`Duration`是关于时间（小时、分钟、秒等）的，`Period`是关于年、周、月等的。与`Duration`一样，它公开了几种算术方法来添加和减去，尽管这些方法处理的是年、月和日。它还提供了`plus(long amount, TemporalUnit unit)`（以及相应的`minus`）。

此外，就像`Duration`一样，`Period`有一个`parse()`方法，它也采用 ISO-8601 格式，看起来像这样--`PnYnMnD`和`PnW`。根据前面的讨论，结构可能是相当明显的：

+   字符串以一个可选的符号开头，后面跟着字母`P`。

+   在第一种形式之后，有三个部分，至少一个必须存在--年（`Y`）、月（`M`）和日（`D`）。

+   对于第二种形式，只有一个部分--周（`W`）。

+   每个部分的金额可以有正数或负数的符号。

+   `W`单位不能与其他单位组合在一起。在内部，金额乘以`7`并视为天数。

# 时钟

`Clock`是一个抽象类，它提供了使用时区的当前时刻（我们将在下面看到），日期和时间的访问。在 Java 8 之前，我们需要调用`System.currentTimeInMillis()`和`TimeZone.getDefault()`来计算这些值。`Clock`提供了一个很好的接口，可以从一个对象中获取这些值。

Javadoc 声明使用`Clock`纯粹是可选的。事实上，主要的日期/时间类有一个`now()`方法，它使用系统时钟来获取它们的值。然而，如果您需要提供一个替代实现（比如在测试中，您需要另一个时区的`LocalTime`），这个抽象类可以被扩展以提供所需的功能，然后可以传递给适当的`now()`方法。

# Instant

`Instant`是一个单一的、确切的时间点（或**在时间线上**，您会看到 Javadoc 说）。这个类提供了算术方法，就像`Period`和`Duration`一样。解析也是一个选项，字符串是一个 ISO-8601 的时间点格式，比如`1977-02-16T08:15:30Z`。

# LocalDate

`LocalDate`是一个没有时区的日期。虽然这个类的值是一个日期（年、月和日），但是还有其他值的访问器方法，如下所示：

+   `getDayOfWeek()`: 这返回日期表示的星期几的`DayOfWeek`枚举。

+   `getDayOfYear()`: 这返回日期表示的一年中的日子（1 到 365，或闰年的 366）。这是从指定年份的 1 月 1 日开始的基于 1 的计数器。

+   `getEra()`: 这返回给定日期的 ISO 纪元。

当然，本地日期可以从字符串中解析，但是，这一次，格式似乎更加合理--`yyyy-mm-dd`。如果您需要不同的格式，`parse()`方法已经被重写，允许您指定可以处理字符串格式的`DateTimeFormatter`。

# LocalTime

`LocalTime`是`LocalDate`的基于时间的等价物。它存储`HH:MM:SS`，但**不**存储时区。解析时间需要上面的格式，但是，就像`LocalDate`一样，允许您指定一个`DateTimeFormatter`来表示替代字符串。

# LocalDateTime

`LocalDateTime`基本上是最后两个类的组合。所有的算术、工厂和提取方法都按预期应用。解析文本也是两者的组合，只是`T`必须分隔字符串的日期和时间部分--`'2016-01-01T00:00:00'`。这个类**不**存储或表示时区。

# ZonedDateTime

如果您需要表示日期/时间**和**时区，那么`ZonedDateTime`就是您需要的类。正如您可能期望的那样，这个类的接口是`LocalDate`和`LocalTime`的组合，还增加了处理时区的额外方法。

正如在持续时间 API 的概述中所展示的（并且在其他类中也有所暗示，尽管没有那么清楚地显示），这个新 API 的一个强大之处是能够在数学上操作和处理各种日期和时间工件。正是这个功能，我们将在这个项目中花费大部分时间来探索这个新的库。

# 回到我们的代码

我们需要解决的过程的第一部分是将用户提供的字符串解析为我们可以在程序中使用的东西。如果您要搜索解析器生成器，您会发现有很多选择，Antlr 和 JavaCC 等工具通常排在前面。诱人的是转向其中一个工具，但我们这里的目的相当简单，语法并不复杂。我们的功能要求包括：

+   我们希望能够向日期或时间添加/减去时间

+   我们希望能够从另一个日期或时间中减去一个日期或时间，以获得两者之间的差异

+   我们希望能够将一个时区的时间转换为另一个时区

对于这样简单的东西，解析器太昂贵了，无论是在复杂性还是二进制大小方面。我们可以很容易地使用 JDK 内置的工具编写解析器，这就是我们将要做的。

在我们进入代码之前，我们的计划是这样的——我们将定义许多**令牌**来表示日期计算表达式的逻辑部分。使用正则表达式，我们将分解给定的字符串，返回这些令牌的列表，然后按**从左到右**的顺序处理以返回结果。

话虽如此，让我们列出我们需要的令牌类型。我们需要一个日期，一个时间，操作符，任何数字金额，度量单位和时区。显然，我们不需要每个表达式中的每一个，但这应该涵盖我们所有给定的用例。

让我们从我们的令牌的基类开始。在定义类型层次结构时，询问是否需要基类或接口总是一个好主意。使用接口可以使开发人员在需要扩展不同类时对类层次结构具有额外的灵活性。然而，基类允许我们以一定的类型层次结构提供默认行为。为了使我们的`Token`实现尽可能简单，我们希望尽可能多地将其放在基类中，因此我们将使用如下的基类：

```java
    public abstract class Token<T> {
      protected T value;
      public interface Info {
        String getRegex();
        Token getToken(String text);
      }
      public T getValue() {
        return value;
      }
    }

```

Java 8 确实引入了一种从接口提供默认行为的方法，即**默认方法**。默认方法是接口上提供具体实现的方法，这是与接口的重大变化。在这一变化之前，所有接口能做的就是定义方法签名并强制实现类定义方法体。这使我们能够向接口添加方法并提供默认实现，以便接口的现有实现无需更改。在我们的情况下，我们提供的行为是存储一个值（实例变量`value`）和它的访问器（`getValue()`），因此具有默认方法的接口不合适。

请注意，我们还定义了一个嵌套接口`Info`，我们将在解析器部分详细介绍。

有了我们定义的基类，我们现在可以创建我们需要的令牌，如下所示：

```java
    public class DateToken extends Token<LocalDate> { 
      private static final String TODAY = "today"; 
      public static String REGEX = 
        "\\d{4}[-/][01]\\d[-/][0123]\\d|today"; 

```

为了开始这个类，我们定义了两个常量。`TODAY`是一个特殊的字符串，我们将允许用户指定今天的日期。第二个是我们将用来识别日期字符串的正则表达式：

```java
    "\\d{4}[-/][01]\\d[-/][0123]\\d|today" 

```

众所周知，正则表达式很丑陋，就像这些东西一样，这个并不太复杂。我们匹配 4 位数字（`\\d{4}`），或者是-或/（`[-/]`），0 或 1 后面跟任意数字（`[01]\\d`），另一个-或/，然后是 0、1、2 或 3 后面跟任意数字。最后一部分`|today`告诉系统匹配前面的模式，**或**文本`today`。所有这些正则表达式能做的就是识别一个**看起来**像日期的字符串。在当前形式下，它实际上不能确保它是有效的。我们可能可以制作一个可以确保这一点的正则表达式，但是引入的复杂性是不值得的。不过，我们可以让 JDK 为我们验证字符串，这就是我们将在`of`方法中做的。

```java
    public static DateToken of(String text) { 
      try { 
        return TODAY.equals(text.toLowerCase()) ? 
          new DateToken(LocalDate.now()) : 
          new DateToken( 
            LocalDate.parse(text.replace("/", "-"))); 
      } catch (DateTimeParseException ex) { 
          throw new DateCalcException( 
            "Invalid date format: " + text); 
        } 
    } 

```

在这里，我们定义了一个静态方法来处理`DateToken`实例的创建。如果用户提供字符串`today`，我们提供值`LocalDate.now()`，这做了你认为它可能会做的事情。否则，我们将字符串传递给`LocalDate.parse()`，将任何斜杠更改为破折号，因为这是该方法所期望的。如果用户提供了无效的日期，但正则表达式仍然匹配了它，我们将在这里得到一个错误。由于我们内置了支持来验证字符串，我们可以满足于让系统为我们做繁重的工作。

其他标记看起来非常相似。与其展示每个类，其中大部分都会非常熟悉，我们将跳过大部分这些类，只看看正则表达式，因为有些非常复杂。看看以下代码：

```java
    public class IntegerToken extends Token<Integer> { 
      public static final String REGEX = "\\d+"; 

```

嗯，这个不算太糟糕，是吧？这里将匹配一个或多个数字：

```java
    public class OperatorToken extends Token<String> { 
      public static final String REGEX = "\\+|-|to"; 

```

另一个相对简单的，它将匹配+、-或`to`文本：

```java
    public class TimeToken extends Token<LocalTime> { 
      private static final String NOW = "now"; 
      public static final String REGEX = 
        "(?:[01]?\\d|2[0-3]):[0-5]\\d *(?:[AaPp][Mm])?|now"; 

```

正则表达式分解如下：

+   `(?:`：这是一个非捕获组。我们需要将一些规则组合在一起，但我们不希望它们在我们的 Java 代码中处理时显示为单独的组。

+   [01]?：这是 0 或 1。`?`表示这应该发生一次或根本不发生。

+   |2[0-3]：我们要么想匹配前半部分，**或**这一部分，它将是 2 后面跟着 0、1、2 或 3。

+   `)`: 这结束了非捕获组。这个组将允许我们匹配 12 或 24 小时制的时间。

+   `:`：这个位置需要一个冒号。它的存在是不可选的。

+   `[0-5]\\d`：接下来，模式必须匹配一个 0-5 后面跟着另一个数字。这是时间的分钟部分。

+   `' *'`: 很难看到，所以我添加了引号来帮助指示，但我们想匹配 0 个或更多个（由星号表示）空格。

+   `(?:`: 这是另一个非捕获组。

+   `[AaPp][Mm]`：这些是`A`或`P`字母（任何大小写）后面跟着一个`M`（也是任何大小写）。

+   ）：我们结束了非捕获组，但用`?`标记它，以指示它应该发生一次或根本不发生。这个组让我们捕获任何`AM`/`PM`指定。

+   |现在：与上面的 today 一样，我们允许用户指定此字符串以指示当前时间。

同样，这个模式可能匹配一个无效的时间字符串，但我们将让`LocalTime.parse()`在`TimeToken.of()`中为我们处理。

```java
    public static TimeToken of(final String text) { 
      String time = text.toLowerCase(); 
      if (NOW.equals(time)) { 
        return new TimeToken(LocalTime.now()); 
      } else { 
          try { 
            if (time.length() <5) { 
                time = "0" + time; 
            } 
            if (time.contains("am") || time.contains("pm")) { 
              final DateTimeFormatter formatter = 
                new DateTimeFormatterBuilder() 
                .parseCaseInsensitive() 
                .appendPattern("hh:mma") 
                .toFormatter(); 
                return new 
                TimeToken(LocalTime.parse( 
                  time.replaceAll(" ", ""), formatter)); 
            } else { 
                return new TimeToken(LocalTime.parse(time)); 
            } 
          } catch (DateTimeParseException ex) { 
              throw new DateCalcException( 
              "Invalid time format: " + text); 
            } 
        }
    } 

```

这比其他的要复杂一些，主要是因为`LocalTime.parse()`期望的默认格式是 ISO-8601 时间格式。通常，时间是以 12 小时制和上午/下午指定的。不幸的是，这不是 API 的工作方式，所以我们必须进行调整。

首先，如果需要，我们填充小时。其次，我们查看用户是否指定了`"am"`或`"pm"`。如果是这样，我们需要创建一个特殊的格式化程序，这是通过`DateTimeFormatterBuilder`完成的。我们首先告诉构建器构建一个不区分大小写的格式化程序。如果我们不这样做，"AM"将起作用，但"am"将不起作用。接下来，我们附加我们想要的模式，即小时、分钟和上午/下午，然后构建格式化程序。最后，我们可以解析我们的文本，方法是将字符串和格式化程序传递给`LocalTime.parse()`。如果一切顺利，我们将得到一个`LocalTime`实例。如果不行，我们将得到一个`Exception`实例，我们将处理它。请注意，我们在字符串上调用`replaceAll()`。我们这样做是为了去除时间和上午/下午之间的任何空格。否则，解析将失败。

最后，我们来到我们的`UnitOfMeasureToken`。这个标记并不一定复杂，但它肯定不简单。对于我们的度量单位，我们希望支持单词`year`、`month`、`day`、`week`、`hour`、`minute`和`second`，所有这些都可以是复数，大多数都可以缩写为它们的首字母。这使得正则表达式很有趣：

```java
    public class UnitOfMeasureToken extends Token<ChronoUnit> { 
      public static final String REGEX =
        "years|year|y|months|month|weeks|week|w|days|
         day|d|hours|hour|h|minutes|minute|m|seconds|second|s"; 
      private static final Map<String, ChronoUnit> VALID_UNITS = 
        new HashMap<>(); 

```

这并不是很复杂，而是很丑陋。我们有一个可能的字符串列表，由逻辑`OR`运算符竖线分隔。可能可以编写一个正则表达式来搜索每个单词，或者它的部分，但这样的表达式很可能很难编写正确，几乎肯定很难调试或更改。简单和清晰几乎总是比聪明和复杂更好。

这里还有一个需要讨论的最后一个元素：`VALID_UNITS`。在静态初始化程序中，我们构建了一个`Map`，以允许查找正确的`ChronoUnit`：

```java
    static { 
      VALID_UNITS.put("year", ChronoUnit.YEARS); 
      VALID_UNITS.put("years", ChronoUnit.YEARS); 
      VALID_UNITS.put("months", ChronoUnit.MONTHS); 
      VALID_UNITS.put("month", ChronoUnit.MONTHS); 

```

等等。

现在我们准备来看一下解析器，它如下所示：

```java
    public class DateCalcExpressionParser { 
      private final List<InfoWrapper> infos = new ArrayList<>(); 

      public DateCalcExpressionParser() { 
        addTokenInfo(new DateToken.Info()); 
        addTokenInfo(new TimeToken.Info()); 
        addTokenInfo(new IntegerToken.Info()); 
        addTokenInfo(new OperatorToken.Info()); 
        addTokenInfo(new UnitOfMeasureToken.Info()); 
      } 
      private void addTokenInfo(Token.Info info) { 
        infos.add(new InfoWrapper(info)); 
      } 

```

当我们构建我们的解析器时，我们在`List`中注册了每个`Token`类，但我们看到了两种新类型：`Token.Info`和`InfoWrapper`。`Token.Info`是嵌套在`Token`类中的一个接口：

```java
    public interface Info { 
      String getRegex(); 
      Token getToken(String text); 
    } 

```

我们添加了这个接口，以便以方便的方式获取`Token`类的正则表达式，以及`Token`，而不必求助于反射。例如，`DateToken.Info`看起来像这样：

```java
    public static class Info implements Token.Info { 
      @Override 
      public String getRegex() { 
        return REGEX; 
      } 

      @Override 
      public DateToken getToken(String text) { 
        return of(text); 
      } 
    } 

```

由于这是一个嵌套类，我们可以轻松访问包含类的成员，包括静态成员。

下一个新类型，`InfoWrapper`，看起来像这样：

```java
    private class InfoWrapper { 
      Token.Info info; 
      Pattern pattern; 

      InfoWrapper(Token.Info info) { 
        this.info = info; 
        pattern = Pattern.compile("^(" + info.getRegex() + ")"); 
      } 
    } 

```

这是一个简单的私有类，所以一些正常的封装规则可以被搁置（尽管，如果这个类曾经被公开，肯定需要清理一下）。不过，我们正在做的是存储令牌的正则表达式的编译版本。请注意，我们用一些额外的字符包装了正则表达式。第一个是插入符（`^`），表示匹配必须在文本的开头。我们还用括号包装了正则表达式。不过，这次这是一个捕获组。我们将在下面的解析方法中看到为什么要这样做：

```java
    public List<Token> parse(String text) { 
      final Queue<Token> tokens = new ArrayDeque<>(); 

      if (text != null) { 
        text = text.trim(); 
        if (!text.isEmpty()) { 
          boolean matchFound = false; 
          for (InfoWrapper iw : infos) { 
            final Matcher matcher = iw.pattern.matcher(text); 
            if (matcher.find()) { 
              matchFound = true; 
              String match = matcher.group().trim(); 
              tokens.add(iw.info.getToken(match)); 
              tokens.addAll( 
                parse(text.substring(match.length()))); 
                break; 
            } 
          } 
          if (!matchFound) { 
            throw new DateCalcException( 
              "Could not parse the expression: " + text); 
          } 
        } 
      } 

      return tokens; 
    } 

```

我们首先确保`text`不为空，然后`trim()`它，然后确保它不为空。完成了这些检查后，我们循环遍历信息包装器的`List`以找到匹配项。请记住，编译的模式是一个捕获组，查看文本的开头，所以我们循环遍历每个`Pattern`直到找到匹配项。如果我们找不到匹配项，我们会抛出一个`Exception`。

一旦我们找到匹配，我们从`Matcher`中提取匹配的文本，然后使用`Token.Info`调用`getToken()`来获取匹配`Pattern`的`Token`实例。我们将其存储在我们的列表中，然后递归调用`parse()`方法，传递文本的子字符串，从我们的匹配后开始。这将从原始文本中删除匹配的文本，然后重复这个过程，直到字符串为空。一旦递归结束并且事情解开，我们将返回一个代表用户提供的字符串的`Queue`。我们使用`Queue`而不是，比如，`List`，因为这样处理会更容易一些。现在我们有了一个解析器，但我们的工作只完成了一半。现在我们需要处理这些令牌。

在关注关注关注的精神下，我们将这些令牌的处理——实际表达式的计算——封装在一个单独的类`DateCalculator`中，该类使用我们的解析器。考虑以下代码：

```java
    public class DateCalculator { 
      public DateCalculatorResult calculate(String text) { 
        final DateCalcExpressionParser parser = 
          new DateCalcExpressionParser(); 
        final Queue<Token> tokens = parser.parse(text); 

        if (tokens.size() > 0) { 
          if (tokens.peek() instanceof DateToken) { 
            return handleDateExpression(tokens); 
          } else if (tokens.peek() instanceof TimeToken) { 
              return handleTimeExpression(tokens); 
            } 
        } 
        throw new DateCalcException("An invalid expression
          was given: " + text); 
    } 

```

每次调用`calculate()`时，我们都会创建解析器的新实例。另外，请注意，当我们查看代码的其余部分时，我们会传递`Queue`。虽然这确实使方法签名变得有点大，但它也使类线程安全，因为类本身没有保存状态。

在我们的`isEmpty()`检查之后，我们可以看到`Queue` API 的方便之处。通过调用`poll()`，我们可以得到集合中下一个元素的引用，但是——这很重要——**我们保留了集合中的元素**。这让我们可以查看它而不改变集合的状态。根据集合中第一个元素的类型，我们委托给适当的方法。

对于处理日期，表达式语法是`<date> <operator> <date | number unit_of_measure>`。因此，我们可以通过提取`DateToken`和`OperatorToken`来开始我们的处理，如下所示：

```java
    private DateCalculatorResult handleDateExpression( 
      final Queue<Token> tokens) { 
        DateToken startDateToken = (DateToken) tokens.poll(); 
        validateToken(tokens.peek(), OperatorToken.class); 
        OperatorToken operatorToken = (OperatorToken) tokens.poll(); 
        Token thirdToken = tokens.peek(); 

        if (thirdToken instanceof IntegerToken) { 
          return performDateMath(startDateToken, operatorToken,
            tokens); 
        } else if (thirdToken instanceof DateToken) { 
            return getDateDiff(startDateToken, tokens.poll()); 
          } else { 
              throw new DateCalcException("Invalid expression"); 
            } 
    } 

```

从`Queue`中检索元素，我们使用`poll()`方法，我们可以安全地将其转换为`DateToken`，因为我们在调用方法中检查了这一点。接下来，我们`peek()`下一个元素，并通过`validateToken()`方法验证元素不为空且为所需类型。如果令牌有效，我们可以安全地`poll()`和转换。接下来，我们`peek()`第三个令牌。根据其类型，我们委托给正确的方法来完成处理。如果我们发现意外的`Token`类型，我们抛出一个`Exception`。

在查看这些计算方法之前，让我们看一下`validateToken()`：

```java
    private void validateToken(final Token token,
      final Class<? extends Token> expected) { 
        if (token == null || ! 
          token.getClass().isAssignableFrom(expected)) { 
            throw new DateCalcException(String.format( 
              "Invalid format: Expected %s, found %s", 
               expected, token != null ? 
               token.getClass().getSimpleName() : "null")); 
        } 
    } 

```

这里没有太多令人兴奋的东西，但敏锐的读者可能会注意到我们正在返回我们令牌的类名，并且通过这样做，我们向最终用户泄露了一个未导出类的名称。这可能不是理想的，但我们将把修复这个问题留给读者作为一个练习。

执行日期数学的方法如下：

```java
    private DateCalculatorResult performDateMath( 
      final DateToken startDateToken, 
      final OperatorToken operatorToken, 
      final Queue<Token> tokens) { 
        LocalDate result = startDateToken.getValue(); 
        int negate = operatorToken.isAddition() ? 1 : -1; 

        while (!tokens.isEmpty()) { 
          validateToken(tokens.peek(), IntegerToken.class); 
          int amount = ((IntegerToken) tokens.poll()).getValue() *
            negate; 
          validateToken(tokens.peek(), UnitOfMeasureToken.class); 
          result = result.plus(amount, 
          ((UnitOfMeasureToken) tokens.poll()).getValue()); 
        } 

        return new DateCalculatorResult(result); 
    } 

```

由于我们已经有了我们的起始和操作符令牌，我们将它们传递进去，以及`Queue`，以便我们可以处理剩余的令牌。我们的第一步是确定操作符是加号还是减号，根据需要给`negate`分配正数`1`或负数`-1`。我们这样做是为了能够使用一个方法`LocalDate.plus()`。如果操作符是减号，我们添加一个负数，得到与减去原始数相同的结果。

最后，我们循环遍历剩余的令牌，在处理之前验证每一个。我们获取`IntegerToken`；获取其值；将其乘以我们的负数修饰符`negate`，然后使用`UnitOfMeasureToken`将该值添加到`LocalDate`中，以告诉我们正在添加的值的**类型**。

计算日期之间的差异非常简单，如下所示：

```java
    private DateCalculatorResult getDateDiff( 
      final DateToken startDateToken, final Token thirdToken) { 
        LocalDate one = startDateToken.getValue(); 
        LocalDate two = ((DateToken) thirdToken).getValue(); 
        return (one.isBefore(two)) ? new
          DateCalculatorResult(Period.between(one, two)) : new
            DateCalculatorResult(Period.between(two, one)); 
    } 

```

我们从两个`DateToken`变量中提取`LocalDate`，然后调用`Period.between()`，它返回一个指示两个日期之间经过的时间量的`Period`。我们确实检查了哪个日期先出现，以便向用户返回一个正的`Period`，作为一种便利，因为大多数人通常不会考虑负周期。

基于时间的方法基本相同。最大的区别是时间差异方法：

```java
    private DateCalculatorResult getTimeDiff( 
      final OperatorToken operatorToken, 
      final TimeToken startTimeToken, 
      final Token thirdToken) throws DateCalcException { 
        LocalTime startTime = startTimeToken.getValue(); 
        LocalTime endTime = ((TimeToken) thirdToken).getValue(); 
        return new DateCalculatorResult( 
          Duration.between(startTime, endTime).abs()); 
    } 

```

这里值得注意的区别是使用了`Duration.between()`。它看起来与`Period.between()`相同，但`Duration`类提供了一个`Period`没有的方法：`abs()`。这个方法让我们返回`Period`的绝对值，所以我们可以按任何顺序将我们的`LocalTime`变量传递给`between()`。

在我们离开之前的最后一点注意事项是--我们将结果封装在`DateCalculatorResult`实例中。由于各种操作返回几种不同的、不相关的类型，这使我们能够从我们的`calculate()`方法中返回一个单一类型。由调用代码来提取适当的值。我们将在下一节中查看我们的命令行界面。

# 关于测试的简短插曲

在我们继续之前，我们需要讨论一个我们尚未讨论过的话题，那就是测试。在这个行业工作了一段时间的人很可能听说过**测试驱动开发**（或简称**TDD**）这个术语。这是一种软件开发方法，认为应该首先编写一个测试，这个测试将失败（因为没有代码可以运行），然后编写使测试**通过**的代码，这是指 IDE 和其他工具中给出的绿色指示器，表示测试已经通过。这个过程根据需要重复多次来构建最终的系统，总是以小的增量进行更改，并始终从测试开始。关于这个主题已经有大量的书籍写成，这个主题既备受争议，又常常被严格细分。这种方法的确切实现方式，如果有的话，几乎总是有不同的版本。

显然，在我们的工作中，我们并没有严格遵循 TDD 原则，但这并不意味着我们没有进行测试。虽然 TDD 纯粹主义者可能会挑剔，但我的一般方法在测试方面可能会有些宽松，直到我的 API 开始变得稳定为止。这需要多长时间取决于我对正在使用的技术的熟悉程度。如果我对它们非常熟悉，我可能会草拟一个快速的接口，然后基于它构建一个测试，作为测试 API 本身的手段，然后对其进行迭代。对于新的库，我可能会编写一个非常广泛的测试，以帮助驱动对新库的调查，使用测试框架作为引导运行环境的手段，以便我可以进行实验。无论如何，在开发工作结束时，新系统应该经过**完全**测试（**完全**的确切定义是另一个备受争议的概念），这正是我在这里努力追求的。关于测试和测试驱动开发的全面论述超出了我们的范围。

在 Java 中进行测试时，你有很多选择。然而，最常见的两种是 TestNG 和 JUnit，其中 JUnit 可能是最受欢迎的。你应该选择哪一个？这取决于情况。如果你正在处理一个现有的代码库，你可能应该使用已经在使用的东西，除非你有充分的理由做出其他选择。例如，该库可能已经过时并且不再受支持，它可能明显不符合你的需求，或者你已经得到了明确的指令来更新/替换现有系统。如果这些条件中的任何一个，或者类似这些的其他条件是真实的，我们就回到了这个问题--*我应该选择哪一个？*同样，这取决于情况。JUnit 非常受欢迎和常见，因此在项目中使用它可能是有道理的，以降低进入项目的门槛。然而，一些人认为 TestNG 具有更好、更清晰的 API。例如，TestNG 不需要对某些测试设置方法使用静态方法。它还旨在不仅仅是一个单元测试框架，还提供了用于单元、功能、端到端和集成测试的工具。在这里，我们将使用 TestNG 进行测试。

要开始使用 TestNG，我们需要将其添加到我们的项目中。为此，我们将在 Maven POM 文件中添加一个测试依赖项，如下所示：

```java
    <properties>
      <testng.version>6.9.9</testng.version>
    </properties>
    <dependencies> 
      <dependency> 
        <groupId>org.testng</groupId>   
        <artifactId>testng</artifactId>   
        <version>${testng.version}</version>   
        <scope>test</scope> 
      </dependency>   
    </dependencies> 

```

编写测试非常简单。使用 TestNG Maven 插件的默认设置，类只需要在`src/test/java`中，并以`Test`字符串结尾。每个测试方法都需要用`@Test`进行注释。

库模块中有许多测试，所以让我们从一些非常基本的测试开始，这些测试测试了标记使用的正则表达式，以识别和提取表达式的相关部分。例如，考虑以下代码片段：

```java
    public class RegexTest { 
      @Test 
      public void dateTokenRegex() { 
        testPattern(DateToken.REGEX, "2016-01-01"); 
        testPattern(DateToken.REGEX, "today"); 
      } 
      private void testPattern(String pattern, String text) { 
        testPattern(pattern, text, false); 
      } 

      private void testPattern(String pattern, String text, 
        boolean exact) { 
          Pattern p = Pattern.compile("(" + pattern + ")"); 
          final Matcher matcher = p.matcher(text); 

          Assert.assertTrue(matcher.find()); 
          if (exact) { 
            Assert.assertEquals(matcher.group(), text); 
          } 
      } 

```

这是对`DateToken`正则表达式的一个非常基本的测试。测试委托给`testPattern()`方法，传递要测试的正则表达式和要测试的字符串。我们的功能通过以下步骤进行测试：

1.  编译`Pattern`。

1.  创建`Matcher`。

1.  调用`matcher.find()`方法。

有了这个，被测试系统的逻辑就得到了执行。剩下的就是验证它是否按预期工作。我们通过调用`Assert.assertTrue()`来做到这一点。我们断言`matcher.find()`返回`true`。如果正则表达式正确，我们应该得到一个`true`的响应。如果正则表达式不正确，我们将得到一个`false`的响应。在后一种情况下，`assertTrue()`将抛出一个`Exception`，测试将失败。

这个测试确实非常基础。它可能——应该——更加健壮。它应该测试更多种类的字符串。它应该包括一些已知的坏字符串，以确保我们在测试中没有得到错误的结果。可能还有许多其他的增强功能可以实现。然而，这里的重点是展示一个简单的测试，以演示如何设置基于 TestNG 的环境。在继续之前，让我们看几个更多的例子。

这是一个用于检查失败的测试（**负面测试**）：

```java
    @Test 
    public void invalidStringsShouldFail() { 
      try { 
        parser.parse("2016/12/25 this is nonsense"); 
        Assert.fail("A DateCalcException should have been
          thrown (Unable to identify token)"); 
      } catch (DateCalcException dce) { 
      } 
    } 

```

在这个测试中，我们期望调用`parse()`失败，并抛出一个`DateCalcException`。如果调用**没有**失败，我们会调用`Assert.fail()`，强制测试失败并提供消息。如果抛出了`Exception`，它会被悄悄地吞没，测试将成功结束。

吞没`Exception`是一种方法，但你也可以告诉 TestNG 期望抛出一个`Exception`，就像我们在这里通过`expectedExceptions`属性所做的那样：

```java
    @Test(expectedExceptions = {DateCalcException.class}) 
    public void shouldRejectBadTimes() { 
      parser.parse("22:89"); 
    } 

```

同样，我们将一个坏的字符串传递给解析器。然而，这一次，我们通过注解告诉 TestNG 期望抛出异常——`@Test(expectedExceptions = {DateCalcException.class})`。

关于测试一般和特别是 TestNG，还可以写更多。对这两个主题的全面讨论超出了我们的范围，但如果你对任何一个主题不熟悉，最好找到其中的一些优秀资源并进行深入学习。

现在，让我们把注意力转向命令行界面。

# 构建命令行界面

在上一章中，我们使用了 Tomitribe 的 Crest 库构建了一个命令行工具，并且效果非常好，所以我们将在构建这个命令行时再次使用这个库。

要在我们的项目中启用 Crest，我们必须做两件事。首先，我们必须按照以下方式配置我们的 POM 文件：

```java
    <dependency> 
      <groupId>org.tomitribe</groupId> 
      <artifactId>tomitribe-crest</artifactId> 
      <version>0.8</version> 
    </dependency> 

```

我们还必须按照以下方式更新`src/main/java/module-info.java`中的模块定义：

```java
    module datecalc.cli { 
      requires datecalc.lib; 
      requires tomitribe.crest; 
      requires tomitribe.crest.api; 

      exports com.steeplesoft.datecalc.cli; 
    } 

```

我们现在可以像这样定义我们的 CLI 类：

```java
    public class DateCalc { 
      @Command 
      public void dateCalc(String... args) { 
        final String expression = String.join(" ", args); 
        final DateCalculator dc = new DateCalculator(); 
        final DateCalculatorResult dcr = dc.calculate(expression); 

```

与上一章不同，这个命令行将非常简单，因为我们唯一需要的输入是要评估的表达式。通过前面的方法签名，我们告诉 Crest 将所有命令行参数作为`args`值传递，然后我们通过`String.join()`将它们重新连接成`expression`。接下来，我们创建我们的计算器并计算结果。

现在我们需要询问我们的`DateCalcResult`来确定表达式的性质。考虑以下代码片段作为示例：

```java
    String result = ""; 
    if (dcr.getDate().isPresent()) { 
      result = dcr.getDate().get().toString(); 
    } else if (dcr.getTime().isPresent()) { 
      result = dcr.getTime().get().toString(); 
    } else if (dcr.getDuration().isPresent()) { 
      result = processDuration(dcr.getDuration().get()); 
    } else if (dcr.getPeriod().isPresent()) { 
      result = processPeriod(dcr.getPeriod().get()); 
    } 
    System.out.println(String.format("'%s' equals '%s'", 
      expression, result)); 

```

`LocalDate`和`LocalTime`的响应非常直接——我们可以简单地调用它们的`toString()`方法，因为默认值对于我们的目的来说是完全可以接受的。`Duration`和`periods`则更加复杂。两者都提供了许多提取细节的方法。我们将把这些细节隐藏在单独的方法中：

```java
    private String processDuration(Duration d) { 
      long hours = d.toHoursPart(); 
      long minutes = d.toMinutesPart(); 
      long seconds = d.toSecondsPart(); 
      String result = ""; 

      if (hours > 0) { 
        result += hours + " hours, "; 
      } 
      result += minutes + " minutes, "; 
      if (seconds > 0) { 
        result += seconds + " seconds"; 
      } 

      return result; 
    } 

```

这个方法本身非常简单——我们从`Duration`中提取各个部分，然后根据部分是否返回值来构建字符串。

与日期相关的`processPeriod()`方法类似：

```java
    private String processPeriod(Period p) { 
      long years = p.getYears(); 
      long months = p.getMonths(); 
      long days = p.getDays(); 
      String result = ""; 

      if (years > 0) { 
        result += years + " years, "; 
      } 
      if (months > 0) { 
        result += months + " months, "; 
      } 
      if (days > 0) { 
        result += days + " days"; 
      } 
      return result; 
    } 

```

这些方法中的每一个都将结果作为字符串返回，然后我们将其写入标准输出。就是这样。这不是一个非常复杂的命令行实用程序，但这里的练习目的主要在于库中。

# 总结

我们的日期计算器现在已经完成。这个实用程序本身并不是太复杂，尽管它确实如预期般发挥作用，这必须成为尝试使用 Java 8 的日期/时间 API 的工具。除了新的日期/时间 API，我们还初步了解了正则表达式，这是一种非常强大和复杂的工具，用于解析字符串。我们还重新访问了上一章的命令行实用程序库，并在单元测试和测试驱动开发的领域涉足了一点。

在下一章中，我们将更加雄心勃勃地进入社交媒体的世界，构建一个应用程序，帮助我们将一些喜爱的服务聚合到一个单一的应用程序中。
