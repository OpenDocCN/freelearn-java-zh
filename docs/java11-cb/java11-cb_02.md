# 第二章：OOP 的快速通道-类和接口

在本章中，我们将涵盖以下示例：

+   实现**面向对象设计**（**OOD**）

+   使用内部类

+   使用继承和聚合

+   编码到一个接口

+   创建具有默认和静态方法的接口

+   创建具有私有方法的接口

+   使用`Optional`更好地处理空值

+   使用实用类`Objects`

本章的示例不需要对 OOD 有任何先前的了解。但是，在 Java 中编写代码的一些经验将是有益的。本章中的代码示例完全可用，并与 Java 11 兼容。为了更好地理解，我们建议您尝试运行所呈现的示例。

我们还鼓励您根据您团队的经验，将本章中的提示和建议调整到您的需求中。考虑与同事分享您新获得的知识，并讨论所描述的原则如何应用到您的领域和当前项目中。

# 介绍

本章为您快速介绍了**面向对象编程**（**OOP**）的概念，并涵盖了自 Java 8 以来引入的一些增强功能。我们还将尝试在适用的地方涵盖一些良好的 OOD 实践，并使用具体的代码示例加以演示。

一个人可以花费很多时间阅读关于 OOD 的文章和实用建议，无论是在书籍上还是在互联网上。对一些人来说，这样做可能是有益的。但是根据我们的经验，掌握 OOD 的最快方法是在自己的代码中尝试其原则。这正是本章的目标——让您有机会看到和使用 OOD 原则，以便立即理解其正式定义。

良好编写代码的主要标准之一是意图的清晰。良好的动机和清晰的设计有助于实现这一点。代码由计算机运行，但由人类维护——阅读和修改。牢记这一点将确保您的代码的长期性，甚至可能会得到一些后来处理它的人的感激和赞赏。

在本章中，您将学习如何使用五个基本的 OOP 概念：

+   **对象/类**：将数据和方法放在一起

+   **封装**：隐藏数据和/或方法

+   **继承**：扩展另一个类的数据和/或方法

+   **接口**：隐藏实现和为一种类型编码

+   **多态**：使用指向子类对象的基类类型引用

如果您在互联网上搜索，您可能会注意到许多其他概念和对它们的补充，以及所有 OOD 原则，都可以从前面列出的五个概念中推导出来。这意味着对它们的扎实理解是设计面向对象系统的先决条件。

# 实现面向对象设计（OOD）

在这个示例中，您将学习前两个 OOP 概念——对象/类和封装。这些概念是 OOD 的基础。

# 准备就绪

术语*对象*通常指的是将数据和可以应用于这些数据的过程耦合在一起的实体。数据和过程都不是必需的，但其中一个是——通常情况下，两者都是——总是存在的。数据称为对象字段（或属性），而过程称为方法。字段值描述了对象的*状态*。方法描述了对象的*行为*。每个对象都有一个类型，由其类——用于对象创建的模板——定义。对象也被称为类的实例。

*类*是字段和方法的定义集合，这些字段和方法将存在于基于该类创建的每个实例中。

封装是隐藏那些不应该被其他对象访问的字段和方法。

封装是通过在字段和方法的声明中使用`public`、`protected`或`private` Java 关键字，称为*访问修饰符*来实现的。当未指定访问修饰符时，还有一种默认级别的封装。

# 如何做...

1.  创建一个带有`horsePower`字段的`Engine`类。添加`setHorsePower(int horsePower)`方法，用于设置该字段的值，以及`getSpeedMph(double timeSec, int weightPounds)`方法，用于根据车辆开始移动以来经过的时间、车辆重量和发动机功率计算车辆的速度：

```java
public class Engine { 
  private int horsePower; 
  public void setHorsePower(int horsePower) { 
     this.horsePower = horsePower; 
  } 
  public double getSpeedMph(double timeSec, int weightPounds){ 
    double v = 2.0 * this.horsePower * 746 * timeSec * 
                                       32.17 / weightPounds; 
    return Math.round(Math.sqrt(v) * 0.68); 
 } 
}
```

1.  创建`Vehicle`类：

```java
      public class Vehicle { 
          private int weightPounds; 
          private Engine engine; 
          public Vehicle(int weightPounds, Engine engine) { 
            this.weightPounds = weightPounds; 
            this.engine = engine; 
          } 
          public double getSpeedMph(double timeSec){ 
            return this.engine.getSpeedMph(timeSec, weightPounds); 
         } 
     } 
```

1.  创建将使用前述类的应用程序：

```java
public static void main(String... arg) { 
   double timeSec = 10.0; 
   int horsePower = 246; 
   int vehicleWeight = 4000;  
   Engine engine = new Engine(); 
   engine.setHorsePower(horsePower); 
   Vehicle vehicle = new Vehicle(vehicleWeight, engine); 
   System.out.println("Vehicle speed (" + timeSec + " sec)=" 
                   + vehicle.getSpeedMph(timeSec) + " mph"); 
 } 
```

正如你所看到的，`engine`对象是通过调用`Engine`类的默认构造函数而创建的，该构造函数没有参数，并且使用`new` Java 关键字在堆上为新创建的对象分配内存。

第二个对象`vehicle`是使用`Vehicle`类的显式定义的带有两个参数的构造函数创建的。构造函数的第二个参数是`engine`对象，它携带了`horsePower`值，使用`setHorsePower(int horsePower)`方法设置为`246`。

`engine`对象包含`getSpeedMph(double timeSec, int weightPounds)`方法，可以被任何对象调用（因为它是`public`），就像在`Vehicle`类的`getSpeedMph(double timeSec)`方法中所做的那样。

# 它是如何工作的...

前述应用程序产生以下输出：

![](img/5e877a71-e07d-4cdf-a2ba-4d06cf4abe6a.png)

值得注意的是，`Vehicle`类的`getSpeedMph(double timeSec)`方法依赖于为`engine`字段分配的值的存在。这样，`Vehicle`类的对象*委托*速度计算给`Engine`类的对象。如果后者未设置（例如在`Vehicle()`构造函数中传递了`null`），将在运行时抛出`NullPointerException`，如果应用程序未处理，将被 JVM 捕获并强制其退出。为了避免这种情况，我们可以在`Vehicle()`构造函数中放置一个检查，检查`engine`字段值的存在：

```java
if(engine == null){ 
   throw new RuntimeException("Engine" + " is required parameter."); 
}   
```

或者，我们可以在`Vehicle`类的`getSpeedMph(double timeSec)`方法中放置一个检查：

```java
if(getEngine() == null){ 
  throw new RuntimeException("Engine value is required."); 
} 
```

这样，我们避免了`NullPointerException`的歧义，并告诉用户问题的确切来源。

正如你可能已经注意到的，`getSpeedMph(double timeSec, int weightPounds)`方法可以从`Engine`类中移除，并且可以完全在`Vehicle`类中实现：

```java
public double getSpeedMph(double timeSec){
  double v =  2.0 * this.engine.getHorsePower() * 746 * 
                                timeSec * 32.17 / this.weightPounds;
  return Math.round(Math.sqrt(v) * 0.68);
}
```

为此，我们需要在`Engine`类中添加`getHorsePower()`公共方法，以便使其可以被`Vehicle`类的`getSpeedMph(double timeSec)`方法使用。目前，我们将`getSpeedMph(double timeSec, int weightPounds)`方法留在`Engine`类中。

这是你需要做出的设计决策之一。如果你认为`Engine`类的对象将被传递并被不同类的对象使用（不仅仅是`Vehicle`），那么你需要在`Engine`类中保留`getSpeedMph(double timeSec, int weightPounds)`方法。否则，如果你认为只有`Vehicle`类将负责速度计算（这是有道理的，因为这是车辆的速度，而不是发动机的速度），你应该在`Vehicle`类中实现这个方法。

# 还有更多...

Java 提供了扩展类的能力，并允许子类访问基类的所有非私有字段和方法。例如，你可以决定每个可以被询问其速度的对象都属于从`Vehicle`类派生的子类。在这种情况下，`Car`类可能如下所示：

```java
public class Car extends Vehicle {
  private int passengersCount;
  public Car(int passengersCount, int weightPounds, Engine engine){
    super(weightPounds, engine);
    this.passengersCount = passengersCount;
  }
  public int getPassengersCount() {
    return this.passengersCount;
  }
}
```

现在，我们可以通过用`Car`类的对象替换`Vehicle`类对象来更改我们的测试代码：

```java
public static void main(String... arg) { 
  double timeSec = 10.0; 
  int horsePower = 246; 
  int vehicleWeight = 4000; 
  Engine engine = new Engine(); 
  engine.setHorsePower(horsePower); 
  Vehicle vehicle = new Car(4, vehicleWeight, engine); 
  System.out.println("Car speed (" + timeSec + " sec) = " + 
                             vehicle.getSpeedMph(timeSec) + " mph"); 
} 
```

当执行前面的代码时，它产生与`Vehicle`类对象相同的值：

![](img/2366a79d-188e-42bb-87d8-accfac3a3c97.png)

由于多态性，对`Car`类对象的引用可以赋给其基类`Vehicle`的引用。`Car`类对象有两种类型——它自己的类型`Car`和基类`Vehicle`的类型。

在 Java 中，一个类也可以实现多个接口，这样类的对象也会有每个实现接口的类型。我们将在随后的配方中讨论这一点。

# 使用内部类

在这个配方中，您将了解三种内部类的类型：

+   **内部类**：这是一个在另一个（封闭）类内部定义的类。它的可访问性由`public`、`protected`和`private`访问修饰符调节。内部类可以访问封闭类的私有成员，封闭类也可以访问其内部类的私有成员，但是无法从封闭类外部访问私有内部类或非私有内部类的私有成员。

+   **方法局部内部类**：这是一个在方法内部定义的类。它的可访问性受限于方法内部。

+   **匿名内部类**：这是一个没有声明名称的类，在对象实例化时基于接口或扩展类定义。

# 准备就绪

当一个类只被一个其他类使用时，设计者可能会决定不需要将这样的类设为公共类。例如，假设`Engine`类只被`Vehicle`类使用。

# 如何做...

1.  将`Engine`类创建为`Vehicle`类的内部类：

```java
        public class Vehicle {
          private int weightPounds;
          private Engine engine;
          public Vehicle(int weightPounds, int horsePower) {
            this.weightPounds = weightPounds;
            this.engine = new Engine(horsePower);
          }
          public double getSpeedMph(double timeSec){
            return this.engine.getSpeedMph(timeSec);
          }
          private int getWeightPounds(){ return weightPounds; }
          private class Engine {
            private int horsePower;
            private Engine(int horsePower) {
              this.horsePower = horsePower;
            }
            private double getSpeedMph(double timeSec){
              double v = 2.0 * this.horsePower * 746 * 
                         timeSec * 32.17 / getWeightPounds();
              return Math.round(Math.sqrt(v) * 0.68);
            }
          }
        }
```

1.  请注意，`Vehicle`类的`getSpeedMph(double timeSec)`方法可以访问`Engine`类，即使它被声明为`private`。它甚至可以访问`Engine`类的`getSpeedMph(double timeSec)`私有方法。内部类也可以访问封闭类的所有私有元素。这就是为什么`Engine`类的`getSpeedMph(double timeSec)`方法可以访问封闭`Vehicle`类的私有`getWeightPounds()`方法。

1.  更仔细地看一下内部`Engine`类的用法。只使用了`Engine`类的`getSpeedMph(double timeSec)`方法。如果设计者认为将来也会是这种情况，他们可能会合理地决定将`Engine`类设为方法局部内部类，这是内部类的第二种类型：

```java
        public class Vehicle {
          private int weightPounds;
          private int horsePower;
          public Vehicle(int weightPounds, int horsePower) {
            this.weightPounds = weightPounds;
            this.horsePower = horsePower;
          }
          private int getWeightPounds() { return weightPounds; }
          public double getSpeedMph(double timeSec){
            class Engine {
              private int horsePower;
              private Engine(int horsePower) {
                this.horsePower = horsePower;
              }
              private double getSpeedMph(double timeSec){
                double v = 2.0 * this.horsePower * 746 * 
                          timeSec * 32.17 / getWeightPounds();
                return Math.round(Math.sqrt(v) * 0.68);
              }
            }
            Engine engine = new Engine(this.horsePower);
            return engine.getSpeedMph(timeSec);
          }
        }
```

在前面的代码示例中，根本没有必要有一个`Engine`类。速度计算公式可以直接使用，而不需要`Engine`类的介入。但也有一些情况下可能不那么容易做到。例如，方法局部内部类可能需要扩展其他类以继承其功能，或者创建的`Engine`对象可能需要经过一些转换，因此需要创建。其他考虑可能需要方法局部内部类。

无论如何，将不需要从封闭类外部访问的所有功能设为不可访问是一个好的做法。封装——隐藏对象的状态和行为——有助于避免意外更改或覆盖对象行为导致的意外副作用。这使得结果更加可预测。这就是为什么一个好的设计只暴露必须从外部访问的功能。通常是封闭类的功能首先促使类的创建，而不是内部类或其他实现细节。

# 它是如何工作的...

无论`Engine`类是作为内部类还是方法局部内部类实现的，测试代码看起来都是一样的：

```java
public static void main(String arg[]) {
  double timeSec = 10.0;
  int engineHorsePower = 246;
  int vehicleWeightPounds = 4000;
  Vehicle vehicle = 
          new Vehicle(vehicleWeightPounds, engineHorsePower);
  System.out.println("Vehicle speed (" + timeSec + " sec) = " 
                    + vehicle.getSpeedMph(timeSec) + " mph");
}
```

如果我们运行前面的程序，我们会得到相同的输出：

![](img/23149b17-7ecf-404f-ae5b-77abdb9deb27.png)

现在，假设我们需要测试`getSpeedMph()`方法的不同实现：

```java
public double getSpeedMph(double timeSec){ return -1.0d; }
```

如果这个速度计算公式对你来说没有意义，那么你是正确的，它确实没有意义。我们这样做是为了使结果可预测，并且与先前实现的结果不同。

有许多方法可以引入这个新的实现。例如，我们可以改变`Engine`类中`getSpeedMph(double timeSec)`方法的代码。或者，我们可以改变`Vehicle`类中相同方法的实现。

在这个示例中，我们将使用第三种内部类，称为匿名内部类。当你想尽可能少地编写新代码，或者你想通过临时覆盖旧代码来快速测试新行为时，这种方法特别方便。匿名类的使用将如下所示：

```java
public static void main(String... arg) {
  double timeSec = 10.0;
  int engineHorsePower = 246;
  int vehicleWeightPounds = 4000;
  Vehicle vehicle = 
    new Vehicle(vehicleWeightPounds, engineHorsePower) {
        public double getSpeedMph(double timeSec){ 
           return -1.0d;
        }
    };
  System.out.println("Vehicle speed (" + timeSec + " sec) = "
                    + vehicle.getSpeedMph(timeSec) + " mph");
}
```

如果我们运行这个程序，结果将是这样的：

![](img/c6f8fa7d-3a9c-416c-a8ff-c78167eaf68e.png)

正如你所看到的，匿名类实现已经覆盖了`Vehicle`类的实现。新的匿名类中只有一个方法——`getSpeedMph()`方法，它返回了硬编码的值。但我们也可以覆盖`Vehicle`类的其他方法或者添加新的方法。我们只是想为了演示目的保持示例简单。

根据定义，匿名内部类必须是语句的一部分，该语句以分号结束（与任何语句一样）。这样的表达式由以下部分组成：

+   `new`操作符

+   实现的接口或扩展类的名称后跟括号`()`，表示默认构造函数或扩展类的构造函数（后者是我们的情况，扩展类是`Vehicle`）

+   类体与方法

像任何内部类一样，匿名内部类可以访问外部类的任何成员，但有一个注意事项——要被内部匿名类使用，外部类的字段必须要么声明为`final`，要么隐式地变为`final`，这意味着它们的值不能被改变。一个好的现代 IDE 会在你试图改变这样的值时警告你违反了这个约束。

使用这些特性，我们可以修改我们的示例代码，并为新实现的`getSpeedMph(double timeSec)`方法提供更多的输入数据，而无需将它们作为方法参数传递：

```java
public static void main(String... arg) {
  double timeSec = 10.0;
  int engineHorsePower = 246;
  int vehicleWeightPounds = 4000;
  Vehicle vehicle = 
    new Vehicle(vehicleWeightPounds, engineHorsePower){
      public double getSpeedMph(double timeSec){
        double v = 2.0 * engineHorsePower * 746 * 
             timeSec * 32.17 / vehicleWeightPounds;
        return Math.round(Math.sqrt(v) * 0.68);
      }
    };
  System.out.println("Vehicle speed (" + timeSec + " sec) = " 
                    + vehicle.getSpeedMph(timeSec) + " mph");
}
```

请注意，`timeSec`、`engineHorsePower`和`vehicleWeightPounds`变量可以被内部类的`getSpeedMph(double timeSec)`方法访问，但不能被修改。如果我们运行上述代码，结果将与之前一样：

![](img/3b0f5b59-fcd0-4847-8141-cb56a9c81995.png)

在只有一个抽象方法的接口（称为函数式接口）的情况下，可以使用另一种构造，称为*lambda 表达式*，而不是匿名内部类。它提供了更简洁的表示。我们将在第四章 *进入函数式*中讨论函数式接口和 lambda 表达式。

# 还有更多...

内部类是一个非静态嵌套类。Java 还允许我们创建一个静态嵌套类，当内部类不需要访问外部类的非静态字段和方法时可以使用。下面是一个示例（`Engine`类中添加了`static`关键字）：

```java
public class Vehicle {
  private Engine engine;
  public Vehicle(int weightPounds, int horsePower) {
    this.engine = new Engine(horsePower, weightPounds)
  }
  public double getSpeedMph(double timeSec){
    return this.engine.getSpeedMph(timeSec);
  }
  private static class Engine {
    private int horsePower;
    private int weightPounds;
    private Engine(int horsePower, int weightPounds) {
      this.horsePower = horsePower;
    }
    private double getSpeedMph(double timeSec){
      double v = 2.0 * this.horsePower * 746 * 
                       timeSec * 32.17 / this.weightPounds;
      return Math.round(Math.sqrt(v) * 0.68);
    }
  }
}
```

由于静态类无法访问非静态成员，我们被迫在构造`Engine`类时传递重量值，并且我们移除了`getWeightPounds()`方法，因为它不再需要了。

# 使用继承和聚合

在这个示例中，你将学习更多关于两个重要的面向对象编程概念，继承和多态，这些概念已经在前面的示例中提到并被使用。结合聚合，这些概念使设计更具可扩展性。

# 准备就绪

继承是一个类获取另一个类的非私有字段和方法的能力。

扩展的类称为基类、超类或父类。类的新扩展称为子类或子类。

多态性是使用基类类型引用其子类对象的能力。

为了演示继承和多态的威力，让我们创建代表汽车和卡车的类，每个类都有它可以达到的重量、发动机功率和速度（作为时间函数）的最大载荷。此外，这种情况下的汽车将以乘客数量为特征，而卡车的重要特征将是其有效载荷。

# 如何做...

1.  看看`Vehicle`类：

```java
        public class Vehicle {
          private int weightPounds, horsePower;
          public Vehicle(int weightPounds, int horsePower) {
            this.weightPounds = weightPounds;
            this.horsePower = horsePower;
          }
          public double getSpeedMph(double timeSec){
            double v = 2.0 * this.horsePower * 746 * 
                     timeSec * 32.17 / this.weightPounds;
            return Math.round(Math.sqrt(v) * 0.68);
          }
        }
```

`Vehicle`类中实现的功能不特定于汽车或卡车，因此将这个类用作`Car`和`Truck`类的基类是有意义的，这样每个类都可以将这个功能作为自己的功能。

1.  创建`Car`类：

```java
        public class Car extends Vehicle {
          private int passengersCount;
          public Car(int passengersCount, int weightPounds, 
                                             int horsepower){
            super(weightPounds, horsePower);
            this.passengersCount = passengersCount;
          }
          public int getPassengersCount() { 
            return this.passengersCount; 
          }
        }
```

1.  创建`Truck`类：

```java
         public class Truck extends Vehicle {
           private int payload;
           public Truck(int payloadPounds, int weightPounds, 
                                              int horsePower){
             super(weightPounds, horsePower);
             this.payload = payloadPounds;
           }
           public int getPayload() { 
             return this.payload; 
           }
         }
```

由于`Vehicle`基类既没有隐式构造函数也没有没有参数的显式构造函数（因为我们选择只使用带参数的显式构造函数），所以我们必须在`Vehicle`类的每个子类的构造函数的第一行调用基类构造函数`super()`。

# 它是如何工作的...

让我们编写一个测试程序：

```java
public static void main(String... arg) {
  double timeSec = 10.0;
  int engineHorsePower = 246;
  int vehicleWeightPounds = 4000;
  Vehicle vehicle = new Car(4, vehicleWeightPounds, engineHorsePower);
  System.out.println("Passengers count=" + 
                                 ((Car)vehicle).getPassengersCount());
  System.out.println("Car speed (" + timeSec + " sec) = " + 
                               vehicle.getSpeedMph(timeSec) + " mph");
  vehicle = new Truck(3300, vehicleWeightPounds, engineHorsePower);
  System.out.println("Payload=" + 
                           ((Truck)vehicle).getPayload() + " pounds");
  System.out.println("Truck speed (" + timeSec + " sec) = " + 
                               vehicle.getSpeedMph(timeSec) + " mph");
}
```

注意，`Vehicle`类型的`vehicle`引用指向`Car`子类的对象，稍后指向`Truck`子类的对象。这是由多态性实现的，根据多态性，对象具有其继承线中每个类的类型，包括所有接口。

如果需要调用仅存在于子类中的方法，必须将这样的引用转换为子类类型，就像在前面的示例中所做的那样。

前面代码的结果如下：

![](img/d383484b-d020-48a8-b025-444f48b764a4.png)

我们不应该对看到相同的速度计算结果感到惊讶，因为相同的重量和发动机功率用于计算每个车辆的速度。但是，直观上，我们感觉到一个装载重的卡车不应该能够在相同的时间内达到与汽车相同的速度。为了验证这一点，我们需要在速度的计算中包括汽车的总重量（乘客和行李）和卡车的总重量（有效载荷）。一种方法是在每个子类中覆盖`Vehicle`基类的`getSpeedMph(double timeSec)`方法。

我们可以在`Car`类中添加`getSpeedMph(double timeSec)`方法，它将覆盖基类中具有相同签名的方法。这个方法将使用特定于汽车的重量计算：

```java
public double getSpeedMph(double timeSec) {
  int weight = this.weightPounds + this.passengersCount * 250;
  double v = 2.0 * this.horsePower * 746 * timeSec * 32.17 / weight;
  return Math.round(Math.sqrt(v) * 0.68);
}
```

在前面的代码中，我们假设一个带行李的乘客平均重量为`250`磅。

类似地，我们可以在`Truck`类中添加`getSpeedMph(double timeSec)`方法：

```java
public double getSpeedMph(double timeSec) {
  int weight = this.weightPounds + this.payload;
  double v = 2.0 * this.horsePower * 746 * timeSec * 32.17 / weight;
  return Math.round(Math.sqrt(v) * 0.68);
}
```

对这些修改的结果（如果我们运行相同的测试类）将如下：

![](img/2063b24d-ad9e-45ad-9fe8-387b077e2c86.png)

结果证实了我们的直觉——装载完全的汽车或卡车的速度不会达到空车的速度。

子类中的新方法覆盖了`Vehicle`基类的`getSpeedMph(double timeSec)`，尽管我们是通过基类引用来访问它的：

```java
Vehicle vehicle =  new Car(4, vehicleWeightPounds, engineHorsePower);
System.out.println("Car speed (" + timeSec + " sec) = " + 
                              vehicle.getSpeedMph(timeSec) + " mph");
```

覆盖的方法是动态绑定的，这意味着方法调用的上下文是由实际对象的类型决定的。因此，在我们的示例中，引用`vehicle`指向`Car`子类的对象，`vehicle.getSpeedMph(double timeSec)`调用子类的方法，而不是基类的方法。

在这两个新方法中存在明显的代码冗余，我们可以通过在`Vehicle`基类中创建一个方法，然后在每个子类中使用它来重构。

```java
protected double getSpeedMph(double timeSec, int weightPounds) {
  double v = 2.0 * this.horsePower * 746 * 
                              timeSec * 32.17 / weightPounds;
  return Math.round(Math.sqrt(v) * 0.68);
}
```

由于这个方法只被子类使用，它可以是`protected`，因此只能被子类访问。

现在，我们可以更改`Car`类中的`getSpeedMph(double timeSec)`方法，如下所示：

```java
public double getSpeedMph(double timeSec) {
  int weightPounds = this.weightPounds + this.passengersCount * 250;
  return getSpeedMph(timeSec, weightPounds);
}
```

在上述代码中，调用`getSpeedMph(timeSec, weightPounds)`方法时不需要使用`super`关键字，因为这样的签名方法只存在于`Vehicle`基类中，对此没有任何歧义。

`Truck`类的`getSpeedMph(double timeSec)`方法中也可以进行类似的更改：

```java
public double getSpeedMph(double timeSec) {
  int weightPounds = this.weightPounds + this.payload;
  return getSpeedMph(timeSec, weightPounds);
}
```

现在，我们需要通过添加转换来修改测试类，否则会出现运行时错误，因为`Vehicle`基类中不存在`getSpeedMph(double timeSec)`方法：

```java
public static void main(String... arg) {
    double timeSec = 10.0;
    int engineHorsePower = 246;
    int vehicleWeightPounds = 4000;
    Vehicle vehicle = new Car(4, vehicleWeightPounds, 
    engineHorsePower);
    System.out.println("Passengers count=" + 
    ((Car)vehicle).getPassengersCount());
    System.out.println("Car speed (" + timeSec + " sec) = " +
                       ((Car)vehicle).getSpeedMph(timeSec) + " mph");
    vehicle = new Truck(3300, vehicleWeightPounds, engineHorsePower);
    System.out.println("Payload=" + 
                          ((Truck)vehicle).getPayload() + " pounds");
    System.out.println("Truck speed (" + timeSec + " sec) = " + 
                     ((Truck)vehicle).getSpeedMph(timeSec) + " mph");
  }
}
```

正如您所期望的那样，测试类产生相同的值：

![](img/a3b6ad65-2ded-4313-9556-3f48a4060cb4.png)

为了简化测试代码，我们可以放弃转换，改为写以下内容：

```java
public static void main(String... arg) {
  double timeSec = 10.0;
  int engineHorsePower = 246;
  int vehicleWeightPounds = 4000;
  Car car = new Car(4, vehicleWeightPounds, engineHorsePower);
  System.out.println("Passengers count=" + car.getPassengersCount());
  System.out.println("Car speed (" + timeSec + " sec) = " + 
                                  car.getSpeedMph(timeSec) + " mph");
  Truck truck = 
              new Truck(3300, vehicleWeightPounds, engineHorsePower);
  System.out.println("Payload=" + truck.getPayload() + " pounds");
  System.out.println("Truck speed (" + timeSec + " sec) = " + 
                                truck.getSpeedMph(timeSec) + " mph");
}
```

此代码产生的速度值保持不变。

然而，有一种更简单的方法可以实现相同的效果。我们可以将`getMaxWeightPounds()`方法添加到基类和每个子类中。现在`Car`类将如下所示：

```java
public class Car extends Vehicle {
  private int passengersCount, weightPounds;
  public Car(int passengersCount, int weightPounds, int horsePower){
    super(weightPounds, horsePower);
    this.passengersCount = passengersCount;
    this.weightPounds = weightPounds;
  }
  public int getPassengersCount() { 
    return this.passengersCount;
  }
  public int getMaxWeightPounds() {
    return this.weightPounds + this.passengersCount * 250;
  }
}
```

现在`Truck`类的新版本如下所示：

```java
public class Truck extends Vehicle {
  private int payload, weightPounds;
  public Truck(int payloadPounds, int weightPounds, int horsePower) {
    super(weightPounds, horsePower);
    this.payload = payloadPounds;
    this.weightPounds = weightPounds;
  }
  public int getPayload() { return this.payload; }
  public int getMaxWeightPounds() {
    return this.weightPounds + this.payload;
  }
}
```

我们还需要在基类中添加`getMaxWeightPounds()`方法，以便用于速度计算：

```java
public abstract class Vehicle {
  private int weightPounds, horsePower;
  public Vehicle(int weightPounds, int horsePower) {
    this.weightPounds = weightPounds;
    this.horsePower = horsePower;
  }
  public abstract int getMaxWeightPounds();
  public double getSpeedMph(double timeSec){
    double v = 2.0 * this.horsePower * 746 * 
                             timeSec * 32.17 / getMaxWeightPounds();
    return Math.round(Math.sqrt(v) * 0.68);
  }
}
```

向`Vehicle`类添加一个抽象方法`getMaxWeightPounds()`会使该类成为抽象类。这有一个积极的副作用——它强制在每个子类中实现`getMaxWeightPounds()`方法。否则，子类将无法实例化，必须声明为抽象类。

测试类保持不变，产生相同的结果：

![](img/a5729936-bd5b-478f-80f4-4b0f0737bdb9.png)

但是，说实话，我们这样做只是为了演示使用抽象方法和类的一种可能方式。事实上，一个更简单的解决方案是将最大重量作为参数传递给`Vehicle`基类的构造函数。结果类将如下所示：

```java
public class Car extends Vehicle {
  private int passengersCount;
  public Car(int passengersCount, int weightPounds, int horsepower){
    super(weightPounds + passengersCount * 250, horsePower);
    this.passengersCount = passengersCount;
  }
  public int getPassengersCount() { 
    return this.passengersCount; }
}
```

我们将乘客的重量添加到我们传递给超类构造函数的值中；这是这个子类中唯一的变化。`Truck`类中也有类似的变化：

```java
public class Truck extends Vehicle {
  private int payload;
  public Truck(int payloadPounds, int weightPounds, int horsePower) {
    super(weightPounds + payloadPounds, horsePower);
    this.payload = payloadPounds;
  }
  public int getPayload() { return this.payload; }
}
```

`Vehicle`基类与原始的一样：

```java
public class Vehicle {
  private int weightPounds, horsePower;
  public Vehicle(int weightPounds, int horsePower) {
    this.weightPounds = weightPounds;
    this.horsePower = horsePower;
  }
  public double getSpeedMph(double timeSec){
    double v = 2.0 * this.horsePower * 746;
    v = v * timeSec * 32.174 / this.weightPounds;
    return Math.round(Math.sqrt(v) * 0.68);
  }
}
```

测试类不会改变，并且产生相同的结果：

![](img/fda2f8c4-58be-44e1-afd1-02c378f1b7a0.png)

这个最后的版本——将最大重量传递给基类的构造函数——现在将成为进一步代码演示的起点。

# 聚合使设计更具可扩展性

在上面的例子中，速度模型是在`Vehicle`类的`getSpeedMph(double timeSec)`方法中实现的。如果我们需要使用不同的速度模型（例如包含更多输入参数并且更适合特定驾驶条件），我们需要更改`Vehicle`类或创建一个新的子类来覆盖该方法。在需要尝试几十甚至数百种不同模型的情况下，这种方法变得不可行。

此外，在现实生活中，基于机器学习和其他先进技术的建模变得如此复杂和专业化，以至于汽车加速的建模通常是由不同的团队完成的，而不是组装车辆模型的团队。

为了避免子类的激增和车辆构建者与速度模型开发者之间的代码合并冲突，我们可以使用聚合创建一个更具可扩展性的设计。

聚合是一种面向对象设计原则，用于使用不属于继承层次结构的类的行为来实现必要的功能。该行为可以独立于聚合功能存在。

我们可以将速度计算封装在`SpeedModel`类的`getSpeedMph(double timeSec)`方法中：

```java
public class SpeedModel{
  private Properties conditions;
  public SpeedModel(Properties drivingConditions){
    this.drivingConditions = drivingConditions;
  }
  public double getSpeedMph(double timeSec, int weightPounds,
                                               int horsePower){
    String road = 
         drivingConditions.getProperty("roadCondition","Dry");
    String tire = 
         drivingConditions.getProperty("tireCondition","New");
    double v = 2.0 * horsePower * 746 * timeSec * 
                                         32.17 / weightPounds;
    return Math.round(Math.sqrt(v)*0.68)-road.equals("Dry")? 2 : 5) 
                                       -(tire.equals("New")? 0 : 5);
  }
}
```

可以创建这个类的对象，然后将其设置为`Vehicle`类字段的值：

```java
public class Vehicle {
   private SpeedModel speedModel;       
   private int weightPounds, horsePower;
   public Vehicle(int weightPounds, int horsePower) {
      this.weightPounds = weightPounds;
      this.horsePower = horsePower;
   }
   public void setSpeedModel(SpeedModel speedModel){
      this.speedModel = speedModel;
   }
   public double getSpeedMph(double timeSec){
      return this.speedModel.getSpeedMph(timeSec,
                       this.weightPounds, this.horsePower);
   }
}
```

测试类的更改如下：

```java
public static void main(String... arg) {
  double timeSec = 10.0;
  int horsePower = 246;
  int vehicleWeight = 4000;
  Properties drivingConditions = new Properties();
  drivingConditions.put("roadCondition", "Wet");
  drivingConditions.put("tireCondition", "New");
  SpeedModel speedModel = new SpeedModel(drivingConditions);
  Car car = new Car(4, vehicleWeight, horsePower);
  car.setSpeedModel(speedModel);
  System.out.println("Car speed (" + timeSec + " sec) = " + 
                         car.getSpeedMph(timeSec) + " mph");
}
```

上述代码的结果如下：

![](img/97d4f021-cc4a-4235-bae2-4388a0c31ba8.png)

我们将速度计算功能隔离到一个单独的类中，现在可以修改或扩展它，而不需要改变`Vehicle`继承层次结构中的任何类。这就是聚合设计原则允许您在不改变实现的情况下改变行为的方式。

在下一个示例中，我们将向您展示接口的面向对象编程概念如何释放出更多聚合和多态的力量，使设计更简单，甚至更具表现力。

# 编码到接口

在这个示例中，您将学习面向对象编程概念中的最后一个接口，并进一步练习聚合和多态的使用，以及内部类和继承。

# 准备工作

接口定义了一个类中可以期望看到的方法的签名。它是对客户端可访问的功能的公共界面，因此通常被称为**应用程序编程接口**（**API**）。它支持多态和聚合，并促进更灵活和可扩展的设计。

接口是隐式抽象的，这意味着它不能被实例化。不能仅基于接口创建对象，而不实现它。它仅用于包含抽象方法（无方法体）。但自 Java 8 以来，可以向接口添加默认和私有方法，这是我们将在以下示例中讨论的功能。

每个接口可以扩展多个其他接口，并且类继承类似，继承所有扩展接口的默认和抽象方法。静态成员不能被继承，因为它们属于特定接口。

# 如何做...

1.  创建描述 API 的接口：

```java
public interface SpeedModel {
   double getSpeedMph(double timeSec, int weightPounds, 
                                            int horsePower);
 }
 public interface Vehicle {
   void setSpeedModel(SpeedModel speedModel);
   double getSpeedMph(double timeSec);
 }
 public interface Car extends Vehicle {
   int getPassengersCount();
 }
 public interface Truck extends Vehicle {
   int getPayloadPounds();
 }
```

1.  使用工厂，这些工厂是生成实现特定接口的对象的类。工厂隐藏了实现的细节，因此客户端只处理接口。当实例创建需要复杂的过程和/或大量代码重复时，这是特别有帮助的。在我们的情况下，有一个`FactoryVehicle`类是有意义的，它创建实现`Vehicle`、`Car`或`Truck`接口的类的对象。我们还将创建`FactorySpeedModel`类，它生成实现`SpeedModel`接口的类的对象。这样的 API 允许我们编写以下代码：

```java
public static void main(String... arg) {
   double timeSec = 10.0;
   int horsePower = 246;
   int vehicleWeight = 4000;
   Properties drivingConditions = new Properties();
   drivingConditions.put("roadCondition", "Wet");
   drivingConditions.put("tireCondition", "New");
   SpeedModel speedModel  = FactorySpeedModel.
                 generateSpeedModel(drivingConditions);
   Car car = FactoryVehicle.
                buildCar(4, vehicleWeight, horsePower);
   car.setSpeedModel(speedModel);
   System.out.println("Car speed (" + timeSec + " sec) = " 
                      + car.getSpeedMph(timeSec) + " mph");
}
```

1.  请注意，代码行为与先前示例中的相同：

![](img/350a455b-060d-47ea-ac7a-b0b904f5f119.png)

然而，设计更具可扩展性。

# 它是如何工作的...

我们已经看到了`SpeedModel`接口的一个可能的实现。这是另一种通过在`FactorySpeedModel`类内聚`SpeedModel`类型对象来实现的方法：

```java
public class FactorySpeedModel {
  public static SpeedModel generateSpeedModel(
  Properties drivingConditions){
    //if drivingConditions includes "roadCondition"="Wet"
    return new SpeedModelWet(...);
    //if drivingConditions includes "roadCondition"="Dry"
    return new SpeedModelDry(...);
  }
  private class SpeedModelWet implements SpeedModel{
    public double getSpeedMph(double timeSec, int weightPounds, 
                                                     int horsePower){
       //method code goes here
    }
  }
  private class SpeedModelDry implements SpeedModel{
    public double getSpeedMph(double timeSec, int weightPounds, 
                                                     int horsePower){
      //method code goes here
    }
  }
}
```

我们将注释作为伪代码，并使用`...`符号代替实际代码，以便简洁。

如您所见，工厂类可能隐藏许多不同的私有类，每个类都包含特定驾驶条件的专门模型。每个模型产生不同的结果。

`FactoryVehicle`类的实现可能如下所示：

```java
public class FactoryVehicle {
  public static Car buildCar(int passengersCount, 
                               int weightPounds, int horsePower){
    return new CarImpl(passengersCount, weightPounds,horsePower);
  }
  public static Truck buildTruck(int payloadPounds, 
                               int weightPounds, int horsePower){
    return new TruckImpl(payloadPounds, weightPounds,horsePower);
  }
}
```

`CarImpl`私有嵌套类在`FactoryVehicle`类内部可能如下所示：

```java
  private static class CarImpl extends VehicleImpl implements Car {
    private int passengersCount;
    private CarImpl(int passengersCount, int weightPounds,
                                                 int horsePower){
      super(weightPounds + passengersCount * 250, horsePower);
      this.passengersCount = passengersCount;
    }
    public int getPassengersCount() { 
      return this.passengersCount;
    }
  }
```

同样，`TruckImpl`类可以是`FactoryImpl`类的私有嵌套类：

```java
  private static class TruckImpl extends VehicleImpl implements Truck {
    private int payloadPounds;
    private TruckImpl(int payloadPounds, int weightPounds, 
                                                     int horsePower){
      super(weightPounds+payloadPounds, horsePower);
      this.payloadPounds = payloadPounds;
    }
    public int getPayloadPounds(){ return payloadPounds; }
  }
```

我们也可以将`VehicleImpl`类作为`FactoryVehicle`类的私有内部类放置，这样`CarImpl`和`TruckImpl`类就可以访问它，但是`FactoryVehicle`之外的任何其他类都不能访问它：

```java
  private static abstract class VehicleImpl implements Vehicle {
    private SpeedModel speedModel;
    private int weightPounds, horsePower;
    private VehicleImpl(int weightPounds, int horsePower){
      this.weightPounds = weightPounds;
      this.horsePower = horsePower;
    }
    public void setSpeedModel(SpeedModel speedModel){ 
      this.speedModel = speedModel; 
    }
    public double getSpeedMph(double timeSec){
      return this.speedModel.getSpeedMph(timeSec, weightPounds, 
                                                    horsePower);
    }
  }
```

如您所见，接口描述了如何调用对象行为，而工厂可以为不同的请求生成不同的实现，而不改变客户端应用程序的代码。

# 还有更多...

让我们尝试建模一个乘员舱——一个具有多个乘客座位的卡车，它结合了汽车和卡车的特性。Java 不允许多重继承。这是另一个接口发挥作用的案例。

`CrewCab`类可能如下所示：

```java
public class CrewCab extends VehicleImpl implements Car, Truck {
  private int payloadPounds;
  private int passengersCount;
  private CrewCabImpl(int passengersCount, int payloadPounds,
                             int weightPounds, int horsePower) {
     super(weightPounds + payloadPounds + passengersCount * 250, 
                                                     horsePower);
     this.payloadPounds = payloadPounds;
     this. passengersCount = passengersCount;
  }
  public int getPayloadPounds(){ return payloadPounds; }
  public int getPassengersCount() { 
     return this.passengersCount;
  }
}
```

这个类同时实现了`Car`和`Truck`接口，并将车辆、货物和乘客及其行李的总重量传递给基类构造函数。

我们还可以向`FactoryVehicle`添加以下方法：

```java
public static Vehicle buildCrewCab(int passengersCount, 
                      int payload, int weightPounds, int horsePower){
  return new CrewCabImpl(passengersCount, payload, 
                                           weightPounds, horsePower);
}
```

`CrewCab`对象的双重性质可以在以下测试中得到证明：

```java
public static void main(String... arg) {
  double timeSec = 10.0;
  int horsePower = 246;
  int vehicleWeight = 4000;
  Properties drivingConditions = new Properties();
  drivingConditions.put("roadCondition", "Wet");
  drivingConditions.put("tireCondition", "New");
  SpeedModel speedModel = 
      FactorySpeedModel.generateSpeedModel(drivingConditions);
  Vehicle vehicle = FactoryVehicle.
             buildCrewCab(4, 3300, vehicleWeight, horsePower);
  vehicle.setSpeedModel(speedModel);
  System.out.println("Payload = " +
            ((Truck)vehicle).getPayloadPounds()) + " pounds");
  System.out.println("Passengers count = " + 
                         ((Car)vehicle).getPassengersCount());
  System.out.println("Crew cab speed (" + timeSec + " sec) = "  
                     + vehicle.getSpeedMph(timeSec) + " mph");
}
```

正如您所看到的，我们可以将`CrewCub`类的对象转换为它实现的每个接口。如果我们运行这个程序，结果将如下所示：

![](img/e5b5257c-8012-4426-9d71-ac9820ccbbf1.png)

# 创建具有默认和静态方法的接口

在这个示例中，您将了解到 Java 8 中首次引入的两个新功能——接口中的默认和静态方法。

# 准备就绪

接口中的默认方法允许我们添加一个新的方法签名，而不需要改变在添加新方法签名之前已经实现了该接口的类。该方法被称为*default*，因为它在该方法未被类实现时提供功能。然而，如果类实现了它，接口的默认实现将被忽略并被类实现覆盖。

接口中的静态方法可以提供与类中的静态方法相同的功能。类似于类静态方法可以在不实例化类的情况下调用一样，接口静态方法也可以使用点运算符应用于接口来调用，`SomeInterface.someStaticMethod()`。

接口的静态方法不能被实现该接口的类覆盖，也不能隐藏任何类的静态方法，包括实现该接口的类。

例如，让我们为我们已经在示例中使用的系统添加一些功能。到目前为止，我们已经创建了一个了不起的软件，可以计算车辆的速度。如果系统变得受欢迎（应该是这样），我们希望它对更喜欢使用公制单位的读者更友好，而不是我们在速度计算中使用的英里和磅。在我们的速度计算软件变得受欢迎之后，为了满足这样的需求，我们决定向`Car`和`Truck`接口添加更多方法，但我们不想破坏现有的实现。

默认接口方法正是为这种情况而引入的。使用它，我们可以发布`Car`和`Truck`接口的新版本，而无需协调与现有实现的相应修改，即`CarImpl`、`TruckImpl`和`FactoryVehicle`类。

# 如何做...

例如，我们将更改`Truck`接口。`Car`接口可以以类似的方式修改：

1.  通过添加一个新的默认方法来增强`Truck`接口，该方法返回卡车的载重量（以千克为单位）。您可以在不强制更改实现`Truck`接口的`TruckImpl`类的情况下完成这一点——通过向`Truck`接口添加一个新的默认方法：

```java
      public interface Truck extends Vehicle {
          int getPayloadPounds();
          default int getPayloadKg(){
            return (int) Math.round(0.454 * getPayloadPounds());
          }
      }
```

注意新的`getPayloadKg()`方法如何使用现有的`getPayloadPounds()`方法，就好像后者也是在接口内实现的一样，尽管实际上它是在实现`Truck`接口的类中实现的。魔术发生在运行时，当这个方法动态绑定到实现该接口的类的实例时。

我们无法将`getPayloadKg()`方法设为静态，因为它无法访问非静态的`getPayloadPounds()`方法，我们必须使用`default`关键字，因为只有接口的默认或静态方法才能有方法体。

1.  编写使用新方法的客户端代码：

```java
      public static void main(String... arg) {
         Truck truck = FactoryVehicle.buildTruck(3300, 4000, 246);
         System.out.println("Payload in pounds: " + 
                                        truck.getPayloadPounds());
         System.out.println("Payload in kg: " + 
                                            truck.getPayloadKg());
      }
```

1.  运行上述程序并检查输出：

![](img/b8a5a262-530f-4fda-8c70-f6a7031828ef.png)

1.  请注意，即使不改变实现它的类，新方法也可以工作。

1.  当您决定改进`TruckImpl`类的实现时，您可以通过添加相应的方法来实现，例如：

```java
       class TruckImpl extends VehicleImpl implements Truck {
          private int payloadPounds;
          private TruckImpl(int payloadPounds, int weightPounds,
                                                int horsePower) {
            super(weightPounds + payloadPounds, horsePower);
            this.payloadPounds = payloadPounds;
          }
          public int getPayloadPounds(){ return payloadPounds; }
          public int getPayloadKg(){ return -2; }
       }
```

我们已经实现了`getPyloadKg()`方法，使其为`return -2`，以便明确使用了哪种实现。

1.  运行相同的演示程序。结果将如下所示：

![](img/ec934ae1-b1ca-46e6-bd7c-c4ddd443e171.png)

正如您所看到的，这次在`TruckImpl`类中使用了方法实现。它已覆盖了`Truck`接口中的默认实现。

1.  增强`Truck`接口的功能，使其能够以千克为单位输入有效载荷，而不改变`FactoryVehicle`和`Truck`接口的实现。此外，我们不希望添加一个 setter 方法。在所有这些限制下，我们唯一的选择是在`Truck`接口中添加`convertKgToPounds(int kgs)`方法，并且它必须是静态的，因为我们将在实现`Truck`接口的对象构造之前使用它：

```java
      public interface Truck extends Vehicle {
          int getPayloadPounds();
          default int getPayloadKg(){
            return (int) Math.round(0.454 * getPayloadPounds());
          }
          static int convertKgToPounds(int kgs){
            return (int) Math.round(2.205 * kgs);
          }
       }
```

# 工作原理...

现在，喜欢使用公制单位的人可以利用新的方法：

```java
public static void main(String... arg) {
  int horsePower = 246;
  int payload = Truck.convertKgToPounds(1500);
  int vehicleWeight = Truck.convertKgToPounds(1800);
  Truck truck = FactoryVehicle.
           buildTruck(payload, vehicleWeight, horsePower);
  System.out.println("Payload in pounds: " + 
                                truck.getPayloadPounds());
  int kg = truck.getPayloadKg();
  System.out.println("Payload converted to kg: " + kg);
  System.out.println("Payload converted back to pounds: " + 
                              Truck.convertKgToPounds(kg));
}
```

结果将如下所示：

![](img/9b6dd7f4-683a-4c17-9fe9-36f269429277.png)

1,502 的值接近原始的 1,500，而 3,308 接近 3,312。差异是由转换过程中的近似误差引起的。

# 创建具有私有方法的接口

在本教程中，您将了解 Java 9 中引入的新功能——私有接口方法，它有两种类型：静态和非静态。

# 准备就绪

私有接口方法必须有实现（具有代码的主体）。同一接口的其他方法未使用的私有接口方法是没有意义的。私有方法的目的是包含在同一接口中具有主体的两个或多个方法之间的常见功能，或者将代码部分隔离在单独的方法中，以获得更好的结构和可读性。私有接口方法不能被覆盖，既不能被任何其他接口的方法覆盖，也不能被实现接口的类中的方法覆盖。

非静态私有接口方法只能被同一接口的非静态方法访问。静态私有接口方法可以被同一接口的非静态和静态方法访问。

# 如何做...

1.  添加`getWeightKg(int pounds)`方法的实现：

```java
     public interface Truck extends Vehicle {
         int getPayloadPounds();
         default int getPayloadKg(){
            return (int) Math.round(0.454 * getPayloadPounds());
         }
         static int convertKgToPounds(int kilograms){
            return (int) Math.round(2.205 * kilograms);
         }
         default int getWeightKg(int pounds){
            return (int) Math.round(0.454 * pounds);
         }
     }
```

1.  通过使用私有接口方法删除冗余代码：

```java
    public interface Truck extends Vehicle {
        int getPayloadPounds();
        default int getPayloadKg(int pounds){
            return convertPoundsToKg(pounds);
        }
        static int convertKgToPounds(int kilograms){
            return (int) Math.round(2.205 * kilograms);
        }
        default int getWeightKg(int pounds){
            return convertPoundsToKg(pounds);
        }
        private int convertPoundsToKg(int pounds){
            return (int) Math.round(0.454 * pounds);
        }
    }
```

# 工作原理...

以下代码演示了新的添加内容：

```java
public static void main(String... arg) {
  int horsePower = 246;
  int payload = Truck.convertKgToPounds(1500);
  int vehicleWeight = Truck.convertKgToPounds(1800);
  Truck truck = 
      FactoryVehicle.buildTruck(payload, vehicleWeight, horsePower);
  System.out.println("Weight in pounds: " + vehicleWeight);
  int kg = truck.getWeightKg(vehicleWeight);
  System.out.println("Weight converted to kg: " + kg);
  System.out.println("Weight converted back to pounds: " + 
                                       Truck.convertKgToPounds(kg));
}
```

测试结果不会改变：

![](img/6e275e58-b2e0-4b84-bf31-fd7cba0ce165.png)

# 还有更多...

由于`getWeightKg(int pounds)`方法接受输入参数，方法名称可能会误导，因为它没有捕捉输入参数的重量单位。我们可以尝试将其命名为`getWeightKgFromPounds(int pounds)`，但这并不会使方法功能更清晰。在意识到这一点后，我们决定将`convertPoundsToKg(int pounds)`方法设为公共方法，并完全删除`getWeightKg(int pounds)`方法。由于`convertPoundsToKg(int pounds)`方法不需要访问对象字段，它也可以是静态的：

```java
public interface Truck extends Vehicle {
  int getPayloadPounds();
  default int getPayloadKg(int pounds){
    return convertPoundsToKg(pounds);
  }
  static int convertKgToPounds(int kilograms){
    return (int) Math.round(2.205 * kilograms);
  }
  static int convertPoundsToKg(int pounds){
    return (int) Math.round(0.454 * pounds);
  }
}
```

仍然可以将英镑转换为千克，而且由于两种转换方法都是静态的，我们不需要创建实现`Truck`接口的类的实例来进行转换：

```java
public static void main(String... arg) {
  int payload = Truck.convertKgToPounds(1500);
  int vehicleWeight = Truck.convertKgToPounds(1800);
  System.out.println("Weight in pounds: " + vehicleWeight);
  int kg = Truck.convertPoundsToKg(vehicleWeight);
  System.out.println("Weight converted to kg: " + kg);
  System.out.println("Weight converted back to pounds: " + 
                                     Truck.convertKgToPounds(kg));
}
```

结果不会改变：

![](img/7e402e41-cd4e-42b6-9ed2-5549b764a5ce.png)

# 使用 Optional 更好地处理空值

在本教程中，您将学习如何使用`java.util.Optional`类来表示可选值，而不是使用`null`引用。它是在 Java 8 中引入的，并在 Java 9 中进一步增强，增加了三种方法：`or()`、`ifPresentOrElse()`和`stream()`。我们将演示它们全部。

# 准备就绪

`Optional`类是一个围绕值的包装器，可以是`null`或任何类型的值。它旨在帮助避免可怕的`NullPointerException`。但是，到目前为止，引入`Optional`只在流和函数式编程领域有所帮助。

创建`Optional`类的愿景是在`Optional`对象上调用`isPresent()`方法，然后仅在`isPresent()`方法返回`true`时应用`get()`方法（获取包含的值）。不幸的是，当无法保证`Optional`对象本身的引用不是`null`时，需要检查它以避免`NullPointerException`。如果是这样，那么使用`Optional`的价值就会减少，因为即使写入更少的代码，我们也可以检查值本身是否为`null`并避免包装在`Optional`中？让我们编写代码来说明我们所说的。

假设我们想编写一个方法来检查彩票结果，如果你和朋友一起买的彩票中奖了，计算你的 50%份额。传统的做法是：

```java
void checkResultInt(int lotteryPrize){
    if(lotteryPrize <= 0){
        System.out.println("We've lost again...");
    } else {
        System.out.println("We've won! Your half is " + 
                     Math.round(((double)lotteryPrize)/2) + "!");
    }
}
```

但是，为了演示如何使用`Optional`，我们将假设结果是`Integer`类型。然后，我们还需要检查`null`，如果我们不能确定传入的值不可能是`null`：

```java
void checkResultInt(Integer lotteryPrize){
    if(lotteryPrize == null || lotteryPrize <= 0){
        System.out.println("We've lost again...");
    } else {
        System.out.println("We've won! Your half is " + 
                    Math.round(((double)lotteryPrize)/2) + "!");
    }
}
```

使用`Optional`类并不能帮助避免对`null`的检查。它甚至需要添加额外的检查`isPresent()`，以避免在获取值时出现`NullPointerException`：

```java
void checkResultOpt(Optional<Integer> lotteryPrize){
    if(lotteryPrize == null || !lotteryPrize.isPresent() 
                                        || lotteryPrize.get() <= 0){
        System.out.println("We lost again...");
    } else {
        System.out.println("We've won! Your half is " + 
                   Math.round(((double)lotteryPrize.get())/2) + "!");
    }
}
```

显然，先前使用`Optional`并没有帮助改进代码或使编码更容易。在 Lambda 表达式和流管道中使用`Optional`具有更大的潜力，因为`Optional`对象提供了可以通过点运算符调用的方法，并且可以插入到流畅式处理代码中。

# 如何做...

1.  使用已经演示过的任何方法创建一个`Optional`对象，如下所示：

```java
Optional<Integer> prize1 = Optional.empty();
System.out.println(prize1.isPresent()); //prints: false
System.out.println(prize1);   //prints: Optional.empty

Optional<Integer> prize2 = Optional.of(1000000);
System.out.println(prize2.isPresent()); //prints: true
System.out.println(prize2);  //prints: Optional[1000000]

//Optional<Integer> prize = Optional.of(null); 
                                  //NullPointerException

Optional<Integer> prize3 = Optional.ofNullable(null);
System.out.println(prize3.isPresent());  //prints: false
System.out.println(prize3);     //prints: Optional.empty

```

请注意，可以使用`ofNullable()`方法将`null`值包装在`Optional`对象中。

1.  可以使用`equals()`方法比较两个`Optional`对象，该方法通过值进行比较：

```java
Optional<Integer> prize1 = Optional.empty();
System.out.println(prize1.equals(prize1)); //prints: true

Optional<Integer> prize2 = Optional.of(1000000);
System.out.println(prize1.equals(prize2)); //prints: false

Optional<Integer> prize3 = Optional.ofNullable(null);
System.out.println(prize1.equals(prize3)); //prints: true

Optional<Integer> prize4 = Optional.of(1000000);
System.out.println(prize2.equals(prize4)); //prints: true
System.out.println(prize2 == prize4); //prints: false

Optional<Integer> prize5 = Optional.of(10);
System.out.println(prize2.equals(prize5)); //prints: false

Optional<String> congrats1 = Optional.empty();
System.out.println(prize1.equals(congrats1));//prints: true

Optional<String> congrats2 = Optional.of("Happy for you!");
System.out.println(prize1.equals(congrats2));//prints: false
```

请注意，空的`Optional`对象等于包装`null`值的对象（上述代码中的`prize1`和`prize3`对象）。上述代码中的`prize2`和`prize4`对象相等，因为它们包装相同的值，尽管它们是不同的对象，引用不匹配（`prize2 != prize4`）。还要注意，包装不同类型的空对象是相等的（`prize1.equals(congrats1)`），这意味着`Optional`类的`equals()`方法不比较值类型。

3. 使用`Optional`类的`or(Suppier<Optional<T>> supplier)`方法可靠地从`Optional`对象中返回非空值。如果对象为空并包含`null`，则它将返回由提供的`Supplier`函数生成的`Optional`对象中的另一个值。

例如，如果`Optional<Integer> lotteryPrize`对象可能包含`null`值，则以下结构将在遇到`null`值时每次返回零：

```java
       int prize = lotteryPrize.or(() -> Optional.of(0)).get();

```

1.  使用`ifPresent(Consumer<T> consumer)`方法来忽略`null`值，并使用提供的`Consumer<T>`函数处理非空值。例如，这是`processIfPresent(Optional<Integer>)`方法，它处理`Optional<Integer> lotteryPrize`对象：

```java
void processIfPresent(Optional<Integer> lotteryPrize){
   lotteryPrize.ifPresent(prize -> {
      if(prize <= 0){
          System.out.println("We've lost again...");
      } else {
          System.out.println("We've won! Your half is " + 
                    Math.round(((double)prize)/2) + "!");
     }
});
```

我们可以通过创建`checkResultAndShare(int prize)`方法简化上述代码：

```java
void checkResultAndShare(int prize){
    if(prize <= 0){
        System.out.println("We've lost again...");
    } else {
        System.out.println("We've won! Your half is " + 
                   Math.round(((double)prize)/2) + "!");
    }
}
```

现在，`processIfPresent()`方法看起来简单得多：

```java
void processIfPresent(Optional<Integer> lotteryPrize){
    lotteryPrize.ifPresent(prize -> checkResultAndShare(prize));
}
```

1.  如果您不想忽略`null`值并且也要处理它，可以使用`ifPresentOrElse(Consumer<T> consumer, Runnable processEmpty)`方法将`Consumer<T>`函数应用于非空值，并使用`Runnable`函数接口来处理`null`值：

```java
void processIfPresentOrElse(Optional<Integer> lotteryPrize){
   Consumer<Integer> weWon = 
                       prize -> checkResultAndShare(prize);
   Runnable weLost = 
           () -> System.out.println("We've lost again...");
   lotteryPrize.ifPresentOrElse(weWon, weLost);
}
```

正如您所见，我们已经重用了刚刚创建的`checkResultAndShare(int prize)`方法。

1.  使用`orElseGet(Supplier<T> supplier)`方法允许我们用由提供的`Supplier<T>`函数产生的值来替换`Optional`对象中的空值或`null`值：

```java
void processOrGet(Optional<Integer> lotteryPrize){
   int prize = lotteryPrize.orElseGet(() -> 42);
   lotteryPrize.ifPresentOrElse(p -> checkResultAndShare(p),
      () -> System.out.println("Better " + prize 
                                     + " than nothing..."));
 }
```

1.  如果需要在`Optional`对象为空或包含`null`值的情况下抛出异常，请使用`orElseThrow()`方法：

```java
void processOrThrow(Optional<Integer> lotteryPrize){
   int prize = lotteryPrize.orElseThrow();
   checkResultAndShare(prize);
}
```

`orElseThrow()`方法的重载版本允许我们指定异常和当`Optional`对象中包含的值为`null`时要抛出的消息：

```java
void processOrThrow(Optional<Integer> lotteryPrize){
    int prize = lotteryPrize.orElseThrow(() -> 
           new RuntimeException("We've lost again..."));
    checkResultAndShare(prize);
}
```

1.  使用`filter()`、`map()`和`flatMap()`方法来处理流中的`Optional`对象：

```java
void useFilter(List<Optional<Integer>> list){
   list.stream().filter(opt -> opt.isPresent())
         .forEach(opt -> checkResultAndShare(opt.get()));
}
void useMap(List<Optional<Integer>> list){
   list.stream().map(opt -> opt.or(() -> Optional.of(0)))
         .forEach(opt -> checkResultAndShare(opt.get()));
}
void useFlatMap(List<Optional<Integer>> list){
   list.stream().flatMap(opt -> 
           List.of(opt.or(()->Optional.of(0))).stream())
        .forEach(opt -> checkResultAndShare(opt.get()));
}
```

在前面的代码中，`useFilter()`方法只处理那些非空数值的流元素。`useMap()`方法处理所有流元素，但是用没有值的`Optional`对象或者用包装了`null`值的`Optional`对象替换它们。最后一个方法使用了`flatMap()`，它需要从提供的函数返回一个流。在这方面，我们的示例是相当无用的，因为我们传递给`flatMap()`参数的函数产生了一个对象的流，所以在这里使用`map()`（就像前面的`useMap()`方法中一样）是一个更好的解决方案。我们只是为了演示`flatMap()`方法如何插入到流管道中才这样做的。

# 它是如何工作的...

以下代码演示了所描述的`Optional`类的功能。`useFlatMap()`方法接受一个`Optional`对象列表，创建一个流，并处理每个发出的元素：

```java
void useFlatMap(List<Optional<Integer>> list){
    Function<Optional<Integer>, 
      Stream<Optional<Integer>>> tryUntilWin = opt -> {
        List<Optional<Integer>> opts = new ArrayList<>();
        if(opt.isPresent()){
            opts.add(opt);
        } else {
            int prize = 0;
            while(prize == 0){
                double d = Math.random() - 0.8;
                prize = d > 0 ? (int)(1000000 * d) : 0;
                opts.add(Optional.of(prize));
            }
        }
        return opts.stream();
    };
    list.stream().flatMap(tryUntilWin)
        .forEach(opt -> checkResultAndShare(opt.get()));
}
```

原始列表的每个元素首先作为输入进入`flatMap()`方法，然后作为`tryUntilWin`函数的输入。这个函数首先检查`Optional`对象的值是否存在。如果是，`Optional`对象将作为流的单个元素发出，并由`checkResultAndShare()`方法处理。但是如果`tryUntilWin`函数确定`Optional`对象中没有值或者值为`null`，它会在`-0.8`和`0.2`之间生成一个随机双精度数。如果值为负数，就会向结果列表中添加一个值为零的`Optional`对象，并生成一个新的随机数。但如果生成的数是正数，它将用于奖金值的计算，并添加到包装在`Optional`对象中的结果列表中。`Optional`对象的结果列表然后作为流返回，并且流的每个元素都由`checkResultAndShare()`方法处理。

现在，让我们对以下列表运行前面的方法：

```java
List<Optional<Integer>> list = List.of(Optional.empty(), 
                                       Optional.ofNullable(null), 
                                       Optional.of(100000));
useFlatMap(list);

```

结果将如下所示：

![](img/94c3889d-7c68-497b-a360-7604d8c4f8ca.png)

如您所见，当第一个列表元素`Optional.empty()`被处理时，`tryUntilWin`函数在第三次尝试中成功获得了一个正的奖金值。第二个`Optional.ofNullable(null)`对象导致了两次尝试，直到`tryUntilWin`函数成功。最后一个对象成功通过，并奖励您和您的朋友各 50000。

# 还有更多...

`Optional`类的对象不可序列化，因此不能用作对象的字段。这是`Optional`类的设计者打算在无状态过程中使用的另一个指示。

它使流处理管道更加简洁和表达，专注于实际值而不是检查流中是否有空元素。

# 使用实用类 Objects

在这个配方中，您将学习`java.util.Objects`实用类如何允许更好地处理与对象比较、计算哈希值和检查`null`相关的功能。这是早就该有的功能，因为程序员们一遍又一遍地编写相同的代码来检查对象是否为`null`。

# 准备工作

`Objects`类只有 17 种方法，全部都是静态的。为了更好地概述，我们将它们组织成了七个组：

+   `compare()`: 使用提供的`Comparator`比较两个对象的方法

+   `toString()`: 将`Object`转换为`String`值的两种方法

+   `checkIndex()`: 三种允许我们检查集合或数组的索引和长度是否兼容的方法

+   `requireNonNull()`: 如果提供的对象为`null`，则五种方法抛出异常

+   `hash()`, `hashCode():` 计算单个对象或对象数组的哈希值的两种方法

+   `isNull()`, `nonNull()`: 包装`obj == null`或`obj != null`表达式的两种方法

+   `equals()`, `deepEquals()`: 比较两个可以为 null 或数组的对象的两种方法

我们将按照前述顺序编写使用这些方法的代码。

# 如何做...

1.  `int compare(T a, T b, Comparator<T> c)`方法使用提供的比较器来比较两个对象：

+   当对象相等时返回 0

+   当第一个对象小于第二个对象时返回负数

+   否则返回正数

`int compare(T a, T b, Comparator<T> c)`方法的非零返回值取决于实现。对于`String`，根据它们的排序位置定义较小和较大（较小的放在有序列表的前面），返回值是第一个和第二个参数在列表中的位置之间的差异，根据提供的比较器排序：

```java
int res =
      Objects.compare("a", "c", Comparator.naturalOrder());
System.out.println(res);       //prints: -2
res = Objects.compare("a", "a", Comparator.naturalOrder());
System.out.println(res);       //prints: 0
res = Objects.compare("c", "a", Comparator.naturalOrder());
System.out.println(res);       //prints: 2
res = Objects.compare("c", "a", Comparator.reverseOrder());
System.out.println(res);       //prints: -2
```

另一方面，`Integer`值在值不相等时返回`-1`或`1`：

```java
res = Objects.compare(3, 5, Comparator.naturalOrder());
System.out.println(res);       //prints: -1
res = Objects.compare(3, 3, Comparator.naturalOrder());
System.out.println(res);       //prints: 0
res = Objects.compare(5, 3, Comparator.naturalOrder());
System.out.println(res);       //prints: 1
res = Objects.compare(5, 3, Comparator.reverseOrder());
System.out.println(res);       //prints: -1
res = Objects.compare("5", "3", Comparator.reverseOrder());
System.out.println(res);       //prints: -2
```

请注意，在前面代码块的最后一行中，当我们将数字作为`String`文字进行比较时，结果会发生变化。

当两个对象都为`null`时，`compare()`方法将认为它们相等：

```java
res = Objects.compare(null,null,Comparator.naturalOrder());
System.out.println(res);       //prints: 0

```

但是当其中一个对象为 null 时，会抛出`NullPointerException`：

```java
//Objects.compare(null, "c", Comparator.naturalOrder());   
//Objects.compare("a", null, Comparator.naturalOrder());   
```

如果需要将对象与 null 进行比较，最好使用`org.apache.commons.lang3.ObjectUtils.compare(T o1, T o2)`。

1.  当`null`时，`toString(Object obj)`方法很有用：

+   `String toString(Object obj)`: 当第一个参数不为`null`时，返回调用`toString()`的结果，当第一个参数值为`null`时，返回`null`

+   `String toString(Object obj, String nullDefault)`: 当第一个参数不为`null`时，返回调用`toString()`的结果，当第一个参数值为`null`时，返回第二个参数值`nullDefault`

`toString(Object obj)`方法的使用很简单：

```java
System.out.println(Objects.toString("a")); //prints: a
System.out.println(Objects.toString(null)); //prints: null
System.out.println(Objects.toString("a", "b")); //prints: a
System.out.println(Objects.toString(null, "b"));//prints: b
```

1.  `checkIndex()`重载方法检查集合或数组的索引和长度是否兼容：

+   `int checkIndex(int index, int length)`: 如果提供的`index`大于`length - 1`，则抛出`IndexOutOfBoundsException`，例如：

```java
List<Integer> list = List.of(1, 2);
try {
   Objects.checkIndex(3, list.size());
} catch (IndexOutOfBoundsException ex){
   System.out.println(ex.getMessage()); 
       //prints: Index 3 out-of-bounds for length 2
}
```

1.  +   `int checkFromIndexSize(int fromIndex, int size, int length)`: 如果提供的`index + size`大于`length - 1`，则抛出`IndexOutOfBoundsException`，例如：

```java
List<Integer> list = List.of(1, 2);
try {
   Objects.checkFromIndexSize(1, 3, list.size());
} catch (IndexOutOfBoundsException ex){
   System.out.println(ex.getMessage());
//prints:Range [1, 1 + 3) out-of-bounds for length 2
}
```

1.  +   `int checkFromToIndex(int fromIndex, int toIndex, int length)`: 如果提供的`fromIndex`大于`toIndex`，或者`toIndex`大于`length - 1`，则抛出`IndexOutOfBoundsException`，例如：

```java
List<Integer> list = List.of(1, 2);
try {
   Objects.checkFromToIndex(1, 3, list.size());
} catch (IndexOutOfBoundsException ex){
   System.out.println(ex.getMessage()); 
   //prints:Range [1, 3) out-of-bounds for length 2
}
```

1.  `requireNonNull()`组的五种方法检查第一个参数`obj`的值。如果值为`null`，它们要么抛出`NullPointerException`，要么返回提供的默认值：

+   `T requireNonNull(T obj)`: 如果参数为`null`，则抛出没有消息的`NullPointerException`，例如：

```java
String obj = null;
try {
  Objects.requireNonNull(obj);
} catch (NullPointerException ex){
  System.out.println(ex.getMessage());//prints: null
}
```

1.  +   `T requireNonNull(T obj, String message)`: 如果第一个参数为`null`，则抛出带有提供的消息的`NullPointerException`，例如：

```java
String obj = null;
try {
  Objects.requireNonNull(obj,  
                          "Parameter 'obj' is null");
} catch (NullPointerException ex){
  System.out.println(ex.getMessage()); 
                 //prints: Parameter 'obj' is null
}
```

1.  +   `T requireNonNull(T obj, Supplier<String> messageSupplier)`: 如果第一个参数为`null`，则返回由提供的函数生成的消息，如果生成的消息或函数本身为`null`，则抛出`NullPointerException`，例如：

```java
String obj = null;
Supplier<String> supplier = () -> "Message";
try {
  Objects.requireNonNull(obj, supplier);
} catch (NullPointerException ex){
  System.out.println(ex.getMessage()); 
                         //prints: Message
}
```

1.  +   `T requireNonNullElse(T obj, T defaultObj)`: 返回第一个参数（如果它不是 null），第二个参数（如果它不是 null），抛出`NullPointerException`（如果两个参数都是`null`），例如：

```java
String object = null;
System.out.println(Objects
          .requireNonNullElse(obj, "Default value")); 
                          //prints: Default value
```

1.  +   `T requireNonNullElseGet(T obj, Supplier<T> supplier)`: 返回第一个参数（如果它不是 null），由提供的供应商函数产生的对象（如果它不是 null 且`supplier.get()`不是 null），抛出`NullPointerException`（如果两个参数都是`null`或第一个参数和 supplier.get()都是`null`），例如：

```java
Integer obj = null;
Supplier<Integer> supplier = () -> 42;
try {
    System.out.println(Objects
              .requireNonNullElseGet(obj, supplier));
} catch (NullPointerException ex){
    System.out.println(ex.getMessage()); //prints: 42
} 
```

1.  `hash()`或`hashCode()`方法通常用于覆盖默认的`hashCode()`实现：

+   `int hashCode(Object value)`: 为单个对象计算哈希值，例如：

```java
System.out.println(Objects.hashCode(null)); 
                                       //prints: 0
System.out.println(Objects.hashCode("abc")); 
                                   //prints: 96354 
```

+   `int hash(Object... values)`: 为对象数组计算哈希值，例如：

```java
System.out.println(Objects.hash(null));  //prints: 0
System.out.println(Objects.hash("abc"));  
                                     //prints: 96385
String[] arr = {"abc"};
System.out.println(Objects.hash(arr));
                                     //prints: 96385
Object[] objs = {"a", 42, "c"};
System.out.println(Objects.hash(objs));  
                                    //prints: 124409
System.out.println(Objects.hash("a", 42, "c")); 
                                    //prints: 124409
```

请注意，`hashCode(Object value)`方法返回一个不同的哈希值(`96354`)，而`Objects.hash(Object... values)`方法返回(`96385`)，尽管它们为相同的单个对象计算哈希值。

1.  `isNull()`和`nonNull()`方法只是布尔表达式的包装器：

+   `boolean isNull(Object obj)`: 返回与`obj == null`相同的值，例如：

```java
String obj = null;
System.out.println(obj == null);     //prints: true
System.out.println(Objects.isNull(obj));
                                     //prints: true
obj = "";
System.out.println(obj == null);    //prints: false
System.out.println(Objects.isNull(obj));  
                                    //prints: false
```

1.  +   `boolean nonNull(Object obj)`: 返回与`obj != null`相同的值，例如：

```java
String obj = null;
System.out.println(obj != null);    //prints: false
System.out.println(Objects.nonNull(obj)); 
                                    //prints: false
obj = "";
System.out.println(obj != null);     //prints: true
System.out.println(Objects.nonNull(obj));  
                                     //prints: true
```

1.  `equals()`和`deepEquals()`方法允许我们通过它们的状态来比较两个对象：

+   `boolean equals(Object a, Object b)`: 使用`equals(Object)`方法比较两个对象，并处理它们中的一个或两个为`null`的情况，例如：

```java
String o1 = "o";
String o2 = "o";
System.out.println(Objects.equals(o1, o2));       
                                   //prints: true
System.out.println(Objects.equals(null, null));   
                                   //prints: true
Integer[] ints1 = {1,2,3};
Integer[] ints2 = {1,2,3};
System.out.println(Objects.equals(ints1, ints2)); 
                                  //prints: false
```

在上面的例子中，`Objects.equals(ints1, ints2)`返回`false`，因为数组不能覆盖`Object`类的`equals()`方法，而是通过引用而不是值进行比较。

1.  +   `boolean deepEquals(Object a, Object b)`: 比较两个数组的元素值，例如：

```java
String o1 = "o";
String o2 = "o";
System.out.println(Objects.deepEquals(o1, o2));    
                                   //prints: true
System.out.println(Objects.deepEquals(null, null));
                                   //prints: true
Integer[] ints1 = {1,2,3};
Integer[] ints2 = {1,2,3};
System.out.println(Objects.deepEquals(ints1,ints2));
                                      //prints: true
Integer[][] iints1 = {{1,2,3},{1,2,3}};
Integer[][] iints2 = {{1,2,3},{1,2,3}};
System.out.println(Objects.
         deepEquals(iints1, iints2)); //prints: true  
```

正如你所看到的，`deepEquals()`方法在数组的相应值相等时返回`true`。但是如果数组有不同的值或相同值的不同顺序，该方法将返回`false`：

```java
Integer[][] iints1 = {{1,2,3},{1,2,3}};
Integer[][] iints2 = {{1,2,3},{1,3,2}};
System.out.println(Objects.
      deepEquals(iints1, iints2)); //prints: false
```

# 它是如何工作的...

`Arrays.equals(Object a, Object b)`和`Arrays.deepEquals(Object a, Object b)`方法的行为与`Objects.equals(Object a, Object b)`和`Objects.deepEquals(Object a, Object b)`方法相同：

```java
Integer[] ints1 = {1,2,3};
Integer[] ints2 = {1,2,3};
System.out.println(Arrays.equals(ints1, ints2));         
                                            //prints: true
System.out.println(Arrays.deepEquals(ints1, ints2));     
                                            //prints: true
System.out.println(Arrays.equals(iints1, iints2));       
                                            //prints: false
System.out.println(Arrays.deepEquals(iints1, iints2));   
                                            //prints: true
```

实际上，`Arrays.equals(Object a, Object b)`和`Arrays.deepEquals(Object a, Object b)`方法在`Objects.equals(Object a, Object b)`和`Objects.deepEquals(Object a, Object b)`方法的实现中被使用。

总之，如果你想要比较两个对象`a`和`b`的字段值，那么：

+   如果它们不是数组且`a`不是`null`，则使用`a.equals(Object b)`

+   如果它们不是数组且每个或两个对象都可以是`null`，则使用`Objects.equals(Object a, Object b)`

+   如果两者都可以是数组且每个或两者都可以是`null`，则使用`Objects.deepEquals(Object a, Object b)`

`Objects.deepEquals(Object a, Object b)`方法似乎是最安全的，但这并不意味着你必须总是使用它。大多数情况下，你会知道比较的对象是否可以是`null`或者可以是数组，因此你也可以安全地使用其他方法。
