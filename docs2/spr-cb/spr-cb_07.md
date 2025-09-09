# 第七章。单元测试

在本章中，我们将介绍以下菜谱：

+   使用 JUnit 4 进行单元测试

+   使用 TestNG 6 进行单元测试

+   使用 Mockito 模拟依赖关系

+   使用 Spring 应用程序上下文进行 JUnit 4 的单元测试

+   使用 Spring 应用程序上下文进行 TestNG 6 的单元测试

+   使用事务进行单元测试

+   单元测试控制器方法

# 简介

我们经常跳过单元测试，因为我们不知道如何进行测试，或者我们相信测试 Web 应用程序是困难的。实际上，单元测试很容易，Spring 使得 Web 应用程序测试变得轻松。

在本章中，你将首先学习如何使用 JUnit、TestNG 和 Mockito 编写单元测试。然后，我们将在测试中使用 Spring 上下文、依赖注入和事务。最后，我们将使用几行代码来测试 Spring 控制器方法。

# 使用 JUnit 4 进行单元测试

**JUnit**，首次发布于 2000 年，是最广泛使用的 Java 单元测试框架。Eclipse 默认支持它。

在这个菜谱中，我们将编写和执行一个 JUnit 4 测试。

## 准备工作

在这个菜谱中，我们将测试这个简单的方法（位于`NumberUtil`类中），该方法用于添加两个整数：

```java
public static int add(int a, int b) {
  return a + b;
}
```

## 如何做…

按照以下步骤使用 JUnit 4 测试一个方法：

1.  在`pom.xml`中添加`junit` Maven 依赖项：

    ```java
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.10</version>
      <scope>test</scope>
    </dependency>
    ```

1.  为你的测试类创建一个 Java 包。标准做法是将测试类放在一个单独的文件夹中，保持与包结构相同。例如，我们要测试的类`NumberUtil`位于`src/main/java`文件夹中，`com.spring_cookbook.util`包下。我们的对应测试类将位于`src/test/java`文件夹中，同样在`com.spring_cookbook.util`包下。

1.  创建测试类；在 Eclipse 中，在**文件**菜单中选择**新建** | **JUnit 测试用例**。我们将使用`NumberUtilTest`作为类名。

1.  通过替换默认的`test()`方法来创建单元测试方法：

    ```java
    @Test
    public void testAdd() {
      assertEquals(NumberUtil.add(5, 3), 8);
      assertEquals(NumberUtil.add(1500, 32), 1532);
    }
    ```

1.  在 Eclipse 中运行测试；在类中右键单击某处，选择**运行方式** | **JUnit 测试**。

1.  如果你使用了步骤 2 中描述的文件夹和包结构，你也可以使用 Maven 运行测试：

    ```java
    mvn test
    ```

## 它是如何工作的…

JUnit 类是一个带有一些用`@Test`注解的方法的正常 Java 类。

`assertEquals(x, y)`是 JUnit 方法，如果`x`不等于`y`，则使测试方法失败。

在`testAdd()`方法中，我们检查该方法是否适用于不同的数据集。

## 还有更多…

一些其他有用的 JUnit 方法注解包括：

+   `@Test(expected=Exception.class)`：此方法预期会抛出此异常。例如，确保在给定情况下某些代码会抛出此异常。

+   `@Before`：此方法在每个测试方法执行前执行。例如，用于重新初始化方法使用的一些类属性。

+   `@After`：此方法在每个测试方法执行后执行。例如，用于回滚数据库修改。

+   `@BeforeClass`：此方法在执行完类的所有测试方法前执行一次。例如，此方法可以包含一些初始化代码。

+   `@AfterClass`：此方法在执行完类的所有测试方法后执行一次。例如，此方法可以包含一些清理代码。

+   `@Test(timeout=1000)`：如果此方法执行时间超过 1 秒，则测试失败。例如，确保某些代码的执行时间保持在一定时长以下。

为测试方法命名一个命名约定有助于使代码更易于维护和阅读。要获取有关不同命名约定的想法，你可以访问

[`java.dzone.com/articles/7-popular-unit-test-naming`](http://java.dzone.com/articles/7-popular-unit-test-naming)。

# 使用 TestNG 6 进行单元测试

**TestNG**，首次发布于 2004 年，是第二受欢迎的 Java 单元测试框架。它拥有大多数 JUnit 功能，还提供了参数化测试（使用不同数据集执行测试方法）和方便的集成测试功能。

在这个菜谱中，我们将编写一个参数化测试来测试与上一个菜谱中相同的方法。

## 准备中

在这个菜谱中，我们将测试这个简单的方法（位于 `NumberUtil` 类中），它将两个整数相加：

```java
public static int add(int a, int b) {
  return a + b;
}
```

## 如何做到这一点…

按照以下步骤使用 TestNG 6 测试一个方法：

1.  在 `pom.xml` 中添加 `testng` Maven 依赖项：

    ```java
    <dependency>
      <groupId>org.testng</groupId>
      <artifactId>testng</artifactId>
      <version>6.1.1</version>
      <scope>test</scope>
    </dependency>
    ```

1.  在 Eclipse 中，安装 TestNG 插件。在 **帮助** 菜单中，选择 **安装新软件...**。在 **工作与** 字段中，输入 `http://beust.com/eclipse` 并按 *Enter* 键。在 **工作与** 字段下方选择 **TestNG**。

1.  为你的测试类创建一个 Java 包。标准做法是将测试类放在一个单独的文件夹中，具有相同的包结构。例如，我们要测试的类 `NumberUtil` 在 `src/main/java` 文件夹中，位于 `com.spring_cookbook.util` 包中。我们的对应测试类将在 `src/test/java` 文件夹中，同样位于 `com.spring_cookbook.util` 包中。

1.  创建 `NumberUtilTest` TestNG 测试类：

    ```java
    import static org.testng.Assert.*;

    public class NumberUtilTest {

    }
    ```

1.  添加一个具有多个数据集的 `@DataProvider` 方法：

    ```java
    @DataProvider
    public Object[][] values() {  
    return new Object[][] {
      new Object[] { 1, 2, 3 },
      new Object[] { 4, 5, 9 },
      new Object[] { 3000, 2000, 5000 },
      new Object[] { 25, 50, 75 },
     };
    ```

1.  添加单元测试方法，该方法接受三个整数并检查前两个整数的和是否等于第三个：

    ```java
    @Test(dataProvider = "values")
    public void testAdd(int a, int b, int c) {
      assertEquals(NumberUtil.add(a, b), c);
    }
    ```

1.  在 Eclipse 中运行测试；在类中某处右键点击，选择 **运行方式** | **TestNG 测试**。

1.  如果你使用了步骤 3 中描述的文件夹和包结构，你也可以使用 Maven 运行测试：

    ```java
    mvn test
    ```

## 它是如何工作的…

`@Test` 方法的 `dataProvider` 属性将用于使用 `@DataProvider` 方法中的数组测试该方法。

在控制台中，验证每个数据集是否已执行测试方法：

```java
PASSED: testAdd(1, 2, 3)
PASSED: testAdd(4, 5, 9)
PASSED: testAdd(3000, 2000, 5000)
```

## 还有更多…

一些其他有用的 TestNG 方法注解包括：

+   `@Test(expectedExceptions=Exception.class)`：此方法预期会抛出此异常。例如，确保在某种情况下某些代码会抛出此异常。

+   `@BeforeMethod`: 这个方法在每个测试方法执行之前执行。例如，重新初始化方法使用的一些类属性。

+   `@AfterMethod`: 这个方法在每个测试方法执行之后执行。例如，用于回滚数据库修改。

+   `@BeforeClass`: 这个方法在类中测试方法执行之前执行一次。例如，这个方法可能包含一些初始化代码。

+   `@AfterClass`: 这个方法在类中所有测试方法执行之后执行一次。例如，这个方法可能包含一些清理代码。

+   `@Test(invocationTimeOut=1000)`: 如果这个方法运行时间超过 1 秒，则测试失败。例如，确保某些代码的执行时间保持在一定时长内。

只有在另一个测试成功的情况下才能执行测试（对于集成测试很有用）：

```java
@Test
public void connectToDatabase() {}

@Test(dependsOnMethods = { "connectToDatabase" })
public void testMyFancySQLQuery() {
  ...
}
```

其他高级特性在 TestNG 的文档中有详细解释，请参阅[`testng.org/doc/documentation-main.html#parameters-dataproviders`](http://testng.org/doc/documentation-main.html#parameters-dataproviders)。

关于选择 TestNG 而不是 JUnit 的更多原因，请参阅[`kaczanowscy.pl/tomek/sites/default/files/testng_vs_junit.txt.slidy_.html`](http://kaczanowscy.pl/tomek/sites/default/files/testng_vs_junit.txt.slidy_.html)。

为测试方法命名约定有助于代码更易于维护和阅读。你可以在以下位置找到关于不同命名约定的一些想法：

[`java.dzone.com/articles/7-popular-unit-test-naming`](http://java.dzone.com/articles/7-popular-unit-test-naming).

# 使用 Mockito 通过模拟来模拟依赖关系

与集成测试相比，单元测试的目的是独立测试每个类。然而，许多类都有我们不希望依赖的依赖关系。因此，我们使用模拟。

**模拟**是智能对象，其输出可以取决于输入。**Mockito**是最受欢迎的模拟框架，语法简洁，易于理解。

## 准备工作

我们将使用`concat()`方法将两个`String`对象连接起来的`StringUtil`类进行模拟：

```java
public class StringUtil {
  public String concat(String a, String b) {
    return a + b;
  }
}
```

### 注意

注意，没有很好的理由去模拟这个类，因为它只是一个方便的例子，用来展示如何使用 Mockito。

## 如何做到这一点...

使用 Mockito 通过模拟来模拟依赖关系的步骤如下：

1.  在`pom.xml`中添加`mockito-core` Maven 依赖项：

    ```java
    <dependency>
      <groupId>org.mockito</groupId>
      <artifactId>mockito-core</artifactId>
      <version>1.10.8</version>
        <scope>test</scope>
    </dependency>
    ```

1.  在测试方法中，使用 Mockito 创建`StringUtil`的`mock`实例：

    ```java
    StringUtil stringUtilMock = Mockito.mock(StringUtil.class);
    ```

1.  编程模拟：

    ```java
    Mockito.when(stringUtilMock.concat("a", "b")).thenReturn("ab");
    Mockito.when(stringUtilMock.concat("aa", "bb")).thenReturn("aabb");
    ```

1.  就这样。现在让我们检查模拟实际上是如何工作的：

    ```java
    assertEquals(stringUtilMock.concat("a", "b"), "ab");    
    assertEquals(stringUtilMock.concat("aa", "bb"), "aabb");    
    ```

## 它是如何工作的...

使用模拟，我们不需要实际的`StringUtil`类来执行我们的测试方法。Mockito 在幕后创建并使用代理类。

再次强调，在现实世界中，为这个类创建模拟可能是过度行为。模拟对于模拟复杂的依赖关系很有用，例如 SMTP 方法、其后的 SMTP 服务器或 REST 服务。

## 还有更多...

使用 `String` 参数测试方法是否恰好被调用两次很容易：

```java
Mockito.verify(stringUtilMock, VerificationModeFactory.times(2)).concat(Mockito.anyString(), Mockito.anyString());
```

Mockito 提供了许多其他类似的方法：

```java
VerificationModeFactory.atLeastOnce()
VerificationModeFactory.atLeast(minNumberOfInvocations)
VerificationModeFactory.atMost(maxNumberOfInvocations)

Mockito.anyObject()
Mockito.any(class)
Mockito.anyListOf(class)
```

在某个点上，也可以重置模拟对象的编程行为：

```java
Mockito.reset(stringUtilMock);
```

要获取 Mockito 功能的更详细列表，请参阅 [`mockito.github.io/mockito/docs/current/org/mockito/Mockito.html`](http://mockito.github.io/mockito/docs/current/org/mockito/Mockito.html)。

# 使用 Spring 应用程序上下文进行 JUnit 4 单元测试

JUnit 测试在 Spring 外部运行；在运行测试之前不会初始化 Spring。为了能够使用配置文件中定义的 bean 和依赖注入，需要在测试类中添加一些引导代码。

## 如何做到这一点…

按照以下步骤使用 Spring 的应用程序上下文和 JUnit 4 测试方法：

1.  在 `pom.xml` 中添加 `spring-test` Maven 依赖项：

    ```java
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>4.1.1.RELEASE</version>
      <scope>test</scope>
    </dependency>
    ```

1.  向测试类添加以下注解：

    ```java
    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(classes = {AppConfig.class})
    @WebAppConfiguration
    public class TestControllerTest {
    …
    ```

1.  按照常规使用 Spring bean，例如，作为 `@Autowired` 字段：

    ```java
    @Autowired
    private UserDAO userDAO;

    @Test
    public void testListUsers() {
      List<User> users = userDAO.findAll();
      ...
    }  
    ```

## 它是如何工作的…

`@RunWith(SpringJUnit4ClassRunner.class)` 使用 Spring 运行器而不是默认的 JUnit 运行器来执行测试。运行器是一个运行 JUnit 测试的类。

`@ContextConfiguration(classes = {AppConfig.class})` 加载 Spring 配置类，并使类的 bean 可用。

`@WebAppConfiguration` 防止抛出异常。如果没有它，`@EnableWebMvc`（在 Spring 配置中）将抛出“**需要 ServletContext 来配置默认的 Servlet 处理**”异常。

## 更多内容…

你可以选择使用单独的 Spring 配置类来运行你的测试：

```java
@ContextConfiguration(classes = {AppTestConfig.class})
```

你还可以将 Spring 主配置类与特定的测试配置类结合使用：

```java
@ContextConfiguration(classes = {AppConfig.class, AppTestConfig.class})
```

声明类的顺序很重要。在这个例子中，`AppConfig` 中的 bean 可以在 `AppTestConfig` 中被覆盖。例如，你可以选择在测试中使用内存数据库数据源来覆盖 MySQL 数据源。

# 使用 Spring 应用程序上下文进行 TestNG 6 单元测试

TestNG 测试在 Spring 外部运行；在运行测试之前不会初始化 Spring。为了能够使用配置文件中定义的 bean 和依赖注入，需要在测试类中添加一些引导代码。

## 如何做到这一点…

按照以下步骤使用 TestNG 6 测试方法，并使用 Spring 应用程序上下文：

1.  在 `pom.xml` 中添加 `spring-test` Maven 依赖项：

    ```java
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>4.1.1.RELEASE</version>
      <scope>test</scope>
    </dependency>
    ```

1.  使测试类继承 `AbstractTestNGSpringContextTests` 并向其添加以下注解：

    ```java
    @ContextConfiguration(classes = {AppConfig.class})
    @WebAppConfiguration
    public class TestControllerTest extends AbstractTestNGSpringContextTests {
    …
    ```

1.  按照常规使用 Spring bean，例如，作为 `@Autowired` 字段：

    ```java
    @Autowired
    private UserDAO userDAO;

    @Test
    public void testListUsers() {
      List<User> users = userDAO.findAll();
      ...
    }  
    ```

## 它是如何工作的…

扩展 `AbstractTestNGSpringContextTests` 将初始化 Spring 的上下文，并使其对测试类可用。

`@ContextConfiguration(classes = {AppConfig.class})` 在 Spring 的上下文中加载 Spring 配置文件。

`@WebAppConfiguration`防止抛出异常。没有它，`@EnableWebMvc`（在 Spring 配置中）会抛出“**需要 ServletContext 来配置默认 servlet 处理**”的异常。

## 更多内容...

你可以选择使用单独的 Spring 配置类来运行你的测试：

```java
@ContextConfiguration(classes = {AppTestConfig.class})
```

你也可以将 Spring 的主配置与特定的测试配置结合使用：

```java
@ContextConfiguration(classes = {AppConfig.class, AppTestConfig.class})
```

声明类的顺序很重要。在这个例子中，`AppConfig`中的 bean 可以被`AppTestConfig`中的 bean 覆盖。例如，你可以选择在测试中用内存数据库数据源覆盖 MySQL 数据源。

# 基于事务的单元测试

例如，要测试 DAO 类，你需要执行不会持久化的数据库查询。例如，要测试添加用户的 DAO 方法，你需要确保用户实际上在数据库中创建，但你不想让这个测试用户留在数据库中。事务可以帮助你以最小的努力做到这一点。

## 如何做...

按照以下步骤自动撤销测试方法执行的数据库修改：

使用 TestNG，使测试类继承：

```java
public class UserDAOTest extends AbstractTransactionalTestNGSpringContextTests  {
...
```

使用 JUnit，在测试类中添加`@Transactional`注解：

```java
@Transactional
public class UserDAOTest {
...
```

## 它是如何工作的...

类中的每个测试方法将自动：

+   开始一个新的事务

+   正常执行

+   回滚事务（这样任何对数据库的修改都将被撤销）

# 单元测试控制器方法

单元测试控制器方法的逻辑通常很困难，但 Spring 通过提供模拟请求并测试 Spring 生成的响应的方法，使它变得简单。

## 准备工作

我们将测试这个将两个参数连接并传递结果到`concat.jsp` JSP 文件的控制器方法：

```java
@RequestMapping("concat")
public String concat(@RequestParam String a, @RequestParam String b, Model model) {
  String result = a + b;
  model.addAttribute("result", result);
  return "concat";
}
```

## 如何做...

要测试控制器方法，构建并执行一个 HTTP 请求，然后对控制器方法返回的响应进行测试。我们将测试对于给定的一组参数，是否将正确的属性传递给 JSP，并且用户被重定向到正确的 URL。以下是执行此操作的步骤：

1.  在`pom.xml`中添加`spring-test`和`hamcrest-all` Maven 依赖项：

    ```java
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>4.1.1.RELEASE</version>
      <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.hamcrest</groupId>
        <artifactId>hamcrest-all</artifactId>
        <version>1.3</version>
        <scope>test</scope>
     </dependency>
    ```

1.  将`@WebAppConfiguration`和`@ContextConfiguration`（以 Spring 配置类作为参数）注解添加到测试类中：

    ```java
    @ContextConfiguration(classes = {AppConfig.class})
    @WebAppConfiguration
    public class StringControllerTest {
    ...
    ```

1.  在测试类中添加一个`WebApplicationContext`属性：

    ```java
    @Autowired
    private WebApplicationContext wac;
    ```

1.  在测试类中添加一个`MockMvc`属性，并在`setup()`方法中使用`WebApplicationContext`属性初始化它：

    ```java
    private MockMvc mockMvc;

    @BeforeMethod
    public void setup() {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(this.wac).build();
    }
    ```

    ### 小贴士

    如果你使用 JUnit，使用`@Before`注解。

1.  将以下`static`导入添加到测试类中：

    ```java
    import static org.springframework.test.web.servlet.request. MockMvcRequestBuilders.*;
    import static org.springframework.test.web.servlet.result. MockMvcResultMatchers.*;
    ```

1.  在测试方法中，我们将构建一个带有参数`a`和`b`的 POST 请求，执行该请求，并测试 Web 应用程序是否响应该 URL，如果模型中设置了正确的`String`，以及是否使用了正确的 JSP：

    ```java
    @Test
    public void testTest1() throws Exception {
        this.mockMvc.perform(post("/concat").param("a", "red").param("b", "apple"))
        .andExpect(status().isOk())
        .andExpect(model().attribute("result", "redapple"))
        .andExpect(forwardedUrl("/WEB-INF/jsp/concat.jsp"));
    }
    ```

## 它是如何工作的...

`setup()`方法在每个测试方法执行之前执行。

在 `setup()` 方法中，`MockMvcBuilders.webAppContextSetup` 对控制器及其依赖项进行完全初始化，允许 `this.mockMvc.perform()` 根据给定的 URL 检索正确的控制器。

## 还有更多……

对于调试，使用 `andDo(MockMvcResultHandlers.print())` 来打印请求和响应的详细信息：

```java
this.mockMvc.perform(...)
    ...
        .andDo(MockMvcResultHandlers.print());
```

此菜谱的输出看起来像：

```java
MockHttpServletRequest:
         HTTP Method = POST
         Request URI = /concat
          Parameters = {a=[red], b=[apple]}
             Headers = {}

             Handler:
                Type = com.spring_cookbook.controllers.StringController
              Method = public java.lang.String com.spring_cookbook.controllers.StringController.concat (java.lang.String,java.lang.String,org.springframework.ui.Model)

               Async:
   Was async started = false
        Async result = null

  Resolved Exception:
                Type = null

        ModelAndView:
           View name = concat
                View = null
           Attribute = result
               value = redapple

            FlashMap:

MockHttpServletResponse:
              Status = 200
       Error message = null
             Headers = {}
        Content type = null
                Body = 
       Forwarded URL = /WEB-INF/jsp/concat.jsp
      Redirected URL = null
             Cookies = []
```

探索 `MockMvcRequestBuilders` 类以找到更多可以测试的元素，请参阅 [`docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/web/servlet/request/MockMvcRequestBuilders.html`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/web/servlet/request/MockMvcRequestBuilders.html)。

例如，你可以测试一个 GET 请求是否得到一些 JSON 内容作为响应，并检查响应中特定元素的价值。

```java
this.mockMvc.perform(get("/user/5"))
    .andExpect(content().contentType("application/json"))
    .andExpect(jsonPath("$.firstName").value("Scott."));
```
