# 第四章：4. 集合、列表和 Java 的内置 API

概述

本章向您介绍强大的 Java 集合框架，该框架用于存储、排序和过滤数据。它将首先带您了解内置的集合**应用程序编程接口**（**API**），即 Java 集合框架，这将简化您与复杂数据结构的交互，并允许您以最小的努力使用和创建 API。通过这个框架，您将检查列表和数组之间的关系，并学习如何从数组中填充列表。最后，在本章的最后一个活动中，您将创建并完成一个程序，在这个程序中，您将被要求对存储在集合、列表和映射中的数据进行标准操作，为未来的章节做准备。

# 简介

Java 自带内置的集合 API，允许你以极少的努力操作数据结构。集合是一个包含多个元素的对象。集合用于存储、共享、处理和通信聚合数据。我们称这个系统为**Java 集合框架**。

作为这个框架的一部分，有不同组件用于优化我们与实际数据的交互：

+   **接口**：表示集合的抽象数据类型

+   **实现**：集合接口的具体实现

+   **算法**：用于在集合中处理数据的多态方法，例如排序和搜索操作

    注意

    其他编程语言也有自己的集合框架。例如，C++有**标准模板库**（**STL**）。Java 在集合框架方面以简单著称。

使用集合框架有许多好处，包括减少处理数据结构程序的复杂性、提高程序性能、简化 API 创建和使用，以及增加功能软件的重用。

即使在可以由多个进程同时访问数据的情况下，集合框架也是相关的，例如在多线程编程场景中。然而，本章的目的并不是处理并发编程。

集合 API 包含五个主要接口：

+   `集合`：不包含重复元素的集合

+   `列表`：一个有序集合或序列，允许有重复元素

+   `队列`：按到达顺序排序数据的集合，通常作为**先进先出**（**FIFO**）过程处理

+   `双端队列`：本质上是一个允许在两端插入数据的队列，这意味着它可以作为**先进先出**（**FIFO**）和**后进先出**（**LIFO**）处理

+   `映射`：将键（必须是唯一的）与值相关联

在本章中，我们将定义主要接口（列表、集合和映射），并探讨它们各自的使用示例。该框架比之前列出的接口更多，但其他接口要么是那些列出的变体，要么超出了本章的范围。此外，我们将比以前更深入地探讨数组的工作原理。

简单集合的定义——在这种情况下，特定类型的集合的定义如下：

```java
Set mySet = new HashSet();
```

注意

可用的不同集合类（集合、列表、队列、双端队列和映射）以接口命名。不同的类具有不同的属性，我们将在本章后面看到。

# 数组

数组是集合框架的一部分。有一些静态方法可以用来操作数组。您可以执行的操作包括创建、排序、搜索、比较、流式传输和转换数组。您在*第二章*，*学习基础知识*中介绍了数组，您看到了它们如何用于存储相同类型的数据。数组的声明相当简单。让我们看看字符串数组会是什么样子：

```java
String[] text = new String[] { "spam", "more", "buy" };
```

在数组上运行操作与调用`java.util.Arrays`包中包含的一些方法一样简单。例如，对前面的数组进行排序需要调用以下代码：

```java
java.util.Arrays.sort( text );
```

专门用于处理数组的函数包括一个可以用来打印完整数组的方法，就像它们是字符串一样。这在调试程序时非常有用：

```java
System.out.println( java.util.Arrays.toString( text ) );
```

这将打印数组并显示每个元素，用逗号分隔，并用方括号括起来，`[]`。如果您在排序声明的字符串数组之后执行前面的命令，结果将是：

```java
[buy, more, spam]
```

如您所见，数组已按升序字母顺序排序。打印数组的方式与使用`for`循环遍历数组的方式不同：

```java
for (int i = 0; i < text.length; i++)
    System.out.print(text[i] + " ");
```

这将给出以下结果：

```java
buy more spam
```

如果您想以稍微干净一些的方式编写代码，可以在程序开始时导入整个`java.util.Arrays` API，这将允许您通过省略命令中的`java.util`部分来调用方法。以下示例突出了这一技术：

```java
import java.util.Arrays;
public class Example01 {
    public static void main(String[] args) {
        String[] text = new String[] { "spam", "more", "buy" };
        Arrays.sort(text);
        System.out.println(Arrays.toString(text));
        for (int i = 0; i < text.length; i++)
            System.out.print(text[i] + " ");
    }
}
```

结果将是：

```java
[buy, more, spam]
buy more spam 
Process finished with exit code 0
```

如果您想要创建一个新数组，希望所有单元格都填充相同的数据，可以使用`java.util.Arrays.fill()`方法，如下所示：

```java
int[] numbers = new int[5];
Arrays.fill(numbers, 0);
```

这样的命令将创建一个填充为零的数组：

```java
[0, 0, 0, 0, 0]
```

使用现有数组的副本也可以创建带有预填充数据的数组。可以通过复制一个数组的部分来创建一个数组，或者实例化一个更大的数组，其中旧数组只是它的一部分。以下示例展示了这两种方法，您可以在您的编辑器中测试：

```java
import java.util.Arrays;
public class Example02 {
    public static void main(String[] args) {
        int[] numbers = new int[5];
        Arrays.fill(numbers, 1);
        System.out.println(Arrays.toString(numbers));
        int [] shortNumbers = Arrays.copyOfRange(numbers, 0, 2);
        System.out.println(Arrays.toString(shortNumbers));
        int [] longNumbers = Arrays.copyOf(numbers, 10);
        System.out.println(Arrays.toString(longNumbers));
    }
}
```

此示例将打印`numbers`、`shortNumbers`（较短）和`longNumbers`（较长）数组。数组中新添加的位置将被零填充。如果它是一个字符串数组，它们将被`null`填充。此示例的结果是：

```java
[1, 1, 1, 1, 1]
[1, 1]
[1, 1, 1, 1, 1, 0, 0, 0, 0, 0]
Process finished with exit code 0
```

您可以通过调用`java.utils.Arrays.equals()`或`java.util.Arrays.deepEquals()`方法来比较数组。它们之间的区别在于后者可以查看嵌套数组。以下是一个使用前者的简单比较示例：

```java
import java.util.Arrays;
public class Example03 {
    public static void main(String[] args) {
        int[] numbers1 = new int[3];
        Arrays.fill(numbers1, 1);
        int[] numbers2 = {0, 0, 0};
        boolean comparison = Arrays.equals(numbers1, numbers2);
        System.out.println(comparison);
        int[] numbers3 = {1, 1, 1};
        comparison = Arrays.equals(numbers1, numbers3);
        System.out.println(comparison);
        int[] numbers4 = {1, 1};
        comparison = Arrays.equals(numbers1, numbers4);
        System.out.println(comparison);
    }
}
```

在这个例子中，我们创建了四个数组：`numbers1`、`numbers2`、`numbers3`和`numbers4`。其中只有两个数组相同，包含三个`1`的实例。在示例中，您可以看到最后三个数组是如何与第一个数组进行比较的。您还可以看到最后一个数组在内容上没有区别，但大小不同。此代码的结果是：

```java
false
true
false
Process finished with exit code 0
```

注意

由于本章没有探讨像嵌套数组这样复杂的数据结构，因此我们不会展示`java.util.Arrays.deepEquals()`的示例。如果您感兴趣，应该考虑查阅[`packt.live/2MuRrNa`](https://packt.live/2MuRrNa)上的 Java 参考文档。

在数组中进行搜索是通过后台的不同算法完成的。显然，在排序数组上执行搜索比在未排序数组上要快得多。要在排序数组上运行此类搜索，应调用`Arrays.binarySearch()`方法。由于它有许多可能的参数组合，建议访问该方法的官方文档。以下示例说明了它是如何工作的：

```java
import java.util.Arrays;
public class Example04 {
    public static void main(String[] args) {
        String[] text = {"love","is", "in", "the", "air"};
        int search = Arrays.binarySearch(text, "is");
        System.out.println(search);
    }
}
```

此代码将在数组 text 中搜索单词`the`。结果是：

```java
-4
Process finished with exit code 0
```

这是错误的！`binarySearch`是集合框架内的优化搜索算法，但与未排序数组一起使用时并不最优。这意味着`binarySearch`主要用于确定是否可以在数组中找到对象（通过先对其进行排序）。同时，当我们必须搜索未排序的数组或存在多个值时，我们需要不同的算法。

尝试以下对先前示例的修改：

```java
String[] text = {"love","is", "in", "the", "air"};
Arrays.sort(text);
int search = Arrays.binarySearch(text, "is");
System.out.println(search);
```

由于数组已排序，结果将是：

```java
2
Process finished with exit code 0
```

在这种情况下，"`is`"恰好出现在未排序和排序数组版本中的相同位置只是一个巧合。利用您一直在学习的工具，您应该能够创建一个算法，该算法可以遍历数组并计算所有现有项目，即使它们是重复的，以及它们在数组中的位置。请参阅本章中的*活动 1*，*在数组中搜索多个出现*，其中我们挑战您编写这样的程序。

你还可以使用`java.util.Arrays`类的`Arrays.toString()`方法将对象转换为字符串，就像我们在本节开头看到的那样，使用`Arrays.asList()`（我们将在后面的章节中看到，以及`Example05`）或使用`Arrays.setAll()`转换为集合。

数组和集合在软件开发中扮演着重要的角色。本章的这一部分深入探讨了它们之间的区别以及它们如何一起使用。如果你在网上搜索这两个构造之间的关系，你找到的大多数参考资料都将集中在它们的区别上，例如：

+   数组有固定的大小，而集合有可变的大小。

+   数组可以持有任何类型的对象，也可以持有原始数据类型；集合不能包含原始数据类型。

+   数组将持有同质元素（所有元素性质相同），而集合可以持有异质元素。

+   数组没有底层的数据结构，而集合使用标准结构实现。

如果你知道你将要处理的数据量，数组是首选的工具，主要是因为在这种情况下数组的表现优于列表或集合。然而，会有无数的情况，你不知道你将要处理的数据量，这时候列表就会变得很有用。

此外，数组还可以用于以编程方式填充集合。我们将在本章中这样做，以节省你手动输入最终将存储在集合中的所有数据的时间，例如。以下示例显示了如何使用数组填充一个集合：

```java
import java.util.*;
public class Example05 {
    public static void main(String[] args) {
        Integer[] myArray = new Integer[] {3, 25, 2, 79, 2};
        Set mySet = new HashSet(Arrays.asList(myArray));
        System.out.println(mySet);
    }
}
```

在这个程序中，有一个`Integer`类型的数组被用来初始化`HashSet`类的一个对象，这个对象随后被打印出来。

这个示例的结果是：

```java
[2, 3, 25, 79]
Process finished with exit code 0
```

之前的代码示例显示了几个有趣的事情。首先，你会注意到程序输出的结果是排序的；这是因为使用`Arrays.asList()`将数组转换为列表会使数据集继承列表的性质，这意味着它将是排序的。此外，由于数据已经添加到集合中，而集合不包含重复项，所以第二个重复的数字被省略了。

重要的是要注意，使用集合，你可以指定要存储的类型。因此，在之前的示例中，我们展示了泛型声明，以及接下来的声明之间会有所不同。类型在这里使用尖括号内的名称声明，即`<>`。在这种情况下，它是`<Integer>`。你可以将对象的实例化重写如下：

```java
Set<Integer> mySet = new HashSet<Integer>(Arrays.asList(myArray));
```

你会发现程序执行的结果将是相同的。

## 活动 1：在数组中搜索多个出现

编写一个程序，用于在字符串数组中搜索某个单词的多个出现，其中每个对象都是一个单独的单词。以下是一个著名的弗兰克·扎帕语录的数组，作为出发点：

```java
String[] text = {"So", "many", "books", "so", "little", "time"};
```

要搜索的单词是 `so`，但您必须考虑到它出现了两次，并且其中一个实例不是小写。作为一个提示，比较两个字符串而不看它们中任何字母的具体大小写的方方法是 `text1.compareToIgnoreCase(text2)`。为此，请执行以下步骤：

1.  创建 `text` 数组。

1.  创建包含要搜索的单词的变量：`so`

1.  将变量 `occurrence` 初始化为 -1。

1.  创建一个循环来遍历数组以检查出现。

这将给出以下结果：

```java
Found query at: 0
Found query at: 3
Process finished with exit code 0
```

注意

此活动的解决方案可以在第 538 页找到。

# 集合

集合框架中的集合是数学集合的程序等效。这意味着它们可以存储特定类型的对象，同时避免重复。同样，集合提供的方法将允许您以数学的方式处理数据。您可以将对象添加到集合中，检查集合是否为空，将两个集合的元素合并以将所有元素添加到单个集合中，查看两个集合之间有什么对象是相同的，以及计算两个集合之间的差异。

在 `java.util.Sets` 类中，我们发现有三个接口用于表示集合：`HashSet`、`TreeSet` 和 `LinkedHashSet`。它们之间的区别是直接的：

+   `HashSet` 将存储数据，但不保证迭代顺序。

+   `TreeSet` 按值对集合进行排序。

+   `LinkedHashSet` 按到达时间对集合进行排序。

每个接口都旨在在特定情况下使用。让我们从 `Example05` 中的集合出发，看看我们如何添加其他方法来检查如何操作集合。第一步是从数组中填充集合。有几种方法可以做到这一点；让我们使用最快速实现的方法：

```java
import java.util.*;
public class Example06 {
    public static void main(String[] args) {
        String[] myArray = new String[] {"3", "25", "2", "79", "2"};
        Set mySet = new HashSet();
        Collections.addAll(mySet, myArray);
        System.out.println(mySet);
    }
}
```

上述代码行显示了如何将数组的所有元素添加到集合中；当打印结果时，我们得到：

```java
[2, 79, 3, 25]
Process finished with exit code 0
```

请注意，输出的顺序可能因您而异。如前所述，`HashSet` 由于其实现方式，无法保证内容的任何排序。如果您使用 `Integer` 而不是 `String` 作为数据执行以下示例，最终结果将是排序的：

```java
import java.util.*;
public class Example07 {
    public static void main(String[] args) {
        Integer[] myArray = new Integer[] {3, 25, 2, 79, 2};
        Set mySet = new HashSet();
        Collections.addAll(mySet, myArray);
        System.out.println(mySet);
    }
}
```

该程序的结果如下：

```java
[2, 3, 25, 79]
Process finished with exit code 0
```

这意味着结果最终是排序的，即使我们没有请求它。

注意

在此示例中，集合是排序的只是一个巧合。请意识到在其他情况下可能并非如此。`Example08` 将展示两个集合之间的并集操作，那里的数据将不会排序。

与集合一起工作涉及处理数据包并对其执行操作。以下示例显示了两个集合的并集操作：

```java
import java.util.*;
public class Example08 {
    public static void main(String[] args) {
        Integer[] numbers1 = new Integer[] {3, 25, 2, 79, 2};
        Integer[] numbers2 = new Integer[] {7, 12, 14, 79};
        Set set1 = new HashSet();
        Collections.addAll(set1, numbers1);
        Set set2 = new HashSet();
        Collections.addAll(set2, numbers2);
        set1.addAll(set2);
        System.out.println(set1);
    }
}
```

此程序将打印出两个数组在示例主方法开头描述的集合的并集的结果：

```java
[2, 3, 7, 25, 12, 14, 79]
Process finished with exit code 0
```

除了`HashSet`，我们还发现`TreeSet`，在这里数据将按值排序。让我们简单地改变上一个例子中集合的类型，看看结果：

```java
Set set1 = new TreeSet();
Collections.addAll(set1, numbers1);
Set set2 = new TreeSet();
Collections.addAll(set2, numbers2);
```

当在上一个例子中改变时，这将给出以下排序后的集合作为结果：

```java
[2, 3, 7, 12, 14, 25, 79]
```

你可能想知道使用每种类型集合的优缺点。在排序时，你是在速度和整洁之间做权衡。因此，如果你正在处理大量数据且速度是一个问题，你必须决定是让系统运行得更快，还是让结果排序，这样就可以更快地通过数据集进行二分搜索。

给了最后一个修改，我们可以对数据进行其他操作，例如交集操作，该操作通过`set1.retainAll(set2)`方法调用。让我们看看它的实际效果：

```java
import java.util.*;
public class Example09 {
    public static void main(String[] args) {
        Integer[] numbers1 = new Integer[] {3, 25, 2, 79, 2};
        Integer[] numbers2 = new Integer[] {7, 12, 14, 79};
        Set set1 = new TreeSet();
        Collections.addAll(set1, numbers1);
        Set set2 = new TreeSet();
        Collections.addAll(set2, numbers2);
        set1.retainAll(set2);
        System.out.println(set1);
    }
}
```

对于输出，由于数组被用来填充数组，我们只会得到存在于两个数组中的那些数字；在这种情况下，只是数字`79`：

```java
[79]
Process finished with exit code 0
```

第三种类型的集合，`LinkedHashSet`，将按对象到达的顺序对对象进行排序。为了演示这种行为，让我们编写一个程序，该程序将使用`set.add(element)`命令逐个向集合中添加元素。

```java
import java.util.*;
public class Example10 {
    public static void main(String[] args) {
        Set set1 = new LinkedHashSet();
        set1.add(35);
        set1.add(19);
        set1.add(11);
        set1.add(83);
        set1.add(7);
        System.out.println(set1);
    }
}
```

当运行这个例子时，结果将按数据到达集合的方式排序：

```java
[35, 19, 11, 83, 7]
Process finished with exit code 0
```

为了实验，请用接下来的 2 分钟再次将集合构造为`HashSet`：

```java
Set set1 = new LinkedHashSet();
```

这个修改后的程序的结果是不确定的。例如，我们得到：

```java
[35, 19, 83, 7, 11]
Process finished with exit code 0
```

这又是同一组数据的未排序版本。

为了结束我们对你可以与集合一起使用的可能方法的解释，让我们使用`LinkedHashSet`运行一个实验，我们将找到两个集合之间的差异。

```java
import java.util.*;
public class Example11 {
    public static void main(String[] args) {
        Set set1 = new LinkedHashSet();
        set1.add(35);
        set1.add(19);
        set1.add(11);
        set1.add(83);
        set1.add(7);
        Set set2 = new LinkedHashSet();
        set2.add(3);
        set2.add(19);
        set2.add(11);
        set2.add(0);
        set2.add(7);
        set1.removeAll(set2);
        System.out.println(set1);
    }
}
```

在这种情况下，两个集合略有不同，通过确定差异，`set1.removeAll(set2)`算法背后的算法将在`set1`中查找`set2`中每个项目的出现，并将它们消除。这个程序的结果是：

```java
[35, 83]
Process finished with exit code 0
```

最后，如果你只想检查一个集合是否完全包含在另一个集合中，你可以调用`set1.containsAll(set2)`方法。我们将把这个留给你去探索——只需注意，该方法简单地返回一个布尔值，表示该语句是真是假。

# 列表

列表是有序的数据集合。与集合不同，列表可以有重复的数据。在列表中包含数据允许你执行搜索，这将给出给定列表中某些对象的位置。给定一个位置，你可以直接访问列表中的项目，添加新项目，删除项目，甚至添加完整的列表。列表是顺序的，这使得它们很容易通过迭代器进行导航，这一特性将在本章后面的部分中详细探讨。还有一些方法可以对子列表执行基于范围的操作。

有两种不同的列表实现：`ArrayList`和`LinkedList`。根据情况，每个都是理想的。在这里，我们将主要使用`ArrayList`。让我们首先创建并填充一个实例，然后搜索列表中的某个值，并根据其在列表中的位置打印出该值。

```java
import java.util.*;
public class Example12 {
    public static void main(String[] args) {
        List list = new ArrayList();
        list.add(35);
        list.add(19);
        list.add(11);
        list.add(83);
        list.add(7);
        System.out.println(list);
        int index = list.indexOf(19);
        System.out.println("Find 19 at: " + index);
        System.out.println("Component: " + list.get(index));
    }
}
```

这个示例的输出如下：

```java
[35, 19, 11, 83, 7]
Find 19 at: 1
Component: 19
Process finished with exit code 0
```

`indexOf`方法会告诉你传递给方法的对象的位置。它的兄弟方法`lastIndexOf`报告列表中对象的最后一个出现位置。

你应该将列表视为由链接连接的一系列节点。如果一个节点被消除，曾经指向它的链接将被重定向到列表中的下一个项目。当添加节点时，它们默认附加到列表的末尾（如果它们不是重复的）。由于集合中的所有节点都是同一类型，因此应该可以在列表中交换两个节点的位置。

让我们实验一下从列表中删除一个项目，并确定删除项目前后立即定位的对象的位置：

```java
import java.util.*;
public class Example13 {
    public static void main(String[] args) {
        List list = new ArrayList();
        list.add(35);
        list.add(19);
        list.add(11);
        list.add(83);
        list.add(7);
        System.out.println(list);
        int index = list.lastIndexOf(83);
        System.out.println("Before: find 83 at: " + index);
        list.remove(index - 1);
        System.out.println(list);
        index = list.lastIndexOf(83);
        System.out.println("After: find 83 at: " + index);
    }
}
```

这个程序创建了一个列表，将其打印出来，然后在列表中查找一个节点并打印其位置。然后，它从列表中删除一个项目，并重复之前的步骤以显示该节点已从列表中删除。这与数组的情况明显不同，在数组中无法删除项目，因此无法更改其大小。观察前一个示例的输出：

```java
[35, 19, 11, 83, 7]
Before: find 83 at: 3
[35, 19, 83, 7]
After: find 83 at: 2
Process finished with exit code 0
```

也可以更改节点的内容。在前面的示例中，不是删除节点，而是将`list.remove(index-1);`更改为以下内容并检查结果：

```java
list.set(index - 1, 99); 
```

最终数组将用`11`替换`99`。

如果你不想删除一个节点，而是想清空整个列表，那么向其发出的命令将是：

```java
list.clear();
```

使用`subList()`操作符，可以从列表生成列表，例如，可以删除列表中一系列单元格。请看以下示例，它删除了字符串数组的一部分，在打印时改变了其含义：

```java
import java.util.*;
public class Example14 {
    public static void main(String[] args) {
        List list = new ArrayList();
        list.add("No");
        list.add("matter");
        list.add("what");
        list.add("you");
        list.add("do");
        System.out.println(list);
        list.subList(2,4).clear();
        System.out.println(list);
    }
}
```

看看以下结果：

```java
[No, matter, what, you, do]
[No, matter, do]
Process finished with exit code 0
```

通过运行示例代码修改了`list`对象，使其变短。`subList()`方法中使用的两个索引数字是列表中方法开始和停止的位置。`subList()`的结果也可以分配给相同类型的另一个变量，在执行`subList()`操作后，代码中的列表将减少一个副本。

看看最新代码列表中的以下修改：

```java
List list1 = list.subList(2,4);
System.out.println(list1);
```

这将打印出由前一个示例中删除的节点组成的列表。

集合框架中有许多有趣的算法，提供了操作列表的相关功能：

+   `sort`：将列表的元素按特定顺序排列。

+   `shuffle`：随机化列表中所有对象的位置。

+   `reverse`：反转列表的顺序。

+   `rotate`：将对象移动到列表的末尾，当它们到达末尾时，在另一端显示。

+   `swap`：交换两个元素。

+   `replaceAll`：使用参数替换列表中所有元素的出现。

+   `fill`：使用一个值填充列表的内容。

+   `copy`：创建列表的更多实例。

+   `binarySearch`：在列表中执行优化的搜索。

+   `indexOfSubList`：搜索列表中某个片段（一组连续节点）的出现。

+   `lastIndexOfSubList`：搜索列表中某个片段的最后一个出现位置。

    注意

    使用 `Arrays.asList()` 从数组生成的列表与本章中描述的 `List` 类对象的行为不同。来自数组的列表具有固定长度，这意味着无法从数组中删除元素。这是因为 `java.util.Arrays` 在包内部实现了自己的 `ArrayList` 类，它与集合框架中的类不同。这不是很令人困惑吗？

## 练习 1：创建 AnalyzeInput 应用程序

在这个练习中，我们将创建一个新的应用程序，该应用程序将通过存储提供给它的任何字符串来响应 CLI，然后对数据进行一些统计操作，例如单词计数（确定最频繁的单词或最频繁的字母等）。目的是让你了解如何使用集合框架而不是其他工具来完成此类操作。这次，我们将做一些特别的事情；而不是从 CLI 作为脚本的参数获取数据，我们将使用 `java.io.Console` API，它允许从终端读取不同类型的字符串，例如用户名（普通字符串）和密码。这个应用程序的目标是读取输入，直到捕获到只有 "`*`" 符号（星号）的行。一旦输入了终止符号，文本将被处理，并将统计结果发送到终端：

1.  打开 IntelliJ 并使用 CLI 模板创建一个新的 Java 程序。将项目命名为 `AnalyzeInput`。

1.  首先，创建一个简单的程序，可以从终端读取一行并将其打印出来：

    ```java
    import java.io.Console;
    public class AnalyzeInput {
      public static void main(String[] args) {
        Console cons;
        String line = "";
          if ((cons = System.console()) != null && (line = cons.readLine()) !=         null) {
          System.out.println("You typed: " + line);
        }
      }
    }
    ```

1.  从 CLI 中执行程序，通过在正确的文件夹中调用 `java AnalyzeInput` 来与之交互：

    ```java
    usr@localhost:~/IdeaProjects/ch04/out/production/ch04$ java AnalyzeInput 
    hej this is an example
    You typed: hej this is an example
    ```

1.  您必须导入 `java.io.Console`，这允许您实例化 `Console` 类的对象。您还可以看到对 `cons = System.console()` 的调用，这将确保终端已准备好供您读取数据，以及 `line = cons.readLine()`，这将确保在按下键盘上的 *Enter* 键时，结果数据不为空。

1.  下一步是将我们正在捕获的数据存储在集合中。由于我们不知道这个大小，我们应该使用`ArrayList <String>`来存储数据。此外，为了存储我们想要存储的数据，我们可以修改`if`语句并将其改为`while`循环。最后，使用`add`方法将行添加到列表中（请注意，以下代码列表永远不会退出，所以请耐心等待，不要现在执行它）：

    ```java
    import java.util.*;
    import java.io.Console;
    public class Exercise01 {
      public static void main(String[] args) {
        ArrayList <String> text = new ArrayList<String>();
        Console cons;
        String line = "";
        while ((cons = System.console()) != null && (line = cons.readLine())       != null) {
        text.add(line);
        }
        System.out.println("You typed: " + text);
      }
    }
    ```

1.  修改`while`循环，包括我们为完成数据捕获过程设定的条件——只有星号符号的一行：

    ```java
            while (!line.equals("*") 
                && (cons = System.console()) != null 
                && (line = cons.readLine()) != null) {
    ```

1.  结果只有在你在一行中单独输入星号符号时才会发生，就像在与程序交互时的这个日志中看到的那样：

    ```java
    usr@localhost:~/IdeaProjects/ch04/out/production/ch04$ java AnalyzeInput 
    this is the array example
    until you type *
    alone in a line
    *
    You typed: [this is the array example, until you type *, alone in a line, *]
    ```

1.  由于我们使用了`ArrayList`来存储不同的字符串，你可能需要一直输入，直到耗尽计算机的内存。现在，我们可以执行一些命令来处理字符串。第一步是将整个文本转换成一个列表。这需要遍历不同的字符串并将它们分割成将要添加到更大列表中的部分。最简单的技巧是使用`split()`方法，以空格字符作为分隔符。修改`main`方法，使其看起来如下，你就会看到结果现在是一个列表，其中所有的单词都作为单独的节点分开：

    ```java
        public static void main(String[] args) {
            ArrayList <String> text = new ArrayList<String>();
            Console cons;
            String line = "";
            while (!line.equals("*")
                && (cons = System.console()) != null
                && (line = cons.readLine()) != null) {
                    List<String> lineList = new                   ArrayList<String>(Arrays.asList(line.split(" ")));
                text.addAll(lineList);
            }
            System.out.println("You typed: " + text);
        }
    ```

1.  以这种方式存储所有数据允许使用集合框架中可用的许多方法，这将让你能够对数据进行操作。让我们从计算文本中的所有单词（包括关闭符号，“`*`”）开始。只需在`main`方法的末尾添加以下内容：

    ```java
    System.out.println("Word count: " + text.size());
    ```

这个练习的结果是一个可以用于进一步分析数据的程序。但是，为了继续这样做，我们需要使用一个尚未介绍的工具——迭代器。我们将在本章的后面回到这个例子，并通过添加一些额外的功能来完成应用程序。

# 映射

集合框架提供了一个额外的接口，`java.util.Map`，当处理以键值对形式存储的数据时可以使用。这种类型的数据存储越来越相关，因为像 JSON 这样的数据格式正逐渐接管互联网。JSON 是一种基于嵌套数组形式存储数据的数据格式，这些数组总是响应键值结构。

以这种方式组织数据提供了通过键而不是，例如，使用索引（就像我们在数组中做的那样）来查找数据的一种非常简单的方法。键是我们可以在映射中识别我们正在寻找的数据块的方式。在我们查看映射的替代方案之前，让我们先看看一个简单的映射示例：

以下示例展示了如何创建一个简单的映射以及如何根据映射中可用的信息打印一些消息。与其他集合框架中的接口相比，你首先会注意到的是，我们不是向映射中*添加*元素，而是将元素*放入*映射中。此外，元素有两个部分：**键**（在我们的例子中，我们使用字符串）和**值**（其本质可以是异质的）：

```java
import java.util.*;
public class Example15 {
    public static void main(String[] args) {
        Map map = new HashMap();
        map.put("number", new Integer(1));
        map.put("text", new String("hola"));
        map.put("decimal", new Double(5.7));
        System.out.println(map.get("text"));
        if (!map.containsKey("byte")) {
            System.out.println("There are no bytes here!");
        }
    }
}
```

这个程序将给出以下结果：

```java
hola
There are no bytes here!
Process finished with exit code 0
```

由于代码中没有名为"`bytes`"的键，`maps.containsKey()`方法将相应地回答，程序将通知用户这一点。此接口中可用的主要方法有：

+   `put` (Object key, Object value)

+   `putAll` (Map map)

+   `remove` (Object key)

+   `get` (Object key)

+   `containsKey` (Object key)

+   `keySet()`

+   `entrySet()`

除了最后两个之外，其他都是不言自明的。让我们关注增强我们之前的示例，看看这两个方法的作用。在代码中添加以下内容以查看`keySet()`和`entrySet()`能提供什么：

```java
System.out.println(map.entrySet());
System.out.println(map.keySet());
```

修改后的代码列表的结果将是：

```java
hola
There are no bytes here!
[number=1, text=hola, decimal=5.7]
[number, text, decimal]
Process finished with exit code 0
```

换句话说，`entrySet()`将使用键 = 值公式打印整个映射，而`keySet()`将返回映射中的键集合。

注意

你可能现在已经意识到了：键必须是唯一的——映射中不能有两个相同的键。

我们在此处不会深入探讨映射，因为它们在一定程度上是集合的重复。映射有三个不同的类：`HashMap`、`TreeMap`和`LinkedHashMap`。后两者是有序的，而第一个既没有排序也没有按到达顺序排列。你应该根据你的需求使用这些类。

# 集合的迭代

在本章早期，当我们正在处理*练习 01，创建 AnalyzeInput 应用程序*时，我们停止了搜索数据的操作。我们做到了必须遍历数据并查找诸如词频等特征的程度。

迭代器用于 Java 遍历集合。让我们看看一个简单的例子，该例子涉及逐个提取简单列表中的元素并打印它们。

```java
import java.util.*;
public class Example16 {
    public static void main(String[] args) {
        List array = new ArrayList();
        array.add(5);
        array.add(2);
        array.add(37);
        Iterator iterator = array.iterator();
        while (iterator.hasNext()) {
            //  point to next element
            int i = (Integer) iterator.next();
            // print elements
            System.out.print(i + " ");
        }
    }
}
```

这个程序的结果是：

```java
5 2 37 
Process finished with exit code 0
```

这种类型的迭代器是集合框架中最通用的，可以与列表、集合、队列以及甚至映射一起使用。还有其他更窄的迭代器实现，允许以不同的方式浏览数据，例如在列表中。正如你在最新的代码列表中看到的，`iterator.hasNext()`方法检查列表中我们所在节点之后的节点是否存在。当启动迭代器时，对象指向列表中的第一个元素。然后，`hasNext()`会响应一个布尔值，表示是否有更多的节点悬挂在其上。`iterator.next()`方法将迭代器移动到集合中的下一个节点。这种迭代器没有在集合中回退的可能性；它只能向前移动。迭代器中还有一个最终的方法，称为`remove()`，它将从集合中删除迭代器所指向的当前元素。

如果我们使用`listIterator()`，我们将有更多的选项来导航集合，例如添加新元素和修改元素。以下代码示例演示了如何遍历列表，添加元素并修改它们。`listIterator`仅与列表一起工作：

```java
import java.util.*;
public class Example17 {
    public static void main(String[] args) {
        List <Double> array = new ArrayList();
        array.add(5.0);
        array.add(2.2);
        array.add(37.5);
        array.add(3.1);
        array.add(1.3);
        System.out.println("Original list: " + array);
        ListIterator listIterator = array.listIterator();
        while (listIterator.hasNext()) {
            //  point to next element
            double d = (Double) listIterator.next();
            // round up the decimal number
            listIterator.set(Math.round(d));
        }
        System.out.println("Modified list: " + array);
    }
}
```

在这个例子中，我们创建了一个`Double`列表，遍历列表，并对每个数字进行四舍五入。这个程序的输出是：

```java
Original list: [5.0, 2.2, 37.5, 3.1, 1.3]
Modified list: [5, 2, 38, 3, 1]
Process finished with exit code 0
```

通过调用`listIterator.set()`，我们修改列表中的每个项目，第二个`System.out.println()`命令显示了数字是如何被四舍五入或向下取整的。

在本节中我们将看到的最后一个迭代器示例是一个遍历映射的技巧。这可能在需要在对映射中的数据进行某些操作的场景中很有用。通过使用`entrySet()`方法——它返回一个列表——可以有一个映射的迭代器。请看以下示例以了解这是如何工作的：

```java
import java.util.*;
public class AnalyzeInput {
     public static void main(String[] args) {
        Map map = new HashMap ();
        map.put("name", "Kristian");
        map.put("family name", "Larssen");
        map.put("address", "Jumping Rd");
        map.put("mobile", "555-12345");
        map.put("pet", "cat");
        Iterator <Map.Entry> iterator = map.entrySet().iterator();
        while (iterator.hasNext()) {
            Map.Entry entry = iterator.next();
            System.out.print("Key = " + entry.getKey());
            System.out.println( ", Value = " + entry.getValue());
        }
    }
}
```

这个程序将遍历一个映射，并按它们在`HashMap`中存储的内容打印出来。请记住，这些类型的对象没有按任何特定的方式进行排序。你可以期待以下输出：

```java
Key = address, Value = Jumping Rd
Key = family name, Value = Larssen
Key = name, Value = Kristian
Key = mobile, Value = 555-12345
Key = pet, Value = cat
Process finished with exit code 0
```

既然我们现在有了遍历集合的方法，我们可以继续进行一个练习，即迭代列表进行数据分析。

## 练习 2：将分析功能引入 AnalyzeInput 应用程序

我们将从*练习 1*，*创建 AnalyzeInput 应用程序*的结尾开始。我们成功捕获了在终端中输入的文本，并将其存储为字符串列表。这次，我们将使用集合框架中的一个方法，称为`frequency`，它将返回一个对象在列表中可以找到的次数。由于句子中的单词可能会重复，我们首先需要找出一种方法来提取列表中的唯一元素：

1.  集合是集合框架中的对象，它只保留每个元素的一个副本。我们在本章的早期看到了一个例子。我们将创建一个`HashSet`实例，并将列表中的所有元素复制到其中。这将自动消除重复项：

    ```java
    Set <String> textSet = new HashSet <String> ();
    textSet.addAll(text);
    ```

1.  现在我们有了集合，下一步是创建一个迭代器，它会检查集合中的每个元素在列表中可以找到多少个副本：

    ```java
    Iterator iterator = textSet.iterator();
    ```

1.  使用我们在前面的例子中看到的技术，即如何遍历一个集合，我们将找到集合中的下一个节点，并在列表中检查节点中存储的字符串的频率：

    ```java
    while (iterator.hasNext()) {
        //point to next element
        String s = (String) iterator.next();
        // get the amount of times this word shows up in the text
        int freq = Collections.frequency(text, s);
        // print out the result
        System.out.println(s + " appears " + freq + " times");
    }
    ```

    注意

    最终的代码可以参考：[`packt.live/2BrplvS`](https://packt.live/2BrplvS)。

1.  结果将取决于你输入的文本类型。为了测试，请尝试以下内容（我们将在本章的其余部分坚持使用这个数据输入 – 你每次调用应用程序时都可以将其复制并粘贴到终端中）：

    ```java
    this is a test
    is a test
    test is this
    *
    ```

1.  这个输入的完整结果将是：

    ```java
    You typed: [this, is, a, test, is, a, test, test, is, this, *]
    Word count: 11
    a appears 2 times
    test appears 3 times
    this appears 2 times
    is appears 3 times
    * appears 1 times
    ```

    虽然结果是正确的，但阅读起来并不容易。理想情况下，结果应该按顺序排序。例如，按频率的降序排列，这样就可以一眼看出最频繁和最不频繁的单词。这是我们在继续前进之前再次停下来进行练习的时候，因为我们需要在继续之前介绍排序的概念。

# 排序集合

正如我们所见，在集合框架中有些类强制其内部的项进行排序。例如`TreeSet`和`TreeMap`。本节要探讨的方面是如何使用现有的排序机制对列表进行排序，以及对于每个数据点有多个值的数据集的情况。

我们在本章中进行的练习是一个很好的例子，其中存在具有多个值的点。对于每个数据点，我们需要存储我们正在计算频率的单词以及频率本身。你可能认为一个好的技术是将信息以映射的形式存储。独特的单词可以是键，而频率可以是值。这可以通过修改上一个程序的最后一部分来实现，如下所示：

```java
Map map = new HashMap();
while (iterator.hasNext()) {
    //  point to next element
    String s = (String) iterator.next();
    // get the amount of times this word shows up in the text
    int freq = Collections.frequency(text, s);
    // print out the result
    System.out.println(s + " appears " + freq + " times");
    // add items to the map
    map.put(s, freq);
}
TreeMap mapTree = new TreeMap();
mapTree.putAll(map);
System.out.println(mapTree);
```

虽然这是一个有趣且简单的方法来排序（将数据复制到按自然排序的结构中），但它提出了一个问题，即数据是按键排序而不是按值排序，正如以下代码的输出所强调的：

```java
Word count: 11
a appears 2 times
test appears 3 times
this appears 2 times
is appears 3 times
* appears 1 times
{*=1, a=2, is=3, test=3, this=2}
```

因此，如果我们想按值对这些结果进行排序，我们需要找出一种不同的策略。

但让我们退一步，分析一下集合框架中提供的哪些工具用于排序。有一个名为`sort()`的方法可以用来排序列表。以下是一个例子：

```java
import java.util.*;
public class Example19 {
    public static void main(String[] args) {
        List <Double> array = new ArrayList();
        array.add(5.0);
        array.add(2.2);
        array.add(37.5);
        array.add(3.1);
        array.add(1.3);
        System.out.println("Original list: " + array);
        Collections.sort(array);
        System.out.println("Modified list: " + array);
    }
}
```

这个程序的结果是：

```java
Original list: [5.0, 2.2, 37.5, 3.1, 1.3]
Modified list: [1.3, 2.2, 3.1, 5.0, 37.5]
Process finished with exit code 0
```

给定一个列表，我们可以这样排序它；甚至可以使用`listIterator`向后导航，以降序排序列表。然而，这些方法并不能解决对具有多个值的数据点进行排序的问题。在这种情况下，我们需要创建一个类来存储我们自己的键值对。让我们通过继续我们在本章中一直在处理的练习来看看如何实现它。

## 练习 3：从 AnalyzeInput 应用程序中排序结果

我们现在有一个程序，给定一些输入文本，可以识别文本的一些基本特征，例如文本中的单词数量或每个单词的频率。我们的目标是能够按降序排序结果，使其更容易阅读。这个解决方案需要实现一个类来存储我们的键值对，并从这个类中创建一个对象列表：

1.  创建一个包含两个数据点的类：单词及其频率。实现一个构造函数，它将接受值并将它们传递给类变量。这将简化后面的代码：

    ```java
    class DataPoint {
        String key = "";
        Integer value = 0;
        // constructor
        DataPoint(String s, Integer i) {
            key = s;
            value = i;
        }
    }
    ```

1.  在计算每个单词的频率时，将结果存储在新创建的新类的对象列表中：

    ```java
            List <DataPoint> frequencies = new ArrayList <DataPoint> ();
            while (iterator.hasNext()) {
            //point to next element
            String s = (String) iterator.next();
            // get the amount of times this word shows up in the text
            int freq = Collections.frequency(text, s);
            // print out the result
            System.out.println(s + " appears " + freq + " times");
            // create the object to be stored
            DataPoint datapoint = new DataPoint (s, freq);
            // add datapoints to the list
            frequencies.add(datapoint);
        }
    ```

1.  排序需要创建一个新的类，使用`Comparator`接口，我们现在刚刚介绍。这个接口应该实现一个方法，该方法将在数组中的对象内运行比较。这个新类必须实现`Comparator <DataPoint>`并包含一个名为`compare()`的单个方法。它应该有两个参数，即正在排序的类的对象：

    ```java
    class SortByValue implements Comparator<DataPoint>
    {
        // Used for sorting in ascending order
        public int compare(DataPoint a, DataPoint b)
        {
            return a.value - b.value;
        }
    }
    ```

1.  我们使用这个新比较器调用`Collections.sort()`算法的方式是将该类的对象作为参数添加到`sort`方法中。我们直接在调用中实例化它：

    ```java
    Collections.sort(frequencies,new SortByValue());
    ```

1.  这将按升序排序频率列表。要打印结果，不再可以直接调用`System.out.println(frequencies)`，因为它现在是一个对象数组，它不会将数据点的内容打印到终端。而是以以下方式遍历列表：

    ```java
    System.out.println("Results sorted");
    for (int i = 0; i < frequencies.size(); i++)
        System.out.println(frequencies.get(i).value
                        + " times for word "
                        + frequencies.get(i).key);
    ```

1.  如果你使用我们之前在几个示例中使用过的相同输入运行程序，结果将是：

    ```java
    Results sorted
    1 times for word *
    2 times for word a
    2 times for word this
    3 times for word test
    3 times for word is
    ```

1.  我们的目的是按降序排序结果，为此，我们需要在调用`sort`算法时添加一个额外的元素。在实例化`SortByValue()`类时，我们需要告诉编译器我们希望列表按逆序排序。集合框架已经有一个方法可以做到这一点：

    ```java
    Collections.sort(frequencies, Collections.reverseOrder(new SortByValue()));
    ```

    注意

    为了清晰起见，最终代码可以参考：[`packt.live/2W5qhzP`](https://packt.live/2W5qhzP)。

1.  与这个程序的全交互路径，从我们调用它到包括数据输入，如下所示：

    ```java
    user@localhost:~/IdeaProjects/ch04/out/production/ch04$ java AnalyzeInput 
    this is a test
    is a test
    test is this
    *
    You typed: [this, is, a, test, is, a, test, test, is, this, *]
    Word count: 11
    a appears 2 times
    test appears 3 times
    this appears 2 times
    is appears 3 times
    * appears 1 times
    Results sorted
    3 times for word test
    3 times for word is
    2 times for word a
    2 times for word this
    1 times for word *
    ```

# 属性

在集合框架中，`Properties` 用于维护键值对列表，其中两者都是 `String` 类。当从操作系统获取环境值时，`Properties` 是相关的，并且是许多其他类的基类。`Properties` 类的一个主要特征是，它允许在搜索某个键不满意的情况下定义默认响应。以下示例突出了这种情况的基本原理：

```java
Example20.java
1  import java.util.*;
2  
3  public class Example20 {
4  
5     public static void main(String[] args) {
6         Properties properties = new Properties();
7         Set setOfKeys;
8         String key;
9  
10        properties.put("OS", "Ubuntu Linux");
11        properties.put("version", "18.04");
12        properties.put("language", "English (UK)");
13
14        // iterate through the map
15        setOfKeys = properties.keySet();
https://packt.live/2N0CzoS
```

在深入研究结果之前，您会注意到在属性中，我们使用 `put` 而不是 `add` 新元素/节点。这与我们之前看到的映射相同。此外，您会注意到，为了迭代，我们使用了 `keySet()` 技术，这是我们之前在遍历映射时看到的。最后，`Properties` 的特殊性在于，您可以在找不到搜索属性的情况下设置默认响应。这就是在示例中搜索 `getProperty()` 方法将返回其默认消息而不会使程序崩溃的原因。

这个程序的结果是：

```java
version = 18.04
OS = Ubuntu Linux
language = English (UK)
keyboard layout = not found
Process finished with exit code 0
```

在 `Properties` 类中还可以找到另一个有趣的方法，即 `list()`；它提供了两种不同的实现，允许您将列表的内容发送到不同的数据处理程序。我们可以将整个属性列表流式传输到 `PrintStreamer` 对象，例如 `System.out`。这提供了一种简单的方式来显示列表中的内容，而无需遍历它。以下是一个示例：

```java
import java.util.*;
public class Example21 {
    public static void main(String[] args) {
        Properties properties = new Properties();
        properties.put("OS", "Ubuntu Linux");
        properties.put("version", "18.04");
        properties.put("language", "English (UK)");
        properties.list(System.out);
    }
}
```

这将导致：

```java
version=18.04
OS=Ubuntu Linux
language=English (UK)
Process finished with exit code 0
```

`propertyNames()` 方法返回一个 `Enumeration` 列表，通过遍历它，我们将获得整个列表的键。这是创建集合并运行 `keySet()` 方法的替代方法。

```java
import java.util.*;
public class Example22 {
    public static void main(String[] args) {
        Properties properties = new Properties();
        properties.put("OS", "Ubuntu Linux");
        properties.put("version", "18.04");
        properties.put("language", "English (UK)");
        Enumeration enumeration = properties.propertyNames();
        while (enumeration.hasMoreElements()) {
            System.out.println(enumeration.nextElement());
        }
    }
}
```

这将导致：

```java
version
OS
language
Process finished with exit code 0
```

在这一点上，我们将从“属性”部分介绍给您的方法是 `setProperty()`。它将修改现有键的值，或者在找不到键的情况下最终创建一个新的键值对。如果键存在，该方法将返回旧值，否则返回 `null`。下一个示例将展示它是如何工作的：

```java
import java.util.*;
public class Example23 {
    public static void main(String[] args) {
        Properties properties = new Properties();
        properties.put("OS", "Ubuntu Linux");
        properties.put("version", "18.04");
        properties.put("language", "English (UK)");
        String oldValue = (String) properties.setProperty("language", "German");
        if (oldValue != null) {
            System.out.println("modified the language property");
        }   
        properties.list(System.out);
    }
}
```

下面是结果：

```java
modified the language property
-- listing properties --
version=18.04
OS=Ubuntu Linux
language=German
Process finished with exit code 0
```

注意

`Properties` 类中还有更多方法用于处理从文件中存储和检索属性列表。虽然这是 Java API 中的一个非常强大的功能，但由于我们尚未在本书中介绍文件的使用，所以我们不会在这里讨论这些方法。有关更多信息，请参阅 Java 的官方文档。

## 活动二：遍历大型列表

在当代计算中，我们处理大量数据集。这个活动的目的是创建一个随机大小的随机数列表，以便对数据进行一些基本操作，例如获取平均值。

1.  首先，你应该创建一个随机数字列表。

1.  要计算平均值，你可以创建一个迭代器，它会遍历值列表，并为每个元素添加相应的加权值。

1.  从`iterator.next()`方法返回的值必须在与其总元素数进行比较之前转换为`Double`类型。

如果你正确实现了所有内容，平均的结果应该类似于：

```java
Total amount of numbers: 3246
Average: 49.785278826074396
```

或者，它可能是：

```java
Total amount of numbers: 6475
Average: 50.3373892275651
```

注意

本活动的解决方案可以在第 539 页找到。

如果你设法使这个程序运行起来，你应该考虑如何利用能够模拟如此大量数据的能力。这些数据可能代表你的应用程序中不同数据到达之间的时间间隔，或者每秒从物联网网络中的节点捕获的温度数据。可能性是无限的。通过使用列表，你可以使数据集的大小像它们的可能性一样无限。

# 摘要

本章向您介绍了 Java 集合框架，这是 Java 语言中一个非常强大的工具，可以用来存储、排序和过滤数据。该框架非常庞大，提供了接口、类和方法等工具，其中一些超出了本章的范围。我们专注于`Arrays`、`Lists`、`Sets`、`Maps`和`Properties`。但还有其他一些，如队列和双端队列，值得你自己去探索。

与它们的数学等价物类似，集合存储唯一的项目副本。列表类似于可以无限扩展的数组，并支持重复项。当处理键值对时使用映射，这在当代计算中非常常见，并且不支持使用两个相同的键。属性的工作方式非常类似于`HashMap`（`Map`的一种特定类型），但提供了一些额外功能，例如将所有内容列出到流中，这简化了列表内容的打印。

框架中提供的某些类按设计排序，例如`TreeHash`和`TreeMap`，而其他类则不是。根据你想要如何处理数据，你必须决定哪种集合是最好的。

使用迭代器遍历数据有一些标准技术。这些迭代器在创建时将指向列表中的第一个元素。迭代器提供了一些基本方法，如`hasNext()`和`next()`，分别用于判断列表中是否有更多数据以及从列表中提取数据。虽然这两个方法对所有迭代器都是通用的，但还有一些其他方法，如`listIterator`，功能更强大，例如，在遍历列表的同时向列表中添加新元素。

我们已经查看了一章长度的示例，其中使用了这些技术中的许多，并且我们介绍了如何通过终端使用控制台读取数据。在下一章中，我们将介绍异常及其处理方法。
