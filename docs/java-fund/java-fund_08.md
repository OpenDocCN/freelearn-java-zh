# *第八章*

# Java 中的高级数据结构

## 学习目标

在本课结束时，您将能够：

+   实现一个链表

+   实现二叉搜索树

+   使用枚举更好地处理常量

+   解释 HashSet 中唯一性背后的逻辑

## 介绍

在之前的课程中，您学习了 Java 中各种数据结构，如列表、集合和映射。您还学习了如何在许多不同的方式上迭代这些数据结构，比较对象；以及如何以高效的方式对这些集合进行排序。

在本课中，您将学习高级数据结构的实现细节，如链表和二叉搜索树。随着我们的进展，您还将了解一个称为枚举的强大概念，并探索如何有效地使用它们而不是常量。在课程结束时，您将了解`equals()`和`hashCode()`背后的魔力和神秘。

## 实现自定义链表

列表有两种实现方式：

+   **ArrayList**：这是使用数组作为底层数据结构实现的。它具有与数组相同的限制。

+   **链表**：链表中的元素分布在内存中，与数组不同，数组中的元素是连续的。

### ArrayList 的缺点

ArrayList 的缺点如下：

+   虽然 ArrayList 是动态的，创建时不需要指定大小。但是由于数组的大小是固定的，因此当向列表添加更多元素时，ArrayList 通常需要隐式调整大小。调整大小遵循创建新数组并将先前数组的所有元素添加到新数组的过程。

+   在 ArrayList 的末尾插入新元素通常比在中间添加要快，但是当在列表中间添加元素时，代价很高，因为必须为新元素创建空间，并且为了创建空间，现有元素必须移动。

+   删除 ArrayList 的最后一个元素通常更快，但是当在中间删除元素时，代价很高，因为元素必须进行调整，将元素向左移动。

### 链表优于数组的优点

以下是链表优于数组的优点：

+   动态大小，大小不固定，没有调整大小的问题。每个节点都持有对下一个节点的引用。

+   在链表中随机位置添加和删除元素，与向量和数组相比要简单得多。

在本主题中，您将学习如何为特定目的构建自定义链表。通过这样做，我们将欣赏链表的强大之处，并了解实现细节。

这是链表的图示表示：

![图 8.1：链表的表示](img/C09581_09_01.jpg)

###### 图 8.1：链表的表示

动态内存分配是链表的一个常见应用。链表的其他应用包括实现数据结构，如栈、各种队列的实现、图、树等。

### 练习 31：向链表添加元素

让我们创建一个简单的链表，允许我们添加整数，并打印列表中的元素：

1.  创建一个名为`SimpleIntLinkedList`的类如下：

```java
public class SimpleIntLinkedList 
{
```

1.  创建另一个代表链表中每个元素的`Node`类。每个节点都有数据（一个整数值）需要保存；它将有一个对下一个`Node`的引用。实现数据和`next`变量的 getter 和 setter：

```java
static class Node {
Integer data;
Node next;
Node(Integer d) {
data = d;
next = null;
}
Node getNext() {
return next;
}
void setNext(Node node) {
next = node;
}
Object getData() {
return data;
}
}
```

1.  实现`add(Object item)`方法，以便可以将任何项目/对象添加到此列表中。通过传递`newItem = new Node(item)`项目构造一个新的`Node`对象。从`head`节点开始，向列表的末尾移动，访问每个节点。在最后一个节点中，将下一个节点设置为我们新创建的节点（`newItem`）。通过调用`incrementIndex()`来增加索引以跟踪索引：

```java
// appends the specified element to the end of this list.
    public void add(Integer element) {
        // create a new node
        Node newNode = new Node(element);
        //if head node is empty, create a new node and assign it to Head
        //increment index and return
        if (head == null) {
            head = newNode;
            return;
        }
        Node currentNode = head;

        while (currentNode.getNext() != null) {
                currentNode = currentNode.getNext();
        }
        // set the new node as next node of current
        currentNode.setNext(newNode);
    }
```

1.  实现一个 toString()方法来表示这个对象。从头节点开始，迭代所有节点直到找到最后一个节点。在每次迭代中，构造存储在每个节点中的整数的字符串表示。表示将类似于这样：[Input1,Input2,Input3]

```java
  public String toString() {
    String delim = ",";
    StringBuffer stringBuf = new StringBuffer();
    if (head == null)
      return "LINKED LIST is empty";
    Node currentNode = head;
    while (currentNode != null) {
      stringBuf.append(currentNode.getData());
      currentNode = currentNode.getNext();
      if (currentNode != null)
        stringBuf.append(delim);
      }
    return stringBuf.toString();
  }
```

1.  为 SimpleIntLinkedList 创建一个类型为 Node 的成员属性（指向头节点）。在 main 方法中，创建一个 SimpleIntLinkedList 对象，并依次添加五个整数（13, 39, 41, 93, 98）到其中。打印 SimpleIntLinkedList 对象。

```java
Node head;
public static void main(String[] args) {
  SimpleLinkedList list = new SimpleLinkedList();
  list.add(13);
  list.add(39);
  list.add(41);
  list.add(93);
  list.add(98);
  System.out.println(list);
  }
}
```

输出将如下所示：

```java
[13, 39, 41, 93, 98]
```

### 活动 32：在 Java 中创建自定义链表

在我们的练习中，我们创建了一个可以接受整数值的链表。作为一个活动，让我们创建一个自定义链表，可以将任何对象放入其中，并显示添加到列表中的所有元素。此外，让我们添加另外两种方法来从链表中获取和删除值。

这些步骤将帮助您完成此活动：

1.  创建一个名为 SimpleObjLinkedList 的类，并创建一个类型为 Node 的成员属性（指向头节点）。添加一个类型为 int 的成员属性（指向节点中的当前索引或位置）

1.  创建一个表示链表中每个元素的 Node 类。每个节点将有一个需要保存的对象，并且它将有对下一个节点的引用。LinkedList 类将有一个对头节点的引用，并且可以使用 Node.getNext()来遍历到下一个节点。因为头是第一个元素，我们可以通过在当前节点中移动 next 来遍历到下一个元素。这样，我们可以遍历到列表的最后一个元素。

1.  实现 add(Object item)方法，以便可以向该列表添加任何项目/对象。通过传递 newItem = new Node(item)项目来构造一个新的 Node 对象。从头节点开始，爬行到列表的末尾。在最后一个节点中，将 next 节点设置为我们新创建的节点(newItem)。增加索引。

1.  实现 get(Integer index)方法，根据索引从列表中检索项目。索引不能小于 0。编写逻辑来爬行到指定的索引并识别节点并从节点返回值。

1.  实现 remove(Integer index)方法，根据索引从列表中删除项目。编写逻辑来爬行到指定索引的前一个节点并识别节点。在此节点中，将下一个设置为 getNext()。如果找到并删除元素，则返回 true。如果未找到元素，则返回 false。

1.  实现一个 toString()方法来表示这个对象。从头节点开始，迭代所有节点直到找到最后一个节点。在每次迭代中，构造存储在每个节点中的对象的字符串表示。

1.  编写一个 main 方法，创建一个 SimpleObjLinkedList 对象，并依次添加五个字符串（"INPUT-1"，"INPUT-2"，"INPUT-3"，"INPUT-4"，"INPUT-5"）到其中。打印 SimpleObjLinkedList 对象。在 main 方法中，使用 get(2)从列表中获取项目并打印检索到的项目的值，还从列表中删除项目 remove(2)并打印列表的值。列表中应该已经删除了一个元素。

输出将如下所示：

```java
[INPUT-1 ,INPUT-2 ,INPUT-3 ,INPUT-4 ,INPUT-5 ]
INPUT-3
[INPUT-1 ,INPUT-2 ,INPUT-3 ,INPUT-5 ]
```

#### 注意

此活动的解决方案可以在第 356 页找到。

### 链表的缺点

链表的缺点如下：

+   访问元素的唯一方法是从第一个元素开始，然后顺序移动；无法随机访问元素。

+   搜索速度慢。

+   链表需要额外的内存空间。

## 实现二叉搜索树

我们在第 7 课中已经简要介绍了树，Java 集合框架和泛型，让我们看看树的一种特殊实现，称为二叉搜索树（BSTs）。

要理解 BSTs，让我们看看什么是二叉树。树中每个节点最多有两个子节点的树是**二叉树**。

BST 是二叉树的一种特殊实现，其中左子节点始终小于或等于父节点，右子节点始终大于或等于父节点。二叉搜索树的这种独特结构使得更容易添加、删除和搜索树的元素。以下图表表示了 BST：

![图 8.2：二叉搜索树的表示](img/C09581_09_02.jpg)

###### 图 8.2：二叉搜索树的表示

二叉搜索树的应用如下：

+   实现字典。

+   在数据库中实现多级索引。

+   实现搜索算法。

### 练习 32：在 Java 中创建二叉搜索树

在这个练习中，我们将创建一个二叉搜索树并实现左右遍历。

1.  在其中创建一个`BinarySearchTree`类，其中包含一个`Node`类。`Node`类应该有两个指向其左节点和右节点的元素。

```java
//Public class holding the functions of Entire Binary Tree structure
public class BinarySearchTree
{
    private Node parent;
    private int  data;
    private int  size = 0;
    public BinarySearchTree() {
        parent = new Node(data);
    }
private class Node {
        Node left; //points to left node
        Node right; //points to right node
        int  data;
        //constructor of Node
        public Node(int data) {
            this.data = data;
        }
}
```

1.  我们将创建一个`add(int data)`函数，它将检查父节点是否为空。如果为空，它将将值添加到父节点。如果父节点有数据，我们需要创建一个新的`Node(data)`并找到正确的节点（根据 BST 规则）将此新节点附加到。

为了帮助找到正确的节点，已经实现了一个方法`add(Node root, Node newNode)`，使用递归逻辑深入查找实际应该属于这个新节点的节点。

根据 BST 规则，如果根数据大于`newNode`数据，则`newNode`必须添加到左节点。再次递归检查是否有子节点，并且 BST 的相同逻辑适用，直到达到叶节点以添加值。如果根数据小于`newNode`数据，则`newNode`必须添加到右节点。再次递归检查是否有子节点，并且 BST 的相同逻辑适用，直到达到叶节点以添加值：

```java
/**
* This is the method exposed as public for adding elements into the Tree.
     * it checks if the size == 0 and then adds the element into parent node. if
     * parent is already filled, creates a New Node with data and calls the
     * add(parent, newNode) to find the right root and add it to it.
     * @param data
     */
  public void add(int data) {
    if (size == 0) {
      parent.data = data;
      size++;
    } else {
      add(parent, new Node(data));
    }
  }
/**
 * Takes two params, root node and newNode. As per BST, check if the root
 * data is > newNode data if true: newNode has to be added in left Node
 * (again recursively check if it has child nodes and the same logic of BST
 * until it reaches the leaf node to add value) else: newNode has to be
 * added in right (again recursively check if it has child nodes and the
 * same logic of BST until it reaches the leaf node to add value)
* 
 * @param root
 * @param newNode
 */
  private void add(Node root, Node newNode) {
    if (root == null) {
      return;
    }
  if (newNode.data < root.data) {
      if (root.left == null) {
        root.left = newNode;
        size++;
      } else {
        add(root.left, newNode);
      }
    }
    if ((newNode.data > root.data)) {
      if (root.right == null) {
        root.right = newNode;
        size++;
      } else {
        add(root.right, newNode);
      }
    }
  }
```

1.  创建一个`traverseLeft()`函数来遍历并打印 BST 根节点左侧的所有值：

```java
  public void traverseLeft() {
  Node current = parent;
  System.out.print("Traverse the BST From Left : ");
        while (current.left != null && current.right != null) {
            System.out.print(current.data + "->[" + current.left.data + " " + current.right.data + "] ");
            current = current.left;
        }
        System.out.println("Done");
    }
```

1.  创建一个`traverseRight()`函数来遍历并打印 BST 根节点右侧的所有值：

```java
    public void traverseRight() {
        Node current = parent;
        System.out.print("Traverse the BST From Right");
        while (current.left != null && current.right != null) {
            System.out.print(current.data + "->[" + current.left.data + " " + current.right.data + "] ");
            current = current.right;
        }
        System.out.println("Done");
    }
```

1.  让我们创建一个示例程序来测试 BST 的功能：

```java
    /**
     * Main program to demonstrate the BST functionality.
     * - Adding nodes
     * - finding High and low 
     * - Traversing left and right
     * @param args
     */
    public static void main(String args[]) {
        BinarySearchTree bst = new BinarySearchTree();
        // adding nodes into the BST
        bst.add(32);
        bst.add(50);
        bst.add(93);
        bst.add(3);
        bst.add(40);
        bst.add(17);
        bst.add(30);
        bst.add(38);
        bst.add(25);
        bst.add(78);
        bst.add(10);
        bst.traverseLeft();
        bst.traverseRight();
}
    }
```

输出如下：

```java
Traverse the BST From Left : 32->[3 50] Done
Traverse the BST From Right32->[3 50] 50->[40 93] Done
```

### 活动 33：在 BinarySearchTree 类中实现查找 BST 中最高和最低值的方法

1.  创建一个实现`while`循环的`getLow()`方法，以迭代检查父节点是否有左子节点，并将左侧 BST 中没有左子节点的节点作为最低值返回。

1.  创建一个实现`while`循环的`getHigh()`方法，以迭代检查父节点是否有右子节点，并将右侧 BST 中没有右子节点的节点作为最高值返回。

1.  在`main`方法中，使用之前实现的`add`方法向二叉搜索树添加元素，并调用`getLow()`和`getHigh()`方法来识别最高和最低值。

输出将如下所示：

```java
Lowest value in BST :3
Highest value in BST :93
```

#### 注意

此活动的解决方案可以在第 360 页找到。

## 枚举

Java 中的枚举（或枚举）是 Java 中的一种特殊类型，其字段由常量组成。它用于强制编译时安全性。

例如，考虑一周的天数，它们是一组固定的常量，因此我们可以定义一个枚举：

```java
public enum DayofWeek { 
 SUNDAY, MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY  
} 
```

现在我们可以简单地检查存储一天的变量是否是声明的枚举的一部分。我们还可以为非通用常量声明枚举，例如：

```java
public enum Jobs { 
  DEVELOPER, TESTER, TEAM LEAD, PROJECT MANAGER 
}
```

这将强制将作业类型设置为`Jobs`枚举中声明的常量。这是一个持有货币的示例枚举：

```java
public enum Currency {
    USD, INR, DIRHAM, DINAR, RIYAL, ASD 
}
```

### 练习 33：使用枚举存储方向

我们将创建一个枚举并找到值并比较枚举。

1.  创建一个类`EnumExample`，并在`main`方法中。使用值作为枚举获取并打印枚举。使用值作为字符串获取并打印枚举：

```java
public class EnumExample
{
    public static void main(String[] args)
    {
        Direction north = Direction.NORTH;
        System.out.println(north + " : " + north.no);
        Direction south = Direction.valueOf("SOUTH");
        System.out.println(south + " : " + south.no);
    }
}
```

1.  让我们创建一个枚举，其中包含具有表示方向的整数值：

```java
public enum Direction
    {
                  EAST(45), WEST(90), NORTH(180), SOUTH(360);
            int no;

Direction(int i){
                no =i;
            }
    }
```

输出如下：

```java
NORTH : 180
SOUTH : 360
```

### 活动 34：使用枚举保存学院部门详情

让我们构建一个完整的枚举来保存学院部门及其编号（BE（“工程学士”，100））。

执行以下步骤：

1.  使用`enum`关键字创建`DeptEnum`枚举。添加两个私有属性（String `deptName`和 int `deptNo`）来保存枚举中的值。

1.  。重写一个构造函数以接受缩写和`deptNo`并将其放入成员变量中。添加符合构造函数的枚举常量。

1.  添加`deptName`和`deptNo`的 getter 方法。

1.  让我们编写一个`main`方法和示例程序来演示枚举的使用：

输出如下：

```java
BACHELOR OF ENGINEERING : 1
BACHELOR OF ENGINEERING : 1
BACHELOR OF COMMERCE : 2
BACHELOR OF SCIENCE : 3
BACHELOR OF ARCHITECTURE : 4
BACHELOR : 0
true
```

#### 注意

这项活动的解决方案可以在第 362 页找到。

### 活动 35：实现反向查找

编写一个应用程序，接受一个值

1.  创建一个枚举`App`，声明常量 BE、BCOM、BSC 和 BARC，以及它们的全称和部门编号。

1.  还声明两个私有变量`accronym`和`deptNo`。

1.  创建一个带有缩写和`deptNo`的参数化构造函数，并将它们分配给作为参数传递的变量。

1.  声明一个公共方法`getAccronym（）`，返回变量`accronym`，以及一个公共方法`getDeptNo（）`，返回变量`deptNo`。

1.  实现反向查找，接受课程名称，并在`App`枚举中搜索相应的缩写。

1.  实现`main`方法，并运行程序。

你的输出应该类似于：

```java
BACHELOR OF SCIENCE : 3
BSC
```

#### 注意

这项活动的解决方案可以在第 363 页找到。

## 集合和集合中的唯一性

在这个主题中，我们将学习集合背后找到正在添加的对象的唯一性的逻辑，并理解两个对象级方法的重要性。

魔术在于`Object`类的两个方法

+   `hashCode（）`

+   `equals（）`

### equals（）和 hashCode（）方法的基本规则

+   只有当使用`hashcode（）`方法返回的值相同并且`equal（）`方法返回 true 时，两个对象才能相同。

+   如果两个对象返回相同的`hashCode（）`值，并不一定意味着两个对象相同（因为哈希值也可能与其他对象发生冲突）。在这种情况下，需要调用`equals（）`并验证身份来找到相等性。

+   我们不能仅仅使用`hashCode（）`来找到相等性；我们需要同时使用`equals（）`来做到这一点。然而，仅仅使用`hashCode（）`就足以找到不相等性。如果`hashCode（）`返回不同的值，可以肯定这些对象是不同的。

### 向集合添加对象

尽管当我们将一个对象添加到集合中时会发生许多事情，但我们只会关注与我们的研究主题相关的细节：

+   该方法首先调用该对象的`hashCode（）`方法并获取`hashCode`，然后`Set`将其与其他对象的`hashCode`进行比较，并检查是否有任何对象匹配该`hashCode`。

+   如果集合中没有任何对象与添加对象的`hashCode`匹配，那么我们可以百分之百地确定没有其他对象具有相同的身份。新添加的对象将安全地添加到集合中（无需检查`equals（）`）。

+   如果任何对象与添加的对象的`hashCode`匹配，这意味着可能添加了相同的对象（因为`hashCode`可能对于两个不同的对象是相同的）。在这种情况下，为了确认怀疑，它将使用`equals（）`方法来查看对象是否真的相等。如果相等，则新添加的对象将不被拒绝，否则新添加的对象将被拒绝。

### 练习 34：了解 equals（）和 hashCode（）的行为

让我们创建一个新的类，并在实现`equals（）`和`hashCode（）`之前了解`Set`的行为：

1.  创建一个带有三个属性的 Student 类：`Name`（`String`），`Age`（`int`）和`Year of passing`（`int`）。还为这些私有成员创建 getter 和 setter：

```java
/**
 * Sample Class student containing attributes name, age and yearOfPassing
 *
 */
import java.util.HashSet;
class Student {
    private String name;
    private Integer age;
    private Integer yearOfPassing;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    public int getYearOfPassing() {
        return yearOfPassing;
    }
    public void setYearOfPassing(int releaseYr) {
        this.yearOfPassing = releaseYr;
    }
}
```

1.  编写一个示例类`HashCodeExample`，以演示集合的行为。在主方法中，创建三个具有不同名称和其他详细信息的`Students`对象（Raymonds，Allen 和 Maggy）：

```java
/**
 * Example class demonstrating the set behavior
 * We will create 3 objects and add into the Set
 * Later will create a new object resembling same as one of the 3 objects created and added into the set
*/
public class HashCodeExample {
    public static void main(String[] args) {
        Student m = new Student();
        m.setName("RAYMONDS");
        m.setAge(20);
        m.setYearOfPassing(2011);
        Student m1 = new Student();
        m1.setName("ALLEN");
        m1.setAge(19);
        m1.setYearOfPassing(2010);
        Student m2 = new Student();
        m2.setName("MAGGY");
        m2.setAge(18);
        m2.setYearOfPassing(2012);
}
}
```

1.  创建一个`HashSet`来保存这些学生对象（`set`）。一个接一个地将三个对象添加到`HashSet`中。然后，打印`HashSet`中的值：

```java
    HashSet<Student> set = new HashSet<Student>();
        set.add(m);
        set.add(m1);
        set.add(m2);
        //printing all the elements of Set
System.out.println("Before Adding ALLEN for second time : ");
        for (Student mm : set) {
            System.out.println(mm.getName() + " " + mm.getAge());
        }
```

1.  在`main`方法中，创建另一个类似于已创建的三个对象的`Student`对象（例如：让我们创建一个类似于 Allen 的学生）。将这个新创建的`Student`对象添加到已经`添加（set）`了三个学生的`HashSet`中。然后，打印`HashSet`中的值。您会注意到 Allen 已经被添加到集合中两次（这意味着集合中未处理重复项）：

```java
    //creating a student similar to m1 (name:ALLEN, age:19, yearOfPassing:2010)
        Student m3 = new Student();
        m3.setName("ALLEN");
        m3.setAge(19);
        m3.setYearOfPassing(2010);
//this Student will be added as hashCode() and equals() are not implemented
        set.add(m3);
        // 2 students with same details (ALLEN 19 will be noticed twice)
System.out.println("After Adding ALLEN for second time: ");
        for (Student mm : set) {
            System.out.println(mm.getName() + " " + mm.getAge());
        }
```

输出如下：

```java
Before Adding ALLEN for second time : 
RAYMONDS 20
MAGGY 18
ALLEN 19
After Adding ALLEN for second time: 
RAYMONDS 20
ALLEN 19
MAGGY 18
ALLEN 19
```

`Allen`确实已经被添加到集合中两次（这意味着集合中尚未处理重复项）。这需要在`Student`类中处理。

### 练习 35：重写 equals()和 hashCode()

让我们重写`Student`的`equals()`和`hashCode()`，看看这之后`Set`的行为如何改变：

1.  在`Students`类中，让我们通过检查`Student`对象的每个属性（`name`，`age`和`yearOfPassing`同等重要）来重写`equals()`方法。`Object`级别的`equals()`方法以`Object`作为参数。要重写该方法，我们需要提供逻辑，用于比较自身属性（`this`）和`object o`参数。这里的相等逻辑是，只有当他们的`name`，`age`和`yearOfPassing`相同时，两个学生才被认为是相同的：

```java
    @Override
    public boolean equals(Object o) {
        Student m = (Student) o;
        return m.name.equals(this.name) && 
                m.age.equals(this.age) && 
                m.yearOfPassing.equals(this.yearOfPassing);
    }
```

1.  在`Student`类中，让我们重写`hashCode()`方法。基本要求是对于相同的对象应该返回相同的整数。实现`hashCode`的一种简单方法是获取对象中每个属性的`hashCode`并将其相加。其背后的原理是，如果`name`，`age`或`yearOfPassing`不同，那么`hashCode`将返回不同的值，这将表明没有两个对象是相同的：

```java
@Override
    public int hashCode() {
        return this.name.hashCode() + 
                this.age.hashCode() + 
                this.yearOfPassing.hashCode();
    }
```

1.  让我们运行`HashCodeExample`的主方法，以演示在`Student`对象中重写`equals()`和`hashCode()`之后集合的行为。

```java
public class HashCodeExample {
    public static void main(String[] args) {
        Student m = new Student();
        m.setName("RAYMONDS");
        m.setAge(20);
        m.setYearOfPassing(2011);
        Student m1 = new Student();
        m1.setName("ALLEN");
        m1.setAge(19);
        m1.setYearOfPassing(2010);
        Student m2 = new Student();
        m2.setName("MAGGY");
        m2.setAge(18);
        m2.setYearOfPassing(2012);

        Set<Student> set = new HashSet<Student>();
        set.add(m);
        set.add(m1);
        set.add(m2);

        //printing all the elements of Set
System.out.println("Before Adding ALLEN for second time : ");
        for (Student mm : set) {
            System.out.println(mm.getName() + " " + mm.getAge());
        }
    //creating a student similar to m1 (name:ALLEN, age:19, yearOfPassing:2010)
        Student m3 = new Student();
        m3.setName("ALLEN");
        m3.setAge(19);
        m3.setYearOfPassing(2010);
//this element will not be added if hashCode and equals methods are implemented
        set.add(m3);
System.out.println("After Adding ALLEN for second time: ");
        for (Student mm : set) {
            System.out.println(mm.getName() + " " + mm.getAge());
        }

    }
}
```

输出如下：

```java
Before Adding ALLEN for second time: 
ALLEN 19
RAYMONDS 20
MAGGY 18
After Adding ALLEN for second time: 
ALLEN 19
RAYMONDS 20
MAGGY 18
```

在添加`hashCode()`和`equals()`之后，我们的`HashSet`有智能识别和删除重复项的能力。

如果我们不重写`equals()`和`hashCode()`，JVM 在内存中创建对象时为每个对象分配一个唯一的哈希码值，如果开发人员不重写`hashcode`方法，那么就无法保证两个对象返回相同的哈希码值。

## 总结

在这节课中，我们学习了 BST 是什么，以及在 Java 中实现 BST 的基本功能的步骤。我们还学习了一种遍历 BST 向右和向左的技巧。我们看了枚举在常量上的用法，并了解了它们解决的问题类型。我们还建立了自己的枚举，并编写了代码来获取和比较枚举的值。

我们还学习了`HashSet`如何识别重复项，并看了重写`equals()`和`hashCode()`的重要性。此外，我们学会了如何正确实现`equals()`和`hashCode()`。
