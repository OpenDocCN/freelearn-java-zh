# 第十一章：链表和映射

本章涵盖了在编码面试中遇到的涉及映射和链表的最受欢迎的编码挑战。由于在技术面试中更喜欢使用单向链表，本章中的大多数问题将利用它们。但是，您可以挑战自己，尝试在双向链表的情况下解决每个问题。通常，对于双向链表来说，问题变得更容易解决，因为双向链表为每个节点维护两个指针，并允许我们在列表内前后导航。

通过本章结束时，您将了解涉及链表和映射的所有热门问题，并且将具有足够的知识和理解各种技术，以帮助您解决此类问题。我们的议程非常简单；我们将涵盖以下主题：

+   链表简介

+   映射简介

+   编码挑战

# 技术要求

本章中的所有代码文件都可以在 GitHub 上找到，网址为[`github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter11`](https://github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter11)。

但在进行编码挑战之前，让我们先了解一下链表和映射。

# 链表简介

链表是表示节点序列的线性数据结构。第一个节点通常被称为**头部**，而最后一个节点通常被称为**尾部**。当每个节点指向下一个节点时，我们有一个*单向链表*，如下图所示：

![11.1：单向链表](img/Figure_11.1_B15403.jpg)

图 11.1 – 单向链表

当每个节点指向下一个节点和前一个节点时，我们有一个*双向链表*，如下图所示：

![11.2：双向链表](img/Figure_11.2_B15403.jpg)

图 11.2 – 双向链表

让我们考虑一个单向链表。如果尾部指向头部，那么我们有一个*循环单向链表*。或者，让我们考虑一个双向链表。如果尾部指向头部，头部指向尾部，那么我们有一个*循环双向链表*。

在单向链表中，一个节点保存数据（例如，整数或对象）和指向下一个节点的指针。以下代码表示单向链表的节点：

```java
private final class Node {
  private int data;
  private Node next;
}
```

双向链表还需要指向前一个节点的指针：

```java
private final class Node {
  private int data;
  private Node next;
  private Node prev;
}
```

与数组不同，链表不提供访问第 n 个元素的常数时间。我们必须迭代 n-1 个元素才能获得第 n 个元素。我们可以在常数时间内从链表（单向和双向）的开头插入，删除和更新节点。如果我们的实现管理双向链表的尾部（称为双头双向链表），那么我们也可以在常数时间内从链表的末尾插入，删除和更新节点；否则，我们需要迭代链表直到最后一个节点。如果我们的实现管理单向链表的尾部（称为双头单向链表），那么我们可以在常数时间内在链表的末尾插入节点；否则，我们需要迭代链表直到最后一个节点。

本书的代码包包括以下应用程序（每个应用程序都公开`insertFirst()`、`insertLast()`、`insertAt()`、`delete()`、`deleteByIndex()`和`print()`方法）：

+   *SinglyLinkedList*：双头单向链表的实现

+   *SinglyLinkedListOneHead*：单头单向链表的实现

+   *DoublyLinkedList*：双头双向链表的实现

+   *DoublyLinkedListOneHead*：单头双向链表的实现

强烈建议您自己彻底分析这些应用程序。每个应用程序都有大量注释，以帮助您理解每个步骤。以下编码挑战依赖于这些链表实现。

# 简而言之，地图

想象一下，您正在字典中查找一个单词。这个单词本身是唯一的，可以被视为*键*。这个单词的意思可以被视为*值*。因此，这个单词及其意思形成了一个*键值对*。同样，在计算中，键值对容纳了一段数据，可以通过键来查找值。换句话说，我们知道键，我们可以用它来找到值。

地图是一个**抽象数据类型**（**ADT**），通过数组管理键值对（称为条目）。地图的特征包括以下内容：

+   键是唯一的（即，不允许重复键）。

+   我们可以查看键的列表，值的列表，或两者。

+   处理地图的最常见方法是`get()`，`put()`和`remove()`。

现在我们已经简要概述了链表和地图的概念，让我们开始我们的编码挑战。

# 编码挑战

在接下来的 17 个编码挑战中，我们将涵盖涉及地图和链表的许多问题。由于链表是技术面试中更受欢迎的话题，我们将为它们分配更多的问题。然而，为了掌握地图数据结构的概念，特别是内置的 Java 地图实现，我强烈建议您购买 Packt Publishing 出版的书籍*Java 编码问题*（[`www.packtpub.com/programming/java-coding-problems`](https://www.packtpub.com/programming/java-coding-problems)）。除了是本书的绝佳伴侣外，*Java 编码问题*还包含以下地图问题（请注意，这不是完整的列表）：

+   创建不可修改/不可变集合

+   映射默认值

+   计算`Map`中值的存在/不存在

+   从`Map`中删除

+   替换`Map`中的条目

+   比较两个地图

+   对`Map`进行排序

+   复制`HashMap`

+   合并两个地图

+   删除与谓词匹配的集合的所有元素

现在我们对链表和地图有了基本的了解，让我们来看看与地图和链表相关的面试中最常见的问题。

## 编码挑战 1 - Map put，get 和 remove

`put(K k, V v)`，一个名为`get(K k)`的方法，和一个名为`remove(K k)`的方法。

**解决方案**：正如您所知，地图是一个键值对数据结构。每个键值对都是地图的一个条目。因此，我们无法实现地图的功能，直到我们实现一个条目。由于一个条目包含两个信息，我们需要定义一个类来以通用的方式包装键和值。

代码非常简单：

```java
private final class MyEntry<K, V> {
  private final K key;
  private V value;
  public MyEntry(K key, V value) {
    this.key = key;
    this.value = value;
  }
  // getters and setters omitted for brevity
}
```

现在我们有了一个条目，我们可以声明一个地图。地图通过具有默认大小的条目数组来管理，这个默认大小称为地图容量。具有 16 个元素的初始容量的地图声明如下：

```java
private static final int DEFAULT_CAPACITY = 16;
private MyEntry<K, V>[] entries 
        = new MyEntry[DEFAULT_CAPACITY];
```

接下来，我们可以专注于使用这个数组作为客户端的地图。只有在条目的键在地图中是唯一的情况下，才能将条目放入地图中。如果给定的键存在，则只需更新其值。除此之外，只要我们没有超出地图的容量，就可以添加一个条目。在这种情况下的典型方法是将地图的大小加倍。基于这些语句的代码如下：

```java
private int size;
public void put(K key, V value) {
  boolean success = true;
  for (int i = 0; i < size; i++) {
    if (entries[i].getKey().equals(key)) {
      entries[i].setValue(value);
      success = false;
    }
  }
  if (success) {
    checkCapacity();
    entries[size++] = new MyEntry<>(key, value);
  }
}
```

以下辅助方法用于将地图的容量加倍。由于 Java 数组无法调整大小，我们需要通过创建初始数组的副本，但大小加倍来解决这个问题：

```java
private void checkCapacity() {
  if (size == entries.length) {
    int newSize = entries.length * 2;
    entries = Arrays.copyOf(entries, newSize);
  }
}
```

使用键来获取值。如果找不到给定的键，则返回`null`。获取值不会从地图中删除条目。让我们看一下代码：

```java
public V get(K key) {
  for (int i = 0; i < size; i++) {
    if (entries[i] != null) {
      if (entries[i].getKey().equals(key)) {
        return entries[i].getValue();
      }
    }
  }
  return null;
}
```

最后，我们需要使用键来删除一个条目。从数组中删除一个元素涉及将剩余的元素向前移动一个位置。元素移动后，倒数第二个和最后一个元素相等。通过将数组的最后一个元素置空，可以避免内存泄漏。忘记这一步是一个常见的错误：

```java
public void remove(K key) {
  for (int i = 0; i < size; i++) {
    if (entries[i].getKey().equals(key)) {
      entries[i] = null;
      size--;
      condenseArray(i);
    }
  }
}
private void condenseArray(int start) {
  int i;
  for (i = start; i < size; i++) {
    entries[i] = entries[i + 1];
  }
  entries[i] = null; // don't forget this line
}
```

地图的生产实现比这里展示的要复杂得多（例如，地图使用桶）。然而，很可能在面试中你不需要了解比这个实现更多的内容。尽管如此，向面试官提到这一点是个好主意。这样，你可以向他们展示你理解问题的复杂性，并且你意识到了这一点。

完成！完整的应用程序名为*Map*。

## 编码挑战 2 - 映射键集和值

`keySet()`）和一个返回值集合的方法（`values()`）。

`Set`。以下代码不言自明：

```java
public Set<K> keySet() {
  Set<K> set = new HashSet<>();
  for (int i = 0; i < size; i++) {
    set.add(entries[i].getKey());
  }
  return set;
}
```

为了返回一个值的集合，我们循环遍历映射并将值逐个添加到`List`中。我们使用`List`，因为值可能包含重复项：

```java
public Collection<V> values() {
  List<V> list = new ArrayList<>();
  for (int i = 0; i < size; i++) {
    list.add(entries[i].getValue());
  }
  return list;
}
```

完成！这很简单；生产中实现的地图比这里展示的要复杂得多。例如，值被缓存而不是每次都被提取。向面试官提到这一点，让他/她看到你知道生产地图是如何工作的。花点时间检查 Java 内置的`Map`和`HashMap`源代码。

完整的应用程序名为*Map*。

## 编码挑战 3 - 螺母和螺栓

**谷歌**，**Adobe**

**问题**：给定*n*个螺母和*n*个螺栓，考虑它们之间的一一对应关系。编写一小段代码，找出螺母和螺栓之间的所有匹配项，使迭代次数最少。

**解决方案**：让我们假设螺母和螺栓分别由以下两个数组表示：

```java
char[] nuts = {'$', '%', '&', 'x', '@'};
char[] bolts = {'%', '@', 'x', '$', '&'};
```

最直观的解决方案依赖于蛮力方法。我们可以选择一个螺母，并迭代螺栓以找到它的配偶。例如，如果我们选择`nuts[0]`，我们可以用`bolts[3]`找到它的配偶。此外，我们可以取`nuts[1]`，并用`bolts[0]`找到它的配偶。这个算法非常简单，可以通过两个`for`语句来实现，并且具有 O(n2)的时间复杂度。

或者，我们可以考虑对螺母和螺栓进行排序。这样，螺母和螺栓之间的匹配将自动对齐。这也可以工作，但不会包括最少的迭代次数。

为了获得最少的迭代次数，我们可以使用哈希映射。在这个哈希映射中，首先，我们将每个螺母作为一个键，将其在给定螺母数组中的位置作为一个值。接下来，我们迭代螺栓，并检查哈希映射是否包含每个螺栓作为一个键。如果哈希映射包含当前螺栓的键，那么我们找到了一个匹配（一对）；否则，这个螺栓没有匹配。让我们看一下代码：

```java
public static void match(char[] nuts, char[] bolts) {
  // in this map, each nut is a key and 
  // its position is as value
  Map<Character, Integer> map = new HashMap<>();
  for (int i = 0; i < nuts.length; i++) {
    map.put(nuts[i], i);
  }
  //for each bolt, search a nut
  for (int i = 0; i < bolts.length; i++) {
    char bolt = bolts[i];
    if (map.containsKey(bolt)) {
      nuts[i] = bolts[i];
    } else {
      System.out.println("Bolt " + bolt + " has no nut");
    }
  }
  System.out.println("Matches between nuts and bolts: ");
  System.out.println("Nuts: " + Arrays.toString(nuts));
  System.out.println("Bolts: " +Arrays.toString(bolts));
}
```

这段代码的运行时间是 O(n)。完整的代码名为*NutsAndBolts*。

## 编码挑战 4 - 删除重复项

**亚马逊**，**谷歌**，**Adobe**，**微软**

**问题**：考虑一个未排序的整数单向链表。编写一小段代码来删除重复项。

`Set<Integer>`。然而，在将当前节点的数据添加到`Set`之前，我们检查数据是否与`Set`的当前内容相匹配。如果`Set`已经包含该数据，我们就从链表中删除节点；否则，我们只是将其数据添加到`Set`中。从单向链表中删除节点可以通过将前一个节点链接到当前节点的下一个节点来完成。

以下图示说明了这个陈述：

![11.3: 从单向链表中删除节点](img/Figure_11.3_B15403.jpg)

图 11.3 - 从单向链表中删除节点

由于单链表只保存指向下一个节点的指针，我们无法知道当前节点之前的节点。技巧是跟踪两个连续的节点，从当前节点作为链表头部和前一个节点作为`null`开始。当当前节点前进到下一个节点时，前一个节点前进到当前节点。让我们看一下将这些语句组合在一起的代码：

```java
// 'size' is the linked list size
public void removeDuplicates() {
  Set<Integer> dataSet = new HashSet<>();
  Node currentNode = head;
  Node prevNode = null;
  while (currentNode != null) {
    if (dataSet.contains(currentNode.data)) {
      prevNode.next = currentNode.next;
      if (currentNode == tail) {
        tail = prevNode;
      }
      size--;
    } else {
      dataSet.add(currentNode.data);
      prevNode = currentNode;
    }
    currentNode = currentNode.next;
  }
}
```

这个解决方案的时间和空间复杂度为 O(n)，其中*n*是链表中的节点数。我们可以尝试另一种方法，将空间复杂度降低到 O(1)。首先，让我们将以下图表作为下一步的指南：

![11.4：从单链表中移除节点](img/Figure_11.4_B15403.jpg)

图 11.4 - 从单链表中移除节点

这种方法使用两个指针：

1.  当前节点从链表的头部开始遍历链表，直到到达尾部（例如，在前面的图表中，当前节点是第二个节点）。

1.  奔跑者节点，从与当前节点相同的位置开始，即链表的头部。

此外，奔跑者节点遍历链表，并检查每个节点的数据是否等于当前节点的数据。当奔跑者节点遍历链表时，当前节点的位置保持不变。

如果奔跑者节点检测到重复，那么它会将其从链表中移除。当奔跑者节点到达链表的尾部时，当前节点前进到下一个节点，奔跑者节点再次从当前节点开始遍历链表。因此，这是一个 O(n2)时间复杂度的算法，但空间复杂度为 O(1)。让我们看一下代码：

```java
public void removeDuplicates() {
  Node currentNode = head;
  while (currentNode != null) {
    Node runnerNode = currentNode;
    while (runnerNode.next != null) {
      if (runnerNode.next.data == currentNode.data) {
        if (runnerNode.next == tail) {
          tail = runnerNode;
        }
        runnerNode.next = runnerNode.next.next;
        size--;
      } else {
        runnerNode = runnerNode.next;
      }
    }
    currentNode = currentNode.next;
  }
}
```

完整的代码名为*LinkedListRemoveDuplicates*。

## 编码挑战 5 - 重新排列链表

**Adobe**，**Flipkart**，**Amazon**

**问题**：考虑一个未排序的整数单链表和一个给定的整数*n*。编写一小段代码，围绕*n*重新排列节点。换句话说，最后，链表将包含所有小于*n*的值，后面跟着所有大于*n*的节点。节点的顺序可以改变，*n*本身可以位于大于*n*的值之间的任何位置。

**解决方案**：假设给定的链表是 1→5→4→3→2→7→null，*n*=3。所以，3 是我们的枢轴。其余的节点应该围绕这个枢轴重新排列，符合问题的要求。解决这个问题的一个方法是逐个遍历链表节点，并将小于枢轴的每个节点放在头部，而大于枢轴的每个节点放在尾部。以下图表帮助我们可视化这个解决方案：

![11.5：链表重新排列](img/Figure_11.5_B15403.jpg)

图 11.5 - 链表重新排列

因此，值为 5、4 和 3 的节点被移动到尾部，而值为 2 的节点被移动到头部。最后，所有小于 3 的值都在虚线的左侧，而所有大于 3 的值都在虚线的右侧。我们可以将此算法编写成以下代码：

```java
public void rearrange(int n) {
  Node currentNode = head;
  head = currentNode;
  tail = currentNode;
  while (currentNode != null) {
    Node nextNode = currentNode.next;
    if (currentNode.data < n) {
      // insert node at the head
      currentNode.next = head;
      head = currentNode;
    } else {
      // insert node at the tail
      tail.next = currentNode;
      tail = currentNode;
    }
    currentNode = nextNode;
  }
  tail.next = null;
}
```

完整的应用程序名为*LinkedListRearranging*。

## 编码挑战 6 - 倒数第 n 个节点

**Adobe**，**Flipkart**，**Amazon**，**Google**，**Microsoft**

**问题**：考虑一个整数单链表和一个给定的整数*n*。编写一小段代码，返回倒数第 n 个节点的值。

**解决方案**：我们有一堆节点，我们必须找到满足给定约束的第*n*个节点。根据我们从*第八章*的经验，*递归和动态规划*，我们可以直觉地认为这个问题有一个涉及递归的解决方案。但我们也可以通过迭代解决它。由于迭代解决方案更有趣，我将在这里介绍它，而递归解决方案在捆绑代码中可用。

让我们使用以下图表来呈现算法（按照从上到下的顺序遵循图表）：

![11.6: The nth to last node](img/Figure_11.6_B15403.jpg)

图 11.6 - 最后第 n 个节点

因此，我们有一个链表，2 → 1 → 5 → 9 → 8 → 3 → 7 → null，并且我们想要找到第五个到最后一个节点值，即 5（您可以在前面的图表顶部看到）。迭代解决方案使用两个指针；让我们将它们表示为*runner1*和*runner2*。最初，它们都指向链表的头部。在步骤 1（前面图表的中间），我们将*runner1*从头移动到第 5 个到头（或*n*到头）节点。这在`for`循环中从 0 到 5（或*n*）中很容易实现。在步骤 2（前面图表的底部），我们同时移动*runner1*和*runner2*，直到*runner1*为`null`。当*runner1*为`null`时，*runner2*将指向距离头部第五个到最后一个节点（或*n*到最后一个）节点。在代码行中，我们可以这样做：

```java
public int nthToLastIterative(int n) {
  // both runners are set to the start
  Node firstRunner = head;
  Node secondRunner = head;
  // runner1 goes in the nth position
  for (int i = 0; i < n; i++) {
    if (firstRunner == null) {
      throw new IllegalArgumentException(
             "The given n index is out of bounds");
    }
    firstRunner = firstRunner.next;
  }
  // runner2 run as long as runner1 is not null
  // basically, when runner1 cannot run further (is null), 
  // runner2 will be placed on the nth to last node
  while (firstRunner != null) {
    firstRunner = firstRunner.next;
    secondRunner = secondRunner.next;
  }
  return secondRunner.data;
}
```

完整的应用程序名为*LinkedListNthToLastNode*。

## 编码挑战 7 - 循环开始检测

**Adobe**，**Flipkart**，**Amazon**，**Google**，**Microsoft**

**问题**：考虑一个包含循环的整数单链表。换句话说，链表的尾部指向之前的一个节点，定义了一个循环或循环。编写一小段代码来检测循环的第一个节点（即循环开始的节点）。

`tail.next`. 如果我们不管理尾部，那么我们可以搜索具有两个指向它的节点的节点。这也很容易实现。如果我们知道链表的大小，那么我们可以从 0 到大小进行迭代，最后一个`node.next`指向标记循环开始的节点。

### 快跑者/慢跑者方法

然而，让我们尝试另一种需要更多想象力的算法。这种方法称为快跑者/慢跑者方法。它很重要，因为它可以用于涉及链表的某些问题。

主要的快跑者/慢跑者方法涉及使用两个指针，它们从链表的头部开始，并同时遍历列表，直到满足某些条件。一个指针被命名为**慢跑者**（**SR**），因为它逐个节点地遍历列表。另一个指针被命名为**快跑者**（**FR**），因为它在每次移动时跳过下一个节点来遍历列表。以下图表是四个移动的示例：

![11.7: Fast Runner/Slow Runner example](img/Figure_11.7_B15403.jpg)

图 11.7 - 快跑者/慢跑者示例

因此，在第一步移动时，*FR*和*SR*指向*head*。在第二步移动时，*SR*指向值为 1 的*head.next*节点，而*FR*指向值为 4 的*head.next.next*节点。移动继续遵循这种模式。当*FR*到达链表的尾部时，*SR*指向中间节点。

正如您将在下一个编码挑战中看到的，快跑者/慢跑者方法可以用于检测链表是否是回文。但是，现在让我们恢复我们的问题。那么，我们可以使用这种方法来检测链表是否有循环，并找到此循环的起始节点吗？这个问题引发了另一个问题。如果我们将快跑者/慢跑者方法应用于具有循环的链表，*FR*和*SR*指针会相撞或相遇吗？答案是肯定的，它们会相撞。

解释一下，假设在开始循环之前，我们有*q*个先行节点（这些节点在循环外）。对于*SR*遍历的每个*q*个节点，*FR*已经遍历了 2**q*个节点（这是显而易见的，因为*FR*在每次移动时都会跳过一个节点）。因此，当*SR*进入循环（到达循环起始节点）时，*FR*已经遍历了 2**q*个节点。换句话说，*FR*在循环部分的 2**q-q*节点处；因此，它在循环部分的*q*个节点处。让我们通过以下测试案例来形象化这一点：

![11.8: 带有循环的链表](img/Figure_11.8_B15403.jpg)

图 11.8 - 带有循环的链表

因此，当*SR*进入循环（到达第四个节点）时，*FR*也到达了循环的第四个节点。当然，我们需要考虑到*q*（先行非循环节点的数量）可能比循环长度要大得多；因此，我们应该将 2**q-q*表示为*Q=modulo(q, LOOP_SIZE)*。

例如，考虑*Q = modulo*(3, 8) =3，其中我们有三个非循环节点（*q*=3），循环大小为八（*LOOP_SIZE*=8）。在这种情况下，我们也可以应用 2**q-q*，因为 2*3-3=3。因此，我们可以得出*SR*距离列表开头三个节点，*FR*距离循环开头三个节点。然而，如果链表前面有 25 个节点，后面有 7 个节点的循环，那么*Q = modulo* (25, 7) = 4 个节点，而 2*25-25=25，这是错误的。

除此之外，*FR*和*SR*在循环内移动。由于它们在一个圆圈内移动，这意味着当*FR*远离*SR*时，它也在向*SR*靠近，反之亦然。下图将循环隔离出来，并展示了它们如何继续移动*FR*和*SR*直到它们相撞：

![11.9: FR and SR collision](img/Figure_11.9_B15403.png)

图 11.9 - FR 和 SR 碰撞

花时间追踪*SR*和*FR*直到它们到达相遇点。我们知道*FR*比*FR*落后*LOOP_SIZE - Q*个节点，*SR*比*FR*落后*Q*个节点。在我们的测试案例中，*FR*比*SR*落后 8-3=5 个节点，*SR*比*FR*落后 3 个节点。继续移动*SR*和*FR*，我们可以看到*FR*以每次移动 1 步的速度追上了。

那么，它们在哪里相遇呢？如果*FR*以每次移动 1 步的速度追上，*FR*比*SR*落后*LOOP_SIZE - Q*个节点，那么它们将在离循环头部*Q*步的地方相遇。在我们的测试案例中，它们将在距离循环头部 3 步的地方相遇，节点值为 8。

如果相遇点距离循环头部的节点数为*Q*，我们可以继续回想相遇点距离循环头部的节点数也为*q*，因为*Q=modulo(q, LOOP_SIZE)*。这意味着我们可以制定以下四步算法：

1.  从链表的头部开始*FR*和*SR*。

1.  将*SR*以 1 个节点的速度移动，*FR*以 2 个节点的速度移动。

1.  当它们相撞（在相遇点），将*SR*移动到链表的头部，保持*FR*在原地。

1.  将*SR*和*FR*以 1 个节点的速度移动，直到它们相撞（这是代表循环头部的节点）。

让我们把这写成代码：

```java
public void findLoopStartNode() {
  Node slowRunner = head;
  Node fastRunner = head;
  // fastRunner meets slowRunner
  while (fastRunner != null && fastRunner.next != null) {
    slowRunner = slowRunner.next;
    fastRunner = fastRunner.next.next;
    if (slowRunner == fastRunner) { // they met
      System.out.println("\nThe meet point is at 
        the node with value: " + slowRunner);
      break;
    }
  }
  // if no meeting point was found then there is no loop
  if (fastRunner == null || fastRunner.next == null) {
    return;
  }
  // the slowRunner moves to the head of the linked list
  // the fastRunner remains at the meeting point
  // they move simultaneously node-by-node and 
  // they should meet at the loop start
  slowRunner = head;
  while (slowRunner != fastRunner) {
    slowRunner = slowRunner.next;
    fastRunner = fastRunner.next;
  }
  // both pointers points to the start of the loop
  System.out.println("\nLoop start detected at 
      the node with value: " + fastRunner);
}
```

作为一个快速的提示，不要期望*FR*能够跳过*SR*，所以它们不会相遇。这种情况是不可能的。想象一下，*FR*已经跳过了*SR*，它在节点*a*，那么*SR*必须在节点*a*-1。这意味着，在上一步中，*FR*在节点*a*-2，*SR*在节点(*a*-1)-1=*a*-2；因此，它们已经相撞了。

完整的应用程序名为*LinkedListLoopDetection*。在这段代码中，你会找到一个名为`generateLoop()`的方法。调用这个方法可以生成带有循环的随机链表。

## 编码挑战 8 - 回文

Adobe，Flipkart，Amazon，Google，Microsoft

如果链表是回文的，则返回`true`。解决方案应该涉及快速运行者/慢速运行者方法（这种方法在先前的编码挑战中有详细介绍）。

**解决方案**：只是一个快速提醒，回文（无论是字符串、数字还是链表）在翻转时看起来没有变化。这意味着处理（读取）回文可以从两个方向进行，得到的结果是相同的（例如，数字 12321 是一个回文，而数字 12322 不是）。

我们可以通过思考，当*FR*到达链表的末尾时，*SR*正好在链表的中间，来直观地得出使用快慢指针方法的解决方案。

如果链表的前半部分是后半部分的倒序，那么链表就是一个回文。因此，如果我们在栈中存储*FR*到达链表末尾之前*SR*遍历的所有节点，那么结果栈将包含链表前半部分的倒序。让我们通过以下图表来可视化这一点：

![11.10：使用快慢指针方法的链表回文](img/Figure_11.10_B15403.jpg)

图 11.10 - 使用快慢指针方法的链表回文

因此，当*FR*到达链表的末尾，*SR*到达第四个节点（链表的中间）时，栈包含值 2、1 和 4。接下来，我们可以继续以 1 个节点的速度移动*SR*，直到链表的末尾。在每次移动时，我们从栈中弹出一个值，并将其与当前节点的值进行比较。如果我们发现不匹配，那么链表就不是回文。在代码中，我们有以下内容：

```java
public boolean isPalindrome() {
  Node fastRunner = head;
  Node slowRunner = head;
  Stack<Integer> firstHalf = new Stack<>();
  // the first half of the linked list is added into the stack
  while (fastRunner != null && fastRunner.next != null) {
    firstHalf.push(slowRunner.data);
    slowRunner = slowRunner.next;
    fastRunner = fastRunner.next.next;
  }
  // for odd number of elements we to skip the middle node
  if (fastRunner != null) {
    slowRunner = slowRunner.next;
  }
  // pop from the stack and compare with the node by node of 
  // the second half of the linked list
  while (slowRunner != null) {
    int top = firstHalf.pop();
    // a mismatch means that the list is not a palindrome
    if (top != slowRunner.data) {
      return false;
    }
    slowRunner = slowRunner.next;
  }
  return true;
}
```

完整的应用程序名为*LinkedListPalindrome*。

## 编码挑战 9 - 两个链表相加

**Adobe**，**Flipkart**，**Microsoft**

**问题**：考虑两个正整数和两个单链表。第一个整数按位存储在第一个链表中（第一个数字是第一个链表的头）。第二个整数按位存储在第二个链表中（第一个数字是第二个链表的头）。编写一小段代码，将这两个数字相加，并将和作为一个链表返回，每个节点一个数字。

**解决方案**：让我们从一个测试案例的可视化开始：

![11.11：将两个数字作为链表相加](img/Figure_11.11_B15403.jpg)

图 11.11 - 将两个数字作为链表相加

如果我们逐步计算前面图表的总和，我们得到以下结果：

我们添加 7 + 7 = 14，所以我们写下 4 并携带 1：

结果链表是 4 →？

我们添加 3 + 9 + 1 = 13，所以我们写下 3 并携带 1：

结果链表是 4 → 3 →？

我们添加 8 + 8 + 1 = 17，所以我们写下 7 并携带 1：

结果链表是 4 → 3 → 7 →？

我们添加 9 + 4 + 1 = 14，所以我们写下 4 并携带 1

结果链表是 4 → 3 → 7 → 4 →？

我们添加 4 + 1 = 5，所以我们写下 5 并携带无：

结果链表是 4 → 3 → 7 → 4 → 5 →？

我们添加 1 + 0 = 1，所以我们写下 1 并携带无：

结果链表是 4 → 3 → 7 → 4 → 5 → 1 →？

我们添加 2 + 0 = 2，所以我们写下 2 并携带无：

结果链表是 4 → 3 → 7 → 4 → 5 → 1 → 2

如果我们将结果链表写成一个数字，我们得到 4374512；因此，我们需要将其反转为 2154734。虽然反转结果链表的方法（可以被视为一个编码挑战）可以在捆绑代码中找到，但以下方法以递归的方式应用了前面的步骤（如果你不擅长递归问题，请不要忘记阅读*第八章*，*递归和动态规划*）。基本上，以下递归通过逐个节点添加数据，将任何多余的数据传递到下一个节点：

```java
private Node sum(Node node1, Node node2, int carry) {
  if (node1 == null && node2 == null && carry == 0) {
    return null;
  }
  Node resultNode = new Node();
  int value = carry;
  if (node1 != null) {
    value += node1.data;
  }
  if (node2 != null) {
    value += node2.data;
  }
  resultNode.data = value % 10;
  if (node1 != null || node2 != null) {
    Node more = sum(node1 == null
        ? null : node1.next, node2 == null
        ? null : node2.next, value >= 10 ? 1 : 0);
    resultNode.next = more;
  }
  return resultNode;
}
```

完整的应用程序名为*LinkedListSum*。

## 编码挑战 10 - 链表交集

**Adobe**，**Flipkart**，**Google**，**Microsoft**

**问题**：考虑两个单链表。编写一小段代码，检查这两个列表是否相交。交集是基于引用的，而不是基于值的，但是你应该返回交集节点的值。因此，通过引用检查交集并返回值。

**解决方案**：如果你不确定*两个链表的交集*是什么意思，那么我们建议你勾画一个测试用例，并与面试官讨论细节。下面的图表展示了这样一个情况：

![11.12: 两个列表的交集](img/Figure_11.12_B15403.jpg)

图 11.12 – 两个列表的交集

在这个图表中，我们有两个相交的列表，它们在值为 8 的节点处相交。因为我们谈论的是引用交集，这意味着值为 9 和值为 4 的节点指向值为 8 的节点的内存地址。

主要问题是列表的大小不同。如果它们的大小相等，我们可以从头到尾遍历它们，逐个节点，直到它们相撞（直到*node_list_1.next= node_list_2.next*）。如果我们能跳过值为 2 和 1 的节点，我们的列表将是相同大小的（参考下一个图表；因为第一个列表比第二个列表长，我们应该从标记为*虚拟头*的节点开始迭代）：

![11.13: Removing the first two nodes of the top list](img/Figure_11.13_B15403.jpg)

图 11.13 – 移除顶部列表的前两个节点

记住这个陈述，我们可以推导出以下算法：

1.  确定列表的大小。

1.  如果第一个列表（我们将其表示为*l1*）比第二个列表（我们将其表示为*l2*）长，那么将第一个列表的指针移动到（*l1-l2*）。

1.  如果第一个列表比第二个列表短，那么将第二个列表的指针移动到（*l2-l1*）。

1.  逐个移动两个指针，直到达到末尾或者它们相撞为止。

将这些步骤转化为代码是直接的：

```java
public int intersection() {
  // this is the head of first list
  Node currentNode1 = {head_of_first_list};
  // this is the head of the second list
  Node currentNode2 = {head_of_second_list};
  // compute the size of both linked lists
  // linkedListSize() is just a helper method
  int s1 = linkedListSize(currentNode1);
  int s2 = linkedListSize(currentNode2);
  // the first linked list is longer than the second one
  if (s1 > s2) {
    for (int i = 0; i < (s1 - s2); i++) {
      currentNode1 = currentNode1.next;
    }
  } else {
    // the second linked list is longer than the first one
    for (int i = 0; i < (s2 - s1); i++) {
      currentNode2 = currentNode2.next;
    }
  }
  // iterate both lists until the end or the intersection node
  while (currentNode1 != null && currentNode2 != null) {
    // we compare references not values!
    if (currentNode1 == currentNode2) {
      return currentNode1.data;
    }
    currentNode1 = currentNode1.next;
    currentNode2 = currentNode2.next;
  }
  return -1;
}
```

完整的应用程序名为*LinkedListsIntersection*。在代码中，你会看到一个名为`generateTwoLinkedListWithInterection()`的辅助方法。这用于生成具有交集点的随机列表。

## 编码挑战 11 – 交换相邻节点

**亚马逊**，**谷歌**

**问题**：考虑一个单链表。编写一小段代码，交换相邻的节点，使得一个列表，比如 1 → 2 → 3 → 4 → null，变成 2 → 1 → 4 → 3 → null。考虑交换相邻的节点，而不是它们的值！

**解决方案**：我们可以将交换两个相邻节点*n1*和*n2*的问题简化为找到解决方案。交换两个值（例如，两个整数*v1*和*v2*）的一个众所周知的技巧依赖于一个辅助变量，并且可以写成如下形式：

*aux = v1; v1 = v2; v2 = aux;*

然而，我们不能对节点应用这种简单的方法，因为我们必须处理它们的链接。仅仅写下面这样是不够的：

*aux = n1; n1 = n2; n2 = aux;*

如果我们依赖这种简单的方法来交换*n1*和*n2*，那么我们将得到类似于以下图表的东西（注意，在交换*n1*和*n2*之后，我们有*n1.next* = *n3*和*n2.next* = *n1*，这是完全错误的）：

![11.14: Plain swapping with broken links (1)](img/Figure_11.14_B15403.jpg)

图 11.14 – 交换破损链接（1）

但是我们可以修复链接，对吧？嗯，我们可以明确地设置*n1.next*指向*n2*，并设置*n2.next*指向*n3*：

*n1.next = n2*

*n2.next = n3*

现在应该没问题了！我们可以交换两个相邻的节点。然而，当我们交换一对节点时，我们也会破坏两对相邻节点之间的链接。下面的图表说明了这个问题（我们交换并修复了*n1-n2*对和*n3-n4*对的链接）：

![11.15: Plain swapping with broken links (2)](img/Figure_11.15_B15403.jpg)

图 11.15 – 交换破损链接（2）

注意，在交换这两对之后，*n2.next*指向了* n4*，这是错误的。因此，我们必须修复这个链接。为此，我们可以存储*n2*，在交换*n3-n4*之后，我们可以通过设置*n2.next=n3*来修复链接。现在，一切看起来都很好，我们可以将其放入代码中：

```java
public void swap() {
  if (head == null || head.next == null) {
    return;
  }
  Node currentNode = head;
  Node prevPair = null;
  // consider two nodes at a time and swap their links
  while (currentNode != null && currentNode.next != null) {
    Node node1 = currentNode;           // first node
    Node node2 = currentNode.next;      // second node                    
    Node node3 = currentNode.next.next; // third node            
    // swap node1 node2
    Node auxNode = node1;
    node1 = node2;
    node2 = auxNode;
    // repair the links broken by swapping
    node1.next = node2;
    node2.next = node3;
    // if we are at the first swap we set the head
    if (prevPair == null) {
      head = node1;
    } else {
      // we link the previous pair to this pair
      prevPair.next = node1;
    }
    // there are no more nodes, therefore set the tail
    if (currentNode.next == null) {
      tail = currentNode;
    }
    // prepare the prevNode of the current pair
    prevPair = node2;
    // advance to the next pair
    currentNode = node3;
  }
}
```

完整的应用程序名为*LinkedListPairwiseSwap*。考虑挑战自己交换*n*个节点的序列。

## 编码挑战 12 - 合并两个排序的链表

**亚马逊**，**谷歌**，**Adobe**，**微软**，**Flipkart**

**问题**：考虑两个排序的单链表。编写一小段代码，将这两个列表合并而不使用额外空间。

**解决方案**：所以，我们有两个排序的列表，*list1*：4 → 7 → 8 → 10 → null 和*list2*：5 → 9 → 11 → null，我们希望得到结果，4 → 5 → 7 → 8 → 9 → 10 → 11 → null。此外，我们希望在不分配新节点的情况下获得这个结果。

由于我们不能分配新节点，我们必须选择其中一个列表成为最终结果或合并的链表。换句话说，我们可以从*list1*开始作为合并的链表，并在*list1*的适当位置添加*list2*的节点。在处理每次比较后，我们将指针（*list1*）移动到合并列表的最后一个节点。

例如，我们首先比较这两个列表的头部。如果*list1*的头部小于*list2*的头部，我们选择*list1*的头部作为合并列表的头部。否则，如果*list1*的头部大于*list2*的头部，我们交换头部。以下图表说明了这一步骤：

![图 11.16 - 合并两个排序的链表（步骤 1）](img/Figure_11.16_B15403.jpg)

图 11.16 - 合并两个排序的链表（步骤 1）

由于*list1*的头部小于*list2*的头部（4 < 5），它成为了合并列表的头部。我们说*list1*将指向合并列表的最后一个节点；因此，下一个要比较的节点应该是*list1.next*（值为 7 的节点）和*list2*（值为 5 的节点）。以下图表显示了这个比较的结果：

![图 11.17 - 合并两个排序的链表（步骤 2）](img/Figure_11.17_B15403.jpg)

图 11.17 - 合并两个排序的链表（步骤 2）

因为*list1*跟随合并后的列表（最终结果），我们必须将*list1.next*移动到值为 5 的节点，但我们不能直接这样做。如果我们说*list1.next=list2*，那么我们就会失去*list1*的其余部分。因此，我们必须执行一次交换，如下所示：

```java
Node auxNode = list1.next; // auxNode = node with value 7
list1.next = list2;        // list1.next = node with value 5
list2 = auxNode;           // list2 = node with value 7
```

接下来，我们将*list1*移动到*list1.next*，也就是值为 9 的节点。我们将*list.next*与*list2*进行比较；因此，我们将 9 与 7 进行比较。以下图表显示了这个比较的结果：

![图 11.18 - 合并两个排序的链表（步骤 3）](img/Figure_11.18_B15403.jpg)

图 11.18 - 合并两个排序的链表（步骤 3）

因为*list1*跟随合并后的列表（最终结果），我们必须将*list1.next*移动到值为 7 的节点（因为 7 < 9），我们使用之前讨论过的交换来完成。接下来，我们将*list1*移动到*list1.next*，也就是值为 8 的节点。我们将*list.next*与*list2*进行比较；因此，我们将 8 与 9 进行比较。以下图表显示了这个比较的结果：

![图 11.19 - 合并两个排序的链表（步骤 4）](img/Figure_11.19_B15403.jpg)

图 11.19 - 合并两个排序的链表（步骤 4）

由于 8 < 9，不需要交换。我们将*list1.next*移动到下一个节点（值为 10 的节点）并将 10 与 9 进行比较。下一个图表显示了这个比较的结果：

![图 11.20 - 合并两个排序的链表（步骤 5）](img/Figure_11.20_B15403.jpg)

图 11.20 - 合并两个排序的链表（步骤 5）

作为*list1*跟随合并后的列表（最终结果），我们必须将*list1.next*移动到值为 9 的节点（因为 9 < 10），我们使用之前讨论过的交换来完成。接下来，我们将*list1*移动到*list1.next*，这是值为 11 的节点。我们将*list.next*与*list2*进行比较；因此，我们将 11 与 10 进行比较。下一个图表显示了这个比较的结果：

![11.21：合并两个排序的链表（第 6 步）](img/Figure_11.21_B15403.jpg)

图 11.21 - 合并两个排序的链表（第 6 步）

因为*list1*跟随合并后的列表（最终结果），我们必须将*list1.next*移动到值为 10 的节点（因为 10 < 11），我们使用之前讨论过的交换来完成。接下来，我们将*list1*移动到*list1.next*，这是`null`；因此，我们从*list2*中复制剩余部分。下一个图表显示了这个比较的结果：

![11.22：合并两个排序的链表（最后一步）](img/Figure_11.22_B15403.jpg)

图 11.22 - 合并两个排序的链表（最后一步）

此时，合并后的链表已经完成。现在是时候揭示代码了（这个方法被添加到了著名的`SinglyLinkedList`中）：

```java
public void merge(SinglyLinkedList sll) {
  // these are the two lists
  Node list1 = head;      // the merged linked list 
  Node list2 = sll.head;  // from this list we add nodes at 
                          // appropriate place in list1
  // compare heads and swap them if it is necessary
  if (list1.data < list2.data) {
    head = list1;
  } else {
    head = list2;
    list2 = list1;
    list1 = head;
  }
  // compare the nodes from list1 with the nodes from list2
  while (list1.next != null) {
    if (list1.next.data > list2.data) {
      Node auxNode = list1.next;
      list1.next = list2;
      list2 = auxNode;
    }
    // advance to the last node in the merged linked list              
    list1 = list1.next;
  }
  // add the remaining list2
  if (list1.next == null) {
    list1.next = list2;
  }
}
```

完整的应用程序名为*LinkedListMergeTwoSorted*。类似的问题可能要求您通过递归合并两个排序的链表。虽然您可以找到名为*LinkedListMergeTwoSortedRecursion*的应用程序，但我建议您挑战自己尝试一种实现。此外，基于这种递归实现，挑战自己合并*n*个链表。完整的应用程序名为*LinkedListMergeNSortedRecursion*。

## 编码挑战 13 - 去除多余路径

**问题**：考虑一个存储矩阵中路径的单链表。节点的数据类型为(*行，列*)或简写为(*r，c*)。路径只能是水平（按*列*）或垂直（按*行*）。完整路径由所有水平和垂直路径的终点给出；因此，中间点（或中间的点）是多余的。编写一小段代码，删除多余的路径。

**解决方案**：让我们考虑一个包含以下路径的链表：(0, 0) → (0, 1) → (0, 2) → (1, 2) → (2, 2) → (3, 2) → (3, 3) → (3, 4) → null。多余的路径包括以下节点：(0, 1)，(1, 2)，(2, 2)和(3, 3)。因此，在移除多余路径后，我们应该保留一个包含四个节点的列表：(0, 0) → (0, 2) → (3, 2) → (3, 4) → null。下一个图表表示了多余的路径：

![11.23：多余的路径](img/Figure_11.23_B15403.jpg)

图 11.23 - 多余的路径

去除多余路径后，我们得到以下图表：

![11.24：去除冗余后的剩余路径](img/Figure_11.24_B15403.jpg)

图 11.24 - 去除冗余后的剩余路径

前面的图表应该提供了这个问题的解决方案。请注意，定义垂直路径的节点具有相同的列，因为我们只在行上下移动，而定义水平路径的节点具有相同的行，因为我们只在列左右移动。这意味着，如果我们考虑具有相同列或行的值的三个连续节点，那么我们可以移除中间节点。对相邻三元组重复此过程将移除所有多余节点。代码应该非常简单易懂：

```java
public void removeRedundantPath() {
  Node currentNode = head;
  while (currentNode.next != null 
          && currentNode.next.next != null) {
    Node middleNode = currentNode.next.next;
    // check for a vertical triplet (triplet with same column)
    if (currentNode.c == currentNode.next.c
            && currentNode.c == middleNode.c) {
      // delete the middle node
      currentNode.next = middleNode;
    } // check for a horizontal triplet 
    else if (currentNode.r == currentNode.next.r
            && currentNode.r == middleNode.r) {
      // delete the middle node
      currentNode.next = middleNode;
    } else {
      currentNode = currentNode.next;
    }
  }
}
```

完整的应用程序名为*LinkedListRemoveRedundantPath*。

## 编码挑战 14 - 将最后一个节点移到最前面

**问题**：考虑一个单链表。编写一小段代码，通过两种方法将最后一个节点移到最前面。因此，链表的最后一个节点变为头节点。

**解决方案**：这是一个听起来简单并且确实简单的问题。第一种方法将遵循以下步骤：

1.  将指针移动到倒数第二个节点（我们将其表示为*currentNode*）。

1.  存储*currentNode.next*（我们将其表示为*nextNode* - 这是最后一个节点）。

1.  将`cu`*rrentNode.next*设置为`null`（因此，最后一个节点变为尾部）。

1.  将新的头部设置为存储的节点（因此，头部变为*nextNode*）。

在代码行中，我们有以下内容：

```java
public void moveLastToFront() {      
  Node currentNode = head;
  // step 1
  while (currentNode.next.next != null) {
    currentNode = currentNode.next;
  }
  // step 2
  Node nextNode = currentNode.next;
  // step 3
  currentNode.next = null;
  // step 4
  nextNode.next = head;
  head = nextNode;
}
```

第二种方法可以通过以下步骤执行：

1.  将指针移动到倒数第二个节点（我们将其表示为*currentNode*）。

1.  将链表转换为循环列表（将*currentNode.next.next*链接到头部）。

1.  将新的头部设置为*currentNode.next*。

1.  通过将*currentNode.next*设置为`null`来打破循环性。

在代码行中，我们有以下内容：

```java
public void moveLastToFront() {
  Node currentNode = head;
  // step 1
  while (currentNode.next.next != null) {
    currentNode = currentNode.next;
  }
  // step 2
  currentNode.next.next = head;
  // step 3
  head = currentNode.next;
  // step 4
 currentNode.next = null;
}
```

完整的应用程序名为*LinkedListMoveLastToFront*。

## 编码挑战 15 - 以 k 组反转单链表

**Amazon**，**Google**，**Adobe**，**Microsoft**

**问题**：考虑一个单链表和一个整数*k*。编写一小段代码，以*k*组反转链表的节点。

**解决方案**：假设给定的链表是 7 → 4 → 3 → 1 → 8 → 2 → 9 → 0 → null，*k*=3。结果应为 3 → 4 → 7 → 2 → 8 → 1 → 0 → 9 → null。

让我们考虑给定的*k*等于链表的大小。在这种情况下，我们将问题简化为反转给定的链表。例如，如果给定的列表是 7 → 4 → 3 → null，*k*=3，则结果应为 3 → 4 → 7 → null。那么，我们如何获得这个结果呢？

为了反转节点，我们需要当前节点（*current*）、当前节点旁边的节点（*next*）和当前节点之前的节点（*previous*），并且我们应用以下代表节点重新排列的算法：

1.  从 0 开始计数。

1.  作为*当前*节点（最初是头节点）不是`null`，并且我们还没有达到给定的*k*，发生以下情况：

a. *next*节点（最初为`null`）变为*current*节点旁边的节点（最初是头节点）。

b. *current*节点（最初是头节点）旁边的节点变为*previous*节点（最初为`null`）。

c. *previous*节点变为*current*节点（最初是头节点）。

d. *current*节点变为*next*节点（*步骤 2a*的节点）。

e. 增加计数器。

因此，如果我们应用此算法，我们可以反转整个列表。但是我们需要按组反转它；因此，我们必须解决我们所做的*k*个子问题。如果这对你来说听起来像递归，那么你是对的。在前述算法的末尾，设置为*步骤 2a*（*next*）的节点指向计数器所指向的节点。我们可以说我们已经反转了前*k*个节点。接下来，我们通过递归从*next*节点开始继续下一组*k*节点。以下图表说明了这个想法：

![11.25：以 k 组（k=3）反转列表](img/Figure_11.25_B15403.jpg)

图 11.25 - 以 k 组（k=3）反转列表

以下代码实现了这个想法：

```java
public void reverseInKGroups(int k) {
  if (head != null) {
    head = reverseInKGroups(head, k);
  }
}
private Node reverseInKGroups(Node head, int k) {
  Node current = head;
  Node next = null;
  Node prev = null;
  int counter = 0;
  // reverse first 'k' nodes of linked list
  while (current != null && counter < k) {
    next = current.next;                        
    current.next = prev;            
    prev = current;
    current = next;
    counter++;
  }
  // 'next' points to (k+1)th node            
  if (next != null) {
    head.next = reverseInKGroups(next, k);
  }
  // 'prev' is now the head of the input list 
  return prev;
}
```

这段代码运行时间为 O(n)，其中*n*是给定列表中的节点数。完整的应用程序名为*ReverseLinkedListInGroups*。

## 编码挑战 16 - 反转双向链表

**Microsoft**，**Flipkart**

**问题**：考虑一个双向链表。编写一小段代码来反转它的节点。

**解决方案**：反转双向链表可以利用双向链表维护到前一个节点的链接的事实。这意味着我们可以简单地交换每个节点的前指针和后指针，如下面的代码所示：

```java
public void reverse() {
  Node currentNode = head;
  Node prevNode = null;
  while (currentNode != null) {
    // swap next and prev pointers of the current node
    Node prev = currentNode.prev;
    currentNode.prev = currentNode.next;
    currentNode.next = prev;
    // update the previous node before moving to the next node
    prevNode = currentNode;
    // move to the next node in the doubly linked list            
    currentNode = currentNode.prev;
  }
  // update the head to point to the last node
  if (prevNode != null) {
    head = prevNode;
  }
}
```

完整的应用程序名为*DoublyLinkedListReverse*。要对单链表和双链表进行排序，请参考*第十四章*，*排序和搜索*。

## 编码挑战 17 - LRU 缓存

**Amazon**，**Google**，**Adobe**，**Microsoft**，**Flipkart**

**问题**：编写一小段代码来实现固定大小的 LRU 缓存。LRU 缓存代表最近最少使用的缓存。这意味着，当缓存已满时，添加新条目将指示缓存自动驱逐最近最少使用的条目。

**解决方案**：任何缓存实现必须提供一种快速有效的检索数据的方式。这意味着我们的实现必须遵守以下约束：

+   **固定大小**：缓存必须使用有限的内存。因此，它需要一些限制（例如，固定大小）。

+   **快速访问数据**：插入和搜索操作应该快速；最好是 O(1)复杂度时间。

+   **快速驱逐数据**：当缓存已满（达到其分配的限制）时，缓存应该提供一个有效的算法来驱逐条目。

在最后一个要点的背景下，从 LRU 缓存中驱逐意味着驱逐最近最少使用的数据。为了实现这一点，我们必须跟踪最近使用的条目和长时间未使用的条目。此外，我们必须确保插入和搜索操作的 O(1)复杂度时间。在 Java 中没有内置的数据结构可以直接给我们提供这样的缓存。

但是我们可以从`HashMap`数据结构开始。在 Java 中，`HashMap`允许我们在 O(1)时间内按键插入和搜索（查找）数据。因此，使用`HashMap`解决了问题的一半。另一半，即跟踪最近使用的条目和长时间未使用的条目，无法通过`HashMap`完成。

然而，如果我们想象一个提供快速插入、更新和删除的数据结构，那么我们必须考虑双向链表。基本上，如果我们知道双向链表中节点的地址，那么插入、更新和删除可以在 O(1)时间内完成。

这意味着我们可以提供一个实现，它依赖于`HashMap`和双向链表之间的共生关系。基本上，对于 LRU 缓存中的每个条目（键值对），我们可以在`HashMap`中存储条目的键和关联链表节点的地址，而这个节点将存储条目的值。以下图表是对这一陈述的可视化表示：

![11.26：使用 HashMap 和双向链表的 LRU 缓存](img/Figure_11.26_B15403.jpg)

图 11.26 - 使用 HashMap 和双向链表的 LRU 缓存

但是双向链表如何帮助我们跟踪最近使用的条目呢？秘密在于以下几点：

+   在缓存中插入新条目将导致将相应的节点添加到双向链表的头部（因此，双向链表的头部保存了最近使用的值）。

+   当访问一个条目时，我们将其对应的节点移动到双向链表的头部。

+   当我们需要驱逐一个条目时，我们驱逐双向链表的尾部（因此，双向链表的尾部保存了最近最少使用的值）。

基于这些陈述，我们可以提供以下直接的实现：

```java
public final class LRUCache {
  private final class Node {
    private int key;
    private int value;
    private Node next;
    private Node prev;
  }
  private final Map<Integer, Node> hashmap;
  private Node head;
  private Node tail;
  // 5 is the maximum size of the cache
  private static final int LRU_SIZE = 5;
  public LRUCache() {
    hashmap = new HashMap<>();
  }
  public int getEntry(int key) {
    Node node = hashmap.get(key);
    // if the key already exist then update its usage in cache
    if (node != null) {
      removeNode(node);
      addNode(node);
      return node.value;
    }
    // by convention, data not found is marked as -1
    return -1;
  }
  public void putEntry(int key, int value) {
    Node node = hashmap.get(key);
    // if the key already exist then update 
    // the value and move it to top of the cache                 
    if (node != null) { 
      node.value = value;
      removeNode(node);
      addNode(node);
    } else {
      // this is new key
      Node newNode = new Node();
      newNode.prev = null;
      newNode.next = null;
      newNode.value = value;
      newNode.key = key;
      // if we reached the maximum size of the cache then 
      // we have to remove the  Least Recently Used
      if (hashmap.size() >= LRU_SIZE) { 
        hashmap.remove(tail.key);
        removeNode(tail);
        addNode(newNode);
      } else {
        addNode(newNode);
      }
      hashmap.put(key, newNode);
    }
  }
  // helper method to add a node to the top of the cache
  private void addNode(Node node) {
    node.next = head;
    node.prev = null;
    if (head != null) {
      head.prev = node;
    }
    head = node;
    if (tail == null) {
      tail = head;
    }
  }
  // helper method to remove a node from the cache
  private void removeNode(Node node) {
    if (node.prev != null) {
      node.prev.next = node.next;
    } else {
      head = node.next;
    }
    if (node.next != null) {
      node.next.prev = node.prev;
    } else {
      tail = node.prev;
    }
  }   
}
```

完整的应用程序名为*LRUCache*。

好了，这是本章的最后一个编码挑战。是时候总结本章了！

# 摘要

本章引起了您对涉及链表和映射的最常见问题的注意。在这些问题中，首选涉及单向链表的问题；因此，本章主要关注了这一类编码挑战。

在下一章中，我们将解决与堆栈和队列相关的编码挑战。
