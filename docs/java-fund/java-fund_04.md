# *第四章*

# 面向对象编程

## 学习目标

通过本课程结束时，您将能够：

+   解释 Java 中的类和对象的概念

+   解释面向对象编程的四个基本原则

+   在 Java 中创建简单的类并使用对象访问它们

+   在 Java 中实现继承

+   在 Java 中尝试方法重载和重写

+   在 Java 中创建和使用注释

## 介绍

到目前为止，我们已经了解了 Java 的基础知识以及如何使用简单的构造，如**条件**语句和循环语句，以及如何在 Java 中实现方法。理解这些基本概念非常重要，并且在构建简单程序时非常有用。然而，要构建和维护大型和复杂的程序，基本类型和构造是不够的。使 Java 真正强大的是它是一种面向对象的编程语言。它允许您有效地构建和集成复杂的程序，同时保持一致的结构，使其易于扩展、维护和重用。

在本课中，我们将介绍一种称为面向对象编程（OOP）的编程范式，它是 Java 的核心。我们将看看在 Java 中如何进行 OOP 以及如何实现它来设计更好的程序。

我们将从 OOP 的定义和其基本原则开始，然后看看称为**类**和**对象**的 OOP 构造，并最后通过查看称为**继承**的概念来结束本课。

我们将在 Java 中编写两个简单的 OOP 应用程序：一个用于表示通常在大学中找到的人，如学生、讲师和工作人员，另一个用于表示农场中的家畜。让我们开始吧！

## 面向对象原则

OOP 受四个主要原则的约束，如下所示。在本课的其余部分，我们将深入研究这些原则中的每一个：

+   **继承**：我们将学习如何通过使用类的层次结构和从派生类继承行为来重用代码

+   **封装**：我们还将看看如何可以隐藏外部世界的实现细节，同时通过方法提供一致的接口与我们的对象进行通信

+   **抽象**：我们将看看如何可以专注于对象的重要细节并忽略其他细节

+   **多态**：我们还将看看如何定义抽象行为并让其他类为这些行为提供实现

## 类和对象

编程中的范式是编写程序的风格。不同的语言支持不同的范式。一种语言可以支持多种范式。

### 面向对象编程

面向对象编程，通常称为 OOP，是一种处理对象的编程风格。对象是具有属性来保存其数据和方法来操作数据的实体。

让我们用更简单的术语来解释这一点。

在 OOP 中，我们主要处理对象和类。对象是现实世界项目的表示。对象的一个例子是您的汽车或您自己。对象具有与之关联的属性和可以执行的操作。例如，您的汽车具有轮子、门、发动机和齿轮，这些都是属性，它可以执行诸如加速、刹车和停止等操作，这些都称为方法。以下图表是您作为一个人所拥有的属性和方法的插图。属性有时可以称为**字段**：

![图 4.1：与人类相关的对象表示](img/C09581_Figure_04_01.jpg)

###### 图 4.1：与人类相关的对象表示

在 OOP 中，我们将类定义为项目的蓝图，将对象定义为类的实例。

类的一个例子是`Person`，`Person`的一个对象/实例的例子是学生或讲师。这些是属于`Person`类的具体示例对象：

![图 4.2 类实例的表示](img/C09581_Figure_04_02.jpg)

###### 图 4.2 类实例的表示

在上图中，`Person`类用于表示所有人，而不考虑他们的性别、年龄或身高。从这个类中，我们可以创建人的具体示例，如`Person`类内部的方框所示。

在 Java 中，我们主要处理类和对象，因此非常重要的是您理解两者之间的区别。

#### 注意

在 Java 中，除了原始数据类型之外，一切都是对象。

以下是 Java 中类定义的格式：

```java
modifier class ClassName {
    //Body
}
```

Java 中的类定义由以下部分组成：

+   `public`，`private`，`protected`，或者没有修饰符。一个`public`类可以从其他包中的其他类访问。一个`private`类只能从声明它的类中访问。一个`protected`类成员可以在同一个包中的所有类中访问。

+   **类名**：名称应以初始字母开头。

+   **主体**：类主体由大括号{ }括起来。这是我们定义类的属性和方法的地方。

### 类名的命名约定

Java 中类的命名约定如下：

+   类名应该使用驼峰命名法。也就是说，第一个单词应以大写字母开头，所有内部单词的第一个字母都应大写，例如`Cat`，`CatOwner`和`House`。

+   类名应该是名词。

+   类名应该是描述性的，不应该是缩写，除非它们是广为人知的。

以下是`Person`类的定义示例：

```java
public class Person {

}
```

修饰符是 public，意味着该类可以从其他 Java 包中访问。类名是`Person`。

以下是`Person`类的更健壮的示例，具有一些属性和方法：

```java
public class Person {

   //Properties
   int age;
   int height;
   String name;
   //Methods
   public void walk(){
       //Do walking operations here
   }
   public void sleep(){
       //Do sleeping operations here
   }
   private void takeShower(){
       //Do take shower operations here
   }
}
```

这些属性用于保存对象的状态。也就是说，`age`保存当前人的年龄，这可能与下一个人的年龄不同。`name`用于保存当前人的名字，这也将与下一个人不同。它们回答了这个问题：这个人是谁？

方法用于保存类的逻辑。也就是说，它们回答了这个问题：这个人能做什么？方法可以是私有的、公共的或受保护的。

方法中的操作可以根据应用程序的需要变得复杂。您甚至可以从其他方法调用方法，以及向这些方法添加参数。

### 练习 11：使用类和对象

执行以下步骤：

1.  打开 IntelliJ IDEA 并创建一个名为`Person.java`的文件。

1.  创建一个名为`Person`的公共类，具有三个属性，即`age`，`height`和`name`。`age`和`height`属性将保存整数值，而`name`属性将保存字符串值：

```java
public class Person {

   //Properties
   int age;
   int height;
   String name;
```

1.  定义三个方法，即`walk()`，`sleep()`和`takeShower()`。为每个方法编写打印语句，以便在调用它们时将文本打印到控制台上：

```java
  //Methods
   public void walk(){
       //Do walking operations here
       System.out.println("Walking...");
   }
   public void sleep(){
       //Do sleeping operations here
       System.out.println("Sleeping...");
   }
   private void takeShower(){
       //Do take shower operations here
       System.out.println("Taking a shower...");
   }
```

1.  现在，将`speed`参数传递给`walk()`方法。如果`speed`超过 10，我们将输出打印到控制台，否则我们不会：

```java
public void walk(int speed){
   //Do walking operations here
   if (speed > 10)
{
       System.out.println("Walking...");
}
```

1.  现在我们有了`Person`类，我们可以使用`new`关键字为其创建对象。在以下代码中，我们创建了三个对象：

```java
Person me = new Person();
Person myNeighbour = new Person();
Person lecturer = new Person();
```

`me`变量现在是`Person`类的对象。它代表了一种特定类型的人，即我。

有了这个对象，我们可以做任何我们想做的事情，比如调用`walk()`方法，调用`sleep()`方法，以及更多。只要类中有方法，我们就可以这样做。稍后，我们将看看如何将所有这些行为添加到一个类中。由于我们没有**main**方法，这段代码不会有任何输出。

### 练习 12：使用 Person 类

要调用类的成员函数，请执行以下步骤：

1.  在 IntelliJ 中创建一个名为`PersonTest`的新类。

1.  在`PersonTest`类中，创建`main`方法。

1.  在`main`方法中，创建`Person`类的三个对象

```java
public static void main(String[] args){
Person me = new Person();
Person myNeighbour = new Person();
Person lecturer = new Person();
```

1.  调用第一个对象的`walk()`方法：

```java
me.walk(20);
me.walk(5);
me.sleep();
```

1.  运行类并观察输出：

```java
Walking...
Sleeping…
```

1.  使用`myNeighbour`和`lecturer`对象来做同样的事情，而不是使用`me`：

```java
myNeighbour.walk(20);
myNeighbour.walk(5);
myNeighbour.sleep();
lecturer.walk(20);
lecturer.walk(5);
lecturer.sleep();
}
```

1.  再次运行程序并观察输出：

```java
Walking...
Sleeping...
Walking...
Sleeping...
Walking...
Sleeping...
```

在这个例子中，我们创建了一个名为`PersonTest`的新类，并在其中创建了`Person`类的三个对象。然后我们调用了`me`对象的方法。从这个程序中，可以明显看出`Person`类是一个蓝图，我们可以根据需要创建尽可能多的对象。我们可以分别操作这些对象，因为它们是完全不同和独立的。我们可以像处理其他变量一样传递这些对象，甚至可以将它们作为参数传递给其他对象。这就是面向对象编程的灵活性。

#### 注意

我们没有调用`me.takeShower()`，因为这个方法在`Person`类中声明为私有。私有方法不能在其类外部调用。

## 构造函数

要能够创建一个类的对象，我们需要一个构造函数。当你想要创建一个类的对象时，就会调用构造函数。当我们创建一个没有构造函数的类时，Java 会为我们创建一个空的默认构造函数，不带参数。如果一个类创建时没有构造函数，我们仍然可以用默认构造函数来实例化它。我们之前使用的`Person`类就是一个很好的例子。当我们想要一个`Person`类的新对象时，我们写下了以下内容：

```java
Person me = new Person();
```

默认构造函数是`Person()`，它返回`Person`类的一个新实例。然后我们将这个返回的实例赋给我们的变量`me`。

构造函数和其他方法一样，只是有一些不同：

+   构造函数的名称与类名相同

+   构造函数可以是`public`或`private`

+   构造函数不返回任何东西，甚至不返回`void`

让我们看一个例子。让我们为我们的`Person`类创建一个简单的构造函数：

```java
public class Person {
   //Properties
   int age;
   int height;
   String name;
   //Constructor
   public Person(int myAge){
       age = myAge;
   }

   //Methods
   public void walk(int speed){
       //Do walking operations here
       if (speed > 10)
           System.out.println("Walking...");
   }
   public void sleep(){
       //Do sleeping operations here
       System.out.println("Sleeping...");
   }
   private void takeShower(){
       //Do take shower operations here
       System.out.println("Taking a shower...");
   }
}
```

这个构造函数接受一个参数，一个名为`myAge`的整数，并将其值赋给类中的`age`属性。记住构造函数隐式返回类的实例。

我们可以使用构造函数再次创建`me`对象，这次传递`age`：

```java
Person me = new Person(30);
```

## this 关键字

在我们的`Person`类中，我们在构造函数中看到了以下行：

```java
age = myAge;
```

在这一行中，正如我们之前看到的，我们正在将当前对象的`age`变量设置为传入的新值`myAge`。有时，我们希望明确指出我们所指的对象。当我们想引用当前正在处理的对象中的属性时，我们使用`this`关键字。例如，我们可以将前面的行重写为以下形式：

```java
this.age = myAge;
```

在这一新行中，`this.age`用于引用当前正在处理的对象中的 age 属性。`this`用于访问当前对象的实例变量。

例如，在前面的行中，我们正在将当前对象的`age`设置为传递给构造函数的值。

除了引用当前对象，如果你有多个构造函数，`this`还可以用来调用类的其他构造函数。

在我们的`Person`类中，我们将创建一个不带参数的第二个构造函数。如果调用此构造函数，它将调用我们创建的另一个构造函数，并使用默认值 28：

```java
//Constructor
public Person(int myAge){
   this.age = myAge;
}
public Person(){
   this(28);
}
```

现在，当调用`Person me = new Person()`时，第二个构造函数将调用第一个构造函数，并将`myAge`设置为 28。第一个构造函数将当前对象的`age`设置为 28。

### 活动 12：在 Java 中创建一个简单的类

场景：假设我们想为一个动物农场创建一个程序。在这个程序中，我们需要跟踪农场上的所有动物。首先，我们需要一种方法来表示动物。我们将创建一个动物类来表示单个动物，然后创建这个类的实例来表示具体的动物本身。

目标：我们将创建一个 Java 类来表示动物，并创建该类的实例。到本次活动结束时，我们应该有一个简单的`Animal`类和该类的几个实例。

目标：了解如何在 Java 中创建类和对象。

按照以下步骤完成活动

1.  在 IDE 中创建一个新项目，命名为`Animals`。

1.  在项目中，在**src/**文件夹下创建一个名为`Animal.java`的新文件。

1.  创建一个名为`Animal`的类，并添加实例变量`legs`、`ears`、`eyes`、`family`和`name`。

1.  定义一个没有参数的构造函数，并将`legs`初始化为 4，`ears`初始化为 2，`eyes`初始化为 2。

1.  定义另一个带有`legs`、`ears`和`eyes`作为参数的带参数构造函数。

1.  为`name`和`family`添加 getter 和 setter。

1.  创建另一个名为`Animals.java`的文件，定义`main`方法，并创建`Animal`类的两个对象。

1.  创建另一个具有两条`legs`、两只`ears`和两只`eyes`的动物。

1.  为了设置动物的`name`和`family`，我们将使用在类中创建的 getter 和 setter，并打印动物的名字。

输出应该类似于以下内容：

![图 4.4：Animal 类的输出](img/C09581_Figure_04_03.jpg)

###### 图 4.3：Animal 类的输出

#### 注意

这项活动的解决方案可以在 314 页找到。

### 活动 13：编写一个 Calculator 类

对于这个活动，你将创建一个 Calculator 类，给定两个操作数和一个运算符，可以执行操作并返回结果。这个类将有一个 operate 方法，它将使用两个操作数执行操作。操作数和运算符将是类中的字段，通过构造函数设置。

有了 Calculator 类准备好后，编写一个应用程序，执行一些示例操作，并将结果打印到控制台。

要完成这项活动，你需要：

1.  创建一个名为`Calculator`的类，有三个字段：`double` `operand1`、`double` `operand2`和`String` `operator`。添加一个设置所有三个字段的构造函数。

1.  在这个类中，添加一个`operate`方法，它将检查运算符是什么（"+"、"-"、"x"或"/"），并执行正确的操作，返回结果。

1.  在这个类中添加一个`main`方法，这样你就可以写几个示例案例并打印结果。

#### 注意

这项活动的解决方案可以在 318 页找到。

## 继承

在这一部分，我们将看一下面向对象编程的另一个重要原则，称为继承。面向对象编程中的继承与英语中的继承意思相同。让我们通过使用我们的家谱来看一个例子。我们的父母继承自我们的祖父母。然后我们从我们的父母那里继承，最后，我们的孩子继承，或者将从我们那里继承。同样，一个类可以继承另一个类的属性。这些属性包括方法和字段。然后，另一个类仍然可以从它那里继承，依此类推。这形成了我们所说的**继承层次结构**。

被继承的类称为**超类**或**基类**，继承的类称为**子类**或**派生类**。在 Java 中，一个类只能从一个超类继承。

### 继承的类型

继承的一个例子是公司或政府中的管理层次结构：

+   **单级继承**：在单级继承中，一个类只从另一个类继承：

![图 4.5：单级继承的表示](img/C09581_Figure_04_05.jpg)

###### 图 4.4：单级继承的表示

+   **多级继承**：在多级继承中，一个类可以继承另一个类，而另一个类也可以继承另一个类：

![图 4.6：多级继承的表示](img/C09581_Figure_04_06.jpg)

###### 图 4.5：多级继承的表示

+   **多重继承**：在这里，一个类可以从多个类继承：

![图 4.7：多重继承的表示](img/C09581_Figure_04_07.jpg)

###### 图 4.6：多重继承的表示

在 Java 中不直接支持多重继承，但可以通过使用**接口**来实现，这将在下一课程中介绍。

### 面向对象编程中继承的重要性

让我们回到我们的`Person`类。

很明显，所有人都支持一些共同的属性和行为，尽管他们的性别或种族不同。例如，在属性方面，每个人都有一个名字，每个人都有年龄、身高和体重。在行为方面，所有人都睡觉，所有人都吃饭，所有人都呼吸，等等。

我们可以在所有的`Person`类中定义所有这些属性和方法的代码，也可以在一个类中定义所有这些常见属性和操作，让其他`Person`类从这个类继承。这样，我们就不必在这些子类中重写属性和方法。因此，继承允许我们通过重用代码来编写更简洁的代码。

一个类从另一个类继承的语法如下：

```java
class SubClassName extends SuperClassName {
}
```

我们使用`extends`关键字来表示继承。

例如，如果我们希望我们的`Student`类扩展`Person`类，我们会这样声明：

```java
public class Student extends Person {
}
```

在这个`Student`类中，我们可以访问我们在`Person`类中之前定义的公共属性和方法。当我们创建这个`Student`类的实例时，我们自动可以访问我们之前在`Person`类中定义的方法，比如`walk()`和`sleep()`。我们不需要再重新创建这些方法，因为我们的`Student`类现在是`Person`类的子类。但是，我们无法访问私有方法，比如`takeShower()`。

#### 注意

请注意，子类只能访问其超类中的公共属性和方法。如果在超类中将属性或方法声明为私有，则无法从子类访问它。默认情况下，我们声明的属性只能从同一包中的类中访问，除非我们在它们之前明确放置`public`修饰符。

在我们的`Person`类中，让我们定义一些所有人都具有的常见属性和方法。然后，我们将从这个类继承这些属性，以创建其他类，比如`Student`和`Lecturer`：

```java
public class Person {
   //Properties
   int age;
   int height;
   int weight;
   String name;
   //Constructors
   public Person(int myAge, int myHeight, int myWeight){
       this.age = myAge;
       this.height = myHeight;
       this.weight = myWeight;
   }
   public Person(){
       this(28, 10, 60);
   }
   //Methods
   public void walk(int speed){
       if (speed > 10)
           System.out.println("Walking...");
   }
   public void sleep(){
       System.out.println("Sleeping...");
   }
   public  void setName(String name){
       this.name = name;
   }
   public String getName(){
       return name;
   }
   public int getAge(){
       return age;
   }
   public int getHeight(){
       return height;
   }
   public int getWeight(){
       return weight;
   }
}
```

在这里，我们定义了四个属性，两个构造函数和七个方法。您能解释每个方法的作用吗？目前这些方法都相当简单，这样我们就可以专注于继承的核心概念。我们还修改了构造函数以接受三个参数。

让我们创建一个从`Person`类继承的`Student`类，创建一个类的对象，并设置学生的名字：

```java
public class Student extends Person {
   public static void main(String[] args){
       Student student = new Student();
       student.setName("James Gosling");
   }
}
```

我们创建了一个新的`Student`类，它继承自`Person`类。我们还创建了`Student`类的一个新实例，并设置了它的名字。请注意，我们没有在`Student`类中重新定义`setName()`方法，因为它已经在`Person`类中定义了。我们还可以在我们的`student`对象上调用其他方法：

```java
public class Student extends Person {
   public static void main(String[] args){
       Student student = new Student();
       student.setName("James Gosling");
       student.walk(20);
       student.sleep();
       System.out.println(student.getName());
       System.out.println(student.getAge());
   }
} 
```

请注意，我们没有在`Student`类中创建这些方法，因为它们已经在`Student`类继承的`Person`类中定义。

### 在 Java 中实现继承

写下上述程序的预期输出。通过查看程序来解释输出。

解决方案是：

```java
Walking...
Sleeping...
James Gosling
28
```

让我们定义一个从相同的`Person`类继承的`Lecturer`类：

```java
public class Lecturer extends Person {
   public static void main(String[] args){
       Lecturer lecturer = new Lecturer();
       lecturer.setName("Prof. James Gosling");
       lecturer.walk(20);
       lecturer.sleep();
       System.out.println(lecturer.getName());
       System.out.println(lecturer.getAge());
   }
}
```

#### 注意

请注意继承如何帮助我们通过重用相同的`Person`类来减少我们编写的代码量。如果没有继承，我们将不得不在所有的类中重复相同的方法和属性。

### 活动 14：使用继承创建计算器

在之前的活动中，您创建了一个`Calculator`类，其中包含了同一类中所有已知的操作。当您考虑添加新操作时，这使得这个类更难扩展。操作方法将无限增长。

为了使这个更好，你将使用面向对象的实践将操作逻辑从这个类中拆分出来，放到它自己的类中。在这个活动中，你将创建一个名为 Operator 的类，默认为求和操作，然后创建另外三个类来实现其他三种操作：减法、乘法和除法。这个 Operator 类有一个`matches`方法，给定一个字符串，如果该字符串表示该操作符，则返回 true，否则返回 false。

将操作逻辑放在它们自己的类中，编写一个名为`CalculatorWithFixedOperators`的新类，其中有三个字段：`double` `operand1`、`double` `operand2`和类型为`Operator`的`operator`。这个类将具有与之前计算器相同的构造函数，但不再将操作符存储为字符串，而是使用`matches`方法来确定正确的操作符。

与之前的计算器一样，这个计算器也有一个返回 double 的`operate`方法，但不再有任何逻辑，而是委托给在构造函数中确定的当前操作符。

要完成这个活动，你需要：

1.  创建一个名为`Operator`的类，它有一个在构造函数中初始化的 String 字段，表示操作符。这个类应该有一个默认构造函数，表示默认操作符，即`sum`。操作符类还应该有一个名为`operate`的方法，接收两个 double 并将操作符的结果作为 double 返回。默认操作是求和。

1.  创建另外三个类：`Subtraction`、`Multiplication`和`Division`。它们继承自 Operator，并重写了代表它们的每种操作的`operate`方法。它们还需要一个不带参数的构造函数，调用 super 传递它们代表的操作符。

1.  创建一个名为`CalculatorWithFixedOperators`的新类。这个类将包含四个常量（finals）字段，表示四种可能的操作。它还应该有另外三个字段：类型为 double 的`operand1`和`operator2`，以及类型为`Operator`的`operator`。这另外三个字段将在构造函数中初始化，该构造函数将接收操作数和操作符作为字符串。使用可能操作符的匹配方法，确定哪一个将被设置为操作符字段。

1.  与之前的`Calculator`类一样，这个类也将有一个`operate`方法，但它只会委托给`operator`实例。

1.  最后，编写一个`main`方法，多次调用新的计算器，打印每次操作的结果。

#### 注意

重写计算器以使用更多的类似乎比最初的代码更复杂。但它抽象了一些重要的行为，打开了一些将在未来活动中探索的可能性。

#### 注意

这个活动的解决方案可以在第 319 页找到。

## 重载

我们将讨论的下一个面向对象的原则叫做重载。重载是面向对象编程中的一个强大概念，它允许我们重用方法名，只要它们具有不同的签名。**方法签名**是方法名、它的参数和参数的顺序：

![图 4.8：方法签名的表示](img/C09581_Figure_04_08.jpg)

###### 图 4.7：方法签名的表示

上述是一个从给定银行名称中提取资金的方法的示例。该方法返回一个 double 并接受一个 String 参数。这里的方法签名是`getMyFundsFromBank()`方法的名称和 String 参数`bankName`。签名不包括方法的返回类型，只包括名称和参数。

通过重载，我们能够定义多个方法，这些方法具有相同的方法名，但参数不同。这在定义执行相同操作但接受不同参数的方法时非常有用。

让我们看一个例子。

让我们定义一个名为`Sum`的类，其中有三个重载的方法，用来对传递的参数进行相加并返回结果：

```java
public class Sum {
    //This sum takes two int parameters
    public int sum(int x, int y) {
        return (x + y);
    }
    //This sum takes three int parameters
    public int sum(int x, int y, int z) {
        return (x + y + z);
    }
    //This sum takes two double parameters
    public double sum(double x, double y) {
        return (x + y);
    }
    public static void main(String args[]) {
        Sum s = new Sum();
        System.out.println(s.sum(10, 20));
        System.out.println(s.sum(10, 20, 30));
        System.out.println(s.sum(10.5, 20.5));
    }
}
```

输出如下：

```java
30
60
31.0
```

在这个例子中，`sum()`方法被重载以接受不同的参数并返回总和。方法名相同，但每个方法都接受不同的参数集。方法签名的差异允许我们使用相同的名称多次。

你可能会想知道重载对面向对象编程带来了什么好处。想象一种情况，我们不能多次重用某个方法名称，就像在某些语言中，比如 C 语言。为了能够接受不同的参数集，我们需要想出六个不同的方法名称。为了那些本质上做同样事情的方法想出六个不同的名称是繁琐和痛苦的，尤其是在处理大型程序时。重载可以避免我们遇到这样的情况。

让我们回到我们的`Student`类，并创建两个重载的方法。在第一个方法中，我们将打印一个字符串来打印“去上课...”，无论这一周的哪一天。在第二个方法中，我们将传递一周的哪一天，并检查它是否是周末。如果是周末，我们将打印出一个与其他工作日不同的字符串。这是我们将如何实现它：

```java
public class Student extends Person {
   //Add this
   public void goToClass(){
       System.out.println("Going to class...");
   }
   public void goToClass(int dayOfWeek){
       if (dayOfWeek == 6 || dayOfWeek == 7){
           System.out.println("It's the weekend! Not to going to class!");
       }else {
           System.out.println("Going to class...");
       }
   }
   public static void main(String[] args){
       Student student = new Student();
       student.setName("James Gosling");
       student.walk(20);
       student.sleep();
       System.out.println(student.getName());
       System.out.println(student.getAge());
       //Add this
       student.goToClass();
       student.goToClass(6);
   }
}
```

输出如下：

```java
Walking...
Sleeping...
James Gosling
28
Going to class...
It's the weekend! Not to going to class!
```

打开我们创建的`Lecturer`类，并添加两个重载的方法，如下所示：

+   `teachClass()`打印出"Teaching a random class"

+   `teachClass(String className)`打印出"`Teaching` " + `className`

以下是代码：

```java
public void teachClass(){
   System.out.println("Teaching a random class.");
}
public void teachClass(String className){
   System.out.println("Teaching " + className);
}
```

我们可以在一个类中重载主方法，但一旦程序启动，JVM 只会调用`main(String[] args)`。我们可以从这个`main`方法中调用我们重载的`main`方法。以下是一个例子：

```java
public class Student {
    public static void main(String[] args){
        // Will be called by the JVM
    }
    public static void main(String[] args, String str1, int num){
        //Do some operations
    }
    public static void main(int num, int num1, String str){

    }
}
```

在这个例子中，`main`方法被重载了三次。然而，当我们运行程序时，只会调用签名为`main(String[] args)`的主方法。从我们的代码的任何地方，我们都可以自由地调用其他主方法。

## 构造函数重载

就像方法一样，构造函数也可以被重载。当在同一个类中使用不同参数声明相同的构造函数时，这被称为**构造函数重载**。编译器根据参数的数量和数据类型来区分应该调用哪个构造函数。

在我们讨论构造函数时，我们为我们的`Person`类创建了第二个构造函数，它接受`age`、`height`和`weight`作为参数。我们可以在同一个类中拥有不接受参数的构造函数和这个构造函数。这是因为这两个构造函数具有不同的签名，因此可以并存。让我们看看我们如何做到这一点：

```java
//Constructors
public Person(){
   this(28, 10, 60);
}
//Overloaded constructor
public Person(int myAge, int myHeight, int myWeight){
   this.age = myAge;
   this.height = myHeight;
   this.weight = myWeight;
}
```

这两个构造函数具有相同的名称（类名），但接受不同的参数。

添加一个接受`age`、`height`、`weight`和`name`的第三个构造函数。在构造函数内，将所有类变量设置为传递的参数。

代码如下：

```java
public Person(int myAge, int myHeight, int myWeight, String name){
   this.age = myAge;
   this.height = myHeight;
   this.weight = myWeight;
   this.name = name;
}
```

## 多态和重写

我们将要讨论的下一个面向对象编程原则是多态。术语“**多态**”源自生物学，即一个生物体可以呈现多种形式和阶段。这个术语也用在面向对象编程中，子类可以定义它们独特的行为，但仍然与父类共享一些功能。

让我们用一个例子来说明这一点。

在我们的`Person`示例中，我们有一个名为`walk`的方法。在我们的`Student`类中，它继承自`Person`类，我们将重新定义相同的`walk`方法，但现在是走去上课而不仅仅是走路。在我们的`Lecturer`类中，我们也将重新定义相同的`walk`方法，这次是走到教职工室而不是走到教室。这个方法必须与超类中的`walk`方法具有相同的签名和返回类型，才能被认为是多态的。以下是我们`Student`类中实现的样子：

```java
public class Student extends Person {
       ….
   public void walk(int speed){
       //Walk to class
       System.out.println("Walking to class ..");
   }
…...
}
```

当我们调用`student.walk(20)`时，我们的`Student`类中的这个方法将被调用，而不是`Person`类中的相同方法。也就是说，我们为我们的`Student`类提供了一种独特的行走方式，这与`Lecturer`和`Person`类不同。

在 Java 中，我们将这样的方法称为重写方法，这个过程称为方法重写。Java 虚拟机（JVM）调用适当的方法来引用对象。

### 重写和重载之间的区别

让我们看一下方法重载和重写之间的区别：

+   方法重载涉及在同一个类中有两个或更多个具有相同名称但不同参数的方法：

```java
void foo(int a)
void foo(int a, float b)
```

+   方法重写意味着有两个具有相同参数但不同实现的方法。其中一个存在于父类中，而另一个存在于子类中：

```java
class Parent {
    void foo(double d) {
        // do something
    }
}
class Child extends Parent {

    void foo(double d){
        // this method is overridden.  
    }
}
```

## 注解

现在我们将介绍另一个将帮助我们编写更好的 Java 程序的重要主题。

注解是我们可以向程序添加元数据的一种方式。这些元数据可以包括我们正在开发的类的版本信息。这在类被弃用或者我们正在重写某个方法的情况下非常有用。这样的元数据不是程序本身的一部分，但可以帮助我们捕捉错误或提供指导。注解对其注释的代码的操作没有直接影响。

让我们看一个场景。我们如何确保我们正在重写某个方法而不是创建另一个完全不同的方法？当重写方法时，一个错误，比如使用不同的返回类型，将导致该方法不再被重写。这样的错误很容易犯，但如果在软件开发阶段没有及时处理，后来可能会导致软件错误。那么，我们如何强制重写？答案，你可能已经猜到了，就是使用注解。

@字符告诉编译器接下来是一个注解。

让我们在我们的`Student`类中使用注解来强制重写：

```java
@Override
public void walk(int speed){
   //Walk to class
   System.out.println("Walking to class ..");
}
```

请注意，我们在方法名称上方添加了`@Override`行，以指示该方法是从超类中重写的。当编译程序时，编译器将检查此注解，并立即知道我们正在尝试重写此方法。它将检查此方法是否存在于超类中，以及重写是否已正确完成。如果没有，它将报告错误以指示该方法不正确。这在某种程度上将防止我们犯错。

Java 包含内置注解，您也可以创建自己的注解。注解可以应用于类、属性、方法和其他程序元素的声明。在声明上使用时，每个注解按照惯例出现在自己的一行上。让我们看一些 Java 中内置注解的例子：

![表 4.1：不同注解及其用途的表格](img/C09581_Table_04_01.jpg)

###### 表 4.1：不同注解及其用途的表格

### 创建您自己的注解类型

注解是使用**interface**关键字创建的。让我们声明一个注解，以便我们可以添加类的作者信息：

```java
public @interface Author {
    String name();
    String date();
}
```

此注释接受作者的姓名和日期。然后我们可以在我们的`Student`类中使用这个注释：

```java
@Author(name = "James Gosling", date = "1/1/1970")
public class Student extends Person {
}
```

您可以在上面的示例中用您的值替换名称和日期。

## 引用

在您使用对象时，重要的是您了解**引用**。引用是一个地址，指示对象的变量和方法存储在哪里。

当我们将对象分配给变量或将它们作为参数传递给方法时，我们实际上并没有传递对象本身或其副本 - 我们传递的是对象本身在内存中的引用。

为了更好地理解引用的工作原理，让我们举个例子。

以下是一个例子：

创建一个名为`Rectangle`的新类，如下所示：

```java
public class Rectangle {
    int width;
    int height;
    public Rectangle(int width, int height){
        this.width = width;
        this.height = height;
    }
    public static void main(String[] args){
        Rectangle r1, r2;
        r1 = new Rectangle(100, 200);
        r2 = r1;
        r1.height = 300;
        r1.width = 400;
        System.out.println("r1: width= " + r1.width + ", height= " + r1.height);
        System.out.println("r2: width= " + r2.width + ", height= " + r2.height);
    }
}
```

以下是输出结果：

```java
r1: width= 400, height= 300
r2: width= 400, height= 300
```

以下是前面程序中发生的事情的总结：

1.  我们创建了两个类型为`Rectangle`的变量`r1`和`r2`。

1.  一个新的`Rectangle`对象被赋给`r1`。

1.  `r1`的值被赋给`r2`。

1.  `r2`的宽度和高度被改变。

1.  最终打印了这两个对象的值。

你可能期望`r1`和`r2`的值不同。然而，输出结果却不是这样。这是因为当我们使用`r2 = r1`时，我们创建了一个从`r2`到`r1`的引用，而不是创建一个从`r1`复制的新对象`r2`。也就是说，`r2`指向了`r1`所指向的相同对象。任何一个变量都可以用来引用对象并改变它的变量：

![图 4.9：对象 r1，r2 的表示](img/C09581_Figure_04_09.jpg)

###### 图 4.8：对象 r1，r2 的表示

如果你想让`r2`引用一个新对象，使用以下代码：

```java
r1 = new Rectangle(100, 200);
r2 = new Rectangle(300, 400);
```

在 Java 中，引用在参数传递给方法时变得特别重要。

#### 注意

在 Java 中没有显式指针或指针算术，就像 C 和 C++中一样。然而，通过使用引用，大多数指针功能被复制，而不带有许多它们的缺点。

### 活动 15：理解 Java 中的继承和多态

场景：想象我们希望我们在活动一中创建的`Animals`类更加面向对象。这样，以后如果我们的农场需要，它将更容易维护和扩展。

目标：我们将创建类来继承我们的`Animals`类，实现重载和重写的方法，并创建一个注解来对我们的类进行版本控制。

目标：理解如何从一个类继承，重载和重写方法，并在 Java 中创建注解。

步骤：

1.  打开我们之前创建的`Animals`项目。

1.  在项目中，在`src/`文件夹中创建一个名为`Cat.java`的新文件。

1.  打开`Cat.java`并从`Animals`类继承。

1.  在其中，创建`Cat`类的一个新实例，并将家庭设置为"`Cat`"，名称设置为"`Puppy`"，`ears`设置为两个，`eyes`设置为两个，`legs`设置为四个。不要重新定义这些方法和字段 - 而是使用从`Animals`类继承的方法。

1.  打印`family`，`name`，`ears`，`legs`和`eyes`。输出是什么？

#### 注意

这个活动的解决方案可以在第 322 页找到。

## 总结

在这节课中，我们学到了类是可以创建对象的蓝图，而对象是类的实例，并提供了该类的具体实现。类可以是公共的、私有的或受保护的。类有一个不带参数的默认构造函数。我们可以在 Java 中有用户定义的构造函数。`this`关键字用于引用类的当前实例。

我们接着学习了继承是一个子类继承了父类的属性的特性。

我们继续学习了 Java 中的重载、多态、注解和引用。

在下一节课中，我们将看一下在 Java 中使用接口和`Object`类。
