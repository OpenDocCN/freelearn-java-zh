# 模拟 - 消除外部依赖

“空谈是廉价的。给我看代码。”

- Linus Torvalds

TDD 是关于速度的。我们希望快速证明一个想法、概念或实现是否有效。此外，我们希望快速运行所有测试。这种速度的主要瓶颈是外部依赖关系。设置测试所需的数据库数据可能是耗时的。执行验证使用第三方 API 的代码的测试可能会很慢。最重要的是，编写满足所有外部依赖关系的测试可能会变得太复杂，不值得。模拟内部和外部依赖关系有助于解决这些问题。

我们将在第三章中构建*红-绿-重构 - 从失败到成功直至完美*中所做的工作。我们将扩展井字棋以使用 MongoDB 作为数据存储。我们的单元测试实际上不会使用 MongoDB，因为所有通信都将被模拟。最后，我们将创建一个集成测试，验证我们的代码和 MongoDB 确实集成在一起。

本章将涵盖以下主题：

+   模拟

+   Mockito

+   井字棋 v2 的要求

+   开发井字棋 v2

+   集成测试

# 模拟

每个做过比*Hello World*更复杂的应用程序的人都知道，Java 代码充满了依赖关系。可能有团队其他成员编写的类和方法、第三方库或我们与之通信的外部系统。甚至 JDK 内部的库也是依赖关系。我们可能有一个业务层，它与数据访问层通信，后者又使用数据库驱动程序来获取数据。在进行单元测试时，我们进一步考虑所有公共和受保护的方法（甚至是我们正在工作的类内部的方法）都是应该被隔离的依赖关系。

在单元测试级别进行 TDD 时，创建考虑所有这些依赖关系的规范可能会非常复杂，以至于测试本身会成为瓶颈。它们的开发时间可能会增加到 TDD 带来的好处很快被不断增加的成本所掩盖。更重要的是，这些依赖关系往往会创建非常复杂的测试，这些测试包含的错误比实际实现本身还要多。

单元测试的想法（特别是与 TDD 结合在一起时）是编写验证单个单元代码是否有效的规范，而不考虑依赖关系。当依赖关系是内部的时，它们已经经过测试，我们知道它们会按我们的期望工作。另一方面，外部依赖关系需要信任。我们必须相信它们能正确工作。即使我们不相信，对 JDK `java.nio`类进行深度测试的任务对大多数人来说太大了。此外，这些潜在问题将在运行功能和集成测试时出现。

在专注于单元时，我们必须尽量消除单元可能使用的所有依赖关系。通过设计和模拟的组合来实现这些依赖关系的消除。

使用模拟的好处包括减少代码依赖性和更快的文本执行。

模拟是测试快速执行和集中在单个功能单元上的能力的先决条件。通过模拟被测试方法外部的依赖关系，开发人员能够专注于手头的任务，而不必花时间设置它们。在更大的团队或多个团队一起工作的情况下，这些依赖关系甚至可能尚未开发。此外，没有模拟的测试执行往往会很慢。模拟的良好候选对象包括数据库、其他产品、服务等。

在我们深入研究模拟之前，让我们先看看为什么有人会首先使用它们。

# 为什么要使用模拟？

以下列表列出了我们使用模拟对象的一些原因：

+   对象生成不确定的结果。例如，`java.util.Date()`每次实例化时都会提供不同的结果。我们无法测试其结果是否符合预期：

```java
java.util.Date date = new java.util.Date(); 
date.getTime(); // What is the result this method returns?
```

+   对象尚不存在。例如，我们可能创建一个接口并针对其进行测试。在我们测试使用该接口的代码时，实现该接口的对象可能尚未编写。

+   对象速度慢，需要时间来处理。最常见的例子是数据库。我们可能有一个检索所有记录并生成报告的代码。这个操作可能持续几分钟、几小时，甚至在某些情况下可能持续几天。

在支持模拟对象的前述原因适用于任何类型的测试。然而，在单元测试的情况下，尤其是在 TDD 的背景下，还有一个原因，也许比其他原因更重要。模拟允许我们隔离当前正在工作的方法使用的所有依赖项。这使我们能够集中精力在单个单元上，并忽略该单元调用的代码的内部工作。

# 术语

术语可能有点令人困惑，特别是因为不同的人对同一件事使用不同的名称。更让事情变得更加复杂的是，模拟框架在命名其方法时往往不一致。

在我们继续之前，让我们简要介绍一下术语。

**测试替身**是以下所有类型的通用名称：

+   虚拟对象的目的是充当真实方法参数的替代品

+   测试存根可用于使用特定于测试的对象替换真实对象，向被测系统提供所需的间接输入

+   **测试间谍**捕获了**被测系统**（**SUT**）间接输出给另一个组件的调用，以便后续由测试进行验证

+   模拟对象替换了 SUT 依赖的对象，使用一个特定于测试的对象来验证 SUT 是否正确使用它

+   虚拟对象用更轻量级的实现替换了 SUT 依赖的组件

如果您感到困惑，知道您并不是唯一一个可能会有帮助。事情比这更复杂，因为在框架或作者之间没有明确的协议，也没有命名标准。术语令人困惑且不一致，前面提到的术语绝不是所有人都接受的。

为了简化事情，在本书中我们将使用 Mockito（我们选择的框架）使用相同的命名。这样，您将使用的方法将与您将在后面阅读的术语对应。我们将继续使用模拟作为其他人可能称为**测试替身**的通用术语。此外，我们将使用模拟或间谍术语来指代`Mockito`方法。

# 模拟对象

模拟对象模拟了真实（通常是复杂的）对象的行为。它允许我们创建一个将替换实现代码中使用的真实对象的对象。模拟对象将期望一个定义的方法和定义的参数返回期望的结果。它预先知道应该发生什么以及我们期望它如何反应。

让我们看一个简单的例子：

```java
TicTacToeCollection collection = mock(TicTacToeCollection.class); 
assertThat(collection.drop()).isFalse();
doReturn(true).when(collection).drop(); 

assertThat(collection.drop()).isTrue();
```

首先，我们定义`collection`为`TicTacToeCollection`的`mock`。此时，来自该模拟对象的所有方法都是虚假的，并且在 Mockito 的情况下返回默认值。这在第二行得到了确认，我们`assert`了`drop`方法返回`false`。接着，我们指定我们的模拟对象`collection`在调用`drop`方法时应返回`true`。最后，我们`assert`了`drop`方法返回`true`。

我们创建了一个模拟对象，它返回默认值，并且对其方法之一定义了应该返回的值。在任何时候都没有使用真实对象。

稍后，我们将使用具有此逻辑反转的间谍；一个对象使用真实方法，除非另有规定。当我们开始扩展我们的井字棋应用程序时，我们将很快看到并学到更多关于模拟的知识。现在，让我们先看看一个名为 Mockito 的 Java 模拟框架。

# Mockito

Mockito 是一个具有清晰简洁 API 的模拟框架。使用 Mockito 生成的测试可读性强，易于编写，直观。它包含三个主要的静态方法：

+   `mock()`: 用于创建模拟。可选地，我们可以使用`when()`和`given()`指定这些模拟的行为。

+   `spy()`: 这可以用于部分模拟。间谍对象调用真实方法，除非我们另有规定。与`mock()`一样，行为可以针对每个公共或受保护的方法进行设置（不包括静态方法）。主要区别在于`mock()`创建整个对象的伪造，而`spy()`使用真实对象。

+   `verify()`: 用于检查是否使用给定参数调用了方法。这是一种断言形式。

一旦我们开始编写井字棋 v2 应用程序，我们将深入研究 Mockito。然而，首先让我们快速浏览一组新的需求。

# 井字棋 v2 需求

我们的井字棋 v2 应用程序的需求很简单。我们应该添加一个持久存储，以便玩家可以在以后的某个时间继续玩游戏。我们将使用 MongoDB 来实现这一目的。

将 MongoDB 持久存储添加到应用程序中。

# 开发井字棋 v2

我们将在第三章中继续进行井字棋的工作，*红-绿-重构 - 从失败到成功直至完美*。到目前为止，已开发的应用程序的完整源代码可以在[`bitbucket.org/vfarcic/tdd-java-ch06-tic-tac-toe-mongo.git`](https://bitbucket.org/vfarcic/tdd-java-ch06-tic-tac-toe-mongo.git)找到。使用 IntelliJ IDEA 的 VCS|从版本控制|Git 选项来克隆代码。与任何其他项目一样，我们需要做的第一件事是将依赖项添加到`build.gradle`中：

```java
dependencies { 
    compile 'org.jongo:jongo:1.1' 
    compile 'org.mongodb:mongo-java-driver:2.+' 
    testCompile 'junit:junit:4.12' 
    testCompile 'org.mockito:mockito-all:1.+' 
} 
```

导入 MongoDB 驱动程序应该是不言自明的。Jongo 是一组非常有用的实用方法，使得使用 Java 代码更类似于 Mongo 查询语言。对于测试部分，我们将继续使用 JUnit，并添加 Mockito 的模拟、间谍和验证功能。

您会注意到，我们直到最后才会安装 MongoDB。使用 Mockito，我们将不需要它，因为我们所有的 Mongo 依赖项都将被模拟。

一旦指定了依赖项，请记得在 IDEA Gradle 项目对话框中刷新它们。

源代码可以在`tdd-java-ch06-tic-tac-toe-mongo` Git 存储库的`00-prerequisites`分支中找到（[`bitbucket.org/vfarcic/tdd-java-ch06-tic-tac-toe-mongo/branch/00-prerequisites`](https://bitbucket.org/vfarcic/tdd-java-ch06-tic-tac-toe-mongo/branch/00-prerequisites)）。

现在我们已经设置了先决条件，让我们开始处理第一个需求。

# 需求 1 - 存储移动

我们应该能够将每个移动保存到数据库中。由于我们已经实现了所有的游戏逻辑，这应该很容易做到。尽管如此，这将是模拟使用的一个非常好的例子。

实现一个选项，可以保存单个移动与轮数、*x*和*y*轴位置以及玩家（`X`或`O`）。

我们应该首先定义代表我们数据存储模式的 Java bean。这没有什么特别的，所以我们将跳过这一部分，只有一个注释。

不要花太多时间为 Java 样板代码定义规范。我们的 bean 实现包含重写的`equals`和`hashCode`。这两者都是由 IDEA 自动生成的，除了满足比较相同类型的两个对象的需求之外，它们并没有提供真正的价值（我们稍后将在规范中使用该比较）。TDD 应该帮助我们设计更好的代码。编写 15-20 个规范来定义可以由 IDE 自动编写的样板代码（如`equals`方法）并不会帮助我们实现这些目标。精通 TDD 不仅意味着学会如何编写规范，还意味着知道何时不值得。

也就是说，查看源代码以查看 bean 规范和实现的全部内容。

源代码可以在`tdd-java-ch06-tic-tac-toe-mongo` Git 存储库的`01-bean`分支中找到（[`bitbucket.org/vfarcic/tdd-java-ch06-tic-tac-toe-mongo/branch/01-bean`](https://bitbucket.org/vfarcic/tdd-java-ch06-tic-tac-toe-mongo/branch/01-bean)）。特定的类是`TicTacToeBeanSpec`和`TicTacToeBean`。

现在，让我们来到一个更有趣的部分（但仍然没有模拟、间谍和验证）。让我们编写与将数据保存到 MongoDB 相关的规范。

对于这个要求，我们将在`com.packtpublishing.tddjava.ch03tictactoe.mongo`包内创建两个新类：

+   `TicTacToeCollectionSpec`（在`src/test/java`内）

+   `TicTacToeCollection`（在`src/main/java`内）

# 规范-数据库名称

我们应该指定我们将使用的数据库的名称：

```java
@Test 
public void whenInstantiatedThenMongoHasDbNameTicTacToe() { 
  TicTacToeCollection collection = new TicTacToeCollection(); 
  assertEquals(
     "tic-tac-toe",
     collection.getMongoCollection().getDBCollection().getDB().getName()); 
} 
```

我们正在实例化一个新的`TicTacToeCollection`类，并验证 DB 名称是否符合我们的预期。

# 实施

实施非常简单，如下所示：

```java
private MongoCollection mongoCollection; 
protected MongoCollection getMongoCollection() { 
  return mongoCollection; 
}
public TicTacToeCollection() throws UnknownHostException { 
  DB db = new MongoClient().getDB("tic-tac-toe"); 
  mongoCollection = new Jongo(db).getCollection("bla"); 
} 
```

在实例化`TicTacToeCollection`类时，我们正在创建一个新的`MongoCollection`，并将指定的 DB 名称（`tic-tac-toe`）分配给局部变量。

请耐心等待。在我们使用模拟和间谍的有趣部分之前，只剩下一个规范了。

# 规范-用于 Mongo 集合的名称

在先前的实现中，我们使用`bla`作为集合的名称，因为`Jongo`强制我们放一些字符串。让我们创建一个规范，来定义我们将使用的 Mongo 集合的名称：

```java
@Test 
public void whenInstantiatedThenMongoCollectionHasNameGame() {
  TicTacToeCollection collection = new TicTacToeCollection(); 
  assertEquals( 
            "game", 
            collection.getMongoCollection().getName()); 
} 
```

这个规范几乎与上一个规范相同，可能是不言自明的。

# 实施

我们要做的就是改变我们用来设置集合名称的字符串：

```java
public TicTacToeCollection() throws UnknownHostException { 
  DB db = new MongoClient().getDB("tic-tac-toe"); 
  mongoCollection = new Jongo(db).getCollection("game"); 
} 
```

# 重构

您可能会有这样的印象，即重构仅适用于实现代码。然而，当我们看重构背后的目标（更易读、更优化和更快的代码）时，它们与规范代码一样适用于实现代码。

最后两个规范重复了`TicTacToeCollection`类的实例化。我们可以将其移动到一个用`@Before`注释的方法中。效果将是相同的（在运行用`@Test`注释的方法之前，类将被实例化），并且我们将删除重复的代码。由于在后续规范中将需要相同的实例化，现在删除重复将在以后提供更多的好处。同时，我们将免去一遍又一遍地抛出`UnknownHostException`的麻烦：

```java
TicTacToeCollection collection; 

@Before 
public void before() throws UnknownHostException { 
  collection = new TicTacToeCollection(); 
} 
@Test 
public void whenInstantiatedThenMongoHasDbNameTicTacToe() { 
//        throws UnknownHostException { 
//  TicTacToeCollection collection = new TicTacToeCollection(); 
  assertEquals(
    "tic-tac-toe", 
    collection.getMongoCollection().getDBCollection().getDB().getName()); 
} 

@Test 
public void whenInstantiatedThenMongoHasNameGame() { 
//        throws UnknownHostException { 
//  TicTacToeCollection collection = new TicTacToeCollection(); 
  assertEquals(
    "game",  
    collection.getMongoCollection().getName()); 
} 
```

使用设置和拆卸方法。这些方法的好处允许在类或每个测试方法之前和之后执行准备或设置和处理或拆卸代码。

在许多情况下，一些代码需要在测试类或类中的每个方法之前执行。为此，JUnit 有`@BeforeClass`和`@Before`注解，应该在设置阶段使用。`@BeforeClass`在类加载之前（在第一个测试方法运行之前）执行相关方法。`@Before`在每次测试运行之前执行相关方法。当测试需要特定的前提条件时，应该使用这两个。最常见的例子是在（希望是内存中的）数据库中设置测试数据。相反的是`@After`和`@AfterClass`注解，应该用作拆卸阶段。它们的主要目的是销毁在设置阶段或测试本身创建的数据或状态。每个测试应该独立于其他测试。此外，没有测试应该受到其他测试的影响。拆卸阶段有助于保持系统，就好像以前没有执行任何测试一样。

现在让我们进行一些模拟、监听和验证！

# 规范-向 Mongo 集合添加项目

我们应该创建一个保存数据到 MongoDB 的方法。在研究 Jongo 文档之后，我们发现了`MongoCollection.save`方法，它正是我们需要的。它接受任何对象作为方法参数，并将其（使用 Jackson）转换为 JSON，这在 MongoDB 中是原生使用的。重点是，在使用 Jongo 玩耍后，我们决定使用并且更重要的是信任这个库。

我们可以以两种方式编写 Mongo 规范。一种更传统的方式，适合**端到端**（E2E）或集成测试，是启动一个 MongoDB 实例，调用 Jongo 的保存方法，查询数据库，并确认数据确实已经保存。这还没有结束，因为我们需要在每个测试之前清理数据库，以始终保证相同的状态不受先前测试的执行而污染。最后，一旦所有测试执行完毕，我们可能希望停止 MongoDB 实例，并为其他任务释放服务器资源。

你可能已经猜到，以这种方式编写单个测试涉及相当多的工作。而且，不仅仅是需要投入编写这些测试的工作。执行时间会大大增加。运行一个与数据库通信的测试不需要很长时间。通常运行十个测试仍然很快。运行数百或数千个测试可能需要很长时间。当运行所有单元测试需要很长时间时会发生什么？人们会失去耐心，开始将它们分成组，或者完全放弃 TDD。将测试分成组意味着我们失去了对没有任何东西被破坏的信心，因为我们不断地只测试它的部分。放弃 TDD...好吧，这不是我们试图实现的目标。然而，如果运行测试需要很长时间，可以合理地期望开发人员不愿意等待它们完成后再转移到下一个规范，这就是我们停止进行 TDD 的时候。允许我们的单元测试运行的合理时间是多久？没有一个适用于所有的规则来定义这一点；然而，作为一个经验法则，如果时间超过 10-15 秒，我们应该开始担心，并且花时间来优化它们。

测试应该快速运行。好处是测试经常被使用。

如果运行测试需要很长时间，开发人员将停止使用它们，或者只运行与他们正在进行的更改相关的一个小子集。快速测试的一个好处，除了促进它们的使用，就是快速反馈。问题被检测到的越早，修复起来就越容易。对产生问题的代码的了解仍然很新鲜。如果开发人员在等待测试执行完成时已经开始了下一个功能的工作，他们可能会决定推迟修复问题，直到开发了新功能。另一方面，如果他们放弃当前的工作来修复错误，那么在上下文切换中就会浪费时间。

如果使用实时数据库来运行单元测试不是一个好选择，那么还有什么选择？模拟和监视！在我们的例子中，我们知道应该调用第三方库的哪个方法。我们还投入了足够的时间来信任这个库（除了以后要执行的集成测试）。一旦我们知道如何使用这个库，我们就可以将我们的工作限制在验证该库的正确调用上。

让我们试一试。

首先，我们应该修改我们现有的代码，并将我们对`TicTacToeCollection`的实例化转换为`spy`：

```java
import static org.mockito.Mockito.*; 
... 
@Before 
public void before() throws UnknownHostException { 
  collection = spy(new TicTacToeCollection()); 
} 
```

对一个类进行**部分**模拟被称为监视。应用后，该类的行为将与正常实例化时完全相同。主要区别在于我们可以应用部分模拟并用模拟替换一个或多个方法。一般规则是，我们倾向于在我们正在工作的类上使用监视。我们希望保留我们为其编写规范的类的所有功能，但在需要时，可以模拟其中的一部分。

现在让我们编写规范本身。它可能是以下内容：

```java
@Test
public void whenSaveMoveThenInvokeMongoCollectionSave() {
  TicTacToeBean bean = new TicTacToeBean(3, 2, 1, 'Y');
  MongoCollection mongoCollection = mock(MongoCollection.class);
  doReturn(mongoCollection).when(collection).getMongoCollection();

  collection.saveMove(bean);

  verify(mongoCollection, times(1)).save(bean);
}
```

静态方法，比如`mock`、`doReturn`和`verify`，都来自`org.mockito.Mockito`类。

首先，我们创建一个新的`TicTacToeBean`。没有什么特别的。接下来，我们将`MongoCollection`创建为一个`mock`对象。由于我们已经确定，在单元级别工作时，我们希望避免与数据库直接通信，模拟这种依赖关系将为我们提供这种功能。它将把一个真实的类转换成一个模拟的类。对于使用`mongoCollection`的类来说，它看起来像是一个真实的类；然而，在幕后，它的所有方法都是浅层的，实际上并不执行任何操作。这就像覆盖该类并用空方法替换所有方法一样：

```java
MongoCollection mongoCollection = mock(MongoCollection.class);
```

接下来，我们告诉一个模拟的`mongoCollection`应该在我们调用集合监视类的`getMongoCollection`方法时返回。换句话说，我们告诉我们的类使用一个假的集合而不是真实的集合：

```java
doReturn(mongoCollection).when(collection).getMongoCollection(); 
```

然后，我们调用我们正在工作的方法：

```java
collection.saveMove(bean); 
```

最后，我们应该验证`Jongo`库的正确调用是否执行了一次：

```java
verify(mongoCollection, times(1)).save(bean);
```

让我们试着实现这个规范。

# 实现

为了更好地理解我们刚刚编写的规范，让我们只进行部分实现。我们将创建一个空方法`saveMove`。这将允许我们的代码在不实现规范的情况下编译：

```java
public void saveMove(TicTacToeBean bean) { 
} 
```

当我们运行我们的规范（`gradle test`）时，结果如下：

```java
Wanted but not invoked: 
mongoCollection.save(Turn: 3; X: 2; Y: 1; Player: Y); 
```

Mockito 告诉我们，根据我们的规范，我们期望调用`mongoCollection.save`方法，但这个期望没有实现。由于测试仍然失败，我们需要回去完成实现。在 TDD 中最大的罪过之一就是有一个失败的测试然后转移到其他事情上。

在编写新测试之前，所有测试都应该通过。这样做的好处是，它可以保持对一个小单位的工作的关注，并且实现代码（几乎）总是处于工作状态。

有时候在实际实现之前编写多个测试是很诱人的。在其他情况下，开发人员会忽略现有测试检测到的问题，转向新功能。尽可能避免这种情况。在大多数情况下，违反这个规则只会引入技术债务，需要付出利息来偿还。TDD 的一个目标是确保实现代码（几乎）总是按预期工作。一些项目由于压力要达到交付日期或维持预算，违反这个规则并将时间用于新功能，留下与失败测试相关的代码修复以后再做。这些项目通常最终推迟了不可避免的事情。

让我们也修改实现，例如，以下内容：

```java
public void saveMove(TicTacToeBean bean) { 
  getMongoCollection().save(null); 
} 
```

如果我们再次运行我们的规范，结果如下：

```java
Argument(s) are different! Wanted: 
mongoCollection.save(Turn: 3; X: 2; Y: 1; Player: Y); 
```

这一次我们调用了期望的方法，但是我们传递给它的参数并不是我们希望的。在规范中，我们将期望设置为一个 bean（新的`TicTacToeBean(3, 2, 1, 'Y')`），而在实现中，我们传递了 null。不仅如此，Mockito 的验证可以告诉我们是否调用了正确的方法，以及传递给该方法的参数是否正确。

规范的正确实现如下：

```java
public void saveMove(TicTacToeBean bean) { 
  getMongoCollection().save(bean); 
} 
```

这一次所有的规范都应该通过，我们可以愉快地继续下一个。

# 规范-添加操作反馈

让我们将`saveMove`方法的返回类型更改为`boolean`：

```java
@Test 
public void whenSaveMoveThenReturnTrue() {
  TicTacToeBean bean = new TicTacToeBean(3, 2, 1, 'Y');
  MongoCollection mongoCollection = mock(MongoCollection.class);
  doReturn(mongoCollection).when(collection).getMongoCollection();
  assertTrue(collection.saveMove(bean));
}
```

# 实施

这个实现非常直接。我们应该改变方法的返回类型。记住 TDD 的一个规则是使用最简单的解决方案。最简单的解决方案是返回`true`，就像下面的例子一样：

```java
public boolean saveMove(TicTacToeBean bean) {
  getMongoCollection().save(bean);
  return true;
}
```

# 重构

你可能已经注意到最后两个规范有前两行重复。我们可以通过将它们移到用`@Before`注释的方法中来重构规范代码：

```java
TicTacToeCollection collection;
TicTacToeBean bean;
MongoCollection mongoCollection;

@Before
public void before() throws UnknownHostException {
  collection = spy(new TicTacToeCollection());
  bean = new TicTacToeBean(3, 2, 1, 'Y');
  mongoCollection = mock(MongoCollection.class);
} 
... 
@Test
public void whenSaveMoveThenInvokeMongoCollectionSave() {
// TicTacToeBean bean = new TicTacToeBean(3, 2, 1, 'Y'); 
// MongoCollection mongoCollection = mock(MongoCollection.class); 
  doReturn(mongoCollection).when(collection).getMongoCollection(); 
  collection.saveMove(bean); 
  verify(mongoCollection, times(1)).save(bean); 
} 

@Test 
public void whenSaveMoveThenReturnTrue() { 
// TicTacToeBean bean = new TicTacToeBean(3, 2, 1, 'Y'); 
// MongoCollection mongoCollection = mock(MongoCollection.class); 
   doReturn(mongoCollection).when(collection).getMongoCollection(); 
   assertTrue(collection.saveMove(bean)); 
} 
```

# 规范-错误处理

现在让我们考虑一下在使用 MongoDB 时可能出现问题的选项。例如，当抛出异常时，我们可能希望从我们的`saveMove`方法中返回`false`：

```java
@Test
public void givenExceptionWhenSaveMoveThenReturnFalse() {
  doThrow(new MongoException("Bla"))
    .when(mongoCollection).save(any(TicTacToeBean.class));
  doReturn(mongoCollection).when(collection).getMongoCollection();
  assertFalse(collection.saveMove(bean));
} 
```

在这里，我们介绍了另一个 Mockito 方法：`doThrow`。它的作用方式类似于`doReturn`，当设置条件满足时抛出一个`Exception`。规范将在调用`mongoCollection`类内部的`save`方法时抛出`MongoException`。这使我们能够`assert`我们的`saveMove`方法在抛出异常时返回`false`。

# 实施

实现可以简单到添加一个`try`/`catch`块：

```java
public boolean saveMove(TicTacToeBean bean) {
  try {
    getMongoCollection().save(bean);
    return true;
  } catch (Exception e) {
    return false;
  }
}
```

# 规范-在游戏之间清除状态

这是一个非常简单的应用程序，至少在这一刻，它只能存储一个游戏会话。每当创建一个新实例时，我们应该重新开始并删除数据库中存储的所有数据。这样做的最简单方法就是简单地删除 MongoDB 集合。Jongo 有`MongoCollection.drop()`方法可以用于这个目的。我们将创建一个新的方法`drop`，它将以类似于`saveMove`的方式工作。

如果你没有使用 Mockito、MongoDB 和/或 Jongo 工作过，那么你可能无法自己完成本章的练习，只能决定按照我们提供的解决方案进行。如果是这种情况，那么现在可能是你想要改变方向，尝试自己编写规范和实现的时候了。

我们应该验证`MongoCollection.drop()`是否从我们自己的`drop()`方法内部的`TicTacToeCollection`类中调用。在查看以下代码之前，请自己尝试一下。这几乎与我们对`save`方法所做的事情相同：

```java
@Test
public void whenDropThenInvokeMongoCollectionDrop() {
  doReturn(mongoCollection).when(collection).getMongoCollection();
  collection.drop();
  verify(mongoCollection).drop();
}
```

# 实施

由于这是一个包装方法，实现这个规范应该相当容易：

```java
public void drop() { 
  getMongoCollection().drop(); 
} 
```

# 规范-删除操作反馈

我们几乎完成了这个类。只剩下两个规范。

让我们确保在正常情况下返回`true`：

```java
@Test 
public void whenDropThenReturnTrue() { 
  doReturn(mongoCollection).when(collection).getMongoCollection();
  assertTrue(collection.drop()); 
}
```

# 实施

如果使用 TDD 看起来太容易了，那是有意为之的。我们将任务分解成如此小的实体，以至于在大多数情况下，实现规范都是小菜一碟。这个也不例外：

```java
public boolean drop() { 
  getMongoCollection().drop(); 
  return true; 
} 
```

# 规范-错误处理

最后，让我们确保`drop`方法在出现`异常`时返回`false`：

```java
@Test 
public void givenExceptionWhenDropThenReturnFalse() {
  doThrow(new MongoException("Bla")).when(mongoCollection).drop(); 
  doReturn(mongoCollection).when(collection).getMongoCollection(); 
  assertFalse(collection.drop()); 
} 
```

# 实施

让我们添加一个`try`/`catch`块：

```java
public boolean drop() { 
  try { 
    getMongoCollection().drop();
    return true; 
  } catch (Exception e) {
    return false; 
  } 
} 
```

通过这个实现，我们完成了`TicTacToeCollection`类，它充当了我们的`main`类和 MongoDB 之间的层。

源代码可以在`tdd-java-ch06-tic-tac-toe-mongo` Git 存储库的`02-save-move`分支中找到（[`bitbucket.org/vfarcic/tdd-java-ch06-tic-tac-toe-mongo/branch/02-save-move`](https://bitbucket.org/vfarcic/tdd-java-ch06-tic-tac-toe-mongo/branch/02-save-move)）。特别的类是`TicTacToeCollectionSpec`和`TicTacToeCollection`。

# 需求 2-存储每一步

让我们在我们的主类`TicTacToe`中使用`TicTacToeCollection`方法。每当玩家成功玩一个回合时，我们应该将其保存到数据库中。此外，我们应该在实例化新类时删除集合，以防新游戏与旧游戏重叠。我们可以把它做得更复杂；然而，对于本章的目的和学习如何使用模拟，这个要求现在就足够了。

将每一步保存到数据库，并确保新会话清除旧数据。

让我们先做一些设置。

# 规范-创建新的集合

由于我们将用于与 MongoDB 通信的所有方法都在`TicTacToeCollection`类中，我们应该确保它被实例化。规范可能如下：

```java
@Test 
public void whenInstantiatedThenSetCollection() {
  assertNotNull(ticTacToe.getTicTacToeCollection());
} 
```

`TicTacToe`的实例化已经在用`@Before`注解的方法中完成了。通过这个规范，我们确保集合也被实例化。

# 实施

这个实现没有什么特别之处。我们应该简单地重写默认构造函数，并将一个新实例分配给`ticTacToeCollection`变量。

首先，我们应该添加一个本地变量和一个`TicTacToeCollection`的 getter：

```java
private TicTacToeCollection ticTacToeCollection;

protected TicTacToeCollection getTicTacToeCollection() {
  return ticTacToeCollection;
} 
```

现在剩下的就是实例化一个新的`collection`并在`main`类实例化时将其分配给变量：

```java
public TicTacToe() throws UnknownHostException {
  this(new TicTacToeCollection()); 
}
protected TicTacToe(TicTacToeCollection collection) {
  ticTacToeCollection = collection; 
} 
```

我们还创建了另一种通过传递`TicTacToeCollection`作为参数来实例化类的方法。这在规范中作为传递模拟集合的简单方法会很方便。

现在让我们回到规范类，并利用这个新的构造函数。

# 规范重构

为了利用新创建的`TicTacToe`构造函数，我们可以做一些类似以下的事情：

```java
private TicTacToeCollection collection; 

@Before 
public final void before() throws UnknownHostException {
  collection = mock(TicTacToeCollection.class);
// ticTacToe = new TicTacToe();
  ticTacToe = new TicTacToe(collection);
} 
```

现在我们所有的规范都将使用`TicTacToeCollection`的模拟版本。还有其他注入模拟依赖的方法（例如，使用 Spring）；然而，可能的话，我们觉得简单胜过复杂的框架。

# 规范-存储当前移动

每当我们玩一个回合，它都应该保存到数据库中。规范可以是以下内容：

```java
@Test 
public void whenPlayThenSaveMoveIsInvoked() {
  TicTacToeBean move = new TicTacToeBean(1, 1, 3, 'X');
  ticTacToe.play(move.getX(), move.getY());
  verify(collection).saveMove(move);
}
```

到目前为止，你应该对 Mockito 很熟悉了，但让我们通过代码来复习一下：

1.  首先，我们实例化一个`TicTacToeBean`，因为它包含了我们的集合所期望的数据：

```java
TicTacToeBean move = new TicTacToeBean(1, 1, 3, 'X'); 
```

1.  接下来，是时候玩一个真正的回合了：

```java
ticTacToe.play(move.getX(), move.getY()); 
```

1.  最后，我们需要验证`saveMove`方法是否真的被调用了：

```java
verify(collection, times(1)).saveMove(move); 
```

正如我们在本章中所做的那样，我们隔离了所有外部调用，只专注于我们正在处理的单元(`play`)。请记住，这种隔离仅限于公共和受保护的方法。当涉及到实际的实现时，我们可能选择将`saveMove`调用添加到`play`公共方法或我们之前重构的一个私有方法中。

# 实施

这个规范提出了一些挑战。首先，我们应该在哪里调用`saveMove`方法？`setBox`私有方法看起来是一个不错的地方。那里我们正在验证轮次是否有效，如果有效，我们可以调用`saveMove`方法。然而，该方法期望一个`bean`而不是当前正在使用的变量`x`，`y`和`lastPlayer`，所以我们可能需要更改`setBox`方法的签名。

这是该方法现在的样子：

```java
private void setBox(int x, int y, char lastPlayer) {
  if (board[x - 1][y - 1] != '\0') {
    throw new RuntimeException("Box is occupied");
  } else {
    board[x - 1][y - 1] = lastPlayer;
  }
}
```

这是在必要的更改应用后的外观：

```java
private void setBox(TicTacToeBean bean) {
  if (board[bean.getX() - 1][bean.getY() - 1] != '\0') {
    throw new RuntimeException("Box is occupied");
  } else {
    board[bean.getX() - 1][bean.getY() - 1] = lastPlayer;
    getTicTacToeCollection().saveMove(bean);
  }
}
```

`setBox`签名的更改触发了一些其他更改。由于它是从`play`方法中调用的，我们需要在那里实例化`bean`：

```java
public String play(int x, int y) {
  checkAxis(x);
  checkAxis(y);
  lastPlayer = nextPlayer();
// setBox(x, y, lastPlayer);
  setBox(new TicTacToeBean(1, x, y, lastPlayer));
  if (isWin(x, y)) {
    return lastPlayer + " is the winner";
  } else if (isDraw()) {
    return RESULT_DRAW;
  } else {
    return NO_WINNER;
  }
}
```

您可能已经注意到我们使用常量值`1`作为轮次。仍然没有规范表明否则，所以我们采取了一种捷径。我们以后再处理它。

所有这些更改仍然非常简单，并且实施它们所花费的时间相当短。如果更改更大，我们可能会选择不同的路径；并进行简单的更改以通过重构最终解决方案。记住速度是关键。您不希望长时间无法通过测试的实现。

# 规范-错误处理

如果移动无法保存会发生什么？我们的辅助方法`saveMove`根据 MongoDB 操作结果返回`true`或`false`。当它返回`false`时，我们可能希望抛出异常。

首先：我们应该更改`before`方法的实现，并确保默认情况下`saveMove`返回`true`：

```java
@Before
public final void before() throws UnknownHostException {
  collection = mock(TicTacToeCollection.class);
  doReturn(true).when(collection).saveMove(any(TicTacToeBean.class));
  ticTacToe = new TicTacToe(collection);
}
```

现在我们已经用我们认为是默认行为（在调用`saveMove`时返回`true`）对模拟集合进行了存根处理，我们可以继续编写规范：

```java
@Test
public void whenPlayAndSaveReturnsFalseThenThrowException() {
  doReturn(false).when(collection).saveMove(any(TicTacToeBean.class));
  TicTacToeBean move = new TicTacToeBean(1, 1, 3, 'X');
  exception.expect(RuntimeException.class);
  ticTacToe.play(move.getX(), move.getY());
}
```

当调用`saveMove`时，我们使用 Mockito 返回`false`。在这种情况下，我们不关心`saveMove`的特定调用，所以我们使用`any(TicTacToeBean.class)`作为方法参数。这是 Mockito 的另一个静态方法。

一切就绪后，我们将像在第三章中一样使用 JUnit 期望，*从失败到成功再到完美的红绿重构*。

# 实施

让我们做一个简单的`if`，当结果不符合预期时抛出`RuntimeException`：

```java
private void setBox(TicTacToeBean bean) {
  if (board[bean.getX() - 1][bean.getY() - 1] != '\0') {
    throw new RuntimeException("Box is occupied");
  } else {
    board[bean.getX() - 1][bean.getY() - 1] = lastPlayer;
//  getTicTacToeCollection().saveMove(bean);
    if (!getTicTacToeCollection().saveMove(bean)) {
      throw new RuntimeException("Saving to DB failed");
    }
  }
}
```

# 规范-交替玩家

您还记得我们硬编码为始终为`1`的轮次吗？让我们修复这个行为。

我们可以调用`play`方法两次并验证轮次从`1`变为`2`：

```java
@Test 
public void whenPlayInvokedMultipleTimesThenTurnIncreases() {
  TicTacToeBean move1 = new TicTacToeBean(1, 1, 1, 'X'); 
  ticTacToe.play(move1.getX(), move1.getY()); 
  verify(collection, times(1)).saveMove(move1);
  TicTacToeBean move2 = new TicTacToeBean(2, 1, 2, 'O'); 
  ticTacToe.play(move2.getX(), move2.getY()); 
  verify(collection, times(1)).saveMove(move2); 
} 
```

# 实施

与几乎所有其他以 TDD 方式完成的工作一样，实施起来相当容易：

```java
private int turn = 0;
...
public String play(int x, int y) {
  checkAxis(x);
  checkAxis(y);
  lastPlayer = nextPlayer();
  setBox(new TicTacToeBean(++turn, x, y, lastPlayer));
  if (isWin(x, y)) {
    return lastPlayer + " is the winner";
  } else if (isDraw()) {
    return RESULT_DRAW;
  } else {
    return NO_WINNER;
  }
}
```

# 练习

还有一些规范及其实施尚未完成。我们应该在我们的`TicTacToe`类实例化时调用`drop()`方法。我们还应该确保在`drop()`返回`false`时抛出`RuntimeException`。我们将把这些规范及其实施留给您作为练习。

源代码可以在`tdd-java-ch06-tic-tac-toe-mongo` Git 存储库的`03-mongo`分支中找到（[`bitbucket.org/vfarcic/tdd-java-ch06-tic-tac-toe-mongo/branch/03-mongo`](https://bitbucket.org/vfarcic/tdd-java-ch06-tic-tac-toe-mongo/branch/03-mongo)）。特别的类是`TicTacToeSpec`和`TicTacToe`。

# 集成测试

我们做了很多单元测试。我们非常依赖信任。一个接一个地指定和实现单元。在编写规范时，我们隔离了除了我们正在处理的单元之外的一切，并验证一个单元是否正确调用了另一个单元。然而，现在是时候验证所有这些单元是否真的能够与 MongoDB 通信了。我们可能犯了一个错误，或者更重要的是，我们可能没有将 MongoDB 启动和运行。发现，例如，我们部署了我们的应用程序，但忘记启动数据库，或者配置（IP、端口等）没有设置正确，这将是一场灾难。

集成测试的目标是验证，正如你可能已经猜到的那样，独立组件、应用程序、系统等的集成。如果你记得测试金字塔，它指出单元测试是最容易编写和最快运行的，因此我们应该将其他类型的测试限制在单元测试未覆盖的范围内。

我们应该以一种可以偶尔运行的方式隔离我们的集成测试（在将代码推送到存储库之前，或作为我们的持续集成（CI）过程的一部分），并将单元测试作为持续反馈循环。

# 测试分离

如果我们遵循某种约定，那么在 Gradle 中分离测试就会相当容易。我们可以将测试放在不同的目录和不同的包中，或者，例如，使用不同的文件后缀。在这种情况下，我们选择了后者。我们所有的规范类都以`Spec`后缀命名（即`TicTacToeSpec`）。我们可以制定一个规则，即所有集成测试都具有`Integ`后缀。

考虑到这一点，让我们修改我们的`build.gradle`文件。

首先，我们将告诉 Gradle 只有以`Spec`结尾的类才应该被`test`任务使用：

```java
test { 
    include '**/*Spec.class' 
} 
```

接下来，我们可以创建一个新任务`testInteg`：

```java
task testInteg(type: Test) { 
    include '**/*Integ.class' 
} 
```

通过这两个对`build.gradle`的添加，我们继续使用本书中大量使用的测试任务；然而，这一次，它们仅限于规范（单元测试）。此外，所有集成测试都可以通过从 Gradle 项目 IDEA 窗口点击`testInteg`任务或从命令提示符运行以下命令来运行：

```java
gradle testInteg

```

让我们写一个简单的集成测试。

# 集成测试

我们将在`src/test/java`目录中的`com.packtpublishing.tddjava.ch03tictactoe`包内创建一个`TicTacToeInteg`类。由于我们知道 Jongo 如果无法连接到数据库会抛出异常，所以测试类可以简单如下：

```java
import org.junit.Test;
import java.net.UnknownHostException;
import static org.junit.Assert.*;

public class TicTacToeInteg {

  @Test
  public void givenMongoDbIsRunningWhenPlayThenNoException()
        throws UnknownHostException {
    TicTacToe ticTacToe = new TicTacToe();
    assertEquals(TicTacToe.NO_WINNER, ticTacToe.play(1, 1));
  }
}
```

`assertEquals`的调用只是作为一种预防措施。这个测试的真正目的是确保没有抛出`Exception`。由于我们没有启动 MongoDB（除非你非常主动并且自己启动了它，在这种情况下你应该停止它），`test`应该失败：

![](img/43e8e2b6-439a-45e2-ac2a-206f307010a9.png)

现在我们知道集成测试是有效的，或者换句话说，当 MongoDB 没有启动和运行时，它确实会失败，让我们再次尝试一下，看看数据库启动后的情况。为了启动 MongoDB，我们将使用 Vagrant 创建一个带有 Ubuntu 操作系统的虚拟机。MongoDB 将作为 Docker 运行。

确保检出了 04-integration 分支：

![](img/c0e5e0fa-4005-4021-9cd1-e2dc393f35fe.png)

从命令提示符运行以下命令：

```java
$ vagrant up

```

请耐心等待 VM 启动和运行（当第一次执行时可能需要一段时间，特别是在较慢的带宽上）。完成后，重新运行集成测试：

![](img/60c4a316-d999-4143-9697-359e4df6e661.png)

它起作用了，现在我们确信我们确实与 MongoDB 集成了。

这是一个非常简单的集成测试，在现实世界中，我们会做更多的工作而不仅仅是这一个测试。例如，我们可以查询数据库并确认数据是否被正确存储。然而，本章的目的是学习如何模拟以及我们不应该仅依赖单元测试。下一章将更深入地探讨集成和功能测试。

源代码可以在`tdd-java-ch06-tic-tac-toe-mongo` Git 存储库的`04-integration`分支中找到（[`bitbucket.org/vfarcic/tdd-java-ch06-tic-tac-toe-mongo/branch/04-integration`](https://bitbucket.org/vfarcic/tdd-java-ch06-tic-tac-toe-mongo/branch/04-integration)）。

# 总结

模拟和间谍技术被用来隔离代码或第三方库的不同部分。它们是必不可少的，如果我们要以极快的速度进行，不仅在编码时，而且在运行测试时也是如此。没有模拟的测试通常太复杂，写起来很慢，随着时间的推移，TDD 往往变得几乎不可能。慢速测试意味着我们将无法在每次编写新规范时运行所有测试。这本身就导致我们对测试的信心下降，因为只有其中的一部分被运行。

模拟不仅作为隔离外部依赖的一种方式，还作为隔离我们自己正在处理的单元的一种方式。

在本章中，我们将 Mockito 作为我们认为在功能和易用性之间具有最佳平衡的框架进行介绍。我们邀请您更详细地调查其文档（[`mockito.org/`](http://mockito.org/)），以及其他专门用于模拟的 Java 框架。EasyMock（[`easymock.org/`](http://easymock.org/)）、JMock（[`www.jmock.org/`](http://www.jmock.org/)）和 PowerMock（[`code.google.com/p/powermock/`](https://code.google.com/p/powermock/)）是一些最受欢迎的框架。

在下一章中，我们将介绍一些函数式编程概念以及应用于它们的一些 TDD 概念。为此，将介绍 Java 函数式 API 的一部分。
