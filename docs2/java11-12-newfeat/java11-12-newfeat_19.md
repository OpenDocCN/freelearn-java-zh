# 第十五章：原始字符串字面量

你是否曾经因为使用字符串类型在 Java 中存储 SQL 查询而感到痛苦，因为需要多次使用开闭引号、单引号和双引号？更糟糕的是，你可能还使用了换行转义序列和连接运算符。类似的痛苦也适用于使用字符串类型与 HTML、XML 或 JSON 代码。如果你像我一样害怕所有这些组合，那么不必再害怕。原始字符串字面量就在这里，帮助你摆脱痛苦。

使用原始字符串字面量，你可以轻松地处理可读的多行字符串值，而不需要包含特殊的新行指示符。由于原始字符串字面量不解释转义序列，因此将转义序列作为字符串值的一部分包含变得非常简单。相关的类还包括对边距的管理。

在本章中，我们将涵盖以下主题：

+   使用转义序列在字符串值中的优点和困难

+   添加到 `String` 类的新方法

+   边距管理

+   分隔符

+   使用原始字符串与多行文本数据的示例

# 技术要求

本章中的代码将使用原始字符串字面量，这些字面量是为 JDK 12（2019 年 3 月）设计的。您可以克隆包含原始字符串字面量的存储库来实验它。

本章中所有代码都可以在[`github.com/PacktPublishing/Java-11-and-12-New-Features`](https://github.com/PacktPublishing/Java-11-and-12-New-Features)找到。

# 快速示例

你是否迫不及待地想看到原始字符串的实际应用？我也是。在深入探讨导致原始字符串引入的问题之前，让我们先快速看一下示例。

以下代码展示了如何使用原始字符串字面量编写多行字符串值，使用反引号（`` ` ``）作为分隔符，而不使用连接运算符或特殊的新行或制表符指示符：

```java
String html =  
`<HTML>                                      
    <BODY> 
        <H1>Meaning of life</H1>  
    </BODY> 
</HTML> 
`; 
```

# 现有多行字符串值的问题

在大多数编程语言中，包括 Java，创建多行字符串值是常见的。你可能使用它们来创建 HTML 代码、JSON 或 XML 值，或者 SQL 查询。但是，使用转义序列（新行或制表符指示符的转义序列）的这个看似简单的任务会变得复杂。

为了使你的多行字符串值可读，你可能会在单独的行上定义字符串值的一部分，使用连接运算符（`+`）将它们粘合在一起。然而，随着字符串长度的增加，这可能会变得难以编写和理解。

让我们概述创建多行字符串的简单任务，然后使用多种方法将其存储为字符串值。

# 一个简单的任务

假设你必须使用字符串类型定义以下多行值，同时保持其缩进位置：

```java
<HTML> 
   <BODY> 
          <H1>Meaning of life</H1>  
   </BODY> 
</HTML>
```

# 传统字符串字面量的转义序列地狱

要向传统字符串值中添加换行符或制表符，你可以使用转义序列 `\n` 和 `\t`。转义序列是前面带有 `\`（反斜杠）的字母，它们在 Java 中有特殊含义。例如，在 Java 字符串中，`\n` 被用作换行符，而 `\t` 被用作制表符。

转义序列是用于表示不能直接表示的值的字母组合。例如，为了使用 ASCII 中的换行控制字符，Java 使用 `\n`。Java 定义了多个其他转义序列。

以下代码展示了如何使用换行符和制表符转义序列存储多行字符串值。换行符和制表符转义序列将包含在变量 HTML 中的缩进：

```java
String html = "<HTML>\n\t<BODY>\n\t\t<H1>Meaning in life</H1>\n\t</BODY>\n</HTML>"; 
```

如你所见，前面的代码可能难以编写。你必须弄清楚新行和计算目标字符串值中的制表符，并将它们作为 `\n` 或 `\t` 插入。另一个挑战是阅读此代码。你必须尝试逐行弄清楚代码的意图。 

# 使用传统字符串字面量的连接地狱

以下是一个旨在通过在多行上定义字符串值的部分来使前面的代码可读的替代方案。然而，在这样做的时候，你应该使用多个字符串连接运算符（`+`）和字符串分隔符（`"`）：

```java
String html =  "<HTML>" + 
"\n\t" + "<BODY>" + 
"\n\t\t" + "<H1>Meaning of life</H1>" + 
"\n\t" + "</BODY>" + 
"\n" + "</HTML>";
```

我更愿意在前面代码中添加空格，使其更易于阅读：

```java
String html =  "<HTML>" + 
                    "\n\t" + "<BODY>" + 
                        "\n\t\t" + "<H1>Meaning of life</H1>" + 
                    "\n\t" + "</BODY>" + 
                "\n" + "</HTML>"; 
```

尽管前面的代码在可读性方面看起来更好，但它将大量责任委托给了程序员，在正确的位置插入空格。作为快速提示，空格（双引号之外）不是变量 HTML 的一部分；它们只是使其易于阅读。作为一个开发者，编写这样的代码是一种痛苦。

Java GUI 库不与控制字符（如换行符）一起工作。因此，这种方法可以与 GUI 类一起使用。

# 将转义序列作为字符串值的一部分包含

想象一下，当由转义序列（比如说，`\n`）表示的字母组合必须作为字符串值的一部分包含时会发生什么。让我们修改我们的示例，如下所示：

```java
<HTML> 
   <BODY> 
          <H1>\n - new line, \t - tab</H1>  
   </BODY> 
</HTML> 
```

在这种情况下，你可以通过使用另一个反斜杠来转义 `\n` 序列，如下所示：

```java
String html =  "<HTML>" + 
                    "\n\t" + "<BODY>" + 
                        "\n\t\t" + "<H1>\\n - new line, \\t - tab</H1>" 
                        +  
                    "\n\t" + "</BODY>" + 
                "\n" + "</HTML>";
```

但是，随着更改，前面的代码变得难以阅读和理解。想象一下，你的团队中的开发者编写了这样的代码。让我们看看你如何可以使它更易于阅读。

作为一种权宜之计，你可以定义 `String` 常量（比如说，`tab` 或 `newLine`），将 `\t` 和 `\n` 的值分配给它们。你可以使用这些常量代替前面代码中 `\n` 和 `\t` 的字面值。这种替换将使代码更容易阅读。

# 字符串和正则表达式模式，另一个地狱

本节将提供另一个转义序列地狱的例子——将正则表达式定义为 Java 字符串。

要移除单词字符和句点（`.`）之间的空白（空格或制表符），你需要以下 **regex**（正则表达式）模式：

```java
(\w)(\s+))[\.] 
```

要使用 Java 字符串存储它，你需要以下代码：

```java
String patternToMatch = "(\\w)(\\s+)([\\.,])";     // Isn't that a lot 
                                                   // to digest? 
                                                   // Does the regex  
                                                   // pattern has
                                                   // a single 
                                                   // backslash, or,
                                                   // two of them? 
```

我在这里分享一个快速的秘密——当我开始使用 Java 时，我最可怕的噩梦之一就是将模式定义为字符串值。我承认；我写模式有困难，使用模式中的转义字符`\`使我的生活变得痛苦。

这也被称为**倾斜牙签综合症**（**LTS**）——通过使用大量的反斜杠使字符串值难以阅读。

让我们通过使用原始字符串字面量来结束这种字符串的痛苦。

# 欢迎原始字符串字面量

原始字符串字面量是非解释字符串字面量。在实际项目中，你需要以原始方式处理字面量字符串值，而不需要对 Unicode 值、反斜杠或换行符进行特殊处理。原始字符串字面量将 Java 转义和 Java 行终止符规范放在一边，以便使代码可读性和可维护性。

# 使用原始字符串重写

你可以使用原始字面量来定义多行字符串值，如下所示：

```java
String html =  
`<HTML>                                      
    <BODY> 
        <H1>Meaning of life</H1>  
    </BODY> 
</HTML> 
`; 
```

通过使用 `` ` `` 作为开头和结尾分隔符，你可以轻松优雅地定义多行字符串字面量。

上述代码不包含 Java 指示符（连接运算符或转义序列）。

# 分隔符（反引号）

一个原始字符串字面量定义为如下：

```java
RawStringDelimeter {RawStringCharacters} RawStringDelimeter 
```

一个反引号用作原始字符串字面量的分隔符。我相信使用 `` ` `` 作为原始字符串字面量的分隔符是一个好决定。`` ` `` 反引号看起来像 `'`（单引号）和 `"`（双引号），它们已经被用作字符和字符串字面量的分隔符。`` ` `` 反引号将帮助 Java 程序员轻松将其视为原始字符串的分隔符。

原始字符串字面量使用 `` ` `` 作为分隔符。传统的字符串字面量使用 `"` 作为分隔符，字符使用 `'` 作为分隔符。

如果你希望将一个反引号作为字符串值的一部分，你可以使用两个反引号 (```java `` ```) 来定义你的值。这适用于 *n* 个反引号；只需确保匹配开闭反引号的数量。这听起来很有趣。让我们看看一个包含 `` ` `` 作为其值一部分的例子：

```java
String html =  
``<HTML>                                      
    <BODY> 
        <H1>I think I like ` as a delimiter!</H1>  
    </BODY> 
</HTML> 
``; 
```

下面是另一个例子，它使用字符串值中的多个反引号作为分隔符。当然，字符串值中包含的反引号数量不等于用作分隔符的反引号数量：

```java
String html =  
````<HTML>

    <BODY>

        <H1>I believe I would have liked ```java too(!)</H1>  
    </BODY> 
</HTML> 
````;

```java

If there is a mismatch between the count of backticks in the opening and closing delimiters, the code won't compile.

# Treating escape values

The Unicode and escape sequences are never interpreted in a raw string literal value.

A lexer is a software program that performs **lexical analysis**. Lexical analysis is the process of tokenizing a stream of characters into words and tokens. As you read this line, you are analyzing this string lexically, using the spaces to separate chunks of letters as words.

This analysis is disabled for the raw string literals at the start of the opening backtick. It is re-enabled at the closing backtick.

Don't replace the backtick in your string value with its Unicode escape (that is, `\u0060 in`), for consistency. 

The only exceptions to this rule are `CR` (carriage return—`\u000D`) and `CRLF` (carriage return and line feed—`\u000D\u000A`). Both of these are translated to `LF` (line feed—`\u000A`).

# Raw string literals versus traditional string literals

The introduction of raw string literals will not change the interpretation of the traditional string literal values. This includes their multiline capabilities and how they handle the escape sequences and delimiters. A traditional string can include Unicode escapes (JLS 3.3) or escape sequences (JLS 3.10.6).

A Java class file, that is, the Java bytecode, does not record whether a string constant was created using a traditional string or a raw string. Both traditional and raw string values are stored as instances of the `java.lang.String` class. 

# Interpreting escape sequences

To interpret escape sequences in multiline raw string values, Java will add methods to the `String` class—`unescape()` and `escape()`:

```

public String unescape() {...}

public String escape() {...}

```java

Considering raw string literals at Oracle, work is in progress. Oracle is considering swapping the names of the methods `unescape()` and `escape()`, or even renaming them.

# The unescape() method

By default, the raw string literals don't interpret the escape sequences. However, what if you want them to do so? When used with a raw string, the `unescape()` method will match the sequence of the characters following `\` with the sequence of Unicode escapes and escape sequences, as defined in the **Java Language Specifications** (**JLS**). If a match is found, the escape sequence will not be used as a regular combination of letters; it will be used as an escape sequence.

The following code doesn't interpret `\n` as a newline escape sequence:

```

System.out.print("eJava");

System.out.print("\\n");

System.out.print("Guru");

```java

The output of the preceding code is as follows:

```

eJava\nGuru

```java

However, the following code will interpret `\n` as a newline escape sequence:

```

System.out.print("eJava");

System.out.print(`\n`.unescape());      // 不要忽略转义字符

System.out.print("Guru");

```java

The output of the preceding code will provide the `eJava` and `Guru` string values on separate lines, as follows:

```

eJava

Guru

```java

When interpreted as escape sequences, a combination of letters that is used to represent them, is counted as a control character of the length `1`. The output of the following code will be `1`:

```

System.out.print(`\n`.unescape().length());

```java

# The escape() method

The `escape()` method will be used to invert the escapes. The following table shows how it will convert the characters:

| **Original character** | **Converted character** |
| Less than `' '` (space) | Unicode or character escape sequences |
| Above `~` (tilde) | Unicode escape sequences |
| `"` (double quotes) | Escape sequence |
| `'` (single quote) | Escape sequence |
| `\` (backslash) | Escape sequence |

The following example doesn't include a newline in the output:

```

System.out.println("eJava" + "\n".escape() + "Guru");

```java

The output of the preceding code is as follows:

```

eJava\nGuru

```java

Consider the following code:

```

System.out.println("eJava" + `•`.escape());

```java

The output of the preceding code is as follows (`•` is converted to its equivalent escape value):

```

eJava\u2022Guru

```java

# Managing margins

Suppose that you have to import multiline text from a file in your Java code. How would you prefer to treat the margins of the imported text? You may have to align or indent the imported text. The text might use a custom newline delimiter (say, `[`), which you might prefer to strip from the text. To work with these requirements, a set of new methods, such as `align()`, `indent()`, and `transform()`, are being added to the `String` class, which we'll cover in the next section.

# The align() method

When you define a multiline string value, you might choose to format the string against the left margin or align it with the indentation used by the code. The string values are stored with the margin intact. The `align()` method will provide incidental indentation support; it will strip off any leading or trailing blank lines, then justify each line, without losing the indentations.

The following is an example:

```

String comment =

    `one

        of

                my

        favorite

            lang

                feature

    from Amber(!)

`.align();

System.out.println(comment);

```java

The output of the preceding code is as follows:

```

one

of

        我的

    最爱

        语言

            功能

from Amber(!)

```java

# The indent(int) method

The `indent(int)` method will enable developers to specify custom indentations for their multiline string values. You can pass a positive number, `i`, to `indent(int)`, to add `i` spaces (`U+0020`) to your text, or you can pass a negative number, to remove a given number of whitespaces from each line of your multiline text.

An example is as follows:

```

String comment =

    `one

        of

                我的

        最爱

            语言

                功能

    from Amber(!)

`.align().indent(15);

System.out.println(comment);

```java

The output of the preceding code is as follows:

```

one

                of

                        我的

                最爱

                    语言

                        功能

            from Amber(!)

```java

The `indent(int)` method can be used to add or remove whitespaces from each line in a multitext value. By passing a positive number, you can add whitespaces, and you can remove them by passing negative values.

# The overloaded align(int) method

The `align(int)` method will first align the rows of a multistring value, and will then indent it with the specified spaces. The following is an example:

```

String comment =

    `one

        of

                我的

        最爱

            语言

                功能

    from Amber(!)

`.align(15);

System.out.println(comment);

```java

The output of the preceding code is as follows (the text on each line is preceded by fifteen spaces):

```

one

                of

                        我的

                最爱

                    语言

                        功能

            from Amber(!)

```java

# The detab(int) and entab methods

A tab (`U+0009`) usually represents four whitespaces (`U+0020`). However, not all of the applications convert between tabs and whitespaces when they use text that includes a mix of whitespaces and tabs. To combat this challenge with the multiline text, the `String` class will include two methods, `detab(int)` and `entab(int)`, which will convert a tab to whitespaces, and vice versa:

```

public String detab(int)

public String entab(int)

```java

Let's modify the preceding example, so that the content includes tabs instead of whitespaces, as follows:

```

String comment =

        `one

    of

            我的

        最爱

            语言

    功能

    from Amber(!)

`.detab(1);

```java

The output of the preceding code is as follows (each tab is converted to one whitespace):

```

one

of

我的

最爱

语言

功能

from Amber(!)

```java

# The transform() method

Suppose that a file includes the following text, using `[` at the beginning of a new line:

```

[ 讲座 - Java11, Amber, CleanCode

[ 海洋 - 塑料污染，人类冷漠

```java

Now, suppose that you must remove the delimiter, `[`, used at the beginning of all lines. You can use the `transform()` method to customize the margin management, adding the `String` class:

```

<R> R transform (Function<String, R> f)

```java

The following is an example that uses the method `transform()` to remove the custom margin characters from the multiline text:

```

String stripped = `

                    [ 讲座 - Java11, Amber, CleanCode

                    [ 海洋 - 濒临灭绝，人类冷漠，塑料

                    污染

                `.transform({

                    multiLineText.stream()

                        .map(e -> e.map(String::strip)

                        .map(s -> s.startsWith("[  ") ?

                            s.substring("[  ".length())

                            : s)

                        .collect(Collectors.joining("\n", "", "\n"));

                });

```java

The next section will include some common cases where using raw string literals over traditional strings will benefit you tremendously.

# Common examples

If you have used stored JSON, XML data, database queries, or file paths as string literals, you know it is difficult to write and read such code. This section will highlight examples of how a traditional raw string will improve the readability of your code when working with these types of data.

# JSON data

Suppose that you have the following JSON data to be stored as a Java string:

```

{"plastic": {

"id": "98751",

"singleuse": {

    "item": [

    {"value": "Water Bottle", "replaceWith": "Steel Bottle()"},

    {"value": "Straw", "replaceWith": "Ban Straws"},

    {"value": "Spoon", "replaceWith": "Steel Spoon"}

    ]

}

}}

```java

The following code shows how you would perform this action with the traditional `String` class, escaping the double quotes within the data by using `\` and adding a newline escape sequence:

```

String data =

"{\"plastic\": { \n" +

"\"id\": \"98751\", \n" +

"\"singleuse\": { \n" +

    "\"item\": [ \n" +

    "{\"value\": \"Water Bottle\", \"replaceWith\": \"Steel

        Bottle()\"}, \n" +

    "{\"value\": \"Straw\", \"replaceWith\": \"Ban Straws\"}, \n" +

    "{\"value\": \"Spoon\", \"replaceWith\": \"Steel Spoon\"} \n" +

    "] \n" +

"} \n" +

"}}";

```java

To be honest, it took me quite a while to write the preceding code, escaping the double quotes with the backslash. As you can see, this code is not readable.

The following example shows how you can code the same JSON data using raw string literals, without giving up the code readability:

```

String data =

```java{"plastic": { 
  "id": "98751", 
  "singleuse": { 
    "item": [ 
      {"value": "Water Bottle", "replaceWith": "Steel Bottle()"}, 
      {"value": "Straw", "replaceWith": "Ban Straws"}, 
      {"value": "Spoon", "replaceWith": "Steel Spoon"} 
    ] 
  } 
}}```;

```java

# XML data

The following code is an example of XML data that you might need to store using a Java string:

```

<plastic id="98751">

<singleuse>

    <item value="Water Bottle" replaceWith="Steel bottle" />

    <item value="Straw" replaceWith="Ban Straws" />

    <item value="spoon" replaceWith="Steel Spoon" />

</singleuse>

</plastic>

```java

The following code shows how you can define the preceding data as a `String` literal by using the appropriate escape sequences:

```

String data =

"<plastic id=\"98751\">\n" +

"<singleuse>\n" +

    "<item value=\"Water Bottle\" replaceWith=\"Steel bottle\" />\n" +

    "<item value=\"Straw\" replaceWith=\"Ban Straws\" />\n" +

    "<item value=\"spoon\" replaceWith=\"Steel Spoon\" />\n" +

"</singleuse>\n" +

"</plastic>";

```java

Again, the escape sequences added to the preceding code (`\` to escape `"` and `\n` to add newline) make it very difficult to read and understand the code. The following example shows how you can drop the programming-specific details from the data by using raw string literals:

```

String dataUsingRawStrings =

```java 
<plastic id="98751"> 
  <singleuse> 
    <item value="Water Bottle" replaceWith="Steel bottle" /> 
    <item value="Straw" replaceWith="Ban Straws" /> 
    <item value="spoon" replaceWith="Steel Spoon" /> 
  </singleuse> 
</plastic> 
```;

```java

# File paths

The file paths on Windows OS use a backslash to separate the directories and their subdirectories. For instance, `C:\Mala\eJavaGuru` refers to the `eJavaGuru` subdirectory in the `Mala` directory in the `C:` drive. Here's how you can store this path by using a traditional string literal:

```

String filePath = "C:\\Mala\\eJavaGuru\\ocp11.txt";

```java

With raw string literals, you can store the same file path as follows (changes are highlighted in bold):

```

String rawFilePath = `C:\Mala\eJavaGuru\ocp11.txt`;

```java

# Database queries

Suppose that you have to create an SQL query that includes the column names, table names, and literal values. The following code shows how you would typically write it, using traditional strings:

```

String query = "SELECT talk_title, speaker_name " +

            "FROM   talks, speakers " +

            "WHERE  talks.speaker_id = speakers.speaker_id " +

            "AND    talks.duration > 50 ";

```java

You can also write the code as follows, using quotes around the column or table names, depending on the target database management system:

```

String query = "SELECT 'talk_title', 'speaker_name' " +

            "FROM   'talks', 'speakers' " +

            "WHERE  'talks.speaker_id' = 'speakers.speaker_id' " +

            "AND    'talks.duration' > 50 ";

```java

The raw string literal values are much more readable, as shown in the following code:

```

String query =

```javaSELECT 'talk_title', 'speaker_name'  
   FROM   'talks', 'speakers'  
   WHERE  'talks.speaker_id' = 'speakers.speaker_id' 
   AND    'talks.duration' > 50  
```;

```

# 摘要

在本章中，我们讨论了开发者在将各种类型的多行文本值作为字符串值存储时面临的挑战。原始字符串字面量解决了这些问题。通过禁用转义字符和转义序列的词法分析，原始字符串字面量显著提高了多行字符串的可写性和可读性。原始字符串字面量将为`String`类引入多个方法，例如`unescape()`、`escape()`、`align()`、`indent()`和`transform()`。这些方法共同使得开发者能够专门处理原始字符串字面量。

在下一章中，我们将介绍 lambda 遗留项目如何改善 Java 中的函数式编程语法和体验。
