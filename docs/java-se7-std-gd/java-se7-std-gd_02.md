# 第二章：Java 数据类型及其使用

在本章中，我们将更多地了解 Java 如何组织和操作数据，特别是基本数据类型和字符串。除此之外，我们还将探讨各种相关概念，如作用域和变量的生命周期。虽然字符串在 Java 中不是基本数据类型，但它们是许多应用程序的重要组成部分，我们将探讨 Java 提供了什么。

在本章中，我们将重点关注：

+   基本数据类型的声明和使用

+   使用`String`和`StringBuilder`类

+   程序堆栈和堆如何相互关联

+   类和对象之间的区别

+   Java 中的常量和文字

+   变量的作用域和生命周期

+   运算符、操作数和表达式

# 理解 Java 如何处理数据

编程的核心是操作数据的代码。作为程序员，我们对数据和代码的组织感兴趣。数据的组织被称为**数据结构**。这些结构可以是静态的或动态的。例如，人口的年龄可以存储在数据结构中连续的位置，这种数据结构称为**数组**。虽然数组数据结构具有固定的大小，但内容可能会改变或不改变。数组在第四章中有详细讨论，*使用数组和集合*。

在本节中，我们将研究变量的几个不同方面，包括：

+   它们是如何声明的

+   基本数据类型与对象

+   它们在内存中的位置

+   它们是如何初始化的

+   它们的作用域和生命周期

## Java 标识符、对象和内存

变量被定义为特定类型，并分配内存。当创建对象时，构成对象的实例变量被分配在堆上。对象的静态变量被分配到内存的特殊区域。当变量被声明为方法的一部分时，变量的内存被分配在程序堆栈上。

## 堆栈和堆

对堆栈/堆和其他问题的彻底理解对于理解程序如何工作以及开发人员如何使用 Java 等语言来完成工作至关重要。这些概念为理解应用程序的工作方式提供了一个框架，并且是 Java 使用的运行时系统的实现的基础，更不用说几乎所有其他编程语言的实现了。

话虽如此，堆栈和堆的概念相当简单。**堆栈** 是每次调用方法时存储方法的参数和局部变量的区域。**堆** 是在调用`new`关键字时分配对象的内存区域。方法的参数和局部变量构成一个**激活记录**，也称为**堆栈帧**。激活记录在方法调用时被推送到堆栈上，并在方法返回时从堆栈上弹出。这些变量的临时存在决定了变量的生命周期。

![堆栈和堆](img/7324_02_01.jpg)

当调用方法时，堆栈向堆增长，并在方法返回时收缩。堆不会按可预测的顺序增长，并且可能变得分散。由于它们共享相同的内存空间，如果堆和堆栈发生碰撞，程序将终止。

### 注意

理解堆栈和堆的概念很重要，因为：

+   它提供了一个基础，用于理解应用程序中数据的组织方式

+   它有助于解释变量的作用域和生命周期

+   它有助于解释递归的工作原理

我们将重复使用第一章中演示的程序，*Java 入门*，以演示堆栈和堆的使用。该程序已经在此处复制以方便您使用：

```java
package com.company.customer;
import java.math.BigDecimal;
import java.util.Locale;

public class Customer {
  private String name;
  private int accountNumber;
  private Locale locale;
  private BigDecimal balance;

  public Customer() {
    this.name = "Default Customer";
    this.accountNumber = 12345;
    this.locale = Locale.ITALY;
    this.balance = new BigDecimal("0");
  }

  public String getName() {
    return name;
  }
  public void setName(String name) throws Exception {
    if(name == null) {
      throw new Exception("Names must not be null");
    } else {
      this.name = name;
    }
  }

  public int getAccountNumber() {
    return accountNumber;
  }

  public void setAccountNumber(int accountNumber) {
    this.accountNumber = accountNumber;
  }

  public BigDecimal getBalance() {
    return balance;
  }

  public void setBalance(float balance) {
    this.balance = new BigDecimal(balance);
  }

  public String toString() {
    java.text.NumberFormat format;
    format = java.text.NumberFormat.getCurrencyInstance(locale);
    return format.format(balance);
  }
 }

package com.company.customer;
public class CustomerDriver {
  public static void main(String[] args) {
    Customer customer;      // defines a reference to a Customer
    customer = new Customer();  // Creates a new Customer object
    customer.setBalance(12506.45f);
    System.out.println(customer.toString());
  }
```

当执行`main`方法时，激活记录被推送到程序堆栈上。如下图所示，它的激活记录仅包括单个`args`参数和`customer`引用变量。当创建`Customer`类的实例时，将在堆上创建并分配一个对象。在此示例中反映的堆栈和堆的状态是在`Customer`构造函数执行后发生的。`args`引用变量指向一个数组。数组的每个元素引用表示应用程序命令行参数的字符串。在下图所示的示例中，我们假设有两个命令行参数，参数 1 和参数 2：

![堆栈和堆](img/7324_02_02.jpg)

当执行`setBalance`方法时，它的激活记录被推送到程序堆栈上，如下所示。`setBalance`方法有一个参数`balance`，它被分配给`balance`实例变量。但首先，它被用作`BigDecimal`构造函数的参数。`this`关键字引用当前对象。

堆是为对象动态分配的内存。堆管理器控制这些内存的组织方式。当对象不再需要时，将执行垃圾收集例程以释放内存以便重新使用。在对象被处理之前，将执行对象的`finalize`方法。但是，不能保证该方法将执行，因为程序可能在不需要运行垃圾收集例程的情况下终止。原始的`BigDecimal`对象最终将被销毁。

![堆栈和堆](img/7324_02_03.jpg)

### 注意

在 C++中，当对象即将被销毁时，其析构函数将被执行。Java 最接近的是`finalize`方法，当对象被垃圾收集器处理时将执行。但是，垃圾收集器可能不会运行，因此`finalize`方法可能永远不会执行。这种范式转变导致了我们在资源管理方面的重要差异。第八章中介绍的“try-with-resources”块，*应用程序中的异常处理*，提供了一种处理这种情况的技术。

## 声明变量

变量也称为标识符。术语“变量”意味着它的值可以更改。这通常是这样的。但是，如果标识符被声明为常量，如*常量*部分所讨论的那样，那么它实际上不是一个变量。尽管如此，变量和标识符这两个术语通常被认为是同义词。

变量的声明以数据类型开头，然后是变量名，然后是分号。数据类型可以是原始数据类型或类。当数据类型是类时，变量是对象引用变量。也就是说，它是对对象的引用。

### 注意

引用变量实际上是一个伪装的 C 指针。

变量可以分为以下三类：

+   实例变量

+   静态变量

+   局部变量

实例变量用于反映对象的状态。静态变量是所有实例共有的变量。局部变量在方法内声明，只在声明它们的块中可见。

标识符区分大小写，只能由以下内容组成：

+   字母、数字、下划线（_）和美元符号（$）

+   标识符只能以字母、下划线或美元符号开头

有效的变量名示例包括：

+   `numberWheels`

+   `ownerName`

+   `mileage`

+   `_byline`

+   `numberCylinders`

+   `$newValue`

+   `_engineOn`

按照惯例，标识符和方法以小写字母开头，后续单词大写，如第一章中的*Java 命名约定*部分所讨论的那样。常规声明的示例包括以下内容：

+   `int numberWheels;`

+   `int numberCylinders;`

+   `float mileage;`

+   `boolean engineOn;`

+   `int $newValue;`

+   `String ownerName;`

+   `String _byline;`

在前面的示例中，除了最后两个变量外，每个变量都声明为原始数据类型。最后一个声明为对`String`对象的引用。引用变量可以引用`String`对象，但在这个示例中，它被赋予了一个`null`值，这意味着它当前没有引用字符串。字符串在*String 类*部分中有更详细的介绍。以下代码片段声明了三个整数类型的变量：

```java
int i;
int j;
int k;
```

也可以在一行上声明所有三个变量，如下所示：

```java
int i, j, k;
```

## 原始数据类型

Java 中定义了八种原始数据类型，如下表所示。在 Java 中，每种数据类型的大小对所有机器来说都是相同的：

| 数据类型 | 字节大小 | 内部表示 | 范围 |
| --- | --- | --- | --- |
| `boolean` | -- | 没有精确定义 | `true` 或 `false` |
| `byte` | 1 | 8 位二进制补码 | `-128` 到 `+127` |
| `char` | 2 | Unicode | `\u0000` 到 `\uffff` |
| `short` | 2 | 16 位二进制补码 | `-32768` 到 `32767` |
| `int` | 4 | 32 位二进制补码 | `-2,147,483,648` 到 `2,147,483,647` |
| `long` | 8 | 64 位二进制补码 | `-9,223,372,036,854,775,808` 到 `9,223,372,036,854,775,807` |
| `float` | 4 | 32 位 IEEE 754 浮点数 | `3.4e +/- 38`（7 位数字） |
| `double` | 8 | 64 位 IEEE 754 浮点数 | `1.7e +/- 308`（15 位数字） |

`String`数据类型也是 Java 的一部分。虽然它不是原始数据类型，但它是一个类，并且在*String 类*部分中有详细讨论。

另一种常见的数据类型是货币。在 Java 中，有几种表示货币的方式，如下表所述。然而，推荐的方法是使用`BigDecimal`类。

| 数据类型 | 优点 | 缺点 |
| --- | --- | --- |
| 整数 | 适用于简单的货币单位，如一分钱。 | 它不使用小数点，如美元和美分中使用的那样。 |
| 浮点数 | 使用小数点 | 舍入误差非常常见。 |
| `BigDecimal`类 |

+   处理大数字。

+   使用小数点。

+   具有内置的舍入模式。

| 更难使用。 |
| --- |

在使用`BigDecimal`时，重要的是要注意以下几点：

+   使用带有`String`参数的构造函数，因为它在放置小数点时做得更好

+   `BigDecimal`是不可变的

+   `ROUND_HALF_EVEN`舍入模式引入了最小偏差

`Currency`类用于控制货币的格式。

### 提示

关于货币表示的另一个建议是基于使用的数字位数。

**数字位数** **推荐的数据类型**

小于 10 的整数或`BigDecimal`

小于 19 的长整型或`BigDecimal`

大于 19 `BigDecimal`

在大多数语言中，浮点数可能是问题的重要来源。考虑以下片段，我们在尝试获得值`1.0`时添加`0.1`：

```java
float f = 0.1f;
for(int i = 0; i<9; i++) {
   f += 0.1f;
}
System.out.println(f);
```

输出如下：

```java
1.0000001

```

它反映了十进制值`0.1`无法在二进制中准确表示的事实。这意味着我们在使用浮点数时必须时刻保持警惕。

## 包装类和自动装箱

包装类用于将原始数据类型值封装在对象中。在装箱可用之前，通常需要显式使用包装类，如`Integer`和`Float`类。这是为了能够将原始数据类型添加到`java.util`包中经常出现的集合中，包括`ArrayList`类，因为这些数据类的方法使用对象作为参数。包装类包括以下数据类型：

+   布尔

+   字节

+   字符

+   短

+   整数

+   长

+   浮点

+   双

这些包装类的对象是不可变的。也就是说，它们的值不能被改变。

**自动装箱**是将原始数据类型自动转换为其对应的包装类的过程。这是根据需要执行的，以消除在原始数据类型和其对应的包装类之间执行琐碎的显式转换的需要。**取消装箱**是指将包装对象自动转换为其等效的原始数据类型。实际上，在大多数情况下，原始数据类型被视为对象。

在处理原始值和对象时有一些需要记住的事情。首先，对象可以是`null`，而原始值不能被赋予`null`值。这有时可能会带来问题。例如，取消装箱一个空对象将导致`NullPointerException`。此外，在比较原始值和对象时要小心，当装箱不发生时，如下表所示：

| 比较 | 两个原始值 | 两个对象 | 一个原始值和一个对象 |
| --- | --- | --- | --- |
| `a == b` | 简单比较 | 比较引用值 | 被视为两个原始值 |
| `a.equals(b)` | 不会编译 | 比较值的相等性 | 如果 a 是原始值，否则它们的值将被比较 |

## 初始化标识符

Java 变量的初始化实际上是一个复杂的过程。Java 支持四种初始化变量的方式：

+   默认初始值

+   实例变量初始化程序

+   实例初始化程序

+   构造函数

在本章中，我们将研究前两种方法。后两种技术在第六章中进行了介绍，*类，构造函数和方法*，在那里整个初始化过程被整合在一起。

当未提供显式值时，对象创建时使用初始默认值。一般来说，当对象的字段被分配时，它会被初始化为零值，如下表所述：

| 数据类型 | 默认值（对于字段） |
| --- | --- |
| `boolean` | `false` |
| `byte` | `0` |
| `char` | `'`\`u0000'` |
| `short` | `0` |
| `int` | `0` |
| `long` | `0L` |
| `float` | `0.0f` |
| `double` | `0.0d` |
| `String`（或任何对象） | `null` |

例如，在以下类中，`name`被赋予`null`，`age`的值为`0`：

```java
class Person {
  String name;
  int age;
  …
}
```

实例变量初始化程序的运算符可以用来显式地为变量分配一个值。考虑`Person`类的以下变化：

```java
class Person {
  String name = "John Doe";
  int age = 23;
  …
}
```

当创建`Person`类型的对象时，`name`和`age`字段分别被赋予值`John Doe`和`23`。

然而，当声明一个局部变量时，它不会被初始化。因此，重要的是要在声明变量时使用初始化运算符，或者在为其分配值之前不使用该变量。否则，将导致语法错误。

## Java 常量，字面量和枚举

常量和字面量在不能被更改方面是相似的。变量可以使用`final`关键字声明为不能更改的原始数据类型，因此被称为常量。字面量是表示值的标记，例如`35`或`'C'`。显然，它也不能被修改。与此概念相关的是不可变对象——不能被修改的对象。虽然对象不能被修改，但指向对象的引用变量可以被更改。

枚举在本质上也是常量。它们用于提供一种方便的方式来处理值的集合作为列表。例如，可以创建一个枚举来表示一副牌的花色。

### 字面量

字面常量是表示数量的简单数字、字符和字符串。有三种基本类型：

+   数字

+   字符

+   字符串

#### 数字字面量

数字常量由一系列数字组成，可选的符号和可选的小数点。包含小数点的数字字面量默认为`double`常量。数字常量也可以以`0x`为前缀表示为十六进制数（基数 16）。以`0`开头的数字是八进制数（基数 8）。后缀`f`或`F`可以用来声明浮点字面量的类型为`float`。

| 数字字面量 | 基数 | 数据类型 | 十进制等价 |
| --- | --- | --- | --- |
| `25` | 10 | `int` | `25` |
| `-235` | 10 | `int` | `-235` |
| `073` | 8 | `int` | `59` |
| `0x3F` | 16 | `int` | `63` |
| `23.5` | 10 | `double` | `23.5` |
| `23.5f` | 10 | `float` | `23.5` |
| `23.5F` | 10 | `float` | `23.5` |
| `35.05E13` | 10 | `double` | `350500000000.00` |

整数字面量很常见。通常它们以十进制表示，但可以使用适当的前缀创建八进制和十六进制字面量。整数字面量默认为`int`类型。可以通过在字面量的末尾添加 L 来指定字面量的类型为`long`。下表说明了字面量及其对应的数据类型：

| 字面量 | 类型 |
| --- | --- |
| `45` | `int` |
| `012` | 以八进制数表示的整数。 |
| `0x2FFC` | 以十六进制数表示的整数。 |
| `10L` | `long` |
| `0x10L` | 以十六进制数表示的长整型。 |

### 注意

可以使用小写或大写的 L 来指定整数的长整型类型。但最好使用大写的 L，以避免将字母与数字 1 混淆。在下面的例子中，一个不小心的读者可能会将字面量看作是一百零一，而不是整数 10：

`10l`与`10L`

浮点字面量是包含小数点的数字，或者使用科学计数法写成的数字。

| 字面量 | 类型 |
| --- | --- |
| `3.14` | `double` |
| `10e6` | `double` |
| `0.042F` | `float` |

Java 7 增加了在数字字面量中使用下划线字符（`_`）的能力。这通过在字面量的重要部分之间添加可视间距来增强代码的可读性。下划线可以几乎添加到数字字面量的任何位置。它可以与浮点数和任何整数基数（二进制、八进制、十六进制或十进制）一起使用。此外，还支持基数 2 字面量。

下表说明了在各种数字字面量中使用下划线的情况：

| 示例 | 用法 |
| --- | --- |
| `111_22_3333` | 社会安全号码 |
| `1234_5678_9012_3456` | 信用卡号码 |
| `0b0110_00_1` | 代表一个字节的二进制字面量 |
| `3._14_15F` | 圆周率 |
| `0xE_44C5_BC_5` | 32 位数量的十六进制字面量 |
| `0450_123_12` | 24 位八进制字面量 |

在代码中使用字面量对数字的内部表示或显示方式没有影响。例如，如果我们使用长整型字面量表示社会安全号码，该数字在内部以二进制补码表示，并显示为整数：

```java
long ssn = 111_22_3333L;
System.out.println(ssn);
```

输出如下：

```java
111223333

```

如果需要以社会安全号码的格式显示数字，需要在代码中进行。以下是其中一种方法：

```java
long ssn = 111_22_3333L;
String formattedSsn = Long.toString(ssn);
for (int i = 0; i < formattedSsn.length(); i++) {
    System.out.print(formattedSsn.charAt(i));
    if (i == 2 || i == 4) {
        System.out.print('-');
    }
}
System.out.println();
```

执行时，我们得到以下输出：

```java
111-22-3333

```

下划线的使用是为了使代码对开发人员更易读，但编译器会忽略它。

在使用文字中的下划线时，还有一些其他要考虑的事情。首先，连续的下划线被视为一个，并且也被编译器忽略。此外，下划线不能放置在：

+   在数字的开头或结尾

+   紧邻小数点

+   在`D`、`F`或`L`后缀之前

以下表格说明了下划线的无效用法。这些将生成语法错误：`非法下划线`：

| 例子 | 问题 |
| --- | --- |
| `_123_6776_54321L` | 不能以下划线开头 |
| `0b0011_1100_` | 不能以下划线结尾 |
| `3._14_15F` | 不能紧邻小数点 |
| `987_654_321_L` | 不能紧邻`L`后缀 |

一些应用程序需要操作值的位。以下示例将对一个值使用掩码执行位 AND 操作。掩码是一系列用于隔离另一个值的一部分的位。在这个例子中，`value`代表一个希望隔离最后四位的位序列。二进制文字代表掩码：

```java
value & 0b0000_11111;
```

当与包含零的掩码进行 AND 操作时，AND 操作将返回零。在前面的例子中，表达式的前四位将是零。最后四位与一进行 AND 操作，结果是结果的最后四位与值的最后四位相同。因此，最后四位已被隔离。

通过执行以下代码序列来说明：

```java
byte value = (byte) 0b0111_1010;
byte result = (byte) (value & 0b0000_1111);
System.out.println("result: " + Integer.toBinaryString(result));
```

执行时，我们得到以下输出：

```java
result: 1010

```

以下图表说明了这个 AND 操作：

![数字文字](img/7324_02_04.jpg)

#### 字符文字

字符文字是用单引号括起来的单个字符。

```java
char letter = 'a';
letter = 'F';
```

然而，一个或多个符号可以用来表示一个字符。反斜杠字符用于“转义”或赋予字母特殊含义。例如，`'\n'`代表回车换行字符。这些特殊的转义序列代表特定的特殊值。这些转义序列也可以在字符串文字中使用。转义序列字符列在下表中：

| 转义序列字符 | 含义 |
| --- | --- |
| `\a` | 警报（响铃） |
| `\b` | 退格 |
| `\f` | 换页 |
| `\n` | 换行 |
| `\r` | 回车 |
| `\t` | 水平制表符 |
| `\v` | 垂直制表符 |
| `\\` | 反斜杠 |
| `\?` | 问号 |
| `\'` | 单引号 |
| `\"` | 双引号 |
| `\ooo` | 八进制数 |
| `\xhh` | 十六进制数 |

#### 字符串文字

字符串文字是一系列用双引号括起来的字符。字符串文字不能跨两行分割：

```java
String errorMessage = "Error – bad input file name";
String columnHeader = "\tColumn 1\tColumn2\n";
```

### 常量

常量是其值不能改变的标识符。它们用于情况，其中应该使用更易读的名称而不是使用文字。在 Java 中，常量是通过在变量声明前加上`final`关键字来声明的。

在下面的例子中，声明了三个常量——`PI`、`NUMSHIPS`和`RATEOFRETURN`。根据标准*Java 命名约定*第第一章的*开始使用 Java*部分，每个常量都是大写的，并赋予一个值。这些值不能被改变：

```java
final double PI = 3.14159;
final int NUMSHIPS = 120;
final float RATEOFRETURN = 0.125F;
```

在下面的语句中，试图改变 PI 的值：

```java
PI = 3.14;
```

根据编译器的不同，将生成类似以下的错误消息：

```java
cannot assign a value to final variable PI

```

这意味着您不能改变常量变量的值。

### 注意

常量除了始终具有相同的值之外，还提供其他好处。常量数字或对象可以更有效地处理和优化。这使得使用它们的应用程序更有效和更易于理解。我们可以简单地使用`PI`而不是在需要的每个地方使用 3.14159。

### final 关键字

虽然`final`关键字用于声明常量，但它还有其他用途，如下表所述。我们将在后面的章节中介绍它在方法和类中的用法：

| 应用于 | 意义 |
| --- | --- |
| 原始数据声明 | 分配给变量的值无法更改。 |
| 引用变量 | 无法更改变量以引用不同的变量。但是，可能可以更改变量引用的对象。 |
| 方法 | 该方法无法被覆盖。 |
| 类 | 该类无法被扩展。 |

### 枚举

枚举实际上是`java.lang.Enum`类的子类。在本节中，我们将看一下简单枚举的创建。有关此主题的更完整处理，请参阅第六章中的*类，构造函数和方法*。

以下示例声明了一个名为`Directions`的枚举。此枚举表示四个基本点。

```java
public enum Directions {NORTH, SOUTH, EAST, WEST}
```

我们可以声明此类型的变量，然后为其分配值。以下代码序列说明了这一点：

```java
Directions direction;
direction = Directions.EAST;
System.out.println(direction);
```

此序列的输出如下：

```java
EAST

```

`enum`调用也可以作为 switch 语句的一部分，如下所示：

```java
switch(direction) {
case NORTH:
  System.out.println("Going North");
  break;
case SOUTH:
  System.out.println("Going South");
  break;
case EAST:
  System.out.println("Going East");
  break;
case WEST:
  System.out.println("Going West");
  break;
}
```

在与前面的代码一起执行时，我们得到以下输出：

```java
Going East

```

### 不可变对象

不可变对象是其字段无法修改的对象。在 Java 核心 SDK 中有几个类的对象是不可变的，包括`String`类。也许令人惊讶的是，`final`关键字并未用于此目的。这些将在第六章中详细讨论，*类，构造函数和方法*。

## 实例与静态数据

类中有两种不同类型的变量（数据）：实例和静态。当实例化对象（使用类名的`new`关键字）时，每个对象由组成该类的实例变量组成。但是，为每个类分配了静态变量的唯一副本。虽然每个类都有其自己的实例变量副本，但所有类共享静态变量的单个副本。这些静态变量分配到内存的一个单独区域，并且存在于类的生命周期内。

考虑添加一个可以选择性地应用于某些客户的常见折扣百分比，但不是所有客户。无论是否应用，百分比始终相同。基于这些假设，我们可以将静态变量添加到类中，如下所示：

```java
private static float discountPercentage;
```

静态方法和字段在第六章中有更详细的介绍，*类，构造函数和方法*。

## 范围和生命周期

范围指的是程序中特定变量可以使用的位置。一般来说，变量在其声明的块语句内可见，但在其外部不可见。块语句是由花括号封装的代码序列。

如果变量在范围内，则对代码可见并且可以访问。如果不在范围内，则无法访问变量，并且任何尝试这样做都将导致编译时错误。

变量的生命周期是指其分配了内存的时间段。当变量声明为方法的局部变量时，分配给变量的内存位于激活记录中。只要方法尚未返回，激活记录就存在，并且为变量分配内存。一旦方法返回，激活记录就从堆栈中移除，变量就不再存在，也无法使用。

从堆中分配的对象的生命周期始于分配内存时，终止于释放内存时。在 Java 中，使用`new`关键字为对象分配内存。当对象不再被引用时，对象及其内存被标记为释放。实际上，如果对象没有被回收，它将在未来的某个不确定的时间点被释放，如果有的话。如果一个对象没有引用，即使垃圾收集器尚未回收它，它也可以被使用或访问。

### 作用域规则

作用域规则对于理解诸如 Java 之类的块结构语言的工作方式至关重要。这些规则解释了变量何时可以使用，以及在命名冲突发生时将使用哪一个。

作用域规则围绕着块的概念。块由开放和闭合的大括号界定。这些块用于将代码分组在一起，并定义变量的范围。以下图表显示了三个变量`i`，`j`和`k`的范围：

![作用域规则](img/7324_02_05.jpg)

## 访问修饰符

在声明实例和静态变量和方法时，可以使用访问修饰符作为前缀。修饰符以各种组合应用以提供特定的行为。修饰符的顺序并不总是重要的，但一致的风格会导致更可读的代码。所有修饰符都是可选的，尽管有一些默认修饰符。访问修饰符包括：

+   `public`：公共对象对其自身类内外的所有方法可见。

+   `protected`：这允许在当前类和子类之间进行保护。受保护的对象在类外是不可见的，对子类完全可见。

+   `private`：私有变量只能被定义它的类（包括子类）看到。

+   **包**：这种可见性是默认保护。只有包内的类才有访问权限（包内公共）。

要解释变量的作用域，请考虑以下图表中显示的包/类组织，箭头表示继承：

![访问修饰符](img/7324_02_05a.jpg)

假设 A 类定义如下：

```java
public class A{
   public int  publicInt;
   private int privateInt;
   protected int  protectedInt;
   int defaultInt;  // default (package)
} 
```

所有变量都是`int`类型。`publicInt`变量是公共变量。它可以被这个类内外的所有方法看到。`privateInt`变量只在这个类内可见。`protectedInt`变量只对这个包内的类可见。`protectedInt`变量对这个类、它的子类和同一个包内的其他类可见。在其他地方是不可见的。以下表格显示了每种声明类型对每个类的可见性：

|   | A | B | C | D | E |
| --- | --- | --- | --- | --- | --- |
| `publicInt` | 可见 | 可见 | 可见 | 可见 | 可见 |
| `privateInt` | 可见 | 不可见 | 不可见 | 不可见 | 不可见 |
| `protectedInt` | 可见 | 可见 | 可见 | 不可见 | 可见 |
| `defaultInt` | 可见 | 可见 | 可见 | 不可见 | 不可见 |

## 数据摘要

以下表格总结了变量类型及其与 Java 编译时和运行时元素的关系：

| 程序元素 | 变量类型 | 的一部分 | 分配给 |
| --- | --- | --- | --- |
| 类 | 实例 | 对象 | 堆 |
| 静态 | 类 | 内存的特殊区域 |
| 方法 | 参数 | 激活记录 | 栈的激活记录 |
| 本地 |

# 使用操作数和运算符构建表达式

表达式由操作数和运算符组成。操作数通常是变量名或文字，而运算符作用于操作数。以下是表达式的示例：

```java
int numberWheels = 4;
System.out.println("Hello");
numberWheels = numberWheels + 1;
```

有几种分类运算符的方法：

+   算术

+   赋值

+   关系

+   逻辑补码

+   逻辑

+   条件

+   按位

表达式可以被认为是程序的构建块。它们用于表达程序的逻辑。

### 优先级和结合性

Java 运算符总结如下优先级和结合性表。这些运算符中的大多数都很简单：

| 优先级 | 运算符 | 结合性 | 运算符 |
| --- | --- | --- | --- |
| 1 | `++` | 右 | 前/后增量 |
| `--` | 右 | 前/后减量 |
| `+,-` | 右 | 一元加或减 |
| `~` | 右 | 位补 |
| `!` | 右 | 逻辑补 |
| (cast) | 右 | 强制转换 |
| 2 | `*`, `/`, 和 `%` | 左 | 乘法、除法和取模 |
| 3 | `+` 和 `-` | 左 | 加法和减法 |
| `+` | 左 | 字符串连接 |
| 4 | `<<` | 左 | 左移 |
| `>>` | 左 | 右移和符号填充 |
| `>>>` | 左 | 右移和零填充 |
| 5 | `<`, `<=`, `>`, `>=` | 左 | 逻辑 |
| `Instanceof` | 左 | 类型比较 |
| 6 | `==` 和 `!=` | 左 | 相等和不相等 |
| 7 | `&` | 左 | 位和布尔与 |
| 8 | `^` | 左 | 位和布尔异或 |
| 9 | ` | ` | 左 | 位和布尔或 |
| 10 | `&&` | 左 | 布尔与 |
| 11 | ` | | ` | 左 | 布尔或 |
| 12 | `?:` | 右 | 条件 |
| 13 | `=` | 右 | 赋值 |
| `+=`, `-=`, `*=`, `/=`, 和 `%=` | 右 | 复合 |

虽然大多数这些运算符的使用是直接的，但它们的更详细的用法示例将在后面的章节中提供。但请记住，在 Java 中没有其他变体和其他可用的运算符。例如，`+=`是一个有效的运算符，而`=+`不是。但是，它可能会带来意想不到的后果。考虑以下情况：

```java
total = 0;
total += 2;  // Increments total by 2
total =+ 2;  // Valid but simply assigns a 2 to total!
```

最后一条语句似乎使用了一个=+运算符。实际上，它是赋值运算符后面跟着的一元加运算符。一个`+2`被赋给`total`。请记住，Java 会忽略除了字符串文字之外的空格。

### 强制转换

当一种类型的数据被分配给另一种类型的数据时，可能会丢失信息。如果数据从更精确的数据类型分配到不太精确的数据类型，就可能会发生**缩小**。例如，如果浮点数`45.607`被分配给整数，小数部分`.607`就会丢失。

在进行此类分配时，应使用强制转换运算符。强制转换运算符只是您要转换为的数据类型，括在括号中。以下显示了几个显式转换操作：

```java
int i;
float f = 1.0F;
double d = 2.0;

i = (int) f;  // Cast a float to an int
i = (int) d;  // Cast a double to an int
f = (float) d;  // Cast a double to a float
```

在这种情况下，如果没有使用强制转换运算符，编译器将发出警告。警告是为了建议您更仔细地查看分配情况。精度的丢失可能是一个问题，也可能不是，这取决于应用程序中数据的使用。没有强制转换运算符，当代码执行时会进行隐式转换。

# 处理字符和字符串

主要类包括`String`、`StringBuffer`、`StringBuilder`和`Character`类。还有几个与字符串和字符操作相关的其他类和接口，列举如下，您应该知道。但并非所有这些类都将在此处详细说明。

+   `Character`：这涉及到字符数据的操作

+   `Charset`：这定义了 Unicode 字符和字节序列之间的映射

+   `CharSequence`：在这里，一个接口由`String`、`StringBuffer`和`StringBuilder`类实现，定义了公共方法

+   `StringTokenizer`：这用于对文本进行标记化

+   `StreamTokenizer`：这用于对文本进行标记化

+   `Collator`：这用于支持特定区域设置字符串的操作

### String、StringBuffer 和 StringBuilder 类

对于 Java 程序员，有几个与字符串相关的类可用。在本节中，我们将研究 Java 中用于操作此类数据的类和技术。

在 JDK 中用于字符串操作的三个主要类是`String`、`StringBuffer`和`StringBuilder`。`String`类是这些类中最广泛使用的。`StringBuffer`和`StringBuilder`类是在 Java 5 中引入的，以解决`String`类的效率问题。`String`类是不可变的，需要频繁更改字符串的应用程序将承受创建新的不可变对象的开销。`StringBuffer`和`StringBuilder`类是可变对象，当字符串需要频繁修改时可以更有效地使用。`StringBuffer`与`StringBuilder`的区别在于它的方法是同步的。

在类支持的方法方面，`StringBuffer`和`StringBuilder`的方法是相同的。它们只在方法是否同步上有所不同。

| 类 | 可变 | 同步 |
| --- | --- | --- |
| `String` | 否 | 否 |
| `StringBuffer` | 是 | 是 |
| `StringBuilder` | 是 | 否 |

当处理使用多个线程的应用程序时，同步方法是有用的。**线程**是一个独立执行的代码序列。它将与同一应用程序中的其他线程同时运行。并发线程不会造成问题，除非它们共享数据。当这种情况发生时，数据可能会变得损坏。同步方法的使用解决了这个问题，并防止数据由于线程的交互而变得损坏。

同步方法的使用包括一些开销。因此，如果字符串不被多个线程共享，则不需要`StringBuffer`类引入的开销。当不需要同步时，大多数情况下应该使用`StringBuilder`类。

### 注意

**使用字符串类的标准**

如果字符串不会改变，请使用`String`类：

+   由于它是不可变的，因此可以安全地在多个线程之间共享

+   线程只会读取它们，这通常是一个线程安全的操作。

如果字符串将要改变并且将在线程之间共享，则使用`StringBuffer`类：

+   这个类是专门为这种情况设计的

+   在这种情况下使用这个类将确保字符串被正确更新

+   主要缺点是方法可能执行得更慢

如果字符串要改变但不会在线程之间共享，请使用`StringBuilder`类：

+   它允许修改字符串，但不会产生同步的开销

+   这个类的方法将执行得和`StringBuffer`类一样快，甚至更快

### Unicode 字符

Java 使用 Unicode 标准来定义字符。然而，这个标准已经发展和改变，而 Java 已经适应了它的变化。最初，Unicode 标准将字符定义为一个 2 字节 16 位值，可以使用可打印字符或`U+0000`到`U+FFFF`来表示。无论可打印与否，十六进制数字都可以用来编码 Unicode 字符。

然而，2 字节编码对于所有语言来说都不够。因此，Unicode 标准的第 4 版引入了新的字符，位于`U+FFFF`以上，称为**UTF-16**（**16 位 Unicode 转换格式**）。为了支持新标准，Java 使用了**代理对**的概念——16 位字符对。这些对用于表示从`U+10000`到`U+10FFFF`的值。代理对的前导或高值范围从`U+D800`到`U+DBFF`。对的尾部或低值范围从`U+DC00`到`U+DFFF`。这些范围内的字符称为**补充字符**。这两个特殊范围用于将任何 Unicode 字符映射到代理对。从 JDK 5.0 开始，一个字符使用 UTF-16 表示。

## 字符类

`Character`类是`char`原始数据类型的包装类。该数据类型支持 Unicode 标准版本 4.0。字符被定义为固定宽度的 16 位数量。

### 字符类-方法

字符串类

| 方法 |
| --- |
| `String`类是 Java 中用于表示字符串的常见类。它是不可变的，这使得它是线程安全的。也就是说，多个线程可以访问同一个字符串，而不用担心破坏字符串。不可变还意味着它是固定大小的。 |
| 如果字符是数字，则返回 true |
| 如果字符是字母，则返回 true |
| 如果字符是字母或数字，则返回 true |
| 如果字符是小写字母，则返回 true |
| 如果字符是空格，则返回 true |
| 如果字符是大写字母，则返回 true |
| 返回字符的小写等价物 |
| 描述 |

## 返回字符的大写等价物

`Character`类具有处理字符的多种方法。许多`Character`方法都是重载的，可以接受`char`或 Unicode 代码点参数。代码点是用于字符的抽象，对于我们的目的是 Unicode 字符。以下表列出了您可能会遇到的几种`Character`方法：

`String`类被设计为不可变的一个原因是出于安全考虑。如果一个字符串用于标识受保护的资源，一旦为该资源授予权限，可能会修改字符串然后获取对用户没有权限的另一个资源的访问权限。通过使其不可变，可以避免这种漏洞。

虽然`String`类是不可变的，但它可能看起来是可变的。考虑以下示例：

```java
String s = "Constant";
s = s + " and unchangeable";
System.out.println(s);
```

输出这个序列的结果是字符串"Constant and unchangeable"。由于`s`被定义为`String`类型，因此由`s`标识符引用的对象不能改变。当进行第二个赋值语句时，将创建一个新对象，将`Constant`和`and unchangeable`组合在一起，生成一个新的字符串`Constant and unchangeable`。在这个过程中创建了三个`String`对象：

+   常量

+   不可改变

+   不可改变

标识符`s`现在引用新的字符串`Constant and unchangeable`。

虽然我们可以访问这些对象，但我们无法更改它们。我们可以访问和读取它们，但不能修改它们。

我们本可以使用`String`类的`concat`方法，但这并不那么直接：

```java
s = "Constant";
s = s.concat(" and unchangeable");
System.out.println(s);
```

以下代码演示了创建`String`对象的几种技术。第一个构造函数只会产生一个空字符串。除非应用程序需要在堆上找到一个空的不可变字符串，否则这对于立即价值不大。

```java
String firstString = new String();
String secondString = new String("The second string");
String thirdString = "The third string";
```

此外，还有两个使用`StringBuffer`和`StringBuilder`类的构造函数。从这些对象创建了新的`String`对象，如下代码序列所示：

```java
StringBuffer stringBuffer =new StringBuffer("A StringBuffer string");
StringBuilder stringBuilder =new StringBuilder("A StringBuilder string");
String stringBufferBasedString = new String(stringBuffer);
String stringBuilderBasedString = new String(stringBuilder);
```

### 注意

在内部，`String`类的字符串表示为`char`数组。

### 字符串比较

字符串比较并不像最初看起来那么直接。如果我们想要比较两个整数，我们可能会使用如下语句：

```java
if (count == max) {
  // Do something
}
```

然而，对于两个字符串的比较，比如`s1`和`s2`，以下通常会评估为`false`：

```java
String s1 = "street";
String s2;

s2 = new String("street");

if (s1 == s2) {
  // False
}
```

问题在于变量`s1`和`s2`可能引用内存中的不同对象。if 语句比较字符串引用变量而不是实际的字符串。由于它们引用不同的对象，比较返回`false`。这完全取决于编译器和运行时系统如何在内部处理字符串。

当使用`new`关键字时，内存是从堆中分配并分配给新对象。但是，在字符串文字的情况下，这个内存不是来自堆，而是来自文字池，更具体地说，是字符串内部池。在 Java 中，内部化的字符串被放置在 JVM 的永久代区域中。该区域还存储 Java 类声明和类静态变量等内容。

内部化字符串仅存储每个不同字符串的一个副本。这是为了改善某些字符串方法的执行并减少用于表示相同字符串的空间量。此区域中的字符串会受到垃圾回收的影响。

例如，如果我们创建两个字符串文字和一个使用`new`关键字的`String`对象：

```java
String firstLiteral = "Albacore Tuna";
String secondLiteral = "Albacore Tuna";
String firstObject = new String("Albacore Tuna");

if(firstLiteral == secondLiteral) {
  System.out.println(
     "firstLiteral and secondLiteral are the same object");
} else {
  System.out.println(
     "firstLiteral and secondLiteral are not the same object");
}
if(firstLiteral == firstObject) {
  System.out.println(
     "firstLiteral and firstObject are the same object");
} else {
  System.out.println(
     "firstLiteral and firstObject are not the same object");
}
```

输出如下：

```java
firstLiteral and secondLiteral are the same object
firstLiteral and firstObject are not the same object

```

`String`类的`intern`方法可用于对字符串进行内部化。对于所有常量字符串，内部化是自动执行的。在比较内部化的字符串时，可以使用等号运算符，而不必使用`equals`方法。这可以节省对字符串密集型应用程序的时间。很容易忘记对字符串进行内部化，因此在使用等号运算符时要小心。除此之外，`intern`方法可能是一个昂贵的方法。

### 注意

Java 还会对`String`类型之外的其他对象进行内部化。这些包括包装对象和小整数值。当使用原始类型的字符串连接运算符时，可能会产生包装对象。有关更多详细信息，请访问[`docs.oracle.com/javase/specs/jls/se7/jls7.pdf`](http://docs.oracle.com/javase/specs/jls/se7/jls7.pdf)，并参考 5.1.7 和 12.5 节。

要执行`String`比较，可以使用一系列`String`方法，包括但不限于以下内容：

| 方法 | 目的 |
| --- | --- |
| `equals` | 比较两个字符串，如果它们等效，则返回`true` |
| `equalsIgnoreCase` | 忽略字母大小写比较两个字符串，如果它们等效，则返回`true` |
| `startsWith` | 如果字符串以指定的字符序列开头，则返回`true` |
| `endsWith` | 如果字符串以指定的字符序列结尾，则返回`true` |
| `compareTo` | 如果第一个字符串在第二个字符串之前，则返回`-1`，如果它们相等，则返回`0`，如果第一个字符串在第二个字符串之后，则返回`1` |

### 注意

记住字符串从索引`0`开始。

以下是使用各种字符串比较的示例：

```java
String location = "Iceberg City";
if (location.equals("iceberg city"))
  System.out.println(location + " equals ' city'!");
else
  System.out.println(location +" does not equal 'iceberg city'");

if (location.equals("Iceberg City"))
  System.out.println(location + " equals 'Iceberg City'!");
else
  System.out.println(location +" does not equal 'Iceberg City'!");

if (location.endsWith("City"))
  System.out.println(location + " ends with 'City'!");
else
  System.out.println(location + " does not end with 'City'!");
```

输出如下所示：

```java
Iceberg City does not equal 'iceberg city'
Iceberg City equals 'Iceberg City'!
Iceberg City ends with 'City'!

```

在使用此方法时有几件事情需要考虑。首先，大写字母在小写字母之前。这是它们在 Unicode 中的排序结果。ASCII 也适用相同的排序。

一个字符串可以有多个内部表示。许多语言使用重音来区分或强调字符。例如，法国名字 Irène 使用重音，可以表示为`I` `r` `è` `n` `e`或序列`I` `r` `e` ```java `n` `e`. The second sequence combines the `e` and ```以形成字符`è`。如果使用`equals`方法比较这两种不同的内部表示，该方法将返回`false`。在这个例子中，`\u0300`将重音与字母`e`组合在一起。

String firstIrene = "Irène";

```java
String secondIrene = "Ire\u0300ne";

if (firstIrene.equals(secondIrene)) {
    System.out.println("The strings are equal.");
} else {
    System.out.println("The strings are not equal.");
}
```

此代码序列的输出如下：

```java
The strings are not equal.

```

`Collator`类可用于以特定于区域设置的方式操作字符串，消除了不同内部字符串表示的问题。

### 基本字符串方法

您可能会遇到几种`String`方法。这些在下表中有所说明：

| 方法 | 目的 |
| --- | --- |
| `length` | 返回字符串的长度。 |
| `charAt` | 返回字符串中给定索引的字符的位置。 |
| `substring` | 此方法是重载的，返回字符串的部分。 |
| `indexOf` | 返回字符或字符串的第一次出现的位置。 |
| `lastIndexOf` | 返回字符或字符串的最后一次出现的位置。 |

以下示例说明了这些方法的使用：

```java
String sample = "catalog";
System.out.println(sample.length());
System.out.println(sample.charAt(0));
System.out.println(sample.charAt(sample.length()-1));
System.out.println(sample.substring(0,3));
System.out.println(sample.substring(4));
```

执行此代码时，我们得到以下输出：

```java
7
c
g
cat
log

```

在许多应用程序中，搜索字符串以查找字符或字符序列是常见的需求。`indexOf`和`lastIndex`方法执行此类操作：

```java
String location = "Irene";
System.out.println(location.indexOf('I'));
System.out.println(location.lastIndexOf('e'));
System.out.println(location.indexOf('e'));
```

这些语句的结果如下：

```java
0
4
2

```

您可以将字符串中的位置视为字符之前的位置。这些位置或索引从`0`开始，如下图所示：

![基本字符串方法](img/7324_02_06.jpg)

### 字符串长度

字符串长度的计算可能比简单使用`length`方法所建议的要复杂一些。它取决于正在计数的内容以及字符串在内部的表示方式。

用于确定字符串长度的方法包括：

+   `length`：标准方法

+   `codePointCount`：与补充字符一起使用

+   字节数组的`length`方法：用于确定用于保存字符串的实际字节数

在存储字符串时，字符串的实际长度（以字节为单位）可能很重要。数据库表中分配的空间量可能需要比字符串中的字符数更长。

### 数字/字符串转换

将数字转换为字符串的过程很重要。我们可以使用两种方法。第一种方法使用静态方法，如下代码序列所示。`valueOf`方法将数字转换为字符串：

```java
String s1 = String.valueOf(304);
String s2 = String.valueOf(778.204);
```

`intValue`和`doubleValue`方法接受`valueOf`静态方法返回的对象，并分别返回整数或双精度数：

```java
int  num1 = Integer.valueOf("540").intValue();
double  num2 = Double.valueOf("3.0654").doubleValue();
```

第二种方法是使用各自的包装类的`parseInt`和`parseDouble`方法。它们的使用如下所示：

```java
num1 = Integer.parseInt("540");
num2 = Double.parseDouble("3.0654");
```

### 杂项字符串方法

有几种杂项方法可能会有用：

+   `replace`：这将字符串的一部分替换为另一个字符串

+   `toLowerCase`：将字符串中的所有字符转换为小写

+   `toUpperCase`：将字符串中的所有字符转换为大写

+   `trim`：删除前导和尾随空格

以下是这些方法的使用示例：

```java
String oldString = " The gray fox ";
String newString;

newString = oldString.replace(' ','.');
System.out.println(newString);

newString = oldString.toLowerCase();
System.out.println(newString);

newString = oldString.toUpperCase();
System.out.println(newString);

newString = oldString.trim();
System.out.println("[" + newString +"]" );
```

结果如下所示：

```java
.The.gray.fox.
 the gray fox
 THE GRAY FOX
[The gray fox]

```

## StringBuffer 和 StringBuilder 类

`StringBuffer`和`StringBuilder`类提供了`String`类的替代方法。与`String`类不同，它们是可变的。这在使程序更有效时有时是有帮助的。有几种常用的方法可用于操作`StringBuffer`或`StringBuilder`对象。以下示例中演示了其中几种。虽然示例使用`StringBuffer`类，但`StringBuilder`方法的工作方式相同。

经常需要将一个字符串附加到另一个字符串。可以使用`append`方法来实现这一点：

```java
StringBuffer buffer = new StringBuffer();
buffer.append("World class");
buffer.append(" buffering mechanism!");
```

以下是将字符串插入缓冲区的示例：

```java
buffer.insert(6,"C");
```

更详细的示例：

```java
StringBuffer buffer;
buffer = new StringBuffer();
buffer.append("World lass");
buffer.append(" buffering mechanism!");
buffer.insert(6,"C");
System.out.println(buffer.toString());
```

结果如下：

```java
World Class buffering mechanism!

```

# 摘要

在本章中，我们已经研究了 Java 如何处理数据。堆栈和堆的使用是重要的编程概念，可以很好地解释变量的作用域和生命周期等概念。介绍了对象和原始数据类型之间的区别以及变量的初始化。初始化过程将在第六章*类，构造函数和方法*中更详细地介绍。列出了 Java 中可用的运算符以及优先级和结合性规则。此外，还介绍了字符和字符串数据的操作。

在下一章中，我们将探讨 Java 中可用的决策结构以及它们如何有效地使用。这将建立在此处介绍的数据类型之上。

# 涵盖的认证目标

在本章中，我们涵盖了以下内容：

+   了解 Java 如何处理数据

+   调查标识符、Java 类和内存之间的关系

+   定义变量的范围

+   初始化标识符

+   使用运算符和操作数构建表达式

+   处理字符串

+   理解对象和原始数据类型之间的区别

# 测试你的知识

1.  当编译和运行以下代码时会发生什么？

```java
public class ScopeClass{
   private int i = 35;
   public static void main(String argv[]){
      int i = 45;
      ScopeClass s = new ScopeClass ();
      s.someMethod();
   }
   public static void someMethod(){
      System.out.println(i);
   }
}
```

a. 35 将被打印出来

b. 45 将被打印出来

c. 将生成编译时错误

d. 将抛出异常

1.  以下哪行将会编译而不会产生警告或错误？

a. `char d="d";`

b. `float f=3.1415;`

c. `int i=34;`

d. `byte b=257;`

e. `boolean isPresent=true;`

1.  给出以下声明：

```java
public class SomeClass{
   public int i;
   public static void main(String argv[]){
      SomeClass sc = new SomeClass();
      // Comment line
   }
}
```

如果它们替换注释行，以下哪些陈述是正确的？

a. `System.out.println(i);`

b. `System.out.println(sc.i);`

c. `System.out.println(SomeClass.i);`

d. `System.out.println((new SomeClass()).i);`

1.  给出以下声明：

```java
StringBuilder sb = new StringBuilder;
```

以下哪些是`sb`变量的有效用法？

a. `sb.append(34.5);`

b. `sb.deleteCharAt(34.5);`

c. `sb.toInteger` `(3);`

d. `sb.toString();`

1.  以下哪个将返回字符串 s 中包含“banana”的第一个字母`a`的位置？

a. `lastIndexOf(2,s);`

b. `s.indexOf('a');`

c. `s.charAt(` `2);`

d. `indexOf(s,'v');`

1.  给出以下代码，哪个表达式显示单词“Equal”？

```java
String s1="Java";
String s2="java";
if(expression) {
   System.out.println("Equal");
} else {
   System.out.println("Not equal");
}
```

a. `s1==s2`

b. `s1.matchCase(s2)`

c. `s1.equalsIgnoreCase(s2)`

d. `s1.equals(s2)`
