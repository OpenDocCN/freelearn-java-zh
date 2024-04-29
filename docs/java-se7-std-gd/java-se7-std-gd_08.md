# 第八章：在应用程序中处理异常

异常是由应用程序或**Java 虚拟机**（**JVM**）在发生某种错误时抛出的对象。Java 提供了各种预定义的异常，并允许开发人员声明和创建自己的异常类。

虽然有许多分类异常的方法，但其中一种方案将它们分类为三种类型：

+   程序错误

+   代码的不当使用

+   与资源相关的故障

程序错误是代码序列中的内部缺陷。程序员可能对这些类型的错误无能为力。例如，常见的异常是`NullPointerException`。这通常是由于未正确初始化或分配值给引用变量。在编写代码时，这种错误很难避免和预料。然而，一旦检测到，可以修改代码以纠正情况。

代码可能被错误地使用。大多数库都是设计用于特定方式的。它们可能期望数据以某种方式组织，如果库的用户未能遵循格式，就可能引发异常。例如，方法的参数可能未按方法所期望的结构化，或者可能是错误的类型。

一些错误与资源故障有关。当底层系统无法满足程序的需求时，可能会发生资源类型的异常。例如，网络故障可能会阻止程序正常执行。这种类型的错误可能需要在以后的时间重新执行程序。

处理异常的传统方法是从过程中返回错误代码。例如，如果函数执行时没有错误，则通常会返回零。如果发生错误，则会返回非零值。这种方法的问题在于函数的调用可能会出现以下情况：

+   不知道函数返回错误代码（例如，C 的`printf`函数）

+   忘记检查错误

+   完全忽略错误

当错误没有被捕获时，程序的继续执行可能会导致不可预测的，可能是灾难性的后果。

这种方法的替代方法是“捕获”错误。大多数现代的块结构化语言，如 Java，使用这种方法。这种技术需要更少的编码，更易读和更健壮。当一个例程检测到错误时，它会“抛出”一个异常对象。然后将异常对象返回给调用者，调用者捕获并处理错误。

异常应该被捕获有许多原因。未能处理异常可能导致应用程序失败，或者以不正确的输出结束处于无效状态。保持一致的环境总是一个好主意。此外，如果打开了资源，比如文件，在完成后应该始终关闭资源，除了最琐碎的程序。

Java 中提供的异常处理机制允许您这样做。当打开资源时，即使程序中发生异常，也可以关闭资源。为了完成这个任务，资源在`try`块中打开，并在`catch`或`finally`块中关闭。`try`、`catch`和`finally`块构成了 Java 中使用的异常处理机制的核心。

# 异常类型

Java 已经提供了一套广泛的类来支持 Java 中的异常处理。异常是直接或间接从`Throwable`类派生的类的实例。从`Throwable`派生了两个预定义的 Java 类——`Error`和`Exception`。从`Exception`类派生了一个`RuntimeException`类。正如我们将很快看到的，程序员定义的异常通常是从`Exception`类派生的：

![异常类型](img/7324_08_01.jpg)

有许多预定义的错误，它们源自`Error`和`RuntimeException`类。程序员对于从`Error`对象派生的异常几乎不会做任何处理。这些异常代表了 JVM 的问题，通常无法恢复。`Exception`类是不同的。从`Exception`类派生的两个类支持两种类型的异常：

+   **经过检查的异常**：这些异常在代码中需要处理

+   **未经检查的异常**：这些异常在代码中不需要处理

经过检查的异常包括所有从`Exception`类派生而不是从`RuntimeException`类派生的异常。这些必须在代码中处理，否则代码将无法编译，导致编译时错误。

未经检查的异常是所有其他异常。它们包括除零和数组下标错误等异常。这些异常不必被捕获，但是像`Error`异常一样，如果它们没有被捕获，程序将终止。

我们可以创建自己的异常类。当我们这样做时，我们需要决定是创建一个经过检查的异常还是未经检查的异常。一个经验法则是，如果客户端代码无法从异常中恢复，将异常声明为未经检查的异常。否则，如果他们可以处理它，就将其作为经过检查的异常。

### 注意

一个类的用户不必考虑未经检查的异常，这些异常可能导致程序终止，如果客户端程序从未处理它们。一个经过检查的异常要求客户端要么捕获异常，要么显式地将其传递到调用层次结构中。

# Java 中的异常处理技术

在处理 Java 异常时，我们可以使用三种常规技术：

+   传统的`try`块

+   Java 7 中引入的新的“try-with-resources”块

+   推卸责任

第三种技术是在当前方法不适合处理异常时使用的。它允许异常传播到方法调用序列中更高的位置。在以下示例中，`anotherMethod`可能会遇到一些条件，导致它可能抛出`IOException`。在`someMethod`中不处理异常，而是在`someMethod`定义中使用`throws`关键字，结果是将异常传递给调用此方法的代码：

```java
public void someMethod() throws IOException {
   …
   object.anotherMethod(); // may throw an IOException
   …
}
```

该方法将跳过方法中剩余的所有代码行，并立即返回给调用者。未捕获的异常会传播到下一个更高的上下文，直到它们被捕获，或者它们从`main`中抛出，那里将打印错误消息和堆栈跟踪。

## 堆栈跟踪

`printStackTrace`是`Throwable`类的一个方法，它将显示程序在该点的堆栈。当异常未被捕获时，它会自动使用，或者可以显式调用。该方法的输出指出了导致程序失败的行和方法。您以前已经看到过这种方法的使用，每当您遇到未处理的运行时异常时。当异常未被处理时，该方法会自动调用。

`ExceptionDemo`程序说明了该方法的显式使用：

```java
public class ExceptionDemo {

   public void foo3() {
      try {
         …
         throw new Exception();
      }
      catch (Exception e) {
         e.printStackTrace();
      }
   }

   public void foo2() { foo3(); }
   public void foo1() { foo2(); }

   public static void main(String args[]) {
      new ExceptionDemo().foo1();
   }
}
```

输出如下所示：

```java
java.lang.Exception
 at ExceptionDemo.foo3(ExceptionDemo.java:8)
 at ExceptionDemo.foo2(ExceptionDemo.java:16)
 at ExceptionDemo.foo1(ExceptionDemo.java:20)
 at ExceptionDemo.main(ExceptionDemo.java:25)

```

## 使用 Throwable 方法

`Throwable`类拥有许多其他方法，可以提供更多关于异常性质的见解。为了说明这些方法的使用，我们将使用以下代码序列。在这个序列中，我们尝试打开一个不存在的文件并检查抛出的异常：

```java
private static void losingStackTrace(){
   try {
      File file = new File("c:\\NonExistentFile.txt");
      FileReader fileReader = new FileReader(file);
   } 
   catch (FileNotFoundException e) {
      e.printStackTrace();

      System.out.println();
      System.out.println("---e.getCause(): " + 
                   e.getCause());
      System.out.println("---e.getMessage(): " + 
                   e.getMessage());
      System.out.println("---e.getLocalizedMessage(): " + 
                   e.getLocalizedMessage());
      System.out.println("---e.toString(): " + 
                   e.toString());
   }
}

```

由于一些 IDE 的性质，应用程序的标准输出和标准错误输出可能会交错。例如，上述序列的执行可能导致以下输出。您可能会在输出中看到交错，也可能不会看到。输出前面的破折号用于帮助查看交错行为：

```java
java.io.FileNotFoundException: c:\NonExistentFile.txt (The system cannot find the file specified)
---e.getCause(): null
---e.getMessage(): c:\NonExistentFile.txt (The system cannot find the file specified)
   at java.io.FileInputStream.open(Native Method)
---e.getLocalizedMessage(): c:\NonExistentFile.txt (The system cannot find the file specified)
---e.toString(): java.io.FileNotFoundException: c:\NonExistentFile.txt (The system cannot find the file specified)
   at java.io.FileInputStream.<init>(FileInputStream.java:138)
   at java.io.FileReader.<init>(FileReader.java:72)
   at packt.Chapter8Examples.losingStackTrace(Chapter8Examples.java:64)
   at packt.Chapter8Examples.main(Chapter8Examples.java:57)

```

在本例中使用的方法总结在以下表中：

| 方法 | 意义 |
| --- | --- |
| `getCause` | 返回异常的原因。如果无法确定原因，则返回 null。 |
| `getMessage` | 返回详细消息。 |
| `getLocalizedMessage` | 返回消息的本地化版本。 |
| `toString` | 返回消息的字符串版本。 |

请注意，`printStackTrace`方法的第一行是`toString`方法的输出。

`getStackTrace`方法返回一个`StackTraceElement`对象数组，其中每个元素表示堆栈跟踪的一行。我们可以使用以下代码序列复制`printStackTrace`方法的效果：

```java
try {
   File file = new File("c:\\NonExistentFile.txt");
   FileReader fileReader = new FileReader(file);
} 
catch (FileNotFoundException e) {
   e.printStackTrace();
   System.out.println();
   StackTraceElement traces[] = e.getStackTrace();
   for (StackTraceElement ste : traces) {
      System.out.println(ste);
   }
}
```

执行时，我们得到以下输出：

```java
java.io.FileNotFoundException: c:\NonExistentFile.txt (The system cannot find the file specified)
 at java.io.FileInputStream.open(Native Method)
 at java.io.FileInputStream.<init>(FileInputStream.java:138)
 at java.io.FileReader.<init>(FileReader.java:72)
 at packt.Chapter8Examples.losingStackTrace(Chapter8Examples.java:64)
 at packt.Chapter8Examples.main(Chapter8Examples.java:57)

java.io.FileInputStream.open(Native Method)
java.io.FileInputStream.<init>(FileInputStream.java:138)
java.io.FileReader.<init>(FileReader.java:72)
packt.Chapter8Examples.losingStackTrace(Chapter8Examples.java:64)
packt.Chapter8Examples.main(Chapter8Examples.java:57)

```

# 传统的 try-catch 块

处理异常的传统技术使用`try`、`catch`和`finally`块的组合。`try`块用于包围可能引发异常的代码，然后是零个或多个`catch`块，最后是一个可选的`finally`块。

`catch`块在`try`块之后添加以“捕获”异常。`catch`块中的语句提供了“处理”错误的代码块。在`catch`块之后可以选择使用`finally`子句。它保证即使`try`或`catch`块中的代码引发或不引发异常，也会执行。

### 注意

但是，如果在 try 或 catch 块中调用`System.exit`方法，则 finally 块将不会执行。

以下序列说明了这些块的使用。在 try 块内，读取一行并提取一个整数。使用两个 catch 块来处理可能抛出的异常：

```java
try {
   inString = is.readLine();
   value = Integer.parseInt (inString);
   …
} 
catch (IOException e) {
   System.out.println("I/O Exception occurred");
} 
catch (NumberFormatException e) {
   System.out.println("Bad format, try again...");
} 
finally {
   // Perform any necessary clean-up action
}

```

在此代码序列中可能出现两种错误中的一种：

+   要么尝试读取输入行时会发生错误，要么

+   尝试将字符串转换为整数时将发生错误

第一个 catch 块将捕获 IO 错误，第二个 catch 块将捕获转换错误。当抛出异常时，只有一个 catch 块会被执行。

可能会发生错误，也可能不会。无论如何，finally 块将在 try 块完成或 catch 块执行后执行。`finally`子句保证运行，并通常包含“清理”代码。

# 使用 try-with-resource 块

当多个资源被打开并发生故障时，使用先前的技术可能会很麻烦。它可能导致多个难以跟踪的 try-catch 块。在 Java 7 中，引入了 try-with-resources 块来解决这种情况。

try-with-resources 块的优势在于，块中打开的所有资源在退出块时会自动关闭。使用 try-with-resources 块的任何资源都必须实现`java.lang.AutoCloseable`接口。

我们将通过创建一个简单的方法来将一个文件复制到另一个文件来说明这种方法。在下面的示例中，一个文件用于读取，另一个文件用于写入。请注意，它们是在`try`关键字和块的左花括号之间创建的：

```java
try (BufferedReader reader = Files.newBufferedReader(
    Paths.get(new URI("file:///C:/data.txt")),
      Charset.defaultCharset());
    BufferedWriter writer = Files.newBufferedWriter(
      Paths.get(new URI("file:///C:/data.bak")),
      Charset.defaultCharset())) {

  String input;
  while ((input = reader.readLine()) != null) {
    writer.write(input);
    writer.newLine();
  }
} catch (URISyntaxException | IOException ex) {
  ex.printStackTrace();
}

```

要管理的资源在一对括号内声明和初始化，并放在`try`关键字和 try 块的左花括号之间。第一个资源是使用`data.txt`文件的`BufferedReader`对象，第二个资源是与`data.bak`文件一起使用的`BufferedWriter`对象。`Paths`类是 Java 7 中的新功能，提供了改进的 IO 支持。

使用 try-with-resources 块声明的资源必须用分号分隔，否则将生成编译时错误。有关 try-with-resources 块的更深入覆盖可以在《Java 7 Cookbook》中找到。

在 catch 块中使用竖线是 Java 7 中的新功能，允许我们在单个 catch 块中捕获多个异常。这在*在 catch 块中使用|运算符*部分有解释。

# catch 语句

catch 语句只有一个参数。如果 catch 语句的参数：

+   完全匹配异常类型

+   是异常类型的基类

+   是异常类型实现的接口

只有与异常匹配的第一个 catch 语句将被执行。如果没有匹配，方法将终止，并且异常将冒泡到调用方法，那里可能会处理它。

之前的`try`块的一部分如下所示重复。`catch`语句的格式由`catch`关键字后面跟着一组括号括起来的异常声明组成。然后是一个块语句中的零个或多个语句：

```java
try {
   …
} 
catch (IOException e) {
   System.out.println("I/O Exception occurred");
} 
catch (NumberFormatException e) {
   System.out.println("Bad format, try again...");
} 
```

处理错误的过程由程序员决定。它可能只是简单地显示一个错误消息，也可能非常复杂。程序员可以使用错误对象重试操作或以其他方式处理它。在某些情况下，这可能涉及将其传播回调用方法。

## catch 块的顺序

在 try 块后列出 catch 块的顺序可能很重要。当抛出异常时，异常对象将按照它们的顺序与 catch 块进行比较。比较检查抛出的异常是否是 catch 块中异常的类型。

例如，如果抛出了`FileNotFoundException`，它将匹配具有`IOException`或`FileNotFoundException`异常的 catch 块，因为`FileNotFoundException`是`IOException`的子类型。由于在找到第一个匹配项时比较会停止，如果`IOException`的 catch 块在`FileNotFoundException`的 catch 块之前列出，`FileNotFoundException`块将永远不会被执行。

考虑以下异常类的层次结构：

![catch 块的顺序](img/7324_08_02.jpg)

给定以下代码序列：

```java
try {
   …
}
catch (AException e) {…}
catch (BException e) {…}
catch (CException e) {…}
catch (DException e) {…}
```

如果抛出的异常是这些类型的异常之一，`AException` catch 块将始终被执行。这是因为`AException`、`BException`、`CException`或`DException`都是`AException`类型。异常将始终匹配`AException`异常。其他`catch`块将永远不会被执行。

通常规则是始终首先列出“最派生”的异常。以下是列出异常的正确方式：

```java
try {
   …
}
catch (DException e) {…}
catch (BException e) {…}
catch (CException e) {…}
catch (AException e) {…}
```

请注意，对于异常的这种层次结构，无论`BException`是紧接着还是跟在`CException`之后，都没有任何区别，因为它们处于同一级别。

## 在 catch 块中使用|运算符

有时希望以相同的方式处理多个异常。我们可以使用竖线来允许一个 catch 块捕获多个异常，而不是在每个 catch 块中重复代码。

考虑可能抛出两个异常并以相同方式处理的情况：

```java
try {
   …
} 
catch (IOException e) {
   e.printStackTrace();
} 
catch (NumberFormatException e) {
   e.printStackTrace();
} 
```

竖线可以用于在相同的`catch`语句中捕获两个或更多的异常，如下面的代码片段所示。这可以减少处理以相同方式处理的两个异常所需的代码量。

```java
try {
   …
} 
catch (IOException | NumberFormatException e) {
   e.printStackTrace();
}
```

当多个异常可以以相同方式处理时，这种方法是有效的。请记住，catch 块的参数是隐式 final 的。无法将不同的异常赋值给该参数。以下尝试是非法的，不会编译通过：

```java
catch (IOException | NumberFormatException e) {
   e = new Exception();  // Compile time error
}
```

# finally 块

`finally`块跟在一系列`catch`块后面，由`finally`关键字后面跟着一系列语句组成。它包含一个或多个语句，这些语句将始终被执行以清理之前的操作。`finally`块将始终执行，无论异常是否存在。但是，如果`try`或`catch`块调用了`System.exit`方法，程序将立即终止，`finally`块将不会执行。

`finally`块的目的是关闭或以其他方式处理在`try`块中打开的任何资源。关闭不再需要的资源是一种良好的实践。我们将在下一个例子中看到这一点。

然而，在实践中，这通常是繁琐的，如果需要关闭多个资源，关闭过程可能也会生成异常，这可能会导致错误。此外，如果一个资源在打开时抛出异常，而另一个资源没有打开，我们必须小心不要尝试关闭第二个资源。因此，在 Java 7 中引入了 try-with-resources 块来解决这种问题。这个块在*使用 try-with-resources 块*部分中进行了讨论。在这里，我们将介绍`finally`块的简化使用。

一个使用`finally`块的简单示例如下所示。在这个序列中，我们将打开一个文件进行输入，然后显示其内容：

```java
BufferedReader reader = null;     
try {
   File file1 = new File("c:\\File1.txt");

   reader = new BufferedReader(new FileReader(file1));
   // Copy file
   String line;
   while((line = reader.readLine()) != null) {
      System.out.println(line);
   }
} 
catch (IOException e) {
   e.printStackTrace();
}
finally {
   if(reader != null) {
      reader.close();
   }
}
```

无论是否抛出异常，文件都将被关闭。如果文件不存在，将抛出`FileNotFoundException`。这将在`catch`块中捕获。请注意我们如何检查`reader`变量以确保它不是 null。

在下面的例子中，我们打开两个文件，然后尝试将一个文件复制到另一个文件。`finally`块用于关闭资源。这说明了在处理多个资源时`finally`块的问题：

```java
BufferedReader br = null;
BufferedWriter bw = null;        
try {
   File file1 = new File("c:\\File1.txt");
   File file2 = new File("c:\\File2.txt");

   br = new BufferedReader(new FileReader(file1));
   bw = new BufferedWriter(new FileWriter(file2));
   // Copy file
} 
catch (FileNotFoundException e) {
   e.printStackTrace();
}
catch (IOException e) {
   e.printStackTrace();
}
finally {
   try {
      br.close();
      bw.close();
   } catch (IOException ex) {
      // Handle close exception
   }
}
```

请注意，`close`方法也可能会抛出`IOException`。我们也必须处理这些异常。这可能需要一个更复杂的异常处理序列，这可能会导致错误。在这种情况下，请注意，如果在关闭第一个文件时抛出异常，第二个文件将不会被关闭。在这种情况下，最好使用 try-with-resources 块，如*使用 try-with-resources 块*部分所述。

### 提示

try 块需要一个 catch 块或一个 finally 块。如果没有一个或两个，将生成编译时错误。

# 嵌套的 try-catch 块

异常处理可以嵌套。当在`catch`或`finally`块中使用也会抛出异常的方法时，这可能是必要的。以下是在`catch`块中使用嵌套`try`块的示例：

```java
try {
   // Code that may throw an exception
}
catch (someException e) {
   try {
      // Code to handle the exception
   }
   catch (anException e) {
      // Code to handle the nested exception
   } 
}
catch (someOtherException e) {
   // Code to handle the exception
} 
```

在上一节的最后一个例子中，我们在`finally`块中使用了`close`方法。然而，`close`方法可能会抛出`IOException`。由于它是一个受检异常，我们需要捕获它。这导致了一个`try`块嵌套在一个`finally`块中。此外，当我们尝试关闭`BufferedReader`时，第二个`try`块将抛出`NullPointerException`，因为我们尝试执行关闭方法针对从未分配值的`reader`变量。

为了完成前面的例子，考虑以下实现：

```java
finally {
   try {
      br.close();
      bw.close();
   } catch (IOException | NullPointerException e) {
       // Handle close exceptions
   }
   }
```

我们使用`|`符号来简化捕获两个异常，如*在 catch 块中使用|操作符*部分所述。这也是我们可能丢失原始异常的另一个例子。在这种情况下，`FileNotFoundException`丢失为`NullPointerException`。这将在*丢失堆栈跟踪*部分中讨论。

# 异常处理指南

本节介绍了处理异常的一般指导方针。它旨在提供如何以更有用和更有效的方式使用异常处理的示例。虽然糟糕的技术可能不会导致编译时错误或不正确的程序，但它们通常反映了糟糕的设计。

## 重复抛出异常的代码

当抛出异常然后捕获时，我们有时会想尝试重新执行有问题的代码。如果代码结构良好，这并不困难。

在这个代码序列中，假设`try`块进入时存在错误。如果生成错误，它将被`catch`块捕获并处理。由于`errorsArePresent`仍然设置为 true，`try`块将被重复执行。然而，如果没有发生错误，在`try`块结束时，`errorsArePresent`标志将被设置为 false，这将允许程序执行 while 循环并继续执行：

```java
boolean errorsArePresent;

…
errorsArePresent = true;	
while (errorsArePresent) {
   try {
      …
      errorsArePresent = false;
   } 

   catch (someException e) {
      // Process error
   } 

}
```

在这个例子中，假设用于处理错误的代码将需要重新执行`try`块。当我们在处理错误代码序列中所做的一切就是显示一个标识错误的错误消息时，比如用户输入了一个错误的文件名时，这可能是情况。

如果所需的资源不可用，使用这种方法时需要小心。这可能导致一个无限循环，我们检查一个不可用的资源，抛出异常，然后再次执行。可以添加一个循环计数器来指定我们尝试处理异常的次数。

## 不具体指明捕获的异常

在捕获异常时，要具体指明需要捕获的异常。例如，在以下示例中捕获了通用的`Exception`。没有具体的信息可以显示异常的原因：

```java
try {
   someMethod();
} catch (Exception e) {
   System.out.println("Something failed" + e);
}
```

接下来是一个更有用的版本，它捕获了实际抛出的异常：

```java
try {
   someMethod();
} catch (SpecificException e) {
   System.out.println("A specific exception message" + e);
}
```

## 丢失堆栈跟踪

有时会捕获异常，然后重新抛出不同的异常。考虑以下方法，其中抛出了一个`FileNotFoundException`异常：

```java
private static void losingStackTrace(){
   try {
      File file = new File("c:\\NonExistentFile.txt");
      FileReader fileReader = new FileReader(file);
   }
   catch(FileNotFoundException e) {
      e.printStackTrace();
   }
}
```

假设文件不存在，将生成以下堆栈跟踪：

```java
java.io.FileNotFoundException: c:\NonExistentFile.txt (The system cannot find the file specified)
   at java.io.FileInputStream.open(Native Method)
   at java.io.FileInputStream.<init>(FileInputStream.java:138)
   at java.io.FileReader.<init>(FileReader.java:72)
   at packt.Chapter8Examples.losingStackTrace(Chapter8Examples.java:49)
   at packt.Chapter8Examples.main(Chapter8Examples.java:42)
```

我们可以知道确切的异常是什么，以及它发生在哪里。接下来，考虑使用`MyException`类而不是`FileNotFoundException`异常：

```java
public class MyException extends Exception {
   private String information;

   public MyException(String information) {
      this.information = information;
   }
}
```

如果重新抛出异常，就像下面的代码片段所示，我们将丢失有关原始异常的信息：

```java
private static void losingStackTrace() throws MyException {
   try {
      File file = new File("c:\\NonExistentFile.txt");
      FileReader fileReader = new FileReader(file);
   }
   catch(FileNotFoundException e) {
      throw new MyException(e.getMessage());
   }
}
```

由此实现产生的堆栈跟踪如下：

```java
Exception in thread "main" packt.MyException
 at packt.Chapter8Examples.losingStackTrace(Chapter8Examples.java:53)
 at packt.Chapter8Examples.main(Chapter8Examples.java:42)

```

请注意，实际异常的细节已经丢失。一般来说，最好不要使用这种方法，因为丢失了用于调试的关键信息。这个问题的另一个例子可以在*嵌套的 try-catch 块*部分找到。

可以重新抛出并保留堆栈跟踪。为此，我们需要做以下操作：

1.  添加一个带有`Throwable`对象作为参数的构造函数。

1.  在需要保留堆栈跟踪时使用这个。

以下显示了将此构造函数添加到`MyException`类中：

```java
public MyException(Throwable cause) {
   super(cause);
}

```

在`catch`块中，我们将使用下面显示的这个构造函数。

```java
catch (FileNotFoundException e) {
   (new MyException(e)).printStackTrace();
}
```

我们本可以抛出异常。相反，我们使用了`printStackTrace`方法，如下所示：

```java
packt.MyException: java.io.FileNotFoundException: c:\NonExistentFile.txt (The system cannot find the file specified)
 at packt.Chapter8Examples.losingStackTrace(Chapter8Examples.java:139)
 at packt.Chapter8Examples.main(Chapter8Examples.java:40)
Caused by: java.io.FileNotFoundException: c:\NonExistentFile.txt (The system cannot find the file specified)
 at java.io.FileInputStream.open(Native Method)
 at java.io.FileInputStream.<init>(FileInputStream.java:138)
 at java.io.FileReader.<init>(FileReader.java:72)
 at packt.Chapter8Examples.losingStackTrace(Chapter8Examples.java:136)

```

## 作用域和块长度

在`try`、`catch`或`finally`块中声明的任何变量的作用域都限于该块。尽可能地限制变量的作用域是一个好主意。在下面的示例中，由于在`finally`块中需要，所以需要在`try`和`catch`块之外定义`reader`变量：

```java
BufferedReader reader = null;
try {
   reader = …
   …
}
catch (IOException e) {
   …
} finally {
   try {
      reader.close();
   } 
   catch (Exception e) {
      …
   }
}
```

块的长度应该是有限的。然而，块太小可能会导致您的代码变得混乱，异常处理代码也会变得混乱。假设有四种方法，每种方法都可能抛出不同的异常。如果我们为每个方法使用单独的 try 块，我们最终会得到类似以下的代码：

```java
try {
   method1();
}
catch (Exception1 e1) {
   …
} 
try {
   method2();
}

catch (Exception1 e2) {
   …
} 
try {
   method3();
}
catch (Exception1 e3) {
   …
} 
try {
   method4();
}
catch (Exception1 e4) {
   …
} 
```

这有点笨拙，而且如果每个`try`块都需要一个`finally`块，也会出现问题。如果这些在逻辑上相关，一个更好的方法是使用一个单独的`try`块，如下所示：

```java
try {
   method1();
   method2();
   method3();
   method4();
}
catch (Exception1 e1) {
   …
} 
catch (Exception1 e2) {
   …
} 
catch (Exception1 e3) {
   …
} 
catch (Exception1 e4) {
   …
} 

finally {
      …
}

```

根据异常的性质，我们还可以使用一个通用的基类异常，或者在 Java 7 中引入的`|`运算符与单个 catch 块。如果异常可以以相同的方式处理，这是特别有用的。

然而，将整个方法体放在一个包含与异常无关的代码的 try/catch 块中是一个不好的做法。如果可能的话，最好将异常处理代码与非执行处理代码分开。

一个经验法则是将异常处理代码的长度保持在一次可以看到的大小。使用多个 try 块是完全可以接受的。但是，请确保每个块包含逻辑相关的操作。这有助于模块化代码并使其更易读。

## 抛出 UnsupportedOperationException 对象

有时，打算被覆盖的方法会返回一个“无效”的值，以指示需要实现该方法。例如，在以下代码序列中，`getAttribute`方法返回`null`：

```java
class Base {
   public String getAttribute() {
      return null;
   }
   …
}
```

但是，如果该方法没有被覆盖并且使用了基类方法，可能会出现问题，例如产生不正确的结果，或者如果针对返回值执行方法，则可能生成`NullPointerException`。

更好的方法是抛出`UnsupportedOperationException`来指示该方法的功能尚未实现。这在以下代码序列中有所体现：

```java
class Base {
   public String getAttribute() {
      throw new UnsupportedOperationException();
   }
   …
}
```

在提供有效的实现之前，该方法无法成功使用。这种方法在 Java API 中经常使用。`java.util.Collection`类的`unmodifiableList`方法使用了这种技术（[`docs.oracle.com/javase/1.5.0/docs/api/java/util/Collections.html#unmodifiableList%28java.util.List%29`](http://docs.oracle.com/javase/1.5.0/docs/api/java/util/Collections.html#unmodifiableList%28java.util.List%29)）。通过将方法声明为抽象，也可以实现类似的效果。

## 忽略异常

通常忽略异常是一个不好的做法。它们被抛出是有原因的，如果有什么可以做来恢复，那么你应该处理它。否则，至少可以优雅地终止应用程序。

例如，通常会忽略`InterruptedException`，如下面的代码片段所示：

```java
while (true) {
   try {
      Thread.sleep(100000);
   } 
   catch (InterruptedException e) {
      // Ignore it
   }
}
```

然而，即使在这里也出了问题。例如，如果线程是线程池的一部分，池可能正在终止，你应该处理这个事件。始终了解程序运行的环境，并且预料到意外。

另一个糟糕的错误处理示例在以下代码片段中显示。在这个例子中，我们忽略了可能抛出的`FileNotFoundException`异常：

```java
private static void losingStackTrace(){
   try {
      File file = new File("c:\\NonExistentFile.txt");
      FileReader fileReader = new FileReader(file);
   }
   catch(FileNotFoundException e) {
      // Do nothing
   }
}
```

这个用户并不知道曾经遇到异常。这很少是一个可以接受的方法。

## 尽可能晚处理异常

当方法抛出异常时，方法的使用者可以在那一点处理它，或者将异常传递到调用序列中的另一个方法。诀窍是在适当的级别处理异常。通常，该级别是可以处理异常的级别。

例如，如果需要应用程序用户的输入才能成功处理异常，那么应该使用最适合与用户交互的级别。如果该方法是库的一部分，那么假设用户应该被提示可能不合适。当我们尝试打开一个文件而文件不存在时，我们不希望调用的方法提示用户输入不同的文件名。相反，我们更倾向于自己处理。在某些情况下，甚至可能没有用户可以提示，就像许多服务器应用程序一样。

## 在单个块中捕获太多

当我们向应用程序添加 catch 块时，我们经常会诱使使用最少数量的 catch 块，通过使用基类异常类来捕获它们。下面的示例中，catch 块使用`Exception`类来捕获多个异常。在这里，我们假设可能会抛出多个已检查的异常，并且需要处理它们：

```java
try {
   …
}
catch (Exception e) {
   …
} 
```

如果它们都以完全相同的方式处理，那可能没问题。但是，如果它们在处理方式上有所不同，那么我们需要包含额外的逻辑来确定实际发生了什么。如果我们忽略了这些差异，那么它可能会使任何调试过程更加困难，因为我们可能已经丢失了有关异常的有用信息。此外，这种方法不仅太粗糙，而且我们还捕获了所有的 `RuntimeException`，而这些可能无法处理。

相反，通常最好在它们自己的捕获块中捕获多个异常，如下面的代码片段所示：

```java
try {
   …
}
catch (Exception1 e1) {
   …
} 
catch (Exception1 e2) {
   …
} 
catch (Exception1 e3) {
   …
} 
catch (Exception1 e4) {
   …
} 
```

## 记录异常

通常的做法是即使成功处理了异常，也要记录异常。这对评估应用程序的行为很有用。当然，如果我们无法处理异常并需要优雅地终止应用程序，错误日志可以帮助确定应用程序出了什么问题。

### 注意

异常只记录一次。多次记录可能会让试图查看发生了什么的人感到困惑，并创建比必要更大的日志文件。

## 不要使用异常来控制正常的逻辑流程

在应该进行验证的地方使用异常是不好的做法。此外，抛出异常会消耗额外的资源。例如，`NullPointerException` 是一种常见的异常，当尝试对一个具有空值分配的引用变量执行方法时会出现。我们应该检测这种情况并在正常的逻辑序列中处理它，而不是捕获这个异常。考虑以下情况，我们捕获了一个 `NullPointerException`：

```java
String state = ...  // Somehow assigned a null value
try {
   if(state.equals("Ready") { … }
}
catch(NullPointerException e) {
   // Handle null state
}
```

相反，在使用状态变量之前应该检查它的值：

```java
String state = ...  // Somehow assigned a null value

if(state != null) {
   if(state.equals("Ready") { … }
} else {
   // Handle null state
}
```

完全消除了 `try` 块的需要。另一种方法使用短路评估，如下面的代码片段所示，并在 第三章 的 *决策结构* 部分进行了介绍。如果 `state` 变量为空，则避免使用 `equals` 方法：

```java
String state = ...  // Somehow assigned a null value

if(state != null && state.equals("Ready") { 
   // Handle ready state
} else {
   // Handle null state
}
```

## 不要尝试处理未经检查的异常

通常不值得花费精力处理未经检查的异常。这些大多数是程序员无法控制的，并且需要大量的努力才能从中恢复。例如，`ArrayIndexOutOfBoundsException`，虽然是编程错误的结果，但在运行时很难处理。假设修改数组索引变量是可行的，可能不清楚应该为其分配什么新值，或者如何重新执行有问题的代码序列。

### 注意

永远不要捕获 `Throwable` 或 `Error` 异常。这些异常不应该被处理或抑制。

# 总结

程序中的正确异常处理将增强其健壮性和可靠性。`try`、`catch` 和 `finally` 块可用于在应用程序中实现异常处理。在 Java 7 中，添加了 try-with-resources 块，更容易处理资源的打开和关闭。还可以将异常传播回调用序列。

我们学到了捕获块的顺序很重要，以便正确处理异常。此外，`|` 运算符可以在捕获块中使用，以相同的方式处理多个异常。

异常处理可能嵌套以解决在捕获块或 finally 块中的代码可能也会抛出异常的问题。当这种情况发生时，程序员需要小心确保之前的异常不会丢失，并且新的异常会得到适当处理。

我们还解决了处理异常时可能出现的一些常见问题。它们提供了避免结构不良和容易出错的代码的指导。这包括在异常发生时不要忽略异常，并在适当的级别处理异常。

现在我们已经了解了异常处理过程，我们准备在下一章结束我们对 Java 认证目标的覆盖。

# 涵盖的认证目标

本章涵盖的认证目标包括：

+   描述 Java 中异常的用途

+   区分已检查的异常、运行时异常和错误

+   创建一个 try-catch 块，并确定异常如何改变正常的程序流程

+   调用一个抛出异常的方法

+   识别常见的异常类和类别

# 测试你的知识

1.  以下哪些实现了已检查的异常？

a. `Class A extends RuntimeException`

b. `Class A extends Throwable`

c. `Class A extends Exception`

d. `Class A extends IOException`

1.  给定以下一组类：

`class Exception A extends Exception {}`

`class Exception B extends A {}`

`class Exception C extends A {}`

`class Exception D extends C {}`

以下`try`块的 catch 块的正确顺序是什么？

```java
try {
   // method throws an exception of the above types
}
```

a. 捕获`A`、`B`、`C`和`D`

b. 捕获`D`、`C`、`B`和`A`

c. 捕获`D`、`B`、`C`和`A`

d. 捕获`C`、`D`、`B`和`A`

1.  以下哪些陈述是真的？

a. 已检查的异常是从`Error`类派生的异常。

b. 应该通常忽略已检查的异常，因为我们无法处理它们。

c. 必须重新抛出已检查的异常。

d. 应该在调用堆栈中的适当方法中处理已检查的异常。

1.  当一个方法抛出一个已检查的异常时，以下哪些是有效的响应？

a. 将方法放在 try-catch 块中。

b. 不要使用这些类型的方法。

c. 通常无法处理已检查的异常，因此不做任何处理。

d. 在调用这个方法的方法上使用`throws`子句。

1.  以下代码可能在运行时生成什么异常？

```java
String s;
int i = 5;
try{
   i = i/0;
   s += "next";
}
```

a. `ArithmeticException`

b. `DivisionByZeroException`

c. `FileNotFoundException`

d. `NullPointerException`
