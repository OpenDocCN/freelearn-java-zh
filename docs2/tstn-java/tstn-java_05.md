# 5

# 语言基础 – 类

**面向对象**（**OO**）程序基于称为类的结构的设计，这些结构用作对象的蓝图。对象是类的实现。这意味着在**面向对象编程**（**OOP**）中编码的第一步是创建类。本章将探讨 Java 中如何实现 OOP 的特性。我们首先看看如何在类中定义变量，然后是控制类成员及其本身的访问方式。从这里开始，我们将查看 Java 为我们提供的用于创建或处理类和对象的类结构。

在本章中，你将学习以下内容：

+   类字段

+   理解访问控制

+   理解类

到本章结束时，你将能够定义类，将它们实例化为对象，并与其他类交互。让我们从查看访问控制开始。在我们开始之前，让我们看看你可以在类中声明的两种变量类别。

# 技术要求

这里是运行本章示例所需的工具：

+   已安装 Java 17

+   文本编辑器

+   已安装 Maven 3.8.6 或更高版本

你可以在 GitHub 仓库[`github.com/PacktPublishing/Transitioning-to-Java/tree/chapter05`](https://github.com/PacktPublishing/Transitioning-to-Java/tree/chapter05)中找到本章的代码。

# 类字段

在类中声明而不是在方法调用中声明的变量被称为字段。它们分为两类：

+   实例变量

+   类变量

在类中定义`double`，如果我们创建了 100 个对象实例，我们就会有 100 个`double`。

在 Java 中，你可以在类中有一个变量，它被从该类创建的所有对象共享。换句话说，每个对象都有一个唯一的实例变量集，但所有对象都共享**类变量**。这是通过将变量指定为静态来实现的。静态变量只有一个内存分配。在我们的 100 个对象实例中，如果你将`double`声明为静态，就只有一个`double`。

静态或类变量的另一个特点是，假设它具有公共访问控制，你可以不实例化对象即可访问它。例如，考虑以下代码块：

```java
 public class TestBed {
    public static String bob;
```

在这个片段中，我们可以通过简单地写出`TestBed.bob`来访问`class`变量`bob`。我们不需要实例化对象。如果我们实例化了它，我们也可以使用引用，尽管这种情况很少。

类变量被所有对象共享的事实使它们成为同一类创建的对象之间通信的理想工具。如果一个对象改变了类变量的值，那么即使是同一类的另一个对象也可以看到更新后的值。

在我们进入下一节关于访问控制的讨论之前，让我们澄清一个术语。我们将所有在类中声明的变量称为**字段**。这包括类和实例变量。

# 理解访问控制

面向对象编程（OOP）的一个显著且宝贵的特性是**访问控制**。如果你已经使用过面向对象的编程语言，那么你可能已经熟悉这个概念；如果不是，让我来解释一下访问控制是什么意思。

Java 中的访问控制涉及类、字段和方法对其他类的可见性。你必须有足够的访问权限来创建对象并访问类中的字段和方法。

在其他语言中，访问控制可能意味着一种安全机制，可以确保对方法访问的请求（例如）来自经过身份验证的用户。Java 并不是这样做的；在 Java 中，它关乎对象如何相互交互。

让我们来看看可见性的选项；第一个将是 Java 包。

## 包

访问控制难题的第一部分是 Java 包及其相应的`import`语句。在*第三章*《Maven 构建工具》中，我们学习了 Java 包以及它们是如何简单地包含 Java 代码的文件夹。除非你通过包含`import`语句来提供访问权限，否则一个包中的代码不能访问另一个包中的代码。导入类的类可以访问它导入的代码。如果没有`import`类，不同包中的`statement`类不能相互交互。

请记住，这种交互是单向的。例如，类 A 对类 B 有一个`import`语句。类 A 可以调用或向类 B 中的代码发送消息，但不能反过来。你可以在类 B 中为类 A 添加一个`import`语句，然后它们可以相互发送消息。

使用包进行访问控制很简单；这是一个简单的二进制设置。你可以看到你导入的类的对象，或者如果你没有导入它，你根本看不到这个类。同一个包中的所有类都对同一包中的其他类有隐式导入，在这种情况下，没有必要显式导入同包中的类。

现在，我们准备查看我们可用的四个访问控制指定符。

## 公共指定符

如果需要，`import`语句可以创建任何其他对象的公共类的对象。

你可以从任何拥有指向第二个对象的引用的对象中访问公共类的字段。我们应该始终将类字段保持为私有，这样我们就不能直接向字段赋值。要与私有变量交互，你需要同一类中的方法，这些方法将成为你读取或写入私有变量的代理。在向字段写入时，你将能够在将其分配给字段之前验证新值。

你可以从任何拥有指向包含公共方法的对象的引用的其他对象中调用公共方法。我们将类中的公共方法称为其接口。我们将在下一章中探讨接口。

## 私有指定符

Java 中的**私有指定符**与 C++和 C#相同——您使用此指定符来定义字段和方法访问控制。

如前所述，类字段始终应该是私有的。您需要验证您想要存储在私有字段中的数据。一种常见的方法是使用一个修改器，通常称为 setter 方法。在这里，您可以添加验证并通过抛出异常拒绝无效数据。

被指定为私有的方法只能由同一类中的其他方法调用。使用私有方法允许我们将复杂任务分解成更小的单元。由于它们是私有的，因此不能从其他对象调用，这确保了复杂任务的必要步骤将按正确顺序执行。

Java 允许您在另一个类中定义一个新类。这是唯一一个类可能是私有的情况。一个私有的非内部类不能被实例化。您不能在声明它的类之外使用私有内部类。您可以在声明它的类中实例化私有内部类。如前所述，实例变量始终应该是私有的，而方法可以是四种访问指定中的任何一种。

## 受保护的指定符

`protected/package`只在您使用**继承**时使用。在非继承情况下，受保护的属性表现得与包访问相同。在 C++和 C#中，包的概念不存在，因此在非继承情况下，这些语言将受保护的属性视为私有。

正如我们在下一章中将要看到的，继承是两个类之间的安排，其中一个类是**超类**，另一个类是**子类**。子类可以访问其超类的所有公共成员，包括在超类中指定的受保护实例变量。与继承无关且引用超类的其他对象将受保护的成员视为私有，或者如果这些对象定义在同一个包中，则视为包。

您指定的受保护的方法和类变量也具有包访问权限，如下一节所述。您不能有受保护的类。

## 包指定符

这个最终的访问指定符包在 C++或 C#中没有等效项，尽管这些语言中的 friend 概念与之有些相似。它定义了在同一个包中定义的其他对象中类的字段、字段和方法的可视性。没有类似于 public、private 或 protected 这样的指定符。当在类、字段或方法上没有显式使用指定符时，则隐式访问控制是包。

在不同包或文件夹中定义的另一个对象的引用被视为私有。同一包中两个不同类的对象可以像访问公共元素一样访问受保护的元素。

注意

在我们继续之前，最后一点——谨慎使用`protected`和包指定符。它们存在于可以加快对象之间交互的情况。问题是它们将字段和方法暴露给不应访问它们的对象。我建议你在开始设计和编写程序时只使用公共和私有。只有在你能够证明仅使用公共或私有组件会导致系统性能下降时，才考虑使用`protected`和包。

最后一点需要重申——在类内部不存在访问控制。这意味着一个公共方法可以调用同一类中的`private`方法。无论其访问控制指定如何，每个方法都可以访问每个字段。

现在，我们准备查看类是如何工作的。我们首先将查看上一章程序中的类，然后我们将创建程序的更新版本。

# 理解类

Java，与其他面向对象的语言一样，使用一种围绕源代码结构的语法，称为**类**。但首先，什么是类？引入对象概念的理论家们将类视为自定义数据类型。想象一下原始整数类型——它有一个允许的值范围和一组预定义的操作，如加法、减法和其他常用运算符。想象一下类作为一个自定义原始类型，你决定你的类型将执行哪些操作，以方法的形式。面向对象编程的一个目标是通过开发将数据和动作结合在一起的自定义数据类型来专注于问题解决，而不是结构化编程方法，其中数据和动作是分开的。

这意味着你通过首先列出类的所有字段来开发一个类，无论是原始类型还是其他类的引用。接下来是执行有用任务的方法，这些任务利用了这些字段。这些变量对类中的每个方法都是可见的，无论其访问控制级别如何。

类不是可执行代码，有一个例外我们很快就会看到。相反，类是一个蓝图，必须在运行时使用`new`关键字实例化或创建。当你的程序开始执行时，**Java 虚拟机**（**JVM**）将类定义存储在称为**类方法区域**的内存区域中。

`new`运算符执行两个任务：

+   为类蓝图中的所有实例变量分配足够的内存。我们称这个内存区域为`Class-Method`区域。

+   将分配的内存区域的地址分配给适当的引用变量。引用变量始终只有 4 个字节长。通过引用变量访问对象称为间接寻址。

存在第三个内存区域，称为栈。JVM 按照需要将所有局部变量——在方法中声明的变量——存储在栈数据结构中。栈是一个动态结构，可以为变量分配空间，然后通过移动指针来释放它们。如果你对内存管理感兴趣，请参阅本章末尾的*进一步阅读*部分以获取更多信息。

在我们继续之前，我们需要了解 JVM 在创建对象时可以调用的两种方法。

## 构造函数和 finalize 方法

在大多数面向对象的语言中，与内存管理相关的有两种特殊方法。`constructor`方法在对象的创建过程中的最后一步运行，而`destructor`方法，在 Java 中称为`finalize`，在对象超出作用域时作为第一步运行。Java 中这种情况与其他语言，如 C++，是不同的。

### finalize

一个类中只能有一个`finalize`方法。你不能对其进行重载。不要使用它——Java 9 已经弃用了它。如果你是 C++开发者，你可能会错误地认为`finalize`是 C++析构函数的 Java 等价物。然而，这并不正确——原因如下。

在 C++中，`delete`运算符首先运行析构函数。一旦析构函数运行，`delete`运算符就会释放对象使用的内存，现在可以重新分配。这是因为当你对一个有效指针执行`delete`运算符时，析构函数中的操作会立即执行。

在 Java 中，没有`delete`运算符；相反，JVM 监视所有对象的引用。当一个对象引用超出作用域时，JVM 将其添加到 JVM 将为你释放的引用列表中。然而，为了效率和性能，JVM 不会立即释放内存。相反，它会尽可能推迟。这是由于释放内存所需的时间，这可能会影响 JVM 中当前运行的程序。我们称内存释放为**垃圾回收**。

JVM 在垃圾回收之前运行`finalize`方法。这种垃圾回收可能每几分钟或更少发生，但这非常不可能。考虑到大量的 RAM（我的系统有 32 GB），这意味着垃圾回收可能每几个小时或甚至几天发生一次。因此，`finalize`是非确定性的。它可能因为不再有效的原因而想要影响程序的一部分或发送消息，或者该程序的一部分已经被垃圾回收。

由于这个和其他原因，Java 架构师决定弃用`finalize`。所以，不要使用它。如果你想在对象超出作用域之前调用一个方法，那么你必须显式地调用你创建的方法。

在讨论了`finalize`方法并将其分配给垃圾或弃用回收站之后，让我们来看看构造函数。

### 构造函数

构造函数的目的是在对象创建的最后一步执行任何必要的操作。通常，您使用构造函数来初始化类实例变量。我们已经看到我们可以在类的声明点直接初始化这些变量。有时，初始化需要比仅仅赋值更多的步骤，这就是构造函数非常有价值的地方。

您通过调用`new`运算符来创建一个对象。`new`运算符执行导致创建对象的任务。我在简化这个过程，但这是您需要了解的内容：

1.  所有实例变量以及其他 JVM 所需的所需结构都在内存的堆区域分配内存。这个内存的地址就是`this`引用捕获的地址。随后，对这个类中非静态方法的每次调用现在都有一个第一个参数，即不可见的`this`参数。

让我们编写以下代码：

```java
public class MyClass() {
   public void doSomething(int value) { … }
   . . .
}
```

然后，我们将实例化它，如下所示：

```java
var testClass = new MyClass();
```

编译后，非静态方法变成了以下内容：

```java
   public void doSomething(MyClass this, int value) { … }
```

您永远不能这样编写代码。`MyClass this`是隐含的，因此可能不写。

现在，我们将调用这个方法：

```java
testClass.doSomething(42);
```

然后，它变成了以下内容：

```java
testClass.doSomething(testClass, 42);
```

1.  一旦分配了内存，以及其他的维护工作，JVM 将调用适当的`constructor`方法。它没有返回类型，因为它没有变量可以返回结果。您必须用与它们所属的类相同的名字来命名它们。

构造函数分为两类：默认和非默认。只有一个**默认构造函数**，它没有参数。一个**非默认构造函数**是带有参数的。它受重载的影响，因此如果参数类型不同，可以有多个非默认构造函数。

Java 提供了调用另一个构造函数的能力。第一个被调用的构造函数由重载的规则决定。然后，被调用的构造函数可以调用另一个构造函数。这个调用必须是代码的第一行。注意可能的`递归构造函数调用`错误，其中构造函数 A 调用构造函数 B，而构造函数 B 又调用构造函数 A。

## 修改复利程序

现在，我们已经准备好回顾我们的复利程序，并将我们刚刚学到的内容应用到这个项目中的类中。

对于这一点，让我们更深入地看看我们在*第二章*，“代码、编译和执行”中讨论的`CompoundInterest04`程序。

我们将首先在`CompoundInterestCalculator04.java`文件中声明包。包，我们将源代码放入的文件夹，允许我们通过功能来管理我们的代码。您可能不想使用包的情况是如果您正在创建一个单文件源代码或 Linux shebang 应用程序。

这里是包声明。文件将位于名为`business`的文件夹中，该文件夹位于`com/kenfogel/compoundinterest04`文件夹中：

```java
package com.kenfogel.compoundinterest04.business;
```

这个程序将使用`NumberFormat`类。这个类是 Java 标准库的一部分，我们知道这一点是因为其包命名的第一个名字是`java`。为了使用这个类，我们必须将其导入到我们的文件中，如下所示：

```java
import java.text.NumberFormat;
```

类声明的第一行显示了以下内容：

+   这个类是公开的，因此可以在任何声明了对其引用的其他类中实例化。

+   公共类名`CompoundInterestCalculator04`也必须是文件名。你可以在一个文件中拥有多个类结构，但其中只有一个可以是公开的。

这是声明中的第一行：

```java
public class CompoundInterestCalculator04 {
```

这是类中的字段：

```java
    private final double principal = 100.0;
    private final double annualInterestRate = 0.05;
    private final double compoundPerTimeUnit = 12.0;
    private final double time = 5.0; // 
```

我们正在声明四个类型为`double`的实例变量。`private`的访问控制指定意味着你不能从任何可能引用这个类的其他类中访问这些变量。`final`修饰符将这些变量定义为不可变的。在一个类中，访问控制不适用，但修饰符适用。你必须在你声明`final`变量的地方或在构造函数中初始化它。

接下来，我们正在声明我们将要在代码中使用的对象引用：

```java
    private final NumberFormat currencyFormat;
    private final NumberFormat percentFormat;
```

我们正在声明`NumberFormat`类的两个实例。你可以从变量名中看出，我们为每个不同的格式规划了一个。这些是最终的，这意味着我们必须用值初始化它们，并且不能再次实例化。我们不仅可以在声明中实例化`NumberFormat`引用，还可以在构造函数中实例化它们，这就是我们将要做的。

以下方法是构造函数：

```java
    public CompoundInterestCalculator04() {
        currencyFormat = 
              NumberFormat.getCurrencyInstance();
        percentFormat = NumberFormat.getPercentInstance();
        percentFormat.setMinimumFractionDigits(0);
        percentFormat.setMaximumFractionDigits(5);
    }
```

构造函数很容易识别，因为它必须与类的名称相同。它不返回值，因为 JVM 将构造函数作为`new`操作的一部分调用。没有从`return`语句分配结果的内容。这是一个默认构造函数，因为括号中没有参数。一个类可能只有一个默认构造函数，但可以通过接受不同数据类型参数的构造函数来重载构造函数。

这个构造函数执行的任务是初始化和配置`NumberFormat`对象。我们不是仅仅使用`new`运算符，而是通过使用工厂方法来实例化这个类。工厂方法在调用`new`之前执行额外的任务。此外，请注意，我们通过类名而不是通过对象名调用方法，这与`Math`库方法非常相似。这告诉我们`getCurrencyInstance`和`getPercentInstance`是可用的静态方法。我们将在本节稍后讨论静态方法。

接下来是对象实例化后我们想要调用的方法：

```java
    public void perform() {
```

`perform`这个名字只是我选择的名字。重要的是要记住，除了构造函数之外，所有方法都应该使用动词。记住，类和变量标识符应该是名词。

方法调用的第一行调用`calculateCompoundInterest`方法进行计算，并将结果存储在`result`变量中：

```java
        var result = calculateCompoundInterest();
```

下一行显示的结果格式正确：

```java
        System.out.printf(
            "If you deposit %s in a savings account "
            + "that pays %s annual interest compounded "     
            + "monthly%nyou will have after %1.0f "
            + "years %s%n", 
            currencyFormat.format(principal),
            percentFormat.format(annualInterestRate),
            time, currencyFormat.format(result));
    }
```

代码中的加号符号表示连接。由于字符串相当长，它已经被拆分成多个由加号运算符连接的字符串。

在这里，我们看到执行答案计算的方法：

```java
    private double calculateCompoundInterest() {
        var result = principal * 
          Math.pow(1 + annualInterestRate / 
          compoundPerTimeUnit, time * compoundPerTimeUnit);
        return result;
    }
}
```

使用实例变量，将结果计算到一个名为`result`的变量中，该方法将其返回给调用者。这是一个私有方法，因此只有本类中的其他方法可以看到它。

复利示例中的第二个类只包含`main`方法。

再次，让我们声明这个文件所在的包：

```java
package com.kenfogel.compoundinterest04.app;
```

这个程序将使用我们编写的`CompoundInterestCalculator04`类。与所有导入一样，我们正在引用我们创建的包/文件夹中编写的类：

```java
import com.kenfogel.compoundinterest04.business
                            .CompoundInterestCalculator04;
```

这里是声明类的第一行：

```java
public class CompoundInterest04 {
```

它展示了以下内容：

+   这个类是公共的，因此可以在任何声明对其有引用的其他类中实例化。

+   类名`CompoundInterest04`也必须是文件名。

在一个文件中可以有多个类结构，但其中只有一个可以是公共的。

每个 Java 程序都必须有一个`main`方法。这是 JVM 开始执行你的程序的地方：

```java
    public static void main(String[] args) {
```

`main`方法是一个静态方法。静态方法与非静态方法的不同之处在于，我们可以调用这些静态方法而不需要实例化对象，就像我们使用`NumberFormat`的静态方法那样。`Math`库也是这样工作的。要使用`pow`函数，我们只需写`Math.pow`。我们不需要首先实例化`Math`对象。

在这里，我们使用名为`banker`的引用实例化`CompoundInterestCalculator04`类：

```java
        var banker = new CompoundInterestCalculator04();
```

我们在`main`方法中调用`CompoundInterestCalculator04`类中的`perform`方法结束：

```java
        banker.perform();
    }
}
```

我们现在已经回顾了`CompoundInterest04`程序是如何构建的，以及我们如何利用访问控制和包。

## 基于功能的功能类组织

我们从本书开始使用的复利程序正确地执行了其特定任务。然而，这种方法的问题在于程序是一个死胡同——这并不意味着死胡同一定是坏事。有时，你只是想要一个可以确定特定问题答案的*一次性*程序。

但如果我们编写一个复杂的程序呢？在现实世界中，意味着在工作中编码，你很少会编写像我们的复利计算器这样的程序。想象一下，你想要创建一个更完善的银行系统。在这个系统中，将需要从用户那里获取输入，而不是在程序的源代码中硬编码它。更进一步，你可能希望将输入和结果存储在外部存储中，如数据库。你可能还希望根据存储的数据生成报告。

现在我们根据功能重新组织`CompoundInterest04`程序，现在重命名为`CompoundInterest05`。

### 数据类

第一步是设计一个只持有数据的类。这个类将没有领域方法，例如计算、存储在数据库中或与最终用户进行输入输出方法交互。我们正在创建一个新的数据类型，我们可以在执行其他动作的类中使用它。这种类型的类遵循一个最初被描述为**JavaBean**的模式。Java 将其引入作为一种可重用的软件组件。

我们创建的不是一个纯 JavaBean，而是一个变体。我经常将这种类型的类称为一个简单的变量盒子。让我们看看我们的复利问题中的一个例子。

我们从`package`语句开始。我们将使用包名，任何需要使用这个包的类都可以通过导入名称来使用，如下所示：

```java
package com.kenfogel.compoundinterest05.data;
```

这里是标准的公共类声明：

```java
public class CompoundInterestData {
```

这里是执行计算所需的四个变量：

```java
    private final double principal;
    private final double annualInterestRate;
    private final double compoundPerTimeUnit;
    private final double time;
```

我们在这里声明了四个实例变量。它们是 final 的，所以一旦赋值，就变为不可变。我们预计这些值将来自程序的用户，而不是像我们之前那样硬编码值。这意味着每次新的计算都需要一个新的`CompoundInterestData`对象。

最后这个变量是我们计划存储计算结果的地方：

```java
    private double result;
```

由于这个类中没有动作，我们无法确定这个值何时会被设置，因此它不能是 final 的。

这是构造函数：

```java
    public CompoundInterestData(double principal, 
               double annualInterestRate, 
               double compoundPerTimeUnit, 
               double time) {
        this.principal = principal;
        this.annualInterestRate = annualInterestRate;
        this.compoundPerTimeUnit = compoundPerTimeUnit;
        this.time = time;
    }
```

当这个类被实例化时，它有四个必需的参数。一旦赋值给类变量，你不能改变它们所持有的值。注意`this`关键字。由于我们使用了与方法参数相同的实例变量名，我们使用`this`来指定实例变量。记住`this`是类中实例变量的地址。对于类或静态变量没有`this`引用，因为每个类只有一个。不使用`this`，你将引用同名的参数。`result`实例变量不是参数之一，因为你在程序中稍后计算它的值。

接下来的四个方法是为四个类实例变量提供的 getter：

```java
    public double getPrincipal() {
        return principal;
    }
    public double getAnnualInterestRate() {
        return annualInterestRate;
    }
    public double getCompoundPerTimeUnit() {
        return compoundPerTimeUnit;
    }
    public double getTime() {
        return time;
    }
```

这个类基于的 JavaBean 规范要求所有实例变量必须是私有的。规范接着定义了 setter 和 getter 方法。由于这前四个变量通过`final`属性是不可变的，因此你只能有一个 getter。当你使用`new`创建对象时，你将提供初始值。

这最后两种方法很特殊，因为`result`变量不是最终的：

```java
    public double getResult() {
        return result;
    }
    public void setResult(double result) {
        this.result = result;
    }
}
```

我们在实例化这个类之后确定值。我们在构造函数中为四个输入值赋值。Java 持久化 API 或 Jakarta 等框架期望这种 getter 和 setter 语法。

### 业务类

现在，让我们编写`calculation`类。它的唯一目的将是计算结果并将其存储在 bean 中。我们正在导入我们刚刚创建的数据类：

```java
package com.kenfogel.compoundinterest05.business;
import com.kenfogel.compoundinterest05.data.
                                    CompoundInterestData05;
public class CompoundInterestCalculator05 {
```

这个类只有一个方法，没有实例变量。由于所有值都在`CompoundInterestData05`中，我们通过调用属性的`getter`方法来检索这些值。最后，我们通过调用唯一的 setter 将结果分配给 bean 的结果变量：

```java
    public void calculateCompoundInterest(
                    CompoundInterestData05 value) {
        var result = value.getPrincipal() * 
               Math.pow(1 + value.getAnnualInterestRate() / 
               value.getCompoundPerTimeUnit(),
               value.getTime() *
               value.getCompoundPerTimeUnit());
        value.setResult(result);
    }
}
```

### 用户界面类

最后一个组件是用户界面，在这里我们可以向用户请求执行计算所需的四条信息。这就是我们将创建此对象的地方：

```java
package com.kenfogel.compoundinterest05.ui;
```

在`package`语句之后，我们有我们将使用的类和库的导入。我们有一个新的导入，那就是`Scanner`库类。`Scanner`类的对象允许我们在控制台应用程序中收集最终用户输入，例如从键盘输入：

```java
import com.kenfogel.compoundinterest05.business.
                         CompoundInterestCalculator05;
import com.kenfogel.compoundinterest05.data.
                         CompoundInterestData05;
import java.text.NumberFormat;
import java.util.Scanner;
public class CompoundInterestUI05 {
    private CompoundInterestData05 inputData;
    private final CompoundInterestCalculator05 calculator;
    private final Scanner sc;
    private final NumberFormat currencyFormat;
    private final NumberFormat percentFormat;
```

在 Java 中，没有关于方法顺序或字段位置的规则，这与 C 和 C++不同，在 C 和 C++中，顺序可能很重要。我个人的风格，正如你在这些示例中看到的，是将实例变量放在第一位，然后将构造函数紧随其后。关于这个问题的建议是，与你的团队就编码风格达成一致。这将使阅读彼此的代码变得容易得多。

这里是实例化`NumberFormat`对象和`Scanner`类的构造函数。你必须向`Scanner`类的构造函数提供输入源。它可以是来自磁盘上的文件，但在这个程序中，它来自键盘。我们调用与键盘交互的对象的`System.in`：

```java
    public CompoundInterestUI05() {
        currencyFormat = 
                NumberFormat.getCurrencyInstance();
        percentFormat = NumberFormat.getPercentInstance();
        percentFormat.setMinimumFractionDigits(0);
        percentFormat.setMaximumFractionDigits(5);

        sc = new Scanner(System.in);
        calculator = new CompoundInterestCalculator05();
    }
```

接下来是这个用户界面类的入口点。这将是这个类中唯一的公共方法。我使用`do`前缀，因为它确保名称是一个动词或动作。我们必须从用户请求的四个值作为局部或方法变量存在。我们将用户输入的结果分配给每一个。在这个表达式中，我们使用四个局部变量作为参数实例化数据对象。`new`运算符通过构造函数将值从局部变量复制到`CompoundInterestData05`对象的实例变量中。然后我们调用`Calculator`类中的`calculateCompoundInterest`来计算结果。最后一步是显示结果：

```java
    public void doUserInterface() {
        doUserInstructions();
        var principal = doPrincipalInput();
        var annualInterestRate = doAnnualInterestRate();
        var compoundPerTimeUnit = doCompoundPerTimeUnit();
        var time = doTimeInput();
        inputData = new CompoundInterestData05(
              principal, 
              annualInterestRate, 
              compoundPerTimeUnit, 
              time);
        calculator.calculateCompoundInterest(inputData);
        displayTheResults();
    }
```

这个版本的`CompoundInterest`程序遵循经典的输入、处理和输出模式。我们现在遇到的第一个方法是`output`：

```java
    private void displayTheResults() {
        System.out.printf(
         "If you deposit %s in a savings account that pays"
         + " %s annual interest compounded monthly%n"
         + "you will have after %1.0f years %s%n", 
         currencyFormat.format(inputData.getPrincipal()),
            percentFormat.format(
                inputData.getAnnualInterestRate()),
                inputData.getTime(), 
                currencyFormat.format(
                    inputData.getResult()));
    }
```

下一个方法是输入过程的一部分。它可以在屏幕上向最终用户提供额外的说明，但我在这里保持了简单：

```java
    private void doUserInstructions() {
        System.out.printf(
                 "Compound Interest Calculator%n%n");
    }
```

现在，我们来到用户输入部分。每个输入都会显示一个提示并等待输入。一旦你输入字符串，`nextDouble`方法就会尝试将其转换为适当的类型——在这种情况下，是`double`：

```java
    private double doPrincipalInput() {
        System.out.printf("Enter the principal: ");
        var value = sc.nextDouble();
        return value;
    }
    private double doAnnualInterestRate() {
        System.out.printf("Enter the interest rate: ");
        var value = sc.nextDouble();
        return value;
    }
    private double doCompoundPerTimeUnit() {
        System.out.printf("Enter periods per year: ");
        var value = sc.nextDouble();
        return value;
    }
    private double doTimeInput() {
        System.out.printf("Enter the years: ");
        var value = sc.nextDouble();
        return value;
    }
}
```

但是等等——四个输入方法中有一个严重的问题。除了`String`提示外，它们完全相同。作为一名教师，我把重复的代码描述为失败的邀请。如果我们决定进行更改，比如从`double`切换到`float`，我们必须记住在八个不同的地方进行四次更改。不小心遗漏其中一个更改的可能性太高。让我们将这四个方法合并为一个。

很简单——只需将提示作为单个输入方法的参数，如下所示：

```java
    private double doUserInput(String prompt) {
        System.out.printf(prompt);
        var value = sc.nextDouble();
        return value;
    }
```

现在，我们可以使用`doUserInput`方法来处理所有四个用户输入：

```java
    public void doUserInterface() {
        doUserInstructions();
        var principal = 
            doUserInput("Enter the principal: ");
        var annualInterestRate = 
            doUserInput("Enter the interest rate: ");
        var compoundPerTimeUnit = 
            doUserInput("Enter periods per year: ");
        var time = doUserInput("Enter the years: ");
        inputData = new CompoundInterestData05(
              principal, 
              annualInterestRate, 
              compoundPerTimeUnit, 
              time);
        calculator.calculateCompoundInterest(inputData);
        displayTheResults();
    }
```

所有用户输入都是`String`对象；你不能输入纯数字、布尔值或字符。`Scanner`类负责将字符串转换为目的地类型，如`next`方法所表达的那样。

在我们的例子中，它们都是`double`类型。如果我们输入`bob`字符串而不是数字会发生什么？Java 会抛出异常。这是一个错误条件。当我们研究循环时，我们将学习如何创建防用户输入错误，当我们研究 GUI 编程时，我们将了解其他管理用户输入的方法。在所有情况下，所有输入都作为字符串到达——如前所述。

最后一个类是`app`类。我们通常使用`app`这个名称来定义一个包含包含`main`方法的类的包。这是一个约定，你可以自由地更改它：

```java
package com.kenfogel.compoundinterest05.app;
import com.kenfogel.compoundinterest05.ui
                             .CompoundInterestUI05;
public class CompoundInterest05 {
    public static void main(String[] args) {
        var calculator = new CompoundInterestUI05();
        calculator.doUserInterface();
    }
}
```

当我们运行这个新版本时，这将是我们得到的输出：

```java
Welcome to the Compound Interest Calculator
Enter the principal: 5000
Enter the interest rate: 0.05
Enter periods per year: 12
Enter the years: 5
If you deposit $5,000.00 in a savings account that pays 5% annual interest compounded monthly
you will have after 5 years $6,416.79
```

程序会要求用户输入它所需的四个值，然后使用这些值计算结果并显示。

# 摘要

在本章中，我们探讨了类的基本组成部分。一旦我们回顾了如何组装`CompoundInterest04`示例，我们就将程序拆分，创建了用于存储数据、显示用户界面和计算结果的类。我们还了解了构造函数和已弃用的`finalize`方法。我们了解了`new`的作用以及 JVM 如何管理程序的内存。

第二个版本，`CompoundInterest05`，展示了如何根据功能专业地组织程序。它将数据、用户界面和操作（通常称为业务）分开。为了收集用户输入，我们首次了解了 Java 库中的`Scanner`类。你现在应该对 Java 类的组织结构以及如何控制类成员的访问有很好的理解。

在下一章中，我们将更深入地探讨执行类操作的策略以及我们如何管理类之间的关系。

# 进一步阅读

+   *Java 中的栈内存和堆空间*：[`www.baeldung.com/java-stack-heap`](https://www.baeldung.com/java-stack-heap)
