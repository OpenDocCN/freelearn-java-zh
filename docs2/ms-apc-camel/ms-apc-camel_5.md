# 第五章：企业集成模式

在前几章中，我们看到了如何使用处理器或 bean 来实现对消息的行为更改。

然而，其中一些功能提供了实现常见问题解决方案的方法，而不是在不同的路由中重新实现相同的功能，我们可以重用现有的一个。其中一些通用消息操作在 Gregor Hohpe 和 Bobby Woolf 的《企业集成模式》（Enterprise Integration Patterns）中有描述([`www.enterpriseintegrationpatterns.com/`](http://www.enterpriseintegrationpatterns.com/))。

本章将介绍 Camel 提供的最常用的 EIPs：

+   消息系统 EIPs

+   消息通道 EIPs

+   消息构建 EIPs

+   消息路由 EIPs

+   消息转换 EIPs

+   消息端点 EIPs

+   系统管理 EIPs

其中一些通用消息操作在 Gregor Hohpe 和 Bobby Woolf 的《企业集成模式》（Enterprise Integration Patterns）中有描述。它描述了模式，Camel 提供了实现。

# EIP 处理器

EIP 模式的目的在于对消息应用更改或创建新的消息：

+   对消息内容本身的更改

+   对消息目标端点的更改

+   根据消息对路由的更改

+   创建新的消息或交换

在前一章中，我们看到了如何使用 Camel 处理器和 beans 来实现这样的更改。

为了提供对 EIPs 的支持，Camel 实际上提供了可直接使用的处理器，以及 DSL 语言来直接使用这些处理器。

因此，您不必在多个路由中重新实现自己的相同处理器，可以直接使用 Camel 提供的 EIP 处理器。

根据对消息执行的改变和实现的路由功能，EIPs 被分为不同的类别。我们将在以下章节中介绍每个类别。

# 消息系统 EIPs

消息系统 EIPs 收集了所有与消息传递相关的模式，这些模式在路由逻辑中移动。

## 消息通道

消息通道 EIP 是 Camel 路由中端点之间通信的通用名称。

在前几章的示例中，我们使用了以下语法的端点：

```java
component:option?key=value&key=value
```

例如，我们可以有一个如下所示的路由：

```java
<from uri="timer:fire?period=5000"/>
<to uri="log:myLog"/>
```

此路由使用两个端点（`timer`和`log`）。Camel 在两个端点之间隐式创建了一个消息通道。

目的是解耦产生消息的端点与应用消费消息的应用。

此 EIP 实际上在基本上所有路由中隐式使用（您不需要使用特殊符号来使用消息通道，它在 Camel 中）。

## 消息

Camel 中另一个隐含的 EIP 是消息 EIP。

此 EIP 基本上是通过 Camel 消息接口实现的，并封装在交换中。

此 EIP 与消息通道一起使用——消息通道传输消息。

多亏了交换消息包装器，Camel 实现了整个消息 EIP，包括对消息交换模式的支持。

在 Camel 交换中，我们有以下模式属性：

+   如果模式设置为`InOnly`，Camel 将实现一个事件消息（单个入站消息）

+   如果模式设置为`InOut`，Camel 将实现一个带有入站和出站消息的`request-reply`。

Camel 路由的第一个端点（`from`）负责创建交换，因此，带有相应模式的消息。每个端点定义了预期的模式（因此，如果它正在等待出站消息返回给客户端或不是）。

## 管道

Pipeline EIP 的目的是通过消息应用一系列操作。为此，我们将消息通过不同的步骤移动，就像在管道中一样。

我们可以用两种方式用 Camel 定义一个管道：

+   隐式管道是我们之前章节中使用过的。我们只需在路由定义本身中简单地定义步骤。

+   显式管道使用管道 DSL 语法。

### 隐式管道

隐式管道是 Camel 的默认行为——包含不同处理器和端点的路由定义实际上是一个管道。

为了说明这一点，我们创建了一个包含使用 Blueprint DSL 编写的 Camel 路由的示例。

首先，我们创建以下 Maven `pom.xml`文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <groupId>com.packt.camel</groupId>
  <artifactId>chapter5a</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>bundle</packaging>

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

这个`pom.xml`文件非常简单——它只是将我们的路由打包为一个 OSGi 包，我们将将其部署到 Apache Karaf 容器中。在项目中，我们创建了两个非常简单的 bean，它们在接收到`in`消息时只显示一条消息。

第一个 bean 命名为`Step1Bean`：

```java
package com.packt.camel.chapter5a;

public class Step1Bean {

  public static void single(String body) {
      System.out.println("STEP 1: " + body);
  }

}
```

第二个 bean 命名为`Step2Bean`：

```java
package com.packt.camel.chapter5a;

public class Step2Bean {

  public static void single(String body) {
      System.out.println("STEP 2: " + body);
  }

}
```

最后，我们创建描述路由的 Blueprint XML（在`src/main/resources/OSGI-INF/blueprint/route.xml`）：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="step1" class="com.packt.camel.chapter5a.Step1Bean"/>

  <bean id="step2" class="com.packt.camel.chapter5a.Step2Bean"/>

  <camelContext >
      <route>
          <from uri="timer:fire?period=5000"/>
          <setBody>
              <constant>Hello Chapter5a</constant>
          </setBody>
          <bean ref="step1"/>
          <bean ref="step2"/>
          <to uri="log:pipeline"/>
      </route>
  </camelContext>

</blueprint>
```

我们可以看到这里的管道；Camel 将从定时器的端点路由交换到`step1` bean，然后到`step2` bean，最后到`log`端点。

这是一个隐式管道。我们可以通过构建和部署包到 Karaf 中，来看到管道的实际路径。

要构建包含路由和 bean 的包，我们只需这样做：

```java
$ mvn clean install

```

我们可以按照以下方式启动 Karaf 容器：

```java
$ bin/karaf

karaf@root()>

```

我们在 Karaf 中安装`camel-blueprint`支持：

```java
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache- camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

我们现在可以安装我们的包：

```java
karaf@root()> bundle:install -s mvn:com.packt.camel/chapter5a/1.0-SNAPSHOT
Bundle ID: 73

```

很快，我们可以看到路由执行：

```java
STEP 1: Hello Chapter5a
STEP 2: Hello Chapter5a
STEP 1: Hello Chapter5a
STEP 2: Hello Chapter5a
STEP 1: Hello Chapter5a
STEP 2: Hello Chapter5a

```

我们可以注意到管道行为，其中消息从定时器端点流向路由执行的各个步骤。

### 显式管道

使用管道 EIP 的另一种方法是使用相应的 DSL 语法显式定义它。

不同的步骤用`pipeline`关键字定义。

为了说明这一点，我们将创建一个与之前完全相同的路由（由定时器创建的消息发送到两个 bean 和一个日志端点），但这次在路由定义中使用`<pipeline/>`元素。

Maven 的`pom.xml`文件与之前的类似：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <groupId>com.packt.camel</groupId>
  <artifactId>chapter5b</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>bundle</packaging>

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

我们仍然有两个显示`in`消息的 Bean。第一个 Bean 如下：

```java
package com.packt.camel.chapter5b;

public class Step1Bean {

  public static void single(String body) {
      System.out.println("STEP 1: " + body);
  }

}
```

显示`in`消息的第二个 Bean 如下：

```java
package com.packt.camel.chapter5b;

public class Step2Bean {

  public static void single(String body) {
      System.out.println("STEP 2: " + body);
  }

}
```

只有描述路由的 Blueprint XML 不同：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="step1" class="com.packt.camel.chapter5b.Step1Bean"/>

  <bean id="step2" class="com.packt.camel.chapter5b.Step2Bean"/>

  <camelContext >
      <route>
          <from uri="timer:fire?period=5000"/>
          <setBody>
              <constant>Hello Chapter5b</constant>
          </setBody>
          <pipeline>
              <bean ref="step1"/>
              <bean ref="step2"/>
              <to uri="log:pipeline"/>
          </pipeline>
      </route>
  </camelContext>

</blueprint>
```

如前例所示，我们使用 Maven 构建 OSGi bundle：

```java
$ mvn clean install

```

我们在 Karaf 中部署我们的 bundle：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache-camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint
karaf@root()> bundle:install -s mvn:com.packt.camel/chapter5b/1.0-SNAPSHOT
Bundle ID: 73

```

我们可以看到路由执行与上一个示例完全相同：

```java
STEP 1: Hello Chapter5b
STEP 2: Hello Chapter5b
STEP 1: Hello Chapter5b
STEP 2: Hello Chapter5b

```

在 Camel 内部，路由基本上完全相同，只是表示法不同。

在大多数情况下，我们使用隐式管道（默认行为），这允许您简化路由定义。

### 消息路由器

消息路由器 EIP 根据条件将消息移动到不同的目的地。

条件实际上是一个谓词，使用 Camel 支持的语言之一（simple、header、xpath、xquery、mvel、ognl 等）定义的。

谓词可以使用任何数据来实现条件。如果它使用消息的内容本身，我们谈论基于内容的路由器（我们将在本章后面看到）。

为了说明消息路由器 EIP，我们创建了一个路由，该路由将消费文件，并根据文件扩展名将文件复制到不同的输出文件夹。

我们直接使用 Blueprint DSL 将此路由写入`route.xml`文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <camelContext >
      <route>
          <from uri="file:/tmp/in"/>
          <choice>
              <when>
                 <simple>${file:ext} == 'xml'</simple>
                 <to uri="file:/tmp/out/xml"/>
              </when>
              <when>
                 <simple>${file:ext} == 'txt'</simple>
                 <to uri="file:/tmp/out/txt"/>
              </when>
              <otherwise>
                 <to uri="file:/tmp/out/binaries"/>
              </otherwise>
          </choice>
      </route>
  </camelContext>

</blueprint>
```

我们可以看到`<choice/>`元素的使用，这是消息路由器 EIP 的表示法。

在这个选择中，我们定义了两个条件路由：

+   使用`simple`语言，我们定义第一个谓词检查文件扩展名是否为`.xml`。如果是，消息将被路由到文件端点，在`/tmp/out/xml`文件夹中创建输出文件。

+   第二个条件也使用简单语言。这个谓词检查文件扩展名是否为`.txt`。如果是，消息将被路由到文件端点，在`/tmp/out/txt`文件夹中创建输出文件。

如果前两个条件不匹配，消息将被路由到文件端点，在`/tmp/out/binaries`文件夹中创建输出文件。我们启动 Karaf 并安装`camel-blueprint`支持：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache-camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

我们现在可以简单地将在`Karaf`的`deploy`文件夹中放置`route.xml`文件。

在`/tmp/in`文件夹中，我们创建了三个文件。

第一文件是`file.xml`，它包含：

```java
<test>foobar</test>

```

第二个文件是`file.txt`，它包含：

```java
Foobar

```

第三个文件是`file.csv`，它包含：

```java
foo,bar

```

我们可以在`/tmp/out`目录中看到创建了三个文件夹，并且它们包含预期的文件：

```java
/tmp/out$ tree
.
├── binaries
│   └── test.csv
├── txt
│   └── file.txt
└── xml
 └── file.xml

```

## 消息翻译器

消息翻译器 EIP 基本上是消息内容的转换。

路由的一些步骤会改变消息的内容。

在 Camel 中，你有三种方式来实现消息翻译器：

+   您可以使用 transform DSL 表示法调用 Camel 支持的所有语言。

+   如果翻译器的目的是将一种数据格式转换为另一种格式，您可以使用 Camel 提供的序列化/反序列化函数。

+   如果你想要完全控制并实现复杂的转换，你可以使用自己的处理器或 bean 来实现转换逻辑

### 转换表示法

在 `transform` 关键字中，可以使用 Camel 支持的任何语言（`simple`、`ruby`、`groovy` 等）。

外部语言用于对消息进行转换。

为了说明转换表示法的用法，我们可以创建一个使用以下 Blueprint 描述符的 Camel 路由：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <camelContext >
      <route>
          <from uri="file:/tmp/in"/>
          <transform>
             <simple>Hello ${in.body}</simple>
          </transform>
          <to uri="file:/tmp/out"/>
      </route>
  </camelContext>

</blueprint>
```

这个 Camel 路由使用消息翻译 EIP 在 `in` 消息的主体前添加 `Hello`。

路由从 `/tmp/in` 文件夹中消费文件（多亏了 `from` 文件端点），使用简单的语言进行转换表示法，并将消息写入 `/tmp/out` 文件夹中的文件（多亏了 `to` 文件端点）。

我们启动 Karaf 并安装 `camel-blueprint` 功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache-camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

我们只需将 Blueprint XML 文件放入 Karaf 的 `deploy` 文件夹中。我们在 `/tmp/in` 文件夹中创建 `test.txt` 文件，只包含：

```java
World

```

Camel 路由在 `/tmp/out` 文件夹中创建一个 `test.txt` 文件，包含：

```java
Hello World

```

我们可以注意到消息翻译 EIP 改变了 `in` 消息的主体（从 `World` 到 `Hello World`）。

### 使用处理器或 bean

在前面的章节中，我们已经使用 Camel 处理器或 bean 来改变 `in` 消息的主体。

我们使用处理器执行与上一个示例相同的任务。

这次，一个简单的 Blueprint XML 文件是不够的，我们必须将 Blueprint XML 和处理器打包在一个 OSGi 包中。

我们创建以下 Maven `pom.xml` 文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <groupId>com.packt.camel</groupId>
  <artifactId>chapter5e</artifactId>
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

这个 Maven `pom.xml` 文件非常简单，它只定义了 `camel-core` 依赖（由我们的 Camel 处理器所需）和 OSGi 包装。

我们创建一个 `PrependProcessor` 类：

```java
package com.packt.camel.chapter5e;

import org.apache.camel.Exchange;
import org.apache.camel.Processor;

public class PrependProcessor implements Processor {

  public void process(Exchange exchange) throws Exception {
      String inBody = exchange.getIn().getBody(String.class);
      inBody = "Hello " + inBody;
      exchange.getIn().setBody(inBody);
  }

}
```

这个处理器实际上是消息翻译 EIP 的实现——它将 `Hello` 预先添加到传入的消息中。

最后，我们使用这个处理器在一个使用 Blueprint DSL 编写的 Camel 路由中：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="prependProcessor" class="com.packt.camel.chapter5e.PrependProcessor"/>

  <camelContext >
      <route>
          <from uri="file:/tmp/in"/>
          <process ref="prependProcessor"/>
          <to uri="file:/tmp/out"/>
      </route>
  </camelContext>

</blueprint>
```

我们使用 Maven 来构建和打包我们的 OSGi 包：

```java
$ mvn clean install

```

我们启动 Karaf 并安装 `camel-blueprint` 功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache-camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

我们在 Karaf 中安装我们的包：

```java
karaf@root()> bundle:install -s mvn:com.packt.camel/chapter5e/1.0-SNAPSHOT
Bundle ID: 73

```

如前例所示，我们将一个 `test.txt` 文件放入 `/tmp/in` 文件夹中，它包含以下内容：

```java
World

```

然后，我们可以看到 `/tmp/in/test.txt` 包含：

```java
Hello World

```

因此，我们实现了相同的消息翻译 EIP，但这次使用了一个处理器。

处理器或 bean 给你完全控制 Camel 交换，并允许你实现非常复杂的消息转换。

### 序列化/反序列化

而不是改变消息本身的内容，消息翻译 EIP 可以用来将消息从一种数据格式转换为另一种数据格式。

Camel 支持不同的数据格式，并提供函数直接从一种数据格式转换为另一种数据格式。

为了说明序列化和反序列化，我们创建一个消费 XML 文件的路线，并将 XML 消息反序列化/序列化为 JSON 消息，这些消息被发送到另一个文件端点。

我们使用 Camel Blueprint DSL 定义路由：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <camelContext >
      <dataFormats>
          <xmljson id="xmljson"/>
      </dataFormats>
      <route>
          <from uri="file:/tmp/in"/>
          <marshal ref="xmljson"/>
          <to uri="file:/tmp/out"/>
      </route>
  </camelContext>

</blueprint>
```

此路由使用 `xmljson` Camel 数据格式。marshal 元素是消息翻译器 EIP 的实现，它将消息从 XML 转换为 JSON。

我们启动 Karaf 并安装 `camel-blueprint` 和 `camel-xmljson` 功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache-camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint
karaf@root()> feature:install camel-xmljson

```

我们直接将 `route.xml` 文件放入 `deploy` 文件夹。

在 `/tmp/in` 文件夹中，我们创建以下 `person.xml` 文件，包含：

```java
<person>
      <name>jbonofre</name>
      <address>My Street, Paris</address>
</person>
```

在 `/tmp/out` 文件夹中，我们可以看到一个 `person.xml` 文件，包含：

```java
{"name":"jbonofre","address":"My Street, Paris"}
```

我们的消息翻译器 EIP 已经执行，使用 marshalling/unmarshalling 到不同的数据格式。

## 消息端点

消息端点 EIP 仅定义了应用程序如何在路由系统中产生或消费消息的方式。基本上，在 Camel 中，它是通过 `endpoint` 接口直接实现和描述的。一个端点由一个组件创建并由 URI 描述。

# 消息通道 EIP

消息通道 EIP 汇集了所有将数据从一个点移动到另一个点的模式，使用通信通道。

## 点对点通道

点对点通道 EIP 确保只有接收器消费一个消息。

在 Camel 中，这个 EIP 的支持是专门针对组件的。

一些组件被设计用来实现和支持这个 EIP。

例如，这种情况适用于：

+   SEDA 和 VM 组件，用于路由之间的通信

+   当与 JMS 队列一起工作时，JMS 组件

为了说明点对点通道 EIP，我们使用 Camel Blueprint DSL 创建了三个路由：

+   第一条路由从一个计时器开始，在 JMS 队列中产生一个消息

+   第二条和第三条路由从 JMS 队列中消费消息

我们将看到只有一个消息会被一个消费者路由消费。

我们创建以下 `route.xml` 文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="amqConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
      <property name="brokerURL" value="vm://broker"/>
  </bean>

  <camelContext >
      <route>
          <from uri="timer:fire?period=1000"/>
          <setBody>
             <constant>Hello chapter5g</constant>
          </setBody>
          <to uri="jms:queue:input?connectionFactory=#amqConnectionFactory"/>
      </route>
      <route>
          <from uri="jms:queue:input?connectionFactory=#amqConnectionFactory"/>
          <delay>
              <constant>2000</constant>
          </delay>
          <to uri="log:consumer1"/>
      </route>
      <route>
          <from uri="jms:queue:input?connectionFactory=#amqConnectionFactory"/>
          <delay>
              <constant>2000</constant>
          </delay>
          <to uri="log:consumer2"/>
      </route>
  </camelContext>

</blueprint>
```

我们定义了一个嵌入 Apache ActiveMQ JMS 代理的 JMS 连接工厂。此连接工厂用于不同的 Camel JMS 端点。

我们启动一个 Karaf 实例并安装 `camel-blueprint` 和 `activemq-camel` 功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache-camel/2.12.4/xml/features
karaf@root()> feature:repo-add activemq 5.7.0
Adding feature url mvn:org.apache.activemq/activemq-karaf/5.7.0/xml/features
karaf@root()> feature:install camel-blueprint
karaf@root()> feature:install activemq-camel
Refreshing bundles org.apache.servicemix.bundles.jaxb-impl (69), org.apache.camel.camel-core (70)

```

现在，我们可以直接将 `route.xml` 文件放入 Karaf 的 `deploy` 文件夹。在日志 (`$KARAF_HOME/data/log`) 中，我们可以看到：

```java
2014-12-15 15:25:22,142 | INFO  | sConsumer[input] | consumer2  | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5g]
2014-12-15 15:25:23,105 | INFO  | sConsumer[input] | consumer1 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5g]
2014-12-15 15:25:24,146 | INFO  | sConsumer[input] | consumer2 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5g]
2014-12-15 15:25:25,107 | INFO  | sConsumer[input] | consumer1 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5g]
2014-12-15 15:25:26,149 | INFO  | sConsumer[input] | consumer2 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5g]
2014-12-15 15:25:27,109 | INFO  | sConsumer[input] | consumer1 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5g]
2014-12-15 15:25:28,151 | INFO  | sConsumer[input] | consumer2 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5g]

```

我们可以看到每个消息都由一个路由消费，说明了点对点通道 EIP。

## 发布/订阅通道

发布/订阅通道 EIP 与点对点通道 EIP 类似，但不同之处在于，每个消息不是只被一个消费者消费，而是被多个消费者消费。

消息被复制到所有消费者。

与点对点通道 EIP 类似，Camel 在组件级别支持发布/订阅通道。

一些组件被设计用来实现和支持这个 EIP，例如：

+   当与 JMS 主题一起工作时，JMS 组件

+   当端点上的 `multipleConsumers=true` 时，SEDA/VM 组件

为了说明这个 EIP，我们将前面的例子更新为使用主题而不是队列：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="amqConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
      <property name="brokerURL" value="vm://broker"/>
  </bean>

  <camelContext >
      <route>
          <from uri="timer:fire?period=1000"/>
          <setBody>
             <constant>Hello chapter5h</constant>
          </setBody>
          <to uri="jms:topic: input?connectionFactory=#amqConnectionFactory"/>
      </route>
      <route>
          <from uri="jms:topic :input?connectionFactory=#amqConnectionFactory"/>
          <delay>
              <constant>2000</constant>
          </delay>
          <to uri="log:consumer1"/>
      </route>
      <route>
          <from uri="jms:topic:input?connectionFactory=#amqConnectionFactory"/>
          <delay>
              <constant>2000</constant>
          </delay>
          <to uri="log:consumer2"/>
      </route>
  </camelContext>

</blueprint>
```

如前例所示，我们启动 Karaf 并安装 `camel-blueprint` 和 `activemq-camel` 功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache-camel/2.12.4/xml/features
karaf@root()> feature:repo-add activemq 5.7.0
Adding feature url mvn:org.apache.activemq/activemq-karaf/5.7.0/xml/features
karaf@root()> feature:install camel-blueprint
karaf@root()> feature:install activemq-camel
Refreshing bundles org.apache.servicemix.bundles.jaxb-impl (69), org.apache.camel.camel-core (70)

```

我们将`route.xml`文件直接放入`deploy`文件夹。现在，我们可以在日志中看到以下内容：

```java
2014-12-15 15:47:45,363 | INFO  | sConsumer[input] | consumer1 | 70 -org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5h]
2014-12-15 15:47:45,363 | INFO  | sConsumer[input] | consumer2 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5h]
2014-12-15 15:47:47,367 | INFO  | sConsumer[input] | consumer2 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5h]
2014-12-15 15:47:47,367 | INFO  | sConsumer[input] | consumer1 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5h]
2014-12-15 15:47:49,369 | INFO  | sConsumer[input] | consumer1 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5h]
2014-12-15 15:47:49,369 | INFO  | sConsumer[input] | consumer2 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5h]
2014-12-15 15:47:51,371 | INFO  | sConsumer[input] | consumer2  | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5h]
2014-12-15 15:47:51,371 | INFO  | sConsumer[input] | consumer1 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5h]
2014-12-15 15:47:53,373 | INFO  | sConsumer[input] | consumer2 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5h]
2014-12-15 15:47:53,373 | INFO  | sConsumer[input] | consumer1 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5h]

```

我们可以注意到每个消息都被两个路由消费了（见时间戳）。

## 死信通道

死信通道 EIP 允许你在实际目的地交付失败时将消息重新路由到另一个目的地。

此 EIP 与 Camel 路由中的错误管理相关。

Camel 通过不同的错误处理程序和政策提供了广泛的支持，我们将看到错误处理程序，因此，死信通道 EIP 将在第七章 *错误处理*中介绍。

## 保证交付

保证交付确保我们不会丢失任何消息。这意味着基本上消息是持久的，并存储在持久存储中。

它允许你在路由中创建一些检查点——如果路由停止，消息将被存储。一旦路由重新启动，*挂起*的消息将被处理。

Camel 本身不提供消息存储，但你可以使用允许存储消息的端点，如下所示：

+   文件端点，在这里消息由路由作为文件在文件系统中生成，并由路由消费。存储实际上就是文件系统。

+   JMS 端点，在这里消息（标记为持久消息）由路由添加到 JMS 队列中，并由其他路由消费。消息存储实际上就是代理的持久消息存储。

+   JPA 端点，在这里消息被生成并存储在数据库中，其他路由轮询数据库。消息存储实际上就是数据库。

我们可以使用两个共享文件系统目录的路由来展示这个 EIP，以存储消息。

目的是确保，即使第二条路由被停止，消息仍然是持久的，并且一旦重新启动，路由就会立即处理这些消息。

我们创建一个名为`route1.xml`的第一个文件，包含：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <camelContext >
      <route>
          <from uri="timer:fire?period=1000"/>
          <setBody>
             <constant>Hello chapter5i</constant>
          </setBody>
          <to uri="file:/tmp/exchange"/>
      </route>
  </camelContext>

</blueprint>
```

我们启动 Karaf 并安装`camel-blueprint`功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache-camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

我们将`route1.xml`文件放入`deploy`文件夹。我们可以看到第一个文件进入`/tmp/exchange`文件夹：

```java
$ /tmp/exchange$ ls -l
total 16
-rw-rw-r-- 1 jbonofre jbonofre 15 Dec 15 16:21 ID-latitude-51643-1418656861977-0-1
-rw-rw-r-- 1 jbonofre jbonofre 15 Dec 15 16:21 ID-latitude-51643-1418656861977-0-3
-rw-rw-r-- 1 jbonofre jbonofre 15 Dec 15 16:21 ID-latitude-51643-1418656861977-0-5
-rw-rw-r-- 1 jbonofre jbonofre 15 Dec 15 16:21 ID-latitude-51643-1418656861977-0-7

```

现在，我们创建一个`route2.xml`文件，包含：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <camelContext >
      <route>
          <from uri="file:/tmp/exchange"/>
            <convertBodyTo type="java.lang.String"/>
          <to uri="log:route2"/>
      </route>
  </camelContext>

</blueprint>
```

我们将此`route2.xml`文件放入 Karaf 的`deploy`文件夹。

现在，我们可以在 Karaf 日志中看到以下内容：

```java
2014-12-15 17:04:11,258 | INFO  | :///tmp/exchange | route2 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5i]
2014-12-15 17:04:11,260 | INFO  | :///tmp/exchange | route2 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5i]
2014-12-15 17:04:11,260 | INFO  | :///tmp/exchange | route2 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5i]
2014-12-15 17:04:11,261 | INFO  | :///tmp/exchange | route2 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5i]
2014-12-15 17:04:11,262 | INFO  | :///tmp/exchange | route2 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5i]
2014-12-15 17:04:11,263 | INFO  | :///tmp/exchange | route2 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5i]
2014-12-15 17:04:11,263 | INFO  | :///tmp/exchange | route2 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5i]

```

因此，即使第二条路由没有部署，消息也不会丢失，并存储在文件系统中。这是保证交付 EIP 的一个实现。

## 消息总线

消息总线 EIP 描述了将应用程序插入并播放的架构，这些应用程序必须交互。此 EIP 汇集了消息基础设施，以及实现路由所需的其他层。

因此，基本上，Camel 本身是实现消息总线 EIP 的。

# 消息构造 EIPs

这些 EIP 负责根据其他消息创建消息。

## 事件消息 EIP

事件消息 EIP 描述了如何使用消息传递将事件从一个应用程序传输到另一个应用程序。

Camel 通过在交换中使用消息交换模式来支持这个 EIP。当定义为`InOnly`时，意味着我们处理的是一个单向事件消息。

因此，基本上，事件消息 EIP 意味着单向消息。

路由的第一个端点定义了期望的交换模式，但在路由的任何点上，你可以强制交换模式为`InOnly`，使其充当事件消息 EIP。

为了做到这一点，你必须使用`inOnly`表示法：

```java
<route>
   <from uri="direct:start"/>
   <inOnly uri="bean:myBean"/>
</route>
```

你也可以使用`setExchangePattern`表示法：

```java
<route>
   <from uri="direct:start"/>
   <setExchangePattern pattern="InOnly"/>
   <to uri="bean:myBean"/>
</route>
```

也可以将模式定义为端点的属性：

```java
<route>
   <from uri="direct:start"/>
   <to uri="bean:myBean" pattern="InOnly"/>
</route>
```

## 请求/响应 EIP

请求/响应 EIP 类似于事件消息 EIP，但这次，期望从目标应用程序得到响应。

与事件消息一样，Camel 通过使用定义为`InOut`的消息交换模式来支持这个 EIP。

再次强调，`from`端点定义了它期望的模式。例如，一个 CXF 端点将定义模式为`InOut`，因为它必须向客户端返回某些内容。

在`InOnly`模式中，你可以使用相同的表示法*强制*模式为`InOut`。

## 关联标识符 EIP

当与请求/响应模式一起使用时，关联标识符 EIP 非常有用。使用这个模式，可以给消息添加一个标识符，该标识符可以用来关联响应消息和请求消息。

Camel 通过在消息中定义一个专用头或在交换中定义一个属性来支持这个 EIP。这个头（或属性）实际上是关联标识符。

一些其他 EIP（我们将在后面看到）利用这个头关联多个消息。例如，Splitter EIP 将关联标识符定义为拆分结果交换的属性（例如，以便能够聚合消息）。

## 返回地址 EIP

返回地址 EIP 描述了目标端点如何知道它需要将响应发送到何处。这个 EIP 必须与请求/响应模式一起使用，因为我们期望从目标端点得到响应。

在涉及 JMS 端点的情况下，Camel 通过在消息中填充`JMSReplyTo`头来支持这个 EIP。

当使用 JMS 组件时，这个`JMSReplyTo`头被直接使用并由代理传输。

还可以在 JMS 端点上使用`ReplyTo`选项来动态填充`JMSReplyTo`头：

```java
<to uri="jms:queue:request?replyTo=response"/>
```

# 消息路由

这些 EIP 是核心路由模式。这通常是 Camel 提供特定语法来处理路由的地方。

## 基于内容路由器 EIP

基于内容路由器 EIP 是消息路由器 EIP 的一个特例。

如我们之前看到的，消息路由器 EIP 是一个通用的路由 EIP，它定义了基于消息的条件路由。

基于内容路由器 EIP 用于条件基于消息体内容的情况。在消息路由器示例中，条件是消费文件的扩展名。

这里，为了说明基于内容的路由器 EIP，我们创建了一个示例，该示例将根据消息体中的`XPath`谓词来路由文件。

我们创建以下`route.xml`蓝图描述：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <camelContext >
      <route>
          <from uri="file:/tmp/in"/>
          <choice>
              <when>
                 <xpath>//address='France'</xpath>
                 <to uri="file:/tmp/out/france"/>
              </when>
              <when>
                 <xpath>//address='USA'</xpath>
                 <to uri="file:/tmp/out/usa"/>
              </when>
              <otherwise>
                 <to uri="file:/tmp/out/others"/>
              </otherwise>
  </choice>
      </route>
  </camelContext>

</blueprint>
```

该路由从`/tmp/in`文件夹中消费文件。消息体包含文件内容。我们使用`XPath`谓词来测试消息中的`address`元素，如下所示：

+   如果地址是法国，消息将被路由到`/tmp/out/france`文件夹。

+   如果地址是 USA，消息将被路由到`/tmp/out/usa`文件夹。

+   如果地址不是法国或 USA，消息将被路由到`/tmp/out/others`文件夹。

我们启动 Karaf 并安装`camel-blueprint`功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache-camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

我们将`route.xml`放入 Karaf 的`deploy`文件夹。在`/tmp/in`文件夹中，我们创建包含以下内容的`first.xml`文件：

```java
<person>
<name>jbonofre</name>
<address>France</address>
</person>
```

我们还丢弃了包含以下内容的`second.xml`文件：

```java
<person>
<name>Bob</name>
<address>USA</address>
</person>
```

我们还丢弃包含以下内容的`third.xml`文件：

```java
<person>
<name>Juan</name>
<address>Spain</address>
</person>
```

我们可以看到文件已按预期路由到不同的文件夹：

```java
/tmp/out$ tree
.
├── france
│ └── first.xml
├── others
│ └── third.xml
└── usa
└── second.xml
```

## 消息过滤器 EIP

消息过滤器 EIP 描述了如何仅选择我们想要处理的消息。

我们定义一个谓词来匹配和处理消息。如果消息不匹配，它们的谓词将被忽略。

与消息路由器 EIP 一样，我们可以使用 Camel 支持的所有语言来编写谓词。

为了说明消息过滤器 EIP，我们使用以下`route.xml`蓝图描述符：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <camelContext >
      <route>
          <from uri="file:/tmp/in"/>
          <filter>
               <xpath>//name='jbonofre'</xpath>
               <to uri="direct:next"/>
          </filter>
      </route>
      <route>
          <from uri="direct:next"/>
          <convertBodyTo type="java.lang.String"/>
          <to uri="log:file"/>
      </route>
  </camelContext>

</blueprint>
```

第一条路由从`/tmp/in`文件夹中消费文件。如果文件包含名为`jbonofre`的元素，则消息将被移动到第二条路由（归功于`direct`端点）。

如果`XPath`谓词不匹配，消息将被忽略。

我们启动 Karaf 并安装`camel-blueprint`功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache- camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

在`/tmp/in`文件夹中，我们创建了与上一个示例相同的三个文件。

包含以下内容的`first.xml`文件：

```java
<person>
<name>jbonofre</name>
<address>France</address>
</person>
```

包含以下内容的`second.xml`文件：

```java
<person>
<name>Bob</name>
<address>USA</address>
</person>
```

包含以下内容的`third.xml`文件：

```java
<person>
<name>Juan</name>
<address>Spain</address>
</person>
```

在 Karaf 的`log`文件中，我们可以看到：

```java
2014-12-15 18:12:10,944 | INFO  | - file:///tmp/in | file | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body:
<person><name>jbonofre</name><address>France</address></person>]

```

这意味着只有来自`first.xml`文件的消息被处理。

## 动态路由器 EIP

动态路由器 EIP 描述了如何动态路由消息。当使用消息路由器 EIP 时，不同的路由目的地和条件在设计时静态定义。

使用动态路由器 EIP，条件和目的地在运行时评估，因此可以动态更改。

在一个条件下，也可以将消息发送到多个目的地。

为了说明这个 EIP，我们创建了一个使用动态路由器的路由。动态路由器使用一个 bean 来评估条件并定义路由目的地。

我们创建一个非常简单的 Maven `pom.xml`文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <groupId>com.packt.camel</groupId>
  <artifactId>chapter5l</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>bundle</packaging>

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

这个 Maven `pom.xml`文件只是将路由和 bean 打包成一个 OSGi 包。

我们创建`DynamicRouterBean`类：

```java
package com.packt.camel.chapter5l;

import java.util.Random;

public class DynamicRouterBean {

  public String slip(String body) {
      Random random = new Random();
      int value = random.nextInt(1000);
      if (value >= 500) {
          return "direct:large";
      } else {
          return "direct:small";
      }
  }

}
```

这个豆子随机且动态地将消息路由到两个端点——如果生成的随机数大于 `500`，则消息被路由到 `direct:large` 端点，否则，消息被路由到 `direct:small` 端点。

我们现在使用 Blueprint 描述符创建路由：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="dynamicRouterBean" class="com.packt.camel.chapter5l.DynamicRouterBean"/>

  <camelContext >
      <route>
          <from uri="timer:fire?period=5000"/>
          <setBody>
              <constant>Hello chapter5l</constant>
          </setBody>
          <dynamicRouter>
              <method ref="dynamicRouterBean" method="slip"/>
          </dynamicRouter>
      </route>
      <route>
          <from uri="direct:large"/>
          <to uri="log:large"/>
      </route>
      <route>
          <from uri="direct:small"/>
          <to uri="log:small"/>
      </route>
  </camelContext>

</blueprint>
```

此路由使用 `dynamicRouter` 中的豆子。我们创建了两个路由，对应于动态路由的目标端点。

我们使用以下方式构建我们的 OSGi 捆绑包：

```java
$ mvn clean install

```

我们捆绑的现在已准备好在 Karaf 中部署。

我们启动 Karaf 并安装 `camel-blueprint` 功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache- camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

我们部署并启动我们的捆绑包：

```java
karaf@root()> bundle:install -s mvn:com.packt.camel/chapter5l/1.0- SNAPSHOT
Bundle ID: 73

```

在 Karaf 的 `log` 文件中，我们可以看到：

```java
2014-12-15 18:45:48,518 | INFO  | 1 - timer://fire | large | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5l]
2014-12-15 18:45:48,518 | INFO  | 1 - timer://fire | small | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5l]
2014-12-15 18:45:48,518 | INFO  | 1 - timer://fire | large | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5l]
2014-12-15 18:45:48,519 | INFO  | 1 - timer://fire | small | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5l]
2014-12-15 18:45:48,519 | INFO  | 1 - timer://fire | large | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5l]
2014-12-15 18:45:48,519 | INFO  | 1 - timer://fire | large | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5l]
2014-12-15 18:45:48,519 | INFO  | 1 - timer://fire | large | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5l]

```

## Multicast 和 Recipient List EIPs

Recipient List EIP 描述了如何将相同的消息发送到多个目的地。

我们有两种接收者列表：

+   当目的地在设计时静态定义时，我们谈论静态接收者列表或多播。

+   当目的地在运行时动态定义时，我们谈论动态接收者列表

### Multicast EIP

让我们从 Multicast EIP 的第一个例子（或静态接收者列表）开始。

我们创建了以下 `route.xml` Blueprint 描述符：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <camelContext >
      <route>
          <from uri="timer:first?period=5000"/>
          <setBody><constant>Hello chapter5m</constant></setBody>
          <multicast>
              <to uri="direct:france"/>
              <to uri="direct:usa"/>
              <to uri="direct:spain"/>
          </multicast>
      </route>
      <route>
          <from uri="direct:france"/>
          <to uri="log:france"/>
      </route>
      <route>
          <from uri="direct:usa"/>
          <to uri="log:usa"/>
      </route>
      <route>
          <from uri="direct:spain"/>
          <to uri="log:spain"/>
      </route>
  </camelContext>

</blueprint>
```

第一个路由每 5 秒创建一个 `Hello chapter5m` 消息。此消息被发送到三个目的地，`france`、`usa` 和 `spain`。

我们启动 Karaf 并安装 `camel-blueprint` 功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache- camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

我们将 `route.xml` 文件放入 Karaf 的 `deploy` 文件夹。

在 Karaf 的 `log` 文件中，我们可以看到：

```java
2014-12-15 18:59:17,198 | INFO  |  - timer://first | france | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5m]
2014-12-15 18:59:17,199 | INFO  |  - timer://first | usa | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5m]
2014-12-15 18:59:17,200 | INFO  |  - timer://first | spain | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5m]
2014-12-15 18:59:22,180 | INFO  |  - timer://first | france | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5m]
2014-12-15 18:59:22,182 | INFO  |  - timer://first | usa | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5m]
2014-12-15 18:59:22,183 | INFO  |  - timer://first | spain | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5m]

```

我们可以看到每条消息都已发送到三个目的地。

### Recipient List EIP

为了说明动态接收者列表，我们创建了一个使用豆子动态定义目标目的地的路由。

我们创建了一个简单的 Maven `pom.xml` 文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <groupId>com.packt.camel</groupId>
  <artifactId>chapter5n</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>bundle</packaging>

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

这个 Maven `pom.xml` 文件只是将我们的路由和豆子打包成一个 OSGi 捆绑包。

我们创建了一个简单的 `RouterBean` 类，该类随机更改接收者列表：

```java
package com.packt.camel.chapter5n;

import java.util.Random;

public class RouterBean {

  public String populate(String body) {
      Random random = new Random();
      int value = random.nextInt(1000);
      if (value >= 500) {
          return "direct:one,direct:two,direct:three";
      } else {
          return "direct:one,direct:two";
      }
  }
}
```

如果随机整数大于 `500`，则消息被路由到 `direct:one`、`direct:two` 和 `direct:three` 端点。否则，消息仅被路由到 `direct:one` 和 `direct:two` 端点。

最后，我们使用这个豆子来填充路由中的头信息。这个头信息被接收者列表使用：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="routerBean" class="com.packt.camel.chapter5n.RouterBean"/>

  <camelContext >
      <route>
          <from uri="timer:fire?period=5000"/>
          <setBody>
              <constant>Hello chapter5n</constant>
          </setBody>
          <setHeader headerName="recipientList">
              <method bean="routerBean" method="populate"/>
          </setHeader>
          <recipientList delimiter=",">
              <header>recipientList</header>
          </recipientList>
      </route>
      <route>
          <from uri="direct:one"/>
          <to uri="log:one"/>
      </route>
      <route>
          <from uri="direct:two"/>
          <to uri="log:two"/>
      </route>
      <route>
          <from uri="direct:three"/>
          <to uri="log:three"/>
      </route>
  </camelContext>

</blueprint>
```

我们构建我们的捆绑包：

```java
$ mvn clean install

```

我们的捆绑包现在可以部署到 Karaf 中。

我们启动 Karaf 并安装 `camel-blueprint` 功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache- camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

我们安装我们的捆绑包：

```java
karaf@root()> bundle:install -s mvn:com.packt.camel/chapter5n/1.0- SNAPSHOT
Bundle ID: 73

```

在 Karaf 的 `log` 文件中，我们可以看到：

```java
2014-12-15 19:12:32,975 | INFO  | 1 - timer://fire | one | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5n]
2014-12-15 19:12:32,976 | INFO  | 1 - timer://fire | two | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5n]
2014-12-15 19:12:37,949 | INFO  | 1 - timer://fire | one | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5n]
2014-12-15 19:12:37,950 | INFO  | 1 - timer://fire | two | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5n]
2014-12-15 19:12:37,950 | INFO  | 1 - timer://fire | three | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5n]

```

我们可以看到目的地根据随机整数的计算结果动态变化。

## Splitter 和 Aggregator EIPs

这些 EIP 负责将大消息分割成块，或将小块聚合成一个完整的消息。

### Splitter EIP

Splitter EIP 描述了如何将大消息分割成多个块，并单独处理。

Camel 支持这个 EIP，允许你使用任何支持的编程语言或处理器/豆子来定义分割逻辑。

为了说明 Splitter EIP，我们创建了以下 `route.xml` Blueprint 描述符：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <camelContext >
      <route>
          <from uri="file:/tmp/in"/>
          <split>
              <xpath>//person</xpath>
              <to uri="log:chunk"/>
          </split>
      </route>
  </camelContext>

</blueprint>
```

此路由从`/tmp/in`文件夹中的文件消费，并使用`XPath`拆分内容。

我们启动 Karaf 并安装`camel-blueprint`功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
karaf@root()> feature:install camel-blueprint

```

我们将`route.xml`放入 Karaf 的`deploy`文件夹。

在`/tmp/in`文件夹中，我们创建了以下`persons.xml`文件：

```java
<persons>
   <person>
    <name>jbonofre</name>
    <address>France</address>
   </person>
   <person>
    <name>Bob</name>
    <address>USA</address>
   </person>
   <person>
    <name>Juan</name>
    <address>Spain</address>
   </person>
</persons>
```

在 Karaf 的`log`文件中，我们可以看到：

```java
2014-12-15 19:30:35,624 | INFO  | - file:///tmp/in | chunk | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: org.apache.xerces.dom.DeferredElementNSImpl, Body: <person> <name>jbonofre</name> <address>France</address> </person>]
2014-12-15 19:30:35,624 | INFO  | - file:///tmp/in | chunk | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: org.apache.xerces.dom.DeferredElementNSImpl, Body: <person> <name>Bob</name> <address>USA</address>   </person>]
2014-12-15 19:30:35,625 | INFO  | - file:///tmp/in | chunk | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: org.apache.xerces.dom.DeferredElementNSImpl, Body: <person>    <name>Juan</name>    <address>Spain</address> </person>]

```

我们可以看到大文件已经被拆分成小消息。

### 注意

Camel 支持多种拆分策略（使用语言、令牌、自定义 bean 等）。

### 聚合器

聚合器 EIP 是拆分器 EIP 的完全相反——我们接收多个小消息，我们希望将它们聚合成一个大的消息。

Camel 支持此 EIP。您必须向聚合器提供两件事：

+   一个实现 Camel `AggregationStrategy`的 bean，该 bean 定义了如何将新消息与先前聚合的消息聚合在一起（*消息增长*）

+   聚合完成，它定义了我们何时认为聚合已完成

完成以下任务有多种不同的选择：

+   `completionTimeout`是一个不活动超时。如果在超时后没有新的交换进入聚合器，则认为聚合已完成。

+   `completionInterval`考虑在给定时间后聚合已完成。

+   `completionSize`是静态的交换次数，用于聚合。

+   `completionPredicate`是最先进的。当谓词为真时，认为聚合已完成。

为了说明聚合器 EIP，我们创建了一个路由，该路由聚合了一定数量的静态消息。

我们将用于聚合策略的 bean（以及路由定义）打包成一个 OSGi 包。

我们创建了以下 Maven `pom.xml`：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <groupId>com.packt.camel</groupId>
  <artifactId>chapter5p</artifactId>
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

我们创建了`StringAggregator`类，该类实现了一个字符串聚合策略：

```java
package com.packt.camel.chapter5p;

import org.apache.camel.Exchange;
import org.apache.camel.processor.aggregate.AggregationStrategy;

public class StringAggregator implements AggregationStrategy {

  public Exchange aggregate(Exchange oldExchange, Exchange newExchange) {
      if (oldExchange == null) {
          return newExchange;
      }
      String oldBody = oldExchange.getIn().getBody(String.class);
      String newBody = newExchange.getIn().getBody(String.class);
      oldExchange.getIn().setBody(oldBody + "+" + newBody);
      return oldExchange;
  }

}
```

我们使用 Blueprint DSL 创建路由，其中包含我们的`StringAggregator`聚合器和`completionSize`类来创建路由：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="aggregator" class="com.packt.camel.chapter5p.StringAggregator"/>

  <camelContext >
      <route>
          <from uri="timer:fire?period=5000"/>
          <setBody>
              <constant>Hello chapter5p</constant>
          </setBody>
          <setHeader headerName="id">
              <constant>same</constant>
          </setHeader>
          <aggregate strategyRef="aggregator" completionSize="5">
              <correlationExpression>
                  <simple>header.id</simple>
              </correlationExpression>
              <to uri="log:aggregated"/>
          </aggregate>
      </route>
  </camelContext>

</blueprint>
```

消息相关性（用于识别我们是否在同一个聚合单元中）使用头部 ID 定义。此路由将五个消息一起聚合。

我们构建我们的 OSGi 包：

```java
$ mvn clean install

```

我们的包已准备好在 Karaf 中部署。

我们启动 Karaf 并安装`camel-blueprint`功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache- camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

我们安装并启动我们的包：

```java
karaf@root()> bundle:install -s mvn:com.packt.camel/chapter5p/1.0- SNAPSHOT
Bundle ID: 73

```

在 Karaf 的`log`文件中，我们可以看到：

```java
2014-12-15 21:07:22,386 | INFO  | 1 - timer://fire | aggregated | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5p+Hello chapter5p+Hello chapter5p+Hello chapter5p+Hello chapter5p]
2014-12-15 21:07:47,380 | INFO  | 1 - timer://fire | aggregated | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5p+Hello chapter5p+Hello chapter5p+Hello chapter5p+Hello chapter5p]

```

我们可以看到我们的路由聚合了 5 条`Hello chapter5p`消息。

## 重排序器 EIP

重排序器 EIP 描述了如何排序消息的处理。它使用比较器来定义消息的顺序。

Camel 使用一个表达式来创建比较器。这意味着比较器可以使用消息体、头部等信息。您在 Camel 重排序符号中定义表达式。

为了说明重排序器 EIP，我们使用了以下`route.xml` Blueprint 描述符：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <camelContext >
      <route>
          <from uri="timer:first?period=2000"/>
          <setBody><constant>one</constant></setBody>
          <to uri="direct:resequencer"/>
      </route
      <route>
          <from uri="timer:second?period=2000"/>
          <setBody><constant>two</constant></setBody>
          <to uri="direct:resequencer"/>
      </route>
      <route>
           <from uri="direct:resequencer"/>
           <resequence>
              <simple>body</simple>
              <to uri="log:requencer"/>
           </resequence>
      </route>
  </camelContext>

</blueprint>
```

两个路由每 2 秒生成一条消息。两个路由都将消息发送到重新排序路由。重新排序路由使用消息体的字符串比较器，以保证相同的处理顺序，`one` 先，`two` 后。

我们启动 Karaf 并安装 `camel-blueprint` 功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache-camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

我们将 `route.xml` 文件放入 Karaf 的 `deploy` 文件夹。

在 Karaf 的 `log` 文件中，我们可以看到：

```java
2014-12-15 21:22:16,769 | INFO  | 0 - Batch Sender | requencer | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: one]
2014-12-15 21:22:16,770 | INFO  | 0 - Batch Sender | requencer | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: two]
2014-12-15 21:22:18,746 | INFO  | 0 - Batch Sender | requencer | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: one]
2014-12-15 21:22:18,747 | INFO  | 0 - Batch Sender | requencer | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: two]

```

我们可以看到，消息总是按照相同的顺序处理，`one` 先，`two` 后。

## 组合消息处理器 EIP

组合消息处理器 EIP 是分割器 EIP 和聚合器 EIP 的组合。其目的是：

+   将大消息分割成块消息

+   独立处理每个块

+   将每个块响应重新聚合为一个大消息

Camel 以两种方式支持此 EIP：

+   使用分割器和聚合器 EIP 的纯组合

+   仅使用分割器

后者是使用起来最简单的。它允许您直接在分割器上定义聚合策略。聚合完成由分割器定义，因为它知道它创建了多少个块。

我们用一个例子来说明基于分割器的组合消息处理器 EIP，该例子使用 `XPath` 分割 `persons.xml` 文件，逐个处理每个人，并将结果消息重新聚合为一个大的消息。

我们创建以下 Maven `pom.xml` 文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <groupId>com.packt.camel</groupId>
  <artifactId>chapter5r</artifactId>
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

我们创建一个自定义聚合策略，`MyAggregator`，它直接使用消息字符串：

```java
package com.packt.camel.chapter5r;

import org.apache.camel.Exchange;
import org.apache.camel.processor.aggregate.AggregationStrategy;

public class MyAggregator implements AggregationStrategy {

  public Exchange aggregate(Exchange oldExchange, Exchange   newExchange) {

      if (oldExchange == null) {
          return newExchange;
      }

      String persons = oldExchange.getIn().getBody(String.class);
      String newPerson = newExchange.getIn().getBody(String.class);

      // put orders together separating by semi colon
      persons = persons + newPerson;
      oldExchange.getIn().setBody(persons);

      return oldExchange;
  }

}
```

我们使用 Blueprint DSL 创建一个路由：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="aggregator" class="com.packt.camel.chapter5r.MyAggregator"/>

  <camelContext >
      <route>
          <from uri="file:/tmp/in"/>
          <split strategyRef="aggregator">
              <xpath>//person</xpath>
              <to uri="log:person"/>
          </split>
          <to uri="log:persons"/>
      </route>
  </camelContext>

</blueprint>
```

此路由从 `/tmp/in` 文件夹中消费文件，使用 `XPath` 表达式分割消息，然后使用 `MyAggregator` 策略重新聚合。

我们现在可以编译我们的包：

```java
$ mvn clean install

```

我们启动 Karaf 并安装 `camel-blueprint` 功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache- camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint
```

我们安装并启动我们的包：

```java
karaf@root()> bundle:install -s mvn:com.packt.camel/chapter5r/1.0- SNAPSHOT
Bundle ID: 73

```

在 Karaf 的 `log` 文件中，我们可以看到：

```java
2014-12-15 21:42:38,803 | INFO  | - file:///tmp/in | person | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: org.apache.xerces.dom.DeferredElementNSImpl, Body: <person><name>jbonofre</name><address>France</address></person>]
2014-12-15 21:42:38,804 | INFO  | - file:///tmp/in | person | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: org.apache.xerces.dom.DeferredElementNSImpl, Body: <person><name>Bob</name><address>USA</address></person>]
2014-12-15 21:42:38,806 | INFO  | - file:///tmp/in | person | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: org.apache.xerces.dom.DeferredElementNSImpl, Body: <person><name>Juan</name><address>Spain</address></person>]
2014-12-15 21:42:38,806 | INFO  | - file:///tmp/in | persons | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: <person><name>jbonofre</name><address>France</address></person><perso n><name>Bob</name><address>USA</address></person><person><name>Juan</ name><address>Spain</address></person>]

```

我们可以看到，分割器将每个已单独处理的人隔离开来。分割后，消息再次被重新聚合，正如我们在最新的 `log` 消息中看到的那样。

## 散列-收集 EIP

散列-收集 EIP 与组合消息处理器 EIP 类似，但不是分割和聚合，我们首先使用一个接收者列表（静态或动态）和一个聚合器，响应来自不同的接收者。

Camel 通过结合接收者列表/多播和聚合支持此 EIP。

## 路由条 EIP

路由条 EIP 描述了如何动态定义消息的处理步骤。

在 Camel 路由中，路由步骤是静态定义的；它是路由本身。然而，您可以使用 `routingSlip` 语法在运行时定义路由的下一步。

它就像一个动态接收者列表，但处理不是并行的，而是顺序的。

为了说明这个 EIP，我们创建一个使用 bean 定位包含下一步处理步骤的头的路由。

我们创建以下 Maven `pom.xml` 文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <groupId>com.packt.camel</groupId>
  <artifactId>chapter5s</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>bundle</packaging>

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

我们创建了以下`RoutingSlipBean`类，它随机定义路由的下一步：

```java
package com.packt.camel.chapter5s;

import java.util.Random;

public class RoutingSlipBean {

  public String nextSteps(String body) {
      Random random = new Random();
      int value = random.nextInt(1000);
      if (value >= 500) {
          return "direct:one,direct:two";
      } else {
          return "direct:one";
      }
  }

}
```

我们使用这个 Bean 在用 Blueprint DSL 编写的 Camel 路由中定义一个`slip`头。这个头由`routingslip`使用：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="routingSlipBean" class="com.packt.camel.chapter5s.RoutingSlipBean"/>

  <camelContext >
      <route>
          <from uri="timer:fire?period=5000"/>
          <setBody>
              <constant>Hello chapter5s</constant>
          </setBody>
          <setHeader headerName="slip">
              <method bean="routingSlipBean" method="nextSteps"/>
          </setHeader>
          <routingSlip uriDelimiter=",">
              <header>slip</header>
          </routingSlip>
      </route>
      <route>
          <from uri="direct:one"/>
          <to uri="log:one"/>
      </route>
      <route>
          <from uri="direct:two"/>
          <to uri="log:two"/>
      </route>
  </camelContext>

</blueprint>
```

我们现在构建我们的 OSGi 包：

```java
$ mvn clean install

```

我们的包已准备好在 Karaf 中部署。

我们启动 Karaf 并安装`camel-blueprint`功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache- camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

我们安装并启动我们的包：

```java
karaf@root()> bundle:install -s mvn:com.packt.camel/chapter5s/1.0- SNAPSHOT
Bundle ID: 73

```

在 Karaf 的`log`文件中，我们可以看到：

```java
2014-12-15 22:02:19,110 | INFO  | 1 - timer://fire | one | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5s]
2014-12-15 22:02:19,111 | INFO  | 1 - timer://fire | two | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5s]
2014-12-15 22:02:24,090 | INFO  | 1 - timer://fire | one | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5s]
2014-12-15 22:02:29,090 | INFO  | 1 - timer://fire | one | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5s]
2014-12-15 22:02:29,091 | INFO  | 1 - timer://fire | two | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5s]
2014-12-15 22:02:34,090 | INFO  | 1 - timer://fire | one | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5s]
2014-12-15 22:02:39,091 | INFO  | 1 - timer://fire | one | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5s]

```

我们可以看到（带有时间戳），有时会调用一个步骤/路由，有时会调用两个步骤/路由。

## 节流和样本 EIP

这些 EIP 提供了消息服务质量（QoS）的支持。这允许你实施某些服务级别协议（SLA），限制某些端点的阈值。

### 节流 EIP

节流 EIP 描述了如何限制达到端点的消息数量，以避免它。这允许你在路由、路由的部分和应用程序上保证服务级别协议（SLA）。

Camel 通过提供`throttle`标记支持这个 EIP。在节流中，你定义一个给定时间段和该时间段内允许的最大消息（或请求）数量。这个数量可以是静态的，也可以是动态的（例如使用头）。

为了说明节流 EIP，我们创建了以下`route.xml`蓝图描述符：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <camelContext >
      <route>
          <from uri="timer:first?period=500"/>
          <setBody><constant>Hello chapter5t</constant></setBody>
          <throttle timePeriodMillis="2000">
              <constant>1</constant>
              <to uri="direct:sla"/>
          </throttle>
      </route>
      <route>
          <from uri="direct:sla"/>
          <to uri="log:sla"/>
      </route>
  </camelContext>

</blueprint>
```

我们启动 Karaf 并安装`camel-blueprint`功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache-camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

我们将`route.xml`放入 Karaf 的`deploy`文件夹。

在 Karaf 的`log`文件中，我们可以看到：

```java
2014-12-16 07:05:03,106 | INFO  |  - timer://first | sla | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5t]
2014-12-16 07:05:05,105 | INFO  |  - timer://first | sla | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5t]
2014-12-16 07:05:07,105 | INFO  |  - timer://first | sla | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5t]
2014-12-16 07:05:09,105 | INFO  |  - timer://first | sla | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5t]
2014-12-16 07:05:11,104 | INFO  |  - timer://first | sla | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5t]

```

我们可以注意到，我们每两秒只有一个消息的时间戳，而定时器每 0.5 秒创建一个消息；这里有一个节流 EIP 的说明。

### 样本 EIP

样本 EIP 与节流 EIP 相关。目的是定期获取消息样本：

+   每给定数量的消息

+   每给定的时间

所有*其他*流量都被忽略。

Camel 通过使用`sample`标记支持这个 EIP。样本标记支持`messageFrequency`或`samplePeriod`属性。

为了说明样本 EIP，我们创建了以下`route.xml`蓝图 XML 描述符：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <camelContext >
      <route>
          <from uri="timer:first?period=500"/>
          <setBody><constant>Hello chapter5u</constant></setBody>
          <to uri="log:regular"/>
          <sample messageFrequency="5">
              <to uri="direct:frequency"/>
          </sample>
      </route>
      <route>
          <from uri="direct:frequency"/>
          <to uri="log:frequency"/>
      </route>
  </camelContext>

</blueprint>
```

第一条路由每 0.5 秒创建一个消息。我们使用`regular`日志记录。我们使用`sample`标记将第五个消息发送到频率路由（因此每五个消息，我们发送样本）。

我们启动 Karaf 并安装`camel-blueprint`功能：

```java
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache- camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

我们将我们的`route.xml`文件放入 Karaf 的`deploy`文件夹。

在 Karaf 的`log`文件中，我们可以看到：

```java
2014-12-16 14:57:59,342 | INFO  |  - timer://first | regular | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5u]
2014-12-16 14:57:59,824 | INFO  |  - timer://first | regular | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5u]
2014-12-16 14:58:00,324 | INFO  |  - timer://first | regular | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5u]
2014-12-16 14:58:00,824 | INFO  |  - timer://first | regular | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5u]
2014-12-16 14:58:01,324 | INFO  |  - timer://first | regular | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5u]
2014-12-16 14:58:01,325 | INFO  |  - timer://first | frequency | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5u]
2014-12-16 14:58:01,824 | INFO  |  - timer://first | regular | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5u]
2014-12-16 14:58:02,324 | INFO  |  - timer://first | regular | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5u]
2014-12-16 14:58:02,825 | INFO  |  - timer://first | regular | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5u]
2014-12-16 14:58:03,325 | INFO  |  - timer://first | regular | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5u]
2014-12-16 14:58:03,825 | INFO  |  - timer://first | regular | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5u]
2014-12-16 14:58:03,826 | INFO  |  - timer://first | frequency | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5u]

```

因此，我们可以看到频率路由为每五个常规负载消息获得一个`sample`标记。

## 延迟 EIP

延迟 EIP 允许你在消息传递过程中添加某种暂停。它就像在路由中的睡眠。

Camel 使用`delay`标记支持这个 EIP。延迟标记接受一个表达式来获取暂停时间。你可以使用 Camel 支持的任何语言来编写这个表达式。

我们在`chapter5g`和`chapter5h`示例中使用了这个 EIP。

在这些示例中，我们使用常数来定义延迟时间。同样，表达式可以使用 Camel 支持的任何语言编写。

为了说明具有动态延迟时间的 Delayer EIP，我们创建了一个使用豆子定义延迟时间（随机）的路由。

我们创建了以下 Maven `pom.xml` 文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <groupId>com.packt.camel</groupId>
  <artifactId>chapter5v</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>bundle</packaging>

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

此 Maven `pom.xml` 文件非常简单，只是将路由和豆子的 Blueprint XML 定义打包成一个 OSGi 包。

我们创建了一个简单的豆子，它随机创建延迟时间：

```java
package com.packt.camel.chapter5v;

import java.util.Random;

public class DelayBean {

  public int delay() {
      Random random = new Random();
      return random.nextInt(10000);
  }

}
```

我们使用蓝图 DSL 创建一个路由：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="delayBean" class="com.packt.camel.chapter5v.DelayBean"/>

  <camelContext >
      <route>
          <from uri="timer:fire?period=1000"/>
          <setBody>
              <constant>Hello chapter5v</constant>
          </setBody>
          <delay>
              <method ref="delayBean" method="delay"/>
          </delay>
          <to uri="log:delay"/>
      </route>
  </camelContext>

</blueprint>
```

此路由每秒使用计时器创建一条消息，并使用`delay`注解中的豆子。

我们构建我们的 OSGi 包：

```java
$ mvn clean install

```

我们的 OSGi 包已准备好在 Karaf 中部署。

我们启动 Karaf 并安装 `camel-blueprint` 功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache-camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

我们在 Karaf 的 OSGi 包中部署我们的包：

```java
karaf@root()> bundle:install -s mvn:com.packt.camel/chapter5v/1.0- SNAPSHOT
Bundle ID: 73

```

如果我们查看 Karaf 的 `log` 文件，我们可以看到：

```java
2014-12-16 17:23:57,477 | INFO  | 1 - timer://fire | delay | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5v]
2014-12-16 17:23:58,914 | INFO  | 1 - timer://fire | delay | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5v]
2014-12-16 17:24:05,976 | INFO  | 1 - timer://fire | delay | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5v]
2014-12-16 17:24:14,083 | INFO  | 1 - timer://fire | delay | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5v]
2014-12-16 17:24:20,893 | INFO  | 1 - timer://fire | delay | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5v]
2014-12-16 17:24:30,350 | INFO  | 1 - timer://fire | delay | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5v]

```

我们可以注意到消息是随机投递的（见时间戳），显示了 Delayer EIP 的作用。

## 负载均衡器 EIP

负载均衡器 EIP 将消息负载分发到不同的端点。

Camel 使用 `loadBalance` 注解支持此 EIP，它还支持不同的平衡策略，例如：

+   轮询策略在各个端点之间使用一种 *循环* 方式。

+   随机策略随机选择一个端点。

+   粘性策略使用表达式来选择目标端点。

+   主题策略将消息发送到所有端点（类似于 JMS 主题）。

+   如果第一个目标失败，故障转移策略将消息转发到下一个端点。

+   加权轮询策略类似于轮询策略，但你可以为不同的端点提供一个比率，以便在较高优先级下使用它们。

+   加权随机策略类似于随机策略，但你可以为不同的端点提供一个比率，以便在较高优先级下使用它们。

+   自定义策略允许你实现自己的负载均衡策略。

为了说明负载均衡器 EIP，我们使用以下 `route.xml` 文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <camelContext >
      <route>
          <from uri="timer:first?period=1000"/>
          <setBody><constant>Hello chapter5w</constant></setBody>
          <loadBalance>
               <roundRobin/>
               <to uri="direct:one"/>
               <to uri="direct:two"/>
               <to uri="direct:three"/>
          </loadBalance>
      </route>
      <route>
          <from uri="direct:one"/>
          <to uri="log:one"/>
      </route>
      <route>
          <from uri="direct:two"/>
          <to uri="log:two"/>
      </route>
      <route>
          <from uri="direct:three"/>
          <to uri="log:three"/>
      </route>
  </camelContext>

</blueprint>
```

第一条路由每秒创建一条消息。该消息使用轮询策略负载均衡到三个端点：`direct:one`、`direct:two` 和 `direct:three`。

我们启动 Karaf 并安装 `camel-blueprint` 功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache- camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

我们将 `route.xml` 文件放入 Karaf 的 `deploy` 文件夹。在 Karaf 的 `log` 文件中，我们可以看到：

```java
2014-12-16 17:58:33,370 | INFO  |  - timer://first | one | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5w]
2014-12-16 17:58:34,356 | INFO  |  - timer://first | two | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5w]
2014-12-16 17:58:35,355 | INFO  |  - timer://first | three | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5w]
2014-12-16 17:58:36,355 | INFO  |  - timer://first | one | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5w]
2014-12-16 17:58:37,355 | INFO  |  - timer://first | two | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5w]
2014-12-16 17:58:38,355 | INFO  |  - timer://first | three | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5w]
2014-12-16 17:58:39,355 | INFO  |  - timer://first | one | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5w]
2014-12-16 17:58:40,356 | INFO  |  - timer://first | two | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5w]

```

在这里，我们注意到每条消息都以轮询方式负载均衡到三个端点。

## 循环 EIP

循环 EIP 描述了如何在同一端点上多次迭代消息。

Camel 使用 `loop` 注解支持此 EIP。迭代次数可以是常数，也可以是表达式的结果（使用 Camel 支持的任何语言）。

为了说明循环 EIP，我们创建以下 `route.xml` 蓝图描述符：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <camelContext >
      <route>
          <from uri="timer:first?period=1000"/>
          <setBody><constant>Hello chapter5x</constant></setBody>
          <to uri="log:main"/>
          <loop>
              <constant>3</constant>
              <to uri="direct:loop"/>
          </loop>
      </route>
      <route>
          <from uri="direct:loop"/>
          <to uri="log:loop"/>
      </route>
  </camelContext>

</blueprint>
```

此路由每秒创建一条消息并记录此消息。消息被发送到一个执行三次迭代的循环中。

我们启动 Karaf 并安装 `camel-blueprint` 功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache- camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

我们将 `route.xml` 文件放入 Karaf 的 `deploy` 文件夹中。在 Karaf 的 `log` 文件中，我们可以看到：

```java
2014-12-16 18:19:35,000 | INFO  |  - timer://first | main | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5x]
2014-12-16 18:19:35,001 | INFO  |  - timer://first | loop | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5x]
2014-12-16 18:19:35,002 | INFO  |  - timer://first | loop | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5x]
2014-12-16 18:19:35,002 | INFO  |  - timer://first | loop | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5x]
2014-12-16 18:19:35,982 | INFO  |  - timer://first | main | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5x]
2014-12-16 18:19:35,982 | INFO  |  - timer://first | loop | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5x]
2014-12-16 18:19:35,983 | INFO  |  - timer://first | loop | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5x]
2014-12-16 18:19:35,983 | INFO  |  - timer://first | loop | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5x]

```

我们可以注意到消息已经被循环路由处理了三次。

# 消息转换 EIPs

这些 EIP 是消息翻译器 EIP 的扩展，并针对一些特殊用例。

## 内容丰富器 EIP

内容丰富器 EIP 描述了如何使用另一个系统丰富消息。例如，消息包含一个标识符，你希望从数据库中填充与该 ID 相关的数据。

要实现这个 EIP，你可以使用一个 bean 或处理器，就像你在消息翻译器 EIP 中做的那样。

你也可以使用一个使用转换工具（如 `Velocity`、`Xslt` 等）的端点。

然而，Camel 提供了两个专门用于内容丰富的符号。它们如下所示：

+   `enrich` 使用生产者端点检索数据，并使用聚合策略（如聚合器 EIP 中的策略）合并数据。例如，`enrich` 用于调用 web 服务或另一个直接端点。

+   另一方面，`pollEnrich` 使用消费者端点轮询数据，并使用聚合策略（如聚合器 EIP 中的策略）合并数据。例如，当在文件系统上轮询文件时使用 `pollEnrich`。

## 内容过滤器 EIP

内容过滤器 EIP 描述了当消息过大时如何移除消息的部分内容。

Camel 通过以下方式支持这个 EIP：

+   使用类似于消息翻译器 EIP 中的 bean 或处理器

+   使用包含一个过滤器表达式（如 `XPath`）的 `setBody` 符号

## 索赔检查 EIP

索赔检查 EIP 描述了如何用索赔检查（唯一键）替换消息的内容，你可以稍后使用它来再次检索消息。消息内容通过标识符识别，并暂时存储在数据库或文件系统等存储中。这种模式对于处理你不想传输到路由所有部分的非常大的消息来说非常有趣。

Camel 通过结合 Pipeline 和一个专门的 bean 来支持这个 EIP，使用标识符存储和检索消息。

## 标准化器 EIP

标准化器 EIP 是消息路由器 EIP 的组合，用于处理多种消息格式，并将消息转换为规范化和标准化的消息格式。一个 *经典* 的用途是将所有消息转换为唯一的规范格式。

Camel 通过结合基于内容的路由器和一系列的 beans 来支持这个 EIP，将消息转换为标准化格式。

## 排序 EIP

排序 EIP 对消息的内容进行排序。基本上，它将比较器应用于消息体。

Camel 使用 `sort` 符号支持这个 EIP。你可以提供你想要排序的内容（基本上是消息体），以及可选的比较器。

## 验证 EIP

验证 EIP 使用表达式或谓词来验证消息的内容。此 EIP 允许您在处理之前验证消息的有效负载。因此，您可以避免因格式无效而造成的错误。

Camel 使用 `validate` 符号支持此 EIP。`validate` 符号期望一个表达式，该表达式可以使用 Camel 支持的任何语言定义。

# 消息端点 EIP

消息端点 EIP 与 Camel 路由中的端点相关。Camel 通过使用端点提供的不同功能隐式地支持它们。

## 消息映射器 EIP

消息映射器 EIP 实际上与消息翻译器 EIP 是同一件事，只是位于端点级别。

在 Camel 中，这意味着您使用一个豆或处理器的方式与实现消息翻译器 EIP 的方式相同。

## 事件驱动消费者 EIP

事件驱动消费者 EIP 描述了一个端点，该端点监听传入的消息。当它收到消息时，端点会做出反应。

Camel 通过提供可以以这种方式工作的组件引导端点来支持此 EIP。例如，对于 CXF 或 JMS 组件来说就是这样。

此 EIP 由 Camel 隐式支持（您不需要使用任何特殊符号）。

## 轮询消费者 EIP

轮询消费者 EIP 描述了一个端点，该端点定期轮询系统（数据库、文件系统）以生成消息。

与事件驱动消费者 EIP 一样，Camel 通过提供可以以这种方式工作的组件引导端点来支持此 EIP。例如，对于文件或 FTP 组件来说就是这样。

此 EIP 由 Camel 隐式支持（您不需要使用任何特殊符号）。

## 竞争消费者 EIP

竞争消费者 EIP 描述了如何在单个端点上使用多个并发消费者。

Camel 在某些组件上支持此 EIP。例如，SEDA、VM 和 JMS 组件使用 `concurrentConsumers` 属性（值大于 `1`）来支持此 EIP。

## 消息调度器 EIP

消息调度器 EIP 描述了如何根据某些条件将消息调度到不同的端点。它基本上与消息路由器 EIP 相同。

Camel 以两种方式支持消息调度器 EIP：

+   使用基于内容的路由器 EIP（以及 `choice` 符号）

+   使用 JMS 组件（以及消息选择器）

## 选择性消费者 EIP

选择性消费者 EIP 描述了端点如何根据过滤器仅消费一些消息。

Camel 以两种方式支持此 EIP：

+   使用消息过滤器 EIP（以及 `filter` 符号）

+   在 JMS 组件上使用消息选择器

## 持久订阅者 EIP

持久订阅者 EIP 描述了如何使用发布-订阅模型，其中当订阅者未连接时，消息将被存储，当它们重新在线时，将等待交付。

Camel 使用 JMS 组件来支持这个 EIP。一个主题上的 JMS 端点消费者支持 `clientId` 和 `durableSubscriptionName` 属性，允许它充当持久订阅者。

## Idempotent Consumer EIP

Idempotent Consumer EIP 用于通过唯一标识每条消息来过滤重复消息。它充当消息过滤器以过滤重复的消息。基本上，每个消息标识符都存储在后端，EIP 会检查每条传入的消息，如果它尚未存在于存储中。

Camel 使用 `idempotentConsumer` 语法支持这个 EIP。不同的消息存储可用：

+   `MemoryIdempotentRepository` 在内存中的 HashMap 中存储消息

+   `FileIdempotentRepository` 在文件系统（属性文件中）上存储消息

+   `HazelcastIdempotentRepository` 在 Hazelcast 分布式 HashMap 中存储消息

+   `JdbcMessageIdRepository` 在数据库中存储消息

## Transactional Client EIP

Transactional Client EIP 描述了一个端点如何参与事务。这意味着客户端可以显式地执行事务的提交和回滚。客户端可以被视为事务性资源，因此可以管理两阶段提交。

Camel 通过提供组件支持的事务来支持这个 EIP。对于 JMS 端点来说也是如此。

## 消息网关和服务激活器 EIP

消息网关 EIP 描述了如何将一种消息格式包装成另一种消息格式。基本上，它将 Java 接口作为消息交换进行包装。

Camel 通过提供支持此类包装的组件（例如 Bean 和 CXF 组件）来支持这个 EIP。

基本上，服务激活器 EIP 与消息网关 EIP 非常相似。

# 系统管理 EIP

这些 EIP 与消息没有直接关系。它们提供了实现系统的一种非常方便的方式，并且在分析和管理路由系统本身时非常有用。

## ControlBus EIP

ControlBus EIP 的目的是能够管理和控制路由系统本身。这意味着能够停止路由系统，重新启动它，获取路由活动的详细信息，等等。

Camel 以两种方式支持这个 EIP：

+   Camel 提供了许多 JMX MBeans，您可以在其中找到许多指标并控制涉及的路线、处理器、组件等。

+   Camel 提供了一个 `controlbus` 组件，您可以使用它来管理 Camel 路线。

使用 `controlbus` 端点，您可以发送消息，例如，用于停止或启动路由。

## Detour EIP

Detour EIP 允许在满足控制条件时在额外的特定步骤上发送消息。您可以在需要时使用它来添加额外的验证、测试和调试步骤。

Camel 通过消息路由器支持这个 EIP。消息路由器的条件通过一个豆子实现；这个豆子实现了定义是否需要绕行的逻辑。

## Wire Tap EIP

Wire Tap EIP 允许您将消息的副本发送到特定的端点，而不会影响主路由。这个 EIP 在实现日志记录或审计系统时非常有用。

Camel 支持这个 EIP，使用 `wireTap` 注记。

为了说明 Wire Tap EIP，我们创建了以下 `route.xml` 蓝图描述符：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <camelContext >
      <route>
          <from uri="timer:fire?period=5000"/>
          <setBody><constant>Hello chapter5y</constant></setBody>
          <wireTap uri="direct:wiretap"/>
          <delay>
             <constant>3000</constant>
             <to uri="log:main"/>
          </delay>
      </route>
      <route>
          <from uri="direct:wiretap"/>
          <to uri="log:wiretap"/>
      </route>
  </camelContext>

</blueprint>
```

第一条路由每 5 秒创建一条消息。Wire Tap EIP 将消息的副本发送到 `wiretap` 路由。主路由继续处理，使用 3 秒的延迟。

我们启动 Karaf 并安装 `camel-blueprint` 功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache- camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

我们将 `route.xml` 文件放入 Karaf 的 `deploy` 文件夹中。在 Karaf 的 `log` 文件中，我们可以看到：

```java
2014-12-16 21:41:12,553 | INFO  | ead #2 - WireTap | wiretap | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5y]
2014-12-16 21:41:15,550 | INFO  | 1 - timer://fire | main | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5y]
2014-12-16 21:41:17,540 | INFO  | ead #3 - WireTap | wiretap | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5y]
2014-12-16 21:41:20,540 | INFO  | 1 - timer://fire | main | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5y]

```

Wire Tap 消息已被记录，而主路由仍在进行中。我们在此指出，`wire tap route` 不会影响主路由（在性能或阻塞消息方面）。

当日志后端可以花费时间（例如使用 JDBC 追踪器）时，Wire Tap EIP 特别有趣。

## Message History EIP

Message History EIP 用于分析和调试消息流。

这基本上意味着将历史记录附加到消息上，提供消息通过的所有端点的列表。

Camel 通过 Camel Tracer 功能支持这个 EIP。Tracer 主要是通道上的拦截器；它追踪所有交换细节。

由于这些信息，您可以看到消息在每个端点通过消息体的情况，等等。

每个 Camel 路由都嵌入 Tracer 功能，但默认情况下是禁用的。您可以通过 JMX 在路由 MBean 上启用 Tracer。

## Log EIP

Log EIP 允许您创建包含消息全部或部分内容的日志消息。

Camel 以两种方式支持这个 EIP：

+   使用日志组件，正如我们在本章的大部分示例中所做的那样

+   提供了 `log` 注记，允许您指定日志消息的格式。

为了说明 Log EIP，我们创建了以下 `route.xml` 蓝图描述符：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <camelContext >
      <route>
          <from uri="timer:fire?period=5000"/>
          <setBody><constant>Hello chapter5z</constant></setBody>
          <to uri="log:component"/>
          <log message="Hey, you said ${body} !" loggingLevel="WARN" logName="EIP"/>
      </route>
  </camelContext>

</blueprint>
```

我们可以看到两种日志记录方式——使用日志组件或使用 `log` 注记。

`log` 注记（EIP）让您完全控制要记录的内容、日志级别、记录器名称，以及可能的日志标记。

我们启动 Karaf 并安装 `camel-blueprint` 功能：

```java
$ bin/karaf
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache-camel/2.12.4/xml/features
karaf@root()> feature:install camel-blueprint

```

我们将 `route.xml` 文件放入 Karaf 的 `deploy` 文件夹中。在 Karaf 的 `log` 文件中，我们可以看到：

```java
2014-12-16 21:47:34,641 | INFO  | 1 - timer://fire | component | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: Hello chapter5z]
2014-12-16 21:47:34,642 | WARN  | 1 - timer://fire | EIP | 70 - org.apache.camel.camel-core - 2.12.4 | Hey, you said Hello chapter5z !

```

在这里，我们可以看到由日志组件和 Log EIP 生成的日志消息。

# 摘要

我们已经看到 Camel 支持所有类型的 Enterprise Integration Patterns；从 Messaging Systems EIPs 到 System Management EIPs，您现在可以准备使用所需的模式，并且可以轻松地在路由中实现它们。

注记是描述和指定复杂路由行为的一种非常方便的方式。连接性组件和 Enterprise Integration Patterns 的组合使 Camel 成为最灵活和强大的路由系统。

如果提供的组件或模式不符合您的需求，您始终可以创建自己的特定组件，正如我们将在下一章中看到的。
