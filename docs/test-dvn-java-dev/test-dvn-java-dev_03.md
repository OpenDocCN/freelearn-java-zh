# 第三章：红绿重构——从失败到成功直至完美

“知道不足以;我们必须应用。愿意不足以;我们必须去做。”

- 李小龙

**红绿重构**技术是**测试驱动开发**（**TDD**）的基础。这是一个乒乓球游戏，在这个游戏中，我们以很快的速度在测试和实现代码之间切换。我们会失败，然后我们会成功，最后，我们会改进。

我们将通过逐个满足每个需求来开发一个井字棋游戏。我们将编写一个测试并查看是否失败。然后，我们将编写实现该测试的代码，运行所有测试，并看到它们成功。最后，我们将重构代码并尝试使其更好。这个过程将重复多次，直到所有需求都成功实现。

我们将从使用 Gradle 和 JUnit 设置环境开始。然后，我们将深入了解红绿重构过程。一旦我们准备好设置和理论，我们将通过应用的高级需求。

一切准备就绪后，我们将立即进入代码——逐个需求。一切都完成后，我们将查看代码覆盖率，并决定是否可以接受，或者是否需要添加更多测试。

本章将涵盖以下主题：

+   使用 Gradle 和 JUnit 设置环境

+   红绿重构过程

+   井字棋的需求

+   开发井字棋

+   代码覆盖率

+   更多练习

# 使用 Gradle 和 JUnit 设置环境

您可能熟悉 Java 项目的设置。但是，您可能以前没有使用过 IntelliJ IDEA，或者您可能使用的是 Maven 而不是 Gradle。为了确保您能够跟上练习，我们将快速浏览一下设置。

# 在 IntelliJ IDEA 中设置 Gradle/Java 项目

本书的主要目的是教授 TDD，因此我们不会详细介绍 Gradle 和 IntelliJ IDEA。两者都是作为示例使用的。本书中的所有练习都可以使用不同的 IDE 和构建工具来完成。例如，您可以使用 Maven 和 Eclipse。对于大多数人来说，遵循本书中提出的相同指南可能更容易，但选择权在您手中。

以下步骤将在 IntelliJ IDEA 中创建一个新的 Gradle 项目：

1.  打开 IntelliJ IDEA。单击创建新项目，然后从左侧菜单中选择 Gradle。然后，单击下一步。

1.  如果您使用的是 IDEA 14 及更高版本，则会要求您输入 Artifact ID。键入`tdd-java-ch03-tic-tac-toe`，然后单击两次“下一步”。将`tdd-java-ch03-tic-tac-toe`输入为项目名称。然后，单击“完成”按钮：

![](img/5fd91dbe-7381-43e8-a2c3-c64d6b955ea6.png)

在新项目对话框中，我们可以观察到 IDEA 已经创建了`build.gradle`文件。打开它，你会看到它已经包含了 JUnit 依赖项。由于这是本章中我们选择的框架，因此我们不需要进行额外的配置。默认情况下，`build.gradle`设置为使用 Java 1.5 作为源兼容性设置。您可以将其更改为任何您喜欢的版本。本章的示例不会使用 Java 5 版本之后的任何功能，但这并不意味着您不能使用其他版本，例如 JDK 8 来解决练习。

我们的`build.gradle`文件应该如下所示：

```java
apply plugin: 'java' 

version = '1.0' 

repositories { 
  mavenCentral()
} 

dependencies { 
  testCompile group: 'junit', name: 'junit', version: '4.11' 
} 
```

现在，剩下的就是创建我们将用于测试和实现的包。从项目对话框中，右键单击以弹出上下文菜单，然后选择 New|Directory。键入`src/test/java/com/packtpublishing/tddjava/ch03tictactoe`，然后单击“确定”按钮以创建测试包。重复相同的步骤，使用`src/main/java/com/packtpublishing/tddjava/ch03tictactoe`目录创建实现包。

最后，我们需要创建测试和实现类。在`src/test/java`目录中的`com.packtpublishing.tddjava.ch03tictactoe`包内创建`TicTacToeSpec`类。这个类将包含所有我们的测试。在`src/main/java`目录中的`TicTacToe`类中重复相同的操作。

你的项目结构应该类似于以下截图中呈现的结构：

![](img/862be672-6457-43c7-b2e0-bad2d7b0925b.png)

源代码可以在`00-setup`分支的`tdd-java-ch03-tic-tac-toe` Git 存储库中找到，网址为[`bitbucket.org/vfarcic/tdd-java-ch03-tic-tac-toe/branch/00-setup`](https://bitbucket.org/vfarcic/tdd-java-ch03-tic-tac-toe/branch/00-setup)。

始终将测试与实现代码分开。

好处如下：这样可以避免意外地将测试与生产二进制文件打包在一起；许多构建工具期望测试位于特定的源目录中。

一个常见的做法是至少有两个源目录。实现代码应该位于`src/main/java`，测试代码位于`src/test/java`。在更大的项目中，源目录的数量可能会增加，但实现和测试之间的分离应该保持不变。

Maven 和 Gradle 等构建工具期望源目录、分离以及命名约定。

就是这样。我们准备使用 JUnit 作为首选的测试框架，使用 Gradle 进行编译、依赖、测试和其他任务，开始开发我们的井字游戏应用程序。在第一章中，*为什么我应该关心测试驱动开发？*，你首次遇到了红-绿-重构过程。由于它是 TDD 的基石，并且是本章练习的主要目标，可能是一个好主意在开始开发之前更详细地了解一下。

# 红-绿-重构过程

红-绿-重构过程是 TDD 的最重要部分。这是主要支柱，没有它，TDD 的其他方面都无法运行。

名称来自于代码在循环中所处的状态。在红色状态下，代码不起作用；在绿色状态下，一切都按预期工作，但不一定是最佳方式。重构是一个阶段，我们知道功能已经得到了充分的测试覆盖，因此我们有信心对其进行更改并使其更好。

# 编写测试

每个新功能都以测试开始。这个测试的主要目标是在编写代码之前专注于需求和代码设计。测试是一种可执行文档，以后可以用来理解代码的功能或意图。

此时，我们处于红色状态，因为测试执行失败。测试对代码的期望与实现代码实际执行的不一致。更具体地说，没有代码满足最后一个测试的期望；我们还没有编写它。在这个阶段，所有测试实际上都通过了，但这是一个问题的迹象。

# 运行所有测试，并确认最后一个测试失败

确认最后一个测试失败，确认测试不会误以为没有引入新代码而通过。如果测试通过，那么该功能已经存在，或者测试产生了错误的积极结果。如果是这种情况，测试实际上总是独立于实现而通过，那么它本身是毫无价值的，应该被移除。

测试不仅必须失败，而且必须因为预期的原因而失败。在这个阶段，我们仍然处于红色阶段。测试已经运行，最后一个测试失败了。

# 编写实现代码

这个阶段的目的是编写代码，使最后一个测试通过。不要试图让它完美，也不要花太多时间。如果它写得不好或者不是最佳的，那也没关系。以后会变得更好。我们真正想做的是创建一种以测试形式确认通过的安全网。不要试图引入任何上一个测试中没有描述的功能。要做到这一点，我们需要回到第一步，从新的测试开始。然而，在所有现有测试都通过之前，我们不应该编写新的测试。

在这个阶段，我们仍处于红色阶段。虽然编写的代码可能会通过所有测试，但这个假设尚未得到确认。

# 运行所有测试

非常重要的是运行所有的测试，而不仅仅是最后编写的测试。我们刚刚编写的代码可能使最后一个测试通过，同时破坏了其他东西。运行所有的测试不仅确认了最后一个测试的实现是正确的，而且确认了它没有破坏整个应用程序的完整性。整个测试套件的缓慢执行表明测试编写不好或者代码耦合度太高。耦合阻止了外部依赖的轻松隔离，从而增加了测试执行所需的时间。

在这个阶段，我们处于绿色状态。所有的测试都通过了，应用程序的行为符合我们的预期。

# 重构

虽然所有之前的步骤都是强制性的，但这一步是可选的。尽管重构很少在每个周期结束时进行，但迟早会被期望，如果不是强制的。并不是每个测试的实现都需要重构。没有规则告诉你何时重构何时不重构。最佳时间是一旦有一种感觉，代码可以以更好或更优的方式重写时。

什么构成重构的候选？这是一个难以回答的问题，因为它可能有很多答案——难以理解的代码、代码片段的不合理位置、重复、名称不清晰的目的、长方法、做太多事情的类等等。列表可以继续下去。无论原因是什么，最重要的规则是重构不能改变任何现有功能。

# 重复

一旦所有步骤（重构是可选的）完成，我们就重复它们。乍一看，整个过程可能看起来太长或太复杂，但实际上并不是。有经验的 TDD 从业者在切换到下一步之前写一到十行代码。整个周期应该持续几秒钟到几分钟。如果时间超过这个范围，测试的范围就太大，应该分成更小的块。快速失败，纠正，重复。

有了这些知识，让我们通过使用红-绿-重构过程开发的应用程序的要求。

# 井字游戏要求

井字游戏通常由年幼的孩子玩。游戏规则相当简单。

井字游戏是一种纸笔游戏，供两名玩家*X*和*O*轮流在 3×3 的网格中标记空格。成功在水平、垂直或对角线上放置三个相应标记的玩家获胜。

有关游戏的更多信息，请访问维基百科（[`en.wikipedia.org/wiki/Tic-tac-toe`](http://en.wikipedia.org/wiki/Tic-tac-toe)）。

更详细的要求将在以后提出。

这个练习包括创建一个与需求相对应的单个测试。测试后面是满足该测试期望的代码。最后，如果需要，对代码进行重构。应该重复相同的过程，直到满意为止，然后转移到下一个需求，直到所有需求都完成。

在现实世界的情况下，您不会得到如此详细的要求，但是可以直接进行既是要求又是验证的测试。然而，在您熟悉 TDD 之前，我们必须将需求与测试分开定义。

尽管所有的测试和实现都已经提供，但请一次只阅读一个需求，并自己编写测试和实现代码。完成后，将您的解决方案与本书中的解决方案进行比较，然后转到下一个需求。没有唯一的解决方案；您的解决方案可能比这里提供的更好。

# 开发井字棋

您准备好编码了吗？让我们从第一个需求开始。

# 需求 1-放置棋子

我们应该首先定义边界和什么构成了一个棋子的无效放置。

一个棋子可以放在 3×3 棋盘的任何空位上。

我们可以将这个需求分成三个测试：

+   当一个棋子被放置在*x*轴之外的任何地方，就会抛出`RuntimeException`

+   当一个棋子被放置在*y*轴之外的任何地方，就会抛出`RuntimeException`

+   当一个棋子被放在一个已占用的空间上时，就会抛出`RuntimeException`

正如您所看到的，与第一个需求相关的测试都是关于验证输入参数的。在需求中没有提到应该对这些棋子做什么。

在我们进行第一个测试之前，有必要简要解释一下如何使用 JUnit 测试异常。

从 4.7 版本开始，JUnit 引入了一个名为`Rule`的功能。它可以用于执行许多不同的操作（更多信息可以在[`github.com/junit-team/junit/wiki/Rules`](https://github.com/junit-team/junit/wiki/Rules)找到），但在我们的情况下，我们对`ExpectedException`规则感兴趣：

```java
public class FooTest {
  @Rule
  public ExpectedException exception = ExpectedException.none();
  @Test
  public void whenDoFooThenThrowRuntimeException() {
    Foo foo = new Foo();
    exception.expect(RuntimeException.class);
    foo.doFoo();
  }
} 
```

在这个例子中，我们定义了`ExpectedException`是一个规则。稍后在`doFooThrowsRuntimeException`测试中，我们指定我们期望在`Foo`类实例化后抛出`RuntimeException`。如果在之前抛出，测试将失败。如果在之后抛出，测试就成功了。

`@Before`可以用来注释一个在每个测试之前运行的方法。这是一个非常有用的功能，例如我们可以实例化一个在测试中使用的类，或者执行一些其他类型的在每个测试之前运行的操作：

```java
private Foo foo; 

@Before 
public final void before() { 
  foo = new Foo(); 
} 
```

在这个例子中，`Foo`类将在每个测试之前实例化。这样，我们就可以避免在每个测试方法中实例化`Foo`的重复代码。

每个测试都应该用`@Test`进行注释。这告诉`JunitRunner`哪些方法构成测试。它们中的每一个都将以随机顺序运行，所以确保每个测试都是自给自足的，并且不依赖于其他测试可能创建的状态：

```java
@Test 
public void whenSomethingThenResultIsSomethingElse() { 
  // This is a test method 
} 
```

有了这个知识，您应该能够编写您的第一个测试，并跟随着实现。完成后，将其与提供的解决方案进行比较。

为测试方法使用描述性的名称。

其中一个好处是它有助于理解测试的目标。

在尝试弄清楚为什么一些测试失败或者测试覆盖率应该增加更多测试时，使用描述测试的方法名称是有益的。在测试之前应该清楚地设置条件，执行什么操作，以及预期的结果是什么。

有许多不同的方法来命名测试方法。我偏好的方法是使用 BDD 场景中使用的给定/当/那么语法来命名它们。`给定`描述（前）条件，`当`描述动作，`那么`描述预期结果。如果一个测试没有前提条件（通常使用`@Before`和`@BeforeClass`注解设置），`给定`可以被省略。

不要仅依靠注释提供有关测试目标的信息。注释在从您喜爱的 IDE 执行测试时不会出现，也不会出现在 CI 或构建工具生成的报告中。

除了编写测试，你还需要运行它们。由于我们使用 Gradle，它们可以从命令提示符中运行：

```java
    $ gradle test
```

IntelliJ IDEA 提供了一个非常好的 Gradle 任务模型，可以通过点击 View|Tool Windows|Gradle 来访问。它列出了所有可以使用 Gradle 运行的任务（其中之一就是`test`）。

选择权在你手中-你可以以任何你认为合适的方式运行测试，只要你运行所有测试。

# 测试-板边界 I

我们应该首先检查一个棋子是否放在 3x3 棋盘的边界内：

```java
package com.packtpublishing.tddjava.ch03tictactoe;

import org.junit.Before;
import org.junit.Rule;
import org.junit.Test;
import org.junit.rules.ExpectedException;

public class TicTacToeSpec {
  @Rule
  public ExpectedException exception = ExpectedException.none();
  private TicTacToe ticTacToe;

  @Before
  public final void before() {
    ticTacToe = new TicTacToe();
  }
  @Test
  public void whenXOutsideBoardThenRuntimeException() {
    exception.expect(RuntimeException.class);
    ticTacToe.play(5, 2);
  }
} 
```

当一个棋子被放置在*x*轴之外的任何地方时，会抛出`RuntimeException`。

在这个测试中，我们定义了当调用`ticTacToe.play(5, 2)`方法时，会抛出`RuntimeException`。这是一个非常简短和简单的测试，使其通过也应该很容易。我们所要做的就是创建`play`方法，并确保当`x`参数小于 1 或大于 3（棋盘是 3x3）时抛出`RuntimeException`。你应该运行这个测试三次。第一次，它应该失败，因为`play`方法不存在。一旦添加了它，它应该失败，因为没有抛出`RuntimeException`。第三次，它应该成功，因为与这个测试对应的代码已经完全实现。

# 实施

现在我们清楚了什么时候应该抛出异常，实现应该很简单：

```java
package com.packtpublishing.tddjava.ch03tictactoe;

public class TicTacToe {
  public void play(int x, int y) {
    if (x < 1 || x > 3) {
      throw new RuntimeException("X is outside board");
    }
  }
}
```

正如你所看到的，这段代码除了让测试通过所需的最低限度之外，没有别的东西。

一些 TDD 实践者倾向于将最小化理解为字面意义。他们会让`play`方法只有`throw new RuntimeException();`这一行。我倾向于将最小化理解为在合理范围内尽可能少。

我们不添加数字，也不返回任何东西。这一切都是关于非常快速地进行小的更改。（记住乒乓球游戏吗？）目前，我们正在进行红绿步骤。我们无法做太多来改进这段代码，所以我们跳过了重构。

让我们继续进行下一个测试。

# 测试-板边界 II

这个测试几乎与上一个测试相同。这次我们应该验证*y*轴：

```java
@Test
public void whenYOutsideBoardThenRuntimeException() {
  exception.expect(RuntimeException.class);
  ticTacToe.play(2, 5);
}
```

当一个棋子被放置在*y*轴之外的任何地方时，会抛出`RuntimeException`。

# 实施

这个规范的实现几乎与上一个相同。我们所要做的就是如果`y`不在定义的范围内，则抛出异常：

```java
public void play(int x, int y) {
  if (x < 1 || x > 3) {
    throw new RuntimeException("X is outside board");
  } else if (y < 1 || y > 3) {
    throw new RuntimeException("Y is outside board");
  }
}
```

为了让最后一个测试通过，我们必须添加检查`Y`是否在棋盘内的`else`子句。

让我们为这个要求做最后一个测试。

# 测试-占用的位置

现在我们知道棋子是放在棋盘边界内的，我们应该确保它们只能放在未占用的空间上：

```java
@Test 
public void whenOccupiedThenRuntimeException() { 
  ticTacToe.play(2, 1); 
  exception.expect(RuntimeException.class); 
  ticTacToe.play(2, 1); 
} 
```

当一个棋子被放在一个已占用的空间上时，会抛出`RuntimeException`。

就是这样；这是我们的最后一个测试。一旦实现完成，我们就可以认为第一个要求已经完成了。

# 实施

要实现最后一个测试，我们应该将放置的棋子的位置存储在一个数组中。每次放置一个新的棋子时，我们应该验证该位置是否被占用，否则抛出异常：

```java
private Character[][] board = {
  {'\0', '\0', '\0'},
  {'\0', '\0', '\0'},
  {'\0', '\0', '\0'}
};

public void play(int x, int y) {
  if (x < 1 || x > 3) {
    throw new RuntimeException("X is outside board");
  } else if (y < 1 || y > 3) {
    throw new RuntimeException("Y is outside board");
  }
  if (board[x - 1][y - 1] != '\0') {
    throw new RuntimeException("Box is occupied");
  } else {
    board[x - 1][y - 1] = 'X';
  }
}
```

我们正在检查所玩的位置是否被占用，如果没有被占用，我们将数组条目值从空（`\0`）更改为占用（`X`）。请记住，我们仍然没有存储是谁玩的（`X`还是`O`）。

# 重构

到目前为止，我们所做的代码满足了测试设置的要求，但看起来有点混乱。如果有人阅读它，就不清楚`play`方法的作用。我们应该通过将代码移动到单独的方法中来重构它。重构后的代码将如下所示：

```java
public void play(int x, int y) {
  checkAxis(x);
  checkAxis(y);
  setBox(x, y);
}

private void checkAxis(int axis) {
  if (axis < 1 || axis > 3) {
    throw new RuntimeException("X is outside board");
  }
}

private void setBox(int x, int y) {
  if (board[x - 1][y - 1] != '\0') {
    throw new RuntimeException("Box is occupied");
  } else {
    board[x - 1][y - 1] = 'X';
  }
}
```

通过这种重构，我们没有改变`play`方法的功能。它的行为与以前完全相同，但新代码更易读。由于我们有测试覆盖了所有现有功能，所以不用担心我们可能做错了什么。只要所有测试一直通过，重构没有引入任何新的行为，对代码进行更改就是安全的。

源代码可以在`01-exceptions` Git 存储库的`tdd-java-ch03-tic-tac-toe`分支中找到[`bitbucket.org/vfarcic/tdd-java-ch03-tic-tac-toe/branch/01-exceptions`](https://bitbucket.org/vfarcic/tdd-java-ch03-tic-tac-toe/branch/01-exceptions)。

# 需求 2-添加双人支持

现在是时候开始规范哪个玩家即将轮到他出手了。

应该有一种方法来找出下一个应该出手的玩家。

我们可以将这个需求分成三个测试：

+   第一轮应该由玩家`X`来玩

+   如果上一轮是由`X`玩的，那么下一轮应该由`O`玩。

+   如果上一轮是由`O`玩的，那么下一轮应该由`X`玩。

到目前为止，我们还没有使用任何 JUnit 的断言。要使用它们，我们需要从`org.junit.Assert`类中`import`静态方法：

```java
import static org.junit.Assert.*;
```

在`Assert`类内部，方法的本质非常简单。它们中的大多数以`assert`开头。例如，`assertEquals`比较两个对象-`assertNotEquals`验证两个对象不相同，`assertArrayEquals`验证两个数组相同。每个断言都有许多重载的变体，以便几乎可以使用任何类型的 Java 对象。

在我们的情况下，我们需要比较两个字符。第一个是我们期望的字符，第二个是从`nextPlayer`方法中检索到的实际字符。

现在是时候编写这些测试和实现了。

在编写实现代码之前编写测试。

这样做的好处如下-它确保编写可测试的代码，并确保为每一行代码编写测试。

通过先编写或修改测试，开发人员在开始编写代码之前专注于需求。这是与在实施完成后编写测试相比的主要区别。另一个好处是，有了先验测试，我们避免了测试作为质量检查而不是质量保证的危险。

# 测试- X 先玩

玩家`X`有第一轮：

```java
@Test
public void givenFirstTurnWhenNextPlayerThenX() {
  assertEquals('X', ticTacToe.nextPlayer());
}
```

第一轮应该由玩家`X`来玩。

这个测试应该是不言自明的。我们期望`nextPlayer`方法返回`X`。如果你尝试运行这个测试，你会发现代码甚至无法编译。那是因为`nextPlayer`方法甚至不存在。我们的工作是编写`nextPlayer`方法，并确保它返回正确的值。

# 实施

没有真正的必要检查是否真的是玩家的第一轮。就目前而言，这个测试可以通过始终返回`X`来实现。稍后的测试将迫使我们完善这段代码：

```java
public char nextPlayer() {
  return 'X';
}
```

# 测试- O 在 X 之后玩

现在，我们应该确保玩家在变化。在`X`完成后，应该轮到`O`，然后再次是`X`，依此类推：

```java
@Test
public void givenLastTurnWasXWhenNextPlayerThenO() {
  ticTacToe.play(1, 1);
  assertEquals('O', ticTacToe.nextPlayer());
}
```

如果上一轮是由`X`玩的，那么下一轮应该由`O`玩。

# 实施

为了追踪谁应该下一步出手，我们需要存储上一次出手的玩家：

```java
private char lastPlayer = '\0';

public void play(int x, int y) {
  checkAxis(x);
  checkAxis(y);
  setBox(x, y);
  lastPlayer = nextPlayer();
}

public char nextPlayer() {
  if (lastPlayer == 'X') {
    return 'O';
  }
  return 'X';
}
```

你可能开始掌握了。测试很小，很容易写。有了足够的经验，应该只需要一分钟，甚至几秒钟来编写一个测试，编写实现的时间也不会超过这个时间。

# 测试- X 在 O 之后玩

最后，我们可以检查`O`下完棋后是否轮到`X`下棋。

如果最后一步是由`O`下的，那么下一步应该由`X`下。

没有什么可以做来满足这个测试，因此这个测试是无用的，应该被丢弃。如果你写这个测试，你会发现它是一个错误的阳性。它会在不改变实现的情况下通过；试一下。写下这个测试，如果它成功了而没有写任何实现代码，就把它丢弃。

源代码可以在`tdd-java-ch03-tic-tac-toe` Git 存储库的`02-next-player`分支中找到[`bitbucket.org/vfarcic/tdd-java-ch03-tic-tac-toe/branch/02-next-player`](https://bitbucket.org/vfarcic/tdd-java-ch03-tic-tac-toe/branch/02-next-player)。

# 需求 3 - 添加获胜条件

现在是根据游戏规则来处理获胜的时候了。与之前的代码相比，这部分工作变得有点繁琐。我们应该检查所有可能的获胜组合，如果其中一个被满足，就宣布获胜者。

玩家通过首先连接棋盘的一侧或角落到另一侧的友方棋子线来获胜。

为了检查友方棋子线是否连接，我们应该验证水平、垂直和对角线。

# 测试 - 默认情况下没有赢家

让我们从定义`play`方法的默认响应开始：

```java
@Test
public void whenPlayThenNoWinner() {
  String actual = ticTacToe.play(1,1);
  assertEquals("No winner", actual);
}
```

如果没有满足获胜条件，那么就没有赢家。

# 实施

默认返回值总是最容易实现的，这个也不例外：

```java
public String play(int x, int y) {
  checkAxis(x);
  checkAxis(y);
  setBox(x, y);
  lastPlayer = nextPlayer();
  return "No winner";
}
```

# 测试 - 获胜条件 I

现在我们已经声明了默认响应是“没有赢家”，是时候开始处理不同的获胜条件了：

```java
@Test
public void whenPlayAndWholeHorizontalLineThenWinner() {
  ticTacToe.play(1, 1); // X
  ticTacToe.play(1, 2); // O
  ticTacToe.play(2, 1); // X
  ticTacToe.play(2, 2); // O
  String actual = ticTacToe.play(3, 1); // X
  assertEquals("X is the winner", actual);
}
```

玩家赢得比赛当整个水平线被他的棋子占据。

# 实施

为了完成这个测试，我们需要检查是否有任何水平线被当前玩家的标记填满。直到此刻，我们并不关心棋盘数组上放了什么。现在，我们不仅需要介绍哪些棋盘格子是空的，还需要介绍哪个玩家下的棋：

```java
public String play(int x, int y) {
  checkAxis(x);
  checkAxis(y);
  lastPlayer = nextPlayer();
  setBox(x, y, lastPlayer);
  for (int index = 0; index < 3; index++) {
    if (board[0][index] == lastPlayer
        && board[1][index] == lastPlayer
        && board[2][index] == lastPlayer) {
      return lastPlayer + " is the winner";
    }
  }
  return "No winner";
}
private void setBox(int x, int y, char lastPlayer) {
  if (board[x - 1][y - 1] != '\0') {
    throw new RuntimeException("Box is occupied");
  } else {
    board[x - 1][y - 1] = lastPlayer;
  }
}
```

# 重构

前面的代码满足了测试，但不一定是最终版本。它达到了尽快实现代码覆盖率的目的。现在，既然我们有了测试来保证预期行为的完整性，我们可以重构代码了：

```java
private static final int SIZE = 3;

public String play(int x, int y) {
  checkAxis(x);
  checkAxis(y);
  lastPlayer = nextPlayer();
  setBox(x, y, lastPlayer);
  if (isWin()) {
    return lastPlayer + " is the winner";
  }
  return "No winner";
}

private boolean isWin() {
  for (int i = 0; i < SIZE; i++) {
    if (board[0][i] + board[1][i] + board[2][i] == (lastPlayer * SIZE)) {
      return true;
    }
  }
  return false;
}
```

这个重构后的解决方案看起来更好。`play`方法保持简短易懂。获胜逻辑被移动到一个单独的方法中。我们不仅保持了`play`方法的目的清晰，而且这种分离还允许我们将获胜条件的代码与其余部分分开发展。

# 测试 - 获胜条件 II

我们还应该检查是否通过填充垂直线来获胜：

```java
@Test
public void whenPlayAndWholeVerticalLineThenWinner() {
  ticTacToe.play(2, 1); // X
  ticTacToe.play(1, 1); // O
  ticTacToe.play(3, 1); // X
  ticTacToe.play(1, 2); // O
  ticTacToe.play(2, 2); // X
  String actual = ticTacToe.play(1, 3); // O
  assertEquals("O is the winner", actual);
}
```

玩家赢得比赛当整个垂直线被他的棋子占据。

# 实施

这个实现应该类似于之前的实现。我们已经有了水平验证，现在我们需要做垂直验证：

```java
private boolean isWin() {
  int playerTotal = lastPlayer * 3;
  for (int i = 0; i < SIZE; i++) {
    if (board[0][i] + board[1][i] + board[2][i] == playerTotal) {
      return true;
    } else if (board[i][0] + board[i][1] + board[i][2] == playerTotal) {
      return true;
    }
  }
  return false;
}
```

# 测试 - 获胜条件 III

现在水平和垂直线都已经覆盖，我们应该把注意力转移到对角线组合上：

```java
@Test 
public void whenPlayAndTopBottomDiagonalLineThenWinner() {
  ticTacToe.play(1, 1); // X
  ticTacToe.play(1, 2); // O
  ticTacToe.play(2, 2); // X
  ticTacToe.play(1, 3); // O
  String actual = ticTacToe.play(3, 3); // X
  assertEquals("X is the winner", actual);
}
```

玩家赢得比赛当整个从左上到右下的对角线被他的棋子占据。

# 实施

由于只有一条线符合要求，我们可以直接检查它，而不需要任何循环：

```java
private boolean isWin() {
  int playerTotal = lastPlayer * 3;
  for (int i = 0; i < SIZE; i++) {
    if (board[0][i] + board[1][i] + board[2][i] == playerTotal) {
      return true;
    } else if (board[i][0] + board[i][1] + board[i][2] == playerTotal) {
      return true;
    } 
  } 
  if (board[0][0] + board[1][1] + board[2][2] == playerTotal) { 
    return true; 
  }   
  return false; 
} 
```

# 代码覆盖率

在整个练习过程中，我们没有使用代码覆盖工具。原因是我们希望您专注于红-绿-重构模型。您编写了一个测试，看到它失败，编写了实现代码，看到所有测试都成功执行，然后在看到机会使代码更好时重构了代码，然后重复了这个过程。我们的测试覆盖了所有情况吗？这是 JaCoCo 等代码覆盖工具可以回答的问题。您应该使用这些工具吗？可能只有在开始时。让我澄清一下。当您开始使用 TDD 时，您可能会错过一些测试或者实现超出了测试定义的内容。在这些情况下，使用代码覆盖是从自己的错误中学习的好方法。随着您在 TDD 方面的经验增加，您对这些工具的需求将会减少。您将编写测试，并编写足够的代码使其通过。无论是否使用 JaCoCo 等工具，您的覆盖率都会很高。由于您会对不值得测试的内容做出有意识的决定，因此只有少量代码不会被测试覆盖。

诸如 JaCoCo 之类的工具主要是作为一种验证实现代码后编写的测试是否提供足够覆盖率的方式。通过 TDD，我们采用了不同的方法，即倒置顺序（先编写测试，再实现）。

尽管如此，我们建议您将 JaCoCo 作为学习工具，并自行决定是否在将来使用它。

要在 Gradle 中启用 JaCoCo，请将以下内容添加到`build.gradle`中：

```java
apply plugin: 'jacoco'
```

从现在开始，每次运行测试时，Gradle 都会收集 JaCoCo 指标。这些指标可以使用`jacocoTestReport` Gradle 目标转换为漂亮的报告。让我们再次运行测试，看看代码覆盖率是多少：

```java
$ gradle clean test jacocoTestReport
```

最终结果是报告位于`build/reports/jacoco/test/html`目录中。结果将取决于您为此练习制定的解决方案。我的结果显示指令覆盖率为 100%，分支覆盖率为 96%；缺少 4%是因为没有测试案例中玩家在 0 或负数的方框上下棋。该情况的实现已经存在，但没有特定的测试覆盖它。总的来说，这是一个相当不错的覆盖率。

![](img/ad410f3d-0ec5-4109-a35a-0e9ca579d7d7.png)

JaCoCo 将被添加到源代码中。这可以在`05-jacoco`分支的` tdd-java-ch03-tic-tac-toe` Git 存储库中找到，网址为[`bitbucket.org/vfarcic/tdd-java-ch03-tic-tac-toe/branch/05-jacoco`](https://bitbucket.org/vfarcic/tdd-java-ch03-tic-tac-toe/branch/05-jacoco)。

# 测试 - 获胜条件 IV

最后，还有最后一个可能的获胜条件要解决：

```java
@Test
public void whenPlayAndBottomTopDiagonalLineThenWinner() {
  ticTacToe.play(1, 3); // X
  ticTacToe.play(1, 1); // O
  ticTacToe.play(2, 2); // X
  ticTacToe.play(1, 2); // O
  String actual = ticTacToe.play(3, 1); // X
  assertEquals("X is the winner", actual);
}
```

当整个对角线从左下到右上的线被玩家的棋子占据时，玩家获胜。

# 实现

这个测试的实现应该几乎与上一个相同：

```java
private boolean isWin() {
  int playerTotal = lastPlayer * 3;
  for (int i = 0; i < SIZE; i++) {
    if (board[0][i] + board[1][i] + board[2][i] == playerTotal) {
      return true;
    } else if (board[i][0] + board[i][1] + board[i][2] == playerTotal) {
      return true;
    }
  }
  if (board[0][0] + board[1][1] + board[2][2] == playerTotal) {
    return true;
  } else if (board[0][2] + board[1][1] + board[2][0] == playerTotal) {
    return true;
  }
  return false;
}
```

# 重构

我们处理可能的对角线获胜的方式，计算看起来不对。也许重新利用现有的循环会更有意义：

```java
private boolean isWin() {
  int playerTotal = lastPlayer * 3;
  char diagonal1 = '\0';
  char diagonal2 = '\0';
  for (int i = 0; i < SIZE; i++) {
    diagonal1 += board[i][i];
    diagonal2 += board[i][SIZE - i - 1];
    if (board[0][i] + board[1][i] + board[2][i]) == playerTotal) {
      return true;
    } else if (board[i][0] + board[i][1] + board[i][2] == playerTotal) {
      return true;
    }
  }
  if (diagonal1 == playerTotal || diagonal2 == playerTotal) {
    return true;
  }
  return false;
}
```

源代码可以在` tdd-java-ch03-tic-tac-toe` Git 存储库的`03-wins`分支中找到，网址为[`bitbucket.org/vfarcic/tdd-java-ch03-tic-tac-toe/branch/03-wins`](https://bitbucket.org/vfarcic/tdd-java-ch03-tic-tac-toe/branch/03-wins)。

现在，让我们来看最后一个要求。

# 要求 4 - 平局条件

唯一缺少的是如何处理平局结果。

当所有方框都填满时，结果是平局。

# 测试 - 处理平局情况

我们可以通过填满棋盘上的所有方框来测试平局结果：

```java
@Test
public void whenAllBoxesAreFilledThenDraw() {
  ticTacToe.play(1, 1);
  ticTacToe.play(1, 2);
  ticTacToe.play(1, 3);
  ticTacToe.play(2, 1);
  ticTacToe.play(2, 3);
  ticTacToe.play(2, 2);
  ticTacToe.play(3, 1);
  ticTacToe.play(3, 3);
  String actual = ticTacToe.play(3, 2);
  assertEquals("The result is draw", actual);
}
```

# 实现

检查是否为平局非常简单。我们只需要检查棋盘上的所有方框是否都填满了。我们可以通过遍历棋盘数组来做到这一点：

```java
public String play(int x, int y) {
  checkAxis(x);
  checkAxis(y);
  lastPlayer = nextPlayer();
  setBox(x, y, lastPlayer);
  if (isWin()) {
    return lastPlayer + " is the winner";
  } else if (isDraw()) {
    return "The result is draw";
  } else {
    return "No winner";
  }
}

private boolean isDraw() {
  for (int x = 0; x < SIZE; x++) {
    for (int y = 0; y < SIZE; y++) {
      if (board[x][y] == '\0') {
        return false;
      }
    }
  }
  return true;
}
```

# 重构

尽管`isWin`方法不是最后一个测试的范围，但它仍然可以进行更多的重构。首先，我们不需要检查所有的组合，而只需要检查与最后一个放置的棋子位置相关的组合。最终版本可能如下所示：

```java
private boolean isWin(int x, int y) {
  int playerTotal = lastPlayer * 3;
  char horizontal, vertical, diagonal1, diagonal2;
  horizontal = vertical = diagonal1 = diagonal2 = '\0';
  for (int i = 0; i < SIZE; i++) {
    horizontal += board[i][y - 1];
    vertical += board[x - 1][i];
    diagonal1 += board[i][i];
    diagonal2 += board[i][SIZE - i - 1];
  }
  if (horizontal == playerTotal
      || vertical == playerTotal
      || diagonal1 == playerTotal
      || diagonal2 == playerTotal) {
    return true;
  }
  return false;
} 
```

重构可以在任何时候的代码的任何部分进行，只要所有测试都成功。虽然通常最容易和最快的是重构刚刚编写的代码，但是回到前天、上个月甚至几年前编写的代码也是非常受欢迎的。重构的最佳时机是当有人看到使其更好的机会时。不管是谁编写的或者何时编写的，使代码更好总是一件好事。

源代码可以在`04-draw` Git 存储库的`tdd-java-ch03-tic-tac-toe`分支中找到，网址为[`bitbucket.org/vfarcic/tdd-java-ch03-tic-tac-toe/branch/04-draw`](https://bitbucket.org/vfarcic/tdd-java-ch03-tic-tac-toe/branch/04-draw)。

# 更多练习

我们刚刚开发了一个（最常用的）井字棋游戏变体。作为额外的练习，从维基百科（[`en.wikipedia.org/wiki/Tic-tac-toe`](http://en.wikipedia.org/wiki/Tic-tac-toe)）中选择一个或多个变体，并使用红-绿-重构程序实现它。完成后，实现一种能够玩`O`的回合的人工智能。由于井字棋通常导致平局，当 AI 成功达到任何`X`的移动组合时，可以认为 AI 已经完成。

在做这些练习时，记得要快速并进行乒乓对打。最重要的是，记得要使用红-绿-重构程序。

# 总结

我们成功地使用红-绿-重构过程完成了我们的井字棋游戏。这些例子本身很简单，你可能没有问题跟随它们。

本章的目标不是深入研究复杂的东西（这将在后面进行），而是养成使用称为红-绿-重构的短而重复的循环习惯。

我们学到了开发某物最简单的方法是将其分解成非常小的块。设计是从测试中出现的，而不是采用大量的前期方法。没有一行实现代码是在没有先编写测试并看到它失败的情况下编写的。通过确认最后一个测试失败，我们确认它是有效的（很容易出错并编写一个始终成功的测试），并且我们即将实现的功能不存在。测试失败后，我们编写了该测试的实现。在编写实现时，我们试图使其尽可能简化，目标是使测试通过，而不是使解决方案最终化。我们重复这个过程，直到我们感到有必要重构代码。重构不会引入任何新功能（我们没有改变应用程序的功能），但会使代码更加优化，更易于阅读和维护。

在下一章中，我们将更详细地阐述在 TDD 环境中什么构成了一个单元，以及如何根据这些单元的创建测试。
