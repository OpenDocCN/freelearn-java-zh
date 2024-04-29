# 第八章：更好地管理操作系统进程

在本章中，我们将介绍以下内容：

+   生成一个新进程

+   将进程输出和错误流重定向到文件

+   更改子进程的工作目录

+   为子进程设置环境变量

+   运行 shell 脚本

+   获取当前 JVM 的进程信息

+   获取生成的进程的进程信息

+   管理生成的进程

+   枚举系统中正在运行的进程

+   使用管道连接多个进程

+   管理子进程

# 介绍

你有多少次编写了生成新进程的代码？不多。然而，可能有一些情况需要编写这样的代码。在这种情况下，您不得不使用第三方 API，如**Apache Commons Exec**（[`commons.apache.org/proper/commons-exec/`](https://commons.apache.org/proper/commons-exec/)）等。为什么会这样？Java API 不够用吗？不，不够用；至少在 Java 9 之前是这样。现在，有了 Java 9 及以上版本，我们在进程 API 中添加了更多功能。

直到 Java 7，重定向输入、输出和错误流并不是一件简单的事。在 Java 7 中，引入了新的 API，允许将输入、输出和错误重定向到其他进程（管道）、文件或标准输入/输出。然后，在 Java 8 中，又引入了一些新的 API。在 Java 9 中，现在有了以下领域的新 API：

+   获取进程信息，如**进程 ID**（**PID**）、启动进程的用户、运行时间等

+   枚举系统中正在运行的进程

+   通过导航到进程层次结构的上层来管理子进程并访问进程树

在本章中，我们将介绍一些配方，这些配方将帮助您探索进程 API 中的新内容，并了解自`Runtime.getRuntime().exec()`以来引入的更改。而且你们都知道使用那个是犯罪。

所有这些配方只能在 Linux 平台上执行，因为我们将在 Java 代码中使用特定于 Linux 的命令来生成新进程。在 Linux 上执行脚本`run.sh`有两种方法：

+   `sh run.sh`

+   `chmod +x run.sh && ./run.sh`

那些使用 Windows 10 的人不用担心，因为微软发布了 Windows 子系统用于 Linux，它允许您在 Windows 上运行您喜欢的 Linux 发行版，如 Ubuntu、OpenSuse 等。有关更多详细信息，请查看此链接：[`docs.microsoft.com/en-in/windows/wsl/install-win10`](https://docs.microsoft.com/en-in/windows/wsl/install-win10)。

# 生成新进程

在这个配方中，我们将看到如何使用`ProcessBuilder`生成新进程。我们还将看到如何使用输入、输出和错误流。这应该是一个非常简单和常见的配方。然而，引入这个的目的是为了使本章内容更加完整，而不仅仅是关注 Java 9 的特性。

# 准备工作

Linux 中有一个名为`free`的命令，它显示系统中空闲的 RAM 量以及被系统使用的量。它接受一个选项`-m`，以便以兆字节显示输出。因此，只需运行 free `-m`即可得到以下输出：

![](img/037b0235-fb00-4a78-8de8-fa94a9246c5b.png)

我们将在 Java 程序中运行上述代码。

# 如何做...

按照以下步骤进行：

1.  通过提供所需的命令和选项来创建`ProcessBuilder`的实例：

```java
        ProcessBuilder pBuilder = new ProcessBuilder("free", "-m");
```

指定命令和选项的另一种方法如下：

```java
        pBuilder.command("free", "-m");
```

1.  为进程生成器设置输入和输出流以及其他属性，如执行目录和环境变量。然后，在`ProcessBuilder`实例上调用`start()`来生成进程并获取对`Process`对象的引用：

```java
        Process p = pBuilder.inheritIO().start();
```

`inheritIO()`函数将生成的子进程的标准 I/O 设置为与当前 Java 进程相同。

1.  然后，我们等待进程的完成，或者等待一秒钟（以先到者为准），如下面的代码所示：

```java
        if(p.waitFor(1, TimeUnit.SECONDS)){
          System.out.println("process completed successfully");
        }else{
          System.out.println("waiting time elapsed, process did 
                              not complete");   
          System.out.println("destroying process forcibly");
          p.destroyForcibly();
        }
```

如果在指定的时间内没有完成，我们可以通过调用`destroyForcibly()`方法来终止进程。

1.  使用以下命令编译和运行代码：

```java
 $ javac -d mods --module-source-path src
      $(find src -name *.java)
      $ java -p mods -m process/com.packt.process.NewProcessDemo
```

1.  我们得到的输出如下：

![](img/03ef0f79-588a-4721-ab8c-47305fac4e57.png)

此示例的代码可以在`Chapter08/1_spawn_new_process`中找到。

# 工作原理...

有两种方法可以让`ProcessBuilder`知道要运行哪个命令：

+   通过在创建`ProcessBuilder`对象时将命令及其选项传递给构造函数

+   通过将命令及其选项作为参数传递给`ProcessBuilder`对象的`command()`方法

在生成进程之前，我们可以执行以下操作：

+   我们可以使用`directory()`方法更改执行目录。

+   我们可以将输入流、输出流和错误流重定向到文件或另一个进程。

+   我们可以为子进程提供所需的环境变量。

我们将在本章的各自示例中看到所有这些活动。

当调用`start()`方法时，将生成一个新的进程，并且调用者以`Process`类的实例形式获得对该子进程的引用。使用这个`Process`对象，我们可以做很多事情，比如以下事情：

+   获取有关进程的信息，包括其 PID

+   获取输出和错误流

+   检查进程的完成情况

+   销毁进程

+   将任务与进程完成后要执行的操作关联起来

+   检查进程生成的子进程

+   查找进程的父进程（如果存在）

在我们的示例中，我们等待一秒钟，或者等待进程完成（以先到者为准）。如果进程已完成，则`waitFor`返回`true`；否则返回`false`。如果进程没有完成，我们可以通过在`Process`对象上调用`destroyForcibly()`方法来终止进程。

# 将进程输出和错误流重定向到文件

在本示例中，我们将看到如何处理从 Java 代码生成的进程的输出和错误流。我们将把生成的进程产生的输出或错误写入文件。

# 准备工作

在本示例中，我们将使用`iostat`命令。此命令用于报告不同设备和分区的 CPU 和 I/O 统计信息。让我们运行该命令并查看它报告了什么：

```java
$ iostat
```

在某些 Linux 发行版（如 Ubuntu）中，默认情况下未安装`iostat`。您可以通过运行`sudo apt-get install sysstat`来安装该实用程序。

上述命令的输出如下：

![](img/3eedacb8-42e4-4082-ad1b-854d6f31be17.png)

# 如何做...

按照以下步骤进行：

1.  通过指定要执行的命令来创建一个新的`ProcessBuilder`对象：

```java
        ProcessBuilder pb = new ProcessBuilder("iostat");
```

1.  将输出和错误流重定向到文件的输出和错误流中：

```java
        pb.redirectError(new File("error"))
          .redirectOutput(new File("output"));
```

1.  启动进程并等待其完成：

```java
        Process p = pb.start();
        int exitValue = p.waitFor();
```

1.  读取输出文件的内容：

```java
        Files.lines(Paths.get("output"))
                         .forEach(l -> System.out.println(l));
```

1.  读取错误文件的内容。只有在命令出现错误时才会创建此文件：

```java
        Files.lines(Paths.get("error"))
                         .forEach(l -> System.out.println(l));
```

步骤 4 和 5 是供我们参考的。这与`ProcessBuilder`或生成的进程无关。使用这两行代码，我们可以检查进程写入输出和错误文件的内容。

完整的代码可以在`Chapter08/2_redirect_to_file`中找到。

1.  使用以下命令编译代码：

```java
 $ javac -d mods --module-source-path src $(find src -name 
      *.java)
```

1.  使用以下命令运行代码：

```java
 $ java -p mods -m process/com.packt.process.RedirectFileDemo
```

我们将得到以下输出：

![](img/5f927baa-5e79-4813-9aae-060d4d14536d.png)

我们可以看到，由于命令成功执行，错误文件中没有任何内容。

# 还有更多...

您可以向`ProcessBuilder`提供错误的命令，然后看到错误被写入错误文件，输出文件中没有任何内容。您可以通过更改`ProcessBuilder`实例创建来实现这一点，如下所示：

```java
ProcessBuilder pb = new ProcessBuilder("iostat", "-Z");
```

使用前面在*如何做...*部分中给出的命令进行编译和运行。

您会看到错误文件中报告了一个错误，但输出文件中没有任何内容：

![](img/4153b745-4da2-4a5e-863c-c38d2f92b1f0.png)

# 更改子进程的工作目录

通常，您会希望在路径的上下文中执行一个进程，比如列出目录中的文件。为了做到这一点，我们将不得不告诉 `ProcessBuilder` 在给定位置的上下文中启动进程。我们可以通过使用 `directory()` 方法来实现这一点。这个方法有两个目的：

+   当我们不传递任何参数时，它返回执行的当前目录。

+   当我们传递参数时，它将执行的当前目录设置为传递的值。

在这个示例中，我们将看到如何执行

`tree` 命令用于递归遍历当前目录中的所有目录，并以树形式打印出来。

# 准备工作

通常，`tree` 命令不是预装的，因此您将不得不安装包含该命令的软件包。要在 Ubuntu/Debian 系统上安装，请运行以下命令：

```java
$ sudo apt-get install tree
```

要在支持 `yum` 软件包管理器的 Linux 上安装，请运行以下命令：

```java
$ yum install tree
```

要验证您的安装，只需运行 `tree` 命令，您应该能够看到当前目录结构的打印。对我来说，它是这样的：

![](img/d9b3ca66-07fc-4a1d-9127-0f5ee9d94985.png)

`tree` 命令支持多个选项。这是供您探索的。

# 如何做...

按照以下步骤进行：

1.  创建一个新的 `ProcessBuilder` 对象：

```java
        ProcessBuilder pb = new ProcessBuilder();
```

1.  将命令设置为 `tree`，并将输出和错误设置为与当前 Java 进程相同的输出和错误：

```java
        pb.command("tree").inheritIO();
```

1.  将目录设置为您想要的任何目录。我将其设置为根文件夹：

```java
        pb.directory(new File("/root"));
```

1.  启动进程并等待其退出：

```java
        Process p = pb.start();
        int exitValue = p.waitFor();
```

1.  使用以下命令进行编译和运行：

```java
$ javac -d mods --module-source-path src $(find src -name *.java)
$ java -p mods -m process/com.packt.process.ChangeWorkDirectoryDemo
```

1.  输出将是指定在 `ProcessBuilder` 对象的 `directory()` 方法中的目录的递归内容，以树状格式打印出来。

完整的代码可以在 `Chapter08/3_change_work_directory` 找到。

# 它是如何工作的...

`directory()` 方法接受 `Process` 的工作目录的路径。路径被指定为 `File` 的实例。

# 为子进程设置环境变量

环境变量就像我们在编程语言中拥有的任何其他变量一样。它们有一个名称并保存一些值，这些值可以变化。这些被 Linux/Windows 命令或 shell/batch 脚本用来执行不同的操作。它们被称为**环境变量**，因为它们存在于正在执行的进程/命令/脚本的环境中。通常，进程从父进程继承环境变量。

它们在不同的操作系统中以不同的方式访问。在 Windows 中，它们被访问为 `%ENVIRONMENT_VARIABLE_NAME%`，在基于 Unix 的操作系统中，它们被访问为 `$ENVIRONMENT_VARIABLE_NAME`。

在基于 Unix 的系统中，您可以使用 `printenv` 命令打印出进程可用的所有环境变量，在基于 Windows 的系统中，您可以使用 `SET` 命令。

在这个示例中，我们将向子进程传递一些环境变量，并使用 `printenv` 命令打印所有可用的环境变量。

# 如何做...

按照以下步骤进行：

1.  创建一个 `ProcessBuilder` 的实例：

```java
        ProcessBuilder pb = new ProcessBuilder();
```

1.  将命令设置为 `printenv`，并将输出和错误流设置为与当前 Java 进程相同的输出和错误：

```java
        pb.command("printenv").inheritIO();
```

1.  提供环境变量 `COOKBOOK_VAR1` 的值为 `First variable`，`COOKBOOK_VAR2` 的值为 `Second variable`，以及 `COOKBOOK_VAR3` 的值为 `Third variable`：

```java
        Map<String, String> environment = pb.environment();
        environment.put("COOKBOOK_VAR1", "First variable");
        environment.put("COOKBOOK_VAR2", "Second variable");
        environment.put("COOKBOOK_VAR3", "Third variable");

```

1.  启动进程并等待其完成：

```java
        Process p = pb.start();
        int exitValue = p.waitFor();
```

这个示例的完整代码可以在 `Chapter08/4_environment_variables` 找到。

1.  使用以下命令编译和运行代码：

```java
 $ javac -d mods --module-source-path src $(find src -name 
      *.java)
      $ java -p mods -m 
       process/com.packt.process.EnvironmentVariableDemo
```

您得到的输出如下：

![](img/8dde13a3-73c4-4d9f-9978-c9157247592d.png)

您可以看到三个变量打印在其他变量中。

# 它是如何工作的...

当您在`ProcessBuilder`的实例上调用`environment()`方法时，它会复制当前进程的环境变量，将它们填充到`HashMap`的一个实例中，并将其返回给调用者代码。

加载环境变量的所有工作都是由一个包私有的最终类`ProcessEnvironment`完成的，它实际上扩展了`HashMap`。

然后我们利用这个映射来填充我们自己的环境变量，但我们不需要将映射设置回`ProcessBuilder`，因为我们将有一个对映射对象的引用，而不是一个副本。对映射对象所做的任何更改都将反映在`ProcessBuilder`实例持有的实际映射对象中。

# 运行 shell 脚本

我们通常会收集在文件中执行操作的一组命令，称为 Unix 世界中的**shell 脚本**和 Windows 中的**批处理文件**。这些文件中的命令按顺序执行，除非脚本中有条件块或循环。

这些 shell 脚本由它们执行的 shell 进行评估。可用的不同类型的 shell 包括`bash`、`csh`、`ksh`等。`bash` shell 是最常用的 shell。

在这个示例中，我们将编写一个简单的 shell 脚本，然后使用`ProcessBuilder`和`Process`对象从 Java 代码中调用它。

# 准备工作

首先，让我们编写我们的 shell 脚本。这个脚本做了以下几件事：

1.  打印环境变量`MY_VARIABLE`的值

1.  执行`tree`命令

1.  执行`iostat`命令

让我们创建一个名为`script.sh`的 shell 脚本文件，其中包含以下命令：

```java
echo $MY_VARIABLE;
echo "Running tree command";
tree;
echo "Running iostat command"
iostat;
```

您可以将`script.sh`放在您的主文件夹中；也就是说，在`/home/<username>`中。现在让我们看看我们如何从 Java 中执行它。

# 如何做...

按照以下步骤进行：

1.  创建`ProcessBuilder`的一个新实例：

```java
        ProcessBuilder pb = new ProcessBuilder();
```

1.  将执行目录设置为指向 shell 脚本文件的目录：

```java
         pb.directory(new File("/root"));
```

请注意，在创建`File`对象时传递的先前路径将取决于您放置脚本`script.sh`的位置。在我们的情况下，我们将它放在`/root`中。您可能已经将脚本复制到了`/home/yourname`中，因此`File`对象将相应地创建为`newFile("/home/yourname")`。

1.  设置一个将被 shell 脚本使用的环境变量：

```java
    Map<String, String> environment = pb.environment();
    environment.put("MY_VARIABLE", "Set by Java process");
```

1.  设置要执行的命令，以及要传递给命令的参数。还要将进程的输出和错误流设置为与当前 Java 进程相同的流：

```java
       pb.command("/bin/bash", "script.sh").inheritIO();
```

1.  启动进程并等待它完全执行：

```java
         Process p = pb.start();
         int exitValue = p.waitFor();
```

您可以从`Chapter08/5_running_shell_script`获取完整的代码。

您可以使用以下命令编译和运行代码：

```java
$ javac -d mods --module-source-path src $(find src -name *.java)
$ java -p mods -m process/com.packt.process.RunningShellScriptDemo
```

我们得到的输出如下：

![](img/c3f299bd-4969-4329-b0d4-14440e75e29b.png)

# 它是如何工作的...

在这个示例中，您必须记下两件事：

+   将进程的工作目录更改为 shell 脚本的位置。

+   使用`/bin/bash`执行 shell 脚本。

如果你没有记下第一步，那么你将不得不使用 shell 脚本文件的绝对路径。然而，在这个示例中，我们做了这个，因此我们只需使用 shell 脚本名称来执行`/bin/bash`命令。

第 2 步基本上是您希望执行 shell 脚本的方式。要执行此操作的方法是将 shell 脚本传递给解释器，解释器将解释和执行脚本。以下代码行就是这样做的：

```java
pb.command("/bin/bash", "script.sh")
```

# 获取当前 JVM 的进程信息

运行中的进程有一组与之关联的属性，例如以下内容：

+   **PID**：这个唯一标识进程

+   **所有者**：这是启动进程的用户的名称

+   **命令**：这是在进程下运行的命令

+   **CPU 时间**：这表示进程已经活动的时间

+   **开始时间**：这表示进程启动的时间

这些是我们通常感兴趣的一些属性。也许我们还对 CPU 使用率或内存使用率感兴趣。现在，在 Java 9 之前，从 Java 中获取这些信息是不可能的。然而，在 Java 9 中，引入了一组新的 API，使我们能够获取有关进程的基本信息。

在本示例中，我们将看到如何获取当前 Java 进程的进程信息；也就是说，正在执行您的代码的进程。

# 如何做...

按照以下步骤进行：

1.  创建一个简单的类，并使用`ProcessHandle.current()`来获取当前 Java 进程的`ProcessHandle`：

```java
        ProcessHandle handle = ProcessHandle.current();
```

1.  我们添加了一些代码，这将为代码增加一些运行时间：

```java
        for ( int i = 0 ; i < 100; i++){
          Thread.sleep(1000);
        }
```

1.  在`ProcessHandle`实例上使用`info()`方法获取`ProcessHandle.Info`的实例：

```java
        ProcessHandle.Info info = handle.info();
```

1.  使用`ProcessHandle.Info`的实例获取接口提供的所有信息：

```java
        System.out.println("Command line: " + 
                                     info.commandLine().get());
        System.out.println("Command: " + info.command().get());
        System.out.println("Arguments: " + 
                     String.join(" ", info.arguments().get()));
        System.out.println("User: " + info.user().get());
        System.out.println("Start: " + info.startInstant().get());
        System.out.println("Total CPU Duration: " + 
                  info.totalCpuDuration().get().toMillis() +"ms");
```

1.  使用`ProcessHandle`的`pid()`方法获取当前 Java 进程的进程 ID：

```java
        System.out.println("PID: " + handle.pid());
```

1.  我们还将打印结束时间，使用代码即将结束时的时间。这将让我们了解进程的执行时间：

```java
        Instant end = Instant.now();
        System.out.println("End: " + end);
```

您可以从`Chapter08/6_current_process_info`获取完整的代码。

使用以下命令编译和运行代码：

```java
$ javac -d mods --module-source-path src $(find src -name *.java) 
$ java -p mods -m process/com.packt.process.CurrentProcessInfoDemo
```

您将看到的输出将类似于这样：

![](img/303f0d5c-4fbf-4752-8208-859e18cbe6d3.png)

程序执行完成需要一些时间。

需要注意的一点是，即使程序运行了大约两分钟，总 CPU 持续时间也只有 350 毫秒。这是 CPU 繁忙的时间段。

# 它是如何工作的...

为了给本地进程更多的控制并获取其信息，Java API 中添加了一个名为`ProcessHandle`的新接口。使用`ProcessHandle`，您可以控制进程执行并获取有关进程的一些信息。该接口还有一个名为`ProcessHandle.Info`的内部接口。该接口提供了一些 API 来获取有关进程的信息。

有多种方法可以获取进程的`ProcessHandle`对象。以下是其中一些方法：

+   `ProcessHandle.current()`: 用于获取当前 Java 进程的`ProcessHandle`实例。

+   `Process.toHandle()`: 用于获取给定`Process`对象的`ProcessHandle`。

+   `ProcessHandle.of(pid)`: 用于获取由给定 PID 标识的进程的`ProcessHandle`。

在我们的示例中，我们使用第一种方法，即使用`ProcessHandle.current()`。这使我们可以处理当前的 Java 进程。在`ProcessHandle`实例上调用`info()`方法将为我们提供`ProcessHandle.Info`接口的实现的实例，我们可以利用它来获取进程信息，如示例代码所示。

`ProcessHandle`和`ProcessHandle.Info`都是接口。JDK 提供的 Oracle JDK 或 Open JDK 将为这些接口提供实现。Oracle JDK 有一个名为`ProcessHandleImpl`的类，它实现了`ProcessHandle`，还有一个名为`Info`的`ProcessHandleImpl`内部类，它实现了`ProcessHandle.Info`接口。因此，每当调用上述方法之一来获取`ProcessHandle`对象时，都会返回`ProcessHandleImpl`的实例。

`Process`类也是如此。它是一个抽象类，Oracle JDK 提供了一个名为`ProcessImpl`的实现，该实现实现了`Process`类中的抽象方法。

在本章的所有示例中，对`ProcessHandle`实例或`ProcessHandle`对象的任何提及都将指的是`ProcessHandleImpl`的实例或对象，或者是您正在使用的 JDK 提供的任何其他实现类。

此外，对`ProcessHandle.Info`的实例或`ProcessHandle.Info`对象的任何提及都将指的是`ProcessHandleImpl.Info`的实例或对象，或者是您正在使用的 JDK 提供的任何其他实现类。

# 获取生成的进程的进程信息

在我们之前的示例中，我们看到了如何获取当前 Java 进程的进程信息。在这个示例中，我们将看看如何获取由 Java 代码生成的进程的进程信息；也就是说，由当前 Java 进程生成的进程。使用的 API 与我们在之前的示例中看到的相同，只是`ProcessHandle`实例的实现方式不同。

# 准备工作

在这个示例中，我们将使用 Unix 命令`sleep`，它用于暂停执行一段时间（以秒为单位）。

# 如何做...

按照以下步骤进行：

1.  从 Java 代码中生成一个新的进程，运行`sleep`命令：

```java
        ProcessBuilder pBuilder = new ProcessBuilder("sleep", "20");
        Process p = pBuilder.inheritIO().start();
```

1.  获取此生成的进程的`ProcessHandle`实例：

```java
        ProcessHandle handle = p.toHandle();
```

1.  等待生成的进程完成执行：

```java
        int exitValue = p.waitFor();
```

1.  使用`ProcessHandle`获取`ProcessHandle.Info`实例，并使用其 API 获取所需信息。或者，我们甚至可以直接使用`Process`对象通过`Process`类中的`info()`方法获取`ProcessHandle.Info`：

```java
        ProcessHandle.Info info = handle.info();
        System.out.println("Command line: " + 
                                     info.commandLine().get());
        System.out.println("Command: " + info.command().get());
        System.out.println("Arguments: " + String.join(" ", 
                                      info.arguments().get()));
        System.out.println("User: " + info.user().get());
        System.out.println("Start: " + info.startInstant().get());
        System.out.println("Total CPU time(ms): " + 
                        info.totalCpuDuration().get().toMillis());
        System.out.println("PID: " + handle.pid());
```

您可以从`Chapter08/7_spawned_process_info`获取完整的代码。

使用以下命令编译和运行代码：

```java
$ javac -d mods --module-source-path src $(find src -name *.java)
$ java -p mods -m process/com.packt.process.SpawnedProcessInfoDemo
```

另外，在`Chapter08/7_spawned_process_info`中有一个`run.sh`脚本，您可以在任何基于 Unix 的系统上运行`/bin/bash run.sh`。

您看到的输出将类似于这样：

![](img/57dda7ad-1e90-4d94-b97b-c79d84cc1a8c.png)

# 管理生成的进程

有一些方法，如`destroy()`、`destroyForcibly()`（在 Java 8 中添加）、`isAlive()`（在 Java 8 中添加）和`supportsNormalTermination()`（在 Java 9 中添加），可以用于控制生成的进程。这些方法既可以在`Process`对象上使用，也可以在`ProcessHandle`对象上使用。在这里，控制只是检查进程是否存活，如果是，则销毁进程。

在这个示例中，我们将生成一个长时间运行的进程，并执行以下操作：

+   检查其是否存活

+   检查它是否可以正常停止；也就是说，根据平台的不同，进程可以通过 destroy 或 force destroy 来停止

+   停止进程

# 如何做...

1.  从 Java 代码中生成一个新的进程，运行`sleep`命令，比如一分钟或 60 秒：

```java
        ProcessBuilder pBuilder = new ProcessBuilder("sleep", "60");
        Process p = pBuilder.inheritIO().start();

```

1.  等待，比如 10 秒：

```java
        p.waitFor(10, TimeUnit.SECONDS);
```

1.  检查进程是否存活：

```java
        boolean isAlive = p.isAlive();
        System.out.println("Process alive? " + isAlive);
```

1.  检查进程是否可以正常停止：

```java
        boolean normalTermination = p.supportsNormalTermination();
        System.out.println("Normal Termination? " + normalTermination);
```

1.  停止进程并检查其是否存活：

```java
        p.destroy();
        isAlive = p.isAlive();
        System.out.println("Process alive? " + isAlive);
```

您可以从`Chapter08/8_manage_spawned_process`获取完整的代码。

我们提供了一个名为`run.sh`的实用脚本，您可以使用它来编译和运行代码——`sh run.sh`。

我们得到的输出如下：

![](img/6d0480b0-fdf0-4d29-b521-9fac1bba1855.png)

如果我们在 Windows 上运行程序，`supportsNormalTermination()`返回`false`，但在 Unix 上，`supportsNormalTermination()`返回`true`（如前面的输出中所见）。

# 枚举系统中的活动进程

在 Windows 中，您可以打开 Windows 任务管理器来查看当前活动的进程，在 Linux 中，您可以使用`ps`命令及其各种选项来查看进程以及其他详细信息，如用户、时间、命令等。

在 Java 9 中，添加了一个名为`ProcessHandle`的新 API，用于控制和获取有关进程的信息。API 的一个方法是`allProcesses()`，它返回当前进程可见的所有进程的快照。在这个示例中，我们将看看这个方法的工作原理以及我们可以从 API 中提取的信息。

# 如何做...

按照以下步骤进行：

1.  在`ProcessHandle`接口上使用`allProcesses()`方法，以获取当前活动进程的流：

```java
         Stream<ProcessHandle> liveProcesses = 
                       ProcessHandle.allProcesses();
```

1.  使用`forEach()`迭代流，并传递 lambda 表达式以打印可用的详细信息：

```java
         liveProcesses.forEach(ph -> {
           ProcessHandle.Info phInfo = ph.info();
           System.out.println(phInfo.command().orElse("") +" " + 
                              phInfo.user().orElse(""));
         });
```

您可以从`Chapter08/9_enumerate_all_processes`获取完整的代码。

我们提供了一个名为`run.sh`的实用脚本，您可以使用它来编译和运行代码——`sh run.sh`。

我们得到的输出如下：

![](img/0f66a1d3-6f82-42f0-83ec-d85d96fdda0a.png)

在前面的输出中，我们打印了命令名称以及进程的用户。我们展示了输出的一小部分。

# 使用管道连接多个进程

在 Unix 中，通常使用`|`符号将一组命令连接在一起，以创建一系列活动的管道，其中命令的输入是前一个命令的输出。这样，我们可以处理输入以获得所需的输出。

一个常见的场景是当您想要在日志文件中搜索某些内容或模式，或者在日志文件中搜索某些文本的出现时。在这种情况下，您可以创建一个管道，通过一系列命令，即`cat`、`grep`、`wc -l`等，传递所需的日志文件数据。

在这个示例中，我们将使用 UCI 机器学习库中提供的 Iris 数据集（[`archive.ics.uci.edu/ml/datasets/Iris`](https://archive.ics.uci.edu/ml/datasets/Iris)）创建一个管道，我们将统计每种花的出现次数。

# 准备工作

我们已经下载了 Iris Flower 数据集（[`archive.ics.uci.edu/ml/datasets/iris`](https://archive.ics.uci.edu/ml/datasets/iris)），可以在本书的代码下载中的`Chapter08/10_connecting_process_pipe/iris.data`中找到。

如果您查看`Iris`数据，您会看到以下格式的 150 行：

```java
4.7,3.2,1.3,0.2,Iris-setosa
```

这里，有多个由逗号（`,`）分隔的属性，属性如下：

+   花萼长度（厘米）

+   花萼宽度（厘米）

+   花瓣长度（厘米）

+   花瓣宽度（厘米）

+   类别：

+   Iris setosa

+   Iris versicolour

+   Iris virginica

在这个示例中，我们将找到每个类别中花的总数，即 setosa、versicolour 和 virginica。

我们将使用以下命令的管道（使用基于 Unix 的操作系统）：

```java
$ cat iris.data.txt | cut -d',' -f5 | uniq -c
```

我们得到的输出如下：

```java
50 Iris-setosa
50 Iris-versicolor
50 Iris-virginica
1
```

末尾的 1 表示文件末尾有一个新行。所以每个类别有 50 朵花。让我们解析上面的 shell 命令管道并理解它们各自的功能：

+   `cat`：此命令读取作为参数给定的文件。

+   `cut`：这使用`-d`选项中给定的字符拆分每一行，并返回由`-f`选项标识的列中的值。

+   `uniq`：这从给定的值返回一个唯一列表，当使用`-c`选项时，它返回列表中每个唯一值的出现次数。

# 操作步骤

1.  创建一个`ProcessBuilder`对象的列表，其中将保存参与我们的管道的`ProcessBuilder`实例。还将管道中最后一个进程的输出重定向到当前 Java 进程的标准输出：

```java
         List<ProcessBuilder> pipeline = List.of(
           new ProcessBuilder("cat", "iris.data.txt"),
           new ProcessBuilder("cut", "-d", ",", "-f", "5"),
           new ProcessBuilder("uniq", "-c")
               .redirectOutput(ProcessBuilder.Redirect.INHERIT)
         );
```

1.  使用`ProcessBuilder`的`startPipeline()`方法，并传递`ProcessBuilder`对象的列表以启动管道。它将返回一个`Process`对象的列表，每个对象代表列表中的一个`ProcessBuilder`对象：

```java
  List<Process> processes = ProcessBuilder.startPipeline(pipeline);
```

1.  获取列表中的最后一个进程并`waitFor`它完成：

```java
     int exitValue = processes.get(processes.size() - 1).waitFor();
```

您可以从`Chapter08/10_connecting_process_pipe`获取完整的代码。

我们提供了一个名为`run.sh`的实用脚本，您可以使用它来编译和运行代码——`sh run.sh`。

我们得到的输出如下：

![](img/1a049dae-3fd1-4fc8-b049-ec623e088e4d.png)

# 工作原理

`startPipeline()`方法为列表中的每个`ProcessBuilder`对象启动一个`Process`。除了第一个和最后一个进程外，它通过使用`ProcessBuilder.Redirect.PIPE`将一个进程的输出重定向到另一个进程的输入。如果您为任何中间进程提供了`redirectOutput`，而不是`ProcessBuilder.Redirect.PIPE`，那么将会抛出错误；类似于以下内容：

```java
Exception in thread "main" java.lang.IllegalArgumentException: builder redirectOutput() must be PIPE except for the last builder: INHERIT. 
```

它指出除了最后一个之外的任何构建器都应将其输出重定向到下一个进程。对于`redirectInput`也是适用的。

# 管理子进程

当一个进程启动另一个进程时，启动的进程成为启动进程的子进程。启动的进程反过来可以启动另一个进程，这个链条可以继续下去。这导致了一个进程树。通常，我们可能需要处理一个有错误的子进程，可能想要终止该子进程，或者可能想要知道启动的子进程并可能想要获取有关它们的一些信息。

在 Java 9 中，`Process`类中添加了两个新的 API——`children()`和`descendants()`。`children()` API 允许您获取当前进程的直接子进程的快照列表，而`descendants()` API 提供了当前进程递归`children()`的进程的快照；也就是说，它们在每个子进程上递归地调用`children()`。

在这个配方中，我们将查看`children()`和`descendants()` API，并看看我们可以从进程的快照中收集到什么信息。

# 准备就绪

让我们创建一个简单的 shell 脚本，我们将在配方中使用它。此脚本可以在`Chapter08/11_managing_sub_process/script.sh`中找到：

```java
echo "Running tree command";
tree;
sleep 60;
echo "Running iostat command";
iostat;
```

在上述脚本中，我们运行了`tree`和`iostat`命令，中间用一分钟的睡眠时间分隔。如果您想了解这些命令，请参考本章的*运行 shell 脚本*配方。当从 bash shell 中执行时，睡眠命令每次被调用时都会创建一个新的子进程。

我们将创建，比如说，10 个`ProcessBuilder`实例来运行上述的 shell 脚本并同时启动它们。

# 如何做...

1.  我们将创建 10 个`ProcessBuilder`实例来运行我们的 shell 脚本（位于`Chapter08/11_managing_sub_process/script.sh`）。我们不关心它的输出，所以让我们通过将输出重定向到预定义的重定向`ProcessHandle.Redirect.DISCARD`来丢弃命令的输出：

```java
        for ( int i = 0; i < 10; i++){
          new ProcessBuilder("/bin/bash", "script.sh")
              .redirectOutput(ProcessBuilder.Redirect.DISCARD)
              .start();
        }
```

1.  获取当前进程的句柄：

```java
        ProcessHandle currentProcess = ProcessHandle.current();
```

1.  使用当前进程通过`children()` API 获取其子进程，并迭代每个子进程以打印它们的信息。一旦我们有了`ProcessHandle`的实例，我们可以做多种事情，比如销毁进程，获取其进程信息等。

```java
        System.out.println("Obtaining children");
        currentProcess.children().forEach(pHandle -> {
          System.out.println(pHandle.info());
        });
```

1.  使用当前进程通过使用`descendants()` API 获取所有子进程，然后迭代每个子进程以打印它们的信息：

```java
        currentProcess.descendants().forEach(pHandle -> {
          System.out.println(pHandle.info());
        });
```

您可以从`Chapter08/11_managing_sub_process`获取完整的代码。

我们提供了一个名为`run.sh`的实用脚本，您可以使用它来编译和运行代码——`sh run.sh`。

我们得到的输出如下：

![](img/2b31919e-2b42-449a-9838-34099c00b560.png)

# 工作原理...

`children()`和`descendants()` API 返回当前进程的直接子进程或后代进程的`ProcessHandler`的`Stream`。使用`ProcessHandler`的实例，我们可以执行以下操作：

+   获取进程信息

+   检查进程的状态

+   停止进程
