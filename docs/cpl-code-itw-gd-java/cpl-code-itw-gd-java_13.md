# 第十二章

*第十章*：

# 数组和字符串

这一章涵盖了涉及字符串和数组的一系列问题。由于 Java 字符串和数组是开发人员常见的话题，我将通过几个你必须记住的标题来简要介绍它们。然而，如果你需要深入研究这个主题，那么请考虑官方的 Java 文档（[`docs.oracle.com/javase/tutorial/java/`](https://docs.oracle.com/javase/tutorial/java/)）。

在本章结束时，你应该能够解决涉及 Java 字符串和/或数组的任何问题。这些问题很可能会出现在技术面试中。因此，本章将涵盖的主题非常简短和清晰：

+   数组和字符串概述

+   编码挑战

让我们从快速回顾字符串和数组开始。

# 技术要求

本章中的所有代码都可以在 GitHub 上找到，网址为[`github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter10`](https://github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter10)。

# 数组和字符串概述

在 Java 中，数组是对象并且是动态创建的。数组可以分配给`Object`类型的变量。它们可以有单个维度（例如，`m[]`）或多个维度（例如，作为三维数组，`m[][][]`）。数组的元素从索引 0 开始存储，因此长度为*n*的数组将其元素存储在索引 0 和*n*-1（包括）之间。一旦创建了数组对象，它的长度就永远不会改变。数组除了长度为 0 的无用数组（例如，`String[] immutable = new String[0]`）外，不能是不可变的。

在 Java 中，字符串是不可变的（`String`是不可变的）。字符串可以包含`char`数据类型（例如，调用`charAt(int index)`可以正常工作-`index`是从 0 到*字符串长度* - 1 变化的索引）。超过 65,535 直到 1,114,111（0x10FFFF）的 Unicode 字符不适合 16 位（Java`char`）。它们以 32 位整数值（称为*代码点*）存储。这一方面在*编码挑战 7-提取代理对的代码点*部分有详细说明。

用于操作字符串的一个非常有用的类是`StringBuilder`（以及线程安全的`StringBuffer`）。

现在，让我们来看一些编码挑战。

# 编码挑战

在接下来的 29 个编码挑战中，我们将解决一组在 Java 技术面试中遇到的流行问题，这些面试由中大型公司（包括 Google、Amazon、Flipkart、Adobe 和 Microsoft）进行。除了这本书中讨论的 29 个编码挑战，你可能还想查看我另一本书*Java 编码问题*（[`www.amazon.com/gp/product/1789801419/`](https://www.amazon.com/gp/product/1789801419/)）中的以下非详尽列表中的字符串和数组编码挑战，该书由 Packt 出版：

+   计算重复字符

+   找到第一个不重复的字符

+   反转字母和单词

+   检查字符串是否只包含数字

+   计算元音和辅音

+   计算特定字符的出现次数

+   从字符串中删除空格

+   用分隔符连接多个字符串

+   检查字符串是否是回文

+   删除重复字符

+   删除给定字符

+   找到出现最多次数的字符

+   按长度对字符串数组进行排序

+   检查字符串是否包含子字符串

+   计算字符串中子字符串出现的次数

+   检查两个字符串是否是变位词

+   声明多行字符串（文本块）

+   将相同的字符串* n *次连接

+   删除前导和尾随空格

+   找到最长的公共前缀

+   应用缩进

+   转换字符串

+   对数组进行排序

+   在数组中找到一个元素

+   检查两个数组是否相等或不匹配

+   按字典顺序比较两个数组

+   数组的最小值、最大值和平均值

+   反转数组

+   填充和设置数组

+   下一个更大的元素

+   改变数组大小

本章中涉及的 29 个编码挑战与前面的挑战没有涉及，反之亦然。

## 编码挑战 1 – 唯一字符（1）

**谷歌**，**Adobe**，**微软**

`true`如果这个字符串包含唯一字符。空格可以忽略。

**解决方案**：让我们考虑以下三个有效的给定字符串：

![图 10.1 字符串](img/Figure_10.1_B15403.jpg)

图 10.1 – 字符串

首先，重要的是要知道我们可以通过`charAt(int index)`方法获取 0 到 65,535 之间的任何字符（`index`是从 0 到*字符串长度* - 1 变化的索引），因为这些字符在 Java 中使用 16 位的`char`数据类型表示。

这个问题的一个简单解决方案是使用`Map<Character, Boolean>`。当我们通过`charAt(int index)`方法循环给定字符串的字符时，我们尝试将`index`处的字符放入这个映射，并将相应的`boolean`值从`false`翻转为`true`。如果给定键（字符）没有映射，则`Map#put(K k, V v)`方法返回`null`。如果给定键（字符）有映射，则`Map#put(K k, V v)`返回与此键关联的先前值（在我们的情况下为`true`）。因此，当返回的值不是`null`时，我们可以得出结论至少有一个字符是重复的，因此我们可以说给定的字符串不包含唯一字符。

此外，在尝试将字符放入映射之前，我们通过`String#codePointAt(index i)`确保其代码在 0 到 65,535 之间。这个方法返回指定`index`处的 Unicode 字符作为`int`，这被称为*代码点*。让我们看看代码：

```java
private static final int MAX_CODE = 65535;
...
public static boolean isUnique(String str) {
  Map<Character, Boolean> chars = new HashMap<>();
  // or use, for(char ch : str.toCharArray()) { ... }
  for (int i = 0; i < str.length(); i++) {
    if (str.codePointAt(i) <= MAX_CODE) {
      char ch = str.charAt(i);
      if (!Character.isWhitespace(ch)) {
        if (chars.put(ch, true) != null) {
          return false;
        }
      }
    } else {
      System.out.println("The given string 
        contains unallowed characters");
      return false;
    }
  }
  return true;
}
```

完整的应用程序称为*UniqueCharacters*。

## 编码挑战 2 – 唯一字符（2）

**谷歌**，**Adobe**，**微软**

`true`如果这个字符串包含唯一字符。空格可以忽略。

**解决方案**：前面编码挑战中提出的解决方案也涵盖了这种情况。但是，让我们试着提出一种特定于这种情况的解决方案。给定的字符串只能包含*a-z*中的字符，因此它只能包含从 97（*a*）到 122（*z*）的 ASCII 码。让我们假设给定的字符串是*afghnqrsuz*。

如果我们回顾一下*第九章**，位操作*中的经验，那么我们可以想象一个位掩码，它用 1 覆盖了*a*-*z*字母，如下图所示（1 的位对应于我们字符串的字母，*afghnqrsuz*）：

![图 10.2 – 独特字符位掩码](img/Figure_10.2_B15403.jpg)

图 10.2 – 唯一字符位掩码

如果我们将*a-z*中的每个字母表示为 1 的位，那么我们将获得一个唯一字符的位掩码，类似于前面图像中显示的位掩码。最初，这个位掩码只包含 0（因为没有处理任何字母，我们所有的位都等于 0 或者未设置）。

接下来，我们窥视给定字符串的第一个字母，并计算其 ASCII 码和 97（*a*的 ASCII 码）之间的差。让我们用*s*表示这个。现在，我们通过将 1 左移*s*位来创建另一个位掩码。这将导致一个位掩码，其最高位为 1，后面跟着*s*位的 0（1000...）。接下来，我们可以在唯一字符的位掩码（最初为 0000...）和这个位掩码（1000...）之间应用 AND[&]运算符。结果将是 0000...，因为 0 & 1 = 0。这是预期的结果，因为这是第一个处理的字母，所以唯一字符的位掩码中没有字母被翻转。

接下来，我们通过将位掩码中的位置*s*的位从 0 翻转为 1 来更新唯一字符的位掩码。这是通过 OR[|]运算符完成的。现在，唯一字符的位掩码是 1000.... 由于我们翻转了一个位，所以现在有一个单独的 1 位，即对应于第一个字母的 1 位。

最后，我们为给定字符串的每个字母重复此过程。如果遇到重复的字符，那么唯一字符的位掩码和当前处理的字母对应的 1000...掩码之间的 AND[&]操作将返回 1（1 & 1 = 1）。如果发生这种情况，那么我们已经找到了一个重复项，所以我们可以返回它。

在代码方面，我们有以下情况：

```java
private static final char A_CHAR = 'a';
...
public static boolean isUnique(String str) {
  int marker = 0;
  for (int i = 0; i < str.length(); i++) {
    int s = str.charAt(i) - A_CHAR;
    int mask = 1 << s;
    if ((marker & mask) > 0) {
      return false;
    }
    marker = marker | mask;
  }
  return true;
}
```

完整的应用程序称为*UniqueCharactersAZ*。

## 编码挑战 3 - 编码字符串

`char[]`，*str*。编写一小段代码，将所有空格替换为序列*%20*。结果字符串应作为`char[]`返回。

`char[]`代表以下字符串：

```java
char[] str = "  String   with spaces  ".toCharArray();
```

预期结果是*%20%20String%20%20%20with%20spaces%20%20*。

我们可以通过三个步骤解决这个问题：

1.  我们计算给定`char[]`中空格的数量。

1.  接下来，创建一个新的`char[]`，其大小为初始`char[]`*str*的大小，加上空格的数量乘以 2（单个空格占据给定`char[]`中的一个元素，而*%20*序列将占据结果`char[]`中的三个元素）。

1.  最后，我们循环给定的`char[]`并创建结果`char[]`。

在代码方面，我们有以下情况：

```java
public static char[] encodeWhitespaces(char[] str) {
  // count whitespaces (step 1)
  int countWhitespaces = 0;
  for (int i = 0; i < str.length; i++) {
    if (Character.isWhitespace(str[i])) {
        countWhitespaces++;
    }
  }
  if (countWhitespaces > 0) {
    // create the encoded char[] (step 2)
    char[] encodedStr = new char[str.length
      + countWhitespaces * 2];
    // populate the encoded char[] (step 3)
    int index = 0;
    for (int i = 0; i < str.length; i++) {
      if (Character.isWhitespace(str[i])) {
        encodedStr[index] = '0';
        encodedStr[index + 1] = '2';
        encodedStr[index + 2] = '%';
        index = index + 3;
      } else {
        encodedStr[index] = str[i];
        index++;
      }
    }
    return encodedStr;
  }
  return str;
}
```

完整的应用程序称为*EncodedString*。

## 编码挑战 4 - 一个编辑的距离

**Google**，**Microsoft**

**问题**：考虑两个给定的字符串*q*和*p*。编写一小段代码，确定我们是否可以通过在*q*或*p*中进行单个编辑来获得两个相同的字符串。更确切地说，我们可以在*q*或*p*中插入、删除或替换一个字符，*q*将变成等于*p*。

**解决方案**：为了更好地理解要求，让我们考虑几个例子：

+   *tank, tanc* 一个编辑：用*c*替换*k*（反之亦然）

+   *tnk, tank* 一个编辑：在*tnk*中的*t*和*n*之间插入*a*，或者从*tank*中删除*a*

+   *tank, tinck* 需要多于一个编辑！

+   *tank, tankist* 需要多于一个编辑！

通过检查这些例子，我们可以得出以下结论：如果发生以下情况，我们离目标只有一个编辑的距离：

+   *q*和*p*之间的长度差异不大于 1

+   *q*和*p*在一个地方不同

我们可以轻松地检查*q*和*p*之间长度的差异，如下所示：

```java
if (Math.abs(q.length() - p.length()) > 1) {
  return false;
}
```

要找出*q*和*p*在一个地方是否不同，我们必须将*q*的每个字符与*p*的每个字符进行比较。如果我们找到多于一个差异，那么我们返回`false`；否则，我们返回`true`。让我们看看这在代码方面是怎样的：

```java
public static boolean isOneEditAway(String q, String p) {
  // if the difference between the strings is bigger than 1 
  // then they are at more than one edit away
  if (Math.abs(q.length() - p.length()) > 1) {
    return false;
  }
  // get shorter and longer string
  String shorter = q.length() < p.length() ? q : p;
  String longer = q.length() < p.length() ? p : q;
  int is = 0;
  int il = 0;
  boolean marker = false;
  while (is < shorter.length() && il < longer.length()) {
    if (shorter.charAt(is) != longer.charAt(il)) {
      // first difference was found
      // at the second difference we return false
      if (marker) {
        return false;
      }
      marker = true;
      if (shorter.length() == longer.length()) {
        is++;
      }
    } else {
      is++;
    }
    il++;
  }
  return true;
}
```

完整的应用程序称为*OneEditAway*。

## 编码挑战 5 - 缩短字符串

**问题**：考虑一个只包含字母*a-z*和空格的给定字符串。这个字符串包含很多连续重复的字符。编写一小段代码，通过计算连续重复的字符并创建另一个字符串，将这个字符串缩小。空格应该按原样复制到结果字符串中（不要缩小空格）。如果结果字符串不比给定字符串短，那么返回给定字符串。

**解决方案**：考虑给定的字符串是*abbb vvvv s rttt rr eeee f*。预期结果将是*a1b3 v4 s1 r1t3 r2 e4 f1*。为了计算连续的字符，我们需要逐个字符循环这个字符串：

+   如果当前字符和下一个字符相同，那么我们增加一个计数器。

+   如果下一个字符与当前字符不同，那么我们将当前字符和计数器值附加到最终结果，并将计数器重置为 0。

+   最后，在处理给定字符串的所有字符之后，我们比较结果的长度与给定字符串的长度，并返回较短的字符串。

在代码方面，我们有以下情况：

```java
public static String shrink(String str) {
  StringBuilder result = new StringBuilder();
  int count = 0;
  for (int i = 0; i < str.length(); i++) {
    count++;
    // we don't count whitespaces, we just copy them
    if (!Character.isWhitespace(str.charAt(i))) {
      // if there are no more characters
      // or the next character is different
      // from the counted one
      if ((i + 1) >= str.length()
           || str.charAt(i) != str.charAt(i + 1)) {
        // append to the final result the counted character
        // and number of consecutive occurrences
        result.append(str.charAt(i))
              .append(count);
        // reset the counter since this 
        // sequence was appended to the result
        count = 0;
      }
    } else {
      result.append(str.charAt(i));
      count = 0;
    }
  }
  // return the result only if it is 
  // shorter than the given string
  return result.length() > str.length()
              ? str : result.toString();
}
```

完整的应用程序称为*StringShrinker*。

## 编码挑战 6 - 提取整数

**问题**：考虑一个包含空格和*a-z*和*0-9*字符的给定字符串。编写一小段代码，从这个字符串中提取整数。您可以假设任何连续数字序列都形成一个有效的整数。

**解决方案**：考虑给定的字符串是*cv dd 4 k 2321 2 11 k4k2 66 4d*。预期结果将包含以下整数：4, 2321, 2, 11, 4, 2, 66 和 4。

一个简单的解决方案将循环给定的字符串，逐个字符连接连续数字序列。数字包含 ASCII 代码在 48（包括）和 97（包括）之间。因此，任何 ASCII 代码在[48, 97]范围内的字符都是数字。我们还可以使用`Character#isDigit(char ch)`方法。当连续数字序列被非数字字符中断时，我们可以将收集到的序列转换为整数并将其附加为整数列表。让我们看看代码方面的内容：

```java
public static List<Integer> extract(String str) {
  List<Integer> result = new ArrayList<>();
  StringBuilder temp = new StringBuilder(
    String.valueOf(Integer.MAX_VALUE).length());
  for (int i = 0; i < str.length(); i++) {
    char ch = str.charAt(i);
    // or, if (((int) ch) >= 48 && ((int) ch) <= 57)
    if (Character.isDigit(ch)) { 
      temp.append(ch);
    } else {
      if (temp.length() > 0) {
        result.add(Integer.parseInt(temp.toString()));
        temp.delete(0, temp.length());
      }
    }
  }
  return result;
}
```

完整的应用程序称为*ExtractIntegers*。

## 编码挑战 7-提取代理对的代码点

**问题**：考虑一个包含任何类型字符的给定字符串，包括在 Java 中表示为*代理对*的 Unicode 字符。编写一小段代码，从列表中提取*代理对*的*代码点*。

**解决方案**：让我们考虑给定的字符串包含以下图像中显示的 Unicode 字符（前三个 Unicode 字符在 Java 中表示为*代理对*，而最后一个不是）：

![图 10.3-Unicode 字符（代理对）](img/Figure_10.3_B15403.jpg)

图 10.3-Unicode 字符（代理对）

在 Java 中，我们可以这样写这样的字符串：

```java
char[] musicalScore = new char[]{'\uD83C', '\uDFBC'}; 
char[] smileyFace = new char[]{'\uD83D', '\uDE0D'};   
char[] twoHearts = new char[]{'\uD83D', '\uDC95'};   
char[] cyrillicZhe = new char[]{'\u04DC'};          
String str = "is" + String.valueOf(cyrillicZhe) + "zhe"
  + String.valueOf(twoHearts) + "two hearts"
  + String.valueOf(smileyFace) + "smiley face and, "
  + String.valueOf(musicalScore) + "musical score";
```

为了解决这个问题，我们必须了解一些事情，如下（牢记以下陈述对于解决涉及 Unicode 字符的问题至关重要）：

+   超过 65,535 直到 1,114,111（0x10FFFF）的 Unicode 字符不适合 16 位，因此 32 位值（称为*代码点*）被考虑用于 UTF-32 编码方案。

不幸的是，Java 不支持 UTF-32！尽管如此，Unicode 已经提出了一个解决方案，仍然使用 16 位来表示这些字符。这个解决方案意味着以下内容：

+   16 位*高代理项*：1,024 个值（U+D800 到 U+DBFF）

+   16 位*低代理项*：1,024 个值（U+DC00 到 U+DFFF）

+   现在，*高代理项*后面跟着*低代理项*定义了所谓的*代理对*。这些*代理对*用于表示介于 65,536（0x10000）和 1,114,111（0x10FFFF）之间的值。

+   Java 利用这种表示并通过一系列方法公开它，例如`codePointAt()`，`codePoints()`，`codePointCount()`和`offsetByCodePoints()`（查看 Java 文档以获取详细信息）。

+   调用`codePointAt()`而不是`charAt()`，`codePoints()`而不是`chars()`等有助于我们编写涵盖 ASCII 和 Unicode 字符的解决方案。

例如，众所周知的双心符号（前图中的第一个符号）是一个 Unicode 代理对，可以表示为包含两个值的`char[]`：\uD83D 和\uDC95。这个符号的*代码点*是 128149。要从这个代码点获取一个`String`对象，请调用以下内容：

```java
String str = String.valueOf(Character.toChars(128149));
```

通过调用`str.codePointCount(0,str.length())`可以计算`str`中的代码点数，即使`str`的长度为 2，它也会返回 1。调用`str.codePointAt(0)`返回 128149，而调用`str.codePointAt(1)`返回 56469。调用`Character.toChars(128149).length`返回 2，因为需要两个字符来表示这个*代码点*作为 Unicode*代理对*。对于 ASCII 和 Unicode 16 位字符，它将返回 1。

基于这个例子，我们可以很容易地识别*代理对*，如下所示：

```java
public static List<Integer> extract(String str) {
  List<Integer> result = new ArrayList<>();
  for (int i = 0; i < str.length(); i++) {
    int cp = str.codePointAt(i);
    if (i < str.length()-1 
        && str.codePointCount(i, i+2) == 1) {
      result.add(cp);
      result.add(str.codePointAt(i+1));
      i++;
    }
  }
  return result;
}
```

或者，像这样：

```java
public static List<Integer> extract(String str) {
  List<Integer> result = new ArrayList<>();
  for (int i = 0; i < str.length(); i++) {
    int cp = str.codePointAt(i);        
    // the constant 2 means a suroggate pair       
    if (Character.charCount(cp) == 2) { 
      result.add(cp);
      result.add(str.codePointAt(i+1));
      i++;
    } 
  }
  return result;
}
```

完整的应用程序称为*ExtractSurrogatePairs*。

## 编码挑战 8-是否旋转

**亚马逊**，**谷歌**，**Adobe**，**微软**

**问题**：考虑两个给定的字符串*str1*和*str2*。编写一行代码，告诉我们*str2*是否是*str1*的旋转。

**解决方案**：假设*str1*是*helloworld*，*str2*是*orldhellow*。由于*str2*是*str1*的旋转，我们可以说*str2*是通过将*str1*分成两部分并重新排列得到的。以下图显示了这些单词：

![图 10.4 - 将 str1 分成两部分并重新排列](img/Figure_10.4_B15403.jpg)

图 10.4 - 将 str1 分成两部分并重新排列

因此，基于这个图像，让我们将剪刀的左侧表示为*p1*，将剪刀的右侧表示为*p2*。有了这些表示，我们可以说*p1 = hellow*，*p2 = orld*。此外，我们可以说*str1 = p1+p2 = hellow + orld*，*str2 = p2+p1 = orld + hellow*。因此，无论我们在*str1*的哪里进行切割，我们都可以说*str1 = p1+p2*，*str2=p2+p1*。然而，这意味着*str1+str2 = p1+p2+p2+p1 = hellow + orld + orld + hellow = p1+p2+p1+p2 = str1 + str1*，所以*p2+p1*是*p1+***p2+p1***+p2*的子字符串。换句话说，*str2*必须是*str1+str1*的子字符串；否则，它就不能是*str1*的旋转。在代码方面，我们可以写成以下形式：

```java
public static boolean isRotation(String str1, String str2) {      
  return (str1 + str1).matches("(?i).*" 
    + Pattern.quote(str2) + ".*");
}
```

完整的代码称为*RotateString*。

## 编码挑战 9 - 将矩阵逆时针旋转 90 度

**亚马逊**，**谷歌**，**Adobe**，**微软**，**Flipkart**

**问题**：考虑一个给定的整数*n* x *n*矩阵*M*。编写一小段代码，将此矩阵逆时针旋转 90 度，而不使用任何额外空间。

**解决方案**：对于这个问题，至少有两种解决方案。一种解决方案依赖于矩阵的转置，而另一种解决方案依赖于逐环旋转矩阵。

### 使用矩阵的转置

让我们来解决第一个解决方案，它依赖于找到矩阵*M*的转置。矩阵的转置是线性代数中的一个概念，意味着我们需要沿着其主对角线翻转矩阵，这将得到一个新的矩阵*M*T。例如，有矩阵*M*和索引*i*和*j*，我们可以写出以下关系：

![图 10.5 关系](img/Figure_10.5_B15403.jpg)

图 10.5 - 矩阵转置关系

一旦我们获得了*M*的转置，我们可以反转转置的列。这将给我们最终结果（矩阵*M*逆时针旋转 90 度）。以下图像阐明了这种关系，对于一个 5x5 的矩阵：

![图 10.6 - 矩阵的转置在左边，最终结果在右边](img/Figure_10.6_B15403.jpg)

图 10.6 - 矩阵的转置在左边，最终结果在右边

要获得转置(*M*T)，我们可以通过以下方法交换*M*[*j*][*i*]和*M*[*i*][*j*]：

```java
private static void transpose(int m[][]) {
  for (int i = 0; i < m.length; i++) {
    for (int j = i; j < m[0].length; j++) {
      int temp = m[j][i];
      m[j][i] = m[i][j];
      m[i][j] = temp;
    }
  }
}
```

反转*M*T 的列可以这样做：

```java
public static boolean rotateWithTranspose(int m[][]) {
  transpose(m);
  for (int i = 0; i < m[0].length; i++) {
    for (int j = 0, k = m[0].length - 1; j < k; j++, k--) {
      int temp = m[j][i];
      m[j][i] = m[k][i];
      m[k][i] = temp;
    }
  }
  return true;
}
```

这个解决方案的时间复杂度为 O(n2)，空间复杂度为 O(1)，因此我们满足了问题的要求。现在，让我们看看这个问题的另一个解决方案。

### 逐环旋转矩阵

如果我们将矩阵视为一组同心环，那么我们可以尝试旋转每个环，直到整个矩阵都被旋转。以下图像是一个 5x5 矩阵这个过程的可视化：

![图 10.7 - 逐环旋转矩阵](img/Figure_10.7_B15403.jpg)

图 10.7 - 逐环旋转矩阵

我们可以从最外层开始，最终逐渐向内部工作。要旋转最外层，我们从顶部(0, 0)开始逐个交换索引。这样，我们将右边缘移到顶边缘的位置，将底边缘移到右边缘的位置，将左边缘移到底边缘的位置，将顶边缘移到左边缘的位置。完成此过程后，最外层环将逆时针旋转 90 度。我们可以继续进行第二个环，从索引(1, 1)开始，并重复此过程，直到旋转第二个环。让我们看看代码方面的表现：

```java
public static boolean rotateRing(int[][] m) {
  int len = m.length;
  // rotate counterclockwise
  for (int i = 0; i < len / 2; i++) {
    for (int j = i; j < len - i - 1; j++) {
      int temp = m[i][j];
      // right -> top 
       m[i][j] = m[j][len - 1 - i];
       // bottom -> right 
       m[j][len - 1 - i] = m[len - 1 - i][len - 1 - j];
       // left -> bottom 
       m[len - 1 - i][len - 1 - j] = m[len - 1 - j][i];
       // top -> left
       m[len - 1 - j][i] = temp;
     }
   }                 
   return true;
 }
```

这个解决方案的时间复杂度为 O(n2)，空间复杂度为 O(1)，因此我们尊重了问题的要求。

完整的应用程序称为*RotateMatrix*。它还包含了将矩阵顺时针旋转 90 度的解决方案。此外，它还包含了将给定矩阵旋转到一个单独矩阵的解决方案。

## 编码挑战 10-包含零的矩阵

**Google**，**Adobe**

**问题**：考虑一个给定的*n* x *m*整数矩阵*M*。如果*M*(*i, j*)等于 0，则整行*i*和整列*j*应该只包含零。编写一小段代码来完成这个任务，而不使用任何额外的空间。

**解决方案**：一个天真的方法是循环遍历矩阵，对于每个(*i, j*) = 0，将行*i*和列*j*设置为零。问题在于当我们遍历这行/列的单元格时，我们会发现零并再次应用相同的逻辑。很有可能最终得到一个全是零的矩阵。

为了避免这种天真的方法，最好是拿一个例子并尝试可视化解决方案。让我们考虑一个 5x8 的矩阵，如下图所示：

![图 10.8-包含零的矩阵](img/Figure_10.8_B15403.jpg)

图 10.8-包含零的矩阵

初始矩阵在(0,4)处有一个 0，在(2,6)处有另一个 0。这意味着解决后的矩阵应该只在第 0 行和第 2 行以及第 4 列和第 6 列上包含零。

一个易于实现的方法是存储零的位置，并在对矩阵进行第二次遍历时，将相应的行和列设置为零。然而，存储零意味着使用一些额外的空间，这是问题所不允许的。

提示

通过一点技巧和一些工作，我们可以将空间复杂度设置为 O(1)。技巧在于使用矩阵的第一行和第一列来标记在矩阵的其余部分找到的零。例如，如果我们在单元格(*i*, *j*)处找到一个零，其中*i*≠0 且*j*≠0，则我们设置*M*[*i*][0] = 0 和*M*[0][*j*] = 0。完成了整个矩阵的这个操作后，我们可以循环遍历第一列（列 0）并传播在行上找到的每个零。之后，我们可以循环遍历第一行（行 0）并传播在列上找到的每个零。

但是第一行和第一列的潜在初始零怎么办？当然，我们也必须解决这个问题，所以我们首先标记第一行/列是否至少包含一个 0：

```java
boolean firstRowHasZeros = false;
boolean firstColumnHasZeros = false;
// Search at least a zero on first row
for (int j = 0; j < m[0].length; j++) {
  if (m[0][j] == 0) {
    firstRowHasZeros = true;
    break;
  }
}
// Search at least a zero on first column
for (int i = 0; i < m.length; i++) {
  if (m[i][0] == 0) {
    firstColumnHasZeros = true;
    break;
  }
}
```

此外，我们应用了我们刚才说的。为此，我们循环遍历矩阵的其余部分，对于每个 0，我们在第一行和列上标记它：

```java
// Search all zeros in the rest of the matrix
for (int i = 1; i < m.length; i++) {
  for (int j = 1; j < m[0].length; j++) {
    if (m[i][j] == 0) {
       m[i][0] = 0;
       m[0][j] = 0;
    }
  }
}
```

接下来，我们可以循环遍历第一列（列 0）并传播在行上找到的每个零。之后，我们可以循环遍历第一行（行 0）并传播在列上找到的每个零：

```java
for (int i = 1; i < m.length; i++) {
  if (m[i][0] == 0) {
    setRowOfZero(m, i);
  }
}
for (int j = 1; j < m[0].length; j++) {
  if (m[0][j] == 0) {
    setColumnOfZero(m, j);
  }
}
```

最后，如果第一行包含至少一个 0，则我们将整行设置为 0。同样，如果第一列包含至少一个 0，则我们将整列设置为 0：

```java
if (firstRowHasZeros) {
  setRowOfZero(m, 0);
}
if (firstColumnHasZeros) {
  setColumnOfZero(m, 0);
}
```

`setRowOfZero()`和`setColumnOfZero()`都很简单：

```java
private static void setRowOfZero(int[][] m, int r) {
  for (int j = 0; j < m[0].length; j++) {
    m[r][j] = 0;
  }
}
private static void setColumnOfZero(int[][] m, int c) {
  for (int i = 0; i < m.length; i++) {
    m[i][c] = 0;
  }
}
```

该应用程序称为*MatrixWithZeros*。

## 编码挑战 11-使用一个数组实现三个堆栈

**亚马逊**，**谷歌**，**Adobe**，**微软**，**Flipkart**

`push()`，`pop()`和`printStacks()`。

**解决方案**：提供所需实现的两种主要方法。我们将在这里讨论的方法是基于交错这三个堆栈的元素。查看以下图片：

![图 10.9-交错堆栈的节点](img/Figure_10.9_B15403.jpg)

图 10.9-交错堆栈的节点

正如您所看到的，有一个单一的数组，保存了这三个堆栈的节点，分别标记为*Stack 1*、*Stack 2*和*Stack 3*。我们实现的关键在于每个推送到堆栈（数组）上的节点都有一个指向其前一个节点的后向链接。每个堆栈的底部都有一个链接到-1。例如，对于*Stack 1*，我们知道索引 0 处的值 2 有一个指向虚拟索引-1 的后向链接，索引 1 处的值 12 有一个指向索引 0 的后向链接，索引 7 处的值 1 有一个指向索引 1 的后向链接。

因此，堆栈节点保存了两个信息 – 值和后向链接：

```java
public class StackNode {
  int value;
  int backLink;
  StackNode(int value, int backLink) {
    this.value = value;
    this.backLink = backLink;
  }
}
```

另一方面，数组管理着到下一个空闲槽的链接。最初，当数组为空时，我们只能创建空闲槽，因此链接的形式如下（注意`initializeSlots()`方法）：

```java
public class ThreeStack {
  private static final int STACK_CAPACITY = 15;
  // the array of stacks
  private final StackNode[] theArray;                   
  ThreeStack() {
    theArray = new StackNode[STACK_CAPACITY];
    initializeSlots();
  }
  ...   
  private void initializeSlots() {
    for (int i = 0; i < STACK_CAPACITY; i++) {
      theArray[i] = new StackNode(0, i + 1);
    }
  }
}
```

现在，当我们将一个节点推送到其中一个堆栈时，我们需要找到一个空闲槽并将其标记为非空闲。以下是相应的代码：

```java
public class ThreeStack {
  private static final int STACK_CAPACITY = 15;
  private int size;
  // next free slot in array
  private int nextFreeSlot;
  // the array of stacks
  private final StackNode[] theArray;                      
  // maintain the parent for each node
  private final int[] backLinks = {-1, -1, -1};  
  ...
  public void push(int stackNumber, int value) 
                throws OverflowException {
    int stack = stackNumber - 1;
    int free = fetchIndexOfFreeSlot();
    int top = backLinks[stack];
    StackNode node = theArray[free];
    // link the free node to the current stack
    node.value = value;
    node.backLink = top;
    // set new top
    backLinks[stack] = free;
  }
  private int fetchIndexOfFreeSlot()  
                throws OverflowException {
    if (size >= STACK_CAPACITY) {
      throw new OverflowException("Stack Overflow");
    }
    // get next free slot in array
    int free = nextFreeSlot;
    // set next free slot in array and increase size
    nextFreeSlot = theArray[free].backLink;
    size++;
    return free;
  }
}
```

当我们从堆栈中弹出一个节点时，我们必须释放该槽。这样，这个槽可以被未来的推送重用。相关的代码如下：

```java
public class ThreeStack {
  private static final int STACK_CAPACITY = 15;
  private int size;
  // next free slot in array
  private int nextFreeSlot;
  // the array of stacks
  private final StackNode[] theArray;                      
  // maintain the parent for each node
  private final int[] backLinks = {-1, -1, -1};  
  ...
  public StackNode pop(int stackNumber)
              throws UnderflowException {
    int stack = stackNumber - 1;
    int top = backLinks[stack];
    if (top == -1) {
      throw new UnderflowException("Stack Underflow");
    }
    StackNode node = theArray[top]; // get the top node
    backLinks[stack] = node.backLink;
    freeSlot(top);
    return node;
  }
  private void freeSlot(int index) {
    theArray[index].backLink = nextFreeSlot;
    nextFreeSlot = index;
    size--;
  }
}
```

完整的代码，包括使用`printStacks()`，被称为*ThreeStacksInOneArray*。

解决这个问题的另一种方法是将堆栈数组分割成三个不同的区域：

+   第一区域分配给第一个堆栈，并位于数组端点的左侧（当我们向这个堆栈推送时，它向右方向增长）。

+   第二区域分配给第二个堆栈，并位于数组端点的右侧（当我们向这个堆栈推送时，它向左方向增长）。

+   第三区域分配给第三个堆栈，并位于数组的中间（当我们向这个堆栈推送时，它可以向任何方向增长）。

以下图像将帮助您澄清这些观点：

![图 10.10 – 将数组分割成三个区域](img/Figure_10.10_B15403.jpg)

图 10.10 – 将数组分割成三个区域

这种方法的主要挑战在于通过相应地移动中间堆栈来避免堆栈碰撞。或者，我们可以将数组分成三个固定区域，并允许各个堆栈在有限的空间中增长。例如，如果数组大小为*s*，那么第一个堆栈可以从 0（包括）到*s*/3（不包括），第二个堆栈可以从*s*/3（包括）到 2*s*/3（不包括），第三个堆栈可以从 2*s*/3（包括）到*s*（不包括）。这种实现在捆绑代码中作为*ThreeStacksInOneArrayFixed*可用。

或者，可以通过交替序列实现中间堆栈以进行后续推送。这种方式，我们也可以减少移位，但我们正在减少均匀性。然而，挑战自己，也实现这种方法。

## 编码挑战 12 – 对

**Amazon**、**Adobe**、**Flipkart**

**问题**：考虑一个整数数组（正数和负数），*m*。编写一小段代码，找到所有和为给定数字*k*的整数对。

**解决方案**：像往常一样，让我们考虑一个例子。假设我们有一个包含 15 个元素的数组，如下所示：-5, -2, 5, 4, 3, 7, 2, 1, -1, -2, 15, 6, 12, -4, 3。另外，如果*k*=10，那么我们有四对和为 10 的数：(-15 + 5), (-2 + 12), (3 + 7), 和 (4 + 6)。但是我们如何找到这些对呢？

解决这个问题有不同的方法。例如，我们有蛮力方法（通常，面试官不喜欢这种方法，所以只在万不得已时使用它 – 尽管蛮力方法可以很好地帮助我们理解问题的细节，但它不被接受为最终解决方案）。按照蛮力方法，我们从数组中取出每个元素，并尝试与其余元素中的每个元素配对。与几乎任何基于蛮力的解决方案一样，这个解决方案的时间复杂度也是不可接受的。

如果我们考虑对给定数组进行排序，我们可以找到更好的方法。我们可以通过 Java 内置的`Arrays.sort()`方法来实现这一点，其运行时间为 O(n log n)。有了排序后的数组，我们可以使用两个指针来扫描整个数组，基于以下步骤（这种技术称为*双指针*，在本章的几个问题中都会看到它的应用）：

1.  一个指针从索引 0 开始（左指针；我们将其表示为*l*），另一个指针从（*m.length* - 1）索引开始（右指针；我们将其表示为*r*）。

1.  如果*m*[*l*] *+ m*[*r*] *= k*，那么我们有一个解决方案，我们可以增加*l*位置并减少*r*位置。

1.  如果*m*[*l*] *+ m*[*r*]*<k*，那么我们增加*l*并保持*r*不变。

1.  如果*m*[*l*] *+ m*[*r*]*>k*，那么我们减少*r*并保持*l*不变。

1.  我们重复*步骤 2-4*，直到*l>= r*。

以下图片将帮助您实现这些步骤：

![图 10.11 - 找到所有和为给定数字的对](img/Figure_10.11_B15403.jpg)

图 10.11 - 找到所有和为给定数字的对

在我们看看它如何适用于*k*=10 时，请留意这张图片：

+   *l*= 0，*r*= 14 → *sum* = *m*[0] + *m*[14] = -5 + 15 = 10 → *sum* = *k* → *l*++，*r*--

+   *l*= 1，*r*= 13 → *sum* = *m*[1] + *m*[13] = -4 + 12 = 8 → *sum < k* → *l*++

+   *l*= 2，*r*= 13 → *sum* = *m*[2] + *m*[13] = -2 + 12 = 10 → *sum* = *k* → *l*++，*r*--

+   *l*= 3，*r*= 12 → *sum* = *m*[3] + *m*[12] = -2 + 7 = 5 → *sum < k* → *l*++

+   *l*= 4，*r*= 12 → *sum* = *m*[4] + *m*[12] = -1 + 7 = 6 → *sum* < *k* → *l*++

+   *l*= 5，*r*= 12 → *sum* = *m*[5] + *m*[12] = 1 + 7 = 8 → *sum* < *k* → *l*++

+   *l*= 6，*r*= 12 → *sum* = *m*[6] + *m*[12] = 2 + 7 = 9 → *sum* < *k* → *l*++

+   *l*= 7，*r*= 12 → *sum* = *m*[7] + *m*[12] = 3 + 7 = 10 → *sum* = *k* → *l*++，*r*--

+   *l*= 8，*r*= 11 → *sum* = *m*[8] + *m*[11] = 3 + 6 = 9 → *sum* < *k* → *l*++

+   *l*= 9，*r*= 11 → *sum* = *m*[9] + *m*[11] = 4 + 6 = 10 → *sum* = *k* → *l*++，*r*--

+   *l*= 10，*r*= 10 → 停止

如果我们将这个逻辑放入代码中，那么我们会得到以下方法：

```java
public static List<String> pairs(int[] m, int k) {
  if (m == null || m.length < 2) {
    return Collections.emptyList();
  }
  List<String> result = new ArrayList<>();
  java.util.Arrays.sort(m);
  int l = 0;
  int r = m.length - 1;
  while (l < r) {
    int sum = m[l] + m[r];
    if (sum == k) {
      result.add("(" + m[l] + " + " + m[r] + ")");
      l++;
      r--;
    } else if (sum < k) {
      l++;
    } else if (sum > k) {
      r--;
    }
  }
  return result;
}
```

完整的应用程序称为*FindPairsSumEqualK*。

## 编码挑战 13 - 合并排序数组

**亚马逊**，**谷歌**，**Adobe**，**微软**，**Flipkart**

**问题**：假设您有*k*个不同长度的排序数组。编写一个应用程序，将这些数组合并到 O(nk log n)中，其中*n*是最长数组的长度。

**解决方案**：假设给定的数组是以下五个数组，分别表示为*a*、*b*、*c*、*d*和*e*：

*a*：{1, 2, 32, 46} *b*：{-4, 5, 15, 18, 20} *c*：{3} *d*：{6, 8} *e*：{-2, -1, 0}

预期结果如下：

{-4, -2, -1, 0, 1, 2, 3, 5, 6, 8, 15, 18, 20, 32, 46}

最简单的方法是将这些数组中的所有元素复制到单个数组中。这将花费 O(nk)的时间，其中*n*是最长数组的长度，*k*是数组的数量。接下来，我们通过 O(n log n)的时间复杂度算法（例如，通过归并排序）对这个数组进行排序。这将导致 O(nk log nk)。然而，问题要求我们编写一个可以在 O(nk log n)中执行的算法。

有几种解决方案可以在 O(nk log n)中执行，其中之一是基于二进制最小堆（这在*第十三章**，树和图*中有详细说明）。简而言之，二进制最小堆是一棵完全二叉树。二进制最小堆通常表示为一个数组（让我们将其表示为*heap*），其根位于*heap*[0]。更重要的是，对于*heap*[*i*]，我们有以下内容：

+   *heap*[(*i* - 1) / 2]：返回父节点

+   *heap*[(2 * *i*) + 1]：返回左子节点

+   *heap*[(2 * *i*) + 2]：返回右子节点

现在，我们的算法遵循以下步骤：

1.  创建大小为*nk*的结果数组。

1.  创建大小为*k*的二进制最小堆，并将所有数组的第一个元素插入到此堆中。

1.  重复以下步骤*nk*次：

从二进制最小堆中获取最小元素，并将其存储在结果数组中。

b. 用来自提取元素的数组的下一个元素替换二进制最小堆的根（如果数组没有更多元素，则用无限大替换根元素；例如，用`Integer.MAX_VALUE`）。

c. 替换根后，*heapify*树。

这段代码太长，无法在本书中列出，因此以下只是其实现的结尾（堆结构和`merge()`操作）：

```java
public class MinHeap {
  int data;
  int heapIndex;
  int currentIndex;
  public MinHeap(int data, int heapIndex,
        int currentIndex) {
    this.data = data;
    this.heapIndex = heapIndex;
    this.currentIndex = currentIndex;
  }
}
```

以下代码是`merge()`操作：

```java
public static int[] merge(int[][] arrs, int k) {
  // compute the total length of the resulting array
  int len = 0;
  for (int i = 0; i < arrs.length; i++) {
    len += arrs[i].length;
  }
  // create the result array
  int[] result = new int[len];
  // create the min heap
  MinHeap[] heap = new MinHeap[k];
  // add in the heap first element from each array
  for (int i = 0; i < k; i++) {
    heap[i] = new MinHeap(arrs[i][0], i, 0);
  }
  // perform merging
  for (int i = 0; i < result.length; i++) {
    heapify(heap, 0, k);
    // add an element in the final result
    result[i] = heap[0].data;
    heap[0].currentIndex++;
    int[] subarray = arrs[heap[0].heapIndex];
    if (heap[0].currentIndex >= subarray.length) {
      heap[0].data = Integer.MAX_VALUE;
    } else {
      heap[0].data = subarray[heap[0].currentIndex];
    }
  }
  return result;
}
```

完整的应用程序称为*MergeKSortedArr*。

## 编码挑战 14 - 中位数

**亚马逊**，**谷歌**，**Adobe**，**微软**，**Flipkart**

**问题**：考虑两个排序好的数组*q*和*p*（它们的长度可以不同）。编写一个应用程序，在对数时间内计算这两个数组的中位数值。

**解决方案**：中位数值将数据样本（例如数组）的较高一半与较低一半分开。例如，下图分别显示了具有奇数元素数量的数组和具有偶数元素数量的数组的中位数值：

![图 10.12 - 奇数和偶数数组的中位数值](img/Figure_10.12_B15403.jpg)

图 10.12 - 奇数和偶数数组的中位数值

因此，对于一个包含*n*个元素的数组，我们有以下两个公式：

+   如果*n*是奇数，则中位数值为(*n*+1)/2

+   如果*n*是偶数，则中位数值为[(*n*/2+(*n*/2+1)]/2

计算单个数组的中位数是相当容易的。但是，我们如何计算两个长度不同的数组的中位数呢？我们有两个排序好的数组，我们必须从中找出一些东西。有经验的求职者应该能够直觉到应该考虑使用著名的二分搜索算法。通常，在实现二分搜索算法时，应该考虑到有序数组。

我们大致可以直觉到，找到两个排序数组的中位数值可以简化为找到必须被这个值遵守的适当条件。

由于中位数值将输入分成两个相等的部分，我们可以得出第一个条件是*q*数组的中位数值应该在中间索引处。如果我们将这个中间索引表示为*qPointer*，那么我们得到两个相等的部分：[0，*qPointer*]和[*qPointer*+1，*q.length*]。如果我们对*p*数组应用相同的逻辑，那么*p*数组的中位数值也应该在中间索引处。如果我们将这个中间索引表示为*pPointer*，那么我们得到两个相等的部分：[0，*pPointer*]和[*pPointer*+1，*p.length*]。让我们通过以下图表来可视化这一点：

![图 10.13 - 将数组分成两个相等的部分](img/Figure_10.13_B15403.jpg)

图 10.13 - 将数组分成两个相等的部分

我们可以从这个图表中得出结论，中位数值应该遵守的第一个条件是*qLeft + pLeft = qRight + pRight*。换句话说，*qPointer + pPointer =* (*q.length- qPointer*) *+* (*p.length - pPointer*)。

然而，由于我们的数组长度不同（它们可以相等，但这只是我们解决方案应该覆盖的特殊情况），我们不能简单地将它们都减半。我们可以假设*p* >= *q*（如果它们没有给出这样的情况，那么我们只需交换它们以强制执行这个假设）。在这个假设的前提下，我们可以写出以下内容：

*qPointer + pPointer =* (*q.length- qPointer*) *+* (*p.length - pPointer*) *→*

2 ** pPointer = q.length + p.length -* 2 ** qPointer →*

*pPointer =* (*q.length + p.length*)*/*2 *- qPointer*

到目前为止，*pPointer*可以落在中间，我们可以通过添加 1 来避免这种情况，这意味着我们有以下起始指针：

+   *qPointer* = ((*q.length* - 1) + 0)/2

+   *pPointer* = (*q.length* + *p.length* + 1)/2 - *qPointer*

如果 *p*>=*q*，那么最小值 (*q.length* + *p.length* + 1)/2 - *qPointer* 将始终导致 *pPointer* 成为正整数。这将消除数组越界异常，并且也遵守第一个条件。

然而，我们的第一个条件还不够，因为它不能保证左数组中的所有元素都小于右数组中的元素。换句话说，左部分的最大值必须小于右部分的最小值。左部分的最大值可以是 *q*[*qPointer*-1] 或 *p*[*pPointer*-1]，而右部分的最小值可以是 *q*[*qPointer*] 或 *p*[*pPointer*]。因此，我们可以得出以下条件也应该被遵守：

+   *q*[*qPointer*-1] <= *p*[*pPointer*]

+   *p*[*pPointer*-1] <= *q*[*qPointer*]

在这些条件下，*q* 和 *p* 的中值将如下所示：

+   *p.length* + *q.length* 是偶数：左部分的最大值和右部分的最小值的平均值

+   *p.length* + *q.length* 是奇数：左部分的最大值，max(*q*[*qPointer*-1], *p*[*pPointer*-1])。

让我们尝试用三个步骤和一个例子总结这个算法。我们以 *q* 的中间值作为 *qPointer*（即[(*q.length* - 1) + 0)/2]），以 (*q.length* + *p.length* + 1)/2 - *qPointer* 作为 *pPointer*。让我们按照以下步骤进行：

1.  如果 *q*[*qPointer*-1] <= *p*[*pPointer*] 并且 *p*[*pPointer*-1] <= *q*[*qPointer*]，那么我们找到了完美的 *qPointer*（完美的索引）。

1.  如果 *p*[*pPointer*-1] >*q*[*qPointer*]，那么我们知道 *q*[*qPointer*] 太小了，所以必须增加 *qPointer* 并减少 *pPointer*。由于数组是排序的，这个操作将导致 *q*[*qPointer*] 变大，*p*[*pPointer*] 变小。此外，我们可以得出结论，*qPointer* 只能在 *q* 的右部分（从 *middle*+1 到 *q.length*）中。回到 *步骤 1*。

1.  如果 *q*[*qPointer*-1] >*p*[*pPointer*]，那么我们知道 *q*[*qPointer*-1] 太大了。我们必须减少 *qPointer* 以使 *q*[*qPointer*-1] <= *p*[*pPointer*]。此外，我们可以得出结论，*qPointer* 只能在 *q* 的左部分（从 0 到 *middle*-1）中。前往 *步骤 2*。

现在，让我们假设 *q*={ 2, 6, 9, 10, 11, 65, 67}，*p*={ 1, 5, 17, 18, 25, 28, 39, 77, 88}，并应用上述步骤。

根据我们之前的陈述，我们知道 *qPointer* = (0 + 6) / 2 = 3，*pPointer* = (7 + 9 + 1) / 2 - 3 = 5。下面的图像说明了这一点：

![图 10.14 - 计算中值（步骤 1）](img/Figure_10.14_B15403.jpg)

图 10.14 - 计算中值（步骤 1）

我们的算法的第 1 步规定 *q*[*qPointer*-1] <= *p*[*pPointer*] 并且 *p*[*pPointer*-1] <= *q*[*qPointer*]。显然，9 < 28，但 25 > 10，所以我们应用 *步骤 2*，然后回到 *步骤 1*。我们增加 *qPointer* 并减少 *pPointer*，所以 *qPointerMin* 变为 *qPointer* + 1。新的 *qPointer* 将是 (4 + 6) / 2 = 5，新的 *pPointer* 将是 (7 + 9 + 1)/2 - 5 = 3。下面的图像将帮助您可视化这种情况：

![图 10.15 - 计算中值（步骤 2）](img/Figure_10.15_B15403.jpg)

图 10.15 - 计算中值（步骤 2）

在这里，您可以看到新的 *qPointer* 和新的 *pPointer* 遵守了我们算法的 *步骤 1*，因为 *q*[*qPointer*-1]，即 11，小于 *p*[*pPointer*]，即 18；而 *p*[*pPointer*-1]，即 17，小于 *q*[*qPointer*]，即 65。有了这个，我们找到了完美的 *qPointer*，为 5。

最后，我们必须找到左侧的最大值和右侧的最小值，并根据两个数组的奇偶长度返回左侧的最大值或左侧的最大值和右侧的最小值的平均值。我们知道左侧的最大值是 max(*q*[*qPointer*-1], *p*[*pPointer*-1])，所以 max(11, 17) = 17。我们也知道右侧的最小值是 min(*q*[*qPointer*], *p*[*pPointer*])，所以 min(65, 18) = 18。由于长度之和为 7 + 9 = 16，我们计算出中位数的值是这两个值的平均值，所以 avg(17, 18) = 17.5。我们可以将其可视化如下：

![图 10.16 - 中位数（最终结果）](img/Figure_10.16_B15403.jpg)

图 10.16 - 中位数（最终结果）

将这个算法转化为代码的结果如下：

```java
public static float median(int[] q, int[] p) {
  int lenQ = q.length;
  int lenP = p.length;
  if (lenQ > lenP) {
    swap(q, p);
  }
  int qPointerMin = 0;
  int qPointerMax = q.length;
  int midLength = (q.length + p.length + 1) / 2;
  int qPointer;
  int pPointer;
  while (qPointerMin <= qPointerMax) {
    qPointer = (qPointerMin + qPointerMax) / 2;
    pPointer = midLength - qPointer;
    // perform binary search
    if (qPointer < q.length 
          && p[pPointer-1] > q[qPointer]) {
      // qPointer must be increased
      qPointerMin = qPointer + 1;
    } else if (qPointer > 0 
          && q[qPointer-1] > p[pPointer]) {
      // qPointer must be decreased
      qPointerMax = qPointer - 1;
    } else { // we found the poper qPointer
      int maxLeft = 0;
      if (qPointer == 0) { // first element on array 'q'?
        maxLeft = p[pPointer - 1];
      } else if (pPointer == 0) { // first element                                   // of array 'p'?
        maxLeft = q[qPointer - 1];
      } else { // we are somewhere in the middle -> find max
        maxLeft = Integer.max(q[qPointer-1], p[pPointer-1]);
      }
      // if the length of 'q' + 'p' arrays is odd, 
      // return max of left
      if ((q.length + p.length) % 2 == 1) {
        return maxLeft;
      }
      int minRight = 0;
      if (qPointer == q.length) { // last element on 'q'?
        minRight = p[pPointer];
      } else if (pPointer == p.length) { // last element                                          // on 'p'?
        minRight = q[qPointer];
      } else { // we are somewhere in the middle -> find min
        minRight = Integer.min(q[qPointer], p[pPointer]);
      }
      return (maxLeft + minRight) / 2.0f;
    }
  }
  return -1;
}
```

我们的解决方案在 O(log(max(*q.length, p.length*))时间内执行。完整的应用程序称为*MedianOfSortedArrays*。

## 编码挑战 15-一个的子矩阵

**亚马逊**，**微软**，**Flipkart**

**问题**：假设你得到了一个只包含 0 和 1（二进制矩阵）的矩阵，*m* x *n*。编写一小段代码，返回只包含元素 1 的最大正方形子矩阵的大小。

**解决方案**：让我们假设给定的矩阵是以下图像中的矩阵（5x7 矩阵）：

![图 10.17 - 给定的 5x7 二进制矩阵](img/Figure_10.17_B15403.jpg)

图 10.17 - 给定的 5 x 7 二进制矩阵

正如你所看到的，只包含元素 1 的正方形子矩阵的大小为 3。蛮力方法，或者说是朴素方法，是找到所有包含所有 1 的正方形子矩阵，并确定哪一个具有最大的大小。然而，对于一个*m* x *n*矩阵，其中*z*=min(*m, n*)，时间复杂度将为 O(z3mn)。你可以在本书附带的代码中找到蛮力实现。当然，在查看解决方案之前，先挑战自己。

现在，让我们试着找到一个更好的方法。让我们假设给定的矩阵是大小为*n* x *n*，并研究一个 4x4 样本矩阵的几种情况。在 4x4 矩阵中，我们可以看到 1s 的最大正方形子矩阵可以有 3x3 的大小，因此在大小为*n* x *n*的矩阵中，1s 的最大正方形子矩阵可以有大小为*n*-1x *n*-1。此外，以下图像显示了对*m x n*矩阵同样适用的两个基本情况：

![图 10.18 - 4x4 矩阵中 1s 的最大子矩阵](img/Figure_10.18_B15403.jpg)

图 10.18 - 4 x 4 矩阵中 1s 的最大子矩阵

这些情况解释如下：

+   如果给定的矩阵只包含一行，那么其中包含 1 的单元格将是最大正方形子矩阵的大小。因此，最大大小为 1。

+   如果给定的矩阵只包含一列，那么其中包含 1 的单元格将是最大正方形子矩阵的大小。因此，最大大小为 1。

接下来，让我们假设*subMatrix*[*i*][*j*]表示以单元格(*i,j*)结尾的只包含 1 的最大正方形子矩阵的大小：

![图 10.19 - 整体递归关系](img/Figure_10.19_B15403.jpg)

图 10.19 - 整体递归关系

前面的图表允许我们在给定矩阵和辅助*subMatrix*（与给定矩阵大小相同的矩阵，应根据递归关系填充）之间建立递归关系：

+   这并不容易直觉到，但我们可以看到，如果*matrix*[*i*][*j*] = 0，那么*subMatrix*[*i*][*j*] = 0

+   如果*matrix*[*i*][*j*] = 1，那么*subMatrix*[*i*][*j*]

= 1 + min(*subMatrix*[*i* - 1][*j*], *subMatrix*[*i*][*j* - 1], *subMatrix*[*i* - 1][*j* - 1])

如果我们将这个算法应用到我们的 5 x 7 矩阵中，那么我们将得到以下结果：

![图 10.20 - 解决我们的 5x7 矩阵](img/Figure_10.20_B15403.jpg)

图 10.20 - 解决我们的 5 x 7 矩阵

将前述基本情况和递归关系结合起来，得到以下算法：

1.  创建一个与给定矩阵大小相同的辅助矩阵（*subMatrix*）。

1.  从给定矩阵中复制第一行和第一列到这个辅助*subMatrix*（这些是基本案例）。

1.  对于给定矩阵的每个单元格（从（1, 1）开始），执行以下操作：

a. 填充符合前述递归关系的*subMatrix*。

b. 跟踪*subMatrix*的最大元素，因为这个元素给出了包含所有 1 的子矩阵的最大大小。

以下实现澄清了任何剩余的细节：

```java
public static int ofOneOptimized(int[][] matrix) {
  int maxSubMatrixSize = 1;
  int rows = matrix.length;
  int cols = matrix[0].length;                
  int[][] subMatrix = new int[rows][cols];
  // copy the first row
  for (int i = 0; i < cols; i++) {
    subMatrix[0][i] = matrix[0][i];
  }
  // copy the first column
  for (int i = 0; i < rows; i++) {
    subMatrix[i][0] = matrix[i][0];
  }
  // for rest of the matrix check if matrix[i][j]=1
  for (int i = 1; i < rows; i++) {
    for (int j = 1; j < cols; j++) {
      if (matrix[i][j] == 1) {
        subMatrix[i][j] = Math.min(subMatrix[i - 1][j - 1],
            Math.min(subMatrix[i][j - 1], 
             subMatrix[i - 1][j])) + 1;
        // compute the maximum of the current sub-matrix
        maxSubMatrixSize = Math.max(
          maxSubMatrixSize, subMatrix[i][j]);
      }
    }
  }        
  return maxSubMatrixSize;
}
```

由于我们迭代*m***n*次来填充辅助矩阵，因此这种解决方案的总体复杂度为 O(mn)。完整的应用程序称为*MaxMatrixOfOne*。

## 编码挑战 16 – 包含最多水的容器

**Google**，**Adobe**，**Microsoft**

**问题**：假设给定了*n*个正整数*p*1，*p*2，...，*p*n，其中每个整数表示坐标点（*i, p*i）。接下来，画出*n*条垂直线，使得线*i*的两个端点分别位于（*i, p*i）和（*i, 0）。编写一小段代码，找到两条线，与 X 轴一起形成一个包含最多水的容器。

**解决方案**：假设给定的整数是 1, 4, 6, 2, 7, 3, 8, 5 和 3。根据问题陈述，我们可以勾画*n*条垂直线（线 1：{(0, 1), (0, 0)}，线 2：{(1, 4), (1,0)}，线 3：{(2, 6), (2, 0)}，依此类推）。这可以在下图中看到：

![图 10.19 – n 条垂直线表示](img/Figure_10.21_B15403.jpg)

图 10.21 – n 条垂直线表示

首先，让我们看看如何解释这个问题。我们必须找到包含最多水的容器。这意味着在我们的 2D 表示中，我们必须找到具有最大面积的矩形。在 3D 表示中，这个容器将具有最大体积，因此它将包含最多的水。

用暴力方法思考解决方案是非常直接的。对于每条线，我们计算显示其余线的面积，同时跟踪找到的最大面积。这需要两个嵌套循环，如下所示：

```java
public static int maxArea(int[] heights) {
  int maxArea = 0;
  for (int i = 0; i < heights.length; i++) {
    for (int j = i + 1; j < heights.length; j++) {
      // traverse each (i, j) pair
      maxArea = Math.max(maxArea, 
          Math.min(heights[i], heights[j]) * (j - i));
    }
  }
  return maxArea;
}
```

这段代码的问题在于它的运行时间是 O(n2)。更好的方法是采用一种称为*双指针*的技术。别担心 - 这是一种非常简单的技术，对你的工具箱非常有用。你永远不知道什么时候会用到它！

我们知道我们正在寻找最大面积。因为我们谈论的是一个矩形区域，这意味着最大面积必须尽可能多地容纳*最大宽度*和*最大高度*之间的最佳报告。最大宽度是从 0 到*n*-1（在我们的例子中，从 0 到 8）。要找到最大高度，我们必须调整最大宽度，同时跟踪最大面积。为此，我们可以从最大宽度开始，如下图所示：

![图 10.22 – 最大宽度的区域](img/Figure_10.22_B15403.jpg)

图 10.22 – 最大宽度的区域

因此，如果我们用两个指针标记最大宽度的边界，我们可以说*i*=0 和*j*=8（或*n*-1）。在这种情况下，容纳水的容器的面积将为*p*i* 8 = 1 * 8 = 8。容器的高度不能超过*p*i = 1，因为水会流出。然而，我们可以增加*i*（*i*=1，*p*i=4）以获得更高的容器，可能是更大的容器，如下图所示：

![图 10.23 – 增加 i 以获得更大的容器](img/Figure_10.23_B15403.jpg)

图 10.23 – 增加 i 以获得更大的容器

一般来说，如果*p*i ≤ *p*j，则增加*i*；否则，减少*j*。通过不断增加/减少*i*和*j*，我们可以获得最大面积。从左到右，从上到下，下面的图像显示了这个语句在接下来的六个步骤中的工作：

![图 10.24 – 在增加/减少 i 和 j 时计算面积](img/Figure_10.24_B15403.jpg)

图 10.24 – 在增加/减少*i*和*j*时计算面积

步骤如下：

1.  在左上角的图像中，我们减少了*j*，因为*p*i *> p*j*，p*1 *> p*8 (4 > 3)。

1.  在顶部中间的图像中，我们增加了*i*，因为*p*i *< p*j*，p*1 *< p*7 (4 < 5)。

1.  在右上角的图像中，我们减少了*j*，因为*p*i *> p*j*，p*2 *> p*7 (6 > 5)。

1.  在左下角的图像中，我们增加了*i*，因为*p*i *< p*j*，p*2 < *p*6 (6 < 8)。

1.  在底部中间的图像中，我们增加了*i*，因为*p*i *< p*j*，p*3 *< p*6 (2 < 8)。

1.  在右下角的图像中，我们增加了*i*，因为*p*i *< p*j*，p*4 *< p*6 (7 < 8)。

完成！如果我们再增加*i*或减少*j*一次，那么*i=j*，面积为 0。此时，我们可以看到最大面积为 25（顶部中间的图像）。嗯，这种技术被称为*双指针*，可以用以下算法实现：

1.  从最大面积为 0*，i*=0 和*j=n*-1 开始。

1.  当*i < j*时，执行以下操作：

a. 计算当前*i*和*j*的面积。

b. 根据需要更新最大面积。

c. 如果*p*i *≤ p*j，则*i++;* 否则，*j--*。

在代码方面，我们有以下内容：

```java
public static int maxAreaOptimized(int[] heights) {
  int maxArea = 0;
  int i = 0; // left-hand side pointer            
  int j = heights.length - 1; // right-hand side pointer
  // area cannot be negative, 
  // therefore i should not be greater than j
  while (i < j) {
    // calculate area for each pair
    maxArea = Math.max(maxArea, Math.min(heights[i],
         heights[j]) * (j - i));
    if (heights[i] <= heights[j]) {
      i++; // left pointer is small than right pointer
    } else {
      j--; // right pointer is small than left pointer
    }
  }
  return maxArea;
}
```

这段代码的运行时间是 O(n)。完整的应用程序称为*ContainerMostWater*。

## 编码挑战 17 – 在循环排序数组中搜索

**亚马逊**，**谷歌**，**Adobe**，**微软**，**Flipkart**

**问题**：考虑到你已经得到了一个没有重复的整数的循环排序数组*m*。编写一个程序，在 O(log n)的时间复杂度内搜索给定的*x*。

**解决方案**：如果我们能在 O(n)的时间复杂度内解决这个问题，那么蛮力方法是最简单的解决方案。在数组中进行线性搜索将给出所搜索的*x*的索引。然而，我们需要提出一个 O(log n)的解决方案，因此我们需要从另一个角度来解决这个问题。

我们有足够的线索指向我们熟知的二分搜索算法，我们在*第七章**，算法的大 O 分析*和*第十四章**，排序和搜索*中讨论过。我们有一个排序后的数组，我们需要找到一个特定的值，并且需要在 O(log n)的时间复杂度内完成。因此，有三个线索指向我们二分搜索算法。当然，最大的问题在于排序后的数组的循环性，因此我们不能应用普通的二分搜索算法。

让我们假设*m* = {11, 14, 23, 24, -1, 3, 5, 6, 8, 9, 10}，*x* = 14，我们期望的输出是索引 1。以下图像介绍了几个符号，并作为解决手头问题的指导：

![图 10.25 – 循环排序数组和二分搜索算法](img/Figure_10.25_B15403.jpg)

图 10.25 – 循环排序数组和二分搜索算法

由于排序后的数组是循环的，我们有一个*pivot*。这是一个指向数组头部的索引。从*pivot*左边的元素已经被旋转。当数组没有旋转时，它将是{-1, 3, 5, 6, 8, 9, 10, **11, 14, 23, 24**}。现在，让我们看一下基于二分搜索算法的解决方案步骤：

1.  我们应用二分搜索算法，因此我们从计算数组的*middle*开始，即(*left + right*) / 2。

1.  我们检查是否*x* = *m*[*middle*]。如果是，则返回*middle*。如果不是，则继续下一步。

1.  接下来，我们检查数组的右半部分是否已排序。如果*m*[*middle*] <= *m*[*right*]，则范围[*middle, right*]中的所有元素都已排序：

a. 如果*x* > *m*[*middle*]并且*x* <= *m*[*right*]，那么我们忽略左半部分，设置*left* = *middle* + 1，并从*步骤 1*重复。

b. 如果*x* <= *m*[*middle*]或*x > m*[*right*]，那么我们忽略右半部分，设置*right = middle* - 1，并从*步骤 1*重复。

1.  如果数组的右半部分没有排序，那么左半部分必须是排序的：

a. 如果*x >= m*[*left*]并且*x < m*[*middle*]，那么我们忽略右半部分，设置*right = middle*- 1，并从*步骤 1*重复。

b. 如果*x < m*[*left*]或*x >= m*[*middle*]，那么我们忽略左半部分，设置*left = middle* + 1，并从*步骤 1*重复。

我们重复*步骤 1-4*，只要我们没有找到*x*或*left <= right*。

让我们将前述算法应用到我们的情况中。

因此，*middle*是(*left + right*) / 2 = (0 + 10) / 2 = 5。由于*m*[5] ≠14（记住 14 是*x*），我们继续进行*步骤 3*。由于*m*[5]<*m*[10]，我们得出右半部分是排序的结论。然而，我们注意到*x>m*[*right*]（14 >10），所以我们应用*步骤 3b*。基本上，我们忽略右半部分，然后设置*right = middle* - 1 = 5 - 1 = 4。我们再次应用*步骤 1*。

新的*middle*是(0 + 4) / 2 = 2。由于*m*[2]≠14，我们继续进行*步骤 3*。由于*m*[2] >*m*[4]，我们得出左半部分是排序的结论。我们注意到*x*>*m*[*left*]（14 >11）和*x*<*m*[*middle*]（14<23），所以我们应用*步骤 4a*。我们忽略右半部分，然后设置*right*= *middle* - 1 = 2 - 1 = 1。我们再次应用*步骤 1*。

新的*middle*是(0 + 1) / 2 = 0。由于*m*[0]≠14，我们继续进行*步骤 3*。由于*m*[0]<*m*[1]，我们得出右半部分是排序的结论。我们注意到*x* > *m*[*middle*]（14 > 11）和*x* = *m*[*right*]（14 = 14），所以我们应用*步骤 3a*。我们忽略左半部分，然后设置*left = middle* + 1 = 0 + 1 = 1。我们再次应用*步骤 1*。

新的*middle*是(1 + 1) / 2 = 1。由于*m*[1]=14，我们停止并返回 1 作为我们找到搜索值的数组索引。

让我们把这些放入代码中：

```java
public static int find(int[] m, int x) {
  int left = 0;
  int right = m.length - 1;
  while (left <= right) {
    // half the search space
    int middle = (left + right) / 2;
    // we found the searched value
    if (m[middle] == x) {
      return middle;
    }
    // check if the right-half is sorted (m[middle ... right])
    if (m[middle] <= m[right]) {
      // check if n is in m[middle ... right]
      if (x > m[middle] && x <= m[right]) {
        left = middle + 1;  // search in the right-half
      } else {
        right = middle - 1;	// search in the left-half
      }
    } else { // the left-half is sorted (A[left ... middle])
      // check if n is in m[left ... middle]
      if (x >= m[left] && x < m[middle]) {
        right = middle - 1; // search in the left-half
      } else {
        left = middle + 1; // search in the right-half
      }
    }
  }
  return -1;
}
```

完整的应用程序称为*SearchInCircularArray*。类似的问题会要求你在一个循环排序的数组中找到最大值或最小值。虽然这两个应用程序都包含在捆绑代码中，分别为*MaximumInCircularArray*和*MinimumInCircularArray*，但建议你利用到目前为止学到的知识，挑战自己找到解决方案。

## 编码挑战 18-合并间隔

**亚马逊**，**谷歌**，**Adobe**，**微软**，**Flipkart**

**问题**：考虑到你已经得到了一个[*start, end*]类型的间隔数组。编写一小段代码，合并所有重叠的间隔。

**解决方案**：让我们假设给定的间隔是[12,15]，[12,17]，[2,4]，[16,18]，[4,7]，[9,11]和[1,2]。在我们合并重叠的间隔之后，我们得到以下结果：[1, 7]，[9, 11] [12, 18]。

我们可以从蛮力方法开始。很直观的是，我们取一个间隔（让我们将其表示为*p*i），并将其结束（*p*ei）与其余间隔的开始进行比较。如果另一个间隔的开始（来自其余间隔）小于*p*的结束，那么我们可以合并这两个间隔。合并间隔的结束变为这两个间隔的结束的最大值。但是这种方法的时间复杂度为 O(n2)，所以它不会给面试官留下深刻印象。

然而，蛮力方法可以为我们尝试更好的实现提供重要提示。在任何时刻，我们必须将*p*的结束与另一个间隔的开始进行比较。这很重要，因为它可以引导我们思考按照它们的开始时间对间隔进行排序的想法。这样，我们可以大大减少比较的次数。有了排序的间隔，我们可以在线性遍历中合并所有间隔。

让我们尝试使用一个图形表示我们的样本间隔，按照它们的开始时间按升序排序（*p*si<*p*si+1<*p*si+2）。此外，每个间隔始终是向前看的（*p*ei>*p*si，*p*ei+1>*p*si+1，*p*ei+2>*p*si+2，依此类推）。这将帮助我们理解我们即将介绍的算法：

![图 10.26-对给定的间隔进行排序](img/Figure_10.26_B15403.jpg)

图 10.26-对给定的间隔进行排序

根据前面的图像，我们可以看到如果*p*的起始值大于前一个*p*的结束值（*p*si>*p*ei-1），那么下一个*p*的起始值也大于前一个*p*的结束值（*p*si+1>*p*ei-1），所以不需要比较前一个*p*和下一个*p*。换句话说，如果*p*i 不与*p*i-1 重叠，则*p*i+1 也不能与*p*i-1 重叠，因为*p*i+1 的起始值必须大于或等于*p*i。

如果*p*si 小于*p*ei-1，则我们应该将*p*ei-1 更新为*p*ei-1 和*p*ei 之间的最大值，并移动到*p*ei+1。这可以通过栈来完成，具体步骤如下：

![图 10.27 – 使用栈解决问题](img/Figure_10.27_B15403.jpg)

图 10.27 – 使用栈解决问题

这些是发生的步骤：

**步骤 0**：我们从一个空栈开始。

**步骤 1**：由于栈是空的，我们将第一个区间（[1, 2]）推入栈中。

**步骤 2**：接下来，我们关注第二个区间（[2, 4]）。[2, 4]的起始值等于栈顶部的区间[1, 2]的结束值，所以我们不将[2, 4]推入栈中。我们继续比较[1, 2]的结束值和[2, 4]的结束值。由于 2 小于 4，我们将区间[1, 2]更新为[1, 4]。所以，我们将[1, 2]与[2, 4]合并。

**步骤 3**：接下来，我们关注区间[4, 7]。[4, 7]的起始值等于栈顶部的区间[1, 4]的结束值，所以我们不将[4, 7]推入栈中。我们继续比较[1, 4]的结束值和[4, 7]的结束值。由于 4 小于 7，我们将区间[1, 4]更新为[1, 7]。所以，我们将[1, 4]与[4, 7]合并。

**步骤 4**：接下来，我们关注区间[9, 11]。[9, 11]的起始值大于栈顶部的区间[1, 7]的结束值，所以区间[1, 7]和[9, 11]不重叠。这意味着我们可以将区间[9, 11]推入栈中。

**步骤 5**：接下来，我们关注区间[12, 15]。[12, 15]的起始值大于栈顶部的区间[9, 11]的结束值，所以区间[9, 11]和[12, 15]不重叠。这意味着我们可以将区间[12, 15]推入栈中。

**步骤 6**：接下来，我们关注区间[12, 17]。[12, 17]的起始值等于栈顶部的区间[12, 15]的结束值，所以我们不将[12, 17]推入栈中。我们继续比较[12, 15]的结束值和[12, 17]的结束值。由于 15 小于 17，我们将区间[12, 15]更新为[12, 17]。所以，这里我们将[12, 15]与[12, 17]合并。

**步骤 7**：最后，我们关注区间[16, 18]。[16, 18]的起始值小于栈顶部的区间[12, 17]的结束值，所以区间[16, 18]和[12, 17]重叠。这时，我们需要使用[16, 18]的结束值和栈顶部区间的结束值之间的最大值来更新栈顶部的区间的结束值。由于 18 大于 17，栈顶部的区间变为[12, 17]。

现在，我们可以弹出栈的内容来查看合并后的区间，[[12, 18], [9, 11], [1, 7]]，如下图所示：

![图 10.28 – 合并后的区间](img/Figure_10.28_B15403.jpg)

图 10.28 – 合并后的区间

基于这些步骤，我们可以创建以下算法：

1.  根据起始值对给定的区间进行升序排序。

1.  将第一个区间推入栈中。

1.  对于剩下的区间，执行以下操作：

a. 如果当前区间与栈顶部的区间不重叠，则将其推入栈中。

b. 如果当前区间与栈顶部的区间重叠，并且当前区间的结束值大于栈顶部的结束值，则使用当前区间的结束值更新栈顶部的结束值。

1.  最后，栈中包含了合并后的区间。

在代码方面，该算法如下所示：

```java
public static void mergeIntervals(Interval[] intervals) {
  // Step 1
  java.util.Arrays.sort(intervals,
          new Comparator<Interval>() {
    public int compare(Interval i1, Interval i2) {
      return i1.start - i2.start;
    }
  });
  Stack<Interval> stackOfIntervals = new Stack();
  for (Interval interval : intervals) {
    // Step 3a
    if (stackOfIntervals.empty() || interval.start
           > stackOfIntervals.peek().end) {
        stackOfIntervals.push(interval);
    }
    // Step 3b
    if (stackOfIntervals.peek().end < interval.end) {
      stackOfIntervals.peek().end = interval.end;
    }
  }
  // print the result
  while (!stackOfIntervals.empty()) {
    System.out.print(stackOfIntervals.pop() + " ");
  }
}
```

这段代码的运行时间是 O(n log n)，辅助空间为 O(n)用于栈。虽然面试官应该对这种方法满意，但他/她可能会要求你进行优化。更确切地说，我们能否放弃栈并获得 O(1)的复杂度空间？

如果我们放弃栈，那么我们必须在原地执行合并操作。能够做到这一点的算法是不言自明的：

1.  根据它们的开始时间，对给定的区间进行升序排序。

1.  对于剩下的区间，做以下操作：

a. 如果当前区间不是第一个区间，并且与前一个区间重叠，则合并这两个区间。对所有先前的区间执行相同的操作。

b. 否则，将当前区间添加到输出数组中。

注意，这次区间按照它们的开始时间降序排序。这意味着我们可以通过比较前一个区间的开始和当前区间的结束来检查两个区间是否重叠。让我们看看这段代码：

```java
public static void mergeIntervals(Interval intervals[]) {
  // Step 1
  java.util.Arrays.sort(intervals,
        new Comparator<Interval>() {
    public int compare(Interval i1, Interval i2) {
      return i2.start - i1.start;
    }
  });
  int index = 0;
  for (int i = 0; i < intervals.length; i++) {
    // Step 2a
    if (index != 0 && intervals[index - 1].start 
             <= intervals[i].end) {
      while (index != 0 && intervals[index - 1].start 
             <= intervals[i].end) {
        // merge the previous interval with 
        // the current interval  
        intervals[index - 1].end = Math.max(
          intervals[index - 1].end, intervals[i].end);
        intervals[index - 1].start = Math.min(
          intervals[index - 1].start, intervals[i].start);
        index--;
      }
    // Step 2b
    } else {
      intervals[index] = intervals[i];
    }
    index++;
  }
  // print the result        
  for (int i = 0; i < index; i++) {
    System.out.print(intervals[i] + " ");
  }
}
```

这段代码的运行时间是 O(n log n)，辅助空间为 O(1)。完整的应用程序称为*MergeIntervals*。

## 编程挑战 19 – 加油站环形旅游

**亚马逊**，**谷歌**，**Adobe**，**微软**，**Flipkart**

**问题**：考虑到你已经得到了沿着圆形路线的*n*个加油站。每个加油站包含两个数据：燃料量(*fuel*[])和从当前加油站到下一个加油站的距离(*dist*[])。接下来，你有一辆带有无限油箱的卡车。编写一小段代码，计算卡车应该从哪个加油站开始以完成一次完整的旅程。你从一个加油站开始旅程时，油箱是空的。用 1 升汽油，卡车可以行驶 1 单位的距离。

**解决方案**：考虑到你已经得到了以下数据：*dist* = {5, 4, 6, 3, 5, 7}, *fuel* = {3, 3, 5, 5, 6, 8}。

让我们使用以下图像更好地理解这个问题的背景，并支持我们找到解决方案：

![图 10.29 – 卡车环形旅游示例](img/Figure_10.29_B15403.jpg)

图 10.29 – 卡车环形旅游示例

从 0 到 5，我们有六个加油站。在图像的左侧，你可以看到给定圆形路线的草图和加油站的分布。第一个加油站有 3 升汽油，到下一个加油站的距离是 5 单位。第二个加油站有 3 升汽油，到下一个加油站的距离是 4 单位。第三个加油站有 5 升汽油，到下一个加油站的距离是 6 单位，依此类推。显然，如果我们希望从加油站*X*到加油站*Y*，一个重要的条件是*X*和*Y*之间的距离小于或等于卡车油箱中的燃料量。例如，如果卡车从加油站 0 开始旅程，那么它不能去加油站 1，因为这两个加油站之间的距离是 5 单位，而卡车的油箱只能装 3 升汽油。另一方面，如果卡车从加油站 3 开始旅程，那么它可以去加油站 4，因为卡车的油箱里会有 5 升汽油。实际上，如图像的右侧所示，这种情况的解决方案是从加油站 3 开始，油箱里有 5 升汽油 – 用纸和笔芯花点时间完成旅程。

蛮力（或者朴素）方法可以依赖于一个简单的陈述：我们从每个加油站开始，尝试完成整个旅程。这很容易实现，但其运行时间将为 O(n2)。挑战自己想出一个更好的实现。

为了更有效地解决这个问题，我们需要理解和使用以下事实：

+   如果*燃料总量≥距离总量*，则旅程可以完成。

+   如果加油站*X*不能在*X → Y → Z*的顺序中到达加油站*Z*，那么*Y*也不能到达。

第一个要点是常识，第二个要点需要一些额外的证明。以下是第二个要点背后的推理：

如果*fuel*[*X*] *< dist*[*X*]，那么*X*甚至无法到达*Y*。因此，要从*X*到*Z*，*fuel*[*X*]必须*≥ dist*[*X*]。

鉴于*X*无法到达*Z*，我们有*fuel*[*X*] *+ fuel*[*Y*] *< dist*[*X*] *+ dist*[*Y*]*，而*fuel*[*X*] *≥ dist*[*X*]。因此，*fuel*[*Y*] *< dist*[*Y*]，*Y*也无法到达*Z*。

基于这两点，我们可以得出以下实现：

```java
public static int circularTour(int[] fuel, int[] dist) {
  int sumRemainingFuel = 0; // track current remaining fuel
  int totalFuel = 0;        // track total remaining fuel
  int start = 0;
  for (int i = 0; i < fuel.length; i++) {
    int remainingFuel = fuel[i] - dist[i];
    //if sum remaining fuel of (i-1) >= 0 then continue 
    if (sumRemainingFuel >= 0) {
      sumRemainingFuel += remainingFuel;
      //otherwise, reset start index to be current
    } else {
      sumRemainingFuel = remainingFuel;
      start = i;
    }
    totalFuel += remainingFuel;
  }
  if (totalFuel >= 0) {
    return start;
  } else {
    return -1;
  }
}
```

要理解这段代码，可以尝试使用纸和笔将给定的数据通过代码传递。此外，您可能希望尝试以下集合：

```java
// start point 1
int[] dist = {2, 4, 1};
int[] fuel = {0, 4, 3};
// start point 1
int[] dist = {6, 5, 3, 5};
int[] fuel = {4, 6, 7, 4};
// no solution, return -1
int[] dist = {1, 3, 3, 4, 5};
int[] fuel = {1, 2, 3, 4, 5};
// start point 2
int[] dist = {4, 6, 6};
int[] fuel = {6, 3, 7};
```

这段代码的运行时间是 O(n)。完整的应用程序称为*PetrolBunks*。

## 编程挑战 20 - 困住雨水

**亚马逊**，**谷歌**，**Adobe**，**微软**，**Flipkart**

**问题**：假设你已经得到了一组不同高度（非负整数）的酒吧。每个酒吧的宽度等于 1。编写一小段代码，计算可以在酒吧之间困住的水量。

**解决方案**：假设给定的一组酒吧是一个数组，如下所示：*bars* = { 1, 0, 0, 4, 0, 2, 0, 1, 6, 2, 3}。以下图片是这些酒吧高度的草图：

![图 10.30 - 给定的一组酒吧](img/Figure_10.30_B15403.jpg)

图 10.30 - 给定的一组酒吧

现在，雨水在这些酒吧之间的空隙中积水。因此，雨后我们将得到以下情况：

![图 10.31 - 雨后的给定酒吧](img/Figure_10.31_B15403.jpg)

图 10.31 - 雨后的给定酒吧

因此，在这里，我们最多可以获得 16 单位的水。这个问题的解决方案取决于我们如何看待水。例如，我们可以看看酒吧之间的水，或者看看每个酒吧顶部的水。第二种观点正是我们想要的。

查看以下图片，其中有一些关于如何隔离每个酒吧顶部的水的额外指导：

![图 10.32 - 每个酒吧顶部的水](img/Figure_10.32_B15403.jpg)

图 10.32 - 每个酒吧顶部的水

因此，在酒吧 0 上方，我们没有水。在酒吧 1 上方，我们有 1 单位的水。在酒吧 2 上方，我们有 1 单位的水，依此类推。如果我们将这些值相加，那么我们得到 0 + 1 + 1 + 0 + 4 + 2 + 4 + 3 + 0 + 1 + 0 = 16，这就是我们拥有的水的精确数量。但是，要确定酒吧*x*顶部的水量，我们必须知道左右两侧最高酒吧之间的最小值。换句话说，对于每个酒吧，即 1、2、3...9（注意我们不使用酒吧 0 和 10，因为它们是边界），我们必须确定左右两侧最高酒吧，并计算它们之间的最小值。以下图片展示了我们的计算（中间的酒吧范围从 1 到 9）：

![图 10.33 - 左右两侧最高的酒吧](img/Figure_10.33_B15403.jpg)

图 10.33 - 左右两侧最高的酒吧

因此，我们可以得出一个简单的解决方案，即遍历酒吧以找到左右两侧的最高酒吧。这两个酒吧的最小值可以被利用如下：

+   如果最小值小于当前酒吧的高度，则当前酒吧无法在其顶部容纳水。

+   如果最小值大于当前酒吧的高度，则当前酒吧可以容纳的水量等于最小值与其顶部的当前酒吧高度之间的差值。

因此，这个问题可以通过计算每个酒吧左右两侧的最高酒吧来解决。这些陈述的有效实现包括在 O(n)时间内预先计算每个酒吧左右两侧的最高酒吧。然后，我们需要使用结果来找到每个酒吧顶部的水量。以下代码应该澄清任何其他细节：

```java
public static int trap(int[] bars) {
  int n = bars.length - 1;
  int water = 0;
  // store the maximum height of a bar to 
  // the left of the current bar
  int[] left = new int[n];
  left[0] = Integer.MIN_VALUE;
  // iterate the bars from left to right and 
  // compute each left[i]
  for (int i = 1; i < n; i++) {
    left[i] = Math.max(left[i - 1], bars[i - 1]);
  }
  // store the maximum height of a bar to the 
  // right of the current bar
  int right = Integer.MIN_VALUE;
  // iterate the bars from right to left 
  // and compute the trapped water
  for (int i = n - 1; i >= 1; i--) {
    right = Math.max(right, bars[i + 1]);
    // check if it is possible to store water 
    // in the current bar           
    if (Math.min(left[i], right) > bars[i]) {
      water += Math.min(left[i], right) - bars[i];
    }
  }
  return water;
}
```

这段代码的运行时间为 O(n)，*left*[]数组的辅助空间为 O(n)。使用基于堆栈的实现也可以获得类似的大 O。那么如何编写一个具有 O(1)空间的实现呢？

好吧，我们可以使用两个变量来存储到目前为止的最大值（这种技术称为*双指针*），而不是维护一个大小为*n*的数组来存储所有左侧的最大高度。正如您可能记得的，您在之前的一些编程挑战中观察到了这一点。这两个指针是`maxBarLeft`和`maxBarRight`。实现如下：

```java
public static int trap(int[] bars) {
  // take two pointers: left and right pointing 
  // to 0 and bars.length-1        
  int left = 0;
  int right = bars.length - 1;
  int water = 0;
  int maxBarLeft = bars[left];
  int maxBarRight = bars[right];
  while (left < right) {
    // move left pointer to the right
    if (bars[left] <= bars[right]) {
      left++;
      maxBarLeft = Math.max(maxBarLeft, bars[left]);
      water += (maxBarLeft - bars[left]);
    // move right pointer to the left
    } else {
      right--;
      maxBarRight = Math.max(maxBarRight, bars[right]);
      water += (maxBarRight - bars[right]);
    }
  }
  return water;
}
```

这段代码的运行时间为 O(n)，空间复杂度为 O(1)。完整的应用程序称为*TrapRainWater*。

## 编程挑战 21 - 购买和出售股票

**亚马逊**，**微软**

**问题**：假设您已经得到了一个表示每天股票价格的正整数数组。因此，数组的第 i 个元素表示第 i 天的股票价格。通常情况下，您可能不会同时进行多次交易（买卖序列称为一次交易），并且必须在再次购买之前出售股票。编写一小段代码，在以下情况中返回最大利润（通常情况下，面试官会给您以下情况中的一个）：

+   您只允许买卖股票一次。

+   您只允许买卖股票两次。

+   您可以无限次地买卖股票。

+   您只允许买卖股票* k*次（* k*已知）。

**解决方案**：假设给定的价格数组为*prices*={200, 500, 1000, 700, 30, 400, 900, 400, 550}。让我们分别解决上述情况。

### 只买卖一次股票

在这种情况下，我们必须通过只买卖一次股票来获得最大利润。这是非常简单和直观的。想法是在股票最便宜时买入，在最昂贵时卖出。让我们通过以下价格趋势图来确认这一说法：

![图 10.34 - 价格趋势图](img/Figure_10.34_B15403.jpg)

图 10.34 - 价格趋势图

根据上图，我们应该在第 5 天以 30 的价格买入股票，并在第 7 天以 900 的价格卖出。这样，利润将达到最大值（870）。为了确定最大利润，我们可以采用一个简单的算法，如下所示：

1.  考虑第 1 天的*最低价格*，没有利润（*最大利润*为 0）。

1.  迭代剩余的天数（2、3、4、...）并执行以下操作：

a. 对于每一天，将*最大利润*更新为 max(*当前最大利润，（今天的价格 - 最低价格）*)。

b. 将*最低价格*更新为 min(*当前最低价格，今天的价格*)。

让我们将这个算法应用到我们的数据中。因此，我们将第 1 天的*最低价格*视为 200，*最大利润*为 0。下图显示了每天的计算：

![图 10.35 - 计算最大利润](img/Figure_10.35_B15403.jpg)

图 10.35 - 计算最大利润

**第 1 天**：*最低价格*为 200；第 1 天的价格 - 最低价格 = 0；因此，到目前为止*最大利润*为 200。

**第 2 天**：*最低价格*为 200（因为 500 > 200）；第 2 天的价格 - 最低价格 = 300；因此，到目前为止*最大利润*为 300（因为 300 > 200）。

**第 3 天**：*最低价格*为 200（因为 1000 > 200）；第 3 天的价格 - 最低价格 = 800；因此，到目前为止*最大利润*为 800（因为 800 > 300）。

**第 4 天**：*最低价格*为 200（因为 700 > 200）；第 4 天的价格 - 最低价格 = 500；因此，到目前为止*最大利润*为 800（因为 800 > 500）。

**第 5 天**：*最低价格*为 30（因为 200 > 30）；第 5 天的价格 - 最低价格 = 0；因此，到目前为止*最大利润*为 800（因为 800 > 0）。

**第 6 天**：*最低价格*是 30（因为 400 > 30）；第 6 天的价格 - 最低价格 = 370；因此，到目前为止*最大利润*是 800（因为 800 > 370）。

**第 7 天**：*最低价格*是 30（因为 900 > 30）；第 7 天的价格 - 最低价格 = 870；因此，到目前为止*最大利润*是 870（因为 870 > 800）。

**第 8 天**：*最低价格*是 30（因为 400 > 30）；第 8 天的价格 - 最低价格 = 370；因此，到目前为止*最大利润*是 870（因为 870 > 370）。

**第 9 天**：*最低价格*是 30（因为 550 > 30）；第 9 天的价格 - 最低价格 = 520；因此，到目前为止*最大利润*是 870（因为 870 >520）。

最后，*最大利润*是 870。

让我们看看代码：

```java
public static int maxProfitOneTransaction(int[] prices) {
  int min = prices[0];
  int result = 0;
  for (int i = 1; i < prices.length; i++) {
    result = Math.max(result, prices[i] - min);
    min = Math.min(min, prices[i]);
  }
  return result;
}
```

这段代码的运行时间是 O(n)。让我们来解决下一个情景。

### 只买卖股票两次

在这种情况下，我们必须通过只买卖股票两次来获得最大利润。想法是在股票最便宜时买入，最昂贵时卖出。我们这样做两次。让我们通过以下价格趋势图来识别这个陈述：

![图 10.36 - 价格趋势图](img/Figure_10.36_B15403.jpg)

图 10.36 - 价格趋势图

根据前面的图表，我们应该在第 1 天以 200 的价格买入股票，然后在第 3 天以 1000 的价格卖出。这笔交易带来了 800 的利润。接下来，我们应该在第 5 天以 30 的价格买入股票，然后在第 7 天以 900 的价格卖出。这笔交易带来了 870 的利润。因此，最大利润是 870+800=1670。

要确定*最大利润*，我们必须找到两笔最有利可图的交易。我们可以通过动态规划和*分治*技术来实现这一点。我们将算法*分*成两部分。算法的第一部分包含以下步骤：

1.  考虑第 1 天的*最便宜的价格*。

1.  迭代剩下的天数（2，3，4，...）并执行以下操作：

a. 更新*最便宜的价格*，作为 min（*当前最便宜的价格*，今天的价格*）。

b. 跟踪今天的*最大利润*，作为 max（*前一天的最大利润*，（*今天的价格 - 最便宜的价格*））。

在这个算法结束时，我们将得到一个数组（让我们称之为*left*[]），表示每天（包括当天）之前可以获得的最大利润。例如，直到第 3 天（包括第 3 天），最大利润是 800，因为你可以在第 1 天以 200 的价格买入，第 3 天以 1000 的价格卖出，或者直到第 7 天（包括第 7 天），最大利润是 870，因为你可以在第 5 天以 30 的价格买入，第 7 天以 900 的价格卖出，依此类推。

这个数组是通过*步骤 2b*获得的。我们可以将它表示为我们的数据如下：

![图 10.37 - 从第 1 天开始计算每天之前的最大利润](img/Figure_10.37_B15403.jpg)

图 10.37 - 从第 1 天开始计算每天之前的最大利润

*left*[]数组在我们覆盖算法的第二部分之后非常有用。接下来，算法的第二部分如下：

1.  考虑最后一天的*最昂贵的价格*。

1.  从（*最后*-1）到*第一*天（*最后-1，最后-2，最后-3，...*）迭代剩下的天数，并执行以下操作：

a. 更新*最昂贵的价格*，作为 max（*当前最昂贵的价格*，今天的价格*）。

b. 跟踪今天的*最大利润*，作为 max（*下一天的最大利润*，（最昂贵的价格 - 今天的价格*））。

在这个算法结束时，我们将得到一个数组（让我们称之为*right*[]），表示每天（包括当天）之后可以获得的最大利润。例如，第 3 天之后（包括第 3 天），最大利润是 870，因为你可以在第 5 天以 30 的价格买入，第 7 天以 900 的价格卖出，或者第 7 天之后最大利润是 150，因为你可以在第 8 天以 400 的价格买入，第 9 天以 550 的价格卖出，依此类推。这个数组是通过*步骤 2b*获得的。我们可以将它表示为我们的数据如下：

![图 10.38 - 从前一天开始计算每天的最大利润](img/Figure_10.38_B15403.jpg)

图 10.38 - 从前一天开始计算每天的最大利润

到目前为止，我们已经完成了*分割*部分。现在，是*征服*部分的时间了。可以通过 max(*left*[*day*]*+right*[*day*])获得可以在两次交易中实现的*最大利润*。我们可以在下图中看到这一点：

![图 10.39 - 计算第 1 和第 2 次交易的最终最大利润](img/Figure_10.39_B15403.jpg)

图 10.39 - 计算第 1 和第 2 次交易的最终最大利润

现在，让我们来看代码：

```java
public static int maxProfitTwoTransactions(int[] prices) {
  int[] left = new int[prices.length];
  int[] right = new int[prices.length];
  // Dynamic Programming from left to right
  left[0] = 0;
  int min = prices[0];
  for (int i = 1; i < prices.length; i++) {
    min = Math.min(min, prices[i]);
    left[i] = Math.max(left[i - 1], prices[i] - min);
  }
  // Dynamic Programming from right to left
  right[prices.length - 1] = 0;
  int max = prices[prices.length - 1];
  for (int i = prices.length - 2; i >= 0; i--) {
    max = Math.max(max, prices[i]);
    right[i] = Math.max(right[i + 1], max - prices[i]);
  }
  int result = 0;
  for (int i = 0; i < prices.length; i++) {
    result = Math.max(result, left[i] + right[i]);
  }
  return result;
}
```

这段代码的运行时间是 O(n)。现在，让我们来处理下一个情景。

### 买卖股票的次数不限

在这种情况下，我们必须通过买卖股票不限次数来获得最大利润。您可以通过以下价格趋势图来确定这一点：

![图 10.40 - 价格趋势图](img/Figure_10.40_B15403.jpg)

图 10.40 - 价格趋势图

根据前面的图表，我们应该在第 1 天以 200 的价格买入股票，然后在第 2 天以 500 的价格卖出。这次交易带来了 300 的利润。接下来，我们应该在第 2 天以 500 的价格买入股票，然后在第 3 天以 1000 的价格卖出。这次交易带来了 500 的利润。当然，我们可以将这两次交易合并为一次，即在第 1 天以 200 的价格买入，然后在第 3 天以 1000 的价格卖出。同样的逻辑可以应用到第 9 天。最终的最大利润将是 1820。花点时间，确定从第 1 天到第 9 天的所有交易。

通过研究前面的价格趋势图，我们可以看到这个问题可以被视为尝试找到所有的升序序列。以下图突出显示了我们数据的升序序列：

![图 10.41 - 升序序列](img/Figure_10.41_B15403.jpg)

图 10.41 - 升序序列

根据以下算法，找到所有的升序序列是一个简单的任务：

1.  将*最大利润*视为 0（无利润）。

1.  迭代所有的天数，从第 2 天开始，并执行以下操作：

a. 计算*今日价格*和*前一天价格*之间的差异（例如，在第一次迭代中，计算（第 2 天的价格 - 第 1 天的价格），所以 500 - 200）。

b. 如果计算出的差异为正数，则将*最大利润*增加这个差异。

在这个算法结束时，我们将知道最终的*最大利润*。如果我们将这个算法应用到我们的数据中，那么我们将得到以下输出：

![图 10.42 - 计算最终最大利润](img/Figure_10.42_B15403.jpg)

图 10.42 - 计算最终最大利润

**第 1 天**：*最大利润*为 0。

**第 2 天**：*最大利润*为 0 + (500 - 200) = 0 + 300 = 300。

**第 3 天**：*最大利润*为 300 + (1000 - 500) = 300 + 500 = 800。

**第 4 天**：*最大利润*仍为 800，因为 700 - 1000 < 0。

**第 5 天**：*最大利润*仍为 800，因为 30 - 700 < 0。

**第 6 天**：*最大利润*为 800 + (400 - 30) = 800 + 370 = 1170。

**第 7 天**：*最大利润*为 1170 + (900 - 400) = 1170 + 500 = 1670。

**第 8 天**：*最大利润*仍为 1670，因为 400 - 900 < 0。

**第 9 天**：*最大利润*为 1670 + (550 - 400) = 1670 + 150 = 1820。

最终的*最大利润*为 1820。

在代码方面，情况如下：

```java
public static int maxProfitUnlimitedTransactions(
          int[] prices) {
  int result = 0;
  for (int i = 1; i < prices.length; i++) {
    int diff = prices[i] - prices[i - 1];
    if (diff > 0) {               
      result += diff;
    }
  }
  return result;
}
```

这段代码的运行时间是 O(n)。接下来，让我们来处理最后一个情景。

### 只买卖股票 k 次（给定 k）

这种情况是*只买卖股票两次*的一般化版本。主要是，通过解决这种情况，我们也解决了*k*=2 时的*只买卖股票两次*情况。 

根据我们从之前情景中的经验，我们知道解决这个问题可以通过动态规划来完成。更确切地说，我们需要跟踪两个数组：

+   第一个数组将跟踪在第*q*天进行最后一笔交易时*p*次交易的*最大利润*。

+   第二个数组将跟踪在第*q*天之前*p*次交易的*最大利润*。

如果我们将第一个数组表示为`temp`，第二个数组表示为`result`，那么我们有以下两个关系：

1.  ```java
    temp[p] = Math.max(result[p - 1] 
                + Math.max(diff, 0), temp[p] + diff);
    ```

```java
result[p] = Math.max(temp[p], result[p]);
```

为了更好地理解，让我们将这些关系放入代码的上下文中：

```java
public static int maxProfitKTransactions(
          int[] prices, int k) {
  int[] temp = new int[k + 1];
  int[] result = new int[k + 1];
  for (int q = 0; q < prices.length - 1; q++) {
    int diff = prices[q + 1] - prices[q];
    for (int p = k; p >= 1; p--) {
      temp[p] = Math.max(result[p - 1] 
              + Math.max(diff, 0), temp[p] + diff);
      result[p] = Math.max(temp[p], result[p]);
     }
  }
  return result[k];
}
```

这段代码的运行时间是 O(kn)。完整的应用程序称为*BestTimeToBuySellStock*。

## 编码挑战 22-最长序列

**亚马逊**，**Adobe**，**微软**

**问题**：考虑到你已经得到了一个整数数组。编写一小段代码，找到最长的整数序列。注意，序列只包含连续不同的元素。给定数组中元素的顺序并不重要。

**解决方案**：假设给定数组是{4, 2, 9, 5, 12, 6, 8}。最长序列包含三个元素，由 4、5 和 6 组成。或者，如果给定数组是{2, 0, 6, 1, 4, 3, 8}，那么最长序列包含五个元素，由 2、0、1、4 和 3 组成。再次注意，给定数组中元素的顺序并不重要。

蛮力或朴素方法包括对数组进行升序排序，并找到最长的连续整数序列。由于数组已排序，间隙会打破序列。然而，这样的实现将具有 O(n log n)的运行时间。

更好的方法是使用*哈希*技术。让我们使用以下图像来支持我们的解决方案：

![图 10.43-序列集](img/Figure_10.43_B15403.jpg)

图 10.43-序列集

首先，我们从给定数组{4, 2, 9, 5, 12, 6, 8}构建一个集合。如前面的图像所示，集合不保持插入顺序，但这对我们来说并不重要。接下来，我们遍历给定数组，并对于每个遍历的元素（我们将其表示为*e*），我们搜索*e*-1 的集合。例如，当我们遍历 4 时，我们搜索 3 的集合，当我们遍历 2 时，我们搜索 1，依此类推。如果*e-*1 不在集合中，那么我们可以说*e*代表连续整数新序列的开始（在这种情况下，我们有以 12、8、4 和 2 开头的序列）；否则，它已经是现有序列的一部分。当我们有新序列的开始时，我们继续搜索连续元素的集合：*e*+1、*e*+2、*e*+3 等等。只要我们找到连续元素，我们就计数它们。如果找不到*e+*i*（1、2、3、...），那么当前序列就完成了，我们知道它的长度。最后，我们将这个长度与迄今为止找到的最长长度进行比较，并相应地进行下一步。

这段代码非常简单：

```java
public static int findLongestConsecutive(int[] sequence) {
  // construct a set from the given sequence
  Set<Integer> sequenceSet = IntStream.of(sequence)
    .boxed()
    .collect(Collectors.toSet());
  int longestSequence = 1;
  for (int elem : sequence) {
    // if 'elem-1' is not in the set then     // start a new sequence
    if (!sequenceSet.contains(elem - 1)) {
      int sequenceLength = 1;
      // lookup in the set for elements 
      // 'elem + 1', 'elem + 2', 'elem + 3' ...
      while (sequenceSet.contains(elem + sequenceLength)) {
        sequenceLength++;
      }
      // update the longest consecutive subsequence
      longestSequence = Math.max(
        longestSequence, sequenceLength);
    }
  }
  return longestSequence;
}
```

这段代码的运行时间是 O(n)，辅助空间是 O(n)。挑战自己并打印最长的序列。完整的应用程序称为*LongestConsecutiveSequence*。

## 编码挑战 23-计分游戏

**亚马逊**，**谷歌**，**微软**

**问题**：考虑一个游戏，玩家可以在单次移动中得分 3、5 或 10 分。此外，考虑到你已经得到了一个总分*n*。编写一小段代码，返回达到这个分数的方法数。

**解决方案**：假设给定的分数是 33。有七种方法可以达到这个分数：

(10+10+10+3) = 33

(5+5+10+10+3) = 33

(5+5+5+5+10+3) = 33

(5+5+5+5+5+5+3) = 33

(3+3+3+3+3+3+3+3+3+3+3) = 33

(3+3+3+3+3+3+5+5+5) = 33

(3+3+3+3+3+3+5+10) = 33

我们可以借助动态规划来解决这个问题。我们创建一个大小等于*n*+1 的表（数组）。在这个表中，我们存储从 0 到*n*的所有分数的计数。对于移动 3、5 和 10，我们增加数组中的值。代码说明了一切：

```java
public static int count(int n) {
  int[] table = new int[n + 1];
  table[0] = 1;
  for (int i = 3; i <= n; i++) {
    table[i] += table[i - 3];
  }
  for (int i = 5; i <= n; i++) {
    table[i] += table[i - 5];
  }
  for (int i = 10; i <= n; i++) {
    table[i] += table[i - 10];
  }
  return table[n];
}
```

这段代码的运行时间是 O(n)，额外空间是 O(n)。完整的应用程序称为*CountScore3510*。

## 编码挑战 24-检查重复项

**亚马逊**，**谷歌**，**Adobe**

如果这个数组包含重复项，则返回`true`。

**解决方案**：假设给定的整数是*arr*={1, 4, 5, 4, 2, 3}，所以 4 是重复的。蛮力方法（或者朴素方法）将依赖嵌套循环，如下面的简单代码所示：

```java
public static boolean checkDuplicates(int[] arr) {
  for (int i = 0; i < arr.length; i++) {
    for (int j = i + 1; j < arr.length; j++) {
      if (arr[i] == arr[j]) {
        return true;
      }
    }
  }
  return false;
}
```

这段代码非常简单，但是它的时间复杂度是 O(n2)，辅助空间复杂度是 O(1)。我们可以在检查重复项之前对数组进行排序。如果数组已经排序，那么我们可以比较相邻的元素。如果任何相邻的元素相等，我们可以说数组包含重复项：

```java
public static boolean checkDuplicates(int[] arr) {
  java.util.Arrays.sort(arr);
  int prev = arr[0];
  for (int i = 1; i < arr.length; i++) {
    if (arr[i] == prev) {
      return true;
    }
    prev = arr[i];
  }
  return false;
}
```

这段代码的时间复杂度是 O(n log n)（因为我们对数组进行了排序），辅助空间复杂度是 O(1)。如果我们想要编写一个时间复杂度为 O(n)的实现，我们还必须考虑辅助空间复杂度为 O(n)。例如，我们可以依赖*哈希*（如果您不熟悉哈希的概念，请阅读*第六章**，面向对象编程*，*哈希表*问题）。在 Java 中，我们可以通过内置的`HashSet`实现来使用哈希，因此无需从头开始编写哈希实现。但是`HashSet`有什么用呢？当我们遍历给定数组时，我们将数组中的每个元素添加到`HashSet`中。但是如果当前元素已经存在于`HashSet`中，这意味着我们找到了重复项，所以我们可以停止并返回：

```java
public static boolean checkDuplicates(int[] arr) {
  Set<Integer> set = new HashSet<>();
  for (int i = 0; i < arr.length; i++) {
    if (set.contains(arr[i])) {
      return true;
    }

    set.add(arr[i]);
  }
  return false;
}
```

因此，这段代码的时间复杂度是 O(n)，辅助空间复杂度是 O(n)。但是，如果我们记住`HashSet`不接受重复项，我们可以简化上述代码。换句话说，如果我们将给定数组的所有元素插入`HashSet`，并且这个数组包含重复项，那么`HashSet`的大小将与数组的大小不同。这个实现和一个基于 Java 8 的实现，具有 O(n)的运行时间和 O(n)的辅助空间，可以在本书附带的代码中找到。

如何实现具有 O(n)的运行时间和 O(1)的辅助空间？如果我们考虑给定数组的两个重要约束，这是可能的：

+   给定的数组不包含负数元素。

+   元素位于[0，*n*-1]的范围内，其中*n=arr.length*。

在这两个约束的保护下，我们可以使用以下算法。

1.  我们遍历给定的数组，对于每个*arr*[*i*]，我们执行以下操作：

a. 如果*arr*[abs(*arr*[*i*])]大于 0，则将其变为负数。

b. 如果*arr*[abs(*arr*[*i*])]等于 0，则将其变为-(*arr.length*-1)。

c. 否则，我们返回`true`（有重复项）。

让我们考虑我们的数组*arr*={1, 4, 5, 4, 2, 3}，并应用上述算法：

+   *i*=0，因为*arr*[abs(*arr*[0])] = *arr*[1] = 4 > 0 导致*arr*[1] = -*arr*[1] = -4。

+   *i*=1，因为*arr*[abs(*arr*[1])] = *arr*[4] = 2 > 0 导致*arr*[4] = -*arr*[4] = -2。

+   *i*=2，因为*arr*[abs(*arr*[5])] = *arr*[5] = 3 > 0 导致*arr*[5] = -*arr*[5] = -3。

+   *i*=3，因为*arr*[abs(*arr*[4])] = *arr*[4] = -2 < 0 返回`true`（我们找到了重复项）。

现在，让我们看看*arr*={1, 4, 5, 3, 0, 2, 0}：

+   *i*=0，因为*arr*[abs(*arr*[0])] = *arr*[1] = 4 > 0 导致*arr*[1] = -*arr*[1] = -4。

+   *i*=1，因为*arr*[abs(*arr*[1])] = *arr*[4] = 0 = 0 导致*arr*[4] = -(*arr.length*-1) = -6。

+   *i*=2，因为*arr*[abs(*arr*[2])] = *arr*[5] = 2 > 0 导致*arr*[5] = -*arr*[5] = -2。

+   *i*=3，因为*arr*[abs(*arr*[3])] = *arr*[3] = 3 > 0 导致*arr*[3] = -*arr*[3] = -3。

+   *i*=4，因为*arr*[abs(*arr*[4])] = *arr*[6] = 0 = 0 导致*arr*[6] = -(*arr.length*-1) = -6。

+   *i*=5，因为*arr*[abs(*arr*[5])] = *arr*[2] = 5 > 0 导致*arr*[2] = -*arr*[2] = -5。

+   *i*=6，因为*arr*[abs(*arr*[6])] = *arr*[6] = -6 < 0 返回`true`（我们找到了重复项）。

让我们把这个算法写成代码：

```java
public static boolean checkDuplicates(int[] arr) {
  for (int i = 0; i < arr.length; i++) {
    if (arr[Math.abs(arr[i])] > 0) {
      arr[Math.abs(arr[i])] = -arr[Math.abs(arr[i])];
    } else if (arr[Math.abs(arr[i])] == 0) {
      arr[Math.abs(arr[i])] = -(arr.length-1);
    } else {
      return true;
    }
  }
  return false;
}
```

完整的应用程序称为*DuplicatesInArray*。

对于接下来的五个编码挑战，您可以在本书附带的代码中找到解决方案。花点时间，挑战自己在查看附带代码之前想出一个解决方案。

## 编码挑战 25 - 最长不同子串

**问题**：假设你已经得到了一个字符串*str*。*str*的接受字符属于扩展 ASCII 表（256 个字符）。编写一小段代码，找到包含不同字符的*str*的最长子串。

**解决方案**：作为提示，使用*滑动窗口*技术。如果您对这种技术不熟悉，请考虑在继续之前阅读 Zengrui Wang 的*滑动窗口技术*（[`medium.com/@zengruiwang/sliding-window-technique-360d840d5740`](https://medium.com/@zengruiwang/sliding-window-technique-360d840d5740)）。完整的应用程序称为*LongestDistinctSubstring*。您可以访问以下链接检查代码：[`github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter10/LongestDistinctSubstring`](https://github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter10/LongestDistinctSubstring)

## 编码挑战 26-用排名替换元素

**问题**：假设你已经得到了一个没有重复元素的数组*m*。编写一小段代码，用数组的排名替换每个元素。数组中的最小元素排名为 1，第二小的排名为 2，依此类推。

`TreeMap`。完整的应用程序称为*ReplaceElementWithRank*。您可以访问以下链接检查代码：[`github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter10/ReplaceElementWithRank`](https://github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter10/ReplaceElementWithRank)

## 编码挑战 27-每个子数组中的不同元素

**问题**：假设你已经得到了一个数组*m*和一个整数*n*。编写一小段代码，计算大小为*n*的每个子数组中不同元素的数量。

`HashMap`用于存储当前窗口（大小为*n*）中元素的频率。完整的应用程序称为*CountDistinctInSubarray*。您可以访问以下链接检查代码：[`github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter10/CountDistinctInSubarray`](https://github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter10/CountDistinctInSubarray)

## 编码挑战 28-将数组旋转 k 次

**问题**：假设你已经得到了一个数组*m*和一个整数*k*。编写一小段代码，将数组向右旋转*k*次（例如，数组{1,2,3,4,5}，旋转三次后结果为{3,4,5,1,2}）。

**解决方案**：作为提示，依赖于取模（%）运算符。完整的应用程序称为*RotateArrayKTimes*。您可以访问以下链接检查代码：[`github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter10/RotateArrayKTimes`](https://github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter10/RotateArrayKTimes)。

## 编码挑战 29-已排序数组中的不同绝对值

**问题**：假设你已经得到了一个已排序的整数数组*m*。编写一小段代码，计算不同的绝对值（例如，-1 和 1 被视为一个值）。

**解决方案**：作为提示，使用*滑动窗口*技术。如果您对这种技术不熟悉，可以考虑在继续之前阅读 Zengrui Wang 的*滑动窗口技术*（[`medium.com/@zengruiwang/sliding-window-technique-360d840d5740`](https://medium.com/@zengruiwang/sliding-window-technique-360d840d5740)）。完整的应用程序称为*CountDistinctAbsoluteSortedArray*。您可以访问以下链接检查代码：[`github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter10/CountDistinctAbsoluteSortedArray`](https://github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter10/CountDistinctAbsoluteSortedArray)

# 摘要

本章的目标是帮助您掌握涉及字符串和/或数组的各种编码挑战。希望本章的编码挑战提供了各种技术和技能，这些技能将在许多属于这一类别的编码挑战中非常有用。不要忘记，您可以通过 Packt 出版的书籍*Java 编码问题*（[`www.amazon.com/gp/product/1789801419/`](https://www.amazon.com/gp/product/1789801419/)）进一步丰富您的技能。*Java 编码问题*包含 35 个以上的字符串和数组问题，这些问题在本书中没有涉及。

在下一章中，我们将讨论链表和映射。
