

# 第三章：与日期和时间一起工作

本章包括 20 个问题，涵盖不同的日期时间主题。这些问题主要关注`Calendar` API 和 JDK Date/Time API。关于后者，我们将介绍一些不太流行的 API，如`ChronoUnit`，`ChronoField`，`IsoFields`，`TemporalAdjusters`等。

在本章结束时，你将拥有大量的技巧和窍门在你的工具箱中，这将非常有用，可以解决各种现实世界的日期时间问题。

# 问题

使用以下问题来测试你在日期和时间上的编程能力。我强烈建议你在查看解决方案并下载示例程序之前尝试每个问题：

1.  **定义一天的时间段**：编写一个应用程序，它超越了 AM/PM 标志，并将一天分为四个时间段：*夜间*，*早晨*，*下午*和*傍晚*。根据给定的日期时间和时区生成这些时间段之一。

1.  **在 Date 和 YearMonth 之间转换**：编写一个应用程序，在`java.util.Date`和`java.time.YearMonth`之间进行转换，反之亦然。

1.  **在 int 和 YearMonth 之间转换**：假设给定一个`YearMonth`（例如，2023-02）。将其转换为整数表示（例如，24277），该整数可以转换回`YearMonth`。

1.  **将周/年转换为 Date**：假设给定两个整数，分别表示周和年（例如，第 10 周，2023 年）。编写一个程序，通过`Calendar`将 10-2023 转换为`java.util.Date`，通过`WeekFields` API 将 10-2023 转换为`LocalDate`。反之亦然：从给定的`Date`/`LocalDate`中提取年份和周作为整数。

1.  **检查闰年**：假设给定一个表示年份的整数。编写一个应用程序来检查这一年是否是闰年。提供至少三种解决方案。

1.  **计算给定日期的季度**：假设给定一个`java.util.Date`。编写一个程序，返回包含此日期的季度作为整数（1，2，3 或 4）和作为字符串（Q1，Q2，Q3 或 Q4）。

1.  **获取季度的第一天和最后一天**：假设给定一个`java.util.Date`。编写一个程序，返回包含此日期的季度的第一天和最后一天。以`Date`（基于`Calendar`的实现）和`LocalDate`（基于 JDK 8 Date/Time API 的实现）表示返回的日期。

1.  **从给定季度中提取月份**：假设给定一个季度（作为一个整数，一个字符串（Q1，Q2，Q3 或 Q4），或一个`LocalDate`）。编写一个程序，提取该季度的月份名称。

1.  **计算预产期**：编写一个预产期计算器。

1.  **实现计时器**：编写一个程序，通过`System.nanoTime()`和`Instant.now()`实现计时器。

1.  **提取自午夜以来的毫秒数**：假设给出了一个`LocalDateTime`。编写一个应用程序，计算从午夜到这个`LocalDateTime`经过的毫秒数。

1.  **将日期时间范围分割成等间隔**：假设我们有一个日期时间范围，通过两个`LocalDateTime`实例给出，以及一个整数`n`。编写一个应用程序，将给定的范围分割成`n`个等间隔（`n`个等`LocalDateTime`实例）。

1.  **解释 Clock.systemUTC()和 Clock.systemDefaultZone()之间的区别**：通过有意义的示例解释`systemUTC()`和`systemDefaultZone()`之间的区别。

1.  **显示一周中各天的名称**：通过`java.text.DateFormatSymbols` API 显示一周中各天的名称。

1.  **获取一年中的第一天和最后一天**：假设给出了一个表示年份的整数。编写一个程序，返回这一年的第一天和最后一天。提供一个基于`Calendar` API 的解决方案和一个基于 JDK 8 Date/Time API 的解决方案。

1.  **获取一周的第一天和最后一天**：假设我们有一个整数表示周数（例如，3 代表从当前日期开始的连续三周）。编写一个程序，返回每周的第一天和最后一天。提供一个基于`Calendar` API 的解决方案和一个基于 JDK 8 Date/Time API 的解决方案。

1.  **计算月份的中间日期**：提供一个包含基于`Calendar` API 的代码片段的应用程序，以及一个基于 JDK 8 Date/Time API 的应用程序，分别用于计算给定月份的中间日期作为一个`Date`对象，以及作为一个`LocalDate`对象。

1.  **计算两个日期之间的季度数**：假设通过两个`LocalDate`实例给出了一个日期时间范围。编写一个程序，计算这个范围内包含的季度数。

1.  **将 Calendar 转换为 LocalDateTime**：编写一个程序，将给定的`Calendar`转换为`LocalDateTime`（默认时区），或者转换为`ZonedDateTime`（亚洲/加尔各答时区）。

1.  **计算两个日期之间的周数**：假设我们有一个日期时间范围，以两个`Date`实例或两个`LocalDateTime`实例给出。编写一个应用程序，返回这个范围内包含的周数。对于`Date`范围，基于`Calendar` API 编写一个解决方案，而对于`LocalDateTime`范围，基于 JDK 8 Date/Time API 编写一个解决方案。

以下部分描述了前面问题的解决方案。请记住，通常没有一种正确的方式来解决特定的问题。此外，请记住，这里所示的解释仅包括解决这些问题所需的最有趣和最重要的细节。下载示例解决方案以查看更多细节，并实验程序，请访问 [`github.com/PacktPublishing/Java-Coding-Problems-Second-Edition/tree/main/Chapter03`](https://github.com/PacktPublishing/Java-Coding-Problems-Second-Edition/tree/main/Chapter03)。

# 68. 定义一天的时间段

让我们想象一下，我们想要根据另一个国家（不同时区）的朋友的当地时间，通过像 *早上好*，*下午好* 等消息来向他们打招呼。所以，仅仅有 AM/PM 标志是不够的，因为我们认为一天（24 小时）可以表示为以下时间段：

+   晚上 9:00（或 21:00）– 上午 5:59 = 夜晚

+   早上 6:00 – 上午 11:59 = 上午

+   下午 12:00 – 晚上 5:59（或 17:59）= 下午

+   下午 6:00（或 18:00）– 晚上 8:59（或 20:59）= 晚上

## 在 JDK 16 之前

首先，我们必须获得我们朋友时区对应的时间。为此，我们可以从我们的本地时间开始，给定为 `java.util.Date`，`java.time.LocalTime` 等。如果我们从 `java.util.Date` 开始，那么我们可以按照以下方式获得我们朋友时区的时间：

```java
LocalTime lt = date.toInstant().atZone(zoneId).toLocalTime(); 
```

在这里，`date` 是 `new Date()`，而 `zoneId` 是 `java.time.ZoneId`。当然，我们可以将区域 ID 作为 `String` 传递，并使用 `ZoneId.of(String zoneId)` 方法来获取 `ZoneId` 实例。

如果我们希望从 `LocalTime.now()` 开始，那么我们可以获得我们朋友时区的时间如下：

```java
LocalTime lt = LocalTime.now(zoneId); 
```

接下来，我们可以定义一天的时间段为一组 `LocalTime` 实例，并添加一些条件来确定当前时间段。以下代码示例说明了这一点：

```java
public static String toDayPeriod(Date date, ZoneId zoneId) {
 LocalTime lt = date.toInstant().atZone(zoneId).toLocalTime();
 LocalTime night = LocalTime.of(21, 0, 0);
 LocalTime morning = LocalTime.of(6, 0, 0);
 LocalTime afternoon = LocalTime.of(12, 0, 0);
 LocalTime evening = LocalTime.of(18, 0, 0);
 LocalTime almostMidnight = LocalTime.of(23, 59, 59);
 LocalTime midnight = LocalTime.of(0, 0, 0);
 if((lt.isAfter(night) && lt.isBefore(almostMidnight)) 
  || lt.isAfter(midnight) && (lt.isBefore(morning))) {
   return "night";
  } else if(lt.isAfter(morning) && lt.isBefore(afternoon)) {
   return "morning";
  } else if(lt.isAfter(afternoon) && lt.isBefore(evening)) {
   return "afternoon";
  } else if(lt.isAfter(evening) && lt.isBefore(night)) {
   return "evening";
  }
  return "day";
} 
```

现在，让我们看看如何在 JDK 16+ 中实现这一点。

## JDK 16+

从 JDK 16+ 开始，我们可以通过以下字符串超越 AM/PM 标志：*早上*，*下午*，*晚上* 和 *夜晚*。

这些友好的输出可以通过新的模式 `B` 获取。这个模式从 JDK 16+ 开始通过 `DateTimeFormatter` 和 `DateTimeFormatterBuilder` 可用（你应该熟悉这些 API，如 *第一章*，*问题 18*，在 *图 1.18* 中所示）。

因此，以下代码使用 `DateTimeFormatter` 来举例说明模式 `B` 的使用，表示一天中的时间段：

```java
public static String toDayPeriod(Date date, ZoneId zoneId) {
 ZonedDateTime zdt = date.toInstant().atZone(zoneId);
 DateTimeFormatter formatter 
    = DateTimeFormatter.ofPattern("yyyy-MMM-dd [B]");
 return zdt.withZoneSameInstant(zoneId).format(formatter);
} 
```

这里是澳大利亚/墨尔本的一个输出示例：

```java
2023-Feb-04 at night 
```

你可以在捆绑的代码中看到更多示例。请随意挑战自己调整此代码以重现第一个示例的结果。

# 69. 日期与 YearMonth 之间的转换

将 `java.util.Date` 转换为 JDK 8 的 `java.time.YearMonth` 可以基于 `YearMonth.from(TemporalAccessor temporal)` 实现。`TemporalAccessor` 是一个接口（更确切地说，是一个框架级接口），它提供了对任何时间对象的只读访问，包括日期、时间和偏移量（这些的组合也是允许的）。因此，如果我们把给定的 `java.util.Date` 转换为 `java.time.LocalDate`，那么转换的结果可以传递给 `YearMonth.from()` 如下所示：

```java
public static YearMonth toYearMonth(Date date) {
  return YearMonth.from(date.toInstant()
                  .atZone(ZoneId.systemDefault())
                  .toLocalDate());
} 
```

反过来可以通过 `Date.from(Instant instant)` 获取，如下所示：

```java
public static Date toDate(YearMonth ym) {
  return Date.from(ym.atDay(1).atStartOfDay(
          ZoneId.systemDefault()).toInstant());
} 
```

好吧，这很简单，不是吗？

# 70. 在 int 和 YearMonth 之间进行转换

假设我们有 `YearMonth.now()`，我们想将其转换为整数（例如，这可能在将年/月日期存储在数据库的数字字段中时很有用）。查看以下解决方案：

```java
public static int to(YearMonth u) {
  return (int) u.getLong(ChronoField.PROLEPTIC_MONTH);
} 
```

*proleptic-month* 是一个 `java.time.temporal.TemporalField`，它基本上代表一个日期时间字段，如 *year-of-month*（我们的情况）或 *minute-of-hour*。proleptic-month 从 0 开始，并按顺序从年份 0 计算月份。因此，`getLong()` 返回从今年月份中指定的字段（在这里是 proleptic-month）的值作为一个 `long`。我们可以将这个 `long` 强制转换为 `int`，因为 proleptic-month 不应该超出 `int` 范围（例如，对于 2023/2 返回的 `int` 是 24277）。

反过来可以通过以下方式完成：

```java
public static YearMonth from(int t) {
  return YearMonth.of(1970, 1)
    .with(ChronoField.PROLEPTIC_MONTH, t);
} 
```

你可以从任何年/月开始。1970/1（称为 *epoch* 和 `java.time.Instant` 的起点）的选择只是一个任意的选择。

# 71. 将周/年转换为 Date

让我们考虑 2023 年，第 10 周。相应的日期是 Sun Mar 05 15:15:08 EET 2023（当然，时间部分是相对的）。将年/周转换为 `java.util.Date` 可以通过 `Calendar` API 实现，如下所示的自解释代码片段：

```java
public static Date from(int year, int week) {
  Calendar calendar = Calendar.getInstance();
  calendar.set(Calendar.YEAR, year);
  calendar.set(Calendar.WEEK_OF_YEAR, week);
  calendar.set(Calendar.DAY_OF_WEEK, 1);
  return calendar.getTime();
} 
```

如果你更喜欢获取 `LocalDate` 而不是 `Date`，那么你可以轻松地进行相应的转换，或者你可以依赖 `java.time.temporal.WeekFields`。这个 API 提供了用于处理 *year-of-week*、*month-of-week* 和 *day-of-week* 的几个字段。话虽如此，以下是通过 `WeekFields` 编写的先前解决方案，用于返回 `LocalDate`：

```java
public static LocalDate from(int year, int week) {
  WeekFields weekFields = WeekFields.of(Locale.getDefault());
  return LocalDate.now()
                  .withYear(year)
                  .with(weekFields.weekOfYear(), week)
                  .with(weekFields.dayOfWeek(), 1);
} 
```

另一方面，如果我们有一个 `java.util.Date`，我们想从中提取年和周，那么我们可以使用 `Calendar` API。在这里，我们提取年份：

```java
public static int getYear(Date date) {
  Calendar calendar = Calendar.getInstance();
  calendar.setTime(date);
  return calendar.get(Calendar.YEAR);
} 
```

然后，我们提取周：

```java
public static int getWeek(Date date) { 
  Calendar calendar = Calendar.getInstance();
  calendar.setTime(date);
  return calendar.get(Calendar.WEEK_OF_YEAR);
} 
```

由于 `ChronoField.YEAR` 和 `ChronoField.ALIGNED_WEEK_OF_YEAR`，从 `LocalDate` 中获取年和周很容易：

```java
public static int getYear(LocalDate date) {
  return date.get(ChronoField.YEAR);
}
public static int getWeek(LocalDate date) {
  return date.get(ChronoField.ALIGNED_WEEK_OF_YEAR);
} 
```

当然，获取周也可以通过 `WeekFields` 实现：

```java
return date.get(WeekFields.of(
  Locale.getDefault()).weekOfYear()); 
```

挑战自己从 `Date`/`LocalDate` 中获取周/月和日/周。

# 72. 检查闰年

只要我们知道了什么是闰年，这个问题就变得简单了。简而言之，闰年是指任何可以被 4 整除的年份（即`year % 4 == 0`），且不是世纪年（例如，100，200，……，n00）。然而，如果这个世纪年可以被 400 整除（即`year % 400 == 0`），那么它就是一个闰年。在这种情况下，我们的代码只是一个简单的`if`语句链，如下所示：

```java
public static boolean isLeapYear(int year) {
  if (year % 4 != 0) {
    return false;
  } else if (year % 400 == 0) {
    return true;
  } else if (year % 100 == 0) {
    return false;
  }
  return true;
} 
```

但是，这段代码可以使用`GregorianCalendar`来简化：

```java
public static boolean isLeapYear(int year) {
  return new GregorianCalendar(year, 1, 1).isLeapYear(year);
} 
```

或者，从 JDK 8 开始，我们可以依赖于`java.time.Year` API，如下所示：

```java
public static boolean isLeapYear(int year) {
  return Year.of(year).isLeap(); 
} 
```

在捆绑的代码中，你可以看到更多的方法。

# 73. 计算给定日期的季度

一年有 4 个季度（通常表示为 Q1，Q2，Q3 和 Q4），每个季度有 3 个月。如果我们考虑 1 月是 0，2 月是 1，……，12 月是 11，那么我们可以观察到 1 月/3 = 0，2 月/3 = 0，3 月/3 = 0，0 可以代表 Q1。接下来，3/3 = 1，4/3 = 1，5/3 = 1，所以 1 可以代表 Q2。基于同样的逻辑，6/3 = 2，7/3 = 2，8/3 = 2，所以 2 可以代表 Q3。最后，9/3 = 3，10/3 = 3，11/3 = 3，所以 3 代表 Q4。

基于这个声明和`Calendar` API，我们可以获得以下代码：

```java
public static String quarter(Date date) {
  String[] quarters = {"Q1", "Q2", "Q3", "Q4"};
  Calendar calendar = Calendar.getInstance();
  calendar.setTime(date);
  int quarter = calendar.get(Calendar.MONTH) / 3;
  return quarters[quarter];
} 
```

但从 JDK 8 开始，我们可以依赖于`java.time.temporal.IsoFields`。这个类包含基于 ISO-8601 标准的日历系统中的字段（和单位）。在这些元素中，我们有基于周的年份和我们所感兴趣的*年份季度*。这次，让我们将季度作为整数返回：

```java
public static int quarter(Date date) {
  LocalDate localDate = date.toInstant()
    .atZone(ZoneId.systemDefault()).toLocalDate();
  return localDate.get(IsoFields.QUARTER_OF_YEAR);
} 
```

在捆绑的代码中，你可以看到更多示例，包括一个使用`DateTimeFormatter.ofPattern("QQQ")`的示例。

# 74. 获取一个季度的第一天和最后一天

让我们假设我们通过这个简单的类来表示一个季度的第一天和最后一天：

```java
public final class Quarter {
  private final Date firstDay;
  private final Date lastDay;
  ...
} 
```

接下来，我们有一个`java.util.Date`，我们想要获取包含这个日期的季度的第一天和最后一天。为此，我们可以使用 JDK 8 的`IsoFields.DAY_OF_QUARTER`（我们在前一个问题中介绍了`IsoFields`）。但在我们能够使用`IsoFields`之前，我们必须将给定的`java.util.Date`转换为`LocalDate`，如下所示：

```java
LocalDate localDate = date.toInstant()
  .atZone(ZoneId.systemDefault()).toLocalDate(); 
```

一旦我们有了给定的`Date`作为`LocalDate`，我们就可以通过`IsoFields.DAY_OF_QUARTER`轻松地提取季度的第一天。接下来，我们将 2 个月加到这一天，进入季度的最后一个月（一个季度有 3 个月，所以一年有 4 个季度），我们依赖于`java.time.temporal.TemporalAdjusters`，更确切地说，依赖于`lastDayOfMonth()`来获取季度的最后一天。最后，我们将两个获得的`LocalDate`实例转换为`Date`实例。以下是完整的代码：

```java
public static Quarter quarterDays(Date date) {
  LocalDate localDate = date.toInstant()
    .atZone(ZoneId.systemDefault()).toLocalDate();
  LocalDate firstDay
    = localDate.with(IsoFields.DAY_OF_QUARTER, 1L);
  LocalDate lastDay = firstDay.plusMonths(2)
    .with(TemporalAdjusters.lastDayOfMonth());
  return new Quarter(
    Date.from(firstDay.atStartOfDay(
      ZoneId.systemDefault()).toInstant()),
    Date.from(lastDay.atStartOfDay(
      ZoneId.systemDefault()).toInstant())
  );
} 
```

当然，如果你直接使用`LocalDate`，这些转换就不需要了。但这样，你就有机会学习更多。

在捆绑的代码中，你可以找到更多示例，包括一个完全依赖于`Calendar` API 的示例。

# 75. 从给定的季度中提取月份

如果我们熟悉 JDK 8 的`java.time.Month`，这个问题就变得相当容易解决。通过这个 API，我们可以找到包含给定`LocalDate`的季度的第一个月（1 月为 0，2 月为 1，……），即`Month.from(LocalDate).firstMonthOfQuarter().getValue()`。

一旦我们有了第一个月，很容易获得其他两个，如下所示：

```java
public static List<String> quarterMonths(LocalDate ld) {
  List<String> qmonths = new ArrayList<>();
  int qmonth = Month.from(ld)
    .firstMonthOfQuarter().getValue();
  qmonths.add(Month.of(qmonth).name());
  qmonths.add(Month.of(++qmonth).name());
  qmonths.add(Month.of(++qmonth).name());
  return qmonths;
} 
```

关于将季度本身作为参数传递，这可以通过数字（1、2、3 或 4）或字符串（Q1、Q2、Q3 或 Q4）来实现。如果给定的`quarter`是数字，那么季度的第一个月可以通过`quarter * 3 – 2`来获得，其中`quarter`是 1、2、3 或 4。这次，让我们以函数式风格表达代码：

```java
int qmonth = quarter * 3 - 2;
List<String> qmonths = IntStream.of(
        qmonth, ++qmonth, ++qmonth)
  .mapToObj(Month::of)
  .map(Month::name)
  .collect(Collectors.toList()); 
```

当然，如果你觉得更简洁，那么你可以使用`IntStream.range(qmonth, qmonth+2)`而不是`IntStream.of()`。在捆绑的代码中，你可以找到更多示例。

# 76. 计算预产期

让我们从这两个常数开始：

```java
public static final int PREGNANCY_WEEKS = 40;
public static final int PREGNANCY_DAYS = PREGNANCY_WEEKS * 7; 
```

让我们将第一天视为一个`LocalDate`，并编写一个计算器，打印预产期、剩余天数、已过天数和当前周数。

基本上，预产期是通过将`PREGNANCY_DAYS`加到给定第一天来获得的。进一步，剩余天数是今天和给定第一天之间的差值，而已过天数是`PREGNANCY_DAYS`减去剩余天数。最后，当前周数是通过将已过天数除以 7（因为一周有 7 天）来获得的。基于这些陈述，代码自解释：

```java
public static void pregnancyCalculator(LocalDate firstDay) {
  firstDay = firstDay.plusDays(PREGNANCY_DAYS);
  System.out.println("Due date: " + firstDay);
  LocalDate today = LocalDate.now();
  long betweenDays =    
    Math.abs(ChronoUnit.DAYS.between(firstDay, today));
  long diffDays = PREGNANCY_DAYS - betweenDays;
  long weekNr = diffDays / 7;
  long weekPart = diffDays % 7;
  String week = weekNr + " | " + weekPart;
  System.out.println("Days remaining: " + betweenDays);
  System.out.println("Days in: " + diffDays);
  System.out.println("Week: " + week);
} 
```

看看你是否能想到一种方法来计算另一个重要的日期。

# 77. 实现一个计时器

一个经典的计时器实现依赖于`System.nanoTime()`、`System.currentTimeMillis()`或`Instant.now()`。在所有情况下，我们必须提供启动和停止计时器的支持，以及一些辅助函数来以不同的时间单位获取测量的时间。

虽然基于`Instant.now()`和`currentTimeMillis()`的解决方案在捆绑代码中可用，但这里我们将展示基于`System.nanoTime()`的解决方案：

```java
public final class NanoStopwatch {
  private long startTime;
  private long stopTime;
  private boolean running;
  public void start() {
    this.startTime = System.nanoTime();
    this.running = true;
   }
   public void stop() {
    this.stopTime = System.nanoTime();
    this.running = false;
   }
  //elaspsed time in nanoseconds
   public long getElapsedTime() {
     if (running) {
       return System.nanoTime() - startTime;
     } else {
       return stopTime - startTime;
     } 
  }
} 
```

如果你需要以毫秒或秒为单位返回测量的时间，那么只需添加以下两个辅助函数：

```java
//elaspsed time in millisecods
public long elapsedTimeToMillis(long nanotime) {
  return TimeUnit.MILLISECONDS.convert(
    nanotime, TimeUnit.NANOSECONDS);
}
//elaspsed time in seconds
public long elapsedTimeToSeconds(long nanotime) {
  return TimeUnit.SECONDS.convert(
    nanotime, TimeUnit.NANOSECONDS);
} 
```

这种方法基于`System.nanoTime()`来测量高精度的经过时间。这种方法返回以纳秒为单位的分辨率高的时间，不依赖于系统时钟或任何其他墙钟（如`Instant.now()`或`System.currentTimeMillis()`），因此它不受墙钟常见问题的影响，如闰秒、时间均匀性、同步性问题等。

无论何时你需要一个专业的测量经过时间的工具，请依赖 Micrometer ([`micrometer.io/`](https://micrometer.io/))、JMH ([`openjdk.org/projects/code-tools/jmh/`](https://openjdk.org/projects/code-tools/jmh/))、Gatling ([`gatling.io/open-source/`](https://gatling.io/open-source/))等等。

# 78. 提取自午夜以来的毫秒数

因此，我们有一个日期时间（让我们假设是一个 `LocalDateTime` 或 `LocalTime`），我们想知道从午夜到这个日期时间已经过去了多少毫秒。让我们考虑给定的日期时间是现在：

```java
LocalDateTime now = LocalDateTime.now(); 
```

午夜相对于 `now` 是相对的，因此我们可以找到以下差异：

```java
LocalDateTime midnight = LocalDateTime.of(now.getYear(),
  now.getMonth(), now.getDayOfMonth(), 0, 0, 0); 
```

最后，计算午夜和现在之间的差异（以毫秒为单位）。这可以通过多种方式完成，但可能最简洁的解决方案依赖于 `java.time.temporal.ChronoUnit`。此 API 提供了一组用于操作日期、时间或日期时间的单位，包括毫秒：

```java
System.out.println("Millis: " 
  + ChronoUnit.MILLIS.between(midnight, now)); 
```

在捆绑的代码中，你可以看到更多关于 `ChronoUnit` 的示例。

# 79. 将日期时间范围分割成等间隔

让我们考虑一个日期时间范围（由两个 `LocalDateTime` 实例表示的起始日期和结束日期界定）和一个整数 `n`。为了将给定的范围分割成 `n` 个等间隔，我们首先定义一个 `java.time.Duration` 如下：

```java
Duration range = Duration.between(start, end); 
```

有了这个日期时间范围，我们可以依赖 `dividedBy()` 来获取它的一个副本，该副本被指定为 `n` 分割：

```java
Duration interval = range.dividedBy(n - 1); 
```

最后，我们可以从起始日期（范围的左端头）开始，并反复用 `interval` 值增加它，直到我们达到结束日期（范围的右端头）。在每一步之后，我们将新的日期存储在一个列表中，该列表将在最后返回。以下是完整的代码：

```java
public static List<LocalDateTime> splitInEqualIntervals(
       LocalDateTime start, LocalDateTime end, int n) {
  Duration range = Duration.between(start, end);
  Duration interval = range.dividedBy(n - 1);
  List<LocalDateTime> listOfDates = new ArrayList<>(); 
  LocalDateTime timeline = start;
  for (int i = 0; i < n - 1; i++) {
    listOfDates.add(timeline);
    timeline = timeline.plus(interval);
  }
  listOfDates.add(end);
  return listOfDates;
} 
```

结果的 `listOfDates` 将包含 `n` 个等间隔的日期。

# 80. 解释 Clock.systemUTC() 和 Clock.systemDefaultZone() 之间的差异

让我们从以下三行代码开始：

```java
System.out.println(Clock.systemDefaultZone());
System.out.println(system(ZoneId.systemDefault()));
System.out.println(Clock.systemUTC()); 
```

输出显示前两行是相似的。它们都显示了默认时区（在我的情况下，是欧洲/布加勒斯特）：

```java
SystemClock[Europe/Bucharest]
SystemClock[Europe/Bucharest] 
```

第三行是不同的。在这里，我们看到 `Z` 时区，它是特定于 UTC 时区的，表示存在时区偏移：

```java
SystemClock[Z] 
```

另一方面，创建一个 `Instant` 显示 `Clock.systemUTC()` 和 `Clock.systemDefaultZone()` 产生相同的结果：

```java
System.out.println(Clock.systemDefaultZone().instant());
System.out.println(system(ZoneId.systemDefault()).instant());
System.out.println(Clock.systemUTC().instant()); 
```

在这三种情况下，瞬时时间都是相同的：

```java
2023-02-07T05:26:17.374159500Z
2023-02-07T05:26:17.384811300Z
2023-02-07T05:26:17.384811300Z 
```

但是，当我们尝试从这两个时钟创建日期、时间或日期时间时，差异就出现了。例如，让我们从 `Clock.systemUTC()` 创建一个 `LocalDateTime`：

```java
// 2023-02-07T05:26:17.384811300
System.out.println(LocalDateTime.now(Clock.systemUTC())); 
```

以及，从 `Clock.systemDefaultZone()` 创建一个 `LocalDateTime`：

```java
// 2023-02-07T07:26:17.384811300
System.out.println(LocalDateTime.now(
  Clock.systemDefaultZone())); 
```

我的时区（默认时区，欧洲/布加勒斯特）是 07:26:17。但是，通过 `Clock.systemUTC()` 的时间是 05:26:17。这是因为欧洲/布加勒斯特位于 UTC-2 的偏移量，所以 `systemUTC()` 产生 UTC 时区的日期时间，而 `systemDefaultZone()` 产生当前默认时区的日期时间。然而，它们都产生了相同的 `Instant`。

# 81. 显示星期几的名称

Java 中的一颗隐藏的宝石是 `java.text.DateFormatSymbols`。这个类是日期时间格式化数据（如星期几的名称和月份的名称）的包装器。所有这些名称都是可本地化的。

通常，您会通过 `DateFormat`（如 `SimpleDateFormat`）使用 `DateFormatSymbols`，但为了解决这个问题，我们可以直接使用它，如下面的代码所示：

```java
String[] weekdays = new DateFormatSymbols().getWeekdays();
IntStream.range(1, weekdays.length)
    .mapToObj(t -> String.format("Day: %d -> %s",
       t, weekdays[t]))
    .forEach(System.out::println); 
```

这段代码将按以下方式输出星期的名称：

```java
Day: 1 -> Sunday
...
Day: 7 -> Saturday 
```

挑战自己，想出另一种解决方案。

# 82. 获取年的第一天和最后一天

获取给定年份（作为数值）的第一天和最后一天可以通过 `LocalDate` 和方便的 `TemporalAdjusters`，`firstDayOfYear()` 和 `lastDayOfYear()` 实现。首先，我们从给定年份创建一个 `LocalDate`。接下来，我们使用这个 `LocalDate` 与 `firstDayOfYear()`/`lastDayOfYear()` 结合，如下面的代码所示：

```java
public static String fetchFirstDayOfYear(int year, boolean name) {
  LocalDate ld = LocalDate.ofYearDay(year, 1);
  LocalDate firstDay = ld.with(firstDayOfYear());
  if (!name) {
    return firstDay.toString();
  }
  return DateTimeFormatter.ofPattern("EEEE").format(firstDay);
} 
```

对于最后一天，代码几乎相同：

```java
public static String fetchLastDayOfYear(int year, boolean name) {
  LocalDate ld = LocalDate.ofYearDay(year, 31);
  LocalDate lastDay = ld.with(lastDayOfYear());
  if (!name) {
    return lastDay.toString();
  }
  return DateTimeFormatter.ofPattern("EEEE").format(lastDay);
} 
```

如果标志参数（`name`）为 `false`，则我们通过 `LocalDate.toString()` 返回第一/最后一天，因此我们将得到类似 2020-01-01（2020 年的第一天）和 2020-12-31（2020 年的最后一天）的结果。如果这个标志参数为 `true`，则我们依赖于 `EEEE` 模式来返回年份的第一/最后一天的名字，例如星期三（2020 年的第一天）和星期四（2020 年的最后一天）。

在捆绑的代码中，您还可以找到一个基于 `Calendar` API 的解决方案。

# 83. 获取周的第一天和最后一天

假设给定一个整数（`nrOfWeeks`）表示我们想要提取从现在开始每周的第一天和最后一天的周数。例如，对于给定的 `nrOfWeeks` = 3 和一个本地日期，例如 06/02/2023，我们想要这样：

```java
[
Mon 06/02/2023,
Sun 12/02/2023,
Mon 13/02/2023,
Sun 19/02/2023,
Mon 20/02/2023,
Sun 26/02/2023
] 
```

这比看起来要简单得多。我们只需要从 0 到 `nrOfWeeks` 的循环，以及两个 `TemporalAdjusters` 来适应每周的第一天/最后一天。更确切地说，我们需要 `nextOrSame(DayOfWeek dayOfWeek)` 和 `previousOrSame(DayOfWeek dayOfWeek)` 调整器。

`nextOrSame()` 调整器的角色是将当前日期调整为调整日期之后给定 *星期几* 的首次出现（这可以是 *下个或相同*）。另一方面，`previousOrSame()` 调整器的角色是将当前日期调整为调整日期之前给定 *星期几* 的首次出现（这可以是 *之前或相同*）。例如，如果今天是 [星期二 07/02/2023]，那么 `previousOrSame(DayOfWeek.MONDAY)` 将返回 [星期一 06/02/2023]，而 `nextOrSame(DayOfWeek.SUNDAY)` 将返回 [星期日 12/02/2023]。

基于这些陈述，我们可以通过以下代码解决问题：

```java
public static List<String> weekBoundaries(int nrOfWeeks) {
  List<String> boundaries = new ArrayList<>();
  LocalDate timeline = LocalDate.now();
  DateTimeFormatter dtf = DateTimeFormatter
    .ofPattern("EEE dd/MM/yyyy");
  for (int i = 0; i < nrOfWeeks; i++) {
    boundaries.add(dtf.format(timeline.with(
      previousOrSame(DayOfWeek.MONDAY))));
    boundaries.add(dtf.format(timeline.with(
      nextOrSame(DayOfWeek.SUNDAY))));
    timeline = timeline.plusDays(7);
  }
  return boundaries;
} 
```

在捆绑的代码中，您还可以看到一个基于 `Calendar` API 的解决方案。

# 84. 计算月份中旬

让我们想象我们有一个 `LocalDate`，我们想要从它计算出代表月份中旬的另一个 `LocalDate`。如果我们知道 `LocalDate` API 有一个名为 `lengthOfMonth()` 的方法，它返回一个表示月份天数的整数，那么这可以在几秒钟内完成。所以，我们只需要计算 `lengthOfMonth()`/2，如下面的代码所示：

```java
public static LocalDate middleOfTheMonth(LocalDate date) {
  return LocalDate.of(date.getYear(), date.getMonth(), 
    date.lengthOfMonth() / 2); 
} 
```

在捆绑的代码中，您可以看到一个基于 `Calendar` API 的解决方案。

# 85. 获取两个日期之间的季度数

这只是另一个需要我们深入掌握 Java 日期/时间 API 的问题。这次，我们要讨论的是 `java.time.temporal.IsoFields`，它在 *问题 73* 中被引入。ISO 字段之一是 `QUARTER_YEARS`，它是一个表示 *季度* 概念的时间单位。因此，拥有两个 `LocalDate` 实例，我们可以写出以下代码：

```java
public static long nrOfQuarters(
    LocalDate startDate, LocalDate endDate) {
  return IsoFields.QUARTER_YEARS.between(startDate, endDate);
} 
```

随意挑战自己，为 `java.util.Date`/`Calendar` 提供解决方案。

# 86. 将 Calendar 转换为 LocalDateTime

在 *问题 68* 中，你看到将 `java.util.Date`（日期）转换为 `LocalTime` 可以如下进行：

```java
LocalTime lt = date.toInstant().atZone(zoneId).toLocalTime(); 
```

以同样的方式，我们可以将 `java.util.Date` 转换为 `LocalDateTime`（这里，`zoneId` 被替换为 `ZoneId.systemDefault()`）：

```java
LocalDateTime ldt = date.toInstant().atZone(
  ZoneId.systemDefault()).toLocalDateTime(); 
```

我们还知道，我们可以通过 `getTime()` 方法从 `Calendar` 获取 `java.util.Date`。因此，通过拼凑拼图碎片，我们得到以下代码：

```java
public static LocalDateTime
       toLocalDateTime(Calendar calendar) {
  Date date = calendar.getTime();
  return date.toInstant().atZone(
    ZoneId.systemDefault()).toLocalDateTime();
} 
```

可以通过以下更简短的路径获得相同的结果：

```java
return LocalDateTime.ofInstant(Instant.ofEpochMilli(
  calendar.getTimeInMillis()), ZoneId.systemDefault()); 
```

或者，甚至更短，如下所示：

```java
return LocalDateTime.ofInstant(
  calendar.toInstant(), ZoneId.systemDefault()); 
```

但是，此代码假设给定 `Calendar` 的时间区域是默认时间区域。如果日历有不同的时区（例如，亚洲/加尔各答），那么我们可能会期望返回 `ZonedDateTime` 而不是 `LocalDateTime`。这意味着我们应该相应地调整之前的代码：

```java
public static ZonedDateTime
       toZonedDateTime(Calendar calendar) {
  Date date = calendar.getTime();
  return date.toInstant().atZone(
    calendar.getTimeZone().toZoneId());
} 
```

再次，有一些更简短的版本可用，但我们没有展示这些，因为它们表达性较差：

```java
return ZonedDateTime.ofInstant(
  Instant.ofEpochMilli(calendar.getTimeInMillis()),
    calendar.getTimeZone().toZoneId());
return ZonedDateTime.ofInstant(calendar.toInstant(),
    calendar.getTimeZone().toZoneId()); 
```

完成！

# 87. 获取两个日期之间的周数

如果给定的两个日期是 `LocalDate`（`Time`）的实例，那么我们可以依赖 `java.time.temporal.ChronoUnit`。此 API 提供了一组用于操作日期、时间或日期时间的单元，我们之前在 *问题 78* 中已经使用过。这次，让我们再次使用它来计算两个日期之间的周数：

```java
public static long nrOfWeeks(
    LocalDateTime startLdt, LocalDateTime endLdt) {
  return Math.abs(ChronoUnit.WEEKS.between(
    startLdt, endLdt));
} 
```

另一方面，如果给定的日期是 `java.util.Date`，那么你可以选择将它们转换为 `LocalDateTime` 并使用之前的代码，或者依赖于 `Calendar` API。使用 `Calendar` API 是从开始日期到结束日期循环，每周递增日历日期：

```java
public static long nrOfWeeks(Date startDate, Date endDate) {
  Calendar calendar = Calendar.getInstance();
  calendar.setTime(startDate);
  int weeks = 0;
  while (calendar.getTime().before(endDate)) {
    calendar.add(Calendar.WEEK_OF_YEAR, 1);
    weeks++;
  }
  return weeks;
} 
```

当日历日期超过结束日期时，我们就有周数了。

# 摘要

任务完成！我希望你喜欢这个充满技巧和窍门的简短章节，这些技巧和窍门关于在现实世界应用程序中操作日期和时间。我强烈建议你阅读来自 *Java 编程问题*，*第一版* 的同类型章节，其中包含另外 20 个涵盖其他日期/时间主题的问题。

# 加入我们的 Discord 社区

加入我们社区的 Discord 空间，与作者和其他读者进行讨论：

[`discord.gg/8mgytp5DGQ`](https://discord.gg/8mgytp5DGQ )

![](img/QR_Code1139613064111216156.png)
