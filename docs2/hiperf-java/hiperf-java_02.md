# 2

# 数据结构

数据结构是我们 Java 应用程序性能的组成部分，它们可能有助于提高性能，也可能降低性能。它们是程序中的基础部分，用于整个程序，可以帮助我们有效地组织和操纵数据。数据结构对于优化我们的 Java 应用程序的性能至关重要，因为它们可以确保我们的数据访问、内存管理和缓存效率。正确使用数据结构可以提高算法效率，提高解决方案的可扩展性，并确保线程的安全性。

数据结构的重要性可以通过减少操作的时间复杂度来证明。通过适当的数据结构实现，我们可以提高应用程序性能的可预测性和一致性。除了提高 Java 应用程序的性能外，适当选择的数据结构还可以提高代码的可读性，使它们更容易维护。

在本章中，我们将涵盖以下主要主题：

+   列表

+   数组

+   树

+   栈和队列

+   高级数据结构

到本章结束时，您应该了解特定数据结构，如**列表**、**数组**、**树**、**栈**和**队列**，如何影响 Java 应用程序的性能。您将有机会通过 Java 代码的实际操作来获得经验，展示如何通过适当的数据结构选择和实现来提高性能。

# 技术要求

要遵循本章中的示例和说明，您需要能够加载、编辑和运行 Java 代码。如果您还没有设置您的开发环境，请参阅*第一章*。

本章的代码可以在以下位置找到：[`github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter02`](https://github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter02)。

# 使用列表提高性能

列表是 Java 编程语言中的基本数据结构。它们使我们能够轻松创建、存储和操作有序元素集合。此数据结构使用`java.util.list`接口，并扩展了`java.util.Collection`接口。

在本节中，我们将仔细研究列表，探讨何时以及为什么使用它们，以及如何从它们中获得最佳性能的技术。

## 为什么使用列表？

解释列表数据结构可以用于的最常见方式可能是作为一个勾选/to-do 列表或购物清单。我们在程序中创建列表，因为我们想利用其一或多个优势：

+   **有序元素**：列表用于保持我们元素的顺序，即使在添加新元素时也是如此。这使我们能够以特定的顺序操纵我们的数据。考虑一个系统日志，它通过日期和时间戳添加新条目。我们希望这些条目保持特定的顺序。

+   **自动调整大小**：列表可以在我们的程序添加和删除元素时动态调整大小。这在**ArrayList**中尤其如此，这将在本章后面进行介绍。

+   **位置数据访问**：列表允许我们通过使用元素的索引（也称为**位置数据**）来高效地获得随机访问。

+   **重复元素**：列表允许我们拥有重复元素。因此，如果这对您的用例很重要，那么您可能需要考虑使用列表作为您的数据结构选择。

+   `forEach`方法。我们将在本章下一节中查看此示例。

+   `LinkedList`、`ArrayList`或`Vector`列表类型。这些列表类型具有独特的特性。以下表格显示了这些列表类型的不同特性。请特别注意性能这一行：

|  | **LinkedList** | **ArrayList** | **Vector** |
| --- | --- | --- | --- |
| **数据结构** | 双向链表 | 动态数组 | 动态数组 |
| **用例** | 经常操纵数据 | 快速读取 | 需要线程安全 |
| **性能** | + 添加和删除- 通过索引访问 | - 添加和删除+ 通过索引访问 | - 添加和删除- 通过索引访问 |
| **线程安全** | 否，默认情况下不是 | 否，默认情况下不是 | 是，默认情况下是 |

表 2.1 – 列表

在选择`LinkedList`、`ArrayList`和`Vector`列表类型时，我们应该根据我们的用例考虑需求。例如，如果我们的用例包括频繁的添加和删除，那么`LinkedList`可能是最佳选择。或者，如果我们添加和删除不频繁，但读取量大，那么`ArrayList`可能是我们的最佳选择。最后，如果我们最关心线程安全，那么`Vector`可能是我们的最佳选择。

关于线程安全的重要注意事项

虽然`LinkedList`和`ArrayList`默认不是线程安全的，但可以通过显式同步访问来使其线程安全。这应该可以防止并发访问相关的数据损坏，但可能会降低 Java 应用程序的性能。

+   `java.util.List`接口，列表实现了它。这些内置方法包括搜索、添加和删除元素的功能。`java.util.List`的完整方法列表可在官方 Java 文档中找到：[`docs.oracle.com/javase/8/docs/api/java/util/List.html`](https://docs.oracle.com/javase/8/docs/api/java/util/List.html)。

现在我们已经回顾了为什么我们应该使用列表，让我们看看常见的实现示例。

## 常见的列表实现

在本节中，我们将查看`ArrayList`、`LinkedList`和`Vector`的实现示例。

### ArrayList 示例

我们的第一个列表实现示例是一个数字的`ArrayList`列表类型。我们将假设这是一个人力资源（HR）系统的一部分，该系统存储开始日期、结束日期和服务长度。如下所示，我们必须导入`java.util.ArrayList`和`java.util.List`：

```java
import java.util.ArrayList;
import java.util.List;
```

接下来，我们有我们的类声明和主方法。在这里，我们必须创建一个名为`hr_numbers`的`ArrayList`列表类型：

```java
public class Example1 {
  public static void main(String[] args) {
    .add method to add elements to our ArrayList:

```

hr_numbers.add(1983);

hr_numbers.get 方法，并使用它们来计算一个值，将其作为 ArrayList 的第三个元素添加：

```java
    int startYear = hr_numbers.get(0);
    int endYear = hr_numbers.get(1);
    hr_numbers.add(endYear-startYear);
```

            代码的最后部分是一个`for`循环。这个循环遍历列表，并将输出提供给终端窗口：

```java
    for (int number : hr_numbers) {
      System.out.println(number);
    }
```

            这是程序的输出：

```java
1983
2008
25
```

            如您所见，我们的输出符合预期；我们只是使用`for-each`循环将三个元素简单地打印到终端窗口。

            for-each 循环

            Java 5 引入了一个增强的`for`循环，称为`for-each`循环。我们可以使用这个循环来遍历元素，而无需使用显式的迭代器或索引。这使得我们的代码更快地编写，更易于阅读。

            链接示例

            我们的`LinkedList`实现示例包括简单的`get`、`remove`、`contains`和`size`方法。如您所见，我们导入了`java.util.LinkedList`和`java.util.List`：

```java
Import java.util.LinkedList;
import java.util.List;
```

            接下来，我们有我们的类声明和主方法。在这里，我们创建一个名为`petNames`的`LinkedList`列表类型：

```java
public class Example2 {
  public static void main(String[] args) {
    .add method to add elements to our LinkedList:

```

petNames.add("Brandy");

petNames.add("Muzz");

petNames.add("Java");

petNames.get 方法：

```java
    String firstPet = petNames.get(0);
    String secondPet = petNames.get(1);
```

            代码的下一部分是一个`for-each`循环，它遍历列表并将输出提供给终端窗口：

```java
    for (String pet : petNames) {
      System.out.println(pet);
    }
```

            这是循环的输出：

```java
Brandy
Muzz
Java
Bougie
```

            我们可以通过使用`remove`方法从我们的`LinkedList`中删除一个元素，如下所示。如您所见，在调用`remove`方法后，`Brandy`不再是`LinkedList`中的一个元素：

```java
petNames.remove("Brandy");
```

            我们代码的下一部分调用`contains`方法来检查特定值是否在`LinkedList`中。由于我们之前已经从这个`LinkedList`中删除了这个宠物，布尔结果是`false`：

```java
boolean containsBrandy = petNames.contains("Brandy");
System.out.println(containsBrandy);
```

            `println`语句的输出符合预期：

```java
false
```

            我们代码的最后一部分展示了`size`方法的使用。在这里，我们调用该方法，它返回一个整数。我们使用这个值在我们的最终输出中：

```java
int size = petNames.size();
System.out.println("You have " + size + " pets.");
```

            最终输出反映了我们的`LinkedList`预期的长度：

```java
You have 3 pets.
```

            现在我们已经了解了如何实现`ArrayList`和`LinkedList`，让我们看看我们的最后一个示例，**Vector**。

            向量示例

            我们最后一个列表实现示例是一个`Vector`列表类型。如您将看到的，`Vector`与`ArrayList`类似。我们将它们实现为动态数组，以便我们可以从向量的元素中获得高效的随机访问。向量默认是线程安全的，这是由于我们之前讨论的默认同步。让我们看看一些示例代码。

            我们的示例程序将存储一组幸运数字。它首先导入`java.util.Vector`和`java.util.Enumeration`包：

```java
import java.util.Vector;
import java.util.Enumeration;
```

            接下来，我们有我们的类声明和主方法。在这里，我们创建一个名为`luckyNumbers`的`Vector`列表类型，它将存储整数：

```java
public class Example3 {
  public static void main(String[] args) {
    .add method to add elements to our Vector.

```

luckyNumbers.add(8);

luckyNumbers.add(19);

luckyNumbers.get 方法。请注意，索引从零（0）开始：

```java
    int firstNumber = luckyNumbers.get(0);
    int secondNumber = luckyNumbers.get(2);
```

            代码的下一部分使用了一个传统的遍历`Vector`列表类型的方法。枚举方法可以用作替代增强型或`for-each`循环。实际上，向量被视为一个正在逐渐被淘汰的列表类型。以下代码遍历列表并向终端窗口提供输出：

```java
    Enumeration<Integer> enumeration = luckyNumbers.elements();
    while (enumeration.hasMoreElements()) {
      int number = enumeration.nextElement();
      System.out.println(number);
    }
```

            下面是程序的输出：

```java
8
19
24
```

            我们可以通过使用`removeElement`方法从我们的`Vector`中移除一个元素，如下所示。在调用`removElement`方法后，幸运数字 19 被从`Vector`中移除作为一个元素：

```java
luckyNumbers.removeElement(19);
```

            我们代码的下一部分调用`contains`方法来检查特定值是否在`Vector`中。由于我们之前已经从`Vector`中移除了这个幸运数字，布尔结果为`false`：

```java
boolean containsNineteen= luckyNumbers.contains(19);
System.out.println(containsNineteen);
```

            `println`语句的输出符合预期：

```java
false
```

            以下代码的最后一部分展示了`size`方法的使用。在这里，我们调用该方法，它返回一个整数。我们使用这个值在我们的最终输出中：

```java
int mySize = luckyNumbers.size();
System.out.println("You have " + mySize + " lucky numbers.");
```

            最终输出反映了我们`LinkedList`列表类型的预期大小：

```java
You have 2 lucky numbers.
```

            本节提供了使用`ArrayList`、`LinkedList`和`Vector`的列表的示例。还有其他实现可以考虑，包括`CopyOnWriteArrayList`、`CopyOnWriteArraySet`和`LinkedHashSet`。栈是另一种实现，将在本章后面介绍。接下来，我们将探讨如何通过列表实现高性能。

            高性能的列表

            让我们看看如何提高我们使用列表时 Java 应用程序的性能。以下代码示例实现了整数类型的`LinkedList`列表类型。在这里，我们将创建一个`LinkedList`列表类型，向其中添加四个元素，然后遍历列表，将每个元素打印到屏幕上：

```java
import java.util.LinkedList;
import java.util.List;
public class Example4 {
  public static void main(String[] args) {
    List<Integer> numbers = new LinkedList<>();
    numbers.add(3);
    numbers.add(1);
    numbers.add(8);
    numbers.add(9);
    System.out.println("Initial LinkedList elements:");
    for (int number : numbers) {
      System.out.println(number);
    }
  }
}
```

            本节的第一部分输出如下：

```java
Initial LinkedList elements:
3
1
8
9
```

            下一个代码部分使用`remove`方法删除一个元素——具体来说，是第一个出现的 8：

```java
numbers.remove(Integer.valueOf(8));
```

            以下代码段使用`contains`方法执行两个检查。首先，它检查 3，然后是 8。结果被打印在屏幕上：

```java
boolean containsThree = numbers.contains(3);
System.out.println("\nThe question of 3: " + containsThree);
boolean containsEight = numbers.contains(8);
System.out.println("The question of 8: " + containsEight);
```

            本节代码的输出如下所示：

```java
The question of 3: true
The question of 8: false
```

            以下代码段遍历`LinkedList`并打印其值：

```java
System.out.println("\nModified LinkedList elements:");
for (int number : numbers) {
  System.out.println(number);
}
```

            以下代码段是本节最后部分的输出：

```java
Modified LinkedList elements:
3
1
9
```

            现在，是时候看看我们如何修改我们的代码来提高整体性能了。我们可以使用几种技术：

                +   `List<Integer>`对象。`Integer`类本质上是对原始`int`数据类型的包装。`Integer`对象只包含一个`int`类型的字段。以下是我们如何修改我们的代码的方法：

    ```java
    LinkedList<Integer> numbers = new LinkedList<>();
    ```

                    +   对于`LinkedList`，我们应该使用`java.util.Iterator`包。这是一个避免诸如**ConcurrentModificationsException**等错误的有效方法，这种异常会在我们尝试同时对一个集合进行两次修改时抛出。以下是我们如何编写这样的迭代器的方法：

    ```java
    Iterator<Integer> iterator = numbers.iterator();
    while (iterator.hasNext()) {
      int number = iterator.next();
      if (number == 8) {
        iterator.remove();
      }
    }
    ```

    完全修订的示例可在[`github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter02/Example5.java`](https://github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter02/Example5.java)找到。

            现在，你应该对在 Java 中使用列表以帮助提高应用程序性能有了更多的知识和信心。接下来，我们将探讨数组以及如何最优地使用它们。

            使用数组提高性能

            在之前，我们讨论了`ArrayList`；在本节中，我们将专注于数组。这两种数据类型之间存在关键差异：数组具有固定的大小，而 ArrayList 没有。在本节中，我们将回顾数组的特性和如何在实现数组时提高 Java 应用程序的性能。

            数组特性

            数组有四个主要特性。让我们看看：

                +   **大小**：数组的大小在创建时确定。在应用程序运行时无法更改。这种固定大小的特性也被称为**静态**。以下是创建数组的语法：

    ```java
    int[] myNumbers = new int[10];
    ```

    如你所见，我们在数组声明中明确指定了大小为`10`。

                    +   **同质**：同质意味着数组中的所有数据必须属于同一类型。就像之前的例子一样，我们创建了一个整数数组。我们可以使用字符串，但不能在单个数组中混合数据类型。我们也可以有对象数组和数组数组。

                +   **索引**：这一点不言而喻，所以作为一个提醒，我们的索引从 0 开始，而不是 1。因此，我们之前创建的数组有 10 个元素，索引从 0 到 9。

                +   **连续内存**：数组为我们提供的一项巨大效率是随机访问。这种效率源于数组中的所有元素都存储在连续内存中的事实。换句话说，元素存储在相邻的内存位置。

            现在我们对数组特性有了牢固的理解，让我们探索一些实现这种重要数据结构的代码。

            实现数组

            本节介绍了一个基本的 Java 应用程序，该程序使用`String`数据类型实现行星数组。随着我们遍历代码，我们将创建数组，访问和打印数组元素，使用`length`方法，使用索引访问数组元素，并修改数组。

            这段代码的第一个部分创建了一个**字符串**数组：

```java
public class Example6 {
  public static void main(String[] args) {
    String[] planets = {
      "Mercury",
      "Venus",
      "Earth",
      "Mars",
      "Jupiter",
      "Saturn",
      "Uranus",
      "Neptune"
    };
  }
}
```

            如你所见，我们创建了一个包含八个字符串的数组。接下来的代码部分展示了如何访问和打印数组的所有元素：

```java
System.out.println("Planets in our solar system:");
for (int i = 0; i < planets.length; i++) {
  System.out.println(planets[i]);
}
```

            以下是前面代码片段的输出：

```java
Planets in our solar system:
Mercury
Venus
Earth
Mars
Jupiter
Saturn
Uranus
Neptune
```

            我们可以使用`length`方法来确定我们数组的大小。虽然看起来我们不需要确定大小，因为数组创建时大小就已经确定了。通常，我们不知道数组的初始大小，因为它们基于外部数据源。以下是确定数组大小的步骤：

```java
int numberOfPlanets = planets.length;
System.out.println("Number of planets: " + numberOfPlanets);
```

            前面代码的输出如下所示：

```java
Number of planets: 8
```

            接下来，我们将探讨如何通过在数组中引用其索引位置来访问数组元素：

```java
String thirdPlanet = planets[2];
System.out.println("The third planet is: " + thirdPlanet);
```

            上一两行代码的输出如下：

```java
The third planet is: Earth
```

            如您所见，我们通过使用索引引用`2`打印了数组中的第三个元素。请记住，我们的索引从 0 开始。

            我们最后的代码段展示了我们如何修改数组中的元素：

```java
planets[1] = "Shrouded Venus";
System.out.println("After renaming Venus:");
for (String planet : planets) {
  System.out.println(planet);
}
```

            上述代码的输出如下。如您所见，索引位置 1 的元素的新名称已反映在输出中：

```java
After renaming Venus:
Mercury
Shrouded Venus
Earth
Mars
Jupiter
Saturn
Uranus
Neptune
```

            完全修订的示例可在[`github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter02/Example6.java`](https://github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter02/Example6.java)找到。

            现在您已经学会了如何在 Java 中实现数组，让我们看看一些提高应用程序性能的方法，特别是关于数组的使用。

            使用数组实现高性能

            正如我们所看到的，我们的列表实现方法对 Java 应用程序的性能有直接影响。本节记录了优化与数组相关的 Java 代码的几种策略和最佳实践。这些策略可以分为算法优化、数据结构、内存管理、并行处理、向量化、缓存以及基准测试和性能分析。让我们逐一查看它们。

            算法优化

            在选择算法时，我们应该确保它们是最适合我们使用的数据类型的。例如，根据我们数组的尺寸和特性选择最有效的排序算法（例如归并排序、快速排序等）是很重要的。此外，在排序数组中进行搜索时可以实现二分查找。

            本书*第五章*提供了更多信息。

            数据结构

            本章的核心内容是针对您的用例和需求选择合适的数据结构。当涉及到数组时，我们应该在我们需要频繁读取访问时选择它们；在这里，我们的数据集可以是固定大小的。此外，知道何时选择`ArrayList`而不是`LinkedList`同样重要。

            内存管理

            在可能的情况下，我们应该避免创建临时对象来支持数组操作。相反，我们应该重用数组或使用`Arrays.copyOf`或`System.arraycopy`等方法以提高效率。

            本书*第八章*提供了更多信息。

            并行处理

            当我们必须对大型数组进行排序和处理时，利用 Java 的并行处理能力，如使用`parallelSort`，可能是有益的。您应该考虑使用多线程进行并发数组处理。这对于大型数组尤为重要。

            本书*第九章*提供了更多信息。

            向量化

            在 Java 数组的上下文中，向量化是一种涉及同时对一个数组的多个元素执行操作的技术。这通常包括优化现代中央处理器。目标是增加数组处理操作。这通常被称为`java.util.Vector`类，它是随着 Java 16 引入的。

            向量化可以提供显著的性能优势，尤其是在处理大型数组时。当操作可以并行化时，这一点更是如此。可以向量化的内容有限，例如依赖关系和复杂操作。

            缓存

            一种经过验证的性能方法是优化数组访问模式以支持缓存局部性。这可以通过访问连续的内存位置来实现。另一种方法是尽可能减少指针别名。虽然指针别名可能支持编译器优化，但为了获得最佳性能，应使用局部变量或数组索引。

            本书*第八章*提供了更多信息。

            基准测试和分析

            在可能的情况下，我们应该进行基准测试，以便比较我们的数组操作方法。有了这种分析，我们可以选择最有效且经过验证的方法。作为我们分析的一部分，我们可以使用分析工具来帮助我们识别特定于我们的数组操作的性能瓶颈。

            本书*第十三章*提供了更多信息。

            现在你已经学会了如何提高处理数组时的 Java 应用程序性能，让我们看看如何使用树来实现这一点。

            使用树提高性能

            **树**是一种层次数据结构，由父节点和子节点**节点**组成，类似于物理树。最顶部的节点被称为**根**，在树数据结构中只能有一个。节点是这个数据结构的基础组件。每个节点都包含数据和指向子节点的引用。还有其他你应该熟悉的关于树的概念。**叶节点**没有子节点。任何节点及其后代都可以被认为是**子树**。

            树结构的示例

            如以下示例所示，我们可以将树实现为对象或类：

```java
class TreeNode {
  int data;
  TreeNode left;
  TreeNode right;
  public TreeNode(int data) {
    this.data = data;
    this.left = null;
    this.right = null;
  }
}
```

            上述代码片段定义了一个`TreeNode`类，它可以与另一个类一起用于树操作管理。我们需要在`main()`方法中创建我们的树。以下是如何做到这一点的示例：

```java
Example7BinarySearchTree bst = new Example7BinarySearchTree();
```

            要将数据插入我们的树，我们可以使用以下代码：

```java
bst.insert(50);
bst.insert(30);
bst.insert(70);
bst.insert(20);
bst.insert(40);
bst.insert(60);
bst.insert(80);
```

            要使用或搜索我们的树，我们可以使用类似于以下代码的代码在树中执行二分查找：

```java
int searchElement = 70;
if (bst.search(searchElement)) {
  System.out.println("\n" + searchElement + " was found in the tree.");
} else {
  System.out.println("\n" + searchElement + " was not found in the 
  tree.");
}
```

            一个完整的示例代码可以在[`github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter02/Example7.java`](https://github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter02/Example7.java)找到。

            高性能考虑因素

            树数据结构可能非常复杂，因此我们应该考虑我们的 Java 应用程序的整体性能，特别是关于操作我们的树的算法。额外的考虑可以归类为类型、安全性、迭代、内存、操作和缓存。让我们简要地看看这些类别。

            类型

            重要的是要仔细构建我们的树，使它们尽可能平衡。我们应该注意**树的高度**，即从根节点到叶节点的最长路径。我们应该审查数据需求，以便我们可以选择最优的树。例如，我们可以使用**二叉搜索树**，它可以为我们提供高效的搜索。

            安全性

            与其他数据结构一样，你应该始终实现线程安全协议，或者至少使用并发树结构。

            迭代

            迭代操作常常会呈现瓶颈，需要调用优化策略。此外，应尽量减少递归算法的使用，以提高整体性能。

            内存

            通常，具体到树，我们应该尽量减少对象创建和开销。我们应该努力高效地管理内存。

            本书*第八章*提供了更多信息。

            操作

            我们可以采取一些措施来优化操作。例如，我们可以使用批量操作来减少处理开销。优化我们的算法可以帮助我们避免过度处理和数据更新。

            本书*第五章*提供了更多信息。

            缓存

            我们可以优化数据布局的方式，以最大化缓存局部性。我们的目标是减少内存访问时间。此外，我们可以访问内存中的几乎所有节点，以进一步提高性能。

            本书*第八章*提供了更多信息。

            使用栈和队列提高性能

            **栈**和**队列**几乎总是放在一起，因为它们都可以用来管理和操作数据元素集合。它们都是线性栈；这就是它们的相似之处。虽然它们被分组，但它们在操作方式上有关键的区别。在本节中，我们将探讨如何实现栈和队列，以及如何为高性能 Java 应用程序优化它们。

            实现栈

            栈是线性数据结构，使用`push`元素到顶部，`peek`查看一个元素，以及`pop`移除顶部元素。

            以下示例演示了如何在 Java 中创建一个栈。你会注意到我们首先导入`java.util.Stack`包：

```java
import java.util.Stack;
public class Example8 {
  public static void main(String[] args) {
    Stack<Double> transactionStack = new Stack<>();
    transactionStack.push(100.0);
    transactionStack.push(-50.0);
    transactionStack.push(200.0);
    while (!transactionStack.isEmpty()) {
      double transactionAmount = transactionStack.pop();
      System.out.println("Transaction: " + transactionAmount);
    }
  }
}
```

            以下代码是 Java 栈的简单实现。该程序的目的是对银行交易进行处理。现在你已经看到了这个简单的实现，让我们看看我们如何可以改进我们的代码以获得更好的性能。

            提高栈的性能

            以我们之前的示例为起点，我们将对其进行优化以提高运行时的性能。我们将首先使用自定义的栈实现。这种优化特别适用于涉及大量事务的使用案例。正如您所看到的，我们必须导入 `java.util.EmptyStackException` 包。

            接下来，我们必须声明一个类并使用具有 **double** 数据类型的数组。我们选择这种方法是为了避免由于 **auto-boxing** 而产生的处理开销：

```java
public class Example9 {
  private double[] stack;
  private int top;
  public Example9(int capacity) {
    stack = new double[capacity];
    top = -1;
  }
```

            以下代码段定义了 `push`、`pop` 和 `isEmpty` 方法：

```java
public void push(double transactionAmount) {
  if (top == stack.length - 1) {
    throw new RuntimeException("Stack is full.");
  }
  stack[++top] = transactionAmount;
}
public double pop() {
  if (isEmpty()) {
    throw new EmptyStackException();
  }
  return stack[top--];
}
public boolean isEmpty() {
  return top == -1;
}
```

            我们最后的代码段是 `main()` 方法，它处理栈：

```java
public static void main(String[] args) {
  Example9 transactionStack = new Example9(10);
  transactionStack.push(100.0);
  transactionStack.push(-50.0);
  transactionStack.push(200.0);
  while (!transactionStack.isEmpty()) {
    double transactionAmount = transactionStack.pop();
    System.out.println("Transaction: " + transactionAmount);
  }
}
```

            简单和优化后的两组程序的输出相同，并在此展示：

```java
Transaction: 200.0
Transaction: -50.0
Transaction: 100.0
```

            我们栈实现的两个版本的完整工作示例可在 [`github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter02/Example8.java`](https://github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter02/Example8.java) 和 [`github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter02/Example9.java`](https://github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter02/Example9.java) 找到。

            实现队列

            队列是另一种线性数据结构，并使用 `enqueue`、`peek` 和 `dequeue` 来管理我们的队列。

            以下示例演示了如何在 Java 中创建一个队列。正如您所看到的，我们首先导入 `java.util.LinkedList` 和 `java.util.Queue` 包：

```java
import java.util.LinkedList;
import java.util.Queue;
public class Example10 {
  public static void main(String[] args) {
    Queue<Double> transactionQueue = new LinkedList<>();
    transactionQueue.offer(100.0);
    transactionQueue.offer(-50.0);
    transactionQueue.offer(200.0);
    while (!transactionQueue.isEmpty()) {
      double transactionAmount = transactionQueue.poll();
      System.out.println("Transaction: " + transactionAmount);
    }
  }
}
```

            上述代码是 Java 队列的简单实现。程序的目的，就像栈示例一样，是处理银行交易。现在您已经看到了这个简单实现，让我们看看我们如何可以优化我们的代码以获得更好的性能。

            提高队列的性能

            我们优化的队列实现已经针对运行时性能进行了优化。这对于高事务性应用尤为重要。我们首先导入 `java.util.NoSuchElementException` 包，然后声明类和一组私有类变量，在创建自定义队列实现构造函数之前：

```java
import java.util.NoSuchElementException;
public class Example11 {
  private double[] queue;
  private int front;
  private int rear;
  private int size;
  private int capacity;
  public Example11(int capacity) {
    this.capacity = capacity;
    queue = new double[capacity];
    front = 0;
    rear = -1;
    size = 0;
  }
```

            以下代码段包括 `enqueue`、`dequeue` 和 `isEmpty` 方法：

```java
public void enqueue(double transactionAmount) {
  if (size == capacity) {
    throw new RuntimeException("Queue is full.");
  }
  rear = (rear + 1) % capacity;
  queue[rear] = transactionAmount;
  size++;
}
public double dequeue() {
  if (isEmpty()) {
    throw new NoSuchElementException("Queue is empty.");
  }
  double transactionAmount = queue[front];
  front = (front + 1) % capacity;
  size--;
  return transactionAmount;
}
public boolean isEmpty() {
  return size == 0;
}
```

            我们最后的代码段是 `main()` 方法，它处理队列：

```java
public static void main(String[] args) {
  Example11 transactionQueue = new Example11(10);
  transactionQueue.enqueue(100.0);
  transactionQueue.enqueue(-50.0);
  transactionQueue.enqueue(200.0);
  while (!transactionQueue.isEmpty()) {
    double transactionAmount = transactionQueue.dequeue();
    System.out.println("Transaction: " + transactionAmount);
  }
}
```

            简单和优化后的两组程序的输出相同，并在此提供：

```java
Transaction: 100.0
Transaction: -50.0
Transaction: 200.0
```

            我们队列实现的两个版本的完整工作示例可在 [`github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter02/Example10.java`](https://github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter02/Example10.java) 和 [`github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter02/Example11.java`](https://github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter02/Example11.java) 找到。

            在 Java 中，栈和队列都可以用于多种用例。确保我们从性能角度出发最合理地使用它们非常重要。此外，我们必须考虑之前详细说明的优化方法。

            使用高级数据结构提高性能

            我们可以使用的比本章迄今为止介绍的数据结构还要多。有时，我们不应该改进我们对数据结构（如列表、数组、树、栈或队列）的使用，而应该实现针对我们特定用例量身定制的高级数据结构，然后优化它们的使用。

            让我们看看一些额外的、更高级的数据结构，它们需要考虑性能因素。

            哈希表

            当我们需要快速的**键值**查找时，我们可以使用**哈希表**。有多种哈希函数可供选择，你应该注意哈希冲突——它们需要被高效地管理。一些额外的考虑因素包括负载因子、调整大小、内存使用和性能。

            下面是一个创建哈希表并为其创建根节点的示例：

```java
import java.util.HashMap;
HashMap<String, Integer> hashMap = new HashMap<>();
hashMap.put("Alice", 25);
```

            图

            图，例如**矩阵**或**邻接表**，可以用来网络化一个应用程序并建模复杂的数据关系。在实现图时，我们应该实现能够高效遍历图的算法。值得探索的两个算法是**广度优先搜索**和**深度优先搜索**。比较这些算法的效率可以帮助你选择最优化的解决方案。

            下面是如何实现图的方法：

```java
import java.util.ArrayList;
import java.util.List;
List<List<Integer>> graph = new ArrayList<>();
int numNodes = 5;
for (int i = 0; i < numNodes; i++) {
  graph.add(new ArrayList<>());
}
graph.get(0).add(1);
```

            字典树

            **字典树**是一种在编程自动完成、前缀匹配等方法时非常有用的数据结构。字典树允许我们高效地存储和搜索字符串序列。

            下面是一个创建字典树并为它创建根节点的示例：

```java
class TrieNode {
  TrieNode[] children = new TrieNode[26];
    boolean isEndOfWord;
}
TrieNode root = new TrieNode();
```

            我们将在本书的*第七章*中介绍字符串操作。

            堆

            **优先队列**通常实现为**堆**，因为它们可以基于元素的顺序或优先级高效地管理元素。这对于包括调度、优先级排序和元素顺序在内的用例来说是一个理想的数据结构。

            下面是一个将优先队列实现为堆的示例。此示例演示了`minHeap`和`maxHeap`，并为每个添加了一个元素：

```java
import java.util.PriorityQueue;
PriorityQueue<Integer> axheap = new PriorityQueue<>();
axheap.offer(3);
PriorityQueue<Integer> axheap = new PriorityQueue<>((a, b) -> b – a);
axheap.offer(3);
```

            四叉树

            **四叉树**是一种用于高效组织空间数据的二维分区。它们在空间索引、地理信息系统、碰撞检测等方面非常有用。

            创建四叉树比实现其他数据结构要复杂一些。下面是一个简单的方法：

```java
class QuadTreeNode {
  int val;
  boolean isLeaf;
  QuadTreeNode topLeft;
  QuadTreeNode topRight;
  QuadTreeNode bottomLeft;
  QuadTreeNode bottomRight;
  public QuadTreeNode() {}
  public QuadTreeNode(int val, boolean isLeaf) {
      this.val = val;
      this.isLeaf = isLeaf;
  }
}
QuadTreeNode root = new QuadTreeNode(0, false);
```

            八叉树

            **八叉树**是一种用于高效组织空间数据的三维分区。像四叉树一样，八叉树在空间索引、地理信息系统、碰撞检测等方面非常有用。

            在 Java 中实现八叉树可能比较困难，这与大多数三维或更高维度的数据结构一样。

            位图集

            当我们需要为整数或布尔数据集提供空间高效的存储，并确保它们的操作是高效的，**位集**数据结构值得考虑。示例用例包括遍历图并标记已访问节点的算法。

            下面是一个实现位集的示例：

```java
import java.util.BitSet;
BitSet bitSet = new BitSet(10); // Creates a BitSet with 10 bits
bitSet.set(2);
boolean isSet = bitSet.get(2);
```

            线索

            线索非常适合实现高效的字符串处理，尤其是大型字符串。它们用于执行连接和子字符串操作。一些示例用例包括文本编辑应用程序和字符串操作功能。

            从零开始实现线索是一个非常复杂的数据结构。开发者通常会转向 JDK 之外的包和工具来完成这项工作。

            我们将在本书的*第七章*中介绍字符串操作。

            这些只是 Java 中可用的部分高级数据结构。当我们采用这些结构时，我们必须了解它们的工作原理、它们的优点和缺点。

            概述

            本章深入探讨了几个数据结构，并展示了它们对 Java 应用程序整体性能的重要性。我们拥抱数据结构的基本性质，以优化我们的 Java 应用程序的性能，因为它们可以确保我们的数据访问、内存管理和缓存都是高效的。正确使用数据结构可以提高算法效率、我们解决方案的可扩展性和线程的安全性。

            通过代码示例，我们研究了列表、数组、树、栈和队列的非优化和优化代码示例。我们还探索了几个高级数据结构，并检查了它们的附加复杂性。当我们正确实现数据结构时，我们可以提高我们应用程序性能的可预测性和一致性，并使我们的代码更易于阅读和维护。

            在下一章中，我们将专注于循环以及如何优化它们以提高我们的 Java 应用程序的性能。这是在介绍数据结构之后的一个自然进展，因为我们将使用它们在下一章的示例中。

```java

```

```java

```

```java

```
