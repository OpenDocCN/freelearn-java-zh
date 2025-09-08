# 评估

# 第一章：开始使用反应式流

1.  反应式宣言的原则是什么？

反应式宣言定义了以下原则：

+   **消息驱动**：所有应用程序组件都应该是松散耦合的，并使用消息进行通信

+   **响应性**：应用程序必须及时响应用户输入

+   **弹性**：应用程序必须将故障隔离到单个组件

+   **可伸缩性**：应用程序必须对工作负载的变化做出反应

1.  反应式扩展是什么？

反应式扩展是命令式语言中的库，使我们能够编写异步、事件驱动的反应式应用程序。这些库使我们能够将异步事件表达为一系列可观察对象。这使得我们能够构建可以接收和处理这些异步事件的组件。另一方面，也存在事件生产者，它们推送这些事件。

1.  反应式流规范旨在满足什么需求？

反应式流是一个规范，它确定了构建大量无界数据异步处理所需的最小接口集。它是一个针对 JVM 和 JavaScript 运行时的规范。反应式流规范的主要目标是标准化应用程序异步边界之间的流数据交换。

1.  反应式流基于哪些原则？

反应式流基于以下两个原则：

+   **异步执行**：这是在不等待先前执行的任务完成的情况下执行任务的能力。执行模型解耦任务，以便每个任务都可以同时执行，利用可用的硬件。

+   **背压**：订阅者可以控制其队列中的事件以避免任何溢出。如果还有额外容量，它还可以请求更多事件。

1.  Reactor 框架的主要特点是什么？

+   **无限数据流**：这指的是 Reactor 生成无限数据序列的能力。

+   **推拉模型**：在 Reactor 中，生产者可以推送事件。另一方面，如果消费者在处理方面较慢，它可以在自己的速率下拉取事件。

+   **无并发性**：Reactor 不强制执行任何并发模型。它允许开发者选择最适合的。

+   **操作词汇**：Reactor 提供了一系列操作符。这些操作符允许我们选择、过滤、转换和组合流。

# 第二章：Reactor 中的发布者和订阅者 API

1.  我们如何验证反应式流的发布者和订阅者实现？

为了验证发布者，反应式流 API 发布了一个名为`reactive-streams-tck`的测试兼容性套件。可以使用`PublisherVerifier`接口验证反应式发布者。同样，可以使用`SubscriberBlackboxVerification<T>`抽象类验证订阅者。

1.  反应式流发布者-订阅者模型与 JMS API 有何不同？

在 JMS 中，生产者负责在队列或主题上生成无界事件，而消费者积极消费事件。生产者和消费者独立工作，以自己的速率。订阅管理的任务由 JMS 代理负责。JMS 中没有背压的概念。此外，它缺乏事件建模，如订阅、错误或完成。

1.  反应式流发布者-订阅者模型与 Observer API 有何不同？

Observable API 负责确定变化并将其发布给所有感兴趣方。API 是关于实体状态变化的。这不是我们使用`Publisher`和`Subscriber`接口所建模的内容。`Publisher`接口负责生成无界事件。另一方面，`Subscriber`列出了所有类型的事件，如数据、错误和完成。

1.  Flux 和 Mono 之间有什么区别？

`Flux`是一个通用反应式发布者。它表示一个异步事件流，其中包含零个或多个值。另一方面，Mono 只能生成最多一个事件。

1.  `SynchronousSink`和`FluxSink`之间有什么区别？

`SynchronousSink`一次只能生成一个事件。它是同步的。订阅者必须在生成下一个事件之前消费事件。另一方面，`FluxSink`可以异步生成多个事件。此外，它不考虑订阅取消或背压。这意味着即使订阅者取消了其订阅，`create` API 也会继续生成事件。

1.  Reactor 提供了哪些不同的生命周期钩子？

+   `doOnSubscribe`: 用于订阅事件

+   `doOnRequest`: 用于请求事件

+   `doOnNext`: 用于下一个事件

+   `doOnCancel`: 用于订阅取消事件

+   `doOnError`: 用于错误事件

+   `doOnCompletion`: 用于完成事件

+   `doOnTerminate`: 用于由于错误、完成或取消而终止

+   `doFinally`: 用于流终止后的清理

+   `doOnEach`: 用于所有事件

# 第三章：数据和流处理

1.  哪个算子用于从流中选择数据元素？

+   `filter`

+   `filterWhen`

+   `take`

+   `takeLast`

+   `last`

+   `distinct`

+   `single`

+   `elementAT`

1.  哪个算子用于从流中拒绝数据元素？

+   `filter`

+   `filterWhen`

+   `skip`

+   `skipLast`

+   `SkipUntil`

+   `ignoreElements`

1.  Reactor 提供了哪些算子用于数据转换？这些算子之间有何不同？

+   `map`: 这用于一对一转换

+   `flatMap`: 这用于一对一转换

1.  我们如何使用 Reactor 算子进行数据聚合？

+   `collectList`

+   `collectMap`

+   `` `collectMultiMap` ``

1.  Reactor 提供了哪些条件算子？

+   `all`: 表示`AND`运算符

+   `any`: 表示`OR`运算符

# 第四章：处理器

1.  `DirectProcessor`有哪些局限性？

`DirectProcessor`不提供任何背压处理。

1.  `UnicastProcessor` 的局限性是什么？

`UnicastProcessor` 只能与单个订阅者一起工作。

1.  `EmitterProcessor` 的功能有哪些？

`EmitterProcessor` 是一个可以与多个订阅者一起使用的处理器。多个订阅者可以根据它们各自的消费速率请求处理器获取下一个值事件

1.  `ReplayProcessor` 的功能有哪些？

`ReplayProcessor` 是一个特殊用途的处理器，能够缓存并回放事件给其订阅者。

1.  `TopicProcessor` 的功能有哪些？

`TopicProcessor` 是一个能够使用事件循环架构与多个订阅者一起工作的处理器。处理器以异步方式从发布者向附加的订阅者传递事件，并通过使用 RingBuffer 数据结构为每个订阅者尊重背压。

1.  `WorkQueueProcessor` 的功能有哪些？

`WorkQueueProcessor` 可以连接到多个订阅者。它不会将所有事件发送给每个订阅者。每个订阅者的需求被添加到队列中，发布者的事件被发送给任何订阅者。

1.  热发布者和冷发布者之间有什么区别？

冷发布者为每个订阅者有一个单独的订阅状态。它们将所有数据发布给每个订阅者，而不考虑订阅时间。另一方面，热发布者将公共数据发布给所有订阅者。因此，新订阅者只能获得当前事件，不会向他们传递旧事件。

# 第五章：SpringWebFlux 用于微服务

1.  我们如何配置 `SpringWebFlux` 项目？

`SpringWebFlux` 可以以两种方式配置：

+   **使用注解**：`SpringWebFlux` 支持 `SpringWebMVC` 注解。这是配置 `SpringWebFlux` 的最简单方法。

+   **使用功能端点**：此模型允许我们构建 Java 8 函数作为网络端点。应用程序可以配置为一系列路由、处理程序和过滤器。

1.  `SpringWebFlux` 支持哪些 `MethodParameter` 注解？

+   `@PathVariable`：此注解用于访问 URI 模板变量的值

+   `@RequestParam`：此注解用于确定作为查询参数传递的值

+   `@RequestHeader`：此注解用于确定请求头中传递的值

+   `@RequestBody`：此注解用于确定请求体中传递的值

+   `@CookieValue`：此注解用于确定作为请求一部分的 HTTP cookie 值

+   `@ModelAttribute`：此注解用于确定请求模型中的属性或在没有时实例化一个

+   `@SessionAttribute`：此注解用于确定现有的会话属性

+   `@RequestAttribute`：此注解用于确定由先前过滤器执行创建的现有请求属性

1.  `ExceptionHandler` 的用途是什么？

`SpringWebFlux` 通过创建带有 `@ExceptionHandler` 注解的方法来支持异常处理。

1.  `HandlerFunction` 的用途是什么？

`SpringWebFlux` 处理函数负责服务给定的请求。它以 `ServerRequest` 类的形式接收请求，并以 `ServerResponse` 形式生成响应。

1.  `RouterFunction` 的用途是什么？

`SpringWebFlux` 路由函数负责将传入的请求路由到正确的处理函数。

1.  `HandlerFilter` 的用途是什么？

`HandlerFilter` 与 Servlet 过滤器类似。它在 `HandlerFunction` 处理请求之前执行。可能存在链式过滤器，它们在请求被服务之前执行。

# 第六章：动态渲染

1.  `SpringWebFlux` 框架如何解析视图？

框架使用端点调用返回的 `HandlerResult` 调用 `ViewResolutionResultHandler`。然后 `ViewResolutionResultHandler` 通过验证以下返回值来确定正确的视图：

+   **字符串**：如果返回值是一个字符串，那么框架将使用配置的 `ViewResolvers` 构建视图

+   **空**：如果没有返回任何内容，它将尝试构建默认视图

+   **映射**：框架查找默认视图，但它还将返回到请求模型中的键值对添加进去

`ViewResolutionResultHandler` 查找请求中传递的内容类型。为了确定应该使用哪个视图，它将传递给 `ViewResolver` 的内容类型与支持的内容类型进行比较。然后选择第一个支持请求内容类型的 `ViewResolver`。

1.  需要配置哪些组件才能使用 Thymeleaf 模板引擎？

+   将 `spring-boot-starter-thymeleaf` 添加到项目中

+   创建一个 `ThymeleafReactiveViewResolver` 实例

+   将解析器添加到 `ViewResolverRegistry` 中，该解析器在 `configureViewResolvers` 方法中可用

1.  哪个 API 用于配置 SpringWebFlux 中的静态资源？

+   `addResourceHandler` 方法接受一个 URL 模式并将其配置为静态位置

+   `addResourceLocations` 方法配置了一个位置，从该位置需要提供静态内容

1.  `WebClient` 的好处是什么？

`WebClient` 是一个非阻塞、异步 HTTP 客户端，用于发送请求。它可以配置 Java 8 lambda 表达式来处理数据。

1.  WebClient 的检索和交换 API 之间有什么区别？

+   `检索`：这可以将请求体解码为 Flux 或 Mono

+   `交换`：`Exchange` 方法提供了完整的信息，可以将其转换回目标类型

# 第七章：流程控制和背压

1.  为什么我们需要 `groupBy` 操作符？

`groupBy()` 操作符将 `Flux<T>` 转换为批次。该操作符将一个键与 `Flux<T>` 的每个元素关联。然后它将具有相同键的元素分组。然后操作符发出这些组。

1.  `groupBy` 和 `buffer` 操作符之间有什么区别？

`groupBy` 操作符根据配置的键对事件流进行分组，但 `buffer` 操作符将流分割成指定大小的块。因此，`buffer` 操作符保持事件的原始顺序。

1.  我们如何在 Reactor 中节流事件？

`sample()` 操作符允许我们实现节流。

1.  `Overflow.Ignore` 和 `Overflow.Latest` 策略之间的区别是什么？

`Overflow.Ignore` 忽略订阅者背压的限制，并继续向订阅者发送下一个事件。`Overflow.Latest` 保持缓冲区中提出的最新事件。当下一次请求提出时，订阅者将只获得最新产生的事件。

1.  哪些操作符可用于更改生产者的背压策略？

+   `onBackpressureDrop()`

+   `onBackpressureLatest()`

+   `onBackpressureError()`

+   `onBackpressureBuffer()`

# 第八章：处理错误

1.  在 Reactor 中是如何处理错误的？

当发布者或订阅者抛出异常时，会出现错误。Reactor 会在拦截异常后构建一个 `Error` 事件，并将其发送给订阅者。订阅者必须实现 `ErrorCallbackHandler` 来处理错误。

1.  哪些操作符允许我们配置错误处理？

+   `onErrorReturn`

+   `onErrorResume`

+   `onErrorMap`

1.  `onErrorResume` 和 `onErrorReturn` 之间的区别是什么？

`OnErrorReturn` 操作符在发生错误时提供后备值。另一方面，`OnErrorResume` 操作符提供后备值流而不是单个后备值。

1.  我们如何为反应式流生成及时响应？

`timeout()` 操作符可以配置时间间隔。当操作符首次发现延迟超过配置时间时，将引发错误。操作符还有一个后备 Flux。当超时到期时，将返回后备值。

1.  `retry` 操作符是如何表现的？

`retry` 操作符允许我们在发现错误时重新订阅已发布的流。`retry` 操作只能执行固定次数。重新订阅的事件由订阅者作为后续事件处理。如果流正常完成，则不会进行后续重试。

# 第九章：执行控制

1.  Reactor 中可用的不同类型的调度器有哪些？

+   `Schedulers.immediate`: 这将在当前线程上调度

+   `Schedulers.single`: 这将在单个线程上调度

+   `Schedulers.parallel`: 这将在线程池上调度

+   `Schedulers.elastic`: 这将在线程池上调度

+   `Schedulers.fromExecutor`: 这将在配置的执行器服务上调度

1.  应该使用哪个调度器来处理阻塞操作？

`Schedulers.elastic` 在线程池上调度。

1.  应该使用哪个调度器来处理计算密集型操作？

+   `Schedulers.single`: 这将在单个线程上调度

+   `Schedulers.parallel`: 这将在线程池上调度

1.  `PublishOn` 和 `SubscriberOn` 与彼此有何不同？

`subscribeOn` 操作符会拦截执行链中的发布者事件并将它们发送到不同的调度器以完成整个链。需要注意的是，该操作符会改变整个链的执行上下文，与仅改变下游链执行的 `publishOn` 操作符不同。

1.  `ParallelFlux` 的局限性是什么？

`ParallelFlux` 不提供 `doFinally` 生命周期钩子。它可以使用 `sequential` 操作符转换回 `Flux`，然后可以使用 `doFinally` 钩子进行配置。

1.  哪些操作符可用于生成 `ConnectedFlux`？

+   `replay`

+   `publish`

# 第十章：测试和调试

1.  在 Reactor 中，哪个测试实用类可用于验证流上调用的操作？

Reactor 提供了 `StepVerifier` 组件来独立验证所需的操作。

1.  `PublisherProbe` 和 `TestPublisher` 之间的区别是什么？

`PublisherProbe` 实用类可以用于对现有的发布者进行监控。该监控器会跟踪发布者发布的信号，这些信号可以在测试结束时进行验证。另一方面，`TestPublisher` 能够生成 `Publisher` 模拟器，这可以用于对 Reactor 操作符进行单元测试。

1.  应如何配置虚拟时钟以验证时间限制操作？

在执行任何基于时间的操作之前，必须注入虚拟时钟。

1.  `onOperatorDebug` 钩子和 `checkpoint` 操作符之间的区别是什么？

`onOperatorDebug` 钩子会对所有反应式管道进行全局更改。另一方面，`checkpoint` 操作符会对应用到的流进行特定更改。

1.  我们如何开启流处理日志记录？

可以使用 `log` 操作符来开启日志记录。
