与接口相关的最后一个非常重要的术语是抽象方法。接口中列出的没有实现的方法签名称为**抽象方法**，接口本身称为**抽象**，因为它抽象化、总结并移除了实现中的方法签名。抽象不能被实例化。例如，如果在任何类前面放置关键字`abstract`并尝试创建其对象，即使类中的所有方法都不是抽象的，编译器也会抛出错误。在这种情况下，类仅作为具有默认方法的接口。然而，在它们的使用上有显著的区别，您将在本章的接下来的*继承*部分中看到。

我们将在第六章*接口，类和对象构建*中更多地讨论接口，并在第七章*包和可访问性（可见性）*中涵盖它们的访问修饰符。

# 实现

一个接口可以被类实现，这意味着该类为接口中列出的每个抽象方法提供了一个具体的实现。这里是一个例子：

```java
interface Car {
  double getWeightInPounds();
  double getMaxSpeedInMilesPerHour();
}

public class CarImpl implements Car{
  public double getWeightInPounds(){
    return 2000d;
  }
  public double getMaxSpeedInMilesPerHour(){
    return 100d;
  }
}
```

我们将类命名为`CarImpl`，表示它是接口`Car`的实现。但是我们可以随意为其命名。

接口及其类实现也可以有其他方法，而不会引起编译错误。接口中额外方法的唯一要求是必须是默认方法并有具体实现。向类添加任何其他方法都不会干扰接口实现。例如：

```java
interface Car {
  double getWeightInPounds();
  double getMaxSpeedInMilesPerHour();
  default int getPassengersCount(){
    return 4;
  } 
}

public class CarImpl implements Car{
  private int doors;
  private double weight, speed;
  public CarImpl(double weight, double speed, int doors){
    this.weight = weight;
    this.speed = speed;
    this.dooes = doors;
  }
  public double getWeightInPounds(){
    return this.weight;
  }
  public double getMaxSpeedInMilesPerHour(){
    return this.speed;
  }
  public int getNumberOfDoors(){
    return this.doors;
  }
}
```

如果我们现在创建一个`CarImpl`类的实例，我们可以调用类中声明的所有方法：

```java
CarImpl car = new CarImpl(500d, 50d, 3); 
car.getWeightInPounds();         //Will return 500.0
car.getMaxSpeedInMilesPerHour(); //Will return 50.0
car.getNumberOfDoors();          //Will return 3

```

这并不令人惊讶。

但是，这里有一些你可能意想不到的：

```java
car.getPassengersCount();          //Will return 4
```

这意味着通过实现一个接口，类获得了接口默认方法。这就是默认方法的目的：为实现接口的所有类添加功能。如果没有默认方法，如果向旧接口添加一个抽象方法，所有当前的接口实现将触发编译错误。但是，如果添加一个带有`default`修饰符的新方法，现有的实现将继续像往常一样工作。

现在，另一个很好的技巧。如果一个类实现了与默认方法相同签名的方法，它将`覆盖`（一个技术术语）接口的行为。这里是一个例子：
