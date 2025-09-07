# 13

# 泛型和集合

组织数据是另一个重要的软件开发主题。Java 为我们提供了集合来处理各种数据结构。它还提供了泛型来实现类型安全和避免在应用程序中重复代码。如果我们不了解如何使用集合和泛型，就不能说我们是 Java 的专家。

因此，我们专门用这一章来介绍 Java 集合框架。在本章中，我们将涵盖以下主题：

+   集合框架及其接口 – `List`、`Set`、`Map` 和 `Queue`

+   每种集合类型的不同实现及其基本操作

+   使用自然排序和 `Comparable` 以及 `Comparator` 接口对集合进行排序

+   使用泛型

+   基本哈希概念及其相关性

到本章结束时，你将牢固地理解 Java 集合框架和泛型，并准备好在程序中管理数据和使用集合。

# 技术要求

本章的代码（*练习*部分）可以在 GitHub 上找到：[`github.com/PacktPublishing/Learn-Java-with-Projects/tree/main/ch13/exercises`](https://github.com/PacktPublishing/Learn-Java-with-Projects/tree/main/ch13/exercises)。

# 了解集合

**集合**值得了解。集合是一种处理一个变量中多个值比数组更优雅的方式。一个常见的集合例子就是列表。

不使用集合编写任何合适的 Java 应用程序都会非常复杂。你可能首先会创建一些将充当 Java 内置集合的类。它们在软件开发中扮演着至关重要的角色，因为它们提供了一种管理和组织数据的方法。

我们需要它们的原因有很多，但让我们只列举（集合双关语）几个：

+   **管理大量数据**：随着应用程序的复杂性增加，它们通常需要处理大量数据。集合帮助存储和管理这些数据集。它们还提供了一些有用的方法，使得对数据进行典型操作（如搜索和过滤）变得更加容易。

+   **存储和操作各种数据结构**：不同的数据结构具有独特的特性，适用于特定的任务。集合提供了一系列的数据结构。这样，我们可以根据需求选择最合适的一个。

+   **确保高效的数据管理和访问**：集合提供了一系列的功能。这有助于我们在应用程序中优化数据管理和数据访问。

由于存在不同的数据结构，我们还需要不同的集合类型。让我们来看看它们。

## 不同集合类型的概述

Java 集合框架提供了相当多的不同集合类型。这确保了开发者不会为各种问题构建自定义数据结构类。这将使得不同应用程序之间的通信变得非常困难，并且需要大量样板代码来完成许多任务。Java 内置了这些集合接口和实现是个好事。让我们首先看看主要接口。您不需要理解编码示例的每一个细节；我们将在之后更详细地解释所有内容。

### 列表

最常见的数据结构之一是列表。**列表**是有序且带索引的集合，允许重复元素。当元素的顺序很重要，并且需要根据索引访问元素时，它们非常有用。

这里有一个列表的例子，我们在这个班级中存储了一系列学生姓名，其中姓名的顺序很重要。这是一个只包含 `String` 类型元素的列表。如您所见，`List` 是一个接口。当我们实例化它时，我们需要选择一个实现 `List` 的类。在这种情况下，我们选择了 `ArrayList`。这是一个非常常见的选项，但还有其他选项，例如 `LinkedList`。有一些重要的区别，但在这里我们不会深入探讨：

```java
List<String> studentNames = new ArrayList<>();studentNames.add("Sarah-Milou");
studentNames.add("Tjed");
studentNames.add("Fahya");
```

通过这样，我们已经看到 `List` 可以存储字符串，但集合可以存储任何类型的对象，包括自定义对象。假设我们有一个 `Person` 对象。这可能看起来是这样的：

```java
List<Person> personNames = new ArrayList<>();personNames.add(new Person("Sarah-Milou", 4));
personNames.add(new Person("Tjed", 6));
personNames.add(new Person("Fahya", 8));
```

为了简单起见，我们将在示例中主要使用 `String`，但请记住，这可以是任何对象（包括其他集合）。

同样存在无序集合，不允许重复元素。这些是 `Set` 类型的。让我们来看看它们。

### 集合

**集合**（通常）是无序集合，不允许重复元素。当您需要存储唯一元素但不需要关心它们的顺序时，它们非常有用。

假设我们需要一个数据结构来存储所有需要发送时事通讯的电子邮件地址。我们不希望有任何重复项存在，因为这会导致接收者收到重复的邮件：

```java
Set<String> emailAddresses = new HashSet<>();emailAddresses.add("sarahmilou@amsterdam.com");
emailAddresses.add("tjed@amsterdam.com");
emailAddresses.add("fahya@amsterdam.com");
```

您无需担心添加重复项，如果您尝试这样做，什么也不会发生。您将看到 `Set` 的不同实现，包括两种维护其元素特定顺序的类型。但让我们先看看另一种数据结构：映射。

### 映射

**映射**存储键值对，并基于键提供查找。当您需要将值与唯一键关联起来时，例如根据用户名存储用户信息时，它们非常有用：

```java
Map<String, String> userInfo = new HashMap<>();userInfo.put("Sarah-Milou", "Sarah-Milou Doyle");
userInfo.put("Tjed", "Tjed Quist");
userInfo.put("Fahya", "Fahya Osei");
```

如您所见，映射使用不同的方法。尽管 `Map` 是集合框架的一部分，但它有点特别。`Map` 是唯一一个没有扩展 `Collection` 接口的主要接口。`List`、`Set` 和 `Queue` 都扩展了。

有时，我们需要一个只允许访问集合开始和/或结束的有序集合。我们可以使用队列来完成这项任务。

### 队列和双端队列

**队列**允许您将元素添加到队列的起始位置，并访问末尾的元素。有一个特殊的队列允许在两端进行插入和删除。这被称为**双端队列**。双端队列代表双端队列。因此，队列遵循**先进先出**（**FIFO**）原则，而双端队列可以用作队列（FIFO）和栈，后者遵循**后进先出**（**LIFO**）原则。

它们对于需要按特定顺序处理元素的任务很有用，例如在实现任务调度器时。以下是一个打印作业队列的示例，其中任务按接收顺序进行处理：

```java
Queue<String> printQueue = new LinkedList<>();printQueue.add("Document1");
printQueue.add("Document2");
printQueue.add("Document3");
String nextJob = printQueue.poll(); // "Document1"
```

让我们更详细地看看这些接口，再次从`List`开始。

# 列表

因此，`List`接口是 Java 集合框架的一部分，用于表示元素的有序集合。`List`接口中的元素可以通过其位置（索引）访问，并且可以包含重复项。由于`List`是一个接口，因此不能被实例化。`List`接口的两种常用实现是`ArrayList`和`LinkedList`。由于这些是实现类，因此可以实例化。让我们来探讨它们是什么。

## 链接列表

`ArrayList`是`List`接口的可调整大小的数组实现。它提供了对元素的快速随机访问，并且对于读取密集型操作非常高效。随机访问意味着直接快速地使用其索引到达任何项目。

`ArrayList`在添加或删除元素时动态调整自身大小。添加和删除元素的速度相对较慢。`LinkedList`对此进行了优化。

## 链接列表

`LinkedList`是基于双链表数据结构的`List`接口实现。它不仅实现了`List`，还实现了`Queue`和`Deque`。它提供了在列表的开始和末尾快速插入和删除元素的功能，以及双向高效的遍历。然而，与`ArrayList`相比，在`LinkedList`中通过索引访问元素可能较慢，因为必须从列表的头部或尾部遍历元素。

即将提供的示例可以在`ArrayList`和`LinkedList`上以相同的方式进行。区别在于性能（在这些示例中的小数据量中，这种差异并不显著）。

## 探索列表的基本操作

我们可以向列表中添加、删除、更改和访问项目。让我们看看如何执行这些日常操作。列表还有很多其他有用的方法，但我们将坚持使用必备的方法，并从向列表中添加元素开始。

### 向列表中添加元素

我们可以使用 `add()` 方法向 `List` 接口中添加元素。`add()` 方法有两种形式：`add(E element)` 和 `add(int index, E element)`。第一种形式将元素添加到列表的末尾，而第二种形式将元素添加到指定的索引。这将使所有后续元素向上移动一个索引。这里的 `E` 是实际类型的占位符。如果它是一个 `String` 类型的列表，我们只能向列表中添加字符串。

让我们看看一个使用名字列表的简单例子：

```java
List<String> names = new ArrayList<>();names.add("Julie"); // Adds "Julie" at the end of the list
names.add(0, "Janice"); // Inserts "Janice" at index 0
```

首先，我们创建了一个 `ArrayList` 的实例。这是一个 `String` 类型的列表，正如我们在尖括号 (`<>`) 之间的 `String` 一词所看到的。然后我们继续将 `Julie` 添加到列表中。之后，我们指定位置。我们不是在 `Julie` 后面添加 `Janice`，而是在索引 `0` 处添加 `Janice`。这使得 `Julie` 从索引 `0` 变为索引 `1`。

在此之后，我们有一个包含两个 `String` 元素的列表。让我们看看我们如何访问这些元素。

### 从列表中获取元素

您可以使用 `get()` 方法从 `List` 接口中获取元素，该方法需要一个索引作为参数。我们将继续使用我们之前的例子。下面是如何做到这一点：

```java
String name = names.get(1);
```

这将获取索引 `1` 处的元素，即 `Julie`，并将其存储在一个名为 `name` 的变量中。我们还可以通过 `set()` 方法更改列表中的元素。

### 更改列表中的元素

我们可以使用 `set()` 方法更改 `List` 接口中的元素，该方法需要一个索引和一个新元素作为参数。在这里，我们将更改索引 `1` 处的元素：

ames.set(1, "Monica");

这样，我们就更新了 `Julie` 的值为 `Monica`。如果我们想的话，我们也可以从列表中移除元素。

### 从列表中移除元素

我们可以使用 `remove()` 方法来移除元素。`remove()` 方法有两种形式：`remove(int index)` 和 `remove(Object o)`。第一种形式移除特定位置的元素，而第二种形式移除具有特定值的元素：

```java
names.remove(1); // Removes the element at index 1names.remove("Janice"); // Removes the first occurrence
```

到这一点，列表再次为空，因为我们已经移除了两个元素。我们通过索引 `1` 移除了 `Monica`，通过查找具有该值的元素移除了 `Janice`。

### 遍历列表

遍历列表有不同的方法。我们将查看两种最常见的方法。

首先，我们可以使用一个普通的 `for` 循环来遍历一个列表。在这种情况下，我们正在遍历列表中的 `names`。假设我们刚才没有移除两个元素，列表中仍然包含 `Janice` 和 `Monica`：

```java
for (int i = 0; i < names.size(); i++) {    System.out.println(names.get(i));
}
```

输出将如下所示：

```java
JaniceMonica
```

我们也可以通过使用 `for-each` 循环达到相同的效果：

```java
for (String name : names) {    System.out.println(name);
}
```

正规 `for` 循环和 `for-each` 循环之间的区别在于，我们可以在常规 `for` 循环中访问索引。`for-each` 循环使得访问元素更容易，因为我们不需要确保我们保持在界限内，使用索引，并更新索引。

还有相当多的其他方法可用，但这些是您开始时最重要的方法。现在，让我们看一下`Set`接口。

注意

您可以在官方文档中找到有关所有集合的更多信息：[`docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/doc-files/coll-overview.html`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/doc-files/coll-overview.html)。

# `Set`

`Set`接口是 Java 集合框架的一部分，表示一个通常无序且唯一的元素集合。这意味着一个元素只能存在于集合中一次。`Set`接口的常用实现包括`HashSet`、`TreeSet`和`LinkedHashSet`。让我们快速看一下每个实现。

## `HashSet`

让我们先看看最流行的集合：`HashSet`。这是基于哈希表的`Set`接口的广泛实现。哈希表以键值对的形式存储数据，通过计算项的键哈希值来实现快速查找。它为基本的操作（如`add`、`remove`和`contains`——检查`Set`接口是否包含某个值）提供了常数时间性能。

*常数时间复杂度*意味着执行这些操作所需的时间不会随着集合中元素数量的增加而增加，前提是用于在桶之间分配元素哈希函数能够很好地完成其工作。我们将在本章末尾更详细地介绍哈希和桶分配，但哈希基本上是将某个值转换为另一个值的过程——例如，将字符串转换为数字。

请注意，基于哈希的数据结构，如`HashSet`，并不能保证存储在其中的元素具有任何特定的顺序。这是因为元素是根据它们的哈希值放入集合中的，而这些哈希值可能与人对顺序的理解（如升序或时间顺序）无关。

## `TreeSet`

`TreeSet`是基于树实现的`Set`接口。它根据元素的天然顺序或实例化时提供的自定义比较器来维护元素的排序顺序。`TreeSet`为常见的操作（如`add`、`remove`和`contains`）提供了对数时间性能。

*对数时间复杂度*意味着执行这些操作所需的时间随着输入大小的增加而对数增长，这使得`TreeSet`成为处理合理大数据集时的有效选择。

与不维护元素特定顺序的基于哈希的数据结构（如`HashSet`）相比，`TreeSets`在需要维护元素排序顺序的集合时是一个极佳的选择。这可以用于诸如维护有序列表中的唯一项目、快速查找集合中的最小或最大元素或对数据集执行范围查询等任务。

树的说明

计算机科学中的*树*并不是你在后院就能拥有的东西。在计算机科学中，树是一种表示不同节点之间关系的分层数据结构。每个节点都是一个数据点。第一个节点，称为根节点，没有父节点。每个其他节点（直接或间接地）沿着单一路径从根节点衍生出来。路径末端的节点，没有子节点，被称为*叶节点*。这种结构非常适合表示分层关系，因为每个节点都有一个父节点（除了根节点）和可能有很多子节点，就像自然树中的分支和叶子一样。

在树中，你可以把从根节点到任何节点的路径看作是一次旅行。路径中的每一步代表父节点和子节点之间的关系。树的*高度*是从根节点到叶节点的最长路径中的步数。节点的*深度*是从根节点到该节点的路径中的步数。与包含的节点数量相比高度较小的树通常在查找节点或添加和删除节点时效率较高。它们对于几个用例很有价值，例如在文件系统中组织文件或存储用于高效查找的排序数据，例如在`TreeSet`中。

## LinkedHashSet

`LinkedHashSet`是`Set`接口的一个实现，它按插入顺序维护元素，并由哈希表和双向链表的组合支持。`LinkedHashSet`为基本操作提供常数时间性能，同时保留插入顺序。

当插入顺序很重要且元素不需要排序时，你通常会选择这种实现方式。而且，由于它是一个`Set`，当然，元素必须是唯一的（否则`List`可能更合理）。`LinkedHashSet`的一个用例示例是维护一个按访问顺序排列的唯一项目列表，例如网页浏览历史或按添加顺序排列的唯一歌曲播放列表。另一个示例是在应用程序中跟踪事件或用户操作，同时确保每个事件或操作只处理一次。

要完成所有这些，我们确实需要能够执行一些基本操作。所以，让我们看看如何做到这一点。

## 在集合上执行基本操作

`Set`接口上的操作与`List`上的操作非常相似。当然，我们在`Set`的方法上不使用索引。我们将从学习如何向集合中添加元素开始。

### 向集合中添加元素

就像我们在`List`中做的那样，我们可以使用`add()`方法向`Set`接口添加元素。以下是这样做的方法：

```java
Set<String> names = new HashSet<>();names.add("Elizabeth");
names.add("Janie");
```

集合不能包含重复的值。添加相同的值两次不会产生错误，也不会再次添加该值。

同样容易，我们可以创建一个`LinkedHashSet`类，如下所示：

```java
Set<String> names = new LinkedHashSet<>();
```

我们还可以创建一个`TreeSet`类：

```java
Set<String> names = new TreeSet<>();
```

这些集合上的操作将是相同的。

### 修改集合中的元素

我们不能直接在 `Set` 中更改元素。要修改元素，我们必须删除旧元素并添加新元素。因此，让我们学习如何删除元素。

### 从集合中删除元素

我们可以使用 `remove()` 方法从 `Set` 接口中删除元素。我们不能像对 `List` 那样按索引删除，因为元素没有索引：

```java
names.remove("Janie");
```

之后，集合将只剩下一个值，即 `Elizabeth`。由于集合没有索引，访问元素的方式也略有不同。我们可以通过迭代来访问元素。

### 遍历集合

我们可以使用 `for-each` 循环遍历集合。我们不能使用常规的 `for` 循环，因为我们没有索引。

这里有一个例子：

```java
for (String name : names) {    System.out.println(name);
}
```

在删除之后，我们的 `Set` 接口只剩下一个名称了。因此，这个 `for-each` 循环将输出以下内容：

```java
Elizabeth
```

对于 `Set` 来说，这就结束了。现在，让我们探索 `Map` 数据结构。

# 映射

集合框架的另一个成员是 `Map` 接口。此接口表示一组键值对。键是唯一的，而值可以重复。这就是为什么我们使用键来添加和访问映射中的键值对。我们将讨论的 `Map` 接口的常用实现是 `HashMap` 和 `TreeMap`。

## HashMap

最受欢迎的可能就是 `HashMap`。这是一个基于哈希表的广泛使用的 `Map` 接口实现。就像 `HashSet` 一样，它为基本操作提供常数时间性能。然而，它不保证键的任何特定顺序。`HashMap` 适用于需要快速查找和修改的情况，例如存储配置设置或统计文本中的单词出现次数。当顺序很重要时，我们可以使用 `TreeMap`。

## TreeMap

`TreeMap` 是基于树实现的 `Map` 接口。它根据其自然排序或实例化时提供的自定义比较器对键值对进行排序。我们很快就会看到自定义比较器，但这基本上是指定它需要按何种顺序排序的一种方式。

`TreeMap` 为常见操作如从映射中获取元素和向映射中添加元素提供对数时间性能。`TreeMap` 适用于需要维护键值对排序集合的场景，例如管理排行榜或跟踪基于时间的事件。

## LinkedHashMap

`LinkedHashMap` 是 `Map` 接口的另一种实现。它通过提供类似于 `HashMap` 的常数时间性能来结合 `HashMap` 和 `TreeMap` 的优点，同时保持键值对的插入顺序。这个顺序是键被添加到映射中的顺序。

`LinkedHashMap`本质上是一个带有附加链接列表的`HashMap`实现，该列表连接所有条目，这使得它可以记住插入顺序。这在数据序列很重要的情况下特别有用，例如缓存操作或维护用户活动记录。

它的使用与其他两个实现非常相似。我们不会在这里展示所有实现，因为每种实现的基本操作都是相同的。唯一的区别是，当你遍历它们时，它们有一个特定的顺序，但遍历的方式是相同的。

# 映射的基本操作

`Map`与其他集合有很大不同。让我们学习如何执行`Map`的基本操作。

### 向映射中添加元素

`Map`没有`add()`方法。我们可以使用`put()`方法向`Map`接口添加元素：

```java
Map<String, Integer> gfNrMap = new HashMap<>();gfNrMap.put("Ross", 12);
gfNrMap.put("Chandler", 8);
```

这向`Map`添加了两个键值对。让我们看看我们如何再次获取值。

### 从映射中获取元素

我们可以使用`get()`方法从`Map`接口获取元素。这就是我们如何获取与`Ross`键关联的`Integer`值的方式：

```java
int rossNrOfGfs = gfNrMap.get("Ross");
```

我们也可以使用键来修改映射的值。

### 更改映射的元素

我们可以使用带有现有键的`put()`方法在`Map`接口中更改元素：

```java
gfNrMap.put("Chandler", 9);
```

以下代码将`Chandler`键的值从`8`更改为`9`。我们不能更改键。如果我们需要这样做，我们需要删除键值对并添加一个新的。

### 从映射中删除元素

键也用于从映射中删除元素。我们可以使用`remove()`方法来完成此操作。

```java
gfNrMap.remove("Ross");
```

到目前为止，我们的映射只包含一个键值对。我们也可以遍历映射。这与我们对`List`和`Set`所做的不太一样。

### 遍历映射

我们可以使用 for-each 循环遍历键值对、值和键。我们需要在我们的映射对象上调用不同的方法来实现这一点。我们可以使用`entrySet()`、`keySet()`和`values()`方法来完成此操作。

假设我们映射中仍然有两个键值对，以`Ross`和`Chandler`作为键。以下代码片段使用`entrySet()`方法遍历键值对：

```java
for (Map.Entry<String, Integer> entry : gfNrMap.entrySet()) {    System.out.println(entry.getKey() + ": " +
      entry.getValue());
}
```

`entrySet()`提供了一组`Map.Entry`对象。在这个对象上，我们可以使用`getKey()`和`getValue()`方法分别获取键和值。这将输出以下内容：

```java
Ross: 12Chandler: 9
```

我们也可以遍历键：

```java
for (String key : gfNrMap.keySet()) {    System.out.println(key + ": " + gfNrMap.get(key));
}
```

这将输出以下内容：

```java
Ross: 12Chandler: 9
```

你可能会惊讶，这与前面的代码片段相同，并且包含值，但这是因为我们正在使用键来获取值。当我们遍历值时，这是不可能的。以下是如何做到这一点的方法：

```java
for (Integer value : gfNrMap.values()) {    System.out.println(value);
}
```

这将输出以下内容：

```java
129
```

现在，我们只能看到值，因为我们正在遍历这些值。接下来，让我们看看最后一个主要接口：`Queue`。

# 队列

最后是 `Queue` 接口。它是 Java 集合框架的一部分，允许 FIFO 数据存储。队列的头部是最老的元素，而尾部是最新的元素。队列对于按接收顺序处理任务非常有用。还有一个名为 `Deque` 的子接口，它是一种特殊的队列，允许从队列的头部和尾部获取元素。这就是为什么它也可以用于 LIFO 系统。

我们将只简要地处理不同类型的队列，因为这是在野外最不常用的集合。

## 队列实现

`Queue` 接口扩展了 `Collection` 接口。有几种实现方式，其中一些最常见的是 `PriorityQueue`、`LinkedList` 和 `ArrayDeque`。扩展了 `Queue` 接口的 `Deque` 接口增加了对双端队列的支持，允许从队列的两端插入和移除元素。`LinkedList` 和 `ArrayDeque` 是 `Deque` 的实现。

## 队列接口的基本操作

`Queue` 接口的基本操作有点特殊，因为元素只能从队列的末端访问。

### 向队列中添加元素

我们可以使用 `add()` 或 `offer()` 方法向队列中添加元素。如果队列达到最大容量，当 `add()` 方法无法向队列中添加元素时，会抛出异常。如果 `offer()` 方法无法将元素添加到队列中，它会返回 `false`。从动词来看，这似乎是合理的；*offer* 没有义务，当队列满时，队列可以拒绝这个请求，因此当队列满时不会抛出异常。如果无法将其附加到队列中，它只会返回 `false`。而 *add* 真正意图添加，如果它不起作用，则会抛出异常。

这是如何使用 `LinkedList` 的示例：

```java
Queue<String> queue = new LinkedList<>();queue.add("Task 1");
queue.offer("Task 2");
```

对于 `Deque` 类型的对象，可以使用不同的方法在队列的头部添加元素。`LinkedList` 正好是 `Deque` 类型。`add` 和 `offer` 方法将元素添加到队列的末尾，`Deque` 类型的特殊方法 `addLast()` 和 `offerLast()` 也是如此：

```java
Deque<String> queue = new LinkedList<>();queue.addLast("Task 1"); // or add
queue.offer("Task 2"); // or offerLast
```

这是如何向队列的头部添加元素的方法：

```java
queue.addFirst("Task 3");queue.offerFirst("Task 4");
```

队列中元素的顺序现在是（从头部到尾部）*任务 4，任务 3，任务 1，* *任务 2*。

### 从队列中获取元素

我们可以使用 `peek()` 或 `element()` 方法从 `Queue` 接口获取队列头部的元素。它们只是返回值，而不会从队列中移除它。

这是使用 `peek()` 方法获取队列头部的方法：

```java
String head = queue.peek();
```

`head` 的值变为 *任务 4*。当 `element()` 方法无法返回值时，会抛出异常，而 `peek()` 方法则不会。当队列空时，`peek()` 方法返回 `null`。

对于`Deque`，我们可以在头部和尾部获取元素。对于头部，我们可以使用`getFirst()`和`peekFirst()`。对于尾部，我们可以使用`getLast()`和`peekLast()`。请注意，`getFirst()`是`Deque`的`Queue`的`element()`的等价物，尽管这些名称在相当大的程度上有所不同。

你可能会想知道，为什么对于所有这些我们都有两个做同样事情的方法。它们并不完全一样，有一个重要的区别。`getFirst()`、`getLast()`和`element()`方法试图检索队列的端点，但如果队列是空的，它会抛出`NoSuchElementException`。相比之下，`peek()`、`peekFirst()`和`peekLast()`方法也检索队列的端点，但如果队列是空的，它们会返回`null`，因此它们不会抛出异常。

### 更改队列中的元素

我们不能直接在`Queue`接口中更改元素。要修改元素，我们必须删除旧元素并添加新元素。所以，让我们看看如何删除元素。

### 从队列中删除元素

我们可以使用`remove()`或`poll()`方法从队列中删除元素。这些方法做两件事：

1.  返回队列的头部。

1.  移除队列的头部。

这里有一个例子：

```java
String removedElement = queue.poll();
```

这将把`Task 4`存储在`removedElement`中。此时，队列中的值将是`Task 3`、`Task 1`、`Task 2`。

这可能不会让你感到惊讶，但对于`Deque`，我们可以从两端删除元素。对于头部，我们使用`removeFirst()`和`pollFirst()`。对于尾部，我们可以使用`removeLast()`和`pollLast()`。

再次强调，区别在于它们如何处理`null`值：

+   `remove()`、`removeFirst()`和`removeLast()`如果队列是空的，会抛出`NoSuchElementException`。

+   `poll()`、`pollFirst()`和`pollLast()`在抛出异常的情况下返回`null`，表示队列是空的。

现在我们知道了如何删除元素，让我们学习如何遍历`Queue`接口。

### 遍历队列或双端队列

我们可以使用 for-each 循环遍历队列或双端队列。这不会从队列中移除“正在遍历”的元素：

```java
for (String element : queue) {    System.out.println(element);
}
```

这将输出以下内容：

```java
Task 3Task 1
Task 2
```

它没有打印出`Task 4`的原因是我们之前已经删除了它。

我们现在已经涵盖了四个主要接口的基础知识以及一些最常见的实现。我们可以对集合做更多的事情，比如排序。让我们看看如何做到这一点。

# 排序集合

到目前为止，我们已经学习了如何创建集合以及如何在它们上执行基本操作。它们有很多有用的内置方法，其中之一帮助我们排序集合。我们之所以关注这个方法，是因为它不像其他一些方法那么直接。

一些类型具有自然顺序，例如数字。它们可以很容易地从大到小排序。字符串也是如此——我们可以按字母顺序排序。但如何对一个包含自定义`Task`类型对象的集合进行排序呢？

跟着我——不久，你将能够在使用集合中内置的`sort`方法时同时进行自然排序和自定义排序。

## 自然排序

当我们谈论自然排序时，我们指的是特定数据类型的默认排序顺序。例如，数字按升序排序，而字符串按字典顺序排序。但是，如果没有我们告诉 Java 我们想要这样，Java 就不会知道这一点。这就是为什么 Java 的内置类，如`Integer`和`String`，实现了`Comparable`接口。这就是告诉 Java 自然顺序是什么。与排序相关的两个接口是`Comparable`和`Comparator`。我们将在下一章中介绍这些。

## `Comparable`和`Comparator`接口

当一个类实现`Comparable`接口时，我们需要实现`compareTo()`方法。以下是一个类如何实现该接口的示例：

```java
public class Person implements Comparable<Person> {...}
```

代码被省略了，但正如你所见，它实现了接口。现在它需要重写`compareTo`方法。

此方法定义了如何对相同类型的两个对象进行排序。`compareTo()`方法接受另一个相同类型的对象作为参数，并根据两个对象比较的结果返回一个负数、零或正整数。

这些是结果值的含义：

+   如果两个对象相等，则返回 0

+   如果对象大于传入的对象，则返回正值

+   如果调用方法的对象小于传入的对象，则返回负值

`Comparator`接口做的是类似的事情，但不建议由类实现。此接口用于动态创建自定义`Comparator`，通常使用 Lambda 表达式实现。我们还没有看到 Lambda 表达式，但将在下一章中介绍。`Comparator`可以传递给`sort`方法，以告诉`sort`方法如何排序项目。

`Comparator`不是用于自然排序顺序，而是用于“一次性”排序顺序。它包含一个方法，`compare()`。此方法接受两个对象作为参数，并根据比较结果返回一个负数、零或正整数。以下是比较结果值的含义：

+   如果两个对象相等，则返回 0。

+   如果第一个对象大于第二个对象（因此它们顺序错误），则返回正值。

+   如果第一个对象小于第二个对象（因此它们顺序正确），则返回负值。

好了，别再说了。让我们看看`Comparable`和`Comparator`的一些实现。

### 实现`compareTo()`

因此，当我们想要对自定义类型进行排序时，大约有两种选择：

+   通过使它们实现`Comparable`来给它们一个自然顺序。

+   实现`Comparator`并将其传递给`sort`方法。

让我们从第一个开始。我们将给我们的`Person`类一个自然顺序。为了为自定义类实现自然排序，我们需要实现`Comparable`接口和`compareTo()`方法。下面是如何做到这一点：

```java
public class Person implements Comparable<Person> {    int age; // not private to keep the example short
    String name;
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    @Override
    public int compareTo(Person other) {
        return Integer.compare(this.age, other.age);
    }
}
```

在这里，`Person`类通过实现`Comparable`接口被赋予了一个自然顺序。

`Person`类现在实现了`Comparable<Person>`。这意味着`Person`对象现在可以根据自然顺序相互比较，这个顺序由`compareTo()`方法确定。此方法接受一个输入参数。它总是将此参数与`compareTo()`被调用的实例进行比较。如果对象相等，则应返回`0`；如果调用该方法的对象大于输入参数，则返回正值；如果输入参数大于，则返回负值。

`Person`类有两个属性：`age`（一个整数）和`name`（一个字符串）。构造函数使用给定的值初始化这些属性。`compareTo()`方法被定义为根据年龄比较`Person`对象，但我们也可以选择名称长度来举例。在这个`compareTo()`方法中，我们使用`Integer.compare()`方法进行比较。它接受两个整数作为参数，并返回以下内容：

+   如果两个整数相等，则返回`0`。

+   如果第一个整数大于第二个整数，则返回正值。

+   如果第一个整数小于第二个整数，则返回负值。

在`compareTo()`方法的上下文中，这意味着以下内容：

+   如果两个`Person`对象年龄相同，该方法将返回`0`。

+   如果当前`Person`对象的年龄大于另一个对象的年龄，则方法将返回正值。

+   如果当前`Person`对象的年龄小于另一个对象的年龄，则方法将返回负值。

这些返回值决定了`Person`对象在排序时的自然顺序。在这种情况下，对象将按年龄排序。接下来，让我们看看如何实现这一点：

```java
List<Person> personList = new ArrayList<>();personList.add(new Person("Huub", 1));
personList.add(new Person("Joep", 4));
personList.add(new Person("Anne", 3));
Collections.sort(personList);
```

在排序之前，元素按照它们被添加的顺序排列。在排序之后，它们按年龄从低到高排序，因此我们得到`Huub`、`Anne`和`Joep`。

但再次强调，既然是我们写的，我们可以选择任何东西。我们选择的内容将决定自然顺序。自然顺序，例如，是按字母顺序排序字符串 A-Z 和数字 0-9。你的自定义类的自然顺序取决于你。它取决于你如何实现`compareTo()`方法。

有时候，我们可能需要比`compareTo()`方法中指定的顺序不同。例如，按单词长度对字符串进行排序。幸运的是，我们也可以创建一个与类无关的顺序。接下来，让我们看看如何进行自定义排序。

### 实现 compare()

有几种方法可以使用`Comparator`接口实现自定义排序：

+   创建一个单独的类（不典型）

+   使用匿名内部类（更好）

+   使用 Lambda 表达式实现（最常见）

例如，要按名称对`Person`对象列表进行排序，我们可以创建这个匿名类：

```java
Comparator<Person> nameComparator = new  Comparator<Person>() {
    @Override
    public int compare(Person p1, Person p2) {
        return p1.getName().compareTo(p2.getName());
    }
};
```

在这里，我们创建了一个新的`Comparator`对象`nameComparator`，该对象实现了`Comparator`接口。这个自定义比较器将用于根据姓名比较`Person`对象。`compare()`方法在匿名内部类中实现。在`compare()`方法内部，我们使用`String`类的`compareTo()`方法对两个`Person`对象的姓名进行字典比较。

`Comparator`接口中的`compare()`方法遵循与`Comparable`接口中的`compareTo()`方法相同的返回值规则：

+   如果被比较的两个对象相等，该方法将返回`0`。

+   如果第一个对象大于第二个对象，该方法将返回一个正值。

+   如果第一个对象小于第二个对象，该方法将返回一个负值。

要使用自定义比较器对`Person`对象列表进行排序，我们可以将`nameComparator`对象作为参数传递给`Collections.sort()`方法，如下所示：

```java
List<Person> personList = new ArrayList<>();personList.add(new Person("Huub", 1));
personList.add(new Person("Joep", 4));
personList.add(new Person("Anne", 3));
Collections.sort(personList, nameComparator);
```

在这个例子中，`personList`将根据`Person`对象的名称按字母顺序排序，正如`nameComparator`所指定的。如果我们没有指定`nameComparator`，它将使用自然顺序并按年龄排序。在排序之前，元素按照添加的顺序排列。排序后，它们按名称排序，A-Z，所以我们得到`Anne`、`Huub`和`Joep`。

使用 Lambda 表达式实现 Comparator

更常见的是使用 Lambda 表达式来实现`Comparator`接口。这样，我们就有了一个更短的语法来创建比较器，而不需要匿名内部类。你现在可能不需要理解这一点，但这里有一个使用 Lambda 表达式创建按姓名排序的`Person`对象比较器的例子：

`Comparator<Person> nameComparatorLambda = (p1,` `p2) ->`

`p1.getName().compareTo(p2.getName());`

这是一样的。我们可以将其作为参数传递给`Collections.sort()`方法：

`Collections.sort(personList, nameComparatorLambda);`

由于我们现在有了自定义比较器，我们可以创建尽可能多的比较器。这里有一个使用 Lambda 表达式按姓名长度排序`Person`对象的另一个例子：

`Comparator<Person> nameLengthComparator = (p1,` `p2) ->`

`Integer.compare(p1.getName().length(),`

`p2.getName().length());`

`Collections.sort(personList, nameLengthComparator);`

在这里，`nameLengthComparator`根据姓名长度比较`Person`对象。`personList`将按姓名长度的升序排序。我们的名字长度都是四个，因此它们将保持添加时的顺序。

使用`Comparator`而不是由`Comparable`接口定义的自然顺序的优势在于，你可以在不修改类本身的情况下为同一类定义多个自定义排序。此外，我们可以通过向`Collections.sort()`方法提供不同的`Comparator`对象来轻松地在运行时更改排序标准。

我们选择哪个选项取决于我们的需求。如果我们想给我们的对象一个自然顺序，我们必须实现 `Comparable` 接口。如果我们无法直接访问类，或者我们想要指定一个不是自然顺序的顺序，我们可以使用 `Comparator`。

我们也可以在创建 `TreeSet` 和 `TreeMap` 时使用比较器。这将确定这些集合中的值将如何排序。

### TreeSets 和 TreeMaps

`TreeSet` 和 `TreeMap` 是使用其元素的自然顺序或自定义比较器进行排序的有序集合。这就是为什么我们无法为没有自然顺序的对象（它们没有实现 `Comparable` 接口）创建 `TreeSet` 或 `TreeMap`，除非在创建 `TreeSet` 或 `TreeMap` 时提供一个自定义比较器。让我们看看如何为每个实现这个操作。

### `TreeSet` 中元素的顺序

作为快速提醒，`TreeSet` 是一个 `Set` 实现类，它以排序顺序存储元素。这就是为什么 `TreeSet` 中的元素必须实现 `Comparable` 接口，或者必须在 `TreeSet` 的构造过程中传递一个自定义比较器。

这是一个使用自然顺序创建 `TreeSet` 类的 `Person` 对象的例子：

```java
TreeSet<Person> personTreeSet = new TreeSet<>();personTreeSet.add(new Person("Huub", 1));
personTreeSet.add(new Person("Joep", 4));
personTreeSet.add(new Person("Anne", 3));
```

在这个例子中，`Person` 类实现了 `Comparable` 接口，所以 `TreeSet` 将使用 `Person` 类中定义的 `compareTo()` 方法所定义的自然顺序（这是按年龄排序的）。

如果你想要创建一个带有自定义比较器的 `TreeSet` 类，你可以将比较器作为参数传递给 `TreeSet` 构造函数，如下所示：

```java
Comparator<Person> nameComparator = (p1, p2) ->  p1.getName().compareTo(p2.getName());
TreeSet<Person> personTreeSetByName = new
  TreeSet<>(nameComparator);
personTreeSetByName.add(new Person("Huub", 1));
personTreeSetByName.add(new Person("Joep", 4));
personTreeSetByName.add(new Person("Anne", 3));
```

在这个例子中，`TreeSet` 将根据 `nameComparator` 指定的名称对 `Person` 对象进行排序。我们可以对 `TreeMap` 做类似的事情。

### `TreeMap` 中元素的顺序

如果你忘记了，`TreeMap` 是一个 `Map` 实现类，它根据键的排序顺序存储键值对。这就是为什么 `TreeMap` 中的键必须实现 `Comparable` 接口，或者我们在创建 `TreeMap` 时应该提供一个自定义比较器。

让我们从使用自然顺序作为键的 `Person` 对象和它们的年龄作为值的 `TreeMap` 类开始：

```java
TreeMap<Person, Integer> personTreeMap = new TreeMap<>();personTreeMap.put(new Person("Huub", 1), 1);
personTreeMap.put(new Person("Joep", 4), 4);
personTreeMap.put(new Person("Anne", 3), 3);
```

在这个例子中，`Person` 类实现了 `Comparable` 接口，所以 `TreeMap` 将使用 `Person` 类中定义的 `compareTo()` 方法所定义的自然顺序。

如果你想要创建一个带有自定义比较器的 `TreeMap` 类，你可以将比较器作为参数传递给 `TreeMap` 构造函数，如下所示：

```java
Comparator<Person> nameComparator = (p1, p2) ->  p1.getName().compareTo(p2.getName());
TreeMap<Person, Integer> personTreeMapByName = new
  TreeMap<>(nameComparator);
personTreeMapByName.put(new Person("Huub", 1), 1);
personTreeMapByName.put(new Person("Joep", 4), 4);
personTreeMapByName.put(new Person("Anne", 3), 3);
```

现在，这个 `TreeMap` 将根据 `nameComparator` 指定的名称对 `Person` 对象进行排序。

因此，`TreeSet` 和 `TreeMap` 是使用其元素的自然顺序或自定义比较器来排序其内容的有序集合。

通过使用 `Comparable` 接口和自定义比较器，你可以为你的自定义类定义多个排序方式，并轻松控制集合的排序行为。

# 与泛型一起工作

我们在本章中一直在使用泛型。泛型是灵活的，用于（包括）集合。我们通过在尖括号之间指定类型来将这些值传递给这些集合。我们可以创建一个具有类型参数的集合，如下所示：

```java
List<String> names = new ArrayList<>();
```

这是因为 `List` 接口和 `ArrayList` 类都是使用类型参数（泛型）创建的。这使得类更加灵活，同时仍然确保类型安全。让我们看看在泛型之前是如何做到这一点的，以了解为什么它们如此出色。

## 泛型之前的生活 – 对象

在我们没有泛型之前，所有集合都会有对象。你必须手动检查列表中的项是否是你希望的类型。如果是，你必须将其转换为这个类型才能使用，就像这样：

```java
List = new ArrayList();list.add("Hello");
list.add("World");
list.add(123); // Integer inserted in a List of strings.
               //  Allowed, but not logical.
for (int i = 0; i < list.size(); i++) {
    Object item = list.get(i);
    if (item instanceof String) {
        String strItem = (String) item; // Type casting
                                        //  required
        System.out.println(strItem);
    } else {
        System.out.println("Item is not a String");
    }
}
```

在前面的代码中，我们创建了一个没有指定任何类型的列表。这创建了一个 `Object` 类型的列表。你可能还记得，所有 Java 对象都是 `Object` 类型。然后，我们向其中添加了两个字符串和一个整数。这在技术上是被允许的，因为列表接受任何类型的对象，但它可能导致你的代码中出现逻辑错误。

后来，当我们遍历列表时，在安全地将每个项目转换为字符串 `(String) item` 之前，我们必须使用 `instanceof` 手动检查每个项目的类型。如果我们尝试将错误类型的项转换为字符串，代码将在运行时抛出 `ClassCastException` 错误。这可能会很耗时且容易出错，这也是泛型被引入的主要原因之一。

让我们更详细地看看泛型，并看看它们在集合使用场景之外的应用。我们将学习如何创建一个泛型类，以及为什么我们会这样做。

## 泛型的用例

让我们先创建两种类型，我们将把它们放入一个包类中。我们首先不使用泛型来做这件事。

这里有一个名为 `Laptop` 的公共 `class`：

```java
public class Laptop {    private String brand;
    private String model;
    public Laptop(String brand, String model) {
        this.brand = brand;
        this.model = model;
    }
    // Getters and setters omitted
}
```

这里还有一个名为 `Book` 的公共 `class`：

```java
public class Book {    private String title;
    private String author;
    public Book(String title, String author) {
        this.title = title;
        this.author = author;
    }
    // Getters and setters omitted
}
```

书和笔记本电脑是典型的可以存放在包里的东西。让我们编写 Java 代码来完成这个任务。在不使用泛型的情况下，我们需要两个类。第一个将是为 `Laptop`：

```java
public class LaptopBag {    private Laptop;
    public LaptopBag(Laptop laptop) {
        this.laptop = laptop;
    }
    public Laptop getLaptop() {
        return laptop;
    }
    public void setLaptop(Laptop laptop) {
        this.laptop = laptop;
    }
}
```

第二个将是为 `Book`：

```java
public class BookBag {    private Book;
    public BookBag(Book book) {
        this.book = book;
    }
    public Book getBook() {
        return book;
    }
    public void setBook(Book book) {
        this.book = book;
    }
}
```

现在，我们有两个自定义类 `Laptop` 和 `Book`，以及两个包类 `LaptopBag` 和 `BookBag`，每个包类都包含特定类型的项。然而，在 `LaptopBag` 和 `BookBag` 类中有很多重复的代码。我们可以通过，而不是让 `Bag` 对一个类型特定化，允许它持有 `Object` 类型，来解决这个问题，如下所示：

```java
public class ObjectBag {    private Object;
    public ObjectBag(Object object) {
        this.object = object;
    }
    public Object getObject() {
        return object;
    }
    public void setObject(Object object) {
        this.object = object;
    }
}
```

这个类允许我们添加 `Laptop`、`Book` 或 `Person` 类。几乎可以是任何东西——它并不关心。但这也带来了一些缺点。由于 `ObjectBag` 类可以存储任何类型的对象，因此在编译时无法确保类型安全。这可能会导致运行时异常，例如 `ClassCastException`，如果我们不小心在代码中混合了不同类型的对象。

非常相关的是，当我们从 `ObjectBag` 中检索对象时，我们需要进行的类型转换。为了访问所有方法和字段，我们需要显式地将它转换回其原始类型。这增加了代码的冗余，并增加了出现 `ClassCastException` 错误的可能性。

幸运的是，泛型出现了！泛型提供了一种创建灵活且类型安全的类的方法，可以处理不同类型，而不具有使用 `Object` 类型相关的缺点。那么，让我们看看我们如何使用泛型重写 `ObjectBag` 类。

## 泛型语法

泛型通过在尖括号内指定类型参数来使用，例如 `<T>`，其中 `T` 代表一个类型。以下是一个使用单个 `Bag` 类的泛型解决方案：

```java
public class Bag<T> {    private T content;
    public Bag(T content) {
        this.content = content;
    }
    public T getContent() {
        return content;
    }
    public void setContent(T content) {
        this.content = content;
    }
}
```

通过使用泛型类型参数 `<T>`，我们现在可以创建一个更灵活的 `Bag` 类，它可以包含任何类型的项，例如 `Laptop` 或 `Book`。同时，我们可以确保类型安全并避免显式转换的需要。以下是使用 `Bag` 类的方法：

```java
Bag<Laptop> laptopBag = new Bag<>(new Laptop("Dell", "XPS  15"));
Bag<Book> bookBag = new Bag<>(new Book("Why Java is fun",
  "Maaike and Seán"));
```

总结来说，泛型在创建可重用类时增加了灵活性，同时保持了类型安全。然而，有时我们可能希望限制可以与泛型类一起使用的类型。这就是有界泛型发挥作用的地方。让我们看看。

## 有界泛型

没有有界泛型，我们可能会遇到需要在泛型类中调用特定类型或其子类的方法的情况。我们不能直接这样做，因为泛型类对其处理的类型的特定方法一无所知。以下是一个简短的例子来说明有界泛型的必要性。

假设我们有一个名为 `Measurable` 的接口：

```java
public interface Measurable {    double getMeasurement();
}
```

我们希望有一个类似于 `Bag` 的类，但只接受实现了 `Measurable` 接口的泛型。这就是为什么我们需要创建一个只能包含实现了 `Measurable` 接口对象的泛型 `MeasurementBag` 类。我们可以使用有界泛型来实现这一点：

```java
public class MeasurementBag<T extends Measurable> {    private T content;
    public MeasurementBag(T content) {
        this.content = content;
    }
    public T getContent() {
        return content;
    }
    public void setContent(T content) {
        this.content = content;
    }
    public double getContentMeasurement() {
        return content.getMeasurement();
    }
}
```

通过使用 `<T extends Measurable>`，我们指定泛型类型 `T` 必须是一个实现了 `Measurable` 接口类的实例。这确保了只有实现了 `Measurable` 接口类型的对象才能与 `MeasurementBag` 类一起使用。这就是为什么我们可以在 `MeasurementBag` 类中安全地调用 `getMeasurement()` 方法——因为我们知道 `T` 保证实现了 `Measurable` 接口。

因此，这些有界泛型允许我们限制在泛型类中使用的类型，并确保它们共享一组公共方法。这就是为什么在泛型类中调用这些方法是安全的。这听起来像集合所做的那样吗？例如，当我们只传递一个参数（集合）时，`Collections.sort()` 需要一个实现了 `Comparable` 接口的对象集合。泛型和有界类型参数在 Java 自身代码中实际上非常常见。

我们现在已经看到了限定泛型，它为泛型类型指定了一个上限（一个超类或接口）。这确保了只能使用该类型或其子类的对象与泛型类一起使用。也存在下限，但这里不涉及。你可能会在 Java 源代码中遇到这些，但你自己实际使用这些的可能性不大。

让我们深入探讨另一个在使用自定义对象与`HashMap`和`HashSet`一起使用时很重要的概念。

# 哈希和重写`hashCode()`

哈希是 Java 中的一个重要概念。它用于在诸如`HashMaps`和`HashSets`之类的各种数据结构中高效地存储和检索数据。它也是一个非常有趣的话题。即使不理解这个功能，你也能走得很远，但某个时候，你可能会对你的`HashMap`类的糟糕性能感到好奇。而要理解发生了什么，没有理解哈希是不可能的。所以，让我们讨论哈希的基本概念，`hashCode()`方法在集合中的作用，以及自定义类中重写`hashCode()`方法的最佳实践。

## 理解基本的哈希概念

哈希是一种将数据转换成称为哈希码的代码片段的方法。想象一下，将一大堆书分配一个唯一的数字。一个好的哈希函数应该给不同的书分配不同的数字，并均匀分布。这使得查找和组织书籍变得容易。Java 中的所有对象都有一个`hashCode()`方法。

## `hashCode()`及其在集合中的作用

`Object`类定义了`hashCode()`方法。由于所有类都从`Object`（间接地）继承，所有对象都有`hashCode()`方法。此方法返回一个整数值。两个相同的对象应该具有相同的哈希码。

当你在`HashMap`或`HashSet`类中使用一个对象时，它的`hashCode()`用于决定其在数据结构中的位置。当我们创建自定义类时，我们有时需要重写`hashCode()`。

## 重写`hashCode()`和最佳实践

当我们创建一个自定义类并计划将其用作`HashMap`类中的键或`HashSet`类中的元素时，我们需要重写`hashCode()`方法。这确保了我们的类有一个一致且高效的哈希函数。

这里有一些重写`hashCode()`的最佳实践：

+   包含在`equals()`方法中使用到的所有字段。这样，相等的对象将具有相同的哈希码。

+   使用一个简单的算法来组合各个字段的哈希码，例如乘以一个素数并加上字段的哈希码。

这里是一个在我们的`Person`类中实现`hashCode()`的例子：

```java
public class Person {    private String name;
    private int age;
    // Constructor, getters, and setters
    @Override
    public int hashCode() {
        int result = 17;
        result = 31 * result + (name == null ? 0 :
          name.hashCode());
        result = 31 * result + age;
        return result;
    }
}
```

如您所见，已经添加了`hashCode()`方法。

更详细地解释`hashCode()`

数字`17`和`31`被用作`Person`类哈希码计算的一部分。这两个都是素数，在哈希码计算中使用素数有助于产生更好的哈希码分布，并减少哈希码冲突的可能性。`17`用作结果变量的初始值。这是一个任意的素数，有助于确保哈希码计算从非零值开始。

通过这样做，它降低了生成不同对象相似哈希码的可能性，这反过来又有助于最小化冲突。`31`在哈希码计算中用作乘数。在添加下一个字段的哈希码之前，将当前结果乘以一个素数（在这种情况下是`31`）有助于更有效地混合各个字段的哈希码。这导致哈希码在可能范围内的更好分布。`31`通常被选择，因为它可以使用位运算（即`x * 31`与`(x << 5) - x)`相同）有效地计算。

## 在自定义泛型类型中使用 hashCode()

在创建自定义泛型类时，我们可能需要使用存储对象的`hashCode()`方法。为此，我们可以在对象上简单地调用`hashCode()`方法或使用`Objects.hashCode()`实用方法，该方法可以优雅地处理空值：

```java
public class Bag<T> {    private T content;
    // Constructor, getters, and setters
    @Override
    public int hashCode() {
        return Objects.hashCode(content);
    }
}
```

在使用 Java 集合工作时，理解哈希和`hashCode()`方法很重要，尤其是在使用自定义类与哈希集合结合时。如果我们遵循重写`hashCode()`和使用自定义泛型类型的最佳实践，我们可以在添加和访问集合中的元素时实现更好的性能。

# 练习

你可能没有直接注意到，但我们一直在期待这个！我们终于可以将集合和泛型添加到我们应用程序的应用中。生活将变得更加容易。让我们看看一些练习：

1.  我们公园里有各种恐龙及其相关数据。实现一个`List`接口来存储自定义的恐龙类。

1.  我们需要确保首先照顾到最危险的恐龙。编写一个`PriorityQueue`类，根据自定义的`Comparator`接口（例如，它们的危险级别）对恐龙进行排序。

1.  泛型可以使我们的代码更具可重用性。创建一个名为`Crate`的类，其中包含一个泛型，用于存储您想存储在该类中的内容。这可以是餐厅的食物或饮料，也可以是恐龙，如果我们需要重新安置它们。

1.  在你的程序中创建三个`Crate`类的实例，使用不同的类 - 例如，`Dinosaur`、`Jeep`和`DinosaurFood`。

1.  哈希对于高效的数据处理至关重要。在你的恐龙类中重写`hashCode()`方法。

1.  挑战：我们在寻找餐厅人员方面有一些问题。让我们自动化我们公园冰淇淋店的订购。编写一个程序来完成以下任务：

    +   询问客人会列出多少冰淇淋。

    +   对于每一款冰淇淋，询问他们想要什么口味（如果你敢的话，可以想出一些口味选择，并使其恐龙主题化）以及多少份。

    +   为了简单起见，让我们假设每位客人只能订购每种口味一次。将所有冰淇淋及其描述添加到一个包含映射的`List`接口中。这些映射将代表冰淇淋和冰淇淋球的份量。

1.  挑战：详细阐述*练习 13.6*。打印订单（遍历列表！）并说明它将在当前时间加 10 分钟后准备好（你需要计算这个时间，而不是直接打印出来！）

# 项目 - 高级恐龙养护系统

随着我们公园中恐龙数量的增加，对更复杂的数据管理系统需求变得明显。泛型和集合来拯救！

我们将继续构建恐龙养护系统。该系统应能处理恐龙集合，允许根据各种参数对恐龙进行排序，确保恐龙的唯一性，等等。

这里是我们将要采取的步骤。

**步骤 1：添加额外的** **Java 类**：

+   创建一个名为`collections`的新包。

+   在这个包内部，创建一个名为`DinosaurComparator`的类。这个类应该实现`Comparator<Dinosaur>`接口。重写`compare()`方法，根据年龄、大小等参数对恐龙进行排序。

注意

通常你不会为比较器创建一个类，但我们直到下一章才会看到 lambda 表达式。

**步骤 2：扩展恐龙** **养护系统**：

+   将`DinosaurCareSystem`类中持有`Dinosaur`对象的`List`接口更改为`Set`接口。这将确保恐龙的唯一性。

+   创建一个名为`sortDinosaurs()`的方法，使用`DinosaurComparator`对`Dinosaur`集合进行排序。

这里有一些示例代码供你开始：

```java
import java.util.*;public class DinosaurCareSystem {
    private Set<Dinosaur> dinosaurs;
    private List<Activity> activities;
    public DinosaurCareSystem() {
        dinosaurs = new HashSet<>();
        activities = new ArrayList<>();
    }
    public void addDinosaur(Dinosaur dinosaur) {
        dinosaurs.add(dinosaur);
    }
    public void logActivity(Activity activity) {
        activities.add(activity);
    }
    public List<Dinosaur> sortDinosaurs() {
        List<Dinosaur> sortedDinosaurs = new
          ArrayList<>(dinosaurs);
        Collections.sort(sortedDinosaurs, new
          DinosaurComparator());
        return sortedDinosaurs;
    }
    //... existing methods for handling exceptions and
          other functionalities here
}
```

以下是你可以使用的`DinosaurComparator`类：

```java
import java.util.Comparator;public class DinosaurComparator implements
  Comparator<Dinosaur> {
    @Override
    public int compare(Dinosaur d1, Dinosaur d2) {
    // assume Dinosaur has a getSize() method
        return d1.getSize().compareTo(d2.getSize());      }
}
```

**步骤 3：与** **系统** **交互**：

+   在你的`main`类中，你可以像之前步骤中那样与`DinosaurCareSystem`对象交互，但现在，添加根据参数对恐龙进行排序的功能。

你想要更多吗？你可以通过添加更多功能来扩展这个项目，例如根据不同参数进行排序，根据恐龙的特性进行搜索等。

# 概述

好的，你已经通过了另一个艰难的章节。在本章中，我们探讨了 Java 中集合和泛型的基础知识。我们首先讨论了编程中集合的需求，并概述了 Java 中可用的不同集合类型，包括`List`、`Set`、`Map`、`Queue`和`Deque`。我们检查了每种集合类型的特定实现，例如`ArrayList`、`LinkedList`、`HashSet`、`TreeSet`、`HashMap`、`TreeMap`等，以及它们的区别和适当的使用场景。我们还涵盖了基本操作，如向集合中添加、删除和遍历元素。

然后，我们转向了集合排序的学习。我们通过使用`Comparable`和`Comparator`接口区分了自然排序和自定义排序。我们学习了如何实现`compareTo()`和`compare()`方法，以及如何使用`Collections.sort()`方法和`TreeSet`、`TreeMap`类对列表、集合和映射进行排序。

接着，我们深入探讨了泛型，解释了它们在提供类型安全方面的重要性。我们演示了泛型的语法和基本用法，包括在有限泛型中使用`extends`关键字。

然后，我们继续学习如何通过定义泛型类来创建自定义泛型类型。我们还讨论了没有泛型时的含义，以及如何创建泛型类型的实例。

最后，我们讨论了基本哈希概念和在集合中`hashCode()`方法的作用。我们提供了重写`hashCode()`的指南和其实施的最佳实践，强调了它在自定义泛型类型中的重要性。

到目前为止，你应该已经对`List`、`Set`、`Map`和`Queue`之间的区别有了扎实的理解，并且对泛型和哈希的基本知识有所掌握。你现在可以准备学习下一个令人兴奋的主题：Lambda 表达式。
