# 第三章：Spring 中的 Flux 和 Mono（Reactor 支持）

在本章中，我们将向读者介绍更多在 Spring 5 中支持 Reactor 的实际方法，包括 Flux 和 Mono。用户将通过简单的 JSON 结果亲身体验 Flux 和 Mono。

本章将涵盖以下主题：

+   Reactive 编程和好处

+   Reactive Core 和 Streams

+   Spring REST 中的 Flux 和 Mono

+   使用 Reactive 的用户类——REST

# Reactive 编程的好处

假设我们的应用程序中有一百万个用户交易正在进行。明年，这个数字将增加到 1000 万，所以我们需要进行扩展。传统的方法是添加足够的服务器（水平扩展）。

如果我们不进行水平扩展，而是选择使用相同的服务器进行扩展，会怎么样？是的，Reactive 编程将帮助我们做到这一点。Reactive 编程是关于非阻塞的、同步的、事件驱动的应用程序，不需要大量线程进行垂直扩展（在 JVM 内部），而不是水平扩展（通过集群）。

Reactive 类型并不是为了更快地处理请求。然而，它们更关注请求并发性，特别是有效地从远程服务器请求数据。通过 Reactive 类型的支持，您将获得更高质量的服务。与传统处理相比，传统处理在等待结果时会阻塞当前线程，而 Reactive API 仅请求可以消耗的数据量。Reactive API 处理数据流，而不仅仅是单个元素。

总的来说，Reactive 编程是关于非阻塞、事件驱动的应用程序，可以通过少量线程进行扩展，背压是确保生产者（发射器）不会压倒消费者（接收器）的主要组成部分。

# Reactive Core 和 Streams

Java 8 引入了 Reactive Core，它实现了 Reactive 编程模型，并建立在 Reactive Streams 规范之上，这是构建 Reactive 应用程序的标准。由于 lambda 语法为事件驱动方法提供了更大的灵活性，Java 8 提供了支持 Reactive 的最佳方式。此外，Java 的 lambda 语法使我们能够创建和启动小型和独立的异步任务。Reactive Streams 的主要目标之一是解决背压问题。我们将在本章的后面部分更多地讨论背压问题。

Java 8 Streams 和 Reactive Streams 之间的主要区别在于 Reactive 是推模型，而 Java 8 Streams 侧重于拉模型。在 Reactive Streams 中，根据消费者的需求和数量，所有事件都将被推送给消费者。

自上次发布以来，Spring 5 对 Reactive 编程模型的支持是其最佳特性。此外，借助 Akka 和 Play 框架的支持，Java 8 为 Reactive 应用程序提供了更好的平台。

Reactor 是建立在 Reactive Streams 规范之上的。Reactive Streams 是四个 Java 接口的捆绑包：

+   `Publisher`

+   `Subscriber`

+   `Subscription`

+   `Processor`

`Publisher`将数据项的流发布给注册在`Publisher`上的订阅者。使用执行器，`Publisher`将项目发布给`Subscriber`。此外，`Publisher`确保每个订阅的`Subscriber`方法调用严格有序。

`Subscriber`只有在请求时才消耗项目。您可以通过使用`Subscription`随时取消接收过程。

`Subscription`充当`Publisher`和`Subscriber`之间的消息中介。

`Processor`代表一个处理阶段，可以包括`Subscriber`和`Publisher`。`Processor`可以引发背压并取消订阅。

Reactive Streams 是用于异步流处理的规范，这意味着所有事件都可以异步产生和消费。

# 背压和 Reactive Streams

反压是一种机制，授权接收器定义它希望从发射器（数据提供者）获取多少数据。响应式流的主要目标是处理反压。它允许：

+   在数据准备好被处理后，控制转到接收器以获取数据

+   定义和控制要接收的数据量

+   高效处理慢发射器/快接收器或快发射器/慢接收器的情况

# WebFlux

截至 2017 年 9 月，Spring 宣布了 5 的一般可用性。Spring 5 引入了一个名为 Spring WebFlux 的响应式 Web 框架。这是一个非阻塞的 Web 框架，使用 Reactor 来支持 Reactive Streams API。

传统上，阻塞线程会消耗资源，因此需要非阻塞异步编程来发挥更好的作用。Spring 技术团队引入了非阻塞异步编程模型，以处理大量并发请求，特别是对延迟敏感的工作负载。这个概念主要用于移动应用程序和微服务。此外，这个 WebFlux 将是处理许多客户端和不均匀工作负载的最佳解决方案。

# 基本 REST API

要理解 Flux 和 Mono 等响应式组件的实际部分，我们将不得不创建自己的 REST API，并开始在 API 中实现 Flux 和 Mono 类。在本章中，我们将构建一个简单的 REST Web 服务，返回`Aloha`。在进入实现部分之前，我们将专注于创建 RESTful Web 服务所涉及的组件。

在本节中，我们将涵盖以下主题：

+   Flux 和 Mono - Spring 5 的介绍：功能性 Web 框架组件

+   Flux 和 Mono - 在 REST API 中

# Flux

Flux 是 Reactor 中的主要类型之一。Flux 相当于 RxJava 的 Observable，能够发出零个或多个项目，然后选择性地完成或失败。

Flux 是实现了 Reactive Streams 宣言中的`Publisher`接口的 Reactive 类型之一。Flux 的主要作用是处理数据流。Flux 主要表示*N*个元素的流。

Flux 是一个发布者，特定**普通旧 Java 对象**（**POJO**）类型的事件序列。

# Mono

Mono 是 Reactor 的另一种类型，最多只能发出一个项目。只想要发出完成信号的异步任务可以使用 Mono。Mono 主要处理一个元素的流，而不是 Flux 的*N*个元素。

Flux 和 Mono 都利用这种语义，在使用一些操作时强制转换为相关类型。例如，将两个 Monos 连接在一起将产生一个 Flux；另一方面，在`Flux<T>`上调用`single()`将返回一个`Mono <T>`。

Flux 和 Mono 都是**Reactive Streams**（**RS**）发布者实现，并符合 Reactive-pull 反压。

Mono 在特定场景中使用，比如只产生一个响应的 HTTP 请求。在这种情况下，使用 Mono 将是正确的选择。

返回`Mono<HttpResponse>`来处理 HTTP 请求，就像前面提到的情况一样，比返回`Flux<HttpResponse>`更好，因为它只提供与零个或一个项目的上下文相关的操作符。

Mono 可以用来表示没有值的异步过程，只有完成的概念。

# 具有 Reactive 的 User 类 - REST

在第一章中，我们介绍了`Ticket`和`User`，这两个类与我们的 Web 服务有关。由于`Ticket`类与`User`类相比有点复杂，我们将使用`User`类来理解响应式组件。

由于 Spring 5 中的响应式还不是完全稳定的，我们只会在几章中讨论响应式。因此，我们将为基于响应式的 REST API 创建一个单独的包。此外，我们将在现有的`pom.xml`文件中添加基于响应式的依赖项。

首先，我们将不得不添加所有的响应式依赖。在这里，我们将在现有的`pom.xml`文件中添加代码：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project  
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">  
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.packtpub.restapp</groupId>
  <artifactId>ticket-management</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>
  <name>ticket-management</name>
  <description>Demo project for Spring Boot</description>  
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
</properties>
<dependencyManagement>
   <dependencies>
    <dependency>
      <groupId>io.projectreactor</groupId>
      <artifactId>reactor-bom</artifactId>
      <version>Bismuth-RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
    </dependency>
        </dependencies>
  </dependencyManagement>
  <dependencies>
      <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>5.0.1.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
      <version>1.5.7.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      <version>1.5.7.RELEASE</version>
    </dependency>  
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.9.2</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>5.0.0.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.0.1.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
      <version>1.5.7.RELEASE</version> 
    </dependency>     
    <dependency>
      <groupId>org.reactivestreams</groupId>
      <artifactId>reactive-streams</artifactId>
    </dependency>
    <dependency>
      <groupId>io.projectreactor</groupId>
      <artifactId>reactor-core</artifactId>
    </dependency>
    <dependency>
      <groupId>io.projectreactor.ipc</groupId>
      <artifactId>reactor-netty</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.tomcat.embed</groupId>
      <artifactId>tomcat-embed-core</artifactId>
      <version>8.5.4</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.0.0.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webflux</artifactId>
      <version>5.0.0.RELEASE</version>
    </dependency>
  </dependencies>
  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

对于与 Reactive 相关的工作，您可以使用现有项目，也可以创建一个新项目，以避免与非 Reactive（普通）REST API 发生冲突。您可以使用[`start.spring.io`](https://start.spring.io)获取基本项目，然后使用上述配置更新 Maven 文件。

在前面的 POM 配置中，我们已经在现有的依赖项上添加了 Reactor 依赖项（如下所示）：

+   `reactive-streams`

+   `reactor-core`

+   `reactor-netty`

+   `tomcat-embed-core`

+   `spring-webflux`

这些是使用 Reactor 所需的库。

`User`类的组件如下：

+   `userid`

+   `username`

+   `user_email`

+   `user_type`（管理员，普通用户，CSR）

在这里，我们使用了`User`类的四个变量。为了更容易理解 Reactive 组件，我们只使用了两个变量（`userid`，`username`）。让我们创建一个只有`userid`和`username`的 POJO 类。

`User` POJO 类如下：

```java
package com.packtpub.reactive;
public class User {
  private Integer userid;
  private String username;  
  public User(Integer userid, String username){
    this.userid = userid;
    this.username = username;
  }
  public Integer getUserid() {
    return userid;
  }
  public void setUserid(Integer userid) {
    this.userid = userid;
  }
  public String getUsername() {
    return username;
  }
  public void setUsername(String username) {
    this.username = username;
  } 
}
```

在上面的类中，我使用了两个变量和一个构造函数来在实例化时填充变量。同时，使用 getter/setter 来访问这些变量。

让我们为`User`类创建一个 Reactive 存储库：

```java
package com.packtpub.reactive;
import reactor.core.publisher.Flux;
public interface UserRepository {
  Flux<User> getAllUsers();
}
```

在上面的代码中，我们为`User`引入了一个 Reactive 存储库和一个只有一个方法的类，名为`getAllUsers`。通过使用这个方法，我们应该能够检索到用户列表。现在先不谈 Flux，因为它将在以后讨论。

您可以看到这个`UserRepository`是一个接口。我们需要有一个具体的类来实现这个接口，以便使用这个存储库。让我们为这个 Reactive 存储库创建一个具体的类：

```java
package com.packtpub.reactive;
import java.util.HashMap;
import java.util.Map;
import reactor.core.publisher.Flux;
public class UserRepositorySample implements UserRepository {  
  // initiate Users
  private Map<Integer, User> users = null;  
  // fill dummy values for testing
  public UserRepositorySample() {
    // Java 9 Immutable map used
    users = Map.of(
      1, (new User(1, "David")),
      2, (new User(2, "John")),
      3, (new User(3, "Kevin"))
    ); 
  }
  // this method will return all users
  @Override
  public Flux<User> getAllUsers() {
    return Flux.fromIterable(this.users.values());
  }
}
```

由于 Java 9 中有不可变映射可用，我们可以在我们的代码中使用不可变映射。然而，这些不可变对象仅适用于本章，因为我们不对现有条目进行任何更新。

在下一章中，我们将使用常规的映射，因为我们需要在 CRUD 操作中对它们进行编辑。

目前，我们能够从具体类中获取用户列表。现在我们需要一个 web 处理程序在控制器中检索用户。现在让我们创建一个处理程序：

```java
package com.packtpub.reactive;
import org.springframework.web.reactive.function.server.ServerRequest;
import org.springframework.web.reactive.function.server.ServerResponse;
import static org.springframework.http.MediaType.APPLICATION_JSON;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;
public class UserHandler {
  private final UserRepository userRepository;  
  public UserHandler(UserRepository userRepository){
    this.userRepository = userRepository;
  }  
  public Mono<ServerResponse> getAllUsers(ServerRequest request){
    Flux<User> users = this.userRepository.getAllUsers();
    return ServerResponse.ok().contentType(APPLICATION_JSON).body(users, User.class); 
  }
}
```

最后，我们将需要创建一个服务器来保留 REST API。在下面的代码中，我们的`Server`类将创建一个 REST API 来获取用户：

```java
package com.packtpub.reactive;
import static org.springframework.http.MediaType.APPLICATION_JSON;
import static org.springframework.web.reactive.function.server.RequestPredicates.GET;
import static org.springframework.web.reactive.function.server.RequestPredicates.POST;
import static org.springframework.web.reactive.function.server.RequestPredicates.accept;
import static org.springframework.web.reactive.function.server.RequestPredicates.contentType;
import static org.springframework.web.reactive.function.server.RequestPredicates.method;
import static org.springframework.web.reactive.function.server.RequestPredicates.path;
import static org.springframework.web.reactive.function.server.RouterFunctions.nest;
import static org.springframework.web.reactive.function.server.RouterFunctions.route;
import static org.springframework.web.reactive.function.server.RouterFunctions.toHttpHandler;
import java.io.IOException;
import org.springframework.http.HttpMethod;
import org.springframework.http.server.reactive.HttpHandler;
import org.springframework.http.server.reactive.ReactorHttpHandlerAdapter;
import org.springframework.web.reactive.function.server.RouterFunction;
import org.springframework.web.reactive.function.server.ServerResponse;
import reactor.ipc.netty.http.server.HttpServer;
public class Server {
  public static final String HOST = "localhost";
  public static final int PORT = 8081;
  public static void main(String[] args) throws InterruptedException, IOException{
    Server server = new Server(); 
    server.startReactorServer();
    System.out.println("Press ENTER to exit.");
    System.in.read();
  }  
  public void startReactorServer() throws InterruptedException {
    RouterFunction<ServerResponse> route = routingFunction();
    HttpHandler httpHandler = toHttpHandler(route);
    ReactorHttpHandlerAdapter adapter = new ReactorHttpHandlerAdapter(httpHandler);
    HttpServer server = HttpServer.create(HOST, PORT);
    server.newHandler(adapter).block();
  }
  public RouterFunction<ServerResponse> routingFunction() {
    UserRepository repository = new UserRepositorySample();
    UserHandler handler = new UserHandler(repository);
    return nest (
        path("/user"),
        nest(
          accept(APPLICATION_JSON),
          route(GET("/{id}"), handler::getAllUsers)
          .andRoute(method(HttpMethod.GET), handler::getAllUsers)
        ).andRoute(POST("/").and(contentType(APPLICATION_JSON)), handler::getAllUsers));
  }
}
```

我们将在接下来的章节中更多地讨论我们是如何做到这一点的。只要确保您能够理解代码是如何工作的，并且可以通过访问 API 在浏览器上看到输出。

运行`Server.class`，您将看到日志：

```java
Press ENTER to exit.
```

现在您可以在浏览器/SoapUI/Postman 或任何其他客户端访问 API：

```java
http://localhost:8081/user/
```

由于我们在 Reactive 服务器中使用了`8081`端口，我们只能访问`8081`而不是`8080`：

```java
[ 
  { 
    "userid": 100, 
    "username": "David" 
  },
  { 
    "userid": 101, 
    "username": "John" 
  },
  { 
    "userid": 102, 
    "username": "Kevin" 
  }, 
]
```

# 总结

到目前为止，我们已经看到如何设置 Maven 构建来支持我们的基本 Web 服务实现。此外，我们还学习了 Maven 在第三方库管理以及 Spring Boot 和基本 Spring REST 项目中的帮助。在接下来的章节中，我们将更多地讨论 Spring REST 端点和 Reactor 支持。
