# 第六章：面向对象编程

本章涵盖了在 Java 面试中遇到的与**面向对象编程**（**OOP**）相关的最流行的问题和问题。

请记住，我的目标不是教你面向对象编程，或者更一般地说，这本书的目标不是教你 Java。我的目标是教你如何在面试的情境下回答问题和解决问题。在这样的情境下，面试官希望得到一个清晰而简洁的答案；你没有时间写论文和教程。你必须能够清晰而有力地表达你的想法。你的答案应该是有意义的，你必须说服面试官你真正理解你在说什么，而不只是背诵一些空洞的定义。大多数情况下，你应该能够用一两个关键段落表达一篇文章或一本书的一章。

通过本章结束时，你将知道如何回答 40 多个涵盖面向对象编程基本方面的问题和问题。作为基本方面，你必须详细了解它们。如果你不知道这些问题的正确和简洁的答案，那么在面试中成功的机会将受到严重影响。

因此，让我们总结我们的议程如下：

+   面向对象编程概念

+   SOLID 原则

+   GOF 设计模式

+   编码挑战

让我们从与面向对象编程概念相关的问题开始。

# 技术要求

你可以在 GitHub 上找到本章中的所有代码。请访问以下链接：[`github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter06`](https://github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/tree/master/Chapter06)

# 理解面向对象编程概念

面向对象模型基于几个概念。这些概念对于计划设计和编写依赖于对象的应用程序的任何开发人员来说都必须熟悉。因此，让我们从以下列举它们开始：

+   对象

+   类

+   抽象

+   封装

+   继承

+   多态

+   协会

+   聚合

+   组合

通常，当这些概念被包含在问题中时，它们会以*什么是...?*为前缀。例如，*什么是对象?*，或者*什么是多态?*

重要说明

这些问题的正确答案是技术知识和现实世界的类比或例子的结合。避免冷冰冰的答案，没有超级技术细节和没有例子（例如，不要谈论对象的内部表示）。注意你说的话，因为面试官可能直接从你的答案中提取问题。如果你的答案中提到了一个概念，那么下一个问题可能会涉及到那个概念。换句话说，不要在你的答案中添加任何你不熟悉的方面。

因此，让我们在面试情境下回答与面向对象编程概念相关的问题。请注意，我们应用了*第五章*中学到的内容，*如何应对编码挑战*。更确切地说，我们遵循**理解问题**|**确定关键词/关键点**|**给出答案**的技巧。首先，为了熟悉这种技巧，我将提取关键点作为一个项目列表，并在答案中用斜体标出它们。

## 什么是对象？

你的答案中应该包含以下关键点：

+   对象是面向对象编程的核心概念之一。

+   对象是一个现实世界的实体。

+   对象具有状态（字段）和行为（方法）。

+   对象表示类的一个实例。

+   对象占据内存中的一些空间。

+   对象可以与其他对象通信。

现在，我们可以按照以下方式提出答案：

*对象是面向对象编程的核心概念之一。对象是现实世界的实体，比如汽车、桌子或猫。在其生命周期中，对象具有状态和行为。例如，猫的状态可以是颜色、名字和品种，而其行为可以是玩耍、吃饭、睡觉和喵喵叫。在 Java 中，对象通常是通过`new`关键字构建的类的实例，它的状态存储在字段中，并通过方法公开其行为。每个实例在内存中占据一些空间，并且可以与其他对象通信。例如，另一个对象男孩可以抚摸一只猫，然后它就会睡觉。

如果需要进一步的细节，那么你可能想谈论对象可以具有不同的访问修饰符和可见性范围，可以是可变的、不可变的或不可变的，并且可以通过垃圾收集器进行收集。

## 什么是类？

你应该在你的答案中封装的关键点是：

+   类是面向对象编程的核心概念之一。

+   类是创建对象的模板或蓝图。

+   类不会占用内存。

+   一个类可以被实例化多次。

+   一个类只做一件事。

现在，我们可以这样提出一个答案：

*类是面向对象编程的核心概念之一。类是构建特定类型对象所需的一组指令。我们可以把类想象成一个模板、蓝图或配方，告诉我们如何创建该类的对象。创建该类的对象是一个称为实例化的过程，通常通过`new`关键字完成。我们可以实例化任意多个对象。类定义不会占用内存，而是保存在硬盘上的文件中。一个类应该遵循的最佳实践之一是**单一职责原则**（SRP）。在遵循这个原则的同时，一个类应该被设计和编写成只做一件事。

如果需要进一步的细节，那么你可能想谈论类可以具有不同的访问修饰符和可见性范围，支持不同类型的变量（局部、类和实例变量），并且可以声明为`abstract`、`final`或`private`，嵌套在另一个类中（内部类），等等。

## 什么是抽象？

你应该在你的答案中封装的关键点是：

+   抽象是面向对象编程的核心概念之一。

+   抽象是将对用户有意义的东西暴露给他们，隐藏其余的细节。

+   抽象允许用户专注于应用程序的功能，而不是它是如何实现的。

+   在 Java 中，通过抽象类和接口实现抽象。

现在，我们可以这样提出一个答案：

爱因斯坦声称*一切都应该尽可能简单，但不要过于简单*。*抽象是面向对象编程的主要概念之一*，旨在尽可能简化用户的操作。换句话说，*抽象只向用户展示对他们有意义的东西，隐藏其余的细节*。在面向对象编程的术语中，我们说一个对象应该向其用户只公开一组高级操作，而这些操作的内部实现是隐藏的。因此，*抽象允许用户专注于应用程序的功能，而不是它是如何实现的*。这样，抽象减少了暴露事物的复杂性，增加了代码的可重用性，避免了代码重复，并保持了低耦合和高内聚。此外，它通过只暴露重要细节来维护应用程序的安全性和保密性。

让我们考虑一个现实生活的例子：一个人开车。这个人知道每个踏板的作用，以及方向盘的作用，但他不知道这些事情是车内部是如何完成的。他不知道赋予这些事情力量的内部机制。这就是抽象。*在 Java 中，可以通过抽象类和接口实现抽象*。

如果需要更多细节，你可以分享屏幕或使用纸和笔编写你的例子。

所以，我们说一个人在开车。这个人可以通过相应的踏板加速或减速汽车。他还可以通过方向盘左转和右转。所有这些操作都被分组在一个名为`Car`的接口中：

```java
public interface Car {
    public void speedUp();
    public void slowDown();
    public void turnRight();
    public void turnLeft();
    public String getCarType();
}
```

接下来，每种类型的汽车都应该实现`Car`接口，并重写这些方法来提供这些操作的实现。这个实现对用户（驾驶汽车的人）是隐藏的。例如，`ElectricCar`类如下所示（实际上，我们有复杂的业务逻辑代替了`System.out.println`）：

```java
public class ElectricCar implements Car {
    private final String carType;
    public ElectricCar(String carType) {
        this.carType = carType;
    }        
    @Override
    public void speedUp() {
        System.out.println("Speed up the electric car");
    }
    @Override
    public void slowDown() {
        System.out.println("Slow down the electric car");
    }
    @Override
    public void turnRight() {
        System.out.println("Turn right the electric car");
    }
    @Override
    public void turnLeft() {
        System.out.println("Turn left the electric car");
    }
    @Override
    public String getCarType() {
        return this.carType;
    }        
}
```

这个类的用户可以访问这些公共方法，而不需要了解具体的实现：

```java
public class Main {
    public static void main(String[] args) {
        Car electricCar = new ElectricCar("BMW");
        System.out.println("Driving the electric car: " 
		  + electricCar.getCarType() + "\n");
        electricCar.speedUp();
        electricCar.turnLeft();
        electricCar.slowDown();
    }
}
```

输出列举如下：

```java
Driving the electric car: BMW
Speed up the electric car
Turn left the electric car
Slow down the electric car
```

所以，这是一个通过接口进行抽象的例子。完整的应用程序名为*Abstraction/AbstractionViaInterface*。在本书附带的代码中，你可以找到通过抽象类实现相同场景的代码。完整的应用程序名为*Abstraction/AbstractionViaAbstractClass*。

接下来，让我们谈谈封装。

## 什么是封装？

你应该在你的答案中封装的关键点如下：

+   封装是面向对象编程的核心概念之一。

+   封装是一种技术，通过它，对象状态被隐藏，同时提供了一组公共方法来访问这个状态。

+   当每个对象将其状态私有化在一个类内部时，封装就实现了。

+   封装被称为*数据隐藏*机制。

+   封装有许多重要的优点，比如松散耦合、可重用、安全和易于测试的代码。

+   在 Java 中，封装是通过访问修饰符（`public`、`private`和`protected`）实现的。

现在，我们可以这样呈现一个答案：

封装是面向对象编程的核心概念之一。主要来说，封装将代码和数据绑定在一个单元（类）中，并充当一个防御屏障，不允许外部代码直接访问这些数据。主要来说，它是隐藏对象状态，向外部提供一组公共方法来访问这个状态的技术。当每个对象将其状态私有化在一个类内部时，我们可以说封装已经实现。这就是为什么封装也被称为公共、私有和受保护。通常，当一个对象管理自己的状态时，其状态通过私有变量声明，并通过公共方法访问和/或修改。让我们举个例子：一个`Cat`类可以通过`mood`、`hungry`和`energy`等字段来表示其状态。虽然`Cat`类外部的代码不能直接修改这些字段中的任何一个，但它可以调用`play()`、`feed()`和`sleep()`等公共方法来在内部修改`Cat`的状态。`Cat`类也可能有私有方法，外部无法访问，比如`meow()`。这就是封装。

如果需要更多细节，你可以分享屏幕或使用纸和笔编写你的例子。

所以，我们的例子中的`Cat`类可以按照下面的代码块进行编码。注意，这个类的状态是通过私有字段封装的，因此不能直接从类外部访问：

```java
public class Cat {
    private int mood = 50;
    private int hungry = 50;
    private int energy = 50;
    public void sleep() {
        System.out.println("Sleep ...");
        energy++;
        hungry++;
    }
    public void play() {
        System.out.println("Play ...");
        mood++;
        energy--;
        meow();
    }
    public void feed() {
        System.out.println("Feed ...");
        hungry--;
        mood++;
        meow();
    }
    private void meow() {
        System.out.println("Meow!");
    }
    public int getMood() {
        return mood;
    }
    public int getHungry() {
        return hungry;
    }
    public int getEnergy() {
        return energy;
    }
}
```

修改状态的唯一方式是通过`play()`、`feed()`和`sleep()`这些公共方法，就像下面的例子一样：

```java
public static void main(String[] args) {
    Cat cat = new Cat();
    cat.feed();
    cat.play();
    cat.feed();
    cat.sleep();
    System.out.println("Energy: " + cat.getEnergy());
    System.out.println("Mood: " + cat.getMood());
    System.out.println("Hungry: " + cat.getHungry());
}
```

输出将如下所示：

```java
Feed ...Meow!Play ...Meow!Feed ...Meow!Sleep ...
Energy: 50
Mood: 53
Hungry: 49
```

完整的应用程序名为*Encapsulation*。现在，让我们来了解一下继承。

## 什么是继承？

你应该在你的答案中封装的关键点如下：

+   继承是面向对象编程的核心概念之一。

+   继承允许一个对象基于另一个对象。

+   继承通过允许一个对象重用另一个对象的代码并添加自己的逻辑来实现代码的可重用性。

+   继承被称为**IS-A**关系，也被称为父子关系。

+   在 Java 中，继承是通过`extends`关键字实现的。

+   继承的对象被称为超类，继承超类的对象被称为子类。

+   在 Java 中，不能继承多个类。

现在，我们可以这样呈现一个答案：

“继承是面向对象编程的核心概念之一。它允许一个对象基于另一个对象”，当不同的对象非常相似并共享一些公共逻辑时，这是很有用的，但它们并不完全相同。“继承通过允许一个对象重用另一个对象的代码来实现代码的可重用性，同时它也添加了自己的逻辑”。因此，为了实现继承，我们重用公共逻辑并将独特的逻辑提取到另一个类中。“这被称为 IS-A 关系，也被称为父子关系”。就像说`Foo`是`Buzz`类型的东西一样。例如，猫是猫科动物，火车是车辆。IS-A 关系是用来定义类层次结构的工作单元。“在 Java 中，继承是通过`extends`关键字实现的，通过从父类派生子类”。子类可以重用其父类的字段和方法，并添加自己的字段和方法。“继承的对象被称为超类，或者父类，继承超类的对象被称为子类，或者子类。在 Java 中，继承不能是多重的；因此，子类或子类不能继承多于一个超类或父类的字段和方法。例如，`Employee`类（父类）可以定义软件公司任何员工的公共逻辑，而另一个类（子类），名为`Programmer`，可以扩展`Employee`以使用这个公共逻辑并添加特定于程序员的逻辑。其他类也可以扩展`Programmer`或`Employee`类。”

如果需要更多细节，你可以分享屏幕或使用纸和笔编写你的例子。

`Employee`类非常简单。它包装了员工的名字：

```java
public class Employee {
    private String name;
    public Employee(String name) {
        this.name = name;
    }
    // getters and setters omitted for brevity
}
```

然后，`Programmer`类扩展了`Employee`。像任何员工一样，程序员有一个名字，但他们也被分配到一个团队中：

```java
public class Programmer extends Employee {
    private String team;
    public Programmer(String name, String team) {
        super(name);
        this.team = team;
    }
    // getters and setters omitted for brevity
}
```

现在，让我们通过创建一个`Programmer`并调用从`Employee`类继承的`getName()`和从`Programmer`类继承的`getTeam()`来测试继承：

```java
public static void main(String[] args) {
    Programmer p = new Programmer("Joana Nimar", "Toronto");
    String name = p.getName();
    String team = p.getTeam();
    System.out.println(name + " is assigned to the " 
          + team + " team");
}
```

输出将如下所示：

```java
Joana Nimar is assigned to the Toronto team
```

完整的应用程序被命名为*继承*。接下来，让我们谈谈多态。

## 什么是多态？

你应该在你的答案中包含的关键点是：

+   多态是面向对象编程的核心概念之一。

+   多态在希腊语中意味着“多种形式”。

+   多态允许对象在某些情况下表现得不同。

+   多态可以通过方法重载（称为编译时多态）或通过方法重写来实现 IS-A 关系（称为运行时多态）。

现在，我们可以这样呈现一个答案：

*多态是面向对象编程的核心概念之一*。多态是由两个希腊单词组成的：*poly*，意思是*多*，*morph*，意思是*形式*。因此，*多态意味着多种形式*。

更准确地说，在面向对象编程的上下文中，*多态性允许对象在某些情况下表现不同*，或者换句话说，允许以不同的方式（方法）完成某个动作。*实现多态性的一种方式是通过方法重载。这被称为编译时多态性*，因为编译器可以在编译时识别调用重载方法的形式（具有相同名称但不同参数的多个方法）。因此，根据调用的重载方法的形式，对象的行为会有所不同。例如，名为`Triangle`的类可以定义多个带有不同参数的`draw()`方法。

*另一种实现多态性的方法是通过方法重写，当我们有一个 IS-A 关系时，这是常见的方法。这被称为运行时多态性，或动态方法分派*。通常，我们从一个包含一堆方法的接口开始。接下来，每个类实现这个接口并重写这些方法以提供特定的行为。这次，多态性允许我们像使用其父类（接口）一样使用这些类中的任何一个，而不会混淆它们的类型。这是可能的，因为在运行时，Java 可以区分这些类并知道使用哪一个。例如，一个名为`Shape`的接口可以声明一个名为`draw()`的方法，而`Triangle`、`Rectangle`和`Circle`类实现了`Shape`接口并重写了`draw()`方法来绘制相应的形状。

如果需要进一步的细节，那么你可以分享屏幕或使用纸和笔编写你的例子。

### 通过方法重载实现多态性（编译时）

`Triangle`类包含三个`draw()`方法，如下所示：

```java
public class Triangle {
    public void draw() {
        System.out.println("Draw default triangle ...");
    }
    public void draw(String color) {
        System.out.println("Draw a triangle of color " 
            + color);
    }
    public void draw(int size, String color) {
        System.out.println("Draw a triangle of color " + color
           + " and scale it up with the new size of " + size);
    }
}
```

接下来，注意相应的`draw()`方法是如何被调用的：

```java
public static void main(String[] args) {
    Triangle triangle = new Triangle();
    triangle.draw();
    triangle.draw("red");
    triangle.draw(10, "blue");
}
```

输出将如下所示：

```java
Draw default triangle ...
Draw a triangle of color red
Draw a triangle of color blue and scale it up 
with the new size of 10
```

完整的应用程序名为*多态性/编译时*。接下来，让我们看一个实现运行时多态性的例子。

### 通过方法重写实现多态性（运行时）

这次，`draw()`方法是在一个接口中声明的，如下所示：

```java
public interface Shape {
    public void draw();
}
```

`Triangle`、`Rectangle`和`Circle`类实现了`Shape`接口并重写了`draw()`方法来绘制相应的形状：

```java
public class Triangle implements Shape {
    @Override
    public void draw() {
        System.out.println("Draw a triangle ...");
    }
}
public class Rectangle implements Shape {
    @Override
    public void draw() {
        System.out.println("Draw a rectangle ...");
    }
}
public class Circle implements Shape {
    @Override
    public void draw() {
        System.out.println("Draw a circle ...");
    }
}
```

接下来，我们创建一个三角形、一个矩形和一个圆。对于这些实例中的每一个，让我们调用`draw()`方法：

```java
public static void main(String[] args) {
    Shape triangle = new Triangle();
    Shape rectangle = new Rectangle();
    Shape circle = new Circle();
    triangle.draw();
    rectangle.draw();
    circle.draw();
}
```

输出显示，在运行时，Java 调用了正确的`draw()`方法：

```java
Draw a triangle ...
Draw a rectangle ...
Draw a circle ...
```

完整的应用程序名为*多态性/运行时*。接下来，让我们谈谈关联。

重要提示

有人认为多态性是面向对象编程中最重要的概念。此外，也有声音认为运行时多态性是唯一真正的多态性，而编译时多态性实际上并不是一种多态性形式。在面试中引发这样的辩论是不建议的。最好是充当调解人，提出事情的两面。我们很快将讨论如何处理这种情况。

## 什么是关联？

你的答案中应该包含的关键点是：

+   关联是面向对象编程的核心概念之一。

+   关联定义了两个相互独立的类之间的关系。

+   关联没有所有者。

+   关联可以是一对一、一对多、多对一和多对多。

现在，我们可以给出如下答案：

*关联是面向对象编程的核心概念之一。关联的目标是定义两个类之间独立于彼此的关系*，也被称为对象之间的多重性关系。*没有关联的所有者*。参与关联的对象可以互相使用（双向关联），或者只有一个使用另一个（单向关联），但它们有自己的生命周期。*关联可以是单向/双向，一对一，一对多，多对一和多对多*。例如，在`Person`和`Address`对象之间，我们可能有一个双向多对多的关系。换句话说，一个人可以与多个地址相关联，而一个地址可以属于多个人。然而，人可以存在而没有地址，反之亦然。

如果需要进一步的细节，那么你可以分享屏幕或使用纸和笔编写你的例子。

`Person`和`Address`类非常简单：

```java
public class Person {
    private String name;
    public Person(String name) {
        this.name = name;
    }
    // getters and setters omitted for brevity
}
public class Address {
    private String city;
    private String zip;
    public Address(String city, String zip) {
        this.city = city;
        this.zip = zip;
    }
    // getters and setters omitted for brevity
}
```

`Person`和`Address`之间的关联是在`main()`方法中完成的，如下面的代码块所示：

```java
public static void main(String[] args) {
    Person p1 = new Person("Andrei");
    Person p2 = new Person("Marin");
    Address a1 = new Address("Banesti", "107050");
    Address a2 = new Address("Bucuresti", "229344");
    // Association between classes in the main method 
    System.out.println(p1.getName() + " lives at address "
            + a2.getCity() + ", " + a2.getZip()
            + " but it also has an address at "
            + a1.getCity() + ", " + a1.getZip());
    System.out.println(p2.getName() + " lives at address "
            + a1.getCity() + ", " + a1.getZip()
            + " but it also has an address at "
            + a2.getCity() + ", " + a2.getZip());
}
```

输出如下所示：

```java
Andrei lives at address Bucuresti, 229344 but it also has an address at Banesti, 107050
Marin lives at address Banesti, 107050 but it also has an address at Bucuresti, 229344
```

完整的应用程序被命名为*Association*。接下来，让我们谈谈聚合。

## 什么是聚合？

你的答案中应该包含的关键点如下：

+   聚合是面向对象编程的核心概念之一。

+   聚合是单向关联的特殊情况。

+   聚合代表一个 HAS-A 关系。

+   两个聚合对象有各自的生命周期，但其中一个对象是 HAS-A 关系的所有者。

现在，我们可以这样呈现一个答案：

*聚合是面向对象编程的核心概念之一*。主要是，聚合是单向关联的特殊情况。当一个关联定义了两个类之间独立于彼此的关系时，*聚合代表这两个类之间的 HAS-A 关系*。换句话说，*两个聚合对象有各自的生命周期，但其中一个对象是 HAS-A 关系的所有者*。有自己的生命周期意味着结束一个对象不会影响另一个对象。例如，一个`TennisPlayer`有一个`Racket`。这是一个单向关联，因为一个`Racket`不能拥有一个`TennisPlayer`。即使`TennisPlayer`死亡，`Racket`也不会受到影响。

重要提示

请注意，当我们定义聚合的概念时，我们也对关联有了一个陈述。每当两个概念紧密相关且其中一个是另一个的特殊情况时，都要遵循这种方法。下一步，同样的做法被应用于将组合定义为聚合的特殊情况。面试官会注意到并赞赏你对事物的概览，并且你能够提供一个有意义的答案，没有忽视上下文。

如果需要进一步的细节，那么你可以分享屏幕或使用纸和笔编写你的例子。

我们从`Rocket`类开始。这是网球拍的简单表示：

```java
public class Racket {
    private String type;
    private int size;
    private int weight;
    public Racket(String type, int size, int weight) {
        this.type = type;
        this.size = size;
        this.weight = weight;
    }
    // getters and setters omitted for brevity
}
```

一个`TennisPlayer`拥有一个`Racket`。因此，`TennisPlayer`类必须能够接收一个`Racket`，如下所示：

```java
public class TennisPlayer {
    private String name;
    private Racket racket;
    public TennisPlayer(String name, Racket racket) {
        this.name = name;
        this.racket = racket;
    }
    // getters and setters omitted for brevity
}
```

接下来，我们创建一个`Racket`和一个使用这个`Racket`的`TennisPlayer`：

```java
public static void main(String[] args) {
    Racket racket = new Racket("Babolat Pure Aero", 100, 300);
    TennisPlayer player = new TennisPlayer("Rafael Nadal", 
        racket);
    System.out.println("Player " + player.getName() 
        + " plays with " + player.getRacket().getType());
}
```

输出如下：

```java
Player Rafael Nadal plays with Babolat Pure Aero
```

完整的应用程序被命名为*Aggregation*。接下来，让我们谈谈组合。

## 什么是组合？

你的答案中应该包含的关键点如下：

+   组合是面向对象编程的核心概念之一。

+   组合是聚合的一种更为严格的情况。

+   组合代表一个包含一个不能独立存在的对象的 HAS-A 关系。

+   组合支持对象的代码重用和可见性控制。

现在，我们可以这样呈现一个答案：

*组合是面向对象编程的核心概念之一*。*主要来说，组合是聚合的一种更严格的情况*。聚合表示两个对象之间具有自己的生命周期的 HAS-A 关系，*组合表示包含一个不能独立存在的对象的 HAS-A 关系*。为了突出这种耦合，HAS-A 关系也可以被称为 PART-OF。例如，一个`Car`有一个`Engine`。换句话说，发动机是汽车的一部分。如果汽车被销毁，那么发动机也会被销毁。组合被认为比继承更好，因为*它维护了对象的代码重用和可见性控制*。

如果需要进一步的细节，那么你可以分享屏幕或使用纸和笔编写你的例子。

`Engine`类非常简单：

```java
public class Engine {
    private String type;
    private int horsepower;
    public Engine(String type, int horsepower) {
        this.type = type;
        this.horsepower = horsepower;
    }
    // getters and setters omitted for brevity
}
```

接下来，我们有`Car`类。查看这个类的构造函数。由于`Engine`是`Car`的一部分，我们用`Car`创建它：

```java
public class Car {
    private final String name;
    private final Engine engine;
    public Car(String name) {
        this.name = name;
        Engine engine = new Engine("petrol", 300);
        this.engine=engine;
    }
    public int getHorsepower() {
        return engine.getHorsepower();
    }
    public String getName() {
        return name;
   }    
}
```

接下来，我们可以从`main()`方法中测试组合如下：

```java
public static void main(String[] args) {
    Car car = new Car("MyCar");
    System.out.println("Horsepower: " + car.getHorsepower());
}
```

输出如下：

```java
Horsepower: 300
```

完整的应用程序被命名为*组合***。**

到目前为止，我们已经涵盖了关于面向对象编程概念的基本问题。请记住，这些问题几乎可以在涉及编码或架构应用程序的任何职位的 Java 技术面试中出现。特别是如果你有大约 2-4 年的经验，那么你被问到上述问题的机会很高，你必须知道答案，否则这将成为你的一个污点。

现在，让我们继续讨论 SOLID 原则。这是另一个基本领域，与面向对象编程概念并列的必须知道的主题。在这个领域缺乏知识将在最终决定你的面试时证明是有害的。

# 了解 SOLID 原则

在这一部分，我们将对与编写类的五个著名设计模式对应的问题进行回答 - SOLID 原则。简而言之，SOLID 是以下内容的首字母缩写：

+   **S**：单一责任原则

+   **O**：开闭原则

+   **L**：里氏替换原则

+   **I**：接口隔离原则

+   **D**：依赖反转原则

在面试中，与 SOLID 相关的最常见的问题是*什么是...?*类型的。例如，*S 是什么？*或者* D 是什么？*通常，与面向对象编程相关的问题是故意模糊的。这样，面试官测试你的知识水平，并希望看到你是否需要进一步的澄清。因此，让我们依次解决这些问题，并提供一个令面试官印象深刻的答案。

## S 是什么？

你应该在你的答案中概括的关键点如下：

+   S 代表**单一责任原则**（SRP）。

+   S 代表*一个类应该只有一个责任*。

+   S 告诉我们为了一个目标编写一个类。

+   S 维护了整个应用程序模块的高可维护性和可见性控制。

现在，我们可以给出以下答案：

首先，SOLID 是 Robert C. Martin（也被称为 Uncle Bob）阐述的前五个**面向对象设计（OOD）**原则的首字母缩写。*S*是 SOLID 的第一个原则，被称为**单一责任原则**（**SRP**）。*这个原则意味着一个类应该只有一个责任*。这是一个非常重要的原则，应该在任何类型的项目中遵循，无论是任何类型的类（模型、服务、控制器、管理类等）。*只要我们为一个目标编写一个类，我们就能在整个应用程序模块中保持高可维护性和可见性控制*。换句话说，通过*保持高可维护性*，这个原则对业务有重大影响，通过*提供应用程序模块的可见性控制*，这个原则维护了封装性。

如果需要更多细节，那么你可以分享屏幕或使用纸和笔编写一个像这里呈现的例子一样的例子。

例如，你想计算一个矩形的面积。矩形的尺寸最初以米为单位给出，面积也以米为单位计算，但我们希望能够将计算出的面积转换为其他单位，比如英寸。让我们看一下违反 SRP 的方法。

违反 SRP

在单个类`RectangleAreaCalculator`中实现前面的问题可以这样做。但是这个类做了不止一件事：它违反了 SRP。请记住，通常当你用“和”这个词来表达一个类做了什么时，这是 SRP 被违反的迹象。例如，下面的类计算面积并将其转换为英寸：

```java
public class RectangleAreaCalculator {
    private static final double INCH_TERM = 0.0254d;
    private final int width;
    private final int height;
    public RectangleAreaCalculator(int width, int height) {
        this.width = width;
        this.height = height;
    }
    public int area() {
        return width * height;
    }
    // this method breaks SRP
    public double metersToInches(int area) {
        return area / INCH_TERM;
    }    
}
```

由于这段代码违反了 SRP，我们必须修复它以遵循 SRP。

### 遵循 SRP

通过从`RectangleAreaCalculator`中删除“metersToInches（）”方法来解决这种情况，如下所示：

```java
public class RectangleAreaCalculator {
    private final int width;
    private final int height;
    public RectangleAreaCalculator(int width, int height) {
        this.width = width;
        this.height = height;
    }
    public int area() {
        return width * height;
    }       
}
```

现在，`RectangleAreaCalculator`只做一件事（计算矩形面积），从而遵守 SRP。

接下来，可以将“metersToInches（）”提取到一个单独的类中。此外，我们还可以添加一个新的方法来将米转换为英尺：

```java
public class AreaConverter {
    private static final double INCH_TERM = 0.0254d;
    private static final double FEET_TERM = 0.3048d;
    public double metersToInches(int area) {
        return area / INCH_TERM;
    }
    public double metersToFeet(int area) {
        return area / FEET_TERM;
    }
}
```

这个类也遵循了 SRP，因此我们的工作完成了。完整的应用程序被命名为“SingleResponsabilityPrinciple”。接下来，让我们谈谈第二个 SOLID 原则，即开闭原则。

O 是什么？

你应该在你的答案中包含的关键点是：

+   O 代表开闭原则（OCP）。

+   O 代表“软件组件应该对扩展开放，但对修改关闭”。

+   O 维持了这样一个事实，即我们的类不应该包含需要其他开发人员修改我们的类才能完成工作的约束条件-其他开发人员应该只能扩展我们的类来完成他们的工作。

+   O 以一种多才多艺、直观且无害的方式维持软件的可扩展性。

现在，我们可以这样回答：

首先，SOLID 是 Robert C. Martin 提出的前五个面向对象设计（OOD）原则的首字母缩写，也被称为 Uncle Bob（可选短语）。 O 是 SOLID 中的第二个原则，被称为开闭原则（OCP）。这个原则代表“软件组件应该对扩展开放，但对修改关闭”。这意味着我们的类应该被设计和编写成其他开发人员可以通过简单地扩展它们来改变这些类的行为。因此，“我们的类不应该包含需要其他开发人员修改我们的类才能完成工作的约束条件-其他开发人员应该只能扩展我们的类来完成工作”。

虽然我们“必须以一种多才多艺、直观且无害的方式维持软件的可扩展性”，但我们不必认为其他开发人员会想要改变我们的类的整个逻辑或核心逻辑。主要是，如果我们遵循这个原则，那么我们的代码将作为一个良好的框架，不会让我们修改它们的核心逻辑，但我们可以通过扩展一些类、传递初始化参数、重写方法、传递不同的选项等来修改它们的流程和/或行为。

如果需要更多细节，那么你可以分享屏幕或使用纸和笔编写一个像这里呈现的例子一样的例子。

现在，例如，你有不同的形状（例如矩形、圆）并且我们想要求它们的面积之和。首先，让我们看一下违反 OCP 的实现。

### 违反 OCP

每个形状都将实现`Shape`接口。因此，代码非常简单：

```java
public interface Shape {    
}
public class Rectangle implements Shape {
    private final int width;
    private final int height;
    // constructor and getters omitted for brevity
}
public class Circle implements Shape {
    private final int radius;
    // constructor and getter omitted for brevity
}
```

在这一点上，我们可以很容易地使用这些类的构造函数来创建不同尺寸的矩形和圆。一旦我们有了几种形状，我们想要求它们的面积之和。为此，我们可以定义一个`AreaCalculator`类，如下所示：

```java
public class AreaCalculator {
    private final List<Shape> shapes;
    public AreaCalculator(List<Shape> shapes) {
        this.shapes = shapes;
    }
    // adding more shapes requires us to modify this class
    // this code is not OCP compliant
    public double sum() {
        int sum = 0;
        for (Shape shape : shapes) {
            if (shape.getClass().equals(Circle.class)) {
                sum += Math.PI * Math.pow(((Circle) shape)
                    .getRadius(), 2);
            } else 
            if(shape.getClass().equals(Rectangle.class)) {
                sum += ((Rectangle) shape).getHeight() 
                    * ((Rectangle) shape).getWidth();
            }
        }
        return sum;
    }
}
```

由于每种形状都有自己的面积公式，我们需要一个`if-else`（或`switch`）结构来确定形状的类型。此外，如果我们想要添加一个新的形状（例如三角形），我们必须修改`AreaCalculator`类以添加一个新的`if`情况。这意味着前面的代码违反了 OCP。修复这段代码以遵守 OCP 会对所有类进行多处修改。因此，请注意，即使是简单的例子，修复不遵循 OCP 的代码可能会非常棘手。

### 遵循 OCP

主要思想是从`AreaCalculator`中提取每种形状的面积公式，并将其放入相应的`Shape`类中。因此，矩形将计算其面积，圆形也是如此，依此类推。为了强制每种形状必须计算其面积，我们将`area()`方法添加到`Shape`合同中：

```java
public interface Shape { 
    public double area();
}
```

接下来，`Rectangle`和`Circle`实现`Shape`如下：

```java
public class Rectangle implements Shape {
    private final int width;
    private final int height;
    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }
    public double area() {
        return width * height;
    }
}
public class Circle implements Shape {
    private final int radius;
    public Circle(int radius) {
        this.radius = radius;
    }
    @Override
    public double area() {
        return Math.PI * Math.pow(radius, 2);
    }
}
```

现在，`AreaCalculator`可以循环遍历形状列表，并通过调用适当的`area()`方法来计算面积。

```java
public class AreaCalculator {
    private final List<Shape> shapes;
    public AreaCalculator(List<Shape> shapes) {
        this.shapes = shapes;
    }
    public double sum() {
        int sum = 0;
        for (Shape shape : shapes) {
            sum += shape.area();
        }
        return sum;
    }
}
```

代码符合 OCP。我们可以添加一个新的形状，而不需要修改`AreaCalculator`。因此，`AreaCalculator`对于修改是封闭的，当然，对于扩展是开放的。完整的应用程序被命名为*开闭原则*。接下来，让我们谈谈第三个 SOLID 原则，Liskov 替换原则。

## 什么是 L？

您应该在您的答案中封装以下关键点：

+   L 代表**Liskov 替换原则** **（LSP）**。

+   L 代表*派生类型必须完全可替换其基类型*。

+   L 支持子类的对象必须与超类的对象以相同的方式行为。

+   L 对于运行时类型识别后跟随转换是有用的。

现在，我们可以如下呈现一个答案：

首先，SOLID 是前五个`foo(p)`的首字母缩写，其中`p`是类型`T`。然后，如果`q`是类型`S`，并且`S`是`T`的子类型，那么`foo(q)`应该正常工作。

如果需要进一步的细节，那么您可以共享屏幕或使用纸张和笔来编写一个像这里呈现的例子一样的例子。

我们有一个接受三种类型会员的国际象棋俱乐部：高级会员、VIP 会员和免费会员。我们有一个名为`Member`的抽象类，它充当基类，以及三个子类-`PremiumMember`、`VipMember`和`FreeMember`。让我们看看这些会员类型是否可以替代基类。

### 违反 LSP

`Member`类是抽象的，它代表了我们国际象棋俱乐部所有成员的基类。

```java
public abstract class Member {
    private final String name;
    public Member(String name) {
        this.name = name;
    }
    public abstract void joinTournament();
    public abstract void organizeTournament();
}
```

`PremiumMember`类可以加入国际象棋比赛，也可以组织这样的比赛。因此，它的实现非常简单。

```java
public class PremiumMember extends Member {
    public PremiumMember(String name) {
        super(name);
    }
    @Override
    public void joinTournament() {
        System.out.println("Premium member joins tournament");         
    }
    @Override
    public void organizeTournament() {
        System.out.println("Premium member organize 
            tournament");        
     }
}
```

`VipMember`类与`PremiumMember`类大致相同，因此我们可以跳过它，专注于`FreeMember`类。`FreeMember`类可以参加比赛，但不能组织比赛。这是我们需要在`organizeTournament()`方法中解决的问题。我们可以抛出一个带有有意义消息的异常，或者我们可以显示一条消息，如下所示：

```java
public class FreeMember extends Member {
    public FreeMember(String name) {
        super(name);
    }
    @Override
    public void joinTournament() {
        System.out.println("Classic member joins tournament 
            ...");

    }
    // this method breaks Liskov's Substitution Principle
    @Override
    public void organizeTournament() {
        System.out.println("A free member cannot organize 
            tournaments");

    }
}
```

但是抛出异常或显示消息并不意味着我们遵循 LSP。由于免费会员无法组织比赛，因此它不能替代基类，因此它违反了 LSP。请查看以下会员列表：

```java
List<Member> members = List.of(
    new PremiumMember("Jack Hores"),
    new VipMember("Tom Johns"),
    new FreeMember("Martin Vilop")
); 
```

以下循环显示了我们的代码不符合 LSP，因为当`FreeMember`类必须替换`Member`类时，它无法完成其工作，因为`FreeMember`无法组织国际象棋比赛。

```java
for (Member member : members) {
    member.organizeTournament();
}
```

这种情况是一个停滞不前的问题。我们无法继续实现我们的应用程序。我们必须重新设计我们的解决方案，以获得符合 LSP 的代码。所以让我们这样做！

### 遵循 LSP

重构过程从定义两个接口开始，这两个接口用于分离两个操作，即加入和组织国际象棋比赛：

```java
public interface TournamentJoiner {
    public void joinTournament();
}
public interface TournamentOrganizer {
    public void organizeTournament();
}
```

接下来，抽象基类实现这两个接口如下：

```java
public abstract class Member 
    implements TournamentJoiner, TournamentOrganizer {
    private final String name;
    public Member(String name) {
        this.name = name;
    }  
}
```

`PremiumMember`和`VipMember`保持不变。它们扩展了`Member`基类。然而，`FreeMember`类不能组织比赛，因此不会扩展`Member`基类。它只会实现`TournamentJoiner`接口：

```java
public class FreeMember implements TournamentJoiner {
    private final String name;
    public FreeMember(String name) {
        this.name = name;
    }
    @Override
    public void joinTournament() {
        System.out.println("Free member joins tournament ...");
    }
}
```

现在，我们可以定义一个能够参加国际象棋比赛的成员列表如下：

```java
List<TournamentJoiner> members = List.of(
    new PremiumMember("Jack Hores"),
    new PremiumMember("Tom Johns"),
    new FreeMember("Martin Vilop")
);
```

循环此列表，并用每种类型的成员替换`TournamentJoiner`接口，可以正常工作并遵守 LSP：

```java
// this code respects LSP
for (TournamentJoiner member : members) {
    member.joinTournament();
}   
```

按照相同的逻辑，可以将能够组织国际象棋比赛的成员列表编写如下：

```java
List<TournamentOrganizer> members = List.of(
    new PremiumMember("Jack Hores"),
    new VipMember("Tom Johns")
);
```

`FreeMember`没有实现`TournamentOrganizer`接口。因此，它不能添加到此列表中。循环此列表，并用每种类型的成员替换`TournamentOrganizer`接口可以正常工作并遵守 LSP：

```java
// this code respects LSP
for (TournamentOrganizer member : members) {
    member.organizeTournament();
}
```

完成！现在我们有一个符合 LSP 的代码。完整的应用程序命名为*LiskovSubstitutionPrinciple*。接下来，让我们谈谈第四个 SOLID 原则，接口隔离原则。

## 什么是 I？

您应该在答案中封装的关键点如下：

+   I 代表**接口隔离原则**（ISP）。

+   I 代表*客户端不应被强制实现他们不会使用的不必要的方法*。

+   我将一个接口分割成两个或更多个接口，直到客户端不被强制实现他们不会使用的方法。

现在，我们可以如下呈现一个答案：

首先，SOLID 是前五个`Connection`接口的首字母缩写，它有三种方法：`connect()`，`socket()`和`http()`。客户端可能只想为通过 HTTP 的连接实现此接口。因此，他们不需要`socket()`方法。大多数情况下，客户端会将此方法留空，这是一个糟糕的设计。为了避免这种情况，只需将`Connection`接口拆分为两个接口；`SocketConnection`具有`socket()`方法，`HttpConnection`具有`http()`方法。这两个接口将扩展保留有共同方法`connect()`的`Connection`接口。

如果需要进一步的细节，那么您可以共享屏幕或使用纸和笔编写一个像这里呈现的示例。由于我们已经描述了前面的例子，让我们跳到关于违反 ISP 的部分。

### 违反 ISP

`Connection`接口定义了三种方法如下：

```java
public interface Connection {
    public void socket();
    public void http();
    public void connect();
}
```

`WwwPingConnection`是一个通过 HTTP 对不同网站进行 ping 的类；因此，它需要`http()`方法，但不需要`socket()`方法。请注意虚拟的`socket()`实现-由于`WwwPingConnection`实现了`Connection`，它被强制提供`socket()`方法的实现：

```java
public class WwwPingConnection implements Connection {
    private final String www;
    public WwwPingConnection(String www) {
        this.www = www;
    }
    @Override
    public void http() {
        System.out.println("Setup an HTTP connection to " 
            + www);
    }
    @Override
    public void connect() {
        System.out.println("Connect to " + www);
    }
    // this method breaks Interface Segregation Principle
    @Override
    public void socket() {
    }
}
```

在不需要的方法中具有空实现或抛出有意义的异常，比如`socket()`，是一个非常丑陋的解决方案。检查以下代码：

```java
WwwPingConnection www 
    = new WwwPingConnection 'www.yahoo.com');
www.socket(); // we can call this method!
www.connect();
```

我们期望从这段代码中获得什么？一个什么都不做的工作代码，或者由于没有 HTTP 端点而导致`connect()`方法引发的异常？或者，我们可以从`socket()`中抛出类型为*Socket is not supported!*的异常。那么，它为什么在这里？！因此，现在是时候重构代码以遵循 ISP 了。

### 遵循 ISP

为了遵守 ISP，我们需要分隔`Connection`接口。由于任何客户端都需要`connect()`方法，我们将其留在这个接口中：

```java
public interface Connection {
    public void connect();
}
```

`http()`和`socket()`方法分布在扩展`Connection`接口的两个单独的接口中，如下所示：

```java
public interface HttpConnection extends Connection {
    public void http();
}
public interface SocketConnection extends Connection {
    public void socket();
}
```

这次，`WwwPingConnection`类只能实现`HttpConnection`接口并使用`http()`方法：

```java
public class WwwPingConnection implements HttpConnection {
    private final String www;
    public WwwPingConnection(String www) {
        this.www = www;
    }
    @Override
    public void http() {
        System.out.println("Setup an HTTP connection to "
            + www);
    }
    @Override
    public void connect() {
        System.out.println("Connect to " + www);
    } 
}
```

完成！现在，代码遵循 ISP。完整的应用程序命名为*InterfaceSegregationPrinciple*。接下来，让我们谈谈最后一个 SOLID 原则，依赖倒置原则。

## 什么是 D？

您应该在答案中封装的关键点如下：

+   *D*代表**依赖倒置原则**（DIP）。

+   *D*代表*依赖于抽象，而不是具体实现*。

+   *D*支持使用抽象层来绑定具体模块，而不是依赖于其他具体模块。

+   *D*维持了具体模块的解耦。

现在，我们可以提出一个答案如下：

首先，SOLID 是 Robert C. Martin 提出的前五个**面向对象设计**（OOD）原则的首字母缩写，也被称为 Uncle Bob（*可选短语*）。*D*是 SOLID 原则中的最后一个原则，被称为**依赖倒置原则**（DIP）。这个原则代表*依赖于抽象，而不是具体实现*。这意味着我们应该*依赖于抽象层来绑定具体模块，而不是依赖于具体模块*。为了实现这一点，所有具体模块应该只暴露抽象。这样，具体模块允许扩展功能或在另一个具体模块中插入，同时保持具体模块的解耦。通常，高级具体模块和低级具体模块之间存在高耦合。

如果需要更多细节，你可以分享屏幕或使用纸和笔编写一个例子。

数据库 JDBC URL，`PostgreSQLJdbcUrl`，可以是一个低级模块，而连接到数据库的类可能代表一个高级模块，比如`ConnectToDatabase#connect()`。

### 打破 DIP

如果我们向`connect()`方法传递`PostgreSQLJdbcUrl`类型的参数，那么我们就违反了 DIP。让我们来看看`PostgreSQLJdbcUrl`和`ConnectToDatabase`的代码：

```java
public class PostgreSQLJdbcUrl {
    private final String dbName;
    public PostgreSQLJdbcUrl(String dbName) {
        this.dbName = dbName;
    }
    public String get() {
        return "jdbc:// ... " + this.dbName;
    }
}
public class ConnectToDatabase {
    public void connect(PostgreSQLJdbcUrl postgresql) {
        System.out.println("Connecting to "
            + postgresql.get());
    }
}
```

如果我们创建另一种类型的 JDBC URL（例如`MySQLJdbcUrl`），那么我们就不能使用之前的`connect(PostgreSQLJdbcUrl postgreSQL)`方法。因此，我们必须放弃对具体的依赖，创建对抽象的依赖。

### 遵循 DIP

抽象可以由一个接口表示，每种类型的 JDBC URL 都应该实现该接口：

```java
public interface JdbcUrl {
    public String get();
}
```

接下来，`PostgreSQLJdbcUrl`实现了`JdbcUrl`以返回特定于 PostgreSQL 数据库的 JDBC URL：

```java
public class PostgreSQLJdbcUrl implements JdbcUrl {
    private final String dbName;
    public PostgreSQLJdbcUrl(String dbName) {
        this.dbName = dbName;
    }
    @Override
    public String get() {
        return "jdbc:// ... " + this.dbName;
    }
}
```

以完全相同的方式，我们可以编写`MySQLJdbcUrl`、`OracleJdbcUrl`等。最后，`ConnectToDatabase#connect()`方法依赖于`JdbcUrl`抽象，因此它可以连接到实现了这个抽象的任何 JDBC URL：

```java
public class ConnectToDatabase {
    public void connect(JdbcUrl jdbcUrl) {
        System.out.println("Connecting to " + jdbcUrl.get());
    }
}
```

完成！完整的应用程序命名为*DependencyInversionPrinciple*。

到目前为止，我们已经涵盖了 OOP 的基本概念和流行的 SOLID 原则。如果你计划申请一个包括应用程序设计和架构的 Java 职位，那么建议你看看**通用责任分配软件原则**（GRASP）（[`en.wikipedia.org/wiki/GRASP_(object-oriented_design`](https://en.wikipedia.org/wiki/GRASP_(object-oriented_design)）。这在面试中并不是一个常见的话题，但你永远不知道！

接下来，我们将扫描一系列结合了这些概念的热门问题。现在你已经熟悉了**理解问题** | **提名关键点** | **回答**的技巧，我将只突出回答中的关键点，而不是事先提取它们作为一个列表。

# 与 OOP、SOLID 和 GOF 设计模式相关的热门问题

在这一部分，我们将解决一些更难的问题，这些问题需要对 OOP 概念、SOLID 设计原则和**四人帮**（GOF）设计模式有真正的理解。请注意，本书不涵盖 GOF 设计模式，但有很多专门讨论这个主题的优秀书籍和视频。我建议你尝试 Aseem Jain 的《用 Java 学习设计模式》（[`www.packtpub.com/application-development/learn-design-patterns-java-video`](https://www.packtpub.com/application-development/learn-design-patterns-java-video)）。

## 面向对象编程（Java）中的方法重写是什么？

方法重写是一种面向对象的编程技术，允许开发人员编写两个具有相同名称和签名但具有不同行为的方法（非静态，非私有和非最终）。在**继承**或**运行时多态**的情况下，可以使用方法重写。

在继承的情况下，我们在超类中有一个方法（称为被重写方法），并且我们在子类中重写它（称为重写方法）。在运行时多态中，我们在一个接口中有一个方法，实现这个接口的类正在重写这个方法。

Java 在运行时决定应该调用的实际方法，取决于对象的类型。方法重写支持灵活和可扩展的代码，换句话说，它支持以最小的代码更改添加新功能。

如果需要更多细节，那么可以列出管理方法重写的主要规则：

+   方法的名称和签名（包括相同的返回类型或子类型）在超类和子类中，或在接口和实现中是相同的。

+   我们不能在同一个类中重写一个方法（但我们可以在同一个类中重载它）。

+   我们不能重写`private`，`static`和`final`方法。

+   重写方法不能降低被重写方法的可访问性，但相反是可能的。

+   重写方法不能抛出比被重写方法抛出的检查异常更高的检查异常。

+   始终为重写方法使用`@Override`注解。

Java 中重写方法的示例可在本书附带的代码中找到，名称为 MethodOverriding。

## 在面向对象编程（Java）中，什么是方法重载？

方法重载是一种面向对象的编程技术，允许开发人员编写两个具有相同名称但不同签名和不同功能的方法（静态或非静态）。通过不同的签名，我们理解为不同数量的参数，不同类型的参数和/或参数列表的不同顺序。返回类型不是方法签名的一部分。因此，当两个方法具有相同的签名但不同的返回类型时，这不是方法重载的有效情况。因此，这是一种强大的技术，允许我们编写具有相同名称但具有不同输入的方法（静态或非静态）。编译器将重载的方法调用绑定到实际方法；因此，在运行时不进行绑定。方法重载的一个著名例子是`System.out.println()`。`println()`方法有几种重载的版本。

因此，有四条主要规则来管理方法重载：

+   通过更改方法签名来实现重载。

+   返回类型不是方法签名的一部分。

+   我们可以重载`private`，`static`和`final`方法。

+   我们可以在同一个类中重载一个方法（但不能在同一个类中重写它）。

如果需要更多细节，可以尝试编写一个示例。Java 中重载方法的示例可在本书附带的代码中找到，名称为 MethodOverloading。

重要提示

除了前面提到的两个问题，您可能需要回答一些其他相关的问题，包括*什么规则管理方法的重载和重写*（见上文）？*方法重载和重写的主要区别是什么*（见上文）？*我们可以重写静态或私有方法吗*（简短的答案是*不可以*，见上文）？*我们可以重写 final 方法吗*（简短的答案是*不可以*，见上文）？*我们可以重载静态方法吗*（简短的答案是*可以*，见上文）？*我们可以改变重写方法的参数列表吗*（简短的答案是*不可以*，见上文）？因此，建议提取和准备这些问题的答案。所有所需的信息都可以在前面的部分找到。此外，注意诸如*只有通过 final 修饰符才能防止重写方法*这样的问题。这种措辞旨在混淆候选人，因为答案需要概述所涉及的概念。这里的答案可以表述为*这是不正确的，因为我们也可以通过将其标记为私有或静态来防止重写方法。这样的方法不能被重写*。

接下来，让我们检查几个与重写和重载方法相关的其他问题。

## 在 Java 中，协变方法重写是什么？

协变方法重写是 Java 5 引入的一个不太知名的特性。通过这个特性，*重写方法可以返回其实际返回类型的子类型*。这意味着重写方法的客户端不需要对返回类型进行显式类型转换。例如，Java 的`clone()`方法返回`Object`。这意味着，当我们重写这个方法返回一个克隆时，我们得到一个`Object`，必须显式转换为我们需要的`Object`的实际子类。然而，如果我们利用 Java 5 的协变方法重写特性，那么重写的`clone()`方法可以直接返回所需的子类，而不是`Object`。

几乎总是，这样的问题需要一个示例作为答案的一部分，因此让我们考虑实现`Cloneable`接口的`Rectangle`类。`clone()`方法可以返回`Rectangle`而不是`Object`，如下所示：

```java
public class Rectangle implements Cloneable {
    ...  
    @Override
    protected Rectangle clone() 
            throws CloneNotSupportedException {
        Rectangle clone = (Rectangle) super.clone();
        return clone;
    }
}
```

调用`clone()`方法不需要显式转换：

```java
Rectangle r = new Rectangle(4, 3);
Rectangle clone = r.clone();
```

完整的应用程序名为*CovariantMethodOverriding*。注意一些关于协变方法重写的间接问题。例如，可以这样表述：*我们可以在重写时修改方法的返回类型为子类吗？* 对于这个问题的答案与*Java 中的协变方法重写是什么？*相同，在这里讨论过。

重要提示

了解针对 Java 的一些不太知名特性的问题的答案可能是面试中的一个重要加分项。这向面试官表明您具有深入的知识水平，并且您对 Java 的发展了如指掌。如果您需要通过大量示例和最少理论来快速了解所有 JDK 8 到 JDK 13 的功能，那么您一定会喜欢我出版的名为*Java 编程问题*的书，由 Packt 出版（[packtpub.com/au/programming/java-coding-problems](https://www.packtpub.com/product/java-coding-problems/9781789801415)）。

## 在重写和重载方法方面，主要的限制是什么？

首先，让我们讨论重写方法。*如果我们谈论未经检查的异常，那么我们必须说在重写方法中使用它们没有限制*。这样的方法可以抛出未经检查的异常，因此任何`RuntimeException`。另一方面，*在检查异常的情况下，重写方法只能抛出被重写方法的检查异常或该检查异常的子类*。换句话说，重写方法不能抛出比被重写方法抛出的检查异常范围更广的检查异常。例如，如果被重写的方法抛出`SQLException`，那么重写方法可以抛出子类，如`BatchUpdateException`，但不能抛出超类，如`Exception`。

其次，让我们讨论重载方法。*这样的方法不会施加任何限制*。这意味着我们可以根据需要修改`throw`子句。

重要提示

注意那些以*主要是什么...？你能列举某些...吗？你能提名...吗？你能强调...吗？*等方式措辞的问题。通常，当问题包含*主要，某些，提名*和*强调*等词时，面试官期望得到一个清晰简洁的答案，应该听起来像一个项目列表。回答这类问题的最佳实践是直接进入回答并将每个项目列举为一个简洁而有意义的陈述。在给出预期答案之前，不要犯常见错误，即着手讲述所涉及的概念的故事或论文。面试官希望看到你的综合和整理能力，并在检查你的知识水平的同时提取本质。

如果需要更多细节，那么你可以编写一个示例，就像这本书中捆绑的代码一样。考虑检查*OverridingException*和*OverloadingException*应用程序。现在，让我们继续看一些更多的问题。

## 如何从子类重写的方法中调用超类重写的方法？

*我们可以通过 Java 的* `super` *关键字从子类重写的方法中调用超类重写的方法*。例如，考虑一个包含方法`foo()`的超类`A`，以及一个名为`B`的`A`子类。如果我们在子类`B`中重写`foo()`方法，并且我们从重写方法`B#foo()`中调用`super.foo()`，那么我们调用被重写的方法`A#foo()`。

## 我们能重写或重载 main()方法吗？

我们必须记住`main()`方法是静态的。这意味着我们可以对其进行重载。但是，我们不能对其进行重写，因为静态方法在编译时解析，而我们可以重写的方法在运行时根据对象类型解析。

## 我们能将非静态方法重写为静态方法吗？

不。*我们不能将非静态方法重写为静态方法*。此外，反之亦然也不可能。两者都会导致编译错误。

重要提示

像前面提到的最后两个问题一样直截了当的问题，值得一个简短而简洁的答案。面试官触发这样的闪光灯问题来衡量你分析情况并做出决定的能力。主要是，答案是简短的，但你需要一些时间来说*是*或*否*。这类问题并不具有很高的分数，但如果你不知道答案，可能会产生重大的负面影响。如果你知道答案，面试官可能会在心里说*好吧，这本来就是一个容易的问题！*但是，如果你不知道答案，他可能会说*他错过了一个简单的问题！她/他的基础知识有严重缺陷*。

接下来，让我们看一些与其他面向对象编程概念相关的更多问题。

我们能在 Java 接口中有一个非抽象方法吗？

*直到 Java 8，我们不能在 Java 接口中有非抽象方法*。接口中的所有方法都是隐式公共和抽象的。然而，从 Java 8 开始，我们可以向接口添加新类型的方法。*从实际角度来看，从 Java 8 开始，我们可以直接在接口中添加具体实现的方法。这可以通过使用* `default` *和* `static` *关键字来实现。* `default` *关键字是在 Java 8 中引入的，用于在接口中包含称为* `static` *方法的方法，接口中的* `static` *方法与默认方法非常相似，唯一的区别是我们不能在实现这些接口的类中重写* `static` *方法*。由于`static`方法不绑定到对象，因此可以通过使用接口名称加上点和方法名称来调用它们。此外，`static`方法可以在其他`default`和`static`方法中调用。

如果需要更多细节，那么您可以尝试编写一个示例。考虑到我们有一个用于塑造蒸汽车辆的接口（这是一种旧的汽车类型，与旧代码完全相同）：

```java
public interface Vehicle {
    public void speedUp();
    public void slowDown();    
}
```

显然，通过以下`SteamCar`类已经建造了不同种类的蒸汽车：

```java
public class SteamCar implements Vehicle {
    private String name;
    // constructor and getter omitted for brevity
    @Override
    public void speedUp() {
        System.out.println("Speed up the steam car ...");
    }
    @Override
    public void slowDown() {
        System.out.println("Slow down the steam car ...");
    }
}
```

由于`SteamCar`类实现了`Vehicle`接口，它重写了`speedUp()`和`slowDown()`方法。过了一段时间，汽油车被发明出来，人们开始关心马力和燃油消耗。因此，我们的代码必须发展以支持汽油车。为了计算消耗水平，我们可以通过添加`computeConsumption()`默认方法来发展`Vehicle`接口，如下所示：

```java
public interface Vehicle {
    public void speedUp();
    public void slowDown();
    default double computeConsumption(int fuel, 
            int distance, int horsePower) {        
        // simulate the computation 
        return Math.random() * 10d;
    }        
}
```

发展`Vehicle`接口不会破坏`SteamCar`的兼容性。此外，电动汽车已经被发明。计算电动汽车的消耗与汽油汽车的情况不同，但公式依赖于相同的术语：燃料、距离和马力。这意味着`ElectricCar`将重写`computeConsumption()`，如下所示：

```java
public class ElectricCar implements Vehicle {
    private String name;
    private int horsePower;
    // constructor and getters omitted for brevity
    @Override
    public void speedUp() {
        System.out.println("Speed up the electric car ...");
    }
    @Override
    public void slowDown() {
        System.out.println("Slow down the electric car ...");
    }
    @Override
    public double computeConsumption(int fuel, 
            int distance, int horsePower) {
        // simulate the computation
        return Math.random()*60d / Math.pow(Math.random(), 3);
    }     
}
```

因此，我们可以重写`default`方法，或者我们可以使用隐式实现。最后，我们必须为我们的接口添加描述，因为现在它服务于蒸汽、汽油和电动汽车。我们可以通过为`Vehicle`添加一个名为`description()`的`static`方法来实现这一点，如下所示：

```java
public interface Vehicle {
    public void speedUp();
    public void slowDown();
    default double computeConsumption(int fuel, 
        int distance, int horsePower) {        
        return Math.random() * 10d;
    }
    static void description() {
        System.out.println("This interface control
            steam, petrol and electric cars");
    }
}
```

这个`static`方法不绑定到任何类型的汽车，可以直接通过`Vehicle.description()`调用。完整的代码名为*Java8DefaultStaticMethods*。

接下来，让我们继续其他问题。到目前为止，您应该对“理解问题”|“提名关键点”|“回答”技术非常熟悉，所以我将停止突出显示关键点。从现在开始，找到它们就是你的工作了。

## 接口和抽象类之间的主要区别是什么？

在 Java 8 接口和抽象类之间的差异中，我们可以提到抽象类可以有构造函数，而接口不支持构造函数。因此，抽象类可以有状态，而接口不能有状态。此外，接口仍然是完全抽象的第一公民，其主要目的是被实现，而抽象类是为了部分抽象。接口仍然被设计为针对完全抽象的事物，它们本身不做任何事情，但是指定了如何在实现中工作的合同。默认方法代表了一种方法，可以在不影响客户端代码和不改变状态的情况下向接口添加附加功能。它们不应该用于其他目的。换句话说，另一个差异在于，拥有没有抽象方法的抽象类是完全可以的，但是只有默认方法的接口是一种反模式。这意味着我们已经创建了接口作为实用类的替代品。这样，我们就打败了接口的主要目的，即被实现。

重要说明

当你不得不列举两个概念之间的许多差异或相似之处时，注意限制你的答案在问题确定的坐标内。例如，在前面的问题中，不要说接口支持多重继承而抽象类不支持这一点。这是接口和类之间的一般变化，而不是特别是 Java 8 接口和抽象类之间的变化。

## 抽象类和接口之间的主要区别是什么？

直到 Java 8，抽象类和接口的主要区别在于抽象类可以包含非抽象方法，而接口不能包含这样的方法。从 Java 8 开始，主要区别在于抽象类可以有构造函数和状态，而接口两者都不能有。

## 可以有一个没有抽象方法的抽象类吗？

是的，我们可以。通过向类添加`abstract`关键字，它变成了抽象类。它不能被实例化，但可以有构造函数和只有非抽象方法。

## 我们可以同时拥有一个既是抽象又是最终的类吗？

最终类不能被子类化或继承。抽象类意味着要被扩展才能使用。因此，最终和抽象是相反的概念。这意味着它们不能同时应用于同一个类。编译器会报错。

## 多态、重写和重载之间有什么区别？

在这个问题的背景下，重载技术被称为**编译时多态**，而重写技术被称为**运行时多态**。重载涉及使用静态（或早期）绑定，而重写使用动态（或晚期）绑定。

接下来的两个问题构成了这个问题的附加部分，但它们也可以作为独立的问题来表述。

什么是绑定操作？

绑定操作确定由于代码行中的引用而调用的方法（或变量）。换句话说，将方法调用与方法体关联的过程称为绑定操作。一些引用在编译时解析，而其他引用在运行时解析。在运行时解析的引用取决于对象的类型。在编译时解析的引用称为静态绑定操作，而在运行时解析的引用称为动态绑定操作。

## 静态绑定和动态绑定之间的主要区别是什么？

首先，静态绑定发生在编译时，而动态绑定发生在运行时。要考虑的第二件事是，私有、静态和最终成员（方法和变量）使用静态绑定，而虚方法根据对象类型在运行时进行绑定。换句话说，静态绑定是通过`Type`（Java 中的类）信息实现的，而动态绑定是通过`Object`实现的，这意味着依赖静态绑定的方法与对象无关，而是在`Type`（Java 中的类）上调用的，而依赖动态绑定的方法与`Object`相关。依赖静态绑定的方法的执行速度比依赖动态绑定的方法的执行速度稍快。静态和动态绑定也用于多态。静态绑定用于编译时多态（重载方法），而动态绑定用于运行时多态（重写方法）。静态绑定在编译时增加了性能开销，而动态绑定在运行时增加了性能开销，这意味着静态绑定更可取。

## 在 Java 中什么是方法隐藏？

方法隐藏是特定于静态方法的。更确切地说，如果我们在超类和子类中声明具有相同签名和名称的两个静态方法，那么它们将互相隐藏。从超类调用方法将调用超类的静态方法，从子类调用相同的方法将调用子类的静态方法。隐藏与覆盖不同，因为静态方法不能是多态的。

如果需要更多细节，你可以写一个例子。考虑`Vehicle`超类具有`move()`静态方法：

```java
public class Vehicle {
    public static void move() {
        System.out.println("Moving a vehicle");
    }
}
```

现在，考虑`Car`子类具有相同的静态方法：

```java
public class Car extends Vehicle {
    // this method hides Vehicle#move()
    public static void move() {
        System.out.println("Moving a car");
    }
}
```

现在，让我们从`main()`方法中调用这两个静态方法：

```java
public static void main(String[] args) {
    Vehicle.move(); // call Vehicle#move()
    Car.move();     // call Car#move()
}
```

输出显示这两个静态方法互相隐藏：

```java
Moving a vehicle
Moving a car
```

注意我们通过类名调用静态方法。在实例上调用静态方法是非常糟糕的做法，所以在面试中要避免这样做！

## 我们可以在 Java 中编写虚方法吗？

是的，我们可以！实际上，在 Java 中，所有非静态方法默认都是虚方法。我们可以通过使用`private`和/或`final`关键字标记来编写非虚方法。换句话说，可以继承以实现多态行为的方法是虚方法。或者，如果我们颠倒这个说法的逻辑，那些不能被继承（标记为`private`）和不能被覆盖（标记为`final`）的方法是非虚方法。

## 多态和抽象之间有什么区别？

抽象和多态代表两个相互依存的基本面向对象的概念。抽象允许开发人员设计可重用和可定制的通用解决方案，而多态允许开发人员推迟在运行时选择应该执行的代码。虽然抽象是通过接口和抽象类实现的，多态依赖于覆盖和重载技术。

## 你认为重载是实现多态的一种方法吗？

这是一个有争议的话题。有些人不认为重载是多态；因此，他们不接受编译时多态的概念。这些声音认为，唯一的覆盖方法才是真正的多态。这种说法背后的论点是，只有覆盖才允许代码根据运行时条件而表现出不同的行为。换句话说，表现多态行为是方法覆盖的特权。我认为只要我们理解重载和覆盖的前提条件，我们也就理解了这两种变体如何维持多态行为。

重要提示

处理有争议的话题的问题是微妙且难以正确处理的。因此，最好直接跳入答案，陈述*这是一个有争议的话题*。当然，面试官也对听到你的观点感兴趣，但他会很高兴看到你了解事情的两面。作为一个经验法则，尽量客观地回答问题，不要以激进的方式或者缺乏论据的方式处理问题的一面。有争议的事情毕竟还是有争议的，这不是揭开它们的神秘面纱的合适时间和地点。

好的，现在让我们继续一些基于 SOLID 原则和著名且不可或缺的**四人帮**（GOF）设计模式的问题。请注意，本书不涵盖 GOF 设计模式，但有很多专门讨论这个话题的优秀书籍和视频。我建议你尝试*Aseem Jain*的*Learn Design Patterns with Java*（[`www.packtpub.com/application-development/learn-design-patterns-java-video)`](https://www.packtpub.com/application-development/learn-design-patterns-java-video)）。

## 哪个面向对象的概念服务于装饰者设计模式？

服务装饰者设计模式的面向对象编程概念是**组合**。通过这个面向对象编程概念，装饰者设计模式在不修改原始类的情况下提供新功能。

## 单例设计模式应该在什么时候使用？

单例设计模式似乎是在我们只需要一个类的应用级（全局）实例时的正确选择。然而，应该谨慎使用单例，因为它增加了类之间的耦合，并且在开发、测试和调试过程中可能成为瓶颈。正如著名的《Effective Java》所指出的，使用 Java 枚举是实现这种模式的最佳方式。在全局配置（例如日志记录器、`java.lang.Runtime`）、硬件访问、数据库连接等方面，依赖单例模式是一种常见情况。

重要提示

每当可以引用或提及著名参考资料时，请这样做。

## 策略和状态设计模式之间有什么区别？

状态设计模式旨在根据*状态*执行某些操作（在不更改类的情况下，在不同*状态*下展示某些行为）。另一方面，策略设计模式旨在用于在不修改使用它的代码的情况下在一系列算法之间进行切换（客户端通过组合和运行时委托可互换地使用算法）。此外，在状态中，我们有清晰的*状态*转换顺序（流程是通过将每个*状态*链接到另一个*状态*来创建的），而在策略中，客户端可以以任何顺序选择它想要的算法。例如，状态模式可以定义发送包裹给客户的*状态*。

包裹从*有序状态*开始，然后继续到*已交付状态*，依此类推，直到通过每个*状态*并在客户端*接收*包裹时达到最终*状态*。另一方面，策略模式定义了完成每个*状态*的不同策略（例如，我们可能有不同的交付包裹策略）。

## 代理和装饰者模式之间有什么区别？

代理设计模式对于提供对某物的访问控制网关非常有用。通常，该模式创建代理对象，代替真实对象。对真实对象的每个请求都必须通过代理对象，代理对象决定如何何时将其转发给真实对象。装饰者设计模式从不创建对象，它只是在运行时用新功能装饰现有对象。虽然链接代理不是一个可取的做法，但以一定顺序链接装饰者可以以正确的方式利用这种模式。例如，代理模式可以表示互联网的代理服务器，而装饰者模式可以用于用不同的自定义设置装饰代理服务器。

## 外观和装饰者模式之间有什么区别？

装饰者设计模式旨在为对象添加新功能（换句话说，装饰对象），而外观设计模式根本不添加新功能。它只是外观现有功能（隐藏系统的复杂性），并通过向客户端暴露的“友好界面”在幕后调用它们。外观模式可以暴露一个简单的接口，调用各个组件来完成复杂的任务。例如，装饰者模式可以用来通过用发动机、变速箱等装饰底盘来建造汽车，而外观模式可以通过暴露一个简单的接口来隐藏建造汽车的复杂性，以便命令了解建造过程细节的工业机器人。

模板方法和策略模式之间的关键区别是什么？

模板方法和策略模式将特定领域的算法集合封装成对象，但它们的实现方式并不相同。关键区别在于策略模式旨在根据需求在运行时在不同策略（算法）之间做出决定，而模板方法模式旨在遵循算法的固定骨架（预定义的步骤序列）实现。一些步骤是固定的，而其他步骤可以根据不同的用途进行修改。例如，策略模式可以在不同的支付策略之间做出决定（例如信用卡或 PayPal），而模板方法可以描述使用特定策略进行支付的预定义步骤序列（例如，通过 PayPal 进行支付需要固定的步骤序列）。

## 生成器和工厂模式之间的关键区别是什么？

工厂模式在单个方法调用中创建对象。我们必须在此调用中传递所有必要的参数，工厂将返回对象（通常通过调用构造函数）。另一方面，生成器模式旨在通过一系列 setter 方法构建复杂对象，允许我们塑造任何组合的参数。在链的末尾，生成器方法公开了一个`build()`方法，表示参数列表已设置，现在是构建对象的时候了。换句话说，工厂充当构造函数的包装器，而生成器更加精细，充当您可能想要传递到构造函数的所有可能参数的包装器。通过生成器，我们避免了望远镜构造函数用于公开所有可能的参数组合。例如，回想一下`Book`对象。一本书由一些固定参数来描述，例如作者、标题、ISBN 和格式。在创建书籍时，您很可能不会在这些参数的数量上纠结，因此工厂模式将是适合创建书籍的选择。但是`Server`对象呢？嗯，服务器是一个具有大量可选参数的复杂对象，因此生成器模式在这里更加合适，甚至是工厂在内部依赖于生成器的这些模式的组合。

## 适配器和桥接模式之间的关键区别是什么？

适配器模式致力于提供现有代码（例如第三方代码）与新系统或接口之间的兼容性。另一方面，桥接模式是提前实现的，旨在将抽象与实现解耦，以避免大量的类。因此，适配器致力于在设计后提供事物之间的兼容性（可以想象为*A 来自 After*），而桥接是提前构建的，以使抽象和实现可以独立变化（可以想象为*B 来自 Before*）。适配器充当`ReadJsonRequest`和`ReadXmlRequest`，它们能够从多个设备读取，例如`D1`、`D2`和`D3`。`D1`和`D2`只产生 JSON 请求，而`D3`只产生 XML 请求。通过适配器，我们可以在 JSON 和 XML 之间进行转换，这意味着这两个类可以与所有三个设备进行通信。另一方面，通过桥接模式，我们可以避免最终产生许多类，例如`ReadXMLRequestD1`、`ReadXMLRequestD2`、`ReadXMLRequestD3`、`ReadJsonRequestD1`、`ReadJsonRequestD2`和`ReadJsonRequestD3`。

我们可以继续比较设计模式，直到完成所有可能的组合。最后几个问题涵盖了类型**设计模式 1 与设计模式 2**的最受欢迎的问题。强烈建议您挑战自己，尝试识别两种或更多给定设计模式之间的相似之处和不同之处。大多数情况下，这些问题使用来自同一类别的两种设计模式（例如，两种结构或两种创建模式），但它们也可以来自不同的类别。在这种情况下，这是面试官期望听到的第一句话。因此，在这种情况下，首先说出每个涉及的设计模式属于哪个类别。

请注意，我们跳过了所有简单问题，比如*什么是接口？什么是抽象类？*等等。通常，这类问题是要避免的，因为它们并不能说明您的理解水平，更多的是背诵一些定义。面试官可以问*抽象类和接口的主要区别是什么？*，他可以从您的回答中推断出您是否知道接口和抽象类是什么。始终要准备好举例。无法举例说明严重缺乏对事物本质的理解。

拥有 OOP 知识只是问题的一半。另一半是具有将这些知识转化为设计应用程序的愿景和灵活性。这就是我们将在接下来的 10 个示例中做的事情。请记住，我们专注于设计，而不是实现。

# 编码挑战

接下来，我们将解决关于面向对象编程的几个编码挑战。对于每个问题，我们将遵循*第五章**中的图 5.2*，*如何处理编码挑战*。主要是，我们将首先向面试官提出一个问题，比如*设计约束是什么？*通常，围绕 OOD 的编码挑战是以一种一般的方式由面试官表达的。这是故意这样做的，以便让您询问有关设计约束的细节。

一旦我们清楚地了解了约束条件，我们可以尝试一个示例（可以是草图、逐步运行时可视化、项目列表等）。然后，我们找出算法/解决方案，最后，我们提供设计骨架。

## 示例 1：自动唱机

**亚马逊**，**谷歌**

**问题**：设计自动唱机音乐机的主要类。

**要问的问题**：自动唱机播放什么-CD、MP3？我应该设计什么-自动唱机建造过程，它是如何工作的，还是其他什么？是免费的自动唱机还是需要钱？

**面试官**：免费的自动唱机只播放 CD 吗？设计它的主要功能，因此设计它是如何工作的。

**解决方案**：为了理解我们的设计应该涉及哪些类，我们可以尝试想象一台自动唱机并确定其主要部分和功能。沿着这里的线条画一个图表也有助于面试官了解您的思维方式。我建议您始终采取以书面形式将问题可视化的方法-草图是一个完美的开始：

![图 6.1 – 自动唱机](img/Figure_6.1_B15403.jpg)

图 6.1 – 自动唱机

因此，我们可以确定自动唱机的两个主要部分：CD 播放器（或特定的自动唱机播放机制）和用户命令的接口。CD 播放器能够管理播放列表并播放这些歌曲。我们可以将命令接口想象为一个由自动唱机实现的 Java 接口，如下面的代码所示。除了以下代码，您还可以使用这里的 UML 图：[`github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/blob/master/Chapter06/Jukebox/JukeboxUML.png`](https://github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/blob/master/Chapter06/Jukebox/JukeboxUML.png)

```java
public interface Selector {
    public void nextSongBtn();
    public void prevSongBtn();
    public void addSongToPlaylistBtn(Song song);
    public void removeSongFromPlaylistBtn(Song song);
    public void shuffleBtn();
}
public class Jukebox implements Selector {
    private final CDPlayer cdPlayer;
    public Jukebox(CDPlayer cdPlayer) {
        this.cdPlayer = cdPlayer;        
    }            
    @Override
    public void nextSongBtn() {...}
    // rest of Selector methods omitted for brevity
}
```

`CDPlayer`是点唱机的核心。通过`Selector`，我们控制`CDPlayer`的行为。`CDPlayer`必须能够访问可用的 CD 和播放列表：

```java
public class CDPlayer {
    private CD cd;
    private final Set<CD> cds;
    private final Playlist playlist;
    public CDPlayer(Playlist playlist, Set<CD> cds) {
        this.playlist = playlist;
        this.cds = cds;
    }                
    protected void playNextSong() {...}
    protected void playPrevSong() {...}   
    protected void addCD(CD cd) {...}
    protected void removeCD(CD cd) {...}
    // getters omitted for brevity
}
```

接下来，`Playlist`管理一个`Song`列表：

```java
public class Playlist {
    private Song song;
    private final List<Song> songs; // or Queue
    public Playlist(List<Song> songs) {
        this.songs = songs;
    }   
    public Playlist(Song song, List<Song> songs) {
        this.song = song;
        this.songs = songs;
    }        
    protected void addSong(Song song) {...}
    protected void removeSong(Song song) {...}
    protected void shuffle() {...}    
    protected Song getNextSong() {...};
    protected Song getPrevSong() {...};
    // setters and getters omitted for brevity
}
```

`User`、`CD`和`Song`类暂时被跳过，但你可以在名为*点唱机*的完整应用程序中找到它们。这种问题可以以多种方式实现，所以也可以尝试你自己的设计。

## 示例 2：自动售货机

**亚马逊**，**谷歌**，**Adobe**

**问题**：设计支持典型自动售货机功能实现的主要类。

**要问的问题**：这是一个带有不同类型硬币和物品的自动售货机吗？它暴露了功能，比如检查物品价格、购买物品、退款和重置吗？

**面试官**：是的，确实！对于硬币，你可以考虑一分硬币、五分硬币、一角硬币和一美元硬币。

**解决方案**：为了理解我们的设计应该涉及哪些类，我们可以尝试勾画一个自动售货机。有各种各样的自动售货机类型。简单地勾画一个你知道的（比如下图中的那种）：

![图 6.2 – 自动售货机](img/Figure_6.2_B15403.jpg)

图 6.2 – 自动售货机

首先，我们立即注意到物品和硬币是 Java 枚举的良好候选。我们有四种硬币和几种物品，所以我们可以编写两个 Java 枚举如下。除了以下代码，你还可以使用这里的 UML 图表：[`github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/blob/master/Chapter06/VendingMachine/VendingMachineUML.png`](https://github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/blob/master/Chapter06/VendingMachine/VendingMachineUML.png)

```java
public enum Coin {
    PENNY(1), NICKEL(5), DIME(10), QUARTER(25);
    ...
}
public enum Item {
    SKITTLES("Skittles", 15), TWIX("Twix", 35) ...
    ...
}
```

自动售货机需要一个内部库存来跟踪物品和硬币的状态。我们可以将其通用地塑造如下：

```java
public final class Inventory<T> {
    private Map<T, Integer> inventory = new HashMap<>();
    protected int getQuantity(T item) {...}
    protected boolean hasItem(T item) {...}
    protected void clear() {...}    
    protected void add(T item) {...}
    protected void put(T item, int quantity) {...}
    protected void deduct(T item) {...}
}
```

接下来，我们可以关注客户用来与自动售货机交互的按钮。正如你在前面的例子中看到的，将这些按钮提取到一个接口中是常见做法，如下所示：

```java
public interface Selector {
    public int checkPriceBtn(Item item);
    public void insertCoinBtn(Coin coin);
    public Map<Item, List<Coin>> buyBtn();
    public List<Coin> refundBtn();
    public void resetBtn();    
}
```

最后，自动售货机可以被塑造成实现`Selector`接口并提供一堆用于完成内部任务的私有方法：

```java
public class VendingMachine implements Selector {
    private final Inventory<Coin> coinInventory
        = new Inventory<>();
    private final Inventory<Item> itemInventory
        = new Inventory<>();
  private int totalSales;
    private int currentBalance;
    private Item currentItem;
    public VendingMachine() {
        initMachine();
    }   
    private void initMachine() {
        System.out.println("Initializing the
            vending machine with coins and items ...");
    }
    // override Selector methods omitted for brevity
}
```

完整的应用程序名为*自动售货机*。通过遵循前面提到的两个例子，你可以尝试设计一个 ATM、洗衣机和类似的东西。

## 示例 3：一副卡牌

**亚马逊**，**谷歌**，**Adobe**，**微软**

**问题**：设计一个通用卡牌组的主要类。

**要问的问题**：由于卡可以是几乎任何东西，你能定义*通用*吗？

**面试官**：一张卡由一个符号（花色）和一个点数来描述。例如，想象一副标准的 52 张卡牌组。

**解决方案**：为了理解我们的设计应该涉及哪些类，我们可以快速勾画一个标准 52 张卡牌组的卡牌和一副卡牌，如图 6.3 所示：

![图 6.3 – 一副卡牌](img/Figure_6.3_B15403.jpg)

图 6.3 – 一副卡牌

由于每张卡都有花色和点数，我们将需要一个封装这些字段的类。让我们称这个类为`StandardCard`。`StandardCard`的花色包括*黑桃，红心，方块*或*梅花*，因此这个花色是 Java 枚举的一个很好的候选。`StandardCard`的点数可以在 1 到 13 之间。

一张卡可以独立存在，也可以是一副卡牌的一部分。多张卡组成一副卡牌（例如，一副标准的 52 张卡牌组形成一副卡牌）。一副卡牌中的卡的数量通常是可能的花色和点数的笛卡尔积（例如，4 种花色 x 13 个点数 = 52 张卡）。因此，52 个`StandardCard`对象形成了`StandardPack`。

最后，一副牌应该是一个能够执行一些与这个“标准包”相关的操作的类。例如，一副牌可以洗牌，可以发牌或发一张牌，等等。这意味着还需要一个`Deck`类。

到目前为止，我们已经确定了一个 Java 的`enum`和`StandardCard`、`StandardPack`和`Deck`类。如果我们添加了所需的抽象层，以避免这些具体层之间的高耦合，那么我们就得到了以下的实现。除了以下代码，您还可以使用这里的 UML 图：[`github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/blob/master/Chapter06/DeckOfCards/DeckOfCardsUML.png`](https://github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/blob/master/Chapter06/DeckOfCards/DeckOfCardsUML.png)

+   对于标准牌实现：

```java
public enum StandardSuit {   
    SPADES, HEARTS, DIAMONDS, CLUBS;
}
public abstract class Card {
    private final Enum suit;
    private final int value;
    private boolean available = Boolean.TRUE;
    public Card(Enum suit, int value) {
        this.suit = suit;
        this.value = value;
    }
    // code omitted for brevity
}
public class StandardCard extends Card {
    private static final int MIN_VALUE = 1;
    private static final int MAX_VALUE = 13;
    public StandardCard(StandardSuit suit, int value) {
        super(suit, value);
    }
    // code omitted for brevity
}
```

+   标准牌组实现提供以下代码：

```java
public abstract class Pack<T extends Card> {
    private List<T> cards;
    protected abstract List<T> build();
  public int packSize() {
        return cards.size();
    }
    public List<T> getCards() {
        return new ArrayList<>(cards);
    }
    protected void setCards(List<T> cards) {
        this.cards = cards;
    }
}
public final class StandardPack extends Pack {
    public StandardPack() {
        super.setCards(build());
    }
    @Override
    protected List<StandardCard> build() {
        List<StandardCard> cards = new ArrayList<>();
        // code omitted for brevity        
        return cards;
    }
}
```

+   牌组实现提供以下内容：

```java
public class Deck<T extends Card> implements Iterable<T> {
    private final List<T> cards;
    public Deck(Pack pack) {
        this.cards = pack.getCards();
    }
    public void shuffle() {...}
    public List<T> dealHand(int numberOfCards) {...}
    public T dealCard() {...}
    public int remainingCards() {...}
    public void removeCards(List<T> cards) {...}
    @Override
    public Iterator<T> iterator() {...}
}
```

代码的演示可以快速写成如下：

```java
// create a single classical card
Card sevenHeart = new StandardCard(StandardSuit.HEARTS, 7);       
// create a complete deck of standards cards      
Pack cp = new StandardPack();                   
Deck deck = new Deck(cp);
System.out.println("Remaining cards: " 
    + deck.remainingCards());
```

此外，您可以通过扩展`Card`和`Pack`类轻松添加更多类型的卡。完整的代码名为*DeckOfCards*。

## 示例 4：停车场

**亚马逊**，**谷歌**，**Adobe**，**微软**

**问题**：设计停车场的主要类。

**需要询问的问题**：这是单层停车场还是多层停车场？所有停车位是否相同？我们应该停放什么类型的车辆？这是免费停车吗？我们使用停车票吗？

**面试官**：这是一个同步自动多层免费停车场。所有停车位大小相同，但我们期望有汽车（需要 1 个停车位）、货车（需要 2 个停车位）和卡车（需要 5 个停车位）。其他类型的车辆应该可以在不修改代码的情况下添加。系统会释放一个停车票，以便以后用于取车。但是，如果司机只提供车辆信息（假设丢失了停车票），系统仍然应该能够工作并在停车场中找到车辆并将其取出。

**解决方案**：为了了解我们的设计应该涉及哪些类，我们可以快速勾画一个停车场，以识别主要的参与者和行为，如图 6.4 所示：

![图 6.4 - 停车场](img/Figure_6.4_B15403.jpg)

图 6.4 - 停车场

该图表显示了两个主要的参与者：停车场和自动停车系统。

首先，让我们专注于停车场。停车场的主要目的是停放车辆；因此，我们需要确定可接受的车辆（汽车、货车和卡车）。这看起来像是一个抽象类（`Vehicle`）和三个子类（`Car`、`Van`和`Truck`）的典型情况。但这并不是真的！司机提供有关他们的车辆的信息。他们并没有真正将车辆（对象）推入停车系统，因此我们的系统不需要为汽车、货车、卡车等专门的对象。从停车场的角度来看。它需要车辆牌照和停车所需的空闲车位。它不关心货车或卡车的特征。因此，我们可以将`Vehicle`塑造如下。除了以下代码，您还可以使用这里的 UML 图：[`github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/blob/master/Chapter06/ParkingLot/ParkingLotUML.png`](https://github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/blob/master/Chapter06/ParkingLot/ParkingLotUML.png)

```java
public enum VehicleType {
    CAR(1), VAN(2), TRUCK(5);
}
public class Vehicle {
    private final String licensePlate;
    private final int spotsNeeded;
    private final VehicleType type;
    public Vehicle(String licensePlate, 
            int spotsNeeded, VehicleType type) {
        this.licensePlate = licensePlate;
        this.spotsNeeded = spotsNeeded;
        this.type = type;
    }
    // getters omitted for brevity    
    // equals() and hashCode() omitted for brevity
}
```

接下来，我们需要设计停车场。主要是，停车场有几层（或级别），每层都有停车位。除其他外，停车场应该暴露出停车/取车的方法。这些方法将把停车/取车的任务委托给每一层（或特定的一层），直到成功或没有要扫描的层为止。

```java
public class ParkingLot {
    private String name;
    private Map<String, ParkingFloor> floors;
    public ParkingLot(String name) {
        this.name = name;
    }
    public ParkingLot(String name, 
            Map<String, ParkingFloor> floors) {
        this.name = name;
        this.floors = floors;
    }    
    // delegate to the proper ParkingFloor
    public ParkingTicket parkVehicle(Vehicle vehicle) {...}
    // we have to find the vehicle by looping floors  
    public boolean unparkVehicle(Vehicle vehicle) {...} 
    // we have the ticket, so we have the needed information
    public boolean unparkVehicle
        ParkingTicket parkingTicket) {...} 
    public boolean isFull() {...}
    protected boolean isFull(VehicleType type) {...}
    // getters and setters omitted for brevity
}
```

停车楼控制某一楼层的停车/取车过程。它有自己的停车票注册表，并能够管理其停车位。主要上，每个停车楼都充当独立的停车场。这样，我们可以关闭一个完整的楼层，而其余楼层不受影响：

```java
public class ParkingFloor{
    private final String name;
    private final int totalSpots;
    private final Map<String, ParkingSpot>
        parkingSpots = new LinkedHashMap<>();
    // here, I use a Set, but you may want to hold the parking 
    // tickets in a certain order to optimize search
    private final Set<ParkingTicket>
        parkingTickets = new HashSet<>();
private int totalFreeSpots;
    public ParkingFloor(String name, int totalSpots) {
        this.name = name;
        this.totalSpots = totalSpots;
        initialize(); // create the parking spots
    }
    protected ParkingTicket parkVehicle(Vehicle vehicle) {...}     
    //we have to find the vehicle by looping the parking spots  
    protected boolean unparkVehicle(Vehicle vehicle) {...} 
    // we have the ticket, so we have the needed information
    protected boolean unparkVehicle(
        ParkingTicket parkingTicket) {...} 
    protected boolean isFull(VehicleType type) {...}
    protected int countFreeSpots(
        VehicleType vehicleType) {...}
    // getters omitted for brevity
    private List<ParkingSpot> findSpotsToFitVehicle(
        Vehicle vehicle) {...}    
    private void assignVehicleToParkingSpots(
        List<ParkingSpot> spots, Vehicle vehicle) {...}    
    private ParkingTicket releaseParkingTicket(
        Vehicle vehicle) {...}    
    private ParkingTicket findParkingTicket(
        Vehicle vehicle) {...}    
    private void registerParkingTicket(
        ParkingTicket parkingTicket) {...}           
    private boolean unregisterParkingTicket(
        ParkingTicket parkingTicket) {...}                    
    private void initialize() {...}
}
```

最后，停车位是一个对象，它保存有关其名称（标签或编号）、可用性（是否空闲）和车辆（是否停放在该位置的车辆）的信息。它还具有分配/移除车辆到/从此位置的方法：

```java
public class ParkingSpot {
    private boolean free = true;
    private Vehicle vehicle;
    private final String label;
    private final ParkingFloor parkingFloor;
    protected ParkingSpot(ParkingFloor parkingFloor, 
            String label) {
        this.parkingFloor = parkingFloor;
        this.label = label;
    }
    protected boolean assignVehicle(Vehicle vehicle) {...}
    protected boolean removeVehicle() {...}
    // getters omitted for brevity
}
```

此刻，我们已经拥有了停车场的所有主要类。接下来，我们将专注于自动停车系统。这可以被塑造为一个作为停车场调度员的单一类：

```java
public class ParkingSystem implements Parking {

    private final String id;
    private final ParkingLot parkingLot;
    public ParkingSystem(String id, ParkingLot parkingLot) {
        this.id = id;
 this.parkingLot = parkingLot;
    }
    @Override
    public ParkingTicket parkVehicleBtn(
        String licensePlate, VehicleType type) {...}
    @Override
    public boolean unparkVehicleBtn(
        String licensePlate, VehicleType type) {...}
    @Override
    public boolean unparkVehicleBtn(
        ParkingTicket parkingTicket) {...}     
    // getters omitted for brevity
}
```

包含部分实现的完整应用程序被命名为*ParkingLot*。

## 示例 5：在线阅读系统

**问题**：设计在线阅读系统的主要类。

**需要询问的问题**：需要哪些功能？可以同时阅读多少本书？

**面试官**：系统应该能够管理读者和书籍。您的代码应该能够添加/移除读者/书籍并显示读者/书籍。系统一次只能为一个读者和一本书提供服务。

**解决方案**：为了理解我们的设计应该涉及哪些类，我们可以考虑草绘一些东西，如图 6.5 所示：

![图 6.5 – 一个在线阅读系统](img/Figure_6.5_B15403.jpg)

图 6.5 – 一个在线阅读系统

为了管理读者和书籍，我们需要拥有这样的对象。这是一个小而简单的部分，在面试中从这样的部分开始对打破僵局和适应手头的问题非常有帮助。当我们在面试中设计对象时，没有必要提出一个对象的完整版本。例如，一个读者有姓名和电子邮件，一本书有作者、标题和 ISBN 就足够了。让我们在下面的代码中看到它们。除了下面的代码，您还可以使用这里的 UML 图：[`github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/blob/master/Chapter06/OnlineReaderSystem/OnlineReaderSystemUML.png`](https://github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/blob/master/Chapter06/OnlineReaderSystem/OnlineReaderSystemUML.png)

```java
public class Reader {
    private String name;
    private String email;
    // constructor omitted for brevity
    // getters, equals() and hashCode() omitted for brevity
}
public class Book {
    private final String author;
    private final String title;
    private final String isbn;
    // constructor omitted for brevity
    public String fetchPage(int pageNr) {...}
    // getters, equals() and hashCode() omitted for brevity
}
```

接下来，如果我们考虑到书籍通常由图书馆管理，那么我们可以将添加、查找和移除书籍等多个功能封装在一个类中，如下所示：

```java
public class Library {
    private final Map<String, Book> books = new HashMap<>();
    protected void addBook(Book book) {
       books.putIfAbsent(book.getIsbn(), book);
    }
    protected boolean remove(Book book) {
       return books.remove(book.getIsbn(), book);
    }
    protected Book find(String isbn) {
       return books.get(isbn);
    }
}
```

读者可以由一个名为`ReaderManager`的类来管理。您可以在完整的应用程序中找到这个类。为了阅读一本书，我们需要一个显示器。`Displayer`应该显示读者和书籍的详细信息，并且应该能够浏览书籍的页面：

```java
public class Displayer {
    private Book book;
    private Reader reader;
    private String page;
    private int pageNumber;
    protected void displayReader(Reader reader) {
        this.reader = reader;
        refreshReader();
    }
    protected void displayBook(Book book) {
        this.book = book;
        refreshBook();
    }
    protected void nextPage() {
        page = book.fetchPage(++pageNumber);
        refreshPage();
    }
    protected void previousPage() {
        page = book.fetchPage(--pageNumber);        
        refreshPage();
    }        
    private void refreshReader() {...}    
    private void refreshBook() {...}    
    private void refreshPage() {...}
}
```

最后，我们所要做的就是将`Library`、`ReaderManager`和`Displayer`封装在`OnlineReaderSystem`类中。这个类在这里列出：

```java
public class OnlineReaderSystem {
    private final Displayer displayer;
    private final Library library;
    private final ReaderManager readerManager;
    private Reader reader;
    private Book book;
    public OnlineReaderSystem() {
        displayer = new Displayer();
        library = new Library();
        readerManager = new ReaderManager();
    }
    public void displayReader(Reader reader) {
        this.reader = reader;
        displayer.displayReader(reader);
    }
    public void displayReader(String email) {
        this.reader = readerManager.find(email);
        if (this.reader != null) {
            displayer.displayReader(reader);
        }
    }
    public void displayBook(Book book) {
        this.book = book;
        displayer.displayBook(book);
    }
    public void displayBook(String isbn) {
        this.book = library.find(isbn);
        if (this.book != null) {
            displayer.displayBook(book);
        }
    }
    public void nextPage() {
        displayer.nextPage();
    }
    public void previousPage() {
        displayer.previousPage();
    }
    public void addBook(Book book) {
        library.addBook(book);
    }
    public boolean deleteBook(Book book) {
        if (!book.equals(this.book)) {
            return library.remove(book);
        }
        return false;
    }
    public void addReader(Reader reader) {
        readerManager.addReader(reader);
    }
    public boolean deleteReader(Reader reader) {
        if (!reader.equals(this.reader)) {
            return readerManager.remove(reader);
        }
        return false;
    }
    public Reader getReader() {
        return reader;
    }
    public Book getBook() {
        return book;
    }
}
```

完整的应用程序名为*OnlineReaderSystem*。

## 示例 6：哈希表

**亚马逊**、**谷歌**、**Adobe**、**微软**

**问题**：设计一个哈希表（这是面试中非常流行的问题）。

**需要询问的问题**：需要哪些功能？应该应用什么技术来解决索引冲突？键值对的数据类型是什么？

`add()`和`get()`操作。为了解决索引冲突，我建议您使用*链接*技术。键值对应该是通用的。

**哈希表的简要概述**：哈希表是一种存储键值对的数据结构。通常，数组保存表中所有键值条目，该数组的大小设置为容纳预期数据量。每个键值的键通过哈希函数（或多个哈希函数）传递，输出哈希值或哈希。主要，哈希值表示哈希表中键值对的索引（例如，如果我们使用数组存储所有键值对，则哈希函数返回应该保存当前键值对的数组的索引）。通过哈希函数传递相同的键应该每次产生相同的索引 - 这对于通过其键查找值很有用。

当哈希函数为不同的键生成两个相同的索引时，我们面临索引冲突。解决索引冲突问题最常用的技术是*线性探测*（这种技术在表中线性搜索下一个空槽位 - 尝试在数组中找到一个不包含键值对的槽位（索引））和*chaining*（这种技术表示作为链表数组实现的哈希表 - 冲突存储在与链表节点相同的数组索引中）。下图是用于存储*名称-电话*对的哈希表。它具有*chaining*功能（检查*马里乌斯-0838234*条目，它被链接到*卡琳娜-0727928*，因为它们的键*马里乌斯*和*卡琳娜*导致相同的数组索引*126*）：

![图 6.6 - 哈希表](img/Figure_6.6_B15403.jpg)

图 6.6 - 哈希表

`HashEntry`）。正如您在前面的图中所看到的，键值对有三个主要部分：键、值和指向下一个键值对的链接（这样，我们实现*chaining*）。由于哈希表条目应该只能通过专用方法（如`get()`和`put()`）访问，因此我们将其封装如下：

```java
public class HashTable<K, V> {
    private static final int SIZE = 10;
    private static class HashEntry<K, V> {
        K key;
        V value;
        HashEntry <K, V> next;
        HashEntry(K k, V v) {
            this.key = k;
            this.value = v;
            this.next = null;
        }        
    }
    ...
```

接下来，我们定义包含`HashEntry`的数组。为了测试目的，大小为`10`的元素足够了，并且可以轻松测试*chaining*（大小较小容易发生碰撞）。实际上，这样的数组要大得多：

```java
private final HashEntry[] entries 
        = new HashEntry[SIZE];
    ...
```

接下来，我们添加`get()`和`put()`方法。它们的代码非常直观：

```java
    public void put(K key, V value) {
        int hash = getHash(key);
        final HashEntry hashEntry = new HashEntry(key, value);
        if (entries[hash] == null) {
            entries[hash] = hashEntry;
        } else { // collision => chaining
            HashEntry currentEntry = entries[hash];
            while (currentEntry.next != null) {
                currentEntry = currentEntry.next;
            }
            currentEntry.next = hashEntry;
        }
    }
    public V get(K key) {
        int hash = getHash(key);
        if (entries[hash] != null) {
            HashEntry currentEntry = entries[hash];
            // Loop the entry linked list for matching 
            // the given 'key'
            while (currentEntry != null) {                
                if (currentEntry.key.equals(key)) {
                    return (V) currentEntry.value;
                }
                currentEntry = currentEntry.next;
            }
        }
        return null;
    }
```

最后，我们添加一个虚拟哈希函数（实际上，我们使用诸如 Murmur 3 之类的哈希函数 - [`en.wikipedia.org/wiki/MurmurHash`](https://en.wikipedia.org/wiki/MurmurHash)）：

```java
    private int getHash(K key) {        
        return Math.abs(key.hashCode() % SIZE);
    }    
}
```

完成！完整的应用程序名为*HashTable*。

对于以下四个示例，我们跳过了书中的源代码。花点时间分析每个示例。能够理解现有设计是塑造设计技能的另一个工具。当然，您可以在查看书中代码之前尝试自己的方法，并最终比较结果。

## 示例 7：文件系统

**问题**：设计文件系统的主要类。

**要问的问题**：需要哪些功能？文件系统的组成部分是什么？

**面试官**：您的设计应支持目录和文件的添加、删除和重命名。我们谈论的是目录和文件的分层结构，就像大多数操作系统一样。

**解决方案**：完整的应用程序名为*FileSystem*。请访问以下链接以查看 UML：[`github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/blob/master/Chapter06/FileSystem/FileSystemUML.png`](https://github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/blob/master/Chapter06/FileSystem/FileSystemUML.png)

## 示例 8：元组

**亚马逊**，**谷歌**

**问题**：设计一个元组数据结构。

**要问的问题**：元组可以有 1 到*n*个元素。那么，您期望什么样的元组？元组中应存储什么数据类型？

**面试官**：我期望一个包含两个通用元素的元组。元组也被称为*pair*。

**解决方案**：完整的应用程序名为*Tuple*。

## 示例 9：带有电影票预订系统的电影院

亚马逊，谷歌，Adobe，微软

问题：设计一个带有电影票预订系统的电影院。

要问什么：电影院的主要结构是什么？它有多个影厅吗？我们有哪些类型的票？我们如何播放电影（只在一个房间，每天只播放一次）？

面试官：我期望一个有多个相同房间的电影院。一部电影可以同时在多个房间播放，同一部电影一天内可以在同一个房间播放多次。有三种类型的票，简单、白银和黄金，根据座位类型。电影可以以非常灵活的方式添加/移除（例如，我们可以在特定的开始时间从某些房间中移除一部电影，或者我们可以将一部电影添加到所有房间）。

解决方案：完整的应用程序名为 MovieTicketBooking。请访问以下链接查看 UML：[`github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/blob/master/Chapter06/MovieTicketBooking/MovieTicketBookingUML.png`](https://github.com/PacktPublishing/The-Complete-Coding-Interview-Guide-in-Java/blob/master/Chapter06/MovieTicketBooking/MovieTicketBookingUML.png)

示例 10：循环字节缓冲区

亚马逊，谷歌，Adobe

问题：设计一个循环字节缓冲区。

要问什么：它应该是可调整大小的？

面试官：是的，它应该是可调整大小的。主要是，我希望你设计所有你认为必要的方法的签名。

解决方案：完整的应用程序名为 CircularByteBuffer。

目前为止一切顺利！我建议你也尝试为前面的 10 个问题设计你自己的解决方案。不要认为所提供的解决方案是唯一正确的。尽可能多地练习，通过改变问题的背景来挑战自己，也尝试其他问题。

本章的源代码包名称为 Chapter06。

# 总结

本章涵盖了关于面向对象编程基础知识的最受欢迎的问题和 10 个设计编码挑战，在面试中非常受欢迎。在第一部分，我们从面向对象的概念（对象、类、抽象、封装、继承、多态、关联、聚合和组合）开始，继续讲解了 SOLID 原则，并以结合了面向对象编程概念、SOLID 原则和设计模式知识的问题为结束。在第二部分，我们解决了 10 个精心设计的设计编码挑战，包括设计点唱机、自动售货机和著名的哈希表。

练习这些问题和问题将使你有能力解决面试中遇到的任何面向对象编程问题。

在下一章中，我们将解决大 O 符号和时间的问题。
