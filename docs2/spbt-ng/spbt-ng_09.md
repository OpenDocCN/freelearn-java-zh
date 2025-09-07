

# 第九章：在 Spring Boot 中编写测试

在上一章中，你已经了解了日志记录器的重要性、它们的概念以及它们如何帮助开发者调试和维护应用程序。你已经学习了 Log4j2，这是一个为 Spring Boot 提供的第三方框架，它提供了诸如**Appenders**、**Filters**和**Markers**等特性，可以帮助开发者对日志事件进行分类和格式化。我们还讨论了 SLF4J，它是对日志框架的抽象，允许我们在运行时或部署期间在多个框架之间切换，最后，我们使用 XML 配置和 Lombok 实现了并配置了日志框架。

本章将现在专注于为我们的 Spring Boot 应用程序编写单元测试；我们将讨论最常用的 Java 测试框架 JUnit 和**AssertJ**，并在我们的应用程序中实现它们。我们还将集成 Mockito 到我们的单元测试中，以进行对象和服务模拟。

在本章中，我们将涵盖以下主题：

+   理解 JUnit 和 AssertJ

+   编写测试

+   在服务中使用 Mockito 编写测试

# 技术要求

本章完成版本的链接在此：[`github.com/PacktPublishing/Spring-Boot-and-Angular/tree/main/Chapter-09`](https://github.com/PacktPublishing/Spring-Boot-and-Angular/tree/main/Chapter-09)。

# 理解 JUnit 和 AssertJ

在每个应用程序开发之后，测试总是下一步，这是在将我们的应用程序交付或部署到生产环境之前最重要的任务之一。测试阶段对公司来说至关重要，因为这确保了他们产品的质量和有效性。

由于这是基本流程之一，测试过程中应该几乎没有错误空间，而手动测试是不够的，因为手动测试容易出错，并且有更大的机会错过应用程序中存在的问题。这就是单元测试发挥作用的地方——单元测试是自动化测试，允许开发者为单个类或实体编写测试。

这是一种**回归测试**的形式，它会运行所有测试以验证在应用代码经过几次更改或更新后，代码是否仍然通过测试用例。单元测试有助于保持我们应用程序的质量，因为它们带来了以下好处：

+   **速度**：与手动测试相比，单元测试将节省更多时间，因为这是可编程的，并且将在短时间内提供结果。

+   **成本降低**：单元测试是自动化的，这意味着将需要更少的测试人员来测试应用程序。

+   **错误减少**：单元测试将显著减少犯错误的数量，因为测试不是由人类手动完成的。

+   **可编程性**：单元测试可以生成复杂的测试，以检测应用程序中的隐藏信息。

单元测试现在在前后端开发中都广泛使用，尤其是在 Java 中，因为它们的优点和测试。Java 中已经存在几个测试框架，但我们将讨论第一个也是最常用的框架，**JUnit**。

## JUnit 框架

JUnit 是一个回归测试框架，主要用于为 Java 应用程序中的单个类编写测试和断言；它提倡“先测试后编码”的理念，即我们需要在实现之前为要测试的代码创建测试数据。JUnit 也是一个开源框架，这使得它更加可靠。

有一个庞大的社区支持此框架，它使用断言来测试预期结果，并使用注解来识别测试方法，它可以有效地利用并集成到 Maven 和 Gradle 项目中。

让我们讨论我们将用于编写测试的 JUnit 特性：

+   `setUp()`: 此方法在每次测试被调用之前执行。

+   `tearDown()`: 此方法在每次测试被调用之后执行：

    ```java
    public class JavaTest extends TestCase {
    ```

    ```java
       protected int value1, value2;
    ```

    ```java
       // will run before testSubtract and testMultiply
    ```

    ```java
       protected void setUp(){
    ```

    ```java
          value1 = 23;
    ```

    ```java
          value2 = 10;
    ```

    ```java
       }
    ```

    ```java
       public void testSubtract(){
    ```

    ```java
          double result = value1 - value2;
    ```

    ```java
          assertTrue(result == 13);
    ```

    ```java
       }
    ```

    ```java
       public void testMultiply(){
    ```

    ```java
          double result = value1 * value2;
    ```

    ```java
          assertTrue(result == 230);
    ```

    ```java
       }}
    ```

在前面的代码示例中，我们可以看到定义了两个测试方法，分别是`testSubtract()`和`testMultiply()`，在每个方法被调用之前。`setUp()`设置将首先被调用，以分配`value1`和`value2`变量的值。

+   `@RunWith`和`@Suite`注解用于运行测试。让我们看看以下示例：

    ```java
    //JUnit Suite Test
    ```

    ```java
    @RunWith(Suite.class)
    ```

    ```java
    @Suite.SuiteClasses({
    ```

    ```java
       TestOne.class, TestTwo.class
    ```

    ```java
    });
    ```

    ```java
    public class JunitTestSuite {
    ```

    ```java
    }
    ```

    ```java
    public class TestOne {
    ```

    ```java
       int x = 1;
    ```

    ```java
       int y = 2;
    ```

    ```java
       @Test
    ```

    ```java
       public void TestOne() {
    ```

    ```java
          assertEquals(x + y, 3);
    ```

    ```java
       }
    ```

    ```java
    }
    ```

    ```java
    public class TestTwo {
    ```

    ```java
       int x = 1;
    ```

    ```java
       int y = 2;
    ```

    ```java
       @Test
    ```

    ```java
       public void TestTwo() {
    ```

    ```java
          assertEquals(y - x, 1);
    ```

    ```java
       }
    ```

    ```java
    }
    ```

在前面的代码示例中，我们可以看到我们定义了两个类，其中一个带有`@Test`注解的方法；测试方法将一起执行，因为我们已经使用`@Suite.SuiteClasses`方法将它们捆绑在一起。

+   `runClasses()`方法用于在特定类中运行测试用例。让我们看看以下基本示例：

    ```java
    public class JUnitTestRunner {
    ```

    ```java
       public static void main(String[] args) {
    ```

    ```java
          Result result =
    ```

    ```java
            JUnitCore.runClasses(TestJunit.class);
    ```

    ```java
          for (Failure failure : result.getFailures()) {
    ```

    ```java
             System.out.println(failure.toString());
    ```

    ```java
          }
    ```

    ```java
          System.out.println(result.wasSuccessful());
    ```

    ```java
       }
    ```

    ```java
    }
    ```

+   **类**：JUnit 类主要用于编写我们应用程序的测试；这些包括以下内容：

    +   **断言**包括断言方法集

    +   **测试用例**包括包含运行多个测试的设置的测试用例

    +   **测试结果**包括收集执行测试用例的所有结果的方法

## JUnit 中的断言

`Assert`类，以及一些来自 Assert 的基本方法如下：

+   `void assertTrue(boolean condition)`: 验证条件是否为`true`

+   `void assertFalse(boolean condition)`: 验证条件是否为`false`

+   `void assertNotNull(Object obj)`: 检查对象是否不为 null

+   `void assertNull(Object obj)`: 检查对象是否为 null

+   `void assertEquals(Object obj1, Object obj2)`: 检查两个对象或原始数据是否相等

+   `void assertArrayEquals(Array array1, Array array2)`: 验证两个数组是否彼此相等

## 注解

**注解**是元标签，我们将其添加到方法和类中；这为 JUnit 提供了额外信息，说明哪些方法应该在测试方法之前和之后运行，哪些将在测试执行期间被忽略。

这里是 JUnit 中我们可以使用的注解：

+   `@Test`: 该注解用于一个`public void`方法，表示该方法是一个可以执行的测试用例。

+   `@Ignore`: 该注解用于忽略未执行的测试用例。

+   `@Before`: 该注解用于一个`public void`方法，在每次测试用例方法之前运行该方法。如果我们想声明所有测试用例都使用的类似对象，这通常会被使用。

+   `@After`: 该注解用于一个`public void`方法，在每次测试用例方法之后运行该方法；如果我们想在运行新的测试用例之前释放或清理多个资源，这通常会被使用。

+   `@BeforeClass`: 该注解允许一个`public static void`方法在所有测试用例执行之前运行一次。

+   `@AfterClass`: 该注解允许一个`public static void`方法在所有测试用例执行之后运行一次。

让我们通过一个带有注解及其执行顺序的示例测试来看看：

```java
public class JunitAnnotationSequence {
   //execute once before all test
   @BeforeClass
   public static void beforeClass() {
      System.out.println("beforeClass()");
   }
   //execute once after all test
   @AfterClass
   public static void  afterClass() {
      System.out.println("afterClass()");
   }
   //execute before each test
   @Before
   public void before() {
      System.out.println("before()");
   }
   //execute after each test
   @After
   public void after() {
      System.out.println("after()");
   }
   @Test
   public void testMethod1() {
      System.out.println("testMethod1()");
   }
   @Test
   public void testMethod2() {
      System.out.println("testMethod2();");
   }
}
```

在前面的代码示例中，我们有一个`JunitAnnotationSequence`类，它有几个注解方法。当我们执行测试时，我们将得到以下输出：

```java
beforeClass()
before()
testMethod1()
after()
before()
testMethod2()
after()
afterClass()
```

在前面的示例中，我们可以看到，使用`@BeforeClass`和`@AfterClass`注解的方法只调用一次，它们在测试执行的开始和结束时被调用。另一方面，使用`@Before`和`@After`注解的方法在每个测试方法的开始和结束时被调用。

我们已经学习了单元测试中 JUnit 的基础知识；现在，让我们讨论 AssertJ 的概念。

## 使用 AssertJ

在上一部分，我们刚刚探讨了 JUnit 的概念和功能，我们了解到，仅使用 JUnit，我们可以通过`Assert`类应用断言，但通过使用 AssertJ，我们可以使断言更加流畅和灵活。**AssertJ**是一个主要用于编写断言的库；它的主要目标是提高测试代码的可读性，并简化测试的维护。

让我们比较一下在 JUnit 和 AssertJ 中如何编写断言：

+   JUnit 检查条件是否返回`true`：

    ```java
    Assert.assertTrue(condition)
    ```

+   AssertJ 检查条件是否返回`true`：

    ```java
    Assertions.assertThat(condition).isTrue()
    ```

在前面的示例中，我们可以看到，在 AssertJ 中，我们将在`assertThat()`方法中始终传递要比较的对象，然后调用下一个方法，即实际的断言。让我们看看在 AssertJ 中我们可以使用哪些不同类型的断言。

### 布尔断言

`true`或`false`。断言方法如下：

+   `isTrue()`: 检查条件是否为`true`：

    ```java
    Assertions.assertThat(4 > 3).isTrue()
    ```

+   `isFalse()`: 检查条件是否为`false`：

    ```java
    Assertions.assertThat(11 > 100).isFalse()
    ```

## 字符断言

**字符断言**用于将对象与字符进行比较或检查字符是否在 Unicode 表中；断言方法如下：

+   `isLowerCase()`: 检查给定的字符是否为小写：

    ```java
    Assertions.assertThat('a').isLowerCase();
    ```

+   `isUpperCase()`: 检查字符是否为大写：

    ```java
    Assertions.assertThat('a').isUpperCase();
    ```

+   `isEqualTo()`: 检查两个给定的字符是否相等：

    ```java
    Assertions.assertThat('a').isEqualTo('a');
    ```

+   `isNotEqualTo()`: 检查两个给定的字符是否不相等：

    ```java
    Assertions.assertThat('a').isEqualTo('b');
    ```

+   `inUnicode()`: 检查字符是否包含在 Unicode 表中：

    ```java
    Assertions.assertThat('a').inUniCode();
    ```

这些只是`AbstractCharacterAssert`下可用的一些断言。对于完整的文档，您可以访问[`joel-costigliola.github.io/assertj/core-8/api/org/assertj/core/api/AbstractCharacterAssert.html`](https://joel-costigliola.github.io/assertj/core-8/api/org/assertj/core/api/AbstractCharacterAssert.html).

## 类断言

**类断言**用于检查特定类的字段、类型、访问修饰符和注解。以下是一些类断言方法：

+   `isNotInterface()`: 验证该类不是接口：

    ```java
    Interface Hero {}
    ```

    ```java
    class Thor implements Hero {}
    ```

    ```java
    Assertions.assertThat(Thor.class).isNotInterface()
    ```

+   `isInterface()`: 验证该类是接口：

    ```java
    Interface Hero {}
    ```

    ```java
    class Thor implements Hero {}
    ```

    ```java
    Assertions.assertThat(Hero.class).isInterface()
    ```

+   `isPublic()`: 验证该类是公开的：

    ```java
    public class Hero {}
    ```

    ```java
    protected class AntiHero {}
    ```

    ```java
    Assertions.assertThat(Hero.class).isPublic()
    ```

+   `isNotPublic()`: 验证该类不是公开的：

    ```java
    public class Hero {}
    ```

    ```java
    protected class AntiHero {}
    ```

    ```java
    Assertions.assertThat(Hero.class).isNotPublic()
    ```

这些只是`AbstractClassAssert`下可用的一些断言。对于完整的文档，您可以访问[`joel-costigliola.github.io/assertj/core-8/api/org/assertj/core/api/AbstractClassAssert.html`](https://joel-costigliola.github.io/assertj/core-8/api/org/assertj/core/api/AbstractClassAssert.html).

## 迭代器断言

**迭代器断言**用于根据其长度和内容验证迭代器或数组对象。以下是一些迭代器断言方法：

+   `contains()`: 展示迭代器包含指定的值：

    ```java
    List test = List.asList("Thor", "Hulk",
    ```

    ```java
                            "Dr. Strange");
    ```

    ```java
    assertThat(test).contains("Thor");
    ```

+   `isEmpty()`: 验证给定的迭代器长度是否大于`0`：

    ```java
    List test = new List();
    ```

    ```java
    assertThat(test).isEmpty();
    ```

+   `isNotEmpty()`: 验证给定的迭代器长度是否为`0`：

    ```java
    List test = List.asList("Thor", "Hulk",
    ```

    ```java
                            "Dr. Strange");
    ```

    ```java
    assertThat(test).isNotEmpty ();
    ```

+   `hasSize()`: 验证迭代器的长度是否等于指定的值：

    ```java
    List test = List.asList("Thor", "Hulk",
    ```

    ```java
                            "Dr. Strange");
    ```

    ```java
    assertThat(test).hasSize(3);
    ```

这些只是`AbstractIterableAssert`下可用的一些断言。对于完整的文档，您可以访问此处提供的链接：[`joel-costigliola.github.io/assertj/core-8/api/org/assertj/core/api/AbstractIterableAssert.html`](https://joel-costigliola.github.io/assertj/core-8/api/org/assertj/core/api/AbstractIterableAssert.html).

## 文件断言

**文件断言**用于验证文件是否存在、是否可写或可读，并验证其内容。以下是一些文件断言方法：

+   `exists()`: 证明文件或目录存在：

    ```java
    File file = File.createTempFile("test", "txt");
    ```

    ```java
    assertThat(tmpFile).exists();
    ```

+   `isFile()`: 验证给定的对象是否是文件（提供目录将导致测试失败）：

    ```java
    File file = File.createTempFile("test", "txt");
    ```

    ```java
    assertThat(tmpFile).isFile();
    ```

+   `canRead()`: 验证给定的文件是否可由应用程序读取：

    ```java
    File file = File.createTempFile("test", "txt");
    ```

    ```java
    assertThat(tmpFile).canRead();
    ```

+   `canWrite()`: 验证给定的文件是否可由应用程序修改：

    ```java
    File file = File.createTempFile("test", "txt");
    ```

    ```java
    assertThat(tmpFile).canWrite();
    ```

这些只是`AbstractFileAssert`下可用的一些断言。对于完整的文档，您可以访问此处提供的链接：[`joel-costigliola.github.io/assertj/core-8/api/org/assertj/core/api/AbstractFileAssert.html`](https://joel-costigliola.github.io/assertj/core-8/api/org/assertj/core/api/AbstractFileAssert.html).

## 映射断言

**映射断言**用于根据其条目、键和大小检查映射。以下是一些映射断言方法：

+   `contains()`: 验证映射是否包含给定的条目：

    ```java
    Map<name, Hero> heroes = new HashMap<>();
    ```

    ```java
    Heroes.put(stark, iron_man);
    ```

    ```java
    Heroes.put(rogers, captain_america);
    ```

    ```java
    Heroes.put(parker, spider_man);
    ```

    ```java
    assertThat(heroes).contains(entry(stark, iron_man),
    ```

    ```java
      entry(rogers, captain_america));
    ```

+   `containsAnyOf()`: 验证映射是否至少包含一个条目：

    ```java
    Map<name, Hero> heroes = new HashMap<>();
    ```

    ```java
    Heroes.put(stark, iron_man);
    ```

    ```java
    Heroes.put(rogers, captain_america);
    ```

    ```java
    Heroes.put(parker, spider_man);
    ```

    ```java
    assertThat(heroes).contains(entry(stark, iron_man), entry(odinson, thor));
    ```

+   `hasSize()`: 验证映射的大小是否等于给定的值：

    ```java
    Map<name, Hero> heroes = new HashMap<>();
    ```

    ```java
    Heroes.put(stark, iron_man);
    ```

    ```java
    Heroes.put(rogers, captain_america);
    ```

    ```java
    Heroes.put(parker, spider_man);
    ```

    ```java
    assertThat(heroes).hasSize(3);
    ```

+   `isEmpty()`: 验证给定的映射是否为空：

    ```java
    Map<name, Hero> heroes = new HashMap<>();
    ```

    ```java
    assertThat(heroes).isEmpty();
    ```

+   `isNotEmpty()`: 验证给定的映射不为空：

    ```java
    Map<name, Hero> heroes = new HashMap<>();
    ```

    ```java
    Heroes.put(stark, iron_man);
    ```

    ```java
    Heroes.put(rogers, captain_america);
    ```

    ```java
    Heroes.put(parker, spider_man);
    ```

    ```java
    assertThat(heroes).isNotEmpty();
    ```

这些只是`AbstractMapAssert`下可用的一些断言。对于完整的文档，你可以访问这里提供的链接：[`joel-costigliola.github.io/assertj/core-8/api/org/assertj/core/api/AbstractMapAssert.html`](https://joel-costigliola.github.io/assertj/core-8/api/org/assertj/core/api/AbstractMapAssert.html)。

我们已经学习了使用 AssertJ 的不同断言方法；现在，我们将在 Spring Boot 应用程序中实现并编写我们的单元测试。

# 编写测试

在本节中，我们现在将开始在 Spring Boot 应用程序中编写我们的单元测试。当我们回到我们的应用程序时，**服务**和**仓库**是我们应用程序的基本部分，我们需要在这里实现单元测试，因为服务包含业务逻辑并且经常被修改，尤其是在添加新功能时。仓库包括 CRUD 和其他操作的方法。

我们将在编写单元测试时采用两种方法。第一种方法是使用内存数据库，如 H2，在运行单元测试时存储我们创建的数据。第二种方法是使用 Mockito 框架来模拟我们的对象和仓库。

## 使用 H2 数据库进行测试

我们将在编写测试时实施的第一个方法是使用 JUnit 和 AssertJ 与 H2 数据库。H2 数据库是一个内存数据库，允许我们在系统内存中存储数据。一旦应用程序关闭，它将删除所有存储的数据。H2 通常用于**概念验证**或单元测试。

我们已经在*第四章*中添加了 H2 数据库，*设置数据库和 Spring Data JPA*，但如果你错过了这部分，为了我们能够添加 H2 依赖项，我们将在`pom.xml`文件中添加以下内容：

```java
<dependency>
<groupId>com.h2database</groupId> <artifactId>h2</artifactId> <scope>runtime</scope>
</dependency>
```

在成功添加依赖项后，我们将在`test/java`文件夹下添加我们的`h2`配置。我们将添加一个新的资源包并创建一个新的应用程序来完成此操作。我们将使用属性文件进行单元测试，并将以下配置放置在其中：

```java
spring.datasource.url=jdbc:h2://mem:testdb;DB_CLOSE_DELAY=-1
spring.datasource.username={username}
spring.datasource.password={password}
spring.datasource.driver-class-name=org.h2.Driver
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.properties.hibernate.format_sql=true
```

在前面的示例配置中，首先，我们指定我们想要使用`spring.datasource.url`属性将数据存储在`test.mv.db`文件中。我们还可以使用`spring.datasource.username`和`spring.datasource.password`属性覆盖我们的*H2*控制台的用户名和密码，并且我们还指定了在应用程序启动时创建表，在应用程序停止时删除表。

### 测试服务

现在，我们将在我们的`test/java`文件夹下创建一个包。这是我们编写测试的地方。我们将从主文件夹创建一个类似的包。在这种情况下，我们将创建`com.example.springbootsuperheroes.superheroes.antiHero.h2.service`。在新建的包下，我们将创建一个名为`AntiHeroH2ServiceTest`的新类，我们将在这里开始编写对`AntiHeroService`的测试。

我们需要采取的第一个步骤是使用`@DataJpaTest`注解注解我们的类。该注解允许服务通过禁用完整的自动配置并仅应用与测试相关的配置来专注于 JPA 组件。下一步是添加我们的`AntiHeroService`的依赖项，即`AntiHeroRepository`。我们将声明一个新的`AntiHeroRepository`并使用`@Autowired`注解注入依赖项，我们还将声明`AntiHeroService`，因为这是我们需要测试的服务。我们将有以下的代码：

```java
@DataJpaTest
public class AntiHeroH2ServiceTest {
    @Autowired
    private AntiHeroRepository repo;
    private AntiHeroService service;
}
```

在注入我们的依赖项并注解我们的类之后，我们接下来想要考虑的是在运行每个测试之前我们想要有哪些可能的属性；在这种情况下，我们希望在运行测试用例之前创建一个`AntiHeroService`的实例。为了实现这一点，我们将创建一个带有`@BeforeEach`注解的方法，并使用`AutoWired AntiHeroRepository`作为参数创建一个新的`AntiHeroService`实例：

```java
@BeforeEach
public void setup() {
    service = new AntiHeroService(repo);
}
```

现在，我们可以为我们的服务编写一个测试用例；我们的目标是编写对`AntiHeroService`拥有的每个方法的测试。

让我们看看`AntiHeroService`的方法列表：

+   `Iterable<AntiHeroEntity> findAllAntiHeroes`: 应返回反英雄的列表。

+   `AntiHeroEntity addAntiHero(AntiHeroEntity antiHero)`: 应添加一个新的反英雄实体。

+   `void updateAntiHero(UUID id, AntiHeroEntity antiHero)`: 应根据给定的 ID 更新反英雄。

+   `AntiHeroEntity findAntiHeroById(UUID id)`: 应返回具有给定 ID 的反英雄；如果未找到，则返回`NotFoundException`。

+   `void removeAntiHeroById(UUID id)`: 应根据给定的 ID 从数据库中删除反英雄。

让我们先为`findAllAntiHeroes()`方法编写一个测试。该方法的可能测试用例是检查方法是否成功检索数据库中的所有反英雄。为了测试这个场景，我们首先需要将单个实体或一系列测试反英雄实体添加到我们的 H2 数据库中。我们可以调用`findAllAntiHeroes()`方法来检索数据库中刚添加的实体。让我们看看下面的示例单元测试：

```java
@Test
public void shouldFindAllAntiHero() {
    AntiHeroEntity antiHero = new AntiHeroEntity();
    antiHero.setFirstName("Eddie");
    antiHero.setLastName("Brock");
    antiHero.setHouse("MCU");
    service.addAntiHero(antiHero);
    Iterable<AntiHeroEntity> antiHeroList =
      service.findAllAntiHeroes();
    AntiHeroEntity savedAntiHero =
      antiHeroList.iterator().next();
    assertThat(savedAntiHero).isNotNull();
}
```

在前面的代码示例中，我们可以看到我们首先创建了一个新的反英雄实例，作为数据库内存中的示范数据。我们使用`addAntiHero()`方法将数据添加到我们的数据库中。在成功插入数据后，我们可以使用`findAllAntiHeroes()`方法检查或断言是否可以检索到新创建的反英雄。在这个场景中，我们检索了反英雄列表中的第一条数据。我们使用`assertThat(savedAntiHero).isNotNull()`来验证列表的第一个元素不是 null。

现在，让我们为`addAntiHero()`方法编写一个测试。我们将为以下方法创建的测试与为`findAllAntiHeroes()`方法创建的测试大致相似。该方法的可能测试用例是检查实体是否成功添加到我们的数据库中。

让我们看看以下示例单元测试：

```java
@Test
public void shouldAddAntiHero() {
    AntiHeroEntity antiHero = new AntiHeroEntity();
    antiHero.setFirstName("Eddie");
    antiHero.setLastName("Brock");
    antiHero.setHouse("MCU");
    service.addAntiHero(antiHero);
    Iterable<AntiHeroEntity> antiHeroList =
      service.findAllAntiHeroes();
    AntiHeroEntity savedAntiHero =
      antiHeroList.iterator().next();
    assertThat(antiHero).isEqualTo(savedAntiHero);
}
```

在前面的代码示例中，我们创建了一个新的反英雄实体，并使用`addAntiHero()`方法将其插入到我们的数据库中。在添加最新数据后，我们可以检索列表并验证我们的新数据是否在数据库中。在给定场景中，我们检索了反英雄列表中的第一条数据，并使用`assertThat(antiHero).isEqualTo(savedAntiHero);`来检查我们检索到的数据是否与我们实例化的数据相等。

接下来，现在让我们编写`updateAntiHeroMethod();`的测试。该方法的可能测试用例是检查该方法是否成功修改了我们数据库中特定实体的信息。

让我们看看满足此测试用例的示例单元测试：

```java
@Test
public void shouldUpdateAntiHero() {
    AntiHeroEntity antiHero = new AntiHeroEntity();
    antiHero.setFirstName("Eddie");
    antiHero.setLastName("Brock");
    antiHero.setHouse("MCU");
    AntiHeroEntity savedAntiHero  =
      service.addAntiHero(antiHero);
    savedAntiHero.setHouse("San Francisco");
    service.updateAntiHero(savedAntiHero.getId(),
                           savedAntiHero);
    AntiHeroEntity foundAntiHero =
      service.findAntiHeroById(savedAntiHero.getId());
    assertThat(foundAntiHero.getHouse()).isEqualTo(
      "San Francisco");
}
```

在前面的代码示例中，我们创建了一个新的反英雄实体，并使用`addAntiHero()`方法将其插入到我们的数据库中。在添加实体后，我们更新了添加的反英雄的住宅信息为`"San Francisco"`，并使用`updateAntiHeroMethod()`将其保存到数据库中。最后，我们使用其 ID 检索了修改后的反英雄，并通过添加`assertThat(foundAntiHero.getHouse()).isEqualTo("San` `Francisco");`断言来验证住宅信息是否已修改。

接下来，我们现在将为`removeAntiHeroById()`方法创建一个单元测试。该方法可能的测试用例是验证具有相应 ID 的实体是否已成功从数据库中删除。

让我们看看满足此测试用例的示例单元测试：

```java
@Test
public void shouldDeleteAntiHero() {
    assertThrows(NotFoundException.class, new Executable() {
        @Override
        public void execute() throws Throwable {
            AntiHeroEntity savedAntiHero  =
              service.addAntiHero(antiHero);
            service.removeAntiHeroById(
              savedAntiHero.getId());
            AntiHeroEntity foundAntiHero =
              service.findAntiHeroById(
                savedAntiHero.getId());
            assertThat(foundAntiHero).isNull();
        }
    });
}
```

在前面的例子中，我们可以看到我们在编写单元测试时添加了一些额外的元素；我们创建了一个新的`Executable()`实例，其中放置了我们的主要代码。我们用`NotFoundException.class`断言了我们的`Executable()`。这样做的主要原因是我们期望`findAntiHeroByID()`将返回`NotFoundException`错误，因为我们已经从我们的数据库中删除了实体。

记住，在断言错误时，我们应该使用`assertThrows()`。

我们已经成功为我们的服务编写了测试，现在，我们将实现仓库级别的单元测试。

## 测试仓库

为我们的应用程序的仓库编写测试与我们在服务级别编写测试的方式大致相同；我们也把它们当作服务来对待，并在仓库中添加了额外的方法时对它们进行测试。

我们将采用的例子是编写我们的`UserRepository`的单元测试。让我们回顾一下`UserRepository`拥有的方法：

+   `Boolean selectExistsEmail(String email)`: 当用户存在给定邮箱时返回`true`

+   `UserEntity findByEmail(String email)`: 当给定邮箱存在于数据库中时返回用户

要开始编写我们的测试，首先，我们将在`com.example.springbootsuperheroes.superheroes`包下创建一个名为`user.repository`的新包，并创建一个名为`UserRepositoryTest`的新类。在成功创建仓库后，我们将使用`@DataJPATest`注解该类，使其仅关注 JPA 组件，并使用`@Autowired`注解注入`AntiHeroRepostiory`。

我们现在的类将如下所示：

```java
@DataJpaTest
class UserRepositoryTest {
    @Autowired
    private UserRepository underTest;
}
```

现在，在成功注入仓库后，我们可以编写我们的测试。首先，我们想要为`selectExistsEmail()`方法编写一个测试。该方法的可能测试用例是，如果邮箱存在于我们的数据库中，它应该返回`true`。

让我们看看以下示例代码：

```java
@Test
void itShouldCheckWhenUserEmailExists() {
    // give
    String email = "seiji@gmail.com";
    UserEntity user = new UserEntity(email, "21398732478");
    underTest.save(user);
    // when
    boolean expected = underTest.selectExistsEmail(email);
    // then
    assertThat(expected).isTrue();
}
```

在示例单元测试中，我们已经将一个示例用户实体添加到我们的数据库中。`selectExistsEmail()`方法预期将返回`true`。这应该检索给定邮箱添加的用户。

下一个测试是为`findByEmail()`方法；这几乎与为`selectExistsEmail()`方法创建的测试相同。唯一需要修改的是断言，因为我们期望返回`User`类型的值。

让我们看看以下示例代码：

```java
@Test
void itShouldFindUserWhenEmailExists() {
    // give
    String email = "dennis@gmail.com";
    UserEntity user = new UserEntity(email, "21398732478");
    underTest.save(user);
    // when
    UserEntity expected = underTest.findByEmail(email);
    // then
    assertThat(expected).isEqualTo(user);
}
```

我们已经使用 JUnit、AssertJ 和 H2 数据库成功为我们的服务和仓库编写了测试。在下一节中，我们将使用 JUnit 和 AssertJ 的第二个实现来编写单元测试，并使用 Mockito。

# 使用 Mockito 在服务中编写测试

在前面的章节中，我们使用 H2 数据库创建了我们的单元测试；在这个方法中，我们将完全省略数据库的使用，并在单元测试中利用模拟的概念来创建样本数据。我们将通过使用 **Mockito** 来实现这一点。Mockito 是一个 Java 模拟框架，允许我们隔离测试类；它不需要任何数据库。

这将使我们能够从模拟的对象或服务中返回虚拟数据。Mockito 非常有用，因为它使得单元测试变得更加简单，尤其是在大型应用程序中，因为我们不想同时测试服务和依赖项。以下是使用 Mockito 的其他好处：

+   **支持返回值**：支持模拟返回值。

+   **支持异常**：可以在单元测试中处理异常。

+   **支持注解**：可以使用注解创建模拟。

+   **安全于重构**：重命名方法名称或更改参数的顺序不会影响测试，因为模拟是在运行时创建的。

让我们探索 Mockito 的不同功能，以编写单元测试。

## 添加行为

Mockito 包含 `when()` 方法，我们可以用它来模拟对象的返回值。这是 Mockito 最有价值的功能之一，因为我们可以为服务或仓库定义一个虚拟的返回值。

让我们看看以下代码示例：

```java
public class HeroTester {
   // injects the created Mock
   @InjectMocks
   HeroApp heroApp = new HeroApp();
   // Creates the mock
   @Mock
   HeroService heroService;
   @Test
   public void getHeroHouseTest(){
      when(heroService.getHouse())).thenReturn(
        "San Francisco ");
   assertThat(heroApp.getHouse()).isEqualTo(
     "San Francisco");
 }
}
```

在前面的代码示例中，我们可以看到我们在测试中模拟了 `HeroService`。我们这样做是为了隔离类，而不是测试 `Heroservice` 本身的功能；我们想要测试的是 `HeroApp` 的功能。我们通过指定模拟返回 `thenReturn()` 方法为 `heroService.getHouse()` 方法添加了行为。在这种情况下，我们期望 `getHouse()` 方法返回 `"San Francisco"` 的值。

## 验证行为

我们可以从 Mockito 中使用的下一个功能是单元测试中的行为验证。这允许我们验证模拟的方法是否被调用并带有参数执行。这可以通过使用 `verify()` 方法来实现。

让我们以相同的类示例为例：

```java
public class HeroTester {
   // injects the created Mock
   @InjectMocks
   HeroApp heroApp = new HeroApp();
   // Creates the mock
   @Mock
   HeroService heroService;
   @Test
   public void getHeroHouseTest(){
      when(heroService.getHouse())).thenReturn(
        "San Francisco ");
   assertThat(heroApp.getHouse()).isEqualTo(
     "San Francisco");
   verify(heroService).getHouse();
 }
}
```

在前面的代码示例中，我们可以看到我们在代码中添加了 `verify(heroService).getHouse()`。这验证了我们是否调用了 `getHouse()` 方法。我们还可以验证该方法是否带有一些给定的参数被调用。

## 期望调用

`times(n)` 方法。同时，我们还可以使用 `never()` 方法验证它是否被调用。

让我们看看以下示例代码：

```java
public class HeroTester {
   // injects the created Mock
   @InjectMocks
   HeroApp heroApp = new HeroApp();
   // Creates the mock
   @Mock
   HeroService heroService;
   @Test
   public void getHeroHouseTest(){
     // gets the values of the house
     when(heroService.getHouse())).thenReturn(
       "San Francisco ");
    // gets the value of the name
    when(heroService.getName())).thenReturn("Stark");
   // called one time
   assertThat(heroApp.getHouse()).isEqualTo(
     "San Francisco");
   // called two times
   assertThat(heroApp.getName()).isEqualTo("Stark");
   assertThat(heroApp.getName()).isEqualTo("Stark");
   verify(heroService, never()).getPowers();
   verify(heroService, times(2)).getName();
 }
}
```

在前面的代码示例中，我们可以看到我们使用了 `times(2)` 方法来验证 `heroService` 的 `getName()` 方法是否被调用了两次。我们还使用了 `never()` 方法，它检查 `getPowers()` 方法是否没有被调用。

Mockito 除了 `times()` 和 `never()` 之外，还提供了其他方法来验证预期的调用次数，这些方法如下：

+   `atLeast (int min)`: 验证方法是否至少被调用*n*次

+   `atLeastOnce ()`: 验证方法是否至少被调用一次

+   `atMost (int max)`: 验证方法是否最多被调用*n*次

## 异常处理

Mockito 还在单元测试中提供了异常处理；它允许我们在模拟上抛出异常以测试应用程序中的错误。

让我们看看下面的示例代码：

```java
public class HeroTester {
   // injects the created Mock
   @InjectMocks
   HeroApp heroApp = new HeroApp();
   // Creates the mock
   @Mock
   HeroService heroService;
   @Test
   public void getHeroHouseTest(){
   doThrow(new RuntimeException("Add operation not
           implemented")).when(heroService.getHouse()))
   .thenReturn("San Francisco ")
  assertThat(heroApp.getHouse()).isEqualTo(
    "San Francisco");
 }
}
```

在前面的示例中，我们配置了`heroService.getHouse()`，一旦被调用，就抛出`RunTimeException`。这将允许我们测试并覆盖应用程序中的错误块。

我们已经了解了 Mockito 中可用的不同功能。现在，让我们继续在我们的 Spring Boot 应用程序中编写我们的测试。

## Spring Boot 中的 Mockito

在本节中，我们现在将在我们的 Spring Boot 应用程序中实现 Mockito 以编写单元测试。我们将再次为我们的服务编写测试，并在我们的`test/java`文件夹下创建另一个包，该包将用于使用 Mockito 进行单元测试；我们将创建`com.example.springbootsuperheroes.superheroes.antiHero.service`。在新创建的包下，我们将创建一个名为`AntiHeroServiceTest`的新类，我们将开始编写对`AntiHeroService`的测试。

在成功创建我们的类之后，我们需要使用`@ExtendWith(MockitoExtension.class)`注解我们的类，以便能够使用 Mockito 的方法和功能。下一步是模拟我们的`AntiHeroRepository`并将其注入到我们的`AntiHeroRepositoryService`中。为了完成这个任务，我们将使用`@Mock`注解来声明仓库，并使用`@InjectMocks`注解来声明服务，此时我们的类将如下所示：

```java
@ExtendWith(MockitoExtension.class)
class AntiHeroServiceTest {
    @Mock
    private AntiHeroRepository antiHeroRepository;
    @InjectMocks
    private AntiHeroService underTest;
}
```

在前面的示例中，我们成功模拟了我们的仓库并将其注入到我们的服务中。现在，我们可以开始在测试中模拟仓库的返回值和行为。

让我们在`AntiHeroService`中添加一些示例测试；在一个示例场景中，我们将为`addAntiHero()`方法编写一个测试。这个测试的可能用例是验证仓库中的`save()`方法是否被调用，并且反英雄是否成功添加。

让我们看看下面的示例代码：

```java
@Test
void canAddAntiHero() {
    // given
    AntiHeroEntity antiHero = new AntiHeroEntity(
            UUID.randomUUID(),
            "Venom",
            "Lakandula",
            "Tondo",
            "Datu of Tondo",
            new SimpleDateFormat(
              "dd-MM-yyyy HH:mm:ss z").format(new Date())
    );
    // when
    underTest.addAntiHero(antiHero);
    // then
    ArgumentCaptor<AntiHeroEntity>
    antiHeroDtoArgumentCaptor =
      ArgumentCaptor.forClass(
            AntiHeroEntity.class
    );
    verify(antiHeroRepository).save(
      antiHeroDtoArgumentCaptor.capture());
    AntiHeroEntity capturedAntiHero =
      antiHeroDtoArgumentCaptor.getValue();
    assertThat(capturedAntiHero).isEqualTo(antiHero);
}
```

在前面的示例中，第一步始终是创建一个样本实体，我们可以将其用作添加新反英雄的参数；在调用我们正在测试的`addAntiHero()`方法之后，我们使用`verify()`方法验证了`AntiHeroRepository`的`save()`方法是否被调用。

我们还使用了`ArgumentCaptor`来捕获我们之前使用的方式中使用的参数值，这些值将被用于进一步的断言。在这种情况下，我们断言捕获的反英雄等于我们创建的反英雄实例。

# 概述

通过这些，我们已经到达了本章的结尾。让我们回顾一下你学到的宝贵知识；你学习了 JUnit 的概念，JUnit 是一个提供诸如固定值、测试套件和用于测试我们应用中方法的类等功能的测试框架。你还学习了 JUnit 中 AssertJ 的应用，它为我们的单元测试提供了更灵活的断言对象的方式；最后，你还学习了 Mockito 的重要性，它为我们提供了模拟对象和服务的能力。

在下一章中，我们将使用 Angular 开发我们的前端应用程序。我们将讨论如何组织我们的功能和模块，在 Angular 文件结构中构建我们的组件，并将 Angular Material 添加到用户界面中。

# 第三部分：前端开发

本部分包含了一个开发 Angular 13 应用程序的真实场景。以下章节包含在本部分中：

+   *第十章*, *设置我们的 Angular 项目和架构*

+   *第十一章*, *构建响应式表单*

+   *第十二章*, *使用 NgRx 管理状态*

+   *第十三章*, *使用 NgRx 进行保存、删除和更新*

+   *第十四章*, *在 Angular 中添加身份验证*

+   *第十五章*, *在 Angular 中编写测试*
