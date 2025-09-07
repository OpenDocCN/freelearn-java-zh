

# 第七章：字符串对象

为了分析我们 Java 应用程序的各个方面，确保它们以高度高效的速率运行，我们需要考虑字符串对象。字符串是 Java 应用程序的重要组成部分，用于从简单的名称列表到复杂的数据库存储需求等无限用途。这些对象的创建、操作和管理应该是首要关注的问题。

本章重点介绍在 Java 应用程序中高效使用字符串对象。第一个概念是合适的字符串池化。我们将探讨这个概念，并探讨使用字符串池化以实现高性能的最佳实践。本章还介绍了延迟初始化的概念，分析了其优势，并通过示例代码进行说明。最后，我们将探讨额外的字符串操作策略，包括高级字符串操作技术。

在本章中，我们将涵盖以下主要主题：

+   合适的字符串池化

+   延迟初始化

+   额外的字符串操作策略

# 技术要求

为了遵循本章中的示例和说明，你需要具备加载、编辑和运行 Java 代码的能力。如果你尚未设置你的开发环境，请参阅*第一章*。

本章的完整代码可以在以下链接找到：[`github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter07`](https://github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter07)。

# 合适的字符串池化

我们的主要关注点是确保我们的 Java 应用程序以高水平运行。为此，内存管理是一个关键问题。我们设计、测试和实施优化内存使用的技术至关重要。**字符串池化**就是这样一种技术，其重点是使字符串对象能够重用，从而提高应用程序的效率。效率的提高源于减少内存开销。

字符串池化是一个重要的概念，它基于将字符串值共享作为创建新字符串实例的替代方案。如果你的应用程序频繁使用字符串字面量，那么你应该会发现字符串池化特别有用。

字符串字面量

字符串字面量是一个由双引号包围的固定值。例如，以下语句中的`"Read more books."`部分是一个字符串字面量：

`System.out.println("Read more books.");`

为了检验字符串池化，我们将从**字符串内部化**的概念开始，然后回顾最佳实践，并通过 Java 的代码示例完成我们的探索。为了进一步深入这一概念，我们将探讨数据库查询中的字符串池化。

## 字符串内部化

字符串池化是一个有趣的概念。它是一种内存减少技术，允许多个字符串使用相同的内存位置。正如你所期望的，这些字符串的内容必须相同才能工作。内存减少是可能的，因为我们能够消除重复的对象。字符串池化依赖于字符串池来实现这一点。正如我们在*第四章*中提到的，池化使用一个特殊的堆区域来存储字符串字面量。

让我们尝试像`intern()`方法一样思考，这是`String`类的一部分，以实现这一点。我们将在本章的*代码示例*部分稍后查看一个示例。

## 最佳实践

通常，被认为在我们的应用程序中使用字符串池化是一个好主意。在实现字符串池化时，有两个重要的考虑因素需要注意。首先，我们不应该过度使用字符串池化。性能和内存使用的益处是明显的，但如果我们的应用程序在非常长的字符串上使用池化，那么我们可能会消耗比我们想要的更多的内存。

另一个值得考虑的最佳实践是避免使用`intern()`方法。这样，我们可以确保我们的应用程序中行为的一致性。

## 代码示例

让我们通过一个简单的示例来演示字符串池化。以下代码首先定义了两个字符串字面量，`s1`和`s2`。这两个字符串具有相同的值。第三个字符串`s3`使用`intern()`方法创建。最后的两个语句用于比较引用：

```java
public class CorgiStringIntern {
    public static void main(String[] args) {
        String s1 = "corgi";
        String s2 = "corgi";
        String s3 = new String("corgi").intern();
        System.out.println("Are s1 and s2 the same object? " + (s1 == 
        s2));
        System.out.println("Are s1 and s3 the same object? " + (s1 == 
        s3));
    }
}
```

如你所见，从我们应用程序的输出中可以看出，两个引用比较都被评估为`true`：

```java
Are s1 and s2 the same object? true
Are s1 and s3 the same object? true
```

## 数据库查询的字符串池化

SQL 和其他数据库查询通常在我们的代码中动态创建，可能会变得相当复杂。与其他大型字符串一样，我们可以使用字符串池来帮助确保我们的应用程序不会不必要地创建字符串对象。我们的目标是重用任何常用的数据库查询，而不是在运行时多次重新创建它们。

# 延迟初始化

不在需要时实例化对象的决定被称为**延迟初始化**。通常，“懒惰”这个词带有负面含义；然而，在软件开发中，延迟初始化是一种性能优化方法，其目标是管理应用程序的开销。这种设计模式在详细处理非常大的对象时特别有用。

我应该实现延迟初始化吗？

如果你拥有大型或复杂的字符串对象，并且需要在初始化时消耗大量开销，那么你应该考虑延迟初始化。

延迟初始化不会减少创建的字符串对象数量；相反，它将初始化延迟到应用程序需要处理时。这种延迟技术可以帮助你进行内存优化。在我们检查源代码之前，让我们先看看一个叙述示例。

想象一下，你有一个使用非常复杂的字符串对象的遗留应用程序。当你的应用程序打开时，会显示一个加载屏幕，并且在幕后，应用程序创建几个字符串对象以支持应用程序的正常操作。用户抱怨应用程序“加载时间过长”，并且当尝试启动应用程序时，计算机经常锁定。在审查源代码后，你意识到在应用程序启动时创建的许多字符串对象仅在用户选择某些功能时被应用程序使用。你的解决方案是实现懒加载，这样这些字符串对象仅在需要时创建。

## 代码示例

现在，让我们看看如何在 Java 中实现懒加载初始化设计模式。我们首先导入`java.util.function.Supplier`。这是一个接口，它提供了一个单一的`get()`方法，我们将使用它来检索一个值：

```java
import java.util.function.Supplier;
```

接下来，我们声明我们的类，并使用`StringBuilder`通过多次调用`append()`方法生成一个复杂字符串：

```java
public class LazyInitializationExample {
  private Supplier<String> lazyString = () -> {
    StringBuilder stringBuilder = new StringBuilder();
    stringBuilder.append("Java is an incredibly ");
    stringBuilder.append("flexible programming language ");
    stringBuilder.append("that is used in a wide range of 
    applications. ");
    stringBuilder.append("This is a complex string ");
    stringBuilder.append("that was lazily initialized.");
    return stringBuilder.toString();
};
```

在`LazyInitializationExample`类内部，我们调用`getLazyString()`方法。此方法用于在需要时创建或检索复杂字符串，而不是在需要之前：

```java
public String getLazyString() {
  return lazyString.get();
}
```

我们代码的最后一部分是`main()`方法。当此方法运行时，我们访问懒加载的字符串：

```java
  public static void main(String[] args) {
    LazyInitializationExample example = new 
    LazyInitializationExample();
    String lazyString = example.getLazyString();
    System.out.println("Lazily initialized string: " + lazyString);
  }
}
```

程序的输出如下：

```java
Lazy-initialized string: Java is an incredibly flexible programming language that is used in a wide range of applications. This is a complex string that was lazily initialized.
```

## 最佳实践

懒加载的概念很简单，决定实现它也很容易。如果你有大型字符串对象，那么你应该尝试这种方法。在懒加载中有两个最佳实践需要考虑。首先，当我们在一个多线程环境中工作时，我们应该实现同步。这是为了帮助促进线程安全。另一个最佳实践是避免过度使用。当我们过度使用懒加载时，我们的代码可能变得难以阅读和维护。

# 额外的字符串操作策略

字符串池和懒加载是优秀的优化策略，可以帮助提高我们 Java 应用程序的整体性能。除了这些策略之外，我们还可以确保我们的字符串连接操作是高效的，我们正确地利用正则表达式，并且我们有效地处理大文本文件。本节回顾了这些领域的各项技术。

## 字符串连接

字符串连接——使用加号（`+`）操作符将两个或多个字符串连接成一个——通常会导致代码效率低下。这种连接会创建一个新的字符串对象，我们希望避免。让我们看看两种提供更好性能的替代方案。

第一种选择使用了`StringBuilder`。在下面的示例中，我们创建了一个`StringBuilder`对象，向其中追加五个字符串字面量，将`StringBuilder`对象转换为`String`，然后输出结果：

```java
public class ConcatenationAlternativeOne {
  public static void main(String[] args) {
    StringBuilder myStringBuilder = new StringBuilder();
    myStringBuilder.append("Java");
    myStringBuilder.append(" is");
    myStringBuilder.append(" my");
    myStringBuilder.append(" dog's");
    myStringBuilder.append(" name.");
    String result = myStringBuilder.toString();
    System.out.println(result);
  }
}
```

另一种替代方案是使用`StringBuffer`。以下程序类似于我们的`StringBuilder`示例，但使用的是`StringBuffer`。如您所见，两种方法都是用相同的方式实现的：

```java
public class ConcatenationAlternativeTwo {
  public static void main(String[] args) {
    StringBuffer myStringBuffer = new StringBuffer();
    myStringBuffer.append("Java");
    myStringBuffer.append(" is");
    myStringBuffer.append(" my");
    myStringBuffer.append(" dog's");
    myStringBuffer.append(" name.");
    String result = myStringBuffer.toString();
    System.out.println(result);
  }
}
```

`StringBuilder`和`StringBuffer`在字符串连接上的区别在于`StringBuffer`提供了线程安全，因此应在多线程环境中使用；否则，`StringBuilder`是使用加号（`+`）运算符进行字符串连接的替代方案。

## 正则表达式

**正则表达式**为我们提供了优秀的模式匹配和字符串操作方法。因为这些表达式可能对处理器和内存资源要求较高，因此学习如何高效使用它们非常重要。我们可以通过仅编译一次模式并按需重用它们来优化正则表达式的使用。

让我们来看一个例子。在我们应用程序的第一部分，我们导入了`Matcher`和`Pattern`类：

```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;
```

接下来，我们建立我们的电子邮件模式正则表达式：

```java
public class GoodRegExExample {
  public static void main(String[] args) {
    String emailRegex = "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]
    {2,}$";
```

以下代码段创建了一个示例电子邮件地址列表。我们稍后将检查这些地址的有效性：

```java
String[] emails = {
  "java@example.com",
  "muzz.acruise@gmail.com",
  "invalid-email",
  "bougie@.com",
  "@example.com",
  "brenda@domain.",
  "edward@domain.co",
  "user@example"
};
```

下一个语句编译了正则表达式模式：

```java
Pattern pattern = Pattern.compile(emailRegex);
```

代码的最后一部分遍历我们列表中的每个电子邮件地址：

```java
    for (String email : emails) {
      Matcher matcher = pattern.matcher(email);
      if (matcher.matches()) {
        System.out.println(email + " is a valid email address.");
      } else {
        System.out.println(email + " is an invalid email address.");
      }
    }
  }
}
```

这个简单的正则表达式实现检查电子邮件地址格式，并演示了如何一次性编译正则表达式模式，并在应用程序的其他地方作为每次执行电子邮件地址验证操作时的有效替代方案使用。

## 大型文本文件

一些 Java 应用程序可以处理非常大的文本文件，甚至长达书籍的文件。一次性加载文件的完整内容是不推荐的。我们的应用程序可能会迅速耗尽可用内存，导致不希望的运行时结果。一种替代方案是使用缓冲方法分段读取文本文件。

以下示例假设本地目录中有一个文本文件。我们使用`BufferedReader`逐行读取：

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
public class LargeTextFileHandlingExample {
  public static void main(String[] args) {
    String filePath = "advanced_guide_to_java.txt";
    try (BufferedReader reader = new BufferedReader(new 
    FileReader(filePath))) {
      String line;
      int lineCount = 0;
      while ((line = reader.readLine()) != null) {
        System.out.println("Line " + (++lineCount) + ": " + line);
      }
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
}
```

这种方法可以帮助我们管理内存，并提升我们应用程序的整体性能。

# 摘要

本章重点介绍了如何创建、操作和管理字符串，以提升您 Java 应用程序的整体性能。您现在应该理解了字符串池，并基于最佳实践有信心有效地使用它。在处理大量字符串使用且关注线程安全时，延迟初始化现在应该是一个您在未来的应用程序中考虑实现的策略。本章还介绍了高级字符串操作策略，以帮助您在设计高性能 Java 应用程序时有所选择。

在下一章“内存泄漏”中，我们将探讨内存泄漏是什么，它们是如何产生的，以及它们对我们应用程序的影响。我们将探讨避免内存泄漏的策略，以提升我们 Java 应用程序的性能。
