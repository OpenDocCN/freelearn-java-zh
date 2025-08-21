# 第四章：微服务配置和容错

在本章中，我们将首先介绍微服务配置，因为它是其他微服务功能配置的基础，除了应用程序级配置。微服务配置规范提供了一种通用方法，用于从各种来源（属性文件、系统属性、环境变量、数据库等）检索配置。

我们将涵盖的主题包括以下内容：

+   从您的应用程序读取配置

+   为您的应用程序提供额外的配置源

+   提供将普通配置转换为特定于应用程序对象的转换

# 理解 Eclipse 微服务配置

每个应用程序都需要一些外部配置，以使其行为适应其正在运行的运行时平台。这可以从应用程序必须连接到的 HTTP 端点，或者某些内部结构的大小。

这些配置参数也可以来自不同的来源：

+   从操作系统或云原生环境中的容器（通过使用环境变量）

+   从 Java 虚拟机（通过系统属性）

+   从一些外部配置文件（如 Java 属性文件）

+   从其他地方（如 LDAP 服务器、数据库、键值存储等）

一方面，这些配置参数来自许多不同的来源。在...

# 从微服务配置 API 读取配置

微服务配置规范定义了两个对象来读取配置参数的值：

+   使用`Config`对象以编程方式访问配置值

+   `@ConfigProperty`注解用于使用**上下文和依赖注入**（**CDI**）注入配置值

让我们详细讨论它们。

# 配置对象

`org.eclipse.microprofile.config.Config`接口是检索 Java 应用程序中配置的入口点。

获取`Config`实例有两种方法：

1.  第一种（且首选）方法是使用 CDI 将其注入代码中：

```java
@Inject
private Config config;
```

1.  第二种方法是调用静态方法`org.eclipse.microprofile.config.ConfigProvider#getConfig()`，以获取`Config`实例：

```java
Config config = ConfigProvider.getConfig();
```

`Config`接口提供两种检索属性的方法：

+   `getValue(String propertyName, Class propertyType)`：如果配置中不存在该属性，此方法将抛出运行时异常。仅对于**必需**的配置才应使用此方法 ...

# `@ConfigProperty`注解

`@ConfigProperty`注解可用于使用 CDI 将配置值注入 Java 字段或方法参数，如所示：

```java
@Inject
@ConfigProperty(name="my.url")
private URL myURL;
```

`@ConfigProperty`注解可以有一个`defaultValue`，如果在底层的`Config`中找不到配置属性，则用于配置字段：

```java
@Inject
@ConfigProperty(name="my.url", defaultValue="http://localhost/")
private URL myURL;
```

如果未设置`defaultValue`且未找到任何属性，应用程序将抛出`DeploymentException`，因为无法正确配置。

如果一个配置属性可能不存在，可以使用`Optional`，如下面的代码块所示：

```java
@Inject
@ConfigProperty(name="my.url")
private Optional<URL> someUrl; // will be set to Optional.empty if the
                               // property `my.url` cannot be found
```

在读取配置之后，我们需要提供源配置源，这部分将在下一节中介绍。

# 提供配置源

配置源由`ConfigSource`接口表示。除非你想提供在你应用程序中使用的 MicroProfile 实现不可用的配置源，否则你不需要实现这个接口。

如果在多个配置源中找到了一个属性，`Config`将返回`ConfigSource`接口中序号最高的值。

排序`ConfigSource`很重要，因为用户可以提供自定义的`ConfigSource`接口，这除了由 MicroProfile Config 实现提供的默认接口。

# 默认的 ConfigSources

默认情况下，一个 MicroProfile Config 实现必须提供三个配置源：

+   来自 Java 虚拟机系统的属性（序号为`400`）

+   环境变量（序号为`300`）

+   存储在`META-INF/microprofile-config.properties`中的属性（序号为`100`）

配置源的`ordinal`值决定了配置源的优先级。特别是，如果一个属性既在系统属性中定义，又在环境变量中定义，将采用系统属性的值（其序号高于环境变量）。

属性名没有限制。然而，一些操作系统可能会对环境变量的名称施加一些限制（例如，大多数 Unix 壳牌不允许`"."`）。如果您有一个可能从环境变量中配置的属性，您必须相应地命名您的属性。

例如，属性名`my_url`可以由环境变量使用，而`my.url`则不行。

**MicroProfile Config 1.3 新特性** MicroProfile Config 1.3 引入了一个从配置属性名到环境变量的映射规则。这个规则为每个属性名搜索三个环境变量的变体：

+   精确匹配

+   将任何非字母数字字符替换为`_`

+   将任何非字母数字字符替换为`_`并使用大写字母

这意味着，在 Java 应用程序中，我们可以有一个名为`app.auth.url`的属性，并使用`APP_AUTH_URL`环境变量来配置它。

接下来让我们看看另一种配置源。

# 自定义 ConfigSources 实现

在您的应用程序中，可以提供额外的配置源，这些源将由 MicroProfile Config 实现自动添加。

你需要定义一个`org.eclipse.microprofile.config.spi.ConfigSource`的实现，并为它添加一个 Java `ServiceLoader`配置，并将该文件放在你的应用程序归档中作为`META-INF/services/` `org.eclipse.microprofile.config.spi.ConfigSource`。供您参考，以下是一个环境`ConfigSource`实现定义的示例：

```java
package io.packt.sample.config;import java.io.Serializable;import java.util.Collections;import java.util.Map;import org.eclipse.microprofile.config.spi.ConfigSource;public class EnvConfigSource ...
```

# 使用转换器进行高级配置

MicroProfile Config 将从其`ConfigSource`读取 Java `String`对象。然而，它提供了将这些`String`对象转换为应用程序中更具体类型的设施。

例如，我们之前描述的`myUrl`字段是一个`URL`对象。相应的属性`my.url`以`String`对象的形式读取，然后在被注入之前转换为`URL`对象。

如果应用程序使用`Config`对象，MicroProfile Config 实现也将把`String`对象转换为`getValue`和`getOptionalValue`方法的第二个参数传递的类型。这个转换可以使用不同的转换器类型：内置、自动和自定义。我们现在将详细讨论它们。

# 内置转换器

MicroProfile Config 实现为基本类型（`boolean`、`int`、`long`、`byte`、`float`和`double`）及其对应的 Java 类型（例如`Integer`）提供了内置转换器。

它还提供了使用`","`作为项目分隔符在属性值中支持数组的功能。如果`","`必须是项目的一部分，它必须用反斜杠`"\"`转义：

```java
private String[] pets = config.getValue("myPets", String[].class)
```

如果`myPets`属性的值是`dog,cat,dog\\,cat`，存储在`pets`中的数组的元素将是`{"dog", "cat", "dog,cat"}`。

# 自动转换器

MicroProfile Config 还定义了*自动转换器*。如果一个转换器对给定的 Java 类型未知，它将尝试使用三种不同的方法之一将`String`对象转换为它：

+   Java 类型有一个带有`String`参数的公共构造函数。

+   它有一个`public static valueOf(String)`方法。

+   它有一个`public static parse(String)`方法。

这就是`my.url`属性如何从`String`转换为`URL`，因为`java.net.URL`类型有一个`public URL(String)`构造函数。

# 自定义转换器

如果你的应用程序定义了 Java 类型，这些类型不提供自动转换器涵盖的这三个案例，MicroProfile Config 仍然可以使用自定义转换器提供转换，这些转换器扩展了以下定义的`org.eclipse.microprofile.config.spi.Converter`接口：

```java
public interface Converter<T> {
    /**
     * Configure the string value to a specified type
     * @param value the string representation of a property value.
     * @return the converted value or null
     *
     * @throws IllegalArgumentException if the value cannot be converted to        the specified type.
     */
    T convert(String value);
```

你必须编写一个`org.eclipse.microprofile.config.spi.Converter`的实现，然后将其名称添加到`/META-INF/services/org.eclipse.microprofile.config.spi.Converter ...`

# 理解 Eclipse MicroProfile 容错

容错提供了一组工具，通过使代码更具弹性来防止代码失败。其中大多数工具受到了良好的开发实践（如重试或回退）或知名开发模式（如断路器或隔舱）的启发。

容错基于 CDI ，更确切地说，是基于 CDI 拦截器实现。它还依赖于 MicroProfile Config 规范，以允许为容错策略提供外部配置。

规范的主要思想是将业务逻辑与容错样板代码解耦。为了实现这一点，规范定义了拦截器绑定注解，以在方法执行或类上（在这种情况下，所有类方法具有相同的策略）应用容错策略。

容错规范中包含的政策如下：

+   **超时**：这是使用`@Timeout`注解 applied。它在当前操作中添加了超时。

+   **重试**：这是使用`@Retry`注解 applied。它添加了重试行为，并允许对当前操作进行配置。

+   **回退**：这是使用`@Fallback`注解 applied。它定义了在当前操作失败时应执行的代码。

+   ** bulkhead**：这是使用`@Bulkhead`注解 applied。它将当前操作中的故障隔离以保留其他操作的执行。

+   **断路器**：这是使用`@CircuitBreaker`注解 applied。它提供了一个自动快速失败的执行，以防止系统过载。

+   **异步**：这是使用`@Asynchronous`注解 applied。它将当前操作设置为异步（也就是说，代码将异步调用）。

应用这些策略之一或多个就像在需要这些策略的方法（或类）上添加所需的注解一样简单。所以，使用容错是非常简单的。但这种简单性并没有阻止灵活性，感谢所有可用配置参数每个策略。

目前，以下供应商为 Fault Tolerance 规范提供实现：

+   Red Hat in Thorntail and Quarkus

+   IBM in Open Liberty

+   Payara in Payara Server

+   Apache Safeguard for Hammock and TomEE

+   KumuluzEE for KumuluzEE framework

所有这些实现都支持容错，因此支持下一节中描述的相同一组功能。

# MicroProfile Fault Tolerance in action

正如我们刚刚讨论的，Fault Tolerance 规范提供了一组注解，您需要将这些注解应用于类或方法以强制执行容错策略。话虽如此，您必须记住这些注解是拦截器绑定，因此仅适用于 CDI bean。所以，在将容错注解应用于它们或其方法之前，请务必小心地将您的类定义为 CDI beans。

在以下各节中，您将找到每个容错注解的用法示例。

# The @Asynchronous policy

使操作异步就像以下这样简单：

```java
@Asynchronous
public Future<Connection> service() throws InterruptedException {
  Connection conn = new Connection() {
    {
      Thread.sleep(1000);
    }

    @Override
    public String getData() {
      return "service DATA";
    }
 };
 return CompletableFuture.completedFuture(conn);
}
```

唯一的限制是`@Asynchronous`方法必须返回`Future`或`CompletionStage`；否则，实现应该抛出异常。

# `@Retry`策略

如果操作失败，你可以应用重试策略，以便再次调用操作。`@Retry`注解可以像这样应用于类或方法级别：

```java
@Retry(maxRetries = 5, maxDuration= 1000, retryOn = {IOException.class})public void operationToRetry() {  ...}
```

在前一个示例中，操作应该只在发生`IOException`时最多重试五次。如果所有重试的总持续时间超过 1,000 毫秒，则操作将被取消。

# `@Fallback`策略

`@Fallback`注解只能应用于方法；注释类将产生意外结果：

```java
@Retry(maxRetries = 2)
@Fallback(StringFallbackHandler.class)
public String shouldFallback() {
  ...
}
```

在达到重试次数后调用回退方法。在前一个示例中，如果出现错误，方法将重试两次，然后使用回退调用另一段代码——在这个例子中，下面的`StringFallbackHandler`类：

```java
import javax.enterprise.context.ApplicationScoped;

import org.eclipse.microprofile.config.inject.ConfigProperty;
import org.eclipse.microprofile.faulttolerance.ExecutionContext;
import org.eclipse.microprofile.faulttolerance.FallbackHandler;

@ApplicationScoped
public class StringFallbackHandler implements FallbackHandler<String> {
    @ConfigProperty(name="app1.requestFallbackReply", defaultValue = "Unconfigured Default Reply")
    private String replyString;

    @Override
    public String handle(ExecutionContext ec) {
        return replyString;
    }
}
```

可以通过实现`FallbackHandler`接口的类或当前 bean 中的方法定义回退代码（见前一个代码）。在`StringFallbackHandler`代码中，使用了名为`app1.requestFallbackReply`的 MicroProfile Config 属性来外部化应用程序的回退字符串值。

# `@Timeout`策略

`@Timeout`注解可以应用于类或方法，以确保操作不会永远持续：

```java
@Timeout(200)public void operationCouldTimeout() {  ...}
```

在前一个示例中，如果操作持续时间超过 200 毫秒，则将停止操作。

# `@CircuitBreaker`策略

`@CircuitBreaker`注解可以应用于类或方法。电路断路器模式由马丁·福勒引入，旨在通过使其在功能失调时快速失败来保护操作的执行：

```java
@CircuitBreaker(requestVolumeThreshold = 4, failureRatio=0.75, delay = 1000)
public void operationCouldBeShortCircuited(){
  ...
}
```

在前一个示例中，方法应用了`CircuitBreaker`策略。如果在四个连续调用滚动窗口中发生三次（*4 x 0.75*）失败，则电路将被打开。电路将在 1,000 毫秒后保持打开状态，然后回到半开状态。在一个成功的调用之后，电路将再次关闭。

# `@Bulkhead`策略

`@Bulkhead`注解也可以应用于类或方法以强制执行 bulkhead 策略。这种模式通过限制给定方法的同时调用次数来隔离当前操作中的故障，以保留其他操作的执行。

```java
@Bulkhead(4)public void bulkheadedOperation() {  ...}
```

在前一个代码中，这个方法只支持四个同时调用。如果超过四个同时请求进入`bulkheadedOperation`方法，系统将保留第五个及以后的请求，直到四个活动调用之一完成。bulkhead 注解还可以与`@Asynchronous`结合使用，以限制异步中的线程数...

# 微配置中的容错性

正如我们在前几节所看到的，Fault Tolerance 策略是通过使用注解来应用的。对于大多数用例来说，这已经足够了，但对于其他一些情况，这种方法可能不令人满意，因为配置是在源代码级别完成的。

这就是为什么 MicroProfile Fault Tolerance 注解的参数可以通过 MicroProfile Config 进行覆盖的原因。

注解参数可以通过使用以下命名约定来通过配置属性覆盖：`<classname>/<methodname>/<annotation>/<parameter>`。

要覆盖`MyService`类中`doSomething`方法的`@Retry`的`maxDuration`，请像这样设置配置属性：

```java
org.example.microservice.MyService/doSomething/Retry/maxDuration=3000
```

如果需要为特定类配置与特定注解相同的参数值，请使用`<classname>/<annotation>/<parameter>`配置属性进行配置。

例如，使用以下配置属性来覆盖`MyService`类上`@Retry`的所有`maxRetries`为 100：

```java
org.example.microservice.MyService/Retry/maxRetries=100
```

有时，需要为整个微服务（即在部署中所有注解的出现）配置相同的参数值。

在这种情况下，`<annotation>/<parameter>`配置属性将覆盖指定注解的相应参数值。例如，要覆盖所有`@Retry`的`maxRetries`为 30，请指定以下配置属性：

```java
Retry/maxRetries=30
```

这使我们结束了关于 MicroProfile 中的 Fault Tolerance 的讨论。

# 总结

在本章中，我们学习了如何使用 MicroProfile Config 来配置 MicroProfile 应用程序以及 MicroProfile Fault Tolerance 使其更具弹性。

在 MicroProfile Config 中，配置来源可以有很多；一些值来自属性文件，其他来自系统属性或环境变量，但它们都是从 Java 应用程序一致访问的。这些值可能会根据部署环境（例如，测试和生产）而有所不同，但在应用程序代码中这是透明的。

MicroProfile Fault Tolerance 通过在代码中应用特定策略来防止应用程序失败。它带有默认行为，但可以通过 MicroProfile 进行配置...

# 问题

1.  MicroProfile Config 支持哪些默认的配置属性来源？

1.  如果您需要将另一个配置属性源集成在一起，您可以做什么？

1.  只支持字符串类型的属性吗？

1.  将配置属性注入代码是否迫使您为该属性提供值？

1.  假设您有复杂的属性类型。是否有方法将它们集成到 MicroProfile Config 中？

1.  当一个 Fault Tolerance 注解应用于一个类时会发生什么？

1.  真或假：至少有 10 种不同的 Fault Tolerance 策略？

1.  `@Retry`策略是否需要在所有失败上进行重试？

1.  我们是否必须使用应用程序代码中使用的 Fault Tolerance 注解设置？

# 进一步阅读

关于 MicroProfile Config 特性的更多详细信息，可以在[`github.com/eclipse/microprofile-config/releases`](https://github.com/eclipse/microprofile-config/releases)的 MicroProfile Config 规范中找到。关于 MicroProfile Fault Tolerance 特性的更多详细信息，可以在[`github.com/eclipse/microprofile-fault-tolerance/releases`](https://github.com/eclipse/microprofile-fault-tolerance/releases)的 MicroProfile Config 规范中找到。
