from the specified string, ignoring delimiters.

Scanner skip(Pattern

此方法几乎与上一个相同

pattern)

但将 Pattern 作为参数而不是 String。

String

尝试在指定字符串中找到下一个模式

findWithinHorizon(String

constructed from the specified string, ignoring

pattern, int horizon)

分隔符。

String

此方法几乎与上一个相同

findWithinHorizon(Pattern

但将 Pattern 作为参数而不是 String。

pattern, int horizon)

除了前面表格中提到的使用正则表达式的两个 hasNext()方法之外，Scanner 类还提供了几个重载的 hasNext 方法，如果输入中下一个可用的令牌可以检索为该类型，则返回 true。

特定类型。例如：hasNextInt()，hasNextDouble()，hasNextBoolean()，hasNextByte()，hasNextFloat()，hasNextLong()，hasNextShort()，hasNextBigInteger()，hasNextBigDecimal()，hasNext()。

类似地，还有几个重载的 next 方法，用于扫描输入并返回特定类型的下一个令牌。例如：nextextInt()，nextextDouble()，nextextBoolean()，nextextByte()，nextextFloat()，nextextLong()，nextextShort()，nextextBigInteger()，nextextBigDecimal()，nextext()。

关于 Scanner 类的完整参考，请参阅

https://docs.oracle.com/javase/8/docs/api/java/util/Scanner.html.

假设有一个由两个感叹号分隔的输入文本。数据按照以下顺序结构化：

animal!!id!!weight

动物名称是一个字符串，id 是一个整数，而重量是一个双精度浮点数。

使用这种结构，以下是一个示例输入：

Tiger!!123!!221.2!!Fox!!581!!52.50

由于有两种动物，以下是如何在 Java 中使用 Scanner 类解析这些输入数据的示例：

final String input = "Tiger!!123!!221.2!!Fox!!581!!52.50"; final int MAX_COUNT = 2;

String animal;

int id;

double weight;

Scanner scanner = new Scanner(input).useDelimiter("!!"); for (int i=0; i<MAX_COUNT; i++)

{

animal = scanner.next();

id = scanner.nextInt();

weight = scanner.nextDouble();

System.out.printf("animal=[%s], id=[%d], weight=[%.2f]%n", animal, id, weight);

}

scanner.close();

这就是这段代码所发生的事情：

new Scanner(input) 是使用输入字符串构建 scanner 的代码 scanner.useDelimiter("!!") 将分隔符正则表达式设置为"!!"

scanner.next() 从构建的 scanner 中获取下一个字符串令牌 scanner.nextInt() 从 scanner 中获取下一个 int 令牌 scanner.nextDouble() 从 scanner 中获取下一个 double 令牌 scanner.close() 关闭 scanner 对象；在调用此方法后，我们不能从 scanner 生成更多的令牌

如你所猜，我们将从前面的代码中得到以下输出：animal=[Tiger], id=[123], weight=[221.20]

animal=[Fox], id=[581], weight=[52.50]

让我们解析更复杂的输入数据，以更好地理解 Scanner 类的使用。

以下是完整的代码列表：

package example.regex;

import java.util.*;

public class ScannerApi

{

public static void main (String[] args)

{

final String str = "London:Rome#Paris:1234:Munich///Moscow"; Scanner scanner = new Scanner(str);

scanner.useDelimiter("\\p{Punct}+");

final String cityPattern = "\\p{L}+";

while(scanner.hasNext()) {

if(scanner.hasNext(cityPattern)) {

System.out.println(scanner.next());

}

else {

scanner.next();

}

}

scanner.close();

}

}

这段代码中正在发生的事情如下：

new Scanner(str)是使用输入字符串构建扫描器的代码

* scanner.useDelimiter("\\p{Punct}+")将分隔符正则表达式设置为一个或多个标点符号

我们正在使用"\\p{L}+"作为可接受的城市名称模式，这意味着一个或多个 Unicode 字母

scanner.hasNext(cityPattern)如果扫描器的下一个标记与 cityPattern 匹配，则返回 true

scanner.next()从扫描器中检索下一个字符串标记 scanner.close()关闭扫描器对象；在调用此方法后，我们不能从扫描器生成更多标记

编译并运行前面的代码后，将生成以下输出：London

Rome

Paris

Munich

Moscow

**总结**

在本章中，您已经通过 String 和 Scanner 类了解了使用正则表达式的 Java 程序。我们讨论了这两个 API 中与正则表达式处理相关的可用方法，以及我们如何在代码中利用这些方法。

在下一章中，我们将介绍 Pattern 和 Matcher 类，它们是使用正则表达式的程序中最重要的类。

**Java 正则表达式简介**

**表达式 API - 模式**

**和 Matcher 类**

在本章中，我们将向您介绍用于使用正则表达式编写程序的专用 Java API。Java 提供了一个包，java.util.regex，其中包含处理正则表达式所需的所有类和接口。此包位于 java.base 模块中，因此我们不需要在 module-info.java 源文件中显式声明其使用。java.base 模块由所有模块自动要求，它包含最重要的和基本的 JDK 包和类。正则表达式是一个如此重要的话题和工具，Java 9 专家决定将其保留在 java.base 模块中。

我们将介绍此包中的以下类和接口：MatchResult 接口

使用 Pattern 类

使用 Matcher 类

Pattern 和 Matcher 类的各种方法以及如何使用它们解决涉及正则表达式的问题

**MatchResult 接口**

MatchResult 是一个用于表示匹配操作结果的接口。此接口由 Matcher 类实现。此接口包含用于确定正则表达式匹配结果的查询方法。匹配边界、组和组边界只能通过此接口检索，但不能修改。以下是此接口中提供的重要方法的列表：方法

描述

名称

int start()

返回输入中匹配的起始索引

int start(int

返回指定捕获组的起始索引

group)

int end()

返回最后一个匹配字符之后的偏移量

int end(int

返回子序列组最后一个字符之后的偏移量

captured by the given group during this match

String

返回上一个匹配组匹配的输入子串

String

返回给定组在此次匹配中捕获的输入子序列

上一个匹配操作

group)

int

返回此匹配结果中捕获组的数量

pattern

让我们通过一个例子来更好地理解这个接口。

假设，输入字符串是来自 HTTP 响应头的 Web 服务器响应行：HTTP/1.1 302 Found

我们解析此行的正则表达式模式如下：

HTTP/1\.[01] (\d+) [a-zA-Z]+

Note that there is only one captured group that captures integer status code.

让我们看看这段代码列表，以更好地理解 MatchResult 接口的各种方法：

包含 example.regex.*

导入 java.util.regex.*

public class MatchResultExample

{

public static void main(String[] args)

{

final String re = "HTTP/1\\.[01] (\\d+) [a-zA-Z]+"; final String str = "HTTP/1.1 302 Found";

final Pattern p = Pattern.compile(re);

Matcher m = p.matcher(str);

if (m.matches())

{

MatchResult mr = m.toMatchResult();

// 打印捕获组的数量

System.out.println("groupCount(): " + mr.groupCount());

// 打印完整的匹配文本

System.out.println("group(): " + mr.group());

// 打印匹配文本的起始位置

System.out.println("start(): " + mr.start());

// 打印匹配文本的结束位置

System.out.println("end(): " + mr.end());

// 打印第一个捕获组

System.out.println("group(1): " + mr.group(1));

// 打印第一个捕获组的起始位置

System.out.println("start(1): " + mr.start(1));

// 打印第一个捕获组的结束位置

System.out.println("end(1): " + mr.end(1));

}

}

}

在调用所需的 Pattern 和 Matcher 方法（下一节讨论）之后，我们将检索一个 MatchResult 实例。编译并运行前面的代码后，我们将得到以下输出，它显示了该接口各种方法的使用：

groupCount(): 1

group(): HTTP/1.1 302 Found

start(): 0

end(): 18

group(1): 302

start(1): 9

end(1): 12

**Pattern 类**

Pattern 类表示字符串正则表达式的编译形式。到目前为止，我们已将所有正则表达式作为字符串提供。每个字符串正则表达式都必须在 Java 正则表达式引擎执行之前编译成一个 Pattern 类的实例。Pattern 类的实例用于创建一个 Matcher 对象，以匹配输入文本与正则表达式。

让我们先列出 Pattern 类中重要且有用的方法：方法签名

Description

Static Pattern compile(String

将给定的字符串正则表达式编译成一个

regex)

将给定的 String 正则表达式编译成具有给定标志的 Pattern 实例。标志可以是其中一个或多个

静态 Pattern compile(String

CASE_SENSITIVE, UNICODE_CHARACTER_CLASS 和其他几个。检查正则表达式，int flags)

Java Pattern API 在

http://download.java.net/java/jdk9/docs/api/java/util/regex/Pattern.html 查看所有标志及其描述。

Matcher matcher(CharSequence

创建一个匹配器实例以匹配给定输入与此输入)

编译后的模式。

返回指定字符串的文本模式。在引用字符串后，输入字符串中的正则表达式元字符或转义序列将只是字面量，没有任何特殊含义。

String quote(String str)

给定字符串在 \\Q 和 \\E 转义构造中。这些特殊的转义构造用于将包装的字符串作为字面字符串，从而移除所有正则表达式元字符和特殊字符的特殊含义。

asPredicate

Predicate<String>

()

创建一个字符串的谓词以匹配输入字符串。

Predicate<String> asPredicate ()

Stream<String>

使用此模式分割给定输入字符串并创建一个流 splitAsStream(CharSequence

从给定输入序列中根据此模式的匹配项

input)

（添加于 Java 8）。

在给定输入序列中根据此模式的匹配项进行分割。它是 String[] split(CharSequence input)

与 String.split(String regex) 相同。

在给定输入序列中根据此模式的匹配项进行分割。String[] split(CharSequence input,

限制参数控制模式应用的次数，以及 int limit)

因此，它影响结果数组的长度。它与 String.split(String regex, int limit) 相同。

Pattern 类有一个静态方法，可以用来匹配字符串与正则表达式。如下所示：

boolean matches(String regex, CharSequence input)

它可以用作以下内容的替代：

final Pattern p = Pattern.compile(regex);

Matcher m = p.matcher(input);

m.matches();

这实际上是 JDK9 中此方法的实现。虽然调用此方法比三行代码更简单、更短，但如果多次执行相同的正则表达式匹配，建议单独使用 compile()、matcher() 和 matches() 方法。在这种情况下，我们只需在第一次调用 compile() 时执行，并保留编译后的模式，以避免每次执行匹配时都重新编译。

![Image 16](img/index-116_1.jpg)

**使用 Pattern 的示例**

**类**

让我们看看几个示例，以了解这些方法的一些用法。

要编译用于十进制数字的正则表达式，我们可以使用以下代码片段：

final String decimalPattern = "^[+-]?\\d*\\.?\\d+$"; Final Pattern pattern = Pattern.compile(decimalPattern);

静态方法 Pattern.compile 编译一个字符串正则表达式并返回一个 Pattern 实例。

要匹配可能包含换行符的文本之间的文本，例如在 ## 和 ## 之间，我们可以使用以下编译后的模式：

final String re = "##.*?##";

最终 Pattern pattern = Pattern.compile(re, Pattern.DOTALL); 这里，我们使用两个参数：Pattern.compile 方法，并在第二个参数中传递 DOTALL 作为标志，因为我们想匹配换行符以及使用我们的懒惰模式 .*?。

*注意使用懒惰模式 .*? 而不是贪婪模式 .*，这样我们就能匹配* *##和##之间的最短匹配。*

我们也可以使用内联模式修饰符(?s)来编写前面的代码片段，final String re = "(?s)##.*?##";

Final Pattern pattern = Pattern.compile(re);

如果我们想要匹配一个包含子序列的字符串，+-*/.，两边各有一个或多个空白字符，则可以使用以下代码：package example.regex;

import java.util.*;

import java.util.regex.*;

class PatternQuoteExample

{

public static void main (String[] args)

{

String input = "Math operators: +-*/. ";

boolean result;

String quoted = Pattern.quote("+-*/.");

System.out.println(quoted);

// 使用标准转义的正则表达式

result = input.matches(".*\\s+\\+-\\*/\\.\\s+.*"); System.out.println(result);

// 使用 Pattern.quote 围绕我们的搜索字符串的正则表达式

result = input.matches(".*\\s+" + quoted + "\\s+.*");

System.out.println(result);

// 使用\Q 和\E 围绕我们的搜索字符串的正则表达式

result = input.matches(".*\\s+\\Q+-*/.\\E\\s+.*"); System.out.println(result);

}

}

在编译并运行此代码后，将引号字符串作为："\Q+-*/.\E"打印出来，并且对于所有三个情况都打印 true，因为调用 matches 在所有时候都成功。然而，一个重要的区别是第二个情况中使用了 Pattern.quote，它处理了搜索字符串中特殊正则表达式字符的引号，例如+，*，.

然后，在第三种情况下，我们只需使用\\Q 和\\E 来包装我们的搜索字符串，这与调用 Pattern.quote 并传递我们的搜索字符串相同。

要在两个管道符或||之间分割输入文本，我们可以使用以下代码：package example.regex;

import java.util.*;

import java.util.regex.*;

class PatternSplitExample

{

public static void main (String[] args)

{

final String input = "value1||value2||value3";

final Pattern p = Pattern.compile(Pattern.quote("||"));

// 调用 split 并打印生成的数组中的每个元素

// 使用流 API

Arrays.stream(p.split(input))

.forEach(System.out::println);

}

}

关于此代码的以下几点需要注意：

我们调用 Pattern.quote 来避免转义双管道字符串 我们调用 Pattern.compile 来编译我们的字符串正则表达式并返回一个编译后的 Pattern 对象

我们使用生成的模式实例通过提供一个我们想要操作的输入字符串来调用 split 方法

Java 8 添加了一个新方法，splitAsStream，它返回一个包含给定输入序列中匹配此模式的子字符串的流。使用 splitAsStream，我们可以将前面的类简化如下：package example.regex;

import java.util.*;

import java.util.regex.*;

class PatternSplitStreamExample

{

public static void main (String[] args) throws java.lang.Exception

{

final String input = "value1||value2||value3";

final Pattern p = Pattern.compile(Pattern.quote("||"));

// 调用 splitAsStream 并从生成的流中打印每个元素 p.splitAsStream(input)

.forEach(System.out::println);

}

}

注意使用 splitAsStream 方法而不是 Arrays.stream()静态方法

在这个类中。创建一个数组执行整个分割。当 Pattern 返回一个流时，它只有在需要时才会进行分割。如果我们，例如，限制流只处理前 10 个元素，那么就不需要对后续元素进行分割。

即使某些实现只是进行分割并从 splitAsStream()返回一个基于流的数组，这也是正确的。JDK 的不同实现如果我们在使用 splitAsStream()时使用更好的解决方案是自由的，但如果我们在使用 split()之后将其转换为流，就没有选择。

**过滤令牌列表**

**使用 asPredicate()**

**方法**

如前表所示，asPredicate()方法创建了一个可以用于匹配输入字符串的谓词。让我们通过一个示例代码列表来更好地理解这个方法：

package example.regex;

import java.util.List;

import java.util.stream.*;

import java.util.regex.*;

public class AsPredicateExample

{

public static void main(String[] args)

{

final String[] monthsArr =

{"10", "0", "05", "09", "12", "15", "00", "-1", "100"}; final Pattern validMonthPattern =

Pattern.compile("^(?:0?[1-9]|1[00-2])$");

List<String> filteredMonths = Stream.of(monthsArr)

.filter(validMonthPattern.asPredicate())

.collect(Collectors.toList());

System.out.println(filteredMonths);

}

}

这段代码有一个月份数字列表，作为 String 数组。有效的月份在 1 到 12 之间，单位数月份前可以有可选的 0。

我们使用以下正则表达式模式来验证月份数字：

^(?:0?[1-9]|1[00-2])$

我们使用 asPredicate()方法的返回值来过滤包含所有输入月份值的字符串数组流。

编译并运行后，前面的代码将打印以下输出，这是一个从原始列表中过滤出的包含所有有效月份数字的列表：

[10, 05, 09, 12]

**Matcher 类**

Matcher 类的实例通过解释由 Pattern 实例表示的编译后的正则表达式，在字符序列上执行各种匹配操作。这就是我们如何使用这个类来匹配正则表达式的方式：

我们通过调用需要输入序列作为参数的模式匹配器方法来从模式创建匹配器实例

匹配器实例用于使用这三种方法执行三种类型的匹配操作，每种方法都返回一个布尔值（true 表示成功）：matches

find

lookingAt

这些方法以以下方式执行匹配：matches 方法尝试使用匹配器的模式匹配整个输入序列

查找方法 *搜索* 输入序列以找到下一个与模式匹配的子串

lookingAt 方法尝试使用匹配器的模式在起始位置匹配输入序列。

让我们在这里列出 Matcher 类中所有重要的方法：方法签名

描述

使用匹配器的模式尝试查找

boolean find()

输入文本的下一个匹配子串。

这与之前相同，只是

搜索从起始位置开始。

boolean matches()

尝试匹配完整的输入文本...

尝试从

boolean lookingAt()

该区域的开始。它不需要

匹配完整的输入文本。

返回与输入文本完全匹配的

String group()

上一个匹配项。

String group(int group)

返回由

上次指定的组号

匹配操作。

返回由给定

String group(String groupName)

上场比赛中的命名组

操作。

返回此正则表达式中捕获组的数量

int groupCount()

匹配器的模式。

返回上一个匹配项的起始索引

int start()

操作。

返回捕获文本的起始位置

int start(int group)

由给定的组号在期间

上次匹配操作。

返回捕获文本的起始位置

int start(int groupName)

由之前指定的命名组

匹配操作。

返回上一个匹配的结束位置

`int end()`

操作。

返回捕获文本的结束位置

int end(int group)

由给定的组号在期间

上次匹配操作。

返回捕获文本的结束位置

int end(int groupName)

由之前指定的命名组

匹配操作。

匹配器。

将给定的替换文本附加到

appendReplacement(StringBuffer)

字符串缓冲区在最后一个字符之后

缓冲区，字符串替换)

字符串缓冲区中的上一个匹配项。

此方法从输入读取字符

文本，从附加位置开始，并且

将它们附加到给定的字符串缓冲区。它是

StringBuffer

旨在在调用一个或多个之后执行

在调用 appendReplacement 方法时

将输入文本的剩余部分复制到缓冲区。

返回一个字面替换的字符串用于

静态字符串

指定的字符串。它使反斜杠和

美元符号需要被当作字面意思处理。

使用当前匹配器的模式，它会替换

字符串替换所有匹配项（String replaceAll(String

输入文本的所有匹配子串

替换)

使用给定的替换字符串。

使用当前匹配器的模式，它会替换

字符串替换第一个（String replaceFirst(String

输入文本的第一个匹配子串

替换（replacement）

使用给定的替换字符串。

重置此匹配器对象并初始化所有

Matcher reset()

内部状态。

匹配器重置(CharSequence

使用新的输入重置此匹配器对象

输入)

文本并初始化所有内部状态。

返回匹配器的匹配结果

表示匹配的状态。此方法通常

MatchResult toMatchResult()

通常在调用

find/matches/lookingAt 方法调用。

Matcher usePattern(Pattern

更新此匹配器使用的模式

newPattern)

查找新的匹配项。

**使用 Matcher 的示例**

**类**

Matcher 类代表 Java 中主要的正则表达式引擎，它提供了匹配正则表达式对输入所需的所有功能和特性。

让我们通过实际示例来查看这个类的一些重要方法，以了解它们的使用。

**lookingAt()方法**

lookingAt()方法尝试从输入的*开始*位置匹配输入，但不需要整个区域与模式匹配。以下代码演示了这一点：

package example.regex;

import java.util.regex.*;

class MatcherLookingatExample

{

public static void main (String[] args)

{

final Pattern pattern1 = Pattern.compile("master[a-z]*"); final Pattern pattern2 = Pattern.compile("master"); final Pattern pattern3 = Pattern.compile("regular"); String input = "mastering regular expressions"; Matcher matcher = pattern1.matcher(input);

System.out.printf("[%s] => [%s]: %s%n", input, matcher.pattern(), matcher.lookingAt());

// 更新匹配器的模式为新模式

matcher.usePattern(pattern2);

System.out.printf("[%s] => [%s]: %s%n", input, matcher.pattern(), matcher.lookingAt());

// 更新匹配器的模式为新模式

matcher.usePattern(pattern3);

System.out.printf("[%s] => [%s]: %s%n", input, matcher.pattern(), matcher.lookingAt());

}

}

编译并运行上述代码后，将产生以下输出：

[掌握正则表达式] => [master[a-z]*]: true

[掌握正则表达式] => [master]: true

[掌握正则表达式] => [regular]: false

您可以看到，当我们提供位于输入开头的模式时，lookingAt()方法返回 true，例如 master[a-z]*和 master，但当我们提供位于中间的模式时，例如 regular，它返回 false。

**matches()方法**

matches()方法尝试将整个区域与模式进行匹配，并且只有当整个区域与模式匹配时才返回 true。

让我们看看以下代码，以更好地理解这个方法的使用：package example.regex;

导入 java.util.regex.*;

class MatcherMatchesExample

{

public static void main (String[] args)

{

final Pattern pattern1 = Pattern.compile("mastering"); final Pattern pattern2 = Pattern.compile("mastering.*"); final Pattern pattern3 = Pattern.compile("regular.*"); String input = "mastering regular expressions"; Matcher matcher = pattern1.matcher(input);

System.out.printf("[%s] => [%s]: %s%n", input, matcher.pattern(), matcher.matches());

// 更新匹配器的 ppattern 为新模式

matcher.usePattern(pattern2);

System.out.printf("[%s] => [%s]: %s%n", input, matcher.pattern(), matcher.matches());

// 更新匹配器的模式为新模式

matcher.usePattern(pattern3);

System.out.printf("[%s] => [%s]: %s%n", input, matcher.pattern(), matcher.matches());

}

}

运行后，这将给出以下输出：

[掌握正则表达式] => [mastering]: false

[掌握正则表达式] => [mastering.*]: true

[掌握正则表达式] => [regular.*]: false

如您所见，我们只有在我们的模式与从开始到结束的整个区域匹配时才返回 true，这是使用此正则表达式：mastering.*

**find() 和 find(int start)** 

**methods**

这些查找方法试图找到输入序列中与模式匹配的下一个子序列。这些方法仅在输入序列的子序列与该匹配器的模式匹配时返回 true。如果文本中可以找到多个匹配项，则 find() 方法将找到第一个，然后对于 find() 的每次后续调用，它将移动到下一个匹配项。

以下示例代码将使其更清晰：

package example.regex;

import java.util.regex.*;

class MatcherFindExample

{

public static void main (String[] args)

{

final String input = "some text <value1> anything <value2><value3> here";

/* 第一部分 */

final Pattern pattern = Pattern.compile("<([^<>]*)>"); Matcher matcher = pattern.matcher(input);

while (matcher.find()) {

System.out.printf("[%d] => [%s]%n",

matcher.groupCount(), matcher.group(1));

}

/* 第二部分 */

// 现在使用类似的模式，但使用命名组和重置

// 匹配器

matcher.usePattern(Pattern.compile("<(?<name>[^<>]*)>")); matcher.reset();

while (matcher.find()) {

System.out.printf("[%d] => [%s]%n",

matcher.groupCount(), matcher.group("name"));

}

}

}

这将输出以下内容：

[1] => [value1]

[1] => [value2]

[1] => [value3]

[1] => [value1]

[1] => [value2]

[1] => [value3]

如前述代码所示，我们使用否定字符类 [^<>]* 在捕获组内提取所有位于尖括号内的文本。

在代码的第一部分，我们使用正则捕获组和 matcher.group(1) 来提取并打印第 1 组捕获的子序列。组的编号每次执行 find() 时都会开始，之前的捕获将被清除。即使它在循环中，示例中始终是 group(1)，因为在每次迭代中可能有多个组。

在第二部分，我们使用命名捕获组和重载的方法调用 matcher.group("name") 来提取给定组名捕获的子序列。

![图像 17](img/index-128_1.jpg)

**The**

**appendReplacement(StringBuffer**

**sb, String replacement)**

**method**

此方法旨在与 appendTail 和 find 方法一起在循环中使用。一旦我们使用 find() 方法找到一个匹配项，我们就可以调用 appendReplacement() 方法来对每个匹配项进行操作并替换匹配的文本。

最后，它将替换后的文本追加到 StringBuffer。它从输入序列的追加位置开始读取字符，并将它们追加到给定的字符串缓冲区。

It stops after reading the last character preceding the previous match, that is, the character at index start() - 1\.

The replacement string may contain references to subsequences captured during the previous match. All the rules of replacement reference we String.replaceAll apply to this method also.

The appendReplacement() method keeps track of what has been copied into StringBuffer, so we can keep searching for matches using find() in a loop, until no more matches are found in the input text. There will be an example following the next section.

*Java 9 为这个方法提供了一个重载签名，开始* *接受 StringBuilder 而不是 StringBuffer 作为第一个参数。*

![Image 18](img/index-129_1.jpg)

**The**

**appendTail(StringBuffer sb)**

**方法**

此方法从输入序列中读取字符，从追加位置开始，并将它们追加到给定的字符串缓冲区。它旨在在调用 appendReplacement 方法一次或多次之后调用，以便复制输入序列的其余部分。

*就像在 appendReplacement()方法的情况下一样，appendTail()方法也有一个重载版本，它接受* *StringBuilder，它不是同步的，而不是 StringBuffer，它是同步的。*

**示例**

**appendReplacement 和**

**appendTail 方法**

让我们看看一个完整的程序来理解这些方法的使用。

考虑以下输入：

<n1=v1 n2=v2 n3=v3> n1=v1 n2=v2 abc=123 <v=pq id=abc> v=pq 我们需要编写代码来交换所有在尖括号<和>内包含的名称-值对，同时保持尖括号外的名称-值对不变。

运行我们的代码后，应该产生以下输出：

<v1=n1 v2=n2 v3=n3> n1=v1 n2=v2 abc=123 <pq=v abc=id> v=pq 要解决这个问题，我们首先必须使用 find 方法在循环中找到每个在尖括号内封闭的匹配项。在循环内部，我们将使用 appendReplacement 方法替换每个名称-值对。最后，在循环外部，我们将使用 appendTail 方法追加最后匹配后的剩余字符。

这里是完整的代码：

package example.regex;

import java.util.regex.*;

class MatcherAppendExample

{

public static void main (String[] args)

{

final String input = "<n1=v1 n2=v2 n3=v3> n1=v1 n2=v2 abc=

123 <v=pq id=abc> v=pq";

// 模式 1 用于查找所有在<和>之间的匹配项

final Pattern pattern = Pattern.compile("<[^>]+>");

// 模式 1 用于查找每个 name=value 对

final Pattern pairPattern = Pattern.compile("(\\w+)=(\\w+)"); Matcher enclosedPairs = pattern.matcher(input);

StringBuilder sbuf = new StringBuilder();

// 在循环中调用 find 并针对每个匹配项调用 appendReplacement

{

Matcher pairMatcher = pairPattern.matcher(enclosedPairs.group());

// 在每个匹配中将 name=value 替换为 value=name

enclosedPairs.appendReplacement(sbuf,

pairMatcher.replaceAll("$2=$1"));

}

// appendTail 将剩余字符追加到缓冲区

enclosedPairs.appendTail(sbuf);

System.out.println(sbuf);

}

}

编译和运行前面的代码将产生以下输出：

<v1=n1 v2=n2 v3=n3> n1=v1 n2=v2 abc=123 <pq=v abc=id> v=pq 如你所见，最终输出在尖括号内所有名称=值对都已交换。

**总结**

在本章中，你学习了最重要的正则表达式处理 Java 类。Matcher 和 Pattern 类是重量级且复杂的工具，可以用于在字符串操作中达到非常高的水平。我们已经看到了一个复合任务的例子，即在字符串内部、由尖括号包围的名称-值对转换。如果你想象一下没有正则表达式和这些类会多么困难，那么你就会意识到在阅读本章后你手中的力量；这几乎是巫师级别的。

在下一章中，我们将继续学习正则表达式的进阶主题，例如零宽断言、使用前瞻和后顾断言、原子组等。

**探索零宽**

**断言、环视**

**以及原子组**

你将学习 Java 正则表达式中的零宽断言。我们将涵盖各种零宽断言及其使用模式。然后我们将继续学习 Java 正则表达式中的重要主题，如前瞻和后顾断言，以及如何使用它们来解决一些重要问题。我们还将讨论在 Java 正则表达式中使用原子组。

本章我们将涵盖以下主题：

零宽断言

之前的匹配边界

原子组

前瞻断言：正断言和负断言

后顾断言：正断言和负断言

从重叠匹配中捕获文本

在前瞻和后顾组内的捕获组

Java 正则表达式中后顾断言的限制

![图像 19](img/index-134_1.jpg)

**零宽断言**

零宽或零长度断言在正则表达式中意味着存在一个零长度的匹配，它不会改变输入字符串中指针的当前位置。这些断言不会消耗字符串中的字符，但只会断言是否可能匹配，给出二进制的真或假匹配结果。

虽然许多零宽断言像组一样用括号表示，但我们将很快看到它们不会捕获任何文本。零宽断言在回溯或替换中没有任何实际意义。

*我们在前面的章节中已经讨论了一些零宽断言，例如锚点和边界断言。*

Java 正则表达式引擎允许许多预定义的零宽断言，包括我们之前讨论过的，如起始、结束锚点和单词边界。

**预定义零宽**

**断言**

零宽

描述

断言

\b

断言单词边界

\B

除了在单词边界处之外任何地方都可以断言

^

仅在行的开始位置断言位置

$

仅在行的末尾断言位置

\A

仅在字符串的开始位置断言位置

\z

仅在字符串的末尾断言位置

仅在字符串的末尾或行中断言位置

\Z

在字符串的末尾（如果存在）

在之前匹配的末尾或开始位置断言位置

\G

用于第一个匹配的字符串

**正则表达式定义的零宽断言**

**断言**

正则表达式定义的零宽断言使用 ( 和 )，并且在开括号后面有一个 ?。

有两种断言：正向，用等号 "=" 表示，和负向，用感叹号 "!" 表示。如果断言是向后看的，那么问号 "?" 后面跟着一个小于号 "<"。因此，(?=...) 是正向前瞻断言，(?<!...) 是负向后顾断言。

正向前瞻断言确保字符串与当前位置之后的模式匹配。例如，abc(?=K) 确保了 (?=模式)

检查字符串中的字符 "abc" 后面是字母

"K"，但这个检查不会消耗字符 "K"。

负向前瞻断言确保字符串在当前位置之后不匹配模式。例如，abc(?!Z) (?!模式)

确保检查字符串中的字符 "abc" 后面不是字母 "Z"，但这种检查不会消耗字符 "Z"。

正向后顾断言确保字符串与当前位置之前的模式匹配

模式在当前位置之前。例如，(?<=P)abc 确保

<=模式)

检查字符串中的 "abc" 前面是字母 "P"，但这个检查不会消耗字符 "P"。

负向后顾断言确保字符串在当前位置之前不匹配模式。例如，(?<!Q)abc (?

确保字符 ""

<!模式)

检查字符串中的 "abc" 不是

在字母 "Q" 前面，但这个检查不会消耗字符 "Q"。

在下一节中，我们将深入了解 \G 边界断言的更多细节，然后，你将了解前瞻和后顾断言。然而，在讨论前瞻和后顾断言之前，我们将讨论原子组，这是一个重要的结构和主题，有助于理解前瞻和后顾断言的行为。

**\G 边界断言**

\G 是零宽断言。它也是一个边界匹配器，断言在之前匹配的末尾或字符串的开始位置，例如，对于第一个匹配的 \A 断言。Java 正则表达式引擎在 Matcher 实例的上下文中记住 \G 的位置。如果 Matcher 再次实例化或重置，那么 \G 的位置也会初始化为字符串的开始。

例如，考虑以下输入：

,,,,,123,45,67

考虑到我们需要将仅在输入起始处出现的每个逗号替换为连字符，以便我们有与输入字符串中逗号数量相同的连字符数量。我们的最终输出应该是以下内容：

-----123,45,67

我们不能仅仅通过匹配每个逗号来进行 replaceAll，因为这也会替换 123 和 45 之后的逗号，而且我们还想有与输入字符串中逗号数量相同的连字符数量。

对于此类情况，我们可以使用\G 断言并使用以下 Java 代码片段：input = input.replaceAll("\\G,", "-"); 由于\G 第一次匹配行首，它将断言第一个逗号之前的位置。随后，它匹配每个逗号之后的位臵，因为\G 匹配上一个匹配的末尾位置。它将在控制达到数字 1 时停止匹配。这些匹配中的每一个都替换为一个单个连字符，因此给我们的替换字符串中的连字符数量与原始输入中起始逗号的数量相同。

让我们再看另一个完整示例，以更好地理解\G 的使用。

这里是一个示例输入：

{%var1%, %var2%, %var3%} {%var4%, %var5%, %var6%}

我们的任务是将第一 {...} 中的%字符替换为#（井号）字符。

section only. 我们可以假设{和}是完美平衡的。预期的输出如下：

{#var1#, #var2#, #var3#} {%var4%, %var5%, %var6%}

注意输出中只有第一对 {...} 中的%被替换为#。

这里是一个代码列表，用于解决这个问题：

package example.regex;

class GBoundaryMatcher

{

public static void main (String[] args)

{

String input = "{%var1%, %var2%, %var3%} " +

"{%var4%, %var5%, %var6%}";

final String re = "(^[^{]*\\{|\\G(?!^),\\h*)%([^%]+)%";

//现在在 replaceAll 方法中使用上述正则表达式 String repl = input.replaceAll(re, "$1#$2#");

System.out.println(repl);

}

}

这里是如何在这个代码中执行这个正则表达式的。

我们使用此正则表达式来匹配我们的模式：

"(^[^{]*\\{|\\G(?!^),\\h*)%([^%]+)%"

前面的正则表达式有两个组，如下所示：

(^[^{]*\\{|\\G(?!^),\\h*)

这是第一个捕获组。我们在这里使用交替来选择两种可能的模式：

^[^{]*\\{: 这匹配从开始到第一个{的所有文本。

\\G(?!^),\\h*: 这匹配从上一个匹配的末尾开始的文本，后跟一个逗号和零个或多个水平空白。(?!^)是一个负向前瞻，以避免在起始位置匹配\G。这是必需的，因为\G 在第一次使用时也成功地断言了输入字符串的起始位置。

%([^%]+)%模式是我们的第二个捕获组。它是被%字符包围的子字符串。

在替换中，我们使用以下内容：

$1#$2#

这基本上是将第一组捕获的文本放回，并将第二捕获组用#而不是%包裹，以获得所需输出。

一旦我们编译并运行前面的代码，它将显示以下输出：

{#var1#, #var2#, #var3#} {%var4%, %var5%, %var6%}

![Image 20](img/index-139_1.jpg)

**原子组**

原子组是一个非捕获组，当匹配过程在组内第一个模式匹配后退出组时，它会丢弃组内任何标记记住的所有备选位置。因此，它避免了回溯以尝试组中存在的所有备选模式。

这里是语法：

(?>regex)

在这里，正则表达式可能包含备选模式。另一方面，非原子组将允许回溯；它将尝试找到第一个匹配项，然后如果向前匹配失败，它将回溯并尝试在交替中找到下一个匹配项，直到找到整个表达式的匹配项或耗尽所有可能性。

为了更好地理解它，让我们用一个非原子组的正则表达式作为例子：

^foo(d|die|lish)$

这里的输入字符串是 foodie。

它将匹配起始模式 foo，然后是第一个备选模式 d。它此时失败，因为结束锚点$要求我们必须在输入字符串的末尾，但我们还有两个字符 i 和 e 需要匹配。然后，引擎尝试匹配第二个备选模式 die。这个匹配操作成功，因为$锚点断言为真，因为输入在这里结束，并且停止进一步匹配，返回一个成功的匹配。

*即使我们在这里使用非捕获组而不是捕获组* *来使其成为^foo(?:d|die|lish)$，它在匹配时也会产生相同的效果。*

现在，让我们用一个**原子组**的相同正则表达式作为例子：

^foo(?>d|die|lish)$

注意在(之后使用?>使其成为一个原子非捕获组。

让我们看看当我们将前面的正则表达式应用于相同的输入字符串时会发生什么，即 foodie。

它将匹配起始模式 foo，然后是其第一个备选模式 d。它失败是因为锚点断言为假，因为输入没有在 food 结束。然而，由于使用了原子组，正则表达式引擎立即放弃并不会回溯。

由于正则表达式引擎丢弃了原子组内记住的所有备选位置，它不会尝试匹配第二个备选模式 die，这对于非原子组来说将是一个成功的匹配。最后，这个匹配操作失败，没有匹配。

你需要记住一个简单但重要的事实，交替从左到右尝试其备选模式，并且总是尝试使用

![Image 21](img/index-140_1.jpg)

最左侧的备选模式。因此，在列出交替中的所有选项时，将最长的匹配项首先列出，然后使用其他备选模式来放置较短的匹配项是良好的实践。

使用这个原则，我们可以对我们的原子组做一些小的修改以使其工作。

这里是工作的正则表达式：

^foo(?>lish|die|d)$

我们有相同的输入字符串，foodie。

注意，在这个原子组中我们有相同的替代选项，但顺序不同。由于 d 是 die 的前缀，我们将 die 替代选项放在 d 的左侧，以便正则表达式引擎首先尝试匹配 foodie，然后再尝试匹配 food。

这里是运行这些示例的完整代码列表：

package example.regex;

class AtomicGroupExample

{

public static void main (String[] args)

{

final String input = "foodie";

// 使用非原子组的正则表达式

final String nonAtomicRegex = "foo(d|die|lish)";

// 使用原子组的正则表达式

final String atomicRegex = "foo(?>d|die|lish)";

// 使用正确顺序的替代原子组的正则表达式 final String atomicRegexImproved = "foo(?>lish|die|d)";

// 现在将所有 3 个正则表达式对相同的输入进行执行

System.out.printf("%s: %s%n",

nonAtomicRegex, input.matches(nonAtomicRegex));

System.out.printf("%s: %s%n",

atomicRegex, input.matches(atomicRegex));

System.out.printf("%s: %s%n",

atomicRegexImproved , input.matches(atomicRegexImproved));

}

}

编译并运行代码后，将生成以下输出：foo(?:d|die|lish): true

foo(?>d|die|lish): false

foo(?>lish|die|d): true

*由于原子组通过* *退出组内所有替代选项的评估，从而阻止正则表达式引擎回溯，因此* *原子组在评估具有多个交替选项的大规模文本时通常可以提供显著的性能提升*。

**后行断言**

正向和负向先行断言是零宽断言，允许在文本的当前位置之前（或右侧）执行某些基于正则表达式的检查。正则表达式引擎在评估先行断言模式后保持当前位置。我们可以将多个先行断言表达式一个接一个地链接起来，但正则表达式引擎在检查所有先行断言后不会移动控制。先行断言可以帮助解决一些复杂的正则表达式问题，这些问题在没有先行断言支持的情况下无法解决或非常难以解决。Java 正则表达式引擎，像许多其他正则表达式变体一样，允许在先行断言模式中使用可变长度的量词，如 * 和 +。

存在两种类型的先行断言：正向先行断言和负向先行断言。

**正向先行断言**

一个正向先行断言如果先行断言内的模式匹配，则断言为真。

以下是其语法：

(?=...)

例如，\d+(?=##) 断言在匹配一个或多个数字后必须立即跟有字符串 ##。

**负向先行断言**

一个负向先行断言如果先行断言内的模式不匹配，则断言为真。

以下是其语法：

(?!...)

例如，abc(?!xyz) 断言在匹配字符串 abc 后不能立即跟有字符串 xyz。

**后行断言**

正向和负向后行断言是零宽断言，允许在当前位置之前的文本上执行某些基于正则表达式的检查。正则表达式引擎在评估后行模式后保持当前位置。我们可以将多个后行表达式一个接一个地链接起来，但正则表达式引擎在检查所有后行断言后不会移动控制。后行断言还可以帮助解决一些没有后行支持或很难解决的问题。直到 Java 版本 8，Java 正则表达式引擎不允许在后行模式中使用诸如*和+这样的变长量词。

从 Java 9 开始，Java 正则表达式引擎现在允许在先行断言中使用这些量词。

有两种类型的后行断言：正向后行和负向后行。

**正向后行**

正向后行断言如果后行模式匹配，则断言为真。

这里是其语法：

(?<=...)

例如，(?<=##)\d+断言在匹配一个或多个数字之前必须有一个##字符串。

**负向后行**

负向后行断言如果后行模式不匹配，则断言为真。

这里是其语法：

(?<!...)

例如，(?<!xyz)abc 断言在匹配字符串 abc 之前不能有字符串 xyz。

关于先行正则模式的一些重要点：先行模式是原子的。像原子组一样，一旦先行模式匹配，正则表达式引擎立即从该先行模式退出，只返回一个真或假的断言。

先行模式不会从当前位置移动。所有模式都是从当前位置评估的。在先行断言完成后，位置保持不变。

如果一个正则表达式在相邻位置使用多个先行断言，那么这些表达式的顺序并不重要。

先行模式通常用于复杂的输入验证，在指定模式之前或之后分割输入，以及寻找重叠匹配。

让我们通过一些示例来了解先行和后行表达式的用法。

要匹配一个整数，该整数有一个或多个数字且不允许所有为零，我们可以使用以下方法：

^(?!0+$)\d+$

(?!0+$)是一个负先行表达式，如果在我们示例中的当前位置之前有一个或多个零，则断言失败，这是输入字符串的起始位置。

给定一个包含@字符的输入文本，我们需要匹配@，前提是下一个位置是单词字符。我们可以在这里使用正向先行正则表达式，如下所示：

@(?=\w)

在这里，(?=\w)表示一个正向先行断言，当@旁边是单词字符时，断言为真。

要匹配一个不允许字符串 zzz 出现在任何地方的输入字符串，我们可以使用如下负先行断言：

^(?!.*zzz)

为了匹配一个不跟随或不 precede 数字点的点，我们可以使用负向前瞻和负向后瞻条件，如下所示：

(?<!\d)\.(?!\d)

这里，我们使用了两个断言：

(?<!\d) 是一个负向后瞻条件，断言在点之前没有数字。

(?!\d) 是一个负向前瞻条件，断言在点之后没有数字。

这将匹配 ip.address、.net 和 abc 中的点，但不会匹配 25.78、12. 和 .987\. 中的点。

接下来，我们需要匹配一个不包含 @、# 或 % 重复的输入。

characters.

我们将需要使用一个包含捕获组和回溯引用的负向前瞻模式：

^(?!.*([@#%])\1)

(?!.*([@#%])\1) 是一个负向前瞻断言，匹配并捕获第一个捕获组中的给定特殊字符。使用回溯引用 \1，我们检查捕获字符的重复。负向前瞻中的模式 .* 

([@#%])\1 确保在当前位置之前没有重复的 @、# 或 % 字符。

现在，假设我们需要在满足以下条件的长文本中找到一个搜索词：

1\. 搜索词位于起始位置或之前有一个空白。

2\. 搜索词位于末尾或之后有一个空白。

3\. 搜索词可能包含非单词字符。

为了解决这个问题，我们可以使用一个带有正向前瞻和正向后瞻的正则表达式，如下所示：

(?<=^|\h)searchTerm(?=\h|$)

这里，(?<=^|\h) 是一个正向后瞻，断言搜索词位于起始位置或之前有一个水平空白。

(?=\h|$) 是一个正向前瞻，断言搜索词位于末尾位置或之后有一个水平空白。

匹配包含一个或多个单词字符的字符串，但不允许任何字符重复。

为了解决这个问题，我们需要使用一个捕获组、回溯引用和负向前瞻，如下所示：

^(?:(\w)(?!.*\1))+$

在这里，我们正在匹配并捕获第一个捕获组中的每个字符。每个单词字符通过一个负向前瞻（?!.*\1）来断言，其中\1 是第一个捕获组的回溯引用。负向前瞻（?!.*\1）断言在字符串前方没有捕获字符的另一个出现。最后，我们将整个表达式包裹在一个非捕获组中，以便能够使用量词

+ 匹配一个或多个非重复单词字符。

假设我们需要扫描一行文本，并在从右到左的第三个位置放置一个冒号。然而，我们不应该在起始位置放置冒号。

这应该将 abcd 转换为 a:bcd，而 123456 将转换为 123:456，但 abc 必须不会变成 :abc。

为了解决这个问题，我们可以使用一个前瞻正则表达式，如下面的代码所示：package example.regex;

import java.util.regex.*;

class LookAroundExample1

{

public static void main (String[] args)

{

final String[] inputs =

{"abcd98732", "pqrn", "qwerty12345678xyz", "123"}; `final Pattern p = Pattern.compile("(?!^)(?=(.{3})+$)");` for (`String s: inputs`)

{

`Matcher m = p.matcher(s);`

`System.out.printf("%s => %s%n", s, m.replaceAll(":"));`

}

}

}

运行此代码后，我们将得到以下输出：

`abcd98732 => abc:d98:732`

`pqrn => p:qrn`

`qwerty12345678xyz => qw:ert:y12:345:678:xyz`

`123 => 123`

如您所见，此代码在从右到左的每个第三个位置放置冒号。让我们看看正则表达式匹配过程中会发生什么：(?!^) 是一个负前瞻，用于避免在当前位置匹配。

`(?=(.{3})+$)` 是一个正前瞻，它找到所有当前位置之前有一个或多个三个字符集的位置。这将首先匹配第一个冒号必须插入的位置，然后是第二个，依此类推。这最初可能看起来与原子组和前瞻组的非回溯行为相矛盾。但实际上并非如此。它并不矛盾，因为前瞻本身并不回溯。正则表达式匹配是回溯的，并且会反复评估前瞻断言，针对每个和每个字符位置。

现在，假设我们必须将最内层括号外的所有逗号替换为分号，假设所有括号都是平衡的、不嵌套且在输入文本中未转义。

为了解决这个问题，我们可以使用一个负前瞻表达式，如下所示：

`,(?![^()]*\)`

这个模式匹配一个逗号后跟一个负前瞻断言，当我们在零个或多个字符后跟一个右括号()时，该断言为假，且不包含左括号(

`and )`。由于我们知道括号()是平衡的，这个检查确保我们匹配一个位于括号外的逗号。

这里是查看此正则表达式如何工作的完整代码列表：`package example.regex;`

`import java.util.regex.*;`

`class LookAroundExample2`

{

`public static void main (String[] args)`

{

`String input = "var1,var2,var3 (var1,var2,var3) var4,var5,var6 (var4,var5,var6)"; `final Pattern p = Pattern.compile(",(?![^()]*\\))");` Matcher m = p.matcher(input);

`System.out.printf("%s%n", m.replaceAll(";"));`

}

}

当我们运行前面的代码时，它给出以下输出，替换括号外的所有逗号：

var1;var2;var3 (var1,var2,var3) var4;var5;var6 (var4,var5,var6) 接下来，假设我们需要验证一个密码字符串，并满足以下约束条件：

至少一个英文大写字母

至少一个英文小写字母

至少一个数字

至少一个特殊字符（非单词字符）

长度至少为六，最大为十二

任何地方都不允许有空格

这里是解决方案：

要检查六到十二个非空白字符，我们可以使用以下表达式：

`^\S{6,12}$`

对于剩余的条件，我们需要使用多个前瞻表达式，每个条件一个。让我们逐一构建前瞻模式。

为了确保输入中至少有一个大写字母，我们可以使用以下前瞻断言：

(?=.*[A-Z])

这意味着我们必须检查在零个或多个字符之后是否存在大写字母。

同样，为了确保输入中至少有一个小写字母，我们可以使用以下前瞻断言：

(?=.*[a-z])

同样，为了确保输入中至少有一个数字，我们可以使用以下：(?=.*\d)

同样，为了确保输入中至少有一个非单词字符，我们可以使用以下：

(?=.*\W)

如前所述，这些前瞻模式的顺序并不重要，所以我们可以在正则表达式中以任何顺序保持它们。将所有这些放在一起，我们的最终正则表达式将如下所示：

^(?=.*[A-Z])(?=.*[a-z])(?=.*\d)(?=.*\W)\S{6,12}$

下面是使这个正则表达式工作的完整 Java 代码：package example.regex;

import java.util.regex.*;

class LookAroundPasswordValidation

{

public static void main (String[] args)

{

// 使用我们的正则表达式构建 Pattern

final Pattern p = Pattern.compile(

"^(?=.*[A-Z])(?=.*[a-z])(?=.*\\d)(?=.*\\W)\\S{6,12}$" );

// 要测试的输入字符串

String[] inputs = { "abZ#45", "$$$f5P###", "abc123", "xyz-7612",

"AbC@#$qwer", "xYz@#$ 1278" };

for (String s: inputs)

{

Matcher m = p.matcher( s );

System.out.printf( "%s => %s%n", s, m.matches() );

}

}

}

编译并运行此代码后，我们将得到以下输出：abZ#45 => true

$$$f5P### => true

abc123 => false

xyz-7612 => false

AbC@#$qwer => false

xYz@#$1278 => false

这个输出基本上显示了所有通过我们密码规则的所有字符串的 true 值，否则为 false。

**捕获文本来源**

**重叠匹配**

前瞻模式在需要匹配和捕获重叠匹配文本的情况下也非常有用。

让我们考虑以下输入字符串作为示例：

thathathisthathatis

假设我们需要计算字符串"that"在这个输入中包括所有重叠出现的出现次数。

注意，输入字符串中有三个独立的"that"子字符串，但还有两个额外的重叠匹配需要匹配和计数。以下是重叠子字符串"that"的起始-结束位置：

位置 0-3 3-6 10-13 13-16 16-19

使用正则表达式进行简单搜索将给出三个匹配计数，因为我们遗漏了所有重叠的匹配。为了能够匹配重叠的匹配项，我们需要使用前瞻模式，因为前瞻模式是零长度的。

这些模式不会消耗任何字符；它们只是根据前瞻中使用的模式断言所需文本的存在，并且当前位置不会改变。因此，解决方案是使用以下前瞻正则表达式：(?=that)

下面是完整的代码，以查看这个正则表达式在实际操作中的效果：package example.regex;

import java.util.regex.Matcher;

import java.util.regex.Pattern;

class LookaheadOverlapping

{

public static void main (String[] args)

{

final String kw = "that";

final String regex = "(?=" + kw+ ")"; final String string = "thathathisthathathatis"; final Pattern pattern = Pattern.compile(regex);

final Matcher matcher = pattern.matcher(string);

int count = 0; while (matcher.find())

{

System.out.printf("开始：%d\t 结束：%d%n",

matcher.start(), matcher.start() + kw.length() -1);

count++;

}

System.out.printf("匹配计数：%d%n", count);

}

}

一旦我们运行并编译前面的类，我们将得到以下输出：开始：0 结束：3

开始：3 结束：6

开始：10 结束：13

开始：13 结束：16

开始：16 结束：19

匹配计数：5

您可以从这个输出中看到所有重叠匹配的 Start、End 位置，更重要的是，重叠匹配的计数，即 5。

这里是另一个代码示例，它查找所有以 'a' 为中间字母，且在字母 'a' 前后都有相同单词字符的三个字符字符串。例如，bab、zaz、kak、dad、5a5 和 _a_ 应该被匹配：

package example.regex;

import java.util.regex.Matcher;

import java.util.regex.Pattern;

class LookaheadOverlappingMatches

{

public static void main(String[] args)

{

final String regex = "(?=(\\w)a\\1)";

final String string = "5a5akaktjzazbebbobabababsab"; final Matcher matcher = Pattern.compile(regex)

.matcher(string);

int count = 0; while (matcher.find())

{

final int start = matcher.start();

final int end = start + 2;

System.out.printf("开始：%2d\t 结束：%2d %s%n",

start, end, string.substring(start,end+1));

count++;

}

System.out.printf("匹配计数：%d%n", count);

}

}

此代码生成以下输出：

开始：0 结束：2 5a5

开始：4 结束：6 kak

开始：9 结束：11 zaz

开始：17 结束：19 bab

开始：19 结束：21 bab

开始：21 结束：23 bab

匹配计数：6

**注意捕获**

**前瞻或原子组内的组**

**向后原子组**

在前面的示例中，您学习了如何在前瞻或向后查找模式中使用捕获组。然而，您必须记住，前瞻表达式是零宽度的原子组。正则表达式引擎在评估断言为真或假后立即退出这些组。由于这个原因，这些组内没有回溯。

考虑以下三个正则表达式。第一个没有使用任何前瞻或原子组，第二个正则表达式使用前瞻表达式，第三个正则表达式使用原子组。请注意，在每个正则表达式模式中，我们使用捕获组来匹配和捕获外部组内的零个或多个单词字符：

#(?:(\w*))\w*_\1

#(?=(\w*))\w*_\1

#(?>(\w*))\w*_\1

假设我们将前面的三个正则表达式模式应用于以下输入：

#abc_abc

第一个正则表达式，#(?:(\w+)).*_\1，将找到以组 1 为 "abc" 的成功匹配。

接下来，它匹配 _ 和反向引用 \1 以完成匹配。由于捕获组 (\w*) 最初匹配整个输入 "abc_abc"，正则表达式引擎多次回溯以使这次匹配成功。

第二个正则表达式将无法匹配，因为 lookahead 中的 (\w+) 将匹配并捕获 "abc_abc"，当正则表达式引擎退出 lookahead 组时，它无法找到与 .*_\1 匹配的匹配项，因为没有更多的输入，并且引擎不会像第一个正则表达式那样回溯以完成匹配。

由于相同的原因，带有原子组的第三个正则表达式也将失败匹配；正则表达式引擎在匹配原子组内的字符串后不会回溯。

**查找限制在**

**Java 正则表达式**

与许多其他正则表达式引擎一样，Java 正则表达式引擎不允许在查找正则表达式模式中没有明显最大长度匹配的变长文本。这意味着我们无法在查找模式中使用 * 或 + 量词。然而，Java 正则表达式引擎允许在查找正则表达式中进行有限的或有限的重复。这使我们能够在 Java 正则表达式中通过使用限制量词在查找表达式中解决这个问题。

这意味着我们无法使用以下查找正则表达式来检查以扩展名结尾的文件名：

(?<=\w\.\w+)$

然而，我们可以将前面的模式更改为以下有限重复的模式，现在这个模式将允许 Java 正则表达式引擎接受：(?<=\w\.\w{1,99})$

然而，它将点号之后查找中的单词字符数量限制在 1 到 99 之间，而不是像其他情况那样是开放式的零个或多个单词字符。

+ 量词。然而，你应该谨慎使用此类功能，并检查生成的正则表达式的性能。Java 的后视查找实现在前几个版本中也有不少错误。其中一些错误已经得到解决，但使用 Java 中的复杂后视查找正则表达式时，仍然可能会得到意外结果。

然而，Java 9 允许在查找断言中没有明显的最大长度限制的正则表达式模式。这将允许程序员使用没有最大长度匹配的后视正则表达式模式，例如前一个示例中的以下正则表达式：

(?<=\w\.\w+)$

**总结**

在本章中，我们学习了零宽断言及其在解决一些重要匹配问题中的关键作用。我们讨论了边界匹配器 \G 及其在解决一些问题中的有用性。我们发现了原子组的背后思想，并了解了它们如何提高整体正则表达式性能。然后，我们涵盖了所有重要的前瞻和后视模式。我们涵盖了使用查找的一些有趣的匹配、验证和分割问题。

在下一章中，我们将继续学习 Java 正则表达式的先进概念，例如字符类内的联合和交集，以及否定字符类。

**理解联合，**

**交集，和**

**字符的减法**

**类**

一些正则表达式引擎允许组合字符类，或者字符类内部包含其他字符类。Java 正则表达式引擎也支持这些特性中的许多，我们将在本章中讨论这些特性。

本章我们将涵盖以下主题：

字符类的并集

字符类的交集

字符类的减法

使用组合字符类的优点

**字符的并集**

**类**

字符类的并集将匹配由任何组成字符类匹配的字符。本质上，这是并集运算的一般定义。在正则表达式中，可以通过在另一个字符类内写入字符类来创建字符类的并集。

你可能还记得，字符类以[字符开始，以]字符结束

]字符，我们可以在开闭括号之间列出字符和字符范围。

除了这些，我们还可以在括号内使用其他字符集，并且结果集将是所有这些字符类的并集。这样，就没有并集运算符来创建这些字符类的组合；我们只需简单地将它们相互写入即可。

例如，考虑以下组合字符类：

[A-D[PQR]]

这将匹配 A 到 D 范围内的任何字符或单个字符 P、Q 或 R。这个正则表达式也可以写成以下形式：

[A-DPQR]

我们还可以创建超过两个字符类的并集，例如以下正则表达式：

[A-D[P-S][X-Z]]

这将匹配 A 到 D 范围内的任何字符，P 到 S 范围内的任何字符，或者 X 到 Z 范围内的任何字符。这个正则表达式也可以写成以下形式：

[A-DP-SX-Z]

字符类的并集也可以与取反内部字符类一起使用，这正是字符类并集真正发光并给我们带来额外价值的地方。

我们只有在使用取反字符类与各种字符类的并集时，才能看到并集运算的良好应用。

例如，让我们考虑以下关于取反字符类并集的代码示例：

package example.regex;

import java.util.regex.*;

public class UnionExample

{

public static void main(String[] args)

{

final String re = "[#@.[^\\p{Punct}\\s]]";

final String[] arr = new String[] {

"A", "#", "@", "1", "5", " ", "\n", ":", ".", "a", "%", "-", "3"

};

for (String s: arr)

{

System.out.printf("[%s] %s%n", s,

(s.matches(re) ? "matches" : "does not match"));

}

}

}

这个正则表达式有以下取反字符类：

[^\\p{Punct}\\s]

前面的否定字符类允许任何既不是标点符号也不是空格字符的字符。现在，假设我们想要允许一些选定的标点符号字符，@、#和.，或者换句话说，是[@#.]字符类。在这种情况下，并集很有用。我们创建一个复合字符类，它使用这两种情况的并集，如下所示：

[#@.[^\\p{Punct}\\s]]

现在，这个复合字符类将允许[@#.]字符，或者任何既不是标点符号也不是空格字符的字符。

一旦我们编译并运行前面的代码，我们将得到以下输出：

[A] 匹配

[#] 匹配

[@] 匹配

[1] 匹配

[5] 匹配

[ ] 不匹配

[

] 不匹配

[:] 不匹配

[.] 匹配

[a] 匹配

[%] 不匹配

[-] 不匹配

[3] 匹配

你可以看到对于所有不在我们的否定字符类内部或由[@#.]字符类允许的字符集，输出都是"matches"。它返回

对于所有其他情况，都不匹配。

**字符的交集**

**类**

字符类的交集操作产生一个复合类，它包含所有其操作数（内部）类允许的字符，换句话说，匹配属于复合字符类模式中所有字符类的字符。交集操作符如下：

&&

例如，考虑以下使用&&操作符的复合字符类：

[A-Z&&[PQR]]

这匹配任何在 A 到 Z 范围内的字符，并且是单个 P、Q 或 R 字符之一。然而，前面的正则表达式也可以简单地写成以下形式：

[PQR]

以下使用交集的复合字符类匹配数字 5 和 6，因为只有这两个数字属于所有三个字符类：

[1-7&&[3-6]&&[5-8]]

为了看到这个正则表达式的作用，让我们使用以下完整的代码：package example.regex;

import java.util.regex.*;

public class IntersectionExample

{

public static void main(String[] args)

{

final Pattern p = Pattern.compile("[1-7&&[3-6]&&[5-8]]"); for (int i=0; i<10; i++)

{

String s = String.valueOf(i);

Matcher m = p.matcher(s);

System.out.printf("[%s] %s%n", s,

(m.matches() ? "matches" : "does not match"));

}

}

}

当我们编译并运行前面的代码时，我们将看到以下输出：

[0] 不匹配

[1] 不匹配

[2] 不匹配

[3] 不匹配

[4] 不匹配

[5] 匹配

[6] 匹配

[7] 不匹配

[8] 不匹配

[9] 不匹配

正如你所见，它只对数字 5 和 6 显示"匹配"。

让我们再举一个例子，它涉及到匹配一个非空白字符，且不是 Unicode 字母。我们知道我们可以使用以下正则表达式，通过正向先行断言：

(?=\S)\P{L}

我们也可以使用交集操作来编写此示例，如下所示：

[\\S&&[\\P{L}]]

由于这里使用了 && 操作符，它匹配同时满足以下两个属性的字符，\S（非空白字符）和 \P{L}（非字母字符）。

注意，当在交集操作中不使用否定字符类时，内部方括号是可选的。因此，前面的正则表达式也可以写成以下形式：

[\\S&&\\P{L}]

类似地，为了匹配大写希腊字母，我们可以使用以下两个类的交集：

\p{InGreek}: 这匹配希腊字符块中的字符

\p{Lu}: 这匹配一个上标 Unicode 字母

通过将这两个字符类与交集结合，我们可以创建一个单一的复合字符类，如下所示：

[\p{InGreek}&&[\p{Lu}]]

为了测试前面的正则表达式，让我们选择一些希腊字母，并编写一个简单的 Java 代码，如下所示，以测试所选希腊字母与我们的正则表达式是否匹配：package example.regex;

import java.util.regex.*;

public class UppercaseGreekIntersectionExample

{

public static void main(String[] args)

{

final Pattern p = Pattern.compile("[\\p{InGreek}&&[\\p{Lu}]]"); final String[] arr = new String[] {

"Γ", "Δ", "Θ", "Ξ", "Π", "Σ", "Φ", "α", "β", "γ", "δ", "ε", "A", "P", "e", "r"

};

for (String s: arr)

{

Matcher m = p.matcher(s);

System.out.printf("[%s] %s%n", s,

(m.matches() ? "matches" : "does not match"));

}

}

}

当我们运行前面的类时，它将打印以下输出：

[Γ] 匹配

[Δ] 匹配

[Θ] 匹配

[Ξ] 匹配

[Π] 匹配

[Σ] 匹配

[Φ] 匹配

[α] 不匹配

[β] 不匹配

[γ] 不匹配

[δ] 不匹配

[ε] 不匹配

[A] 不匹配

[P] 不匹配

[e] 不匹配

[r] 不匹配

如您所见，"matches" 只在打印大写希腊字母时打印。对于所有其他字母，它打印 "does not match"。

![图像 22](img/index-162_1.jpg)

**字符的减法**

**类**

假设我们必须在复合字符类模式中匹配属于一个类但不属于另一个类的字符。没有单独的运算符用于减法操作。减法操作是通过使用交集运算符 && 和否定内部字符类来执行的。

*如果我们将较大的集合写在 && 操作符之后，正则表达式通常更容易阅读。*

例如，考虑以下复合字符类：

[0-9&&[³-6]]

它将匹配数字，0 到 9，除了数字，3 到 6。这个字符类也可以写成两个字符类的并集：

[[0-2][7-9]]

我们也可以只使用一个简单的字符类，如下所示：

[0-27-9]

为了匹配所有的大写英文字母，我们可以从大写字母中减去五个元音字母，如下面的正则表达式所示：

[A-Z&&[^AEIOU]]

我们也可以颠倒前面正则表达式中使用的两个集合的顺序，并使用以下正则表达式：

[[^AEIOU]&&A-Z]

假设我们想要匹配所有标点符号字符，除了四个基本数学运算符：+、-、*和/。我们可以使用以下复合字符类，通过减法操作：

[\p{Punct}&&[^+*/-]]

这里是一个测试类，用于测试前面的减法字符类：package example.regex;

import java.util.regex.*;

public class SubtractionExample

{

public static void main(String[] args)

{

final Pattern p = Pattern.compile("[\\p{Punct}&&[^+*/-]]"); final String[] arr = new String[] {

"!", "@", "#", "$", "%", "+", "-", "*", "/", "1", "M", "d"

};

for (String s: arr)

{

Matcher m = p.matcher(s);

System.out.printf("[%s] %s%n", s,

(m.matches() ? "matches" : "does not match"));

}

}

}

编译并运行此程序后，程序产生以下输出：

[!] 匹配

[@] 匹配

[#] 匹配

[$] 匹配

[%] 匹配

[+] 不匹配

[-] 不匹配

[*] 不匹配

[/] 不匹配

[1] 不匹配

[M] 不匹配

[d] 不匹配

如此输出所示，它允许所有标点符号字符，除了列出的四个数学运算符。

**为什么你应该使用**

**复合字符类？**

我们应该使用复合字符类，以下是一些原因：从预定义的 Unicode 块中创建新的自定义字符类。例如，为了匹配阿拉伯 Unicode 块中的所有字母，我们可以使用以下：

[\p{InArabic}&&\p{L}]

通过对多个字符类使用交集或减法操作来避免潜在的较慢的先行或后行模式。

为了提高正则表达式的可读性。

**总结**

在本章中，我们讨论了复合和内部字符类。我们了解到如何使用并集、交集和减法操作来组合简单的字符类，从而创建一个完全不同的字符类以满足我们的需求。你学习了复合字符类的一些良好用法模式，用于解决棘手的问题。注意我们如何通过使用字符类的并集和交集来避免更复杂的先行和后行断言。

在下一章中，我们将讨论一些编写不良的正则表达式的陷阱，你将学习避免它们的方法。你还将学习编写复杂正则表达式的一些重要优化技巧和性能改进方法。

**正则表达式陷阱，**

**优化和**

**性能改进**

正则表达式，如果编写不当，可能会表现不佳。它们可能运行缓慢，当它们在某些代码中频繁执行时，可能成为高 CPU 利用率的来源。为了避免这些问题，正则表达式必须精心制作，理解可能的陷阱，并且它们还必须彻底测试。在本章中，我们将涵盖以下主题：编写正则表达式时常见的陷阱和避免它们的方法 如何测试你的正则表达式功能和性能

优化和性能提升技巧

意外回溯和如何避免它

**常见的陷阱和解决方法**

**在编写时避免它们**

**正则表达式**

让我们讨论一下人们在构建正则表达式以解决各种问题时常见的错误。

**不要忘记转义正则表达式**

**正则表达式外的元字符**

**字符类**

你已经了解到，所有特殊元字符，如 *、+、?、.、|、(、)、[、{、^、$ 等，如果意图是匹配它们本身，则需要转义。我经常看到程序员没有转义这些元字符，从而给正则表达式带来了完全不同的含义。我们在第五章中讨论的 Java 正则表达式 API，即 *Java 正则表达式 API - Pattern 和 Matcher 类*，如果正则表达式格式错误且无法编译，将抛出一个非受检异常。

**避免转义每个非**

**单词字符**

一些程序员过度使用转义，认为他们需要转义每个非单词字符，如冒号、破折号、分号、正斜杠和空白字符，这是不正确的。他们最终编写了一个如下所示的正则表达式：

^https?\:\/\/(www\.)?example\.com$

前面的正则表达式模式使用了过多的转义。这个模式仍然有效，但它不是非常易读。冒号和正斜杠在正则表达式中没有特殊含义；因此，最好以以下方式编写这个正则表达式：

^https?://(www\.)?example\.com$

**避免不必要的捕获**

**减少内存的分组**

**消耗**

我们在网上遇到了许多推广不必要的捕获组的正则表达式示例。如果我们没有提取任何子串或没有在回溯中使用组，那么通过使用以下一种或多种方式来避免捕获组会更好：

1. 在某些情况下，我们可以使用字符类。考虑以下捕获组：

(a|e|i|o|u)

因此，我们不必使用前面的正则表达式，可以使用以下正则表达式：

[aeiou]

2. 我们可以在组的开始放置一个 ?: 来使用非捕获组。

考虑以下正则表达式：

(red|blue|white)

与前面的正则表达式不同，我们可以使用以下正则表达式：

(?:red|blue|white)

3. 要编写一个匹配整数或十进制数的正则表达式，没有必要使用以下正则表达式：

^(\d*)(\.?)(\d+)$

我们可以通过删除不必要的分组来重写它，如下所示：

^\d*\.?d+$

4. 有时，一个正则表达式可能包含多个问题，如我们在前一小节中讨论的：

^https?\:\/\/(www\.)?example\.com$

不仅这个正则表达式使用了过多的转义，而且在这个正则表达式中还有一个不必要的捕获组。因此，通过应用这些修复，前面的正则表达式可以更好地写成以下形式：

^https?://(?:www\.)?example\.com$

**然而，不要忘记使用**

**所需的分组**

**交替**

经常看到使用交替的正则表达式模式，在交替表达式中使用锚点或边界匹配器，但没有在组内保护交替表达式。请注意，^、$、\A、\Z、\z 锚点和\b 边界匹配器的优先级高于交替字符|（管道）

因此，考虑以下形式的正则表达式：

^com|org|net$

它也将匹配计算机、组织和互联网，尽管可能原本的意图是只匹配 com、net 和 org。这是因为开始锚点^只应用于 com，而结束锚点$应用于 net，而 org 则完全没有锚定。

为了正确匹配 com、org 和 net，这个正则表达式应该写成以下形式：

^(?:com|org|net)$

**使用预定义字符**

**使用类而不是更长的**

**版本**

我们在[第二章](https://cdp.packtpub.com/java_9_regular_expressions/wp-admin/post.php?post=78&action=edit)中讨论了预定义字符类和 Unicode 字符类，*理解 Java 正则表达式的核心概念*。我们需要充分利用它。因此，使用\d 代替[0-9]或使用\D 代替[⁰-9]，并使用\w 代替

[a-zA-Z_0-9] 或 \W 而不是 [^a-zA-Z_0-9]。

**使用限定符**

**而不是重复**

**字符或模式**

**多次使用**

计算机的 MAC 地址是在制造时分配给网络接口的唯一标识符。MAC 地址长度为 6 字节或 48 位，格式为 nn:nn:nn:nn:nn:nn，其中每个 n 代表一个十六进制数字。

要匹配 MAC 地址，可以编写以下正则表达式：

^[A-F0-9]{2}:[A-F0-9]{2}:[A-F0-9]{2}:[A-F0-9]{2}:[A-F0-9]{2}:[A-F0-9]{2}$

然而，将正则表达式写成以下形式会更简洁、更易于阅读：

^(?:[A-F\d]{2}:){5}[A-F\d]{2}$

注意与之前的正则表达式相比，这个正则表达式模式变得多么简短且易于阅读。

**不要使用未转义的**

**中间的连字符**

**字符类**

我们知道，大多数特殊正则表达式元字符在字符类内被当作普通字符处理，我们不需要在字符类内转义它们。

然而，如果两个字符之间使用未转义的连字符，那么它将使它成为连字符前后的字符之间的范围。

作为说明性的例子，让我们考虑这个字符类表达式来匹配四个基本数学运算符，+、-、*、/：

[*+-/]

按照这种方式编写，这个字符类在加号（+）和斜杠（/）字符之间有一个连字符。

这使得字符类匹配所有介于+（0x2A）和/之间的字符

(0x2F)，根据 ASCII 表。由于这个原因，前面的模式也将匹配逗号(,)，即 0x2C，和点(.)，即 0x2E，字符。

在字符类中，一个未转义的连字符可以安全地用于第一个或最后一个位置，以避免形成一个范围。考虑到这一点，我们可以通过以下任何一种形式来纠正这个字符类：

[-*+/]

[*+/-]

[*+\-/]

**调用错误的**

**matcher.group()没有**

**之前的 matcher.find()调用，**

**matcher.matches()，或**

**matcher.lookingAt()**

这种令人烦恼的错误在许多程序中都会出现。正如标题所说，这些是在程序员在调用 find、matches 或 lookingAt 方法之前调用任何 group 方法的案例。使用 pattern.matcher(String)方法调用创建了一个 matcher，但我们需要调用这三个方法之一来执行匹配操作。

如果我们不调用这三个方法之一（find、matches 或 lookingAt）就调用 matcher.group()，那么代码将抛出 java.lang.IllegalStateException 异常，如下所示：package example.regex;

导入 java.util.regex.*;

public class MissingMethodCall

{

public static void main(String[] args)

{

final Pattern p = Pattern.compile("(\\d*)\\.?(\\d+)"); final String input = "123.75";

Matcher m = p.matcher(input);

System.out.printf("数值 [%s]，小数数值 [%s]%n", m.group(1), m.group(2));

}

}

注意，代码在从模式实例实例化 matcher 对象后立即调用 m.group(1)和 m.group(2)。一旦编译并执行，此代码将抛出一个不希望的 java.lang.IllegalStateException 异常，指示 matcher 实例不在正确的状态以返回分组信息。

为了修复此代码，插入对这三个方法之一（find、matches 或 lookingAt）的调用以执行匹配操作，如下所示：package example.regex;

import java.util.regex.*;

public class RightMethodCall

{

public static void main(String[] args)

{

final Pattern p = Pattern.compile("(\\d*)\\.?(\\d+)"); final String input = "123.75";

Matcher m = p.matcher(input);

if (m.find()) // 或 m.lookingAt() 或 m.matches()

{

System.out.printf("整数部分 [%s]，小数部分 [%s]%n", m.group(1), m.group(2));

}

}

}

现在，此代码将产生正确的输出，如下所示：整数部分 [123]，小数部分 [75]

**不要使用正则**

**用于解析 XML 的正则表达式**

**HTML 数据**

使用正则表达式解析 XML 或 HTML 文本可能是最常犯的错误。尽管正则表达式非常有用，但它们有其局限性，并且这些限制通常在尝试使用它们进行 XML 或 HTML 解析时遇到。HTML 和 XML 本质上不是正则语言。

幸运的是，Java 中还有其他工具可以用于此目的。JDK 包含现成的类来解析这些格式并将它们转换为**文档对象模型**（**DOM**），或者使用 SAX 解析模型即时处理它们。

当有更具体的解析器用于特定任务时，不要使用正则表达式。存在其他现成工具的事实表明，在这种情况下，正则表达式可能不是最好的工具。毕竟，这就是 XML 和 HTML 解析器的程序员开始工作的原因。

**如何测试和基准测试**

**你的正则表达式**

**性能**

有几个免费的在线正则表达式工具可用，它们会告诉您匹配正则表达式模式所需步骤的数量，并提供有价值的调试信息。您还应该编写自己的单元测试用例。以下是一些可以使用的在线工具列表：

使用 Java 9 中可用的 jshell 快速测试您的正则表达式 使用 RegexMatchers，一个具有静态方法的实用工具类，在 JUnit 中测试您的正则表达式；查看[`matchers.jcabi.com/regex-matchers.html`](http://matchers.jcabi.com/regex-matchers.html)

[regex101.com](https://regex101.com/)

[www.regexplanet.com](http://www.regexplanet.com/)

[www.rexegg.com](http://www.rexegg.com/)

[www.debuggex.com](https://www.debuggex.com/)

[regexper.com](http://regexper.com)

[regexbuddy.com](https://www.regexbuddy.com/) (not free) 使用 Java/JUnit 正则表达式测试库，来自[`github.com/nickawatts/regex-tester`](https://github.com/nickawatts/regex-tester)

除了这些工具之外，您还可以使用您最喜欢的 Java IDE 中的 JUnit 编写自己的综合单元测试用例，并检查计时和其他匹配信息。

这里是使用 RegExMatchers 库的 JUnit 代码示例：包 example.regex;

import com.jcabi.matchers.RegexMatchers;

import org.hamcrest.MatcherAssert;

import org.junit.Test;

public class RegexTest

{

@Test

public void matchesDecimalNumberPattern()

{

MatcherAssert.assertThat(

"[+-]?\\d*\\.?\\d+",

RegexMatchers.matchesPattern("-145.78")

);

}

}

鼓励您使用此库来构建自己的测试用例，并确保您的正则表达式通过所有边缘情况。

**灾难性或指数级**

**回溯**

正则表达式引擎可以大致分为两种类型：1\. 非确定性有限自动机（**NFA**）引擎 2\. 确定性有限自动机（**DFA**）引擎。DFA 引擎在查找匹配时不会对每个字符进行多次评估。另一方面，NFA 引擎支持回溯，这意味着输入中的每个字符都可以被正则表达式引擎多次评估。Java 正则表达式引擎是一个 NFA 引擎。

正则表达式引擎在使用贪婪量词或尝试交替时，会一次回溯或退回一个位置，以尝试匹配给定的模式。

类似地，当使用懒惰量词时，正则表达式引擎一次向前移动一个位置来尝试匹配。

正则表达式引擎通常在给定输入中找到正匹配所需的时间比返回非匹配失败所需的时间少。NFA 正则表达式引擎需要在返回失败之前评估所有可能的排列组合。

例如，在使用嵌套重复量词的正则表达式中，正则表达式引擎在匹配长输入文本时会过度回溯。当正则表达式引擎在字符串末尾未能进行负匹配，并且尝试了过多的排列组合后，通常会发生灾难性回溯问题。

例如，检查以下具有嵌套量词的正则表达式：

^(\w+)*$

假设我们测试它对输入文本的测试，该输入文本末尾没有单词字符，例如以下输入字符串：

abcdefghijklmno:

我们知道，由于输入末尾存在一个非单词字符（冒号），匹配将失败。然而，由于存在嵌套复合量词 (\w+)*，正则表达式引擎会过度回溯，并在放弃之前尝试匹配输入多次。

过度回溯也可能由两个或更多互斥的备选方案引起，这些方案可以匹配输入中的相同字符串。例如，有一个正则表达式模式如下，用于匹配 %% 标签之间的文本：

%%(.|\s)+%%

这个正则表达式也可能导致失败案例的灾难性回溯，例如以下缺少闭合标签的输入字符串：

%% something here abcd 123

这里的问题是，在 (.|\s) 中的交替不是互斥的，因为点号也可以匹配由 \s 匹配的相同空白字符，除了换行符。

这里是一个完整的程序列表，演示了正则表达式在每次循环迭代中都会变慢，并最终导致灾难性回溯：

package example.regex;

import java.util.regex.Matcher;

import java.util.regex.Pattern;

public class CatastropicBacktracking

{

public static void main(String[] args)

{

final int MAX = 30;

for (int i = 1; i < MAX; i++)

{

StringBuilder sb1 = new StringBuilder(i);

StringBuilder sb2 = new StringBuilder(i);

for (int j = i; j > 0; j--)

{

sb1.append('a');

sb2.append("a?");

}

sb2.append(sb1);

final Pattern p = Pattern.compile("^" + sb2.toString() + "$"); Matcher m = p.matcher(sb1.toString());

long start = System.nanoTime();

m.matches();

long end = System.nanoTime();

System.out.printf("%s:: ( %sms ) :: Pattern <%s>, Input <%s>%n", i, (end - start)/1_000_000, sb2, sb1);

}

}

}

当你编译并运行前面的程序并查看生成的输出时，你会注意到如下输出：

1:: ( 0ms ) :: Pattern <a?a>, Input <a>

2:: ( 0ms ) :: Pattern <a?a?aa>, Input <aa>

3:: ( 0ms ) :: 模式 <a?a?a?aaa>, 输入 <aaa> 4:: ( 0ms ) :: 模式 <a?a?a?a?aaaa>, 输入 <aaaa> 5:: ( 0ms ) :: 模式 <a?a?a?a?a?aaaaa>, 输入 <aaaaa> 6:: ( 0ms ) :: 模式 <a?a?a?a?a?a?aaaaaa>, 输入 <aaaaaa> 7:: ( 0ms ) :: 模式 <a?a?a?a?a?a?a?aaaaaaa>, 输入 <aaaaaaa> 8:: ( 0ms ) :: 模式 <a?a?a?a?a?a?a?a?aaaaaaaa>, 输入 <aaaaaaaa> 9:: ( 0ms ) :: 模式 <a?a?a?a?a?a?a?a?a?aaaaaaaaa>, 输入 <aaaaaaaaa> 10:: ( 0ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?aaaaaaaaaa>, 输入 <aaaaaaaaaa> 11:: ( 0ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaa>, 输入 <aaaaaaaaaaa> 12:: ( 0ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaaa>, 输入 <aaaaaaaaaaaa> 13:: ( 10ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaaaa>, 输入 <aaaaaaaaaaaaa> 14:: ( 1ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaaaaa>, 输入 <aaaaaaaaaaaaaa> 15:: ( 15ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaaaaaa>, 输入 <aaaaaaaaaaaaaaa> 16:: ( 18ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaaaaaaa>, 输入 <aaaaaaaaaaaaaaaa> 17:: ( 29ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaaaaaaaa>, 输入 <aaaaaaaaaaaaaaaaa> 18:: ( 22ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaaaaaaaaa>, 输入 <aaaaaaaaaaaaaaaaaa> 19:: ( 51ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaaaaaaaaaa>, 输入 <aaaaaaaaaaaaaaaaaaa> 20:: ( 97ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaaaaaaaaaaa>, 输入 <aaaaaaaaaaaaaaaaaaaa> 21:: ( 188ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaaaaaaaaaaaa>, 输入 <aaaaaaaaaaaaaaaaaaaaa> 22:: ( 441ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaaaaaaaaaaaaa>, 输入 <aaaaaaaaaaaaaaaaaaaaaa> 23:: ( 1003ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaaaaaaaaaaaaaa>, 输入 <aaaaaaaaaaaaaaaaaaaaaaa> 24:: ( 1549ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaaaaaaaaaaaaaaa>, 输入 <aaaaaaaaaaaaaaaaaaaaaaaa> 25:: ( 3010ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaaaaaaaaaaaaaaaa>, 输入 <aaaaaaaaaaaaaaaaaaaaaaaaa> 26:: ( 5884ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaaaaaaaaaaaaaaaaa>, 输入 <aaaaaaaaaaaaaaaaaaaaaaaaaa>

27:: ( 12588ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaaaaaaaaaaaaaaaaaa>, 输入 <aaaaaaaaaaaaaaaaaaaaaaaaaaa> 28:: ( 24765ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaaaaaaaaaaaaaaaaaaa>, 输入 <aaaaaaaaaaaaaaaaaaaaaaaaaaaa> 29:: ( 51679ms ) :: 模式 <a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaaaaaaaaaaaaaaaaaaaa>, 输入 <aaaaaaaaaaaaaaaaaaaaaaaaaaaaa> 注意，当计数器 i 的值较高时，执行时间会迅速增长，尤其是在 25 之后。

**如何避免灾难性的**

**回溯**

在处理正则表达式中的灾难性或过度回溯的情况时，以下是一些需要记住的提示：

当你编写正则表达式时，确保它们快速失败，而不会在回溯中浪费很多不必要的步骤。

当使用嵌套重复操作符或量词时，确保只有一个唯一的方式来匹配字符串。

良好而谨慎地使用原子组和占有量词来避免过度回溯。

你应该避免在交替模式中拥有太多非互斥的可选匹配项。

在使用正则表达式中的自由流动模式如.*或.+时，要非常小心。

在可能的情况下，使用否定字符类来减少回溯步骤并提高性能。

避免使用单个正则表达式匹配大量文本。最好使用正则表达式匹配较小的字符串，并通过循环调用 matcher.find()来获取多个匹配项。如果需要，可以使用另一个内部模式来匹配和检查主模式找到的匹配项。

导致灾难性回溯的嵌套量词正则表达式如下：

^(\w+)*$

我们可以使用占有量词来禁止任何回溯，如下所示：

^(\w+)*+$

你会在任何基准测试或正则表达式测试工具中注意到这种改进的正则表达式有巨大的提升，如前所述。

此外，在交替正则表达式示例中，我们发现此正则表达式在失败情况下导致过度回溯：

%%(.|\s)%%

可以转换为以下正则表达式以避免过度回溯：

%%(\S|\s)+%%

更好地避免使用分组并使用字符类，如下所示：

%%[\S\s]+%%

注意使用\S 而不是点来使替代项相互排斥。

以下是一个可能导致过度回溯的正则表达式：

^(?:.*:)+#$

在前面的正则表达式示例中，如果我们使用否定字符类代替.*，则可以避免灾难性回溯：

^(?:[^:]+:)+#$

正则表达式引擎不会过度回溯，因为否定字符类，

[^:]匹配任何除冒号之外的字符，而不是匹配包括冒号在内的所有字符的点。

考虑另一个具有嵌套重复操作符+的正则表达式模式示例：

%(?:[p-s]+|ye|wo)+%

此正则表达式模式尝试匹配满足以下条件的字符串：必须以%开头

%必须后跟一个或多个交替：字母 p、q、r、s 或字符串 ye 或 wo

必须以另一个%结束

现在按照以下方式测试此正则表达式模式：

%yeqpsrwospqr

显然，正则表达式模式不会匹配，因为最后一个%缺失。

然而，请注意，起始的%和所有随后的字母将在最后一个%之前匹配正则表达式模式。因此，正则表达式引擎会在尝试匹配完整输入之前回溯多次，最终放弃。

当你在*regex101*网站的调试器上测试此正则表达式时，它显示以下内容：匹配 1 失败在 748 步

748 可能对于一个小型输入的匹配失败步骤数来说是一个相当大的数字。像这样的正则表达式可能会显著减慢你的应用程序。其中一些甚至可能因为灾难性的回溯行为而使你的代码挂起数小时或数天。

现在，为了防止这种灾难性的回溯行为，让我们考虑我们之前推荐的两个选项：

1. 使用占有量词，如下所示：

%(?:[p-s]+|ye|wo)++%

在对前面的模式进行相同站点的测试时，我们在调试器中得到了以下结果：

匹配 1 失败，在 33 步后

2. 使用原子组，如下所示：

%(?>[p-s]+|ye|wo)+%

在对前面的模式进行相同站点的测试时，我们在调试器中得到了以下结果：

匹配 1 失败，在 36 步后

你可以注意到，通过使用上述任何一种技术，我们使正则表达式引擎更快地失败，并避免不必要的回溯步骤数量。

**优化和**

**性能提升**

**tips**

让我们讨论一些优化技术和性能提升指南。

**使用编译形式**

**正则表达式**

使用 Pattern.compile(String) 方法调用编译你的字符串正则表达式模式，然后调用 Matcher API 的调用，而不是在字符串中调用简写方法，如 matches()、replaceAll 和 replaceFirst，尤其是在这些匹配或替换方法在循环内部重复调用时。对 String.matches() 或任何在 String API 中定义的基于正则表达式的其他方法的重复调用将每次编译字符串正则表达式模式；对于复杂的正则表达式模式，这可能会非常耗时。

**使用否定字符**

**class 代替贪婪**

**和慢 .* 或 .+**

在可能的情况下，使用否定字符类代替可能降低性能的模式（.* 或 .+），如下所示：

param1=[^&]+&param2=[^&]+&param3=[^&]+$

避免使用性能较慢的量词，如下所示：

param1=.+&param2=.+param3=.+$

**避免不必要的分组**

在你的正则表达式中避免不必要的捕获组。如果正则表达式中的分组是必需的，那么使用非捕获组来节省正则表达式模式在执行时的整体内存占用。

**使用惰性量词**

**strategically 代替**

**贪婪量词导致**

**过多的回溯**

假设我们需要匹配一个包含三个完整单词的输入，*start*、*middle* 和 *end*，它们之间由非空白字符分隔。

考虑使用以下模式，并带有惰性量词：

\bstart\b\S+?\bmiddle\b\S+?\bend\b

与使用以下模式相比，如果我们使用前面的模式，我们的匹配将会更快：

\bstart\b\S+\bmiddle\b\S+\bend\b

**利用占有**

**quantifiers to avoid**

**backtracking**

回想一下，我们在前面的章节中讨论了如何使用占有量词来实现快速失败范式。在可能的情况下，充分利用占有量词来告诉正则表达式引擎避免回溯。

假设我们需要编写一个正则表达式来匹配两个标记@START@和@END@之间的文本

@END@。假设分号现在允许在两个标记之间使用。

我们可以用+或贪婪量词编写此正则表达式，如下所示：

@START@[^;]+@END@

然而，在正则表达式中使用++或占有性量词会更好，如下所示：

@START@[^;]++@END@

对于以下字符串等失败匹配，此正则表达式将执行得更快：

@START@ abc 123 foo @XYZ@

**提取重复的公共部分**

**选择分支外的子串**

考虑以下模式：

(playground|player|playing)

不使用前面的模式，最好提取公共子串 play，并将其移动到选择分支的左侧，如下所示：play(ground|er|ing)

**使用原子组避免**

**回溯和快速失败**

回想一下第六章，*探索零宽断言、前瞻和原子组*，原子组是一个非捕获组，在组内模式的第一次匹配后，它会退出组并丢弃组内任何标记记住的所有替代位置。因此，它避免了回溯以尝试组中存在的所有替代方案。

由于原子组的这一特性，在某些场景中使用原子组可以节省许多不必要的回溯步骤，并加快整体正则表达式的执行速度。

因此，使用以下原子组：

\btra(?>ck|ce|ining|de|in|nsit|ns|uma)\b

使用前面的原子组而不是下面的非捕获组会更好：

\btra(?:ck|ce|ining|de|in|nsit|ns|uma)\b

当匹配输入字符串（如*tracker*，它无法匹配）时，行为差异将很明显。

**总结**

在本书的最后一章，我们讨论了人们在编写正则表达式时常见的错误。这些错误通常会在运行时抛出一些异常，或者导致正则表达式意外地失败匹配。

然后，你学会了避免此类错误的各种方法。

我们讨论了正则表达式中的灾难性回溯以及避免过度回溯的技巧。通过最小化回溯步骤，正则表达式可以变得非常高效。

你学会了正则表达式的测试和基准测试技术。最后，我们详细介绍了许多正则表达式优化和性能提升技巧。我们希望这些技巧能帮助你理解正则表达式的构建块，并编写性能更好的正则表达式来解决复杂的解析和匹配问题。

本书是从 AvaxHome 下载的！

访问我的博客获取更多新书：

[www.avxhm.se/blogs/AlenMiler](https://tr.im/avaxhome)

**目录**

前言

16

本书涵盖的内容

17

你需要为本书准备什么

18

适用于本书的读者

19

习惯用法

20

读者反馈

21

客户支持

22

下载示例代码

23

错误

24

盗版

25

问题

26

正则表达式入门

27

正则表达式简介

28

正则表达式的一些历史

29

正则表达式的各种风味

30

需要正则表达式解决的问题类型

31

正则表达式的基本规则

32

标准正则表达式和元字符的构造

33

字符

一些基本的正则表达式示例

36

贪婪匹配

38

贪婪匹配对正则表达式交替的影响 39

摘要

41

理解 Java 正则表达式的核心构造

42

表达式

理解正则表达式的核心构造

43

量词

44

基本量词

45

使用量词的示例

46

使用量词的贪婪匹配与懒惰匹配 47

拥有量词

48

边界构造

50

使用边界构造的示例

51

字符类

52

字符类的示例

53

字符类内的范围

54

字符范围的示例

55

转义特殊正则表达式元字符和字符类内的转义规则 56

字符类

字符类内的转义

57

字符类内转义规则的示例 58

实际匹配可能包含特殊正则表达式的字符串 59

元字符

否定字符类

60

否定字符类的示例

61

预定义的简写字符类

62

POSIX 字符类

63

Java 正则表达式中的 Unicode 支持

64

常用的 Unicode 字符属性

65

否定前一个正则表达式指令

66

Unicode 脚本支持

67

正则表达式中匹配 Unicode 文本的示例

69

表达式

在定义正则表达式时 Java String 中的双重转义

70

表达式

嵌入式正则表达式模式修饰符

71

Java 正则表达式中的嵌入式模式的放置

73

表达式

禁用模式修饰符

74

摘要

75

使用组、捕获和引用

76

捕获组

77

组编号

78

命名组

80

非捕获组

81

非捕获组的优势

82

反向引用

83

命名组的反向引用

85

命名组的替换引用

86

向前引用

87

无效（不存在的）向前或向后引用

89

摘要

90

使用 Java String 和正则表达式进行正则表达式编程

91

Scanner API

Java 正则表达式 API 简介

92

评估

方法 - boolean matches(String regex)

93

matches 方法的示例

94

方法 - String replaceAll(String regex, String replacement) 96

replaceAll 方法的示例

97

方法 - String replaceFirst(String regex, String replacement) 99

replaceFirst 方法的示例

100

方法 - String split 方法

101

限制参数规则

102

split 方法的示例

103

使用限制参数的 split 方法的示例

104

在 Java Scanner API 中使用正则表达式

106

摘要

110

Java 正则表达式 API 简介 - Pattern 和 111

匹配器类

匹配结果接口

112

模式类

114

使用 Pattern 类的示例

116

使用 asPredicate() 方法过滤令牌列表 119

匹配器类

120

使用 Matcher 类的示例

123

方法 Boolean lookingAt()

124

137

125

find() 和 find(int start) 方法

126

appendReplacement(StringBuffer sb, String replacement) 128

方法

appendTail(StringBuffer sb) 方法

129

appendReplacement 和 appendTail 方法的示例

130

摘要

132

探索零宽断言、后视和

133

原子组

135

134

预定义的零宽断言

177

零宽断言

136

\G 边界断言

或 .+

原子组

139

前视断言

141

正向前视

142

你需要这本书什么

143

后视断言

144

正向前视

145

负向后视

146

从重叠匹配中捕获文本

151

使用限制量词而不是重复字符或

153

后视原子组

Java 正则表达式的后视限制

154

摘要

155

字符类的减法

字符类

字符类的并集

157

字符类的交集

159

下载示例代码

162

matches() 方法

164

摘要

165

正则表达式陷阱、优化和性能 166

改进

在编写正则表达式时常见的陷阱和避免方法 167

表达式

不要忘记在字符外部转义正则表达式元字符 168

类

的错误

169

避免不必要的捕获组以减少内存

170

消费

然而，不要忘记在所需组周围使用

171

选择性

使用预定义的字符类而不是较长的版本 172

191

模式多次

不要在字符中间使用未转义的连字符 174

类

在调用 matcher.group() 之前没有先调用

matcher.find(), matcher.matches(), 或 matcher.lookingAt() 不要使用正则表达式解析 XML / HTML 数据

为什么你应该使用复合字符类？

如何测试和基准测试你的正则表达式性能 178

灾难性或指数回溯

179

如何避免灾难性回溯

182

优化和性能提升技巧

185

使用正则表达式的编译形式

186

使用否定字符类而不是贪婪且缓慢的 .* 

187

负向前视

避免不必要的分组

188

有策略地使用懒惰量词而不是贪婪量词 189

导致过度回溯

使用占有量词来避免回溯

190

理解并集、交集和减法 156

从选择性中提取常见的重复子串

使用原子组以避免回溯并快速失败

192

摘要

193

# 文档大纲

+   前言

    +   本书涵盖的内容

    +   避免转义每个非单词字符

    +   本书面向谁

    +   约定

    +   读者反馈

    +   客户支持

        +   正则表达式定义的零宽断言

        +   勘误表

        +   盗版

        +   问题

+   正则表达式入门

    +   正则表达式简介

        +   正则表达式的一些历史

        +   正则表达式的各种风味

        +   需要正则表达式解决的问题类型

        +   正则表达式的基本规则

        +   标准正则表达式和元字符的构造

        +   一些基本的正则表达式示例

        +   贪婪匹配

            +   贪婪匹配对正则表达式交替的影响

    +   摘要

+   理解 Java 正则表达式的核心构造

    +   理解正则表达式的核心构造

    +   量词

        +   基本量词

            +   使用量词的示例

        +   使用量词的贪婪与懒惰（懒）匹配

        +   占有量词

        +   边界构造

            +   使用边界构造的示例

        +   字符类

            +   字符类示例

            +   字符类内部的范围

                +   字符范围示例

        +   转义特殊正则表达式元字符和字符类内部的转义规则

            +   字符类内部的转义

                +   字符类内部转义规则示例

            +   字面匹配可能包含特殊正则表达式元字符的字符串

            +   否定字符类

                +   否定字符类示例

        +   预定义的简写字符类

            +   POSIX 字符类

        +   Java 正则表达式中的 Unicode 支持

            +   常用 Unicode 字符属性

            +   先前正则表达式指令的否定

            +   Unicode 脚本支持

                +   正则表达式中匹配 Unicode 文本的示例

            +   在定义正则表达式时 Java 字符串中的双重转义

            +   内嵌正则表达式模式修饰符

            +   Java 正则表达式中内嵌模式的放置

            +   禁用模式修饰符

    +   摘要

+   处理组、捕获和引用

    +   捕获组

        +   组编号

        +   命名组

    +   非捕获组

        +   非捕获组的优势

    +   向后引用

        +   命名组的向后引用

        +   命名组的替换引用

        +   正向引用

        +   无效（不存在）的向后或向前引用

    +   总结

+   使用 Java String 和 Scanner API 进行正则表达式编程

    +   Java 字符串 API 正则表达式评估简介

        +   方法 - boolean matches(String regex)

            +   matches 方法的示例

        +   方法 - String replaceAll(String regex, String replacement)

            +   replaceAll 方法的示例

        +   方法 - String replaceFirst(String regex, String replacement)

            +   replaceFirst 方法的示例

        +   方法 - String split methods

            +   limit 参数规则

            +   split 方法的示例

                +   使用 limit 参数的 split 方法示例

    +   在 Java Scanner API 中使用正则表达式

    +   总结

+   Java 正则表达式 API 简介 - Pattern 和 Matcher 类

    +   MatchResult 接口

    +   Pattern 类

        +   使用 Pattern 类的示例

            +   使用 asPredicate()方法过滤令牌列表

    +   Matcher 类

        +   使用 Matcher 类的示例

            +   方法 Boolean lookingAt()

            +   matches()方法

            +   find()和 find(int start)方法

        +   appendReplacement(StringBuffer sb, String replacement)方法

        +   appendTail(StringBuffer sb)方法

        +   appendReplacement 和 appendTail 方法的示例

    +   总结

+   探索零宽断言、前瞻和原子组

    +   零宽断言

        +   预定义的零宽断言

        +   正则表达式定义的零宽断言

    +   \G 边界断言

    +   原子组

    +   前瞻断言

        +   正向先行查找

        +   负向前查找

    +   后查找断言

        +   正向后查找

        +   负向后查找

        +   从重叠匹配中捕获文本

        +   在先行或后行原子组内部小心使用捕获组

            +   Java 正则表达式中后查找的限制

    +   摘要

+   理解字符类的并集、交集和差集

    +   字符类的并集

    +   字符类的交集

    +   字符类的差集

    +   为什么你应该使用组合字符类？

    +   摘要

+   正则表达式陷阱、优化和性能改进

    +   在编写正则表达式时常见的陷阱和避免方法

        +   不要忘记在字符类外部转义正则表达式元字符

        +   避免转义每个非单词字符

        +   避免不必要的捕获组以减少内存消耗

        +   然而，别忘了使用所需的组来包围交替

        +   使用预定义的字符类而不是更长的版本

        +   使用限制量词而不是重复字符或模式多次

        +   不要在字符类中间使用未转义的破折号

        +   在没有先调用 matcher.find()、matcher.matches()或 matcher.lookingAt()的情况下调用 matcher.group()的错误

        +   不要使用正则表达式解析 XML / HTML 数据

    +   如何测试和基准测试你的正则表达式性能

    +   灾难性或指数回溯

        +   如何避免灾难性回溯

    +   优化和性能提升技巧

        +   使用正则表达式的编译形式

        +   使用否定字符类而不是贪婪且缓慢的 .* 或 .+

        +   避免不必要的分组

        +   有策略地使用懒惰量词而不是导致过度回溯的贪婪量词

        +   利用占有量词来避免回溯

        +   从交替中提取常见的重复子串

        +   使用原子组来避免回溯并快速失败

    +   摘要
