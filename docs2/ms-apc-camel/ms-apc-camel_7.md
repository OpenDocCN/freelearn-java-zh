# 第七章. 错误处理

在任何集成系统中，都有许多可能导致错误发生的原因。许多是未预见的，不容易预测，也不容易模拟。作为一个集成框架，Camel 提供了广泛的支持来处理错误，这非常灵活，能够处理非常不同类型的错误。

在本章中，我们将涵盖以下主题：

+   我们可以使用 Camel 处理哪种类型的错误

+   可用的不同 Camel 错误处理器

+   错误处理器的配置

# 错误类型

我们可以区分两种主要的错误类型——可恢复和不可恢复。让我们详细看看这些。

## 可恢复错误

可恢复错误是一种暂时性错误。这意味着这个错误可能在一定时间后自动恢复。

例如，一个暂时断开的网络连接会导致`IOException`。

在 Camel 中，基本上，异常被表示为可恢复错误。

在那种情况下，Camel 使用`setException`（可抛出原因）方法在交换中存储异常（可恢复错误）：

```java
Exchange.setException(new IOException("My exception"));
```

正如我们稍后将要看到的，包含异常的交换将被错误处理器捕获，并相应地做出反应。

## 不可恢复的错误

不可恢复错误是指无论您尝试多少次执行相同操作，错误都依然存在的错误。

例如，尝试访问数据库中不存在的表，或者访问不存在的 JMS 队列。

不可恢复错误表示为将`setFault`标志设置为`true`的消息。错误消息是正常消息体，如下所示：

```java
Message msg = Exchange.getOut();
msg.setFault(true);
msg.setBody("Some Error Message");
```

程序员可以设置一个错误消息，以便 Camel 可以相应地做出反应并停止路由消息。

可能的问题可能是，在哪种情况下我们在交换中使用异常，在哪种情况下我们在消息上使用错误标志？

+   错误标志存在的第一个原因是 Camel API 是围绕 JBI 设计的，其中包括一个`Fault`消息概念。

+   第二个原因是我们可能希望以不同的方式处理一些错误。例如，在交换中使用异常将使用`ErrorHandler`，这意味着对于`InOut`交换，路由的下一个端点永远不会收到`out`消息。

使用错误标志允许您以特定方式处理这类错误。例如，对于 CXF 端点，创建并返回一个 SOAP 错误是有意义的。然而，我们将看到所有类型的错误都可以通过 Camel 的错误处理器来处理。

# Camel 错误处理器

正如我们所见，Camel 使用`setException(Throwable cause)`方法在交换中存储异常。

Camel 提供了现成的错误处理器，取决于您必须实现的机制。这些错误处理器只会对交换中设置的异常做出反应。默认情况下，如果设置了不可恢复的错误作为故障消息，错误处理器不会做出反应。我们将在本章的后面看到，Camel 提供了一个处理不可恢复错误的选项。

为了做出反应，错误处理器 *存在于* 路由通道上。实际上，错误处理器是一个拦截器（在通道上），它分析交换，并验证交换的异常属性是否不为空。

如果异常不为空，错误处理器 *做出反应*。这意味着错误处理器将 *捕获* 在 Camel 路由或处理消息过程中抛出的任何未捕获的异常。

Camel 根据您的需求提供不同类型的错误处理器。

## 非事务性错误处理器

本节中提到了非事务性错误处理器。

### DefaultErrorHandler

`DefaultErrorHandler` 是默认的错误处理器。它不支持死信队列，它将异常传播回调用者。交换立即结束。

它与死信错误处理器非常相似，但有效载荷已丢失（而 DLQ 保留有效载荷以供处理）。

这意味着它支持重试策略。正如我们稍后将要看到的，我们可以使用一些选项来配置错误处理器。

这个错误处理器涵盖了大多数用例。它捕获处理器中的异常并将它们传播回之前的通道，在那里错误处理器可以捕获它。这给了 Camel 机会相应地做出反应，例如，将消息重新路由到不同的路由路径，尝试重试，或者放弃并传播异常回调用者。

即使你没有明确指定错误处理器，Camel 也会隐式创建一个 `DefaultErrorHandler`，没有重试，没有处理（请参阅错误处理器功能以获取有关处理的详细信息），也没有死信队列（因为它不受 `DefaultErrorHandler` 的支持）。

为了说明 `DefaultErrorHandler`，我们创建一个简单的 Camel 路由，该路由将公开一个 HTTP 服务（使用 Jetty）。

首先，我们创建以下 Maven `pom.xml`：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <groupId>com.packt.camel</groupId>
  <artifactId>chapter7a</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>bundle</packaging>

  <dependencies>
      <dependency>
          <groupId>org.apache.camel</groupId>
          <artifactId>camel-core</artifactId>
          <version>2.12.4</version>
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

路由相当简单。它使用 Camel Jetty 组件公开一个 HTTP 服务，并使用一个 Bean 验证提交的消息。

我们使用 Blueprint DSL 编写这个路由。我们添加以下 `src/main/resources/OSGI-INF/blueprint/route.xml` 文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="checker" class="com.packt.camel.chapter7a.Checker"/>

  <camelContext >
      <route>
          <from uri="jetty:http://0.0.0.0:9999/my/route"/>
          <to uri="bean:checker"/>
      </route>
  </camelContext>

</blueprint>
```

`checker bean` 非常简单。它接收 Jetty 端点接收到的消息并检查其是否有效。消息是一个 HTTP 参数 `key=value`。如果参数的格式为 message=...，则有效，否则我们抛出 `IllegalArgumentException`。

这是 `checker` 代码：

```java
package com.packt.camel.chapter7a;

public class Checker {

  public String validate(String body) throws Exception {
      String[] param = body.split("=");
      if (param.length != 2) {
          throw new IllegalArgumentException("Bad parameter");
      }
      if (!param[0].equalsIgnoreCase("message")) {
          throw new IllegalArgumentException("Message parameter expected");
      }
      return "Hello " + param[1] + "\n";
  }

}
```

我们可以使用以下代码构建我们的包：

```java
$ mvn clean install
[INFO] Scanning for projects...
[INFO]
[INFO] -------------------------------------------------------------
[INFO] Building chapter7a 1.0-SNAPSHOT
[INFO] -------------------------------------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ chapter7a
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ chapter7a ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Copying 1 resource
[INFO]
[INFO] --- maven-compiler-plugin:3.2:compile (default-compile) @ chapter7a ---
[INFO] Changes detected - recompiling the module!
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[INFO] Compiling 1 source file to /home/jbonofre/Workspace/sample/chapter7a/target/classes
[INFO]
[INFO] --- maven-resources-plugin:2.6:testResources (default- testResources) @ chapter7a ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /home/jbonofre/Workspace/sample/chapter7a/src/test/resources
[INFO]
[INFO] --- maven-compiler-plugin:3.2:testCompile (default- testCompile) @ chapter7a ---
[INFO] No sources to compile
[INFO]
[INFO] --- maven-surefire-plugin:2.17:test (default-test) @ chapter7a
[INFO] No tests to run.
[INFO]
[INFO] --- maven-bundle-plugin:2.3.7:bundle (default-bundle) @ chapter7a ---
[INFO]
[INFO] --- maven-install-plugin:2.5.1:install (default-install) @ chapter7a ---
[INFO] Installing /home/jbonofre/Workspace/sample/chapter7a/target/chapter7a-1.0- SNAPSHOT.jar to /home/jbonofre/.m2/repository/com/packt/camel/chapter7a/1.0- SNAPSHOT/chapter7a-1.0-SNAPSHOT.jar
[INFO] Installing /home/jbonofre/Workspace/sample/chapter7a/pom.xml to /home/jbonofre/.m2/repository/com/packt/camel/chapter7a/1.0- SNAPSHOT/chapter7a-1.0-SNAPSHOT.pom
[INFO]
[INFO] --- maven-bundle-plugin:2.3.7:install (default-install) @ chapter7a ---
[INFO] Installing com/packt/camel/chapter7a/1.0-SNAPSHOT/chapter7a- 1.0-SNAPSHOT.jar
[INFO] Writing OBR metadata
[INFO] --------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] --------------------------------------------------------------
[INFO] Total time: 3.648 s
[INFO] Finished at: 2015-03-04T10:19:55+01:00
[INFO] Final Memory: 34M/1222M
[INFO] --------------------------------------------------------------

```

我们现在启动 Apache Karaf 容器并安装 `camel-blueprint` 和 `camel-jetty` 功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache- camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint
karaf@root()> feature:install camel-jetty
Refreshing bundles org.apache.camel.camel-core (60)

```

我们现在可以安装我们的包：

```java
karaf@root()> bundle:install mvn:com.packt.camel/chapter7a/1.0- SNAPSHOT
Bundle ID: 86
karaf@root()> bundle:start 86

```

我们可以使用 curl 提交一个有效的消息：

```java
$ curl -d "message=chapter7a" http://localhost:9999/my/route
Hello chapter7a

```

现在，我们提交一个无效的消息：

```java
$ curl -d "foo=bar" http://localhost:9999/my/route
java.lang.IllegalArgumentException: Message parameter expected

```

在 Karaf 的 `log` 文件（`data/karaf.log`）中，我们可以看到：

```java
2015-03-04 10:25:44,647 | ERROR | qtp766046018-75  | DefaultErrorHandler | rg.apache.camel.util.CamelLogger  215 | 60 - org.apache.camel.camel-core - 2.12.4 | Failed delivery for (MessageId: ID-latitude-40620-1425461026278-0-6 on ExchangeId: ID- latitude-40620-1425461026278-0-5). Exhausted after delivery attempt: 1 caught: java.lang.IllegalArgumentException: Message parameter expected

Message History
---------------------------------------------------------------------------------------------------------------------------------------
RouteId            ProcessorId        Processor                      Elapsed (ms)
[route1]           [route1]           [http://0.0.0.0:9999/my/route] [4]
[route1]           [to1]              [bean:checker]                 [3]

Exchange
---------------------------------------------------------------------------------------------------------------------------------------
Exchange[
 Id                ID-latitude-40620-1425461026278-0-5
 ExchangePattern   InOut
 Headers           {Accept=*/*, breadcrumbId=ID-latitude-40620- 1425461026278-0-6, CamelHttpMethod=POST, CamelHttpPath=, CamelHttpQuery=null, CamelHttpServletRequest=(POST /my/route)@132201555 org.eclipse.jetty.server.Request@7e13c53, CamelHttpServletResponse=HTTP/1.1 200
^M
, CamelHttpUri=/my/route, CamelHttpUrl=http://localhost:9999/my/route, CamelRedelivered=false, CamelRedeliveryCounter=0, CamelServletContextPath=/my/route, Content- Length=7, Content-Type=application/x-www-form-urlencoded, foo=bar, Host=localhost:9999, User-Agent=curl/7.35.0}
 BodyType 
org.apache.camel.converter.stream.InputStreamCache
 Body 
[Body is instance of org.apache.camel.StreamCache]
]

Stacktrace
---------------------------------------------------------------------------------------------------------------------------------------
java.lang.IllegalArgumentException: Message parameter expected at com.packt.camel.chapter7a.Checker.validate(Checker.java:11)[86:com.packt.camel.chapter7a:1.0.0.SNAPSHOT]

```

因此，我们可以看到 `DefaultErrorHandler` 对 `IllegalArgumentException` 做出了反应。投递失败并已被记录。默认情况下（因为处理为 false），异常会被抛回给调用者。

### DeadLetterChannel

`DeadLetterChannel` 错误处理器实现了死信通道 EIP。它支持重试策略，重试会将消息发送到死信端点。

即使如此，死信端点 `DeadLetterChannel` 的行为类似于 `DefaultErrorHandler`。为了说明这一点，我们更新之前的示例，使用一个在发生异常时调用错误路由的 `DeadLetterChannel`。

因此，`DeadLetterChannel` 错误处理器将捕获异常，尝试重试，如果仍然失败，消息将被发送到在死信 URI 中定义的专用路由。

`checker` 实例与之前完全相同。路由定义（使用 Blueprint DSL）不同，因为我们定义了 `DeadLetterChannel` 错误处理器：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="checker" class="com.packt.camel.chapter7b.Checker"/>

  <camelContext >
      <route errorHandlerRef="myDeadLetterErrorHandler">
          <from uri="jetty:http://0.0.0.0:9999/my/route"/>
          <to uri="bean:checker"/>
      </route>
      <route>
          <from uri="direct:error"/>
          <convertBodyTo type="java.lang.String"/>
          <to uri="log:error"/>
      </route>
  </camelContext>

  <bean id="myDeadLetterErrorHandler" class="org.apache.camel.builder.DeadLetterChannelBuilder">
      <property name="deadLetterUri" value="direct:error"/>
      <property name="redeliveryPolicy" ref="myRedeliveryPolicyConfig"/>
  </bean>

  <bean id="myRedeliveryPolicyConfig" class="org.apache.camel.processor.RedeliveryPolicy">
      <property name="maximumRedeliveries" value="3"/>
      <property name="redeliveryDelay" value="5000"/>
  </bean>

</blueprint>
```

我们可以看到我们在 `main` 路由中使用了 `myDeadLetterErrorHandler`。`myDeadLetterErrorHandler` 是使用 `DeadLetterChannelBuilder` 构造函数构建的。

以下属性被设置：

+   包含端点的 `deadLetterUri` 被设置为在投递失败时发送消息的位置。在这里，我们定义端点 `direct:error` 以调用相应的路由。

+   `redeliveryPolicy` 被设置为定义我们尝试重试消息的方式。`myRedeliveryPolicy` 定义了尝试次数（示例中的 3 次），以及每次尝试之间的延迟（这里为 5 秒）。这意味着在 3 次尝试（因此，最多 15 秒）之后，如果消息仍然失败，它将被发送到在死信 URI 中定义的端点（在我们的例子中是 `direct:error`）。

`error` 路由仅记录失败的消息。这意味着 `main` 路由不会失败，它只会将 `in` 消息返回给调用者。

我们使用以下代码构建我们的新包：

```java
$ mvn clean install
[INFO] ------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] -------------------------------------------------------

```

我们启动我们的 Apache Karaf：

```java
bin/karaf

```

如之前所做，我们安装了 `camel-blueprint` 和 `camel-jetty` 功能：

```java
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache- camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint camel-jetty
Refreshing bundles org.apache.camel.camel-core (60)
karaf@root()>

```

我们可以部署我们的包：

```java
karaf@root()> bundle:install mvn:com.packt.camel/chapter7b/1.0- SNAPSHOT
Bundle ID: 86
karaf@root()> bundle:start 86

```

首先，我们使用 curl 发送一个有效的消息：

```java
$ curl -d "message=chapter7b" http://localhost:9999/my/route
Hello chapter7b

```

它与之前一样工作，没有任何变化。

现在，我们发送一个无效的消息：

```java
$ curl -d "foo=bar" http://localhost:9999/my/route

```

我们可以注意到 curl 正在等待响应；这是正常的，因为我们已经在死信错误处理器中定义了重试策略。最后，我们收到了原始的 `in` 消息：

```java
foo=bar

```

如果我们查看 Karaf 的 `log` 文件，我们可以看到与 `error` 路由执行相对应的以下代码：

```java
2015-03-04 15:00:29,838 | INFO  | qtp1470784256-76 | error | org.apache.camel.util.CamelLogger  176 | 60 - org.apache.camel.camel- core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String,
 Body: foo=bar]

```

### LoggingErrorHandler

`LoggingErrorHandler` 将失败的消息以及异常记录下来。

Camel 将默认使用 `LoggingErrorHandler` 日志名称在 `ERROR` 级别记录失败的消息和异常。为了说明 `LoggingErrorHandler` 的行为，我们更新之前的路由 `blueprint` XML 如下：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >
  <bean id="checker" class="com.packt.camel.chapter7c.Checker"/>
  <camelContext >
      <errorHandler id="myLoggingErrorHandler" type="LoggingErrorHandler" logName="packt" level="ERROR"/>
      <route id="main" errorHandlerRef="myLoggingErrorHandler">
          <from uri="jetty:http://0.0.0.0:9999/my/route"/>
          <to uri="bean:checker"/>
      </route>
  </camelContext>

</blueprint>
```

我们在`main`路由中定义了一个`LoggingErrorHandler`。这个错误处理器将只是拦截并记录异常在`packt`记录器中，日志级别为`ERROR`。交换结束，异常返回给调用者。

在 Apache Karaf 中部署我们的 bundle 后，提交一个无效的消息，我们可以看到客户端（curl）得到以下异常：

```java
$ curl -d "foo=bar" http://localhost:9999/my/route
java.lang.IllegalArgumentException: Message parameter expected at com.packt.camel.chapter7c.Checker.validate(Checker.java:11)

```

异常被记录在 Karaf 的`log`文件中：

```java
2015-03-04 16:43:57,910 | ERROR | qtp1055249608-73 | packt | org.apache.camel.util.CamelLogger  215 | 60 - org.apache.camel.camel- core - 2.12.4 | Failed delivery for (MessageId: ID-latitude-41466- 1425483831913-0-2 on ExchangeId: ID-latitude-41466-1
425483831913-0-1). Exhausted after delivery attempt: 1 caught: java.lang.IllegalArgumentException: Message parameter expected

Message History
---------------------------------------------------------------------------------------------------------------------------------------
RouteId            ProcessorId        Processor                                                                      Elapsed (ms)
[route1]           [route1]           [http://0.0.0.0:9999/my/route]                                                      [18]
[route1]           [to1]              [bean:checker]                                                                       [15]

Exchange
---------------------------------------------------------------------------------------------------------------------------------------
Exchange[
 Id                ID-latitude-41466-1425483831913-0-1
 ExchangePattern   InOut
 Headers           {Accept=*/*, breadcrumbId=ID-latitude-41466- 1425483831913-0-2, CamelHttpMethod=POST, CamelHttpPath=, CamelHttpQuery=null, CamelHttpServletRequest=(POST /my/route)@1904375366 org.eclipse.jetty.server.Request@71827646, CamelHttpServletResponse=HTTP/1.1 200

, CamelHttpUri=/my/route, CamelHttpUrl=http://localhost:9999/my/route, CamelRedelivered=false, CamelRedeliveryCounter=0, CamelServletContextPath=/my/route, Content- Length=7, Content-Type=application/x-www-form-urlencoded, foo=bar, Host=localhost:9999, User-Agent=curl/7.35.0}
 BodyType 
org.apache.camel.converter.stream.InputStreamCache
 Body 
[Body is instance of org.apache.camel.StreamCache]
]

Stacktrace
---------------------------------------------------------------------------------------------------------------------------------------
java.lang.IllegalArgumentException: Message parameter expected at com.packt.camel.chapter7c.Checker.validate(Checker.java:11)[86:com.packt.camel.chapter7c:1.0.0.SNAPSHOT]

```

### NoErrorHandler

`NoErrorHandler`完全禁用了错误处理；这意味着任何异常都不会被拦截，只是返回给调用者。

例如，如果我们像这样更新我们的示例的 Blueprint XML：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="checker" class="com.packt.camel.chapter7d.Checker"/>

  <camelContext >
      <errorHandler id="myNoErrorHandler" type="NoErrorHandler"/>
      <route errorHandlerRef="myNoErrorHandler">
          <from uri="jetty:http://0.0.0.0:9999/my/route"/>
          <to uri="bean:checker"/>
      </route>
  </camelContext>

</blueprint>
```

如果我们提交一个无效的消息，交换不会结束，我们没有日志记录任何内容，异常只是返回给调用者。

## TransactedErrorHandler

当路由被标记为事务时，使用`TransactedErrorHandler`。

它基本上与`DefaultErrorHandler`相同（它实际上是从`DefaultErrorHandler`继承的）。区别在于`TransactedErrorHandler`将寻找一个事务管理器。

它使用以下机制来找到它：

+   如果注册表中的只有一个 bean 具有`org.apache.camel.spi.TransactedPolicy`类型，它将使用它

+   如果注册表中的 bean 具有`ID PROPAGATION_REQUIRED`和`org.apache.camel.spi.TransactedPolicy`类型，它将使用它

+   如果注册表中只有一个 bean 具有`org.springframework.transaction.PlatformTransactionManager`，它将使用它

您也可以*强制*使用您想要的交易管理器，因为事务表示法接受 bean ID。

# 错误处理器作用域

可以定义一个错误处理器：

+   在 Camel 上下文级别（Camel 上下文作用域），这意味着在这个 Camel 上下文中的所有路由都将使用这个错误处理器。

+   在路由级别（路由作用域），可能覆盖了使用 Camel 上下文作用域定义的错误处理器。

多亏了作用域，可以定义一个默认错误处理器（Camel 上下文作用域），并且可能定义一个特定于特定路由的错误处理器。

例如，以下 Blueprint XML 包含两个路由和两个不同的错误处理器——一个具有 Camel 上下文作用域，另一个具有`route`作用域。

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="checker" class="com.packt.camel.chapter7e.Checker"/>

  <camelContext  errorHandlerRef="deadLetterErrorHandler">
      <errorHandler id="noErrorHandler" type="NoErrorHandler"/>
      <route>
          <from uri="jetty:http://0.0.0.0:9999/my/route"/>
          <to uri="bean:checker"/>
      </route>
      <route errorHandlerRef="noErrorHandler">
           <from uri="jetty:http://0.0.0.0:8888/my/route"/>
           <to uri="bean:checker"/>
      </route>
  </camelContext>

  <bean id="deadLetterErrorHandler" class="org.apache.camel.builder.DeadLetterChannelBuilder">       <property name="deadLetterUri" value="direct:error"/>
      <property name="redeliveryPolicy"ref="myRedeliveryPolicyConfig"/>
  </bean>

  <bean id="myRedeliveryPolicyConfig" class="org.apache.camel.processor.RedeliveryPolicy">
      <property name="maximumRedeliveries" value="3"/>
      <property name="redeliveryDelay" value="5000"/>
  </bean>

</blueprint>
```

# 错误处理器功能

基本上，所有错误处理器都扩展了`DefaultErrorHandler`。`DefaultErrorHandler`提供了一套有趣的功能，允许您使用非常细粒度的异常管理。

## 重发

`DefaultErrorHandler`（以及`DeadLetterErrorHandler`和`TransactedErrorHandler`）支持一个可以通过重发策略配置的重发机制。

例如，以下 Blueprint XML 创建了一个 Camel 路由，该路由会系统地抛出`IllegalArgumentException`异常（带有`Booooommmmm`消息）。由于我们没有显式定义错误处理器，该路由使用`DefaultErrorHandler`。我们只配置了`DefaultErrorHandler`的重试策略，尝试重发消息三次，每次尝试之间等待两秒钟。如果第四次尝试仍然失败，交换结束，异常发送给调用者。

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="myException" class="java.lang.IllegalArgumentException">
      <argument value="Booooommmmm"/>
  </bean>

  <camelContext >
      <errorHandler id="defaultErrorHandler">
           <redeliveryPolicy maximumRedeliveries="3" redeliveryDelay="2000"/>
      </errorHandler>
      <route errorHandlerRef="defaultErrorHandler">
          <from uri="jetty:http://0.0.0.0:9999/my/route"/>
          <throwException ref="myException"/>
      </route>
  </camelContext>

</blueprint>
```

我们启动 Apache Karaf 容器并安装`camel-blueprint`和`camel-jetty`功能：

```java
$ bin/karaf
karaf@root> features:chooseurl camel 2.12.4
karaf@root> features:install camel-blueprint camel-jetty

```

我们只需将`route.xml`放入 Karaf 的`deploy`文件夹中，然后使用 curl 调用服务：

```java
$ curl http://localhost:9999/my/route

```

经过一段时间后，我们可以看到异常（带有`Booooommmmm`消息的`IllegalArgumentException`）返回给客户端：

```java
$ curl http://localhost:9999/my/route
java.lang.IllegalArgumentException: Booooommmmm

```

在 Karaf 的`log`文件中，我们可以看到以下代码：

```java
2015-03-05 13:04:32,447 | ERROR | qtp307037456-75  | DefaultErrorHandler | rg.apache.camel.util.CamelLogger  215 | 60 - org.apache.camel.camel-core - 2.12.4 | Failed delivery for (MessageId: ID-latitude-34120-1425557044231-0-2 on ExchangeId: ID- latitude-34120-1425557044231-0-1). Exhausted after delivery attempt: 4 caught: java.lang.IllegalArgumentException: Booooommmmm

Message History
---------------------------------------------------------------------------------------------------------------------------------------
RouteId            ProcessorId        Processor                       Elapsed (ms)
[route1]           [route1]           [http://0.0.0.0:9999/my/route]   [6010]
[route1]           [throwException1]    [throwException[java.lang.IllegalArgumentException]]                               [6007]

Exchange
---------------------------------------------------------------------------------------------------------------------------------------
Exchange[
 Id                ID-latitude-34120-1425557044231-0-1
 ExchangePattern   InOut
 Headers           {Accept=*/*, breadcrumbId=ID-latitude-34120- 1425557044231-0-2, CamelHttpMethod=GET, CamelHttpPath=, CamelHttpQuery=null, CamelHttpServletRequest=(GET /my/route)@1911523579 org.eclipse.jetty.server.Request@71ef88fb, CamelHttpServletResponse=HTTP/1.1 200

, CamelHttpUri=/my/route, CamelHttpUrl=http://localhost:9999/my/route, CamelRedelivered=true, CamelRedeliveryCounter=3, CamelRedeliveryMaxCounter=3, CamelServletContextPath=/my/route, Content-Type=null, Host=localhost:9999, User-Agent=curl/7.35.0}
 BodyType          null
 Body              [Body is null]
]

Stacktrace
---------------------------------------------------------------------------------------------------------------------------------------
java.lang.IllegalArgumentException: Booooommmmm at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)[:1.7.0_67]

```

我们可以看到已经使用了重试策略，并且在第四次尝试时交换失败（在第四次尝试后耗尽）。

## 异常策略

`DefaultErrorHandler`（以及`DeadLetterChannel`和`TransactedErrorHandler`）支持异常策略。异常策略用于以特定方式拦截和处理特定的异常。

异常策略使用`onException`语法指定。Camel 将从底部向上遍历异常层次结构，寻找与实际异常匹配的`onException`。

你可以在`CamelContext`作用域或路由作用域中定义错误处理器后使用`onException`。例如，在以下`route.xml`中，我们有两个不同的路由：

+   第一个路由抛出`IllegalArgumentException`异常（`Boooommmm`）

+   第二个路由抛出`IllegalStateException`异常（`Kabooommmm`）

我们希望对这两个异常有不同的反应：

+   对于`IllegalArgumentException`，我们希望定义一个特定的重试策略

+   对于`IllegalStateException`，我们希望将消息重定向到特定的端点

对于这两个异常，交换结束，异常被发送回调用者。

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="illegalArgumentException" class="java.lang.IllegalArgumentException">
      <argument value="Booooommmmm"/>
  </bean>
  <bean id="illegalStateException" class="java.lang.IllegalStateException">
      <argument value="Kaboooommmmm"/>
  </bean>

  <camelContext >
      <onException>
          <exception>java.lang.IllegalArgumentException</exception>
          <redeliveryPolicy maximumRedeliveries="2" redeliveryDelay="1000"/>
      </onException>
      <onException>
          <exception>java.lang.IllegalStateException</exception>
          <to uri="direct:error"/>
      </onException>
      <route>
          <from uri="jetty:http://0.0.0.0:9999/my/route"/>
          <throwException ref="illegalArgumentException"/>
      </route>
      <route>
          <from uri="jetty:http://0.0.0.0:8888/my/route"/>
          <throwException ref="illegalStateException"/>
      </route>
      <route>
          <from uri="direct:error"/>
          <convertBodyTo type="java.lang.String"/>
          <to uri="log:error"/>
      </route>
  </camelContext>

</blueprint>
```

我们启动 Apache Karaf 容器并安装`camel-blueprint`和`camel-jetty`功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
karaf@root()> feature:install camel-blueprint camel-jetty

```

我们将`route.xml`放入 Karaf 的`deploy`文件夹中。

现在，如果你访问第一个路由对应的 HTTP 端点，会抛出`IllegalArgumentException`异常：

```java
$ curl http://localhost:9999/my/route
java.lang.IllegalArgumentException: Booooommmmm

```

在 Karaf 的`log`文件中，我们可以看到以下代码：

```java
2015-03-05 17:24:53,621 | ERROR | qtp486271114-76  | DefaultErrorHandler | rg.apache.camel.util.CamelLogger  215 | 60 - org.apache.camel.camel-core - 2.12.4 | Failed delivery for (MessageId: ID-latitude-34088-1425572554935-0-2 on ExchangeId: ID- latitude-34088-1425572554935-0-1). Exhausted after delivery attempt: 3 caught: java.lang.IllegalArgumentException: Booooommmmm

Message History
---------------------------------------------------------------------------------------------------------------------------------------
RouteId            ProcessorId        Processor                      Elapsed (ms)
[route1]           [route1]           [http://0.0.0.0:9999/my/route]  [2009]
[route1]           [throwException1]          [throwException[java.lang.IllegalArgumentException]    [2005]

Exchange
---------------------------------------------------------------------------------------------------------------------------------------
Exchange[
 Id                ID-latitude-34088-1425572554935-0-1
 ExchangePattern   InOut
 Headers           {Accept=*/*, breadcrumbId=ID-latitude-34088- 1425572554935-0-2, CamelHttpMethod=GET, CamelHttpPath=, CamelHttpQuery=null, CamelHttpServletRequest=(GET /my/route)@1882419051 org.eclipse.jetty.server.Request@70336f6b, CamelHttpServletResponse=HTTP/1.1 200

, CamelHttpUri=/my/route, CamelHttpUrl=http://localhost:9999/my/route, CamelRedelivered=true, CamelRedeliveryCounter=2, CamelRedeliveryMaxCounter=2, CamelServletContextPath=/my/route, Content-Type=null, Host=localhost:9999, User-Agent=curl/7.35.0}
 BodyType          null
 Body              [Body is null]
]

Stacktrace
---------------------------------------------------------------------------------------------------------------------------------------
java.lang.IllegalArgumentException: Booooommmmm at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)[:1.7.0_67]

```

如果你访问第二个路由对应的 HTTP 端点，会抛出`IllegalStateException`异常：

```java
$ curl http://localhost:8888/my/route
java.lang.IllegalStateException: Kaboooommmmm

```

在 Karaf 的`log`文件中，我们可以看到以下代码：

```java
2015-03-05 17:25:02,965 | INFO  | qtp1797782377-82 | error | rg.apache.camel.util.CamelLogger  176 | 60 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOut, BodyType: null, Body: [Body is null]]
2015-03-05 17:25:02,969 | ERROR | qtp1797782377-82 | DefaultErrorHandler | rg.apache.camel.util.CamelLogger 215 | 60 - org.apache.camel.camel-core - 2.12.4 | Failed delivery for (MessageId: ID-latitude-34088-1425572554935-0-4 on ExchangeId: ID- latitude-34088-1425572554935-0-3). Exhausted after delivery attempt: 1 caught: java.lang.IllegalStateException: Kaboooommmmm. Processed by failure processor: FatalFallbackErrorHandler[Channel[sendTo(Endpoint[direct://error])]]

Message History
---------------------------------------------------------------------------------------------------------------------------------------
RouteId            ProcessorId        Processor                                                                      Elapsed (ms)
[route2]           [route2]           [http://0.0.0.0:8888/my/route]              [7]
[route2]           [throwException2]    [throwException[java.lang.IllegalStateException]]                       [to1]              [direct:error]                 [4]
[route3]           [convertBodyTo1] [convertBodyTo[java.lang.String]] [0]
[route3]           [to2]              [log:error]                     [2]

Exchange
---------------------------------------------------------------------------------------------------------------------------------------
Exchange[
 Id                ID-latitude-34088-1425572554935-0-3
 ExchangePattern   InOut
 Headers           {Accept=*/*, breadcrumbId=ID-latitude-34088- 1425572554935-0-4, CamelHttpMethod=GET, CamelHttpPath=, CamelHttpQuery=null, CamelHttpServletRequest=(GET /my/route)@10303
16697 org.eclipse.jetty.server.Request@3d696299, CamelHttpServletResponse=HTTP/1.1 200

, CamelHttpUri=/my/route, CamelHttpUrl=http://localhost:8888/my/route, CamelRedelivered=false, CamelRedeliveryCounter=0, CamelServletContextPath=/my/route, Content- Type=null, Host=localhost:8888, User-Agent=curl/7.35.0}
 BodyType          null
 Body              [Body is null]
]

Stacktrace
---------------------------------------------------------------------------------------------------------------------------------------
java.lang.IllegalStateException: Kaboooommmmm at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)[:1.7.0_67]

```

我们可以看到那里使用了`onException`，将交换重定向到`error`路由。

## 处理和忽略异常

当处理异常时，Camel 会跳出路由执行。

当设置`handled`标志时，异常不会被发送回调用者，并且你可以定义一个`error`消息。

例如，以下是一个`route.xml`文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="illegalArgumentException" class="java.lang.IllegalArgumentException">
      <argument value="Booooommmmm"/>
  </bean>

  <camelContext >
      <onException>
          <exception>java.lang.IllegalArgumentException</exception>
          <redeliveryPolicy maximumRedeliveries="2" redeliveryDelay="1000"/>
          <handled><constant>true</constant></handled>
          <transform><constant>Ouch we got an error</constant></transform>
      </onException>
      <route>
          <from uri="jetty:http://0.0.0.0:9999/my/route"/>
          <throwException ref="illegalArgumentException"/>
      </route>
  </camelContext>

</blueprint>
```

`onException` 上的处理标志防止将异常发送回调用者。在这种情况下，我们使用一个常量字符串定义一个错误消息。

我们启动 Apache Karaf 并安装 `camel-blueprint` 和 `camel-jetty` 功能：

```java
$ bin/karaf
karaf@root> features:chooseurl camel 2.12.4
karaf@root> features:install camel-blueprint camel-jetty

```

我们将 `route.xml` 放在 Karaf 的 `deploy` 文件夹中。如果我们访问 HTTP 端点，我们会得到：

```java
$ curl http://localhost:9999/my/route
Ouch we got an error

```

另一方面，可以使用 `continued` 标志完全忽略异常。例如，以下 `route.xml`：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="illegalArgumentException" class="java.lang.IllegalArgumentException">
      <argument value="Booooommmmm"/>
  </bean>

  <camelContext >
      <onException>
          <exception>java.lang.IllegalArgumentException</exception>
          <continued><constant>true</constant></continued>
      </onException>
      <route>
          <from uri="jetty:http://0.0.0.0:9999/my/route"/>
          <throwException ref="illegalArgumentException"/>
      </route>
  </camelContext>

</blueprint>
```

我们忽略路由抛出的 `IllegalArgumentException`。这意味着如果我们使用 curl 访问 HTTP 端点，我们只会得到一个响应：

```java
$ curl http://localhost:9999/my/route
$

```

这意味着由于 `continued` 标志，`IllegalArgumentException` 被忽略了。

## 备用解决方案

`DefaultErrorHandler`（以及 `DeadLetterChannel` 和 `TransactedErrorHandler`）支持将失败的交换路由到特定的端点。多亏了这个机制，我们可以实现一种备用解决方案，或者路由，可以撤销之前的变化。

以下 `route.xml` 实现了这样的备用方案：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <camelContext >
  <onException>
  <exception>java.io.IOException</exception>
  <redeliveryPolicy maximumRedeliveries="3"/>
  <handled><constant>true</constant></handled>
  <to uri= "ftp://fallbackftp.packt.com?user=anonymous&amp;password=foobar"/>
  </onException>
    <route>
  <from uri="file:/tmp/in"/>
  <to uri="ftp://ftp.packt.com?user=anonymous&amp;password=foobar"/>
    </route>
  </camelContext>

</blueprint>
```

我们尝试将本地文件上传到 FTP 服务器。如果在 FTP 服务器中抛出 `IOException`，我们会响应 `IOException`（意味着 FTP 服务器有问题），尝试在同一个 FTP 服务器上重新投递三次。最后，我们重定向到一个备用（另一个）FTP 服务器。

## onWhen

如果 `onException` 允许你根据异常过滤异常并做出相应反应，也可以添加另一个条件来响应特定的异常。这就是你可以使用 `onWhen` 语法，接受一个谓词来做到的。你有一个更细粒度的方法来过滤异常，如下所示：

```java
<onException>
  <exception>java.io.IOException</exception>
  <onWhen><simple>${header.foo} == bar</simple></onWhen>
  <to uri="direct:my"/>
</onException>
```

## onRedeliver

`onRedeliver` 语法允许你在消息重新投递之前执行一些代码。例如，你可以调用一个用于重新投递的处理程序，如下所示：

```java
<onException onRedeliveryRef="myRedeliveryProcessor">
  <exception>java.io.IOException</exception>
</onException>
```

## retryWhile

与定义静态的重新投递次数不同，你可以使用 `retryWhile` 语法。它允许你在运行时确定是否继续重新投递或放弃。

它允许你拥有细粒度的重新投递控制。

# 尝试、捕获和最终

到目前为止，我们使用了错误处理器（大多数情况下是 `DefaultErrorHandler`），它适用于路由中的所有通道。你可能希望将异常处理“平方”到路由的某个部分。

它类似于 Java 中的 `try/catch/finally` 语句。

```java
<route>
    <from uri="direct:start"/>
    <doTry>
        <process ref="processor"/>
        <to uri="direct:result"/>
        <doCatch>
            <!-- catch multiple exceptions -->
            <exception>java.io.IOException</exception>
            <exception>java.lang.IllegalStateException</exception>
            <to uri="direct:catch"/>
        </doCatch>
        <doFinally>
            <to uri="direct:finally"/>
        </doFinally>
    </doTry>
</route>
```

Camel 错误处理被禁用。当使用 `doTry` .. `doCatch` .. `doFinally` 时，常规 Camel 错误处理器不适用。这意味着任何 `onException` 或类似操作都不会触发。原因是 `doTry` .. `doCatch` .. `doFinally` 实际上是一个自己的错误处理器，它的目的是模仿并像 Java 中的 `try/catch/finally` 一样工作。

# 摘要

通常来说，错误处理是困难的。这就是为什么 Camel 提供了大量关于错误处理的特性。即使你可以使用`doTry/doCatch/doFinally`语法，大多数时候将路由逻辑本身与错误处理分离会更好。

当可能时，尝试恢复是良好的实践。使用恢复策略总是一个好主意。强烈建议构建单元测试来模拟错误。这就是我们在下一章将要看到的内容——使用 Camel 进行测试。
