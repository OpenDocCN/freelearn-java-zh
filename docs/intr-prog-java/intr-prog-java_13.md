# 第十四章：Java 集合

本章将帮助读者更熟悉最常用的 Java 集合。代码示例说明了它们的功能，并允许进行实验，强调了不同集合类型及其实现之间的差异。

在本章中，我们将涵盖以下主题：

+   什么是集合？

+   列表和 ArrayList

+   Set 和 HashSet

+   Map 和 HashMap

+   练习-EnumSet 方法

# 什么是集合？

当你阅读关于 Java 集合框架的内容时，你可能会认为这样的集合有些特殊。与此同时，框架这个词被滥用了，就像技术这个词一样，我们已经拒绝使用了。在英语中，框架这个词的意思是“系统、概念或文本的基本结构”。在计算机编程中，框架是一个软件系统，它的功能可以通过额外的用户编写的代码或配置设置来定制，以满足特定应用程序的要求。

但是当我们仔细研究 Java 集合框架的内容时，我们意识到它的所有成员都属于`java.util`包，这是 Java 类库的一部分，正如我们在前一章中所描述的。而另一方面，`java.awt`、`javax.swing`或`javafx`包中的图形用户界面具有框架的所有特征；它们只提供小工具和其他图形元素，这些元素必须由特定应用程序的内容填充。然而，它们也属于 Java 类库。

这就是为什么我们避免使用框架这个词，并且只在这里提到它，以解释 Java 集合框架标题背后隐藏的东西。

# `java.util`包

`java.util`包中的以下接口和类组成了 Java 集合框架：

+   扩展`java.util.Collection`接口的接口（它又扩展了`java.lang.Iterable`接口）：`List`、`Set`和`Queue`，这些是最流行的接口之一

+   实现上述接口的类：`ArrayList`、`HashSet`、`Stack`和`LinkedList`，作为示例

+   实现`java.util.Map`接口及其子类的类：`HashMap`、`HashTable`和`TreeMap`，只是其中最常用的三个

正如你所看到的，Java 集合框架，或者只是 Java 集合，由扩展`java.util.Collection`接口或`java.util.Map`接口的接口和实现这些接口的类组成-所有这些都包含在`java.util`包中。

请注意，那些直接或间接实现`Collection`接口的类也实现了`Iterable`接口，因此可以在迭代语句中使用，如第十章中所述的“控制流语句”。

# Apache Commons 集合

Apache Commons 项目包含了（在`org.apache.commons.collections`包中）多个 Java 集合接口的实现，这些实现补充了`java.util`包中的实现。但是，除非你在一个需要特定集合算法的应用程序上工作，否则你可能不需要使用它们。尽管如此，我们建议你浏览一下`org.apache.commons.collections`包的 API，这样你就知道它的内容，以防将来遇到需要使用它的情况。

# 集合与数组

所有集合都是类似于数组的数据结构，因为它们也包含元素，并且每个元素都由一个类的对象表示。不过，数组和集合之间有两个重要的区别：

+   数组在实例化时需要分配一个大小，而集合在添加或删除元素时会自动增加和减少大小。

+   集合的元素不能是原始类型的值，而只能是引用类型，包括包装类，如`Integer`或`Double`。好消息是您可以添加原始值：

```java
       List list = new ArrayList();
       list.add(42);

```

在前面的语句中，装箱转换（请参阅第九章，“运算符、表达式和语句”）将自动应用于原始值。

尽管数组在访问其元素时预计会提供更好的性能，但实际上，现代算法使数组和集合的性能差异可以忽略不计，除了一些非常专业的情况。这就是为什么您必须使用数组的唯一原因是当一些算法或方法需要它时。

# 这是我们将要讨论的内容

在接下来的小节中，我们将讨论 Java 集合标准库中最受欢迎的接口和类：

+   `List`接口和`ArrayList`类-它们保留元素的顺序

+   `Set`接口和`HashSe`类-它们不允许重复元素

+   `Map`和`HashMap`接口-它们通过键存储对象，因此允许键值映射

请注意，以下小节中描述的大多数方法都来自`java.util.Collection`接口-几乎所有集合的父接口，除了实现`java.util.Map`接口的集合。

# List-ArrayList 保留顺序

`List`是一个接口，`ArrayList`类是其最常用的实现。两者都驻留在`java.util`包中。`ArrayList`类有一些额外的方法-除了`List`接口中声明的方法之外。例如，`removeRange()`方法在`List`接口中不存在，但在`ArrayList`API 中可用。

# 更喜欢变量类型 List

在创建`ArrayList`对象时，将其引用分配给`List`类型的变量是一个很好的做法：

```java
List listOfNames = new ArrayList();
```

很可能，在您的程序中使用`ArrayList`类型的变量不会改变任何内容，无论是今天还是将来：

```java
ArrayList listOfNames = new ArrayList();
```

前面的引用仍然可以传递给接受`List`类型参数的任何方法。但是，通常编码为接口（当我们将变量设置为接口类型时）是一个很好的习惯，因为您永远不知道代码的要求何时可能会更改，您可能需要使用另一个`List`的实现，例如`LinkedList`类。如果变量类型是`List`，切换实现很容易。但是，如果变量类型是`ArrayList`，将其更改为`List`或`LinkedList`需要跟踪变量使用的所有位置并运行各种测试，以确保没有在任何地方调用`ArrayList`方法。如果代码很复杂，人们永远无法确定是否已检查了所有可能的执行路径，并且代码不会在生产中中断。这就是为什么我们更喜欢使用接口类型来保存对对象的引用的变量，除非您确实需要它成为类类型。我们在第八章中广泛讨论了这一点，“面向对象设计（OOD）原则”。

# 为什么叫 ArrayList？

`ArrayList`类之所以被命名为 ArrayList，是因为它的实现是基于数组的。它实际上在幕后使用数组。如果在 IDE 中右键单击`ArrayList`并查看源代码，您将看到以下内容：

```java
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
public ArrayList() {
  this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

它只是数组`Object[]`的包装器。例如，这是方法`add(E)`的实现方式：

```java
public boolean add(E e) {
  modCount++;
  add(e, elementData, size);
  return true;
}
private void add(E e, Object[] elementData, int s) {
  if (s == elementData.length)
    elementData = grow();
  elementData[s] = e;
  size = s + 1;
}
```

And if you study the source code more and look inside the method `grow()`, you will see how it increases the size of the array when new elements are added to the list:

```java
private Object[] grow() {  return grow(size + 1); }

private Object[] grow(int minCapacity) {
  return elementData = Arrays.copyOf(elementData,
                                    newCapacity(minCapacity));
}
private static final int DEFAULT_CAPACITY = 10;
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
private int newCapacity(int minCapacity) {
  // overflow-conscious code
  int oldCapacity = elementData.length;
  int newCapacity = oldCapacity + (oldCapacity >> 1);
  if (newCapacity - minCapacity <= 0) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
      return Math.max(DEFAULT_CAPACITY, minCapacity);
    if (minCapacity < 0) // overflow
      throw new OutOfMemoryError();
    return minCapacity;
  }
  return (newCapacity - MAX_ARRAY_SIZE <= 0)
            ? newCapacity
            : hugeCapacity(minCapacity);
}
```

As you can see, when the allocated array size is not enough for storing another element, the new array is created with a minimum capacity of 10\. All the already existing elements are copied to the new array using the `Arrays.copyOf()` method (we will talk about the `Arrays` class later in this chapter).

And that is why `ArrayList` was so named.

For using `List` and `ArrayList`, you do not need to know all that, unless you have to process really big lists of elements and the frequent copying of the underlying array affects the performance of your code. In such a case, consider using different data structures that have been designed specifically for the type of processing you need. But that is already outside the scope of this book. Besides, the vast majority of mainstream programmers have probably never used any collections that are not in the `java.util` package.

# Adding elements

The `List` interface provides two methods for adding an element:

+   `add(E)`: This adds the element to the end of the list

+   `add(index, E)`: This inserts the element into the specified (by index, starting with zero) position in the list by shifting the element the specified position (if any) and any subsequent elements to the right by adding 1 to their indices

Both methods can throw a `RuntimeException` if something goes wrong. So, putting a try-catch block around the method makes the code more robust (if the catch block does not simply rethrow the exception but does something meaningful). Read the documentation of the `List` interface API online and see what the names of the exceptions these methods can throw are and under which conditions they can happen.

The `add(E)` method also returns a Boolean value (`true`/`false`) that indicates the success of the operation. This method overrides the method in the `Collection` interface, so all Java collections that extend or implement the `Collection` interface have it. In the case of `List` implementations, this method most likely always returns `true` because list allows duplicate entries. By contrast, the implementations of the `Set` interface return `false` if such an element is present already because `Set` does not allow duplicates. We will discuss this in subsequent sections, as well as how the code determines if two elements are the same.

Now, let's look at the examples of the `add()` method of the `List` interface's usage:

```java
List list = new ArrayList();
list.add(null);
list.add(1);
list.add("ss");
list.add(new A());
list.add(new B());
System.out.println(list);  //prints: [null, 1, ss, A, B]
list.add(2, 42);
System.out.println(list);  //prints: [null, 1, 42, ss, A, B]

```

In the preceding list, we have mixed up in the same list values of different types. The classes `A` and `B`, used in the preceding code, have parent-child relations:

```java
class A {
  @Override
  public String toString() { return "A"; }
}
class B extends A {
  @Override
  public String toString() { return "B"; }
}
```

如您所见，我们已经为它们的每个添加了`toString()`方法，这样我们就可以看到它们的对象以预期的格式打印出来。

# size(), isEmpty(), clear()

这三种方法很简单：

+   `size()`: 这返回列表中的元素数量

+   `isEmpty()`: 如果列表中没有元素，则返回`true`（`size()`返回 0）

+   `clear()`: 这将从列表中移除所有元素，使`isEmpty()`返回`true`，`size()`返回 0

# 遍历和流

实现`Collection`接口（它扩展了`Iterable`接口）的每个集合都可以使用第十章中讨论的增强`for`语句进行迭代。以下是一个示例：

```java
List list = new ArrayList();
list.add(null);
list.add(1);
list.add("ss");
list.add(new A());
list.add(new B());
for(Object o: list){
  //code that does something with each element   
} 
```

`Iterable`接口还向`List`接口添加了以下三种方法：

+   `forEach(Consumer function)`: 它将提供的函数应用于每个集合元素

+   `iterator()`: 它返回一个`Iterator`类的对象，允许遍历集合的每个元素并根据需要操作每个元素

+   `splititerator()`: 它返回一个`Splititerator`类的对象，允许将集合拆分以进行并行处理（讨论此功能的范围超出了本书的范围）

在第十七章中，*Lambda 表达式和函数式编程*，我们将解释如何将函数作为参数传递，所以现在我们只展示`forEach()`方法的用法示例（如果我们重用前面示例中创建的列表）：

```java
list.forEach(System.out::println);

```

如您所见，传入的函数会获取`forEach()`方法生成的每个元素并将其打印出来。它被称为`Consumer`，因为它获取（消耗）输入并不返回任何值，只是打印。如果我们运行上述代码，结果将如下所示：

![](img/42802e7f-83a1-4e82-98ed-3c2ae7c83e06.png)

`forEach()`方法提供了与`for`语句相同的功能（参见前面的示例），但需要编写更少的代码。这就是为什么程序员喜欢函数式编程（当函数可以被视为对象时），因为在多次编写相同的样板代码之后，可以享受简写的风格。

`iterator()`方法返回的`Iterator`接口具有以下方法：

+   `next()`: 它返回迭代中的下一个元素

+   `hasNext()`: 如果迭代还有更多元素，则返回`true`

+   `forEachRemaining (Consumer<? super E> function)`: 它将提供的函数应用于剩余的每个元素

+   `remove()`: 它从基础集合中移除此迭代器返回的最后一个元素

`next()`和`hasNext()`方法由`for`语句在后台使用。您也可以使用它们，实际上可以重现`for`语句的功能。但是为什么呢？`for`语句已经在做这个了。我们能想到使用`Iterator`接口的唯一原因是在遍历列表时删除一些对象（使用`remove()`方法）。这让我们讨论一个初学者经常犯的错误。

假设我们想要从以下列表中删除所有类型为`String`的对象：

```java
List list = new ArrayList();
list.add(null);
list.add(1);
list.add("ss");
list.add(new A());
list.add(new B());

```

以下是尝试执行此操作的代码，但存在缺陷：

```java
for(Object o: list){
  System.out.println(o);
  if(o instanceof String){
    list.remove(o);
  }
}
```

如果我们运行上述代码，结果将如下所示：

![](img/b1378467-d097-4151-a4ec-64ec09d2a9e2.png)

`ConcurrentModificationException`是因为我们在迭代集合时尝试修改它。`Iterator`类有助于避免这个问题。以下代码可以正常工作：

```java
System.out.println(list);  //prints: [null, 1, ss, A, B]
Iterator iter = list.iterator();
while(iter.hasNext()){
  Object o = iter.next();
  if(o instanceof String){
    iter.remove();
  }
}
System.out.println(list);  //prints: [null, 1, A, B]

```

我们不打算讨论为什么`Iterator`允许在迭代过程中删除元素，而集合在类似情况下抛出异常的原因有两个：

+   这需要比入门课程允许的更深入地了解 JVM 实现。

+   在第十八章中，*流和管道*，我们将演示使用 Java 函数式编程更紧凑的方法。这样的代码看起来如此清晰和优雅，以至于许多使用 Java 8 及更高版本的程序员在处理生成流的集合和其他数据结构时几乎不再使用`for`语句。

还有其他四种遍历元素列表的方法：

+   `listIterator()`和`listIterator(index)`：两者都返回`ListIterator`，它与`Iterator`非常相似，但允许沿着列表来回移动（`Iterator`只允许向前移动，正如你所见）。这些方法很少使用，所以我们将跳过它们的演示。但是如果你需要使用它们，看一下前面的`Iterator`示例。`ListIterator`的使用非常相似。

+   `stream()`和`parallelStream()`：两者都返回`Stream`对象，我们将在第十八章中更详细地讨论*流和管道*。

# 使用泛型添加

有时，在同一个列表中具有不同类型的情况正是我们想要的。但是，大多数情况下，我们希望列表包含相同类型的值。同时，代码可能存在逻辑错误，允许添加不同类型到列表中，这可能会产生意想不到的后果。如果导致抛出异常，那就不像一些默认转换和不正确的结果那么糟糕，这可能很长时间甚至永远不会被注意到。

为了避免这样的问题，可以使用允许定义集合元素期望类型的泛型，这样编译器可以检查并在添加不同类型时失败。这里是一个例子：

```java
List<Integer> list1 = new ArrayList<>();
list1.add(null);
list1.add(1);
//list1.add("ss");          //compilation error
//list1.add(new A());       //compilation error
//list1.add(new B());       //compilation error
System.out.println(list1);  //prints: [null, 1]
list1.add(2, 42);
System.out.println(list1);  //prints: [null, 1, 42]

```

正如你所看到的，`null`值无论如何都可以被添加，因为它是任何引用类型的默认值，而正如我们在本节开头已经指出的那样，任何 Java 集合的元素只能是引用类型。

由于子类具有任何其父类的类型，泛型`<Object>`并不能帮助避免先前描述的问题，因为每个 Java 对象都将`Object`类作为其父类：

```java
List<Object> list2= new ArrayList<>();
list2.add(null);
list2.add(1);
list2.add("ss");
list2.add(new A());
list2.add(new B());
System.out.println(list2);    //prints: [null, 1, ss, A, B]
list2.add(2, 42);
System.out.println(list2);    //prints: [null, 1, 42, ss, A, B]

```

但是，以下泛型更加严格：

```java
List<A> list3= new ArrayList<>();
list3.add(null);
//list3.add(1);            //compilation error
//list3.add("ss");         //compilation error
list3.add(new A());
list3.add(new B());
System.out.println(list3); //prints: [null, A, B]
list3.add(2, new A());
System.out.println(list3); //prints: [null, A, A, B]

List<B> list4= new ArrayList<>();
list4.add(null);
//list4.add(1);            //compilation error
//list4.add("ss");         //compilation error
//list4.add(new A());      //compilation error
list4.add(new B());
System.out.println(list4); //prints: [null, B]
list4.add(2, new B());
System.out.println(list4); //prints: [null, B, B]

```

唯一的情况是当您可能使用泛型`<Object>`的情况是，当您希望允许添加不同类型的值到列表中，但不希望允许列表本身的引用引用具有其他泛型的列表时：

```java
List list = new ArrayList();
List<Integer> list1 = new ArrayList<>();
List<Object> list2= new ArrayList<>();
list = list1;
//list2 = list1;   //compilation error

```

正如您所看到的，没有泛型的列表（称为原始类型）允许其引用引用任何其他具有任何泛型的列表，而具有泛型`<Object>`的列表不允许其变量引用具有任何其他泛型的列表。

Java 集合还允许通配符泛型`<?>`，它只允许将`null`分配给集合：

```java
List<?> list5= new ArrayList<>();
list5.add(null);
//list5.add(1);            //compilation error
//list5.add("ss");         //compilation error
//list5.add(new A());      //compilation error
//list5.add(new B());      //compilation error
System.out.println(list5); //prints: [null]
//list5.add(1, 42);        //compilation error

```

可以演示通配符泛型的用法示例。假设我们编写一个具有`List`（或任何集合）作为参数的方法，但我们希望确保此列表在方法内部不会被修改，而这会更改原始列表。这是一个例子：

```java
void doSomething(List<B> list){
  //some othe code goes here
  list.add(null);
  list.add(new B());
  list.add(0, new B());
  //some other code goes here
}
```

如果使用前面的方法，我们会得到一个不良的副作用：

```java
List<B> list= new ArrayList<>();
System.out.println(list); //prints: [B]
list.add(0, null);
System.out.println(list); //prints: [null, B]
doSomething(list);
System.out.println(list); //[B, null, B, null, B]

```

为了避免副作用，可以编写：

```java
void doSomething(List<?> list){
  list.add(null);
  //list.add(1);            //compilation error
  //list.add("ss");         //compilation error
  //list.add(new A());      //compilation error
  //list.add(new B());      //compilation error
  //list.add(0, 42);        //compilation error
}
```

正如您所看到的，这种方式列表无法修改，除了添加`null`。好吧，这是以删除泛型`<B>`的代价。现在，可能传入的列表包含不同类型的对象，类型转换`(B)`将抛出`ClassCastException`。没有免费的东西，但可能性是可用的。

就像封装一样，最佳实践建议尽可能使用尽可能窄（或专门的）类型的泛型。这可以确保意外行为的机会大大降低。

为了防止集合在方法内部被修改，可以使集合不可变。可以在方法内部或外部（在将其作为参数传递之前）进行。我们将向您展示如何在第十四章中执行此操作，*管理集合和数组*。

# 添加集合

`List`接口的两种方法允许将整个对象集合添加到现有列表中：

+   `addAll(Collection<? extends E> collection)`: 它将提供的对象集合添加到列表的末尾。

+   `addAll(int index, Collection<? extends E> collection)`: 它将提供的元素插入到列表的指定位置。该操作将当前在该位置的元素（如果有）和任何后续元素向右移动（将它们的索引增加提供的集合的大小）。

如果出现问题，这两种方法都会抛出几个`RuntimeExceptions`。此外，这两种方法都返回一个布尔值：

+   `false`：如果此方法调用后列表未更改

+   `true`：如果列表已更改

与添加单个元素的情况一样，这些方法的所有`List`实现很可能总是返回 true，因为`List`允许重复，而`Set`不允许（我们将很快讨论这一点）。

如果您在几页前阅读了泛型的描述，您可以猜到符号`Collection<? extends E>`的含义。泛型`<? extends E>`表示的是`E`或`E`的子类类型，其中`E`是用作集合泛型的类型。与我们之前的例子相关，观察以下类`A`和`B`：

```java
class A {
  @Override
  public String toString() { return "A"; }
}
class B extends A {
  @Override
  public String toString() { return "B"; }
}
```

我们可以向`List<A>`对象添加`A`类和`B`类的对象。

符号`addAll(Collection<? extends E> collection)`表示此方法允许向`List<E>`对象添加类型为`E`或`E`的任何子类型的对象。

例如，我们可以这样做：

```java
List<A> list = new ArrayList<>();
list.add(new A());
List<B> list1 = new ArrayList<>();
list1.add(new B());
list.addAll(list1);
System.out.println(list);    //prints: [A, B]
```

`addAll(int index, Collection<? extends E> collection)`方法的作用非常相似，但是只从指定的索引开始。当然，提供的索引值应该等于 0 或小于列表的长度。

对于`addAll(int index, Collection<? extends E> collection)`方法，提供的索引值应该等于 0 或小于列表的长度。

# 实现 equals()和 hashCode()

这是一个非常重要的子部分，因为在创建类时，程序员往往会专注于主要功能，忘记实现`equals()`和`hashCode()`方法。直到使用`equals()`方法比较对象或将对象添加到集合并进行搜索或假定为唯一（在`Set`的情况下）时，才会出现问题。

正如我们在第九章中所演示的，*运算符、表达式和语句*，`equals()`方法可用于对象标识。如果一个类没有覆盖基类`Object`中`equals()`方法的默认实现，那么每个对象都是唯一的。即使两个相同类的对象具有相同的状态，默认的`equals()`方法也会将它们报告为不同。因此，如果您需要将相同状态的同一类的两个对象视为相等，您必须在该类中实现`equals()`方法，以覆盖`Object`中的默认实现。

由于每个 Java 集合在搜索其元素时都使用`equals()`方法，因此您必须实现它，因为典型的业务逻辑要求在对象标识过程中包含对象状态或至少一些状态值。您还必须决定在类继承链的哪个级别应该将两个子类视为相等，就像我们在第九章中讨论的那样，*运算符、表达式和语句*，比较`PersonWithHair`和`PersonWithHairDressed`类的对象时；两者都扩展了`Person`类。我们当时决定，如果这些类的对象根据`Person`类中实现的`equals()`方法相等，则这些类的对象代表同一个人。我们还决定仅考虑`age`和`name`字段，尽管`Person`类可能有其他几个字段（例如`currentAddress`），这些字段对于人的标识并不相关。

因此，如果您期望创建的类将用于生成将用作某些 Java 集合成员的对象，则最好实现`equals()`方法。要做到这一点，您必须做出两个决定：

+   在类继承层次结构的哪个级别实现该方法

+   对象的哪些字段（换句话说，对象状态的哪些方面）包括在考虑中

`equals()`方法被 Java 集合用于元素识别。在实现`equals()`方法时，考虑在一个父类中进行，并决定在比较两个对象时使用哪些字段。

`hashCode()`方法未被`List`实现使用，因此我们将在接下来的代码中更详细地讨论它与接口`Set`和`Map`的实现相关。但是，由于我们正在讨论这个话题，我们想提到最佳的 Java 编程实践建议在实现`equals()`方法时每次都实现`hashCode()`方法。在这样做时，使用`equals()`方法使用的相同字段。例如，我们在第九章中实现的`Person`类，应该如下所示：

```java
class Person{
  private int age;
  private String name, currentAddress;
  public Person(int age, String name, String currAddr) {
    this.age = age;
    this.name = name;
    this.currentAddress = currAddr;
  }
  @Override
  public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null) return false;
    if(!(o instanceof Person)) return false;
      Person person = (Person)o;
      return age == person.getAge() &&
                Objects.equals(name, person.getName());
  }
  @Override
  public int hashCode(){
    return Objects.hash(age, name);
  }
}

```

如您所见，我们已经根据`Objects`类的`hash()`方法添加了`hashCode()`方法的实现，我们将在本章后面讨论。我们还添加了一个新字段，但是在`equals()`方法和`hashCode()`方法中都不会使用它，因为我们认为它与人的身份无关。

每次实现`equals()`方法时，也要实现`hashCode()`方法，因为您创建的类的对象可能不仅在`List`中使用，还可能在`Set`或`Map`中使用，这就需要实现`hashCode()`。

我们将在本章的相应部分讨论`Set`和`Map`的`hashCode()`方法实现的动机。我们还将解释为什么我们不能像使用`equals()`方法一样使用`hashCode()`来进行对象识别的目的。

# 定位元素

`List`接口中有三种方法允许检查列表中元素的存在和位置：

+   `contains(E)`:如果列表中存在提供的元素，则返回`true`。

+   `indexOf(E)`:它返回列表中提供的元素的索引（位置）。如果列表中有几个这样的元素，则返回最小的索引-等于提供的元素的最左边的第一个元素的索引。

+   `lastIndexOf(E)`:它返回列表中提供的元素的索引（位置）。如果列表中有几个这样的元素，则返回最大的索引-等于提供的元素的最后一个元素的索引。

以下是显示如何使用这些方法的代码：

```java
List<String> list = new ArrayList<>();
list.add("s1");
list.add("s2");
list.add("s1");

System.out.println(list.contains("s1"));    //prints: true
System.out.println(list.indexOf("s1"));     //prints: 0
System.out.println(list.lastIndexOf("s1")); //prints: 2

```

有两件事值得注意：

+   列表中的第一个元素的索引为 0（也像数组一样）

+   前面的方法依赖于`equals()`方法的实现来识别列表中提供的对象

# 检索元素

`List`接口中有两种允许检索元素的方法：

+   `get(index)`: 它返回具有提供索引的元素

+   `sublist(index1, index2)`: 它返回从`index1`开始的元素列表，直到`index2`之前的元素

以下代码演示了如何使用这些方法：

```java
List<String> list = new ArrayList<>();
list.add("s1");
list.add("s2");
list.add("s3");

System.out.println(list.get(1));       //prints: s2
System.out.println(list.subList(0,2)); //prints: [s1, s2]

```

# 删除元素

`List`接口中有四种方法可以删除列表中的元素：

+   `remove(index)`: 它删除具有提供索引的元素并返回被删除的元素

+   `remove(E)`: 它删除提供的元素并返回`true`，如果列表包含它

+   `removeAll(Collection)`: 它删除提供的元素并返回`true`，如果列表已更改

+   `retainAll(Collection)`: 它删除不在提供的集合中的所有元素，并返回`true`，如果列表已更改

我们想要提出这些方法的两点：

+   最后三种方法使用`equals()`方法来识别要删除或保留的元素

+   如果列表中有一个或多个元素被移除，剩余元素的索引将被重新计算

现在让我们看看代码示例：

```java
List<String> list = new ArrayList<>();
list.add("s1");
list.add("s2");
list.add("s3");
list.add("s1");

System.out.println(list.remove(1));    //prints: s2
System.out.println(list);              //prints: [s1, s3, s1]
//System.out.println(list.remove(5));  //throws IndexOutOfBoundsException
System.out.println(list.remove("s1")); //prints: true
System.out.println(list);              //prints: [s3, s1]
System.out.println(list.remove("s5")); //prints: false
System.out.println(list);              //prints: [s3, s1]

```

在前面的代码中，值得注意的是列表有两个元素`s1`，但是只有左边的第一个被语句`list.remove("s1")`移除：

```java
List<String> list = new ArrayList<>();
list.add("s1");
list.add("s2");
list.add("s3");
list.add("s1");

System.out.println(list.removeAll(List.of("s1", "s2", "s5")));   //true
System.out.println(list);                                        //[s3]
System.out.println(list.removeAll(List.of("s5")));               //false
System.out.println(list);                                        //[s3]

```

为了节省空间，我们使用`of()`方法创建一个列表，我们将在第十四章中讨论，*管理集合和数组*。与前面的例子相比，在前面的代码中语句`list.removeAll("s1","s2","s5")`移除了列表中的两个元素`s1`：

```java
List<String> list = new ArrayList<>();
list.add("s1");
list.add("s2");
list.add("s3");
list.add("s1");

System.out.println(list.retainAll(List.of("s1","s2","s5"))); //true
System.out.println(list);                                    //[s1, s2, s1]
System.out.println(list.retainAll(List.of("s1","s2","s5"))); //false
System.out.println(list);                                    //[s1, s2, s1]
System.out.println(list.retainAll(List.of("s5")));           //true
System.out.println(list);                                    //[]

```

请注意在前面的代码中，`retainAll()`方法第二次返回`false`，因为列表没有更改。还要注意语句`list.retainAll(List.of("s5")`如何清除列表，因为它的元素都不等于提供的元素。

# 替换元素

`List`接口中有两种允许替换列表中元素的方法：

+   `set(index, E)`: 它用提供的元素替换具有提供索引的元素

+   `replaceAll(UnaryOperator<E>)`: 它用提供的操作返回的结果替换列表的每个元素

以下是使用`set（）`方法的示例：

```java
List<String> list = new ArrayList<>();
list.add("s1");
list.add("s2");

list.set(1, null);
System.out.println(list);    //prints: [s1, null]

```

这很简单，似乎不需要任何评论。

第二种方法`replaceAll（）`基于函数`UnaryOperator <E>`-Java 8 中引入的 Java 功能接口之一。我们将在第十七章中讨论它，*Lambda 表达式和函数式编程*。现在，我们只是想展示代码示例。它们似乎相当简单，所以您应该能够理解它是如何工作的。假设我们从以下列表开始：

```java
List<String> list = new ArrayList<>();
list.add("s1");
list.add("s2");
list.add("s3");
list.add("s1");
```

以下是一些可能的元素修改（只需记住`replaceAll（）`方法用提供的函数返回的结果替换每个元素）：

```java
list.replaceAll(s -> s.toUpperCase()); //cannot process null
System.out.println(list);    //prints: [S1, S2, S3, S1]

list.replaceAll(s -> ("S1".equals(s) ? "S5" : null));
System.out.println(list);    //prints: [S5, null, null, S5]

list.replaceAll(s -> "a");
System.out.println(list);    //prints: [a, a, a, a]

list.replaceAll(s -> {
  String result;
  //write here any code you need to get the value
  // for the variable result based in the value of s
  System.out.println(s);   //prints "a" four times
  result = "42";
  return result;
});
System.out.println(list);    //prints: [42, 42, 42, 42]

```

在最后一个示例中，我们将操作的主体放在大括号`{}`中，并添加了一个显式的`return`语句，这样您就可以看到我们所说的操作返回的结果。

在使用`equals（）`方法将集合的元素与`String`文字或任何其他对象进行比较时，习惯上在文字上调用`equals（）`，例如`"s1"。equals（element）`，或者在您用来比较的对象上调用`equals（element）`，例如`someObject.equals（element）`。这有助于避免`NullPointerException`，以防集合具有`null`值。

可以将上述函数的示例重写如下：

```java
UnaryOperator<String> function = s -> s.toUpperCase();
list.replaceAll(function);

function = s -> ("S1".equals(s) ? "S5" : null);
list.replaceAll(function);

function = s -> "a";
list.replaceAll(function);

function = s -> {
  String result;
  //write here any code you need to get the value
  // for the variable result based in the value of s
  System.out.println(s);   //prints "a" four times
  result = "42";
  return result;
};
list.replaceAll(function);

```

这样，它们可以像任何其他参数一样传递，这就是函数式编程的威力。但是，我们将在第十七章中更多地讨论它，并解释所有的语法，*Lambda 表达式和函数式编程*。

# 排序字符串和数字类型

正如我们已经提到的，类型为`List`的集合保留了元素的顺序，因此自然地，它也有对元素进行排序的能力，`sort（Comparator <E>）`方法就是为此而服务的。这种方法是在 Java 8 引入函数式编程后才可能的。我们将在第十七章中讨论它，*Lambda 表达式和函数式编程*。

现在，我们将向您展示一些示例，并指出标准比较器的位置。我们从以下列表开始：

```java
List<String> list = new ArrayList<>();
list.add("s3");
list.add("s2");
list.add("ab");
//list.add(null); //throws NullPointerException for sorting
                  //     String.CASE_INSENSITIVE_ORDER
                  //     Comparator.naturalOrder()
                  //     Comparator.reverseOrder()
list.add("a");
list.add("Ab");
System.out.println(list);                //[s3, s2, ab, a, Ab]

```

以下是一些排序的示例：

```java
list.sort(String.CASE_INSENSITIVE_ORDER);
System.out.println(list);                //[a, ab, Ab, s2, s3]

list.sort(Comparator.naturalOrder());
System.out.println(list);               //[Ab, a, ab, s2, s3]

list.sort(Comparator.reverseOrder());
System.out.println(list);               //[Ab, a, ab, s2, s3]
```

前述的排序不是空安全的，正如前述的注释所指出的。您可以通过阅读有关前述比较器的 API 文档或仅通过尝试来了解这一点。即使在阅读文档后，人们通常也会尝试各种边缘情况，以更好地理解所描述的功能，并查看自己是否正确理解了描述。

还有处理`null`值的比较器：

```java
list.add(null);

list.sort(Comparator.nullsFirst(Comparator.naturalOrder()));
System.out.println(list);              //[null, Ab, a, ab, s2, s3]

list.sort(Comparator.nullsLast(Comparator.naturalOrder()));
System.out.println(list);              //[Ab, a, ab, s2, s3, null]

```

正如您所看到的，许多流行的比较器都可以在`java.util.Comparator`类的静态方法中找到。但是，如果您找不到所需的现成比较器，也可以编写自己的比较器。例如，假设我们需要对空值进行排序，使其像`String`值“null”一样。对于这种情况，我们可以编写一个自定义比较器：

```java
Comparator<String> comparator = (s1, s2) ->{
  String s = (s1 == null ? "null" : s1);
  return s.compareTo(s2);
};
list.sort(comparator);
System.out.println(list);              //[Ab, a, ab, null, s2, s3]
```

`Comparator`类中还有各种数字类型的比较器：

+   `comparingInt(ToIntFunction<? super T> keyExtractor)`

+   `comparingLong(ToLongFunction<? super T> keyExtractor)`

+   `comparingDouble(ToDoubleFunction<? super T> keyExtractor)`

我们将它们留给读者自行研究，如果需要使用这些方法进行数字比较。但是，似乎大多数主流程序员从不使用它们；我们演示的现成比较器通常已经足够了。

# 排序自定义对象

其中一个经常遇到的情况是需要对自定义对象进行排序，例如`Car`或`Person`类型。为此，有两种选择：

+   实现`Comparable`接口。它只有一个方法`compareTo（T）`，它接受相同类型的对象并返回负整数，零或正整数，如果此对象小于，等于或大于指定对象。这样的实现称为自然排序，因为实现`Comparable`接口的对象可以通过集合的`sort()`方法进行排序。前一小节的许多示例演示了它如何适用于`String`类型的对象。比较器由方法`naturalOrder()`，`reverseOrder()`，`nullFirst()`和`nullLast()`返回-它们都基于使用`compareTo()`实现。

+   实现一个外部比较器，使用`Comparator`类的静态`comparing()`方法比较集合元素类型的两个对象。

让我们看看每个前述选项的代码示例，并讨论每种方法的利弊。首先，增强`Person`，`PersonWithHair`和`PersonWithHairDressed`类，并实现`Comparable`接口：

```java
class Person implements Comparable<Person> {
  private int age;
  private String name, address;
  public Person(int age, String name, String address) {
    this.age = age;
    this.name = name == null ? "" : name;
    this.address = address;
  }
  @Override
  public int compareTo(Person p){
    return name.compareTo(p.getName());
  }
  @Override
  public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null) return false;
    if(!(o instanceof Person)) return false;
      Person person = (Person)o;
      return age == person.getAge() &&
                Objects.equals(name, person.getName());
  }
  @Override
  public int hashCode(){ return Objects.hash(age, name); }
  @Override
  public String toString() { return "Person{age=" + age +
                                   ", name=" + name + "}";
  }
}
```

As you can see, we have added another instance field, `address`, but do not use it in either the  `equals()`, `hashCode()`, or `compareTo()` methods. We did it just to show that it is completely up to you how to define the identity of the object of class `Person` and its children.  We also implemented the `toString()` method (which prints only the fields included in the identity), so we can identify each object when they are displayed. And we have implemented the method of the `Comparable` interface, `compareTo()`, which is going to be used for sorting. Right now it takes into account only the name, so when sorted, the objects will be ordered by name.

The children of class `Person` did not change:

```java
class PersonWithHair extends Person {
  private String hairstyle;
  public PersonWithHair(int age, String name, 
                        String address, String hairstyle) {
    super(age, name, address);
    this.hairstyle = hairstyle;
  }
}

class PersonWithHairDressed extends PersonWithHair{
  private String dress;
  public PersonWithHairDressed(int age, String name, 
           String address, String hairstyle, String dress) {
    super(age, name, address, hairstyle);
    this.dress = dress;
  }
}
```

Now we can create the list that we are going to sort:

```java
List<Person> list = new ArrayList<>();
list.add(new PersonWithHair(45, "Bill", "27 Main Street", 
                                                       "Pompadour"));
list.add(new PersonWithHair(42, "Kelly","15 Middle Street",  
                                                        "Ponytail"));
list.add(new PersonWithHairDressed(34, "Kelly", "10 Central Square",  
                                               "Pompadour", "Suit"));
list.add(new PersonWithHairDressed(25, "Courtney", "27 Main Street",  
                                              "Ponytail", "Tuxedo"));

list.forEach(System.out::println);

```

Execution of the preceding code produces the following output:

```java
Person{age=45, name=Bill}
Person{age=42, name=Kelly}
Person{age=34, name=Kelly}
Person{age=25, name=Courtney}
```

The persons are printed in the order they were added to the list. Now, let's sort them:

```java
list.sort(Comparator.naturalOrder());
list.forEach(System.out::println);

```

The new order looks as follows:

```java
Person{age=45, name=Bill}
Person{age=25, name=Courtney}
Person{age=42, name=Kelly}
Person{age=34, name=Kelly}
```

The objects are ordered alphabetically by name – that is how we have implemented the `compareTo()` method.

If we use the `reverseOrder()` comparator, the order shown be reversed:

```java
list.sort(Comparator.reverseOrder());
list.forEach(System.out::println);

```

This is what we see if we run the preceding code:

```java
Person{age=42, name=Kelly}
Person{age=34, name=Kelly}
Person{age=25, name=Courtney}
Person{age=45, name=Bill}
```

The order was reversed.

We can change our implementation of the `compareTo()` method and order the objects by age:

```java
@Override
public int compareTo(Person p){
  return age - p.getAge();
}
```

Or we can implement it so that the `Person` objects will be sorted by both fields – first by name, then by age:

```java
@Override
public int compareTo(Person p){
  int result = this.name.compareTo(p.getName());
  if (result != 0) {
    return result;
  }
  return this.age - p.getAge();
}
```

If we sort the list in natural order now, the result will be:

```java
Person{age=45, name=Bill}
Person{age=25, name=Courtney}
Person{age=34, name=Kelly}
Person{age=42, name=Kelly}
```

You can see that the objects are ordered by name, but two persons with the same name Kelly are ordered by age too.

That is the advantage of implementing the `Comparable` interface – the sorting is always performed the same way. But this is also a disadvantage because to change the order, one has to re-implement the class. Besides, it might be not possible if the `Person` class comes to us from a library, so we cannot modify its code.

In such cases, the second option—using the `Comparator.comparing()` method—comes to the rescue. And, by the way, we can do it even when the `Person` class does not implement the `Comparable` interface.

`Comparator.comparing()`方法接受一个函数作为参数。我们将在第十七章中更详细地讨论函数式编程，*Lambda 表达式和函数式编程*。现在，我们只会说`Comparator.comparing()`方法基于传递的字段（要排序的类的字段）生成一个比较器。让我们看一个例子：

```java
list.sort(Comparator.comparing(Person::getName));
list.forEach(System.out::println);

```

上面的代码按名称对`Person`对象进行排序。我们唯一需要做的修改是向`Person`类添加`getName()`方法。同样，如果我们添加`getAge()`方法，我们可以按年龄对`Person`对象进行排序：

```java
list.sort(Comparator.comparing(Person::getAge));
list.forEach(System.out::println);

```

或者我们可以按照两个字段对它们进行排序 - 正如我们在实现`Comparable`接口时所做的那样：

```java
list.sort(Comparator.comparing(Person::getName).thenComparing(Person::getAge));
list.forEach(System.out::println);

```

您可以看到在前面的代码中如何使用`thenComparing()`链接排序方法。

大多数类通常都有 getter 来访问字段值，因此通常不需要添加 getter，任何库类都可以这样排序。

# 与另一个集合进行比较

每个 Java 集合都实现了`equals()`方法，用于将其与另一个集合进行比较。在`List`的情况下，当两个列表被认为是相等的（方法`list1.equals(list2)`返回`true`）时：

+   每个集合的类型都是`List`

+   一个列表的每个元素都等于另一个列表中相同位置的元素

以下是说明它的代码：

```java
List<String> list1 = new ArrayList<>();
list1.add("s1");
list1.add("s2");

List<String> list2 = new ArrayList<>();
list2.add("s1");
list2.add("s2");

System.out.println(list1.equals(list2)); //prints: true
list2.sort(Comparator.reverseOrder());
System.out.println(list2);               //prints: [s2, s1]
System.out.println(list1.equals(list2)); //prints: false

```

如果两个列表相等，它们的`hashCode()`方法返回相同的整数值。但是`hashCode()`结果的相等并不能保证列表相等。我们将在下一节讨论`Set`集合中元素的`hashCode()`方法实现时讨论这个原因。

`List`接口（或实现`Collection`接口的任何集合）的`containsAll(Collection)`方法只有在提供的集合的所有元素都存在于列表中时才返回`true`。如果列表的大小和提供的集合的大小相等，我们可以确定比较的每个集合由相同（好吧，相等）的元素组成。但是它并不保证元素是相同类型的，因为它们可能是具有`equals()`方法的不同代的子类。

如果没有，我们可以使用前面在本节中描述的`retainAll(Collection)`和`removeAll(Collection)`方法找到差异。假设我们有两个如下的列表：

```java
List<String> list1 = new ArrayList<>();
list1.add("s1");
list1.add("s1");
list1.add("s2");
list1.add("s3");
list1.add("s4");

List<String> list2 = new ArrayList<>();
list2.add("s1");
list2.add("s2");
list2.add("s2");
list2.add("s5");

```

我们可以找出一个列表中哪些元素不在另一个列表中：

```java
List<String> list = new ArrayList<>(list1);
list.removeAll(list2);
System.out.println(list);    //prints: [s3, s4]

list = new ArrayList<>(list2);
list.removeAll(list1);
System.out.println(list);    //prints: [s5]

```

请注意我们创建了一个临时列表以避免破坏原始列表。

但是这个差异并不能告诉我们每个列表中可能存在的重复元素。为了找到它，我们可以使用`retainAll(Collection)`方法：

```java
List<String> list = new ArrayList<>(list1);
list.retainAll(list2);
System.out.println(list);    //prints: [s1, s1, s2]

list = new ArrayList<>(list2);
list.retainAll(list1);
System.out.println(list);    //prints: [s1, s2, s2]

```

现在我们完整地了解了这两个列表之间的区别。

另外，请注意，`retainAll(Collection)`方法可以用于识别属于每个列表的元素。

但是`retainAll(Collection)`和`removeAll(Collection)`都不能保证比较的列表和传入的集合包含相同类型的元素。它们可能是具有共同父级的子级混合，只在父级中实现了`equals()`方法，而父类型是列表和传入集合的类型。

# 转换为数组

有两种方法允许将列表转换为数组：

+   `toArray()`：它将列表转换为数组`Object[]`

+   `toArray(T[])`：它将列表转换为数组`T[]`，其中`T`是列表中元素的类型

这两种方法都保留了元素的顺序。这是演示代码，显示了如何做到这一点：

```java
List<String> list = new ArrayList<>();
list.add("s1");
list.add("s2");

Object[] arr1 = list.toArray();
for(Object o: arr1){
  System.out.print(o);       //prints: s1s2
}

String[] arr2 = list.toArray(new String[list.size()]);
for(String s: arr2){
  System.out.print(s);      //prints: s1s2
}
```

然而，还有另一种将列表或任何集合转换为数组的方法 - 使用流和函数式编程：

```java
Object[] arr3 = list.stream().toArray();
for (Object o : arr3) {
  System.out.print(o);       //prints: s1s2
}

String[] arr4 = list.stream().toArray(String[]::new);
for (String s : arr4) {
  System.out.print(s);       //prints: s1s2
}
```

流和函数式编程使许多传统的编码解决方案过时了。我们将在第十七章 *Lambda 表达式和函数式编程*和第十八章 *流和管道*中讨论这一点。

# 列表实现

有许多类实现了`List`接口，用于各种目的：

+   `ArrayList`：正如我们在本节中讨论的那样，它是迄今为止最受欢迎的`List`实现

+   `LinkedList`：提供快速添加和删除列表末尾的元素

+   `Stack`：为对象提供**后进先出**（**LIFO**）存储

+   在`List`接口的在线文档中引用了许多其他类。

# Set - HashSet 不允许重复

`Set`接口的创建是有原因的；它的设计目的是不允许重复元素。重复是使用`equals()`方法来识别的，该方法在类中实现，该类的对象是集合的元素。

# 更喜欢变量类型 Set

与`List`一样，对于保存对实现`Set`接口的类的对象引用的变量，使用类型`Set`是一种良好的编程实践，称为编码到接口。它确保客户端代码独立于任何特定的实现。因此，例如，编写`Set<Person> persons = new HashSet<>()`是一个好习惯。

# 为什么叫 HashSet？

在编程中，哈希值是一个代表一些数据的 32 位有符号整数。它在`HashTable`等数据结构中使用。在`HashTable`中创建记录后，其哈希值可以在以后快速找到和检索存储的数据。哈希值也称为哈希码、摘要或简单哈希。

在 Java 中，基本`Object`类中的`hashCode()`方法返回一个哈希值作为对象表示，但它不考虑任何子级字段的值。这意味着如果需要哈希值包括子对象状态，需要在该子类中实现`hashCode()`方法。

哈希值是一个代表一些数据的整数。在 Java 中，它代表一个对象。

`HashSet`类使用哈希值作为唯一键来存储和检索对象。尽管可能的整数数量很大，但对象的种类更多。因此，两个不相等的对象可能具有相同的哈希值。这就是为什么`HashSet`中的每个哈希值不是指向单个对象，而是潜在地指向一组对象（称为桶）。`HashSet`使用`equals()`方法解决这种冲突。

例如，`HashSet`对象中存储了类`A`的多个对象，你想知道类`A`的特定对象是否存在。你可以在`HashSet`对象上调用`contains(object of A)`方法。该方法计算提供的对象的哈希值，并查找具有这样一个键的桶。如果没有找到，则`contains()`方法返回`false`。但是，如果存在具有这样的哈希值的桶，它可能包含多个类`A`的对象。这时就需要使用`equals()`方法。代码使用`equals()`方法将桶中的每个对象与提供的对象进行比较。如果其中一个`equals()`调用返回`true`，`contains()`方法返回`true`，从而确认已经存在这样的对象。否则，它返回`false`。

因此，正如我们在讨论`List`接口时已经提到的那样，如果你创建的类的对象将成为集合的元素，那么实现`equals()`和`hashCode()`方法并使用相同的实例字段非常重要。由于`List`不使用哈希值，因此可以使用`List`接口来处理在`Object`的子类中没有实现`hashCode()`方法的对象。但是，任何在名称中带有“Hash”的集合如果没有实现`hashCode()`方法将无法正常工作。因此，名称。

# 添加元素

`Set`接口只提供了一个方法来添加单个元素：

+   `add(E)`：如果集合中不存在这样一个元素`E2`，使得语句`Objects.equals(E1, E2)`返回`true`，则将提供的元素`E1`添加到集合中

`Objects`类是一个位于`java.util`包中的实用类。它的`equals()`方法以一种空安全的方式比较两个对象，当两个对象都为`null`时返回`true`，否则使用`equals()`方法。我们将在本章的后续部分更多地讨论实用类`Objects`。

`add()`方法可能会在出现问题时抛出`RuntimeException`。因此，在该方法周围放置 try-catch 块可以使代码更加健壮（如果 catch 块不仅仅是重新抛出异常，而是执行一些有意义的操作）。在线上阅读`Set`接口 API 的描述，看看这个方法抛出的异常的名称以及它们可能发生的条件。

`add(E)`方法也返回一个布尔值（`true`/`false`），表示操作的成功。这个方法覆盖了`Collection`接口中的方法，因此所有扩展或实现`Collection`接口的 Java 集合都有这个方法。让我们看一个例子：

```java
Set<String> set = new HashSet<>();
System.out.println(set.add("s1"));  //prints: true
System.out.println(set.add("s1"));  //prints: false
System.out.println(set.add("s2"));  //prints: true
System.out.println(set.add("s3"));  //prints: true
System.out.println(set);            //prints: [s3, s1, s2]  

```

注意，当我们尝试第二次添加元素`s1`时，`add()`方法返回`false`。然后看上面代码的最后一行，观察以下内容：

+   只有一个元素`s1`被添加到集合中

+   元素的顺序不被保留

最后一点观察很重要。Java 规范明确规定，与`List`相反，`Set`不保证元素的顺序。当在不同的 JVM 实例上运行相同的代码，甚至在同一实例的不同运行时，顺序可能会有所不同。工厂方法`Set.of()`在创建无序集合时会对其进行轻微的洗牌（我们将在第十四章中讨论集合工厂方法和数组）。这样，可以在将代码部署到生产环境之前更早地发现对`Set`和其他无序集合元素特定顺序的不恰当依赖。

# size()，isEmpty()和 clear()

这三种方法很直接：

+   `size()`: 它返回集合中元素的数量

+   `isEmpty()`: 如果列表中没有元素，则返回`true`（`size()`返回 0）

+   `clear()`: 它从列表中删除所有元素，使`isEmpty()`返回`true`，`size()`返回 0

# 迭代和流

这个`Set`功能与之前描述的`List`没有区别，因为实现`Collection`接口的每个集合也实现了`Iterable`接口（因为`Collection`扩展了`Iterable`）。可以使用传统的增强`for`语句或其自己的方法`forEach()`来迭代`Set`：

```java
Set set = new HashSet();
set.add(null);
set.add(1);
set.add("ss");
set.add(new A());
set.add(new B());
for(Object o: set){
  System.out.println(o);
}
set.forEach(System.out::println);
```

在第十七章中，*Lambda 表达式和函数式编程*，我们将解释如何将函数作为`forEach()`方法的参数传递。两种迭代样式的结果是相同的：

![](img/acd75458-ede5-4fa8-938a-ec78ec470e33.png)

与`List`一样，来自`Iterable`接口的其他相关方法也与`List`接口中的方法相同：

+   `iterator()`: 它返回一个`Iterator`类的对象，允许遍历（迭代）集合的每个元素并根据需要操作每个元素

+   `splititerator()`: 它返回一个`Splititerator`类的对象，允许对集合进行并行处理（讨论此功能超出了本书的范围）

`iterator()`方法返回的`Iterator`接口具有以下方法：

+   `next()`: 它返回迭代中的下一个元素

+   `hasNext ()`: 如果迭代还有更多元素，则返回`true`

+   `forEachRemaining (Consumer<? super E> function)`: 它将提供的函数应用于剩余的每个元素

+   `remove()`: 它从基础集合中删除迭代器返回的最后一个元素

`next()`和`hasNext()`方法是由`for`语句在后台使用的。

还可以使用`Stream`类的对象对集合元素进行迭代，这可以通过`stream()`和`parallelStream()`方法获得。我们将在第十八章中展示如何做到这一点，*流和管道*。

# 使用泛型添加

与`List`一样，泛型也可以与`Set`一起使用（或者任何集合都可以）。泛型的规则和`Set`的行为与在*List - ArrayList 保留顺序*部分中描述的`List`完全相同。

# 添加集合

`addAll(Collection<? extends E> collection)`方法将提供的对象集合添加到集合中，但仅添加那些尚未存在于集合中的对象。如果集合发生了变化，则该方法返回布尔值`true`，否则返回`false`。

泛型`<? extends E>`表示类型`E`或`E`的任何子类型。

例如，我们可以这样做：

```java
Set<String> set1 = new HashSet<>();
set1.add("s1");
set1.add("s2");
set1.add("s3");

List<String> list = new ArrayList<>();
list.add("s1");

System.out.println(set1.addAll(list)); //prints: false
System.out.println(set1);              //prints: [s3, s1, s2]

list.add("s4");
System.out.println(set1.addAll(list)); //prints: true
System.out.println(set1);              //prints: [s3, s4, s1, s2] 
```

# 实现 equals()和 hashCode()

我们已经多次谈到了实现`equals()`和`hashCode()`方法，这里只会重复一遍，如果你的类要作为`Set`元素使用，那么这两个方法都必须被实现。在前面的*为什么叫 HashSet？*部分有解释。

# 定位元素

与`List`相比，`Set`与直接定位特定元素相关的唯一功能是由`contains(E)`方法提供的，如果提供的元素存在于集合中，则返回`true`。您也可以迭代并以这种方式定位元素，使用`equals()`方法，但这不是直接定位。

# 检索元素

与`List`相比，不可能直接从`Set`中检索元素，因为您不能使用索引或其他方式指向对象。但是可以像之前描述的那样遍历集合，即在*迭代和流*子节中。

# 删除元素

`Set`接口中有四种删除元素的方法：

+   `remove(E)`: 它删除提供的元素并返回`true`如果列表包含它

+   `removeAll(Collection)`: 它删除提供的元素并返回`true`如果列表已更改

+   `retainAll(Collection)`: 它删除不在提供的集合中的所有元素并返回`true`如果列表已更改

+   `removeIf(Predicate<? super E> filter)`: 它删除所有满足提供的谓词返回`true`的元素

前三种方法的行为方式与`List`集合相同，因此我们不会重复解释（请参见*删除元素*部分的*List - ArrayList 保留顺序*部分）。

至于列出的方法中的最后一个，谓词是返回布尔值的函数。这是函数接口（只有一个抽象方法的接口）和函数式编程的另一个例子。

符号`Predicate<? super E>`表示接受类型为`E`或其基类（父类）的参数并返回布尔值的函数。

我们将在第十七章中更多地讨论函数，*Lambda 表达式和函数式编程*。与此同时，以下示例显示了如何使用`removeIf()`方法：

```java
Set<String> set = new HashSet();
set.add(null);
set.add("s1");
set.add("s1");
set.add("s2");
set.add("s3");
set.add("s4");
System.out.println(set);    //[null, s3, s4, s1, s2]

set.removeIf(e -> "s1".equals(e));
System.out.println(set);   //[null, s3, s4, s2]

set.removeIf(e -> e == null);
System.out.println(set);    //[s3, s4, s2] 
```

请注意，当尝试查找等于`s1`的元素`e`时，我们将`s1`放在第一位。这与我们在英语中表达的方式不一样，但它有助于避免`NullPointerException`，以防其中一个元素是`null`（就像我们的情况一样）。

# 替换元素

与`List`相反，不可能直接替换`Set`中的元素，因为您不能使用索引或其他方式指向对象。但是可以像之前描述的那样遍历集合，或者使用`Stream`对象（我们将在第十八章中讨论这一点，*流和管道)*，检查每个元素并查看这是否是您要替换的元素。那些不符合条件的元素，您可以添加到一个新的集合。而那些您想要替换的元素，则跳过并将另一个对象（将替换您跳过的对象）添加到新集合中：

```java
Set<String> set = new HashSet();
set.add(null);
set.add("s2");
set.add("s3");
System.out.println(set);    //[null, s3, s2]

//We want to replace s2 with s5
Set<String> newSet = new HashSet<>();
set.forEach(s -> {
  if("s2".equals(s)){
    newSet.add("s5");
  } else {
     newSet.add(s);
  }
});
set = newSet;
System.out.println(set);    //[null, s3, s5]

```

在我们从原始集合切换到新集合的引用后（`set = newSet`），原始集合最终将被垃圾收集器从内存中删除，结果将与我们只是替换原始集合中的一个元素一样。

# 排序

`Set`接口不允许排序，也不保证顺序保留。如果需要将这些功能添加到集合中，可以使用接口`java.util.SortedSet`或`java.util.NavigableSet`及其实现`java.util.TreeSet`或`java.util.ConcurrentSkipListSet`。

# 与另一个集合进行比较

每个 Java 集合都实现了`equals()`方法，用于将其与另一个集合进行比较。在`Set`的情况下，当两个集合被认为是相等的（`set1.equals(set2)`返回`true`）时：

+   每个集合的类型都是`Set`

+   它们的大小相同

+   一个集合的每个元素都包含在另一个集合中

以下代码说明了定义：

```java
Set<String> set1 = new HashSet<>();
set1.add("s1");
set1.add("s2");

List<String> list = new ArrayList<>();
list.add("s2");
list.add("s1");

System.out.println(set1.equals(list)); //prints: false 
```

前面的集合不相等，因为它们的类型不同。现在，让我们比较两个集合：

```java
Set<String> set2 = new HashSet<>();
set2.add("s3");
set2.add("s1");

System.out.println(set1.equals(set2)); //prints: false

set2.remove("s3");
set2.add("s2");
System.out.println(set1.equals(set2)); //prints: true
```

前面的集合根据其元素的组成而不同，即使集合的大小相同。

如果两个集合相等，则它们的`hashCode()`方法返回相同的整数值。但`hashCode()`结果的相等并不保证集合相等。我们已经在前面的子节*实现 equals()和 hashCode()*中讨论了这个原因。

`Set`接口的`containsAll(Collection)`方法（或者任何实现`Collection`接口的集合）仅在提供的集合的所有元素都存在于集合中时才返回`true`。如果集合的大小和提供的集合的大小相等，我们可以确定比较的集合的每个元素是相同的（好吧，相等的）。但这并不保证元素是相同类型的，因为它们可能是具有`equals()`方法的不同代的子代。

如果不是，我们可以使用`retainAll(Collection)`和`removeAll(Collection)`方法来查找差异，这些方法在本节前面已经描述过。假设我们有两个列表如下：

```java
Set<String> set1 = new HashSet<>();
set1.add("s1");
set1.add("s1");
set1.add("s2");
set1.add("s3");
set1.add("s4");

Set<String> set2 = new HashSet<>();
set2.add("s1");
set2.add("s2");
set2.add("s2");
set2.add("s5"); 

```

我们可以找到一个集合中不在另一个集合中的元素：

```java
Set<String> set = new HashSet<>(set1);
set.removeAll(set2);
System.out.println(set);    //prints: [s3, s4]

set = new HashSet<>(set2);
set.removeAll(set1);
System.out.println(set);    //prints: [s5] 
```

请注意，我们创建了一个临时集合以避免破坏原始集合。

由于`Set`不允许重复元素，因此无需使用`retainAll(Collection)`方法来查找集合之间的更多差异，就像我们为`List`所做的那样。相反，`retainAll(Collection)`方法可用于查找两个集合中的公共元素：

```java
Set<String> set = new HashSet<>(set1);
set.retainAll(set2);
System.out.println(set);    //prints: [s1, s2]

set = new HashSet<>(set2);
set.retainAll(set1);
System.out.println(set);    //prints: [s1, s2]
```

正如您从前面的代码中可以看到的，要找到两个集合之间的公共元素，只需要使用`retainAll()`方法一次就足够了，无论哪个集合是主集合，哪个是参数集合。

另外，请注意，`retainAll(Collection)`方法和`removeAll(Collection)`方法都不能保证比较的集合和传入的集合包含相同类型的元素。它们可能是具有共同父类的子类的混合体，只有父类中实现了`equals()`方法，而父类类型是集合和传入的集合的类型。

# 转换为数组

有两种方法允许将集合转换为数组：

+   `toArray()`: 将集合转换为数组`Object[]`

+   `toArray(T[])`: 将集合转换为数组`T[]`，其中`T`是集合中元素的类型

这两种方法只有在集合保持顺序的情况下才能保留元素的顺序，例如`SortedSet`或`NavigableSet`。以下是演示代码，显示了如何执行此操作：

```java
Set<String> set = new HashSet<>();
set.add("s1");
set.add("s2");

Object[] arr1 = set.toArray();
for(Object o: arr1){
  System.out.print(o);       //prints: s1s2
}

String[] arr2 = set.toArray(new String[set.size()]);

for(String s: arr2){
  System.out.print(s);     //prints: s1s2
}
```

然而，还有另一种将集合或任何集合转换为数组的方法——使用流和函数式编程：

```java
Object[] arr3 = set.stream().toArray();
for (Object o : arr3) {
  System.out.print(o);       //prints: s1s2
}

String[] arr4 = set.stream().toArray(String[]::new);
for (String s : arr4) {
  System.out.print(s);       //prints: s1s2
}
```

流和函数式编程使许多传统的编码解决方案过时。我们将在第十七章 *Lambda 表达式和函数式编程*和第十八章 *流和管道*中讨论它们。

# 集合实现

有许多实现`Set`接口的类，用于各种目的：

+   我们在本节讨论了`HashMap`类；它是迄今为止最受欢迎的`Set`实现

+   `LinkedHashSet`类按顺序存储唯一元素

+   `TreeSet`类根据其值的自然顺序或在创建时提供的`Comparator`对其元素进行排序

+   `Set`接口的在线文档中引用了许多其他类

# Map – HashMap 通过键存储/检索对象

`Map`接口本身与`Collection`接口没有直接关联，但它使用`Set`接口作为其键和`Collection`作为其值。例如，对于`Map<Integer, String> map`：

+   `Set<Integer> keys = map.keySet();`

+   `Collection<String> values = map.values();`

每个值都存储在一个具有唯一键的映射中，当添加到映射中时，该键与值一起传递。在`Map<Integer, String> map`的情况下：

```java
map.put(42, "whatever");        //42 is the key for the value "whatever"
```

然后，稍后可以通过其键检索值：

```java
String v = map.get(42);
System.out.println(v);     //prints: whatever
```

这些是传达`Map`接口目的的基本映射操作——提供键值对的存储，其中键和值都是对象，并且用作键的类实现了`equals()`和`hashCode()`方法，这些方法覆盖了`Object`类中的默认实现。

现在，让我们更仔细地看看`Map`接口、它的实现和用法。

# 更喜欢变量类型 Map

与`List`和`Set`一样，使用类型`Map`来保存对实现`Map`接口的类的对象的引用的变量是一种良好的编程实践，称为编码到接口。它确保客户端代码不依赖于任何特定的实现。因此，将所有人员存储为值的`Map<String, Person> persons = new HashMap<>()`是一个很好的习惯-以他们的键-例如社会安全号码。

# 为什么称为 HashMap？

到目前为止，你应该已经很明显地意识到`HashMap`类之所以有“Hash”在其名称中，是因为它使用它们的哈希值存储键，这些哈希值是由`hashCode()`方法计算得出的。由于我们在前面的章节中已经对此进行了详细讨论，所以我们不打算在这里讨论哈希值及其用法；你可以回到本章的前一节*为什么称为 HashSet？*中参考。

# 添加和可能替换

`Map`接口存储键值对，也称为条目，因为在`Map`中，每个键值对也由`Entry`接口表示，它是`Map`的嵌套接口，因此被称为`Map.Entry`。

可以使用以下方法向`Map`添加键值对或条目：

+   `V put(K, V)`: 它添加一个键值对（或创建一个键值关联）。如果提供的键已经存在，则该方法覆盖旧值并返回它（如果旧值为`null`，则返回`null`）。如果提供的键尚未在映射中，则该方法返回`null`。

+   `V putIfAbsent(K, V)`: 它添加一个键值对（或创建一个键值关联）。如果提供的键已经存在并且关联的值不是`null`，则该方法不会覆盖旧值，而只是返回它。如果提供的键尚未在映射中或者关联的值为`null`，则该方法将覆盖旧值并返回`null`。

以下代码演示了所描述的功能：

```java
Map<Integer, String> map = new HashMap<>();
System.out.println(map.put(1, null));  //prints: null
System.out.println(map.put(1, "s1"));  //prints: null
System.out.println(map.put(2, "s1"));  //prints: null
System.out.println(map.put(2, "s2"));  //prints: s1
System.out.println(map.put(3, "s3"));  //prints: null
System.out.println(map);               //prints: {1=s1, 2=s2, 3=s3}

System.out.println(map.putIfAbsent(1, "s4"));  //prints: s1
System.out.println(map);               //prints: {1=s1, 2=s2, 3=s3}

System.out.println(map.put(1, null));  //prints: s1
System.out.println(map);               //prints: {1=null, 2=s2, 3=s3}

System.out.println(map.putIfAbsent(1, "s4"));  //prints: null
System.out.println(map);               //prints: {1=s4, 2=s2, 3=s3}

System.out.println(map.putIfAbsent(4, "s4"));  //prints: null
System.out.println(map);               //prints: {1=s4, 2=s2, 3=s3, 4=s4}

```

请注意，在返回值为`null`的情况下，结果存在一些歧义-新条目是否已添加（并替换了值为`null`的条目）或者没有。因此，在使用描述的方法时，必须注意可能包含`null`值的映射。

还有`compute()`和`merge()`方法，它们允许您向映射中添加和修改数据，但它们的使用对于入门课程来说过于复杂，因此我们将它们排除在讨论之外。此外，它们在主流编程中并不经常使用。

# size()，isEmpty()和 clear()

这三种方法很简单：

+   `size()`: 它返回映射中键值对（条目）的计数

+   `isEmpty()`: 如果映射中没有键值对（条目），则返回`true`（`size()`返回 0）

+   `clear()`: 它从映射中删除所有键值对（条目），使得`isEmpty()`返回`true`，`size()`返回 0

# 迭代和流

有几种方法可以使用以下`Map`方法对映射内容进行迭代，例如：

+   `Set<K> keySet()`: 它返回与地图中存储的值相关联的键。可以遍历此集合，并从地图中检索每个键的值（请参阅前面的部分，了解如何遍历集合）。

+   `Collection<V> values()`: 它返回地图中存储的值。可以遍历此集合。

+   `Set<Map.Entry<K,V>> entrySet()`: 它返回地图中存储的条目（键值对）。可以遍历此集合，并从每个条目中获取键和值。

+   `forEach (BiConsumer<? super K,? super V> action)`: 它遍历存储在地图中的键值对，并将它们作为输入提供给函数`BiConsumer`，该函数接受地图键和值类型的两个参数，并返回`void`。

以下是如何阅读符号`BiConsumer<? super K,? super V>`的方法：它描述了一个函数。函数名称中的`Bi`表示它接受两个参数：一个是类型为`K`或其任何超类的参数，另一个是类型为`V`或其任何超类的参数。函数名称中的`Consumer`表示它不返回任何内容（`void`）。

我们将在第十七章中更多地讨论函数式编程，*Lambda 表达式和函数式编程*。

为了演示前面的方法，我们将使用以下地图：

```java
Map<Integer, String> map = new HashMap<>();
map.put(1, null);
map.put(2, "s2");
map.put(3, "s3");
```

以下是如何迭代此地图的方法：

```java
for(Integer key: map.keySet()){
  System.out.println("key=" + key + ", value=" + map.get(key));
}
map.keySet().stream()
   .forEach(k->System.out.println("key=" + k + ", value=" + map.get(k)));
for(String value: map.values()){
  System.out.println("value=" + value);
}
map.values().stream().forEach(System.out::println);
map.forEach((k,v) -> System.out.println("key=" + k + ", value=" + v));
map.entrySet().forEach(e -> System.out.println("key=" + e.getKey() + 
                                          ", value=" + e.getValue()));

```

所有前面的方法产生相同的结果，除了`values()`方法，它只返回值。使用哪一个取决于风格，但似乎`map.forEach()`需要较少的按键来实现迭代。

# 使用泛型添加

从我们的示例中，您已经看到了如何将泛型与`Map`一起使用。它通过允许编译器检查地图和尝试存储在其中的对象之间的匹配，为程序员提供了宝贵的帮助。

# 添加另一个地图

`putAll(Map<? extends K, ? extends V> map)`方法从提供的地图中添加每个键值对，就像`put(K, V)`方法对一个键值对一样：

```java
Map<Integer, String> map1 = new HashMap<>();
map1.put(1, null);
map1.put(2, "s2");
map1.put(3, "s3");

Map<Integer, String> map2 = new HashMap<>();
map2.put(1, "s1");
map2.put(2, null);
map2.put(4, "s4");

map1.putAll(map2);
System.out.println(map1); //prints: {1=s1, 2=null, 3=s3, 4=s4}

```

正如您所看到的，`putAll()`方法添加了一个新的键值对，或者覆盖了现有键值对中的值（基于键），并且不返回任何内容。

# 实现 equals()和 hashCode()

如果您要将您编写的类用作`Map`中的键，实现`equals()`和`hashCode()`方法非常重要。请参阅前面部分中的解释，*为什么称为 HashSet？*在我们的示例中，我们已经使用了`Integer`类的对象作为键。该类根据类的整数值实现了这两种方法。

作为`Map`中的值存储的类必须至少实现`equals()`方法（请参阅下一小节*定位元素*）。

# 定位元素

以下两种方法回答了特定键或值是否存在于地图中的问题：

+   `containsKey(K)`: 如果提供的键已经存在，则返回`true`

+   `containsValue(V)`: 如果提供的值已经存在，则返回`true`

这两种方法都依赖于`equals()`方法来识别匹配项。

# 检索元素

要从`Map`中检索元素，可以使用以下四种方法之一：

+   `V get(Object K)`:它返回提供的键的值，如果提供的键不在地图中，则返回`null`

+   `V getOrDefault(K, V)`:它返回提供的键的值，如果提供的键不在地图中，则返回提供的(默认)值

+   `Map.Entry<K,V> entry(K,V)`:将提供的`键-值对`转换为`Map.Entry`的不可变对象的静态方法(不可变意味着可以读取，但不能更改)

+   `Map<K,V> ofEntries(Map.Entry<? extends K,? extends V>... entries)`:它基于提供的条目创建一个不可变的地图

以下代码演示了这些方法：

```java
Map<Integer, String> map = new HashMap<>();
map.put(1, null);
map.put(2, "s2");
map.put(3, "s3");

System.out.println(map.get(2));                 //prints: s2
System.out.println(map.getOrDefault(2, "s4"));  //prints: s2
System.out.println(map.getOrDefault(4, "s4"));  //prints: s4

Map.Entry<Integer, String> entry = Map.entry(42, "s42");
System.out.println(entry);      //prints: 42=s42

Map<Integer, String> entries = 
                Map.ofEntries(entry, Map.entry(43, "s43"));   
System.out.println(entries);   //prints: {42=s42, 43=s43}

```

并且始终可以通过迭代来检索地图的元素，就像我们在*迭代和流*的子节中描述的那样。

# 删除元素

两种方法允许直接删除 Map 元素：

+   `V remove(Object key)`:它删除与键关联的对象并返回其值，或者，如果地图中不存在这样的键，则返回`null`

+   `boolean remove(Object key, Object value)`:它只有在当前与键关联的值等于提供的值时，才删除与键关联的对象；如果元素被删除，则返回`true`

以下是说明所述行为的代码：

```java
Map<Integer, String> map = new HashMap<>();
map.put(1, null);
map.put(2, "s2");
map.put(3, "s3");
System.out.println(map.remove(2));        //prints: s2
System.out.println(map);                  //prints: {1=null, 3=s3}
System.out.println(map.remove(4));        //prints: null
System.out.println(map);                  //prints: {1=null, 3=s3}
System.out.println(map.remove(3, "s4"));  //prints: false
System.out.println(map);                  //prints: {1=null, 3=s3}
System.out.println(map.remove(3, "s3"));  //prints: true
System.out.println(map);                  //prints: {1=null}

```

还有另一种通过键删除`Map`元素的方法。如果从地图中删除了键，则相应的值也将被删除。以下是演示它的代码：

```java
Map<Integer, String> map = new HashMap<>();
map.put(1, null);
map.put(2, "s2");
map.put(3, "s3");

Set<Integer> keys = map.keySet();

System.out.println(keys.remove(2));      //prints: true
System.out.println(map);                 //prints: {1=null, 3=s3}

System.out.println(keys.remove(4));      //prints: false
System.out.println(map);                 //prints: {1=null, 3=s3}

```

同样，还可以使用`Set`接口中的`removeAll(Collection)`、`retainAll(Collection)`和`removeIf(Predicate<? super E> filter)`方法，这些方法在*删除元素*的子节中有描述，*Set - HashSet 不允许重复**,*也可以使用。

# 替换元素

要替换`Map`的元素，可以使用以下方法：

+   `V replace(K, V)`:它只有在提供的键存在于地图中时，才用提供的值替换值；如果这样的键存在，则返回先前(替换的)值，如果这样的键不存在，则返回`null`

+   `boolean  replace(K, oldV, newV) `:它只有在提供的键存在于地图中并且当前与提供的值`oldV`关联时，才用新值`newV`替换当前值(`oldV`)；如果值被替换，则返回`true`

+   `void replaceAll(BiFunction<? super K, ? super V, ? extends V> function)`: 它允许您使用提供的函数替换值，该函数接受两个参数 - 键和值 - 并返回一个新值，该新值将替换此键值对中的当前值

以下是如何阅读符号`BiFunction<? super K, ? super V, ? extends V>`：它描述了一个函数。`Bi`中的函数名称表示它接受两个参数：一个是类型`K`或任何其超类，另一个是类型 V 或任何其超类。函数名称中的`Function`部分表示它返回某些东西。返回的值是最后列出的。在这种情况下，它是`<? extends V>`，这意味着类型`V`或其子类的值。

让我们假设我们要更改的地图如下：

```java
Map<Integer, String> map = new HashMap<>();
map.put(1, null);
map.put(2, "s2");
map.put(3, "s3");

```

然后，说明前两种方法的代码如下：

```java
System.out.println(map.replace(1, "s1"));   //prints: null
System.out.println(map);                    //prints: {1=s1, 2=s2, 3=s3}

System.out.println(map.replace(4, "s1"));   //prints: null
System.out.println(map);                    //prints: {1=s1, 2=s2, 3=s3}

System.out.println(map.replace(1, "s2", "s1"));   //prints: false
System.out.println(map);                    //prints: {1=s1, 2=s2, 3=s3}

System.out.println(map.replace(1, "s1", "s2"));   //prints: true
System.out.println(map);                    //prints: {1=s2, 2=s2, 3=s3}
```

这是帮助理解列出的最后一个替换方法的代码：

```java
Map<Integer, String> map = new HashMap<>();
map.put(1, null);
map.put(2, null);
map.put(3, "s3");

map.replaceAll((k,v) -> v == null? "s" + k : v);
System.out.println(map);                 //prints: {1=s1, 2=s2, 3=s3}

map.replaceAll((k,v) -> k == 2? "n2" : v);
System.out.println(map);                 //prints: {1=s1, 2=n2, 3=s3}

map.replaceAll((k,v) -> v.startsWith("s") ? "s" + (k + 10) : v);
System.out.println(map);                 //prints: {1=s11, 2=n2, 3=s13}

```

请注意，我们只能在用其他东西替换所有`null`值之后才能使用`v.startsWith()`方法。否则，这行可能会抛出`NullPointerException`，我们需要将其更改为以下行：

```java
map.replaceAll((k,v) -> (v != null && v.startsWith("s")) ? 
                                          "s" + (k + 10) : v);

```

# 排序

`Map`接口不允许排序，也不保证顺序保留。如果您需要将这些功能添加到地图中，可以使用接口`java.util.SortedMap`或`java.util.NavigableMap`，以及它们的实现`java.util.TreeMap`或`java.util.ConcurrentSkipListMap`。

# 与另一个集合比较

每个 Java 集合都实现了`equals()`方法，它将其与另一个集合进行比较。在`Map`的情况下，当两个地图被认为是相等的（`map1.equals(map2)`返回`true`）时：

+   两者都是`Map`对象

+   一个地图具有与另一个地图相同的键值对集

这是说明定义的代码：

```java
Map<Integer, String> map1 = new HashMap<>();
map1.put(1, null);
map1.put(2, "s2");
map1.put(3, "s3");

Map<Integer, String> map2 = new HashMap<>();
map2.put(1, null);
map2.put(2, "s2");
map2.put(3, "s3");

System.out.println(map2.equals(map1)); //prints: true

map2.put(1, "s1");
System.out.println(map2.equals(map1)); //prints: false

```

如果你仔细想一想，`map1.equals(map2)`方法返回的结果与`map1.entrySet().equals(map2.entrySet())`方法返回的结果完全相同，因为`entrySet()`方法返回“Set<Map.Entry<K,V>`，我们知道（请参见子部分*与另一个集合比较*）两个集合相等当一个集合的每个元素都包含在另一个集合中。

如果两个地图相等，则它们的`hashCode()`方法返回相同的整数值。但是，`hashCode()`结果的相等并不保证地图相等。我们在讨论`hashCode()`方法的实现时已经谈到了这一点，这是在上一节中讨论`Set`集合的元素时。

如果两个地图不相等，并且需要找出确切的差异，有多种方法可以做到这一点：

```java
map1.entrySet().containsAll(map2.entrySet());
map1.entrySet().retainAll(map2.entrySet());
map1.entrySet().removeAll(map2.entrySet());

map1.keySet().containsAll(map2.keySet());
map1.keySet().retainAll(map2.keySet());
map1.keySet().removeAll(map2.keySet());

map1.values().containsAll(map2.values());
map1.values().retainAll(map2.values());
map1.values().removeAll(map2.values());
```

使用这些方法的任意组合，可以全面了解两个地图之间的差异。

# 地图实现

有许多类实现了`Map`接口，用于各种目的：

+   我们在本节中讨论的`HashMap`；它是迄今为止最流行的`Map`实现

+   `LinkedHashMap`类按其插入顺序存储其键值对

+   `TreeMap`类根据键的自然顺序或在创建时提供的`Comparator`对其键值对进行排序

+   许多其他类在`Map`接口的在线文档中有所提及。

# 练习-EnumSet 方法

我们没有讨论`java.util.EnumSet`集合。它是一个较少人知道但非常有用的类，在需要使用一些`enum`值时。在线查找其 API 并编写代码来演示其四种方法的用法：

+   `of()`

+   `complementOf()`

+   `allOf()`

+   `range()`

# 答案

假设`enum`类看起来像下面这样：

```java
enum Transport { AIRPLANE, BUS, CAR, TRAIN, TRUCK }
```

然后，演示`EnumSet`的四种方法的代码可能如下所示：

```java
EnumSet<Transport> set1 = EnumSet.allOf(Transport.class);
System.out.println(set1);   //prints: [AIRPLANE, BUS, CAR, TRAIN, TRUCK]

EnumSet<Transport> set2 = EnumSet.range(Transport.BUS, Transport.TRAIN);
System.out.println(set2);   //prints: [BUS, CAR, TRAIN]

EnumSet<Transport> set3 = EnumSet.of(Transport.BUS, Transport.TRUCK);
System.out.println(set3);   //prints: [BUS, TRUCK]

EnumSet<Transport> set4 = EnumSet.complementOf(set3);
System.out.println(set4);   //prints: [AIRPLANE, CAR, TRAIN]

```

# 总结

本章使读者熟悉了 Java 集合和最流行的集合接口-`List`，`Set`和`Map`。代码示例使它们的功能更加清晰。代码的注释吸引了读者对可能的陷阱和其他有用细节的注意。

在下一章中，我们将继续概述 Java 标准库和 Apache Commons 中最流行的类。其中大多数是实用程序，例如`Objects`，`Collections`，`StringUtils`和`ArrayUtils`。其他只是类，例如`StringBuilder`，`StringBuffer`和`LocalDateTime`。有些帮助管理集合；其他帮助管理对象。它们的共同之处在于它们属于每个 Java 程序员在成为有效编码人员之前必须掌握的一小组工具。
