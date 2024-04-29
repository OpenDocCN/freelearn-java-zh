# 使用新的日期和时间 API

在本章中，我们将介绍以下内容：

+   如何构建不依赖于时区的日期和时间实例

+   如何构建依赖于时区的时间实例

+   如何创建日期间的基于日期的周期

+   如何创建基于时间的时间实例之间的周期

+   如何表示纪元时间

+   如何操作日期和时间实例

+   如何比较日期和时间

+   如何处理不同的日历系统

+   如何使用`DateTimeFormatter`格式化日期

# 介绍

使用`java.util.Date`和`java.util.Calendar`对于 Java 开发人员来说是一种痛苦，直到 Stephen Colebourne ([`www.joda.org/`](http://www.joda.org/))引入了 Joda-Time ([`www.joda.org/joda-time/`](http://www.joda.org/joda-time/))，这是一个用于在 Java 中处理日期和时间的库。Joda-Time 相对于 JDK API 提供了以下优势：

+   更丰富的 API 用于获取日期组件，如月份的日、星期的日、月份和年份，以及时间组件，如小时、分钟和秒。

+   轻松操作和比较日期和时间。

+   可用的既不依赖于时区又依赖于时区的 API。大多数情况下，我们将使用不依赖于时区的 API，这样更容易使用 API。

+   令人惊叹的 API，可以计算日期和时间之间的持续时间。

+   日期格式化和持续时间计算默认遵循 ISO 标准。

+   支持多个日历，如公历、佛历和伊斯兰历。

Joda-Time 启发了 JSR-310 ([`jcp.org/en/jsr/detail?id=310`](https://jcp.org/en/jsr/detail?id=310))，将 API 移植到了`java.time`包下，并作为 Java 8 的一部分发布。由于新的日期/时间 API 基于 ISO 标准，因此可以轻松地在应用程序的不同层之间集成日期/时间库。例如，在 JavaScript 层，我们可以使用 moment.js ([`momentjs.com/docs/`](https://momentjs.com/docs/))处理日期和时间，并使用其默认格式化样式（符合 ISO 标准）将数据发送到服务器。在服务器层，我们可以使用新的日期/时间 API 根据需要获取日期和时间实例。因此，我们可以使用标准日期表示在客户端和服务器之间进行交互。

在本章中，我们将探讨利用新的日期/时间 API 的不同方法。

# 如何处理不依赖于时区的日期和时间实例

在 JSR-310 之前，要为任何时间点或日历中的任何一天创建日期和时间实例并不直观。唯一的方法是使用`java.util.Calendar`对象设置所需的日期和时间，然后调用`getTime()`方法获取`java.util.Date`的实例。这些日期和时间实例也包含时区信息，有时会导致应用程序中的错误。

在新的 API 中，获取日期和时间实例要简单得多，这些日期和时间实例不包含任何与时区相关的信息。在本示例中，我们将向您展示如何使用`java.time.LocalDate`表示仅日期的实例，使用`java.time.LocalTime`表示仅时间的实例，以及使用`java.time.LocalDateTime`表示日期/时间实例。这些日期和时间实例是不依赖于时区的，并表示机器的当前时区中的信息。

# 准备工作

您需要安装至少 JDK 8 才能使用这些更新的库，本章中的示例使用 Java 10 及更高版本支持的语法。如果您愿意，可以直接在 JShell 中运行这些代码片段。您可以访问第十二章，*使用 JShell 进行读取-求值-打印循环(REPL)*，了解更多关于 JShell 的信息。

# 如何做…

1.  使用`now()`方法可以获取包装在`java.time.LocalDate`中的当前日期，如下所示：

```java
var date = LocalDate.now();
```

1.  我们可以使用通用的`get(fieldName)`方法或特定的方法，如`getDayOfMonth()`、`getDayOfYear()`、`getDayOfWeek()`、`getMonth()`和`getYear()`来获取`java.time.LocalDate`实例的各个字段，如下所示：

```java
var dayOfWeek = date.getDayOfWeek();
var dayOfMonth = date.getDayOfMonth();
var month = date.getMonth();
var year = date.getYear();
```

1.  我们可以使用`of()`方法获取日历中任何日期的`java.time.LocalDate`实例，如下所示：

```java
var date1 = LocalDate.of(2018, 4, 12);
var date2 = LocalDate.of(2018, Month.APRIL, 12);
date2 = LocalDate.ofYearDay(2018, 102);
date2 = LocalDate.parse("2018-04-12");
```

1.  有`java.time.LocalTime`类，用于表示任何时间实例，而不考虑日期。可以使用以下方法获取当前时间：

```java
var time = LocalTime.now();
```

1.  `java.time.LocalTime`类还带有`of()`工厂方法，可用于创建表示任何时间的实例。类似地，有方法来获取时间的不同组件，如下所示：

```java
time = LocalTime.of(23, 11, 11, 11);
time = LocalTime.ofSecondOfDay(3600);

var hour = time.getHour();
var minutes = time.getMinute();
var seconds = time.get(ChronoField.SECOND_OF_MINUTE);
```

1.  `java.time.LocalDateTime`用于表示包含时间和日期的实体。它由`java.time.LocalDate`和`java.time.LocalTime`组成，分别表示日期和时间。可以使用`now()`和不同版本的`of()`工厂方法创建其实例，如下所示：

```java
var dateTime1 = LocalDateTime.of(2018, 04, 12, 13, 30, 22);
var dateTime2 = LocalDateTime.of(2018, Month.APRIL, 12, 13, 30, 22);
dateTime2 = LocalDateTime.of(date2, LocalTime.of(13, 30, 22));
```

# 它是如何工作的…

`java.time`包中的以下三个类代表默认时区（系统的时区）中的日期和时间值：

+   `java.time.LocalDate`: 只包含日期信息

+   `java.time.LocalTime`: 只包含时间信息

+   `java.time.LocalDateTime`: 包含日期和时间信息

每个类都由以下字段组成：

+   日期

+   月

+   年

+   小时

+   分钟

+   秒

+   毫秒

所有类都包含`now()`方法，返回当前的日期和时间值。提供了`of()`工厂方法来根据它们的字段（如日、月、年、小时和分钟）构建日期和时间实例。`java.time.LocalDateTime`由`java.time.LocalDate`和`java.time.LocalTime`组成，因此可以从`java.time.LocalDate`和`java.time.LocalTime`构建`java.time.LocalDateTime`。

从这个示例中学到的重要 API 如下：

+   `now()`: 这会给出当前日期和时间

+   `of()`: 这个工厂方法用于构造所需的日期、时间和日期/时间实例

# 还有更多…

在 Java 9 中，有一个新的 API，`datesUntil`，它接受结束日期并返回从当前对象的日期到结束日期（但不包括结束日期）的顺序日期流（换句话说，`java.time.LocalDate`）。使用此 API 将给定月份和年份的所有日期分组到它们各自的星期几，即星期一、星期二、星期三等。

让我们接受月份和年份，并将其分别存储在`month`和`year`变量中。范围的开始将是该月和年的第一天，如下所示：

```java
var startDate = LocalDate.of(year, month, 1);
```

范围的结束日期将是该月的天数，如下所示：

```java
var endDate = startDate.plusDays(startDate.lengthOfMonth());
```

我们正在使用`lengthOfMonth`方法获取该月的天数。然后我们使用`datesUntil`方法获取`java.time.LocalDate`的流，然后执行一些流操作：

+   按星期几对`java.time.LocalDate`实例进行分组。

+   将分组的实例收集到`java.util.ArrayList`中。但在此之前，我们正在应用转换将`java.time.LocalDate`实例转换为一个简单的月份，这给我们提供了一个表示月份的整数列表。

代码中的前两个操作如下所示：

```java
var dayBuckets = startDate.datesUntil(endDate).collect(

Collectors.groupingBy(date -> date.getDayOfWeek(), 
    Collectors.mapping(LocalDate::getDayOfMonth, 
        Collectors.toList())
));
```

此代码可以在下载的代码的`Chapter13/1_2_print_calendar`中找到。

# 如何构造依赖于时区的时间实例

在上一个示例中，*如何构造不依赖于时区的日期和时间实例*，我们构造了不包含任何时区信息的日期和时间对象。它们隐式地表示了系统时区中的值；这些类是`java.time.LocalDate`、`java.time.LocalTime`和`java.time.LocalDateTime`。

通常我们需要根据某个时区表示时间；在这种情况下，我们将使用`java.time.ZonedDateTime`，它包含了带有`java.time.LocalDateTime`的时区信息。时区信息是使用`java.time.ZoneId`或`java.time.ZoneOffset`实例嵌入的。还有两个类，`java.time.OffsetTime`和`java.time.OffsetDateTime`，它们也是`java.time.LocalTime`和`java.time.LocalDateTime`的特定于时区的变体。

在这个示例中，我们将展示如何使用`java.time.ZonedDateTime`、`java.time.ZoneId`、`java.time.ZoneOffset`、`java.time.OffsetTime`和`java.time.OffsetDateTime`。

# 准备工作

我们将使用 Java 10 的语法，使用`var`来声明局部变量和模块。除了 Java 10 及以上版本，没有其他先决条件。

# 操作步骤

1.  我们将使用`now()`工厂方法根据系统的时区获取当前的日期、时间和时区信息，如下所示：

```java
var dateTime = ZonedDateTime.now();
```

1.  我们将使用`java.time.ZoneId`根据任何给定的时区获取当前的日期和时间信息：

```java
var indianTz = ZoneId.of("Asia/Kolkata");
var istDateTime = ZonedDateTime.now(indianTz);
```

1.  `java.time.ZoneOffset`也可以用来提供日期和时间的时区信息，如下所示：

```java
var indianTzOffset = ZoneOffset.ofHoursMinutes(5, 30);
istDateTime = ZonedDateTime.now(indianTzOffset);
```

1.  我们将使用`of()`工厂方法构建`java.time.ZonedDateTime`的一个实例：

```java
ZonedDateTime dateTimeOf = ZonedDateTime.of(2018, 4, 22, 14, 30, 11, 33, indianTz);
```

1.  我们甚至可以从`java.time.ZonedDateTime`中提取`java.time.LocalDateTime`：

```java
var localDateTime = dateTimeOf.toLocalDateTime();
```

# 工作原理

首先，让我们看看如何捕获时区信息。它是根据**格林威治标准时间（GMT）**的小时和分钟数捕获的，也被称为协调世界时（UTC）。例如，印度标准时间（IST），也称为 Asia/Kolkata，比 GMT 提前 5 小时 30 分钟。

Java 提供了`java.time.ZoneId`和`java.time.ZoneOffset`来表示时区信息。`java.time.ZoneId`根据时区名称捕获时区信息，例如 Asia/Kolkata，US/Pacific 和 US/Mountain。大约有 599 个时区 ID。这是使用以下代码行计算的：

```java
jshell> ZoneId.getAvailableZoneIds().stream().count()
$16 ==> 599
```

我们将打印 10 个时区 ID：

```java
jshell> ZoneId.getAvailableZoneIds().stream().limit(10).forEach(System.out::println)
Asia/Aden
America/Cuiaba
Etc/GMT+9
Etc/GMT+8
Africa/Nairobi
America/Marigot
Asia/Aqtau
Pacific/Kwajalein
America/El_Salvador
Asia/Pontianak
```

时区名称，例如 Asia/Kolkata，Africa/Nairobi 和 America/Cuiaba，基于国际分配的数字管理局（IANA）发布的时区数据库。IANA 提供的时区区域名称是 Java 的默认值。

有时时区区域名称也表示为 GMT+02:30 或简单地+02:30，这表示当前时区与 GMT 时区的偏移（提前或落后）。

这个`java.time.ZoneId`捕获了`java.time.zone.ZoneRules`，其中包含了获取时区偏移转换和其他信息的规则，比如夏令时。让我们调查一下 US/Pacific 的时区规则：

```java
jshell> ZoneId.of("US/Pacific").getRules().getDaylightSavings(Instant.now())
$31 ==> PT1H

jshell> ZoneId.of("US/Pacific").getRules().getOffset(LocalDateTime.now())
$32 ==> -07:00

jshell> ZoneId.of("US/Pacific").getRules().getStandardOffset(Instant.now())
$33 ==> -08:00
```

`getDaylightSavings()`方法返回一个`java.time.Duration`对象，表示以小时、分钟和秒为单位的一些持续时间。默认的`toString()`实现返回使用 ISO 8601 基于秒的表示，其中 1 小时 20 分钟 20 秒的持续时间表示为`PT1H20M20S`。关于这一点将在本章的*如何在时间实例之间创建基于时间的期间*中进行更多介绍。

我们不会详细介绍它是如何计算的。对于那些想了解更多关于`java.time.zone.ZoneRules`和`java.time.ZoneId`的人，请访问[`docs.oracle.com/javase/10/docs/api/java/time/zone/ZoneRules.html`](https://docs.oracle.com/javase/10/docs/api/java/time/zone/ZoneRules.html)和[`docs.oracle.com/javase/10/docs/api/java/time/ZoneId.html`](https://docs.oracle.com/javase/10/docs/api/java/time/ZoneId.html)的文档。

`java.time.ZoneOffset`类以时区领先或落后 GMT 的小时和分钟数来捕获时区信息。让我们使用`of*()`工厂方法创建`java.time.ZoneOffset`类的一个实例：

```java
jshell> ZoneOffset.ofHoursMinutes(5,30)
$27 ==> +05:30
```

`java.time.ZoneOffset`类继承自`java.time.ZoneId`并添加了一些新方法。重要的是要记住根据应用程序中要使用的所需时区构造`java.time.ZoneOffset`和`java.time.ZoneId`的正确实例。

现在我们对时区表示有了了解，`java.time.ZonedDateTime`实际上就是`java.time.LocalDateTime`加上`java.time.ZoneId`或`java.time.ZoneOffset`。还有两个其他类，`java.time.OffsetTime`和`java.time.OffsetDateTime`，分别包装了`java.time.LocalTime`和`java.time.LocalDateTime`，以及`java.time.ZoneOffset`。

让我们看看一些构造`java.time.ZonedDateTime`实例的方法。

第一种方法是使用`now()`：

```java
Signatures:
ZonedDateTime ZonedDateTime.now()
ZonedDateTime ZonedDateTime.now(ZoneId zone)
ZonedDateTime ZonedDateTime.now(Clock clock)

jshell> ZonedDateTime.now()
jshell> ZonedDateTime.now(ZoneId.of("Asia/Kolkata"))
$36 ==> 2018-05-04T21:58:24.453113900+05:30[Asia/Kolkata]
jshell> ZonedDateTime.now(Clock.fixed(Instant.ofEpochSecond(1525452037), ZoneId.of("Asia/Kolkata")))
$54 ==> 2018-05-04T22:10:37+05:30[Asia/Kolkata]
```

`now()`的第一种用法使用系统时钟以及系统时区来打印当前日期和时间。`now()`的第二种用法使用系统时钟，但时区由`java.time.ZoneId`提供，这种情况下是 Asia/Kolkata。`now()`的第三种用法使用提供的固定时钟和`java.time.ZoneId`提供的时区。

使用`java.time.Clock`类及其静态方法`fixed()`创建固定时钟，该方法接受`java.time.Instant`和`java.time.ZoneId`的实例。`java.time.Instant`的实例是在纪元后的一些静态秒数后构建的。`java.time.Clock`用于表示新的日期/时间 API 可以用来确定当前时间的时钟。时钟可以是固定的，就像我们之前看到的那样，然后我们可以创建一个比 Asia/Kolkata 时区的当前系统时间提前一小时的时钟，如下所示：

```java
var hourAheadClock = Clock.offset(Clock.system(ZoneId.of("Asia/Kolkata")), Duration.ofHours(1));
```

我们可以使用这个新的时钟来构建`java.time.LocalDateTime`和`java.time.ZonedDateTime`的实例，如下所示：

```java
jshell> LocalDateTime.now(hourAheadClock)
$64 ==> 2018-05-04T23:29:58.759973700
jshell> ZonedDateTime.now(hourAheadClock)
$65 ==> 2018-05-04T23:30:11.421913800+05:30[Asia/Kolkata]
```

日期和时间值都基于相同的时区，即 Asia/Kolkata，但正如我们已经了解的那样，`java.time.LocalDateTime`没有任何时区信息，它基于系统的时区或在这种情况下提供的`java.time.Clock`的值。另一方面，`java.time.ZonedDateTime`包含并显示时区信息为[Asia/Kolkata]。

另一种创建`java.time.ZonedDateTime`实例的方法是使用其`of()`工厂方法：

```java
Signatures:
ZonedDateTime ZonedDateTime.of(LocalDate date, LocalTime time, ZoneId zone)
ZonedDateTime ZonedDateTime.of(LocalDateTime localDateTime, ZoneId zone)
ZonedDateTime ZonedDateTime.of(int year, int month, int dayOfMonth, int hour, int minute, int second, int nanoOfSecond, ZoneId zone)

jshell> ZonedDateTime.of(LocalDateTime.of(2018, 1, 1, 13, 44, 44), ZoneId.of("Asia/Kolkata"))
$70 ==> 2018-01-01T13:44:44+05:30[Asia/Kolkata]

jshell> ZonedDateTime.of(LocalDate.of(2018,1,1), LocalTime.of(13, 44, 44), ZoneId.of("Asia/Kolkata"))
$71 ==> 2018-01-01T13:44:44+05:30[Asia/Kolkata]

jshell> ZonedDateTime.of(LocalDate.of(2018,1,1), LocalTime.of(13, 44, 44), ZoneId.of("Asia/Kolkata"))
$72 ==> 2018-01-01T13:44:44+05:30[Asia/Kolkata]

jshell> ZonedDateTime.of(2018, 1, 1, 13, 44, 44, 0, ZoneId.of("Asia/Kolkata"))
$73 ==> 2018-01-01T13:44:44+05:30[Asia/Kolkata] 
```

# 还有更多...

我们提到了`java.time.OffsetTime`和`java.time.OffsetDateTime`类。两者都包含特定于时区的时间值。在我们结束这个教程之前，让我们玩一下这些类。

+   使用`of()`工厂方法：

```java
jshell> OffsetTime.of(LocalTime.of(14,12,34), ZoneOffset.ofHoursMinutes(5, 30))
$74 ==> 14:12:34+05:30

jshell> OffsetTime.of(14, 34, 12, 11, ZoneOffset.ofHoursMinutes(5, 30))
$75 ==> 14:34:12.000000011+05:30
```

+   使用`now()`工厂方法：

```java
Signatures:
OffsetTime OffsetTime.now()
OffsetTime OffsetTime.now(ZoneId zone)
OffsetTime OffsetTime.now(Clock clock)

jshell> OffsetTime.now()
$76 ==> 21:49:16.895192800+03:00

jshell> OffsetTime.now(ZoneId.of("Asia/Kolkata"))

jshell> OffsetTime.now(ZoneId.of("Asia/Kolkata"))
$77 ==> 00:21:04.685836900+05:30

jshell> OffsetTime.now(Clock.offset(Clock.systemUTC(), Duration.ofMinutes(330)))
$78 ==> 00:22:00.395463800Z
```

值得注意的是我们如何构建了一个`java.time.Clock`实例，它比 UTC 时钟提前了 330 分钟（5 小时 30 分钟）。另一个类`java.time.OffsetDateTime`与`java.time.OffsetTime`相同，只是它使用`java.time.LocalDateTime`。因此，您将向其工厂方法`of()`传递日期信息，即年、月和日，以及时间信息。

# 如何在日期实例之间创建基于日期的期间

在过去，我们曾试图测量两个日期实例之间的期间，但由于 Java 8 之前缺乏 API 以及缺乏捕获此信息的适当支持，我们采用了不同的方法。我们记得使用基于 SQL 的方法来处理这样的信息。但从 Java 8 开始，我们有了一个新的类`java.time.Period`，它可以用来捕获两个日期实例之间的期间，以年、月和日的数量来表示。

此外，该类支持解析基于 ISO 8601 标准的字符串来表示期间。该标准规定任何期间都可以用`PnYnMnD`的形式表示，其中**P**是表示期间的固定字符，**nY**表示年数，**nM**表示月数，**nD**表示天数。例如，2 年 4 个月 10 天的期间表示为`P2Y4M10D`。

# 准备工作

您至少需要 JDK8 来使用`java.time.Period`，需要 JDK 9 才能使用 JShell，并且至少需要 JDK 10 才能使用本示例中使用的示例。

# 如何做…

1.  让我们使用其`of()`工厂方法创建一个`java.time.Period`的实例，其签名为`Period.of(int years, int months, int days)`：

```java
jshell> Period.of(2,4,30)
$2 ==> P2Y4M30D
```

1.  还有特定变体的`of*()`方法，即`ofDays()`，`ofMonths()`和`ofYears()`，也可以使用：

```java
jshell> Period.ofDays(10)
$3 ==> P10D
jshell> Period.ofMonths(4)
$4 ==> P4M
jshell> Period.ofWeeks(3)
$5 ==> P21D
jshell> Period.ofYears(3)
$6 ==> P3Y
```

请注意，`ofWeeks()`方法是一个辅助方法，用于根据接受的周数构建`java.time.Period`。

1.  期间也可以使用期间字符串构造，该字符串通常采用`P<x>Y<y>M<z>D`的形式，其中`x`，`y`和`z`分别表示年、月和日的数量：

```java
jshell> Period.parse("P2Y4M23D").getDays()
$8 ==> 23
```

1.  我们还可以计算`java.time.ChronoLocalDate`的两个实例之间的期间（其实现之一是`java.time.LocalDate`）：

```java
jshell> Period.between(LocalDate.now(), LocalDate.of(2018, 8, 23))
$9 ==> P2M2D
jshell> Period.between(LocalDate.now(), LocalDate.of(2018, 2, 23))
$10 ==> P-3M-26D
```

这些是创建`java.time.Period`实例的最有用的方法。开始日期是包含的，结束日期是不包含的。

# 它是如何工作的…

我们利用`java.time.Period`中的工厂方法来创建其实例。`java.time.Period`有三个字段分别用于保存年、月和日的值，如下所示：

```java
/**
* The number of years.
*/
private final int years;
/**
* The number of months.
*/
private final int months;
/**
* The number of days.
*/
private final int days;
```

还有一组有趣的方法，即`withDays()`，`withMonths()`和`withYears()`。如果它正在尝试更新的字段具有相同的值，则这些方法返回相同的实例；否则，它返回一个具有更新值的新实例，如下所示：

```java
jshell> Period period1 = Period.ofWeeks(2)
period1 ==> P14D

jshell> Period period2 = period1.withDays(15)
period2 ==> P15D

jshell> period1 == period2
$19 ==> false

jshell> Period period3 = period1.withDays(14)
period3 ==> P14D

jshell> period1 == period3
$21 ==> true
```

# 还有更多…

我们甚至可以使用`java.time.ChronoLocalDate`中的`until()`方法计算两个日期实例之间的`java.time.Period`：

```java
jshell> LocalDate.now().until(LocalDate.of(2018, 2, 23))
$11 ==> P-3M-26D

jshell> LocalDate.now().until(LocalDate.of(2018, 8, 23))
$12 ==> P2M2D
```

给定`java.time.Period`的一个实例，我们可以使用它来操作给定的日期实例。有两种可能的方法：

+   使用期间对象的`addTo`或`subtractFrom`方法

+   使用日期对象的`plus`或`minus`方法

这两种方法都显示在以下代码片段中：

```java
jshell> Period period1 = Period.ofWeeks(2)
period1 ==> P14D

jshell> LocalDate date = LocalDate.now()
date ==> 2018-06-21

jshell> period1.addTo(date)
$24 ==> 2018-07-05

jshell> date.plus(period1)
$25 ==> 2018-07-05
```

同样，您可以尝试`subtractFrom`和`minus`方法。还有另一组用于操作`java.time.Period`实例的方法，即以下方法：

+   `minus`，`minusDays`，`minusMonths`和`minusYears`：从期间中减去给定的值。

+   `plus`，`plusDays`，`plusMonths`和`plusYears`：将给定的值添加到期间。

+   `negated`：返回每个值都取反的新期间。

+   `normalized`：通过规范化其更高阶字段（如月和日）返回一个新的期间。例如，15 个月被规范化为 1 年和 3 个月。

我们将展示这些方法的操作，首先是`minus`方法：

```java
jshell> period1.minus(Period.of(1,3,4))
$28 ==> P2Y12M25D

jshell> period1.minusDays(4)
$29 ==> P3Y15M25D

jshell> period1.minusMonths(3)
$30 ==> P3Y12M29D

jshell> period1.minusYears(1)
$31 ==> P2Y15M29D
```

然后，我们将看到`plus`方法：

```java
jshell> Period period1 = Period.of(3, 15, 29)
period1 ==> P3Y15M29D

jshell> period1.plus(Period.of(1, 3, 4))
$33 ==> P4Y18M33D

jshell> period1.plusDays(4)
$34 ==> P3Y15M33D

jshell> period1.plusMonths(3)
$35 ==> P3Y18M29D

jshell> period1.plusYears(1)
$36 ==> P4Y15M29D
```

最后，这里是`negated()`和`normalized()`方法：

```java
jshell> Period period1 = Period.of(3, 15, 29)
period1 ==> P3Y15M29D

jshell> period1.negated()
$38 ==> P-3Y-15M-29D

jshell> period1
period1 ==> P3Y15M29D

jshell> period1.normalized()
$40 ==> P4Y3M29D

jshell> period1
period1 ==> P3Y15M29D
```

请注意，在前面的两种情况下，它并没有改变现有的期间，而是返回一个新的实例。

# 如何创建基于时间的期间实例

在我们之前的示例中，我们创建了一个基于日期的期间，由`java.time.Period`表示。在这个示例中，我们将看看如何使用`java.time.Duration`类来以秒和纳秒的方式创建时间实例之间的时间差异。

我们将看看创建`java.time.Duration`实例的不同方法，操作持续时间实例，并以小时和分钟等不同单位获取持续时间。ISO 8601 标准指定了表示持续时间的可能模式之一为`PnYnMnDTnHnMnS`，其中以下内容适用：

+   `Y`，`M`和`D`代表日期组件字段，即年、月和日

+   `T`用于将日期与时间信息分隔开

+   `H`，`M`和`S`代表时间组件字段，即小时、分钟和秒

`java.time.Duration`的字符串表示实现基于 ISO 8601。在*它是如何工作*部分中有更多内容。

# 准备好了

您至少需要 JDK 8 才能使用`java.time.Duration`，并且需要 JDK 9 才能使用 JShell。

# 如何做...

1.  可以使用`of*()`工厂方法创建`java.time.Duration`实例。我们将展示如何使用其中的一些方法，如下所示：

```java
jshell> Duration.of(56, ChronoUnit.MINUTES)
$66 ==> PT56M
jshell> Duration.of(56, ChronoUnit.DAYS)
$67 ==> PT1344H
jshell> Duration.ofSeconds(87)
$68 ==> PT1M27S
jshell> Duration.ofHours(7)
$69 ==> PT7H
```

1.  它们也可以通过解析持续时间字符串来创建，如下所示：

```java
jshell> Duration.parse("P12D")
$70 ==> PT288H
jshell> Duration.parse("P12DT7H5M8.009S")
$71 ==> PT295H5M8.009S
jshell> Duration.parse("PT7H5M8.009S")
$72 ==> PT7H5M8.009S
```

1.  它们可以通过查找两个支持时间信息的`java.time.Temporal`实例之间的时间跨度来构建，这些实例支持时间信息（即`java.time.LocalDateTime`等的实例），如下所示：

```java
jshell> LocalDateTime time1 = LocalDateTime.now()
time1 ==> 2018-06-23T10:51:21.038073800
jshell> LocalDateTime time2 = LocalDateTime.of(2018, 6, 22, 11, 00)
time2 ==> 2018-06-22T11:00
jshell> Duration.between(time1, time2)
$77 ==> PT-23H-51M-21.0380738S
jshell> ZonedDateTime time1 = ZonedDateTime.now()
time1 ==> 2018-06-23T10:56:57.965606200+03:00[Asia/Riyadh]
jshell> ZonedDateTime time2 = ZonedDateTime.of(LocalDateTime.now(), ZoneOffset.ofHoursMinutes(5, 30))
time2 ==> 2018-06-23T10:56:59.878712600+05:30
jshell> Duration.between(time1, time2)
$82 ==> PT-2H-29M-58.0868936S
```

# 它是如何工作的...

`java.time.Duration`所需的数据存储在两个字段中，分别表示秒和纳秒。提供了一些便利方法，以分钟、小时和天为单位获取持续时间，即`toMinutes()`、`toHours()`和`toDays()`。

让我们讨论字符串表示实现。`java.time.Duration`支持解析 ISO 字符串表示，其中日期部分仅包含天组件，时间部分包含小时、分钟、秒和纳秒。例如，`P2DT3M`是可接受的，而解析`P3M2DT3M`将导致`java.time.format.DateTimeParseException`，因为字符串包含日期部分的月份组件。

`java.time.Duration`的`toString()`方法始终返回`PTxHyMz.nS`形式的字符串，其中`x`表示小时数，`y`表示分钟数，`z.n`表示秒数到纳秒精度。让我们看一些例子：

```java
jshell> Duration.parse("P2DT3M")
$2 ==> PT48H3M

jshell> Duration.parse("P3M2DT3M")
| Exception java.time.format.DateTimeParseException: Text cannot be parsed to a Duration
| at Duration.parse (Duration.java:417)
| at (#3:1)

jshell> Duration.ofHours(4)
$4 ==> PT4H

jshell> Duration.parse("PT3H4M5.6S")
$5 ==> PT3H4M5.6S

jshell> Duration d = Duration.parse("PT3H4M5.6S")
d ==> PT3H4M5.6S

jshell> d.toDays()
$7 ==> 0

jshell> d.toHours()
$9 ==> 3
```

# 还有更多...

让我们来看一下提供的操作方法，这些方法允许从特定的时间单位（如天、小时、分钟、秒或纳秒）中添加/减去一个值。每个方法都是不可变的，因此每次都会返回一个新实例，如下所示：

```java
jshell> Duration d = Duration.parse("PT1H5M4S")
d ==> PT1H5M4S

jshell> d.plusDays(3)
$14 ==> PT73H5M4S

jshell> d
d ==> PT1H5M4S

jshell> d.plusDays(3)
$16 ==> PT73H5M4S

jshell> d.plusHours(3)
$17 ==> PT4H5M4S

jshell> d.plusMillis(4)
$18 ==> PT1H5M4.004S

jshell> d.plusMinutes(40)
$19 ==> PT1H45M4S
```

类似地，您可以尝试`minus*()`方法，进行减法。然后有一些方法可以操作`java.time.LocalDateTime`、`java.time.ZonedDateTime`等的实例。这些方法将持续时间添加/减去日期/时间信息。让我们看一些例子：

```java
jshell> Duration d = Duration.parse("PT1H5M4S")
d ==> PT1H5M4S

jshell> d.addTo(LocalDateTime.now())
$21 ==> 2018-06-25T21:15:53.725373600

jshell> d.addTo(ZonedDateTime.now())
$22 ==> 2018-06-25T21:16:03.396595600+03:00[Asia/Riyadh]

jshell> d.addTo(LocalDate.now())
| Exception java.time.temporal.UnsupportedTemporalTypeException: Unsupported unit: Seconds
| at LocalDate.plus (LocalDate.java:1272)
| at LocalDate.plus (LocalDate.java:139)
| at Duration.addTo (Duration.java:1102)
| at (#23:1)
```

您可以观察到在前面的示例中，当我们尝试将持续时间添加到仅包含日期信息的实体时，我们得到了一个异常。

# 如何表示纪元时间

在本教程中，我们将学习如何使用`java.time.Instant`来表示一个时间点，并将该时间点转换为纪元秒/毫秒。Java 纪元用于指代时间瞬间 1970-01-01 00:00:00Z，`java.time.Instant`存储了从 Java 纪元开始的秒数。正值表示时间超过了纪元，负值表示时间落后于纪元。它使用 UTC 中的系统时钟来计算当前时间瞬间值。

# 准备工作

您需要安装支持新日期/时间 API 和 JShell 的 JDK，才能尝试提供的解决方案。

# 如何做...

1.  我们将创建一个`java.time.Instant`实例，并打印出纪元秒，这将给出 Java 纪元后的 UTC 时间：

```java
jshell> Instant.now()
$40 ==> 2018-07-06T07:56:40.651529300Z

jshell> Instant.now().getEpochSecond()
$41 ==> 1530863807
```

1.  我们还可以打印出纪元毫秒，这显示了纪元后的毫秒数。这比仅仅秒更精确：

```java
jshell> Instant.now().toEpochMilli()
$42 ==> 1530863845158
```

# 它是如何工作的...

`java.time.Instant`类将时间信息存储在其两个字段中：

+   秒，类型为`long`：这存储了从 1970-01-01T00:00:00Z 纪元开始的秒数。

+   纳秒，类型为`int`：这存储了纳秒数

当您调用`now()`方法时，`java.time.Instant`使用 UTC 中的系统时钟来表示该时间瞬间。然后我们可以使用`atZone()`或`atOffset()`将其转换为所需的时区，我们将在下一节中看到。

如果您只想表示 UTC 中的操作时间线，那么存储不同事件的时间戳将基于 UTC，并且您可以在需要时将其转换为所需的时区。

# 还有更多...

我们可以通过添加/减去纳秒、毫秒和秒来操纵`java.time.Instant`，如下所示：

```java
jshell> Instant.now().plusMillis(1000)
$43 ==> 2018-07-06T07:57:57.092259400Z

jshell> Instant.now().plusNanos(1991999)
$44 ==> 2018-07-06T07:58:06.097966099Z

jshell> Instant.now().plusSeconds(180)
$45 ==> 2018-07-06T08:01:15.824141500Z
```

同样，您可以尝试`minus*()`方法。我们还可以使用`java.time.Instant`方法获取依赖于时区的日期时间，如`atOffset()`和`atZone()`所示：

```java
jshell> Instant.now().atZone(ZoneId.of("Asia/Kolkata"))
$36 ==> 2018-07-06T13:15:13.820694500+05:30[Asia/Kolkata]

jshell> Instant.now().atOffset(ZoneOffset.ofHoursMinutes(2,30))
$37 ==> 2018-07-06T10:15:19.712039+02:30
```

# 如何操纵日期和时间实例

日期和时间类`java.time.LocalDate`、`java.time.LocalTime`、`java.time.LocalDateTime`和`java.time.ZonedDateTime`提供了从它们的组件中添加和减去值的方法，即天、小时、分钟、秒、周、月、年等。

在这个示例中，我们将看一些可以用来通过添加和减去不同的值来操纵日期和时间实例的方法。

# 准备就绪

您将需要安装支持新的日期/时间 API 和 JShell 控制台的 JDK。

# 如何做到这一点...

1.  让我们操纵`java.time.LocalDate`：

```java
jshell> LocalDate d = LocalDate.now()
d ==> 2018-07-27

jshell> d.plusDays(3)
$5 ==> 2018-07-30

jshell> d.minusYears(4)
$6 ==> 2014-07-27
```

1.  让我们操纵日期和时间实例，`java.time.LocalDateTime`：

```java
jshell> LocalDateTime dt = LocalDateTime.now()
dt ==> 2018-07-27T15:27:40.733389700

jshell> dt.plusMinutes(45)
$8 ==> 2018-07-27T16:12:40.733389700

jshell> dt.minusHours(4)
$9 ==> 2018-07-27T11:27:40.733389700
```

1.  让我们操纵依赖于时区的日期和时间，`java.time.ZonedDateTime`：

```java
jshell> ZonedDateTime zdt = ZonedDateTime.now()
zdt ==> 2018-07-27T15:28:28.309915200+03:00[Asia/Riyadh]

jshell> zdt.plusDays(4)
$11 ==> 2018-07-31T15:28:28.309915200+03:00[Asia/Riyadh]

jshell> zdt.minusHours(3)
$12 ==> 2018-07-27T12:28:28.309915200+03:00[Asia/Riyadh]
```

# 还有更多...

我们刚刚看了一些由`plus*()`和`minus*()`表示的添加和减去 API。还提供了不同的方法来操纵日期和时间的不同组件，如年、日、月、小时、分钟、秒和纳秒。您可以尝试这些 API 作为练习。

# 如何比较日期和时间

通常，我们希望将日期和时间实例与其他实例进行比较，以检查它们是在之前、之后还是与其他实例相同。为了实现这一点，JDK 在`java.time.LocalDate`、`java.time.LocalDateTime`和`java.time.ZonedDateTime`类中提供了`isBefore()`、`isAfter()`和`isEqual()`方法。在这个示例中，我们将看看如何使用这些方法来比较日期和时间实例。

# 准备就绪

您将需要安装具有新的日期/时间 API 并支持 JShell 的 JDK。

# 如何做到这一点...

1.  让我们尝试比较两个`java.time.LocalDate`实例：

```java
jshell> LocalDate d = LocalDate.now()
d ==> 2018-07-28

jshell> LocalDate d2 = LocalDate.of(2018, 7, 27)
d2 ==> 2018-07-27

jshell> d.isBefore(d2)
$4 ==> false

jshell> d.isAfter(d2)
$5 ==> true

jshell> LocalDate d3 = LocalDate.of(2018, 7, 28)
d3 ==> 2018-07-28

jshell> d.isEqual(d3)
$7 ==> true

jshell> d.isEqual(d2)
$8 ==> false
```

1.  我们还可以比较依赖于时区的日期和时间实例：

```java
jshell> ZonedDateTime zdt1 = ZonedDateTime.now();
zdt1 ==> 2018-07-28T14:49:34.778006400+03:00[Asia/Riyadh]

jshell> ZonedDateTime zdt2 = zdt1.plusHours(4)
zdt2 ==> 2018-07-28T18:49:34.778006400+03:00[Asia/Riyadh]

jshell> zdt1.isBefore(zdt2)
$11 ==> true

jshell> zdt1.isAfter(zdt2)
$12 ==> false
jshell> zdt1.isEqual(zdt2)
$13 ==> false
```

# 还有更多...

比较可以在`java.time.LocalTime`和`java.time.LocalDateTime`上进行。这留给读者去探索。

# 如何使用不同的日历系统

到目前为止，在我们的示例中，我们使用了 ISO 日历系统，这是世界上遵循的事实标准日历系统。世界上还有其他地区遵循的日历系统，如伊斯兰历、日本历和泰国历。JDK 也为这些日历系统提供了支持。

在这个示例中，我们将看看如何使用两个日历系统：日本和伊斯兰历。

# 准备就绪

您应该安装支持新的日期/时间 API 和 JShell 工具的 JDK。

# 如何做到这一点...

1.  让我们打印 JDK 支持的不同日历系统中的当前日期：

```java
jshell> Chronology.getAvailableChronologies().forEach(chrono -> 
System.out.println(chrono.dateNow()))
2018-07-30
Minguo ROC 107-07-30
Japanese Heisei 30-07-30
ThaiBuddhist BE 2561-07-30
Hijrah-umalqura AH 1439-11-17
```

1.  让我们玩弄一下用日本日历系统表示的日期：

```java
jshell> JapaneseDate jd = JapaneseDate.now()
jd ==> Japanese Heisei 30-07-30

jshell> jd.getChronology()
$7 ==> Japanese

jshell> jd.getEra()
$8 ==> Heisei

jshell> jd.lengthOfYear()
$9 ==> 365

jshell> jd.lengthOfMonth()
$10 ==> 31
```

1.  日本日历中支持的不同纪元可以使用`java.time.chrono.JapeneseEra`进行枚举：

```java
jshell> JapaneseEra.values()
$42 ==> JapaneseEra[5] { Meiji, Taisho, Showa, Heisei, NewEra }
```

1.  让我们在伊斯兰历中创建一个日期：

```java
jshell> HijrahDate hd = HijrahDate.of(1438, 12, 1)
hd ==> Hijrah-umalqura AH 1438-12-01
```

1.  我们甚至可以将 ISO 日期/时间转换为伊斯兰历的日期/时间，如下所示：

```java
jshell> HijrahChronology.INSTANCE.localDateTime(LocalDateTime.now())
$23 ==> Hijrah-umalqura AH 1439-11-17T19:56:52.056465900

jshell> HijrahChronology.INSTANCE.localDateTime(LocalDateTime.now()).toLocalDate()
$24 ==> Hijrah-umalqura AH 1439-11-17

jshell> HijrahChronology.INSTANCE.localDateTime(LocalDateTime.now()).toLocalTime()
$25 ==> 19:57:07.705740500
```

# 它是如何工作的...

日历系统由`java.time.chrono.Chronology`及其实现表示，其中一些是`java.time.chrono.IsoChronology`、`java.time.chrono.HijrahChronology`和`java.time.chrono.JapaneseChronology`。`java.time.chrono.IsoChronology`是世界上使用的基于 ISO 的事实标准日历系统。每个日历系统中的日期由`java.time.chrono.ChronoLocalDate`及其实现表示，其中一些是`java.time.chrono.HijrahDate`、`java.time.chrono.JapaneseDate`和著名的`java.time.LocalDate`。

要能够在 JShell 中使用这些 API，您需要导入相关的包，如下所示：

```java
jshell> import java.time.*

jshell> import java.time.chrono.*
```

这适用于所有使用 JShell 的示例。

我们可以直接使用`java.time.chrono.ChronoLocalDate`的实现，例如`java.time.chrono.JapaneseDate`，或者使用`java.time.chrono.Chronology`的实现来获取相关的日期表示，如下所示：

```java
jshell> JapaneseDate jd = JapaneseDate.of(JapaneseEra.SHOWA, 26, 12, 25)
jd ==> Japanese Showa 26-12-25

jshell> JapaneseDate jd = JapaneseDate.now()
jd ==> Japanese Heisei 30-07-30

jshell> JapaneseDate jd = JapaneseChronology.INSTANCE.dateNow()
jd ==> Japanese Heisei 30-07-30

jshell> JapaneseDate jd = JapaneseChronology.INSTANCE.date(LocalDateTime.now())
jd ==> Japanese Heisei 30-07-30

jshell> ThaiBuddhistChronology.INSTANCE.date(LocalDate.now())
$41 ==> ThaiBuddhist BE 2561-07-30
```

从前面的代码片段中，我们可以看到可以使用其日历系统的`date(TemporalAccessor temporal)`方法将 ISO 系统日期转换为所需日历系统中的日期。

# 还有更多…

您可以尝试使用 JDK 支持的其他日历系统，即泰国、佛教和民国（中国）日历系统。还值得探索如何通过编写`java.time.chrono.Chronology`、`java.time.chrono.ChronoLocalDate`和`java.time.chrono.Era`的实现来创建我们自定义的日历系统。

# 如何使用 DateTimeFormatter 格式化日期

在使用`java.util.Date`时，我们使用`java.text.SimpleDateFormat`将日期格式化为不同的文本表示形式，反之亦然。格式化日期意味着，以不同格式表示给定日期或时间对象，例如以下格式：

+   2018 年 6 月 23 日

+   2018 年 8 月 23 日

+   2018-08-23

+   2018 年 6 月 23 日上午 11:03:33

这些格式由格式字符串控制，例如以下格式：

+   `dd MMM yyyy`

+   `dd-MM-yyyy`

+   `yyyy-MM-DD`

+   `dd MMM yyyy hh:mm:ss`

在这个示例中，我们将使用`java.time.format.DateTimeFormatter`来格式化新日期和时间 API 中的日期和时间实例，并查看最常用的模式字母。

# 准备工作

您将需要一个具有新的日期/时间 API 和`jshell`工具的 JDK。

# 如何做…

1.  让我们使用内置格式来格式化日期和时间：

```java
jshell> LocalDate ld = LocalDate.now()
ld ==> 2018-08-01

jshell> ld.format(DateTimeFormatter.ISO_DATE)
$47 ==> "2018-08-01"

jshell> LocalDateTime.now().format(DateTimeFormatter.ISO_DATE_TIME)
$49 ==> "2018-08-01T17:24:49.1985601"
```

1.  让我们创建一个自定义的日期/时间格式：

```java
jshell> DateTimeFormatter dtf = DateTimeFormatter.ofPattern("dd MMM yyyy hh:mm:ss a")
dtf ==> Value(DayOfMonth,2)' 'Text(MonthOfYear,SHORT)' 'V ... 2)' 'Text(AmPmOfDay,SHORT)
```

1.  让我们使用自定义的`java.time.format.DateTimeFormatter`来格式化当前的日期/时间：

```java
jshell> LocalDateTime ldt = LocalDateTime.now()
ldt ==> 2018-08-01T17:36:22.442159

jshell> ldt.format(dtf)
$56 ==> "01 Aug 2018 05:36:22 PM"
```

# 它是如何工作的…

让我们了解最常用的格式字母：

| **符号** | **意义** | **示例** |
| --- | --- | --- |
| `d` | 一个月中的日期 | 1,2,3,5 |
| `M`, `MMM`, `MMMM` | 一年中的月份 | `M`: 1,2,3,`MMM`: 六月，七月，八月`MMMM`: 七月，八月 |
| `y`, `yy` | 年 | `y`, `yyyy`: 2017, 2018`yy`: 18, 19 |
| `h` | 一天中的小时（1-12） | 1, 2, 3 |
| `k` | 一天中的小时（0-23） | 0, 1, 2, 3 |
| `m` | 分钟 | 1, 2, 3 |
| `s` | 秒 | 1, 2, 3 |
| `a` | 一天中的上午/下午 | 上午，下午 |
| `VV` | 时区 ID | 亚洲/加尔各答 |
| `ZZ` | 时区名称 | IST, PST, AST |
| `O` | 时区偏移 | GMT+5:30, GMT+3 |

基于前面的格式字母，让我们格式化`java.time.ZonedDateTime`：

```java
jshell> DateTimeFormatter dtf = DateTimeFormatter.ofPattern("dd MMMM yy h:mm:ss a VV")
dtf ==> Value(DayOfMonth,2)' 'Text(MonthOfYear)' 'Reduced ... mPmOfDay,SHORT)' 'ZoneId()

jshell> ZonedDateTime.now().format(dtf)
$67 ==> "01 August 18 6:26:04 PM Asia/Kolkata"

jshell> DateTimeFormatter dtf = DateTimeFormatter.ofPattern("dd MMMM yy h:mm:ss a zz")
dtf ==> Value(DayOfMonth,2)' 'Text(MonthOfYear)' 'Reduced ... y,SHORT)' 'ZoneText(SHORT)

jshell> ZonedDateTime.now().format(dtf)
$69 ==> "01 August 18 6:26:13 PM IST"

jshell> DateTimeFormatter dtf = DateTimeFormatter.ofPattern("dd MMMM yy h:mm:ss a O")
dtf ==> Value(DayOfMonth,2)' 'Text(MonthOfYear)' 'Reduced ... )' 'LocalizedOffset(SHORT)

jshell> ZonedDateTime.now().format(dtf)
$72 ==> "01 August 18 6:26:27 PM GMT+5:30"
```

`java.time.format.DateTimeFormatter`附带了基于 ISO 标准的大量默认格式。当您处理日期操作而没有用户参与时，这些格式应该足够了，也就是说，当日期和时间在应用程序的不同层之间交换时。

但是，为了向最终用户呈现日期和时间信息，我们需要以可读的格式对其进行格式化，为此，我们需要一个自定义的`DateTimeFormatter`。如果您需要自定义的`java.time.format.DateTimeFormatter`，有两种创建方式：

+   使用模式，例如 dd MMMM yyyy 和`java.time.format.DateTimeFormatter`中的`ofPattern()`方法

+   使用`java.time.DateTimeFormatterBuilder`

**使用模式**：

我们创建一个`java.time.format.DateTimeFormatter`的实例，如下所示：

```java
jshell> DateTimeFormatter dtf = DateTimeFormatter.ofPattern("dd MMMM yy h:mm:ss a VV")
dtf ==> Value(DayOfMonth,2)' 'Text(MonthOfYear)' 'Reduced ... mPmOfDay,SHORT)' 'ZoneId()
```

然后我们将格式应用到日期和时间实例：

```java
jshell> ZonedDateTime.now().format(dtf)
$92 ==> "01 August 18 7:25:00 PM Asia/Kolkata"
```

模式方法也使用`DateTimeFormatterBuilder`，其中构建器解析给定的格式字符串以构建`DateTimeFormatter`对象。

**使用`java.time.format.DateTimeFormatterBuilder`：**

让我们使用`DateTimeFormatterBuilder`来构建`DateTimeFormatter`，如下所示：

```java
jshell> DateTimeFormatter dtf = new DateTimeFormatterBuilder().
 ...> appendValue(DAY_OF_MONTH, 2).
 ...> appendLiteral(" ").
 ...> appendText(MONTH_OF_YEAR).
 ...> appendLiteral(" ").
 ...> appendValue(YEAR, 4).
 ...> toFormatter()
dtf ==> Value(DayOfMonth,2)' 'Text(MonthOfYear)' 'Value(Year,4)

jshell> LocalDate.now().format(dtf) E$106 ==> "01 August 2018"
```

您可以观察到`DateTimeFormatter`对象由一组指令组成，用于表示日期和时间。这些指令以`Value()`、`Text()`和分隔符的形式呈现。
