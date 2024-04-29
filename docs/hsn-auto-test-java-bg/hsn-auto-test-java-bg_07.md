# 在 Java 11 中理解 Date 类和构造函数

`Date`类和构造函数是 Java 的重要部分。在本章中，我们将通过一些例子详细讨论这些内容。

在本章中，我们将涵盖：

+   日期类

+   日历类

+   构造函数

+   参数化构造函数

# 日期类

为了理解`Date`类的概念，我们将从创建我们的`dateDemo`类的源代码开始。假设我们想要打印当前日期或当前时间。我们该如何打印呢？

有时，我们被要求输入日期到当前日期字段中，我们需要从 Java 中获取它。在这种情况下，我们将使用`Date`类，它将给我们当前日期和当前时间，以及秒数。因此，关于日期、星期、月份、年份或小时的每个细节都可以通过 Java 类来读取。Java 开发了一个叫做`Date`的类，我们可以从中获取所有这些细节。以下截图显示了源代码：

![](img/2127fc87-0ea8-4136-89ad-c91a2b6ad5d1.png)

显示使用日期类的源代码

基本上，我们需要使用那个特定类中存在的方法。要使用该类中存在的方法，我们需要创建该特定类的对象。为此，让我们考虑以下代码语法：

```java
Date d= new Date();
```

这个`Date`类来自`util`包，`d`是`Date`类的对象，其中包含日期和时间。在前一章中，我们看到 Java 有一些包，比如`java.lang`包，其中包含了所有基本的 Java 东西，还有`java.util`，其中包含了集合框架和`Date`类。

前面的代码语法表明我们不知道`Date`类在哪里。为了使这个类在我们的 Java 文件中可用，我们需要导入`util` Java 包，因为这个`Date`类被打包到那个特定的包中。如果我们在前面的类中使用它来导入这个包，你就可以成功地使用那个日期。将鼠标移到这里，它会显示`import 'Date' (java.util)`，如下面的截图所示：

![](img/6315b3ed-ec0c-4ee5-8394-852d79fbf046.png)

快速修复下拉菜单，提供纠正代码错误的建议

一旦你点击那个，你会看到：

```java
import java.util.Date
```

其中`util`是包，`Date`是一个类。

正如我们所见，`d`是包含日期和时间的对象，但我们如何打印它呢？因为它是一个对象格式，我们不能简单地使用以下内容：

```java
System.out.println(d)
```

要将其转换为可读文本，请参考以下截图：

![](img/abcf8219-d429-4217-9600-392a79c407a1.png)

将代码转换为可读文本格式

在这里，我们将`Date`转换为字符串，以便我们可以在输出中直观地看到它。在运行前面的代码时，如截图所示，它打印出以下内容：

```java
Fri Apr 15 17:37:27 EDT 2016
```

这就是我们如何从我们当前系统的 Java 日期中打印整个日期、时间和月份。前面输出的格式不是我们通常得到的，但它可能是特定的格式，比如：

```java
mm//dd//yyyy
```

如果我们想以前面的格式提取我们的日期，我们该如何做？

`d`对象给我们所有的细节。但我们如何将所有这些细节转换为前面的格式呢？为此，我们将使用以下内容：

```java
       Date d= new Date();

        SimpleDateFormat sdf=new SimpleDateFormat("M/d/yyyy");
        System.out.println(sdf.format(d));
        System.out.println(d.toString());
```

前面的代码语法的输出将是：

![](img/464c47f3-8bda-4d73-8a42-83e235e38004.png)

按照代码显示日期和时间的输出

请参考以下 URL 获取`SimpleDateFormat`格式代码：

+   [`www.tutorialspoint.com/java/java_date_time.htm`](http://www.tutorialspoint.com/java/java_date_time.htm)

现在，当改变对象和`SimpleDateFormat`代码时，我们看到以下内容：

```java
 Date d= new Date();

        SimpleDateFormat sdf=new SimpleDateFormat("M/d/yyyy");
        SimpleDateFormat sdf=new SimpleDateFormat("M/d/yyyy hh:mm:ss");
        System.out.println(sdf.format(d));
        System.out.println(sd.toString());
        System.out.println(d.toString());
```

输出将是：

![](img/eed47275-9213-438e-8db8-6ee259e99d2f.png)

以新格式显示日期和时间的输出

因此，我们实际上可以根据我们的需求格式化我们的日期，并将其传递到`SimpleDateFormat`方法中。我们可以将`d`对象带入并放入一个参数中，这样它将以特定的方式格式化。这就是使用 Java 检索日期的方法。

在下一节中，我们将看到如何使用`Calendar`类。

# 日历类

在前一节中，我们探讨了`Date`类，学习了`Date`方法以及如何使用简单的日期格式标准对它们进行操作。在本节中，我们将学习`Calendar`类，它类似于`Date`类，但具有一些额外的功能。让我们看看它们是什么，以及我们如何使用它们来提取我们的日期格式。

首先，我们将创建一个不同名称的类以避免冲突。要创建一个`Calendar`实例，请运行以下命令：

```java
Calendar cal=Calendar.getInstance();
Date d=new Date();
```

这些步骤与`Date`类的步骤相似。但是，`Calendar`对象具有一些`Date`不支持的独特功能。让我们来探索一下。

使用以下代码片段：

```java
        Calendar cal=Calendar.getInstance();
        SimpleDateFormat sd=new SimpleDateFormat("M/d/yyyy hh:mm:ss");
        System.out.println(sd.format(cal.getTime()));
```

前面代码的输出将是：

![](img/5f8f48c1-689c-4c56-b27a-debebf8a514f.png)

使用日历类显示日期和时间的输出

现在，假设我们想要打印月份和星期几。我们将在前面的代码片段中添加以下代码行：

```java
System.out.println(cal.get(Calendar.DAY_OF_MONTH));
System.out.println(cal.get(Calendar.DAY_OF_WEEK_IN_MONTH));
```

输出将如下所示：

![](img/e36a6cb9-d914-4aab-9863-b121a07af33a.png)

使用日历类显示日期、时间、月份的日期和星期几的输出

同样，我们可以从以下屏幕截图中看到有多个属性可供选择：

![](img/675cc76c-07cd-4a88-b00f-c26c75ed89e1.png)

下拉菜单显示日历类的多个属性

因此，在这里我们使用了`Calendar`实例来实际获取系统日期和时间，但在之前的类中我们使用了`Date`实例；这是唯一的区别。在这个`Calendar`实例中存在很多方法，你在`Date`类中找不到。

这就是根据我们的要求检索系统日期的方法。

# 构造函数

构造函数是 Java 编程语言中最重要的概念之一。因此，在看一个例子之前，让我们先了解一下构造函数是什么。

构造函数在创建对象时执行一块代码。这意味着，每当我们为类创建一个对象时，自动执行一块代码。换句话说，每当创建对象时，都会调用构造函数。

那么构造函数在哪里使用，我们如何定义它呢？应该编写一个构造函数，就像一个方法一样，但构造函数和方法之间的唯一区别是构造函数不会返回任何值，构造函数的名称应该始终是类名。

要为这个类创建一个构造函数，我们将编写以下代码语法：

```java
public class constructDemo()
{
//
}
```

从前面的代码语法可以看出，无论在这个构造函数中写了什么，只要创建对象并调用构造函数，这个块中的一组行就会被执行。这就是构造函数的主要目的：

```java
package coreJava;

public class constructDemo {
    public constructDemo()
    {
        System.out.println("I am in the constructor");
    }
    public-void getdata()
    {
        System.out.println("I am the method");
    }
    // will not return value
    //name of constructor should be the class name
    public static void main(String[] args)  {
        // TODO Auto-generated method stub
        constructDemo cd= new constructDemo(); 
```

每当执行前面的代码时，控制将自动检查是否有显式定义的构造函数。如果定义了，它将执行特定的块。在 Java 中，每当创建一个对象时，都会调用构造函数。

前面代码的输出将是：

```java
I am in the constructor
```

我们实际上并没有为每个类创建构造函数，但是现在我们特别引入了构造函数的概念，因为在之前，我们在定义构造函数时没有使用任何概念。现在，如果我们使用这个命令，程序仍然会运行，但这次它不会执行那个块。如果我们不定义任何构造函数，编译器将调用默认构造函数。我们可以称之为隐式构造函数。

我们在实时中大多依赖构造函数来初始化对象，或为我们的程序定义变量。构造函数和普通方法看起来很相似，因为它们在括号中定义了访问修饰符，但不接受任何返回类型，但在这种情况下它接受。因此，如果我们写：

```java
public constructDemo()
{
    System.out.println("I am in the constructor");
    System.out.println("I am in the constructor lecture 1");

}
```

前面代码的输出将是：

```java
I am in the constructor
I am in the constructor lecture 1
```

因此，通常人们使用上述代码块来在实时中定义变量或初始化属性，并继续使用构造函数。

在下一节中，我们将看一下 Java 中另一个构造函数。

# 参数化构造函数

我们在上一节学习的构造函数是默认构造函数，因为它不接受任何值。在具有相同语法的参数化构造函数中，我们实际上提供了一些参数，如下面的截图所示：

![](img/c78cb8ee-e3e5-48fb-a444-d6050418e31a.png)

使用给定代码的参数化构造函数的输出

前一个构造函数和这个的唯一区别是这里我们传递了参数，在默认的情况下不传递任何参数。当我们运行我们的代码时，每当我们创建一个对象，如果我们不传递任何参数，编译器会自动选择默认构造函数，如下面的截图所示：

![](img/29799d03-478d-4528-a35b-42f7eeac0ea7.png)

当传递默认参数时的输出

现在，让我们为同一个类创建另一个带参数的对象：

```java
constructDemo c=new constructDemo(4,5);
```

当我们按照上述语法定义参数时，编译器在执行运行时时检查是否有两个整数类型参数的构造函数。如果找到构造函数，它将执行以下代码语法：

```java
public constructDemo(int a, int b)
{
    System.out.println("I am in the parameterized constructor");
}
```

在未定义参数的情况下，编译器执行默认构造函数。上述代码的输出将是：

```java
 I am in the parameterized constructor
```

在运行时，创建对象时，我们必须给出参数，因此在执行过程中，它将与定义的构造函数进行比较。同样，我们可以为同一个类创建多个对象：

```java
constructDemo cd=new constructDemo();
constructDemo c=new constructDemo(4,5);
```

当两个构造函数一起运行时，输出将是：

```java
I am in the constructor
I am in the constructor lecture 1
I am in the parameterized constructor
```

现在，我们将创建另一个类似类型的构造函数，但这次只有一个参数：

```java
public constructDemo(String str)
{
    System.out.println(str);
}
public static void main(String[] args) 
{
    constructDemo cd=new constructDemo("hello");
}
```

输出将是：

```java
hello
```

因此，如果我们明确定义了某些内容，Java 编译器会优先选择显式构造函数，否则会打印隐式构造函数。这里需要注意的关键点是它不会返回任何值，并且构造函数必须仅用类名定义。

# 总结

在本章中，我们运行了一些代码示例，以了解`Date`类、`Calendar`类和构造函数的工作原理。

在本章中，我们将介绍三个关键字：`super`，`this`和讨论`finally`块。
