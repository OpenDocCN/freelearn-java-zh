# 第七章：接口，类和对象构造

本章向读者解释了 Java 编程的最重要方面：应用程序编程接口（API），对象工厂，方法重写，隐藏和重载。接着是聚合（而不是继承）的设计优势的解释，开始讨论软件系统设计。本章最后概述了 Java 数据结构。

在本章中，我们将涵盖以下主题：

+   什么是 API？

+   接口和对象工厂作为 API

+   重写，隐藏和重载

+   `this`和`super`关键字

+   构造函数和构造函数重载

+   最终变量，最终方法和最终类

+   对象关联（聚合）

+   练习-将类实例化限制为单个共享实例

# API 是什么？

术语**应用程序编程接口**（API）是程序员用来实现所需功能的协议，程序和服务的规范。API 可以代表基于 Web 的系统，操作系统，数据库系统，计算机硬件或软件库。

除此之外，在日常生活中，术语 API 经常用于实现规范的系统。例如，您可能熟悉 Twitter APIs（[`developer.twitter.com/en/docs`](https://developer.twitter.com/en/docs)）或 Amazon APIs（[`developer.amazon.com/services-and-apis`](https://developer.amazon.com/services-and-apis)），或者您可能已经使用能够通过提供数据（测量结果）来响应请求的设备（传感器）。因此，当程序员说*我们可以使用 Amazon API*时，他们不仅指提供的程序描述，还指服务本身。

在 Java 中，我们还有一些关于*API 使用*的术语变体，我们希望在以下小节中进行识别和描述。

# Java API

Java API 包括两大类 API 和实现它们的库：

+   Java 核心包（[`www.oracle.com/technetwork/java/api-141528.html`](http://www.oracle.com/technetwork/java/api-141528.html)）随 Java 安装提供并包含在 JDK 中

+   其他可以单独下载的框架和库，例如 Apache Commons APIs（[`commons.apache.org`](https://commons.apache.org)），或者我们已经在 Maven 的`pom.xml`文件中包含为依赖项的三个库。其中绝大多数可以在 Maven 仓库（[`mvnrepository.com`](https://mvnrepository.com)）中找到，但也可以在其他地方找到各种新的和实验性的库和框架。

# 命令行 API

命令行 API 描述了命令格式及其可能的选项，可用于执行应用程序（工具）。我们在第一章中讨论使用`java`和`javac`工具（应用程序）时看到了这样的例子，*您的计算机上的 Java 虚拟机（JVM）*。我们甚至在第四章中构建了自己的应用程序，定义了其 API，并描述了其命令行 API，接受整数作为参数。

# 基于 HTTP 的 API

基于 Web 的应用程序通常使用各种协议（[`en.wikipedia.org/wiki/List_of_web_service_protocols`](https://en.wikipedia.org/wiki/List_of_web_service_protocols)）提供基于 HTTP 的 API，允许通过互联网访问应用程序功能。HTTP 代表超文本传输协议，是分布式信息系统的应用协议，是**万维网**（**WWW**）数据通信的基础。

最流行的两种 Web 服务协议是：

+   基于 XML 的**SOAP**（Simple Object Access Protocol）协议

+   基于 JSON 的 REST 或 RESTful（**REpresentational State Transfer**）风格的 HTTP 协议

两者都描述了如何访问功能（服务）并将其合并到应用程序中。我们在本书中不描述 Web 服务。

# 软件组件 API

软件组件可以是一个库，一个应用子系统，一个应用层，甚至是一个单独的类——可以通过调用其方法直接从 Java 代码中使用的东西。软件组件的 API 看起来像描述方法签名的接口，可以在实现接口的类的对象上调用这些方法。如果组件有公共静态方法（不需要对象，只能使用类调用），这些方法也必须包含在 API 描述中。但是，对于组件 API 的完整描述，正如我们在第二章中已经提到的那样，关于如何创建组件的对象的信息也应该是 API 描述的一部分。

在本书中，我们不会超越应用程序边界，并且只会在先前描述的软件组件 API 的意义上使用术语 API。而且，我们将按其名称称呼实现 API 的实体（API 描述的服务）：应用子系统，应用层，库，类，接口和方法。

这就是为什么我们开始了一个关于接口和对象工厂的 API 相关讨论，它们相互补充，并且与静态方法一起组成了软件组件 API 的完整描述。

# 接口和对象工厂作为 API

名词抽象意味着书籍、文章或正式演讲的内容摘要。形容词抽象意味着存在于思想中或作为一个想法，但没有具体的或实体的存在。动词抽象意味着从理论上或与其他事物分开考虑（某事）。

这就是为什么接口被称为抽象——因为它只捕捉方法签名，不描述如何实现结果。相同接口的各种实现——不同的类——可能行为完全不同，即使它们接收相同的参数并返回相同的结果。最后一句是一个有深意的陈述，因为我们还没有定义行为这个术语。现在让我们来做。

类或其对象的行为由其方法执行的操作和它们返回的结果定义。如果一个方法不返回任何东西（`void`），则称这样的方法仅用于其副作用。

这种观点意味着返回值的方法具有直接（而不是副作用）的效果。然而，它也可能具有副作用，例如向另一个应用程序发送消息，或者在数据库中存储数据。理想情况下，应该尝试在方法名称中捕捉副作用。如果这不容易，因为方法做了很多事情，这可能表明需要将这样的方法分解为几个更好聚焦的方法。

同一方法签名的两个实现可能具有不同的行为的说法只有在方法名称没有捕捉到所有副作用，或者实现的作者没有遵守方法名称的含义时才有意义。但即使不同实现的行为相同，代码本身、它使用的库以及其有效性可能是不同的。

为什么隐藏实现细节很重要，我们将在第八章中解释，*面向对象设计（OOD）原则*。现在，我们只是提到客户端与实现的隔离允许系统更灵活地采用相同实现的新版本或完全切换到另一个实现。

# 接口

我们在第二章中讨论了接口，现在我们只看一些例子。让我们创建一个新的包，`com.packt.javapath.ch06demo.api`。然后，我们可以右键单击`com.packt.javapath.ch06demo.api`，打开 New | Java Class，选择 Interface，输入`Calculator`，然后单击 OK 按钮。我们已经创建了一个接口，并且可以向其添加一个方法签名，`int multiplyByTwo(int i)`，结果如下：

![](img/3bf0e6a7-cecc-4cfe-bc2b-3337a3a24d78.png)

这将是实现此接口的每个类的公共接口。在现实生活中，我们不会使用包名称`api`，而是使用`calculator`，因为它更具体和描述性。但是我们正在讨论术语“API”，这就是我们决定以这种方式命名包的原因。

让我们创建另一个包，`com.packt.javapath.ch06demo.api.impl`，其中将保存所有`Calculator`的实现和我们将添加到`com.packt.javapath.ch06demo.api`包中的其他接口。第一个实现是`CalulatorImpl`类。到目前为止，您应该已经知道如何在其中创建`com.packt.javapath.ch06demo.api.impl`包和`CalulatorImpl`类。结果应该如下所示：

![](img/d6e713bb-a6b6-40d0-8fe0-7be774bf7f0d.png)

我们将实现放在了比`api`更深一级的包中，这表明这些细节不应该暴露给我们创建的 API 的用户。

此外，我们需要编写一个测试并使用它来确保我们的功能对用户来说是正确和方便的。同样，我们假设您现在知道如何做到这一点。结果应该如下所示：

![](img/8199defb-2557-47f7-97e8-35febe1cb351.png)

然后，我们添加缺失的测试主体和注释，如下所示：

```java
@DisplayName("API Calculator tests")
public class CalculatorTest {
  @Test
  @DisplayName("Happy multiplyByTwo()")
  void multiplyByTwo(){
    CalculatorImpl calculator = new CalculatorImpl();
    int i = 2;
    int result = calculator.multiplyByTwo(i);
    assertEquals(4, result);
  }
}
```

这段代码不仅作为功能测试，还可以被视为 API 用户编写的客户端代码的示例。因此，测试帮助我们从客户端的角度看待我们的 API。通过观察这段代码，我们意识到我们无法完全隐藏实现细节。即使我们将创建对象的行更改为以下内容：

```java
Calculator calculator = new CalculatorImpl();

```

这意味着，如果我们更改`CalculatorImpl`构造函数的签名或切换到同一接口的另一个实现（称为`AnotherCalculatorImpl`），客户端代码也必须更改。为了避免这种情况，程序员使用称为对象工厂的类。

# Object factory

对象工厂的目的是隐藏对象创建的细节，以便客户端在实现更改时无需更改代码。让我们创建一个生产`Calculator`对象的工厂。我们将把它放在与`Calculator`接口的实现位于同一包`com.packt.javapath.ch06demo.api.impl`中：

![](img/525f9e16-73db-40de-ae66-c526bd1ab207.png)

我们可以更改测试（客户端代码）以使用此工厂：

```java
@DisplayName("API Calculator tests")
public class CalculatorTest {
  @Test
  @DisplayName("Happy multiplyByTwo()")
  void multiplyByTwo(){
    Calculator calculator = CalculatorFactory.createInstance();
    int i = 2;
    int result = calculator.multiplyByTwo(i);
    assertEquals(4, result);
  }
}
```

通过这样做，我们已经实现了我们的目标：客户端代码不会对实现`Calculator`接口的类有任何概念。例如，我们可以更改工厂，以便它创建另一个类的对象：

```java
public static Calculator create(){
  return AnotherCalculatorImpl();
}
```

`AnotherCalculatorImpl`类可能如下所示：

```java
class AnotherCalculatorImpl  implements Calculator {
  public int multiplyByTwo(int i){
    System.out.println(AnotherCalculatorImpl.class.getName());
    return i + i;
  }
}
```

这个`multiplyByTwo()`方法是将两个值相加，而不是将输入参数乘以 2。

我们还可以使工厂读取配置文件，并根据配置文件的值实例化实现：

```java
public class CalculatorFactory {
  public static Calculator create(){
    String whichImpl = 
       Utils.getStringValueFromConfig("calculator.conf", "which.impl");
    if(whichImpl.equals("multiplies")){
      return new CalculatorImpl();
    } else if (whichImpl.equals("adds")){
      return new AnotherCalculatorImpl();
    } else {
      throw new RuntimeException("Houston, we have a problem. " +
        "Unknown key which.impl value " + whichImpl + " is in config.");
    } 
  }     
}
```

我们还没有讨论`if...else`结构或`RuntimeException`类（参见第十章，*控制流语句*）。我们很快会讨论`Utils.getStringValueFromConfig()`方法。但是，我们希望你理解这段代码的作用：

+   读取配置文件

+   根据`which.impl`键的值实例化类

+   如果没有与`which.impl`键的值对应的类，则通过抛出异常退出方法（因此通知客户端存在必须解决的问题）

这是配置文件`calculator.conf`可能的样子：

```java
{
  "which.impl": "multiplies"
}
```

这称为**JavaScript 对象表示**（**JSON**）格式，它基于由冒号（`:`）分隔的键值对。您可以在[`www.json.org/`](http://www.json.org/)上了解更多关于 JSON 的信息。

`calculator.conf`文件位于`resources`目录（`main`目录的子目录）中。默认情况下，Maven 将此目录的内容放在类路径上，因此应用程序可以找到它。

要告诉工厂使用另一个`Calculator`实现，我们只需要做以下事情：

+   更改文件`calculator.conf`中键`which.impl`的值

+   更改工厂的`create()`方法以根据这个新值实例化新的实现

重要的是要注意，当我们切换`Calculator`实现时，客户端代码（`CalculatorTest`类）不受影响。这是使用接口和对象工厂类隐藏实现细节对客户端代码的优势。

现在，让我们看看`Utils`类及其`getStringValueFromConfig()`方法的内部。

# 读取配置文件

通过查看`getStringValueFromConfig()`方法的真实实现，我们超前于你对 Java 和 Java 库的了解。因此，我们不希望你理解所有的细节，但我们希望这种暴露会让你了解事情是如何做的，我们的课程目标是什么。

# 使用 json-simple 库

`getStringValueFromConfig()`方法位于`Utils`类中，我们已经创建了这个类来从`.conf`文件中读取值。这个类有以下代码：

```java
import org.json.simple.JSONObject;
import org.json.simple.parser.JSONParser;
import org.json.simple.parser.ParseException;

public class Utils {
  private static JSONObject config = null;
  public static String getStringValueFromConfig(String configFileName, 
                                                            String key){
    if(config == null){
      ClassLoader classLoader = Utils.class.getClassLoader();
      File file =
           new File(classLoader.getResource(configFileName).getFile());
      try(FileReader fr = new FileReader(file)){
        JSONParser parser = new JSONParser();
        config = (JSONObject) parser.parse(fr);
      } catch (ParseException | IOException ex){
        ex.printStackTrace();
        return "Problem reading config file.";
      }
    }
    return config.get(key) == null ? "unknown" : (String)config.get(key);
  }
}
```

首先，请注意称为缓存的技术。我们首先检查`config`静态类字段的值。如果它不是`null`，我们就使用它。否则，我们使用相同的类加载器在类路径上找到`config`文件，该类加载器用于加载我们传递的已知类。我们解析配置文件，这意味着将其分解为键值对。结果是我们分配给`config`字段的`JSONObject`类的生成对象的引用（缓存它，以便下次可以使用）。

这是缓存技术，用于避免浪费时间和其他资源。这种解决方案的缺点是，对配置文件的任何更改都需要重新启动应用程序，以便重新读取文件。在我们的情况下，我们假设这是可以接受的。但在其他情况下，我们可以添加一个定时器，并在定义的时间段过后刷新缓存数据，或者做类似的事情。

为了读取配置文件，我们使用 Apache Commons 库中的`FileReader`类（[`commons.apache.org/proper/commons-io`](https://commons.apache.org/proper/commons-io)）。为了让 Maven 知道我们需要这个库，我们已经将以下依赖项添加到`pom.xml`文件中：

```java
<dependency>
  <groupId>commons-io</groupId>
  <artifactId>commons-io</artifactId>
  <version>2.5</version>
</dependency>

```

要处理 JSON 格式的数据，我们使用 JSON.simple 库（也是根据 Apache 许可发布的），并将以下依赖项添加到`pom.xml`中：

```java
<dependency>
  <groupId>com.googlecode.json-simple</groupId>
  <artifactId>json-simple</artifactId>
  <version>1.1</version>
</dependency>

```

`JSONObject`类以 JSON 格式存储键值对。如果传入的键在文件中不存在，`JSONObject`类的对象返回值为`null`。在这种情况下，我们的`getStringValueFromConfig()`方法返回一个`String`字面量 unknown。否则，它将返回值转换为`String`。我们可以这样做，因为我们知道该值可以赋给`String`类型的变量。

`<condition>? <option1> : <option2>`构造被称为三元运算符。当条件为真时，它返回`option1`，否则返回`option2`。我们将在第九章中更多地讨论它，*运算符、表达式和语句*。

# 使用 json-api 库

或者，我们可以使用另一个 JSON 处理 API 及其实现：

```java
<dependency>
  <groupId>javax.json</groupId>
  <artifactId>javax.json-api</artifactId>
  <version>1.1.2</version>
</dependency>
<dependency>
  <groupId>org.glassfish</groupId>
  <artifactId>javax.json</artifactId>
  <version>1.1.2</version>
</dependency>

```

然后`getStringValueFromConfig()`方法的代码看起来会有些不同：

```java
import javax.json.Json;
import javax.json.JsonObject;
import javax.json.JsonReader;
public class Utils {
  private static JsonObject config = null;
  public static String getStringValueFromConfig(String FileName, 
                                                           String key){
    if(config == null){
      ClassLoader classLoader = Utils.class.getClassLoader();
      File file = new File(classLoader.getResource(fileName).getFile());
      try(FileInputStream fis = new FileInputStream(file)){
        JsonReader reader = Json.createReader(fis);
        config = reader.readObject();
      } catch (IOException ex){
        ex.printStackTrace();
        return "Problem reading config file.";
      }
    }
    return config.get(key) == null ? "unknown" : config.getString(key);
  }
}
```

这个第二个实现需要的代码稍微少一些，并且使用了更一致的驼峰命名风格（`JsonObject`与`JSONObject`）。但是，由于它们的性能并没有太大的不同，使用哪个库在很大程度上取决于个人偏好。

# 单元测试

让我们创建一个单元测试，证明该方法按预期工作。到目前为止，你应该能够在`test/java/com/packt/javapath/ch06demo`目录（或在 Windows 的`test\java\com\packt\javapath\ch06demo`目录）中创建一个`UtilsTest`类。测试应该如下所示:

```java
@DisplayName("Utils tests")
public class UtilsTest {
  @Test
  @DisplayName("Test reading value from config file by key")
  void getStringValueFromConfig(){
    //test body we will write here
  }
}
```

接下来，我们添加`test/resources/utilstest.conf`文件（对于 Windows 是`test\resources\utilstest.conf`）:

```java
{
  "unknown": "some value"
}
```

它将扮演`config`文件的角色。有了这个，测试代码看起来如下:

```java
@Test
@DisplayName("Test reading value from config file by key")
void getStringValueFromConfig(){
  String fileName = "utilstest.conf";
  String value = Utils.getStringValueFromConfig(fileName, "some value");
  assertEquals("some value", value);

  value = Utils.getStringValueFromConfig(fileName, "some value");
  assertEquals("unknown", value);
}
```

我们测试两种情况:

+   返回的值应该在第一种情况下等于`some value`

+   如果在配置文件中键不存在，则值应该返回为`unknown`

我们运行这个测试并观察成功。为了确保，我们还可以将`utilstest.conf`文件的设置更改为以下内容:

```java
{
  "unknown": "another value"
}
```

这应该导致测试在第一种情况下失败。

让我们重新审视一下 Calculator API。

# 计算器 API

根据前面的讨论，我们可以在`Calculator`接口中描述 Calculator API 如下:

```java
public interface Calculator {
  int multiplyByTwo(int i);
}
static Calculator createInstance(){
  return CalculatorFactory.create();
}
```

如果`Calculator`实现的构造函数需要参数，我们将把它们添加到接口的`create()`工厂方法和`createInstance()`静态方法中。

当`Calculator`接口只存在一个实现时，前面的 API 声明就足够了。但是当你给客户端提供两个或更多的实现选择时，就像我们之前描述的那样，API 还应该包括`calculator.conf`配置文件的描述。

`配置描述`将不得不列出`which.impl`键的所有可能值（在我们的例子中是`multiplies`和`adds`）。我们还需要解释实现之间的差异，以便使用我们的计算器的程序员能够做出知情的选择。

如果这听起来太多了，那么你可能需要退一步重新审视你的 API 设计，因为它可能没有很好地聚焦，试图涵盖太多东西。考虑将这样的 API 分解为几个更简单的 API。描述每个较小的 API 更容易编写和理解。

例如，这是如何在我们的情况下将配置描述添加到接口中的:

```java
public interface Calculator {
  int multiplyByTwo(int i);
  static Calculator createInstance(){
    return  CalculatorFactory.create();
  }
  String CONF_NAME = "calculator.conf";
  String CONF_WHICH_IMPL = "which.impl";
  enum WhichImpl{
    multiplies, //use multiplication operation
    adds        //use addition operation
  }
}
```

正如你所看到的，我们在常量中捕获了配置文件名，以及配置键名。我们还为键的所有可能值创建了一个`enum`。我们还添加了实现之间差异的解释作为注释。如果解释太长，注释可以提供对文档、网站名称或 URL 的引用，例如。

Since there are two implementations and two possible values in the configuration file, we need to run our unit test `CalculatorTest` twice—for each possible value of the configuration—to make sure that both implementations work as expected. But we do not want to change the configuration inside the deliverable software component itself.

That is when the `test/resources` directory (`test\resources` for Windows) comes into play again. Let's create a `calculator.conf` file in it and add the following lines to the `CalculatorTest` test, which will print the current settings in that file:

```java
String whichImpl = 
   Utils.getStringValueFromConfig(Calculator.CONF_NAME, 
                                     Calculator.CONF_WHICH_IMPL);
System.out.println(Calculator.CONF_WHICH_IMPL + "=" + whichImpl);

```

The `CalculatorTest` code should look as follows:

```java
void multiplyByTwo() {
  WhichImpl whichImpl = 
      Utils.getWhichImplValueFromConfig(Calculator.CONF_NAME, 
                                        Calculator.CONF_WHICH_IMPL);
  System.out.println("\n" + Calculator.CONF_WHICH_IMPL + 
                                                   "=" + whichImpl);
  Calculator calculator = Calculator.createInstance();
  int i = 2;
  int result = calculator.multiplyByTwo(i);
  assertEquals(4, result);
}
```

Let's also add a line that prints out the class name of each implementation:

```java
public class CalculatorImpl implements Calculator {
  public int multiplyByTwo(int i){
    System.out.println(CalculatorImpl.class.getClass().getName());
    return i * 2;
  }
}
public class AnotherCalculatorImpl implements Calculator {
  public int multiplyByTwo(int i){
    System.out.println(AnotherCalculatorImpl.class.getClass().getName());
    return i + i;
 }
}
```

If we set the value of `which.impl` (in the `calculator.conf` file in the `test` directory) to `adds`, it will look like this:

![](img/a123ab51-0369-4fe3-ac54-a73a829b2d6a.png)

And the result of the `CalculatorTest` test will be:

![](img/4af08c61-2654-40bc-89c4-a10f08681e58.png)

The output tells us three things:

+   The value of `which.impl` in `calculator.conf` was set to `adds`

+   The corresponding implementation of `AnotherCalculatorImpl` was used

+   The invoked implementation worked as expected

Similarly, we can run our unit test for the `calculator.conf` file set to `multiplies`.

The result looks very good, but we still can improve the code and make it less susceptible to error, if sometime in the future somebody decides to enhance the functionality by adding a new implementation or something similar. We can take advantage of the constants added to the `Calculator` interface and make the `create()`  factory method more protected from a human mistake:

```java
public static Calculator create(){
  String whichImpl = Utils.getStringValueFromConfig(Calculator.CONF_NAME, 
                                       Calculator.CONF_WHICH_IMPL);         
  if(whichImpl.equals(Calculator.WhichImpl.multiplies.name())){
    return new CalculatorImpl();
  } else if (whichImpl.equals(Calculator.WhichImpl.adds.name())){
    return new AnotherCalculatorImpl();
  } else {
    throw new RuntimeException("Houston, we have a problem. " +
                     "Unknown key " + Calculator.CONF_WHICH_IMPL +
                     " value " + whichImpl + " is in config.");
  }
}
```

Just to make sure that the test doing its job, we change the value in the `calculator.conf` file in the test directory to `add` (instead of `adds`) and run the test again. The output will be as follows:

![](img/252fb2df-1745-4fa7-8b42-92fae0a4f19d.png)

The test failed, as was expected. It gives us a level of confidence that the code works and doesn't just always show success.

Yet, the code can be improved to become more readable, more testable, and less susceptible to human errors when it is modified or expanded. Using the knowledge of the `enum` functionality, we can write a method that converts the value of the key `which.impl` in the `calculator.conf` file to one of the constants (instances) of the class `enum WhichImpl`. To do it, we add this new method to the class `Utils`:

```java
WhichImpl getWhichImplValueFromConfig(String configFileName, String key){
  String whichImpl = getStringValueFromConfig(configFileName, key);
  try{
    return Enum.valueOf(WhichImpl.class, whichImpl);
  } catch (IllegalArgumentException ex){
    throw new RuntimeException("Houston, we have a problem. " +
                     "Unknown key " + Calculator.CONF_WHICH_IMPL +
                     " value " + whichImpl + " is in config.");
  }
}
```

这段代码基于`getStringValueFromConfig()`方法的使用，我们已经测试过并知道它按预期工作。`try...catch`结构允许我们捕获和处理一些代码（在这种情况下是`Enum.valueOf()`方法）遇到无法解决的条件并抛出异常的情况（我们将在第十章中学到更多关于这个的知识，*控制流语句*）。人们必须阅读 Java API 文档，才能知道`Enum.valueOf()`方法可能会抛出异常。例如，这是关于`Enum.valueOf()`方法的文档中的一句引用：

"Throws: IllegalArgumentException - 如果指定的枚举类型没有具有指定名称的常量，或者指定的类对象不表示枚举类型"

阅读即将使用的任何第三方类的 API 文档是一个好主意。在我们的代码中，我们捕获它并以一致的方式用我们自己的措辞抛出一个新的异常。

正如你所期望的，我们还为`getWhichImplValueFromConfig()`方法编写了一个单元测试，并将其添加到`UtilsTest`中：

```java
@Test
@DisplayName("Test matching config value to enum WhichImpl")
void getWhichImpValueFromConfig(){
  String confifFileName = "utilstest.conf";
  for(int i = 1; i <= WhichImpl.values().length; i++){
    String key = String.valueOf(i);
    WhichImpl whichImpl = 
       Utils.getWhichImplValueFromConfig(confifFileName, key);
    System.out.println(key + "=" + whichImpl);
  }
  try {
    WhichImpl whichImpl = 
       Utils.getWhichImplValueFromConfig(confifFileName, "unknown");
    fail("Should not get here! whichImpl = " + whichImpl);
  } catch (RuntimeException ex){
    assertEquals("Houston, we have a problem. " +
                 "Unknown key which.impl value unknown is in config.", 
                 ex.getMessage());
  }
  try {
    WhichImpl whichImpl = 
       Utils.getWhichImplValueFromConfig(confifFileName, "some value");
    fail("Should not get here! whichImpl = " + whichImpl);
  } catch (RuntimeException ex){
    assertEquals("Houston, we have a problem. " +
                 "Unknown key which.impl value unknown is in config.", 
                 ex.getMessage());
  }
}
```

为了支持这个测试，我们还在`utilstest.conf`文件中添加了两个条目：

```java
{
  "1": "multiplies",
  "2": "adds",
  "unknown": "unknown"
}
```

这个测试涵盖了三种情况：

+   如果`enum WhichImpl`中的所有常量都存在于配置文件中，那么`getWhichImplValueFromConfig()`方法就可以正常工作——它会找到它们中的每一个，不会抛出异常

+   如果传递给`getWhichImplValueFromConfig()`方法的键不是来自`enum WhichImpl`，则该方法会抛出一个异常，其中包含消息`Houston, we have a problem. Unknown key which.impl value unknown is in config`

+   如果传递给`getWhichImplValueFromConfig()`方法的键在配置文件中不存在，则该方法会抛出一个异常，其中包含消息`Houston, we have a problem. Unknown key which.impl value unknown is in config`

当我们确信这个方法按预期工作时，我们可以重写`create()`工厂方法如下：

```java
public static Calculator create(){
  WhichImpl whichImpl = 
    Utils.getWhichImplValueFromConfig(Calculator.CONF_NAME, 
                                      Calculator.CONF_WHICH_IMPL);
  switch (whichImpl){
    case multiplies:
      return new CalculatorImpl();
    case adds:
      return new AnotherCalculatorImpl();
    default:
      throw new RuntimeException("Houston, we have another " + 
                "problem. We do not have implementation for the key " +
                Calculator.CONF_WHICH_IMPL + " value " + whichImpl);
  }
}
```

`switch()`结构非常简单：它将执行线程定向到与匹配相应值的 case 下的代码块（更多信息请参阅第十章，*控制流语句*）。

The benefit of creating and using the method `getWhichImplValueFromConfig()` is that the `create()` method became much cleaner and focused on one task only: creating the right object. We will talk about the *Single Responsibility Principle* in section *So many OOD principles and so little time* of Chapter 8, *Object-Oriented Design (OOD) Principles*.

We have captured the Calculator API in one place—the interface `Calculator` —and we have tested it and proved that it works as designed. But there is another possible API aspect—the last one—we have not covered, yet.

# Adding static methods to API

Each of the classes that implement the `Calculator` interface may have static methods in addition to the instance methods defined in the interface. If such static methods could be helpful to the API's users, we should be able to document them in the `Calculator` interface, too, and that is what we are going to do now.

Let's assume that each of the implementations of the `Calculator` interface has a static method, `addOneAndConvertToString()`:

```java
public class CalculatorImpl implements Calculator {
  public static String addOneAndConvertToString(double d){
    System.out.println(CalculatorImpl.class.getName());
    return Double.toString(d + 1);
  }
  //...
}
public class AnotherCalculatorImpl implements Calculator {
  public static String addOneAndConvertToString(double d){
    System.out.println(AnotherCalculatorImpl.class.getName());
    return String.format("%.2f", d + 1);
  }
  //...
}
```

Notice that the methods have the same signature but slightly different implementations. The method in `CalculatorImpl` returns the result as is, while the method in `AnotherCalculatorImpl` returns the formatted value with two decimal places (we will show the result shortly).

Usually, static methods are called via a dot-operator applied to a class:

```java
String s1 = CalculatorImpl.addOneAndConvertToString(42d);
String s2 = AnotherCalculatorImpl.addOneAndConvertToString(42d);
```

But, we would like to hide (encapsulate) from an API client the implementation details so that the client code continues to use only the interface `Calculator`. To accomplish that goal, we will use the class `CalculatorFactory` again and add to it the following method:

```java
public static String addOneAndConvertToString(double d){
  WhichImpl whichImpl = 
       Utils.getWhichImplValueFromConfig(Calculator.CONF_NAME, 
                                         Calculator.CONF_WHICH_IMPL);
  switch (whichImpl){
    case multiplies:
      return CalculatorImpl.addOneAndConvertToString(d);
    case adds:
      return AnotherCalculatorImpl.addOneAndConvertToString(d);
    default:
      throw new RuntimeException("Houston, we have another " +
                "problem. We do not have implementation for the key " +
                Calculator.CONF_WHICH_IMPL + " value " + whichImpl);
  }
}
```

As you may have noticed, it looks very similar to the factory method `create()`. We also used the same values of the `which.impl` property—`multiplies` and `adds`—as identification of the class. With that, we can add the following static method to the `Calculator` interface:

```java
static String addOneAndConvertToString(double d){
  return CalculatorFactory.addOneAndConvertToString(d);
}
```

As you can see, this way we were able to hide the names of the classes that implemented the interface `Calculator` and the static method `addOneAndConvertToString ()`, too.

To test this new addition, we have expanded code in `CalculatorTest` by adding these lines:

```java
double d = 2.12345678;
String mString = "3.12345678";
String aString = "3.12";
String s = Calculator.addOneAndConvertToString(d);
if(whichImpl.equals(Calculator.WhichImpl.multiplies)){
  assertEquals(mString, s);
} else {
  assertNotEquals(mString, s);
}
if(whichImpl.equals(Calculator.WhichImpl.adds)){
  assertEquals(aString, s);
} else {
  assertNotEquals(aString, s);
}
```

在测试中，我们期望`String`类型的一个值，在`WhichImpl.multiplies`的情况下是相同的值，而在`WhichImpl.adds`的情况下是不同格式的值（只有两位小数）。让我们在`calculator.conf`中使用以下设置运行`CalculatorTest`：

```java
{
  "which.impl": "adds"
}
```

结果是：

![](img/d70771b8-dd6c-442b-a6cd-8ea6393c6e3d.png)

当我们将`calculator.conf`设置为值`multiplies`时，结果如下：

![](img/bbaa099b-1060-4191-8a6e-87d12be01e1f.png)

有了这个，我们完成了对计算器 API 的讨论。

# API 已完成

我们的 API 的最终版本如下：

```java
public interface Calculator {
  int multiplyByTwo(int i);
  static Calculator createInstance(){
    return  CalculatorFactory.create();
  }
  static String addOneAndConvertToString(double d){
    return  CalculatorFactory.addOneAndConvertToString(d);
  }
  String CONF_NAME = "calculator.conf";  //file name
  String CONF_WHICH_IMPL = "which.impl"; //key in the .conf file
  enum WhichImpl{
    multiplies, //uses multiplication operation
                // and returns addOneAndConvertToString() 
                // result without formating
    adds    //uses addition operation 
            // and returns addOneAndConvertToString()
            // result with two decimals only
  }
}
```

这样，我们保持了单一的记录源——捕获所有 API 细节的接口。如果需要更多细节，注释可以引用一些外部 URL，其中包含描述每个`Calculator`实现的完整文档。并且，重复我们在本节开头已经说过的，方法名称应该描述方法产生的所有副作用。

实际上，程序员试图编写小巧、重点突出的方法，并在方法名称中捕获方法的所有内容，但他们很少在接口中添加更多的抽象签名。当他们谈论 API 时，他们通常只指的是抽象签名，这是 API 最重要的方面。但我们认为在一个地方记录所有其他 API 方面也是一个好主意。

# 重载、重写和隐藏

我们已经提到了方法重写，并在第二章中解释了它，*Java 语言基础*。方法重写是用子类（或实现接口的类中的默认方法）的方法替换父类中实现的方法，这些方法具有相同的签名（或在实现接口的类中，或在相应的子接口中）。方法重载是在同一个类或接口中创建几个具有相同名称和不同参数（因此，不同签名）的方法。在本节中，我们将更详细地讨论接口、类和类实例的重写和重载成员，并解释隐藏是什么。我们从一个接口开始。

# 接口方法重载

我们在第二章，*Java 语言基础*中已经说过，除了抽象方法，接口还可以有默认方法和静态成员——常量、方法和类。

如果接口中已经存在抽象、默认或静态方法`m()`，就不能添加另一个具有相同签名（方法名称和参数类型列表）的方法`m()`。因此，以下示例生成编译错误，因为每对方法具有相同的签名，而访问修饰符（`private`、`public`）、`static`或`default`关键字、返回值类型和实现不是签名的一部分：

```java
interface A {
  int m(String s);
  double m(String s);  
} 
interface B {
  int m(int s);
  static int m(int i) { return 42; }
}
interface C {
  int m(double i);
  private double m(double s) { return 42d; }
}
interface D {
  int m(String s);
  default int m(String s) { return 42; }
}
interface E {
  private int m(int s) { return 1; };
  default double m(int i) { return 42d; }
}
interface F {
  default int m(String s) { return 1; };
  static int m(String s) { return 42; }
}
interface G {
  private int m(double d) { return 1; };
  static int m(double s) { return 42; }
}
interface H {
  default int m(int i) { return 1; };
  default double m(int s) { return 42d; }
}

```

要创建不同的签名，要么更改方法名称，要么更改参数类型列表。具有相同方法名称和不同参数类型的两个或多个方法构成方法重载。以下是接口中合法的方法重载示例：

```java
interface A {
  int m(String s);
  int m(String s, double d);
  int m(double d, String s);
  String m(int i);
  private double m(double d) { return 42d; }
  private int m(int i, String s) { return 1; }
  default int m(String s, int i) { return 1; }
} 
interface B {
  static int m(String s, int i) { return 42; }
  static int m(String s) { return 42; }
}

```

重载也适用于继承的方法，这意味着以下非静态方法的重载与前面的示例没有区别：

```java
interface D {
  default int m(int i, String s) { return 1; }
  default int m(String s, int i) { return 1; }
}
interface C {
  default double m(double d) { return 42d; }
}
interface B extends C, D {
  int m(double d, String s);
  String m(int i);
}
interface A extends B {
  int m(String s);
  int m(String s, double d);
}

```

您可能已经注意到我们在上一个代码中将`private`方法更改为`default`。我们这样做是因为`private`访问修饰符会使方法对子接口不可访问，因此无法在子接口中重载。

至于静态方法，以下组合的静态和非静态方法虽然允许，但不构成重载：

```java
interface A {
  int m(String s);
  static int m(String s, double d) { return 1 }
} 
interface B {
  int m(String s, int i);
  static int m(String s) { return 42; }
}
interface D {
  default int m(String s, int s) { return 1; }
  static int m(String s, double s) { return 42; }
}
interface E {
  private int m() { return 1; }
  static int m(String s) { return 42; }
}
```

静态方法属于类（因此在应用程序中是唯一的），而非静态方法与实例相关（每个对象都会创建一个方法副本）。

出于同样的原因，不同接口的静态方法不会相互重载，即使这些接口存在父子关系：

```java
interface G {
  static int m(String s) { return 42; }
}

interface F extends G {
  static int m(String s, int i) { return 42; }
}

```

只有属于同一接口的静态方法才能相互重载，而非静态接口方法即使属于不同接口也可以重载，前提是它们具有父子关系。

# 接口方法重写

与重载相比，重写只发生在非静态方法，并且只有当它们具有完全相同的签名时才会发生。

另一个区别是，重写方法位于子接口中，而被重写的方法属于父接口。以下是方法重写的示例：

```java
interface D {
  default int m(String s) { // does not override anything
    return 1; 
  } 
}

interface C extends D {
  default int m(String d) { // overrides method of D
    return 42; 
  } 
}

```

直接实现接口`C`的类，如果没有实现方法`m()`，将从接口`C`获取该方法的实现，而不会从接口`D`获取该方法的实现。只有直接实现接口`D`的类，如果没有实现方法`m()`，将从接口`D`获取该方法的实现。

注意我们使用了直接这个词。通过说类`X`直接实现接口`C`，我们的意思是类`X`定义如下：`class X implements C`。如果接口`C`扩展 D，则类`X`也实现接口`D`，但不是直接实现。这是一个重要的区别，因为在这种情况下，接口`C`的方法可以覆盖具有相同签名的接口`D`的方法，从而使它们对类`X`不可访问。

在编写依赖于覆盖的代码时，一个好的做法是使用注解`@Override`来表达程序员的意图。然后，Java 编译器和使用它的 IDE 将检查覆盖是否发生，并在带有此注解的方法没有覆盖任何内容时生成错误。以下是一些例子：

```java
interface B {
  int m(String s);
}
interface A extends B {
  @Override             //no error 
  int m(String s);
}
interface D {
  default int m1(String s) { return 1; }
}
interface C extends D {
  @Override            //error
  default int m(String d) { return 42; }
}
```

错误将帮助您注意到父接口中的方法拼写不同（`m1()`与`m()`）。以下是另一个例子：

```java
interface D {
  static int m(String s) { return 1; }
}
interface C extends D {
  @Override                  //error
  default int m(String d) { return 42; }
}
```

这个例子会生成一个错误，因为实例方法不能覆盖静态方法，反之亦然。此外，静态方法不能覆盖父接口的静态方法，因为接口的每个静态方法都与接口本身相关联，而不是与类实例相关联：

```java
interface D {
  static int m(String s) { return 1; }
}
interface C extends D{
  @Override               //error
  static int m(String d) { return 42; }
}
```

但是子接口中的静态方法可以隐藏父接口中具有相同签名的静态方法。实际上，任何静态成员——字段、方法或类——都可以隐藏父接口的相应静态成员，无论是直接父接口还是间接父接口。我们将在下一节讨论隐藏。

# 接口静态成员隐藏

让我们看一下以下两个接口：

```java
interface B {
  String NAME = "B";
  static int m(String d) { return 1; }
  class Clazz{
    String m(){ return "B";}
  }
}

interface A extends B {
  String NAME = "A";
  static int m(String d) { return 42; }
  class Clazz{
    String m(){ return "A";}
  }
}
```

接口`B`是接口`A`的父接口（也称为超接口或基接口），接口的所有成员默认都是`public`。接口字段和类也默认都是`static`。因此，接口`A`和`B`的所有成员都是`public`和`static`。让我们运行以下代码：

```java
public static void main(String[] args) {
  System.out.println(B.NAME);
  System.out.println(B.m(""));
  System.out.println(new B.Clazz().m());
}
```

结果将如下所示：

![](img/de04757c-fb2d-4658-a5be-bad01309bd8c.png)

正如您所看到的，效果看起来像是覆盖，但产生它的机制是隐藏。在类成员隐藏的情况下，差异更为显著，我们将在下一节讨论。

# 类成员隐藏

让我们看看这两个类：

```java
class ClassC {
  public static String field = "static field C";
  public static String m(String s){
    return "static method C";
  }
}

class ClassD extends ClassC {
  public static String field = "static field D";
  public static String m(String s){
    return "static method D";
  }
}
```

}

```java
System.out.println(ClassD.field);
System.out.println(ClassD.m(""));
System.out.println(new ClassD().field);
System.out.println(new ClassD().m(""));
ClassC object = new ClassD();
System.out.println(object.field);
System.out.println(object.m(""));
```java

System.out.println(ClassD.field);

System.out.println(ClassD.m(""));

System.out.println(new ClassD().field);

System.out.println(new ClassD().m(""));

ClassC 对象 = new ClassD();

System.out.println(object.field);

System.out.println(object.m(""));

```java
1 System.out.println(ClassD.field);       //static field D
2 System.out.println(ClassD.m(""));       //static method D
3 System.out.println(new ClassD().field); //static field D
4 System.out.println(new ClassD().m("")); //static method D
5 ClassC object = new ClassD();
6 System.out.println(object.field);       //static field C
7 System.out.println(object.m(""));       //static method C

```java

1 System.out.println(ClassD.field); //静态字段 D

2 System.out.println(ClassD.m("")); //静态方法 D

3 System.out.println(new ClassD().field); //静态字段 D

4 System.out.println(new ClassD().m("")); //静态方法 D

5 ClassC object = new ClassD();

6 System.out.println(object.field); //静态字段 C

7 System.out.println(object.m("")); //静态方法 C

```java
class ClassC {
  public static String field1 = "instance field C";
  public String m1(String s){
    return "instance method C";
  }
}
class ClassD extends ClassC {
  public String field1 = "instance field D";
  public String m1(String s){
    return "instance method D";
  }
}
```java

类 ClassC {

public static String field1 = "实例字段 C";

public String m1(String s){

返回"实例方法 C";

}

}

类 ClassD 扩展自 ClassC {

public String field1 = "实例字段 D";

public String m1(String s){

返回"实例方法 D";

}

}

```java
System.out.println(new ClassD().field1);
System.out.println(new ClassD().m1(""));
ClassC object1 = new ClassD();
System.out.println(object1.m1(""));
System.out.println(object1.field1);
System.out.println(((ClassD)object1).field1);

```java

System.out.println(new ClassD().field1);

System.out.println(new ClassD().m1(""));

ClassC object1 = new ClassD();

System.out.println(object1.m1(""));

System.out.println(object1.field1);

System.out.println(((ClassD)object1).field1);

```java
1 System.out.println(new ClassD().field1);     //instance field D
2 System.out.println(new ClassD().m1(""));     //instance method D
3 ClassC object1 = new ClassD();
4 System.out.println(object1.m1(""));          //instance method D
5 System.out.println(object1.field1);          //instance field C
6 System.out.println(((ClassD)object1).field1);//instance field D

```java

1 System.out.println(new ClassD().field1); //实例字段 D

2 System.out.println(new ClassD().m1("")); //实例方法 D

3 ClassC object1 = new ClassD();

4 System.out.println(object1.m1("")); //实例方法 D

5 System.out.println(object1.field1); //实例字段 C

6 System.out.println(((ClassD)object1).field1); //实例字段 D

```java
class ClassC {
  private String field1 = "instance field C";
  public String getField(){ return field1; }
  public void setField(String s){ field1 = s; }
  public String m1(String s){
    return "instance class C";
  }
}
class ClassD extends ClassC {
  private String field1 = "instance field D";
  public String getField(){ return field1; }
  public void setField(String s){ field1 = s; }
  public String m1(String s){
    return "instance class D";
  }
}
```java

类 ClassC {

私有字符串字段 1 = "实例字段 C";

public String getField(){ return field1; }

public void setField(String s){ field1 = s; }

public String m1(String s){

return "实例类 C";

}

}

class ClassD extends ClassC {

private String field1 = "实例字段 D";

public String getField(){ return field1; }

public void setField(String s){ field1 = s; }

public String m1(String s){

return "实例类 D";

}

}

```java
void m() {
  // some code
}
int m(String s){
  // some code
  return 1;
}
void m(int i){
  // some code
}
int m(String s, double d){
  // some code
  return 1;
}
int m(double d, String s){
  // some code
  return 1;
}
```java

void m() {

// 一些代码

}

int m(String s){

// 一些代码

return 1;

}

void m(int i){

// 一些代码

}

int m(String s, double d){

// 一些代码

return 1;

}

int m(double d, String s){

// 一些代码

return 1;

}

```java
public class SimpleMath {
    public int multiplyByTwo(int i){
       return i * 2;
    }
}
```java

public class SimpleMath {

public int multiplyByTwo(int i){

return i * 2;

}

}

```java
public class SimpleMath {
    public int multiplyByTwo(int i){
        return 2 * i;
    }
    public int multiplyByTwo(String s){
        int i = Integer.parseInt(s);
        return 2 * i;
    }
}
```java

public class SimpleMath {

public int multiplyByTwo(int i){

return 2 * i;

}

public int multiplyByTwo(String s){

int i = Integer.parseInt(s);

return 2 * i;

}

}

```java
public class SimpleMath {
    public int multiplyByTwo(int i){
       return 2 * i;
    }
    public int multiplyByTwo(String s){
       int i = Integer.parseInt(s);
       return multiplyByTwo(i);
    }
}
```java

public class SimpleMath {

public int multiplyByTwo(int i){

return 2 * i;

}

public int multiplyByTwo(String s){

int i = Integer.parseInt(s);

return multiplyByTwo(i);

}

}

```java
public class SimpleMath {
  private int i;
  private String s;
  public SimpleMath() {
  }
  public SimpleMath(int i) {
    this.i = i;
  }
  public SimpleMath(String s) {
    this.s = s;
  }
  // Other methods that use values of the fields i and s
  // go here
}
```java

public class SimpleMath {

private int i;

private String s;

public SimpleMath() {

}

public SimpleMath(int i) {

this.i = i;

}

public SimpleMath(String s) {

this.s = s;

}

// Other methods that use values of the fields i and s

// go here

}

```java
public SimpleMath(int i) {
  this.i = i;
}
```java

public SimpleMath(int i) {

this.i = i;

}

```java
public class Person {
  private String firstName;
  private String lastName;
  private LocalDate dob;
  public Person(String firstName, String lastName, LocalDate dob) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.dob = dob;
  }
  public String getFirstName() { return firstName; }
  public String getLastName() { return lastName; }
  public LocalDate getDob() { return dob; }

  @Override
  public boolean equals(Object other){
    if (other == null) return false;
    if (this == other) return true;
    if (!(other instanceof Person)) return false;
    final Person that = (Person) other;
    return this.getFirstName().equals(that.getFirstName()) &&
           this.getLastName().equals(that.getLastName()) &&
           this.getDob().equals(that.getDob());
  }
}
```java

public class Person {

private String firstName;

private String lastName;

private LocalDate dob;

public Person(String firstName, String lastName, LocalDate dob) {

this.firstName = firstName;

this.lastName = lastName;

this.dob = dob;

}

public String getFirstName() { return firstName; }

public String getLastName() { return lastName; }

public LocalDate getDob() { return dob; }

@Override

public boolean equals(Object other){

if (other == null) return false;

if (this == other) return true;

if (!(other instanceof Person)) return false;

final Person that = (Person) other;

return this.getFirstName().equals(that.getFirstName()) &&

this.getLastName().equals(that.getLastName()) &&

this.getDob().equals(that.getDob());

}

}

```java
public class PersonTest {
  @Test
  void equals() {
    LocalDate dob = LocalDate.of(2001, 01, 20);
    LocalDate dob1 = LocalDate.of(2001, 01, 21);

    Person p = new Person("Joe", "Blow", dob);
    assertTrue(p.equals(p));
    assertTrue(p.equals(new Person("Joe", "Blow", dob)));

    assertFalse(p.equals(new Person("Joe1", "Blow", dob)));
    assertFalse(p.equals(new Person("Joe", "Blow1", dob)));
    assertFalse(p.equals(new Person("Joe", "Blow", dob1)));
    assertFalse(p.equals( new Person("Joe1", "Blow1", dob1)));
  }
}
```java

public class PersonTest {

@Test

void equals() {

LocalDate dob = LocalDate.of(2001, 01, 20);

LocalDate dob1 = LocalDate.of(2001, 01, 21);

Person p = new Person("Joe", "Blow", dob);

assertTrue(p.equals(p));

assertTrue(p.equals(new Person("Joe", "Blow", dob)));

assertFalse(p.equals(new Person("Joe1", "Blow", dob)));

assertFalse(p.equals(new Person("Joe", "Blow1", dob)));

assertFalse(p.equals(new Person("Joe", "Blow", dob1)));

assertFalse(p.equals( new Person("Joe1", "Blow1", dob1)));

}

}

```java
assertFalse(p.equals(null));
assertFalse(p.equals(new Person(null, "Blow", dob)));
assertFalse(p.equals(new Person("Joe", null, dob)));
assertFalse(p.equals(new Person(null, null, dob)));
assertFalse(p.equals(new Person(null, null, null)));

assertTrue(new Person(null, "Blow", dob)
   .equals(new Person(null, "Blow", dob)));
assertTrue(new Person("Joe", null, dob)
   .equals(new Person("Joe", null, dob)));
assertTrue(new Person("Joe", "Blow", null)
   .equals(new Person("Joe", "Blow", null)));
assertTrue(new Person(null, null, null)
   .equals(new Person(null, null, null)));

```java

assertFalse(p.equals(null));

assertFalse(p.equals(new Person(null, "Blow", dob)));

assertFalse(p.equals(new Person("Joe", null, dob)));

assertFalse(p.equals(new Person(null, null, dob)));

assertFalse(p.equals(new Person(null, null, null)));

assertTrue(new Person(null, "Blow", dob)

.equals(new Person(null, "Blow", dob)));

assertTrue(new Person("Joe", null, dob)

.equals(new Person("Joe", null, dob)));

assertTrue(new Person("Joe", "Blow", null)

.equals(new Person("Joe", "Blow", null)));

assertTrue(new Person(null, null, null)

.equals(new Person(null, null, null)));

```java
return this.getFirstName().equals(that.getFirstName()) &&
       this.getLastName().equals(that.getLastName()) &&
       this.getDob().equals(that.getDob());

```java

return this.getFirstName().equals(that.getFirstName()) &&

this.getLastName().equals(that.getLastName()) &&

this.getDob().equals(that.getDob());

```java
public Person(String firstName, String lastName, LocalDate dob) {
  this.firstName = firstName == null ? "" : firstName;
  this.lastName = lastName == null ? "" : lastName;
  this.dob = dob;
  if(dob == null){
    throw new RuntimeException("Date of birth is null");
  }
}
```java

public Person(String firstName, String lastName, LocalDate dob) {

this.firstName = firstName == null ? "" : firstName;

this.lastName = lastName == null ? "" : lastName;

this.dob = dob;

if(dob == null){

throw new RuntimeException("Date of birth is null");

}

}

```java
assertFalse(p.equals(null));
assertFalse(p.equals(new Person(null, "Blow", dob)));
assertFalse(p.equals(new Person("Joe", null, dob)));
assertFalse(p.equals(new Person(null, null, dob)));
try {
  new Person("Joe", "Blow", null);
} catch (RuntimeException ex){
  assertNotNull(ex.getMessage());
  //add the record ex.getMessage() to the log here
}

assertTrue(new Person(null, "Blow", dob)
   .equals(new Person(null, "Blow", dob)));
assertTrue(new Person("Joe", null, dob)
   .equals(new Person("Joe", null, dob)));
assertTrue(new Person(null, null, dob)
   .equals(new Person(null, null, dob)));
```java

assertFalse(p.equals(null));

assertFalse(p.equals(new Person(null, "Blow", dob)));

assertFalse(p.equals(new Person("Joe", null, dob)));

assertFalse(p.equals(new Person(null, null, dob)));

try {

new Person("Joe", "Blow", null);

} catch (RuntimeException ex){

assertNotNull(ex.getMessage());

//在这里将记录 ex.getMessage()添加到日志

}

assertTrue(new Person(null, "Blow", dob)

.equals(new Person(null, "Blow", dob)));

assertTrue(new Person("Joe", null, dob)

.equals(new Person("Joe", null, dob)));

assertTrue(new Person(null, null, dob)

.equals(new Person(null, null, dob)));

```java
public class Vehicle {
  private int weightPounds, horsePower;
  public Vehicle(int weightPounds, int horsePower) {
    this.weightPounds = weightPounds;
    this.horsePower = horsePower;
  }
  protected int getWeightPounds(){ return this.weightPounds; }
  protected double getSpeedMph(double timeSec, int weightPounds){
    double v = 
        2.0 * this.horsePower * 746 * timeSec * 32.174 / weightPounds;
    return Math.round(Math.sqrt(v) * 0.68);
  }
}
```java

public class Vehicle {

private int weightPounds, horsePower;

public Vehicle(int weightPounds, int horsePower) {

this.weightPounds = weightPounds;

this.horsePower = horsePower;

}

protected int getWeightPounds(){ return this.weightPounds; }

protected double getSpeedMph(double timeSec, int weightPounds){

double v =

2.0 * this.horsePower * 746 * timeSec * 32.174 / weightPounds;

返回 Math.round（Math.sqrt（v）* 0.68）;

}

}

```java
public class Truck extends Vehicle {
  private int payloadPounds;
  public Truck(int payloadPounds, int weightPounds, int horsePower) {
    super(weightPounds, horsePower);
    this.payloadPounds = payloadPounds;
  }
  public void setPayloadPounds(int payloadPounds) {
    this.payloadPounds = payloadPounds;
  }
  protected int getWeightPounds(){ 
    return this.payloadPounds + getWeightPounds(); 
  }
  public double getSpeedMph(double timeSec){
    return getSpeedMph(timeSec, getWeightPounds());
  }
}
```java

public class Truck extends Vehicle {

private int payloadPounds;

public Truck（int payloadPounds，int weightPounds，int horsePower）{

super（weightPounds，horsePower）;

this.payloadPounds = payloadPounds;

}

public void setPayloadPounds（int payloadPounds）{

this.payloadPounds = payloadPounds;

}

protected int getWeightPounds（）{

返回 this.payloadPounds + getWeightPounds（）;

}

public double getSpeedMph（double timeSec）{

返回以英里/小时为单位的速度（timeSec，getWeightPounds（））。

}

}

```java
public class Car extends Vehicle {
  private int passengersCount;
  public Car(int passengersCount, int weightPounds, int horsePower) {
    super(weightPounds , horsePower);
    this.passengersCount = passengersCount;
  }
  public void setPassengersCount(int passengersCount) {
    this.passengersCount = passengersCount;
  }

  protected int getWeightPounds(){ 
    return this.passengersCount * 200 + getWeightPounds(); }
  public double getSpeedMph(double timeSec){
    return getSpeedMph(timeSec, getWeightPounds());
  }
}
```java

public class Car extends Vehicle {

private int passengersCount;

public Car（int passengersCount，int weightPounds，int horsePower）{

super（weightPounds，horsePower）;

this.passengersCount = passengersCount;

}

public void setPassengersCount（int passengersCount）{

this.passengersCount = passengersCount;

}

protected int getWeightPounds（）{

返回 this.passengersCount * 200 + getWeightPounds（）;}

public double getSpeedMph（double timeSec）{

返回以英里/小时为单位的速度（timeSec，getWeightPounds（））;

}

}

```java
Truck truck = new Truck(500, 2000, 300);
System.out.println(truck.getSpeedMph(10));

```java

Truck truck = new Truck（500，2000，300）;

System.out.println（truck.getSpeedMph（10））;

```java
protected int getWeightPounds(){ 
  return this.payloadPounds + getWeightPounds(); 
}
```java

protected int getWeightPounds(){

return this.payloadPounds + getWeightPounds();

}

```java
protected int getWeightPounds(){ 
  return this.payloadPounds + super.getWeightPounds(); 
}
```java

protected int getWeightPounds(){

return this.payloadPounds + super.getWeightPounds();

}

```java
public double getSpeedMph(double timeSec){
  return getSpeedMph(timeSec, getWeightPounds());
}
```java

public double getSpeedMph(double timeSec){

return getSpeedMph(timeSec, getWeightPounds());

}

```java
public double getSpeedMph(double timeSec){
  return getSpeedMph(timeSec, this.getWeightPounds());
}
```java

public double getSpeedMph(double timeSec){

return getSpeedMph(timeSec, this.getWeightPounds());

}

```java
public ClassName(){
  super();
}
```java

public ClassName(){

super();

}

```java
public class Parent {
}
public class Child extends Parent{
}
```java

public class Parent {

}

public class Child extends Parent{

}

```java
new Child();
```java

new Child();

```java
public class Parent {
  public Parent(int i) {
  }
}
```java

public class Parent {

public Parent(int i) {

}

}

```java
public class Parent {
  public Parent() {
  }
  public Parent(int i) {
  }
}
```java

public class Parent {

public Parent() {

}

public Parent(int i) {

}

}

```java
public class Child extends Parent{
  public Child() {
    super(10);
  }
}
```java

public class Child extends Parent{

public Child() {

super(10);

}

}

```java
public class Child extends Parent{
  public Child(int i) {
    super(i);
  }
}
```java

子类继承父类

public Child(int i) {

super(i);

}

}

```java
public class GrandDad{
  private String name = "GrandDad";
  public GrandDad() {
    System.out.println(name);
  }
}
public class Parent extends GrandDad{
  private String name = "Parent";
  public Parent() {
    System.out.println(name);
  }
}
public class Child extends Parent{
  private String name = "Child";
  public Child() {
    System.out.println(name);
  }
}
```java

public class GrandDad{

private String name = "GrandDad";

public GrandDad() {

System.out.println(name);

}

}

public class Parent extends GrandDad{

private String name = "Parent";

public Parent() {

System.out.println(name);

}

}

public class Child extends Parent{

private String name = "Child";

public Child() {

System.out.println(name);

}

}

```java
GrandDad.class.getSimpleName(); //always returns "GrandDad"
```java

GrandDad.class.getSimpleName(); //总是返回"GrandDad"

```java
public class GrandDad{
  private static String NAME = GrandDad.class.getSimpleName();
  public GrandDad() {
    System.out.println(NAME);
  }
}
public class Parent extends GrandDad{
  private static String NAME = Parent.class.getSimpleName();
  public Parent() {
    System.out.println(NAME);
  }
}
public class Child extends Parent{
  private static String NAME = Child.class.getSimpleName();
  public Child() {
    System.out.println(NAME);
  }
}
```java

public class GrandDad{

private static String NAME = GrandDad.class.getSimpleName();

public GrandDad() {

System.out.println（NAME）;

}

}

public class Parent extends GrandDad{

私有静态字符串名称= Parent.class.getSimpleName（）;

public Parent（）{

System.out.println（NAME）;

}

}

public class Child extends Parent{

私有静态字符串名称= Child.class.getSimpleName（）;

public Child（）{

System.out.println（NAME）;

}

}

```java
public class GrandDad{
  private static String NAME = GrandDad.class.getSimpleName()
  public GrandDad() {
    System.out.println(NAME);
  }
  public GrandDad(String familyName) {
    System.out.println(familyName + ": " + NAME);
  }
}
public class Parent extends GrandDad{
  private static String NAME = Parent.class.getSimpleName()
  public Parent() {
    System.out.println(NAME);
  }
  public Parent(String familyName) {
    System.out.println(familyName + ": " + NAME);
  }
}
public class Child extends Parent{
  private static String NAME = Child.class.getSimpleName()
  public Child() {
    System.out.println(NAME);
  }
  public Child(String familyName) {
    System.out.println(familyName + ": " + NAME);
  }
}
```java

public class GrandDad{

私有静态字符串名称= GrandDad.class.getSimpleName（）

public GrandDad（）{

System.out.println（NAME）;

}

public GrandDad（String familyName）{

System.out.println（familyName +“：”+ NAME）;

}

}

public class Parent extends GrandDad{

私有静态字符串名称= Parent.class.getSimpleName（）

public Parent（）{

System.out.println（NAME）;

}

public Parent（String familyName）{

System.out.println（familyName +“：”+ NAME）;

}

}

public class Child extends Parent{

私有静态字符串名称= Child.class.getSimpleName（）

public Child（）{

System.out.println（NAME）;

}

public Child（String familyName）{

System.out.println（familyName +“：”+ NAME）;

}

}

```java
public GrandDad(String familyName) {
  System.out.println(familyName + ": " + NAME);
}
public Parent(String familyName) {
  super(familyName);
  System.out.println(familyName + ": " + NAME);
}
public Child(String familyName) {
  super(familyName);
  System.out.println(familyName + ": " + NAME);
}
```java

public GrandDad（String familyName）{

System.out.println（familyName +“：”+ NAME）;

}

public Parent（String familyName）{

super（familyName）;

System.out.println（familyName +“：”+ NAME）;

}

public Child（String familyName）{

super（familyName）;

System.out.println（familyName +“：”+ NAME）;

}

```java
public class Child extends Parent{
  private static String NAME = Child.class.getSimpleName()
  public Child() {
    this("The Defaults");
  }
  public Child(String familyName) {
    super(familyName);
    System.out.println(familyName + ": " + NAME);
  }
}
```java

public class Child extends Parent{

私有静态字符串名称= Child.class.getSimpleName（）

public Child（）{

this（“The Defaults”）;

}

public Child（String familyName）{

super（familyName）;

System.out.println（familyName +“：”+ NAME）;

}

}

```java
        class SomeClass{
          private String someValue = "Initial value";
          public void setSomeValue(String someValue) {
            this.someValue = someValue;
          }
          public String getSomeValue() {
            return someValue;
          }
        }
        public class FinalDemo {
          public static void main(String... args) {
            final SomeClass o = new SomeClass();
            System.out.println(o.getSomeValue());   //Initial value
            o.setSomeValue("Another value");
            System.out.println(o.getSomeValue());   //Another value
            o.setSomeValue("Yet another value");
            System.out.println(o.getSomeValue());   //Yet another value

            final String s1, s2;
            final int x, y;
            y = 2;
            int v = y + 2;
            x = v - 4;
            System.out.println("x = " + x);        //x = 0
            s1 = "1";
            s2 = s1 + " and 2";
            System.out.println(s2);                // 1 and 2 
            //o = new SomeClass();                 //error
            //s2 = "3";                            //error
            //x = 5;                               //error
            //y = 6;                               //error
          }
        }
```java

类 SomeClass {

私有字符串 someValue =“初始值”;

public void setSomeValue（String someValue）{

this.someValue = someValue;

}

public String getSomeValue（）{

返回 someValue;

}

}

公共类 FinalDemo {

public static void main（String ... args）{

最终 SomeClass o = new SomeClass（）;

System.out.println（o.getSomeValue（））; //初始值

o.setSomeValue（“另一个值”）;

System.out.println（o.getSomeValue（））; //另一个值

o.setSomeValue（“另一个值”）;

System.out.println（o.getSomeValue（））; //另一个值

最终字符串 s1，s2;

最终 int x，y;

y = 2;

int v = y + 2;

x = v-4;

System.out.println（“x =”+ x）; // x = 0

s1 =“1”;

s2 = s1 +“和 2”;

System.out.println（s2）; // 1 和 2

// o = new SomeClass（）; //错误

// s2 =“3”; //错误

// x = 5; //错误

// y = 6; //错误

}

}

```java
        public class FinalDemo {
          final SomeClass o = new SomeClass();
          final String s1 = "Initial value";
          final String s2;
          final String s3;
          final int i = 1;
          final int j;
          final int k;
          {
            j = 2;
            s2 = "new value";
          }
          public FinalDemo() {
            k = 3;
            s3 = "new value";
          }
          public void method(){
            //this.i = 4;         //error
            //this.j = 4;         //error
            //this.k = 4;         //error
            //this.s3 = "";       //error
            this.o.setSomeValue("New value");
          }
        }
```java

公共类 FinalDemo {

最终 SomeClass o = new SomeClass（）;

最终字符串 s1 =“初始值”;

最终 s2;

最终字符串 s3;

最终 int i = 1;

最终 int j; 

最终 k;

{

j = 2;

s2 =“新值”;

}

公共 FinalDemo（）{

k = 3;

s3 =“新值”;

}

public void method（）{

// this.i = 4; //错误

// this.j = 4; //错误

// this.k = 4; //错误

// this.s3 =“”; //错误

this.o.setSomeValue（“新值”）;

}

}

```java
        public class FinalDemo {
          final static SomeClass OBJ = new SomeClass();
          final static String S1 = "Initial value";
          final static String S2;
          final static int INT1 = 1;
          final static int INT2;
          static {
            INT2 = 2;
            S2 = "new value";
          }    
          void method2(){
            OBJ.setSomeValue("new value");
            //OBJ = new SomeClass();
            //S1 = "";
            //S2 = "";
            //INT1 = 0;
            //INT2 = 0;
          }
        }
```java

公共类 FinalDemo {

最终静态 SomeClass OBJ = new SomeClass（）;

最终静态字符串 S1 =“初始值”;

最终静态字符串 S2;

最终静态 int INT1 = 1;

最终静态 int INT2;

静态{

INT2 = 2;

S2 =“新值”;

}

void method2（）{

OBJ.setSomeValue（“新值”）;

// OBJ = new SomeClass（）;

// S1 =“”;

// S2 =“”;

// INT1 = 0;

// INT2 = 0;

}

}

```java
void someMethod(final int i, final String s, final SomeClass o){
    //... 
}
```java

void someMethod(final int i, final String s, final SomeClass o){

//...

}

```java
class FinalVariable{
    private int i;
    public FinalVariable() { this.i = 1; }
    public void setInt(int i){
        this.i = 100;
        i = i;
    }
    public int getInt(){
        return this.i;
    }
}
```java

class FinalVariable{

private int i;

public FinalVariable() { this.i = 1; }

public void setInt(int i){

this.i = 100;

i = i;

}

public int getInt(){

return this.i;

}

}

```java
FinalVariable finalVar = new FinalVariable();
System.out.println("Initial setting: finalVar.getInt()=" + 
                                                 finalVar.getInt());
finalVar.setInt(5);
System.out.println("After setting to 5: finalVar.getInt()=" + 
                                                 finalVar.getInt());
```java

FinalVariable finalVar = new FinalVariable();

System.out.println("初始设置：finalVar.getInt()=" +

finalVar.getInt());

finalVar.setInt(5);

System.out.println("设置为 5 后：finalVar.getInt()=" +

finalVar.getInt());

```java
public void setInt(final int i){
  this.i = 100;
  i = i;
}
```java

public void setInt(final int i){

this.i = 100;

i = i;

}

```java
public void setInt(final int i){
    this.i = 100;
    this.i = i;
}
```java

public void setInt(final int i){

this.i = 100;

this.i = i;

}

```java
public class SingletonClassExample {
  private static SingletonClassExample OBJECT = null;

  private SingletonClassExample(){}

  public final SingletonClassExample getInstance() {
    if(OBJECT == null){
      OBJECT = new SingletonClassExample();
    }
    return OBJECT;
  }

  //... other class functionality
}
```java

public class SingletonClassExample {

private static SingletonClassExample OBJECT = null;

private SingletonClassExample(){}

public final SingletonClassExample getInstance() {

if(OBJECT == null){

OBJECT = new SingletonClassExample();

}

return OBJECT;

}

//... 其他类功能

}

```

另一种解决方案可能是将类私有化到工厂类中，并将其存储在工厂字段中，类似于以前的代码。

但要注意，如果这样一个单一对象具有正在改变的状态，就必须确保可以同时修改状态并依赖于它，因为这个对象可能会被不同的方法同时使用。

# 总结

本章还对最常用的术语 API 进行了详细讨论，以及相关主题的对象工厂、重写、隐藏和重载。此外，还详细探讨了关键字`this`和`super`的使用，并在构造函数的解释过程中进行了演示。本章以关键字`final`及其在局部变量、字段、方法和类中的使用进行了概述。

在下一章中，我们将描述包和类成员的可访问性（也称为可见性），这将帮助我们扩展面向对象编程的一个关键概念，封装。这将为我们讨论面向对象设计原则奠定基础。
