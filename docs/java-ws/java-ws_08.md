# 第八章：8. 套接字、文件和流

概述

本章将教会你如何与外部数据存储系统协同工作。在早期部分，你将学习如何列出目录的内容——这是学习使用 Java 创建、打开、读取和写入外部文件的逻辑第一步。从那里，你将研究不同的方法，缓冲和非缓冲，以及如何区分它们。然后，你将学习识别两个主要的`java.io`和`java.nio`，它们与上述方法的相应关系，以及何时何地使用它们。在本章的最终活动中，你将被要求使用所有这些 Java 技能和工具，以便在远程计算机上运行的两个不同的程序之间进行通信，为接下来的章节做准备。

# 简介

在操作系统层面，文件和目录在某种程度上是相似的。它们是代表存储中某个链接的名称，无论是你的硬盘、云中的某个地方，还是你口袋里的 USB 驱动器。然而，在概念层面上，它们本质上是不同的。文件包含信息，而目录链接到其他目录和文件。

有两个主要的`java.io`和`java.nio`。这两个 API 都可以用来导航目录和操作文件。关于文件位置的信息称为路径名。它包含文件所在硬盘上目录的完整信息，一直到文件名和扩展名。它应该具有以下形式：

```java
/folder_1/folder_2/[...]/folder_n/file.extension
```

不同的操作系统对文件和文件夹结构的称呼不同。在 Unix 系统（如 Linux 或 macOSX）中，`/`符号代表文件夹之间的分隔。在路径名开头有一个`/`表示相对于系统根文件夹的绝对定位。没有这个符号将表示相对于`classpath`或程序执行的路径的相对定位。在 Windows 计算机中，文件夹分隔符是`\`，根目录由硬盘标签确定。默认情况下，Windows 的根文件夹是`C:`，但你也可以在任何其他驱动器中存储文件，例如`D:`。

之前提到的两个 API（即`java.io`和`java.nio`）之间的主要区别在于它们读取和写入数据的方式。第一个，`java.io`，可以与流（这是我们将在本章后面探讨的概念）协同工作，并以阻塞方式从一点到另一点逐字节传输数据。第二个，`java.nio`，与缓冲区协同工作。这意味着数据以块的形式读入和写入内存的一部分（缓冲区），而不是直接从流中读取。这允许非阻塞通信，例如，允许你的代码在不需要等待所有数据发送的情况下继续做其他事情——你只需开始将信息复制到缓冲区，然后继续做其他事情。

当涉及到文件时，主要区别在于使用一种方法或另一种方法在尝试以不同的方式执行相同任务时，程序的速度会更快或更慢。我们将主要关注使用`java.nio`，因为它更容易用它来使用文件，然后偶尔会提到`java.io`。`java.nio.file` API（注意与`java.io.File`的区别）定义了 JVM 使用的类和接口——包括文件、它们的属性和文件系统——是较新的，并提供了一种更简单的方式来使用接口。然而，并非所有情况都如此，正如我们将在本章中看到的。

# 列出文件和目录

我们将探讨如何以不同的方式列出文件和目录。当检查某个文件是否存在时，这些技术可能会派上用场，这将在您尝试找到属性文件等情况下，允许您向用户提供更敏感的信息。如果您发现您正在寻找的文件不存在，同时您还注意到您不在正确的目录中，您可以让您的程序定位文件实际所在的文件夹，或者您可以简单地通知用户这种情况。

注意

在您的计算机上的任何位置列出文件和目录都有不同的技术。您必须根据具体情况明智地选择。虽然乍一看最新的 API 似乎更复杂，但正如您将在以下示例中看到的，它比之前的任何版本都要强大得多。

让我们从列出目录内容的老方法开始。在下一个练习中，我们只会使用`java.io`。它需要调用`File(dir).list()`，其中`dir`是一个表示您想要访问的文件夹名称的字符串。为了确保本书中的代码与您的操作系统兼容，我们选择检查您的操作系统的临时文件夹。Java 将其存储在 JVM 属性中，标记为`java.io.tmpdir`。因此，方法开始时的`getProperty()`调用提取了文件夹的名称。例如，对于任何 Unix 操作系统，该属性指向`/tmp`文件夹。

您的临时文件夹将被许多由您计算机中运行的不同程序创建的文件和文件夹填满。因此，我们选择只显示操作系统列出的前五个——顺序由操作系统决定。除非您对`list()`调用的结果进行排序，否则您很可能找不到输出排序的逻辑：

```java
import java.io.*;
import java.util.*;
public class Example01 {
    public static void main(String[] args) throws IOException {
        String pathString = System.getProperty("java.io.tmpdir");
        String [] fileNames = new File(pathString).list();
        for (int i = 0; i < 5; i++ ) {
            System.out.println(fileNames[i]);
        }
    }
}
```

本例的输出将如下所示：

```java
Slack Crashes
+~JF8916325484854780029.tmp
gnome-software-CAXF1Z
.XIM-unix
.X1001-lock
Process finished with exit code 0
```

注意

由于每个人的计算机内容都不同——即使在特定的文件夹中也是如此——因此，您将在本章的代码列表中看到的输出信息将与您在终端中看到的不同。

在之前的示例中，我们故意隐藏了处理每个代码块的 API 部分，以简化代码列表。如果你从代码中删除三个导入语句，并按照 IDE 的指示添加更细粒度的 API 来处理此代码，你将得到以下结果：

```java
import java.io.File;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
```

到目前为止，你在本书中几乎已经学习了所有这些 API。甚至在异常章节中，为了捕获`IOException`，也简要介绍了`java.io.File`。在接下来的示例中，我们将遵循同样的原则，只是为了尽可能缩短程序头。然而，最好还是减少代码行数。

让我们探索另一种列出目录内容的方法，但这次使用`java.nio`：

```java
import java.io.IOException;
import java.nio.file.*;
import java.util.*;
public class Example02 {
    public static void main(String[] args) throws IOException {
        String pathString = System.getProperty("java.io.tmpdir");
        List<String> fileNames = new ArrayList<>();
        DirectoryStream<Path> directoryStream;
        directoryStream = Files.newDirectoryStream(Paths.get(pathString));
        for (Path path : directoryStream) {
            fileNames.add(path.toString());
        }
        for (int i = 0; i < 5; i++ ) {
            System.out.println(fileNames.get(i));
        }
    }
}
```

如你所见，这个列表的输出与之前的示例不同：

```java
/tmp/Slack Crashes
/tmp/+~JF8916325484854780029.tmp
/tmp/gnome-software-CAXF1Z
/tmp/.XIM-unix
/tmp/.X1001-lock
Process finished with exit code 0
```

在这里，显示了目录和文件的完整路径。这与`DirectoryStream`从操作系统捕获信息的方式有关。这个例子中的`for`循环可能对你来说看起来很新。这与我们处理流的方式有关。我们还没有解释它们，而且我们将在本章的后面部分进行解释。但你可以看到它在做什么：它创建了一个缓冲区，用于存储关于不同目录的信息。然后，如果缓冲区中有数据，就可以使用`for(Path path : directoryStream)`语句遍历缓冲区。由于我们一开始不知道它的大小，我们需要一个列表来存储包含目录内容的字符串。然而，在这个阶段，我们还没有调用`java.util.stream` API，因为`DirectoryStream`属于`java.nio` API。

这里展示了另一个正确使用流的代码示例。请注意，我们没有显示其输出，因为它与之前的示例相同：

```java
import java.io.IOException;
import java.nio.file.*;
import java.util.stream.Stream;
public class Example03 {
    public static void main(String[] args) throws IOException {
        String pathString = System.getProperty("java.io.tmpdir");
        Path path = Paths.get(pathString);
        Stream<Path> fileNames = Files.list(path);
        fileNames.limit(5).forEach(System.out::println);
    }
}
```

## 将目录与文件分离

假设你希望在列出文件夹内容时将文件与目录区分开来。为了做到这一点，你可以使用`java.nio`中的一个方法，即`isDirectory()`，如下面的示例所示：

```java
Example04.java
17         for (int i = 0; i < 5; i++ ) {
18             String filePath = fileNames.get(i);
19             String fileType = Files.isDirectory(Paths.get(filePath)) ? "Dir" :                "Fil";
20            System.out.println(fileType + " " + filePath);
21         }
https://packt.live/2o43Yhe
```

我们已经突出了与之前使用 java.nio API 访问目录的示例相比的新代码部分。`Files.isDirectory()`需要一个`Paths`类的对象。`Paths.get()`将路径从目录项（作为字符串传递）转换为`Paths`类的实际实例。有了这个，`Files.isDirectory()`将返回一个布尔值，如果是目录则为`true`，如果不是则为`false`。我们使用内联`if`语句根据我们处理的是目录还是文件来分配字符串`Dir`或`Fil`。这段代码的输出结果如下：

```java
Dir /tmp/Slack Crashes
Fil /tmp/+~JF8916325484854780029.tmp
Dir /tmp/gnome-software-CAXF1Z
Dir /tmp/.XIM-unix
Fil /tmp/.X1001-lock
Process finished with exit code 0
```

如您所见，在临时目录中，既有文件也有子目录。下一个问题是如何列出子目录的内容。我们将把这个问题作为一个练习来处理，但在我们这样做之前，尝试一个更进一步的例子，该例子将只列出目录项。这是一个更高级的技术，但它将给我们一个机会，回顾并尝试使用我们到目前为止所获得的知识来实现自己的解决方案：

```java
import java.io.IOException;
import java.nio.file.*;
import java.util.*;
import java.util.stream.Collectors;
public class Example05 {
    public static void main(String[] args) throws IOException {
        String pathString = System.getProperty("user.home");
        List<Path> subDirectories = Files.walk(Paths.get(pathString), 1)
                    .filter(Files::isDirectory)
                    .collect(Collectors.toList());
        for (int i = 0; i < 5; i++ ) {
            Path filePath = subDirectories.get(i);
            String fileType = Files.isDirectory(filePath) ? "Dir" : "Fil";
            System.out.println(fileType + " " + filePath);
        }
    }
}
```

首先，为了展示使用其他环境变量的可能性（这就是我们所说的系统属性，它是为您的操作系统定义的），我们将文件夹更改为用户主目录，这对应于您的用户空间，或者您通常存储文件的目录。请从现在开始小心，以避免任何与您的文件相关的意外。

`Files.walk()` 将提取到一定深度的目录结构，在我们的例子中，是深度一。深度表示您的代码将挖掘多少层子目录。`filter(Files::isDirectory)` 将排除任何不是目录的东西。我们还没有看到过滤器，但这个概念足够清晰，不需要进一步解释。调用最后的部分，`collect(Collectors.toList())`，将创建一个输出列表。这意味着 `subDirectories` 对象将包含目录路径的列表。这就是为什么在这个例子中，与上一个例子不同，我们不需要调用 `Paths.get(filePath)`。该调用的输出将取决于您的操作系统以及您主文件夹中的内容。在我的计算机上，它运行的是 Linux 版本，结果如下：

```java
Dir /home/<userName>
Dir /home/<userName>/.gnome
Dir /home/<userName>/Vídeos
Dir /home/<userName>/.shutter
Dir /home/<userName>/opt
Process finished with exit code 0
```

在这里，`<userName>` 对应于计算机上用户的昵称。如您所见，这仅表示在 `pathString` 初始化的目录的内容。问题是，我们能否在我们的程序中表示嵌套子目录的内容到初始的 `pathString`？

## 练习 1：列出子目录的内容

让我们利用到目前为止所获得的知识，编写一个程序来导航子目录。这可能不是解决这个挑战的最佳方式，但它会有效：

1.  让我们从最新的例子开始，我们使用 `Files.walk()` 调用，深度为 1，并使用过滤器来列出特定目录 `pathString` 的内容——只是目录。目录搜索中的深度决定了我们的程序将导航到多少层子目录。级别 1 与搜索发起的同一级别相同。级别 2 表示我们还应该表示主目录内部的目录内容。原则上，这应该像给调用一个更高的深度值一样简单，如下所示：

    ```java
    List<Path> subDirectories = Files.walk(Paths.get(pathString), 2)
                    .filter(Files::isDirectory)
                    .collect(Collectors.toList());
    ```

1.  但有一个问题。当运行这样的调用时，很可能存在程序不允许访问的目录或文件。将触发一个关于权限的异常，并且程序将停止：

    ```java
    Exception in thread "main" java.io.UncheckedIOException: java.nio.file.AccessDeniedException: /home/<userName>/.gvfs
          at java.nio.file.FileTreeIterator.fetchNextIfNeeded(FileTreeIterator.java:88)
          at java.nio.file.FileTreeIterator.hasNext(FileTreeIterator.java:104)
    [...]
          at java.util.stream.ReferencePipeline.collect(ReferencePipeline.java:499)
          at Example04.main(Example04.java:13)
    Caused by: java.nio.file.AccessDeniedException: /home/<userName>/.gvfs
          at sun.nio.fs.UnixException.translateToIOException(UnixException.java:84)
          at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:102)
    [...]
          at java.nio.file.FileTreeIterator.fetchNextIfNeeded(FileTreeIterator.java:84)
          ... 9 more
    Process finished with exit code 1
    ```

1.  访问任何包含在这些子目录中的目录或文件，这些子目录处于严格的行政用户权限之下，将导致此程序崩溃。捕获这个异常没有任何用处，因为结果仍然是一个无法使用的目录列表。有一种相当高级的技术可以使它工作，但你还没有被介绍到完成这个任务所需知道的一切。相反，让我们专注于你已经获得的工具，以便创建自己的方法来深入子目录并提取其内容。

1.  让我们回到*示例 03*并修改它，使其仅显示 user.home 目录内的目录：

    ```java
    String pathString = System.getProperty("user.home");
    Path path = Paths.get(pathString);
    Stream<Path> fileNames = Files.list(path).filter(Files::isDirectory);
    fileNames.limit(5).forEach(System.out::println);
    ```

1.  如您所见，我们应用了之前看到的`filter()`方法。我们也可以实现使用`isDirectory()`的替代方案，就像我们在*示例 04*中看到的那样，但这更简洁，简洁是关键。

1.  基于这样的想法，即`list()`可以给出任何文件夹的内容，让我们再次为每个文件名调用它。这意味着我们将不得不修改我们正在使用的`forEach()`语句，以便我们可以访问嵌套目录的第二层：

    ```java
    fileNames.limit(5).forEach( (item) -> {
        System.out.println(item.toString());
        try {
            Stream<Path> fileNames2 = Files.list(item).filter(Files::isDirectory);
            fileNames2.forEach(System.out::println);
        } catch (IOException ioe) {}
    });
    ```

1.  如您所见，高亮显示的代码是我们之前代码的重复，只是对象名称改为`fileNames2`。这次，我们移除了限制，这意味着它将打印出每个目录拥有的任何子目录的输出。真正的创新之处在于我们是如何从仅调用`System.out::print`转变为编写更复杂的代码，首先打印出我们所在的路径，然后打印该路径的子文件夹路径。我们在这里期待的是一种称为 lambda 表达式的功能。它们将在后面的章节中解释。然而，这里的代码足够简单，您可以理解。对于`fileNames`缓冲区中的每个`(item)`，我们将执行上述提到的操作。结果看起来像这样：

    ```java
    /home/<userName>/.gnome
    /home/<userName>/.gnome/apps
    /home/<userName>/Vídeos
    /home/<userName>/Vídeos/technofeminism
    /home/<userName>/Vídeos/Webcam
    /home/<userName>/Vídeos/thumbnail
    /home/<userName>/.shutter
    /home/<userName>/.shutter/profiles
    /home/<userName>/opt
    /home/<userName>/opt/Python-3.4.4
    /home/<userName>/.local
    /home/<userName>/.local/share
    /home/<userName>/.local/bin
    /home/<userName>/.local/lib
    Process finished with exit code 0
    ```

1.  此外，必须在生成列表时捕获`IOException`，否则代码将无法编译。在`main`方法的声明中`throw IOException`不适用于`forEach()`表达式，因为它在程序的作用域中更深一层。在这种情况下，我们正在查看方法的内联定义。但问题是，我们如何绕过目录探索任意深度的想法？

1.  在深入`java.nio` API 中，我们发现`walkFileTree()`方法，它可以浏览目录结构，直到达到一定的深度——在下面的示例中是两层——并提供覆盖其部分方法的可能性，以决定到达目录项并尝试访问时会发生什么。对这个方法的调用可能看起来像这样：

    ```java
    Path path = Paths.get(System.getProperty("user.home"));
    Files.walkFileTree(path, Collections.emptySet(), 2, new SimpleFileVisitor<Path>() {
        @Override
        public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes       attrs) {
            System.out.println(dir.toString());
            return FileVisitResult.CONTINUE;
        }
    });
    ```

1.  在这里，你可以看到当尝试在文件夹中打开一个目录项时，`preVisitDirectory()` 方法是如何被调用的。包含该行的程序将一直运行，直到例如，出现与权限相关的异常。如果没有异常情况，重写的方法将打印出所有深度达两级的目录名称。在我们实验的主目录的情况下，我们知道有一个文件夹，Java 的默认用户权限不足以让我们的程序访问。因此，如果我们运行这个程序，我们会看到它正常操作直到遇到异常：

    ```java
    /home/<userName>/.gnome/apps
    /home/<userName>/Vídeos/technofeminism
    /home/<userName>/Vídeos/Webcam
    [...]
    /home/<userName>/.local/lib
    Exception in thread "main" java.nio.file.AccessDeniedException: /home/<userName>/.gvfs
          at sun.nio.fs.UnixException.translateToIOException(UnixException.        java:84)
          at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.        java:102)
          at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.        java:107)
          at sun.nio.fs.UnixFileSystemProvider.        newDirectoryStream(UnixFileSystemProvider.java:427)
          at java.nio.file.Files.newDirectoryStream(Files.java:457)
          at java.nio.file.FileTreeWalker.visit(FileTreeWalker.java:300)
          at java.nio.file.FileTreeWalker.next(FileTreeWalker.java:372)
          at java.nio.file.Files.walkFileTree(Files.java:2706)
          at Exercise01.main(Exercise01.java:11)
    Process finished with exit code 1
    ```

1.  `preVisitDirectory()` 方法将告诉 `walkFileTree` 方法它应该继续通过其返回值工作。这里的问题是，由于 `AccessDeniedException`，我们的程序将不会进入 `preVisitDirectory()`。我们需要重写另一个名为 `visitFileFailed()` 的方法来了解如何处理尝试访问目录项时发生的任何类型的异常：

    ```java
    @Override
    public FileVisitResult visitFileFailed(Path file, IOException exc)
            throws IOException {
        System.out.println("visitFileFailed: " + file);
        return FileVisitResult.CONTINUE;
    }
    ```

    这将产生预期的结果，如下所示：

    ```java
    /home/<userName>/.gnome/apps
    /home/<userName>/Vídeos/technofeminism
    [...]
    /home/<userName>/.local/lib
    visitFileFailed: /home/<userName>/.gvfs
    /home/<userName>/.config/Atom
    [...]
    /home/<userName>/drive_c/Program Files
    /home/<userName>/drive_c/Program Files (x86)
    /home/<userName>/drive_c/users
    /home/<userName>/drive_c/windows
    /home/<userName>/.swt/lib
    Process finished with exit code 0
    ```

    从这个过程我们可以得出结论，尽管有多种方法可以执行相同的任务，但那些解决方案的实现方式将使我们能够有所控制。在这种情况下，`walk()` 方法不足以让我们轻松处理异常，因此我们必须探索一个替代方案，最终发现这个方案更容易理解。

    作为参考，这个练习的最终代码应该如下所示：

```java
Exercise01.java
1  import java.io.IOException;
2  import java.nio.file.*;
3  import java.nio.file.attribute.BasicFileAttributes;
4  import java.util.Collections;
5  
6  public class Exercise01 {
7      public static void main(String[] args) throws IOException {
8          Path path = Paths.get(System.getProperty("user.home"));
9  
10         Files.walkFileTree(path, Collections.emptySet(), 2, new          SimpleFileVisitor<Path>() {
11 
12              @Override
13              public FileVisitResult preVisitDirectory(Path dir,                 BasicFileAttributes attrs) {
14                 System.out.println(dir.toString());
15                 return FileVisitResult.CONTINUE;
16             }
https://packt.live/35MN9Zd
```

# 创建和写入文件

一旦我们熟悉了如何列出目录的内容，下一步合乎逻辑的步骤就是继续创建文件和文件夹。让我们先通过使用 `java.nio` 创建并写入文件。使用此 API 创建文件的最简单方法需要调用以下代码：

```java
Files.createFile(newFilePath);
```

同时，创建一个目录就像这样简单：

```java
Files.createDirectories(newDirPath);
```

作为一种良好的实践，你应该在创建具有相同名称的目录和/或文件之前检查它们是否存在。有一个简单的方法可以检查 Path 类的任何对象是否可以在程序正在探索的文件夹中找到：

```java
Files.exists(path);
```

让我们把所有这些放在一起，通过一个示例来创建一个文件夹，然后在文件夹内创建一个文件：

```java
Example06.java
1  import java.io.IOException;
2  import java.nio.file.Files;
3  import java.nio.file.Path;
4  import java.nio.file.Paths;
5  
6  public class Example06 {
7      public static void main(String[] args) {
8          String pathString = System.getProperty("user.home") + "/javaTemp/";
9          Path pathDirectory = Paths.get(pathString);
10         if(Files.exists(pathDirectory)) {
11             System.out.println("WARNING: directory exists already at: " +                  pathString);
12         } else {
13             try {
14                 // Create the directory
15                 Files.createDirectories(pathDirectory);
16                 System.out.println("New directory created at: " + pathString);
17             } catch (IOException ioe) {
18                 System.out.println("Could not create the directory");
19                 System.out.println("EXCEPTION: " + ioe.getMessage());
20             }
21         }
https://packt.live/2MSEPhX
```

第一次执行此代码列表的结果应该如下所示：

```java
New directory created at: /home/<userName>/javaTemp/
New file created at: /home/<userName>/javaTemp/temp.txt
Process finished with exit code 0
```

任何后续执行都应该给出以下结果：

```java
WARNING: directory exists already at: /home/<userName>/javaTemp/
WARNING: file exists already at: /home/<userName>/javaTemp/temp.txt
Process finished with exit code 0
```

这创建了一个本质上为空的文件。使用终端，你可以通过调用 `ls -lah ~/javaTemp/temp.txt` 命令来列出文件的大小，这将返回如下结果：

```java
-rw-r--r--  1 userName dialout   0 maj 15 13:57 /[...]/temp.txt
```

这意味着文件在硬盘上不占用任何字节的存储空间。这意味着文件存在，但它为空。使用`java.nio.file.Files` API 中的`write()`方法轻松地将文本写入文件：`write()`。唯一的问题是传递给此方法的参数不是显而易见的。在其最简单的接口中，你必须传递两个参数：`Path`对象和一个包含文本的`List`。除此之外，还存在着文件可能不存在的风险，这需要处理经典的`IOException`。它可能看起来像这样：

```java
try {
    Files.write(pathFile, Arrays.asList("hola"));
    System.out.println("Text added to the file: " + pathFile);
} catch (IOException ioe) {
    System.out.println("EXCEPTION: " + ioe.getMessage());
}
```

注意

当调用`write()`向文件写入文本时，你不需要在字符串的末尾添加换行符。它将自动由方法添加，就像使用`println()`等命令时预期的那样。

一旦你将最后一个代码片段添加到最新的例子中，程序将给出以下结果：

```java
WARNING: directory exists already at: /home/<userName>/javaTemp/
WARNING: file exists already at: /home/<userName>/javaTemp/temp.txt
Text added to the file: /home/<userName>/javaTemp/temp.txt
Process finished with exit code 0
```

之前的例子只是将文本写入文件，同时也删除了之前的内容。如果你想追加文本而不是覆盖，你需要修改对写入命令的调用：

```java
Files.write(pathFile, Arrays.asList("hola"), StandardOpenOption.APPEND);
```

调用中高亮的部分负责确定在文件末尾添加什么文本，而不是删除所有内容并从头开始写入。以下示例简单地追加文本到一个现有文件：

```java
Example07.java
8  public class Example07 {
9      public static void main(String[] args) {
10         String pathString = System.getProperty("user.home") +              "/javaTemp/temp.txt";
11         Path pathFile = Paths.get(pathString);
12         String text = "Hola,\nme da un refresco,\npor favor?";
13 
14         if(Files.exists(pathFile))
15             try {
16                 Files.write(pathFile, Arrays.asList(text),                      StandardOpenOption.APPEND);
17                 System.out.println("Text added to the file: " + pathFile);
18             } catch (IOException ioe) {
19                 System.out.println("EXCEPTION: " + ioe.getMessage());
20             }
21     }
https://packt.live/2MrBV4B
```

这个程序将整个句子追加到了示例文本文件中。文件最终的读取内容如下：

```java
hola
Hola,
me da un refresco,
por favor?
```

这是在西班牙语中点一杯苏打水的说法。在下一节中，我们将探讨如何读取我们刚刚创建的文件。

## 活动一：将目录结构写入文件

这个活动的目标是编写一个应用程序，该程序将读取目录结构，从存储在变量中的目录开始。结果将写入一个文本文件，这样，对于每个嵌套级别，你将包括一个制表符或四个空格来从其父目录中视觉上缩进嵌套文件夹。此外，你还需要只显示文件夹的名称，而不是其完整路径。换句话说，文件的内容应该对应以下结构：

```java
Directory structure for folder: /folderA/folderB/.../folderN
folderN
    folderN1
        folderN11
        folderN12
        ...
    folderN2
        folderN21
        folderN22
        ...
    folderN3
        folderN31
        folderN32
        ...
    ...
    folderNN
```

1.  你将要创建的程序需要将目录的深度作为参数，但我们建议你不要过于深入——最多 10 层是合适的：

    ```java
    Files.walkFileTree(path, Collections.emptySet(), 10, new SimpleFileVisitor<Path>() ...
    ```

1.  当处理获取到的目录路径时，你需要使用/符号作为分隔符来分割结果字符串，然后取最后一个项目。此外，你还需要根据深度打印缩进的数量，这需要有一些代码可以估计给定初始路径的当前深度。解决这些问题的技巧可能是使`preVisitDirectory()`的内容如下：

    ```java
    // get the path to the init directory
    String [] pathArray = path.toString().split("/");
    int depthInit = pathArray.length;
    // get the path to the current folder
    String [] fileArray = dir.toString().split("/");
    int depthCurrent = fileArray.length;
    // write the indents
    for (int i = depthInit; i < depthCurrent; i++) {
        System.out.print("    ");
        // HINT: copy to list or write to file here
    }
    // write the directory name
    System.out.println(fileArray[fileArray.length – 1]);
    // HINT: copy to list or write to file here
    ```

    注意

    这个活动的解决方案可以在第 552 页找到。

# 读取现有文件

以简单的方式读取文件。问题是关于你将数据存储在哪里。我们将使用列表，遍历列表，然后将结果打印到 `System.out`。下一个示例使用 `readAllLines()` 打开现有文件，并将内容读取到计算机内存中，放入 `fileContent` 列表中。之后，我们使用迭代器遍历每一行并将它们发送到终端：

```java
import java.io.IOException;
import java.nio.file.*;
import java.util.List;
public class Example08 {
    public static void main(String[] args) {
        String pathString = System.getProperty("user.home") + "/javaTemp/temp.txt";
        Path pathFile = Paths.get(pathString);
        try {
            List<String> fileContent = Files.readAllLines(pathFile);
            // this will go through the buffer containing the whole file
            // and print it line by one to System.out
            for (String content:fileContent){
                System.out.println(content);
            }
        } catch (IOException ioe) {
            System.out.println("WARNING: there was an issue with the file");
        }
    }
}
```

`temp.txt` 文件是我们之前保存消息的地方；因此，结果如下：

```java
hola
Hola,
me da un refresco,
por favor?
Process finished with exit code 0
```

如果文件不存在（你可能在之前的练习中删除了它），你将得到以下结果：

```java
WARNING: there was an issue with the file
Process finished with exit code 0
```

另一种达到相同结果但避免使用列表并使用流的方法如下：

```java
import java.io.IOException;
import java.nio.file.*;
public class Example09 {
    public static void main(String[] args) {
        String pathString = System.getProperty("user.home") + "/javaTemp/temp.txt";
        Path pathFile = Paths.get(pathString);
        try {
            Files.lines(pathFile).forEach(System.out::println);
        } catch (IOException ioe) {
            System.out.println("WARNING: there was an issue with the file");
        }
    }
}
```

# 读取属性文件

属性文件以标准格式存储键值对（也称为键映射）。此类文件的示例内容如下：

```java
#user information
name=Ramiro
familyName=Rodriguez
userName=ramiroz
age=37
bgColor=#000000
```

这是一个虚构的用户属性文件的示例。注意，注释是用井号符号标记的。你将使用属性文件来存储应用程序的可配置参数，甚至用于本地化字符串。

让我们尝试读取一个属性文件。你可以在本章前面创建的用户空间中的同一个临时文件夹中创建一个文本文件。命名为 `user.properties`，并将前面示例的内容写入其中。这遵循了一个使用 `java.io` 读取并打印属性文件内容的程序示例。鉴于 Java 的工作方式，没有比使用 `java.nio` 更好的替代方案来完成这项任务。

注意

阅读属性文件的内容并不仅限于获取文件的每一行，还包括解析键值对并能够从中提取数据。

你首先会注意到，读取属性文件需要以流的形式打开一个文件——再次强调，这是我们将在本章后面探讨的概念——使用 `FileInputStream`。从那里开始，`Properties` 类包含一个名为 `load()` 的方法，可以从数据流中提取键值对。为了清理代码列表，我们将代码的加载和打印方面与处理文件打开的部分分开。此外，我们确保所有异常都在主类中处理，以便有一个单一的管理点，这使得代码更易于阅读。

```java
Example10.java
17     public static void main(String[] args) throws IOException {
18         String pathString = System.getProperty("user.home") +              "/javaTemp/user.properties";
19 
20         FileInputStream fileStream = null;
21         try {
22             fileStream = new FileInputStream(pathString);
23             PrintOutProperties(fileStream);
24         } catch (FileNotFoundException fnfe) {
25             System.out.println("WARNING: could not find the properties file");
26         } catch (IOException ioe) {
27             System.out.println("WARNING: problem processing the properties                  file");
28         } finally {
29             if (fileStream != null) {
30                 fileStream.close();
31             }
32         }
33     }
https://packt.live/2Bry4OK
```

在本章中，我们还没有讨论的一个方面是，一旦你完成与流的工作，就必须关闭流。这意味着关闭后它们将无法用于进一步的数据处理。这一步骤对于避免运行时任何类型的 JVM 内存问题非常重要。因此，示例代码在加载属性文件后调用 `fileStream.close()`。如果你记得 *第五章* 中的 *良好实践* 部分，*异常* 部分提到你应该在 `finally` 语句中关闭流。这也是为什么这个程序必须在主方法中抛出 `IOException` 的原因。如果你想要以干净的方式处理这个问题（通过避免嵌套 try-catch 语句或在主方法中使用 `throws IOException`），你可以将整个 `try` 块包裹在一个方法中，然后从主方法中调用该方法，在那里你可以捕获 `IOException`。查看即将到来的练习，看看这是如何完成的。

上述示例的输出如下：

```java
name: Ramiro
family name: Rodriguez
nick: ramiroz
age: 37
background color: #000000
Process finished with exit code 0
```

`Properties` 类中包含一些有趣的方法供你探索。例如，`properties.keys()` 将返回文件中所有键的枚举，在我们的例子中是 name、familyName、userName 等等。这个特定方法由于与 `Hashtable` 类的关系而被 `Properties` 类继承。建议你阅读该类的 API 文档，以发现你可以使用的其他有趣方法。

当涉及到属性文件的位置时，它们可以存储在类路径中，有时甚至存储在实际的 JAR 文件中，这为带有属性文件的应用程序的紧凑式分发提供了一种方式。

接下来要探索的另一个方面是如何以编程方式创建自己的属性文件。让我们通过一个逐步练习来探讨这个主题。

## 练习 2：从 CLI 创建属性文件

在这个练习中，你将制作一个能够从 CLI 输入创建属性文件（或修改现有文件）的应用程序。你将通过将属性文件的名称和键值对作为参数传递给程序来实现这一点。这将使你能够轻松地创建任何类型的属性文件。应用程序预期调用的示例如下：

```java
usr@localhost:~/[...]/Exercise02$ java Exercise02 myProperties.properties name=Petra
```

这样一个程序的操作过程很简单。首先，你需要检查文件是否存在。如果存在，则加载属性。然后，添加新的属性或使用作为参数传递的数据修改现有的属性。稍后，将信息写入文件，并向用户反馈最终发送到文件的内容。这样，用户将能够看到他们所做的修改正在生效，而无需打开文件。

让我们一步步看看如何制作这样的程序：

1.  打开 IntelliJ 并创建一个名为 `Exercise02` 的新 Java CLI 项目。

1.  首先，我们需要检查在 CLI 中定义的属性文件是否已经存在。我们将要实现的程序将检查文件是否存在。如果是这样，它将打开它并加载现有的属性。CLI 中的其余参数将用于修改现有的键值对或添加新的键值对。为了查看属性文件是否存在并加载它，我们需要执行以下操作：

    ```java
    if (Files.exists(pathFile)) {
        properties = LoadProperties(pathString);
    }
    ```

1.  加载属性是通过重用*示例 10*中的代码来完成的，但将其包装在我们在上一步中调用的`LoadProperties()`方法中。让我们实现它以返回`Properties`类的对象（注意我们为了确保在可能发生的异常后关闭流而实现的`finally`语句。我们必须将流初始化为 null）：

    ```java
    public static Properties LoadProperties (String pathString)   throws IOException {
        Properties properties = new Properties();
        FileInputStream fileInputStream = null;
        try {
            fileInputStream = new FileInputStream(pathString);
            properties.load(fileInputStream);
        } catch (FileNotFoundException fnfe) {
            System.out.println("WARNING: could not find the properties file");
        } catch (IOException ioe) {
            System.out.println("WARNING: problem processing the properties           file");
        } finally {
            if (fileInputStream != null) {
                fileInputStream.close();
            }
        }
        return properties;
    }
    ```

1.  如果文件不存在，在稍后调用`store()`方法时将创建它——在这个阶段不需要创建一个空文件。

1.  接下来，我们需要从`arg[]`数组中读取 CLI 上的剩余参数，并将它们逐个推入属性对象中。属性对象从`Hashtable`类继承其行为，该类处理键值对。将使用`setProperty()`方法来修改现有属性或写入新属性。由于参数以 key=value 的字符串格式表达，我们可以使用`split()`来分隔需要传递给`setProperty()`的参数：

    ```java
    for (int i = 1; i < args.length; i++) {
        String [] keyValue = args[i].split("=");
        properties.setProperty(keyValue[0], keyValue[1]);
    }
    ```

1.  我们将要写入文件，但不是使用输入数据的流，而是使用输出数据的流。它的名字很容易推断，`FileOutputStream`。该类的变量声明如下：

    ```java
    FileOutputStream fileOutputStream = new FileOutputStream(pathString);
    ```

1.  要向属性文件添加一些注释，我们只需向`store()`方法添加一个参数。在这种情况下，只是为了添加一些上下文信息，让我们通过调用以下操作添加时间戳：

    ```java
    java.time.LocalDate.now()
    ```

1.  我们调用`store()`方法，该方法将属性发送到文件中。我们将覆盖之前文件中的任何内容。这个调用使用输出`Stream`和我们所选择的任何注释作为参数：

    ```java
    properties.store(fileOutputStream, "# modified on: " + java.time.LocalDate.now());
    ```

1.  为了提高程序的可用性，创建一个方法来遍历整个属性集并打印出来。这样，用户就可以看到他们是否正确地写下了内容：

    ```java
    public static void PrintOutProperties(Properties properties) {
        Enumeration keys = properties.keys();
        for (int i = 0; i < properties.size(); i++) {
            String key = keys.nextElement().toString();
            System.out.println( key + ": " + properties.getProperty(key) );
        }
    }
    ```

1.  使用 CLI 中的以下调用运行代码，例如。在这种情况下，我们故意修改了本章中一直在工作的文件。程序将打印出修改后的集合。请注意，键值对没有明确的顺序：

    ```java
    [...]/Exercise02$ java Exercise02 user.properties name=Pedro
    age: 37
    familyName: Rodriguez
    name: Pedro
    bgColor: #000000
    userName: ramiroz
    ```

1.  在文本编辑器中打开生成的文件，查看您的更改是否生效。同时请注意，注释以及`store()`方法添加的`\`符号，以避免将颜色参数（使用井号符号以十六进制格式表达）误认为是注释。

1.  现在，你可以考虑对程序进行其他修改，使其能够清除现有文件、追加多个文件等。你可以使用不同的命令作为参数来完成这项工作。完整的练习代码可在 GitHub 上找到：[`packt.live/2JjUHZL`](https://packt.live/2JjUHZL)

## 什么是流？

Java 中的流是字节序列，最终通过扩展也可以是对象。你可以将流理解为两个地方之间的数据流动。创建流类型的变量就像打开一个窥视孔，查看两个容器之间运送水的管道，并看到水通过。我们试图表达的是，流内部的数据始终在变化。

正如我们之前所看到的，在本章中，我们有两种不同的方式来看待事物：一种是通过`java.io` API 的视角，另一种是通过`java.nio` API 的视角。虽然后者在更抽象的层面上工作，因此更容易理解，但前者非常强大且底层。继续使用水的类比，`java.io`将允许你看到水滴，而`java.nio`则只允许你一次玩一个 1 升瓶子的水。每个都有其优点。

`java.io`中的流可以细化到字节级别。例如，如果我们查看来自计算机麦克风输入的音频数据流，我们会看到代表声音的不同字节，一个接一个。另一个 API，`java.nio`是面向缓冲区的，而不是面向流的。虽然这是真的，但有一种在`java.nio`中处理流的方法。由于其简单性，在本节中，我们将看到一个与`java.nio`相关的示例，而在下一节中，我们将使用最适合处理它们的 API 来处理流：`java.io`。

`java.nio`中的流是对象的序列（不是任意无序的数据）。由于这些对象属于特定的类，流提供了直接将对象对应的方法应用于流的可能性。将方法应用于流的结果是另一个流，这意味着方法可以被管道化。

在本章中，我们已经看到了不同的流，主要是因为流在 Java 中扮演着如此重要的角色，以至于几乎不可能在不使用它们的情况下进行任何类型的文件相关示例。现在，你将更深入地了解它们是如何工作的。这将帮助你理解一些可能之前并不那么清晰的方面。

流的本质通常很难一开始就理解。正如之前提到的，它们不是普通的数据结构。信息以对象的形式排列。输入来自`Arrays`、程序中的 I/O 通道或`Collections`。我们可以在流上执行的操作类型如下：

+   `map`（中间操作）：这将允许你将对象映射到一个你可以作为参数提供的谓词。

+   `filter`（中间操作）：这个操作用于从整个流中排除某些元素。

+   `sorted`（中间操作）：这将排序流。

+   `collect`（终端操作）：这将把不同操作的结果放入一个对象形式，例如，一个列表。

+   `forEach`（终端操作）：这将遍历流中的所有对象。

+   `reduce`（终端操作）：这个操作对流进行操作以得到一个单一值。

我们已经用中间或终端来标记每个操作。前者意味着将要执行的操作将产生另一个流作为结果，因此可以在其后链式调用另一个操作。后者意味着在该操作完成后，不能执行进一步的操作。

到目前为止，你已经在这个章节中看到了一些这些操作的实际应用。你可以回到那些操作出现的地方，重新审视它们。这将使 `filter()`、`collect()` 和 `forEach()` 的作用更加清晰。让我们看看其他三个操作的实际应用：

```java
import java.io.IOException;
import java.nio.file.*;
import java.util.*;
public class Example11 {
    public static void main(String[] args) {
        String pathString = System.getProperty("user.home") + "/javaTemp/numbers.txt";
        Path pathFile = Paths.get(pathString);
        // if the numbers file doesn't exist, create a file with 10 random numbers
        // between 0 and 10, so that we can make something with them
        if (Files.notExists(pathFile)) {
            int [] numbers = new int[10];
            for (int i = 0; i < 10; i++) {
                numbers[i] = (int) (Math.random() * 10);
            }
```

`Example11.java` 的完整代码可以在 `Chapter 1/Code.java` 中找到。

这个例子分为两部分。程序的前半部分检查我们一直在使用的 `javaTemp` 文件夹中是否存在名为 `numbers.txt` 的文件。如果这个文件不存在，程序将使用 `Files.createFile(pathFile)` 创建它，然后用之前存储在名为 `numbers` 的 `int` 数组中的 10 个随机数字填充它。调用 `Files.write(pathFile, Arrays.asList("" + n), StandardOpenOption.APPEND)` 负责将数组中的每个数字作为单独的行添加到文件中。生成的文件将如下所示：

```java
<contents of javaTemp/numbers.txt>
5
3
1
3
6
2
6
2
7
8
```

每行一个数字的想法是，我们可以然后以列表的形式读取文件，将列表转换成流，然后开始进行不同的操作。最简单的操作是调用 `fileContent.forEach(System.out::print)`，这将把原始数据作为输出打印出来：

```java
Raw data
5313626278
```

在应用其他操作，例如 `sorted()` 之前，我们需要将数据转换成一个流，这可以通过 `stream()` 方法实现。以下是具体操作：

```java
fileContent.stream().sorted().forEach(System.out::print) 
```

这个操作的结果将会被排序。相等的值将并排显示，重复：

```java
Sorted data
1223356678
```

使用 `map()`，我们将能够处理数据并对它执行不同的操作。例如，在这里，我们将它乘以 2 并打印到终端：

```java
fileContent.stream().map( x -> Integer.parseInt(x)*2).forEach(System.out::print):
```

结果如下：

```java
Mapped data
106261241241416
```

最后，有不同类型的终止操作可以使用。为此，我们将使用直到更后面的章节才会介绍到的 lambda 表达式。然而，以下内容足够简单，不需要进一步解释。为了计算所有数字的总和，我们需要执行以下操作：

```java
System.out.println(
                    fileContent
                            .stream()
                            .map(x -> Integer.parseInt(x))
                            .reduce(Integer::sum));
```

以下为结果：

```java
Sum of data
Optional[43]
```

注意，在读取文件时，我们将其读取为`String`的`List`，因此数字被存储为字符串。这意味着，为了将它们作为数字操作，我们需要将它们转换回整数，这通过调用`Integer.parseInt(x)`来完成。

## Java 语言的流的不同类型

要讨论流类型，我们需要退一步，从`java.nio`转向`java.io`。这个 API 是支持流最好的 API。根据情况，流可以进入程序或从程序中出来。这为我们提供了两个主要的流接口：`InputStream`和`OutputStream`。

在这两个主要类别中，有四种从它们处理的数据类型的角度来看待流的方法：`File`、`ByteArray`、`Filter`或`Object`。换句话说，有一个`FileInputStream`类、一个`FileOutputStream`类、一个`ByteArrayInputStream`类等等。

根据 Javadocs，重要的是要理解流有一个层次结构。所有流都是建立在字节流之上的。但我们应尽可能使用与我们所使用的数据类型在层次结构中最接近的流类型。例如，如果我们需要处理来自互联网的一系列图像，我们应该避免在低级别使用字节流来存储图像，而应使用对象流。

注意

在官方 Java 文档中了解更多关于流的信息，请访问[`docs.oracle.com/javase/tutorial/essential/io/bytestreams.html`](https://docs.oracle.com/javase/tutorial/essential/io/bytestreams.html)。

那么如何使用 java.io 和`FileInputStream`打开并打印文件呢？我们在处理属性文件时已经看到了一些这方面的内容。让我们来看一个最低级别的例子，它将读取一个文件并逐字节打印其内容：

```java
import java.io.FileInputStream;
import java.io.IOException;
public class Example12 {
    public static void main(String[] args) throws IOException {
        FileInputStream inStream = null;
        try {
            inStream = new FileInputStream(
                     System.getProperty("user.home") + "/javaTemp/temp.txt");
            int c;
            while ((c = inStream.read()) != -1) {
                System.out.print(c);
            }
        } finally {
            if (inStream != null) {
                inStream.close();
            }
        }
    }
}
```

此示例打开我们在本章中创建的 temp.txt 文件，并打印其内容。请记住，它包含了一些简单的文本，如`hola\nHola,\nme da un ...`。当查看终端时，你将读到如下内容：

```java
1041111089710721111089744101091013210097321171103211410110211410111599111441011211111432102971181111146310
Process finished with exit code 0
```

你可能会想知道——文本怎么了？正如你所知，英语字母表中的每个符号都由一个称为 ASCII 的标准来表示。这个标准用数字表示每个符号。它区分了大写和小写，不同的符号，如感叹号或井号，数字等等。以下是一个表示小写符号的 ASCII 表的摘录：

```java
97    a    107    k    117    u
98    b    108    l    118    v
99    c    109    m    119    w
100    d    110    n    120    x
101    e    111    o    121    y
102    f    112    p    122    z
103    g    113    q
104    h    114    r
105    I    115    s
106    j    116    t
```

如果你开始使用你得到的数字流，并使用 ASCII 符号表进行解析，你会发现`104`对应于`h`，`111`对应于`o`，`108`对应于`l`，`97`对应于`a`。如果你有一个完整的 ASCII 表（包括大写字母、符号和数字），你将能够解码整个消息。我们确实得到了文件的内容，但我们没有在程序中解释我们得到的数据，这使得输出不可读。这就是为什么你应该尝试使用更高层次的流，这样你就不必在如此低级别解码信息，对于字符——就像这个例子一样——这不是什么大问题。但软件实体之间的数据传输可以很快变得复杂。

让我们考察另一种执行相同操作（打开文件）的方法，但使用不同类型的流。在这种情况下，我们将在`FileInputStream`之上使用`FileReader`，这是一种不同类型的流。为了以字符的形式获取流并将其传递给`BufferedReader`，这是一个可以读取文本完整行的流类。由于我们知道我们的文件包含按行排列的文本，这可能是以整洁方式查看文件内容的最优方法：

```java
Example13.java
5  public class Example13 {
6      public static void main(String[] args) throws IOException {
7          BufferedReader inStream = null;
8  
9          try {
10             FileReader fileReader = new FileReader(
11                     System.getProperty("user.home") + "/javaTemp/temp.txt");
12             inStream = new BufferedReader(fileReader);
13             String line;
14             while ((line = inStream.readLine()) != null) {
15                 System.out.println(line);
16             }
https://packt.live/2BsKIgh
```

这个示例的输出将是我们最初期望看到的结果：

```java
hola
Hola,
me da un refresco,
por favor?
Process finished with exit code 0
```

简而言之，信息是相同的，但关键在于我们如何看待它。使用流家族中的高级类将为我们提供更好的方法来以不同但更实用的方式处理相同的信息。

还有另一个我们尚未介绍的概念，那就是缓冲流和非缓冲流之间的区别。在用 java.io 进行低级工作时，你很可能会以非缓冲的方式工作。这意味着你将从代码中直接与操作系统交互。这些交换计算密集，尤其是在与在 JVM 内部加载任何信息到缓冲区并直接在那里操作相比（这并不意味着它不会直接访问操作系统——它会，但它将优化其使用）。

这个示例明显使用了`BufferedReader`，这与前一个示例不同。我们在本章前面提到了`java.nio`如何与缓冲区一起工作——这意味着，与`java.io`不同，它不提供直接调用操作系统的可能性。从某种意义上说，这更好，因为它更不容易出错。如果你有一个构建良好的 API，其中包含了执行你想要执行的所有操作所需的所有方法，你应该避免使用其他不太理想的工具。

## 什么是套接字？

套接字是两个在网络上运行的程序之间的双向通信通道的端点。这就像一条虚拟电缆连接了这两个程序，提供了来回发送数据的能力。Java 的 API 有类可以轻松构建通信两端的程序。例如，互联网上的交换发生在 TCP/IP 网络上，我们区分参与通信的角色。有服务器和客户端。前者可以使用 ServerSocket 类实现，而后者可以使用套接字类。

通信过程的工作方式涉及双方。客户端将向服务器发送请求，请求建立连接。这是通过计算机上可用的一个 TCP/IP 端口完成的。如果连接被接受，两端都会打开套接字。服务器和客户端的端点将是唯一可识别的。这意味着您可以使用该端口进行多个连接。

了解如何处理套接字，以及与流一起使用，将使您能够直接从互联网处理信息，这将使您的程序达到新的水平。在接下来的章节中，我们将看到如何实现客户端和服务器以原型化这种通信。

注意

在使用这些示例进行工作时，请确保您的计算机安全系统（防火墙等）允许通过您决定使用的任何端口进行通信。这种情况并非第一次发生，有人浪费了几个小时以为自己的代码有误，而问题实际上出在其他地方。

## 创建 SocketServer

从套接字中读取数据需要现有网络资源的一点点参与。如果您想要一个连接到服务器的程序，您在尝试连接之前需要知道一个服务器。在互联网上，有提供连接可能性的服务器，可以打开套接字，发送数据，并接收回数据。这些服务器被称为 EchoServer——这个名字几乎不会让人对其功能产生怀疑。

另一方面，您可以自己实现服务器并确保安全。Oracle 提供了一个简单的 EchoServer 示例供您测试。这将是一个新的挑战，因为您需要在计算机上同时运行两个程序：EchoServer 和您将实现的任何客户端。

让我们从实现 EchoServer 开始，您可以从[`packt.live/33LmH0k`](https://packt.live/33LmH0k)获取。您要分析的代码包含在下一个示例中。请注意，我们已经删除了开头的免责声明和代码注释，以保持其简洁：

```java
Example14.java
14         try (
15             ServerSocket serverSocket =
16                 new ServerSocket(Integer.parseInt(args[0]));
17             Socket clientSocket = serverSocket.accept();     
18             PrintWriter out =
19                 new PrintWriter(clientSocket.getOutputStream(),                      true);                   
20             BufferedReader in = new BufferedReader(
21                 new InputStreamReader(clientSocket.getInputStream()));
22         ) {
23             String inputLine;
24             while ((inputLine = in.readLine()) != null) {
25                 out.println(inputLine);
26             }
27         } catch (IOException e) {
28             System.out.println("Exception caught when trying to listen on port "
29                 + portNumber + " or listening for a connection");
30             System.out.println(e.getMessage());
31         }
https://packt.live/2oLURSR
```

代码的第一部分检查您是否为服务器选择了一个监听端口。这个端口号作为 CLI 上的参数给出：

```java
if (args.length != 1) {
    System.err.println("Usage: java EchoServer <port number>");
    System.exit(1);
}
```

如果没有选择端口号，此程序将简单地退出。记住，正如我们之前提到的，确保您使用的任何端口号都没有被计算机的防火墙阻止。

调用 `ServerSocket(Integer.parseInt(args[0]))` 将启动 `ServerSocket` 类的对象，配置在参数中定义的端口号，以便调用程序作为监听程序。稍后，`serverSocket.accept()` 将阻塞服务器，使其等待连接的到来。一旦到来，它将自动接受。

在这个示例的初始代码中，有两个不同的流：`BufferedReader in` 用于输入，`PrintWriter out` 用于输出。一旦建立连接，`in` 将获取数据，而 `out` 将发送它——无需任何进一步处理——回套接字。服务器程序将一直运行，直到在终端上按下 *Ctrl*+*C* 强制退出。

要启动服务器，您需要使用构建图标（锤子）编译它，并在终端中使用特定的端口号调用它。尝试端口号 8080，因为这个端口号通常用于我们即将进行的实验：

```java
usr@localhost:~/IdeaProjects/[...]/Example14$ java Example14 8080
```

如果一切按计划进行，程序将开始运行，不会打印任何消息。它只是在那里等待建立连接。

注意

请记住，默认情况下，您的计算机始终具有 IP 地址 127.0.0.1，这允许您确定计算机在网络中的 IP 地址。我们将使用此地址与客户端建立连接。

## 在套接字上写入数据和从套接字读取数据

当我们的服务器在后台运行时，我们需要生成一个简单的程序来打开套接字并向服务器发送一些内容。为此，您需要在 IDE 中创建一个新的项目，但在一个单独的窗口中。记住，您的服务器目前正在运行！

您可以生成的最简单的客户端是 Oracle 的 *EchoServer* 伴侣。出于明显的原因，它被称为 *EchoClient*，您可以在 [`packt.live/2PbLNBx`](https://packt.live/2PbLNBx) 找到它。

```java
Example15.java 
15         try (
16                 Socket echoSocket = new Socket(hostName, portNumber);
17                 PrintWriter out =18                   new PrintWriter(echoSocket.getOutputStream(), true);
19                 BufferedReader in =20                   new BufferedReader(
21                      new InputStreamReader(echoSocket.getInputStream()));
22                 BufferedReader stdIn =23                   new BufferedReader(
24                      new InputStreamReader(System.in))
25         ) {
26             String userInput;
27             while ((userInput = stdIn.readLine()) != null) {
28                 out.println(userInput);
29                 System.out.println("echo: " + in.readLine());
30             }
https://packt.live/33OrP3t
```

注意，在这种情况下，我们不是创建一个 `SocketServer` 对象，而是创建一个 `Socket` 对象。这个第二个程序介绍了使用系统流之一来捕获数据并发送到套接字：`System.in`。这个程序将一直运行，直到 `System.in` 中的输入为 `null`。这是通过直接与 `System.in` 交互无法真正实现的事情，因为我们只是按键盘上的键。因此，您需要按 *Ctrl* + *C* 来停止客户端，就像服务器的情况一样。

注意发送数据到服务器是通过`out.println()`完成的，其中`out`是一个`PrinterWriter`对象，一个流，它是基于`Socket`构建的。另一方面，为了读取传入的`Socket`，我们实现了一个名为`in`的`BufferedReader`对象。由于它是缓冲的，我们可以随时轮询该对象。对`out.readLine()`和`in.readLine()`的调用是阻塞的。它不会停止从`System.in`或从套接字读取，直到行尾到达。

这使得这个读者是同步的，因为它等待用户输入，发送数据，最后等待从套接字获取答案。

注意

每个操作系统都向 JVM 提供了三个不同的系统流：System.in、System.out 和 System.err。由于它们是流，您可以使用 Stream 类的全部功能从它们读取数据，将它们放入缓冲区，解析它们等。

要启动客户端，您需要使用构建图标（锤子图标）编译它，并通过特定的 IP 地址和端口号从终端调用它。尝试使用 IP 地址 127.0.0.1 和端口号 8080。记住，在启动客户端之前，您需要先启动服务器：

```java
usr@localhost:~/IdeaProjects/[...]/Example14$ java Example15 127.0.0.1 8080
```

从那时起，直到您发出 Ctrl + C 命令，只要服务器保持连接，您就可以在终端上输入任何内容，当您按下 Enter 键时，它将被发送到服务器，并从服务器返回。到达后，客户端会在它前面添加消息 echo 将其写入终端。我们通过使来自服务器的响应字体加粗来突出显示：

```java
Hello
echo: Hello
```

此外，当强制客户端退出时，它也会强制服务器退出。

## 活动二：改进 EchoServer 和 EchoClient 程序

在这个活动中，您需要对最后两个部分中的程序进行改进。首先，您需要向服务器上传输的数据中添加一些文本。这将使用户更容易理解数据是从服务器发送回来的。让我们将其制作成一个计数器，它将充当交换的唯一 ID。这样，服务器的回答将显示在消息中添加的数字：

```java
Hello
echo: 37-Hello
```

另一方面，您应该在客户端添加一个命令，该命令将发送一个终止信号到服务器。这个命令将退出服务器，然后退出客户端。要终止任何程序，您可以在向终端发送消息告知用户程序即将结束时调用`System.exit()`。作为一个终止命令，您可以创建一个简单的消息，其中包含单词“bye”。

1.  预期结果将需要您以非常相似的方式修改服务器和客户端。在客户端，您需要做如下操作：

    ```java
    while ((userInput = stdIn.readLine()) != null) {
        out.println(userInput);
        if (userInput.substring(0,3).equals("bye")) {
            System.out.println("Bye bye!");
            System.exit(0);
        }
        System.out.println("echo: " + in.readLine());
    }
    ```

1.  在服务器上，修改应该如下所示：

    ```java
    int contID = 0;
    while ((inputLine = in.readLine()) != null) {
        contID++;
        out.println(contID + "-" + inputLine);
        if (inputLine.substring(0,3).equals("bye")) {
            System.out.println("Bye bye!");
            System.exit(0);
        }
    }
    ```

    服务器和客户端之间的预期交互应该如下所示：

![图 8.1：客户端与服务器之间的交互。

![img/C13927_08_01.jpg]

图 8.1：客户端与服务器之间的交互。

注意

这个活动的解决方案可以在第 555 页找到。

## 阻塞和非阻塞调用

这是本章一直在讨论的话题，但我们没有直接解决它。`java.io`的读写操作是阻塞的。这意味着程序将等待直到数据完全读取或数据已经完全写入。然而，使用`java.nio`中实现的缓冲流可以让你检查数据是否准备好读取。在写入数据时，`java.nio`会将数据复制到缓冲区，并让 API 自己将数据写入通道。这允许一种完全不同的编程风格，我们不需要等待操作发生。同时，这也意味着我们将不会对通信有低级控制。JVM 的另一个部分会为我们执行这个动作。

# 摘要

在本章中，你已经了解了 Java 语言中的两个主要 API：java.io 和 java.nio。它们有一些重叠的功能，并且它们被用来处理流和文件。除此之外，你还看到了如何使用流来处理套接字，套接字是数据的一个自然来源，只能通过流来处理。

已经有一系列示例展示了如何从终端捕获数据，最终发现是`stream (System.in)`。然后你探索了如何使用各种高级函数，如 filter、map、sorted、foreach、reduce 和 collect 来处理它。你已经看到了如何打开文件和属性文件，以及 java.nio 在前者上非常强大，但在后者上则不然。

从更实际的角度来看，本章介绍了一种在早期章节中仅从理论上解释的重要技术：如何使用`finally`来关闭流，并在运行时避免潜在的内存问题。你已经看到，为了干净地处理异常，你可能不得不将代码块移动到方法中。这样，你可以避免抛出异常，并且总是可以通过 try-catch 语句来处理它们。

为了与套接字进行实验，你已经尝试构建了一个 EchoServer 和一个 EchoClient。你有两个不同的程序相互交互，并通过互联网发送数据。你已经看到了如何在你的电脑上运行服务器和客户端，现在是时候尝试在不同的电脑上运行这两个程序了。

最后，本章中介绍的两个活动通过在程序中键入键值对作为参数来动态创建或修改属性文件，以及通过互联网上的命令远程控制另一个程序。

在下一章中，你将学习关于 HTTP 以及如何创建一个连接到特定 Web 服务器并下载数据的程序。
