## 第七章。放心，试驾一下

应用开发是一个漫长、耗时且成本高昂的过程。开发依赖于从客户和市场收集的需求。但是，如果在工作完成后出了问题，一切都会崩溃呢？冲突并非由于解决方案错误，而是因为开发者在工作开始前基于错误的假设。这种冲突恰好在向客户交付日期前发生。现在什么都无法挽回了！我们不必深究为什么会出现问题和具体情况。但我感兴趣的是，这种情况可以避免吗？有没有什么方法可以在最后一刻避免这种冲突呢？我们总是听说“预防胜于治疗”。这个原则也适用于应用开发。通过开发人员逐步付出的少许额外努力，可以避免失败的状况。开发人员开发的代码进行交叉检查，以满足需求，这有助于确保代码的正确运行。这种交叉检查称为应用测试。在本章中，我们将通过以下几点深入讨论测试：

+   为什么测试？

+   测试 Spring 控制器的问题。

+   模拟测试。

+   春季测试上下文框架。

+   使用 Mokitoto 测试 Spring 控制器。

+   使用 Arquillian 介绍 Spring 控制器测试。

## 测试是一个重要的步骤

***

应用开发是一个昂贵且耗时长的过程。在最后的部署中出现的错误和失误会导致非常严重的后果。开发者根据需求编写的代码基于一些可能基于一些假设的规则。作为人类，我们可能在需求收集或假设制定上犯错误。如果这是我们完成的工作，还有谁比我们更了解它呢？单元测试测试代码，并帮助确保其正常运行。

开发者完成了开发。他们的开发基于一些假设，他们可能也会遗漏一些盲点。开发之后，他们进行测试。由同一个人进行测试是高风险的，因为他们可能会重复同样的错误。理想情况下，应该是其他人来做检查，确保他们知道他们在测试什么。

以下是一些使测试成为应用开发中难忘的一部分的主要因素：

+   它有助于尽早发现开发过程中产生的缺陷和错误。

+   它确保应用程序执行中的失败次数最少

+   它有助于提高应用程序的一致性

+   它有助于确保更好的应用程序质量

+   它通过检查认证和授权来提高安全性

+   帮助节省金钱，更重要的是节省时间

每个应用程序在发布之前都要经过严格的测试，以确保应用程序符合要求并确保其所有功能正确无误。单元测试、集成测试、系统测试和验收测试是每个应用程序必须通过的四个主要阶段。

### 单元测试

单元测试关注组件的单元，确保功能的正确性。单元可以指单个函数或过程。单元测试的主要目的是确保单元按设计工作。它允许快速解决提出的问题。由于单元是应用程序的最小部分，因此代码可以很容易地修改。通常由编写代码的开发者进行。

### 集成测试

一旦单元测试成功完成，测试单元时出现的大部分问题都已经修改以符合要求。集成测试提供了在程序执行内测试这些单元组的机会。它有助于确定多个单元是如何一起运行的。单元可能运行良好，但当与其他单元结合时，相同的单元可能会导致一些副作用，需要解决。集成测试有助于捕获此类错误，并有机会进行更正。

### 系统测试

在前两个阶段，已经对单个单元或单元之间的相互交互进行了测试。这是第一次全面测试完整应用程序的阶段。系统测试通常由独立测试员在接近生产环境中进行。系统测试确保应用程序开发的所有功能和业务要求是否已经满足。

### 用户验收测试

这是测试的最后阶段，它确定系统是否准备好最终发布。验收测试通常由最终用户执行，以确定应用程序符合要求并涵盖了所有必要的功能以给出最终验收。它使最终应用程序在生产环境中有了预览。

本章我们将分三个阶段进行单元测试、集成测试和系统测试。但在前进之前，让我们先了解一下市场上可用的测试工具的概况。

## 测试工具

* * *

以下是适用于 Java 平台的可用测试工具，

### JTest

JTest 是由 Parasoft 自 1997 年以来为 Java 平台开发的自动化软件测试、编码标准合规工具。该工具利用单元测试以及集成测试。该工具便于分析类，以与 JUnit 测试用例相同的格式生成和执行测试用例。

以下是一些 JTest 功能：

+   除了测试之外，它还涵盖了正常情况下开发者无法捕获的运行时异常。

+   该工具还验证类是否遵循**契约式设计**（**DbC**）基础。

+   它确保代码遵循 400 个标准编码规则，并将代码与 200 个违规规则进行比对。

+   它还可以识别功能错误、内存泄漏和安全漏洞等问题。

+   **Jcontract** 是 JTest 工具的一部分，它在集成测试期间验证功能需求，而不会影响应用程序的性能。

### **Grinder**

**Grinder** 是一个为 Java 编程语言设计的负载测试工具，遵循 BSD 风格的开放源代码许可。它的目标是简化使用负载注入机器进行的分布式测试。它具备负载测试、能力测试、功能测试和压力测试的能力。它对系统资源的要求最低，同时在其测试上下文中管理自己的线程，如果需要可以将其分割到不同的进程。

以下是 Grinder 的特点：

+   易于使用的基于 Java Swing 的用户界面

+   它可以用于具有 Java API 的任何负载测试。它可以用于 Web 服务器、基于 SOAP 和 Rest API 的 Web 服务、应用服务器等。

+   Jython 和 Clojure 语言支持编写灵活、动态的测试脚本。

+   它还管理客户端连接和 cookie。

### **JWalk**

**JWalk** 是一个为 Java 平台设计的单元测试工具，支持懒惰系统性单元测试范式。它由 Anthony Simons 开发。JWalk 通过“懒惰规格”和“系统性测试”的概念来测试单个类并生成测试报告。它更适合敏捷开发，在这种开发中不需要产生正式的规格说明。通过构建和展示自动化测试用例，它能节省大量时间和精力。

以下是 JWalk 的特点：

+   系统性地提出所有可能的测试用例。

+   测试人员无需确认测试结果的子集。

+   可以预测测试结果。

+   如果类被修改，它会生成新的测试用例。

+   适用于软件开发中的极限编程的 TDD。

### **PowerMock**

**PowerMock** 是一个开源项目，作为 EasyMock 和 Mokito 框架的扩展，通过添加一些方法和注解来实现。它允许从 Java 代码中创建模拟对象的实现。有时应用程序的架构是这样设计的，它使用最终类、私有方法或静态方法来设计类。这些方法或类无法测试，因为无法创建它们的模拟对象。开发者可以选择良好的设计或可测试性。PowerMock 通过使用自定义类加载器和字节码操作，使静态方法和最终类可以被模拟。

### **TestNG**

TestNG 是一个受 JUnit 和 NUnit 测试启发的强大测试框架，适用于单元测试、功能测试和集成测试。它支持参数化测试，这是 JUnit 不可能实现的。它配备了诸如每个测试方法（@BeforeMethod, @AfterMethod）和每个类（@BeforeClass, @AfterClass）之前和之后的数据预处理等许多有用注解。

以下 是 TestNG 的功能：

+   易于编写测试用例

+   它可以生成 HTML 报告

+   它可以生成日志

+   良好的集成测试支持

### Arquillian Framework

Arquillian 是一个针对 Java 应用程序的测试框架。该框架使开发人员能够在运行时环境部署应用程序，以使用 JUnit 和 TestNG 执行测试用例。由于 Arquillian 管理以下测试生命周期管理事物，因此可以在测试内部管理运行时环境：

+   它可以管理多个容器

+   它使用 ShrinkWrap 捆绑类、资源和测试用例

+   它将归档部署到容器中

+   在容器内执行测试用例

+   将结果返回给测试运行器

#### ShrinkWrap

该框架由三个主要组件组成，

##### 测试运行器

执行测试用例时，JUnit 或 TestNG 使用 Arquillian 测试运行器。这使得在测试用例中使用组件模型成为可能。它还管理容器生命周期和依赖注入，使模型可供使用。

##### Java 容器

Java 容器是测试环境的主要组件。Arquillian 测试可以在任何兼容的容器中执行。Arquillian 选择容器以确定在类路径中可用的哪个容器适配器。这些容器适配器控制并帮助与容器通信。Arquillian 测试用例甚至可以在没有基于 JVM 的容器的情况下执行。我们可以使用**@RunsClientto**注解在 Java 容器外部执行测试用例。

##### 将测试用例集成到 Java 容器中

该框架使用名为 ShrinkWrap 的外部依赖。它有助于定义要加载到 Java 容器中的应用程序的部署和描述符。测试用例针对这些描述符运行。Shrinkwrap 支持生成动态 Java 归档文件，类型为 JAR、WAR 和 EAR。它还可以用于添加部署描述符以及创建 DD 程序化。

Arquillian 可以在以下场景中使用，

+   您要测试的应用程序部分需要在内嵌服务器中部署

+   测试应在每小时、一定时间间隔后或有人提交代码时执行

+   通过外部工具自动化应用程序的验收测试

### JUnit

JUnit 是用于 Java 测试驱动开发的最受欢迎的开源框架。JUnit 有助于对组件进行单元测试。它还广泛支持诸如 ANT、Maven、Eclipse IDE 等工具。单元测试类是像其他任何类一样的普通类，主要区别在于使用**@Test**注解。@Test 注解让 JUnit 测试运行器知道需要执行这个注解的方法来进行测试。

org.junit.Assert 类提供了一系列静态的 assertXXX()方法，这些方法通过比较被测试方法的预期输出和实际输出来进行测试。如果测试的比较返回正常，表示测试通过了。但是，如果比较失败，执行停止，表示测试失败。

单元测试类通常被称为单元测试用例。测试用例可以有多个方法，这些方法将按照编写顺序一个接一个地执行。JUnit 为公共单元测试提供设置测试数据的功能，并针对它进行测试。数据的初始化可以在`setUp()`方法中完成，或者在用@Before 注解标记的方法中完成。默认情况下，它使用 JUnit 运行器来运行测试用例。但它还有 Suite、Parameterised 和 Categories 等几个内置的运行器。除了这些运行器之外，JUnit 还支持第三方运行器，如 SpringJUnit4ClassRunner、MokitoJUnitRunner、HierarchicalContextRunner。它还支持使用@RunWith 注解，以便使用自定义运行器。我们将在稍后详细讨论这个注解以及 Spring 测试框架。

以下是一些通过比较进行测试的断言方法，

+   assertEquals : 这个方法通过调用 equals()方法来测试两个对象的相等性。

+   assertTrue 和 assertFalse : 它用于将布尔值与 true 或 false 条件进行比较。

+   assertNull 和 assetNotNull : 这个方法测试值的 null 或非 null。

+   assertSame 和 assertNotSame : 它用于测试传递给它的两个引用是否指向同一个对象。

+   assertArrayEquals : 它用于测试两个数组是否包含相等的元素，并且数组中每个元素与另一个数组中相同索引的元素相等。

+   assertThat : 它用于测试对象是否与 org.hamcrest.Matcher 中的对象匹配。

## 第一阶段 单元测试 DAO 使用 JUnit 进行单元测试

* * *

现在，是编写实际测试用例的时候了。我们将从单元测试 DAO 层开始。以下是为编写基于注解的测试用例而遵循的一般步骤，

1.  创建一个类，该类的名称以'Test'为前缀，紧跟被测试类的名称。

1.  为初始化我们所需的数据和释放我们使用的资源分别编写`setUp()`和`testDown()`方法。

1.  进行测试的方法将它们的名称命名为被测试方法名称前加上'test'。

1.  `4.` 测试运行器应该认识的方法的需用`@Test`注解标记。

1.  使用`assertXXX()`方法根据测试的数据比较值。

让我们为第三章中开发的 DAO 层编写测试。我们将使用 Ch03_JdbcTemplates 作为基础项目。您可以创建一个新的项目，或者通过仅添加测试包来使用 Ch03_JdbcTemplates。让我们按照以下步骤操作：

### 创建基本应用程序。

1.  创建 Ch07_JdbcTemplates_Testing 作为 Java 项目。

1.  为 Spring 核心、Spring JDBC 和 JDBC 添加所有必需的 jar 文件，这些文件我们已经为 Ch03_JdbcTemplates 项目添加了。

1.  从基础项目中复制 com.packt.ch03.beans 和 com.packt.ch03.dao 包。我们只对 BookDAO_JdbcTemplate 类进行测试。

1.  将`connection_new.xml`复制到类路径中

### 执行测试

1.  创建`com.packt.ch07.tests`包

1.  使用 Eclipse IDE 中的 JUnit 测试用例模板：

    1.  输入测试用例的名称 TestBookDAO_JdbcTemplate

    1.  为初始化和释放测试用例组件选择 setUp 和 teardown 复选框。

    1.  点击浏览按钮，选择 BookDAO_JdbcTemplate 作为测试类。

    1.  点击下一步按钮

    1.  在测试方法对话框中选择 BookDAO_JdbcTemplate 类中的所有方法。

    1.  点击完成。

    1.  将出现一个对话框，询问是否在构建路径上添加 JUnit4。点击确定按钮。

以下图表总结了这些步骤：

![](img/image_07_001.png)

1.  点击下一步按钮后，您将看到下一个对话框：

![](img/image_07_002.png)

1.  在测试用例中声明一个数据成员作为`BookDAO_JdbcTemplate`。

1.  更新`setUp()`方法，使用 ApplicationContext 容器初始化测试用例的数据成员。

1.  更新`tearDown()`以释放资源。

1.  更新`testAddBook()`如下：

1.  创建一个 Book 类型的对象，并确保 ISBN 的值在 Book 表中不可用。

1.  从`BookDAO_JdbcTemplate`类调用`addBook()`。

1.  使用以下代码中的`assertEquals()`方法测试结果：

```java
      public classTestBookDAO_JdbcTemplate { 
        BookDAO_JdbcTemplatebookDAO_JdbcTemplate; 

        @Before 
        publicvoidsetUp() throws Exception { 
          ApplicationContextapplicationContext = new 
          ClassPathXmlApplicationContext("connection_new.xml"); 
          bookDAO_JdbcTemplate = (BookDAO_JdbcTemplate)  
          applicationContext.getBean("bookDAO_jdbcTemplate"); 
        } 
        @After 
        publicvoidtearDown() throws Exception { 
          bookDAO_JdbcTemplate = null; 
        } 

        @Test 
        publicvoidtestAddBook() { 
          Book book = newBook("Book_Test", 909090L, "Test  
          Publication", 1000, "Test Book description", "Test  
          author"); 
          introws_insert= bookDAO_JdbcTemplate.addBook(book); 
          assertEquals(1, rows_insert); 
        } 
      } 

```

1.  选择`testAddBook()`方法并将其作为 JUnit 测试运行。

1.  如以下图表所示，JUnit 窗口将显示一个绿色标记，表示代码已通过单元测试：

![](img/image_07_003.png)

1.  在 Book 表中，ISBN 是一个主键，如果你重新运行相同的`testAddBook()`，它将显示红色而不是绿色，从而失败。尽管如此，这证明了代码是根据逻辑工作的。如果测试条件之一失败，测试用例执行将停止，并显示断言错误。

### 注意

尝试编写一个总是通过的测试条件。

1.  让我们添加`TestAddBook_Negative ()`以测试如果我们尝试添加具有相同 ISBN 的书籍会发生什么。不要忘记通过`@Test`注解 annotate the method。代码将如下所示：

```java
      @Test(expected=DuplicateKeyException.class) 
      publicvoidtestAddBook_Negative() { 
        Book book = newBook("Book_Test", 909090L, "Test  
        Publication", 1000, "Test Book description", "Test  
        author"); 
        introws_insert= bookDAO_JdbcTemplate.addBook(book); 
        assertEquals(0, rows_insert); 
      } 

```

### 注意

如果添加重复键，代码将抛出 DuplicateKeyException。在`@Test`注解中，我们添加了`DuplicateKey`Exception 作为期望的结果，指示 JUnit 运行器这是期望的行为。

1.  同样，让我们将以下代码添加到其他测试方法中：

```java
      @Test 
      publicvoidtestUpdateBook() { 
        //with ISBN which does exit in the table Book 
        longISBN = 909090L; 
        intupdated_price = 1000; 
        introws_insert = bookDAO_JdbcTemplate.updateBook(ISBN,  
          updated_price); 
        assertEquals(1, rows_insert); 
      } 
      @Test 
      publicvoidtestUpdateBook_Negative() { 
        // code for deleting the book with ISBN not in the table 
      } 
      @Test 
      publicvoidtestDeleteBook() { 
        // with ISBN which does exit in the table Book 
        longISBN = 909090L; 
        booleandeleted = bookDAO_JdbcTemplate.deleteBook(ISBN); 
        assertTrue(deleted); 
      } 
      @Test 
      publicvoidtestDeleteBook_negative() { 
        // deleting the book with no iSBN present in the table. 
      } 
      @Test 
      publicvoidtestFindAllBooks() { 
        List<Book>books =  
        bookDAO_JdbcTemplate.findAllBooks(); 
        assertTrue(books.size()>0); 
        assertEquals(4, books.size()); 
        assertEquals("Learning Modular Java  
        Programming",books.get(3).getBookName()); 
      } 
      @Test 
      publicvoidtestFindAllBooks_Author() { 
        List<Book>books =  
          bookDAO_JdbcTemplate.findAllBooks("T.M.Jog"); 
        assertEquals("Learning Modular Java  
          Programming",books.get(1).getBookName()); 
      } 

```

上述代码构建了几个对象，如 `BookDAO_JdbcTemplate`，这些对象是使用 Spring 容器构建的。在代码中，我们使用了在 `setUp()` 中通过 Spring 容器获得的 `BookDAO_JdbcTemplate` 对象。我们不能手动完成，而有更好的选择吗？是的，我们可以通过使用 Spring 提供的自定义运行器来实现。SprinJUnit4ClassRunner 是一个自定义运行器，它是 JUnit4Runner 类的扩展，提供了一个使用 Spring TestContext Framework 的设施，消除了复杂性。

### Spring TestContext Framework

Spring 为开发者提供了丰富的 Spring TestContext Framework，该框架为单元测试和集成测试提供了强大的支持。它支持基于 API 的和基于注解的测试用例创建。该框架强烈支持 JUnit 和 TestNG 作为测试框架。TestContext 封装了将执行测试用例的 spring 上下文。如果需要，它还可以用于加载 ApplicationContext。TestContextManager 是管理 TestContext 的主要组件。TestContextManager 通过事件发布，而 TestExecutionListener 为发布的事件提供采取的动作。

类级注解 @RunWith 指示 JUnit 调用其引用的类来运行测试用例，而不是使用内置的运行器。Spring 提供的 SpringJUnit4ClassRunner 使 JUnit 能够使用 TestContextManager 提供的 Spring 测试框架功能。org.springframework.test.context 包提供了测试的注解驱动支持。以下注解用于初始化上下文，

#### @ContextConfiguration

类级注解加载了构建 Spring 容器的定义。上下文是通过引用一个类或 XML 文件来构建的。让我们逐一讨论它们：

+   使用单个 XML 文件：

```java
      @ContextConfiguration("classpath:connection_new.xml") 
      publicclassTestClass{ 
        //code to test 
      } 

```

+   使用配置类：

```java
      @ContextConfiguration(class=TestConfig.class) 
      publicclassTestClass{ 
        //code to test 
      } 

```

+   使用配置类以及 XML 文件：

```java
      @ContextConfiguration(locations="connection_new.xml", 
      loader=TestConfig.class) 
      publicclassTestClass{ 
        //code to test 
      } 

```

+   使用上下文初始化器：

```java
      @ContextConfiguration(initializers = 
        TestContextInitializer.class) 
      publicclassTestClass{ 
        //code to test 
      } 

```

#### @WebAppConfiguration

类级注解用于指示如何加载 ApplicationContext，并由默认位置的 WebApplicationContext（WAC）使用，文件路径为 "file:/src/main/webapp"。以下代码段显示了加载资源以初始化用于测试的 WebApplicationContext：

```java
@WebAppConfiguration("classpath: myresource.xml") 
publicclassTestClass{ 
 //code to test 
} 

```

之前开发的测试用例使用显式初始化 Spring 上下文。在这个示例中，我们将讨论如何使用 SprinJUnit4ClassRunner 和 @RunWith。我们将使用 Ch07_JdbcTemplates_Testing 项目和测试 BookDAO_JdbcTemplates 的测试方法，步骤如下，

1.  下载 spring-test-5.0.0.M1.jar 文件以使用 Spring 测试 API。

1.  在 com.packt.ch07.tests 包中创建一个名为 SpringRunner_TestBookDAO_JdbcTemplate 的 JUnit 测试用例。选择 BookDAO_JdbcTemplate 作为测试类和其所有测试方法。

1.  使用以下代码中的 @RunWith 和 @ContextConfiguration 注解注释类。

1.  在代码中添加一个类型为 BookDAO 的数据成员，并应用自动装配注解，如下所示：

```java
      @RunWith(SpringJUnit4ClassRunner.class) 
      @ContextConfiguration("classpath:connection_new.xml") 
      publicclassSpringRunner_TestBookDAO_JdbcTemplate { 
        @Autowired 
        @Qualifier("bookDAO_jdbcTemplate") 
        BookDAObookDAO_JdbcTemplate; 

        @Test 
        publicvoidtestAddBook() { 
          Book book = newBook("Book_Test", 909090L, "Test  
          Publication", 1000, "Test Book description", "Test  
          author"); 
          introws_insert = bookDAO_JdbcTemplate.addBook(book); 
          assertEquals(1, rows_insert); 
        } 
      } 

```

1.  `@RunWith`注解接受`SpringJUnit4ClassRunner`。`@ContextConfiguration`接受文件以初始化容器。此外，我们使用基于注解的自动装配来测试 BookDAO 实例，而不是像早期演示中那样在`setUp()`方法中使用 Spring API。`testAddBook()`中的测试代码保持不变，因为我们没有更改逻辑。

1.  将其作为 JUnit 测试执行，如果您的 ISBN 尚未在书籍表中可用，则测试将通过。

上述代码我们对实际数据库进行了测试，这使得它变得更慢，并且始终如此。这些测试与环境不是孤立的，并且它们总是依赖于外部依赖，在我们的案例中是数据库。单元测试案例总是根据实时值基于几个假设来编写的，以便理解处理实时值时的问题和复杂性。

我们有一个更新书籍详情的函数。要更新书籍，该函数有两个参数，第一个是接受 ISBN，第二个是使用指定的 ISBN 更新书籍的价格，如下所示：

```java
publicintupdateBook(long ISBN, intupdated_price() 
{ 
   // code which fires the query to database and update the price     
   of the book whose ISBN has specified 
   // return 1 if book updated otherwise 0 
} 

```

我们编写了以下测试用例，以确定书籍是否已更新：

```java
@Test 
public void testUpdatedBook() 
{ 
  long ISBN=2;   // isbn exists in the table 
  intupdated_price=200; 
  introws_updated=bookDAO.updateBook( ISBN, updated_price); 
  assertEquals(1, rows_updated); 
} 

```

我们假设 ISBN 存在于数据库中以更新书籍详情。所以，测试用例执行成功。但是，如果在其中有人更改了 ISBN，或者有人删除了具有该 ISBN 的行怎么办？我们编写的测试用例将失败。问题不在我们的测试用例中，唯一的问题是我们假设 ISBN 存在。

另外，有时实时环境可能无法访问。控制器层测试高度依赖于请求和响应对象。这些请求和响应将在应用程序部署到服务器后由容器初始化。要么服务器不适合部署，要么控制器编码所依赖的层尚未开发。所有这些问题使得测试越来越困难。这些问题使用模拟对象测试可以轻松解决。

## 模拟测试

***

模拟测试涉及使用假对象进行测试，这些对象不是真实的。这些假对象返回进行测试所需的数据。在实际对象操作中可以节省大量工作。这些假对象通常被称为“模拟对象”。模拟对象用于替换实际对象，以避免不必要的复杂性和依赖，如数据库连接。这些模拟对象与环境隔离，导致执行速度更快。通过设置数据然后指定方法的的行为来创建模拟对象。行为包括在特定场景下返回的数据。Mockito 是使用模拟对象的一个著名的测试框架。

### Mockito

Mockito 是一个开源的 Java 基础应用程序测试框架，发布在 MIT 许可证下。它允许开发人员为**测试驱动开发**（**TDD**）创建模拟对象，使其与框架隔离。它使用 Java 反射 API 来创建模拟对象，并具有编写测试用例的简单 API。它还允许开发人员检查方法被调用的顺序。

Mockito 有一个静态的`mock()`方法，可以用来创建模拟对象。它还通过使用@Mock 注解来创建模拟对象。`methodMockitoAnnotations.initMocks(this)`指示初始化所有由@Mock 注解的注解字段。如果我们忘记这样做，对象将是 null。`@RunWith(MokitoJUnitRunner.class)`也做同样的事情。MockitoJUnitRunner 是 JUnit 使用的自定义运行器。

Mockito 的工作原理是在调用函数时返回预定义的值，**Mokito**，when()方法提供了关于将调用哪个方法的信息，Mokito，thenXXX()用于指定函数将返回的值。以下是用以来指定要返回的值的方法，

+   `thenReturn` - 用于返回一个指定的值

+   `thenThrow`- 抛出指定的异常

+   `then`和`thenAnswer`通过用户定义的代码返回一个答案

+   `thenCallRealMethod`- 调用真实的方法

模拟测试是一个简单的三个步骤的过程，如下所示，

1.  通过模拟对象初始化被测试类的依赖项

1.  执行测试操作

1.  编写测试条件以检查操作是否给出了预期的结果

让我们逐步使用 Mockito 创建`BookDAO`的模拟对象并在测试步骤中使用它，

1.  下载 mokito-all-1.9.5.jar 并将其添加到我们用作基础项目的 Ch07_JdbeTemplate_Testing 项目中。

1.  在 com.packt.ch07.unit_tests 包中创建`Spring_Mokito_TestBookDAO_JdbcTemplate`作为一个 Junit 测试用例。

1.  添加一个类型为`BookDAO`的数据成员并使用@Mock 注解标注它。

1.  在`setup()`方法中调用 Mockito 的`initMocks()`方法来初始化模拟对象，如下所示：

```java
      publicclassSpring_Mokito_TestBookDAO_JdbcTemplate { 
        @Mock 
        BookDAObookDAO_JdbcTemplate; 

        @Before 
        publicvoidsetUp()throws Exception 
        { 
          MockitoAnnotations.initMocks(this); 
        } 
      } 

```

1.  现在让我们添加代码来测试`addBook()`函数，我们首先定义期望测试函数返回的值。然后我们使用`assertXXX()`方法来测试以下行为：

```java
      @Test 
      publicvoidtestAddBook() { 
        Book book = newBook("Book_Test", 909090L, "Test  
        Publication", 1000, "Test Book description",  
        "Test author"); 
        //set the behavior for values to return in our case addBook() 
        //method 
        Mockito.when(bookDAO_JdbcTemplate.addBook(book)).thenReturn(1); 

        // invoke the function under test 
        introws_insert = bookDAO_JdbcTemplate.addBook(book); 

        // assert the actual value return by the method under test to        
        //the expected behaiour by mock object 
        assertEquals(1, rows_insert); 
      } 

```

1.  执行测试用例并测试行为。我们将得到所有测试用例成功执行。

1.  接下来让我们也添加`findAllBooks(String)`和`deleteBook()`方法的其他代码：

```java
      @Test 
      publicvoidtestDeleteBook() { 

        //with ISBN which does exit in the table Book 
        longISBN = 909090L; 
        Mockito.when(bookDAO_JdbcTemplate.deleteBook(ISBN)). 
          thenReturn(true); 
        booleandeleted = bookDAO_JdbcTemplate.deleteBook(ISBN); 
        assertTrue(deleted); 
      } 

      @Test 
      publicvoidtestFindAllBooks_Author() { 
        List<Book>books=newArrayList(); 
        books.add(new Book("Book_Test", 909090L, "Test  
          Publication", 1000, "Test Book description", "Test  
          author") ); 

        Mockito.when(bookDAO_JdbcTemplate.findAllBooks("Test  
          author")).thenReturn(books); 
        assertTrue(books.size()>0); 
        assertEquals(1, books.size()); 
        assertEquals("Book_Test",books.get(0).getBookName()); 
      } 

```

在之前的示例中，我们讨论了在实时环境以及在使用模拟对象时 DAO 层的单元测试。现在让我们在接下来的部分使用 Spring MVC 测试框架来测试控制器。

#### 使用 Spring TestContext 框架进行 Spring MVC 控制器测试

Mockito 为开发人员提供了创建 DAO 层模拟对象的功能。在前面的讨论中，我们没有 DAO 对象，但即使没有它，测试也是可能的。没有模拟对象，Spring MVC 层测试是不可能的，因为它们高度依赖于初始化由容器完成的请求和响应对象。spring-test 模块支持创建 Servlet API 的模拟对象，使在不实际部署容器的情况下测试 Web 组件成为可能。以下表格显示了由 Spring TestContext 框架提供的用于创建模拟对象包列表：

| **包名** | **提供模拟实现** |
| --- | --- |
| org.springframework.mock.env | 环境和属性源 |
| org.springframework.mock.jndi | JNDI SPI |
| org.springframework.mock.web | Servlet API |
| org.springframework.mock.portlet | Portlet API |

org.springframework.mock.web 提供了 MockHttpServletRequest，MockHttpServletResponse，MockHttpSession 作为 HttpServletRequest，HttpServletResponse 和 HttpSession 的模拟对象，供使用。它还提供了 ModelAndViewAssert 类，以测试 Spring MVC 框架中的 ModelAndView 对象。让我们逐步测试我们的 SearchBookController 如下：

1.  将 spring-test.jar 添加到`ReadMyBooks`应用程序中，我们将在测试中使用它。

1.  创建`com.packt.ch06.controllers.test_controllers`包，以添加控制器的测试用例。

1.  在先前步骤创建的包中创建`TestSearchBookController`作为 JUnit 测试用例。

1.  使用`@WebAppConfiguration`进行注解。

1.  声明类型为 SearchBookController 的数据成员并如代码所示自动注入：

```java
      @WebAppConfiguration 
      @ContextConfiguration({ "file:WebContent/WEB-INF/book- 
        servlet.xml" }) 
      @RunWith(value = SpringJUnit4ClassRunner.class) 
      publicclassTestSearchBookController { 
         @Autowired 
        SearchBookControllersearchBookController; 
      } 

```

1.  让我们测试 add testSearchBookByAuthor()以测试 searchBookByAuthor()方法。该方法接受用户在 Web 表单中输入的作者名称，并返回该作者所写的书籍列表。代码将如下所示：

    1.  初始化测试方法所需的数据

    1.  调用测试方法

    1.  断言值。

1.  最终代码将如下所示：

```java
      @Test 
      publicvoidtestSearchBookByAuthor() { 

        String author_name="T.M.Jog"; 
        ModelAndViewmodelAndView =   
          searchBookController.searchBookByAuthor(author_name); 
        assertEquals("display",modelAndView.getViewName()); 
      } 

```

1.  我们正在测试名为'display'的视图名称，该视图是从控制器方法中编写出来的。

1.  Spring 提供了 ModelAndViewAssert，提供了一个测试控制器方法返回的 ModelAndView 的方法，如下面的代码所示：

```java
      @Test 
      publicvoidtestSerachBookByAuthor_New() 
      { 
        String author_name="T.M.Jog"; 
        List<Book>books = newArrayList<Book>(); 
        books.add(new Book("Learning Modular Java Programming",  
          9781235, "packt pub publication", 800, 
          "explore the power of modular Programming ", author_name)); 
        books.add(new Book("Learning Modular Java Programming",  
          9781235, "packt pub publication", 800, 
          "explore the power of modular Programming ", author_name)); 
        ModelAndViewmodelAndView = 
          searchBookController.searchBookByAuthor(author_name); 
        ModelAndViewAssert.assertModelAttributeAvailable( 
          modelAndView, "book_list"); 
      } 

```

1.  执行测试用例，绿色表示测试用例已通过。

1.  我们成功测试了 SearchBookController，其具有无需任何表单提交、表单模型属性绑定、表单验证等简单编码。我们刚刚处理的这些复杂的代码测试变得更加复杂。

#### Spring MockMvc

Spring 提供了 MockMVC，作为主要的入口点，并配备了启动服务器端测试的方法。将使用 MockMVCBuilder 接口的实现来创建一个 MockMVC 对象。MockMVCBuilders 提供了以下静态方法，可以获取 MockMVCBuilder 的实现：

+   xmlConfigSetUp(String ...configLocation) - 当使用 XML 配置文件来配置应用程序上下文时使用，如下所示：

```java
      MockMvcmockMVC=   
      MockMvcBuilders.xmlConfigSetUp("classpath:myConfig.xml").build(); 

```

+   annotationConfigSetUp(Class ... configClasses) - 当使用 Java 类来配置应用程序上下文时使用。以下代码显示了如何使用 MyConfig.java 作为一个配置类：

```java
      MockMvcmockMVC=  
         MockMvcBuilders.annotationConfigSetUp(MyConfiog.class). 
                                                         build(); 

```

+   standaloneSetUp(Object ... controllers) - 当开发者配置了测试控制器及其所需的 MVC 组件时使用。以下代码显示了使用 MyController 进行配置：

```java
      MockMvcmockMVC= MockMvcBuilders.standaloneSetUp( 
        newMyController()).build(); 

```

+   webApplicationContextSetUp(WebApplicationContext context) - 当开发者已经完全初始化 WebApplicationContext 实例时使用。以下代码显示了如何使用该方法：

```java
      @Autowired 
      WebApplicationContextwebAppContext; 
      MockMvcmockMVC= MockMVCBuilders.webApplicationContextSetup( 
        webAppContext).build(); 

```

MockMvc has `perform()` method which accepts the instance of RequestBuilder and returns the ResultActions. The `MockHttpServletRequestBuilder` is an implementation of RequestBuilder who has methods to build the request by setting request parameters, session. The following table shows the methods which facilitate building the request,

| **Method name** | **The data method description** |
| --- | --- |
| accept | 用于将“Accept”头设置为给定的媒体类型 |
| buildRequest | 用于构建 MockHttpServletRequest |
| createServletRequest | 根据 ServletContext，该方法创建一个新的 MockHttpServletRequest |
| Param | 用于将请求参数设置到 MockHttpServletRequest。 |
| principal | 用于设置请求的主体。 |
| locale . | 用于设置请求的区域设置。 |
| requestAttr | 用于设置请求属性。 |
| Session, sessionAttr, sessionAttrs | 用于设置会话或会话属性到请求 |
| characterEncoding | 用于将字符编码设置为请求 |
| content and contentType | 用于设置请求的正文和内容类型头。 |
| header and headers | 用于向请求添加一个或所有头信息。 |
| contextPath | 用于指定表示请求 URI 的上下文路径部分 |
| Cookie | 用于向请求添加 Cookie。 |
| flashAttr | 用于设置输入的闪存属性。 |
| pathInfo | 用于指定表示请求 URI 的 pathInfo 部分。 |
| Secure | 用于设置 ServletRequest 的安全属性，如 HTTPS。 |
| servletPath | 用于指定表示 Servlet 映射路径的请求 URI 部分。 |

The `perfom()` method of MockMvc returns the ResultActions, which facilitates the assertions of the expected result by following methods:

| **Method name** | **Description** |
| --- | --- |
| andDo | 它接受一个通用操作。 |
| andExpect | 它接受预期的操作 |
| annReturn | 它返回预期请求的结果，可以直接访问。 |

Let's use MockMvc to test AddBookController step by step:

1.  Add TestAddBookController as JUnit test case in `com.packt.ch06.controllers.test_controllers package`.

1.  像早期代码中一样，用`@WebAppConfiguration`、`@ContextConfiguration`和`@RunWith`注解类。

1.  添加类型为 WebApplicationContext 和`AddBookController`的数据成员，并用`@Autowired`注解两者。

1.  添加类型为 MockMvc 的数据成员，并在 setup()方法中初始化它，如以下所示释放内存：

```java
      @WebAppConfiguration 
      @ContextConfiguration( 
        { "file:WebContent/WEB-INF/books-servlet.xml"}) 
      @RunWith(value = SpringJUnit4ClassRunner.class) 
      publicclassTestAddBookController { 
        @Autowired 
        WebApplicationContextwac; 

        MockMvcmockMVC; 

        @Autowired 
        AddBookControlleraddBookController; 

        @Before 
        publicvoidsetUp() throws Exception { 
          mockMVC= 
            MockMvcBuilders.standaloneSetup(addBookController).build(); 
        } 
      } 

```

+   让我们在 testAddBook()中添加测试 addBook()方法的代码：

    1.  通过设置以下值初始化请求：

+   模型属性'book'使用默认值

+   将表单提交结果设置为内容类型

+   方法将被调用的 URI

+   表单的请求参数：

    1.  通过检查测试结果：

+   视图名称

+   模型属性名称

    1.  使用 andDo()在控制台上打印测试动作的结果

测试 AddBook()方法的代码如下：

```java
      @Test 
      publicvoidtestAddBook() { 
        try { 
          mockMVC.perform(MockMvcRequestBuilders.post("/addBook.htm") 
          .contentType(MediaType.APPLICATION_FORM_URLENCODED) 
          .param("bookName", "Book_test") 
          .param("author", "author_test") 
          .param("description", "adding book for test") 
          .param("ISBN", "1234") 
          .param("price", "9191") 
          .param("publication", "This is the test publication") 
          .requestAttr("book", new Book())) 
          .andExpect(MockMvcResultMatchers.view().name("display")) 
          .andExpect(MockMvcResultMatchers.model(). 
            attribute("auth_name","author_test")) 
          .andDo(MockMvcResultHandlers.print()); 
        } catch (Exception e) { 
          // TODO: handle exception 
          fail(e.getMessage()); 
        } 
      } 

```

在 andExpect( )中的预期行为匹配由 ResultMatcher 提供。MockMvcResultMatcher 是 ResultMatcher 的一个实现，提供了匹配视图、cookie、header、模型、请求和其他许多参数的方法。andDo()方法将 MvcResult 打印到 OutputStream。

1.  运行测试用例，令人惊讶的是它会失败。输出的一部分如下所示：

![](img/image_07_004.png)

1.  它显示了验证错误，但我们已经根据验证规则给出了所有输入。哪个验证失败了从输出中看不清楚。不，没必要惊慌，也不需要逐个检查验证。

1.  与其制造更多混乱，不如添加使用 attributeHasErrors()的验证测试代码，如下划线语句所示：

```java
      @Test 
publicvoidtestAddBook_Form_validation() { 
        try { 
          mockMVC.perform(MockMvcRequestBuilders.post("/addBook.htm")                        .contentType(MediaType.APPLICATION_FORM_URLENCODED) 
          .param("bookName", "Book_test") 
          .param("author", "author_test") 
          .param("description", "adding book for test") 
          .param("ISBN", "12345") 
          .param("price", "9191") 
          .param("publication", "This is the test publication") 
          .requestAttr("book", new Book())) 
          .andExpect(MockMvcResultMatchers.view().name("bookForm")) 
          .andExpect(MockMvcResultMatchers .model(). 
            attributeHasErrors("book")) 
          .andDo(MockMvcResultHandlers.print()); 
        }  
        catch (Exception e) { 
          fail(e.getMessage()); 
          e.printStackTrace(); 
        } 
      }  

```

1.  测试运行成功，证明输入存在验证错误。我们可以在控制台输出的'errors'中获取到验证失败的字段：

```java
      MockHttpServletRequest: 
        HTTP Method = POST 
        Request URI = /addBook.htm 
        Parameters = {bookName=[Book_test],  
      author=[author_test], 
      description=[adding book for test],  
      ISBN=[1234],  
      price=[9191], 
      publication=[This is the test publication]} 
      Headers = { 
        Content-Type=[application/x-www-form-urlencoded]} 
      Handler: 
        Type = com.packt.ch06.controllers.AddBookController 
        Method = public  
      org.springframework.web.servlet.ModelAndView 
      com.packt.ch06.controllers.AddBookController. 
      addBook(com.packt.ch06.beans.Book,org. 
      springframework.validation.BindingResult)       
      throwsjava.lang.Exception 
      Async: 
      Async started = false 
      Async result = null 

      Resolved Exception: 
        Type = null 
      ModelAndView: 
        View name = bookForm 
        View = null 
        Attribute = priceList 
        value = [300, 350, 400, 500, 550, 600] 
        Attribute = book 
        value = Book_test  adding book for test  9191 
        errors = [Field error in object 'book' on field  
          'description':  
          rejected value [adding book for test];  
          codes 
          [description.length.book.description, 
          description.length.description,description. 
          length.java.lang.String,description.length]; 
          arguments []; 
          default message [Please enter description  
          within 40 charaters only]] 
      FlashMap: 
        Attributes = null 
      MockHttpServletResponse: 
        Status = 200 
      Error message = null 
      Headers = {} 
      Content type = null 
      Body =  
      Forwarded URL = bookForm 
      Redirected URL = null 
      Cookies = [] 

```

1.  尽管描述符中的字符在 10 到 40 个指定字符的限制内。让我们找出在 Validator2 中犯错的规则。

1.  设置发布验证规则的 validate 方法中的代码是：

```java
      if (book.getDescription().length() < 10 ||   
        book.getDescription().length() < 40)  
      { 
        errors.rejectValue("description", "description.length", 
          "Please enter description within 40 charaters only"); 
      } 

```

1.  是的，我们将发布长度设置为小于 40 的验证，导致失败。我们犯了一个错误。让我们更改代码，以设置规则，长度大于 40 将不允许。以下是更新的代码：

```java
      if (book.getDescription().length() < 10 ||
        book.getDescription().length() > 40)  
      { 
        errors.rejectValue("description", "description.length", 
        "Please enter description within 40 charaters only"); 
      } 

```

1.  现在重新运行 testAddController 以查看发生了什么。

1.  测试用例成功通过。这就是我们进行测试用例的原因。

1.  现在让我们在 testAddBook_Form_validation()中添加测试字段验证的代码：

```java
      @Test 
      publicvoidtestAddBook_Form_Field_Validation() 
      { 
        try { 
          mockMVC.perform(MockMvcRequestBuilders.post("/addBook.htm") 
          .param("bookName", "") 
          .param("author", "author_test") 
          .param("description"," no desc") 
          .param("ISBN", "123") 
          .param("price", "9191") 
          .param("publication", " ") 
          .requestAttr("book", new Book())) 
          .andExpect(MockMvcResultMatchers.view().name("bookForm"))  
          .andExpect(MockMvcResultMatchers.model() 
          .attributeHasFieldErrors("book", "description")).andExpect(
            MockMvcResultMatchers.model() 
          .attributeHasFieldErrors("book", "ISBN")).andExpect( 
            MockMvcResultMatchers.model() 
          .attributeHasFieldErrors("book", "bookName")). 
            andDo(MockMvcResultHandlers.print()); 
        }catch(Exception ex) 
        { 
          fail(ex.getMessage()); 
        } 
      } 

```

1.  运行测试用例，其中验证错误失败。

控制器和 DAO 正常工作。服务层使用 DAO，所以让我们对服务层进行集成测试。您可以按照我们讨论的和对 DAO 层测试进行模拟对象测试。我们将进入服务层集成测试的下一阶段。

## 第二阶段 集成测试

* * *

### 服务和 DAO 层的集成测试

让我们逐步进行应用程序的集成测试，Ch05_Declarative_Transaction_Management 如下：

1.  创建 com.packt.ch05.service.integration_tests 包。

1.  创建 JUnit 测试用例 TestBookService_Integration，将 BookServiceImpl 作为测试类。选择其所有方法进行测试。

1.  声明类型为 BookService 的数据成员，并用@Autowired 注解注释它，如下所示：

```java
      @RunWith(SpringJUnit4ClassRunner.class) 
      @ContextConfiguration("classpath:connection_new.xml") 
      publicclassTestBookService_Integration 
      { 
        @Autowired 
        BookServicebookService; 
      }   

```

1.  让我们测试 addBook()方法，就像我们之前在 JUnit 测试中做的那样。你可以参考下面的代码：

```java
      @Test 
      publicvoidtestAddBook() { 
        // Choose ISBN which is not there in book table 
        Book book = newBook("Book_Test", 909098L, "Test  
        Publication", 1000, "Test Book description", "Test  
        author"); 
        booleanflag=bookService.addBook(book); 
        assertEquals(true, flag); 
      } 

```

1.  你可以运行测试用例，它将成功运行。

### 注意

BookService 中的所有其他测试方法可以从源代码中参考。

我们开发的两个层次都在按我们的预期工作。我们分别开发了控制器、服务和 DAO，并进行了测试。现在，我们将它们组合到单个应用程序中，这样我们就会有一个完整的应用程序，然后通过集成测试，我们将检查它是否如预期般工作。

### 控制器和 Service 层的集成测试

让我们将以下三个层次从 Ch05_Declarative_Transaction_Management 中组合到 ReadMyBooks 中：

1.  在 ReadMyBooks 的 lib 文件夹中添加 jdbc 和 spring-jdbc 以及其他所需的 jar 文件。

1.  从 Ch05_Declarative_Transaction_Management 中将 com.packt.ch03.dao 和 com.packt.ch05.service 包复制到 ReadMyBooks 应用程序。

1.  在 ReadMyBooks 应用程序的类路径中复制 connection_new.xml。

1.  在 Book 类的表单提交中，我们注释了默认构造函数，服务中的 addBook 逻辑是检查 98564567las 的默认值。

1.  如下所示，通过下划线修改 BookService，其余代码保持不变：

```java
      @Override 
      @Transactional(readOnly=false) 
      publicbooleanaddBook(Book book) { 
        // TODO Auto-generated method stub 

        if (searchBook(book.getISBN()).getISBN() == 0) { 
          // 
        } 
      } 

```

1.  控制器需要更新以与底层层进行通信，如下所示：

    +   在控制器中添加类型为 BookService 的自动装配数据成员。

    +   根据业务逻辑要求，在控制器的 method 中调用服务层的 method。

1.  下面将更新 addBook()方法：

```java
      @RequestMapping("/addBook.htm") 
      publicModelAndViewaddBook(@Valid@ModelAttribute("book") 
      Book book, BindingResultbindingResult) 
      throws Exception { 
        // same code as developed in the controller 
        // later on the list will be fetched from the table 
        List<Book>books = newArrayList(); 

        if (bookService.addBook(book)) { 
          books.add(book); 
        } 
        modelAndView.addObject("book_list", books);   
        modelAndView.addObject("auth_name", book.getAuthor());  
        returnmodelAndView; 
      } 

```

### 注意

同样，我们可以更新所有控制器中的方法。你可以参考完整的源代码。

让我们执行测试用例 TestAddBookController.java 以获取结果。

代码将执行并给出成功消息。同时在表中添加了一行，包含了我们指定的 ISBN 和其他值。

我们已经成功测试了所有组件。现在我们可以直接开始系统测试。

但是要有耐心，因为我们将在测试框架“Arquillian”中讨论新的条目。

## 第三阶段系统测试

* * *

现在所有层次都按照预期工作，是时候通过网络测试应用程序了，即逐一检查功能，同时非常注意逐步进行，不仅要关注结果，还要观察演示，这将接近实际的部署环境。让我们部署应用程序，以检查所有功能是否正常工作，并在数据库和演示方面给出正确的结果，通过以下任一方式进行：

### 使用 Eclipse IDE 进行部署

在 Eclipse 中，一旦开发完成，配置服务器并从项目浏览器中选择项目以选择**`Run on server`**选项，如下面的箭头所示：

![](img/image_07_005.png)

IDE 会将应用程序包装在战争文件中，并将其部署到容器中。现在，你可以逐一检查功能，以确保一切按预期进行。我们还将关注演示文稿、外观和准确性的数据，这些数据由演示文稿显示。

### 手动部署应用程序

手动部署应用程序可以通过以下步骤进行：

1.  首先，我们需要获取它的 jar 文件。我们可以使用 Eclipse IDE 通过右键点击应用程序并选择**`Export`**来轻松获取战争文件，如下面的箭头所示：

![](img/image_07_006.png)

1.  选择你想要创建战争文件的目标位置。如果你想的话，可以更改战争文件的名字。我将保持 ReadMyBooks 不变。

1.  点击**`finish`**完成过程。你将在选定的目标位置得到一个战争文件。

1.  复制我们在上一步创建的 WAR 文件，并将其粘贴到 Tomcat 目录下的'webapps'文件夹中。

1.  通过点击**`bin`**文件夹中的`startup.bat`文件来启动 tomcat。

1.  一旦 tomcat 启动，打开浏览器并输入主页 URL，格式为[`host_name:port_number_of_tomcat/war_file_name`](http://host_name:port_number_of_tomcat/war_file_name)。在我们的案例中，它是[`locathost:8080/ReadMyBooks`](http://locathost:8080/ReadMyBooks)。

1.  在继续之前，请确保数据库参数已正确设置，否则应用程序将失败。

1.  主页将打开，我们可以在这里测试应用程序的功能和外观。

## 摘要

* * *

在本章中，我们讨论了什么是测试以及为什么它如此重要。我们还讨论了单元测试、集成测试和用户接受测试作为测试的阶段。市场上有很多测试工具，我们对此进行了概述，以便您明智地选择工具。测试中一个非常重要的工具是'Junit 测试'，我们使用它来执行 DAO 层的单元测试，这是测试阶段 1 的开始。但是 JUnit 使用实时数据库，我们讨论了在外部参数上测试的困难。我们通过使用模拟对象解决了这个问题。Mokito 是创建模拟对象的工具之一，我们探索它来测试 DAO 层。在 DAO 层之后，我们测试了 Web 层，这也依赖于 Web 容器来初始化请求和响应对象。我们深入讨论了 Spring TestContext 框架，其 MockMVC 模块便于创建 Web 相关组件（如请求和响应）的模拟对象。我们还使用该框架进行表单验证测试。在单元测试之后，我们执行了 DAO 和 Service 层的集成测试，然后是 Web 和 Service 层的集成测试。故事不会在这里结束，我们通过进行系统测试来成功部署并最终检查产品。我们所开发的的所有组件都在正常工作，我们通过成功执行系统测试证明了这一点！！

在下一章中，我们将进一步讨论安全性在应用程序中的角色以及 Spring 框架提供的实现安全性的方法。请继续阅读！！！
