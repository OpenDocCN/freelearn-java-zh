# 第九章：*第九章*

# 异常处理

## 学习目标

到本课程结束时，您将能够：

+   使用抛出异常的库

+   有效使用异常处理

+   以一种尊重异常的方式获取和释放资源，而不会造成泄漏

+   实施最佳实践以在 Java 中引入异常

## 介绍

异常处理是一种处理代码运行时发生错误情况的强大机制。它使我们能够专注于程序的主要执行，并将错误处理代码与预期执行路径分开。Java 语言强制程序员为库方法编写异常处理代码，而诸如 IntelliJ、Eclipse 等的 IDE 则帮助我们生成必要的样板代码。然而，如果没有适当的指导和理解，标准的异常代码可能会带来更多的害处。本课程是异常的实际介绍，将促使您思考异常处理的各个方面，并提供一些在处理编程生活中的异常时可能有帮助的经验法则。

## 异常背后的动机

当我们创建程序时，通常会关注预期的情况。例如，我们将从某处获取数据，我们将从数据中提取我们假定存在的某些信息，然后将其发送到其他地方，依此类推。我们希望我们的代码能够清晰可读，这样我们团队的成员可以清楚地理解业务逻辑，并且可以发现我们可能犯的错误。然而，在实践中，我们的假设可能不成立，预期情况可能会出现偏差。例如，由于网络或磁盘出现问题，我们可能无法获取数据。我们可能会收到不符合我们假设的数据。或者，由于类似的问题，我们可能无法发送数据。我们必须创建能够在意外情况下优雅地运行的程序。例如：我们应该让用户在网络连接中断时能够重试。异常是我们在 Java 中处理这种情况的方式，而不会使我们的代码过于复杂。

作为程序员，我们必须编写能够在各种意外情况下正常运行的代码。然而，我们也希望我们的代码干净且易于理解。这两个目标经常会相互竞争。

我们希望编写的代码能够清晰地阅读，如下所示：

```java
Do step 1
Do step 2
Do step 3
Done
```

这反映了一个乐观的情景，即没有发生意外情况。然而，通常情况下会发生意外情况。用户的互联网连接可能中断，网络资源可能中断，客户端可能耗尽内存，可能发生磁盘错误，等等。除非我们编写能够预见这些问题的代码，否则当出现这些问题时，我们的程序可能会崩溃。预见每种可能发生的问题可能会非常困难。即使我们简化事情并以相同的方式处理大多数错误，我们仍然可能需要对我们的代码进行许多检查。例如：我们可能不得不编写更像这样的代码：

```java
Do step 1
If there was a problem with step 1, 
     Handle the error, stop
Else 
    Do step 2
    If there was a problem with step 2, 
           Handle the error, stop
    Else 
           Do step 3
           If there was a problem with step 3
                 Handle the error, stop
           Else
Done
```

您可以提出替代的代码结构，但一旦您在每个步骤中加入额外的错误处理代码，您的代码就会变得不那么可读，不那么易于理解，也不那么易于维护。如果您不包括这样的错误处理代码，您的程序可能会导致意外情况，例如崩溃。

以下是一个在 C 中处理错误类似于我们之前的伪代码的函数。

```java
int other_idea()
{
    int err = minor_func1();
    if (!err)
        err = minor_func2();
    if (!err)
        err = minor_func3();
    return err;
}
```

当您使用诸如 C 之类的原始语言编写代码时，您不可避免地会感到可读性和完整性之间的紧张关系。幸运的是，在大多数现代编程语言中，我们有异常处理能力，可以减少这种紧张关系。您的代码既可以清晰可读，又可以同时处理错误。

异常处理背后的主要语言构造是 try-catch 块。在 try 之后放置的代码逐行执行。如果任何一行导致错误，try 块中的其余行将不会执行，执行将转到 catch 块，让您有机会优雅地处理错误。在这里，您会收到一个包含有关问题详细信息的异常对象。但是，如果 try 块中没有发生错误，catch 块将不会执行。

在这里，我们修改了我们最初的示例，使用 try-catch 块来处理错误，而不是使用许多 if 语句：

```java
Try
Do step 1
Do step 2
Do step 3
Catch error
    Handle error appropriately
Done
```

在这个版本中，我们的代码被放置在 try 和 catch 关键字之间。我们的代码没有错误处理代码，否则会影响可读性。代码的默认预期路径非常清晰：步骤 1，步骤 2 和步骤 3。然而，如果发生错误，执行立即转移到 catch 块。在那里，我们会收到关于问题的信息，以异常对象的形式，并有机会优雅地处理错误。

大多数情况下，您的代码片段会相互依赖。因此，如果一个步骤发生错误，通常不希望执行其余的步骤，因为它们依赖于较早步骤的成功。您可以创造性地使用 try-catch 块来表示代码依赖关系。例如：在以下伪代码中，步骤 2 和步骤 5 中存在错误。成功执行的步骤是步骤 1 和步骤 4。由于步骤 4 和后续步骤与前三个步骤的成功无关，我们能够使用两个单独的 try-catch 块来表示它们的依赖关系。步骤 2 中的错误阻止了步骤 3 的执行，但没有阻止步骤 4 的执行：

```java
Try
Do step 1
Do step 2 - ERROR
Do step 3
Catch error
    Handle error appropriately
Done
Try
Do step 4
Do step 5 - ERROR
Do step 6
Catch error
    Handle error appropriately
Done
```

如果发生异常而您没有捕获它，错误将传播到调用者。如果这是您的应用程序，您不应该让错误传播出您的代码，以防止应用程序崩溃。但是，如果您正在开发一个被其他代码调用的库，有时让错误传播到调用者是一个好主意。我们将在稍后更详细地讨论这个问题。

### 练习 36：引入异常

现在让我们实际看看异常的作用。其中一个经典的异常是尝试用零除以一个数字。在这里，我们将使用它来创建异常并验证我们之前的伪代码：

1.  创建一个新的 Main 类，并添加如下的主方法：

```java
public class Main {
   public static void main(String[] args) { 
```

1.  编写代码来打印两个数字的除法结果。添加 try-catch 块来处理异常：

```java
try {
System.out.println("result 1: " + (2 / 2));
System.out.println("result 2: " + (4 / 0));
System.out.println("result 3: " + (6 / 2));
    } catch (ArithmeticException e) {
System.out.println("---- An exception in first block");
}
try {
System.out.println("result 4: " + (8 / 2));
System.out.println("result 5: " + (10 / 0));
System.out.println("result 6: " + (12 / 2));
} catch (ArithmeticException e) {
System.out.println("---- An exception in second block");
}
}
}
```

运行代码并验证输出是否如下所示：

```java
result 1: 1
---- An exception in block 1
result 4: 4
---- An exception in block 2
```

请注意，结果 2 和 5 包含除以零的除法运算，这将导致异常。这样，我们有意在这两行中创建异常，以查看在异常情况下执行的进展。以下是预期执行的详细情况：

+   结果 1 应该正常打印。

+   在结果 2 的执行过程中，我们应该得到一个异常，这应该阻止结果 2 的打印。

+   由于异常，执行应该跳转到 catch 块，这应该阻止结果 3 的打印。

+   结果 4 应该正常打印。

+   就像结果 2 一样，在结果 5 的执行过程中，我们应该得到一个异常，这应该阻止结果 5 的打印。

+   同样，由于异常，执行应该跳转到 catch 块，这应该阻止结果 6 的打印。

借助两个 try-catch 块的帮助，由于结果 2 和 5 的异常，我们应该跳过结果 3 和 6。这应该只留下结果 1 和 4，它们将成功执行。

这表明我们之前的讨论是正确的。另外，为了验证执行顺序，请在结果 1 行中设置断点，然后单击“逐步执行”以观察执行如何逐步进行，使用 try-catch 块。

通过异常和`try-catch`块的帮助，我们能够编写更专注于预期的默认执行路径的代码，同时确保我们处理意外的错误情况，并根据错误的严重程度进行恢复或优雅失败。

### 异常的不可避免介绍

实际上，大多数新手 Java 开发者在调用库中抛出异常的方法时会遇到异常。这样的方法可以使用 throws 语句指定它会抛出异常。当你调用这种方法时，除非你编写处理该方法可能抛出的异常的代码，否则你的代码将无法编译。

因此，作为一个新手 Java 开发者，你所想要的只是调用一个方法，现在你被迫处理它可能抛出的异常。你的 IDE 可以生成处理异常的代码。然而，默认生成的代码通常不是最好的。一个没有指导的新手和 IDE 代码生成的能力可能会创建相当糟糕的代码。在本节中，你将得到如何最好地使用 IDE 生成的异常处理代码的指导。

假设你写了以下代码来打开和读取一个文件：

```java
import java.io.File;
import java.io.FileInputStream;
public class Main {
   public static void main(String[] args) {
       File file = new File("./tmp.txt");
       FileInputStream inputStream = new FileInputStream(file);
   }
}
```

目前，你的代码将无法编译，你的 IDE 用红色下划线标出了`FileInputStream`构造函数。这是因为它可能会抛出异常，就像在它的源代码中指定的那样：

```java
public FileInputStream(File file) throws FileNotFoundException {
```

在这一点上，你的 IDE 通常会试图提供帮助。例如，当你将光标移动到`FileInputStream`上并在 IntelliJ 中按下*Alt* + *Enter*时，你会看到两个快速修复选项：**在方法签名中添加异常**和**用 try/catch 包围**。这对应于处理指定异常时你所拥有的两个选项，我们稍后会更深入地学习。第一个选项将你的代码转换为以下内容：

```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
public class Main {
   public static void main(String[] args) throws FileNotFoundException {
       File file = new File("input.txt");
       FileInputStream inputStream = new FileInputStream(file);
   }
}
```

现在你的主函数也指定了它可能会抛出异常。这样的异常会导致程序立即退出，这可能是你想要的，也可能不是。如果这是一个你作为库提供给其他人的函数，这个改变将阻止他们的代码编译，除非他们反过来处理指定的异常，就像你一样。同样，这可能是你想要做的，也可能不是。

如果你选择了“**用 try/catch 包围**”，这是 IntelliJ 提供的第二个选项，你的代码将变成这样：

```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
public class Main {
   public static void main(String[] args) {
       File file = new File("input.txt");
       try {
           FileInputStream inputStream = new FileInputStream(file);
       } catch (FileNotFoundException e) {
           e.printStackTrace();
       }
   }
}
```

在这个例子中，我们正在编写代码来自己处理异常。这感觉更合适；我们正在承担责任并编写代码来处理异常。然而，当前形式的代码实际上更有害无益。首先，它实际上并没有对异常做任何有用的事情；它只是捕获它，将有关它的信息打印到`stdout`，然后继续执行，就好像什么都没有发生一样。特别是在一个不是控制台应用程序的项目中（像大多数 Java 程序一样），打印到日志几乎没有用处。

如果我们找不到这个文件来打开，我们应该聪明地考虑我们可以做什么。我们应该要求用户查找文件吗？我们应该从互联网上下载吗？无论我们做什么，把问题记录在一个晦涩的日志文件中，然后把问题搁置起来可能是处理问题的最糟糕的方式之一。如果我们无法做任何有用的事情，也许不处理异常，让我们的调用者处理它，可能是更诚实地处理问题的方式。

请注意，这里没有银弹，也没有一刀切的建议。每个特殊情况，每个应用程序，每个上下文和每个用户群体都是不同的，我们应该提出一个最适合当前情况的异常处理策略。然而，如果你所做的只是`e.printStackTrace()`，那你可能做错了什么。

### 练习 37：使用 IDE 生成异常处理代码

在这个练习中，我们将看看如何使用 IDE 生成异常处理代码：

1.  在 IntelliJ 中创建一个新的 Java 控制台项目。导入`File`和`FileInputStream`类：

```java
import java.io.File;
import java.io.FileInputStream;
```

1.  创建一个名为`Main`的类并添加`main()`方法：

```java
public class Main {
   public static void main(String[] args) {
```

1.  按以下方式打开文件：

```java
File file = new File("input.txt");
FileInputStream fileInputStream = new FileInputStream(file);
```

1.  按照以下方式读取文件：

```java
int data = 0;
while(data != -1) {
data = fileInputStream.read();
System.out.println(data);
     }
     fileInputStream.close();
   }
}
```

请注意，在四个地方，IntelliJ 用红色下划线标出了我们的代码。这些是指定抛出异常的函数。这会阻止您的代码执行。

1.  转到第一个问题（`FileInputStream`），按*Alt* + *Enter*，选择"`main`函数可以抛出`FileNotFoundException`，但这还不够，因为这不是其他函数抛出的异常类型。现在转到剩下的第一个问题（`read`），按*Alt* + *Enter*，选择"`input.txt`与此同时，这是您应该看到的输出：

```java
Exception in thread "main" java.io.FileNotFoundException: input.txt (The system cannot find the file specified)
at java.io.FileInputStream.open0(Native Method)
at java.io.FileInputStream.open(FileInputStream.java:195)
at java.io.FileInputStream.<init>(FileInputStream.java:138)
at Main.main(Main.java:9)
```

异常从我们的主函数传播出来，JVM 捕获并记录到控制台中。

两件事情发生了。首先，修复`read()`的问题足以消除代码中的所有问题，因为`read`和`close`都会抛出相同的异常：`IOException`，它在主函数声明的 throws 语句中列出。然而，我们在那里列出的`FileNotFoundException`异常消失了。为什么呢？

这是因为异常类是一个层次结构，`IOException`是`FileNotFoundException`的祖先类。由于每个`FileNotFoundException`也是`IOException`，指定`IOException`就足够了。如果这两个类不是以这种方式相关的，IntelliJ 将列出可能抛出的异常作为逗号分隔的列表。

1.  现在让我们将`input.txt`提供给我们的程序。您可以在硬盘的任何位置创建`input.txt`并在代码中提供完整的路径；但是，我们将使用一个简单的方法：IntelliJ 在主项目文件夹中运行您的程序。在这里右键单击您项目的`input.txt`文件，并在其中写入文本"`abc`"。如果您再次运行程序，您应该会看到类似于这样的输出：

```java
97
98
99
-1
```

1.  指定异常是使我们的程序工作的一种方法。另一种方法是捕获它们。现在让我们尝试一下。返回到您文件的以下版本；您可以重复使用撤消来做到这一点：

```java
import java.io.File;
import java.io.FileInputStream;
public class Main {
   public static void main(String[] args) {
       File file = new File("input.txt");
       FileInputStream fileInputStream = new FileInputStream(file);
       int data = 0;
       while(data != -1) {
          data = fileInputStream.read();
          System.out.println(data);
       }
       fileInputStream.close();
   }
}
```

1.  现在将光标移动到`FileInputStream`，按*Alt* + *Enter*，选择"`try/catch`块，它实际上将引用变量的创建与引发异常的构造函数调用分开。这主要是因为`fileInputStream`稍后在代码中使用，并且将其移动到`try/catch`块内将阻止它对这些用法可见。这实际上是一个常见的模式；您在`try/catch`块之前声明变量，处理其创建的任何问题，并在以后如果需要的话使其可用。

1.  当前代码存在一个问题：如果`try/catch`块内的`FileInputStream`失败，`fileInputStream`将继续为空。在`try`/`catch`块之后，它将被取消引用，您将获得一个空引用异常。您有两个选择：要么将对象的所有用法放在`try/catch`块中，要么检查引用是否为空。以下是两种选择中的第一种：

```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
public class Main {
   public static void main(String[] args) {
       File file = new File("input.txt");
       FileInputStream fileInputStream = null;
       try {
           fileInputStream = new FileInputStream(file);

           int data = 0;
           while(data != -1) {
               data = fileInputStream.read();
               System.out.println(data);
           }
           fileInputStream.close();
       } catch (FileNotFoundException e) {
           e.printStackTrace();
       }
   }
}
```

1.  我们将代码移到`try`/`catch`块内，以确保我们不会在`fileInputStream`为空时取消引用。然而，`read()`和`close()`仍然有红色下划线。在`read()`上按*Alt* + *Enter*会给你一些选项，其中第一个选项是添加一个`catch`子句：

```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
public class Main {
   public static void main(String[] args) {
       File file = new File("input.txt");
       FileInputStream fileInputStream = null;
       try {
           fileInputStream = new FileInputStream(file);
           int data = 0;
           while(data != -1) {
               data = fileInputStream.read();
               System.out.println(data);
           }
           fileInputStream.close();
       } catch (FileNotFoundException e) {
           e.printStackTrace();
       } catch (IOException e) {
           e.printStackTrace();
       }
   }
}
```

现在我们已经解决了代码中的所有问题，我们实际上可以运行它。请注意，第二个 catch 子句放在第一个之后，因为`IOException`是`FileNotFoundException`的父类。如果它们的顺序相反，类型为`FileNotFoundException`的异常实际上将被`IOException`捕获块捕获。

1.  这是两种选择中的第二种选择，不将所有代码放在第一个 try 中：

```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
public class Main {
   public static void main(String[] args) {
       File file = new File("input.txt");
       FileInputStream fileInputStream = null;
       try {
           fileInputStream = new FileInputStream(file);
       } catch (FileNotFoundException e) {
           e.printStackTrace();
       }
       if (fileInputStream != null) {
           int data = 0;
           while(data != -1) {
               data = fileInputStream.read();
               System.out.println(data);
           }
           fileInputStream.close();
       }
   }
}
```

如果`fileInputStream`不为空，我们就运行代码的第二部分。这样，如果创建`FileInputStream`不成功，我们就可以阻止第二部分运行。单独这样写可能没有太多意义，但如果中间有其他不相关的代码，那么这样写就有意义。你不能把所有东西都放在同一个`try`块中，在以后的代码中，你可能会依赖于那个`try`块的成功。这种简单的空值检查在这方面是有用的。

1.  尽管我们的代码仍然存在问题。让我们在`read()`和`close()`上使用*Alt* + *Enter*，并选择`try`/`catch`块。

1.  更好的方法是将整个代码块放在一个`try`/`catch`中。在这种情况下，我们在第一个错误后放弃，这是一个更简单且通常更正确的方法：

```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
public class Main {
   public static void main(String[] args) {
       File file = new File("input.txt");
       FileInputStream fileInputStream = null;
       try {
           fileInputStream = new FileInputStream(file);
       } catch (FileNotFoundException e) {
           e.printStackTrace();
       }
       if (fileInputStream != null) {
           try {
               int data = 0;
               while(data != -1) {
                   data = fileInputStream.read();
                   System.out.println(data);
               }
               fileInputStream.close();
           } catch (IOException e) {
               e.printStackTrace();
           }
       }
   }
}
```

为了创建这段代码，我们没有依赖 IntelliJ 的快速修复功能*Alt* + *Enter*。虽然通常它很好，你可能会认为它创建的代码是正确的。然而，你必须运用自己的判断力，有时要纠正它创建的代码，就像这个例子一样。

现在你已经体验了使用 IDE 快速而简单地处理异常的方法。在这一节中获得的技能应该在你面临截止日期时指导你，并帮助你避免在使用 IDE 生成的异常代码时出现问题。

### 异常与错误代码

回想一下我们之前给出的 C 代码示例：

```java
int other_idea()
{
    int err = minor_func1();
    if (!err)
        err = minor_func2();
    if (!err)
        err = minor_func3();
    return err;            
}
```

这里使用的错误处理方法存在一些缺点。在这段代码中，我们只是尝试调用三个函数。然而，对于每个函数调用，我们都在传递值来跟踪错误状态，并且对于每个函数调用，如果出现错误，都要使用`if`语句。此外，函数的返回值是错误状态——你不能返回自己选择的值。所有这些额外的工作都使原始代码变得模糊，并且难以理解和维护。

这种方法的另一个局限性是，单个整数值可能无法充分表示错误。相反，我们可能希望有关于错误的更多细节，比如发生时间、关于哪个资源等等。

在异常处理之前，程序员们必须编写代码来确保程序的完整性。异常处理带来了许多好处。考虑一下这个替代的 Java 代码：

```java
int otherIdea() {
   try {
       minorFunc1();
       minorFunc2();
       minorFunc3();
   } catch (IOException e) {
       // handle IOException
   } catch (NullPointerException e) {
       // handle NullPointerException
   }
}
```

在这里，我们有三个函数调用，没有任何与错误相关的代码污染它们。这些放在了一个`try`/`catch`块中，错误处理是在`catch`块中单独完成的。出于以下原因，这更加可取：

+   我们不必为每个函数调用都有一个`if`语句。我们可以将异常处理集中在一个地方。不管是哪个函数引发了异常，我们都在一个地方捕获所有异常。

+   一个函数中可能发生的问题不止一种。每个函数可能引发多种异常。这些可以在单独的 catch 块中处理，而不是像没有异常处理那样，这将需要每个函数多个 if 语句。

+   异常由对象表示，而不是单个整数值。虽然整数可以告诉我们出了什么问题，但对象可以告诉我们更多：异常发生时的调用堆栈、相关资源、关于问题的用户可读解释等等，都可以与异常对象一起提供。与单个整数值相比，这使得更容易对异常做出适当的反应。

### 练习 38：异常与错误代码

为了完成关于异常与错误代码的讨论，让我们体验一下两者，看看哪一个更容易处理。在这个练习中，我们有一个类，其中包含两种不同类型的函数，每种函数有两个函数。`thFunction1()`和`thFunction2()`是在发生错误时可以抛出异常的函数。`ecFunction1()`和`ecFunction2()`是返回指示是否发生错误的值的函数。我们使用随机数来模拟有时会发生错误：

1.  导入`IOException`和`Random`类如下：

```java
import java.io.IOException;
import java.util.Random;
```

1.  创建一个名为`Main`的类，其中包含`Random`类的一个实例：

```java
public class Main {
   Random rand = new Random();
```

1.  创建`thFunction1()`和`thFunction2()`函数，它们抛出`IOException`如下：

```java
void thFunction1() throws IOException {
       System.out.println("thFunction1 start");
       if (rand.nextInt(10) < 2) {
           throw new IOException("An I/O exception occurred in thFunction1");
       }
       System.out.println("thFunction1 done");
   }
   void thFunction2() throws IOException, InterruptedException {
       System.out.println("thFunction2 start");
       int r = rand.nextInt(10);
       if (r < 2) {
           throw new IOException("An I/O exception occurred in thFunction2");
       }
       if (r > 8) {
           throw new InterruptedException("An interruption occurred in thFunction2");
       }
       System.out.println("thFunction2 done");
   }
```

1.  声明三个具有最终值的变量如下：

```java
private static final int EC_NONE = 0;
private static final int EC_IO = 1;
private static final int EC_INTERRUPTION = 2;
```

1.  创建两个函数`ecFunction1()`和`ecFunction2()`如下：

```java
int ecFunction1() {
System.out.println("ecFunction1 start");
if (rand.nextInt(10) < 2) {
return EC_IO;
}
System.out.println("thFunction1 done");
return EC_NONE;
}
int ecFunction2() {
System.out.println("ecFunction2 start");
int r = rand.nextInt(10);
if (r < 2) {
return EC_IO;
}
if (r > 8) {
return EC_INTERRUPTION;
}
System.out.println("ecFunction2 done");
       return EC_NONE;
}
```

1.  创建`callThrowingFunctions()`如下：

```java
private void callThrowingFunctions() {
try {
thFunction1();
thFunction2();
} catch (IOException e) {
System.out.println(e.getLocalizedMessage());
e.printStackTrace();
} catch (InterruptedException e) {
System.out.println(e.getLocalizedMessage());
e.printStackTrace();
}
}
```

1.  创建一个名为`callErrorCodeFunctions()`的方法如下：

```java
private void callErrorCodeFunctions() {
int err = ecFunction1();
if (err != EC_NONE) {
if (err == EC_IO) {
System.out.println("An I/O exception occurred in ecFunction1.");
}
}
err = ecFunction2();
switch (err) {
case EC_IO:
System.out.println("An I/O exception occurred in ecFunction2.");
break;
case EC_INTERRUPTION:
System.out.println("An interruption occurred in ecFunction2.");
break;
}
}
```

1.  添加`main`方法如下：

```java
   public static void main(String[] args) {
       Main main = new Main();
       main.callThrowingFunctions();
       main.callErrorCodeFunctions();
   }
}
```

在我们的`main`函数中，我们首先调用抛出函数，然后是错误代码函数。

多次运行此程序，观察每种情况下如何处理错误。以下是使用异常处理捕获错误的示例：

```java
thFunction1 start
thFunction1 done
thFunction2 start
An interruption occurred in thFunction2
java.lang.InterruptedException: An interruption occurred in thFunction2
    at Main.thFunction2(Main.java:24)
    at Main.callThrowingFunctions(Main.java:58)
    at Main.main(Main.java:88)
ecFunction1 start
thFunction1 done
ecFunction2 start
thFunction2 done
```

请注意，`thFunction2`已经启动，但尚未完成。它抛出的异常包含有关`thFunction2`的信息。共享的`catch`块不必知道此异常来自何处；它只是捕获异常。这样，单个异常捕获块就能够处理多个函数调用。`thFunction2`抛出并被`catch`块捕获的异常对象能够传递有关问题的详细信息（例如堆栈跟踪）。这样，默认的预期执行路径保持干净，异常捕获块可以以细致的方式处理问题。

另一方面，看一下这个示例执行输出：

```java
thFunction1 start
thFunction1 done
thFunction2 start
thFunction2 done
ecFunction1 start
An I/O exception occurred in ecFunction1.
ecFunction2 start
ecFunction2 done
```

在`ecFunction1`中，发生了意外错误。这只是通过从该函数返回的错误代码值来表示的。请注意，此函数无法返回任何其他值；员工编号、某物是否活动等都是函数可能返回的一些示例。以这种方式从函数返回的错误代码禁止在返回值中传递此类信息。

此外，由于错误仅由一个数字表示，我们无法在错误处理代码中获得详细信息。我们还必须为每个函数调用编写错误处理代码，否则我们将无法区分错误位置。这会导致代码变得比应该更复杂和冗长。

进一步使用代码，多次运行它，并观察其行为。这应该让您更好地理解异常与错误代码，以及异常为什么更优越。

### 活动 36：处理数字用户输入中的错误

现在我们将在一个真实场景中使用异常处理。我们将创建一个控制台应用程序，在其中我们要求用户输入三个整数，将它们相加，并打印结果。如果用户没有输入非数字文本或分数，我们将要求用户提供一个整数。我们将为每个数字分别执行此操作——第三个数字的错误只需要我们重新输入第三个数字，我们的程序将很好地记住前两个数字。

以下步骤将帮助您完成此活动：

1.  从一个空的 Java 控制台项目开始。将以下代码放入其中，该代码从键盘读取输入，并在用户按下*Enter*键后将其打印出来。

1.  将此作为起点，并使用`Integer.parseInt()`函数将输入转换为数字。

1.  请注意，与我们之前的例子不同，IDE 没有警告我们可能出现异常。这是因为有两种类型的异常，我们将在接下来的主题中学习。现在，要知道`Integer.parseInt()`可能会引发`java.lang.NumberFormatException`。使用我们之前学到的知识，将这行代码放入一个期望`NumberFormatException`的`try/catch`块中。

1.  现在将其放入一个`while`循环中。只要用户没有输入有效的整数（整数），它就应该循环。一旦我们有这样的值，`while`循环就不应再循环。如果用户没有输入有效的整数，向用户打印出适当的消息。不要打印原始异常消息或堆栈跟踪。这样，我们坚持要求用户输入一个整数，并且不会放弃，直到我们得到一个整数。

1.  使用这种策略，输入三个整数并将它们相加。如果您没有为任何输入提供有效的整数，程序应该一遍又一遍地询问。将结果打印到控制台。

#### 注意

此活动的解决方案可在 365 页找到。

## 异常来源

当代码中出现异常情况时，问题源会抛出一个异常对象，然后被调用堆栈中的一个调用者捕获。异常对象是异常类的一个实例。有许多这样的类，代表各种类型的问题。在本主题中，我们将看看不同类型的异常，了解一些来自 Java 库的异常类，学习如何创建自己的异常，以及如何抛出它们。

在上一个主题中，我们首先使用了`IOException`。然后，在活动中，我们使用了`NumberFormatException`。这两个异常之间有所不同。IDE 会强制我们处理`IOException`，否则代码将无法编译。然而，它并不在乎我们是否捕获了`NumberFormatException`，它仍然会编译和运行我们的代码。区别在于类层次结构。虽然它们都是`Exception`类的后代，但`NumberFormatException`是`RuntimeException`的后代，是`Exception`的子类：

![图 9.1：RuntimeException 类的层次结构](img/C09581_09_011.jpg)

###### 图 9.1：RuntimeException 类的层次结构

上图显示了一个简单的类层次结构。任何是`Throwable`的后代类都可以作为异常抛出和捕获。然而，Java 对`Error`和`RuntimeException`类的后代类提供了特殊处理。我们将在接下来的部分中进一步探讨这些。

### 已检查异常

`Throwable`的任何后代，如果不是`Error`或`RuntimeException`的后代，都属于已检查异常的范畴。例如：`IOException`，我们在上一个主题中使用过的，就是一个已检查异常。IDE 强制我们要么捕获它，要么在我们的函数中指定抛出它。

要能够抛出已捕获的异常，您的函数必须指定它抛出异常。

### 抛出已检查异常

创建一个新项目，并粘贴以下代码：

```java
import java.io.IOException;
public class Main {
   private static void myFunction() {
       throw new IOException("hello");
   }
   public static void main(String[] args) {
       myFunction();
   }
}
```

在这里，我们创建了一个函数，并希望它抛出`IOException`。然而，我们的 IDE 不会让我们这样做，因为这是一个已检查的异常。以下是它的类型层次结构：

![图 9.2：IOException 类的层次结构](img/C09581_09_021.jpg)

###### 图 9.2：IOException 类的层次结构

由于`IOException`是`Exception`的后代，它是一个已检查的异常，每个抛出已检查异常的函数都必须指定它。将光标移动到错误行，按下*Alt* + *Enter*，然后选择“**将异常添加到方法签名**”。代码将如下所示：

```java
import java.io.IOException;
public class Main {
   private static void myFunction() throws IOException {
       throw new IOException("hello");
   }
   public static void main(String[] args) {
       myFunction();
   }
}
```

请注意，我们的代码仍然存在问题。我们将在下一个练习中继续处理它。

已检查异常的另一个要求是，如果你调用指定了已检查异常的方法，你必须要么捕获异常，要么指定你也抛出该异常。这也被称为“捕获或指定规则”。

### 练习 39：使用 catch 或指定

让我们来看看抛出已检查异常和调用抛出它们的方法。你应该已经打开了这个项目：

1.  如果你的 IDE 中没有前面的示例，请创建一个项目并添加以下代码：

```java
import java.io.IOException;
public class Main {
   private static void myFunction() throws IOException {
       throw new IOException("hello");
   }
   public static void main(String[] args) {
       myFunction();
   }
}
```

请注意，带有`myFunction()`的那一行被标记为红色下划线，因为这一行调用了一个已检查的异常，而我们没有对潜在的异常做任何处理。我们需要指定我们也抛出它，或者我们需要捕获和处理它。IntelliJ 可以帮助我们做这两件事中的任何一件。将光标移动到`myFunction1()`行上，然后按*Alt* + *Enter*。

1.  选择**将异常添加到方法签名**，以成功指定我们抛出异常。这是它生成的代码：

```java
import java.io.IOException;
public class Main {
   private static void myFunction() throws IOException {
       throw new IOException("hello");
   }
   public static void main(String[] args) throws IOException {
       myFunction();
   }
}
```

正如你所看到的，这个编译和运行都很顺利。现在撤销（*Ctrl* + *Z*）然后再次按*Alt*+ *Enter*来获取选项。

1.  或者，如果我们选择**用 try/catch 包围**，我们将成功捕获异常。这是它生成的代码：

```java
import java.io.IOException;
public class Main {
   private static void myFunction() throws IOException {
       throw new IOException("hello");
   }
   public static void main(String[] args) {
       try {
           myFunction();
       } catch (IOException e) {
           e.printStackTrace();
       }
   }
}
```

虽然这个编译和运行，但记住，简单地打印有关它的信息并不是处理异常的最佳方式。

在这些练习中，我们看到了如何抛出已检查的异常以及如何调用抛出它们的方法。

### 未检查异常

回顾异常类层次结构的顶部：

![图 9.3：RuntimeException 类的层次结构](img/C09581_09_03.jpg)

###### 图 9.3：RuntimeException 类的层次结构

在这里，`RuntimeException`的后代被称为运行时异常。`Error`的后代被称为错误。这两者都被称为未检查异常。它们不需要被指定，如果被指定了，也不需要被捕获。

未检查异常代表可能发生的事情，与已检查异常相比更加意外。假设你有选择确保它们不会被抛出；因此，它们不必被期望。但是，如果你怀疑它们可能被抛出，你应该尽力处理它们。

NumberFormatException 的层次结构如下：

![图 9.4：NormalFormatException 类的层次结构](img/C09581_09_04.jpg)

###### 图 9.4：NormalFormatException 类的层次结构

由于它是`RuntimeException`的后代，因此它是运行时异常，因此是未检查异常。

### 练习 40：使用抛出未检查异常的方法

在这个练习中，我们将编写一些会抛出运行时异常的代码：

1.  在 IntelliJ 中创建一个项目，并粘贴以下代码：

```java
public class Main {
public static void main(String[] args) {
int i = Integer.parseInt("this is not a number");
}
}
```

请注意，这段代码试图将一个字符串解析为整数，但显然该字符串不包含整数。因此，将抛出`NumberFormatException`。但是，由于这是一个未检查的异常，我们不必捕获或指定它。当我们运行代码时，就会看到这种情况：

```java
Exception in thread "main" java.lang.NumberFormatException: For input string: "this is not a number"
    at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
    at java.lang.Integer.parseInt(Integer.java:580)
    at java.lang.Integer.parseInt(Integer.java:615)
    at Main.main(Main.java:6)
```

1.  由于我们没有捕获它，`NumberFormatException`从`main`函数中抛出并使应用程序崩溃。相反，我们可以捕获它并打印关于它的消息，如下所示：

```java
public class Main {
public static void main(String[] args) {
try {
int i = Integer.parseInt("this is not a number");
} catch (NumberFormatException e) {
System.out.println("Sorry, the string does not contain an integer.");
}
}
}
```

现在，当我们运行代码时，我们会得到一个输出，显示我们意识到了这种情况：

```java
Sorry, the string does not contain an integer.
```

尽管捕获未检查异常是可选的，但你应该确保捕获它们，以便创建完整的代码。

对于错误来说，情况几乎是一样的，它们是`Error`类的后代。在接下来的部分中，我们将讨论运行时异常和错误之间的语义差异。

### 异常类层次结构

任何可以作为异常抛出的对象都是从`Error`或`RuntimeException`派生的类的实例，被视为未经检查的异常，而从`Throwable`派生的任何其他类都是经过检查的异常。因此，您使用哪个异常类决定了异常处理的机制（经过检查与未经检查）。

除了异常处理的机制之外，异常类的选择还携带语义信息。例如：如果库方法遇到一个应该在硬盘上的文件丢失的情况，它会抛出一个`FileNotFoundException`的实例。如果一个字符串中应该包含一个数值，但出现了问题，您给出该字符串的方法会抛出一个`NumberFormatException`。Java 类库包含了许多适合大多数意外情况的异常类。以下是此层次结构中的类的一个子集：

![图 9.5：层次结构中的类子集](img/C09581_09_05.jpg)

###### 图 9.5：层次结构中的类子集

阅读此列表，您会注意到各种场合有很多异常类型。

### 浏览异常层次结构

在 IntelliJ 中，打开任何 Java 项目或创建一个新项目。在您的代码中的任何地方，创建一个`Throwable`引用变量如下：

```java
Throwable t;
```

现在将光标移动到`Throwable`上，然后按`Ctrl` + `H`。层次结构窗口应该打开，并将`Throwable`类放在焦点位置。它应该看起来像这样：

![图 9.6：Throwable 类的层次结构](img/C09581_09_06.jpg)

###### 图 9.6：Throwable 类的层次结构

现在展开`Error`和`Exception`，并浏览类列表。这些是各种可抛出的类，定义在您的代码可以访问的各种库中。正如您所看到的，有相当广泛的异常列表可供选择。在每个异常类旁边，用括号括起来的是它所属的包。作为一个经验法则，如果您要自己抛出异常，应该尽量使用您也在使用的库中的异常。例如：仅仅为了使用其中定义的`ParseException`而导入`com.sun.jmx.snmp.IPAcl`是不好的做法。

现在您对 Java 类库中存在的异常类有了更好的了解，以及您选择的异常类对代码用户传达的信息。

### 抛出异常和自定义异常

作为程序员，您将编写您或其他人将调用的方法。不可避免地，在您的代码中会出现不希望的情况。在这些情况下，您应该抛出适当异常类的实例。

要抛出异常，首先需要创建一个是`Throwable`祖先类的实例。然后，填充该实例并使用`throw`关键字将其抛出。然后，可抛出实例将沿着调用堆栈向上移动并弹出条目，直到遇到一个带有匹配此`Throwable`类型或其子类的 catch 语句的`try`/`catch`块。可抛出实例将作为捕获的异常给该 catch 块，并从那里继续执行。

### 练习 41：抛出异常

在这个练习中，我们将使用现有的异常类来抛出异常：

1.  创建一个新的 Java 项目，并添加以下代码，其中有一个函数期望一个包含单个数字的长度为一的字符串并打印它。如果字符串为空，它将抛出一个`IllegalArgumentException`。如果字符串包含除了单个数字以外的任何内容，它将抛出一个`NumberFormatException`。由于这些是未经检查的异常，我们不必指定它们：

```java
public class Main {
public static void useDigitString(String digitString) {
if (digitString.isEmpty()) {
throw new IllegalArgumentException("An empty string was given instead of a digit");
}
if (digitString.length() > 1) {
throw new NumberFormatException("Please supply a string with a single digit");
}
}
}
```

1.  现在我们将调用此函数并处理它抛出的异常。我们故意调用另一个调用此函数的函数，并在两个不同的地方有 catch 块，以演示异常传播。完整的代码如下所示：

```java
public class Main {
   public static void useDigitString(String digitString) {
       if (digitString.isEmpty()) {
           throw new IllegalArgumentException("An empty string was given instead of a digit");
       }
       if (digitString.length() > 1) {
           throw new NumberFormatException("Please supply a string with a single digit");
       }
       System.out.println(digitString);
   }
   private static void runDigits() {
       try {
           useDigitString("1");
           useDigitString("23");
           useDigitString("4");
       } catch (NumberFormatException e) {
           System.out.println("A number format problem occurred: " + e.getMessage());
       }
       try {
           useDigitString("5");
           useDigitString("");
           useDigitString("7");
       } catch (NumberFormatException e) {
           System.out.println("A number format problem occured: " + e.getMessage());
       }
   }
```

1.  添加`main()`方法如下：

```java
   public static void main(String[] args) {
       try {
           runDigits();
       } catch (IllegalArgumentException e) {
           System.out.println("An illegal argument was provided: " + e.getMessage());
       }
   }
}
```

注意，从`main`中调用`runDigits`，然后调用`useDigitString`。主函数捕获`IllegalArgumentException`，`runDigits`捕获`NumberFormatException`。尽管我们在`useDigitString`中抛出了所有异常，但它们被不同的地方捕获。

### 练习 42：创建自定义异常类

在以前的练习中，我们为我们的异常使用了现有的异常类。`NumberFormatException`听起来合适，但`IllegalArgumentException`有点奇怪。而且，它们都是未经检查的异常；也许我们想要有检查的异常。因此，现有的异常类不适合我们的需求。在这种情况下，我们可以创建自己的异常类。让我们继续沿着上一个练习的路线：

1.  假设我们对`NumberFormatException`感到满意，但我们想要一个是检查的`EmptyInputException`。我们可以扩展`Exception`来实现这一点：

```java
class EmptyInputException extends Exception {
}
```

1.  如果我们有额外的信息要放入此异常中，我们可以为此添加字段和构造函数。但是，在我们的情况下，我们只想表明输入为空；对于调用者来说，不需要其他信息。现在让我们修复我们的代码，使我们的函数抛出`EmptyInputException`而不是`IllegalArgumentException`：

```java
class EmptyInputException extends Exception {
}
public class Main {
   public static void useDigitString(String digitString) throws EmptyInputException {
       if (digitString.isEmpty()) {
           throw new EmptyInputException();
       }
       if (digitString.length() > 1) {
           throw new NumberFormatException("Please supply a string with a single digit");
       }
       System.out.println(digitString);
   }
   private static void runDigits() throws EmptyInputException {
       try {
           useDigitString("1");
           useDigitString("23");
           useDigitString("4");
       } catch (NumberFormatException e) {
           System.out.println("A number format problem occured: " + e.getMessage());
       }
       try {
           useDigitString("5");
           useDigitString("");
           useDigitString("7");
       } catch (NumberFormatException e) {
           System.out.println("A number format problem occured: " + e.getMessage());
       }
   }
```

1.  按照以下方式添加`main()`方法：

```java
   public static void main(String[] args) {
       try {
           runDigits();
       } catch (EmptyInputException e) {
           System.out.println("An empty string was provided");
       }
   }
}
```

请注意，这使我们的代码变得简单得多——我们甚至不必写消息，因为异常的名称清楚地传达了问题。以下是输出：

```java
1
A number format problem occured: Please supply a string with a single digit
5
An empty string was provided
```

现在您知道如何抛出异常并创建自己的异常类（如果现有的异常类不够用）。

### 活动 37：在 Java 中编写自定义异常。

我们将为过山车乘坐的入场系统编写一个程序。对于每位游客，我们将从键盘获取他们的姓名和年龄。然后，我们将打印出游客的姓名以及他们正在乘坐过山车。

由于过山车只适合成年人，我们将拒绝年龄小于 15 岁的游客。我们将使用自定义异常`TooYoungException`来处理拒绝。此异常对象将包含游客的姓名和年龄。当我们捕获异常时，我们将打印一个适当的消息，解释为什么他们被拒绝。

我们将继续接受游客，直到姓名为空为止。

要实现这一点，请执行以下步骤：

1.  创建一个新类，并输入`RollerCoasterWithAge`作为类名。

1.  还要创建一个异常类`TooYoungException`。

1.  导入`java.util.Scanner`包。

1.  在`main()`中，创建一个无限循环。

1.  获取用户的姓名。如果是空字符串，则跳出循环。

1.  获取用户的年龄。如果低于 15 岁，则抛出一个`TooYoungException`，包含姓名和年龄。

1.  将姓名打印为"John 正在乘坐过山车"。

1.  捕获异常并为其打印适当的消息。

1.  运行主程序。

输出应类似于以下内容：

```java
Enter name of visitor: John
Enter John's age: 20
John is riding the roller coaster.
Enter name of visitor: Jack
Enter Jack's age: 13
Jack is 13 years old, which is too young to ride.
Enter name of visitor: 
```

#### 注意

此活动的解决方案可在第 366 页找到。

## 异常机制

在以前的主题中，我们抛出并捕获了异常，并对异常的工作原理有了一定的了解。现在让我们重新访问机制，以确保我们做对了一切。

### `try/catch`的工作原理

`try`/`catch`语句有两个块：`try`块和`catch`块，如下所示：

```java
try {
   // the try block
} catch (Exception e) {
   // the catch block, can be multiple 
}
```

`try`块是您的主要执行路径代码所在的地方。您可以在这里乐观地编写程序。如果`try`块中的任何一行发生异常，执行将在该行停止并跳转到`catch`块：

```java
try {
   // line1, fine
   // line2, fine
   // line3, EXCEPTION!
   // line4, skipped
   // line5, skipped
} catch (Exception e) {
   // comes here after line3
}
```

`catch`块捕获可分配给其包含的异常引用（在本例中为`Exception e`）的可抛出对象。因此，如果在此处有一个在异常层次结构中较高的异常类（如`Exception`），它将捕获所有异常。这不会捕获错误，这通常是您想要的。

如果您想更具体地捕获异常的类型，可以提供一个在层次结构中较低的异常类。

### 练习 43：异常未被捕获，因为它不能分配给 catch 块中的参数

1.  创建一个新项目并添加以下代码：

```java
public class Main {
   public static void main(String[] args) {
       try {
           for (int i = 0; i < 5; i++) {
               System.out.println("line " + i);
               if (i == 3) throw new Exception("EXCEPTION!");
           }
       } catch (InstantiationException e) {
           System.out.println("Caught an InstantiationException");
       }
   }
}
```

请注意，这段代码甚至无法编译。代码抛出异常，但 catch 子句期望一个`InstantiationException`，它是`Exception`的一个后代，不能分配给异常实例。因此，异常既不被捕获，也不被抛出。

1.  指定一个异常，以便代码可以编译如下：

```java
public class Main {
   public static void main(String[] args) throws Exception {
       try {
           for (int i = 0; i < 5; i++) {
               System.out.println("line " + i);
               if (i == 3) throw new Exception("EXCEPTION!");
           }
       } catch (InstantiationException e) {
           System.out.println("Caught an InstantiationException");
       }
   }
}
```

当我们运行代码时，我们发现我们无法捕获我们抛出的异常：

```java
line 0
line 1
line 2
line 3
Exception in thread "main" java.lang.Exception: EXCEPTION!
    at Main.main(Main.java:8)
```

有时，您捕获特定异常的一种类型，但您的代码也可能抛出其他类型的异常。在这种情况下，您可以提供多个 catch 块。被捕获的异常类型可以在类层次结构的不同位置。被抛出的异常可以分配给其参数的第一个 catch 块被执行。因此，如果两个异常类具有祖先关系，那么后代的 catch 子句必须在祖先的 catch 子句之前；否则，祖先也会捕获后代的异常。

### 练习 44：多个 catch 块及其顺序

在这个练习中，我们将看一下程序中的多个 catch 块及其执行顺序。让我们继续上一个练习：

1.  返回代码的初始形式：

```java
public class Main {
   public static void main(String[] args) {
       try {
           for (int i = 0; i < 5; i++) {
               System.out.println("line " + i);
               if (i == 3) throw new Exception("EXCEPTION!");
           }
       } catch (InstantiationException e) {
           System.out.println("Caught an InstantiationException");
       }
   }
}
```

1.  当我们在`Exception`上按*Alt* + *Enter*添加一个 catch 子句时，它会在现有的 catch 子句之后添加，这是正确的：

```java
public class Main {
   public static void main(String[] args) {
       try {
           for (int i = 0; i < 5; i++) {
               System.out.println("line " + i);
               if (i == 3) throw new Exception("EXCEPTION!");
           }
       } catch (InstantiationException e) {
           System.out.println("Caught an InstantiationException");
       } catch (Exception e) {
           e.printStackTrace();
       }
   }
}
```

1.  如果抛出的异常是`InstantiationException`，它将被第一个 catch 捕获。否则，如果是其他任何异常，它将被第二个 catch 捕获。让我们尝试重新排列 catch 块：

```java
public class Main {
   public static void main(String[] args) {
       try {
           for (int i = 0; i < 5; i++) {
               System.out.println("line " + i);
               if (i == 3) throw new Exception("EXCEPTION!");
           }
       } catch (Exception e) {
           e.printStackTrace();
       } catch (InstantiationException e) {
           System.out.println("Caught an InstantiationException");
       }
   }
}
```

现在我们的代码甚至无法编译，因为`InstantiationException`的实例可以分配给`Exception e`，并且它们将被第一个 catch 块捕获。第二个块永远不会被调用。IDE 很聪明地为我们解决了这个问题。

异常的另一个属性是它们沿着调用堆栈传播。每个被调用的函数本质上都会将执行返回给它的调用者，直到其中一个能够捕获异常。

### 练习 45：异常传播

在这个练习中，我们将通过一个例子来看一下多个函数相互调用的情况：

1.  我们从最深的方法中抛出异常，这个异常被调用堆栈中更高的一个方法捕获：

```java
public class Main {
   private static void method3() throws Exception {
       System.out.println("Begin method 3");
       try {
           for (int i = 0; i < 5; i++) {
               System.out.println("line " + i);
               if (i == 3) throw new Exception("EXCEPTION!");
           }
       } catch (InstantiationException e) {
           System.out.println("Caught an InstantiationException");
       }
       System.out.println("End method 3");
   }
   private static void method2() throws Exception {
       System.out.println("Begin method 2");
       method3();
       System.out.println("End method 2");
   }
   private static void method1() {
       System.out.println("Begin method 1");
       try {
           method2();
       } catch (Exception e) {
           System.out.println("method1 caught an Exception!: " + e.getMessage());
           System.out.println("Also, below is the stack trace:");
           e.printStackTrace();
       }
       System.out.println("End method 1");
   }
```

1.  添加`main()`方法如下：

```java
   public static void main(String[] args) {
       System.out.println("Begin main");
       method1();
       System.out.println("End main");
   }
}
```

当我们运行代码时，我们得到以下输出：

```java
Begin main
Begin method 1
Begin method 2
Begin method 3
line 0
line 1
line 2
line 3
method1 caught an Exception!: EXCEPTION!
Also, below is the stack trace:
java.lang.Exception: EXCEPTION!
    at Main.method3(Main.java:8)
    at Main.method2(Main.java:18)
    at Main.method1(Main.java:25)
    at Main.main(Main.java:36)
End method 1
End main
```

注意，方法 2 和方法 3 没有运行到完成，而方法 1 和`main`运行到完成。方法 2 抛出异常；方法 3 没有捕获它，而是让它传播上去。最后，方法 1 捕获它。方法 2 和方法 3 突然返回到调用堆栈中更高的方法。由于方法 1 和 main 不让异常传播上去，它们能够运行到完成。

catch 块的另一个特性是我们应该谈论的。假设我们想要捕获两个特定的异常，但不捕获其他异常，但我们将在它们的 catch 块中做完全相同的事情。在这种情况下，我们可以使用管道字符组合这些异常的 catch 块。这个特性是在 Java 7 中引入的，在 Java 6 及以下版本中不起作用。

### 一个块中的多个异常类型

我们已经在一段代码中处理了单一类型的异常。现在我们将看一下一段代码中的多个异常类型。

考虑以下代码：

```java
import java.io.IOException;
public class Main {
public static void method1() throws IOException {
System.out.println(4/0);
}
public static void main(String[] args) {
try {
System.out.println("line 1");
method1();
System.out.println("line 2");
} catch (IOException|ArithmeticException e) {
System.out.println("An IOException or a ArithmeticException was thrown. Details below.");
e.printStackTrace();
}
}
}
```

在这里，我们有一个 catch 块，可以使用多个异常类型的 catch 块捕获`IOException`或`ArithmeticException`。当我们运行代码时，我们看到我们引起的`ArithmeticException`被成功捕获：

```java
line 1
An IOException or a ArithmeticException was thrown. Details below.
java.lang.ArithmeticException: / by zero
    at Main.method1(Main.java:6)
    at Main.main(Main.java:12)
```

如果异常是`IOException`，它将以相同的方式被捕获。

现在你更了解`try`/`catch`块的机制、异常传播、多个 catch 块和块中的多个异常。

### 活动 38：处理块中的多个异常

记住我们之前为过山车乘坐的入场系统编写了一个程序吗？这一次，我们还将考虑访客的身高。对于每位访客，我们将从键盘获取他们的姓名、年龄和身高。然后，我们将打印出访客的姓名和他们正在乘坐过山车。

由于过山车只适合特定身高的成年人，我们将拒绝 15 岁以下或低于 130 厘米的访客。我们将使用自定义异常`TooYoungException`和`TooShortException`来处理拒绝。这些异常对象将包含人的姓名和相关属性（年龄或身高）。当我们捕获异常时，我们将打印一个适当的消息，解释为什么他们被拒绝。

我们将继续接受访客，直到姓名为空为止。

为了实现这一点，执行以下步骤：

1.  创建一个新类，并输入`RollerCoasterWithAgeAndHeight`作为类名。

1.  还要创建两个异常类，`TooYoungException`和`TooShortException`。

1.  导入`java.util.Scanner`包。

1.  在`main()`中，创建一个无限循环。

1.  获取用户的姓名。如果是空字符串，跳出循环。

1.  获取用户的年龄。如果低于 15，抛出一个带有这个名字和年龄的`TooYoungException`。

1.  获取用户的身高。如果低于 130，抛出一个带有这个名字和年龄的`TooShortException`。

1.  将姓名打印为"John 正在乘坐过山车"。

1.  分别捕获两种类型的异常。为每种情况打印适当的消息。

1.  运行主程序。

输出应该类似于以下内容：

```java
Enter name of visitor: John
Enter John's age: 20
Enter John's height: 180
John is riding the roller coaster.
Enter name of visitor: Jack
Enter Jack's age: 13
Jack is 13 years old, which is too young to ride.
Enter name of visitor: Jill
Enter Jill's age: 16
Enter Jill's height: 120
Jill is 120 cm tall, which is too short to ride.
Enter name of visitor: 
```

#### 注意

这个活动的解决方案可以在第 368 页找到。

### 在 catch 块中我们应该做什么？

当你捕获异常时，你应该对它做些什么。理想情况下，你可以找到一种从错误中恢复并恢复执行的策略。然而，有时你无法做到这一点，可能会选择在你的函数中指定让这个异常使用 throws 语句传播。我们在上一个主题中看到了这些。

然而，在某些情况下，你可能有能力向你的调用者添加更多信息到异常中。例如：假设你调用一个方法来解析用户的年龄，它抛出了一个`NumberFormatException`。如果你简单地让它传播给你的调用者，你的调用者将不知道这与用户的年龄有关。也许在将异常传播给你的调用者之前，添加这些信息会有益处。你可以通过捕获异常，将其包装在另一个异常中作为原因，并将该异常抛出给你的调用者来实现这一点。这也被称为“链接异常”。

### 练习 46：链接异常

在这个练习中，我们将看一下链接异常的工作原理：

1.  创建一个新项目并添加这段代码：

```java
public class Main {
public static int parseUsersAge(String ageString) {
return Integer.parseInt(ageString);
}
public static void readUserInfo()  {
int age = parseUsersAge("fifty five");
}
public static void main(String[] args) {
readUserInfo();
}
}
```

请注意，尝试将"fifty five"解析为整数将导致`NumberFormatException`。我们没有捕获它，而是让它传播。以下是我们得到的输出结果：

```java
Exception in thread "main" java.lang.NumberFormatException: For input string: "fifty five"
    at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
    at java.lang.Integer.parseInt(Integer.java:580)
    at java.lang.Integer.parseInt(Integer.java:615)
    at Main.parseUsersAge(Main.java:4)
    at Main.readUserInfo(Main.java:8)
    at Main.main(Main.java:12)
```

请注意，异常的输出没有任何迹象表明这个问题与用户的年龄有关。

1.  捕获异常并链接它以添加关于年龄的信息：

```java
public class Main {
public static int parseUsersAge(String ageString) {
return Integer.parseInt(ageString);
}
public static void readUserInfo() throws Exception {
try {
int age = parseUsersAge("fifty five");
} catch (NumberFormatException e) {
throw new Exception("Problem while parsing user's age", e);
}
}
```

1.  按照以下步骤添加`main()`方法：

```java
public static void main(String[] args) throws Exception {
readUserInfo();
}
}
```

在这种情况下，这是我们得到的输出：

```java
Exception in thread "main" java.lang.Exception: Problem while parsing user's age
    at Main.readUserInfo(Main.java:11)
    at Main.main(Main.java:16)
Caused by: java.lang.NumberFormatException: For input string: "fifty five"
    at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
    at java.lang.Integer.parseInt(Integer.java:580)
    at java.lang.Integer.parseInt(Integer.java:615)
    at Main.parseUsersAge(Main.java:4)
    at Main.readUserInfo(Main.java:9)
    ... 1 more
```

请注意，这包含有关年龄的信息。这是一个异常，它有另一个异常作为原因。如果你愿意，你可以使用`e.getCause()`方法获取它，并相应地采取行动。当简单记录时，它按顺序打印异常详细信息。

### 最后的块及其机制

`try`/`catch`块在捕获异常时非常有用。但是，在这里有一个常见的情况，它可能有一些缺点。在我们的代码中，我们想获取一些资源。我们负责在完成后释放资源。但是，一个天真的实现可能会导致在发生异常时文件被保持打开状态。

### 练习 47：由于异常而保持文件打开

在这个练习中，我们将处理`finally`块：

1.  假设我们将读取文件的第一行并将其打印出来。我们可以将其编码如下：

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
public class Main {
private static void useTheFile(String s) {
System.out.println(s);
throw new RuntimeException("oops");
}
```

1.  添加`main（）`方法如下：

```java
public static void main(String[] args) throws Exception {
try {
BufferedReader br = new BufferedReader(new FileReader("input.txt"));
System.out.println("opened the file");
useTheFile(br.readLine());
br.close();
System.out.println("closed the file");
} catch (Exception e) {
System.out.println("caught an exception while reading the file");
}
}
}
```

请注意，`useTheFile`函数在我们关闭文件之前引发了异常。当我们运行它时，我们会得到这个结果：

```java
opened the file
line 1 from the file
caught an exception while reading the file
```

请注意，我们没有看到“关闭文件”输出，因为执行永远无法通过`useTheFile（）`调用。捕获异常后，即使我们无法访问`BufferedReader`引用，操作系统仍然持有文件资源。我们刚刚泄漏了一个资源。如果我们在循环中多次执行此操作，我们的应用程序可能会崩溃。

1.  您可以尝试设计各种解决此资源泄漏问题的解决方案。例如：您可以复制文件关闭代码并将其粘贴到 catch 块中。现在您在`try`块和`catch`块中都有它。如果有多个`catch`块，所有这些都应该如下所示：

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
public class Main {
private static void useTheFile(String s) {
System.out.println(s);
throw new RuntimeException("oops");
}
public static void main(String[] args) throws Exception {
BufferedReader br = null;
try {
br = new BufferedReader(new FileReader("input.txt"));
System.out.println("opened the file");
useTheFile(br.readLine());
br.close();
System.out.println("closed the file");
} catch (IOException e) {
System.out.println("caught an I/O exception while reading the file");
br.close();
System.out.println("closed the file");
} catch (Exception e) {
System.out.println("caught an exception while reading the file");
br.close();
System.out.println("closed the file");
}
}
}
```

1.  前面的代码是正确的，但它存在代码重复，这使得难以维护。相反，您可能认为可以在一个地方的`catch`块之后关闭文件：

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
public class Main {
private static void useTheFile(String s) {
System.out.println(s);
throw new RuntimeException("oops");
}
public static void main(String[] args) throws Exception {
BufferedReader br = null;
try {
br = new BufferedReader(new FileReader("input.txt"));
System.out.println("opened the file");
useTheFile(br.readLine());
} catch (IOException e) {
System.out.println("caught an I/O exception while reading the file");
throw new Exception("something is wrong with I/O", e);
} catch (Exception e) {
System.out.println("caught an exception while reading the file");
}
br.close();
System.out.println("closed the file");
}
}
```

虽然这几乎是正确的，但它缺少一个可能性。请注意，我们现在在第一个`catch`块中抛出异常。这将绕过 catch 块后面的代码，文件仍将保持打开状态。

1.  因此，我们需要确保无论发生什么，文件关闭代码都将运行。`try`/`catch`/`finally`块是这个问题的解决方案。它就像`try`/`catch`块，有一个额外的 finally 块，在我们完成块后执行，无论发生什么。以下是带有`finally`块的解决方案：

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
public class Main {
private static void useTheFile(String s) {
System.out.println(s);
throw new RuntimeException("oops");
}
public static void main(String[] args) throws Exception {
BufferedReader br = null;
try {
br = new BufferedReader(new FileReader("input.txt"));
System.out.println("opened the file");
useTheFile(br.readLine());
} catch (IOException e) {
System.out.println("caught an I/O exception while reading the file");
throw new Exception("something is wrong with I/O", e);
} catch (Exception e) {
System.out.println("caught an exception while reading the file");
} finally {
br.close();
System.out.println("closed the file");
}
}
}
```

这个新版本关闭文件，无论是否引发异常，或者在最初捕获异常后引发另一个异常。在每种情况下，finally 块中的文件关闭代码都会被执行，并且文件资源会被操作系统适当释放。

这段代码还有一个问题。问题是，在`BufferedReader`构造函数中打开文件时可能会引发异常，`br`变量可能仍然为空。然后，当我们尝试关闭文件时，我们将取消引用一个空变量，这将创建一个异常。

1.  为了避免这个问题，我们需要忽略`br`如果它是空的。以下是完整的代码：

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
public class Main {
   private static void useTheFile(String s) {
       System.out.println(s);
       throw new RuntimeException("oops");
   }
   public static void main(String[] args) throws Exception {
       BufferedReader br = null;
       try {
           br = new BufferedReader(new FileReader("input.txt"));
           System.out.println("opened the file");
           useTheFile(br.readLine());
       } catch (IOException e) {
           System.out.println("caught an I/O exception while reading the file");
           throw new Exception("something is wrong with I/O", e);
       } catch (Exception e) {
           System.out.println("caught an exception while reading the file");
       } finally {
           if (br != null) {
               br.close();
               System.out.println("closed the file");
           }
       }
   }
}
```

### 活动 39：使用多个自定义异常处理

请记住，我们为过山车乘坐的入场系统编写了一个程序，该程序验证了访问者的年龄和身高。这一次，我们将假设我们必须在过山车区域之外护送每个申请人，无论他们是否乘坐过山车。

我们将逐个接纳访客。对于每个访客，我们将从键盘获取他们的姓名，年龄和身高。然后，我们将打印出访客的姓名以及他们正在乘坐过山车。

由于过山车只适合特定身高的成年人，我们将拒绝年龄小于 15 岁或身高低于 130 厘米的访客。我们将使用自定义异常`TooYoungException`和`TooShortException`来处理拒绝。这些异常对象将包含人的姓名和相关属性（年龄或身高）。当我们捕获异常时，我们将打印出一个适当的消息，解释为什么他们被拒绝。

一旦我们完成了与游客的互动，无论他们是否乘坐过山车，我们都会打印出我们正在护送游客离开过山车区域。

我们将继续接受游客，直到姓名为空。

为了实现这一点，执行以下步骤：

1.  创建一个新的类，并输入`RollerCoasterWithEscorting`作为类名。

1.  还要创建两个异常类，`TooYoungException`和`TooShortException`。

1.  导入`java.util.Scanner`包。

1.  在`main()`中，创建一个无限循环。

1.  获取用户的姓名。如果是空字符串，跳出循环。

1.  获取用户的年龄。如果低于 15，抛出一个名为`TooYoungException`的异常。

1.  获取用户的身高。如果低于 130，抛出一个名为`TooShortException`的异常。

1.  将姓名打印为"约翰正在乘坐过山车"。

1.  分别捕获两种类型的异常。为每个打印适当的消息。

1.  打印出你正在护送用户离开场地。您必须小心姓名变量的范围。

1.  运行主程序。

输出应该类似于以下内容：

```java
Enter name of visitor: John
Enter John's age: 20
Enter John's height: 180
John is riding the roller coaster.
Escorting John outside the premises. 
Enter name of visitor: Jack
Enter Jack's age: 13
Jack is 13 years old, which is too young to ride.
Escorting Jack outside the premises. 
Enter name of visitor: Jill
Enter Jill's age: 16
Enter Jill's height: 120
Jill is 120 cm tall, which is too short to ride.
Escorting Jill outside the premises. 
Enter name of visitor: 
```

#### 注意

这个活动的解决方案可以在第 370 页找到。

### 带资源的 try 块

`try`/`catch`/`finally`块是处理已分配资源的一种很好的方式。然而，您可能会同意，它感觉有点像样板文件。在 finally 块中分配资源并释放它们是一种非常常见的模式。Java 7 引入了一个新的块，简化了这种常见模式——`try with resource`块。在这个新的块中，我们将资源分配放在 try 块后面的括号中，然后忘记它们。系统将自动调用它们的`.close()`方法：

```java
try(Resource r1 = Resource(); OtherResource r2 = OtherResource()) {
    r1.useResource();
    r2.useOtherResource();
} // don't worry about closing the resources
```

为了使这个工作，所有这些资源都必须实现`AutoCloseable`接口。

### 练习 48：带资源的 try 块

在这个练习中，我们将看一下带资源的 try 块：

1.  按照以下方式导入所需的类：

```java
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
```

1.  创建一个`Main`类，其中包含`useTheFile()`方法，该方法接受一个字符串参数，如下所示：

```java
public class Main {
   private static void useTheFile(String s) {
       System.out.println(s);
       throw new RuntimeException("oops");
   }
```

1.  将我们之前的例子转换为使用带资源的 try 块，如下所示：

```java
public static void main(String[] args) throws Exception {
       try (BufferedReader br = new BufferedReader(new FileReader("input.txt"))) {
           System.out.println("opened the file, which will be closed automatically");
           useTheFile(br.readLine());
       } catch (IOException e) {
           System.out.println("caught an I/O exception while reading the file");
           throw new Exception("something is wrong with I/O", e);
       } catch (Exception e) {
           System.out.println("caught an exception while reading the file");
       }
   }
}
```

## 最佳实践

虽然学习异常处理及其语句、机制和类是使用它所必需的，但对于大多数程序员来说，这可能还不够。通常，这套理论信息需要各种情况的实际经验，以更好地了解异常。在这方面，关于异常的实际使用的一些经验法则值得一提：

+   除非您真正处理了异常，否则不要压制异常。

+   通知用户并让他们承担责任，除非您可以悄悄地解决问题。

+   注意调用者的行为，不要泄漏异常，除非它是预期的。

+   尽可能包装和链接更具体的异常。

### 压制异常

在您的函数中，当您捕获异常并不抛出任何东西时，您正在表明您已经处理了异常情况，并且您已经修复了这种情况，使得好像这种异常情况从未发生过一样。如果您不能做出这样的声明，那么您就不应该压制那个异常。

### 练习 49：压制异常

例如：假设我们有一个字符串列表，我们期望其中包含整数数字：

1.  我们将解析它们并将它们添加到相应的整数列表中：

```java
import java.util.ArrayList;
import java.util.List;
public class Main {
   private static List<Integer> parseIntegers(List<String> inputList) {
       List<Integer> integers = new ArrayList<>();
       for(String s: inputList) {
           integers.add(Integer.parseInt(s));
       }
       return integers;
   }
```

1.  添加一个如下所示的`main()`方法：

```java
   public static void main(String[] args) {
       List<String> inputList = new ArrayList<>();
       inputList.add("1");
       inputList.add("two");
       inputList.add("3");

       List<Integer> outputList = parseIntegers(inputList);

       int sum = 0;
       for(Integer i: outputList) {
           sum += i;
       }
       System.out.println("Sum is " + sum);
   }
}
```

当我们运行这个时，我们得到这个输出：

```java
Exception in thread "main" java.lang.NumberFormatException: For input string: "two"
    at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
    at java.lang.Integer.parseInt(Integer.java:580)
    at java.lang.Integer.parseInt(Integer.java:615)
    at Main.parseIntegers(Main.java:9)
    at Main.main(Main.java:20)
```

1.  我们应该对此做些什么；至少，我们不应该让我们的代码崩溃。正确的行动是什么？我们应该在`parseIntegers`函数内捕获错误，还是应该在主函数中捕获错误？让我们在`parseIntegers`中捕获它，看看会发生什么：

```java
import java.util.ArrayList;
import java.util.List;
public class Main {
   private static List<Integer> parseIntegers(List<String> inputList) {
       List<Integer> integers = new ArrayList<>();
       for(String s: inputList) {
           try {
               integers.add(Integer.parseInt(s));
           } catch (NumberFormatException e) {
               System.out.println("could not parse an element: " + s);
           }
       }
       return integers;
   }
```

1.  添加一个如下所示的`main()`方法：

```java
   public static void main(String[] args) {
       List<String> inputList = new ArrayList<>();
       inputList.add("1");
       inputList.add("two");
       inputList.add("3");
       List<Integer> outputList = parseIntegers(inputList);
       int sum = 0;
       for(Integer i: outputList) {
           sum += i;
       }
       System.out.println("Sum is " + sum);
   }
}
```

现在这是我们的输出：

```java
could not parse an element: two
Sum is 4
```

它将 1 和 3 相加，忽略了"two"。这是我们想要的吗？我们假设"two"是正确的数字，并期望它包含在总和中。然而，目前我们将它排除在总和之外，并在日志中添加了一个注释。如果这是一个真实的场景，可能没有人会查看日志，我们提供的结果将是不准确的。这是因为我们捕捉了错误，但没有对其进行有意义的处理。

什么才是更好的方法？我们有两种可能性：要么我们可以假设列表中的每个元素实际上都应该是一个数字，要么我们可以假设会有错误，我们应该对其进行处理。

后者是一个更棘手的方法。也许我们可以将有问题的条目收集到另一个列表中，并将其返回给调用者，然后调用者会将其发送回原始位置进行重新评估。例如，它可以将它们显示给用户，并要求他们进行更正。

前者是一个更简单的方法：我们假设初始列表包含数字字符串。然而，如果这个假设不成立，我们必须让调用者知道。因此，我们应该抛出异常，而不是提供一半正确的总和。

我们不应该采取第三种方法：希望列表包含数字，但忽略那些不是数字的元素。请注意，这是我们做出的选择，但这并不是我们在上面列举两个选项时考虑的。这样编程很方便，但它创建了一个原始业务逻辑中不存在的假设。在这样的情况下要非常小心。确保你写下你的假设，并严格执行它们。不要让编程的便利性迫使你接受奇怪的假设。

如果我们假设初始列表包含数字字符串，我们应该这样编码：

```java
import java.util.ArrayList;
import java.util.List;
public class Main {
   private static List<Integer> parseIntegers(List<String> inputList) {
       List<Integer> integers = new ArrayList<>();
       for(String s: inputList) {
           integers.add(Integer.parseInt(s));
       }
       return integers;
   }
   public static void main(String[] args) {
       List<String> inputList = new ArrayList<>();
       inputList.add("1");
       inputList.add("two");
       inputList.add("3");
       try {
           List<Integer> outputList = parseIntegers(inputList);
           int sum = 0;
           for(Integer i: outputList) {
               sum += i;
           }
           System.out.println("Sum is " + sum);
       } catch (NumberFormatException e) {
           System.out.println("There was a non-number element in the list. Rejecting.");
       }
   }
}
```

输出将简单地如下所示：

```java
There was a non-number element in the list. Rejecting.
```

### 让用户参与

以前的经验法则建议我们不要把问题搁置一边，提供一半正确的结果。现在我们将其扩展到程序是交互式的情况。除非你的程序是批处理过程，通常它与用户有一些交互。在这种情况下，让用户成为问题情况的仲裁者通常是正确的方法。

在我们的例子中，一个字符串无法解析为数字，程序无法做太多事情。然而，如果用户看到了"two"，他们可以用"2"替换它来解决问题。因此，我们不应该试图悄悄地修复问题，而是应该找到方法让用户参与决策过程，并寻求他们的帮助来解决问题。

### 练习 50：向用户寻求帮助

我们可以扩展我们之前的例子，以便我们识别列表中的有问题的条目，并要求用户进行更正：

1.  这是一个处理这种情况的方法：

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;
class NonNumberInListException extends Exception {
   public int index;
   NonNumberInListException(int index, Throwable cause) {
       super(cause);
       this.index = index;
   }
}
public class Main {
   private static List<Integer> parseIntegers(List<String> inputList) throws NonNumberInListException {
       List<Integer> integers = new ArrayList<>();
       int index = 0;
       for(String s: inputList) {
           try {
               integers.add(Integer.parseInt(s));
           } catch (NumberFormatException e) {
               throw new NonNumberInListException(index, e);
           }
           index++;
       }
       return integers;
   }
```

1.  添加一个`main()`方法如下：

```java
   public static void main(String[] args) {
       List<String> inputList = new ArrayList<>();
       inputList.add("1");
       inputList.add("two");
       inputList.add("3");
       boolean done = false;
       while (!done) {
           try {
               List<Integer> outputList = parseIntegers(inputList);
               int sum = 0;
               for(Integer i: outputList) {
                   sum += i;
               }
               System.out.println("Sum is " + sum);
               done = true;
           } catch (NonNumberInListException e) {
               System.out.println("This element does not seem to be a number: " + inputList.get(e.index));
               System.out.print("Please provide a number instead: ");
               Scanner scanner = new Scanner(System.in);
               String newValue = scanner.nextLine();
               inputList.set(e.index, newValue);
           }
       }
   }
}
```

这是一个示例输出：

```java
This element does not seem to be a number: two
Please provide a number instead: 2
Sum is 6
```

请注意，我们确定了有问题的元素，并要求用户对其进行修正。这是让用户参与并给他们一个机会来解决问题的好方法。

### 除非预期会抛出异常

到目前为止，我们一直在建议抛出异常是一件好事，我们不应该压制它们。然而，在某些情况下，这可能并非如此。这提醒我们，关于异常的一切都取决于上下文，我们应该考虑每种情况，而不是盲目地遵循模式。

偶尔，您可能会使用第三方库，并且您可能会向它们提供您的类，以便它们调用您的方法。例如：游戏引擎可能会获取您的对象并调用其`update()`方法，每秒 60 次。在这种情况下，您应该仔细了解如果抛出异常会意味着什么。如果您抛出的异常导致游戏退出，或者显示一个错误发生的弹窗，也许您不应该为不是 showstoppers 的事情抛出异常。假设您在这一帧无法计算所需的值，但也许在下一帧会成功。这值得为此停止游戏吗？也许不值得。

特别是当您重写类/实现接口并将您的对象交给另一个实体来管理时，您应该注意传播异常出您的方法意味着什么。如果调用者鼓励异常，那很好。否则，您可能需要将所有方法包装在广泛的`try/catch`中，以确保您不会因为不是 showstoppers 的事情而泄漏异常。

### 考虑链式和更具体的异常传播

当您将异常传播给调用者时，通常有机会向该异常添加更多信息，以使其对调用者更有用。例如：您可能正在从用户提供的字符串中解析用户的年龄、电话号码、身高等。简单地引发`NumberFormatException`，而不告知调用者是哪个值，这并不是一个很有帮助的策略。相反，为每个解析操作单独捕获`NumberFormatException`给了我们识别有问题的值的机会。然后，我们可以创建一个新的异常对象，在其中提供更多信息，将`NumberFormatException`作为初始原因，并抛出该异常。然后，调用者可以捕获它，并了解哪个实体是有问题的。

之前的练习中，我们使用我们自定义的`NonNumberInListException`来识别列表中有问题的条目的索引，这是这个经验法则的一个很好的例子。在可能的情况下，最好抛出一个我们自己创建的更具信息性的异常，而不是让内部异常在没有太多上下文的情况下传播。

## 总结

在这节课中，我们从实际角度讨论了 Java 中的异常。首先，我们讨论了异常处理背后的动机，以及它如何比其他处理错误情况的方式更有优势。然后，我们以一个新手 Java 程序员的角度，结合强大的 IDE，提供了如何最好地处理和指定异常的指导。之后，我们深入探讨了异常的原因和各种异常类型，以及使用 try/catch、try/catch/finally 和 try with resource 块处理异常的机制。我们最后讨论了一系列最佳实践，以指导您在涉及异常的各种情况下的决策过程。
