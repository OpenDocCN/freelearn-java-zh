# 11. 进程

概述

在本章中，我们将快速了解 Java 如何处理进程。你将从探索`Runtime`和`ProcessBuilder`类开始，了解它们的功能以及如何启动它们，然后从任一类创建一个进程。你将学习在父进程和子进程之间发送和接收数据，以及如何将进程的结果存储在文件中。在本章的最终活动中，你将使用这些技能创建一个父进程，该进程将启动一个子进程，该子进程将打印一个结果（然后由父进程捕获）到终端。

# 简介

`java.lang.Process`类用于查找有关运行时进程的信息并启动它们。如果你想了解`Process`类是如何工作的，你可以从查看`Runtime`类开始。所有 Java 程序都包含一个`Runtime`类的实例。可以通过调用`getRuntime()`方法并将结果分配给`Runtime`类的变量来获取有关`Runtime`类的信息。有了这个，就可以获取有关控制你程序的**JVM**环境的信息：

```java
public class Example01 {
    public static void main(String[] args) {
        Runtime runtime = Runtime.getRuntime();
        System.out.println("Processors: " + runtime.availableProcessors());
        System.out.println("Total memory: " + runtime.totalMemory());
        System.out.println("Free memory: " + runtime.freeMemory());
    }
}
```

进程携带有关在计算机上启动的程序的信息。每个操作系统处理进程的方式都不同。`Process`类提供了一个机会，以相同的方式控制它们。这是通过`Runtime`类的一个单独的方法实现的，称为`exec()`，它返回一个`Process`类的对象。`Exec`有不同的实现，允许你简单地发出一个命令，或者通过修改环境变量甚至程序运行的目录来实现。

# 启动进程

如前所述，进程是通过`exec()`启动的。让我们看看一个简单的例子，我们将调用 Java 编译器，这在任何操作系统的终端上都是用相同的方式完成的：

```java
import java.io.IOException;
public class Example02 {
public static void main(String[] args) {
    Runtime runtime = Runtime.getRuntime();
    try {
        Process process = runtime.exec("firefox");
    } catch (IOException ioe) {
        System.out.println("WARNING: something happened with exec");
    }
  }
}
```

当运行此示例时，如果你恰好安装了 Firefox，它将自动启动。你可以将其更改为计算机上的任何其他应用程序。程序将无错误退出，但除了这一点之外，它不会做任何事情。

现在，让我们在先前的例子中添加几行，以便你刚刚打开的程序将在 5 秒后关闭：

```java
import java.io.IOException;
import java.util.concurrent.TimeUnit;
public class Example03 {
    public static void main(String[] args) {
        Runtime runtime = Runtime.getRuntime();
        Process process = null;
        try {
            process = runtime.exec("firefox");
        } catch (IOException ioe) {
            System.out.println("WARNING: something happened with exec");
        }
        try {
            process.waitFor(5, TimeUnit.SECONDS);
        } catch (InterruptedException ie) {
            System.out.println("WARNING: interruption happened");
        }
        process.destroy();
    }
}
```

`waitFor(timeOut, timeUnit)`方法将等待进程结束 5 秒钟。如果它是没有参数的`waitFor()`，它将等待程序自行结束。在 5 秒超时之后，进程变量将调用`destroy()`方法，这将立即停止进程。因此，在短时间内打开和关闭应用程序。

有一种启动进程的方法，不需要创建`Runtime`对象。这种方法使用`ProcessBuilder`类。构建`ProcessBuilder`对象将需要实际要执行的命令作为参数。以下是对先前示例的修订，增加了这个新构造函数：

```java
import java.io.IOException;
public class Example04 {
    public static void main(String[] args) {
        ProcessBuilder processBuilder = new ProcessBuilder("firefox");
        Process process = null;
        try {
            process = processBuilder.start();
        } catch (IOException ioe) {
            System.out.println("WARNING: something happened with exec");
        }
        try {
            process.waitFor(10, TimeUnit.SECONDS);
        } catch (InterruptedException ie) {
            System.out.println("WARNING: interruption happened");
        }
        process.destroy();
    }
}
```

有几件事情你应该注意。首先，该过程包括在构造函数中将命令作为参数调用。然而，直到你调用 `processBuilder.start()`，这个过程才启动。唯一的问题是，`ProcessBuilder` 对象不包含与 `Process` API 相同的方法。例如，`waitFor()` 和 `destroy()` 等方法不可用，因此，如果需要这些方法，你必须在程序中调用它之前实例化一个 `Process` 对象。

## 向子进程发送输入

一旦进程开始运行，向其中传递一些数据将很有趣。让我们创建一个小程序，它会将你在 CLI 上输入的任何内容 `echo` 回来。稍后，我们将编写一个程序来启动第一个应用程序并向它发送文本。这个简单的 `echo` 程序可能如下所示：

```java
public class Example05 {
    public static void main(String[] args) throws java.io.IOException
    {
        int c;
        System.out.print ("Let's echo: ");
        while ((c = System.in.read ()) != '\n')
        System.out.print ((char) c);
    }
}
```

如你所见，这个简单的程序将读取 `System.in` 流，直到你按下 *Enter*。一旦发生这种情况，它将优雅地退出：

```java
Enter some text: Hello World
Hello World
Process finished with exit code 0
```

在前面的输出第一行中，我们输入字符串 '`Hello World`' 作为此示例，它在下一行被 echo。接下来，你可以创建另一个程序来启动此示例并向它发送一些文本：

```java
Example06.java
20     try {
21       process.waitFor(5, TimeUnit.SECONDS);
22     } catch (InterruptedException ie) {
23       System.out.println("WARNING: interrupted exception fired");
24     }
25
26     OutputStream out = process.getOutputStream();
27     Writer writer = new OutputStreamWriter(out);
28     writer.write("This is how we roll!\n"); // EOL to ensure the process sends          back
29
30     writer.flush();
31     process.destroy();
32   }
33 }
https://packt.live/2pEJLiw
```

此示例有两个有趣的技巧，你需要注意。第一个是调用前一个示例。由于我们必须启动一个 Java 应用程序，我们需要使用 `cp` 参数调用 `java` 可执行文件，这将指示 JVM 应在哪个目录中查找编译的示例。你刚刚编译并尝试了 *Example05*，这意味着你的计算机中已经有一个编译好的类。

注意

在调用 `cp` 参数之后，在 Linux/macOS 中，你需要在类名之前添加一个冒号 (`:`)，而在 Windows 的情况下，你应该使用分号 (`;`)。

一旦编译此示例，其相对于前一个示例的相对路径是 `../../../../Example05/out/production/Example05`。这可能会因你如何命名项目文件夹而完全不同。

第二件需要注意的事情也在代码列表中突出显示。在那里，你可以看到与来自进程的 `OutStream` 链接的 `OutStream` 的声明。换句话说，我们正在将 *Example06* 的输出流链接到 *Example05* 应用程序的 `System.in`。为了能够向它写入字符串，我们构建了一个 `Writer` 对象，该对象公开了一个具有向流发送字符串能力的 `write` 方法。

我们可以使用以下命令从 CLI 调用此示例：

```java
usr@localhost:~/IdeaProjects/chapter11/[...]production/Example06$ java Example06
```

结果什么也没有。原因是 echo 示例 (*Example05*) 中的 `System.out` 没有对启动进程的应用程序开放。如果我们想使用它，我们需要在 *Example06* 中捕获它。我们将在下一节中看到如何做到这一点。

# 捕获子进程的输出

现在我们有两个不同的程序；一个可以独立运行（*Example05*），另一个是从另一个程序中执行的，它也会尝试向它发送信息并捕获其输出。本节的目的就是捕获*Example05*的输出并将其打印到终端。

为了捕获子进程发送到`System.out`的内容，我们需要在父类中创建一个`BufferedReader`，它将从可以由进程实例化的`InputStream`中获取数据。换句话说，我们需要增强*Example06*，如下所示：

```java
InputStream in = process.getInputStream();
Reader reader = new InputStreamReader(in);
BufferedReader bufferedReader = new BufferedReader(reader);
String line = bufferedReader.readLine();
System.out.println(line);
```

需要使用`BufferedReader`的原因是我们使用行尾（`EOL`或"`\n`"）作为进程间消息的标记。这允许使用`readLine()`等方法，这些方法将阻塞程序直到捕获到 EOL；否则，我们可以坚持使用`Reader`对象。

一旦将此内容添加到示例中，从终端调用之前的程序将产生以下输出：

```java
usr@localhost:~/IdeaProjects/chapter11/[...]production/Example06$ java Example06
Let's echo: This is how we roll!
```

在此输出之后，程序将结束。

需要考虑的一个重要方面是，由于`BufferedReader`具有缓冲性质，它需要使用`flush()`方法来强制将我们发送到缓冲区的数据发送到子进程。否则，当**JVM**给它优先级时，它将永远等待，这最终可能导致程序停滞。

# 将子进程的输出存储到文件中

将数据存储在文件中不是很有用吗？这可能是你可能会对有一个进程来运行程序（或一系列程序）感兴趣的原因之一——将它们的输出捕获到日志文件中以供研究。通过向进程启动器添加一个小修改，你可以捕获其他程序发送到`System.out`的任何内容。这真的很有用，因为你可以创建一个程序，它可以用来启动操作系统中任何现有的命令并捕获所有输出，这些输出可以用于稍后进行某种类型的成果分析：

```java
Example07.java
26         // write to the child's System.in
27         OutputStream out = process.getOutputStream();
28         Writer writer = new OutputStreamWriter(out);
29         writer.write("This is how we roll!\n");
30         writer.flush();
31 
32         // prepare the data logger
33         File file = new File("data.log");
34         FileWriter fileWriter = new FileWriter(file);
35         BufferedWriter bufferedWriter = new BufferedWriter(fileWriter);
36 
37         // read from System.out from the child
38         InputStream in = process.getInputStream();
39         Reader reader = new InputStreamReader(in);
40         BufferedReader bufferedReader = new BufferedReader(reader);
41         String line = bufferedReader.readLine();
https://packt.live/33X3Wal
```

结果不仅会将结果写入终端，还会创建一个`data.log`文件，其中包含完全相同的句子。

## 活动一：创建一个父进程以启动子进程

在这个活动中，我们将创建一个父进程，该进程将启动一个子进程，该子进程将打印出一系列递增的数字。子进程的结果将被父进程捕获，然后将其打印到终端。

为了防止程序无限运行达到无穷大，子进程应在达到某个数字时停止。让我们以`50`作为这个活动的限制，此时计数器将退出。

同时，父进程将读取输入并将其与某个数字进行比较，例如 37，之后计数器应重新启动。为了请求子进程重新启动，父进程应向子进程发送一个单字节命令。让我们用星号（`*`）来完成这项活动。你应该使用`sleep()`命令，以便终端上的打印不会太快。一个好的配置是`sleep(200)`。

根据上述简要说明，单独运行子程序预期的输出如下：

```java
0
1
2
3
[...]
49
50
```

但是，当从父程序调用时，结果应该是：

```java
0
1
2
[...]
36
37
0
1
[loops forever]
```

1.  子程序应该有一个类似于以下算法：

    ```java
    int cont = 0;
    while(cont <= 50) {
      System.out.println(cont++);
      sleep(200);
      if (System.in.available() > 0) {
        ch = System.in.read();
        if (ch == '*') {
        cont = 0;
        }
      }
    }
    ```

    这里有一个调用`System.in.available()`来检查子程序输出缓冲区中是否有数据。

1.  另一方面，父程序应考虑包含类似以下内容：

    ```java
    if (Integer.parseInt(line) == 37) {
      writer.write('*');
      writer.flush(); // needed because of the buffered output
    }
    ```

这将检测刚刚到达的作为`String`的数字是否将被转换为`Integer`，然后它将与我们为计数重置所建议的限制进行比较。

我们没有深入探讨`Process`类提供的所有方法。因此，建议将本章的工作包裹在传统的参考文档中，并访问 JavaDoc 以了解该类还能提供什么。

注意

该活动的解决方案可以在第 559 页找到。你可以在 Oracle 的官方文档中了解更多关于`Process`类的信息：[`docs.oracle.com/javase/8/docs/api/java/lang/Process.html`](https://www.packtpub.com/application-development/mastering-java-9)。

# 摘要

在本章简短的介绍中，你了解了 Java 中的`Process`类。你看到了一个输出到`System.out`的过程如何在父程序中被捕获。同时，你也看到了父程序如何轻松地向子程序发送数据。示例表明，不仅可以启动自己的程序，还可以启动任何其他程序，例如网页浏览器。使用包含`Process` API 的程序构建软件自动化的可能性是无限的。

我们还看到了在进程间通信方面流的重要性。基本上，你必须在上层流的基础上创建流来开发更复杂的数据结构，这将使代码运行得更快。下一章将介绍正则表达式。
