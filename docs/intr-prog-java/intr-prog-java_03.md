# Java 语言基础

现在您对 Java 及其相关术语和工具有了一个大致的了解，我们将开始讨论 Java 作为一种编程语言。

本章将介绍 Java 作为**面向对象编程**（**OOP**）语言的基本概念。您将了解类、接口和对象及其关系。您还将学习 OOP 的概念和特性。

在本章中，我们将涵盖以下主题：

+   Java 编程的基本术语

+   类和对象（实例）

+   类（静态）和对象（实例）成员

+   接口、实现和继承

+   OOP 的概念和特性

+   练习-接口与抽象类

我们称它们为基础，因为它们是 Java 作为一种语言的基本原则，而在您可以开始专业编程之前还有更多要学习。对于那些第一次学习 Java 的人来说，学习 Java 的基础是一个陡峭的斜坡，但之后的道路会变得更容易。

# Java 编程的基本术语

Java 编程基础的概念有很多解释。一些教程假设基础对于任何面向对象的语言都是相同的。其他人讨论语法和基本语言元素和语法规则。还有一些人将基础简化为允许计算的值类型、运算符、语句和表达式。

我们对 Java 基础的看法包括了前面各种方法的一些元素。我们选择的唯一标准是实用性和逐渐增加的复杂性。我们将从本节的简单定义开始，然后在后续章节中深入探讨。

# 字节码

在最广泛的意义上，Java 程序（或任何计算机程序）意味着一系列顺序指令，告诉计算机该做什么。在计算机上执行之前，程序必须从人类可读的高级编程语言编译成机器可读的二进制代码。

在 Java 的情况下，人类可读的文本，称为源代码，存储在一个`.java`文件中，并可以通过 Java 编译器`javac`编译成字节码。Java 字节码是 JVM 的指令集。字节码存储在一个`.class`文件中，并可以由 JVM 或更具体地说是由 JVM 使用的**即时**（**JIT**）编译器解释和编译成二进制代码。然后由微处理器执行二进制代码。

字节码的一个重要特点是它可以从一台机器复制到另一台机器的 JVM 上执行。这就是 Java 可移植性的含义。

# 缺陷（bug）及其严重程度和优先级

*bug*这个词，意思是*小故障和困难*，早在 19 世纪就存在了。这个词的起源是未知的，但看起来好像动词*to bug*的意思是*打扰*，来自于一种讨厌的感觉，来自于一个嗡嗡作响并威胁要咬你或其他东西的昆虫-虫子。这个词在计算机第一次建造时就被用于编程缺陷。

缺陷的严重程度各不相同-它们对程序执行或结果的影响程度。一些缺陷是相当微不足道的，比如数据以人类可读的格式呈现。如果同样的数据必须由其他无法处理这种格式的系统消耗，那就另当别论了。那么这样的缺陷可能被归类为关键，因为它将不允许系统完成数据处理。

缺陷的严重程度取决于它对程序的影响，而不是修复它有多困难。

一些缺陷可能会导致程序在达到期望结果之前退出。例如，一个缺陷可能导致内存或其他资源的耗尽，并导致 JVM 关闭。

缺陷优先级，缺陷在待办事项列表中的高度，通常与严重性相对应。但是，由于客户的感知，一些低严重性的缺陷可能会被优先考虑。例如，网站上的语法错误，或者可能被视为冒犯的拼写错误。

缺陷的优先级通常对应于其严重性，但有时，优先级可能会根据客户的感知而提高。

# Java 程序依赖

我们还提到，程序可能需要使用已编译为字节码的其他程序和过程。为了让 JVM 找到它们，您必须在`java`命令中使用`-classpath`选项列出相应的`.class`文件。几个程序和过程组成了一个 Java 应用程序。

应用程序用于其任务的其他程序和过程称为应用程序依赖项。

请注意，JVM 在其他类代码请求之前不会读取`.class`文件。因此，如果在应用程序执行期间不发生需要它们的条件，那么类路径上列出的一些`.class`文件可能永远不会被使用。

# 语句

语句是一种语言构造，可以编译成一组指令给计算机。与日常生活中的 Java 语句最接近的类比是英语语句，这是一种表达完整思想的基本语言单位。Java 中的每个语句都必须以`;`（分号）结尾。

以下是一个声明语句的示例：

```java

int i;

```

前面的语句声明了一个`int`类型的变量`i`，代表*整数*（见第五章，*Java 语言元素和类型*）。

以下是一个表达式语句：

```java

i + 2;

```

前面的语句将 2 添加到现有变量`i`的值中。当声明时，`int`变量默认被赋值为 0，因此此表达式的结果为`2`，但未存储。这就是为什么它经常与声明和赋值语句结合使用的原因：

```java

int j = i + 2;

```

这告诉处理器创建一个`int`类型的变量`j`，并为其分配一个值，该值等于变量`i`当前分配的值加 2。在第九章，*运算符、表达式和语句*中，我们将更详细地讨论语句和表达式。

# 方法

Java 方法是一组语句，总是一起执行，目的是对某个输入产生某个结果。方法有一个名称，要么一组输入参数，要么根本没有参数，一个在`{}`括号内的主体，以及一个返回类型或`void`关键字，表示该消息不返回任何值。以下是一个方法的示例：

```java

int multiplyByTwo(int i){

int j = i * 2;

return j;

}

```

在前面的代码片段中，方法名为`multiplyByTwo`。它有一个`int`类型的输入参数。方法名和参数类型列表一起称为**方法签名**。输入参数的数量称为**arity**。如果两个方法具有相同的名称、相同的 arity 和相同的输入参数列表中类型的顺序，则它们具有相同的签名。

这是从 Java 规范第*8.4.2 节方法签名*中摘取的方法签名定义的另一种措辞。另一方面，在同一规范中，人们可能会遇到诸如：*具有相同名称和签名的多个方法*，*类*`Tuna`*中的方法*`getNumberOfScales`*具有名称、签名和返回类型*等短语。因此，要小心；即使是规范的作者有时也不将方法名包括在方法签名的概念中，如果其他程序员也这样做，不要感到困惑。

同一个前面的方法可以用许多风格重写，并且得到相同的结果：

```java

int multiplyByTwo(int i){

return i * 2;

}

```

另一种风格如下：

```java

int multiplyByTwo(int i){ return i * 2; }

```

一些程序员更喜欢最紧凑的风格，以便能够在屏幕上看到尽可能多的代码。但这可能会降低另一个程序员理解代码的能力，这可能会导致编程缺陷。

另一个例子是一个没有输入参数的方法：

```java

int giveMeFour(){ return 4; }

```

这是相当无用的。实际上，没有参数的方法会从数据库中读取数据，例如，或者从其他来源读取数据。我们展示这个例子只是为了演示语法。

这是一个什么都不做的代码示例：

```java

void multiplyByTwo(){ }

```

前面的方法什么也不做，也不返回任何东西。语法要求使用关键字`void`来指示没有返回值。实际上，没有返回值的方法通常用于将数据记录到数据库，或者发送数据到打印机、电子邮件服务器、另一个应用程序（例如使用 Web 服务），等等。

为了完整起见，这是一个具有许多参数的方法的示例：

```java

String doSomething(int i, String s, double a){

double result = Math.round(Math.sqrt(a)) * i;

返回 s + Double.toString(result);

}

```

上述方法从第三个参数中提取平方根，将其乘以第一个参数，将结果转换为字符串，并将结果附加（连接）到第二个参数。将在第五章中介绍使用的`Math`类的类型和方法，*Java 语言元素和类型*。这些计算并没有太多意义，仅供说明目的。

# 类

Java 中的所有方法都声明在称为**类**的结构内。一个类有一个名称和一个用大括号`{}`括起来的主体，在其中声明方法：

```java

我的类 {

int multiplyByTwo(int i){ return i * 2; }

int giveMeFour(){ return 4;}

}

```

类也有字段，通常称为属性；我们将在下一节讨论它们。

# 主类和主方法

一个类作为 Java 应用程序的入口。在启动应用程序时，必须在`java`命令中指定它：

```java

java -cp <所有.class 文件的位置> MyGreatApplication

```

在上述命令中，`MyGreatApplication`是作为应用程序起点的类的名称。当 JVM 找到文件`MyGreatApplication.class`时，它会将其读入内存，并在其中查找名为`main()`的方法。这个方法有一个固定的签名：

```java

public static void main(String[] args) {

// 在这里放语句

}

```

让我们把前面的代码片段分成几部分：

+   `public`表示这个方法对任何外部程序都是可访问的（参见第七章，*包和可访问性（可见性）*）

+   `static`表示该方法在所有内存中只存在一个副本（参见下一节）

+   `void`表示它不返回任何东西

+   `main`是方法名

+   `String[] args`表示它接受一个 String 值的数组作为输入参数（参见第五章，*Java 语言元素和类型*）

+   `//`表示这是一个注释，JVM 会忽略它，这里只是为了人类（参见第五章，*Java 语言元素和类型*）

前面的`main()`方法什么也不做。如果运行，它将成功执行但不会产生结果。

您还可以看到输入参数写成如下形式：

```java

public static void main(String... args) {

//执行一些操作的主体

}

```

它看起来像是不同的签名，但实际上是相同的。自 JDK 5 以来，Java 允许将方法签名的*最后一个参数*声明为相同类型的变量可变性的一系列参数。这被称为**varargs**。在方法内部，可以将最后一个输入参数视为数组`String[]`，无论它是显式声明为数组还是作为可变参数。如果你一生中从未使用过 varargs，那么你会没问题。我们告诉你这些只是为了让你在阅读其他人的代码时避免混淆。

`main（）`方法的最后一个重要特性是其输入参数的来源。没有其他代码调用它。它是由 JVM 本身调用的。那么参数是从哪里来的呢？人们可能会猜想命令行是参数值的来源。在`java`命令中，到目前为止，我们假设没有参数传递给主类。但是如果主方法期望一些参数，我们可以构造命令行如下：

```java

java -cp <所有.class 文件的位置> MyGreatApplication 1 2

```

这意味着在`main（）`方法中，输入数组`args [0]`的第一个元素的值将是`1`，而输入数组`args [1]`的第二个元素的值将是`2`。是的，你注意到了，数组中元素的计数从`0`开始。我们将在第五章中进一步讨论这个问题，*Java 语言元素和类型*。无论是显式地使用数组`String[] args`描述`main（）`方法签名，还是使用可变参数`String... args`，结果都是一样的。

然后`main（）`方法中的代码调用同一 main`.class`文件中的方法或使用`-classpath`选项列出的其他`.class`文件中的方法。在接下来的部分中，我们将看到如何进行这样的调用。

# 类和对象（实例）

类用作创建对象的模板。创建对象时，类中声明的所有字段和方法都被复制到对象中。对象中字段值的组合称为**对象状态**。方法提供对象行为。对象也称为类的实例。

每个对象都是使用运算符`new`和看起来像一种特殊类型的方法的构造函数创建的。构造函数的主要职责是设置初始对象状态。

现在让我们更仔细地看一看 Java 类和对象。

# Java 类

Java 类存储在`.java`文件中。每个`.java`文件可以包含多个类。它们由 Java 编译器`javac`编译并存储在`.class`文件中。每个`.class`文件只包含一个已编译的类。

每个`.java`文件只包含一个`public`类。类名前的关键字`public`使其可以从其他文件中的类访问。文件名必须与公共类名匹配。文件还可以包含其他类，它们被编译成自己的`.class`文件，但只能被给出其名称的公共类访问`.java`文件。

这就是文件`MyClass.java`的内容可能看起来像的样子：

```java

public class MyClass {

private int field1;

private String field2;

public String method1（int i）{

//语句，包括返回语句

}

私有 void 方法 2（字符串 s）{

//没有返回语句的语句

}

}

```

它有两个字段。关键字`private`使它们只能从类内部，从它的方法中访问。前面的类有两个方法 - 一个是公共的，一个是私有的。公共方法可以被任何其他类访问，而私有方法只能从同一类的其他方法中访问。

这个类似乎没有构造函数。那么，基于这个类的对象的状态将如何初始化？答案是，事实上，每个没有显式定义构造函数但获得一个默认构造函数的类。这里有两个显式添加的构造函数的例子，一个没有参数，另一个有参数：

```java

public class SomeClass {

private int field1;

public MyClass(){

this.field1 = 42;

}

//... 类的其他内容 - 方法

//    定义对象行为

}

public class MyClass {

private int field1;

private String field2;

public MyClass(int val1, String val2){

this.field1 = val1;

this.field2 = val2;

}

//... 方法在这里

}

```

在上面的代码片段中，关键字`this`表示当前对象。它的使用是可选的。我们可以写`field1 = val1;`并获得相同的结果。但是最好使用关键字`this`来避免混淆，特别是当（程序员经常这样做）参数的名称与字段的名称相同时，比如在下面的构造函数中：

```java

public MyClass(int field1, String field1){

field1 = field1;

field2 = field2;

}

```

添加关键字`this`使代码更友好。有时候，这是必要的。我们将在第六章中讨论这样的情况，*接口、类和对象构造*。

一个构造函数也可以调用这个类或任何其他可访问类的方法：

```java

public class MyClass {

private int field1;

private String field2;

public MyClass(int val1, String val2){

this.field1 = val1;

this.field2 = val2;

method1(33);

method2(val2);

}

public String method1(int i){

//语句，包括返回语句

}

private void method2(String s){

//没有返回语句的语句

}

}

```

如果一个类没有显式定义构造函数，它会从默认的基类`java.lang.Object`中获得一个默认构造函数。我们将在即将到来的*继承*部分解释这意味着什么。

一个类可以有多个不同签名的构造函数，用于根据应用程序逻辑创建具有不同状态的对象。一旦在类中添加了带参数的显式构造函数，除非也显式添加默认构造函数，否则默认构造函数将不可访问。澄清一下，这个类只有一个默认构造函数：

```java

public class MyClass {

private int field1;

private String field2;

//... 其他方法在这里

}

```

这个类也只有一个构造函数，但没有默认构造函数：

```java

public class MyClass {

private int field1;

private String field2;

public MyClass(int val1, String val2){

this.field1 = val1;

this.field2 = val2;

}

//... 其他方法在这里

}

```

这个类有两个构造函数，一个有参数，一个没有参数：

```java

public class MyClass {

private int field1;

private String field2;

public MyClass(){ }

public MyClass(int val1, String val2){

this.field1 = val1;

this.field2 = val2;

}

//... 其他方法在这里

}

```

没有参数的前面构造函数什么也不做。它只是为了方便客户端代码创建这个类的对象，但不关心对象的特定初始状态。在这种情况下，JVM 创建默认的初始对象状态。我们将在第六章中解释默认状态，*接口、类和对象构造*。

同一个类的每个对象，由任何构造函数创建，都有相同的方法（相同的行为），即使它的状态（分配给字段的值）是不同的。

这些关于 Java 类的信息对于初学者来说已经足够了。尽管如此，我们还想描述一些其他类，这些类可以包含在同一个`.java`文件中，这样你就可以在其他人的代码中识别它们。这些其他类被称为**嵌套类**。它们只能从同一个文件中的类中访问。

我们之前描述的类-`.java`文件中唯一的一个公共类-也被称为顶级类。它可以包括一个称为内部类的嵌套类：

```java

public class MyClass { //顶级类

class MyOtherClass { //内部类

//内部类内容在这里

}

}

```

顶级类还可以包括一个静态（关于静态成员的更多信息请参见下一节）嵌套类。`static`类不被称为内部类，只是一个嵌套类：

```java

public class MyClass { //顶级类

static class MyYetAnotherClass { //嵌套类

//嵌套类内容在这里

}

}

```

任何方法都可以包括一个只能在该方法内部访问的类。它被称为本地类：

```java

public class MyClass { //顶级类

void someMethod（）{

class MyInaccessibleAnywhereElseClass { //本地类

//本地类内容在这里

}

}

}

```

本地类并不经常使用，但并不是因为它没有用。程序员只是不记得如何创建一个只在一个方法内部需要的类，而是创建一个外部或内部类。

最后但并非最不重要的一种可以包含在与公共类相同文件中的类是匿名类。它是一个没有名称的类，允许在原地创建一个对象，可以覆盖现有方法或实现一个接口。让我们假设我们有以下接口，`InterfaceA`，和类`MyClass`：

```java

public interface InterfaceA {

void doSomething（）;

}

public class MyClass {

void someMethod1（）{

System.out.println("1.常规被称为");

}

void someMethod2（InterfaceA interfaceA）{

interfaceA.doSomething（）;

}

}

```

我们可以执行以下代码：

```java

MyClass myClass = new MyClass（）;

myClass.someMethod1（）;

myClass = new MyClass（）{ //匿名类扩展类 MyClass

public void someMethod1（）{ //并覆盖 someMethod1（）

System.out.println("2.匿名被称为");

}

};

我的类。someMethod1（）;

myClass.someMethod2（new InterfaceA（）{ //匿名类实现

public void doSomething（）{ // InterfaceA

System.out.println("3.匿名被称为");

}

});

```

结果将是：

```java

1.常规被称为

2.匿名被称为

3.匿名被称为

```

我们不希望读者完全理解前面的代码。我们希望读者在阅读本书后能够做到这一点。

这是一个很长的部分，包含了很多信息。其中大部分只是供参考，所以如果你记不住所有内容，不要感到难过。在完成本书并获得一些 Java 编程的实际经验后，再回顾这一部分。

接下来还有几个介绍性部分。然后[第三章]（18c6e8b8-9d8a-4ece-9a3f-cd00474b713e.xhtml），*您的开发环境设置*，将引导您配置计算机上的开发工具，并且在[第四章]（64574f55-0e95-4eda-9ddb-b05da6c41747.xhtml），*您的第一个 Java 项目*，您将开始编写代码并执行它-每个软件开发人员都记得的时刻。

再走几步，你就可以称自己为 Java 程序员了。

# Java 对象（类实例）

人们经常阅读-甚至 Oracle 文档也不例外-对象被*用于模拟现实世界的对象*。这种观点起源于面向对象编程之前的时代。那时，程序有一个用于存储中间结果的公共或全局区域。如果不小心管理，不同的子例程和过程-那时称为方法-修改这些值，互相干扰，使得很难追踪缺陷。自然地，程序员们试图规范对数据的访问，并且使中间结果只能被某些方法访问。一组方法和只有它们可以访问的数据开始被称为对象。

这些构造也被视为现实世界对象的模型。我们周围的所有对象可能都有某种内在状态，但我们无法访问它，只知道对象的行为。也就是说，我们可以预测它们对这个或那个输入会有什么反应。在类（对象）中创建只能从同一类（对象）的方法中访问的私有字段似乎是隐藏对象状态的解决方案。因此，模拟现实世界对象的原始想法得以延续。

但是经过多年的面向对象编程，许多程序员意识到这样的观点可能会产生误导，并且在试图将其一贯应用于各种软件对象时实际上可能会产生相当大的危害。例如，一个对象可以携带用作算法参数的值，这与任何现实世界的对象无关，但与计算效率有关。或者，另一个例子，一个带回计算结果的对象。程序员通常称之为**数据传输对象**（**DTO**）。除非扩展现实世界对象的定义，否则它与现实世界对象无关，但那将是一个伸展。

软件对象只是计算机内存中的数据结构，实际值存储在其中。内存是一个现实世界的对象吗？物理内存单元是，但它们携带的信息并不代表这些单元。它代表软件对象的值和方法。关于对象的这些信息甚至不是存储在连续的内存区域中：对象状态存储在一个称为堆的区域中，而方法存储在方法区中，具体取决于 JVM 实现，可能或可能不是堆的一部分。

在我们的经验中，对象是计算过程的一个组成部分，通常不是在现实世界对象的模型上运行。对象用于传递值和方法，有时相关，有时不相关。方法和值的集合可能仅仅为了方便或其他考虑而被分组在一个类中。

公平地说，有时软件对象确实代表现实世界对象的模型。但关键是这并不总是如此。因此，除非真的是这样，让我们不将软件对象视为现实世界对象的模型。相反，让我们看看对象是如何创建和使用的，以及它们如何帮助我们构建有用的功能 - 应用程序。

正如我们在前一节中所描述的，对象是基于类创建的，使用关键字`new`和构造函数 - 要么是默认的，要么是显式声明的。例如，考虑以下类：

```java

public class MyClass {

private int field1;

private String field2;

public MyClass(int val1, String val2){

this.field1 = val1;

this.field2 = val2;

}

public String method1(int i){

//语句，包括返回语句

}

//... 其他方法在这里

}

```

如果我们有这个类，我们可以在其他类的方法中写以下内容：

```java

public AnotherClass {

...

public void someMethod(){

MyClass myClass = new MyClass(3, "some string");

String result = myClass.method1(2);

}

...

}

```

在前面的代码中，语句`MyClass myClass = new MyClass(3, "some string");`创建了一个`MyClass`类的对象，使用了它的构造函数和关键字`new`，并将新创建的对象的引用分配给变量`myClass`。我们选择了一个对象引用的标识符，它与类名匹配，第一个字母小写。这只是一个约定，我们也可以选择另一个标识符（比如`boo`），结果是一样的。在第五章中，*Java 语言元素和类型*，我们会更详细地讨论标识符和变量。正如你在前面的例子中看到的，在下一行中，一旦创建了一个引用，我们就可以使用它来访问新创建对象的公共成员。

任何 Java 对象都只能通过使用关键字（运算符）`new`和构造函数来创建。这个过程也被称为**类实例化**。对对象的引用可以像任何其他值一样传递（作为变量、参数或返回值），每个有权访问引用的代码都可以使用它来访问对象的公共成员。我们将在下一节中解释什么是**公共成员**。

# 类（静态）和对象（实例）成员

我们已经提到了与对象相关的公共成员这个术语。在谈到`main()`方法时，我们还使用了关键字`static`。我们还声明了一个被声明为`static`的成员在 JVM 内存中只能有一个副本。现在，我们将定义所有这些，以及更多。

# 私有和公共

关键字`private`和`public`被称为**访问修饰符**。还有默认和`protected`访问修饰符，但我们将在第七章中讨论它们，*包和可访问性（可见性）*。它们被称为访问修饰符，因为它们调节类、方法和字段的可访问性（有时也被称为可见性），并且它们修改相应的类、方法或字段的声明。

一个类只有在它是嵌套类时才能是私有的。在前面的*Java 类*部分，我们没有为嵌套类使用显式访问修饰符（因此，我们使用了默认的），但如果我们希望只允许从顶级类和同级访问这些类，我们也可以将它们设为私有。

私有方法或私有字段只能从声明它的类（对象）中访问。

相比之下，公共类、方法或字段可以从任何其他类中访问。请注意，如果封闭类是私有的，那么方法或字段就不能是公共的。这是有道理的，不是吗？如果类本身在公共上是不可访问的，那么它的成员如何能是公共的呢？

# 静态成员

只有当类是嵌套类时，才能声明一个类为静态。类成员——方法和字段——也可以是静态的，只要类不是匿名的或本地的。任何代码都可以访问类的静态成员，而不需要创建类实例（对象）。在前面的章节中，我们在一个代码片段中使用了类`Math`，就是这样的一个例子。静态类成员在字段的情况下也被称为类变量，方法的情况下被称为类方法。请注意，这些名称包含`class`这个词作为形容词。这是因为静态成员与类相关联，而不是与类实例相关联。这意味着在 JVM 内存中只能存在一个静态成员的副本，尽管在任何时刻可以创建和驻留在那里的类的许多实例（对象）。

这里是另一个例子。假设我们有以下类：

```java

公共类 MyClass {

私有 int 字段 1;

公共静态字符串字段 2;

public MyClass(int val1, String val2){

this.field1 = val1;

this.field2 = val2;

}

public String method1(int i){

//语句，包括返回语句

}

public static void method2(){

//语句

}

//... other methods are here

}

```

从任何其他类的任何方法，可以通过以下方式访问前述`MyClass`类的公共静态成员：

```java

MyClass.field2 = "any string";

String s = MyClass.field2 + " and another string";

```

前述操作的结果将是将变量`s`的值分配为`any string and another string`。`String`类将在第五章中进一步讨论，*Java 语言元素和类型*。

同样，可以通过以下方式访问类`MyClass`的公共静态方法`method2()`：

```java

MyClass.method2();

```

类`MyClass`的其他方法仍然可以通过实例（对象）访问：

```java

MyClass mc = new MyClass(3, "any string");

String someResult = mc.method1(42);

```

显然，如果所有成员都是静态的，就没有必要创建`MyClass`类的对象。

然而，有时可以通过对象引用访问静态成员。以下代码可能有效 - 这取决于`javac`编译器的实现。如果有效，它将产生与前面代码相同的结果：

```java

MyClass mc = new MyClass(3, "any string");

mc.field2 = "Some other string";

mc.method2();

```

有些编译器会提供警告，比如*通过实例引用访问静态成员*，但它们仍然允许你这样做。其他编译器会产生错误*无法使静态引用非静态方法/字段*，并强制你纠正代码。Java 规范不规定这种情况。但是，通过对象引用访问静态类成员不是一个好的做法，因为它使得代码对于人类读者来说是模棱两可的。因此，即使你的编译器更宽容，最好还是避免这样做。

# 对象（实例）成员

非静态类成员在字段的情况下也称为实例变量，或者在方法的情况下称为实例方法。它只能通过对象的引用后跟一个点“。”来访问。我们已经看到了几个这样的例子。

按照长期以来的传统，对象的字段通常声明为私有的。如果必要，提供`set()`和/或`get()`方法来访问这些私有值。它们通常被称为 setter 和 getter，因为它们设置和获取私有字段的值。这是一个例子：

```java

public class MyClass {

private int field1;

private String field2;

public void setField1(String val){

this.field1 = val;

}

public String getField1(){

return this.field1;

}

public void setField2(String val){

this.field2 = val;

}

public String getField2(){

return this.field2;

}

//... other methods are here

}

```

有时，有必要确保对象状态不能被改变。为了支持这种情况，程序员使用构造函数来设置状态并删除 setter：

```java

public class MyClass {

private int field1;

private String field2;

public MyClass(int val1, String val2){

this.field1 = val1;

this.field2 = val2;

}

public String getField1(){

return this.field1;

}

public String getField2(){

return this.field2;

}

//... other non-setting methods are here

}

```

这样的对象称为不可变的。

# 方法重载

具有相同名称但不同签名的两个方法代表方法重载。这是一个例子：

```java

public class MyClass {

public String method(int i){

//statements

}

public int method(int i, String v){

//statements

}

}

```

以下是不允许的，会导致编译错误，因为返回值不是方法签名的一部分，如果它们具有相同的签名，则无法用于区分一个方法和另一个方法：

```java

public class MyClass {

public String method(int i){

//statements

}

public int method(int i){ //error

//statements

}

}

```

然而，这是允许的，因为这些方法具有不同的签名：

```java

public String method(String v, int i){

//statements

}

public String method(int i, String v){

//语句

}

```

# 接口、实现和继承

现在，我们要进入 Java 编程的最重要领域——接口、实现和继承这些广泛使用的 Java 编程术语。

# 接口

在日常生活中，“接口”这个词非常流行。它的含义与 Java 接口在编程中所扮演的角色非常接近。它定义了对象的公共界面。它描述了如何与对象进行交互以及可以期望它具有什么。它隐藏了内部类的工作原理，只公开了具有返回值和访问修饰符的方法签名。接口不能被实例化。接口类型的对象只能通过创建实现该接口的类的对象来创建（接口实现将在下一节中更详细地介绍）。

例如，看下面的类：

```java

public class MyClass {

private int field1;

private String field2;

public MyClass(int val1, String val2){

this.field1 = val1;

this.field2 = val2;

}

public String method(int i){

//语句

}

public int method(int i, String v){

//语句

}

}

```

它的接口如下：

```java

public interface MyClassInterface {

String method(int i);

int method(int i, String v);

}

```

因此，我们可以写`public class MyClass implements MyClassInterface {...}`。我们将在下一节中讨论它。

由于接口是*公共*的界面，默认情况下假定方法访问修饰符`public`，可以省略。

接口不描述如何创建类的对象。要发现这一点，必须查看类并查看它的构造函数的签名。还可以检查并查看是否存在可以在不创建对象的情况下访问的公共静态类成员。因此，接口只是类*实例*的公共界面。

让我们来看看接口的其余功能。根据 Java 规范，*接口的主体可以声明接口的成员，即字段、方法、类和接口。*如果您感到困惑，并问接口和类之间的区别是什么，您有一个合理的关注，我们现在将解决这个问题。

接口中的字段隐式地是公共的、静态的和最终的。修饰符`final`表示它们的值不能被改变。相比之下，在类中，类本身、它的字段、方法和构造函数的默认访问修饰符是包私有的，这意味着它只在自己的包内可见。包是相关类的命名组。您将在第七章中了解它们，*包和可访问性（可见性）*。

接口主体中的方法可以声明为默认、静态或私有。默认方法的目的将在下一节中解释。静态方法可以通过接口名称和点“`.`”从任何地方访问。私有方法只能被同一接口内的其他方法访问。相比之下，类中方法的默认访问修饰符是包私有的。

至于在接口内声明的类，它们隐式地是静态的。它们也是公共的，可以在没有接口实例的情况下访问，而创建接口实例是不可能的。我们不会再多谈论这样的类，因为它们用于超出本书范围的非常特殊的领域。

与类类似，接口允许在其内部声明内部接口。可以像任何静态成员一样从外部访问它，使用顶级接口和点“`.`”。我们想提醒您，接口默认是公共的，不能被实例化，因此默认是静态的。

There is one last very important term related to interfaces. A method signature listed in the interface without an implementation is called an **abstract method** and the interface itself is called **abstraction** because it abstracts, summarizes, and removes the signatures from the implementation. An abstraction cannot be instantiated. As an example, if you put the keyword `abstract` in front of any class and try to create its object, the compiler will throw an error even if all the methods in the class are not abstract. In such a case, the class behaves as an interface with the default methods only. Yet, there is a significant difference in their usage, which you will see after reading the upcoming *Inheritance* section of this chapter.

We will talk more about interfaces in Chapter 6, *Interfaces, Classes, and Objects Construction*, and cover their access modifiers in Chapter 7, *Packages and Accessibility (Visibility)*.

# Implementation

An interface can be implemented by a class, which means that the class has a body for each of the abstract methods listed in the interface. Here is an example:

```java

interface Car {

double getWeightInPounds();

double getMaxSpeedInMilesPerHour();

}

public class CarImpl implements Car{

public double getWeightInPounds(){

return 2000d;

}

public double getMaxSpeedInMilesPerHour(){

return 100d;

}

}

```

We named the class `CarImpl` to indicate that it is an implementation of the interface `Car`. But we could name it any other way we like.

Both interface and its class implementation can have other methods too without causing a compiler error. The only requirement for the extra method in the interface is that it has to be default and have a body. Adding any other method to a class does not interfere with the interface implementation.  For example:

```java

interface Car {

double getWeightInPounds();

double getMaxSpeedInMilesPerHour();

default int getPassengersCount(){

return 4;

}

}

public class CarImpl implements Car{

private int doors;

private double weight, speed;

public CarImpl(double weight, double speed, int doors){

this.weight = weight;

this.speed = speed;

this.dooes = doors;

}

public double getWeightInPounds(){

return this.weight;

}

public double getMaxSpeedInMilesPerHour(){

return this.speed;

}

public int getNumberOfDoors(){

return this.doors;

}

}

```

If we now create an instance of a class `CarImpl`, we can call all the methods we have declared in the class:

```java

CarImpl car = new CarImpl(500d, 50d, 3);

car.getWeightInPounds();         //Will return 500.0

car.getMaxSpeedInMilesPerHour(); //Will return 50.0

car.getNumberOfDoors();          //Will return 3

```

That was not surprising.

But, here is something you might not have expected:

```java

car.getPassengersCount();          //Will return 4

```

This means that by implementing an interface class acquires all the default methods the interface has. That is the purpose of the default methods: to add functionality to all classes that implement the interface. Without it, if we add an abstract method to an old interface, all current interface implementations will trigger a compiler error. But, if we add a new method with the modifier default, the existing implementations will continue working as usual.

Now, another nice trick. If a class implements a method with the same signature as the default method, it will `override` (a technical term) the behavior of the interface. Here is an example:

```java

interface Car {

double getWeightInPounds();

double getMaxSpeedInMilesPerHour();

default int getPassengersCount(){

return 4;

}

}

public class CarImpl implements Car{

private int doors;

private double weight, speed;

public CarImpl(double weight, double speed, int doors){

this.weight = weight;

this.speed = speed;

this.dooes = doors;

}

public double getWeightInPounds(){

return this.weight;

}

public double getMaxSpeedInMilesPerHour(){

return this.speed;

}

public int getNumberOfDoors(){

return this.doors;

}

public int getPassengersCount(){

返回 3;

}

}

```

如果我们使用本例中描述的接口和类，我们可以编写以下代码：

```java

CarImpl car = new CarImpl(500d, 50d, 3);

car.getPassengersCount();        //现在将返回 3 !!!!

```

如果接口的所有抽象方法都没有被实现，那么类必须声明为抽象类，并且不能被实例化。

接口的目的是代表它的实现-所有实现它的类的所有对象。例如，我们可以创建另一个实现`Car`接口的类：

```java

public class AnotherCarImpl implements Car{

public double getWeightInPounds(){

返回 2d;

}

public double getMaxSpeedInMilesPerHour(){

返回 3d;

}

public int getNumberOfDoors(){

返回 4;

}

public int getPassengersCount(){

return 5;

}

}

```

然后我们可以让`Car`接口代表它们中的每一个：

```java

Car car = new CarImpl(500d, 50d, 3);

car.getWeightInPounds();          //将返回 500.0

car.getMaxSpeedInMilesPerHour();  //将返回 50.0

car.getNumberOfDoors();           //将产生编译器错误

car.getPassengersCount();         //仍然返回 3 !!!!

car = new AnotherCarImpl();

car.getWeightInPounds();          //将返回 2.0

car.getMaxSpeedInMilesPerHour();  //将返回 3.0

car.getNumberOfDoors();           //将产生编译器错误

car.getPassengersCount();         //将返回 5

```

从前面的代码片段中可以得出一些有趣的观察。首先，当变量`car`声明为接口类型时（而不是类类型，如前面的例子），不能调用接口中未声明的方法。

其次，`car.getPassengersCount()`方法第一次返回`3`。人们可能期望它返回`4`，因为`car`被声明为接口类型，人们可能期望默认方法起作用。但实际上，变量`car`指的是`CarImpl`类的对象，这就是为什么执行`car.getPassengersCount()`方法的是类的实现。

使用接口时，应该记住签名来自接口，但实现来自类，或者来自默认接口方法（如果类没有实现它）。这里还有默认方法的另一个特性。它们既可以作为可以实现的签名，也可以作为实现（如果类没有实现它）。

如果接口中有几个默认方法，可以创建私有方法，只能由接口的默认方法访问。它们可以用来包含公共功能，而不是在每个默认方法中重复。私有方法无法从接口外部访问。

有了这个，我们现在可以达到 Java 基础知识的高峰。在此之后，直到本书的结尾，我们只会添加一些细节并增强您的编程技能。这将是在高海拔高原上的一次漫步-您走得越久，就会感到越舒适。但是，要到达那个高度，我们需要爬上最后的上坡路；继承。

# 继承

一个类可以获取（继承）所有非私有非静态成员，因此当我们使用这个类的对象时，我们无法知道这些成员实际上位于哪里-在这个类中还是在继承它们的类中。为了表示继承，使用关键字`extends`。例如，考虑以下类：

```java

class A {

private void m1(){...}

public void m2(){...}

}

class B extends class A {

public void m3(){...}

}

class C extends class B {

}

```

在这个例子中，类`B`和`C`的对象的行为就好像它们各自有方法`m2()`和`m3()`。唯一的限制是一个类只能扩展一个类。类`A`是类`B`和类`C`的基类。类`B`只是类`C`的基类。正如我们已经提到的，它们每个都有默认的基类`java.lang.Object`。类`B`和`C`是类`A`的子类。类`C`也是类`B`的子类。

相比之下，一个接口可以同时扩展许多其他接口。如果`AI`，`BI`，`CI`，`DI`，`EI`和`FI`是接口，那么允许以下操作：

```java

接口 AI 扩展 BI，CI，DI {

//接口主体

}

接口 DI 扩展 EI，FI {

//接口主体

}

```

在上述例子中，接口`AI`继承了接口`BI`，`CI`，`DI`，`EI`和`FI`的所有非私有非静态签名，以及任何其他是接口`BI`，`CI`，`DI`，`EI`和`FI`的基接口。

回到上一节的话题，*实现*，一个类可以实现多个接口：

```java

类 A 扩展 B 实现 AI，BI，CI，DI {

//类主体

}

```

这意味着类`A`继承了类`B`的所有非私有非静态成员，并实现了接口`AI`，`BI`，`CI`和`DI`，以及它们的基接口。实现多个接口的能力来自于前面的例子，如果重写成这样，结果将完全相同：

```java

接口 AI 扩展 BI，CI，DI {

//接口主体

}

类 A 扩展 B 实现 AI {

//类主体

}

```

`扩展`接口（类）也称为超级接口（超类）或父接口（父类）。扩展接口（类）称为子接口（子类）或子接口（子类）。

让我们用例子来说明这一点。我们从接口继承开始：

```java

接口车辆 {

double getWeightInPounds();

}

接口 Car 扩展车辆 {

int getPassengersCount();

}

public class CarImpl 实现 Car {

public double getWeightInPounds(){

return 2000d;

}

public int getPassengersCount(){

return 4;

}

}

```

在上述代码中，类`CarImpl`必须实现两个签名（列在接口`Vehicle`和接口`Car`中），因为从它的角度来看，它们都属于接口`Car`。否则，编译器会抱怨，或者类`CarImpl`必须声明为抽象的（不能被实例化）。

现在，让我们看另一个例子：

```java

接口车辆 {

double getWeightInPounds();

}

public class VehicleImpl 实现车辆 {

public double getWeightInPounds(){

return 2000d;

}

}

接口 Car 扩展车辆 {

int getPassengersCount();

}

public class CarImpl 扩展 VehicleImpl 实现 Car {

public int getPassengersCount(){

return 4;

}

}

```

在这个例子中，类`CarImpl`不需要实现`getWeightInPounds()`的抽象方法，因为它已经从基类`VehicleImpl`继承了实现。

所述类继承的一个后果通常对于初学者来说并不直观。为了证明这一点，让我们在类`CarImpl`中添加方法`getWeightInPounds()`：

```java

public class VehicleImpl {

public double getWeightInPounds(){

return 2000d;

}

}

public class CarImpl 扩展 VehicleImpl {

public double getWeightInPounds(){

return 3000d;

}

public int getPassengersCount(){

return 4;

}

}

```

在这个例子中，为了简单起见，我们不使用接口。因为类`CarImpl`是类`VehicleImpl`的子类，它可以作为类`VehicleImpl`的对象行为，这段代码将编译得很好：

```java

VehicleImpl vehicle = new CarImpl();

vehicle.getWeightInPounds();

```

问题是，你期望在前面片段的第二行中返回什么值？如果你猜测是 3,000，你是正确的。如果不是，不要感到尴尬。习惯需要时间。规则是，基类类型的引用可以引用其任何子类的对象。它被广泛用于覆盖基类行为。

峰会就在眼前。只剩下一步了，尽管它带来了一些你在读这本书之前可能没有预料到的东西，如果你对 Java 一无所知。

# java.lang.Object 类

所以，这里有一个惊喜。每个 Java 类，默认情况下（没有显式声明），都扩展了`Object`类。准确地说，它是`java.lang.Object`，但我们还没有介绍包，只会在第七章中讨论它们，*包和可访问性（可见性）*。

所有 Java 对象都继承了它的所有方法。共有十个：

+   `public boolean equals (Object obj)`

+   `public int hashCode()`

+   `public Class getClass()`

+   `public String toString()`

+   `protected Object clone()`

+   `public void wait()`

+   `public void wait(long timeout)`

+   `public void wait(long timeout, int nanos)`

+   `public void notify()`

+   `public void notifyAll()`

让我们简要地访问每个方法。

在我们这样做之前，我们想提一下，你可以在你的类中重写它们的默认行为，并以任何你需要的方式重新实现它们，程序员经常这样做。我们将在第六章中解释如何做到这一点，*接口、类和对象构造*。

# equals()方法

`java.lang.Object`类的`equals()`方法看起来是这样的：

```java

public boolean equals(Object obj) {

//比较当前对象的引用

//和引用对象

}

```

这是它的使用示例：

```java

Car car1 = new CarImpl();

Car car2 = car1;

Car car3 = new CarImpl();

car1.equals(car2);    //返回 true

car1.equals(car3);    //返回 false

```

从前面的例子中可以看出，默认方法`equals()`的实现只比较指向存储对象的地址的内存引用。这就是为什么引用`car1`和`car2`是相等的——因为它们指向同一个对象（内存的相同区域，相同的地址），而`car3`引用指向另一个对象。

`equals()`方法的典型重新实现使用对象的状态进行比较。我们将在第六章中解释如何做到这一点，*接口、类和对象构造*。

# `hashCode()`方法

`java.lang.Object`类的`hashCode()`方法看起来是这样的：

```java

public int hashCode(){

//返回对象的哈希码值

//基于内存地址的整数表示

}

```

Oracle 文档指出，如果两个方法根据`equals()`方法的默认行为是相同的，那么它们具有相同的`hashCode()`返回值。这很棒！但不幸的是，同一份文档指出，根据`equals()`方法，两个不同的对象可能具有相同的`hasCode()`返回值。这就是为什么程序员更喜欢重新实现`hashCode()`方法，并在重新实现`equals()`方法时使用它，而不是使用对象状态。尽管这种需要并不经常出现，我们不会详细介绍这种实现的细节。如果感兴趣，你可以在互联网上找到很好的文章。

# `getClass()`方法

`java.lang.Object`类的`getClass()`方法看起来是这样的：

```java

public Class getClass(){

//返回具有的 Class 类的对象

//提供有用信息的许多方法

}

```

从这个方法中最常用的信息是作为当前对象模板的类的名称。我们将在第六章中讨论为什么可能需要它，*接口、类和对象构造**.*可以通过这个方法返回的`Class`类的对象来访问类的名称。

# `toString()`方法

`java.lang.Object`类的`toString()`方法看起来像这样：

```java

public String toString(){

//返回对象的字符串表示

}

```

这个方法通常用于打印对象的内容。它的默认实现看起来像这样：

```java

public String toString() {

return getClass().getName()+"@"+Integer.toHexString(hashCode());

}

```

正如你所看到的，它并不是非常具有信息性，所以程序员们会在他们的类中重新实现它。这是类`Object`中最常重新实现的方法。程序员们几乎为他们的每个类都这样做。我们将在第九章中更详细地解释`String`类及其方法，*运算符、表达式和语句*。

# `clone()`方法

`java.lang.Object`类的`clone()`方法看起来像这样：

```java

protected Object clone(){

//创建对象的副本

}

```

这个方法的默认结果返回对象字段的副本，这是可以接受的，如果值不是对象引用。这样的值被称为**原始类型**，我们将在第五章中精确定义，*Java 语言元素和类型*。但是，如果对象字段持有对另一个对象的引用，那么只有引用本身会被复制，而不是引用的对象本身。这就是为什么这样的副本被称为浅层副本。要获得深层副本，必须重新实现`clone()`方法，并遵循可能相当广泛的对象树的所有引用。幸运的是，`clone()`方法并不经常使用。事实上，你可能永远不会遇到需要使用它的情况。

在阅读本文时，你可能会想知道，当对象被用作方法参数时会发生什么。它是使用`clone()`方法作为副本传递到方法中的吗？如果是，它是作为浅层副本还是深层副本传递的？答案是，都不是。只有对象的引用作为参数值传递进来，所以所有接收相同对象引用的方法都可以访问存储对象状态的内存区域。

这为意外数据修改和随后的数据损坏带来了潜在风险，将它们带入不一致的状态。这就是为什么，在传递对象时，程序员必须始终意识到他们正在访问可能在其他方法和类之间共享的值。我们将在第五章中更详细地讨论这一点，并在第十一章中扩展这一点，*JVM 进程和垃圾回收*，在讨论线程和并发处理时。

# The wait() and notify() methods

`wait()`和`notify()`方法及其重载版本用于线程之间的通信——轻量级的并发处理进程。程序员们不会重新实现这些方法。他们只是用它们来增加应用程序的吞吐量和性能。我们将在第十一章中更详细地讨论`wait()`和`notify()`方法，*JVM 进程和垃圾回收*。

现在，恭喜你。你已经踏上了 Java 基础复杂性的高峰，现在将继续水平前行，添加细节并练习所学知识。在阅读前两章的过程中，你已经在脑海中构建了 Java 知识的框架。如果有些东西不清楚或者忘记了，不要感到沮丧。继续阅读，你将有很多机会来刷新你的知识，扩展它，并保持更长时间。这将是一段有趣的旅程，最终会有一个不错的奖励。

# 面向对象编程概念

现在，我们可以谈论一些对你来说更有意义的概念，与在你学习主要术语并看到代码示例之前相比。这些概念包括：

+   对象/类：它将状态和行为保持在一起

+   封装：它隐藏了状态和实现的细节

+   继承：它将行为/签名传播到类/接口扩展链中

+   接口：它将签名与实现隔离开来

+   多态：这允许一个对象由多个实现的接口和任何基类表示，包括`java.lang.Object`。

到目前为止，你已经熟悉了上述所有内容，因此这将主要是一个总结，只添加一些细节。这就是我们学习的方式——观察特定事实，构建更大的图景，并随着新的观察不断改进这个图景。我们一直在做这件事，不是吗？

# 对象/类

一个 Java 程序和整个应用程序可以在不创建一个对象的情况下编写。只需在你创建的每个类的每个方法和每个字段前面使用`static`关键字，并从静态的`main()`方法中调用它们。你的编程能力将受到限制。你将无法创建一支可以并行工作的对象军队，他们可以在自己的数据副本上做类似的工作。但你的应用程序仍然可以工作。

此外，在 Java 8 中，添加了函数式编程特性，允许我们像传递对象一样传递函数。因此，你的无对象应用程序可能会非常强大。而且，一些没有对象创建能力的语言被使用得非常有效。然而，在面向对象的语言被证明有用并变得流行之后，第一个是 Smalltalk，一些传统的过程式语言，如 PHP、Perl、Visual Basic、COBOL 2002、Fortran 2003 和 Pascal 等，都添加了面向对象的能力。

正如我们刚才提到的，Java 还将其功能扩展到覆盖函数式编程，从而模糊了过程式、面向对象和函数式语言之间的界限。然而，类的存在和使用它们来创建对象的能力是编程语言必须支持的第一个概念，才能被归类为面向对象。

# 封装

封装——使数据和函数（方法）无法从外部访问或者有受控的访问——是创建面向对象语言的主要驱动因素之一。Smalltalk 是基于对象之间的消息传递的想法创建的，当一个对象调用另一个对象的方法时，这在 Smalltalk 和 Java 中都是这样做的。

封装允许调用对象的服务，而不知道这些服务是如何实现的。它减少了软件系统的复杂性，增加了可维护性。每个对象都可以独立地完成其工作，而无需与其客户端协调实现的更改，只要它不违反接口中捕获的合同。

我们将在第七章中进一步详细讨论封装，*包和可访问性（可见性）*。

# 继承

继承是另一个面向对象编程概念，受到每种面向对象语言的支持。通常被描述为能够重用代码的能力，这是一个真实但经常被误解的说法。一些程序员认为继承能够在应用程序之间实现代码的重用。根据我们的经验，应用程序之间的代码重用可以在没有继承的情况下实现，并且更多地依赖于应用程序之间的功能相似性，而不是特定的编程语言特性。这更多地与将通用代码提取到共享可重用库中的技能有关。

在 Java 或任何其他面向对象的语言中，继承允许在基类中实现的公共功能*在其子类中重用*。它可以用于通过将基类组装到一个共享的库中，实现模块化并提高代码的可重用性。但在实践中，这种方法很少被使用，因为每个应用程序通常具有特定的要求，一个共同的基类要么太简单而实际上无用，要么包含许多特定于每个应用程序的方法。此外，在第六章《接口、类和对象构造》中，我们将展示，使用聚合更容易实现可重用性，这是基于使用独立对象而不是继承。

与接口一起，继承使多态成为可能。

# 接口（抽象）

有时，接口的面向对象编程概念也被称为抽象，因为接口总结（抽象）了对象行为的公共描述，隐藏了其实现的细节。接口是封装和多态的一个组成部分，但足够重要，以至于被作为一个单独的概念来阐述。其重要性将在第八章《面向对象设计（OOD）原则》中变得特别明显，当我们讨论从项目想法和愿景到具体编程解决方案的过渡时。

接口和继承为多态提供了基础。

# 多态

从我们提供的代码示例中，您可能已经意识到，一个对象具有所有实现的接口中列出的方法和其基类的所有非私有非静态方法，包括`java.lang.Object`。就像一个拥有多重国籍的人一样，它可以被视为其基类或实现的接口的对象。这种语言能力被称为多态（来自*poly* - 许多和*morphos* - 形式）。

请注意，广义上讲，方法重载——当具有相同名称的方法根据其签名可以具有不同行为时——也表现出多态行为。

# 练习-接口与抽象类

接口和抽象类之间有什么区别？我们没有讨论过，所以您需要进行一些研究。

在 Java 8 中引入接口的默认方法后，差异显著缩小，在许多情况下可以忽略不计。

# 答案

抽象类可以有构造函数，而接口不能。

抽象类可以有状态，而接口不能。抽象类的字段可以是私有的和受保护的，而在接口中，字段是公共的、静态的和最终的。

抽象类可以具有任何访问修饰符的方法实现，而接口中实现的默认方法只能是 public。

如果您想要修改的类已经扩展到另一个类，您就不能使用抽象类，但是您可以实现一个接口，因为一个类只能扩展到另一个类，但可以实现多个接口。

# 总结

在本章中，您已经学习了 Java 和任何面向对象编程语言的基本概念。您现在了解了类和对象作为 Java 的基本构建模块，知道了静态和实例成员是什么，以及了解了接口、实现和继承。这是本初学者章节中最复杂和具有挑战性的练习，将读者带到了 Java 语言的核心，介绍了我们将在本书的其余部分中使用的语言框架。这个练习让读者接触到了关于接口和抽象类之间差异的讨论，这在 Java 8 发布后变得更加狭窄。

在下一章中，我们将转向编程的实际问题。读者将被引导完成在他们的计算机上安装必要工具和配置开发环境的具体步骤。之后，所有新的想法和软件解决方案将被演示，包括具体的代码示例。
