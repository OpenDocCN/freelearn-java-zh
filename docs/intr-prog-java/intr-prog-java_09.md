# 第九章：运算符、表达式和语句

在本章中，将详细定义和解释 Java 编程的三个核心元素-运算符、表达式和语句。讨论将通过具体示例来支持，以说明这些元素的关键方面。

将涵盖以下主题：

+   Java 编程的核心元素是什么？

+   Java 运算符、表达式和语句

+   运算符优先级和操作数的求值顺序

+   原始类型的扩展和缩小转换

+   原始类型和引用类型之间的装箱和拆箱

+   引用类型的 equals()方法

+   练习-命名语句

# Java 编程的核心元素是什么？

在第二章中，*Java 语言基础*，我们概述了 Java 作为一种语言的许多方面，甚至定义了语句是什么。现在，我们将更系统地研究 Java 的核心元素。

“元素”这个词有点过载（玩弄方法重载的类比）。在第五章中，*Java 语言元素和类型*，我们介绍了输入元素，这些元素是由 Java 规范标识的：空格、注释和标记。这就是 Java 编译器解析源代码并理解其含义的方式。标记列表包括标识符、关键字、分隔符、文字和运算符。这就是 Java 编译器如何为其遇到的标记添加更多含义。

在讨论输入元素时，我们解释了它们用于构建语言的更复杂元素。在本章中，我们将从运算符标记开始，展示如何使用表达式-更复杂的 Java 元素来构建它。

但并非所有 Java 运算符都是标记。`instanceof`和`new`运算符是关键字，而`.`运算符（字段访问或方法调用）、`::`方法引用运算符和`( type )`强制转换运算符是分隔符。

正如我们在第二章中所说的，*Java 语言基础*，在 Java 中，语句的作用类似于英语中的句子，它表达了一个完整的思想。在编程语言中，语句是一行完整的代码，执行某些操作。

另一方面，表达式是语句的一部分，它求值为一个值。每个表达式都可以是一个语句（如果结果值被忽略），而大多数语句不包括表达式。

这就是 Java 的三个核心元素-运算符、表达式和语句的关系。

# 运算符

以下是 Java 中所有 44 个运算符的列表：

| **运算符** **描述** |
| --- |
| 算术一元和二元运算符 |
| 递增和递减一元运算符 |
| 相等运算符 |
| 关系运算符 |
| 逻辑运算符 |
| 条件运算符 |
| 赋值运算符 |
| 赋值运算符 |
| 按位运算符 |
| 箭头和方法引用运算符 |
| 实例创建运算符 |
| 字段访问/方法调用运算符 |
| 类型比较运算符 |
| （目标类型）强制转换运算符 |

一元意味着与单个操作数一起使用，而二元意味着它需要两个操作数。

在接下来的小节中，我们将定义并演示大多数运算符，除了很少使用的赋值运算符`&=`、`|=`、`^=`、`<<=`、`>>=`和`>>>=`，以及按位运算符。

另外，请注意，如果应用于整数（按位）和布尔值（逻辑），`&`和`|`运算符的行为是不同的。在本书中，我们将仅讨论这些运算符作为逻辑运算符。

箭头运算符`->`和方法引用运算符`::`将在第十七章中定义和讨论，*Lambda 表达式和函数式编程*。

# 算术一元（+ -）和二进制运算符：+  -  *  /  %

理解运算符的最佳方法是看它们的实际应用。以下是我们的演示应用程序代码（其中包含在注释中捕获的结果），解释了一元运算符`+`和`-`：

```java
public class Ch09DemoApp {
  public static void main(String[] args) {
    int i = 2;   //unary "+" is assumed by default
    int x = -i;  //unary "-" makes positive become negative
    System.out.println(x);   //prints: -2
    int y = -x;  //unary "-" makes negative become positive
    System.out.println(y);   //prints: 2
  }
}
```

以下代码演示了二进制运算符`+`、`-`、`*`、`/`和`%`：

```java
int z = x + y;              //binary "+" means "add"
System.out.println(z);      //prints: 0

z = x - y;                  //binary "-" means "subtract"
System.out.println(z);      //prints: -4
System.out.println(y - x);  //prints: 4

z = x * y;
System.out.println(z);      //prints: -4

z = x / y;
System.out.println(z);      //prints: -1

z = x * y;
System.out.println(z % 3);  //prints: -1
System.out.println(z % 2);  //prints: 0
System.out.println(z % 4);  //prints: 0

```

你可能已经猜到了，`%`运算符（称为模数）将左操作数除以右操作数，并返回余数。

一切看起来都很合乎逻辑和预期。但是，当我们尝试用余数除以另一个整数时，却没有得到预期的结果：

```java
int i1 = 11;
int i2 = 3;
System.out.println(i1 / i2); //prints: 3 instead of 3.66...
System.out.println(i1 % i2); //prints remainder: 2
```

结果`i1/i2`应该大于`3`。它必须是`3.66...`或类似的值。问题是由于操作中涉及的所有数字都是整数引起的。在这种情况下，Java 假设结果也应该表示为整数，并丢弃（不四舍五入）小数部分。

现在，让我们将操作数之一声明为`double`类型，值为 11，并再次尝试除法：

```java
double d1 = 11;
System.out.println(d1/i2);    //prints: 3.6666666666666665

```

这一次，我们得到了预期的结果，还有其他方法可以实现相同的结果：

```java
System.out.println((float)i1 / i2);  //prints: 3.6666667
System.out.println(i1 / (double)i2); //prints: 3.6666666666666665
System.out.println(i1 * 1.0 / i2);   //prints: 3.6666666666666665
System.out.println(i1 * 1f / i2);    //prints: 3.6666667
System.out.println(i1 * 1d / i2);    //prints: 3.6666666666666665
```

正如你所看到的，你可以将任何操作数转换为`float`或`double`类型（取决于你需要的精度），或者你可以包含`float`或`double`类型的数字。你可能还记得第五章中所述，带有小数部分的值默认为`double`。或者，你可以明确选择要添加的值的类型，就像我们在前面代码的最后两行中所做的那样。

无论你做什么，只要小心两个整数相除。如果你不希望小数部分被丢弃，至少将一个操作数转换为`float`或`double`（稍后在*Cast operator: ( target type )*部分详细了解转换运算符）。然后，如果需要，你可以将结果四舍五入到任何你喜欢的精度，或者将其转换回`int`：

```java
int i1 = 11;
int i2 = 3;
float r = (float)i1 / i2;
System.out.println(r);                 //prints: 3.6666667
float f = Math.round(r * 100f) / 100f;
System.out.println(f);                 //prints: 3.67
int i3 = (int)f;
System.out.println(i3);                //prints: 3
```

Java 整数除法：如果不确定，将其中一个操作数设为`double`或`float`，或者简单地给其中一个添加`1.0`的乘数。

在`String`的情况下，二进制运算符`+`表示连接，这个运算符通常被称为连接运算符：

```java
String s1 = "Nick";
String s2 = "Samoylov";
System.out.println(s1 + " " + s2);  //prints: Nick Samoylov
String s3 = s1 + " " + s2;
System.out.println(s3);             //prints: Nick Samoylov

```

并且只是作为提醒，在第五章中，*Java 语言元素和类型*，我们演示了应用于原始类型`char`的算术运算使用字符的代码点-字符的数值：

```java
char c1 = 'a';
char c2 = '$';

System.out.println(c1 + c2);       //prints: 133
System.out.println(c1/c2);         //prints: 2 
System.out.println((float)c1/c2);  //prints: 2.6944444

```

只有在记住符号`a`的代码点是 97，而符号`$`的代码点是 36 时，这些结果才有意义。

在大多数情况下，Java 中的算术运算都相当直观，不会引起混淆，除了两种情况：

+   当除法的所有操作数都是整数时

+   当`char`变量用作算术运算符的操作数时

# 递增和递减一元运算符：++ --

以下代码显示了`++`和`--`运算符的工作原理，取决于它们的位置，变量之前（前缀）还是变量之后（后缀）：

```java
int i = 2;
System.out.println(++i);        //prints: 3
System.out.println("i=" + i);   //prints: i=3
System.out.println(--i);        //prints: 2
System.out.println("i=" + i);   //prints: i=2

System.out.println(i++);        //prints: 2
System.out.println("i=" + i);   //prints: i=3
System.out.println(i--);        //prints: 3
System.out.println("i=" + i);   //prints: i=2

```

如果放在前缀位置，它会在返回变量的值之前将其值减 1。但是当放在后缀位置时，它会在返回变量的值之后将其值减 1。

`++x`表达式在返回结果之前增加`x`变量的值，而`x++`表达式在返回结果后增加`x`变量的值。

习惯这需要时间。但一旦你习惯了，写`++x;`或`x++`会感觉很容易，而不是`x = x + 1;`。在这种情况下使用前缀或后缀递增没有区别，因为它们都最终会增加`x`：

```java
int x = 0;
++x;
System.out.println(x);   //prints: 1
x = 0;
x++;
System.out.println(x);   //prints: 1

```

前缀和后缀之间的区别只有在使用返回值而不是后缀返回后变量的值时才会出现。例如，这是演示代码：

```java
int x = 0;
int y = x++ + x++;
System.out.println(y);   //prints: 1
System.out.println(x);   //prints: 2
```

`y`的值由第一个`x++`返回 0 形成，然后将`x`增加 1。第二个`x++`得到 1 作为当前的`x`值并返回它，所以`y`的值变为 1。同时，第二个`x++`再次增加`x`的值 1，所以`x`的值变为 2。

这种功能在表达式中更有意义：

```java
int n = 0;
int m = 5*n++;
System.out.println(m);   //prints: 0
System.out.println(n);   //prints: 1

```

它允许我们首先使用变量的当前值，然后将其增加 1。因此，后缀递增（递减）运算符具有增加（递减）变量值的副作用。正如我们已经提到的，这对于数组元素访问特别有益：

```java
int k = 0;
int[] arr = {88, 5, 42};
System.out.println(arr[k++]);  //prints: 88
System.out.println(k);         //prints: 1
System.out.println(arr[k++]);  //prints: 5
System.out.println(k);         //prints: 2
System.out.println(arr[k++]);  //prints: 42
System.out.println(k);         //prints: 3
```

通过将`k`设置为`-1`并将`++`移到前面也可以实现相同的结果：

```java
int k = -1;
int[] arr = {88, 5, 42};
System.out.println(arr[k++]);  //prints: 88
System.out.println(k);         //prints: 1
System.out.println(arr[++k]);  //prints: 5
System.out.println(k);         //prints: 2
System.out.println(arr[++k]);  //prints: 42
System.out.println(k);         //prints: 3
```

但是，使用`k=0`和`k++`读起来更好，因此成为访问数组组件的典型方式。但是，只有在需要按索引访问数组元素时才有用。例如，如果需要从索引`2`开始访问数组，则需要使用索引：

```java
int[] arr = {1,2,3,4};
int j = 2;
System.out.println(arr[j++]);  //prints: 3
System.out.println(arr[j++]);  //prints: 4
```

但是，如果您要按顺序访问数组，从索引 0 开始，那么有更经济的方法。请参见第十章，*控制流语句*。

# 相等运算符：  ==   !=

等号运算符`==`（表示相等）和`!=`（表示不相等）比较相同类型的值，并返回`Boolean`值`true`，如果操作数的值相等，则返回`false`。整数和布尔原始类型的相等性很简单：

```java
char a = 'a';
char b = 'b';
char c = 'a';
System.out.println(a == b);  //prints: false
System.out.println(a != b);  //prints: true
System.out.println(a == c);  //prints: true
System.out.println(a != c);  //prints: false

int i1 = 1;
int i2 = 2;
int i3 = 1;
System.out.println(i1 == i2);  //prints: false
System.out.println(i1 != i2);  //prints: true
System.out.println(i1 == i3);  //prints: true

System.out.println(i1 != i3);  //prints: false

boolean b1 = true;
boolean b2 = false;
boolean b3 = true;
System.out.println(b1 == b2);  //prints: false
System.out.println(b1 != b2);  //prints: true
System.out.println(b1 == b3);  //prints: true
System.out.println(b1 != b3);  //prints: false

```

在这段代码中，`char`类型与算术运算一样，被视为等于其代码点的数值。否则，很难理解以下行的结果：

```java
System.out.println((a + 1) == b); //prints: true

```

但是，从以下结果可以明显看出这行的解释：

```java
System.out.println(b - a);        //prints: 1
System.out.println((int)a);       //prints: 97
System.out.println((int)b);       //prints: 98

```

`a`的代码点是`97`，`b`的代码点是`98`。

对于基本类型`float`和`double`，等号运算符似乎以相同的方式工作。以下是`double`类型相等的示例：

```java
double d1 = 0.42;
double d2 = 0.43;
double d3 = 0.42;
System.out.println(d1 == d2);  //prints: false
System.out.println(d1 != d2);  //prints: true
System.out.println(d1 == d3);  //prints: true
System.out.println(d1 != d3);  //prints: false

```

但是，这是因为我们比较的是作为文字创建的数字，带有固定小数部分。如果我们比较以下计算的结果，很有可能得到的值永远不会等于预期的结果，因为有些数字（例如`1/3`）无法准确表示。那么`1/3`的情况是什么？以小数表示，它有一个永无止境的小数部分：

```java
System.out.println((double)1/3);    //prints: 0.3333333333333333 

```

这是为什么在比较`float`和`double`类型的值时，使用关系运算符`<`、`>`、`<=`或`=>`更可靠（请参见下一小节）。

在对象引用的情况下，等号运算符比较的是引用本身，而不是对象及其值：

```java
SomeClass c1 = new SomeClass();
SomeClass c2 = new SomeClass();
SomeClass c3 = c1;
System.out.println(c1 == c2);     //prints: false
System.out.println(c1 != c2);     //prints: true
System.out.println(c1 == c3);     //prints: true
System.out.println(c1 != c3);     //prints: false
System.out.println(new SomeClass() == new SomeClass());  //prints: false

```

Object equality based on the values they contain has to be performed using the `equals()` method. We talked about it in Chapter 2, *Java Language Basics*, and will discuss it more in the *Method equals() of reference types* section later.

# Relational operators:  <  >  <=  >=

Relational operators can only be used with primitive types:

```java
int i1 = 1;
int i2 = 2;
int i3 = 1;
System.out.println(i1 > i2);    //prints: false
System.out.println(i1 >= i2);   //prints: false
System.out.println(i1 >= i3);   //prints: true
System.out.println(i1 < i2);    //prints: true
System.out.println(i1 <= i2);   //prints: true
System.out.println(i1 <= i3);   //prints: true

System.out.println('a' >= 'b');  //prints: false
System.out.println('a' <= 'b');  //prints: true

double d1 = 1/3;
double d2 = 0.34;
double d3 = 0.33;
System.out.println(d1 < d2);  //prints: true
System.out.println(d1 >= d3); //prints: false     
```

In the preceding code, we see that `int` type values compare to each other as expected, and `char` type values compare to each other based on their numeric code point values.

当将原始类型`char`的变量用作算术、相等或关系运算符的操作数时，它们分配的数值等于它们表示的字符的代码点。

到目前为止，除了最后一行之外，没有什么意外。我们已经确定，作为小数表示的`1/3`应该是`0.3333333333333333`，这比`0.33`大。为什么`d1 >= d3`返回`false`？如果你说这是因为整数除法，那么你是正确的。即使赋值给`double`类型的变量，结果也是 0.0，因为整数除法`1/3`先发生，然后才将结果赋给`d1`。以下是演示它的代码：

```java
double d1 = 1/3;
double d2 = 0.34;
double d3 = 0.33;
System.out.println(d1 < d2);   //prints: true
System.out.println(d1 >= d3);  //prints: false
System.out.println(d1);        //prints: 0.0
double d4 = 1/3d;
System.out.println(d4);        //prints: 0.3333333333333333
System.out.println(d4 >= d3);  //prints: true

```

但除此之外，使用关系运算符与等式运算符相比，使用`float`和`double`类型的值会产生更可预测的结果。

在比较`float`和`double`类型的值时，请使用关系运算符`<`、`>`、`<=`或`=>`，而不是等式运算符`==`和`!=`。

就像在实验物理学中一样，在比较`float`和`double`类型的值时，请考虑精度。

# Logical operators:  !  &  |

首先让我们定义每个逻辑运算符：

+   一元运算符`!`如果操作数为`false`则返回`true`，否则返回`false`

+   二进制运算符`&`如果两个操作数都为`true`，则返回`true`

+   二进制运算符`|`如果两个操作数中至少有一个为`true`，则返回`true`

以下是演示代码：

```java
boolean x = false;
System.out.println(!x);  //prints: true
System.out.println(!!x); //prints: false
boolean y = !x;
System.out.println(y & x); //prints: false
System.out.println(y | x); //prints: true
boolean z = true;
System.out.println(y & z); //prints: true
System.out.println(y | z); //prints: true

```

注意`!`运算符可以多次应用于同一个值。

# 条件运算符：  &&   ||    ? : (三元)

我们可以重用先前的代码示例，但使用`&&`和`||`运算符，而不是`&`和`|`运算符：

```java
boolean x = false;
boolean y = !x;
System.out.println(y && x); //prints: false
System.out.println(y || x); //prints: true
boolean z = true;
System.out.println(y && z); //prints: true
System.out.println(y || z); //prints: true

```

结果并没有不同，但执行上有区别。运算符`&`和`|`总是检查两个操作数的值。与此同时，在`&&`的情况下，如果左操作数返回`false`，`&&`运算符会在不评估右操作数的情况下返回`false`。而在`||`的情况下，如果左操作数返回`true`，`||`运算符会在不评估右操作数的情况下返回`true`。以下是演示这种差异的代码：

```java
int i = 1, j = 3, k = 10;
System.out.println(i > j & i++ < k);  //prints: false
System.out.println("i=" + i);         //prints: i=2
System.out.println(i > j && i++ < k); //prints: false
System.out.println("i=" + i);         //prints: i=2

```

`&`和`&&`两个运算符都返回`false`。但是在`&&`的情况下，第二个操作数`i++ < k`不会被检查，变量`i`的值也不会改变。如果第二个操作数需要花费时间来评估，这样的优化可以节省时间。

`&&`和`||`运算符在`&&`的情况下，如果左操作数返回`false`，则不评估右操作数；在`||`的情况下，如果左操作数返回`true`，则不评估右操作数。

然而，`&`运算符在需要始终检查第二个操作数时是有用的。例如，第二个操作数可能是一个可能抛出异常并在某些罕见条件下改变逻辑流程的方法。

第三个条件运算符称为三元运算符。它的工作原理如下：

```java
int n = 1, m = 2;
System.out.println(n > m ? "n > m" : "n <= m"); //prints: n <= m
System.out.println(n > m ? true : false);       //prints: false
int max = n > m ? n : m;      
System.out.println(max);                        //prints: 2

```

它评估条件，如果条件为真，则返回第一个条目（问号后面的内容，`?`）；否则，返回第二个条目（冒号后面的内容，`:`）。这是一种非常方便和紧凑的方式，可以选择两个选项，而不是使用完整的`if-else`语句结构：

```java
String result;
if(n > m){
  result = "n > m";
} else {
  result = "n <= m";
} 
```

我们将在第十章中讨论这样的语句（称为条件语句），*控制流语句*。

# 赋值运算符（最受欢迎的）： =  +=  -=  *=  /=  %=

尽管我们不是第一次讨论它们，但这些是最常用的运算符，特别是`=`简单赋值运算符，它只是将一个值赋给一个变量（也可以说是*给变量赋值*）。我们已经多次看到了简单赋值的用法示例。

在使用简单赋值时唯一可能的注意事项是，当左侧的变量类型与右侧的值或变量类型不同时。类型的差异可能导致原始类型的值*变窄*或*变宽*，或者在一个类型是原始类型而另一个类型是引用类型时导致*装箱*或*拆箱*。我们将在稍后的*原始类型的扩宽和变窄转换*和*原始类型和引用类型之间的装箱和拆箱*部分讨论这样的赋值。

其余的赋值运算符（`+=` `-=` `*=` `/=` `%=`）称为复合赋值运算符：

+   `x += 2;` 分配这个加法的结果：`x = x + 2;`

+   `x -= 2;` 分配这个减法的结果：`x = x - 2;`

+   `x *= 2;` 分配这个乘法的结果：`x = x * 2;`

+   `x /= 2;` 分配这个除法的结果：`x = x / 2;`

+   `x %= 2;` 分配这个除法的余数：`x = x + x % 2;`

操作`x = x + x % 2;`是基于运算符优先级规则的，我们将在稍后的*运算符优先级和操作数的评估顺序*部分讨论这些规则。根据这些规则，`%`运算符（取模）首先执行，然后是`+`运算符（加法），然后将结果分配给左操作数变量`x`。这是演示代码：

```java
int x = 1;
x += 2;
System.out.println(x);    //prints: 3
x -= 1;
System.out.println(x);    //prints: 2
x *= 2;
System.out.println(x);    //prints: 4
x /= 2;
System.out.println(x);    //prints: 2
x %= 2;
System.out.println(x);    //prints: 0

```

再次，每当遇到整数除法时，最好将其转换为`float`或`double`除法，然后根据需要四舍五入或将其转换为整数。在我们的例子中，我们没有任何小数部分的损失。但是，如果我们不知道`x`的值，代码可能如下所示：

```java
x = 11;
double y = x;
y /= 3;          //That's the operation we wanted to do on x

System.out.println(y);        //prints: 3.6666666666666665
x = (int)y;
System.out.println(x);        //prints: 3

//or, if we need to round up the result:
double d = Math.round(y);     //prints: 4.0
System.out.println(d);
x = (int) d;
System.out.println(x);        //prints: 4

```

在这段代码中，我们假设我们不知道`x`的值，所以我们切换到`double`类型以避免失去小数部分。计算结果后，我们要么将其转换为`int`（小数部分丢失），要么四舍五入到最接近的整数。

在这个简单的除法中，我们可能会失去小数部分并得到`3`，即使不转换为`double`类型。但在现实生活中的计算中，公式通常不会那么简单，所以人们可能永远不知道整数除法可能发生的确切位置。这就是为什么在开始计算之前最好将值转换为`float`和`double`的良好做法。

# 实例创建运算符：new

到目前为止，我们已经看到`new`运算符被使用了很多次。它通过为新对象分配内存并返回对该内存的引用来实例化（创建）一个类。然后，这个引用通常被分配给与用于创建对象的类相同类型的变量，或者它的父类型，尽管我们也看到过一个情况，即引用从未被分配。在第六章中，*接口、类和对象构造*，例如，我们使用这段代码来演示构造函数是如何被调用的：

```java
new Child();
new Child("The Blows");

```

但这种情况非常罕见，大多数时候我们需要一个对新创建的对象的引用，以便调用它的方法：

```java
SomeClass obj = new SomeClass();
obj.someMethod();
```

在调用`new`运算符并分配内存后，相应的（显式或默认）构造函数初始化新对象的状态。我们在第六章中对此进行了广泛讨论，*接口、类和对象构造*。

由于数组也是对象，因此也可以使用`new`运算符和任何 Java 类型来创建数组：

```java
int[] arrInt = new int[42];

```

`[]`符号允许我们设置数组长度（最大组件数，也称为元素）-在前面的代码中是`42`。可能会产生混淆的一个潜在来源是，在编译时，Java 允许将值分配给大于数组长度的索引的组件：

```java
int[] arrInt = new int[42];
arrInt[43] = 22;

```

但当程序运行时，行`arrInt[43] = 22`将抛出异常：

![](img/4fbc4c0a-5e3a-4788-a04c-fa205a28259a.png)

也可以使用数组初始化程序而不使用`new`运算符来创建数组：

```java
int[] arrInt = {1,2,3,4};

```

只能使用`new`运算符创建类实例。数组可以使用`new`运算符或`{}`初始化程序创建。

我们在第五章中对此进行了广泛讨论，*Java 语言元素和类型*。如果没有明确初始化，则数组的值将设置为取决于类型的默认值（我们在第五章中也描述了它们，*Java 语言元素和类型*）。以下是一个代码示例：

```java
int[] arrInt = new int[42];
//arrInt[43] = 22;
System.out.println(arrInt[2]);      //prints: 0
System.out.println(arrInt.length);  //prints: 42
int[] arrInit = {1,2,3,4};
System.out.println(arrInit[2]);      //prints: 3
System.out.println(arrInit.length);  //prints: 4

```

而且，只是为了提醒你，数组的第一个元素的索引是 0。

# 类型比较运算符:  instanceof

`instanceof`运算符需要两个引用类型的操作数。这是因为它检查对象的父子关系，包括接口的实现。如果左操作数（对象引用）扩展或实现右侧的类型，则求值为`true`，否则为`false`。显然，每个引用`instanceof Object`都返回`true`，因为在 Java 中，每个类都隐式继承了`Object`类。当`instanceof`应用于任何类型的数组时，它仅对右操作数`Object`返回`true`。而且，由于`null`不是任何类型的实例，所以`null instanceof`对于任何类型都返回`false`。以下是演示代码：

```java
interface IntrfA{}
class ClassA implements IntrfA {}
class ClassB extends ClassA {}
class ClassX implements IntrfA {}

private void instanceofOperator() {
  ClassA classA = new ClassA();
  ClassB classB = new ClassB();
  ClassX classX = new ClassX();
  int[] arrI = {1,2,3};
  ClassA[] arrA = {new ClassA(), new ClassA()};

  System.out.println(classA instanceof Object); //prints: true
  System.out.println(arrI instanceof Object);   //prints: true
  System.out.println(arrA instanceof Object);   //prints: true
//System.out.println(arrA instanceof ClassA);   //error

  System.out.println(classA instanceof IntrfA); //prints: true
  System.out.println(classB instanceof IntrfA); //prints: true
  System.out.println(classX instanceof IntrfA); //prints: true

  System.out.println(classA instanceof ClassA); //prints: true
  System.out.println(classB instanceof ClassA); //prints: true
  System.out.println(classA instanceof ClassB); //prints: false
//System.out.println(classX instanceof ClassA); //error

  System.out.println(null instanceof ClassA);   //prints: false
//System.out.println(classA instanceof null);   //error
  System.out.println(classA == null);           //prints: false
  System.out.println(classA != null);           //prints: true
}
```

大多数结果都是直接的，可能是预期的。唯一可能预期的是`classX instanceof ClassA`。`ClassX`和`ClassA`都实现了相同的接口`IntrfA`，所以它们之间有一些亲和力-每个都可以转换为`IntrfA`接口：

```java
IntrfA intA = (IntrfA)classA;
intA = (IntrfA)classX;

```

但是这种关系不是父子类型的，所以`instanceof`运算符甚至不能应用于它们。

`instanceof`运算符允许我们检查类实例（对象）是否具有某个类作为父类或实现了某个接口。 

我们看到了`classA instanceof null`的类似问题，因为`null`根本不引用任何对象，尽管`null`是引用类型的文字。

在前面代码的最后两个语句中，我们展示了如何将对象引用与`null`进行比较。在调用引用之前，通常会使用此类比较，以确保引用不是`null`。它有助于避免令人恐惧的`NullPointerException`，它会中断执行流程。我们将在第十章中更多地讨论异常，*控制流语句*。

# 更喜欢多态而不是 instanceof 运算符

`instance of` 运算符 非常有帮助。我们在本书中多次使用它。但是，有些情况可能需要我们重新考虑使用它的决定。

每次你考虑使用`instanceof`运算符时，试着看看是否可以通过多态来避免它。

为了说明这个提示，这里有一些代码可以从多态中受益，而不是使用`intanceof`运算符：

```java
class ClassBase {
}
class ClassY extends ClassBase {
  void method(){

    System.out.println("ClassY.method() is called");
  }
}
class ClassZ extends ClassBase {
  void method(){
    System.out.println("ClassZ.method() is called");
  }
}
class SomeClass{
  public void doSomething(ClassBase object) {
    if(object instanceof ClassY){
      ((ClassY)object).method();
    } else if(object instanceof ClassZ){
      ((ClassZ)object).method();
    }
    //other code 
  }
}
```

如果我们运行以下代码片段：

```java
SomeClass cl = new SomeClass();
cl.doSomething(new ClassY());

```

我们将看到这个：

![](img/4b39a48b-3c7e-4adf-81a8-ca3ddd69c988.png)

然后，我们注意到`ClassY`和`ClassZ`中的方法具有相同的签名，因此我们可以将相同的方法添加到基类`ClassBase`中：

```java
class ClassBase {
  void method(){
    System.out.println("ClassBase.method() is called");
  }
}
```

并简化`SomeClass`的实现：

```java
class SomeClass{
  public void doSomething(ClassBase object) {
    object.method();
    //other code 
  }
```

在调用`new SomeClass().doSomething(new ClassY())`之后，我们仍然会得到相同的结果：

![](img/2a6f5a08-0855-4ced-b30a-b4cf0c925523.png)

这是因为`method()`在子类中被重写。在`ClassBase`中实现的方法可以做一些事情或什么都不做。这并不重要，因为它永远不会被执行（除非你使用`super`关键字从子类中将其强制转换来特别调用它）。

并且在重写时，不要忘记使用`@Override`注解：

```java
class ClassZ extends ClassBase {
  @Override
  void method(){
    System.out.println("ClassY.method() is called");
  }
}
```

注解将帮助您验证您没有错误，并且每个子类中的方法与父类中的方法具有相同的签名。

# 字段访问或方法调用运算符：  .

在类或接口内部，可以通过名称访问该类或接口的字段或方法。但是从类或接口外部，非私有字段或方法可以使用点（`.`）运算符访问和：

+   如果字段或方法是非静态的（实例成员），则对象名称

+   如果字段或方法是静态的，则接口或类名

点运算符（`.`）可以用于访问非私有字段或方法。如果字段或方法是静态的，则点运算符应用于接口或类名。如果字段或方法是非静态的，则点运算符应用于对象引用。

我们已经看到了许多这样的例子。因此，我们将所有情况总结在一个接口和实现它的类中。假设我们有以下名为`InterfaceM`的接口：

```java
interface InterfaceM {
  String INTERFACE_FIELD = "interface field";
  static void staticMethod1(){
    System.out.println("interface static method 1");
  }
  static void staticMethod2(){
    System.out.println("interface static method 2");
  }
  default void method1(){
    System.out.println("interface default method 1");
  }
  default void method2(){
    System.out.println("interface default method 2");
  }
  void method3();
}
```

我们可以使用点运算符（`.`）来访问非私有字段或方法，如下所示：

```java
System.out.println(InterfaceM.INTERFACE_FIELD);    //1: interface field
InterfaceM.staticMethod1();               //2: interface static method
InterfaceM.staticMethod2();               //3: interface static method
//InterfaceM.method1();                         //4: compilation error
//InterfaceM.method2();                         //5: compilation error
//InterfaceM.method3();                         //6: compilation error

System.out.println(ClassM.INTERFACE_FIELD);       //7: interface field
```

案例 1、2 和 3 很简单。案例 4、5 和 6 会生成编译错误，因为非静态方法只能通过实现接口的类的实例（对象）访问。案例 7 是可能的，但不是访问接口字段（也称为常量）的推荐方式。使用接口名称访问它们（如案例 1 中）使代码更易于理解。

现在让我们创建一个实现`InterfaceM`接口的`ClassM`类：

```java
class ClassM implements InterfaceM {
  public static String CLASS_STATIC_FIELD = "class static field";
  public static void staticMethod2(){
    System.out.println("class static method 2");
  }
  public static void staticMethod3(){
    System.out.println("class static method 3");
  }
  public String instanceField = "instance field";
  public void method2(){
    System.out.println("class instance method 2");
  }
  public void method3(){
      System.out.println("class instance method 3");
    }
}
```

以下是使用点运算符（`。）访问类字段和方法的所有可能情况：

```java
  //ClassM.staticMethod1();                       //8: compilation error
  ClassM.staticMethod2();                     //9: class static method 2
  ClassM.staticMethod3();                    //10: class static method 3

  ClassM classM = new ClassM();
  System.out.println(ClassM.CLASS_STATIC_FIELD);//11: class static field
  System.out.println(classM.CLASS_STATIC_FIELD);//12: class static field
  //System.out.println(ClassM.instanceField);    //13: compilation error
  System.out.println(classM.instanceField);         //14: instance field
  //classM.staticMethod1();                      //15: compilation error
  classM.staticMethod2();                   //16: class static method  2
  classM.staticMethod3();                    //17: class static method 3
  classM.method1();                     //18: interface default method 1
  classM.method2();                        //19: class instance method 2
  classM.method3();                        //20: class instance method 3
}
```

案例 8 会生成编译错误，因为静态方法属于实现它的类或接口（在这种情况下）。

案例 9 是静态方法隐藏的一个例子。接口中实现了具有相同签名的方法，但被类实现隐藏了。

案例 10 和 11 很简单。

案例 12 是可能的，但不建议。使用类名访问静态类字段使代码更易于理解。

案例 13 显然是一个错误，因为只能通过实例（对象）访问实例字段。

案例 14 是案例 13 的正确版本。

类 15 是一个错误，因为静态方法属于实现它的类或接口（在这种情况下），而不是类实例。

案例 16 和 17 是可能的，但不是访问静态方法的推荐方式。使用类名（而不是实例标识符）访问静态方法使代码更易于理解。

案例 18 演示了接口如何为类提供默认实现。这是可能的，因为`ClassM implements InterfaceM`有效地继承了接口的所有方法和字段。我们说有效地是因为在法律上正确的术语是类*implements*接口。但事实上，实现接口的类以与子类继承它们相同的方式获得接口的所有字段和方法。

案例 19 是类覆盖接口默认实现的一个例子。

案例 20 是经典接口实现的一个例子。这是接口的最初想法：提供 API 的抽象。

# 强制转换运算符：（目标类型）

强制转换运算符用于类型转换，将一个类型的值分配给另一个类型的变量。通常，它用于启用编译器否则不允许的转换。例如，我们在讨论整数除法、`char`类型作为数值类型以及将类引用分配给一个已实现接口类型的变量时，我们使用了类型转换：

```java
int i1 = 11;
int i2 = 3;
System.out.println((float)i1 / i2);  //prints: 3.6666667

System.out.println((int)a);          //prints: 97

IntrfA intA = (IntrfA)classA;

```

在进行强制转换时，有两个潜在的问题需要注意：

+   对于原始类型，值应该小于目标类型可以容纳的最大值（我们将在*原始类型的扩展和缩小转换*部分中详细讨论这一点）

+   对于引用类型，左操作数应该是右操作数的父类（即使是间接的），或者左操作数应该是右操作数所代表的类实现的接口（即使是间接的）：

```java
interface I1{}
interface I2{}
interface I3{}
class A implements I1, I2 {}
class B extends A implements I3{}
class C extends B {}
class D {}
public static void main(String[] args) {
   C c = new C();    //1
   A a = (A)c;       //2
   I1 i1 = (I1)c;    //3
   I2 i2 = (I2)c;    //4
   I3 i3 = (I3)c;    //5
   c = (C)a;         //6
   D d = new D();    //7
   //a = (A)d;       //8 compilation error
   i1 = (I1)d;       //9 run-time error
}
```

在这段代码中，第 6 种情况是可能的，因为我们知道对象`a`最初是基于对象`c`进行转换的，所以我们可以将其转换回类型`C`并期望它能够完全作为类`C`的对象正常运行。

第 8 种情况不会编译，因为其父子关系可以由编译器验证。

对于第 9 种情况，由于超出了本书范围的原因，编译器并不容易。因此，在编写代码时，IDE 不会给出提示，你可能认为一切都会按照你的期望工作。但是在运行时，你可能会得到`ClassCastException`：

![](img/c7514f1c-0e5b-4e2e-8520-35b894fdcf81.png)

程序员们看到这种情况就像看到`NullPointerException`或`ArrayOutOfBoundException`一样高兴。这就是为什么与类的强制转换相比，对接口的强制转换必须更加小心。

类型转换是将一个类型的值分配给另一个类型的变量。在执行此操作时，请确保目标类型可以容纳该值，并在必要时检查其是否超过最大目标类型值。

也可以将原始类型转换为匹配的引用类型：

```java
Integer integer1 = 3;                  //line 1 
System.out.println(integer1);          //prints: 3
Integer integer2 = Integer.valueOf(4); 
int i = integer2;                      //line 4
System.out.println(i);                 //prints: 4

```

在第 1 行和第 4 行，强制转换是隐式进行的。我们将在*原始类型和引用类型之间的装箱和拆箱*部分中更详细地讨论这种转换（也称为转换或装箱和拆箱）。

# 表达式

正如我们在本节开头所说的，表达式只存在于语句的一部分，后者是完整的动作（我们将在下一小节中讨论）。这意味着表达式可以是一个动作的构建块。一些表达式甚至可以在添加分号后成为一个完整的动作（表达式语句）。

表达式的区别特征在于它可以被评估，这意味着它可以产生执行结果。这个结果可以是三种之一：

+   一个变量，比如`i = 2`

+   一个值，比如`2*2`

+   当表达式是返回空（void）的方法的调用时，什么都没有。这样的表达式只能是完整的动作——带有分号的表达式语句。

表达式通常包括一个或多个运算符并进行求值。它可以产生一个变量，一个值（包含在进一步的求值中），或者可以调用一个返回空（void）的方法。

表达式的求值也可能产生副作用。也就是说，除了变量赋值或返回值之外，它还可以执行其他操作，例如：

```java
int x = 0, y;
y = x++;                  //line 2
System.out.println(y);    //prints: 0
System.out.println(x);    //prints: 1

```

第 2 行的表达式给变量`y`赋值，但也具有将`1`添加到变量`x`的值的副作用。

根据其形式，表达式可以是：

+   主表达式：

+   字面量（某个值）

+   对象创建（使用`new`运算符或`{}`数组初始化器）

+   字段访问（使用外部类的点运算符或者不使用该运算符来访问此实例）

+   方法调用（使用外部类的点运算符或者不使用该运算符来调用此实例）

+   方法引用（在 lambda 表达式中使用`::`运算符）

+   数组访问（使用`[]`符号，其中包含要访问的元素的索引）

+   一元运算符表达式（`x++`或`-y`，例如）

+   二元运算符表达式（`x+y`或`x*y`，例如）

+   三元运算符表达式（例如`x > y ? "x>y" : "x<=y"`）

+   一个 lambda 表达式 `i -> i + 1`（见第十七章，*Lambda 表达式和函数式编程*）

这些表达式根据它们产生的动作命名：对象创建表达式、强制类型转换表达式、方法调用表达式、数组访问表达式、赋值表达式等等。

由其他表达式组成的表达式称为复杂表达式。通常使用括号来清楚地标识每个子表达式，而不是依赖于运算符优先级（参见稍后的*运算符优先级和操作数的求值顺序*部分）。

# 语句

我们实际上在第二章，*Java 语言基础*中定义了一条语句。它是一个可以执行的完整动作。它可以包括一个或多个表达式，并以分号`;`结束。

Java 语句描述一个动作。它是一个可以执行的最小结构。它可能包括一个或多个表达式，也可能不包括。

Java 语句的可能种类有：

+   一个类或接口声明语句，比如`class A {...}`

+   只包含一个符号的空语句，`;`

+   局部变量声明语句，`int x;`

+   同步语句-超出本书范围

+   表达式语句，可以是以下之一：

+   方法调用语句，比如`method();`

+   赋值语句，比如`x = 3;`

+   对象创建语句，比如`new SomeClass();`

+   一个一元递增或递减语句，比如`++x ;` `--x;` `x++;` `x--;`

+   控制流语句（见第十章，*控制流语句*）：

+   选择语句：`if-else`或`switch-case`

+   迭代语句：`for`、`while`或`do-while`

+   异常处理语句，比如`try-catch-finally`或`throw`

+   分支语句，比如`break`、`continue`、`label:`、`return`、`assert`

通过在语句前面放置标识符和冒号`:`来*标记*语句。这个标签可以被分支语句`break`和`continue`使用来重定向控制流。在第十章，*控制流语句*中，我们将向您展示如何做到这一点。

通常，语句组成一个方法体，这就是程序的编写方式。

# 运算符优先级和操作数的求值顺序

当在同一个表达式中使用多个运算符时，如果没有已建立的规则，可能不明显如何执行它们。例如，在评估以下右侧表达式后，将分配给变量`x`的值是什么：

```java
int x = 2 + 4 * 5 / 6 + 3 + 7 / 3 * 11 - 4;
```

我们知道如何做，因为我们在学校学习了运算符优先级-从左到右首先应用乘法和除法运算符，然后从左到右进行加法和减法。但是，事实证明作者实际上想要这个运算符执行顺序：

```java
int x = 2 + 4 * 5 / 6 + ( 3 + 7 / 3 * (11 - 4));
```

这将产生不同的结果。

运算符优先级和括号决定了表达式的各部分的计算顺序。操作数的评估顺序为每个操作定义了其操作数的计算顺序。

括号有助于识别复杂表达式的结构并建立评估顺序，这将覆盖运算符优先级。

# 运算符优先级

Java 规范没有在一个地方提供运算符优先级。必须从各个部分整理出来。这就是为什么互联网上的不同来源有时对运算符执行顺序有点不同，所以不要感到惊讶，如果有疑问，可以进行实验或者只需设置括号以指导所需的计算顺序。

以下列表显示了从最高（第一个执行）到最低优先级（最后）的运算符优先级。具有相同优先级的运算符按其在表达式中的位置从左到右执行（如果没有使用括号）：

+   计算数组元素的索引的表达式，如`x = 4* arr[i+1]`；字段访问和方法调用点运算符`.`，如`x = 3*someClass.COUNT`或`x = 2*someClass.method(2, "b")`

+   一元后缀递增`++`和递减`--`运算符，如`x++`或`x--`，如`int m = 5*n++`; 请注意，这种运算符返回变量在递增/递减其值之前的旧值，因此具有递增值的副作用

+   一元前缀与`++`和`--`运算符，如`++x`或`--x`；一元`+`和`-`运算符，如`+x`或`-x`；逻辑运算符 NOT，如`!b`，其中 b 是布尔变量；一元位 NOT `~`（超出本书范围）

+   转换运算符`()`，如`double x = (double)11/3`，其中 11 首先转换为`double`，从而避免了整数除法丢失小数部分的问题；实例创建运算符`new`，如`new SomeClass()`

+   乘法运算符`*`, `/`, `%`

+   加法运算符`+`, `-`, 字符串连接`+`

+   位移运算符`<<`, `>>`, `>>>`;

+   关系运算符`<`, `>`, `>=`, `<=`, `instanceof`

+   相等运算符`==`, `!=`

+   逻辑和位运算符`&`

+   位运算符`^`

+   逻辑和位运算符`|`

+   条件运算符`&&`

+   条件运算符`||`

+   条件运算符`?:`（三元）

+   箭头运算符`->`

+   赋值运算符`=`, `+=`, `-=`, `*=`, `/=`, `%=`, `>>=`, `<<=`, `>>>=`, `&=`, `^=`, `|=`

如果存在括号，则首先计算最内层括号内的表达式。例如，看一下这段代码片段：

```java
int p1 = 10, p2 = 1;
int q = (p1 += 3)  +  (p2 += 3);
System.out.println(q);         //prints: 17
System.out.println(p1);        //prints: 13
System.out.println(p2);        //prints: 4

```

赋值运算符的优先级最低，但如果在括号内，它们将首先执行，如前面的代码。为了证明这一点，我们可以删除第一组括号，然后再次运行相同的代码：

```java
p1 = 10;
p2 = 1;
q = p1 += 3  +  (p2 += 3);
System.out.println(q);         //prints: 17
System.out.println(p1);        //prints: 17
System.out.println(p2);        //prints: 4

```

正如你所看到的，现在第一个操作符赋值`+=`在右侧表达式中最后执行。

使用括号可以增加复杂表达式的可读性。

您可以利用运算符优先级并编写一个表达式，其中几乎没有括号，如果有的话。但是，代码的质量不仅取决于其正确性。易于理解，以便其他程序员（也许不太熟悉运算符优先级）可以维护它也是良好编写代码的标准之一。此外，即使是代码的作者，在一段时间后，也可能难以理解结构不清晰的表达式。

# 操作数的评估顺序

在评估表达式时，首先考虑括号和运算符优先级。然后，评估具有相同执行优先级的表达式部分，因为它们在从左到右移动时出现。

使用括号可以改善对复杂表达式的理解，但太多嵌套的括号可能会使其变得模糊。如果有疑问，考虑将复杂表达式分解为几个语句。

最终，评估归结为每个运算符及其操作数。二元运算符的操作数从左到右进行评估，以便在右操作数的评估开始之前完全评估左操作数。正如我们所见，左操作数可能具有影响右操作数行为的副作用。这里是一个简单的例子：

```java
int a = 0, b = 0;
int c = a++ + (a * ++b);       //evaluates to: 0 + (1 * 1);
System.out.println(c);         //prints: 1

```

在现实生活中的例子中，表达式可以包括具有复杂功能和广泛副作用的方法。左操作数甚至可以抛出异常，因此右操作数永远不会被评估。但是，如果左操作数的评估在没有异常的情况下完成，Java 保证在执行运算符之前会完全评估两个操作数。

这个规则不适用于条件运算符`&&`、`||`和`?:`（参见*条件运算符：&& || ? : (三元)*部分）。

# 扩展和缩小引用类型

在引用类型的情况下，将子对象引用分配给父类类型的变量称为扩展引用转换或向上转换。将父类类型引用分配给子类类型的变量称为缩小引用转换或向下转换。

# 扩展

例如，如果一个类`SomeClass`扩展了`SomeBaseClass`，则以下声明和初始化也是可能的：

```java
SomeBaseClass someBaseClass = new SomeBaseClass();
someBaseClass = new SomeClass();
```

而且，由于每个类默认都扩展了`java.lang.Object`类，因此以下声明和初始化也是可能的：

```java
Object someBaseClass = new SomeBaseClass();
someBaseClass = new SomeClass();             //line 2
```

在第 2 行，我们将子类实例引用分配给了超类类型的变量。子类中存在但在超类中不存在的方法无法通过超类类型的引用访问。第 2 行的赋值被称为引用的扩展，因为它变得不太专业化。

# 缩小

将父对象引用分配给子类类型的变量称为缩小引用转换或向下转换。只有在应用了扩展引用转换之后才可能发生。

下面是一个演示情况的代码示例：

```java
class SomeBaseClass{
  void someMethod(){
    ...
  }
} 
class SomeClass extends SomeBaseClass{
  void someOtherMethod(){
    ...
  }
}
SomeBaseClass someBaseClass = new SomeBaseClass();
someBaseClass = new SomeClass();
someBaseClass.someMethod();                  //works just fine
//someBaseClass.someOtherMethod();           //compilation error
((SomeClass)someBaseClass).someOtherMethod(); //works just fine
//The following methods are available as they come from Object:
int h = someBaseClass.hashCode();
Object o = someBaseClass.clone();
//All other public Object's methods are accessible too
```

缩小转换需要转换，当我们讨论转换运算符时，我们已经详细讨论过这一点（参见*转换运算符*部分），包括转换为接口，这是另一种向上转换的形式。

# 原始类型的扩展和缩小转换

当一个数值类型的值（或变量）被赋给另一个数值类型的变量时，新类型可能包含一个更大的数字或更小的最大数字。如果目标类型可以容纳更大的数字，则转换是扩展的。否则，它是一个缩小的转换，通常需要使用转换运算符进行类型转换。

# 扩展

数值类型可以容纳的最大数字由分配给该类型的位数确定。为了提醒您，这里是每种数值类型表示的位数：

+   `byte`：8 位

+   `char`：16 位

+   `short`：16 位

+   `int`：32 位

+   `long`：64 位

+   `float`：32 位

+   `double`：64 位

Java 规范定义了 19 种扩展原始转换：

+   `byte` 到 `short`，`int`，`long`，`float`，或 `double`

+   `short` 到 `int`，`long`，`float`，或 `double`

+   `char` 到 `int`，`long`，`float`，或 `double`

+   `int` 到 `long`，`float`，或 `double`

+   `long` 到 `float` 或 `double`

+   `float` 到 `double`

在整数类型之间的扩展转换和一些整数类型到浮点值的一些转换中，结果值保持与原始值相同。但是，从 `int` 到 `float`，或从 `long` 到 `float`，或从 `long` 到 `double`，根据规范可能会导致：

“在精度损失方面 - 也就是说，结果可能会丢失一些值的最低有效位。在这种情况下，得到的浮点值将是整数值的正确舍入版本，使用 IEEE 754 最接近模式。”

让我们通过代码示例来看一下这种效果，首先从 `int` 类型转换到 `float` 和 `double` 开始：

```java
int n = 1234567899;
float f = (float)n;
int r = n - (int)f;
System.out.println(r);    //prints: -46

double d = (double)n;
r = n - (int)d;
System.out.println(r);    //prints: 0
```

正如规范所述，只有从 `int` 到 `float` 的转换丢失了精度。从 `int` 到 `double` 的转换很好。现在，让我们转换 `long` 类型：

```java
long l = 1234567899123456L;
float f = (float)l;
long rl = l - (long)f;
System.out.println(rl);    //prints: -49017088

double d = (double)l;
rl = l - (long)d;
System.out.println(rl);    //prints: 0

l = 12345678991234567L;
d = (double)l;
rl = l - (long)d;
System.out.println(rl);    //prints: -1

```

从 `long` 到 `float` 的转换严重丢失了精度。规范警告了我们。但是从 `long` 到 `double` 的转换一开始看起来很好。然后，我们将 `long` 值增加了大约十倍，得到了 `-1` 的精度损失。所以，这也取决于值有多大。

尽管如此，Java 规范不允许由扩展转换引起的任何运行时异常。在我们的例子中，我们也没有遇到异常。

# 缩小

数值原始类型的缩小转换是相反的，从更宽的类型到更窄的类型，通常需要转换。Java 规范确定了 22 种缩小的原始转换：

+   `short` 到 `byte` 或 `char`

+   `char` 到 `byte` 或 `short`

+   `int` 到 `byte`，`short`，或 `char`

+   `long` 到 `byte`，`short`，`char`，或 `int`

+   `float` 到 `byte`，`short`，`char`，`int`，或 `long`

+   `double` 到 `byte`，`short`，`char`，`int`，`long`，或 `float`

它可能导致值的大小和可能导致精度的损失。缩小过程比扩展过程更复杂，讨论它超出了入门课程的范围。至少可以做的是确保原始值小于目标类型的最大值：

```java
double dd = 1234567890.0;
System.out.println(Integer.MAX_VALUE); //prints: 2147483647
if(dd < Integer.MAX_VALUE){
  int nn = (int)dd;
  System.out.println(nn);              //prints: 1234567890
} else {
  System.out.println(dd - Integer.MAX_VALUE);
}

dd = 2234567890.0;
System.out.println(Integer.MAX_VALUE); //prints: 2147483647
if(dd < Integer.MAX_VALUE){
  int nn = (int)dd;
  System.out.println(nn);            
} else {
  System.out.println(dd - Integer.MAX_VALUE); //prints: 8.7084243E7
}
```

从这些示例中可以看出，当数字适合目标类型时，缩小转换就可以很好地进行，但是如果原始值大于目标类型的最大值，我们甚至不会尝试进行转换。

在进行强制转换之前，考虑一下目标类型可以容纳的最大值，特别是在缩小值类型时。

但是，避免完全丢失值并不是全部。在`char`类型和`byte`或`short`类型之间的转换中，事情变得特别复杂。其原因在于`char`类型是无符号数值类型，而`byte`和`short`类型是有符号数值类型，因此可能会丢失一些信息。

# 原始类型转换的方法

强制转换并不是将一个原始类型转换为另一个类型的唯一方法。每种原始类型都有一个对应的引用类型 - 称为原始类型的包装类的类。

所有包装类都位于`java.lang`包中：

+   `java.lang.Boolean`

+   `java.lang.Byte`

+   `java.lang.Character`

+   `java.lang.Short`

+   `java.lang.Integer`

+   `java.lang.Long`

+   `java.lang.Float`

+   `java.lang.Double`

它们中的大多数（除了`Boolean`和`Character`类）都扩展了`java.lang.Number`类，该类具有以下抽象方法声明：

+   `byteValue()`

+   `shortValue()`

+   `intValue()`

+   `longValue()`

+   `floatValue()`

+   `doubleValue()`

这意味着每个`Number`类子类都必须实现所有这些方法。这些方法也在`Character`类中实现，而`Boolean`类具有`booleanValue()`方法。这些方法也可以用于扩大和缩小原始类型。

此外，每个包装类都有方法，允许将数值的`String`表示转换为相应的原始数值类型或引用类型，例如：

```java
byte b = Byte.parseByte("3");
Byte bt = Byte.decode("3");
boolean boo = Boolean.getBoolean("true");
Boolean bool = Boolean.valueOf("false");
int n = Integer.parseInt("42");
Integer integer = Integer.getInteger("42");
double d1 = Double.parseDouble("3.14");
Double d2 = Double.valueOf("3.14");
```

之后，可以使用先前列出的方法（`byteValue()`，`shortValue()`等）将值转换为另一种原始类型。

每个包装类都有静态方法`toString(原始值)`，将原始类型值转换为其`String`表示：

```java
String s1 = Integer.toString(42);
String s2 = Double.toString(3.14);
```

包装类有许多其他有用的方法，可以将一种原始类型转换为另一种原始类型，以及不同的格式和表示形式。因此，如果需要这样的功能，请首先查看`java.lang`包中的其数值类型类包装器。

其中一种类型转换允许从相应的原始类型创建包装类对象，反之亦然。我们将在下一节讨论这样的转换。

# 原始类型和引用类型之间的装箱和拆箱

装箱将原始类型的值转换为相应包装类的对象。拆箱将包装类的对象转换为相应原始类型的值。

# 装箱

装箱原始类型可以通过自动方式（称为自动装箱）或显式地使用每个包装类型中可用的`valueOf()`方法来完成：

```java
int n = 12;
Integer integer = n; //an example of autoboxing
System.out.println(integer);      //prints: 12
integer = Integer.valueOf(n);
System.out.println(integer);      //prints: 12

Byte b = Byte.valueOf((byte)n);
Short s = Short.valueOf((short)n);
Long l = Long.valueOf(n);
Float f = Float.valueOf(n);
Double d = Double.valueOf(n);

```

请注意，`Byte`和`Short`包装器的`valueOf()`方法的输入值需要转换，因为它是原始类型的缩小，这是我们在上一节中讨论的。

# 拆箱

拆箱可以使用每个包装类中实现的`Number`类的方法来完成：

```java
Integer integer = Integer.valueOf(12);
System.out.println(integer.intValue());    //prints: 12
System.out.println(integer.byteValue());   //prints: 12
System.out.println(integer.shortValue());  //prints: 12
System.out.println(integer.longValue());   //prints: 12
System.out.println(integer.floatValue());  //prints: 12.0
System.out.println(integer.doubleValue()); //prints: 12.0
```

类似于自动装箱，也可以自动拆箱：

```java
Long longWrapper = Long.valueOf(12L);
long lng = longWrapper;    //implicit unboxing
System.out.println(lng);   //prints: 12
```

但是，它不被称为自动装箱。而是使用隐式拆箱这个术语。

# 引用类型的 equals()方法

当应用于引用类型时，等式运算符比较引用值，而不是对象的内容。只有当两个引用（变量值）指向同一个对象时，它才返回`true`。我们已经多次证明了这一点：

```java
SomeClass o1 = new SomeClass();
SomeClass o2 = new SomeClass();
System.out.println(o1 == o2);  //prints: false
System.out.println(o1 == o1);  //prints: true
o2 = o1;
System.out.println(o1 == o2);  //prints: true
```

这意味着即使比较具有相同字段值的相同类的两个对象时，等式运算符也会返回`false`。这通常不是程序员所需要的。相反，我们通常需要在两个对象具有相同类型和相同字段值时将它们视为相等。有时，我们甚至不想考虑所有字段，而只想考虑那些在程序逻辑中唯一标识对象的字段。例如，如果一个人改变了发型或服装，我们仍然认为他或她是同一个人，即使描述该人的对象具有字段`hairstyle`或`dress`。

# 使用基类 Object 的实现

对于这种对象的比较-按照它们的字段值-应使用`equals()`方法。在第二章中，*Java 语言基础*，我们已经确定所有引用类型都扩展（隐式）`java.lang.Object`类，该类已实现了`equals()`方法：

```java
public boolean equals(Object obj) {
  return (this == obj);
}
```

正如你所看到的，它只使用相等运算符比较引用，这意味着如果一个类或其父类没有实现`equals()`方法（覆盖`Object`类的实现），使用`equals()`方法的结果将与使用相等运算符`==`相同。让我们来演示一下。以下类没有实现`equals()`方法：

```java
class PersonNoEquals {
  private int age;
  private String name;

  public PersonNoEquals(int age, String name) {
    this.age = age;
    this.name = name;
  }
}
```

如果我们使用它并比较`equals()`方法和`==`运算符的结果，我们将看到以下结果：

```java
PersonNoEquals p1 = new PersonNoEquals(42, "Nick");
PersonNoEquals p2 = new PersonNoEquals(42, "Nick");
PersonNoEquals p3 = new PersonNoEquals(25, "Nick");
System.out.println(p1.equals(p2));     //false
System.out.println(p1.equals(p3));     //false
System.out.println(p1 == p2);          //false
p1 = p2;
System.out.println(p1.equals(p2));     //true
System.out.println(p1 == p2);          //true

```

正如我们所预期的，无论我们使用`equals()`方法还是`==`运算符，结果都是相同的。

# 覆盖 equals()方法

现在，让我们实现`equals()`方法：

```java
class PersonWithEquals{
  private int age;
  private String name;
  private String hairstyle;

  public PersonWithEquals(int age, String name, String hairstyle) {
    this.age = age;
    this.name = name;

    this.hairstyle = hairstyle;
  }

  @Override
  public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    PersonWithEquals person = (PersonWithEquals) o;
    return age == person.age && Objects.equals(name, person.name);
  }
}
```

请注意，当建立对象的相等性时，我们忽略了“发型”字段。需要评论的另一个方面是使用`java.utils.Objects`类的`equals()`方法。以下是它的实现：

```java
public static boolean equals(Object a, Object b) {
  return (a == b) || (a != null && a.equals(b));
}
```

如您所见，它首先比较引用，然后确保一个不是`null`（以避免`NullPointerException`），然后使用`java.lang.Object`基类的`equals()`方法或可能存在的子类中的重写实现作为参数值传递。在我们的情况下，我们传递了类型为`String`的参数对象，它们已经实现了`equals()`方法，用于比较`String`类型的值，而不仅仅是引用（我们将很快讨论它）。因此，`PersonWithEquals`对象的任何字段的任何差异都将导致该方法返回 false。

如果我们再次运行测试，我们将看到这个：

```java
PersonWithEquals p11 = new PersonWithEquals(42, "Kelly", "Ponytail");
PersonWithEquals p12 = new PersonWithEquals(42, "Kelly", "Pompadour");
PersonWithEquals p13 = new PersonWithEquals(25, "Kelly", "Ponytail");
System.out.println(p11.equals(p12));    //true
System.out.println(p11.equals(p13));    //false
System.out.println(p11 == p12);         //false
p11 = p12;
System.out.println(p11.equals(p12));    //true
System.out.println(p11 == p12);         //true
```

现在，`equals()`方法不仅在引用相等时返回 true（因此它们指向相同的对象），而且在引用不同但它们引用的对象具有相同类型和包含在对象标识中的某些字段的相同值时也返回 true。

# 使用在父类中实现的标识

我们可以创建一个基类`Person`，它只有两个字段“年龄”和“名字”，以及`equals()`方法，如前所述实现。然后，我们可以用`PersonWithHair`类扩展它（它有额外的字段“发型”）：

```java
class Person{
  private int age;
  private String name;
  public Person(int age, String name) {
    this.age = age;
    this.name = name;
  }
  @Override
  public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Person person = (Person) o;
    return age == person.age && Objects.equals(name, person.name);
  }
}
class PersonWithHair extends Person{
  private String hairstyle;
  public PersonWithHair(int age, String name, String hairstyle) {
    super(age, name);
    this.hairstyle = hairstyle;
  }
}
```

`PersonWithHair`的对象将与`PersonWithEquals`的先前测试中的方式进行比较。

```java
PersonWithHair p21 = new PersonWithHair(42, "Kelly", "Ponytail");
PersonWithHair p22 = new PersonWithHair(42, "Kelly", "Pompadour");
PersonWithHair p23 = new PersonWithHair(25, "Kelly", "Ponytail");
System.out.println(p21.equals(p22));    //true
System.out.println(p21.equals(p23));    //false
System.out.println(p21 == p22);         //false
p21 = p22;
System.out.println(p21.equals(p22));    //true
System.out.println(p21 == p22);         //true

```

这是可能的，因为`PersonWithHair`的对象也是`Person`的类型，所以接受这一行：

```java
Person person = (Person) o;
```

`equals()`方法中的前一行不会抛出`ClassCastException`。

然后我们可以创建`PersonWithHairDresssed`类：

```java
PersonWithHairDressed extends PersonWithHair{
  private String dress;
  public PersonWithHairDressed(int age, String name, 
                               String hairstyle, String dress) {
    super(age, name, hairstyle);
    this.dress = dress;
  }
}
```

如果我们再次运行相同的测试，结果将是一样的。但我们认为服装和发型不是身份识别的一部分，所以我们可以运行测试来比较`Person`的孩子们：

```java
Person p31 = new PersonWithHair(42, "Kelly", "Ponytail");
Person p32 = new PersonWithHairDressed(42, "Kelly", "Pompadour", "Suit");
Person p33 = new PersonWithHair(25, "Kelly", "Ponytail");
System.out.println(p31.equals(p32));    //false
System.out.println(p31.equals(p33));    //false
System.out.println(p31 == p32);         //false

```

这不是我们期望的！孩子们被认为不相等，因为在`Person`基类的`equals()`方法中有这行：

```java
if (o == null || getClass() != o.getClass()) return false;

```

前面的行失败了，因为`getClass()`和`o.getClass()`方法返回的是子类名 - 使用`new`操作符实例化的类。为了摆脱这个困境，我们使用以下逻辑：

+   我们的`equals()`方法的实现位于`Person`类中，所以我们知道当前对象是`Person`类型

+   要比较类，我们只需要确保另一个对象也是`Person`类型

如果我们替换这行：

```java
if (o == null || getClass() != o.getClass()) return false;
```

使用以下代码：

```java
if (o == null) return false;
if(!(o instanceof Person)) return false;

```

结果将是这样的：

```java
Person p31 = new PersonWithHair(42, "Kelly", "Ponytail");
Person p32 = new PersonWithHairDressed(42, "Kelly", "Pompadour", "Suit");
Person p33 = new PersonWithHair(25, "Kelly", "Ponytail");
System.out.println(p31.equals(p32));    //true
System.out.println(p31.equals(p33));    //false
System.out.println(p31 == p32);         //false
```

这就是我们想要的，不是吗？这样，我们已经实现了最初的想法，即不包括发型和服装在人的身份识别中。

在对象引用的情况下，等号运算符`==`和`!=`比较的是引用本身 - 而不是对象字段（状态）的值。如果需要比较对象状态，请使用重写了`Object`类中的`equals()`方法。

`String`类和原始类型的包装类也重写了`equals()`方法。

# String 类的 equals()方法

在第五章中，*Java 语言元素和类型*，我们已经讨论过这个问题，甚至审查了源代码。这里是源代码：

```java
public boolean equals(Object anObject) {
  if (this == anObject) {
    return true;
  }
  if (anObject instanceof String) {

    String aString = (String)anObject;
    if (coder() == aString.coder()) {
      return isLatin1() ? 
               StringLatin1.equals(value, aString.value)
               : StringUTF16.equals(value, aString.value);
    }
  }
  return false;
}
```

如你所见，它重写了`Object`类的实现，以便比较值，而不仅仅是引用。这段代码证明了这一点：

```java
String sl1 = "test1";
String sl2 = "test2";
String sl3 = "test1";

System.out.println(sl1 == sl2);              //1: false
System.out.println(sl1.equals(sl2));         //2: false

System.out.println(sl1 == sl3);              //3: true
System.out.println(sl1.equals(sl3));         //4: true

String s1 = new String("test1");
String s2 = new String("test2");
String s3 = new String("test1");

System.out.println(s1 == s2);                //5: false
System.out.println(s1.equals(s2));           //6: false

System.out.println(s1 == s3);                //7: false
System.out.println(s1.equals(s3));           //8: true

System.out.println(sl1 == s1);               //9: false
System.out.println(sl1.equals(s1));          //10: true
```

你可以看到等号运算符`==`有时会正确比较`String`对象的值，有时则不会。然而，`equal()`方法总是正确比较值，即使它们被包装在不同的对象中，而不仅仅是引用文字。

我们在测试中包含了等号运算符，以澄清在互联网上经常读到的关于`String`值的不正确解释的情况。这种不正确的解释基于支持`String`实例不可变性的 JVM 实现（在第五章中阅读关于`String`不可变性及其动机的内容）。JVM 不会两次存储相同的`String`值，并且会重用已存储在称为**字符串池**的区域中的值，这个过程称为**字符串池化**。了解了这一点后，有些人认为使用`equals()`方法与`String`值是不必要的，因为相同的值无论如何都会有相同的引用值。我们的测试证明，在`String`类中包装的`String`值的情况下，等号运算符无法正确比较其值，必须使用`equals()`方法。还有其他情况，`String`值没有存储在字符串池中。

要比较两个`String`对象的值，总是使用`equals()`方法，而不是等号`==`。

一般来说，`equals()`方法不如`==`运算符快。但是，正如我们在第五章中指出的那样，*Java 语言元素和类型*，String 类的`equals()`方法首先比较引用，这意味着在调用`equals()`方法之前没有必要尝试节省性能时间并比较引用。只需调用`equals()`方法。

`String`类型行为的模糊性 - 有时像原始类型，有时像引用类型 - 让我想起了物理学中基本粒子的双重性质。粒子有时表现得像小而集中的物体，但有时像波。背后到底发生了什么？那里也是不可变的吗？

# 原始类型的包装类中的 equals()方法

如果我们对包装类运行测试，结果将是：

```java
long ln = 42;
Integer n = 42;
System.out.println(n.equals(42));      //true

System.out.println(n.equals(ln));      //false
System.out.println(n.equals(43));      //false

System.out.println(n.equals(Integer.valueOf(42)));  //true
System.out.println(n.equals(Long.valueOf(42)));     //false

```

根据我们对`Person`的子类的经验，我们可以相当自信地假设包装类的`equals()`方法包括类名的比较。让我们看看源代码。这是`Integer`类的`equals()`方法：

```java
public boolean equals(Object obj) {
  if (obj instanceof Integer) {
    return value == ((Integer)obj).intValue();
  }
  return false;
}
```

这正是我们所期望的。如果一个对象不是`Integer`类的实例，即使它携带完全相同的数值，也永远不能被认为等于另一个类的对象。这看起来就像古代社会阶级制度一样，不是吗？

# 练习 - 命名语句

以下语句称为什么？

+   `i++;`

+   `String s;`

+   `s = "I am a string";`

+   `doSomething(1, "23");`

# 答案

以下语句称为：

+   递增语句：`i++;`

+   变量声明语句：`String s;`

+   赋值语句：`s = "I am a string";`

+   方法调用语句：`doSomething(1, "23");`

# 总结

在本章中，我们学习了 Java 编程的三个核心元素——运算符、表达式和语句——以及它们之间的关系。我们为您介绍了所有的 Java 运算符，讨论了一些最受欢迎的运算符，并通过示例解释了它们的潜在问题。本章的相当部分专门讨论了数据类型转换：扩宽和缩窄、装箱和拆箱。还演示了引用类型的`equals()`方法，并针对各种类和实现进行了具体示例的测试。`String`类被广泛使用，并解决了关于其行为的流行错误解释。

在下一章中，我们将开始编写程序逻辑——任何执行流程的支柱——使用控制流语句，这些语句将被定义、解释并通过许多示例进行演示：条件语句、迭代语句、分支语句和异常。
