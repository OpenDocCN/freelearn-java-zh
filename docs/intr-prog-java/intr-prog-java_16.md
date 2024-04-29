# 第十六章：管理对象、字符串、时间和随机数

我们将在本章讨论的类，与前几章讨论的 Java 集合和数组一起，属于每个程序员都必须掌握的类（主要是来自 Java 标准库和 Apache Commons 的实用程序），以便成为有效的编码人员。它们还说明了各种有益的软件设计和解决方案，可以作为最佳编码实践的模式。

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

您可能不需要管理数组，甚至可能不需要管理集合（至少有一段时间），但您无法避免管理对象，这意味着您可能每天都会使用本节中描述的类。

尽管`java.util.Objects`类是在 2011 年（Java 7 发布时）添加到 Java 标准库中的，而`ObjectUtils`类自 2002 年以来就存在于 Apache Commons 库中，但它们的使用增长缓慢。这可能部分原因是它们最初的方法数量很少-2003 年`ObjectUtils`只有六个方法，2011 年`Objects`只有九个。然而，它们是非常有用的方法，可以使代码更易读和更健壮，减少错误的发生。因此，为什么这些类最初没有被更频繁地使用仍然是一个谜。我们希望您立即在您的第一个项目中开始使用它们。

# Class java.util.Objects

`Objects`类只有 17 个方法-全部是静态的。我们在上一章中已经使用了其中一些方法，当时我们实现了`Person`类：

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

```

@Override

public int hashCode(){

return Objects.hash(age, name);

}

@Override

public String toString() {

return "Person{age=" + age + ", name=" + name + "}";

}

}

```java

我们在以前的`equals()`和`hashCode()`方法中使用了`Objects`类。一切都运行正常。但是，请注意我们如何检查前面构造函数中的参数`name`。如果参数是`null`，我们将为字段`name`分配一个空的`String`值。我们这样做是为了避免第 25 行的`NullPointerException`。另一种方法是使用 Apache Commons 库中的`ObjectUtils`类。我们将在下一节中进行演示。`ObjectUtils`类的方法处理`null`值，并且使将`null`参数转换为空`String`变得不必要。

但首先，让我们回顾一下`Objects`类的方法。

# equals() and deepEquals()

我们已经广泛讨论了`equals()`方法的实现，但始终假设它是在非`null`对象`obj`上调用的，因此调用`obj.equals(anotherObject)`不会生成`NullPointerException`。

然而，有时我们需要比较两个对象`a`和`b`，当它们中的一个或两个都可以是`null`时。以下是这种情况的典型代码：

```java

boolean equals(Object a, Object b) {

return (a == b) || (a != null && a.equals(b));

}

```

This is the actual source code of the `boolean Objects.equals(Object a, Object b)` method. It allows comparing two objects using the method `equals(Object)` and handles cases where one or both of them are `null`.

Another related method of the class `Objects` is `boolean deepEquals(Object a, Object b)`. Here is its source code:

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

As you can see, it is based on `Arrays.deepEquals()`, which we discussed in the previous section. The demonstration code for these methods helps to understand the difference:

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

In the preceding code, `Objects.equals(as1, as2)` and `Objects.equals(aas1, aas2)` return `false` because arrays cannot override the method `equals()` of the class `Object` and are compared by references, not by value.

The method `Arrays.equals(aas1, aas2)` returns `false` for the same reason: because the elements of the nested array are arrays and are compared by references.

To summarize, if you would like to compare two objects, `a` and `b`, by the values of their fields, then:

+   If they are not arrays and `a` is not `null`, use `a.equals(b)`

+   If they are not arrays and both objects can be `null`, use `Objects.equals(a, b)`

+   If both can be arrays and both can be `null`, use `Objects.deepEquals(a, b)`

That said, we can see that the method `Objects.deepEquals()` is the safest one, but it does not mean you must always use it. Most of the time, you will know whether the compared objects can be `null` or can be arrays, so you can safely use other `equals()` methods too.

# hash() and hashCode()

The hash values returned by the methods `hash()` or `hashCode()` are typically used as a key for storing the object in a hash-using collection, such as `HashSet()`. The default implementation in the  `Object` superclass is based on the object reference in memory.  It returns different hash values for two objects of the same class with the same values of the instance fields. That is why, if you need two class instances to have the same hash value for the same state, it is important to override the default `hashCode()` implementation using one of these methods:

+   `int hashCode(Object value)`: calculates a hash value for a single object

+   `int hash(Object... values)`: calculates a hash value for an array of objects (see how we used it in the class `Person` in our previous example)

Please notice that these two methods return different hash values for the same object when it is used as a single-element input array of the method `Objects.hash()`:

```java

System.out.println(Objects.hash("s1"));           //prints: 3645

System.out.println(Objects.hashCode("s1"));       //prints: 3614

```

The only value that yields the same hash from both methods is `null`:

```java

System.out.println(Objects.hash(null));      //prints: 0

System.out.println(Objects.hashCode(null));  //prints: 0

```

When used as a single not-null parameter, the same value has different hash values returned from the methods `Objects.hashCode(Object value)` and `Objects.hash(Object... values)`. The value `null` yields the same hash value, `0`, returned from each of these methods.

使用类`Objects`进行哈希值计算的另一个优点是它容忍`null`值，而在`null`引用上调用实例方法`hashCode()`会生成`NullPointerException`。

# isNull()和 nonNull()

这两种方法只是对布尔表达式`obj == null`和`obj != null`的薄包装：

+   `boolean isNull(Object obj)`: 返回与`obj == null`相同的值

+   `boolean nonNull(Object obj)`: 返回与`obj != null`相同的值

这是演示代码：

```java

String object = null;

System.out.println(object == null);           //打印：true

System.out.println(Objects.isNull(object));   //打印：true

System.out.println(object != null);           //打印：false

System.out.println(Objects.nonNull(object));  //打印：false

```

# requireNonNull()

类`Objects`的以下方法检查第一个参数的值，如果值为`null`，则抛出`NullPointerException`或返回提供的默认值：

+   `T requireNonNull(T obj)`: 如果参数为`null`，则抛出没有消息的`NullPointerException`：

```java

String object = null;

尝试{

Objects.requireNonNull(object);

} 捕获 (NullPointerException ex){

System.out.println(ex.getMessage());  //打印：null

}

```

+   `T requireNonNull(T obj, String message)`: 如果第一个参数为`null`，则抛出带有提供的消息的`NullPointerException`：

```java

String object = null;

尝试{

Objects.requireNonNull(object, "参数'object'为空");

} 捕获 (NullPointerException ex){

System.out.println(ex.getMessage());

//参数'object'为空

}

```

+   `T requireNonNull(T obj, Supplier<String> messageSupplier)`: 如果第一个参数为`null`，则返回由提供的函数生成的消息，如果生成的消息或函数本身为`null`，则抛出`NullPointerException`：

```java

String object = null;

Supplier<String> msg1 = () -> {

String msg = "来自数据库的消息";

//从数据库获取相应的消息

返回消息;

};

尝试{

Objects.requireNonNull(object, msg1);

} 捕获 (NullPointerException ex){

System.out.println(ex.getMessage());  //打印：来自数据库的消息

}

Supplier<String> msg2 = () -> null;

尝试{

Objects.requireNonNull(object, msg2);

} 捕获 (NullPointerException ex){

System.out.println(ex.getMessage());  //打印：null

}

Supplier<String> msg3 = null;

尝试{

Objects.requireNonNull(object, msg3);

} 捕获 (NullPointerException ex){

System.out.println(ex.getMessage());  //打印：null

}

```

+   `T requireNonNullElse(T obj, T defaultObj)`: 如果第一个参数非空，则返回第一个参数值，如果第二个参数非空，则返回第二个参数值，否则抛出带有消息`defaultObj`的`NullPointerException`：

```java

String object = null;

System.out.println(Objects.requireNonNullElse(object,

"默认值"));

//打印：默认值

尝试{

Objects.requireNonNullElse(object, null);

} 捕获 (NullPointerException ex){

System.out.println(ex.getMessage());     //打印：defaultObj

}

```

+   `T requireNonNullElseGet(T obj, Supplier<? extends T> supplier)`: 如果第一个参数非空，则返回第一个参数值，如果第一个参数非空，则返回由提供的函数生成的对象，否则抛出带有消息`defaultObj`的`NullPointerException`：

```java

String object = null;

Supplier<String> msg1 = () -> {

String msg = "来自数据库的消息";

//从数据库获取相应的消息

返回消息;

};

String s = Objects.requireNonNullElseGet(object, msg1);

System.out.println(s);                //打印：来自数据库的消息

Supplier<String> msg2 = () -> null;

尝试{

System.out.println(Objects.requireNonNullElseGet(object, msg2));

} 捕获 (NullPointerException ex){

System.out.println(ex.getMessage()); //打印：supplier.get()

}

尝试{

System.out.println(Objects.requireNonNullElseGet(object, null));

} 捕获 (NullPointerException ex){

System.out.println(ex.getMessage()); //打印：供应商

}

```

# checkIndex()

以下一组方法检查集合或数组的索引和长度是否兼容：

+   `int checkIndex(int index, int length)`: throws `IndexOutOfBoundsException` if the provided `index` is bigger than `length - 1`

+   `int checkFromIndexSize(int fromIndex, int size, int length)`: throws `IndexOutOfBoundsException` if the provided `index + size` is bigger than `length - 1`

+   `int checkFromToIndex(int fromIndex, int toIndex, int length)`: throws `IndexOutOfBoundsException` if the provided `fromIndex` is bigger than `toIndex`, or `toIndex` is bigger than `length - 1`

Here is the demo code:

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

The method `int compare(T a, T b, Comparator<T> c)` of the class `Objects` uses the provided comparator's method `compare(T o1, T o2)` for comparing the two objects. We have described already the behavior of the `compare(T o1, T o2)` method while talking about sorting collections, so the following results should be expected:

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

As we have mentioned already, the method `compare(T o1, T o2)` returns the difference of positions of objects `o1` and `o2` in a sorted list for `String` objects and just `-1`, `0`, or `1` for `Integer` objects. The API describes it as returning `0` when objects are equal and a negative number when the first object is smaller than the second; otherwise, it returns a positive number.

To demonstrate how the method `compare(T a, T b, Comparator<T> c)` works, let's assume that we want to sort objects of the class `Person` so that the name and age are arranged in a natural order of `String` and `Integer` classes, respectively:

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

And here is the result of this new implementation of the `compareTo(Object)` method of the class `Person`:

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

As you can see, the `Person` objects are ordered by name in their natural order first, then by age in their natural order too. If we need to reverse the order of names, for example, we change the `compareTo(Object)` method to the following:

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

The results looks as like we expected:

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

方法 compare(T a, T b, Comparator<T> c)的弱点是它不处理 null 值。将 new Person(25, null)对象添加到列表中会在排序过程中触发 NullPointerException。在这种情况下，最好使用 org.apache.commons.lang3.ObjectUtils.compare(T o1, T o2)方法，我们将在下一节中进行演示。

# toString()

有时您需要将对象（它是对某个类类型的引用）转换为其字符串表示形式。当引用 obj 被赋予 null 值（对象尚未创建）时，编写 obj.toString()会生成 NullPointerException。对于这种情况，使用 Objects 类的以下方法是更好的选择：

+   `String toString(Object o)`: 当第一个参数不为 null 时，返回调用 toString()的结果；当第一个参数值为 null 时，返回 null

+   `String toString(Object o, String nullDefault)`: 当第一个参数不为 null 时，返回调用 toString()的结果；当第一个参数值为 null 时，返回第二个参数值 nullDefault

以下是演示如何使用这些方法的代码：

```java

List<String> list = new ArrayList<>(List.of("s0 "));

list.add(null);

对于列表中的每个元素 e：

System.out.print(e);                   //prints: s0 null

}

System.out.println();

对于列表中的每个元素 e：

System.out.print(Objects.toString(e)); //prints: s0 null

}

System.out.println();

对于列表中的每个元素 e：

System.out.print(Objects.toString(e, "element was null"));

//prints: s0 element was null

}

```

顺便说一句，与当前讨论无关，请注意我们如何使用 print()方法而不是 println()方法来在一行中显示所有结果，因为 print()方法不会添加换行符号。

# Class lang3.ObjectUtils

Apache Commons 库的 org.apache.commons.lang3.ObjectUtils 类补充了先前描述的 java.util.Objects 类的方法。本书的范围和分配的大小不允许对 ObjectUtils 类的所有方法进行详细审查，因此我们将简要描述它们，按相关功能进行分组，并仅演示与我们已经提供的示例一致的方法。

类 ObjectUtils 的所有方法可以分为七组：

+   对象克隆方法：

+   `T clone(T obj)`: 如果实现了 Cloneable 接口，则返回提供对象的副本；否则返回 null。

+   `T cloneIfPossible(T obj)`: 如果实现了 Cloneable 接口，则返回提供对象的副本；否则返回原始提供的对象。

+   支持对象比较的方法：

+   `int compare(T c1, T c2)`: 比较实现 Comparable 接口的两个对象的新排序位置；允许任何或两个参数为 null；将 null 值放在非 null 值前面。

+   `int compare(T c1, T c2, boolean nullGreater)`: 如果参数 nullGreater 的值为 false，则与前一个方法的行为完全相同；否则，将 null 值放在非 null 值后面。我们可以通过在我们的 Person 类中使用它们来演示最后两种方法：

```java

@Override

public int compareTo(Person p){

int result = ObjectUtils.compare(this.name, p.getName());

if (result != 0) {

返回 result;

}

返回 ObjectUtils.compare(this.age, p.getAge())的值

}

```

这种改变的结果使我们可以对 name 字段使用 null 值：

```java

创建一个名为 p1 的人类对象，年龄为 15 岁，名字为 Zoe

Person p2 = new Person(45, "Adam");

Person p3 = new Person(37, "Bob");

Person p4 = new Person(30, "Bob");

Person p5 = new Person(25, null);

List<Person> list = new ArrayList<>(List.of(p1, p2, p3, p4, p5));

System.out.println(list);  //[{15, Zoe}, {45, Adam}, {37, Bob}, {30, Bob}, {25, }]

Collections.sort(list);

System.out.println(list);  //[{25, }, {45, Adam}, {30, Bob}, {37, Bob}, {15, Zoe}]

```

由于我们使用了`Objects.compare(T c1, T c2)`方法，`null`值被放在了非空值的前面。顺便说一句，您是否注意到我们不再显示`null`了？那是因为我们已经修改了`Person`类的`toString()`方法如下：

```java

@Override

public String toString() {

//return "{" + age + ", " + name + "}";

return "{" + age + ", " + Objects.toString(name, "") + "}";

}

```

与其仅仅显示字段`name`的值，我们使用了`Objects.toString(Object o, String nullDefault)`方法，当对象为`null`时，它会用提供的`nullDefault`值替换对象。在这种情况下是否使用这种方法，是一个风格问题。许多程序员可能会认为我们必须显示实际值，而不是用其他值替换它。但是，我们这样做只是为了展示方法`Objects.toString(Object o, String nullDefault)`的用法。

如果我们现在使用第二个`compare(T c1, T c2, boolean nullGreater)`方法，`Person`类的`compareTo()`方法将如下所示：

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

然后，`name`设置为`null`的`Person`对象将显示在排序后的列表末尾：

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

并且，为了完成对`null`值的讨论，当将`null`对象添加到列表中时，上述代码将会因为`NullPointerException`而中断：`list.add(null)`。为了避免异常，您可以使用一个特殊的`Comparator`对象来处理列表中的`null`元素：

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

在这段代码中，您可以看到我们已经表明了希望在列表末尾看到`null`对象。相反，我们可以使用另一个`Comparator`，将`null`对象放在排序后的列表的开头：

```java

Collections.sort(list,

Comparator.nullsFirst(Comparator.naturalOrder()));

System.out.println(list);

//[null, {45, Adam}, {30, Bob}, {37, Bob}, {15, Zoe}, {25, }]

```

+   `notEqual`:

+   `boolean notEqual(Object object1, Object object2)`: compares two objects for inequality, where either one or both objects may be `null`

+   `identityToString`:

+   `String identityToString(Object object)`: returns the `String` representation of the provided object as if produced by the default method `toString()` of the base class `Object`

+   `void identityToString(StringBuffer buffer, Object object)`: appends the `String` representation of the provided object as if produced by the default method `toString()` of the base class `Object`

+   `void identityToString(StringBuilder builder, Object object)`: appends the `String` representation of the provided object as if produced by the default method `toString()` of the base class `Object`

+   `void identityToString(Appendable appendable, Object object)`: 将提供的对象的`String`表示附加到`appendable`，就好像是由基类`Object`的默认方法`toString()`生成的一样

以下代码演示了这两种方法：

```java

String s = "s0 " + ObjectUtils.identityToString("s1");

System.out.println(s);  //打印：s0 java.lang.String@5474c6c

StringBuffer sb = new StringBuffer();

sb.append("s0");

ObjectUtils.identityToString(sb, "s1");

System.out.println(s);  //打印：s0 java.lang.String@5474c6c

```

+   `allNotNull` 和 `anyNotNull`:

+   `boolean allNotNull(Object... values)`: 当提供的数组中的所有值都不为`null`时返回`true`

+   `boolean anyNotNull(Object... values)`: 当提供的数组中至少有一个值不为`null`时返回`true`

+   `firstNonNull` 和 `defaultIfNull`:

+   `T firstNonNull(T... values)`: 从提供的数组中返回第一个不为`null`的值

+   `T defaultIfNull(T object, T defaultValue)`: 如果第一个参数为`null`，则返回提供的默认值

+   `max`, `min`, `median`, 和 `mode`:

+   `T max(T... values)`: 返回实现`Comparable`接口的提供的值的有序列表中的最后一个值；只有当所有值都为`null`时才返回`null`

+   `T min(T... values)`: 返回实现`Comparable`接口的提供的值的有序列表中的第一个值；只有当所有值都为`null`时才返回`null`

+   `T median(T... items)`: 返回实现`Comparable`接口的提供的值的有序列表中间的值；如果值的数量是偶数，则返回中间两个中较小的一个

+   `T median(Comparator<T> comparator, T... items)`: 返回根据提供的`Comparator`对象排序的提供的值列表中间的值；如果值的数量是偶数，则返回中间两个中较小的一个

+   `T mode(T... items)`: 从提供的项目中返回出现频率最高的项目；当最常出现此类项目或没有一个项目最常出现时，返回`null`；以下是演示此最后一个方法的代码：

```java

String s = ObjectUtils.mode("s0", "s1", "s1");

System.out.println(s);     //打印：s1

s = ObjectUtils.mode("s0", "s1", "s2");

System.out.println(s);     //打印：null

s = ObjectUtils.mode("s0", "s1", "s2", "s1", "s2");

System.out.println(s);     //打印：null

s = ObjectUtils.mode(null);

System.out.println(s);     //打印：null

s = ObjectUtils.mode("s0", null, null);

System.out.println(s);     //打印：null

```

# 管理字符串

类`String`被广泛使用。因此，您必须熟悉其功能。我们已经在第五章中讨论了`String`值的不可变性，*Java 语言元素和类型*。我们已经表明，每次“修改”`String`值时，都会创建该值的新副本，这意味着在多次“修改”的情况下，会创建许多`String`对象，消耗内存并给 JVM 带来负担。

在这种情况下，建议使用类`java.lang.StringBuilder`或`java.lang.StringBuffer`，因为它们是可修改的对象，不需要创建`String`值副本。我们将展示如何使用它们，并在本节的第一部分解释这两个类之间的区别。

之后，我们将回顾类`String`的方法，然后概述类`org.apache.commons.lang3.StringUtils`，它补充了类`String`的功能。

# StringBuilder 和 StringBuffer

`StringBuilder`和`StringBuffer`类具有完全相同的方法列表。不同之处在于`StringBuilder`类的方法执行速度比`StringBuffer`类的相同方法更快。这是因为`StringBuffer`类在不允许从不同的应用程序线程并发访问其值时有开销。因此，如果您不是为多线程处理编码，请使用`StringBuilder`。

`StringBuilder`和`StringBuffer`类中有许多方法。但是，我们将展示如何仅使用`append()`方法，这是目前最受欢迎的方法，用于需要多个`String`值修改的情况。它的主要功能是将一个值附加到已存储在`StringBuilder`（或`StringBuffer`）对象中的值的末尾。

`append()`方法对所有原始类型和`String`、`Object`、`CharSequence`和`StringBuffer`类进行了重载，这意味着传入任何这些类的对象的`String`表示都可以附加到现有值上。对于我们的演示，我们将只使用`append(String s)`版本，因为这是您可能大部分时间要使用的。这是一个例子：

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

`StringBuilder`（和`StringBuffer`）类中还有`replace()`、`substring()`和`insert()`方法，允许进一步修改值。但是，它们的使用频率远低于`append()`方法，我们不会讨论它们，因为它们超出了本书的范围。

# Class java.lang.String

`String`类有 15 个构造函数和近 80 个方法。详细讨论并演示每一个对于本书来说太多了，因此我们只会评论最受欢迎的方法，并提及其余部分。掌握基础知识后，您可以阅读在线文档，了解`String`类的其他方法可以做什么。

# 构造函数

如果您担心应用程序创建的字符串占用太多内存，则`String`类的构造函数很有用。问题在于，`String`文字（例如`abc`）存储在内存的特殊区域中，称为“字符串常量池”，并且永远不会被垃圾回收。这样设计的理念是，`String`文字占用的内存比数字多得多。此外，处理这样大的实体会产生开销，可能会对 JVM 产生负担。这就是为什么设计者认为将它们存储并在所有应用程序线程之间共享比分配新内存然后多次清理相同值更便宜。

但是，如果`String`值的重用率很低，而存储的`String`值占用的内存太多，使用构造函数创建`String`对象可能是解决问题的方法。这是一个例子：

```java

String veryLongText = new String("asdakjfakjn akdb aakjn... akdjcnak");

```

以这种方式创建的`String`对象驻留在堆区域（存储所有对象的地方），并且在不再使用时进行垃圾回收。这就是`String`构造函数的用武之地。

如果需要，可以使用`String`类的`intern()`方法在字符串常量池中创建堆`String`对象的副本。它不仅允许我们与其他应用程序线程共享值（在多线程处理中），还允许我们通过引用（使用运算符`==`）将其与另一个文字值进行比较。如果引用相等，则表示它们指向池中相同的`String`值。

但是，主流程序员很少以这种方式管理内存，因此我们不会进一步讨论这个话题。

# format()

`String format(String format, Object... args)`方法允许将提供的对象插入到字符串的指定位置并根据需要进行格式化。在`java.util.Formatter`类中有许多格式说明符。我们这里只演示`%s`，它通过调用对象的`toString()`方法将传入的对象转换为其`String`表示形式： 

```java

String format = "There is a %s in the %s";

String s = String.format(format, "bear", "woods");

System.out.println(s); //prints: There is a bear in the woods

format = "Class %s is very useful";

s = String.format(format, new A());

System.out.println(s);  //prints: Class A is very useful

```

# replace()

`String replace(CharSequence target, CharSequence replacement)`方法在`String`值中用第一个参数的值替换第二个参数的值：

```java

String s1 = "There is a bear in the woods";

String s2 = s1.replace("bear", "horse").replace("woods", "field");

System.out.println(s2);     //prints: There is a horse in the field

```

还有`String replaceAll(String regex, String replacement)`和`String replaceFirst(String regex, String replacement)`方法，具有类似的功能。

# compareTo()

我们已经在示例中使用了`int compareTo(String anotherString)`方法。它返回此`String`值的位置和有序列表中`anotherString`值的位置之间的差异。它用于字符串的自然排序，因为它是`Comparable`接口的实现。

`int compareToIgnoreCase(String str)`方法执行相同的功能，但忽略比较字符串的大小写，并且不用于自然排序，因为它不是`Comparable`接口的实现。

# valueOf(Objectj)

静态方法`String valueOf(Object obj)`如果提供的对象为`null`，则返回`null`，或者在提供的对象上调用`toString()`方法。

# valueOf(primitive or char[])

任何原始类型值都可以作为参数传递给静态方法`String valueOf(primitive value)`，它返回提供的值的字符串表示形式。例如，`String.valueOf(42)`返回`42`。这组方法包括以下静态方法：

+   `String valueOf(boolean b)`

+   `String valueOf(char c)`

+   `String valueOf(double d)`

+   `String valueOf(float f)`

+   `String valueOf(int i)`

+   `String valueOf(long l)`

+   `String valueOf(char[] data)`

+   `String valueOf(char[] data, int offset, int count)`

# copyValueOf(char[])

`String copyValueOf(char[] data)`方法等同于`valueOf(char[])`，而`String copyValueOf(char[] data, int offset, int count)`方法等同于`valueOf(char[], int, int)`。它们返回字符数组或其子数组的`String`表示形式。

`void getChars(int srcBegin, int srcEnd, char[] dest, int dstBegin)`方法将此`String`值中的字符复制到目标字符数组中。

# indexOf() and substring()

各种`int indexOf(String str)`和`int lastIndexOf(String str)`方法返回字符串中子字符串的位置：

```java

String s = "Introduction";

System.out.println(s.indexOf("I"));      //prints: 0

System.out.println(s.lastIndexOf("I"));  //prints: 0

System.out.println(s.lastIndexOf("i"));  //prints: 9

System.out.println(s.indexOf("o"));      //prints: 4

System.out.println(s.lastIndexOf("o"));  //prints: 10

System.out.println(s.indexOf("tro"));    //prints: 2

```

注意，位置计数从零开始。

`String substring(int beginIndex)`方法返回字符串值的其余部分，从作为参数传递的位置（索引）开始：

```java

String s = "Introduction";

System.out.println(s.substring(1));        //prints: ntroduction

System.out.println(s.substring(2));        //prints: troduction

```

`beginIndex`位置的字符是前面子字符串中存在的第一个字符。

方法`String substring(int beginIndex, int endIndex)`返回从作为第一个参数传递的位置开始到作为第二个参数传递的位置结束的子字符串：

```java

String s = "Introduction";

System.out.println(s.substring(1, 2));        //输出：n

System.out.println(s.substring(1, 3));        //输出：nt

```

与`substring(beginIndex)`方法一样，`beginIndex`位置的字符是前面子字符串中存在的第一个字符，而`endIndex`位置的字符不包括在内。`endIndex - beginIndex`的差等于子字符串的长度。

这意味着以下两个子字符串是相等的：

```java

System.out.println(s.substring(1));              //输出：ntroduction

System.out.println(s.substring(1, s.length()));  //输出：ntroduction

```

# contains()和 matches()

方法`boolean contains(CharSequence s)`返回`true`当提供的字符序列（子字符串）存在时：

```java

String s = "Introduction";

System.out.println(s.contains("x"));          //输出：false

System.out.println(s.contains("o"));          //输出：true

System.out.println(s.contains("tro"));        //输出：true

System.out.println(s.contains("trx"));        //输出：false

```

其他类似的方法有：

+   `boolean matches(String regex)`: 使用正则表达式（不是本书的主题）

+   `boolean regionMatches(int tOffset, String other, int oOffset, int length)`: 比较两个字符串的区域

+   `boolean regionMatches(boolean ignoreCase, int tOffset, String other, int oOffset, int length)`: 与上面相同，但标志`ignoreCase`指示是否忽略大小写

# split()，concat()和 join()

方法`String[] split(String regex)`和`String[] split(String regex, int limit)`使用传递的正则表达式将字符串拆分为子字符串。我们在本书中不解释正则表达式。但是，有一个非常简单的正则表达式，即使您对正则表达式一无所知，也很容易使用：如果您只是将字符串中存在的任何符号或子字符串传递到此方法中，字符串将被拆分为由传递的值分隔的部分，例如：

```java

String[] substrings = "Introduction".split("o");

System.out.println(Arrays.toString(substrings));

//输出：[Intr, ducti, n]

substrings = "Introduction".split("duct");

System.out.println(Arrays.toString(substrings));

//输出：[Intro, ion]

```

这段代码只是说明了功能。但是以下代码片段更实用：

```java

String s = "There is a bear in the woods";

String[] arr = s.split(" ");

System.out.println(Arrays.toString(arr));

//输出：[There, is, a, bear, in, the, woods]

arr = s.split(" ", 3);

System.out.println(Arrays.toString(arr));

//输出：[There, is, a bear in the woods]

```

正如你所看到的，`split()`方法中的第二个参数限制了结果子字符串的数量。

方法`String concat(String str)`将传递的值添加到字符串的末尾：

```java

String s1 =  "There is a bear";

String s2 =  " in the woods";

String s = s1.concat(s2);

System.out.println(s);  //输出：There is a bear in the woods

```

`concat()`方法创建一个新的`String`值，其结果是连接，因此它是非常经济的。但是，如果您需要添加（连接）许多值，使用`StringBuilder`（或`StringBuffer`，如果您需要保护免受并发访问）将是更好的选择。我们在前一节中讨论过。另一个选择是使用运算符`+`：

```java

String s =  s1 + s2;

System.out.println(s);  //输出：There is a bear in the woods

```

当与`String`值一起使用时，运算符`+`是基于`StringBuilder`实现的，因此允许通过修改现有的`String`值来添加`String`值。使用 StringBuilder 和仅使用运算符`+`添加`String`值之间没有性能差异。

方法`String join(CharSequence delimiter, CharSequence... elements)`和`String join(CharSequence delimiter, Iterable<? extends CharSequence> elements)`也基于`StringBuilder`。它们使用传入的`delimiter`将提供的值组装成一个`String`值，在创建的`String`结果中使用`delimiter`分隔组装的值。这是一个例子：

```java

s = String.join(" ", "There", "is", "a", "bear", "in", "the", "woods");

System.out.println(s); //输出：There is a bear in the woods

List<String> list =

List.of("There", "is", "a", "bear", "in", "the", "woods");

s = String.join(" ", list);

System.out.println(s); //输出：There is a bear in the woods

```

# startsWith()和 endsWith()

以下方法在字符串值以提供的子字符串`prefix`开头（或结尾）时返回`true`：

+   `boolean startsWith(String prefix)`

+   `boolean startsWith(String prefix, int toffset)`

+   `boolean endsWith(String suffix)`

以下是演示代码：

```java

boolean b = "Introduction".startsWith("Intro");

System.out.println(b); //输出：true

b = "Introduction".startsWith("tro", 2);

System.out.println(b); //输出：true

b = "Introduction".endsWith("ion");

System.out.println(b); //输出：true

```

# equals()和 equalsIgnoreCase()

我们已经多次使用了`String`类的`boolean equals(Object anObject)`方法，并指出它将此`String`值与其他对象进行比较。当传入的对象是具有相同值的`String`时，此方法仅返回`true`。

方法`boolean equalsIgnoreCase(String anotherString)`也是如此，但还忽略大小写，因此字符串`AbC`和`ABC`被视为相等。

# contentEquals()和 copyValueOf()

方法`boolean contentEquals(CharSequence cs)`将此`String`值与实现接口`CharSequence`的对象的`String`表示进行比较。流行的`CharSequence`实现包括`CharBuffer`、`Segment`、`String`、`StringBuffer`和`StringBuilder`。

方法`boolean contentEquals(StringBuffer sb)`也是如此，但仅适用于`StringBuffer`。它的实现与`contentEquals(CharSequence cs)`略有不同，并且在某些情况下可能具有一些性能优势，但我们不打算讨论这些细节。此外，当您在`String`值上调用`contentEquals()`时，您可能甚至不会注意到使用了这两种方法中的哪一种，除非您努力利用差异。

# length()、isEmpty()和 hashCode()

方法`int length()`返回`String`值中的字符数。

方法`boolean isEmpty()`在`String`值中没有字符时返回`true`，而方法`length()`返回零。

方法`int hashCode()`返回`String`对象的哈希值。

# trim()、toLowerCase()和 toUpperCase()

方法`String trim()`从`String`值中删除前导和尾随空格。

以下方法更改`String`值中字符的大小写：

+   `String toLowerCase()`

+   `String toUpperCase()`

+   `String toLowerCase(Locale locale)`

+   `String toUpperCase(Locale locale)`

# getBytes()、getChars()和 toCharArray()

以下方法将`String`值转换为字节数组，可选择使用给定的字符集进行编码：

+   `byte[] getBytes()`

+   `byte[] getBytes(Charset charset)`

+   `byte[] getBytes(String charsetName)`

这些方法将`String`值全部转换为其他类型，或仅转换部分：

+   `IntStream chars()`

+   `char[] toCharArray()`

+   `char charAt(int index)`

+   `CharSequence subSequence(int beginIndex, int endIndex)`

# 按索引或流获取代码点

以下一组方法将`String`值全部或部分转换为其字符的 Unicode 代码点：

+   `IntStream codePoints()`

+   `int codePointAt(int index)`

+   `int codePointBefore(int index)`

+   `int codePointCount(int beginIndex, int endIndex)`

+   `int offsetByCodePoints(int index, int codePointOffset)`

我们在第五章中解释了 Unicode 代码点，*Java 语言元素和类型*。当您需要表示*不适合*`char`类型的字符时，这些方法特别有用。这些字符的代码点大于`Character.MAX_VALUE`，即`65535`。

# Class lang3.StringUtils

Apache Commons 库的`org.apache.commons.lang3.StringUtils`类具有超过 120 个静态实用方法，这些方法补充了我们在上一节中描述的`String`类的方法。

最受欢迎的静态方法包括以下内容：

+   `boolean isBlank(CharSequence cs)`: 当传入的参数是空字符串""、`null`或空格时返回`true`

+   `boolean isNotBlank(CharSequence cs)`: 当传入的参数不是空字符串""、`null`或空格时返回`true`

+   `boolean isAlpha(CharSequence cs)`: 当传入的参数只包含 Unicode 字母时返回`true`

+   `boolean isAlphaSpace(CharSequence cs)`: 当传入的参数只包含 Unicode 字母和空格(' ')时返回`true`

+   `boolean isNumeric(CharSequence cs)`: 当传入的参数只包含数字时返回`true`

+   `boolean isNumericSpace(CharSequence cs)`: 当传入的参数只包含数字和空格(' ')时返回`true`

+   `boolean isAlphaNumeric(CharSequence cs)`: 当传入的参数只包含 Unicode 字母和数字时返回`true`

+   `boolean isAlphaNumericSpace(CharSequence cs)`: 当传入的参数只包含 Unicode 字母、数字和空格(' ')时返回`true`

我们强烈建议您查看此类的 API，并了解您可以在其中找到什么。

# 时间管理

`java.time`包及其子包中有许多类。它们被引入作为处理日期和时间的其他旧包的替代品。新类是线程安全的（因此更适合多线程处理），而且同样重要的是，设计更加一致且更易于理解。此外，新的实现遵循**国际标准化组织**（**ISO**）的日期和时间格式，但也允许使用任何其他自定义格式。

我们将描述主要的五个类，并演示如何使用它们：

+   `java.util.LocalDate`

+   `java.util.LocalTime`

+   `java.util.LocalDateTime`

+   `java.util.Period`

+   `java.util.Duration`

所有这些以及`java.time`包及其子包的其他类都具有各种功能，涵盖了所有实际和任何想象得到的情况。但我们不打算覆盖所有这些，只是介绍基础知识和最流行的用例。

# java.time.LocalDate

`LocalDate`类不包含时间。它表示 ISO 8601 格式的日期，yyyy-MM-DD：

```java

System.out.println(LocalDate.now());   //输出：2018-04-14

```

如您所见，`now()`方法返回您计算机上设置的当前日期：`2018 年 4 月 14 日`是编写本节时的日期。

同样，您可以使用静态方法`now(ZoneId zone)`在任何其他时区获取当前日期。`ZoneId`对象可以使用静态方法`ZoneId.of(String zoneId)`构造，其中`String zoneId`是方法`ZonId.getAvailableZoneIds()`返回的任何`String`值之一：

```java

Set<String> zoneIds = ZoneId.getAvailableZoneIds();

for(String zoneId: zoneIds){

System.out.println(zoneId);

}

```

此代码打印了许多时区 ID，其中之一是`Asia/Tokyo`。现在，我们可以找到在该时区中现在是什么日期：

```java

ZoneId zoneId = ZoneId.of("Asia/Tokyo");

System.out.println(LocalDate.now(zoneId));   //输出：2018-04-15

```

`LocalDate`对象可以使用以下方法表示过去或未来的任何日期：

+   `LocalDate parse(CharSequence text)`: 从 ISO 8601 格式的字符串 yyyy-MM-DD 构造对象

+   `LocalDate parse(CharSequence text, DateTimeFormatter formatter) `: 用`DateTimeFormatter`对象指定的格式从字符串构造一个对象，该对象有许多预定义的格式

+   `LocalDate of(int year, int month, int dayOfMonth)`: 从年、月和日构造一个对象

+   `LocalDate of(int year, Month month, int dayOfMonth)`: 从年、月（作为`enum`常量）和日构造一个对象

+   `LocalDate ofYearDay(int year, int dayOfYear)`: 从年和一年中的某一天构造一个对象

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

使用`LocalDate`对象，你可以得到各种值：

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

`LocalDate`对象可以进行比较：

```java

LocalDate lc6 =  LocalDate.parse("2020-02-22");

LocalDate lc7 =  LocalDate.parse("2020-02-23");

System.out.println(lc6.isAfter(lc7));       //prints: false

System.out.println(lc6.isBefore(lc7));      //prints: true

```

`LocalDate`类中还有许多其他有用的方法。如果你必须处理日期，我们建议你阅读这个类和`java.time`包及其子包的 API。

# java.time.LocalTime

`LocalTime`类包含没有日期的时间。它有类似于`LocalDate`类的方法。

下面是如何创建`LocalTime`类的对象：

```java

System.out.println(LocalTime.now());         //prints: 21:15:46.360904

ZoneId zoneId = ZoneId.of("Asia/Tokyo");

System.out.println(LocalTime.now(zoneId));   //prints: 12:15:46.364378

LocalTime lt1 =  LocalTime.parse("20:23:12");

System.out.println(lt1);                     //prints: 20:23:12

LocalTime lt2 =  LocalTime.of(20, 23, 12);

System.out.println(lt2);                     //prints: 20:23:12

```

时间值的每个组件可以从`LocalTime`对象中提取如下：

```java

System.out.println(lt2.getHour());          //prints: 20

System.out.println(lt2.getMinute());        //prints: 23

System.out.println(lt2.getSecond());        //prints: 12

System.out.println(lt2.getNano());          //prints: 0

```

对象可以被修改：

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

And two objects of the `LocalTime` class can be compared as well:

```java

LocalTime lt3 =  LocalTime.parse("20:23:12");

LocalTime lt4 =  LocalTime.parse("20:25:12");

System.out.println(lt3.isAfter(lt4));       //prints: false

System.out.println(lt3.isBefore(lt4));      //prints: true

```

There are many other helpful methods in the `LocalTime` class. If you have to work with time, we recommend you read the API of this class and other classes of the `java.time` package and its sub-packages.

# java.time.LocalDateTime

The class `LocalDateTime` contains both date and time, and has all the methods the classes `LocalDate` and `LocalTime` have, so we are not going to repeat them here. We will only show how an object of `LocalDateTime` can be created:

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

There are many other helpful methods in the `LocalDateTime` class. If you have to work with date and time, we recommend you read the API of this class and other classes of the `java.time` package and its sub-packages.

# Period and Duration

The classes `java.time.Period` and `java.time.Duration` are designed to contain an amount of time:

+   The `Period` object contains an amount of time in units of years, months, and days

+   The `Duration` object contains an amount of time in hours, minutes, seconds, and nanoseconds

The following code demonstrates their creation and use with the class `LocalDateTime`, but the same methods exist in the classes `LocalDate` (for `Period`) and `LocalTime` (for `Duration`):

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

Some other ways to create and use `Period` objects are demonstrated in the following code:

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

`Duration`的对象可以类似地创建和使用：

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

`Period`和`Duration`类中还有许多其他有用的方法。如果您必须处理时间量，我们建议您阅读这些类和`java.time`包及其子包的 API。

# 管理随机数

生成真正随机数是一个大课题，不属于本书的范围。但对于绝大多数实际目的而言，Java 提供的伪随机数生成器已经足够好，这就是我们将在本节讨论的内容。

在 Java 标准库中有两种主要生成随机数的方法：

+   `java.lang.Math.random()`方法

+   `java.util.Random`类

还有`java.security.SecureRandom`类，它提供了一个密码强度的随机数生成器，但它超出了入门课程的范围。

# 方法`java.lang.Math.random()`

类`Math`的静态方法`double random()`返回大于或等于`0.0`且小于`1.0`的`double`类型值：

```java

for(int i =0; i < 3; i++){

System.out.println(Math.random());

//0.9350483840148613

//0.0477353019234189

//0.25784245516898985

}

```java

我们在前面的注释中捕获了结果。但实际上，通常需要从某个范围内获得随机整数。为了满足这种需求，我们可以编写一个方法，例如，从 0（包括）到 10（不包括）产生一个随机整数：

```java

int getInteger(int max){

return (int)(Math.random() * max);

}

```

以下是前面代码的一次运行结果：

```java

for(int i =0; i < 3; i++){

System.out.print(getInteger(10) + " "); //prints: 2 5 6

}

```

如您所见，它生成一个随机整数值，可以是以下 10 个数字之一：0、1、...、9。以下是使用相同方法并生成从 0（包括）到 100（不包括）的随机整数的代码：

```java

for(int i =0; i < 3; i++){

System.out.print(getInteger(100) + " "); //prints: 48 11 97

}

```

当您需要一个介于 100（包括）和 200（不包括）之间的随机数时，您只需将前面的结果加上 100：

```java

for(int i =0; i < 3; i++){

System.out.print(100 + getInteger(100) + " "); //prints: 114 101 127

}

```

在结果中包括范围的两端可以通过四舍五入生成的`double`值来实现：

```java

int getIntegerRound(int max){

return (int)Math.round(Math.random() * max);

}

```

当我们使用上述方法时，结果是：

```java

for(int i =0; i < 3; i++){

System.out.print(100 + getIntegerRound(100) + " "); //179 147 200

}

```

如您所见，范围的上限（数字 200）包括在可能的结果集中。通过仅将 1 添加到请求的上限范围，可以实现相同的效果：

```java

int getInteger2(int max){

return (int)(Math.random() * (max + 1));

}

```

如果我们使用前面的方法，我们可以得到以下结果：

```java

for(int i =0; i < 3; i++){

System.out.print(100 + getInteger2(100) + " "); //167 200 132

}

```

但是，如果您查看`Math.random()`方法的源代码，您会发现它使用`java.util.Random`类及其`nextDouble()`方法生成随机双精度值。因此，让我们看看如何直接使用`java.util.Random`类。

# 类 java.util.Random

类`Random`的方法`doubles()`生成一个大于或等于`0.0`且小于`1.0`的`double`类型的值：

```java

Random random = new Random();

for(int i =0; i < 3; i++){

System.out.print(random.nextDouble() + " ");

//prints: 0.8774928230544553 0.7822070124559267 0.09401796000707807

}

```

我们可以像在前一节中使用`Math.random()`一样使用方法`nextDouble()`。但是当需要某个范围内的随机整数值时，类还有其他方法可以使用，而无需创建自定义的`getInteger()`方法。例如，`nextInt()`方法返回一个介于`Integer.MIN_VALUE`（包括）和`Integer.MAX_VALUE`（包括）之间的整数值：

```java

for(int i =0; i < 3; i++){

System.out.print(random.nextInt() + " ");

//prints: -2001537190 -1148252160 1999653777

}

```

而带有参数的相同方法允许我们通过上限（不包括）限制返回值的范围：

```java

for(int i =0; i < 3; i++){

System.out.print(random.nextInt(11) + " "); //prints: 4 6 2

}

```

这段代码生成一个介于 0（包括）和 10（包括）之间的随机整数值。以下代码返回介于 11（包括）和 20（包括）之间的随机整数值：

```java

for(int i =0; i < 3; i++){

System.out.print(11 + random.nextInt(10) + " "); //prints: 13 20 15

}

```

另一种生成范围内随机整数的方法是使用方法`ints(int count, int min, int max)`返回的`IntStream`对象，其中`count`是请求的值的数量，`min`是最小值（包括），`max`是最大值（不包括）：

```java

String result = random.ints(3, 0, 101)

.mapToObj(String::valueOf)

.collect(Collectors.joining(" ")); //prints: 30 48 52

```

这段代码返回从 0（包括）到 100（包括）的三个整数值。我们将在第十八章中更多地讨论流，*流和管道*。

# Exercise – Objects.equals() result

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

当我们运行`Exercise`类的`main()`方法时，会显示什么？`错误`？`假`？`真`？

# Answer

显示将只显示一个值：`True`。原因是两个私有字段——`a`和`b`——都被初始化为`null`。

# Summary

在本章中，我们向读者介绍了 Java 标准库和 Apache Commons 库中最受欢迎的实用程序和其他一些类。每个 Java 程序员都必须对它们的功能有扎实的理解，才能成为一个有效的编码人员。研究它们还有助于接触各种软件设计模式和解决方案，这些模式和解决方案具有教育意义，并可以作为任何应用程序中最佳编码实践的模式。

在下一章中，我们将向读者演示如何编写 Java 代码，以便在数据库中操作（插入、读取、更新和删除）数据。它还将简要介绍 SQL 和基本的数据库操作。
