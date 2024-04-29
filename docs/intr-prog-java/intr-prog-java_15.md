# 第十五章：管理对象、字符串、时间和随机数

在本章中我们将讨论的类，与前几章讨论的 Java 集合和数组一起属于每个程序员都必须掌握的一类（主要是来自 Java 标准库和 Apache Commons 的工具类），以成为一名高效的编码人员。它们也展示了各种软件设计和解决方案，具有指导意义，并可作为最佳编码实践的模式。

我们将涵盖以下功能领域：

+   管理对象

+   管理字符串

+   管理时间

+   管理随机数

概述的类列表包括：

+   `java.util.Objects`

+   `org.apache.commons.lang3.ObjectUtils`

+   `java.lang.String`

+   `org.apache.commons.lang3.StringUtils`

+   `java.time.LocalDate`

+   `java.time.LocalTime`

+   `java.time.LocalDateTime`

+   `java.lang.Math`

+   `java.util.Random`

# 管理对象

你可能不需要管理数组，甚至可能一段时间内不需要管理集合，但你无法避免管理对象，这意味着本节描述的类你可能每天都会使用。

尽管`java.util.Objects`类是在 2011 年（Java 7 发布时）添加到 Java 标准库中的，而`ObjectUtils`类自 2002 年以来就存在于 Apache Commons 库中，但它们的使用增长缓慢。这可能部分地可以解释它们最初的方法数量很少-2003 年`ObjectUtils`只有 6 个方法，2011 年`Objects`只有 9 个方法。然而，它们是非常有用的方法，可以使代码更易读、更健壮，减少错误的可能性。因此，为什么这些类从一开始就没有被更频繁地使用至今仍然是个谜。我们希望你能立即在你的第一个项目中开始使用它们。

# 类 java.util.Objects

类`Objects`只有 17 个方法-全部是静态的。在前一章中，当我们实现了类`Person`时，我们已经使用了其中的一些方法：

```java
class Person implements Comparable<Person> {
    private int age;
    private String name;
    public Person(int age, String name) {
        this.age = age;
        this.name = name == null ? "" : name;
    }
    public int getAge(){ return this.age; }
    public String getName(){ return this.name; }
    @Override
    public int compareTo(Person p){
        int result = this.name.compareTo(p.getName());
        if (result != 0) {
            return result;
        }
        return this.age - p.getAge();
    }
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null) return false;
        if(!(o instanceof Person)) return false;
        Person person = (Person)o;
        return age == person.getAge() &&
               Objects.equals(name, person.getName()); //line 25
    }
    @Override
    public int hashCode(){
        return Objects.hash(age, name);
    }
    @Override
    public String toString() {
        return "Person{age=" + age + ", name=" + name + "}";
    }
}
```

我们以前在`equals()`和`hashCode()`方法中使用了`Objects`类。一切都运行良好。但是，请注意我们如何检查前一个构造函数中的参数`name`。如果参数是`null`，我们将空的`String`值赋给字段`name`。我们这样做是为了避免第 25 行的`NullPointerException`。另一种方法是使用 Apache Commons 库中的`ObjectUtils`类。我们将在下一节中进行演示。`ObjectUtils`类的方法处理`null`值，并使将`null`参数转换为空的`String`变得不必要。

但首先，让我们回顾一下`Objects`类的方法。

# equals()和 deepEquals()

我们详细讨论了`equals()`方法的实现，但一直假设它是在一个非`null`的对象`obj`上调用的，`obj.equals(anotherObject)`的调用不会产生`NullPointerException`。

然而，有时我们需要比较两个对象`a`和`b`，当它们中的一个或两个可以是`null`时。以下是这种情况的典型代码：

```java
boolean equals(Object a, Object b) {
    return (a == b) || (a != null && a.equals(b));
}
```

这是`boolean Objects.equals(Object a, Object b)`方法的实际源代码。它允许使用方法`equals(Object)`比较两个对象，并处理其中一个或两个为`null`的情况。

`Objects`类的另一个相关方法是`boolean deepEquals(Object a, Object b)`。以下是其源代码：

```java
boolean deepEquals(Object a, Object b) {
    if (a == b)
        return true;
    else if (a == null || b == null)
        return false;
    else
        return Arrays.deepEquals0(a, b);
}
```

正如您所见，它是基于我们在前一节中讨论的`Arrays.deepEquals()`。这些方法的演示代码有助于理解它们之间的区别：

```java
Integer[] as1 = {1,2,3};
Integer[] as2 = {1,2,3};
System.out.println(Arrays.equals(as1, as2));        //prints: true
System.out.println(Arrays.deepEquals(as1, as2));    //prints: true

System.out.println(Objects.equals(as1, as2));        //prints: false
System.out.println(Objects.deepEquals(as1, as2));    //prints: true

Integer[][] aas1 = {{1,2,3},{1,2,3}};
Integer[][] aas2 = {{1,2,3},{1,2,3}};
System.out.println(Arrays.equals(aas1, aas2));       //prints: false
System.out.println(Arrays.deepEquals(aas1, aas2));   //prints: true

System.out.println(Objects.equals(aas1, aas2));       //prints: false
System.out.println(Objects.deepEquals(aas1, aas2));   //prints: true

```

在上述代码中，`Objects.equals(as1, as2)`和`Objects.equals(aas1, aas2)`返回`false`，因为数组无法覆盖`Object`类的`equals()`方法，而是通过引用而不是值进行比较。

方法`Arrays.equals(aas1, aas2)`返回`false`的原因相同：因为嵌套数组的元素是数组，通过引用进行比较。

总而言之，如果您想比较两个对象`a`和`b`的字段值，则：

+   如果它们不是数组且`a`不是`null`，请使用`a.equals(b)`

+   如果它们不是数组且两个对象都可以是`null`，请使用`Objects.equals(a, b)`

+   如果两者都可以是数组且都可以是`null`，请使用`Objects.deepEquals(a, b)`

也就是说，我们可以看到方法`Objects.deepEquals()`是最安全的方法，但这并不意味着您必须总是使用它。大多数情况下，您将知道要比较的对象是否可以为`null`或可以是数组，因此您也可以安全地使用其他`equals()`方法。

# hash()和 hashCode()

方法`hash()`或`hashCode()`返回的哈希值通常用作将对象存储在使用哈希的集合中的键，例如`HashSet()`。在`Object`超类中的默认实现基于内存中的对象引用。对于具有相同类的两个对象且具有相同实例字段值的情况，它返回不同的哈希值。因此，如果需要两个类实例具有相同状态的相同哈希值，则重写默认的`hashCode()`实现使用以下方法至关重要：

+   `int hashCode(Object value)`: 计算单个对象的哈希值

+   `int hash(Object... values)`: 计算对象数组的哈希值（请看我们在前面示例中的`Person`类中如何使用它）

请注意，当将同一对象用作方法`Objects.hash()`的单个输入数组时，这两种方法返回不同的哈希值：

```java
System.out.println(Objects.hash("s1"));           //prints: 3645
System.out.println(Objects.hashCode("s1"));       //prints: 3614

```

仅一个值会从两种方法中返回相同的哈希值：`null`

```java
System.out.println(Objects.hash(null));      //prints: 0
System.out.println(Objects.hashCode(null));  //prints: 0

```

当作为单个非空参数使用时，相同的值从方法`Objects.hashCode(Object value)`和`Objects.hash(Object... values)`返回的哈希值不同。值`null`从这些方法中返回相同的哈希值`0`。

使用类`Objects`进行哈希值计算的另一个优点是它能容忍`null`值，而在尝试对`null`引用调用实例方法`hashCode()`时会生成`NullPointerException`。

# isNull() 和 nonNull()

这两个方法只是对布尔表达式`obj == null`和`obj != null`的简单包装：

+   `boolean isNull(Object obj)`: 返回与`obj == null`相同的值。

+   `boolean nonNull(Object obj)`: 返回与`obj != null`相同的值。

这是演示代码：

```java
String object = null;

System.out.println(object == null);           //prints: true
System.out.println(Objects.isNull(object));   //prints: true

System.out.println(object != null);           //prints: false
System.out.println(Objects.nonNull(object));  //prints: false
```

# requireNonNull()

类`Objects`的以下方法检查第一个参数的值，如果值为`null`，则抛出`NullPointerException`或返回提供的默认值：

+   `T requireNonNull(T obj)`: 如果参数为`null`，则抛出没有消息的`NullPointerException`：

```java
      String object = null;
      try {
          Objects.requireNonNull(object);
      } catch (NullPointerException ex){
          System.out.println(ex.getMessage());  //prints: null
      }
```

+   `T requireNonNull(T obj, String message)`: 如果第一个参数为`null`，则抛出带有提供消息的`NullPointerException`：

```java
      String object = null;
      try {
          Objects.requireNonNull(object, "Parameter 'object' is null");
      } catch (NullPointerException ex){
          System.out.println(ex.getMessage());  
          //Parameter 'object' is null
      }
```

+   `T requireNonNull(T obj, Supplier<String> messageSupplier)`: 如果第一个参数为`null`，则返回由提供的函数生成的消息，如果生成的消息或函数本身为`null`，则抛出`NullPointerException`：

```java
      String object = null;
      Supplier<String> msg1 = () -> {
          String msg = "Msg from db";
          //get the corresponding message from database
          return msg;
      };
      try {
          Objects.requireNonNull(object, msg1);
      } catch (NullPointerException ex){
          System.out.println(ex.getMessage());  //prints: Msg from db
      }
      Supplier<String> msg2 = () -> null;
      try {
          Objects.requireNonNull(object, msg2);
      } catch (NullPointerException ex){
          System.out.println(ex.getMessage());  //prints: null
      }
      Supplier<String> msg3 = null;
      try {
          Objects.requireNonNull(object, msg3);
      } catch (NullPointerException ex){
          System.out.println(ex.getMessage());  //prints: null
      }
```

+   `T requireNonNullElse(T obj, T defaultObj)`: 如果第一个参数非空，则返回第一个参数的值，如果第二个参数非空，则返回第二个参数的值，如果都为空，则抛出带有消息`defaultObj`的`NullPointerException`：

```java
      String object = null;
      System.out.println(Objects.requireNonNullElse(object, 
                              "Default value"));   
                              //prints: Default value
      try {
          Objects.requireNonNullElse(object, null);
      } catch (NullPointerException ex){
          System.out.println(ex.getMessage());     //prints: defaultObj
      }
```

+   `T requireNonNullElseGet(T obj, Supplier<? extends T> supplier)`: 如果第一个参数非空，则返回第一个参数的值，否则返回由提供的函数生成的对象，如果都为空，则抛出带有消息`defaultObj`的`NullPointerException`：

```java
      String object = null;
      Supplier<String> msg1 = () -> {
          String msg = "Msg from db";
          //get the corresponding message from database
          return msg;
      };
      String s = Objects.requireNonNullElseGet(object, msg1);
      System.out.println(s);                //prints: Msg from db

      Supplier<String> msg2 = () -> null;
      try {
       System.out.println(Objects.requireNonNullElseGet(object, msg2));
      } catch (NullPointerException ex){
       System.out.println(ex.getMessage()); //prints: supplier.get()
      }
      try {
       System.out.println(Objects.requireNonNullElseGet(object, null));
      } catch (NullPointerException ex){
       System.out.println(ex.getMessage()); //prints: supplier
      }
```

# checkIndex()

以下一组方法检查集合或数组的索引和长度是否兼容：

+   `int checkIndex(int index, int length)`: 如果提供的`index`大于`length - 1`，则抛出`IndexOutOfBoundsException`。

+   `int checkFromIndexSize(int fromIndex, int size, int length)`: 如果提供的`index + size`大于`length - 1`，则抛出`IndexOutOfBoundsException`。

+   `int checkFromToIndex(int fromIndex, int toIndex, int length)`: 如果提供的`fromIndex`大于`toIndex`，或`toIndex`大于`length - 1`，则抛出`IndexOutOfBoundsException`。

这是演示代码：

```java
List<String> list = List.of("s0", "s1");
try {
    Objects.checkIndex(3, list.size());
} catch (IndexOutOfBoundsException ex){
    System.out.println(ex.getMessage());  
                         //prints: Index 3 out-of-bounds for length 2
}
try {
    Objects.checkFromIndexSize(1, 3, list.size());
} catch (IndexOutOfBoundsException ex){
    System.out.println(ex.getMessage());  
                //prints: Range [1, 1 + 3) out-of-bounds for length 2
}

try {
    Objects.checkFromToIndex(1, 3, list.size());
} catch (IndexOutOfBoundsException ex){
    System.out.println(ex.getMessage());  
                    //prints: Range [1, 3) out-of-bounds for length 2
}
```

# compare()

类`Objects`的方法`int compare(T a, T b, Comparator<T> c)`使用提供的比较器的方法`compare(T o1, T o2)`来比较两个对象。我们已经在谈论排序集合时描述了`compare(T o1, T o2)`方法的行为，因此应该期望以下结果：

```java
int diff = Objects.compare("a", "c", Comparator.naturalOrder());
System.out.println(diff);  //prints: -2
diff = Objects.compare("a", "c", Comparator.reverseOrder());
System.out.println(diff);  //prints: 2
diff = Objects.compare(3, 5, Comparator.naturalOrder());
System.out.println(diff);  //prints: -1
diff = Objects.compare(3, 5, Comparator.reverseOrder());
System.out.println(diff);  //prints: 1

```

如前所述，方法`compare(T o1, T o2)`返回`String`对象中对象`o1`和`o2`在排序列表中的位置之间的差异，而对于`Integer`对象，则返回`-1`、`0`或`1`。API 描述它返回`0`当对象相等时，返回负数当第一个对象小于第二个对象时；否则，它返回正数。

为了演示方法`compare(T a, T b, Comparator<T> c)`的工作原理，假设我们要按照`Person`类对象的名称和年龄以分别按照`String`和`Integer`类的自然排序方式进行排序：

```java
@Override
public int compareTo(Person p){
    int result = Objects.compare(this.name, p.getName(),
                                         Comparator.naturalOrder());
    if (result != 0) {
        return result;
    }
    return Objects.compare(this.age, p.getAge(), 
                                         Comparator.naturalOrder());
}
```

下面是`Person`类中`compareTo(Object)`方法的新实现的结果：

```java
Person p1 = new Person(15, "Zoe");
Person p2 = new Person(45, "Adam");
Person p3 = new Person(37, "Bob");
Person p4 = new Person(30, "Bob");
List<Person> list = new ArrayList<>(List.of(p1, p2, p3, p4));
System.out.println(list);//[{15, Zoe}, {45, Adam}, {37, Bob}, {30, Bob}]
Collections.sort(list);
System.out.println(list);//[{45, Adam}, {30, Bob}, {37, Bob}, {15, Zoe}] 
```

正如您所看到的，`Person`对象首先按照它们的名称的自然顺序排序，然后按照它们的年龄的自然顺序排序。如果我们需要反转名称的顺序，例如，我们将`compareTo(Object)`方法更改为以下内容：

```java
@Override
public int compareTo(Person p){
    int result = Objects.compare(this.name, p.getName(),
                                         Comparator.reverseOrder());
    if (result != 0) {
        return result;
    }
    return Objects.compare(this.age, p.getAge(), 
                                         Comparator.naturalOrder());
}
```

结果的样子就像我们期望的一样：

```java
Person p1 = new Person(15, "Zoe");
Person p2 = new Person(45, "Adam");
Person p3 = new Person(37, "Bob");
Person p4 = new Person(30, "Bob");
List<Person> list = new ArrayList<>(List.of(p1, p2, p3, p4));
System.out.println(list);//[{15, Zoe}, {45, Adam}, {37, Bob}, {30, Bob}]
Collections.sort(list);
System.out.println(list);//[{15, Zoe}, {30, Bob}, {37, Bob}, {45, Adam}] 
```

方法`compare(T a, T b, Comparator<T> c)`的弱点在于它不能处理`null`值。将`new Person(25, null)`对象添加到列表中，在排序时会触发`NullPointerException`异常。在这种情况下，最好使用`org.apache.commons.lang3.ObjectUtils.compare(T o1, T o2)`方法，我们将在下一节中演示。

# toString()

有时需要将一个`Object`对象（它是对某个类类型的引用）转换为它的`String`表示。当引用`obj`被赋予`null`值（对象还未创建）时，编写`obj.toString()`会生成`NullPointerException`异常。对于这种情况，使用`Objects`类的以下方法是更好的选择：

+   `String toString(Object o)`: 当第一个参数值不为`null`时，返回调用第一个参数的`toString()`的结果，否则返回`null`

+   `String toString(Object o, String nullDefault)`: 当第一个参数值不为`null`时，返回调用第一个参数的`toString()`的结果，否则返回第二个参数值`nullDefault`

这是演示如何使用这些方法的代码：

```java
List<String> list = new ArrayList<>(List.of("s0 "));
list.add(null);
for(String e: list){
    System.out.print(e);                   //prints: s0 null
}
System.out.println();
for(String e: list){
    System.out.print(Objects.toString(e)); //prints: s0 null
}
System.out.println();
for(String e: list){
    System.out.print(Objects.toString(e, "element was null")); 
                                        //prints: s0 element was null
}
```

顺便提一下，与当前讨论无关的是，请注意我们如何使用`print()`方法而不是`println()`方法来显示所有结果在一行中，因为`print()`方法不会添加行结束符。

# `ObjectUtils`类

Apache Commons 库的`org.apache.commons.lang3.ObjectUtils`类补充了先前描述的`java.util.Objects`类的方法。本书的范围和分配的大小不允许详细审查`ObjectUtils`类的所有方法，因此我们将根据相关功能进行简要描述，并仅演示那些与我们已经提供的示例相关的方法。

`ObjectUtils`类的所有方法可以分为七个组：

+   对象克隆方法：

    +   `T clone(T obj)`: 如果提供的对象实现了`Cloneable`接口，则返回提供的对象的副本；否则返回`null`。

    +   `T cloneIfPossible(T obj)`: 如果提供的对象实现了`Cloneable`接口，则返回提供的对象的副本；否则返回原始提供的对象。

+   支持对象比较的方法：

    +   `int compare(T c1, T c2)`: 比较实现`Comparable`接口的两个对象的新排序位置；允许任意参数或两个参数都为`null`；将一个`null`值放在非空值的前面。

    +   `int compare(T c1, T c2, boolean nullGreater)`: 如果参数`nullGreater`的值为`false`，则行为与前一个方法完全相同；否则，将一个`null`值放在非空值的后面。我们可以通过在我们的`Person`类中使用最后两个方法来演示这两个方法：

```java
@Override
public int compareTo(Person p){
    int result = ObjectUtils.compare(this.name, p.getName());
    if (result != 0) {
        return result;
    }
    return ObjectUtils.compare(this.age, p.getAge());
}
```

这种改变的结果使我们可以为`name`字段使用`null`值。

```java
Person p1 = new Person(15, "Zoe");
Person p2 = new Person(45, "Adam");
Person p3 = new Person(37, "Bob");
Person p4 = new Person(30, "Bob");
Person p5 = new Person(25, null);
List<Person> list = new ArrayList<>(List.of(p1, p2, p3, p4, p5));
System.out.println(list);  //[{15, Zoe}, {45, Adam}, {37, Bob}, {30, Bob}, {25, }]
Collections.sort(list);
System.out.println(list);  //[{25, }, {45, Adam}, {30, Bob}, {37, Bob}, {15, Zoe}]

```

由于我们使用了`Objects.compare(T c1, T c2)`方法，`null`值被放在非空值的前面。顺便问一下，你是否注意到我们不再显示`null`了？那是因为我们已经按照以下方式更改了类`Person`的方法`toString()`：

```java
@Override
public String toString() {
    //return "{" + age + ", " + name + "}";
    return "{" + age + ", " + Objects.toString(name, "") + "}";
}
```

不仅仅显示字段`name`的值，我们还使用了方法`Objects.toString(Object o, String nullDefault)`，当对象为`null`时，用提供的`nullDefault`值替换对象。在这种情况下，是否使用此方法是一种风格问题。许多程序员可能会认为我们必须显示实际值，而不是将其替换为其他内容。但是，我们这样做只是为了展示方法`Objects.toString(Object o, String nullDefault)`的用法。

如果我们现在使用第二个`compare(T c1, T c2, boolean nullGreater)`方法，那么类`Person`的`compareTo()`方法将如下所示：

```java
@Override
public int compareTo(Person p){
    int result = ObjectUtils.compare(this.name, p.getName(), true);
    if (result != 0) {
        return result;
    }
    return ObjectUtils.compare(this.age, p.getAge());
}
```

接着，具有其`name`设置为`null`的`Person`对象将显示在排序列表的末尾：

```java
Person p1 = new Person(15, "Zoe");
Person p2 = new Person(45, "Adam");
Person p3 = new Person(37, "Bob");
Person p4 = new Person(30, "Bob");
Person p5 = new Person(25, null);
List<Person> list = new ArrayList<>(List.of(p1, p2, p3, p4, p5));
System.out.println(list);  
               //[{15, Zoe}, {45, Adam}, {37, Bob}, {30, Bob}, {25, }]
Collections.sort(list);
System.out.println(list);  
               //[{45, Adam}, {30, Bob}, {37, Bob}, {15, Zoe}, {25, }]

```

为了完成关于`null`值的讨论，当将`null`对象添加到列表中时，上述代码将抛出`NullPointerException`异常：`list.add(null)`。为了避免异常，可以使用一个特殊的`Comparator`对象来处理列表的`null`元素：

```java
Person p1 = new Person(15, "Zoe");
Person p2 = new Person(45, "Adam");
Person p3 = new Person(37, "Bob");
Person p4 = new Person(30, "Bob");
Person p5 = new Person(25, null);
List<Person> list = new ArrayList<>(List.of(p1, p2, p3, p4, p5));
list.add(null);
System.out.println(list);  
        //[{15, Zoe}, {45, Adam}, {37, Bob}, {30, Bob}, {25, }, null]
Collections.sort(list, 
 Comparator.nullsLast(Comparator.naturalOrder()));
System.out.println(list);  
        //[{45, Adam}, {30, Bob}, {37, Bob}, {15, Zoe}, {25, }, null]
```

在这段代码中，你可以看到我们已经表明希望在列表的末尾看到`null`对象。相反，我们可以使用另一个将空对象放在排序列表开头的`Comparator`：

```java
Collections.sort(list, 
                   Comparator.nullsFirst(Comparator.naturalOrder()));
System.out.println(list);  
        //[null, {45, Adam}, {30, Bob}, {37, Bob}, {15, Zoe}, {25, }]

```

+   `notEqual`：

    +   `boolean notEqual(Object object1, Object object2)`: 比较两个对象是否不相等，其中一个或两个对象都可以是`null`。

+   `identityToString`：

    +   `String identityToString(Object object)`: 返回提供的对象的`String`表示，就好像是由基类`Object`的默认方法`toString()`生成的一样。

    +   `void identityToString(StringBuffer buffer, Object object)`: 将所提供对象的`String`表示追加到提供的`StringBuffer`对象上，就好像由基类`Object`的默认方法`toString()`生成一样。

    +   `void identityToString(StringBuilder builder, Object object)`: 将所提供对象的`String`表示追加到提供的`StringBuilder`对象上，就好像由基类`Object`的默认方法`toString()`生成一样。

    +   `void identityToString(Appendable appendable, Object object)`: 将所提供对象的`String`表示追加到提供的`Appendable`对象上，就好像由基类`Object`的默认方法`toString()`生成一样。

以下代码演示了其中的两种方法：

```java
String s = "s0 " + ObjectUtils.identityToString("s1");
System.out.println(s);  //prints: s0 java.lang.String@5474c6c

StringBuffer sb = new StringBuffer();
sb.append("s0");
ObjectUtils.identityToString(sb, "s1");
System.out.println(s);  //prints: s0 java.lang.String@5474c6c

```

+   `allNotNull`和`anyNotNull`：

    +   `boolean allNotNull(Object... values)`: 当所提供数组中所有值都不为`null`时返回`true`。

    +   `boolean anyNotNull(Object... values)`: 当所提供数组中至少有一个值不为`null`时返回`true`。

+   `firstNonNull`和`defaultIfNull`：

    +   `T firstNonNull(T... values)`: 返回所提供数组中第一个不为`null`的值。

    +   `T defaultIfNull(T object, T defaultValue)`: 如果第一个参数为`null`，则返回提供的默认值。

+   `max`、`min`、`median`和`mode`：

    +   `T max(T... values)`: 返回所提供值列表中实现了`Comparable`接口的最后一个值；仅当所有值都为`null`时返回`null`。

    +   `T min(T... values)`: 返回所提供值列表中实现了`Comparable`接口的第一个值；仅当所有值都为`null`时返回`null`。

    +   `T median(T... items)`: 返回所提供值列表中实现了`Comparable`接口的有序列表中位于中间的值；如果值的计数是偶数，则返回中间两个中较小的一个。

    +   `T median(Comparator<T> comparator, T... items)`: 返回根据提供的`Comparator`对象对提供的值列表排序后位于中间的值；如果值的计数是偶数，则返回中间两个中较小的一个。

    +   `T mode(T... items)`: 返回提供的项目中出现频率最高的项目；当没有出现最频繁的项目或没有一个项目最频繁地出现时返回`null`；下面是演示此最后一个方法的代码：

```java
String s = ObjectUtils.mode("s0", "s1", "s1");
System.out.println(s);     //prints: s1

s = ObjectUtils.mode("s0", "s1", "s2");
System.out.println(s);     //prints: null

s = ObjectUtils.mode("s0", "s1", "s2", "s1", "s2");
System.out.println(s);     //prints: null

s = ObjectUtils.mode(null);
System.out.println(s);     //prints: null

s = ObjectUtils.mode("s0", null, null);
System.out.println(s);     //prints: null

```

# 管理字符串

类`String`经常被使用。因此，您必须对其功能有很好的掌握。我们已经在第五章中讨论了`String`值的不可变性，*Java 语言元素和类型*。我们已经表明，每次“修改”`String`值时，都会创建一个新副本，这意味着在多次“修改”的情况下，会创建许多`String`对象，消耗内存并给 JVM 带来负担。

在这种情况下，建议使用类`java.lang.StringBuilder`或`java.lang.StringBuffer`，因为它们是可修改的对象，不需要创建`String`值的副本。我们将展示如何使用它们，并在本节的第一部分解释这两个类之间的区别。

之后，我们会回顾类`String`的方法，然后提供一个对`org.apache.commons.lang3.StringUtils`类的概述，该类补充了类`String`的功能。

# `StringBuilder`和`StringBuffer`

类`StringBuilder`和`StringBuffer`具有完全相同的方法列表。不同之处在于类`StringBuilder`的方法执行速度比类`StringBuffer`的相同方法更快。这是因为类`StringBuffer`不允许不同应用程序线程同时访问其值，所以如果你不是为多线程处理编码，就使用`StringBuilder`。

类`StringBuilder`和`StringBuffer`中有许多方法。但是，我们将展示如何只使用方法`append()`，这显然是最受欢迎的方法，用于需要多次修改`String`值的情况。它的主要功能是将一个值追加到已存储在`StringBuilder`（或`StringBuffer`）对象中的值的末尾。

方法`append()`被重载为所有原始类型和类`String`、`Object`、`CharSequence`和`StringBuffer`，这意味着传入任何这些类的对象的`String`表示都可以追加到现有值中。为了演示，我们将只使用`append(String s)`版本，因为这可能是你大部分时间都会使用的。这里是一个例子：

```java
List<String> list = 
  List.of("That", "is", "the", "way", "to", "build", "a", "sentence");
StringBuilder sb = new StringBuilder();
for(String s: list){
    sb.append(s).append(" ");
}
String s = sb.toString();
System.out.println(s);  //prints: That is the way to build a sentence

```

类`StringBuilder`（和`StringBuffer`）中还有`replace()`、`substring()`和`insert()`方法，允许进一步修改值。虽然它们不像方法`append()`那样经常使用，但我们不打算讨论它们，因为它们超出了本书的范围。

# 类 java.lang.String

类`String`有 15 个构造函数和近 80 个方法。在这本书中详细讨论和演示每一个方法对来说有点过分，所以我们只会评论最受欢迎的方法，其他的只会提到。当你掌握了基础知识后，你可以阅读在线文档，看看类`String`的其他方法还可以做什么。

# 构造函数

如果您担心应用程序创建的字符串消耗过多的内存，则`String`类的构造函数很有用。问题在于，`String`字面值（例如`abc`）存储在内存的特殊区域中，称为“字符串常量池”，并且永远不会被垃圾回收。这样设计的理念是，`String`字面值消耗的内存远远超过数字。此外，处理这样的大型实体会产生开销，可能会使 JVM 负担过重。这就是设计者认为将它们存储并在所有应用程序线程之间共享比分配新内存然后多次清理相同值更便宜的原因。

但是，如果`String`值的重用率较低，而存储的`String`值消耗过多内存，则使用构造函数创建`String`对象可能是解决问题的方法。这里是一个例子：

```java
String veryLongText = new String("asdakjfakjn akdb aakjn... akdjcnak");
```

以这种方式创建的`String`对象位于堆区（存储所有对象的地方），并且在不再使用时进行垃圾回收。这就是`String`构造函数发挥作用的时候。

如有必要，您可以使用`String`类的`intern()`方法，在字符串常量池中创建堆`String`对象的副本。它不仅允许我们与其他应用程序线程共享值（在多线程处理中），还允许我们通过引用（使用运算符`==`）将其与另一个字面值进行比较。如果引用相等，则意味着它们指向池中的相同`String`值。

但是，主流程序员很少以这种方式管理内存，因此我们将不再进一步讨论这个话题。

# `format()`

方法`String format(String format, Object... args)`允许将提供的对象插入字符串的指定位置，并根据需要进行格式化。在`java.util.Formatter`类中有许多格式说明符。我们这里只演示`%s`，它通过调用对象的`toString()`方法将传入的对象转换为其`String`表示形式：

```java
String format = "There is a %s in the %s";
String s = String.format(format, "bear", "woods");
System.out.println(s); //prints: There is a bear in the woods

format = "Class %s is very useful";
s = String.format(format, new A());
System.out.println(s);  //prints: Class A is very useful

```

# `replace()`

`String` 类中的方法`String replace(CharSequence target, CharSequence replacement)`，该方法会用第二个参数的值替换第一个参数的值：

```java
String s1 = "There is a bear in the woods";
String s2 = s1.replace("bear", "horse").replace("woods", "field");
System.out.println(s2);     //prints: There is a horse in the field

```

还有一些方法，比如`String replaceAll(String regex, String replacement)`和`String replaceFirst(String regex, String replacement)`，它们具有类似的功能。

# `compareTo()`

我们已经在示例中使用了`int compareTo(String anotherString)`方法。它返回此`String`值和`anotherString`值在有序列表中的位置差异。它用于字符串的自然排序，因为它是`Comparable`接口的实现。

方法`int compareToIgnoreCase(String str)`执行相同的功能，但会忽略比较字符串的大小写，并且不用于自然排序，因为它不是`Comparable`接口的实现。

# `valueOf(Object j)`

静态方法 `String valueOf(Object obj)` 如果提供的对象为 `null`，则返回 `null`，否则调用提供对象的 `toString()` 方法。

# valueOf(基本类型或字符数组)

任何基本类型的值都可以作为参数传递给静态方法 `String valueOf(primitive value)`，该方法返回所提供值的字符串表示形式。例如，`String.valueOf(42)` 返回`42`。该组方法包括以下静态方法：

+   `String valueOf(boolean b)`

+   `String valueOf(char c)`

+   `String valueOf(double d)`

+   `String valueOf(float f)`

+   `String valueOf(int i)`

+   `String valueOf(long l)`

+   `String valueOf(char[] data)`

+   `String valueOf(char[] data, int offset, int count)`

# copyValueOf(char[])

方法 `String copyValueOf(char[] data)` 等效于 `valueOf(char[])`，而方法 `String copyValueOf(char[] data, int offset, int count)` 等效于 `valueOf(char[], int, int)`。它们返回字符数组或其子数组的 `String` 表示形式。

而方法 `void getChars(int srcBegin, int srcEnd, char[] dest, int dstBegin)` 将此 `String` 值中的字符复制到目标字符数组中。

# indexOf() 和 substring()

各种 `int indexOf(String str)` 和 `int lastIndexOf(String str)` 方法返回字符串中子字符串的位置：

```java
String s = "Introduction";
System.out.println(s.indexOf("I"));      //prints: 0
System.out.println(s.lastIndexOf("I"));  //prints: 0
System.out.println(s.lastIndexOf("i"));  //prints: 9
System.out.println(s.indexOf("o"));      //prints: 4
System.out.println(s.lastIndexOf("o"));  //prints: 10
System.out.println(s.indexOf("tro"));    //prints: 2
```

注意位置计数从零开始。

方法 `String substring(int beginIndex)` 返回从作为参数传递的位置（索引）开始的字符串的剩余部分：

```java
String s = "Introduction";
System.out.println(s.substring(1));        //prints: ntroduction
System.out.println(s.substring(2));        //prints: troduction

```

位置为 `beginIndex` 的字符是前一个子字符串中存在的第一个字符。

方法 `String substring(int beginIndex, int endIndex)` 返回从作为第一个参数传递的位置开始到作为第二个参数传递的位置的子字符串：

```java
String s = "Introduction";
System.out.println(s.substring(1, 2));        //prints: n
System.out.println(s.substring(1, 3));        //prints: nt

```

与方法 `substring(beginIndex)` 一样，位置为 `beginIndex` 的字符是前一个子字符串中存在的第一个字符，而位置为 `endIndex` 的字符不包括在内。 `endIndex - beginIndex` 的差等于子字符串的长度。

这意味着以下两个子字符串相等：

```java
System.out.println(s.substring(1));              //prints: ntroduction
System.out.println(s.substring(1, s.length()));  //prints: ntroduction

```

# contains() 和 matches()

方法 `boolean contains(CharSequence s)` 在提供的字符序列（子字符串）存在时返回 `true`：

```java
String s = "Introduction";
System.out.println(s.contains("x"));          //prints: false
System.out.println(s.contains("o"));          //prints: true
System.out.println(s.contains("tro"));        //prints: true
System.out.println(s.contains("trx"));        //prints: false

```

其他类似的方法有：

+   `boolean matches(String regex)`: 使用正则表达式（本书不讨论此内容）

+   `boolean regionMatches(int tOffset, String other, int oOffset, int length)`: 比较两个字符串的区域

+   `boolean regionMatches(boolean ignoreCase, int tOffset, String other, int oOffset, int length)`: 与上述相同，但使用标志 `ignoreCase` 指示是否忽略大小写

# split(), concat() 和 join()

方法`String[] split(String regex)`和`String[] split(String regex, int limit)`使用传入的正则表达式将字符串拆分成子字符串。我们在本书中不解释正则表达式。但是，有一个非常简单的正则表达式，即使您对正则表达式一无所知也很容易使用：如果您只是将字符串中存在的任何符号或子字符串传递到此方法中，该字符串将被拆分为以传入的值分隔的部分，例如：

```java
String[] substrings = "Introduction".split("o");
System.out.println(Arrays.toString(substrings)); 
                                       //prints: [Intr, ducti, n]
substrings = "Introduction".split("duct");
System.out.println(Arrays.toString(substrings)); 
                                      //prints: [Intro, ion] 

```

此代码仅说明了功能。但是以下代码片段更实用：

```java
String s = "There is a bear in the woods";
String[] arr = s.split(" ");
System.out.println(Arrays.toString(arr));  
                       //prints: [There, is, a, bear, in, the, woods]
arr = s.split(" ", 3);
System.out.println(Arrays.toString(arr));  
                          //prints: [There, is, a bear in the woods]

```

正如您所见，`split()`方法中的第二个参数限制了生成的子字符串的数量。

方法`String concat(String str)`将传入的值添加到字符串的末尾：

```java
String s1 =  "There is a bear";
String s2 =  " in the woods";
String s = s1.concat(s2);
System.out.println(s);  //prints: There is a bear in the woods

```

`concat()`方法创建一个新的`String`值，其中包含连接的结果，因此非常经济。但是，如果您需要添加（连接）许多值，则使用`StringBuilder`（或`StringBuffer`，如果需要保护免受并发访问）将是更好的选择。我们在前一节中讨论过这个问题。另一个选择是使用运算符`+`：

```java
String s =  s1 + s2;
System.out.println(s);  //prints: There is a bear in the woods
```

当与`String`值一起使用时，运算符`+`是基于`StringBuilder`实现的，因此允许通过修改现有的值来添加`String`值。使用 StringBuilder 和仅使用运算符`+`添加`String`值之间没有性能差异。

方法`String join(CharSequence delimiter, CharSequence... elements)`和`String join(CharSequence delimiter, Iterable<? extends CharSequence> elements)`也基于`StringBuilder`。它们使用传入的`delimiter`将提供的值组装成一个`String`值，以在创建的`String`结果中分隔组装的值。以下是一个示例：

```java
s = String.join(" ", "There", "is", "a", "bear", "in", "the", "woods");
System.out.println(s);  //prints: There is a bear in the woods

List<String> list = 
             List.of("There", "is", "a", "bear", "in", "the", "woods");
s = String.join(" ", list);
System.out.println(s);  //prints: There is a bear in the woods
```

# startsWith() 和 endsWith()

以下方法在字符串值以提供的子字符串`prefix`开始（或结束）时返回`true`：

+   `boolean startsWith(String prefix)`

+   `boolean startsWith(String prefix, int toffset)`

+   `boolean endsWith(String suffix)`

这是演示代码：

```java
boolean b = "Introduction".startsWith("Intro");
System.out.println(b);             //prints: true

b = "Introduction".startsWith("tro", 2);
System.out.println(b);             //prints: true

b = "Introduction".endsWith("ion");
System.out.println(b);             //prints: true

```

# equals() 和 equalsIgnoreCase()

我们已经多次使用了`String`类的`boolean equals(Object anObject)`方法，并指出它将此`String`值与其他对象进行比较。此方法仅在传入的对象是具有相同值的`String`时返回`true`。

方法`boolean equalsIgnoreCase(String anotherString)`也执行相同的操作，但还忽略大小写，因此字符串`AbC`和`ABC`被视为相等。

# contentEquals() 和 copyValueOf()

方法`boolean contentEquals(CharSequence cs)`将此`String`值与实现接口`CharSequence`的对象的`String`表示进行比较。流行的`CharSequence`实现包括`CharBuffer`、`Segment`、`String`、`StringBuffer`和`StringBuilder`。

方法 `boolean contentEquals(StringBuffer sb)` 仅对 `StringBuffer` 有效。它的实现略有不同于 `contentEquals(CharSequence cs)`，在某些情况下可能具有一些性能优势，但我们不打算讨论这些细节。此外，当你在 `String` 值上调用 `contentEquals()` 时，你可能甚至不会注意到使用了哪种方法，除非你努力利用差异。

# length()、isEmpty() 和 hashCode()

方法 `int length()` 返回 `String` 值中字符的数量。

方法 `boolean isEmpty()` 在 `String` 值中没有字符且方法 `length()` 返回零时返回 `true`。

方法 `int hashCode()` 返回 `String` 对象的哈希值。

# trim()、toLowerCase() 和 toUpperCase()

方法 `String trim()` 从 `String` 值中删除前导和尾随空格。

以下方法更改 `String` 值中字符的大小写：

+   `String toLowerCase()`

+   `String toUpperCase()`

+   `String toLowerCase(Locale locale)`

+   `String toUpperCase(Locale locale)`

# getBytes()、getChars() 和 toCharArray()

以下方法将 `String` 值转换为字节数组，可选择使用给定的字符集进行编码：

+   `byte[] getBytes()`

+   `byte[] getBytes(Charset charset)`

+   `byte[] getBytes(String charsetName)`

这些方法将所有或部分 `String` 值转换为其他类型：

+   `IntStream chars()`

+   `char[] toCharArray()`

+   `char charAt(int index)`

+   `CharSequence subSequence(int beginIndex, int endIndex)`

# 按索引或流获取代码点

以下一组方法将 `String` 值的全部或部分转换为其字符的 Unicode 代码点：

+   `IntStream codePoints()`

+   `int codePointAt(int index)`

+   `int codePointBefore(int index)`

+   `int codePointCount(int beginIndex, int endIndex)`

+   `int offsetByCodePoints(int index, int codePointOffset)`

我们在第五章 *Java 语言元素和类型*中解释了 Unicode 代码点。当你需要表示*不能适应* `char` 类型的两个字节时，这些方法特别有用。这样的字符具有大于 `Character.MAX_VALUE` 的代码点，即  `65535`。

# 类 lang3.StringUtils

Apache Commons 库的 `org.apache.commons.lang3.StringUtils` 类具有 120 多个静态实用方法，这些方法补充了我们在前一节中描述的 `String` 类的方法。

最受欢迎的是以下静态方法：

+   `boolean isBlank(CharSequence cs)`: 当传入的参数为空字符串""、`null` 或空格时返回 `true`

+   `boolean isNotBlank(CharSequence cs)`: 当传入的参数不为空字符串""、`null` 或空格时返回 `true`

+   `boolean isAlpha(CharSequence cs)`: 当传入的参数只包含 Unicode 字母时返回 `true`

+   `boolean isAlphaSpace(CharSequence cs)`: 当传入的参数仅包含 Unicode 字母和空格（' '）时返回`true`

+   `boolean isNumeric(CharSequence cs)`: 当传入的参数仅包含数字时返回`true`

+   `boolean isNumericSpace(CharSequence cs)`: 当传入的参数仅包含数字和空格（' '）时返回`true`

+   `boolean isAlphaNumeric(CharSequence cs)`: 当传入的参数仅包含 Unicode 字母和数字时返回`true`

+   `boolean isAlphaNumericSpace(CharSequence cs)`: 当传入的参数仅包含 Unicode 字母、数字和空格（' '）时返回`true`

我们强烈建议您查看该类的 API 并了解您可以在其中找到什么。

# 管理时间

`java.time`包及其子包中有许多类。它们被引入作为处理日期和时间的其他旧包的替代品。新类是线程安全的（因此更适合多线程处理），而且同样重要的是，设计更一致，更容易理解。此外，新实现遵循国际标准组织（ISO）的日期和时间格式，但也允许使用任何其他自定义格式。

我们将描述主要的五个类，并演示如何使用它们：

+   `java.util.LocalDate`

+   `java.util.LocalTime`

+   `java.util.LocalDateTime`

+   `java.util.Period`

+   `java.util.Duration`

所有这些以及`java.time`包及其子包中的其他类都具有丰富的各种功能，涵盖了所有实际情况和任何想象得到的情况。但我们不打算覆盖所有内容，只是介绍基础知识和最常见的用例。

# `java.time.LocalDate`

`LocalDate`类不包含时间。它表示 ISO 8601 格式的日期，即 yyyy-MM-DD：

```java
System.out.println(LocalDate.now());   //prints: 2018-04-14
```

正如您所见，方法`now()`返回当前日期，即它设置在您计算机上的日期：`April 14, 2018`是撰写本节时的日期。

类似地，您可以使用静态方法`now(ZoneId zone)`获取任何其他时区的当前日期。`ZoneId`对象可以使用静态方法`ZoneId.of(String zoneId)`构造，其中`String zoneId`是方法`ZonId.getAvailableZoneIds()`返回的任何`String`值之一：

```java
Set<String> zoneIds = ZoneId.getAvailableZoneIds();
for(String zoneId: zoneIds){
    System.out.println(zoneId);     
}
```

该代码打印了许多时区 ID，其中之一是`Asia/Tokyo`。现在，我们可以找出当前日期在该时区的日期：

```java
ZoneId zoneId = ZoneId.of("Asia/Tokyo");
System.out.println(LocalDate.now(zoneId));   //prints: 2018-04-15

```

`LocalDate`的对象也可以表示过去或未来的任何日期，使用以下方法：

+   `LocalDate parse(CharSequence text)`: 从 ISO 8601 格式的字符串 yyyy-MM-DD 构造对象

+   `LocalDate parse(CharSequence text, DateTimeFormatter formatter) `: 根据对象`DateTimeFormatter`指定的格式从字符串构造对象，该对象有许多预定义格式

+   `LocalDate of(int year, int month, int dayOfMonth)`: 从年、月和日构造对象

+   `LocalDate of(int year, Month month, int dayOfMonth)`：根据年份、月份（作为`enum`常量）和日期构造对象

+   `LocalDate ofYearDay(int year, int dayOfYear)`：根据年份和年份中的日数构造对象

以下代码演示了这些方法：

```java
LocalDate lc1 =  LocalDate.parse("2020-02-23");
System.out.println(lc1);                     //prints: 2020-02-23

DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate lc2 =  LocalDate.parse("23/02/2020", formatter);
System.out.println(lc2);                     //prints: 2020-02-23

LocalDate lc3 =  LocalDate.of(2020, 2, 23);
System.out.println(lc3);                     //prints: 2020-02-23

LocalDate lc4 =  LocalDate.of(2020, Month.FEBRUARY, 23);
System.out.println(lc4);                     //prints: 2020-02-23

LocalDate lc5 = LocalDate.ofYearDay(2020, 54);
System.out.println(lc5);                     //prints: 2020-02-23

```

使用`LocalDate`对象，可以获取各种值：

```java
System.out.println(lc5.getYear());          //prints: 2020
System.out.println(lc5.getMonth());         //prints: FEBRUARY
System.out.println(lc5.getMonthValue());    //prints: 2
System.out.println(lc5.getDayOfMonth());    //prints: 23

System.out.println(lc5.getDayOfWeek());     //prints: SUNDAY
System.out.println(lc5.isLeapYear());       //prints: true
System.out.println(lc5.lengthOfMonth());    //prints: 29
System.out.println(lc5.lengthOfYear());     //prints: 366

```

`LocalDate`对象可以被修改：

```java
System.out.println(lc5.withYear(2021));     //prints: 2021-02-23
System.out.println(lc5.withMonth(5));       //prints: 2020-05-23
System.out.println(lc5.withDayOfMonth(5));  //prints: 2020-02-05
System.out.println(lc5.withDayOfYear(53));  //prints: 2020-02-22

System.out.println(lc5.plusDays(10));       //prints: 2020-03-04
System.out.println(lc5.plusMonths(2));      //prints: 2020-04-23
System.out.println(lc5.plusYears(2));       //prints: 2022-02-23

System.out.println(lc5.minusDays(10));      //prints: 2020-02-13
System.out.println(lc5.minusMonths(2));     //prints: 2019-12-23
System.out.println(lc5.minusYears(2));      //prints: 2018-02-23 
```

`LocalDate`对象也可以进行比较：

```java
LocalDate lc6 =  LocalDate.parse("2020-02-22");
LocalDate lc7 =  LocalDate.parse("2020-02-23");
System.out.println(lc6.isAfter(lc7));       //prints: false
System.out.println(lc6.isBefore(lc7));      //prints: true

```

`LocalDate`类中还有许多其他有用的方法。如果您需要处理日期，我们建议您阅读该类及其他`java.time`包及其子包的 API。

# java.time.LocalTime

`LocalTime`类包含没有日期的时间。它具有类似于`LocalDate`类的方法。

以下是如何创建`LocalTime`类的对象的方法：

```java
System.out.println(LocalTime.now());         //prints: 21:15:46.360904

ZoneId zoneId = ZoneId.of("Asia/Tokyo");
System.out.println(LocalTime.now(zoneId));   //prints: 12:15:46.364378

LocalTime lt1 =  LocalTime.parse("20:23:12");
System.out.println(lt1);                     //prints: 20:23:12

LocalTime lt2 =  LocalTime.of(20, 23, 12);
System.out.println(lt2);                     //prints: 20:23:12

```

`LocalTime`对象的每个时间值组件可以按以下方式提取：

```java
System.out.println(lt2.getHour());          //prints: 20
System.out.println(lt2.getMinute());        //prints: 23
System.out.println(lt2.getSecond());        //prints: 12
System.out.println(lt2.getNano());          //prints: 0

```

该对象可以被修改：

```java
System.out.println(lt2.withHour(3));        //prints: 03:23:12
System.out.println(lt2.withMinute(10));     //prints: 20:10:12
System.out.println(lt2.withSecond(15));     //prints: 20:23:15
System.out.println(lt2.withNano(300));      //prints: 20:23:12:000000300

System.out.println(lt2.plusHours(10));      //prints: 06:23:12
System.out.println(lt2.plusMinutes(2));     //prints: 20:25:12
System.out.println(lt2.plusSeconds(2));     //prints: 20:23:14
System.out.println(lt2.plusNanos(200));     //prints: 20:23:14:000000200

System.out.println(lt2.minusHours(10));      //prints: 10:23:12
System.out.println(lt2.minusMinutes(2));     //prints: 20:21:12
System.out.println(lt2.minusSeconds(2));     //prints: 20:23:10
System.out.println(lt2.minusNanos(200));     //prints: 20:23:11.999999800

```

`LocalTime`类的两个对象也可以进行比较：

```java
LocalTime lt3 =  LocalTime.parse("20:23:12");
LocalTime lt4 =  LocalTime.parse("20:25:12");
System.out.println(lt3.isAfter(lt4));       //prints: false
System.out.println(lt3.isBefore(lt4));      //prints: true

```

`LocalTime`类中还有许多其他有用的方法。如果您需要处理时间，请阅读该类及其他`java.time`包及其子包的 API。

# java.time.LocalDateTime

`LocalDateTime`类同时包含日期和时间，并且具有`LocalDate`类和`LocalTime`类拥有的所有方法，因此我们不会在此重复。我们只会展示如何创建`LocalDateTime`对象：

```java
System.out.println(LocalDateTime.now());  //2018-04-14T21:59:00.142804
ZoneId zoneId = ZoneId.of("Asia/Tokyo");
System.out.println(LocalDateTime.now(zoneId));  
                                   //prints: 2018-04-15T12:59:00.146038
LocalDateTime ldt1 =  LocalDateTime.parse("2020-02-23T20:23:12");
System.out.println(ldt1);                 //prints: 2020-02-23T20:23:12
DateTimeFormatter formatter = 
          DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm:ss");
LocalDateTime ldt2 =  
       LocalDateTime.parse("23/02/2020 20:23:12", formatter);
System.out.println(ldt2);                 //prints: 2020-02-23T20:23:12
LocalDateTime ldt3 = LocalDateTime.of(2020, 2, 23, 20, 23, 12);
System.out.println(ldt3);                 //prints: 2020-02-23T20:23:12
LocalDateTime ldt4 =  
        LocalDateTime.of(2020, Month.FEBRUARY, 23, 20, 23, 12);
System.out.println(ldt4);                     //prints: 2020-02-23T20:23:12
LocalDate ld = LocalDate.of(2020, 2, 23);
LocalTime lt =  LocalTime.of(20, 23, 12);
LocalDateTime ldt5 = LocalDateTime.of(ld, lt);
System.out.println(ldt5);                     //prints: 2020-02-23T20:23:12

```

`LocalDateTime`类中还有许多其他有用的方法。如果你需要处理日期和时间，我们建议你阅读该类及其他`java.time`包及其子包的 API。

# `Period`和`Duration`

`java.time.Period`和`java.time.Duration`类的设计目的是包含一定的时间量：

+   `Period`对象包含以年、月和日为单位的时间量

+   `Duration`对象包含以小时、分、秒和纳秒为单位的时间量

以下代码演示了它们如何在`LocalDateTime`类中创建和使用，但是对于`Period`来说，相同的方法也存在于`LocalDate`类中，而对于`Duration`来说，相同的方法也存在于`LocalTime`类中：

```java
LocalDateTime ldt1 = LocalDateTime.parse("2020-02-23T20:23:12");
LocalDateTime ldt2 = ldt1.plus(Period.ofYears(2));
System.out.println(ldt2); //prints: 2022-02-23T20:23:12

//The following methods work the same way:
ldt.minus(Period.ofYears(2));
ldt.plus(Period.ofMonths(2));
ldt.minus(Period.ofMonths(2));
ldt.plus(Period.ofWeeks(2));
ldt.minus(Period.ofWeeks(2));
ldt.plus(Period.ofDays(2));
ldt.minus(Period.ofDays(2));

ldt.plus(Duration.ofHours(2));
ldt.minus(Duration.ofHours(2));
ldt.plus(Duration.ofMinutes(2));
ldt.minus(Duration.ofMinutes(2));
ldt.plus(Duration.ofMillis(2));
ldt.minus(Duration.ofMillis(2));
```

在以下代码中还演示了创建和使用`Period`对象的其他方法：

```java
LocalDate ld1 =  LocalDate.parse("2020-02-23");
LocalDate ld2 =  LocalDate.parse("2020-03-25");

Period period = Period.between(ld1, ld2);
System.out.println(period.getDays());       //prints: 2
System.out.println(period.getMonths());     //prints: 1
System.out.println(period.getYears());      //prints: 0
System.out.println(period.toTotalMonths()); //prints: 1

period = Period.between(ld2, ld1);
System.out.println(period.getDays());       //prints: -2
```

`Duration`对象可以类似地创建和使用：

```java
LocalTime lt1 =  LocalTime.parse("10:23:12");
LocalTime lt2 =  LocalTime.parse("20:23:14");
Duration duration = Duration.between(lt1, lt2);
System.out.println(duration.toDays());     //prints: 0
System.out.println(duration.toHours());    //prints: 10
System.out.println(duration.toMinutes());  //prints: 600
System.out.println(duration.toSeconds());  //prints: 36002
System.out.println(duration.getSeconds()); //prints: 36002
System.out.println(duration.toNanos());    //prints: 36002000000000
System.out.println(duration.getNano());    //prints: 0

```

`Period`和`Duration`类中还有许多其他有用的方法。如果您需要处理时间量，请阅读这些类及其他`java.time`包及其子包的 API。

# 管理随机数

生成一个真正的随机数是一个大问题，不属于本书。但是对于绝大多数实际目的来说，Java 提供的伪随机数生成器已经足够好了，这就是我们将在本节中讨论的内容。

在 Java 标准库中生成随机数有两种主要方式：

+   `java.lang.Math.random()`方法

+   `java.util.Random`类

还有 `java.security.SecureRandom` 类，它提供了一个加密强度很高的随机数生成器，但超出了入门课程的范围。

# 方法 `java.lang.Math.random()`

类 `Math` 的静态方法 `double random()` 返回一个大于或等于 `0.0` 且小于 `1.0` 的 `double` 类型值：

```java
for(int i =0; i < 3; i++){
    System.out.println(Math.random());
    //0.9350483840148613
    //0.0477353019234189
    //0.25784245516898985
}
```

在前面的注释中我们已经捕获了结果。但在实践中，更多时候需要的是某个范围内的随机整数。为了满足这样的需求，我们可以编写一个方法，例如，生成一个从 0（包含）到 10（不包含）的随机整数：

```java
int getInteger(int max){
    return (int)(Math.random() * max);
}
```

以下是前述代码的一次运行结果：

```java
for(int i =0; i < 3; i++){
    System.out.print(getInteger(10) + " "); //prints: 2 5 6
}
```

如你所见，它生成一个随机整数值，可以是以下 10 个数字之一：0、1、...、9。以下是使用相同方法的代码，并生成从 0（包含）到 100（不包含）的随机整数：

```java
for(int i =0; i < 3; i++){
    System.out.print(getInteger(100) + " "); //prints: 48 11 97
}
```

当你需要一个介于 100（包含）和 200（不包含）之间的随机数时，你可以直接将前述结果加上 100：

```java
for(int i =0; i < 3; i++){
    System.out.print(100 + getInteger(100) + " "); //prints: 114 101 127
}
```

将范围的两个端点包括在结果中可以通过四舍五入生成的 `double` 值来实现：

```java
int getIntegerRound(int max){
    return (int)Math.round(Math.random() * max);
}
```

当我们使用前述方法时，结果为：

```java
for(int i =0; i < 3; i++){
    System.out.print(100 + getIntegerRound(100) + " "); //179 147 200
}
```

如你所见，范围的上限（数字 200）包含在可能的结果集中。可以通过将所请求的上限范围加 1 来达到同样的效果：

```java
int getInteger2(int max){
    return (int)(Math.random() * (max + 1));
}
```

如果我们使用前述方法，我们可以得到以下结果：

```java
for(int i =0; i < 3; i++){
    System.out.print(100 + getInteger2(100) + " "); //167 200 132
}
```

但是，如果你查看 `Math.random()` 方法的源代码，你会看到它使用了 `java.util.Random` 类及其 `nextDouble()` 方法来生成一个随机的 double 值。因此，让我们看看如何直接使用 `java.util.Random` 类。

# 类 `java.util.Random`

类 `Random` 的方法 `doubles()` 生成一个大于或等于 `0.0` 且小于 `1.0` 的 `double` 类型值：

```java
Random random = new Random();
for(int i =0; i < 3; i++){
    System.out.print(random.nextDouble() + " "); 
    //prints: 0.8774928230544553 0.7822070124559267 0.09401796000707807 
}
```

我们可以像在上一节中使用 `Math.random()` 一样使用方法 `nextDouble()`。但是在需要某个范围内的随机整数值时，类还有其他方法可用，而无需创建自定义的 `getInteger()` 方法。例如，`nextInt()` 方法返回介于 `Integer.MIN_VALUE`（包含）和 `Integer.MAX_VALUE`（包含）之间的整数值：

```java
for(int i =0; i < 3; i++){
    System.out.print(random.nextInt() + " "); 
                        //prints: -2001537190 -1148252160 1999653777
}
```

并且带有参数的相同方法允许我们通过上限（不包含）限制返回值的范围：

```java
for(int i =0; i < 3; i++){
    System.out.print(random.nextInt(11) + " "); //prints: 4 6 2
}
```

该代码生成一个介于 0（包含）和 10（包含）之间的随机整数值。以下代码返回介于 11（包含）和 20（包含）之间的随机整数值：

```java
for(int i =0; i < 3; i++){
    System.out.print(11 + random.nextInt(10) + " "); //prints: 13 20 15
}
```

从范围中生成随机整数的另一种方法是使用方法 `ints(int count, int min, int max)` 返回的 `IntStream` 对象，其中 `count` 是所请求的值的数量，`min` 是最小值（包含），`max` 是最大值（不包含）：

```java
String result = random.ints(3, 0, 101)
        .mapToObj(String::valueOf)
        .collect(Collectors.joining(" ")); //prints: 30 48 52

```

此代码从 0（包含）到 100（包含）返回三个整数值。我们将在第十八章中更多地讨论流，*流和管道*。

# 练习 - Objects.equals() 结果

有三个类：

```java
public class A{}
public class B{}
public class Exercise {
    private A a;
    private B b;
    public Exercise(){
        System.out.println(java.util.Objects.equals(a, b));
    }
    public static void main(String... args){
        new Exercise();
    }
}
```

当我们运行`Exercise`类的`main()`方法时，会显示什么？ `错误`？ `假`？ `真`？

# 答案

显示将只显示一个值：`真`。原因是两个私有字段——`a`和`b`——都初始化为`null`。

# 总结

在本章中，我们向读者介绍了 Java 标准库和 Apache Commons 库中最受欢迎的实用程序和一些其他类。每个 Java 程序员都必须对它们的功能有很好的理解，才能成为有效的编码者。研究它们还有助于了解各种软件设计模式和解决方案，这些模式和解决方案具有指导意义，并且可以用作任何应用程序中最佳编码实践的模式。

在下一章中，我们将向读者演示如何编写能够操作数据库中数据的 Java 代码——插入、读取、更新和删除。它还将提供 SQL 和基本数据库操作的简要介绍。
