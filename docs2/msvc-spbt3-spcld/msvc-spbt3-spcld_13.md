# 13

# 使用 Resilience4j 提高弹性

在本章中，我们将学习如何使用 Resilience4j 使我们的微服务更具弹性，也就是说，如何减轻和从错误中恢复。正如我们在*第一章*，*微服务简介*，*电路断路器*部分，以及*第八章*，*Spring Cloud 简介*，*使用 Resilience4j 提高弹性*部分中已经讨论过的，电路断路器可以用来最小化一个慢速或无响应的下游微服务在一个大规模同步通信微服务系统景观中可能造成的损害。我们将看到 Resilience4j 中的电路断路器如何与时间限制器和重试机制一起使用，以防止两种最常见的错误情况：

+   开始响应缓慢或根本不响应的微服务

+   随时随机失败的请求，例如，由于临时网络问题

本章将涵盖以下主题：

+   介绍三个 Resilience4j 机制：电路断路器、时间限制器和重试

+   将机制添加到源代码中

+   在系统景观中部署时尝试这些机制

# 技术要求

关于如何安装本书中使用的工具以及如何访问本书源代码的说明，请参阅：

+   *第二十一章*，*macOS 安装说明*

+   *第二十二章*，*使用 WSL 2 和 Ubuntu 的 Microsoft Windows 安装说明*

本章中的代码示例均来自`$BOOK_HOME/Chapter13`的源代码。

如果你想查看对本章源代码所做的更改，即查看使用 Resilience4j 添加弹性的过程，你可以将其与*第十二章*，*集中配置*的源代码进行比较。你可以使用你喜欢的`diff`工具比较两个文件夹，`$BOOK_HOME/Chapter12`和`$BOOK_HOME/Chapter13`。

# 介绍 Resilience4j 弹性机制

电路断路器、时间限制器和重试机制在两个软件组件之间的任何同步通信中都可能很有用，例如，微服务。在本章中，我们将将这些机制应用于一个地方，即在`product-composite`服务对`product`服务的调用中。以下图示说明了这一点：

![图描述自动生成](img/B19825_13_01.png)

图 13.1：向系统景观添加弹性功能

注意，其他微服务对发现和配置服务器的同步调用在先前的图中没有显示（为了便于阅读）。

在 2019 年 11 月的 Spring Cloud Hoxton 版本中，添加了**Spring Cloud 断路器**项目。它为断路器提供了一个抽象层。Resilience4j 可以配置为在底层使用。该项目不像 Resilience4j 项目那样以集成的方式提供其他弹性机制，如重试、时间限制器、舱壁或速率限制器。有关该项目的更多信息，请参阅[`spring.io/projects/spring-cloud-circuitbreaker`](https://spring.io/projects/spring-cloud-circuitbreaker)。

许多其他替代方案也存在。例如，Reactor 项目自带对重试和超时的内置支持；请参阅`Mono.retryWhen()`和`Mono.timeout()`。Spring 也有重试机制（请参阅[`github.com/spring-projects/spring-retry`](https://github.com/spring-projects/spring-retry)），但它不支持响应式编程模型。

然而，没有任何替代方案像 Resilience4j 那样提供如此一致和高度集成的弹性机制集合，特别是在 Spring Boot 环境中，其中依赖项、注解和配置以优雅和一致的方式使用。最后，值得注意的是，Resilience4j 注解独立于使用的编程风格工作，无论是响应式还是命令式。

## 介绍断路器

让我们快速回顾一下*第八章*，*Spring Cloud 简介*中*使用 Resilience4j 提高弹性*部分提到的断路器状态图：

![图描述自动生成](img/B19825_13_02.png)

图 13.2：断路器状态图

断路器的关键特性如下：

+   如果断路器检测到过多的故障，它将打开其电路，即不允许新的调用。

+   当电路断开时，断路器将执行快速失败逻辑。这意味着它不会等待后续调用发生新的故障，例如超时。相反，它直接将调用重定向到**回退方法**。回退方法可以应用各种业务逻辑以产生最佳努力响应。例如，回退方法可以从本地缓存中返回数据或简单地返回一个立即的错误消息。这将防止微服务在它依赖的服务停止正常响应时变得无响应。这在高负载下特别有用。

+   经过一段时间后，断路器将处于半开状态，允许新的调用检查导致失败的問題是否已解决。如果断路器检测到新的故障，它将再次打开电路并回到快速失败逻辑。否则，它将关闭电路并恢复正常操作。这使得微服务能够对故障具有弹性，或自我修复，这是在微服务系统中不可或缺的能力，这些微服务以同步方式相互通信。

Resilience4j 以多种方式在运行时公开有关断路器的信息：

+   可以使用微服务的 `actuator` `health` 端点 `/actuator/health` 监控断路器的当前状态。

+   断路器还会在 `actuator` 端点发布事件，例如状态转换和 `/actuator/circuitbreakerevents`。

+   最后，断路器与 Spring Boot 的指标系统集成，并可以使用它将指标发布到监控工具，例如 Prometheus。

我们将在本章尝试 `health` 和 `event` 端点。在 *第二十章*，*监控微服务* 中，我们将看到 Prometheus 的实际应用以及它如何收集由 Spring Boot 公开的指标，例如来自我们的断路器的指标。

为了控制断路器中的逻辑，Resilience4j 可以通过标准的 Spring Boot 配置文件进行配置。我们将使用以下配置参数：

+   `slidingWindowType`: 为了确定是否需要打开断路器，Resilience4j 使用滑动窗口，通过计算最近的事件来做出决策。滑动窗口可以是基于固定数量的调用或固定的时间间隔。此参数用于配置使用哪种类型的滑动窗口。

    我们将使用基于计数的滑动窗口，将此参数设置为 `COUNT_BASED`。

+   `slidingWindowSize`: 在关闭状态下，用于确定电路是否应该打开的调用次数。

    我们将此参数设置为 `5`。

+   `failureRateThreshold`: 导致电路打开的失败调用百分比阈值。

    我们将此参数设置为 `50%`。此设置，连同 `slidingWindowSize` 设置为 `5`，意味着如果最后五个调用中有三个或更多是故障，则电路将打开。

+   `automaticTransitionFromOpenToHalfOpenEnabled`: 确定断路器在等待期结束后是否会自动过渡到半开状态。否则，它将在等待期结束后等待第一个调用，然后过渡到半开状态。

    我们将此参数设置为 `true`。

+   `waitDurationInOpenState`: 指定电路保持打开状态的时间长度，即过渡到半开状态之前。

    我们将此参数设置为 `10000 ms`。此设置，连同启用由前一个参数设置的自动过渡到半开状态，意味着断路器将保持电路打开 10 秒，然后过渡到半开状态。

+   `permittedNumberOfCallsInHalfOpenState`: 在半开状态下，用于确定电路是否会再次打开或返回到正常、关闭状态的调用次数。

    我们将此参数设置为 `3`，这意味着电路断路器将根据电路过渡到半打开状态后的前三次调用来决定是否打开或关闭电路。由于 `failureRateThreshold` 参数设置为 50%，如果两次或三次调用失败，电路将再次打开。否则，电路将关闭。

+   `ignoreExceptions`：此参数可以用来指定不应计为故障的异常。预期的业务异常，如“未找到”或“无效输入”，是电路断路器应该忽略的典型异常；搜索不存在数据或输入无效的用户不应导致电路打开。

    我们将此参数设置为包含异常 `NotFoundException` 和 `InvalidInputException` 的列表。

    最后，为了正确配置 Resilience4j 在 `actuator` `health` 端点中报告电路断路器状态，以下参数被设置：

+   `registerHealthIndicator = true` 启用 Resilience4j 填充 `health` 端点，其中包含其电路断路器状态的信息。

+   `allowHealthIndicatorToFail = false` 告诉 Resilience4j 不要影响 `health` 端点的状态。这意味着即使组件的一个电路断路器处于打开或半打开状态，`health` 端点仍然会报告 `"UP"`。非常重要的一点是，组件的健康状态不应仅因为其一个电路断路器未处于关闭状态而被报告为 `"DOWN"`。这意味着即使它依赖的某个组件不正常，该组件仍然被认为是正常的。

这实际上是电路断路器的核心价值，因此将此值设置为 `true` 大概会削弱引入电路断路器的价值。在 Resilience4j 的早期版本中，这实际上就是行为。在更近期的版本中，这一行为已被纠正，并且 `false` 实际上是此参数的默认值。但鉴于我认为理解组件的健康状态与其电路断路器状态之间的关系非常重要，我已经将其添加到配置中。

+   最后，我们还必须配置 Spring Boot Actuator，以便在对其 `health` 端点的请求响应中添加 Resilience4j 生成的电路断路器健康信息：

    ```java
    management.health.circuitbreakers.enabled: true 
    ```

对于可用的配置参数的完整列表，请参阅 [`resilience4j.readme.io/docs/circuitbreaker#create-and-configure-a-circuitbreaker`](https://resilience4j.readme.io/docs/circuitbreaker#create-and-configure-a-circuitbreaker)。

## 介绍时间限制器

为了帮助电路断路器处理缓慢或无响应的服务，超时机制可能很有帮助。Resilience4j 的超时机制，称为 **TimeLimiter**，可以通过标准的 Spring Boot 配置文件进行配置。我们将使用以下配置参数：

+   `timeoutDuration`：指定 `TimeLimiter` 实例在抛出超时异常之前等待调用完成的时长。我们将它设置为 `2s`。

## 介绍重试机制

**重试**机制对于随机和不频繁的故障非常有用，例如临时的网络故障。重试机制可以在尝试之间配置一个延迟，简单地多次重试失败的请求。对重试机制使用的一个重要限制是，它重试的服务必须是**幂等的**，也就是说，使用相同的请求参数调用服务一次或多次会产生相同的结果。例如，读取信息是幂等的，但创建信息通常不是。你不希望重试机制意外地创建两个订单，仅仅因为第一个订单创建的响应在网络中丢失。

当涉及到事件和指标时，Resilience4j 以与断路器相同的方式公开重试信息，但并不提供任何健康信息。重试事件可以在 `actuator` 端点 `/actuator/retryevents` 上访问。为了控制重试逻辑，可以使用标准的 Spring Boot 配置文件来配置 Resilience4j。我们将使用以下配置参数：

+   `maxAttempts`：在放弃之前尝试的次数，包括第一次调用。我们将此参数设置为 `3`，允许在初始失败调用后进行最多两次的重试尝试。

+   `waitDuration`：下一次重试尝试之前的等待时间。我们将此值设置为 `1000` 毫秒，这意味着我们将等待 1 秒钟进行重试。

+   `retryExceptions`：将触发重试的异常列表。我们只会在 `InternalServerError` 异常上触发重试，即当 HTTP 请求以 `500` 状态码响应时。

在配置重试和断路器设置时要小心，例如，确保断路器在完成预期数量的重试之前不要打开电路！

要查看可用的配置参数的完整列表，请参阅[`resilience4j.readme.io/docs/retry#create-and-configure-retry`](https://resilience4j.readme.io/docs/retry#create-and-configure-retry)。

通过这次介绍，我们准备好查看如何将这些弹性机制添加到 `product-composite` 服务的源代码中。

# 将弹性机制添加到源代码中

在我们将弹性机制添加到源代码之前，我们将添加代码以使其能够强制发生错误，作为一个延迟和/或随机故障。接下来，我们将添加一个断路器以及一个时间限制器来处理缓慢或无响应的 API，以及一个可以处理随机发生的故障的重试机制。从 Resilience4j 添加这些功能遵循 Spring Boot 的方式，这是我们之前章节中使用的方式：

+   在构建文件中添加 Resilience4j 的启动依赖

+   在将应用弹性机制的源代码位置添加注释

+   添加一个控制弹性机制行为的配置

处理弹性挑战是集成层的责任；因此，弹性机制将被放置在`ProductCompositeIntegration`类中。在业务逻辑中实现的源代码，在`ProductCompositeServiceImpl`类中，将不会意识到弹性机制的存在。

一旦我们建立了机制，我们最终将扩展我们的测试脚本`test-em-all.bash`，添加自动验证电路断路器在系统环境中部署时按预期工作的测试。

## 添加可编程延迟和随机错误

为了能够测试我们的弹性机制，我们需要一种控制错误发生时间的方法。实现这一目标的一种简单方法是在用于检索产品和复合产品的 API 中添加可选查询参数。

本节中添加的代码和 API 参数，用于强制延迟和错误发生，应仅在开发测试期间使用，不应在生产环境中使用。当我们学习到第十八章中关于服务网格的概念时，即*使用服务网格提高可观察性和管理*，我们将了解在生产环境中可以使用的更佳方法，以受控的方式引入延迟和错误。使用服务网格，我们可以引入延迟和错误，通常用于验证弹性能力，而不会影响微服务的源代码。

复合产品 API 将直接将参数传递给产品 API。以下查询参数已被添加到这两个 API 中：

+   `delay`: 导致`product`微服务上的`getProduct` API 延迟其响应。该参数以秒为单位指定。例如，如果参数设置为`3`，则会在响应返回之前造成三秒的延迟。

+   `faultPercentage`: 导致`product`微服务上的`getProduct` API 随机抛出异常，其概率由查询参数指定，范围从 0 到 100%。例如，如果参数设置为`25`，则平均每四次调用 API 时，会有一次调用失败并抛出异常。在这些情况下，它将返回`HTTP 错误 500（内部服务器错误）`。

### API 定义的变化

我们上面引入的两个查询参数`delay`和`faultPercentage`已在`api`项目的以下两个 Java 接口中定义：

+   `ProductCompositeService`:

    ```java
    Mono<ProductAggregate> getProduct(
        @PathVariable int productId,
        @RequestParam(value = "delay", required = false, defaultValue =
        "0") int delay,
        @RequestParam(value = "faultPercent", required = false, 
        defaultValue = "0") int faultPercent
    ); 
    ```

+   `ProductService`:

    ```java
    Mono<Product> getProduct(
         @PathVariable int productId,
         @RequestParam(value = "delay", required = false, defaultValue
         = "0") int delay,
         @RequestParam(value = "faultPercent", required = false, 
         defaultValue = "0") int faultPercent
    ); 
    ```

查询参数被声明为可选的，并具有默认值，这些默认值禁用了错误机制的使用。这意味着如果请求中没有使用任何查询参数，则不会应用延迟也不会抛出错误。

### 产品-复合微服务的变化

`product-composite` 微服务只是将参数传递给产品 API。服务实现接收 API 请求并将参数传递给进行产品 API 调用的集成组件：

+   `ProductCompositeServiceImpl` 类对集成组件的调用看起来如下：

    ```java
    public Mono<ProductAggregate> getProduct(int productId,
      int delay, int faultPercent) {
        return Mono.zip(
            ...
            integration.getProduct(productId, delay, faultPercent),
            .... 
    ```

+   `ProductCompositeIntegration` 类对产品 API 的调用如下：

    ```java
    public Mono<Product> getProduct(int productId, int delay, 
      int faultPercent) {

        URI url = UriComponentsBuilder.fromUriString(
          PRODUCT_SERVICE_URL + "/product/{productId}?delay={delay}" 
          + "&faultPercent={faultPercent}")
          .build(productId, delay, faultPercent);
      return webClient.get().uri(url).retrieve()... 
    ```

### 产品微服务的变更

`product` 微服务通过扩展用于从 MongoDB 数据库读取产品信息的现有流，在 `ProductServiceImpl` 类中实现了实际的延迟和随机错误生成器。它看起来是这样的：

```java
public Mono<Product> getProduct(int productId, int delay, 
  int faultPercent) {
  ...
  return repository.findByProductId(productId)
    .map(e -> throwErrorIfBadLuck(e, faultPercent))
    .delayElement(Duration.ofSeconds(delay))
    ...
} 
```

当流返回 Spring Data 存储库的响应时，它首先应用 `throwErrorIfBadLuck` 方法来查看是否需要抛出异常。接下来，它使用 `Mono` 类中的 `delayElement` 函数应用延迟。

随机错误生成器 `throwErrorIfBadLuck()` 生成一个介于 `1` 和 `100` 之间的随机数，如果它高于或等于指定的故障百分比，则抛出异常。如果没有抛出异常，则将产品实体传递到流中。源代码如下：

```java
private ProductEntity throwErrorIfBadLuck(
  ProductEntity entity, int faultPercent) {
  if (faultPercent == 0) {
    return entity;
  }
  int randomThreshold = getRandomNumber(1, 100);
  if (faultPercent < randomThreshold) {
    LOG.debug("We got lucky, no error occurred, {} < {}", 
      faultPercent, randomThreshold);

  } else {
    LOG.info("Bad luck, an error occurred, {} >= {}",
      faultPercent, randomThreshold);

    throw new RuntimeException("Something went wrong...");
  }
  return entity;
}
private final Random randomNumberGenerator = new Random();
private int getRandomNumber(int min, int max) {
  if (max < min) {
    throw new IllegalArgumentException("Max must be greater than min");
  }
  return randomNumberGenerator.nextInt((max - min) + 1) + min;
} 
```

在程序化延迟和随机错误函数就绪后，我们准备开始向代码中添加弹性机制。我们将从断路器和时间限制器开始。

## 添加断路器和时间限制器

如我们之前提到的，我们需要添加依赖项、注解和配置。我们还需要添加一些代码来实现快速失败场景的回退逻辑。我们将在接下来的章节中看到如何做到这一点。

### 在构建文件中添加依赖项

要添加断路器和时间限制器，我们必须在构建文件 `build.gradle` 中添加适当的 Resilience4j 库的依赖项。

从产品文档（[`resilience4j.readme.io/docs/getting-started-3#setup`](https://resilience4j.readme.io/docs/getting-started-3#setup)）中，我们可以了解到需要添加以下三个依赖项。当编写这一章节时，我们将使用最新可用的版本（`v2.0.2`）：

```java
ext {
   resilience4jVersion = "2.0.2"
}
dependencies {
    implementation "io.github.resilience4j:resilience4j-spring-
boot2:${resilience4jVersion}"
    implementation "io.github.resilience4j:resilience4j-reactor:${resilience4jVersion}"
    implementation 'org.springframework.boot:spring-boot-starter-aop'
    ... 
```

为了避免 Spring Cloud 覆盖它捆绑的较旧版本的 Resilience4j 所使用的版本，我们还导入了一个 `resilience4j-bom`（物料清单）文件，如 Spring Boot 3 示例项目 [`github.com/resilience4j/resilience4j-spring-boot3-demo`](https://github.com/resilience4j/resilience4j-spring-boot3-demo) 中所述。我们将此 `bom` 文件添加到现有的 `bom` 文件中，该文件位于 `dependencyManagement` 部分：

```java
dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
        mavenBom "io.github.resilience4j:resilience4j-bom:${resilience4jVersion}"
    }
} 
```

### 在源代码中添加注解

可以通过注解要保护的方法来应用熔断器，在本例中是`ProductCompositeIntegration`类中的`getProduct()`方法。熔断器是由异常触发的，而不是由超时本身触发的。为了能够在超时后触发熔断器，我们将添加一个可以注解为`@TimeLimiter(...)`的时间限制器。源代码如下：

```java
@TimeLimiter(name = "product")
@CircuitBreaker(
     name = "product", fallbackMethod = "getProductFallbackValue")
public Mono<Product> getProduct(
  int productId, int delay, int faultPercent) {
  ...
} 
```

熔断器和时间限制器注解的`name` `"product"`用于标识将要应用配置。熔断器注解中的`fallbackMethod`参数用于指定当熔断器开启时调用哪个回退方法（在本例中为`getProductFallbackValue`）；下面将介绍其用法。

要激活熔断器，必须以 Spring bean 的形式调用注解的方法。在我们的案例中，是 Spring 注入到服务实现类`ProductCompositeServiceImpl`中的集成类，因此用作 Spring bean：

```java
private final ProductCompositeIntegration integration;
@Autowired
public ProductCompositeServiceImpl(... ProductCompositeIntegration integration) {
  this.integration = integration;
}
public Mono<ProductAggregate> getProduct(int productId, int delay, int faultPercent) {
  return Mono.zip(
    ..., 
    integration.getProduct(productId, delay, faultPercent), 
    ... 
```

### 添加快速失败回退逻辑

要能够在熔断器开启时应用回退逻辑，即当请求快速失败时，我们可以在`CircuitBreaker`注解上指定一个回退方法，如前一个源代码所示。该方法必须遵循熔断器应用的方法的签名，并且还有一个额外的最后一个参数，用于传递触发熔断器的异常。在我们的案例中，回退方法的签名如下：

```java
private Mono<Product> getProductFallbackValue(int productId, 
  int delay, int faultPercent, CallNotPermittedException ex) { 
```

最后一个参数指定了我们想要能够处理类型为`CallNotPermittedException`的异常。我们只对当熔断器处于开启状态时抛出的异常感兴趣，以便我们可以应用快速失败逻辑。当熔断器开启时，它将不允许调用底层方法；相反，它将立即抛出一个`CallNotPermittedException`异常。因此，我们只对捕获`CallNotPermittedException`异常感兴趣。

回退逻辑可以根据`productId`从替代源中查找信息，例如，内部缓存。在我们的案例中，我们将根据`productId`返回硬编码的值，以模拟缓存中的命中。为了模拟缓存中的未命中，当`productId`为`13`时，我们将抛出一个`not found`异常。

回退方法的实现看起来是这样的：

```java
private Mono<Product> getProductFallbackValue(int productId, 
  int delay, int faultPercent, CallNotPermittedException ex) {
  if (productId == 13) {
    String errMsg = "Product Id: " + productId 
      + " not found in fallback cache!";
    throw new NotFoundException(errMsg);
  }
  return Mono.just(new Product(productId, "Fallback product" 
    + productId, productId, serviceUtil.getServiceAddress()));
} 
```

### 添加配置

最后，将熔断器和时间限制器的配置添加到配置存储库中的`product-composite.yml`文件中，如下所示：

```java
resilience4j.timelimiter:
  instances:
    product:
      timeoutDuration: 2s
management.health.circuitbreakers.enabled: true
resilience4j.circuitbreaker:
  instances:
    product:
      allowHealthIndicatorToFail: false
      registerHealthIndicator: true
      slidingWindowType: COUNT_BASED
      slidingWindowSize: 5
      failureRateThreshold: 50
      waitDurationInOpenState: 10000
      permittedNumberOfCallsInHalfOpenState: 3
      automaticTransitionFromOpenToHalfOpenEnabled: true
      ignoreExceptions:
        - se.magnus.api.exceptions.InvalidInputException
        - se.magnus.api.exceptions.NotFoundException 
```

配置中的值已在之前的章节中描述，*介绍熔断器*和*介绍时间限制器*。

## 添加重试机制

与断路器一样，通过添加依赖项、注解和配置来设置重试机制。依赖项已在*向构建文件添加依赖项*部分中添加，因此我们只需添加注解并设置配置。

### 添加重试注解

可以通过使用`@Retry(name="nnn")`注解将重试机制应用于一个方法，其中`nnn`是要用于此方法的配置条目名称。有关配置的详细信息，请参阅以下*添加配置*部分。在我们的案例中，方法与断路器和时间限制器相同，即`ProductCompositeIntegration`类中的`getProduct()`方法：

```java
 @Retry(name = "product")
  @TimeLimiter(name = "product")
  @CircuitBreaker(name = "product", fallbackMethod =
    "getProductFallbackValue")
  public Mono<Product> getProduct(int productId, int delay, 
    int faultPercent) { 
```

### 添加配置

在配置仓库中的`product-composite.yml`文件中，与断路器和时间限制器一样，以相同的方式添加了重试机制的配置，如下所示：

```java
resilience4j.retry:
  instances:
    product:
      maxAttempts: 3
      waitDuration: 1000
      retryExceptions:
      - org.springframework.web.reactive.function.client.WebClientResponseException$InternalServerError 
```

实际值在上面的*介绍重试机制*部分中已讨论。

当使用多个 Resilience4j 机制（在我们的案例中，是断路器、时间限制器和重试机制）时，了解这些方面应用的顺序是很重要的。有关信息，请参阅[`resilience4j.readme.io/docs/getting-started-3#aspect-order`](https://resilience4j.readme.io/docs/getting-started-3#aspect-order)。

这就是所有所需的依赖项、注解、源代码和配置。让我们通过扩展测试脚本并添加验证已部署系统景观中断路器按预期工作的测试来结束。

## 添加自动化测试

已将断路器的自动化测试添加到`test-em-all.bash`测试脚本中的单独函数`testCircuitBreaker()`：

```java
...
function testCircuitBreaker() {
    echo "Start Circuit Breaker tests!"
    ...
}
...
testCircuitBreaker
...
echo "End, all tests OK:" `date` 
```

为了能够执行一些必需的验证，我们需要访问`product-composite`微服务的`actuator`端点，这些端点不是通过边缘服务器公开的。因此，我们将通过在`product-composite`微服务中运行命令并使用 Docker Compose 的`exec`命令来访问`actuator`端点。微服务使用的基镜像`Eclipse Temurin`捆绑了`curl`，因此我们可以在`product-composite`容器中简单地运行一个`curl`命令来获取所需的信息。命令看起来是这样的：

```java
docker-compose exec -T product-composite curl -s http://product-composite:8080/actuator/health 
```

`-T`参数用于禁用`exec`命令的终端使用。这对于在不存在终端的环境中运行`test-em-all.bash`测试脚本很重要，例如，在用于 CI/CD 的自动化构建管道中。

为了能够提取我们测试所需的信息，我们可以将输出管道到`jq`工具。例如，要提取断路器的实际状态，我们可以运行以下命令：

```java
docker-compose exec -T product-composite curl -s http://product-composite:8080/actuator/health | jq -r .components.circuitBreakers.details.product.details.state 
```

根据实际状态，它将返回`CLOSED`、`OPEN`或`HALF_OPEN`。

测试开始时，正是这样做，即在执行测试之前验证断路器是否关闭：

```java
assertEqual "CLOSED" "$(docker-compose exec -T product-composite curl -s http://product-composite:8080/actuator/health | jq -r .components.circuitBreakers.details.product.details.state)" 
```

接下来，测试将通过连续运行三个命令来强制断路器开启，所有这些命令都会因为`product`服务（`delay`参数设置为`3`秒）的缓慢响应而超时：

```java
for ((n=0; n<3; n++))
do
    assertCurl 500 "curl -k https://$HOST:$PORT/product-composite/$PROD_ID_REVS_RECS?delay=3 $AUTH -s"
    message=$(echo $RESPONSE | jq -r .message)
    assertEqual "Did not observe any item or terminal signal within 2000ms" "${message:0:57}"
done 
```

**快速提醒配置**：`product`服务的超时设置为两秒，以便三秒的延迟会导致超时。断路器在关闭时配置为评估最后五个调用。脚本中先于断路器特定测试的测试已经执行了一些成功的调用。失败阈值设置为 50%；三个带有三秒延迟的调用足以打开电路。

在电路开启的情况下，我们期望出现快速失败的行为，即我们不需要等待超时就能得到响应。我们还期望调用`fallback`方法以返回最佳尝试响应。这也应该适用于正常调用，即不请求延迟。这通过以下代码得到验证：

```java
assertEqual "OPEN" "$(docker-compose exec -T product-composite curl -s http://product-composite:8080/actuator/health | jq -r .components.circuitBreakers.details.product.details.state)"
assertCurl 200 "curl -k https://$HOST:$PORT/product-composite/$PROD_ID_REVS_RECS?delay=3 $AUTH -s"
assertEqual "Fallback product$PROD_ID_REVS_RECS" "$(echo "$RESPONSE" | jq -r .name)"
assertCurl 200 "curl -k https://$HOST:$PORT/product-composite/$PROD_ID_REVS_RECS $AUTH -s"
assertEqual "Fallback product$PROD_ID_REVS_RECS" "$(echo "$RESPONSE" | jq -r .name)" 
```

产品 ID `1`存储在一个变量中，`$PROD_ID_REVS_RECS`，以便在需要时更容易修改脚本。

我们还可以验证模拟的`未找到`错误逻辑在回退方法中按预期工作，即回退方法对于产品 ID `13`返回`404, NOT_FOUND`：

```java
assertCurl 404 "curl -k https://$HOST:$PORT/product-composite/$PROD_ID_NOT_FOUND $AUTH -s"
assertEqual "Product Id: $PROD_ID_NOT_FOUND not found in fallback cache!" "$(echo $RESPONSE | jq -r .message)" 
```

产品 ID `13`存储在一个变量中，`$PROD_ID_NOT_FOUND`。

如配置所示，断路器将在`10`秒后将状态更改为半开。为了验证这一点，测试等待`10`秒：

```java
echo "Will sleep for 10 sec waiting for the CB to go Half Open..."
sleep 10 
```

在验证了预期的状态（半开）后，测试运行三个正常请求，使断路器回到其正常状态，这也得到了验证：

```java
assertEqual "HALF_OPEN" "$(docker-compose exec -T product-composite curl -s http://product-composite:8080/actuator/health | jq -r .components.circuitBreakers.details.product.details.state)"
for ((n=0; n<3; n++))
do
    assertCurl 200 "curl -k https://$HOST:$PORT/product-composite/$PROD_ID_REVS_RECS $AUTH -s"
    assertEqual "product name C" "$(echo "$RESPONSE" | jq -r .name)"
done
assertEqual "CLOSED" "$(docker-compose exec -T product-composite curl -s http://product-composite:8080/actuator/health | jq -r .components.circuitBreakers.details.product.details.state)" 
```

测试代码还验证了它收到了来自底层数据库的数据响应。它是通过比较返回的产品名称与数据库中存储的值来做到这一点的。对于产品 ID 为`1`的产品，名称是`"product name C"`。

**快速提醒配置**：断路器在半开状态下会评估前三个调用。因此，我们需要运行三个请求，其中超过 50%的请求成功，才能关闭断路器。

测试通过使用由断路器公开的`/actuator/circuitbreakerevents`操作器 API 来完成，以揭示内部事件。它用于找出断路器执行了哪些状态转换。我们预计最后三个状态转换如下：

+   第一个状态转换：关闭到开启

+   下一个状态转换：开启到半开

+   最后的状态转换：半开到关闭

这可以通过以下代码进行验证：

```java
assertEqual "CLOSED_TO_OPEN"      "$(docker-compose exec -T product-composite curl -s http://product-composite:8080/actuator/circuitbreakerevents/product/STATE_TRANSITION | jq -r
.circuitBreakerEvents[-3].stateTransition)"
assertEqual "OPEN_TO_HALF_OPEN"   "$(docker-compose exec -T product-composite curl -s http://product-composite:8080/actuator/circuitbreakerevents/product/STATE_TRANSITION | jq -r .circuitBreakerEvents[-2].stateTransition)"
assertEqual "HALF_OPEN_TO_CLOSED" "$(docker-compose exec -T product-composite curl -s http://product-composite:8080/actuator/circuitbreakerevents/product/STATE_TRANSITION | jq -r .circuitBreakerEvents[-1].stateTransition)" 
```

`jq`表达式`circuitBreakerEvents[-1]`表示断路器事件数组中的最后一个条目，`[-2]`是倒数第二个事件，而`[-3]`是倒数第三个事件。它们一起构成了最新的三个事件，即我们感兴趣的事件。

我们在测试脚本中添加了很多步骤，但有了这个，我们可以自动验证我们的断路器预期的基本行为已经到位。在下文中，我们将尝试它。我们将通过运行测试脚本和手动运行测试脚本中的命令来运行测试。

# 尝试断路器和重试机制

现在，是时候尝试断路器和重试机制了。我们将像往常一样，首先构建 Docker 镜像并运行测试脚本`test-em-all.bash`。之后，我们将手动运行我们之前描述的测试，以确保我们理解正在发生的事情！我们将执行以下手动测试：

+   快乐的日子测试断路器，以验证在正常操作下断路器是关闭的

+   断路器的负面测试，以验证当事情开始出错时，断路器是否会打开

+   回到正常操作，以验证一旦问题解决，断路器会回到关闭状态

+   尝试使用随机错误的重试机制

## 构建和运行自动化测试

要构建和运行自动化测试，我们需要做以下事情：

1.  首先，使用以下命令构建 Docker 镜像：

    ```java
    cd $BOOK_HOME/Chapter13
    ./gradlew build && docker-compose build 
    ```

1.  接下来，在 Docker 中启动系统景观并使用以下命令运行常规测试：

    ```java
    ./test-em-all.bash start 
    ```

当测试脚本打印出**开始断路器测试**时，我们之前描述的测试已经执行完毕！

## 验证在正常操作下断路器是否关闭

在我们能够调用 API 之前，我们需要一个访问令牌。运行以下命令以获取访问令牌：

```java
unset ACCESS_TOKEN
ACCESS_TOKEN=$(curl -k https://writer:secret-writer@localhost:8443/oauth2/token -d grant_type=client_credentials -d scope="product:read product:write" -s | jq -r .access_token)
echo $ACCESS_TOKEN 
```

由授权服务器签发的访问令牌有效期为 1 小时。因此，如果您一段时间后开始收到`401 – 未授权`错误，可能需要获取新的访问令牌。

尝试一个正常请求并验证它返回 HTTP 响应代码`200`：

```java
curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/1 -w "%{http_code}\n" -o /dev/null -s 
```

`-w "%{http_code}\n"`开关用于打印 HTTP 返回状态。只要命令返回`200`，我们就对响应体不感兴趣，因此我们使用开关`-o /dev/null`来抑制它。

使用`health` API 验证断路器是否关闭：

```java
docker-compose exec product-composite curl -s http://product-composite:8080/actuator/health | jq -r .components.circuitBreakers.details.product.details.state 
```

我们期望它响应为`CLOSED`。

## 在事情出错时强制断路器打开

现在，是时候让事情出错！我的意思是，现在是时候尝试一些负面测试，以验证当事情开始出错时，断路器是否会打开。调用 API 三次，并将`product`服务指向每次调用时超时，即延迟响应`3`秒。这应该足以触发断路器：

```java
curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/1?delay=3 -s | jq . 
```

我们期望每次都得到以下响应：

![文本描述自动生成](img/B19825_13_03.png)

图 13.3：超时后的响应

电路断路器现在是开启的，所以如果你在`waitInterval`（即 10 秒）内进行第四次尝试，你会看到快速失败行为和`fallback`方法在行动。一旦在 2 秒后时间限制器启动，你将立即得到响应而不是错误消息：

![图形用户界面，文本描述自动生成](img/B19825_13_04.png)

图 13.4：电路断路器开启时的响应

响应将来自回退方法。这可以通过查看名称字段中的值来识别，`Fallback product1`。

快速失败和回退方法是电路断路器的关键功能。一个在开启状态下设置等待时间仅为 10 秒的配置要求你相当快才能看到快速失败逻辑和回退方法在行动！一旦处于半开启状态，你总是可以提交三个新的请求，这些请求会导致超时，迫使电路断路器回到开启状态，然后快速尝试第四个请求。然后，你应该从回退方法得到快速失败的响应。你还可以将等待时间增加到一分钟或两分钟，但等待那么长时间直到电路切换到半开启状态可能会相当无聊。

等待 10 秒，让电路断路器过渡到半开启状态，然后运行以下命令以验证电路现在处于半开启状态：

```java
docker-compose exec product-composite curl -s http://product-composite:8080/actuator/health | jq -r .components.circuitBreakers.details.product.details.state 
```

期望它响应`HALF_OPEN`。

## 再次关闭电路断路器

一旦电路断路器处于半开启状态，它将等待三个调用以确定是否应该再次开启电路，或者通过关闭它回到正常状态。

让我们提交三个正常请求来关闭电路断路器：

```java
curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/1 -w "%{http_code}\n" -o /dev/null -s 
```

它们都应该响应`200`。通过使用`health` API 验证电路是否再次关闭：

```java
docker-compose exec product-composite curl -s http://product-composite:8080/actuator/health | jq -r .components.circuitBreakers.details.product.details.state 
```

我们期望它响应为`CLOSED`。

使用以下命令列出最后三个状态转换来完成这个总结：

```java
docker-compose exec product-composite curl -s http://product-composite:8080/actuator/circuitbreakerevents/product/STATE_TRANSITION | jq -r '.circuitBreakerEvents[-3].stateTransition, .circuitBreakerEvents[-2].stateTransition, .circuitBreakerEvents[-1].stateTransition' 
```

期望它响应以下内容：

![文本描述自动生成](img/B19825_13_05.png)

图 13.5：电路断路器状态变化

这个响应告诉我们，我们已经将电路断路器在其状态图上完成了一个完整的循环：

+   当超时错误开始阻止请求成功时，从关闭到开启

+   从开启到半开启，查看错误是否已消失

+   当错误消失时，即我们回到正常操作时，从半开启到关闭

到此为止，我们已经完成了电路断路器的测试；让我们继续并看看重试机制在起作用。

## 尝试由随机错误引起的重试

让我们模拟我们的`product`服务或与其通信存在一个——希望是暂时性的——随机问题。

我们可以通过使用`faultPercent`参数来做这件事。如果我们将其设置为`25`，我们期望平均每四次请求中有一个会失败。我们希望重试机制会启动，通过自动重试失败的请求来帮助我们。注意到重试机制已启动的一种方法是通过测量`curl`命令的响应时间。正常的响应应该大约需要 100 毫秒。由于我们已经配置了重试机制等待 1 秒（参见重试机制配置部分的`waitDuration`参数），我们期望每次重试尝试的响应时间增加 1 秒。为了强制发生随机错误，运行以下命令几次：

```java
time curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/1?faultPercent=25 -w "%{http_code}\n" -o /dev/null -s 
```

命令应该以`200`响应，表示请求成功。以`real`开头的前缀响应时间，例如，`real 0m0.078s`，表示响应时间为 0.078 秒，或 78 毫秒。正常响应，即没有任何重试，应该报告大约 100 毫秒的响应时间，如下所示：

![图形用户界面、文本、应用程序、聊天或文本消息  自动生成的描述](img/B19825_13_06.png)

图 13.6：无重试请求的耗时

一次重试后的响应应该略超过 1 秒，如下所示：

![图形用户界面、应用程序  自动生成的描述](img/B19825_13_07.png)

图 13.7：有重试请求的耗时

HTTP 状态码`200`表示请求已成功，即使它需要重试一次才能成功！

在你注意到 1 秒的响应时间后，表示请求需要重试一次才能成功，运行以下命令以查看最后两个重试事件：

```java
docker-compose exec product-composite curl -s http://product-composite:8080/actuator/retryevents | jq '.retryEvents[-2], .retryEvents[-1]' 
```

你应该能够看到失败的请求和下一次成功的尝试。`creationTime`时间戳预计会相差 1 秒。期望得到如下响应：

![文本、聊天或文本消息  自动生成的描述](img/B19825_13_08.png)

图 13.8：请求一次重试后捕获的重试事件

如果你真的很不幸，你可能会连续两次出现故障，然后你会得到 2 秒的响应时间而不是 1 秒。如果你重复前面的命令，你将能够看到`numberOfAttempts`字段为每次重试尝试计数，在这种情况下设置为`1`："numberOfAttempts": 1。如果调用继续失败，断路器将启动并打开其电路，即后续调用将应用快速失败逻辑，并应用回退方法！

这就结束了本章。请随意在配置参数中实验，以了解更多关于弹性机制的信息。

不要忘记关闭系统景观：

```java
docker-compose down 
```

# 摘要

在本章中，我们看到了 Resilience4j 及其断路器、时间限制器和重试机制的实际应用。

一个对其他服务有同步依赖的微服务，如果这些服务停止按预期响应，尤其是在高负载下，可能会变得无响应或甚至崩溃。通过使用断路器，可以避免这些类型的错误场景，断路器应用快速失败逻辑，并在打开时调用回退方法。断路器还可以通过允许在半开状态下发出请求来检查失败的服务是否再次正常运行，如果是，则关闭电路。为了支持断路器处理无响应的服务，可以使用时间限制器来最大化断路器在启动之前等待的时间。

重试机制可以重试偶尔随机失败请求，例如，由于临时网络问题。仅对幂等服务应用重试请求非常重要，即可以处理同一请求被发送两次或更多次的那些服务。

断路器和重试机制是通过遵循 Spring Boot 习惯来实现的：声明依赖项、添加注解和配置。Resilience4j 在运行时通过 `actuator` 端点公开其断路器和重试机制的信息。对于断路器，有关健康、事件和指标的信息可用。对于重试，有关事件和指标的信息可用。

我们在本章中已经看到了端点在健康和事件方面的使用，但我们必须等到 *第二十章*，*监控微服务*，我们才能使用任何指标。

在下一章中，我们将介绍 Spring Cloud 的最后一部分使用，我们将学习如何使用 Spring Cloud Sleuth 和 Zipkin 通过一组协作的微服务来跟踪调用链。前往 *第十四章*，*理解分布式跟踪*，开始学习！

# 问题

1.  断路器的状态有哪些以及它们是如何被使用的？

1.  我们如何处理断路器中的超时错误？

1.  当断路器快速失败时，我们如何应用回退逻辑？

1.  重试机制和断路器如何相互干扰？

1.  提供一个无法应用重试机制的服务的示例。
