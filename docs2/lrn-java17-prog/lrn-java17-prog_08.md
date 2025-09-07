# *第六章*：数据结构、泛型和常用工具

本章介绍了 Java 集合框架及其三个主要接口 `List`、`Set` 和 `Map`，包括泛型的讨论和演示。在 Java 集合的上下文中还讨论了 `equals()` 和 `hashCode()` 方法。管理数组、对象和时间/日期值的实用工具类也有相应的专用章节。学习完本章后，您将能够在您的程序中使用所有主要的数据结构。

本章将涵盖以下主题：

+   `List`、`Set` 和 `Map` 接口

+   `Collections` 工具

+   `Arrays` 工具

+   `Object` 工具

+   `java.time` 包

让我们开始吧！

# 技术要求

要能够执行本章提供的代码示例，您需要以下内容：

+   一台装有 Microsoft Windows、Apple macOS 或 Linux 操作系统的计算机

+   Java **标准版**（**SE**）版本 17 或更高版本

+   一个 **集成开发环境**（**IDE**）或您偏好的代码编辑器

如何设置 Java SE 和 IntelliJ IDEA 编辑器的说明提供在本书的 *第一章* 中，*Java 17 入门*。本章的代码示例文件可在 GitHub 的 [`github.com/PacktPublishing/Learn-Java-17-Programming.git`](https://github.com/PacktPublishing/Learn-Java-17-Programming.git) 仓库中的 `examples/src/main/java/com/packt/learnjava/ch06_collections` 文件夹中找到。

# 列表、集合和映射接口

`short`、`int` 或 `double`。如果您需要存储此类类型值，元素必须是相应的包装类型，例如 `Short`、`Integer` 或 `Double`。

Java 集合支持存储和访问集合元素的多种算法：一个有序列表，一个唯一集合，一个字典（称为 `java.util` 包的 `java.util` 包包含以下内容：

+   扩展 `Collection` 接口的接口：`List`、`Set` 和 `Queue`，最常见的是

+   实现先前列出的接口的类：`ArrayList`、`HashSet`、`Stack`、`LinkedList` 和一些其他类

+   `Map` 接口及其 `ConcurrentMap` 和 `SortedMap` 子接口，例如

+   实现 `Map` 相关接口的类：`HashMap`、`HashTable` 和 `TreeMap`，例如最常用的三个

审查 `java.util` 包中的所有类和接口需要一本专门的书籍。因此，在本节中，我们将简要概述三个主要接口——`List`、`Set` 和 `Map`——以及每个接口的一个实现类——`ArrayList`、`HashSet` 和 `HashMap`。我们首先介绍 `List` 和 `Set` 接口共有的方法。`List` 和 `Set` 之间的主要区别是 `Set` 不允许元素重复。另一个区别是 `List` 保留元素的顺序，并允许对它们进行排序。

要在集合中识别一个元素，使用`equals()`方法。为了提高性能，实现`Set`接口的类通常也会使用`hashCode()`方法。这有助于快速计算具有相同哈希值的每个元素的整数（称为`equals()`方法）。这种方法比逐个比较集合中的每个元素要快。

正因如此，我们经常看到类的名字前面有一个`hash`前缀，表示该类使用哈希值，因此元素必须实现`hashCode()`方法。在这样做的时候，你必须确保它被实现，以便每次`equals()`方法返回`true`时，这两个对象通过`hashCode()`方法返回的哈希值也相等。否则，使用哈希值描述的算法将不会工作。

最后，在讨论`java.util`接口之前，先说几句关于泛型的话。

## 泛型

你最常见到这些泛型声明，如下所示：

```java
List<String> list = new ArrayList<String>();
```

```java
Set<Integer> set = new HashSet<Integer>();
```

在前面的例子中，`<>`被称为**菱形**，如下面的代码片段所示：

```java
List<String> list = new ArrayList<>();
```

```java
Set<Integer> set = new HashSet<>();
```

泛型通知编译器集合元素的预期类型。这样，编译器可以检查程序员尝试添加到声明的集合中的元素是否为兼容类型。例如，观察以下内容：

```java
List<String> list = new ArrayList<>();
```

```java
list.add("abc");
```

```java
list.add(42);   //compilation error
```

这有助于避免运行时错误。它还向程序员（因为当程序员编写代码时，IDE 会编译代码）提示可能对集合元素进行的操作。

我们还将看到这些其他类型的泛型：

+   `<? extends T>`表示类型是`T`或`T`的子类，其中`T`是作为集合泛型的类型。

+   `<? super T>`表示类型`T`或其任何基类（父类），其中`T`是作为集合泛型的类型。

有了这些，让我们从如何创建实现`List`或`Set`接口的类的对象开始——或者说，初始化`List`或`Set`类型的变量。为了演示这两个接口的方法，我们将使用两个类：`ArrayList`（实现`List`）和`HashSet`（实现`Set`）。

## 如何初始化 List 和 Set

自 Java 9 以来，`List`或`Set`接口有了静态的`of()`工厂方法，可以用来初始化一个集合，如下所述：

+   `of()`：返回一个空集合。

+   `of(E... e)`: 返回一个包含在调用期间传入的元素数量的集合。它们可以作为逗号分隔的列表或数组传入。

这里有一些例子：

```java
//Collection<String> coll 
```

```java
//        = List.of("s1", null); //does not allow null
```

```java
Collection<String> coll = List.of("s1", "s1", "s2");
```

```java
//coll.add("s3");         //does not allow add element
```

```java
//coll.remove("s1");   //does not allow remove element
```

```java
//((List<String>) coll).set(1, "s3");    
```

```java
                       //does not allow modify element
```

```java
System.out.println(coll);       //prints: [s1, s1, s2]
```

```java
//coll = Set.of("s3", "s3", "s4");     
```

```java
                            //does not allow duplicate
```

```java
//coll = Set.of("s2", "s3", null);     
```

```java
                                 //does not allow null
```

```java
coll = Set.of("s3", "s4");
```

```java
System.out.println(coll);  
```

```java
                        //prints: [s3, s4] or [s4, s3]
```

```java
//coll.add("s5");         //does not allow add element
```

```java
//coll.remove("s2");   //does not allow remove element
```

如您所预期的那样，`Set` 类型的工厂方法不允许重复元素，因此我们已将该行注释掉（否则，前面的示例会在该行停止运行）。但您可能不会想到，您不能有 `null` 元素，并且在使用 `of()` 方法之一初始化集合后，您不能添加/删除/修改集合的元素。这就是为什么我们注释掉了前面示例中的一些行。如果您需要在集合初始化后添加元素，您必须使用构造函数或其他创建可修改集合的实用工具来初始化它（我们很快将看到一个 `Arrays.asList()` 的示例）。

`Collection` 接口为实现了 `Collection` 接口（`List` 和 `Set` 的父接口）的对象提供了两种添加元素的方法，其形式如下：

+   `boolean add(E e)`：这尝试将提供的元素 `e` 添加到集合中；如果成功，则返回 `true`，如果无法完成（例如，当该元素已存在于 `Set` 接口中），则返回 `false`。

+   `boolean addAll(Collection<? extends E> c)`：这尝试将提供的集合中的所有元素添加到集合中；如果至少添加了一个元素，则返回 `true`，如果无法将元素添加到集合中（例如，当提供的集合 `c` 中的所有元素已存在于 `Set` 接口中），则返回 `false`。

下面是一个使用 `add()` 方法的示例：

```java
List<String> list1 = new ArrayList<>();
```

```java
list1.add("s1");
```

```java
list1.add("s1");
```

```java
System.out.println(list1);     //prints: [s1, s1]
```

```java
Set<String> set1 = new HashSet<>();
```

```java
set1.add("s1");
```

```java
set1.add("s1");
```

```java
System.out.println(set1);      //prints: [s1]
```

下面是使用 `addAll()` 方法的示例：

```java
List<String> list1 = new ArrayList<>();
```

```java
list1.add("s1");
```

```java
list1.add("s1");
```

```java
System.out.println(list1);      //prints: [s1, s1]
```

```java
List<String> list2 = new ArrayList<>();
```

```java
list2.addAll(list1);
```

```java
System.out.println(list2);      //prints: [s1, s1]
```

```java
Set<String> set = new HashSet<>();
```

```java
set.addAll(list1);
```

```java
System.out.println(set);        //prints: [s1]
```

下面是 `add()` 和 `addAll()` 方法功能的示例：

```java
List<String> list1 = new ArrayList<>();
```

```java
list1.add("s1");
```

```java
list1.add("s1");
```

```java
System.out.println(list1);     //prints: [s1, s1]
```

```java
List<String> list2 = new ArrayList<>();
```

```java
list2.addAll(list1);
```

```java
System.out.println(list2);      //prints: [s1, s1]
```

```java
Set<String> set = new HashSet<>();
```

```java
set.addAll(list1);
```

```java
System.out.println(set);        //prints: [s1]
```

```java
Set<String> set1 = new HashSet<>();
```

```java
set1.add("s1");
```

```java
Set<String> set2 = new HashSet<>();
```

```java
set2.add("s1");
```

```java
set2.add("s2");
```

```java
System.out.println(set1.addAll(set2)); //prints: true
```

```java
System.out.println(set1);              //prints: [s1, s2]
```

注意，在前面的代码片段中的最后一个示例中，`set1.addAll(set2)` 方法返回 `true`，尽管并非所有元素都被添加。要查看 `add()` 和 `addAll()` 方法返回 `false` 的情况，请看以下示例：

```java
Set<String> set = new HashSet<>();
```

```java
System.out.println(set.add("s1"));   //prints: true
```

```java
System.out.println(set.add("s1"));   //prints: false
```

```java
System.out.println(set);             //prints: [s1]
```

```java
Set<String> set1 = new HashSet<>();
```

```java
set1.add("s1");
```

```java
set1.add("s2");
```

```java
Set<String> set2 = new HashSet<>();
```

```java
set2.add("s1");
```

```java
set2.add("s2");
```

```java
System.out.println(set1.addAll(set2)); //prints: false
```

```java
System.out.println(set1);              //prints: [s1, s2]
```

`ArrayList` 和 `HashSet` 类也有接受集合的构造函数，如下面的代码片段所示：

```java
Collection<String> list1 = List.of("s1", "s1", "s2");
```

```java
System.out.println(list1);      //prints: [s1, s1, s2]
```

```java
List<String> list2 = new ArrayList<>(list1);
```

```java
System.out.println(list2);      //prints: [s1, s1, s2]
```

```java
Set<String> set = new HashSet<>(list1);
```

```java
System.out.println(set);        //prints: [s1, s2]
```

```java
List<String> list3 = new ArrayList<>(set);
```

```java
System.out.println(list3);      //prints: [s1, s2]
```

现在，在我们学习了如何初始化集合之后，我们可以转向 `List` 和 `Set` 接口中的其他方法。

## `java.lang.Iterable` 接口

`Collection` 接口扩展了 `java.lang.Iterable` 接口，这意味着实现 `Collection` 接口（无论是直接还是间接）的类也实现了 `java.lang.Iterable` 接口。`Iterable` 接口中只有三个方法，如下所述：

+   `Iterator<T> iterator()`：这返回一个实现了 `java.util.Iterator` 接口的类的对象；它允许集合在 `FOR` 语句中使用，如下面的示例所示：

    ```java
    Iterable<String> list = List.of("s1", "s2", "s3");
    System.out.println(list);       //prints: [s1, s2, s3]
    for(String e: list){
        System.out.print(e + " ");  //prints: s1 s2 s3
    }
    ```

+   `default void forEach (Consumer<? super T> function)`: 这个方法将提供的`Consumer`类型的函数应用于集合中的每个元素，直到所有元素都被处理或函数抛出异常。我们将在*第十三章*中讨论什么是函数，*函数式编程*；现在，我们在这里只提供一个示例：

    ```java
    Iterable<String> list = List.of("s1", "s2", "s3");
    System.out.println(list);   //prints: [s1, s2, s3]
    list.forEach(e -> System.out.print(e + " "));
                                //prints: s1 s2 s3
    ```

+   `default Spliterator<T> splititerator()`: 这个方法返回一个实现`java.util.Spliterator`接口的类的对象；主要用于实现允许并行处理的方法，并且超出了本书的范围。

## 集合接口

如我们之前提到的，`List`和`Set`接口扩展了`Collection`接口，这意味着`List`和`Set`继承了`Collection`接口的所有方法。这些方法在此列出：

+   `boolean add(E e)`: 这个方法尝试将一个元素添加到集合中。

+   `boolean addAll(Collection<? extends E> c)`: 这个方法尝试添加提供的集合中的所有元素。

+   `boolean equals(Object o)`: 这个方法将集合与提供的`o`对象进行比较。如果提供的对象不是集合，则此对象返回`false`；否则，它比较集合的组成与提供的集合（作为`o`对象）的组成。在`List`的情况下，它还比较元素的顺序。以下是一些示例，以说明这一点：

    ```java
    Collection<String> list1 = List.of("s1", "s2", "s3");
    System.out.println(list1);     //prints: [s1, s2, s3]
    Collection<String> list2 = List.of("s1", "s2", "s3");
    System.out.println(list2);     //prints: [s1, s2, s3]
    System.out.println(list1.equals(list2));  
                                          //prints: true
    Collection<String> list3 = List.of("s2", "s1", "s3");
    System.out.println(list3);     //prints: [s2, s1, s3]
    System.out.println(list1.equals(list3));  
                                          //prints: false
    Collection<String> set1 = Set.of("s1", "s2", "s3");
    System.out.println(set1);   
                //prints: [s2, s3, s1] or different order
    Collection<String> set2 = Set.of("s2", "s1", "s3");
    System.out.println(set2);   
                //prints: [s2, s1, s3] or different order
    System.out.println(set1.equals(set2));  
                                           //prints: true
    Collection<String> set3 = Set.of("s4", "s1", "s3");
    System.out.println(set3);   
                //prints: [s4, s1, s3] or different order
    System.out.println(set1.equals(set3));  
                                          //prints: false
    ```

+   `int hashCode()`: 这个方法返回集合的哈希值；在集合是需要实现`hashCode()`方法实现的集合的元素的情况下使用。

+   `boolean isEmpty()`: 这个方法返回`true`，如果集合没有任何元素。

+   `int size()`: 这个方法返回集合中元素的数量；当`isEmpty()`方法返回`true`时，此方法返回`0`。

+   `void clear()`: 这个方法从集合中删除所有元素；在此方法调用之后，`isEmpty()`方法返回`true`，而`size()`方法返回`0`。

+   `boolean contains(Object o)`: 如果集合包含提供的`o`对象，则此方法返回`true`。为了使此方法正确工作，集合中的每个元素和提供的对象都必须实现`equals()`方法，在`Set`的情况下，应该实现`hashCode()`方法。

+   `boolean containsAll(Collection<?> c)`: 如果集合包含提供的集合中的所有元素，则此方法返回`true`。为了使此方法正确工作，集合中的每个元素和提供的集合中的每个元素都必须实现`equals()`方法，在`Set`的情况下，应该实现`hashCode()`方法。

+   `boolean remove(Object o)`: 这尝试从该集合中删除指定的元素，并在元素存在时返回`true`。为了正确执行此方法，集合的每个元素和提供的对象都必须实现`equals()`方法，在`Set`的情况下，应实现`hashCode()`方法。

+   `boolean removeAll(Collection<?> c)`: 这尝试从集合中删除提供的集合的所有元素；类似于`addAll()`方法，如果至少删除了一个元素，则此方法返回`true`；否则返回`false`。为了正确执行此方法，集合的每个元素和提供的集合的每个元素都必须实现`equals()`方法，在`Set`的情况下，应实现`hashCode()`方法。

+   `default boolean removeIf(Predicate<? super E> filter)`: 这尝试从集合中删除所有满足给定谓词的元素；这是我们将在*第十三章*，*函数式编程*中描述的函数。如果至少删除了一个元素，则返回`true`。

+   `boolean retainAll(Collection<?> c)`: 这尝试在集合中仅保留提供的集合中包含的元素。类似于`addAll()`方法，如果至少保留了一个元素，则此方法返回`true`；否则返回`false`。为了正确执行此方法，集合的每个元素和提供的集合的每个元素都必须实现`equals()`方法，在`Set`的情况下，应实现`hashCode()`方法。

+   `Object[] toArray()`，`T[] toArray(T[] a)`: 这将集合转换为数组。

+   `default T[] toArray(IntFunction<T[]> generator)`: 这使用提供的函数将集合转换为数组。我们将在*第十三章*，*函数式编程*中解释函数。

+   `default Stream<E> stream()`: 这返回一个`Stream`对象（我们将在*第十四章*，*Java 标准流*中讨论流）。

+   `default Stream<E> parallelStream()`: 这返回一个可能并行的`Stream`对象（我们将在*第十四章*，*Java 标准流*中讨论流）。

## `List`接口

`List`接口有几个不属于其任何父接口的其他方法，如下所述：

+   静态工厂`of()`方法，在*如何初始化 List 和 Set*子节中描述。

+   `void add(int index, E element)`: 这将在列表中提供的位置插入提供的元素。

+   `static List<E> copyOf(Collection<E> coll)`: 这返回一个包含给定`Collection`接口元素的不可修改的`List`接口，并保留它们的顺序。以下代码片段演示了此方法的功能：

    ```java
    Collection<String> list = List.of("s1", "s2", "s3");
    System.out.println(list);    //prints: [s1, s2, s3]
    List<String> list1 = List.copyOf(list);
    //list1.add("s4");                //run-time error
    //list1.set(1, "s5");             //run-time error
    //list1.remove("s1");             //run-time error
    Set<String> set = new HashSet<>();
    System.out.println(set.add("s1"));
    System.out.println(set);          //prints: [s1]
    Set<String> set1 = Set.copyOf(set);
    //set1.add("s2");                 //run-time error
    //set1.remove("s1");              //run-time error
    Set<String> set2 = Set.copyOf(list);
    System.out.println(set2);    //prints: [s1, s2, s3] 
    ```

+   `E get(int index)`: 这返回列表中指定位置处的元素。

+   `List<E> subList(int fromIndex, int toIndex)`: 提取 `fromIndex`（包含）和 `toIndex`（不包含）之间的子列表。

+   `int indexOf(Object o)`: 这返回列表中指定元素的第一个索引（位置）；列表中的第一个元素具有索引（位置）`0`。

+   `int lastIndexOf(Object o)`: 这返回列表中指定元素的最后索引（位置）；列表中的最后一个元素具有 `list.size() - 1` 的索引位置。

+   `E remove(int index)`: 这将删除列表中指定位置处的元素；它返回被删除的元素。

+   `E set(int index, E element)`: 这将替换列表中指定位置处的元素；它返回被替换的元素。

+   `default void replaceAll(UnaryOperator<E> operator)`: 这通过将提供的函数应用于每个元素来转换列表。`UnaryOperator` 函数将在 *第十三章*，*函数式编程* 中描述。

+   `ListIterator<E> listIterator()`: 返回一个 `ListIterator` 对象，允许列表向后遍历。

+   `ListIterator<E> listIterator(int index)`: 返回一个 `ListIterator` 对象，允许从提供的位置开始遍历子列表。例如，观察以下内容：

    ```java
    List<String> list = List.of("s1", "s2", "s3");
    ListIterator<String> li = list.listIterator();
    while(li.hasNext()){
        System.out.print(li.next() + " ");     
                                  //prints: s1 s2 s3
    }
    while(li.hasPrevious()){
        System.out.print(li.previous() + " ");  
                                   //prints: s3 s2 s1
    }
    ListIterator<String> li1 = list.listIterator(1);
    while(li1.hasNext()){
        System.out.print(li1.next() + " ");       
                                      //prints: s2 s3
    }
    ListIterator<String> li2 = list.listIterator(1);
    while(li2.hasPrevious()){
        System.out.print(li2.previous() + " ");   
                                          //prints: s1
    }
    ```

+   `default void sort(Comparator<? super E> c)`: 这根据提供的 `Comparator` 接口生成的顺序对列表进行排序。例如，观察以下内容：

    ```java
    List<String> list = new ArrayList<>();
    list.add("S2");
    list.add("s3");
    list.add("s1");
    System.out.println(list);     //prints: [S2, s3, s1]
    list.sort(String.CASE_INSENSITIVE_ORDER);
    System.out.println(list);     //prints: [s1, S2, s3]
    //list.add(null);      //causes NullPointerException
    list.sort(Comparator.naturalOrder());
    System.out.println(list);     //prints: [S2, s1, s3]
    list.sort(Comparator.reverseOrder());
    System.out.println(list);     //prints: [s3, s1, S2]
    list.add(null);
    list.sort(Comparator.nullsFirst(Comparator
                                       .naturalOrder()));
    System.out.println(list);  
                            //prints: [null, S2, s1, s3]
    list.sort(Comparator.nullsLast(Comparator
                                      .naturalOrder()));
    System.out.println(list);         
                            //prints: [S2, s1, s3, null]
    Comparator<String> comparator = 
         (s1, s2) -> s1 == null ? -1 : s1.compareTo(s2);
    list.sort(comparator);
    System.out.println(list);         
                             //prints: [null, S2, s1, s3]
    Comparator<String> comparator = (s1, s2) -> 
     s1 == null ? -1 : s1.compareTo(s2);
    list.sort(comparator);
    System.out.println(list);    
                                 //prints: [null, S2, s1, s3]
    ```

主要有两种方式对列表进行排序，如下所示：

+   使用 `Comparable` 接口实现（称为 **自然顺序**）

+   使用 `Comparator` 接口实现

`Comparable` 接口只有一个 `compareTo()` 方法。在上面的示例中，我们根据 `String` 类中的 `Comparable` 接口实现实现了 `Comparator` 接口。如您所见，此实现提供了与 `Comparator.nullsFirst(Comparator.naturalOrder())` 相同的排序顺序。这种实现方式称为 **函数式编程**，我们将在 *第十三章*，*函数式编程* 中更详细地讨论。

## `Set` 接口

`Set` 接口有以下不属于其任何父接口的方法：

+   静态 `of()` 工厂方法，在 *如何初始化列表和集合* 子部分中描述。

+   `static Set<E> copyOf(Collection<E> coll)` 方法：此方法返回一个包含给定 `Collection` 元素的不可修改的 `Set` 接口；它的工作方式与在 *列表接口* 部分描述的 `static <E> List<E> copyOf(Collection<E> coll)` 方法相同。

## `Map` 接口

`Map` 接口有许多与 `List` 和 `Set` 方法类似的方法，如下所示：

+   `int size()`

+   `void clear()`

+   `int hashCode()`

+   `boolean isEmpty()`

+   `boolean equals(Object o)`

+   `default void forEach(BiConsumer<K,V> action)`

+   静态工厂方法：`of()`、`of(K, V v)`、`of(K k1, V v1, K k2, V v2)` 以及许多其他方法

然而，`Map` 接口并没有扩展 `Iterable`、`Collection` 或其他任何接口。它被设计成能够存储 `Entry`，这是 `Map` 的内部接口。`value` 和 `key` 对象都必须实现 `equals()` 方法。`key` 对象还必须实现 `hashCode()` 方法。

`Map` 接口中的许多方法与 `List` 和 `Set` 接口中的签名和功能完全相同，因此我们在这里不再重复。我们只将遍历 `Map` 特定的方法，如下所示：

+   `V get(Object key)`: 根据提供的键检索值；如果没有这样的键，则返回 `null`。

+   `Set<K> keySet()`: 这将检索地图中的所有键。

+   `Collection<V> values()`: 这将检索地图中的所有值。

+   `boolean containsKey(Object key)`: 如果提供的键存在于地图中，则返回 `true`。

+   `boolean containsValue(Object value)`: 如果提供的值存在于地图中，则返回 `true`。

+   `V put(K key, V value)`: 这将值及其键添加到地图中；它返回与相同键存储的先前值。

+   `void putAll(Map<K,V> m)`: 这将从提供的地图中复制所有键值对。

+   `default V putIfAbsent(K key, V value)`: 如果地图尚未使用提供的键，则存储提供的值并将其映射到提供的键。它返回映射到提供的键的值——现有的或新的。

+   `V remove(Object key)`: 从地图中删除键和值；如果没有这样的键或值是 `null`，则返回值或 `null`。

+   `default boolean remove(Object key, Object value)`: 如果地图中存在这样的键值对，则从地图中删除键值对。

+   `default V replace(K key, V value)`: 如果提供的键当前映射到提供的值，则替换该值。如果替换了旧值，则返回旧值；否则返回 `null`。

+   `default boolean replace(K key, V oldValue, V newValue)`: 如果提供的键当前映射到 `oldValue` 值，则将 `oldValue` 值替换为提供的 `newValue` 值。如果替换了 `oldValue` 值，则返回 `true`；否则返回 `false`。

+   `default void replaceAll(BiFunction<K,V,V> function)`: 这将提供的函数应用于地图中的每个键值对，并用结果替换它，如果不可能这样做，则抛出异常。

+   `Set<Map.Entry<K,V>> entrySet()`: 这返回一个包含所有键值对的 `Map.Entry` 对象的集合。

+   `default V getOrDefault(Object key, V defaultValue)`: 这返回映射到提供的键的值，或者如果没有提供键，则返回 `defaultValue` 值。

+   `static Map.Entry<K,V> entry(K key, V value)`: 这返回一个包含提供的 `key` 对象和 `value` 对象的不可修改的 `Map.Entry` 对象。

+   `static Map<K,V> copy(Map<K,V> map):` 这将提供的 `Map` 接口转换为不可修改的一个。

以下 `Map` 方法对于本书的范围来说过于复杂，所以我们只是为了完整性而提及它们。它们允许在 `Map` 接口中将多个值组合或计算并聚合到单个现有值中，或者创建一个新的值：

+   `default V merge(K key, V value, BiFunction<V,V,V> remappingFunction)`: 如果提供的键值对存在且值不是 `null`，则使用提供的函数来计算一个新的值；如果新计算出的值是 `null`，则删除键值对。如果提供的键值对不存在或值是 `null`，则提供的非 `null` 值替换当前的值。此方法可用于聚合多个值；例如，它可以用于连接以下字符串值：`map.merge(key, value, String::concat)`。我们将在 *第十三章*，*函数式编程* 中解释 `String::concat` 的含义。

+   `default V compute(K key, BiFunction<K,V,V> remappingFunction)`: 使用提供的函数计算一个新的值。

+   `default V computeIfAbsent(K key, Function<K,V> mappingFunction)`: 仅当提供的键尚未与一个值相关联，或者该值是 `null` 时，使用提供的函数计算一个新的值。

+   `default V computeIfPresent(K key, BiFunction<K,V,V> remappingFunction)`: 仅当提供的键已经与一个值相关联且该值不是 `null` 时，使用提供的函数计算一个新的值。

这最后一批 *计算* 和 *合并* 方法很少使用。最流行的方法无疑是 `V put(K key, V value)` 和 `V get(Object key)` 方法，它们允许使用主要的 `Map` 函数来存储键值对，并使用键来检索值。`Set<K> keySet()` 方法通常用于遍历映射的键值对，尽管 `entrySet()` 方法似乎是一个更自然的方式来完成这个任务。以下是一个例子：

```java
Map<Integer, String> map = Map.of(1, "s1", 2, "s2", 3, "s3");
```

```java
for(Integer key: map.keySet()){
```

```java
    System.out.print(key + ", " + map.get(key) + ", ");  
```

```java
                                 //prints: 3, s3, 2, s2, 1, s1,
```

```java
}
```

```java
for(Map.Entry e: map.entrySet()){
```

```java
    System.out.print(e.getKey() + ", " + e.getValue() + ", "); 
```

```java
                                 //prints: 2, s2, 3, s3, 1, s1,
```

```java
}
```

上述代码示例中的第一个 `for` 循环使用了一种更广泛的方式来访问映射的键值对，通过遍历键来实现。第二个 `for` 循环遍历了条目集合，这在我们的观点中是一个更自然的方式来完成这个任务。请注意，打印出来的值并不与我们放入映射中的顺序相同。这是因为，从 Java 9 开始，不可修改的集合（即 `of()` 工厂方法产生的）已经向 `Set` 元素的顺序中添加了随机化，这改变了不同代码执行之间元素的顺序。这样的设计是为了确保程序员不会依赖于 `Set` 元素的特定顺序，这对于集合来说是不保证的。

## 不可修改的集合

请注意，`of()` 工厂方法产生的集合曾经被称为 `Person1` 类，如下所示：

```java
class Person1 {
```

```java
    private int age;
```

```java
    private String name;
```

```java
    public Person1(int age, String name) {
```

```java
        this.age = age;
```

```java
        this.name = name == null ? "" : name;
```

```java
    }
```

```java
    public void setName(String name){ this.name = name; }
```

```java
    @Override
```

```java
    public String toString() {
```

```java
        return "Person{age=" + age +
```

```java
                ", name=" + name + "}";
```

```java
    }
```

```java
}
```

在下面的代码片段中，为了简单起见，我们将创建一个只有一个元素的列表，然后尝试修改该元素：

```java
Person1 p1 = new Person1(45, "Bill");
```

```java
List<Person1> list = List.of(p1);
```

```java
//list.add(new Person1(22, "Bob"));
```

```java
                         //UnsupportedOperationException
```

```java
System.out.println(list);    
```

```java
                    //prints: [Person{age=45, name=Bill}]
```

```java
p1.setName("Kelly");       
```

```java
System.out.println(list);    
```

```java
                   //prints: [Person{age=45, name=Kelly}]
```

如您所见，尽管无法向由 `of()` 工厂方法创建的列表中添加元素，但如果存在指向该元素的引用，则其元素仍然可以被修改。

# `Collections` 工具类

有两个处理集合的静态方法类非常流行且有用，如下所示：

+   `java.util.Collections`

+   `org.apache.commons.collections4.CollectionUtils`

这些方法是静态的，意味着它们不依赖于对象状态，因此也被称为 **无状态方法** 或 **实用方法**。

## `java.util.Collections` 类

`Collections` 类中的许多方法用于管理集合、分析、排序和比较它们。它们有超过 70 个，所以我们没有机会讨论所有这些。相反，我们将查看主流应用程序开发者最常使用的方法，如下所示：

+   `static copy(List<T> dest, List<T> src)`: 这个方法将 `src` 列表的元素复制到 `dest` 列表中，并保留元素在列表中的顺序和位置。`dest` 列表的大小必须等于或大于 `src` 列表的大小，否则会抛出运行时异常。以下是这个方法使用的一个示例：

    ```java
    List<String> list1 = Arrays.asList("s1","s2");
    List<String> list2 = Arrays.asList("s3", "s4", "s5");
    Collections.copy(list2, list1);
    System.out.println(list2);    //prints: [s1, s2, s5]
    ```

+   `static void sort(List<T> list)`: 这个方法根据每个元素实现的 `compareTo(T)` 方法（称为 `Comparable` 接口（要求实现 `compareTo(T)` 方法）对列表进行排序。在下面的示例中，我们使用 `List<String>`，因为 `String` 类实现了 `Comparable`：

    ```java
    //List<String> list = 
             //List.of("a", "X", "10", "20", "1", "2");
    List<String> list = 
         Arrays.asList("a", "X", "10", "20", "1", "2");
    Collections.sort(list);
    System.out.println(list);      
                         //prints: [1, 10, 2, 20, X, a]
    ```

注意，我们无法使用 `List.of()` 方法创建列表，因为列表将是不可修改的，其顺序无法更改。此外，看看结果顺序：数字排在前面，然后是大写字母，接着是小写字母。这是因为 `String` 类中的 `compareTo()` 方法使用字符的代码点来建立顺序。以下是一个演示此功能的代码示例：

```java
List<String> list = 
           Arrays.asList("a", "X", "10", "20", "1", "2");
Collections.sort(list);
System.out.println(list);  //prints: [1, 10, 2, 20, X, a]
list.forEach(s -> {
    for(int i = 0; i < s.length(); i++){
       System.out.print(" " + 
                            Character.codePointAt(s, i));
    }
    if(!s.equals("a")) {
       System.out.print(","); 
                   //prints: 49, 49 48, 50, 50 48, 88, 97
    }
});
```

如您所见，顺序是由组成字符串的字符的代码点值定义的。

+   `static void sort(List<T> list, Comparator<T> comparator)`: 这个方法根据提供的 `Comparator` 对象对列表进行排序，无论列表元素是否实现了 `Comparable` 接口。例如，让我们按照以下方式对一个由 `Person` 类对象组成的列表进行排序：

    ```java
    class Person  {
        private int age;
        private String name;
        public Person(int age, String name) {
            this.age = age;
            this.name = name == null ? "" : name;
        }
        public int getAge() { return this.age; }
        public String getName() { return this.name; }
        @Override
        public String toString() {
            return "Person{name=" + name + 
                           ", age=" + age + "}";
        }
    }
    ```

+   以下是用于对 `Person` 对象列表进行排序的 `Comparator` 类：

    ```java
    class ComparePersons implements Comparator<Person> {
        public int compare(Person p1, Person p2){
            int result = p1.getName().compareTo(p2.getName());
            if (result != 0) { return result; }
            return p1.age - p2.getAge();
        }
    }
    ```

现在，我们可以使用 `Person` 和 `ComparePersons` 类，如下所示：

```java
List<Person> persons = 
```

```java
      Arrays.asList(new Person(23, "Jack"),
```

```java
                    new Person(30, "Bob"), 
```

```java
                    new Person(15, "Bob"));
```

```java
Collections.sort(persons, new ComparePersons());
```

```java
System.out.println(persons);    
```

```java
                //prints: [Person{name=Bob, age=15}, 
```

```java
                //         Person{name=Bob, age=30}, 
```

```java
                //         Person{name=Jack, age=23}]
```

如我们之前提到的，`Collections` 类中还有许多其他实用工具，所以我们建议您至少查阅一次相关文档，并了解其所有功能。

## `CollectionUtils` 类

在*Apache Commons*项目中，`org.apache.commons.collections4.CollectionUtils`类包含静态无状态方法，这些方法补充了`java.util.Collections`类的方法。它们有助于搜索、处理和比较 Java 集合。

要使用这个类，您需要在 Maven 的`pom.xml`配置文件中添加以下依赖项：

```java
 <dependency>
```

```java
    <groupId>org.apache.commons</groupId>
```

```java
    <artifactId>commons-collections4</artifactId>
```

```java
    <version>4.4</version>
```

```java
 </dependency>
```

这个类中有许多方法，并且可能随着时间的推移添加更多方法。这些实用程序是在`Collections`方法之外创建的，因此它们更加复杂和微妙，不适合本书的范围。为了给您一个关于`CollectionUtils`类中可用方法的概述，以下是按功能分组的方法的简要描述：

+   从集合中检索元素的方法

+   向集合中添加元素或一组元素的方法

+   将`Iterable`元素合并到集合中的方法

+   根据条件移除或保留元素的方法

+   比较两个集合的方法

+   转换集合的方法

+   从集合中选择并过滤的方法

+   生成两个集合的并集、交集或差集的方法

+   创建不可变空集合的方法

+   检查集合大小和空的方法

+   反转数组的方法

最后一个方法可能属于处理数组的实用程序类，这正是我们现在要讨论的。

# 数组实用程序

有两个类提供了处理集合的静态方法，它们非常流行且非常有用，如下所示：

+   `java.util.Arrays`

+   `org.apache.commons.lang3.ArrayUtils`

我们将简要回顾每一个。

## java.util.Arrays 类

我们已经多次使用了`java.util.Arrays`类。它是数组管理的主要实用程序类。这个实用程序类曾经因为`asList(T...a)`方法而非常流行。这是创建和初始化集合最紧凑的方式，如下代码片段所示：

```java
List<String> list = Arrays.asList("s0", "s1");
```

```java
Set<String> set = new HashSet<>(Arrays.asList("s0", "s1");
```

这仍然是一种创建可修改列表的流行方式——我们也使用了它。然而，在引入了`List.of()`工厂方法之后，`Arrays`类的使用大幅下降。

然而，如果您需要管理数组，那么`Arrays`类可能非常有帮助。它包含超过 160 个方法，其中大多数方法都通过不同的参数和数组类型进行了重载。如果我们按方法名称分组，将会有 21 组，如果我们进一步按功能分组，以下 10 组将涵盖所有`Arrays`类的功能：

+   `asList()`: 根据提供的数组或逗号分隔的参数列表创建一个`ArrayList`对象。

+   `binarySearch()`：根据索引范围搜索数组或其指定部分。

+   `compare()`、`mismatch()`、`equals()`和`deepEquals()`：这些方法比较两个数组或它们的元素（根据索引范围）。

+   `copyOf()` 和 `copyOfRange()`: 这复制所有数组或仅复制指定（根据索引范围）的部分。

+   `hashcode()` 和 `deepHashCode()`: 这基于提供的数组生成哈希码值。

+   `toString()` 和 `deepToString()`: 这创建数组的 `String` 表示形式。

+   `fill()`, `setAll()`, `parallelPrefix()`, 和 `parallelSetAll()`: 这为数组或指定范围内的数组元素设置一个值（固定或由提供的函数生成）。

+   `sort()` 和 `parallelSort()`: 这对数组或其一部分（根据索引范围指定）的元素进行排序。

+   `splititerator()`: 这返回一个 `Splititerator` 对象，用于并行处理数组或其一部分（根据索引范围指定）。

+   `stream()`: 这生成数组元素或其中一些元素的流（根据索引范围指定）；参见 *第十四章*，*Java 标准流*。

所有这些方法都很有用，但我们想引起您的注意，特别是 `equals(a1, a2)` 和 `deepEquals(a1, a2)` 方法。它们在数组比较方面特别有用，因为数组对象不能实现自定义的 `equals()` 方法，而是使用 `Object` 类的实现（仅比较引用）。`equals(a1,` `a2)` 和 `deepEquals(a1, a2)` 方法允许比较不仅 `a1` 和 `a2` 引用，而且还使用 `equals()` 方法来比较元素。以下是一个代码示例，演示这些方法的工作原理：

```java
String[] arr1 = {"s1", "s2"};
```

```java
String[] arr2 = {"s1", "s2"};
```

```java
System.out.println(arr1.equals(arr2));   //prints: false
```

```java
System.out.println(Arrays.equals(arr1, arr2));     
```

```java
                                         //prints: true
```

```java
System.out.println(Arrays.deepEquals(arr1, arr2));  
```

```java
                                         //prints: true
```

```java
String[][] arr3 = {{"s1", "s2"}};
```

```java
String[][] arr4 = {{"s1", "s2"}};
```

```java
System.out.println(arr3.equals(arr4));   //prints: false
```

```java
System.out.println(Arrays.equals(arr3, arr4));     
```

```java
                                         //prints: false
```

```java
System.out.println(Arrays.deepEquals(arr3, arr4));
```

```java
                                         //prints: true
```

如您所见，`Arrays.deepEquals()` 在比较两个数组时，每当一个数组中的每个元素等于另一个数组中相同位置的元素时，都会返回 `true`，而 `Arrays.equals()` 方法执行相同的操作，但仅适用于 **一维** （**1D**）数组。

## ArrayUtils 类

`org.apache.commons.lang3.ArrayUtils` 类通过向数组管理工具添加新方法，并处理在可能抛出 `NullPointerException` 的情况下能够处理 `null`，从而补充了 `java.util.Arrays` 类。要使用此类，您需要将以下依赖项添加到 Maven 的 `pom.xml` 配置文件中：

```java
<dependency>
```

```java
   <groupId>org.apache.commons</groupId>
```

```java
   <artifactId>commons-lang3</artifactId>
```

```java
   <version>3.12.0</version>
```

```java
</dependency>
```

`ArrayUtils` 类大约有 300 个重载方法，可以归纳为以下 12 组：

+   `add()`, `addAll()`, 和 `insert()`: 这些向数组添加元素。

+   `clone()`: 这克隆数组，类似于 `Arrays` 类的 `copyOf()` 方法以及 `java.lang.System` 的 `arraycopy()` 方法。

+   `getLength()`: 这返回数组长度或 `0`，当数组本身为 `null` 时。

+   `hashCode()`: 这计算数组的哈希值，包括嵌套数组。

+   `contains()`, `indexOf()`, 和 `lastIndexOf()`: 这些用于搜索数组。

+   `isSorted()`, `isEmpty`, 和 `isNotEmpty()`: 这些检查数组并处理 `null`。

+   `isSameLength()` 和 `isSameType()`: 这些用于比较数组。

+   `nullToEmpty()`: 这将 `null` 数组转换为空数组。

+   `remove()`, `removeAll()`, `removeElement()`, `removeElements()`, 和 `removeAllOccurances()`: 这些方法会移除某些或所有元素。

+   `reverse()`, `shift()`, `shuffle()`, 和 `swap()`: 这些方法会改变数组元素的顺序。

+   `subarray()`: 这个方法根据索引范围提取数组的一部分。

+   `toMap()`, `toObject()`, `toPrimitive()`, `toString()`, 和 `toStringArray()`: 这些方法将数组转换为其他类型并处理 `null` 值。

# 对象实用工具

本节中描述了以下两个实用工具：

+   `java.util.Objects`

+   `org.apache.commons.lang3.ObjectUtils`

它们在类创建期间特别有用，因此我们将主要关注与此任务相关的这些方法。

## java.util.Objects 类

`Objects` 类只有 17 个方法，它们都是静态的。让我们在将它们应用于 `Person` 类的同时看看其中的一些方法。假设这个类将是集合的一个元素，这意味着它必须实现 `equals()` 和 `hashCode()` 方法。代码如下所示：

```java
class Person {
```

```java
    private int age;
```

```java
    private String name;
```

```java
    public Person(int age, String name) {
```

```java
        this.age = age;
```

```java
        this.name = name;
```

```java
    }
```

```java
    public int getAge(){ return this.age; }
```

```java
    public String getName(){ return this.name; }
```

```java
    @Override
```

```java
    public boolean equals(Object o) {
```

```java
        if (this == o) return true;
```

```java
        if (o == null) return false;
```

```java
        if(!(o instanceof Person)) return false;
```

```java
        Person = (Person)o;
```

```java
        return age == person.getAge() &&
```

```java
                Objects.equals(name, person.getName()); 
```

```java
    }
```

```java
    @Override
```

```java
    public int hashCode(){
```

```java
        return Objects.hash(age, name);
```

```java
    }
```

```java
}
```

注意，我们不会检查 `name` 属性是否为 `null`，因为 `Object.equals()` 在任何参数为 `null` 时不会中断，它只是执行比较对象的工作。如果只有一个参数为 `null`，则返回 `false`。如果两个都是 `null`，则返回 `true`。

使用 `Object.equals()` 是实现 `equals()` 方法的安全方式；然而，如果你需要比较可能为数组的对象，最好使用 `Objects.deepEquals()` 方法，因为它不仅处理 `null`，就像 `Object.equals()` 方法一样，而且还比较所有数组元素的值，即使数组是多维的，如下所示：

```java
String[][] x1 = {{"a","b"},{"x","y"}};
```

```java
String[][] x2 = {{"a","b"},{"x","y"}};
```

```java
String[][] y =  {{"a","b"},{"y","y"}};
```

```java
System.out.println(Objects.equals(x1, x2));
```

```java
                                       //prints: false
```

```java
System.out.println(Objects.equals(x1, y));  
```

```java
                                       //prints: false
```

```java
System.out.println(Objects.deepEquals(x1, x2));
```

```java
                                       //prints: true
```

```java
System.out.println(Objects.deepEquals(x1, y));
```

```java
                                      //prints: false
```

`Objects.hash()` 方法也处理 `null` 值。需要记住的一个重要事项是，`equals()` 方法中比较的属性列表必须与传递给 `Objects.hash()` 作为参数的属性列表相匹配。否则，两个相等的 `Person` 对象将具有不同的哈希值，这会使基于哈希的集合工作不正确。

另一个值得注意的事情是，还有一个只接受一个参数的与哈希相关的 `Objects.hashCode()` 方法，但它生成的值不等于只接受一个参数的 `Objects.hash()` 生成的值。例如，观察以下内容：

```java
System.out.println(Objects.hash(42) ==
```

```java
               Objects.hashCode(42));    //prints: false
```

```java
System.out.println(Objects.hash("abc") ==
```

```java
               Objects.hashCode("abc")); //prints: false
```

为了避免这个注意事项，始终使用 `Objects.hash()`。

另一个潜在的混淆来源如下代码片段所示：

```java
System.out.println(Objects.hash(null));      //prints: 0
```

```java
System.out.println(Objects.hashCode(null));  //prints: 0
```

```java
System.out.println(Objects.hash(0));         //prints: 31
```

```java
System.out.println(Objects.hashCode(0));     //prints: 0
```

正如你所见，`Objects.hashCode()` 方法为 `null` 和 `0` 生成相同的哈希值，这可能会对基于哈希值的某些算法造成问题。

`static <T> int compare (T a, T b, Comparator<T> c)` 是另一个流行的返回 `0`（如果参数相等）的方法；否则，它返回 `c.compare(a, b)` 的结果。这对于实现 `Comparable` 接口（为自定义对象排序建立自然顺序）非常有用。例如，观察以下内容：

```java
class Person implements Comparable<Person> {
```

```java
    private int age;
```

```java
    private String name;
```

```java
    public Person(int age, String name) {
```

```java
        this.age = age;
```

```java
        this.name = name;
```

```java
    }
```

```java
    public int getAge(){ return this.age; }
```

```java
    public String getName(){ return this.name; }
```

```java
    @Override
```

```java
    public int compareTo(Person p){
```

```java
        int result = Objects.compare(name, p.getName(),
```

```java
                                    Comparator.naturalOrder());
```

```java
        if (result != 0) { 
```

```java
           return result;
```

```java
        }
```

```java
        return Objects.compare(age, p.getAge(),
```

```java
                                    Comparator.naturalOrder());
```

```java
    }
```

```java
}
```

这样，您可以通过设置 `Comparator.reverseOrder()` 的值或添加 `Comparator.nullFirst()` 或 `Comparator.nullLast()` 来轻松更改排序算法。

此外，我们之前章节中使用的 `Comparator` 实现可以通过使用 `Objects.compare()` 方法变得更加灵活，如下所示：

```java
class ComparePersons implements Comparator<Person> {
```

```java
    public int compare(Person p1, Person p2){
```

```java
        int result = Objects.compare(p1.getName(),
```

```java
           p2.getName(), Comparator.naturalOrder());
```

```java
        if (result != 0) { 
```

```java
           return result;
```

```java
        }
```

```java
        return Objects.compare(p1.getAge(), p2.getAge(),
```

```java
                              Comparator.naturalOrder());
```

```java
    }
```

```java
}
```

最后，我们将讨论 `Objects` 类的最后两个方法，这些方法是生成对象字符串表示的方法。当您需要对对象调用 `toString()` 方法但不确定对象引用是否为 `null` 时，它们非常有用。例如，观察以下内容：

```java
List<String> list = Arrays.asList("s1", null);
```

```java
for(String e: list){
```

```java
    //String s = e.toString();  //NullPointerException
```

```java
}
```

在前面的例子中，我们知道每个元素的确切值；然而，想象一下这样的场景：列表作为参数传递给方法。然后，我们被迫编写如下内容：

```java
void someMethod(List<String> list){
```

```java
    for(String e: list){
```

```java
        String s = e == null ? "null" : e.toString();
```

```java
    }
```

这似乎不是什么大问题。但是，编写了十几次这样的代码后，程序员自然会考虑某种类型的实用程序方法，它能够完成所有这些，这就是 `Objects` 类的以下两个方法帮助的时候：

+   `static String toString(Object o)`：当参数不是 `null` 时，返回调用 `toString()` 的结果，当参数值为 `null` 时返回 `null`。

+   `static String toString(Object o, String nullDefault)`：当第一个参数不是 `null` 时，返回调用第一个参数 `toString()` 的结果，当第一个参数值为 `null` 时返回第二个 `nullDefault` 参数值。

以下代码片段演示了这两个方法：

```java
List<String> list = Arrays.asList("s1", null);
```

```java
for(String e: list){
```

```java
    String s = Objects.toString(e);
```

```java
    System.out.print(s + " ");          //prints: s1 null
```

```java
}
```

```java
for(String e: list){
```

```java
    String s = Objects.toString(e, "element was null");
```

```java
    System.out.print(s + " ");        
```

```java
                                  //prints: s1 element was null
```

```java
}
```

到写作时为止，`Objects` 类有 17 个方法。我们建议您熟悉它们，以便在存在相同实用程序的情况下避免编写自己的实用程序。

## `ObjectUtils` 类

前一节中的最后一条语句适用于 `Apache Commons` 库中的 `org.apache.commons.lang3.ObjectUtils` 类，该库补充了前一节中描述的 `java.util.Objects` 类的方法。本书的范围和分配的大小不允许对 `ObjectUtils` 类下的所有方法进行详细审查，因此我们将根据它们的相关功能将它们简要地分组描述。要使用此类，您需要在 Maven 的 `pom.xml` 配置文件中添加以下依赖项：

```java
<dependency>
```

```java
    <groupId>org.apache.commons</groupId>
```

```java
    <artifactId>commons-lang3</artifactId>
```

```java
    <version>3.12.0</version>
```

```java
</dependency>
```

`ObjectUtils` 类的所有方法可以分为七个组，如下所示：

+   对象克隆方法

+   支持比较两个对象的方法

+   `notEqual()` 方法，用于比较两个对象的不相等性，其中一个或两个对象可能是 `null`

+   几个 `identityToString()` 方法，它们生成提供的对象的 `String` 表示形式，就像由 `Object` 基类的默认方法生成一样，并可选地将其附加到另一个对象上。

+   `allNotNull()` 和 `anyNotNull()` 方法，用于分析对象数组中的 `null`

+   `firstNonNull()`和`defaultIfNull()`方法，这些方法分析一个对象数组，并返回第一个非`null`对象或默认值

+   `max()`、`min()`、`median()`和`mode()`方法，这些方法分析一个对象数组，并返回与方法名称相对应的对象

# `java.time`包

`java.time`包及其子包中有很多类。它们被引入作为替代其他（较旧的包）处理日期和时间的包。新类是线程安全的（因此，更适合多线程处理），而且也很重要的是，它们的设计更加一致，更容易理解。此外，新实现遵循**国际标准化组织**（**ISO**）关于日期和时间格式的标准，但也允许使用任何其他自定义格式。

我们将描述以下五个主要类，并演示如何使用它们：

+   `java.time.LocalDate`

+   `java.time.LocalTime`

+   `java.time.LocalDateTime`

+   `java.time.Period`

+   `java.time.Duration`

所有这些以及`java.time`包及其子包中的其他类都功能丰富，涵盖了所有实际案例。但我们不会讨论所有这些；我们只会介绍基础知识以及最常用的用例。

## `LocalDate`类

`LocalDate`类不携带时间。它以*ISO 8601*格式（`yyyy-MM-dd`）表示日期，如下面的代码片段所示：

```java
System.out.println(LocalDate.now()); 
```

```java
                    //prints: current date in format yyyy-MM-dd
```

那是撰写时的当前位置的当前日期。该值是从计算机时钟中获取的。同样，您可以使用那个静态的`now(ZoneId zone)`方法获取任何其他时区的当前日期。可以使用静态`ZoneId.of(String zoneId)`方法构造一个`ZoneId`对象，其中`String zoneId`是`ZoneId.getAvailableZoneIds()`方法返回的任何字符串值，如下面的代码片段所示：

```java
Set<String> zoneIds = ZoneId.getAvailableZoneIds();
```

```java
for(String zoneId: zoneIds){
```

```java
    System.out.println(zoneId);
```

```java
}
```

上述代码打印了近 600 个时区**标识符**（**ID**）。这里有一些例子：

```java
Asia/Aden
```

```java
Etc/GMT+9
```

```java
Africa/Nairobi
```

```java
America/Marigot
```

```java
Pacific/Honolulu
```

```java
Australia/Hobart
```

```java
Europe/London
```

```java
America/Indiana/Petersburg
```

```java
Asia/Yerevan
```

```java
Europe/Brussels
```

```java
GMT
```

```java
Chile/Continental
```

```java
Pacific/Yap
```

```java
CET
```

```java
Etc/GMT-1
```

```java
Canada/Yukon
```

```java
Atlantic/St_Helena
```

```java
Libya
```

```java
US/Pacific-New
```

```java
Cuba
```

```java
Israel
```

```java
GB-Eire
```

```java
GB
```

```java
Mexico/General
```

```java
Universal
```

```java
Zulu
```

```java
Iran
```

```java
Navajo
```

```java
Egypt
```

```java
Etc/UTC
```

```java
SystemV/AST4ADT
```

```java
Asia/Tokyo
```

例如，让我们尝试使用`"Asia/Tokyo"`，如下所示：

```java
ZoneId = ZoneId.of("Asia/Tokyo");
```

```java
System.out.println(LocalDate.now(zoneId)); 
```

```java
           //prints: current date in Tokyo in format yyyy-MM-dd
```

一个`LocalDate`对象可以使用以下方法表示过去或未来的任何日期：

+   `LocalDate parse(CharSequence text)`: 这将从*ISO 8601*格式（`yyyy-MM-dd`）的字符串中构造一个对象。

+   `LocalDate parse(CharSequence text, DateTimeFormatter formatter)`: 这将从由 `DateTimeFormatter` 对象指定的格式中的字符串构造一个对象，该对象具有丰富的模式系统以及许多预定义的格式——这里有一些例子：

    +   `BASIC_ISO_DATE`—例如，`20111203`

    +   `ISO_LOCAL_DATE ISO`—例如，`2011-12-03`

    +   `ISO_OFFSET_DATE`—例如，`2011-12-03+01:00`

    +   `ISO_DATE`—例如，`2011-12-03+01:00; 2011-12-03`

    +   `ISO_LOCAL_TIME`—例如，`10:15:30`

    +   `ISO_OFFSET_TIME`—例如，`10:15:30+01:00`

    +   `ISO_TIME`—例如，`10:15:30+01:00; 10:15:30`

    +   `ISO_LOCAL_DATE_TIME`—例如，`2011-12-03T10:15:30`

+   `LocalDate of(int year, int month, int dayOfMonth)`: 从年、月和日构建一个对象。

+   `LocalDate of(int year, Month, int dayOfMonth)`: 从年、月（枚举常量）和日构建一个对象。

+   `LocalDate ofYearDay(int year, int dayOfYear)`: 从年和年内的日构建一个对象。

以下代码片段演示了前面提到的列表中的方法：

```java
LocalDate lc1 = LocalDate.parse("2023-02-23");
```

```java
System.out.println(lc1);           //prints: 2023-02-23
```

```java
LocalDate lc2 = LocalDate.parse("20230223",
```

```java
                     DateTimeFormatter.BASIC_ISO_DATE);
```

```java
System.out.println(lc2);           //prints: 2023-02-23
```

```java
DateTimeFormatter frm =
```

```java
              DateTimeFormatter.ofPattern("dd/MM/yyyy");
```

```java
LocalDate lc3 =  LocalDate.parse("23/02/2023", frm);
```

```java
System.out.println(lc3);           //prints: 2023-02-23
```

```java
LocalDate lc4 =  LocalDate.of(2023, 2, 23);
```

```java
System.out.println(lc4);           //prints: 2023-02-23
```

```java
LocalDate lc5 =  LocalDate.of(2023, Month.FEBRUARY, 23);
```

```java
System.out.println(lc5);           //prints: 2023-02-23
```

```java
LocalDate lc6 = LocalDate.ofYearDay(2023, 54);
```

```java
System.out.println(lc6);           //prints: 2023-02-23
```

`LocalDate`对象可以提供各种值，如下面的代码片段所示：

```java
LocalDate lc = LocalDate.parse("2023-02-23");
```

```java
System.out.println(lc);                  //prints: 2023-02-23
```

```java
System.out.println(lc.getYear());        //prints: 2023
```

```java
System.out.println(lc.getMonth());       //prints: FEBRUARY
```

```java
System.out.println(lc.getMonthValue());  //prints: 2
```

```java
System.out.println(lc.getDayOfMonth());  //prints: 23
```

```java
System.out.println(lc.getDayOfWeek());   //prints: THURSDAY
```

```java
System.out.println(lc.isLeapYear());     //prints: false
```

```java
System.out.println(lc.lengthOfMonth());  //prints: 28
```

```java
System.out.println(lc.lengthOfYear());   //prints: 365
```

一个`LocalDate`对象可以被修改，如下所示：

```java
LocalDate lc = LocalDate.parse("2023-02-23");
```

```java
System.out.println(lc.withYear(2024));     //prints: 2024-02-23
```

```java
System.out.println(lc.withMonth(5));       //prints: 2023-05-23
```

```java
System.out.println(lc.withDayOfMonth(5));  //prints: 2023-02-05
```

```java
System.out.println(lc.withDayOfYear(53));  //prints: 2023-02-22
```

```java
System.out.println(lc.plusDays(10));       //prints: 2023-03-05
```

```java
System.out.println(lc.plusMonths(2));      //prints: 2023-04-23
```

```java
System.out.println(lc.plusYears(2));       //prints: 2025-02-23
```

```java
System.out.println(lc.minusDays(10));      //prints: 2023-02-13
```

```java
System.out.println(lc.minusMonths(2));     //prints: 2022-12-23
```

```java
System.out.println(lc.minusYears(2));      //prints: 2021-02-23
```

一个`LocalDate`对象可以进行比较，如下所示：

```java
LocalDate lc1 = LocalDate.parse("2023-02-23");
```

```java
LocalDate lc2 = LocalDate.parse("2023-02-22");
```

```java
System.out.println(lc1.isAfter(lc2));       //prints: true
```

```java
System.out.println(lc1.isBefore(lc2));      //prints: false
```

`LocalDate`类中有许多其他有用的方法。如果您必须处理日期，我们建议您阅读`java.time`包及其子包。

## LocalTime 类

`LocalTime`类包含没有日期的时间。它具有与`LocalDate`类类似的方法。以下是如何创建`LocalTime`类对象的示例：

```java
System.out.println(LocalTime.now()); //prints: 21:15:46.360904
```

```java
ZoneId = ZoneId.of("Asia/Tokyo");
```

```java
System.out.println(LocalTime.now(zoneId)); 
```

```java
                                     //prints: 12:15:46.364378
```

```java
LocalTime lt1 =  LocalTime.parse("20:23:12");
```

```java
System.out.println(lt1);                     //prints: 20:23:12
```

```java
LocalTime lt2 = LocalTime.of(20, 23, 12);
```

```java
System.out.println(lt2);                     //prints: 20:23:12
```

可以从`LocalTime`对象中提取时间值的每个组成部分，如下所示：

```java
LocalTime lt2 =  LocalTime.of(20, 23, 12);
```

```java
System.out.println(lt2);                     //prints: 20:23:12
```

```java
System.out.println(lt2.getHour());           //prints: 20
```

```java
System.out.println(lt2.getMinute());         //prints: 23
```

```java
System.out.println(lt2.getSecond());         //prints: 12
```

```java
System.out.println(lt2.getNano());           //prints: 0
```

`LocalTime`类的对象可以被修改，如下所示：

```java
LocalTime lt2 = LocalTime.of(20, 23, 12);
```

```java
System.out.println(lt2.withHour(3));      //prints: 03:23:12
```

```java
System.out.println(lt2.withMinute(10));   //prints: 20:10:12
```

```java
System.out.println(lt2.withSecond(15));   //prints: 20:23:15
```

```java
System.out.println(lt2.withNano(300)); 
```

```java
                                   //prints: 20:23:12.000000300
```

```java
System.out.println(lt2.plusHours(10));    //prints: 06:23:12
```

```java
System.out.println(lt2.plusMinutes(2));   //prints: 20:25:12
```

```java
System.out.println(lt2.plusSeconds(2));   //prints: 20:23:14
```

```java
System.out.println(lt2.plusNanos(200));
```

```java
                                   //prints: 20:23:12.000000200
```

```java
System.out.println(lt2.minusHours(10));   //prints: 10:23:12
```

```java
System.out.println(lt2.minusMinutes(2));  //prints: 20:21:12
```

```java
System.out.println(lt2.minusSeconds(2));  //prints: 20:23:10
```

```java
System.out.println(lt2.minusNanos(200));
```

```java
                                   //prints: 20:23:11.999999800
```

并且两个`LocalTime`类的对象也可以进行比较，如下所示：

```java
LocalTime lt2 =  LocalTime.of(20, 23, 12);
```

```java
LocalTime lt4 =  LocalTime.parse("20:25:12");
```

```java
System.out.println(lt2.isAfter(lt4));       //prints: false
```

```java
System.out.println(lt2.isBefore(lt4));      //prints: true
```

`LocalTime`类中有许多其他有用的方法。如果您必须处理日期，我们建议您阅读此类以及其他`java.time`包及其子包的类的 API。

## LocalDateTime 类

`LocalDateTime`类包含日期和时间，并且具有`LocalDate`和`LocalTime`类所有的所有方法，所以我们在这里不会重复它们。我们只会展示如何创建一个`LocalDateTime`类的对象，如下所示：

```java
System.out.println(LocalDateTime.now());       
```

```java
                     //prints: 2019-03-04T21:59:00.142804
```

```java
ZoneId = ZoneId.of("Asia/Tokyo");
```

```java
System.out.println(LocalDateTime.now(zoneId)); 
```

```java
                    //prints: 2019-03-05T12:59:00.146038
```

```java
LocalDateTime ldt1 = 
```

```java
           LocalDateTime.parse("2020-02-23T20:23:12");
```

```java
System.out.println(ldt1);  //prints: 2020-02-23T20:23:12
```

```java
DateTimeFormatter formatter =
```

```java
     DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm:ss");
```

```java
LocalDateTime ldt2 =
```

```java
  LocalDateTime.parse("23/02/2020 20:23:12", formatter);
```

```java
System.out.println(ldt2);  //prints: 2020-02-23T20:23:12
```

```java
LocalDateTime ldt3 = 
```

```java
               LocalDateTime.of(2020, 2, 23, 20, 23, 12);
```

```java
System.out.println(ldt3);  //prints: 2020-02-23T20:23:12
```

```java
LocalDateTime ldt4 =
```

```java
  LocalDateTime.of(2020, Month.FEBRUARY, 23, 20, 23, 12);
```

```java
System.out.println(ldt4);  //prints: 2020-02-23T20:23:12
```

```java
LocalDate ld = LocalDate.of(2020, 2, 23);
```

```java
LocalTime lt = LocalTime.of(20, 23, 12);
```

```java
LocalDateTime ldt5 = LocalDateTime.of(ld, lt);
```

```java
System.out.println(ldt5); //prints: 2020-02-23T20:23:12
```

`LocalDateTime`类中有许多其他有用的方法。如果您必须处理日期，我们建议您阅读此类以及其他`java.time`包及其子包的类的 API。

## Period 和 Duration 类

`java.time.Period`和`java.time.Duration`类被设计用来包含一定的时间量，如下所述：

+   `Period`对象包含以年、月和日为单位的时间量。

+   `Duration`对象包含小时、分钟、秒和纳秒的时间量。

以下代码片段演示了使用`LocalDateTime`类创建和使用它们的方法，但相同的方法也存在于`LocalDate`（对于`Period`）和`LocalTime`（对于`Duration`）类中：

```java
LocalDateTime ldt1 = LocalDateTime.parse("2023-02-23T20:23:12");
```

```java
LocalDateTime ldt2 = ldt1.plus(Period.ofYears(2));
```

```java
System.out.println(ldt2);      //prints: 2025-02-23T20:23:12
```

以下方法与`LocalTime`类的方法以相同的方式工作：

```java
LocalDateTime ldt = LocalDateTime.parse("2023-02-23T20:23:12");
```

```java
ldt.minus(Period.ofYears(2));
```

```java
ldt.plus(Period.ofMonths(2));
```

```java
ldt.minus(Period.ofMonths(2));
```

```java
ldt.plus(Period.ofWeeks(2));
```

```java
ldt.minus(Period.ofWeeks(2));
```

```java
ldt.plus(Period.ofDays(2));
```

```java
ldt.minus(Period.ofDays(2));
```

```java
ldt.plus(Duration.ofHours(2));
```

```java
ldt.minus(Duration.ofHours(2));
```

```java
ldt.plus(Duration.ofMinutes(2));
```

```java
ldt.minus(Duration.ofMinutes(2));
```

```java
ldt.plus(Duration.ofMillis(2));
```

```java
ldt.minus(Duration.ofMillis(2));
```

在以下代码片段中演示了创建和使用`Period`对象的一些其他方法：

```java
LocalDate ld1 =  LocalDate.parse("2023-02-23");
```

```java
LocalDate ld2 =  LocalDate.parse("2023-03-25");
```

```java
Period = Period.between(ld1, ld2);
```

```java
System.out.println(period.getDays());       //prints: 2
```

```java
System.out.println(period.getMonths());     //prints: 1
```

```java
System.out.println(period.getYears());      //prints: 0
```

```java
System.out.println(period.toTotalMonths()); //prints: 1
```

```java
period = Period.between(ld2, ld1);
```

```java
System.out.println(period.getDays());       //prints: -2
```

`Duration`对象可以类似地创建和使用，如下面的代码片段所示：

```java
LocalTime lt1 =  LocalTime.parse("10:23:12");
```

```java
LocalTime lt2 =  LocalTime.parse("20:23:14");
```

```java
Duration = Duration.between(lt1, lt2);
```

```java
System.out.println(duration.toDays());     //prints: 0
```

```java
System.out.println(duration.toHours());    //prints: 10
```

```java
System.out.println(duration.toMinutes());  //prints: 600
```

```java
System.out.println(duration.toSeconds());  //prints: 36002
```

```java
System.out.println(duration.getSeconds()); //prints: 36002
```

```java
System.out.println(duration.toNanos());    
```

```java
                                       //prints: 36002000000000
```

```java
System.out.println(duration.getNano());    //prints: 0.
```

`Period`和`Duration`类中有许多其他有用的方法。如果您必须处理日期，我们建议您阅读此类以及其他`java.time`包及其子包的类的 API。

## 一天的 Period

Java 16 包含了一种新的时间格式，它将一天中的时间段显示为 `AM`、`in the morning` 等类似的形式。以下两个方法演示了使用 `DateTimeFormatter.ofPattern()` 方法与 `LocalDateTime` 和 `LocalTime` 类的使用：

```java
void periodOfDayFromDateTime(String time, String pattern){
```

```java
   LocalDateTime date = LocalDateTime.parse(time);
```

```java
   DateTimeFormatter frm =
```

```java
            DateTimeFormatter.ofPattern(pattern);
```

```java
   System.out.print(date.format(frm));
```

```java
} 
```

```java
void periodOfDayFromTime(String time, String pattern){
```

```java
   LocalTime date = LocalTime.parse(time);
```

```java
   DateTimeFormatter frm =
```

```java
           DateTimeFormatter.ofPattern(pattern);
```

```java
   System.out.print(date.format(frm));
```

```java
}
```

以下代码演示了 `"h a"` 和 `"h B"` 模式的效果：

```java
periodOfDayFromDateTime("2023-03-23T05:05:18.123456", 
```

```java
           "MM-dd-yyyy h a"); //prints: 03-23-2023 5 AM
```

```java
periodOfDayFromDateTime("2023-03-23T05:05:18.123456", 
```

```java
       "MM-dd-yyyy h B"); //prints: 03-23-2023 5 at night
```

```java
periodOfDayFromDateTime("2023-03-23T06:05:18.123456", 
```

```java
                  "h B");   //prints: 6 in the morning
```

```java
periodOfDayFromTime("11:05:18.123456", "h B"); 
```

```java
                            //prints: 11 in the morning
```

```java
periodOfDayFromTime("12:05:18.123456", "h B"); 
```

```java
                          //prints: 12 in the afternoon
```

```java
periodOfDayFromTime("17:05:18.123456", "h B"); 
```

```java
                          //prints: 5 in the afternoon
```

```java
periodOfDayFromTime("18:05:18.123456", "h B"); 
```

```java
                          //prints: 6 in the evening
```

```java
periodOfDayFromTime("20:05:18.123456", "h B"); 
```

```java
                          //prints: 8 in the evening
```

```java
periodOfDayFromTime("21:05:18.123456", "h B"); 
```

```java
                         //prints: 9 at night
```

您可以使用 `"h a"` 和 `"h B"` 模式使时间表示更友好。

# 概述

本章向您介绍了 Java 集合框架及其三个主要接口：`List`、`Set` 和 `Map`。每个接口都进行了讨论，并使用实现类演示了其方法。泛型也得到了解释和演示。为了使对象能够被 Java 集合正确处理，必须实现 `equals()` 和 `hashCode()` 方法。

`Collections` 和 `CollectionUtils` 工具类提供了许多用于集合处理的实用方法，并在示例中展示了这些方法，包括 `Arrays`、`ArrayUtils`、`Objects` 和 `ObjectUtils` 类。

`java.time` 包的类方法允许管理时间/日期值，并在特定的实际代码片段中进行了演示。

您现在可以在您的程序中使用本章讨论的所有主要数据结构。

在下一章中，我们将概述 JCL 以及一些外部库，包括支持测试的库。具体来说，我们将探索 `org.junit`、`org.mockito`、`org.apache.log4j`、`org.slf4j` 和 `org.apache.commons` 包及其子包。

# 测验

1.  Java 集合框架是什么？选择所有适用的选项：

    1.  集合框架

    1.  `java.util` 包的类和接口

    1.  `List`、`Set` 和 `Map` 接口

    1.  实现集合数据结构的类和接口

1.  在集合中，“泛型”指的是什么？选择所有适用的选项：

    1.  集合结构定义

    1.  元素类型声明

    1.  类型泛化

    1.  提供编译时安全性的机制

1.  `of()` 工厂方法集合的限制是什么？选择所有适用的选项：

    1.  它不允许 `null` 元素。

    1.  它不允许向初始化的集合中添加元素。

    1.  它不允许修改与初始化集合相关的元素。

1.  实现 `java.lang.Iterable` 接口允许什么？选择所有适用的选项：

    1.  它允许逐个访问集合的元素。

    1.  它允许集合在 `FOR` 语句中使用。

    1.  它允许集合在 `WHILE` 语句中使用。

    1.  它允许集合在 `DO...WHILE` 语句中使用。

1.  实现 `java.util.Collection` 接口允许什么？选择所有适用的选项：

    1.  向集合中添加来自另一个集合的元素

    1.  从另一个集合中移除集合中的对象

    1.  仅修改属于另一个集合的集合中的元素

    1.  从不属于另一个集合的对象集合中删除

1.  选择所有与 `List` 接口方法相关的正确陈述：

    1.  `z get(int index)`: 这个方法返回列表中指定位置的元素。

    1.  `E remove(int index)`: 这个方法从列表中删除指定位置的元素；它返回被删除的元素。

    1.  `static List<E> copyOf(Collection<E> coll)`: 这个方法返回一个包含给定 `Collection` 接口元素的不可修改的 `List` 接口，并保留它们的顺序。

    1.  `int indexOf(Object o)`: 这个方法返回列表中指定元素的位置。

1.  选择所有与 `Set` 接口方法相关的正确陈述：

    1.  `E get(int index)`: 这个方法返回列表中指定位置的元素。

    1.  `E remove(int index)`: 这个方法从列表中删除指定位置的元素；它返回被删除的元素。

    1.  `static Set<E> copyOf(Collection<E> coll)`: 这个方法返回一个包含给定 `Collection` 接口元素的不可修改的 `Set` 接口。

    1.  `int indexOf(Object o)`: 这个方法返回列表中指定元素的位置。

1.  选择所有与 `Map` 接口方法相关的正确陈述：

    1.  `int size()`: 这个方法返回存储在映射中的键值对数量；当 `isEmpty()` 方法返回 `true` 时，此方法返回 `0`。

    1.  `V remove(Object key)`: 这个方法从映射中删除键和值；如果没有这样的键或值是 `null`，则返回 `value` 或 `null`。

    1.  `default boolean remove(Object key, Object value)`: 如果映射中存在这样的键值对，则删除键值对；如果值被删除，则返回 `true`。

    1.  `default boolean replace(K key, V oldValue, V newValue)`: 如果提供的键当前映射到 `oldValue` 值，则用提供的 `newValue` 值替换 `oldValue` 值——如果替换了 `oldValue` 值，则返回 `true`；否则返回 `false`。

1.  选择所有与 `Collections` 类的 `static void sort(List<T> list, Comparator<T> comparator)` 方法相关的正确陈述：

    1.  如果列表元素实现了 `Comparable` 接口，它将根据列表的自然顺序对列表进行排序。

    1.  它根据提供的 `Comparator` 对象对列表的顺序进行排序。

    1.  如果列表元素实现了 `Comparable` 接口，它将根据提供的 `Comparator` 对象对列表的顺序进行排序。

    1.  不论列表元素是否实现了 `Comparable` 接口，它都将根据提供的 `Comparator` 对象对列表的顺序进行排序。

1.  执行以下代码的结果是什么？

    ```java
    List<String> list1 = Arrays.asList("s1","s2", "s3");
    List<String> list2 = Arrays.asList("s3", "s4");
    Collections.copy(list1, list2);
    System.out.println(list1);    
    ```

    1.  `[s1, s2, s3, s4]`

    1.  `[s3, s4, s3]`

    1.  `[s1, s2, s3, s3, s4]`

    1.  `[s3, s4]`

1.  `CollectionUtils` 类方法的功能是什么？选择所有适用的：

    1.  它通过处理 `null` 来匹配 `Collections` 类方法的函数。

    1.  它补充了 `Collections` 类方法的函数。

    1.  它以 `Collections` 类方法不进行的方式搜索、处理和比较 Java 集合。

    1.  它重复了`Collections`类方法的功能

1.  执行以下代码的结果是什么？

    ```java
    Integer[][] ar1 = {{42}};
    Integer[][] ar2 = {{42}};
    System.out.print(Arrays.equals(ar1, ar2) + " "); 
    System.out.println(Arrays.deepEquals(arr3, arr4)); 
    ```

    1.  `false true`

    1.  `false`

    1.  `true false`

    1.  `true`

1.  执行以下代码的结果是什么？

    ```java
    String[] arr1 = { "s1", "s2" };
    String[] arr2 = { null };
    String[] arr3 = null;
    System.out.print(ArrayUtils.getLength(arr1) + " "); 
    System.out.print(ArrayUtils.getLength(arr2) + " "); 
    System.out.print(ArrayUtils.getLength(arr3) + " "); 
    System.out.print(ArrayUtils.isEmpty(arr2) + " "); 
    System.out.print(ArrayUtils.isEmpty(arr3));
    ```

    1.  `1 2 0 false true`

    1.  `2 1 1 false true`

    1.  `2 1 0 false true`

    1.  `2 1 0 true false`

1.  执行以下代码的结果是什么？

    ```java
     String str1 = "";
     String str2 = null;
     System.out.print((Objects.hash(str1) == 
                       Objects.hashCode(str2)) + " ");
     System.out.print(Objects.hash(str1) + " ");
     System.out.println(Objects.hashCode(str2) + " "); 
    ```

    1.  `true 0 0`

    1.  `Error`

    1.  `false -1 0`

    1.  `false 31 0`

1.  执行以下代码的结果是什么？

    ```java
    String[] arr = {"c", "x", "a"};
    System.out.print(ObjectUtils.min(arr) + " ");
    System.out.print(ObjectUtils.median(arr) + " ");
    System.out.println(ObjectUtils.max(arr));
    ```

    1.  `c x a`

    1.  `a c x`

    1.  `x c a`

    1.  `a x c`

1.  执行以下代码的结果是什么？

    ```java
    LocalDate lc = LocalDate.parse("1900-02-23");
    System.out.println(lc.withYear(21)); 
    ```

    1.  `1921-02-23`

    1.  `21-02-23`

    1.  `0021-02-23`

    1.  `Error`

1.  执行以下代码的结果是什么？

    ```java
    LocalTime lt2 = LocalTime.of(20, 23, 12);
    System.out.println(lt2.withNano(300));      
    ```

    1.  `20:23:12.000000300`

    1.  `20:23:12.300`

    1.  `20:23:12:300`

    1.  `Error`

1.  执行以下代码的结果是什么？

    ```java
    LocalDate ld = LocalDate.of(2020, 2, 23);
    LocalTime lt = LocalTime.of(20, 23, 12);
    LocalDateTime ldt = LocalDateTime.of(ld, lt);
    System.out.println(ldt);                
    ```

    1.  `2020-02-23 20:23:12`

    1.  `2020-02-23T20:23:12`

    1.  `2020-02-23:20:23:12`

    1.  `Error`

1.  执行以下代码的结果是什么？

    ```java
    LocalDateTime ldt = 
                  LocalDateTime.parse("2020-02-23T20:23:12");
    System.out.print(ldt.minus(Period.ofYears(2)) + " ");
    System.out.print(ldt.plus(Duration.ofMinutes(12)) + " ");
    System.out.println(ldt);
    ```

    1.  `2020-02-23T20:23:12 2020-02-23T20:23:12`

    1.  `2020-02-23T20:23:12 2020-02-23T20:35:12`

    1.  `2018-02-23T20:23:12 2020-02-23T20:35:12 2020-02-23T20:23:12`

    1.  `2018-02-23T20:23:12 2020-02-23T20:35:12 2018-02-23T20:35:12`
