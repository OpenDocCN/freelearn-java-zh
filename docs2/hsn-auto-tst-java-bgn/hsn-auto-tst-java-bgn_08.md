# Java 中 super 和 this 关键字以及异常的重要性

在本章中，我们将探讨两个关键字：`super`和`this`。我们将挑选示例并解释在编写 Java 代码时如何在各种情况下使用它们。我们还将探讨异常以及如何使用它们来处理由于某些错误而导致代码失败的情况。我们将以关于`finally`块的章节结束本章。

在本章中，我们将涵盖以下主题：

+   super 关键字

+   super 关键字的实际用法

+   this 关键字的重要性

+   不同类型的异常

+   try...catch 机制用于处理异常

+   Java 中 finally 块的重要性

# super 关键字

通常情况下，当人们从不同的类继承属性时，如果在父类和子类中都使用了相同的变量名，可能会出现冗余。为了区分父变量和子变量，我们使用`super`关键字。

让我们用一个例子来解释这一点。让我们创建两个类，分别命名为`childDemo`和`parentDemo`。在`parentDemo`类中，我们定义一个名为`name`的字符串，并将其赋值为`'rahul'`字符串。

现在，在`childDemo`类中，我们继承了`parentDemo`类的属性。我们知道如何使用`extends`关键字继承父类的属性，这是我们在第五章“关于接口和继承你需要知道的一切”中学到的。继承属性的代码如下：

```java
public class childDemo extend parentDemo{
```

在此代码中，`childDemo`正在继承`parentDemo`的属性。

在`childDemo`类中添加一个字符串，命名为`name`，并将其赋值为`QAClickAcademy`字符串。然后我们在`childDemo`类内部定义一个名为`public void getStringdata()`的方法，并给出一个打印`name`值的输出语句。我们还在`getStringdata()`方法外部定义另一个方法，命名为`public static void main(String[] args)`，并创建子类`childDemo`的对象，`childDemo cd = new childDemo();`。一旦创建了对象，我们就在其下方添加另一行代码：`cd.getStringdata();`。这调用了`getrStringdata()`方法，所以显然输出的是`name`的值，即`QAClickAcademy`。尽管我们继承了`parentDemo`类的属性，该类也包含一个同名的字符串，但打印语句调用的是`childDemo`中的字符串值。这是因为 Java 优先考虑局部变量。

当父类和子类的变量名发生冲突时，它优先考虑局部变量，在这种情况下是`childDemo`类。如果我们正在处理一个需要同时在`parentDemo`类中打印字符串`name`的项目，该怎么办？为此，我们使用`super`关键字来引用`parentDemo`类，从该类继承属性到`childDemo`类。因此，如果我们想从`parentDemo`类中调用`name`变量，我们添加一个打印语句，并在我们想要打印的变量前添加一个`super`关键字——这将获取`parentDemo`的值。当我们现在运行代码时，我们得到父对象和子对象作为输出，因为我们已经在两个类中都留下了打印字符串`name`的语句。`parentDemo`类的代码如下：

```java
public class parentDemo{
    String name= "Rahul";
    public static viod main (String[] args){
    }
}
```

`childDemo`类的代码如下：

```java
public class childDemo extends parentDemo{

    String name = "QAClickAcademy";
    public void getStringdata();
    {
        System.out.println(name);
        System.out.println(super.name);
    }

    public static void main(String[] args){
    childDemo cd = new childDemo();
    cd.getStringdata();
    }
}
```

最终输出将是：

```java
QAClickAcademy
Rahul
```

# super 关键字的实际用法

在本节中，我们将探讨在 Java 中使用`super`关键字的多种方式。

# 使用方法中的 super 关键字

我们看到了如何借助`super`关键字处理父变量。在本节中，我们将看到如何处理在`parentDemo`类和`childDemo`类中名称相同的两个方法。我们也将在本节中使用之前的示例。

在`parentDemo`类中，添加一个名为`getData()`的方法，并在方法内部添加一个打印语句以显示`"I am in parent class"`信息。如果我们想在`childDemo`类中执行`getData()`方法，我们在`childDemo`类的`main`方法中写`cd.getData()`。我们可以像继承`parentDemo`类的属性一样访问`getData()`。如果我们运行`childDemo`类，我们将收到之前示例的输出以及我们在`parentDemo`类中添加的新句子，即`I am in parent class`。

在`childDemo`类中，我们将定义另一个与`parentDemo`类同名的方法，并添加一个打印语句以显示`I am in child class`信息。如果我们运行`childDemo`类，我们将得到之前示例的输出，然后显示`I am in child class`。这是因为优先考虑局部类，所以`childDemo`类中的`getData()`方法覆盖了`parentDemo`类中的`getData()`方法。

现在，我们想在`childDemo`类中使用`parentDemo`类的`getData()`方法。为此，我们只需像处理变量那样做：在`childDemo`类的`getData()`方法中添加`super.getData()`。当我们运行`childDemo()`类时，我们将得到之前示例的输出，然后是`I am in parent class`，然后是`I am in child class`。

# 使用构造函数中的 super 关键字

让我们在本节中使用`super`关键字来为构造函数。我们也将使用之前的示例。

在 `parentDemo` 类中，我们定义了一个构造函数，`parentDemo()`，并添加了一个打印语句来打印：`父类构造函数`。

在 `childDemo` 中，我们定义了一个构造函数 `childDemo()` 并添加了一个打印语句来打印：`子类构造函数`。如果我们想在 `childDemo` 类中使用 `parentDemo` 类的构造函数，我们在 `childDemo()` 构造函数中添加 `super()` 方法。这使得控制器调用 `parentDemo` 类中的构造函数。

在使用构造函数时，我们需要遵循的一个重要规则是：在子构造函数中，每次使用 `super` 构造函数时，它都应该始终是其第一行。

当我们运行 `childDemo` 类时，控制器首先执行 `super()` 方法。它进入 `parentDemo()` 构造函数并执行它，然后是 `childDemo()`。所以最终的输出将是：

```java
Parent class constructor
Child class constructor
QAClickAcademy
Rahul
I am parent class
I am in child class
```

# `this` 关键字的重要性

在 Java 中还有一个与 `super` 关键字类似的关键字：`this`。在本节中，我们将探讨 `this` 关键字。

让我们用一个例子来解释 `this` 关键字。创建一个名为 `thisDemo` 的类，并声明一个变量 `a`，将其值设置为 `2`。我们在其类中定义一个 `getData()` 方法，在其中声明 `a` 变量，并将其值设置为 `3`。我们还在其中添加了一个打印语句。代码将如下所示：

```java
package coreJava;public class thisDemo
{
    int a= 2;
    public void getData()
    {
        int a= 3;
        System.out.println(a);
    }
```

如我们所见，`a` 的值在整个类中都是 `2`，但在一个特定的方法 `getData()` 中，我们希望变量的值是 `3`。在这个代码中，我们想要调用 `a` 的两个值，即 `2` 和 `3`。我们在主方法中创建了一个对象，并将 `td` 对象添加到其中。`td` 对象的代码如下：

```java
thisDemo td=new thisDemo();
td.getData();
```

如果我们运行代码，我们得到的输出是 `3`。但我们也想在同一个块中打印 `a` 的值为 `2`。这就是 `this` 关键字发挥作用的时候。类对象的范围将在类级别，而不是方法级别。因此，我们说 `getData()` 方法指的是当前对象，对象的作用域位于类级别。因此，`a=2` 对整个类有效，而 `a=3` 只对 `getData()` 方法有效。这就是为什么我们在 `getData()` 方法中调用 `a` 变量时称其为局部变量，而在类中的 `a` 变量称为全局变量。

为了打印我们正在工作的示例的全局变量，我们需要在 `getData()` 方法中添加一个打印语句，并在打印语句中添加 `this.a`。打印语句将如下所示：

```java
System.out.println(this.a);
```

当我们运行代码时，我们得到以下输出：

```java
3
2
```

这就结束了我们对 `this` 变量的示例。现在让我们学习异常。

# 不同类型的异常

在本节中，我们将探讨如何在 Java 中处理异常。

通常，如果代码中存在错误，我们需要捕获它并打印一条消息而不失败；这可以通过使用`try...catch`机制来完成。所以通常，当我们尝试编写代码并且怀疑其中可能存在错误时，我们将使用该错误进行异常处理。

我们将通过一个练习来解释它。让我们创建一个新的类，`exceptionDemo`，并在`main`块内部声明变量`a`、`b`和`c`，并分别将`4`、`7`和`0`的值赋给它们。我们在`main`块内部添加一个`try`块，并声明一个整数变量`k`，它等于`b`除以`c`。每次我们在`try`块中添加任何内容时，我们都在尝试查看代码是否工作。如果它失败，控制器将退出这个`try`块并进入包含异常的`catch`块。一个需要记住的重要点是`catch`块紧随`try`块之后。在`catch`块内部，我们写入一个打印消息来显示`我捕获了错误/异常`。

当控制器进入`k`变量行时，脚本会失败，因为`7/0`是无穷大，这是一个算术异常，但脚本不会立即失败。如果我们不写`try...catch`块，我们会看到不同类型的错误。

让我们移除`try...catch`块，运行代码，并查看我们得到的错误。我们在输出部分看到一个错误，`Java.lang.ArithmeticException`；这是因为我们不能将`7`除以`0`，所以脚本会突然失败。

如果我们一开始觉得我们的代码会出现错误，我们可以简单地编写一个脚本来传递和捕获错误，通过放置一个可以由`try...catch`机制处理的适当调试信息。现在，让我们再次添加`try...catch`块并调试整个代码。输出将会是`我捕获了错误/异常`；这是因为`7`除以`0`是无穷大，所以这里的脚本应该失败，但我们没有在输出部分看到代码失败的任何错误。这是因为控制器简单地移动到`catch`块并执行它。最终的代码将如下所示：

```java
public static void main(String[] args)
{
    int b=7; 
    int c=0;
    try
    {
        int k=b/c;
        System.out.println(k);
    }
    catch(Exception e)
    {
        System.out.println("I caught the error/exception")
    }
}
```

输出将如下所示：

```java
I caught the error/exception
```

# 处理异常的`try...catch`机制

在本节中，我们将使用一个`try`后跟多个`catch`块。Java 中有不同类型的异常，并且对于每个异常，我们都可以添加单独的`catch`块。

让我们用之前的例子来解释这一点。为之前的代码编写的异常是一个通用异常，所以对于`try`块中的任何错误，都会执行通用异常。现在让我们尝试捕获一个特定的异常。我们可以在`try`块下面添加一个`catch`块，并添加一个特定的异常和一个打印语句来打印`我捕获了 Arithmeticerror/异常`。特定`catch`块的代码如下：

```java
catch(arithmeticException et)
{
    System.out.println("I caught the Arithmeticerror/exception");
}
```

当我们运行代码时，我们得到以下输出：

```java
I caught the Arithmeticerror/exception
```

我们看到，当我们运行代码时，控制器进入了`catch`块，因为`catch`块是专门为算术异常编写的，而抛出的错误也属于算术错误。所以一旦控制器收到一个错误，`try`块将看到哪个`catch`块与之相关，并运行它。

Java 中还有许多其他的异常：我们只需在 Google 上搜索并查看它们。

# Java 中`finally`块的重要性

还有一个与`try...catch`块类似的块：就是`finally`块。无论是否抛出异常，`finally`块都将被执行。如果程序运行成功，该块将被执行，即使程序没有运行。

我们将使用我们在*处理异常的 try...catch 机制*部分使用的示例来解释这一点。我们只是在`catch`块之后添加一个`finally`块，并在其中给出一个打印语句，说“删除 cookies”。代码块将看起来像这样：

```java
finally
{
    System.out.println("delete cookie")
}
```

当我们运行代码时，我们得到以下输出：

```java
I caught the Arithmeticerror/exception
delete cookie
```

一个重要的点是，`finally`可以与或不与`catch`块一起工作；它只需要写在`try`块下面。

# 摘要

在本章中，我们探讨了`super`和`this`关键字。我们还通过示例来解释我们可以在哪些情况下使用这些关键字来克服某些障碍。我们学习了异常，并在代码因错误而失败的各种实例中实现了它们。我们还学习了`finally`块。

在下一章中，我们将深入探讨集合框架，它由接口和类组成。我们还将查看三个主要的集合：`List`、`Set`和`Map`。
