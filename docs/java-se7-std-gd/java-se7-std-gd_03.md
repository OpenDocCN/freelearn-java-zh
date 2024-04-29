# 第三章。决策结构

每个应用程序都会做出某种决定。在 Java 中，有几种编程构造可以用来做出这些决定。这些包括逻辑表达式、if 语句和 switch 语句。本章的目的是向您介绍这些工具，并说明它们如何使用。

我们将从逻辑表达式的讨论开始，因为它们对做决定至关重要。逻辑表达式是返回布尔值的表达式。

接下来，我们将研究逻辑表达式如何与`if`语句和条件运算符一起使用。`if`语句的结构有许多变化，我们将看看它们的优缺点。

接下来将讨论`switch`语句。在 Java 7 之前，`switch`语句基于整数或枚举值。在 Java 7 中，我们现在可以使用`String`值。我们将研究字符串的使用及其潜在的问题。

最后一节讨论了一般的控制结构问题，以及在做决定、比较对象时浮点数的影响，以及组织代码的有用方法的讨论。

在本章中，我们将：

+   研究决策结构的性质

+   研究逻辑表达式的基础知识

+   学习如何使用`if`语句，并查看其变体

+   了解条件运算符以及何时应该使用它

+   探索`switch`语句和 Java 7 在该语句中使用字符串的方式

+   确定浮点数比较如何影响控制

+   研究与比较对象相关的潜在问题

# 控制流

在任何应用程序中，程序内的控制流由语句执行的顺序决定。方便起见，可以将语句组视为由决策语句控制执行顺序的块。块可以被视为一个单独的语句或包含在块语句中的多个语句。在 Java 中，块语句是一组用大括号括起来的语句。

## 控制语句 - 概述

控制结构是语言中确定个别语句执行顺序的部分。没有控制结构，语句将按顺序执行，如下面的代码片段所示：

```java
hours ==35;
payRate = 8.55;
pay = hours * payRate;
System.out.println(pay);
```

为了改变语句的执行顺序，使用控制语句。在 Java 中，这些语句包括：

+   `if`语句：这个语句经常用于决定采取哪个分支

+   条件运算符：这个语句是`if`语句的简化和有限形式

+   `switch`语句：这个语句用于决定采取哪个分支

`switch`语句使用整数、枚举或字符串值来做出决定。要理解`if`语句需要理解逻辑表达式。这将在下一节中介绍。

# 逻辑表达式

与所有表达式一样，逻辑表达式由运算符和操作数组成。在 Java 中，有限数量的逻辑运算符如下表所总结的那样。它是第二章中列出的运算符的子集，*Java 数据类型及其用法*：

| 优先级 | 运算符 | 结合性 | 意义 |
| --- | --- | --- | --- |
| 1 | … |   |   |
| `!` | 右 | 逻辑补 |
| … |   |   |
| … |   |   |   |
| 5 | `<`, `<=`, `>`, 和 `>=` | 左 | 逻辑 |
| `instanceof` | 左 | 类型比较 |
| 6 | `==` 和 `!=` | 左 | 相等和不相等 |
| … |   |   |   |
| 10 | `&&` | 左 | 逻辑与 |
| 11 | ` | | ` | 左 | 逻辑或 |
| 12 | `?:` | 右 | 条件 |
| … |   |   |   |

逻辑表达式的操作数可以是任何数据类型，但逻辑表达式总是评估为`true`或`false`值。

### 注意

不要将按位运算符`&`、`^`和`|`与相应的逻辑运算符`&&`和`||`混淆。按位运算符执行与逻辑运算符类似的操作，但是逐位执行。

## 布尔变量

`true`和`false`是 Java 中的关键字。它们的名称对应它们的值，并且可以赋给布尔变量。布尔变量可以用`boolean`关键字声明，后面跟着变量名和可选的初始值：

```java
boolean isComplete;
boolean isReady = true;  // Initialized to true
boolean errorPresent;
```

当逻辑表达式被评估时，它将返回`true`或`false`值。逻辑表达式的示例包括以下内容：

```java
age > 45
age > 45 && departmentNumber == 200
((flowRate > minFlowRate) || ((flowRate > maxFlowRate) && (valveA == off)))
```

给布尔变量一个反映`true`或`false`状态的名称是一个好习惯。`isComplete`变量意味着一个操作已经完成。如果`isReady`变量设置为 true，则表示某物已经准备好。

## 等于运算符

等于运算符由两个等号组成，当评估时将返回`true`或`false`值。赋值运算符使用单个等号，并将修改其左操作数。为了说明这些运算符，考虑以下示例。如果`rate`变量的值等于`100`，我们可以假设存在错误。为了反映这种错误条件，我们可以将`true`值赋给`errorPresent`变量。这可以使用赋值和等于运算符来执行。

```java
int rate;
rate = 100;
boolean errorPresent = rate==100;
System.out.println(errorPresent);
```

当执行前面的代码片段时，我们得到以下输出：

```java
true

```

逻辑表达式`rate==100`比较存储在`rate`中的值与整数文字`100`。如果它们相等，这里就是这种情况，表达式返回`true`。然后将`true`值赋给`errorPresent`。如果存储在`rate`中的值不是`100`，则表达式将返回`false`值。我们将更深入地研究等于运算符在*比较浮点数*和*比较对象*部分的使用。

## 关系运算符

关系运算符用于确定两个操作数之间的关系或相对顺序。这些运算符经常使用两个符号。例如，大于或等于使用`>=`运算符表示。符号的顺序很重要。使用`=>`是不合法的。

关系运算符列在下表中：

| 操作符 | 意义 | 简单示例 |
| --- | --- | --- |
| --- | --- | --- |
| 小于 | `<` | `age<35` |
| 小于或等于 | `<=` | `age<=35` |
| 大于 | `>` | `age>35` |
| 大于或等于 | `>=` | `age>=35` |
| 等于 | `==` | `age==35` |

如果我们希望确定一个年龄是否大于 25 且小于 35，我们将不得不两次使用`age`变量，并与`&&`运算符结合使用，如下所示：

```java
age > 25 && age < 35
```

虽然以下表达式对我们可能有意义，但在 Java 中是不合法的。

```java
25 < age < 35
```

之所以在前面的示例中变量`age`必须使用两次，是因为关系运算符是二元运算符。也就是说，每个二元运算符作用于两个操作数。在前面的表达式中，我们比较`25`，看它是否小于`age`。操作将返回`true`或`false`值。接下来，将 true 或 false 结果与`35`进行比较，这是没有意义的，也是不合法的。

这些是语言的规则。我们不能违反这些规则，因此重要的是我们理解这些规则。

## 逻辑运算符

当我们考虑如何做决定时，我们经常使用逻辑结构，比如 AND 和 OR。如果两个条件都为真，我们可能会做出决定，或者如果两个条件中的任何一个为真，我们可能会决定做某事。AND 运算符意味着两个条件都必须为真，而 OR 意味着两个条件中只有一个需要为真。

这两个操作是大多数逻辑表达式的基础。我们经常会决定在某些条件不成立时做一些事情。如果下雨，我们可能决定不遛狗。NOT 也是用于做决定的运算符。使用时，它将 true 变为 false，false 变为 true。

在 Java 中有三个逻辑运算符来实现这些逻辑结构。它们总结如下表：

| 运算符 | 含义 | 简单示例 |
| --- | --- | --- |
| `&&` | AND | `age > 35 && height < 67` |
| `&#124;&#124;` | OR | `age > 35 &#124;&#124; height < 67` |
| `!` | NOT | `!(age > 35)` |

AND、OR 和 NOT 运算符基于以下真值表：

![逻辑运算符](img/7324_02_07.jpg)

一些决策可能更加复杂，我们使用`&&`、`||`或`!`运算符的更复杂的组合来表达这些决策评估。如果下雨，我们可能决定去看电影，如果我们有足够的钱或朋友将付我们的路费。

如果（不下雨）并且

（（我们有足够的钱）或（朋友会付我们的路费））然后

我们将去看电影

括号可以用来控制逻辑运算符的评估顺序，就像它们控制算术运算符的评估顺序一样。在下面的代码序列中，错误的存在是由`rate`和`period`变量中存储的值决定的。这些语句是等价的，但在使用括号的方式上有所不同。在第二个语句中使用括号并不是严格需要的，但它确实使其更清晰：

```java
errorPresent = rate == 100 || period > 50;
errorPresent = (rate == 100) || (period > 50);
```

在下面的语句中，使用一组括号来强制执行`||`运算符先于`&&`运算符。由于`&&`运算符的优先级高于`||`运算符，我们需要使用括号来改变评估顺序：

```java
errorPresent = ((period>50) || (rate==100)) && (yaw>56);
```

括号始终优先于其他运算符。

## 短路评估

**短路**是一种在结果变得明显后不完全评估逻辑表达式的过程。在 Java 中有两个运算符可以进行短路——逻辑`&&`和`||`运算符。

### 使用`&&`运算符

首先考虑逻辑`&&`运算符。在下面的例子中，我们试图确定`sum`是否大于`1200`且`amount`是否小于`500`。为了逻辑表达式返回 true，必须满足两个条件：

```java
if (sum > 1200 && amount <500)...
```

然而，如果第一个条件为 false，则没有理由评估表达式的其余部分。无论第二个条件的值如何，`&&`运算符都将返回 false。通过短路，第二个条件不会被评估，尤其是在操作耗时的情况下，可以节省一些处理时间。

我们可以通过使用以下两个函数来验证这种行为。它们都返回`false`值并在执行时显示消息：

```java
private static boolean evaluateThis() {
    System.out.println("evaluateThis executed");
    return false;
}
private static boolean evaluateThat() {
    System.out.println("evaluateThat executed");
    return false;
}
```

接下来，我们在`if`语句中使用它们，如下所示：

```java
if(evaluateThis() && evaluateThat()) {
    System.out.println("The result is true");
} else {
    System.out.println("The result is false");
}
```

当我们执行前面的代码序列时，我们得到以下输出：

```java
evaluateThis executed
The result is false

```

`evaluateThis`方法执行并返回`false`。由于它返回`false`，`evaluateThat`方法没有被执行。

### 使用`||`运算符

逻辑`||`运算符的工作方式类似。如果第一个条件评估为`true`，就没有理由评估第二个条件。这在下面的代码序列中得到了证明，其中`evaluateThis`方法已被修改为返回`true`：

```java
private static boolean evaluateThis() {
    System.out.println("evaluateThis executed");
    return true;
}

    ...

if(evaluateThis() || evaluateThat()) {
    System.out.println("The result is true");
} else {
    System.out.println("The result is false");
}
```

执行此代码序列将产生以下输出：

```java
evaluateThis executed
The result is true

```

### 避免短路评估

通常，短路表达式是一种有效的技术。然而，如果我们像在上一个例子中那样调用一个方法，并且程序依赖于第二个方法的执行，这可能会导致意想不到的问题。假设我们已经编写了`evaluateThat`方法如下：

```java
private static boolean evaluateThat() {
   System.out.println("evaluateThat executed");
   state = 10;
   return false;
}
```

当逻辑表达式被短路时，`state`变量将不会被改变。如果程序员错误地假设`evaluateThat`方法总是会被执行，那么当分配给`state`的值不正确时，这可能导致逻辑错误。

据说`evaluateThat`方法具有副作用。人们可以争论是否使用具有副作用的方法是一种良好的实践。无论如何，您可能会遇到使用副作用的代码，您需要了解其行为。

避免逻辑表达式短路的一种替代方法是使用按位与（`&`）和按位或（`|`）运算符。这些按位运算符对操作数的每个位执行`&&`或`||`运算。由于关键字`true`和`false`的内部表示使用单个位，因此结果应该与相应的逻辑运算符返回的结果相同。不同之处在于不执行短路操作。

使用前面的例子，如果我们使用`&`运算符而不是`&&`运算符，如下面的代码片段所示：

```java
if (evaluateThis() & evaluateThat()) {
   System.out.println("The result is true");
} else {
   System.out.println("The result is false");
}
```

当我们执行代码时，我们将得到以下输出，显示两种方法都被执行：

```java
evaluateThis executed
evaluateThat executed
The result is false

```

# if 语句

`if`语句用于根据布尔表达式控制执行流程。有两种基本形式可以使用，还有几种变体。`if`语句由`if`关键字组成，后面跟着用括号括起来的逻辑表达式，然后是一个语句。在下图中，呈现了一个简单`if`语句的图形描述：

![if 语句](img/7324_03_01.jpg)

以下说明了`if`语句的这种形式，我们比较`rate`和`100`，如果它等于`100`，我们显示相应的消息：

```java
if (rate==100) System.out.println("rate is equal to 100");
```

然而，这不如下面的等效示例易读，我们将`if`语句分成两行：

```java
if (rate == 100) 
   System.out.println("rate is equal to 100");
```

我们将在后面看到，最好总是在`if`语句中使用块语句。以下逻辑上等同于先前的`if`语句，但更易读和易维护：

```java
if (rate == 100) {
    System.out.println("rate is equal to 100");
}
```

`if`语句的第二种形式使用`else`关键字来指定逻辑表达式评估为`false`时要执行的语句。以下图表形象地说明了`if`语句的这个版本：

![if 语句](img/7324_03_02.jpg)

使用前面的例子，`if`语句如下所示：

```java
if (rate == 100) {
   System.out.println("rate is equal to 100");
} else {
   System.out.println("rate is not equal to 100");
}
```

如果表达式评估为`true`，则执行第一个块，然后控制传递到`if`语句的末尾。如果表达式评估为`false`，则执行第二个块。在这个例子中，每个块都包含一个语句，但不一定非要这样。块内可以使用多个语句。选择使用多少语句取决于我们要做什么。

`if`语句的简化形式消除了`else`子句。假设我们想在超过某个限制时显示错误消息，否则什么也不做。这可以通过不使用`else`子句来实现，如下面的代码片段所示：

```java
if (amount > limit) {
  System.out.println("Your limit has been exceeded");
}
```

只有在超过限制时才显示消息。请注意块语句的使用。即使它只包含一个语句，使用它仍然是一个良好的实践。如果我们决定需要做的事情不仅仅是显示错误消息，比如更改限制或重置金额，那么我们将需要一个块语句。最好做好准备：

一些开发人员不喜欢这种简化形式，他们总是使用 else 子句。

```java
if (amount > limit) {
  System.out.println("Your limit has been exceeded");
} else {
  // Do nothing
}
```

`Do nothing`注释用于记录`else`子句。如果我们决定实际做一些事情，比如下订单，那么这就是我们将添加代码的地方。通过使用显式的`else`子句，我们至少需要考虑可能或应该在那里做什么。

您还可能遇到**空语句**。这个语句由一个分号组成。执行时，它什么也不做。它通常用作占位符，表示不需要做任何事情。在下面的代码片段中，修改了前一个`if`语句，以使用空语句：

```java
if (amount > limit) {
   System.out.println("Your limit has been exceeded");
} else {
   ;    // Do nothing
}
```

这并没有给`if`语句增加任何东西，这里使用它也不是问题。在第五章中，*循环结构*，我们将研究如何粗心地使用空语句可能会导致问题。

## 嵌套 if 语句

在彼此内部嵌套`if`语句提供了另一种决策技术。如果`if`语句被包含在另一个`if`语句的`then`或`else`子句中，则称为嵌套`if`语句。在下面的例子中，一个`if`语句在第一个`if`语句的`then`子句中找到：

```java
if (limitIsNotExceeded) {
   System.out.println("Ready");
   if (variationIsAcceptable) {
      System.out.println(" to go!");
   } else {
      System.out.println(" – Not!");
   }
   // Additional processing
} else {
   System.out.println("Not Ok");
}
```

嵌套`if`的使用没有限制。它可以在`then`或`else`子句中。此外，它们可以嵌套的深度也没有限制。我们可以把`if`放在`if`的内部，依此类推。

## else-if 变体

在一些编程语言中，有一个`elseif`关键字，提供了一种实现多选择`if`语句的方法。从图形上看，这个语句的逻辑如下图所示：

![else-if 变体](img/7324_03_03.jpg)

Java 没有`elseif`关键字，但可以使用嵌套的 if 语句来实现相同的效果。假设我们想要计算一个取决于我们要运送到国家的四个地区中的哪一个而定的运费——东部、北部中部、南部中部或西部。我们可以使用一系列`if`语句来实现这一点，其中每个`if`语句实际上都嵌套在前一个`if`语句的`else`子句中。第一个评估为 true 的`if`语句将执行其主体，其他`if`语句将被忽略：

```java
if (zone.equals("East")) {
   shippingCost = weight * 0.23f;
} else if (zone.equals("NorthCentral")) {
   shippingCost = weight * 0.35f;
} else if (zone.equals("SouthCentral")) {
   shippingCost = weight * 0.17f;
} else {
   shippingCost = weight * 0.25f;
}
```

这个代码序列等同于以下内容：

```java
if (zone.equals("East")) {
   shippingCost = weight * 0.23f;
} else 
   if (zone.equals("NorthCentral")) {
      shippingCost = weight * 0.35f;
   } else 
      if (zone.equals("SouthCentral")) {
         shippingCost = weight * 0.17f;
      } else {
         shippingCost = weight * 0.25f;
      }
```

第二个例子实现了与第一个相同的结果，但需要更多的缩进。在*switch 语句*部分，我们将演示如何使用 switch 语句实现相同的结果。

## if 语句-使用问题

在处理`if`语句时，有几个问题需要记住。在本节中，我们将讨论以下问题：

+   滥用等号运算符

+   使用布尔变量而不是逻辑表达式

+   在逻辑表达式中使用 true 或 false

+   不使用块语句的危险

+   悬空 else 问题

### 滥用等号运算符

Java 语言的一个很好的特性是，无法编写意外使用赋值运算符而意味着等号运算符的代码。这在 C 编程语言中经常发生，代码可以编译，但会导致逻辑错误，或者更糟糕的是在运行时异常终止。

例如，下面的代码片段比较`rate`，看它是否等于`100`：

```java
if(rate == 100) {
   …
}

```

然而，如果我们使用赋值运算符，如下面的代码片段所示，将会生成语法错误：

```java
  if(rate = 100) {
…
}
```

将生成类似以下的语法错误：

```java
incompatible types
 required: boolean
 found:    int

```

这种类型的错误在 Java 中被消除。使用等号运算符与浮点数的比较在*比较浮点数*部分中有所涉及。

### 提示

请注意，错误消息说它找到了一个`int`值。这是因为赋值运算符返回了一个**剩余值**。赋值运算符将修改其左边的操作数，并返回分配给该操作数的值。这个值是剩余值。它是操作的剩余部分。

理解剩余值的概念解释了错误消息。它还解释了为什么以下表达式有效：

```java
i = j = k = 10;
```

表达式的效果是将`10`赋给每个变量。赋值的结合性是从右到左。也就是说，当表达式中有多个赋值运算符时，它们会从右到左进行评估。值`10`被赋给`k`，并且赋值运算符返回了一个剩余值`10`。然后将剩余值赋给`j`，依此类推。

### 使用逆操作

在使用关系运算符时，通常有多种编写表达式的方法。例如，以下代码序列确定某人是否达到法定年龄：

```java
final int LEGAL_AGE = 21;
int age = 12;

if(age >= LEGAL_AGE) {
   // Process
} else {
   // Do not process
}
```

然而，这个代码序列也可以这样写：

```java
if(age < LEGAL_AGE) {
   // Do not process
} else {
   // Process
}
```

哪种方法更好？在这个例子中，可以说任何一种方法都可以。然而，最好使用最符合问题的形式。

请注意，下表中显示的操作是逆操作：

| 操作 | 逆操作 |
| --- | --- |
| `<` | `>=` |
| `>` | `<=` |

注意常量`LEGAL_AGE`的使用。尽可能使用标识符来表示诸如法定年龄之类的值是更好的。如果我们没有这样做，并且该值在多个地方使用，那么只需在一个地方更改该值即可。此外，这样做可以避免在其某个出现中意外使用错误的数字。此外，将数字设为常量可以消除在程序运行时意外修改不应修改的值的可能性。

### 使用布尔变量而不是逻辑表达式

正如我们在*布尔变量*部分看到的，我们可以声明一个布尔变量，然后将其用作逻辑表达式的一部分。我们可以使用布尔变量来保存逻辑表达式的结果，如下面的代码片段所示：

```java
boolean isLegalAge = age >= LEGAL_AGE;

if (isLegalAge) {
   // Process
} else {
   // Do not process
}
```

这有两个优点：

+   如果需要，它允许我们稍后重复使用结果

+   如果我们使用一个有意义的布尔变量名，代码会更易读

我们还可以使用否定运算符来改变`then`和`else`子句的顺序，如下所示：

```java
if (!isLegalAge) {
   // Do not process
} else {
   // Process
}
```

这个例子通常会比前一个更令人困惑。我们可以通过使用一个用词不当的布尔变量来使它变得更加令人困惑：

```java
if (!isNotLegalAge) {
   // Process
} else {
   // Do not process
}
```

虽然这是可读的和有效的，但一个一般规则是要避免双重否定，就像我们在英语中尝试做的那样。

### 在逻辑表达式中使用 true 或 false

`true`和`false`关键字可以用在逻辑表达式中。但它们并不是必需的，是多余的，并且使代码变得混乱，增加了很少的价值。请注意在以下逻辑`if`语句中使用`true`关键字：

```java
if (isLegalAge == true) {
   // Process
} else {
   // Do not process
}
```

显式使用子表达式`== true`是不必要的。当使用`false`关键字时也是如此。使用布尔变量本身更清晰、更简单，就像前面的例子中使用的那样。

### 不使用块语句的危险

由于块语句被视为一个语句，这允许在`if`语句的任一部分中包含多个语句，如下面的代码片段所示：

```java
if (isLegalAge) {
   System.out.println("Of legal age");
   System.out.println("Also of legal age");
} else {
   System.out.println("Not of legal age");
} 
```

当`then`或`else`子句只需要一个语句时，块语句实际上并不是必需的，但是是被鼓励的。类似但无效的`if`语句如下所示：

```java
if (isLegalAge) 
   System.out.println("Of legal age");
   System.out.println("Also of legal age");
else {
   System.out.println("Not of legal age");
}
```

块语句用于将代码组合在一起。打印语句的缩进不会将代码分组。虽然这可能暗示前两个`println`语句是`if`语句的一部分，但实际上，`if`语句将导致编译时错误。

这里呈现了相同的代码，但缩进不同。`if`语句只有一个`if`子句和一个`println`语句。第二个`println`语句紧随其后，无论逻辑表达式的值如何都会执行。然后是独立的 else 子句。编译器将此视为语法错误：

```java
if (isLegalAge) 
   System.out.println("Of legal age");
System.out.println("Also of legal age");
else {
   System.out.println("Not of legal age");
}
```

生成的语法错误将如下所示：

```java
'else' without 'if'

```

### 注意

一个经验法则是，对于`if`语句的`then`和`else`部分，始终使用块语句。

如果`else`子句中有额外的语句，可能会出现更隐匿的问题。考虑以下示例：

```java
if (isLegalAge) 
   System.out.println("Of legal age");
else
   System.out.println("Not of legal age");
   System.out.println("Also not of legal age");
```

第三个`println`语句不是 else 子句的一部分。它的缩进是误导性的。使用正确缩进的等效代码如下：

```java
if (isLegalAge) 
   System.out.println("Of legal age");
else 
   System.out.println("Not of legal age");
System.out.println("Also not of legal age");
```

很明显，第三个`println`语句将始终被执行。正确的写法如下：

```java
if (isLegalAge) {
   System.out.println("Of legal age");
} else {
   System.out.println("Not of legal age");
   System.out.println("Also not of legal age");
}
```

### 悬挂 else 问题

不使用块语句的另一个问题是悬挂 else 问题。考虑以下一系列测试，我们需要做出一些决定：

+   如果`limit`大于`100`且`stateCode`等于`45`，我们需要将`limit`增加`10`

+   如果`limit`不大于`100`，我们需要将`limit`减少`10`

以下是实现这个逻辑的方式：

```java
if (limit > 100) 
   if (stateCode == 45) 
      limit = limit+10;
else
   limit = limit-10;
```

然而，这个例子没有正确实现决策。这个例子至少有两个问题。首先，`else`关键字的缩进与语句的评估无关，且具有误导性。`else`关键字总是与最近的`if`关键字配对，这在这种情况下是第二个`if`关键字。编译器不关心我们如何缩进我们的代码。这意味着代码等同于以下内容：

```java
if (limit > 100) 
   if (stateCode == 45) 
      limit = limit+10;
   else
      limit = limit-10;
```

在这里，只有在限制超过`100`时才测试`stateCode`，然后`limit`要么增加要么减少`10`。

请记住，编译器会忽略任何语句中的空白（空格、制表符、换行等）。代码序列可以不带空白地编写，但这样会使其更难阅读：

```java
if (limit > 100) if (stateCode == 45) limit = limit+10; else limit = limit-10;
```

这个例子的第二个问题是没有使用块语句。块语句不仅提供了一种分组语句的方式，还提供了一种更清晰地传达应用程序逻辑的方式。问题可以通过以下代码解决：

```java
if (limit > 100) {
   if (stateCode == 45) {
      limit = limit+10;
   }
} else {
   limit = limit-10;
}
```

这样更清晰，实现了预期的效果。这样可以更容易地调试程序，代码更易读，更易维护。

# 条件运算符

条件运算符是`if`语句的一种简化、有限形式。它是简化的，因为决策仅限于单个表达式。它是有限的，因为`then`或`else`子句中不能包含多个语句。有时它被称为**三元运算符**，因为它有三个组成部分。

运算符的基本形式如下：

*LogicalExpression ? ThenExpression : ElseExpression*

如果*LogicalExpression*评估为 true，则返回*ThenExpression*的结果。否则返回*ElseExpression*的结果。

以下是一个简单的例子，用于测试一个数字是否小于 10。如果是，返回 1，否则返回 2。示例中的`then`和`else`表达式是琐碎的整数文字。

```java
result = (num < 10) ? 1 : 2;
```

这等同于以下的`if`语句：

```java
if (num < 10) {
   result = 1;
} else {
   result = 2;
}
```

考虑计算加班的过程。如果员工工作时间不超过 40 小时，工资按照工作时间乘以他的工资率计算。如果工作时间超过 40 小时，那么员工将为超过 40 小时的时间支付加班费。

```java
float hoursWorked;
float payRate;
float pay;

if (hoursWorked <= 40) {
   pay = hoursWorked * payRate;
} else {
   pay = 40 * payRate + (hoursWorked - 40) * payRate;
}
```

这个操作可以使用条件运算符来执行，如下所示：

```java
payRate = (hoursWorked <= 40) ? 
   hoursWorked * payRate : 
   40 * payRate + (hoursWorked - 40) * payRate;
```

虽然这种解决方案更加紧凑，但可读性不如前者。此外，`then`和`else`子句需要是返回某个值的表达式。这个值不一定是一个数字，但除非调用包含这些语句的方法，否则不能包含多个语句。

### 注意

除了在琐碎的情况下，不鼓励使用条件运算符，主要是因为它的可读性问题。通常更重要的是拥有可读性强、可维护的代码，而不是节省几行代码。

# switch 语句

`switch`语句的目的是提供一种方便和简单的方法，根据整数、枚举或`String`表达式进行多分支选择。`switch`语句的基本形式如下：

```java
switch ( expression ) {
  //case clauses
}
```

通常在语句块中有多个`case`子句。`case`子句的基本形式使用`case`关键字后跟一个冒号，零个或多个语句，通常是一个`break`语句。`break`语句由一个关键字`break`组成，如下所示：

```java
case <constant-expression>:
  //statements
break;
```

还可以使用一个可选的 default 子句。这将捕获任何未被`case`子句捕获的值。如下所示：

```java
default:
  //statements
break;  // Optional
```

`switch`语句的基本形式如下所示：

```java
switch (expression) {
  case value: statements
  case value: statements
  …
  default: statements
}
```

`switch`语句中的两个情况不能具有相同的值。`break`关键字用于有效地结束代码序列并退出`switch`语句。

当表达式被评估时，控制将传递给与相应常量表达式匹配的 case 表达式。如果没有任何 case 与表达式的值匹配，则控制将传递给`default`子句（如果存在）。如果没有`default`前缀，则`switch`的任何语句都不会被执行。

我们将说明`switch`语句用于整数、枚举和`String`表达式。在 Java 7 中，`switch`语句中使用字符串是新的。

## 基于整数的 switch 语句

`if`语句可以用于在多个整数值之间进行选择。考虑以下示例。一系列`if`语句可以用于根据整数`zone`值计算运费，如下所示：

```java
private static float computeShippingCost(
         int zone, float weight) {
   float shippingCost;

   if (zone == 5) {
      shippingCost = weight * 0.23f;
   } else if (zone == 6) {
      shippingCost = weight * 0.23f;
   } else if (zone == 15) {
      shippingCost = weight * 0.35f;
   } else if (zone == 18) {
      shippingCost = weight * 0.17f;
   } else {
      shippingCost = weight * 0.25f;
   }

   return shippingCost;
}
```

`switch`语句可以用于相同的目的，如下所示：

```java
switch (zone) {
   case 5:
      shippingCost = weight * 0.23f;
      break;
   case 6:
      shippingCost = weight * 0.23f;
      break;
   case 15:
      shippingCost = weight * 0.35f;
      break;
   case 18:
      shippingCost = weight * 0.17f;
      break;
   default:
      shippingCost = weight * 0.25f;
}
```

### 注意

不要忘记整数数据类型包括`byte`、`char`、`short`和`int`。这些数据类型中的任何一个都可以与整数`switch`语句一起使用。不允许使用数据类型`long`。

case 和 default 前缀的顺序并不重要。唯一的限制是常量表达式必须是唯一的。如果`break`语句不是最后一个 case 子句，那么它可能需要一个`break`语句，否则控制将传递给它后面的`case`子句：

```java
switch (zone) {
   case 15:
      shippingCost = weight * 0.35f;
      break;
   default:
      shippingCost = weight * 0.25f;
      break; // Only needed if default is not
             // the last case clause
   case 5:
      shippingCost = weight * 0.23f;
      break;
   case 18:
      shippingCost = weight * 0.17f;
      break;
   case 6:
      shippingCost = weight * 0.23f;
      break;
}
```

### 注意

出于可读性的目的，通常会保持自然顺序，这通常是顺序的。使用这个顺序可以更容易地找到`case`子句，并确保不会意外地遗漏情况。

case 和 default 前缀不会改变控制流。控制将从一个 case 流向下一个后续 case，除非使用了 break 语句。由于区域 5 和 6 使用相同的公式来计算运费，我们可以在不使用 break 语句的情况下使用连续的 case 语句：

```java
switch (zone) {
   case 5:
   case 6:
      shippingCost = weight * 0.23f;
      break;
   case 15:
      shippingCost = weight * 0.35f;
      break;
   case 18:
      shippingCost = weight * 0.17f;
      break;
   default:
      shippingCost = weight * 0.25f;
}
```

需要使用`break`语句来确保只有与`case`相关的语句被执行。在`default`子句的末尾不一定需要`break`，因为控制通常会流出`switch`语句。然而，出于完整性的目的，通常会包括它，如果`default`子句不是`switch`语句中的最后一个情况，则是必需的。

## 基于枚举的 switch 语句

枚举也可以与`switch`语句一起使用。这可以使代码更易读和易维护。以下内容是从第二章中复制的，*Java 数据类型及其用法*。变量`direction`用于控制`switch`语句的行为，如下所示：

```java
private static enum Directions {
    NORTH, SOUTH, EAST, WEST
};

Directions direction = Directions.NORTH;

switch (direction) {
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

当执行此操作时，我们得到以下输出：

```java
Going North

```

## 基于字符串的 switch 语句

为了说明在`switch`语句中使用字符串，我们将演示根据区域计算运费的实现，如*else-if variation*部分所示。该实现如下所示，供您参考：

```java
if (zone.equals("East")) {
   shippingCost = weight * 0.23f;
} else if (zone.equals("NorthCentral")) {
   shippingCost = weight * 0.35f;
} else if (zone.equals("SouthCentral")) {
   shippingCost = weight * 0.17f;
} else {
   shippingCost = weight * 0.25f;
}
```

在 Java 7 之前，只有整数变量可以与`switch`语句一起使用。通过允许使用字符串，程序可以包含更易读的代码。

以下代码片段说明了如何在`case`语句中使用`String`变量。该示例提供了前一个嵌套`if`语句的替代实现：

```java
switch (zone) {
   case "East":
      shippingCost = weight * 0.23f;
      break;
   case "NorthCentral":
      shippingCost = weight * 0.35f;
      break;
   case "SouthCentral":
      shippingCost = weight * 0.17f;
      break;
   default:
      shippingCost = weight * 0.25f;
}
```

### 使用 switch 语句的字符串问题

在使用带有 switch 语句的字符串时，还应考虑另外两个问题：

+   遇到空值时

+   字符串的大小写敏感性

当在 switch 语句中使用了一个已分配了空值的字符串变量时，将抛出`java.lang.NullPointerException`异常。当针对已分配了空值的引用变量执行方法时，这将发生。在 Java 7 中，对于`java.util.Objects`类中发现的空值，还有额外的支持。

关于字符串和 switch 语句的第二件事是，在`switch`语句中进行的比较是区分大小写的。在前面的示例中，如果使用了字符串值`east`，则`East`情况将不匹配，并且将执行`defa` `ult`情况。

# 控制结构问题

到目前为止，我们已经确定了几种在 Java 中可用的决策结构。例如，简单的决策可以使用`if`语句轻松处理。要么-要么类型的决策可以使用`else if`子句或`switch`语句来处理。

正确使用控制结构在开发良好的代码中至关重要。然而，做决定不仅仅是在不同的控制结构之间进行选择。我们还需要测试我们的假设并处理意外情况。

在本节中，我们将首先解决在使用决策结构时应牢记的一些一般问题。然后，我们将检查各种浮点问题，这可能对不熟悉浮点数限制的人造成麻烦。接下来，我们将简要介绍比较对象的主题，并总结三种基本编码活动的概述，这可能有助于理解编程的本质。

## 一般决策结构问题

在使用决策结构时有几个重要问题：

+   决策语句的结构

+   测试你的假设

+   为失败做计划

决策过程的整体结构可以是良好结构化的，也可以是一系列难以遵循的临时语句。对这种结构的良好组织方法可以提高决策过程的可读性和可维护性。

一个程序可能结构良好，但可能无法按预期工作。这往往是由于无效的假设。例如，如果假定年龄的值是非负的，那么使用的代码可能形式良好，从逻辑上讲可能是无可挑剔的。然而，如果假设使用的年龄值不正确，那么结果可能不如预期。例如，如果一个人的年龄输入为负数，那么逻辑可能会失败。始终测试你的假设，或者至少确保基础数据已经通过了某种质量控制检查。始终预料意外。帮助这一过程的技术包括：

+   始终保留`else`子句

+   测试你的假设

+   抛出异常（在第八章中介绍，*在应用程序中处理异常*）

+   始终使用块语句

当一切都失败时，请使用调试技术。

## 浮点数考虑

浮点数在内部使用 IEEE 754 浮点算术标准([`ieeexplore.ieee.org/xpl/mostRecentIssue.jsp?punumber=4610933`](http://ieeexplore.ieee.org/xpl/mostRecentIssue.jsp?punumber=4610933))来表示。这些操作通常在软件中执行，因为并非所有平台都提供对该标准的硬件支持。在软件中执行这些操作将比直接在硬件中执行慢。在软件中执行这些操作的优势在于它支持应用程序的可移植性。

Java 支持两种浮点类型，`float`和`double`，它们的精度如下表所示。此外，`Integer`和`Float`类是这两种数据类型的包装类。包装类用于封装值，比如整数或浮点数：

| 数据类型 | 大小（字节） | 精度 |
| --- | --- | --- |
| `float` | 4 | 二进制数字 |
| `double` | 8 | 二进制数字 |

处理浮点数可能比处理其他数据类型更复杂。需要考虑浮点数的几个方面。这些包括：

+   特殊的浮点值

+   比较浮点数

+   舍入误差

### 特殊的浮点值

总结在下表中有几个特殊的浮点值。它们存在是为了当发生错误条件时，有一个可以用来识别错误的表示。

这些值存在是为了当发生算术溢出、对负数取平方根和除以 0 等错误条件时，可以得到一个可以在浮点值内表示的结果，而不会抛出异常或以其他方式终止应用程序：

| 值 | 含义 | 可能由以下产生 |
| --- | --- | --- |
| 非数字 | **NaN**：表示生成未定义值的操作的结果 | 除以零取平方根 |
| 负无穷 | 一个非常小的值 | 一个负数除以零 |
| 正无穷 | 一个非常大的值 | 一个正数除以零 |
| 负零 | 负零 | 一个负数非常接近零，但不能正常表示 |

如果需要，NaN 可以通过`Float.NaN`和`Double.NaN`在代码中表示。对 NaN 值进行算术运算将得到 NaN 的结果。将 NaN 转换为整数将返回`0`，这可能导致应用程序错误。NaN 的使用在以下代码序列中进行了说明：

```java
float num1 = 0.0f;

System.out.println(num1 / 0.0f);
System.out.println(Math.sqrt(-4));
System.out.println(Double.NaN + Double.NaN);
System.out.println(Float.NaN + 2);
System.out.println((int) Double.NaN);
```

当执行时，我们得到以下输出：

```java
NaN
NaN
NaN
NaN
0

```

在 Java 中，无穷大使用以下字段表示。正如它们的名称所暗示的，我们可以表示负无穷或正无穷。负无穷意味着一个非常小的数，正无穷代表一个非常大的数：

+   `Float.NEGATIVE_INFINITY`

+   `Double.NEGATIVE_INFINITY`

+   `Float.POSITIVE_INFINITY`

+   `Double.POSITIVE_INFINITY`

一般来说，涉及无限值的算术运算将得到一个无限值。涉及 NaN 的运算将得到 NaN 的结果。除以零将得到正无穷。以下代码片段说明了其中一些操作：

```java
System.out.println(Float.NEGATIVE_INFINITY);
System.out.println(Double.NEGATIVE_INFINITY);
System.out.println(Float.POSITIVE_INFINITY);
System.out.println(Double.POSITIVE_INFINITY);
System.out.println(Float.POSITIVE_INFINITY+2);
System.out.println(1.0 / 0.0);
System.out.println((1.0 / 0.0) - (1.0 / 0.0));
System.out.println(23.0f / 0.0f);
System.out.println((int)(1.0 / 0.0)); 
System.out.println(Float.NEGATIVE_INFINITY == Double.NEGATIVE_INFINITY);
```

这个序列的输出如下：

```java
-Infinity
-Infinity
Infinity
Infinity
Infinity
Infinity
NaN
Infinity
2147483647
True

```

通过将负数除以正无穷或将正数除以负无穷，可以生成负零，如下面的代码片段所示。这两个语句的输出将是负零：

```java
System.out.println(-1.0f / Float.POSITIVE_INFINITY);
System.out.println(1.0f / Float.NEGATIVE_INFINITY);
```

`0`和`-0`是不同的值。然而，当它们相互比较时，它们将被确定为相等：

```java
System.out.println(0 == -0);
```

这将生成以下输出：

```java
True

```

### 比较浮点数

浮点数，在计算机中的表示，实际上并不是真实的数字。也就是说，在数字系统中有无限多个浮点数。然而，只有 32 位或 64 位用于表示一个浮点数。这意味着只能精确表示有限数量的浮点数。例如，分数 1/3 在十进制中无法精确表示。如果我们尝试，会得到类似 0.333333 的结果。同样，在二进制中，有一些浮点数无法精确表示，比如分数 1/10。

这意味着比较浮点数可能会很困难。考虑以下例子，我们将两个数字相除，并将结果与期望的商 6 进行比较：

```java
double num2 = 1.2f;
double num3 = 0.2f;
System.out.println((num2 / num3) == 6);
```

执行时的结果给出了一个意外的值，如下所示：

```java
false

```

这是因为这些数字使用`double`类型并不是精确表示。为了解决这个问题，我们可以检查操作的结果，并查看我们期望的结果与实际结果之间的差异。在以下序列中，定义了一个差异`epsilon`，它是可接受的最大差异：

```java
float epsilon = 0.000001f;
if (Math.abs((num2 / num3) - 6) < epsilon) {
   System.out.println("They are effectively equal");
} else {
   System.out.println("They are not equal");
}
```

当执行此代码时，我们得到以下输出：

```java
They are effectively equal

```

此外，当使用`compareTo`方法比较`Float`或`Double`对象时，记住这些对象按从低到高的顺序排列：

+   负无穷

+   负数

+   -0.0

+   0.0

+   正数

+   正无穷

+   NaN

例如，以下代码将返回`-1`，表示负数小于`-0.0`。输出将是`true`：

```java
System.out.println((new Float(-2.0)).compareTo(-0.0f));
```

### 舍入误差

在某些情况下，注意舍入误差是很重要的。考虑以下代码序列：

```java
for(int i = 0; i < 10; i++) {
   sum += 0.1f;
}
System.out.println(sum);
```

当执行此代码时，我们得到以下输出：

```java
1.0000001

```

这是由于舍入误差导致的，其根源来自于对分数 1/10 的不准确表示。

### 提示

对于精确值，使用浮点数并不是一个好主意。这适用于美元和美分。相反，使用`BigDecimal`，因为它提供更好的精度，并且设计用于支持这种类型的操作。

### strictfp 关键字

`strictfp`关键字可以应用于类、接口或方法。在 Java 2 之前，所有浮点计算都是按照 IEEE 754 规范执行的。在 Java 2 之后，中间计算不再受标准限制，允许使用一些处理器上可用的额外位来提高精度。这可能导致应用程序在舍入方面存在差异，从而导致可移植性较差。通过使用`strictfp`关键字，所有计算都将严格遵守 IEEE 标准。

## 比较对象

在比较对象时，我们需要考虑：

+   比较对象引用

+   使用`equals`方法比较对象

在比较引用时，我们确定两个引用变量是否指向相同的对象。如果我们想确定指向两个不同对象的两个引用变量是否相同，我们使用`equals`方法。

这两种比较如下图所示。三个引用变量`r1`、`r2`和`r3`用于引用两个对象。变量`r1`和`r2`引用对象 1，而`r3`引用对象 2：

![比较对象](img/7324_03_04.jpg)

在这个例子中，以下条件为真：

+   `r1 == r2`

+   `r1 != r3`

+   `r2 != r3`

+   `r1.equals(r2)`

然而，根据对象的`equals`方法的实现和对象本身，对象 1 可能与对象 2 等价，也可能不等价。字符串的比较在第二章的*字符串比较*部分中有更详细的介绍，*Java 数据类型及其使用*。覆盖`equals`方法在第六章中有讨论，*类、构造函数和方法*。 

## 三种基本编码活动

在编写代码时，很难确定如何最好地组织代码。为了保持透视，记住这三个一般的编码活动：

+   你想做什么

+   如何做

+   何时做

如果应用程序的要求是计算小时工的工资，那么：

+   “什么”是计算工资

+   “如何”决定如何编写代码来使用工作小时和工资率计算工资

+   “when”涉及放置代码的位置，即在确定工作小时和工资率之后

虽然这可能看起来很简单，但许多初学者程序员会在编程的“when”方面遇到问题。这对于今天的基于**图形用户界面**（**GUI**）的事件驱动程序尤其如此。

## goto 语句

`goto`语句在旧的编程语言中可用，并提供了程序内部转移控制的强大但不受约束的方式。它的使用经常导致程序组织混乱，并且是不被鼓励的。在 Java 中，`goto`关键字的使用受到限制。它根本不能被使用。它已经被彻底从 Java 编程中驱逐出去。

然而，与`goto`语句功能类似的语句仍然存在于许多语言中。例如，`break`语句会导致控制立即转移到 switch 语句的末尾，并且我们稍后会看到，退出循环。标签也可以与 break 语句一起使用，正如我们将在第五章的*使用标签*部分中看到的，*循环结构*。这种转移是即时和无条件的。它实际上是一个`goto`语句。然而，`break`语句，以类似的方式，return 语句和异常处理，被认为更加结构化和安全。控制不会转移到程序内的任意位置。它只会转移到`switch`语句末尾附近的特定位置。

# 总结

决策是编程的重要方面。大多数程序的实用性基于其做出某些决定的能力。决策过程基于控制结构的使用，如逻辑表达式、`if`语句和 switch 语句。

有不同类型的决策需要做出，并且在 Java 中使用不同的控制结构进行支持。本章讨论的主要内容包括`if`语句和`switch`语句。

在使用这些语句时必须小心，以避免可能出现的问题。这些问题包括误用比较运算符，不习惯使用块语句，以及避免悬空 else 问题。我们还研究了在使用浮点数时可能出现的一些问题。

在 Java 中做决策可以是简单的或复杂的。简单和复杂的二选一决策最好使用`if then else`语句来处理。对于一些更简单的决策，可以使用简单的`if`语句或条件语句。

多选决策可以使用`if`语句或`switch`语句来实现，具体取决于决策的性质。更复杂的决策可以通过嵌套的`if`语句和`switch`语句来处理。

现在我们已经了解了决策结构，我们准备研究如何使用数组和集合，这是下一章的主题。

# 涵盖的认证目标

关于认证目标，我们将研究：

+   使用运算符和决策结构

+   使用 Java 关系和逻辑运算符

+   使用括号来覆盖运算符优先级

+   创建 if 和 if/else 结构

+   使用`switch`语句

# 测试你的知识

1.  以下操作的结果是什么？

```java
System.out.println(4 % 3);
```

a. 0

b. 1

c. 2

d. 3

1.  以下哪个表达式将计算为 7？

a. `2 + 4 * 3- 7`

b. `(2 + 4) * (3 - 7)`

c. `2 + (4 * 3) - 7`

d. `((2 + 4) * 3) - 7)`

1.  以下语句的输出是什么？

```java
System.out.println( 16  >>>  3);
```

a. 1

b. 2

c. 4

d. 8

1.  给定以下声明，哪些 if 语句将在没有错误的情况下编译？

```java
int i = 3;
int j = 3;
int k = 3;
```

a. `if(i > j) {}`

b. `if(i > j > k) {}`

c. `if(i > j && i > k) {}`

d. `if(i > j && > k) {}`

1.  当执行以下代码时，将会打印出什么？

```java
switch (5) {
case 0:
   System.out.println("zero");
   break;
case 1:
   System.out.println("one");
default:
   System.out.println("default");
case 2:
   System.out.println("two");
}
```

a. 一个

b. 默认和两个

c. 一个，两个和默认

d. 什么也不会输出，会生成一个编译时错误
