# 第五章：循环结构

通常希望一遍又一遍地重复一系列操作。例如，我们可能希望显示存储在数组中的组织中员工的信息。数组的每个元素可能包含对`Employee`对象的引用。对象的方法调用将放置在循环结构内部。

在 Java 中有四种循环结构可用：

+   for 语句

+   For-each 语句

+   while 语句

d. while 语句

此外，break 和 continue 语句用于控制循环的行为。break 语句用于提前退出或短路循环，并在*break 语句*部分讨论。正如我们在第三章中观察到的，在*决策结构*中，break 也在 switch 语句中使用。continue 语句用于绕过循环中的语句并继续执行循环。它在*continue 语句*部分中介绍。我们还将研究在 Java 中使用标签，尽管它们应该谨慎使用。

循环的循环体根据循环结构的特定次数进行迭代。迭代是通常用来描述这种执行的术语。

循环使用控制信息来确定循环体将被执行多少次。对于大多数循环，有一组初始值，一组在循环体结束时执行的操作，以及一个终止条件，它将停止循环的执行。并非所有循环都有这些部分，因为其中一些部分要么缺失，要么隐含。终止条件几乎总是存在的，因为这是终止循环迭代所需的。如果终止条件缺失，则会创建一个无限循环。

无限循环指的是那些可能永远不会终止的循环，而不使用 break 语句等语句。尽管它们的名称是无限循环，但它们并不会无限执行，因为它们总会在某个时刻终止。它们在某些情况下很有用，这些情况中提供循环终止条件是不方便或尴尬的。

我们还将介绍嵌套循环的使用以及与循环相关的各种陷阱。还提供了一个处理编程逻辑开发的部分，以帮助在创建程序逻辑时提供一种方法。

# for 语句

当需要知道循环需要执行的次数时，使用 for 语句。for 循环有两种变体。第一种在本节中讨论，是传统形式。for-each 语句是第二种形式，引入于 Java 5。它在*for-each 语句*部分中讨论。

for 语句由以下三部分组成：

+   初始操作

+   终止条件

+   结束循环操作

for 循环的一般形式如下：

```java
for (<initial-expression>;<terminal-expression>;<end-loopoperation>)
  //statements;
```

for 循环的循环体通常是一个块语句。初始操作在循环的第一次迭代之前进行，只执行一次。结束循环操作在每次循环执行结束时进行。终止条件确定循环何时终止，并且是一个逻辑表达式。它在每次循环重复开始时执行。因此，如果第一次评估终止条件时，它的值为 false，则 for 循环的循环体可能会执行零次。

通常作为初始操作、终止条件和结束循环操作的一部分使用变量。变量要么作为循环的一部分声明，要么作为循环外部声明。以下代码片段是声明变量`i`作为循环一部分的示例。使用外部变量的示例在*for 语句和作用域*部分中介绍：

```java
for (int i = 1; i <= 10; i++) {
   System.out.print(i + "  ");
}
System.out.println();
```

在这个例子中，我们在循环体中使用了一个语句。变量`i`被赋予初始值 1，并且每次循环执行时增加 1。循环执行了 10 次，并产生了 1 行输出。语句`i++`是一种更简洁的方式，表示`i = i + 1`。输出应该是以下内容：

```java
1  2  3  4  5  6  7  8  9  10 

```

以下示例使用 for 语句计算从`1`到`64`的整数的平方：

```java
for (int i = 1; i <= 64; i++) {
  System.out.println (i + " squared is = " + i * i);
  }

```

输出的部分列表如下：

**1 的平方是= 1**

**2 的平方是= 4**

**3 的平方是= 9**

**4 的平方是= 16**

**...**

循环变量的初始值可以是任何值。此外，结束循环操作可以根据需要递减或修改变量。在下一个例子中，从`10`到`1`显示数字：

```java
for (int i = 10; i > 0; i--) {
   System.out.print(i + "  ");
}
System.out.println();
```

以下是此序列的输出：

```java
10  9  8  7  6  5  4  3  2  1 

```

一个常见的操作是计算累积和，如下面的代码序列所示。这个例子在*时间就是一切*部分中有更详细的讨论：

```java
int sum = 0;
for(i = 1; i <= 10; i++) {
  sum += i;
}
System.out.println(sum);
```

`sum`的值应该是`55`。

## 逗号运算符

逗号运算符可以作为 for 语句的一部分，用于添加其他变量以在循环内使用和/或控制循环。它用于分隔 for 循环的初始表达式和结束循环操作部分的各个部分。逗号运算符的使用如下所示：

```java
for(int i = 0, j = 10; j > 5; i++, j--) {
   System.out.printf("%3d  %3d%n",i , j);
}
```

注意在`printf`语句中使用了`%n`格式说明符。这指定应生成一个新行字符。此外，这个新行分隔符是特定于平台的，使应用程序更具可移植性。

执行此代码序列时，将产生以下输出：

```java
0   10
1    9
2    8
3    7
4    6

```

循环声明了两个变量，`i`和`j`。变量`i`初始化为`0`，`j`初始化为`10`。循环结束时，`i`增加了`1`，`j`减少了`1`。只要`j`大于`5`，循环就会执行。

我们可以使用更复杂的终止条件，如下面的代码片段所示：

```java
for(int i = 0, j = 10; j > 5 && i < 3; i++, j--) {
   System.out.printf("%3d  %3d%n",i , j);
}
```

在这个例子中，循环将在第三次迭代后终止，产生以下输出：

```java
  0   10
  1    9
  2    8
```

在这里分别声明变量是非法的：

```java
for(int i = 0, int j = 10; j > 5; i++, j--) {
```

生成了一个语法错误，如下所示。由于消息过长，只提供了消息的第一部分。这也说明了 Java 和大多数其他编程语言生成的错误消息的神秘性质：

```java
<identifier> expected
'.class' expected
...

```

## for 语句和作用域

for 语句使用的索引变量可以根据其声明方式具有不同的作用域。我们可以利用这一点来控制循环的执行，然后根据需要在循环外部使用变量。第一个 for 循环的示例如下重复。在这个代码序列中，`i`变量的作用域仅限于 for 循环的主体：

```java
for (int i = 1; i <= 10; i++) {
   System.out.println(i);
}
System.out.println();
```

一种替代方法是将`i`声明为循环外部，如下所示：

```java
int i;
for (i = 1; i <= 10; i++) {
  System.out.print(i + " ");
}
System.out.println();
```

这两个 for 循环是等价的，因为它们都在一行上显示 1 到 10 的数字。它们在`i`变量的作用域上有所不同。在第一个例子中，作用域仅限于循环体。尝试在循环外部使用变量，如下面的代码所示，将导致语法错误：

```java
for (int i = 1; i <= 10; i++) {
   System.out.println(i);
}
System.out.println(i);
```

错误消息如下：

```java
cannot find symbol
 symbol:   variable i

```

在第二个例子中，循环终止后，变量将保留其值，并可供后续使用。以下示例说明了这一点：

```java
int i;
for (i = 1; i <= 10; i++) {
  System.out.print(i + "  ");
}
System.out.println();
System.out.println(i);
```

以下是此序列的输出：

```java
1  2  3  4  5  6  7  8  9  10 
11

```

作用域在第二章的*作用域和生存期*部分中有更详细的讨论，*Java 数据类型及其用法*。

## for 循环的变体

for 循环的主体可能由多个语句组成。重要的是要记住 for 循环的主体只包含一个语句。下面说明了循环中多个语句的使用。这个循环将读取一系列数字并将它们逐行打印出来。它将继续读取，直到读取到一个负值，然后退出循环。`java.util.Scanner`类用于从输入源中读取数据。在这种情况下，它使用`System.in`指定键盘作为输入源：

```java
Scanner scanner = new Scanner(System.in);
int number = 0;

for (int i = 0; number >= 0; i++) {
   System.out.print("Enter a number: ");
   number = scanner.nextInt();
   System.out.printf("%d%n", number);
}
```

执行这段代码序列的一个可能的输出如下：

```java
Enter a number: 3
3
Enter a number: 56
56
Enter a number: -5
-5

```

初始操作、终止条件或结束循环操作是不需要的。例如，以下语句将执行`i++`语句 5 次，退出循环时`i`的值为`5`：

```java
int i = 0;
for (;i<5;) {
   i++;
}
```

在下面的例子中，循环的主体将永远执行，创建一个无限循环：

```java
int i = 0;
for (;;i++)
   ;
```

对于以下的 for 循环也是一样的：

```java
int i = 0;
for(;;) 
   ;
```

这被称为**无限循环**，在*无限循环*部分中有更详细的介绍。

### 注意

当你知道循环将被执行多少次时，通常使用 for 循环。通常使用一个控制整数变量作为数组的索引或在循环体内进行计算。

# for-each 语句

for-each 语句是在 Java 5 发布时引入的。有时它被称为增强型 for 循环。使用 for-each 语句的优点包括：

+   不需要为计数变量提供结束条件

+   它更简单，更易读

+   该语句提供了编译器优化的机会

+   泛型的使用变得更简单

for-each 语句与集合和数组一起使用。它提供了一种更容易遍历数组或实现了`java.util.Iterable`接口的类的每个成员的方法。由于`Iterable`接口是`java.util.Collection`接口的超级接口，因此 for-each 语句可以与实现`Collection`接口的类一起使用。

这个语句的语法与常规的 for 语句类似，除了括号内的内容。括号内包括数据类型，后跟一个变量，一个冒号，然后是数组名或集合，如下所示：

```java
for (<dataType variable>:<collection/array>)
   //statements;
```

它与集合的使用在*使用 for-each 语句与列表*部分进行了说明。在下面的序列中，声明了一个整数数组，初始化了它，并使用 for-each 语句显示了数组的每个元素：

```java
int numbers[] = new int[10];

for (int i = 0; i < 10; i++) {
   numbers[i] = i;
}

for (int element : numbers) {
   System.out.print(element + "  ");
}    
System.out.println();
```

numbers 数组的元素被初始化为它们的索引。请注意使用了 for 语句。这是因为我们无法在 for-each 语句中直接访问索引变量。在上述代码片段中，for-each 语句被读作“对于 numbers 中的每个元素”。在循环的每次迭代中，`element`对应数组的一个元素。它从第一个元素开始，直到最后一个元素结束。这个序列的输出如下：

```java
0 1 2 3 4 5 6 7 8 9

```

使用 for-each 语句与数组存在一些缺点。无法执行以下操作：

+   修改数组或列表中的当前位置

+   直接迭代多个数组或集合

例如，使用前面的例子，如果我们尝试使用以下代码修改包含 5 的数组元素，它不会导致语法错误。但它也不会修改相应的数组元素：

```java
for (int element : numbers) {
   if (element == 5) {
      element = -5;
   }
}

for (int element : numbers) {
   System.out.print(element + "  ");
}
System.out.println();
```

这个序列的输出如下：

```java
0 1 2 3 4 5 6 7 8 9

```

如果我们想要使用一个循环来访问两个不同的数组，就不能使用 for-each 循环。例如，如果我们想要将一个数组复制到另一个数组，我们需要使用 for 循环，如下所示：

```java
int source[] = new int[5];
int destination[] = new int[5];

for(int number : source) {
   number = 100;
}

for(int i = 0; i < 5; i++) {
   destination[i] = source[i];
}
```

虽然我们使用 for-each 来初始化源数组，但我们一次只能处理一个数组。因此，在第二个循环中，我们被迫使用 for 语句。

## 使用 for-each 语句与列表

我们将首先说明如何使用 for-each 语句与`ArrayList`。`ArrayList`类实现了`List`接口，该接口扩展了`Collection`接口。接口的使用和声明在第六章中有更详细的介绍，*类，构造函数和方法*。由于 for-each 语句可以与实现`Collection`接口的类一起使用，我们也可以将其与`ArrayList`类一起使用。在下一节中，我们将创建自己的`Iterable`类：

```java
ArrayList<String> list = new ArrayList<String>();

list.add("Lions and");
list.add("tigers and");
list.add("bears.");
list.add("Oh My!");

for(String word : list) {
   System.out.print(word + "  ");
}
System.out.println();
```

输出，正如你可能预测的那样，如下所示：

```java
Lions and tigers and bears. Oh My!

```

在这个例子中，使用 for-each 与数组的使用并没有太大的不同。我们只是使用了`ArrayList`的名称而不是数组名称。

使用 for-each 语句与列表具有与我们之前在数组中看到的类似的限制：

+   可能无法在遍历列表时删除元素

+   无法修改列表中的当前位置

+   不可能同时迭代多个集合

`remove`方法可能会抛出`UnsupportedOperationException`异常。这是可能的，因为`Iteratable`接口的`Iterator`的实现可能没有实现`remove`方法。这将在下一节详细说明。

在`ArrayList`的情况下，我们可以移除一个元素，如下面的代码片段所示：

```java
for(String word : list) {
   if(word.equals("bears.")) {
      list.remove(word);
      System.out.println(word + " removed");
   }
}

for(String word : list) {
   System.out.print(word + "  ");
}
System.out.println();
```

for-each 语句用于遍历列表。当找到`bears.`字符串时，它被移除。前面序列的输出如下：

```java
Lions and tigers and bears. Oh My! 
bears. removed
Lions and tigers and Oh My!

```

我们不能在 for-each 语句内修改列表。例如，以下代码序列尝试修改`word`并向`list`添加一个字符串。列表不会受到影响：

```java
for(String word : list) {
   if(word.equals("bears.")) {
      word = "kitty cats";
      list.add("kitty cats");
   }
}
```

尝试修改`word`变量并不会产生任何效果，也不会生成异常。但`add`方法不是这样。在前面的 for-each 语句中使用它将生成一个`java.util.ConcurrentModificationException`异常。

### 注意

与数组一样，使用 for-each 语句一次只能迭代一个集合。由于 for-each 语句只支持一个引用变量，因此一次只能访问一个列表。

如果您需要从列表中删除一个元素，请使用迭代器而不是 for-each 语句。

## 实现迭代器接口

如前所述，任何实现`Iterable`接口的类都可以与 for-each 语句一起使用。为了说明这一点，我们将创建两个类：

+   `MyIterator`：这实现了`Iterator`接口并支持一个简单的迭代

+   `MyIterable`：这使用`MyIterator`来支持它在 for-each 语句中的使用

首先，让我们来看一下接下来的`MyIterator`类。该类将迭代 1 到 10 的数字。它通过将`value`变量与 10 的上限进行比较，并在其`hasNext`方法中返回`true`或`false`来实现这一点。`next`方法只是返回并增加当前值。`remove`方法不受支持：

```java
import java.util.Iterator;

public class MyIterator implements Iterator<Integer> {
   private int value;
   private final int size;

   public MyIterator() {
      value = 1;
      size = 10;
   }

   @Override
   public boolean hasNext() {
      return value<=size;
   }

   @Override
   public Integer next() {
      return value++;
   }

   @Override
   public void remove() {
      throw new UnsupportedOperationException(
            "Not supported yet.");
   }
}
```

`MyIterable`类实现了`Iterable`接口。该接口包含一个方法`iterator`。在这个类中，它使用`MyIterator`类的一个实例来提供一个`Iterator`对象：

```java
import java.util.Iterator;

public class MyIterable implements Iterable<Integer> {
   private MyIterator iterator;

   public MyIterable() {
      iterator = new MyIterator();
   }

   @Override
   public Iterator<Integer> iterator() {
      return iterator;
   }
}
```

我们可以使用以下代码序列测试这些类：

```java
MyIterable iterable = new MyIterable();

for(Integer number : iterable) {
   System.out.print(number + "  ");
}
System.out.println();
```

输出将显示数字从 1 到 10，如下所示：

```java
1 2 3 4 5 6 7 8 9 10

```

### 注意

在许多情况下，并不总是需要使用`Iterator`方法来迭代集合。在许多情况下，for-each 语句提供了一种更方便和简单的技术。

## for-each 语句-使用问题

在使用 for-each 语句时，有几个问题需要注意：

+   如果数组/集合为空，您将会得到一个空指针异常

+   它可以很好地与具有可变数量参数的方法一起使用

### 空值

如果数组/集合为 null，您将获得一个空指针异常。考虑以下示例。我们创建了一个字符串数组，但未初始化第三个元素：

```java
String names[] = new String[5];
names[0] = "Will Turner";
names[1] = "Captain Jack Sparrow";
names[3] = "Barbossa";
names[4] = "Elizabeth Swann";
```

我们可以使用 for-each 语句显示名称，如下所示：

```java
for(String name : names) {
   System.out.println(name);
}
```

输出如下所示，将为缺失的条目显示`null`。这是因为`println`方法检查其参数是否为 null 值，当为 null 时，它会打印`null`：

```java
Will Turner
Captain Jack Sparrow
null
Barbossa
Elizabeth Swann

```

但是，如果我们对名称应用`toString`方法，我们将在第三个元素上得到`java.lang.NullPointerException`：

```java
for(String name : names) {
   System.out.println(name.toString());
}
```

如下输出所示，这是经过验证的：

```java
Will Turner
Captain Jack Sparrow
java.lang.NullPointerException

```

### 可变数量的参数

for-each 语句在使用可变数量的参数的方法中效果很好。关于使用可变数量的参数的方法的更详细解释可以在第六章的*可变数量的参数*部分找到。

在以下方法中，我们传递了可变数量的整数参数。接下来，我们计算这些整数的累积和并返回总和：

```java
public int total(int ... array) {
   int sum = 0;
   for(int number : array) {
      sum+=number;
   }
   return sum;
}
```

当使用以下调用执行时，我们得到`15`和`0`作为输出：

```java
result = total(1,2,3,4,5);
result = total();
```

但是，我们需要小心，不要传递`null`值，因为这将导致`java.lang.NullPointerException`，如下面的代码片段所示：

```java
result = total(null);
```

### 注意

尽可能使用 for-each 循环，而不是 for 循环。

# while 语句

while 语句提供了另一种重复执行一组语句的方法。当要执行块的次数未知时，它经常被使用。它的一般形式包括`while`关键字后跟一组括号括起来的逻辑表达式，然后是一个语句。只要逻辑表达式评估为 true，循环的主体将执行：

```java
while (<boolean-expression>) <statements>;
```

一个简单的示例重复了第一个 for 循环示例，其中我们在一行上显示数字 1 到 10：

```java
int i = 1;
while(i <= 10) {
   System.out.print(i++ + "  ");
}
System.out.println();
```

输出如下：

```java
1 2 3 4 5 6 7 8 9 10

```

以下示例稍微复杂一些，计算了`number`变量的因子：

```java
int number;
int divisor = 1;
Scanner scanner = new Scanner(System.in);
System.out.print("Enter a number: ");
number = scanner.nextInt();
while (number >= divisor) {
   if ((number % divisor) == 0) {
      System.out.printf("%d%n", divisor);
   }
   divisor++;
}
```

当使用输入`6`执行时，我们得到以下输出：

```java
Enter a number: 6
1
2
3
6

```

以下表格说明了语句序列的操作：

| 迭代次数 | 除数 | 数字 | 输出 |
| --- | --- | --- | --- |
| 1 | 1 | 6 | 1 |
| 2 | 2 | 6 | 2 |
| 3 | 3 | 6 | 3 |
| 4 | 4 | 6 |   |
| 5 | 5 | 6 |   |
| 6 | 6 | 6 | 6 |

在以下示例中，当用户输入负数时，循环将终止。在此过程中，它计算了输入数字的累积和：

```java
int number;
System.out.print("Enter a number: ");
number = scanner.nextInt();
while (number > 0) {
   sum += number;
   System.out.print("Enter a number: ");
   number = scanner.nextInt();
}
System.out.println("The sum is " + sum);
```

请注意，此示例重复了提示用户输入数字所需的代码。可以更优雅地使用 do-while 语句来处理这个问题，如下一节所讨论的。以下输出说明了对一系列数字执行此代码的执行情况：

```java
Enter a number: 8
Enter a number: 12
Enter a number: 4
Enter a number: -5
The sum is 24

```

while 语句对于需要未知循环次数的循环很有用。while 循环的主体将执行，直到循环表达式变为 false。当终止条件相当复杂时，它也很有用。

### 注意

while 语句的一个重要特点是在循环开始时对表达式进行评估。因此，如果逻辑表达式的第一次评估为 false，则循环的主体可能永远不会执行。

# do-while 语句

do-while 语句类似于 while 循环，不同之处在于循环的主体始终至少执行一次。它由`do`关键字后跟一个语句，`while`关键字，然后是括号括起来的逻辑表达式组成：

```java
do <statement> while (<boolean-expression>);
```

通常，do-while 循环的主体，如语句所表示的那样，是一个块语句。以下代码片段说明了 do 语句的用法。它是对前一节中使用的等效 while 循环的改进，因为它避免了在循环开始之前提示输入一个数字：

```java
int sum = 0;
int number;
Scanner scanner = new Scanner(System.in);
do {
   System.out.print("Enter a number: ");
   number = scanner.nextInt();
   if(number > 0 ) {
     sum += number;
   }
} while (number > 0);
System.out.println("The sum is " + sum);
```

当执行时，您应该获得类似以下的输出：

```java
Enter a number: 8
Enter a number: 12
Enter a number: 4
Enter a number: -5
The sum is 24

```

### 注意

`do-while`语句与`while`语句不同，因为表达式的评估发生在循环结束时。这意味着该语句至少会执行一次。

这个语句并不像`for`或`while`语句那样经常使用，但在循环底部进行测试时很有用。下一个语句序列将确定整数数字中的数字位数：

```java
int numOfDigits;
System.out.print("Enter a number: ");
Scanner scanner = new Scanner(System.in);
int number = scanner.nextInt();
numOfDigits = 0;
do {
   number /= 10;
   numOfDigits++;
} while (number != 0);
System.out.printf("Number of digits: %d%n", numOfDigits);
```

这个序列的输出如下：

```java
Enter a number: 452
Number of digits: 3

```

值`452`的结果如下表所示：

| 迭代次数 | 数字 | 数字位数 |
| --- | --- | --- |
| 0 | 452 | 0 |
| 1 | 45 | 1 |
| 2 | 4 | 2 |
| 3 | 0 | 3 |

# `break`语句

`break`语句的效果是终止当前循环，无论是`while`、`for`、`for-each`还是`do-while`语句。它也用于`switch`语句。`break`语句将控制传递给循环后面的下一个语句。`break`语句由关键字`break`组成。

考虑以下语句序列的影响，该序列会在无限循环中重复提示用户输入命令。当用户输入`Quit`命令时，循环将终止：

```java
String command;
while (true) {
   System.out.print("Enter a command: ");
   Scanner scanner = new Scanner(System.in);
   command = scanner.next();
   if ("Add".equals(command)) {
      // Process Add command
   } else if ("Subtract".equals(command)) {
      // Process Subtract command
   } else if ("Quit".equals(command)) {
      break;
   } else {
      System.out.println("Invalid Command");
   }
}

```

注意`equals`方法的使用方式。`equals`方法针对字符串字面量执行，并将命令用作其参数。这种方法避免了`NullPointerException`，如果命令包含空值，将会导致该异常。由于字符串字面量永远不会为空，这种异常永远不会发生。

# `continue`语句

`continue`语句用于将控制从循环内部转移到循环的末尾，但不像`break`语句那样退出循环。`continue`语句由关键字`continue`组成。

执行时，它会强制执行循环的逻辑表达式。在以下语句序列中：

```java
while (i < j) {
   …
   if (i < 0) {
      continue;
   }
   …
}
```

如果`i`小于`0`，它将绕过循环体的其余部分。如果循环条件`i<j`不为假，将执行循环的下一次迭代。

`continue`语句通常用于消除通常是必要的嵌套级别。如果没有使用`continue`语句，前面的示例将如下所示：

```java
while (i < j) {
   …
   if (i < 0) {
      // Do nothing
   } else {
      …
   }
}
```

# 嵌套循环

循环可以嵌套在彼此之内。可以嵌套组合`for`、`for-each`、`while`或`do-while`循环。这对解决许多问题很有用。接下来的示例计算了二维数组中一行元素的总和。它首先将每个元素初始化为其索引的总和。然后显示数组。然后使用嵌套循环计算并显示每行元素的总和：

```java
final int numberOfRows = 2;
final int numberOfColumns = 3;
int matrix[][] = new int[numberOfRows][numberOfColumns];

for (int i = 0; i < matrix.length; i++) {
   for (int j = 0; j < matrix[i].length; j++) {
      matrix[i][j] = i + j;
   }
}

for (int i = 0; i < matrix.length; i++) {
   for(int element : matrix[i]) {
      System.out.print(element + "  ");
   }
   System.out.println();
}  

for (int i = 0; i < matrix.length; i++) {
   int sum = 0;
   for(int element : matrix[i]) {
      sum += element;
   }
   System.out.println("Sum of row " + i + " is " +sum);
}
```

注意使用`length`方法来控制循环执行的次数。如果数组的大小发生变化，这将使代码更易于维护。执行时，我们得到以下输出：

```java
0 1 2 
1 2 3 
Sum of row 0 is 3
Sum of row 1 is 6

```

注意在显示数组和计算行的总和时使用`for-each`语句。这简化了计算。

`break`和`continue`语句也可以在嵌套循环中使用。但是，它们只能与当前循环一起使用。也就是说，从内部循环中跳出只会跳出内部循环，而不是外部循环。正如我们将在下一节中看到的，我们可以使用标签从内部循环中跳出外部循环。

在最后一个嵌套循环序列的修改中，当总和超过 2 时，我们跳出内部循环：

```java
for (int i = 0; i < matrix.length; i++) {
   int sum = 0;
   for(int element : matrix[i]) {
      sum += element;
      if(sum > 2) {
         break;
      }
   }
   System.out.println("Sum of row " + i + " is " +sum);
}
```

这个嵌套循环的执行将改变最后一行的总和，如下所示：

```java
Sum of row 0 is 3
Sum of row 1 is 3

```

`break`语句使我们跳出内部循环，但没有跳出外部循环。如果在外部循环的立即体中有相应的`break`语句，我们可以跳出外部循环。`continue`语句在内部和外部循环方面的行为类似。

# 使用标签

标签是程序内的位置名称。它们可以用于改变控制流，并且应该谨慎使用。在前面的例子中，我们无法使用 break 语句跳出最内层的循环。但是，标签可以用于跳出多个循环。

在下面的例子中，我们在外部循环前放置一个标签。在内部循环中，当`i`大于 0 时执行 break 语句，有效地在第一行的求和计算完成后终止外部循环。标签由名称后跟一个冒号组成：

```java
outerLoop:
for(int i = 0; i < 2; i++) {
   int sum = 0;
   for(int element : matrix[i]) {
      sum += element;
      if(i > 0) {
         break outerLoop;
      }
   }
   System.out.println("Sum of row " + i + " is " +sum);
}
```

这个序列的输出如下：

```java
Sum of row 0 is 3

```

我们也可以使用带有标签的 continue 语句来达到类似的效果。

### 注意

标签应该避免，因为它们会导致代码难以阅读和维护。

# 无限循环

无限循环是一种除非使用 break 语句等语句强制终止，否则将永远执行的循环。无限循环非常有用，可以避免循环的尴尬逻辑条件。

一个无限 while 循环应该使用`true`关键字作为其逻辑表达式：

```java
while (true) {
   // body
}
```

一个 for 循环可以简单到使用 nulls 作为 for 语句的每个部分：

```java
for (;;) {
   // body
}
```

一个永远不会终止的循环通常对大多数程序没有价值，因为大多数程序最终应该终止。然而，大多数无限循环都设计为使用 break 语句终止，如下所示：

```java
while (true) {
   // first part
   if(someCondition) {
      break;
   }
   // last part
}
```

这种技术相当常见，用于简化程序的逻辑。考虑需要读取年龄并在年龄为负时终止的需要。需要为年龄分配一个非负值，以确保循环至少执行一次：

```java
int age;
age = 1;
Scanner scanner = new Scanner(System.in);
while (age > 0) {
   System.out.print("Enter an age: ");
   age = scanner.nextInt();
   // use the age
}
```

另一个选择是在循环开始之前复制用户提示和用于读取年龄的语句：

```java
System.out.print("Enter an age: ");
age = scanner.nextInt();
while (age > 0) {
   System.out.print("Enter an age: ");
   age = scanner.nextInt();
   // use the age
}
```

在循环开始之前，要么需要为年龄分配一个任意值，要么需要复制代码。这两种方法都不令人满意。

然而，使用无限循环会导致更清晰的代码。不需要分配任意值，也不需要复制代码：

```java
while (true) {
   System.out.print("Enter an age: ");
   age = scanner.nextInt();
   if (age < 0) {
      break;
   }
   // use the age
}
```

虽然有许多情况下无限循环是可取的，但程序员在不小心的情况下也可能出现无限循环，导致意想不到的结果。一个常见的方式是构建一个没有有效终止条件的 for 循环，如下所示：

```java
for(int i = 1; i > 0; i++) {
   // Body
}
```

循环从`i`的值为 1 开始，并且在每次循环迭代时递增`i`。终止条件表明循环不会终止，因为`i`只会变得更大，因此总是大于 0。然而，最终变量会溢出，`i`会变成负数，循环将终止。这需要多长时间取决于机器的执行速度。

故事的寓意是，“小心循环”。无限循环既可以是解决某些问题的有用构造，也可以是在无意中使用时出现问题的构造。

# 时间就是一切

一个常见的编程需求是执行某种求和。我们在之前的例子中已经计算了几个数字序列的和。虽然求和过程相对简单，但对于新手程序员来说可能会很困难。更重要的是，我们将在这里使用它来提供对编程过程的洞察。

编程本质上是一个抽象的过程。程序员需要查看静态代码清单并推断其动态执行。这对很多人来说可能很困难。帮助开发人员编写代码的一种方法是考虑以下三个问题：

+   我们想要做什么？

+   我们想要怎么做？

+   我们想要做什么时候？

在这里，我们将询问并应用这三个问题的答案来解决求和问题。然而，这些问题同样适用于其他编程问题。

让我们专注于计算一组学生的平均年龄，这将涉及到求和过程。假设年龄存储在`age`数组中，然后按照以下代码片段中所示进行初始化：

```java
final int size = 5;
int age[] = new int[size];
int total;
float average;

age[0] = 23;
age[1] = 18;
age[2] = 19;
age[3] = 18;
age[4] = 21;
```

然后求和可以如下计算：

```java
total = 0;
for (int number : age) {
   total = total + number;
}
average = total / (age.length * 1.0f);
```

请注意，`total`明确地被赋予了零值。for 循环的每次迭代将把下一个`age`添加到`total`中。在循环完成时，`total`将被数组的长度乘以`1.0f`来计算平均值。通过使用数组长度，如果数组大小发生变化，代码表达式就不需要更改。乘以`1.0f`是为了避免整数除法。以下表格说明了循环执行时变量的值：

| 循环计数 | i | total |
| --- | --- | --- |
| 0 | - | 0 |
| 1 | 1 | 23 |
| 2 | 2 | 41 |
| 3 | 3 | 60 |
| 4 | 4 | 78 |
| 5 | 5 | 99 |

让我们从以下三个基本问题的角度来考虑这个问题：

+   **我们想要做什么**：我们想要计算一个部门的总工资。

+   **我们想要如何做**：这有一个多部分的答案。我们知道我们需要一个变量来保存总工资，`total`，并且它需要初始化为 0。

```java
total = 0;
```

我们还了解到计算累积和的基本操作如下：

```java
total = total + number;
```

循环需要使用数组的每个元素，因此使用 for-each 语句：

```java
for (int number : age) {
   …
}
```

我们已经为解决问题奠定了基础。

+   **我们想要什么时候做**：在这种情况下，“何时”暗示着三个基本选择：

+   在循环之前

+   在循环中

+   在循环之后

我们的解决方案的三个部分可以以不同的方式组合。基本操作需要在循环内部，因为它需要执行多次。只执行一次基本操作将不会得到我们喜欢的答案。

变量`total`需要初始化为 0。如何做到这一点？我们通过使用赋值语句来实现。这应该在循环之前、之中还是之后完成？在循环之后这样做是愚蠢的。当循环完成时，`total`应该包含答案，而不是零。如果我们在循环内部将其初始化为 0，那么在循环的每次迭代中，`total`都会被重置为 0。这让我们只能选择在循环之前放置该语句作为唯一有意义的选项。我们想要做的第一件事是将`total`赋值为 0。

大多数问题的解决方案似乎总是有所变化。例如，我们可以使用 while 循环而不是 for-each 循环。`+=`运算符可以用来简化基本操作。使用这些技术的一个潜在解决方案引入了一个索引变量：

```java
int index = 0;
total = 0;

while(index < age.length) {
   total += age[index++];
}
average = total / (age.length * 1.0f);
```

显然，并不总是有一个特定问题的最佳解决方案。这使得编程过程既是一种创造性的，也是一种潜在有趣的活动。

# 陷阱

与大多数编程结构一样，循环也有自己一套潜在的陷阱。在本节中，我们将讨论可能对不慎的开发人员造成问题的领域。

当程序员在每个语句后使用分号时，常见的问题之一是出现问题。例如，以下语句由于额外的分号导致无限循环：

```java
int i = 1;
while(i < 10) ;
  i++;
```

单独一行上的分号是空语句。这个语句什么也不做。然而，在这个例子中，它构成了 while 循环的主体部分。增量语句不是 while 循环的一部分。它是跟在 while 循环后面的第一个语句。缩进虽然是可取的，但并不会使语句成为循环的一部分。因此，`i`永远不会增加，逻辑控制表达式将始终返回 true。

在循环的主体部分没有使用块语句可能会导致问题。在下面的例子中，我们尝试计算从 1 到 5 的数字的乘积的总和。然而，这并不起作用，因为循环的主体部分只包括了乘积的计算。当求和语句缩进时，它不是循环的主体部分，只会执行一次：

```java
int sum = 0;
int product = 0;
for(int i = 1; i <= 5; i++)
   product = i * i;;
   sum += product;
```

循环的正确实现如下所示使用块语句：

```java
int sum = 0;
int product = 0;
for(int i = 1; i <= 5; i++) {
   product = i * i;;
   sum += product;
}
```

### 注意

在循环的主体中始终使用块语句是一个很好的策略，即使主体只包含一个语句。

在以下序列中，循环的主体由多个语句组成。然而，`i`从未递增。这也将导致无限循环，除非限制之一被更改或遇到 break 语句：

```java
int i = 0;
while(i<limit) {
  // Process i
}
```

即使是看似简单的循环，如果对浮点运算不小心，实际上可能是无限循环。在这个例子中，每次循环迭代时，`x`都会加上`0.1`。循环应该在`x`恰好等于`1.1`时停止。这永远不会发生，因为某些值的浮点数存储方式存在问题：

```java
float x = 0.1f;
while (x != 1.1) {
   System.out.printf("x = %f%n", x);
   x = x + 0.1f;
}
```

在二进制基数中，`0.1`不能被精确存储，就像分数 1/3 的十进制等价物无法被精确表示一样（0.333333…）。将这个数字重复添加到`x`的结果将导致一个不完全等于`1.1`的数字。比较`x != 1.1`将返回 true，循环永远不会结束。`printf`语句的输出不显示这种差异：

```java
…
x = 0.900000
x = 1.000000
x = 1.100000
x = 1.200000
x = 1.300000
…

```

在处理涉及自动装箱的操作时要小心。根据实现方式，如果频繁发生装箱和拆箱，可能会导致性能损失。

虽然不一定是陷阱，但要记住逻辑表达式可能会短路。也就是说，逻辑 AND 或 OR 操作的最后部分可能不会被评估，这取决于从第一部分评估返回的值。这在第三章的*短路评估*部分中有详细讨论，*决策结构*。

### 注意

请记住，数组、字符串和大多数集合都是从零开始的。忘记从`0`开始循环将忽略第一个元素。

始终使用块语句作为循环的主体。

# 总结

在本章中，我们研究了 Java 为循环提供的支持。我们已经说明了 for、for-each、while 和 do-while 语句的使用。这些演示提供了正确使用它们的见解，以及何时应该使用它们，何时不应该使用它们。

展示了使用 break 和 continue 语句，以及标签的使用。我们看到了 break 语句的实用性，特别是在支持无限循环时。标签虽然应该避免使用，但在跳出嵌套循环时可能会有用。

研究了各种陷阱，并研究了总结过程的创建，以深入了解一般编程问题。具体而言，它解决了代码段应放置在何处的问题。

现在我们已经了解了循环，我们准备研究类、方法和数据封装的创建，这是下一章的主题。

# 认证目标涵盖

在本章中，我们解决了以下认证目标：

+   创建和使用 while 循环

+   创建和使用 for 循环，包括增强的 for 循环

+   创建和使用 do/while 循环

+   比较循环结构

+   使用 break 和 continue

此外，我们还提供了对这些目标的额外覆盖：

+   定义变量的作用域

+   使用运算符和决策结构

+   声明和使用 ArrayList

# 测试你的知识

1.  给定以下声明，哪个语句将编译？

```java
int i = 5;
int j = 10;
```

a. `while(i < j) {}`

b. `while(i) {}`

c. `while(i = 5) {}`

d. `while((i = 12)!=5) {}`

1.  给定数组的以下声明，哪个语句将显示数组的每个元素？

```java
int arr[] = {1,2,3,4,5};
```

a. `for(int n : arr[]) { System.out.println(n); }`

b. `for(int n : arr) { System.out.println(n); }`

c. `for(int n=1; n < 6; n++) { System.out.println(arr[n]); }`

d. `for(int n=1; n <= 5; n++) { System.out.println(arr[n]); }`

1.  以下哪个 do/while 循环将在没有错误的情况下编译？

a.

```java
int i = 0;
do {
	System.out.println(i++);
} while (i < 5);

```

b.

```java
int i = 0;
do
	System.out.println(i++);
while (i < 5);

```

c.

```java
int i = 0;
do 
	System.out.println(i++);
while i < 5;

```

d.

```java
i = 0;
do
	System.out.println(i);
	i++;
while (i < 5);

```

1.  以下哪个循环是等价的？

a.

```java
for(String n : list) { 
	System.out.println(n);
}

```

b.

```java
for(int n = 0; n < list.size(); n++ ){ 
	System.out.println(list.get(n));
}

```

c.

```java
Iterator it = list.iterator();
while(it.hasNext()) {
	System.out.println(it.next());
}

```

1.  以下代码将输出什么？

```java
int i;
int j;
for (i=1; i < 4; i++) {
   for (j=2; j < 4; j++) {
      if (j == 3) {
         continue;
      }
      System.out.println("i: " + i + " j: " + j);
   }
}
```

a. `i: 1 j: 2i: 2 j: 2i: 3 j: 2`

b. `i: 1 j: 3i: 2 j: 3i: 3 j: 3`

c. `i: 1 j: 1i: 2 j: 1i: 3 j: 1`
