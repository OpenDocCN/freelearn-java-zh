# *第三章*

# 控制流

## 学习目标

通过本课程结束时，你将能够：

+   使用 Java 中的`if`和`else`语句控制执行流程

+   使用 Java 中的 switch case 语句检查多个条件

+   利用 Java 中的循环结构编写简洁的代码来执行重复的操作

## 介绍

到目前为止，我们已经看过由 Java 编译器按顺序执行的一系列语句组成的程序。然而，在某些情况下，我们可能需要根据程序的当前状态执行操作。

考虑一下安装在 ATM 机中的软件的例子-它执行一系列操作，也就是说，当用户输入的 PIN 正确时，它允许交易发生。然而，当输入的 PIN 不正确时，软件执行另一组操作，也就是告知用户 PIN 不匹配，并要求用户重新输入 PIN。你会发现，几乎所有现实世界的程序中都存在依赖于值或阶段的这种逻辑结构。

也有时候，可能需要重复执行特定任务，也就是说，在特定时间段内，特定次数，或者直到满足条件为止。延续我们关于 ATM 机的例子，如果输入错误密码的次数超过三次，那么卡就会被锁定。

这些逻辑结构作为构建复杂 Java 程序的基本构件。本课程将深入探讨这些基本构件，可以分为以下两类：

+   条件语句

+   循环语句

## 条件语句

条件语句用于根据某些条件控制 Java 编译器的执行流程。这意味着我们根据某个值或程序的状态做出选择。Java 中可用的条件语句如下：

+   `if`语句

+   `if-else`语句

+   `else-if`语句

+   `switch`语句

### if 语句

if 语句测试一个条件，当条件为真时，执行 if 块中包含的代码。如果条件不为真，则跳过块中的代码，执行从块后的行继续执行。

`if`语句的语法如下：

```java
if (condition) {
//actions to be performed when the condition is true
}
```

考虑以下例子：

```java
int a = 9;
if (a < 10){
System.out.println("a is less than 10");
}
```

由于条件`a<10`为真，打印语句被执行。

我们也可以在`if`条件中检查多个值。考虑以下例子：

```java
if ((age > 50) && (age <= 70) && (age != 60)) {
System.out.println("age is above 50 but at most 70 excluding 60");
}
```

上述代码片段检查`age`的值是否超过 50，但最多为 70，不包括 60。

当`if`块中的语句只有一行时，我们不需要包括括号：

```java
if (color == 'Maroon' || color == 'Pink')
System.out.println("It is a shade of Red");
```

### else 语句

对于某些情况，如果`if`条件失败，我们需要执行不同的代码块。为此，我们可以使用`else`子句。这是可选的。

`if else`语句的语法如下：

```java
if (condition) {
//actions to be performed when the condition is true
}
else {
//actions to be performed when the condition is false
}
```

### 练习 6：实现简单的 if-else 语句

在这个练习中，我们将创建一个程序，根据空座位的数量来检查是否可以预订公交车票。完成以下步骤来实现：

1.  右键单击`src`文件夹，然后选择**新建** | **类**。

1.  输入`Booking`作为类名，然后点击**OK**。

1.  设置`main`方法：

```java
public class Booking{
public static void main(String[] args){
}
}
```

1.  初始化两个变量，一个用于空座位数量，另一个用于请求的票数：

```java
int seats = 3; // number of empty seats
int req_ticket = 4; // Request for tickets
```

1.  使用`if`条件检查所请求的票数是否小于或等于可用的空座位，并打印适当的消息：

```java
if( (req_ticket == seats) || (req_ticket < seats) ) {
     System.out.print("This booing can be accepted");
     }else
         System.out.print("This booking is rejected");
```

1.  运行程序。

你应该得到以下输出：

```java
This booking is rejected
```

### else-if 语句

当我们希望在评估`else`子句之前比较多个条件时，可以使用`else if`语句。

`else if`语句的语法如下：

```java
if (condition 1) {
//actions to be performed when condition 1 is true
}
else if (Condition 2) {
//actions to be performed when condition 2 is true
}
else if (Condition 3) {
//actions to be performed when condition 3 is true
}
…
…
else if (Condition n) {
//actions to be performed when condition n is true
}
else {
//actions to be performed when the condition is false
}
```

### 练习 7：实现 else-if 语句

我们正在构建一个电子商务应用程序，根据卖家和买家之间的距离计算交付费用。买家在我们的网站上购买物品并输入交付地址。根据距离，我们计算交付费用并显示给用户。在这个练习中，我们得到了以下表格，并需要编写一个程序来向用户输出交付费用：

![表 3.1：显示距离及其对应费用的表](img/C09581_Table_03_01.jpg)

###### 表 3.1：显示距离及其对应费用的表

要做到这一点，请执行以下步骤：

1.  右键单击`src`文件夹，然后选择**新建** | **类**。

1.  输入`DeliveryFee`作为类名，然后单击**OK**。

1.  打开创建的类，然后创建主方法：

```java
public class DeliveryFee{
public static void main(String[] args){
}
}
```

1.  在`main`方法中，创建两个整数变量，一个称为`distance`，另一个称为`fee`。这两个变量将分别保存`distance`和交付费用。将`distance`初始化为 10，`fee`初始化为零：

```java
int distance = 10;
int fee = 0;
```

1.  创建一个`if`块来检查表中的第一个条件：

```java
if (distance > 0 && distance < 5){
   fee = 2;
}
```

这个`if`语句检查`distance`是否大于 0 但小于 5，并将交付`fee`设置为 2 美元。

1.  添加一个`else if`语句来检查表中的第二个条件，并将`fee`设置为 5 美元：

```java
else if (distance >= 5 && distance < 15){
   fee = 5;
}
```

1.  添加两个`else if`语句来检查表中的第三和第四个条件，如下面的代码所示：

```java
else if (distance >= 15 && distance < 25){
   fee = 10;
}else if (distance >= 25 && distance < 50){
   fee = 15;
}
```

1.  最后，添加一个`else`语句来匹配表中的最后一个条件，并设置适当的交付`fee`：

```java
else {
   fee = 20;
}
```

1.  打印出`fee`的值：

```java
System.out.println("Delivery Fee: " + fee);
```

1.  运行程序并观察输出：

```java
Delivery Fee: 5
```

### 嵌套的 if 语句

我们可以在其他`if`语句内部使用`if`语句。这种结构称为嵌套的`if`语句。我们首先评估外部条件，如果成功，然后评估第二个内部`if`语句，依此类推，直到所有`if`语句都完成：

```java
if (age > 20){

   if (height > 170){

       if (weight > 60){
           System.out.println("Welcome");
       }    
   }
}
```

我们可以嵌套任意多的语句，并且编译器将从顶部向下评估它们。

### switch case 语句

`switch case`语句是在相同的值进行相等比较时，执行多个`if` `else`语句的更简单更简洁的方法。以下是一个快速比较：

传统的`else if`语句如下所示：

```java
if(age == 10){
   discount = 300;
} else if (age == 20){
   discount = 200;
} else if (age == 30){
   discount = 100;
} else {
   discount = 50;
}
```

然而，使用`switch case`语句实现相同逻辑时，将如下所示：

```java
switch (age){
   case 10:
       discount = 300;
   case 20:
       discount = 200;
   case 30:
       discount = 100;
   default:
       discount = 50;
}
```

请注意，这段代码更易读。

要使用`switch`语句，首先需要使用关键字`switch`声明它，后跟括号中的条件。`case`语句用于检查这些条件。它们按顺序检查。

编译器将检查`age`的值与所有`case`进行匹配，如果找到匹配，那么将执行该`case`中的代码以及其后的所有`case`。例如，如果我们的`age`等于 10，将匹配第一个`case`，然后第二个`case`，第三个`case`和`default` `case`。如果所有其他情况都不匹配，则执行`default` `case`。例如，如果`age`不是 10、20 或 30，则折扣将设置为 50。它可以被解释为`if-else`语句中的`else`子句。`default` `case`是可选的，可以省略。

如果`age`等于 30，那么第三个`case`将被匹配并执行。由于`default` `case`是可选的，我们可以将其省略，执行将在第三个`case`之后结束。

大多数情况下，我们真正希望的是执行结束于匹配的`case`。我们希望如果匹配了第一个`case`，那么就执行该`case`中的代码，并忽略其余的情况。为了实现这一点，我们使用`break`语句告诉编译器继续在`switch`语句之外执行。以下是带有`break`语句的相同`switch case`：

```java
switch (age){
   case 10:
       discount = 300;
       break;
   case 20:
       discount = 200;
       break;
   case 30:
       discount = 100;
       break;
   default:
       discount = 50;
}
```

因为`default`是最后一个`case`，所以我们可以安全地忽略`break`语句，因为执行将在那里结束。

#### 注意：

在未来，另一个程序员添加额外的情况时，始终添加一个 break 语句是一个好的设计。

### 活动 6：使用条件控制执行流程

工厂每小时支付工人 10 美元。标准工作日是 8 小时，但工厂为额外的工作时间提供额外的补偿。它遵循的政策是计算工资如下：

+   如果一个人工作少于 8 小时-每小时* $10

+   如果一个人工作超过 8 小时但少于 12 小时-额外 20%的工资

+   超过 12 小时-额外的一天工资被记入

创建一个程序，根据工作小时数计算并显示工人赚取的工资。

为了满足这个要求，执行以下步骤：

1.  初始化两个变量和工作小时和工资的值。

1.  在`if`条件中，检查工人的工作小时是否低于所需小时。如果条件成立，则工资应为（工作小时* 10）。

1.  使用`else if`语句检查工作小时是否介于 8 小时和 12 小时之间。如果是这样，那么工资应该按照每小时 10 美元计算前 8 小时，剩下的小时应该按照每小时 12 美元计算。

1.  使用`else`块为默认的每天$160（额外的一天工资）。

1.  执行程序以观察输出。

#### 注意

此活动的解决方案可以在第 308 页找到。

### 活动 7：开发温度系统

在 Java 中编写一个程序，根据温度显示简单的消息。温度概括为以下三个部分：

+   高：在这种情况下，建议用户使用防晒霜

+   低：在这种情况下，建议用户穿外套

+   潮湿：在这种情况下，建议用户打开窗户

要做到这一点，执行以下步骤：

1.  声明两个字符串，`temp`和`weatherWarning`。

1.  用`High`、`Low`或`Humid`初始化`temp`。

1.  创建一个检查`temp`不同情况的 switch 语句。

1.  将变量`weatherWarning`初始化为每种温度情况的适当消息（`High`、`Low`、`Humid`）。

1.  在默认情况下，将`weatherWarning`初始化为“天气看起来不错。出去散步”。

1.  完成 switch 结构后，打印`weatherWarning`的值。

1.  运行程序以查看输出，应该类似于：

```java
Its cold outside, do not forget your coat.
```

#### 注意

此活动的解决方案可以在第 309 页找到。

## 循环结构

循环结构用于在满足条件的情况下多次执行特定操作。它们通常用于对列表项执行特定操作。例如，当我们想要找到从 1 到 100 所有数字的总和时。Java 支持以下循环结构：

+   `for`循环

+   `for each`循环

+   `while`循环

+   `do while`循环

### for 循环

`for`循环的语法如下：

```java
for( initialization ; condition ; expression) {
    //statements
}
```

初始化语句在`for`循环开始执行时执行。可以有多个表达式，用逗号分隔。所有表达式必须是相同类型的：

```java
for( int i  = 0, j = 0; i <= 9; i++)
```

`for`循环的条件部分必须评估为 true 或 false。如果没有表达式，则条件默认为 true。

在语句的每次迭代后执行表达式部分，只要条件为真。可以有多个用逗号分隔的表达式。

#### 注意

表达式必须是有效的 Java 表达式，即可以以分号终止的表达式。

以下是`for`循环的工作原理：

1.  首先，初始化被评估。

1.  然后，检查条件。如果条件为真，则执行`for`块中包含的语句。

1.  在执行语句后，执行表达式，然后再次检查条件。

1.  如果仍然不是 false，则再次执行语句，然后执行表达式，再次评估条件。

1.  这将重复，直到条件评估为 false。

1.  当条件求值为 false 时，`for`循环完成，循环后的代码部分被执行。

### 练习 8：实现一个简单的 for 循环

为了打印所有递增和递减的个位数，执行以下步骤：

1.  右键单击`src`文件夹，选择**新建** | **类**。

1.  输入`Looping`作为类名，然后点击**OK**。

1.  设置`main`方法：

```java
public class Looping
{
   public static void main(String[] args) {
   }
}
```

1.  实现一个`for`循环，初始化一个变量`i`为零，一个条件使得值保持在 10 以下，并且`i`应该在每次迭代中递增一个：

```java
System.out.println("Increasing order");
for( int i  = 0; i <= 9; i++)
System.out.println(i);
```

1.  实现另一个`for`循环，初始化一个变量`k`为 9，一个条件使得值保持在 0 以上，并且`k`应该在每次迭代中减少一个：

```java
System.out.println("Decreasing order");
for( int k  = 9; k >= 0; k--)
System.out.println(k);
```

输出：

```java
Increasing order 
0
1
2
3
4
5
6
7
8
9
Decreasing order
9
8
7
6
5
4
3
2
1
0
```

### 活动 8：实现 for 循环

约翰是一个桃农，他从树上摘桃子，把它们放进水果箱里然后运输。如果一个水果箱里装满了 20 个桃子，他就可以运输。如果他的桃子少于 20 个，他就必须摘更多的桃子，这样他就可以装满一个水果箱，然后运输。

我们想通过编写一个自动化软件来帮助约翰启动填充和运输箱子。我们从约翰那里得到桃子的数量，然后为每组 20 个桃子打印一条消息，并说明到目前为止已经运输了多少桃子。例如，对于第三个箱子，我们打印“到目前为止已经运输了 60 个桃子”。我们想用`for`循环来实现这一点。我们不需要担心剩下的桃子。为了实现这一点，执行以下步骤：

1.  创建一个新的类，输入`PeachBoxCounter`作为类名

1.  导入`java.util.Scanner`包：

1.  在`main()`中使用`System.out.print`询问用户`numberOfPeaches`。

1.  编写一个`for`循环，计算到目前为止运输的桃子数量。这从零开始，每次增加 20，直到剩下的桃子少于 20。

1.  在`for`循环中，打印到目前为止运输的桃子数量。

1.  运行主程序。

输出应该类似于：

```java
Enter the number of peaches picked: 42
shipped 0 peaches so far
shipped 20 peaches so far
shipped 40 peaches so far  
```

#### 注意

这个活动的解决方案可以在 310 页找到。

`for`循环的所有三个部分都是可选的。这意味着行`for( ; ;) `将提供任何错误。它只提供一个邀请循环。

这个`for`循环什么也不做，也不会终止。在`for`循环声明的变量在`for`循环的语句中是可用的。例如，在我们的第一个例子中，我们从语句部分打印了`i`的值，因为变量`i`是在`for`循环中声明的。然而，这个变量在`for`循环后不可用，并且可以自由声明。但是不能在`for`循环内再次声明：

```java
for (int i = 0; i <= 9; i++)
   int i  = 10;            //Error, i is already declared
```

`for`循环也可以有括号括住的语句，如果我们有多于一个语句。这就像我们之前讨论的`if-else`语句一样。如果只有一个语句，那么我们不需要括号。当语句多于一个时，它们需要被括在大括号内。在下面的例子中，我们打印出`i`和`j`的值：

```java
for (int i = 0, j = 0; i <= 9; i++, j++) {
   System.out.println(i);
   System.out.println(j);
}
```

#### 注意

表达式必须是有效的 Java 表达式，即可以用分号终止的表达式。

`break`语句可以用来中断`for`循环并跳出循环。它将执行超出`for`循环的范围。

例如，如果`i`等于 5，我们可能希望终止我们之前创建的`for`循环：

```java
for (int i = 0; i <= 9; i++){

   if (i == 5)
       break;
   System.out.println(i);
}
```

输出：

```java
0
1
2
3
4
```

前面的`for`循环从 0、1、2 和 3 迭代，终止于 4。这是因为在满足条件`i`即 5 之后，执行了`break`语句，这结束了`for`循环，循环后的语句不会被执行。执行继续在循环外部。

`continue`语句用于告诉循环跳过它后面的所有其他语句，并继续执行下一次迭代：

```java
for (int i = 0; i <= 9; i++){
   if (i == 5)
       continue;
   System.out.println(i);
}
```

输出：

```java
0
1
2
3
4
6
7
8
9
```

数字 5 没有被打印出来，因为一旦遇到`continue`语句，它后面的语句都会被忽略，并且开始下一次迭代。当处理多个项目时，`continue`语句可能会很有用，因为它可以跳过一些异常。

### 嵌套 for 循环

循环内的一组语句可以是另一个循环。这样的结构称为嵌套循环：

```java
public class Nested{
     public static void main(String []args){
        for(int i = 1; i <= 3; i++) {
   //Nested loop
   for(int j = 1; j <= 3; j++) {
       System.out.print(i + "" + j);
       System.out.print("\t");
   }
   System.out.println();
}
     }
}
```

输出：

```java
11    12    13
21    22    23
31    32    33
```

对于每个`i`的单个循环，我们循环`j`三次。您可以将这些`for`循环理解为如下：

重复`i`三次，对于每次重复，重复`j`三次。这样，我们总共有 9 次`j`的迭代。对于每次`j`的迭代，我们打印出`i`和`j`的值。

### 练习 9：实现嵌套 for 循环

我们在这个练习中的目标是打印一个有七行的星号金字塔，如下所示：

![](img/C09581_Figure_03_01.jpg)

###### 图 3.1：有七行的星号金字塔

为了实现这个目标，请执行以下步骤：

1.  右键单击`src`文件夹，然后选择**New** | **Class**。

1.  输入`NestedPattern`作为类名，然后点击**OK**。

1.  在主方法中，创建一个`for`循环，初始化变量`i`为 1，引入条件，使得`i`的值最多为 15，并将`i`的值增加 2：

```java
public class NestedPattern{ 
public static void main(String[] args) {
for (int i = 1; i <= 15; i += 2) {
}
}
}
}
```

1.  在这个循环内，创建另外两个`for`循环，一个用于打印空格，另一个用于打印*：

```java
for (int k = 0; k < (7 - i / 2); k++) {
   System.out.print(" ");
   }
for (int j = 1; j <= i; j++) {
   System.out.print("*");
   }
```

1.  在外部`for`循环中，添加以下代码以添加下一行：

```java
System.out.println();
```

运行程序。您将看到结果金字塔。

### for-each 循环

`for each`循环是 Java 5 中引入的`for`循环的高级版本。它们用于对数组或项目列表中的每个项目执行给定操作。

让我们来看看这个`for`循环：

```java
int[] arr = { 1, 2, 3, 4, 5 , 6, 7, 8, 9,10};
for (int i  = 0; i < 10; i++){
   System.out.println(arr[i]);
}
```

第一行声明了一个整数数组。数组是相同类型项目的集合。在这种情况下，变量 arr 持有 10 个整数的集合。然后我们使用`for`循环从`0`到`10`，打印出这个数组的元素。我们使用`i < 10`是因为最后一个项目在索引`9`处，而不是`10`。这是因为数组的元素从索引 0 开始。第一个元素在索引`0`处，第二个在索引`1`处，第三个在`2`处，依此类推。`arr[0]`将返回第一个元素，`arr[1]`第二个，`arr[2]`第三个，依此类推。

这个`for`循环可以用更短的`for each`循环来替代。`for each`循环的语法如下：

```java
for( type item : array_or_collection){
    //Code to executed for each item in the array or collection
}
```

对于我们之前的例子，`for each`循环将如下所示：

```java
for(int item : arr){
   System.out.println(item);
}
```

`int` `item`是我们当前所在数组中的元素。`for each`循环将遍历数组中的所有元素。在大括号内，我们打印出这个元素。请注意，我们不必像之前的`for`循环中那样使用`arr[i]`。这是因为`for each`循环会自动为我们提取值。此外，我们不必使用额外的`int` `i`来保持当前索引并检查我们是否在`10`以下`(i < 10)`，就像我们之前使用的`for`循环那样。`for each`循环更短，会自动为我们检查范围。

例如，我们可以使用`for each`循环来打印数组`arr`中所有元素的平方：

```java
for(int item : arr){
   int square = item * item;
   System.out.println(square);
}
```

输出：

```java
1
4
9
16
25
36
49
64
81
10
```

### while 和 do while 循环

有时，我们希望重复执行某些语句，也就是说，只要某个布尔条件为真。这种情况需要我们使用`while`循环或`do while`循环。`while`循环首先检查一个布尔语句，如果布尔为真，则执行一段代码块，否则跳过`while`块。`do while`循环首先在检查布尔条件之前执行一段代码块。当您希望代码至少执行一次时，请使用`do while`循环，当您希望在第一次执行之前首先检查布尔条件时，请使用`while`循环。以下是`while`和`do while`循环的格式：

`while`循环的语法：

```java
while(condition) {
//Do something
}
```

`do while`循环的语法：

```java
do {
//Do something
}
while(condition);
```

例如，要使用`while`循环打印从 0 到 10 的所有数字，我们将使用以下代码：

```java
public class Loops {
   public static void main(String[] args){
       int number = 0;
       while (number <= 10){
           System.out.println(number);
           number++;
       }
   }
}
```

输出：

```java
0
1
2
3
4
5
6
7
8
9
10
```

我们也可以使用`do while`循环编写上述代码：

```java
public class Loops {
   public static void main(String[] args){
       int number = 0;
       do {
           System.out.println(number);
           number++;
       }while (number <= 10);
   }
}
```

使用`do while`循环，条件最后被评估，所以我们确信语句至少会被执行一次。

### 练习 10：实现 while 循环

要使用`while`循环打印斐波那契数列的前 10 个数字，执行以下步骤：

1.  右键单击`src`文件夹，然后选择**新建** | **类**。

1.  输入`FibonacciSeries`作为类名，然后单击**确定**。

1.  声明`main`方法中所需的变量：

```java
public class FibonacciSeries {
    public static void main(String[] args) {
        int i = 1, x = 0, y = 1, sum=0;
    }
}
```

这里，`i`是计数器，`x`和`y`存储斐波那契数列的前两个数字，`sum`是一个用于计算变量`x`和`y`的和的变量。

1.  实现一个`while`循环，条件是计数器`i`不超过 10：

```java
while (i <= 10)
{
}
```

1.  在`while`循环内，实现打印`x`的值的逻辑，然后分配适当的值给`x`、`y`和`sum`，这样我们总是打印最后一个和倒数第二个数字的`sum`：

```java
System.out.print(x + " ");
sum = x + y;
x = y;
y = sum;
i++;
```

### 活动 9：实现 while 循环

记得 John，他是一个桃子种植者。他从树上摘桃子，把它们放进水果箱里然后运输。如果一个水果箱装满了 20 个桃子，他就可以运输一个水果箱。如果他的桃子少于 20 个，他就必须摘更多的桃子，这样他就可以装满一个装有 20 个桃子的水果箱并运输它。

我们想通过编写一个自动化软件来帮助 John 启动箱子的填充和运输。我们从 John 那里得到桃子的数量，并为每组 20 个桃子打印一条消息，说明我们已经运输了多少箱子，还剩下多少桃子，例如，“已运输 2 箱，剩余 54 个桃子”。我们想用`while`循环来实现这一点。只要我们有足够的桃子可以装满至少一个箱子，循环就会继续。与之前的`for`活动相反，我们还将跟踪剩余的桃子。为了实现这一点，执行以下步骤：

1.  创建一个新类，输入`PeachBoxCounter`作为类名

1.  导入`java.util.Scanner`包：

1.  在`main()`中使用`System.out.print`询问用户`numberOfPeaches`。

1.  创建一个`numberOfBoxesShipped`变量。

1.  编写一个 while 循环，只要我们至少有 20 个桃子就继续。

1.  在循环中，从`numberOfPeaches`中移除 20 个桃子，并将`numberOfBoxesShipped`增加 1。打印这些值。

1.  运行主程序。

输出应该类似于：

```java
Enter the number of peaches picked: 42
1 boxes shipped, 22 peaches remaining
2 boxes shipped, 2 peaches remaining
```

#### 注意

此活动的解决方案可在第 311 页找到。

### 活动 10：实现循环结构

我们的目标是创建一个订票系统，这样当用户提出票务请求时，票务会根据餐厅剩余座位的数量来批准。

要创建这样一个程序，执行以下步骤：

1.  导入从用户读取数据所需的包。

1.  声明变量以存储总座位数、剩余座位和请求的票数。

1.  在`while`循环内，实现`if else`循环，检查请求是否有效，这意味着请求的票数少于剩余座位数。

1.  如果前一步的逻辑为真，则打印一条消息表示票已处理，将剩余座位设置为适当的值，并要求下一组票。

1.  如果第 3 步的逻辑为假，则打印适当的消息并跳出循环。

#### 注意

此活动的解决方案可在第 312 页找到。

### 活动 11：嵌套循环连续桃子运输。

记得 John，他是一个桃子种植者。他从树上摘桃子，把它们放进水果箱里然后运输。如果一个水果箱装满了 20 个桃子，他就可以运输一个水果箱。如果他的桃子少于 20 个，他就必须摘更多的桃子，这样他就可以装满一个装有 20 个桃子的水果箱并运输它。

我们希望通过编写一个自动化软件来帮助约翰启动装箱和运输。在我们的自动化软件的这个新版本中，我们将允许约翰自行选择批量带来桃子，并将上一批剩下的桃子与新批次一起使用。

我们从约翰那里得到了桃子的进货数量，并将其加到当前的桃子数量中。然后，我们为每组 20 个桃子打印一条消息，说明我们已经运送了多少箱子，还剩下多少桃子，例如，“已运送 2 箱，剩余 54 个桃子”。我们希望用`while`循环来实现这一点。只要我们有足够多的桃子可以装至少一箱，循环就会继续。我们将有另一个`while`循环来获取下一批桃子，如果没有，则退出。为了实现这一点，执行以下步骤：

1.  创建一个新的类，并输入`PeachBoxCount`作为类名

1.  导入`java.util.Scanner`包：

1.  创建一个`numberOfBoxesShipped`变量和一个`numberOfPeaches`变量。

1.  在`main()`中，编写一个无限的`while`循环。

1.  使用`System.out.print`询问用户`incomingNumberOfPeaches`。如果这是零，则跳出这个无限循环。

1.  将进货的桃子加到现有的桃子中。

1.  编写一个`while`循环，只要我们至少有 20 个桃子就继续。

1.  在 for 循环中，从`numberOfPeaches`中减去 20 个桃子，并将`numberOfBoxesShipped`增加 1。打印这些值。

1.  运行主程序。

输出应类似于：

```java
Enter the number of peaches picked: 23
1 boxes shipped, 3 peaches remaining
Enter the number of peaches picked: 59
2 boxes shipped, 42 peaches remaining
3 boxes shipped, 22 peaches remaining
4 boxes shipped, 2 peaches remaining
Enter the number of peaches picked: 0
```

#### 注意

此活动的解决方案可在第 313 页找到。

## 总结

在本课程中，我们通过查看一些简单的例子，涵盖了 Java 和编程中一些基本和重要的概念。条件语句和循环语句通常是实现逻辑的基本要素。

在下一课中，我们将专注于另外一些基本概念，如函数、数组和字符串。这些概念将帮助我们编写简洁和可重用的代码。
