# 第七章：包和可访问性（可见性）

到目前为止，您已经非常熟悉包了。在本章中，我们将完成其描述，然后讨论类和类成员（方法和字段）的不同访问级别（也称为可见性）。这将涉及到面向对象编程的关键概念——封装，并为我们讨论面向对象设计原则奠定基础。

在本章中，我们将涵盖以下主题：

+   什么是导入?

+   静态导入

+   接口访问修饰符

+   类访问修饰符

+   方法访问修饰符

+   属性访问修饰符

+   封装

+   练习-阴影跟读

# 什么是导入?

导入允许我们在`.java`文件的开始（类或接口声明之前）只指定一次完全限定的类或接口名称。导入语句的格式如下：

```java
import <package>.<class or interface name>;
```

例如，看下面的：

```java
import com.packt.javapath.ch04demo.MyApplication;
```

从现在开始，这个类只能通过它的名称`MyApplication`在代码中引用。也可以使用通配符（`*`）导入包的所有类或接口:

```java
import com.packt.javapath.ch04demo.*;
```

注意，前面的导入语句导入了`com.packt.javapath.ch04demo`包的子包的类和接口。如果需要，必须逐个导入每个子包。

但在继续之前，让我们谈谈`.java`文件结构和包。

# `.java`文件和包的结构

正如您所知道的，包名反映了目录结构，从包含`.java`文件的项目目录开始。每个`.java`文件的名称必须与其中定义的公共类的名称相同。`.java`文件的第一行是以`package`关键字开头的包声明，其后是实际的包名称——本文件的目录路径，其中斜线替换为句点。让我们看一些例子。我们主要关注包含类定义的`.java`文件，但我们也会看一些带有接口和`enum`类定义的文件，因为特殊的导入类型（称为静态导入）主要用于接口和`enum`。

我们假设`src/main/java`（对于 Linux）或`src\main\java`（对于 Windows）项目目录包含所有`.java`文件，并且定义在`com.packt.javapath`包的`MyClass`和`MyEnum`类和`MyInterface`接口的定义存储在文件中：

```java
src/main/java/com/packt/javapath/MyClass.java (for Linux) 
src/main/java/com/packt/javapath/MyEnum.java
src/main/java/com/packt/javapath/MyInterface.java 
```

或（对于 Windows）

```java
src\main\java\com\packt\javapath\MyClass.java (for Windows) 
src\main\java\com\packt\javapath\MyEnum.java
src\main\java\com\packt\javapath\MyInterface.java 
```

这些文件的第一行如下所示：

```java
package com.packt.javapath;
```

如果我们什么都不导入，则每个文件的下一行是一个类或接口声明。

`MyClass`类的声明如下：

```java
public class MyClass extends SomeClass 
     implements Interface1, Interface2, ... {...}
```

它包括以下内容：

+   访问修饰符；该文件中的其中一个类必须是`public`

+   `class`关键字

+   类名（按约定以大写字母开头的标识符）

+   如果类是另一个类的子类，则有`extends`关键字和父类的名称

+   如果类实现了一个或多个接口，则有`implements`关键字，后跟它实现的接口的逗号分隔列表

+   类的主体（其中定义了字段和方法）用大括号`{}`括起来

`MyEnum`类的声明如下所示：

```java
public enum MyEnum implements Interface1, Interface2, ... {...}
```

它包括以下内容：

+   访问修饰符；如果它是文件中定义的唯一类，则必须是`public`

+   `enum`关键字

+   类名（标识符），按约定以大写字母开头

+   没有`extends`关键字，因为枚举类型隐式地扩展了`java.lang.Enum`类，在 Java 中，一个类只能有一个父类

+   如果类实现了一个或多个接口，则有`implements`关键字，后跟它实现的接口的逗号分隔列表

+   类的主体（其中定义了常量和方法）用大括号`{}`括起来

`MyInterface`接口的声明如下所示：

```java
public interface MyInterface extends Interface1, Interface2, ... {...}
```

它包括以下内容：

+   访问修饰符；文件中的一个接口必须是`public`

+   `interface`关键字

+   接口名称（标识符），按约定以大写字母开头

+   如果接口是一个或多个接口的子接口，则接口后跟`extends`关键字，后跟父接口的逗号分隔列表

+   接口的主体（其中定义了字段和方法）用大括号`{}`括起来

如果没有导入，我们需要通过其完全限定名来引用我们正在使用的每个类或接口，其中包括包名和类或接口名。例如，`MyClass`类的声明将如下所示：

```java
public class MyClass 
          extends com.packt.javapath.something.AnotherMyClass 
          implements com.packt.javapath.something2.Interface1,
                     com.packt.javapath.something3.Interface2
```

或者，假设我们想要实例化`com.packt.javapath.something`包中的`SomeClass`类。该类的完全限定名称将是`com.packt.javapath.something.SomeClass`，其对象创建语句将如下所示：

```java
com.packt.javapath.something.SomeClass someClass =
                    new com.packt.javapath.something.SomeClass();
```

这太冗长了，不是吗？这就是包导入发挥作用的地方。

# 单个类导入

为了避免在代码中使用完全限定的类或接口名称，我们可以在包声明和类或接口声明之间的空间中添加一个导入语句：

```java
package com.packt.javapath;
import com.packt.javapath.something.SomeClass;
public class MyClass {
  //... 
  SomeClass someClass = new SomeClass();
  //...
}
```

如您所见，导入语句允许避免使用完全限定的类名，这使得代码更易于阅读。

# 多个类导入

如果从同一包中导入了多个类或接口，则可以使用星号（`*`）通配符字符导入所有包成员。

如果`SomeClass`和`SomeOtherClass`属于同一个包，则导入语句可能如下所示：

```java
package com.packt.javapath;
import com.packt.javapath.something.*;
public class MyClass {
  //... 
  SomeClass someClass = new SomeClass();
  SomeOtherClass someClass1 = new SomeOtherClass();
  //...
}
```

使用星号的优点是导入语句的列表较短，但这样的风格隐藏了导入的类和接口的名称。因此，程序员可能不知道它们确切来自哪里。此外，当两个或更多的包包含具有相同名称的成员时，你只需将它们明确地导入为单个类导入。否则，编译器会生成一个错误。

另一方面，偏爱通配符导入的程序员认为它有助于防止意外地创建一个已经存在于其中一个导入包中的类的名称。因此，在风格和配置 IDE 以使用或不使用通配符导入时，你必须自己做出选择。

在 IntelliJ IDEA 中，默认的导入风格是使用通配符。如果你想切换到单个类导入，请点击 文件 | 其他设置 | 默认设置，如下面的截图所示：

![](img/15e4c441-99b0-4266-b669-26820d240037.png)

在打开的界面上，选择编辑器 | Java 并勾选使用单个类导入复选框：

![](img/7dbd6160-11da-4075-afe8-394c3b78582c.png)

在这个页面上还有其他你可能会觉得有用的设置，所以尽量记住如何访问它。

# 静态导入

静态导入允许单独导入一个类或接口的公共成员——字段和方法。如果你查看我们的一个测试类，你会看到以下的静态导入语句：

```java
import static org.junit.jupiter.api.Assertions.*;

```

这个语句允许我们写成以下形式：

```java
Person p = new Person("Joe", "Blow", dob);
assertTrue(p.equals(p));

```

那就是不再写这样的代码：

```java
Person p = new Person("Joe", "Blow", dob);
Assertions.assertTrue(p.equals(p));

```

这是静态导入用法的一个广泛案例。另一个常见的用例是静态导入接口或 `enum` 的常量。例如，如果我们有一个如下所示的接口：

```java
package com.packt.javapath.api;
public interface Constants {
  String NAME = "name";
}
```

然后，要使用它的常量，可以静态导入它们：

```java
package com.packt.javapath;
import static com.packt.javapath.api.Constants.*;
public class MyClass {
  //...
  String s = "My " + NAME + " is Joe";
  System.out.println(s);        //Prints: My name is Joe
  //...
} 
```

顺便说一句，同样的效果也可以通过非静态导入那个 `Constants` 接口并让类实现它来实现：

```java
package com.packt.javapath;
import com.packt.javapath.api.Constants;
public class MyClass implements Constants {
  //...
  String s = "My " + NAME + " is Joe";
  System.out.println(s);        //Prints: My name is Joe
  //...
} 
```

这种实现接口以使用它们的常量的风格在 Java 程序员中非常流行。

为了使用 `enum` 常量，使用静态导入的示例看起来类似：

```java
import static java.time.DayOfWeek.*;
```

它允许代码使用 `DayOfWeek` 常量作为 `MONDAY`，而不是 `DayOfWeek.MONDAY`。

# 访问修饰符

有三个明确的访问修饰符——public、private 和 protected——以及一个隐式的（默认的）访问修饰符，当没有设置访问修饰符时会被暗示。它们可以应用于顶级类或接口、它们的成员和构造函数。顶级类或接口可以包括成员类或接口。类的其他成员包括字段和方法。类还有构造函数。

为了演示可访问性，让我们创建一个包名为 `com.packt.javapath.Ch07demo.pack01` 的包，其中包含两个类和两个接口：

```java
public class PublicClass01 {
  public static void main(String[] args){
    //We will write code here
  }
}

class DefaultAccessClass01 {
}

public interface PublicInterface01 {
  String name = "PublicInterface01";
}

interface DefaultAccessInterface01 {
  String name = "DefaultAccessInterface01";
}
```

我们还将创建另一个包名为 `com.packt.javapath.Ch07demo.pack02` 的包，并在其中放置一个类：

```java
public class PublicClass02 {
  public static void main(String[] args){
    //We will write code here
  }
}
```

前述的每个类和接口都在自己的文件中：

![](img/eac9098f-e718-4bd2-ab43-b02aac7aa66e.png)

现在我们准备探讨类、接口、它们的成员和构造函数的可访问性。

# 顶级类或接口的可访问性

公共类或接口可从任何地方访问。我们可以导入它们并从另一个包中访问它们：

```java
import com.packt.javapath.Ch07demo.pack01.PublicClass01;
import com.packt.javapath.Ch07demo.pack01.PublicInterface01;
//import com.packt.javapath.Ch07demo.pack01.DefaultAccessClass01;
//import com.packt.javapath.Ch07demo.pack01.DefaultAccessInterface01;

public class PublicClass02 {
  public static void main(String[] args){
    System.out.println(PublicInterface01.name);
    PublicClass01 o = new PublicClass01();

  }
}
```

在上述代码中，两个导入语句被注释掉了，因为它们会生成错误。这是因为在`DefaultAccessClass01`类和`DefaultAccessClass01`接口中，我们没有使用访问修饰符，这使它们只能被同一包中的成员访问。

没有访问修饰符，顶级类或接口只能被同一包中的成员访问。

将顶级类或接口的访问修饰符声明为`private`将使它们无法访问，因此对于顶级类或接口使用`private`访问修饰符是没有意义的。

`protected`关键字不能应用于顶级。这个限制并不明显。我们将在下一节中看到，`protected`意味着它对包成员和子类可访问。因此，有人可能会认为`protected`访问也适用于顶级类或接口。然而，Java 的作者决定不这样做，如果您尝试将顶级类或接口设为`protected`，编译器将生成异常。

但是，`private`和`protected`访问修饰符可以应用于内部类或接口——顶级类或接口的成员。

# 类或接口成员的访问

即使类或接口成员被声明为公共的，如果封闭类或接口是不可访问的，则无法访问它们。因此，以下所有讨论都将在假设类或接口是可访问的情况下进行。

类或接口的成员可以访问同一类或接口的其他成员，无论它们有什么访问修饰符。这是有道理的，不是吗？这一切都发生在同一个封闭类或接口中。

默认情况下，接口成员是公共的。因此，如果可以访问接口本身，则可以访问没有访问修饰符的成员。而且，只是提醒您，接口字段默认为静态和最终（常量）。

另一方面，没有访问修饰符的类成员只能被包成员访问。因此，类或接口可能是公共的，但它们的成员是不太可访问的，除非明确地公开。

私有类或接口成员只能被同一类或接口的其他成员访问。这是最受限制的访问。即使类的子类也不能访问其父类的私有成员。

包内受保护成员可被同一包中的其他成员以及类或接口的子类访问，这意味着受保护成员可以被重写。这通常被程序员用作意图的表达：他们将那些期望被重写的成员设置为受保护的。否则，他们将它们设置为私有或公共。默认的——无访问修饰符——访问极少被使用。

**私有**：只允许同一类（或接口）访问

**无修饰符（默认）**：允许从同一类（或接口）和同一包中访问

**受保护**：允许从同一类（或接口）、同一包和任何子类中访问

**公共**：允许从任何地方访问

内部类和接口也遵循相同的访问规则。下面是一个包含内部类和接口的类的示例：

```java
public class PublicClass01 {
  public static void main(String[] args){
    System.out.println(DefaultAccessInterface01.name);
    DefaultAccessClass01 o = new DefaultAccessClass01();
  }
  class DefaultAccessClass{
  }
  protected class ProtectedClass{
  }
  private class PrivateClass{
  }
  interface DefaultAccessInterface {
  }
  protected class ProtectedInterface{
  }
  private class PrivateInterface{
  }
}
```

下面是一个带有内部类和接口的接口：

```java
public interface PublicInterface01 {
  String name = "PublicInterface01";

  class DefaultAccessClass{
  }
  interface DefaultAccessInterface {
  }
}
```

正如您所见，接口的内部类和接口只允许默认（公共）访问。

并且，为了重申我们已经讨论过的内容，我们将简要提及成员可访问性的一些其他相关方面：

+   静态嵌套类（在静态类的情况下被称为嵌套类）无法访问同一类的非静态成员，而它们可以访问它

+   作为某个顶层类的成员，静态嵌套类可以是公共的、受保护的、包可访问的（默认）、或私有的

+   类的公共、受保护和包可访问成员会被子类继承

# 构造函数的可访问性与任何类成员相同

正如本节标题所述，这就是我们可以说的关于构造函数的可访问性的一切。当然，当我们谈论构造函数时，我们只谈论类。

构造函数有一个有趣的特性，就是它们只能具有私有访问权限。这意味着一个类可以提供自己的工厂方法（见第六章，*接口、类和对象构造*），控制每个对象如何构造，甚至控制可以将多少个对象放入循环中。在每个对象都需要访问某个资源（文件或另一个数据库）的情况下，最后一个特性尤为有价值，因为该资源对并发访问的支持有限。以下是这样一个具有限制创建对象数量的最简单版本的工厂方法的样子：

```java
private String field;
private static int count;
private PublicClass02(String s){
  this.field = s;
}
public static PublicClass02 getInstance(String s){
  if(count > 5){
    return null;
  } else {
    count++;
    return new PublicClass02(s);
  }
}
```

这段代码的用处不大，我们只是展示它来演示私有可访问构造函数的使用方式。这是可能的，因为每个类成员都可以访问所有其他类成员，无论它们的访问修饰符如何。

所有与可访问性相关的特性除非产生了一些优势，否则都不会被需要。这就是我们接下来要讨论的内容 - 关于面向对象编程的中心概念，称为封装，它是不可能没有可访问性控制。

# 封装

面向对象编程的概念诞生于管理软件系统不断增加的复杂性的努力中。封装将数据和程序捆绑在一个对象中，并对它们进行了受控访问（称为封装），从而实现了更好地组织分层的数据和程序，其中一些隐藏，其他则可以从外部访问。前面部分描述的可访问性控制是它的重要部分之一。与继承、接口（也称为抽象）和多态性一起，封装成为面向对象编程的中心概念之一。

往往没有一个面向对象编程的概念能清晰地与另一个分开。接口也有助于隐藏（封装）实现细节。继承可以覆盖和隐藏父类的方法，为可访问性增加了动态性。所有这三个概念使得可以增加多态性的概念 - 相同的对象能够根据上下文呈现为不同类型（基于继承或已实现的接口），或者根据数据可用性改变其行为（使用组合 - 我们将在第八章中讨论，*面向对象设计(OOD)原则*或方法重载、隐藏和覆盖）。

但是，如果没有封装，上述任何一个概念都是不可能的。这就是为什么它是面向对象编程四个概念中最基本的概念。你可能会经常听到它被提到，所以我们决定专门讲解封装概念的术语及其提供的优势：

+   数据隐藏和解耦

+   灵活性、可维护性、重构

+   可重用性

+   可测试性

# 数据隐藏和解耦

当我们将对象状态（字段的值）和一些方法私有化或施加其他限制访问内部对象数据的措施时，我们参与了*数据隐藏*。对象功能的用户只能根据其可访问性调用特定方法，而不能直接操纵对象的内部状态。对象的用户可能不知道功能的具体实现方式和数据存储方式。他们将所需的输入数据传递给可访问的方法，并获得结果。这样，我们将内部状态与其使用和 API 的实现细节*解耦*了。

在同一个类中将相关方法和数据分组也增加了*解耦*，这次是在不同功能的不同区域之间。

您可能会听到密集耦合这个词，作为一种应该只在没有其他选择的情况下允许的东西，因为通常意味着更改一个部分就需要相应更改另一个部分。即使在日常生活中，我们也喜欢处理模块化的系统，允许只替换一个模块而不更改其余系统的任何其他组件。

这就是为什么程序员通常喜欢松散耦合，虽然这通常会以无法确定在所有可能的执行路径上都不存在意外惊喜的代价。一个经过深思熟虑的覆盖关键用例的测试系统通常有助于降低缺陷在生产中传播的可能性。

# 灵活性、可维护性和重构

在我们谈到解耦时，灵活性和可维护性的想法可能会因为联想而产生。松散耦合的系统更加灵活和易于维护。

例如，在第六章中，*接口、类和对象构造*，我们演示了一种灵活的解决方案来实现对象工厂：

```java
public static Calculator createInstance(){
  WhichImpl whichImpl = 
      Utils.getWhichImplValueFromConfig(Utils.class,
            Calculator.CONF_NAME, Calculator.CONF_WHICH_IMPL);
  switch (whichImpl){
    case multiplies:
      return new CalculatorImpl();
    case adds:
      return new AnotherCalculatorImpl();
    default:
      throw new RuntimeException("Houston, we have another problem."+
                  " We do not have implementation for the key " +
                  Calculator.CONF_WHICH_IMPL + " value " + whichImpl);
    }
}
```

它与其 `Calculator` 接口（其 API）紧密耦合，但这是不可避免的，因为它是实现必须遵守的协议。至于工厂内部的实现，只要它遵循协议就可以更自由地从任何限制中脱颖而出。

我们只能创建实现的每个实例一次，并只返回那个实例（使每个类成为单例）。以下是以单例模式实现 `CalculatorImpl` 的示例：

```java
private static Calculator calculator = null;
public static Calculator createInstance(){
  WhichImpl whichImpl = 
      Utils.getWhichImplValueFromConfig(Utils.class,
            Calculator.CONF_NAME, Calculator.CONF_WHICH_IMPL);
  switch (whichImpl){
    case multiplies:
      if(calculator == null){
        calculator = new CalculatorImpl();
      }
      return calculator;
    case adds:
      return new AnotherCalculatorImpl();
    default:
      throw new RuntimeException("Houston, we have another problem."+
                      " We do not have implementation for the key " +
                  Calculator.CONF_WHICH_IMPL + " value " + whichImpl);
    }
}
```

或者我们可以在工厂中添加另一个 `Calculator` 实现作为嵌套类，并使用它来替代 `CalculatorImpl`：

```java
public static Calculator createInstance(){
  String whichImpl = Utils.getStringValueFromConfig(CalculatorFactory.class,
            "calculator.conf", "which.impl");
  if(whichImpl.equals("multiplies")){
    return new Whatever();
  } else if (whichImpl.equals("adds")){
    return new AnotherCalculatorImpl();
  } else {
    throw new RuntimeException("Houston, we have a problem. " +
              "Unknown key which.impl value " + whichImpl +
              " is in config.");
  }

}

static class Whatever implements Calculator {
  public static String addOneAndConvertToString(double d){
    System.out.println(Whatever.class.getName());
    return Double.toString(d + 1);
  }
  public int multiplyByTwo(int i){
    System.out.println(Whatever.class.getName());
    return i * 2;
  }
}
```

工厂的客户端代码不会发现任何区别，除非它在从工厂返回的对象上使用 `getClass()` 方法打印有关类的信息。但这是另一件事情。从功能上讲，我们的新实现 `Whatever` 将像旧实现一样工作。

实际上，这是一个常见的做法，可以在一个发布版中从一个内部实现改变到另一个。当然会有漏洞修复和新功能添加。随着实现代码的不断发展，其程序员会不断地关注重构的可能性。在计算机科学中，Factoring 是 Decomposition 的同义词，Decomposition 是将复杂代码拆分为更简单的部分的过程，以使代码更易于阅读和维护。例如，假设我们被要求编写一个方法，该方法接受 `String` 类型的两个参数（每个参数都表示一个整数），并将它们相加作为一个整数返回。经过一番思考，我们决定这样做：

```java
public long sum(String s1, String s2){
  int i1 = Integer.parseInt(s1);
  int i2 = Integer.parseInt(s1);
  return i1 + i2;
}
```

但然后我们要求提供可能输入值的样本，这样我们就可以在接近生产条件的情况下测试我们的代码。结果发现，一些值可以高达 10,000,000,000，这超过了 2,147,483,647（Java 允许的最大`Integer.MAX_VALUE`整数值）。因此，我们已经将我们的代码更改为以下内容：

```java
public long sum(String s1, String s2){
  long l1 = Long.parseLong(s1);
  long l2 = Long.parseLong(s2);
  return l1 + l2;
}
```

现在我们的代码可以处理高达 9,223,372,036,854,775,807 的值（这是`Long.MAX_VALUE`）。我们将代码部署到生产环境，并且在几个月内一直运行良好，被一个处理统计数据的大型软件系统使用。然后系统切换到了新的数据源，代码开始出现问题。我们进行了调查，发现新的数据源产生的值可以包含字母和一些其他字符。我们已经测试了我们的代码以处理这种情况，并发现以下行抛出`NumberFormatException`：

```java
long l1 = Long.parseLong(s1);

```

我们与领域专家讨论了情况，他们建议我们记录不是整数的值，跳过它们，并继续进行求和计算。因此，我们已经修复了我们的代码，如下所示：

```java
public long sum(String s1, String s2){
  long l1 = 0;
  try{
    l1 = Long.parseLong(s1);
  } catch (NumberFormatException ex){
    //make a record to a log
  }
  long l2 = 0;
  try{
    l2 = Long.parseLong(s2);
  } catch (NumberFormatException ex){
    //make a record to a log
  }
  return l1 + l2;
}
```

我们迅速将代码发布到生产环境，但是在下一个发布中获得了新的要求：输入的`String`值可以包含小数。因此，我们已经改变了处理输入`String`值的方式，假设它们带有小数值（这也包括整数值），并重构了代码，如下所示：

```java
private long getLong(String s){
  double d = 0;
  try{
    d = Double.parseDouble(s);
  } catch (NumberFormatException ex){
    //make a record to a log
  }
  return Math.round(d);
}
public long sum(String s1, String s2){
  return getLong(s1) + getLong(s2);
}
```

这就是重构所做的事情。它重新构造了代码而不改变其 API。随着新的需求不断出现，我们可以修改`getLong()`方法，甚至不用触及`sum()`方法。我们还可以在其他地方重用`getLong()`方法，这将是下一节的主题。

# 可重用性

封装绝对使得实现可重用性变得更容易，因为它隐藏了实现细节。例如，在前一节中我们编写的`getLong()`方法可以被同一类的另一个方法重用：

```java
public long sum(int i, String s2){
  return i + getLong(s2);
}
```

它甚至可以被公开并被其他类使用，就像下面的代码一样：

```java
int i = new Ch07DemoApp().getLong("23", "45.6");
```

这将是一个组合的例子，当某些功能是使用不相关的类的方法（通过组合）构建时。而且，由于它不依赖于对象状态（这样的方法称为无状态），因此它可以是静态的：

```java
int i = Ch07DemoApp.getLong("23", "45.6");
```

如果该方法在运行时由多个其他方法同时使用，甚至这样一个简单的代码也可能需要受到保护（同步），防止并行使用。但是这样的考虑超出了本书的范围。如果有疑问，请不要使方法静态。

如果您阅读面向对象编程的历史，您会发现继承最初被赋予了，除其他外，成为代码重用的主要机制。而它确实完成了任务。子类继承（重用）了其父类的所有方法，并且只覆盖那些需要为子类专业化的方法。

但在实践中，似乎其他重复使用技术更受欢迎，尤其是对于重复使用的方法是无状态的情况。我们将在第八章中更详细地讨论这一原因，*面向对象设计（OOD）原则*。

# 可测试性

代码可测试性是另一个封装有所帮助的领域。如果实现细节没有被隐藏，我们就需要测试每一行代码，并且每次更改实现中的任何行时都需要更改测试。但是，隐藏细节在 API 外观后面允许我们仅专注于所需的测试用例，并且受可能输入数据集（参数值）的限制。

此外，还有一些框架允许我们创建一个对象，根据输入参数的特定值返回特定结果。Mockito 是一个流行的框架，它可以做到这一点（[`site.mockito.org`](http://site.mockito.org)）。这样的对象称为模拟对象。当您需要从一个对象的方法中获取特定结果以测试其他方法时，它们特别有帮助，但您不能运行作为数据源的方法的实际实现，因为您没有必要的数据在数据库中，例如，或者它需要一些复杂的设置。为了解决这个问题，您可以用返回您需要的数据的实际实现替换某些方法的实际实现——模拟它们，无条件地或以对某些输入数据做出响应。没有封装，这样模拟方法行为可能是不可能的，因为客户端代码将与特定实现绑定，您将无法在不更改客户端代码的情况下更改它。

# 练习 - 遮蔽

编写演示变量遮蔽的代码。我们还没有讨论过它，所以您需要做一些研究。

# 回答

这是一个可能的解决方案：

```java
public class ShadowingDemo {
  private String x = "x";
  public void printX(){
    System.out.println(x);   
    String x = "y";
    System.out.println(x);   
  }
}
```

如果您运行 `new ShadowingDemo().printX();`，它将首先打印 `x`，然后打印 `y`，因为以下行中的局部变量 `x` 遮蔽了 `x` 实例变量：

```java
String x = "y";

```

请注意，遮蔽可能是缺陷的源泉，也可能有益于程序。如果没有它，您将无法使用已经被实例变量使用的局部变量标识符。这里还有另一个案例的例子，变量遮蔽有助于：

```java
private String x = "x";
public void setX(String x) {
  this.x = x;
}
```

`x` 局部变量（参数）遮蔽了 `x` 实例变量。它允许使用相同的标识符来命名一个局部变量，该标识符已经被用于实例变量名。为了避免可能的混淆，建议使用关键字 `this` 引用实例变量，就像我们在上面的示例中所做的那样。

# 摘要

在这一章中，你了解了面向对象语言的一个基本特性——类、接口、它们的成员和构造函数的可访问性规则。现在你可以从其他包中导入类和接口，并避免使用它们的完全限定名。所有这些讨论使我们能够介绍面向对象编程的核心概念——封装。有了这个，我们就可以开始对**面向对象设计**（**OOD**）原则进行有根据的讨论。

下一章介绍了 Java 编程的更高层次视角。它讨论了良好设计的标准，并提供了一份对经过验证的 OOD 原则的指南。每个设计原则都有详细的描述，并使用相应的代码示例进行了说明。
