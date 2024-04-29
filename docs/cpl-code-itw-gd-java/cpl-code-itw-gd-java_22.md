# 第二十章：单元测试

作为开发人员（或软件工程师），您必须在测试领域也具备技能。例如，开发人员负责编写其代码的单元测试（例如，使用 JUnit 或 TestNG）。很可能，不包含单元测试的拉取请求也不会被接受。

在本章中，我们将涵盖单元测试面试问题，如果您申请开发人员或软件工程师等职位，可能会遇到这些问题。当然，如果您正在寻找测试人员（手动/自动化）职位，那么本章可能只代表测试的另一个视角，因此不要期望在这里看到特定于手动/自动化测试人员职位的问题。在本章中，我们将涵盖以下主题：

+   单元测试简介

+   问题和编码问题

让我们开始吧！

# 技术要求

本章中使用的代码可以在 GitHub 上找到：[`github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter18`](https://github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter18)

# 单元测试简介

测试应用程序的过程包含几个测试层。其中之一是*单元测试*层。

主要的，一个应用程序是由称为单元的小功能部分构建的（例如，一个常规的 Java 方法可以被认为是一个单元）。测试这些单元在特定输入/条件/约束下的功能和正确性称为单元测试。

这些单元测试是由开发人员使用源代码和测试计划编写的。理想情况下，每个开发人员都应该能够编写测试/验证其代码的单元测试。单元测试应该是有意义的，并提供被接受的代码覆盖率。

如果单元测试失败，那么开发人员负责修复问题并再次执行单元测试。以下图表描述了这一陈述：

![图 18.1 – 单元测试流程](img/Figure_18.1_B15403.jpg)

图 18.1 – 单元测试流程

单元测试使用**单元测试用例**。*单元测试用例*是一对输入数据和预期输出，用于塑造对某个功能的测试。

如果您参加的面试要求了解单元测试，如果被问及功能测试和/或集成测试的问题，不要感到惊讶。因此，最好准备好这些问题的答案。

功能测试是基于给定的输入和产生的输出（行为）来测试功能要求，需要将其与预期输出（行为）进行比较。每个功能测试都使用功能规范来验证表示该功能要求实现的组件（或一组组件）的正确性。这在下图中有解释：

![图 18.2 – 功能测试](img/Figure_18.2_B15403.jpg)

图 18.2 – 功能测试

**集成测试**的目标是在软件组件被迭代增量地集成时发现缺陷。换句话说，已经进行单元测试的模块被集成（分组或聚合）并按照集成计划进行测试。这在下图中有所描述：

![图 18.3 – 集成测试](img/Figure_18.3_B15403.jpg)

图 18.3 – 集成测试

关于单元测试和集成测试的问题经常被问及面试候选人，问题是突出这两者之间的主要区别。以下表格将帮助您准备回答这个问题：

![图 18.4 – 单元测试和集成测试的比较](img/Figure_18.4_B15403.jpg)

图 18.4 – 单元测试和集成测试的比较

一个好的测试人员能够在不做任何关于输入的假设或约束的情况下对测试对象进行压力测试和滥用。这也适用于单元测试。现在我们已经涉及了单元测试，让我们来看看一些关于单元测试的编码挑战和问题。

# 问题和编码挑战

在这一部分，我们将涵盖与单元测试相关的 15 个问题和编码挑战，这在面试中非常受欢迎。让我们开始吧！

## 编码挑战 1 - AAA

**问题**：单元测试中的 AAA 是什么？

**解决方案**：AAA 首字母缩写代表[**A**]rrange，[**A**]ct，[**A**]ssert，它代表一种构造测试的方法，以维持清晰的代码和可读性。今天，AAA 是一种几乎成为行业标准的测试模式。以下代码片段说明了这一点：

```java
@Test
public void givenStreamWhenSumThenEquals6() {
  // Arrange
  Stream<Integer> theStream = Stream.of(1, 2, 3);
  // Act
  int sum = theStream.mapToInt(i -> i).sum();
  // Assert
  assertEquals(6, sum);
}
```

**安排**部分：在这一部分，我们准备或设置测试。例如，在前面的代码中，我们准备了一个整数流，其中的元素是 1、2 和 3。

**行动**部分：在这一部分，我们执行必要的操作以获得测试的结果。例如，在前面的代码中，我们对流的元素求和，并将结果存储在一个整数变量中。

**断言**部分：在这一部分，我们检查单元测试的结果是否与预期结果相匹配。这是通过断言来完成的。例如，在前面的代码中，我们检查元素的总和是否等于 6。

你可以在名为*junit5/ArrangeActAssert*的应用程序中找到这段代码。

## 编码挑战 2 - FIRST

**问题**：单元测试中的**FIRST**是什么？

**解决方案**：好的测试人员使用 FIRST 来避免在单元测试中遇到的许多问题。FIRST 首字母缩写代表[**F**]ast，[**I**]solated，[**R**]epeatable，[**S**]elf-validating，[**T**]imely。让我们看看它们各自的含义：

**快速**：建议编写运行快速的单元测试。快速是一个依赖于你有多少单元测试、你多频繁运行它们以及你愿意等待它们运行多长时间的任意概念。例如，如果每个单元测试的平均完成时间为 200 毫秒，你运行 5000 个单元测试，那么你将等待约 17 分钟。通常，单元测试很慢，因为它们访问外部资源（例如数据库和文件）。

**隔离**：理想情况下，你应该能够随时以任何顺序运行任何测试。如果你的单元测试是隔离的，并且专注于小代码片段，这是可能的。良好的单元测试不依赖于其他单元测试，但这并不总是可实现的。尽量避免依赖链，因为当出现问题时它们是有害的，你将不得不进行调试。

**可重复**：单元测试应该是可重复的。这意味着单元测试的断言每次运行时都应该产生相同的结果。换句话说，单元测试不应该依赖于可能给断言引入可变结果的任何东西。

**自我验证**：单元测试应该是自我验证的。这意味着你不应该手动验证测试的结果。这是耗时的，并且会显示断言没有完成它们的工作。努力编写断言，使它们按预期工作。

及时：重要的是不要推迟编写单元测试。你推迟得越久，面对的缺陷就会越多。你会发现自己找不到时间回来编写单元测试。想想如果我们不断推迟倒垃圾会发生什么。我们推迟得越久，拿出来就会越困难，我们的健康也会受到风险。我有没有提到气味？所以，及时地编写单元测试。这是一个好习惯！

## 编码挑战 3 - 测试夹具

**问题**：什么是测试夹具？

**解决方案**：通过测试夹具，我们指的是任何存在于测试之外并用于设置应用程序的测试数据，以便它处于固定状态。应用程序的固定状态允许对其进行测试，并且处于一个恒定和已知的环境中。

## 编码挑战 4-异常测试

**问题**：在 JUnit 中测试异常的常见方法有哪些？

`try`/`catch`习语，`@Test`的`expected`元素，以及通过`ExpectedException`规则。

`try`/`catch`习语在 JUnit 3.x 中盛行，并且可以如下使用：

```java
@Test
public void givenStreamWhenGetThenException() {
  Stream<Integer> theStream = Stream.of();
  try {
    theStream.findAny().get();
    fail("Expected a NoSuchElementException to be thrown");
  } catch (NoSuchElementException ex) {
    assertThat(ex.getMessage(), is("No value present"));
  }
}
```

由于`fail()`抛出`AssertionError`，它不能用来测试这种错误类型。

从 JUnit 4 开始，我们可以使用`@Test`注解的`expected`元素。该元素的值是预期异常的类型（`Throwable`的子类）。查看以下示例，该示例使用了`expected`：

```java
@Test(expected = NoSuchElementException.class)
public void givenStreamWhenGetThenException() {
  Stream<Integer> theStream = Stream.of();
  theStream.findAny().get();
}
```

只要您不想测试异常消息的值，这种方法就可以。此外，请注意，如果任何代码行抛出`NoSuchElementException`，则测试将通过。您可能期望此异常是由特定代码行引起的，而实际上可能是由其他代码引起的。

另一种方法依赖于`ExpectedException`规则。从 JUnit 4.13 开始，此方法已被弃用。让我们看看代码：

```java
@Rule
public ExpectedException thrown = ExpectedException.none();
@Test
public void givenStreamWhenGetThenException() 
    throws NoSuchElementException {
  Stream<Integer> theStream = Stream.of();
  thrown.expect(NoSuchElementException.class);
  thrown.expectMessage("No value present");
  theStream.findAny().get();
}
```

通过这种方法，您可以测试异常消息的值。这些示例已被分组到一个名为*junit4/TestingExceptions*的应用程序中。

从 JUnit5 开始，我们可以使用两种方法来测试异常。它们都依赖于`assertThrows()`方法。此方法允许我们断言给定的函数调用（作为 lambda 表达式甚至作为方法引用传递）导致抛出预期类型的异常。以下示例不言自明：

```java
@Test
public void givenStreamWhenGetThenException() {
  assertThrows(NoSuchElementException.class, () -> {
    Stream<Integer> theStream = Stream.of();
    theStream.findAny().get();
  });
}
```

这个例子只验证了异常的类型。但是，由于异常已被抛出，我们可以断言抛出异常的更多细节。例如，我们可以断言异常消息的值如下：

```java
@Test
public void givenStreamWhenGetThenException() {
  Throwable ex = assertThrows(
    NoSuchElementException.class, () -> {
      Stream<Integer> theStream = Stream.of();
      theStream.findAny().get();
    });
  assertEquals(ex.getMessage(), "No value present");
}
```

只需使用`ex`对象来断言您认为从`Throwable`中有用的任何内容。每当您不需要断言有关异常的详细信息时，请依靠`assertThrows()`，而不捕获返回。这两个示例已被分组到一个名为*junit5/TestingExceptions*的应用程序中。

## 编码挑战 5-开发人员还是测试人员

**问题**：谁应该使用 JUnit-开发人员还是测试人员？

**解决方案**：通常，JUnit 由开发人员用于编写 Java 中的单元测试。编写单元测试是测试应用程序代码的编码过程。JUnit 不是一个测试过程。但是，许多测试人员愿意学习并使用 JUnit 进行单元测试。

## 编码挑战 6-JUnit 扩展

**问题**：您知道/使用哪些有用的 JUnit 扩展？

**解决方案**：最常用的 JUnit 扩展是 JWebUnit（用于 Web 应用程序的基于 Java 的测试框架）、XMLUnit（用于测试 XML 的单个 JUnit 扩展类）、Cactus（用于测试服务器端 Java 代码的简单测试框架）和 MockObject（模拟框架）。您需要对这些扩展中的每一个都说几句话。

## 编码挑战 7-@Before*和@After*注释

您知道/使用哪些`@Before*`/`@After*`注释？

`@Before`，`@BeforeClass`，`@After`和`@AfterClass`。

在每个测试之前执行方法时，我们使用`@Before`注解对其进行注释。这对于在运行测试之前执行常见的代码片段非常有用（例如，我们可能需要在每个测试之前执行一些重新初始化）。在每个测试之后清理舞台时，我们使用`@After`注解对方法进行注释。

当仅在所有测试之前执行一次方法时，我们使用`@BeforeClass`注解对其进行注释。该方法必须是`static`的。这对于全局和昂贵的设置非常有用，例如打开到数据库的连接。在所有测试完成后清理舞台时，我们使用`@AfterClass`注解对一个`static`方法进行注释；例如，关闭数据库连接。

您可以在名为*junit4/BeforeAfterAnnotations*的简单示例中找到一个简单的示例。

从 JUnit5 开始，我们有`@BeforeEach`作为`@Before`的等效项，`@BeforeAll`作为`@BeforeClass`的等效项。实际上，`@Before`和`@BeforeClass`被重命名为更具指示性的名称，以避免混淆。

您可以在名称为*junit5/BeforeAfterAnnotations*的简单示例中找到这个。

## 编码挑战 8 - 模拟和存根

**问题**：模拟和存根是什么？

**解决方案**：模拟是一种用于创建模拟真实对象的对象的技术。这些对象可以预先编程（或预设或预配置）期望，并且我们可以检查它们是否已被调用。在最广泛使用的模拟框架中，我们有 Mockito 和 EasyMock。

存根类似于模拟，只是我们无法检查它们是否已被调用。存根预先配置为使用特定输入产生特定输出。

## 编码挑战 9 - 测试套件

**问题**：什么是测试套件？

**解决方案**：测试套件是将多个测试聚合在多个测试类和包中，以便它们一起运行的概念。

在 JUnit4 中，我们可以通过`org.junit.runners.Suite`运行器和`@SuiteClasses(...)`注解来定义测试套件。例如，以下代码片段是一个聚合了三个测试（`TestConnect.class`，`TestHeartbeat.class`和`TestDisconnect.class`）的测试套件：

```java
@RunWith(Suite.class)
@Suite.SuiteClasses({
  TestConnect.class,
  TestHeartbeat.class,
  TestDisconnect.class
})
public class TestSuite {
    // this class was intentionally left empty
}
```

完整的代码称为*junit4/TestSuite*。

在 JUnit5 中，我们可以通过`@SelectPackages`和`@SelectClasses`注解来定义测试套件。

`@SelectPackages`注解对于从不同包中聚合测试非常有用。我们只需要指定包的名称，如下例所示：

```java
@RunWith(JUnitPlatform.class)
@SuiteDisplayName("TEST LOGIN AND CONNECTION")
@SelectPackages({
  "coding.challenge.connection.test",
  "coding.challenge.login.test"
})
public class TestLoginSuite {
  // this class was intentionally left empty
}
```

`@SelectClasses`注解对于通过类名聚合测试非常有用：

```java
@RunWith(JUnitPlatform.class)
@SuiteDisplayName("TEST CONNECTION")
@SelectClasses({
  TestConnect.class, 
  TestHeartbeat.class, 
  TestDisconnect.class
})
public class TestConnectionSuite {
  // this class was intentionally left empty
}
```

完整的代码称为*junit5/TestSuite*。

此外，可以通过以下注解来过滤测试包、测试类和测试方法：

+   过滤包：`@IncludePackages`和`@ExcludePackages`

+   过滤测试类：`@IncludeClassNamePatterns`和`@ExcludeClassNamePatterns`

+   过滤测试方法：`@IncludeTags`和`@ExcludeTags`

## 编码挑战 10 - 忽略测试方法

**问题**：如何忽略测试？

`@Ignore`注解。在 JUnit5 中，我们可以通过`@Disable`注解做同样的事情。

忽略测试方法在我们预先编写了一些测试并且希望在运行当前测试时不运行这些特定测试时是有用的。

## 编码挑战 11 - 假设

**问题**：什么是假设？

**解决方案**：假设用于执行测试，如果满足指定条件，则使用假设。它们通常用于处理测试执行所需的外部条件，但这些条件不在我们的控制范围之内，或者与被测试的内容不直接相关。

在 JUnit4 中，假设是可以在`org.junit.Assume`包中找到的`static`方法。在这些假设中，我们有`assumeThat()`，`assumeTrue()`和`assumeFalse()`。以下代码片段举例说明了`assumeThat()`的用法：

```java
@Test
public void givenFolderWhenGetAbsolutePathThenSuccess() {
  assumeThat(File.separatorChar, is('/'));
  assertThat(new File(".").getAbsolutePath(),
    is("C:/SBPBP/GitHub/Chapter18/junit4"));
}
```

如果`assumeThat()`不满足给定条件，则测试将被跳过。完整的应用程序称为*junit4/Assumptions*。

在 JUnit5 中，假设是可以在`org.junit.jupiter.api.Assumptions`包中找到的`static`方法。在这些假设中，我们有`assumeThat()`，`assumeTrue()`和`assumeFalse()`。所有三种都有不同的用法。以下代码片段举例说明了`assumeThat()`的用法：

```java
@Test
public void givenFolderWhenGetAbsolutePathThenSuccess() {
  assumingThat(File.separatorChar == '/',
   () -> {
     assertThat(new File(".").getAbsolutePath(), 
       is("C:/SBPBP/GitHub/Chapter18/junit5"));
   });
   // run these assertions always, just like normal test
   assertTrue(true);
}
```

请注意，测试方法（`assertThat()`）只有在满足假设时才会执行。lambda 之后的所有内容都将被执行，而不管假设的有效性如何。完整的应用程序称为*junit5/Assumptions*。

## 编码挑战 12 - @Rule

`@Rule`？

**解决方案**：JUnit 通过所谓的*规则*提供了高度的灵活性。规则允许我们创建和隔离对象（代码），并在多个测试类中重用这些代码。主要是通过可重用的规则增强测试。JUnit 提供了内置规则和可以用来编写自定义规则的 API。

## 编码挑战 13 - 方法测试返回类型

在 JUnit 测试方法中使用`void`？

将`void`转换为其他内容，但 JUnit 不会将其识别为测试方法，因此在测试执行期间将被忽略。

## 编码挑战 14 - 动态测试

**问题**：我们能在 JUnit 中编写动态测试（在运行时生成的测试）吗？

`@Test`是在编译时完全定义的静态测试。JUnit5 引入了动态测试 - 动态测试是在运行时生成的。

动态测试是通过一个工厂方法生成的，这个方法使用`@TestFactory`注解进行注释。这样的方法可以返回`DynamicTest`实例的`Iterator`、`Iterable`、`Collection`或`Stream`。工厂方法没有被`@Test`注解，并且不是`private`或`static`。此外，动态测试不能利用生命周期回调（例如，`@BeforeEach`和`@AfterEach`会被忽略）。

让我们看一个简单的例子：

```java
1: @TestFactory
2: Stream<DynamicTest> dynamicTestsExample() {
3:
4:   List<Integer> items = Arrays.asList(1, 2, 3, 4, 5);
5:
6:   List<DynamicTest> dynamicTests = new ArrayList<>();
7:
8:   for (int item : items) {
9:     DynamicTest dynamicTest = dynamicTest(
10:        "pow(" + item + ", 2):", () -> {
11:        assertEquals(item * item, Math.pow(item, 2));
12:    });
13:    dynamicTests.add(dynamicTest);
14:  }
15:
16:  return dynamicTests.stream();
17: }
```

现在，让我们指出主要的代码行：

`@TestFactory`注解来指示 JUnit5 这是一个动态测试的工厂方法。

`Stream<DynamicTest>`。

**4**：我们测试的输入是一个整数列表。对于每个整数，我们生成一个动态测试。

`List<DynamicTest>`。在这个列表中，我们添加每个生成的测试。

**8-12**：我们为每个整数生成一个测试。每个测试都有一个名称和包含必要断言的 lambda 表达式。

**13**：我们将生成的测试存储在适当的列表中。

测试的`Stream`。

运行这个测试工厂将产生五个测试。完整的例子被称为*junit5/TestFactory*。

## 编码挑战 15 - 嵌套测试

**问题**：我们能在 JUnit5 中编写嵌套测试吗？

`@Nested`注解。实际上，我们创建了一个嵌套测试类层次结构。这个层次结构可能包含设置、拆卸和测试方法。然而，我们必须遵守一些规则，如下：

+   嵌套测试类使用`@Nested`注解进行注释。

+   嵌套测试类是非`static`的内部类。

+   嵌套测试类可以包含一个`@BeforeEach`方法，一个`@AfterEach`方法和测试方法。

+   内部类中不允许使用`static`成员，这意味着嵌套测试中不能使用`@BeforeAll`和`@AfterAll`方法。

+   类层次结构的深度是无限的。

嵌套测试的一些示例代码可以在这里看到：

```java
@RunWith(JUnitPlatform.class)
public class NestedTest {
  private static final Logger log 
    = Logger.getLogger(NestedTest.class.getName());
  @DisplayName("Test 1 - not nested")
  @Test
  void test1() {
    log.info("Execute test1() ...");
  }
  @Nested
  @DisplayName("Running tests nested in class A")
  class A {
    @BeforeEach
    void beforeEach() {
      System.out.println("Before each test 
        method of the A class");
    }
    @AfterEach
    void afterEach() {
      System.out.println("After each test 
        method of the A class");
    }
    @Test
    @DisplayName("Test2 - nested in class A")
    void test2() {
      log.info("Execute test2() ...");
    }
  }
}
```

完整的例子被称为*junit5/NestedTests*。

# 总结

在本章中，我们涵盖了关于通过 JUnit4 和 JUnit5 进行单元测试的几个热门问题和编码挑战。不要忽视这个话题是很重要的。很可能，在 Java 开发人员或软件工程师职位的面试的最后部分，你会得到一些与测试相关的问题。此外，这些问题将与单元测试和 JUnit 相关。

在下一章中，我们将讨论与扩展和扩展相关的面试问题。
