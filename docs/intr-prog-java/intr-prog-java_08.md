# 第八章：面向对象设计（OOD）原则

在本章中，我们将回到对编程和特别是 Java 编程的高层视图。我们将展示设计在软件系统过程中的作用，从最早的可行性阶段开始，经过高层设计、详细设计，最终到编码和测试。我们将讨论良好设计的标准，并提供一份经过验证的 OOD 原则指南。讨论将通过代码示例加以说明，演示主要 OOD 原则的应用。

在本章中，我们将涵盖以下主题：

+   设计的目的是什么？

+   封装和编程到接口

+   利用多态性

+   尽可能解耦

+   优先使用聚合而不是继承

+   这么多 OOD 原则，时间却如此有限

+   单一职责原则

+   开闭原则

+   里斯科夫替换原则

+   接口隔离原则

+   依赖反转原则

+   练习 - 设计模式

# 设计的目的是什么？

任何项目都需要规划和对将要构建的东西的愿景。当同一个团队的几个成员必须协调他们的活动时，这尤为重要。但即使你是一个人工作，你也必须制定某种计划，无论是设计文档还是只是编写代码而没有以其他形式记录你的想法。这就是设计的目的——清晰地设想未来的系统，以便能够开始构建它。

在这个过程中，设计会不断演变、改变并变得更加详细。项目生命周期的每个阶段都需要不同的东西。这就是我们现在要讨论的——随着项目从最初的想法到完整实施的进展，设计的目的如何演变。

这里描述的项目步骤看起来是顺序的，但实际上它们是有重叠的。更重要的是，软件开发的敏捷方法鼓励将每个功能移动到所有项目步骤中，而不是等到发现未来产品的所有功能。

在敏捷方法论中，交付物不是需求、设计或任何其他文档，而是部署到生产环境并产生价值的功能代码（也称为最小可行产品（MVP））。每次迭代都必须在一两周内完成。然后，基于真实客户体验的反馈循环允许不断调整最初的愿景，并驱动所有努力以在最短时间内实现最有价值的解决方案，并最小化资源浪费。

许多现代成功的产品，如果不是大多数，都是以这种方式推向市场的。它们的作者经常承认，只有少数原创的想法被实现了，如果有的话。生活是一个伟大的笑话，不是吗？它偏爱那些更快适应变化的人。

现在，让我们走过项目生命周期，看看系统设计是如何随着项目的进展而演变的。

# 项目的可行性

决定某个项目是否值得融资必须在非常早期就做出。否则，它可能根本就不会开始。这意味着决策者必须提供足够的信息，以提供一定程度的信心，即风险是合理的，值得承担。这些信息包括高层需求、高层设计，甚至原型设计或其他证明可用技术可以用于成功实施。基于这些数据和市场调研，项目倡导者估计工作量、费用、潜在收入和未来利润——一切目标的母亲。

甚至在项目获得绿灯之前，产品成功最关键的特性就已经被确定，并以可与未来客户沟通的形式呈现，并与他们讨论甚至测试。如果团队中包括过去做过类似事情的人，肯定有助于简化决策过程。

这个阶段的目的是以一种所有参与者和潜在客户都能理解的形式呈现未来的系统。

# 需求收集和原型制作

一旦项目获得批准和预算，需求收集就会全速进行，同时进行原型实现。事实上，原型通常被用作需求收集的工具。它有助于讨论具体的关键细节并避免误解。

在这个项目阶段，高级设计不断进展，同时发现有关输入信息来源、消耗它所需的过程（和产生必要结果的过程）、可以用来执行它的技术，以及客户可能如何与系统交互的更多细节。

随着对未来系统的更多数据，以及它可能如何工作和实现，可以确定可能妨碍进展或使整个项目不可能的障碍。因此，决策者继续密切关注结果并进行批判性评估。

在这个阶段，设计的目的是将所有输入数据整合成未来运行系统的连贯动态图像。在面向对象编程的四个支柱中，封装和接口处于高级设计的前沿。实现细节应在关键领域进行核查，并证明可以使用所选的技术。但它们保持隐藏在接口后面，后者专注于系统与客户的互动以及发现实现的新功能和非功能要求。

# 高级设计

高级设计最明显的特征是其专注于子系统和它们之间的接口的系统结构。如果产品必须与外部系统交互，这些交互的接口和协议也是高级设计的一部分。架构也被确认和验证为能够支持设计。

对于典型的中型软件系统，高级设计可以用包及其公共接口的列表来表达。如果系统具有图形用户界面，通常原型和线框图就足够了。

# 详细设计

一旦确定要实现的用例，详细设计就开始发挥作用。业务代表为新产品功能设置优先级。程序员确定并调整接口以支持第一个功能，并开始创建类来实现将在第一次迭代中交付的第一个用例。

最初，实现可能在某些地方使用硬编码（虚拟）数据。因此，用例可能具有有限的应用范围。尽管如此，这样的实现是有价值的，因为它允许执行所有必需的过程，因此生产中的客户可以测试该功能并了解预期的情况。程序员还为每个实现的方法创建单元测试，即使是虚拟的方法也是如此。与此同时，用例被捕获在执行跨类和子系统的场景的集成测试中。

在第一次迭代结束时，高优先级的用例已经实现并通过自动化测试进行了全面测试。第一次迭代通常非常忙碌。但程序员们有动力不再重复他们的错误，通常会充满热情并具有比平时更高的生产力。

详细设计的目的是为编码提供模板。一旦模板建立，所有未来的类将主要是从现有类中剪切和粘贴。这就是为什么第一个类通常由高级程序员实现或在他们的密切监督下实现。在这样做的同时，他们试图尽可能保持封装封闭，以获得最小和直观的接口，并在可能的情况下利用继承和多态性。

命名约定也是第一次迭代的重要组成部分。它必须反映领域术语，并且所有团队成员都能理解。因此，这个阶段的设计目的是为项目创建编码模式和词汇。

# 编码

正如你所看到的，编码从高层设计开始，甚至可能更早。随着详细设计产生了第一个结果，编码变得更加紧张。新成员可以加入团队，其中一些可能是初级成员。增加团队成员是最喜欢的管理活动，但必须以受控的方式进行，以便每个新成员都能得到指导，并且能够充分理解所有关于新产品功能的业务讨论。

这个阶段的设计活动侧重于实现细节及其测试。在详细设计期间创建的模式必须根据需要进行应用和调整。编码期间的设计目的是验证到目前为止所做的所有设计决策，并产生具体的解决方案，表达为代码行。重构是这个阶段的主要活动之一，也有几次迭代。

# 测试

在编码完成时，测试也已编写，并且运行了多次。它们通常在每次向源代码库提交新的更改块时执行。一些公司正在实践持续集成模型，一旦提交到源代码库，就会触发自动回归和集成测试，并随后部署到生产环境。

然而，仍然有许多开发团队专门有专门的测试专家，在代码部署到测试环境后，会手动测试并使用一些专门的工具。

这个阶段的设计工作侧重于测试覆盖率、测试自动化以及与其他系统的集成，无论是自动化的还是非自动化的。部署和在生产环境中进行有限测试（称为**冒烟测试**）也是这个阶段设计工作的一部分。

测试期间的设计目的是确保所有交付的用例都经过测试，包括负面和非功能性测试。监控和报告系统性能也是这个阶段的重要活动。

# 良好设计的路线图

正如我们在前一节中讨论的设计演变，我们已经暗示了确保设计质量的标准：

+   它必须足够灵活，以适应即将到来的变化（它们像税收一样不可避免，所以最好做好准备）

+   它必须清晰地传达项目结构和每个部分的专业化

+   它必须使用明确定义的领域术语

+   它必须允许独立测试部分并将其集成在一起

+   它必须以一种允许我们与未来客户讨论的形式呈现，并且理想情况下，由他们测试。

+   它必须充分利用四个面向对象的概念——封装、接口、继承和多态性

这些是任何项目和任何面向对象语言的一般标准。但在本书中，我们介绍了 Java 最佳实践，因此我们需要主要讨论 Java 中的详细设计、编码和测试，所有这些都与最后一个标准有关。这就是我们现在要做的。

# 封装和编码到接口

我们多次在不同的上下文中提到了封装和接口。这既不是偶然的，也不是有意的。这是不可避免的。封装和接口是出于尽可能隐藏实现的必要性而产生的。它解决了早期编程中的两个问题：

+   未受监管的数据共享访问

+   以下是输出的屏幕截图：

当部分之间的关系结构不够完善时更改代码时的困难

正如我们在第六章中所演示的，*接口、类和对象构造*，使对象的状态私有化也解决了涉及继承时实例字段和实例方法之间可访问性的差异。子类不能覆盖父类的非私有字段，只能隐藏它们。只有方法可以被覆盖。为了演示这种差异，让我们创建以下三个类：

```java
public class Grandad {
  public String name = "Grandad";
  public String getName() { return this.name; }
}

public class Parent extends Grandad {
  public String name = "Parent";
  public String getName() { return this.name; }
}

public class Child extends Parent {
  public String name = "Child";
  public String getName() { return this.name; }
}
```

车辆数量

![](img/8ff835e5-929c-4271-b026-757e044dd29e.png)

每个都有一个具有相同名称的公共字段和相同签名的方法。现在，在不往下看的情况下，尝试猜测以下代码的输出：

```java
Grandad grandad = new Child();
System.out.println(grandad.name);
System.out.println(grandad.getName());

```java

+   所有车辆开始移动后的秒数

+   车辆负载：汽车乘客数量和卡车的有效载荷

最后一个参数应该是可选的。它可以是以下之一：

+   基于目标城市的当前交通统计数据建模

+   设置特定值，以评估新交通法规的影响

以下是位于`com.packt.javapath.ch08demo.traffic`包中的建模系统 API 的详细设计：

```java
public interface Vehicle {
  double getSpeedMph(double timeSec);
  static List<Vehicle> getTraffic(int vehiclesCount){
    return TrafficFactory.get(vehiclesCount);
  }
}
public interface Car extends Vehicle {
  void setPassengersCount(int passengersCount);
}
public interface Truck extends Vehicle {
  void setPayloadPounds(int payloadPounds);
}
```

正如您所看到的，我们只向客户端公开接口并隐藏实现（关于这一点我们将在下一节详细讨论）。只要满足合同，它允许我们以我们认为最好的方式实现接口。如果以后更改了实现，客户端不需要更改他们的代码。这是封装和解耦接口与实现的一个例子。正如我们在上一章中讨论的那样，它还有助于代码的可维护性、可测试性和可重用性。更多关于后者的内容请参见*更喜欢聚合而不是继承*部分，尽管我们应该指出，继承也有助于代码重用，我们将在下一节中看到它的证明。

通过从`Vehicle`接口扩展`Car`和`Truck`接口，我们已经暗示了我们将使用多态性，这就是我们将在接下来的部分讨论的内容。

# 利用多态性

`Car`和`Truck`接口正在扩展（子类）`Vehicle`接口。这意味着实现`Car`接口的类（例如，我们给这样的类命名为`CarImpl`），在实例化时，创建了一个具有三种类型的对象——`Vehicle`、`Car`和`CarImpl`。这些类型类似于一个人拥有三个国家的护照。每种国籍都有特定的权利和限制，一个人可以选择在国际旅行的不同情况下如何呈现自己，同样，`CarImpl`类的对象可以*转换*为这些类型中的任何一个，只要在进行转换的代码中可以访问该类型。这就是我们所说的类型可访问性的含义：

+   我们已经将`Car`、`Truck`和`Vehicle`接口声明为 public，这意味着任何包中的任何代码都可以访问这些类型

+   我们不希望客户端代码能够访问这些接口的实现，因此我们创建了`com.packt.javapath.ch08demo.traffic.impl`包，并将所有实现放在那里，而不指定访问修饰符（因此使用默认访问，使它们只对同一包中的其他成员可见）

这里是交通接口的实现：

```java
class VehicleImpl implements Vehicle {
  public double getSpeedMph(double timeSec){
    return 42;
  }
}
class TruckImpl implements Truck {
  public void setPayloadPounds(int payloadPounds){
  }
}
class CarImpl implements Car {
  public void setPassengersCount(int passengersCount){
  }
}
```

我们在`com.packt.javapath.ch08demo.traffic.impl`包中创建了这些类，并使用了一些虚拟数据，只是为了使它们编译通过。但是`CarImpl`和`TruckImpl`类仍然会生成编译错误，因为`Vehicle`接口中列出了`getSpeedMph()`方法，而这两个类中没有实现。`Car`和`Truck`接口扩展了`Vehicle`接口，因此继承了它的抽象`getSpeedMph()`方法。

因此，现在我们需要在这两个类中实现`getSpeedMph()`方法，或者将它们都作为`VehicleImpl`类的子类，而这个方法已经被实现了。我们决定汽车和卡车的速度可能会以相同的方式计算，所以扩展`VehicleImpl`类是正确的方法。如果以后我们发现`CarImpl`或`TruckImpl`类需要不同的实现，我们可以覆盖父类中的实现。以下是相同两个类的新版本：

```java
abstract class VehicleImpl implements Vehicle {
  public double getSpeedMph(double timeSec){
    return 42;
  }
}
class TruckImpl extends VehicleImpl implements Truck {
  public void setPayloadPounds(int payloadPounds){
  }
}
class CarImpl extends VehicleImpl implements Car {
  public void setPassengersCount(int passengersCount){
  }
}
```

请注意，我们还将`VehicleImpl`类设为抽象类，这使得不可能创建`VehicleImpl`类的对象。只能创建它的子类的对象。我们这样做是因为我们将其用作包含一些通用功能的基类，但我们永远不会需要通用的`Vehicle`对象，只需要特定的对象——`Car`或`Truck`。

我们遵循了尽可能封装一切的建议。受限制的访问权限可以在以后更改为更可访问的权限。这比在已经编写了依赖于现有较不受限制访问级别的客户端代码之后再限制访问权限要容易得多。

所以，回到`CarImpl`和`TruckImpl`交通接口的实现。它们无法从包外访问，但这并不是问题，因为我们定义的 API 不需要它。如果`TrafficFactory`类可以访问它们，那就足够了。这就是为什么我们在`com.packt.javapath.ch08demo.traffic.impl`包中创建`TrafficFactor`类，它可以作为同一包的成员访问这两个实现：

```java
package com.packt.javapath.ch08demo.traffic.impl;

import com.packt.javapath.ch08demo.traffic.Vehicle;
import java.util.ArrayList;
import java.util.List;

public class TrafficFactory {
  public static List<Vehicle> get(int vehiclesCount) {
    List<Vehicle> list = new ArrayList();
    return list;
  }
}
```

它并没有做太多事情，但在设计阶段足够好，以确保所有类都就位并具有适当的访问权限，然后我们开始编码。我们将在第十三章中更多地讨论`List<Vehicle>`构造。现在，假设它代表实现`Vehicle`接口的对象列表就足够了。

现在，我们可以编写以下客户端代码：

```java
double timeSec = 5;
int vehiclesCount = 4;
List<Vehicle> traffic = Vehicle.getTraffic(vehiclesCount);
for(Vehicle vehicle: traffic){
  System.out.println("Loaded: " + vehicle.getSpeedMph(timeSec));
  if(vehicle instanceof Car){
    ((Car) vehicle).setPassengersCount(0); 
    System.out.println("Car(no load): " + vehicle.getSpeedMph(timeSec));
  } else {
    ((Truck) vehicle).setPayloadPounds(0);
    System.out.println("Truck(no load): " + vehicle.getSpeedMph(timeSec));
  }
}
```

前面的代码从`TrafficFactory`中检索任意数量的车辆（在本例中为 4 辆）。工厂隐藏（封装）了交通建模实现的细节。然后，代码在 for 循环中对列表进行迭代（参见第十章，*控制流语句*），并打印出每辆车在车辆开始移动后 5 秒的速度。

然后，代码演示了客户端可以更改车辆携带的负载，这是必需的。对于汽车，我们将乘客人数设置为零，对于卡车，我们将它们的有效载荷设置为零。

我们执行此代码并没有得到结果，因为交通工厂返回了一个空列表。但是代码编译并运行，我们可以开始实现接口。我们可以将任务分配给不同的团队成员，只要他们不改变接口，我们就不必担心协调他们之间的工作。

确保接口、继承和多态性得到充分利用后，我们可以将注意力转向编码细节。

# 尽量解耦

我们选择了继承来实现代码在不同实现之间的共享。结果如下。这是`VehicleImpl`类：

```java
abstract class VehicleImpl implements Vehicle {
  private int weightPounds, horsePower;
  public VehicleImpl(int weightPounds, int horsePower) {
    this.weightPounds = weightPounds;
    this.horsePower = horsePower;
  }
  protected int getWeightPounds(){ return this.weightPounds; }
  protected double getSpeedMph(double timeSec, int weightPounds){
    double v = 2.0 * this.horsePower * 746 * timeSec * 
                                          32.174 / weightPounds;
    return Math.round(Math.sqrt(v) * 0.68);
  }
}
```

请注意，一些方法具有`protected`访问权限，这意味着只有相同包和类子类的成员才能访问它们。这也是为了更好地封装。我们的代码客户端不需要访问这些方法，只有子类需要。以下是其中一个：

```java
class CarImpl extends VehicleImpl implements Car {
  private int passengersCount;
  public CarImpl(int passengersCount, int weightPounds, int horsePower){
    super(weightPounds , horsePower);
    this.passengersCount = passengersCount;
  }
  public void setPassengersCount(int passengersCount) {
    this.passengersCount = passengersCount;
  }
  protected int getWeightPounds(){ 
    return this.passengersCount * 200 + super.getWeightPounds(); 
  }
  public double getSpeedMph(double timeSec){
    return getSpeedMph(timeSec, this.getWeightPounds());
  }
}
```

在前面的代码中，`this`和`super`关键字允许我们区分应该调用哪个方法-当前子对象中的方法还是父对象中的方法。

前面实现的另外两个方面值得注意：

+   `getWeightPounds()` 方法的访问修饰符设置为`protected`。这是因为在父类中也声明了具有相同签名和`protected`访问修饰符的方法。但是，重写的方法不能比被重写的方法具有更严格的访问权限。或者，为了加强封装性，我们可以在`CarImpl`中更改方法名称为`getCarWeightPounds()`，并将其设置为私有。然后，就不需要使用`this`和`super`关键字了。但是，另一个包中的类无法访问`protected`方法，因此我们决定保留`getWeightPounds()`名称并使用`this`和`super`关键字，承认这只是一种风格问题。

+   构造函数的访问权限也可以设置为默认（包级别）。

`TruckImpl`类看起来类似于以下代码片段：

```java
class TruckImpl extends VehicleImpl implements Truck {
  private int payloadPounds;
  TruckImpl(int payloadPounds, int weightPounds, int horsePower) {
    super(weightPounds, horsePower);
    this.payloadPounds = payloadPounds;
  }
  public void setPayloadPounds(int payloadPounds) {
    this.payloadPounds = payloadPounds;
  }
  protected int getWeightPounds(){ 
    return this.payloadPounds + super.getWeightPounds(); 
  }
  public double getSpeedMph(double timeSec){
    return getSpeedMph(timeSec, this.getWeightPounds());
  }
}
```

`TrafficFactory`类可以访问这些类和它们的构造函数来根据需要创建对象：

```java
public class TrafficFactory {
  public static List<Vehicle> get(int vehiclesCount) {
    List<Vehicle> list = new ArrayList();
    for (int i = 0; i < vehiclesCount; i++){
      Vehicle vehicle;
      if (Math.random() <= 0.5) {
        vehicle = new CarImpl(2, 2000, 150);
      } else {
        vehicle = new TruckImpl(500, 3000, 300);
      }
      list.add(vehicle);
    }
    return list;
  }
}
```


`Math`类的`random()`静态方法生成 0 到 1 之间的随机十进制数。我们用它来使交通的结果看起来有些真实。而且，目前我们在每辆车辆的构造函数中传递的值是硬编码的。

现在，我们可以运行以下代码（我们已经在前面的几页中讨论过）：

```java
public class TrafficApp {
  public static void main(String... args){
    double timeSec = 5;
    int vehiclesCount = 4;
    List<Vehicle> traffic = Vehicle.getTraffic(vehiclesCount);
    for(Vehicle vehicle: traffic){
      System.out.println("Loaded: " + vehicle.getSpeedMph(timeSec));
      if(vehicle instanceof Car){
        ((Car) vehicle).setPassengersCount(0);
        System.out.println("Car(no load): " + 
                           vehicle.getSpeedMph(timeSec));
      } else {
        ((Truck) vehicle).setPayloadPounds(0);
        System.out.println("Truck(no load): " + 
                           vehicle.getSpeedMph(timeSec));
      }
    }
  }
}
```

结果如下：

![](img/3ff20e22-6228-435a-9211-4fcd408691b6.png)

计算得到的速度是相同的，因为输入数据在`TrafficFactory`中是硬编码的。但在我们继续并使输入数据不同之前，让我们创建一个速度计算测试：

```java
package com.packt.javapath.ch08demo.traffic.impl;

class SpeedCalculationTest {
  @Test
  void speedCalculation() {
    double timeSec = 5;
    Vehicle vehicle = new CarImpl(2, 2000, 150);
    assertEquals(83.0, vehicle.getSpeedMph(timeSec));
    ((Car) vehicle).setPassengersCount(0);
    assertEquals(91.0, vehicle.getSpeedMph(timeSec));

    vehicle = new TruckImpl(500, 3000, 300);
    assertEquals(98.0, vehicle.getSpeedMph(timeSec));
    ((Truck) vehicle).setPayloadPounds(0);
    assertEquals(105.0, vehicle.getSpeedMph(timeSec));
   }
}
```

我们可以访问`CarImpl`和`TruckImpl`类，因为该测试属于同一个包，尽管它位于项目的不同目录中（在`test`目录下，而不是`main`）。在类路径上，它们根据其包的位置放置，即使源来自另一个源树。

我们已经测试了我们的代码，现在我们可以专注于处理真实数据并为客户在`TrafficFactory`中创建相应的对象。实现与接口解耦，直到准备好为止，我们可以保持其硬编码状态，以便客户端可以开始编写和测试他们的代码，而无需等待我们的系统完全功能可用。这是封装和接口的另一个优点。

# 优先选择聚合而非继承

在现实项目中工作过的人都知道需求随时可能变化。在我们的项目中，甚至在第二次迭代完成之前，就需要向`Car`和`Truck`接口添加新的方法，同时速度计算在自己的项目中增长。负责实现接口的程序员和负责速度计算的程序员开始修改`CarImpl`、`TruckImpl`和`VehicleImpl`文件。

不仅如此，另一个项目决定使用我们的速度计算功能，但他们想将其应用于其他对象，而不是汽车和卡车。那时我们意识到需要改变我们的实现，以支持聚合功能而非继承功能，这也是一般情况下推荐的设计策略之一，因为它增加了解耦和促进了更灵活的设计。这是什么意思。

我们将`VehicleImpl`类的`getSpeedMph()`方法复制到一个新的`com.packt.javapath.ch08demo.speedmodel.impl`包中的`SpeedModelImpl`类中。

```java
class SpeedModelImpl implements SpeedModel {
  public double getSpeedMph(double timeSec, int weightPounds,
                            int horsePower){
    double v = 2.0 * horsePower * 746 * timeSec * 32.174 / weightPounds;
    return Math.round(Math.sqrt(v) * 0.68);
  }
}
```

我们将`SpeedModelFactory`添加到同一个包中：


```java
public class SpeedModelFactory {
  public static SpeedModel speedModel(){
    return new SpeedModelImpl();
  }
}
```

然后我们在`com.packt.javapath.ch08demo.speedmodel`包中创建了一个`SpeedModel`接口：

```java
public interface SpeedModel {
  double getSpeedMph(double timeSec, int weightPounds, int horsePower);
  static SpeedModel getInstance(Month month, int dayOfMonth, int hour){
    return SpeedModelFactory.speedModel(month, dayOfMonth, hour);
  }
}
```

现在，我们通过为`SpeedModel`对象添加一个 setter 并在速度计算中使用此对象来更改`VehicleImpl`类：

```java
abstract class VehicleImpl implements Vehicle {
  private int weightPounds, horsePower;
  private SpeedModel speedModel;
  public VehicleImpl(int weightPounds, int horsePower) {
    this.weightPounds = weightPounds;
    this.horsePower = horsePower;
  }
  protected int getWeightPounds(){ return this.weightPounds; }
  protected double getSpeedMph(double timeSec, int weightPounds){
    if(this.speedModel == null){
      throw new RuntimeException("Speed model is required");
    } else {
      return speedModel.getSpeedMph(timeSec, weightPounds, horsePower);
    }
  }
  public void setSpeedModel(SpeedModel speedModel) {
    this.speedModel = speedModel;
  }
}
```

正如您所看到的，如果在设置 SpeedModel 对象之前调用`getSpeedMph（）`方法，它现在会抛出异常（并停止工作）。

我们还更改了`TrafficFactory`并让它在交通对象上设置`SpeedModel`：

```java
public class TrafficFactory {
  public static List<Vehicle> get(int vehiclesCount) {
    SpeedModel speedModel = SpeedModelFactory.speedModel();
    List<Vehicle> list = new ArrayList();
    for (int i = 0; i < vehiclesCount; i++) {
      Vehicle vehicle;
      if (Math.random() <= 0.5) {
        vehicle = new CarImpl(2, 2000, 150);
      } else {
        vehicle = new TruckImpl(500, 3000, 300);
      }
      ((VehicleImpl)vehicle).setSpeedModel(speedModel);
      list.add(vehicle);
    }
    return list;
  }
}
```

现在，速度模型继续独立于交通模型进行开发，我们完成了所有这些而不改变客户端的代码（这种不影响接口的内部代码更改称为**重构**）。这是封装和接口解耦的好处。`Vehicle`对象的行为现在是聚合的，这使我们能够在不修改其代码的情况下更改其行为。

尽管本节的标题是*优先使用聚合而不是继承*，但这并不意味着继承应该总是被避免。继承有其自身的用途，对于多态行为尤其有益。但是当我们谈论设计灵活性和代码可重用性时，它有两个弱点：

+   Java 类不允许我们扩展超过一个父类，因此，如果类已经是子类，则不能扩展另一个类以重用其方法

+   继承需要类之间的父子关系，而无关的类通常共享相同的功能

有时，继承是解决手头问题的唯一方法，有时使用它会在以后引起问题。现实情况是我们永远无法可靠地预测未来会发生什么，因此如果使用继承或不使用继承的决定最终是错误的话，不要感到难过。

# 这么多 OOD 原则，时间却那么少

如果您在互联网上搜索 OOD 原则，您很容易找到许多包含数十个推荐设计原则的列表。它们都有意义。

例如，以下是经常捆绑在一起的五个最受欢迎的 OOD 原则，缩写为 SOLID（由原则标题的第一个字母组成）：

+   **单一责任原则**：一个类应该只有一个责任

+   **开闭原则**：一个类应该封装其功能（关闭），但应该能够扩展

+   **里氏替换原则**：对象应该能够被其子对象替换（替换）而不会破坏程序

+   **接口隔离原则**：许多面向客户的接口比一个通用接口更好

+   **依赖反转原则**：代码应该依赖于接口，而不是实现。

正如我们之前所说，关于如何实现更好的设计还有许多其他好主意。你应该学习所有这些吗？答案很大程度上取决于你喜欢学习新技能的方式。有些人通过实验来学习，其他人通过借鉴他人的经验来学习，大多数人则是通过这两种方法的结合来学习。

好消息是，我们在本章讨论的设计标准、面向对象的概念以及良好设计的路线图，能够在大多数情况下引导你找到一个坚实的面向对象设计解决方案。

但如果你决定了解更多关于面向对象设计，并看看其他人是如何解决软件设计问题的，不要犹豫去了解它们。毕竟，人类是通过将他们的经验传递给下一代，才走出了洞穴，登上了宇宙飞船。

# 练习-设计模式

有许多面向对象设计模式共享了特定编码问题的软件设计解决方案。面向对象设计模式也经常被程序员用来讨论不同的实现方式。

它们通常被分为四类：创建、行为、结构和并发模式。阅读它们并：

+   在每个类别中列出一种模式

+   列出我们已经使用过的三种模式

# 答案

四种模式——每种类别中的一种——可能是以下这些：

+   **创建模式**：工厂方法

+   **结构模式**：组合

+   **行为模式**：访问者

+   **并发模式**：消息模式

在这本书中，我们已经使用了以下模式：

+   **延迟初始化**：在第六章中，*接口、类和对象构造*，我们初始化了`SingletonClassExample OBJECT`静态字段，但只有在调用`getInstance()`方法时才会初始化

+   **单例模式**：在第六章中，*接口、类和对象构造*，查看`SingletonClassExample`类

+   **外观模式**：在第六章中，*接口、类和对象构造*，当我们创建了一个`Calculator`接口，用于捕捉对实现功能的所有可能交互

# 总结

在本章中，我们重新审视了编程的高层视图，特别是 Java 编程。我们讨论了软件系统开发过程中的设计演变，从最早的可行性阶段开始，经过高层设计、详细设计，最终到编码和测试。我们讨论了良好设计的标准，面向对象的概念，主要的面向对象设计原则，并提供了一个良好面向对象设计的路线图。我们通过代码示例来说明所有讨论过的面向对象设计原则的应用。

在下一章中，我们将更深入地探讨 Java 编程的三个核心元素：运算符、表达式和语句。我们将定义并讨论所有 Java 运算符，更详细地探讨最流行的运算符，并在具体示例中演示它们，以及表达式和语句。
