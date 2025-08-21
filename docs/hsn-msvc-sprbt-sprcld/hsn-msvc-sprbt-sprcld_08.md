# 第七章：开发反应式微服务

在本章中，我们将学习如何开发反应式微服务，即如何使用 Spring 开发非阻塞同步 REST API 和基于事件的异步服务。我们还将学习如何在这两种替代方案之间进行选择。最后，我们将了解如何创建和运行反应式微服务架构的手动和自动化测试。

正如在第一章的*响应式微服务*部分所描述的，反应式系统的基础是它们是消息驱动的——它们使用异步通信。这使得它们具有弹性，即可伸缩和有韧性，意味着它们将能够忍受失败。弹性和韧性相结合将使反应式系统能够变得*响应性*；它们将能够及时做出反应。

本章将涵盖以下主题：

+   在非阻塞同步 API 和基于事件的异步服务之间进行选择

+   使用 Spring 开发非阻塞同步 REST API

+   开发基于事件驱动的异步服务

+   反应式微服务架构的手动测试

+   反应式微服务架构的自动化测试

# 技术要求

本书中描述的所有命令都是在 MacBook Pro 上使用 macOS Mojave 运行的，但应该很容易修改，以便它们可以在其他平台如 Linux 或 Windows 上运行。

在本章中不需要安装任何新工具。

本章的源代码可以在 GitHub 上找到：[`github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud/tree/master/Chapter07`](https://github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud/tree/master/Chapter07)。

为了能够运行书中描述的命令，将源代码下载到一个文件夹中，并设置一个环境变量`$BOOK_HOME`，使其指向该文件夹。一些示例命令如下：

```java
export BOOK_HOME=~/Documents/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud
git clone https://github.com/PacktPublishing/Hands-On-Microservices-with-Spring-Boot-and-Spring-Cloud $BOOK_HOME
cd $BOOK_HOME/Chapter07
```

Java 源代码是为 Java 8 编写的，并在 Java 12 上进行了测试。本章使用 Spring Cloud 2.1.0（也称为**Greenwich**版本），Spring Boot 2.1.2 和 Spring 5.1.4，这些是编写本章时可用的 Spring 组件的最新版本。

源代码包含以下 Gradle 项目：

+   `api`

+   `util`

+   `microservices/product-service`

+   `microservices/review-service`

+   `microservices/recommendation-service`

+   `microservices/product-composite-service`

本章中的代码示例均来自`$BOOK_HOME/Chapter07`的源代码，但在许多情况下进行了编辑，以删除源代码中不相关的内容，例如注释和`import`以及日志语句。

在本章*中，*您可以查看已对源代码所做的更改以及使微服务变得响应式所需的努力。此代码可与第六章的*添加持久化*源代码进行比较。您可以使用您喜欢的`diff`工具并比较两个文件夹—`$BOOK_HOME/Chapter06`和`$BOOK_HOME/Chapter07`。

# 在非阻塞的同步 API 和事件驱动的异步服务之间进行选择

在开发响应式微服务时，并不总是明显何时使用非阻塞的同步 API，何时使用事件驱动的异步服务。通常，为了使微服务具有鲁棒性和可伸缩性，使其尽可能自治是很重要的，例如，最小化其运行时依赖。这也被称为**松耦合**。因此，异步消息传递事件优于同步 API。这是因为微服务仅在运行时依赖于对消息系统的访问，而不是依赖于对多个其他微服务的同步访问。

然而，有许多情况下使用非阻塞的同步 API 可能是合适的，例如：

+   对于读操作，用户端正在等待响应

+   客户端平台更适合消耗同步 API，例如，移动应用或 SPA 网络应用

+   客户端将连接到来自其他组织的服务—在这些情况下，可能很难就跨组织使用的共同消息系统达成一致

对于本书中使用的系统架构，我们将使用以下内容：

+   产品组合微服务暴露的创建、读取和删除服务将基于同步 API。组合微服务假定具有 web 和移动平台以及来自其他组织（而非操作系统架构的组织）的客户端。因此，同步 API 似乎是一个自然的匹配。

+   核心微服务提供的读取服务也将开发为非阻塞的同步 API，因为有一个终端用户在等待它们的响应。

+   核心微服务提供的创建和删除服务将开发为事件驱动的异步服务。组合微服务提供的创建和删除聚合产品信息的同步 API 将简单地在核心服务监听的主题上发布、创建和删除事件，然后返回 200（OK）响应。

以下图表说明了这一点：

![](img/a8f1c5bd-e29b-4f4d-a527-a0f86fd863b1.png)

首先，让我们学习如何开发非阻塞的同步 REST API，之后，我们将查看如何开发事件驱动的异步服务。

# 使用 Spring 开发非阻塞的同步 REST API

在本节中，我们将学习如何开发读取 API 的非阻塞版本。复合服务将对三个核心服务并行地做出反应性的，即非阻塞的调用。当复合服务从核心服务收到响应后，它将创建一个复合响应并将其发送回调用者。以下图示说明了这一点：

![](img/676c4d79-653b-438c-b8ff-7ffe0477644d.png)

我们将介绍以下内容：

+   介绍 Spring Reactor

+   使用 Spring Data for MongoDB 进行非阻塞持久化

+   核心服务中的非阻塞 REST API，包括如何处理基于 JPA 的持久化层的阻塞代码

+   非阻塞 REST API 在复合服务中

# 介绍 Spring Reactor

正如我们在第二章《Spring Boot 入门》中的 *Beginning with Spring WebFlux* 部分提到的，Spring 5 中的反应式支持基于 **Project Reactor** ([`projectreactor.io`](https://projectreactor.io))。 Project Reactor 基于 *Reactive Streams 规范* ([`www.reactive-streams.org`](http://www.reactive-streams.org))，用于构建反应式应用程序的标准。 Spring Reactor 是基础，它是 Spring WebFlux、Spring WebClient 和 Spring Data 提供其反应性和非阻塞特性的依赖。

编程模型基于处理数据流，Project Reactor 的核心数据类型是 `Flux` 和 `Mono`。`Flux` 对象用于处理一个元素流 *0*...*n*，而 `Mono` 对象用于处理 *0*...*1* 个元素。在本章中我们将看到许多使用它们的示例。作为一个简短的介绍，让我们看看下面的测试：

```java
@Test
public void TestFlux() {

    List<Integer> list = new ArrayList<>();

    Flux.just(1, 2, 3, 4)
        .filter(n -> n % 2 == 0)
        .map(n -> n * 2)
        .log()
        .subscribe(n -> list.add(n));

    assertThat(list).containsExactly(4, 8);
}
```

以下是前面源代码的解释：

1.  我们用整数 `1`、`2`、`3` 和 `4` 初始化流。

1.  接下来，我们 `filter` 掉奇数——我们只允许偶数通过流继续进行——在这个测试中，这些是 `2` 和 `4`。

1.  接下来，我们将通过乘以 `2` 对流中的值进行转换，即得到 `4` 和 `8`。

1.  然后，我们在 `map` 操作后的流中 `log` 数据。

1.  到目前为止，我们只是声明了数据流 processing。要实际处理数据流，我们需要有人来订阅它。`subscribe` 方法的最终调用将注册一个订阅者，订阅者将对从流中获取的每个元素应用 `subscribe` 方法中的 lambda 函数。此后，它将把它们添加到 `list` 元素。

1.  最后，我们可以断言，在数据流处理后 `list` 包含期望的结果——整数 `4` 和 `8`。

日志输出将如下代码所示：

```java
20:01:45.714 [main] INFO reactor.Flux.MapFuseable.1 - | onSubscribe([Fuseable] FluxMapFuseable.MapFuseableSubscriber)
20:01:45.716 [main] INFO reactor.Flux.MapFuseable.1 - | request(unbounded)
20:01:45.716 [main] INFO reactor.Flux.MapFuseable.1 - | onNext(4)
20:01:45.717 [main] INFO reactor.Flux.MapFuseable.1 - | onNext(8)
20:01:45.717 [main] INFO reactor.Flux.MapFuseable.1 - | onComplete()
```

以下是前面源代码的解释：

1.  数据流的 processing 是由一个订阅者启动的，该订阅者订阅流并请求其内容。

1.  接下来，整数 `4` 和 `8` 通过了 `log` 操作。

1.  处理以调用订阅者的`onComplete`方法结束，通知它流已经结束。

完整的源代码请参阅`util`项目中的`se.magnus.util.reactor.ReactorTests`测试类。

通常，我们不会初始化流的处理。相反，我们只定义它应该如何被处理，而发起处理的职责将留给一个基础架构组件，比如 Spring WebFlux，例如，作为对传入 HTTP 请求的响应。这个规则的一个例外是阻塞代码需要从反应式流中获取响应的情况。在这些情况下，阻塞代码可以调用`Flux`或`Mono`对象上的`block()`方法，以阻塞方式从`Flux`或`Mono`对象获取响应。

# 非阻塞式持久化使用 Spring Data for MongoDB

将基于 MongoDB 的`product`和`recommendation`服务的存储库变为反应式非常简单：

+   将`ReactiveCrudRepository`基类更改为存储库

+   将自定义查找方法更改为返回一个`Mono`或`Flux`对象

更改后的`ProductRepository`和`RecommendationRepository`看起来像这样：

```java
public interface ProductRepository extends ReactiveCrudRepository<ProductEntity, String> {
    Mono<ProductEntity> findByProductId(int productId);
}

public interface RecommendationRepository extends ReactiveCrudRepository<RecommendationEntity, String> {
    Flux<RecommendationEntity> findByProductId(int productId);
}
```

对于`review`服务的持久化代码没有进行任何更改，它将保持使用 JPA 存储库的阻塞式！

完整的源代码请参考以下类：

+   `se.magnus.microservices.core.product.persistence.ProductRepository`在`product`项目中。

+   `se.magnus.microservices.core.recommendation.persistence.RecommendationRepository`在`recommendation`项目中。

# 测试代码的变化

当涉及到测试持久层时，我们必须做一些改变。由于我们现在的持久化方法返回了一个`Mono`或`Flux`对象，测试方法必须等待响应在返回的反应式对象中可用。测试方法可以调用`Mono`/`Flux`对象的`block()`方法来等待响应可用，或者使用来自 Project Reactor 的`StepVerifier`帮助类来声明一个可验证的异步事件序列。

下面的示例展示了如何更改测试代码以适应存储库的反应式版本：

```java
ProductEntity foundEntity = repository.findById(newEntity.getId()).get();
assertEqualsProduct(newEntity, foundEntity);
```

我们可以在`repository.findById()`方法返回的`Mono`对象上调用`block()`方法，并保持命令式编程风格，如下所示：

```java
ProductEntity foundEntity = repository.findById(newEntity.getId()).block();
assertEqualsProduct(newEntity, foundEntity);
```

另外，我们可以使用`StepVerifier`类来设置一个处理步骤序列，既执行存储库查找操作，又验证结果。该序列通过最终调用`verifyComplete()`方法来初始化，如下所示：

```java
StepVerifier.create(repository.findById(newEntity.getId()))
    .expectNextMatches(foundEntity -> areProductEqual(newEntity, 
     foundEntity))
    .verifyComplete();
```

有关使用`StepVerifier`类编写测试的示例，请参阅`product`项目中的`se.magnus.microservices.core.product.PersistenceTests`测试类。

有关使用`block()`方法编写测试的相应示例，请参阅`recommendation`项目中的`se.magnus.microservice.core.recommendation.PersistenceTests`测试类。

# 核心服务的非阻塞 REST API

在非阻塞持久层就位之后，是时候也让核心服务的 API 变为非阻塞式的了。我们需要进行以下更改：

+   修改 API，使它们只返回反应式数据类型

+   修改服务实现，使它们不包含任何阻塞代码

+   修改我们的测试，使它们能够测试反应式服务

+   处理阻塞代码—将仍需阻塞的代码与非阻塞代码隔离

# API 的变化

为了使核心服务的 API 变为反应式的，我们需要更新它们的方法，使它们返回一个`Mono`或`Flux`对象。

例如，`product`服务中的`getProduct()`现在返回`Mono<Product>`而不是一个`Product`对象：

```java
Mono<Product> getProduct(@PathVariable int productId);
```

完整的源代码请参阅`api`项目中的以下类：

+   `se.magnus.api.core.product.ProductService`

+   `se.magnus.api.core.recommendation.RecommendationService`

+   `se.magnus.api.core.review.ReviewService`

# 服务实现的变化

对于在`product`和`recommendation`服务中使用反应式持久层的服务实现，我们可以使用 Project Reactor 中的流式 API。例如，`getProduct()`方法的实现如下所示：

```java
public Mono<Product> getProduct(int productId) {

    if (productId < 1) throw new InvalidInputException("Invalid 
        productId: " + productId);

    return repository.findByProductId(productId)
        .switchIfEmpty(error(new NotFoundException("No product found 
         for productId: " + productId)))
        .log()
        .map(e -> mapper.entityToApi(e))
        .map(e -> {e.setServiceAddress(serviceUtil.getServiceAddress()); return e;});
} 
```

以下是前述源代码的解释：

1.  该方法将返回一个`Mono`对象；这里的处理是声明式的，而不是触发式的。一旦`WebFlux`接收到对此服务的请求，它就会被触发！

1.  将使用其`productId`从底层数据库中检索产品，使用持久性仓库中的`findByProductId()`方法。

1.  如果为给定的`productId`找不到产品，将抛出`NotFoundException`。

1.  `log`方法将产生日志输出。

1.  将调用`mapper.entityToApi()`方法将来自持久层返回的实体转换为 API 模型对象。

1.  最终的`map`方法将在模型对象的`serviceAddress`字段中设置处理请求的微服务的 DNS 名称和 IP 地址。

成功处理的一些示例日志输出如下：

```java
2019-02-06 10:09:47.006 INFO 62314 --- [ctor-http-nio-2] reactor.Mono.SwitchIfEmpty.1 : onSubscribe(FluxSwitchIfEmpty.SwitchIfEmptySubscriber)
2019-02-06 10:09:47.007 INFO 62314 --- [ctor-http-nio-2] reactor.Mono.SwitchIfEmpty.1 : request(unbounded)
2019-02-06 10:09:47.034 INFO 62314 --- [ntLoopGroup-2-2] reactor.Mono.SwitchIfEmpty.1 : onNext(ProductEntity: 1)
2019-02-06 10:09:47.048 INFO 62314 --- [ntLoopGroup-2-2] reactor.Mono.SwitchIfEmpty.1 : onComplete()
```

以下是处理失败的一个示例（抛出一个未找到异常）：

```java
2019-02-06 10:09:52.643 INFO 62314 --- [ctor-http-nio-3] reactor.Mono.SwitchIfEmpty.2 : onSubscribe(FluxSwitchIfEmpty.SwitchIfEmptySubscriber)
2019-02-06 10:09:52.643 INFO 62314 --- [ctor-http-nio-3] reactor.Mono.SwitchIfEmpty.2 : request(unbounded)
2019-02-06 10:09:52.648 ERROR 62314 --- [ntLoopGroup-2-2] reactor.Mono.SwitchIfEmpty.2 : onError(se.magnus.util.exceptions.NotFoundException: No product found for productId: 2)
2019-02-06 10:09:52.654 ERROR 62314 --- [ntLoopGroup-2-2] reactor.Mono.SwitchIfEmpty.2 : 

se.magnus.util.exceptions.NotFoundException: No product found for productId: 2
 at se.magnus.microservices.core.product.services.ProductServiceImpl.getProduct(ProductServiceImpl.java:58) ~[classes/:na]
 ...
```

完整的源代码请参阅以下类：

+   `product`项目中的`se.magnus.microservices.core.product.services.ProductServiceImpl`

+   `recommendation`项目中的`se.magnus.microservices.core.recommendation.services.RecommendationServiceImpl`

# 测试代码的变化

服务实现测试代码已经按照我们之前描述的持久层测试进行了更改。为了处理反应式返回类型的异步行为，`Mono` 和 `Flux`，测试中混合了调用`block()`方法和使用`StepVerifier` 助手类。

完整的源代码可以在以下测试类中找到：

+   `se.magnus.microservices.core.product.ProductServiceApplicationTests` 在 `product` 项目中

+   `se.magnus.microservices.core.recommendation.RecommendationServiceApplicationTests` 在 `recommendation` 项目中

# 处理阻塞代码

对于使用 JPA 在其关系型数据库中访问数据的`review` 服务，我们不支持非阻塞编程模型。相反，我们可以使用`Scheduler`来运行阻塞代码，它能够在有限线程数的专用线程池中运行线程。使用线程池来运行阻塞代码，避免了耗尽微服务中可用的线程（避免了影响微服务中的非阻塞处理）。

让我们看看这个过程是如何按照以下步骤展开的：

1.  首先，我们在`main` `ReviewServiceApplication` 类中配置线程池，如下所示：

```java
@Autowired
public ReviewServiceApplication (
    @Value("${spring.datasource.maximum-pool-size:10}") Integer 
    connectionPoolSize
) {
    this.connectionPoolSize = connectionPoolSize;
}

@Bean
public Scheduler jdbcScheduler() {
    LOG.info("Creates a jdbcScheduler with connectionPoolSize = " + 
    connectionPoolSize);
    return Schedulers.fromExecutor(Executors.newFixedThreadPool
    (connectionPoolSize));
}
```

我们可以使用`spring.datasource.maximum-pool-size` 参数配置线程池的大小。如果没有设置，它将默认为 10 个线程。完整的源代码可以在`se.magnus.microservices.core.review.ReviewServiceApplication` 类中找到，该类在`review` 项目中。

1.  接下来，我们将调度器注入到`review` 服务实现类中，如下所示：

```java
@RestController
public class ReviewServiceImpl implements ReviewService {

    private final Scheduler scheduler;

    @Autowired
    public ReviewServiceImpl(Scheduler scheduler, ...) {
        this.scheduler = scheduler;
    }
```

1.  最后，我们在反应式实现中的`getReviews()` 方法中使用线程池，如下所示：

```java
@Override
public Flux<Review> getReviews(int productId) {

    if (productId < 1) throw new InvalidInputException("Invalid 
        productId: " + productId);

    return asyncFlux(getByProductId(productId)).log();
}

protected List<Review> getByProductId(int productId) {

    List<ReviewEntity> entityList = 
    repository.findByProductId(productId);
    List<Review> list = mapper.entityListToApiList(entityList);
    list.forEach(e -> 
            e.setServiceAddress(serviceUtil.getServiceAddress()));

    LOG.debug("getReviews: response size: {}", list.size());

    return list;
}

private <T> Flux<T> asyncFlux(Iterable<T> iterable) {
    return Flux.fromIterable(iterable).publishOn(scheduler);
}
```

以下是前述代码的解释：

+   阻塞代码放在了`getByProductId()` 方法中

+   `getReviews()` 方法使用`asyncFlux()` 方法在线程池中运行阻塞代码

完整的源代码可以在`se.magnus.microservices.core.review.services.ReviewServiceImpl` 类中找到，该类在`review` 项目中。

# 复合服务中的非阻塞 REST API

为了使复合服务中的 REST API 非阻塞，我们需要做以下工作：

+   更改 API，使其只返回反应式数据类型

+   更改集成层，使其使用非阻塞 HTTP 客户端

+   更改服务实现，使其以并行和非阻塞的方式调用核心服务 API

+   更改我们的测试，以便它们可以测试反应式服务

# API 的更改

为了使复合服务的 API 反应式，我们需要应用与之前描述的核心服务 API 相同的更改。这意味着`getCompositeProduct` 方法的返回类型`ProductAggregate`需要替换为`Mono<ProductAggregate>`。

完整的源代码可以在`se.magnus.api.composite.product.ProductCompositeService` 类中找到，该类在`api` 项目中。

# 集成层的变更

在`ProductCompositeIntegration`集成类中，我们将`RestTemplate`阻塞式 HTTP 客户端替换为 Spring 5 提供的`WebClient`非阻塞式 HTTP 客户端。

`WebClient`的构建器自动注入到构造函数中。如果需要自定义，例如设置公共头或过滤器，可以在构造函数中完成。有关可用的配置选项，请参阅[`docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-client-builder`](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-client-builder)。请查看以下步骤：

1.  在这里，我们简单地构建了将在集成类中使用的`WebClient`实例，而不进行任何配置：

```java
public class ProductCompositeIntegration implements ProductService, RecommendationService, ReviewService {

    private final WebClient webClient;

    @Autowired
    public ProductCompositeIntegration(
        WebClient.Builder webClient, ...
    ) {
        this.webClient = webClient.build();
    }
```

1.  接下来，我们使用`webClient`实例来调用`product`服务的非阻塞请求：

```java
@Override
public Mono<Product> getProduct(int productId) {
    String url = productServiceUrl + "/product/" + productId;

    return webClient.get().uri(url).retrieve().bodyToMono(Product.class).log().onErrorMap(WebClientResponseException.class, ex -> handleException(ex));
}
```

如果对`product`服务的 API 调用失败，整个请求将会失败。`WebClient onErrorMap()`方法将调用我们的`handleException(ex)`方法，该方法将之前由 HTTP 层抛出的异常映射到我们自己的异常，例如`NotFoundException`和`InvalidInputException`。

然而，如果对`product`服务的调用成功，但对推荐或评论 API 的调用失败，我们不希望让整个请求失败。相反，我们希望能够返回尽可能多的可用信息给调用者。因此，在这些情况下，我们不会传播异常，而是使用`WebClient onErrorResume(error -> empty())`方法返回推荐或评论的空列表。考虑以下代码：

```java
@Override
public Flux<Recommendation> getRecommendations(int productId) {

    String url = recommendationServiceUrl + "/recommendation?
    productId=" + productId;

    // Return an empty result if something goes wrong to make it 
    // possible for the composite service to return partial responses
    return webClient.get().uri(url).retrieve().bodyToFlux(Recommendation.class).log().onErrorResume(error -> empty());
}
```

要查看完整的源代码，请参阅`product-composite`项目中的`se.magnus.microservices.composite.product.services.ProductCompositeIntegration`类。

# 服务实现变更

为了能够并行调用三个 API，服务实现使用了`Mono`类上的静态`zip()`方法。`zip`方法能够处理多个并行请求，并在它们都完成后将它们组合在一起。代码如下：

```java
@Override
public Mono<ProductAggregate> getCompositeProduct(int productId) {
    return Mono.zip(
        values -> createProductAggregate((Product) values[0], 
        (List<Recommendation>) values[1], (List<Review>) values[2], 
        serviceUtil.getServiceAddress()),
        integration.getProduct(productId),
        integration.getRecommendations(productId).collectList(),
        integration.getReviews(productId).collectList())
        .doOnError(ex -> LOG.warn("getCompositeProduct failed: {}", 
         ex.toString()))
        .log();
}
```

以下是先前源代码的解释：

1.  `zip`方法的第一参数是一个 lambda 函数，该函数将接收响应数组。三个 API 调用响应的实际聚合由之前的同一个助手方法处理，即`createProductAggregate`，没有进行任何更改。

1.  在 lambda 函数后面的参数是一个请求列表，`zip`方法将并行调用这些请求，每个请求对应一个`Mono`对象。在我们这个案例中，我们发送了三个由集成类方法创建的`Mono`对象，每个对象对应发送到每个核心微服务的每个请求。

要查看完整的源代码，请参阅`product-composite`项目中的`se.magnus.microservices.composite.product.services.ProductCompositeServiceImpl`类。

# 测试代码中的更改

测试类中唯一需要更改的是更新集成类的 mock 设置，以便使用`Mono.just()`帮助方法和`Flux.fromIterable()`返回`Mono`和`Flux`对象，如下面的代码所示：

```java
public class ProductCompositeServiceApplicationTests {

    @Before
    public void setUp() {

        when(compositeIntegration.getProduct(PRODUCT_ID_OK)).
            thenReturn(just(new Product(PRODUCT_ID_OK, "name", 1, 
             "mock-address")));

        when(compositeIntegration.getRecommendations(PRODUCT_ID_OK)).
            thenReturn(Flux.fromIterable(singletonList(new 
             Recommendation(PRODUCT_ID_OK, 1, "author", 1, "content", 
             "mock address"))));

        when(compositeIntegration.getReviews(PRODUCT_ID_OK)).
            thenReturn(Flux.fromIterable(singletonList(new 
             Review(PRODUCT_ID_OK, 1, "author", "subject", "content", 
             "mock address"))));
```

完整的源代码，请参阅`product-composite`项目中的`se.magnus.microservices.composite.product.ProductCompositeServiceApplicationTests`测试类。

现在我们已经使用 Spring 开发了非阻塞 REST API，是时候开发一个基于事件的同步服务了。

# 开发基于事件的异步服务

在本节中，我们将学习如何开发基于事件的异步创建和删除服务版本。组合服务将在每个核心服务主题上发布创建和删除事件，然后不等待核心服务中的处理，向调用者返回一个 OK 响应。以下图表说明了这一点：

![](img/6a55381b-4a64-4aa4-84fb-02f258aca361.png)

我们将涵盖以下主题：

+   配置 Spring Cloud Stream 以处理消息传递挑战

+   定义主题和事件

+   Gradle 构建文件中的更改

+   在组合服务中发布事件

+   在核心服务中消费事件

# 配置 Spring Cloud Stream 以处理消息传递挑战

为了实现基于事件创建和删除服务，我们将使用 Spring Cloud Stream。在第二章，《Spring Boot 入门》中的*Spring Cloud Stream*部分，我们已经看到了使用 Spring Cloud Stream 在主题上发布和消费消息是多么简单。

例如，要发布一个由`mysource`定义的主题上的消息，我们只需要写以下内容：

```java
mysource.output().send(MessageBuilder.withPayload(message).build());
```

为了消费消息，我们编写以下代码：

```java
@StreamListener(target = Sink.INPUT)
 public void receive(MyMessage message) {
   LOG.info("Received: {}",message);
```

这种编程模型可以独立于使用的消息系统，例如，RabbitMQ 或 Apache Kafka！

尽管异步消息传递优先于同步 API 调用，但它带来了挑战。我们将了解如何使用 Spring Cloud Stream 来处理其中一些问题。以下 Spring Cloud Stream 功能将得到覆盖：

+   消费者群体

+   重试和死信队列

+   保证顺序和分区

我们将在以下章节中研究每个这些内容。

# 消费者群体

这里的问题在于，如果我们增加消息消费者的实例数量，例如，启动产品微服务的两个实例，两个产品微服务实例都将消费相同的消息，如下面的图表所示：

![](img/37c34a80-a777-4301-b2dd-c36e1ac8b50d.png)

这个问题的解决方案是我们只希望每个消费者实例处理每条消息。这可以通过引入一个*消费者组*来解决，如下面的图表所示：

![](img/e4713a81-9342-4fa1-a32d-ea3f544a7596.png)

在 Spring Cloud Stream 中，消费者组可以在消费者端进行配置，例如，对于产品微服务，如下所示：

```java
spring.cloud.stream:
  bindings.input:
    destination: products
    group: productsGroup
```

在前面的配置中，Spring Cloud Stream 将使用`group`字段的值将`product`微服务的实例添加到名为`productsGroup`的消费者组中。这意味着发送到`products`主题的消息将只由 Spring Cloud Stream 交付给产品微服务的一个实例。

# 重试和死信队列

在本节中，我们将学习消息消费者如何使用重试和死信队列。

如果消费者未能处理消息，它可能会丢失或被重新排队，直到失败消费者成功处理。如果消息内容无效，也称为**毒消息**，它将阻塞消费者处理其他消息，直到手动移除。如果失败是由于临时问题，例如，由于临时网络错误无法访问数据库，经过多次重试后处理可能会成功。

必须能够指定重试次数，直到消息被移动到另一个存储进行故障分析和修正。失败的消息通常会被移动到一个专门的队列，称为死信队列。为了避免在临时故障时，例如网络错误，过度负载基础架构，必须能够配置重试的频率，最好每次重试之间的时间逐渐增加。

在 Spring Cloud Stream 中，这可以在消费者端进行配置，例如，对于产品微服务，如下所示：

```java
spring.cloud.stream.bindings.input.consumer:
  maxAttempts: 3
  backOffInitialInterval: 500
  backOffMaxInterval: 1000
  backOffMultiplier: 2.0

spring.cloud.stream.rabbit.bindings.input.consumer:
  autoBindDlq: true
  republishToDlq: true

spring.cloud.stream.kafka.bindings.input.consumer:
  enableDlq: true
```

在前面的示例中，我们指定 Spring Cloud Stream 在将消息放置到死信队列之前应执行`3`次重试。第一次重试将在`500`毫秒后尝试，其余两次尝试将在`1000`毫秒后进行。

启用死信队列的使用是与绑定特定的；因此，我们有针对 RabbitMQ 和 Kafka 各一个配置。

# 保证顺序和分区

我们可以使用分区来确保消息按发送顺序交付，同时不失去性能和可扩展性。

如果业务逻辑要求消息按发送顺序被消费和处理，我们不能为了提高处理性能而使用每个消费者多个实例；例如，我们不能使用消费者组。在某些情况下，这可能导致处理传入消息时出现不可接受的延迟。

在大多数情况下，消息处理中的严格顺序仅对影响相同业务实体的消息 required，例如，产品。

例如，影响产品 ID 为`1`的消息在很多情况下可以独立于影响产品 ID 为`2`的消息进行处理。这意味着只需要为具有相同产品 ID 的消息保证顺序。

这个问题的解决办法是，使其能够为每个消息指定一个键，消息传递系统可以使用该键来保证具有相同键的消息之间的顺序。这可以通过在主题中引入子主题（也称为**分区**）来解决。消息传递系统根据其键将消息放置在特定的分区中。具有相同键的消息总是放置在同一个分区中。消息传递系统只需要保证同一分区的消息的交付顺序。为了确保消息的顺序，我们在消费者组内的每个分区配置一个消费者实例。通过增加分区数，我们可以允许消费者增加其实例数。这在不失去交付顺序的情况下增加了其处理消息的性能。这在下面的图中说明：

![](img/2962fb3e-d606-4550-8328-c846114f8965.png)

在 Spring Cloud Stream 中，这需要在发布者和消费者双方进行配置。在发布者方面，必须指定键和分区数。例如，对于`product-composite`服务，我们有以下内容：

```java
spring.cloud.stream.bindings.output:
  destination: products
  producer:
    partition-key-expression: payload.key
    partition-count: 2
```

前面的配置意味着将使用名为`key`的字段从消息负载中获取键，并使用两个分区。

每个消费者可以指定它想要接收消息的分区。例如，对于`product`微服务，我们有以下内容：

```java
spring.cloud.stream.bindings.input:
  destination: products
  group:productsGroup
  consumer:
    partitioned: true
    instance-index: 0
```

前面的配置告诉 Spring Cloud Stream 这个消费者只将接收来自分区编号`0`的消息，即第一个分区。

# 定义主题和事件

正如我们在第二章的*Spring Cloud Stream*部分提到的，*Spring Boot 入门*，Spring Cloud Stream 基于发布和订阅模式，发布者将消息发布到主题，订阅者订阅他们感兴趣的主题以接收消息。

我们将为每种类型的实体使用一个**主题**：`products`、`recommendations`和`reviews`。

消息传递系统处理**消息**，这些消息通常由标题和正文组成。**事件**是描述已经发生的事情的消息。对于事件，消息正文可以用来描述事件类型、事件数据以及事件发生的日期时间戳。

事件在本书的范围内由以下内容定义：

+   事件**类型**，例如，创建或删除事件

+   一个**键**，用于标识数据，例如，产品 ID

+   一个**数据**元素，即事件中的实际数据

+   一个**时间戳**，描述事件发生的时间

我们将使用的事件类如下所示：

```java
public class Event<K, T> {

    public enum Type {CREATE, DELETE}

    private Event.Type eventType;
    private K key;
    private T data;
    private LocalDateTime eventCreatedAt;

    public Event() {
        this.eventType = null;
        this.key = null;
        this.data = null;
        this.eventCreatedAt = null;
    }

    public Event(Type eventType, K key, T data) {
        this.eventType = eventType;
        this.key = key;
        this.data = data;
        this.eventCreatedAt = now();
    }

    public Type getEventType() {
        return eventType;
    }

    public K getKey() {
        return key;
    }

    public T getData() {
        return data;
    }

    public LocalDateTime getEventCreatedAt() {
        return eventCreatedAt;
    }
}
```

让我们详细解释一下前面的源代码：

+   `Event`类是一个泛型类，其`key`和`data`字段类型为`K`和`T`。

+   事件类型被声明为一个枚举器，其允许的值是，即`CREATE`和`DELETE`。

+   这个类定义了两个构造函数，一个空构造函数和一个可以用来初始化类型、键和值成员的构造函数。

+   最后，这个类为其成员变量定义了 getter 方法。

要查看完整的源代码，请参阅`api`项目中的`se.magnus.api.event.Event`类。

# 在 Gradle 构建文件中的更改

为了引入 Spring Cloud Stream 及其对 RabbitMQ 和 Kafka 的绑定器，我们需要添加两个启动依赖项，分别称为`spring-cloud-starter-stream-rabbit`和`spring-cloud-starter-stream-kafka`。我们还需要一个测试依赖项，`spring-cloud-stream-test-support`，以引入测试支持。下面的代码展示了这一点：

```java
dependencies {
 implementation('org.springframework.cloud:spring-cloud-starter-stream-rabbit')
 implementation('org.springframework.cloud:spring-cloud-starter-stream-kafka')
 testImplementation('org.springframework.cloud:spring-cloud-stream-test-support')
}
```

为了指定我们想要使用的 Spring Cloud 版本，我们首先声明一个版本变量的变量：

```java
ext {
    springCloudVersion = "Greenwich.RELEASE"
}
```

为了完成那个版本的依赖管理设置，我们使用了以下代码：

```java
dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-
        dependencies:${springCloudVersion}"
    }
}
```

要查看完整的源代码，请参阅`product-composite`项目中的`build.gradle`构建文件。

# 在复合服务中发布事件

当复合服务接收到创建或删除产品的请求时，它应将相应的事件发布到核心服务的主题上。为了能够在复合服务中发布事件，我们需要执行以下步骤：

1.  在集成层声明消息源并发布事件。

1.  添加发布事件的配置。

1.  更改我们的测试，以便它们可以测试事件的发布。

复合服务实现类中不需要进行任何更改！

# 在集成层声明消息源并发布事件。

为了能够将事件发布到不同的主题，我们需要在 Java 接口中声明一个`MessageChannel` per topic，并声明我们想要使用它与`EnableBinding` annotation。让我们看看如何做到这一点：

1.  我们在`ProductCompositeIntegration`类中的`MessageSources`接口中声明我们的消息通道，并请求 Spring 在构造函数中注入它的一个实例，如下所示：

```java
@EnableBinding(ProductCompositeIntegration.MessageSources.class)
@Component
public class ProductCompositeIntegration implements ProductService, RecommendationService, ReviewService {

    private MessageSources messageSources;

    public interface MessageSources {

        String OUTPUT_PRODUCTS = "output-products";
        String OUTPUT_RECOMMENDATIONS = "output-recommendations";
        String OUTPUT_REVIEWS = "output-reviews";

        @Output(OUTPUT_PRODUCTS)
        MessageChannel outputProducts();

        @Output(OUTPUT_RECOMMENDATIONS)
        MessageChannel outputRecommendations();

        @Output(OUTPUT_REVIEWS)
        MessageChannel outputReviews();
    }

    public ProductCompositeIntegration(
        MessageSources messageSources,
    ) {
        this.messageSources = messageSources;
    }
```

当我们想要在某个主题上发表一个事件时，我们会使用注入的`messageSources`对象。例如，要为一个产品发送一个删除事件，我们可以使用`outputProducts()`方法获取产品的主题的消息通道，然后使用其`send()`方法发布一个事件。

1.  要创建包含事件的消息，我们可以使用内置的`MessageBuilder`类，如下所示：

```java
@Override
public void deleteProduct(int productId) {       
    messageSources.outputProducts().send(MessageBuilder.
    withPayload(new Event(DELETE, productId, null)).build());
}
```

要查看完整的源代码，请参阅`product-composite`项目中的`se.magnus.microservices.composite.product.services.ProductCompositeIntegration`类。

# 添加发布事件的配置

我们还需要为消息系统设置一个配置，以便能够发布事件。为此，我们需要完成以下步骤：

1.  我们声明 RabbitMQ 是默认的消息系统，默认的内容类型是 JSON：

```java
spring.cloud.stream:
  defaultBinder: rabbit
  default.contentType: application/json
```

1.  接下来，我们将我们的输出通道绑定到特定的主题名称，如下所示：

```java
  bindings:
    output-products:
      destination: products
    output-recommendations:
      destination: recommendations
    output-reviews:
      destination: reviews
```

1.  最后，我们声明了 Kafka 和 RabbitMQ 的连接信息：

```java
spring.cloud.stream.kafka.binder:
  brokers: 127.0.0.1
  defaultBrokerPort: 9092

spring.rabbitmq:
  host: 127.0.0.1
  port: 5672
  username: guest
  password: guest

---
spring.profiles: docker

spring.rabbitmq.host: rabbitmq
spring.cloud.stream.kafka.binder.brokers: kafka
```

在默认的 Spring 配置文件中，我们指定了当不使用 Docker 在`localhost`上运行我们的系统景观时使用的主机名，IP 地址为`127.0.0.1`。在`docker`Spring 配置文件中，我们指定了在 Docker 和 Docker Compose 中运行时将使用的主机名，即`rabbitmq`和`kafka`。

为了查看完整的源代码，请查看`product-composite`项目中的`src/main/resources/application.yml`配置文件。

# 测试代码的变化

测试异步事件驱动的微服务，按其性质来说，是困难的。测试通常需要以某种方式同步异步后台处理，以能够验证其结果。Spring Cloud Stream 提供了支持，通过`TestSupportBinder`，在测试中不使用任何消息系统就可以验证发送了哪些消息！

测试支持包括一个`MessageCollector`助手类，可以用来获取测试期间发送的所有消息。要了解如何做到这一点，请查看以下步骤：

1.  在`MessagingTests`测试类中，我们设置了一个队列，可以用来检查发送到每个主题的消息，如下所示：

```java
  @Autowired
  private MessageCollector collector;

  BlockingQueue<Message<?>> queueProducts = null;
  BlockingQueue<Message<?>> queueRecommendations = null;
  BlockingQueue<Message<?>> queueReviews = null;

  @Before
  public void setUp() {
      queueProducts = getQueue(channels.outputProducts());
      queueRecommendations = 
      getQueue(channels.outputRecommendations());
      queueReviews = getQueue(channels.outputReviews());
  }

  private BlockingQueue<Message<?>> getQueue(MessageChannel 
  messageChannel) {
      return collector.forChannel(messageChannel);
  } 
```

1.  一个实际的测试可以验证队列中的内容，如下面的测试可以验证产品的创建：

```java
@Test
public void createCompositeProduct1() {

    ProductAggregate composite = new ProductAggregate(1, "name", 1, 
    null, null, null);
    postAndVerifyProduct(composite, OK);

    // Assert one expected new product events queued up
    assertEquals(1, queueProducts.size());

    Event<Integer, Product> expectedEvent = new Event(CREATE, 
    composite.getProductId(), new Product(composite.getProductId(), 
    composite.getName(), composite.getWeight(), null));
    assertThat(queueProducts, 
    is(receivesPayloadThat (sameEventExceptCreatedAt 
    (expectedEvent))));

    // Assert none recommendations and review events
    assertEquals(0, queueRecommendations.size());
    assertEquals(0, queueReviews.size());
}
```

`receivesPayloadThat()`方法是 Spring Cloud Stream 中另一个测试支持类`MessageQueueMatcher`的静态方法。这个类包含了一组方法，可以简化队列中消息的验证。

`sameEventExceptCreatedAt()`方法是`IsSameEvent`类中的一个静态方法，它比较`Event`对象，如果所有字段都相等，除了`eventCreatedAt`字段，则认为它们相等。

为了查看完整的源代码，请查看`product-composite`项目中的以下测试类：

+   `se.magnus.microservices.composite.product.MessagingTests`

+   `se.magnus.microservices.composite.product.IsSameEvent`

# 在核心服务中消费事件

为了在核心服务中消费事件，我们需要做以下事情：

1.  声明监听其主题上事件的消息处理器。

1.  更改我们的服务实现，使其正确使用反应式持久层。

1.  添加用于消费事件的配置。

1.  更改我们的测试，使它们可以测试事件的异步处理。

# 声明消息处理器

创建和删除实体的 REST API 已经被每个核心微服务中的消息处理器所取代，该处理器监听每个实体主题上的创建和删除事件。为了能够消费已经发布到主题的消息，我们需要绑定到`SubscribableChannel`，这与我们想要发布消息时绑定到`MessageChannel`类似。由于每个消息处理器只监听一个主题，我们可以使用内置的`Sink`接口来绑定该主题。我们使用`EnableBinding`注解来声明使用`Sink`接口，如下所示：

```java
@EnableBinding(Sink.class)
public class MessageProcessor {
```

为了实际消费和处理消息，我们可以用`StreamListener` 注解标注一个方法，其中我们指定我们要监听哪个通道：

```java
@StreamListener(target = Sink.INPUT)
public void process(Event<Integer, Product> event) {
```

`process()`方法的实现使用一个`switch`语句来调用服务组件中的创建方法以创建事件和删除方法以删除事件。源代码如下所示：

```java
switch (event.getEventType()) {

case CREATE:
    Product product = event.getData();
    LOG.info("Create product with ID: {}", product.getProductId());
    productService.createProduct(product);
    break;

case DELETE:
    int productId = event.getKey();
    LOG.info("Delete recommendations with ProductID: {}", productId);
    productService.deleteProduct(productId);
    break;

default:
    String errorMessage = "Incorrect event type: " + 
    event.getEventType() + ", expected a CREATE or DELETE event";
    LOG.warn(errorMessage);
 throw new EventProcessingException(errorMessage);
}
```

让我们详细解释一下前面的源代码：

1.  `switch`语句期望一个事件类型，该事件类型是一个`CREATE`或`DELETE`事件。

1.  `productService.createProduct()` 方法用于创建事件。

1.  `productService.deleteProduct()` 方法用于删除事件。

1.  如果事件类型既不是`CREATE`也不是`DELETE`事件；将抛出`EventProcessingException`类型的异常。

服务组件像往常一样通过构造函数注入，如下所示：

```java
private final ProductService productService;

@Autowired
public MessageProcessor(ProductService productService) {
    this.productService = productService;
}
```

要查看完整的源代码，请查看以下类：

+   `se.magnus.microservices.core.product.services.MessageProcessor` 在`product`项目中

+   `se.magnus.microservices.core.recommendation.services.MessageProcessor` 在`recommendation`项目中

+   `se.magnus.microservices.core.review.services.MessageProcessor` 在`review`项目中

# 服务实现中的更改

`product`和`recommendation`服务的创建和删除方法的实现已重写，以使用非阻塞的反应式 MongoDB 持久层。例如，创建产品实体的操作如下所示：

```java
public class ProductServiceImpl implements ProductService {

    @Override
    public Product createProduct(Product body) {

        if (body.getProductId() < 1) throw new 
        InvalidInputException("Invalid productId: " + 
        body.getProductId());

        ProductEntity entity = mapper.apiToEntity(body);
        Mono<Product> newEntity = repository.save(entity)
            .log()
            .onErrorMap(
                DuplicateKeyException.class,
                ex -> new InvalidInputException("Duplicate key, Product 
                Id: " + body.getProductId()))
            .map(e -> mapper.entityToApi(e));

        return newEntity.block();
    }
```

`onErrorMap()` 方法用于将`DuplicateKeyException` 持久化异常映射到我们自己的`InvalidInputException` 异常。

由于我们的消息处理程序基于阻塞编程模型，因此在我们将其返回给消息处理程序之前，需要在持久层返回的`Mono`对象上调用`block()`方法。如果我们不调用`block()`方法，如果在服务实现中处理失败，我们将无法触发消息系统中的错误处理；事件将不会重新入队，最终，它将被移动到死信队列中，如预期的那样。

使用阻塞持久层`JPA`的`review`服务，如前所述，不需要更新。

要查看完整的源代码，请查看以下类：

+   `se.magnus.microservices.core.product.services.ProductServiceImpl` 在`product`项目中

+   `se.magnus.microservices.core.recommendation.services.RecommendationServiceImpl` 在`recommendation`项目中

# 添加用于消费事件的配置

我们还需要为消息系统设置配置，以便能够消费事件；这类似于我们对发布者所做的工作。将 RabbitMQ 声明为默认的消息系统，JSON 作为默认内容类型，以及 Kafka 和 RabbitMQ 的连接信息与发布者相同。除了公共部分，消费者配置还指定了消费者组；重试处理和死信队列与之前在*配置 Spring Cloud Stream 以处理消息挑战*部分中描述的一致。

要查看完整的源代码，请查看以下配置文件：

+   `src/main/resources/application.yml` 在`product`项目中

+   `src/main/resources/application.yml` 在`recommendation`项目中

+   `src/main/resources/application.yml` 在`review`项目中

# 测试代码中的更改

由于核心服务现在接收创建和删除实体的事件，测试需要更新，以便它们发送事件而不是像以前那样调用 REST API。在下面的源代码中，我们可以看到如何使用`input`方法通道的`send()`方法发送一个事件：

```java
private void sendCreateProductEvent(int productId) {
    Product product = new Product(productId, "Name " + productId, 
    productId, "SA");
    Event<Integer, Product> event = new Event(CREATE, productId, 
    product);
    input.send(new GenericMessage<>(event));
}

private void sendDeleteProductEvent(int productId) {
    Event<Integer, Product> event = new Event(DELETE, productId, null);
    input.send(new GenericMessage<>(event));
}
```

`input`通道由测试类在运行任何测试之前设置。它基于与消息处理器使用的相同内置`Sink`接口。在下面的源代码中，我们可以看到`input`通道是在`setupDb()`方法中创建的。由于`setupDb()`方法用`@Before`注解，所以它将在执行任何测试之前运行：

```java
@Autowired
private Sink channels;

private AbstractMessageChannel input = null;

@Before
public void setupDb() {
   input = (AbstractMessageChannel) channels.input();
   repository.deleteAll().block();
}
```

这种构造绕过了消息系统，`input`通道上的`send()`方法的调用将由消息处理器同步处理，也就是说，它的`process()`方法就像一个普通的方法调用。这意味着测试代码不需要为事件的异步处理实现任何同步或*等待逻辑*。相反，测试代码可以在调用`sendCreateProductEvent`和`sendDeleteProductEvent`发送助手方法返回后直接应用验证逻辑。

要查看完整的源代码，请查看以下测试类：

+   `se.magnus.microservices.core.product.ProductServiceApplicationTests` 在`product`项目中

+   `se.magnus.microservices.core.recommendation.RecommendationServiceApplicationTests` 在`recommendation`项目中

+   `se.magnus.microservices.core.review.ReviewServiceApplicationTests` 在`review`项目中

# 手动测试反应式微服务架构

现在，我们拥有完全反应式的微服务，无论是在非阻塞同步 REST API 还是在事件驱动的异步服务方面。让我们尝试一下它们！

准备了三种不同的配置，每个都在一个单独的 Docker Compose 文件中：

+   使用不使用分区的 RabbitMQ

+   使用每个主题两个分区的 RabbitMQ

+   使用每个主题两个分区的 Kafka

然而，在测试这三个配置之前，我们首先需要简化对响应式微服务架构的测试。简化后，我们可以继续测试微服务。

因此，需要检查以下两个功能：

+   使用 RabbitMQ 保存事件以供稍后检查。

+   一个可以用来监控景观状态的健康 API。

# 保存事件。

在对事件驱动的异步服务进行一些测试后，可能会有兴趣查看实际发送了哪个事件。当使用 Spring Cloud Stream 和 Kafka 时，事件即使在消费者处理后也会保留在主题中。然而，当使用 Spring Cloud Stream 和 RabbitMQ 时，事件在成功处理后被移除。

为了能够查看每个主题上已经发布的事件，Spring Cloud Stream 被配置为在每个主题上保存发布的事件到一个单独的`auditGroup`消费者组中。对于`products`主题，配置如下所示：

```java
spring.cloud.stream:
  bindings:
    output-products:
      destination: products
      producer:
        required-groups: auditGroup
```

当使用 RabbitMQ 时，这将导致创建额外的队列，以便将事件存储以供稍后检查。

要查看完整的源代码，请参阅`product-composite`项目中的`src/main/resources/application.yml`配置文件。

# 添加健康 API。

测试使用同步 API 和异步消息传递的微服务系统架构是具有挑战性的。例如，我们如何知道一个新启动的微服务架构（及其数据库和消息系统）是否准备好处理请求和消息？

为了更容易地知道所有微服务是否准备好处理请求和消息，我们在所有微服务中添加了一个健康 API。它们基于 Spring Boot 模块中名为 Actuator 的支持健康端点的支持。默认情况下，基于 Actuator 的健康端点回答`UP`（并给出 200 作为 HTTP 返回状态）如果微服务本身以及 Spring Boot 知道的所有的依赖项都可用，例如，对数据库和消息系统的依赖；否则，健康端点回答`DOWN`（并返回 500 作为 HTTP 返回状态）。

我们还可以扩展`health`端点以覆盖 Spring Boot 不知道的依赖项。我们将使用这个特性来扩展产品组合的`health`端点，这也将包括三个核心服务的健康状况。这意味着产品组合的`health`端点只会在自身和三个核心微服务都健康的情况下回答`UP`。这可以手动或自动地由`test-em-all.bash`脚本来使用，以找出所有微服务及其依赖项是否都已启动并运行。

在`ProductCompositeIntegration`集成类中，我们添加了用于检查三个核心微服务健康状况的帮助方法，如下所示：

```java
public Mono<Health> getProductHealth() {
    return getHealth(productServiceUrl);
}

public Mono<Health> getRecommendationHealth() {
    return getHealth(recommendationServiceUrl);
}

public Mono<Health> getReviewHealth() {
    return getHealth(reviewServiceUrl);
}

private Mono<Health> getHealth(String url) {
    url += "/actuator/health";
    LOG.debug("Will call the Health API on URL: {}", url);
    return webClient.get().uri(url).retrieve().bodyToMono(String.class)
        .map(s -> new Health.Builder().up().build())
        .onErrorResume(ex -> Mono.just(new 
         Health.Builder().down(ex).build()))
        .log();
}
```

这段代码与我们之前用于调用核心服务以读取 API 的代码相似。

有关完整源代码，请参阅 `product-composite` 项目中的 `se.magnus.microservices.composite.product.services.ProductCompositeIntegration` 类。

在主 `ProductCompositeServiceApplication` 应用程序类中，我们使用这些辅助方法注册三个健康检查，每个核心微服务一个：

```java
@Autowired
HealthAggregator healthAggregator;

@Autowired
ProductCompositeIntegration integration;

@Bean
ReactiveHealthIndicator coreServices() {

    ReactiveHealthIndicatorRegistry registry = new 
    DefaultReactiveHealthIndicatorRegistry(new LinkedHashMap<>());

    registry.register("product", () -> integration.getProductHealth());
    registry.register("recommendation", () -> 
    integration.getRecommendationHealth());
    registry.register("review", () -> integration.getReviewHealth());

    return new CompositeReactiveHealthIndicator(healthAggregator, 
    registry);
}
```

有关完整源代码，请参阅 `product-composite` 项目中的 `se.magnus.microservices.composite.product.ProductCompositeServiceApplication` 类。

最后，在所有四个微服务的 `application.yml` 文件中，我们配置了 Spring Boot Actuator，使其执行以下操作：

+   显示有关健康状态的详细信息，这不仅包括 `UP` 或 `DOWN`，还包括有关其依赖项的信息：

+   通过 HTTP 暴露其所有端点：

这两个设置的配置如下所示：

```java
management.endpoint.health.show-details: "ALWAYS"
management.endpoints.web.exposure.include: "*"
```

有关完整源代码的示例，请参阅 `product-composite` 项目中的 `src/main/resources/application.yml` 配置文件。

**警告**：这些配置设置在开发过程中很好，但在生产系统中暴露太多信息在 actuator 端点上可能是一个安全问题。因此，计划最小化在生产中 actuator 端点暴露的信息！

有关由 Spring Boot Actuator 暴露的端点的详细信息，请参阅 [`docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html`](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-endpoints.html)：

+   尝试一下（当你使用 Docker Compose 启动所有微服务时，如下一节所述）：

```java
curl localhost:8080/actuator/health -s | jq .
```

+   这将导致以下响应：

![](img/2ae85fa6-c021-4370-9850-fb6daec78d0c.png)

在前面的输出中，我们可以看到复合服务报告它是健康的，即它的状态是 `UP`。在响应的末尾，我们可以看到三个核心微服务也被报告为健康。

有了健康 API，我们就准备好测试我们的反应式微服务了。

# 不使用分区来使用 RabbitMQ：

在本节中，我们将测试与 RabbitMQ 一起使用的反应式微服务，但不用分区。

在此配置中使用默认的 `docker-compose.yml` Docker Compose 文件。已对文件应用了以下更改：

+   **RabbitMQ** 已经被添加，如图所示：

```java
rabbitmq:
  image: rabbitmq:3.7.8-management
  mem_limit: 350m
  ports:
    - 5672:5672
    - 15672:15672
  healthcheck:
    test: ["CMD", "rabbitmqctl", "status"]
    interval: 10s
    timeout: 5s
    retries: 10
```

+   微服务现在对 RabbitMQ 服务有了依赖声明。这意味着 Docker 不会启动微服务容器，直到 RabbitMQ 服务被报告为健康：

```java
depends_on:
  rabbitmq:
    condition: service_healthy
```

要运行我们的测试，请执行以下步骤：

1.  使用以下命令构建并启动系统架构：

```java
cd $BOOK_HOME/Chapter07
./gradlew build && docker-compose build && docker-compose up -d
```

1.  现在，我们必须等待微服务架构运行起来。

    尝试运行以下命令几次：

```java
curl -s localhost:8080/actuator/health | jq -r .status
```

当它返回 `UP` 时，我们就准备好运行我们的测试了！

1.  首先，使用以下命令创建一个复合产品：

```java
body='{"productId":1,"name":"product name C","weight":300, "recommendations":[
 {"recommendationId":1,"author":"author 1","rate":1,"content":"content 1"},
 {"recommendationId":2,"author":"author 2","rate":2,"content":"content 2"},
 {"recommendationId":3,"author":"author 3","rate":3,"content":"content 3"}
], "reviews":[
 {"reviewId":1,"author":"author 1","subject":"subject 1","content":"content 1"},
 {"reviewId":2,"author":"author 2","subject":"subject 2","content":"content 2"},
 {"reviewId":3,"author":"author 3","subject":"subject 3","content":"content 3"}
]}'

curl -X POST localhost:8080/product-composite -H "Content-Type: application/json" --data "$body"
```

当 Spring Cloud Stream 与 RabbitMQ 一起使用时，它将根据我们的配置为每个主题创建一个 RabbitMQ 交换和一个队列集。

看看 Spring Cloud Stream 为我们创建了哪些队列吧！

1.  在网页浏览器中打开以下 URL：`http://localhost:15672/#/queues`。你应该看到以下队列：

![](img/00372e65-3efd-4733-8c3a-773b383cde7e.png)

对于每个主题，我们可以看到一个 auditGroup 队列，一个由相应核心微服务使用的消费者组队列，以及一个死信队列。我们还可以看到 auditGroup 队列中包含消息，正如我们所期望的那样！

1.  点击`products.auditGroup`队列，向下滚动到 Get Message(s)，展开它，然后点击名为 Get Message(s)的按钮查看队列中的消息：

![](img/1ce813e3-39eb-4b9a-a69e-b2d4f0a6be85.png)

1.  接下来，尝试使用以下代码获取产品组合：

```java
curl localhost:8080/product-composite/1 | jq 
```

1.  最后，像这样删除它：

```java
curl -X DELETE localhost:8080/product-composite/1
```

试图再次获取已删除的产品应该会导致一个`404 - "NotFound"`的响应！

如果你再次查看 RabbitMQ 审计队列，你应该能够找到包含删除事件的新消息。

1.  通过以下命令结束测试，关闭微服务架构：

```java
docker-compose down
```

这样就完成了我们使用没有分区的 RabbitMQ 的测试。现在，让我们继续测试带有分区的 RabbitMQ。

# 使用每个主题两个分区的 RabbitMQ

现在，让我们尝试一下 Spring Cloud Stream 中的分区支持！

我们为使用每个主题两个分区的 RabbitMQ 准备了一个单独的 Docker Compose 文件：`docker-compose-partitions.yml`。它还将为每个核心微服务启动两个实例，每个分区一个。例如，第二个`product`实例的配置如下：

```java
product-p1:
  build: microservices/product-service
  mem_limit: 350m
  environment:
    - SPRING_PROFILES_ACTIVE=docker
    - SPRING_CLOUD_STREAM_BINDINGS_INPUT_CONSUMER_PARTITIONED=true
    - SPRING_CLOUD_STREAM_BINDINGS_INPUT_CONSUMER_INSTANCECOUNT=2
    - SPRING_CLOUD_STREAM_BINDINGS_INPUT_CONSUMER_INSTANCEINDEX=1
  depends_on:
    mongodb:
      condition: service_healthy
    rabbitmq:
      condition: service_healthy
```

以下是前述源代码的解释：

+   我们使用与第一个`product`实例相同的源代码和 Dockerfile，但它们进行了不同的配置。

+   具体来说，我们将两个`product`实例分配到不同的分区，使用的是我们本章前面描述的`instance-index`属性。

+   当使用系统环境变量来指定 Spring 属性时，我们必须使用大写字母格式，其中点被下划线替换。

+   这个`product`实例只处理异步事件；它不会响应 API 调用。由于它的名称不同，`product-p1`（也用作其 DNS 名称），所以它不会响应以`http://product:8080`开头的 URL 调用。

使用以下命令启动`microservice landscape`：

```java
export COMPOSE_FILE=docker-compose-partitions.yml
docker-compose build && docker-compose up -d
```

重复前一部分的测试，但也要创建一个产品 ID 设置为`2`的产品。如果你查看 Spring Cloud Stream 设置的队列，你会看到每个分区有一个队列，并且产品审计队列现在每个都包含一个消息，即产品 ID `1`的事件放在一个分区的，而产品 ID `2`的事件放在另一个分区的。如果你回到浏览器中的`http://localhost:15672/#/queues`，你应该会看到类似以下的内容：

![](img/4d333160-75e8-4a56-87f6-aeee8765402e.png)

要结束使用分区的 RabbitMQ 测试，请使用以下命令关闭微服务架构：

```java
docker-compose down
unset COMPOSE_FILE
```

我们现在完成了使用 RabbitMQ 的测试，包括有分区和没有分区的情况。我们将尝试的最后一种测试配置是同时测试微服务与 Kafka。

# 使用 Netflix Eureka 作为发现服务

发现服务可能是使一组合作的微服务生产就绪所需的最重要的支持功能。正如我们在第一章、*微服务介绍*中的*服务发现*部分已经描述的，服务发现服务可以用来跟踪现有的微服务和它们实例。Spring Cloud 支持的第一个发现服务是*Netflix Eureka*。

我们将在第九章、*使用 Netflix Eureka 和 Ribbon 添加服务发现*中使用这个，以及负载均衡器和新的 Spring Cloud 负载均衡器。

我们将看到在使用 Spring Cloud 时注册微服务有多么简单，以及当客户端发送 HTTP 请求（例如对注册在 Netflix Eureka 中的一个实例的 RESTful API 的调用）时会发生什么。我们还将了解如何扩展微服务的实例数量，以及如何将请求负载均衡到微服务的可用实例上（基于，默认情况下，轮询调度）。

以下屏幕快照展示了 Eureka 的网页用户界面，我们可以看到我们已经注册了哪些微服务：

![](img/9d6bf02d-f466-40cf-9bf1-140cbf7d1766.png)

评论服务有三个实例可用，而其他两个服务只有一个实例。

随着 Netflix Eureka 的引入，让我们介绍一下如何使用 Spring Cloud Gateway 作为边缘服务器。

# 使用每个主题两个分区的 Kafka

现在，我们将尝试 Spring Cloud Stream 的一个非常酷的功能：将消息系统从 RabbitMQ 更改为 Apache Kafka！

这可以通过将`spring.cloud.stream.defaultBinder`属性的值从`rabbit`更改为`kafka`来简单实现。这由`docker-compose-kafka.yml`Docker Compose 文件处理，该文件也将 RabbitMQ 替换为 Kafka 和 Zookeeper。Kafka 和 Zookeeper 的配置如下所示：

```java
kafka:
  image: wurstmeister/kafka:2.12-2.1.0
  mem_limit: 350m
  ports:
    - "9092:9092"
  environment:
    - KAFKA_ADVERTISED_HOST_NAME=kafka
    - KAFKA_ADVERTISED_PORT=9092
    - KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181
  depends_on:
    - zookeeper

zookeeper:
  image: wurstmeister/zookeeper:3.4.6
  mem_limit: 350m
  ports:
    - "2181:2181"
  environment:
    - KAFKA_ADVERTISED_HOST_NAME=zookeeper
```

Kafka 还配置为每个主题使用两个分区，像以前一样，我们为每个核心微服务启动两个实例，每个分区一个。详情请查看 Docker Compose 文件`docker-compose-kafka.yml`！

使用以下命令启动微服务架构：

```java
export COMPOSE_FILE=docker-compose-kafka.yml
docker-compose build && docker-compose up -d
```

重复上一节的测试，例如，创建两个产品，一个产品 ID 设置为`1`，另一个产品 ID 设置为`2`。

不幸的是，Kafka 没有附带任何可以用来检查主题、分区以及其中的消息的图形工具。相反，我们可以在 Kafka Docker 容器中运行 CLI 命令。

要查看主题列表，请运行以下命令：

```java
docker-compose exec kafka /opt/kafka/bin/kafka-topics.sh --zookeeper zookeeper --list
```

预期输出如下所示：

![](img/824bee38-2547-4e15-9c1f-d4a7550eb6a0.png)

以下是对前面源代码的解释：

+   前缀为`error`的主题是对应于死信队列的主题。

+   在 RabbitMQ 的情况下，你找不到`auditGroup`；相反，所有消息都可供任何消费者处理。

要查看特定主题的分区，例如`products`主题，请运行以下命令：

```java
docker-compose exec kafka /opt/kafka/bin/kafka-topics.sh --describe --zookeeper zookeeper --topic products
```

预期输出如下所示：

![](img/0ba3ea90-0eb6-49b9-94f2-71b377357e60.png)

要查看特定主题的所有消息，例如`products`主题，请运行以下命令：

```java
docker-compose exec kafka /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic products --from-beginning --timeout-ms 1000
```

预期输出如下所示：

![](img/7570391d-189e-4c89-b544-547f054cfc71.png)

要查看特定分区的所有消息，例如`products`主题中的分区`1`，请运行以下命令：

```java
docker-compose exec kafka /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic products --from-beginning --timeout-ms 1000 --partition 1
```

预期输出如下所示：

![](img/9dd71100-be1f-4d1c-b3aa-06094d47fe58.png)

输出将以超时异常结束，因为我们通过指定`1000`毫秒的命令超时来停止命令。

使用以下命令关闭微服务架构：

```java
docker-compose down
unset COMPOSE_FILE
```

现在，我们已经了解到如何使用 Spring Cloud Stream 将消息代理从 RabbitMQ 切换到 Kafka，而无需更改源代码。它只需要在 Docker Compose 文件中进行一些更改。

# 反应式微服务架构的自动化测试

为了能够自动运行反应式微服务架构的测试，而不是手动运行，自动`test-em-all.bash`测试脚本已经得到增强。最重要的变化如下：

+   脚本使用新的`health`端点来了解微服务架构何时运行正常，如下所示：

```java
waitForService curl http://$HOST:$PORT/actuator/health
```

+   脚本有一个新的`waitForMessageProcessing()`函数，它在测试数据设置后调用。它的目的是简单地等待异步创建服务完成测试数据的创建。

要使用测试脚本自动运行与 RabbitMQ 和 Kafka 相关的测试，请执行以下步骤：

1.  使用默认的 Docker Compose 文件运行测试，即不使用 RabbitMQ 分区，使用以下命令：

```java
unset COMPOSE_FILE
./test-em-all.bash start stop
```

1.  使用以下命令运行带有两个分区的 RabbitMQ 测试：

```java
export COMPOSE_FILE=docker-compose-partitions.yml 
./test-em-all.bash start stop
unset COMPOSE_FILE
```

1.  最后，使用以下命令运行带有 Kafka 和每个主题两个分区的测试：

```java
export COMPOSE_FILE=docker-compose-kafka.yml 
./test-em-all.bash start stop
unset COMPOSE_FILE
```

在本节中，我们学习了如何使用`test-em-all.bash`测试脚本自动运行使用 RabbitMQ 或 Kafka 作为消息代理配置的反应式微服务架构的测试。

# 总结

在本章中，我们看到了我们如何可以开发反应式微服务！

使用 Spring WebFlux 和 Spring WebClient，我们可以开发非阻塞同步 API，这些 API 可以处理传入的 HTTP 请求并发送非阻塞线程的出站 HTTP 请求。利用 Spring Data 对 MongoDB 的反应式支持，我们还可以以非阻塞方式访问 MongoDB 数据库，即在等待数据库响应时不会阻塞任何线程。Spring WebFlux、Spring WebClient 和 Spring Data 依赖于 Spring Reactor 提供它们的反应式和非阻塞特性。当我们必须使用阻塞代码时，例如在使用 Spring Data for JPA 时，我们可以通过在专用线程池中安排处理来封装阻塞代码的处理。

我们还看到了 Spring Data Stream 如何用于开发既适用于 RabbitMQ 又适用于 Kafka 作为消息系统的基于事件的异步服务，而无需更改代码。通过进行一些配置，我们可以使用 Spring Cloud Stream 中的特性，如消费者组、重试、死信队列和分区，以处理异步消息的各种挑战。

我们还学习了如何手动和自动测试由反应式微服务组成的系统架构。

这是关于如何在 Spring Boot 和 Spring Framework 中使用基本特性的最后一章。

接下来将介绍 Spring Cloud 以及如何使用它来使我们的服务达到生产级、可扩展、健壮、可配置、安全和有弹性！

# 问题

1.  为什么知道如何开发反应式微服务很重要？

1.  您如何选择非阻塞同步 API 和事件/消息驱动的异步服务？

1.  消息与事件有什么不同？

1.  列出一些消息驱动异步服务的挑战。我们如何处理它们？

1.  为什么以下测试失败？

```java
    @Test
    public void TestFlux() {

        List<Integer> list = new ArrayList<>();

        Flux.just(1, 2, 3, 4)
            .filter(n -> n % 2 == 0)
            .map(n -> n * 2)
            .log();

        assertThat(list).containsExactly(4, 8);
```

1.  使用 JUnit 编写反应式代码的测试时面临哪些挑战，我们该如何应对？
