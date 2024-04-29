# 使用高级 JUnit 功能简化测试

简单是终极的复杂。

- 列奥纳多·达·芬奇

到目前为止，我们已经了解了 Jupiter 的基础知识，这是 JUnit 5 框架提供的全新编程模型。此外，Jupiter 提供了丰富的可能性，可以创建不同类型的测试用例。在本章中，我们将回顾这些高级功能。为此，本章结构如下：

+   **依赖注入**：本节首先介绍了测试类中构造函数和方法的依赖注入。然后，它回顾了 Jupiter 中提供的三个参数解析器。这些解析器允许在测试中注入`TestInfo`、`RepetitionInfo`和`TestReporter`对象。

+   **动态测试**：本节讨论了如何在 JUnit 5 中实现动态测试，使用`dynamicTest`和`stream`方法。

+   **测试接口**：本节介绍了可以在测试接口和默认方法上声明的 Jupiter 注解。

+   **测试模板**：JUnit 5 引入了测试用例的模板概念。这些模板将根据调用上下文多次调用。

+   **参数化测试**：与 JUnit 4 一样，JUnit 5 提供了创建由不同输入数据驱动的测试的功能，即参数化测试。我们将发现，对这种测试的支持在 Jupiter 编程模型中得到了显着增强。

+   **Java 9**：2017 年 9 月 21 日，Java 9 发布。正如我们将发现的那样，JUnit 5 已经实现为与 Java 9 兼容，特别强调了 Java 9 的模块化特性。

# 依赖注入

在以前的 JUnit 版本中，不允许测试构造函数和方法带有参数。JUnit 5 的一个主要变化是现在允许测试构造函数和方法都包含参数。这个特性使得构造函数和方法可以进行依赖注入。

正如本书的第二章中介绍的，JUnit 5 的扩展模型具有一个扩展，为 Jupiter 测试提供依赖注入，称为`ParameterResolver`，它定义了希望在运行时动态解析参数的测试扩展的 API。

如果测试构造函数或方法带有`@Test`、`@TestFactory`、`@BeforeEach`、`@AfterEach`、`@BeforeAll`或`@AfterAll`注解，并接受一个参数，那么这个参数将由解析器（具有父类`ParameterResolver`的对象）在运行时解析。在 JUnit 5 中，有三个内置的解析器自动注册：`TestInfoParameterResolver`、`RepetitionInfoParameterResolver`和`TestReporterParameterResolver`。我们将在本节中回顾这些解析器中的每一个。

# TestInfoParameterResolver

给定一个测试类，如果方法参数的类型是`TestInfo`，则 JUnit 5 解析器`TestInfoParameterResolver`会提供一个与声明的参数对应的`TestInfo`实例，该实例对应于当前测试。`TestInfo`对象用于检索有关当前测试的信息，例如测试显示名称、测试类、测试方法或关联的标签。

`TestInfo`充当 JUnit 4 的`TestName`规则的替代品。

`TestInfo`类位于`org.junit.jupiter.api`包中，并提供以下 API：

+   `String getDisplayName()`：这会返回测试或容器的显示名称。

+   `Set<String> getTags()`：这会获取当前测试或容器的所有标签集。

+   `Optional<Class<?>> getTestClass()`：如果可用，这会获取与当前测试或容器关联的类。

+   `Optional<Method> getTestMethod()`：如果可用，这会获取与当前测试关联的方法。

![](img/00067.jpeg)*TestInfo* API

让我们看一个例子。请注意，在以下类中，使用`@BeforeEach`和`@Test`注解的两个方法都接受`TestInfo`参数。这个参数是由`TestInfoParameterResolver`注入的：

```java
package io.github.bonigarcia;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInfo;

class TestInfoTest {

    @BeforeEach
    void init(TestInfo testInfo) {
        String displayName = testInfo.getDisplayName();
        System.*out*.printf("@BeforeEach %s %n", displayName);
    }

    @Test
    @DisplayName("My test")
    @Tag("my-tag")
    void testOne(TestInfo testInfo) {
        System.*out*.println(testInfo.getDisplayName());
        System.*out*.println(testInfo.getTags());
        System.*out*.println(testInfo.getTestClass());
        System.*out*.println(testInfo.getTestMethod());
    }

    @Test
    void testTwo() {
    }

}
```

因此，在每个方法的主体中，我们能够在运行时使用`TestInfo` API 来获取测试信息，如下面的屏幕截图所示：

![](img/00068.gif)

*TestInfo*对象的依赖注入的控制台输出

# RepetitionInfoParameterResolver

JUnit 5 中提供的第二个内置解析器称为`RepetitionInfoParameterResolver`。给定一个测试类，如果`@RepeatedTest`、`@BeforeEach`或`@AfterEach`方法的方法参数是`RepetitionInfo`类型，`RepetitionInfoParameterResolver`将提供`RepetitionInfo`的实例。

`RepetitionInfo`可用于检索有关当前重复和相应`@RepeatedTest`的总重复次数的信息。`RepetitionInfo`的 API 提供了两种方法，如列表后的屏幕截图所示：

+   `int getCurrentRepetition()`: 获取相应`@RepeatedTest`方法的当前重复次数

+   `int getTotalRepetitions()`: 获取相应`@RepeatedTest`方法的总重复次数

![](img/00069.jpeg)*RepetitionInfo* API

这里的类包含了对`RepetitionInfo`的简单示例：

```java
package io.github.bonigarcia;

import org.junit.jupiter.api.RepeatedTest;
import org.junit.jupiter.api.RepetitionInfo;

class RepetitionInfoTest {

    @RepeatedTest(2)
    void test(RepetitionInfo repetitionInfo) {
        System.*out*.println("** Test " + 
            repetitionInfo.getCurrentRepetition()
            + "/" + repetitionInfo.getTotalRepetitions());
    }

}
```

正如在测试输出中所看到的，我们能够在运行时读取有关重复测试的信息：

![](img/00070.gif)

*RepetitionInfo*对象的依赖注入的控制台输出。

# TestReporterParameterResolver

JUnit 5 中的最后一个内置解析器是`TestReporterParameterResolver`。同样，给定一个测试类，如果方法参数的类型是`TestReporter`，`TestReporterParameterResolver`将提供`TestReporter`的实例。

`TestReporter`用于发布有关测试执行的附加数据。数据可以通过`reportingEntryPublished`方法进行消耗，然后可以被 IDE 请求或包含在测试报告中。每个`TestReporter`对象都将信息存储为一个映射，即键值对集合：

![](img/00071.jpeg)*TestReporter* API

这个测试提供了`TestReporter`的一个简单示例。正如我们所看到的，我们使用注入的`testReporter`对象使用键值对添加自定义信息：

```java
package io.github.bonigarcia;

import java.util.HashMap;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestReporter;

class TestReporterTest {

    @Test
    void reportSingleValue(TestReporter testReporter) {
        testReporter.publishEntry("key", "value");
    }

    @Test
    void reportSeveralValues(TestReporter testReporter) {
        HashMap<String, String> values = new HashMap<>();
        values.put("name", "john");
        values.put("surname", "doe");
        testReporter.publishEntry(values);
    }

}
```

# 动态测试

我们知道，在 JUnit 3 中，我们通过解析方法名称并检查它们是否以单词 test 开头来识别测试。然后，在 JUnit 4 中，我们通过收集带有`@Test`注解的方法来识别测试。这两种技术共享相同的方法：测试在编译时定义。这个概念就是我们所说的静态测试。

静态测试被认为是一种有限的方法，特别是对于同一个测试应该针对各种输入数据执行的常见情况。在 JUnit 4 中，这个限制以几种方式得到解决。解决这个问题的一个非常简单的解决方案是循环输入测试数据并练习相同的测试逻辑（这里是 JUnit 4 的示例）。按照这种方法，一个测试会一直执行，直到第一个断言失败：

```java
package io.github.bonigarcia;

import org.junit.Test;

public class MyTest {

    @Test
    public void test() {
        String[] input = { "A", "B", "C" };
        for (String s : input) {
            exercise(s);
        }
    }

    private void exercise(String s) {
        System.*out.*println*(s);
*    }

}
```

更详细的解决方案是使用 JUnit 4 支持参数化测试，使用参数化运行器。这种方法也不会在运行时创建测试，它只是根据参数多次重复相同的测试：

```java
package io.github.bonigarcia;

import java.util.Arrays;
import java.util.Collection;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.Parameterized;
import org.junit.runners.Parameterized.Parameter;
import org.junit.runners.Parameterized.Parameters;

@RunWith(Parameterized.class)
public class ParameterizedTest {

    @Parameter(0)
    public Integer input1;

    @Parameter(1)
    public String input2;

    @Parameters(name = "My test #{index} -- input data: {0} and {1}")
    public static Collection<Object[]> data() {
        return Arrays
           .*asList*(new Object[][] { { 1, "hello" }, { 2, "goodbye" } });
    }

    @Test
    public void test() {
        System.*out*.println(input1 + " " + input2);
    }
}
```

我们可以在 Eclipse IDE 中看到前面示例的执行：

![](img/00072.jpeg)

在 Eclipse 中执行 JUnit 4 的参数化测试

另一方面，JUnit 5 允许通过一个使用`@TestFactory`注释的工厂方法在运行时生成测试。与`@Test`相比，`@TestFactory`方法不是一个测试，而是一个工厂。`@TestFactory`方法必须返回`DynamicTest`实例的`Stream`、`Collection`、`Iterable`或`Iterator`。这些`DynamicTest`实例是惰性执行的，可以动态生成测试用例。

为了创建动态测试，我们可以使用位于`org.junit.jupiter.api`包中的`DynamicTest`类的静态方法`dynamicTest`。如果我们检查这个类的源代码，我们可以看到`DynamicTest`由一个字符串形式的显示名称和一个可执行对象组成，可以提供为 lambda 表达式或方法引用。

让我们看一些动态测试的例子。在下面的例子中，第一个动态测试将失败，因为我们没有返回预期的`DynamicTests`集合。接下来的三个方法是非常简单的例子，演示了`DynamicTest`实例的`Collection`、`Iterable`和`Iterator`的生成：

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.DynamicTest.dynamicTest;

import java.util.Arrays;
import java.util.Collection;
import java.util.Iterator;
import java.util.List;
import org.junit.jupiter.api.DynamicTest;
import org.junit.jupiter.api.TestFactory;

class CollectionTest {

    // Warning: this test will raise an exception
    @TestFactory
    List<String> dynamicTestsWithInvalidReturnType() {
        return Arrays.*asList*("Hello");
    }

    @TestFactory
    Collection<DynamicTest> dynamicTestsFromCollection() {
        return Arrays.*asList*(
                *dynamicTest*("1st dynamic test", () -> 
 *assertTrue*(true)),
                *dynamicTest*("2nd dynamic test", () -> *assertEquals*(4, 2 
                     * 2)));
    }

    @TestFactory
    Iterable<DynamicTest> dynamicTestsFromIterable() {
        return Arrays.*asList*(
                *dynamicTest*("3rd dynamic test", () -> 
 *assertTrue*(true)),
                *dynamicTest*("4th dynamic test", () -> *assertEquals*(4, 2 
                    * 2)));
    }

    @TestFactory
    Iterator<DynamicTest> dynamicTestsFromIterator() {
        return Arrays.*asList*(
                *dynamicTest*("5th dynamic test", () -> 
 *assertTrue*(true)),
                *dynamicTest*("6th dynamic test", () -> *assertEquals*(4, 2 
                     * 2))).iterator();
    }

}
```

这些示例并没有真正展示动态行为，而只是演示了支持的返回类型。请注意，第一个测试将由于`JUnitException`而失败：

![](img/00073.gif)

第一个动态测试示例的控制台输出

以下示例演示了为给定的输入数据集生成动态测试有多么容易：

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.DynamicTest.dynamicTest;

import java.util.stream.Stream;
import org.junit.jupiter.api.DynamicTest;
import org.junit.jupiter.api.TestFactory;

class DynamicExampleTest {

    @TestFactory
    Stream<DynamicTest> dynamicTestsFromStream() {
        Stream<String> inputStream = Stream.*of*("A", "B", "C");
        return inputStream.*map*(
                input -> *dynamicTest*("Display name for input " + input, 
                () -> {
                     System.*out*.println("Testing " + input);
                }));
    }

}
```

请注意，最终执行了三个测试，并且这三个测试是由 JUnit 5 在运行时创建的：

![](img/00074.gif)

第二个动态测试示例的控制台输出

在 JUnit 5 中，还有另一种创建动态测试的可能性，使用`DynamicTest`类的`stream`静态方法。这个方法需要一个输入生成器，一个根据输入值生成显示名称的函数，以及一个测试执行器。

让我们看另一个例子。我们创建一个测试工厂，提供输入数据作为`Iterator`，使用 lambda 表达式作为显示名称函数，最后，使用另一个 lambda 表达式实现的测试执行器。在这个例子中，测试执行器基本上断言输入的整数是偶数还是奇数：

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.DynamicTest.stream;

import java.util.Arrays;
import java.util.Iterator;
import java.util.function.Function;
import java.util.stream.Stream;
import org.junit.jupiter.api.DynamicTest;
import org.junit.jupiter.api.TestFactory;
import org.junit.jupiter.api.function.ThrowingConsumer;

class StreamExampleTest {

    @TestFactory
    Stream<DynamicTest> streamTest() {
        // Input data
        Integer array[] = { 1, 2, 3 };
        Iterator<Integer> inputGenerator = Arrays.*asList*(array).iterator();

        // Display names
        Function<Integer, String> displayNameGenerator = (
                input) -> "Data input:" + input;

        // Test executor
        ThrowingConsumer<Integer> testExecutor = (input) -> {
            System.*out*.println(input);
            *assertTrue*(input % 2 == 0);
        };

        // Returns a stream of dynamic tests
        return *stream*(inputGenerator, displayNameGenerator, 
            testExecutor);
    }

}
```

奇数输入的测试将失败。我们可以看到，三个测试中有两个会失败：

![](img/00075.gif)

动态测试执行的控制台输出（第三个示例）

# 测试接口

在 JUnit 5 中，有关 Java 接口中注解使用的规则是不同的。首先，我们需要意识到`@Test`、`@TestFactory`、`@BeforeEach`和`@AfterEach`可以在接口默认方法上声明。

默认方法是 Java 8 版本引入的一个特性。这些方法（使用保留关键字`default`声明）允许在 Java 接口中为给定方法定义默认实现。这种能力对于与现有接口的向后兼容性可能很有用。

关于 JUnit 5 和接口的第二条规则是，`@BeforeAll`和`@AfterAll`可以在测试接口中的`static`方法上声明。此外，如果实现给定接口的测试类被注解为`@TestInstance(Lifecycle.PER_CLASS)`，则接口上声明的`@BeforeAll`和`@AfterAll`方法不需要是`static`，而是`default`方法。

关于 JUnit 5 中接口的第三条和最后一条规则是，可以在测试接口上声明`@ExtendWith`和`@Tag`来配置扩展和标签。

让我们看一些简单的例子。在下面的类中，我们创建的是一个接口，而不是一个类。在这个接口中，我们使用了注解`@BeforeAll`、`@AfterAll`、`@BeforeEach`和`@AfterEach`。一方面，我们将`@BeforeAll`、`@AfterAll`定义为静态方法。另一方面，我们将`@BeforeEach`和`@AfterEach`定义为 Java 8 默认方法：

```java
package io.github.bonigarcia;

import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.TestInfo;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public interface TestLifecycleLogger {

 static final Logger ***log*** = LoggerFactory
            .getLogger(TestLifecycleLogger.class.getName());

    @BeforeAll
    static void beforeAllTests() {
        ***log***.info("beforeAllTests");
    }

    @AfterAll
    static void afterAllTests() {
        ***log***.info("afterAllTests");
    }

    @BeforeEach
    default void beforeEachTest(TestInfo testInfo) {
        ***log***.info("About to execute {}", testInfo.getDisplayName());
    }

    @AfterEach
    default void afterEachTest(TestInfo testInfo) {
        ***log***.info("Finished executing {}", testInfo.getDisplayName());
    }

}
```

在这个例子中，我们使用了 Simple Logging Facade for Java (SLF4J)库。请查看 GitHub 上的代码（[`github.com/bonigarcia/mastering-junit5`](https://github.com/bonigarcia/mastering-junit5)）以获取有关依赖声明的详细信息。

在这个例子中，我们使用注解`TestFactory`来定义 Java 接口中的默认方法：

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.DynamicTest.dynamicTest;

import java.util.Arrays;
import java.util.Collection;
import org.junit.jupiter.api.DynamicTest;
import org.junit.jupiter.api.TestFactory;

interface TestInterfaceDynamicTestsDemo {

    @TestFactory
    default Collection<DynamicTest> dynamicTestsFromCollection() {
        return Arrays.*asList*(
                *dynamicTest*("1st dynamic test in test interface",
                        () -> *assertTrue*(true)),
                *dynamicTest*("2nd dynamic test in test interface",
                        () -> *assertTrue*(true)));
    }

}
```

最后，我们在另一个接口中使用注解`@Tag`和`@ExtendWith`：

```java
package io.github.bonigarcia;

import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.extension.ExtendWith;

@Tag("timed")
@ExtendWith(TimingExtension.class)
public interface TimeExecutionLogger {
}
```

总的来说，我们可以在我们的 Jupiter 测试中使用这些接口：

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.Assertions.*assertEquals*;

import org.junit.jupiter.api.Test;

class TestInterfaceTest implements TestLifecycleLogger, 
        TimeExecutionLogger, 
        TestInterfaceDynamicTestsDemo {

    @Test
    void isEqualValue() {
        *assertEquals*(1, 1);
    }

}
```

在这个测试中，实现所有先前定义的接口将提供默认方法中实现的日志记录功能：

![](img/00076.gif)

实现多个接口的测试的控制台输出

# 测试模板

`@TestTemplate` 方法不是一个常规的测试用例，而是测试用例的模板。像这样注释的方法将根据注册的提供程序返回的调用上下文多次调用。因此，测试模板与注册的 `TestTemplateInvocationContextProvider` 扩展一起使用。

```java
@TestTemplate, and also declaring an extension of the type MyTestTemplateInvocationContextProvider:
```

```java
package io.github.bonigarcia;

import org.junit.jupiter.api.TestTemplate;
import org.junit.jupiter.api.extension.ExtendWith;

class TemplateTest {

    @TestTemplate
    @ExtendWith(MyTestTemplateInvocationContextProvider.class)
    void testTemplate(String parameter) {
        System.*out*.println(parameter);
    }

}
```

所需的提供程序实现了 Jupiter 接口 `TestTemplateInvocationContextProvider`。检查这个类的代码，我们可以看到如何为测试模板提供了两个 `String` 参数（在这种情况下，这些参数的值为 `parameter-1` 和 `parameter-2`）：

```java
package io.github.bonigarcia;

import java.util.Collections;
import java.util.List;
import java.util.stream.Stream;
import org.junit.jupiter.api.extension.Extension;
import org.junit.jupiter.api.extension.ExtensionContext;
import org.junit.jupiter.api.extension.ParameterContext;
import org.junit.jupiter.api.extension.ParameterResolver;
import org.junit.jupiter.api.extension.TestTemplateInvocationContext;
import org.junit.jupiter.api.extension.TestTemplateInvocationContextProvider;

public class MyTestTemplateInvocationContextProvider
        implements TestTemplateInvocationContextProvider {

    @Override
    public boolean supportsTestTemplate(ExtensionContext context) {
        return true;
    }

    @Override
    public Stream<TestTemplateInvocationContext> 
        provideTestTemplateInvocationContexts(
       ExtensionContext context) {
        return Stream.*of*(invocationContext("parameter-1"),
                invocationContext("parameter-2"));
    }

    private TestTemplateInvocationContext invocationContext(String parameter) {
        return new TestTemplateInvocationContext() {
            @Override
            public String getDisplayName(int invocationIndex) {
                return parameter;
            }

            @Override
            public List<Extension> getAdditionalExtensions() {
                return Collections.singletonList(new ParameterResolver() {
                    @Override
                    public boolean supportsParameter(
                            ParameterContext parameterContext,
                            ExtensionContext extensionContext) {
                        return parameterContext.getParameter().getType()
                             .equals(String.class);
                    }

                    @Override
                    public Object resolveParameter(
                            ParameterContext parameterContext,
                            ExtensionContext extensionContext) {
                        return parameter;
                    }
                });
            }
        };
    }

}
```

当测试被执行时，测试模板的每次调用都会像常规的 `@Test` 一样行为。在这个例子中，测试只是将参数写入标准输出。

![](img/00077.gif)

测试模板示例的控制台输出

# 参数化测试

参数化测试是一种特殊类型的测试，其中数据输入被注入到测试中，以便重用相同的测试逻辑。这个概念在 JUnit 4 中已经讨论过，如 第一章 *关于软件质量和 Java 测试的回顾* 中所解释的。正如我们所期望的，参数化测试也在 JUnit 5 中实现了。

首先，为了在 Jupiter 中实现参数化测试，我们需要将 `junit-jupiter-params` 添加到我们的项目中。在使用 Maven 时，这意味着添加以下依赖项：

```java
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-params</artifactId>
    <version>${junit.jupiter.version}</version>
    <scope>test</scope>
</dependency>
```

通常情况下，建议使用最新版本的构件。我们可以通过 Maven 中央仓库 ([`search.maven.org/`](http://search.maven.org/)) 找到最新版本。

在使用 Gradle 时，可以声明 `junit-jupiter-params` 依赖项如下：

```java
dependencies {
      testCompile("org.junit.jupiter:junit-jupiter-
      params:${junitJupiterVersion}")
}
```

然后，我们需要使用注解 `@ParameterizedTest`（位于包 `org.junit.jupiter.params` 中）来声明一个 Java 类中的方法作为参数化测试。这种类型的测试行为与常规的 `@Test` 完全相同，意味着所有的生命周期回调（`@BeforeEach`、`@AfterEach` 等）和扩展都会以相同的方式工作。

然而，使用 `@ParameterizedTest` 还不足以实现参数化测试。与 `@ParameterizedTest` 一起，我们需要至少指定一个参数提供程序。正如我们将在本节中发现的那样，JUnit 5 实现了不同的注解，以从不同的来源提供数据输入（即测试的参数）。这些参数提供程序（在 JUnit 5 中作为注解实现）总结在下表中（这些注解中的每一个都位于包 `org.junit.jupiter.params.provider` 中）：

| **参数提供程序注解** | **描述** |
| --- | --- |
| `@ValueSource` | 用于指定 `String`、`int`、`long` 或 `double` 的字面值数组 |
| `@EnumSource` | 用于指定指定枚举类型（`java.lang.Enum`）的常量的参数源 |
| `@MethodSource` | 提供对声明此注解的类的静态方法返回的值的访问权限 |
| `@CsvSource` | 从其属性读取逗号分隔值（CSV）的参数源 |
| `@CsvFileSource` | 用于从一个或多个类路径资源加载 CSV 文件的参数源 |
| `@ArgumentsSource` | 用于指定自定义参数提供程序（即实现接口 `org.junit.jupiter.params.provider.ArgumentsProvider` 的 Java 类） |

# @ValueSource

`@ValueSource`注解与`@ParameterizedTest`结合使用，用于指定一个参数化测试，其中参数源是`String`，`int`，`long`或`double`的文字值数组。这些值在注解中指定，使用`strings`，`ints`，`longs`或`doubles`元素。考虑以下示例：

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.Assertions.*assertNotNull*;

import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;
 class ValueSourceStringsParameterizedTest {

    @ParameterizedTest
    @ValueSource(strings = { "Hello", "World" })
    void testWithStrings(String argument) {
      System.*out*.println("Parameterized test with (String) parameter:  "             
        + argument);
      *assertNotNull*(argument);
    }
}
```

此类的方法（`testWithStrings`）定义了一个参数化测试，其中指定了一个 String 数组。由于在`@ValueSource`注解中指定了两个 String 参数（在本例中为`"Hello"`和`"World"`），测试逻辑将被执行两次，每次一个值。这些数据通过方法的参数注入到测试方法中，这里是通过名为 argument 的`String`变量。总的来说，当执行此测试类时，输出将如下所示：

![](img/00078.gif)

使用*@ValueSource*和 String 参数提供程序执行参数化测试

我们还可以在`@ValueSource`注解中使用整数原始类型（`int`，`long`和`double`）。以下示例演示了如何使用。此示例类的方法（命名为`testWithInts`，`testWithLongs`和`testWithDoubles`）使用`@ValueSource`注解以整数值的形式定义参数，分别使用原始类型 int，long 和 double。为此，需要指定`@ValueSource`的`ints`，`longs`和`doubles`元素：

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.Assertions.*assertNotNull*;

import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;

class ValueSourcePrimitiveTypesParameterizedTest {

   @ParameterizedTest
    @ValueSource(ints = { 0, 1 })
    void testWithInts(int argument) {
        System.*out*.println("Parameterized test with (int) argument: " + 
            argument);
        *assertNotNull*(argument);
    }

    @ParameterizedTest
    @ValueSource(longs = { 2L, 3L })
    void testWithLongs(long argument) {
        System.*out*.println(
        "Parameterized test with (long) 
              argument: " + argument);
        *assertNotNull*(argument);
    }

    @ParameterizedTest
    @ValueSource(doubles = { 4d, 5d })
    void testWithDoubles(double argument) {
        System.*out*.println("Parameterized test with (double)
              argument: " + argument);
        *assertNotNull*(argument);
    }

}
```

正如图中所示，每个测试都会执行两次，因为在每个`@ValueSource`注解中，我们指定了两个不同的输入参数（类型为`int`，`long`和`double`）。

![](img/00079.gif)

使用*@ValueSource*和原始类型执行参数化测试

# @EnumSource

`@EnumSource`注解允许指定一个参数化测试，其中参数源是一个 Java 枚举类。默认情况下，枚举的每个值将被用来提供参数化测试，依次进行测试。

例如，在以下测试类中，方法`testWithEnum`使用`@ParameterizedTest`与`@EnumSource`一起进行注释。正如我们所看到的，此注解的值是`TimeUnit.class`，这是一个标准的 Java 注解（java.util.concurrent 包），用于表示时间持续。此枚举中定义的可能值是`NANOSECONDS`，`MICROSECONDS`，`MILLISECONDS`，`SECONDS`，`MINUTES`，`HOURS`和`DAYS`：

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.Assertions.*assertNotNull*;

import java.util.concurrent.TimeUnit;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.EnumSource;

class EnumSourceParameterizedTest {

    @ParameterizedTest
    @EnumSource(TimeUnit.class)
    void testWithEnum(TimeUnit argument) {
        System.*out*.println("Parameterized test with (TimeUnit)         
            argument: " + argument);
        *assertNotNull*(argument);
    }

}
```

因此，此测试的执行将进行七次，即每个`TimeUnit`枚举值一次。在执行测试时，可以在输出控制台的跟踪中检查到这一点：

![](img/00080.gif)

使用*@EnumSource*和*TimeUnit.class*执行参数化测试

此外，`@EnumSource`注解允许以多种方式过滤枚举的成员。为了实现这种选择，可以在`@EnumSource`注解中指定以下元素：

+   `mode`：常量值，确定过滤的类型。这在内部类`org.junit.jupiter.params.provider.EnumSource.Mode`中定义为枚举，并且可能的值是：

+   `INCLUDE`：用于选择那些名称通过`names`元素提供的值。这是默认选项。

+   `EXCLUDE`：用于选择除了通过`names`元素提供的所有值之外的所有值。

+   `MATCH_ALL`：用于选择那些名称与`names`元素中的模式匹配的值。

+   `MATCH_ANY`：用于选择那些名称与`names`元素中的任何模式匹配的值。

+   `names`：字符串数组，允许选择一组`enum`常量。包含/排除的标准与 mode 的值直接相关。此外，该元素还允许定义正则表达式来选择要匹配的`enum`常量的名称。

考虑以下例子。在这个类中，有三个参数化测试。第一个，名为`testWithFilteredEnum`，使用`TimeUnit`类来提供`@EnumSource`参数提供程序。此外，枚举常量集使用元素名称进行过滤。正如我们所看到的，只有常量``"DAYS"``和``"HOURS"``将用于提供这个测试（请注意，默认模式是`INCLUDE`）：

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.Assertions.*assertNotNull*;
import static org.junit.jupiter.params.provider.EnumSource.Mode.*EXCLUDE*;
import static org.junit.jupiter.params.provider.EnumSource.Mode.*MATCH_ALL*;

import java.util.concurrent.TimeUnit;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.EnumSource;

class EnumSourceFilteringParameterizedTest {

    @ParameterizedTest
    @EnumSource(value = TimeUnit.class, names = { "DAYS", "HOURS" })
    void testWithFilteredEnum(TimeUnit argument) {
        System.*out*.println("Parameterized test with some (TimeUnit) 
            argument: "+ argument);
        *assertNotNull*(argument);
    }

    @ParameterizedTest
    @EnumSource(value = TimeUnit.class, mode = *EXCLUDE*, names = { 
    "DAYS", "HOURS" })
    void testWithExcludeEnum(TimeUnit argument) {
        System.*out*.println("Parameterized test with excluded (TimeUnit) 
            argument: " + argument);
        *assertNotNull*(argument);
    }

    @ParameterizedTest
    @EnumSource(value = TimeUnit.class, mode = *MATCH_ALL*, names = 
    "^(M|N).+SECONDS$")
    void testWithRegexEnum(TimeUnit argument) {
        System.*out*.println("Parameterized test with regex filtered 
            (TimeUnit) argument: " + argument);
        *assertNotNull*(argument);
    }

}
```

因此，在控制台中执行这个类时，我们得到的输出如下。关于第一个测试，我们可以看到只有``"DAYS"``和``"HOURS"``的迹象：

![](img/00081.gif)

使用*@EnumSource*使用过滤功能执行参数化测试

现在考虑第二个测试方法，名为`testWithExcludeEnum`。这个测试与之前完全相同，唯一的区别是这里的模式是`EXCLUSION`（而不是之前测试中默认选择的`INCLUSION`）。总的来说，在执行中（见之前的截图）我们可以看到这个测试被执行了五次，每次都是使用一个不同于`DAYS`和`HOURS`的枚举常量。要检查这一点，可以跟踪带有句子“使用排除（TimeUnit）参数的参数化测试”的迹象。

这个类的第三个也是最后一个方法（名为`testWithRegexEnum`）定义了一个包含模式，`MATCH_ALL`，使用正则表达式来过滤枚举（在这种情况下，也是`TimeUnit`）。在这个例子中使用的具体正则表达式是`^(M|N).+SECONDS$`，这意味着只有以`M`或`N`开头并以`SECONDS`结尾的枚举常量将被包含在内。正如在执行截图中可以看到的，有三个`TimeUnit`常量符合这些条件：`NANOSECONDS`、`MICROSECONDS`和`MILISECONDS`。

# @MethodSource

注解`@MethodSource`允许定义静态方法的名称，该方法提供测试的参数作为 Java 8 的`Stream`。例如，在下面的例子中，我们可以看到一个参数化测试，其中参数提供程序是一个名为`stringProvider`的静态方法。在这个例子中，这个方法返回一个`String`的`Stream`，因此测试方法的参数（名为`testWithStringProvider`）接受一个`String`参数：

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.Assertions.*assertNotNull*;

import java.util.stream.Stream;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.MethodSource;

class MethodSourceStringsParameterizedTest {

    static Stream<String> stringProvider() {
        return Stream.*of*("hello", "world");
    }

    @ParameterizedTest
    @MethodSource("stringProvider")
    void testWithStringProvider(String argument) {
        System.*out*.println("Parameterized test with (String) argument: "
           + argument);
        *assertNotNull*(argument);
    }

}
```

在运行示例时，我们可以看到测试被执行两次，每次都是使用`Stream`中包含的`String`。

![](img/00082.gif)

使用*@MethodSource*和 String 参数提供程序执行参数化测试

`Stream`中包含的对象的类型不需要是`String`。实际上，这种类型可以是任何类型。让我们考虑另一个例子，其中`@MethodSource`与一个静态方法关联，该方法返回自定义对象的`Stream`。在这个例子中，这种类型被命名为`Person`，并且在这里它被实现为一个内部类，具有两个属性（`name`和`surname`）。

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.Assertions.*assertNotNull*;

import java.util.stream.Stream;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.MethodSource;

class MethodSourceObjectsParameterizedTest {

    static Stream<Person> personProvider() {
        Person john = new Person("John", "Doe");
        Person jane = new Person("Jane", "Roe");
        return Stream.*of*(john, jane);
    }

    @ParameterizedTest
    @MethodSource("personProvider")
    void testWithPersonProvider(Person argument) {
        System.*out*.println("Parameterized test with (Person) argument: " + 
                argument);
        *assertNotNull*(argument);
    }

    static class Person {
        String name;
        String surname;

        public Person(String name, String surname) {
            this.name = name;
            this.surname = surname;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public String getSurname() {
            return surname;
        }

        public void setSurname(String surname) {
            this.surname = surname;
        }

        @Override
        public String toString() {
            return "Person [name=" + name + ", surname=" + surname + "]";
        }

    }

}
```

正如下面的截图所示，在执行这个例子时，参数化测试被执行两次，每次都是使用`Stream`中包含的`Person`对象（``"John Doe"``和``"Jane Roe"`）。

![](img/00083.gif)

使用*@MethodSource*和自定义对象参数提供程序执行参数化测试

我们还可以使用`@MethodSource`来指定包含整数原始类型的参数提供程序，具体来说是`int`、`double`和`long`。以下的类包含了一个例子。我们可以看到三个参数化测试。第一个（名为`testWithIntProvider`）使用注解`@MethodSource`与静态方法`intProvider`关联。在这个方法的主体中，我们使用标准的 Java 类`IntStream`来返回一个`int`值的`Stream`。第二个和第三个测试（名为`testWithDoubleProvider`和`testWithLongProvider`）非常相似，但分别使用`double`和`long`值的`Stream`：

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.Assertions.*assertNotNull*;

import java.util.stream.DoubleStream;
import java.util.stream.IntStream;
import java.util.stream.LongStream;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.MethodSource;

class MethodSourcePrimitiveTypesParameterizedTest {

    static IntStream intProvider() {
        return IntStream.*of*(0, 1);
    }

    @ParameterizedTest
    @MethodSource("intProvider")
    void testWithIntProvider(int argument) {
        System.*out*.println("Parameterized test with (int) argument: " + 
            argument);
        *assertNotNull*(argument);
    }

    static DoubleStream doubleProvider() {
        return DoubleStream.*of*(2d, 3d);
    }

    @ParameterizedTest
    @MethodSource("doubleProvider")
    void testWithDoubleProvider(double argument) {
        System.*out*.println(
            "Parameterized test with (double) argument: " + argument);
        *assertNotNull*(argument);
    }

    static LongStream longProvider() {
        return LongStream.*of*(4L, 5L);
    }

    @ParameterizedTest
    @MethodSource("longProvider")
    void testWithLongProvider(long argument) {
        System.*out*.println(
            "Parameterized test with (long) argument: " + argument);
        *assertNotNull*(argument);
   }

}
```

因此，在执行这个类时，将执行六个测试（三个参数化测试，每个测试有两个参数）。

在下面的截图中，我们可以通过跟踪每个测试写入标准输出的迹象来检查这一点：

![](img/00084.gif)

使用*@MethodSource*和原始类型参数提供程序执行参数化测试

最后，关于`@MethodSource`参数化测试，值得知道的是，方法提供程序允许返回不同类型（对象或原始类型）的流。这对于真实世界的测试用例非常方便。例如，下面的类实现了一个参数化测试，其中参数提供程序是一个返回混合类型（`String`和`int`）参数的方法。这些参数作为方法参数（在示例中称为 first 和 second）注入到测试中。

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.Assertions.*assertNotEquals*;
import static org.junit.jupiter.api.Assertions.*assertNotNull*;

import java.util.stream.Stream;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.Arguments;
import org.junit.jupiter.params.provider.MethodSource;

class MethodSourceMixedTypesParameterizedTest {

    static Stream<Arguments> stringAndIntProvider() {
        return Stream.*of*(Arguments.*of*("Mastering", 10),
            Arguments.*of*("JUnit 5", 20));
    }

    @ParameterizedTest
    @MethodSource("stringAndIntProvider")
    void testWithMultiArgMethodSource(String first, int second) {
        System.*out*.println("Parameterized test with two arguments: 
            (String) " + first + " and (int) " + second);
        *assertNotNull*(first);
        *assertNotEquals*(0, second);
    }
}
```

通常情况下，测试执行将作为流中包含的条目。在这种情况下，有两个条目："Mastertering"和 10，然后是"JUnit 5"和 20。

![](img/00085.gif)

使用*@MethodSource*执行参数化测试，使用不同类型的参数

# @CsvSource 和@CsvFileSource

使用逗号分隔的值（CSV）指定参数化测试参数的另一种方法。这可以通过注解`@CsvSource`来实现，它允许将 CSV 内容嵌入到注解的值中作为字符串。

考虑以下示例。它包含了一个 Jupiter 参数化测试（名为`testWithCsvSource`），该测试使用了注解`@CsvSource`。该注解包含一个字符串数组。在数组的每个元素中，我们可以看到由逗号分隔的不同值。

CSV 的内容会自动转换为字符串和整数。要了解 JUnit 5 在参数中进行的隐式类型转换的更多信息，请查看本章节中的*参数转换*部分。

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.Assertions.*assertNotEquals*;
import static org.junit.jupiter.api.Assertions.*assertNotNull*;

import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;

class CsvSourceParameterizedTest {

    @ParameterizedTest
    @CsvSource({ "hello, 1", "world, 2", "'happy, testing', 3" })
    void testWithCsvSource(String first, int second) {
        System.*out*.println("Parameterized test with (String) " + first
            + " and (int) " + second);
        *assertNotNull*(first);
        *assertNotEquals*(0, second);
    }

}
```

总的来说，当执行这个测试类时，将会有三个单独的测试，每个测试对应数组中的一个条目。每次执行都会传递两个参数给测试。第一个参数名为`first`，类型为`String`，第二个参数名为`second`，类型为`int`。

![](img/00086.gif)

使用*@CsvSource*执行参数化测试

如果 CSV 数据量很大，使用注解`@CsvFileSource`可能更方便。该注解允许使用项目类路径中的 CSV 文件来为参数化测试提供数据。在下面的示例中，我们使用文件`input.csv`：

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.Assertions.*assertNotEquals*;
import static org.junit.jupiter.api.Assertions.*assertNotNull*;

import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvFileSource;

class CsvFileSourceParameterizedTest {

    @ParameterizedTest
    @CsvFileSource(resources = "/input.csv")
    void testWithCsvFileSource(String first, int second) {
        System.*out*.println("Yet another parameterized test with 
            (String) " + first + " and (int) " + second);
        *assertNotNull*(first);
        *assertNotEquals*(0, second);
    }

}
```

在内部，注解`@CsvFileSource`使用标准 Java 类`java.lang.Class`的`getResourceAsStream()`方法来定位文件。因此，文件的路径被解释为相对于我们调用它的包类的本地路径。由于我们的资源位于类路径的根目录（在示例中位于文件夹`src/test/resources`中），我们需要将其定位为`/input.csv`。

![](img/00087.jpeg)

在*@CsvFileSource*示例中的 input.csv 的位置和内容

下面的截图显示了使用 Maven 执行测试时的输出。由于 CSV 有三行数据，因此有三个测试执行，每个执行有两个参数（第一个为`String`，第二个为`int`）：

![](img/00088.gif)

使用*@CsvFileSource*执行参数化测试

# @ArgumentsSource

JUnit 5 中用于指定参数化测试参数来源的最后一个注解是`@ArgumentsSource`。使用此注解，我们可以指定一个自定义的（并且可以在不同测试中重用）类，该类将包含测试的参数。该类必须实现接口`org.junit.jupiter.params.provider.ArgumentsProvider`。

让我们看一个例子。下面的类实现了一个 Jupiter 参数化测试，其中参数来源将在类`CustomArgumentsProvider1`中定义：

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.Assertions.*assertNotNull*;
import static org.junit.jupiter.api.Assertions.*assertTrue*;

import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ArgumentsSource;

class ArgumentSourceParameterizedTest {

    @ParameterizedTest
    @ArgumentsSource(CustomArgumentsProvider1.class)
    void testWithArgumentsSource(String first, int second) {
        System.*out*.println("Parameterized test with (String) " + first
             + " and (int) " + second);
        *assertNotNull*(first);
        *assertTrue*(second > 0);
    }

}
```

这个类（名为`CustomArgumentsProvider1`）已经在我们这边实现了，由于它实现了`ArgumentsProvider`接口，必须重写`provideArguments`方法，在这个方法中实现了测试参数的实际定义。从例子的代码中可以看出，这个方法返回一个`Arguments`的`Stream`。在这个例子中，我们返回了一个`Stream`中的一对条目，每个条目分别有两个参数（`String`和`int`）：

```java
package io.github.bonigarcia;

import java.util.stream.Stream;
import org.junit.jupiter.api.extension.ExtensionContext;
import org.junit.jupiter.params.provider.Arguments;
import org.junit.jupiter.params.provider.ArgumentsProvider;
 public class CustomArgumentsProvider1 implements ArgumentsProvider {

    @Override
    public Stream<? extends Arguments> provideArguments(
            ExtensionContext context) {
        System.*out*.println("Arguments provider to test "
            + context.getTestMethod().get().getName());
        return Stream.*of*(Arguments.*of*("hello", 1), 
            Arguments.*of*("world", 2));
    }

}
```

还要注意，这个参数有一个`ExtensionContext`类型的参数（包`org.junit.jupiter.api.extension`）。这个参数非常有用，可以知道测试执行的上下文。正如这里的截图所示，`ExtensionContext` API 提供了不同的方法来找出测试实例的不同属性（测试方法名称、显示名称、标签等）。

在我们的例子（`CustomArgumentsProvider1`）中，上下文被用来将测试方法名称写入标准输出：

![](img/00089.jpeg)*ExtensionContext* API

因此，当执行这个例子时，我们可以看到两个测试被执行。此外，我们可以通过`ExtensionContext`对象内部的`ArgumentsProvider`实例来检查测试方法的日志跟踪：

![](img/00090.gif)

使用*@ArgumentsSource*执行参数化测试

多个参数来源可以应用于同一个参数化测试。事实上，在 Jupiter 编程模型中可以通过两种不同的方式来实现这一点：

+   使用多个`@ArgumentsSource`注解与相同的`@ParameterizedTest`。这可以通过`@ArgumentsSource`是一个`java.lang.annotation.Repeatable`注解来实现。

+   使用注解`@ArgumentsSources`（注意这里的来源是复数）。这个注解只是一个容器，用于一个或多个`@ArgumentsSource`。下面的类展示了一个简单的例子：

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.Assertions.*assertNotNull*;
import static org.junit.jupiter.api.Assertions.*assertTrue*;

import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ArgumentsSource;
import org.junit.jupiter.params.provider.ArgumentsSources;

class ArgumentSourcesParameterizedTest {

    @ParameterizedTest
    @ArgumentsSources({ 
    @ArgumentsSource(CustomArgumentsProvider1.class),
    @ArgumentsSource(CustomArgumentsProvider2.class) })
    void testWithArgumentsSource(String first, int second) {
        System.*out*.println("Parameterized test with (String) " + first
            + " and (int) " + second);
        *assertNotNull*(first);
        *assertTrue*(second > 0);
    }

}
```

假设第二个参数提供程序（`CustomArgumentsProvider2.**class**`）指定了两个或更多组参数，当执行测试类时，将有四个测试执行：

![](img/00091.gif)

使用*@ArgumentsSources*执行参数化测试

# 参数转换

为了支持`@CsvSource`和`@CsvFileSource`等用例，Jupiter 提供了一些内置的隐式转换器。此外，这些转换器可以根据特定需求实现显式转换器。本节涵盖了两种类型的转换。

# 隐式转换

在内部，JUnit 5 处理了一组规则，用于将参数从`String`转换为实际的参数类型。例如，如果`@ParameterizedTests`声明了一个`TimeUnit`类型的参数，但声明的来源是一个`String`，那么这个`String`将被转换为`TimeUnit`。下表总结了 JUnit 5 中参数化测试参数的隐式转换规则：

| **目标类型** | **示例** |
| --- | --- |
| `boolean/Boolean` | `"false"` -> `false` |
| `byte/Byte` | `"1"` -> `(byte) 1` |
| `char/Character` | `"a"` -> `'a'` |
| `short/Short` | `"2"` -> `(short) 2` |
| `int/Integer` | `"3"` -> `3` |
| `long/Long` | `"4"` -> `4L` |
| `float/Float` | `"5.0"` -> `5.0f` |
| `double/Double` | `"6.0"` -> `6.0d` |
| `Enum 子类` | `"SECONDS"` -> `TimeUnit.SECONDS` |
| `java.time.Instant` | `"1970-01-01T00:00:00Z"` -> `Instant.ofEpochMilli(0)` |
| `java.time.LocalDate` | `"2017-10-24"` -> `LocalDate.of(2017, 10, 24)` |
| `java.time.LocalDateTime` | `"2017-03-14T12:34:56.789"` -> `LocalDateTime.of(2017, 3, 14, 12, 34, 56, 789_000_000)` |
| `java.time.LocalTime` | `"12:34:56.789"` -> `LocalTime.of(12, 34, 56, 789_000_000)` |
| `java.time.OffsetDateTime` | `"2017-03-14T12:34:56.789Z"` -> `OffsetDateTime.of(2017, 3, 14, 12, 34, 56, 789_000_000, ZoneOffset.UTC)` |
| `java.time.OffsetTime` | `"12:34:56.789Z"` -> `OffsetTime.of(12, 34, 56, 789_000_000, ZoneOffset.UTC)` |
| `java.time.Year` | `"2017"` -> `Year.of(2017)` |
| `java.time.YearMonth` | `"2017-10"` -> `YearMonth.of(2017, 10)` |
| `java.time.ZonedDateTime` | `"2017-10-24T12:34:56.789Z"` -> `ZonedDateTime.of(2017, 10, 24, 12, 34, 56, 789_000_000, ZoneOffset.UTC)` |

下面的例子展示了隐式转换的几个例子。第一个测试（`testWithImplicitConversionToBoolean`）声明了一个`String`源为`"true"`，但预期的参数类型是`Boolean`。类似地，第二个测试（`"testWithImplicitConversionToInteger"`）对`String`进行了隐式转换为`Integer`。第三个测试（`testWithImplicitConversionToEnum`）将输入的`String`转换为`TimeUnit`（枚举），最后第四个测试（`testWithImplicitConversionToLocalDate`）进行了转换为`LocalDate`：

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.Assertions.*assertNotNull*;
import static org.junit.jupiter.api.Assertions.*assertTrue*;

import java.time.LocalDate;
import java.util.concurrent.TimeUnit;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;

class ImplicitConversionParameterizedTest {

    @ParameterizedTest
    @ValueSource(strings = "true")
    void testWithImplicitConversionToBoolean(Boolean argument) {
        System.*out*.println("Argument " + argument + " is a type of "
            + argument.getClass());
        *assertTrue*(argument);
    }

    @ParameterizedTest
    @ValueSource(strings = "11")
    void testWithImplicitConversionToInteger(Integer argument) {
        System.*out*.println("Argument " + argument + " is a type of "
            + argument.getClass());
        *assertTrue*(argument > 10);
    }

    @ParameterizedTest
    @ValueSource(strings = "SECONDS")
    void testWithImplicitConversionToEnum(TimeUnit argument) {
        System.*out*.println("Argument " + argument + " is a type of "
            + argument.getDeclaringClass());
        *assertNotNull*(argument.name());
    }

    @ParameterizedTest
    @ValueSource(strings = "2017-07-25")
    void testWithImplicitConversionToLocalDate(LocalDate argument) {
        System.*out*.println("Argument " + argument + " is a type of "
            + argument.getClass());
        *assertNotNull*(argument);
    }

}
```

我们可以在控制台中检查参数的实际类型。每个测试都会在标准输出中写入一行，显示每个参数的值和类型：

![](img/00092.gif)

使用隐式参数转换执行参数化测试

# 显式转换

如果 JUnit 5 提供的隐式转换不足以满足我们的需求，我们可以使用显式转换功能。有了这个功能，我们可以指定一个类来对参数类型进行自定义转换。这个自定义转换器使用`@ConvertWith`注释进行标识，指定要进行转换的参数。考虑下面的例子。这个参数化测试为其测试方法参数声明了一个自定义转换器：

```java
package io.github.bonigarcia;

import static org.junit.jupiter.api.Assertions.*assertNotNull*;

import java.util.concurrent.TimeUnit;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.converter.ConvertWith;
import org.junit.jupiter.params.provider.EnumSource;

class ExplicitConversionParameterizedTest {

    @ParameterizedTest
    @EnumSource(TimeUnit.class)
    void testWithExplicitArgumentConversion(
            @ConvertWith(CustomArgumentsConverter.class) String 
            argument) {
          System.*out*.println("Argument " + argument + " is a type of "
              + argument.getClass());
          *assertNotNull*(argument);
    }

}
```

我们的自定义转换器是一个扩展了 JUnit 5 的`SimpleArgumentConverter`的类。这个类重写了 convert 方法，在这个方法中进行了实际的转换。在这个例子中，我们简单地将任何参数源转换为`String`。

```java
package io.github.bonigarcia;

import org.junit.jupiter.params.converter.SimpleArgumentConverter;
 public class CustomArgumentsConverter extends SimpleArgumentConverter {

    @Override
    protected Object convert(Object source, Class<?> targetType) {
          return String.*valueOf*(source);
    }
}
```

总的来说，当测试被执行时，`TimeUnit`中定义的七个枚举常量将作为参数传递给测试，然后在`CustomArgumentsConverter`中转换为`String`：

![](img/00093.gif)

使用显式参数转换执行参数化测试

# 自定义名称

JUnit 5 中与参数化测试相关的最后一个特性与每次测试执行的显示名称有关。正如我们所学到的，参数化测试通常被执行为多个单独的测试。因此，为了追踪性，将每个测试执行与参数源关联起来是一个好的做法。

为此，注释`@ParameterizedTest`接受一个名为 name 的元素，在其中我们可以为测试执行指定自定义名称（`String`）。此外，在这个字符串中，我们可以使用几个内置的占位符，如下表所述：

| **占位符** | **描述** |
| --- | --- |
| `{index}` | 当前调用索引（第一个为 1，第二个为 2，...） |
| `{arguments}` | 逗号分隔的参数完整列表 |
| `{0}, {1}, …` | 一个单独参数的值（第一个为 0，第二个为 2，...） |

让我们看一个简单的例子。下面的类包含一个参数化测试，其参数是使用`@CsvSource`注释定义的。测试方法接受两个参数（`String`和`int`）。此外，我们使用占位符为注释`@ParameterizedTest`的元素名称指定了一个自定义消息，用于当前测试调用的占位符（`{index}`）以及每个参数的值：第一个（`{0}`）和第二个（`{1}`）：

```java
package io.github.bonigarcia;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.CsvSource;
 class CustomNamesParameterizedTest {

    @DisplayName("Display name of test container")
    @ParameterizedTest(name = "[{index}] first argument=\"{0}\", second                 
       argument={1}")
    @CsvSource({ "mastering, 1", "parameterized, 2", "tests, 3" })
    void testWithCustomDisplayNames(String first, int second) {
        System.*out*.println("Testing with parameters: " + first + " and " +       
           second);
    }

}
```

在 IDE（如下面的 IntelliJ 截图）中执行此测试时，我们可以看到每个测试执行的显示名称是不同的：

![](img/00094.jpeg)

在 IntelliJ IDE 中使用自定义名称执行参数化测试

# Java 9

Java 9 于 2017 年 9 月 21 日发布为**通用可用性**（**GA**）。Java 9 中有许多新功能。其中，模块化是 Java 9 的主要特性。

到目前为止，Java 中存在模块化问题，特别是对于大型代码库来说非常重要。每个公共类都可以被类路径中的任何其他类访问，导致意外使用类。此外，类路径存在潜在问题，例如无法知道是否存在重复的 JAR。为了解决这些问题，Java 9 提供了 Java 平台模块系统，允许创建模块化的 JAR 文件。这种类型的模块包含一个额外的模块描述符，称为`module-info.java`。这些文件的内容非常简单：它使用关键字 requires 声明对其他模块的依赖，并使用关键字`exports`导出自己的包。所有未导出的包默认情况下都被模块封装，例如：

```java
module mymodule {
  exports io.github.bonigarcia;

  requires mydependency;
}
```

我们可以表示这些模块之间的关系如下：

![](img/00095.jpeg)

Java 9 中模块之间的关系示例

Java 9 的其他新功能总结如下：

+   使用模块允许创建一个针对特定应用程序进行了优化的最小运行时 JDK，而不是使用完整的 JDK 安装。这可以通过使用 JDK 9 附带的工具*jlink*来实现。

+   Java 9 提供了一个交互式环境，可以直接从 shell 中执行 Java 代码。这种类型的实用程序通常被称为**Read-Eval-Print-Loop**（**REPL**），在 JDK 9 中称为**JShell**。

+   集合工厂方法，Java 9 提供了创建集合（例如列表或集合）并在一行中填充它们的能力：

```java
      Set<Integer> ints = Set.of(1, 2, 3);
      List<String> strings = List.*of*("first", "second");
```

+   **Stream API 改进**：流是在 Java 8 中引入的，它们允许在集合上创建声明性转换管道。在 Java 9 中，Stream API 添加了`dropWhile`、`takeWhile`和`ofNullable`方法。

+   **私有接口方法**：Java 8 在接口上提供了默认方法。到目前为止，Java 8 中默认方法的限制是默认方法必须是公共的。现在，在 Java 9 中，这些默认方法也可以是私有的，有助于更好地结构化它们的实现。

+   **HTTP/2**：Java 9 支持开箱即用的 HTTP 版本 2 和 WebSockets。

+   **多版本 JAR**：此功能允许根据执行 JAR 的 JRE 版本创建类的替代版本。为此，在文件夹`META-INF/versions/<java-version>`下，我们可以指定不同版本的已编译类，仅当 JRE 版本与该版本匹配时才使用。

+   **改进的 Javadoc**：最后但并非最不重要的是，Java 9 允许创建具有集成搜索功能的 HTML5 兼容 Javadoc。

# JUnit 5 和 Java 9 的兼容性

自 M5 以来，所有 JUnit 5 构件都附带了为 Java 9 编译的模块描述符，在其 JAR 清单（文件`MANIFEST.MF`）中声明。例如，构件`junit-jupiter-api` M6 的清单内容如下：

```java
Manifest-Version: 1.0
Implementation-Title: junit-jupiter-api
Automatic-Module-Name: org.junit.jupiter.api
Build-Date: 2017-07-18
Implementation-Version: 5.0.0-M6
Built-By: JUnit Team
Specification-Vendor: junit.org
Specification-Title: junit-jupiter-api
Implementation-Vendor: junit.org
Build-Revision: 3e6482ab8b0dc5376a4ca4bb42bef1eb454b6f1b
Build-Time: 21:26:15.224+0200
Created-By: 1.8.0_131 (Oracle Corporation 25.131-b11)
Specification-Version: 5.0.0
```

关于 Java 9，有趣的是声明`Automatic-Module-Name`。这允许测试模块通过将以下行添加到其模块描述符文件（`module-info.java`）来要求 JUnit 5 模块：

```java
module foo.bar {
      requires org.junit.jupiter.api;
}
```

# JUnit 5.0 之后

JUnit 5.0 GA（正式版本）于 2017 年 9 月 10 日发布。此外，JUnit 是一个不断发展的项目，新功能计划在下一个版本 5.1 中发布（目前尚未安排发布日程）。JUnit 5 的下一个版本的待办事项可以在 GitHub 上查看：[`github.com/junit-team/junit5/milestone/3`](https://github.com/junit-team/junit5/milestone/3)。其中，计划为 JUnit 5.1 添加以下功能：

+   场景测试：这个功能涉及在一个类中对不同的测试方法进行排序的能力。为了做到这一点，计划使用以下注释：

+   `@ScenarioTest`：用于表示测试类包含组成单个场景测试的步骤的类级别注释。

+   `@Step`：用于表示测试方法是场景测试中的单个步骤的方法级别注释。

+   支持并行测试执行：并发是 JUnit 5.1 中需要改进的主要方面之一，因此计划支持开箱即用的并发测试执行。

+   提前终止动态测试的机制：这是对 JUnit 5.0 对动态测试的增强支持，引入了超时以在测试自行终止之前停止执行（以避免不受控制的非确定性执行）。

+   测试报告方面的几项改进，比如捕获`stdout`/`stderr`并包含在测试报告中，提供了可靠的方式来获取执行测试方法的类（类名），或者在测试报告中指定测试的顺序，等等。

# 总结

本章包含了一个全面的摘要，介绍了编写丰富的 Jupiter 测试的先进能力。首先，我们已经了解到参数可以被注入到测试类的构造函数和方法中。JUnit 5 提供了三个参数解析器，分别是用于类型为`TestInfo`的参数（用于检索有关当前测试的信息），用于类型为`RepetitionInfo`的参数（用于检索有关当前重复的信息），以及用于类型为`TestReporter`的参数（用于发布有关当前测试运行的附加数据）。

Jupiter 中实现的另一个新功能是动态测试的概念。到目前为止，在 JUnit 3 和 4 中，测试是在编译时定义的（即静态测试）。Jupiter 引入了`@TestFactory`注解，允许在运行时生成测试。Jupiter 编程模型提供的另一个新概念是测试模板。这些模板使用`@TestTemplate`注解定义，不是常规的测试用例，而是测试用例的模板。

JUnit 5 实现了对参数化测试的增强支持。为了实现这种类型的测试，必须使用`@ParameterizedTest`注解。除了这个注解，还应该指定一个参数提供者。为此，Jupiter 提供了几个注解：`@ValueSource`、`@EnumSource`、`@MethodSource`、`@CsvSource`、`@CsvFileSource`和`@ArgumentSource`。

在第五章中，*JUnit 5 与外部框架的集成*，我们将学习 JUnit 5 如何与外部框架交互。具体来说，我们将回顾几个 JUnit 5 扩展，它们提供了使用 Mockito、Spring、Selenium、Cucumber 或 Docker 的能力。此外，我们还介绍了一个 Gradle 插件，允许在 Android 项目中执行测试。最后，我们将了解如何使用几个 REST 库（例如 REST Assured 或 WireMock）来测试 RESTful 服务。
