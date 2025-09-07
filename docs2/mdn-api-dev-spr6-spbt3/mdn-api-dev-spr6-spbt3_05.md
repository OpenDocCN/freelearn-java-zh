# 5

# 异步 API 设计

到目前为止，我们已经基于命令式模型开发了 RESTful Web 服务，其中调用是同步的。如果你想要使代码异步和非阻塞，我们将在本章中介绍这一点。你将学习本章中的异步 API 设计，其中调用是异步和非阻塞的。我们将使用基于 Project Reactor ([`projectreactor.io`](https://projectreactor.io))的 Spring **WebFlux**来开发这些 API。Reactor 是一个用于在**Java 虚拟机**（**JVM**）上构建非阻塞应用程序的库。

首先，我们将介绍响应式编程的基础知识，然后我们将通过比较现有的（命令式）编程方式和响应式编程方式，将现有的电子商务 REST API（我们在*第四章*，*为 API 编写业务逻辑）迁移到异步（响应式）API，以简化事情。代码将使用支持响应式编程的 R2DBC 进行数据库持久化。

本章我们将讨论以下主题：

+   理解响应式流

+   探索 Spring WebFlux

+   理解`DispatcherHandler`

+   控制器

+   功能端点

+   为我们的电子商务应用实现响应式 API

到本章结束时，你将学会如何开发和实现响应式 API，并探索异步 API。你还将能够实现响应式控制器和功能端点，并利用 R2DBC 进行数据库持久化。

# 技术要求

本章的代码可在[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter05`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter05)找到。

# 理解响应式流

正常的 Java 代码通过使用线程池来实现异步性。你的 Web 服务器使用线程池来处理请求——它为每个传入的请求分配一个线程。应用程序也使用线程池来处理数据库连接。每个数据库调用都使用一个单独的线程并等待结果。因此，每个 Web 请求和数据库调用都使用自己的线程。然而，这伴随着等待，因此这些是阻塞调用。线程等待并利用资源，直到从数据库收到响应或写入响应对象。当你进行扩展时，这是一个限制，因为你只能使用 JVM 可用的资源。通过使用带有服务其他实例的负载均衡器来克服这种限制，这是一种水平扩展的类型。

在过去十年中，客户端-服务器架构有所增长。大量的物联网设备、具有原生应用的智能手机、一流的网络应用和传统的网络应用纷纷涌现。应用不仅拥有第三方服务，还有各种数据来源，这导致了更高规模的应用。除此之外，基于微服务的架构增加了服务之间的通信。你需要大量的资源来满足这种更高的网络通信需求。这使得扩展成为必要。线程很昂贵，且不是无限的。你不想阻塞它们以实现有效利用。例如，假设你的代码正在调用数据库以获取数据。在这种情况下，调用会等待直到你在阻塞调用中收到响应。然而，非阻塞调用不会阻塞任何东西。它仅在从依赖代码（在这种情况下是数据库）收到响应时才响应。在这段时间内，系统可以服务其他调用。这就是异步性发挥作用的地方。在异步调用中，一旦调用完成，线程就会变得空闲，并使用回调实用工具（在 JavaScript 中很常见）。当数据在源处可用时，它会推送数据。Reactor 项目基于**响应式流**。响应式流使用**发布者-订阅者模型**，其中数据源，即发布者，将数据推送到订阅者。

你可能知道，另一方面，Node.js 使用单个线程来利用大多数资源。它基于异步非阻塞设计，称为**事件循环**。

响应式 API 也基于事件循环设计，并使用推送式通知。如果你仔细观察，响应式流还支持 Java 流操作，如`map`、`flatMap`和`filter`。内部，响应式流使用推送式，而 Java 流则根据拉模型工作；也就是说，项目是从源（如 Java 集合）中拉取的。在响应式编程中，源（发布者）推送数据。

在响应式流中，数据流是异步和非阻塞的，并支持背压。（有关背压的解释，请参阅本章的*订阅者*子节。）

根据响应式流规范，有四种基本类型的接口：

+   发布者

+   订阅者

+   订阅

+   处理器

让我们来看看这些类型中的每一个。

## 发布者

发布者向一个或多个订阅者提供数据流。订阅者使用`subscribe()`方法订阅发布者。每个订阅者只能向一个发布者订阅一次。

最重要的是，发布者根据从订阅者收到的需求推送数据。响应式流是懒加载的；因此，只有当有订阅者时，发布者才会推送元素。

`Publisher`接口定义如下：

```java
package org.reactivestreams;// T – type of element Publisher sends
public interface Publisher<T> {
  public void subscribe(Subscriber<? super T> s); }
```

在这里，`发布者`接口包含`subscribe`方法。让我们在下一小节中了解`订阅者`类型。

## 订阅者

订阅者消费发布者推送的数据。发布者-订阅者通信工作如下：

1.  当一个`订阅者`实例传递给`Publisher.subscribe()`方法时，它会触发`onSubscribe()`方法。它包含一个`Subscription`参数，该参数控制背压，即订阅者从发布者那里请求的数据量。

1.  在第一步之后，`发布者`等待`Subscription.request(long)`调用。它只有在`Subscription.request()`调用之后才会向`订阅者`推送数据。此方法指示`发布者`订阅者一次可以接收多少项。

通常，发布者将数据推送到订阅者，无论订阅者是否能够安全处理。然而，订阅者最清楚它能安全处理多少数据；因此，在 Reactive Streams 中，`订阅者`使用`Subscription`实例将元素数量的需求传达给`发布者`。这被称为**背压**或**流量控制**。

你可能正在想，如果`发布者`要求`订阅者`减速，但`订阅者`无法做到，那会怎样？在这种情况下，`发布者`必须决定是失败、放弃还是缓冲。

1.  一旦在*步骤 2*中提出需求，`发布者`会发送数据通知，并使用`onNext()`方法来消费数据。此方法将在`发布者`根据`Subscription.request()`传达的需求推送数据通知之前被触发。

1.  最后，无论是`onError()`还是`onCompletion()`都会被触发，作为终端状态。在这些调用之一被触发后，即使调用`Subscription.request()`也不会发送任何通知。以下是一些终端方法：

    +   一旦发生任何错误，`onError()`将被调用

    +   当所有元素都推送完毕时，`onCompletion()`将被调用

`订阅者`接口被定义为如下：

```java
package org.reactivestreams;// T – type of element Publisher sends
public interface Subscriber<T> {
  public void onSubscribe(Subscription s);
  public void onNext(T t);
  public void onError(Throwable t);
  public void onComplete();
}
```

## 订阅

订阅是发布者和订阅者之间的调解者。订阅者的责任是调用`Subscription.subscriber()`方法，并让发布者知道需求。它可以根据订阅者的需要随时调用。

`cancel()`方法要求发布者停止发送数据通知并清理资源。

订阅被定义为如下：

```java
package org.reactivestreams;public interface Subscription {
  public void request(long n);
  public void cancel();
}
```

## 处理器

处理器是发布者和订阅者之间的桥梁，代表处理阶段。它既作为发布者又作为订阅者工作，并遵守双方定义的合同。它被定义为如下：

```java
package org.reactivestreams;public interface Processor<T, R>
  extends Subscriber<T>, Publisher<R> {
}
```

让我们看看以下示例。在这里，我们通过使用`Flux.just()`静态工厂方法创建`Flux`。`Flux`是 Project Reactor 中的一个发布者类型。这个发布者包含四个整数元素。然后，我们使用`reduce`操作符（就像我们在 Java 流中做的那样）对它执行`求和`操作：

```java
Flux<Integer> fluxInt = Flux.just(1, 10, 100, 1000).log();fluxInt.reduce(Integer::sum)
  .subscribe(sum ->
      System.out.printf("Sum is: %d", sum));
```

当你运行前面的代码时，它将打印以下输出：

```java
11:00:38.074 [main] INFO reactor.Flux.Array.1 - | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)11:00:38.074 [main] INFO reactor.Flux.Array.1 - |request(unbounded)
11:00:38.084 [main] INFO reactor.Flux.Array.1 - | onNext(1)
11:00:38.084 [main] INFO reactor.Flux.Array.1 - | onNext(10)
11:00:38.084 [main] INFO reactor.Flux.Array.1 - | onNext(100)
11:00:38.084 [main] INFO reactor.Flux.Array.1 - | onNext(1000)
11:00:38.084 [main] INFO reactor.Flux.Array.1 - | onComplete() Sum is: 1111
Process finished with exit code 0
```

观察输出，当`Publisher`被订阅时，`Subscriber`发送无界的`Subscription.request()`。当第一个元素被通知时，调用`onNext()`，依此类推。最后，当发布者完成推送元素时，调用`onComplete()`事件。这就是响应式流的工作方式。

既然你已经了解了响应式流的工作原理，让我们看看 Spring WebFlux 模块是如何以及为什么使用响应式流的。

# 探索 Spring WebFlux

现有的 Servlet API 是阻塞 API。它们使用输入和输出流，这些是阻塞 API。Servlet 3.0 容器不断进化，并使用底层的事件循环。异步请求异步处理，但读写操作仍然使用阻塞的输入/输出流。*Servlet 3.1*容器进一步进化，支持异步性，并具有非阻塞 I/O 流 API。然而，某些 Servlet API，如`request.getParameters()`，解析阻塞请求体，并提供如`Filter`之类的同步合约。**Spring MVC**框架基于 Servlet API 和 Servlet 容器。

因此，Spring 提供了**Spring WebFlux**，这是一个完全非阻塞的，并提供背压功能的框架。它使用少量线程提供并发性，并且随着硬件资源的减少而扩展。WebFlux 提供了流畅的、函数式的和延续风格的 API，以支持声明式异步逻辑的组合。编写异步函数式代码比编写命令式代码更复杂。然而，一旦你上手了，你会爱上它，因为它允许你编写精确且易于阅读的代码。

Spring WebFlux 和 Spring MVC 可以共存；然而，为了确保响应式编程的有效使用，你绝不应该将响应式流程与阻塞调用混合。

Spring WebFlux 支持以下特性和架构：

+   事件循环并发模型

+   既有注解控制器也有功能端点

+   响应式客户端

+   基于 Netty 和 Servlet 3.1 容器（如 Tomcat、Undertow 和 Jetty）的 Web 服务器

既然你对 WebFlux 有了些了解，你可以通过理解响应式 API 和 Reactor Core 来深入了解 WebFlux 的工作原理。让我们首先探索响应式 API。你将在后续小节中探索 Reactor Core。

## 理解响应式 API

Spring WebFlux API 是响应式 API，接受`Publisher`作为普通输入。WebFlux 然后将其适配为响应式库（如 Reactor Core 或 RxJava）支持的类型。然后处理输入，并以响应式库支持的格式返回输出。这使得 WebFlux API 可以与其他响应式库互操作。

默认情况下，Spring WebFlux 使用 Reactor ([`projectreactor.io`](https://projectreactor.io)) 作为核心依赖。Project Reactor 提供了响应式流库。如前所述，WebFlux 接受输入作为 `Publisher`，然后将其适配为 Reactor 类型，然后作为 `Mono` 或 `Flux` 输出返回。

你知道在响应式流中，`Publisher` 根据需求将数据推送到其订阅者。它可以推送一个或多个（可能是无限个）元素。Project Reactor 进一步扩展了这一点，并提供了两个 `Publisher` 实现，即 `Mono` 和 `Flux`。`Mono` 可以返回 `0` 或 `1` 个元素给 `Subscriber`，而 `Flux` 返回 `0` 到 `N` 个元素。这两个都是实现了 `CorePublisher` 接口的抽象类。`CorePublisher` 接口扩展了发布者。

通常，我们在仓库中有以下方法：

```java
public Product findById(UUID id);public List<Product> getAll();
```

这些可以替换为 `Mono` 和 `Flux`：

```java
Public Mono<Product> findById(UUID id);public Flux<Product> getAll();
```

根据源是否可以重新启动，流可以是热流或冷流。如果冷流有多个订阅者，则源会被重新启动，而在热流中，多个订阅者使用相同的源。Project Reactor 流默认是冷流。因此，一旦你消费了一个流，你无法重用它，直到它重新启动。然而，Project Reactor 允许你使用 `cache()` 方法将冷流转换为热流。`Mono` 和 `Flux` 抽象类都支持冷流和热流。

让我们通过一些示例来理解冷流和热流的概念：

```java
Flux<Integer> fluxInt = Flux.just(1, 10, 100).log();fluxInt.reduce(Integer::sum).subscribe(sum ->       System.out.printf("Sum is: %d\n", sum));
fluxInt.reduce(Integer::max).subscribe(max ->   System.out.printf("Maximum is: %d", max));
```

在这里，我们创建了一个包含三个数字的 `Flux` 对象，`fluxInt`。然后，我们分别执行两个操作——`sum` 和 `max`。你可以看到有两个订阅者。默认情况下，Project Reactor 流是冷流；因此，当第二个订阅者注册时，它会重新启动，如下面的输出所示：

```java
11:23:35.060 [main] INFO reactor.Flux.Array.1 - | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)11:23:35.060 [main] INFO reactor.Flux.Array.1 - | request(unbounded)
11:23:35.060 [main] INFO reactor.Flux.Array.1 - | onNext(1)
11:23:35.060 [main] INFO reactor.Flux.Array.1 - | onNext(10)
11:23:35.060 [main] INFO reactor.Flux.Array.1 - | onNext(100)
11:23:35.060 [main] INFO reactor.Flux.Array.1 - | onComplete()
Sum is: 111
11:23:35.076 [main] INFO reactor.Flux.Array.1 - | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
11:23:35.076 [main] INFO reactor.Flux.Array.1 - | request(unbounded)
11:23:35.076 [main] INFO reactor.Flux.Array.1 - | onNext(1)
11:23:35.076 [main] INFO reactor.Flux.Array.1 - | onNext(10)
11:23:35.076 [main] INFO reactor.Flux.Array.1 - | onNext(100)
11:23:35.076 [main] INFO reactor.Flux.Array.1 - | onComplete()
Maximum is: 100
```

源是在同一程序中创建的，但如果源在其他地方，比如在 HTTP 请求中，或者你不想重新启动源怎么办？在这些情况下，你可以使用 `cache()` 将冷流转换为热流，如下面的代码块所示。以下代码与之前代码的唯一区别是我们向 `Flux.just()` 添加了一个 `cache()` 调用：

```java
Flux<Integer> fluxInt = Flux.just  (1, 10, 100).log().cache();
fluxInt.reduce(Integer::sum).subscribe(sum ->   System.out.printf("Sum is: %d\n", sum));
fluxInt.reduce(Integer::max).subscribe(max ->   System.out.printf("Maximum is: %d", max));
```

现在，看看输出。源没有重新启动；相反，再次使用了相同的源：

```java
11:29:25.665 [main] INFO reactor.Flux.Array.1 - | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)11:29:25.665 [main] INFO reactor.Flux.Array.1 - | request(unbounded)
11:29:25.665 [main] INFO reactor.Flux.Array.1 - | onNext(1)
11:29:25.665 [main] INFO reactor.Flux.Array.1 - | onNext(10)
11:29:25.665 [main] INFO reactor.Flux.Array.1 - | onNext(100)
11:29:25.665 [main] INFO reactor.Flux.Array.1 - | onComplete()
Sum is: 111
Maximum is: 100
```

现在我们已经触及了响应式 API 的核心，让我们看看 Spring WebFlux 的响应式核心包含什么。

## 响应式核心

响应式核心为使用 Spring 开发响应式 Web 应用程序提供了一个基础。Web 应用程序需要三个级别的支持来服务 HTTP 网络请求：

+   服务器使用以下方式处理网络请求：

    +   `HttpHandler`：`reactor.core.publisher.Mono` 包中的一个接口，它是对不同 HTTP 服务器 API（如 Netty 或 Tomcat）上的请求/响应处理程序的抽象：

        ```java
        public interface HttpHandler {  Mono<Void> handle(ServerHttpRequest request,     ServerHttpResponse response);}
        ```

    +   `WebHandler`：`org.springframework.web.server`包中的一个接口，它为用户会话、请求和会话属性、请求的本地化和主体、表单数据等提供支持。您可以在[`docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-web-handler-api`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-web-handler-api)找到有关`WebHandler`的更多信息。

+   客户端使用`WebClient`处理网络请求调用。

+   用于在服务器和客户端级别对请求和响应内容进行序列化和反序列化的编解码器（`Encoder`、`Decoder`、`HttpMessageWriter`、`HttpMessageReader`和`DataBuffer`）。

这些组件是 Spring WebFlux 的核心。WebFlux 应用程序配置还包含以下 bean - `webHandler` (`DispatcherHandler`)、`WebFilter`、`WebExceptionHandler`、`HandlerMapping`、`HandlerAdapter`和`HandlerResultHandler`。

对于 REST 服务实现，以下 Web 服务器有特定的`HandlerAdapter`实例 - Tomcat、Jetty、Netty 和 Undertow。支持反应式流的 Web 服务器，如 Netty，处理订阅者的需求。然而，如果服务器处理程序不支持反应式流，则使用`org.springframework.http.server.reactive.ServletHttpHandlerAdapter` HTTP `HandlerAdapter`。`HandlerAdapter`处理反应式流和 Servlet 3.1 容器异步 I/O 之间的适配，并实现一个`Subscriber`类。`HandlerAdapter`使用 OS TCP 缓冲区。OS TCP 使用自己的背压（控制流）；也就是说，当缓冲区满时，操作系统使用 TCP 背压来停止传入的元素。

浏览器或任何 HTTP 客户端都使用 HTTP 协议来消费 REST API。当网络服务器接收到请求时，它将其转发到 Spring WebFlux 应用程序。然后，WebFlux 构建通往控制器的反应式管道。`HttpHandler`是 WebFlux 与网络服务器之间的接口，它使用 HTTP 协议进行通信。如果底层服务器支持反应式流，例如 Netty，则服务器会原生地完成订阅。否则，WebFlux 使用`ServletHttpHandlerAdapter`来适配 Servlet 3.1 容器化服务器。`ServletHttpHandlerAdapter`随后将流适配到异步 I/O Servlet API，反之亦然。然后，通过`ServletHttpHandlerAdapter`进行反应式流的订阅。

因此，总结来说，`Mono`/`Flux` 流由 WebFlux 内部类订阅，当控制器发送 `Mono`/`Flux` 流时，这些类将其转换为 HTTP 数据包。HTTP 协议支持事件流。然而，对于其他媒体类型，如 JSON，Spring WebFlux 订阅 `Mono`/`Flux` 流，并等待触发 `onComplete()` 或 `onError()`。然后，它将整个元素列表序列化，或者对于 `Mono` 的情况，在一个 HTTP 响应中序列化单个元素。

Spring WebFlux 需要一个类似于 Spring MVC 中的 `DispatcherServlet` 的组件——一个前端控制器。让我们在下一节中讨论这个问题。

# 理解 `DispatcherHandler`

`DispatcherHandler` 是 Spring WebFlux 中的前端控制器，相当于 Spring MVC 框架中的 `DispatcherServlet`。`DispatcherHandler` 包含一个算法，该算法利用特殊组件——`HandlerMapping`（将请求映射到处理器）、`HandlerAdapter`（`DispatcherHandler` 的助手，用于调用映射到请求的处理器）和 `HandlerResultHandler`（用于处理结果并形成结果）——来处理请求。`DispatcherHandler` 组件由名为 `webHandler` 的 bean 标识。

它以以下方式处理请求：

1.  网络请求由 `DispatcherHandler` 接收。

1.  `DispatcherHandler` 使用 `HandlerMapping` 来查找与请求匹配的处理器，并使用第一个匹配项。

1.  然后它使用相应的 `HandlerAdapter` 来处理请求，该适配器在处理后会暴露 `HandlerResult`（`HandlerAdapter` 处理请求后返回的值）。返回值可能是以下之一——`ResponseEntity`、`ServerResponse`，或者来自 `@RestController` 的值，或者来自视图解析器的值（`CharSequence`、`view`、`map` 等等）。

1.  然后，它利用相应的 `HandlerResultHandler` 根据从 *步骤 2* 收到的 `HandlerResult` 类型来写入响应或渲染视图。`ResponseEntityResultHandler` 用于 `ResponseEntity`，`ServerResponseResultHandler` 用于 `ServerResponse`，`ResponseBodyResultHandler` 用于由 `@RestController` 或 `@ResponseBody` 注解的方法返回的值，而 `ViewResolutionResultHandler` 用于视图解析器返回的值。

1.  请求完成。

你可以在 Spring WebFlux 中使用注解控制器（如 Spring MVC）或功能端点来创建 REST 端点。让我们在下一节中探讨这些内容。

## 控制器

Spring 团队为 Spring MVC 和 Spring WebFlux 保留了相同的注解，因为这些注解是非阻塞的。因此，你可以使用我们在前几章中使用过的相同注解来创建 REST 控制器。在 Spring WebFlux 中，注解运行在响应式核心上，并提供非阻塞流。然而，作为开发人员，你有责任维护一个完全非阻塞的流和响应式链（管道）。任何在响应式链中的阻塞调用都将将响应式链转换为阻塞调用。

让我们创建一个简单的支持非阻塞和响应式调用的 REST 控制器：

```java
@RestControllerpublic class OrderController {
  @RequestMapping(value = "/api/v1/orders",  method =
    RequestMethod.POST)
  public ResponseEntity<Order> addOrder(
           @RequestBody NewOrder newOrder){
    // …
  }
  @RequestMapping(value = "/api/v1/orders/{id}", method =
     RequestMethod.GET)
  public ResponseEntity<Order>getOrderById(
    @PathVariable("id") String id){
    // …
  }
}
```

你可以看到，它使用了我们在 Spring MVC 中使用过的所有注解：

+   `@RestController` 用于标记一个类为 REST 控制器。如果没有这个注解，端点将不会注册，请求将返回 `NOT FOUND 404`。

+   `@RequestMapping` 用于定义路径和 HTTP 方法。在这里，你也可以仅使用路径使用 `@PostMapping`。同样，对于每个 HTTP 方法，都有一个相应的映射，例如 `@GetMapping`。

+   `@RequestBody` 注解将一个参数标记为请求体，并使用适当的编解码器进行转换。同样，`@PathVariable` 和 `@RequestParam` 分别用于路径参数和查询参数。

我们将使用基于注解的模型来编写 REST 端点。当我们使用 WebFlux 实现电子商务应用控制器时，你将更详细地了解它。Spring WebFlux 还提供了一种使用函数式编程风格编写 REST 端点的方法，你将在下一节中探索。

## 函数式端点

我们使用 Spring MVC 编写的 REST 控制器是命令式编程风格的。另一方面，响应式编程是函数式编程风格。因此，Spring WebFlux 也允许使用函数式端点来定义 REST 端点。这些端点也使用相同的响应式核心基础。

让我们看看我们如何使用函数式端点编写示例电子商务应用的相同 `Order` REST 端点：

```java
import static org.springframework.http.MediaType.  APPLICATION_JSON;
import static org.springframework.web.reactive.
  function.server. RequestPredicates.*;
import staticorg.springframework.
web.reactive.function.server. RouterFunctions.route;
// ...
  OrderRepository repository = ...
  OrderHandler handler = new OrderHandler(repository);
  RouterFunction<ServerResponse> route = route()
    .GET("/v1/api/orders/{id}",
          accept(APPLICATION_JSON),
          handler::getOrderById)
    .POST("/v1/api/orders", handler::addOrder)
    .build();
  public class OrderHandler {
    public Mono<ServerResponse> addOrder
       (ServerRequest req){
      // ...
    }
    public Mono<ServerResponse> getOrderById(
       ServerRequest req) {
    // ...
    }
  }
```

在前面的代码中，你可以看到 `RouterFunctions.route()` 构建器允许你使用函数式编程风格在一个语句中编写所有的 REST 路由。然后，它使用处理类的方法引用来处理请求，这与基于注解模型的 `@RequestMapping` 主体相同。

让我们在 `OrderHandler` 方法中添加以下代码：

```java
public class OrderHandler {  public Mono<ServerResponse> addOrder(
     ServerRequest req){
    Mono<NewOrder> order = req.bodyToMono(NewOrder.class);
    return ok()
      .build(repository.save(toEntity(order)));
  }
  public Mono<ServerResponse> getOrderById(
      ServerRequest req) {
    String orderId = req.pathVariable("id");
    return repository
      .getOrderById(UUID.fromString(orderId))
      .flatMap(order -> ok()
          .contentType(APPLICATION_JSON)
          .bodyValue(toModel(order)))
      .switchIfEmpty(ServerResponse.notFound()
      .build());
  }
}
```

与 REST 控制器中的`@RequestMapping()`映射方法不同，处理方法没有多个参数，如 body、path 或查询参数。它们只有一个`ServerRequest`参数，可以用来提取 body、path 和查询参数。在`addOrder`方法中，使用`request.bodyToMono()`提取`Order`对象，它解析请求体并将其转换为`Order`对象。同样，`getOrderById()`方法通过调用`request.pathVariable("id")`从服务器请求对象中检索由给定 ID 标识的`order`对象。

现在，让我们讨论响应。处理方法使用`ServerResponse`对象，而不是 Spring MVC 中的`ResponseEntity`。因此，`ok()`静态方法看起来像是来自`ResponseEntity`，但实际上来自`org.springframework.web.reactive.function.server.ServerResponse.ok`。Spring 团队试图使 API 尽可能类似于 Spring MVC；然而，底层实现不同，提供了一个非阻塞的反应式接口。

关于这些处理方法最后一点是响应的编写方式。它使用函数式风格而不是命令式风格，并确保反应式链不会断裂。在两种情况下，仓库都返回`Mono`对象（一个发布者），作为包裹在`ServerResponse`中的响应。

你可以在`getOrderById()`处理方法中找到有趣的代码。它对从仓库接收到的`Mono`对象执行`flatMap`操作。它将其从实体转换为模型，然后将其包装在`ServerResponse`对象中，并返回响应。你可能想知道如果仓库返回 null 会发生什么。根据契约，仓库返回`Mono`，这与 Java 的`Optional`类性质相似。因此，根据契约，`Mono`对象可以是空的但不能为 null。如果仓库返回一个空的`Mono`，则将使用`switchIfEmpty()`操作符，并发送`NOT FOUND 404`响应。

在出错的情况下，可以使用不同的错误操作符，例如`doOnError()`或`onErrorReturn()`。

我们已经讨论了使用`Mono`类型进行逻辑流程；如果你用`Flux`类型代替`Mono`类型，同样的解释也适用。

我们已经讨论了在 Spring 环境中与反应式、异步和非阻塞编程相关的许多理论。现在让我们开始编码，将第*第四章*中开发的电子商务 API，*为 API 编写业务逻辑*，迁移到反应式 API。

# 为我们的电子商务应用实现反应式 API

既然你已经了解了 Reactive Streams 的工作原理，我们可以继续实现异步和非阻塞的 REST API。

你会记得我们正在遵循设计优先的方法，因此我们首先需要 API 设计规范。然而，我们可以重用我们在*第三章*中创建的电子商务 API 规范，*API 规范和实现*。

OpenAPI Codegen 用于生成创建符合 Spring MVC 规范的 API Java 接口的 API 接口/合约。让我们看看我们需要进行哪些更改来生成 reactive API 接口。

## 修改 OpenAPI Codegen 以支持 reactive API

你需要调整一些 OpenAPI Codegen 配置以生成符合 Spring WebFlux 规范的 Java 接口，如下所示：

```java
{  "library": "spring-boot",
  "dateLibrary": "java8",
  "hideGenerationTimestamp": true,
  "modelPackage": "com.packt.modern.api.model",
  "apiPackage": "com.packt.modern.api",
  "invokerPackage": "com.packt.modern.api",
  "serializableModel": true,
  "useTags": true,
  "useGzipFeature" : true,
  "reactive": true,
  "interfaceOnly": true,
  …
  …
}
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/resources/api/config.json`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/resources/api/config.json)

只有当你选择`spring-boot`作为库时，才会提供 Reactive API 支持。此外，你需要将`reactive`标志设置为`true`。默认情况下，`reactive`标志是`false`。

现在，你可以运行以下命令：

```java
$ gradlew clean generateSwaggerCode
```

这将生成符合 Reactive Streams 规范的 Java 接口，这些接口是基于注解的 REST 控制器接口。当你打开任何 API 接口时，你会在其中找到`Mono`/`Flux`反应器类型，如下面的`OrderAPI`接口代码块所示：

```java
@Operation(  operationId = "addOrder",
  summary = "Creates a new order for the …",
  tags = { "order" },
  responses = {
    @ApiResponse(responseCode = "201",
      description = "Order added successfully",
      content = {
        @Content(mediaType = "application/xml",
          schema = @Schema(
            implementation = Order.class)),
        @Content(mediaType = "application/json",
          schema = @Schema(
            implementation = Order.class))
      }),
      @ApiResponse(responseCode = "406",
        description = "If payment not authorized")
  }
)
@RequestMapping(
  method = RequestMethod.POST,
  value = "/api/v1/orders",
  produces = { "application/xml",
               "application/json" },
  consumes = { "application/xml",
               "application/json" }
)
default Mono<ResponseEntity<Order>> addOrder(
 @Parameter(name = "NewOrder", description =
     "New Order Request object")
     @Valid @RequestBody(required = false)
          Mono<NewOrder> newOrder,
 @Parameter(hidden = true)
  final ServerWebExchange exg) throws Exception {
```

你会观察到另一个变化：对于 reactive 控制器，还需要一个额外的参数，`ServerWebExchange`。

现在，当你编译你的代码时，你可能会发现编译错误，因为我们还没有添加所需的 reactive 支持依赖项。让我们在下一节中学习如何添加它们。

## 在 build.xml 中添加 Reactive 依赖

首先，我们将移除`spring-boot-starter-web`，因为我们现在不需要 Spring MVC。其次，我们将添加`spring-boot-starter-webflux`和`reactor-test`以支持 Spring WebFlux 和 Reactor 支持测试。一旦成功添加这些依赖项，你就不应该在 OpenAPI 生成的代码中看到任何编译错误。

你可以将所需的 reactive 依赖项添加到`build.gradle`中，如下所示：

```java
implementation 'org.springframework.boot:  spring-boot-starter-webflux'
testImplementation('org.springframework.boot:
  spring-boot-starter-test')
testImplementation 'io.projectreactor:reactor-test'
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/build.gradle`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/build.gradle)

我们需要从 REST 控制器到数据库的完整 reactive 管道。然而，现有的 JDBC 和 Hibernate 依赖项仅支持阻塞调用。JDBC 是一个完全阻塞的 API。Hibernate 也是阻塞的。因此，我们需要为数据库提供 reactive 依赖项。

Hibernate Reactive（[`github.com/hibernate/hibernate-reactive`](https://github.com/hibernate/hibernate-reactive)）是在本书第一版之后发布的。Hibernate Reactive 支持 PostgreSQL、MySQL/MariaDB、Db2 11.5+、CockroachDB 22.1+、MS SQL Server 2019+和 Oracle Database 21+。在撰写本文时，Hibernate Reactive 不支持 H2。因此，我们将简单地使用 Spring Data，这是一个提供`spring-data-r2dbc`库以处理响应式流的 Spring 框架。

许多 NoSQL 数据库，如 MongoDB，已经提供了响应式数据库驱动程序。对于关系数据库，应使用基于 R2DBC 的驱动程序来替代 JDBC，以实现完全非阻塞/响应式 API 调用。**R2DBC**代表**响应式关系数据库连接**。R2DBC 是一个响应式 API 开放规范，它为数据库驱动程序建立了一个**服务提供者接口**（**SPI**）。几乎所有流行的关系数据库都支持 R2DBC 驱动程序——H2、Oracle Database、MySQL、MariaDB、SQL Server、PostgreSQL 和 R2DBC Proxy。

让我们在`build.gradle`文件中添加 Spring Data 和 H2 的 R2DBC 依赖项：

```java
implementation 'org.springframework.boot:spring-boot-starter-data-r2dbc'implementation 'com.h2database:h2'
runtimeOnly 'io.r2dbc:r2dbc-h2'
```

现在，我们可以编写端到端（从控制器到仓库）的代码，而不会出现任何编译错误。在我们开始编写 API Java 接口的实现之前，让我们先添加全局异常处理。

## 处理异常

我们将以与在*第三章*中添加 Spring MVC 全局异常处理相同的方式添加全局异常处理器，*API 规范和实现*。在此之前，你可能想知道如何在响应式管道中处理异常。响应式管道是一系列流，你不能像在命令式代码中那样添加异常处理。你需要在管道流程中仅提出错误。

查看以下代码：

```java
.flatMap(card -> {  if (Objects.isNull(card.getId())) {
    return service.registerCard(mono)
      .map(ce -> status(HttpStatus.CREATED)
        .body(assembler.entityToModel(
           ce, exchange)));
  } else {
    return Mono.error(() -> new
      CardAlreadyExistsException(
        " for user with ID - " + d.getId()));
  }
})
```

在这里，执行了一个`flatMap`操作。如果`card`无效，即`card`没有请求的`ID`，则应该抛出一个错误。在这里，使用了`Mono.error()`，因为管道期望返回`Mono`对象。同样，如果你期望返回类型为`Flux`，也可以使用`Flux.error()`。

假设你期待从服务或仓库调用中获取一个对象，但结果却收到了一个空对象。这时，你可以使用`switchIfEmpty()`操作符，如下面的代码所示：

```java
Mono<List<String>> monoIds =  itemRepo.findByCustomerId( customerId)
    .switchIfEmpty(Mono.error(new
       ResourceNotFoundException(". No items
         found in Cart of customer with Id - " +
           customerId)))
    .map(i -> i.getId().toString())
    .collectList().cache();
```

在这里，代码期望从`item`仓库中获取`List`类型的`Mono`对象。然而，如果返回的对象为空，则它将简单地抛出`ResourceNotFoundException.switchIfEmpty()`异常，并接受替代的`Mono`实例。

到现在为止，你可能对异常的类型有所疑问。它会抛出一个运行时异常。请在此处查看`ResourceNotFoundException`类的声明：

```java
public class ResourceNotFoundException     extends RuntimeException
```

同样，你也可以使用来自响应式流的`onErrorReturn()`、`onErrorResume()`或类似的错误操作符。看看下一个代码块中`onErrorReturn()`的使用：

```java
return service.getCartByCustomerId(customerId)  .map(cart -> assembler
    .itemfromEntities(cart.getItems().stream()
      .filter(i -> i.getProductId().toString()
       .equals(itemId.trim())).collect(toList()))
      .get(0)).map(ResponseEntity::ok)
  .onErrorReturn(notFound().build())
```

所有异常都应该被处理，并且应该向用户发送错误响应。我们将在下一节中查看全局异常处理器。

## 处理控制器的全局异常

我们在 Spring MVC 中使用 `@ControllerAdvice` 创建了一个全局异常处理器。对于 Spring WebFlux 中的错误处理，我们将采取不同的路线。首先，我们将创建 `ApiErrorAttributes` 类，这个类也可以在 Spring MVC 中使用。这个类扩展了 `DefaultErrorAttributes`，它是 `ErrorAttributes` 接口的一个默认实现。`ErrorAttributes` 接口提供了一种处理映射、错误字段映射及其值的方式。这些错误属性可以用来向用户显示错误或用于日志记录。

`DefaultErrorAttributes` 类提供了以下属性：

+   `timestamp`: 错误被捕获的时间

+   `status`: 状态码

+   `error`: 错误描述

+   `exception`: 根异常的类名（如果已配置）

+   `message`: 异常消息（如果已配置）

+   `errors`: 来自 `BindingResult` 异常的任何 `ObjectError`（如果已配置）

+   `trace`: 异常堆栈跟踪（如果已配置）

+   `path`: 异常被抛出的 URL 路径

+   `requestId`: 与当前请求关联的唯一 ID

我们已经向状态和消息中添加了两个默认值——一个内部服务器错误和一个通用错误消息（`系统无法完成请求。请联系系统支持。`），分别添加到 `ApiErrorAttributes` 中，如下所示：

```java
@Componentpublic class ApiErrorAttributes
  extends DefaultErrorAttributes {
  private HttpStatus status =
      HttpStatus.INTERNAL_SERVER_ERROR;
  private String message =
     ErrorCode.GENERIC_ERROR.getErrMsgKey();
  @Override
  public Map<String, Object>
    getErrorAttributes( ServerRequest request,
       ErrorAttributeOptions options) {
    var attributes =
      super.getErrorAttributes(request, options);
    attributes.put("status", status);
    attributes.put("message", message);
    attributes.put("code", ErrorCode.
      GENERIC_ERROR.getErrCode());
    return attributes;
  }
  // Getters and setters
}
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/exception/ApiErrorAttributes.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/exception/ApiErrorAttributes.java)

现在，我们可以在自定义的全局异常处理器类中使用这个 `ApiErrorAttributes` 类。我们将创建 `ApiErrorWebExceptionHandler` 类，它扩展了 `AbstractErrorWebExceptionHandler` 抽象类。

`AbstractErrorWebExceptionHandler` 类实现了 `ErrorWebExceptionHandler` 和 `InitializingBean` 接口。`ErrorWebExceptionHandler` 是一个扩展了 `WebExceptionHandler` 接口的功能接口，这表明 `WebExceptionHandler` 用于渲染异常。`WebExceptionHandler` 是在服务器交换处理过程中处理异常的契约。

`InitializingBean` 接口是 Spring 核心框架的一部分。它被用于当所有属性被填充时做出反应的组件。它也可以用来检查是否设置了所有必需的属性。

现在我们已经学习了基础知识，让我们开始编写 `ApiErrorAttributes` 类：

```java
@Component@Order(-2)
public class ApiErrorWebExceptionHandler extends
  AbstractErrorWebExceptionHandler {
 public ApiErrorWebExceptionHandler(
    ApiErrorAttributes errorAttributes,
    ApplicationContext appCon,
    ServerCodecConfigurer serverCodecConfigurer){
  super(errorAttributes,
     new WebProperties().getResources(),appCon);
  super.setMessageWriters(
     serverCodecConfigurer.getWriters());
  super.setMessageReaders(
     serverCodecConfigurer.getReaders());
 }
 @Override
 protected RouterFunction<ServerResponse>
   getRoutingFunction(ErrorAttributes errA) {
   return RouterFunctions.route(
     RequestPredicates.all(),
       this::renderErrorResponse);
}
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/exception/ApiErrorWebExceptionHandler.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/exception/ApiErrorWebExceptionHandler.java)

关于此代码的第一个重要观察是，我们添加了`@Order`注解，它告诉我们执行的优先级。Spring 框架将`ResponseStatusExceptionHandler`放置在`0`索引处，而`DefaultErrorWebExceptionHandler`则按`-1`索引排序。这两个都是像我们创建的那样异常处理器。如果你不给`ApiErrorWebExceptionHandler`设置优先级，以超过两者，那么它将永远不会执行。因此，优先级设置为`-2`。

接下来，此类覆盖了`getRoutingFunction()`方法，它调用私有定义的`renderErrorResponse()`方法，其中我们有自己的自定义错误处理实现，如下所示：

```java
private Mono<ServerResponse> renderErrorResponse(    ServerRequest request) {
  Map<String, Object> errorPropertiesMap =
     getErrorAttributes(request,
      ErrorAttributeOptions.defaults());
  Throwable throwable = (Throwable) request
     .attribute("org.springframework.boot.web
                .reactive.error
                .DefaultErrorAttributes.ERROR")
     .orElseThrow(() -> new IllegalStateException
     ("Missing exception attribute in ServerWebExchange"));
  ErrorCode errorCode = ErrorCode.GENERIC_ERROR;
  if (throwable instanceof
      IllegalArgumentException || throwable
      instanceof DataIntegrityViolationException
      || throwable instanceof
      ServerWebInputException) {
     errorCode = ILLEGAL_ARGUMENT_EXCEPTION;
  } else if (throwable instanceof
      CustomerNotFoundException) {
    errorCode = CUSTOMER_NOT_FOUND;
  } else if (throwable instanceof
      ResourceNotFoundException) {
    errorCode = RESOURCE_NOT_FOUND;
  } // other else-if
  // …
}
```

https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/exception/ApiErrorWebExceptionHandler.java

在这里，首先，我们从`errorPropertiesMap`中提取错误属性。这将在我们形成错误响应时使用。接下来，我们使用`throwable`捕获发生的异常。然后，我们检查异常的类型，并为其分配适当的代码。我们保留默认的`GenericError`，这不过是`InternalServerError`。

接下来，我们使用`switch`语句根据引发的异常形成错误响应，如下所示：

```java
switch (errorCode) {  case ILLEGAL_ARGUMENT_EXCEPTION ->{errorPropertiesMap.put
    ("status", HttpStatus.BAD_REQUEST);
    errorPropertiesMap.put("code",
       ILLEGAL_ARGUMENT_EXCEPTION.getErrCode());
    errorPropertiesMap.put("error",
       ILLEGAL_ARGUMENT_EXCEPTION);
    errorPropertiesMap.put("message", String
      .format("%s %s",
       ILLEGAL_ARGUMENT_EXCEPTION.getErrMsgKey(),
       throwable.getMessage()));
    return ServerResponse.status(
          HttpStatus.BAD_REQUEST)
        .contentType(MediaType.APPLICATION_JSON)
        .body(BodyInserters.fromValue(
          errorPropertiesMap));
  }
  case CUSTOMER_NOT_FOUND -> {
    errorPropertiesMap.put("status",
       HttpStatus.NOT_FOUND);
    errorPropertiesMap.put("code",
       CUSTOMER_NOT_FOUND.getErrCode());
    errorPropertiesMap.put("error",
       CUSTOMER_NOT_FOUND);
    errorPropertiesMap.put("message", String
       .format("%s %s",
        CUSTOMER_NOT_FOUND.getErrMsgKey(),
        throwable.getMessage()));
    return ServerResponse.status(
         HttpStatus.NOT_FOUND)
        .contentType(MediaType.APPLICATION_JSON)
        .body(BodyInserters.fromValue(
          errorPropertiesMap));
  }
  case RESOURCE_NOT_FOUND -> {
    // rest of the code …
}
```

在 Java 的下一个版本中，我们可能能够将`if-else`和`switch`块结合起来，使代码更加简洁。你还可以创建一个单独的方法，该方法接受`errorPropertiesMap`作为参数，并根据它返回形成的服务器响应。然后，你可以使用`switch`。

正在使用来自现有代码的自定义应用程序异常类，例如`CustomerNotFoundException`，以及其他异常处理支持的类，例如`ErrorCode`和`Error`（来自*第四章*，*为 API 编写业务逻辑*）。

现在我们已经研究了异常处理，我们可以专注于 HATEOAS。

## 在 API 响应中添加超媒体链接

对于反应式 API，存在 HATEOAS 支持，这与我们在上一章中使用 Spring MVC 所做的是类似的。我们再次创建这些组装器以支持 HATEOAS。我们还使用 HATEOAS 组装器类将模型转换为实体，反之亦然。

Spring WebFlux 提供了用于形成超媒体链接的`ReactiveRepresentationModelAssembler`接口。我们将覆盖其`toModel()`方法，向响应模型添加链接。

在这里，我们将做一些基础工作来填充链接。我们将创建一个具有单个默认方法的`HateoasSupport`接口，如下所示：

```java
public interface HateoasSupport {  default UriComponentsBuilder
     getUriComponentBuilder(@Nullable
            ServerWebExchange exchange) {
    if (exchange == null) {
      return UriComponentsBuilder.fromPath("/");
    }
    ServerHttpRequest request = exchange.getRequest();
    PathContainer contextPath = request.getPath().
      contextPath();
    return UriComponentsBuilder
          .fromHttpRequest(request)
          .replacePath(contextPath.toString())
          .replaceQuery("");
  }
}
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/hateoas/HateoasSupport.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/hateoas/HateoasSupport.java)

在这里，这个类包含一个默认方法，`getUriCompononentBuilder()`，它接受`ServerWebExchange`作为参数，并返回`UriComponentsBuilder`实例。然后，可以使用此实例提取用于添加带有协议、主机和端口的链接的服务器 URI。如果您还记得，`ServerWebExchange`参数被添加到控制器方法中。此接口用于获取 HTTP 请求、响应和其他属性。

现在，我们可以使用这两个接口——`HateoasSupport`和`ReactiveRepresentation` ModelAssembler——来定义表示模型组装器。

让我们定义地址的表示模型组装器，如下所示：

```java
@Componentpublic class AddressRepresentationModelAssembler
  implements ReactiveRepresentationModelAssembler
      <AddressEntity, Address>, HateoasSupport {
  private static String serverUri = null;
  private String getServerUri(
        @Nullable ServerWebExchange exch) {
    if (Strings.isBlank(serverUri)) {
      serverUri = getUriComponentBuilder
        (exch).toUriString();
    }
    return serverUri;
  }
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/hateoas/AddressRepresentationModelAssembler.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/hateoas/AddressRepresentationModelAssembler.java)

在这里，我们定义了另一个私有方法，`getServerUri()`，它从`UriComponentBuilder`中提取服务器 URI，而`UriComponentBuilder`本身是由`HateoasSupport`接口的默认`getUriComponentBuilder()`方法返回的。

现在，我们可以覆盖`toModel()`方法，如下面的代码块所示：

AddressRepresentationModelAssembler.java

```java
@Overridepublic Mono<Address> toModel(AddressEntity entity,
  ServerWebExchange exch) {
  return Mono.just(entityToModel(entity, exch));
}
public Address entityToModel(AddressEntity entity,
  ServerWebExchange exch) {
  Address resource = new Address();
  if(Objects.isNull(entity)) {
    return resource;
  }
  BeanUtils.copyProperties(entity, resource);
  resource.setId(entity.getId().toString());
  String serverUri = getServerUri(exchange);
  resource.add(Link.of(String.format(
      "%s/api/v1/addresses", serverUri))
      .withRel("addresses"));
  resource.add(Link.of(String.format(
      "%s/api/v1/addresses/%s",serverUri,
      entity.getId())).withSelfRel());
  return resource;
}
```

`toModel()`方法返回一个包含超媒体链接的`Mono<Address>`对象，这些链接是通过使用`entityToModel()`方法从`AddressEntity`实例形成的。

`entityToModel()`将实体实例的属性复制到模型实例。最重要的是，它使用`resource.add()`方法向模型添加超媒体链接。`add()`方法接受`org.springframework.hateoas.Link`实例作为参数。然后，我们使用`Link`类的`of()`静态工厂方法来形成链接。您可以看到这里使用服务器 URI 来添加到链接中。您可以形成尽可能多的链接，并使用`add()`方法将这些链接添加到资源中。

`ReactiveRepresentationModelAssembler`接口提供了`toCollectionModel()`方法，它有一个默认实现，返回`Mono<CollectionModel<D>>`集合模型。然而，我们也可以添加`toListModel()`方法，如下所示，它使用`Flux`返回地址列表：

AddressRepresentationModelAssembler.java

```java
public Flux<Address> toListModel(         Flux<AddressEntity> ent,
         ServerWebExchange exchange) {
  if (Objects.isNull(ent)) {
    return Flux.empty();
  }
  return Flux.from(ent.map(e ->
            entityToModel(e, exchange)));
}
```

此方法内部使用`entityToModel()`方法。同样，你可以为其他 API 模型创建表示模型装配器。你可以在[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter05/src/main/java/com/packt/modern/api/hateoas`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter05/src/main/java/com/packt/modern/api/hateoas)找到所有这些模型。

现在我们已经完成了基本的代码基础设施，我们可以根据 OpenAPI Codegen 生成的接口开发 API 实现。在这里，我们首先将开发将被服务消费的仓库。最后，我们将编写控制器实现。让我们从仓库开始。

## 定义实体

实体的定义方式与我们在*第四章*，“为 API 编写业务逻辑”中定义和使用它们的方式大致相同。然而，我们不会使用 Hibernate 映射和 JPA，而是使用 Spring Data 注解，如下所示：

```java
@Table("ecomm.orders")public class OrderEntity {
  @Id
  @Column("id")
  private UUID id;
  @Column("customer_id")
  private UUID customerId;
  @Column("address_id")
  private UUID addressId;
  @Column("card_id")
  private UUID cardId;
  @Column("order_date")
  private Timestamp orderDate;
  // other fields mapped to table columns
  private UUID cartId;
  private UserEntity userEntity;
  private AddressEntity addressEntity;
  private PaymentEntity paymentEntity;
  private List<ShipmentEntity> shipments = new ArrayList<>();
  // other entities fields and getters/setters
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/entity/OrderEntity.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/entity/OrderEntity.java)

在这里，因为我们使用 Spring Data 代替 Hibernate，所以我们使用 Spring Data 注解，即`@Table`来关联实体类和表名，以及`@Column`来映射字段到表的列。显然，`@Id`用作标识列。同样，你可以定义其他实体。

在定义了实体之后，我们将在下一小节中添加仓库。

## 添加仓库

仓库是我们应用程序代码和数据库之间的接口。它与你在 Spring MVC 中使用的仓库相同。然而，我们正在使用反应式范式编写代码。因此，需要具有使用 R2DBC-/反应式驱动程序的仓库，并在 Reactive Streams 之上返回反应式类型实例。这就是为什么我们不能使用 JDBC 的原因。

Spring Data R2DBC 为 Reactor 和 RxJava 提供了不同的仓库，例如`ReactiveCrudRepository`、`ReactiveSortingRepository`、`RxJava2CrudRepository`和`RxJava3CrudRepository`。此外，你也可以编写自己的自定义实现。

我们将使用`ReactiveCrudRepository`并编写一个自定义实现。

我们将为`Order`实体编写仓库。对于其他实体，你可以在[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter05/src/main/java/com/packt/modern/api/repository`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter05/src/main/java/com/packt/modern/api/repository)找到仓库。

首先，让我们为`Order`实体编写 CRUD 仓库，如下所示：

```java
@Repositorypublic interface OrderRepository extends
   ReactiveCrudRepository<OrderEntity, UUID>,
      OrderRepositoryExt {
  @Query("select o.* from ecomm.orders o join
           ecomm.\"user\" u on o.customer_id =
           u.id where u.id = :cusId")
  Flux<OrderEntity> findByCustomerId(UUID cusId);
}
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/repository/OrderRepository.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/repository/OrderRepository.java)

这就像显示的那样简单。`OrderRepository`接口扩展了`ReactiveCrudRepository`和我们的自定义仓库接口`OrderRepositoryExt`。

我们稍后会讨论`OrderRepositoryExt`；让我们先讨论`OrderRepository`。我们在`OrderRepository`接口中添加了一个额外的方法`findByCustomerId()`，通过给定的客户 ID 查找订单。`ReactiveCrudRepository`接口和`Query()`注解是 Spring Data R2DBC 库的一部分。`Query()`消耗原生 SQL 查询，与我们在上一章中创建的仓库不同。

我们也可以编写自己的自定义仓库。让我们为它编写一个简单的合约，如下所示：

```java
public interface OrderRepositoryExt {  Mono<OrderEntity> insert(Mono<NewOrder> m);
  Mono<OrderEntity> updateMapping(OrderEntity e);
}
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/repository/OrderRepositoryExt.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/repository/OrderRepositoryExt.java)

在这里，我们编写了两个方法签名——第一个将新订单记录插入数据库，第二个更新订单项和购物车项的映射。想法是，一旦下单，项目应该从购物车中移除并添加到订单中。如果您愿意，您也可以合并这两个操作。

让我们先定义`OrderRepositoryExtImpl`类，它扩展了`OrderRepositoryExt`接口，如下面的代码块所示：

```java
@Repositorypublic class OrderRepositoryExtImpl implements OrderRepositoryExt {
  private ConnectionFactory connectionFactory;
  private DatabaseClient dbClient;
  private ItemRepository itemRepo;
  private CartRepository cartRepo;
  private OrderItemRepository oiRepo;
  public OrderRepositoryExtImpl(ConnectionFactory
     connectionFactory, ItemRepository itemRepo,
     OrderItemRepository oiRepo, CartRepository
     cartRepo, DatabaseClient dbClient) {
    this.itemRepo = itemRepo;
    this.connectionFactory = connectionFactory;
    this.oiRepo = oiRepo;
    this.cartRepo = cartRepo;
    this.dbClient = dbClient;
  }
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/repository/OrderRepositoryExtImpl.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/repository/OrderRepositoryExtImpl.java)

我们刚刚定义了一些类属性，并在构造函数中将这些属性作为参数添加，用于基于构造函数的依赖注入。

根据合同，它接收`Mono<NewOrder>`。因此，我们需要在`OrderRepositoryExtImpl`类中添加一个将模型转换为实体的方法。我们还需要一个额外的参数，因为`CartEntity`包含了购物车中的商品。下面是代码：

OrderRepositoryExtImpl.java

```java
private OrderEntity toEntity(NewOrder order, CartEntity c) {  OrderEntity orderEntity = new OrderEntity();
  BeanUtils.copyProperties(order, orderEntity);
  orderEntity.setUserEntity(c.getUser());
  orderEntity.setCartId(c.getId());
  orderEntity.setItems(c.getItems())
     .setCustomerId(UUID.fromString(order.getCustomerId()))
     .setAddressId(UUID.fromString
       (order.getAddress().getId()))
     .setOrderDate(Timestamp.from(Instant.now()))
     .setTotal(c.getItems().stream()
     .collect(Collectors.toMap(k ->
       k.getProductId(), v ->
         BigDecimal.valueOf(v.getQuantity())
         .multiply(v.getPrice())))
     .values().stream().reduce(
      BigDecimal::add).orElse(BigDecimal.ZERO));
  return orderEntity;
}
```

这个方法很简单，除了设置总额的代码。总额是通过流计算的。让我们分解它以了解它：

1.  首先，它从`CartEntity`中获取项目。

1.  然后，它从项目中创建流。

1.  它创建了一个以产品 ID 为键，以数量和价格的乘积为值的映射。

1.  它从映射中获取值并将其转换为流。

1.  它通过向`BigDecimal`添加一个方法来执行 reduce 操作。然后给出总金额。

1.  如果没有值，它将简单地返回`0`。

在`toEntity()`方法之后，我们还需要另一个映射器，它从数据库中读取行并将它们转换为`OrderEntity`。为此，我们将编写`BiFunction`，它是`java.util.function`包的一部分：

OrderRepositoryExtImpl.java

```java
class OrderMapper implements BiFunction<Row,Object,  OrderEntity> {
  @Override
  public OrderEntity apply(Row row, Object o) {
    OrderEntity oe = new OrderEntity();
    return oe.setId(row.get("id", UUID.class))
        .setCustomerId(
            row.get("customer_id", UUID.class))
        .setAddressId(
            row.get("address_id", UUID.class))
        .setCardId(
            row.get("card_id", UUID.class))
        .setOrderDate(Timestamp.from(
            ZonedDateTime.of(
           (LocalDateTime) row.get("order_date"),
            ZoneId.of("Z")).toInstant()))
        .setTotal(
            row.get("total", BigDecimal.class))
        .setPaymentId(
            row.get("payment_id", UUID.class))
        .setShipmentId(
            row.get("shipment_id", UUID.class))
        .setStatus(StatusEnum.fromValue(
            row.get("status", String.class)));
  }
}
```

在这里，我们通过将行属性映射到`OrderEntity`来覆盖了`apply()`方法，它返回`OrderEntity`。`apply()`方法的第二个参数没有被使用，因为它包含我们不需要的元数据。

让我们先实现`OrderRepositoryExt`接口中的`updateMapping()`方法：

OrderRepositoryExtImpl.java

```java
public Mono<OrderEntity> updateMapping(OrderEntity  orderEntity) {
  return oiRepo.saveAll(orderEntity.getItems()
    .stream().map(i -> new OrderItemEntity()
      .setOrderId(orderEntity.getId())
      .setItemId(i.getId())).collect(toList()))
      .then(
        itemRepo.deleteCartItemJoinById(
           orderEntity.getItems().stream()
             .map(i -> i.getId())
             .collect(toList()),
           orderEntity.getCartId())
             .then(Mono.just(orderEntity))
      );
}
```

在这里，我们创建了一个响应式流的管道并执行了两个连续的数据库操作。首先，它使用`OrderItemRepository`创建订单项映射，然后使用`ItemRepository`删除购物车项映射。

在第一个操作中，Java 流用于创建`OrderItemEntity`实例的输入列表，在第二个操作中创建项目 ID 列表。

到目前为止，我们已经使用了`ReactiveCrudRepository`方法。让我们使用实体模板实现一个自定义方法，如下所示：

OrderRepositoryExtImpl.java

```java
@Overridepublic Mono<OrderEntity> insert(Mono<NewOrder> mdl) {
  AtomicReference<UUID> orderId =new AtomicReference<>();
  Mono<List<ItemEntity>> itemEntities =
       mdl.flatMap(m ->
          itemRepo.findByCustomerId(
            UUID.fromString(m.getCustomerId()))
          .collectList().cache());
  Mono<CartEntity> cartEntity =
       mdl.flatMap(m ->
          cartRepo.findByCustomerId(
            UUID.fromString(m.getCustomerId())))
         .cache();
  cartEntity = Mono.zip(cartEntity, itemEntities,
       (c, i) -> {
         if (i.size() < 1) {
          throw new ResourceNotFoundException(
          String.format("There is no item found
           in customer's (ID:%s) cart.",
             c.getUser().getId()));
       }
    return c.setItems(i);
  }).cache();
```

在这里，我们覆盖了`OrderRepositoryExt`接口中的`insert()`方法。`insert()`方法使用流畅的、函数式的和响应式的 API 填充。`insert()`方法接收一个包含创建新订单负载的`NewOrder`模型`Mono`实例作为参数。Spring Data R2DBC 不允许获取嵌套实体。然而，你可以像为`Order`编写的那样编写一个自定义的`Cart`存储库，它可以一起获取`Cart`及其项目。

我们正在使用`ReactiveCrudRepository`为`Cart`和`Item`实体。因此，我们逐个获取它们。首先，我们使用项目存储库根据给定的客户 ID 获取购物车项目。`Customer`与`Cart`有一个一对一的映射。然后，我们使用`CartRepository`通过客户 ID 获取`Cart`实体。

我们得到了两个单独的`Mono`对象 - `Mono<List<ItemEntity>>`和`Mono<CartEntity>`。现在，我们需要将它们组合起来。`Mono`有一个`zip()`操作符，它允许你取两个`Mono`对象，然后使用 Java 的`BiFunction`来合并它们。`zip()`仅在给定的两个`Mono`对象都产生项目时返回一个新的`Mono`对象。`zip()`是多态的，因此也有其他形式可用。

我们有了购物车及其项目，以及`NewOrder`负载。让我们将这些项目插入到数据库中，如下所示：

OrderRepositoryExtImpl.java

```java
R2dbcEntityTemplate template = new R2dbcEntityTemplate  (connectionFactory);
Mono<OrderEntity> orderEntity = Mono.zip(mdl,
   cartEntity, (m, c) -> toEntity(m, c)).cache();
return orderEntity.flatMap(oe -> dbClient.sql("""
    INSERT INTO ecomm.orders (address_id,
    card_id, customer_id, order_date, total,
    status) VALUES($1, $2, $3, $4, $5, $6)
    """)
    .bind("$1", Parameter.fromOrEmpty(
       oe.getAddressId(), UUID.class))
    .bind("$2", Parameter.fromOrEmpty(
       oe.getCardId(), UUID.class))
    .bind("$3", Parameter.fromOrEmpty(
       oe.getCustomerId(), UUID.class))
    .bind("$4",OffsetDateTime.ofInstant(
       oe.getOrderDate().toInstant(), ZoneId.of(
        "Z")).truncatedTo(ChronoUnit.MICROS))
    .bind("$5", oe.getTotal())
    .bind("$6", StatusEnum.CREATED.getValue())
      .map(new OrderMapper()::apply).one())
    .then(orderEntity.flatMap(x ->
       template.selectOne(
        query(where("customer_id").is(
           x.getCustomerId()).and("order_date")
            .greaterThanOrEquals(OffsetDateTime.
            ofInstant(x.getOrderDate().
              toInstant(),ZoneId.of("Z"))
              .truncatedTo(ChronoUnit.MICROS))),
        OrderEntity.class).map(t -> x.setId(
             t.getId()).setStatus(t.getStatus()))
    ));
```

在这里，我们再次使用`Mono.zip()`创建一个`OrderEntity`实例。现在，我们可以使用此实例的值插入到`orders`表中。

与数据库交互以运行 SQL 查询有两种方式——使用`DatabaseClient`或`R2dbcEntityTemplate`。现在，`DatabaseClient`是一个轻量级实现，它使用`sql()`方法直接处理 SQL，而`R2dbcEntityTemplate`提供了一个用于 CRUD 操作的流畅 API。我们已经使用了这两个类来展示它们的用法。

首先，我们使用`DatabaseClient.sql()`将新订单插入到`orders`表中。我们使用`OrderMapper`将数据库返回的行映射到实体。然后，我们使用`then()`反应性操作符选择新插入的记录，然后使用`R2dbcEntityTemplate.selectOne()`方法将其映射回`orderEntity`。

类似地，你可以为其他实体创建存储库。现在，我们可以在服务中使用这些存储库。让我们在下一小节中定义服务。

## 添加服务

让我们为`Order`添加一个服务。`OrderService`接口没有变化，如下所示。你只需要确保接口方法签名具有作为返回类型的反应性类型，以保持非阻塞流程：

```java
public interface OrderService {  Mono<OrderEntity> addOrder(@Valid Mono<NewOrder>
    newOrder);
  Mono<OrderEntity> updateMapping(@Valid OrderEntity
    orderEntity);
  Flux<OrderEntity> getOrdersByCustomerId(@NotNull @Valid
    String customerId);
  Mono<OrderEntity> getByOrderId(String id);
}
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/service/OrderService.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/service/OrderService.java)

接下来，你将实现`OrderService`中描述的每个方法。让我们首先以下这种方式实现`OrderService`的前两个方法：

```java
@Overridepublic Mono<OrderEntity> addOrder(@Valid Mono<NewOrder>
  newOrder) {
  return repository.insert(newOrder);
}
@Override
public Mono<OrderEntity> updateMapping(
  @Valid OrderEntity orderEntity) {
  return repository.updateMapping(orderEntity);
}
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/service/OrderServiceImpl.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/service/OrderServiceImpl.java)

前两个方法很简单；我们只是使用`OrderRepository`实例来调用相应的方法。在空闲场景中，重写的`updateMapping`方法将在更新映射后触发其余过程：

1.  启动支付。

1.  一旦支付被授权，将状态更改为`已支付`。

1.  启动运输并将状态更改为`运输启动`或`已运输`。

由于我们的应用程序不是一个真实世界的应用程序，我们是为了学习目的而编写的代码，所以我们没有编写执行所有三个步骤的代码。为了简单起见，我们只是更新映射。

让我们实现第三个方法（`getOrdersByCustomerId`）。这有点棘手，如下所示：

OrderServiceImpl.java

```java
private BiFunction<OrderEntity, List<ItemEntity>, OrderEntity> biOrderItems = (o, fi) -> o    .setItems(fi);
@Override
public Flux<OrderEntity> getOrdersByCustomerId(
   String customerId) {
 return repository.findByCustomerId(UUID
  .fromString(customerId)).flatMap(order ->
   Mono.just(order)
    .zipWith(userRepo.findById(order.getCustomerId()))
    .map(t -> t.getT1().setUserEntity(t.getT2()))
    .zipWith(addRepo.findById(order.getAddressId()))
    .map(t ->
          t.getT1().setAddressEntity(t.getT2()))
    .zipWith(cardRepo.findById(
       order.getCardId() != null
       ? order.getCardId() : UUID.fromString(
         "0a59ba9f-629e-4445-8129-b9bce1985d6a"))
              .defaultIfEmpty(new CardEntity()))
    .map(t -> t.getT1().setCardEntity(t.getT2()))
    .zipWith(itemRepo.findByCustomerId(
        order.getCustomerId()).collectList(),biOrderItems)
  );
}
```

前一个方法看起来很复杂，但实际上并不复杂。你在这里做的基本上是从多个仓库中获取数据，然后使用`zipWith()`操作符，通过与之并用的`map()`操作符或作为单独参数的`BiFunction`来填充`OrderEntity`内部的嵌套实体。

前一个方法首先使用客户 ID 获取订单，然后使用`flatMap()`操作符将订单扁平化以填充其嵌套实体，如`Customer`、`Order`和`Items`。因此，我们在`flatMap()`操作符内部使用`zipWith()`。如果你观察第一个`zipWith()`，它会获取用户实体，然后使用`map()`操作符设置嵌套用户实体的属性。同样，其他嵌套实体也被填充。

在最后一个`zipWith()`操作符中，我们使用`BiFunction` `biOrderItems`来设置`OrderEntity`实例中的`item`实体。

实现接口`OrderService`的最后一个方法（`getOrderById`）使用了相同的算法，如下面的代码所示：

OrderServiceImpl.java

```java
@Overridepublic Mono<OrderEntity> getByOrderId(String id) {
  return repository.findById(UUID.fromString(id))
   .flatMap(order ->
     Mono.just(order)
      .zipWith(userRepo.findById(order.getCustomerId()))
      .map(t -> t.getT1().setUserEntity(t.getT2()))
      .zipWith(addRepo.findById(order.getAddressId()))
      .map(t -> t.getT1().setAddressEntity(t.getT2()))
      .zipWith(cardRepo.findById(order.getCardId()))
      .map(t -> t.getT1().setCardEntity(t.getT2()))
      .zipWith(itemRepo.findByCustomerId
         (order.getCustomerId()).collectList()
            ,biOrderItems)
  );
}
```

到目前为止，你已经使用了`zipWith()`操作符来合并不同的对象。你可能发现另一种使用`Mono.zip()`操作符合并两个`Mono`实例的方法，如下所示：

```java
private BiFunction<CartEntity, List<ItemEntity>, CartEntity> cartItemBiFun = (c, i) -> c    .setItems(i);
@Override
public Mono<CartEntity> getCartByCustomerId(String
  customerId) {
  Mono<CartEntity> cart = repository.findByCustomerId(
     UUID.fromString(customerId))
      .subscribeOn(Schedulers.boundedElastic());
  Mono<UserEntity> user = userRepo.findById(
     UUID.fromString(customerId))
      .subscribeOn(Schedulers.boundedElastic());
  cart = Mono.zip(cart, user, cartUserBiFun);
  Flux<ItemEntity> items =
      itemRepo.findByCustomerId(
         UUID.fromString(customerId))
      .subscribeOn(Schedulers.boundedElastic());
  return Mono.zip(cart, items.collectList(),
     cartItemBiFun);
}
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/service/CardServiceImpl.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/service/CardServiceImpl.java)

这个例子是从`CartServiceImpl`类中提取的。在这里，我们进行了两次独立的调用——一次使用`cart`仓库，另一次使用`item`仓库。结果，这两个调用产生了两个`Mono`实例，并使用`Mono.zip()`操作符将它们合并。我们直接使用`Mono`来调用这个操作；上一个例子是在`Mono`/`Flux`实例上使用`zipWith()`操作符。

使用类似的技术，创建了剩余的服务。你可以在[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter05/src/main/java/com/packt/modern/api/service`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter05/src/main/java/com/packt/modern/api/service)找到它们。

你已经实现了允许你执行异步操作（包括数据库调用）的异步服务。现在，你可以在控制器中消费这些服务类。让我们将我们的重点转移到我们的反应式 API 实现开发的最后一个子部分（控制器）。

## 添加控制器实现

REST 控制器接口已经由 OpenAPI Codegen 工具生成。我们现在可以创建这些接口的实现。在实现反应式控制器时，唯一的不同之处在于需要具有反应式管道来调用服务和组装器。您还应该仅根据生成的契约返回封装在`Mono`或`Flux`中的`ResponseEntity`对象。

让我们实现`OrderApi`，这是`Orders` REST API 的控制接口：

```java
@RestControllerpublic class OrderController implements OrderApi {
  private final OrderRepresentationModelAssembler
    assembler;
  private OrderService service;
  public OrderController(OrderService service,
   OrderRepresentationModelAssembler assembler) {
     this.service = service;
     this.assembler = assembler;
  }
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/controller/OrderController.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/controller/OrderController.java)

在这里，`@RestController`是一个结合了`@Controller`和`@ResponseBody`的技巧。这些是我们用于在*第四章*，“为 API 编写业务逻辑”中创建 REST 控制器所使用的相同注解。然而，现在方法有不同的签名，以便应用反应式管道。确保您不要打破调用链的反应式性或添加任何阻塞调用。如果您这样做，要么 REST 调用将不会完全非阻塞，或者您可能会看到不期望的结果。

我们使用基于构造函数的依赖注入来注入订单服务和组装器。让我们添加方法实现：

OrderController.java

```java
@Overridepublic Mono<ResponseEntity<Order>>
   addOrder(@Valid Mono<NewOrder> newOrder,
      ServerWebExchange exchange) {
  return service.addOrder(newOrder.cache())
    .zipWhen(x -> service.updateMapping(x))
    .map(t -> status(HttpStatus.CREATED)
      .body(assembler.entityToModel(
         t.getT2(), exchange)))
    .defaultIfEmpty(notFound().build());
}
```

方法的参数和返回类型都是反应式类型（`Mono`），用作包装器。反应式控制器还有一个额外的参数，`ServerWebExchange`，我们之前讨论过。

在这个方法中，我们简单地将`newOrder`实例传递给服务。我们使用了`cache()`，因为我们需要多次订阅它。我们通过`addOrder()`调用获取新创建的`EntityOrder`。然后，我们使用`zipWhen()`运算符，它使用新创建的订单实体执行`updateMapping`操作。最后，我们通过将`Order`对象封装在`ResponseEntity`中发送它。如果返回一个空实例，它还会返回`NOT FOUND 404`。

让我们看看`order` API 接口的其他方法实现：

OrderController.java

```java
@Overridepublic Mono<ResponseEntity<Flux<Order>>>
   getOrdersByCustomerId(@NotNull
     @Valid String customerId, ServerWebExchange
       exchange) {
  return Mono
     .just(ok(assembler.toListModel(service
       .getOrdersByCustomerId(customerId),
         exchange)));
}
@Override
public Mono<ResponseEntity<Order>>
   getByOrderId(String id, ServerWebExchange
     exchange) {
  return service.getByOrderId(id).map(o ->
     assembler.entityToModel(o, exchange))
      .map(ResponseEntity::ok)
      .defaultIfEmpty(notFound().build());
}
```

在前面的代码中，两个方法在本质上有些相似；服务根据给定的客户 ID 和订单 ID 分别返回`OrderEntity`。然后它被转换成模型，并封装在`ResponseEntity`和`Mono`中。

类似地，其他 REST 控制器也是使用相同的方法实现的。您可以在[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter05/src/main/java/com/packt/modern/api/controller`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter05/src/main/java/com/packt/modern/api/controller)找到它们的其余部分。

我们几乎完成了反应式 API 的实现。让我们看看一些其他的细微变化。

## 将 H2 控制台添加到应用程序中

H2 控制台应用程序在 Spring WebFlux 中默认不可用，就像它在 Spring MVC 中可用一样。然而，你可以通过定义自己的 bean 来添加它，如下所示：

```java
@Componentpublic class H2ConsoleComponent {
    private Server webServer;
    @Value("${modern.api.h2.console.port:8081}")
    Integer h2ConsolePort;
    @EventListener(ContextRefreshedEvent.class)
    public void start()
       throws java.sql.SQLException {
      this.webServer = org.h2.tools.Server
         .createWebServer("-webPort",
           h2ConsolePort.toString(), "-
             tcpAllowOthers").start();
    }
    @EventListener(ContextClosedEvent.class)
    public void stop() {
      this.webServer.stop();
    }
}
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/H2ConsoleComponent.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/java/com/packt/modern/api/H2ConsoleComponent.java)

之前的代码（`H2ConsoleComponent`）很简单；我们添加了`start()`和`stop()`方法，它们分别在`ContextRefreshEvent`和`ContextStopEvent`上执行。`ContextRefreshEvent`是一个应用程序事件，当`ApplicationContext`刷新或初始化时被触发。`ContextStopEvent`也是一个应用程序事件，当`ApplicationContext`关闭时被触发。

`start()`方法使用 H2 库创建网络服务器，并在指定的端口上启动它。`stop()`方法停止 H2 网络服务器，即 H2 控制台应用程序。

你需要一个不同的端口来执行 H2 控制台，这可以通过将`modern.api.h2.console.port=8081`属性添加到`application.properties`文件中来配置。`h2ConsolePort`属性被注解为`@Value("${modern.api.h2.console.port:8081}")`；因此，当 Spring 框架初始化`H2ConsoleComponent`bean 时，将选择并分配`application.properties`中配置的值到`h2ConsolePort`。如果`application.properties`文件中没有定义该属性，则分配的值将是`8081`。

由于我们正在讨论`application.properties`，让我们看看一些其他的更改。

## 添加应用程序配置

我们将使用 Flyway 进行数据库迁移。让我们添加所需的配置：

```java
spring.flyway.url=jdbc:h2:file:./data/ecomm;AUTO_SERVER=TRUE;DB_CLOSE_DELAY=-1;IGNORECASE=TRUE;DATABASE_TO_UPPER=FALSEspring.flyway.schemas=ecomm
spring.flyway.user=
spring.flyway.password=
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/resources/application.properties`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/src/main/resources/application.properties)

你可能想知道为什么我们在这里使用 JDBC，而不是 R2DBC。这是因为 Flyway 还没有开始支持 R2DBC（在撰写本文时）。一旦添加了支持，你可以将其更改为 R2DBC。

我们已经指定了`ecomm`模式，并设置了一个空的用户名和密码。

同样，你可以在`application.properties`文件中添加 Spring Data 配置：

```java
spring.r2dbc.url=r2dbc:h2:file://././data/ecomm?options=AUTO_SERVER=TRUE;DB_CLOSE_DELAY=-1;IGNORECASE=TRUE;DATABASE_TO_UPPER=FALSE;;TRUNCATE_LARGE_LENGTH=TRUE;DB_CLOSE_ON_EXIT=FALSEspring.r2dbc.driver=io.r2dbc:r2dbc-h2
spring.r2dbc.name=
spring.r2dbc.password=
```

Spring Data 支持 R2DBC；因此，我们使用基于 R2DBC 的 URL。我们将`io.r2dbc:r2dbc-h2`设置为 H2 的驱动程序，并设置了一个空的用户名和密码。

同样，我们已经将以下日志属性添加到`logback-spring.xml`中，以向控制台添加 Spring R2DBC 和 H2 的调试语句：

```java
<logger name="org.springframework.r2dbc"      level="debug" additivity="false">
   <appender-ref ref="STDOUT"/>
</logger>
<logger name="reactor.core" level="debug"
   additivity="false">
   <appender-ref ref="STDOUT"/>
</logger>
<logger name="io.r2dbc.h2" level="debug"
    additivity="false">
   <appender-ref ref="STDOUT"/>
</logger>
```

这标志着我们反应式 RESTful API 实现的结束。现在，您可以测试它们了。

## 测试反应式 API

现在，您一定期待着测试。您可以在以下位置找到 API 客户端集合。您可以导入它，然后使用支持 HAR 类型文件导入的任何 API 客户端来测试 API。

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/Chapter05-API-Collection.har`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter05/Chapter05-API-Collection.har)

构建 05 章代码并运行

您可以通过在项目的根目录下运行`gradlew clean build`来构建代码，并使用`java -jar build/libs/Chapter05-0.0.1-SNAPSHOT.jar`来运行服务。请确保在路径中使用 Java 17。

# 概述

我希望您喜欢使用异步、非阻塞和函数式范式学习反应式 API 开发。乍一看，如果您不太熟悉流畅和函数式范式，可能会觉得它很复杂，但通过实践，您将开始只编写函数式风格的代码。当然，熟悉 Java 流和函数将帮助您轻松掌握这些概念。

现在您已经到达本章的结尾，您已经拥有了编写函数式和反应式代码的技能。您可以编写反应式、异步和非阻塞的代码以及 REST API。您还了解了 R2DBC，只要继续使用反应式编程，它将在未来变得更加稳固和增强。

在下一章中，我们将探讨 RESTful 服务开发的安全性方面。

# 问题

1.  您真的需要反应式范式来进行应用程序开发吗？

1.  使用反应式范式有什么缺点吗？

1.  在 Spring WebFlux 中，对于 HTTP 请求的情况，谁扮演订阅者的角色？

# 答案

1.  是的，只有在您需要垂直扩展时才需要。在云中，您为使用资源付费，反应式应用程序帮助您优化资源使用。这是一种实现扩展的新方法。与无反应式应用程序相比，您需要的线程数量较少。连接到数据库、I/O 或任何外部源的成本是一个回调；因此，基于反应式应用程序不需要太多内存。然而，尽管在垂直扩展方面反应式编程更优越，您应该继续使用现有的或非反应式应用程序。甚至 Spring 也建议这样做。没有新或旧的风格；两者可以共存。然而，当您需要扩展任何特殊组件或应用程序时，您可以选择反应式方法。几年前，Netflix 用反应式的 Zuul2 API 网关替换了 Zuul API 网关。这帮助他们实现了扩展。然而，他们仍然/使用非反应式应用程序。

1.  任何事物都有其利弊。反应式编程也不例外。与命令式风格相比，反应式代码编写起来并不容易。由于它不使用单个线程，因此调试起来非常困难。然而，如果你有精通反应式范式的开发者，这并不是一个问题。

1.  WebFlux 内部类订阅由控制器发送的`Mono`/`Flux`流，并将它们转换为 HTTP 数据包。HTTP 协议确实支持事件流。然而，对于其他媒体类型，如 JSON，Spring WebFlux 订阅`Mono`/`Flux`流，并等待`onComplete()`或`onError()`被触发。然后，它将整个元素列表序列化，或者对于`Mono`，将单个元素序列化到一个 HTTP 响应中。你可以在*反应式* *核心*部分了解更多相关信息。

# 进一步阅读

+   Project Reactor：[`projectreactor.io`](https://projectreactor.io)

+   Spring Reactive 文档：[`docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html`](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html)

+   Spring Data R2DBC – 参考文档：[`docs.spring.io/spring-data/r2dbc/docs/current/reference/html/`](https://docs.spring.io/spring-data/r2dbc/docs/current/reference/html/)

+   *《Spring 5 实战反应式编程》* 第 5 版：[`www.packtpub.com/product/hands-on-reactive-programming-in-spring-5/9781787284951`](https://www.packtpub.com/product/hands-on-reactive-programming-in-spring-5/9781787284951)

+   *《Java 17 编程学习——第二版》*: [`www.packtpub.com/product/learn-java-17-programming-second-edition/9781803241432`](https://www.packtpub.com/product/learn-java-17-programming-second-edition/9781803241432)

# 第二部分 – 安全性、UI、测试和部署

在这部分，你将学习如何使用 JWT 和 Spring Security 保护 REST API。完成这部分后，你还将能够根据用户角色授权 REST 端点。你将了解 UI 应用如何消费 API，以及如何自动化 API 的单元测试和集成测试。到这部分结束时，你将能够将构建的应用容器化，然后在 Kubernetes 集群中部署它。

这一部分包含以下章节：

+   *第六章*, *使用授权和认证保护 REST 端点*

+   *第七章*, *设计用户界面*

+   *第八章*, *测试 API*

+   *第九章*, *Web 服务的部署*
