# 第九章：重构遗留代码-使其年轻化

TDD 可能不会立即适应遗留代码。你可能需要稍微调整一下步骤才能使其工作。要明白，在这种情况下，你的 TDD 可能会发生变化，因为你不再执行你习惯的 TDD。本章将向你介绍遗留代码的世界，尽可能多地从 TDD 中获取。

我们将从头开始，处理目前正在生产中的遗留应用程序。我们将以微小的方式进行修改，而不引入缺陷或回归，甚至有时间提前吃午饭！

本章涵盖以下主题：

+   遗留代码

+   处理遗留代码

+   REST 通信

+   依赖注入

+   不同级别的测试：端到端、集成和单元

# 遗留代码

让我们从遗留代码的定义开始。虽然有许多作者对此有不同的定义，比如对应用程序或测试的不信任、不再受支持的代码等等。我们最喜欢迈克尔·菲瑟斯创造的定义：

"遗留代码是没有测试的代码。这个定义的原因是客观的：要么有测试，要么没有测试。"

- 迈克尔·菲瑟斯

我们如何检测遗留代码？尽管遗留代码通常等同于糟糕的代码，但迈克尔·菲瑟斯在他的书《与遗留代码有效地工作》中揭露了一些问题，由 Dorling Kindersley（印度）私人有限公司（1993 年）出版。

**代码异味**。

代码异味是指代码中的某些结构，表明违反了基本设计原则，并对设计质量产生了负面影响。

代码异味通常不是错误——它们在技术上不是不正确的，也不会阻止程序当前的运行。相反，它们表明设计上的弱点可能会减缓开发速度或增加将来出现错误或故障的风险。

来源：[`en.wikipedia.org/wiki/Code_smell`](http://en.wikipedia.org/wiki/Code_smell)。

遗留代码的常见问题之一是*我无法测试这段代码*。它正在访问外部资源，引入其他副作用，使用新的操作符等。一般来说，良好的设计易于测试。让我们看一些遗留代码。

# 遗留代码示例

软件概念通常通过代码最容易解释，这个也不例外。我们已经看到并使用了井字棋应用程序（参见第三章，*红-绿-重构-从失败到成功直至完美*）。以下代码执行位置验证：

```java
public class TicTacToe { 

  public void validatePosition(int x, int y) { 
    if (x < 1 || x > 3) { 
      throw new RuntimeException("X is outside board"); 
    } 
    if (y < 1 || y > 3) { 
      throw new RuntimeException("Y is outside board"); 
    } 
  } 
} 
```

与此代码对应的规范如下：

```java
public class TicTacToeSpec { 
  @Rule 
  public ExpectedException exception = 
      ExpectedException.none(); 

  private TicTacToe ticTacToe; 

  @Before 
  public final void before() { 
    ticTacToe = new TicTacToe(); 
  } 

  @Test 
  public void whenXOutsideBoardThenRuntimeException() { 
    exception.expect(RuntimeException.class); 
    ticTacToe.validatePosition(5, 2); 
  } 

  @Test 
  public void whenYOutsideBoardThenRuntimeException() { 
    exception.expect(RuntimeException.class); 
    ticTacToe.validatePosition(2, 5); 
  } 
} 
```

JaCoCo 报告表明一切都被覆盖了（除了最后一行，方法的结束括号）：

![](img/5ab5211c-027a-4b93-ad7c-e7b568e5a1c3.png)

由于我们相信我们有很好的覆盖率，我们可以进行自动和安全的重构（片段）：

```java
public class TicTacToe { 

  public void validatePosition(int x, int y) { 
    if (isOutsideTheBoard(x)) { 
      throw new RuntimeException("X is outside board"); 
    } 
    if (isOutsideTheBoard(y)) { 
      throw new RuntimeException("Y is outside board"); 
    } 
  } 

  private boolean isOutsideTheBoard(final int position) { 
    return position < 1 || position > 3; 
  } 
} 
```

这段代码应该准备好了，因为测试成功，并且测试覆盖率非常高。

也许你已经意识到了，但有一个问题。`RuntimeException`块中的消息没有经过正确检查；即使代码覆盖率显示它覆盖了该行中的所有分支。

覆盖率到底是什么？

覆盖率是一个用来描述程序源代码被特定测试套件测试的程度的度量。来源：[`en.wikipedia.org/wiki/Code_coverage`](http://en.wikipedia.org/wiki/Code_coverage)。

让我们想象一个覆盖了代码的一部分简单部分的单一端到端测试。这个测试将使你的覆盖率百分比很高，但安全性不高，因为还有许多其他部分没有被覆盖。

我们已经在我们的代码库中引入了遗留代码——异常消息。只要这不是预期行为，这可能没有什么问题——没有人应该依赖异常消息，不是程序员调试他们的程序，或者日志，甚至用户。那些没有被测试覆盖的程序部分很可能在不久的将来遭受回归。如果你接受风险，这可能没问题。也许异常类型和行号就足够了。

我们已经决定删除异常消息，因为它没有经过测试：

```java
public class TicTacToe { 

  public void validatePosition(int x, int y) { 
    if (isOutsideTheBoard(x)) { 
      throw new RuntimeException(""); 
    } 
    if (isOutsideTheBoard(y)) { 
      throw new RuntimeException(""); 
    } 
  } 

  private boolean isOutsideTheBoard(final int position) { 
    return position < 1 || position > 3; 
  } 
} 
```

# 识别遗留代码的其他方法

你可能熟悉以下一些常见的遗留应用程序的迹象：

+   在一个补丁的基础上，就像一个活着的弗兰肯斯坦应用程序

+   已知的错误

+   更改是昂贵的

+   脆弱

+   难以理解

+   旧的，过时的，静态的或者经常是不存在的文档

+   散弹手术

+   破窗效应

关于维护它的团队，这是它对团队成员产生的一些影响：

+   辞职：负责软件的人看到了面前的巨大任务

他们

+   没有人再关心：如果你的系统已经有了破窗，引入新的破窗就更容易。

由于遗留代码通常比其他类型的软件更难，你会希望你最好的人来处理它。然而，我们经常受到截止日期的催促，想要尽快编程所需的功能，并忽略解决方案的质量。

因此，为了避免以这种糟糕的方式浪费我们才华横溢的开发人员，我们期望非遗留应用程序能够实现完全相反的情况。它应该是：

+   易于更改

+   可概括，可配置和可扩展

+   易于部署

+   健壮

+   没有已知的缺陷或限制

+   易于教给他人/从他人学习

+   广泛的测试套件

+   自我验证

+   能够使用钥匙孔手术

由于我们已经概述了遗留和非遗留代码的一些属性，应该很容易用其他质量替换一些质量。对吧？停止散弹手术，使用钥匙孔手术，再加上一些细节，你就完成了。对吧？

这并不像听起来那么容易。幸运的是，有一些技巧和规则，当应用时，可以改进我们的代码，应用程序更接近非遗留代码。

# 缺乏依赖注入

这是遗留代码库中经常检测到的一种味道。由于没有必要单独测试类，协作者在需要时被实例化，将创建协作者和使用它们的责任放在同一个类中。

这里有一个例子，使用`new`操作符：

```java
public class BirthdayGreetingService { 

  private final MessageSender messageSender; 

  public BirthdayGreetingService() { 
    messageSender = new EmailMessageSender(); 
  } 

  public void greet(final Employee employee) { 
    messageSender.send(employee.getAddress(), 
     "Greetings on your birthday"); 
  } 
} 
```

在当前状态下，`BirthdayGreeting`服务不可单元测试。它在构造函数中硬编码了对`EmailMessageSender`的依赖。除了使用反射注入对象或在`new`操作符上替换对象之外，无法替换这种依赖。

修改代码库总是可能引起回归的源头，所以应该谨慎进行。重构需要测试，除非不可能。

遗留代码困境。

当我们改变代码时，应该有测试。要进行测试，我们经常必须改变代码。

# 遗留代码更改算法

当你必须在遗留代码库中进行更改时，这是一个你可以使用的算法：

+   识别更改点

+   找到测试点

+   打破依赖关系

+   编写测试

+   进行更改和重构

# 应用遗留代码更改算法

要应用这个算法，我们通常从一套测试开始，并在重构时始终保持绿色。这与 TDD 的正常周期不同，因为重构不应引入任何新功能（也就是说，不应编写任何新的规范）。

为了更好地解释这个算法，想象一下我们收到了以下更改请求：为了以更非正式的方式向我的员工致以问候，我想发送一条推文而不是一封电子邮件。

# 识别更改点

系统目前只能发送电子邮件，因此需要进行更改。在哪里？快速调查显示，发送祝福的策略是在`BirthdayGreetingService`类的构造函数中决定的，遵循策略模式（[`en.wikipedia.org/?title=Strategy_pattern`](https://en.wikipedia.org/?title=Strategy_pattern)）：

```java
public class BirthdayGreetingService { 

  public BirthdayGreetingService() { 
    messageSender = new EmailMessageSender(); 
  } 
  [...] 
} 
```

# 找到测试点

由于`BirthdayGreetingService`类没有注入的协作者可以用来给对象附加额外的责任，唯一的选择是离开这个服务类来进行测试。一个选择是将`EmailMessageSender`类更改为模拟或虚拟实现，但这会对该类的实现造成风险。

另一个选择是为这个功能创建一个端到端的测试：

```java
public class EndToEndTest { 

  @Test 
  public void email_an_employee() { 
    final StringBuilder systemOutput = 
       injectSystemOutput(); 
    final Employee john = new Employee( 
       new Email("john@example.com")); 

    new BirthdayGreetingService().greet(john); 

    assertThat(systemOutput.toString(),  
      equalTo("Sent email to " 
        + "'john@example.com' with " 
        + "the body 'Greetings on your " 
        + "birthday'\n")); 
  } 

  // This code has been used with permission from 
  //GMaur's LegacyUtils: 
  // https://github.com/GMaur/legacyutils 
  private StringBuilder injectSystemOutput() { 
    final StringBuilder stringBuilder = 
      new StringBuilder(); 
    final PrintStream outputPrintStream = 
      new PrintStream( 
        new OutputStream() { 
        @Override 
        public void write(final int b) 
          throws IOException { 
          stringBuilder.append((char) b); 
        } 
      }); 
    System.setOut(outputPrintStream); 
    return stringBuilder; 
  } 
} 
```

此代码已经获得了[`github.com/GMaur/legacyutils`](https://github.com/GMaur/legacyutils)的许可使用。这个库可以帮助你执行捕获系统输出（`System.out`）的技术。

文件的名称不以 Specification（或`Spec`）结尾，比如`TicTacToeSpec`，因为这不是一个规范。这是一个测试，以确保功能保持不变。文件被命名为`EndToEndTest`，因为我们试图尽可能多地覆盖功能。

# 打破依赖关系

在创建了一个保证预期行为不会改变的测试之后，我们将打破`BirthdayGreetingService`和`EmailMessageSender`之间的硬编码依赖。为此，我们将使用一种称为**提取**和**重写调用**的技术，这首先在 Michael Feathers 的书中解释过：

```java
public class BirthdayGreetingService { 

  public BirthdayGreetingService() { 
    messageSender = getMessageSender(); 
  } 

  private MessageSender getMessageSender() { 
    return new EmailMessageSender(); 
  } 

[...] 
```

再次执行测试，并验证我们之前创建的孤立测试仍然是绿色的。此外，我们需要将这个方法`protected`或更加开放以便进行重写：

```java
public class BirthdayGreetingService { 

  protected MessageSender getMessageSender() { 
    return new EmailMessageSender(); 
  } 

[...] 
```

现在该方法可以被重写，我们创建一个虚拟服务来替换原始服务的实例。在代码中引入虚拟是一种模式，它包括创建一个可以替换现有对象的对象，其特点是我们可以控制其行为。这样，我们可以注入一些定制的虚拟来实现我们的需求。更多信息请参阅[`xunitpatterns.com/`](http://xunitpatterns.com/)。

在这种特殊情况下，我们应该创建一个扩展原始服务的虚拟服务。下一步是重写复杂的方法，以便为测试目的绕过代码的无关部分：

```java
public class FakeBirthdayGreetingService 
 extends BirthdayGreetingService { 

  @Override 
  protected MessageSender getMessageSender() { 
    return new EmailMessageSender(); 
  } 
} 
```

现在我们可以使用虚拟，而不是`BirthdayGreetingService`类：

```java
public class EndToEndTest { 

  @Test 
  public void email_an_employee() { 
    final StringBuilder systemOutput = 
      injectSystemOutput(); 
    final Employee john = new Employee( 
       new Email("john@example.com")); 

    new FakeBirthdayGreetingService().greet(john); 

    assertThat(systemOutput.toString(), 
      equalTo("Sent email to " 
        + "'john@example.com' with " 
        + "the body 'Greetings on  
        + "your birthday'\n")); 
  } 
```

测试仍然是绿色的。

现在我们可以应用另一种打破依赖关系的技术，即参数化构造函数，Feathers 在[`archive.org/details/WorkingEffectivelyWithLegacyCode`](https://archive.org/details/WorkingEffectivelyWithLegacyCode)的论文中有解释。生产代码可能如下所示：

```java
public class BirthdayGreetingService { 

  public BirthdayGreetingService(final MessageSender 
     messageSender) { 
    this.messageSender = messageSender; 
  } 
  [...] 
} 
```

与此实现对应的测试代码可能如下：

```java
public class EndToEndTest { 

  @Test 
  public void email_an_employee() { 
    final StringBuilder systemOutput = 
      injectSystemOutput(); 
    final Employee john = new Employee( 
      new Email("john@example.com")); 

    new BirthdayGreetingService(new 
         EmailMessageSender()).greet(john); 

    assertThat(systemOutput.toString(),  
      equalTo("Sent email to " 
        + "'john@example.com' with " 
        + "the body 'Greetings on " 
        + "your birthday'\n")); 
  } 
  [...] 
```

我们还可以删除`FakeBirthday`，因为它已经不再使用。

# 编写测试

在保留旧的端到端测试的同时，创建一个交互来验证`BirthdayGreetingService`和`MessageSender`的集成：

```java
  @Test 
  public void the_service_should_ask_the_messageSender() { 
    final Email address = 
      new Email("john@example.com"); 
    final Employee john = new Employee(address); 
    final MessageSender messageSender = 
      mock(MessageSender.class); 

    new BirthdayGreetingService(messageSender) 
      .greet(john); 

    verify(messageSender).send(address, 
         "Greetings on your birthday"); 
  } 
```

在这一点上，可以编写一个新的`TweetMessageSender`，完成算法的最后一步。

# kata 练习

程序员唯一能够提高的方法是通过实践。创建不同类型的程序并使用不同的技术通常会为程序员提供对软件构建的新见解。基于这个想法，kata 是一种定义了一些要求或固定特性的练习，以实现一些目标。

程序员被要求实现一个可能的解决方案，然后与其他解决方案进行比较，试图找到最佳解决方案。这个练习的关键价值不在于获得最快的实现，而在于讨论在设计解决方案时所做的决定。在大多数情况下，kata 中创建的所有程序最终都会被丢弃。

本章的 kata 练习是关于一个传统系统。这是一个足够简单的程序，在本章中可以处理，但也足够复杂，会带来一些困难。

# 传统 kata

您已经被分配了一个任务，即接管一个已经在生产中的系统，一个用于图书馆的工作软件：Alexandria 项目。

该项目目前缺乏文档，旧的维护者也不再提供讨论。因此，如果您接受这个任务，这将完全是您的责任，因为没有其他人可以依靠。

# 描述

我们已经能够从原始项目编写时恢复这些规范片段：

+   Alexandria 软件应该能够存储图书并将它们借给用户，用户有权归还图书。用户还可以通过作者、书名、状态和 ID 查询系统中的图书。

+   没有时间限制归还图书。

+   图书也可以被审查，因为这对业务原因很重要。

+   软件不应接受新用户。

+   用户应该在任何时候被告知服务器的时间。

# 技术评论

Alexandria 是一个用 Java 编写的后端项目，它使用 REST API 向前端通信信息。为了这个 kata 练习的目的，持久性已经实现为一个内存对象，使用了在[`xunitpatterns.com/Fake%20Object.html`](http://xunitpatterns.com/Fake%20Object.html)中解释的假测试替身。

代码可以在[`bitbucket.org/vfarcic/tdd-java-alexandria`](https://bitbucket.org/vfarcic/tdd-java-alexandria/)找到。

# 添加新功能

在添加新功能之前，传统代码可能不会干扰程序员的生产力。代码库的状态比期望的要差，但生产系统可以正常工作，没有任何不便。

现在是问题开始出现的时候。**产品所有者**（**PO**）想要添加一个新功能。

例如，作为图书管理员，我想知道给定图书的所有历史，以便我可以衡量哪些图书比其他图书更受欢迎。

# 黑盒或尖刺测试

由于 Alexandria 项目的旧维护者不再提供问题，并且没有文档，黑盒测试变得更加困难。因此，我们决定通过调查更好地了解软件，然后进行一些会泄露系统内部知识的尖刺。

我们将稍后使用这些知识来实现新功能。

黑盒测试是一种软件测试方法，它检查应用程序的功能，而不查看其内部结构或工作原理。这种类型的测试可以应用于几乎每个软件测试的级别：单元、集成、系统和验收。它通常占据大部分，如果不是所有的高级别测试，但也可以主导单元测试。

来源：[`en.wikipedia.org/wiki/Black-box_testing`](http://en.wikipedia.org/wiki/Black-box_testing)。

# 初步调查

当我们知道所需的功能时，我们将开始查看 Alexandria 项目：

+   15 个文件

+   基于 Gradle（`build.gradle`）

+   0 个测试

首先，我们想确认这个项目从未经过测试，缺少测试文件夹也证实了这一点：

```java
    $ find src/test
    find: src/test: No such file or directory

```

这些是 Java 部分的文件夹内容：

```java
    $ cd src/main/java/com/packtpublishing/tddjava/ch09/alexandria/
    $ find .
    .
    ./Book.java
    ./Books.java
    ./BooksEndpoint.java
    ./BooksRepository.java
    ./CustomExceptionMapper.java
    ./MyApplication.java
    ./States.java
    ./User.java
    ./UserRepository.java
    ./Users.java

```

以下是剩下的内容：

```java
    $ cd src/main
    $ find resources webapp
    resources
    resources/applicationContext.xml
    webapp
    webapp/WEB-INF
    webapp/WEB-INF/web.xml

```

这似乎是一个 Web 项目（由`web.xml`文件指示），使用 Spring（由`applicationContext.xml`指示）。`build.gradle`中的依赖项显示如下（片段）：

```java
compile 'org.springframework:spring-web:4.1.4.RELEASE'
```

拥有 Spring 已经是一个好迹象，因为它可以帮助进行依赖注入，但快速查看显示上下文并没有真正被使用。也许这是过去使用过的东西？

在`web.xml`文件中，我们可以找到这个片段：

```java
<?xml version="1.0" encoding="UTF-8"?> 
<web-app version="3.0"  

         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
          http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"> 

    <module-name>alexandria</module-name> 

    <context-param> 
        <param-name>contextConfigLocation</param-name> 
        <param-value>classpath:applicationContext.xml</param-value> 
    </context-param> 

    <servlet> 
        <servlet-name>SpringApplication</servlet-name> 
        <servlet-class>
 org.glassfish.jersey.servlet.ServletContainer</servlet-class> 
        <init-param> 
            <param-name>javax.ws.rs.Application</param-name> 
            <param-value>com.packtpublishing.tddjava.alexandria.MyApplication</param-value> 
        </init-param> 
        <load-on-startup>1</load-on-startup> 
    </servlet> 
```

在这个文件中，我们发现了以下内容：

+   `applicationContext.xml`中的上下文将被加载

+   有一个应用文件（`com.packtpublishing.tddjava.alexandria.MyApplication`）将在一个 servlet 内执行

`MyApplication`文件如下：

```java
public class MyApplication extends ResourceConfig { 

  public MyApplication() { 
    register(RequestContextFilter.class); 
    register(BooksEndpoint.class); 
    register(JacksonJaxbJsonProvider.class); 
    register(CustomExceptionMapper.class); 
  } 
} 
```

配置执行`BooksEndpoint`端点所需的必要类（片段）：

```java
@Path("books") 
@Component 
public class BooksEndpoint { 

  private BooksRepository books = new BooksRepository(); 

  private UserRepository users = new UserRepository(); 
books and users) are created inside the endpoint and not injected. This makes unit testing more difficult.
```

我们可以从写下将在重构过程中使用的元素开始；我们在`BooksEndpoint`中编写**依赖注入**的代码。

# 如何寻找重构的候选对象

有不同的编程范式（例如，函数式、命令式和面向对象）和风格（例如，紧凑、冗长、简约和过于聪明）。因此，不同的人对重构的候选对象也不同。

还有另一种方式，与主观相反，可以客观地寻找重构的候选对象。有许多论文调查了如何客观地寻找重构的候选对象。这只是一个介绍，读者可以自行了解更多有关这些技术的信息。

# 引入新功能

在更深入了解代码之后，似乎最重要的功能性更改是替换当前的“状态”（片段）：

```java
@XmlRootElement 
public class Book { 

  private final String title; 
  private final String author; 
  private int status; //<- this attribute 
  private int id; 
```

并用它们的集合替换（片段）：

```java
@XmlRootElement 
public class Book { 
  private int[] statuses; 
  // ... 
```

这可能看起来可以工作（例如，将所有对该字段的访问更改为数组），但这也引发了一个功能性需求。

Alexandria 软件应该能够存储图书并将它们借给有权归还的用户。用户还可以通过作者、书名、状态和 ID 查询系统中的图书。

PO 确认通过“状态”搜索图书现在已经更改，它还允许搜索任何先前的“状态”。

这个改变越来越大。每当我们觉得是时候移除这个传统代码时，我们就开始应用传统代码算法。

我们还发现了原始执念和特性嫉妒的迹象：将“状态”存储为整数（原始执念），然后对另一个对象的状态进行操作（特性嫉妒）。我们将把这加入以下待办事项清单：

+   `BooksEndpoint`中的依赖注入

+   将“状态”更改为“状态”

+   删除对“状态”的原始执念（可选）

# 应用传统代码算法

在这种情况下，整个中间端作为独立运行，使用内存持久性。如果持久性保存在数据库中，可以使用相同的算法，但我们需要一些额外的代码来在测试运行之间清理和填充数据库。

我们将使用 DbUnit。更多信息可以在[`dbunit.sourceforge.net/`](http://dbunit.sourceforge.net/)找到。

# 编写端到端测试用例

我们决定采取的第一步，以确保在重构过程中保持行为的一致性，是编写端到端测试。在包括前端的其他应用程序中，可以使用更高级的工具，如 Selenium/Selenide。

在我们的情况下，由于前端不需要重构，所以工具可以是更低级的。我们选择编写 HTTP 请求，以进行端到端测试。

这些请求应该是自动的和可测试的，并且应该遵循所有现有的自动测试或规范。当我们在编写这些测试时发现真实的应用行为时，我们决定在一个名为 Postman 的工具中编写一个试验。

产品网站在这里：[`www.getpostman.com/`](https://www.getpostman.com/)。这也可以使用一个名为 curl 的工具（[`curl.haxx.se/`](http://curl.haxx.se/)）。

curl 是什么？

curl 是一个命令行工具和库，用于使用 URL 语法传输数据，支持`[...] HTTP`、`HTTPS`、`HTTP POST`、`HTTP PUT`和`[...]`。

curl 用于什么？

curl 用于命令行或脚本中传输数据。

来源：[`curl.haxx.se/`](http://curl.haxx.se/)。

为此，我们决定使用以下命令在本地执行传统软件：

```java
./gradlew clean jettyRun
```

这将启动一个处理请求的本地 jetty 服务器。最大的好处是部署是自动完成的，不需要打包一切并手动部署到应用服务器（例如，JBoss AS，GlassFish，Geronimo 和 TomEE）。这可以大大加快进行更改并看到效果的过程，从而减少反馈时间。以后，我们将从 Java 代码中以编程方式启动服务器。

我们开始寻找功能。正如我们之前发现的那样，`BooksEndpoint`类包含了 webservice 端点的定义，这是一个开始寻找功能的好地方。它们列如下：

1.  添加一本新书

1.  列出所有的书

1.  按 ID、作者、标题和状态搜索书籍

1.  准备这本书出租

1.  租借这本书

1.  审查这本书

1.  取消审查这本书

我们手动启动服务器并开始编写请求：

![](img/d210ad49-9c7a-48e2-b98e-a6fb7c9f6819.png)

这些测试对于一个暂时的测试来说似乎足够好。我们意识到的一件事是，每个响应都包含一个时间戳，所以这使得我们的自动化更加困难：

![](img/a2cb8545-b5bd-4d7a-adb3-bcd27600297a.png)

为了使测试具有更多的价值，它们应该是自动化和详尽的。目前它们不是，所以我们认为它们是暂时的。它们将在将来自动化。

我们进行的每一个测试都没有自动化。在这种情况下，Postman 界面的测试比自动化测试更快。而且，这种体验更加符合实际生产的使用情况。测试客户端（幸运的是，在这种情况下）可能会对生产环境产生一些问题，因此不能返回可信的结果。

在这种特殊情况下，我们发现 Postman 测试是一个更好的投资，因为即使在编写完它们之后，我们也会将它们丢弃。它们对 API 和结果提供了非常快速的反馈。我们还使用这个工具来原型化 REST API，因为它的工具既有效又有用。

这里的一般想法是：根据你是否想要将这些测试保存到未来，使用一个工具或另一个工具。这也取决于你想要多频繁地执行它们，以及在哪个环境中执行。

在写下所有请求后，这些是我们在应用程序中发现的状态，由状态图表示：

![](img/3ea6c926-39d3-41cc-b9e4-8b78c3c4263b.png)

这些测试准备好后，我们开始理解应用程序，现在是时候自动化测试了。毕竟，如果它们没有自动化，我们对重构就不够自信。

# 自动化测试用例

我们以编程方式启动服务器。为此，我们决定使用 Grizzly（[`javaee.github.io/grizzly/`](https://javaee.github.io/grizzly/)），它允许我们使用 Jersey 的`ResourceConfig`（FQCN：`org.glassfish.jersey.server.ResourceConfig`）的配置来启动服务器，如测试`BooksEndpointTest`（片段）中所示。

代码可以在[`bitbucket.org/vfarcic/tdd-java-alexandria`](https://bitbucket.org/vfarcic/tdd-java-alexandria)找到：

```java
public class BooksEndpointTest { 
    public static final URI FULL_PATH =  
      URI.create("http://localhost:8080/alexandria"); 
    private HttpServer server; 

    @Before 
    public void setUp() throws IOException { 
        ResourceConfig resourceConfig = 
          new MyApplication(); 
        server = GrizzlyHttpServerFactory 
          .createHttpServer(FULL_PATH, resourceConfig); 
        server.start(); 
    } 

    @After 
    public void tearDown(){ 
        server.shutdownNow(); 
    } 
```

这将在地址`http://localhost:8080/alexandria`上准备一个本地服务器。它只会在短时间内可用（测试运行时），所以，如果你需要手动访问服务器，每当你想要暂停执行时，插入一个调用以下方法：

```java
public void pauseTheServer() throws Exception { 
    System.in.read(); 
} 
```

当你想要停止服务器时，停止执行或在分配的控制台中按*Enter*。

现在我们可以以编程方式启动服务器，暂停它（使用前面的方法），并再次执行暂时测试。结果是一样的，所以重构是成功的。

我们向系统添加了第一个自动化测试。

代码可以在[`bitbucket.org/vfarcic/tdd-java-alexandria`](https://bitbucket.org/vfarcic/tdd-java-alexandria)找到：

```java
public class BooksEndpointTest { 

   public static final String AUTHOR_BOOK_1 = 
     "Viktor Farcic and Alex Garcia"; 
    public static final String TITLE_BOOK_1 = 
      "TDD in Java"; 
    private final Map<String, String> TDD_IN_JAVA; 

    public BooksEndpointTest() { 
      TDD_IN_JAVA = getBookProperties(TITLE_BOOK_1, 
        AUTHOR_BOOK_1); 
    } 

    private Map<String, String> getBookProperties 
      (String title, String author) { 
        Map<String, String> bookProperties = 
          new HashMap<>(); 
        bookProperties.put("title", title); 
        bookProperties.put("author", author); 
        return bookProperties; 
    } 

    @Test 
    public void add_one_book() throws IOException { 
        final Response books1 = addBook(TDD_IN_JAVA); 
        assertBooksSize(books1, is("1")); 
    } 

     private void assertBooksSize(Response response, 
        Matcher<String> matcher) { 
        response.then().body(matcher); 
    } 

    private Response addBook 
      (Map<String, ?> bookProperties) { 
        return RestAssured 
            .given().log().path() 
            .contentType(ContentType.URLENC) 
            .parameters(bookProperties) 
            .post("books"); 
    } 
```

为了测试目的，我们使用了一个名为`RestAssured`的库（[`github.com/rest-assured/rest-assured`](https://github.com/rest-assured/rest-assured)），它可以更轻松地测试 REST 和 JSON。

为了完成自动化测试套件，我们创建了这些测试：

+   `add_one_book()`

+   `add_a_second_book()`

+   `get_book_details_by_id()`

+   `get_several_books_in_a_row()`

+   `censor_a_book()`

+   `cannot_retrieve_a_censored_book()`

代码可以在[ https://bitbucket.org/vfarcic/tdd-java-alexandria/](https://bitbucket.org/vfarcic/tdd-java-alexandria/)找到。

现在我们有了一个确保没有引入回归的测试套件，我们来看一下以下的待办事项清单：

1.  书籍的`BooksEndpoint`中的依赖注入

1.  将`status`更改为`statuses`

1.  使用`status`（可选）去除原始偏执

我们将首先解决依赖注入。

# 注入`BookRepository`依赖项

`BookRepository`的依赖代码在`BooksEndpoint`中（片段）：

```java
@Path("books") 
@Component 
public class BooksEndpoint { 

    private BooksRepository books = 
      new BooksRepository(); 
[...] 
```

# 提取和重写调用

我们将应用已经介绍的重构技术提取和重写调用。为此，我们创建一个失败的规范，如下所示：

```java
@Test 
public void add_one_book() throws IOException { 
    addBook(TDD_IN_JAVA); 

    Book tddInJava = new Book(TITLE_BOOK_1, 
      AUTHOR_BOOK_1, 
       States.fromValue(1)); 

    verify(booksRepository).add(tddInJava); 
} 
```

为了通过这个红色的规范，也被称为失败的规范，我们首先将依赖项创建提取到`BookRepository`类的`protected`方法中：

```java
@Path("books") 
@Component 
public class BooksEndpoint { 

    private BooksRepository books = 
      getBooksRepository(); 

    [...] 

     protected BooksRepository 
       getBooksRepository() { 
        return new BooksRepository(); 
    } 

    [...] 
```

我们将`MyApplication`启动器复制到这里：

```java
public class TestApplication 
    extends ResourceConfig { 

    public TestApplication 
      (BooksEndpoint booksEndpoint) { 
        register(booksEndpoint); 
        register(RequestContextFilter.class); 
        register(JacksonJaxbJsonProvider.class); 
        register(CustomExceptionMapper.class); 
    } 

    public TestApplication() { 
        this(new BooksEndpoint( 
          new BooksRepository())); 
    } 
} 
```

这允许我们注入任何`BooksEndpoint`。在这种情况下，在测试`BooksEndpointInteractionTest`中，我们将使用模拟重写依赖项获取器。这样，我们可以检查是否进行了必要的调用（来自`BooksEndpointInteractionTest`的片段）：

```java
@Test 
public void add_one_book() throws IOException { 
    addBook(TDD_IN_JAVA); 
    verify(booksRepository) 
      .add(new Book(TITLE_BOOK_1, 
          AUTHOR_BOOK_1, 1)); 
} 
```

运行测试；一切正常。尽管规范是成功的，但我们为了测试目的引入了一段设计，并且生产代码没有执行这个新的启动器`TestApplication`，而是仍然执行旧的`MyApplication`。为了解决这个问题，我们必须将两个启动器统一为一个。这可以通过重构参数化构造函数来解决，这也在 Roy Osherove 的书《单元测试的艺术》中有解释（[`artofunittesting.com`](http://artofunittesting.com)）。

# 构造函数参数化

我们可以通过接受`BooksEndpoint`依赖项来统一启动器。如果我们不指定，它将使用`BooksRepository`的真实实例注册依赖项。否则，它将注册接收到的依赖项：

```java
public class MyApplication 
      extends ResourceConfig { 

    public MyApplication() { 
        this(new BooksEndpoint( 
          new BooksRepository())); 
    } 

    public MyApplication 
      (BooksEndpoint booksEndpoint) { 
        register(booksEndpoint); 
        register(RequestContextFilter.class); 
        register(JacksonJaxbJsonProvider.class); 
        register(CustomExceptionMapper.class); 
    } 
} 
```

在这种情况下，我们选择了**构造函数链接**来避免构造函数中的重复。

在进行了这次重构之后，`BooksEndpointInteractionTest`类如下

在最终状态中：

```java
public class BooksEndpointInteractionTest { 

    public static final URI FULL_PATH = URI. 
        create("http://localhost:8080/alexandria"); 
    private HttpServer server; 
    private BooksRepository booksRepository; 

    @Before 
    public void setUp() throws IOException { 
        booksRepository = mock(BooksRepository.class); 
        BooksEndpoint booksEndpoint = 
          new BooksEndpoint(booksRepository); 
        ResourceConfig resourceConfig = 
          new MyApplication(booksEndpoint); 
        server = GrizzlyHttpServerFactory 
           .createHttpServer(FULL_PATH, resourceConfig); 
        server.start(); 
    } 
```

第一个测试通过了，所以我们可以将依赖注入任务标记为完成。

已执行的任务：

+   书籍的`BooksEndpoint`中的依赖注入

待办事项清单：

+   将`status`更改为`statuses`

+   去除原始偏执`status`（可选）

# 添加一个新功能

一旦我们有了必要的测试环境，我们就可以添加新功能。

作为图书管理员，我想知道给定书籍的所有历史，以便我可以衡量哪些书籍比其他书籍更受欢迎。

我们将从一个红色的规范开始：

```java
public class BooksSpec { 

    @Test 
    public void should_search_for_any_past_state() { 
        Book book1 = new Book("title", "author", 
           States.AVAILABLE); 
        book1.censor(); 

        Books books = new Books(); 
        books.add(book1); 

        String available = 
          String.valueOf(States.AVAILABLE); 
        assertThat( 
          books.filterByState(available).isEmpty(), 
           is(false)); 
    } 
} 
```

运行所有测试并查看最后一个失败。

实现所有状态的搜索（片段）：

```java
public class Book { 

    private ArrayList<Integer> status; 

    public Book(String title, String author, int status) { 
        this.title = title; 
        this.author = author; 
        this.status = new ArrayList<>(); 
        this.status.add(status); 
    } 

    public int getStatus() { 
        return status.get(status.size()-1); 
    } 

     public void rent() { 
        status.add(States.RENTED); 
    } 
    [...] 

    public List<Integer> anyState() { 
        return status; 
    } 
    [...] 
```

在这个片段中，我们省略了不相关的部分——未修改的部分，或者更改了实现方式的更多修改方法，比如`rent`，它们以相同的方式改变了实现：

```java
public class Books { 
    public Books filterByState(String state) { 
        Integer expectedState = Integer.valueOf(state); 
        return new Books( 
            new ConcurrentLinkedQueue<>( 
                books.stream() 
                  .filter(x 
                 -> x.anyState() 
                  .contains(expectedState)) 
                  .collect(toList()))); 
    } 
    [...] 
```

外部方法，特别是转换为 JSON 的方法，都没有受到影响，因为`getStatus`方法仍然返回一个`int`值。

我们运行所有测试，一切正常。

已执行的任务：

+   书籍的`BooksEndpoint`中的依赖注入

+   将`status`更改为`statuses`

待办事项清单：

+   去除原始偏执`status`（可选）

# 将状态的原始偏执移除为 int

我们决定也解决待办事项清单中的可选项目。

待办事项清单：

+   书籍的`BooksEndpoint`中的依赖注入

+   将`status`更改为`statuses`

+   删除对`status`的原始执着（可选）

气味：原始执着涉及使用原始数据类型来表示领域思想。 例如，我们使用字符串表示消息，整数表示金额，或者使用结构/字典/哈希表示特定对象。

来源是[`c2.com/cgi/wiki?PrimitiveObsession`](http://c2.com/cgi/wiki?PrimitiveObsession)。

由于这是一项重构步骤（即，我们不会向系统引入任何新行为），因此我们不需要任何新的规范。 我们将继续努力，尽量保持绿色，或者尽可能少的时间离开。

我们已将`States`从具有常量的 Java 类转换为：

```java
public class States { 
    public static final int BOUGHT = 1; 
    public static final int RENTED = 2; 
    public static final int AVAILABLE = 3; 
    public static final int CENSORED = 4; 
} 
```

并将其转换为`enum`：

```java
enum States { 
    BOUGHT (1), 
    RENTED (2), 
    AVAILABLE (3), 
    CENSORED (4); 

    private final int value; 

    private States(int value) { 
        this.value = value; 
    } 

    public int getValue() { 
        return value; 
    } 

    public static States fromValue(int value) { 
        for (States states : values()) { 
            if(states.getValue() == value) { 
                return states; 
            } 
        } 
        throw new IllegalArgumentException( 
          "Value '" + value 
    + "' could not be found in States"); 
    } 
} 
```

调整测试如下：

```java
public class BooksEndpointInteractionTest { 
    @Test 
    public void add_one_book() throws IOException { 
        addBook(TDD_IN_JAVA); 
        verify(booksRepository).add( 
            new Book(TITLE_BOOK_1, AUTHOR_BOOK_1, 
              States.BOUGHT)); 
    } 
    [...] 
public class BooksTest { 

    @Test 
    public void should_search_for_any_past_state() { 
        Book book1 = new Book("title", "author", 
           States.AVAILABLE); 
        book1.censor(); 

        Books books = new Books(); 
        books.add(book1); 

        assertThat(books.filterByState( 
            String.valueOf( 
              States.AVAILABLE.getValue())) 
            .isEmpty(), is(false)); 
    } 
    [...] 
```

调整生产代码。 代码片段如下：

```java
@XmlRootElement 
public class Books { 
      public Books filterByState(String state) { 
        State expected = 
          States.fromValue(Integer.valueOf(state)); 
        return new Books( 
            new ConcurrentLinkedQueue<>( 
                books.stream() 
                  .filter(x -> x.anyState() 
                    .contains(expected)) 
                  .collect(toList()))); 
    } 
    [...] 
```

还有以下内容：

```java
@XmlRootElement 
public class Book { 

    private final String title; 
    private final String author; 
    @XmlTransient 
    private ArrayList<States> status; 
    private int id; 

    public Book 
      (String title, String author, States status) { 
        this.title = title; 
        this.author = author; 
        this.status = new ArrayList<>(); 
        this.status.add(status); 
    } 

    public States getStatus() { 
        return status.get(status.size() - 1); 
    } 

    @XmlElement(name = "status") 
    public int getStatusAsInteger(){ 
        return getStatus().getValue(); 
    } 

    public List<States> anyState() { 
        return status; 
    } 
    [...] 
```

在这种情况下，使用注释进行了序列化：

```java
@XmlElement(name = "status") 
```

将方法的结果转换为名为`status`的字段。

此外，现在将`status`字段标记为`ArrayList<States>`，并使用`@XmlTransient`标记，因此不会序列化为 JSON。

我们执行了所有测试，它们都是绿色的，因此我们现在可以划掉待办清单中的可选元素。

执行的任务：

+   对`BooksEndpoint`进行依赖注入

+   将`status`更改为`statuses`

+   删除对`status`的原始执着（可选）

# 总结

如您所知，继承传统代码库可能是一项艰巨的任务。

我们声明传统代码是没有测试的代码，因此处理传统代码的第一步是创建测试，以帮助您在过程中保持相同的功能。 不幸的是，创建测试并不总是像听起来那么容易。 许多时候，传统代码是紧密耦合的，并呈现出其他症状，表明过去设计不佳，或者至少对代码质量缺乏兴趣。 请不要担心：您可以逐步执行一些繁琐的任务，如[`martinfowler.com/bliki/ParallelChange.html`](http://martinfowler.com/bliki/ParallelChange.html)所示。 此外，众所周知，软件开发是一个学习过程。 工作代码是一个副作用。 因此，最重要的部分是更多地了解代码库，以便能够安全地修改它。 请访问[`www.slideshare.net/ziobrando/model-storming`](http://www.slideshare.net/ziobrando/model-storming)获取更多信息。

最后，我们鼓励您阅读迈克尔·菲瑟斯的书《与传统代码有效地工作》。 它有很多针对这种类型代码库的技术，因此对于理解整个过程非常有用。
