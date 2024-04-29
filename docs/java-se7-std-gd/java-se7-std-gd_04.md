# 第四章。使用数组和集合

这一章，归根结底，是关于数据结构。具体来说，它是关于数组——`java.util.Arrays`和`java.util.ArrayList`类。数组是可以使用单个变量名寻址的内存区域。它提供了一种有效的访问数据的技术，可以按顺序或随机方式访问数据。`Arrays`类提供了对数组的支持，而`ArrayList`类提供了类似数组的行为，但大小不固定。

我们关心如何创建和使用这些数据结构。一个常见的操作是遍历数组或集合。我们将看到 Java 支持几种方法，允许我们遍历数组或`ArrayList`对象的元素。数组和集合都可以使用 for-each 语句。迭代器提供了访问集合的另一种方法，比如`ArrayList`，我们也会讨论到。

我们将从详细讨论数组开始。这将包括创建和使用单维和多维数组。将演示常见的数组操作，比如复制和排序。

由于数组是一种简单的数据结构，大多数语言对它们的操作提供的支持不多。`java.util.Arrays`类填补了这一空白，并支持对数组的重要操作。这些操作包括复制、填充和排序数组。

`java.util`包含许多接口和类，可以使处理数据集合变得更加容易。在本章中，我们将研究迭代器和`ArrayList`类的使用，它们是这个包的一部分。迭代器提供了一种遍历集合的技术，这可能非常有用。`ArrayList`类经常用于代替数组，当集合的大小可能会改变时。它提供了许多有价值的方法来修改集合。我们还将研究如何将集合（比如`ArrayList`）封装在另一个类中。

# 数组

数组允许使用单个变量名访问多个值。数组的每个元素都是相同类型的。元素类型可以是简单的原始数据类型或对象的引用。

一维数组分配到内存的连续区域。这意味着可以有效地访问数组的元素，因为它们是相邻的。数组使用整数索引来访问数组中的元素。索引范围从 0 到数组长度减一。我们能够直接访问数组的元素，按照应用程序需要的任何顺序，而无需访问每个元素。

尽管 Java 支持多维数组，但一维数组最常用。数组可用于各种目的，包括：

+   代表年龄的数字集合

+   员工姓名列表

+   商店商品价格列表

数组的主要缺点是它们具有固定的大小。这使得向数组表示的任何列表或集合添加、删除或调整大小更加困难和低效。

我们的讨论将从一维和多维数组的覆盖开始。接下来是关于常见数组技术的讨论，比如遍历数组和复制数组。在前两个部分中，我们将使用简单的“for 循环”来遍历数组元素。替代方法在*遍历数组*部分中介绍。

## 一维数组

一维数组旨在表示一个简单的线性列表。以下代码片段说明了一维数组的声明和使用。数组`ages`在第一条语句中声明为`int`类型的数组。在第二条语句中，使用`new`运算符为数组分配内存。在这个例子中，数组由`5`个元素组成：

```java
int[] ages;
ages = new int[5];
```

`ages`数组有`5`个元素分配给它。任何数组的第一个索引都是 0。数组的最大索引是其长度-1。因此，数组的最后一个索引是 4。如果使用的索引超出了数组的合法值范围，将生成运行时异常。数组可以使用单个语句声明和创建，如下所示：

```java
int[] ages = new int[5];
```

由于数组名称是对数组的引用，因此可以在程序的后期将不同的数组分配给变量。我们将在讨论中稍后演示这一点。

数组是从称为**堆**的内存区域分配的对象。堆和程序堆栈在第二章的*堆栈和堆*部分中介绍，*Java 数据类型及其使用*。在以下示例中，`ages`的第一个元素被赋予值`35`，然后显示：

```java
ages[0] = 35;
System.out.println(ages[0]);
```

数组具有`length`属性，返回数组中的元素数。当执行下一个代码序列时，它将返回`5`。请注意，`length`不是一个方法：

```java
int length = ages.length;
System.out.println(length);
```

在 Java 中，数组表示为对象。在上一个示例中，`ages`是一个对象引用变量，引用了分配给堆的数组。这在下图中有所说明：

![一维数组](img/7324_04_01.jpg)

在此示例中，数组的每个元素都默认初始化为 0，然后将第一个元素赋值为 35。

### 注意

任何尝试使用超出数组范围的索引都将生成`java.lang.ArrayIndexOutOfBoundsException`异常。

### 数组括号的放置

在声明数组时，括号的放置还有第二种选择。我们还可以在数组名称后放置括号，如下所示：

```java
int ages[];
```

对于编译器来说，这等同于先前的声明。但是，括号与数组名称的其他用法的放置是受限制的。例如，当我们声明或引用数组的元素时，必须在数组名称后放置括号。如果我们在声明数组时尝试以下操作：

```java
ages = new [5]int;
```

我们将得到以下语法错误：

```java
<identifier> expected
';' expected

```

同样，如果我们尝试在引用数组的元素时在数组名称之前使用括号，如下所示：

```java
[0]ages = 0;
```

我们得到以下语法错误消息：

```java
illegal start of expression
incompatible types
 required: int[]
 found:    int

```

更常见的是在数组的数据类型后使用括号。例如，大多数 IDE 在某个时候都会生成一个`main`方法。它经常出现在数据类型后面跟着括号的下面：

```java
public static void main(String[] args) {
   ...
}
```

另外，考虑以下声明：

```java
int[] arr1, arr2;
```

`arr1`和`arr2`都声明为数组。这是在单行上声明多个数组的简单方法。然而，可以说以下格式更好，因为它更明确：

```java
int arr1[], arr2[];
```

也可以说，在同一行声明多个变量是一种不好的形式。声明这两个数组的最佳方法如下：

```java
int[] arr1;
int[] arr2;
```

### 初始化数组

数组的元素被初始化为默认值，如下表所示。此表是从第二章的*初始化标识符*部分中复制的，*Java 数据类型及其使用*，以方便您查看：

| 数据类型 | 默认值（对于字段） |
| --- | --- |
| `boolean` | false |
| `byte` | 0 |
| `char` | '\u0000' |
| `short` | 0 |
| `int` | 0 |
| `long` | 0L |
| `float` | 0.0f |
| `double` | 0.0d |
| `String`（或任何对象） | null |

在上一个示例中，我们将值 35 赋给了数组的第一个元素。这是一种简单但繁琐的初始化数组的方法，以便将数组初始化为除默认值之外的值。

另一种技术是使用块语句初始化数组。在下面的示例中，`ages`初始化为五个不同的值。在使用块语句初始化数组时，不需要指定数组大小：

```java
int ages[] = {35, 10, 43, -5, 12};
```

如果尝试指定数组的大小，将生成语法错误，如下所示：

```java
int ages[5] = {35, 10, 43, -5, 12};
```

消息将显示如下：

```java
']' expected
';' expected

```

如果要显示数组的内容，有几种方法可用。在这里，我们将使用简单的索引和`length`属性。在*遍历数组*部分，我们将演示其他技术。

以下代码序列显示了使用`toString`方法和 for 循环显示数组之间的区别：

```java
int ages[] = {35, 10, 43, -5, 12};
System.out.println(ages.toString());

for(int i = 0; i < ages.length; i++) {
   System.out.println(ages[i]);
}
```

执行时，我们得到以下输出：

```java
[I@27341e11
35
10
43
-5
12

```

请注意，`toString`方法的使用并不返回数组的内容。相反，它返回数组的奇怪表示。我们无法控制`toString`方法返回的字符串。然而，for 循环给了我们预期的结果。

### 注意

请记住，Java 中的数组始终从索引 0 开始。

与在之前的示例中硬编码数组的大小为`5`不同，更好的方法是使用一个常量。例如，整个序列可以重写如下：

```java
static final int SIZE = 5;

int ages[] = new int[SIZE];
// initialize ages as needed

for(int i = 0; i < ages.length; i++) {
   System.out.println(ages[i]);
}
```

### 注意

使用命名常量来表示数组大小。但是，一旦数组声明后，使用`length`属性更可取，因为如果数组大小发生变化，它更易于维护。

## 对象数组

保持对象引用变量和对象本身之间的清晰区分是很重要的。对象数组使用引用变量，例如下面声明的`names`变量，它是包含对数组对象引用的单个内存位置。数组的每个元素是另一个引用，可能引用一个字符串。最初，它们被分配了`null`值。

```java
public static void main(String args[]) {
   String names[] = new String[5];
   ...
}
```

这个示例的内存分配如下图所示。但是，我们没有在图中包含数组的索引。我们可以假设顶部元素的索引为 0，最后一个元素的索引为 4：

![对象数组](img/7324_04_02.jpg)

当将字符串分配给数组的一个元素时，该数组元素将被修改为引用该字符串，如下所示：

```java
names[2] = "Steve";
```

下图说明了将索引 2 处的引用修改为引用字符串：

![对象数组](img/7324_04_03.jpg)

在使用可能包含空值的数组时要小心。考虑以下代码序列，其中我们显示`names`数组的内容：

```java
for(int i = 0; i < names.length; i++) {
   System.out.println(names[i]);
}
```

执行时，我们将得到以下输出：

```java
null
null
Steve
null
null

```

## 多维数组

许多应用程序需要使用具有两个或更多维度的数组。具有行和列的表格数据或使用 x/y 坐标系的数据是使用二维数组表示的良好候选。三个或更多的高维度不太常见，但使用 x、y 和 z 值的坐标系将使用三个维度。在本节中，我们将使用整数来演示多维数组。然而，这些技术同样适用于对象数组。

二维数组的声明示例如下所示：

```java
static final int ROWS = 2;
static final int COLS = 3;

int grades[][] = new int[ROWS][COLS];
```

这将创建一个具有 2 行 3 列的数组，逻辑上如下图所示：

![多维数组](img/7324_04_04.jpg)

请注意，索引从零开始。我们可以使用一系列赋值语句初始化每个元素，如下所示：

```java
grades[0][0] = 0;
grades[0][1] = 1;
grades[0][2] = 2;
grades[1][0] = 3;
grades[1][1] = 4;
grades[1][2] = 5;
```

这有点繁琐，但它说明了将数字放入数组的位置，如下图所示：

![多维数组](img/7324_04_05.jpg)

嵌套循环对于处理二维数组非常有用。例如，要显示这些数组的内容，我们将使用一组嵌套的 for 循环，如下所示：

```java
for (int rows = 0; rows < ROWS; rows++) {
   for (int cols = 0; cols < COLS; cols++) {
      System.out.printf("%d  ", grades[rows][cols]);
   }
   System.out.println();
}
```

执行时，我们得到以下输出：

```java
0  1  2 
3  4  5 

```

实际上，Java 不严格支持二维数组。实际上它们是数组的数组。在诸如 C 之类的语言中，二维数组按行列顺序存储。这意味着二维数组被映射到一个一维空间，其中第一行存储在内存中，然后是第二行，然后是第三行，依此类推。这在 Java 中不适用。

实际上，我们拥有的是一个一维数组，其中包含对一系列其他一维数组的引用。例如，我们可以创建相同的`grades`数组：

```java
grades = new int[ROWS][];
grades[0] = new int[COLS];
grades[1] = new int[COLS];
```

数组在内存中分配，如下图所示：

![多维数组](img/7324_04_06.jpg)

在二维数组中，行不一定要具有相同的大小。在以下代码序列中，我们创建了一个具有不同行长度的数组。这种类型的数组称为**不规则数组**。

```java
grades[0] = new int[4];
grades[1] = new int[2];
```

内存分配与前面的示例类似，只是数组长度不同，如下图所示：

![多维数组](img/7324_04_07.jpg)

## 数组技术

有许多处理数组的技术。在本节中，我们将研究其中许多技术，包括：

+   遍历数组

+   比较数组

+   复制数组

+   传递数组

+   使用命令行参数

我们将根据需要演示每种技术的变化。将可变数量的参数传递给方法在第六章*类、构造函数和方法*中有介绍。

### 遍历数组

遍历数组是访问数组的每个元素的过程。通常从第一个元素开始，逐个元素移动，直到到达数组的末尾。但是，也可以向后移动或跳过元素。在这里，我们将重点介绍如何使用两种不同的技术从头到尾遍历数组：

+   使用简单的 for 循环

+   使用 for-each 语句

我们将使用如下声明的`ages`数组来说明如何遍历数组：

```java
static final int SIZE = 5;
int[] ages = new int[SIZE];
```

在每个示例中，我们将使用此代码将数组的每个元素初始化为 5。

#### 使用简单循环

任何简单的循环都可以用来遍历数组。循环结构将在第五章*循环结构*中更详细地介绍。在这里，我们将使用 for 循环和 while 循环。首先，让我们来看看 for 循环。在以下序列中，一个整数变量从 0 开始，逐步增加到数组长度减一：

```java
for(int i = 0; i < ages.length; i++) {
      ages[i] = 5;
}
```

等效的 while 循环如下。请注意，`i`变量在循环外声明：

```java
int i = 0;
while(i < ages.length) {
   ages[i++] = 5;
}      
```

通常情况下，for 循环是首选，因为我们知道数组的长度，并且对于这些类型的问题来说更简单。在这两个示例中，我们使用数组的`length`属性来控制循环。这比使用可能用于声明数组的常量变量更可取。考虑以下情况，我们重新定义数组：

```java
int[] ages = new int[SIZE];
...
for(int i = 0; i < SIZE; i++) {
   ages[i] = 5;
}

// Array redefined
int[] ages = new int[DIFFERENT_SIZE];
...
for(int i = 0; i < SIZE; i++) {
   ages[i] = 5;
}
```

第二个 for 循环将无法正确执行，因为我们忘记更改`SIZE`常量，如果数组小于`SIZE`甚至可能抛出异常。如果我们使用`length`属性，就不会有问题。

请注意，如写的 for 循环在循环内声明了变量`i`。这限制了对该变量的访问仅限于 for 循环内的语句。在 while 循环示例中，我们在循环外声明了`i`，使其在 while 循环内外都可以访问。我们可以重写 for 循环以使用外部的`i`变量。但是，最好的做法是将对变量的访问限制在只有需要访问的语句中。因此，如果只在循环内部需要它，那么 for 循环提供了更好的选择。

### 注意

使用简单的 for 语句可能会导致偏移一个错误（从错误的开始或结束索引开始）。例如，如果用作最后一个索引的值大于数组大小减一，则会抛出`ArrayIndexOutOfBoundsException`异常。

#### 使用 for-each 语句

如果我们不需要显式访问每个元素的索引值，for-each 语句提供了一种更方便的遍历数组的方法。for-each 括号的主体由数据类型、变量名、冒号，然后是一个数组（或集合）组成。该语句将从第一个元素开始迭代数组，并以最后一个元素结束。在每次迭代期间，变量引用该数组元素。以下是使用该语句与`ages`数组的示例。在第一次迭代中，`number`引用`ages[0]`。在第二次迭代中，`number`引用`ages[1]`。对数组的每个元素都是如此：

```java
for(int number : ages) {
   number = 5;
}
```

for-each 语句使得遍历数组变得容易。但是，如果我们需要使用数组元素的索引，该语句不提供对其值的访问。需要使用传统的 for 循环来访问索引。

以下表格总结了使用 for 循环和 for-each 循环的差异：

|   | for 循环 | for-each 循环 |
| --- | --- | --- |
| 提供对数组元素的访问 | ![使用 for-each 语句](img/7324EN_04_10.jpg) | ![使用 for-each 语句](img/7324EN_04_10.jpg) |
| 提供对数组索引的访问 | ![使用 for-each 语句](img/7324EN_04_10.jpg) | ![使用 for-each 语句](img/7324EN_04_11.jpg) |
| 使用逻辑表达式来控制循环 | ![使用 for-each 语句](img/7324EN_04_10.jpg) | ![使用 for-each 语句](img/7324EN_04_11.jpg) |
| 最简单的 | ![使用 for-each 语句](img/7324EN_04_11.jpg) | ![使用 for-each 语句](img/7324EN_04_10.jpg) |

### 比较数组

由于数组变量是引用变量，因此比较数组引用变量以确定相等性的方法并不总是有效。在这里，我们将研究几种比较数组的技术，包括：

+   逐个元素比较

+   使用等号运算符

+   使用`equals`方法

+   使用`deepEquals`方法

我们将通过比较两个整数数组来演示这些技术。考虑以下示例，其中两个数组`arr1`和`arr2`在我们将它们初始化为包含相同数据后是等价的：

```java
public static void main(String[] args) {
   int arr1[];
   int arr2[];
   arr1 = new int[5];
   arr2 = new int[5];

   for(int i = 0; i < 5; i++) {
      arr1[i] = 0;
      arr2[i] = 0;
    }
  }
```

以下图表显示了如何为两个数组分配内存：

![比较数组](img/7324_04_08.jpg)

#### 逐个元素比较

这种简单的方法将比较每个数组的对应元素，以确定数组是否相等。它首先假定它们是相等的，并将`areEqual`变量赋值为`true`。如果任何比较为 false，则变量将被赋值为`false`：

```java
boolean areEqual = true;
for (i = 0; i < 5; i++) {
   if(arr1[i]!= arr2[i]) {
      areEqual = false;
   }
}
System.out.println(areEqual);
```

当执行此序列时，它将显示`true`。这不是最佳方法。使用索引是一种容易出错和繁琐的方法。

#### 使用等号运算符

如果我们尝试使用等号运算符比较两个数组，我们会发现比较的结果将是`false`：

```java
System.out.println(arr1 == arr2);  //Displays false
```

这是因为我们正在比较数组引用变量`arr1`和`arr2`而不是数组。变量`arr1`和`arr2`在内存中引用不同的对象。这两个引用变量的内容是不同的，因此当它们相互比较时它们是不相等的。它们不引用相同的对象。

#### 使用 equals 方法

我们可以像对其他对象一样对数组使用`equals`方法。在以下示例中，即使它们是等价的，输出也将是 false。这是因为`equals`方法作用于数组时，测试的是对象的等价性而不是对象值的等价性。

```java
System.out.println(arr1.equals(arr2));  // Displays false
```

对象等价性是指比较两个对象引用变量。如果这些变量引用相同的对象，则它们被视为等价的。对象值等价性是指当两个不同的对象被认为是等价的，因为它们的内部值相同。

#### 使用`deepEquals`方法

要正确比较两个数组，我们需要使用`Arrays`类的`equals`或`deepEquals`方法。`equals`方法使用对象标识进行比较。`deepEquals`方法对元素进行更深入的检查以进行值等价性的比较。

以下语句将显示`true`：

```java
System.out.println(Arrays.equals(arr1,arr2));
```

`deepEquals`方法需要一个对象数组。在*多维数组*部分使用的二维`grades`数组满足要求，因为它是一个数组的数组，即一个引用其他数组（即对象）的数组。

如果我们创建第二个成绩数组`grades2`，并用与`grades`相同的值填充它，我们可以使用这些方法来测试它们是否相等。创建和初始化`grades2`数组如下：

```java
int grades2[][];
grades2 = new int[ROWS][];
grades2[0] = new int[COLS];
grades2[1] = new int[COLS];

grades2[0][0] = 0;
grades2[0][1] = 1;
grades2[0][2] = 2;
grades2[1][0] = 3;
grades2[1][1] = 4;
grades2[1][2] = 5;
```

如果我们执行以下序列：

```java
System.out.println(grades == grades2);
System.out.println(grades.equals(grades2));
System.out.println(Arrays.equals(grades, grades2));
System.out.println(Arrays.deepEquals(grades, grades2));
```

我们将得到以下输出：

```java
false
false
false
true

```

前三个比较返回`false`，因为它们没有充分比较两个数组。第四种技术对数组进行了深入比较，并准确确定了它们的等价性。

下表总结了这些技术：

| 技术 | 注释 |
| --- | --- |
| 逐元素比较 | 如果正确实现，这将正确比较数组。 |
| 使用等号运算符 | 只有当两个引用变量引用相同的对象时才有效。 |
| 使用数组的`equals`方法 | 只有当两个引用变量引用相同的对象时才有效。 |
| 使用`Array`类的`equals`方法 | 这将适用于一维数组。 |
| 使用`Array`类的`deepEquals`方法 | 这使用对象的`equals`方法进行更深层次的比较。 |

### 复制数组

有时我们需要将一个数组复制到另一个数组。在本节中，我们将研究实现这一目标的各种技术。这些包括：

+   简单的逐元素复制

+   使用`System.arraycopy`方法

+   使用`Arrays.copyOf`方法

+   使用`Arrays.copyOfRange`方法

+   使用`clone`方法

我们将使用以下声明的两个一维数组来演示这些技术：

```java
int arr1[] = new int[5];
int arr2[] = new int[5];
```

我们将使用以下代码将`arr1`的每个元素初始化为其索引：

```java
for(int i = 0; i < arr1.length; i++) {
   arr1[i] = i;
}
```

在本节的示例中，目标数组的内容作为注释跟随。

我们还将使用术语**浅复制**和**深复制**。浅复制是指只复制引用值。复制操作后，原始对象未被复制。在深复制中，对象的引用未被复制。相反，将创建对象的新副本。我们将看到这里演示的一些技术只执行浅复制，这可能并不总是理想的。

#### 简单的逐元素复制

一个简单的技巧是使用如下所示的 for 循环：

```java
for(int i = 0; i < arr1.length; i++) {
   arr2[i] = arr1[i];
}  
// 0, 1, 2, 3, 4
```

这是一种简单的方法，但您需要小心使用正确的数组索引。这种技术在多维数组中变得更加复杂。

#### 使用`System.arraycopy`方法

`System`类的`arraycopy`方法将尝试将一个数组的所有或部分内容复制到另一个数组中。每个数组中的起始位置都有指定，并指定要复制的元素数量。

要将`arr1`的所有元素复制到`arr2`，我们可以使用以下代码：

```java
System.arraycopy(arr1, 0, arr2, 0, 5);
// 0, 1, 2, 3, 4
```

该方法的参数在下表中详细说明：

| 参数 | 描述 |
| --- | --- |
| `1` | 源数组 |
| `2` | 源数组中的起始索引 |
| `3` | 目标数组 |
| `4` | 目标数组中的起始索引 |
| `5` | 要复制的元素数量 |

下一个序列将`arr1`的前三个元素复制到`arr2`的最后三个元素：

```java
System.arraycopy(arr1, 0, arr2, 2, 3);
// 0  0  0  1  2
```

我们还可以将一个数组的一部分复制到同一数组的其他位置。在这里，我们将前两个元素复制到`arr1`数组的最后两个元素：

```java
System.arraycopy(arr1, 0, arr1, 3, 2);
// 0  1  2  0  1
```

在使用这种技术时，有许多异常发生的机会。如果任一数组引用为 null，则会抛出`NullPointerException`异常。如果数组索引无效，则会得到`IndexOutOfBoundsException`异常。

`arraycopy`方法将源数组的指定元素复制到目标数组的相应元素。根据数组的数据类型，可能会有两种可能的结果。它们如下：

+   如果数组元素类型是原始数据类型，则两个数组实际上是相同的。

+   如果数组元素类型是引用类型，则两个数组将是相同的，但它们都将引用相同的对象。这通常不是预期或期望的效果。

在以下代码序列中，尝试创建`StringBuilder`数组`arr3`的相同副本：

```java
StringBuilder arr3[] = new StringBuilder[4];
arr3[0] = new StringBuilder("Pine");
arr3[1] = new StringBuilder("Oak");
arr3[2] = new StringBuilder("Maple");
arr3[3] = new StringBuilder("Walnut");
StringBuilder arr4[] = new StringBuilder[4];
System.arraycopy(arr3, 0, arr4, 0, 4);
```

然而，`arr4`包含与`arr3`使用的相同对象引用变量。两个数组的相应元素引用相同的对象。通过以下代码实现了具有对不同字符串的引用的相同数组的创建：

```java
for (int i = 0; i < arr3.length; i++) {
   arr4[i] = new StringBuilder(arr3[i]);
}
```

我们为目标数组的每个元素创建了一个新的`StringBuilder`对象。如果需要深复制，则需要使用这种方法。

#### 使用 Arrays.copyOf 方法

`Arrays`类的`copyOf`方法将基于现有数组创建一个新数组。该方法的第一个参数指定原始数组。它的第二个参数指定要复制的元素数量。在下面的示例中，我们基于`arr1`的前三个元素创建一个新数组：

```java
arr2 = Arrays.copyOf(arr1, 3);
// 0  1  2
```

新数组可以比原始数组大，如下面的代码所示：

```java
arr2 = Arrays.copyOf(arr1, 10);
// 0  1  2  3  4  0  0  0  0  0
```

`arr2`的最后五个元素将填充为零。

如果数组是对象数组，则将原始对象的副本分配给新数组。

#### 使用 Arrays.copyOfRange 方法

`Arrays`类的`copyOfRange`方法将基于现有数组中的子范围的元素创建一个新数组。该方法的第一个参数指定原始数组。它的第二个参数指定开始索引，最后一个参数指定结束索引（不包括）。在下面的示例中，我们基于`arr1`的最后两个元素创建一个新数组：

```java
arr2 = Arrays.copyOfRange(arr1, 3, 5);
//  3  4
```

请注意，最后一个参数对于`arr1`数组不是有效的索引。这里有效是因为最后一个参数是排他的。它不包括该元素。

实际上，如果在下一个示例中指定一个值，比如`8`，新数组将填充为零：

```java
arr2 = Arrays.copyOfRange(arr1, 3, 8);
//     3  4  0  0  0
```

#### 使用 clone 方法

您还可以使用`Object`类的`clone`方法来创建数组的副本：

```java
arr2 = arr1.clone();
```

然而，这只是对原始对象进行浅复制。对于原始数据类型的数组，比如上面的整数数组，这不是问题。对于对象的引用数组，两个数组将引用相同的对象。

以下表格总结了本节介绍的复制技术：

| 技术 | 评论 |
| --- | --- |
| 逐个元素简单复制 | 繁琐，但可以实现浅复制或深复制 |
| 使用`System.arraycopy`方法 | 执行浅复制 |
| 使用`Arrays.copyOf`方法 | 执行整个数组的深复制 |
| 使用`Arrays.copyOfRange`方法 | 对数组的一部分执行深复制 |
| 使用`clone`方法 | 执行浅复制 |

### 传递数组

将数组传递给方法的优点是，它允许我们对一个以上的数组执行相同的操作。要将数组传递给方法，我们在方法调用中使用数组名称，并在方法中声明对传递数组的引用。下面通过调用`displayArray`方法来说明这一点。该方法只是显示数组。

```java
displayArray(arr2);
   ...
private static void displayArray(int arr[]) {
   for(int number : arr) {
      System.out.print(number + "  ");
   }
   System.out.println();
}
```

请注意，我们是通过值“传递引用”到`arr2`数组的。也就是说，如果需要，我们可以在方法中读取和写入`arr2`数组的元素。但是，如果修改`arr`参数，则原始的`arr2`变量不会被修改。

考虑以下代码中的方法，该方法尝试更改`arr2`引用变量指向的内容：

```java
System.out.println("Length of arr2: " + arr2.length);
changeArray(arr2);
System.out.println("Length of arr2: " + arr2.length);
...    
private static void changeArray(int arr[]) {
   arr = new int[100];
   System.out.println("Length of arr: " + arr.length);
}
```

当我们执行此代码时，我们会得到以下输出：

```java
Length of arr2: 5
Length of arr: 100
Length of arr2: 5

```

`arr`的值被改变，但`arr2`的值没有改变。以下的图表应该有助于澄清这种行为：

![传递数组](img/7324_04_09.jpg)

### 使用命令行参数

当 Java 应用程序执行时，执行的第一个方法是`main`方法。该方法传递一个参数，一个名为`args`的`String`对象数组。这些字符串对应于命令行提供的字符串。

Java 数组的`length`属性将告诉我们使用了多少个命令行参数。数组的第一个参数将包含第一个命令行参数。第二个将包含第二个命令行参数，依此类推。

以下`CommandLineDemo`应用程序演示了`args`数组的使用：

```java
public class CommandLineDemo {

   public static void main(String args[]) {
      System.out.println("The command line has " + args.length + " arguments");
      for (int i = 0; i < args.length; i++) {
         System.out.println("\tArgument Number " + i + 
                  ": " + args[i]);
      }
   }
}
```

考虑应用程序使用以下命令行参数调用：

```java
java CommandLineDemo /D 1024 /f test.dat

```

程序的输出将如下所示：

```java
The command line has 4 arguments
 Argument Number 0: /D
 Argument Number 1: 1024
 Argument Number 2: /f
 Argument Number 3: test.dat

```

## 数组类

`java.util.Arrays`类拥有几种对数组进行操作的有用方法。该类的每个方法都是静态方法，这意味着我们在使用其方法之前不必创建`Arrays`类的实例。该类旨在处理数组并对数组执行常见操作。可用的操作类型包括：

+   基于数组返回一个`List`

+   执行二分搜索

+   复制数组

+   确定两个数组的相等性

+   填充数组

+   对数组进行排序

我们已经在前面的部分看到了几种这些技术的使用。在这里，我们将演示`asList`，`fill`，`toString`和`deepToString`方法的使用。

考虑以下声明。我们将声明一个整数数组，然后声明一个数组列表。两个字符串将被添加到`ArrayList`对象中。我们还将创建一个混合对象数组和一个字符串数组。`ArrayList`类在*ArrayList*部分中有更详细的讨论：

```java
int arr1[] = new int[5];
ArrayList list = new ArrayList();
list.add("item 1");
list.add("item 2");

Object arr2[] = {"item 3", new Integer(5), list};
String arr3[] = {"Pine", "Oak", "Maple", "Walnut"};
```

接下来，我们将使用`fill`方法用数字`5`填充整数数组：

```java
Arrays.fill(arr1,5);
```

然后使用`asList`，`toString`和`deepToString`方法对这些数组进行操作，如下所示：

```java
System.out.println(Arrays.asList(arr3));
System.out.println(Arrays.toString(arr1));
System.out.println(Arrays.deepToString(arr2));
```

执行时，我们得到以下输出：

```java
[Pine, Oak, Maple, Walnut]
 [5, 5, 5, 5, 5]
[item 3, 5, [item 1, item 2]]
```

`asList`方法接受其数组参数并返回表示数组的`java.util.List`对象。如果修改了数组或列表，它们对应的元素也会被修改。这在以下示例中得到了证明：

```java
List list2 = Arrays.asList(arr3);
list2.set(0, "Birch");
System.out.println(Arrays.toString(arr3));
```

这个序列的输出如下：

```java
[Birch, Oak, Maple, Walnut]

```

`toString`方法返回数组的字符串表示。`deepToString`方法旨在返回其数组参数的字符串表示，其中数组更复杂。这在`arr2`中得到了体现，其中包含了不同的对象，包括一个列表。

## 在使用数组时要记住的关键点

在处理数组时，请记住：

+   数组索引从 0 开始

+   索引必须是整数

+   数组可以保存原始数据类型或对象

+   数组提供了常数时间的随机访问，这是一种访问数据的高效方式

+   数组提供良好的引用局部性

+   与其他数据结构相比，数组更难插入或删除元素

+   可能存在对无效元素的索引

引用的局部性是指如果访问一个数据项，很可能也会访问附近的另一个数据项。这导致更快的读写操作，是虚拟操作系统中的一个重要概念。在内存中分布的链表的访问元素可能比链表的访问元素更快。

在访问数组元素时要小心。如果数组没有正确初始化，那么索引的元素可能无效，导致运行时或逻辑错误。

# 集合

集合框架是在 Java 2 中引入的一组接口和类，它们优于早期`java.util`包中找到的许多接口和类，如`Vector`、`Stack`和`HashTable`。这些接口和类应该在可能的情况下始终使用，而不是旧的接口和类。集合框架的许多接口和类在以下表中进行了总结：

| 接口 | 类 |
| --- | --- |
| 集合 | `HashSet``TreeSet` |
| 列表 | `ArrayList``LinkedList` |
| 映射 | `HashMap``TreeMap` |

集合框架在[`java.sun.com/developer/onlineTraining/collections/Collection.html`](http://java.sun.com/developer/onlineTraining/collections/Collection.html)中有更详细的介绍。在这里，我们将讨论`ArrayList`类，因为它是一个认证主题。建议在需要`List`时使用`ArrayList`类。正如我们将看到的，迭代器用于`ArrayList`来支持列表的遍历。我们将从这个主题的覆盖开始我们的讨论。

## 迭代器

迭代器提供了遍历一组数据的方法。它可以与数组和集合框架中的各种类一起使用。`Iterator`接口支持以下方法：

+   `next`：该方法返回下一个元素

+   `hasNext`：如果有附加元素，则该方法返回`true`

+   `remove`：该方法从列表中移除元素

`remove`方法是一个可选的`Iterator`方法。如果尝试使用此方法，而接口的实现不支持此方法，则会抛出`UnsupportedOperationException`异常。

`ListIterator`接口，当可用时，是`Iterator`接口的替代品。它使用相同的方法并提供额外的功能，包括：

+   在任一方向上遍历列表

+   修改其元素

+   访问元素的位置

`ListIterator`接口的方法包括以下内容：

+   `next`：该方法返回下一个元素

+   `previous`：该方法返回前一个元素

+   `hasNext`：如果有跟随当前元素的附加元素，则该方法返回`true`

+   `hasPrevious`：如果有前面的附加元素，则该方法返回`true`

+   `nextIndex`：该方法返回`next`方法将返回的下一个元素的索引

+   `previousIndex`：该方法返回`previous`方法将返回的上一个元素的索引

+   `add`：该方法在列表中插入一个元素（可选）

+   `remove`：该方法从列表中移除元素（可选）

+   `set`：该方法替换列表中的一个元素（可选）

## ArrayList

`ArrayList`类具有几个有用的特性：

+   它是灵活的

+   根据需要增长

+   具有许多有用的方法

+   访问在常数时间内执行

+   插入/删除在线性时间内执行

+   可以使用索引、for-each 循环或迭代器遍历

`ArrayList`在内部使用数组。当需要增长时，元素从旧数组复制到新数组。

`ArrayList`类不是同步的。当为`ArrayList`对象获取迭代器时，如果以并发方式修改，可能会导致数据丢失的可能同时覆盖。当多个线程访问同一对象时，它们可能同时写入对象，即并发。当发生这种同时覆盖时，会抛出`ConcurrentModificationException`异常。

### 创建 ArrayList

`ArrayList`类具有以下三个构造函数：

+   默认构造函数

+   接受`Collection`对象

+   接受初始容量

`ArrayList`对象的容量指的是列表可以容纳多少元素。当需要添加更多元素且列表已满时，列表的大小将自动增加。使用其默认构造函数创建的`ArrayList`的初始容量为`10`。以下示例创建了两个列表，一个容量为`10`，另一个容量为`20`：

```java
ArrayList list1 = new ArrayList();
ArrayList list2 = new ArrayList(20);
```

`ArrayList`类支持泛型。在这里，创建了一个字符串列表：

```java
ArrayList<String> list3 = new ArrayList<String>();
```

我们将在接下来的示例中使用`list3`。

### 添加元素

有几种方法可用于向`ArrayList`添加元素。它们可以被放入以下两个类别之一：

+   将一个或多个元素附加到列表的末尾

+   在列表中的位置插入一个或多个元素

最简单的情况如下所示，其中一个字符串被添加到`creatures`的末尾：

```java
ArrayList<String> creatures = new ArrayList<String>();
creatures.add("Mutant");
creatures.add("Alien");
creatures.add("Zombie");
System.out.println(creatures);
```

打印语句的输出如下：

```java
[Mutant, Alien, Zombie]

```

要在第一个元素之后的索引处插入一个元素，我们使用索引 1：

```java
creatures.add(1,"Godzilla");
System.out.println(creatures);
```

执行代码将验证操作，如下所示：

```java
 [Mutant, Godzilla, Alien, Zombie]

```

`addAll`方法也可以与`Collections`一起使用，如下所示：

```java
ArrayList<String> cuddles = new ArrayList<String>();
cuddles.add("Tribbles");
cuddles.add("Ewoks");

creatures.addAll(2, cuddles);
System.out.println(creatures);
```

这将导致`cuddles`被放置在列表中第二个元素之后，如下所示：

```java
[Mutant, Godzilla, Tribbles, Ewoks, Alien, Zombie]
```

`addAll`方法也可以不带索引参数使用。在这种情况下，新元素将添加到列表的末尾。

### 检索元素

要检索给定位置的元素，请使用`get`方法。此方法接受一个整数索引值。在下面的示例中，我们检索列表的第三个元素。假设 creatures 列表包含`[Mutant, Godzilla, Tribbles, Ewoks, Alien, Zombie]`，以下语句将检索`Tribbles`：

```java
String element = creatures.get(2);
```

可以使用`indexOf`方法获取元素的索引，如下一个代码序列所示。如果元素不存在，该方法将返回-1。

```java
System.out.println(creatures.indexOf("Tribbles"));
System.out.println(creatures.indexOf("King Kong"));
```

执行此代码将生成以下输出：

```java
2
-1

```

`indexOf`方法将返回找到的第一个元素的索引。`lastIndexOf`方法将返回列表中找到的最后一个元素的索引。

`toArray`方法将返回列表中对象的数组。在此示例中，返回`creatures`列表并将其分配给`complete`数组。如果数组不够大，就像这里一样，将创建并返回一个新数组。

```java
String[] complete = new String[0];
complete = creatures.toArray(complete);
for(String item : complete) {
   System.out.print(item + " ");
}
System.out.println();
```

执行时，我们得到以下输出：

```java
Mutant Godzilla Tribbles Ewoks Alien Zombie

```

还有一个`subList`方法，根据起始和结束索引返回列表的一部分。

### 遍历`ArrayList`对象

要遍历`ArrayList`对象，我们可以使用几种方法之一：

+   一个简单的 for 语句

+   一个 for-each 语句

+   使用`Iterator`

+   使用`ListIterator`

我们可以使用 for 循环，但更容易出错。以下代码将从头到尾显示列表：

```java
for(int i = 0; i < creatures.size(); i++) {
   System.out.print(creatures.get(i) + " ");
}
System.out.println();
```

注意使用`size`方法，它返回列表中的元素数。

for-each 语句是最简单的方法，如下面的代码片段所示：

```java
for(String creature : creatures) {
   System.out.print(creature + " ");
}
System.out.println();
```

`iterator`方法返回一个`Iterator`对象，如下所示：

```java
Iterator<String> iterator = creatures.iterator();
while(iterator.hasNext()) {
   System.out.print(iterator.next() + " ");
}
System.out.println();
```

`ListIterator`方法返回一个`ListIterator`对象：

```java
ListIterator<String> listIterator = 
            creatures.listIterator();
while(listIterator.hasNext()) {
   System.out.print(listIterator.next() + " ");
}
System.out.println();
```

这四种技术都将产生以下相同的输出：

```java
Mutant Godzilla Tribbles Ewoks Alien Zombie

```

如果我们将以下代码添加到上一个代码序列的末尾，我们可以按照逆序遍历列表，如下面的代码片段所示：

```java
while(listIterator.hasPrevious()) {
   System.out.print(listIterator.previous() + " ");
}
System.out.println();
```

输出如下：

```java
Zombie Alien Ewoks Tribbles Godzilla Mutant

```

### 对`ArrayList`对象进行排序

虽然`ArrayList`类中没有特定的排序方法，但我们可以使用`Arrays`类的`sort`方法，如下面的代码片段所示：

```java
Collections.sort(creatures);
System.out.println(creatures);
```

输出如下：

```java
[Alien, Ewoks, Godzilla, Mutant, Tribbles, Zombie]

```

此方法的重载版本使用`Comparator`对象。此对象确定如何进行比较。

### 其他`ArrayList`方法

我们可以使用`set`方法修改列表的一个元素。此方法接受要替换的元素的索引和新值。例如，要用字符串`Ghoul`替换 creatures 列表的第一个元素，我们可以使用以下代码：

```java
creatures.set(0,"Ghoul");
System.out.println(creatures);
```

以下输出验证了替换：

```java
[Ghoul, Godzilla, Tribbles, Ewoks, Alien, Zombie]

```

我们可以移除列表的所有或部分元素。`clear`方法将移除所有元素。`remove`方法将移除单个元素，`removeAll`方法将从列表中移除给定集合中的所有值。以下代码序列说明了这些方法。`cuddles` `ArrayList`在*添加元素*部分中被定义：

```java
System.out.println(creatures);
creatures.remove(0);
System.out.println(creatures);

creatures.remove("Alien");
System.out.println(creatures);

creatures.removeAll(cuddles);
System.out.println(creatures);

creatures.clear();
System.out.println(creatures); 
```

此序列的输出如下：

```java
[Mutant, Godzilla, Tribbles, Ewoks, Alien, Zombie]
[Godzilla, Tribbles, Ewoks, Alien, Zombie]
[Godzilla, Tribbles, Ewoks, Zombie]
[Godzilla, Zombie]
[]

```

虽然`ArrayList`是一个强大的类，但如果：

+   已知元素的数量

+   它有一个小的固定上限

+   需要原始数据类型以提高效率

+   不需要插入元素

## 封装集合

在类中使用集合时，隐藏集合以防止无意修改集合。例如，如果一个类封装了一个`ArrayList`的`Books`，那么应该提供公共方法来允许访问集合。在下面的例子中，一个名为`Library`的类隐藏了`Book`对象的`ArrayList`：

```java
public class Library {

   private ArrayList<Book> books = new ArrayList<Book>();

   public Book getBook(int index) {
      return books.get(index);
   }

   public void addBook(Book book) {
      books.add(book);
   }

   public List getBooks() {
      return books;
   }
}
```

这是数据封装的一个很好的例子。但是，请确保不要无意中暴露私有数据。在`getBook`方法中，我们返回了对书的引用。这个引用允许用户修改书。如果不允许进行此修改，则可以返回书的副本，如下所示。这假设`Book`类有一个构造函数，根据构造函数的参数制作书的新副本：

```java
public Book getBook (int index) {
   return new Book(books.get(index));
}
```

`getBooks`方法也存在相同的问题。它返回`Library`类的私有`books`引用变量的引用。可以用以下实现替换此方法以确保正确的数据封装：

```java
public List getBooks() {
   ArrayList list = new ArrayList(books.size());
   for(Book book : books) {
      list.add(new Book(book));
   }
   return list;
}
```

# 总结

在本章中，我们研究了数组的创建和使用以及`ArrayList`类的实例。我们还详细介绍了`Arrays`类在支持各种数组操作方面的使用。

数组包含一个或多个维度，并被视为对象。在使用数组时必须小心，以避免访问其元素时出现问题。通过对内存中的数组分配以及如何执行各种操作（例如复制和比较数组）有很好的理解，可以避免出现问题。当我们需要一个固定大小的列表时，数组是很有用的，因为它允许高效地访问其元素。

`Arrays`类提供了许多支持数组的静态方法。例如，我们可以使用`Arrays`类来复制数组，对数组进行排序和填充数组。

`ArrayList`类提供了处理数据列表的另一种方法。它提供了许多用于操作列表的方法，并且在需要时会根据添加到列表中的新元素而增长。这是它相对于数组的主要优势之一。与大多数数据结构一样，将信息封装在类中以帮助减少软件开发的复杂性是很重要的。

现在我们已经了解了数组，我们准备更仔细地查看 Java 中可用的各种循环结构。我们将在下一章中详细讨论这些结构。

集合框架引入了几个新的接口和类来替换`java.util`包中的旧版本。我们研究了`ArrayList`类及其用于操作其元素的方法。`ArrayList`类比数组更灵活，特别适用于插入和移除元素。

# 涵盖的认证目标

在本章中，我们涵盖了以下认证目标：

+   使用一维数组

+   使用多维数组

+   声明和使用`ArrayList`

# 测试你的知识

1.  以下哪个语句将编译而不会出错？

a. `int arr[];`

b. `int arr[5];`

c. `int arr[5] = {1,2,3,4,5};`

d. `int arr[] = {1,2,3,4,5};`

1.  以下哪个声明了一个支持两行和可变列数的数组？

a. `int arr[][] = new int[2][3];`

b. `int arr[][] = new int[2][];`

c. `int arr[][] = new int[][];`

d. `int arr[][] = new int[][3];`

1.  根据以下代码，哪些语句可以用来确定列表中是否可以找到`cat`？

```java
ArrayList<String> list = new ArrayList<>();
list.add("dog");
list.add("cat");
list.add("frog");
```

a. `list.contains("cat")`

b. `list.hasObject("cat")`

c. `list.indexOf("cat")`

d. `list.indexOf(1)`
