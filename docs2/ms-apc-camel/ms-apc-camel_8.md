# 第八章 测试

当我们处理集成项目时，测试对于确保你的逻辑按预期工作至关重要。这意味着测试不同的路由逻辑，并管理在路由过程中可能发生的错误。

此外，一个集成项目意味着我们使用不同团队或第三方提供的服务或端点。而不是等待团队提供的服务和端点，我们可以通过模拟依赖服务来开始实现我们的项目。

我们可以区分两种测试类型：

+   单元测试主要关注测试你的路由逻辑。基本上，它测试你的路由行为。

+   集成测试更多地关注于在你的容器中安装和部署你的路由。这些测试依赖于你用来运行 Camel 路由的运行时容器。

Apache Camel 提供了一个易于实现单元测试的工具——称为 Camel 测试套件。

本章将介绍：

+   单元测试方法和如何使用测试套件提供的不同模块。

+   如何在 Apache Karaf 和 OSGi 的特殊情况下启动集成测试

# 使用 Camel 测试套件的单元测试方法

实现单元测试基本上意味着你启动你的路由——你在测试中加载 `CamelContext` 和路由，它就准备好执行了。

你现在定义你想要模拟的端点，如下所示：

1.  在模拟端点上，你定义断言。

1.  在路由的某些点上创建和 *注入* 交换。

1.  检查断言是否被验证。

Camel 根据你用来编写路由的 DSL 提供不同的测试套件：

+   `camel-test` 是如果你使用 Java DSL 可以使用的核心和抽象测试套件。

+   `camel-test-spring` 扩展 `camel-test`，提供对 Spring DSL 的支持。

+   `camel-test-blueprint` 也在 `camel-test` 的基础上扩展，并提供对 Blueprint DSL 的支持。此外，它还提供了一个类似于 OSGi 的服务支持，利用 iPOJO。

所有 Camel 测试套件都提供：

+   JUnit 扩展：JUnit 是最常用的 Java 单元测试框架，并且是免费可用的。Camel 直接提供 JUnit 扩展，这意味着你的单元测试类将扩展 Camel JUnit 扩展，你将能够使用 JUnit 注解（例如 `@Test`）。

+   模拟组件：模拟组件直接由 `camel-core` 提供。模拟组件提供了一个强大的声明式测试机制，并且可以用于 *之上* 实际组件。在测试开始之前，可以在任何模拟端点上创建声明式期望。

+   ProducerTemplate：`ProducerTemplate` 由 Camel 测试基类（或 `CamelContext`）提供。这是一个方便的特性，允许你轻松创建交换，并设置消息，你可以在你选择的路线端点发送这些消息。

## ProducerTemplate

`ProducerTemplate` 是一个模板，它提供了一种简单的方法在 Camel 中创建消息。它允许你将消息实例发送到端点。它支持各种通信风格——`InOnly`、`InOut`、`Sync`、`Async` 和 `Callback`。你可以从 `CamelContext` 获取 `ProducerTemplate`：

```java
ProducerTemplate producerTemplate = camelContext.createProducerTemplate();

```

在单元测试中，一旦你的测试类扩展到 Camel 测试基类之一，你就有 `producerTemplate` 可以使用了。

例如，生产模板可以创建一个消息，设置 `in` 消息的主体，并将其发送到 `direct:input` 端点：

```java
producerTemplate.sendBody("direct:input", "Hello World", );
```

除了 `in` 消息的主体之外，还可以设置一个头部：

```java
producerTemplate.sendBodyAndHeader("direct:input", "Hello World", "myHeader", "headerValue");
```

`sendBody()` 方法也接受一个 `MessageExchangePattern` 参数（如果你想要模拟 `InOnly` 或 `InOut` 交换）。

当使用 `InOut` 时，你可能在交换执行后想要获取 `out` 消息。

在这种情况下，你必须使用 `producerTemplate` 上的 `requestBody()` 方法而不是 `sendBody()` 方法：

```java
String out = producerTemplate.requestBody("jetty:http://0.0.0.0:8888/service", "request", String.class);
```

## JUnit 扩展

Camel 直接提供了你需要在测试中扩展的类。

### CamelTestSupport

`CamelTestSupport` 是如果你使用 Java DSL 那样你必须扩展的类。

你必须重写 `createRouteBuilder()` 方法。这是你通过调用在路由类中定义的 `createRouteBuilder()` 方法来启动路由的地方。

你还必须重写 `isMockEndpoints()` 或 `isMockEndpointsAndSkip()` 方法。此方法返回一个正则表达式——所有匹配此 `regex` 的端点 URI 都将由模拟组件模拟。`isMockEndpoints()` 和 `isMockEndpointsAndSkip()` 方法相同，但跳过的那个不会将交换发送到实际端点。

现在，你准备好使用 `@Test` 注解创建方法了。这些方法是实际的测试。

这里有一个完整的示例：

```java
public class MyTest extends CamelTestSupport {

  @Override
  protected RouteBuilder createRouteBuilder() throws Exception {
     MyRoute route = new MyRoute();
     return route.createRouteBuilder();
  }

  @Override
  public String isMockEndpointsAndSkip() {
    return "*";
  }

  @Test
  public void myTest() throws Exception {
     ...
  }

}
```

### CamelSpringTestSupport

`CamelSpringTestSupport` 是如果你使用 Spring DSL 那样你的测试类必须扩展的类。

这与 `CamelTestSupport` 类完全相同。唯一的区别是，你不必重写 `createRouteBuilder()` 方法，而是必须重写 `createApplicationContext()` 方法。`createApplicationContext()` 方法实际上直接加载包含你的路由定义的 Spring XML 文件：

```java
public class MySpringTest extends CamelSpringTestSupport {

  @Override
  protected AbstractXmlApplicationContext createApplicationContext() throws Exception {
     return new ClassPathXmlApplicationContext("myroute.xml");
  }

  @Override
  public String isMockEndpointsAndSkip() {
    return "*";
  }

  @Test
  public void myTest() throws Exception {
     ...
  }

}
```

### CamelBlueprintTestSupport

`CamelBlueprintTestSupport` 是如果你使用 Blueprint DSL 那样必须扩展的类。

这与 `CamelSpringTestSupport` 类非常相似，但与 `createApplicationContext()` 方法不同，你必须重写 `getBlueprintDescriptor()` 方法：

```java
public class MyBlueprintTest extends CamelBlueprintTestSupport {

  @Override
  protected String getBlueprintDescriptor() throws Exception {
     return "OSGI-INF/blueprint/route.xml";
  }

  @Override
  public String isMockEndpointsAndSkip() {
    return "*";
  }

  @Test
  public void myTest() throws Exception {
     ...
  }

}
```

你还可以重写 `addServicesOnStartup()` 方法，如果你想要在路由蓝图 XML 中 *模拟* 一些 OSGi 服务。

## 模拟组件

模拟组件由 `camel-core` 提供。这意味着你可以使用 `mock:name` URI 显式创建模拟端点。然而，真正有意义使用模拟组件的地方是在单元测试中——它是那里的基石。

就像碰撞测试假人一样，模拟组件用于模拟真实组件并伪造实际端点。

没有模拟组件，你就必须使用真实的组件和端点，这在测试中并不总是可能的。此外，在测试时，你需要应用断言来查看结果是否如预期——我们可以轻松地使用模拟组件来做这件事。

模拟组件是对以下情况的回应：

+   真实组件或端点尚不存在。例如，你想调用由另一个团队开发的 Web 服务。不幸的是，该 Web 服务尚未准备好。在这种情况下，你可以使用模拟组件伪造 Web 服务。

+   真实组件不易启动或需要时间来启动。

+   真实组件难以设置。一些组件难以设置，或者需要其他难以设置的程序，例如，当你使用`camel-hbase`组件在你的路由中时。该组件使用一个正在运行的 HBase 实例，这意味着一个正在运行的 ZooKeeper 和一个正在运行的 Hadoop HDFS 集群。在实际单元测试中真正使用一个 HBase 实例并没有太多意义（它可以在集成测试中使用）。在这种情况下，我们将模拟 HBase 端点。

+   真实组件返回非确定性值。例如，你的路由调用一个永远不会为相同请求返回相同响应的 Web 服务（例如，包含时间戳）。在非确定性值上定义断言是困难的。在这种情况下，我们将模拟 Web 服务以始终返回一个样本响应。

+   你必须模拟错误。如前一章所示，模拟错误以测试错误处理器等非常重要。当我们模拟端点时，通过在模拟端点中抛出异常，可以模拟错误。

### 使用 MockComponent

当你在你的`test`类中重写`isMockEndpoints()`或`isMockEndpointsAndSkip()`方法时，Camel 会自动将实际端点替换为模拟端点，并在端点 URI 前加上模拟前缀。

例如，在你的路由中，你有文件`:/tmp/in`端点。`isMockEndpointsAndSkip()`方法返回`*`，意味着所有端点都将被模拟。Camel Test 创建`mock:file:/tmp/in`模拟端点。

你可以在`test()`方法中使用`getMockEndpoint()`方法检索模拟端点：

```java
MockEndpoint mockEndpoint = getMockEndpoint("mock:file:/tmp/in");
```

你可以在模拟端点上定义断言：

+   `expectedMessageCount(int)`定义了端点期望接收的消息数量。这个计数在`CamelContext`创建时重置和初始化。

+   `expectedMinimumMessageCount(int)`定义了端点期望接收的最小消息数量。

+   `expectedBodiesReceived(...)`定义了期望按此顺序接收的`in`消息体。

+   `expectedHeaderRecevied(...)`定义了期望接收的`in`消息头。

+   `expectsAscending(Expression)`定义了接收消息的顺序期望。顺序由给定的表达式定义。

+   `expectsDescending(Expression)`类似于`expectsAscending()`，但顺序相反。

+   `expectsNoDuplicate(Expression)`检查是否存在重复的消息。重复模式由表达式表示。

一旦你在模拟端点的预期上定义了期望，你就可以调用`assertIsSatisfied()`方法来验证这些期望是否得到满足：

```java
MockEndpoint mockEndpoint = getMockEndpoint("mock:file:/tmp/in");
mockEndpoint.expectedMessageCount(2);
// send message with producerTemplate, see later
...
mockEndpoint.assertIsSatisfied();
```

默认情况下，`assertIsSatisfied()`方法执行路由，并在关闭路由前等待 10 秒。可以通过`setResultWaitTime(ms)`方法更改超时时间。当断言得到满足时，Camel 停止等待并继续到`assertIsSatisfied()`方法的调用。如果一个消息在`assertIsSatisfied()`语句之后到达端点，它将不会被考虑。例如，如果你想验证端点没有收到任何消息（使用`expectedMessageCount(0)`），由于断言在开始时已经得到满足，Camel 不会等待。因此，你必须显式地使用`setAssertPeriod()`方法等待断言等待时间：

```java
MockEndpoint mockEndpoint = getMockEndpoint("mock:file:/tmp/in");
mockEndpoint.setAssertPeriod(10000);
mockEndpoint.expectedMessageCount(0);
// send message with producerTemplate
...
mockEndpoint.assertIsSatisfied();
```

还可以在特定消息上定义断言。`message()`方法允许你访问模拟端点接收到的特定消息：

```java
MockEndpoint mockEndpoint = getMockEndpoint("mock:file:/tmp/in");
mockEndpoint.message(0).header("CamelFileName").isEqualTo("myfile");
// send message with producerTemplate
...
mockEndpoint.assertIsSatisfied();
```

模拟端点将接收到的消息存储在内存中。除了消息本身，它还存储了消息的到达时间。

这意味着你可以在消息上定义时间断言：

```java
mock.message(0).arrives().noLaterThan(2).seconds().beforeNext();
mock.message(1).arrives().noLaterThan(2).seconds().afterPrevious();
mock.message(2).arrives().between(1, 4).seconds().afterPrevious();
```

你也可以在模拟端点上模拟错误。这允许你在发生错误时测试路由（尤其是错误处理器）的行为。

如前一章所述，错误实际上是由端点引发的异常。

在模拟端点，你可以使用`whenAnyExchangeReceived()`方法来调用处理器。如果处理器抛出异常，我们将模拟一个错误：

```java
MockEndpoint mockEndpoint = getMockEndpoint("mock:file:/tmp/in");
mockEndpoint.whenAnyExchangeReceived(new Processor() {
  public void process(Exchange exchange) throws Exception {
    throw new IOException("Full filesystem error simulation for instance");
  }
});
// send message with producerTemplate
...
mockEndpoint.assertIsSatisfied();
```

# 一个完整的示例

我们使用 Blueprint DSL 定义了以下路由的 bundle：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

      <camelContext >
          <route id="test">
              <from uri="direct:input"/>
              <onException>
                  <exception>java.lang.Exception</exception>
                  <redeliveryPolicy maximumRedeliveries="2"/>
                  <handled>
                      <constant>true</constant>
                  </handled>
                  <to uri="direct:error"/>
              </onException>
              <choice>
                  <when>
                      <xpath>//country='France'</xpath>
                      <to uri="direct:france"/>
                  </when>
                  <when>
                      <xpath>//country='USA'</xpath>
                      <to uri="direct:usa"/>
                  </when>
                  <otherwise>
                      <to uri="direct:other"/>
                  </otherwise>
              </choice>
          </route>
      </camelContext>

</blueprint>
```

如同往常，这个路由 Blueprint XML 位于我们项目的`src/main/resources/OSGI-INF/blueprint/route.xml`中。

路由逻辑相当简单：

1.  我们在`direct:input`端点接收 XML 消息

1.  我们使用以下逻辑实现了一个基于内容的路由 EIP：

    +   如果消息包含一个具有`France`值的`country`元素（使用`//country=France xpath`表达式），我们将消息发送到`direct:france`端点。

    +   如果消息包含一个具有`USA`值的`country`元素（使用`//country=USA xpath`表达式），我们将消息发送到`direct:usa`端点。

    +   否则，消息将被发送到`direct:other`端点。

    +   我们还配置了路由的`DefaultErrorHandler`。对于所有异常，我们尝试：

        重发消息两次

1.  我们设置了异常处理的意义，这样我们不会将异常*发送*回路由外部

1.  我们将*故障*消息转发到`direct:error`端点

项目的`pom.xml`文件定义了测试所需的依赖，特别是`camel-test-blueprint`组件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

      <modelVersion>4.0.0</modelVersion>

      <groupId>com.packt.camel</groupId>
      <artifactId>chapter8a</artifactId>
      <version>1.0-SNAPSHOT</version>
      <packaging>bundle</packaging>

      <dependencies>
          <dependency>
              <groupId>org.apache.camel</groupId>
              <artifactId>camel-test-blueprint</artifactId>
              <version>2.12.4</version>
              <scope>test</scope>
          </dependency>
          <dependency>
              <groupId>org.slf4j</groupId>
              <artifactId>slf4j-simple</artifactId>
              <version>1.7.5</version>
              <scope>test</scope>
          </dependency>
      </dependencies>

      <build>
          <plugins>
              <plugin>
                  <groupId>org.apache.felix</groupId>
                  <artifactId>maven-bundle-plugin</artifactId>
                  <version>2.3.7</version>
                  <extensions>true</extensions>
                  <configuration>
                      <instructions>
                          <Import-Package>*</Import-Package>
                      </instructions>
                  </configuration>
              </plugin>
          </plugins>
      </build>

</project>
```

现在是时候实现我们的单元测试了。我们在`src/test/java/com/packt/camel/test folder`目录下创建一个名为`RouteTest.java`的类：

```java
package com.packt.camel.test;

import org.apache.camel.Exchange;
import org.apache.camel.Processor;
import org.apache.camel.component.mock.MockEndpoint;
import org.apache.camel.test.blueprint.CamelBlueprintTestSupport;
import org.junit.Test;

import java.io.IOException;

public class RouteTest extends CamelBlueprintTestSupport {

      @Override
      protected String getBlueprintDescriptor() {
          return "OSGI-INF/blueprint/route.xml";
      }

      @Override
      public String isMockEndpointsAndSkip() {
          return "((direct:error)|(direct:france)|(direct:usa)|(direct:other))";
      }

      @Test
      public void testRoutingFrance() throws Exception {
          String message = "<company><country>France</country></company>";

    //define the expectations on the direct:france mocked endpoint
          MockEndpoint franceEndpoint = getMockEndpoint("mock:direct:france");
          franceEndpoint.expectedMessageCount(1);

    //define the expectations on the direct:usa mocked endpoint
          MockEndpoint usaEndpoint = getMockEndpoint("mock:direct:usa");
          usaEndpoint.expectedMessageCount(0);

    //define the expectations on the direct:error mocked endpoint
          MockEndpoint errorEndpoint = getMockEndpoint("mock:direct:error");
          errorEndpoint.expectedMessageCount(0);

    //define the expectations on the direct:other mocked endpoint
          MockEndpoint otherEndpoint = getMockEndpoint("mock:direct:other");
          otherEndpoint.expectedMessageCount(0);

    //sending the message in the direct:input mocked endpoint
          template.sendBody("direct:input", message);

    //validate the expectations
          assertMockEndpointsSatisfied();
      }
      @Test
      public void testRoutingUsa() throws Exception {
          String message = "<company><country>USA</country></company>";

    //define the expectations on the direct:france mocked endpoint
          MockEndpoint franceEndpoint = getMockEndpoint("mock:direct:france");
          franceEndpoint.expectedMessageCount(0);

    //define the expectations on the direct:usa mocked endpoint
          MockEndpoint usaEndpoint = getMockEndpoint("mock:direct:usa");
          usaEndpoint.expectedMessageCount(1);

    //define the expectations on the direct:error mocked endpoint
          MockEndpoint errorEndpoint = getMockEndpoint("mock:direct:error");
          errorEndpoint.expectedMessageCount(0);

    //define the expectations on the direct:other mocked endpoint
          MockEndpoint otherEndpoint = getMockEndpoint("mock:direct:other");
          otherEndpoint.expectedMessageCount(0);

    //sending the message in the direct:input mocked endpoint
          template.sendBody("direct:input", message);

    //validate the expectations
          assertMockEndpointsSatisfied();
      }

      @Test
      public void testRoutingOther() throws Exception {
          String message = "<company><country>Spain</country></company>";

    //define the expectations on the direct:france mocked endpoint
          MockEndpoint franceEndpoint = getMockEndpoint("mock:direct:france");
          franceEndpoint.expectedMessageCount(0);

    //define the expectations on the direct:usa mocked endpoint
          MockEndpoint usaEndpoint = getMockEndpoint("mock:direct:usa");
          usaEndpoint.expectedMessageCount(0);

    //define the expectations on the direct:error mocked endpoint
          MockEndpoint errorEndpoint = getMockEndpoint("mock:direct:error");
          errorEndpoint.expectedMessageCount(0);

    //define the expectations on the direct:other mocked endpoint
          MockEndpoint otherEndpoint = getMockEndpoint("mock:direct:other");
          otherEndpoint.expectedMessageCount(1);

    //sending the message in the direct:input mocked endpoint
          template.sendBody("direct:input", message);

    //validate the expectations
          assertMockEndpointsSatisfied();
      }

      @Test
      public void testError() throws Exception {
          String message = "<company><country>France</country></company>";

    // fake an error on the direct:france mocked endpoint
          MockEndpoint franceEndpoint = getMockEndpoint("mock:direct:france");
          franceEndpoint.whenAnyExchangeReceived(new Processor() {
              public void process(Exchange exchange) throws Exception {
                  throw new IOException("Simulated error");
              }
          });

     //define the expectations on the direct:usa mocked endpoint
          MockEndpoint usaEndpoint = getMockEndpoint("mock:direct:usa");
          usaEndpoint.expectedMessageCount(0);

    //define the expectations on the direct:error mocked endpoint
          MockEndpoint errorEndpoint = getMockEndpoint("mock:direct:error");
          errorEndpoint.expectedMessageCount(1);

    //define the expectations on the direct:other mocked endpoint
          MockEndpoint otherEndpoint = getMockEndpoint("mock:direct:other");
          otherEndpoint.expectedMessageCount(0);

    //sending the message in the direct:input mocked endpoint
          template.sendBody("direct:input", message);

          // validate the expectations
          assertMockEndpointsSatisfied();
      }

}
```

这个类扩展了`CamelBlueprintTestSupport`类，因为我们的路由是用 Blueprint DSL 编写的。在实际上实现测试之前，我们必须*引导*测试。

第一步是加载 Blueprint XML。为此，我们重写了`getBlueprintDescriptor()`方法。这个方法只是返回 Blueprint XML 文件的位置。

第二步是定义我们想要模拟的端点。因此，我们重写了`isMockEndpointsAndSkip()`方法。这个方法返回一个用于匹配端点 URI 的正则表达式。Camel 将模拟相应的端点，并且不会将消息发送到实际端点。在这里，我们想要模拟所有路由的*出站*端点——`direct:error`、`direct:france`、`direct:usa`和`direct:other`。我们不想模拟`direct:input`的*入站*端点，因为我们将在那里使用生产者模板发送交换。

现在我们已经准备好实现单元测试了。

测试通过方法注解`@Test`实现。

第一个测试方法是`testRoutingFrance()`。这个测试：

+   创建一个包含国家元素且值为`France`的 XML 消息

+   在模拟的`direct:france`端点，我们期望根据基于内容的路由 EIP 接收一条消息

+   在模拟的`direct:usa`端点，我们期望不接收任何消息

+   在模拟的`direct:error`端点，我们期望不接收任何消息

+   在模拟的`direct:other`端点，我们期望不接收任何消息

+   我们使用生产者模板将 XML 消息发送到`direct:input`端点

+   一旦消息被发送，我们检查期望是否得到满足

第二个测试方法是`testRoutingUsa()`。这个测试基本上与`testRoutingFrance()`方法相同。然而，我们想要测试带有包含`<country/>`元素且值为`USA`的 XML 消息的`ContentBasedRouter`。我们在不同的模拟端点上更新期望。

第三个测试方法是`testRoutingOther()`。这个测试基本上与前面两个方法相同。然而，我们想要测试带有包含`<country/>`元素且值为`Spain`的 XML 消息的`ContentBasedRouter`。我们相应地更新期望。

我们还想要测试我们的`DefaultErrorHandling`。因此，我们想要模拟一个错误，看看错误处理器是否如预期那样响应。

为了模拟一个错误，我们在模拟的`direct:france`端点上添加一个处理器。这个处理器抛出一个`IOException`。这个异常将被错误处理器捕获。

作为错误处理器，它将消息转发到`direct:error`端点，我们可以定义对模拟的`direct:error`端点的期望，以确保端点接收到了（由错误处理器转发的）*失败*的消息。

要执行我们的测试，我们只需运行：

```java
$ mvn clean test

[main] INFO org.apache.camel.blueprint.BlueprintCamelContext - Apache Camel 2.12.4 (CamelContext: 22-camel-12) is starting
[main] INFO org.apache.camel.management.DefaultManagementStrategy - JMX is disabled
[main] INFO org.apache.camel.impl.InterceptSendToMockEndpointStrategy - Adviced endpoint [direct://error] with mock endpoint [mock:direct:error]
[main] INFO org.apache.camel.impl.InterceptSendToMockEndpointStrategy - Adviced endpoint [direct://france] with mock endpoint [mock:direct:france]
[main] INFO org.apache.camel.impl.InterceptSendToMockEndpointStrategy - Adviced endpoint [direct://usa] with mock endpoint [mock:direct:usa]
[main] INFO org.apache.camel.impl.InterceptSendToMockEndpointStrategy - Adviced endpoint [direct://other] with mock endpoint [mock:direct:other]
[main] INFO org.apache.camel.blueprint.BlueprintCamelContext - AllowUseOriginalMessage is enabled. If access to the original message is not needed, then its recommended to turn this option off as it may improve performance.
[main] INFO org.apache.camel.blueprint.BlueprintCamelContext - StreamCaching is not in use. If using streams then its recommended to enable stream caching. See more details at http://camel.apache.org/stream-caching.html
[main] INFO org.apache.camel.blueprint.BlueprintCamelContext - Route: test started and consuming from: Endpoint[direct://input]
[main] INFO org.apache.camel.blueprint.BlueprintCamelContext - Total 1 routes, of which 1 is started.
[main] INFO org.apache.camel.blueprint.BlueprintCamelContext - Apache Camel 2.12.4 (CamelContext: 22-camel-12) started in 0.015 seconds
[main] INFO org.apache.camel.component.mock.MockEndpoint - Asserting: Endpoint[mock://direct:france] is satisfied
[main] INFO org.apache.camel.component.mock.MockEndpoint - Asserting: Endpoint[mock://direct:usa] is satisfied
[main] INFO org.apache.camel.component.mock.MockEndpoint - Asserting: Endpoint[mock://direct:error] is satisfied
[main] INFO org.apache.camel.component.mock.MockEndpoint - Asserting: Endpoint[mock://direct:other] is satisfied
[main] INFO com.packt.camel.test.RouteTest - ********************************************************************************
[main] INFO com.packt.camel.test.RouteTest - Testing done: testRoutingOther(com.packt.camel.test.RouteTest)
[main] INFO com.packt.camel.test.RouteTest - Took: 0.021 seconds (21 millis)
[main] INFO com.packt.camel.test.RouteTest - ********************************************************************************
[main] INFO org.apache.camel.blueprint.BlueprintCamelContext - Apache Camel 2.12.4 (CamelContext: 22-camel-12) is shutting down
[main] INFO org.apache.camel.impl.DefaultShutdownStrategy - Starting to graceful shutdown 1 routes (timeout 10 seconds)
[Camel (22-camel-12) thread #3 - ShutdownTask] INFO org.apache.camel.impl.DefaultShutdownStrategy - Route: test shutdown complete, was consuming from: Endpoint[direct://input]
[main] INFO org.apache.camel.impl.DefaultShutdownStrategy - Graceful shutdown of 1 routes completed in 0 seconds
[main] INFO org.apache.camel.blueprint.BlueprintCamelContext - Apache Camel 2.12.4 (CamelContext: 22-camel-12) uptime 0.024 seconds
[main] INFO org.apache.camel.blueprint.BlueprintCamelContext - Apache Camel 2.12.4 (CamelContext: 22-camel-12) is shutdown in 0.002 seconds
[main] INFO org.apache.aries.blueprint.container.BlueprintExtender - Destroying BlueprintContainer for bundle RouteTest
[main] INFO org.apache.aries.blueprint.container.BlueprintExtender - Destroying BlueprintContainer for bundle org.apache.aries.blueprint
[main] INFO org.apache.aries.blueprint.container.BlueprintExtender - Destroying BlueprintContainer for bundle org.apache.camel.camel-blueprint
[main] INFO org.apache.camel.impl.osgi.Activator - Camel activator stopping
[main] INFO org.apache.camel.impl.osgi.Activator - Camel activator stopped
[main] INFO org.apache.camel.test.blueprint.CamelBlueprintHelper - Deleting work directory target/bundles/1427661985280
Tests run: 4, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 3.025 sec - in com.packt.camel.test.RouteTest

Results :

Tests run: 4, Failures: 0, Errors: 0, Skipped: 0

[INFO] --------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] --------------------------------------------------------------
[INFO] Total time: 4.842 s
[INFO] Finished at: 2015-03-29T22:46:25+02:00
[INFO] Final Memory: 18M/303M

```

我们可以在输出消息中看到 Camel 创建的不同模拟端点（例如 `[main] INFO org.apache.camel.component.mock.MockEndpoint - Asserting: Endpoint[mock://direct:other]` 已满足）。

# 额外的注解

Camel 测试套件还提供了额外的注解，以便简化测试代码。

你可以使用`@EndpointInject`注解而不是使用`getMockEndpoint()`方法来获取模拟端点：

```java
@EndpointInject(uri = "mock:direct:france")
protected MockEndpoint franceEndpoint;
```

现在，我们可以在测试方法中直接使用`franceEndpoint`模拟端点：

```java
@Test
public void aTest() throws Exception {
  …
  franceEndpoint.expectedBodiesReceived("<foo/>");
  …
  franceEndpoint.assertIsSatisfied();
}
```

类似地，你可以在生产者模板上而不是定义端点 URI 时，使用`@Producer`注解来定义生产者模板发送消息的位置：

```java
@Produce(uri = "direct:input");
protected ProducerTemplate template;
```

我们现在可以直接使用生产者模板，而不必指定端点：

```java
@Test
public void aTest() throws Exception {
  …
  template.sendBodyAndHeader("<message/>", "foo", "bar"); 

}
```

# 模拟 OSGi 服务

Camel 蓝图测试套件允许你模拟和原型化 OSGi 服务。

为了实现这一点，套件使用了`PojoSR`库。

例如，我们想要测试以下路由：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

      <reference id="service" interface="org.apache.camel.Processor"/>

      <camelContext >
          <route id="test">
              <from uri="direct:input"/>
              <process ref="service"/>
              <to uri="direct:output"/>
          </route>
      </camelContext>

</blueprint>
```

如果这个路由非常简单，它通过`<reference/>`元素使用 OSGi 服务。在 OSGi 容器中，引用元素正在 OSGi 服务注册表中查找实际的服务。

与使用真实的蓝图容器而不是，Camel 蓝图测试套件允许你注册服务。为此，我们只需覆盖`addServicesOnStartup()`方法，在其中添加提供路由中使用的服务的 bean。

测试类如下：

```java
package com.packt.camel.test;

import org.apache.camel.Exchange; import org.apache.camel.Processor;
import org.apache.camel.component.mock.MockEndpoint;
import org.apache.camel.test.blueprint.CamelBlueprintTestSupport;
import org.apache.camel.util.KeyValueHolder;
import org.junit.Test;

import java.util.Dictionary;
import java.util.Map;

public class RouteTest extends CamelBlueprintTestSupport {

      @Override
      protected String getBlueprintDescriptor() {
          return "OSGI-INF/blueprint/route.xml";
      }

      @Override
      public String isMockEndpointsAndSkip() {
          return "direct:output";
      }

      @Override
      public void addServicesOnStartup(Map<String, KeyValueHolder<Object, Dictionary>> services) {
          KeyValueHolder serviceHolder = new KeyValueHolder(new Processor() {
             public void process(Exchange exchange) throws Exception {
                exchange.getIn().setBody("DONE", String.class);
             }
          }, null);
          services.put(Processor.class.getName(), serviceHolder);
      }

      @Test
      public void testRoute() throws Exception {
          String message = "BEGIN";

          MockEndpoint franceEndpoint = getMockEndpoint("mock:direct:output");
          franceEndpoint.expectedMessageCount(1);
          franceEndpoint.expectedBodiesReceived("DONE");

          template.sendBody("direct:input", message);

          assertMockEndpointsSatisfied();
      }
}
```

我们可以看到，我们直接在测试中定义了模拟服务。至于之前的测试，我们通过以下方式执行测试：

```java
$ mvn clean test

[main] INFO org.apache.aries.blueprint.container.BlueprintExtender - No quiesce support is available, so blueprint components will not participate in quiesce operations
[main] INFO com.packt.camel.test.RouteTest - *********************************************************************
[main] INFO com.packt.camel.test.RouteTest - Testing: testService(com.packt.camel.test.RouteTest)
[main] INFO com.packt.camel.test.RouteTest - *********************************************************************
[Blueprint Extender: 3] INFO org.apache.aries.blueprint.container.BlueprintContainerImpl - Bundle RouteTest is waiting for namespace handlers [http://camel.apache.org/schema/blueprint]
[main] INFO com.packt.camel.test.RouteTest - Skipping starting CamelContext as system property skipStartingCamelContext is set to be true.
[main] INFO org.apache.camel.blueprint.BlueprintCamelContext - Apache Camel 2.12.4 (CamelContext: 22-camel-3) is starting
[main] INFO org.apache.camel.management.DefaultManagementStrategy - JMX is disabled
[main] INFO org.apache.camel.impl.InterceptSendToMockEndpointStrategy - Adviced endpoint [direct://output] with mock endpoint [mock:direct:output]
[main] INFO org.apache.camel.blueprint.BlueprintCamelContext - AllowUseOriginalMessage is enabled. If access to the original message is not needed, then its recommended to turn this option off as it may improve performance.
[main] INFO org.apache.camel.blueprint.BlueprintCamelContext - StreamCaching is not in use. If using streams then its recommended to enable stream caching. See more details at http://camel.apache.org/stream-caching.html
[main] INFO org.apache.camel.blueprint.BlueprintCamelContext - Route: test started and consuming from: Endpoint[direct://input]
[main] INFO org.apache.camel.blueprint.BlueprintCamelContext - Total 1 routes, of which 1 is started.
[main] INFO org.apache.camel.blueprint.BlueprintCamelContext - Apache Camel 2.12.4 (CamelContext: 22-camel-3) started in 0.050 seconds
[main] INFO org.apache.camel.component.mock.MockEndpoint - Asserting: Endpoint[mock://direct:output] is satisfied
[main] INFO com.packt.camel.test.RouteTest - *********************************************************************
[main] INFO com.packt.camel.test.RouteTest - Testing done: testService(com.packt.camel.test.RouteTest)
[main] INFO com.packt.camel.test.RouteTest - Took: 0.062 seconds (62 millis)
[main] INFO com.packt.camel.test.RouteTest - *********************************************************************
[main] INFO org.apache.camel.blueprint.BlueprintCamelContext - Apache Camel 2.12.4 (CamelContext: 22-camel-3) is shutting down
[main] INFO org.apache.camel.impl.DefaultShutdownStrategy - Starting to graceful shutdown 1 routes (timeout 10 seconds)
[Camel (22-camel-3) thread #0 - ShutdownTask] INFO org.apache.camel.impl.DefaultShutdownStrategy - Route: test shutdown complete, was consuming from: Endpoint[direct://input]
[main] INFO org.apache.camel.impl.DefaultShutdownStrategy - Graceful shutdown of 1 routes completed in 0 seconds
[main] INFO org.apache.camel.blueprint.BlueprintCamelContext - Apache Camel 2.12.4 (CamelContext: 22-camel-3) uptime 0.070 seconds
[main] INFO org.apache.camel.blueprint.BlueprintCamelContext - Apache Camel 2.12.4 (CamelContext: 22-camel-3) is shutdown in 0.007 seconds
[main] INFO org.apache.aries.blueprint.container.BlueprintExtender - Destroying BlueprintContainer for bundle RouteTest
[main] INFO org.apache.aries.blueprint.container.BlueprintExtender - Destroying BlueprintContainer for bundle org.apache.aries.blueprint
[main] INFO org.apache.aries.blueprint.container.BlueprintExtender - Destroying BlueprintContainer for bundle org.apache.camel.camel-blueprint
[main] INFO org.apache.camel.impl.osgi.Activator - Camel activator stopping
[main] INFO org.apache.camel.impl.osgi.Activator - Camel activator stopped
[main] INFO org.apache.camel.test.blueprint.CamelBlueprintHelper - Deleting work directory target/bundles/1427662210482
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.744 sec - in com.packt.camel.test.RouteTest

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] --------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] --------------------------------------------------------------
[INFO] Total time: 3.573 s
[INFO] Finished at: 2015-03-29T22:50:12+02:00
[INFO] Final Memory: 18M/303M
[INFO] --------------------------------------------------------------

```

正如我们在本章中看到的，Camel 测试套件允许你轻松地原型化服务和端点，并测试你的路由。

测试对于确保实现的集成逻辑，以及确保错误处理程序和路由按预期反应，是非常重要的。

# 摘要

正如我们在本章中看到的，Camel 提供了丰富的功能，允许你轻松实现单元测试和集成测试。

由于这一点，你可以测试你想要在路由中实现集成逻辑，并且可以通过模拟集成逻辑的部分来继续你的实现。

使用此类测试，你可以使用测试驱动实现，即首先根据你的预期实现测试，然后根据这些预期实现你的路由。
