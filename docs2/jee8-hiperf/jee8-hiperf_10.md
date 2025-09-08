# 持续性能评估

我们在前一章中看到，驱动基准测试需要工作，并且不是一个小的操作。然而，它并不能解决软件开发的所有需求，因为你只是偶尔进行它。此外，在两次基准测试之间，你可能会遇到巨大的性能回归，而这些回归没有被验证应用程序功能和行为的单元测试捕捉到。

为了解决这个问题，尝试添加针对性能的专用测试是一个好主意。这正是本章的内容。因此，我们将学习：

+   你可以使用哪些工具来*测试*性能

+   你如何为性能设置一些持续集成

+   在这种背景下如何编写性能测试

# 编写性能测试——陷阱

性能测试有一些挑战，当你编写测试时需要考虑，以避免出现大量误报结果（实际性能可接受但测试未通过的情况）。

你最常遇到的问题包括：

+   如何管理外部系统：我们知道外部系统在当今的应用程序中非常重要，但在测试期间拥有它们并不总是微不足道的。

+   如何确保确定性：持续集成/测试平台通常是共享的。你如何确保你有必要的资源，以便有确定性的执行时间，并且不会被拖慢，因为另一个构建正在使用所有可用资源？

+   如何处理基础设施：为了进行端到端测试，你需要多个注入器（客户端）和可能多个后端服务器。如果你使用像**Amazon Web Services**（**AWS**）这样的云平台，你如何确保它们可用，同时又不至于过于昂贵？

你可以将性能测试的设置看作是一个基准准备阶段。主要区别在于你不应长期依赖外部咨询，并且你将确保基准迭代几乎完全自动化——*几乎*因为如果测试失败，你仍然需要采取一些行动，否则拥有持续系统就没有意义了。

# 编写性能测试

性能测试存在于 Java EE 应用的各个阶段，因此，编写性能测试有几种方法。在本节中，我们将探讨几种主要方法，从最简单的一个（算法验证）开始。

# JMH – OpenJDK 工具

**Java Microbenchmark Harness**（**JMH**）是由 OpenJDK 团队开发的一个小型库——是的，就是那个做 JVM 的团队——它使你能够轻松地开发微基准测试。

微基准测试设计了一个基准测试，针对应用程序非常小的一部分。大多数情况下，你可以将其视为*单元基准测试*，与单元测试进行类比。

然而，在设置性能测试时，这是一个重要的因素，因为它将允许您快速识别由最近更改引入的关键性能回归。想法是将每个基准测试与一小部分代码相关联，而基准测试将包含几个层次。因此，如果基准测试失败，您将花费大量时间来识别原因，而不是简单地检查相关的代码部分。

# 使用 JMH 编写基准测试

在能够使用 JMH 编写代码之前，您需要将其添加为项目的依赖项。以下示例我们将使用 Maven 语法，但 Gradle 等其他构建工具也有相应的语法。

首先要做的是将以下依赖项添加到您的`pom.xml`文件中；我们将使用`test`范围，因为依赖项仅用于我们的性能测试，而不是“主”代码：

```java
<dependencies>
  <!-- your other dependencies -->

  <dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>${jmh.version}</version>
    <scope>test</scope>
  </dependency>
  <dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>${jmh.version}</version>
    <scope>test</scope> <!-- should be provided but for tests only this
    is more accurate -->
  </dependency>
</dependencies>
```

写这本书的时候，`jmh.version`属性可以取`1.19`的值。`jmh-core`依赖项将为您提供 JMH 本身及其基于注解的 API，而`jmh-generator-annprocess`则提供了一个注解处理器框架——在编译测试类/基准测试时使用，它将生成一些索引文件，这些文件由运行时用于执行基准测试和基准测试类本身。

# 状态

一旦您有了正确的依赖项，您就可以开发您的基准测试。API 非常直观。它使用*状态*的概念，状态与基准测试相关联，并具有生命周期和（易失性）存储。状态的生命周期是通过标记方法来定义的：

+   `@Setup`用于执行初始化任务

+   `@Teardown`用于在设置方法中释放任何创建的资源

状态类也可以包含带有`@Param`装饰的字段，使它们具有上下文化和可配置性（例如，根据执行启用不同的目标 URL 等）。

状态类用`@State`标记，它作为参数接受基准测试状态实例的作用域：

+   `Benchmark`意味着状态将在 JVM 的作用域内是一个单例。

+   `Group`定义了每个组的状态为一个单例。组是在执行期间将多个基准测试方法（场景）放入同一线程桶的方法。

+   `Thread`定义了每个线程的状态为一个单例（有点类似于 CDI 中的`@RequestScoped`）。

注意，您可以通过将运行时传递给`@Setup`和`@Teardown`以不同的`Level`来稍微改变生命周期管理方式：

+   `Trial`：这是默认的，正如我们刚才解释的，基准测试被视为一个整体

+   `Iteration`：生命周期在每次迭代（预热或测量）中执行

+   `Invocation`：状态实例的生命周期在每次方法调用中执行；实际上不推荐这样做，以避免一些测量错误

因此，一旦您有了状态对象，您就定义包含基准测试的类，这些基准测试实际上就是标记了`@Benchmark`的方法。然后，您可以在方法上添加几个注解来自定义基准测试的执行：

+   `@Measurement`：用于自定义基准的迭代次数或持续时间。

+   `@Warmup`：与`@Measurement`相同，但用于预热（这是一种基准的预执行，不计入测量，目标是只测量热 JVM 上的指标）。

+   `@OutputTimeUnit`：用于自定义用于指标的单元。

+   `@Threads`：用于自定义用于基准的线程数。你可以将其视为用户数。

+   `@Fork`：基准将在另一个 JVM 中执行，以避免测试副作用。这个注解允许你向分叉的 JVM 添加自定义参数。

+   `@OperationsPerInvocation`：如果你的基准方法中有一个循环，这个选项（一个数字）将标准化测量。例如，如果你在基准中执行了五次相同的操作，并将此值设置为五，那么执行时间将除以五。

+   `@Timeout`：它允许你定义基准执行的最大持续时间。如果超过这个时间，JMH 将中断线程。

+   `@CompilerControl`：用于自定义注解处理器生成代码的方式。对于我们的用例，你很少需要这个选项，但在调整某些代码部分时，测试它可能很有趣。

# 创建你的第一个 JMH 基准

在所有这些理论之后，这里是一个使用 JMH 开发的基本基准：

```java
public class QuoteMicrobenchmark {
    @Benchmark
    public void compute() {
        //
    }
}
```

在这种情况下，我们有一个名为`compute`的单个基准场景。它不使用任何状态，也没有自定义任何线程或分支计数。

实际上，这还不够，你通常会需要一个状态来获取服务。所以，它会更像这样：

```java
public class QuoteMicrobenchmark {
    @Benchmark
    public void compute(final QuoteState quoteState) {
        quoteState.service.findByName("test");
    }

    @State(Scope.Benchmark)
    public static class QuoteState {
        private QuoteService service;

        @Setup
        public void setup() {
            service = new QuoteService();
        }
    }
}
```

在这里，我们创建了一个嵌套的`QuoteState`实例，它将负责为我们获取服务，并将其注入到基准方法（`compute`）中，并使用它来获取我们的服务实例。这避免了每次迭代创建一个实例，因此避免了考虑容器启动持续时间。

这种实现方式在需要实际容器之前效果良好。但是，当你需要容器时，它将需要一些模拟，这绝对是你应该去除的东西——即使是对于单元测试——因为它不会在代码完全不使用 Java EE 的情况下（这也意味着在这种情况下你不需要模拟任何东西）代表接近真实部署的任何内容。

如果你使用支持独立 API 的 CDI 2.0 实现——并且你的应用程序不需要更多用于你想要测试的内容——那么你可以将状态更改为启动/停止 CDI 容器，并使用新的 CDI 独立 API 查找你需要的服务：

```java
@State(Scope.Benchmark)
public class QuoteState {
    private QuoteService service;
    private SeContainer container;

    @Setup
    public void setup() {
        container = SeContainerInitializer.newInstance().initialize();
        service = container.select(QuoteService.class).get();
    }

    @TearDown
    public void tearDown() {
        container.close();
    }
}
```

在这里，设置启动`container`——你可以在调用`initialize()`之前自定义要部署的类——然后使用`container`实例 API 查找`QuoteService`。`tearDown`方法只是正确关闭容器。

然而，在 GlassFish 中，你不能使用那个新 API。但是有一个来自 Java EE 6 的`EJBContainer`，它允许你结合`CDI`类做同样的事情：

```java
@State(Scope.Benchmark)
public class QuoteState {
    private QuoteService service;
    private EJBContainer container;

    @Setup
    public void setup() {
        container = EJBContainer.createEJBContainer(new HashMap<>());
        service = CDI.current().select(QuoteService.class).get();
    }

    @TearDown
    public void tearDown() {
        container.close();
    }
}
```

这与之前完全相同的逻辑，只是基于`EJBContainer` API 启动`container`。这看起来很棒，但它并不总是与所有容器都兼容。一个陷阱是，如果你没有任何 EJB，一些容器甚至不会尝试部署应用程序。

你可以找到几种解决方案，但更明智的解决方案是检查你是否真的需要完整的容器或只是子集——比如只是 CDI——在这种情况下，你只需启动这个子集（Weld 或 OpenWebBeans 仅用于 CDI，使用之前的状态）。如果你真的需要一个完整的容器，并且你的供应商不支持上述两种启动容器的方法，你也可以使用供应商特定的 API，模拟容器（但请注意，你将绕过一些执行时间*成本*），如果与你的最终容器足够接近，也可以使用另一个供应商，或者使用第三方容器管理器，如`arquillian`容器 API。

# 黑洞将成为一颗恒星

JMH 提供了一个特定的类——`org.openjdk.jmh.infra.Blackhole`——它可能看起来很奇怪，因为它主要只允许你`consume`一个实例。它是通过将其注入到基准的参数中检索的：

```java
public void compute(final QuoteState quoteState, final Blackhole blackhole) {
    blackhole.consume(quoteState.service.findByName("test"));
}
```

为什么不处理方法返回的值呢？记住，Oracle JVM 有一个称为**即时编译**（**JIT**）的功能，它会根据代码路径的统计数据在运行时优化代码。如果你不调用那个`consume`方法，你可能会最终测量不到你想要的实际代码，而是这种代码的一些非常优化的版本，因为，在大多数情况下，在你的基准方法中，你会忽略部分返回值，因此可以被*死代码消除*规则优化。

# 运行基准测试

JMH 提供了一个友好的 API 来运行基准测试。它基于定义执行选项，这些选项大致上是你可以与基准方法关联的选项（我们在上一节中通过注解看到的）和一个包含/排除列表，以选择要运行的类/方法。然后，你将这些选项传递给运行器并调用其`run()`方法：

```java
final Collection<RunResult> results = new Runner(
    new OptionsBuilder()
      .include("com.packt.MyBenchmark")
      .build())
 .run();
```

在这里，我们通过包含我们想要包含的基准类，从`OptionsBuilder`构建一个简单的`Options`实例。最后，我们通过具有与运行器相同名称的方法运行它，并将基准的结果收集到一个集合中。

# 将 JMH 与 JUnit 集成

JMH 没有官方的 JUnit 集成。尽管如此，自己实现它并不难。有许多可能的设计，但在这个书的背景下，我们将做以下事情：

+   我们的集成将通过 JUnit 运行器进行

+   基准类将通过提取测试类的嵌套类来识别，这将避免使用任何扫描来查找基准类或显式列出

+   我们将引入`@ExpectedPerformances`注解，以便能够根据执行添加断言

结构上，使用这种结构的微基准测试将看起来像这样：

```java
@RunWith(JMHRunner.class)
public class QuoteMicrobenchmarkTest {
    @ExpectedPerformances(score = 2668993660.)
    public static class QuoteMicrobenchmark {
        @Benchmark
        @Fork(1) @Threads(2)
        @Warmup(iterations = 5) @Measurement(iterations = 50)
        @ExpectedPerformances(score = 350000.)
        public void findById(final QuoteState quoteState, final
        Blackhole blackhole) {
            blackhole.consume(quoteState.service.findById("test"));
        }

        // other benchmark methods
    }

    public static class CustomerMicrobenchmark {
        // benchmark methods
    }
}
```

一旦你有了整体测试结构，你只需要在正确的嵌套类中添加基准测试本身。具体来说，一个基准测试类可以看起来像这样：

```java
public static/*it is a nested class*/ class QuoteMicrobenchmark {
    @Benchmark
    @Fork(1) @Threads(2)
    @Warmup(iterations = 5) @Measurement(iterations = 50)
    @ExpectedPerformances(score = 350000.)
    public void findById(final QuoteState quoteState, final Blackhole
    blackhole) {
        blackhole.consume(quoteState.service.findById("test"));
    }

    @Benchmark
    @Fork(1) @Threads(2) @Warmup(iterations = 10)
    @Measurement(iterations = 100)
    @ExpectedPerformances(score = 2660000.)
    public void findByName(final QuoteState quoteState, final Blackhole
    blackhole) {
        blackhole.consume(quoteState.service.findByName("test"));
    }

    @State(Scope.Benchmark)
    public static class QuoteState {
        private QuoteService service;

        @Setup public void setup() { service = new QuoteService(); }
    }
}
```

在这个测试类中，我们有两个基准测试类，它们使用不同的状态（我们 EE 应用程序中的不同服务），并且每个类都可以有不同的基准测试方法数量。执行由使用`@RunWith`设置的运行器处理，使用标准的 JUnit（4）API。我们将在所有基准测试上注意`@ExpectedPerformances`的使用。

如果你已经迁移到 JUnit 5，或者正在使用 TestNG，可以实现类似的集成，但你会使用 JUnit 5 的扩展，可能还会使用 TestNG 的抽象类或监听器。

在介绍如何实现该运行器之前，我们必须有这个`@ExpectedPerformances`注解。它是允许我们断言基准测试性能的注解。因此，我们至少需要：

+   一个分数（持续时间，但不指定单位，因为 JMH 支持自定义报告单位）

+   分数上有一定的容差，因为你不可能两次得到完全相同的分数

为了做到这一点，我们可以使用这个简单的定义：

```java
@Target(METHOD)
@Retention(RUNTIME)
public @interface ExpectedPerformances {
    double score();
    double scoreTolerance() default 0.1;
}
```

如你所见，我们告诉用户定义一个分数，但我们使用默认的分数容差`0.1`。在我们的实现中，我们将它视为百分比（10%）。这将避免在机器负载不稳定时运行作业时频繁失败。不要犹豫，降低这个值，甚至可以通过系统属性使其可配置。

要使我们的上一个片段正常工作，我们需要实现一个 JUnit 运行器。这是一个设计选择，但你也可以使用规则，暴露一些程序性 API 或不暴露。为了保持简单，我们在这里不会这样做，而是将整个基准测试设置视为通过注解完成的。然而，对于实际项目来说，启用环境（系统属性）以自定义注解值可能很有用。一个常见的实现是使用它们作为模板，并在 JVM 中将所有数值乘以一个配置的比率，以便在任何机器上运行测试。

我们的运行器将具有以下四个角色：

+   找到基准测试类

+   验证这些类的模型——通常，验证每个基准测试都有一个`@ExpectedPerformances`

+   运行每个基准测试

+   验证期望，如果出现回归，使测试失败

对于我们来说，扩展`ParentRunner<Class<?>>`更简单。我们可以使用`BlockJUnit4ClassRunner`，但它基于方法，而 JMH 只支持按类过滤执行。所以，我们暂时坚持使用它。如果你在每个嵌套类中只放一个基准测试，那么你可以模拟按方法运行的运行行为。

我们需要做的第一件事是找到我们的基准测试类。使用 JUnit 运行器 API，你可以通过`getTestClass()`访问测试类。要找到我们的基准测试，我们只需要通过`getClasses()`检查该类的嵌套类，并确保该类至少有一个方法上的`@Benchmark`来验证它是一个 JMH 类：

```java
children = Stream.of(getTestClass().getJavaClass().getClasses())
        .filter(benchmarkClass ->
        Stream.of(benchmarkClass.getMethods())
        .anyMatch(m -> m.isAnnotationPresent(Benchmark.class)))
        .collect(toList());
```

我们遍历测试类的所有嵌套类，然后只保留（或过滤）至少有一个基准方法的那部分。请注意，我们存储结果，因为我们的跑步者需要多次使用这些结果。

然后，验证过程就像遍历这些类及其基准方法，验证它们是否有`@ExpectedPerformances`注解：

```java
errors.addAll(children.stream()
        .flatMap(c -> Stream.of(c.getMethods()).filter(m ->
        m.isAnnotationPresent(Benchmark.class)))
        .filter(m ->
        !m.isAnnotationPresent(ExpectedPerformances.class))
        .map(m -> new IllegalArgumentException("No
        @ExpectedPerformances on " + m))
        .collect(toList()));
```

在这里，为了列出 JUnit 验证的错误，我们为每个用`@Benchmark`注解但没有`@ExpectedPerformances`注解的方法添加一个异常。我们首先通过将类转换为基准方法的流来实现，然后只保留没有`@ExpectedPerformances`注解的方法以保持*集合视图*。

最后，跑步者代码的最后关键部分是将一个类转换为实际的执行：

```java
final Collection<RunResult> results;
try {
    results = new Runner(buildOptions(benchmarkClass)).run();
} catch (final RunnerException e) {
    throw new IllegalStateException(e);
}

// for all benchmarks assert the performances from the results
final List<AssertionError> errors = Stream.of(benchmarkClass.getMethods())
        .filter(m -> m.isAnnotationPresent(Benchmark.class))
        .map(m -> {
            final Optional<RunResult> methodResult = results.stream()
                  .filter(r ->
                  m.getName().equals(r.getPrimaryResult().getLabel()))
                  .findFirst();
                  assertTrue(m + " didn't get any result",
                  methodResult.isPresent());

            final ExpectedPerformances expectations =
            m.getAnnotation(ExpectedPerformances.class);
            final RunResult result = results.iterator().next();
            final BenchmarkResult aggregatedResult =
            result.getAggregatedResult();

            final double actualScore =
            aggregatedResult.getPrimaryResult().getScore();
            final double expectedScore = expectations.score();
            final double acceptedError = expectedScore *
            expectations.scoreTolerance();
            try { // use assert to get a common formatting for errors
                assertEquals(m.getDeclaringClass().getSimpleName() +
                "#" + m.getName(), expectedScore, actualScore,
                acceptedError);
                return null;
            } catch (final AssertionError ae) {
                return ae;
            }
        }).filter(Objects::nonNull).collect(toList());
if (!errors.isEmpty()) {
    throw new AssertionError(errors.stream()
            .map(Throwable::getMessage)
            .collect(Collectors.joining("\n")));
}
```

首先，我们执行基准持有类并收集结果。然后，我们遍历基准方法，并对每个方法，我们提取结果（主要结果标签是方法名）。最后，我们提取我们的`@ExpectedPerformances`约束并将其与测试的主要结果进行比较。这里的技巧是捕获断言可以抛出的`AssertionError`，并将它们全部收集到一个列表中，然后转换为另一个`AssertionError`。你可以按你想要的方式格式化消息，但这样做可以保持标准 JUnit 的错误格式。这里的另一个提示是将基准类和方法放入错误消息中，以确保你可以识别哪个基准失败了。另一种方式是引入另一个注解，为每个基准使用自定义名称。

现在我们已经审视了所有的技术细节，让我们将它们整合起来。首先，我们将定义我们的跑步者将使用`Class`子类，这些子类将代表我们嵌套的每个类：

```java
public class JMHRunner extends ParentRunner<Class<?>> {
  private List<Class<?>> children;

  public JMHRunner(final Class<?> klass) throws InitializationError {
    super(klass);
  }

  @Override
  protected List<Class<?>> getChildren() {
    return children;
  }

  @Override
  protected Description describeChild(final Class<?> child) {
    return Description.createTestDescription(getTestClass().getJavaClass(), child.getSimpleName());
  }
```

在父构造函数（`super(klass)`）执行的代码中，JUnit 将触发一个测试验证，其中我们计算之前看到的子类，以便在`getChildren()`中返回它们，并让 JUnit 处理所有我们的嵌套类。我们实现`describeChild`以让 JUnit 将一个`Description`与每个嵌套类关联，并实现与 IDE 的更平滑集成（目标是当你运行测试时在树中显示它们）。为了计算子类并验证它们，我们可以使用这个 JUnit 的`collectInitializationErrors`实现——使用这个钩子可以避免在每个测试类中多次计算：

```java
  @Override
  protected void collectInitializationErrors(final List<Throwable>
  errors) {
    super.collectInitializationErrors(errors);

    children = Stream.of(getTestClass().getJavaClass().getClasses())
      .filter(benchmarkClass -> Stream.of(benchmarkClass.getMethods())
        .anyMatch(m -> m.isAnnotationPresent(Benchmark.class)))
      .collect(toList());

    errors.addAll(children.stream()
        .flatMap(c -> Stream.of(c.getMethods())
        .filter(m -> m.isAnnotationPresent(Benchmark.class)))
        .filter(m ->
        !m.isAnnotationPresent(ExpectedPerformances.class))
        .map(m -> new IllegalArgumentException("No
        @ExpectedPerformances on " + m))
        .collect(toList()));
  }
```

然后，我们需要确保我们可以正确地运行我们的子类（基准）。为此，我们扩展了另一个 JUnit 钩子，该钩子旨在运行每个子类。我们主要关心的是确保 JUnit 的`@Ignored`注解支持我们的子类：

```java
  @Override
  protected boolean isIgnored(final Class<?> child) {
    return child.isAnnotationPresent(Ignore.class);
  }

  @Override
  protected void runChild(final Class<?> child, final RunNotifier 
  notifier) {
    final Description description = describeChild(child);
    if (isIgnored(child)) {
      notifier.fireTestIgnored(description);
    } else {
      runLeaf(benchmarkStatement(child), description, notifier);
    }
  }
```

在 `runChild()` 中，我们通过添加作为我们测试执行的代码（基于 JMH 运行器，但封装在 JUnit `Statement` 中以使其能够与 JUnit 通知器集成）来委托给 JUnit 引擎执行。现在，我们只需要这个执行实现（`benchmarkStatement`）。这是通过以下代码来完成类的：

```java
  private Statement benchmarkStatement(final Class<?> benchmarkClass) {
    return new Statement() {
      @Override
      public void evaluate() throws Throwable {
        final Collection<RunResult> results;
        try {
          results = new Runner(buildOptions(benchmarkClass)).run();
        } catch (final RunnerException e) {
          throw new IllegalStateException(e);
        }

        assertResults(benchmarkClass, results);
      }
    };
  }

  // all options will use JMH annotations so just
  include the class to run
  private Options buildOptions(final Class<?> test) {
    return new OptionsBuilder()
        .include(test.getName().replace('$', '.'))
        .build();
  }

```

这重用了我们之前看到的所有内容；`buildOptions` 方法将强制 JMH 运行器使用基准测试上的注解来配置执行，我们一次只包含一个测试。最后，我们像之前解释的那样实现了 `assertResults` 方法：

```java
  public void assertResults(final Class<?> benchmarkClass, final
  Collection<RunResult> results) {
    // for all benchmarks assert the performances from the results
    final List<AssertionError> errors =
    Stream.of(benchmarkClass.getMethods())
        .filter(m -> m.isAnnotationPresent(Benchmark.class))
        .map(m -> {
          final Optional<RunResult> methodResult = results.stream()
              .filter(r ->
              m.getName().equals(r.getPrimaryResult().getLabel()))
              .findFirst();
              assertTrue(m + " didn't get any result",
              methodResult.isPresent());

          final ExpectedPerformances expectations =
          m.getAnnotation(ExpectedPerformances.class);
          final RunResult result = results.iterator().next();
          final BenchmarkResult aggregatedResult =
          result.getAggregatedResult();

          final double actualScore =
          aggregatedResult.getPrimaryResult().getScore();
          final double expectedScore = expectations.score();
          final double acceptedError = expectedScore *
          expectations.scoreTolerance();
          try { // use assert to get a common formatting for errors
            assertEquals(m.getDeclaringClass().getSimpleName() + "#" +
            m.getName(), expectedScore, actualScore, acceptedError);
            return null;
          } catch (final AssertionError ae) {
            return ae;
          }
        }).filter(Objects::nonNull).collect(toList());
    if (!errors.isEmpty()) {
      throw new AssertionError(errors.stream()
          .map(Throwable::getMessage)
          .collect(Collectors.joining("\n")));
    }
  }
}
```

现在，使用这个运行器，你可以在主构建中使用 surefire 或 failsafe 执行测试，并确保如果你的性能回归很大，构建将不会通过。

这是一个简单的实现，你可以通过多种方式丰富它，例如通过模拟每个基准测试一个子项来在 IDE 和 surefire 报告中获得更好的报告（只需在第一次遇到封装类时运行它，然后存储结果并对每个方法进行断言）。你还可以断言更多结果，例如次要结果或迭代结果（例如，没有迭代比 X 慢）。最后，你可以实现一些“活文档”功能，添加一个 @Documentation 注解，该注解将被运行器用于创建报告文件（例如，在 asciidoctor 中）。

# ContiPerf

JMH 的一个替代方案是 ContiPerf，它稍微简单一些，但更容易使用。你可以通过以下依赖关系将其添加到你的项目中：

```java
<dependency>
    <groupId>org.databene</groupId>
    <artifactId>contiperf</artifactId>
    <version>2.3.4</version>
    <scope>test</scope>
</dependency>
```

由于我们已经花费了大量时间在 JMH 上，我们不会在这本书中详细说明它。但简单来说，它基于 JUnit 4 规则。因此，它可以与其他规则结合并排序，这得益于 JUnit `RuleChain`，这使得它非常强大，所有轻量级 EE 容器都拥有基于 JUnit 的测试堆栈，例如 TomEE 或 Meecrowave 等。

ContiPerf 的一个很大的优势是它与 JUnit 模型对齐：

+   它基于标准的 JUnit `@Test` 方法标记

+   你可以重用 JUnit 标准的生命周期（`@BeforeClass`、`@Before` 等等）

+   你可以将它与其他 JUnit 功能（运行器、规则等）结合使用

从结构上来说，一个测试可以看起来是这样的：

```java
public class QuoteMicrobenchmarkTest {
    private static QuoteService service;

    @Rule
    public final ContiPerfRule rule = new ContiPerfRule();

    @BeforeClass
    public static void init() {
        service = new QuoteService();
    }

    @Test
    @Required(throughput = 7000000)
    @PerfTest(rampUp = 100, duration = 10000, threads = 10, warmUp =
    10000)
    public void test() {
        service.findByName("test").orElse(null);
    }
}
```

我们立即识别出 JUnit 结构，有一个 `@BeforeClass` 初始化测试（你可以在这里启动容器，如果需要的话，在 `@AfterClass` 中关闭它），以及一个 `@Test`，它是我们的基准/场景。与 JUnit 测试的唯一区别是 `ContiPerfRule` 和 `@Required` 以及 `@PerfTest` 注解。

`@PerfTest` 描述了测试环境——多少线程、多长时间、多少迭代次数、预热持续时间等。

另一方面，`@Required` 描述了要进行的断言（验证）。它相当于我们 JMH 集成中的 `@ExpectedPerformances`。它支持大多数常见的验证，如吞吐量、平均时间、总时间等。

# Arquillian 和 ContiPerf – 这对恶魔般的组合

在 JMH 部分，我们遇到了启动容器有时很难且并不直接的问题。由于 ContiPerf 是一个规则，它与 `Arquillian` 兼容，可以为你完成所有这些工作。

`Arquillian` 是由 JBoss（现在是 Red Hat）创建的一个项目，用于抽象化容器背后的 **服务提供者接口**（**SPI**），并将其与 JUnit 或 TestNG 集成。想法是像往常一样从你的 IDE 中运行测试，而不必担心需要容器。

在高层次上，它要求你定义要部署到容器中的内容，并使用与 JUnit（或 TestNG 的抽象类）一起使用的 `Arquillian` 运行器。多亏了扩展和增强器的机制，你可以将大多数你需要的内容注入到测试类中，例如 CDI 容器，这对于编写测试来说非常方便。以下是一个示例：

```java
@RunWith(Arquillian.class)
public class QuoteServicePerformanceTest {
    @Deployment
    public static Archive<?> quoteServiceApp() {
        return ShrinkWrap.create(WebArchive.class, "quote-manager.war")
                .addClasses(QuoteService.class,
                InMemoryTestDatabaseConfiguration.class)
                .addAsWebResource(new ClassLoaderAsset("META
                -INF/beans.xml"), "WEB-INF/classes/META-INF/beans.xml")
                .addAsWebResource(new ClassLoaderAsset("META-
                INF/persistence.xml"), "WEB-INF/classes/META-
                INF/persistence.xml");
    }

    @Inject
    private QuoteService service;

    @Before
    public void before() {
        final Quote quote = new Quote();
        quote.setName("TEST");
        quote.setValue(10.);
        service.create(quote);
    }

    @After
    public void after() {
        service.deleteByName("TEST");
    }

    @Test
    public void findByName() {
        assertTrues(service.findByName("TEST").orElse(false));
    }
}
```

此代码片段说明了 `Arquillian` 的使用及其常见特性：

+   正在使用 `Arquillian` 运行器；这是启动容器（一次）、部署应用程序（默认情况下按测试类）、执行测试（继承自 JUnit 默认行为）、在执行完所有测试后卸载应用程序，并在测试执行后关闭容器的魔法。

+   静态的 `@Deployment` 方法返回一个 `Archive<?>`，描述了要部署到容器（应用程序）中的内容。你不需要部署完整的应用程序，如果你愿意，可以按测试更改它。例如，在我们的示例中，我们没有部署我们的 `DataSourceConfiguration`，它指向 MySQL，而是部署了一个 `InMemoryDatabaseConfiguration`，我们可以假设它使用嵌入式数据库，如 derby 或 h2。

+   我们直接将 CDI 注入到测试中，我们的 `QuoteService`。

+   测试的其余部分是一个标准的 JUnit 测试，具有其生命周期（`@Before`/`@After`）和测试方法（`@Test`）。

如果你发现构建存档过于复杂，有一些项目，例如 TomEE 的 `ziplock` 模块，通过重用 *当前* 项目元数据（例如 `pom.xml` 和编译后的类）来简化它，这允许你通过单次方法调用：`Mvn.war()` 来创建存档。一些容器，包括 TomEE，允许你一次性部署每个存档。但如果你的容器不支持，你可以使用 `Arquillian` 套件扩展来实现几乎相同的结果。总体目标是将你的测试分组，一次性部署你的应用程序，并在测试执行持续时间上节省宝贵的时间。

`Arquillian` 还允许我们更进一步，从容器内部（如前一个示例所示）或使用 `@RunAsClient` 注解从客户端角度执行测试。在这种情况下，你的测试不再在容器内部执行，而是在 JUnit JVM（这可以相同或不同，取决于你的容器是否使用另一个 JVM）中执行。无论如何，将 `Arquillian` 与 ContiPerf 集成可以让你在没有太多烦恼的情况下验证性能。你只需在你想验证的方法上添加 ContiPerf 规则和注解即可：

```java
@RunWith(Arquillian.class)
public class QuoteServicePerformanceTest {
    @Deployment
    public static Archive<?> quoteServiceApp() {
        final WebArchive baseArchive =
        ShrinkWrap.create(WebArchive.class, "quote-manager.war")
              .addClasses(QuoteService.class)
              .addAsWebResource(new ClassLoaderAsset("META
              -INF/beans.xml"), "WEB-INF/classes/META-INF/beans.xml")
              .addAsWebResource(new ClassLoaderAsset("META
              -INF/persistence.xml"), "WEB-INF/classes/META
              -INF/persistence.xml");
        if (Boolean.getBoolean("test.performance." +
        QuoteService.class.getSimpleName() + ".database.mysql")) {
            baseArchive.addClasses(DataSourceConfiguration.class);
        } else {
            baseArchive.addClasses(InMemoryDatabase.class);
        }
        return baseArchive;
    }

    @Rule
    public final ContiPerfRule rule = new ContiPerfRule();

    @Inject
    private QuoteService service;

    private final Collection<Long> id = new ArrayList<Long>();

    @Before
    public void before() {
        IntStream.range(0, 1000000).forEach(i -> insertQuote("Q" + i));
        insertQuote("TEST");
    }

    @After
    public void after() {
        id.forEach(service::delete);
    }

    @Test
    @Required(max = 40)
    @PerfTest(duration = 500000, warmUp = 200000)
    public void findByName() {
        service.findByName("TEST");
    }

    private void insertQuote(final String name) {
        final Quote entity = new Quote();
        entity.setName(name);
        entity.setValue(Math.random() * 100);
        id.add(service.create(entity).getId());
    }
}
```

这几乎与之前的测试相同，但我们添加了更多数据以确保数据集大小不会在很大程度上影响初始化数据库时使用的 `@Before` 方法中的性能。为了将测试与 ContiPerf 集成，我们在方法和 ContiPerf 规则上添加了 ContiPerf 注解。你最后可以看到的一个技巧是在归档创建中设置系统属性，可以根据 JVM 配置切换数据库。它可以用来测试多个数据库或环境，并验证你是否与所有目标平台兼容。

要能够运行此示例，你需要在你的 pom 中添加这些依赖项——考虑到你将针对 GlassFish 5.0 进行测试，你需要更改容器依赖项和 `arquillian` 容器集成：

```java
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.12</version>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.jboss.arquillian.junit</groupId>
  <artifactId>arquillian-junit-container</artifactId>
  <version>1.1.13.Final</version>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.jboss.arquillian.container</groupId>
  <artifactId>arquillian-glassfish-embedded-3.1</artifactId>
  <version>1.0.1</version>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.glassfish.main.extras</groupId>
  <artifactId>glassfish-embedded-all</artifactId>
  <version>5.0</version>
  <scope>test</scope>
</dependency>
```

JUnit 是我们使用的测试框架，我们导入其 Arquillian 集成（`arquillian-junit-container`）。然后，我们导入我们的 `arquillian` 容器集成（`arquillian-glassfish-embedded-3.1`）和 Java EE 容器，因为我们使用嵌入式模式（`glassfish-embedded-all`）。如果你计划使用它，别忘了添加 ContiPerf 依赖项。

# JMeter 和构建集成

在前面的部分中，我们了解到 JMeter 可以用来构建针对你的应用程序执行的场景。它也可以通过编程方式执行——毕竟它是基于 Java 的——或者通过其 Maven 集成之一执行。

如果你使用其 Maven 插件来自 lazerycode ([`github.com/jmeter-maven-plugin/jmeter-maven-plugin`](https://github.com/jmeter-maven-plugin/jmeter-maven-plugin))，你甚至可以配置远程模式以进行真正的压力测试：

```java
<plugin>
  <groupId>com.lazerycode.jmeter</groupId>
  <artifactId>jmeter-maven-plugin</artifactId>
  <version>2.2.0</version>
  <executions>
    <execution>
      <id>jmeter-tests</id>
      <goals>
        <goal>jmeter</goal>
      </goals>
    </execution>
  </executions>
  <configuration>
    <remoteConfig>
      <startServersBeforeTests>true</startServersBeforeTests>
      <serverList>jmeter1.company.com,jmeter2.company.com</serverList>
      <stopServersAfterTests>true</stopServersAfterTests>
    </remoteConfig>
  </configuration>
</plugin>
```

此代码片段定义了 `jmeter` 插件使用 `jmeter1.company.com` 和 `jmeter2.company.com` 进行负载测试。服务器将在运行计划之前/之后初始化和销毁。

不深入插件的具体细节——你可以在相关的 GitHub wiki 上找到它们——该插件默认使用存储在项目中的 `src/test/jmeter` 中的配置。这就是你可以放置你的场景（`.jmx` 文件）的地方。

这个解决方案的挑战是提供 `jmeter[1,2].company.com` 机器。当然，你可以创建一些机器并让它们运行，尽管这样做并不是管理 AWS 机器的好方法，而且最好是在构建过程中启动它们（允许你在多个分支上同时进行并发构建，如果需要的话）。

对于这种需求，有几种解决方案，但最简单的方法可能是在一个 CI 平台上安装 AWS 客户端（或插件），并在 Maven 构建相应的命令来提供机器、设置机器主机为构建属性并将它传递给 Maven 构建之前/之后启动它。这需要您对插件配置进行变量化，但不需要复杂，因为 Maven 支持占位符和系统属性输入。尽管如此，从您的机器上运行测试可能很困难，因为您需要为自己提供将要使用的机器。因此，这减少了项目的可共享性。

不要忘记确保关闭实例的任务始终被执行，即使测试失败，否则您可能会泄露一些机器，并在月底收到账单惊喜。

最后，由于 JMeter 是一个主流解决方案，您可以轻松找到原生支持它并为您处理基础设施的平台。主要的有：

+   BlazeMeter ([`www.blazemeter.com/`](https://www.blazemeter.com/))

+   Flood.IO ([`flood.io/`](https://flood.io/))

+   Redline.13 ([`www.redline13.com/`](https://www.redline13.com/))

如果您在 CI 上没有专用机器，不妨看看他们的网站、价格，并将它们与您直接使用 AWS 可以构建的内容进行比较。这可以帮助您解决性能测试经常遇到的环境和基础设施问题。

# Gatling 和持续集成

与 JMeter 类似，Gatling 有自己的 Maven 插件，但它还提供了一些 AWS 集成伴侣插件 ([`github.com/electronicarts/gatling-aws-maven-plugin`](https://github.com/electronicarts/gatling-aws-maven-plugin))，这些插件与 Maven 本地集成。

这是官方 Gatling Maven 插件声明在您的 `pom.xml` 中的样子：

```java
<plugin>
  <groupId>io.gatling</groupId>
  <artifactId>gatling-maven-plugin</artifactId>
  <version>${gatling-plugin.version}</version>
  <executions>
    <execution>
      <phase>test</phase>
      <goals>
        <goal>execute</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

第一个插件（1）定义了如何运行 Gatling。默认配置将在 `src/test/scala` 中查找模拟。

此设置将在本地运行您的模拟。因此，您可能会迁移到非官方插件，但与 AWS 集成，以便能够控制注入器。以下是声明可能的样子：

```java
<plugin> 
  <groupId>com.ea.gatling</groupId>
  <artifactId>gatling-aws-maven-plugin</artifactId>
  <version>1.0.11</version>
  <executions>
    <execution>
      <phase>test</phase>
      <goals>
        <goal>execute</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

此插件将在 AWS 上集成 Gatling。它需要一些 AWS 配置（如您的密钥），但您通常会在 `pom.xml` 之外配置它们，以免将凭证放在公共位置——在 `settings.xml` 中的配置文件中的属性是一个好的开始。以下是您需要定义的属性：

```java
<properties>
  <ssh.private.key>${gatling.ssh.private.key}</ssh.private.key>
  <ec2.key.pair.name>loadtest-keypair</ec2.key.pair.name>
  <ec2.security.group>default</ec2.security.group>
  <ec2.instance.count>3</ec2.instance.count>

  <gatling.local.home>${project.build.directory}/gatling-charts
  -highcharts-bundle-2.2.4/bin/gatling.sh</gatling.local.home>
   <gatling.install.script>${project.basedir}/src/test/resources/install-
  gatling.sh</gatling.install.script>
  <gatling.root>gatling-charts-highcharts-bundle-2.2.4</gatling.root>
  <gatling.java.opts>-Xms1g -Xmx16g -Xss4M 
  -XX:+CMSClassUnloadingEnabled -
  XX:MaxPermSize=512M</gatling.java.opts>

  <!-- Fully qualified name of the Gatling simulation and a name
  describing the test -->
  <gatling.simulation>com.FooTest</gatling.simulation>
  <gatling.test.name>LoadTest</gatling.test.name>

  <!-- (3) -->
  <s3.upload.enabled>true</s3.upload.enabled>
  <s3.bucket>loadtest-results</s3.bucket>
  <s3.subfolder>my-loadtest</s3.subfolder>
</properties>
```

定义所有这些属性并不能阻止你通过系统属性来更改它们的值。例如，设置 `-Dec2.instance.count=9` 将允许你更改注入器的数量（从三个变为九个）。第一组属性（ec2 属性）定义了如何创建 AWS 实例以及创建多少个实例。第二组（Gatling 属性）定义了 Gatling 的位置以及如何运行它。第三组定义了要运行的模拟。最后，最后一组（s3 属性）定义了测试结果的上传位置。

此配置尚未启用，因为它尚未自给自足：

+   它依赖于尚未安装的 Gatling 发行版（`gatling.local.home`）

+   它依赖于我们尚未创建的脚本（`install-gatling.sh`）

为了能够不依赖于本地的 Gatling 安装，我们可以使用 Maven 来下载它。为此，我们只需要 Maven 的依赖插件：

```java
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-dependency-plugin</artifactId>
  <version>3.0.2</version>
  <executions>
    <execution>
      <id>download-gatling-distribution</id>
      <phase>generate-test-resources</phase>
      <goals>
        <goal>unpack</goal>
      </goals>
      <configuration>
        <artifactItems>
          <artifactItem>
            <groupId>io.gatling.highcharts</groupId>
            <artifactId>gatling-charts-highcharts-bundle</artifactId>
            <version>2.2.4</version>
            <classifier>bundle</classifier>
            <type>zip</type>
            <overWrite>false</overWrite>
            <outputDirectory>${project.build.directory}/gatling</outputDirectory>
            <destFileName>gatling-charts-highcharts-bundle
            -2.2.4.jar</destFileName>
          </artifactItem>
        </artifactItems>
        <outputDirectory>${project.build.directory}/wars</outputDirectory>
        <overWriteReleases>false</overWriteReleases>
        <overWriteSnapshots>true</overWriteSnapshots>
      </configuration>
    </execution>
  </executions>
</plugin>
```

此配置将 Gatling 发行版提取到`target/gatling/gatling-charts-highcharts-bundle-2.2.4`文件夹中，并在插件运行时使用它。

对于脚本，你可以使用这个，它是为 Fedora 准备的。然而，如果你在 EC2 上选择另一个镜像，它很容易适应任何发行版：

```java
#!/bin/sh
# Increase the maximum number of open files
sudo ulimit -n 65536
echo "* soft nofile 65535" | sudo tee --append /etc/security/limits.conf
echo "* hard nofile 65535" | sudo tee --append /etc/security/limits.conf

sudo yum install --quiet --assumeyes java-1.8.0-openjdk-devel.x86_64 htop screen

# Install Gatling
GATLING_VERSION=2.2.4
URL=https://repo.maven.apache.org/maven2/io/gatling/highcharts/gatling-charts-highcharts-bundle/${GATLING_VERSION}/gatling-charts-highcharts-bundle-${GATLING_VERSION}-bundle.zip
GATLING_ARCHIVE=gatling-charts-highcharts-bundle-${GATLING_VERSION}-bundle.zip

wget --quiet ${URL} -O ${GATLING_ARCHIVE}
unzip -q -o ${GATLING_ARCHIVE}

# Remove example code to reduce Scala compilation time at the beginning of load test
rm -rf gatling-charts-highcharts-bundle-${GATLING_VERSION}/user-files/simulations/computerdatabase/
```

此脚本执行三个主要任务：

+   增加 ulimit 以确保注入器可以使用足够的文件处理器，而不会受到操作系统配置的限制

+   安装 Java

+   从 Maven 中心下载 Gatling，提取存档（就像我们使用之前的 Maven 插件所做的那样），但是在不需要使用 Maven 的注入器机器上，最后清理提取的存档（移除示例模拟）

如果你需要任何依赖项，你需要创建一个 shade（并使用默认的`jar-with-dependencies`作为分类器）。你可以使用`maven-assembly-plugin`和`single`目标来实现这一点。

# 部署你的（基准）应用程序

在前两个部分中，我们学习了如何处理注入器，但如果你可能的话，还需要将应用程序部署到测试专用的实例上进行测试。这并不意味着你需要忘记上一章中我们讨论的内容，例如确保机器在生产中使用，以及其他可能影响的服务正在运行。相反，你必须确保机器（们）做你所期望的事情，而不是一些意外的、会影响数据/测试的事情。

在这里，再次使用云服务来部署你的应用程序可能是最简单的解决方案。最简单的解决方案可能依赖于某些云 CLI（例如 AWS CLI 或`aws`命令），或者你可以使用云提供商的客户端 API 或 SDK 编写的`main(String[])`。

根据您是否自己编写部署代码（或不是），与 Maven（或 Gradle）构建的集成将更容易或更难。作为您项目的一部分，`exec-maven-plugin` 可以让您在 Maven 生命周期中精确地集成它。大多数情况下，这将在性能测试之前完成，但在测试编译之后，甚至在打包应用程序之后（如果您将性能测试保持在同一模块中，这是可行的）。

如果您不自己编写部署代码，您将需要定义您的性能构建阶段：

1.  编译/打包项目和测试。

1.  部署应用程序（并且如果需要，别忘了重置环境，包括清理数据库或 JMS 队列）。

1.  开始性能测试。

1.  取消部署应用程序/关闭创建的服务器（如果相关）。

使用 Maven 或 Gradle，很容易跳过一些任务，无论是通过标志还是配置文件，因此您最终可能会得到这样的命令：

```java
mvn clean install # (1)
mvn exec:java@deploy-application # (2)
mvn test -Pperf-tests # (3)
mvn exec:java@undeploy-application #(4)
```

第一个命令 `(1)` 将构建完整的项目，但绕过性能测试，因为我们默认没有激活 `perf-tests` 配置文件。第二个命令将使用基于 AWS SDK 的自定义实现将应用程序部署到目标环境，例如，可能从头开始创建它。别忘了记录您所做的一切，即使您正在等待某事，否则您或其他人可能会认为进程挂起了。然后，我们运行性能测试 `(3)`，最后我们使用与 `(2)` 对称的命令取消部署应用程序。

使用此类解决方案时，您需要确保 `(4)` 在 `(2)` 执行后立即执行。通常，您将强制执行它始终执行，并在预期环境不存在时处理快速退出条件。

要编排这些步骤，您可以查看 Jenkins 管道功能：它将为您提供很多灵活性，以直接方式实现此类逻辑。

进一步的内容超出了本书的范围，但为了给您一些提示，部署可以依赖于基于 Docker 的工具，这使得在云平台上部署变得非常容易。然而，别忘了 Docker 不是一个配置工具。如果您的 *配方*（创建实例的步骤）不简单（从仓库安装软件、复制/下载您的应用程序并启动它），那么您可以考虑投资于 chef 或 puppet 等配置工具，以获得更大的灵活性、更强的功能和避免黑客攻击。

# 持续集成平台和性能

Jenkins 是最常用的持续集成平台。虽然有一些替代品，如 Travis，但 Jenkins 的生态系统以及扩展的便捷性使其在 Java 和企业应用中成为明显的领导者。

我们在性能构建/测试执行平台上想要解决的首要问题是构建的隔离，显然目标是确保获得的数字不受其他构建的影响。

要做到这一点，Jenkins 提供了几个插件：

+   Amazon EC2 容器服务插件 ([`wiki.jenkins.io/display/JENKINS/Amazon+EC2+Container+Service+Plugin`](https://wiki.jenkins.io/display/JENKINS/Amazon+EC2+Container+Service+Plugin)): 允许您在基于 Docker 镜像创建的专用机器上运行构建（测试）。

+   限制并发构建插件 ([`github.com/jenkinsci/throttle-concurrent-builds-plugin/blob/master/README.md`](https://github.com/jenkinsci/throttle-concurrent-builds-plugin/blob/master/README.md)): 允许您控制每个项目可以执行多少个并发构建。具体来说，为了性能，我们希望确保每个项目只有一个。

在配置方面，您需要确保性能测试以准确的配置执行：

+   通常使用 Jenkins 调度，但可能不是每次提交或拉取请求时都这样做。根据项目的关键性、稳定性和性能测试的持续时间，可能是每天一次或每周一次。

+   之前使用的插件或等效插件已正确配置。

+   如果构建失败，它会通知正确的渠道（邮件、Slack、IRC 等）。

确保存储运行历史记录以便进行比较也很重要，尤其是如果您不与每个提交一起运行性能测试，这将给出确切的提交，引入回归。为此，您可以使用另一个 Jenkins 插件，该插件正是为了存储常见性能工具的历史记录：性能插件 ([`wiki.jenkins.io/display/JENKINS/Performance+Plugin`](https://wiki.jenkins.io/display/JENKINS/Performance+Plugin))。此插件支持 Gatling 和 JMeter，以及一些其他工具。这是一个很好的插件，允许您直接从 Jenkins 可视化报告，这在调查某些更改时非常方便。更重要的是，它与 Jenkins 管道脚本兼容。

# 摘要

在本章中，我们介绍了一些确保您的应用程序性能处于控制之下并限制在进入基准测试阶段或更糟的生产阶段时遇到意外不良惊喜的风险的常见方法。每周（甚至每天）为波动（临时）基准测试设置简单的测试或完整的环境是非常可行的步骤，一旦支付了入门成本，就可以使产品以更高的质量水平交付。

在理解了如何使用 Java EE 仪器化您的应用程序以便您能专注于业务后，如何监控和仪器化您的应用程序以优化您的应用程序，以及如何通过一些调整或缓存来提升您的应用程序后，我们现在知道如何自动控制性能回归，以便能够尽快修复它们。

因此，您现在已经涵盖了与性能相关的所有产品或库创建部分，并且您能够交付高性能的软件。您做到了！
