# 理解集合框架

在本章中，我们将深入研究包含接口和类的集合框架。我们将看一下三个主要的集合：`List`、`Set`和`Map`。本章将讨论`List`集合中的`ArrayList`，`Set`集合中的`HashSet`，以及`Map`集合中的`HashMap`和`HashTable`。我们将通过示例来逐个概念进行讨论。

在本章中，我们将涉及以下主题：

+   集合框架

+   列表集合

+   集合

+   映射集合

# 集合框架

Java 集合框架基本上是一组接口和类。为了高效编程或使用 Java 方法的灵活性，Java 设计了一个框架，其中包含不同的类和接口。集合框架有助于高效存储和处理数据。这个框架有几个有用的类，拥有大量有用的函数，使程序员的任务变得非常容易。

我们已经看到了关于数组和多维数组的很多概念。例如，在一个数组中，如果我们想要删除一个新数组集合中的一个索引，我们可以使用集合框架来做到这一点。比如说在一个数组中有 10 个值，我们想要删除第五个值，或者在第五个和第六个值之间插入一个值——在集合框架中有一些灵活的方法。

在接下来的章节中，将讨论这个集合框架中可用的方法类型以及它们如何有效地使用。所以只是给你一个概念，记住集合是一组类和接口。

我们将看一下这个框架提供的集合。

# 列表集合

第一个是`List`集合/接口。列表是一个有序的集合，有时我们也称之为序列。列表可能包含重复的元素，就像数组一样，但数组和`ArrayList`之间有很多不同之处。你可以将多个值插入到这个`List`容器中，它可能也包含重复的元素。你实际上可以从任何索引添加任何值和删除任何值。比如说你按顺序向列表中添加了 15 个元素，现在你想要删除第 6 个元素，或者在第 10 个和第 11 个元素之间插入一个元素，或者想知道在这 15 个元素中某个元素在哪个索引上。在列表容器中有很多有用的 API 来检索元素，而这些在数组中是得不到的。数组只能被初始化；除此之外，你不能对数组执行任何方法，而`ArrayList`你有很多灵活的方法来玩耍。

`List`接口是一个集合，`ArrayList`、`LinkedList`和`vector`是实现这个接口的三个类。这个接口提供了一组方法。它公开了一些方法，而这三个类使用这些方法在它们的类中。

在这三个中，让我们讨论`ArrayList`。这是最著名的之一，大多数 Java 程序员都在使用。一旦你理解了`ArrayList`，你就可以很容易地理解`LinkedLists`和`vector`。在下一节中，我们将创建一个`ArrayList`类，并实现`List`接口中的方法，以查看这些方法在检索或组织数据时有多灵活。当你有一组数据在一个容器中时，你可以很容易地使用`List`接口来组织这些数据。

# ArrayList 类

让我们从`ArrayList`类开始，它实现了`List`接口。创建一个新的类，命名为`arrayListexample`。我们将首先查看`ArrayList`中的方法，然后讨论数组和`ArrayList`之间的区别。

我们首先声明`ArrayList`如下。如果你在 IDE 中悬停在`ArrayList`上，你会看到一个建议，告诉你要导入`java.util`来使用`ArrayList`：

```java
package coreJava;

public class arrayListexample {

    public static void main(String[] args) {

        ArrayList a=new ArrayList();

    }
}
```

一旦你这样做了，它仍然会显示一个关于`ArrayList`的建议，如果你将鼠标悬停在上面，它会建议添加参数类型。要删除这个建议，你可以给`ArrayList`传递一个参数类型，比如`Integer`或`String`：

```java
        ArrayList<String> a=new ArrayList<String>();
        a.add("rahul");
        a.add("java");
```

在传递了参数类型之后，你可以通过使用`a.`轻松地添加一些字符串实例，它会显示出`ArrayList`支持的不同类型的列表。对于`ArrayList`，我们没有定义特定的数组大小，而在数组中，我们已经明确定义了一个大小。在数组中，一旦我们定义了大小，就不能减少或增加大小。但在`ArrayList`中，你可以随时添加或删除元素，它是一个动态大小的数组。这是数组和`ArrayList`之间的基本区别之一。

如果我们想打印这个`ArrayList`，我们可以通过添加以下代码行来简单地实现：

```java
        System.out.println(a);
```

运行时，它打印出`[rahul, java]`。但如果你想以数组的形式打印出来，我们需要写一个`for`循环。我们添加另一个对象，这次我们指定了我们想要字符串放入的索引：

```java
        a.add("rahul");
        a.add("java");
        System.out.println(a);
        a.add(0, "student");
        System.out.println(a);
```

当我们打印这个时，它会给出以下输出：

```java
[rahul, java]
[student, rahul, java]
```

你可以看到在第二行中，`student`被添加到`rahul`之前的列表中，因为我们已经指定了它的索引为`0`。

如果我们想从列表中删除一个条目，可以通过添加以下代码行来实现：

```java
        a.remove(1);
        a.remove("java");
```

第一行代码将从列表中删除位于第一个索引处的条目，而第二行将在列表中查找并删除字符串。如果你想获取特定索引的条目，可以使用`get`方法来做到这一点：

```java
       a.get(2);
```

上一行代码将打印出`java`作为输出，因为它是在索引`2`处的元素。

假设你有一个包含 50 个元素的列表，并且你需要找出该列表中是否存在特定的字符串/整数。如果你使用数组，你将不得不创建一个`for`循环，并找出元素是否存在，但在`ArrayList`中，我们有一个`contains`方法，它可以为我们检查整个列表，并以`true`或`false`的形式给出输出：

```java
        System.out.println(a.contains("java"));
```

这将打印出`true`作为输出，因为该元素存在于我们的列表中；如果你将它改为，例如，`testing`，它将返回值为`false`，因为它不在我们的列表中。

`ArrayList`中还有另一个有用的方法是`indexOf`方法。如果我们想要找到列表中特定元素的索引值，我们可以使用`indexOf`来知道：

```java
        System.out.println(a.indexOf("rahul"))
```

这将返回这个字符串的索引号。

现在，如果我们想要检查数组是否为空，我们可以使用`ArrayList`中的`isEmpty`方法来做到这一点，它将返回值为`true`或`false`：

```java
        System.out.println(a.isEmpty());
```

这将返回值为`false`，因为我们的列表不是空的。

`ArrayList`中最后一个最重要的方法是`size`方法，它返回列表的长度：

```java
        System.out.println(a.size());
```

关于`ArrayList`，你需要知道的另一件事是，实现`List`接口的所有类都可以接受重复的值。我们知道在集合接口中扩展`List`的类有：`ArrayList`、`LinkedList`和`vector`。所有这些类都可以接受重复的值。

# ArrayList 的例子

假设我们有一个包含重复数字的数组，比如`{4, 5, 5, 5, 4, 6, 6, 9, 4}`，我们想打印出这个数组中的唯一数字，以及这个数字在数组中重复的次数。我们的输出应该是"four is repeated three times, five is repeated three times, six twice, nine once."

让我们在这里引入`ArrayList`的概念来解决这个谜题：

```java
package demopack;
import java.util.ArrayList;
public class collectiondemo {
    public static void main(String[] args) { 
        int a[] ={ 4,5,5,5,4,6,6,9,4}; 
        ArrayList<Integer>ab =new ArrayList<Integer>(); 
        for(int i=0;i<a.length;i++) 
        { 
            int k=0; 
            if(!ab.contains(a[i])) 
            { 
                ab.add(a[i]); 
                k++; 
                for(int j=i+1;j<a.length;j++) 
                { 
                    if(a[i]==a[j]) 
                    { 
                       k++; 
                    } 
                } 
                System.out.println(a[i]); 
                System.out.println(k); 
                if(k==1) 
                    System.out.println(a[i]+"is unique number"); 
            } 
        } 
    }
}
ArrayList with the ab object type. Then we create a for loop, and within it we use an if loop with !ab.contains to check whether the element is present within the loop. We need another for loop within this if loop to iterate through the remaining part of the array. The if loop within this for loop will work as a counter for us to increment the number of times a number is repeated in the array.
```

我们已经完成了`for`和`if`循环。我们打印出数组中的每个元素以及每个元素在数组中出现的次数。要打印出唯一的数字，也就是在数组中不重复的数字，我们使用一个`if`循环并打印它。

就这个例子而言就是这样；你可以尝试用你自己的逻辑编写这个例子。

# 集合 Set

Java 中还有一个重要的集合是`Set`集合/接口。`HashSet`、`TreeSet`和`LinkedHashSet`是实现`Set`接口的三个类。`Set`和`List`之间的主要区别是`Set`不接受重复的值。`Set`和`List`接口之间的另一个区别是没有保证元素按顺序存储。

在本节中，我们主要讨论`HashSet`。我们将以一个示例类来尝试理解这个概念。为本节创建一个名为`hashSetexample`的类，并在类中创建一个对象来使用`HashSet`；它会建议你添加参数类型，在我们的情况下是`String`：

```java
package coreJava;

import java.util.HashSet;

public class hashSetexample {

    public static void main(String[] args) {

       HashSet<String> hs= new HashSet<String>();

    }
}
```

在你的 IDE 中，当你输入`hs.`时，它会显示`HashSet`提供的所有方法：

![](img/43eefee7-fe95-435d-a7c3-e83ec95f6ce2.png)

首先添加一些重复条目的字符串实例：

```java
        HashSet<String hs= new HashSet<String>();
        hs.add("USA");
        hs.add("UK");
        hs.add("INDIA");
        hs.add("INDIA");
        System.out.println(hs);
```

当你打印这个时，输出将如下所示：

```java
[USA, UK, INDIA]
```

我们看到`HashSet`拒绝了`INDIA`的重复条目，我们只看到一个实例。

如果我们希望删除任何对象，我们可以使用`remove`方法，要获取列表的大小，请使用`size`方法：

```java
        System.out.println(hs.remove("UK"));
        System.out.println(hs.isEmpty());
        System.out.println(hs.size());
```

`isEmpty`方法告诉我们列表是否为空——如果为空，它将返回`true`，否则返回`false`。

# 使用迭代器

为了遍历列表中的每个元素，我们使用`iterator`方法。我们需要为这个`Iterator`类创建另一个对象，以及`String`参数类型：

```java
        Iterator<String> i=hs.iterator();
```

假设我们有一组元素，它们按顺序从零、一、二等开始。`iterator`遍历每个元素，从零开始打印每个值。我们创建了一个迭代器对象，并打印了以下值：

```java
        System.out.println(i.next());
        System.out.println(i.next());
```

`i.next()`的第一个实例将打印出零索引处的值，下一个`i.next()`实例打印出索引一处的值。如果我们有一个包含大约 100 个值的集合，我们将不得不使用`while`循环：

```java
        while(i.hasNext())
        {
            System.out.println(i.next());
        }
```

在这里，我们使用了`hasNext`方法，它检查下一个值是否存在。如果下一个索引中存在值，它将返回`true`，如果没有，它将返回`false`。在我们的情况下，它将返回 100 个值的`true`，之后返回`false`，并退出`while`循环。

这就是你如何使用`iterator`遍历`Set`接口中的对象。如果你正在进行自动化测试，比如 Selenium，你会经常使用这个`while`循环。

# 映射集合

我们还有一个叫做`Map`的集合。我们将以一个例子讨论`Map`，并随着代码的进行进行讨论。这个接口以键和值对的形式接受值。

我们创建一个类，`hashMapexample`，在其中定义`HashMap`。`HashMap`需要两种类型的参数，比如`Integer`和`String`：

```java
package coreJava;

import java.util.HashMap;

public class hashMapexample {

    public static void main(String[] args) {

       HashMap<Integer, String> hm= new HashSet<Integer, String>();

    }
}
```

这里，`Integer`是键，`String`是值。现在，如果你在 IDE 中输入`hm.`，你会看到`HashMap`中存在的一些方法；让我们使用`put`方法：

```java
        hm.put(0, "hello");
        hm.put(1, "goodbye");
        hm.put(2, "morning");
        hm.put(3, "evening");
```

`put`方法以键和值的形式接受输入。此外，键的值需要是整数，也可以是字符串。键只是我们为值定义的东西。我们可以使用`remove`方法删除一个值：

```java
        hm.remove(2);
```

`HashMap`中的`entrySet`方法以集合索引的形式存储每个键和值：

```java
        Set sn= hm.entrySet();
```

我们现在将这个`HashMap`转换成一个集合。为了遍历这个集合的每个索引，我们使用`iterator`，就像在前一节中一样，我们使用`while`循环：

```java
        Iterator it= sn.iterator();

        while(it.hasNext())
        {
            Map.Entry mp=(Map.Entry)it.next();
            System.out.println(mp.getKey());
            System.out.println(mp.getValues());
        }
```

在这里，我们需要使用`Map.Entry`，因为每个索引中的元素都包括一个键和一个值，`Map.Entry`帮助我们分离键和值。当你打印这个`while`循环时，你应该得到以下输出：

```java
0
hello
1
goodbye
2
morning
3
evening
```

不使用`Map.Entry`，它会抛出一个错误。这就是`HashMap`的工作原理。

# 哈希表

还有一个集合，叫做`HashTable`，但它与`HashMap`沿着同样的线路。你只需要把`HashMap`改成`HashTable`就可以了。不过`HashMap`和`HashTable`之间有一点小区别。

`HashMap`和`HashTable`之间的区别如下：

+   同步或线程安全

+   空键和空值

+   遍历值

# 同步或线程安全

这是两者之间最重要的区别。`HashMap`是非同步的，不是线程安全的。那么什么是非同步？这意味着如果多个程序同时访问`HashMap`，它会不断更新。现在假设有五个线程在操作`HashMap`。这意味着五个不同的程序或线程可以同时访问`HashMap`，这意味着没有同步。但是在`HashTable`中，如果一个程序正在访问`HashTable`，另一个程序需要等待，直到第一个程序释放`HashTable`资源。这是主要的区别。另一方面，`HashTable`是线程安全和同步的。什么时候应该使用`HashMap`？如果你的应用程序不需要多线程任务，换句话说，`HashMap`对于非线程应用程序更好。`HashTable`应该在多线程应用程序中使用。

# 空键和空值

`HashMap`允许一个空键和任意数量的空值，而`HashTable`不允许`HashTable`对象中的空键和空值。假设你正在将员工记录输入到数据库中，也许在上传员工详细信息到数据库时，你可能觉得你不知道他们的电话号码，但你在一个键值中输入了名为电话号码的字段，并且索引值暂时为空；你可以稍后更新它。这在`HashMap`中可以工作，但当你使用`HashTable`时，它不允许任何空键和空值。如果你觉得你想让你的程序非常安全，并且你想阻止多个线程同时访问它，那么你应该选择`HashTable`。`HashTable`是线程安全的，直到一个程序在`HashTable`上完成操作之前，它不会释放其对象给另一个程序。

# 遍历值

`HashMap`对象的值通过`iterator`进行迭代。`HashTable`是除了向量之外唯一使用枚举器来迭代`HashTable`对象的类。

除了我们刚刚描述的三个区别之外，`HashMap`和`HashTable`的操作是相同的。

# 总结

在本章中，我们看了集合框架和三种类型的集合：`List`，`Set`和`Map`。我们在`List`集合中探索了`ArrayList`，并且也探索了`ArrayList`的一个例子。`Set`集合与`ArrayList`不同——主要的区别是`Set`不接受重复的值。在最后一个集合中，也就是`Map`集合中，我们看到了两种类型，`HashMap`和`HashTable`，以及它们之间的区别。
