# 第十一章：杂项

在本章中，我们将涵盖以下内容：

+   在 Java 7 中处理周

+   在 Java 7 中使用货币

+   使用 NumericShaper.Range 枚举支持数字显示

+   Java 7 中的 JavaBean 改进

+   在 Java 7 中处理区域设置和 Locale.Builder 类

+   处理空引用

+   在 Java 7 中使用新的 BitSet 方法

# 介绍

本章将介绍 Java 7 中许多不适合前几章的新内容。其中许多增强功能具有潜在的广泛应用，例如在*在 Java 7 中处理区域设置和 Locale.Builder 类*中讨论的`java.lang.Objects`类和`java.util.Locale`类的改进。其他更专业，例如对`java.util.BitSet`类的改进，这在*在 Java 7 中使用新的 BitSet 方法*中有所涉及。

在处理周和货币方面进行了许多改进。当前周数和每年的周数计算受区域设置的影响。此外，现在可以确定平台上可用的货币。这些问题在*在 Java 7 中处理周*和*在 Java 7 中使用货币*中有所说明。

添加了一个新的枚举，以便在不同语言中显示数字。讨论了使用`java.awt.font.NumericShaper`类来支持此工作的*使用 NumericShaper.Range 枚举支持数字显示*配方。在 JavaBeans 的支持方面也有改进，这在*Java 7 中的 JavaBean 改进*配方中有所讨论。

还有许多增强功能，不值得单独列为配方。本介绍的其余部分都致力于这些主题。

## Unicode 6.0

**Unicode 6.0**是 Unicode 标准的最新修订版。Java 7 通过添加数千个更多的字符和许多新方法来支持此版本。此外，正则表达式模式匹配使用**\u**或**\x**转义序列支持 Unicode 6.0。

`Character.UnicodeBlock`类中添加了许多新的字符块。Java 7 中添加了`Character.UnicodeScript`枚举，用于表示**Unicode 标准附录＃24：脚本名称**中定义的字符脚本。

### 注意

有关 Unicode 标准附录＃24：脚本名称的更多信息，请访问[`download.oracle.com/javase/7/docs/api/index.html`](http://download.oracle.com/javase/7/docs/api/index.html)。

`Character`类中添加了几种方法，以支持 Unicode 操作。以下是它们在字符串朝鲜圆上的使用示例，这是基于区域设置的朝鲜圆的中文显示名称，以及在中国大陆使用的简化脚本。将以下代码序列添加到新应用程序中：

```java
int codePoint = Character.codePointAt("朝鲜圆", 0);
System.out.println("isBmpCodePoint: " + Character.isBmpCodePoint(codePoint));
System.out.println("isSurrogate: " + Character.isSurrogate('朝'));
System.out.println("highSurrogate: " + (int)Character.highSurrogate(codePoint));
System.out.println("lowSurrogate: " + (int)Character.lowSurrogate(codePoint));
System.out.println("isAlphabetic: " + Character.isAlphabetic(codePoint));
System.out.println("isIdeographic: " + Character.isIdeographic(codePoint));
System.out.println("getName: " + Character.getName(codePoint));

```

执行时，您的输出应如下所示：

**isBmpCodePoint: true**

**isSurrogate: false**

**highSurrogate: 55257**

**lowSurrogate: 57117**

**isAlphabetic: true**

**isIdeographic: true**

**getName: CJK UNIFIED IDEOGRAPHS 671D**

由于字符不是 Unicode 代理代码，因此`highSurrogate`和`lowSurrogate`方法的结果是无用的。

### 注意

有关 Unicode 6.0 的更多信息，请访问[`www.unicode.org/versions/Unicode6.0.0/`](http://www.unicode.org/versions/Unicode6.0.0/)。

## 原始类型和比较方法

Java 7 引入了用于比较原始数据类型`Boolean, byte, long, short`和`int`的新静态方法。每个包装类现在都有一个`compare`方法，它接受两个数据类型的实例作为参数，并返回表示比较结果的整数。例如，您以前需要使用`compareTo`方法来比较两个布尔变量 x 和 y，如下所示：

```java
Boolean.valueOf(x).compareTo(Boolean.valueOf(y))

```

现在可以使用`compare`方法如下：

```java
Boolean.compare(x,y);

```

虽然这对于布尔数据类型是 Java 的新功能，但`compare`方法以前已经适用于`double`和`float`。此外，在 7 中，`parse, valueof`和`decode`方法用于将字符串转换为数值，将接受`Byte, Short, Integer, Long`和`BigInteger`的前导加号（+）标记，以及`Float, Double`和`BigDecimal`，这些类型以前接受该标记。

## 全局记录器

`java.util.logging.Logger`类有一个新方法`getGlobal`，用于检索名为`GLOBAL_LOGGER_NAME`的全局记录器对象。`Logger`类的静态字段`global`在`Logger`类与`LogManager`类一起使用时容易发生死锁，因为两个类都会等待对方完成初始化。`getGlobal`方法是访问`全局记录器`对象的首选方式，以防止这种死锁。

## JavaDocs 改进

从结构上讲，JavaDocs 在 Java 7 中有了重大改进。现在，通过使用`HTMLTree`类来创建文档树来生成 HTML 页面，从而实现了更准确的 HTML 生成和更少的无效页面。

JavaDocs 的外部变化也有一些，其中一些是为了符合新的**第五百零八部分**可访问性指南。这些指南旨在确保屏幕阅读器能够准确地将 HTML 页面翻译成可听的输出。主要结果是在表格上添加了更多的标题和标题。JavaDocs 现在还使用 CSS 样式表来简化页面外观的更改。

## JVM 性能增强

Java HotSpotTM 虚拟机的性能已经得到了改进。这些改进大多数不在开发人员的控制范围之内，而且具有专业性质。感兴趣的读者可以在[`docs.oracle.com/javase/7/docs/technotes/guides/vm/performance-enhancements-7.html`](http://docs.oracle.com/javase/7/docs/technotes/guides/vm/performance-enhancements-7.html)找到有关这些增强的更多详细信息。

# 在 Java 7 中处理周

一些应用程序关心一年中的周数和本年的当前周数。众所周知，一年有 52 周，但 52 周乘以每周 7 天等于每年 364 天，而不是实际的 365 天。**周数**用于指代一年中的周。但是如何计算呢？Java 7 引入了几种方法来支持确定一年中的周。在本教程中，我们将检查这些方法，并看看如何计算与周相关的值。**ISO 8601**标准提供了表示日期和时间的方法。`java.util.GregorianCalendar`类支持此标准，除了以下部分中描述的内容。

## 准备工作

使用这些基于周的方法，我们需要：

1.  创建`Calendar`类的实例。

1.  根据需要使用其方法。

## 如何做...

某些抽象`java.util.Calendar`类的实现不支持周计算。要确定`Calendar`实现是否支持周计算，我们需要执行`isWeekDateSupported`方法。如果提供支持，则返回`true`。要返回当前日历年的周数，请使用`getWeeksInWeekYear`方法。要确定当前日期的周，请使用`get`方法，并将`WEEK_OF_YEAR`作为其参数。

1.  创建一个新的控制台应用程序。将以下代码添加到`main`方法：

```java
Calendar calendar = Calendar.getInstance();
if(calendar.isWeekDateSupported()) {
System.out.println("Number of weeks in this year: " + calendar.getWeeksInWeekYear());
System.out.println("Current week number: " + calendar.get(Calendar.WEEK_OF_YEAR));
}

```

1.  执行应用程序。您的输出应如下所示，但值将取决于应用程序执行的日期：

**今年的周数：53**

**当前周数：48**

## 工作原理...

创建了`Calendar`类的一个实例。这通常是`GregorianCalendar`类的一个实例。`if`语句由`isWeekDateSupported`方法控制。它返回`true`，导致执行`getWeeksInWeekYear`和`get`方法。`get`方法传入了字段`WEEK_OF_YEAR`，返回当前的周数。

## 还有更多...

可以使用`setWeekDate`方法设置日期。此方法有三个参数，指定年、周和日。它提供了一种根据周设置日期的便捷技术。以下是通过将年份设置为 2012 年，将周设置为该年的第 16 周，将日期设置为该周的第三天来说明此过程：

```java
calendar.setWeekDate(2012, 16, 3);
System.out.println(DateFormat.getDateTimeInstance(
DateFormat.LONG, DateFormat.LONG).format(calendar.getTime()));

```

执行此代码时，我们得到以下输出：

**2012 年 4 月 17 日下午 12:00:08 CDT**

一年中第一周和最后一周的计算方式取决于区域设置。`GregorianCalendar`类的`WEEK_OF_YEAR`字段范围从 1 到 53，其中 53 代表闰周。一年中的第一周是：

+   最早的七天周期

+   从一周的第一天开始（`getFirstDayOfWeek`）

+   其中至少包含一周的最小天数（`getMinimalDaysInFirstWeek`）

`getFirstDayOfWeek`和`getMinimalDaysInFirstWeek`方法是与区域设置相关的。例如，`getFirstDayOfWeek`方法返回一个整数，表示该区域设置的一周的第一天。在美国，它是星期日，但在法国是星期一。

一年中的第一周和最后一周可能有不同的日历年。考虑以下代码序列。日历设置为 2022 年第一周的第一天：

```java
calendar.setWeekDate(2022, 1, 1);
System.out.println(DateFormat.getDateTimeInstance(
DateFormat.LONG, DateFormat.LONG).format(calendar.getTime()));

```

执行时，我们得到以下输出：

**2021 年 12 月 26 日下午 12:15:39 CST**

这表明这周实际上是从上一年开始的。

此外，`TimeZone`和`SimpleTimeZone`类有一个`observesDaylightTime`方法，如果时区遵守夏令时，则返回`true`。以下代码序列创建了一个`SimpleTimeZone`类的实例，然后确定是否支持夏令时。使用的时区是**中央标准时间**（**CST**）：

```java
SimpleTimeZone simpleTimeZone = new SimpleTimeZone(
-21600000,
"CST",
Calendar.MARCH, 1, -Calendar.SUNDAY,
7200000,
Calendar.NOVEMBER, -1, Calendar.SUNDAY,
7200000,
3600000);
System.out.println(simpleTimeZone.getDisplayName() + " - " +
simpleTimeZone.observesDaylightTime());

```

执行此序列时，您应该获得以下输出：

中央标准时间-真

# 在 Java 7 中使用 Currency 类

`java.util.Currency`类引入了四种检索有关可用货币及其属性的信息的新方法。本示例说明了以下方法的使用：

+   `getAvailableCurrencies：`此方法返回一组可用的货币

+   `getNumericCode：`此方法返回货币的 ISO 4217 数字代码

+   `getDisplayName：`此重载方法返回表示货币显示名称的字符串。一个方法传递了一个`Locale`对象。返回的字符串是特定于该区域设置的。

## 准备就绪

`getAvailableCurrencies`方法是静态的，因此应该针对类名执行。其他方法针对`Currency`类的实例执行。

## 如何做...

1.  创建一个新的控制台应用程序。将以下代码添加到`main`方法中：

```java
Set<Currency> currencies = Currency.getAvailableCurrencies();
for (Currency currency : currencies) {
System.out.printf("%s - %s - %s\n", currency.getDisplayName(),
currency.getDisplayName(Locale.GERMAN),
currency.getNumericCode());
}

```

1.  执行应用程序时，您应该获得类似以下内容的输出。但是，每个的第一部分可能会有所不同，这取决于当前的区域设置。

**朝鲜元 - 朝鲜元 - 408**

**欧元 - 欧元 - 978**

**荷兰盾 - 荷兰盾 - 528**

**福克兰群岛镑 - 福克兰-镑 - 238**

**丹麦克朗 - 丹麦克朗 - 208**

**伯利兹元 - 伯利兹元 - 84**

## 它是如何工作的...

代码序列从生成代表当前系统配置的`Currency`对象的`Set`开始。对每个集合元素执行了重载的`getDisplayName`方法。使用了`Locale.GERMAN`参数来说明此方法的使用。显示的最后一个值是货币的数字代码。

# 使用 NumericShaper.Range 枚举来支持数字的显示

在本示例中，我们将演示使用`java.awt.font.NumericShaper.Range`枚举来支持使用`java.awt.font.NumericShaper`类显示数字。有时希望使用不同于当前使用的语言显示数字。例如，在关于蒙古语的英语教程中，我们可能希望用英语解释数字系统，但使用蒙古数字显示数字。`NumericShaper`类提供了这种支持。新的`NumericShaper.Range`枚举简化了这种支持。

## 准备工作

使用`NumericShaper.Range`枚举来显示数字：

1.  创建一个`HashMap`来保存显示属性信息。

1.  创建一个`Font`对象来定义要使用的字体。

1.  指定要显示文本的 Unicode 字符范围。

1.  创建一个`FontRenderContext`对象来保存有关如何测量要显示的文本的信息。

1.  创建一个`TextLayout`的实例，并在`paintComponent`方法中使用它来渲染文本。

## 操作步骤...

我们将演示使用`NumericShaper.Range`枚举来显示蒙古数字。这是在[`download.oracle.com/javase/tutorial/i18n/text/shapedDigits.html`](http://download.oracle.com/javase/tutorial/i18n/text/shapedDigits.html)中找到的示例的简化版本。

1.  创建一个扩展`JFrame`类的应用程序，如下所示。我们将在`NumericShaperPanel`类中演示`NumericShaper`类的使用：

```java
public class NumericShaperExample extends JFrame {
public NumericShaperExample() {
Container container = this.getContentPane();
container.add("Center", new NumericShaperPanel());
this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
this.setTitle("NumericShaper Example");
this.setSize(250, 120);
}
public static void main(String[] args) {
new NumericShaperExample();.setVisible(true)
}
NumericShaper.Range enumeration using, for digit display}

```

1.  接下来，将`NumericShaperPanel`类添加到项目中，如下所示：

```java
public class NumericShaperPanel extends JPanel {
private TextLayout layout;
public NumericShaperPanel() {
String text = "0 1 2 3 4 5 6 7 8 9";
HashMap map = new HashMap();
Font font = new Font("Mongolian Baiti", Font.PLAIN, 32);
map.put(TextAttribute.FONT, font);
map.put(TextAttribute.NUMERIC_SHAPING,
NumericShaper.getShaper(NumericShaper.Range. MONGOLIAN));
FontRenderContext fontRenderContext =
new FontRenderContext(null, false, false);
layout = new TextLayout(text, map, fontRenderContext);
}
public void paintComponent(Graphics g) {
Graphics2D g2d = (Graphics2D) g;
layout.draw(g2d, 10, 50);
}
}

```

1.  执行应用程序。您的输出应该如下所示：

![操作步骤...](img/5627_11_01.jpg)

## 工作原理...

在`main`方法中，创建了`NumericShaperExample`类的一个实例。在其构造函数中，创建了`NumericShaperPanel`类的一个实例，并将其添加到窗口的中心。设置了窗口的标题、默认关闭操作和大小。接下来，窗口被显示出来。

在`NumericShaperPanel`类的构造函数中，创建了一个文本字符串以及一个`HashMap`来保存显示的基本特性。将此映射用作`TextLayout`构造函数的参数，以及要显示的字符串和映射。使用蒙古 Baiti 字体和 MONGOLIAN 范围显示蒙古文。我们使用这种字体来演示`NumericShaper`类的新方法。

`NumericShaper`类已添加了新方法，使得在不同语言中显示数字值更加容易。`getShaper`方法被重载，其中一个版本接受一个`NumericShaper.Range`枚举值。该值指定要使用的语言。`NumericShaper.Range`枚举已添加以表示给定语言中数字的 Unicode 字符范围。

在`paintComponent`方法中，使用`Graphics2D`对象作为`draw`方法的参数来将字符串渲染到窗口中。

## 还有更多...

`getContextualShaper`方法用于控制在与不同脚本一起使用时如何显示数字。这意味着如果在数字之前使用日语脚本，则会显示日语数字。该方法接受一组`NumericShaper.Range`枚举值。

`shape`方法还使用范围来指定要在数组中的起始和结束索引处使用的脚本。`getRangeSet`方法返回`NumericShaper`实例使用的一组`NumericShaper.Range`。

# Java 7 中的 JavaBean 增强功能

**JavaBean**是构建 Java 应用程序可重用组件的一种方式。它们是遵循特定命名约定的 Java 类。在 Java 7 中添加了几个 JavaBean 增强功能。在这里，我们将重点关注`java.beans.Expression`类，它在执行方法时非常有用。`execute`方法已经添加以实现这一功能。

## 准备工作

使用`Expression`类来执行方法：

1.  为方法创建参数数组，如果需要的话。

1.  创建`Expression`类的一个实例，指定要执行方法的对象、方法名称和任何需要的参数。

1.  针对表达式调用`execute`方法。

1.  如有必要，使用`getValue`方法获取方法执行的结果。

## 如何做...

1.  创建一个新的控制台应用程序。创建两个类：`JavaBeanExample`，其中包含`main`方法和`Person`类。`Person`类包含一个用于名称的单个字段，以及构造函数、getter 方法和 setter 方法：

```java
public class Person {
private String name;
public Person() {
this("Jane", 23);
}
public Person(String name, int age) {
this.name = name;
}
public String getName() {
return name;
}
public void setName(String name) {
this.name = name;
}
}

```

1.  在`JavaBeanExample`类的`main`方法中，我们将创建`Person`类的一个实例，并使用`Expression`类来执行其`getName`和`setName`方法：

```java
public static void main(String[] args) throws Exception {
Person person = new Person();
String arguments[] = {"Peter"};
Expression expression = new Expression(null, person, "setName", arguments);
System.out.println("Name: " + person.getName());
expression.execute();
System.out.println("Name: " + person.getName());
System.out.println();
expression = new Expression(null, person, "getName", null);
System.out.println("Name: " + person.getName());
expression.execute();
System.out.println("getValue: " + expression.getValue());
}

```

1.  执行应用程序。其输出应如下所示：

**名称：Jane**

**名称：Peter**

**名称：Peter**

**getValue：Peter**

## 它是如何工作的...

`Person`类使用了一个名为 name 的字段。`getName`和`setName`方法是从`main`方法中使用的，其中创建了一个`Person`实例。`Expression`类的构造函数有四个参数。第一个参数在本例中没有使用，但可以用来定义方法执行的返回值。第二个参数是方法将被执行的对象。第三个参数是包含方法名称的字符串，最后一个参数是包含方法使用的参数的数组。

在第一个序列中，使用`Peter`作为参数执行了`setName`方法。应用程序的输出显示名称最初为`Jane`，但在执行`execute`方法后更改为`Peter`。

在第二个序列中，执行了`getName`方法。`getValue`方法返回方法执行的结果。输出显示`getName`方法返回了`Peter`。

## 还有更多...

`java.bean`包的类还有其他增强。例如，`FeatureDescriptor`和`PropertyChangeEvent`类中的`toString`方法已被重写，以提供更有意义的描述。

`Introspector`类提供了一种了解 Java Bean 的属性、方法和事件的方式，而不使用可能很繁琐的反射 API。该类已添加了一个`getBeanInfo`方法，该方法使用`Inspector`类的控制标志来影响返回的`BeanInfo`对象。

`Transient`注解已添加以控制包含什么。属性的`true`值意味着应忽略带注解的特性。

`XMLDecoder`类中添加了一个新的构造函数，接受一个`InputSource`对象。此外，添加了`createHandler`方法，返回一个`DefaultHandler`对象。此处理程序用于解析`XMLEncoder`类创建的 XML 存档。

`XMLEncoder`类中添加了一个新的构造函数。这允许使用特定的字符集和特定的缩进将 JavaBeans 写入`OutputStream`。

# 在 Java 7 中处理区域设置和`Locale.Builder`类

`java.util.Locale.Builder`类已添加到 Java 7 中，并提供了一种简单的创建区域设置的方法。`Locale.Category`枚举也是新的，使得在显示和格式化目的上使用不同的区域设置变得容易。我们首先将看一下`Locale.Builder`类的使用，然后检查其他区域设置的改进以及在*还有更多..*部分中使用`Locale.Category`枚举。

## 准备工作

构建和使用新的`Locale`对象：

1.  创建`Builder`类的一个实例。

1.  使用类的相关方法设置所需的属性。

1.  根据需要使用`Locale`对象。

## 如何做...

1.  创建一个新的控制台应用程序。在`main`方法中，添加以下代码。我们将创建一个基于东亚美尼亚语的区域设置，使用意大利的拉丁文。通过使用`setWeekDate`方法，演示了该区域设置，显示了 2012 年第 16 周的第三天的日期。这种方法在*Java 7 中处理周*中有更详细的讨论：

```java
Calendar calendar = Calendar.getInstance();
calendar.setWeekDate(2012, 16, 3);
Builder builder = new Builder();
builder.setLanguage("hy");
builder.setScript("Latn");
builder.setRegion("IT");
builder.setVariant("arevela");
Locale locale = builder.build();
Locale.setDefault(locale);
System.out.println(DateFormat.getDateTimeInstance(
DateFormat.LONG, DateFormat.LONG).format(calendar.getTime()));
System.out.println("" + locale.getDisplayLanguage());

```

1.  第二个示例构建了一个基于中国语言的区域设置，使用了在中国大陆使用的简体字：

```java
builder.setLanguage("zh");
builder.setScript("Hans");
builder.setRegion("CN");
locale = builder.build();
Locale.setDefault(locale);
System.out.println(DateFormat.getDateTimeInstance(
DateFormat.LONG, DateFormat.LONG).format(calendar.getTime()));
System.out.println("" + locale.getDisplayLanguage());

```

1.  执行时，输出应如下所示：

**April 17, 2012 7:25:42 PM CDT**

**亚美尼亚语**

**2012 年 4 月 17 日 下午 07 时 25 分 42 秒**

**中文**

## 工作原理...

创建了`Builder`对象。使用该对象，我们应用了方法来设置区域设置的语言、脚本和地区。然后执行了`build`方法，并返回了一个`Locale`对象。我们使用这个区域设置来显示日期和区域设置的显示语言。这是两次执行的。首先是亚美尼亚语，然后是中文。

## 还有更多...

能够标记一条信息以指示所使用的语言是很重要的。为此目的使用了一个标签。一组标准标签由**IETF BCP 47**标准定义。Java 7 符合这一标准，并添加了几种方法来处理标签。

该标准支持对标签的扩展概念。这些扩展可用于提供有关区域设置的更多信息。有两种类型：

+   Unicode 区域设置扩展

+   私有使用扩展

Unicode 区域设置扩展由**Unicode 通用区域设置数据存储库**（**CLDR**）（[`cldr.unicode.org/`](http://cldr.unicode.org/)）定义。这些扩展涉及非语言信息，如货币和日期。CLDR 维护了一个区域设置信息的标准存储库。私有使用扩展用于指定特定于平台的信息，例如与操作系统或编程语言相关的信息。

### 注意

有关 IETF BCP 47 标准的更多信息，请访问[`tools.ietf.org/rfc/bcp/bcp47.txt`](http://tools.ietf.org/rfc/bcp/bcp47.txt)。

扩展由键/值对组成。键是一个单个字符，值遵循以下格式：

```java
SUBTAG ('-' SUBTAG)*

```

`SUBTAG`由一系列字母数字字符组成。对于 Unicode 区域设置扩展，值必须至少为两个字符，但不超过 8 个字符的长度。对于私有使用扩展，允许 1 到 8 个字符。所有扩展字符串不区分大小写。

Unicode 区域设置扩展的键为**u**，私有使用扩展的键为**x**。这些扩展可以添加到区域设置中，以提供额外的信息，例如要使用的日历编号类型。

可以使用的键列在下表中：

| 键代码 | 描述 |
| --- | --- |
| *ca* | 用于确定日期的日历算法 |
| *co* | 整理—语言中使用的排序 |
| *ka* | 整理参数—用于指定排序 |
| *cu* | 货币类型信息 |
| *nu* | 编号系统 |
| *va* | 常见变体类型 |

键和类型的示例列在下表中：

| 键/类型 | 含义 |
| --- | --- |
| nu-armnlow | 亚美尼亚小写数字 |
| ca-indian | 印度日历 |

已添加了几种方法来使用这些扩展。`getExtensionKeys`方法返回一个包含区域设置中使用的所有键的`Character`对象集。同样，`getUnicodeLocaleAttributes`和`getUnicodeLocaleKeys`方法返回一个列出属性和可用的 Unicode 键的字符串集。如果没有可用的扩展，这些方法将返回一个空集。如果已知键，则`getExtension`方法或`getUnicodeLocaleType`方法将返回一个包含该键值的字符串。

对于给定的区域设置，`getScript, getDisplayScript`和`toLanguageTag`方法分别返回脚本、脚本的可显示名称和区域设置的格式良好的**BCP 47**标签。`getDisplayScript`方法还将返回给定区域设置的脚本的可显示名称。

接下来的部分讨论了使用`setDefault`方法同时控制使用两种不同区域设置显示信息的方法。

### 使用`Locale.Category`枚举来使用两种不同的区域设置显示信息

`Locale.Category`枚举已添加到 Java 7。它有两个值，`DISPLAY`和`FORMAT`。这允许为格式类型资源（日期、数字和货币）和显示资源（应用程序的 GUI 方面）设置默认区域设置。例如，应用程序的一部分可以将格式设置为适应一个区域设置，比如`JAPANESE`，同时在另一个区域设置中显示相关信息，比如`GERMAN`。

考虑以下示例：

```java
Locale locale = Locale.getDefault();
Calendar calendar = Calendar.getInstance();
calendar.setWeekDate(2012, 16, 3);
System.out.println(DateFormat.getDateTimeInstance(
DateFormat.LONG, DateFormat.LONG).format(calendar.getTime()));
System.out.println(ocale.getDisplayLanguage());
Locale.setDefault(Locale.Category.FORMAT, Locale.JAPANESE);
Locale.setDefault(Locale.Category.DISPLAY, Locale.GERMAN);
System.out.println(DateFormat.getDateTimeInstance(
DateFormat.LONG, DateFormat.LONG).format(calendar.getTime()));
System.out.println(locale.getDisplayLanguage());

```

当执行此代码序列时，您应该会得到类似以下的输出。初始日期和显示语言可能会因默认区域设置而有所不同。

**2012 年 4 月 17 日下午 7:15:14 CDT**

**英语**

**2012/04/17 19:15:14 CDT**

**英语**

已检索默认区域设置，并使用`setWeekDate`方法设置了一个日期。这个方法在*在 Java 7 中使用星期*示例中有更详细的讨论。接下来，打印日期和显示语言。显示被重复，只是使用`setDefault`方法更改了默认区域设置。显示资源已更改为使用`Locale.JAPANESE`，格式类型资源已更改为`Locale.GERMAN`。输出反映了这一变化。

# 处理 null 引用

`java.lang.NullPointerException`是一个相当常见的异常。当尝试对包含 null 值的引用变量执行方法时，就会发生这种情况。在这个示例中，我们将研究各种可用的技术来解决这种类型的异常。

`java.util.Objects`类已被引入，并提供了许多静态方法来处理需要处理 null 值的情况。使用这个类简化了对 null 值的测试。

*还有更多..*部分讨论了使用空列表的情况，这可以用来代替返回 null。`java.util.Collections`类有三个返回空列表的方法。

## 准备就绪

使用`Objects`类来覆盖`equals`和`hashCode`方法：

1.  覆盖目标类中的方法。

1.  使用`Objects`类的`equals`方法来避免在`equals`方法中检查 null 值的显式代码。

1.  使用`Objects`类的`hashCode`方法来避免在`hashCode`方法中检查 null 值的显式代码。

## 如何做...

1.  创建一个新的控制台应用程序。我们将创建一个`Item`类来演示`Objects`类的使用。在`Item`类中，我们将覆盖`equals`和`hashCode`方法。这些方法是由 NetBeans 的插入代码命令生成的。我们使用这些方法，因为它们说明了`Objects`类的方法并且结构良好。首先按以下方式创建类：

```java
public class Item {
private String name;
private int partNumber;
public Item() {
this("Widget", 0);
}
public Item(String name, int partNumber) {
this.name = Objects.requireNonNull(name);
this.partNumber = partNumber;
}
public String getName() {
return name;
}
public void setName(String name) {
this.name = Objects.requireNonNull(name);
}
public int getPartNumber() {
return partNumber;
null referenceshandling}
public void setPartNumber(int partNumber) {
this.partNumber = partNumber;
}
}

```

1.  接下来，按以下方式覆盖`equals`和`hashCode`方法。它们提供了检查 null 值的代码：

```java
@Override
public boolean equals(Object obj){
if (obj == null) {
return false;
}
if (getClass() != obj.getClass()) {
return false;
}
final Item other = (Item) obj;
if (!Objects.equals(this.name, other.name)) {
return false;
}
if (this.partNumber != other.partNumber) {
return false;
}
return true;
}
@Override
public int hashCode() {
int hash = 7;
hash = 47 * hash + Objects.hashCode(this.name);
hash = 47 * hash + this.partNumber;
return hash;
}

```

1.  通过添加`toString`方法完成类：

```java
@Override
public String toString() {
return name + " - " + partNumber;
}

```

1.  接下来，在`main`方法中添加以下内容：

```java
Item item1 = new Item("Eraser", 2200);
Item item2 = new Item("Eraser", 2200);
Item item3 = new Item("Pencil", 1100);
Item item4 = null;
System.out.println("item1 equals item1: " + item1.equals(item1));
System.out.println("item1 equals item2: " + item1.equals(item2));
System.out.println("item1 equals item3: " + item1.equals(item3));
System.out.println("item1 equals item4: " + item1.equals(item4));
item2.setName(null);
System.out.println("item1 equals item2: " + item1.equals(item2));

```

1.  执行应用程序。您的输出应如下所示：

**item1 等于 item1：true**

**item1 等于 item2：true**

**item1 等于 item3：false**

**item1 等于 item4：false**

**线程"main"中的异常 java.lang.NullPointerException**

**在 java.util.Objects.requireNonNull(Objects.java:201)**

**在 packt.Item.setName(Item.java:23)**

**在 packt.NullReferenceExamples.main(NullReferenceExamples.java:71)**

正如我们将很快看到的，`NullPointerException`是尝试将 null 值分配给 Item 的名称字段的结果。

## 它是如何工作的...

在`equals`方法中，首先进行了一个测试，以确定传递的对象是否为 null。如果是，则返回`false`。进行了一个测试，以确保类是相同类型的。然后使用`equals`方法来查看两个名称字段是否相等。

`Objects`类的`equals`方法的行为如下表所示。相等性的含义由第一个参数的`equals`方法确定：

| 第一个参数 | 第二个参数 | 返回 |
| --- | --- | --- |
| 非 null | 非 null | 如果它们是相同的对象，则为`true`，否则为`false` |
| 非 null | null | `false` |
| null | 非 null | `false` |
| null | null | `true` |

最后的测试比较了两个整数`partNumber`字段的相等性。

在`Item`类的`hashCode`方法中，`Objects`类的`hashCode`方法被应用于名称字段。如果其参数为 null，则该方法将返回 0，否则返回参数的哈希码。然后使用`partNumber`来计算哈希码的最终值。

注意在两个参数构造函数和`setName`方法中使用了`requireNonNull`方法。该方法检查非空参数。如果参数为 null，则抛出`NullPointerException`。这有效地在应用程序中更早地捕获潜在的错误。

`requireNonNull`方法有两个版本，第二个版本接受第二个字符串参数。当发生异常时，此参数会改变生成的消息。用以下代码替换`setName`方法的主体：

```java
this.name = Objects.requireNonNull(name, "The name field requires a non-null value");

```

重新执行应用程序。异常消息现在将显示如下：

**Exception in thread "main" java.lang.NullPointerException: The name field requires a non-null value**

## 还有更多...

有几个其他`Objects`类的方法可能会引起兴趣。此外，第二部分将讨论使用空迭代器来避免空指针异常。

### 其他`Objects`类方法

`Objects`类的`hashCode`方法是重载的。第二个版本接受可变数量的对象作为参数。该方法将使用这些对象的序列生成哈希码。例如，`Item`类的`hashCode`方法可以这样编写：

```java
@Override
public int hashCode() {
return Objects.hash(name,partNumber);
}

```

`deepEquals`方法深度比较两个对象。这意味着它比较的不仅仅是引用值。两个 null 参数被认为是深度相等的。如果两个参数都是数组，则调用`Arrays.deepEqual`方法。对象的相等性由第一个参数的`equals`方法确定。

`compare`方法用于比较前两个参数，根据参数之间的关系返回负值、零或正值。通常，返回 0 表示参数相同。负值表示第一个参数小于第二个参数。正值表示第一个参数大于第二个参数。

如果其参数相同，或者两个参数都为 null，则该方法将返回零。否则，返回值将使用`Comparator`接口的`compare`方法确定。

`Objects`类的`toString`方法用于确保即使对象为 null 也返回字符串。以下序列说明了这个重载方法的使用：

```java
Item item4 = null;
System.out.println("toString: " + Objects.toString(item4));
System.out.println("toString: " + Objects.toString(item4, "Item is null"));

```

当执行时，该方法的第一次使用将显示单词**null**。在第二个版本中，字符串参数显示如下：

**toString: null**

**toString: Item is null**

### 使用空迭代器来避免空指针异常

避免`NullPointerException`的一种方法是在无法创建列表时返回非空值。返回空的`Iterator`可能是有益的。

在 Java 7 中，`Collections`类添加了三种新方法，返回一个`Iterator`、一个`ListIterator`或一个`Enumeration`，它们都是空的。通过返回空，它们可以在不引发空指针异常的情况下使用。

演示使用空列表迭代器，创建一个新的方法，返回一个通用的`ListIterator<String>`，如下所示。使用`if`语句来返回`ListIterator`或空的`ListIterator`：

```java
public static ListIterator<String> returnEmptyListIterator() {
boolean someConditionMet = false;
if(someConditionMet) {
ArrayList<String> list = new ArrayList<>();
// Add elements
ListIterator<String> listIterator = list.listIterator();
return listIterator;
}
else {
return Collections.emptyListIterator();
}
}

```

使用以下`main`方法来测试迭代器的行为：

```java
public static void main(String[] args) {
ListIterator<String> list = returnEmptyListIterator();
while(())String item: list {
System.out.println(item);
}
}

```

执行时，不应有输出。这表示迭代器是空的。如果我们返回 null，我们将收到`NullPointerException`。

`Collections`类的静态`emptyListIterator`方法返回一个`ListIterator`，其方法如下表所列：

| 方法 | 行为 |
| --- | --- |
| `hasNext``hasPrevious` | 总是返回`false` |
| `next``Previous` | 总是抛出`NoSuchElementException` |
| `remove``set` | 总是抛出`IllegalStateException` |
| `add` | 总是抛出`UnsupportedOperationException` |
| `nextIndex` | 总是返回 0 |
| `previousIndex` | 总是返回-1 |

`emptyIterator`方法将返回一个具有以下行为的空迭代器：

| 方法 | 行为 |
| --- | --- |
| `hasNext` | 总是返回`false` |
| `next` | 总是抛出`NoSuchElementException` |
| `remove` | 总是抛出`IllegalStateException` |

`emptyEnumeration`方法返回一个空枚举。它的`hasMoreElements`将始终返回`false`，它的`nextElement`将始终抛出`NoSuchElementException`异常。

# 在 Java 7 中使用新的 BitSet 方法

`java.util.BitSet`类在最新的 Java 版本中增加了几种新方法。这些方法旨在简化大量位的操作，并提供更容易访问有关位位置的信息。位集可用于优先级队列或压缩数据结构。本示例演示了一些新方法。

## 准备工作

要使用新的`BitSet`方法：

1.  创建一个`BitSet`的实例。

1.  根据需要对`BitSet`对象执行方法。

## 如何做...

1.  创建一个新的控制台应用程序。在`main`方法中，创建一个`BitSet`对象的实例。然后声明一个长数字的数组，并使用静态的`valueOf`方法将我们的`BitSet`对象设置为这个长数组的值。添加一个`println`语句，这样我们就可以看到我们的长数字在`BitSet`中的表示方式：

```java
BitSet bitSet = new BitSet();
long[] array = {1, 21, 3};
bitSet = BitSet.valueOf(array);
System.out.println(bitSet);

```

1.  然后，使用`toLongArray`方法将`BitSet`转换回长数字的数组。使用 for 循环打印数组中的值：

```java
long[] tmp = bitSet.toLongArray();
for (long number : tmp) {
System.out.println(number);
}

```

1.  执行应用程序。您应该看到以下输出：

**{0, 64, 66, 68, 128, 129}**

**1**

**21**

**3**

## 它是如何工作的...

创建`BitSet`对象后，我们创建了一个包含三个`long`数字的数组，这些数字用作我们在`BitSet`中希望使用的位序列的表示。`valueOf`方法接受这个表示并将其转换为位序列。

当我们打印出`BitSet`时，我们看到了序列{0, 64, 66, 68, 128, 129}。这个`BitSet`中的每个数字代表了在我们的位序列中设置的位的索引。例如，0 代表数组中的`long`数字 1，因为用于表示 1 的位的索引在位置 0。同样，位 64、66 和 68 被设置为表示我们的`long`数字 21。序列中的第 128 和 129 位被设置为表示我们的`long`数字 3。在下一节中，我们使用`toLongArray`方法将`BitSet`返回到其原始形式。

在我们的示例中，我们使用了一个`long`数字的数组。类似的`valueOf`方法也适用于`byte, LongBuffer`和`ByteBuffer`数组。当使用`LongBuffer`或`ByteBuffer`数组时，缓冲区不会被`valueOf`方法修改，并且`BitSet`不能被转换回缓冲区。相反，必须使用`toLongArray`方法或类似的`toByteArray`方法将`BitSet`转换为字节数组。

## 还有更多...

有两种有用的方法用于定位`BitSet`中的设置或清除位。方法`previousSetBit`以整数表示特定索引作为其参数，并返回表示`BitSet`中最接近的设置位的整数。例如，将以下代码序列添加到我们的先前示例中（使用由长数字`{1, 21, 3}`表示的`BitSet`）：

```java
System.out.println(bitSet.previousSetBit(1));

```

这将导致输出整数 0。这是因为我们将索引 1 的参数传递给`previousSetBit`方法，而我们的`BitSet`中最接近的前一个设置位是在索引 0 处。

`previousClearBit`方法以类似的方式运行。如果我们在上一个示例中执行以下代码：

```java
System.out.println(bitSet.previousClearBit(66));

```

我们将得到整数 65 的输出。位于索引 65 的位是最接近我们的参数 66 的最近的清除位。如果在`BitSet`中不存在这样的位，则两种方法都将返回-1。
