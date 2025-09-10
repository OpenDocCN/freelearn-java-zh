# 第四章。豆子

在上一章中，我们看到了 Camel 的一个关键且非常有用的组件——处理器。然而，处理器与 Camel 相关联，因为它扩展了 `org.apache.camel.Processor` 接口。

这意味着为了在应用程序中重用一些现有的豆子，你必须将其包装在处理器中，这意味着需要维护额外的代码。

幸运的是，Camel 对 POJO 和豆子以及 Spring 或 Blueprint 等豆模型框架提供了广泛的支持。

在本章中，我们将看到：

+   Camel 在不同注册表中查找豆子以及不同的注册表实现

+   Camel 如何作为服务激活器来加载豆子并绑定参数

+   允许进行 *高级* 绑定的 Camel 注解

+   允许在参数绑定中使用代码的 Camel 语言注解

# Registry

可以将豆子直接用作处理器，这意味着直接内联在路由中。这允许我们使用轻量级、简单的编程模型，在 Camel 路由中重用现有组件。

当一个豆子在 Camel 路由中使用时，这个豆必须在注册表中注册。根据运行的环境不同，Camel 会启动不同的注册表。当 Camel 与豆子一起工作时，它会查找注册表以定位它们。

注册表在 `CamelContext` 级别定义。Camel 会自动为你创建一个与 `CamelContext` 相关的注册表。如果你手动创建 `CamelContext`，你可以实例化一个注册表并将其放入 `CamelContext`。

以下是一些与 Camel 一起提供的注册表实现：

+   `SimpleRegistry`

+   `JndiRegistry`

+   `ApplicationContextRegistry`

+   `OsgiServiceRegistry`

让我们逐一详细看看这些内容。

## SimpleRegistry

`SimpleRegistry` 是一个简单的实现，主要用于测试，在测试环境中只有有限数量的 JDK 类可用。它基本上是一个简单的 `Map`。

在使用之前，你必须手动创建 `SimpleRegistry` 的一个实例。Camel 默认不会加载任何 `SimpleRegistry`。

以下示例（在 `chapter4a` 文件夹中）展示了如何创建一个 `SimpleRegistry`，注册一个豆子，并在 Camel 路由中使用它。

在这个例子中，我们实例化了一个 `SimpleRegistry`，并将其放入我们创建的 `CamelContext` 中。

我们使用 `SimpleBean` 来填充 `SimpleRegistry`

在 `CamelContext` 中，我们添加了一个调用 `SimpleBean` 的路由。

为了简化执行，我们将此代码嵌入到一个主方法中，并通过 Maven 插件执行。

Maven 的 `pom.xml` 如下所示：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.packt.camel</groupId>
    <artifactId>chapter4a</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

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
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>1.3.2</version>
                <executions>
                    <execution>
                        <id>launch</id>
                        <phase>verify</phase>
                        <goals>
                            <goal>java</goal>
                        </goals>
                        <configuration>
                            <mainClass>com.packt.camel.chapter4a.Main</mainClass>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

在项目的 `src/main/java` 文件夹中，我们创建 `com.packt.camel.chapter4a` 包。

在这个包中，我们包含了：

+   一个 `SimpleBean` 类

+   一个 `Main` 类

`SimpleBean` 类相当简单；它只是说“你好”：

```java
package com.packt.camel.chapter4a;

public class SimpleBean {

    public String hello(String message) {
        System.out.println("***** Hello " + message + " *****");
        return "Hello" + message;
    }
}
```

`Main` 类只包含主方法。正是在这个方法中：

+   我们创建一个 `SimpleRegistry`

+   我们使用 `SimpleBean` 的一个实例来填充注册表

+   我们创建一个 `CamelContext`，它使用我们的 `SimpleRegistry`

+   我们在 `CamelContext` 中创建并添加了一个路由。这个路由使用注册表中的 `SimpleBean`。

这里是代码：

```java
package com.packt.camel.chapter4a;

import org.apache.camel.CamelContext;
import org.apache.camel.ProducerTemplate;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.impl.DefaultCamelContext;
import org.apache.camel.impl.SimpleRegistry;

public final class Main {

  public static void main(String[] args) throws Exception {
    SimpleRegistry registry = new SimpleRegistry();
    registry.put("simpleBean", new SimpleBean());

    CamelContext camelContext = new DefaultCamelContext(registry);
    camelContext.addRoutes(new RouteBuilder() {
      @Override
      public void configure() throws Exception {
        from("direct:start").to("bean:simpleBean").to("mock:stop");
      }
    }
    );

  camelContext.start();

   ProducerTemplate producerTemplate = camelContext.createProducerTemplate();
  producerTemplate.sendBody("direct:start", "Packt");
  camelContext.stop();
  }
}
```

要运行项目，请使用以下命令：

```java
mvn clean install

```

您应该看到执行结果：

```java
[INFO] --- exec-maven-plugin:1.3.2:java (launch) @ chapter4a ---
Hello Packt
[INFO] 

```

这证明了我们的 `CamelContext` 使用了 `SimpleRegistry`。Camel 成功在注册表中查找并使用了该豆。

## JndiRegistry

`JndiRegistry` 是一个使用现有 **Java 命名和目录服务**（**JNDI**）注册表来查找豆的实现。它是 Camel 在使用 Camel Java DSL 时使用的默认注册表。

可以使用 JNDI InitialContext 构建一个 `JndiRegistry`。它提供了使用现有 JNDI InitialContext 的灵活性。Camel 本身提供了一个简单的 `JndiContext`，您可以使用它与 `JndiRegistry` 一起使用。

我们可以通过实现一个与前一个示例非常相似的示例（使用 `SimpleRegistry`）来说明 `JndiRegistry` 的用法。

Maven 的 `pom.xml` 基本上与上一个示例相同：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <groupId>com.packt.camel</groupId>
  <artifactId>chapter4b</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>

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
              <groupId>org.codehaus.mojo</groupId>
              <artifactId>exec-maven-plugin</artifactId>
              <version>1.3.2</version>
              <executions>
                  <execution>
                      <id>launch</id>
                      <phase>verify</phase>
                      <goals>
                          <goal>java</goal>
                      </goals>
                      <configuration>
                          <mainClass>com.packt.camel.chapter4b.Main</mainClass>
                      </configuration>
                  </execution>
              </executions>
          </plugin>
      </plugins>
  </build>

</project>
```

在项目的 `src/main/java` 目录中，我们创建了 `com.packt.camel.chapter4b` 包。

这个包包含一个 `SimpleBean` 类，类似于前一个示例中的类：

```java
package com.packt.camel.chapter4b;

public class SimpleBean {

  public String hello(String message) {
    System.out.println("Hello " + message);
    return "Hello" + message;
  }
}
```

最后，主要区别在于 `Main` 类；我们只是将 `SimpleRegistry` 替换为 `JndiRegistry`：

```java
package com.packt.camel.chapter4b;

import org.apache.camel.CamelContext;
import org.apache.camel.ProducerTemplate;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.impl.DefaultCamelContext;
import org.apache.camel.impl.JndiRegistry;
import org.apache.camel.util.jndi.JndiContext;

public final class Main {

  public static void main(String[] args) throws Exception {
    JndiRegistry registry = new JndiRegistry(new JndiContext());
    registry.bind("simpleBean", new SimpleBean());

    CamelContext camelContext = new DefaultCamelContext(registry);
    camelContext.addRoutes(new RouteBuilder() {
      @Override
      public void configure() throws Exception {
             from("direct:start").to("bean:simpleBean").to("mock:stop");
      }
    }
    );

  camelContext.start();

    ProducerTemplate producerTemplate = camelContext.createProducerTemplate();
    producerTemplate.sendBody("direct:start", "Packt");

    camelContext.stop();
  }
}
```

要运行项目，请使用以下命令：

```java
mvn clean install

```

执行结果基本上相同：

```java
[INFO] --- exec-maven-plugin:1.3.2:java (launch) @ chapter4b ---
[WARNING] Warning: killAfter is now deprecated. Do you need it ? Please comment on MEXEC-6.

Hello Packt

```

我们切换到另一个注册表实现，对执行没有影响。

作为提醒，当您使用 Java DSL 为您的路由时，`JndiRegistry` 会隐式地由 Camel 创建。

## ApplicationContextRegistry

`ApplicationContextRegistry` 是一个基于 Spring 的实现，用于从 Spring `ApplicationContext` 中查找豆。当您在 Spring 环境中使用 Camel 时，此实现会自动使用。

## OsgiServiceRegistry

`OsgiServiceRegistry` 是连接到 OSGi 服务注册表的钩子。当在 OSGi 环境中运行时，Camel 会使用它。

# 创建 CompositeRegistry

这些注册表可以通过 `CompositeRegistry` 组合起来创建一个多层注册表。

您可以通过添加其他注册表来创建一个 `CompositeRegistry`。

为了说明 `CompositeRegistry` 的用法，我们创建了一个新的示例。

再次，Maven 的 `pom.xml` 基本上相同：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.packt.camel</groupId>
    <artifactId>chapter4c</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

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
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>1.3.2</version>
                <executions>
                    <execution>
                        <id>launch</id>
                        <phase>verify</phase>
                        <goals>
                            <goal>java</goal>
                        </goals>
                        <configuration>
                            <mainClass>com.packt.camel.chapter4c.Main</mainClass>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

在项目的 `src/main/java` 目录中，我们有一个 `com.packt.camel.chapter4c` 包。

我们再次有示例 `SimpleBean` 类：

```java
package com.packt.camel.chapter4c;

public class SimpleBean {

    public static String hello(String message) {
        System.out.println("Hello " + message);
        return "Hello" + message;
    }
}
```

但这次，在 `Main` 类中，我们创建了两个注册表，我们将它们收集到一个复合注册表中。

为了说明用法，我们在复合注册表的每个注册表部分创建了两个 `SimpleBean` 实例，每个实例在注册表中具有不同的名称。

我们现在在 `CamelContext` 中创建了两个路由；一个路由使用 `SimpleBean` 实例，另一个使用 `otherBean` 实例：

```java
package com.packt.camel.chapter4c;

import org.apache.camel.CamelContext;
import org.apache.camel.ProducerTemplate;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.impl.CompositeRegistry;
import org.apache.camel.impl.DefaultCamelContext;
import org.apache.camel.impl.JndiRegistry;
import org.apache.camel.impl.SimpleRegistry;
import org.apache.camel.util.jndi.JndiContext;

public final class Main {

  public static void main(String[] args) throws Exception {
    SimpleRegistry simpleRegistry = new SimpleRegistry();
    simpleRegistry.put("simpleBean", new SimpleBean());
    JndiRegistry jndiRegistry = new JndiRegistry(new JndiContext());
    jndiRegistry.bind("otherBean", new SimpleBean());
    CompositeRegistry registry = new CompositeRegistry();
    registry.addRegistry(simpleRegistry);
    registry.addRegistry(jndiRegistry);

    CamelContext camelContext = new DefaultCamelContext(registry);
    camelContext.addRoutes(new RouteBuilder() {
      @Override
      public void configure() throws Exception {
        from("direct:start").to("bean:simpleBean").to("mock:stop");
        from("direct:other").to("bean:otherBean").to("mock:stop");
      }
    }
    );

    camelContext.start();

    ProducerTemplate producerTemplate = camelContext.createProducerTemplate();
    producerTemplate.sendBody("direct:start", "Packt");
    producerTemplate.sendBody("direct:other", "Other");

    camelContext.stop();
  }
}
```

要运行项目，请使用以下命令：

```java
mvn clean install

```

现在，在执行时间，我们可以看到两个路由已经执行，每个路由都使用注册表中的`bean`实例。但实际上，每个实例都在不同的注册表中：

```java
[INFO] --- exec-maven-plugin:1.3.2:java (launch) @ chapter4c ---
[WARNING] Warning: killAfter is now deprecated. Do you need it ? Please comment on MEXEC-6.

Hello Packt
Hello Other

```

# 服务激活器

Camel 充当服务激活器，使用`BeanProcessor`，它位于调用者和实际 bean 之间。

`BeanProcessor`是一个特殊的处理器，它将传入的交换转换为对 bean（POJO）的方法调用。

当被调用时，`BeanProcessor`执行以下步骤：

1.  它在注册表中查找 bean。

1.  它选择要调用的 bean 的方法。

1.  它绑定到所选方法的参数。

1.  它实际上调用了该方法。

1.  它可能处理发生的任何调用错误。

1.  它将方法的响应设置为输出消息的主体。

# Bean 和方法绑定

在步骤 2 中，当`BeanProcessor`选择要调用的方法时，消息/方法绑定可以以不同的方式发生。Camel 尝试以下步骤来解决 bean 方法：

1.  如果传入的消息（`in`消息）包含`CamelBeanMethodName`头，则调用此方法，将`in`消息主体转换为方法参数的类型。

1.  您可以直接在路由定义中（在 bean 端点）指定方法名称。

1.  如果 bean 包含带有`@Handler`注解的方法，则调用此方法。

1.  如果 bean 可以转换为处理器（包含`process()`方法），我们将回退到上一章中看到的常规处理器使用方式。

1.  如果传入消息的主体可以转换为`org.apache.camel.component.bean.BeanInvocation`组件，则它是`getMethod()`方法的结果，用作方法名称。

1.  否则，使用传入消息的主体类型来查找匹配的方法。

在方法查找过程中可能会发生几个异常。它们如下：

+   如果 Camel 找不到方法，它会抛出`MethodNotFoundException`异常

+   如果 Camel 无法唯一解析一个方法（例如，根据方法参数），它会抛出`AmbigiousMethodCallException`异常。

+   在 Camel 调用所选方法之前，它必须将传入消息的主体转换为方法所需的参数类型。如果失败，则抛出`NoTypeConversionAvailableException`异常。

一旦确定了方法名称，Camel 就会填充方法参数；这就是我们所说的方法参数绑定。

一些 Camel 类型会自动绑定，例如：

+   `org.apache.camel.Exchange`

+   `org.apache.camel.Message`

+   `org.apache.camel.CamelContext`

+   `org.apache.camel.TypeConverter`

+   `org.apache.camel.spi.Registry`

+   `java.lang.Exception`

这意味着您可以直接在方法参数中使用这些类型中的任何一种。

例如，您的 bean 可能只包含一个方法：

```java
public void doMyStuff(Exchange exchange);
```

Camel 将提供当前交换给您的函数。

默认情况下，Camel 会尝试将`in`消息主体转换为方法的第一个参数。

`bean` 方法的返回语句用于填充 `in` 消息的正文（如果 bean 通过 Camel bean 组件使用）或头部值（如果 bean 通过 `setHeader` Camel 语句使用）。

然而，根据你的 bean，你可能会有一些歧义。Camel 通过提供一系列注解，让你可以精细控制方法参数，这些注解将在下一节中介绍。

# 注解

根据你的 bean，你可能会有一些歧义。Camel 通过提供一系列注解，让你可以精细控制方法参数。

多亏了注解，你可以描述方法绑定和参数绑定预期的绑定。

对于方法绑定，Camel 提供了 `@Handler` 注解。这个注解允许你指定 Camel 在执行期间将使用的方法。

例如，你可能有一个以下 bean：

```java
public class MyBean {
  public void other(String class) {  }

public void doMyStuff(String class) { ... }
}
```

在这种情况下，Camel（在路由定义中没有指定要使用的方法）将无法找到要调用的方法。

`@Handler` 注解消除了歧义：

```java
public class MyBean {
  public void other(String class) { … }
  @Handler
public void doMyStuff(String class) { ... }
}
```

Camel 还为方法参数提供了注解。`@Body` 将参数绑定到 `in` 消息体。它允许直接绑定类型，就像直接绑定一个 POJO：

```java
@Handler
public void doMyLogic(@Body MyPojo pojo) { … }
```

Camel 将使用转换器将实际消息体中的内容转换为方法参数期望的类型。`@ExchangeException` 将参数绑定到 `Exchange` 异常。这个注解允许你直接在你的方法中注入 `Exchange` 异常。例如，你可以测试异常是否不为空，并相应地做出反应。

```java
public void doMyLogic(@Body String body, @ExchangeException Exception exception) {
  if (exception != null) { … } else { … }
}
```

`@Header` 将参数绑定到 `in` 消息的头部。你可以在注解中指定头部名称，如下所示：

```java
public void doMyLogic(@Body String body, @Header("FirstHeader") String firstHeader, @Header("SecondHeader") String second header) { … }
```

`@Headers` 将参数绑定到包含 `in` 消息所有头部的 `Map`。当你需要在方法中操作多个头部时，这特别有趣。使用这个注解，参数必须是 `Map` 类型。

```java
public void doMyLogic(@Body String body, @Headers Map headers) { … }
```

另一方面，类似于 `@Headers` 对于 `in` 消息，`@OutHeaders` 注解将参数绑定到包含 `out` 消息所有头部的 `Map`。当你需要使用 `put()` 方法在 Map 中填充一些头部时，这特别有趣：

```java
public void doMyLogic(@Body String body, @Headers Map inHeaders, @OutHeaders Map outHeaders) { … }
```

`@Property` 将 `Exchange` 的一个属性绑定。提醒一下，属性的生存期是 `Exchange`，而头部与消息相关。属性名称直接在注解中提供。

```java
public void doMyLogic(@Body String body, @Property("TheProperty") String exProperty) { … }
```

对于头部，`@Properties` 将属性绑定到包含 `Exchange` 所有属性的 `Map`。同样，向方法添加新属性（使用 Map 的 `put()` 方法）很有趣：

```java
public void doMyLogic(@Body String body, @Properties Map exProperties) { … }
```

## 表达式语言注解

也可以直接利用 Camel 支持的语言来填充方法参数。

提供以下注解：

+   `@Bean` 将另一个 bean 绑定到参数。它允许你将 bean 注入到 bean 中。Camel 将寻找注解中提供的 ID 对应的 bean：

    ```java
    public void doMyLogic(@Body String body, @Bean("anotherBean") AnotherBean anotherBean) { … }
    ```

+   `@BeanShell` 将 bean 方法调用的结果绑定到参数。BeanShell 是一种方便的语言，允许你显式调用 bean 方法。bean 脚本直接在注解上定义：

    ```java
    public void doMyLogic(@Body String body, @BeanShell("myBean.thisIsMyMethod()") methodResult) { … }
    ```

+   `@Constant` 将静态 String 绑定到参数：

    ```java
    public void doMyLogic(@Body String body, @Constant("It doesn't change") String myConstant) { … }
    ```

+   `@EL` 将表达式语言（JUEL）的结果绑定到参数。表达式在注解中定义：

    ```java
    public void doMyLogic(@Body String body, @EL("in.header.myHeader == 'expectedValue') boolean matched) { … }
    ```

+   `@Groovy` 将 Groovy 表达式的结果绑定到参数。表达式在注解中定义。请求关键字对应于`in`消息：

    ```java
    public void doMyLogic(@Body String body, @Groovy("request.attribute") String attributeValue) { … }
    ```

+   `@JavaScript` 将 JavaScript 表达式的结果绑定到参数。表达式在注解中定义：

    ```java
    public void doMyLogic(@Body String body, @JavaScript(in.headers.get('myHeader') == 'expectedValue') boolean matched) { … }
    ```

+   `@MVEL` 将 MVEL 表达式的结果绑定到参数。表达式在注解中定义。请求关键字对应于`in`消息：

    ```java
    public void doMyLogic(@Body String body, @MVEL("in.headers.myHeader == 'expectedValue'") boolean matched) { … }
    ```

+   `@OGNL` 将 OGNL 表达式的结果绑定到参数。表达式在注解中定义。请求关键字对应于`in`消息：

    ```java
    public void doMyLogic(@Body String body, @OGNL("in.headers.myHeader == 'expectedValue'") boolean matched) { … }
    ```

+   `@PHP` 将 PHP 表达式的结果绑定到参数。表达式在注解中定义。请求关键字对应于`in`消息：

    ```java
    public void doMyLogic(@Body String body, @PHP("in.headers.myHeader == 'expectedValue'") boolean matched) { … }
    ```

+   `@Python` 将 Python 表达式的结果绑定到参数。表达式在注解中定义。请求关键字对应于`in`消息：

    ```java
    public void doMyLogic(@Body String body, @Python("in.headers.myHeader == 'expectedValue'") boolean matched) { … }
    ```

+   `@Ruby` 将 Ruby 表达式的结果绑定到参数。表达式在注解中定义。请求的 Ruby 变量对应于`in`消息：

    ```java
    public void doMyLogic(@Body String body, @Ruby("$request.headers['myHeader'] == 'expectedValue'") boolean matched) { … }
    ```

+   `@Simple` 将简单表达式的结果绑定到参数。表达式在注解中定义。Simple 是 Camel 语言，允许你直接使用 Camel 对象定义简单表达式：

    ```java
    public void doMyLogic(@Body String body, @Simple("${in.header.myHeader}") String myHeader) { … }
    ```

+   `@XPath` 将 XPath 表达式的结果绑定到参数。表达式在注解中定义。它非常方便从 XML `in`消息中提取部分内容：

    ```java
    public void doMyLogic(@XPath("//person/name") String name) { … }
    ```

+   `@XQuery` 将 XQuery 表达式的结果绑定到参数。表达式在注解中定义。像 XPath 一样，它非常方便从 XML `in`消息中提取部分内容：

    ```java
    public void doMyLogic(@XQuery("/person/@name") String name) { … }
    ```

当然，可以将不同的注解与多个参数组合使用。

Camel 提供了极大的灵活性，无论你已知的语言是什么，你都可以在表达式和谓词定义中使用它。

# 示例 - 创建包含 bean 的 OSGi 包

我们通过一个简单的示例来说明 bean 的使用。此示例将创建一个包含由 Camel 路由调用的 bean 的 OSGi 包。

我们将创建一个在路由的两个部分中使用的 bean：

+   一种直接使用 Camel bean 组件来更改入站消息的正文

+   另一种在路由中定义一个报头

首先，我们为我们的 bundle 创建 Maven 项目`pom.xml`：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <groupId>com.packt.camel</groupId>
  <artifactId>chapter4</artifactId>
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

这个`pom.xml`文件相当简单：

+   它定义了 Camel 核心依赖项，以便获取 bean 注解

+   它使用 Maven bundle 插件将 bean 和路由打包为 OSGi bundle

## 创建 MyBean 类

我们创建包含两个方法的`MyBean`类：

+   `doMyLogic()`方法如前所述被注解为`@Handler`。这是 Camel Bean 组件将使用的方法。此方法有一个唯一的`body`参数，类型为`String`。多亏了`@Body`注解，此参数将由 Camel 用入消息的正文填充。

+   `setMyHeader()`方法仅返回`String`。此方法将由 Camel 用于填充入消息的头。

`MyBean`类的代码如下：

```java
package com.packt.camel.chapter4;

import org.apache.camel.Body;
import org.apache.camel.Handler;

public class MyBean {

  @Handler
  public String doMyLogic(@Body String body) {
      return "My Logic got " + body;
  }

  public String setMyHeader() {
      return "Here's my header definition, whatever the logic is";
  }

}
```

我们可以注意到`doMyLogic()`方法将 bean 定义为消息转换器：它将入消息的正文转换为另一个消息正文。它看起来像前一章中使用的`PrefixerProcessor`。

## 使用 Camel Blueprint DSL 编写路由定义

我们将使用 Blueprint DSL 来编写路由的定义。多亏了这一点，我们不需要提供所有管道代码来创建`CamelContext`并将其作为 OSGi 服务引用。

`CamelContext`由 Camel 隐式创建，我们直接使用 XML 描述路由。

在我们的 bundle 的`OSGI-INF/blueprint`文件夹中，我们创建以下`route.xml`定义：

```java
<?xml version="1.0" encoding="UTF-8"?>
<blueprint >

  <bean id="myBean" class="com.packt.camel.chapter4.MyBean"/>

  <camelContext >
      <route>
          <from uri="timer:fire?period=5000"/>
          <setBody><constant>Hello Chapter4</constant></setBody>
          <to uri="bean:myBean"/>
          <setHeader headerName="myHeaderSetByTheBean">
              <method bean="myBean" method="setMyHeader"/>
          </setHeader>
          <to uri="log:blueprintRoute"/>
      </route>
  </camelContext>

</blueprint>
```

首先，我们在 Blueprint 容器中声明我们的 bean。这意味着 Blueprint 容器将使用我们的类来创建此 bean 的实例并为其分配一个 ID。

当使用 Blueprint DSL 时，Camel 使用 Blueprint 容器注册表；这意味着 Camel 将使用 Blueprint 容器中的 ID 查找 bean。

使用 Camel，DSL 将使用完全相同的行为。

`<route/>`元素定义了以下路由：

1.  路由从计时器开始，每 5 秒创建一个空交换。

1.  我们使用`<setBody/>`定义静态内容 Hello Chapter4 作为`in`消息的正文。

1.  交换被发送到我们的 bean。我们使用 Camel Bean 组件直接调用`myBean`。Camel 将在 Blueprint 容器中寻找名为 myBean 的 bean。一旦找到，它将使用带有`@Handler`注解的`doMyLogic()`方法。Camel 将绑定入消息的正文与`doMyLogic()`的正文参数。

1.  在 bean 处理器之后，我们可以看到 bean 的另一种使用。这次，我们使用 bean（相同的实例）来定义入消息的`myHeaderSetByTheBean`头。在这里，我们使用`<method/>`语法提供`myBean`bean ID 和`setMyHeader()`方法。Camel 将在 Blueprint 容器中查找具有`myBean` ID 的 bean，并将调用`setMyHeader()`方法。此方法的返回值将用于填充`myHeaderSetByTheBean`头。

1.  最后，我们将交换发送到日志端点。

# 构建和部署

我们现在准备好构建我们的 bundle。

使用 Maven，我们运行以下命令：

```java
$ mvn clean install
[INFO] Scanning for projects...
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building chapter4 1.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.4.1:clean (default-clean) @ chapter4 ---
[INFO] Deleting /home/jbonofre/Workspace/sample/chapter4/target
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ chapter4 ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Copying 1 resource
[INFO]
[INFO] --- maven-compiler-plugin:3.2:compile (default-compile) @ chapter4 ---
[INFO] Changes detected - recompiling the module!
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[INFO] Compiling 1 source file to /home/jbonofre/Workspace/sample/chapter4/target/classes
[INFO]
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ chapter4 ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /home/jbonofre/Workspace/sample/chapter4/src/test/resources
[INFO]
[INFO] --- maven-compiler-plugin:3.2:testCompile (default-testCompile) @ chapter4 ---
[INFO] No sources to compile
[INFO]
[INFO] --- maven-surefire-plugin:2.17:test (default-test) @ chapter4 ---
[INFO] No tests to run.
[INFO]
[INFO] --- maven-bundle-plugin:2.3.7:bundle (default-bundle) @ chapter4 ---
[INFO]
[INFO] --- maven-install-plugin:2.5.1:install (default-install) @ chapter4 ---
[INFO] Installing /home/jbonofre/Workspace/sample/chapter4/target/chapter4-1.0-SNAPSHOT.jar to /home/jbonofre/.m2/repository/com/packt/camel/chapter4/1.0-SNAPSHOT/chapter4-1.0-SNAPSHOT.jar
[INFO] Installing /home/jbonofre/Workspace/sample/chapter4/pom.xml to /home/jbonofre/.m2/repository/com/packt/camel/chapter4/1.0-SNAPSHOT/chapter4-1.0-SNAPSHOT.pom
[INFO]
[INFO] --- maven-bundle-plugin:2.3.7:install (default-install) @ chapter4 ---
[INFO] Installing com/packt/camel/chapter4/1.0-SNAPSHOT/chapter4-1.0-SNAPSHOT.jar
[INFO] Writing OBR metadata
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 7.037s
[INFO] Finished at: Sun Nov 30 23:07:59 CET 2014
[INFO] Final Memory: 32M/1343M
[INFO] ------------------------------------------------------------------------

```

我们现在可以在我们的本地 Maven 仓库中找到我们的包（默认情况下位于主目录的 `.m2/repository` 文件夹中）。

我们可以将这个包部署到 Karaf OSGi 容器中。

在启动 Karaf（例如使用 `bin/karaf` 脚本）之后，我们使用 `feature:repo-add` 命令添加 Camel 功能：

```java
karaf@root()> feature:repo-add camel 2.12.4
Adding feature url mvn:org.apache.camel.karaf/apache-camel/2.12.4/xml/features

```

我们安装 camel-blueprint 功能：

```java
karaf@root()> feature:install -v camel-blueprint
Installing feature camel-blueprint 2.12.4
Installing feature camel-core 2.12.4
Installing feature xml-specs-api 2.2.0
Found installed bundle: org.apache.servicemix.specs.activation-api-1.1 [64]
Found installed bundle: org.apache.servicemix.specs.stax-api-1.0 [65]
Found installed bundle: org.apache.servicemix.specs.jaxb-api-2.2 [66]
Found installed bundle: stax2-api [67]
Found installed bundle: woodstox-core-asl [68]
Found installed bundle: org.apache.servicemix.bundles.jaxb-impl [69]
Found installed bundle: org.apache.camel.camel-core [70]
Found installed bundle: org.apache.camel.karaf.camel-karaf-commands [71]
Found installed bundle: org.apache.camel.camel-blueprint [72]

```

我们现在可以安装我们的包并启动它：

```java
karaf@root()> bundle:install mvn:com.packt.camel/chapter4/1.0-SNAPSHOT
Bundle ID: 73
karaf@root()> bundle:start 73

```

我们可以看到我们的路由正在运行，因为我们可以看到日志消息（使用 `log:display` 命令）：

```java
karaf@root()> log:display
...
2014-11-30 23:13:52,944 | INFO  | 1 - timer://fire | blueprintRoute                 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: My Logic got Hello Chapter4]
2014-11-30 23:13:57,943 | INFO  | 1 - timer://fire | blueprintRoute                 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: My Logic got Hello Chapter4]
2014-11-30 23:14:02,945 | INFO  | 1 - timer://fire | blueprintRoute                 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: My Logic got Hello Chapter4]
2014-11-30 23:14:07,944 | INFO  | 1 - timer://fire | blueprintRoute                 | 70 - org.apache.camel.camel-core - 2.12.4 | Exchange[ExchangePattern: InOnly, BodyType: String, Body: My Logic got Hello Chapter4]

```

我们可以使用 `camel:route-list` 命令查看我们的路由：

```java
karaf@root()> camel:route-list
 Context      Route        Status 
 -------      -----        ------ 
 73-camel-3   route1       Started 

```

`camel:route-info` 命令提供了关于我们路由的详细信息，如下所示：

```java
karaf@root()> camel:route-info route1
Camel Route route1
 Camel Context: 73-camel-3

Properties
 id = route1
 parent = 58a5a53e

Statistics
 Inflight Exchanges: 0
 Exchanges Total: 32
 Exchanges Completed: 32
 Exchanges Failed: 0
 Min Processing Time: 1 ms
 Max Processing Time: 13 ms
 Mean Processing Time: 2 ms
 Total Processing Time: 92 ms
 Last Processing Time: 3 ms
 Delta Processing Time: 0 ms
 Load Avg: 0.00, 0.00, 0.00
 Reset Statistics Date: 2014-11-30 23:13:36
 First Exchange Date: 2014-11-30 23:13:37
 Last Exchange Completed Date: 2014-11-30 23:16:12

Definition
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<route id="route1" >
 <from uri="timer:fire?period=5000"/>
 <setBody id="setBody1">
 <constant>Hello Chapter4</constant>
 </setBody>
 <to uri="bean:myBean" id="to1"/>
 <setHeader headerName="myHeaderSetByTheBean" id="setHeader1">
 <method bean="myBean" method="setMyHeader"></method>
 </setHeader>
 <to uri="log:blueprintRoute" id="to2"/>
</route>

```

多亏了豆子支持，我们可以在 Camel 路由中轻松使用现有代码。

此外，凭借广泛的注释和所支持的语言，你可以完全控制你的豆子的使用。

使用编写路由定义所用的 DSL，Camel 知道它在哪个系统上运行，因此，它加载不同的豆子注册实现，使得以标准方式定义豆子成为可能。

Camel 豆子支持是 Camel 处理器的绝佳补充。

如果我们将在下一章中看到的 EIP 的大部分都是使用 Camel 处理器实现的，那么一些 EIP 可以使用豆子实现（例如 MessageTranslator EIP）。

# 摘要

在本章中，我们看到了如何在 Camel 路由中使用豆子。

首先，我们看到了 Camel 支持的不同注册表，Camel 在这些注册表中寻找豆子。具体来说，我们看到了 Camel DSL 与默认注册表加载之间的映射。我们看到了不同注册表在实际操作中的示例，包括组合注册表。对于这个查找，Camel 充当服务激活器。示例展示了如何利用 Spring 或 Blueprint 服务注册表。

我们还看到了使用注解来限定方法和参数绑定的情况。这些注解可以与语言注解结合使用，允许以非常强大的方式填充方法参数。

在下一章中，我们将看到 Camel 的一个关键特性——路由和企业集成模式支持。我们将看到可用于实现不同 EIP 的现成处理器和 DSL。
