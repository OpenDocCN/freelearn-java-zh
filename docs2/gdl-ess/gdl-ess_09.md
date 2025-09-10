# 第九章：多语言项目

我们生活在一个一个语言不足以应对的时代。开发者们被期望成为多语言程序员，并为工作选择合适的工具。虽然这始终是一个主观的决定，但我们尝试根据各种参数来选择语言和生态系统，例如执行速度、开发者生产力、可用的库和资源、团队对语言的舒适度等等。

当我们已经在处理不同语言时承受着认知负荷，Gradle 便成了我们的好朋友，因为我们不需要改变我们的构建工具，即使我们在用其他语言构建项目。我们甚至可以在同一个项目中使用多种语言，由 Gradle 来协调整个项目的构建。除了 JVM 基础语言之外，Gradle 还支持 C、C++、Objective C 等其他语言，以生成原生应用程序。Gradle 也是 Android 平台的官方构建工具。支持的语言列表正在不断增加。除了官方插件之外，还有许多社区支持的编程语言插件。

尽管在整个书中我们主要关注 Java 语言，但我们完全可以使用 Groovy 或 Scala 来编写示例。`java` 插件（以及由 `java` 插件应用于项目的 `java-base` 插件）为 JVM 基础项目提供了基本功能。特定语言的插件，如 `scala` 和 `groovy`，以一致的方式扩展了 `java` 插件以支持常见的编程习惯。因此，一旦我们使用了 `java` 插件，我们就已经熟悉了 `sourceSet` 是什么，`configuration` 如何工作，如何添加库依赖等等，这些知识在我们使用这些语言插件时非常有用。在本章中，我们将看到如何通过添加 Groovy 或 Scala 来轻松地为 Java 项目增添更多色彩。

# 多语言应用程序

对于代码示例，在本章中，让我们构建一个简单的“每日名言”服务，该服务根据年份的某一天返回一个名言。由于我们存储的名言可能较少，该服务应以循环方式重复名言。再次强调，我们将尽量保持简单，以便更多地关注构建方面而不是应用逻辑。我们将创建两个独立的 Gradle 项目来实现完全相同的功能，一次使用 Groovy，然后使用 Scala。

在深入探讨特定语言细节之前，让我们先定义 `QotdService` 接口，它仅声明了一个方法，即 `getQuote`。合同规定，只要我们传递相同的日期，就应该返回相同的名言：

```java
package com.packtpub.ge.qotd;

import java.util.Date;

interface QotdService {
  String getQuote(Date day);
}
```

实现`getQuote`的逻辑可以使用`Date`对象以任何方式，例如使用包括时间在内的整个日期来确定引语。然而，为了简单起见，我们将在我们的实现中仅使用`Date`对象的日期部分。此外，因为我们想让我们的接口对未来的实现开放，所以我们让`getQuote`接受一个`Date`对象作为参数。

此接口是一个 Java 文件，我们将在两个项目中都有。这只是为了演示在一个项目中集成 Java 和 Groovy/Scala 源。

# 构建 Groovy 项目

让我们先在 Groovy 中实现`QotdService`接口。此外，我们还将编写一些单元测试以确保功能按预期工作。为了启动项目，让我们创建以下目录结构：

```java
qotd-groovy
├── build.gradle
└── src
 ├── main
 │   ├── groovy
 │   │   └── com
 │   │       └── packtpub
 │   │           └── ge
 │   │               └── qotd
 │   │                   └── GroovyQotdService.groovy
 │   └── java
 │       └── com
 │           └── packtpub
 │               └── ge
 │                   └── qotd
 │                       └── QotdService.java
 └── test
 └── groovy
 └── com
 └── packtpub
 └── ge
 └── qotd
 └── GroovyQotdServiceTest.groovy

```

`src/main/java`目录是 Java 源文件的默认目录。同样，`src/main/groovy`默认用于编译 Groovy 源文件。再次强调，这只是一种约定，源目录的路径和名称可以通过`sourceSets`轻松配置。

让我们先为我们的 Groovy 项目编写构建脚本。在项目根目录中创建一个`build.gradle`文件，内容如下：

```java
apply plugin: 'groovy'

repositories {
  mavenCentral()
}

dependencies {
  compile 'org.codehaus.groovy:groovy-all:2.4.5'
  testCompile 'junit:junit:4.11'
}
```

构建 Groovy 项目就像构建 Java 项目一样简单。我们不是应用`java`插件，而是应用`groovy`插件，它会自动为我们应用`java`插件。除了应用插件之外，我们还需要将 Groovy 添加为库依赖项，以便它在编译时可用，并在运行时也可用。我们还在`testCompile`配置中添加了`junit`，以便它可用于单元测试。我们声明 Maven central 作为要使用的仓库，但这也可能被更改为任何可以为我们项目依赖项服务的有效仓库配置。

### 注意

Gradle 构建脚本是一个 Groovy DSL，Gradle 的部分是用 Groovy 编写的。然而，就像 Gradle 在运行时依赖的任何其他库一样，Groovy 并不是隐式地对我们正在构建的项目可用。因此，我们必须显式地将 Groovy 声明为项目依赖项，具体取决于我们是否在生产或测试源中使用 Groovy。

Groovy 插件负责编译项目中的 Java 源文件。让我们用 Groovy 实现`QotdService`接口：

```java
package com.packtpub.ge.qotd

class GroovyQotdService implements QotdService {
  List quotes

  GroovyQotdService(List quotes) {
    this.quotes = quotes
  }

  @Override
  String getQuote(Date day) {
    quotes[day[Calendar.DAY_OF_YEAR] % quotes.size()]
  }
}
```

服务的实现接受一个包含引语的构造函数中的列表。`getQuote`方法通过列表中的索引获取引语。为了确保计算出的索引始终保持在引语大小的范围内，我们获取了年份和列表大小的余数。

为了测试服务，让我们用 Groovy 编写非常基本的 JUnit 测试用例：

```java
package com.packtpub.ge.qotd

import org.junit.Before
import org.junit.Test

import static org.junit.Assert.assertEquals
import static org.junit.Assert.assertNotSame

public class GroovyQotdServiceTest {

  QotdService service
  Date today, tomorrow, dayAfterTomorrow

  def quotes = [
    "Be the change you wish to see in the world" +
      " - Mahatma Gandhi",
    "A person who never made a mistake never tried anything new" +
      " - Albert Einstein"
  ]

  @Before
  public void setup() {
    service = new GroovyQotdService(quotes)
    today = new Date()
    tomorrow = today + 1
    dayAfterTomorrow = tomorrow + 1
  }

  @Test
  void "return same quote for same date"() {
    assertEquals(service.getQuote(today), service.getQuote(today))
  }

  @Test
  void "return different quote for different dates"() {
    assertNotSame(service.getQuote(today),
      service.getQuote(tomorrow))
  }

  @Test
  void "repeat quotes"() {
    assertEquals(service.getQuote(today),
      service.getQuote(dayAfterTomorrow))
  }
}
```

我们在设置中准备测试数据，每个测试用例都确保引语服务的契约得到维护。由于引语列表中只有两个引语，它们应该每隔一天重复一次。

我们可以使用以下代码从命令行运行测试：

```java
$ gradle test

```

# 构建 Scala 项目

在上一节之后，本节的大部分内容从应用程序构建的角度来看应该是可预测的。所以让我们快速浏览一下要点。目录结构如下：

```java
qotd-scala
├── build.gradle
└── src
 ├── main
 │   ├── java
 │   │   └── com/packtpub/ge/qotd
 │   │                       └── QotdService.java
 │   └── scala
 │       └── com/packtpub/ge/qotd
 │                           └── ScalaQotdService.scala
 └── test
 └── scala
 └── com/packtpub/ge/qotd
 └── ScalaQotdServiceTest.scala

```

所有 Scala 源文件都从 `src/main/scala` 和 `src/test/scala` 读取，除非使用 `sourceSets` 进行配置。这次，我们只需要应用 `scala` 插件，就像 `groovy` 插件一样，它隐式地将 `java` 插件应用到我们的项目中。让我们为这个项目编写 `build.gradle` 文件：

```java
apply plugin: 'scala'

repositories {
  mavenCentral()
}

dependencies {
  compile 'org.scala-lang:scala-library:2.11.7'
  testCompile 'org.specs2:specs2-junit_2.11:2.4.15',
    'junit:junit:4.11'
}
```

在这里，我们必须提供 `scala-library` 作为依赖项。我们还为测试配置添加了 `specs2` 作为依赖项。我们正在使用 JUnit 运行器进行测试。

### 注意

`specs2` 是一个流行的 Scala 测试库，它支持单元测试和验收测试，以及 BDD/TDD 风格的测试编写。更多信息可以在 [`etorreborre.github.io/specs2/`](http://etorreborre.github.io/specs2/) 找到。

接下来，我们将实现服务的 Scala 版本，可以按照以下方式实现：

```java
package com.packtpub.ge.qotd

import java.util.{Calendar, Date}

class ScalaQotdService(quotes: Seq[String]) extends QotdService {

  def getQuote(day: Date) = {
    val calendar = Calendar.getInstance()
    calendar.setTime(day)

    quotes(calendar.get(Calendar.DAY_OF_YEAR) % quotes.size)
  }
}
```

实现并不是非常符合 Scala 的习惯用法，但这本书的范围之外。该类在构造函数中接受 `Seq` 引用，并以类似 Groovy 对应方式实现 `getQuote` 方法。

现在服务已经实现，让我们通过编写单元测试来验证它是否遵循 `QotdService` 的语义。为了简洁，我们将只涵盖重要的测试用例：

```java
package com.packtpub.ge.qotd

import java.util.{Calendar, Date}

import org.junit.runner.RunWith
import org.specs2.mutable._
import org.specs2.runner.JUnitRunner

@RunWith(classOf[JUnitRunner])
class ScalaQotdServiceTest extends SpecificationWithJUnit {

  def service = new ScalaQotdService(Seq(
    "Be the change you wish to see in the world" +
      " - Mahatma Gandhi",
    "A person who never made a mistake never tried anything new" +
      " - Albert Einstein"
  ))

  val today = new Date()
  val tomorrow = incrementDay(today)
  val dayAfterTomorrow = incrementDay(tomorrow)

  "Quote service" should {
    "return same quote for same day in multiple invocations" in {
      service.getQuote(today) must be(service.getQuote(today))
    }

    "return different quote for different days" in {
      service.getQuote(today) must not be (
        service.getQuote(tomorrow))
    }

    "repeat quote if total quotes are less than days in year" in {
      service.getQuote(today) must be(
        service.getQuote(dayAfterTomorrow))
    }
  }

  def incrementDay(date: Date) = {
    val cal = Calendar.getInstance()
    cal.setTime(date)
    cal.add(Calendar.DATE, 1)
    cal.getTime
  }
}
```

运行测试用例的任务与 Groovy 对应的任务相同。我们可以使用以下代码来运行测试：

```java
$ gradle test

```

# 联合编译

在本章前面的示例中，我们分别在 Java 中声明了一个接口，并在 Groovy 和 Scala 中分别实现了它。这是可能的，因为由 `java` 插件编译的类对 Groovy 和 Scala 类是可用的。

如果我们想让 Java 类在编译时能够访问 Groovy 或 Scala 类，那么我们必须使用相应插件支持的 **联合编译** 来编译 Java 源文件。`groovy` 和 `scala` 插件都支持联合编译，并且可以编译 Java 源代码。

在 Java 类中引用 Groovy 类的最简单方法是，将相应的 Java 源文件移动到 `src/main/groovy`（或为 `sourceSets` 配置的任何 Groovy `srcDirs`），Groovy 编译器在编译时使 Groovy 类对 Java 类可用。Scala 联合编译也是如此。我们可以将需要 Scala 类进行编译的 Java 文件放在 Scala 的任何 `srcDirs` 中（默认为 `src/main/scala`）。

# 参考

本章讨论的语言插件的详细官方文档可以在以下 URL 中找到：

+   **Java 插件**：[`docs.gradle.org/current/userguide/java_plugin.html`](https://docs.gradle.org/current/userguide/java_plugin.html)

+   **Groovy 插件**：[`docs.gradle.org/current/userguide/groovy_plugin.html`](https://docs.gradle.org/current/userguide/groovy_plugin.html)

+   **Scala 插件**: [`docs.gradle.org/current/userguide/scala_plugin.html`](https://docs.gradle.org/current/userguide/scala_plugin.html)

可以在以下网址找到各种语言和 Gradle 一起提供的其他插件的官方文档链接：

[`docs.gradle.org/current/userguide/standard_plugins.html`](https://docs.gradle.org/current/userguide/standard_plugins.html)

# 摘要

我们选取了一个简单的示例问题，并在 Groovy 和 Scala 中实现了解决方案，以展示 Gradle 如何使多语言项目开发变得简单。我们试图专注于 Gradle 带来的共性和一致性，而不是深入到语言和插件特定的细节和差异。
