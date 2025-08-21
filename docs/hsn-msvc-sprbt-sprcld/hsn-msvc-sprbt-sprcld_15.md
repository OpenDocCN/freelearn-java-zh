# 第十三章：使用 Resilience4j 提高弹性

在本章中，我们将学习如何使用 Resilience4j 使我们的微服务更具弹性，也就是说，如何减轻和恢复错误。正如我们在第一章*微服务介绍*，"断路器"部分，和第八章*Spring Cloud 介绍*，"Resilience4j 以提高弹性"部分，所讨论的，断路器可以用来自动减少一个慢速或无响应的后端微服务在一个大规模的同步微服务景观中所造成的损害。我们将看到 Resilience4j 中的断路器如何与超时和重试机制一起使用，以防止我经验中最为常见的两个错误情况：

+   响应缓慢或根本不响应的微服务

+   请求偶尔会因临时网络问题而失败

本章将涵盖以下主题：

+   介绍 Resilience4j 断路器和重试机制

+   向源代码添加**断路器**和**重试机制**

+   尝试使用**断路器**和**重试机制**

# 技术要求

本书中描述的所有命令都是在 MacBook Pro 上使用 macOS Mojave 运行的，但如果你想在其他平台（如 Linux 或 Windows）上运行它们，应该是非常直接的。

在本章中不需要安装任何新工具。

本章的源代码可以在本书的 GitHub 仓库中找到：[`github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud/tree/master/Chapter13`](https://github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud/tree/master/Chapter13)。

为了能够运行本书中描述的命令，将源代码下载到一个文件夹中，并设置一个环境变量`$BOOK_HOME`，该变量指向该文件夹。一些示例命令如下：

```java
export BOOK_HOME=~/Documents/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud
git clone https://github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud $BOOK_HOME
cd $BOOK_HOME/Chapter13
```

Java 源代码是为 Java 8 编写的，并在 Java 12 上进行了测试。本章使用 Spring Cloud 2.1.0, SR1（也被称为**Greenwich**版本），Spring Boot 2.1.4 和 Spring 5.1.6，即在撰写本章时可用的 Spring 组件的最新版本。

所有 Dockerfile 中都使用了`openjdk:12.0.2`基础 Docker 镜像。

源代码包含以下 Gradle 项目：

+   `api`

+   `util`

+   `microservices/product-service`

+   `microservices/review-service`

+   `microservices/recommendation-service`

+   `microservices/product-composite-service`

+   `spring-cloud/eureka-server`

+   `spring-cloud/gateway`

+   `spring-cloud/authorization-server`

+   `spring-cloud/config-server`

配置文件可以在 config 仓库中找到，`config-repo`。

本章中的所有源代码示例均来自`$BOOK_HOME/Chapter13`中的源代码，但在某些情况下，去除了源代码中不相关部分，例如注释、导入和日志语句。

如果你想查看本章中应用于源代码的变化，即了解使用 Resilience4j 添加弹性所需的内容，你可以与第十二章的*集中配置*源代码进行比较。你可以使用你喜欢的`diff`工具，比较两个文件夹，`$BOOK_HOME/Chapter12`和`$BOOK_HOME/Chapter13`。

# 介绍 Resilience4j 电路断路器和重试机制

重试和电路断路器在两个软件组件之间的任何同步通信中都有潜在的用处，例如微服务。由于 Spring Cloud Gateway 目前只支持较旧的断路器 Netflix Hystrix，我们的所有微服务都可以使用 Resilience4j，除了边缘服务器。在本章中，我们将在一个地方应用电路断路器和重试机制，即从`product-composite`服务调用`product`服务。以下图表说明了这一点：

![](img/1242dd3a-010d-4a6d-8ce2-edc8fd88c290.png)

请注意，在前面的图表中没有显示其他微服务对发现和配置服务器的同步调用（为了更容易阅读）。

随着本章的写作，一直在进行的工作是为 Spring Cloud 添加一个电路断路器的抽象层，这对 Spring Cloud Gateway 可能是有益的。详情请参阅[`spring.io/blog/2019/04/16/introducing-spring-cloud-circuit-breaker`](https://spring.io/blog/2019/04/16/introducing-spring-cloud-circuit-breaker)。

# 介绍电路断路器

让我们快速回顾一下来自第八章的*Spring Cloud 简介*中的*Resilience4j 改进弹性*部分的电路断路器状态图：

![](img/87b2c812-ab46-45fe-a6d7-0f06a030d556.png)

电路断路器的键特性如下：

+   如果电路断路器检测到太多故障，它将打开其电路，即不允许新的调用。

+   当电路处于断开状态时，电路断路器将执行快速失败逻辑。这意味着它不是等待新的故障发生，例如超时，在后续调用中发生。相反，它直接将调用重定向到一个**回退** **方法**。回退方法可以应用各种业务逻辑以产生最佳努力响应。例如，回退方法可以从本地缓存返回数据，或者简单地返回一个立即的错误消息。这可以防止微服务在它依赖的服务停止正常响应时变得无响应。在高负载下，这特别有用。

+   过了一段时间后，断路器将变为半开放*状态*，允许新的调用查看导致失败的问题是否已解决。如果断路器检测到新的失败，它将再次打开电路并回到快速失败逻辑。否则，它将关闭电路并恢复正常操作。这使得微服务能够抵抗故障，而在与其他微服务同步通信的系统架构中，这种能力是不可或缺的！

Resilience4j 以多种方式在运行时暴露有关断路器的信息：

+   可以通过微服务的 actuator`health`端点监控断路器的当前状态，即`/actuator/health`。

+   断路器还会在`actuator`端点上发布事件，例如，状态转换、`/actuator/circuitbreakerevents`。

+   最后，断路器与 Spring Boot 的度量系统集成，并可以使用它将指标发布到监控工具，例如 Prometheus。

在本章中，我们将尝试使用`health`和`event`端点。在第二十章“微服务监控”中，我们将看到 Prometheus 的实际应用情况，以及它如何收集由 Spring Boot 暴露出来的指标，例如，我们的断路器中的指标。

为了控制断路器中的逻辑，Resilience4J 可以使用标准 Spring Boot 配置文件进行配置。我们将使用以下配置参数：

+   `ringBufferSizeInClosedState`：在关闭状态中的调用次数，用于确定电路是否应打开。

+   `failureRateThreshold`：导致电路打开的失败调用百分比阈值。

+   `waitInterval`：指定电路保持开放状态的时间长度，即，在过渡到半开放状态之前。

+   `ringBufferSizeInHalfOpenState`：在半开放状态下用于确定电路是否应再次打开或回到正常、关闭状态的调用次数。

+   `automaticTransitionFromOpenToHalfOpenEnabled`：确定电路在等待期结束后是否自动变为半开放状态，或者在等待期间等待第一个调用直到变为半开放状态。

+   `ignoreExceptions`：可以用来指定不应被计为错误的异常。例如，找不到或输入无效的业务异常通常是断路器应该忽略的异常，即，搜索不存在的数据或输入无效输入的用户不应该导致电路打开。

Resilience4j 在关闭状态和半开放状态下使用环形缓冲区跟踪成功和失败的调用，因此有了参数名`ringBufferSizeInClosedState`和`ringBufferSizeInHalfOpenState`。

本章将使用以下设置：

+   `ringBufferSizeInClosedState = 5`和`failureRateThreshold = 50%`，意味着如果最后五个调用中有三个或更多是故障，那么电路将打开。

+   `waitInterval = 10000`和`automaticTransitionFromOpenToHalfOpenEnabled = true`，意味着断路器将保持电路开启 10 秒，然后过渡到半开状态。

+   `ringBufferSizeInHalfOpenState = 3`，意味着断路器将基于断路器过渡到半开状态后的三个首次调用来决定是否打开或关闭电路。由于`failureRateThreshold`参数设置为 50%，如果两个或所有三个调用失败，电路将再次打开。否则，电路将关闭。

+   `ignoreExceptions = InvalidInputException`和`NotFoundException`，意味着我们的两个业务异常在断路器中不会被视为故障。

# 引入重试机制

重试机制对于随机和偶尔出现的故障非常有用，例如暂时的网络问题。重试机制可以简单地尝试失败请求多次，每次尝试之间有可配置的延迟。使用重试机制的一个非常重要的限制是，它重试的服务必须是**幂等的**，也就是说，用相同的请求参数调用服务一次或多次会得到相同的结果。例如，读取信息是幂等的，但创建信息通常不是。你不希望重试机制因为第一次创建订单的响应在网络中丢失而意外地创建两个订单。

当涉及到事件和指标时，Resilience4j 以与断路器相同的方式暴露重试信息，但不提供任何健康信息。重试事件可以在`actuator`端点，`/actuator/retryevents`上访问。为了控制重试逻辑，可以使用标准的 Spring Boot 配置文件配置 Resilience4J。我们将使用以下配置参数：

+   `maxRetryAttempts`: 包括第一次调用在内的重试次数上限

+   `waitDuration`: 下次重试尝试之前的等待时间

+   `retryExceptions`: 需要触发重试的异常列表

在本章中，我们将使用以下值：

+   `maxRetryAttempts = 3`: 我们将最多尝试两次重试。

+   `waitDuration= 1000`: 我们将在重试之间等待一秒钟。

+   `retryExceptions = InternalServerError`: 我们只会在遇到`InternalServerError`异常时触发重试，也就是说，当 HTTP 请求响应码为 500 时。

配置重试和断路器设置时要小心，例如，确保断路器在预期的重试次数完成之前不要打开电路！

# 在源代码中添加断路器和重试机制

在向源代码中添加断路器和重试机制之前，我们将添加代码，使其能够强制发生错误——要么是延迟，要么是随机故障。然后，我们将添加一个断路器来处理慢速或无响应的 API，以及一个可以处理随机发生故障的重试机制。从 Resilience4j 添加这些功能遵循传统的 Spring Boot 方式：

+   在构建文件中添加一个针对 Resilience4j 的启动依赖。

+   在源代码中添加注解，以在断路器和重试机制应适用的位置使用。

+   添加控制断路器和重试机制行为的配置。

一旦我们实施了断路器和重试机制，我们将扩展我们的测试脚本`test-em-all.bash`，以包含断路器的测试。

# 添加可编程延迟和随机错误

为了能够测试我们的断路器和重试机制，我们需要一种控制错误发生时间的方法。实现这一目标的一种简单方法是在 API 中添加可选的查询参数，以检索产品和组合产品。组合产品 API 将参数传递给产品 API。以下查询参数已添加到两个 API 中：

+   `delay`：导致`product` 微服务的`getProduct` API 延迟其响应。参数以秒为单位指定。例如，如果参数设置为`3`，它将在返回响应之前造成三秒的延迟。

+   `faultPercentage`：导致`product` 微服务的`getProduct` API 以查询参数指定的概率随机抛出异常，从 0 到 100%。例如，如果参数设置为`25`，它将使平均每四次 API 调用中的第四次失败并抛出异常。在这些情况下，它将返回 HTTP 错误 500 内部服务器错误。

# API 定义的更改

我们之前引入的两个查询参数`delay`和`faultPercentage`，已在`api`项目中的以下两个 Java 接口中定义：

+   `se.magnus.api.composite.product.ProductCompositeService`：

```java
Mono<ProductAggregate> getCompositeProduct(
    @PathVariable int productId,
    @RequestParam(value = "delay", required = false, defaultValue = 
    "0") int delay,
    @RequestParam(value = "faultPercent", required = false, 
    defaultValue = "0") int faultPercent
);
```

+   `se.magnus.api.core.product.ProductService`：

```java
Mono<Product> getProduct(
     @PathVariable int productId,
     @RequestParam(value = "delay", required = false, defaultValue
     = "0") int delay,
     @RequestParam(value = "faultPercent", required = false, 
     defaultValue = "0") int faultPercent
);
```

# 产品组合微服务的更改

`product-composite` 微服务只是将参数传递给产品 API。服务实现接收到 API 请求，并将参数传递给调用产品 API 的集成组件：

+   对`se.magnus.microservices.composite.product.services.ProductCompositeServiceImpl` 类的调用：

```java
public Mono<ProductAggregate> getCompositeProduct(int productId, int delay, int faultPercent) {
    return Mono.zip(
        ...
        integration.getProduct(productId, delay, faultPercent),
        ....
```

+   对`se.magnus.microservices.composite.product.services.ProductCompositeIntegration` 类的调用：

```java
public Mono<Product> getProduct(int productId, int delay, int faultPercent) {
    URI url = UriComponentsBuilder
        .fromUriString(productServiceUrl + "/product/{pid}?delay=
         {delay}&faultPercent={fp}")
        .build(productId, delay, faultPercent);
    return getWebClient().get().uri(url)...
```

# 产品微服务的更改

`product` 微服务在`se.magnus.microservices.core.product.services.ProductServiceImpl`中实现实际延迟和随机错误生成器，如下所示：

```java
public Mono<Product> getProduct(int productId, int delay, int faultPercent) {
    if (delay > 0) simulateDelay(delay);
    if (faultPercent > 0) throwErrorIfBadLuck(faultPercent);
    ...
}
```

延迟函数`simulateDelay()`使用`Thread.sleep()`函数来模拟延迟：

```java
private void simulateDelay(int delay) {
    LOG.debug("Sleeping for {} seconds...", delay);
    try {Thread.sleep(delay * 1000);} catch (InterruptedException e) {}
    LOG.debug("Moving on...");
}
```

随机错误生成器`throwErrorIfBadLuck()`创建一个在`1`和`100`之间的随机数，如果它等于或大于指定的故障百分比，则抛出异常：

```java
private void throwErrorIfBadLuck(int faultPercent) {
    int randomThreshold = getRandomNumber(1, 100);
    if (faultPercent < randomThreshold) {
        LOG.debug("We got lucky, no error occurred, {} < {}", 
        faultPercent, randomThreshold);
    } else {
        LOG.debug("Bad luck, an error occurred, {} >= {}", 
        faultPercent, randomThreshold);
        throw new RuntimeException("Something went wrong...");
    }
}

private final Random randomNumberGenerator = new Random();
private int getRandomNumber(int min, int max) {
    if (max < min) {
        throw new RuntimeException("Max must be greater than min");
    }
    return randomNumberGenerator.nextInt((max - min) + 1) + min;
}
```

# 添加断路器

正如我们之前提到的，我们需要添加依赖项、注解和配置。我们还需要添加一些处理超时和回退逻辑的代码。我们将在接下来的章节中看到如何进行操作。

# 向构建文件添加依赖项

要在电路中添加断路器，我们必须在构建文件`build.gradle`中添加对适当 Resilience4j 库的依赖：

```java
ext {
   resilience4jVersion = "0.14.1"
}
dependencies {
   implementation("io.github.resilience4j:resilience4j-spring-
    boot2:${resilience4jVersion}")
   implementation("io.github.resilience4j:resilience4j-
    reactor:${resilience4jVersion}")
   ...
```

# 添加断路器和超时逻辑

断路器可以通过在期望其保护的方法上使用`@CircuitBreaker(name="nnn")`注解来应用，这里是指`se.magnus.microservices.composite.product.services.ProductCompositeIntegration`类中的`getProduct()`方法。断路器是由异常触发的，而不是由超时本身触发的。为了能够在超时后触发断路器，我们必须添加在超时后生成异常的代码。使用基于 Project Reactor 的`WebClient`，我们可以通过使用其`timeout(Duration)`方法方便地做到这一点。源代码如下所示：

```java
@CircuitBreaker(name = "product")
public Mono<Product> getProduct(int productId, int delay, int faultPercent) {
    ...
    return getWebClient().get().uri(url)
        .retrieve().bodyToMono(Product.class).log()
        .onErrorMap(WebClientResponseException.class, ex -> 
         handleException(ex))
        .timeout(Duration.ofSeconds(productServiceTimeoutSec));
}
```

断路器的名称`"product"`用于标识我们将要通过的配置。超时参数`productServiceTimeoutSec`作为可配置参数值注入到构造函数中：

```java
private final int productServiceTimeoutSec;

@Autowired
public ProductCompositeIntegration(
    ...
    @Value("${app.product-service.timeoutSec}") int productServiceTimeoutSec
) {
    ...
    this.productServiceTimeoutSec = productServiceTimeoutSec;
}
```

要激活断路器，必须作为 Spring Bean 调用注解方法。在我们的情况下，是 Spring 将集成类注入到服务实现类中，因此作为 Spring Bean 使用：

```java
private final ProductCompositeIntegration integration;

@Autowired
public ProductCompositeServiceImpl(... ProductCompositeIntegration integration) {
    this.integration = integration;
}

public Mono<ProductAggregate> getCompositeProduct(int productId, int delay, int faultPercent) {
    return Mono.zip(..., integration.getProduct(productId, delay, faultPercent), ...
```

# 添加快速失败回退逻辑

为了在断路器打开时应用回退逻辑，即在请求快速失败时，我们可以捕获断路器打开时抛出的`CircuitBreakerOpenException`异常，并调用回退方法。这必须在断路器之外完成，即在调用者中。在我们的情况下，是`product-composite`服务的实现调用集成类。

在这里，我们使用`onErrorReturn`方法在捕获`CircuitBreakerOpenException`时调用`getProductFallbackValue()`方法：

```java
public Mono<ProductAggregate> getCompositeProduct(int productId, int delay, int faultPercent) {
    return Mono.zip(
        ...
        integration.getProduct(productId, delay, faultPercent)
           .onErrorReturn(CircuitBreakerOpenException.class, 
            getProductFallbackValue(productId)),
        ...
```

回退逻辑可以根据从替代来源获取的产品`productId`查找信息，例如，内部缓存。在我们的情况下，除非`productId`是`13`，否则我们返回一个硬编码的值；否则，我们抛出一个未找到异常：

```java
private Product getProductFallbackValue(int productId) {
    if (productId == 13) {
        throw new NotFoundException("Product Id: " + productId + " not 
        found in fallback cache!");
    }
    return new Product(productId, "Fallback product" + productId, 
    productId, serviceUtil.getServiceAddress());
}
```

# 添加配置

最后，断路器的配置添加到配置存储库中的`product-composite.yml`文件中，如下所示：

```java
app.product-service.timeoutSec: 2

resilience4j.circuitbreaker:
  backends:
    product:
      registerHealthIndicator: true
      ringBufferSizeInClosedState: 5
      failureRateThreshold: 50
      waitInterval: 10000
      ringBufferSizeInHalfOpenState: 3
      automaticTransitionFromOpenToHalfOpenEnabled: true
      ignoreExceptions:
        - se.magnus.util.exceptions.InvalidInputException
        - se.magnus.util.exceptions.NotFoundException
```

配置中的大多数值已经在*介绍断路器*部分中描述过，除了以下内容：

+   `app.product-service.timeoutSec`：用于配置我们之前引入的超时。这个设置为两秒。

+   `registerHealthIndicator`：决定熔断器是否在`health`端点显示信息。这设置为`true`。

# 添加重试机制

与熔断器类似，通过添加依赖项、注解和配置来设置重试机制。依赖项已经在之前添加，所以我们只需要添加注解并设置一些配置。然而，由于重试机制会抛出特定的异常，我们还需要添加一些错误处理逻辑。

# 添加重试注解

重试机制可以通过注解`@Retry(name="nnn")`应用于方法，其中`nnn`是用于此方法的配置条目的名称。关于配置的详细信息，请参见*添加配置*部分。在我们这个案例中，与熔断器相同，是`se.magnus.microservices.composite.product.services.ProductCompositeIntegration`类中的`getProduct()`方法：

```java
@Retry(name = "product")
@CircuitBreaker(name = "product")
public Mono<Product> getProduct(int productId, int delay, int faultPercent) {
```

# 处理重试特定异常

通过`@Retry`注解的方法抛出的异常可以被重试机制用`RetryExceptionWrapper`异常包装。为了能够处理方法抛出的实际异常，例如在抛出`CircuitBreakerOpenException`时应用备用方法，调用者需要添加解包`RetryExceptionWrapper`异常并将它们替换为实际异常的逻辑。

在我们的案例中，是`ProductCompositeServiceImpl`类中的`getCompositeProduct`方法使用 Project Reactor API 对`Mono`对象进行调用。`Mono` API 有一个方便的方法`onErrorMap`，可以用来解包`RetryExceptionWrapper`异常。它被用在`getCompositeProduct`方法中，如下所示：

```java
public Mono<ProductAggregate> getCompositeProduct(int productId, int delay, int faultPercent) {
    return Mono.zip(
        ...
        integration.getProduct(productId, delay, faultPercent)
            .onErrorMap(RetryExceptionWrapper.class, retryException -> 
             retryException.getCause())
            .onErrorReturn(CircuitBreakerOpenException.class, 
             getProductFallbackValue(productId)),
```

# 添加配置

重试机制的配置是以与熔断器相同的方式添加的，即在配置存储库中的`product-composite.yml`文件中，如下所示：

```java
resilience4j.retry:
  backends:
    product:
      maxRetryAttempts: 3
      waitDuration: 1000
      retryExceptions:
      - org.springframework.web.reactive.function.client.WebClientResponseException$InternalServerError
```

实际值在*介绍重试机制*部分进行了讨论。

# 添加自动化测试

已经向`test-em-all.bash`测试脚本中的单独函数`testCircuitBreaker()`添加了电路 breaker 的自动化测试：

```java
...
function testCircuitBreaker() {
    echo "Start Circuit Breaker tests!"
    ...
}
...
testCircuitBreaker
echo "End, all tests OK:" `date`
```

为了能够进行一些必要的验证，我们需要访问`product-composite`微服务的`actuator`端点，这些端点不会通过边缘服务器暴露。因此，我们将通过一个独立的 Docker 容器访问`actuator`端点，这个容器将连接到由 Docker Compose 为我们的微服务设置的内部网络。

默认情况下，网络名称基于放置 Docker Compose 文件的文件夹名称。为了避免这种不确定的依赖关系，在`docker-compose`文件中定义了一个显式的网络名称`my-network`。所有容器定义都已更新，以指定它们应附加到`my-network`网络。以下是来自`docker-compose.yml`的一个例子：

```java
...
  product:
    build: microservices/product-service
    networks:
      - my-network
...
networks:
  my-network:
    name: my-network
```

由于容器附属于内部网络，它可以直接访问产品组合的`actuator`端点，而不需要通过边缘服务器。我们将使用 Alpine 作为 Docker 镜像，并使用`wget`而不是`curl`，因为`curl`默认不包括在 Alpine 发行版中。例如，为了能够找出名为`product`的电路 breaker 在`product-composite`微服务中的状态，我们可以运行以下命令：

```java
docker run --rm -it --network=my-network alpine wget product-composite:8080/actuator/health -qO - | jq -r .details.productCircuitBreaker.details.state
```

命令预期返回值为`CLOSED`。

由于我们使用`--rm`标志创建了 Docker 容器，`wget`命令完成后，Docker 引擎将停止并销毁它。

测试开始执行正好这一点，即在执行测试之前验证断路器是否关闭：

```java
EXEC="docker run --rm -it --network=my-network alpine"
assertEqual "CLOSED" "$($EXEC wget product-composite:8080/actuator/health -qO - | jq -r .details.productCircuitBreaker.details.state)"

```

接下来，测试将依次运行三个命令，迫使断路器打开，所有这些命令都将因为`product`服务响应缓慢而失败：

```java
for ((n=0; n<3; n++))
do
    assertCurl 500 "curl -k https://$HOST:$PORT/product-
    composite/$PROD_ID_REVS_RECS?delay=3 $AUTH -s"
    message=$(echo $RESPONSE | jq -r .message)
 assertEqual "Did not observe any item or terminal signal within 
    2000ms" "${message:0:57}"
done
```

**快速重复配置**：`product`服务的超时设置为两秒，因此三秒的延迟将导致超时。当电路断开时，断路器配置为评估最后五个调用。脚本中先于断路器特定测试的测试已经执行了几次成功的调用。失败阈值设置为 50%，即，三次带有三秒延迟的调用足以打开电路。

在电路断开的情况下，我们期望快速失败，也就是说，我们不需要等待超时就能得到响应。我们还期望调用回退方法返回尽力而为的响应。这也适用于正常调用，即，没有请求延迟。以下代码验证了这一点：

```java
assertCurl 200 "curl -k https://$HOST:$PORT/product-composite/$PROD_ID_REVS_RECS?delay=3 $AUTH -s"
assertEqual "Fallback product2" "$(echo "$RESPONSE" | jq -r .name)"

assertCurl 200 "curl -k https://$HOST:$PORT/product-composite/$PROD_ID_REVS_RECS $AUTH -s"
assertEqual "Fallback product2" "$(echo "$RESPONSE" | jq -r .name)"

```

我们还可以验证模拟未找到错误逻辑在回退方法中按预期工作，即回退方法返回`404`、`NOT_FOUND`对于产品 ID `13`：

```java
assertCurl 404 "curl -k https://$HOST:$PORT/product-composite/$PROD_ID_NOT_FOUND $AUTH -s"
assertEqual "Product Id: $PROD_ID_NOT_FOUND not found in fallback cache!" "$(echo $RESPONSE | jq -r .message)"
```

如配置所示，断路器在`10`秒后会将其状态更改为半打开。为了能够验证这一点，测试等待`10`秒：

```java
echo "Will sleep for 10 sec waiting for the CB to go Half Open..."
sleep 10

```

在验证预期状态（半关闭）后，测试运行三个正常请求，使断路器回到正常状态，这也得到了验证：

```java
assertEqual "HALF_OPEN" "$($EXEC wget product-composite:8080/actuator/health -qO - | jq -r .details.productCircuitBreaker.details.state)"

for ((n=0; n<3; n++))
do
    assertCurl 200 "curl -k https://$HOST:$PORT/product-
    composite/$PROD_ID_REVS_RECS $AUTH -s"
    assertEqual "product name C" "$(echo "$RESPONSE" | jq -r .name)"
done

assertEqual "CLOSED" "$($EXEC wget product-composite:8080/actuator/health -qO - | jq -r .details.productCircuitBreaker.details.state)"
```

**快速重复配置：**断路器在半打开状态下配置为评估前三个调用。因此，我们需要运行三个请求，其中超过 50%的成功率，然后电路才会关闭。

测试通过使用由断路器暴露出的`/actuator/circuitbreakerevents`actuator API 结束，该 API 用于揭示内部事件。例如，它可以用来找出断路器执行了哪些状态转换。我们期望最后三个状态转换如下：

+   首先状态转换：从关闭到开放

+   下一个状态转换：从开放到半关闭

+   最后状态转换：从半关闭到关闭

这由以下代码验证：

```java
assertEqual "CLOSED_TO_OPEN"      "$($EXEC wget product-composite:8080/actuator/circuitbreakerevents/product/STATE_TRANSITION -qO - | jq -r .circuitBreakerEvents[-3].stateTransition)"
assertEqual "OPEN_TO_HALF_OPEN"   "$($EXEC wget product-composite:8080/actuator/circuitbreakerevents/product/STATE_TRANSITION -qO - | jq -r .circuitBreakerEvents[-2].stateTransition)"
assertEqual "HALF_OPEN_TO_CLOSED" "$($EXEC wget product-composite:8080/actuator/circuitbreakerevents/product/STATE_TRANSITION -qO - | jq -r .circuitBreakerEvents[-1].stateTransition)"
```

`jq`表达式`circuitBreakerEvents[-1]`意味着数组中的最后一个事件`[-2]`是倒数第二个事件，而`[-3 ]`是倒数第三个事件。它们一起是三个最新的事件，即我们感兴趣的事件。默认情况下，Resilience4j 为每个断路器保持最后 100 个事件。这可以通过`eventConsumerBufferSize`配置参数进行自定义。

我们在测试脚本中添加了许多步骤，但有了这个，我们可以自动验证我们断路器预期的基本行为是否到位。在下一节，我们将尝试它！

# 尝试断路器和重试机制

现在，是尝试断路器和重试机制的时候了。我们将像往常一样开始，构建 Docker 镜像并运行测试脚本`test-em-all.bash`。之后，我们将手动运行我们之前描述的测试，以确保我们了解发生了什么！我们将执行以下手动测试：

+   断路器的快乐日测试，也就是说，验证在正常操作中断路器是关闭的

+   断路器的负面测试，也就是说，当事情开始出错时，验证断路器是否会打开

+   恢复正常操作，也就是说，一旦问题解决，验证断路器是否回到了关闭状态

+   尝试带有随机错误的的重试机制

# 构建和运行自动化测试

为了构建和运行自动化测试，我们需要做以下工作：

1.  首先，使用以下命令构建 Docker 镜像：

```java
cd $BOOK_HOME/Chapter13
./gradlew build && docker-compose build
```

1.  接下来，在 Docker 中启动系统架构并使用以下命令运行常规测试：

```java
./test-em-all.bash start
```

当测试脚本打印出`Start Circuit Breaker tests!`时，我们之前描述的测试被执行！

# 验证在正常操作中断路器是关闭的

在我们能够调用 API 之前，我们需要一个访问令牌。运行以下命令以获取访问令牌：

```java
unset ACCESS_TOKEN
ACCESS_TOKEN=$(curl -k https://writer:secret@localhost:8443/oauth/token -d grant_type=password -d username=magnus -d password=password -s | jq -r .access_token)
echo $ACCESS_TOKEN
```

尝试一个正常请求并验证它返回 HTTP 响应代码`200`：

```java
curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/2 -w "%{http_code}\n" -o /dev/null -s
```

`-w "%{http_code}\n"`选项用于打印 HTTP 返回状态。只要命令返回`200`，我们就对响应体不感兴趣，因此使用该选项抑制它，即`-o /dev/null`。

使用`health`API 验证断路器是否关闭：

```java
docker run --rm -it --network=my-network alpine wget product-composite:8080/actuator/health -qO - | jq -r .details.productCircuitBreaker.details.state
```

我们期望它响应`CLOSED`。

# 当事情出错时强制打开断路器

现在，是让事情变糟的时候了！我的意思是，是时候尝试一些负测试，以验证当事情开始出错时电路是否会打开。调用 API 三次，并将`product`服务导致超时，即每次调用延迟`3`秒的响应。这应该足以触发断路器：

```java
curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/2?delay=3 -s | jq .
```

我们期望每次都得到如下响应：

```java
{
  "timestamp": "2019-05-03T15:12:57.554+0000",
  "path": "/product-composite/2",
  "status": 500,
  "error": "Internal Server Error",
  "message": "Did not observe any item or terminal signal within 2000ms 
   in 'onErrorResume' (and no fallback has been configured)"
}
```

断路器现在打开了，所以如果你在`waitInterval`内尝试第四次（即`10`秒），你会看到快速失败的响应和回退方法的行动。你将立即得到响应，而不是在`2`秒超时触发后得到错误消息：

```java
{
  "productId": 2,
  "name": "Fallback product2",
  ...
}
```

响应将来自回退方法。这可以通过查看 name 字段中的值来识别，即`Fallback product2`。

快速失败和回退方法是断路器的关键能力！

鉴于我们的配置中设置的等待时间仅为 10 秒，这要求你必须非常迅速，才能看到快速失败和回退方法在行动中！处于半开启状态时，你总是可以提交三个新的请求导致超时，迫使断路器回到开启状态，然后迅速尝试第四个请求。然后，你应该从回退方法中得到一个快速失败的响应！你也可以将等待时间增加到一两分钟，但等待这么长时间才能看到电路切换到半开启状态可能会相当无聊。

等待 10 秒钟，让断路器切换到半开启状态，然后运行以下命令验证电路现在是否处于半开启状态：

```java
docker run --rm -it --network=my-network alpine wget product-composite:8080/actuator/health -qO - | jq -r .details.productCircuitBreaker.details.state
```

预期它会响应`HALF_OPEN`。

# 再次关闭断路器

一旦断路器处于半开启状态，它等待三个调用以确定它应该再次打开电路还是恢复正常，即关闭它。

让我们提交三个普通请求来关闭断路器：

```java
curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/2 -w "%{http_code}\n" -o /dev/null -s
```

它们都应该响应`200`。通过使用`health` API 验证电路是否再次关闭：

```java
docker run --rm -it --network=my-network alpine wget product-composite:8080/actuator/health -qO - | jq -r .details.productCircuitBreaker.details.state
```

我们期望它响应为`CLOSED`。

用以下命令列出最后三个状态转换：

```java
docker run --rm -it --network=my-network alpine wget product-composite:8080/actuator/circuitbreakerevents/product/STATE_TRANSITION -qO - | jq -r '.circuitBreakerEvents[-3].stateTransition, .circuitBreakerEvents[-2].stateTransition, .circuitBreakerEvents[-1].stateTransition'
```

预期它会响应以下命令：

```java
CLOSED_TO_OPEN
OPEN_TO_HALF_OPEN
HALF_OPEN_TO_CLOSED
```

这个响应告诉我们，我们已经将我们的断路器带遍了它的状态图：

+   当错误开始阻止请求成功时，从关闭状态变为开启状态

+   从开启状态变为半开启状态，以查看错误是否消失

+   当错误消失时，即当我们恢复正常操作时，从半开启状态变为关闭状态

# 尝试由随机错误引起的重试

让我们模拟我们的`product`服务或与其通信存在一个-希望是暂时的-随机问题。

我们可以通过使用`faultPercent`参数来实现。如果我们将其设置为`25`，我们期望每个第四个请求都会失败。我们希望重试机制会自动重试请求来帮助我们。注意到重试机制已经启动的一个方法是测量`curl`命令的响应时间。正常响应应该不会超过 100 毫秒。由于我们配置了重试机制等待一秒钟（参见前面的重试机制中的`waitDuration`参数），我们期望每次重试尝试的响应时间会增加一秒钟。要强制发生随机错误，多次运行以下命令：

```java
time curl -H "Authorization: Bearer $ACCESS_TOKEN" -k https://localhost:8443/product-composite/2?faultPercent=25 -w "%{http_code}\n" -o /dev/null -s
```

命令应当返回`200`状态码，表示请求成功。响应时间前缀为`real`的，例如`real 0m0.078s`，意味着响应时间为 0.078 秒或 78 毫秒。正常的响应，即没有进行任何重试的响应，应该如下所示：

```java
200
real 0m0.078s
...
```

一次重试后的响应应该如下所示：

```java
200
real 0m1.077s
```

HTTP 状态码 200 表示请求已经成功，即使它需要重试一次才能成功！

在你注意到响应时间为一秒之后，即请求需要重试一次才能成功时，运行以下命令来查看最后的两次重试事件：

```java
docker run --rm -it --network=my-network alpine wget product-composite:8080/actuator/retryevents -qO - | jq '.retryEvents[-2], .retryEvents[-1]'
```

你应该能够看到失败的请求和下一次成功的尝试。`creationTime`时间戳预计会相差一秒钟。期待如下的响应：

```java
{
  "retryName": "product",
  "type": "RETRY",
  "creationTime": "2019-05-01T05:40:18.458858Z[Etc/UTC]",
  "errorMessage": "org.springframework.web.reactive.
    function.client.WebClientResponseException$InternalServerError: 500 
    Internal Server Error",
  "numberOfAttempts": 1
}
{
  "retryName": "product",
  "type": "SUCCESS",
  "creationTime": "2019-05-01T05:40:19.471136Z[Etc/UTC]",
  "numberOfAttempts": 1
}
```

如果你真的非常倒霉，你会连续得到两个错误，然后你的响应时间会变成两秒而不是一秒。如果你重复执行前面的命令，你可以看到`numberOfAttempts`字段对每次重试尝试进行计数，在本例中设置为`2`：`"numberOfAttempts": 2`。如果调用继续失败，熔断器将启动并打开其电路，即后续的调用将会快速失败并应用回退方法！

**就是这么简单！**

随意发挥配置中的参数，以更好地了解熔断器和重试机制！

# 总结

在本章中，我们看到了 Resilience4j 及其熔断器和重试机制的实际应用。

当熔断器打开时，使用快速失败和`fallback`方法，可以防止微服务在它依赖的正常响应的同步服务停止响应时变得无响应。熔断器还可以通过在半开状态下允许请求来使微服务具有弹性，以查看失败的服务是否再次正常运行并关闭电路。

重试机制可以使微服务具有弹性，通过重试偶尔由于临时网络问题而失败的请求。非常重要的一点是，只有对幂等性服务应用重试请求，也就是说，可以处理相同请求发送两次或多次的服务。

断路器和重试机制遵循 Spring Boot 约定实现，即声明依赖项，并添加注解和配置。Resilience4j 在运行时通过`actuator`端点暴露有关其断路器和重试机制的信息，包括断路器和事件以及重试的事件和度量指标。

在本章中，我们看到了健康和事件端点的使用，但我们必须等到第二十章，*监控微服务*，我们才能使用任何度量指标。

在下一章中，我们将涵盖使用 Spring Cloud 的最后部分，届时我们将学习如何使用 Spring Cloud Sleuth 和 Zipkin 通过一组协作的微服务跟踪调用链。前往第十四章，*理解分布式跟踪*，开始学习吧！

# 问题

+   断路器有哪些状态，它们是如何使用的？

+   我们如何处理断路器中的超时错误？

+   当断路器快速失败时，我们如何应用回退逻辑？

+   重试机制和断路器如何相互干扰？

+   提供一个无法应用重试机制的服务示例。
