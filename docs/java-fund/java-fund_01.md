# *第一章*

# 介绍 Java

## 学习目标

在本课结束时，你将能够：

+   描述 Java 生态系统的工作

+   编写简单的 Java 程序

+   从用户那里读取输入

+   利用 java.util 包中的类

## 介绍

在这第一课中，我们开始学习 Java。如果你是从其他编程语言的背景下来学习 Java，你可能知道 Java 是一种用于编程计算机的语言。但 Java 不仅仅是如此。它不仅仅是一种无处不在的非常流行和成功的语言，它还是一系列技术。除了语言之外，它还包括一个非常丰富的生态系统，并且有一个充满活力的社区，致力于使生态系统尽可能动态。

## Java 生态系统

Java 生态系统的三个最基本部分是**Java 虚拟机（JVM）**，**Java 运行时环境（JRE）**和**Java 开发工具包（JDK）**，它们是 Java 实现提供的*标准*部分。

![图 1.1：Java 生态系统的表示](img/C09581_01_01.jpg)

###### 图 1.1：Java 生态系统的表示

每个 Java 程序都在**JVM**的控制下运行。每次运行 Java 程序时，都会创建一个 JVM 实例。它为正在运行的 Java 程序提供安全性和隔离。它防止代码运行与系统中的其他程序发生冲突。它的工作原理类似于一个非严格的沙箱，使其可以安全地提供资源，即使在敌对环境（如互联网）中，但允许与其运行的计算机进行互操作。简单来说，JVM 就像一个*计算机内的计算机*，专门用于运行 Java 程序。

#### 注意

服务器通常同时执行许多 JVM。

在 Java 技术的*标准*层次结构中是`java`命令）。它包括所有基本的 Java 类（运行时）以及与主机系统交互的库（如字体管理，与图形系统通信，播放声音的能力以及在浏览器中执行 Java 小程序的插件）和实用程序（如 Nashorn JavaScript 解释器和 keytool 加密操作工具）。如前所述，JRE 包括 JVM。

在 Java 技术的顶层是`javac`。JDK 还包括许多辅助工具，如 Java 反汇编器（`javap`），用于创建 Java 应用程序包的实用程序（`jar`），从源代码生成文档的系统（`javadoc`）等等。JDK 是 JRE 的超集，这意味着如果你有 JDK，那么你也有 JRE（和 JVM）。

但这三个部分并不是 Java 的全部。Java 的生态系统包括社区的大量参与，这是该平台受欢迎的原因之一。

#### 注意

对 GitHub 上顶级 Java 项目使用的最流行的 Java 库进行的研究（根据 2016 年和 2017 年的重复研究）显示，JUnit，Mockito，Google 的 Guava，日志库（log4j，sl4j）以及所有 Apache Commons（Commons IO，Commons Lang，Commons Math 等）都标志着它们的存在，还有连接到数据库的库，用于数据分析和机器学习的库，分布式计算等几乎你能想象到的任何其他用途。换句话说，几乎任何你想编写程序的用途都有现有的工具库来帮助你完成任务。

除了扩展 Java 标准发行版功能的众多库之外，还有大量工具可以自动化构建（例如 Apache Ant，Apache Maven 和 Gradle），自动化测试，分发和持续集成/交付程序（例如 Jenkins 和 Apache Continuum），以及更多其他工具。

## 我们的第一个 Java 应用程序

正如我们之前简要提到的，Java 中的程序是用源代码（即普通文本，人类可读文件）编写的，这些源代码由编译器（在 Java 的情况下是`javac`）处理，以生成包含 Java 字节码的类文件。包含 Java 字节码的类文件，然后被提供给一个名为 java 的程序，其中包含执行我们编写的程序的 Java 解释器/JVM：

![图 1.2：Java 编译过程](img/C09581_01_02.jpg)

###### 图 1.2：Java 编译过程

### 简单 Java 程序的语法

像所有编程语言一样，Java 中的源代码必须遵循特定的语法。只有这样，程序才能编译并提供准确的结果。由于 Java 是一种面向对象的编程语言，Java 中的所有内容都包含在类中。一个简单的 Java 程序看起来类似于这样：

```java
public class Test { //line 1
    public static void main(String[] args) { //line 2
        System.out.println("Test"); //line 3
    } //line 4
} //line 5
```

每个 java 程序文件的名称应与包含`main()`的类的名称相同。这是 Java 程序的入口点。

因此，只有当这些指令存储在名为`Test.java`的文件中时，前面的程序才会编译并运行而不会出现任何错误。

Java 的另一个关键特性是它区分大小写。这意味着`System.out.Println`会抛出错误，因为它的大小写没有正确。正确的指令应该是`System.out.println`。

`main()`应该始终声明如示例所示。这是因为，如果`main()`不是一个`public`方法，编译器将无法访问它，java 程序将无法运行。`main()`是静态的原因是因为我们不使用任何对象来调用它，就像你对 Java 中的所有其他常规方法一样。

#### 注意

我们将在本书的后面讨论这些`public`和`static`关键字。

注释用于提供一些额外的信息。Java 编译器会忽略这些注释。

单行注释用`//`表示，多行注释用`/* */`表示。

### 练习 1：一个简单的 Hello World 程序

1.  右键单击`src`文件夹，选择**新建** | **类**。

1.  输入`HelloWorld`作为类名，然后点击**确定**。

1.  在类中输入以下代码：

```java
public class HelloWorld{    
public static void main(String[] args) {  // line 2
        System.out.println("Hello, world!");  // line 3
    }
}
```

1.  通过点击**运行** | **运行“Main”**来运行程序。

程序的输出应该如下所示：

```java
Hello World!
```

### 练习 2：执行简单数学运算的简单程序

1.  右键单击`src`文件夹，选择**新建** | **类**。

1.  输入`ArithmeticOperations`作为类名，然后点击**确定**。

1.  用以下代码替换此文件夹中的代码：

```java
public class ArithmeticOperations {
    public static void main(String[] args) {
            System.out.println(4 + 5);
            System.out.println(4 * 5);
            System.out.println(4 / 5);
            System.out.println(9 / 2);
    }
}
```

1.  运行主程序。

输出应该如下所示：

```java
9
20
0
4
```

在 Java 中，当您将一个整数（例如 4）除以另一个整数（例如 5）时，结果总是一个整数（除非您另有指示）。在前面的情况下，不要惊讶地看到 4/5 的结果是 0，因为这是 4 除以 5 的商（您可以使用%而不是除法线来获得除法的余数）。

要获得 0.8 的结果，您必须指示除法是浮点除法，而不是整数除法。您可以使用以下行来实现：

```java
System.out.println(4.0 / 5);
```

是的，这意味着，像大多数编程语言一样，Java 中有多种类型的数字。

### 练习 3：显示非 ASCII 字符

1.  右键单击`src`文件夹，选择**新建** | **类**。

1.  输入`ArithmeticOperations`作为类名，然后点击**确定**。

1.  用以下代码替换此文件夹中的代码：

```java
public class HelloNonASCIIWorld {
    public static void main(String[] args) {
            System.out.println("Non-ASCII characters: ☺");
            System.out.println("∀x ∈ ℝ: ⌈x⌉ = −⌊−x⌋");
            System.out.println("π ≅ " + 3.1415926535); // + is used to concatenate 
    }
}
```

1.  运行主程序。

程序的输出应该如下所示：

```java
Non-ASCII characters: ☺
∀x ∈ ℝ: ⌈x⌉ = −⌊−x⌋
π ≅ 3.1415926535
```

### 活动 1：打印简单算术运算的结果

要编写一个打印任意两个值的和和乘积的 java 程序，请执行以下步骤：

1.  创建一个新类。

1.  在`main()`中，打印一句描述您将执行的值的操作以及结果。

1.  运行主程序。您的输出应该类似于以下内容：

```java
The sum of 3 + 4 is 7
The product of 3 + 4 is 12
```

#### 注意

此活动的解决方案可以在 304 页找到。

### 从用户那里获取输入

我们之前学习过一个创建输出的程序。现在，我们要学习一个补充性的程序：一个从用户那里获取输入，以便程序可以根据用户给程序的内容来工作：

```java
import java.io.IOException; // line 1
public class ReadInput { // line 2
    public static void main(String[] args) throws IOException { // line 3
        System.out.println("Enter your first byte");
        int inByte = System.in.read(); // line 4
        System.out.println("The first byte that you typed: " + (char) inByte); // line 5
        System.out.printf("%s: %c.%n", "The first byte that you typed", inByte); // line 6
    } // line 7
} // line 8
```

现在，我们必须剖析我们的新程序的结构，即具有公共类`ReadInput`的程序。你可能注意到它有更多的行，而且显然更复杂，但不要担心：在合适的时候，每一个细节都会被揭示出来（以其全部、光辉的深度）。但是，现在，一个更简单的解释就足够了，因为我们不想失去对主要内容的关注，即从用户那里获取输入。

首先，在第 1 行，我们使用了`import`关键字，这是我们之前没有见过的。所有的 Java 代码都是以分层方式组织的，有许多包（我们稍后会更详细地讨论包，包括如何创建自己的包）。

这里，层次结构意味着“像树一样组织”，类似于家谱。在程序的第 1 行，`import`这个词简单地意味着我们将使用`java.io.Exception`包中组织的方法或类。

在第 2 行，我们像以前一样创建了一个名为`ReadInput`的新公共类，没有任何意外。正如预期的那样，这个程序的源代码必须在一个名为`ReadInput.java`的源文件中。

在第 3 行，我们开始定义我们的`main`方法，但是这次在括号后面加了几个词。新词是`throws IOException`。为什么需要这个呢？

简单的解释是：“否则，程序将无法编译。”更长的解释是：“因为当我们从用户那里读取输入时，可能会出现错误，Java 语言强制我们告诉编译器关于程序在执行过程中可能遇到的一些错误。”

另外，第 3 行是需要第 1 行的`import`的原因：`IOException`是一个特殊的类，位于`java.io.Exception`层次结构之下。

第 5 行是真正行动开始的地方：我们定义了一个名为`inByte`（缩写为“将要输入的字节”）的变量，它将包含`System.in.read`方法的结果。

`System.in.read`方法在执行时，将从标准输入（通常是键盘，正如我们已经讨论过的）中取出第一个字节（仅一个），并将其作为答案返回给执行它的人（在这种情况下，就是我们，在第 5 行）。我们将这个结果存储在`inByte`变量中，并继续执行程序。

在第 6 行，我们打印（到标准输出）一条消息，说明我们读取了什么字节，使用了调用`System.out.println`方法的标准方式。

注意，为了打印字节（而不是代表计算机字符的内部数字），我们必须使用以下形式的结构：

+   一个开括号

+   单词`char`

+   一个闭括号

我们在名为`inByte`的变量之前使用了这个。这个结构被称为类型转换，将在接下来的课程中更详细地解释。

在第 7 行，我们使用了另一种方式将相同的消息打印到标准输出。这是为了向你展示有多少任务可以以不止一种方式完成，以及“没有单一正确”的方式。在这里，我们使用了`System.out.println`函数。

其余的行只是关闭了`main`方法定义和`ReadInput`类的大括号。

`System.out.printf`的一些主要格式字符串列在下表中：

![表 1.1：格式字符串及其含义](img/C09581_Table_01_01.jpg)

###### 表 1.1：格式字符串及其含义

还有许多其他格式化字符串和许多变量，你可以在 Oracle 的网站上找到完整的规范。

我们将看到一些其他常见（修改过的）格式化字符串，例如%.2f（指示函数打印小数点后恰好两位小数的浮点数，例如 2.57 或-123.45）和%03d（指示函数打印至少三位数的整数，可能左侧填充 0，例如 001 或 123 或 27204）。

### 练习 4：从用户那里读取值并执行操作

从用户那里读取两个数字并打印它们的乘积，执行以下步骤：

1.  右键单击`src`文件夹，然后选择**新建** | **类**。

1.  输入`ProductOfNos`作为类名，然后单击**确定**。

1.  导入`java.io.IOException`包：

```java
import java.io.IOException;
```

1.  在`main()`中输入以下代码以读取整数：

```java
public class ProductOfNos{
public static void main(String[] args){
System.out.println("Enter the first number");
int var1 = Integer.parseInt(System.console().readLine());
System.out.println("Enter the Second number");
int var2 = Integer.parseInt(System.console().readLine());
```

1.  输入以下代码以显示两个变量的乘积：

```java
System.out.printf("The product of the two numbers is %d", (var1 * var2));
}
}
```

1.  运行程序。您应该看到类似于以下内容的输出：

```java
Enter the first number
10
Enter the Second number
20
The product of the two numbers is 200
```

干得好，这是你的第一个 Java 程序。

## 包

包是 Java 中的命名空间，可用于在具有相同名称的多个类时避免名称冲突。

例如，我们可能有由 Sam 开发的名为`Student`的多个类，另一个类由 David 开发的同名类。如果我们需要在代码中使用它们，我们需要区分这两个类。我们使用包将这两个类放入两个不同的命名空间。

例如，我们可能有两个类在两个包中：

+   `sam.Student`

+   `david.Student`

这两个包在文件资源管理器中如下所示：

![图 1.3：文件资源管理器中 sam.Student 和 david.Student 包的屏幕截图](img/C09581_01_03.jpg)

###### 图 1.3：文件资源管理器中 sam.Student 和 david.Student 包的屏幕截图

所有对 Java 语言基本的类都属于`java.lang`包。Java 中包含实用类的所有类，例如集合类、本地化类和时间实用程序类，都属于`java.util`包。

作为程序员，您可以创建和使用自己的包。

### 使用包时需要遵循的规则

在使用包时需要考虑一些规则：

+   包应该用小写字母编写

+   为了避免名称冲突，包名应该是公司的反向域。例如，如果公司域是`example.com`，那么包名应该是`com.example`。因此，如果我们在该包中有一个`Student`类，可以使用`com.example.Student`访问该类。

+   包名应该对应文件夹名。对于前面的例子，文件夹结构将如下所示：![图 1.4：文件资源管理器中的文件夹结构的屏幕截图](img/C09581_01_04.jpg)

###### 图 1.4：文件资源管理器中的文件夹结构的屏幕截图

要在代码中使用包中的类，您需要在 Java 文件的顶部导入该类。例如，要使用 Student 类，您可以按如下方式导入它：

```java
import com.example.Student;
public class MyClass {
}
```

`Scanner`是`java.util`包中的一个有用的类。这是一种输入类型（例如 int 或字符串）的简单方法。正如我们在早期的练习中看到的，包使用`nextInt()`以以下语法输入整数：

```java
sc = new Scanner(System.in);
int x =  sc.nextIn()
```

### 活动 2：从用户那里读取值并使用 Scanner 类执行操作

从用户那里读取两个数字并打印它们的和，执行以下步骤：

1.  创建一个新类，并将`ReadScanner`作为类名输入

1.  导入`java.util.Scanner`包

1.  在`main()`中使用`System.out.print`要求用户输入两个变量`a`和`b`的数字。

1.  使用`System.out.println`输出两个数字的和。

1.  运行主程序。

输出应该类似于这样：

```java
Enter a number: 12
Enter 2nd number: 23
The sum is 35\.  
```

#### 注意

此活动的解决方案可在 304 页找到。

### 活动 3：计算金融工具的百分比增长或减少

用户期望看到股票和外汇等金融工具的日增长或减少百分比。我们将要求用户输入股票代码，第一天的股票价值，第二天相同股票的价值，计算百分比变化并以格式良好的方式打印出来。为了实现这一点，执行以下步骤：

1.  创建一个新类，并输入`StockChangeCalculator`作为类名

1.  导入`java.util.Scanner`包：

1.  在`main()`中使用`System.out.print`询问用户股票的`symbol`，然后是股票的`day1`和`day2`值。

1.  计算`percentChange`值。

1.  使用`System.out.println`输出符号和带有两位小数的百分比变化。

1.  运行主程序。

输出应类似于：

```java
Enter the stock symbol: AAPL
Enter AAPL's day 1 value: 100
Enter AAPL's day 2 value: 91.5
AAPL has changed -8.50% in one day.
```

#### 注意

此活动的解决方案可在 305 页找到。

## 摘要

本课程涵盖了 Java 的基础知识。我们看到了 Java 程序的一些基本特性，以及如何在控制台上显示或打印消息。我们还看到了如何使用输入控制台读取值。我们还研究了可以用来分组类的包，并看到了`java.util`包中`Scanner`的一个示例。

在下一课中，我们将更多地了解值是如何存储的，以及我们可以在 Java 程序中使用的不同值。
