# 使用 JShell 的读取-评估-打印循环（REPL）

在本章中，我们将涵盖以下内容：

+   熟悉 REPL

+   导航 JShell 及其命令

+   评估代码片段

+   JShell 中的面向对象编程

+   保存和恢复 JShell 命令历史

+   使用 JShell Java API

# 介绍

**REPL**代表**读取-评估-打印循环**，正如其名称所示，它读取在命令行上输入的命令，评估它，打印评估结果，并在输入任何命令时继续此过程。

所有主要语言，如 Ruby、Scala、Python、JavaScript 和 Groovy，都有 REPL 工具。Java 一直缺少这个必不可少的 REPL。如果我们要尝试一些示例代码，比如使用`SimpleDateFormat`解析字符串，我们必须编写一个包含所有仪式的完整程序，包括创建一个类，添加一个主方法，然后是我们想要进行实验的单行代码。然后，我们必须编译和运行代码。这些仪式使得实验和学习语言的特性变得更加困难。

使用 REPL，您只需输入您感兴趣的代码行，并且您将立即得到有关表达式是否在语法上正确并且是否给出所需结果的反馈。REPL 是一个非常强大的工具，特别适合初次接触该语言的人。假设您想展示如何在 Java 中打印*Hello World*；为此，您必须开始编写类定义，然后是`public static void main(String [] args)`方法，最后您将解释或尝试解释许多概念，否则对于新手来说将很难理解。

无论如何，从 Java 9 开始，Java 开发人员现在可以停止抱怨缺少 REPL 工具。一个名为 JShell 的新的 REPL 被捆绑到了 JDK 安装中。因此，我们现在可以自豪地将*Hello World*作为我们的第一个*Hello World*代码。

在本章中，我们将探索 JShell 的特性，并编写代码，这些代码将真正使我们惊叹并欣赏 REPL 的力量。我们还将看到如何使用 JShell Java API 创建我们自己的 REPL。

# 熟悉 REPL

在这个配方中，我们将看一些基本操作，以帮助我们熟悉 JShell 工具。

# 准备工作

确保您安装了最新的 JDK 版本，其中包含 JShell。JShell 从 JDK 9 开始可用。

# 如何做...

1.  您应该将`%JAVA_HOME%/bin`（在 Windows 上）或`$JAVA_HOME/bin`（在 Linux 上）添加到您的`PATH`变量中。如果没有，请访问第一章中的*在 Windows 上安装 JDK 18.9 并设置 PATH 变量*和*在 Linux（Ubuntu，x64）上安装 JDK 18.9 并配置 PATH 变量*这两个配方。

1.  在命令行上，输入`jshell`并按*Enter*。

1.  您将看到一条消息，然后是一个`jshell>`提示：

![](img/e2b2e440-013a-437a-b2fe-0667e558a108.png)

1.  斜杠(`/`)，后跟 JShell 支持的命令，可帮助您与 JShell 进行交互。就像我们尝试`/help intro`以获得以下内容：

![](img/5c77960d-77a3-4adc-b739-afbe540bc99c.png)

1.  让我们打印一个`Hello World`消息：

![](img/91029a86-a887-453e-b956-f2970368704a.png)

1.  让我们打印一个自定义的`Hello World`消息：

![](img/446baf78-c749-42d9-9e80-b174d6a0168f.png)

1.  您可以使用上下箭头键浏览执行的命令。

# 它是如何工作的...

在`jshell`提示符中输入的代码片段被包装在足够的代码中以执行它们。因此，变量、方法和类声明被包装在一个类中，表达式被包装在一个方法中，该方法又被包装在一个类中。其他东西，如导入和类定义，保持原样，因为它们是顶级实体，即在另一个类定义中包装一个类定义是不需要的，因为类定义是一个可以独立存在的顶级实体。同样，在 Java 中，导入语句可以单独出现，它们出现在类声明之外，因此不需要被包装在一个类中。

在接下来的示例中，我们将看到如何定义一个方法，导入其他包，并定义类。

在前面的示例中，我们看到了`$1 ==> "Hello World"`。如果我们有一些值没有与之关联的变量，`jshell`会给它一个变量名，如`$1`或`$2`。

# 导航 JShell 及其命令

为了利用工具，我们需要熟悉如何使用它，它提供的命令以及我们可以使用的各种快捷键，以提高生产力。在这个示例中，我们将看看我们可以通过 JShell 导航的不同方式，以及它提供的不同键盘快捷键，以便在使用它时提高生产力。

# 如何做...

1.  通过在命令行上键入`jshell`来生成`JShell`。您将收到一个欢迎消息，其中包含开始的说明。

1.  键入`/help intro`以获得关于 JShell 的简要介绍：

![](img/b70a0a34-b052-4ee7-913d-280051978691.png)

1.  键入`/help`以获取支持的命令列表：

![](img/eaef19b9-06dc-4b27-a94f-e613306bf0cc.png)

1.  要获取有关命令的更多信息，请键入`/help <command>`。例如，要获取有关`/edit`的信息，请键入`/help /edit`：

![](img/9b27e73d-2cc1-4d0d-b575-201dfd8ad866.png)

1.  JShell 中有自动补全支持。这使得 Java 开发人员感到宾至如归。您可以使用*Tab*键来调用自动补全：

![](img/be642668-7b04-4b58-89a2-2377e5816a36.png)

1.  您可以使用`/!`来执行先前执行的命令，使用`/line_number`在行号重新执行表达式。

1.  要通过命令行导航光标，使用*Ctrl* + *A*到达行的开头，使用*Ctrl* + *E*到达行的结尾。

# 评估代码片段

在这个示例中，我们将看到执行以下代码片段：

+   导入语句

+   类声明

+   接口声明

+   方法声明

+   字段声明

+   语句

# 如何做...

1.  打开命令行并启动 JShell。

1.  默认情况下，JShell 导入了一些库。我们可以通过发出`/imports`命令来检查：

![](img/53d40193-23d2-4d38-9add-d41744f823a0.png)

1.  通过发出`import java.text.SimpleDateFormat`命令来导入`java.text.SimpleDateForm`。这将导入`SimpleDateFormat`类。

1.  让我们声明一个`Employee`类。我们将每行发出一个语句，以便它是一个不完整的语句，并且我们将以与任何普通编辑器相同的方式进行。下面的插图将澄清这一点：

```java
        class Employee{
          private String empId;
          public String getEmpId() {
            return empId;
          }
          public void setEmpId ( String empId ) {
            this.empId = empId;
          }
        }

```

您将得到以下输出：

![](img/f1891d8e-b111-4e17-8bfe-6507c85e19c3.png)

1.  让我们声明一个`Employability`接口，它定义了一个名为`employable()`的方法，如下面的代码片段所示：

```java
        interface Employability { 
          public boolean employable();
        }
```

通过`jshell`创建的前面的接口如下截图所示：

![](img/c9c87021-e754-422a-ae41-94d6894e703c.png)

1.  让我们声明一个`newEmployee(String empId)`方法，它用给定的`empId`构造一个`Employee`对象：

```java
        public Employee newEmployee(String empId ) {
          Employee emp = new Employee();
          emp.setEmpId(empId);
          return emp;
        }
```

JShell 中定义的前面的方法如下所示：

![](img/9bd0dddc-3b37-433a-ad81-33da5acb6c52.png)

1.  我们将使用前一步中定义的方法来创建一个声明`Employee`变量的语句：

```java
        Employee e = newEmployee("1234");
e.get + Tab key generates autocompletion as supported by the IDEs:
```

![](img/8b923e7a-b343-4711-8557-b54d6aab68fe.png)

# 还有更多...

我们可以调用一个未定义的方法。看一下下面的例子：

```java
public void newMethod(){
  System.out.println("New  Method");
  undefinedMethod();
}
```

下面的图片显示了`newMethod()`调用`undefinedMethod()`的定义：

![](img/5a00c521-41c3-4a74-9c11-8cd8eb7e0b36.png)

但是，在使用方法之前，不能调用该方法：

```java
public void undefinedMethod(){
  System.out.println("Now defined");
}
```

下面的图片显示了定义`undefinedMethod()`，然后可以成功调用`newMethod()`：

![](img/98ba3d0a-baba-47c9-aa22-c64fdbe1fde4.png)

只有在我们定义了`undefinedMethod()`之后才能调用`newMethod()`。

# JShell 中的面向对象编程

在这个示例中，我们将使用预定义的 Java 类定义文件并将它们导入到 JShell 中。然后，我们将在 JShell 中使用这些类。

# 如何做...

1.  我们将在这个示例中使用的类定义文件在本书的代码下载中的`Chapter12/4_oo_programming`中可用。

1.  有三个类定义文件：`Engine.java`，`Dimensions.java`和`Car.java`。

1.  导航到这三个类定义文件可用的目录。

1.  `/open`命令允许我们从文件中加载代码。

1.  加载`Engine`类定义并创建一个`Engine`对象：

![](img/7b878742-330f-4cb5-aef2-7b8394a76bb2.png)

1.  加载`Dimensions`类定义并创建一个`Dimensions`对象：

![](img/4998d93c-85f2-4174-abe9-fbf9719054a0.png)

1.  加载`Car`类定义并创建一个`Car`对象：

![](img/d0ee5237-e262-4cd5-82f3-fa32cba224f5.png)

# 保存和恢复 JShell 命令历史

我们将尝试在`jshell`中执行一些代码片段，作为向新手解释 Java 编程的手段。此外，记录执行的代码片段的形式对于正在学习语言的人将是有用的。

在这个示例中，我们将执行一些代码片段并将它们保存到一个文件中。然后我们将从保存的文件中加载代码片段。

# 如何做...

1.  让我们执行一系列的代码片段，如下所示：

```java
        "Hello World"
        String msg = "Hello, %s. Good Morning"
        System.out.println(String.format(msg, "Friend"))
        int someInt = 10
        boolean someBool = false
        if ( someBool ) {
          System.out.println("True block executed");
        }
        if ( someBool ) {
          System.out.println("True block executed");
        }else{
          System.out.println("False block executed");
        }
        for ( int i = 0; i < 10; i++ ){
          System.out.println("I is : " + i );
        }
```

您将得到以下输出：

![](img/3a9285e5-04ba-4e58-b76f-f9a75bf7c4a1.png)

1.  使用`/save history`命令将执行的代码片段保存到名为`history`的文件中。

1.  使用`dir`或`ls`退出 shell，并列出目录中的文件，具体取决于操作系统。列表中将会有一个`history`文件。

1.  打开`jshell`并使用`/list`检查执行的代码片段的历史记录。您会看到没有执行任何代码片段。

1.  使用`/open history`加载`history`文件，然后使用`/list`检查执行的代码片段的历史记录。您将看到所有先前执行的代码片段被执行并添加到历史记录中：

![](img/47ce2ead-0eb4-4a84-8a94-0b4c07917ad0.png)

# 使用 JShell Java API

JDK 11 提供了用于评估 Java 代码片段的工具（如`jshell`）的 Java API。这个 Java API 存在于`jdk.jshell`模块中（[`cr.openjdk.java.net/~rfield/arch/doc/jdk/jshell/package-summary.html`](http://cr.openjdk.java.net/~rfield/arch/doc/jdk/jshell/package-summary.html)）。因此，如果您想在应用程序中使用 API，您需要声明对`jdk.jshell`模块的依赖。

在这个示例中，我们将使用 JShell JDK API 来评估简单的代码片段，并且您还将看到不同的 API 来获取 JShell 的状态。这个想法不是重新创建 JShell，而是展示如何使用其 JDK API。

在这个示例中，我们将不使用 JShell；相反，我们将按照通常的方式使用`javac`进行编译，并使用`java`进行运行。

# 如何做...

1.  我们的模块将依赖于`jdk.jshell`模块。因此，模块定义将如下所示：

```java
        module jshell{
          requires jdk.jshell;
        }
```

1.  使用`jdk.jshell.JShell`类的`create()`方法或`jdk.jshell.JShell.Builder`中的构建器 API 创建一个实例：

```java
        JShell myShell = JShell.create();
```

1.  使用`java.util.Scanner`从`System.in`中读取代码片段：

```java
        try(Scanner reader = new Scanner(System.in)){
          while(true){
            String snippet = reader.nextLine();
            if ( "EXIT".equals(snippet)){
              break;
            }
          //TODO: Code here for evaluating the snippet using JShell API
          }
        }
```

1.  使用`jdk.jshell.JShell#eval(String snippet)`方法来评估输入。评估将导致`jdk.jshell.SnippetEvent`的列表，其中包含评估的状态和输出。上述代码片段中的`TODO`将被以下行替换：

```java
        List<SnippetEvent> events = myShell.eval(snippet);
        events.stream().forEach(se -> {
          System.out.print("Evaluation status: " + se.status());
          System.out.println(" Evaluation result: " + se.value());
        });
```

1.  当评估完成时，我们将使用`jdk.jshell.JShell.snippets()`方法打印处理的代码片段，该方法将返回已处理的`Snippet`的`Stream`。

```java
        System.out.println("Snippets processed: ");
        myShell.snippets().forEach(s -> {
          String msg = String.format("%s -> %s", s.kind(), s.source());
          System.out.println(msg);
        });
```

1.  类似地，我们可以打印活动方法和变量，如下所示：

```java
        System.out.println("Methods: ");
        myShell.methods().forEach(m -> 
          System.out.println(m.name() + " " + m.signature()));

        System.out.println("Variables: ");
        myShell.variables().forEach(v -> 
          System.out.println(v.typeName() + " " + v.name()));
```

1.  在应用程序退出之前，我们通过调用其`close()`方法关闭`JShell`实例：

```java
        myShell.close();
```

此示例的代码可以在`Chapter12/6_jshell_api`中找到。您可以使用同一目录中提供的`run.bat`或`run.sh`脚本来运行示例。示例执行和输出如下所示：

![](img/47744154-6e86-4218-b780-966a546e8b13.png)

# 工作原理...

```java
eval(String snippet) method. We can even drop the previously-evaluated snippet using the drop(Snippet snippet) method. Both these methods result in a change of the internal state maintained by jdk.jshell.JShell.
```

传递给`JShell`评估引擎的代码片段被分类如下：

+   **错误**：语法错误的输入

+   **表达式**：可能会产生一些输出的输入

+   **导入**：导入语句

+   **方法**：方法声明

+   **语句**：语句

+   类型声明：类型，即类/接口声明

+   **变量声明**：变量声明

所有这些类别都在`jdk.jshell.Snippet.Kind`枚举中捕获。

```java
jdk.jshell.Snippet class.
```
