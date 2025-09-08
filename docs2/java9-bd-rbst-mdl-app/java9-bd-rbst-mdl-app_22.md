# 日期计算器

如果你认真开发 Java 有一段时间了，你知道一件事是真实的——处理日期是糟糕的。`java.util.Date`类及其相关类在 1.0 版本中发布，`Calendar`及其相关类在 1.1 版本中推出。即使在早期，问题就很明显。例如，`Date`的 Javadoc 说：“不幸的是，这些函数的 API 不适合国际化。”因此，`Calendar`在 1.1 版本中被引入。当然，多年来已经进行了其他增强，但鉴于 Java 对向后兼容性的严格遵循，语言架构师能做的事情是有限的。尽管他们可能想修复这些 API，但他们的手被束缚了。

幸运的是，**Java 规范请求**（**JSR 310**）已被提交。由 Stephen Colebourne 领导，开始了一个创建新 API 的努力，该 API 基于非常流行的开源库 Joda-Time。在本章中，我们将深入探讨这个新 API，然后构建一个简单的命令行工具来执行日期和时间计算，这将给我们一个机会看到一些 API 的实际应用。

因此，本章将涵盖以下主题：

+   Java 8 日期/时间 API

+   重新审视命令行工具

+   文本解析

# 开始

与第十八章中的项目类似，管理 Java 中的进程，这个项目在概念上相当简单。最终目标是创建一个命令行工具，用于执行各种日期和时间计算。然而，既然我们在做这件事，将实际的日期/时间工作放入一个可重用的库中将会非常棒，所以这就是我们将要做的。这让我们剩下两个项目，我们将像上次一样，将其设置为一个多模块 Maven 项目。

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

如果你阅读过第十八章，*管理 Java 中的进程*，或者之前使用过多模块 Maven 构建，这里没有什么新的内容。它只是为了完整性而包含的。如果你对此感到陌生，请在继续之前花点时间回顾第十八章的前几页。

# 构建库

由于我们希望能够在其他项目中重用这个工具，我们将首先构建一个暴露其功能的库。我们需要的所有功能都内置在平台中，所以我们的 POM 文件非常简单：

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

几乎没有外部依赖。列出的唯一依赖是测试库 TestNG。在上一个章节中我们没有过多地讨论测试（请放心，项目中确实有测试）。在本章中，我们将介绍测试的话题，并展示一些示例。

现在我们需要定义我们的模块。记住，这些是 Java 9 项目，所以我们要利用模块功能来帮助保护我们的内部类免受意外公开暴露。我们的模块非常简单。我们需要给它一个名字，然后导出我们的公共 API 包，如下所示：

```java
    module datecalc.lib { 
      exports com.steeplesoft.datecalc; 
    } 
```

由于我们所需的一切都已经包含在 JDK 中，所以我们不需要声明除了我们导出之外的内容。

在我们的项目设置完成后，让我们快速看一下功能需求。我们这个项目的意图是构建一个系统，允许用户提供一个表示日期或时间计算表达式的任意字符串，并得到一个响应。这个字符串可能看起来像`"today + 2 weeks"`来找出今天之后的 2 周日期，`"now + 3 hours 15 minutes"`来找出 3 小时 15 分钟后的时间，或者`"2016/07/04 - 1776/07/04"`来找出两个日期之间有多少年、月和日。这些表达式的处理将逐行进行，因此明确排除了例如传递包含多个表达式的文本文档并得到多个结果的能力。当然，任何消费应用程序或库都可以轻松实现这一点。

因此，现在我们的项目已经设置好并准备就绪，我们也对其相对简单的功能需求有一个大致的草图。我们准备开始编码。在我们这样做之前，让我们快速浏览一下新的`java.time`包，以便更好地了解在这个项目中我们将看到什么，以及在这个简单项目中我们将不会使用的一些功能。

# 一个及时的小憩

在 Java 8 之前，主要的两个日期相关类是`Date`和`Calendar`（当然，还有`GregorianCalendar`）。新的`java.time`包提供了几个新的类，例如`Duration`、`Period`、`Clock`、`Instant`、`LocalDate`、`LocalTime`、`LocalDateTime`和`ZonedDateTime`。有许多支持类，但这些都是主要的起点。让我们快速看一下每个类。

# 持续时间

`Duration`是一个基于时间的单位。虽然这样表述可能听起来有些奇怪，但这样的措辞是为了将其与基于日期的时间单位区分开来，我们将在下一节中讨论。用简单的话说，它是一种时间的度量，例如**10 秒**、**1 小时**或**100 纳秒**。`Duration`是以秒为单位的，但有一些方法可以将持续时间表示为其他度量单位，如下所示：

+   `getNano()`: 这是基于纳秒的`Duration`

+   `getSeconds()`: 这是基于秒的`Duration`

+   `get(TemporalUnit)`: 这是基于指定度量单位的`Duration`

此外，还有一些不同的算术方法，如下所述：

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

还提供了一个`parse()`方法。不幸的是，也许对某些人来说，这个方法输入可能不是你预期的。因为我们处理的是通常以小时和分钟为单位的持续时间，你可能会期望这个方法接受类似于"1:37"的输入来表示 1 小时 37 分钟。然而，这将导致系统抛出`DateTimeParseException`。该方法期望接收的是符合 ISO-8601 格式的字符串，看起来像这样--`PnDTnHnMn.nS`。这相当神奇，不是吗？虽然一开始可能会感到困惑，但一旦理解了它，就不会太糟糕：

+   第一个字符是一个可选的`+`（加号）或`-`（减号）符号。

+   下一个字符是`P`，可以是大写或小写。

+   接下来至少有一个四个部分，表示天数(`D`)、小时(`H`)、分钟(`M`)和秒(`S`)。同样，大小写无关紧要。

+   它们必须按照这个顺序声明。

+   每个部分都有一个数字部分，包括可选的`+`或`-`符号，一个或多个 ASCII 数字，以及度量单位指示符。秒数可以是分数（以浮点数表示）并且可以使用点或逗号。

+   字母`T`必须出现在小时、分钟或秒的第一个实例之前。

简单，对吧？它可能对非技术受众不太友好，但它支持将持续时间编码为字符串，允许无歧义的解析，这是一个巨大的进步。

# 期间

`Period`是一个基于日期的时间单位。而`Duration`是关于时间（小时、分钟、秒等），`Period`是关于年、周、月等。像`Duration`一样，它公开了几个算术方法来添加和减去，尽管这些处理的是年、月和日。它还提供了`plus(long amount, TemporalUnit unit)`（以及等效的`minus`）。

此外，与`Duration`类似，`Period`也有一个`parse()`方法，它也接受类似于`PnYnMnD`和`PnW`的 ISO-8601 格式。根据前面的讨论，结构可能非常明显：

+   字符串以可选的符号开头，后跟字母`P`。

+   之后，对于第一种形式，有三个部分，其中至少有一个必须存在--年(`Y`)、月(`M`)和日(`D`)。

+   对于第二种形式，只有一个部分--周(`W`)。

+   每个部分的量可以带有正号或负号。

+   `W`单位不能与其他单位组合。内部，量乘以`7`并作为天数处理。

# 时钟

`Clock`是一个抽象类，它提供了一个使用时区访问当前时刻（我们将在下一节看到）、日期和时间的接口。在 Java 8 之前，我们必须调用`System.currentTimeInMillis()`和`TimeZone.getDefault()`来计算这些值。`Clock`提供了一个很好的接口，可以从一个对象中获取这些值。

Javadoc 表示使用 `Clock` 是纯粹可选的。实际上，主要的日期/时间类都有一个 `now()` 方法，它使用系统时钟来获取它们的值。然而，如果你需要提供替代实现（比如，在测试中，你需要另一个时区的 `LocalTime`），这个抽象类可以被扩展以提供所需的功能，然后可以传递给适当的 `now()` 方法。

# Instant

An `Instant` 是一个单一、精确的时间点（或者**在时间线上**，你会在 Javadoc 中看到）。这个类提供了类似于 `Period` 和 `Duration` 的算术方法。解析也是一个选项，字符串是一个 ISO-8601 即时格式，如 `1977-02-16T08:15:30Z`。

# LocalDate

`LocalDate` 是一个不带时区的日期。虽然这个类的值是一个日期（年、月和日），但还有其他值的访问器方法，如下所示：

+   `getDayOfWeek()`: 这返回由日期表示的星期的 `DayOfWeek` 枚举。

+   `getDayOfYear()`: 这返回由日期表示的年份（1 到 365，闰年为 366）。这是一个从指定年份 1 月 1 日起的 1 为基础的计数器。

+   `getEra()`: 这返回给定日期的 ISO 时代。

当然，本地日期可以从字符串解析，但这次，格式似乎更加合理--`yyyy-mm-dd`。如果你需要不同的格式，`parse()` 方法已被重写，允许你指定可以处理字符串格式的 `DateTimeFormatter`。

# LocalTime

`LocalTime` 是 `LocalDate` 的基于时间的等效物。它存储 `HH:MM:SS`，但**不**存储时区。解析时间需要上述格式，但就像 `LocalDate` 一样，也允许你为不同的字符串表示指定 `DateTimeFormatter`。

# LocalDateTime

`LocalDateTime` 实际上是后两个类的组合。所有的算术、工厂和提取方法都按预期应用。解析文本也是两者的组合，但 `T` 必须分隔字符串的日期和时间部分--`'2016-01-01T00:00:00'`。这个类**不**存储或表示时区。

# ZonedDateTime

如果你需要表示日期/时间和时区，那么你需要的是 `ZonedDateTime` 类。正如你所期望的，这个类的接口是 `LocalDate` 和 `LocalTime` 的组合，并添加了处理时区的一些额外方法。

如在持续时间 API 概述中详细展示（尽管在其他类中暗示，但并不那么明显），这个新 API 的一个强点是能够以数学方式操作和处理各种日期和时间元素。正是这个功能，我们将在这个项目中花费大部分时间来探索这个新库。

# 回到我们的代码

我们需要解决的过程的第一部分是将用户提供的字符串解析成我们可以编程使用的东西。如果你去搜索解析器生成器，你会找到许多选项，其中 Antlr 和 JavaCC 等工具通常会出现在顶部。转向这些工具之一很有吸引力，但我们的目的在这里相当简单，语法也不是特别复杂。我们的功能需求包括：

+   我们希望能够向日期或时间添加/减去时间

+   我们希望能够从一个日期或时间减去另一个日期或时间，以获取两个日期或时间的差值

+   我们希望能够将一个时区的时间转换到另一个时区

对于这样简单的事情，解析器在复杂性和二进制大小方面都过于昂贵。我们可以轻松地使用 JDK 内置的工具编写解析器，这正是我们将要做的。

在我们进入代码之前，计划是这样的——我们将定义许多**标记**来表示日期计算表达式中的逻辑部分。使用正则表达式，我们将分解给定的字符串，返回一个这些标记的列表，然后这些标记将被**从左到右**处理以返回结果。

话虽如此，让我们列出我们需要标记的类型。我们需要一个用于日期，一个用于时间，一个用于运算符，任何数值量，度量单位和时区。显然，我们不需要在每一个表达式中都需要这些，但这应该涵盖了我们的所有给定用例。

让我们从我们的标记的基础类开始。在定义类型层次结构时，总是很好地问自己是否需要一个基础类或接口。使用接口给开发者提供了额外的灵活性，如果需要扩展不同的类，那么在类层次结构中。然而，基础类允许我们在类型层次结构中提供默认行为，但以牺牲一些刚性为代价。为了使我们的`Token`实现尽可能简单，我们希望尽可能地将内容放在基础类中，所以我们将以以下方式使用基础类：

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

Java 8 确实引入了一种从接口提供默认行为的方法，即**默认方法**。默认方法是一个接口上的方法，它提供了一个具体的实现，这与接口有显著的不同。在此更改之前，所有接口所能做的就是定义方法签名并强制实现类定义方法体。这允许我们在接口中添加方法并提供默认实现，这样现有的接口实现就不需要改变。在我们的情况下，我们提供的行为是存储一个值（实例变量`value`）及其访问器（`getValue()`），因此具有默认方法的接口是不合适的。

注意，我们还定义了一个嵌套接口，`Info`，当我们到达解析器时，我们将更详细地介绍它。

在定义了基本类之后，我们现在可以创建我们需要的标记，如下所示：

```java
    public class DateToken extends Token<LocalDate> { 
      private static final String TODAY = "today"; 
      public static String REGEX = 
        "\d{4}[-/][01]\d[-/][0123]\d|today"; 
```

要开始这个类，我们定义了两个常量。`TODAY`是一个特殊字符串，我们将允许用户指定今天的日期。第二个是我们将用来识别日期字符串的正则表达式：

```java
    "\d{4}[-/][01]\d[-/][0123]\d|today" 
```

没有人会否认正则表达式很丑陋，而且按照这些规则，这个并不太复杂。我们正在匹配 4 个数字（`\d{4}`），要么是一个破折号或斜杠（`[-/]`），要么是一个 0 或 1 后面跟着任意数字（`[01]\d`），然后是另一个破折号或斜杠，接着是一个 0、1、2 或 3 后面跟着任意数字。最后，最后一个部分，`|today`，告诉系统匹配前面的模式，**或者**文本`today`。这个正则表达式所能做的只是识别一个看起来像日期的字符串。在其当前形式下，它实际上不能确保它是有效的。我们可能可以创建一个可以做到这一点的正则表达式，但引入的复杂性并不值得。不过，我们可以让 JDK 为我们验证字符串，就像在这里的`of`方法中所示：

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

在这里，我们定义了一个静态方法来处理`DateToken`实例的创建。如果用户提供了字符串`today`，我们提供`LocalDate.now()`的值，它可能做你想的事情。否则，我们将字符串传递给`LocalDate.parse()`，将任何正斜杠转换为破折号，因为这是该方法所期望的。如果用户提供了无效的日期，但正则表达式仍然匹配它，我们会在这里得到一个错误。由于我们有内置的支持来验证字符串，我们可以放心地让系统为我们做繁重的工作。

其他标记看起来非常相似。我们不会展示每个类，因为其中大部分都非常熟悉，我们将跳过大多数这些类，只看看正则表达式，因为其中一些相当复杂。看看以下代码：

```java
    public class IntegerToken extends Token<Integer> { 
      public static final String REGEX = "\d+"; 
```

嗯，那个并不太糟糕，对吧？这里将匹配一个或多个数字：

```java
    public class OperatorToken extends Token<String> { 
      public static final String REGEX = "\+|-|to"; 
```

另一个相对简单的，它将匹配一个加号、一个减号或`to`文本：

```java
    public class TimeToken extends Token<LocalTime> { 
      private static final String NOW = "now"; 
      public static final String REGEX = 
        "(?:[01]?\d|2[0-3]):[0-5]\d *(?:[AaPp][Mm])?|now"; 
```

正则表达式分解如下：

+   `(?:`: 这是一个非捕获组。我们需要将一些规则组合在一起，但我们不希望在 Java 代码处理时将它们显示为单独的组。

+   `[01]?`: 这是一个零或一个一。`?`表示这可能发生一次或根本不发生。

+   `|2[0-3]`: 我们要么匹配前半部分，**或者**这个部分，它将是一个 2 后面跟着一个 0、1、2 或 3。

+   `)`: 这结束了非捕获组。这个组将允许我们匹配 12 小时或 24 小时的时间。

+   `:`: 这个位置需要一个冒号。它的存在不是可选的。

+   `[0-5]\d`: 接下来，模式必须匹配一个`0-5`的数字后面跟着另一个数字。这是时间的分钟部分。

+   `' *'`: 这很难看，所以我添加了引号来帮助指示，但我们想匹配 0 个或多个（如星号所示）空格。

+   `(?:`: 这是另一个非捕获组。

+   `[AaPp][Mm]`: 这些是`A`或`P`字母（任意大小写）后面跟着一个`M`（也是任意大小写）。

+   `)?`：我们结束非捕获组，但用`?`标记它，表示它应该出现一次或不出现。这个组让我们能够捕获任何`AM`/`PM`标识。

+   `|now`：与上面提到的今天类似，我们允许用户指定此字符串以指示当前时间。

再次强调，这个模式可能会匹配一个无效的时间字符串，但我们将让`LocalTime.parse()`在`TimeToken.of()`中为我们处理这个问题：

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

这比其他模式复杂一些，主要是因为`LocalTime.parse()`期望的默认格式，这是一个 ISO-8601 时间格式。通常，时间以 12 小时格式指定，带有 am/pm 标识。不幸的是，API 并不是这样工作的，所以我们必须进行调整。

首先，如果需要，我们填充小时。其次，我们查看用户是否指定了`"am"`或`"pm"`。如果是这样，我们需要创建一个特殊的格式化器，这是通过`DateTimeFormatterBuilder`完成的。我们首先告诉构建器构建一个不区分大小写的格式化器。如果我们不这样做，`"AM"`将工作，但`"am"`将不会。接下来，我们附加我们想要的模式，即小时、分钟和 am/pm，然后构建格式化器。最后，我们可以解析我们的文本，这是通过将字符串和格式化器传递给`LocalTime.parse()`来完成的。如果一切顺利，我们将得到一个`LocalTime`实例。如果不顺利，我们将得到一个`Exception`实例，我们将处理它。注意，我们在我们的字符串上调用`replaceAll()`。我们这样做是为了去除时间和 am/pm 之间的任何空格。否则，解析将失败。

最后，我们来到我们的`UnitOfMeasureToken`。这个标记不一定复杂，但绝对不简单。对于我们的度量单位，我们希望支持单词`year`、`month`、`day`、`week`、`hour`、`minute`和`second`，所有这些都可以是复数形式，并且大多数可以缩写为其首字母。这使得正则表达式变得有趣：

```java
    public class UnitOfMeasureToken extends Token<ChronoUnit> { 
      public static final String REGEX =
        "years|year|y|months|month|weeks|week|w|days|
         day|d|hours|hour|h|minutes|minute|m|seconds|second|s"; 
      private static final Map<String, ChronoUnit> VALID_UNITS = 
        new HashMap<>(); 
```

这并不是那么复杂，而是丑陋。我们有一个可能的字符串列表，由逻辑“或”运算符，即竖线分隔。可能可以编写一个正则表达式来搜索每个单词或其部分，但这样的表达式可能非常难以正确编写，并且几乎肯定难以调试或更改。简单明了通常比巧妙复杂要好。

这里还有一个需要讨论的最后元素：`VALID_UNITS`。在静态初始化器中，我们构建一个`Map`以允许查找正确的`ChronoUnit`：

```java
    static { 
      VALID_UNITS.put("year", ChronoUnit.YEARS); 
      VALID_UNITS.put("years", ChronoUnit.YEARS); 
      VALID_UNITS.put("months", ChronoUnit.MONTHS); 
      VALID_UNITS.put("month", ChronoUnit.MONTHS); 
```

等等。

我们现在可以查看解析器了，如下所示：

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

当我们构建我们的解析器时，我们在一个`List`中注册每个`Token`类，但我们看到两种新的类型：`Token.Info`和`InfoWrapper`。`Token.Info`是嵌套在`Token`类中的一个接口：

```java
    public interface Info { 
      String getRegex(); 
      Token getToken(String text); 
    } 
```

我们添加了这个接口，以便我们能够方便地获取`Token`类的正则表达式以及`Token`，而无需求助于反射。例如，`DateToken.Info`看起来是这样的：

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

由于这是一个嵌套类，我们可以轻松访问包括静态成员在内的封装类成员。

下一个新类型`InfoWrapper`看起来如下：

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

这是一个简单、私有的类，因此可以暂时忽略一些常规的封装规则（尽管，如果这个类将来被公开，这肯定需要清理）。不过，我们正在存储一个标记的正则表达式的编译版本。请注意，我们在正则表达式周围添加了一些额外的字符。第一个是尖括号（`^`），表示匹配必须位于文本的开头。我们还将正则表达式放在括号中。然而，这次这是一个捕获组。我们将在接下来的解析方法中看到原因：

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

我们首先确保`text`不是 null，然后`trim()`它，然后确保它不是空的。在完成这些合理性检查后，我们遍历`List`中的信息包装器以找到匹配项。请记住，编译的模式是一个捕获组，它查看文本的开始，所以我们遍历每个`Pattern`直到找到一个匹配项。如果我们找不到匹配项，我们抛出一个`Exception`。

一旦我们找到匹配项，我们就从`Matcher`中提取匹配的文本，然后使用`Token.Info`调用`getToken()`来获取匹配`Pattern`的`Token`实例。我们将它存储在我们的列表中，然后递归调用`parse()`方法，传递从我们的匹配项之后开始的文本子串。这将从原始文本中移除匹配的文本，然后重复此过程，直到字符串为空。一旦递归结束并且事情展开，我们返回一个表示用户提供的字符串的`Queue`。我们使用`Queue`而不是，比如说，`List`，因为这会使处理更容易。我们现在有了解析器，但我们的工作还只完成了一半。现在我们需要处理这些标记。

在关注点分离的精神下，我们将这些标记的处理——实际的表达式计算——封装在一个单独的类`DateCalculator`中，该类使用我们的解析器。考虑以下代码：

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

每次调用`calculate()`时，我们都会创建一个新的解析器实例。同时，请注意，当我们查看其余的代码时，我们在传递`Queue`。虽然这确实使方法签名更大，但它也使类线程安全，因为类本身不持有任何状态。

在我们的`isEmpty()`检查之后，我们可以看到`Queue` API 是如何派上用场的。通过调用`poll()`，我们得到集合中下一个元素的引用，但——这很重要——**我们保留元素在集合中**。这让我们可以查看它而不改变集合的状态。根据集合中第一个元素的类型，我们将任务委托给适当的方法。

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

要从`Queue`中检索一个元素，我们使用`poll()`方法，并且我们可以安全地将它转换为`DateToken`，因为我们已经在调用方法中检查了这一点。接下来，我们`peek()`查看下一个元素，并通过`validateToken()`方法验证该元素不是 null 并且是所需类型。如果标记有效，我们可以安全地`poll()`并转换。接下来，我们`peek()`第三个标记。根据其类型，我们将任务委托给正确的方法以完成处理。如果我们发现意外的`Token`类型，我们抛出`Exception`。

在查看这些计算方法之前，让我们看看`validateToken()`：

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

这里没有什么特别激动人心的，但细心的读者可能会注意到我们正在返回我们的标记的类名，并且通过这样做，我们将非导出类的名称泄露给了最终用户。这可能不是最佳做法，但我们将把修复这个问题留给读者作为练习。

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

由于我们已经有起始和操作标记，我们将它们传递进去，以及`Queue`，这样我们就可以处理剩余的标记。我们的第一步是确定操作符是加号还是减号，根据需要将正`1`或负`-1`赋值给`negate`。我们这样做是为了可以使用一个单一的方法`LocalDate.plus()`。如果操作符是减号，我们添加一个负数，得到的结果与减去原始数字相同。

最后，我们遍历剩余的标记，在处理之前验证每一个。我们获取`IntegerToken`；获取其值；将其乘以我们的负数修饰符`negate`；然后使用`UnitOfMeasureToken`来告知我们添加的是哪种类型的值，将该值加到`LocalDate`上。

计算日期之间的差异相当直接，正如我们所看到的：

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

我们从两个`DateToken`变量中提取`LocalDate`，然后调用`Period.between()`，它返回一个`Period`，表示两个日期之间的时间差。我们检查哪个日期先到，以便我们返回一个正的`Period`给用户作为便利，因为大多数人通常不会考虑负的时间段。

基于时间的方 法在很大程度上是相同的。最大的区别是时间差方法：

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

这里明显的不同之处在于使用了`Duration.between()`。它看起来与`Period.between()`相同，但`Duration`类提供了一个`Period`类没有的方法：`abs()`。这个方法允许我们返回`Period`的绝对值，因此我们可以以任何顺序将我们的`LocalTime`变量传递给`between()`。

在我们离开之前还有一个最后的注意事项——我们正在将结果包装在`DateCalculatorResult`实例中。由于各种操作返回几种不同且不相关的类型，这使得我们可以从`calculate()`方法返回单一类型。这将由调用代码负责提取适当的价值。我们将在我们的命令行界面中这样做，我们将在下一节中查看。

# 简短的小憩：测试

在我们继续前进之前，我们需要讨论一个我们还没有讨论过的话题，那就是测试。任何在业界工作了一段时间的人可能都听说过术语**测试驱动开发**（或简称**TDD**）。这是一种软件开发方法，它认为首先应该编写的是测试，这个测试会失败（因为没有代码可以运行），然后应该编写代码使测试**通过**，这是一个对 IDE 和其他工具中给出的绿色指示器的引用，表示测试已经通过。这个过程会根据需要重复多次以构建最终系统，总是以小增量进行更改，并且始终从测试开始。关于这个主题已经写出了许多书籍，这个话题既被激烈争论，又往往具有很多细微差别。如果确实实施了这个方法，其具体实施方式几乎总是以不同的风味出现。

显然，在我们这里的工作中，我们没有严格遵循 TDD 原则，但这并不意味着我们没有进行测试。虽然 TDD 的纯粹主义者可能会挑剔，但我的总体方法在测试方面通常比较宽松，直到我的 API 开始稳固一些。这需要多长时间取决于我对所使用技术的熟悉程度。如果我对它们非常熟悉，我可能会快速勾勒出一个接口，然后基于这个接口构建一个测试，以此来测试 API 本身，然后进行迭代。对于新的库，我可能会编写一个非常广泛的测试，以帮助推动对新库的调查，使用测试框架作为在可以实验的运行环境中启动的途径。无论如何，在开发工作的最后，新的系统应该被**全面**测试（**全面**的确切定义是另一个激烈争论的概念），这正是我在这里努力追求的。不过，关于测试和测试驱动开发的全面论述超出了我们这里的范围。

当涉及到 Java 的测试时，你有许多选择。然而，最常见的是 TestNG 和 JUnit，其中 JUnit 可能是最受欢迎的。你应该选择哪一个？这取决于。如果你正在使用现有的代码库，你可能会使用已经存在的任何东西，除非你有很好的理由去做其他的事情。例如，库可能已经过时且不再受支持，它可能明显不足以满足你的需求，或者你被明确指示更新/替换现有的系统。如果这些条件中的任何一个，或者类似的情况是真实的，我们将回到这个问题——*我应该选择哪一个？* 再次，这取决于。JUnit 非常流行和常见，所以使用它可能会降低进入项目的门槛。然而，TestNG 被认为有一个更好、更干净的 API。例如，TestNG 不需要使用静态方法来执行某些测试设置方法。它还旨在成为不仅仅是一个单元测试框架，提供单元、功能、端到端和集成测试的工具。在我们的测试中，我们将使用 TestNG。

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

编写测试非常简单。使用 TestNG Maven 插件的默认设置，类只需要位于`src/test/java`目录中，并以`Test`字符串结尾。每个测试方法都需要用`@Test`注解。

图书馆模块中有很多测试，所以让我们从一些非常基础的测试开始，这些测试用于检查标记符使用的正则表达式，以识别和提取表达式的相关部分。例如，考虑以下代码片段：

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

这是对`DateToken`正则表达式的一个非常基础的测试。测试委托给`testPattern()`方法，传递要测试的正则表达式和一个用于测试的字符串。我们的功能通过以下步骤进行测试：

1.  编译`Pattern`。

1.  创建一个`Matcher`。

1.  调用`matcher.find()`方法。

通过这样，我们测试了被测试系统的逻辑。剩下的是验证它是否按预期工作。我们通过调用`Assert.assertTrue()`来完成这个验证。我们断言`matcher.find()`返回`true`。如果正则表达式正确，我们应该得到一个`true`响应。如果正则表达式不正确，我们将得到一个`false`响应。在后一种情况下，`assertTrue()`将抛出一个`Exception`，测试将失败。

这个测试当然非常基础。它本可以——应该——更加健壮。它应该测试更多的字符串。它应该包括一些已知是错误的字符串，以确保我们的测试中没有得到错误的结果。可能还有无数的其他改进可以做出。然而，这里的目的是展示一个简单的测试，以演示如何设置基于 TestNG 的环境。在继续之前，让我们看看更多的一些例子。

这里有一个用于检查失败的测试（一个**负面测试**）：

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

在这个测试中，我们期望`parse()`的调用会失败，并抛出`DateCalcException`。如果调用**不**失败，我们将有一个调用`Assert.fail()`的调用，这将强制测试以提供的信息失败。如果抛出了`Exception`，它将被静默捕获，并且测试将成功完成。

捕获`Exception`是一种方法，但你也可以告诉 TestNG 预期会抛出`Exception`，就像我们在这里通过`expectedExceptions`属性所做的那样：

```java
    @Test(expectedExceptions = {DateCalcException.class}) 
    public void shouldRejectBadTimes() { 
      parser.parse("22:89"); 
    } 
```

再次强调，我们向解析器传递了一个错误的字符串。然而，这次，我们通过注解告诉 TestNG 预期异常--`@Test(expectedExceptions = {DateCalcException.class})`。

关于测试的一般性和 TestNG 的特定性可以写更多。这两个主题的彻底处理超出了我们的范围，但如果你不熟悉这两个主题中的任何一个，你最好找到许多可用的优秀资源并彻底研究它们。

现在，让我们把注意力转向命令行界面。

# 构建命令行界面

在上一章中，我们使用 Tomitribe 的 Crest 库构建了一个命令行工具，并且效果相当不错，所以我们将回到这个库来构建这个命令行。

要在我们的项目中启用 Crest，我们必须做两件事。首先，我们必须按照以下方式配置我们的 POM 文件：

```java
    <dependency> 
      <groupId>org.tomitribe</groupId> 
      <artifactId>tomitribe-crest</artifactId> 
      <version>0.8</version> 
    </dependency> 
```

我们还必须更新我们的模块定义在`src/main/java/module-info.java`中，如下所示：

```java
    module datecalc.cli { 
      requires datecalc.lib; 
      requires tomitribe.crest; 
      requires tomitribe.crest.api; 

      exports com.steeplesoft.datecalc.cli; 
    } 
```

我们现在可以定义我们的 CLI 类如下：

```java
    public class DateCalc { 
      @Command 
      public void dateCalc(String... args) { 
        final String expression = String.join(" ", args); 
        final DateCalculator dc = new DateCalculator(); 
        final DateCalculatorResult dcr = dc.calculate(expression); 
```

与上一章不同，这个命令行将非常简单，因为我们需要的唯一输入是要评估的表达式。根据前面的方法签名，我们告诉 Crest 将所有命令行参数作为`args`值传递，然后我们通过`String.join()`将它们重新组合成`expression`。接下来，我们创建我们的计算器并计算结果。

现在我们需要调查我们的`DateCalcResult`以确定表达式的性质。以下代码片段作为示例：

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

`LocalDate`和`LocalTime`的响应相当直接--我们只需在它们上调用`toString()`方法，因为默认值对我们来说在这里是完全可以接受的。持续时间周期稍微复杂一些。两者都提供了一些方法来提取详细信息。我们将把这些细节隐藏在单独的方法中：

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

该方法本身相当简单--我们从`Duration`中提取各个部分，然后根据部分是否返回值来构建字符串。

与日期相关的方法`processPeriod()`类似：

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

这些方法中的每一个都将结果作为字符串返回，然后我们将其写入标准输出。就是这样。这不是一个特别复杂的命令行工具，但这个练习的目的主要在于库。

# 摘要

我们的数据计算器现在已经完成了。这个实用工具本身并不太复杂，尽管如此，它确实按照预期工作，这为我们提供了一个实验 Java 8 的日期/时间 API 的平台。除了新的日期/时间 API 之外，我们还对正则表达式进行了初步探索，这是一个非常强大且复杂的工具，用于解析字符串。我们还回顾了上一章中的命令行实用工具库，并尝试了单元测试和测试驱动开发。

在下一章中，我们将变得更加雄心勃勃，进入社交媒体的世界，构建一个应用程序，帮助我们把我们的一些最喜欢的服务聚合到一个单一的应用程序中。
