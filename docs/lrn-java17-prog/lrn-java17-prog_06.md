# 第四章：异常处理

在*第一章*，*Java 17 入门*中，我们简要介绍了异常。在本章中，我们将更系统地探讨这个主题。Java 中有两种异常：检查型和非检查型。我们将演示每种类型，并解释两者之间的区别。此外，你还将了解与异常处理相关的 Java 构造的语法以及处理这些异常的最佳实践。本章将以断言语句的相关主题结束，断言语句可用于在生产环境中调试代码。

在本章中，我们将涵盖以下主题：

+   Java 异常框架

+   检查型和非检查型（运行时）异常

+   `try`、`catch`和`finally`块

+   `throws`语句

+   `throw`语句

+   `assert`语句

+   异常处理的最佳实践

那么，让我们开始吧！

# 技术要求

要能够执行本章提供的代码示例，你需要以下内容：

+   搭载 Microsoft Windows、Apple macOS 或 Linux 操作系统的计算机

+   Java SE 版本 17 或更高

+   你偏好的 IDE 或代码编辑器

在*第一章*，*Java 17 入门*中提供了如何设置 Java SE 和 IntelliJ IDEA 编辑器的说明。本章的代码示例文件可在 GitHub 的[`github.com/PacktPublishing/Learn-Java-17-Programming.git`](https://github.com/PacktPublishing/Learn-Java-17-Programming.git)仓库中找到。请在`examples/src/main/java/com/packt/learnjava/ch04_exceptions`文件夹中搜索。

# Java 异常框架

如*第一章*，*Java 17 入门*所述，一个意外的条件可以导致`catch`子句，即如果异常是在`try`块内部抛出的。让我们看一个例子。考虑以下方法：

```java
void method(String s){
```

```java
    if(s.equals("abc")){
```

```java
        System.out.println("Equals abc");
```

```java
    } else {
```

```java
        System.out.println("Not equal");
```

```java
    }
```

```java
}
```

如果输入参数值为`null`，你可能会看到输出为`Not equal`。不幸的是，情况并非如此。`s.equals("abc")`表达式在由`s`变量引用的对象上调用`equals()`方法；然而，如果`s`变量为`null`，它不指向任何对象。让我们看看接下来会发生什么。

让我们运行以下代码（即`Framework`类中的`catchException1()`方法）：

```java
try {
```

```java
    method(null);
```

```java
} catch (Exception ex){
```

```java
    System.out.println("catchException1():");
```

```java
    System.out.println(ex.getClass().getCanonicalName());  
```

```java
                       //prints: java.lang.NullPointerException
```

```java
    waitForStackTrace();
```

```java
    ex.printStackTrace();  //prints: see the screenshot
```

```java
    if(ex instanceof NullPointerException){
```

```java
        //do something
```

```java
    } else {
```

```java
        //do something else
```

```java
    }
```

```java
}
```

上述代码包括`waitForStackTrace()`方法，允许你等待一段时间直到生成堆栈跟踪。否则，输出将会顺序混乱。此代码的输出如下所示：

```java
catchException1():                                                  
java.lang.NullPointerException                                      
java.lang.NullPointerException: Cannot invoke "String.equals(Object)" because "s" is null                                                 
     at com.packt.learnjava.ch04_exceptions.Framework.method(Framework.java:14)
     at com.packt.learnjava.ch04_exceptions.Framework.catchException1(Framework.java:24)
     at com.packt.learnjava.ch04_exceptions.Framework.main(Framework.java:8)
```

如您所见，该方法打印出异常类的名称，然后是**堆栈跟踪**。**堆栈跟踪**这个名字来源于方法调用在 JVM 内存中存储的方式（作为一个堆栈）：一个方法调用另一个方法，然后这个方法又调用另一个方法，依此类推。在最内层方法返回后，堆栈被回溯，返回的方法（**堆栈帧**）从堆栈中移除。我们将在*第九章*中更详细地讨论 JVM 内存结构，*JVM 结构和垃圾回收*。当发生异常时，所有堆栈内容（如堆栈帧）都作为堆栈跟踪返回。这使我们能够追踪到导致问题的代码行。

在前面的代码示例中，根据异常的类型执行了不同的代码块。在我们的情况下，它是`java.lang.NullPointerException`。如果应用程序代码没有捕获它，这个异常就会通过被调用方法的堆栈传播到 JVM，然后 JVM 将停止执行应用程序。为了避免这种情况发生，可以捕获异常并执行代码以从异常状态中恢复。

Java 中异常处理框架的目的是保护应用程序代码免受意外情况的影响，并在可能的情况下从该情况中恢复。在接下来的章节中，我们将更详细地剖析这个概念，并使用框架功能重写给定的示例。

# 检查型异常和非检查型异常

如果你查阅`java.lang`包 API 的文档，你会发现该包包含近三十个异常类和几十个错误类。这两组都扩展了`java.lang.Throwable`类，继承其所有方法，并不添加其他方法。在`java.lang.Throwable`类中最常使用的方法包括以下内容：

+   `void printStackTrace()`: 这将输出方法调用的堆栈跟踪（堆栈帧）。

+   `StackTraceElement[] getStackTrace()`: 这返回与`printStackTrace()`相同的信息，但允许以编程方式访问堆栈跟踪的任何帧。

+   `String getMessage()`: 这将检索通常包含对异常或错误原因的友好解释的消息。

+   `Throwable getCause()`: 这将检索一个可选的`java.lang.Throwable`对象，它是异常的原始原因（但代码的作者决定将其包装在另一个异常或错误中）。

所有错误都扩展了`java.lang.Error`类，该类反过来又扩展了`java.lang.Throwable`类。通常，错误由 JVM 抛出，根据官方文档，*表示合理的应用程序不应该尝试捕获的严重问题*。以下是一些例子：

+   `OutOfMemoryError`: 当 JVM 耗尽内存且无法使用垃圾回收清理内存时抛出。

+   `StackOverflowError`：当为方法调用栈分配的内存不足以存储另一个栈帧时，会抛出此异常。

+   `NoClassDefFoundError`：当 JVM 无法找到当前加载的类请求的类定义时，会抛出此异常。

框架的作者假设应用程序无法自动从这些错误中恢复，这证明是一个大体正确的假设。这就是为什么程序员通常不捕获错误，但这超出了本书的范围。

另一方面，异常通常与特定于应用程序的问题相关，并且通常不需要我们关闭应用程序并允许恢复。通常，这就是程序员捕获它们并实现应用程序逻辑的替代（主流程）路径，或者至少在不关闭应用程序的情况下报告问题。以下是一些示例：

+   `ArrayIndexOutOfBoundsException`：当代码尝试通过等于或大于数组长度的索引访问元素时，会抛出此异常（请记住，数组的第一个元素具有索引`0`，因此索引等于数组长度指向数组外部）。

+   `ClassCastException`：当代码尝试将引用转换为与变量所引用的对象不关联的类或接口时，会抛出此异常。

+   `NumberFormatException`：当代码尝试将字符串转换为数值类型，但字符串不包含必要的数字格式时，会抛出此异常。

所有异常都扩展了`java.lang.Exception`类，而该类反过来又扩展了`java.lang.Throwable`类。这就是为什么通过捕获`java.lang.Exception`类的对象，代码可以捕获任何异常类型的对象。在*Java 异常框架*部分，我们通过以相同的方式捕获`java.lang.NullPointerException`来演示了这一点。

其中一个异常是`java.lang.RuntimeException`。扩展它的异常被称为`NullPointerException`、`ArrayIndexOutOfBoundsException`、`ClassCastException`和`NumberFormatException`。它们被称为运行时异常的原因很明显；它们被称为非受检异常的原因将在下一节中变得清晰。

那些没有`java.lang.RuntimeException`在其祖先中的异常被称为方法的`throws`子句（请参阅*抛出语句*部分）。这种设计迫使程序员做出一个有意识的决策，要么捕获受检异常，要么通知方法的客户端该方法可能会抛出此异常，并且必须由客户端处理（处理）。以下是一些受检异常的示例：

+   `ClassNotFoundException`：当尝试使用`Class`类的`forName()`方法通过字符串名称加载一个类失败时，会抛出此异常。

+   `CloneNotSupportedException`：当代码尝试克隆一个没有实现`Cloneable`接口的对象时，会抛出此异常。

+   `NoSuchMethodException`：当代码中没有方法被调用时抛出。

并非所有异常都位于`java.lang`包中。许多其他包包含与包支持的功能相关的异常。例如，存在一个`java.util.MissingResourceException`运行时异常和一个`java.io.IOException`检查型异常。

尽管没有强制要求，程序员经常捕获运行时（非检查型）异常以更好地控制程序流程，使应用程序的行为更加稳定和可预测。顺便说一下，所有错误也都是运行时（非检查型）异常。然而，正如我们之前提到的，通常无法通过程序来处理它们，因此捕获`java.lang.Error`类的子类没有意义。

# try、catch 和 finally 块

当在`try`块内部抛出异常时，它将控制流重定向到第一个`catch`子句。如果没有可以捕获异常的`catch`块（但必须存在`finally`块），异常将向上传播并退出方法。如果有多个`catch`子句，编译器会强制你按照子异常在父异常之前列出的顺序排列它们。让我们看看以下示例：

```java
void someMethod(String s){
```

```java
    try {
```

```java
       method(s);
```

```java
    } catch (NullPointerException ex){
```

```java
       //do something
```

```java
    } catch (Exception ex){
```

```java
       //do something else
```

```java
    }
```

```java
}
```

在前面的示例中，一个带有`NullPointerException`的`catch`块被放置在带有`Exception`的块之前，因为`NullPointerException`扩展了`RuntimeException`，而`RuntimeException`又扩展了`Exception`。我们甚至可以按照以下方式实现这个示例：

```java
void someMethod(String s){
```

```java
    try {
```

```java
        method(s);
```

```java
    } catch (NullPointerException ex){
```

```java
        //do something
```

```java
    } catch (RuntimeException ex){
```

```java
        //do something else
```

```java
    } catch (Exception ex){
```

```java
        //do something different
```

```java
    }
```

```java
}
```

注意，第一个`catch`子句仅捕获`NullPointerException`。其他扩展`RuntimeException`的异常将由第二个`catch`子句捕获。其余的异常类型（即所有检查型异常）将由最后的`catch`块捕获。注意，错误不会被这些`catch`子句捕获。要捕获它们，你应该添加一个`catch`子句来捕获`Error`（在任何位置）或`Throwable`（在前一个示例中的最后一个`catch`子句之后）。然而，通常程序员不会这样做，并允许错误传播到 JVM。

为每种异常类型都拥有一个`catch`块允许我们提供特定的异常类型处理。然而，如果没有异常处理上的差异，你可以简单地使用一个带有`Exception`基类的`catch`块来捕获所有类型的异常：

```java
void someMethod(String s){
```

```java
    try {
```

```java
        method(s);
```

```java
    } catch (Exception ex){
```

```java
        //do something
```

```java
    }
```

```java
}
```

如果没有任何子句捕获异常，它将被进一步抛出，直到它被方法调用者中的一个`try...catch`语句处理，或者传播到应用程序代码之外。在这种情况下，JVM 将终止应用程序并退出。

添加`finally`块不会改变描述的行为。如果存在，它总是会被执行，无论是否已生成异常。通常，`finally`块用于释放资源，关闭数据库连接、文件或类似资源。然而，如果资源实现了`Closeable`接口，最好使用`try-with-resources`语句，它允许你自动释放资源。以下是如何使用 Java 7 实现的示例：

```java
try (Connection conn = DriverManager
```

```java
               .getConnection("dburl", "username", "password");
```

```java
     ResultSet rs = conn.createStatement()
```

```java
               .executeQuery("select * from some_table")) {
```

```java
    while (rs.next()) {
```

```java
        //process the retrieved data
```

```java
    }
```

```java
} catch (SQLException ex) {
```

```java
    //Do something
```

```java
    //The exception was probably caused 
```

```java
    //by incorrect SQL statement
```

```java
}
```

上述示例创建了数据库连接，检索数据并处理它，然后关闭（调用`close()`方法）`conn`和`rs`对象。

Java 9 通过允许在`try`块外部创建表示资源的对象，并在`try-with-resources`语句中使用它们，增强了`try-with-resources`语句的功能，如下所示：

```java
void method(Connection conn, ResultSet rs) {
```

```java
    try (conn; rs) {
```

```java
        while (rs.next()) {
```

```java
            //process the retrieved data
```

```java
        }
```

```java
    } catch (SQLException ex) {
```

```java
        //Do something
```

```java
        //The exception was probably caused 
```

```java
        //by incorrect SQL statement
```

```java
    }
```

```java
}
```

上一段代码看起来更简洁，尽管在实践中，程序员更喜欢在相同上下文中创建和释放（关闭）资源。如果这也是你的偏好，考虑使用`throws`语句与`try-with-resources`语句结合使用。

# 抛出语句

我们必须处理`SQLException`，因为它是一个检查型异常，`getConnection()`、`createStatement()`、`executeQuery()`和`next()`方法都在它们的`throws`子句中声明了它。以下是一个示例：

```java
Statement createStatement() throws SQLException;
```

这意味着方法的作者警告方法的使用者，它可能会抛出这样的异常，迫使他们要么捕获异常，要么在它们方法的`throws`子句中声明它。在我们的前一个例子中，我们选择使用两个`try...catch`语句来捕获它。或者，我们可以在`throws`子句中列出异常，从而通过有效地将异常处理的负担推给我们的方法的使用者来清除杂乱：

```java
void throwsDemo() throws SQLException {
```

```java
    Connection conn = 
```

```java
      DriverManager.getConnection("url","user","pass");
```

```java
    ResultSet rs = conn.createStatement().executeQuery(
```

```java
      "select * ...");
```

```java
    try (conn; rs) {
```

```java
        while (rs.next()) {
```

```java
            //process the retrieved data
```

```java
        }
```

```java
    } finally { 
```

```java
        try {
```

```java
           if(conn != null) {
```

```java
              conn.close();
```

```java
           }
```

```java
        } finally {
```

```java
           if(rs != null) {
```

```java
               rs.close();
```

```java
           }
```

```java
        }
```

```java
    }
```

```java
}
```

我们去掉了捕获子句，但我们需要`finally`块来关闭创建的 conn 和 rs 对象。

请注意，我们如何在 try 块中包含了关闭 conn 对象的代码，在 finally 块中包含了关闭 rs 对象的代码。这样我们确保在关闭 conn 对象时发生的异常不会阻止我们关闭 rs 对象。

这段代码比我们在上一节中演示的`try-with-resources`语句看起来更不清楚。我们展示它只是为了演示所有可能性以及如何避免可能的危险（不关闭资源），如果你决定自己这样做，而不是让`try-with-resources`语句自动为你做。

但让我们回到对`throws`语句的讨论。

`throws`子句允许但不强制我们列出未检查的异常。添加未检查的异常不会强制方法的使用者处理它们。

最后，如果方法抛出几个不同的异常，可以列出基类 `Exception` 异常类，而不是列出所有异常。这将使编译器高兴；然而，这并不被认为是一种好做法，因为它隐藏了方法用户可能期望的特定异常的细节。

请注意，编译器不会检查方法体中的代码可以抛出哪种类型的异常。因此，可以在 `throws` 子句中列出任何异常，这可能会导致不必要的开销。如果程序员错误地将一个永远不会实际抛出的检查型异常包含在 `throws` 子句中，方法用户可能会为它编写一个永远不会执行的 `catch` 块。

# throw 语句

`throw` 语句允许抛出程序员认为必要的任何异常。您甚至可以创建自己的异常。要创建一个检查型异常，如下扩展 `java.lang.Exception` 类：

```java
class MyCheckedException extends Exception{
```

```java
    public MyCheckedException(String message){
```

```java
        super(message);
```

```java
    }
```

```java
    //add code you need to have here
```

```java
}
```

同样，要创建一个未检查型异常，如下扩展 `java.lang.RunitmeException` 类：

```java
class MyUncheckedException extends RuntimeException{
```

```java
    public MyUncheckedException(String message){
```

```java
        super(message);
```

```java
    }
```

```java
    //add code you need to have here
```

```java
}
```

注意到 *在此处添加您需要的代码* 注释。您可以将方法和属性添加到自定义异常中，就像添加任何其他常规类一样，但程序员很少这样做。事实上，最佳实践明确建议避免使用异常来驱动业务逻辑。异常应该是其名称所暗示的，仅覆盖异常或非常罕见的情况。

然而，如果您需要宣布一个异常条件，请使用 `throw` 关键字和 `new` 运算符来创建和触发异常对象的传播。以下是一些示例：

```java
throw new Exception("Something happened"); 
```

```java
throw new RuntimeException("Something happened");
```

```java
throw new MyCheckedException("Something happened");
```

```java
throw new MyUncheckedException("Something happened");
```

甚至可以抛出 `null`，如下所示：

```java
throw null;
```

前一个语句的结果与这个语句的结果相同：

```java
throw new NullPointerException;
```

在这两种情况下，一个未检查的 `NullPointerException` 异常对象开始在整个系统中传播，直到它被应用程序或 JVM 捕获。

# assert 语句

有时，程序员需要知道代码中是否发生了特定的条件，即使应用程序已经部署到生产环境中。同时，也没有必要一直运行这个检查。这就是 `assert` 分支语句派上用场的地方。以下是一个示例：

```java
public someMethod(String s){
```

```java
    //any code goes here
```

```java
    assert(assertSomething(x, y, z));
```

```java
    //any code goes here
```

```java
}
```

```java
boolean assertSomething(int x, String y, double z){
```

```java
    //do something and return boolean
```

```java
}
```

在前面的代码中，`assert()` 方法从 `assertSomething()` 方法获取输入。如果 `assertSomething()` 方法返回 `false`，则程序停止执行。

`assert()` 方法仅在 JVM 使用 `-ea` 选项运行时执行。在生产环境中不应使用 `-ea` 标志，除非可能是为了测试目的而临时使用。这是因为它会产生额外的开销，从而影响应用程序的性能。

# 异常处理最佳实践

已检查异常是为了在应用程序可以自动采取措施修正或绕过问题时使用的可恢复条件而设计的。在实践中，这种情况并不常见。通常，当捕获到异常时，应用程序会记录堆栈跟踪并中止当前操作。根据记录的信息，应用程序支持团队修改代码以解决未记录的条件或防止其未来发生。

每个应用程序都是不同的，因此最佳实践取决于特定的应用程序需求、设计和上下文。一般来说，开发社区似乎达成了一致，避免使用已检查异常并尽量减少它们在应用程序代码中的传播。以下是一些已被证明有用的其他建议列表：

+   总是捕获所有接近源头的已检查异常。

+   如果有疑问，捕获接近源头的未检查异常。

+   尽可能地在源头附近处理异常，因为那里的上下文最为具体，根本原因也位于那里。

+   除非你真的需要，否则不要抛出已检查异常，因为这会强制为可能永远不会发生的情况构建额外的代码。

+   如果你必须，通过将它们作为带有相应消息的`RuntimeException`重新抛出，将第三方已检查异常转换为未检查异常。

+   除非你真的需要，否则不要创建自定义异常。

+   除非你真的需要，否则不要通过异常处理机制来驱动业务逻辑。

+   通过使用消息系统和可选的`enum`类型而不是使用异常类型来传达错误原因，自定义通用的`RuntimeException`异常。

有许多其他可能的提示和建议；然而，如果你遵循这些，你很可能在大多数情况下都会做得很好。因此，我们结束这一章。

# 摘要

在本章中，你被介绍了 Java 异常处理框架，并学习了两种类型的异常——已检查的和未检查的（运行时）——以及如何使用`try-catch-finally`和`throws`语句来处理它们。你还学习了如何生成（抛出）异常以及如何创建自己的（自定义）异常。本章以异常处理的最佳实践结束，如果始终如一地遵循这些实践，将有助于你编写干净、清晰的代码，这样的代码易于编写、理解和维护。

在下一章中，我们将详细讨论字符串及其处理，以及输入/输出流和文件读写技术。

# 问答

1.  什么是堆栈跟踪？选择所有适用的：

    1.  当前加载的类列表

    1.  当前正在执行的列表

    1.  当前正在执行的代码行列表

    1.  当前使用的变量列表

1.  有哪些类型的异常？选择所有适用的：

    1.  编译异常

    1.  运行时异常

    1.  读取异常

    1.  编写异常

1.  以下代码的输出是什么？

    ```java
    try {
        throw null;
    } catch (RuntimeException ex) {
        System.out.print("RuntimeException ");
    } catch (Exception ex) {
        System.out.print("Exception ");
    } catch (Error ex) {
        System.out.print("Error ");
    } catch (Throwable ex) {
        System.out.print("Throwable ");
    } finally {
        System.out.println("Finally ");
    }
    ```

    1.  一个`RuntimeException`错误

    1.  `Exception Error Finally`

    1.  `RuntimeException Finally`

    1.  `Throwable Finally`

1.  以下哪个方法可以无错误地编译？

    ```java
    void method1() throws Exception { throw null; }
    void method2() throws RuntimeException { throw null; }
    void method3() throws Throwable { throw null; }
    void method4() throws Error { throw null; }
    ```

    1.  `method1()`

    1.  `method2()`

    1.  `method3()`

    1.  `method4()`

1.  以下哪个语句可以无错误地编译？

    ```java
    throw new NullPointerException("Hi there!"); //1
    throws new Exception("Hi there!");          //2
    throw RuntimeException("Hi there!");       //3
    throws RuntimeException("Hi there!");     //4
    ```

    1.  1

    1.  2

    1.  3

    1.  4

1.  假设`int x = 4`，以下哪个语句可以无错误地编译？

    ```java
    assert (x > 3); //1
    assert (x = 3); //2
    assert (x < 4); //3
    assert (x = 4); //4
    ```

    1.  1

    1.  2

    1.  3

    1.  4

1.  以下列表中哪些是最佳实践？

    1.  总是捕获所有异常和错误。

    1.  总是捕获所有异常。

    1.  永远不要抛出未检查的异常。

    1.  除非你不得不这样做，否则尽量不要抛出受检异常。
