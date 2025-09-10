# 第三章：路由和处理程序

在上一章中，我们看到了 Camel 提供消息和路由系统实现的核心概念。

在本章中，我们将介绍路由——Camel 最重要的功能之一。没有路由，Camel 将只是一个简单的**连接**框架。路由是 Camel 的关键功能，这意味着我们可以将所有转换应用于消息。它可以修改消息的内容或消息的目的地，所有这些都是在运行时完成的。

本章介绍了：

+   如何使用处理器来改变交换

+   包含处理器的完整路由示例

# 什么是处理器？

消费者端点从环境中接收一个事件，并将其封装为一个**交换**。

路由引擎将这个交换从端点传输到处理器，这可以从一个处理器传输到另一个处理器，直到通过**通道**到达最终的端点。如果消息交换模式（MEP）是`InOut`（并使用输出消息），则路由可以在返回消费者端点的处理器处结束，或者通过生产者端点停止，将消息发送到环境。

这意味着处理器充当交换修改器——它消费一个交换，并最终更新它。我们可以将处理器视为消息翻译器。实际上，所有 Camel**交换集成模式**（**EIP**）都是使用处理器实现的。

处理器通过`org.apache.camel.Processor`接口进行描述。此接口仅提供一种方法：

```java
void process(Exchange exchange) throws Exception(Exchange exchange) throws Exception
```

由于处理器直接接收交换，它可以访问交换中包含的所有数据：

1.  消息交换模式

1.  输入消息

1.  输出消息

1.  交换异常

# 包含处理器的 Camel 路由示例

在这里，我们将通过一个示例来说明处理器在其中的使用。这个示例将创建一个**OSGi**包，该包将创建两个 Camel 上下文，每个上下文有一个路由；它们如下所示：

+   使用 Camel Java DSL 的一个路由

+   使用 Camel Blueprint DSL 的一个路由

首先，我们为我们的包创建 Maven 项目`pom.xml`：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <groupId>com.packt.camel</groupId>
  <artifactId>chapter3</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>bundle</packaging>

  <dependencies>
        <dependency>
            <groupId>org.apache.camel</groupId>
            <artifactId>camel-core</artifactId>
            <version>2.12.4</version>
        </dependency>
        <dependency>
            <groupId>org.osgi</groupId>
            <artifactId>org.osgi.core</artifactId>
            <version>4.3.1</version>
            <scope>provided</scope>
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
                            <Bundle-Activator>com.packt.camel.chapter3.Activator</Bundle-Activator>
                      </instructions>
                </configuration>
          </plugin>
      </plugins>
  </build>

</project>
```

在这个`pom.xml`文件中，我们可以看到：

+   我们依赖于 camel-core（我们选择的版本；这里为 2.12.3）

+   我们也依赖于 osgi-core，因为我们创建了一个 OSGi 包

+   我们使用 maven-bundle 插件创建 OSGi 包（特别是包含 OSGi 头部的 MANIFEST）。在这里，我们提供了一个包激活器（`com.pack.camel.chapter3.Activator`），其详细信息我们将在本章后面看到。

## Prefixer 处理器

我们现在可以创建一个 Prefixer 处理器。这个处理器将获取传入的交换，提取`in`消息体，并在该体前加上*Prefixed*。

处理器是一个简单的类，实现了 Camel 处理器接口。

我们创建`PrefixerProcessor`如下：

```java
package com.packt.camel.chapter3;

import org.apache.camel.Exchange;
import org.apache.camel.Processor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class PrefixerProcessor implements Processor {

  private final static Logger LOGGER = LoggerFactory.getLogger(PrefixerProcessor.class);

  public void process(Exchange exchange) throws Exception {
      String inBody = exchange.getIn().getBody(String.class);
      LOGGER.info("Received in message with body {}", inBody);
      LOGGER.info("Prefixing body ...");
      inBody = "Prefixed " + inBody;
      exchange.getIn().setBody(inBody);
  }

}
```

在这里，我们可以看到处理器如何作为消息翻译器；它转换`in`消息。

## 使用 Java DSL 创建路由

现在是时候使用 Camel Java DSL 创建第一条路由了，使用 `PrefixerProcessor`。为了使用 Camel Java DSL，我们创建一个扩展 Camel `RouteBuilder` 类（`org.apache.camel.RouteBuilder`）的类。这个类在 `configure()` 方法中描述了路由。

我们创建了一个 `MyRouteBuilder` 类，如下所示：

```java
package com.packt.camel.chapter3;

import org.apache.camel.Exchange;
import org.apache.camel.Processor;
import org.apache.camel.builder.RouteBuilder;

public class MyRouteBuilder extends RouteBuilder {

  public void configure() {
      from("timer:fire?period=5000")
              .setBody(constant("Hello Chapter3"))
              .process(new PrefixerProcessor())
              .to("log:MyRoute")
              .process(new Processor() {
                  public void process(Exchange exchange) throws Exception {
                      String body = exchange.getIn().getBody(String.class);
                      if (body.startsWith("Prefixed ")) {
                          body = body.substring("Prefixed ".length());
                          exchange.getIn().setBody(body);
                      }
                  }
              })
              .to("log:MyRoute");
  }

}
```

此路由以计时器开始。计时器每 5 秒创建一个空的 Exchange。

我们使用 `.setBody()` 方法定义了 `in` 消息（此 Exchange 的）体，包含 `Hello Chapter3` 常量字符串。

我们使用 `.process()` 方法调用 `PrefixerProcessor`。正如预期的那样，`PrefixerProcessor` 将 `Prefixed` 追加到 `in` 消息的体中，结果为 `Prefixed Hello Chapter3`。我们可以看到 `PrefixerProcesser` 已经在（`.to("log:MyRoute")`）之后的日志端点正确调用。

除了创建一个用于处理器的专用类之外，还可以创建一个内联处理器。这就是我们使用 `.process(new Processor(){…})` 所做的。我们实现了一个内联处理器，该处理器移除了 `PrefixerProcessor` 追加的前缀。

最后，我们可以看到在最新的日志端点（`.to("log:MyRoute")`）中我们又回到了原始的消息。

`MyRouteBuilder` 类是路由构建器。一个路由必须嵌入到 `CamelContext` 中。这是我们的包激活器的目的；激活器创建 `CamelContext`，构建路由，并在该 `CamelContext` 中注册路由。为了方便，我们还注册了 `CamelContext` 作为 OSGi 服务（它允许我们使用 `camel:* Karaf` 命令查看 `CamelContext` 和路由）。

`Activator` 是一个实现 `BundleActivator` 接口（`org.osgi.framework.BundleActivator`）的类：

```java
package com.packt.camel.chapter3;

import org.apache.camel.CamelContext;
import org.apache.camel.impl.DefaultCamelContext;
import org.osgi.framework.BundleActivator;
import org.osgi.framework.BundleContext;
import org.osgi.framework.ServiceRegistration;

public class Activator implements BundleActivator {

  private CamelContext camelContext;
  private ServiceRegistration serviceRegistration;

  public void start(BundleContext bundleContext) throws Exception {
      camelContext = new DefaultCamelContext();
      camelContext.addRoutes(new MyRouteBuilder());
      serviceRegistration = bundleContext.registerService(CamelContext.class, camelContext, null);
      camelContext.start();
  }

  public void stop(BundleContext bundleContext) throws Exception {
      if (serviceRegistration != null) {
          serviceRegistration.unregister();
      }
      if (camelContext != null) {
          camelContext.stop();
      }
  }

}
```

当包启动时，OSGi 框架会调用 `Activator` 的 `start()` 方法。

在启动时，我们使用 `new DefaultCamelContext()` 创建 `CamelContext`。

我们使用 `camelContext.addRoutes(new MyRouteBuilder())` 在这个 `CamelContext` 中创建和注册我们的路由。

为了方便，我们使用 `bundleContext.registerService(CamelContext.class, camelContext, null)` 注册 `CamelContext` 作为 OSGi 服务。多亏了这项服务注册，我们的 `CamelContext` 和路由将对我们 Apache Karaf OSGi 容器中的 Camel:`*` 命令可见。

最后，我们使用 `camelContext.start()` 启动 Camel Context（以及路由）。

另一方面，当我们停止包时，OSGi 框架将调用激活器的 `stop()` 方法。

在 `stop()` 方法中，我们注销 Camel Context OSGi 服务（使用 `serviceRegistration.unregister()`），并停止 Camel Context（使用 `camelContext.stop()`）。

## 使用 Camel Blueprint DSL 的路由

在同一个包中，我们可以创建另一个 Camel 上下文并设计另一个路由，但这次使用 Camel Blueprint DSL（OSGi 特定）。当使用 Camel Blueprint DSL 时，我们不需要编写像第一个 Camel 上下文那样所有的管道代码。Camel 上下文由 Camel 隐式创建，我们使用 XML 描述在 Camel 上下文中声明路由。

在我们的包的 `OSGI-INF/blueprint` 文件夹中，我们创建 `route.xml`：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="prefixerProcessor" class="com.packt.camel.chapter3.PrefixerProcessor"/>

  <camelContext >
      <route>
          <from uri="timer:fire?period=5000"/>
          <setBody>
              <constant>Hello Chapter3</constant>
          </setBody>
          <process ref="prefixerProcessor"/>
          <to uri="log:blueprintRoute"/>
      </route>
  </camelContext>

</blueprint>
```

在这个 Blueprint 描述符中，我们首先创建 `prefixerProcessor` bean。Blueprint 容器将创建处理器。

Camel Blueprint DSL 提供了 `<camelContext/>` 元素。Camel 将为我们创建 Camel 上下文并注册我们描述的路由。`<route/>` 元素允许我们描述路由。

基本上，路由与上一个非常相似：

+   它从一个定时器开始，每 5 秒创建一个空的 Exchange。

+   它将正文设置为 `Hello Chapter3`。

+   它调用 `prefixerProcessor`。在这种情况下，我们使用已注册的 bean 的引用。`ref="prefixerProcessor"` 对应于 `PrefixerProcessor` bean 的 `id="prefixerProcessor"`。

+   我们还调用一个日志端点。

重要的是要理解，即使我们使用相同的类，我们也有两个不同的 `PrefixerProcessor` 实例：

+   第一个实例在 `Activator` 包中创建，并用于使用 Camel Java DSL 描述的路由

+   第二个实例在 `blueprint` 容器中创建，并用于使用 Camel Blueprint DSL 描述的路由

构建和部署我们的包后，我们现在准备好构建我们的包。

使用 Maven，我们只需运行以下命令：

```java
$ mvn clean install
...
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.489s
[INFO] Finished at: Thu Sep 11 15:14:42 CEST 2014
[INFO] Final Memory: 33M/1188M
[INFO] ------------------------------------------------------------------------

```

我们的可执行包现在可在我们的本地 Maven 仓库中可用（默认情况下，位于 `home` 目录的 `.m2/repository` 文件夹中）。

我们现在可以部署 Karaf OSGi 容器中的包。

在启动 Karaf（例如使用 `bin/karaf`，提供 Karaf 命令行控制台）后，我们首先必须安装 Camel 支持。为此，我们注册 camel 特性仓库并安装 `camel-blueprint` 功能。

`camel-blueprint` 功能为 Camel 核心（因此是 Camel Java DSL，以及所有核心类，如 `CamelContext`、`Processor` 等）和 Camel Blueprint DSL 提供支持。

要注册 Camel 特性仓库，我们使用 Karaf 的 `feature:repo-add` 命令指定我们想要使用的 Camel 版本：

```java
karaf@root()> feature:repo-add camel 2.12.3
Adding feature url mvn:org.apache.camel.karaf/apache-camel/2.12.3/xml/features

```

我们使用 `feature:install` 命令安装 `camel-blueprint` 功能：

```java
karaf@root()> feature:install -v camel-blueprint
Installing feature camel-blueprint 2.12.3
Installing feature camel-core 2.12.3
Installing feature xml-specs-api 2.2.0
Found installed bundle: org.apache.servicemix.specs.activation-api-1.1 [78]
Found installed bundle: org.apache.servicemix.specs.stax-api-1.0 [79]
Found installed bundle: org.apache.servicemix.specs.jaxb-api-2.2 [80]
Found installed bundle: stax2-api [81]
Found installed bundle: woodstox-core-asl [82]
Found installed bundle: org.apache.servicemix.bundles.jaxb-impl [83]
Found installed bundle: org.apache.camel.camel-core [84]
Found installed bundle: org.apache.camel.karaf.camel-karaf-commands [85]
Found installed bundle: org.apache.camel.camel-blueprint [86]

```

现在我们准备好安装和启动我们的包。

对于安装，我们使用 `bundle:install` 命令与在 `pom.xml` 中定义的 Maven 位置：

```java
karaf@root()> bundle:install mvn:com.packt.camel/chapter3/1.0-SNAPSHOT
Bundle ID: 87

```

我们使用 `bundle:start` 命令并使用前一个命令给出的 `Bundle ID` 启动包：

```java
karaf@root()> bundle:start 87

```

日志消息显示我们的路由正在运行（使用 `log:display` 命令）：

```java
karaf@root()> log:display
…
2014-09-11 15:25:45,542 | INFO  | 0 - timer://fire | MyRoute                      | 84 - org.apache.camel.camel-core - 2.12.3 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Prefixed Hello Chapter3]
2014-09-11 15:25:45,543 | INFO  | 0 - timer://fire | MyRoute                      | 84 - org.apache.camel.camel-core - 2.12.3 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello Chapter3]
2014-09-11 15:25:46,253 | INFO  | 2 - timer://fire | PrefixerProcessor              | 87 - com.packt.camel.chapter3 - 1.0.0.SNAPSHOT | Received in message with body Hello Chapter3
2014-09-11 15:25:46,253 | INFO  | 2 - timer://fire | PrefixerProcessor              | 87 - com.packt.camel.chapter3 - 1.0.0.SNAPSHOT | Prefixing body ...
2014-09-11 15:25:46,254 | INFO  | 2 - timer://fire | blueprintRoute               | 84 - org.apache.camel.camel-core - 2.12.3 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Prefixed Hello Chapter3]
2014-09-11 15:25:50,542 | INFO  | 0 - timer://fire | PrefixerProcessor              | 87 - com.packt.camel.chapter3 - 1.0.0.SNAPSHOT | Received in message with body Hello Chapter3
2014-09-11 15:25:50,542 | INFO  | 0 - timer://fire | PrefixerProcessor              | 87 - com.packt.camel.chapter3 - 1.0.0.SNAPSHOT | Prefixing body ...
2014-09-11 15:25:50,543 | INFO  | 0 - timer://fire | MyRoute    | 84 - org.apache.camel.camel-core - 2.12.3 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Prefixed Hello Chapter3]
2014-09-11 15:25:50,543 | INFO  | 0 - timer://fire | MyRoute    | 84 - org.apache.camel.camel-core - 2.12.3 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello Chapter3]
2014-09-11 15:25:51,253 | INFO  | 2 - timer://fire | PrefixerProcessor              | 87 - com.packt.camel.chapter3 - 1.0.0.SNAPSHOT | Received in message with body Hello Chapter3
2014-09-11 15:25:51,254 | INFO  | 2 - timer://fire | PrefixerProcessor              | 87 - com.packt.camel.chapter3 - 1.0.0.SNAPSHOT | Prefixing body ...
2014-09-11 15:25:51,254 | INFO  | 2 - timer://fire | blueprintRoute    | 84 - org.apache.camel.camel-core - 2.12.3 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Prefixed Hello Chapter3]

```

Camel 特性还提供了 Karaf 命令，我们可以使用这些命令查看正在运行的 Camel 上下文和路由。

例如，`camel:context-list`命令显示了可用的 Camel Contexts，如下所示：

```java
karaf@root()> camel:context-list
 Context      Status       Uptime 
 -------      ------       ------ 
 87-camel-4   Started      7 minutes 
 camel-1      Started      7 minutes 

```

在这里，我们可以看到我们在我们的 bundle 中创建的两个 Camel Contexts。

我们可以使用`camel:context-info`命令来获取每个 Camel Context 的详细信息，如下所示：

```java
karaf@root()> camel:context-info camel-1
Camel Context camel-1
 Name: camel-1
 ManagementName: camel-1
 Version: 2.12.3
 Status: Started
 Uptime: 12 minutes

Statistics
 Exchanges Total: 148
 Exchanges Completed: 148
 Exchanges Failed: 0
 Min Processing Time: 0ms
 Max Processing Time: 8ms
 Mean Processing Time: 2ms
 Total Processing Time: 307ms
 Last Processing Time: 2ms
 Delta Processing Time: 0ms
 Load Avg: 0.00, 0.00, 0.00
 Reset Statistics Date: 2014-09-11 15:25:04
 First Exchange Date: 2014-09-11 15:25:05
 Last Exchange Completed Date: 2014-09-11 15:37:20
 Number of running routes: 1
 Number of not running routes: 0

Miscellaneous
 Auto Startup: true
 Starting Routes: false
 Suspended: false
 Shutdown timeout: 300 sec.
 Allow UseOriginalMessage: true
 Message History: true
 Tracing: false

Properties

Advanced
 ClassResolver: org.apache.camel.impl.DefaultClassResolver@a44950b
 PackageScanClassResolver: org.apache.camel.impl.DefaultPackageScanClassResolver@1c950a71
 ApplicationContextClassLoader: org.apache.camel.camel-core [84]

Components
 timer
 log

Dataformats

Routes
 route1

```

我们可以看到`camel-1`上下文包含一个名为`route1`的路由。

实际上，`camel-1`是我们之前在 Activator 中创建的 Camel Context，而`route1`是使用 Camel Java DSL 的路由。在这里，我们能够看到`CamelContext`，这得益于我们在 Activator 中执行的服务注册。

另一方面，我们还有一个名为`87-camel-4`的另一个 Camel Context，如下所示：

```java
karaf@root()> camel:context-info 87-camel-4
Camel Context 87-camel-4
 Name: 87-camel-4
 ManagementName: 87-87-camel-4
 Version: 2.12.3
 Status: Started
 Uptime: 15 minutes

Statistics
 Exchanges Total: 188
 Exchanges Completed: 188
 Exchanges Failed: 0
 Min Processing Time: 0ms
 Max Processing Time: 2ms
 Mean Processing Time: 1ms
 Total Processing Time: 264ms
 Last Processing Time: 1ms
 Delta Processing Time: 0ms
 Load Avg: 0.00, 0.00, 0.00
 Reset Statistics Date: 2014-09-11 15:25:05
 First Exchange Date: 2014-09-11 15:25:06
 Last Exchange Completed Date: 2014-09-11 15:40:41
 Number of running routes: 1
 Number of not running routes: 0

Miscellaneous
 Auto Startup: true
 Starting Routes: false
 Suspended: false
 Shutdown timeout: 300 sec.
 Allow UseOriginalMessage: true
 Message History: true
 Tracing: false

Properties

Advanced
 ClassResolver: org.apache.camel.core.osgi.OsgiClassResolver@60e8b22b
 PackageScanClassResolver: org.apache.camel.core.osgi.OsgiPackageScanClassResolver@4d0956c1
 ApplicationContextClassLoader: BundleDelegatingClassLoader(com.packt.camel.chapter3 [87])

Components
 timer
 properties
 log

Dataformats

Routes
 route2

```

在这个 Camel Context（由 Camel 在 Blueprint 描述符中声明创建的）中，我们可以看到使用 Camel Blueprint DSL 描述的`route2`。

我们也可以使用`camel:route-info`命令来获取路由的详细信息（`camel:route-list`命令显示所有 Camel Contexts 的所有路由列表）：

```java
karaf@root()> camel:route-list
 Context      Route        Status 
 -------      -----        ------ 
 87-camel-4   route2       Started 
 camel-1      route1       Started 

```

我们可以在以下代码中查看`route1`的详细信息：

```java
karaf@root()> camel:route-info route1
Camel Route route1
 Camel Context: camel-1

Properties
 id = route1
 parent = 31b3f7f4
 group = com.packt.camel.chapter3.MyRouteBuilder

Statistics
 Inflight Exchanges: 0
 Exchanges Total: 382
 Exchanges Completed: 382
 Exchanges Failed: 0
 Min Processing Time: 0 ms
 Max Processing Time: 8 ms
 Mean Processing Time: 1 ms
 Total Processing Time: 674 ms
 Last Processing Time: 2 ms
 Delta Processing Time: 0 ms
 Load Avg: 0.00, 0.00, 0.00
 Reset Statistics Date: 2014-09-11 15:25:04
 First Exchange Date: 2014-09-11 15:25:05
 Last Exchange Completed Date: 2014-09-11 15:56:50

Definition
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<route group="com.packt.camel.chapter3.MyRouteBuilder" id="route1" >
 <from uri="timer:fire?period=5000"/>
 <setBody id="setBody1">
 <expressionDefinition>Hello Chapter3</expressionDefinition>
 </setBody>
 <process id="process1"/>
 <to uri="log:MyRoute" id="to1"/>
 <process id="process2"/>
 <to uri="log:MyRoute" id="to2"/>
</route>

```

我们可以看到该路由已成功执行 382 次，没有错误。我们还可以看到来自`MyRouteBuilder`的两个处理器的路由转储。

我们还可以看到与使用 Camel Blueprint DSL 描述的路由对应的`route2`的详细信息：

```java
karaf@root()> camel:route-info route2
Camel Route route2
 Camel Context: 87-camel-4

Properties
 id = route2
 parent = 465796cf

Statistics
 Inflight Exchanges: 0
 Exchanges Total: 414
 Exchanges Completed: 414
 Exchanges Failed: 0
 Min Processing Time: 0 ms
 Max Processing Time: 3 ms
 Mean Processing Time: 1 ms
 Total Processing Time: 529 ms
 Last Processing Time: 1 ms
 Delta Processing Time: 0 ms
 Load Avg: 0.00, 0.00, 0.00
 Reset Statistics Date: 2014-09-11 15:25:05
 First Exchange Date: 2014-09-11 15:25:06
 Last Exchange Completed Date: 2014-09-11 15:59:31

Definition
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<route id="route2" >
 <from uri="timer:fire?period=5000"/>
 <setBody id="setBody2">
 <constant>Hello Chapter3</constant>
 </setBody>
 <process ref="prefixerProcessor" id="process3"/>
 <to uri="log:blueprintRoute" id="to3"/>
</route>

```

# 摘要

处理器是 Camel 中最重要组件之一。它就像瑞士军刀。您可以使用处理器来实现消息翻译和转换，以及任何类型的 EIPs。所有 Camel EIPs 都是通过使用`ProcessorEndpoint`实现 Camel 组件的处理器来实现的。我们稍后将会看到，处理器在错误处理或单元测试中也非常有用。为了使其更加简单，您还可以使用现有的 bean 作为处理器。由于扩展了 bean 支持，Camel 可以直接使用您现有的 bean，这一点我们将在下一章中看到。
