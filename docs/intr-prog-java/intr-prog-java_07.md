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

断言（4，结果）;

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

返回一个新的 AnotherCalculatorImpl();

} else {

throw new RuntimeException("休斯顿，我们有问题。" +

"未知的键 which.impl 值" + whichImpl + "在配置中。");

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

@DisplayName("Utils 测试")

公共类 UtilsTest{

@Test

@DisplayName("按键从配置文件中读取值的测试")

void getStringValueFromConfig(){

//我们将在这里编写测试主体

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

@DisplayName("按键从配置文件中读取值的测试")

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

公共接口计算器{

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

公共接口计算器{

int multiplyByTwo(int i);

static Calculator createInstance(){

return  CalculatorFactory.create();

}

String CONF_NAME = "calculator.conf";

String CONF_WHICH_IMPL = "which.impl";

enum WhichImpl{

multiplies, //使用乘法运算

adds        //使用加法运算

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

静态计算器 createInstance(){

return  CalculatorFactory.create();

}

static String addOneAndConvertToString(double d){

return  CalculatorFactory.addOneAndConvertToString(d);

}

String CONF_NAME = "calculator.conf";  //文件名

String CONF_WHICH_IMPL = "which.impl"; // .conf 文件中的键

枚举 WhichImpl{

multiplies, //使用乘法运算

// 并返回 addOneAndConvertToString()

// 无格式的结果

adds    //使用加法运算

// 并返回 addOneAndConvertToString()

// 仅显示两位小数的结果

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

接口 A {

int m(String s);

双重 m(String s);

}

接口 B {

int m(int s);

static int m(int i) { return 42; }

}

接口 C {

int m(double i);

private double m(double s) { return 42d; }

}

接口 D {

int m(String s);

default int m(String s) { return 42; }

}

接口 E {

private int m(int s) { return 1; };

default double m(int i) { return 42d; }

}

接口 F {

default int m(String s) { return 1; };

static int m(String s) { return 42; }

}

接口 G {

private int m(double d) { return 1; };

static int m(double s) { return 42; }

}

接口 H {

default int m(int i) { return 1; };

default double m(int s) { return 42d; }

}

```

要创建不同的签名，要么更改方法名称，要么更改参数类型列表。具有相同方法名称和不同参数类型的两个或多个方法构成方法重载。以下是接口中合法的方法重载示例：

```java

接口 A {

int m(String s);

int m(String s, double d);

int m(double d, String s);

String m(int i);

private double m(double d) { return 42d; }

private int m(int i, String s) { return 1; }

default int m(String s, int i) { return 1; }

}

接口 B {

static int m(String s, int i) { return 42; }

static int m(String s) { return 42; }

}

```

重载也适用于继承的方法，这意味着以下非静态方法的重载与前面的示例没有区别：

```java

接口 D {

default int m(int i, String s) { return 1; }

default int m(String s, int i) { return 1; }

}

接口 C {

default double m(double d) { return 42d; }

}

接口 B 扩展自 C, D {

int m(double d, String s);

String m(int i);

}

接口 A 扩展自 B {

int m(String s);

int m(String s, double d);

}

```

您可能已经注意到我们在上一个代码中将`private`方法更改为`default`。我们这样做是因为`private`访问修饰符会使方法对子接口不可访问，因此无法在子接口中重载。

至于静态方法，以下组合的静态和非静态方法虽然允许，但不构成重载：

```java

接口 A {

int m(String s);

static int m(String s, double d) { return 1 }

}

接口 B {

int m(String s, int i);

static int m(String s) { return 42; }

}

接口 D {

default int m(String s, int s) { return 1; }

static int m(String s, double s) { return 42; }

}

接口 E {

private int m() { return 1; }

static int m(String s) { return 42; }

}

```

静态方法属于类（因此在应用程序中是唯一的），而非静态方法与实例相关（每个对象都会创建一个方法副本）。

出于同样的原因，不同接口的静态方法不会相互重载，即使这些接口存在父子关系：

```java

接口 G {

static int m(String s) { return 42; }

}

接口 F 扩展自 G {

static int m(String s, int i) { return 42; }

}

```

只有属于同一接口的静态方法才能相互重载，而非静态接口方法即使属于不同接口也可以重载，前提是它们具有父子关系。

# 接口方法重写

与重载相比，重写只发生在非静态方法，并且只有当它们具有完全相同的签名时才会发生。

另一个区别是，重写方法位于子接口中，而被重写的方法属于父接口。以下是方法重写的示例：

```java

接口 D {

default int m(String s) { // 不重写任何内容

返回 1;

}

}

接口 C 扩展自 D {

default int m(String d) { // 重写 D 的方法

return 42;

}

}

```

直接实现接口`C`的类，如果没有实现方法`m()`，将从接口`C`获取该方法的实现，而不会从接口`D`获取该方法的实现。只有直接实现接口`D`的类，如果没有实现方法`m()`，将从接口`D`获取该方法的实现。

注意我们使用了直接这个词。通过说类`X`直接实现接口`C`，我们的意思是类`X`定义如下：`class X implements C`。如果接口`C`扩展 D，则类`X`也实现接口`D`，但不是直接实现。这是一个重要的区别，因为在这种情况下，接口`C`的方法可以覆盖具有相同签名的接口`D`的方法，从而使它们对类`X`不可访问。

在编写依赖于覆盖的代码时，一个好的做法是使用注解`@Override`来表达程序员的意图。然后，Java 编译器和使用它的 IDE 将检查覆盖是否发生，并在带有此注解的方法没有覆盖任何内容时生成错误。以下是一些例子：

```java

接口 B {

int m(String s);

}

接口 A 扩展 B {

@Override             //no error

int m(String s);

}

接口 D {

默认 int m1(String s) { return 1; }

}

接口 C 扩展 D {

@Override            //error

默认 int m(String d) { return 42; }

}

```

错误将帮助您注意到父接口中的方法拼写不同（`m1()`与`m()`）。以下是另一个例子：

```java

接口 D {

静态 int m(String s) { return 1; }

}

接口 C 扩展 D {

@Override                  //error

默认 int m(String d) { return 42; }

}

```

这个例子会生成一个错误，因为实例方法不能覆盖静态方法，反之亦然。此外，静态方法不能覆盖父接口的静态方法，因为接口的每个静态方法都与接口本身相关联，而不是与类实例相关联：

```java

接口 D {

静态 int m(String s) { return 1; }

}

接口 C 扩展 D{

@Override               //error

静态 int m(String d) { return 42; }

}

```

但是子接口中的静态方法可以隐藏父接口中具有相同签名的静态方法。实际上，任何静态成员——字段、方法或类——都可以隐藏父接口的相应静态成员，无论是直接父接口还是间接父接口。我们将在下一节讨论隐藏。

# 接口静态成员隐藏

让我们看一下以下两个接口：

```java

接口 B {

String NAME = "B";

静态 int m(String d) { return 1; }

类 Clazz{

String m(){ return "B";}

}

}

接口 A 扩展 B {

String NAME = "A";

静态 int m(String d) { return 42; }

类 Clazz{

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

类 ClassC {

public static String field = "static field C";

public static String m(String s){

return "static method C";

}

}

类 ClassD 扩展 ClassC {

public static String field = "static field D";

public static String m(String s){

return "static method D";

```

}

```

它们每个都有两个静态成员——一个字段和一个方法。有了这个，看看以下代码：

```java

System.out.println(ClassD.field);

System.out.println(ClassD.m(""));

System.out.println(new ClassD().field);

System.out.println(new ClassD().m(""));

ClassC 对象 = new ClassD();

System.out.println(object.field);

System.out.println(object.m(""));

```

停止阅读并尝试猜测输出将是什么。

以下是相同的代码，带有行号和输出（在注释中捕获）：

```java

1 System.out.println(ClassD.field); //静态字段 D

2 System.out.println(ClassD.m("")); //静态方法 D

3 System.out.println(new ClassD().field); //静态字段 D

4 System.out.println(new ClassD().m("")); //静态方法 D

5 ClassC object = new ClassD();

6 System.out.println(object.field); //静态字段 C

7 System.out.println(object.m("")); //静态方法 C

```

前两行的输出可能是预期的。第 3 行和第 4 行不太直观，但可能也是有道理的。类的任何对象都应该能够访问类成员。然而，不建议通过对象访问静态成员，因为这样的代码隐藏了所访问的成员是静态的事实，这使得代码不太可读，并可能导致不必要的对象创建。同样适用于第 6 行和第 7 行。并且，正如我们在第二章中讨论的那样，*Java 语言基础*，第 5、6 和 7 行表明我们能够将`ClassD`的对象分配给`ClassC`类型的引用，因为`ClassC`是父类，所有子类（所有世代的）都与父类具有相同的类型。这意味着子类可以从所有父类（直接和间接的）继承许多类型。这看起来像是遗传继承，不是吗？

因此，您可以看到子类的静态成员（也称为派生类、扩展类、子类或子类型）可以隐藏其父类的静态成员（也称为基类或超类）。

隐藏字段和隐藏方法之间有两个区别。一个静态字段：

+   隐藏实例变量

+   甚至隐藏一个具有相同名称但不同类型的字段

这里是隐藏的允许情况，之前描述过：

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

```

为了证明这一点，我们可以运行以下代码：

```java

System.out.println(new ClassD().field1);

System.out.println(new ClassD().m1(""));

ClassC object1 = new ClassD();

System.out.println(object1.m1(""));

System.out.println(object1.field1);

System.out.println(((ClassD)object1).field1);

```

再次，你现在可以停止阅读并猜测输出将是什么。

这里是结果：

```java

1 System.out.println(new ClassD().field1); //实例字段 D

2 System.out.println(new ClassD().m1("")); //实例方法 D

3 ClassC object1 = new ClassD();

4 System.out.println(object1.m1("")); //实例方法 D

5 System.out.println(object1.field1); //实例字段 C

6 System.out.println(((ClassD)object1).field1); //实例字段 D

```

正如你所看到的，第 5 行输出了静态变量`ClassC.field1`的值，尽管`ClassD`中也存在同名的字段`field1`。即使我们将`ClassC`中的`field1`更改为非静态，也会显示相同的结果：第 5 行打印出使用引用`object1`的声明类型的字段的值，而不是分配给它的对象的实际类型。更让事情变得更加复杂的是，正如我们之前所述，`ClassC`中`field1`的类型可能与`ClassD`中同名字段的类型不同，而公共字段隐藏的效果仍然是相同的。

为了避免混淆，始终遵循这两个最佳实践：

+   将静态变量的标识符写成大写字符，而实例变量的标识符应该是小写的

+   尝试永远不要公开访问实例字段；将它们设为私有，并通过 getter 和 setter 访问它们的值：

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

```

这样，在覆盖或隐藏的情况下，您将只有一组与方法相关的规则。这更简单，更直接。此外，您可以更好地控制私有字段的值。例如，您可以向 setter 添加代码，以确保字段永远不会被赋予`null`或其他不良值。

# 实例方法覆盖

因此，正如我们在前一节中所看到的，类（或静态）成员不能相互覆盖，而只能隐藏，我们只能谈论覆盖实例成员。我们也已经确定实例字段相互隐藏，字段隐藏的规则与方法覆盖的规则相当不同，因此最佳实践是不公开实例字段，只通过 getter 和 setter 访问它们的值。这样，实例成员覆盖就减少到实例方法覆盖，这就是我们将在本节中描述的内容。

实例方法覆盖的规则与接口默认方法覆盖的规则没有不同：子类中的方法覆盖具有相同签名的父类中的方法。如果签名不同，但方法名称相同，则该方法是重载而不是覆盖。因此，如果您想要覆盖一个方法，最好始终向方法添加`@Override`注释，以确保它不仅仅是静默地重载。

正如我们之前所确定的，子类中的静态方法不能覆盖父类中的实例方法。这使得类实例覆盖规则比接口覆盖规则简单得多。

一个重要的特点要注意的是，尽管构造函数看起来像方法，但它们不是类的方法甚至不是类的成员。构造函数没有返回类型，并且与类具有相同的名称。因此，构造函数不能被覆盖，但可以被重载。它的唯一目的是在创建类（对象）的新实例时被调用。

有了这个，我们就转到了*重载、覆盖和隐藏*部分的最后一个小节：实例方法重载。

# 实例方法重载

对于实例方法重载，只有两个语句来描述它：

+   要使非静态方法重载，它必须与同一类或具有父子关系的类中的另一个非静态方法具有相同的名称和不同的参数类型集。

+   私有非静态方法只能由同一类的非静态方法重载。

因此，这是一个方法重载的示例：

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

```

正如您所看到的，重载方法的名称保持不变，但参数的数量、它们的类型或参数类型的顺序必须不同。否则，它不是重载，编译器将生成错误。返回类型在重载中不起任何作用。它可以相同也可以不同。

从另一个角度来看，所有重载的方法都被认为是不同的。在前面的例子中，我们可以为每个方法赋予不同的名称，并且具有完全相同的代码行为。因此，当您有几个具有相同功能的方法（这就是为什么您不想更改方法名称）但具有不同的参数或不同的参数类型时，重载是有用的。

Here is one possible case of overloading use. You might remember one of the first classes we created was called `SimpleMath`:

```java

public class SimpleMath {

public int multiplyByTwo(int i){

return i * 2;

}

}

```

Then, we might want to add to it another method that, for user convenience, will accept a number as a `String` type: `multiplyByTwo(String s)`. We could do it the following way:

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

```

Or, if we would like to keep the complicated code of multiplying by two in one place (so we can change it in one place only if there is a need to modify it), we could write the following:

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

```

A constructor cannot be overloaded in the same manner:

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

```

With that, we conclude the topic of overloading, overriding, and hiding. It is time to explain in more detail the use of the keywords `this` (used earlier) and `super` (not used yet), and talk more about constructors.

# This, super, and constructors

The keyword `this` provides a reference to the current object. The keyword `super` refers to the parent class object. A constructor is used to initialize the object state (values of the instance fields). It can be accessed using the keywords `new`, `this`, or `super`.

# Keyword this and its usage

We saw several examples of its usage in a constructor similar to the following:

```java

public SimpleMath(int i) {

this.i = i;

}

```

It allows us to clearly distinguish between the object property and local variable, especially when they have the same name.

Another use of the keyword `this` can be demonstrated in the implementation of the method `equals()` in the following `Person` class:

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

```

The reason we need to override the `equals()` method in the parent class `java.lang.Object` (which is the default parent class for all Java classes) is that we would like two objects of the class `Person` to be equal not only when they are actually the same object, but also when the value of each property of one object is the same as the value of the corresponding property of another object. As you can see, we have added an annotation `@Override` to make sure that this method does override the method `equals()` in the parent class `java.lang.Object`. Otherwise, if we make a mistake in the method signature, it may just overload the method `equals()` in the class `java.lang.Object` or, if we make mistake in the method name, be added as just another unrelated to the `equals()` method and we never know about it or would struggle to understand why two different objects of class `Person` are not equal, although all their property values are the same.

第一行检查传入的引用值是否为`null`。如果是，返回的值是`false`，因为显然当前（`this`）对象不是`null`。

我们的`equals()`方法的第二行检查引用的相等性，并且如果它们引用相同的对象，则返回`true`。这与父类`Object`中的默认`equals()`方法的工作方式相同。

我们的`equals()`方法的第三行检查对象`other`是否是类`Person`的实例。我们需要这一行，因为在下一行中，我们将对象`other`转换为类型`Person`，以便能够访问`Person`的 getter。如果对象`other`不能转换为类型`Person`（这意味着引用`other`不引用具有类`Person`作为直接或间接父类的对象），第四行将抛出异常并中断执行流程（JVM 将以错误退出）。因此，我们检查并确保对象`other`在其祖先中具有类`Person`，并且在转换期间不会中断执行流程。

我们新方法`equals()`的最后一行是一个布尔表达式（我们将在第九章中讨论这样的表达式，*运算符、表达式和语句*），它比较当前对象的三个属性的值与另一个对象的相应值，并且仅当两个对象的每个字段都具有相同的值时才返回`true`。

让我们为我们的新方法`equals()`创建一个单元测试：

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

```

正如你所看到的，我们已经为不同出生日期创建了两个对象。然后，我们创建一个`Person`对象并进行比较：

+   对于自身-应该相等

+   对于具有相同状态（属性值）的另一个对象-应该相等

+   对于只有名字不同的另一个对象-不应该相等

+   对于只有姓氏不同的另一个对象-不应该相等

+   对于出生日期不同的另一个对象-不应该相等

+   对于所有属性值都不同的另一个对象-不应该相等

我们运行这个测试并得到了绿色的成功。

但是然后，我们决定测试如果一个或所有的值都是`null`会发生什么，并将以下行添加到测试中：

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

```

首先，我们比较现有对象和所有属性不为`null`的新对象，或者所有属性设置为`null`的新对象。我们期望前四个比较告诉我们这些对象不相等。然后，我们比较两个状态相同的对象，其中一个或所有值设置为`null`。我们期望所有这些对都被报告为相等。

如果我们运行测试，我们会得到以下结果：

![](img/afc4e8bc-30cb-4dac-9b07-8ceb89e62803.png)

错误（`NullPointerExceptions`）表明我们正在尝试在一个尚未分配的引用（具有`null`值）上调用方法，在类`Person`的第 57 行。以下是该行：

```java

return this.getFirstName().equals(that.getFirstName()) &&

this.getLastName().equals(that.getLastName()) &&

this.getDob().equals(that.getDob());

```

我们意识到，当我们在`equals()`方法上调用所有的 getter 时，它们都会返回`null`值，这是`NullPointerException`的源头。我们需要改变`equals()`方法的实现（考虑到`null`值的可能性），或者改变构造函数的实现（不允许传入的值为`null`）。通常，决定可以基于业务需求。例如，在我们要处理的数据中，是否可能存在没有名字、姓氏、出生日期，甚至这些值中的任何一个的人？最后一个——没有任何属性的人——可能不现实。然而，真实的数据经常出现错误，我们可能会问我们的业务人员（也称为领域专家）代码应该如何处理这种情况。然后，我们改变代码以反映新的需求。

假设他们告诉我们，一个或甚至两个属性可以是`null`，我们应该处理这种情况，就好像它们不是`null`一样。但是，他们说，我们不应该处理所有属性都是`null`的情况。

在审查新的需求后，我们再次去找领域专家，并建议，例如，我们将`String`类型的`null`值转换为空字符串`""`，将`LocalDate`类型转换为零年一月一日的日期，但只有当不是所有值都是`null`时。在添加相应的记录到日志文件后，我们跳过这个人的数据。他们建议我们允许名字和姓氏为空，并将它们转换为空字符串类型的`""`，但不处理没有出生日期的人，并在日志文件中记录这种情况。因此，我们将构造函数更改如下：

```java

public Person(String firstName, String lastName, LocalDate dob) {

this.firstName = firstName == null ? "" : firstName;

this.lastName = lastName == null ? "" : lastName;

this.dob = dob;

if(dob == null){

throw new RuntimeException("Date of birth is null");

}

}

```

处理空值的测试部分现在变成了以下内容：

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

```

我们运行它，得到了绿色的成功颜色。

这是关键字`this`的一个例子。我们将在*构造函数*部分展示另一个例子。在*最终变量*部分的末尾，还有一个非常重要的关键字`this`的用法。你不想错过它！否则，它可能是一个非常难以找到的错误的源头。

现在，我们将解释关键字`super`的用法。

# 关键字 super 及其用法

关键字`super`代表父类对象。为了演示它的用法，让我们创建一个车辆、一辆卡车和一辆汽车的编程模型。让我们从车辆开始。模拟车辆的类计算车辆在指定时间段内（以秒为单位）可以达到的速度。它看起来是这样的：

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

```

该类由构造函数设置了两个属性和两个受保护的方法。受保护是一种访问修饰符，表示该方法只能被此类的子类访问。我们将在第七章中更多地讨论访问修饰符，*包和可访问性（可见性）*。

`Car`和`Truck`类（模拟汽车和卡车）可以扩展此类并继承两个受保护的方法，因此它们可用于计算汽车和卡车的速度。还有其他可能的代码组织方式。通常，使用聚合（将`Vehicle`对象设置为`Car`和`Truck`类的字段值）是首选的，除非有理由具有共同的父级（我们将在第八章中讨论这一点，*面向对象设计（OOD）原则*）。但是现在，让我们假设我们有充分的理由使用继承，以便演示关键字`super`的使用。总的来说，这是有道理的：汽车和卡车都是车辆，不是吗？

因此，这就是`Truck`类的外观：

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

```

该类具有一个属性：卡车当前的有效载荷重量。它考虑了速度计算。有效载荷越重，卡车达到相同速度所需的时间就越长。由于有效载荷重量可能在对象创建后随时更改，因此提供了有效载荷重量的 setter，并且受保护的方法`getWeightPounds（）`返回车辆及其有效载荷的总重量。所有建模的主要方法和目的是方法`getSpeedMph（）`，它返回卡车在启动后`timeSec`秒内可以达到的速度（以每小时英里为单位）。

但是，我们现在讨论关键字`super`的使用。您可能已经注意到它已经包含在构造函数中。您可以猜到，它代表父类的构造函数。在这种情况下，关键字`super`必须是子类构造函数的第一行。我们将在下一节*构造函数*中讨论它。

以下是模拟汽车速度的类：

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

```

它看起来与`Truck`类非常相似。唯一的区别是计算有效载荷的方式。假设每位乘客重量为`200`磅。因此，当设置乘客数时，有效载荷被计算为乘客数量乘以`200`。

`Car`和`Truck`两个类都有一个缺陷（也称为错误或错误）。为了发现它，让我们尝试通过运行以下代码来计算卡车在开始时间后 10 秒的速度：

```java

Truck truck = new Truck（500，2000，300）;

System.out.println（truck.getSpeedMph（10））;

```

如果我们这样做，结果将是`StackOverflowError`错误：

！[]（img/926fdbd6-6ee7-4eef-90c4-096d1d553e6b.png）

堆栈是 JVM 内存区域，用于存储方法调用链。最后调用的方法名称存储在顶部。当完成最后调用的方法时，其名称将从顶部移除，并执行下一个方法，依此类推，直到堆栈为空，即当 `main()` 方法完成时，应用程序完成其执行（JVM 退出）。

在我们的案例中，堆栈不受控制地增长，并最终溢出。 JVM 无法在堆栈顶部添加另一个方法名称，并带有错误退出。这种情况的原因是我们的代码在这一行请求了一个递归调用：

```java

protected int getWeightPounds(){

return this.payloadPounds + getWeightPounds();

}

```

我们希望将卡车的有效载荷添加到父类中存储的车辆自身的重量中，但却告诉 JVM 调用相同的方法，因为这个方法被覆盖并且在子类中具有相同的名称，从而递归调用自身。这就是关键字 `super` 发挥作用的地方。通过在方法 `getWeightPounds()` 前面添加它，我们告诉 JVM 不要调用子类的方法，而是调用父类的方法：

```java

protected int getWeightPounds(){

return this.payloadPounds + super.getWeightPounds();

}

```

如果我们再次运行相同的代码，我们将得到预期的结果：

![](img/3425b1a1-e668-4157-bc08-a22eff82094d.png)

嗯，我们的速度计算公式似乎过于乐观了。但谁知道呢？也许到书印刷时，电动卡车将接近这个速度，或者超级环路交通将到达那里。

另外，请注意，我们在计算速度的代码中没有在相同的方法前面添加 `super`：

```java

public double getSpeedMph(double timeSec){

return getSpeedMph(timeSec, getWeightPounds());

}

```

这是因为我们不想调用父类的方法。相反，我们想从子类中覆盖的版本中获取重量。为了确保，并通过使代码更易于阅读来避免混淆，我们可以在其前面添加关键字 `this`：

```java

public double getSpeedMph(double timeSec){

return getSpeedMph(timeSec, this.getWeightPounds());

}

```

实际上，这是我们建议始终遵循的最佳实践之一。

这就结束了关键字 `super` 的用法讨论。我们将在 *构造函数* 部分再次看到它，以及关键字 `this`，在那里我们将解释构造函数是如何工作的，以及默认构造函数是什么。

# 构造函数

对象是作为对象创建模板使用的类的实例。每个对象由其状态和行为定义。对象的状态由其字段的值（也称为属性）定义，对象的行为由其方法定义。由于所有 Java 对象都是 `java.lang.Object` 的后代，因此不可能有没有状态和行为的对象，因为每个对象都继承自 `java.lang.Object` 的方法和一些基本状态。但是，当我们谈论应用程序代码、类和此代码创建的对象时，我们指的是我们定义的方法和状态，以便构建所需的功能。在这种意义上，可能有没有方法的对象。这种对象通常称为数据对象、数据结构或数据传输对象。也可能有没有状态，只有方法的对象。这种对象通常称为实用程序。

如果对象可以有状态，则在对象创建期间必须初始化状态。这意味着必须为表示对象字段的变量分配一些值。这种初始赋值可以通过使用赋值运算符`=`显式地完成，也可以通过让 JVM 将默认值分配给对象的字段来完成。这些默认值取决于字段类型。我们已经在第五章中讨论了每种类型的默认值，*Java 语言元素和类型*。原始数值类型的默认类型为零，布尔类型为`false`，引用类型为`null`。

使用运算符`new`创建对象。此运算符需要指定用于对象创建的构造函数。构造函数的主要职责是初始化对象状态。但是，有三种情况下，不需要在类中显式定义构造函数：

+   当对象或其任何父对象都不能有状态（未定义字段）

+   当在类型声明中为每个字段分配初始值时（`int x = 42;`）

+   当默认值足够好时（例如，默认情况下将字段`int x;`初始化为零的值）

那么如何创建对象呢？运算符`new`期望构造函数。答案是，在这种情况下——当类中没有显式定义构造函数时——编译器会为类生成默认构造函数。此默认构造函数如下所示：

```java

public ClassName(){

super();

}

```

正如您所看到的，它只做了一件事：使用关键字`super`调用父类的构造函数（没有参数的构造函数）。这个没有参数的父构造函数也可能是默认的，也可能是显式创建的。这里存在一个可能的混淆源：如果一个类有一个显式定义的构造函数，则默认的构造函数（无参数）不会自动生成。这种限制的原因是它让程序员掌握控制，并让类的作者决定是否向类添加一个没有参数的构造函数。否则，想象一下，您已经创建了一个类`Person`，并且不想允许在没有填充某些字段的情况下创建此类的实例。您也不希望这些值是默认值，而是希望强制客户端代码在每次创建新的`Person`对象时都显式填充它们。这就是为什么只要在类中定义了一个构造函数（带或不带参数），就不会在幕后自动生成构造函数。让我们测试一下这种行为。以下是两个没有显式定义构造函数（或任何代码）的类：

```java

public class Parent {

}

public class Child extends Parent{

}

```

我们可以很好地运行以下代码：

```java

new Child();

```

它什么也不做，但这不是重点。让我们在父类中添加一个带参数的构造函数：

```java

public class Parent {

public Parent(int i) {

}

}

```

如果我们再次尝试创建类`Child`的对象，我们将会得到一个错误：

![](img/98593acd-b710-4f3a-92f1-02e2f4603e30.png)

点击红线以在 IntelliJ 中查看此错误消息，因为编译器错误消息不够有用：

![](img/f59184e5-c542-45ae-b001-196162ea43f0.png)

它将显式定义的构造函数（带有`int`类型的参数）标识为必需的，并将其参数列表标识为形式参数列表。同时，类`Child`的默认构造函数尝试（如前所述）调用类`Parent`的无参数构造函数，并找不到。错误措辞在这一点上并不是很清楚。

所以，让我们在类`Parent`中添加一个没有参数的构造函数：

```java

public class Parent {

public Parent() {

}

public Parent(int i) {

}

}

```

现在可以创建一个`Child`类的对象而不会出现任何问题。这就是类作者如何控制对象创建过程的方式。

如果你决定只能使用带参数的构造函数来创建`Parent`对象，你可以再次从中删除无参数的构造函数，并向`Child`类添加一个调用带参数的`Parent`构造函数的构造函数：

```java

public class Child extends Parent{

public Child() {

super(10);

}

}

```

或者，你可以向子类添加一个带参数的构造函数：

```java

子类继承父类

public Child(int i) {

super(i);

}

}

```

这些都可以正常工作。

从这个演示中，你可能已经意识到，为了创建一个子类的对象，必须首先创建所有的父对象（并初始化它们的状态）。而且必须从最古老的祖先开始。让我们看看下面的例子：

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

```

你能猜到我们尝试创建子类`new Child()`时的输出吗？如果你猜到`GrandDad`构造函数首先完成，然后是`Parent`，然后是`Child`，那么你是正确的。这是结果：

![](img/e65391e6-3a1e-406c-a12f-7d5f588b3abc.png)

`Child`构造函数调用`Parent`构造函数，然后调用`GrandDad`构造函数，然后调用`java.lang.Object`构造函数。只有在创建了父对象（并且其构造函数已经完成了其任务）之后，子类构造函数才会执行，依此类推，直到父子关系链的末端。

经过一番思考，我们决定从类名中派生字段`name`的值。每个 Java 对象都有一个基类`java.lang.Object`，它通过方法`getClass()`提供对类信息的访问。此方法返回一个`java.lang.Class`类的对象，其中包含有关作为对象模板的类的所有信息，包括其名称。当然，我们首先考虑在`Child`，`Parent`和`GrandDad`中使用`this.getClass().getName()`来获取类名。但是，如果我们通过调用`new Child()`（就像我们的示例中所做的那样）来启动调用链，那么构造`this.getClass().getName()`总是返回类`Child`的名称，即使我们在`GrandDad`中使用该构造。

原因是，尽管关键字`this`表示当前对象（例如，如果在`GrandDad`中使用`this`，则表示`GrandDad`对象），但方法`getClass()`返回的是关于*当前对象*的信息，而不是关于*运行时*对象的信息（由`new`操作符创建的对象），在这种情况下是`Child`的实例。这就是为什么在我们的示例中，构造`this.getClass().getName()`总是返回类`Child`的名称，无论此构造是在`Child`，`Parent`还是`GrandDad`中使用。

但是，还有另一种更适合我们需求的访问类信息的方法。我们可以显式使用类名，为其添加扩展名`.class`，然后再获取类名。这是一个例子：

```java

GrandDad.class.getSimpleName(); //总是返回"GrandDad"

```

看起来与我们之前使用的`String`字面量没有太大区别，是吗？然而，这是一个改进，因为如果类的名称更改，变量`NAME`的值也会更改，而在`String`字面量的情况下，其值不会自动绑定到类的名称。

因此，我们为我们的三个类中的每一个添加了一个带初始化的静态字段`NAME`：

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

```

请注意，我们遵循了仅使用大写字母编写静态变量标识符的约定。

如果我们调用`new Child()`，结果将如下所示：

！[]（img / 32346bc4-8430-4745-b9c5-f076a9180520.png）

如果我们添加一个带参数的构造函数，代码将如下所示：

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

```

执行行`new Child（“The Blows”）`现在将仅更改子项的输出：

！[]（img / b5431dce-b244-456c-b391-1cacaff45e5c.png）

这是因为新的子构造函数继续默认调用父构造函数而不带参数。要调用新父构造函数，我们需要显式地这样做，使用关键字`super`（我们只在这里显示构造函数）：

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

```

通过执行相同的行`new Child（“The Blows”）`，我们得到了期望的结果：

！[]（img / 4d82cd27-f808-4711-8a56-be19b801524a.png）

请注意，关键字`super`必须是构造函数的第一行。如果您尝试将它放在其他任何地方，将生成错误。这是因为所有构造函数必须在调用任何其他代码之前完全执行。父子链中的所有对象必须首先创建并初始化它们的状态，从最顶层的基类开始。

我们想在这里提到的最后一个与构造函数相关的特性是：一个构造函数可以使用关键字`this`调用同一类的另一个构造函数。例如，假设我们不希望家庭没有家庭名称存在，但客户端代码可能永远无法提供一个。因此，我们决定在没有参数的构造函数中添加默认的家庭名称：

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

```

如果我们再次执行行`new Child（）`，我们将得到以下结果：

！[]（img / 4ef79cfa-e0bf-4034-9227-629400f76540.png）

正如您所看到的，相同类的构造函数可以被重载并且可以像方法一样相互调用。但是构造函数不会被继承，因此不能被隐藏或覆盖。这是不可能的。

对于任何其他方法，如果不是私有或最终的，它可以被覆盖。私有是什么；您可能已经有一个想法。我们将在第七章中更详细地讨论它，*包和可访问性（可见性）*。我们将在下一节中讨论*最终变量*。

# 最终变量，最终方法或最终类

关键字`final`的使用及其影响取决于上下文。它可以使变量值不可更改，方法不可覆盖，或类不可扩展。我们将简要讨论每种情况。

# 最终变量

如果在变量声明前面放置关键字`final`，则一旦分配了此变量的值（初始化了变量），则无法更改此变量的值。变量可以初始化的方式取决于变量的使用方式。有三种变量使用方式，每种方式都有不同的初始化规则：

+   局部变量是在代码块中声明的变量；可以使用相同语句中的赋值进行初始化，或者稍后进行初始化，但只能进行一次；这里有一些例子：

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

```

基本类型的最终变量-初始化后第一次-变成常量，不能更改（请参见最后两行，已注释为错误）。同样，引用类型`String`类型的最终变量也不能更改，因为在[第五章]（ddf91055-8610-4b8c-acc5-453cfa981760.xhtml）中讨论的`String`类型不可变性。 * Java 语言元素和类型*。但是，其他引用类型对象（包括数组）只是由最终变量引用。因此，引用本身也不能更改（或重新分配），也保持不变。但是，所引用对象的状态可以更改，如前面使用`SomeClass`对象演示的那样。

+   可以使用相同语句的赋值进行初始化实例变量（与局部变量相同），使用实例初始化块或构造函数：

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

```

但是，在初始化之后，无法更改最终实例变量的基本类型和`String`值，而对象的属性（或数组的组件）可以更改，类似于最终局部变量。

+   （静态）类最终变量可以使用相同语句的赋值进行初始化（与局部或实例变量相同），或者使用静态初始化块：

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

```

与局部和实例 final 变量一样，原始类型和`String`的静态 final 变量在第一次赋值后变为常量，而对象的属性（或数组的组件）可以在以后多次更改。

如果你认为在本节之前从未见过 final 变量，请注意接口字段隐式地是 final 的。一旦赋值，就不能更改。还有另外两种隐式 final 的变量：作为`try...with...resources`语句的资源声明的变量（我们将在第十六章中看到示例，*数据库编程*）和多重捕获子句的异常参数（我们将在第十章中讨论它们，*控制流语句*）。

final 变量对于安全性很重要，但我们不打算在本书中讨论安全性。相反，我们将在第十七章中讨论 Java 函数式编程时看到 final 变量的许多用途，*Lambda 表达式和函数式编程*。

在阅读其他人的代码时，你可能会注意到方法参数被声明为 final：

```java

void someMethod(final int i, final String s, final SomeClass o){

//...

}

```

这通常是为了防止在方法外更改值的副作用。但我们已经在第五章中展示了，原始类型的副本被传递，它们的重新赋值只会更改副本，而不会更改原始值。对于引用类型`String`，我们在同一章节中也展示了它的值是不可变的，因为对于每个`String`变量的重新赋值，都会创建一个新的值的副本，原始值不受影响。至于其他引用类型，使引用本身为 final 只能防止分配新对象。但如果不是这种情况，引用本身为 final 并不能防止在方法外更改原始对象的属性。

因此，除非真的有必要（例如匿名类等情况——编译器和 IDE 会告诉你），将这些变量设为`final`只能防止在方法内部重新分配新值，并不能帮助避免方法外的副作用。

有些程序员也认为，尽可能在声明变量时使用 final 关键字可以使代码作者的意图更容易理解。这是正确的，但前提是必须一致地遵循这种约定，并且所有可以声明为`final`的变量都声明为 final。否则，如果有些变量声明为 final，有些没有（尽管它们可以），代码可能会误导。有人可能会认为，没有`final`关键字的变量是有意这样声明的，因为它们在其他地方被重新赋值为不同的值。如果你不是代码的作者（甚至是代码的作者，但是在一段时间后再看代码），你可以合理地假设可能存在一些逻辑分支利用了某个变量不是 final。你不愿意将`final`添加到现有的变量中，因为你不确定这是有意遗漏还是故意省略，这意味着代码不够清晰，更易读的代码的想法就会破灭。

公平地说，如果一个人能够在任何地方应用关键字`final`，就可以轻松避免一类难以发现的错误。请看这个例子：

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

```

这个类有一个字段`i`，在构造函数中初始化为值`1`。该类还有一个用于该字段的 getter 和 setter。在 setter 中，程序员犯了一个错误。你能发现吗？让我们运行以下代码：

```java

FinalVariable finalVar = new FinalVariable();

System.out.println("初始设置：finalVar.getInt()=" +

finalVar.getInt());

finalVar.setInt(5);

System.out.println("设置为 5 后：finalVar.getInt()=" +

finalVar.getInt());

```

在代码中，我们创建了一个`FinalVariable`类的对象。构造函数为其赋值`1`，我们使用 getter 确认了这一点。然后，我们尝试为其赋值`5`，并期望 getter 返回此值。相反，我们得到了以下输出：

![](img/eab4c3b7-7a1c-4f4c-b2f8-51d8a0c086e8.png)

让我们看看如果声明参数`final`会发生什么，如下所示：

```java

public void setInt(final int i){

this.i = 100;

i = i;

}

```

编译器和 IDE 会警告我们，我们试图将变量`i`赋给另一个值。我们会看到问题并进行修复，就像这样：

```java

public void setInt(final int i){

this.i = 100;

this.i = i;

}

```

然后代码将开始按我们的预期行为：

![](img/b909084a-8295-4ad3-8470-576f6d6ee355.png)

但这样的情况并不多，很快你就会学会如何避免这样的陷阱，并开始自动在实例变量前添加`this`。因此，在我们看来，广泛使用关键字`final`来提高代码质量是不合理的，但一些程序员仍然喜欢这样做，因此我们将其作为编程风格问题留下。

顺便说一句，据报道，在某些特殊情况下，添加关键字`final`作为提高应用程序性能的一种方法是有用的，但我们自己并没有遇到这些情况，因此将其留给那些能够证明这种情况的人。

# Final method

在方法前面加上关键字`final`会使得在子类实例中无法覆盖它，如果方法是静态的，则无法隐藏它。它确保了方法功能不能通过覆盖进行更改。例如，类`java.lang.Object`有许多方法是 final 的。

但是，如果一个 final 方法使用非 final 方法，这可能会导致不良更改的后门引入。当然，这些考虑对于安全性非常重要。

有时，人们会说将方法设置为 final 可以提高代码性能。这可能是事实，但在一些非常特殊的情况下，并不似乎在主流编程中有很大帮助。对于性能改进，通常有更好的机会可用，包括经过验证的面向对象设计原则（参见第八章，*面向对象设计（OOD）原则*）。

所有私有方法和最终类的方法（不会被继承）实际上都是 final 的，因为不能对其进行覆盖。

# Final class

声明为 final 的类不能被扩展。也就是说，它不能有子类，这使得类的所有方法实际上都是 final 的。

这个特性被广泛用于安全性或者当程序员希望确保类功能不能被覆盖、重载或隐藏时。

# 练习-将类实例化限制为单个共享实例

编写一个类，确保只能创建一个对象。

# 答案

以下是一个可能的解决方案：

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
