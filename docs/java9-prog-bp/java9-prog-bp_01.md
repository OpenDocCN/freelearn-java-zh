# 第一章：介绍

在建造新建筑的过程中，一套蓝图帮助所有相关方进行沟通--建筑师、电工、木工、管道工等等。它详细说明了形状、大小和材料等细节。如果没有这些蓝图，每个分包商都将被迫猜测该做什么、在哪里做以及如何做。没有这些蓝图，现代建筑几乎是不可能的。

你手中的--或者你面前屏幕上的--是一套不同类型的蓝图。与其详细说明如何构建你特定的软件系统，因为每个项目和环境都有独特的约束和要求，这些蓝图提供了如何构建各种基于 Java 的系统的示例，提供了如何在**Java 开发工具包**（或**JDK**）中使用特定功能的示例，特别关注 Java 9 的新功能，然后你可以将其应用到你的具体问题上。

由于仅使用新的 Java 9 功能构建应用程序是不可能的，我们还将使用和突出显示 JDK 中许多最新功能。在我们深入讨论这意味着什么之前，让我们简要讨论一下最近几个主要 JDK 版本中的一些这些伟大的新功能。希望大多数 Java 公司已经在 Java 7 上，所以我们将专注于版本 8，当然还有版本 9。

在本章中，我们将涵盖以下主题：

+   Java 8 中的新功能

+   Java 9 中的新功能

+   项目

# Java 8 中的新功能

Java 8 于 2014 年 3 月 8 日发布，自 2004 年发布的 Java 5 以来，带来了可能是两个最重要的功能--lambda 和流。随着函数式编程在 JVM 世界中日益流行，尤其是在 Scala 等语言的帮助下，Java 的拥护者多年来一直在呼吁更多的函数式语言特性。最初计划在 Java 7 中发布，该功能在那个版本中被删除，最终在 Java 8 中稳定发布。

虽然可以希望每个人都熟悉 Java 的 lambda 支持，但经验表明，出于各种原因，许多公司都很慢地采用新的语言版本和特性，因此快速介绍可能会有所帮助。

# Lambda

lambda 这个术语源自 1936 年由阿隆佐·邱奇开发的λ演算，简单地指的是匿名函数。通常，函数（或者更正式的 Java 术语中的方法）是 Java 源代码中的一个静态命名的实体：

```java
    public int add(int x, int y) { 
      return x + y; 
    } 

```

这个简单的方法是一个名为`add`的方法，它接受两个`int`参数，并返回一个`int`参数。引入 lambda 后，现在可以这样写：

```java
    (int x, int y) → x + y 

```

或者，更简单地说：

```java
    (x, y) → x + y 

```

这种简化的语法表明我们有一个函数，它接受两个参数并返回它们的总和。根据这个 lambda 的使用位置，参数的类型可以被编译器推断出来，使得第二种更简洁的格式成为可能。最重要的是，注意这个方法不再有名称。除非它被分配给一个变量或作为参数传递（稍后会详细介绍），否则它不能被引用--或者在系统中使用。

当然，这个例子太简单了。更好的例子可能在许多 API 中，其中方法的参数是所谓的**单一抽象方法**（**SAM**）接口的实现，至少在 Java 8 之前，这是一个只有一个方法的接口。单一抽象方法的经典例子之一是`Runnable`。以下是使用 lambda 之前的`Runnable`用法的示例：

```java
    Runnable r = new Runnable() { 
      public void run() { 
        System.out.println("Do some work"); 
      } 
    }; 
    Thread t = new Thread(r); 
    t.start(); 

```

有了 Java 8 的 lambda，这段代码可以被大大简化为：

```java
    Thread t = new Thread(() ->
      System.out.println("Do some work")); 
    t.start(); 

```

`Runnable`方法的主体仍然相当琐碎，但在清晰度和简洁度方面的收益应该是相当明显的。

虽然 lambda 是匿名函数（即，它们没有名称），但是在 Java 中，就像许多其他语言一样，lambda 也可以被分配给变量并作为参数传递（实际上，如果没有这种能力，功能几乎没有价值）。重新访问前面代码中的`Runnable`方法，我们可以将声明和使用`Runnable`分开如下：

```java
    Runnable r = () { 
      // Acquire database connection 
      // Do something really expensive 
    }; 
    Thread t = new Thread(r); 
    t.start(); 

```

这比前面的例子更加冗长是有意的。`Runnable`方法的存根体意在模仿，以某种方式，一个真实的`Runnable`可能看起来的样子，以及为什么人们可能希望将新定义的`Runnable`方法分配给一个变量，尽管 lambda 提供了简洁性。这种新的 lambda 语法允许我们声明`Runnable`方法的主体，而不必担心方法名称、签名等。虽然任何像样的 IDE 都会帮助处理这种样板，但这种新语法给你和将来会维护你的代码的无数开发人员更少的噪音来调试代码。

任何 SAM 接口都可以被写成 lambda。你有一个比较器，你只需要使用一次吗？

```java
    List<Student> students = getStudents(); 
    students.sort((one, two) -> one.getGrade() - two.getGrade()); 

```

`ActionListener`怎么样？

```java
    saveButton.setOnAction((event) -> saveAndClose()); 

```

此外，你可以在 lambda 中使用自己的 SAM 接口，如下所示：

```java
    public <T> interface Validator<T> { 
      boolean isValid(T value); 
    } 
    cardProcessor.setValidator((card) 
    card.getNumber().startsWith("1234")); 

```

这种方法的优点之一是它不仅使消费代码更加简洁，而且还减少了创建一些具体 SAM 实例的努力水平。也就是说，开发人员不必再在匿名类和具体命名类之间做选择，可以在内联中声明它，干净而简洁。

除了 Java 开发人员多年来一直在使用的 SAM 之外，Java 8 还引入了许多功能接口，以帮助促进更多的函数式编程风格。Java 8 的 Javadoc 列出了 43 个不同的接口。其中，有一些基本的函数**形状**，你应该知道其中一些如下：

| `BiConsumer<T,U>` | 这代表了接受两个输入参数并且不返回结果的操作 |
| --- | --- |
| `BiFunction<T,U,R>` | 这代表了一个接受两个参数并产生结果的函数 |
| `BinaryOperator<T>` | 这代表了对两个相同类型的操作数进行操作，产生与操作数相同类型的结果 |
| `BiPredicate<T,U>` | 这代表了一个接受两个参数的谓词（布尔值函数） |
| `Consumer<T>` | 这代表了接受单个输入参数并且不返回结果的操作 |
| `Function<T,R>` | 这代表了一个接受一个参数并产生结果的函数 |
| `Predicate<T>` | 这代表了一个接受一个参数的谓词（布尔值函数） |
| `Supplier<T>` | 这代表了一个结果的供应者 |

这些接口有无数的用途，但也许展示其中一些最好的方法是把我们的注意力转向 Java 8 的下一个重大特性--Streams。

# 流

Java 8 的另一个重大增强，也许是 lambda 发挥最大作用的地方，是新的**Streams API**。如果你搜索 Java 流的定义，你会得到从有些循环的**数据元素流**到更技术性的**Java 流是单子**的答案，它们可能都是正确的。Streams API 允许 Java 开发人员通过一系列步骤与数据元素流进行交互。即使这样说也不够清晰，所以让我们通过查看一些示例代码来看看它的含义。

假设你有一个特定班级的成绩列表。你想知道班级中女生的平均成绩是多少。在 Java 8 之前，你可能会写出类似这样的代码：

```java
    double sum = 0.0; 
    int count = 0; 
    for (Map.Entry<Student, Integer> g : grades.entrySet()) { 
      if ("F".equals(g.getKey().getGender())) { 
        count++; 
        sum += g.getValue(); 
      } 
    } 
    double avg = sum / count; 

```

我们初始化两个变量，一个用于存储总和，一个用于计算命中次数。接下来，我们循环遍历成绩。如果学生的性别是女性，我们增加计数器并更新总和。当循环终止时，我们就有了计算平均值所需的信息。这样做是可以的，但有点冗长。新的 Streams API 可以帮助解决这个问题：

```java
    double avg = grades.entrySet().stream() 
     .filter(e -> "F".equals(e.getKey().getGender())) // 1 
     .mapToInt(e -> e.getValue()) // 2 
     .average() // 3 
     .getAsDouble(); //4 

```

这个新版本并没有显著变小，但代码的目的更加清晰。在之前的预流代码中，我们必须扮演计算机的角色，解析代码并揭示其预期目的。有了流，我们有了一个清晰的、声明性的方式来表达应用逻辑。对于映射中的每个条目，执行以下操作：

1.  过滤掉`gender`不是`F`的每个条目。

1.  将每个值映射为原始 int。

1.  计算平均成绩。

1.  以 double 形式返回值。

有了基于流和 lambda 的方法，我们不需要声明临时的中间变量（成绩计数和总数），也不需要担心计算明显简单的平均值。JDK 为我们完成了所有繁重的工作。

# 新的 java.time 包

虽然 lambda 和 streams 是非常重要的改变性更新，但是在 Java 8 中，我们得到了另一个期待已久的改变，至少在某些领域中同样令人兴奋：一个新的日期/时间 API。任何在 Java 中使用日期和时间的人都知道`java.util.Calendar`等的痛苦。显然，你可以完成工作，但并不总是美观的。许多开发人员发现 API 太痛苦了，所以他们将极其流行的 Joda Time 库集成到他们的项目中。Java 的架构师们同意了，并邀请了 Joda Time 的作者 Stephen Colebourne 来领导 JSR 310，这将 Joda Time 的一个版本（修复了各种设计缺陷）引入了平台。我们将在本书后面详细介绍如何在我们的日期/时间计算器中使用一些这些新的 API。

# 默认方法

在我们将注意力转向 Java 9 之前，让我们再看看另一个重要的语言特性：默认方法。自 Java 开始以来，接口被用来定义类的外观，暗示一种特定的行为，但无法实现该行为。在许多情况下，这使得多态性变得更简单，因为任意数量的类都可以实现给定的接口，并且消费代码将它们视为该接口，而不是它们实际上是什么具体类。

多年来，API 开发人员面临的问题之一是如何在不破坏现有代码的情况下发展 API 及其接口。例如，考虑 JavaServer Faces 1.1 规范中的`ActionSource`接口。当 JSF 1.2 专家组在制定规范的下一个修订版时，他们确定需要向接口添加一个新属性，这将导致两个新方法——getter 和 setter。他们不能简单地将方法添加到接口中，因为那样会破坏规范的每个实现，需要实现者更新他们的类。显然，这种破坏是不可接受的，因此 JSF 1.2 引入了`ActionSource2`，它扩展了`ActionSource`并添加了新方法。虽然许多人认为这种方法很丑陋，但 1.2 专家组有几种选择，而且都不是很好的选择。

然而，通过 Java 8，接口现在可以在接口定义上指定默认方法，如果扩展类没有提供方法实现，编译器将使用该默认方法。让我们以以下代码片段为例：

```java
    public interface Speaker { 
      void saySomething(String message); 
    } 
    public class SpeakerImpl implements Speaker { 
      public void saySomething(String message) { 
        System.out.println(message); 
      } 
    } 

```

我们开发了我们的 API 并向公众提供了它，它被证明非常受欢迎。随着时间的推移，我们发现了一个我们想要做出的改进：我们想要添加一些便利方法，比如`sayHello()`和`sayGoodbye()`，以节省我们的用户一些时间。然而，正如前面讨论的那样，如果我们只是将这些新方法添加到接口中，一旦他们更新到库的新版本，我们就会破坏我们用户的代码。默认方法允许我们扩展接口，并通过定义一个实现来避免破坏：

```java
    public interface Speaker { 
      void saySomething(String message); 
      default public void sayHello() { 
        System.out.println("Hello"); 
      } 
      default public void sayGoodbye() { 
        System.out.println("Good bye"); 
      } 
    } 

```

现在，当用户更新他们的库 JAR 时，他们立即获得这些新方法及其行为，而无需进行任何更改。当然，要使用这些方法，用户需要修改他们的代码，但他们不需要在想要使用之前这样做。

# Java 9 中的新功能

与 JDK 的任何新版本一样，这个版本也充满了许多很棒的新功能。当然，最吸引人的是基于您的需求而变化的，但我们将专注于一些最相关于我们将共同构建的项目的这些新功能。首先是最重要的，Java 模块系统。

# Java 平台模块系统/项目 Jigsaw

尽管 Java 8 是一个功能丰富的稳定版本，但许多人认为它有点令人失望。它缺乏备受期待的**Java 平台模块系统**（**JPMS**），也更为通俗，尽管不太准确地称为项目 Jigsaw。Java 平台模块系统最初计划在 2011 年的 Java 7 中发布，但由于一些悬而未决的技术问题，它被推迟到了 Java 8。Jigsaw 项目不仅旨在完成模块系统，还旨在将 JDK 本身模块化，这将有助于 Java SE 缩小到更小的设备，如手机和嵌入式系统。Jigsaw 原计划在 2014 年发布的 Java 8 中发布，但由于 Java 架构师认为他们仍需要更多时间来正确实现系统，因此又一次推迟了。不过，最终，Java 9 将终于交付这个长期承诺的项目。

话虽如此，它到底是什么？长期以来困扰 API 开发人员的一个问题，包括 JDK 架构师在内，就是无法隐藏公共 API 的实现细节。JDK 中一个很好的例子是开发人员不应直接使用的私有类`com.sun.*/sun.*`包和类。私有 API 广泛公开使用的一个完美例子是`sun.misc.Unsafe`类。除了在 Javadoc 中强烈警告不要使用这些内部类之外，几乎没有什么可以阻止它们的使用。直到现在。

有了 JPMS，开发人员将能够使实现类公开，以便它们可以在其项目内轻松使用，但不将它们暴露给模块外部，这意味着它们不会暴露给 API 或库的消费者。为此，Java 架构师引入了一个新文件`module-info.java`，类似于现有的`package-info.java`文件，位于模块的根目录，例如`src/main/java/module-info.java`。它被编译为`module-info.class`，并且可以通过反射和新的`java.lang.Module`类在运行时使用。

那么这个文件是做什么的，它是什么样子的？Java 开发人员可以使用这个文件来命名模块，列出其依赖关系，并向系统表达，无论是编译时还是运行时，哪些包被导出到世界上。例如，假设在我们之前的流示例中，我们有三个包：`model`，`api`和`impl`。我们想要公开模型和 API 类，但不公开任何实现类。我们的`module-info.java`文件可能看起来像这样：

```java
    module com.packt.j9blueprints.intro { 
      requires com.foo; 
      exports com.packt.j9blueprints.intro.model; 
      exports com.packt.j9blueprints.intro.api; 
    } 

```

这个定义暴露了我们想要导出的两个包，并声明了对`com.foo`模块的依赖。如果这个模块在编译时不可用，项目将无法构建，如果在运行时不可用，系统将抛出异常并退出。请注意，`requires`语句没有指定版本。这是有意的，因为决定不将版本选择问题作为模块系统的一部分来解决，而是留给更合适的系统，比如构建工具和容器。

当然，关于模块系统还可以说更多，但对其所有功能和限制的详尽讨论超出了本书的范围。我们将把我们的应用程序实现为模块，因此我们将在整本书中看到这个系统的使用——也许会更详细地解释一下。

想要更深入讨论 Java 平台模块系统的人可以搜索马克·莱恩霍尔德的文章《模块系统的现状》。

# 进程处理 API

在之前的 Java 版本中，与本地操作系统进程交互的开发人员必须使用一个相当有限的 API，一些操作需要使用本地代码。作为**Java Enhancement Proposal**（**JEP**）102 的一部分，Java 进程 API 被扩展了以下功能（引用自 JEP 文本）：

+   获取当前 Java 虚拟机的 pid（或等效值）以及使用现有 API 创建的进程的 pid。

+   枚举系统上的进程的能力。每个进程的信息可能包括其 pid、名称、状态，以及可能的资源使用情况。

+   处理进程树的能力；特别是一些销毁进程树的方法。

+   处理数百个子进程的能力，可能会将输出或错误流多路复用，以避免为每个子进程创建一个线程。

我们将在我们的第一个项目中探索这些 API 的变化，即进程查看器/管理器（详细信息请参见以下各节）。

# 并发变化

与 Java 7 中所做的一样，Java 架构师重新审视了并发库，做出了一些非常需要的改变，这一次是为了支持反应式流规范。这些变化包括一个新的类，`java.util.concurrent.Flow`，带有几个嵌套接口：`Flow.Processor`、`Flow.Publisher`、`Flow.Subscriber`和`Flow.Subscription`。

# REPL

一个似乎激动了很多人的变化并不是语言上的改变。它是增加了一个**REPL**（**读取-求值-打印-循环**），这是一个对语言外壳的花哨术语。事实上，这个新工具的命令是`jshell`。这个工具允许我们输入或粘贴 Java 代码并立即得到反馈。例如，如果我们想要尝试前一节讨论的 Streams API，我们可以这样做：

```java
$ jshell 
|  Welcome to JShell -- Version 9-ea 
|  For an introduction type: /help intro 

jshell> List<String> names = Arrays.asList(new String[]{"Tom", "Bill", "Xavier", "Sarah", "Adam"}); 
names ==> [Tom, Bill, Xavier, Sarah, Adam] 

jshell> names.stream().sorted().forEach(System.out::println); 
Adam 
Bill 
Sarah 
Tom 
Xavier 

```

这是一个非常受欢迎的补充，应该有助于 Java 开发人员快速原型和测试他们的想法。

# 项目

通过这个简短而高层次的概述，我们可以看到有哪些新功能可以使用，那么我们将要涵盖的这些蓝图是什么样的呢？我们将构建十个不同的应用程序，涉及各种复杂性和种类，并涵盖各种关注点。在每个项目中，我们将特别关注我们正在突出的新功能，但我们也会看到一些旧的、经过验证的语言特性和广泛使用的库，其中任何有趣或新颖的用法都会被标记出来。因此，这是我们的项目阵容。

# 进程查看器/管理器

当我们实现一个 Java 版本的古老的 Unix 工具——**top**时，我们将探索一些进程处理 API 的改进。结合这个 API 和 JavaFX，我们将构建一个图形工具，允许用户查看和管理系统上运行的进程。

这个项目将涵盖以下内容：

+   Java 9 进程 API 增强

+   JavaFX

# 重复文件查找器

随着系统的老化，文件系统中杂乱的机会，特别是重复的文件，似乎呈指数增长。利用一些新的文件 I/O 库，我们将构建一个工具，扫描一组用户指定的目录以识别重复项。我们将从工具箱中取出 JavaFX，添加一个图形用户界面，以提供更加用户友好的交互式处理重复项的方式。

这个项目将涵盖以下内容：

+   Java 文件 I/O

+   哈希库

+   JavaFX

# 日期计算器

随着 Java 8 的发布，Oracle 集成了一个基于 Joda Time 重新设计的新库到 JDK 中。这个新库被官方称为 JSR 310，它解决了 JDK 的一个长期的问题——官方的日期库不够充分且难以使用。在这个项目中，我们将构建一个简单的命令行日期计算器，它将接受一个日期，并且例如添加任意数量的时间。例如，考虑以下代码片段：

```java
$ datecalc "2016-07-04 + 2 weeks" 
2016-07-18 
$ datecalc "2016-07-04 + 35 days" 
2016-08-08 
$ datecalc "12:00CST to PST" 
10:00PST 

```

这个项目将涵盖以下内容：

+   Java 8 日期/时间 API

+   正则表达式

+   Java 命令行库

# 社交媒体聚合器

在许多社交媒体网络上拥有帐户的问题之一是难以跟踪每个帐户上发生的情况。拥有 Twitter、Facebook、Google+、Instagram 等帐户的活跃用户可能会花费大量时间从一个站点跳转到另一个站点，或者从一个应用程序跳转到另一个应用程序，阅读最新的更新。在本章中，我们将构建一个简单的聚合应用程序，从用户的每个社交媒体帐户中获取最新的更新，并在一个地方显示它们。功能将包括以下内容：

+   各种社交媒体网络的多个帐户：

+   Twitter

+   Pinterest

+   Instagram

+   只读的、丰富的社交媒体帖子列表

+   链接到适当的站点或应用程序，以便快速简便地进行后续跟进

+   桌面和移动版本

这个项目将涵盖以下内容：

+   REST/HTTP 客户端

+   JSON 处理

+   JavaFX 和 Android 开发

考虑到这一努力的规模和范围，我们将在两章中实际完成这个项目：第一章是 JavaFX，第二章是 Android。

# 电子邮件过滤

管理电子邮件可能会很棘手，特别是如果你有多个帐户。如果您从多个位置访问邮件（即从多个桌面或移动应用程序），管理您的电子邮件规则可能会更加棘手。如果您的邮件系统不支持存储在服务器上的规则，您将不得不决定在哪里放置规则，以便它们最常运行。通过这个项目，我们将开发一个应用程序，允许我们编写各种规则，然后通过可选的后台进程运行它们，以保持您的邮件始终得到适当的管理。

一个样本`rules`文件可能看起来像这样：

```java
    [ 
      { 
        "serverName": "mail.server.com", 
        "serverPort": "993", 
        "useSsl": true, 
        "userName": "me@example.com", 
        "password": "password", 
        "rules": [ 
           {"type": "move", 
               "sourceFolder": "Inbox", 
               "destFolder": "Folder1", 
               "matchingText": "someone@example.com"}, 
            {"type": "delete", 
               "sourceFolder": "Ads", 
               "olderThan": 180} 
         ] 
      } 
    ] 

```

这个项目将涵盖以下内容：

+   JavaMail

+   JavaFX

+   JSON 处理

+   操作系统集成

+   文件 I/O

# JavaFX 照片管理

Java 开发工具包有一个非常强大的图像处理 API。在 Java 9 中，这些 API 得到了改进，增强了对 TIFF 规范的支持。在本章中，我们将使用这个 API 创建一个图像/照片管理应用程序。我们将添加支持从用户指定的位置导入图像到配置的官方目录。我们还将重新访问重复文件查找器，并重用作为项目一部分开发的一些代码，以帮助我们识别重复的图像。

这个项目将涵盖以下内容：

+   新的`javax.imageio`包

+   JavaFX

+   NetBeans 丰富的客户端平台

+   Java 文件 I/O

# 客户端/服务器笔记应用程序

您是否曾经使用过基于云的笔记应用？您是否想知道制作自己的笔记应用需要什么？在本章中，我们将创建这样一个应用程序，包括完整的前端和后端。在服务器端，我们将把数据存储在备受欢迎的文档数据库 MongoDB 中，并通过 REST 接口公开应用程序的业务逻辑的适当部分。在客户端，我们将使用 JavaScript 开发一个非常基本的用户界面，让我们可以尝试并演示如何在我们的 Java 项目中使用 JavaScript。

该项目将涵盖以下内容：

+   文档数据库（MongoDB）

+   JAX-RS 和 RESTful 接口

+   JavaFX

+   JavaScript 和 Vue 2

# 无服务器 Java

无服务器，也被称为**函数即服务**（**FaaS**），是当今最热门的趋势之一。这是一种应用/部署模型，其中一个小函数部署到一个服务中，该服务几乎管理函数的每个方面——启动、关闭、内存等，使开发人员不必担心这些细节。在本章中，我们将编写一个简单的无服务器 Java 应用程序，以了解如何完成，以及如何在自己的应用程序中使用这种新技术。

该项目将涵盖以下内容：

+   创建 Amazon Web Services 账户

+   配置 AWS Lambda、简单通知服务、简单邮件服务和 DynamoDB

+   编写和部署 Java 函数

# Android 桌面同步客户端

通过这个项目，我们将稍微改变方向，专注于 Java 生态系统的另一个部分：Android。为了做到这一点，我们将专注于一个仍然困扰一些 Android 用户的问题——Android 设备与桌面（或笔记本电脑）系统的同步。虽然各种云服务提供商都在推动我们将更多内容存储在云端并将其流式传输到设备上，但一些人仍然更喜欢直接在设备上存储照片和音乐，原因各种各样，从云资源成本到不稳定的无线连接和隐私问题。

在本章中，我们将构建一个系统，允许用户在他们的设备和桌面或笔记本电脑之间同步音乐和照片。我们将构建一个 Android 应用程序，提供用户界面来配置和监视从移动设备端进行同步，以及在后台执行同步的 Android 服务（如果需要）。我们还将在桌面端构建相关组件——一个图形应用程序来配置和监视来自桌面端的同步过程，以及一个后台进程来处理来自桌面端的同步。

该项目将涵盖以下内容：

+   Android

+   用户界面

+   服务

+   JavaFX

+   REST

# 入门

我们已经快速浏览了一些我们将要使用的新语言特性。我们也简要概述了我们将要构建的项目。最后一个问题仍然存在：我们将使用什么工具来完成我们的工作？

当涉及到开发工具时，Java 生态系统拥有丰富的选择，因此我们有很多选择。我们面临的最基本的选择是构建工具。在这里，我们将使用 Maven。虽然有一个强大而有声望的社区支持 Gradle，但 Maven 似乎是目前最常见的构建工具，并且似乎得到了主要 IDE 的更健壮、更成熟和更本地的支持。如果您尚未安装 Maven，您可以访问[`maven.apache.org`](http://maven.apache.org/)并下载适合您操作系统的分发版，或者使用您的操作系统支持的任何软件包管理系统。

对于 IDE，所有的截图、指导等都将使用 NetBeans——来自 Oracle 的免费开源 IDE。当然，也有 IntelliJ IDEA 和 Eclipse 的支持者，它们都是不错的选择，但是 NetBeans 提供了一个完整而强大的开发工具，并且快速、稳定且免费。要下载 NetBeans，请访问[`netbeans.org`](http://netbeans.org/)并下载适合您操作系统的安装程序。由于我们使用 Maven，而 IDEA 和 Eclipse 都支持，您应该能够在您选择的 IDE 中打开这里提供的项目。但是，当 GUI 中显示步骤时，您需要根据您选择的 IDE 进行调整。

在撰写本文时，NetBeans 的最新版本是 8.2，使用它进行 Java 9 开发的最佳方法是在 Java 8 上运行 IDE，并将 Java 9 添加为 SDK。有一个可以在 Java 9 上运行的 NetBeans 开发版本，但是由于它是一个开发版本，有时可能不稳定。稳定的 NetBeans 9 应该会在 Java 9 本身发布时大致同时推出。与此同时，我们将继续使用 8.2：

1.  要添加 Java 9 支持，我们需要添加一个新的 Java 平台，我们将通过点击“工具”|“平台”来实现。

1.  这将打开 Java 平台管理器屏幕：

![](img/9212fff4-28c7-4275-9304-ccf89a7720a5.png)

1.  点击屏幕左下角的“添加平台”。

![](img/12d5edc8-c463-425c-9728-8b0ec8225d09.png)

1.  我们想要添加一个 Java 标准版平台，所以我们将接受默认设置并点击“下一步”。

![](img/a4bb0f81-3af0-4268-babe-5772f0880e9c.png)

1.  在“添加 Java 平台”屏幕上，我们将导航到我们安装 Java 9 的位置，选择 JDK 目录，然后点击“下一步”。

![](img/0cf9f89c-a226-425c-835a-8b1c7d0b27ef.png)

1.  我们需要给新的 Java 平台命名（NetBeans 默认为一个非常合理的 JDK 9），所以我们将点击“完成”现在可以看到我们新添加的 Java 9 选项。

![](img/b1400a67-3da0-4cc6-84f0-820fe8cb437e.png)

设置了项目 SDK 后，我们准备好尝试一下这些新的 Java 9 功能，我们将从第二章“在 Java 中管理进程”开始进行。

如果您在 Java 9 上运行 NetBeans，这本书出版时应该是可能的，您将已经配置了 Java 9。但是，如果您需要特定版本，可以使用前面的步骤来配置 Java 8。

# 摘要

在本章中，我们快速浏览了 Java 8 中一些出色的新功能，包括 lambda、streams、新的日期/时间包和默认方法。从 Java 9 开始，我们快速浏览了 Java 平台模块系统和项目 Jigsaw、进程处理 API、新的并发更改以及新的 Java REPL。对于每个功能，我们都讨论了“是什么”和“为什么”，并查看了一些示例，了解了它们可能如何影响我们编写的系统。我们还看了一下本书中将要构建的项目类型和我们将要使用的工具。

在我们继续之前，我想重申一个早前的观点——每个软件项目都是不同的，因此不可能以一种简单的方式来编写这本书，让您可以简单地将大段代码复制粘贴到您的项目中。同样，每个开发人员编写代码的方式也不同；我构建代码的方式可能与您的大不相同。因此，在阅读本书时，重要的是不要被细节困扰。这里的目的不是向您展示使用这些 API 的唯一正确方式，而是给您一个示例，让您更好地了解它们可能如何使用。从每个示例中学习，根据自己的需要进行修改，然后构建出令人惊叹的东西。

说了这么多，现在让我们把注意力转向我们的第一个项目，进程管理器和新的进程处理 API。
