# *第六章*

# 数据结构、数组和字符串

## 学习目标

通过本课程结束时，您将能够：

+   创建和操作各种数据结构，如数组

+   描述编程算法的基本原理

+   为数组编写简单的排序程序

+   输入并对字符串执行操作

## 介绍

这是我们关于 OOP 讨论的最后一个主题。到目前为止，我们已经看过类和对象，以及如何使用类作为蓝图来创建多个对象。我们看到了如何使用方法来保存我们类的逻辑和字段来保存状态。我们讨论了类如何从其他类继承一些属性，以便轻松地重用代码。

我们还看过多态性，或者一个类如何重新定义从超类继承的方法的实现；以及重载，或者我们如何可以有多个使用相同名称的方法，只要它们具有不同的签名。我们还讨论了函数或方法。

我们在上一课中已经讨论了类型转换和接口，以及类型转换是我们将对象从一种类型更改为另一种类型的方法，只要它们在同一层次结构树上。我们谈到了向上转型和向下转型。另一方面，接口是我们定义通用行为的一种方式，我们的类可以提供自己的特定实现。

在本节中，我们将看一些 Java 自带的常见类。这些是您每天都会使用的类，因此了解它们非常重要。我们还将讨论数据结构，并讨论 Java 自带的常见数据结构。请记住，Java 是一种广泛的语言，这个列表并不是详尽无遗的。请抽出时间查看官方 Java 规范，以了解更多关于您可以使用的其他类的信息。在本课程中，我们将介绍一个主题，提供示例程序来说明概念，然后完成一个练习。

## 数据结构和算法

算法是一组指令，应该遵循以实现最终目标。它们是特定于计算的，但我们经常谈论算法来完成计算机程序中的某个任务。当我们编写计算机程序时，通常实现算法。例如，当我们希望对一组数字进行排序时，通常会想出一个算法来实现。这是计算机科学的核心概念，对于任何优秀的程序员来说都很重要。我们有用于排序、搜索、图问题、字符串处理等的算法。Java 已经为您实现了许多算法。但是，我们仍然有机会定义自己的算法。

数据结构是一种存储和组织数据以便于访问和修改的方式。数据结构的一个示例是用于保存相同类型的多个项目的数组或用于保存键值对的映射。没有单一的数据结构适用于所有目的，因此了解它们的优势和局限性非常重要。Java 有许多预定义的数据结构，用于存储和修改不同类型的数据。我们也将在接下来的部分中涵盖其中一些。

在计算机程序中对不同类型的数据进行排序是一项常见任务。

### 数组

我们在*第 3 课* *控制* *流*中提到了数组，当时我们正在讨论循环，但是值得更仔细地看一下，因为它们是强大的工具。数组是有序项目的集合。它用于保存相同类型的多个项目。Java 中数组的一个示例可能是`{1, 2, 3, 4, 5, 6, 7}`，其中保存了整数 1 到 7。这个数组中的项目数是 7。数组也可以保存字符串或其他对象，如下所示：

```java
{"John","Paul","George", "Ringo"}
```

我们可以通过使用其索引来访问数组中的项。索引是数组中项的位置。数组中的元素从`0`开始索引。也就是说，第一个数字在索引`0`处，第二个数字在索引`1`处，第三个数字在索引`2`处，依此类推。在我们的第一个示例数组中，最后一个数字在索引`6`处。

为了能够访问数组中的元素，我们使用`myArray[0]`来访问`myArray`中的第一个项目，`myArray[1]`来访问第二个项目，依此类推，`myArray[6]`来访问第七个项目。

Java 允许我们定义原始类型和引用类型等对象的数组。

数组也有一个大小，即数组中的项数。在 Java 中，当我们创建一个数组时，必须指定其大小。一旦数组被创建，大小就不能改变。

![图 6.1：一个空数组](img/C09581_Figure_06_01.jpg)

###### 图 6.1：一个空数组

### 创建和初始化数组

要创建一个数组，您需要声明数组的名称、它将包含的元素的类型和其大小，如下所示：

```java
int[] myArray = new int[10];
```

我们使用方括号`[]`来表示数组。在这个例子中，我们正在创建一个包含 10 个项目的整数数组，索引从 0 到 9。我们指定项目的数量，以便 Java 可以为元素保留足够的内存。我们还使用`new`关键字来指示一个新数组。

例如，要声明包含 10 个双精度数的数组，请使用以下方法：

```java
double[] myArray = new double[10];
```

要声明包含 10 个布尔值的数组，请使用以下方法：

```java
boolean[] myArray = new boolean[10];
```

要声明包含 10 个`Person`对象的数组，请使用以下方法：

```java
Person[] people = new Person[10];
```

您还可以创建一个数组，并在同一时间声明数组中的项（初始化）：

```java
int[] myArray = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
```

### 访问元素

要访问数组元素，我们使用方括号括起的索引。例如，要访问第四个元素，我们使用`myArray[3]`，要访问第十个元素，我们使用`myArray[9]`。

这是一个例子：

```java
int first_element = myArray[0];
int last_element = myArray[9];
```

要获取数组的长度，我们使用`length`属性。它返回一个整数，即数组中的项数：

```java
int length = myArray. length;
```

如果数组没有任何项，`length`将为 0。我们可以使用`length`和循环将项插入数组中。

### 练习 14：使用循环创建数组

使用控制流命令创建长数组可能很有用。在这里，我们将使用`for`循环创建一个从 0 到 9 的数字数组。

1.  创建一个名为`DataStr`的新类，并设置`main`方法如下：

```java
public class DataStr {
public static void main(String[] args){
}
```

1.  创建一个长度为 10 的整数数组如下：

```java
int[] myArray = new int[10];
```

1.  初始化一个`for`循环，变量从零开始，每次迭代增加一个，条件是小于数组长度：

```java
for (int i = 0; i < myArray.length; i++)
```

1.  将项`i`插入数组中：

```java
{
myArray[i] = i;
}
```

1.  使用类似的循环结构来打印循环：

```java
for (int i = 0; i < myArray.length; i++){
System.out.println(myArray[i]);
}
```

完整的代码应该如下所示：

```java
public class DataStr {
    public static void main(String[] args){
        int[] myArray = new int[10];
        for (int i = 0; i < myArray.length; i++){
            myArray[i] = i;
        }
        for (int i = 0; i < myArray.length; i++){
            System.out.println(myArray[i]);
        }
    }
}
```

您的输出应该如下所示：

![图 6.2：DataStr 类的输出](img/C09581_Figure_06_02.jpg)

###### 图 6.2：DataStr 类的输出

在这个练习中，我们使用第一个`for`循环将项目插入`myArray`中，使用第二个循环将项目打印出来。

正如我们之前讨论的，我们可以用`for-each`循环替换第二个`for`循环，这样代码会更简洁，更易读：

```java
for (int i : myArray) {
System.out.println(i);
}
```

Java 会自动为我们进行边界检查-如果您创建了一个大小为 N 的数组，并使用值小于 0 或大于 N-1 的索引，您的程序将以`ArrayOutOfBoundsException`异常终止。

### 练习 15：在数组中搜索一个数字

在这个练习中，您将检查用户输入的数字是否存在于数组中。为此，请执行以下步骤：

1.  定义一个名为`NumberSearch`的新类，并在其中包含`main`方法：

```java
public class NumberSearch {
public static void main(String[] args){
}
}
```

1.  确保在顶部导入此包，用于从输入设备读取值：

```java
import java.util.Scanner;
```

1.  声明一个名为 sample 的数组，其中存储整数 2、4、7、98、32、77、81、62、45、71：

```java
int [] sample = { 2, 4, 7, 98, 32, 77, 81, 62, 45, 71 }; 
```

1.  从用户那里读取一个数字：

```java
Scanner sc = new Scanner(System.in);
System.out.print("Enter the number you want to find: ");
int ele = sc.nextInt();
```

1.  检查`ele`变量是否与数组样本中的任何项目匹配。为此，我们遍历循环，并检查数组的每个元素是否与用户输入的元素匹配：

```java
for (int i = 0; i < 10; i++) {
  if (sample[i] == ele) {
    System.out.println("Match found at element " + i);
    break;
}
else
  {
    System.out.println("Match not found");
    break;
  }
}
```

您的输出应类似于此：

![图 6.3：NumberSearch 类的输出](img/C09581_Figure_06_03.jpg)

###### 图 6.3：NumberSearch 类的输出

### 活动 21：在数组中找到最小的数字

在这个活动中，我们将取一个包含 20 个未排序数字的数组，并循环遍历数组以找到最小的数字。

步骤如下：

1.  创建一个名为`ExampleArray`的类，并创建`main`方法。

1.  创建一个由 20 个浮点数组成的数组，如下所示：

```java
14, 28, 15, 89, 46, 25, 94, 33, 82, 11, 37, 59, 68, 27, 16, 45, 24, 33, 72, 51
```

1.  通过数组创建一个`for-each`循环，并找到数组中的最小元素。

1.  打印出最小的浮点数。

#### 注意

此活动的解决方案可在 335 页找到。

### 活动 22：具有操作符数组的计算器

在这个活动中，您将改变您的计算器，使其更加动态，并且更容易添加新的操作符。为此，您将不是将所有可能的操作符作为不同的字段，而是将它们添加到一个数组中，并使用 for 循环来确定要使用的操作符。

要完成此活动，您需要：

1.  创建一个名为`Operators`的类，其中包含根据字符串确定要使用的操作符的逻辑。在这个类中创建一个名为`default_operator`的公共常量字段，它将是`Operators`类的一个实例。然后创建另一个名为`operators`的常量字段，类型为`Operators`数组，并用每个操作符的实例进行初始化。

1.  在`Operators`类中，添加一个名为`findOperator`的公共静态方法，它接收操作符作为字符串，并返回`Operators`的一个实例。在其中，遍历可能的操作符数组，并对每个操作符使用 matches 方法，返回所选操作符，如果没有匹配任何操作符，则返回默认操作符。

1.  创建一个新的`CalculatorWithDynamicOperator`类，有三个字段：`operand1`和`operator2`为 double 类型，`operator`为`Operators`类型。

1.  添加一个构造函数，接收三个参数：类型为 double 的 operand1 和 operand2，以及类型为 String 的 operator。在构造函数中，不要使用 if-else 来选择操作符，而是使用`Operators.findOperator`方法来设置操作符字段。

1.  添加一个`main`方法，在其中多次调用`Calculator`类并打印结果。

#### 注意

此活动的解决方案可在 336 页找到。

### 二维数组

到目前为止我们看到的数组都被称为一维数组，因为所有元素都可以被认为在一行上。我们也可以声明既有列又有行的数组，就像矩阵或网格一样。多维数组是我们之前看到的一维数组的数组。也就是说，您可以将其中一行视为一维数组，然后列是多个一维数组。

描述多维数组时，我们说数组是一个 M 乘 N 的多维数组，表示数组有 M 行，每行长度为 N，例如，一个 6 乘 7 的数组：

![图 6.4：多维数组的图形表示](img/C09581_Figure_06_04.jpg)

###### 图 6.4：多维数组的图形表示

在 java 中，要创建一个二维数组，我们使用双方括号`[M][N]`。这种表示法创建了一个 M 行 N 列的数组。然后，我们可以使用`[i][j]`的表示法来访问数组中的单个项目，以访问第 i 行和第 j 列的元素。

要创建一个 8x10 的双精度多维数组，我们需要执行以下操作：

```java
double[][] a = new double[8][10];
```

Java 将所有数值类型初始化为零，布尔类型初始化为 false。我们也可以循环遍历数组，并手动将每个项目初始化为我们选择的值：

```java
double[][] a = new double[8][10];
for (int i = 0; i < 8; i++)
for (int j = 0; j < 10; j++)
a[i][j] = 0.0;
```

### 练习 16：打印简单的二维数组

要打印一个简单的二维数组，请执行以下步骤：

1.  在名为`Twoarray`的新类文件中设置`main`方法：

```java
public class Twoarray {
    public static void main(String args[]) {
    }
}
```

1.  通过向数组添加元素来定义`arr`数组：

```java
int arr[][] = {{1,2,3}, {4,5,6}, {7,8,9}};
```

1.  创建一个嵌套的`for`循环。外部的`for`循环是按行打印元素，内部的`for`循环是按列打印元素：

```java
        System.out.print("The Array is :\n");
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                System.out.print(arr[i][j] + "  ");
            }
            System.out.println();
        }
```

1.  运行程序。您的输出应该类似于这样：

![图 6.5：Twoarray 类的输出](img/C09581_Figure_06_05.jpg)

###### 图 6.5：Twoarray 类的输出

大多数与数组相关的操作与一维数组基本相同。要记住的一个重要细节是，在多维数组中，使用`a[i]`返回一个一维数组的行。您必须使用第二个索引来访问您希望的确切位置，`a[i][j]`。

#### 注意

Java 还允许您创建高阶维度的数组，但处理它们变得复杂。这是因为我们的大脑可以轻松理解三维数组，但更高阶的数组变得难以可视化。

### 练习 17：创建一个三维数组

在这里，我们将创建一个三维`(x,y,z)`整数数组，并将每个元素初始化为其行、列和深度（x * y * z）索引的乘积。

1.  创建一个名为`Threearray`的新类，并设置`main`方法：

```java
public class Threearray
{
    public static void main(String args[])
    {
    }
}
```

1.  声明一个维度为`[2][2][2]`的`arr`数组：

```java
int arr[][][] = new int[2][2][2];
```

1.  声明迭代的变量：

```java
int i, j, k, num=1;
```

1.  创建三个嵌套在彼此内部的`for`循环，以便将值写入三维数组：

```java
for(i=0; i<2; i++)
  {
    for(j=0; j<2; j++)
      {
        for(k=0; k<2; k++)
         {
         arr[i][j][k] = no;
         no++;
     }
  }
}
```

1.  使用嵌套在彼此内部的三个`for`循环打印数组的元素：

```java
for(i=0; i<2; i++)
  {
  for(j=0; j<2; j++)
    {
      for(k=0; k<2; k++)
      {
      System.out.print(arr[i][j][k]+ "\t");
      }
    System.out.println();
    }
  System.out.println();
  }
}
}
}
}
}
```

完整的代码应该是这样的：

```java
public class Threearray
{
    public static void main(String args[])
    {
        int arr[][][] = new int[2][2][2];
        int i, j, k, num=1;
        for(i=0; i<2; i++)
        {
            for(j=0; j<2; j++)
            {
                for(k=0; k<2; k++)
                {
                    arr[i][j][k] = num;
                    num++;
                }
            }
        }
        for(i=0; i<2; i++)
        {
            for(j=0; j<2; j++)
            {
                for(k=0; k<2; k++)
                {
                    System.out.print(arr[i][j][k]+ "\t");
                }
                System.out.println();
            }
            System.out.println();
        }
    }
}
```

输出如下：

![图 6.6：Threearray 类的输出](img/C09581_Figure_06_06.jpg)

###### 图 6.6：Threearray 类的输出

### Java 中的 Arrays 类

Java 提供了`Arrays`类，它提供了我们可以与数组一起使用的静态方法。通常更容易使用这个类，因为我们可以访问排序、搜索等方法。这个类在`java.util.Arrays`包中可用，所以在使用它之前，将这一行放在任何要使用它的文件的顶部：

```java
import java.util.Arrays;
```

在下面的代码中，我们可以看到如何使用`Arrays`类和一些我们可以使用的方法。所有的方法都在代码片段后面解释：

```java
import java.util.Arrays;
class ArraysExample {
public static void main(String[] args) {
double[] myArray = {0.0, 1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 9.0};
System.out.println(Arrays.toString (myArray)); 
Arrays.sort(myArray);
System.out.println(Arrays.toString (myArray));
Arrays.sort(myArray);
int index = Arrays.binarySearch(myArray,7.0);
System.out.println("Position of 7.0 is: " + index);
}
}
```

这是输出：

![图 6.7：ArraysExample 类的输出](img/C09581_Figure_06_07.jpg)

###### 图 6.7：ArraysExample 类的输出

在这个程序中，我们有`Arrays`类的三个示例用法。在第一个示例中，我们看到如何使用`Arrays.toString()`轻松打印数组的元素，而不需要我们之前使用的`for`循环。在第二个示例中，我们看到如何使用`Arrays.sort()`快速对数组进行排序。如果我们要自己实现这样一个方法，我们将使用更多的行，并且在过程中容易出现很多错误。

在最后一个示例中，我们对数组进行排序，然后使用`Arrays.binarySearch()`搜索 7.0，它使用一种称为**二分查找**的搜索算法。

#### 注意

`Arrays.sort()`使用一种称为双轴快速排序的算法来对大数组进行排序。对于较小的数组，它使用插入排序和归并排序的组合。最好相信`Arrays.sort()`针对每种用例进行了优化，而不是实现自己的排序算法。`Arrays.binarySearch()`使用一种称为二分查找的算法来查找数组中的项。它首先要求数组已排序，这就是为什么我们首先调用`Arrays.sort()`。二分查找递归地将排序后的数组分成两个相等的部分，直到无法再分割数组为止，此时该值就是答案。

### 插入排序

排序是计算机科学中算法的基本应用之一。插入排序是排序算法的一个经典示例，尽管它效率低下，但在查看数组和排序问题时是一个很好的起点。算法的步骤如下：

1.  取数组中的第一个元素，并假设它已经排序，因为它只有一个。

1.  选择数组中的第二个元素。将其与第一个元素进行比较。如果它大于第一个元素，则两个项目已经排序。如果它小于第一个元素，则交换两个元素，使它们排序。

1.  取第三个元素。将其与已排序子数组中的第二个元素进行比较。如果较小，则交换两者。然后再次将其与第一个元素进行比较。如果较小，则再次交换两者，使其成为第一个。这三个元素现在将被排序。

1.  取第四个元素并重复此过程，如果它小于其左邻居，则交换，否则保持在原位。

1.  对数组中的其余项目重复此过程。

1.  结果数组将被排序。

### 例子

取数组`[3, 5, 8, 1, 9]`：

1.  让我们取第一个元素并假设它已排序：`[3]`。

1.  取第二个元素，5。由于它大于 3，我们保持数组不变：`[3, 5]`。

1.  取第三个元素，8。它大于 5，所以这里也没有交换：`[3, 5, 8]`。

1.  取第四个元素，1。由于它小于 8，我们交换 8 和 1 得到：`[3, 5, 1, 8]`。

1.  由于 1 仍然小于 5，我们再次交换两者：`[3, 1, 5, 8]`。

1.  1 仍然小于 3。我们再次交换：`[1, 3, 5, 8]`。

1.  现在它是最小的。

1.  取最后一个元素，9。它大于 8，所以没有交换。

1.  整个数组现在已排序：`[1, 3, 5, 8, 9]`。

### 练习 18：实现插入排序

在这个练习中，我们将实现插入排序。

1.  创建一个名为`InsertionSort`的新类，并在这个类中创建`main`方法：

```java
public class InsertionSort {
public static void main(String[] args){
}
}
```

1.  在我们的`main`方法中，创建一个随机整数样本数组，并将其传递给我们的`sort`方法。使用以下数组，[1, 3, 354, 64, 364, 64, 3, 4, 74, 2, 46]：

```java
int[] arr = {1, 3,354,64,364,64, 3,4 ,74,2 , 46};
System.out.println("Array before sorting is as follows: ");
System.out.println(Arrays.toString(arr));
```

1.  在使用我们的数组调用`sort()`后，使用`foreach`循环在单行中打印排序后数组中的每个项目并用空格分隔：

```java
sort(arr);
        System.out.print("Array after sort looks as follows: ");
        for (int i : arr) {
            System.out.print(i + " ");
        }
    }
}
```

1.  创建一个名为`sort()`的公共静态方法，该方法接受一个整数数组并返回`void`。这是我们排序算法的方法：

```java
public static void sort(int[] arr){
}
```

在`sort`方法中，实现前面说明的算法。

1.  在`sort()`方法中将整数`num`定义为数组的长度：

```java
int num = arr.length;
```

1.  创建一个`for`循环，直到`i`达到数组的长度为止。在循环内，创建比较数字的算法：`k`将是由索引`i`定义的整数，`j`将是索引`i-1`。在`for`循环内添加一个`while`循环，根据以下条件交换`i`和`i-1`处的整数：`j`大于或等于`0`，并且索引`j`处的整数大于`k`：

```java
for (int i = 1; i < num; i++) {
        int k = arr[i];
        int j = i - 1;
    while (j>= 0 && arr[j] > k) {
        arr[j + 1] = arr[j];
        j = j - 1;
    }
    arr[j + 1] = k;
    }
}
```

完成的代码如下所示：

```java
import java.util.Arrays;
public class InsertionSort {
    public static void sort(int[] arr) {
        int num = arr.length;
        for (int i = 1; i < num; i++) {
            int k = arr[i];
            int j = i - 1;
        while (j>= 0 && arr[j] > k) {
            arr[j + 1] = arr[j];
            j = j - 1;
        }
        arr[j + 1] = k;
        }
    }
    public static void main(String[] args) {
        int[] arr = {1, 3, 354, 64, 364, 64, 3, 4, 74, 2, 46};
        System.out.println("Array before sorting is as follows: ");
        System.out.println(Arrays.toString(arr));
        sort(arr);
        System.out.print("Array after sort looks as follows: ");
        for (int i : arr) {
            System.out.print(i + " ");
        }
    }
}
```

输出如下：

![图 6.8：InsertionSort 类的输出](img/C09581_Figure_06_08.jpg)

###### 图 6.8：InsertionSort 类的输出

Java 使我们能够处理常用的数据结构，如列表、堆栈、队列和映射变得容易。它配备了 Java 集合框架，提供了易于使用的 API，用于处理这些数据结构。一个很好的例子是当我们想要对数组中的元素进行排序或者想要搜索数组中的特定元素时。我们可以应用于我们的集合的方法，只要它们符合集合框架的要求，而不是自己从头开始重写这些方法。集合框架的类可以保存任何类型的对象。

现在我们将看一下集合框架中的一个常见类，称为`ArrayList`。有时我们希望存储元素，但不确定我们期望的项目数量。我们需要一个数据结构，可以向其中添加任意数量的项目，并在需要时删除一些。到目前为止，我们看到的数组在创建时需要指定项目的数量。之后，除非创建一个全新的数组，否则无法更改该数组的大小。ArrayList 是一个动态列表，可以根据需要增长和缩小；它们是以初始大小创建的，当我们添加或删除一个项目时，大小会根据需要自动扩大或缩小。

### 创建 ArrayList 并添加元素

创建`ArrayList`时，您需要指定要存储的对象类型。数组列表仅支持引用类型（即对象）的存储，不支持原始类型。但是，由于 Java 提供了带有要添加的对象作为参数的`add()`方法。ArrayList 还有一个方法来获取列表中的项目数，称为`size()`。该方法返回一个整数，即列表中的项目数：

```java
import java.util.ArrayList;
public class Person {
public static void main(String[] args){
Person john=new Person();
//Initial size of 0
ArrayList<Integer> myArrayList = new ArrayList<>();
System.out.println("Size of myArrayList: "+myArrayList.size());

//Initial size of 5
ArrayList<Integer> myArrayList1 = new ArrayList<>(5);
myArrayList1.add(5);System.out.println("Size of myArrayList1: "+myArrayList1.size());
//List of Person objectsArrayList<Person> people = new ArrayList<>();
people.add(john);System.out.println("Size of people: "+people.size());
 }
}
```

输出如下：

![图 6.9：Person 类的输出](img/C09581_Figure_06_09.jpg)

###### 图 6.9：Person 类的输出

在第一个示例中，我们创建了一个大小为 0 的`myArrayList`，其中包含`Integer`类型的`ArrayList`。在第二个示例中，我们创建了一个大小为 5 的`Integer`类型的`ArrayList`。尽管初始大小为 5，但当我们添加更多项目时，列表将自动增加大小。在最后一个示例中，我们创建了一个`Person`对象的`ArrayList`。从这三个示例中，创建数组列表时应遵循以下规则：

1.  从`java.util`包中导入`ArrayList`类。

1.  在`<>`之间指定对象的数据类型。

1.  指定列表的名称。

1.  使用`new`关键字创建`ArrayList`的新实例。

以下是向 ArrayList 添加元素的一些方法：

```java
myArrayList.add( new Integer(1));
myArrayList1.add(1);
people.add(new Person());
```

在第一个示例中，我们创建一个新的`Integer`对象并将其添加到列表中。新对象将附加到列表的末尾。在第二行中，我们插入了 1，但由于`ArrayList`仅接受对象，JVM 将`Person`类并将其附加到列表中。我们可能还希望在同一类中将元素插入到特定索引而不是在列表末尾附加。在这里，我们指定要插入对象的索引和要插入的对象：

```java
myArrayList1.add(1, 8);
System.out.println("Elements of myArrayList1: " +myArrayList1.toString());
```

输出如下：

![图 6.10：添加元素到列表后的输出](img/C09581_Figure_06_10.jpg)

###### 图 6.10：添加元素到列表后的输出

#### 注意

在索引小于 0 或大于数组列表大小的位置插入对象将导致`IndexOutOfBoundsException`，并且您的程序将崩溃。在指定要插入的索引之前，始终检查列表的大小。

### 替换和删除元素

`ArrayList`还允许我们用新元素替换指定位置的元素。在上一个代码中添加以下内容并观察输出：

```java
myArrayList1.set(1, 3);
System.out.println("Elements of myArrayList1 after replacing the element: " +myArrayList1.toString());
```

这是输出：

![图 6.11：替换元素后的列表](img/C09581_Figure_06_11.jpg)

###### 图 6.11：替换元素后的列表

在这里，我们将在索引 2 处的元素替换为值为 3 的新`Integer`对象。如果我们尝试替换列表大小大于的索引或小于零的索引，此方法还会抛出`IndexOutOfBoundsException`。

如果您还希望删除单个元素或所有元素，ArrayList 也支持：

```java
//Remove at element at index 1
myArrayList1.remove(1);
System.out.println("Elements of myArrayList1 after removing the element: " +myArrayList1.toString());
//Remove all the elements in the list
myArrayList1.clear();
System.out.println("Elements of myArrayList1 after clearing the list: " +myArrayList1.toString());
```

这是输出：

![图 6.12：清除所有元素后的列表](img/C09581_Figure_06_12.jpg)

###### 图 6.12：清除所有元素后的列表

要获取特定索引处的元素，请使用`get()`方法，传入索引。该方法返回一个对象：

```java
myArrayList1.add(10);
Integer one = myArrayList1.get(0);
System.out.println("Element at given index: "+one);
```

输出如下：

![图 6.13：给定索引处元素的输出](img/C09581_Figure_06_13.jpg)

###### 图 6.13：给定索引处元素的输出

如果传递的索引无效，此方法还会抛出`IndexOutOfBoundsException`。为了避免异常，始终先检查列表的大小。考虑以下示例：

```java
Integer two = myArrayList1.get(1);
```

![图 6.14：IndexOutOfBounds 异常消息](img/C09581_Figure_06_14.jpg)

###### 图 6.14：IndexOutOfBounds 异常消息

### 练习 19：在数组中添加、删除和替换元素

数组是存储信息的基本但有用的方式。在这个练习中，我们将看看如何在学生名单中添加和删除元素：

1.  导入`java.util`的`ArrayList`和`List`类：

```java
import java.util.ArrayList;
import java.util.List;
```

1.  创建一个`public`类和`main`方法：

```java
public class StudentList {
    public static void main(String[] args) {
```

1.  将学生`List`定义为包含字符串的新 ArrayList：

```java
List<String> students = new ArrayList<>();
```

1.  添加四个学生的名字：

```java
students.add("Diana");
students.add("Florence");
students.add("Mary");
students.add("Betty");
```

1.  打印数组并删除最后一个学生：

```java
System.out.println(students);
students.remove("Betty");
```

1.  打印数组：

```java
System.out.println(students);
```

1.  替换第一个学生（在索引 0 处）：

```java
students.set(0, "Jean");
```

1.  打印数组：

```java
System.out.println(students);  
}
}
```

输出如下：

![图 6.15：StudentList 类的输出](img/C09581_Figure_06_15.jpg)

###### 图 6.15：StudentList 类的输出

### 迭代器

集合框架还提供了迭代器，我们可以使用它们来循环遍历`ArrayList`的元素。迭代器就像是列表中项目的指针。我们可以使用迭代器来查看列表中是否有下一个元素，然后检索它。将迭代器视为集合框架的循环。我们可以使用`array.iterator()`对象和`hasNext()`来循环遍历数组。

### 练习 20：遍历 ArrayList

在这个练习中，我们将创建一个世界上城市的`ArrayList`，并使用迭代器逐个打印整个`ArrayList`中的城市：

1.  导入 ArrayList 和 Iterator 包：

```java
import java.util.ArrayList;
import java.util.Iterator;
```

1.  创建一个`public`类和`main`方法：

```java
public class Cities {
public static void main(String[] args){
```

1.  创建一个新数组并添加城市名称：

```java
ArrayList<String> cities = new ArrayList<>();
cities.add( "London");
cities.add( "New York");
cities.add( "Tokyo");
cities.add( "Nairobi");
cities.add( "Sydney");
```

1.  定义一个包含字符串的迭代器：

```java
Iterator<String> citiesIterator = cities.iterator(); 
```

1.  使用`hasNext()`循环迭代器，使用`next()`打印每个城市：

```java
while (citiesIterator.hasNext()){
String city = citiesIterator.next();
System.out.println(city);
}
}
}
```

输出如下：

![图 6.16：Cities 类的输出](img/C09581_Figure_06_16.jpg)

###### 图 6.16：Cities 类的输出

在这个类中，我们创建了一个包含字符串的新 ArrayList。然后我们插入了一些名字，并创建了一个名为`citiesIterator`的迭代器。集合框架中的类支持`iterator()`方法，该方法返回一个用于集合的迭代器。迭代器有`hasNext()`方法，如果在我们当前位置之后列表中还有另一个元素，则返回 true，并且`next()`方法返回下一个对象。`next()`返回一个对象实例，然后将其隐式向下转换为字符串，因为我们声明`citiesIterator`来保存字符串类型：`Iterator<String> citiesIterator`。

![图 6.17：next()和 hasNext()的工作方式](img/C09581_Figure_06_17.jpg)

###### 图 6.17：next()和 hasNext()的工作方式

除了使用迭代器进行循环，我们还可以使用普通的`for`循环来实现相同的目标：

```java
for (int i = 0; i < cities.size(); i++){
String name = cities.get(i);
System.out .println(name);
}
```

输出如下：

![图 6.18：使用 for 循环输出 Cities 类的输出](img/C09581_Figure_06_18.jpg)

###### 图 6.18：使用 for 循环输出 Cities 类的输出

在这里，我们使用`size()`方法来检查列表的大小，并使用`get()`来检索给定索引处的元素。无需将对象转换为字符串，因为 Java 已经知道我们正在处理一个字符串列表。

同样，我们可以使用更简洁的`for-each`循环，但实现相同的目标：

```java
for (String city : cities) {
System.out.println(city);
}
```

输出如下：

![图 6.19：使用 for-each 循环输出 Cities 类的输出](img/C09581_Figure_06_18.jpg)

###### 图 6.19：使用 for-each 循环输出 Cities 类的输出

### 活动 23：使用 ArrayList

我们有几个学生希望在我们的程序中跟踪。但是，我们目前不确定确切的数量，但预计随着越来越多的学生使用我们的程序，数量会发生变化。我们还希望能够循环遍历我们的学生并打印他们的名字。我们将创建一个对象的 ArrayList，并使用迭代器来循环遍历 ArrayList：

这些步骤将帮助您完成该活动：

1.  从`java.util`导入`ArrayList`和`Iterator`。

1.  创建一个名为`StudentsArray`的新类。

1.  在`main`方法中，定义一个`Student`对象的`ArrayList`。插入四个学生实例，用我们之前创建的不同类型的构造函数实例化。

1.  为您的列表创建一个迭代器，并打印每个学生的姓名。

1.  最后，从`ArrayList`中清除所有对象。

输出如下：

![图 6.20：StudentsArray 类的输出](img/C09581_Figure_06_20.jpg)

###### 图 6.20：StudentsArray 类的输出

#### 注意

ArrayList 是一个重要的类，你会发现自己在日常生活中经常使用它。这个类有更多的功能，这里没有涵盖，比如交换两个元素，对项目进行排序等。

#### 注意

此活动的解决方案可以在第 338 页找到。

## 字符串

Java 有字符串数据类型，用于表示一系列字符。字符串是 Java 中的基本数据类型之一，你几乎在所有程序中都会遇到它。

字符串只是一系列字符。"Hello World"，"London"和"Toyota"都是 Java 中字符串的例子。字符串在 Java 中是对象而不是原始类型。它们是不可变的，也就是说，一旦它们被创建，就不能被修改。因此，我们将在接下来的部分中考虑的方法只会创建包含操作结果的新字符串对象，而不会修改原始字符串对象。

### 创建一个字符串

我们使用双引号表示字符串，而单引号表示字符：

```java
public class StringsDemo {
    public static void main(String[] args) {
        String hello="Hello World";
        System.out.println(hello);
    }
}
```

输出如下：

![图 6.21：StringsDemo 类的输出](img/C09581_Figure_06_21.jpg)

###### 图 6.21：StringsDemo 类的输出

`hello`对象现在是一个字符串，是不可变的。我们可以在字符串中使用分隔符，比如`\n`表示换行，`\t`表示制表符，或者`\r`表示回车：

```java
String data = '\t'+ "Hello"+ '\n'+" World";
System.out.println(data);
```

输出如下：

![图 6.22：使用分隔符的输出](img/C09581_Figure_06_22.jpg)

###### 图 6.22：使用分隔符的输出

我们在`Hello`之前有一个制表符，然后在`World`之前有一个换行符，这会在下一行打印`World`。

### 连接

我们可以将多个字符串文字组合在一起，这个过程通常被称为连接。我们使用`+`符号来连接两个字符串，如下所示：

```java
String str = "Hello " + "World";
System.out.println(str);
```

输出如下：

```java
Hello World
```

当我们想要替换在运行时计算的值时，通常使用连接。代码如下所示：

```java
String userName = getUserName(); // get the username from an external location like database or input field
System.out.println( " Welcome " + userName);
```

在第一行，我们从一个我们在这里没有定义的方法中得到了`userName`。然后我们打印出一个欢迎消息，用`userName`替换了我们之前得到的`userName`。

当我们想要表示跨越多行的字符串时，连接也很重要：

```java
String quote = "I have a dream that " +
"all Java programmers will " +
"one day be free from " +
"all computer bugs!";
System.out.println(quote);
```

这是输出：

![图 6.23：连接的字符串](img/C09581_Figure_06_23.jpg)

###### 图 6.23：连接的字符串

除了`+`符号，Java 还提供了`concat()`方法来连接两个字符串文字：

```java
String wiseSaying = "Java programmers are " . concat("wise and knowledgeable").concat("." );
System.out.println(wiseSaying);
```

这是输出：

![图 6.24：使用 concat()连接的字符串](img/C09581_Figure_06_24.jpg)

###### 图 6.24：使用 concat()连接的字符串

### 字符串长度和字符

字符串提供了**length()**方法来获取字符串中的字符数。字符数是所有有效的 java 字符的计数，包括换行符、空格和制表符：

```java
String saying = "To be or not to be, that is the question."
int num = saying.length();
System.out.println(num);
```

这是输出：

```java
4
```

要访问给定索引处的字符，请使用`charAt(i)`。这个方法接受你想要的字符的索引并返回一个 char：

```java
char c = quote.charAt(7);
System.out.println(c);
```

这是输出：

```java
r
```

使用大于字符串中字符数或负数的索引调用`charAt(i)`将导致您的程序崩溃，并出现`StringIndexOutOfBoundsException`异常：

```java
char d = wiseSaying.charAt(-3);
```

![图 6.25：StringIndexOutOfBoundsException message](img/C09581_Figure_06_25.jpg)

###### 图 6.25：`StringIndexOutOfBoundsException message`

我们还可以使用`getChars()`方法将字符串转换为字符数组。此方法返回一个我们可以使用的字符数组。我们可以转换整个字符串或字符串的一部分：

```java
char[] chars = new char [quote.length()]; 
quote.getChars(0, quote.length(), chars, 0); 
System.out.println(Arrays.toString (chars));
```

输出如下：

![图 6.26：字符数组](img/C09581_Figure_06_26.jpg)

###### 图 6.26：字符数组

### 活动 24：输入一个字符串并输出其长度和作为数组

为了检查输入到系统中的名称是否过长，我们可以使用之前提到的一些功能来计算名称的长度。在这个活动中，您将编写一个程序，将输入一个名称，然后导出名称的长度和第一个字母。

步骤如下：

1.  导入`java.util.Scanner`包。

1.  创建一个名为`nameTell`的公共类和一个`main`方法。

1.  使用`Scanner`和`nextLine`在提示"`输入您的姓名：`"处输入一个字符串。

1.  计算字符串的长度并找到第一个字符。

1.  打印输出如下：

```java
Your name has 10 letters including spaces.
The first letter is: J
```

输出将如下所示：

![图 6.27：NameTell 类的输出](img/C09581_Figure_06_27.jpg)

###### 图 6.27：NameTell 类的输出

#### 注意

此活动的解决方案可以在第 340 页找到。

### 活动 25：计算器从输入中读取

将所有计算器逻辑封装起来，我们将编写一个命令行计算器，您可以在其中给出运算符、两个操作数，它将显示结果。这样的命令行应用程序以一个永不结束的 while 循环开始。然后从用户那里读取输入，并根据输入做出决定。

对于这个活动，你将编写一个应用程序，只有两个选择：退出或执行操作。如果用户输入`Q`（或`q`），应用程序将退出循环并结束。其他任何内容都将被视为操作。您将使用`Operators.findOperator`方法来查找运算符，然后从用户那里请求更多输入。每个输入都将被转换为双精度（使用`Double.parse`或`Scanner.nextDouble`）。使用找到的运算符对它们进行操作，并将结果打印到控制台上。

由于无限循环，应用程序将重新开始，要求另一个用户操作。

要完成这个活动，您需要：

1.  创建一个名为`CommandLineCalculator`的新类，其中包含一个`main`方法。

1.  使用无限循环使应用程序保持运行，直到用户要求退出。

1.  收集用户输入以决定要执行的操作。如果操作是`Q`或`q`，退出循环。

1.  如果操作是其他任何内容，请找到一个运算符，并请求另外两个输入，它们将是操作数，将它们转换为双精度。

1.  在找到的运算符上调用`operate`方法，并将结果打印到控制台上。

#### 注意

此活动的解决方案可以在第 341 页找到。

### 转换

有时我们可能希望将给定类型转换为字符串，以便我们可以打印它出来，或者我们可能希望将字符串转换为给定类型。例如，当我们希望将字符串"`100`"转换为整数`100`，或者将整数`100`转换为字符串"`100`"时。

使用`+`运算符将原始数据类型连接到字符串将返回该项的字符串表示。

例如，这是如何在整数和字符串之间转换的：

```java
String str1 = "100";
Integer number = Integer.parseInt(str1);
String str2 = number.toString();
System.out.println(str2);
```

输出如下：

```java
100
```

这里我们使用`parseInt()`方法获取字符串的整数值，然后使用`toString()`方法将整数转换回字符串。

要将整数转换为字符串，我们将其与空字符串""连接：

```java
int a = 100;
String str = "" + a;
```

输出如下：

```java
100
```

#### 注意

Java 中的每个对象都有一个字符串表示。Java 提供了`Object`超类中的`toString()`方法，我们可以在我们的类中重写它，以提供我们类的字符串表示。当我们想以字符串格式打印我们的类时，字符串表示很重要。

### 比较字符串和字符串的部分

`String`类支持许多用于比较字符串和字符串部分的方法。

比较两个字符串是否相等：

```java
String data= "Hello";
String data1 = "Hello";
if (data == data1){
System. out .println("Equal");
}else{
System. out .println("Not Equal");
}
```

输出如下：

```java
Equal
```

如果这个字符串以给定的子字符串结尾或开始，则返回`true`：

```java
boolean value= data.endsWith( "ne");
System.out.println(value);
boolean value1 = data.startsWith("He");
System.out.println(value);
```

输出如下：

```java
False
True
```

### StringBuilder

我们已经说明了字符串是不可变的，也就是说，一旦它们被声明，就不能被修改。然而，有时我们希望修改一个字符串。在这种情况下，我们使用`StringBuilder`类。`StringBuilder`就像普通字符串一样，只是它是可修改的。`StringBuilder`还提供了额外的方法，比如`capacity()`，它返回为其分配的容量，以及`reverse()`，它颠倒其中的字符。`StringBuilder`还支持`String`类中的相同方法，比如`length()`和`toString()`。

### 练习 21：使用 StringBuilder

这个练习将追加三个字符串以创建一个字符串，然后打印出它的长度、容量和反转：

1.  创建一个名为`StringBuilderExample`的公共类，然后创建一个`main`方法：

```java
import java.lang.StringBuilder;
public class StringBuilder {
public static void main(String[] args) { 
```

1.  创建一个新的`StringBuilder()`对象，命名为`stringbuilder`：

```java
StringBuilder stringBuilder = new StringBuilder(); 
```

1.  追加三个短语：

```java
stringBuilder.append( "Java programmers "); 
stringBuilder.append( "are wise " ); 
stringBuilder.append( "and knowledgeable");
```

1.  使用`\n`作为换行打印出字符串：

```java
System.out.println("The string is \n" + stringBuilder.toString()); 
```

1.  找到字符串的长度并打印出来：

```java
int len = stringBuilder.length();
System.out.println("The length of the string is: " + len);
```

1.  找到字符串的容量并打印出来：

```java
int capacity = stringBuilder.capacity(); 
System.out.println("The capacity of the string is: " + capacity);
```

1.  颠倒字符串并使用换行打印出来：

```java
stringBuilder.reverse(); 
      System.out.println("The string reversed is: \n" + stringBuilder);
}
}
```

以下是输出：

![图 6.28：StringBuilder 类的输出](img/C09581_Figure_06_28.jpg)

###### 图 6.28：StringBuilder 类的输出

在这个练习中，我们使用默认容量为 16 创建了一个`StringBuilder`的新实例。然后我们插入了一些字符串，然后打印出整个字符串。我们还通过`length()`获取了构建器中的字符数。然后我们得到了`StringBuilder`的容量。容量是为`StringBuilder`分配的字符数。它通常高于或等于构建器的长度。最后，我们颠倒了构建器中的所有字符，然后打印出来。在最后的打印输出中，我们没有使用`stringBuilder.toString()`，因为 Java 会隐式地为我们执行这个操作。

### 活动 26：从字符串中删除重复字符

为了创建安全的密码，我们决定需要创建不包含重复字符的字符串行。在这个活动中，您将创建一个程序，它接受一个字符串，删除任何重复的字符，然后打印出结果。

一种方法是遍历字符串的所有字符，对于每个字符，再次遍历字符串，检查字符是否已经存在。如果找到重复的字符，立即将其删除。这种算法是一种蛮力方法，不是在运行时间方面最好的方法。事实上，它的运行时间是指数级的。

这些步骤将帮助您完成这个活动：

1.  创建一个名为`Unique`的新类，并在其中创建一个`main`方法。现在先留空。

1.  创建一个名为`removeDups`的新方法，它接受并返回一个字符串。这就是我们的算法所在的地方。这个方法应该是`public`和`static`的。

1.  在方法内部，检查字符串是否为 null，空或长度为 1。如果这些情况中有任何一个为真，则只需返回原始字符串，因为不需要进行检查。

1.  创建一个名为`result`的空字符串。这将是要返回的唯一字符串。

1.  创建一个`for`循环，从 0 到传入方法的字符串的长度。

1.  在`for`循环内，获取字符串当前索引处的字符。将变量命名为`c`。

1.  还要创建一个名为`isDuplicate`的布尔变量，并将其初始化为`false`。当我们遇到重复时，我们将把它改为`true`。

1.  创建另一个嵌套的`for`循环，从 0 到结果的`length()`。

1.  在`for`循环内，还要获取结果当前索引处的字符。将其命名为`d`。

1.  比较`c`和`d`。如果它们相等，则将`isDuplicate`设置为 true 并`break`。

1.  关闭内部的`for`循环并进入第一个`for`循环。

1.  检查`isDuplicate`是否为`false`。如果是，则将`c`追加到结果中。

1.  退出第一个`for`循环并返回结果。这就完成了我们的算法。

1.  返回到我们空的`main`方法。创建以下几个测试字符串：

```java
aaaaaaa 
aaabbbbb
abcdefgh
Ju780iu6G768
```

1.  将字符串传递给我们的方法，并打印出方法返回的结果。

1.  检查结果。返回的字符串中应该删除重复的字符。

输出应该是这样的：

![图 6.29：Unique 类的预期输出](img/C09581_Figure_06_29.jpg)

###### 图 6.29：Unique 类的预期输出

#### 注意

此活动的解决方案可在第 342 页找到。

## 总结

这节课将我们带到面向对象编程核心原则讨论的尽头。在这节课中，我们已经看过了数据类型、算法和字符串。

我们已经看到了数组是相同类型项目的有序集合。数组用方括号`[ ]`声明，它们的大小不能被修改。Java 提供了集合框架中的`Arrays`类，它有额外的方法可以用在数组上。

我们还看到了`StringBuilder`类的概念，它基本上是一个可修改的字符串。`stringbuilder`有`length`和`capacity`函数。
