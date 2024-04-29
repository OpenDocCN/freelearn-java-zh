# 进行函数式编程

本章介绍了一种称为函数式编程的编程范式，以及它在 Java 11 中的适用性。我们将涵盖以下内容：

+   使用标准功能接口

+   创建函数式接口

+   理解 lambda 表达式

+   使用 lambda 表达式

+   使用方法引用

+   在程序中利用 lambda 表达式

# 介绍

函数式编程是将某个功能作为对象对待，并将其作为方法的参数或返回值传递的能力。这个特性存在于许多编程语言中，Java 在 Java 8 发布时获得了这个特性。

它避免了创建类、对象和管理对象状态。函数的结果仅取决于输入数据，无论调用多少次。这种风格使结果更可预测，这是函数式编程最吸引人的方面。

它也让我们能够通过将并行性的责任从客户端代码转移到库中，来改进 Java 中的并行编程能力。在此之前，为了处理 Java 集合的元素，客户端代码必须从集合中获取迭代器并组织集合的处理。

Java 集合的一些默认方法接受一个函数（函数式接口的实现）作为参数，然后将其应用于集合的每个元素。因此，库的责任是组织处理。一个例子是在每个 Iterable 接口中都可用的 forEach(Consumer)方法，其中 Consumer 是一个函数式接口。另一个例子是在每个 Collection 接口中都可用的 removeIf(Predicate)方法，其中 Predicate 也是一个函数式接口。此外，List 接口中添加了 sort(Comparator)和 replaceAll(UnaryOperator)方法，Map 中添加了 compute()方法。

Lambda 表达式利用函数式接口，并显著简化了它们的实现，使代码更短、更清晰、更具表现力。

在本章中，我们将讨论函数式编程的优势，定义和解释函数式接口和 lambda 表达式，并在代码示例中演示所有相关功能。

使函数成为语言的一等公民为 Java 增加了更多的功能。但利用这种语言能力需要——对于尚未接触函数式编程的人来说——一种新的思维方式和代码组织方式。

解释这一新特性并分享使用它的最佳实践是本章的目的。

# 使用标准功能接口

在这个示例中，您将学习什么是函数式接口，以及为什么它被添加到 Java 中，以及 JDK 8 中附带的标准 Java 库中的 43 个可用的函数式接口。

没有函数式接口，将功能传递到方法的唯一方法是通过编写一个类，创建其对象，然后将其作为参数传递。但即使是最不涉及的样式——使用匿名类——也需要编写太多的代码。使用函数式接口有助于避免所有这些。

# 准备工作

任何具有一个且仅有一个抽象方法的接口都被称为函数式接口。为了避免运行时错误，可以在接口前面添加@FunctionalInterface 注解。它告诉编译器意图，因此编译器可以检查该接口中是否实际上有一个抽象方法，包括从其他接口继承的方法。

在前几章的演示代码中，我们已经有了一个函数式接口的示例，即使我们没有将其注释为函数式接口。

```java
public interface SpeedModel {
  double getSpeedMph(double timeSec, int weightPounds, int horsePower);
  enum DrivingCondition {
    ROAD_CONDITION,
    TIRE_CONDITION
  }
  enum RoadCondition {
    //...
  }
  enum TireCondition {
    //...
  }
}
```

`enum`类型的存在或任何实现的（默认或静态）方法并不会使其成为非功能接口。只有抽象（未实现）方法才算。因此，这也是一个功能接口的例子：

```java
public interface Vehicle {
  void setSpeedModel(SpeedModel speedModel);
  default double getSpeedMph(double timeSec){ return -1; };
  default int getWeightPounds(){ return -1; }
  default int getWeightKg(){ 
    return convertPoundsToKg(getWeightPounds());
  }
  private int convertPoundsToKg(int pounds){
    return (int) Math.round(0.454 * pounds);
  }
  static int convertKgToPounds(int kilograms){
    return (int) Math.round(2.205 * kilograms);
  }
}
```

回顾您在第二章中已经学到的关于接口的默认方法，`getWeightPounds()`方法在被`getWeightKg()`调用或直接调用时将返回`-1`，使用实现`Vehicle`接口的类的对象。但是，只有在类中未实现`getWeightPounds()`方法时才会如此。否则，将使用类的实现并返回不同的值。

除了默认和静态接口方法，功能接口还可以包括`java.lang.Object`基类的任何和所有抽象方法。在 Java 中，每个对象都提供了`java.lang.Object`方法的默认实现，因此编译器和 Java 运行时会忽略这样的抽象方法。

例如，这也是一个功能接口：

```java
public interface SpeedModel {
  double getSpeedMph(double timeSec, int weightPounds, int horsePower);
  boolean equals(Object obj);
  String toString();
}
```

以下虽然不是功能接口：

```java
public interface Car extends Vehicle {
   int getPassengersCount();
}
```

这是因为`Car`接口有两个抽象方法——它自己的`getPassengersCount()`方法和从`Vehicle`接口继承的`setSpeedModel(SpeedModel speedModel)`方法。

我们可以尝试将`@FunctionalInterface`注解添加到`Car`接口中：

```java
@FunctionalInterface 
public interface Car extends Vehicle {
   int getPassengersCount();
}
```

如果我们这样做，编译器将生成以下错误：

![](img/df18df69-1aed-4f20-b3cf-b2c5f741d2f6.png)

使用`@FunctionalInterface`注解不仅有助于在编译时捕获错误，而且还确保了程序员之间设计意图的可靠沟通。它可以帮助您或其他程序员记住该接口不能有多个抽象方法，这在已经存在依赖于这种假设的一些代码时尤其重要。

出于同样的原因，`Runnable`和`Callable`接口（它们自 Java 早期版本以来就存在）在 Java 8 中被注释为`@FunctionalInterface`，以明确区分：

```java
@FunctionalInterface
interface Runnable { void run(); }

@FunctionalInterface
interface Callable<V> { V call() throws Exception; }
```

# 如何做…

在创建自己的功能接口之前，首先考虑使用`java.util.function`包中提供的 43 个功能接口中的一个。它们中的大多数是`Function`、`Consumer`、`Supplier`和`Predicate`接口的特殊化。

以下是您可以遵循的步骤，以熟悉功能接口：

1.  看看`Function<T,R>`功能接口：

```java
        @FunctionalInterface
        public interface Function<T,R>
```

从`<T,R>`泛型中可以看出，该接口的唯一方法接受`T`类型的参数并返回`R`类型的值。根据 JavaDoc，该接口具有`R apply(T t)`方法。我们可以使用匿名类创建该接口的实现：

```java
      Function<Integer, Double> ourFunc = 
          new Function<Integer, Double>() {
              public Double apply(Integer i){
                  return i * 10.0;
              }
          };
```

我们的实现中的`R apply(T t)`方法接受`Integer`类型的值（或将自动装箱的`int`原始类型），将其乘以`10`，并返回`Double`类型的值，以便我们可以如下使用我们的新函数：

```java
        System.out.println(ourFunc.apply(1));  //prints: 10
```

在下面的*理解 lambda 表达式*的示例中，我们将介绍 lambda 表达式，并向您展示它的使用方式如何使实现变得更短。但现在，我们将继续使用匿名类。

1.  看看`Consumer<T>`功能接口。名称帮助我们记住该接口的方法接受一个值，但不返回任何东西——它只消耗。它的唯一方法是`void accept(T)`。该接口的实现可以如下所示：

```java
        Consumer<String> ourConsumer = new Consumer<String>() {
          public void accept(String s) {
            System.out.println("The " + s + " is consumed.");
          }
        };
```

我们的实现中的`void accept(T t)`方法接收`String`类型的值并打印它。例如，我们可以这样使用它：

```java
          ourConsumer.accept("Hello!");  
                        //prints: The Hello! is consumed.
```

1.  看看`Supplier<T>`功能接口。名称帮助您记住该接口的方法不接受任何值，但确实返回一些东西——只提供。它的唯一方法是`T get()`。基于此，我们可以创建一个函数：

```java
        Supplier<String> ourSupplier = new Supplier<String>() {
          public String get() {
            String res = "Success";
            //Do something and return result—Success or Error.
            return res;
          }
        };
```

我们的实现中的`T get()`方法执行某些操作，然后返回`String`类型的值，因此我们可以编写以下内容：

```java
        System.out.println(ourSupplier.get());   //prints: Success
```

1.  看一下`Predicate<T>`函数接口。名称有助于记住该接口的方法返回一个布尔值——它预测某些东西。它的唯一方法是`boolean test(T t)`，这意味着我们可以创建以下函数：

```java
        Predicate<Double> ourPredicate = new Predicate<Double>() {
          public boolean test(Double num) {
            System.out.println("Test if " + num + 
                               " is smaller than 20");
            return num < 20;
          }
        };
```

我们的实现的`boolean test(T t)`方法接受`Double`类型的值作为参数，并返回`boolean`类型的值，因此我们可以这样使用它：

```java
        System.out.println(ourPredicate.test(10.0) ? 
                           "10 is smaller" : "10 is bigger");
```

这将产生以下结果：

![](img/e7d9f55b-d73b-41f4-9142-39cfb2f4fe29.png)

1.  查看`java.util.function`包中的其他 39 个函数接口。请注意，它们是我们已经讨论过的四个接口的变体。这些变体是为以下原因而创建的：

+   +   为了通过显式使用`int`、`double`或`long`原始类型来避免自动装箱和拆箱而获得更好的性能

+   接受两个输入参数

+   更短的表示法

以下函数接口只是 39 个接口列表中的几个示例。

`IntFunction<R>`函数接口具有`R apply(int i)`抽象方法。它提供了更短的表示法（不带参数类型的泛型）并避免了自动装箱（通过将`int`原始类型定义为参数）。以下是其用法示例：

```java
        IntFunction<String> iFunc = new IntFunction<String>() {
          public String apply(int i) {
            return String.valueOf(i * 10);
          }
        };
        System.out.println(iFunc.apply(1));    //prints: 10
```

`BiFunction<T,U,R>`函数接口具有抽象方法`R apply(T,U)`。以下是其实现示例：

```java
        BiFunction<String, Integer, Double> biFunc = 
                         new BiFunction<String, Integer, Double >() {
           public Double apply(String s, Integer i) {
             return (s.length() * 10d) / i;
           }
        };
        System.out.println(biFunc.apply("abc", 2)); //prints: 15.0

```

`BinaryOperator<T>`函数接口具有一个抽象方法`T apply(T,T)`。它通过避免重复三次相同类型提供了更短的表示法。以下是其用法示例：

```java
       BinaryOperator<Integer> function = new BinaryOperator<Integer>(){
           public Integer apply(Integer i, Integer j) {
             return i >= j ? i : j;
           }
       };
        System.out.println(binfunc.apply(1, 2));     //prints: 2

```

`IntBinaryOperator`函数接口具有`int applyAsInt(int,int)`抽象方法。我们可以使用它来复制前面示例中的相同功能：

```java
        IntBinaryOperator intBiFunc = new IntBinaryOperator(){
            public int applyAsInt(int i, int j) {
                return i >= j ? i : j;
            }
        };
        System.out.println(intBiFunc.applyAsInt(1, 2)); //prints: 2
```

接下来的食谱将提供更多此类专业化用法示例。

# 它的工作原理...

我们可以仅使用函数来组成整个方法：

```java
void calculate(Supplier<Integer> source, 
  Function<Integer, Double> process, Predicate<Double> condition,
              Consumer<Double> success, Consumer<Double> failure){
    int i = source.get();
    double res = process.apply(i);
    if(condition.test(res)){
        success.accept(res);
    } else {
        failure.accept(res);
    }
}
```

前面的代码从源中获取值，处理它，然后根据提供的函数决定结果是否成功。现在，让我们创建这些函数并调用该方法。我们决定源参数如下：

```java
Supplier<Integer> source = new Supplier<Integer>() {
    public Integer get() {
        Integer res = 42;
        //Do something and return result value
        return res;
    }
};

```

在实际代码中，此函数可以从数据库或任何其他数据源中提取数据。我们保持简单——使用硬编码的返回值——以获得可预测的结果。

处理函数和谓词将保持与以前相同的方式：

```java
Function<Integer, Double> process = new Function<Integer, Double>(){
    public Double apply(Integer i){
        return i * 10.0;
    }
};
Predicate<Double> condition = new Predicate<Double>() {
    public boolean test(Double num) {
        System.out.println("Test if " + num + 
                                    " is smaller than " + 20);
        return num < 20;
    }
}; 
```

消费者几乎相同，只是在打印结果之前有不同的前缀：

```java
Consumer<Double> success = new Consumer<Double>() {
    public void accept(Double d) {
        System.out.println("Success: " + d);
    }
};
Consumer<Double> failure = new Consumer<Double>() {
    public void accept(Double d) {
        System.out.println("Failure: " + d);
    }
};

```

现在我们可以调用 calculate 方法，如下所示：

```java
calculate(source, process, condition, success, failure);

```

结果将如下所示：

```java
Test if 420.0 is smaller than 20.0
Failure: 420.0
```

如果我们需要快速测试源值和谓词条件的各种组合，我们可以创建`testSourceAndCondition(int src, int limit)`方法，如下所示：

```java
void testSourceAndCondition(int src, double condition) {
    Supplier<Integer> source = new Supplier<Integer>() {
        public Integer get() {
            Integer res = src;
            //Do something and return result value
            return res;
        }
    };
    Function<Integer, Double> process = 
      new Function<Integer, Double>() {
         public Double apply(Integer i){
            return i * 10.0;
         }
      };
    Predicate<Double> condition = new Predicate<Double>() {
        public boolean test(Double num) {
            System.out.println("Test if " + num + 
                                " is smaller than " + limit);
            return num < limit;
        }
    };
    Consumer<Double> success = new Consumer<Double>() {
        public void accept(Double d) {
            System.out.println("Success: " + d);
        }
    };
    Consumer<Double> failure = new Consumer<Double>() {
        public void accept(Double d) {
            System.out.println("Failure: " + d);
        }
    };
    calculate(source, process, cond, success, failure);
}
```

注意我们如何将`src`值传递给`source`供应商，将`limit`值传递给`condition`谓词。现在，我们可以运行`testSourceAndCondition(int src, int limit)`方法，使用不同的输入值寻找`src`值和`limit`值的组合，以获得成功：

```java
testSourceAndCondition(10, 20);
testSourceAndCondition(1, 20);
testSourceAndCondition(10, 200);

```

结果将如下所示：

```java
Test if 100.0 is smaller than 20.0
Failure: 100.0
Test if 10.0 is smaller than 20.0
Success: 10.0
Test if 100.0 is smaller than 200.0
Success: 100.0
```

# 还有更多...

`java.util.function`包中的许多函数接口都具有默认方法，不仅增强了它们的功能，还允许您将函数链接在一起，并将一个函数的结果作为输入参数传递给另一个函数。例如，我们可以使用`Function<T,V> andThen(Function<R,V> after)`接口的默认方法：

```java
Function<Integer, Double> before = new Function<Integer, Double>(){
    public Double apply(Integer i){
        return i * 10.0;
    }
};
Function<Double, Double> after = new Function<Double, Double>(){
    public Double apply(Double d){
        return d + 10.0;
    }
};
Function<Integer, Double> process = before.andThen(after);
```

如您所见，我们的`process`函数现在是我们原始函数（将源值乘以 10.0）和一个新函数`after`的组合，该函数将 10.0 添加到第一个函数的结果中。如果我们调用`testSourceAndCondition(int source, int condition)`方法，如`testSourceAndCondition(42, 20)`，结果将如下所示：

```java
Test if 430.0 is smaller than 20
Failure: 430.0
```

`Supplier<T>`接口没有允许我们链接多个函数的方法，但`Predicate<T>`接口有`and(Predicate<T> other)`和`or(Predicate<T> other)`默认方法，允许我们构造更复杂的布尔表达式。`Consumer<T>`接口也有`andThen(Consumer<T> after)`默认方法。

注意`after`函数的输入值类型必须与`before`函数的结果类型匹配：

```java
Function<T,R> before = ...
Function<R,V> after = ...
Function<T,V> result = before.andThen(after);
```

生成的函数接受`T`类型的值并产生`V`类型的值。

实现相同结果的另一种方法是使用`Function<V,R> compose(Function<V,T> before)`默认方法：

```java
Function<Integer, Double> process = after.compose(before);
```

要使用`andThen()`或`compose()`中的哪种方法取决于哪个函数可用于调用聚合方法。然后，一个被认为是基础，而另一个是参数。

如果这种编码看起来有点过度设计和复杂，那是因为确实如此。我们只是为了演示目的而这样做的。好消息是，下一个示例中介绍的 lambda 表达式可以让我们以更简洁和更清晰的方式实现相同的结果。

`java.util.function`包的函数接口还有其他有用的默认方法。其中一个突出的是`identity()`方法，它返回一个始终返回其输入参数的函数：

```java
Function<Integer, Integer> id = Function.identity();
System.out.println(id.apply(4));  //prints: 4
```

`identity()`方法在需要提供某个函数但不希望该函数修改结果时非常有用。

其他默认方法大多与转换、装箱、拆箱以及提取两个参数的最小值和最大值有关。我们鼓励您浏览`java.util.function`的所有函数接口的 API，并了解可能性。

# 创建函数接口

在本示例中，您将学习如何在`java.util.function`包中没有满足要求的标准接口时创建和使用自定义函数接口。

# 准备工作

创建函数接口很容易。只需确保接口中只有一个抽象方法，包括从其他接口继承的方法：

```java
@FunctionalInterface
interface A{
    void m1();
}

@FunctionalInterface
interface B extends A{
    default void m2(){};
}

//@FunctionalInterface
interface C extends B{
    void m3();
}
```

在前面的示例中，接口`C`不是函数接口，因为它有两个抽象方法-`m1()`，从接口`A`继承，以及它自己的方法`m3()`。

我们已经看到了`SpeedModel`函数接口：

```java
@FunctionalInterface
public interface SpeedModel {
  double getSpeedMph(double timeSec, int weightPounds, int horsePower);
}
```

我们已经对其进行了注释以表达意图，并在`SpeedModel`接口中添加另一个抽象方法时得到警告。为了简化，我们已将`enum`类从中删除。此接口用于`Vehicle`接口：

```java
public interface Vehicle {
    void setSpeedModel(SpeedModel speedModel);
    double getSpeedMph(double timeSec);
}
```

`Vehicle`实现需要它的原因是`SpeedModel`是计算速度功能的来源：

```java
public class VehicleImpl implements Vehicle {
    private SpeedModel speedModel;
    private int weightPounds, hoursePower;
    public VehicleImpl(int weightPounds, int hoursePower){
        this.weightPounds = weightPounds;
        this.hoursePower = hoursePower;
    }
    public void setSpeedModel(SpeedModel speedModel){
        this.speedModel = speedModel;
    }
    public double getSpeedMph(double timeSec){
        return this.speedModel.getSpeedMph(timeSec, 
                                 this.weightPounds, this.hoursePower);
    };
}
```

正如我们在第二章中提到的*OOP 快速通道-类和接口*，这种设计被称为聚合。这是组合所需行为的首选方式，因为它允许更灵活性。

使用函数接口，这种设计变得更加灵活。为了演示，让我们实现我们的自定义接口`SpeedModel`。

# 如何做...

传统的方法是创建一个实现`SpeedModel`接口的类：

```java
public class SpeedModelImpl implements SpeedModel {
   public double getSpeedMph(double timeSec, 
                       int weightPounds, int horsePower){
      double v = 2.0 * horsePower * 746 * 
                       timeSec * 32.17 / weightPounds;
      return (double) Math.round(Math.sqrt(v) * 0.68);
   }
}
```

然后，我们可以按以下方式使用此实现：

```java
Vehicle vehicle = new VehicleImpl(3000, 200);
SpeedModel speedModel = new SpeedModelImpl();
vehicle.setSpeedModel(speedModel);
System.out.println(vehicle.getSpeedMph(10.)); //prints: 122.0

```

要更改速度计算的方式，我们需要更改`SpeedModelImpl`类。

或者，利用`SpeedModel`是一个接口的事实，我们可以更快地引入更改，甚至避免首先拥有`SpeedModelImpl`类：

```java
Vehicle vehicle = new VehicleImpl(3000, 200);
SpeedModel speedModel = new SpeedModel(){
   public double getSpeedMph(double timeSec, 
                       int weightPounds, int horsePower){
      double v = 2.0 * horsePower * 746 * 
                       timeSec * 32.17 / weightPounds;
      return (double) Math.round(Math.sqrt(v) * 0.68);
   }
};
vehicle.setSpeedModel(speedModel);
System.out.println(vehicle.getSpeedMph(10.)); //prints: 122.0
```

然而，前面的实现没有利用接口是功能性的优势。如果我们注释掉注解，我们可以向`SpeedModel`接口添加另一个方法：

```java
//@FunctionalInterface
public interface SpeedModel {
    double getSpeedMph(double timeSec, 
                    int weightPounds, int horsePower);
    void m1();
}
Vehicle vehicle = new VehicleImpl(3000, 200);
SpeedModel speedModel = new SpeedModel(){
   public double getSpeedMph(double timeSec, 
                     int weightPounds, int horsePower){
      double v = 2.0 * horsePower * 746 * 
                       timeSec * 32.17 / weightPounds;
      return (double) Math.round(Math.sqrt(v) * 0.68);
   }
   public void m1(){}
   public void m2(){}
};
vehicle.setSpeedModel(speedModel);
System.out.println(vehicle.getSpeedMph(10.)); //prints: 122.0
```

从前面的代码中可以看出，不仅`SpeedModel`接口有另一个抽象方法`m1()`，而且匿名类还有另一个未在`SpeedModel`接口中列出的方法`m2()`。因此，匿名类不需要接口是功能性的。但是 lambda 表达式需要。

# 它是如何工作的...

使用 lambda 表达式，我们可以将前面的代码重写如下：

```java
Vehicle vehicle = new VehicleImpl(3000, 200);
SpeedModel speedModel =  (t, wp, hp) -> {
    double v = 2.0 * hp * 746 * t * 32.17 / wp;
    return (double) Math.round(Math.sqrt(v) * 0.68);
};
vehicle.setSpeedModel(speedModel);
System.out.println(vehicle.getSpeedMph(10.)); //prints: 122.0

```

我们将在下一个示例中讨论 lambda 表达式的格式。现在，我们只想指出功能接口对于前面的实现非常重要。正如您所看到的，只指定了接口的名称，没有任何方法名称。这是可能的，因为功能接口只有一个必须实现的方法，这就是 JVM 如何找出并在幕后生成功能接口实现的方式。

# 还有更多...

可以定义一个类似于标准功能接口的通用自定义功能接口。例如，我们可以创建以下自定义功能接口：

```java
@FunctionalInterface
interface Func<T1,T2,T3,R>{ 
   R apply(T1 t1, T2 t2, T3 t3);
}
```

它允许三个输入参数，这正是我们计算速度所需要的：

```java
Func<Double, Integer, Integer, Double> speedModel = (t, wp, hp) -> {
    double v = 2.0 * hp * 746 * t * 32.17 / wp;
    return (double) Math.round(Math.sqrt(v) * 0.68);
};

```

使用这个函数而不是`SpeedModel`接口，我们可以将`Vehicle`接口及其实现更改如下：

```java
interface Vehicle {
   void setSpeedModel(Func<Double, Integer, Integer, 
                                         Double> speedModel);
   double getSpeedMph(double timeSec);
}
class VehicleImpl  implements Vehicle {
   private Func<Double, Integer, Integer, Double> speedModel;
   private int weightPounds, hoursePower;
   public VehicleImpl(int weightPounds, int hoursePower){
       this.weightPounds = weightPounds;
       this.hoursePower = hoursePower;
   }
   public void setSpeedModel(Func<Double, Integer, 
                               Integer, Double> speedModel){
       this.speedModel = speedModel;
   }
   public double getSpeedMph(double timeSec){
       return this.speedModel.apply(timeSec, 
                             weightPounds, hoursePower);
   };
}
```

前面的代码产生了与`SpeedModel`接口相同的结果。

自定义接口的名称和其唯一方法的名称可以是我们喜欢的任何东西。例如：

```java
@FunctionalInterface
interface FourParamFunction<T1,T2,T3,R>{
     R caclulate(T1 t1, T2 t2, T3 t3);
}
```

既然我们无论如何都要创建一个新接口，使用`SpeedModel`名称和`getSpeedMph()`方法名称可能是更好的解决方案，因为这样可以使代码更易读。但是在某些情况下，通用自定义功能接口是更好的选择。在这种情况下，您可以使用前面的定义，并根据需要进行增强。

# 理解 lambda 表达式

我们已经多次提到 lambda 表达式，并指出它们在 Java 中的使用证明了在`java.util.function`包中引入功能接口的必要性。lambda 表达式允许我们通过删除匿名类的所有样板代码来简化函数实现，只留下最少必要的信息。我们还解释了这种简化是可能的，因为功能接口只有一个抽象方法，所以编译器和 JVM 将提供的功能与方法签名进行匹配，并在幕后生成功能接口实现。

现在，是时候定义 lambda 表达式语法并查看 lambda 表达式的可能形式范围了，在我们开始使用它们使我们的代码比使用匿名类时更短更易读之前。

# 准备工作

在 20 世纪 30 年代，数学家阿隆佐·邱奇在研究数学基础时引入了 lambda 演算——一种通用的计算模型，可以用来模拟任何图灵机。那个时候，图灵机还没有被创建。只有后来，当艾伦·图灵发明了他的*a-机器*（自动机），也称为*通用图灵机*时，他和邱奇联手提出了一个邱奇-图灵论题，表明 lambda 演算和图灵机具有非常相似的能力。

Church 使用希腊字母*lambda*来描述匿名函数，它成为了编程语言理论领域的非官方符号。第一个利用 lambda 演算形式的编程语言是 Lisp。Java 在 2014 年发布 Java 8 时添加了函数式编程能力。

lambda 表达式是一个允许我们省略修饰符、返回类型和参数类型的匿名方法。这使得它非常紧凑。lambda 表达式的语法包括参数列表、箭头标记(`->`)和主体。参数列表可以为空（只有括号，`()`），没有括号（如果只有一个参数），或者由括号括起来的逗号分隔的参数列表。主体可以是一个没有括号的单个表达式，也可以是由括号括起来的语句块。

# 如何做到...

让我们看一些例子。以下 lambda 表达式没有输入参数，总是返回`33`：

```java
() -> 33;
```

以下 lambda 表达式接受一个整数类型的参数，将其增加 1，并返回结果：

```java
i -> i++;
```

以下 lambda 表达式接受两个参数并返回它们的和：

```java
(a, b) -> a + b;
```

以下 lambda 表达式接受两个参数，比较它们，并返回`boolean`结果：

```java
(a, b) -> a == b;
```

最后一个 lambda 表达式接受两个参数，计算并打印结果：

```java
(a, b) -> { 
     double c = a +  Math.sqrt(b); 
     System.out.println("Result: " + c);
}
```

正如你所看到的，lambda 表达式可以包含任意大小的代码块，类似于任何方法。前面的例子没有返回任何值。这里是另一个返回`String`值的代码块的例子：

```java
(a, b) -> { 
     double c = a +  Math.sqrt(b); 
     return c > 10.0 ? "Success" : "Failure";
}
```

# 它是如何工作的...

让我们再次看看最后一个例子。如果在*functional*接口`A`中定义了一个`String m1(double x, double y)`方法，并且有一个接受`A`类型对象的`m2(A a)`方法，我们可以这样调用它：

```java
A a = (a, b) -> { 
     double c = a +  Math.sqrt(b); 
     return c > 10.0 ? "Success" : "Failure";
}
m2(a);
```

前面的代码意味着传入的对象有以下`m1()`方法的实现：

```java
public String m1(double x, double y){
     double c = a +  Math.sqrt(b); 
     return c > 10.0 ? "Success" : "Failure";
}
```

`m2(A a)`有`A`对象作为参数告诉我们，`m2(A a)`的代码可能使用了`A`接口的至少一个方法（`A`接口中也可能有默认或静态方法）。但是，一般来说，不能保证方法使用了传入的对象，因为程序员可能决定停止使用它，但保持签名不变，以避免破坏客户端代码，例如。

然而，客户端必须传入实现`A`接口的对象到方法中，这意味着它的唯一抽象方法必须被实现。这就是 lambda 表达式所做的事情。它使用最少的代码定义抽象方法的功能——输入参数列表和方法实现的代码块。这就是编译器和 JVM 生成实现所需的一切。

编写这样紧凑和高效的代码成为可能，是因为 lambda 表达式和函数接口的结合。

# 还有更多...

与匿名类一样，外部创建但在 lambda 表达式内部使用的变量实际上是最终的，不能被修改。你可以写下以下代码：

```java
double v = 10d;
Function<Integer, Double> multiplyBy10 = i -> i * v;
```

然而，你不能在 lambda 表达式外部改变`v`变量的值：

```java
double v = 10d;
v = 30d; //Causes compiler error
Function<Integer, Double> multiplyBy10 = i -> i * v;
```

你也不能在表达式内部改变它：

```java
double v = 10d;
Function<Integer, Double> multiplyBy10 = i -> {
  v = 30d; //Causes compiler error
  return i * v;
};

```

这种限制的原因是一个函数可以在不同的上下文（例如不同的线程）中传递和执行不同的参数，试图同步这些上下文会破坏函数的分布式评估的原始想法。

另一个值得一提的 lambda 表达式特性是它对`this`关键字的解释，这与匿名类的解释有很大不同。在匿名类内部，`this`指的是匿名类的实例，但在 lambda 表达式内部，`this`指的是包围表达式的类的实例。让我们来演示一下，假设我们有以下类：

```java
class Demo{
    private String prop = "DemoProperty";
    public void method(){
        Consumer<String> consumer = s -> {
            System.out.println("Lambda accept(" + s 
                                      + "): this.prop=" + this.prop);
        };
        consumer.accept(this.prop);
        consumer = new Consumer<>() {
            private String prop = "ConsumerProperty";
            public void accept(String s) {
                System.out.println("Anonymous accept(" + s 
                                      + "): this.prop=" + this.prop);
            }
        };
        consumer.accept(this.prop);
    }
}
```

正如你所看到的，在`method()`代码中，`Consumer`函数接口被实现了两次——使用 lambda 表达式和使用匿名类。让我们在以下代码中调用这个方法：

```java
  Demo d = new Demo();
  d.method();
```

输出将如下所示：

![](img/324ceaa7-1714-4153-bd7a-43ae67258283.png)

Lambda 表达式不是内部类，也不能被`this`引用。Lambda 表达式没有字段或属性。它是无状态的。这就是为什么在 lambda 表达式中，`this`关键字指的是周围的上下文。这也是 lambda 表达式要求周围上下文中的所有变量必须是 final 或有效 final 的另一个原因。

# 使用 lambda 表达式

在这个示例中，你将学习如何在实践中使用 lambda 表达式。

# 准备工作

创建和使用 lambda 表达式实际上比编写方法简单得多。只需要列出输入参数（如果有的话），以及执行所需操作的代码。

让我们重新审视本章第一个示例中标准功能接口的实现，并使用 lambda 表达式重写它们。以下是我们使用匿名类实现了四个主要功能接口的方式：

```java
Function<Integer, Double> ourFunc = new Function<Integer, Double>(){
    public Double apply(Integer i){
        return i * 10.0;
    }
};
System.out.println(ourFunc.apply(1));       //prints: 10.0
Consumer<String> consumer = new Consumer<String>() {
    public void accept(String s) {
        System.out.println("The " + s + " is consumed.");
    }
};
consumer.accept("Hello!"); //prints: The Hello! is consumed.
Supplier<String> supplier = new Supplier<String>() {
    public String get() {
        String res = "Success";
        //Do something and return result—Success or Error.
        return res;
    }
};
System.out.println(supplier.get());      //prints: Success
Predicate<Double> pred = new Predicate<Double>() {
    public boolean test(Double num) {
       System.out.println("Test if " + num + " is smaller than 20");
       return num < 20;
    }
};
System.out.println(pred.test(10.0)? "10 is smaller":"10 is bigger");
                           //prints: Test if 10.0 is smaller than 20
                           //        10 is smaller

```

以下是使用 lambda 表达式的样子：

```java
Function<Integer, Double> ourFunc = i -> i * 10.0;
System.out.println(ourFunc.apply(1)); //prints: 10.0

Consumer<String> consumer = 
            s -> System.out.println("The " + s + " is consumed.");
consumer.accept("Hello!");       //prints: The Hello! is consumed.

Supplier<String> supplier = () - > {
        String res = "Success";
        //Do something and return result—Success or Error.
        return res;
    };
System.out.println(supplier.get());  //prints: Success

Predicate<Double> pred = num -> {
   System.out.println("Test if " + num + " is smaller than 20");
   return num < 20;
};
System.out.println(pred.test(10.0)? "10 is smaller":"10 is bigger");
                          //prints: Test if 10.0 is smaller than 20
                          //        10 is smaller
```

我们提供的专门功能接口示例如下：

```java
IntFunction<String> ifunc = new IntFunction<String>() {
    public String apply(int i) {
        return String.valueOf(i * 10);
    }
};
System.out.println(ifunc.apply(1));   //prints: 10
BiFunction<String, Integer, Double> bifunc =
        new BiFunction<String, Integer, Double >() {
            public Double apply(String s, Integer i) {
                return (s.length() * 10d) / i;
            }
        };

System.out.println(bifunc.apply("abc",2));     //prints: 15.0
BinaryOperator<Integer> binfunc = new BinaryOperator<Integer>(){
    public Integer apply(Integer i, Integer j) {
        return i >= j ? i : j;
    }
};
System.out.println(binfunc.apply(1,2));  //prints: 2
IntBinaryOperator intBiFunc = new IntBinaryOperator(){
    public int applyAsInt(int i, int j) {
        return i >= j ? i : j;
    }
};
System.out.println(intBiFunc.applyAsInt(1,2)); //prints: 2

```

以下是使用 lambda 表达式的样子：

```java
IntFunction<String> ifunc = i -> String.valueOf(i * 10);
System.out.println(ifunc.apply(1));             //prints: 10

BiFunction<String, Integer, Double> bifunc = 
                            (s,i) -> (s.length() * 10d) / i;
System.out.println(bifunc.apply("abc",2));      //prints: 15.0

BinaryOperator<Integer> binfunc = (i,j) -> i >= j ? i : j;
System.out.println(binfunc.apply(1,2));         //prints: 2

IntBinaryOperator intBiFunc = (i,j) -> i >= j ? i : j;
System.out.println(intBiFunc.applyAsInt(1,2));  //prints: 2
```

正如你所看到的，代码更简洁，更易读。

# 如何做...

那些有一些传统代码编写经验的人，在开始进行函数式编程时，将函数等同于方法。他们首先尝试创建函数，因为这是我们以前编写传统代码的方式——通过创建方法。然而，在函数式编程中，方法继续提供代码结构，而函数则是它的良好和有用的补充。因此，在函数式编程中，首先创建方法，然后再定义函数。让我们来演示一下。

以下是代码编写的基本步骤。首先，我们确定可以作为方法实现的精心设计的代码块。然后，在我们知道新方法将要做什么之后，我们可以将其功能的一些部分转换为函数：

1.  创建`calculate()`方法：

```java
void calculate(){
    int i = 42;        //get a number from some source
    double res = 42.0; //process the above number 
    if(res < 42){ //check the result using some criteria
        //do something
    } else {
        //do something else
    }
}
```

上述伪代码概述了`calculate()`方法的功能。它可以以传统方式实现——通过使用方法，如下所示：

```java
int getInput(){
   int result;
   //getting value for result variable here
   return result;
}
double process(int i){
    double result;
    //process input i and assign value to result variable
}
boolean checkResult(double res){
    boolean result = false;
    //use some criteria to validate res value
    //and assign value to result
    return result;
}
void processSuccess(double res){
     //do something with res value
}
void processFailure(double res){
     //do something else with res value
}
void calculate(){
    int i = getInput();
    double res = process(i); 
    if(checkResult(res)){     
        processSuccess(res);
    } else {
        processFailure(res);
    }
}
```

但是其中一些方法可能非常小，因此代码变得分散，使用这么多额外的间接会使代码变得不太可读。这个缺点在方法来自实现`calculate()`方法的类外部的情况下尤为明显：

```java
void calculate(){
    SomeClass1 sc1 = new SomeClass1();
    int i = sc1.getInput();
    SomeClass2 sc2 = new SomeClass2();
    double res = sc2.process(i); 
    SomeClass3 sc3 = new SomeClass3();
    SomeClass4 sc4 = new SomeClass4();
    if(sc3.checkResult(res)){     
        sc4.processSuccess(res);
    } else {
        sc4.processFailure(res);
    }
}
```

正如你所看到的，在每个外部方法都很小的情况下，管道代码的数量可能大大超过它所支持的负载。此外，上述实现在类之间创建了许多紧密的依赖关系。

1.  让我们看看如何使用函数来实现相同的功能。优势在于函数可以尽可能小，但是管道代码永远不会超过负载，因为没有管道代码。使用函数的另一个原因是，当我们需要灵活地在算法研究目的上更改功能的部分时。如果这些功能部分需要来自类外部，我们不需要为了将方法传递给`calculate()`而构建其他类。我们可以将它们作为函数传递：

```java
void calculate(Supplier<Integer> souc e, Function<Integer,
             Double> process, Predicate<Double> condition,
      Consumer<Double> success, Consumer<Double> failure){
    int i = source.get();
    double res = process.apply(i);
    if(condition.test(res)){
        success.accept(res);
    } else {
        failure.accept(res);
    }
} 
```

1.  以下是函数可能的样子：

```java
Supplier<Integer> source = () -> 4;
Function<Integer, Double> before = i -> i * 10.0;
Function<Double, Double> after = d -> d + 10.0;
Function<Integer, Double> process = before.andThen(after);
Predicate<Double> condition = num -> num < 100;
Consumer<Double> success = 
                  d -> System.out.println("Success: "+ d);
Consumer<Double> failure = 
                  d -> System.out.println("Failure: "+ d);
calculate(source, process, condition, success, failure);
```

上述代码的结果将如下：

```java
Success: 50.0
```

# 它是如何工作的...

Lambda 表达式就像一个普通的方法，除了当你考虑单独测试每个函数时。如何做呢？

有两种方法来解决这个问题。首先，由于函数通常很小，通常不需要单独测试它们，它们在使用它们的代码测试时间接测试。其次，如果您仍然认为函数必须进行测试，总是可以将其包装在返回函数的方法中，这样您就可以像测试其他方法一样测试该方法。以下是如何做的一个例子：

```java
public class Demo {
  Supplier<Integer> source(){ return () -> 4;}
  Function<Double, Double> after(){ return d -> d + 10.0; }
  Function<Integer, Double> before(){return i -> i * 10.0; }
  Function<Integer, Double> process(){return before().andThen(after());}
  Predicate<Double> condition(){ return num -> num < 100.; }
  Consumer<Double> success(){ 
     return d -> System.out.println("Failure: " + d); }
  Consumer<Double> failure(){ 
     return d-> System.out.println("Failure: " + d); }
  void calculate(Supplier<Integer> souce, Function<Integer,
              Double> process, Predicate<Double> condition,
       Consumer<Double> success, Consumer<Double> failure){
    int i = source.get();
    double res = process.apply(i);
    if(condition.test(res)){
        success.accept(res);
    } else {
        failure.accept(res);
    }
}
void someOtherMethod() {
   calculate(source(), process(), 
                       condition(), success(), failure());
}
```

现在我们可以编写函数单元测试如下：

```java
public class DemoTest {

    @Test
    public void source() {
        int i = new Demo().source().get();
        assertEquals(4, i);
    }
    @Test
    public void after() {
        double d = new Demo().after().apply(1.);
        assertEquals(11., d, 0.01);
    }
    @Test
    public void before() {
        double d = new Demo().before().apply(10);
        assertEquals(100., d, 0.01);
    }
    @Test
    public void process() {
        double d = new Demo().process().apply(1);
        assertEquals(20., d, 0.01);
    }
    @Test
    public void condition() {
        boolean b = new Demo().condition().test(10.);
        assertTrue(b);
    }
}
```

通常，lambda 表达式（以及一般的函数）用于为通用功能添加业务逻辑，从而实现特定功能。一个很好的例子是流操作，我们将在第五章《流和管道》中讨论。库的作者已经创建了它们以便能够并行工作，这需要很多专业知识。现在库的用户可以通过传递 lambda 表达式（函数）来专门定制操作，从而提供应用程序的业务逻辑。

# 还有更多...

由于，正如我们已经提到的，函数通常是简单的一行代码，当作为参数传递时通常会内联，例如：

```java
Consumer<Double> success = d -> System.out.println("Success: " + d);
Consumer<Double> failure = d-> System.out.println("Failure: " + d);
calculate(() -> 4, i -> i * 10.0 + 10, n -> n < 100, success, failure);
```

但是，不要过分推动，因为这样的内联可能会降低代码的可读性。

# 使用方法引用

在这个示例中，您将学习如何使用方法引用，构造函数引用是其中的一种情况。

# 准备工作

当一行 lambda 表达式只包含对其他地方实现的现有方法的引用时，可以进一步简化 lambda 表示法，使用方法引用。

方法引用的语法是`Location::methodName`，其中`Location`表示`methodName`方法所在的位置（对象或类）。两个冒号(`::`)作为位置和方法名之间的分隔符。如果在指定的位置有多个同名方法（因为方法重载），则引用方法由 lambda 表达式实现的函数接口的抽象方法的签名来确定。

# 如何做...

方法引用的确切格式取决于所引用的方法是静态的还是非静态的。方法引用也可以是*绑定的*或*未绑定的*，或者更正式地说，方法引用可以有*绑定的接收者*或*未绑定的接收者*。接收者是用于调用方法的对象或类。它*接收*调用。它可以绑定到特定的上下文或不绑定。我们将在演示过程中解释这意味着什么。

方法引用也可以引用带参数或不带参数的构造函数。

请注意，方法引用仅适用于表达式只包含一个方法调用而没有其他内容的情况。例如，方法引用可以应用于`() -> SomeClass.getCount()` lambda 表达式。它看起来像`SomeClass::getCount`。但是表达式`() -> 5 + SomeClass.getCount()`不能用方法引用替换，因为这个表达式中有比方法调用更多的操作。

# 静态未绑定方法引用

为了演示静态方法引用，我们将使用`Food`类和两个静态方法：

```java
class Food{
    public static String getFavorite(){ return "Donut!"; }
    public static String getFavorite(int num){
        return num > 1 ? String.valueOf(num) + " donuts!" : "Donut!";
    }
}
```

由于第一个方法`String getFavorite()`不接受任何输入参数并返回一个值，它可以作为一个函数接口`Supplier<T>`来实现。实现调用`String getFavorite()`静态方法的 lambda 表达式如下：

```java
Supplier<String> supplier = () -> Food.getFavorite();
```

使用方法引用，前面的行变成了以下内容：

```java
Supplier<String> supplier = Food::getFavorite;
```

正如您所看到的，前面的格式定义了方法的位置（作为`Food`类），方法的名称和返回类型的值（作为`String`）。函数接口的名称表示没有输入参数，因此编译器和 JVM 可以在`Food`类的方法中识别该方法。

静态方法引用是未绑定的，因为没有对象用于调用该方法。在静态方法的情况下，类是调用接收器，而不是对象。

第二个静态方法`String getFavorite(int num)`接受一个参数并返回一个值。这意味着我们可以使用`Function<T,R>`函数接口来实现仅调用此方法的函数：

```java
Function<Integer, String> func = i -> Food.getFavorite(i); 
```

但是当使用方法引用时，它会变成与前面示例完全相同的形式：

```java
Function<Integer, String> func = Food::getFavorite; 
```

区别在于指定的函数接口。它允许编译器和 Java 运行时识别要使用的方法：方法名为`getFavorite()`，接受`Integer`值，并返回`String`值。`Food`类中只有一个这样的方法。实际上，甚至不需要查看方法返回的值，因为仅通过返回值无法重载方法。方法的签名——名称和参数类型列表——足以标识方法。

我们可以按以下方式使用实现的函数：

```java
Supplier<String> supplier = Food::getFavorite;
System.out.println("supplier.get() => " + supplier.get());

Function<Integer, String> func = Food::getFavorite;
System.out.println("func.getFavorite(1) => " + func.apply(1));
System.out.println("func.getFavorite(2) => " + func.apply(2));
```

如果运行上述代码，结果将如下所示：

![](img/b59f41b3-3172-45ff-b858-7967323967de.png)

# 非静态绑定方法引用

为了演示非静态绑定方法引用，让我们通过添加`name`字段、两个构造函数和两个`String sayFavorite()`方法来增强`Food`类：

```java
class Food{
     private String name;
     public Food(){ this.name = "Donut"; }
     public Food(String name){ this.name = name; }
     public static String getFavorite(){ return "Donut!"; }
     public static String getFavorite(int num){
         return num > 1 ? String.valueOf(num) + " donuts!" : "Donut!";
     }
     public String sayFavorite(){
         return this.name + (this.name.toLowerCase()
                             .contains("donut")?"? Yes!" : "? D'oh!");
     }
     public String sayFavorite(String name){
         this.name = this.name + " and " + name;
         return sayFavorite();
     }
}
```

现在，让我们创建`Food`类的三个实例：

```java
Food food1 = new Food();
Food food2 = new Food("Carrot");
Food food3 = new Food("Carrot and Broccoli");
```

上述是上下文——我们将要创建的 lambda 表达式周围的代码。我们使用前面上下文的局部变量来实现三个不同的供应商：

```java
Supplier<String> supplier1 = () -> food1.sayFavorite();
Supplier<String> supplier2 = () -> food2.sayFavorite();
Supplier<String> supplier3 = () -> food3.sayFavorite();
```

我们使用`Supplier<T>`，因为`String sayFavorite()`方法不需要任何参数，只产生（提供）`String`值。使用方法引用，我们可以将前面的 lambda 表达式重写如下：

```java
Supplier<String> supplier1 = food1::sayFavorite;
Supplier<String> supplier2 = food2::sayFavorite;
Supplier<String> supplier3 = food3::sayFavorite;
```

方法`sayFavorite()`属于在特定上下文中创建的对象。换句话说，这个对象（调用接收器）绑定到特定的上下文，这就是为什么这样的方法引用被称为*绑定方法引用*或*绑定接收器方法引用*。

我们可以将新创建的函数作为任何其他对象传递，并在需要的任何地方使用它们，例如：

```java
System.out.println("new Food().sayFavorite() => " + supplier1.get());
System.out.println("new Food(Carrot).sayFavorite() => " 
                                                  + supplier2.get());
System.out.println("new Food(Carrot,Broccoli).sayFavorite() => " 
                                                  + supplier3.get());
```

结果将如下所示：

![](img/8cf07e4c-f18c-4df7-8b95-20f98446228f.png)

请注意，接收器仍然绑定到上下文，因此其状态可能会改变并影响输出。这就是*绑定*的区别的重要性。使用这样的引用时，必须小心不要在其原始上下文中更改接收器的状态。否则，可能会导致不可预测的结果。在并行处理时，同一函数可以在不同的上下文中使用，这一考虑尤为重要。

让我们看看使用第二个非静态方法`String sayFavorite(String name)`的绑定方法引用的另一个案例。首先，我们使用相同的`Food`类的对象创建了一个`UnaryOperator<T>`函数接口的实现，这与前面的示例中使用的相同：

```java
UnaryOperator<String> op1 = s -> food1.sayFavorite(s);
UnaryOperator<String> op2 = s -> food2.sayFavorite(s);
UnaryOperator<String> op3 = s -> food3.sayFavorite(s);

```

我们使用`UnaryOperator<T>`函数接口的原因是，`String sayFavorite(String name)`方法接受一个参数并产生相同类型的值。这就是它们名称中带有`Operator`的函数接口的目的——支持输入值和结果类型相同的情况。

方法引用允许我们将 lambda 表达式更改如下：

```java
UnaryOperator<String> op1 = food1::sayFavorite;
UnaryOperator<String> op2 = food2::sayFavorite;
UnaryOperator<String> op3 = food3::sayFavorite;
```

现在我们可以在代码的任何地方使用前面的函数（操作符），例如：

```java
System.out.println("new Food()
       .sayFavorite(Carrot) => " + op1.apply("Carrot"));
System.out.println("new Food(Carrot)
   .sayFavorite(Broccoli) => " + op2.apply("Broccoli"));
System.out.println("new Food(Carrot, Broccoli)
       .sayFavorite(Donuts) => " + op3.apply("Donuts"));
```

上述代码的结果如下：

![](img/f873fc74-fbd0-4b23-bde2-85d3643d2469.png)

# 非静态未绑定方法引用

为了演示对`String sayFavorite()`方法的非绑定方法引用，我们将使用`Function<T,R>`函数接口，因为我们希望使用`Food`类的对象（调用接收器）作为参数，并返回一个`String`值：

```java
Function<Food, String> func = f -> f.sayFavorite();

```

方法引用允许我们将前面的 lambda 表达式重写为以下形式：

```java
Function<Food, String> func = Food::sayFavorite;
```

使用在前面的例子中创建的`Food`类的相同对象，我们在以下代码中使用新创建的函数，例如：

```java
System.out.println("new Food()
              .sayFavorite() => " + func.apply(food1));
System.out.println("new Food(Carrot)
              .sayFavorite() => " + func.apply(food2));
System.out.println("new Food(Carrot, Broccoli)
              .sayFavorite() => " + func.apply(food3));
```

正如您所看到的，参数（调用接收对象）仅来自当前上下文，就像任何参数一样。无论函数传递到哪里，它都不携带上下文。它的接收器不绑定到用于函数创建的上下文。这就是为什么这个方法引用被称为*未绑定*的原因。

前面代码的输出如下：

![](img/3dda0849-946f-42dd-a499-97cd5b6628aa.png)

为了演示未绑定方法引用的另一个案例，我们将使用第二个方法`String sayFavorite(String name)`，并使用一直使用的`Food`对象。我们要实现的功能接口这次叫做`BiFunction<T,U,R>`：

```java
BiFunction<Food, String, String> func = (f,s) -> f.sayFavorite(s);
```

我们选择这个功能接口的原因是它接受两个参数——这正是我们在这种情况下需要的——以便将接收对象和`String`值作为参数。前面 lambda 表达式的方法引用版本如下所示：

```java
BiFunction<Food, String, String> func = Food::sayFavorite;

```

我们可以通过编写以下代码来使用前面的函数，例如：

```java
System.out.println("new Food()
  .sayFavorite(Carrot) => " + func.apply(food1, "Carrot"));
System.out.println("new Food(Carrot)
  .sayFavorite(Broccoli) => " 
                         + func2.apply(food2, "Broccoli"));
System.out.println("new Food(Carrot,Broccoli)
  .sayFavorite(Donuts) => " + func2.apply(food3,"Donuts"));

```

结果如下：

![](img/ce301b7b-80bd-49e0-afb6-7b4b0e72e77e.png)

# 构造函数方法引用

使用构造函数的方法引用与静态方法引用非常相似，因为它使用类作为调用接收器，而不是对象（它尚未被创建）。这是实现`Supplier<T>`接口的 lambda 表达式：

```java
Supplier<Food> foodSupplier = () -> new Food();

```

以下是它的方法引用版本：

```java
Supplier<Food> foodSupplier = Food::new;
System.out.println("new Food()
  .sayFavorite() => " + foodSupplier.get().sayFavorite());
```

如果我们运行前面的代码，我们会得到以下输出：

![](img/34b6cfe4-9407-41fb-81a5-f23f2f307e52.png)

现在，让我们向`Food`类添加另一个构造函数：

```java
public Food(String name){ 
     this.name = name; 
} 
```

一旦我们这样做，我们可以通过方法引用来表示前面的构造函数：

```java
Function<String, Food> createFood = Food::new;
Food food = createFood.apply("Donuts");
System.out.println("new Food(Donuts).sayFavorite() => " 
                                   + food.sayFavorite());
food = createFood.apply("Carrot");
System.out.println("new Food(Carrot).sayFavorite() => " 
                                   + food.sayFavorite());
```

以下是前面代码的输出：

![](img/882157fe-d4e8-4439-ba19-cc505020fbf8.png)

同样地，我们可以添加一个带有两个参数的构造函数：

```java
public Food(String name, String anotherName) {
     this.name = name + " and " + anotherName;
}
```

一旦我们这样做，我们可以通过`BiFunction<String, String>`来表示它：

```java
BiFunction<String, String, Food> createFood = Food::new;
Food food = createFood.apply("Donuts", "Carrots");
System.out.println("new Food(Donuts, Carrot)
        .sayFavorite() => " + food.sayFavorite());
food = constrFood2.apply("Carrot", "Broccoli");
System.out.println("new Food(Carrot, Broccoli)
          .sayFavorite() => " food.sayFavorite());
```

前面的代码的结果如下：

![](img/981b14a2-265d-4fe6-a524-5785d675f49a.png)

为了表示接受多于两个参数的构造函数，我们可以创建一个自定义的功能接口，带有任意数量的参数。例如，我们可以使用前面一篇文章中讨论的以下自定义功能接口：

```java
        @FunctionalInterface
        interface Func<T1,T2,T3,R>{ R apply(T1 t1, T2 t2, T3 t3);}
```

假设我们需要使用`AClass`类：

```java
class AClass{
    public AClass(int i, double d, String s){ }
    public String get(int i, double d){ return ""; }
    public String get(int i, double d, String s){ return ""; }
}
```

我们可以通过使用方法引用来编写以下代码：

```java
Func<Integer, Double, String, AClass> func1 = AClass::new;
AClass obj = func1.apply(1, 2d, "abc");

Func<Integer, Double, String, String> func2 = obj::get;    //bound
String res1 = func2.apply(42, 42., "42");

Func<AClass, Integer, Double, String> func3 = AClass::get; //unbound
String res21 = func3.apply(obj, 42, 42.);

func1 function that allows us to create an object of class AClass. The func2 function applies to the resulting object obj the method String get(int i, double d) using the bound method reference because its call receiver (object obj) comes from a particular context (bound to it). By contrast, the func3 function is implemented as an unbound method reference because it gets its call receiver (class AClass) not from a context. 
```

# 还有更多...

有几个简单但非常有用的方法引用，因为它得到了通常在实践中使用的调用接收器：

```java
Function<String, Integer> strLength = String::length;
System.out.println(strLength.apply("3"));  //prints: 1

Function<String, Integer> parseInt = Integer::parseInt;
System.out.println(parseInt.apply("3"));    //prints: 3

Consumer<String> consumer = System.out::println;
consumer.accept("Hello!");             //prints: Hello!
```

还有一些用于处理数组和列表的有用方法：

```java
Function<Integer, String[]> createArray = String[]::new;
String[] arr = createArray.apply(3);
System.out.println("Array length=" + arr.length); 

int i = 0;
for(String s: arr){ arr[i++] = String.valueOf(i); }
Function<String[], List<String>> toList = Arrays::<String>asList;
List<String> l = toList.apply(arr);
System.out.println("List size=" + l.size());
for(String s: l){ System.out.println(s); }
```

以下是前面代码的结果：

![](img/dda4ac9d-511a-44dd-918d-7d48e26f2c87.png)

让我们由你来分析前面的 lambda 表达式是如何创建和使用的。

# 利用 lambda 表达式在您的程序中

在这个示例中，您将学习如何将 lambda 表达式应用到您的代码中。我们将回到演示应用程序，并通过引入 lambda 表达式来修改它。

# 准备工作

配备了功能接口、lambda 表达式和友好的 lambda API 设计最佳实践，我们可以通过使其设计更加灵活和用户友好来大大改进我们的速度计算应用程序。让我们尽可能接近真实问题的背景，而不要使它过于复杂。

无人驾驶汽车如今成为新闻头条，有充分的理由相信它将在相当长的一段时间内保持这种状态。在这个领域的任务之一是基于真实数据对城市区域的交通流进行分析和建模。这样的数据已经存在很多，并且将继续在未来被收集。假设我们可以通过日期、时间和地理位置访问这样的数据库。还假设来自该数据库的交通数据以单位形式存在，每个单位捕捉有关一个车辆和驾驶条件的详细信息：

```java
public interface TrafficUnit {
  VehicleType getVehicleType();
  int getHorsePower();
  int getWeightPounds();
  int getPayloadPounds();
  int getPassengersCount();
  double getSpeedLimitMph();
  double getTraction();
  RoadCondition getRoadCondition();
  TireCondition getTireCondition();
  int getTemperature();
} 
```

`enum`类型——`VehicleType`、`RoadCondition`和`TireCondition`——已经在第二章中构建，*OOP 快速通道-类和接口*：

```java
enum VehicleType { 
  CAR("Car"), TRUCK("Truck"), CAB_CREW("CabCrew");
  private String type;
  VehicleType(String type){ this.type = type; }
  public String getType(){ return this.type;}
}
enum RoadCondition {
  DRY(1.0), 
  WET(0.2) { public double getTraction() { 
    return temperature > 60 ? 0.4 : 0.2; } }, 
  SNOW(0.04);
  public static int temperature;
  private double traction;
  RoadCondition(double traction){ this.traction = traction; }
  public double getTraction(){return this.traction;}
}
enum TireCondition {
  NEW(1.0), WORN(0.2);
  private double traction;
  TireCondition(double traction){ this.traction = traction; }
  public double getTraction(){ return this.traction;}
}

```

访问交通数据的接口可能如下所示：

```java
TrafficUnit getOneUnit(Month month, DayOfWeek dayOfWeek, 
                       int hour, String country, String city, 
                       String trafficLight);
List<TrafficUnit> generateTraffic(int trafficUnitsNumber, 
                  Month month, DayOfWeek dayOfWeek, int hour,
                  String country, String city, String trafficLight);
```

以下是访问前述方法的示例：

```java
TrafficUnit trafficUnit = FactoryTraffic.getOneUnit(Month.APRIL, 
               DayOfWeek.FRIDAY, 17, "USA", "Denver", "Main103S");
```

数字`17`是一天中的小时（下午 5 点），`Main1035`是交通灯的标识。

对第二个方法的调用返回多个结果：

```java
List<TrafficUnit> trafficUnits = 
    FactoryTrafficModel.generateTraffic(20, Month.APRIL, 
        DayOfWeek.FRIDAY, 17, "USA", "Denver", "Main103S");
```

第一个参数`20`是请求的交通单位数。

如您所见，这样的交通工厂提供了关于特定时间（在我们的示例中为下午 5 点至 6 点之间）特定地点的交通数据。每次调用工厂都会产生不同的结果，而交通单位列表描述了统计上正确的数据（包括指定位置的最可能天气条件）。

我们还将更改`FactoryVehicle`和`FactorySpeedModel`的接口，以便它们可以基于`TrafficUnit`接口构建`Vehicle`和`SpeedModel`。结果演示代码如下：

```java
double timeSec = 10.0;
TrafficUnit trafficUnit = FactoryTraffic.getOneUnit(Month.APRIL, 
              DayOfWeek.FRIDAY, 17, "USA", "Denver", "Main103S");
Vehicle vehicle = FactoryVehicle.build(trafficUnit);
SpeedModel speedModel =  
               FactorySpeedModel.generateSpeedModel(trafficUnit);
vehicle.setSpeedModel(speedModel);
printResult(trafficUnit, timeSec, vehicle.getSpeedMph(timeSec));
```

`printResult()`方法包含以下代码：

```java
void printResult(TrafficUnit tu, double timeSec, double speedMph){
   System.out.println("Road " + tu.getRoadCondition()
                 + ", tires " + tu.getTireCondition() + ": " 
                              + tu.getVehicleType().getType() 
                              + " speedMph (" + timeSec + " sec)=" 
                                              + speedMph + " mph");
}
```

此代码的输出可能如下所示：

![](img/51e23141-3a9a-4765-9790-3e73337b7c89.png)

由于我们现在使用“真实”数据，因此该程序的每次运行都会产生不同的结果，这取决于数据的统计特性。在某个地点，汽车或干燥天气可能更常出现在该日期和时间，而在另一个地点，卡车或雪更典型。

在这次运行中，交通单位带来了湿地面、新轮胎和`Truck`，具有这样的发动机功率和负载，以至于在 10 秒内能够达到 22 英里/小时的速度。我们用来计算速度的公式（在`SpeedModel`对象内部）对您来说是熟悉的：

```java
double weightPower = 2.0 * horsePower * 746 * 32.174 / weightPounds;
double speed = (double) Math.round(Math.sqrt(timeSec * weightPower) 
                                                 * 0.68 * traction);
```

这里，`traction`值来自`TrafficUnit`。在实现`TrafficUnit`接口的类中，`getTraction()`方法如下所示：

```java
public double getTraction() {
  double rt = getRoadCondition().getTraction();
  double tt = getTireCondition().getTraction();
  return rt * tt;
}
```

`getRoadCondition()`和`getTireCondition()`方法返回我们刚刚描述的相应`enum`类型的元素。

现在我们准备使用前面讨论的 lambda 表达式来改进我们的速度计算应用程序。

# 如何做…

按照以下步骤学习如何使用 lambda 表达式：

1.  让我们开始构建一个 API。我们将其称为`Traffic`。如果不使用函数接口，它可能如下所示：

```java
public interface Traffic {
   void speedAfterStart(double timeSec, int trafficUnitsNumber);
}  
```

其实现可能如下所示：

```java
public class TrafficImpl implements Traffic {
   private int hour;
   private Month month;
   private DayOfWeek dayOfWeek;
   private String country, city, trafficLight;
   public TrafficImpl(Month month, DayOfWeek dayOfWeek, int hour, 
                String country, String city, String trafficLight){
      this.hour = hour;
      this.city = city;
      this.month = month;
      this.country = country;
      this.dayOfWeek = dayOfWeek;
      this.trafficLight = trafficLight;
   }
   public void speedAfterStart(double timeSec, 
                                      int trafficUnitsNumber){
      List<TrafficUnit> trafficUnits = 
        FactoryTraffic.generateTraffic(trafficUnitsNumber, 
          month, dayOfWeek, hour, country, city, trafficLight);
      for(TrafficUnit tu: trafficUnits){
         Vehicle vehicle = FactoryVehicle.build(tu);
         SpeedModel speedModel = 
                      FactorySpeedModel.generateSpeedModel(tu);
         vehicle.setSpeedModel(speedModel);
         double speed = vehicle.getSpeedMph(timeSec);
         printResult(tu, timeSec, speed);
      }
   }
}
```

1.  让我们编写使用`Traffic`接口的示例代码：

```java
Traffic traffic = new TrafficImpl(Month.APRIL, 
  DayOfWeek.FRIDAY, 17, "USA", "Denver", "Main103S");
double timeSec = 10.0;
int trafficUnitsNumber = 10;
traffic.speedAfterStart(timeSec, trafficUnitsNumber); 
```

我们得到类似以下的结果：

![](img/038d044d-970e-4fa4-bb1f-cea021a97788.png)

如前所述，由于我们使用真实数据，因此相同的代码不会每次产生完全相同的结果。不应该期望看到前面截图中的速度值，而是看到非常相似的结果。

1.  让我们使用 lambda 表达式。前面的 API 相当有限。例如，它不允许您在不更改`FactorySpeedModel`的情况下测试不同的速度计算公式。同时，`SpeedModel`接口只有一个名为`getSpeedMph()`的抽象方法（这使它成为函数接口）：

```java
public interface SpeedModel {
  double getSpeedMph(double timeSec, 
           int weightPounds, int horsePower);
}
```

我们可以利用`SpeedModel`是函数接口的特性，并向`Traffic`接口添加另一个方法，该方法能够接受`SpeedModel`实现作为 lambda 表达式：

```java
public interface Traffic {
  void speedAfterStart(double timeSec, 
                       int trafficUnitsNumber);
  void speedAfterStart(double timeSec, 
    int trafficUnitsNumber, SpeedModel speedModel);
}
```

不过问题在于`traction`值不作为`getSpeedMph()`方法的参数传递，因此我们无法将其作为一个函数传递到`speedAfterStart()`方法中。仔细查看`FactorySpeedModel.generateSpeedModel(TrafficUnit trafficUnit)`的速度计算：

```java
double getSpeedMph(double timeSec, int weightPounds, 
                                           int horsePower) {
    double traction = trafficUnit.getTraction();
    double v = 2.0 * horsePower * 746 * timeSec * 
                                    32.174 / weightPounds;
    return Math.round(Math.sqrt(v) * 0.68 * traction);
}
```

正如你所看到的，`traction`值是计算出的`speed`值的乘数，这是对交通单位的唯一依赖。我们可以从速度模型中移除`traction`，并在使用速度模型计算速度后应用`traction`。这意味着我们可以改变`TrafficImpl`类的`speedAfterStart()`的实现，如下所示：

```java
public void speedAfterStart(double timeSec, 
           int trafficUnitsNumber, SpeedModel speedModel) {
   List<TrafficUnit> trafficUnits = 
     FactoryTraffic.generateTraffic(trafficUnitsNumber, 
       month, dayOfWeek, hour, country, city, trafficLight);
   for(TrafficUnit tu: trafficUnits){
       Vehicle vehicle = FactoryVehicle.build(tu);
       vehicle.setSpeedModel(speedModel);
       double speed = vehicle.getSpeedMph(timeSec);
       speed = (double) Math.round(speed * tu.getTraction());
       printResult(tu, timeSec, speed);
   }
}
```

这个改变允许`Traffic` API 的用户将`SpeedModel`作为一个函数传递：

```java
Traffic traffic = new TrafficImpl(Month.APRIL, 
     DayOfWeek.FRIDAY, 17, "USA", "Denver", "Main103S");
double timeSec = 10.0;
int trafficUnitsNumber = 10;
SpeedModel speedModel = (t, wp, hp) -> {
   double weightPower = 2.0 * hp * 746 * 32.174 / wp;
   return (double) Math
              .round(Math.sqrt(t * weightPower) * 0.68);
};
traffic.speedAfterStart(timeSec, trafficUnitsNumber, 
                                            speedModel);
```

1.  上述代码的结果与通过`FactorySpeedModel`生成`SpeedModel`时相同。但现在 API 用户可以自己想出自己的速度计算函数。

1.  我们可以将`SpeedModel`接口注释为`@FunctionalInterface`，这样所有试图向其添加另一个方法的人都会得到警告，并且不能在删除此注释并意识到破坏已经实现此功能接口的现有客户端代码的风险的情况下添加另一个抽象方法。

1.  我们可以通过添加各种标准来丰富 API，将所有可能的交通划分为不同的片段。

例如，API 用户可能只想分析汽车、卡车、引擎大于 300 马力的汽车，或引擎大于 400 马力的卡车。传统的方法是创建这样的方法：

```java
void speedAfterStartCarEngine(double timeSec, 
              int trafficUnitsNumber, int horsePower);
void speedAfterStartCarTruckOnly(double timeSec, 
                              int trafficUnitsNumber);
void speedAfterStartEngine(double timeSec, 
         int trafficUnitsNumber, int carHorsePower, 
                                 int truckHorsePower);
```

相反，我们可以将标准的函数接口添加到`Traffic`接口的现有`speedAfterStart()`方法中，并让 API 用户决定要提取哪一部分交通：

```java
void speedAfterStart(double timeSec, int trafficUnitsNumber,
  SpeedModel speedModel, Predicate<TrafficUnit> limitTraffic);
```

`TrafficImpl`类中`speedAfterStart()`方法的实现将如下更改：

```java
public void speedAfterStart(double timeSec, 
          int trafficUnitsNumber, SpeedModel speedModel, 
                    Predicate<TrafficUnit> limitTraffic) {
  List<TrafficUnit> trafficUnits = 
    FactoryTraffic.generateTraffic(trafficUnitsNumber, 
    month, dayOfWeek, hour, country, city, trafficLight);
  for(TrafficUnit tu: trafficUnits){
      if(limitTraffic.test(tu){
         Vehicle vehicle = FactoryVehicle.build(tu);
         vehicle.setSpeedModel(speedModel);
         double speed = vehicle.getSpeedMph(timeSec);
         speed = (double) Math.round(speed * 
                                   tu.getTraction());
         printResult(tu, timeSec, speed);
      }
    }
}
```

然后，`Traffic` API 用户可以按以下方式定义他们需要的交通情况：

```java
Predicate<TrafficUnit> limit = tu ->
  (tu.getHorsePower() < 250 
      && tu.getVehicleType() == VehicleType.CAR) || 
  (tu.getHorsePower() < 400 
      && tu.getVehicleType() == VehicleType.TRUCK);
traffic.speedAfterStart(timeSec, 
            trafficUnitsNumber, speedModel, limit);
```

结果现在被限制为引擎小于 250 `hp`的汽车和引擎小于 400 `hp`的卡车：

![](img/d601ea9c-2be1-4f90-971f-e6ecb0259f05.png)

事实上，`Traffic` API 用户现在可以应用任何限制交通的标准，只要它们适用于`TrafficUnit`对象中的值。例如，用户可以写以下内容：

```java
Predicate<TrafficUnit> limitTraffic = 
 tu -> tu.getTemperature() > 65 
 && tu.getTireCondition() == TireCondition.NEW 
 && tu.getRoadCondition() == RoadCondition.WET;
```

或者，他们可以写任何其他限制`TrafficUnit`值的组合。如果用户决定移除限制并分析所有交通情况，这段代码也可以做到：

```java
traffic.speedAfterStart(timeSec, trafficUnitsNumber, 
                              speedModel, tu -> true);
```

1.  如果需要通过速度选择交通单位，我们可以在速度计算后应用谓词标准（请注意，我们用`BiPredicate`替换了`Predicate`，因为我们现在需要使用两个参数）：

```java
public void speedAfterStart(double timeSec,  
           int trafficUnitsNumber, SpeedModel speedModel,
             BiPredicate<TrafficUnit, Double> limitSpeed){
   List<TrafficUnit> trafficUnits = 
     FactoryTraffic.generateTraffic(trafficUnitsNumber, 
     month, dayOfWeek, hour, country, city, trafficLight);
   for(TrafficUnit tu: trafficUnits){
      Vehicle vehicle = FactoryVehicle.build(tu);
      vehicle.setSpeedModel(speedModel);
      double speed = vehicle.getSpeedMph(timeSec);
      speed = (double) Math.round(speed*tu.getTraction());
      if(limitSpeed.test(tu, speed)){
           printResult(tu, timeSec, speed);
      }
   }
}
```

`Traffic` API 用户现在可以编写以下代码：

```java
BiPredicate<TrafficUnit, Double> limit = (tu, sp) ->
   (sp > (tu.getSpeedLimitMph() + 8.0) && 
          tu.getRoadCondition() == RoadCondition.DRY) || 
   (sp > (tu.getSpeedLimitMph() + 5.0) && 
          tu.getRoadCondition() == RoadCondition.WET) || 
    (sp > (tu.getSpeedLimitMph() + 0.0) && 
           tu.getRoadCondition() == RoadCondition.SNOW);
traffic.speedAfterStart(timeSec, 
                 trafficUnitsNumber, speedModel, limit);
```

上面的谓词选择超过一定数量的速度限制的交通单位（对于不同的驾驶条件是不同的）。如果需要，它可以完全忽略速度，并且以与之前的谓词完全相同的方式限制交通。这种实现的唯一缺点是它略微不那么高效，因为谓词是在速度计算之后应用的。这意味着速度计算将针对每个生成的交通单位进行，而不是像之前的实现中那样限制数量。如果这是一个问题，你可以留下我们在本文中讨论过的所有不同签名：

```java
public interface Traffic {
   void speedAfterStart(double timeSec, int trafficUnitsNumber);
   void speedAfterStart(double timeSec, int trafficUnitsNumber,
                                         SpeedModel speedModel);
   void speedAfterStart(double timeSec, 
            int trafficUnitsNumber, SpeedModel speedModel, 
                           Predicate<TrafficUnit> limitTraffic);
   void speedAfterStart(double timeSec, 
             int trafficUnitsNumber, SpeedModel speedModel,
                  BiPredicate<TrafficUnit,Double> limitTraffic);
}
```

这样，API 用户可以决定使用哪种方法，更灵活或更高效，并决定默认的速度计算实现是否可接受。

# 还有更多...

到目前为止，我们还没有给 API 用户选择输出格式的选择。目前，它是作为`printResult()`方法实现的：

```java
void printResult(TrafficUnit tu, double timeSec, double speedMph) {
  System.out.println("Road " + tu.getRoadCondition() +
                  ", tires " + tu.getTireCondition() + ": " 
                     + tu.getVehicleType().getType() + " speedMph (" 
                     + timeSec + " sec)=" + speedMph + " mph");
}
```

为了使其更加灵活，我们可以向我们的 API 添加另一个参数：

```java
Traffic traffic = new TrafficImpl(Month.APRIL, DayOfWeek.FRIDAY, 17,
                                        "USA", "Denver", "Main103S");
double timeSec = 10.0;
int trafficUnitsNumber = 10;
BiConsumer<TrafficUnit, Double> output = (tu, sp) ->
  System.out.println("Road " + tu.getRoadCondition() + 
                  ", tires " + tu.getTireCondition() + ": " 
                     + tu.getVehicleType().getType() + " speedMph (" 
                     + timeSec + " sec)=" + sp + " mph");
traffic.speedAfterStart(timeSec, trafficUnitsNumber, speedModel, output);
```

注意我们取`timeSec`值不是作为函数参数之一，而是从函数的封闭范围中取得。我们之所以能够这样做，是因为它在整个计算过程中保持不变（并且可以被视为最终值）。同样地，我们可以向`output`函数添加任何其他对象，比如文件名或另一个输出设备，从而将所有与输出相关的决策留给 API 用户。为了适应这个新函数，API 的实现发生了变化，如下所示：

```java
public void speedAfterStart(double timeSec, int trafficUnitsNumber,
        SpeedModel speedModel, BiConsumer<TrafficUnit, Double> output) {
  List<TrafficUnit> trafficUnits = 
     FactoryTraffic.generateTraffic(trafficUnitsNumber, month, 
                      dayOfWeek, hour, country, city, trafficLight);
  for(TrafficUnit tu: trafficUnits){
     Vehicle vehicle = FactoryVehicle.build(tu);
     vehicle.setSpeedModel(speedModel);
     double speed = vehicle.getSpeedMph(timeSec);
     speed = (double) Math.round(speed * tu.getTraction());
     output.accept(tu, speed);
  }
}
```

我们花了一些时间才达到这一点——函数式编程的威力开始显现并证明了学习它的努力是值得的。然而，当用于处理流时，如下一章所述，lambda 表达式会产生更大的威力。
