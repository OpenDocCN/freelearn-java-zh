# *第五章*

# 深入了解面向对象编程

## 学习目标

在本课结束时，您将能够：

+   在 Java 中实现接口

+   执行类型转换

+   利用`Object`类

+   使用抽象类和方法

## 介绍

在上一课中，我们看了面向对象编程的基础知识，如类和对象、继承、多态和重载。

我们看到类如何作为一个蓝图，我们可以从中创建对象，并看到方法如何定义类的行为，而字段保存状态。

我们看了一个类如何通过继承从另一个类获得属性，以便我们可以重用代码。然后，我们学习了如何通过重载重用方法名称 - 也就是说，只要它们具有不同的签名。最后，我们看了子类如何通过覆盖超类的方法重新定义自己独特的行为。

在本课中，我们将深入探讨面向对象编程的原则，以及如何更好地构建我们的 Java 程序。

我们将从接口开始，这些构造允许我们定义任何类都可以实现的通用行为。然后，我们将学习一个称为**类型转换**的概念，通过它我们可以将一个变量从一种类型转换为另一种类型，然后再转回来。同样，我们将使用 Java 提供的包装类将原始数据类型作为对象处理。最后，我们将详细了解抽象类和方法，这是一种让继承您的类的用户运行其自己独特实现的方法。

在这节课中，我们将通过使用我们在上一课创建的“动物”类来进行三个活动。我们还将使用我们的“人”类来演示一些概念。

让我们开始吧！

## 接口

在 Java 中，您可以使用接口提供一组类必须实现的方法。

让我们以我们的“人”类为例。我们想定义一组行为，定义任何人的行为，而不管他们的年龄或性别。

这些操作的一些示例包括睡觉、呼吸和移动/行走。我们可以将所有这些常见操作放在一个接口中，让任何声称是人的类来实现它们。实现此接口的类通常被称为“人”类型。

在 Java 中，我们使用关键字 interface 来表示接下来的代码块将是一个接口。接口中的所有方法都是空的，没有实现。这是因为任何实现此接口的类都将提供其独特的实现细节。因此，接口本质上是一组没有主体的方法。

让我们创建一个接口来定义一个人的行为：

```java
public interface PersonBehavior {
   void breathe();
   void sleep();
   void walk(int speed);
}
```

这个接口称为`PersonBehavior`，它包含三个方法：一个用于呼吸，另一个用于睡觉，还有一个用于以给定速度行走。实现此接口的每个类都必须实现这三个方法。

当我们想要实现一个给定的接口时，我们在类名后面使用`implements`关键字，然后是接口名。

让我们举个例子。我们将创建一个新的类`Doctor`来代表医生。这个类将实现`PersonBehavior`接口：

```java
public class Doctor implements PersonBehavior {
}
```

因为我们已经声明要符合`PersonBehavior`接口，如果我们不实现接口中的三个方法，编译器将给出错误。

```java
public class Doctor implements PersonBehavior {
   @Override
   public void breathe() {

   }
   @Override
   public void sleep() {
   }
   @Override
   public void walk(int speed) {
   }
```

我们使用`@Override`注解来指示这个方法来自接口。在这些方法中，我们可以自由地执行与我们的“医生”类相关的任何操作。

在相同的精神下，我们也可以创建一个实现相同接口的“工程师”类：

```java
public class Engineer implements PersonBehavior {
   @Override
   public void breathe() {

   }
   @Override
   public void sleep() {
   }
   @Override
   public void walk(int speed) {
   }
}
```

在*第 1 课*，*Java 简介*中，我们提到抽象是面向对象编程的基本原则之一。抽象是我们为类提供一致的接口的一种方式。

让我们以手机为例。使用手机，您可以给朋友打电话和发短信。打电话时，您按下通话按钮，立即与朋友连接。该通话按钮形成了您和朋友之间的接口。我们并不真正知道按下按钮时会发生什么，因为所有这些细节都对我们进行了抽象（隐藏）。

您经常会听到**API**这个术语，它代表应用程序编程接口。这是不同软件和谐交流的一种方式。例如，当您想要使用 Facebook 或 Google 登录应用程序时。应用程序将调用 Facebook 或 Google API。然后 Facebook API 将定义要遵循的登录规则。

Java 中的类可以实现多个接口。这些额外的接口用逗号分隔。类必须为接口中它承诺实现的所有方法提供实现：

```java
public class ClassName implements  InterfaceA, InterfaceB, InterfaceC {

}
```

### 用例：监听器

接口最重要的用途之一是为程序中的条件或事件创建监听器。基本上，监听器在发生动作时通知您任何状态更改。监听器也称为回调 - 这个术语源自过程式语言。

例如，当单击或悬停在按钮上时，可以调用事件监听器。

这种事件驱动的编程在使用 Java 制作 Android 应用程序时很受欢迎。

想象一下，我们想要知道一个人行走或睡觉时，以便我们可以执行一些其他操作。我们可以通过使用一个监听此类事件的接口来实现这一点。我们将在以下练习中看到这一点。

### 练习 13：实现接口

我们将创建一个名为`PersonListener`的接口，用于监听两个事件：`onPersonWalking`和`onPersonSleeping`。当调用`walk(int speed)`方法时，我们将分派`onPersonWalking`事件，当调用`sleep()`时，将调用`onPersonSleeping`：

1.  创建一个名为`PersonListener`的接口，并将以下代码粘贴到其中：

```java
public interface PersonListener {
   void onPersonWalking();
   void onPersonSleeping();
}
```

1.  打开我们的`Doctor`类，并在`PersonBehavior`接口之后添加`PersonListener`接口，用逗号分隔：

```java
public class Doctor implements PersonBehavior, PersonListener {
```

1.  实现我们的`PersonListener`接口中的两个方法。当医生行走时，我们将执行一些操作并触发`onPersonWalking`事件，以让其他监听器知道医生正在行走。当医生睡觉时，我们将触发`onPersonSleeping`事件。修改`walk()`和`sleep()`方法如下：

```java
@Override
public void breathe() {
}
@Override
public void sleep() {
    //TODO: Do other operations here
    // then raise event
    this.onPersonSleeping();
}
@Override
public void walk(int speed) {
    //TODO: Do other operations here
    // then raise event
    this.onPersonWalking();
}
@Override
public void onPersonWalking() {
    System.out.println("Event: onPersonWalking");
}
@Override
public void onPersonSleeping() {
    System.out.println("Event: onPersonSleeping");
} 
```

1.  通过调用`walk()`和`sleep()`来添加主方法以测试我们的代码：

```java
public static void main(String[] args){
   Doctor myDoctor = new Doctor();
   myDoctor.walk(20);
   myDoctor.sleep();
}
```

1.  运行`Doctor`类并在控制台中查看输出。您应该看到类似于这样的内容：

![图 5.1：Doctor 类的输出](img/C09581_05_01.jpg)

###### 图 5.1：Doctor 类的输出

完整的`Doctor`类如下：

```java
public class Doctor implements PersonBehavior, PersonListener {

   public static void main(String[] args){
       Doctor myDoctor = new Doctor();
       myDoctor.walk(20);
       myDoctor.sleep();
   }
   @Override
   public void breathe() {
   }
   @Override
   public void sleep() {
       //TODO: Do other operations here
       // then raise event
       this.onPersonSleeping();
   }
   @Override
   public void walk(int speed) {
       //TODO: Do other operations here
       // then raise event
       this.onPersonWalking();
   }
   @Override
   public void onPersonWalking() {
       System.out.println("Event: onPersonWalking");
   }
   @Override
   public void onPersonSleeping() {
       System.out.println("Event: onPersonSleeping");
   }
}
```

#### 注意

由于一个类可以实现多个接口，我们可以在 Java 中使用接口来模拟多重继承。

### 活动 16：在 Java 中创建和实现接口

场景：在我们之前的动物农场中，我们希望所有动物都具备的共同动作，而不管它们的类型如何。我们还想知道动物何时移动或发出任何声音。移动可以帮助我们跟踪每个动物的位置，声音可以表明动物是否处于困境。

目标：我们将实现两个接口：一个包含所有动物必须具备的两个动作`move()`和`makeSound()`，另一个用于监听动物的移动和声音。

目标：了解如何在 Java 中创建接口并实现它们。

这些步骤将帮助您完成此活动：

1.  打开上一课的`Animals`项目。

1.  创建一个名为`AnimalBehavior`的新接口。

1.  在其中创建两个方法：`void move()`和`void makeSound()`

1.  创建另一个名为`AnimalListener`的接口，其中包含`onAnimalMoved()`和`onAnimalSound()`方法。

1.  创建一个名为`Cow`的新公共类，并实现`AnimalBehavior`和`AnimalListener`接口。

1.  在`Cow`类中创建实例变量`sound`和`movementType`。

1.  重写`move()`，使`movementType`为"Walking"，并调用`onAnimalMoved()`方法。

1.  重写`makeSound()`，使`movementType`为"Moo"，并调用`onAnimalMoved()`方法。

1.  重写`onAnimalMoved()`和`inAnimalMadeSound()`方法。

1.  创建一个`main()`来测试代码。

输出应该类似于以下内容：

```java
Animal moved: Walking
Sound made: Move
```

#### 注意

此活动的解决方案可在第 323 页找到。

## 类型转换

我们已经看到，当我们写`int a = 10`时，`a`是整数数据类型，通常大小为 32 位。当我们写`char c = 'a'`时，`c`的数据类型是字符。这些数据类型被称为原始类型，因为它们可以用来保存简单的信息。

对象也有类型。对象的类型通常是该对象的类。例如，当我们创建一个对象，比如`Doctor myDoctor = new Doctor()`，`myDoctor`对象的类型是`Doctor`。`myDoctor`变量通常被称为引用类型。正如我们之前讨论的那样，这是因为`myDoctor`变量并不持有对象本身。相反，它持有对象在内存中的引用。

类型转换是我们将一个类型转换为另一个类型的一种方式。重要的是要注意，只有属于同一个超类或实现相同接口（统称为类型）的类或接口，即它们具有父子关系，才能被转换或转换为彼此。

让我们回到我们的`Person`例子。我们创建了`Student`类，它继承自这个类。这基本上意味着`Student`类是`Person`家族中的一员，任何从`Person`类继承的其他类也是如此：

![图 5.3：从基类继承子类](img/C09581_05_02.jpg)

###### 图 5.2：从基类继承子类

我们在 Java 中使用对象前使用括号进行类型转换：

```java
Student student = new Student();
Person person = (Person)student;
```

在这个例子中，我们创建了一个名为`student`的`Student`类型的对象。然后，我们通过使用`(Person)student`语句将其转换为`Person`类型。这个语句将`student`标记为`Person`类型，而不是`Student`类型。这种类型的类型转换，即我们将子类标记为超类，称为向上转换。这个操作不会改变原始对象；它只是将其标记为不同的类型。

向上转换减少了我们可以访问的方法的数量。例如，`student`变量不能再访问`Student`类中的方法和字段。

我们通过执行向下转换将`student`转换回`Student`类型：

```java
Student student = new Student();
Person person = (Person)student;
Student newStudent = (Student)person;
```

向下转换是将超类类型转换为子类类型。此操作使我们可以访问子类中的方法和字段。例如，`newStudent`现在可以访问`Student`类中的所有方法。

为了使向下转换起作用，对象必须最初是子类类型。例如，以下操作是不可能的：

```java
Student student = new Student();
Person person = (Person)student;
Lecturer lecturer = (Lecturer) person;
```

如果您尝试运行此程序，您将收到以下异常：

![图 5.4：向下转换时的异常消息](img/C09581_05_03.jpg)

###### 图 5.3：向下转换时的异常消息

这是因为`person`最初不是`Lecturer`类型，而是`Student`类型。我们将在接下来的课程中更多地讨论异常。

为了避免这种类型的异常，您可以使用`instanceof`运算符首先检查对象是否是给定类型：

```java
if (person instanceof  Lecturer) {
  Lecturer lecturer() = (Lecturer) person;
}
```

如果`person`最初是`Lecturer`类型，则`instanceof`运算符返回`true`，否则返回 false。

### 活动 17：使用 instanceof 和类型转换

在以前的活动中，您使用接口声明了有关员工接口的工资和税收的常见方法。随着 JavaWorks 有限公司的扩张，销售人员开始获得佣金。这意味着现在，您需要编写一个新的类：`SalesWithCommission`。这个类将扩展自`Sales`，这意味着它具有员工的所有行为，但还将具有一个额外的方法：`getCommission`。这个新方法返回这个员工的总销售额（将在构造函数中传递）乘以销售佣金，即 15%。

作为这个活动的一部分，您还将编写一个具有生成员工方法的类。这将作为此活动和其他活动的`数据源`。这个`EmployeeLoader`类将有一个方法：`getEmployee()`，它返回一个 Employee。在这个方法中，您可以使用任何方法返回一个新生成的员工。使用`java.util.Random`类可能会帮助您完成这个任务，并且如果需要的话，仍然可以获得一致性。

使用您的数据源和新的`SalesWithCommission`，您将编写一个应用程序，使用`for`循环多次调用`EmployeeLoader.getEmployee`方法。对于每个生成的员工，它将打印他们的净工资和所支付的税款。它还将检查员工是否是`SalesWithCommission`的实例，对其进行转换并打印他的佣金。

完成此活动，您需要：

1.  创建一个`SalesWithCommission`类，它扩展自`Sales`。添加一个接收 double 类型的总销售额并将其存储为字段的构造函数。还添加一个名为`getCommission`的方法，它返回总销售额乘以 15%（0.15）的 double 类型。

1.  创建另一个类，作为数据源，生成员工。这个类有一个名为`getEmployee()`的方法，将创建一个 Employee 实现的实例并返回它。方法的返回类型应该是 Employee。

1.  编写一个应用程序，在`for`循环中重复调用`getEmployee()`并打印有关员工工资和税收的信息。如果员工是`SalesWithCommission`的实例，还要打印他的佣金。

#### 注意

此活动的解决方案可以在第 325 页找到。

## 对象类

Java 提供了一个特殊的类称为`Object`，所有类都隐式继承自它。您不必手动从这个类继承，因为编译器会为您执行。`Object`是所有类的超类：

![](img/C09581_05_04.jpg)

###### 图 5.4：超类 Object

这意味着 Java 中的任何类都可以向上转型为`Object`：

```java
Object object = (Object)person;
Object object1 = (Object)student;
```

同样，您可以向原始类进行向下转换：

```java
Person newPerson = (Person)object;
Student newStudent  = (Student)object1;
```

当您想要传递您不知道类型的对象时，可以使用这个`Object`类。当 JVM 想要执行垃圾回收时，也会使用它。

## 自动装箱和拆箱

有时，我们需要处理只接受对象的方法中的原始类型。一个很好的例子是当我们想要在 ArrayList 中存储整数时（稍后我们将讨论）。这个类`ArrayList`只接受对象，而不是原始类型。幸运的是，Java 提供了所有原始类型作为类。包装类可以保存原始值，我们可以像操作普通类一样操作它们。

`Integer`类的一个示例，它可以保存一个`int`如下：

```java
Integer a = new Integer(1);
```

我们还可以省略`new`关键字，编译器会自动为我们进行包装：

```java
Integer a = 1;
```

然后，我们可以像处理其他对象一样使用这个对象。我们可以将其向上转型为`Object`，然后将其向下转型为`Integer`。

将原始类型转换为对象（引用类型）的操作称为自动装箱。

我们还可以将对象转换回原始类型：

```java
Integer a = 1;
int b = a;
```

这里，将原始类型`b`赋值为`a`的值，即 1。将引用类型转换回原始类型的操作称为拆箱。编译器会自动为我们执行自动装箱和拆箱。

除了`Integer`，Java 还为以下基本类型提供了以下包装类：

![](img/C09581_Table_05_01.jpg)

###### 表 5.1：表示基本类型的包装类的表格

### 活动 18：理解 Java 中的类型转换

场景：让我们使用我们一直在使用的`Animal`类来理解类型转换的概念。

目标：我们将为我们的`Animal`类创建一个测试类，并对`Cow`和`Cat`类进行向上转型和向下转型。

目标：内化类型转换的概念。

这些步骤将帮助您完成此活动：

执行以下步骤：

1.  打开`Animals`项目。

1.  创建一个名为`AnimalTest`的新类，并在其中创建`main`方法

1.  在`main()`方法中创建`Cat`和`Cow`类的对象。

1.  打印 Cat 对象的所有者。

1.  将`Cat`类的对象向上转型为`Animal`，并尝试再次打印所有者。注意错误。

1.  打印 Cow 类的对象的声音。

1.  将`Cow`类的对象向上转型为`Animal`，并尝试再次打印所有者。注意错误。

1.  将 Animal 类的对象向下转型为 Cat 类的新对象，并再次打印所有者。

输出应该类似于这样：

![图 5.8：AnimalTest 类的输出](img/C09581_05_05.jpg)

###### 图 5.5：AnimalTest 类的输出

#### 注意

此活动的解决方案可以在第 327 页找到。

## 抽象类和方法

早些时候，我们讨论了接口以及当我们希望与我们的类在它们必须实现的方法上有一个合同时，它们可以是有用的。然后我们看到了我们只能转换共享相同层次树的类。

Java 还允许我们拥有具有抽象方法的类，所有从它继承的类必须实现这些方法。这样的类在访问修饰符之后被称为`abstract`关键字。

当我们将一个类声明为`abstract`时，从它继承的任何类必须在其中实现`abstract`方法。我们不能实例化抽象类：

```java
public abstract class AbstractPerson {
     //this class is abstract and cannot be instantiated
}
```

因为`abstract`类首先仍然是类，它们可以有自己的逻辑和状态。这使它们比方法为空的接口具有更多的优势。此外，一旦我们从`abstract`类继承，我们可以沿着该类层次结构执行类型转换。

Java 还允许我们拥有`abstract`方法，必须声明为`abstract`。

我们在访问修饰符之后使用`abstract`关键字来声明一个方法为`abstract`。

当我们从一个`abstract`类继承时，我们必须在其中实现所有的`abstract`方法：

```java
public class SubClass extends  AbstractPerson {
       //TODO: implement all methods in AbstractPerson
}
```

### 活动 19：在 Java 中实现抽象类和方法

场景：想象一下，当地医院委托您构建一款软件来管理使用该设施的不同类型的人。您必须找到一种方式来代表医生、护士和患者。

目标：我们将创建三个类：一个是抽象类，代表任何人，另一个代表医生，最后一个代表患者。所有的类都将继承自抽象人类。

目标：了解 Java 中`abstract`类和方法的概念。

这些步骤将帮助您完成此活动：

1.  创建一个名为`Hospital`的新项目并打开它。

1.  在`src`文件夹中，创建一个名为`Person`的抽象类：

```java
public abstract class Patient {
}
```

1.  创建一个返回医院中人员类型的`abstract`方法。将此方法命名为 String `getPersonType()`，返回一个字符串：

```java
public abstract String getPersonType();
```

我们已经完成了我们的`abstract`类和方法。现在，我们将继续从中继承并实现这个`abstract`方法。

1.  创建一个名为`Doctor`的新类，它继承自`Person`类：

```java
public class Doctor extends Patient {
}
```

1.  在我们的`Doctor`类中重写`getPersonType`抽象方法。返回"`Arzt`"字符串。这是医生的德语名称：

```java
@Override
public String getPersonType() {
   return "Arzt";
}
```

1.  创建另一个名为`Patient`的类来代表医院里的病人。同样，确保该类继承自`Person`并重写`getPersonType`方法。返回"`Kranke`"。这是德语中的病人：

```java
public class People extends Patient{
   @Override
   public String getPersonType() {
       return "Kranke";
   }
}
```

现在我们有了两个类，我们将使用第三个测试类来测试我们的代码。

1.  创建一个名为`HospitalTest`的第三个类。我们将使用这个类来测试之前创建的两个类。

1.  在`HospitalTest`类中，创建`main`方法：

```java
public class HospitalTest {
   public static void main(String[] args){

   }
}
```

1.  在`main`方法中，创建一个`Doctor`的实例和一个`Patient`的实例：

```java
Doctor doctor = new Doctor();
People people = new People();
```

1.  尝试为每个对象调用`getPersonType`方法并将其打印到控制台上。输出是什么？

```java
String str = doctor.getPersonType();
String str1 = patient.getPersonType();
System.out.println(str);
System.out.println(str1);
```

输出如下：

![](img/C09581_05_06.jpg)

###### 图 5.6：调用 getPersonType()的输出

#### 注意

此活动的解决方案可在第 329 页找到。

### 活动 20：使用抽象类封装公共逻辑

JavaWorks 不断发展。现在他们有了许多员工，他们注意到之前构建的应用程序不支持工资变化。到目前为止，每个工程师的工资都必须与其他人相同。经理、销售和带佣金的销售人员也是如此。为了解决这个问题，您将使用一个封装根据税收计算净工资的逻辑的抽象类。为了使其工作，抽象类将有一个接收总工资的构造函数。它不会实现`getTax()`方法，而是将其委托给子类。使用接收总工资作为构造函数参数的新通用员工的子类。

您还将在`EmployeeLoader`中添加一个新方法`getEmployeeWithSalary()`，它将生成一个新的通用员工，并随机生成总工资。

最后，在您的应用程序中，您将像以前一样，打印工资信息和税，如果员工是`GenericSalesWithCommission`的实例，还要打印他的佣金。

要完成此活动，您需要：

1.  创建一个抽象类`GenericEmployee`，它有一个接收总工资并将其存储在字段中的构造函数。它应该实现 Employee 接口并有两个方法：`getGrossSalary()`和`getNetSalary()`。第一个方法只会返回传入构造函数的值。后者将返回总工资减去调用`getTax()`方法的结果。

1.  为每种类型的员工创建一个新的通用版本：`GenericEngineer`、`GenericManager`、`GenericSales`和`GenericSalesWithCommission`。它们都需要一个接收总工资并将其传递给超级构造函数的构造函数。它们还需要实现`getTax()`方法，返回每个类的正确税值。记得在`GenericSalesWithCommission`类中也接收总销售额，并添加计算佣金的方法。

1.  在`EmployeeLoader`类中添加一个新方法`getEmployeeWithSalary`。这个方法将在返回之前为新创建的员工生成一个介于 70,000 和 120,000 之间的随机工资。在创建`GenericSalesWithCommission`员工时，也记得提供一个总销售额。

1.  编写一个应用程序，从`for`循环内多次调用`getEmployeeWithSalary`方法。这个方法将像前一个活动中一样工作：打印所有员工的净工资和税。如果员工是`GenericSalesWithCommission`的实例，还要打印他的佣金。

#### 注意

此活动的解决方案可在第 331 页找到。

## 总结

在这节课中，我们学到了接口是一种定义一组方法的方式，所有实现它们的类必须提供特定的实现。接口可以用于在代码中实现事件和监听器，当特定动作发生时。

然后我们了解到，类型转换是一种让我们将一个类型的变量改变为另一个类型的方法，只要它们在同一层次树上或实现了一个共同的接口。

我们还研究了在 Java 中使用`instanceof`运算符和`Object`类，并学习了自动装箱、拆箱、抽象类和抽象方法的概念。

在下一课中，我们将研究一些 Java 中附带的常见类和数据结构。
