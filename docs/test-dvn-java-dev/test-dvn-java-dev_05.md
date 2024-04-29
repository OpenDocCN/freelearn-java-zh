# 第五章：设计-如果不能进行测试，那就不是设计良好

“简单是终极的复杂。”

- 列奥纳多·达·芬奇

过去，软件行业专注于以高速开发软件，只考虑成本和时间。质量是次要目标，人们错误地认为客户对此不感兴趣。

如今，随着各种平台和设备的连接性不断增加，质量已成为客户需求中的一等公民。良好的应用程序在合理的响应时间内提供良好的服务，而不会受到许多用户的大量并发请求的影响。

在质量方面，良好的应用程序是那些经过良好设计的。良好的设计意味着可扩展性、安全性、可维护性和许多其他期望的属性。

在本章中，我们将探讨 TDD 如何通过使用传统和 TDD 方法来实现相同的应用程序，从而引导开发人员走向良好的设计和最佳实践。

本章将涵盖以下主题：

+   我们为什么要关心设计？

+   设计考虑和原则

+   传统的开发过程

+   使用 Hamcrest 的 TDD 方法

# 我们为什么要关心设计？

在软件开发中，无论您是专家还是初学者，都会遇到一些代码看起来不自然的情况。在阅读时，您无法避免感觉到代码有问题。有时，您甚至会想知道为什么以前的程序员以这种扭曲的方式实现了特定的方法或类。这是因为相同的功能可以以大量不同的方式实现，每种方式都是独一无二的。在如此多的可能性中，哪一个是最好的？什么定义了一个好的解决方案？为什么一个比其他的更好？事实是，只要达到目标，所有这些都是有效的。然而，选择正确解决方案时应考虑一些方面。这就是解决方案的设计变得相关的地方。

# 设计原则

**软件设计原则**是软件开发人员的指导原则，推动他们朝着智能和可维护的解决方案前进。换句话说，设计原则是代码必须满足的条件，以便被认为是客观良好设计的。

大多数资深开发人员和经验丰富的程序员都了解软件设计原则，很可能无论他们是否实践 TDD，他们都在日常工作中应用这些原则。TDD 哲学鼓励程序员-甚至是初学者-遵循一些原则和良好实践，使代码更清晰、更可读。这些实践是由红-绿-重构周期强制执行的。

红-绿-重构周期倡导通过一次引入一个失败的测试来实现小的功能增量。程序员添加尽可能简洁和短小的代码片段，以便新的测试或旧的测试都不再失败。最终，他们重构代码，包括清理和改进任务，如去除重复或优化代码。

作为过程的结果，代码变得更容易理解，并且在将来修改时更安全。让我们来看一些最流行的软件设计原则。

# 你不会需要它

**YAGNI**是**You Ain't Gonna Need It**原则的缩写。它旨在消除所有不必要的代码，专注于当前的功能，而不是未来的功能。您的代码越少，您需要维护的代码就越少，引入错误的可能性就越低。

有关 YAGNI 的更多信息，请访问 Martin Fowler 的文章，网址为[`martinfowler.com/bliki/Yagni.html`](http://martinfowler.com/bliki/Yagni.html)。

# 不要重复自己

**不要重复自己**（DRY）原则的理念是重用之前编写的代码，而不是重复它。好处是减少需要维护的代码，使用已知可行的代码，这是一件好事。它可以帮助你发现代码中的新抽象层级。

欲了解更多信息，请访问[`en.wikipedia.org/wiki/Don%27t_repeat_yourself`](http://en.wikipedia.org/wiki/Don%27t_repeat_yourself)。

# 保持简单，愚蠢

这个原则有一个令人困惑的缩写**保持简单，愚蠢**（KISS），并且陈述了事物如果保持简单而不是复杂，它们会更好地发挥功能。这是由凯利·约翰逊创造的。

要了解这个原则背后的故事，请访问[`en.wikipedia.org/wiki/KISS_principle`](http://en.wikipedia.org/wiki/KISS_principle)。

# 奥卡姆剃刀

尽管**奥卡姆剃刀**是一个哲学原则，而不是软件工程原则，但它仍然适用于我们的工作。它与前一个原则非常相似，主要陈述如下：

“当你有两种竞争解决同一个问题的方案时，简单的那个更好。”

– 奥卡姆的威廉

欲了解更多奥卡姆剃刀原理，请访问[`en.wikipedia.org/wiki/Occam%27s_razor`](http://en.wikipedia.org/wiki/Occam%27s_razor)。

# SOLID 原则

**SOLID**这个词是罗伯特·C·马丁为面向对象编程的五个基本原则创造的缩写。通过遵循这五个原则，开发人员更有可能创建一个出色、耐用和易于维护的应用程序：

+   **单一职责原则**：一个类应该只有一个改变的原因。

+   **开闭原则**：一个类应该对扩展开放，对修改关闭。这被归因于贝尔特兰·梅耶。

+   **里氏替换原则**：这是由芭芭拉·里斯科夫创建的，她说*一个类应该可以被扩展该类的其他类替换*。

+   **接口隔离原则**：几个特定的接口比一个通用接口更可取。

+   **依赖反转原则**：一个类应该依赖于抽象而不是实现。这意味着类的依赖必须专注于做什么，而忘记了如何做。

欲了解更多关于 SOLID 或其他相关原则的信息，请访问[`butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod`](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod)。

前四个原则是 TDD 思想的核心部分，因为它们旨在简化我们编写的代码。最后一个原则侧重于应用程序组装过程中的类构建和依赖关系。

所有这些原则在测试驱动开发和非测试驱动开发中都是适用且可取的，因为除了其他好处外，它们使我们的代码更易于维护。它们的正确实际应用值得一整本书来讨论。虽然我们没有时间深入研究，但我们鼓励你进一步调查。

在本章中，我们将看到 TDD 如何使开发人员轻松地将这些原则付诸实践。我们将使用 TDD 和非 TDD 方法实现一个小型但完全功能的四子连线游戏版本。请注意，重复的部分，如 Gradle 项目创建等，被省略了，因为它们不被认为与本章的目的相关。

# 四子连线

四子连线是一款受欢迎、易于玩的棋盘游戏。规则有限且简单。

四子连线是一款双人对战的连接游戏，玩家首先选择一种颜色，然后轮流将有颜色的圆盘从顶部放入一个七列六行的垂直悬挂网格中。棋子直接下落，占据列中的下一个可用空间。游戏的目标是在对手连接四个自己颜色的圆盘之前，垂直、水平或对角线连接四个相同颜色的圆盘。

有关游戏的更多信息，请访问维基百科（[`en.wikipedia.org/wiki/Connect_Four`](http://en.wikipedia.org/wiki/Connect_Four)）。

# 要求

为了编写 Connect 4 的两种实现，游戏规则被转录为以下需求的形式。这些需求是两种开发的起点。我们将通过一些解释来查看代码，并在最后比较两种实现：

1.  棋盘由七列和六行组成；所有位置都是空的。

1.  玩家在列的顶部放入圆盘。如果列为空，则放入的圆盘会下落到棋盘上。将来在同一列中放入的圆盘将堆叠在之前的圆盘上。

1.  这是一个双人游戏，所以每个玩家都有一个颜色。一个玩家使用红色（*R*），另一个使用绿色（*G*）。玩家轮流进行，每次插入一个圆盘。

1.  我们希望在游戏中发生事件或错误时得到反馈。输出显示每次移动后棋盘的状态。

1.  当不能再插入圆盘时，游戏结束，被视为平局。

1.  如果玩家插入一个圆盘并连接了三个以上的同色圆盘，那么该玩家就赢了。

1.  在水平线方向上也是一样的。

1.  在对角线方向上也是一样的。

# Connect 4 的测试后实现

这是传统的方法，侧重于解决问题的代码，而不是测试。一些人和公司忘记了自动化测试的价值，并依赖于用户所谓的**用户验收测试**。

这种用户验收测试包括在一个受控环境中重新创建真实世界的场景，理想情况下与生产环境完全相同。一些用户执行许多不同的任务来验证应用程序的正确性。如果这些操作中的任何一个失败，那么代码就不会被接受，因为它破坏了某些功能或者不符合预期的工作。

此外，许多这些公司还使用单元测试作为进行早期回归检查的一种方式。这些单元测试是在开发过程之后创建的，并试图尽可能多地覆盖代码。最后，执行代码覆盖率分析以获得这些单元测试实际覆盖的内容。这些公司遵循一个简单的经验法则：代码覆盖率越高，交付的质量就越好。

这种方法的主要问题是事后编写测试只能证明代码的行为方式是按照程序编写的方式，这未必是代码预期行为的方式。此外，专注于代码覆盖率会导致糟糕的测试，将我们的生产代码变成不可变的实体。我们可能想要添加的每个修改都可能导致代码中不相关部分的多个测试失败。这意味着引入更改的成本变得非常高，进行任何轻微的修改可能会变成一场噩梦，非常昂贵。

为了演示前面描述的一些要点，让我们使用 TDD 和非 TDD 方法来实现 Connect 4 游戏。随着我们进一步进行，每个确定需求的相关代码将被呈现出来。这些代码并非是逐步编写的，因此一些代码片段可能包含一些与所提到的需求无关的代码行。

# 需求 1 - 游戏的棋盘

让我们从第一个需求开始。

棋盘由七个水平和六个垂直的空位置组成。

这个需求的实现非常直接。我们只需要表示一个空位置和保存游戏的数据结构。请注意，玩家使用的颜色也已经定义：

```java
public class Connect4 {
  public enum Color {
    RED('R'), GREEN('G'), EMPTY(' ');

    private final char value;

    Color(char value) { this.value = value; }

    @Override
    public String toString() {
      return String.valueOf(value);
    }
  }

  public static final int COLUMNS = 7;

  public static final int ROWS = 6;

  private Color[][] board = new Color[COLUMNS][ROWS];

  public Connect4() {
    for (Color[] column : board) {
      Arrays.fill(column, Color.EMPTY);
    }
  }
}
```

# 需求 2 - 插入圆盘

这个需求介绍了游戏的一部分逻辑。

玩家在列的顶部放入圆盘。如果列为空，则放入的圆盘会下落到棋盘上。将来在同一列中放入的圆盘将堆叠在之前的圆盘上。

在这一部分，棋盘边界变得相关。我们需要标记哪些位置已经被占据，使用`Color.RED`来指示它们。最后，创建了第一个`private`方法。这是一个帮助方法，用于计算在给定列中插入的圆盘数量：

```java
public void putDisc(int column) {
  if (column > 0 && column <= COLUMNS) {
    int numOfDiscs = getNumberOfDiscsInColumn(column - 1);
    if (numOfDiscs < ROWS) {
      board[column - 1][numOfDiscs] = Color.RED;
    }
  }
}

private int getNumberOfDiscsInColumn(int column) {
  if (column >= 0 && column < COLUMNS) {
    int row;
    for (row = 0; row < ROWS; row++) {
      if (Color.EMPTY == board[column][row]) {
        return row;
      }
    }
    return row;
  }
  return -1;
}
```

# 要求 3 - 玩家轮换

这个要求引入了更多的游戏逻辑。

这是一个双人游戏，所以每个玩家有一种颜色。一个玩家使用红色（*R*），另一个使用绿色（*G*）。玩家轮流进行，每次插入一个圆盘。

我们需要保存当前玩家以确定哪个玩家在进行这一轮。我们还需要一个函数来切换玩家以重新创建轮换的逻辑。在`putDisc`函数中，一些代码变得相关。具体来说，使用当前玩家进行棋盘位置分配，并且按照游戏规则在每次移动后进行切换：

```java
...
private Color currentPlayer = Color.RED;

private void switchPlayer() {
  if (Color.RED == currentPlayer) {
    currentPlayer = Color.GREEN;
  } else {
    currentPlayer = Color.RED;
  }
}

public void putDisc(int column) {
  if (column > 0 && column <= COLUMNS) {
    int numOfDiscs = getNumberOfDiscsInColumn(column - 1);
    if (numOfDiscs < ROWS) {
      board[column - 1][numOfDiscs] = currentPlayer;
      switchPlayer();
    }
  }
}
...
```

# 要求 4 - 游戏的输出

应该添加一些输出，让玩家知道游戏的当前状态。

我们希望在游戏中发生事件或错误时得到反馈。输出显示每次移动后棋盘的状态。

没有指定输出通道。为了更容易，我们决定使用系统标准输出来在事件发生时打印事件。在每个动作上添加了几行代码，以便让用户了解游戏的状态：

```java
... 
private static final String DELIMITER = "|";

private void switchPlayer() {
  if (Color.RED == currentPlayer) {
    currentPlayer = Color.GREEN;
  } else {
    currentPlayer = Color.RED;
  }
  System.out.println("Current turn: " + currentPlayer);
}

public void printBoard() {
  for (int row = ROWS - 1; row >= 0; --row) {
    StringJoiner stringJoiner =
      new StringJoiner(DELIMITER, DELIMITER, DELIMITER);
    for (int col = 0; col < COLUMNS; ++col) {
      stringJoiner.add(board[col][row].toString());
    }
    System.out.println(stringJoiner.toString());
  }
}

public void putDisc(int column) {
  if (column > 0 && column <= COLUMNS) {
    int numOfDiscs = getNumberOfDiscsInColumn(column - 1); 
    if (numOfDiscs < ROWS) { 
      board[column - 1][numOfDiscs] = currentPlayer; 
      printBoard();
      switchPlayer();
    } else {
      System.out.println(numOfDiscs); 
      System.out.println("There's no room " + 
        "for a new disc in this column"); 
      printBoard(); 
    } 
  } else { 
    System.out.println("Column out of bounds"); 
    printBoard(); 
  } 
}
... 
```

# 要求 5 - 胜利条件（I）

第一局游戏有一个结束条件。

当不能再插入圆盘时，游戏结束并被视为平局。

以下代码显示了可能的一种实现：

```java
...
public boolean isFinished() {
  int numOfDiscs = 0;
  for (int col = 0; col < COLUMNS; ++col) {
    numOfDiscs += getNumberOfDiscsInColumn(col);
  }
  if (numOfDiscs >= COLUMNS * ROWS) {
    System.out.println("It's a draw");
    return true;
  }
  return false;
}
...
```

# 要求 6 - 胜利条件（II）

第一个胜利条件。

如果一个玩家插入一个圆盘并连接了三个以上的同色圆盘，那么该玩家获胜。

`checkWinCondition`私有方法通过扫描最后一步是否是获胜来实现这一规则：

```java
... 
private Color winner;

public static final int DISCS_FOR_WIN = 4;

public void putDisc(int column) {
  ...
  if (numOfDiscs < ROWS) {
    board[column - 1][numOfDiscs] = currentPlayer;
    printBoard();
    checkWinCondition(column - 1, numOfDiscs);
    switchPlayer();
    ...
}

private void checkWinCondition(int col, int row) {
  Pattern winPattern = Pattern.compile(".*" +
    currentPlayer + "{" + DISCS_FOR_WIN + "}.*");

  // Vertical check
  StringJoiner stringJoiner = new StringJoiner("");
  for (int auxRow = 0; auxRow < ROWS; ++auxRow) {
    stringJoiner.add(board[col][auxRow].toString());
  }
  if (winPattern.matcher(stringJoiner.toString()).matches()) {
    winner = currentPlayer;
    System.out.println(currentPlayer + " wins");
  }
}

public boolean isFinished() {
  if (winner != null) return true;
  ...
}
...
```

# 要求 7 - 胜利条件（III）

这是相同的胜利条件，但是在不同的方向上。

如果一个玩家插入一个圆盘并连接了三个以上的同色圆盘，那么该玩家获胜。

实现这一规则的几行代码如下：

```java
...
private void checkWinCondition(int col, int row) {
  ...
  // Horizontal check
  stringJoiner = new StringJoiner("");
  for (int column = 0; column < COLUMNS; ++column) {
    stringJoiner.add(board[column][row].toString());
  }
  if (winPattern.matcher(stringJoiner.toString()).matches()) { 
    winner = currentPlayer;
    System.out.println(currentPlayer + " wins");
    return;
  }
  ...
}
...
```

# 要求 8 - 胜利条件（IV）

最后一个要求是最后的胜利条件。这与前两个非常相似；在这种情况下，是在对角线方向上。

如果一个玩家插入一个圆盘并连接了三个以上的同色圆盘，那么该玩家获胜。

这是对最后一个要求的一个可能的实现。这段代码与其他胜利条件非常相似，因为必须满足相同的条件：

```java
...
private void checkWinCondition(int col, int row) {
  ...
  // Diagonal checks
  int startOffset = Math.min(col, row);
  int column = col - startOffset, auxRow = row - startOffset; 
  stringJoiner = new StringJoiner("");
  do {
    stringJoiner.add(board[column++][auxRow++].toString());
  } while (column < COLUMNS && auxRow < ROWS);

  if (winPattern.matcher(stringJoiner.toString()).matches()) {
    winner = currentPlayer;
    System.out.println(currentPlayer + " wins");
    return;
  }

  startOffset = Math.min(col, ROWS - 1 - row);
  column = col - startOffset;
  auxRow = row + startOffset;
  stringJoiner = new StringJoiner("");
  do {
    stringJoiner.add(board[column++][auxRow--].toString());
  } while (column < COLUMNS && auxRow >= 0);

  if (winPattern.matcher(stringJoiner.toString()).matches()) {
    winner = currentPlayer;
    System.out.println(currentPlayer + " wins");
  }
}
...
```

我们得到了一个带有一个构造函数、三个公共方法和三个私有方法的类。应用程序的逻辑分布在所有方法中。这里最大的缺陷是这个类非常难以维护。关键的方法，比如`checkWinCondition`，都是非平凡的，有潜在的 bug 可能在未来的修改中出现。

如果你想查看完整的代码，你可以在[`bitbucket.org/vfarcic/tdd-java-ch05-design.git`](https://bitbucket.org/vfarcic/tdd-java-ch05-design.git)存储库中找到。

我们制作了这个小例子来演示这种方法的常见问题。像 SOLID 原则这样的主题需要一个更大的项目来更具说明性。

在拥有数百个类的大型项目中，问题变成了在一种类似手术的开发中浪费了数小时。开发人员花费大量时间调查棘手的代码并理解其工作原理，而不是创建新功能。

# TDD 或先测试的实现

此时，我们知道 TDD 是如何工作的——在测试之前编写测试，然后实现测试，最后进行重构。我们将通过这个过程，只展示每个要求的最终结果。剩下的就是让你去理解迭代的红绿重构过程。如果可能的话，让我们在测试中使用 Hamcrest 框架，让这更有趣。

# Hamcrest

如第二章所述，*工具、框架和环境*，Hamcrest 提高了我们测试的可读性。它使断言更有语义和全面性，通过使用**匹配器**减少了复杂性。当测试失败时，通过解释断言中使用的匹配器，显示的错误更具表现力。开发人员还可以添加消息。

`Hamcrest`库中充满了不同类型对象和集合的不同匹配器。让我们开始编码，尝试一下。

# 要求 1 - 游戏的棋盘

我们将从第一个要求开始。

棋盘由七个水平和六个垂直的空位置组成。

这个要求没有太大的挑战。棋盘边界已经指定，但在其中没有描述行为；只是在游戏开始时考虑了一个空棋盘。这意味着游戏开始时没有圆盘。然而，这个要求以后必须考虑。

这是针对此要求的测试类的外观。有一个方法来初始化`tested`类，以便在每个测试中使用一个完全新的对象。还有第一个测试来验证游戏开始时没有圆盘，这意味着所有的棋盘位置都是空的：

```java
public class Connect4TDDSpec {
  private Connect4TDD tested;

  @Before
  public void beforeEachTest() {
    tested = new Connect4TDD();
  }
  @Test
  public void whenTheGameIsStartedTheBoardIsEmpty() {
    assertThat(tested.getNumberOfDiscs(), is(0));
  }
}
```

这是前述规范的 TDD 实现。观察给出的解决方案对于这个第一个要求的简单方法，一个简单的方法在一行中返回结果：

```java
public class Connect4TDD {
  public int getNumberOfDiscs() {
    return 0;
  }
}
```

# 要求 2 - 引入圆盘

这是第二个要求的实现。

玩家在列的顶部放入圆盘。如果列为空，则放入的圆盘会下落到棋盘上。未来放入同一列的圆盘将堆叠在前面的圆盘上。

我们可以将此要求分为以下测试：

+   当一个圆盘插入到一个空列中时，它的位置是`0`

+   当第二个圆盘插入到同一列时，它的位置是`1`

+   当一个圆盘插入到棋盘上时，圆盘的总数增加

+   当一个圆盘放在边界外时，会抛出`Runtime Exception`

+   当一个圆盘插入到一列中，没有可用的空间时，就会抛出`Runtime Exception`

此外，这些其他测试源自第一个要求。它们与棋盘限制或棋盘行为有关。

上述测试的 Java 实现如下：

```java
@Test 
public void whenDiscOutsideBoardThenRuntimeException() {
  int column = -1;
  exception.expect(RuntimeException.class);
  exception.expectMessage("Invalid column " + column);
  tested.putDiscInColumn(column);
}

@Test
public void whenFirstDiscInsertedInColumnThenPositionIsZero() {
  int column = 1;
  assertThat(tested.putDiscInColumn(column),  is(0));
}

@Test
public void whenSecondDiscInsertedInColumnThenPositionIsOne() {
  int column = 1;
  tested.putDiscInColumn(column);
  assertThat(tested.putDiscInColumn(column), is(1));
}

@Test
public void whenDiscInsertedThenNumberOfDiscsIncreases() {
  int column = 1;
  tested.putDiscInColumn(column);
  assertThat(tested.getNumberOfDiscs(), is(1));
}

@Test 
public void whenNoMoreRoomInColumnThenRuntimeException() {
  int column = 1;
  int maxDiscsInColumn = 6; // the number of rows
  for (int times = 0; times < maxDiscsInColumn; ++times) {
    tested.putDiscInColumn(column);
  }
  exception.expect(RuntimeException.class);
  exception.expectMessage("No more room in column " + column);
  tested.putDiscInColumn(column);
}
```

这是满足测试的必要代码：

```java
private static final int ROWS = 6;

private static final int COLUMNS = 7;

private static final String EMPTY = " ";

private String[][] board = new String[ROWS][COLUMNS];

public Connect4TDD() {
  for (String[] row : board) Arrays.fill(row, EMPTY);
}

public int getNumberOfDiscs() {
  return IntStream
           .range(0, COLUMNS)
           .map(this::getNumberOfDiscsInColumn)
           .sum(); 
} 

private int getNumberOfDiscsInColumn(int column) {
  return (int) IntStream
                 .range(0, ROWS)
                 .filter(row -> !EMPTY.equals(board[row][column]))
                 .count();
}

public int putDiscInColumn(int column) {
  checkColumn(column);
  int row = getNumberOfDiscsInColumn(column);
  checkPositionToInsert(row, column);
  board[row][column] = "X";
  return row;
}

private void checkColumn(int column) {
  if (column < 0 || column >= COLUMNS)
    throw new RuntimeException("Invalid column " + column);
}

private void checkPositionToInsert(int row, int column) {
  if (row == ROWS)
    throw new RuntimeException("No more room in column " + column); 
} 
```

# 要求 3 - 玩家轮换

第三个要求涉及游戏逻辑。

这是一个双人游戏，所以每个玩家都有一个颜色。一个玩家使用红色（*R*），另一个玩家使用绿色（*G*）。玩家轮流进行，每次插入一个圆盘。

这些测试涵盖了新功能的验证。为了简单起见，红色玩家将始终开始游戏：

```java
@Test
public void whenFirstPlayerPlaysThenDiscColorIsRed() {
  assertThat(tested.getCurrentPlayer(), is("R"));
}

@Test
public void whenSecondPlayerPlaysThenDiscColorIsRed() {
  int column = 1;
  tested.putDiscInColumn(column);
  assertThat(tested.getCurrentPlayer(), is("G"));
}
```

需要创建一些方法来覆盖这个功能。在`putDiscInColumn`方法中返回行之前调用`switchPlayer`方法：

```java
private static final String RED = "R";

private static final String GREEN = "G";

private String currentPlayer = RED;

public Connect4TDD() {
  for (String[] row : board) Arrays.fill(row, EMPTY);
}

public String getCurrentPlayer() {
  return currentPlayer;
}

private void switchPlayer() {
  if (RED.equals(currentPlayer)) currentPlayer = GREEN;
  else currentPlayer = RED;
}

public int putDiscInColumn(int column) {
  ...
  switchPlayer();
  return row;
}
```

# 要求 4 - 游戏输出

接下来，我们应该让玩家知道游戏的状态。

我们希望在游戏中发生事件或错误时得到反馈。输出显示每次移动时棋盘的状态。

当发生错误时我们抛出异常，这已经涵盖了，所以我们只需要实现这两个测试。此外，为了便于测试，我们需要在构造函数中引入一个参数。通过引入这个参数，输出变得更容易测试：

```java
private OutputStream output;

@Before
public void beforeEachTest() {
  output = new ByteArrayOutputStream(); 
  tested = new Connect4TDD(new PrintStream(output)); 
}

@Test
public void whenAskedForCurrentPlayerTheOutputNotice() {
  tested.getCurrentPlayer();
  assertThat(output.toString(), containsString("Player R turn")); 
}

@Test
public void whenADiscIsIntroducedTheBoardIsPrinted() {
  int column = 1;
  tested.putDiscInColumn(column);
  assertThat(output.toString(), containsString("| |R| | | | | |"));
}
```

一种可能的实现是通过前面的测试。如您所见，类构造函数现在有一个参数。这个参数在几个方法中用于打印事件或动作描述：

```java
private static final String DELIMITER = "|";

public Connect4TDD(PrintStream out) {
  outputChannel = out;
  for (String[] row : board) Arrays.fill(row, EMPTY); 
}

public String getCurrentPlayer() {
  outputChannel.printf("Player %s turn%n", currentPlayer);
  return currentPlayer;
}

private void printBoard() {
  for (int row = ROWS - 1; row >= 0; row--) {
    StringJoiner stringJoiner = new StringJoiner(DELIMITER, DELIMITER, DELIMITER); 
    Stream.of(board[row]).forEachOrdered(stringJoiner::add); 
    outputChannel.println(stringJoiner.toString()); 
  }
}

public int putDiscInColumn(int column) {
  ... 
  printBoard();
  switchPlayer();
  return row;
} 
```

# 要求 5 - 胜利条件（I）

此要求告诉系统游戏是否结束。

当不能再插入圆盘时，游戏结束，被视为平局。

有两个条件需要测试。第一个条件是新游戏必须未完成；第二个条件是完整的棋盘游戏必须完成：

```java
@Test
public void whenTheGameStartsItIsNotFinished() {
  assertFalse("The game must not be finished", tested.isFinished()); 
} 

@Test 
public void whenNoDiscCanBeIntroducedTheGamesIsFinished() { 
  for (int row = 0; row < 6; row++)
    for (int column = 0; column < 7; column++)
      tested.putDiscInColumn(column);
    assertTrue("The game must be finished", tested.isFinished()); 
}
```

这两个测试的一个简单解决方案如下：

```java
public boolean isFinished() {
  return getNumberOfDiscs() == ROWS * COLUMNS;
}
```

# 需求 6 - 获胜条件（II）

这是玩家的第一个获胜条件要求。

如果玩家插入一个圆盘并连接他的颜色超过三个圆盘成一条垂直直线，那么该玩家获胜。

实际上，这只需要一次检查。如果当前插入的圆盘连接其他三个圆盘成一条垂直线，当前玩家就赢得了比赛：

```java
@Test
public void when4VerticalDiscsAreConnectedThenPlayerWins() {
  for (int row = 0; row < 3; row++) {
    tested.putDiscInColumn(1); // R
    tested.putDiscInColumn(2); // G
  }
  assertThat(tested.getWinner(), isEmptyString());
  tested.putDiscInColumn(1); // R
  assertThat(tested.getWinner(), is("R"));
}
```

`putDiscInColumn`方法有一些改变。还创建了一个名为`checkWinner`的新方法：

```java
private static final int DISCS_TO_WIN = 4;

private String winner = "";

private void checkWinner(int row, int column) {
  if (winner.isEmpty()) {
    String colour = board[row][column];
    Pattern winPattern =
      Pattern.compile(".*" + colour + "{" +
           DISCS_TO_WIN + "}.*");

    String vertical = IntStream
                       .range(0, ROWS)
                       .mapToObj(r -> board[r][column])
                       .reduce(String::concat).get();
    if (winPattern.matcher(vertical).matches()) 
      winner = colour;
  }
}
```

# 需求 7 - 获胜条件（III）

这是第二个获胜条件，与前一个条件非常相似。

如果玩家插入一个圆盘并连接他的颜色超过三个圆盘成一条水平直线，那么该玩家获胜。

这一次，我们试图通过将圆盘插入相邻的列来赢得比赛：

```java
@Test
public void when4HorizontalDiscsAreConnectedThenPlayerWins() {
  int column;
  for (column = 0; column < 3; column++) {
    tested.putDiscInColumn(column); // R
    tested.putDiscInColumn(column); // G
  }
  assertThat(tested.getWinner(), isEmptyString());
  tested.putDiscInColumn(column); // R
  assertThat(tested.getWinner(), is("R"));
}
```

通过这个测试的代码被放入了`checkWinners`方法中：

```java
  if (winner.isEmpty()) { 
    String horizontal = Stream
                         .of(board[row])
                         .reduce(String::concat).get();
    if (winPattern.matcher(horizontal).matches())
      winner = colour; 
  }
```

# 需求 8 - 获胜条件（IV）

最后的要求是最后的获胜条件。

如果玩家插入一个圆盘并连接他的颜色超过三个圆盘成一条对角线，那么该玩家获胜。

我们需要执行有效的游戏动作来实现这个条件。在这种情况下，我们需要测试整个棋盘上的对角线：从右上到左下，从右下到左上。以下测试使用列的列表来重新创建一个完整的游戏，以重现测试场景：

```java
@Test
public void when4Diagonal1DiscsAreConnectedThenThatPlayerWins() {
  int[] gameplay = new int[] {1, 2, 2, 3, 4, 3, 3, 4, 4, 5, 4};
  for (int column : gameplay) {
    tested.putDiscInColumn(column);
  }
  assertThat(tested.getWinner(), is("R"));
}

@Test
public void when4Diagonal2DiscsAreConnectedThenThatPlayerWins() { 
  int[] gameplay = new int[] {3, 4, 2, 3, 2, 2, 1, 1, 1, 1};
  for (int column : gameplay) {
    tested.putDiscInColumn(column);
  }
  assertThat(tested.getWinner(), is("G"));
}
```

再次，`checkWinner`方法需要修改，添加新的棋盘验证：

```java
    if (winner.isEmpty()) { 
      int startOffset = Math.min(column, row); 
      int myColumn = column - startOffset, 
        myRow = row - startOffset; 
      StringJoiner stringJoiner = new StringJoiner(""); 
      do { 
        stringJoiner .add(board[myRow++][myColumn++]); 
      } while (myColumn < COLUMNS && myRow < ROWS); 
      if (winPattern .matcher(stringJoiner.toString()).matches()) 
        winner = currentPlayer; 
    } 

    if (winner.isEmpty()) {
      int startOffset = Math.min(column, ROWS - 1 - row);
      int myColumn = column - startOffset,
        myRow = row + startOffset;
      StringJoiner stringJoiner = new StringJoiner("");
      do {
        stringJoiner.add(board[myRow--][myColumn++]);
      } while (myColumn < COLUMNS && myRow >= 0);
      if (winPattern.matcher(stringJoiner.toString()).matches())
        winner = currentPlayer; 
    } 
```

# 最后的考虑

使用 TDD，我们得到了一个构造函数，五个公共方法和六个私有方法的类。总的来说，所有方法看起来都很简单易懂。在这种方法中，我们还得到了一个检查获胜条件的大方法：`checkWinner`。优点是，通过这种方法，我们得到了一堆有用的测试，以确保未来的修改不会意外地改变方法的行为，从而可以轻松引入新的更改。代码覆盖率不是目标，但我们得到了一个非常高的百分比。

另外，为了测试目的，我们重构了类的构造函数，接受输出通道作为参数（依赖注入）。如果我们需要修改游戏状态的打印方式，这种方式将比传统方式更容易。因此，它更具可扩展性。在测试后的方法中，我们一直在滥用`System.println`方法，如果我们决定更改所有出现的内容，这将是一个非常繁琐的任务。

在大型项目中，当您发现必须为单个类创建大量测试时，这使您能够遵循单一职责原则来拆分类。由于输出打印被委托给了一个在初始化参数中传递的外部类，一个更优雅的解决方案是创建一个具有高级打印方法的类。这将使打印逻辑与游戏逻辑分离。就像下图所示的大量代码覆盖率一样，这些都是使用 TDD 进行良好设计的好处的几个例子：

![](img/e3d0f91a-75fd-4966-9fb6-643ac5ea040c.png)

这种方法的代码可在[`bitbucket.org/vfarcic/tdd-java-ch05-design.git`](https://bitbucket.org/vfarcic/tdd-java-ch05-design.git)找到。

# 总结

在本章中，我们简要讨论了软件设计和一些基本的设计原则。我们使用了传统和 TDD 两种方法来实现了一个完全功能的连四棋盘游戏。

我们分析了两种解决方案的优缺点，并使用 Hamcrest 框架来增强我们的测试。

最后，我们得出结论，良好的设计和良好的实践可以通过两种方法来实现，但 TDD 是更好的方法。

关于本章涵盖的主题的更多信息，请参考罗伯特·C·马丁所著的两本高度推荐的书籍：《代码整洁之道：敏捷软件工艺》和《敏捷软件开发：原则、模式和实践》。
