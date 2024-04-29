# Java 中 super 和 this 关键字以及异常的重要性

在本章中，我们将看看两个关键字：`super` 和 `this`。我们将举例并解释它们在编写 Java 代码时在各种情况下的使用方式。我们还将看看异常以及如何使用它们来处理代码由于某些错误而失败的情况。我们将以 `finally` 块的部分结束本章。

在本章中，我们将涵盖以下主题：

+   super 关键字

+   super 关键字的实际用法

+   this 关键字的重要性

+   不同类型的异常

+   try...catch 机制用于处理异常

+   Java 中 finally 块的重要性

# super 关键字

通常，当人们从不同的类中继承属性时，如果父类和子类中使用相同的变量名，可能会出现冗余。为了区分父变量和子变量，我们使用 `super` 关键字。

让我们用一个例子来解释这个问题。我们创建两个类，分别命名为 `childDemo` 和 `parentDemo`。在 `parentDemo` 类中，我们定义一个名为 `name` 的字符串，并将字符串 `'rahul'` 赋给它。

现在，在 `childDemo` 类中，我们继承了 `parentDemo` 的属性。我们知道如何使用 `extends` 关键字继承父类的属性，这是我们在第五章中学到的，*关于接口和继承的一切*。继承属性的代码如下所示：

```java
public class childDemo extend parentDemo{
```

在这段代码中，`childDemo` 继承了 `parentDemo` 的属性。

在 `childDemo` 类中添加一个字符串，称为 `name`，并将字符串 `QAClickAcademy` 赋给它。然后我们在 `childDemo` 类内定义一个名为 `public void getStringdata()` 的方法，并给出一个语句来打印 `name` 的值作为输出。我们在 `getStringdata()` 外定义另一个方法，称为 `public static void main(String[] args)`，并为子类创建一个对象，`childDemo cd = new childDemo();`。一旦对象被创建，我们在其下面添加另一行代码：`cd.getStringdata();`。这将调用 `getrStringdata()` 方法，因此显然名称将作为输出打印，即 `QAClickAcademy`。尽管我们继承了 `parentDemo` 类的属性，该类也包含一个同名的字符串，但打印语句调用的是 `childDemo` 中字符串的值。这是因为 Java 优先使用局部变量。

当父类和子类的变量名发生冲突时，它会优先使用局部变量，即 `childDemo` 类。如果我们需要在 `parentDemo` 类中打印字符串名称，该怎么办呢？为此，我们使用 `super` 关键字来引用从中继承属性到 `childDemo` 类的 `parentDemo` 类。因此，如果我们想要从 `parentDemo` 类中调用名称变量，我们添加一个打印语句，并在要打印的变量前添加 `super` 关键字，这将获取来自 `parentDemo` 的值。现在运行代码，我们将得到父对象和子对象作为输出，因为我们在两个类中都留下了名称字符串的打印语句。`parentDemo` 类的代码如下所示：

```java
public class parentDemo{
    String name= "Rahul";
    public static viod main (String[] args){
    }
}
```

`childDemo` 类的代码如下所示：

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

在这一部分，我们将看看在 Java 中使用 `super` 关键字的不同方式。

# 使用 super 关键字处理方法

我们看到了如何使用 `super` 关键字处理父变量。在本节中，我们还将看到如何处理 `parentDemo` 和 `childDemo` 类中名称相同的两个方法。我们也将在本节中使用之前的例子。

在`parentDemo`类中，添加一个名为`getData()`的方法，并在方法内部添加一个打印语句来显示`"I am in parent class"`消息。如果我们想在`childDemo`类中执行`getData()`方法，我们在`childDemo`类的`main`方法中写入`cd.getData()`。我们可以访问`getData()`，因为我们继承了`parentDemo`类的属性。如果我们运行`childDemo`类，我们将收到先前示例的输出以及我们在`parentDemo`类中添加的新句子`I am in parent class`。

在`childDemo`类中，我们将定义另一个与`parentDemo`类相同名称的方法，并添加一个打印语句来显示`I am in child class`消息。如果我们运行`childDemo`类，我们将得到先前示例的输出，然后显示`I am in child class`。这是因为优先考虑本地类，所以`childDemo`类中的`getData()`方法覆盖了`parentDemo`类中的`getData()`方法。

现在，我们想在`childDemo`类中使用`parentDemo`类的`getData()`方法。为此，我们只需像处理变量一样，在`childDemo`类的`getData()`方法中添加`super.getData()`。当我们运行`childDemo()`类时，我们得到先前示例的输出，然后是`I am in parent class`，然后是`I am in child class`。

# 使用`super`关键字进行构造函数

让我们在本节中使用`super`关键字进行构造函数。我们也将在这里使用先前的示例。

在`parentDemo`类中，我们定义一个构造函数`parentDemo()`，并添加一个打印语句来打印：`Parent class constructor`。

在`childDemo`中，我们定义一个构造函数`childDemo()`并添加一个打印语句来打印：`Child class constructor`。如果我们想在`childDemo`类中使用`parentDemo`类的构造函数，我们在`childDemo()`构造函数中添加`super()`方法。这样控制器就会调用`parentDemo`类中的构造函数。

在使用构造函数时，我们需要遵循一个重要的规则：每当在子构造函数中使用`super`构造函数时，它应该始终是第一行。

当我们运行`childDemo`类时，控制器首先执行`super()`方法。它进入`parentDemo()`构造函数并执行它，然后执行`childDemo()`。因此最终输出将是：

```java
Parent class constructor
Child class constructor
QAClickAcademy
Rahul
I am parent class
I am in child class
```

# `this`关键字的重要性

在 Java 中还有一个与`super`关键字类似的关键字：`this`。在本节中，我们将看一下`this`关键字。

让我们用一个例子来解释`this`关键字。创建一个名为`thisDemo`的类，并声明一个变量`a`，并将值`2`赋给它。我们在其类中定义一个`getData()`方法，在其中声明`a`变量，并将值`3`赋给它。我们还在其中添加一个打印语句。代码将如下所示：

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

正如我们所看到的，在整个类中`a`的值是`2`，但在一个特定的方法`getData()`中，我们希望变量的值是`3`。在这段代码中，我们想要调用`a`的两个值，即`2`和`3`。我们在主方法中创建一个对象，并将`td`对象添加到其中。`td`对象的代码如下：

```java
thisDemo td=new thisDemo();
td.getData();
```

如果我们运行代码，我们得到的输出是`3`。但是我们也希望在同一个块中将`a`的值打印为`2`。这就是`this`关键字发挥作用的时候。类对象的范围将在类级别而不是方法级别。因此，我们说`getData()`方法是指当前对象，对象范围位于类级别。因此`a=2`对整个类是有效的，而`a=3`仅对`getData()`方法有效。这就是为什么我们称`getData()`方法中的`a`变量为局部变量，而类中的`a`变量为全局变量。

要打印我们正在处理的示例的全局变量，我们需要在`getData()`方法中添加一个打印语句，并在打印语句中添加`this.a`。打印语句将如下所示：

```java
System.out.println(this.a);
```

当我们运行代码时，我们得到以下输出：

```java
3
2
```

这就结束了我们关于这个变量的示例。现在让我们学习一下异常。

# 不同种类的异常

在本节中，我们将看看如何在 Java 中处理异常。

一般来说，如果代码中有错误，我们需要捕获它并打印一条消息而不是失败；这可以使用`try...catch`机制来实现。因此，一般来说，当我们尝试编写代码并怀疑其中可能有错误时，我们将使用该错误进行异常处理。

我们将通过一个练习来解释它。让我们创建一个名为`exceptionDemo`的新类，在`main`块内声明`a`、`b`和`c`变量，并分别为它们赋值`4`、`7`和`0`。我们在主块内添加一个`try`块，并声明一个整数变量`k`，它等于`b`除以`c`。每当我们在`try`块中添加任何内容时，我们都在尝试看代码是否能正常工作。如果失败，控制器将退出这个`try`块并进入包含异常的`catch`块。一个重要的要点是`catch`块紧跟在`try`块后面。在`catch`块内，我们编写一个打印消息来显示`I caught the error/exception`。

当控制器进入`k`变量行时，脚本失败，因为`7/0`是无穷大，这是一个算术异常，但脚本不会立即失败。如果我们不编写`try...catch`块，我们会看到一种不同的错误。

让我们去掉`try...catch`块，运行代码，看看我们得到的错误。我们在输出部分看到一个错误，`Java.lang.ArithmeticException`；这是因为我们不能将`7`除以`0`，所以脚本突然失败了。

如果我们最初觉得我们的代码会出错，我们可以简单地编写一个脚本来通过并捕获错误，通过放置一个适当的调试消息，可以通过`try...catch`机制来处理。现在，让我们再次添加`try...catch`块并调试整个代码。输出将是`I caught the error/exception`；这是因为`7`除以`0`是无穷大，所以脚本应该失败，但我们在输出部分没有看到任何错误，说代码已经失败。这是因为控制器简单地移动到`catch`块并执行它。最终的代码将如下所示：

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

# 使用 try...catch 机制处理异常

在本节中，我们将使用一个`try`后面跟着多个`catch`块。Java 中有不同类型的异常，对于每个异常，我们可以添加单独的`catch`块。

让我们用之前的例子来解释一下。为之前的代码编写的异常是一个通用异常，因此对于`try`块中的任何错误，都会执行通用异常。现在让我们尝试捕获特定的异常。我们可以在`try`块下添加一个`catch`块，并添加一个特定的异常和一个打印语句来打印`I caught the Arithmeticerror/exception`。特定 catch 块的代码如下：

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

我们看到，当我们运行代码时，控制器进入`catch`块，因为`catch`块专门针对算术异常编写，而抛出的错误也属于算术错误。因此，一旦控制器收到错误，`try`块将查看与之相关的`catch`块的类型，并运行它。

Java 中还有许多其他异常：我们可以搜索一下看看它们。

# Java 中 finally 块的重要性

还有一个块就像`try...catch`块一样：就是`finally`块。`finally`块将被执行，无论是否抛出异常。如果程序成功运行，这个块将被执行，即使程序不运行也会执行。

我们将使用在*使用 try...catch 机制处理异常*部分中使用的示例来解释这一点。我们只需在`catch`块后面添加一个`finally`块，并在其中加上一个打印语句，说`delete cookies`。代码块将如下所示：

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

一个重要的点是`finally`可以与或不与`catch`块一起工作；它只需要在`try`块下面写就可以了。

# 总结

在本章中，我们看了一下`super`和`this`关键字。我们还看了一些例子来解释我们可以在哪些地方使用这些关键字来克服某些障碍。我们学习了异常，并在代码由于错误而失败时在各种情况下实现了它们。我们还学习了`finally`块。

在下一章中，我们将深入研究集合框架，其中包括接口和类。我们还将看一下三个主要的集合：`List`、`Set`和`Map`。
