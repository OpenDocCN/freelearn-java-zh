# 第五章：关于接口和继承你需要知道的一切

在这一章中，我们将介绍一些重要的概念，比如接口、它们的工作原理以及它们在 Java 中的使用。我们将使用一个实际的例子来讨论继承。本章还将探讨函数重载和函数重写的概念以及它们之间的区别。

在本章中，我们将涵盖以下主题：

+   接口

+   继承介绍

+   函数重载

+   函数重写

# 接口

接口是 Java 面向对象编程中使用的核心概念之一，因此我们有必要熟悉接口及其用法。

接口和类很相似。接口和类之间唯一的区别是接口有方法但没有方法体。困惑了吗？在类中，我们通常定义一个方法，然后开始编写代码。例如，在一个类中，如果我们想要写任何代码，我们只需从`public void`开始声明类，并在该类中继续编写其余的代码，如下所示：

```java
public void getData()
{
}
```

在接口中，我们只能定义方法的签名；我们不能在方法内部编写任何代码。但是为什么？在接口中写方法签名有什么用？这个面向对象的概念在 Java 中有什么用？你可能会在心中有这些问题，所以让我们尝试用一个现实生活中的场景来理解接口的概念。

# 使用接口与交通灯系统

考虑到典型的交通灯系统，它在世界各地都被用来维护交通规则。每个国家都有自己的交通规则，比如在道路的左侧或右侧行驶。尽管交通规则因国家而异，但有一些规则是全球适用的，需要每个国家遵守。其中一个规则是使用交通灯来管理交通流量，红灯表示停车，黄灯表示准备发动引擎，绿灯表示行驶。假设这些全球规则是由一个中央交通管理机构实施的，我们想要实现，例如，澳大利亚的交通系统。这个系统将有自己的规则，但我们需要确保它遵循中央交通管理机构实施的全球规则。

通过这个例子，我们将尝试理解接口的概念。在这里，中央交通管理机构充当接口，澳大利亚交通规则充当实现接口的类；也就是说，澳大利亚交通系统必须遵循中央交通管理机构接口中提到的规则/方法。在任何接口中定义的方法只是签名，所以类将定义和实现接口中存在的方法。让我们在我们的 Java 代码中看看这个例子。

我们定义接口的方式与定义类的方式相同。在这个交通灯的例子中，让我们将类命名为`CentralTraffic`。我们现在有了一个准备好的接口，如下所示：

```java
package demopack;

public interface CentralTraffic {

    public void greenGo();
    public void redStop();
    public void FlashYellow();

}
```

我们可以看到语法中，我们使用了`interface`而不是`class`。我们使用与在类中定义方法相同的方法在接口中定义方法，但要记住，我们不能在接口中定义方法体，因为这是一个接口，这样做会报错。创建另一个类来实现这个接口，并将其命名为`AustralianTraffic`。一旦我们有了一个 Java 类，我们需要使用`implements`关键字将`CentralTraffic`接口实现到它上面，如下所示：

```java
public class AustralianTraffic implements CentralTraffic {
```

在使用上述句子后，我们的 IDE 会显示一个错误，当你将鼠标悬停在错误上时，会看到一些与错误相关的建议。一个建议是导入`CentralTraffic`，另一个是添加未实现的方法。点击这些建议来解决错误，你应该会得到以下代码：

```java
package coreJava;
import demopack.CentralTraffic;
public class AustralianTraffic implements CentralTraffic {

    public static void main(String[] args) {

    }
    @Override
    public void greenGo() {
        // TODO Auto-generated method stub
        System.out.println(" greengo implementation")
    }
    @Override
    public void redStop() {
        // TODO Auto-generated method stub
        System.out.println(" redstop implementation")
    }    
    @Override
    public void FlashingYellow() {
        // TODO Auto-generated method stub
        System.out.println(" flash yellow implementation")
    }

}
```

在`AustralianTraffic`类中可以看到`CentralTraffic`接口中定义的所有方法，我们也可以根据需要实现这些方法。现在，如果我们从 Java 类中删除`greenGo`方法，它将给我们一个错误。因为它是在接口中定义的方法，我们必须实现接口中定义的所有方法。

接口方法在`public static void main`之外定义，要执行这些方法，我们应该在`main`方法中为它们创建一个类对象，如下所示：

```java
        CentralTraffic a= new AustralianTraffic();
```

这行代码表示我们已经为`AustralianTraffic`类创建了一个对象，以实现`CentralTraffic`接口中存在的方法。主类应该如下所示：

```java
public class AustralianTraffic implements CentralTraffic {

    public static void main(String[] args) {
    CentralTraffic a= new AustralianTraffic();
    a.redStop();
    a.FlashYellow();
    a.greenGo();    
    }
```

现在，在实现接口的方法之后，我们可以在我们的 Java 类中定义我们自己的特定于国家的方法（规则），如下所示：

```java
public void walkonsymbol()
{
    System.out.println("walking");
} 
```

在我们的`main`方法中，如果我们尝试使用`a.`来调用我们的特定于国家的方法，就像我们在`main`类中为其他方法所做的那样，那么我们会发现我们无法这样做，因为`walkonsymbol`方法是特定于特定国家的（即`AustralianTraffic`类），并且它没有在`CentralTraffic`中实现。对于`walkonsymbol`方法，我们需要在`main`类中为`AustralianTraffic`类创建另一个对象，如下所示：

```java
        AustralianTraffic at=new AustralianTraffic();
        at.walkonsymbol();
```

与接口相关的另一条信息是，一个类可以实现多个接口。假设我们创建另一个接口，比如`ContinentalTraffic`，并定义与交通灯相关的另一条规则，比如火车符号表示火车正在通过。我们可以通过在`AustralianTraffic`类中添加逗号来实现这个接口，如下所示：

```java
public class AustralianTraffic implements CentralTraffic, ContinentalTraffic {
```

对于这个接口，我们需要遵循与`CentralTraffic`接口相同的步骤，比如将`ContinentalTraffic`导入`AustralianTraffic`，添加未实现的方法，在主类中创建一个特定于`ContinentalTraffic`的对象等。

现在你对接口和类之间的区别有了一个大致的了解。我们学会了如何定义接口，如何在另一个类中实现它们，以及如何使用对象调用它们。

# 继承

继承是 Java 中另一个重要的面向对象编程概念。让我们以车辆为例来理解继承的概念，就像我们在使用交通灯系统的例子中理解接口一样。车辆的基本属性是颜色、齿轮、镜子、刹车等。假设我们正在制造一辆具有某些属性的新车辆，比如具有更高 CC 的发动机，可能与旧车不同的设计。现在，要创建具有这些新特性的新车辆，我们仍然需要旧车辆的基本特性，比如默认情况下存在的镜子和刹车。

让我们以前面的例子为例，使用 Java 来反映这些关系，以便理解继承的概念。在我们的例子中，如果我们有一个车辆类，并将车辆的基本特征作为该类中存在的方法输入，那么当我们为新车辆创建一个类时，它可以继承为车辆创建的类的特征，我们不必编写这些特征的代码，因为它们通过继承对我们可用。

让我们开始编写代码。创建一个`parentClassdemo`类，这将是我们的父类。在这个类中，我们将定义我们的方法，如下：

```java
package coreJava;
public class parentClassdemo {

    String color = "red";

    public void Gear()
    {
        System.out.println("gear code is implemented");
    }
    public void Brakes()
    {
        System.out.println("brakes code is implemented");
    }
    public void audiosystem()
    {
        System.out.println("audiosystem code is implemented");
    }
}
```

现在我们将在子类中继承这些方法。在 Java 中创建一个`childClassDemo`。我们使用`extends`关键字继承父类，如下所示：

```java
package coreJava;
public class childClassDemo extends parentClassdemo {

    public void engine()
    {
        System.out.println("new engine");
    }
    public void color
    {
        System.out.println(color);
    }

    public static void main(String[] args) {
        childClassDemo cd=new childClassDemo();
        cd.color();
    }
```

在这里，我们使用`extends`关键字在`childClassDemo`类中继承了`parentClassdemo`类。在这个`childClassDemo`类中，我们定义了自己的`engine`方法，并使用了我们从`parentClassdemo`类继承的`color`方法。然后我们创建了一个`cd`对象，并用它来调用从继承类中的方法。

# 更多关于继承的内容

让我们讨论一些关于 Java 继承的臭名昭著的棘手问题和误解。

让我们开始讨论一些关于继承的更为著名的问题。看一下下面的代码块：

```java
class X
{
    //Class X members
}

class Y
{
    //Class Y members
}

class Z extends X, Y
{
    //Class Z members
}
X and Y class and some data fields or methods inside it. The Z class inherits the X and Y classes. Is this allowed? The answer is no. Java does not allows multiple inheritances, whereas it is allowed in C++. So here, we can conclude that the preceding code snippet is not right and will throw an error.
```

这也是继承和接口之间的一个区别，因为接口允许我们同时使用多个接口。

看一下下面的例子：

```java
class A
{
    int i = 10;
}

class B extends A
{
    int i = 20;
}

public class MainClass
{
    public static void main(String[] args)
    {
        A a = new B();
        System.out.println(a.i);
    }
}
```

在这里，我们有一个`A`类，它有一个`i`变量。还有一个`B`类，它扩展了`A`类，并且我们还有它的本地`i`变量设置为`20`。现在，在`MainClass`中，我们为`B`类创建一个对象。这一步实际上意味着什么？在这里，我们正在创建一个对象，并且说这个`B`类的对象应该引用`A`类的属性。虽然我们有权限通过这个`a`对象访问`B`类，但我们只能访问`A`类的属性或方法，因为`B`类在这里有权限访问`A`类，因为我们正在扩展它。

这里的问题是`a.i`会打印出`20`还是`10`？答案是，它会打印出`10`的变量值，因为`A a = new B();`明确告诉`a`它是`B`类的对象，但我们需要访问`A`类中的方法。如果我们想要输出`20`，我们将语法改为`B a = new B();`。

如果你参加 Java 测验或复杂的面试，你可能会遇到这样的问题。这些是你必须了解的关于继承的重要信息，你可以相应地进行计划。

# 函数重载

函数重载发生在一个类中有多个同名方法的情况下。如果我们在类中两次定义了`getData`方法，我们可以说`getData`函数被重载了，就像下面的代码所示：

```java
package coreJava;
//function overloading
public class childlevel extends childClassDemo {

    public void getData(int a)
    {

    }
    public void getData(String a)
    {

    }

    public static void main(String[] args) {
        childlevel cl=new childlevel();
        cl.getData(2);
        cl.getData("hello")
    }
}
```

在使用相同名称的函数的多个实例时，我们需要记住一些规则。第一条规则是函数重载的方法中的参数数量应该不同，第二条是参数数据类型应该不同。如果我们保留两个`getData`方法，都带有`int a`参数，它会抛出错误，所以我们需要为每个方法有不同数量的参数。现在，当你打印这些时，你会得到`2`和`hello`的输出。我们可以看到打印了两个不同的参数，但是使用了相同的方法名。让我们再添加一个带有两个参数的`getData`实例，如下所示：

```java
    public void getData(int a, int b)
    {

    }
```

现在我们有两个具有相同数据类型的`getData`实例，但参数数量不同。

你可能在现实世界中也会遇到函数重载，比如当你在电子商务网站中以分批方式被要求支付方式时。网站可能会使用不同的`getPayment`方法来确认支付——一个`getPayment`方法以借记卡作为参数，另一个`getPayment`方法以信用卡作为参数，另一个`getPayment`可能以礼品卡作为参数。因此，我们向同一个`getPayment`方法传递不同类型的参数。在这种情况下，我们坚持将`getPayment`作为方法名，并传递不同的参数，将函数重载的概念带入到这种特定的情景中。

# 函数覆盖

在这一部分，让我们讨论 Java 中另一个重要的特性——函数覆盖。让我们继续使用我们在学习继承时看到的相同例子。

在这个例子中，我们有一个名为`parentClassdemo`的父类和一个名为`childClassDemo`的子类，子类继承了父类，如下所示：

```java
package coreJava;
public class childClassDemo extends parentClassdemo {

    public void engine()
    {
        System.out.println("new engine");
    }

    public static void main(String[] args) {
        childClassDemo cd=new childClassDemo();
        cd.color();
    }
```

在这里，我们在子类中定义了`engine`方法，它打印一个新的引擎，还有另一个方法`color`，它在父类中定义，并且我们使用一个对象来调用它。如果我们打印这个，我们将得到`color`方法的输出，因为它在父类中定义。现在，我们在子类中创建一个新的方法，也将其命名为`color`，并定义如下：

```java
    public void color()
    {
        System.out.println("update color");
    }
```

我们有两个`color`方法的实例——一个在父类中定义，另一个在子类中定义。这就是函数重写概念发挥作用的地方。如果你运行子类，你将得到`update color`的输出。这是因为子类中定义的新`color`方法覆盖了父类中的`color`方法。

这总结了函数重写的整个概念，其中两个方法具有相同的名称、签名和参数。在函数重载中，我们有具有相同名称但不同参数的方法。这是函数重载和函数重写之间的一个主要区别。

# 总结

在本章中，我们介绍了一些重要的 Java 面向对象编程概念，如接口、继承、函数重载和函数重写。我们通过示例来看每个概念，这有助于我们更详细地理解这些概念。

在下一章中，我们将介绍 Java 代码中最重要的概念之一：数组。我们将看到不同类型的数组是如何样的，以及如何初始化和显示它们。
