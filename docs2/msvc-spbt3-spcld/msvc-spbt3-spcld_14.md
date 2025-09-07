# 14

# 理解分布式追踪

在本章中，我们将学习如何使用分布式追踪更好地理解我们的微服务如何协作，例如，在完成发送到外部 API 的请求时。能够利用分布式追踪对于管理协作微服务的系统景观至关重要。正如已在*第八章*，*Spring Cloud 简介*中描述的那样，Micrometer Tracing 将被用于收集追踪信息，而 Zipkin 将被用于存储和可视化这些追踪信息。

在本章中，我们将学习以下主题：

+   使用 Micrometer Tracing 和 Zipkin 引入分布式追踪。

+   如何将分布式追踪添加到源代码中。

+   如何以编程方式向追踪添加信息。

+   如何执行分布式追踪，可视化成功和失败的 API 请求。我们将看到同步和异步处理如何被可视化。

# 技术要求

关于如何安装本书中使用的工具以及如何访问本书源代码的说明，请参阅：

+   *第二十一章*，*macOS 的安装说明*

+   *第二十二章*，*使用 WSL 2 和 Ubuntu 的 Microsoft Windows 安装说明*

本章中的代码示例均来自`$BOOK_HOME/Chapter14`的源代码。

如果你想查看本章源代码中应用的变化，即查看如何使用 Micrometer Tracing 和 Zipkin 添加分布式追踪，你可以将其与*第十三章*，*使用 Resilience4j 提高弹性*的源代码进行比较。你可以使用你喜欢的`diff`工具比较两个文件夹，`$BOOK_HOME/Chapter13`和`$BOOK_HOME/Chapter14`。

# 使用 Micrometer Tracing 和 Zipkin 引入分布式追踪

回顾*第八章*，*Spring Cloud 简介*中关于*使用 Micrometer Tracing 和 Zipkin 进行分布式追踪*的部分，一个完整工作流程的追踪信息被称为**追踪**或**追踪树**，而树中的子部分，例如工作的基本单元，被称为**跨度**。跨度可以由形成追踪树的子跨度组成。可以在追踪及其跨度中添加元数据，作为键值对称为**标签**。Zipkin UI 可以如下可视化追踪树及其跨度：

![图形用户界面，应用程序描述自动生成](img/B19825_14_01.png)

图 14.1：带有其跨度的追踪示例

Micrometer 跟踪用于收集跟踪信息，在调用其他微服务时传播跟踪上下文（例如，跟踪和跨度 ID），并将跟踪信息导出到像 Zipkin 这样的跟踪分析工具。跟踪信息的处理是在底层由**跟踪器**完成的。Micrometer 支持基于**OpenTelemetry**([`opentelemetry.io/`](https://opentelemetry.io/))或**OpenZipkin Brave**([`github.com/openzipkin/brave`](https://github.com/openzipkin/brave))的自动配置跟踪器。将跟踪信息导出到跟踪分析工具是由**报告器**完成的。

默认情况下，跟踪头通过使用**W3C 跟踪上下文**头（[`www.w3.org/TR/trace-context/`](https://www.w3.org/TR/trace-context/))在微服务之间传播，最重要的是`traceparent`头，但也可以配置为使用 OpenZipkin 的`B3`头。在本章中，我们将使用 W3C 跟踪上下文头。在*第十八章*中，我们将使用`B3`头。

一个示例 W3C 跟踪上下文`traceparent`头看起来像这样：

```java
traceparent:"00-2425f26083814f66c985c717a761e810-fbec8704028cfb20-01" 
```

`traceparent`头的值包含四个部分，由`-`分隔：

+   `00`，表示使用的版本。根据当前规范，始终是“`00`"。

+   `124…810`是跟踪 ID。

+   `fbe…b20`是跨度 ID。

+   `01`，最后一部分包含各种标志。当前规范支持的唯一标志是名为`sampled`的标志，其值为`01`。这意味着调用者正在记录此请求的跟踪数据。我们将配置我们的微服务以记录所有请求的跟踪数据，因此此标志始终具有`01`的值。

使用 OpenZipkin Brave `B3`头的样子如下：

```java
X-B3-TraceId:"64436ea679e8eb6e6fa028bb3459e703"
X-B3-SpanId:"120678270898ddd5"
X-B3-ParentSpanId:"3c431d3d01987c22"
X-B3-Sampled:"1" 
```

头名称是自解释的，我们可以看到头不仅提供了跟踪和跨度 ID，还提供了父跨度 ID。

跟踪和跨度由 Spring Boot 自动为传入流量创建，无论是传入的 HTTP 请求还是 Spring Cloud Stream 接收到的消息。如果传入请求包含跟踪 ID，它将在创建跨度时使用；如果没有，将创建一个新的跟踪 ID。跟踪和跨度 ID 将自动传播到传出流量，无论是作为 HTTP 请求还是通过使用 Spring Cloud Stream 发送消息。

如果需要，可以通过编程方式添加额外的跟踪信息，无论是通过添加自定义跨度，还是通过向由微服务创建的所有跨度添加自定义标签。这是通过使用**Micrometer 可观察性**([`micrometer.io/docs/observation`](https://micrometer.io/docs/observation))及其`Observation` API 来完成的。

Micrometer 跟踪的初始版本与 Spring Boot 3 一起发布时，在支持反应式客户端的分布式跟踪方面存在一些限制。这影响了本书中使用的底层使用 Project Reactor 的微服务。在*添加对反应式客户端缺乏支持的解决方案*部分，我们将学习如何缓解这些不足。

Zipkin 内置了对存储跟踪信息原生的支持，无论是存储在内存中，还是存储在 Apache Cassandra、Elasticsearch 或 MySQL 等数据库中。除此之外，还有许多扩展可用。有关详细信息，请参阅 [`zipkin.io/pages/extensions_choices.html`](https://zipkin.io/pages/extensions_choices.html)。在本章中，我们将存储跟踪信息在内存中。

在引入 Micrometer 跟踪和 Zipkin 之后，让我们看看在源代码中需要做出哪些更改才能启用分布式跟踪。

# 将分布式跟踪添加到源代码

在本节中，我们将学习如何更新源代码以启用分布式跟踪。这可以通过以下步骤完成：

1.  将依赖项添加到构建文件中，以引入带有跟踪器实现和报告器的 Micrometer 跟踪。

1.  将 Zipkin 服务器添加到 Docker Compose 文件中。

1.  配置微服务以将跟踪信息发送到 Zipkin。

1.  为缺乏对反应式客户端的支持添加解决方案。

1.  在现有跨度中添加创建自定义跨度自定义标签的代码。

我们将依次介绍每个步骤。

## 将依赖项添加到构建文件中。

为了能够利用 Micrometer 跟踪和将跟踪信息导出到 Zipkin 的能力，我们需要将所选跟踪器和报告器的依赖项添加到 Gradle 项目构建文件 `build.gradle` 中。

这通过添加以下两行来完成：

```java
implementation 'io.micrometer:micrometer-tracing-bridge-otel'
implementation 'io.opentelemetry:opentelemetry-exporter-zipkin' 
```

对于审查服务，还添加了一个依赖项以启用有关 SQL 数据库操作的跟踪信息。它看起来像：

```java
implementation 'net.ttddyy.observation:datasource-micrometer-spring-boot:1.0.0' 
```

这个库可以为审查服务执行的 SQL 操作创建跨度。这些跨度将包含有关已执行的 SQL 查询及其执行时间的信息。

## 添加 Micrometer 跟踪和 Zipkin 的配置

使用 Micrometer 跟踪和 Zipkin 的配置已添加到通用配置文件 `config-repo/application.yml` 中。在默认配置文件中，指定了跟踪信息将通过以下 URL 发送到 Zipkin：

```java
management.zipkin.tracing.endpoint: http://zipkin:9411/api/v2/spans 
```

默认情况下，Micrometer 跟踪只发送 10% 的跟踪到 Zipkin。为了确保所有跟踪都发送到 Zipkin，以下属性被添加到默认配置文件中：

```java
management.tracing.sampling.probability: 1.0 
```

我们还希望将跟踪和跨度 ID 写入日志；这将使我们能够关联来自协作微服务的日志输出，例如，满足发送到外部 API 的请求。

我们将在 *第十九章* 中研究如何使用它，即使用 EFK 栈进行集中式日志记录。

我们可以通过指定以下日志格式来在日志输出中包含跟踪和跨度 ID：

```java
logging.pattern.level: "%5p [${spring.application.name:},%X{traceId:-},%X{spanId:-}]" 
```

使用上述日志格式，日志输出将如下所示：

```java
2023-04-22T14:02:07.417Z  INFO [product-composite,01234,56789] 
```

`product-composite` 是微服务的名称，`01234` 是跟踪 ID，`56789` 是跨度 ID。

为了减少日志输出，我们还更改了 `config-repo` 中每个微服务的配置文件的日志级别，从 `DEBUG` 更改为 `INFO`。这使得验证跟踪和跨度 ID 是否按预期添加变得更加容易。此更改通过以下行应用：

```java
 se.magnus: INFO 
```

对于 `product-composite` 微服务，由于相同的原因，`HttpWebHandlerAdapter` 类的日志级别已从 `TRACE` 更改为 `INFO`：

```java
org.springframework.web.server.adapter.HttpWebHandlerAdapter: INFO 
```

## 将 Zipkin 添加到 Docker Compose 文件中

要将 Zipkin 服务器作为 Docker 容器运行，我们将使用 Zipkin 项目发布的 Docker 镜像 `openzipkin/zipkin`。有关详细信息，请参阅 [`hub.docker.com/r/openzipkin/zipkin`](https://hub.docker.com/r/openzipkin/zipkin)。Zipkin 服务器的定义如下：

```java
 zipkin:
    image: openzipkin/zipkin:2.24.0
    restart: always
    mem_limit: 1024m
    environment:
      - STORAGE_TYPE=mem
    ports:
      - 9411:9411 
```

让我们解释一下前面的源代码：

+   Docker 镜像 `openzipkin/zipkin` 的版本指定为 2.24.0。

+   使用 `STORAGE_TYPE=mem` 环境变量来指定 Zipkin 将在内存中保留所有跟踪信息。

+   Zipkin 的内存限制增加到 1,024 MB，而所有其他容器的内存限制为 512 MB。原因是 Zipkin 被配置为在内存中保留所有跟踪信息，因此一段时间后它将比其他容器消耗更多的内存。

+   如果 Zipkin 碰巧耗尽内存并停止，我们已经应用了一个重启策略，要求 Docker 引擎始终重启容器。这既适用于容器本身崩溃的情况，也适用于 Docker 引擎重启的情况。

+   Zipkin 将 HTTP 端口 `9411` 暴露给网络浏览器，以便访问其网络用户界面。

虽然在开发和测试活动中将跟踪信息存储在 Zipkin 的内存中是可以的，但 Zipkin 应该配置为在生产环境中将跟踪信息存储在数据库中。

## 添加对缺乏反应式客户端支持的解决方案

如上所述，当前版本的 Spring Boot、Project Reactor 和 Micrometer Tracing 还未完全协同工作。因此，已经对反应式客户端的源代码应用了一些解决方案。也就是说，四个微服务和网关。问题主要与在反应式异步处理中涉及的不同线程之间传播跟踪上下文（例如跟踪和跨度 ID）的复杂性有关，特别是如果处理的部分涉及命令式同步处理。

如果一个请求的所有处理都使用同步实现，即使用同一个线程进行所有处理，那么传播跟踪上下文不是问题。可以使用 `ThreadLocal` 变量来存储跟踪上下文。由于所有代码都在同一个线程中运行，因此可以在实现中的任何地方从 `ThreadLocal` 变量中检索跟踪上下文。

这种反应式和命令式处理混合的例子是 `review` 微服务的实现，其中对底层 SQL 数据库的调用是同步进行的。

如果你想更详细地了解挑战，请参阅 Spring 团队发布的“*统一连接反应式和非反应式*”三篇博客系列。它可以在以下链接找到：[`spring.io/blog/2023/03/30/context-propagation-with-project-reactor-3-unified-bridging-between-reactive`](https://spring.io/blog/2023/03/30/context-propagation-with-project-reactor-3-unified-bridging-between-reactive)。

值得注意的是，Micrometer Tracing 的前身 Spring Cloud Sleuth 比当前版本的 Spring Boot、Project Reactor 和 Micrometer Tracing 对反应式客户端的支持更好。希望即将推出的版本将很快解决这些不足。

如果你有一个已经使用 Spring Cloud Sleuth 的代码库，你可以在以下链接找到迁移指南：[`github.com/micrometer-metrics/tracing/wiki/Spring-Cloud-Sleuth-3.1-Migration-Guide`](https://github.com/micrometer-metrics/tracing/wiki/Spring-Cloud-Sleuth-3.1-Migration-Guide)。

为了解决上下文传播的许多挑战，我们可以在反应式客户端的`main()`方法中调用`Hooks.enableAutomaticContextPropagation()`方法来开启自动上下文传播。有关详细信息，请参阅上述提到的“*统一连接反应式和非反应式*”博客系列。对于`product-composite`服务，它看起来是这样的：

```java
public static void main(String[] args) {
  Hooks.enableAutomaticContextPropagation();
  SpringApplication.run(ProductCompositeServiceApplication.class, args);
} 
```

然而，对于`product-composite`服务，仍然存在一个问题。为了确保`WebClient`实例被正确地用于观察，例如，能够将当前的跟踪和跨度 ID 作为出站请求的头部传播，期望使用自动装配将`WebClient.Builder`实例注入。不幸的是，当使用 Eureka 进行服务发现时，建议将`WebClient.Builder`实例创建为一个带有`@LoadBalanced`注解的 bean，如下所示：

```java
 @Bean
  @LoadBalanced
  public WebClient.Builder loadBalancedWebClientBuilder() {
    return WebClient.builder();
  } 
```

因此，在使用 Eureka 和 Micrometer Tracing 时创建`WebClient`实例的方式存在冲突。为了解决这个冲突，可以将`@LoadBalanced` bean 替换为一个负载均衡器感知的交换过滤器函数`ReactorLoadBalancerExchangeFilterFunction`。可以在自动装配的`WebClient.Builder`实例上设置交换过滤器函数，如下所示：

```java
 @Autowired
  private ReactorLoadBalancerExchangeFilterFunction lbFunction;
  @Bean
  public WebClient webClient(WebClient.Builder builder) {
    return builder.filter(lbFunction).build();
  } 
```

这意味着应用程序类`ProductCompositeServiceApplication`注册了一个`WebClient` bean 而不是`WebClient.Builder` bean。这影响了`ProductCompositeIntegration`类；现在它需要自动装配一个`WebClient` bean 而不是`WebClient.Builder` bean。

要访问`ReactorLoadBalancerExchangeFilterFunction`函数，需要在构建文件`build.gradle`中添加对`org.springframework.cloud:spring-cloud-starter-loadbalancer`的依赖。

## 向现有跨度添加自定义跨度以及自定义标签

除了依赖于内置的创建跟踪和跨度的支持，我们还可以使用 Micrometer Observability 提供的 `Observation` API，例如，添加我们自己的跨度或向由微服务创建的现有跨度添加自定义标签。`Observation` API 是在具有相同名称的 Java 接口后面实现的。

让我们先看看如何添加自定义跨度，然后看看我们如何可以向由微服务创建的所有跨度添加自定义标签。

要使用 `Observation` API 进行任何观察，观察对象需要在一个 `ObservationRegistry` bean 中注册。可以使用自动装配，如以下示例所示：

```java
@Component
public class MyComponent {
  private final ObservationRegistry registry;
  public MyComponent(ObservationRegistry registry) {
    this.registry = registry;
  } 
```

### 添加自定义跨度

要添加自定义的跨度，`Observation` 接口提供了一个静态方法 `createNotStarted()`，可以用来创建跨度。要执行跨度，可以使用 `observe()` 方法提供跨度应覆盖的代码。如果代码不返回任何值，它可以指定为一个 `Runnable` 函数；否则，需要指定为一个 `Supplier` 函数。

创建用于执行最小化 `Supplier` 函数的自定义跨度的示例如下：

```java
 int y = Observation.createNotStarted("my observation", registry)
      .observe(() -> {
        int x = 1;
        LOG.info("Will return {}", x);
        return 1;
      });
    LOG.info("Got {}", y); 
```

`registry` 参数是一个类型为 `ObservationRegistry` 的 bean，并且如上所述自动装配。

执行此代码将产生如下日志输出：

```java
2023-04-22T14:02:07.417Z  INFO [product-composite,9761b2b2b2da59c5096e78252c48ab3d,d8bcbd9cde9fe2d7] 1 --- [     parallel-6] s.m.m.c.p.s.ProductCompositeServiceImpl  : Will return 1
2023-04-22T14:02:07.417Z  INFO [product-composite,9761b2b2b2da59c5096e78252c48ab3d,4c8ea2820fb74ec9] 1 --- [     parallel-6] s.m.m.c.p.s.ProductCompositeServiceImpl  : Got 1 
```

从日志输出中，我们可以看到这两个日志语句都引用了相同的跟踪 ID，`9761b2b2b2da59c5096e78252c48ab3d`，但指定了不同的跨度 ID，`d8bcbd9cde9fe2d7` 是自定义跨度的跨度 ID！

如果我们想添加有关跨度的元数据，我们可以通过添加上下文名称和一组键值对形式的标签来指定。上下文名称将是跨度的名称，并且可以在 Zipkin 中可视化跟踪树时用来识别跨度。如果信息的可能值是有限的，限制在有限数量的替代方案中，则应使用 `lowCardinalityKeyValue()` 方法指定标签。对于无界值，应使用 `highCardinalityKeyValue()` 方法。调用这些方法将在当前跨度中添加标签，而低基数值也将被添加到由观察创建的指标中。

我们将研究如何在**第二十章**，监控微服务中，使用指标。

将信息指定为标签的示例如下：

```java
 int y = Observation.createNotStarted("my observation", registry)
      .contextualName("product info")
      .lowCardinalityKeyValue("template-url", "/product-composite/{productId}")
      .highCardinalityKeyValue("actual-url", "/product-composite/12345678")
      .observe(() -> {
        int x = 1;
        LOG.info("Will return {}", x);
        return x;
      });
    LOG.info("Got {}", y); 
```

从上面的示例中，我们可以看到：

+   上下文名称设置为 `"product info"`。

+   使用 `lowCardinalityKeyValue()` 方法指定了一个只有几个可能值的键，`"template-url"`。在我们的案例中，它为创建、获取和删除方法提供了三个可能的值。

+   使用 `highCardinalityKeyValue()` 方法指定了一个键 `"actual-url"`，其值数量无限，取决于指定的 `productId`。

让我们将此应用于 `product-composite` 服务，创建一个提供 `productId` 作为标签的自定义跨度。由于产品 ID 的数量是无限的，我们将使用 `highCardinalityKeyValue()` 方法指定它。我们将创建一个指定每个三个 API 方法（创建、检索和删除复合产品）的当前 `productId` 的跨度。包含一个高基数标签的自定义跨度的创建由实用类 `ObservationUtil` 处理。实用类位于包 `se.magnus.microservices.composite.product.services.tracing` 中。该类中的实用方法 `observe()` 看起来是这样的：

```java
public <T> T observe(String observationName, String contextualName, String highCardinalityKey, String highCardinalityValue, Supplier<T> supplier) {
  return Observation.createNotStarted(observationName, registry)
    .contextualName(contextualName)
    .highCardinalityKeyValue(highCardinalityKey, highCardinalityValue)
    .observe(supplier);
} 
```

`observe()` 方法包装了对 `Observation.createNotStarted()` 方法的调用。该方法的用法在上面的示例中已解释，因此无需进一步解释。

这个实用方法由 `ProductCompositeServiceImpl` 类中的辅助方法 `observationWithProductInfo()` 使用，该方法将常用值应用于 `ProductCompositeServiceImpl` 类：

```java
private <T> T observationWithProductInfo(int productInfo, Supplier<T> supplier) {
  return observationUtil.observe(
    "composite observation",
    "product info",
    "productId",
    String.valueOf(productInfo),
    supplier);
} 
```

最后，辅助方法被三个 API 方法 `createProduct()`、`getProduct()` 和 `deleteProduct()` 使用。自定义跨度是通过在每个方法中包装现有代码创建的。现有代码已被移动到相应的“内部”方法以简化解决方案的结构。“内部”方法由 `observationWithProductInfo()` 方法调用。对于 `getProduct()`，实现现在看起来是这样的：

```java
 public Mono<ProductAggregate> getProduct(int productId, ...) {
    return observationWithProductInfo(productId,
      () -> getProductInternal(productId, ...));
  }
  private Mono<ProductAggregate> getProductInternal(int productId, ...) {
    return observationWithProductInfo(productId, () -> {
      LOG.info("Will get composite product info for product.id={}", productId);
      return Mono.zip(
          values -> createProductAggregate(...
          integration.getProduct(productId, ...),
          integration.getRecommendations(productId).collectList(),
          integration.getReviews(productId).collectList())
          ...);
  } 
```

如果与第十三章中的相应实现进行比较，我们可以看到创建自定义跨度所需的变化仅限于添加一个新的“*内部*”方法，并从 `observationWithProductInfo()` 方法调用它：

```java
 public Mono<ProductAggregate> getProduct(int productId, ...) {
    LOG.info("Will get composite product info for product.id={}", productId);
    return Mono.zip(
      values -> createProductAggregate(...
      integration.getProduct(productId, ...),
      integration.getRecommendations(productId).collectList(),
      integration.getReviews(productId).collectList())
      ...);
  } 
```

因此，通过提供一个适当的实用方法来处理设置自定义跨度的细节，我们可以通过非常小的更改将自定义跨度添加到现有代码中。当我们在本章后面尝试分布式跟踪时，我们将看到这个自定义跨度的实际应用。有了自定义跨度，让我们看看我们如何向微服务中创建的任何跨度添加自定义标签！

### 向现有跨度添加自定义标签

如果我们想向由微服务创建的所有跨度添加一些自定义信息，我们可以使用 `ObservationFilter`。它需要使用 `ObservationRegistryCustomizer` Bean 在 `ObservationRegistry` Bean 中注册。

让我们应用一个过滤器，将 `product-composite` 微服务的当前版本注册为标签，应用于它创建的每个跨度。我们需要做以下事情：

+   更新构建文件，以便 Gradle 创建构建信息，包括在 `build.gradle` 文件中由 `version` 属性指定的当前版本。

+   创建一个过滤器，将当前版本作为低基数标签添加到所有跨度中。

+   创建一个注册配置 Bean，用于注册该过滤器。

要使 Gradle 创建构建信息，以下内容被添加到构建文件 `build.gradle` 中：

```java
springBoot {
    buildInfo()
} 
```

当执行`./gradlew build`命令时，这将导致创建文件`build/resources/main/META-INF/build-info.properties`。此文件将指定当前版本为：

```java
build.version=1.0.0-SNAPSHOT 
```

构建信息文件将被捆绑到微服务的 JAR 文件中，其信息可以通过`BuildProperties` Bean 访问。

过滤器看起来像这样：

```java
public class BuildInfoObservationFilter implements ObservationFilter {
  private final BuildProperties buildProperties;
  public BuildInfoObservationFilter(BuildProperties buildProperties) {
    this.buildProperties = buildProperties;
  }
  @Override
  public Observation.Context map(final Observation.Context context) {
    KeyValue buildVersion = KeyValue.of("build.version", buildProperties.getVersion());
    return context.addLowCardinalityKeyValue(buildVersion);
  }
} 
```

从上面的源代码中，我们可以看到：

+   一个`BuildProperties` Bean 被注入到过滤器的构造函数中。

+   过滤器的`map()`方法从`BuildProperties` Bean 中检索微服务版本，并将其设置为提供的观察上下文上的低基数标签。

注册配置 Bean 看起来像这样：

```java
@Configuration(proxyBeanMethods = false)
public class ObservationRegistryConfig implements ObservationRegistryCustomizer<ObservationRegistry> {
  private final BuildProperties buildProperties;
  public ObservationRegistryConfig(BuildProperties buildProperties) {
    this.buildProperties = buildProperties;
  }
  @Override
  public void customize(final ObservationRegistry registry) {
    registry.observationConfig().observationFilter(new BuildInfoObservationFilter(buildProperties));
  }
} 
```

从上面的源代码中，我们可以了解到：

+   `BuildProperties` Bean 也被注入到配置类的构造函数中。

+   在`customize()`方法中，创建并注册了过滤器。过滤器还在这里注入了`BuildProperties` Bean。

过滤器和注册配置 Bean 可以在`se.magnus.microservices.composite.product.services.tracing`包中找到。当我们在本章后面尝试分布式跟踪时，我们将看到这个观察过滤器的作用。

关于处理自定义跨度更多的方式，例如，设置何时应用观察过滤器的谓词或使用注解来描述观察，请参阅[`micrometer.io/docs/observation`](https://micrometer.io/docs/observation)。

这就是使用 Micrometer Tracing 和 Zipkin 添加分布式跟踪所需的所有步骤，所以让我们在下一节中尝试一下！

# 尝试分布式跟踪

在源代码中进行必要的更改后，我们可以尝试分布式跟踪。我们将通过以下步骤来完成：

1.  构建、启动和验证系统景观。

1.  发送一个成功的 API 请求并查看在 Zipkin 中与该 API 请求相关的跟踪信息。

1.  发送一个失败的 API 请求并查看我们可以找到的错误信息。

1.  发送一个触发异步处理的成功 API 请求并查看其跟踪信息是如何表示的。

我们将在接下来的章节中详细讨论这些步骤。

## 启动系统景观

让我们启动系统景观。使用以下命令构建 Docker 镜像：

```java
cd $BOOK_HOME/Chapter14
./gradlew build && docker-compose build 
```

在 Docker 中启动系统景观并使用以下命令运行常规测试：

```java
./test-em-all.bash start 
```

在我们能够调用 API 之前，我们需要一个访问令牌。运行以下命令以获取访问令牌：

```java
unset ACCESS_TOKEN
ACCESS_TOKEN=$(curl -k https://writer:secret-writer@localhost:8443/oauth2/token -d grant_type=client_credentials -d scope="product:read product:write" -s | jq -r .access_token)
echo $ACCESS_TOKEN 
```

如前几章所述，授权服务器颁发的访问令牌有效期为一个小时。因此，如果你过一会儿开始收到`401 未授权`错误，可能就是时候获取一个新的访问令牌了。

## 发送一个成功的 API 请求

现在，我们已经准备好向 API 发送正常请求。运行以下命令：

```java
curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/1 -w "%{http_code}\n" -o /dev/null -s 
```

期望命令返回成功的 HTTP 状态码`200`。

我们现在可以启动 Zipkin UI 来查看已发送到 Zipkin 的跟踪信息：

1.  在您的网络浏览器中打开以下 URL：`http://localhost:9411/zipkin/`。

1.  要找到我们请求的跟踪信息，我们可以搜索通过`gateway`服务经过的跟踪。执行以下步骤：

    1.  点击大加号（红色背景上的白色**+**号）并选择**serviceName**然后**gateway**。

    1.  点击**RUN****QUERY**按钮。

    1.  点击**开始时间**标题以按最新顺序查看结果（**开始时间**标题左侧应可见向下箭头）。

查找跟踪的响应应类似于以下截图：

![计算机截图描述自动生成](img/B19825_14_02.png)

图 14.2：使用 Zipkin 搜索分布式跟踪

1.  我们先前 API 请求的跟踪信息是列表中的第一个。点击其**SHOW**按钮以查看与跟踪相关的详细信息：

![图形用户界面，应用程序描述自动生成](img/B19825_14_03.png)

图 14.3：在 Zipkin 中可视化的示例分布式跟踪

在详细的跟踪信息视图中，我们可以观察到以下内容：

1.  请求被`gateway`服务接收。

1.  `gateway`服务将请求的处理委托给了`product-composite`服务。

1.  `product-composite`服务反过来向核心服务发送了三个并行请求：`product`、`recommendation`和`review`。查看名为**product-composite: http get**的跨度。

1.  一旦`product-composite`服务从所有三个核心服务收到响应，它就创建了一个组合响应并通过`gateway`服务将其发送回调用者。

1.  在上一节中创建的自定义跨度名为**product-composite: product info**。点击它以查看其标签。在右侧的详细信息视图中，我们可以看到由自定义跨度创建的标签**productId = 1**和由观察过滤器创建的标签**build.version = 1.0.0-SNAPSHOT**。

1.  为了验证由观察过滤器创建的标签是否按预期工作，点击由`product-composite`服务创建的其他跨度，并验证**build.version**是否存在。

1.  选择名为**review: query**的跨度以查看由`review`微服务的数据库层报告的跨度：

![图形用户界面描述自动生成](img/B19825_14_04.png)

图 14.4：描述 SQL 查询执行的跨度

1.  在跨度的标签列表中，我们可以看到其实际执行的 SQL 查询。我们还可以看到其执行时间为 0.8 毫秒。非常有价值的信息！

为了更好地理解跟踪和跨度 ID 在微服务之间是如何传播的，我们可以更改`product-composite`服务的日志配置，以便将出站请求的 HTTP 头写入其日志。这可以通过以下步骤实现：

1.  将以下两行添加到配置文件`config-repo/product-composite.yml`中：

    ```java
    spring.codec.log-request-details: true
    logging.level.org.springframework.web.reactive.function.client.ExchangeFunctions: TRACE 
    ```

1.  这两行已经存在于配置文件中，但已被注释掉。它们前面有以下注释：

    ```java
    # To see tracing headers, uncomment the following two lines and restart the product-composite service 
    ```

1.  在配置文件中找到前面的注释，并取消注释其下方的两行。

1.  之后，重新启动`product-composite`服务：

    ```java
    docker-compose restart product-composite 
    ```

1.  显示`product-composite`服务的日志输出：

    ```java
    docker-compose log -f --tail 0 product-composite 
    ```

1.  重新运行上述`curl`请求，你将看到包含上述提到的`traceparent` HTTP 头部的日志输出。例如，发送到`recommendation`服务的请求：

    ```java
    chapter14-product-composite-1  | 2023-04-23T09:24:50.849Z TRACE [product-composite,e1420dcc38901378e888b8ce7022510e,06867b65cf84b552] 1 --- [     parallel-2] o.s.w.r.f.client.ExchangeFunctions       : [14606b71] HTTP GET http://d40874197b77:8080/recommendation?productId=1, headers=[traceparent:"00-e1420dcc38901378e888b8ce7022510e-06867b65cf84b552-01"] 
    ```

1.  在示例日志输出中，我们可以看到`traceparent` HTTP 头部的值，其中跟踪 ID 设置为`e1420dcc38901378e888b8ce7022510e`，而跨度 ID 设置为`06867b65cf84b552`。

1.  如果你不想保留`traceparent` HTTP 头部的日志记录，请在`config-repo/product-composite.yml`中注释掉两行，并重新启动`product-composite`服务。

## 发送一个不成功的 API 请求

让我们看看如果我们发起一个失败的 API 请求，跟踪信息会是什么样子；例如，搜索一个导致超时的产品：

1.  发送一个针对产品 ID `1`的 API 请求，并强制延迟三秒，这将触发时间限制器，并验证它返回的 HTTP 状态码为`500`：

    ```java
    curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/1?delay=3 -w "%{http_code}\n" -o /dev/null -s 
    ```

1.  在 Zipkin UI 中，返回到搜索页面（使用网络浏览器的后退按钮），然后再次点击**RUN QUERY**按钮。要按最新顺序查看结果，请点击**Start Time**标题。

预期结果将与以下截图类似：

![计算机截图  自动生成的描述](img/B19825_14_05.png)

图 14.5：使用 Zipkin 查找失败的请求

1.  你应该会在返回列表的顶部看到失败的请求。注意，其持续时间条是红色的，表示发生了错误。点击其**SHOW**按钮查看详细信息：

![计算机错误截图  自动生成的描述](img/B19825_14_06.png)

图 14.6：使用 Zipkin 查看失败的请求的跟踪

在这里，我们可以看到一个带有错误符号的跨度，**product-composite: secured request**（一个带有感叹号的红色圆圈）。

1.  点击跨度以查看其标签。你将找到一个名为**error**的标签，它清楚地表明错误是由两秒后发生的超时引起的。

## 发送一个触发异步处理的 API 请求

在 Zipkin UI 中，有趣的是看到第三种请求类型，这种请求的部分处理是异步完成的。让我们尝试一个删除请求，其中核心服务的删除过程是异步完成的。

`product-composite`服务通过消息代理向三个核心服务的每个服务发送删除事件，每个核心服务都会接收到删除事件并异步处理它。多亏了 Micrometer Tracing，跟踪信息被添加到发送到消息代理的事件中，从而形成了对删除请求总处理的连贯视图。

执行以下步骤：

1.  运行以下命令以删除产品 ID 为`12345`的产品，并验证它返回请求被接受的 HTTP 状态码`202`：

    ```java
    curl -X DELETE -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/12345 -w "%{http_code}\n" -o /dev/null -s 
    ```

记住，删除操作是幂等的，也就是说，即使产品不存在，它也会成功！

1.  在 Zipkin UI 中，返回搜索页面（使用网络浏览器的后退按钮）并再次点击**运行查询**按钮。要按最新顺序查看结果，请点击**开始时间**标题。预期结果如下截图所示：

![计算机屏幕截图  自动生成描述](img/B19825_14_07.png)

图 14.7：使用 Zipkin 查找删除请求

1.  您应该在返回列表的顶部看到删除请求。请注意，根服务名称**gateway**由使用的 HTTP 方法**delete**后缀。点击其**显示**按钮以查看详细信息：

![计算机屏幕截图  自动生成描述](img/B19825_14_08.png)

图 14.8：使用 Zipkin 查看删除请求

在这里，我们可以看到处理删除请求的跟踪信息：

1.  请求被`gateway`服务接收。

1.  `gateway`服务将请求的处理委托给了`product-composite`服务。

1.  如预期，`product-composite`服务创建了一个名为**product-composite: product info**的自定义 span。

1.  然后，`product-composite`服务在消息代理（在本例中为 RabbitMQ）上发布了三个事件。查看以**send**结尾的 span。

1.  `product-composite`服务现在已完成，并通过`gateway`服务返回给调用者一个 HTTP `成功`状态码，`200`。请注意，这是在核心服务完成所有处理之前完成的！

1.  核心服务（`product`、`recommendation`和`review`）接收`delete`事件并开始异步处理它们，即相互独立。查看以**receive**结尾的 span。

1.  要确认消息代理的参与，请点击第一个产品 span：![计算机屏幕截图  自动生成描述](img/B19825_14_09.png)

    图 14.9：使用 Zipkin 查看事件异步处理的信息

    选定的 span 包含一个名为**peer.service**的标签，揭示了 RabbitMQ 的使用，而标签**spring.rabbit.listener.id**指出消息由消费者组**productsGroup**接收。

1.  最后，为了在通过 RabbitMQ 发送的消息中查看`traceparent`头，我们可以检查存储在产品审计队列中的消息。在您的网页浏览器中打开[`localhost:15672/#/queues/%2F/products.auditGroup`](http://localhost:15672/#/queues/%2F/products.auditGroup)。

1.  点击名为**Get Messages(s)**的按钮以查看队列中最旧的消息。网页应如下所示：

![图形用户界面，文本，应用程序  自动生成的描述](img/B19825_14_10.png)

图 14.10：RabbitMQ 中的带有 traceparent 头的消息

1.  在**属性**部分，您将找到此消息的**traceparent**头。在这种情况下，追踪 ID 是**99a9f2501e4d454643184c6b1cb0a232**，跨度 ID 是**7458430fe56d6df1**。

这完成了本章的分布式追踪测试！

Zipkin UI 包含更多用于查找感兴趣追踪的功能！

为了更熟悉 Zipkin UI，通过点击加号并选择**tagQuery**来尝试查询功能。例如，为了找到在`403 - Forbidden`错误上失败的请求，将其值设置为`tagQuery=http.status_code=403`，搜索在禁止（`403`）错误上失败的追踪。还可以尝试通过点击**RUN QUERY**按钮右侧的齿轮图标来设置回溯范围（开始和结束时间）和最大命中数限制。

通过关闭系统环境来总结测试。运行以下命令：

```java
docker-compose down 
```

# 摘要

在本章中，我们学习了如何使用分布式追踪来理解我们的微服务是如何协作的。我们学习了如何使用 Micrometer Tracing 来收集追踪信息，以及使用 Zipkin 来存储和可视化追踪信息。

我们看到了如何通过在构建文件中添加几个依赖项并设置一些配置参数来将 Micrometer Tracing 添加到微服务中。目前，反应式微服务尚未完全支持，但我们学习了如何解决最重要的问题。如果需要，我们可以使用`Observation` API 来创建自定义跨度或为微服务创建的所有跨度添加标签。我们还看到了 Zipkin UI 如何使识别复杂工作流中导致意外长响应时间或错误的特定部分变得非常容易。Zipkin UI 可以可视化同步和异步工作流。

在下一章中，我们将学习容器编排器，特别是 Kubernetes。我们将学习如何使用 Kubernetes 来部署和管理微服务，同时提高重要的运行时特性，如可伸缩性、高可用性和弹性。

# 问题

1.  `management.tracing.sampling.probability`配置参数的目的是什么？

1.  如何在执行`test-em-all.bash`测试脚本后识别运行时间最长的请求？

1.  我们如何找到被*第十三章*，*使用 Resilience4j 提高弹性*中引入的超时中断的请求？

1.  当*第十三章*，*使用 Resilience4j 提高弹性*中引入的断路器打开时，API 请求的跟踪看起来是什么样子？

1.  我们如何定位那些在调用者未被授权执行请求时失败的 API？

1.  我们如何以编程方式添加跟踪信息？
