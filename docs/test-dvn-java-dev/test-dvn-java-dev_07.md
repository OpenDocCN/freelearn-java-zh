# 第七章：TDD 和函数式编程-完美匹配

“任何足够先进的技术都是不可区分的魔术。”

-阿瑟·C·克拉克

到目前为止，我们在本书中看到的所有代码示例都遂循了一种特定的编程范式：**面向对象编程**（**OOP**）。这种范式已经垄断了软件行业很长时间，大多数软件公司都采用了 OOP 作为标准的编程方式。

然而，OOP 成为最常用的范式并不意味着它是唯一存在的范式。事实上，还有其他值得一提的范式，但本章将只关注其中之一：函数式编程。此外，本书的语言是 Java，因此所有的代码片段和示例都将基于 Java 8 版本中包含的函数式 API。

本章涵盖的主题包括：

+   Optional 类

+   函数的再思考

+   流

+   将 TDD 应用于函数式编程

# 设置环境

为了以测试驱动的方式探索 Java 函数式编程的一些好处，我们将使用 JUnit 和 AssertJ 框架设置一个 Java 项目。后者包含了一些方便的`Optional`方法。

让我们开始一个新的 Gradle 项目。这就是`build.gradle`的样子：

```java
apply plugin: 'java'

sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
  mavenCentral()
}

dependencies {
  testCompile group: 'junit', name: 'junit', version: '4.12'
  testCompile group: 'org.assertj', name: 'assertj-core', version: '3.9.0'
}
```

在接下来的章节中，我们将探索 Java 8 中包含的一些增强编程体验的实用程序和类。它们大多不仅适用于函数式编程，甚至可以在命令式编程中使用。

# Optional - 处理不确定性

自从创建以来，`null`已经被开发人员无数次在无数个程序中使用和滥用。`null`的常见情况之一是代表值的缺失。这一点一点也不方便；它既可以代表值的缺失，也可以代表代码片段的异常执行。

此外，为了访问可能为`null`的变量，并减少不希望的运行时异常，比如`NullPointerException`，开发人员倾向于用`if`语句包装变量，以便以安全模式访问这些变量。虽然这样做是有效的，但对空值的保护增加了一些与代码的功能或目标无关的样板代码：

```java
if (name != null) {
  // do something with name
}
```

前面的代码克服了`null`的创造者在 2009 年的一次会议上所发现的问题：

“我称它为我的十亿美元的错误。这是在 1965 年发明了空引用。那时，我正在设计第一个综合的面向对象语言（ALGOL W）的引用类型系统。我的目标是确保所有引用的使用都是绝对安全的，由编译器自动执行检查。但我无法抵制放入一个空引用的诱惑，因为它太容易实现了。这导致了无数的错误、漏洞和系统崩溃，这可能在过去四十年中造成了十亿美元的痛苦和损害。”

-托尼·霍尔

随着 Java 8 的发布，实用类`Optional`作为替代前面的代码块被包含了进来。除了其他好处，它还带来了编译检查和零样板代码。让我们通过一个简单的例子来看`Optional`的实际应用。

# Optional 的示例

作为`Optional`的演示，我们将创建一个内存中的学生存储库。这个存储库有一个按`name`查找学生的方法，为了方便起见，将被视为 ID。该方法返回的值是`Optional<Student>`；这意味着响应可能包含也可能不包含`Student`。这种模式基本上是`Optional`的常见情况之一。

此时，读者应该熟悉 TDD 过程。为了简洁起见，完整的红-绿-重构过程被省略了。测试将与实现一起按照方便的顺序呈现，这可能与 TDD 迭代的顺序不一致。

首先，我们需要一个`Student`类来表示我们系统中的学生。为了简单起见，我们的实现将非常基本，只有两个参数：学生的`name`和`age`：

```java
public class Student {
  public final String name;
  public final int age;

  public Student(String name, int age) {
    this.name = name;
    this.age = age;
  }
}
```

下一个测试类验证了两种情况：成功的查找和失败的查找。请注意，AssertJ 对`Optional`有一些有用且有意义的断言方法。这使得测试非常流畅和可读：

```java
public class StudentRepositoryTest {

  private List<Student> studentList = Arrays.asList(
    new Student("Jane", 23),
    new Student("John", 21),
    new Student("Tom", 25) 
  );

  private StudentRepository studentRepository = 
    new StudentRepository(studentList);

  @Test
  public void whenStudentIsNotFoundThenReturnEmpty() {
    assertThat(studentRepository.findByName("Samantha"))
      .isNotPresent();
  }

  @Test
  public void whenStudentIsFoundThenReturnStudent() {
    assertThat(studentRepository.findByName("John"))
      .isPresent();
  }
}
```

在某些情况下，仅验证具有该`name`的学生的存在是不够的，我们可以对返回的对象执行一些断言。在大多数情况下，这是正确的做法：

```java
@Test
public void whenStudentIsFoundThenReturnStudent() {
  assertThat(studentRepository.findByName("John"))
    .hasValueSatisfying(s -> {
      assertThat(s.name).isEqualTo("John");
      assertThat(s.age).isEqualTo(21);
    });
}
```

现在，是时候专注于`StudentRepository`类了，它只包含一个构造函数和执行学生查找的方法。如下所示，查找方法`findByName`返回一个包含`Student`的`Optional`。请注意，这是一个有效但不是功能性的实现，只是作为一个起点使用：

```java
public class StudentRepository {
  StudentRepository(Collection<Student> students) { }

  public Optional<Student> findByName(String name) {
    return Optional.empty();
  }
}
```

如果我们对前面的实现运行测试，我们会得到一个成功的测试，因为查找方法默认返回`Optional.empty()`。另一个测试会抛出一个错误，如下所示：

```java
java.lang.AssertionError: 
Expecting Optional to contain a value but was empty.
```

为了完整起见，这是一个可能的实现之一：

```java
public class StudentRepository {
  private final Set<Student> studentSet;

  StudentRepository(Collection<Student> students) {
    studentSet = new HashSet<>(students);
  }

  public Optional<Student> findByName(String name) {
    for (Student student : this.studentSet) {
      if (student.name.equals(name))
        return Optional.of(student);
    }
    return Optional.empty();
  }
}
```

在下一节中，我们将看到对*函数*的不同观点。在 Java 8 中，如果以特定方式使用函数，它们将增加一些额外的功能。我们将通过一些示例来探索其中的一些功能。

# 重新审视函数

与面向对象的程序不同，以函数式方式编写的程序不持有任何可变状态。相反，代码由接受参数并返回值的函数组成。因为没有涉及可以改变执行的内部状态或副作用，所有函数都是确定性的。这是一个非常好的特性，因为它意味着对于相同参数的同一函数的不同执行将产生相同的结果。

以下片段说明了一个不会改变任何内部状态的函数：

```java
public Integer add(Integer a, Integer b) {
  return a + b;
}
```

以下是使用 Java 的函数式 API 编写的相同函数：

```java
public final BinaryOperator<Integer> add =
  new BinaryOperator<Integer>() {

    @Override
    public Integer apply(Integer a, Integer b) {
      return a + b;
    }
  };
```

第一个例子对于任何 Java 开发人员来说应该是非常熟悉的；它遵循了以两个整数作为参数并返回它们的和的常见语法。然而，第二个例子与我们习惯的传统代码有些不同。在这个新版本中，函数是一个作为值的对象，并且可以分配给一个字段。在某些情况下，这是非常方便的，因为它仍然可以在某些情况下用作函数，在其他情况下也可以用作返回值，在函数中作为参数或在类中作为字段。

有人可能会认为第一个版本的函数更合适，因为它更短，不需要创建一个新对象。嗯，这是真的，但函数也可以是对象，增强了它们的一系列新功能。就代码冗长而言，可以通过使用 lambda 表达式将其大大减少到一行：

```java
public final BinaryOperator<Integer> addLambda = (a, b) -> a + b;
```

在下一节中，将介绍**逆波兰表示法**（**RPN**）的一个可能解决方案。我们将使用函数式编程的强大和表现力，特别是 lambda 表示法，在需要函数作为某些函数的参数时变得非常方便。使用 lambda 使我们的代码非常简洁和优雅，提高了可读性。

# Kata - 逆波兰表示法

RPN 是用于表示数学表达式的一种表示法。它在运算符和操作数的顺序上与传统和广泛使用的中缀表示法不同。

在中缀表示法中，运算符放置在操作数之间，而在 RPN 中，操作数首先放置，运算符位于末尾。

这是使用中缀表示法编写的表达式：

```java
3 + 4
```

使用 RPN 编写的相同表达式：

```java
3 4 +
```

# 要求

我们将忽略如何读取表达式，以便我们可以专注于解决问题。此外，我们将仅使用正整数来简化问题，尽管接受浮点数或双精度数也不应该很困难。为了解决这个 kata，我们只需要满足以下两个要求：

+   对于无效输入（不是 RPN），应抛出错误消息

+   它接收使用 RPN 编写的算术表达式并计算结果

以下代码片段是我们开始项目的一个小脚手架：

```java
public class ReversePolishNotation {
  int compute(String expression) {
    return 0;
  }
}

public class NotReversePolishNotationError extends RuntimeException {
  public NotReversePolishNotationError() {
    super("Not a Reverse Polish Notation");
  }
}
```

以前面的代码片段作为起点，我们将继续进行，将要求分解为更小的规范，可以逐个解决。

# 要求 - 处理无效输入

鉴于我们的实现基本上什么都没做，我们将只专注于一件事 - 读取单个操作数。如果输入是单个数字（没有运算符），那么它是有效的逆波兰表示法，并返回数字的值。除此以外的任何内容目前都被视为无效的 RPN。

这个要求被转化为这四个测试：

```java
public class ReversePolishNotationTest {
  private ReversePolishNotation reversePolishNotation =
    new ReversePolishNotation();

  @Test(expected = NotReversePolishNotationError.class)
  public void emptyInputThrowsError() {
    reversePolishNotation.compute("");
  }

  @Test(expected = NotReversePolishNotationError.class)
  public void notANumberThrowsError() {
    reversePolishNotation.compute("a");
  }

  @Test
  public void oneDigitReturnsNumber() {
    assertThat(reversePolishNotation.compute("7")).isEqualTo(7);
  }

  @Test
  public void moreThanOneDigitReturnsNumber() {
    assertThat(reversePolishNotation.compute("120")).isEqualTo(120);
  }
}
```

当提供无效输入时，我们现在要求我们的`compute`方法抛出`IllegalArgumentException`。在任何其他情况下，它将作为整数值返回数字。可以通过以下代码行实现：

```java
public class ReversePolishNotation {
  int compute(String expression) {
    try {
      return (Integer.parseInt(expression));
    } catch (NumberFormatException e) {
      throw new NotReversePolishNotationError();
    }
  }
}
```

这个要求已经实现。另一个要求更复杂一些，所以我们将其分为两个部分 - 单一操作，意味着只有一个操作，和复杂操作，涉及任何类型的多个操作。

# 要求 - 单一操作

因此，计划是支持加法、减法、乘法和除法操作。如在 kata 演示中所解释的，在 RPN 中，运算符位于表达式的末尾。

这意味着*a - b*表示为*a b -*，其他运算符也是如此：加法*+*，乘法*，和除法*/*。

让我们在测试中添加每个支持的操作中的一个：

```java
@Test
public void addOperationReturnsCorrectValue() {
  assertThat(reversePolishNotation.compute("1 2 +")).isEqualTo(3);
}

@Test
public void subtractOperationReturnsCorrectValue() {
  assertThat(reversePolishNotation.compute("2 1 -")).isEqualTo(1);
}

@Test
public void multiplyOperationReturnsCorrectValue() {
  assertThat(reversePolishNotation.compute("2 1 *")).isEqualTo(2);
}

@Test
public void divideOperationReturnsCorrectValue() {
  assertThat(reversePolishNotation.compute("2 2 /")).isEqualTo(1);
}
```

这还包括必要的更改，使它们成功通过。行为基本上是将运算符放在表达式之间，并在输入表达式时执行操作。如果`expression`中只有一个元素，则适用前面的规则：

```java
int compute(String expression) {
  String[] elems = expression.trim().split(" ");
  if (elems.length != 1 && elems.length != 3)
    throw new NotReversePolishNotationError();
  if (elems.length == 1) {
    return parseInt(elems[0]);
  } else {
    if ("+".equals(elems[2]))
      return parseInt(elems[0]) + parseInt(elems[1]);
    else if ("-".equals(elems[2]))
      return parseInt(elems[0]) - parseInt(elems[1]);
    else if ("*".equals(elems[2]))
      return parseInt(elems[0]) * parseInt(elems[1]);
    else if ("/".equals(elems[2]))
      return parseInt(elems[0]) / parseInt(elems[1]);
    else
      throw new NotReversePolishNotationError();
  }
}
```

`parseInt`是一个`private`方法，用于解析输入并返回整数值或抛出异常：

```java
private int parseInt(String number) {
  try {
    return Integer.parseInt(number);
  } catch (NumberFormatException e) {
    throw new NotReversePolishNotationError();
  }
}
```

下一个要求是魔术发生的地方。我们将支持`expression`中的多个操作。

# 要求 - 复杂操作

复杂的操作很难处理，因为混合操作使得非受过训练的人眼难以理解操作应该以何种顺序进行。此外，不同的评估顺序通常会导致不同的结果。为了解决这个问题，逆波兰表达式的计算由队列的实现支持。以下是我们下一个功能的一些测试：

```java
@Test
public void multipleAddOperationsReturnCorrectValue() {
  assertThat(reversePolishNotation.compute("1 2 5 + +"))
    .isEqualTo(8);
}

@Test
public void multipleDifferentOperationsReturnCorrectValue() {
  assertThat(reversePolishNotation.compute("5 12 + 3 -"))
    .isEqualTo(14);
}

@Test
public void aComplexTest() {
  assertThat(reversePolishNotation.compute("5 1 2 + 4 * + 3 -"))
    .isEqualTo(14);
}
```

计算应该按顺序将表达式中的数字或操作数堆叠在 Java 中的队列或堆栈中。如果在任何时候找到运算符，则堆栈将用应用该运算符于这些值的结果替换顶部的两个元素。为了更好地理解，逻辑将被分成不同的函数。

首先，我们将定义一个函数，该函数接受一个堆栈和一个操作，并将该函数应用于顶部的前两个项目。请注意，由于堆栈的实现，第一次检索第二个操作数：

```java
private static void applyOperation(
    Stack<Integer> stack,
    BinaryOperator<Integer> operation
) {
  int b = stack.pop(), a = stack.pop();
  stack.push(operation.apply(a, b));
}
```

下一步是创建程序必须处理的所有函数。对于每个运算符，都定义了一个函数作为对象。这有一些优势，比如更好的隔离测试。在这种情况下，单独测试函数可能没有意义，因为它们是微不足道的，但在一些其他场景中，单独测试这些函数的逻辑可能非常有用：

```java
static BinaryOperator<Integer> ADD = (a, b) -> a + b;
static BinaryOperator<Integer> SUBTRACT = (a, b) -> a - b;
static BinaryOperator<Integer> MULTIPLY = (a, b) -> a * b;
static BinaryOperator<Integer> DIVIDE = (a, b) -> a / b;
```

现在，将所有部分放在一起。根据我们找到的运算符，应用适当的操作：

```java
int compute(String expression) {
  Stack<Integer> stack = new Stack<>();
  for (String elem : expression.trim().split(" ")) {
    if ("+".equals(elem))
      applyOperation(stack, ADD);
    else if ("-".equals(elem))
      applyOperation(stack, SUBTRACT);
    else if ("*".equals(elem))
      applyOperation(stack, MULTIPLY);
    else if ("/".equals(elem))
      applyOperation(stack, DIVIDE);
    else {
      stack.push(parseInt(elem));
    }
  }
  if (stack.size() == 1) return stack.pop();
  else throw new NotReversePolishNotationError();
}
```

代码可读性很强，非常容易理解。此外，这种设计允许通过轻松添加对其他不同操作的支持来扩展功能。

对于读者来说，将模数（*％*）操作添加到提供的解决方案可能是一个很好的练习。

另一个很好的例子是 lambda 完全适合的 Streams API，因为大多数函数都有一个名副其实的名称，如`filter`、`map`或`reduce`等。让我们在下一节更深入地探讨这一点。

# 流

Java 8 中包含的顶级实用程序之一是 Streams。在本章中，我们将在小的代码片段中使用 lambda 与 Streams 结合，并创建一个测试来验证它们。

为了更好地理解 Streams 是什么，该做什么，以及不该做什么，强烈建议阅读 Oracle 的 Stream 页面。一个很好的起点是[`docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html`](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)。

长话短说，流提供了一堆设施来处理可以以并行或顺序顺序执行的长计算。并行编程超出了本书的范围，因此下一个示例将仅顺序执行。此外，为了保持本章简洁，我们将专注于：

+   `filter`

+   `映射`

+   `flatMap`

+   `reduce`

# 过滤

让我们从`filter`操作开始。Filters 是一个名副其实的函数；它根据值是否满足条件来过滤流中的元素，如下例所示：

```java
@Test
public void filterByNameReturnsCollectionFiltered() {
  List<String> names = Arrays.asList("Alex", "Paul", "Viktor",
         "Kobe", "Tom", "Andrea");
  List<String> filteredNames = Collections.emptyList();

  assertThat(filteredNames)
      .hasSize(2)
      .containsExactlyInAnyOrder("Alex", "Andrea");
}
```

计算`filteredNames`列表的一种可能性如下：

```java
List<String> filteredNames = names.stream()
      .filter(name -> name.startsWith("A"))
      .collect(Collectors.toList());
```

那个是最简单的。简而言之，`filter`过滤输入并返回一个值，而不是过滤掉所有的元素。使用 lambda 使得代码优雅且易于阅读。

# 映射

`map`函数将流中的所有元素转换为另一个。结果对象可以与输入共享类型，但也可以返回不同类型的对象：

```java
@Test
public void mapToUppercaseTransformsAllElementsToUppercase() {
  List<String> names = Arrays.asList("Alex", "Paul", "Viktor");
  List<String> namesUppercase = Collections.emptyList();

  assertThat(namesUppercase)
      .hasSize(3)
      .containsExactly("ALEX", "PAUL", "VIKTOR");
}
```

`namesUppercase`列表应按以下方式计算：

```java
List<String> namesUppercase = names.stream()
  .map(String::toUpperCase)
  .collect(Collectors.toList());
```

注意`toUpperCase`方法的调用。它属于 Java 类`String`，只能通过引用函数和函数所属的类在该场景中使用。在 Java 中，这称为**方法引用**。

# flatMap

`flatMap`函数与`map`函数非常相似，但当操作可能返回多个值并且我们想保持单个元素流时使用它。在`map`的情况下，将返回一个集合流。让我们看看`flatMap`的使用：

```java
@Test
public void gettingLettersUsedInNames() {
  List<String> names = Arrays.asList("Alex", "Paul", "Viktor");
  List<String> lettersUsed = Collections.emptyList();

  assertThat(lettersUsed)
    .hasSize(12)
    .containsExactly("a","l","e","x","p","u","v","i","k","t","o","r");
}
```

一个可能的解决方案可能是：

```java
List<String> lettersUsed = names.stream()
  .map(String::toLowerCase)
  .flatMap(name -> Stream.of(name.split("")))
  .distinct()
  .collect(Collectors.toList());
```

这次我们使用了`Stream.of()`，这是一个创建流的便捷方法。另一个非常好的特性是`distinct()`方法，它使用`equals()`方法比较它们并返回唯一元素的集合。

# 减少

在前面的例子中，函数返回作为输入传递的所有名称中使用的字母列表。但是，如果我们只对不同字母的数量感兴趣，有一种更简单的方法。`reduce`基本上将函数应用于所有元素并将它们组合成一个单一的结果。让我们看一个例子：

```java
@Test
public void countingLettersUsedInNames() {
  List<String> names = Arrays.asList("Alex", "Paul", "Viktor");
  long count = 0;

  assertThat(count).isEqualTo(12);
}
```

这个解决方案与我们用于上一个练习的解决方案非常相似：

```java
long count = names.stream()
  .map(String::toLowerCase)
  .flatMap(name -> Stream.of(name.split("")))
  .distinct()
  .mapToLong(l -> 1L)
  .reduce(0L, (v1, v2) -> v1 + v2);
```

尽管前面的代码片段解决了问题，但有一种更好的方法来做到这一点：

```java
long count = names.stream()
  .map(String::toLowerCase)
  .flatMap(name -> Stream.of(name.split("")))
  .distinct()
  .count();
```

`count()`函数是 Streams 包含的另一个内置工具。它是一个特殊的快捷方式，用于计算流中包含的元素数量的`reduction`函数。

# 总结

函数式编程是一个古老的概念，因为它更容易在尝试通过并行执行任务来提高性能时使用而变得流行。在本章中，一些来自函数式世界的概念以及 AssertJ 提供的一些测试工具被介绍了。

测试没有副作用的函数非常容易，因为测试范围被缩小了。不需要测试函数可能对不同对象造成的更改，唯一需要验证的是调用的结果。没有副作用意味着只要参数相同，函数的结果就是相同的。因此，执行可以重复多次，并且在每次执行时都会得到相同的结果。此外，测试更容易阅读和理解。

总之，如果您需要在项目中使用这种范式，Java 包含了一个很好的函数式编程 API。但是有一些语言，其中一些是纯函数式的，提供了更强大的功能，更好的语法和更少的样板代码。如果您的项目或方法可以是纯函数式的，您应该评估是否使用其中一种其他语言是合理的。

本章中介绍的所有示例都可以在[`bitbucket.org/alexgarcia/tdd-java-funcprog.git`](https://bitbucket.org/alexgarcia/tdd-java-funcprog.git)找到。

现在是时候看一看遗留代码以及如何对其进行调整，使其更符合 TDD 的要求。
