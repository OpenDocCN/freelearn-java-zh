# 5. 异常

概述

本章讨论了 Java 中异常的处理方式。你将首先学习如何识别代码中产生异常的情况。这种知识将简化处理这些异常的过程，因为它会提醒你在最有可能出现这些异常的情况下。在此过程中，本章还提供了一份最佳实践列表，指导你通过常见场景和最佳方法来捕获异常或将其抛给调用类，并在执行过程中记录其详细信息。你还将学习区分不同类型的异常，并练习处理每种异常的技术。到本章结束时，你甚至能够创建自己的异常类，能够按照严重程度顺序记录每种类型的异常。

# 简介

异常不是错误，或者更准确地说，异常不是 bug，即使它们可能在你程序崩溃时让你觉得它们是。异常是在你的代码中发生的情况，当处理的数据与用于处理它的方法或命令不匹配时。

在 Java 中，有一个类是专门用于错误的。错误是影响程序在**Java 虚拟机**（**JVM**）层面的意外情况。例如，如果你通过非传统方式使用内存填满程序栈，那么整个 JVM 都会崩溃。与错误不同，异常是当你的代码设计得当，可以即时捕获的情况。

异常并不像错误那样严重，即使对你这个开发者来说结果可能相同——也就是说，一个无法工作的程序。在本章中，我们邀请你通过故意引发你将后来学习如何捕获（即处理）并避免的异常来让你的程序崩溃。根据你如何开发捕获机制，你可以决定是让程序恢复并继续运行，还是优雅地结束其执行，并显示一个人类可读的错误消息。

# 简单异常示例

首先，在你的代码中引发一个简单的异常。首先，在**集成开发环境**（**IDE**）中输入以下程序并执行它：

```java
public class Example01 {
    public static void main(String[] args) {
        // declare a string with nothing inside
        String text = null;
        // you will see this at the console
        System.out.println("Go Java Go!");
        // null'ed strings should crash your program
        System.out.println(text.length());
        // you will never see this print
        System.out.println("done");
    }
}
```

这里是输出：

```java
Go Java Go!
Exception in thread "main" java.lang.NullPointerException
     at Example01.main(Example01.java:11)
Process finished with exit code 1
```

之前的代码列表显示了程序开始执行一个运行良好的命令。在控制台上打印了`Go Java Go!`这句话，但随后出现了一个`NullPointerException`，突出显示发生了异常情况。在这种情况下，我们尝试通过调用`text.length()`来打印一个由 null 初始化的字符串的长度。由于没有长度可以计算（也就是说，我们甚至没有空字符串），`System.out.println()`或`text.length()`触发了异常。此外，在那个点还有一个错误，所以程序退出了，最后的`System.out.println("done")`调用没有执行。你可以尝试将这两个命令分开，看看结果会怎样：

```java
// null'ed strings should crash your program
int number = text.length();
System.out.println(number);
```

这里是输出：

```java
Go Java Go!
Exception in thread "main" java.lang.NullPointerException
     at Example01.main(Example01.java:11)
Process finished with exit code 1
```

如果你检查 IDE 中的行号，你会看到异常发生在我们尝试获取字符串长度的那一行。既然我们已经知道了问题的原因，就有两种方法可以解决这个问题：要么修复数据（注意，有些情况下这是不可能的），要么在我们的代码中包含一种对策来检测异常，然后处理或忽略它们。处理意外事件的行为就是我们所说的捕获异常。另一方面，绕过事件的行为被称为抛出异常。在章节的后面，我们将探讨执行这两种行为的不同方法，以及编写代码处理异常时的良好实践。

然而，在了解如何避免或处理异常之前，让我们再引发一些异常。几乎每个 Java API 都包含了异常的定义，这有助于将错误传播到主类，从而传递给开发者。这样，就可以避免代码在用户面前崩溃的情况。

Java API 涵盖的异常就是我们所说的内置异常。当定义一个类时，你也可以创建自己的异常。谈到类，让我们尝试从一个由`String`实例化的对象中获取一个不存在位置的字符，看看会发生什么：

```java
public class Example02 {
    public static void main(String[] args) {
        // declare a string of a fixed length
        String text = "I <3 bananas"; // 12 characters long
        // provoke an exception
        char character = text.charAt(15); // get the 15th element
        // you will never see this print
        System.out.println("done");
    }
}
```

IDE 将做出以下响应：

```java
Exception in thread "main" java.lang.StringIndexOutOfBoundsException: String index out of range: 15
      at java.lang.String.charAt(String.java:658)
      at Example02.main(Example02.java:8)
Process finished with exit code 1
```

注意，文本变量只有 12 个字符长。当尝试提取第 15 个字符时，IDE 将抛出异常并终止程序。在这种情况下，我们得到了一个名为`StringOutOfBoundsException`的异常。存在许多不同类型的内置异常。

这里是各种异常类型的一个列表：

+   `NullPointerException`

+   `StringOutOfBoundsException`

+   `ArithmeticException`

+   `ClassCastException`

+   `IllegalArgumentException`

+   `IndexOutOfBoundsException`

+   `NumberFormatException`

+   `IllegalAccessException`

+   `InstantiationException`

+   `NoSuchMethodException`

如你所见，不同异常的名称相当具有描述性。当你遇到一个异常时，应该很容易在 Java 文档中找到更多关于它的信息，以便减轻问题。我们将异常分为检查型或非检查型：

+   **检查型异常**：这些在编译期间会被突出显示。换句话说，你的程序将无法完成编译过程，因此你将无法运行它。

+   `NullPointerException`和`StringOutOfBoundsException`都是非检查型。

    为什么有两种类型的异常？

    异常有两种可能性：要么我们作为开发者犯错，没有意识到我们处理数据的方式会产生错误（例如，当我们试图获取空字符串的长度或当我们除以零时），要么错误发生是因为我们对与程序外部交换的数据的性质不确定（例如，从 CLI 获取参数且类型错误）。在第一种情况下，检查异常更有意义。第二种场景是我们需要未检查异常的原因。在这种情况下，我们应该制定策略来处理可能威胁程序正确执行的风险。

创建一个检查异常的例子稍微复杂一些，因为我们必须预测一些直到稍后的章节才会深入介绍的内容。然而，我们认为以下示例，它展示了`IOException`的例子，即使它包含了一些书中尚未涉及到的类，也足够简单：

```java
import java.nio.file.*;
import java.util.*;
public class Example03 {
    public static void main(String[] args) {
        // declare a list that will contain all of the files
        // inside of the readme.txt file
        List<String> lines = Collections.emptyList();
        // provoke an exception
        lines = Files.readAllLines(Paths.get("readme.txt"));
        // you will never see this print
        Iterator<String> iterator = lines.iterator();
        while (iterator.hasNext())
            System.out.println(iterator.next());
    }
}
```

在这个代码列表中最新的是`java.nio.file.*`的使用。这是一个包括用于管理文件等内容的类和方法的 API。这个程序的目标是将整个名为 readme.txt 的文本文件读入一个列表中，然后使用迭代器打印出来，正如我们在*第四章*，*集合、列表和 Java 内置 API*中看到的。

这是一个在调用`Files.readAllLines()`时可能发生检查异常的情况，因为没有文件可读，例如，由于文件名声明错误。IDE 知道这一点，因此它会标记存在潜在风险。

注意 IDE 从我们编写代码的那一刻起就显示警告。此外，当我们尝试编译程序时，IDE 将响应如下：

```java
Error:(11, 35) java: unreported exception java.io.IOException; must be caught or declared to be thrown
```

**捕获**和**抛出**是你可以用来避免异常的两种策略。我们将在本章后面更详细地讨论它们。

# NullPointerException – 不要害怕

我们在之前的一章中介绍了 Java 中的`null`概念。你可能还记得，`null`是在创建对象时隐式分配给对象的值，除非你给它分配不同的值。与`null`相关的是`NullPointerException`值。这是一个非常常见的事件，可能会发生，原因有很多。在本节中，我们将突出一些最常见的场景，以便让你在处理代码中的任何类型异常时有一个不同的思维方式。

在*Example01*中，我们检查了尝试对指向`null`的对象执行操作的过程。让我们看看一些其他可能的案例：

```java
public class Example04 {
    public static void main(String[] args) {
        String vehicleType = null;
        String vehicle = "car";
        if (vehicleType.equals(vehicle)) {
            System.out.println("it's a car");
        } else {
            System.out.println("it's not a car");
        } 
    }
}
```

这个示例的结果将是以下内容：

```java
Exception in thread "main" java.lang.NullPointerException
      at Example04.main(Example04.java:5)
Process finished with exit code 1
```

你本可以防止这个异常，如果你在编写代码时将现有变量与可能为`null`的变量进行比较的话。

```java
public class Example05 {
    public static void main(String[] args) {
        String vehicleType = null;
        String vehicle = "car";
        if (vehicle.equals(vehicleType)) {
            System.out.println("it's a car");
        } else {
            System.out.println("it's not a car");
        }
    }
}
```

上述代码将产生以下结果：

```java
it's not a car
Process finished with exit code 0
```

如你所见，这些示例在概念上没有区别；然而，在代码级别上存在差异。这种差异足以在编译时使你的代码引发异常。这是因为`String`类的`equals()`方法已经准备好处理其参数为`null`的情况。另一方面，初始化为`null`的`String`变量无法访问`equals()`方法。

当尝试从一个初始化为`null`的对象调用非静态方法时，会引发`NullPointerException`的非常常见的情况。以下示例展示了一个包含两个方法的类，你可以调用这些方法来查看它们是否会产生异常。你可以通过简单地注释或取消注释`main()`中调用这些方法的每一行来实现这一点。将代码复制到 IDE 中并尝试两种情况：

```java
public class Example06 {
    private static void staticMethod() {
        System.out.println("static method, accessible from null reference");
    }
    private void nonStaticMethod() {
        System.out.print("non-static method, inaccessible from null reference");
    }
    public static void main(String args[]) {
        Example06 object = null;
        object.staticMethod();
        //object.nonStaticMethod();
    }
}
```

当这种异常出现时，还有其他情况，但让我们专注于如何处理异常。以下章节将描述你可以使用的不同机制来使你的程序能够从意外情况中恢复。

# 捕获异常

如前所述，处理异常有两种方式：捕获和抛出。在本节中，我们将处理这些方法中的第一种。捕获异常需要将可能产生不期望结果的代码封装到特定的语句中，如下面的代码片段所示：

```java
try {
  // code that could generate an exception of the type ExceptionM
} catch (ExceptionM e) {
  // code to be executed in case of exception happening
}
```

我们可以用之前的任何示例来测试这段代码。让我们演示如何停止章节第一个示例中发现的异常，在那个示例中，我们尝试检查一个初始化为`null`的字符串的长度：

```java
public class Example07 {
    public static void main(String[] args) {
        // declare a string with nothing inside
        String text = null;
        // you will see this at the console
        System.out.println("Go Java Go!");
        try {
            // null'ed strings should crash your program
            System.out.println(text.length());
        } catch (NullPointerException ex) {
            System.out.println("Exception: cannot get the text's length");
        }
        // you will now see this print
        System.out.println("done");
    }
}
```

如你所见，我们将可能出错的代码包裹在一个`try-catch`语句中。这段代码列表的结果与我们之前看到的结果非常不同：

```java
Go Java Go!
Exception: cannot get the text's length
done
Process finished with exit code 0
```

主要，我们发现程序直到结束都不会被中断。程序的`try`部分检测到异常的到来，如果异常是`NullPointerException`类型，`catch`部分将执行特定的代码。

可以在`try`调用之后按顺序放置多个`catch`语句，作为检测不同类型异常的一种方式。为了尝试这一点，让我们回到我们尝试打开一个不存在的文件并尝试捕获`readAllLines()`停止程序的例子里：

```java
Example08.java
5  public class Example08 {
6      public static void main(String[] args) {
7          // declare a list that will contain all of the files
8          // inside of the readme.txt file
9          List<String> lines = Collections.emptyList();
10 
11         try {
12             // provoke an exception
13             lines = Files.readAllLines(Paths.get("readme.txt"));
14         } catch (NoSuchFileException fe) {
15             System.out.println("Exception: File Not Found");
16         } catch (IOException ioe) {
17             System.out.println("Exception: IOException");
18         }
https://packt.live/2VU59wh
```

如我们在本章前面所见，我们编写了一个尝试打开一个不存在的文件的程序。我们当时得到的异常是`IOException`。实际上，这个异常是由`NoSuchFileException`触发的，它被升级并触发`IOException`。因此，我们在 IDE 中得到了这个异常。在实现多个`try-catch`语句，如前例所示时，我们得到以下结果：

```java
Exception: File Not Found
Process finished with exit code 0
```

这意味着程序检测到`NoSuchFileException`，因此打印出相应的捕获语句中的消息。然而，如果你想看到由不存在的 readme.txt 文件触发的异常的完整序列，你可以使用一个名为`printStackTrace()`的方法。这将向输出发送程序正确执行过程中的一切。要查看这一点，只需将以下突出显示的更改添加到前面的示例中：

```java
try {
    // provoke an exception
    lines = Files.readAllLines(Paths.get("readme.txt"));
} catch (NoSuchFileException fe) {
    System.out.println("Exception: File Not Found");
    fe.printStackTrace();
} catch (IOException ioe) {
    System.out.println("Exception: IOException");
}
```

程序的输出现在将包括程序执行期间触发的不同异常的完整打印输出。你会看到堆栈输出是倒置的：首先，你会看到程序停止的原因（`NoSuchFileException`），然后它将以引发异常的过程开始的方法结束（`readAllLines`）。这是由于异常构建的方式。正如我们稍后将要讨论的，有许多不同类型的异常。每一种类型都被定义为异常类，这些类可以由几个其他异常子类扩展。如果发生某种类型的扩展，那么它所扩展的类也会在打印堆栈时出现。在我们的例子中，`NoSuchFileException`是`IOException`的子类。

注意

根据你的操作系统，处理打开文件的不同嵌套异常可能被称为不同的名称。

我们已经捕获了两种不同的异常——一个嵌套在另一个内部。也应该能够处理来自不同类别的异常，例如`IOException`和`NullPointerException`。以下示例演示了如何做到这一点。如果你正在处理不是彼此子类的异常，你可以使用逻辑或运算符来捕获这两个异常：

```java
import java.io.*;•
import java.nio.file.*;
import java.util.*;
public class Example09 {
    public static void main(String[] args) {
        List<String> lines = Collections.emptyList();
        try {
            lines = Files.readAllLines(Paths.get("readme.txt"));
        } catch (NullPointerException|IOException ex) {
            System.out.println("Exception: File Not Found or NullPointer");
            ex.printStackTrace();
        }
        // you will never see this print
        Iterator<String> iterator = lines.iterator();
        while (iterator.hasNext())
            System.out.println(iterator.next());
    }
}
```

如你所见，你可以在单个`catch`语句中处理这两个异常。然而，如果你想以不同的方式处理异常，你必须与包含异常信息的对象一起工作，在这个例子中是`ex`。你需要区分你可能同时处理的异常的关键字是`instanceof`，如下面前面示例的修改所示：

```java
try {
    // provoke an exception
    lines = Files.readAllLines(Paths.get("readme.txt"));
} catch (NullPointerException|IOException ex) {
    if (ex instanceof IOException) {
        System.out.println("Exception: File Not Found");
    }
    if (ex instanceof NullPointerException) {
        System.out.println("Exception: NullPointer");
    }
}
```

在一个单独的`try`块中你能捕获多少种不同的异常？

事实上，你可以根据需要将尽可能多的`catch`语句链接起来。如果你使用本章讨论的第二种方法（即使用 OR 语句），你应该记住，不可能同时有一个子类及其父类。例如，不可能在同一个语句中将`NoSuchFileException`和`IOException`放在一起——它们应该放在两个不同的`catch`语句中。

## 练习 1：记录异常

在捕获异常时，除了您可能想要执行以响应情况的任何类型的创造性编码之外，您还可以执行两种主要操作；这些操作是记录或抛出。在本练习中，您将学习如何记录异常。在后续练习中，您将学习如何抛出它。正如我们将在本章的“异常处理最佳实践”部分中重申的那样，您永远不应该同时执行这两者：

1.  在 IntelliJ 中使用 CLI 模板创建一个新的 Java 项目。将其命名为 LoggingExceptions。您将在其中创建类，以后可以在其他程序中使用它们。

1.  在代码中，您需要通过以下命令导入日志 API：

    ```java
    import java.util.logging.*;
    ```

1.  声明一个您将用于记录数据的对象。此对象将在程序终止时打印到终端；因此，您无需担心它此时会出现在哪里：

    ```java
    Logger logger = Logger.getAnonymousLogger();
    ```

1.  按如下方式引发异常：

    ```java
    String s = null;
    try {
        System.out.println(s.length());
    } catch (NullPointerException ne) {
        // do something here
    }
    ```

1.  在捕获异常时，使用`log()`方法将数据发送到日志对象：

    ```java
    logger.log(Level.SEVERE, "Exception happened", ne);
    ```

1.  您的完整程序应如下所示：

    ```java
    import java.util.logging.*;
    public class LoggingExceptions {
        public static void main(String[] args) {
            Logger logger = Logger.getAnonymousLogger();
            String s = null;
            try {
                System.out.println(s.length());
            } catch (NullPointerException ne) {
                logger.log(Level.SEVERE, "Exception happened", ne);
            }
        }
    }
    ```

1.  当您执行代码时，输出应如下所示：

    ```java
    may 09, 2019 7:42:05 AM LoggingExceptions main
    SEVERE: Exception happened
    java.lang.NullPointerException
          at LoggingExceptions.main(LoggingExceptions .java:10)
    Process finished with exit code 0
    ```

1.  如您所见，异常被记录在确定的`SEVERE`级别，但由于我们能够处理异常，代码结束时没有错误代码。日志很有用，因为它告诉我们异常发生在代码的哪个位置，并且还帮助我们找到可以进一步深入代码并修复任何潜在问题的位置。

# 抛出和抛出

您可以选择不在代码的低级别处理一些捕获的异常，如前所述。过滤掉异常的父类并关注检测对我们可能更有重要性的子类可能很有趣。`throws`关键字用于您正在创建的方法的定义以及可能发生异常的地方。在以下情况下，这是对*示例 09*的修改，我们应该在`main()`的定义中调用`throws`：

```java
import java.io.*;
import java.nio.file.*;
import java.util.*;
public class Example10 {
    public static void main(String[] args) throws IOException {
        // declare a list that will contain all of the files
        // inside of the readme.txt file
        List<String> lines = Collections.emptyList();
        try {
            lines = Files.readAllLines(Paths.get("readme.txt"));
        } catch (NoSuchFileException fe) {
            System.out.println("Exception: File Not Found");
            //fe.printStackTrace();
        }
        // you will never see this print
        Iterator<String> iterator = lines.iterator();
        while (iterator.hasNext())
            System.out.println(iterator.next());
    }
}
```

如您所见，我们在运行时抛出了任何`IOException`。这样，我们可以专注于捕获实际发生的异常：`NoSuchFileException`。通过使用逗号分隔，可以以这种方式抛出多个异常类型。

这种方法定义的一个例子如下：

```java
public static void main(String[] args) throws IOException, NullPointerException {
```

一件不可能的事情是在同一个方法定义中抛出异常类及其子类——正如我们在尝试在单个`catch`语句中捕获多个异常时所看到的那样。还有一点也很有趣，即`throws`在某个范围内操作；例如，我们可以在类的方法中忽略某个异常，但不能在另一个类中忽略。

另一方面，随着你对术语理解的深入，你还会发现另一个关键字对于处理异常非常有用。`throw` 关键字（注意这不同于 `throws`）将显式地引发一个异常。你可以使用它来创建自己的异常并在代码中尝试它们。我们将在后面的部分演示如何创建自己的异常，然后我们将使用 `throw` 作为示例的一部分来查看异常是如何传播的。使用 `throw` 的主要原因是如果你想让你的代码将类内部发生的异常传递给层次结构中的另一个类。为了了解这是如何工作的，让我们看看以下示例：

```java
public class Example11 {
    public static void main(String args[]) {
        String text = null;
        try {
            System.out.println(text.length());
        } catch (Exception e) {
            System.out.println("Exception: this should be a NullPointerException");
            throw new RuntimeException();
        }
    }
}
```

在这种情况下，我们通过尝试在初始化为 `null` 的字符串上调用 `length()` 方法来重现我们之前看到的 `NullPointerException` 示例。然而，如果你运行此代码，你会看到显示的异常是 `RuntimeException`：

```java
Exception: this should be a NullPointerException
Exception in thread "main" java.lang.RuntimeException
      at Example11.main(Example11.java:9)
Process finished with exit code 1
```

这是因为我们在 `catch` 块中发出的 `throw new RuntimeException()` 调用。正如你所看到的，在处理异常时，我们正在引发一个不同的异常。这可以非常有助于捕获异常并将它们通过你自己的异常传递，或者简单地捕获异常，给出一个有意义的消息来帮助用户理解发生了什么，然后让异常继续其自己的路径，如果异常没有被代码中的更高层次处理，最终导致程序崩溃。

## 练习 2：违反规则（并修复它）

在这个例子中，你将创建自己的检查异常类。你将定义一个类，然后通过引发该异常、记录其结果并分析它们来进行实验：

1.  在 IntelliJ 中使用 CLI 模板创建一个新的 Java 项目。将其命名为 `BreakingTheLaw`。你将在其中创建类，稍后可以在其他程序中使用这些类。

1.  在代码中创建一个新的类来描述你的异常。这个类应该扩展基本 `Exception` 类。命名为 `MyException` 并包含一个空构造函数：

    ```java
    public class BreakingTheLaw {
        class MyException extends Exception {
            // Constructor
            MyException() {};
        }
        public static void main(String[] args) {
          // write your code here
        }
    }
    ```

1.  你的构造函数应该包含所有可能抛出的异常。这意味着构造函数需要考虑几个不同的案例：

    ```java
    // Constructor
    public MyException() { 
        super(); 
    }
    public MyException(String message) { 
        super(message); 
    }
    public MyException(String message, Throwable cause) { 
        super(message, cause); 
    }
    public MyException(Throwable cause) { 
        super(cause); 
    }
    ```

1.  这将允许我们现在将任何异常包裹在我们的新形成的异常中。然而，我们需要对我们的程序进行一些修改，以便它能够编译。首先，我们需要将异常类设置为静态，以便在当前使用它的上下文中工作：

    ```java
    public static class MyException extends Exception {
    ```

1.  接下来，你需要确保主类正在抛出你新创建的异常，因为你将在代码中发出这个异常：

    ```java
    public static void main(String[] args) throws MyException {
    ```

1.  最后，你需要生成一些代码，当尝试获取初始化为 `null` 的 `String` 的长度时，将引发异常，例如 `NullPointerException`，然后捕获它，并使用我们新创建的类将其丢弃：

    ```java
    public static void main(String[] args) throws MyException {
        String s = null;
        try {
            System.out.println(s.length());
        } catch (NullPointerException ne) {
            throw new MyException("Exception: my exception happened");
        }
    }
    ```

1.  运行此代码的结果如下：

    ```java
    Exception in thread "main" BreakingTheLaw$MyException: Exception: my exception happened
          at BreakingTheLaw.main(BreakingTheLaw.java:26)
    Process finished with exit code 1
    ```

1.  你现在可以通过使用类中的任何其他构造函数来实验`throw`的调用。我们刚刚尝试了一个包含我们自己的错误消息的例子，所以让我们添加异常的堆栈跟踪：

    ```java
    throw new MyException("Exception: my exception happened", ne);
    ```

1.  将使输出稍微更有信息量的，是它现在将包括有关生成我们自己的`NullPointerException`的异常的信息：

    ```java
    Exception in thread "main" BreakingTheLaw$MyException: Exception: my exception happened
          at BreakingTheLaw.main(BreakingTheLaw.java:26)
    Caused by: java.lang.NullPointerException
          at BreakingTheLaw.main(BreakingTheLaw.java:24)
    Process finished with exit code 1
    ```

    你现在已经学会了如何使用`throw`将异常包装到自己的异常类中。当处理大型代码库并需要在长日志文件中查找由你的代码生成的异常时，这会非常有用。

    注意

    最终代码可以参考：[`packt.live/2VVdy2f`](https://packt.live/2VVdy2f)。

# `finally`块

可以使用`finally`块在代码中用于处理一系列不同异常的任何`catch`块之后执行一些通用代码。回到我们尝试打开一个不存在的文件的例子，包括一个`finally`语句的修改版本如下：

```java
Example12.java
11         try {
12             // provoke an exception
13             lines = Files.readAllLines(Paths.get("readme.txt"));
14         } catch (NoSuchFileException fe) {
15             System.out.println("Exception: File Not Found");
16         } catch (IOException ioe) {
17             System.out.println("Exception: IOException");
18         } finally {
19             System.out.println("Exception: Case Closed");
20         }
https://packt.live/2VTBFOS
```

上述示例的输出如下：

```java
Exception: File Not Found
Exception: Case Closed
Process finished with exit code 0
```

在检测到`NoSuchFileException`的`catch`块之后，处理机制跳入`finally`块并执行其中的任何内容，在这种情况下，意味着向输出打印另一行文本。

## 活动一：设计一个记录数据的异常类

我们已经看到了如何记录异常和抛出异常的例子。我们还学习了如何创建异常类并抛出它们。有了所有这些信息，这个活动的目标是创建一个自己的异常类，该类应该根据严重程度记录不同的异常。你应该制作一个基于程序参数的应用程序，程序将以不同的方式响应记录的异常。为了有一个共同的基础，使用以下标准：

1.  如果输入是数字 1，则抛出`NullPointerException`，严重程度为 SEVERE。

1.  如果输入是数字 2，则抛出`NoSuchFileException`，严重程度为 WARNING。

1.  如果输入是数字 3，则抛出`NoSuchFileException`，严重程度为 INFO。

1.  为了制作这个程序，你需要考虑创建自己的异常抛出方法，例如以下内容：

    ```java
    public static void issuePointerException() throws NullPointerException {
        throw new NullPointerException("Exception: file not found");
    }
    public static void issueFileException() throws NoSuchFileException {
        throw new NoSuchFileException("Exception: file not found");
    }
    ```

    注意

    这个活动的解决方案可以在第 540 页找到。

# 处理异常的最佳实践

在你的代码中处理异常需要遵循一系列最佳实践，以避免在编写程序时出现更深层的问题。这个常见实践列表对你的代码来说很重要，以保持一定程度的专业编程一致性：

第一条建议是避免抛出或捕获主`Exception`类。处理异常时，你需要尽可能具体。因此，以下情况是不推荐的：

```java
public class Example13 {
    public static void main(String args[]) {
        String text = null;
        try {
            System.out.println(text.length());
        } catch (Exception e) {
            System.out.println("Exception happened");
        }
    }
}
```

这段代码将捕获任何异常，没有粒度。那么，你应该如何以这种方式正确处理异常呢？

在下一节中，我们将快速回顾 Exception 类在 Java API 结构中的位置。我们将检查它如何与 Error 类在同一级别悬挂在 Throwable 类上。因此，如果你捕获了 Throwable 类，你可能会掩盖代码中发生的可能错误，而不仅仅是异常。记住，错误是那些你的代码应该退出的情况，因为它们会警告到可能导致 JVM 资源误用的真实故障。

在`catch`块后面掩盖这样的场景可能会使整个 JVM 停滞。因此，避免以下代码：

```java
try {
    System.out.println(text.length());
} catch (Throwable e) {
    System.out.println("Exception happened");
}
```

在*练习 2*，*违法（并修复它）*中，你看到了如何创建自己的异常类。正如讨论的那样，通过使用`throw`可以将异常重定向到其他地方。不忽视原始异常的堆栈跟踪是一种好习惯，因为它将帮助你更好地调试问题的来源。因此，在捕获原始异常时，你应该考虑将整个堆栈跟踪作为参数传递给异常构造函数：

```java
} catch (OriginalException e) {
    throw new MyVeryOwnException("Exception trace: ", e);
}
```

在同样的练习中，当你自己创建异常时，你学习了如何使用系统日志来存储异常信息。你应该避免记录异常和再次抛出它。你应该尽量在代码中记录最高级别的信息。否则，你的日志中会出现关于情况的重复信息，使得调试变得更加复杂。因此，我们建议你使用以下方法：

```java
throw new NewException();
```

或者，你可以在同一个`catch`块中使用以下内容，但不能同时使用：

```java
log.error("Exception trace: ", e);
```

此外，在记录信息时，尽量使用系统日志的单次调用。随着你的代码越来越大，会有多个进程并行工作，因此会有很多不同的来源发出日志命令：

```java
log.debug("Exception trace happened here");
log.debug("It was a bad thing");
```

这很可能不会在日志中显示为连续的两行，而是显示为间隔较远的两行。相反，你应该这样做：

```java
log.debug("Exception trace happened here. It was a bad thing");
```

当处理多个异常时，一些是其他异常的子类，你应该按照顺序捕获它们，从最具体的开始。我们在本章的一些示例中看到了这一点，例如处理`NoSuchFileException`和`IOException`。你的代码应该看起来像这样：

```java
try {
    tryAnExceptionCode();
} catch (SpecificException se) {
    doTheCatch1();
} catch (ParentException pe) {
    doTheCatch2();
}
```

如果你根本不打算捕获异常，但仍然被迫使用`try`块来编译代码，请使用`finally`块来关闭在异常之前启动的所有操作。一个例子是打开一个在离开方法之前应该关闭的文件，这将会因为异常而发生：

```java
try {
    tryAnExceptionCode();
} finally {
    closeWhatever();
}
```

`throw` 关键字是一个非常强大的工具，正如你所注意到的。能够重定向异常允许你为不同的情境创建自己的策略，并且此外，这意味着你不必依赖于 JVM 默认提供的策略。然而，你在捕获时应该小心放置 `throw` 在某些块中。你应该避免在 `finally` 块中使用 `throw`，因为这会掩盖异常的原始原因。

在处理 Java 异常时，这在某种程度上遵循了“尽早抛出，晚些捕获”的原则。想象一下，你正在进行一个低级操作，它是更大方法的一部分。例如，你正在打开一个文件，作为解析其内容并查找模式的一段代码的一部分。如果打开文件的操作由于异常而失败，那么简单地 `throw` 该异常给后续的方法，以便它能够将其置于上下文中，并能够在更高层次上决定如何继续整个任务，这是一个更好的选择。你应该只在能够做出最终决策的更高层次上处理异常。

我们在之前的例子中看到了 `printStackTrace()` 的使用，作为一种查看异常完整来源的方法。虽然当调试代码时能够看到这一点非常有趣，但如果不处于那种心态，它几乎是没有关系的。因此，你应该确保删除或注释掉你可能使用过的所有 `printStackTrace()` 命令。如果以后需要，其他开发者将不得不在分析代码时确定他们想要放置探针的位置。

以类似的方式，当你在方法内部以任何方式处理异常时，你应该记得在 Javadoc 中正确地记录事情。你应该添加一个 `@throws` 声明来明确指出哪种异常到达，以及它是被处理、传递还是其他什么情况：

```java
/**
* Method name and description
*
* @param input
* @throws ThisStrangeException when ...
*/
public void myMethod(Integer input) throws ThisStrangeException {
    ...
}
```

# 异常从何而来？

离开我们在本章中遵循的更实际的途径，现在是时候从更宏观的角度来看待问题，并理解在 Java API 的更大框架中事物从何而来。正如前一个部分提到的，异常悬挂在 `Throwable` 类上，它是 `java.lang` 包的一部分。它们与错误（我们之前解释过）处于同一级别。换句话说，`Exception` 和 `Error` 都是 `Throwable` 的子类。

只有`Throwable`类的对象实例可以通过 Java 的`throw`语句抛出；因此，我们必须使用这个类作为起点来定义自己的异常。正如 Java 文档中关于`Throwable`类的说明，这包括创建时的执行栈快照。这允许您查找异常（或错误）的来源，因为它包括了当时计算机内存的状态。可抛出对象可以包含构建它的原因。这就是所谓的链式异常功能，因为一个异常事件可能是由一系列异常引起的。这是我们分析本章中某些程序堆栈跟踪时看到的情况。

# 摘要

我们在本章中采用了非常实际的方法。我们首先让您的代码以不同的方式出错，然后解释了错误和异常之间的区别。我们专注于处理后者，因为这些是唯一不应该立即使您的程序崩溃的情况。

异常可以通过捕获或抛出进行处理。前者是通过观察不同的异常并定义不同的策略，通过 try-catch 语句来应对这些情况。您可以选择将异常重新发送到不同的类中使用`throw`，或者在`catch`块中响应。无论您遵循哪种策略，您都可以通过`finally`块设置系统在处理异常后执行一些最后的代码行。

本章还包括了一系列关于如何在更概念层面上处理异常的建议。您有一份最佳实践清单，任何专业程序员都会遵循。

最后，在实践层面，您完成了一系列练习，这些练习引导您通过处理异常的经典场景，并且您已经看到了可以用来调试代码的不同工具，例如日志和`printStackTrace()`。
