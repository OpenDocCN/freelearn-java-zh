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
boolean equals(Object a, Object b) {
    return (a == b) || (a != null && a.equals(b));
}
```java

boolean equals(Object a, Object b) {

return (a == b) || (a != null && a.equals(b));

}

```java
boolean deepEquals(Object a, Object b) {
    if (a == b)
        return true;
    else if (a == null || b == null)
        return false;
    else
        return Arrays.deepEquals0(a, b);
}
```java

boolean deepEquals(Object a, Object b) {

if (a == b)

return true;

else if (a == null || b == null)

return false;

else

return Arrays.deepEquals0(a, b);

}

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

```java
System.out.println(Objects.hash("s1"));           //prints: 3645
System.out.println(Objects.hashCode("s1"));       //prints: 3614

```java

System.out.println(Objects.hash("s1"));           //prints: 3645

System.out.println(Objects.hashCode("s1"));       //prints: 3614

```java
System.out.println(Objects.hash(null));      //prints: 0
System.out.println(Objects.hashCode(null));  //prints: 0

```java

System.out.println(Objects.hash(null));      //prints: 0

System.out.println(Objects.hashCode(null));  //prints: 0

```java
String object = null;

System.out.println(object == null);           //prints: true
System.out.println(Objects.isNull(object));   //prints: true

System.out.println(object != null);           //prints: false
System.out.println(Objects.nonNull(object));  //prints: false
```java

String object = null;

System.out.println(object == null);           //打印：true

System.out.println(Objects.isNull(object));   //打印：true

System.out.println(object != null);           //打印：false

System.out.println(Objects.nonNull(object));  //打印：false

```java
      String object = null;
      try {
          Objects.requireNonNull(object);
      } catch (NullPointerException ex){
          System.out.println(ex.getMessage());  //prints: null
      }
```java

String object = null;

尝试{

Objects.requireNonNull(object);

} 捕获 (NullPointerException ex){

System.out.println(ex.getMessage());  //打印：null

}

```java
      String object = null;
      try {
          Objects.requireNonNull(object, "Parameter 'object' is null");
      } catch (NullPointerException ex){
          System.out.println(ex.getMessage());  
          //Parameter 'object' is null
      }
```java

String object = null;

尝试{

Objects.requireNonNull(object, "参数'object'为空");

} 捕获 (NullPointerException ex){

System.out.println(ex.getMessage());

//参数'object'为空

}

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

```java
int diff = Objects.compare("a", "c", Comparator.naturalOrder());
System.out.println(diff);  //prints: -2
diff = Objects.compare("a", "c", Comparator.reverseOrder());
System.out.println(diff);  //prints: 2
diff = Objects.compare(3, 5, Comparator.naturalOrder());
System.out.println(diff);  //prints: -1
diff = Objects.compare(3, 5, Comparator.reverseOrder());
System.out.println(diff);  //prints: 1

```java

int diff = Objects.compare("a", "c", Comparator.naturalOrder());

System.out.println(diff);  //prints: -2

diff = Objects.compare("a", "c", Comparator.reverseOrder());

System.out.println(diff);  //prints: 2

diff = Objects.compare(3, 5, Comparator.naturalOrder());

System.out.println(diff);  //prints: -1

diff = Objects.compare(3, 5, Comparator.reverseOrder());

System.out.println(diff);  //prints: 1

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

```java
Person p1 = new Person(15, "Zoe");
Person p2 = new Person(45, "Adam");
Person p3 = new Person(37, "Bob");
Person p4 = new Person(30, "Bob");
List<Person> list = new ArrayList<>(List.of(p1, p2, p3, p4));
System.out.println(list);//[{15, Zoe}, {45, Adam}, {37, Bob}, {30, Bob}]
Collections.sort(list);
System.out.println(list);//[{45, Adam}, {30, Bob}, {37, Bob}, {15, Zoe}] 
```java

Person p1 = new Person(15, "Zoe");

Person p2 = new Person(45, "Adam");

Person p3 = new Person(37, "Bob");

Person p4 = new Person(30, "Bob");

List<Person> list = new ArrayList<>(List.of(p1, p2, p3, p4));

System.out.println(list);//[{15, Zoe}, {45, Adam}, {37, Bob}, {30, Bob}]

Collections.sort(list);

System.out.println(list);//[{45, Adam}, {30, Bob}, {37, Bob}, {15, Zoe}]

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

```java
Person p1 = new Person(15, "Zoe");
Person p2 = new Person(45, "Adam");
Person p3 = new Person(37, "Bob");
Person p4 = new Person(30, "Bob");
List<Person> list = new ArrayList<>(List.of(p1, p2, p3, p4));
System.out.println(list);//[{15, Zoe}, {45, Adam}, {37, Bob}, {30, Bob}]
Collections.sort(list);
System.out.println(list);//[{15, Zoe}, {30, Bob}, {37, Bob}, {45, Adam}] 
```java

Person p1 = new Person(15, "Zoe");

Person p2 = new Person(45, "Adam");

Person p3 = new Person(37, "Bob");

Person p4 = new Person(30, "Bob");

List<Person> list = new ArrayList<>(List.of(p1, p2, p3, p4));

System.out.println(list);//[{15, Zoe}, {45, Adam}, {37, Bob}, {30, Bob}]

Collections.sort(list);

System.out.println(list);//[{15, Zoe}, {30, Bob}, {37, Bob}, {45, Adam}]

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

```java
@Override
public int compareTo(Person p){
    int result = ObjectUtils.compare(this.name, p.getName());
    if (result != 0) {
        return result;
    }
    return ObjectUtils.compare(this.age, p.getAge());
}
```java

@Override

public int compareTo(Person p){

int result = ObjectUtils.compare(this.name, p.getName());

if (result != 0) {

返回 result;

}

返回 ObjectUtils.compare(this.age, p.getAge())的值

}

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

```java
@Override
public String toString() {
    //return "{" + age + ", " + name + "}";
    return "{" + age + ", " + Objects.toString(name, "") + "}";
}
```java

@Override

public String toString() {

//return "{" + age + ", " + name + "}";

return "{" + age + ", " + Objects.toString(name, "") + "}";

}

```java
@Override
public int compareTo(Person p){
    int result = ObjectUtils.compare(this.name, p.getName(), true);
    if (result != 0) {
        return result;
    }
    return ObjectUtils.compare(this.age, p.getAge());
}
```java

@Override

public int compareTo(Person p){

int result = ObjectUtils.compare(this.name, p.getName(), true);

if (result != 0) {

return result;

}

return ObjectUtils.compare(this.age, p.getAge());

}

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

```java
Collections.sort(list, 
                   Comparator.nullsFirst(Comparator.naturalOrder()));
System.out.println(list);  
        //[null, {45, Adam}, {30, Bob}, {37, Bob}, {15, Zoe}, {25, }]

```java

Collections.sort(list,

Comparator.nullsFirst(Comparator.naturalOrder()));

System.out.println(list);

//[null, {45, Adam}, {30, Bob}, {37, Bob}, {15, Zoe}, {25, }]

```java
String s = "s0 " + ObjectUtils.identityToString("s1");
System.out.println(s);  //prints: s0 java.lang.String@5474c6c

StringBuffer sb = new StringBuffer();
sb.append("s0");
ObjectUtils.identityToString(sb, "s1");
System.out.println(s);  //prints: s0 java.lang.String@5474c6c

```java

String s = "s0 " + ObjectUtils.identityToString("s1");

System.out.println(s);  //打印：s0 java.lang.String@5474c6c

StringBuffer sb = new StringBuffer();

sb.append("s0");

ObjectUtils.identityToString(sb, "s1");

System.out.println(s);  //打印：s0 java.lang.String@5474c6c

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

```java
List<String> list = 
  List.of("That", "is", "the", "way", "to", "build", "a", "sentence");
StringBuilder sb = new StringBuilder();
for(String s: list){
    sb.append(s).append(" ");
}
String s = sb.toString();
System.out.println(s);  //prints: That is the way to build a sentence

```java

List<String> list =

List.of("That", "is", "the", "way", "to", "build", "a", "sentence");

StringBuilder sb = new StringBuilder();

for(String s: list){

sb.append(s).append(" ");

}

String s = sb.toString();

System.out.println(s);  //prints: That is the way to build a sentence

```java
String veryLongText = new String("asdakjfakjn akdb aakjn... akdjcnak");
```java

String veryLongText = new String("asdakjfakjn akdb aakjn... akdjcnak");

```java
String format = "There is a %s in the %s";
String s = String.format(format, "bear", "woods");
System.out.println(s); //prints: There is a bear in the woods

format = "Class %s is very useful";
s = String.format(format, new A());
System.out.println(s);  //prints: Class A is very useful

```java

String format = "There is a %s in the %s";

String s = String.format(format, "bear", "woods");

System.out.println(s); //prints: There is a bear in the woods

format = "Class %s is very useful";

s = String.format(format, new A());

System.out.println(s);  //prints: Class A is very useful

```java
String s1 = "There is a bear in the woods";
String s2 = s1.replace("bear", "horse").replace("woods", "field");
System.out.println(s2);     //prints: There is a horse in the field

```java

String s1 = "There is a bear in the woods";

String s2 = s1.replace("bear", "horse").replace("woods", "field");

System.out.println(s2);     //prints: There is a horse in the field

```java
String s = "Introduction";
System.out.println(s.indexOf("I"));      //prints: 0
System.out.println(s.lastIndexOf("I"));  //prints: 0
System.out.println(s.lastIndexOf("i"));  //prints: 9
System.out.println(s.indexOf("o"));      //prints: 4
System.out.println(s.lastIndexOf("o"));  //prints: 10
System.out.println(s.indexOf("tro"));    //prints: 2
```java

String s = "Introduction";

System.out.println(s.indexOf("I"));      //prints: 0

System.out.println(s.lastIndexOf("I"));  //prints: 0

System.out.println(s.lastIndexOf("i"));  //prints: 9

System.out.println(s.indexOf("o"));      //prints: 4

System.out.println(s.lastIndexOf("o"));  //prints: 10

System.out.println(s.indexOf("tro"));    //prints: 2

```java
String s = "Introduction";
System.out.println(s.substring(1));        //prints: ntroduction
System.out.println(s.substring(2));        //prints: troduction

```java

String s = "Introduction";

System.out.println(s.substring(1));        //prints: ntroduction

System.out.println(s.substring(2));        //prints: troduction

```java
String s = "Introduction";
System.out.println(s.substring(1, 2));        //prints: n
System.out.println(s.substring(1, 3));        //prints: nt

```java

String s = "Introduction";

System.out.println(s.substring(1, 2));        //输出：n

System.out.println(s.substring(1, 3));        //输出：nt

```java
System.out.println(s.substring(1));              //prints: ntroduction
System.out.println(s.substring(1, s.length()));  //prints: ntroduction

```java

System.out.println(s.substring(1));              //输出：ntroduction

System.out.println(s.substring(1, s.length()));  //输出：ntroduction

```java
String s = "Introduction";
System.out.println(s.contains("x"));          //prints: false
System.out.println(s.contains("o"));          //prints: true
System.out.println(s.contains("tro"));        //prints: true
System.out.println(s.contains("trx"));        //prints: false

```java

String s = "Introduction";

System.out.println(s.contains("x"));          //输出：false

System.out.println(s.contains("o"));          //输出：true

System.out.println(s.contains("tro"));        //输出：true

System.out.println(s.contains("trx"));        //输出：false

```java
String[] substrings = "Introduction".split("o");
System.out.println(Arrays.toString(substrings)); 
                                       //prints: [Intr, ducti, n]
substrings = "Introduction".split("duct");
System.out.println(Arrays.toString(substrings)); 
                                      //prints: [Intro, ion] 

```java

String[] substrings = "Introduction".split("o");

System.out.println(Arrays.toString(substrings));

//输出：[Intr, ducti, n]

substrings = "Introduction".split("duct");

System.out.println(Arrays.toString(substrings));

//输出：[Intro, ion]

```java
String s = "There is a bear in the woods";
String[] arr = s.split(" ");
System.out.println(Arrays.toString(arr));  
                       //prints: [There, is, a, bear, in, the, woods]
arr = s.split(" ", 3);
System.out.println(Arrays.toString(arr));  
                          //prints: [There, is, a bear in the woods]

```java

String s = "There is a bear in the woods";

String[] arr = s.split(" ");

System.out.println(Arrays.toString(arr));

//输出：[There, is, a, bear, in, the, woods]

arr = s.split(" ", 3);

System.out.println(Arrays.toString(arr));

//输出：[There, is, a bear in the woods]

```java
String s1 =  "There is a bear";
String s2 =  " in the woods";
String s = s1.concat(s2);
System.out.println(s);  //prints: There is a bear in the woods

```java

String s1 =  "There is a bear";

String s2 =  " in the woods";

String s = s1.concat(s2);

System.out.println(s);  //输出：There is a bear in the woods

```java
String s =  s1 + s2;
System.out.println(s);  //prints: There is a bear in the woods
```java

String s =  s1 + s2;

System.out.println(s);  //输出：There is a bear in the woods

```java
s = String.join(" ", "There", "is", "a", "bear", "in", "the", "woods");
System.out.println(s);  //prints: There is a bear in the woods

List<String> list = 
             List.of("There", "is", "a", "bear", "in", "the", "woods");
s = String.join(" ", list);
System.out.println(s);  //prints: There is a bear in the woods
```java

s = String.join(" ", "There", "is", "a", "bear", "in", "the", "woods");

System.out.println(s); //输出：There is a bear in the woods

List<String> list =

List.of("There", "is", "a", "bear", "in", "the", "woods");

s = String.join(" ", list);

System.out.println(s); //输出：There is a bear in the woods

```java
boolean b = "Introduction".startsWith("Intro");
System.out.println(b);             //prints: true

b = "Introduction".startsWith("tro", 2);
System.out.println(b);             //prints: true

b = "Introduction".endsWith("ion");
System.out.println(b);             //prints: true

```java

boolean b = "Introduction".startsWith("Intro");

System.out.println(b); //输出：true

b = "Introduction".startsWith("tro", 2);

System.out.println(b); //输出：true

b = "Introduction".endsWith("ion");

System.out.println(b); //输出：true

```java
System.out.println(LocalDate.now());   //prints: 2018-04-14
```java

System.out.println(LocalDate.now());   //输出：2018-04-14

```java
Set<String> zoneIds = ZoneId.getAvailableZoneIds();
for(String zoneId: zoneIds){
    System.out.println(zoneId);     
}
```java

Set<String> zoneIds = ZoneId.getAvailableZoneIds();

for(String zoneId: zoneIds){

System.out.println(zoneId);

}

```java
ZoneId zoneId = ZoneId.of("Asia/Tokyo");
System.out.println(LocalDate.now(zoneId));   //prints: 2018-04-15

```java

ZoneId zoneId = ZoneId.of("Asia/Tokyo");

System.out.println(LocalDate.now(zoneId));   //输出：2018-04-15

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

```java
System.out.println(lc5.getYear());          //prints: 2020
System.out.println(lc5.getMonth());         //prints: FEBRUARY
System.out.println(lc5.getMonthValue());    //prints: 2
System.out.println(lc5.getDayOfMonth());    //prints: 23

System.out.println(lc5.getDayOfWeek());     //prints: SUNDAY
System.out.println(lc5.isLeapYear());       //prints: true
System.out.println(lc5.lengthOfMonth());    //prints: 29
System.out.println(lc5.lengthOfYear());     //prints: 366

```java

System.out.println(lc5.getYear());          //prints: 2020

System.out.println(lc5.getMonth());         //prints: FEBRUARY

System.out.println(lc5.getMonthValue());    //prints: 2

System.out.println(lc5.getDayOfMonth());    //prints: 23

System.out.println(lc5.getDayOfWeek());     //prints: SUNDAY

System.out.println(lc5.isLeapYear());       //prints: true

System.out.println(lc5.lengthOfMonth());    //prints: 29

System.out.println(lc5.lengthOfYear());     //prints: 366

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

```java
LocalDate lc6 =  LocalDate.parse("2020-02-22");
LocalDate lc7 =  LocalDate.parse("2020-02-23");
System.out.println(lc6.isAfter(lc7));       //prints: false
System.out.println(lc6.isBefore(lc7));      //prints: true

```java

LocalDate lc6 =  LocalDate.parse("2020-02-22");

LocalDate lc7 =  LocalDate.parse("2020-02-23");

System.out.println(lc6.isAfter(lc7));       //prints: false

System.out.println(lc6.isBefore(lc7));      //prints: true

```java
System.out.println(LocalTime.now());         //prints: 21:15:46.360904

ZoneId zoneId = ZoneId.of("Asia/Tokyo");
System.out.println(LocalTime.now(zoneId));   //prints: 12:15:46.364378

LocalTime lt1 =  LocalTime.parse("20:23:12");
System.out.println(lt1);                     //prints: 20:23:12

LocalTime lt2 =  LocalTime.of(20, 23, 12);
System.out.println(lt2);                     //prints: 20:23:12

```java

System.out.println(LocalTime.now());         //prints: 21:15:46.360904

ZoneId zoneId = ZoneId.of("Asia/Tokyo");

System.out.println(LocalTime.now(zoneId));   //prints: 12:15:46.364378

LocalTime lt1 =  LocalTime.parse("20:23:12");

System.out.println(lt1);                     //prints: 20:23:12

LocalTime lt2 =  LocalTime.of(20, 23, 12);

System.out.println(lt2);                     //prints: 20:23:12

```java
System.out.println(lt2.getHour());          //prints: 20
System.out.println(lt2.getMinute());        //prints: 23
System.out.println(lt2.getSecond());        //prints: 12
System.out.println(lt2.getNano());          //prints: 0

```java

System.out.println(lt2.getHour());          //prints: 20

System.out.println(lt2.getMinute());        //prints: 23

System.out.println(lt2.getSecond());        //prints: 12

System.out.println(lt2.getNano());          //prints: 0

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

```java
LocalTime lt3 =  LocalTime.parse("20:23:12");
LocalTime lt4 =  LocalTime.parse("20:25:12");
System.out.println(lt3.isAfter(lt4));       //prints: false
System.out.println(lt3.isBefore(lt4));      //prints: true

```java

LocalTime lt3 =  LocalTime.parse("20:23:12");

LocalTime lt4 =  LocalTime.parse("20:25:12");

System.out.println(lt3.isAfter(lt4));       //prints: false

System.out.println(lt3.isBefore(lt4));      //prints: true

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

```java
for(int i =0; i < 3; i++){
    System.out.println(Math.random());
    //0.9350483840148613
    //0.0477353019234189
    //0.25784245516898985
}
```java

for(int i =0; i < 3; i++){

System.out.println(Math.random());

//0.9350483840148613

//0.0477353019234189

//0.25784245516898985

}

```java
int getInteger(int max){
    return (int)(Math.random() * max);
}
```java

int getInteger(int max){

return (int)(Math.random() * max);

}

```java
for(int i =0; i < 3; i++){
    System.out.print(getInteger(10) + " "); //prints: 2 5 6
}
```java

for(int i =0; i < 3; i++){

System.out.print(getInteger(10) + " "); //prints: 2 5 6

}

```java
for(int i =0; i < 3; i++){
    System.out.print(getInteger(100) + " "); //prints: 48 11 97
}
```java

for(int i =0; i < 3; i++){

System.out.print(getInteger(100) + " "); //prints: 48 11 97

}

```java
for(int i =0; i < 3; i++){
    System.out.print(100 + getInteger(100) + " "); //prints: 114 101 127
}
```java

for(int i =0; i < 3; i++){

System.out.print(100 + getInteger(100) + " "); //prints: 114 101 127

}

```java
int getIntegerRound(int max){
    return (int)Math.round(Math.random() * max);
}
```java

int getIntegerRound(int max){

return (int)Math.round(Math.random() * max);

}

```java
for(int i =0; i < 3; i++){
    System.out.print(100 + getIntegerRound(100) + " "); //179 147 200
}
```java

for(int i =0; i < 3; i++){

System.out.print(100 + getIntegerRound(100) + " "); //179 147 200

}

```java
int getInteger2(int max){
    return (int)(Math.random() * (max + 1));
}
```java

int getInteger2(int max){

return (int)(Math.random() * (max + 1));

}

```java
for(int i =0; i < 3; i++){
    System.out.print(100 + getInteger2(100) + " "); //167 200 132
}
```java

for(int i =0; i < 3; i++){

System.out.print(100 + getInteger2(100) + " "); //167 200 132

}

```java
Random random = new Random();
for(int i =0; i < 3; i++){
    System.out.print(random.nextDouble() + " "); 
    //prints: 0.8774928230544553 0.7822070124559267 0.09401796000707807 
}
```java

Random random = new Random();

for(int i =0; i < 3; i++){

System.out.print(random.nextDouble() + " ");

//prints: 0.8774928230544553 0.7822070124559267 0.09401796000707807

}

```java
for(int i =0; i < 3; i++){
    System.out.print(random.nextInt() + " "); 
                        //prints: -2001537190 -1148252160 1999653777
}
```java

for(int i =0; i < 3; i++){

System.out.print(random.nextInt() + " ");

//prints: -2001537190 -1148252160 1999653777

}

```java
for(int i =0; i < 3; i++){
    System.out.print(random.nextInt(11) + " "); //prints: 4 6 2
}
```java

for(int i =0; i < 3; i++){

System.out.print(random.nextInt(11) + " "); //prints: 4 6 2

}

```java
for(int i =0; i < 3; i++){
    System.out.print(11 + random.nextInt(10) + " "); //prints: 13 20 15
}
```java

for(int i =0; i < 3; i++){

System.out.print(11 + random.nextInt(10) + " "); //prints: 13 20 15

}

```java
String result = random.ints(3, 0, 101)
        .mapToObj(String::valueOf)
        .collect(Collectors.joining(" ")); //prints: 30 48 52

```java

String result = random.ints(3, 0, 101)

.mapToObj(String::valueOf)

.collect(Collectors.joining(" ")); //prints: 30 48 52

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
