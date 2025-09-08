# 第五章：关于接口和继承你需要知道的一切

在本章中，我们将探讨一些重要概念，例如接口、它们的工作原理以及在 Java 中的用法。我们将通过一个实际例子来讨论继承。本章还将探讨函数重载和函数重写的概念以及它们之间的区别。

我们将在本章中介绍以下内容：

+   接口

+   继承简介

+   函数重载

+   函数重写

# 接口

接口是 Java 面向对象编程中使用的核心概念之一，因此我们有必要熟悉接口及其用法。

接口与类相似。接口与类之间的唯一区别是接口将包含方法但没有方法体。困惑了吗？在一个类中，我们通常定义一个方法然后开始向其中编写代码。例如，在一个类中，如果我们想编写任何代码，我们只需通过使用`public void`声明该类，然后在该类中继续编写其余的代码，如下所示：

```java
public void getData()
{
}
```

在接口中，我们只能定义方法的签名；我们无法在方法内部编写任何代码。但为什么？在接口内部编写方法签名有什么用？这个面向对象的概念在 Java 中有什么用？你可能会有这些疑问，所以让我们尝试通过一个现实生活中的场景来理解接口的概念。

# 使用接口的交通信号灯系统

考虑典型的交通信号灯系统，它被全世界用来维护交通规则。每个国家都有自己的交通规则，例如在道路的左侧或右侧驾驶。尽管各国交通规则不同，但有一些规则是全球通用的，并且每个国家都需要遵守。其中一条规则是使用交通信号灯来控制交通流量，红灯表示停车，琥珀/黄色灯表示准备引擎，绿灯表示移动车辆。假设这些全球规则是由一个中央交通当局制定的，我们想要实现，例如，澳大利亚的交通系统。这个系统将有自己的规则，但我们需要确保它遵循中央交通当局规定的全球规则。

通过这个例子，我们将尝试理解接口的概念。在这里，中央交通当局充当接口，澳大利亚交通规则充当实现接口的类；也就是说，澳大利亚交通系统将必须遵循中央交通当局接口中提到的规则/方法。任何接口中定义的方法只是签名，因此类将定义并实现接口中存在的方法。让我们看看我们的 Java 代码中的这个例子。

我们定义接口的方式与定义类的方式相同。在这个交通信号灯示例中，让我们将类命名为`CentralTraffic`。我们现在有一个现成的接口，如下所示：

```java
package demopack;

public interface CentralTraffic {

    public void greenGo();
    public void redStop();
    public void FlashYellow();

}
```

我们可以在语法中看到，我们写的是`interface`而不是`class`。我们使用与在类中定义方法相同的方法在接口中定义一个方法，但请记住，我们不能有定义方法的主体，因为这是一个接口，这样做将引发错误。创建另一个类来实现这个接口，并将其命名为`AustralianTraffic`。一旦我们有了 Java 类，我们需要将`CentralTraffic`接口实现到它上面，我们使用`implements`关键字这样做，如下所示：

```java
public class AustralianTraffic implements CentralTraffic {
```

在使用前面的句子后，我们的集成开发环境（IDE）将显示一个错误，当你将鼠标悬停在错误上时，你会看到一些与错误相关的建议。其中一个建议是导入`CentralTraffic`，另一个建议是添加未实现的方法。点击这些建议以解决错误，你应该得到以下代码：

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

在`CentralTraffic`接口中定义的所有方法都可以在`AustralianTraffic`类中看到，我们也可以按自己的意愿实现这些方法。现在，如果我们从我们的 Java 类中移除`greenGo`方法，它将给出一个错误。因为这个方法是接口中定义的，所以我们必须实现接口中定义的所有方法。

接口方法是在`public static void main`外部定义的，要执行这些方法，我们应该在`main`方法中为它们创建一个类对象，如下所示：

```java
        CentralTraffic a= new AustralianTraffic();
```

这行代码表示我们为`AustralianTraffic`类创建了一个对象来实现`CentralTraffic`接口中的方法。主类应该如下所示：

```java
public class AustralianTraffic implements CentralTraffic {

    public static void main(String[] args) {
    CentralTraffic a= new AustralianTraffic();
    a.redStop();
    a.FlashYellow();
    a.greenGo();    
    }
```

现在，在实现了接口中的方法之后，我们可以在我们的 Java 类中定义我们自己的特定国家的方法（规则），如下所示：

```java
public void walkonsymbol()
{
    System.out.println("walking");
} 
```

在我们的`main`方法中，如果我们尝试使用`a.`调用我们的特定国家的方法，就像我们在`main`类中的其他方法那样做，那么我们会发现我们无法这样做，因为`walkonsymbol`方法特定于某个国家（即`AustralianTraffic`类）并且没有在`CentralTraffic`中实现。对于`walkonsymbol`方法，我们需要在`main`类中为`AustralianTraffic`类创建另一个对象，如下所示：

```java
        AustralianTraffic at=new AustralianTraffic();
        at.walkonsymbol();
```

与接口相关的一条信息是，一个类可以实现多个接口。假设我们创建另一个接口，例如`ContinentalTraffic`，并定义另一个与交通信号灯相关的规则，例如一个火车符号来表示火车正在通过。我们可以在我们的`AustralianTraffic`类中简单地通过添加逗号来实现这个接口，如下所示：

```java
public class AustralianTraffic implements CentralTraffic, ContinentalTraffic {
```

对于这个接口，我们需要遵循与 `CentralTraffic` 接口相同的步骤，例如将 `ContinentalTraffic` 导入到 `AustralianTraffic` 中，添加未实现的方法，在主类中创建特定的 `ContinentalTraffic` 对象，等等。

现在你对接口和类之间的区别有了相当的了解。我们学习了如何定义接口，如何在另一个类中实现它们，以及如何使用对象调用它们。

# 继承

继承是 Java 中另一个重要的面向对象编程（OOP）概念。让我们以一个车辆为例来理解继承的概念，就像我们使用交通信号系统示例来理解接口一样。车辆的基本属性包括其颜色、档位、后视镜、刹车等。假设我们正在制造一种新的车辆，它对某些属性进行了改进，例如具有更高 CC 的引擎，也许与旧款的设计不同。现在，要创建具有这些新功能的新的车辆，我们仍然需要旧车辆的基本功能，例如后视镜和刹车，这些功能在车辆中默认存在。

让我们以先前的例子为例，使用 Java 来反映这些关系，以便理解继承的概念。在我们的例子中，如果我们有一个车辆类，并将车辆的基本特征作为存在于该类中的方法，那么当我们创建一个新的车辆类时，它可以继承为车辆创建的类的特征，我们不需要为这些特征编写代码，因为它们通过继承对我们可用。

让我们从代码开始。创建一个 `parentClassdemo` 类，这将是我们的父类。在这个类中，我们将定义我们的方法，如下所示：

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

我们现在将在子类中继承这些方法。创建一个 Java 中的 `childClassDemo`。我们使用 `extends` 关键字继承父类，如下所示：

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

在这里，我们使用 `extends` 关键字在 `childClassDemo` 类中继承了 `parentClassdemo` 类。在这个 `childClassDemo` 类中，我们定义了自己的 `engine` 方法，并使用了从 `parentClassdemo` 类继承来的 `color` 方法。然后我们创建了一个 `cd` 对象，并使用它来调用继承类中的方法。

# 更多关于继承

让我们讨论一些关于 Java 继承中臭名昭著的棘手问题和误解。

让我们开始讨论一些关于继承的更常见问题。看一下以下代码块：

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

这也是继承与接口之间区别之一，因为接口允许我们同时使用多个接口。

看一下以下示例：

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

在这里，我们有一个`A`类，它有一个`i`变量。还有一个扩展了`A`类的`B`类，并且我们也将它的局部`i`变量设置为`20`。现在，在`MainClass`中，我们为`B`类创建了一个对象。这一步实际上意味着什么？在这里，我们正在创建一个对象，并说明这个`B`类的对象应该引用`A`类的属性。尽管我们有权限通过这个`a`对象访问`B`类，但我们只能访问`A`类的属性或方法，因为`B`类有权限在这里访问`A`类，因为我们正在扩展它。

这里的问题是`a.i`将打印什么——`20`还是`10`？答案是，它将打印变量值`10`，因为`A a = new B();`明确告诉`a`它是一个`B`类的对象，但我们需要访问`A`类中的方法。如果我们想得到`20`的输出，我们将语法改为`B a = new B();`。

如果你参加 Java 测验或复杂的面试，可能会遇到这样的问题。这些都是你必须知道的关于继承的重要信息，你可以据此进行规划。

# 函数重载

当一个类有多个同名方法时，就会发生函数重载。如果我们在我们类中定义两次`getData`方法，我们可以说`getData`函数被重载了，如下面的代码所示：

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

在使用具有相同名称的多个函数实例时，我们需要记住一些规则。第一条规则是函数重载方法中参数的数量应该不同，第二条规则是参数的数据类型应该不同。如果我们保留两个具有`int a`参数的`getData`方法，它将引发错误，因此我们需要为每个方法有不同的参数数量。现在，当你打印这些时，你会得到`2`和`hello`的输出。我们可以看到打印了两个不同的参数，但具有相同的名称。让我们再添加一个具有两个参数的`getData`实例，如下所示：

```java
    public void getData(int a, int b)
    {

    }
```

现在我们有两个`getData`实例，它们的数据类型相同，但参数的数量不同。

你在现实生活中也可能遇到函数重载，例如，当你在电子商务网站上被要求选择支付方式时。网站可能使用不同的`getPayment`方法来确认支付——一个`getPayment`方法接受借记卡作为参数，另一个`getPayment`方法接受信用卡作为参数，另一个`getPayment`可能接受礼品卡作为参数。所以我们将不同类型的参数传递给同一个`getPayment`方法。在这种情况下，我们坚持使用`getPayment`作为方法名，并传递不同的参数，将函数重载的概念应用到这个特定的场景中。

# 函数重写

在本节中，让我们再讨论 Java 中的一个重要特性——函数重写。让我们继续使用我们在学习继承时看到的相同示例。

在那个示例中，我们有一个名为`parentClassdemo`的父类和一个名为`childClassDemo`的子类，子类继承了父类，如下所示：

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

在这里，我们在子类中定义了`engine`方法，它打印一个新的引擎，我们还有一个名为`color`的方法，它在父类中定义，我们通过一个对象来调用它。如果我们打印这个，我们将得到`color`方法的输出，因为它是在父类中定义的。现在，我们在子类中创建一个新的方法，并将其命名为`color`，定义如下：

```java
    public void color()
    {
        System.out.println("update color");
    }
```

我们有两个`color`方法的实例——一个在父类中定义，另一个在子类中定义。这就是函数重写概念发挥作用的地方。如果你运行子类，你将得到`update color`的输出。这是因为子类中定义的新`color`方法覆盖了父类中的`color`方法。

这总结了函数重写的整个概念，其中两个方法具有相同的名称、签名和参数。在函数重载中，我们有具有相同名称但不同参数的方法。这是函数重载和函数重写之间的一项主要区别。

# 概述

在本章中，我们介绍了几个重要的 Java 面向对象概念，例如接口、继承、函数重载和函数重写。我们通过示例查看每个概念，这有助于我们更好地理解这些概念。

在下一章中，我们将探讨 Java 代码中最重要概念之一：数组。我们将看到不同数组的形态，以及如何初始化和显示它们。
