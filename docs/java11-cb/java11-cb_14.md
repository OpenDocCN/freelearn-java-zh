# 第十四章：测试

本章展示了如何测试你的应用程序——如何捕获和自动化测试用例，如何在将 API 与其他组件集成之前对 API 进行单元测试，以及如何集成所有单元。我们将向您介绍**行为驱动开发**（**BDD**）并展示它如何成为应用程序开发的起点。我们还将演示如何使用 JUnit 框架进行单元测试。有时，在单元测试期间，我们必须使用一些虚拟数据存根依赖项，这可以通过模拟依赖项来完成。我们将向您展示如何使用模拟库来做到这一点。我们还将向您展示如何编写固定装置来填充测试数据，然后如何通过集成不同的 API 并一起测试它们来测试应用程序的行为。我们将涵盖以下内容：

+   使用 Cucumber 进行行为测试

+   使用 JUnit 对 API 进行单元测试

+   单元测试通过模拟依赖关系

+   使用固定装置来填充测试数据

+   集成测试

# 介绍

经过良好测试的代码为开发人员提供了心灵上的安宁。如果你觉得为你正在开发的新方法编写测试太过繁琐，那么通常第一次就做不对。无论如何，你都必须测试你的方法，而在长远来看，设置或编写单元测试比构建和启动应用程序多次要少时间消耗——每次代码更改和每次逻辑通过都要这样做。

我们经常感到时间紧迫的原因之一是我们在估算时间时没有包括编写测试所需的时间。一个原因是我们有时会忘记这样做。另一个原因是我们不愿意给出更高的估计，因为我们不想被认为技能不够。不管原因是什么，这种情况经常发生。只有经过多年的经验，我们才学会在估算中包括测试，并赢得足够的尊重和影响力，能够公开断言正确的做事方式需要更多的时间，但从长远来看节省了更多的时间。此外，正确的做法会导致健壮的代码，减少了很多压力，这意味着整体生活质量更好。

早期测试的另一个优势是在主要代码完成之前发现代码的弱点，这时修复它很容易。如果需要，甚至可以重构代码以提高可测试性。

如果你还不相信，记下你阅读此文的日期，并每年回顾一次，直到这些建议对你来说变得显而易见。然后，请与他人分享你的经验。这就是人类取得进步的方式——通过将知识从一代传递到下一代。

从方法上讲，本章的内容也适用于其他语言和职业，但示例主要是为 Java 开发人员编写的。

# 使用 Cucumber 进行行为测试

以下是程序员经常提出的三个反复出现的抱怨：

+   缺乏需求

+   需求的模糊性

+   需求一直在变化

有很多建议和流程可以帮助缓解这些问题，但没有一个能够完全消除它们。在我们看来，最成功的是敏捷过程方法与 BDD 相结合，使用 Cucumber 或其他类似框架。短迭代允许快速调整和业务（客户）与程序员之间的协调，而 BDD 与 Cucumber 以 Gherkin 捕获需求，但没有维护大量文档的开销。

Gherkin 中编写的需求必须被分解成**特性**。每个特性存储在一个扩展名为`.feature`的文件中，包含一个或多个描述特性不同方面的**场景**。每个场景由描述用户操作或输入数据以及应用程序对其的响应的步骤组成。

程序员实现必要的应用程序功能，然后使用它来在一个或多个`.java`文件中实现场景。每个步骤都在一个方法中实现。

在实施后，这些场景将成为一套测试，可以是像单元测试一样细粒度，也可以是像集成测试一样高级，以及介于两者之间的任何形式。这完全取决于谁编写了场景以及应用程序代码的结构。如果场景的作者是业务人员，那么场景往往更高级。但是，如果应用程序的结构使得每个场景（可能有多个输入数据的排列组合）都被实现为一个方法，那么它就可以有效地作为一个单元测试。或者，如果一个场景涉及多个方法甚至子系统，它可以作为一个集成测试，而程序员可以用更细粒度（更像单元测试）的场景来补充它。之后，在代码交付后，所有场景都可以作为回归测试。

您所付出的代价是场景的开销、维护，但回报是捕获需求并确保应用程序确实符合要求的正式系统。话虽如此，需要说明的一点是：捕获 UI 层的场景通常更加棘手，因为 UI 往往更频繁地发生变化，特别是在应用程序开发的初期。然而，一旦 UI 稳定下来，对其的需求也可以使用 Selenium 或类似的框架在 Cucumber 场景中进行捕获。

# 如何做...

1.  安装 Cucumber。Cucumber 的安装只是将框架作为 Maven 依赖项添加到项目中。由于我们将添加多个 Cucumber JAR 文件，而且它们都必须是相同版本，因此在`pom.xml`中添加`cucumber.version`属性是有意义的。

```java
    <properties>
        <cucumber.version>3.0.2</cucumber.version>
    </properties>
```

现在我们可以在`pom.xml`中将 Cucumber 主 JAR 文件添加为依赖项：

```java
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-java</artifactId>
    <version>${cucumber.version}</version>
    <scope>test</scope>
</dependency>

```

或者，如果您更喜欢流畅的基于流的编码风格，您可以添加一个不同的 Cucumber 主 JAR 文件：

```java
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-java8</artifactId>
    <version>${cucumber.version}</version>
    <scope>test</scope>
</dependency>

```

如果您的项目尚未设置 JUnit 作为依赖项，您可以按照以下步骤添加它以及另一个`cucumber-junit` JAR 文件：

```java
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-junit</artifactId>
    <version>${cucumber.version}</version>
    <scope>test</scope>
</dependency> 
```

以上是必要的，如果您计划利用 JUnit 断言。请注意，目前为止，Cucumber 不支持 JUnit 5。

或者，您可以使用 TestNG（[`testng.org`](https://testng.org)）中的断言：

```java
<dependency>
    <groupId>org.testng</groupId>
    <artifactId>testng</artifactId>
    <version>6.14.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-testng</artifactId>
    <version>${cucumber.version}</version>
    <scope>test</scope>
</dependency>
```

如您所见，在这种情况下，您需要添加`cucumber-testng` JAR 文件，而不是`cucumber-junit` JAR 文件。TestNG 提供了丰富多样的断言方法，包括深度集合和其他对象比较。

1.  运行 Cucumber。`cucumber-junit` JAR 文件还提供了一个`@RunWith`注解，将一个类指定为测试运行器：

```java
package com.packt.cookbook.ch16_testing;

import cucumber.api.CucumberOptions;
import cucumber.api.junit.Cucumber;
import org.junit.runner.RunWith;

@RunWith(Cucumber.class)
public class RunScenariousTest {
}
```

执行前述类将执行与运行器所在的相同包中的所有场景。Cucumber 读取每个`.feature`文件及其中的场景。对于每个场景的每个步骤，它尝试在与运行器和`.feature`文件相同的包中找到其实现。它按照场景中列出的顺序执行每个已实现的步骤。

1.  创建一个`.feature`文件。正如我们已经提到的，一个`.feature`文件包含一个或多个场景。文件的名称对 Cucumber 没有任何意义。文件的内容以`Feature`关键字（后面跟着冒号`:`）开始。接下来的文本描述了功能，并且与文件名类似，对 Cucumber 没有任何意义。功能描述在`Scenario`关键字（后面跟着冒号`:`）开始新行时结束。这就是第一个场景描述开始的地方。以下是一个例子：

```java
Feature: Vehicle speed calculation
 The calculations should be made based on the assumption
 that a vehicle starts moving, and driving conditions are 
 always the same.

Scenario: Calculate speed
 This the happy path that demonstrates the main case
```

当以下关键字之一在新行上开始时，场景描述结束：`Given`、`When`、`Then`、`And`或`But`。每个这些关键字在新行开始时，都表示步骤定义的开始。对于 Cucumber 来说，这样的关键字除了表示步骤定义的开始外，没有其他意义。但对于人类来说，如果场景以`Given`关键字开始，即描述系统的初始状态的步骤，那么阅读起来更容易。可能会有几个其他步骤（前提条件）跟随；每个步骤都以新行和`And`或`But`关键字开头，例如如下所示：

```java
Given the vehicle has 246 hp engine and weighs 4000 pounds
```

之后，步骤组描述了动作或事件。为了人类可读性，该组通常以新行的`When`关键字开头。其他动作或事件随后，每个都以新行和`And`或`But`关键字开头。建议将该组中的步骤数量保持在最小限度，以便每个场景都能够集中精力，例如如下所示：

```java
When the application calculates its speed after 10.0 sec
```

场景中的最后一组步骤以新行中的`Then`关键字开始。它们描述了预期的结果。与前两组步骤一样，该组中的每个后续步骤都以新行和`And`或`But`关键字开头，例如如下所示：

```java
Then the result should be 117.0 mph
```

总结之前，该功能如下：

```java
Feature: Vehicle speed calculation
 The calculations should be made based on the assumption
 that a vehicle starts moving, and driving conditions are
 always the same.

Scenario: Calculate speed
 This the happy path that demonstrates the main case

 Given the vehicle has 246 hp engine and weighs 4000 pounds
 When the application calculates its speed after 10.0 sec
 Then the result should be 117.0 mph
```

我们将其放在以下文件夹中的`src/test/resources/com/packt/cookbook/Chapter14_testing`中的`CalculateSpeed.feature`文件中。

请注意，它必须位于`test/resources`文件夹中，并且其路径必须与`RunScenariosTest`测试运行器所属的包名称匹配。

测试运行器像执行任何 JUnit 测试一样，例如使用`mvn test`命令，或者只需在 JDE 中运行它。执行时，它会查找同一包中的所有`.feature`文件（Maven 将它们从`resources`文件夹复制到`target/classes`文件夹，因此将它们设置在类路径上）。然后按顺序读取每个场景的步骤，并尝试在同一包中找到每个步骤的实现。

正如我们已经提到的，文件的名称对于 Cucumber 来说没有任何意义。它首先寻找`.feature`扩展名，然后找到第一个步骤，并在同一目录中尝试找到一个类，该类中有一个与步骤相同的注释方法。

为了说明其含义，让我们通过执行测试运行器来运行创建的特性。结果将如下所示：

```java
cucumber.runtime.junit.UndefinedThrowable: 
The step "the vehicle has 246 hp engine and weighs 4000 pounds" 
                                                     is undefined
cucumber.runtime.junit.UndefinedThrowable: 
The step "the application calculates its speed after 10.0 sec" 
                                                     is undefined
cucumber.runtime.junit.UndefinedThrowable: 
The step "the result should be 117.0 mph" is undefined

Undefined scenarios:
com/packt/cookbook/ch16_testing/CalculateSpeed.feature:6 
                                                # Calculate speed
1 Scenarios (1 undefined)
3 Steps (3 undefined)
0m0.081s

You can implement missing steps with the snippets below:

@Given("the vehicle has {int} hp engine and weighs {int} pounds")
public void the_vehicle_has_hp_engine_and_weighs_pounds(Integer 
                                             int1, Integer int2) {
 // Write code here that turns the phrase above 
 // into concrete actions
 throw new PendingException();
}

@When("the application calculates its speed after {double} sec")
public void the_application_calculates_its_speed_after_sec(Double 
                                                         double1) {
 // Write code here that turns the phrase above 
 // into concrete actions
 throw new PendingException();
}

@Then("the result should be {double} mph")
public void the_result_should_be_mph(Double double1) {
 // Write code here that turns the phrase above 
 // into concrete actions
 throw new PendingException();
}
```

正如您所看到的，Cucumber 不仅告诉我们有多少个`undefined`特性和场景，它甚至提供了一种可能的实现方式。请注意，Cucumber 允许使用大括号中的类型传递参数。以下是内置类型：`int`、`float`、`word`、`string`、`biginteger`、`bigdecimal`、`byte`、`short`、`long`和`double`。`word`和`string`之间的区别在于后者允许空格。但 Cucumber 还允许我们定义自定义类型。

1.  编写并运行步骤定义。Cucumber 术语中的`undefined`可能会令人困惑，因为我们确实定义了特性和场景。我们只是没有实现它们。因此，Cucumber 消息中的`undefined`实际上意味着`未实现`。

要开始实现，我们首先在与测试运行器相同的目录中创建一个名为`CalculateSpeedSteps`的类。类名对于 Cucumber 来说没有意义，所以您可以根据自己的喜好命名它。然后，我们将之前建议的三种方法与注释一起复制并放入该类中：

```java
package com.packt.cookbook.ch16_testing;

import cucumber.api.PendingException;
import cucumber.api.java.en.Given;
import cucumber.api.java.en.Then;
import cucumber.api.java.en.When;

public class Calc {
  @Given("the vehicle has {int} hp engine and weighs {int} pounds")
  public void the_vehicle_has_hp_engine_and_weighs_pounds(Integer 
                                              int1, Integer int2) {
        // Write code here that turns the phrase above 
        // into concrete actions
        throw new PendingException();
  }

  @When("the application calculates its speed after {double} sec")
  public void the_application_calculates_its_speed_after_sec(Double 
                                                         double1) {
        // Write code here that turns the phrase above 
        // into concrete actions
        throw new PendingException();
  }

  @Then("the result should be {double} mph")
    public void the_result_should_be_mph(Double double1) {
        // Write code here that turns the phrase above 
        // into concrete actions
        throw new PendingException();
  }
}
```

如果我们再次执行测试运行器，输出将如下所示：

```java
cucumber.api.PendingException: TODO: implement me
 at com.packt.cookbook.ch16_testing.CalculateSpeedSteps.the_vehicle
      _has_hp_engine_and_weighs_pounds(CalculateSpeedSteps.java:13)
 at *.the vehicle has 246 hp engine and weighs 4000 pounds(com/packt/cookbook/ch16_testing/CalculateSpeed.feature:9)

Pending scenarios:
com/packt/cookbook/ch16_testing/CalculateSpeed.feature:6 
                                                 # Calculate speed
1 Scenarios (1 pending)
3 Steps (2 skipped, 1 pending)
0m0.055s

cucumber.api.PendingException: TODO: implement me
 at com.packt.cookbook.ch16_testing.CalculateSpeedSteps.the_vehicle       has_hp_engine_and_weighs_pounds(CalculateSpeedSteps.java:13)
 at *.the vehicle has 246 hp engine and weighs 4000 pounds(com/packt/cookbook/ch16_testing/CalculateSpeed.feature:9)
```

运行器在第一个`PendingException`处停止执行，因此其他两个步骤被跳过。如果系统地应用 BDD 方法论，那么特性将首先编写——在编写应用程序的任何代码之前。因此，每个特性都会产生前面的结果。

随着应用程序的开发，每个新功能都得到了实现，并且不再失败。

# 它是如何工作的...

在要求被表达为功能后，应用程序会逐个功能地实现。例如，我们可以从创建`Vehicle`类开始：

```java
class Vehicle {
    private int wp, hp;
    public Vehicle(int weightPounds, int hp){
        this.wp = weightPounds;
        this.hp = hp;
    }
    protected double getSpeedMpH(double timeSec){
        double v = 2.0 * this.hp * 746 ;
        v = v*timeSec * 32.174 / this.wp;
        return Math.round(Math.sqrt(v) * 0.68);
    }
}
```

然后，先前显示的第一个功能的步骤可以实现如下：

```java
package com.packt.cookbook.ch16_testing;

import cucumber.api.java.en.Given;
import cucumber.api.java.en.Then;
import cucumber.api.java.en.When;
import static org.junit.Assert.assertEquals;

public class CalculateSpeedSteps {
  private Vehicle vehicle;
  private double speed;

  @Given("the vehicle has {int} hp engine and weighs {int} pounds")
  public void the_vehicle_has_hp_engine_and_weighs_pounds(Integer 
                                                  wp, Integer hp) {
        vehicle = new Vehicle(wp, hp);
  }

  @When("the application calculates its speed after {double} sec")
  public void 
        the_application_calculates_its_speed_after_sec(Double t) {
        speed = vehicle.getSpeedMpH(t);
  }

  @Then("the result should be {double} mph")
  public void the_result_should_be_mph(Double speed) {
        assertEquals(speed, this.speed, 0.0001 * speed);
  }
}
```

如果我们再次在`com.packt.cookbook.ch16_testing`包中运行测试运行器，步骤将成功执行。

现在，如果需求发生变化，并且`.feature`文件相应地进行了修改，除非应用程序代码也进行了更改并符合要求，否则测试将失败。这就是 BDD 的力量。它使要求与代码保持同步。它还允许 Cucumber 测试作为回归测试。如果代码更改违反了要求，测试将失败。

# 使用 JUnit 对 API 进行单元测试

根据维基百科，GitHub 上托管的项目中超过 30%包括 JUnit，这是一组单元测试框架，统称为 xUnit，起源于 SUnit。它在编译时作为 JAR 链接，并且（自 JUnit 4 以来）驻留在`org.junit`包中。

在面向对象编程中，一个单元可以是整个类，也可以是一个单独的方法。在实践中，我们发现最有用的是作为一个单独方法的单元。它为本章的示例提供了基础。

# 准备工作

在撰写本文时，JUnit 的最新稳定版本是 4.12，可以通过将以下 Maven 依赖项添加到`pom.xml`项目级别来使用：

```java
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.12</version>
  <scope>test</scope>
</dependency>
```

之后，您可以编写您的第一个 JUnit 测试。假设您已经在`src/main/java/com/packt/cookbook.ch02_oop.a_classes`文件夹中创建了`Vehicle`类（这是我们在第二章中讨论的代码，*OOP - 类和接口的快速跟踪*）：

```java
package com.packt.cookbook.ch02_oop.a_classes;
public class Vehicle {
  private int weightPounds;
  private Engine engine;
  public Vehicle(int weightPounds, Engine engine) {
    this.weightPounds = weightPounds;
    if(engine == null){
      throw new RuntimeException("Engine value is not set.");
    }
    this.engine = engine;
  }
  protected double getSpeedMph(double timeSec){
    double v = 2.0*this.engine.getHorsePower()*746;
    v = v*timeSec*32.174/this.weightPounds;
    return Math.round(Math.sqrt(v)*0.68);
  }
}
```

现在，您可以创建`src/test/java/com/packt/cookbook.ch02_oop.a_classes`文件夹，并在其中创建一个名为`VehicleTest.java`的新文件，其中包含`VehicleTest`类：

```java
package com.packt.cookbook.ch02_oop.a_classes;
import org.junit.Test;
public class VehicleTest {
  @Test
  public void testGetSpeedMph(){
    System.out.println("Hello!" + " I am your first test method!");
  }
}
```

使用您喜欢的 IDE 运行它，或者只需使用`mvn test`命令。您将看到包括以下内容的输出：

![](img/3ae6e5f2-a622-4f77-b934-2867786402f4.png)

恭喜！您已经创建了您的第一个测试类。它还没有测试任何东西，但这是一个重要的设置——这是以正确的方式进行操作所必需的开销。在下一节中，我们将开始实际测试。

# 如何做...

让我们更仔细地看一下`Vehicle`类。测试 getter 的价值不大，但我们仍然可以这样做，确保传递给构造函数的值由相应的 getter 返回。构造函数中的异常也属于必须测试的功能，以及`getSpeedMph()`方法。还有一个`Engine`类的对象，它具有`getHorsePower()`方法。它能返回`null`吗？为了回答这个问题，让我们看一下`Engine`类：

```java
public class Engine {
  private int horsePower;
  public int getHorsePower() {
    return horsePower;
  }
  public void setHorsePower(int horsePower) {
    this.horsePower = horsePower;
  }
}
```

`getHorsePower()`方法不能返回`null`。如果没有通过`setHorsePower()`方法显式设置，`horsePower`字段将默认初始化为零。但是返回负值是一个明显的可能性，这反过来可能会导致`getSpeedMph()`方法的`Math.sqrt()`函数出现问题。我们应该确保马力值永远不会是负数吗？这取决于方法的使用限制程度以及输入数据的来源。

类`Vehicle`的`weightPounds`字段的值也适用类似的考虑。它可能会在`getSpeedMph()`方法中由于除以零而导致`ArithmeticException`而使应用程序停止。

然而，在实践中，发动机马力和车辆重量的值几乎不可能是负数或接近零，因此我们将假设这一点，并不会将这些检查添加到代码中。

这样的分析是每个开发人员的日常例行公事和背景思考，这是朝着正确方向迈出的第一步。第二步是在单元测试中捕获所有这些思考和疑虑，并验证假设。

让我们回到我们创建的测试类。你可能已经注意到，`@Test`注解使某个方法成为测试方法。这意味着每次你发出运行测试的命令时，它都会被你的 IDE 或 Maven 运行。方法可以以任何你喜欢的方式命名，但最佳实践建议指出你正在测试的方法（在这种情况下是`Vehicle`类）。因此，格式通常看起来像`test<methodname><scenario>`，其中`scenario`表示特定的测试用例：一个成功的路径，一个失败，或者你想测试的其他条件。在第一个示例中，虽然我们没有使用后缀，但为了保持代码简单，我们将展示稍后测试其他场景的方法示例。

在测试中，你可以调用正在测试的应用程序方法，提供数据，并断言结果。你可以创建自己的断言（比较实际结果和预期结果的方法），或者你可以使用 JUnit 提供的断言。要做到后者，只需添加静态导入：

```java
import static org.junit.Assert.assertEquals;
```

如果你使用现代 IDE，你可以输入`import static org.junit.Assert`，看看有多少不同的断言可用（或者去 JUnit 的 API 文档中查看）。有十几个或更多的重载方法可用：`assertArrayEquals()`，`assertEquals()`，`assertNotEquals()`，`assertNull()`，`assertNotNull()`，`assertSame()`，`assertNotSame()`，`assertFalse()`，`assertTrue()`，`assertThat()`和`fail()`。如果你花几分钟阅读这些方法的作用将会很有帮助。你也可以根据它们的名称猜测它们的目的。下面是`assertEquals()`方法的使用示例：

```java
import org.junit.Test;
import static org.junit.Assert.assertEquals;
public class VehicleTest {
  @Test
  public void testGetSpeedMph(){
    System.out.println("Hello!" + " I am your first test method!");
    assertEquals(4, "Hello".length());
  }
}
```

我们比较单词`Hello`的实际长度和预期长度`4`。我们知道正确的数字应该是`5`，但我们希望测试失败以演示失败的行为。如果你运行前面的测试，你会得到以下结果：

![](img/e1062cd4-ecc4-430c-ab74-b23fe96f171d.png)

正如你所看到的，最后一行告诉你出了什么问题：预期值是`4`，而实际值是`5`。假设你像这样交换参数的顺序：

```java
assertEquals("Assert Hello length:","Hello".length(), 4);
```

结果将如下所示：

![](img/264588c3-e7e2-41ef-a779-f2450921c91e.png)

现在最后一条消息是误导性的。

重要的是要记住，在每个断言方法中，预期值的参数位于（在断言的签名中）**实际值之前**。

写完测试后，你会做其他事情，几个月后，你可能会忘记每个断言实际评估了什么。但有一天测试可能会失败（因为应用程序代码已更改）。你会看到测试方法名称，预期值和实际值，但你必须深入代码以找出哪个断言失败（每个测试方法通常有几个断言）。你可能会被迫添加调试语句并多次运行测试以找出原因。

为了帮助你避免这种额外的挖掘，JUnit 的每个断言都允许你添加描述特定断言的消息。例如，运行测试的这个版本：

```java
public class VehicleTest {
  @Test
  public void testGetSpeedMph(){
    System.out.println("Hello!" + " I am your first test method!");
    assertEquals("Assert Hello length:", 4, "Hello".length());
  }
}
```

如果你这样做，结果会更容易阅读：

![](img/240214a0-c228-4e70-a98e-c153f7a27484.png)

为了完成这个演示，我们将预期值更改为`5`：

```java
assertEquals("Assert Hello length:", 5, "Hello".length());
```

现在测试结果显示没有失败：

![](img/88e0720a-1f8b-4fdd-a58a-754ace2cc9aa.png)

# 它是如何工作的...

具备了对 JUnit 框架使用的基本理解，我们现在可以为计算具有特定重量和特定马力发动机的车辆速度的主要情况编写一个真正的测试方法。我们首先使用速度计算的公式手动计算预期值。例如，如果车辆的发动机功率为 246 hp，重量为 4,000 磅，那么在 10 秒内，其速度可以达到 117 英里/小时。由于速度是`double`类型，我们将使用带有一些 delta 的断言。否则，由于`double`值在计算机中的表示方式，两个`double`值可能永远不会相等。这是`org.junit.Assert`类的断言方法：

```java
void assertEquals(String message, double expected, 
                       double actual, double delta)
```

`delta`值是允许的精度。`test`方法的最终实现将如下所示：

```java
@Test
public void testGetSpeedMph(){
  double timeSec = 10.0;
  int engineHorsePower = 246;
  int vehicleWeightPounds = 4000;

  Engine engine = new Engine();
  engine.setHorsePower(engineHorsePower);

  Vehicle vehicle = new Vehicle(vehicleWeightPounds, engine);
  double speed = vehicle.getSpeedMph(timeSec);
  assertEquals("Assert vehicle (" + engineHorsePower 
            + " hp, " + vehicleWeightPounds + " lb) speed in " 
            + timeSec + " sec: ", 117, speed, 0.001 * speed);
}
```

如您所见，我们已经决定值的千分之一是我们目的的足够精度。如果我们运行前面的测试，输出将如下所示：

![](img/c294b3f1-8103-4030-8038-af6f531c11b1.png)

为了确保测试有效，我们可以将预期值设置为 119 英里/小时（与实际值相差超过 1％）并再次运行测试。结果将如下所示：

![](img/f79b70e9-475a-488a-8edc-f53d54f8b2b9.png)

我们将预期值改回 117，并继续编写我们在分析代码时讨论的其他测试用例。

让我们确保在预期时抛出异常。为此，我们添加另一个导入：

```java
import static org.junit.Assert.fail;

```

然后，我们可以编写测试代码，测试当`Vehicle`类的构造函数中传递的值为 null 时的情况（因此应该抛出异常）：

```java
@Test
public void testGetSpeedMphException(){
  int vehicleWeightPounds = 4000;
  Engine engine = null;
  try {
    Vehicle vehicle = new Vehicle(vehicleWeightPounds, engine);
    fail("Exception was not thrown");
  } catch (RuntimeException ex) {}
}
```

这个测试成功运行，这意味着`Vehicle`构造函数抛出了异常，并且代码从未到达过这一行：

```java
    fail("Exception was not thrown");

```

为了确保测试正确工作，我们临时将非 null 值传递给`Vehicle`构造函数：

```java
Engine engine = new Engine();
```

然后，我们观察输出：

![](img/e2efee35-c323-40dd-8a33-0dfb9df40e43.png)

通过这种方式，我们可以确保我们的测试按预期工作。或者，我们可以创建另一个测试，当抛出异常时失败：

```java
@Test
public void testGetSpeedMphException(){
  int vehicleWeightPounds = 4000;
  Engine engine = new Engine();
  try {
    Vehicle vehicle = new Vehicle(vehicleWeightPounds, engine);
  } catch (RuntimeException ex) {
    fail("Exception was thrown");
  }
}
```

编写这样的测试的最佳方式是在编写应用程序代码的过程中，这样您可以随着代码的复杂性增长而测试代码。否则，特别是在更复杂的代码中，您可能在编写所有代码后有问题调试它。

还有一些其他注释和 JUnit 功能对您可能有帮助，因此请参考 JUnit 文档，以更深入地了解所有框架功能。

# 通过模拟依赖项进行单元测试

编写单元测试需要控制所有输入数据。如果一个方法从其他对象接收其输入，就需要限制测试的深度，以便每个层可以作为一个单元独立测试。这就是模拟较低级别的需求时出现的情况。

模拟不仅可以垂直进行，还可以在同一级别水平进行。如果一个方法很大且复杂，您可能需要考虑将其拆分为几个较小的方法，这样您可以在模拟其他方法的同时仅测试其中一个。这是单元测试代码与其开发一起的另一个优势；在开发的早期阶段更容易重新设计代码以获得更好的可测试性。

# 准备就绪

模拟其他方法和类很简单。编码到接口（如第二章中描述的*快速跟踪到 OOP-类和接口*）使得这变得更容易，尽管有一些模拟框架允许您模拟不实现任何接口的类（我们将在本食谱的下一部分看到此类框架使用的示例）。此外，使用对象和方法工厂可以帮助您创建特定于测试的工厂实现，以便它们可以生成具有返回预期硬编码值的方法的对象。

例如，在第四章*函数式编程*中，我们介绍了`FactoryTraffic`，它生产了一个或多个`TrafficUnit`对象。在真实系统中，这个工厂会从某个外部系统中获取数据。使用真实系统作为数据源可能会使代码设置变得复杂。正如你所看到的，为了解决这个问题，我们通过根据与真实系统相似的分布生成数据来模拟数据：汽车比卡车多一点，车辆的重量取决于汽车的类型，乘客数量和有效载荷的重量等。对于这样的模拟，重要的是值的范围（最小值和最大值）应该反映出来自真实系统的值，这样应用程序就可以在可能的真实数据的全部范围内进行测试。

模拟代码的重要约束是它不应该太复杂。否则，它的维护将需要额外的开销，这将要么降低团队的生产力，要么降低测试覆盖率。

# 如何做...

`FactoryTraffic`的模拟可能如下所示：

```java
public class FactoryTraffic {
  public static List<TrafficUnit> generateTraffic(int 
    trafficUnitsNumber, Month month, DayOfWeek dayOfWeek, 
    int hour, String country, String city, String trafficLight){
    List<TrafficUnit> tms = new ArrayList();
    for (int i = 0; i < trafficUnitsNumber; i++) {
      TrafficUnit trafficUnit = 
        FactoryTraffic.getOneUnit(month, dayOfWeek,  hour, country, 
                                  city, trafficLight);
        tms.add(trafficUnit);
    }
    return tms;
  }
}
```

它组装了一个`TrafficUnit`对象的集合。在真实系统中，这些对象可以从例如某个数据库查询结果的行创建。但在我们的情况下，我们只是硬编码了这些值：

```java
public static TrafficUnit getOneUnit(Month month, 
              DayOfWeek dayOfWeek, int hour, String country, 
              String city, String trafficLight) {
  double r0 = Math.random(); 
  VehicleType vehicleType = r0 < 0.4 ? VehicleType.CAR :
  (r0 > 0.6 ? VehicleType.TRUCK : VehicleType.CAB_CREW);
  double r1 = Math.random();
  double r2 = Math.random();
  double r3 = Math.random();
  return new TrafficModelImpl(vehicleType, gen(4,1),
             gen(3300,1000), gen(246,100), gen(4000,2000),
             (r1 > 0.5 ? RoadCondition.WET : RoadCondition.DRY),    
             (r2 > 0.5 ? TireCondition.WORN : TireCondition.NEW),
             r1 > 0.5 ? ( r3 > 0.5 ? 63 : 50 ) : 63 );
}
```

如你所见，我们使用随机数生成器来为每个参数选择一个范围内的值。这个范围与真实数据的范围一致。这段代码非常简单，不需要太多的维护，但它提供了与真实数据类似的数据流给应用程序。

你可以使用另一种技术。例如，让我们重新审视`VechicleTest`类。我们可以使用其中一个模拟框架来模拟而不是创建一个真实的`Engine`对象。在这种情况下，我们使用 Mockito。以下是它的 Maven 依赖项：

```java
<dependency>
  <groupId>org.mockito</groupId>
  <artifactId>mockito-core</artifactId>
  <version>2.7.13</version>
  <scope>test</scope>
</dependency>

```

测试方法现在看起来像这样（已更改的两行已突出显示）：

```java
@Test
public void testGetSpeedMph(){
  double timeSec = 10.0;
  int engineHorsePower = 246;
  int vehicleWeightPounds = 4000;

 Engine engine = Mockito.mock(Engine.class);
  Mockito.when(engine.getHorsePower()).thenReturn(engineHorsePower);

  Vehicle vehicle =  new Vehicle(vehicleWeightPounds, engine);
  double speed = vehicle.getSpeedMph(timeSec);
  assertEquals("Assert vehicle (" + engineHorsePower 
               + " hp, " + vehicleWeightPounds + " lb) speed in " 
               + timeSec + " sec: ", 117, speed, 0.001 * speed);
}
```

如你所见，我们指示`mock`对象在调用`getHorsePower()`方法时返回一个固定值。我们甚至可以为我们想要测试的方法创建一个模拟对象：

```java
Vehicle vehicleMock = Mockito.mock(Vehicle.class);
Mockito.when(vehicleMock.getSpeedMph(10)).thenReturn(30d);

double speed = vehicleMock.getSpeedMph(10);
System.out.println(speed);

```

因此，它总是返回相同的值：

![](img/8d517f9e-a80c-4179-abb1-a42149aed65b.png)

然而，这将违背测试的目的，因为我们想测试计算速度的代码，而不是模拟它。

对于测试流的管道方法，还可以使用另一种技术。假设我们需要测试`TrafficDensity1`类中的`trafficByLane()`方法（我们也将有`TrafficDensity2`和`TrafficDensity3`）：

```java
public class TrafficDensity1 {
  public Integer[] trafficByLane(Stream<TrafficUnit> stream, 
  int trafficUnitsNumber, double timeSec,
  SpeedModel speedModel, double[] speedLimitByLane) {

    int lanesCount = speedLimitByLane.length;

    Map<Integer, Integer> trafficByLane = stream
      .limit(trafficUnitsNumber)
      .map(TrafficUnitWrapper::new)
      .map(tuw -> tuw.setSpeedModel(speedModel))
      .map(tuw -> tuw.calcSpeed(timeSec))
      .map(speed ->  countByLane(lanesCount, speedLimitByLane, speed))
      .collect(Collectors.groupingBy(CountByLane::getLane, 
               Collectors.summingInt(CountByLane::getCount)));

    for(int i = 1; i <= lanesCount; i++){
      trafficByLane.putIfAbsent(i, 0);
    }
    return trafficByLane.values()
      .toArray(new Integer[lanesCount]);
  }

  private CountByLane countByLane(int lanesCount, 
                 double[] speedLimit, double speed) {
    for(int i = 1; i <= lanesCount; i++){
      if(speed <= speedLimit[i - 1]){
        return new CountByLane(1, i);
      }
    }
    return new CountByLane(1, lanesCount);
  }
}
```

它使用了两个支持类：

```java
private class CountByLane{
  int count, lane;
  private CountByLane(int count, int lane){
    this.count = count;
    this.lane = lane;
  }
  public int getLane() { return lane; }
  public int getCount() { return count; }
}
```

它还使用以下内容：

```java
private static class TrafficUnitWrapper {
  private Vehicle vehicle;
  private TrafficUnit trafficUnit;
  public TrafficUnitWrapper(TrafficUnit trafficUnit){
    this.vehicle = FactoryVehicle.build(trafficUnit);
    this.trafficUnit = trafficUnit;
  }
  public TrafficUnitWrapper setSpeedModel(SpeedModel speedModel) {
    this.vehicle.setSpeedModel(speedModel);
    return this;
  }
  public double calcSpeed(double timeSec) {
    double speed = this.vehicle.getSpeedMph(timeSec);
    return Math.round(speed * this.trafficUnit.getTraction());
  }
}
```

我们在第三章*模块化编程*中演示了这些支持类的使用，同时讨论了流。现在我们意识到测试这个类可能不容易。

因为`SpeedModel`对象是`trafficByLane()`方法的输入参数，我们可以单独测试它的`getSpeedMph()`方法：

```java
@Test
public void testSpeedModel(){
  double timeSec = 10.0;
  int engineHorsePower = 246;
  int vehicleWeightPounds = 4000;
  double speed = getSpeedModel().getSpeedMph(timeSec,
                 vehicleWeightPounds, engineHorsePower);
  assertEquals("Assert vehicle (" + engineHorsePower 
               + " hp, " + vehicleWeightPounds + " lb) speed in " 
               + timeSec + " sec: ", 117, speed, 0.001 * speed);
}

private SpeedModel getSpeedModel(){
  //FactorySpeedModel possibly
}
```

参考以下代码：

```java
public class FactorySpeedModel {
  public static SpeedModel generateSpeedModel(TrafficUnit trafficUnit){
    return new SpeedModelImpl(trafficUnit);
  }
  private static class SpeedModelImpl implements SpeedModel{
    private TrafficUnit trafficUnit;
    private SpeedModelImpl(TrafficUnit trafficUnit){
      this.trafficUnit = trafficUnit;
    }
    public double getSpeedMph(double timeSec, 
                              int weightPounds, int horsePower) {
      double traction = trafficUnit.getTraction();
      double v = 2.0 * horsePower * 746 
                 * timeSec * 32.174 / weightPounds;
      return Math.round(Math.sqrt(v) * 0.68 * traction);
    }
  }
```

如你所见，`FactorySpeedModel`的当前实现需要`TrafficUnit`对象以获取牵引值。为了解决这个问题，我们可以修改前面的代码并移除`SpeedModel`对`TrafficUnit`的依赖。我们可以通过将牵引应用到`calcSpeed()`方法来实现。`FactorySpeedModel`的新版本可以看起来像这样：

```java
public class FactorySpeedModel {
  public static SpeedModel generateSpeedModel(TrafficUnit 
                                                   trafficUnit) {
    return new SpeedModelImpl(trafficUnit);
  }
 public static SpeedModel getSpeedModel(){
 return SpeedModelImpl.getSpeedModel();
 }
  private static class SpeedModelImpl implements SpeedModel{
    private TrafficUnit trafficUnit;
    private SpeedModelImpl(TrafficUnit trafficUnit){
      this.trafficUnit = trafficUnit;
    }
    public double getSpeedMph(double timeSec, 
                     int weightPounds, int horsePower) {
      double speed = getSpeedModel()
             .getSpeedMph(timeSec, weightPounds, horsePower);
      return Math.round(speed *trafficUnit.getTraction());
    }
    public static SpeedModel getSpeedModel(){
      return  (t, wp, hp) -> {
        double weightPower = 2.0 * hp * 746 * 32.174 / wp;
        return Math.round(Math.sqrt(t * weightPower) * 0.68);
      };
    }
  }
}
```

现在可以实现测试方法如下：

```java
@Test
public void testSpeedModel(){
  double timeSec = 10.0;
  int engineHorsePower = 246;
  int vehicleWeightPounds = 4000;
  double speed = FactorySpeedModel.generateSpeedModel()
                 .getSpeedMph(timeSec, vehicleWeightPounds, 
                              engineHorsePower);
  assertEquals("Assert vehicle (" + engineHorsePower 
               + " hp, " + vehicleWeightPounds + " lb) speed in " 
               + timeSec + " sec: ", 117, speed, 0.001 * speed);
}
```

然而，`TrafficUnitWrapper`中的`calcSpeed()`方法仍未经过测试。我们可以将`trafficByLane()`方法作为一个整体进行测试：

```java
@Test
public void testTrafficByLane() {
  TrafficDensity1 trafficDensity = new TrafficDensity1();
  double timeSec = 10.0;
  int trafficUnitsNumber = 120;
  double[] speedLimitByLane = {30, 50, 65};
  Integer[] expectedCountByLane = {30, 30, 60};
  Integer[] trafficByLane = 
    trafficDensity.trafficByLane(getTrafficUnitStream2(
      trafficUnitsNumber), trafficUnitsNumber, timeSec, 
      FactorySpeedModel.getSpeedModel(),speedLimitByLane);
    assertArrayEquals("Assert count of " 
              + trafficUnitsNumber + " vehicles by " 
              + speedLimitByLane.length +" lanes with speed limit " 
              + Arrays.stream(speedLimitByLane)
                      .mapToObj(Double::toString)
                      .collect(Collectors.joining(", ")),
                      expectedCountByLane, trafficByLane);
}
```

但这将需要创建一个具有固定数据的`TrafficUnit`对象流：

```java
TrafficUnit getTrafficUnit(int engineHorsePower, 
                           int vehicleWeightPounds) {
  return new TrafficUnit() {
    @Override
    public Vehicle.VehicleType getVehicleType() {
      return Vehicle.VehicleType.TRUCK;
    }
    @Override
    public int getHorsePower() {return engineHorsePower;}
    @Override
    public int getWeightPounds() { return vehicleWeightPounds; }
    @Override
    public int getPayloadPounds() { return 0; }
    @Override
    public int getPassengersCount() { return 0; }
    @Override
    public double getSpeedLimitMph() { return 55; }
    @Override
    public double getTraction() { return 0.2; }
    @Override
    public SpeedModel.RoadCondition getRoadCondition(){return null;}
    @Override
    public SpeedModel.TireCondition getTireCondition(){return null;}
    @Override
    public int getTemperature() { return 0; }
  };
}
```

这样的解决方案不能为不同车辆类型和其他参数提供各种测试数据。我们需要重新审视`trafficByLane()`方法的设计。

# 它是如何工作的...

如果你仔细观察`trafficByLane()`方法，你会注意到问题是由于计算的位置——在私有类`TrafficUnitWrapper`内部。我们可以将其移出，并在`TrafficDensity`类中创建一个新的`calcSpeed()`方法：

```java
double calcSpeed(double timeSec) {
  double speed = this.vehicle.getSpeedMph(timeSec);
  return Math.round(speed * this.trafficUnit.getTraction());
}
```

然后，我们可以改变其签名，并将`Vehicle`对象和`traction`系数作为参数包括进去：

```java
double calcSpeed(Vehicle vehicle, double traction, double timeSec){
  double speed = vehicle.getSpeedMph(timeSec);
  return Math.round(speed * traction);
}
```

让我们还向`TrafficUnitWrapper`类添加两个方法（您马上就会看到我们为什么需要它们）：

```java
public Vehicle getVehicle() { return vehicle; }
public double getTraction() { return trafficUnit.getTraction(); }
```

前面的更改允许我们重写流管道如下（更改的行用粗体标出）：

```java
Map<Integer, Integer> trafficByLane = stream
  .limit(trafficUnitsNumber)
  .map(TrafficUnitWrapper::new)
  .map(tuw -> tuw.setSpeedModel(speedModel))
  .map(tuw -> calcSpeed(tuw.getVehicle(), tuw.getTraction(), timeSec))
  .map(speed -> countByLane(lanesCount, speedLimitByLane, speed))
      .collect(Collectors.groupingBy(CountByLane::getLane, 
            Collectors.summingInt(CountByLane::getCount)));

```

通过将`calcSpeed()`方法设置为 protected，并假设`Vehicle`类在其自己的测试类`VehicleTest`中进行了测试，我们现在可以编写`testCalcSpeed()`方法：

```java
@Test
public void testCalcSpeed(){
  double timeSec = 10.0;
  TrafficDensity2 trafficDensity = new TrafficDensity2();

  Vehicle vehicle = Mockito.mock(Vehicle.class);
  Mockito.when(vehicle.getSpeedMph(timeSec)).thenReturn(100d);
  double traction = 0.2;
  double speed = trafficDensity.calcSpeed(vehicle, traction, timeSec);
  assertEquals("Assert speed (traction=" + traction + ") in " 
               + timeSec + " sec: ",20,speed,0.001 *speed);
}
```

剩下的功能可以通过模拟`calcSpeed()`方法来测试：

```java
@Test
public void testCountByLane() {
  int[] count ={0};
  double[] speeds = 
                  {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12};
  TrafficDensity2 trafficDensity = new TrafficDensity2() {
    @Override
    protected double calcSpeed(Vehicle vehicle, 
                     double traction, double timeSec) {
      return speeds[count[0]++];
    }
  };
  double timeSec = 10.0;
  int trafficUnitsNumber = speeds.length;

  double[] speedLimitByLane = {4.5, 8.5, 12.5};
  Integer[] expectedCountByLane = {4, 4, 4};

  Integer[] trafficByLane = trafficDensity.trafficByLane( 
    getTrafficUnitStream(trafficUnitsNumber), 
    trafficUnitsNumber, timeSec, FactorySpeedModel.getSpeedModel(),
    speedLimitByLane );
  assertArrayEquals("Assert count of " + speeds.length 
          + " vehicles by " + speedLimitByLane.length 
          + " lanes with speed limit " 
          + Arrays.stream(speedLimitByLane)
             .mapToObj(Double::toString).collect(Collectors
             .joining(", ")), expectedCountByLane, trafficByLane);
}
```

# 还有更多...

这种经验使我们意识到，使用内部私有类可能会使功能在隔离中无法测试。让我们试着摆脱`private`类`CountByLane`。这将导致`TrafficDensity3`类的第三个版本（更改的代码已突出显示）：

```java
Integer[] trafficByLane(Stream<TrafficUnit> stream, 
int trafficUnitsNumber, double timeSec,
SpeedModel speedModel, double[] speedLimitByLane) {
  int lanesCount = speedLimitByLane.length;
  Map<Integer, Integer> trafficByLane = new HashMap<>();
  for(int i = 1; i <= lanesCount; i++){
    trafficByLane.put(i, 0);
  }
  stream.limit(trafficUnitsNumber)
    .map(TrafficUnitWrapper::new)
    .map(tuw -> tuw.setSpeedModel(speedModel))
    .map(tuw -> calcSpeed(tuw.getVehicle(), tuw.getTraction(), 
                                                         timeSec))
 .forEach(speed -> trafficByLane.computeIfPresent(
 calcLaneNumber(lanesCount, 
                         speedLimitByLane, speed), (k, v) -> ++v));    return trafficByLane.values().toArray(new Integer[lanesCount]);}
protected int calcLaneNumber(int lanesCount, 
  double[] speedLimitByLane, double speed) {
 for(int i = 1; i <= lanesCount; i++){
 if(speed <= speedLimitByLane[i - 1]){
 return i;
      }
 }
 return lanesCount;
}
```

这个改变允许我们在我们的测试中扩展这个类：

```java
class TrafficDensityTestCalcLaneNumber extends TrafficDensity3 {
  protected int calcLaneNumber(int lanesCount, 
    double[] speedLimitByLane, double speed){
    return super.calcLaneNumber(lanesCount, 
    speedLimitByLane, speed);
  }
}
```

它还允许我们单独更改`calcLaneNumber()`测试方法：

```java
@Test
public void testCalcLaneNumber() {
  double[] speeds = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12};
  double[] speedLimitByLane = {4.5, 8.5, 12.5};
  int[] expectedLaneNumber = {1, 1, 1, 1, 2, 2, 2, 2, 3, 3, 3, 3};

  TrafficDensityTestCalcLaneNumber trafficDensity = 
               new TrafficDensityTestCalcLaneNumber();
  for(int i = 0; i < speeds.length; i++){
    int ln = trafficDensity.calcLaneNumber(
               speedLimitByLane.length, 
               speedLimitByLane, speeds[i]);
    assertEquals("Assert lane number of speed " 
                + speeds + " with speed limit " 
                + Arrays.stream(speedLimitByLane)
                        .mapToObj(Double::toString).collect(
                              Collectors.joining(", ")), 
                expectedLaneNumber[i], ln);
  }
}
```

# 使用 fixtures 来为测试填充数据

在更复杂的应用程序中（例如使用数据库），通常需要在每个测试之前设置数据，并在测试完成后清理数据。一些数据的部分需要在每个测试方法之前设置和/或在每个测试方法完成后清理。其他数据可能需要在测试类的任何测试方法运行之前设置，并/或在测试类的最后一个测试方法完成后清理。

# 如何做...

为了实现这一点，您在其前面添加了一个`@Before`注释，这表示这个方法必须在每个测试方法之前运行。相应的清理方法由`@After`注释标识。类级别的设置方法由`@BeforeClass`和`@AfterClass`注释标识，这意味着这些设置方法只会在测试类的任何测试方法执行之前执行一次（`@BeforeClass`），并在测试类的最后一个测试方法执行之后执行一次（`@AfterClass`）。这是一个快速演示：

```java
public class DatabaseRelatedTest {
  @BeforeClass
  public static void setupForTheClass(){
    System.out.println("setupForTheClass() is called");
  }
  @AfterClass
  public static void cleanUpAfterTheClass(){
    System.out.println("cleanAfterClass() is called");
  }
  @Before
  public void setupForEachMethod(){
    System.out.println("setupForEachMethod() is called");
  }
  @After
  public void cleanUpAfterEachMethod(){
    System.out.println("cleanAfterEachMethod() is called");
  }
  @Test
  public void testMethodOne(){      
    System.out.println("testMethodOne() is called"); 
  }
  @Test
  public void testMethodTwo(){ 
    System.out.println("testMethodTwo() is called"); 
  }
}
```

如果现在运行测试，你会得到以下结果：

![](img/5c124906-d843-4d1d-8a75-ba87f6e17401.png)

这种修复测试上下文的方法称为**fixtures**。请注意，它们必须是公共的，类级别的设置/清理 fixtures 必须是静态的。然而，即将推出的 JUnit 版本 5 计划取消这些限制。

# 它是如何工作的...

这种用法的典型例子是在第一个测试方法运行之前创建必要的表，并在测试类的最后一个方法完成后删除它们。设置/清理方法也可以用于创建/关闭数据库连接，除非您的代码在 try-with-resources 结构中执行（参见第十一章，*内存管理和调试*）。

这是 fixtures 的一个使用示例（参见第六章，*数据库编程*，了解更多关于*如何设置数据库运行*的内容）。假设我们需要测试`DbRelatedMethods`类：

```java
class DbRelatedMethods{
  public void updateAllTextRecordsTo(String text){
    executeUpdate("update text set text = ?", text);
  }
  private void executeUpdate(String sql, String text){
    try (Connection conn = getDbConnection();
      PreparedStatement st = conn.prepareStatement(sql)){
        st.setString(1, text);
        st.executeUpdate();
      } catch (Exception ex) {
        ex.printStackTrace();
      }
    }
    private Connection getDbConnection(){
       //...  code that creates DB connection 
    }
}
```

我们希望确保前一个方法`updateAllTextRecordsTo()`总是使用提供的值更新`text`表的所有记录。我们的第一个测试`updateAllTextRecordsTo1()`是更新一个现有记录：

```java
@Test
public void updateAllTextRecordsTo1(){
  System.out.println("updateAllTextRecordsTo1() is called");
  String testString = "Whatever";
  System.out.println("  Update all records to " + testString);
  dbRelatedMethods.updateAllTextRecordsTo(testString);
  int count = countRecordsWithText(testString);
  assertEquals("Assert number of records with " 
                                  + testString + ": ", 1, count);
  System.out.println("All records are updated to " + testString);
}
```

这意味着表必须存在于测试数据库中，并且其中应该有一条记录。

我们的第二个测试，`updateAllTextRecordsTo2()`，确保即使每条记录包含不同的值，也会更新两条记录：

```java
@Test
public void updateAllTextRecordsTo2(){
  System.out.println("updateAllTextRecordsTo2() is called");
  String testString = "Unexpected";
  System.out.println("Update all records to " + testString);
  dbRelatedMethods.updateAllTextRecordsTo(testString);
  executeUpdate("insert into text(id,text) values(2, ?)","Text 01");

  testString = "Whatever";
  System.out.println("Update all records to " + testString);
  dbRelatedMethods.updateAllTextRecordsTo(testString);
  int count = countRecordsWithText(testString);
  assertEquals("Assert number of records with " 
               + testString + ": ", 2, count);
  System.out.println("  " + count + " records are updated to " +
                                                        testString);
}
```

前面的两个测试都使用了相同的表，即`text`。因此，在每次测试后无需删除表。这就是为什么我们在类级别创建和删除它的原因：

```java
@BeforeClass
public static void setupForTheClass(){
  System.out.println("setupForTheClass() is called");
  execute("create table text (id integer not null, 
          text character varying not null)");
}
@AfterClass
public static void cleanUpAfterTheClass(){
  System.out.println("cleanAfterClass() is called");
  execute("drop table text");
}
```

这意味着我们只需要在每个测试之前填充表格，并在每个测试完成后清理它：

```java
@Before
public void setupForEachMethod(){
  System.out.println("setupForEachMethod() is called");
  executeUpdate("insert into text(id, text) values(1,?)", "Text 01");
}
@After
public void cleanUpAfterEachMethod(){
  System.out.println("cleanAfterEachMethod() is called");
  execute("delete from text");
}
```

此外，由于我们可以为所有测试使用相同的对象`dbRelatedMethods`，因此让我们也在类级别上创建它（作为测试类的属性），这样它只会被创建一次：

```java
private DbRelatedMethods dbRelatedMethods = new DbRelatedMethods();

```

如果我们现在运行`test`类的所有测试，输出将如下所示：

![](img/05dd11a5-0699-4dfe-98cd-0ed6cbeb4627.png)

打印的消息可以让您跟踪所有方法调用的顺序，并查看它们是否按预期执行。

# 集成测试

如果您已经阅读了所有章节并查看了代码示例，您可能已经注意到，到目前为止，我们已经讨论并构建了典型分布式应用程序所需的所有组件。现在是将所有组件放在一起并查看它们是否按预期协作的时候了。这个过程被称为**集成**。

在这个过程中，我们将仔细评估应用程序是否符合要求。在功能需求以可执行形式呈现的情况下（例如使用 Cucumber 框架），我们可以运行它们并检查是否所有检查都通过。许多软件公司遵循行为驱动开发流程，并在很早的时候进行测试，有时甚至在编写大量代码之前（当然，这样的测试会失败，但一旦实现了预期的功能就会成功）。正如前面提到的，早期测试对于编写专注、清晰和易于测试的代码非常有帮助。

然而，即使不严格遵循“先测试”流程，集成阶段自然也包括某种行为测试。在本章中，我们将看到几种可能的方法和与此相关的具体示例。

# 准备就绪

您可能已经注意到，在本书的过程中，我们构建了几个组成应用程序的类，用于分析和建模交通。为了方便起见，我们已经将它们全部包含在`com.packt.cookbook.ch16_testing`包中。

从前面的章节中，您已经熟悉了`api`文件夹中的五个接口——`Car`、`SpeedModel`、`TrafficUnit`、`Truck`和`Vehicle`。它们的实现被封装在同名文件夹中的类中：`FactorySpeedModel`、`FactoryTraffic`和`FactoryVehicle`。这些工厂为我们的演示应用程序的核心类`AverageSpeed`（第七章，*并发和多线程编程*）和`TrafficDensity`（基于第五章，*流和管道*，但在本章中创建和讨论）提供输入。它们产生了激发开发这个特定应用程序的值。

应用程序的主要功能很简单。对于每条车道的车道数和速度限制，`AverageSpeed`计算（估计）每条车道的实际速度（假设所有驾驶员都是理性的，根据他们的速度选择车道），而`TrafficDensity`计算了 10 秒后每条车道上的车辆数（假设所有车辆在交通灯后同时开始）。这些计算是基于在特定位置和时间收集的`numberOfTrafficUnits`辆车的数据。这并不意味着所有的 1,000 辆车都在同一时间比赛。这 1,000 个测量点是在 50 年内收集的，用于在指定的交叉口在指定的小时内行驶的大约 20 辆车（平均每三分钟一辆车）。

应用程序的整体基础设施由`process`文件夹中的类支持：`Dispatcher`、`Processor`和`Subscription`。我们讨论了它们的功能，并在第七章，*并发和多线程编程*中进行了演示。这些类允许分发处理。

`Dispatcher`类向池中的`Processors`群发请求进行处理，使用`Subscription`类。每个`Processor`类根据请求执行任务（使用`AverageSpeed`和`TrafficDensity`类），并将结果存储在数据库中（使用`utils`文件夹中的`DbUtil`类，基于第六章中讨论的功能，*数据库编程*）。

我们已经将大多数这些类作为单元进行了测试。现在我们将对它们进行集成，并测试整个应用程序的正确行为。

这些要求仅用于演示目的。演示的目标是展示一些有动机的东西（类似真实数据），同时又足够简单，不需要特殊的交通分析和建模知识即可理解。

# 如何做...

集成有几个级别。我们需要集成应用程序的类和子系统，还需要将我们的应用程序与外部系统集成（由第三方开发和维护的交通数据源）。

这是使用`Chapter14Testing`类中的`demo1_class_level_integration()`方法进行类级别集成的示例：

```java
String result = IntStream.rangeClosed(1, 
  speedLimitByLane.length).mapToDouble(i -> {
    AverageSpeed averageSpeed = 
      new AverageSpeed(trafficUnitsNumber, timeSec, 
                       dateLocation, speedLimitByLane, i,100);
    ForkJoinPool commonPool = ForkJoinPool.commonPool();
    return commonPool.invoke(averageSpeed);
}).mapToObj(Double::toString).collect(Collectors.joining(", "));
System.out.println("Average speed = " + result);

TrafficDensity trafficDensity = new TrafficDensity();
Integer[] trafficByLane = 
     trafficDensity.trafficByLane(trafficUnitsNumber,
                    timeSec, dateLocation, speedLimitByLane );
System.out.println("Traffic density = "+Arrays.stream(trafficByLane)
                                .map(Object::toString)
                                .collect(Collectors.joining(", ")));

```

在这个例子中，我们集成了两个主要类，即`AverageSpeed`和`TrafficDensity`，并使用它们的接口的工厂和实现。

结果如下：

![](img/30015170-2ee5-4be1-8ace-87c4e98f8982.png)

请注意，结果在每次运行时略有不同。这是因为`FactoryTraffic`生成的数据在每个请求之间都有所不同。但是，在这个阶段，我们只需要确保一切协同工作，并产生一些看起来更或多或少准确的结果。我们已经通过单元测试了代码，并且对每个单元是否按预期工作有一定的信心。在实际集成*测试*过程中，而不是在集成过程中，我们将回到结果的验证。

在类级别完成集成后，使用`Chapter14Testing`类中的`demo1_subsystem_level_integration()`方法查看子系统如何一起工作：

```java
DbUtil.createResultTable();
Dispatcher.dispatch(trafficUnitsNumber, timeSec, dateLocation, 
                    speedLimitByLane);
try { Thread.sleep(2000L); } 
catch (InterruptedException ex) {}
Arrays.stream(Process.values()).forEach(v -> {
  System.out.println("Result " + v.name() + ": " 
                     + DbUtil.selectResult(v.name()));
});

```

在这段代码中，我们使用`DBUtil`创建了一个必要的表，用于保存`Processor`生成和记录的输入数据和结果。`Dispatcher`类向`Processor`类的对象发送请求和输入数据，如下所示：

```java
void dispatch(int trafficUnitsNumber, double timeSec, 
         DateLocation dateLocation, double[] speedLimitByLane) {
  ExecutorService execService =  ForkJoinPool.commonPool();
  try (SubmissionPublisher<Integer> publisher = 
                              new SubmissionPublisher<>()){
    subscribe(publisher, execService,Process.AVERAGE_SPEED, 
              timeSec, dateLocation, speedLimitByLane);
   subscribe(publisher,execService,Process.TRAFFIC_DENSITY, 
             timeSec, dateLocation, speedLimitByLane);
    publisher.submit(trafficUnitsNumber);
  } finally {
    try {
      execService.shutdown();
      execService.awaitTermination(1, TimeUnit.SECONDS);
    } catch (Exception ex) {
      System.out.println(ex.getClass().getName());
    } finally {
      execService.shutdownNow();
    }
  }
}
```

`Subscription`类用于发送/接收消息（参考第七章，*并发和多线程编程*，了解此功能的描述）：

```java
void subscribe(SubmissionPublisher<Integer> publisher, 
              ExecutorService execService, Process process, 
              double timeSec, DateLocation dateLocation, 
              double[] speedLimitByLane) {
  Processor<Integer> subscriber =  new Processor<>(process, timeSec, 
                                 dateLocation, speedLimitByLane);
  Subscription subscription = 
                       new Subscription(subscriber, execService);
  subscriber.onSubscribe(subscription);
  publisher.subscribe(subscriber);
}
```

处理器正在执行它们的工作；我们只需要等待几秒钟（如果您使用的计算机需要更多时间来完成工作，可以调整此时间）然后我们就可以得到结果。我们使用`DBUtil`从数据库中读取结果：

![](img/e2e9ca71-45f0-42b1-955e-071d82ab264d.png)

`Process`枚举类的名称指向数据库中`result`表中的相应记录。同样，在这个阶段，我们主要是希望得到任何结果，而不是关注值的正确性。

在基于`FactoryTraffic`生成的数据的应用程序子系统之间成功集成后，我们可以尝试连接提供真实交通数据的外部系统。在`FactoryTraffic`中，我们现在将从生成`TrafficUnit`对象切换到从真实系统获取数据：

```java
public class FactoryTraffic {
  private static boolean switchToRealData = true;
  public static Stream<TrafficUnit> 
  getTrafficUnitStream(DateLocation dl, int trafficUnitsNumber){
    if(switchToRealData){
      return getRealData(dL,  trafficUnitsNumber);
    } else {
      return IntStream.range(0, trafficUnitsNumber)
      .mapToObj(i -> generateOneUnit());
    }
  }

  private static Stream<TrafficUnit> 
  getRealData(DateLocation dl, int trafficUnitsNumber) {
    //connect to the source of the real data 
    // and request the flow or collection of data
    return new ArrayList<TrafficUnit>().stream();
  }
}
```

该开关可以作为类中的`Boolean`属性实现（如前面的代码所示），也可以作为项目配置属性。我们不会详细介绍连接到特定真实交通数据源的细节，因为这与本书的目的无关。

在这个阶段，主要关注性能，并在外部真实数据源和我们的应用程序之间实现平稳的数据流。在确保一切正常并且具有令人满意的性能的情况下，我们可以转向集成*测试*，并断言实际结果。

# 它是如何工作的...

对于测试，我们需要设置预期值，然后与处理真实数据的应用程序产生的实际值进行比较。但是真实数据在每次运行时都会略有变化，试图预测结果值要么使测试变得脆弱，要么迫使引入巨大的误差范围，这可能会有效地破坏测试的目的。

我们甚至不能模拟生成的数据（就像我们在单元测试中所做的那样），因为我们处于集成阶段，必须使用真实数据。

有一个可能的解决方案是将传入的真实数据和我们应用程序生成的结果存储在数据库中。然后，领域专家可以浏览每条记录，并断言结果是否符合预期。

为了实现这一点，我们在`TrafficDensity`类中引入了一个`boolean`开关，这样它就记录了每个计算单元的输入：

```java
public class TrafficDensity {
 public static Connection conn;
 public static boolean recordData = false;
  //... 
  private double calcSpeed(TrafficUnitWrapper tuw, double timeSec){
    double speed = calcSpeed(tuw.getVehicle(),       
    tuw.getTrafficUnit().getTraction(), timeSec);
 if(recordData) {
 DbUtil.recordData(conn, tuw.getTrafficUnit(), speed);
 }
    return speed;
  }
  //...
} 
```

我们还引入了一个静态属性，以保持所有类实例之间相同的数据库连接。否则，连接池应该很大，因为正如你可能从第七章中所记得的那样，*并发和多线程编程*，执行任务的工作人员数量随着要执行的工作量的增加而增加。

如果你看看`DbUtils`，你会看到一个创建`data`表的新方法，该表设计用于保存来自`FactoryTraffic`的`TrafficUnits`，以及保存用于数据请求和计算的主要参数的`data_common`表：请求的交通单位数量，交通数据的日期和地理位置，以秒为单位的时间（速度计算的时间点），以及每条车道的速度限制（其大小定义了我们在建模交通时计划使用多少条车道）。这是我们配置来进行记录的代码：

```java
private static void demo3_prepare_for_integration_testing(){
  DbUtil.createResultTable();
  DbUtil.createDataTables();
  TrafficDensity.recordData = true;
  try(Connection conn = DbUtil.getDbConnection()){
    TrafficDensity.conn = conn;
    Dispatcher.dispatch(trafficUnitsNumber, timeSec, 
                        dateLocation, speedLimitByLane);
  } catch (SQLException ex){
    ex.printStackTrace();
  }
}
```

记录完成后，我们可以将数据交给领域专家，他可以断言应用程序行为的正确性。

验证的数据现在可以用于集成测试。我们可以在`FactoryTrafficUnit`中添加另一个开关，并强制它读取记录的数据，而不是不可预测的真实数据：

```java
public class FactoryTraffic {
  public static boolean readDataFromDb = false;
  private static boolean switchToRealData = false;
  public static Stream<TrafficUnit> 
     getTrafficUnitStream(DateLocation dl, int trafficUnitsNumber){
 if(readDataFromDb){
 if(!DbUtil.isEnoughData(trafficUnitsNumber)){
 System.out.println("Not enough data");
        return new ArrayList<TrafficUnit>().stream();
      }
 return readDataFromDb(trafficUnitsNumber);
    }
    //....
}
```

正如你可能已经注意到的，我们还添加了`isEnoughData()`方法，用于检查是否有足够的记录数据：

```java
public static boolean isEnoughData(int trafficUnitsNumber){
  try (Connection conn = getDbConnection();
  PreparedStatement st = 
      conn.prepareStatement("select count(*) from data")){
    ResultSet rs = st.executeQuery();
    if(rs.next()){
      int count = rs.getInt(1);
      return count >= trafficUnitsNumber;
    }
  } catch (Exception ex) {
    ex.printStackTrace();
  }
  return false;
}
```

这将有助于避免在测试更复杂的系统时不必要的调试问题所带来的挫败感。

现在，我们不仅控制输入值，还可以控制预期结果，这些结果可以用来断言应用程序的行为。这两者现在都包含在`TrafficUnit`对象中。为了能够做到这一点，我们利用了第二章中讨论的新的 Java 接口特性，即接口默认方法：

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
 default double getSpeed(){ return 0.0; }
}
```

因此，我们可以将结果附加到输入数据。请参阅以下方法：

```java
List<TrafficUnit> selectData(int trafficUnitsNumber){...}
```

我们可以将结果附加到`DbUtil`类和`TrafficUnitImpl`类中的`DbUtil`中：

```java
class TrafficUnitImpl implements TrafficUnit{
  private int horsePower, weightPounds, payloadPounds, 
              passengersCount, temperature;
  private Vehicle.VehicleType vehicleType;
  private double speedLimitMph, traction, speed;
  private RoadCondition roadCondition;
  private TireCondition tireCondition;
  ...
  public double getSpeed() { return speed; }
}
```

我们也可以将其附加到`DbUtil`类中。

前面的更改使我们能够编写集成测试。首先，我们将使用记录的数据测试速度模型：

```java
void demo1_test_speed_model_with_real_data(){
  double timeSec = DbUtil.getTimeSecFromDataCommon();
  FactoryTraffic.readDataFromDb = true;
  TrafficDensity trafficDensity = new TrafficDensity();
  FactoryTraffic.
           getTrafficUnitStream(dateLocation,1000).forEach(tu -> {
    Vehicle vehicle = FactoryVehicle.build(tu);
    vehicle.setSpeedModel(FactorySpeedModel.getSpeedModel());
    double speed = trafficDensity.calcSpeed(vehicle, 
                               tu.getTraction(), timeSec);
    assertEquals("Assert vehicle (" + tu.getHorsePower() 
                 + " hp, " + tu.getWeightPounds() + " lb) speed in " 
                 + timeSec + " sec: ", tu.getSpeed(), speed, 
                 speed * 0.001);
  });
}
```

可以使用类似的方法来测试`AverageSpeed`类的速度计算。

然后，我们可以为类级别编写一个集成测试。

```java
private static void demo2_class_level_integration_test() {
  FactoryTraffic.readDataFromDb = true;
  String result = IntStream.rangeClosed(1, 
              speedLimitByLane.length).mapToDouble(i -> {
    AverageSpeed averageSpeed = new AverageSpeed(trafficUnitsNumber, 
               timeSec, dateLocation, speedLimitByLane, i,100);
    ForkJoinPool commonPool = ForkJoinPool.commonPool();
    return commonPool.invoke(averageSpeed);
  }).mapToObj(Double::toString).collect(Collectors.joining(", "));
  String expectedResult = "7.0, 23.0, 41.0";
  String limits = Arrays.stream(speedLimitByLane)
                        .mapToObj(Double::toString)
                        .collect(Collectors.joining(", "));
  assertEquals("Assert average speeds by " 
                + speedLimitByLane.length 
                + " lanes with speed limit " 
                + limits, expectedResult, result);

```

类似的代码也可以用于对 TrafficDensity 类进行类级别的测试：

```java
TrafficDensity trafficDensity = new TrafficDensity();
String result = Arrays.stream(trafficDensity.
       trafficByLane(trafficUnitsNumber, timeSec, 
                     dateLocation, speedLimitByLane))
       .map(Object::toString)
       .collect(Collectors.joining(", "));
expectedResult = "354, 335, 311";
assertEquals("Assert vehicle count by " + speedLimitByLane.length + 
         " lanes with speed limit " + limits, expectedResult, result);
```

最后，我们也可以为子系统级别编写集成测试：

```java
void demo3_subsystem_level_integration_test() {
  FactoryTraffic.readDataFromDb = true;
  DbUtil.createResultTable();
  Dispatcher.dispatch(trafficUnitsNumber, 10, dateLocation, 
                      speedLimitByLane);
  try { Thread.sleep(3000l); } 
  catch (InterruptedException ex) {}
  String result = DbUtil.selectResult(Process.AVERAGE_SPEED.name());
  String expectedResult = "7.0, 23.0, 41.0";
  String limits = Arrays.stream(speedLimitByLane)
                        .mapToObj(Double::toString)
                        .collect(Collectors.joining(", "));
  assertEquals("Assert average speeds by " + speedLimitByLane.length 
        + " lanes with speed limit " + limits, expectedResult, result);
  result = DbUtil.selectResult(Process.TRAFFIC_DENSITY.name());
  expectedResult = "354, 335, 311";
  assertEquals("Assert vehicle count by " + speedLimitByLane.length 
        + " lanes with speed limit " + limits, expectedResult, result);
}
```

所有前面的测试现在都成功运行，并且随时可以用于应用程序的回归测试。

只有当后者具有测试模式时，我们才能创建应用程序与真实交通数据源之间的自动集成测试，从而可以以与使用记录数据相同的方式发送相同的数据流。

所有这些集成测试都是可能的，当处理数据的数量在统计上是显著的时候。这是因为我们无法完全控制工作人员的数量以及 JVM 如何决定分配负载。很可能，在特定情况下，本章中演示的代码可能无法正常工作。在这种情况下，尝试增加请求的流量单位数量。这将确保更多的空间用于负载分配逻辑。
