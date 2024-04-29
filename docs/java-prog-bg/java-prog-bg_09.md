# 第九章：文件输入和输出

文件 I/O 功能是一个非常强大的工具，可以使现代编程中最困难和令人沮丧的任务之一，即在代码的逻辑上分离的实体之间传输信息，比原本更容易。话虽如此，在本章中，您将学习如何使用`FileWriter`和`BufferedWriter`和`FileReader`和`BufferedReader`类来编写和读取数据文件。我们还将看一下`close()`方法和`Scanner`类的用法。然后您将学习异常处理。最后，我们将看到 I/O 的另一个方面：`Serializable`类。

具体来说，我们将在本章中涵盖以下主题：

+   向文件写入数据

+   从文件读取数据

+   Serializable 类

# 向文件写入数据

这将是一个令人兴奋的章节。首先，我们将看看如何使用 Java 写入文件。为此，我们将声明一个数学序列，前 50 个数字将是数学序列的前两个数字的和。当我们运行以下程序时，我们将看到这 50 个数字打印到我们的`System.out`流中，并且我们将能够在控制台窗口中查看它们：

```java
package writingtofiles; 

public class WritingToFiles { 
    public static void main(String[] args) { 
        for(long number : FibonacciNumbers()) 
        { 
            System.out.println(number); 
        } 
    } 

    private static long[] FibonacciNumbers() 
    { 
        long[] fibNumbers = new long[50]; 
        fibNumbers[0] = 0; 
        fibNumbers[1] = 1; 
        for(int i = 2; i < 50; i++) 
        { 
            fibNumbers[i] = fibNumbers[i - 1] + fibNumbers[i - 2]; 
        } 
        return fibNumbers; 
    } 
} 
```

然而，当我们永久关闭控制台时，这些数字将丢失。为了帮助我们完成这项任务，我们将利用`java.io`库；在这里，`io`代表**输入和输出**：

```java
import java.io.*; 
```

我们将利用这个库中的一个类：`FileWriter`。

# FileWriter 类

`FileWriter`类及其用法可以解释如下：

1.  让我们声明一个新的`FileWriter`类，并且出于稍后会变得明显的原因，让我们明确地将这个`FileWriter`类设置为 null：

```java
        public class WritingToFiles { 
            public static void main(String[] args) { 
                FileWriter out = null; 
```

1.  一旦我们这样做，我们就可以继续实例化它。为了写入文件，我们需要知道两件重要的事情：

+   首先，当然，我们需要知道要写入文件的内容

+   其次，我们的`FileWriter`类需要知道它应该写入哪个文件

1.  当我们使用`FileWriter`类时，我们将它与特定文件关联起来，因此我们将文件名传递给它的构造函数，我们希望它写入该文件。我们的`FileWriter`类能够在没有文件的情况下创建文件，因此我们应该选择一个以`.txt`结尾的名称，这样我们的操作系统就会知道我们正在创建一个文本文件：

```java
        public class WritingToFiles { 
            public static void main(String[] args) { 
                FileWriter out = null; 
                    out = new FileWriter("out.txt"); 
```

尽管我们使用有效的参数调用了`FileWriter`构造函数，NetBeans 仍会告诉我们，我们在这段代码中会得到一个编译器错误。它会告诉我们有一个未报告的异常，即可能在此处抛出`IOException`错误。Java 中的许多异常都标记为已处理异常。这些是函数明确声明可能抛出的异常。`FileWriter`是一个明确声明可能抛出`IOException`错误的函数。因此，就 Java 而言，我们的代码不明确处理这种可能的异常是错误的。

1.  当然，为了处理这个问题，我们只需用`try...catch`块包装我们使用`FileWriter`类的代码部分，捕获`IOException`错误：

1.  如果我们捕获到`IOException`错误，现在可能是打印有用消息到**错误流**的好时机：

```java
        catch(IOException e) 
        { 
             System.err.println("File IO Failed."); 
        } 
```

然后，我们的程序将完成运行，并且将终止，因为它已经到达了`main`方法的末尾。有了这个异常捕获，`FileWriter`的实例化现在是有效和合法的，所以让我们把它用起来。

我们不再需要我们的程序将数字打印到控制台，所以让我们注释掉我们的`println`语句，如下面的代码块所示：

```java
        for(long number : FibonacciNumbers()) 
        { 
            // System.out.println(number); 
        } 
```

我们将用我们的`FileWriter`类做同样的逻辑处理：

```java
            try{ 
                out = new FileWriter("out.txt"); 
                for(long number : FibonacciNumbers()) 
                { 
                    // System.out.println(number); 
                } 
```

`FileWriter`类没有`println`语句，但它有`write`方法。每当我们的`foreach`循环执行时，我们希望使用`out.write(number);`语法将数字写入我们的文件。

1.  不幸的是，`write`方法不知道如何以“长数字”作为输入；它可以接受一个字符串，也可以接受一个整数。因此，让我们使用静态的`String`类方法`valueOf`来获取我们的“长数字”的值，以便将数字打印到我们的文件中：

```java
        for(long number : FibonacciNumbers()) 
        { 
            out.write(String.valueOf(number)); 
            // System.out.println(number); 
        } 
```

因此，我们现在应该拥有一个成功的程序的所有部分：

+   +   首先，我们声明并实例化了我们的`FileWriter`类，并给它一个文件名

+   然后，我们循环遍历我们的斐波那契数列，并告诉我们的`FileWriter`类将这些数字写入`out.txt`

然而，问题是`out.txt`在哪里？我们没有给`FileWriter`类一个完整的系统路径，只是一个文件名。我们知道`FileWriter`类有能力创建这个文件，如果它不存在，但在我们系统的目录中，`FileWriter`类会选择在哪里创建这个文件？

要回答这个问题，我们需要知道 NetBeans 将为我们编译的程序创建`.jar`文件的位置。为了找出这一点，我们可以打开控制台窗口并构建我们的程序。在这里，NetBeans 会告诉我们它正在创建所有文件的位置。例如，在我的情况下，有一个名为`WritingToFiles`的文件夹；如果我们导航到这个文件夹，我们会看到我们的项目文件。其中一个文件是`dist`，缩写为**可分发**，这就是我们的 JAR 文件将被编译到的地方：

![](img/4b4712a9-692f-47dc-bac8-085a3768f239.png)

JAR 文件是我们能够获得的最接近原始 Java 代码的可执行文件。因为 Java 代码必须由 Java 虚拟机解释，我们实际上无法创建 Java 可执行文件；然而，在大多数安装了 Java 的操作系统中，我们可以通过双击运行 JAR 文件，就像运行可执行文件一样。我们还可以告诉 Java 虚拟机使用 Java 命令行`-jar`命令启动和运行 JAR 文件，后面跟着我们想要执行的文件的名称，当然：

![](img/0df1bcb5-9a5d-4ff4-9675-e3e73121a04d.png)

当我们提交这个命令时，Java 虚拟机解释并执行了我们的`WritingToFiles.jar`程序。看起来好像成功了，因为在目录中创建了一个新文件，如前面的截图所示。这是工作目录，直到我们移动它，这就是执行 JAR 文件的命令将执行的地方。所以这就是我们的`FileWriter`类选择创建`out.txt`的地方。

# 使用 close()方法释放资源

不幸的是，当我们打开`out.txt`时，我们看不到任何内容。这让我们相信我们的文件写入可能没有成功。那么出了什么问题呢？嗯，使用`FileWriter`的一个重要部分我们没有考虑到。当我们创建我们的`FileWriter`时，它会打开一个文件，每当我们打开一个文件时，我们应该确保最终关闭它。从代码的角度来看，这是相当容易做到的；我们只需在我们的`FileWriter`上调用`close`方法：

```java
public class WritingToFiles { 
    public static void main(String[] args) { 
        FileWriter out = null; 
        try{ 
            out = new FileWriter("out.txt"); 
            for(long number : FibonacciNumbers()) 
            { 
                out.write(String.valueOf(number)); 
                // System.out.println(number); 
            } 

        } 
        catch(IOException e) 
        { 
            System.err.println("File IO Failed."); 
        } 

        finally{ 
            out.close(); 
        } 
    } 
```

有一个熟悉的错误消息出现，如下面的截图所示；`out.close`也可以报告一个`IOException`错误：

![](img/9d911cd7-9921-4e02-87e7-44af424facee.png)

我们可以将`out.close`放在另一个`try...catch`块中，并处理这个`IOException`错误，但如果我们的文件无法关闭，那就意味着有非常严重的问题。在这种情况下，将这个异常传播到更健壮的代码而不是我们相当封闭的`WritingToFiles`程序可能更合适。如果我们不处理这个异常，这将是默认的行为，但我们确实需要让 Java 知道从我们当前的代码中向上传播这个异常是可能的。

当我们声明我们的`main`方法时，我们还可以让 Java 知道这个方法可能抛出哪些异常类型：

```java
public static void main(String[] args) throws IOException 
```

在这里，我们告诉 Java，在某些情况下，我们的`main`方法可能无法完美执行，而是会抛出`IOException`错误。现在，任何调用`WritingToFiles`的`main`方法的人都需要自己处理这个异常。如果我们构建了 Java 程序，然后再次执行它，我们会看到`out.txt`已经被正确打印出来。不幸的是，我们忘记在输出中加入新的行，所以数字之间没有可辨认的间距。当我们写入时，我们需要在每个数字后面添加`\r\n`。这是一个新的换行转义字符语法，几乎可以在每个操作系统和环境中看到：

```java
for(long number : FibonacciNumbers()) 
{ 
    out.write(String.valueOf(number) + "\r\n"); 
    // System.out.println(number); 
} 
```

再次构建、运行并查看`out.txt`，现在看起来非常有用：

![](img/b2524382-1cf8-49fa-8b30-73d4e1caacba.png)

所以这是我们最初的目标：将这个斐波那契数列打印到一个文件中。在我们完成之前，还有一些事情要快速看一下。让我们看看如果我们再次运行程序会发生什么，然后看看我们的输出文本文件。文本文件看起来和之前一样，这可能是预期的，也可能不是。似乎`FileWriter`是否清除这个文件并写入全新的文本是一种抉择，或者它是否会在文件中现有文本后面放置追加的文本。默认情况下，我们的`FileWriter`会在写入新内容之前清除文件，但我们可以通过`FileWriter`构造函数中的参数来切换这种行为。比如，我们将其追加行为设置为`true`：

```java
try { 
    out = new FileWriter("out.txt", true); 
```

现在构建项目，运行它，并查看`out.txt`；我们会看到比以前多两倍的信息。我们的文本现在被追加到末尾。

# BufferedWriter 类

最后，在 Java 中有很多不同的写入器可供我们使用，`FileWriter`只是其中之一。我决定在这里向你展示它，因为它非常简单。它接受一些文本并将其打印到文件中。然而，很多时候，你会看到`FileWriter`被`BufferedWriter`类包裹。现在`BufferedWriter`类的声明将看起来像以下代码块中给出的声明，其中`BufferedWriter`被创建并给定`FileWriter`作为其输入。

`BufferedWriter`类非常酷，因为它会智能地接受你给它的所有命令，并尝试以最有效的方式将内容写入文件：

```java
package writingtofiles; 

import java.io.*; 

public class WritingToFiles { 
    public static void main(String[] args) throws IOException { 
        BufferedWriter out = null; 

        try { 
            out = new BufferedWriter(new FileWriter
             ("out.txt", true)); 

            for(long number : FibonacciNumbers()) 
            { 
                out.write(String.valueOf(number) + "\r\n"); 
                //System.out.println(number); 
            } 
        } 
        catch(IOException e) { 
            System.err.println("File IO Failed."); 
        } 
        finally{ 
            out.close(); 
        } 
    } 
```

我们刚刚编写的程序从我们的角度来看，做的事情与我们现有的程序一样。然而，在我们进行许多小写入的情况下，`BufferedWriter`可能会更快，因为在适当的情况下，它会智能地收集我们给它的写入命令，并以适当的块执行它们，以最大化效率：

```java
out = new BufferedWriter(new FileWriter("out.txt", true)); 
```

因此，很多时候你会看到 Java 代码看起来像前面的代码块，而不是单独使用`FileWriter`。

# 从文件中读取数据

作为程序员，我们经常需要从文件中读取输入。在本节中，我们将快速看一下如何从文件中获取文本输入。

我们已经告诉 Java，有时我们的`main`方法会简单地抛出`IOException`错误。以下代码块中的`FileWriter`和`FileReader`对象可能会因为多种原因创建多个`IOException`错误，例如，如果它们无法连接到它们应该连接的文件。

```java
package inputandoutput; 

import java.io.*; 

public class InputAndOutput { 
    public static void main(String[] args) throws IOException { 
        File outFile = new File("OutputFile.txt"); 
        File inFile = new File("InputFile.txt"); 

        FileWriter out = new FileWriter(outFile); 
        FileReader in = new FileReader(inFile); 

        //Code Here... 

        out.close(); 
        in.close(); 
    } 
} 
```

在为实际应用编写实际程序时，我们应该始终确保以合理的方式捕获和处理异常，如果真的有必要，就将它们向上抛出。但是我们现在要抛出所有的异常，因为我们这样做是为了学习，我们不想现在被包裹在`try...catch`块中的所有代码所拖累。

# FileReader 和 BufferedReader 类

在这里，您将通过我们已经有的代码（请参阅前面的代码）学习`FileReader`类。首先，按照以下步骤进行：

1.  我已经为我们声明了`FileWriter`和`FileReader`对象。`FileReader`是`FileWriter`的姊妹类。它能够，信不信由你，从文件中读取文本输入，并且它的构造方式非常相似。它在构造时期望被给予一个文件，以便在其生命周期内与之关联。

1.  与其简单地给`FileReader`和`FileWriter`路径，我选择创建`File`对象。Java 文件对象只是对现有文件的引用，我们告诉该文件在创建时将引用哪个文件，如下面的代码块所示：

```java
        package inputandoutput; 

        import java.io.*; 

        public class InputAndOutput { 
            public static void main(String[] args)
             throws IOException { 
                File outFile = new File("OutputFile.txt"); 
                File inFile = new File("InputFile.txt"); 

                FileWriter out = new FileWriter(outFile); 
                FileReader in = new FileReader(inFile); 

                //Code Here... 
                out.write(in.read()); 
                out.close(); 
                in.close(); 
            } 
        } 
```

在这个程序中，我们将使用包含一些信息的`InputFile.txt`。此外，我们将使用`OutputFile.txt`，目前里面没有信息。我们的目标是将`InputFile`中的信息移动到`OutputFile`中。`FileWriter`和`FileReader`都有一些在这里会有用的方法。

我们的`FileWriter`类有`write`方法，我们知道可以用它来将信息放入文件中。同样，`FileReader`有`read`方法，它将允许我们从文件中获取信息。如果我们简单地按顺序调用这些方法并运行我们的程序，我们会看到信息将从`InputFile`中取出并放入`OutputFile`中：

![](img/bf4cf512-f991-429b-b835-a717aeea1821.png)

不幸的是，`OutputFile`中只出现了一个字符：`InputFile`文本的第一个字符。看起来我们的`FileReader`类的`read`方法只获取了最小可获取的文本信息。不过这对我们来说并不是问题，因为我们是程序员。

1.  我们可以简单地使用`in.read`方法循环遍历文件，以获取在`InputFile`文件中对我们可用的所有信息：

```java
        String input = ""; 
        String newInput; 
        out.write(in.read()); 
```

1.  然而，我们可以通过用`BufferedReader`类包装`FileReader`来使生活变得更加轻松。类似于我们用`BufferedWriter`包装`FileWriter`的方式，用`BufferedReader`包装`FileReader`将允许我们在任何给定时间收集不同长度的输入：

```java
        FileWriter out = new FileWriter(outFile); 
        BufferedReader in = new BufferedReader(new FileReader(inFile)); 
```

与包装我们的`FileWriter`类一样，包装我们的`FileReader`类几乎总是一个好主意。`BufferedReader`类还可以保护`FileReader`类，使其不受`FileReader`类一次性无法容纳的过大文件的影响。这种情况并不经常发生，但当发生时，可能会是一个相当令人困惑的错误。这是因为`BufferedReader`一次只查看文件的部分；它受到了那个实例的保护。

`BufferedReader`类还将让我们使用`nextLine`方法，这样我们就可以逐行从`InputFile`中收集信息，而不是逐个字符。不过，无论如何，我们的`while`循环看起来都会非常相似。这里唯一真正的挑战是我们需要知道何时停止在`InputFile`文件中寻找信息。为了弄清楚这一点，我们实际上会在`while`循环的条件部分放一些功能代码。

1.  我们将为这个`newInput`字符串变量分配一个值，这个值将是`in.readLine`。我们之所以要在`while`循环的条件部分进行这个赋值，是因为我们可以检查`newInput`字符串被分配了什么值。这是因为如果`newInput`字符串根本没有被分配任何值，那就意味着我们已经到达了文件的末尾：

```java
        while((newInput = in.readLine()) !=null) 
        { 

        } 
```

如果`newInput`有一个值，如果变量不是空的，那么我们会知道我们已经从文件中读取了合法的文本，实际上是一整行合法的文本，因为我们使用了`readLine`方法。

1.  在这种情况下，我们应该添加一行新的文本，即 `input += newInput;` 到我们的输入字符串。当我们执行完我们的 `while` 循环时，当 `newInput` 字符串被赋予值 `null`，因为读者没有其他内容可读时，我们应该打印出我们一直在构建的字符串：

```java
        while((newInput = in.readLine()) != null) 
        { 
            input += newInput; 
        } 
        out.write(input); 
```

1.  现在，因为我们的 `BufferedReader` 类的 `readLine` 方法专门读取文本行，它不会在这些行的末尾附加结束行字符，所以我们必须自己做：

```java
         while((newInput = in.readLine()) != null) 
        { 
             input += (newInput + "\r\n"); 
        } 
```

所以，我们已经执行了这个程序。让我们去我们的目录，看看复制到 `OutputFile` 的内容：

![](img/5f074b43-77be-47bf-955d-653d75b19acf.png)

好了；`InputFile` 和 `OutputFile` 现在具有相同的内容。这就是 Java 中基本文件读取的全部内容。

还有一些其他需要注意的事情。就像我们可以用 `BufferedReader` 包装 `FileReader` 一样，如果我们导入 `java.util`，我们也可以用 `Scanner` 包装 `BufferedReader`：

```java
        Scanner in = new Scanner(new BufferedReader
        (new FileReader(inFile))); 
```

这将允许我们使用 `Scanner` 类的方法来获取我们正在读取的文本中与某些模式匹配的部分。还要注意的是，`FileReader` 类及其包装类只适用于从 Java 文件中读取文本。如果我们想要读取二进制信息，我们将使用不同的类；当您学习如何在 Java 中序列化对象时，您将看到更多相关内容。

# 可序列化类

通常，当我们处理实际代码之外的信息时，我们处理的是从文件中获取的或写入文件的人类可读的文本，或者来自输入或输出流的文本。然而，有时，人类可读的文本并不方便，我们希望使用更适合计算机的信息。通过一种称为**序列化**的过程，我们可以将一些 Java 对象转换为二进制流，然后可以在程序之间传输。这对我们来说不是一种友好的方法，我们将在本节中看到。对我们来说，序列化的对象看起来像是一团乱码，但另一个了解该对象类的 Java 程序可以从序列化的信息中重新创建对象。

然而，并非所有的 Java 对象都可以被序列化。为了使对象可序列化，它需要被标记为可以被序列化的对象，并且它只能包含那些本身可以被序列化的成员。对于一些对象来说，那些依赖外部引用或者那些只是没有所有成员都标记为可序列化的对象，序列化就不合适。参考以下代码块：

```java
package serialization; 

public class Car { 
    public String vin; 
    public String make; 
    public String model; 
    public String color; 
    public int year; 

    public Car(String vin, String make, String model, String 
     color, int year) 
    { 
        this.vin = vin; 
        this.make = make; 
        this.model = model; 
        this.color = color; 
        this.year = year; 
    } 

    @Override  
    public String toString() 
    { 
        return String.format
         ("%d %s %s %s, vin:%s", year, color, make, model, vin); 
    } 
} 
```

在给定程序中的类（在上面的代码块中）是序列化的一个主要候选对象。它的成员是一些字符串和整数，这些都是 Java 标记为可序列化的类。然而，为了将 `Car` 对象转换为二进制表示，我们需要让 Java 知道 `Car` 对象也是可序列化的。

我们可以通过以下步骤来实现这一点：

1.  我们将需要 `io` 库来实现这一点，然后我们将让 Java 知道我们的 `Car` 对象实现了 `Serializable`：

```java
        import java.io.*; 
        public class Car implements Serializable{ 
```

这告诉 Java，`Car` 对象的所有元素都可以转换为二进制表示。除非我们已经查看了对象并经过深思熟虑并确定这是一个安全的假设，否则我们不应该告诉 Java 对象实现了 `Serializable`。

所以，我们现在将 `Car` 标记为 `Serializable` 类，但这当然是本节的简单部分。我们的下一个目标是利用这个新功能来创建一个 `Car` 对象，将其序列化，打印到文件中，然后再读取它。

1.  为此，我们将创建两个新的 Java 类：一个用于序列化我们的对象并将其打印到文件中，另一个类用于反序列化我们的对象并从文件中读取它。

1.  在这两个类中，我们将创建 `main` 方法，以便我们可以将我们的类作为单独的 Java 程序运行。

# 序列化对象

让我们从`Serialize`类开始，如下所示：

1.  我们要做的第一件事是为我们序列化的对象。所以让我们继续实例化一个新的`Car`对象。`Car`类需要四个字符串和一个整数作为它的变量。它需要一个车辆识别号码、制造商、型号、颜色和年份。因此，我们将分别给它所有这些：

```java
        package serialization; 
        public class Serialize { 
            public static void main(String[] args) { 
                Car c =  new Car("FDAJFD54254", "Nisan", "Altima",
                "Green", 2000); 
```

一旦我们创建了我们的`Car`对象，现在是时候打开一个文件并将这个`Car`序列化输出。在 Java 中打开文件时，我们将使用一些不同的管理器，这取决于我们是否想要将格式化的文本输出写入这个文件，还是我们只打算写入原始二进制信息。

1.  序列化对象是二进制信息，所以我们将使用`FileOutputStream`来写入这些信息。`FileOutputStream`类是使用文件名创建的：

```java
        FileOutputStream outFile = new FileOutputStream("serialized.dat"); 
```

因为我们正在写入原始二进制信息，所以指定它为文本文件并不那么重要。我们可以指定它为我们想要的任何东西。无论如何，我们的操作系统都不会知道如何处理这个文件，如果它尝试打开它的话。

我们将想要将所有这些信息包围在一个`try...catch`块中，因为每当我们处理外部文件时，异常肯定会被抛出。如果我们捕获到异常，让我们只是简单地打印一个错误消息：

```java
            try{ 
                FileOutputStream outFile =  
                new FileOutputStream("serialized.dat"); 
            } 
            catch(IOException e) 
            { 
                 System.err.println("ERROR"); 
             } 
```

请注意，我们需要在这里添加很多输入；让我们只是导入整个`java.io`库，也就是说，让我们导入`java.io.*;`包。

现在我认为我们可以继续了。我们已经创建了我们的`FileOutputStream`类，这个流非常好。但是，我们可以用另一个更专门用于序列化 Java 对象的字符串来包装它。

1.  这是`ObjectOutputStream`类，我们可以通过简单地将它包装在现有的`FileOutputStream`对象周围来构造`ObjectOutputStream`对象。一旦我们创建了这个`ObjectOutputStream`对象并将文件与之关联，将我们的对象序列化并将其写入这个文件变得非常容易。我们只需要使用`writeObject`方法，并提供我们的`Car`类作为要写入的对象。

1.  一旦我们将这个对象写入文件，我们应该负责关闭我们的输出字符串：

```java
        try{ 
            FileOutputStream outFile = new
            FileOutputStream("serialized.dat"); 
            ObjectOutputStream out = new ObjectOutputStream(outFile); 
            out.writeObject(c); 
            out.close(); 
        } 
        catch(IOException e) 
        { 
             System.err.println("ERROR"); 
         } 
```

现在我认为我们可以运行我们接下来的程序了。让我们看看会发生什么：

```java
        package serialization; 

        import java.io.*; 

        public class Serialize { 
            public static void main(String argv[]) { 
                Car c = new Car("FDAJFD54254", "Nisan", "Altima",
                "Green", 2000); 

                try { 
                    FileOutputStream outFile = new
                    FileOutputStream("serialized.dat"); 
                    ObjectOutputStream out = new
                    ObjectOutputStream(outFile); 
                    out.writeObject(c); 
                    out.close(); 
                } 
                catch(IOException e) 
                { 
                    System.err.println("ERROR"); 
                } 
            }
        } 
```

在这个 Java 项目中，我们有多个`main`方法。因此，就 NetBeans 而言，当我们运行程序时，我们应该确保右键单击要输入的`main`方法的类，并专门运行该文件。当我们运行这个程序时，我们实际上并没有得到任何有意义的输出，因为我们没有要求任何输出，至少没有抛出错误。但是，当我们前往这个项目所在的目录时，我们会看到一个新文件：`serialize.dat`。如果我们用记事本编辑这个文件，它看起来相当荒谬：

![](img/d8f4ce67-e72d-41fa-a47a-4844defd3871.png)

这肯定不是一种人类可读的格式，但有一些单词，或者单词的片段，我们是能够识别的。它看起来肯定是正确的对象被序列化了。

# 反序列化对象

让我们从我们的另一个类开始，也就是`DeSerialize`类，并尝试编写一个方法，从我们已经将其序列化信息写入的文件中提取`Car`对象。这样做的步骤如下：

1.  再一次，我们需要一个`Car`对象，但这一次，我们不打算用构造函数值来初始化它；相反，我们将把它的值设置为我们从文件中读取回来的对象。我们在反序列化器中要使用的语法将看起来非常类似于我们在`Serialize`类`main`方法中使用的语法。让我们只是复制`Serialize`类的代码，这样我们就可以在构建`DeSerialize`类的`main`方法时看到镜像相似之处。

在之前讨论的`Serialize`类中，我们在`Serialize`类的方法中犯了一个不负责任的错误。我们关闭了`ObjectOutputStream`，但没有关闭`FileOutputStream`。这并不是什么大问题，因为我们的程序立即打开了这些文件，执行了它的功能，并在终止 Java 时销毁了这些对象，文件知道没有其他东西指向它们。因此，我们的操作系统知道这些文件已关闭，现在可以自由地写入。但是，在一个持续很长时间甚至无限期的程序中，不关闭文件可能会产生一些非常奇怪的后果。

当我们像在这个程序中所做的那样嵌套`FileInput`或`Output`类时，通常会以我们访问它们的相反顺序关闭文件。在这个程序中，我们在调用`out.close`之前调用`outFile.close`是没有意义的，因为在这一瞬间，我们的`ObjectOutputStream`对象将引用一个它无法访问的文件，因为内部的`FileOutputStream`类已经关闭了。现在删除`Car c = new Car("FDAJFD54254", " Nisan", "Altima", "Green", 2000);`在当前的`DeSerialize.java`类中。

搞定了这些，我们已经复制了我们的代码，现在我们要对其进行一些修改。所以，我们现在不是将对象序列化到文件中，而是从文件中读取序列化的对象。

1.  因此，我们将使用它的姐妹类`FileInputStream`，而不是`FileOutputStream`：

```java
        FileInputStream outFile = new FileInputStream("serialized.dat"); 
```

1.  让我们再次导入`java.io`。我们希望引用与前面的代码中给出的相同的文件名；另外，让我们聪明地命名我们的变量。

1.  以类似的方式，我们将`FileInputStream`包装为`ObjectInputStream`，而不是`ObjectOutputStream`，它仍然引用相同的文件：

```java
        ObjectInputStream in = new ObjectInputStream(outFile); 
```

当然，这一次我们对将对象写入文件没有兴趣，这很好，因为我们的`InputStream`类没有权限或知识来写入这个文件；然而，它可以从文件中读取对象。

1.  `ReadObject`不需要任何参数；它只是简单地读取那个文件中的任何对象。当它读取到那个对象时，将其赋给我们的`Car`对象。当然，`ReadObject`只知道它将从文件中获取一个对象；它不知道那个对象的类型是什么。序列化的一个弱点是，我们确实被迫去相信并将这个对象转换为预期的类型：

```java
         c = (Car)in.readObject(); 
```

1.  一旦我们这样做了，就是时候以相反的顺序关闭我们的文件读取器了：

```java
        try { 
            FileInputStream inFile = new FileInputStream("serialized.dat"); 
            ObjectInputStream in = new ObjectInputStream(inFile); 
            c = (Car)in.readObject(); 
            in.close(); 
            inFile.close(); 
        } 
```

1.  现在有另一种被处理的异常类型，即`ClassNotFoundException`：![](img/f1a66c24-7b73-458f-a70f-b7e7bfb59c3c.png)

如果我们的`readObject`方法失败，就会抛出这个异常。

所以，让我们捕获`ClassNotFoundException`，为了保持简单和流畅，我们将像处理之前的 I/O 异常一样，抛出或打印出错误消息：

```java
            catch(ClassNotFoundException e) 

            { 
                System.err.println("ERROR"); 
            } 
```

1.  现在我们需要一种方法来判断我们的程序是否工作。因此，在最后，让我们尝试使用自定义的`toString`函数打印出我们汽车的信息，也就是`System.out.println(c.toString());`语句。NetBeans 提示我们，变量`c`在这个时候可能尚未初始化，如下面的截图所示：

![](img/209ae27d-b146-44b4-99d6-623c1f270534.png)

有些编程语言会让我们犯这个错误，我们的`Car`对象可能尚未初始化，因为这个`try`块可能已经失败了。为了让 NetBeans 知道我们意识到了这种情况，或者说，让 Java 知道我们意识到了这种情况，我们应该初始化我们的`Car`对象。我们可以简单地将其初始化为值`null`：

```java
            public class DeSerialize { 
                public static void main(String[] args) { 
                    Car c = null; 

                    try { 
                        FileInputStream inFile = new
                        FileInputStream("serialized.dat"); 
                        ObjectInputStream in = new
                        ObjectInputStream(inFile); 
                        c = (Car)in.readObject(); 
                        in.close(); 
                        inFile.close(); 
                    } 
                    catch(IOException e) 
                    { 
                         System.err.println("ERROR"); 
                    } 
                    catch(ClassNotFoundException e) 

                    { 
                         System.err.println("ERROR"); 
                    } 
                    System.out.println(c.toString()); 
                } 

            } 
```

现在是我们真相的时刻。让我们执行主方法。当我们在控制台中运行我们的文件时，我们得到的输出是一个 PIN 码：`2000 Green Nisan Altima with vin: FDAJFD54254`。如下面的截图所示：

![](img/ff4aebb5-e973-413e-98e8-99e163e4e226.png)

这是我们在`Serialize.java`类的`main`方法中声明并序列化到文件中的同一辆车。显然，我们取得了成功。对象的序列化是 Java 非常优雅和出色的功能之一。

# 总结

在本章中，我们经历了编写和读取数据文件的过程，我们看到了`FileWriter`和`FileReader`类的用法，以及如何使用`close()`方法释放资源。我们还学习了如何捕获异常并处理它。然后，您学习了如何使用`BufferedWriter`和`BufferedReader`类分别包装`FileWriter`和`FileReader`类。最后，我们看到了 I/O 的另一个方面：`Serializable`类。我们分析了序列化的含义以及在序列化和反序列化对象方面的用法。

在下一章中，您将学习基本的 GUI 开发。
