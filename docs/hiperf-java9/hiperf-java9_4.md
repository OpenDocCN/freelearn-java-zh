# 第四章：微服务

只要我们一直在谈论一个过程的设计、实施和调优，我们就能够用生动的形象（尽管只存在于我们的想象中）来说明它，比如金字塔建筑。基于平等原则的多线程管理，也具有集中规划和监督的意义。不同的优先级是根据程序员经过深思熟虑后根据预期负载进行编程分配的，并在监控后进行调整。可用资源的上限是固定的，尽管在一个相对较大的集中决策后可以增加。

这些系统取得了巨大的成功，仍然构成当前部署到生产环境的大多数 Web 应用程序。其中许多是单体应用，封装在一个`.ear`或`.war`文件中。对于相对较小的应用程序和相应的团队规模来说，这样做效果很好。它们易于（如果代码结构良好）维护、构建，并且如果生产负载不是很高，它们可以很容易地部署。如果业务不增长或对公司的互联网存在影响不大，它们将继续发挥作用，可能在可预见的未来也是如此。许多服务提供商急于通过收取少量费用来托管这样的网站，并解除网站所有者与业务无直接关系的生产维护的技术烦恼。但并非所有情况都是如此。

负载越高，扩展就会变得越困难和昂贵，除非代码和整体架构进行重构，以使其更灵活和能够承受不断增长的负载。本课程描述了许多行业领袖在解决这个问题时采取的解决方案以及背后的动机。

我们将在本课程中讨论微服务的特定方面，包括以下内容：

+   微服务兴起的动机

+   最近开发的支持微服务的框架

+   微服务开发过程，包括实际示例，以及在微服务构建过程中的考虑和决策过程

+   无容器、自包含和容器内三种主要部署方法的优缺点

# 为什么要使用微服务？

一些企业由于需要跟上更大量的流量，对部署计划有更高的需求。对这一挑战的自然回答是并且已经是将具有相同`.ear`或`.war`文件部署的服务器加入到集群中。因此，一个失败的服务器可以自动被集群中的另一个服务器替换，用户不会感受到服务中断。支持所有集群服务器的数据库也可以进行集群化。连接到每个集群的连接都通过负载均衡器，确保没有一个集群成员比其他成员工作更多。

Web 服务器和数据库集群有所帮助，但只能在一定程度上，因为随着代码库的增长，其结构可能会产生一个或多个瓶颈，除非采用可扩展的设计来解决这些问题。其中一种方法是将代码分成层：前端（或 Web 层）、中间层（或应用层）和后端（或后端层）。然后，每个层都可以独立部署（如果层之间的协议没有改变），并在自己的服务器集群中，因为每个层都可以根据需要独立地水平扩展。这样的解决方案提供了更多的扩展灵活性，但使部署计划更加复杂，特别是如果新代码引入了破坏性变化。其中一种方法是创建一个将托管新代码的第二个集群，然后逐个从旧集群中取出服务器，部署新代码，并将它们放入新集群。只要每个层中至少有一个服务器具有新代码，新集群就会启动。这种方法对 Web 和应用层效果很好，但对后端来说更复杂，因为偶尔需要数据迁移和类似的愉快练习。加上部署过程中由人为错误、代码缺陷、纯粹的意外或所有前述因素的组合引起的意外中断，很容易理解为什么很少有人喜欢将主要版本发布到生产环境。

程序员天生是问题解决者，他们尽力防止早期的情景，通过编写防御性代码、弃用而不是更改、测试等。其中一种方法是将应用程序分解为更独立部署的部分，希望避免同时部署所有内容。他们称这些独立单元为**服务**，**面向服务的架构**（**SOA**）应运而生。

很不幸，在许多公司中，代码库的自然增长没有及时调整到新的挑战。就像那只最终在慢慢加热的水壶里被煮熟的青蛙一样，他们从来没有时间通过改变设计来跳出热点。向现有功能的一团泥中添加另一个功能总是比重新设计整个应用程序更便宜。时间到市场和保持盈利始终是决策的主要标准，直到结构不良的源代码最终停止工作，将所有业务交易一并拖垮，或者，如果公司幸运的话，让他们度过风暴并显示重新设计的重要性。

由此产生的结果是，一些幸运的公司仍然在经营中，他们的单片应用程序仍然如预期般运行（也许不久，但谁知道），一些公司倒闭，一些从错误中吸取教训，进入新挑战的勇敢世界，另一些则从错误中吸取教训，并从一开始就设计他们的系统为 SOA。

有趣的是观察社会领域中类似的趋势。社会从强大的中央集权政府转向更松散耦合的半独立国家联盟，通过相互有利的经济和文化交流联系在一起。

不幸的是，维护这样一种松散的结构是有代价的。每个参与者都必须在维护合同（社会上的社会合同，在软件上的 API）方面更加负责，不仅在形式上，而且在精神上。否则，例如，来自一个组件新版本的数据流，虽然类型正确，但在值上可能对另一个组件是不可接受的（太大或太小）。保持跨团队的理解和责任重叠需要不断保持文化的活力和启发。鼓励创新和冒险，这可能导致业务突破，与来自同一业务人员的稳定和风险规避的保护倾向相矛盾。

从单一团队开发的整体式系统转变为多个团队和基于独立组件的系统需要企业各个层面的努力。你所说的“不再有质量保证部门”是什么意思？那么谁来关心测试人员的专业成长呢？IT 团队又怎么办？你所说的“开发人员将支持生产”是什么意思？这些变化影响人的生活，并不容易实施。这就是为什么 SOA 架构不仅仅是一个软件原则。它影响公司中的每个人。

与此同时，行业领袖们已经成功地超越了我们十年前所能想象的任何事情，他们被迫解决更加艰巨的问题，并带着他们的解决方案回到软件社区。这就是我们与金字塔建筑的类比不再适用的地方。因为新的挑战不仅仅是建造以前从未建造过的如此庞大的东西，而且要快速完成，不是几年的时间，而是几周甚至几天。而且结果不仅要持续千年，而且必须能够不断演变，并且足够灵活，以适应实时的新、意想不到的需求。如果功能的某个方面发生了变化，我们应该能够重新部署只有这一个服务。如果任何服务的需求增长，我们应该能够只扩展这一个服务，并在需求下降时释放资源。

为了避免全体人员参与的大规模部署，并接近持续部署（缩短上市时间，因此得到业务支持），功能继续分割成更小的服务块。为了满足需求，更复杂和健壮的云环境、部署工具（包括容器和容器编排）以及监控系统支持了这一举措。在前一课中描述的反应流开始发展之前，甚至在反应宣言出台之前，它们就已经开始发展，并在现代框架堆栈中插入了一个障碍。

将应用程序拆分为独立的部署单元带来了一些意想不到的好处，增加了前进的动力。服务的物理隔离允许更灵活地选择编程语言和实施平台。这不仅有助于选择最适合工作的技术，还有助于雇佣能够实施它的人，而不受公司特定技术堆栈的约束。它还帮助招聘人员扩大网络，利用更小的单元引入新的人才，这对于可用专家数量有限、快速增长的数据处理行业的无限需求来说是一个不小的优势。

此外，这样的架构强化了复杂系统各个部分之间的接口讨论和明确定义，从而为进一步的增长和调整提供了坚实的基础。

这就是微服务如何出现并被 Netflix、Google、Twitter、eBay、亚马逊和 Uber 等交通巨头所采用的情况。现在，让我们谈谈这一努力的结果和所学到的教训。

# 构建微服务

在着手构建过程之前，让我们重新审视一下代码块必须具备的特征，以便被视为微服务。我们将无序地进行这项工作：

+   一个微服务的源代码大小应该比 SOA 的源代码小，一个开发团队应该能够支持其中的几个。

+   它必须能够独立部署，而不依赖于其他服务。

+   每个微服务都必须有自己的数据库（或模式或一组表），尽管这种说法仍在争论中，特别是在几个服务修改相同数据集或相互依赖的数据集的情况下；如果同一个团队拥有所有相关服务，那么更容易实现。否则，我们将在后面讨论几种可能的策略。

+   它必须是无状态的和幂等的。如果服务的一个实例失败了，另一个实例应该能够完成服务所期望的工作。

+   它应该提供一种检查其**健康**的方式，意味着服务正在运行并且准备好做工作。

在设计、开发和部署后，必须考虑共享资源，并对假设进行监控验证。在上一课中，我们谈到了线程同步。您可以看到这个问题并不容易解决，我们提出了几种可能的解决方法。类似的方法可以应用于微服务。尽管它们在不同的进程中运行，但如果需要，它们可以相互通信，以便协调和同步它们的操作。

在修改相同的持久数据时，无论是跨数据库、模式还是同一模式内的表，都必须特别小心。如果可以接受最终一致性（这在用于统计目的的较大数据集中经常发生），则不需要采取特殊措施。然而，对事务完整性的需求提出了一个更为困难的问题。

支持跨多个微服务的事务的一种方法是创建一个扮演**分布式事务管理器**（**DTM**）角色的服务。需要协调的其他服务将新修改的值传递给它。DTM 服务可以将并发修改的数据暂时保存在数据库表中，并在所有数据准备好（并且一致）后一次性将其移入主表中。

如果访问数据的时间成为问题，或者您需要保护数据库免受过多的并发连接，将数据库专门用于某些服务可能是一个答案。或者，如果您想尝试另一种选择，内存缓存可能是一个好方法。添加一个提供对缓存的访问（并根据需要更新它）的服务可以增加对使用它的服务的隔离，但也需要（有时很困难的）对管理相同缓存的对等体之间进行同步。

在考虑了数据共享的所有选项和可能的解决方案之后，重新考虑为每个微服务创建自己的数据库（或模式）的想法通常是有帮助的。如果与动态同步数据相比，数据隔离（以及随后在数据库级别上的同步）的工作并不像以前那样艰巨，人们可能会发现。

说到这里，让我们来看看微服务实现的框架领域。一个人肯定可以从头开始编写微服务，但在这之前，值得看看已有的东西，即使最终发现没有什么符合你的特定需求。

目前有十多种框架用于构建微服务。最流行的两种是 Spring Boot（[`projects.spring.io/spring-boot/`](https://projects.spring.io/spring-boot/)）和原始的 J2EE。J2EE 社区成立了 MicroProfile 倡议（[`microprofile.io/`](https://microprofile.io/)），旨在**优化企业 Java**以适应微服务架构。KumuluzEE（[`ee.kumuluz.com/`](https://ee.kumuluz.com/)）是一个轻量级的开源微服务框架，符合 MicroProfile。

其他一些框架的列表包括以下内容（按字母顺序排列）：

+   **Akka**：这是一个用于构建高度并发、分布式和具有韧性的消息驱动应用程序的工具包，适用于 Java 和 Scala（[akka.io](https://akka.io/)）

+   **Bootique**：这是一个对可运行的 Java 应用程序没有太多主观看法的框架（[bootique.io](http://bootique.io)）

+   **Dropwizard**：这是一个用于开发友好的、高性能的、RESTful Web 服务的 Java 框架（[www.dropwizard.io](http://www.dropwizard.io)）

+   **Jodd**：这是一组 Java 微框架、工具和实用程序，大小不到 1.7MB（[jodd.org](http://jodd.org)）

+   **Lightbend Lagom**：这是一个基于 Akka 和 Play 构建的主观微服务框架（[www.lightbend.com](http://www.lightbend.com)）

+   **Ninja**：这是一个用于 Java 的全栈 Web 框架（[www.ninjaframework.org](http://www.ninjaframework.org)）

+   **Spotify Apollo**：这是 Spotify 用于编写微服务的一组 Java 库（[spotify.github.io/apollo](http://spotify.github.io/apollo)）

+   **Vert.x**：这是一个在 JVM 上构建反应式应用程序的工具包（[vertx.io](http://vertx.io)）

所有框架都支持微服务之间的 HTTP/JSON 通信；其中一些还有其他发送消息的方式。如果没有后者，可以使用任何轻量级的消息系统。我们在这里提到它是因为，正如您可能记得的那样，基于消息驱动的异步处理是由微服务组成的反应式系统的弹性、响应能力和韧性的基础。

为了演示微服务构建的过程，我们将使用 Vert.x，这是一个事件驱动、非阻塞、轻量级和多语言工具包（组件可以用 Java、JavaScript、Groovy、Ruby、Scala、Kotlin 和 Ceylon 编写）。它支持异步编程模型和分布式事件总线，甚至可以延伸到浏览器中的 JavaScript（从而允许创建实时 Web 应用程序）。

通过创建实现`io.vertx.core.Verticle`接口的`Verticle`类来开始使用 Vert.x：

```java
package io.vertx.core;
public interface Verticle {
  Vertx getVertx();
  void init(Vertx vertx, Context context);
  void start(Future<Void> future) throws Exception;
  void stop(Future<Void> future) throws Exception;
}
```

先前提到的方法名不言自明。方法`getVertex()`提供对`Vertx`对象的访问，这是进入 Vert.x Core API 的入口点。它提供了构建微服务所需的以下功能：

+   创建 TCP 和 HTTP 客户端和服务器

+   创建 DNS 客户端

+   创建数据报套接字

+   创建周期性服务

+   提供对事件总线和文件系统 API 的访问

+   提供对共享数据 API 的访问

+   部署和取消部署 verticles

使用这个 Vertx 对象，可以部署各种 verticles，它们彼此通信，接收外部请求，并像其他任何 Java 应用程序一样处理和存储数据，从而形成一个微服务系统。使用`io.vertx.rxjava`包中的 RxJava 实现，我们将展示如何创建一个反应式的微服务系统。

Vert.x 世界中的一个构建块是 verticle。它可以通过扩展`io.vertx.rxjava.core.AbstractVerticle`类轻松创建：

```java
package io.vertx.rxjava.core;
import io.vertx.core.Context;
import io.vertx.core.Vertx;
public class AbstractVerticle 
               extends io.vertx.core.AbstractVerticle {
  protected io.vertx.rxjava.core.Vertx vertx;
  public void init(Vertx vertx, Context context) {
     super.init(vertx, context);
     this.vertx = new io.vertx.rxjava.core.Vertx(vertx);
  }
}
```

前面提到的类，反过来又扩展了`io.vertx.core.AbstractVerticle`：

```java
package io.vertx.core;
import io.vertx.core.json.JsonObject;
import java.util.List;
public abstract class AbstractVerticle 
                               implements Verticle {
    protected Vertx vertx;
    protected Context context;
    public Vertx getVertx() { return vertx; }
    public void init(Vertx vertx, Context context) {
        this.vertx = vertx;
        this.context = context;
    }
    public String deploymentID() {
        return context.deploymentID();
    }
    public JsonObject config() {
        return context.config();
    }
    public List<String> processArgs() {
        return context.processArgs();
    }
    public void start(Future<Void> startFuture) 
                                throws Exception {
        start();
        startFuture.complete();
    }
    public void stop(Future<Void> stopFuture) 
                                throws Exception {
        stop();
        stopFuture.complete();
    }
    public void start() throws Exception {}
    public void stop() throws Exception {}

}
```

也可以通过扩展`io.vertx.core.AbstractVerticle`类来创建 verticle。但是，我们将编写反应式微服务，因此我们将扩展其 rx-fied 版本，`io.vertx.rxjava.core.AbstractVerticle`。

要使用 Vert.x 并运行提供的示例，您只需要添加以下依赖项：

```java
<dependency>
    <groupId>io.vertx</groupId>
    <artifactId>vertx-web</artifactId>
    <version>${vertx.version}</version>
</dependency>

<dependency>
    <groupId>io.vertx</groupId>
    <artifactId>vertx-rx-java</artifactId>
    <version>${vertx.version}</version>
</dependency>
```

其他 Vert.x 功能可以根据需要添加其他 Maven 依赖项。

使`Vert.x` `Verticle`具有反应性的是事件循环（线程）的底层实现，它接收事件并将其传递给`Handler`（我们将展示如何为其编写代码）。当`Handler`获得结果时，事件循环将调用回调函数。

### 注意

如您所见，重要的是不要编写阻塞事件循环的代码，因此 Vert.x 的黄金规则是：不要阻塞事件循环。

如果没有阻塞，事件循环将非常快速地工作，并在短时间内传递大量事件。这称为反应器模式（[`en.wikipedia.org/wiki/Reactor_pattern`](https://en.wikipedia.org/wiki/Reactor_pattern)）。这种事件驱动的非阻塞编程模型非常适合响应式微服务。对于某些本质上是阻塞的代码类型（JDBC 调用和长时间计算是很好的例子），可以异步执行工作 verticle（不是由事件循环，而是使用`vertx.executeBlocking()`方法由单独的线程执行），这保持了黄金规则的完整性。

让我们看几个例子。这是一个作为 HTTP 服务器工作的`Verticle`类：

```java
import io.vertx.rxjava.core.http.HttpServer;
import io.vertx.rxjava.core.AbstractVerticle;

public class Server extends AbstractVerticle{
  private int port;
  public Server(int port) {
    this.port = port;
  }
  public void start() throws Exception {
    HttpServer server = vertx.createHttpServer();
    server.requestStream().toObservable()
       .subscribe(request -> request.response()
       .end("Hello from " + 
          Thread.currentThread().getName() + 
                       " on port " + port + "!\n\n")
       );
    server.rxListen(port).subscribe();
    System.out.println(Thread.currentThread().getName()
               + " is waiting on port " + port + "...");
  }
}
```

在上面的代码中，创建了服务器，并将可能的请求的数据流包装成`Observable`。然后，我们订阅了来自`Observable`的数据，并传入一个函数（请求处理程序），该函数将处理请求并生成必要的响应。我们还告诉服务器要监听哪个端口。使用这个`Verticle`，我们可以部署几个实例，监听不同的端口的 HTTP 服务器。这是一个例子：

```java
import io.vertx.rxjava.core.RxHelper;
import static io.vertx.rxjava.core.Vertx.vertx;
public class Demo01Microservices {
  public static void main(String... args) {
    RxHelper.deployVerticle(vertx(), new Server(8082));
    RxHelper.deployVerticle(vertx(), new Server(8083));
  }
}
```

如果我们运行此应用程序，输出将如下所示：

![构建微服务](img/04_01.jpg)

如您所见，同一个线程同时监听两个端口。如果我们现在向每个正在运行的服务器发送请求，我们将得到我们已经硬编码的响应：

![构建微服务](img/04_02.jpg)

我们从`main()`方法运行示例。插件`maven-shade-plugin`允许您指定要作为应用程序起点的 verticle。以下是来自[`vertx.io/blog/my-first-vert-x-3-application`](http://vertx.io/blog/my-first-vert-x-3-application)的示例：

```java
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-shade-plugin</artifactId>
  <version>2.3</version>
  <executions>
    <execution>
      <phase>package</phase>
      <goals>
        <goal>shade</goal>
      </goals>
      <configuration>
        <transformers>
          <transformer
            implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
            <manifestEntries>
              <Main-Class>io.vertx.core.Starter</Main-Class>
              <Main-Verticle>io.vertx.blog.first.MyFirstVerticle</Main-Verticle>
            </manifestEntries>
          </transformer>
        </transformers>
        <artifactSet/>
        <outputFile>${project.build.directory}/${project.artifactId}-${project.version}-fat.jar</outputFile>
      </configuration>
    </execution>
  </executions>
</plugin>
```

现在，运行以下命令：

```java
mvn package

```

它将生成一个指定的 JAR 文件（在本例中称为`target/my-first-app-1.0-SNAPSHOT-fat.jar`）。它被称为`fat`，因为它包含所有必要的依赖项。此文件还将包含`MANIFEST.MF`，其中包含以下条目：

```java
Main-Class: io.vertx.core.Starter
Main-Verticle: io.vertx.blog.first.MyFirstVerticle
```

您可以使用任何 verticle 来代替此示例中使用的`io.vertx.blog.first.MyFirstVerticle`，但必须有`io.vertx.core.Starter`，因为那是知道如何读取清单并执行指定 verticle 的`Vert.x`类的名称。现在，您可以运行以下命令：

```java
java -jar target/my-first-app-1.0-SNAPSHOT-fat.jar

```

此命令将执行`MyFirstVerticle`类的`start()`方法，就像我们的示例中执行`main()`方法一样，我们将继续使用它来简化演示。

为了补充 HTTP 服务器，我们也可以创建一个 HTTP 客户端。但是，首先，我们将修改`server` verticle 中的`start()`方法，以接受参数`name`：

```java
public void start() throws Exception {
    HttpServer server = vertx.createHttpServer();
    server.requestStream().toObservable()
       .subscribe(request -> request.response()
       .end("Hi, " + request.getParam("name") + 
             "! Hello from " + 
             Thread.currentThread().getName() + 
                       " on port " + port + "!\n\n")
       );
    server.rxListen(port).subscribe();
    System.out.println(Thread.currentThread().getName()
               + " is waiting on port " + port + "...");
}
```

现在，我们可以创建一个 HTTP“客户端”verticle，每秒发送一个请求并打印出响应，持续 3 秒，然后停止：

```java
import io.vertx.rxjava.core.AbstractVerticle;
import io.vertx.rxjava.core.http.HttpClient;
import java.time.LocalTime;
import java.time.temporal.ChronoUnit;

public class Client extends AbstractVerticle {
  private int port;
  public Client(int port) {
    this.port = port;
  }
  public void start() throws Exception {
    HttpClient client = vertx.createHttpClient();
    LocalTime start = LocalTime.now();
    vertx.setPeriodic(1000, v -> {
       client.getNow(port, "localhost", "?name=Nick",
         r -> r.bodyHandler(System.out::println));
         if(ChronoUnit.SECONDS.between(start, 
                             LocalTime.now()) > 3 ){
           vertx.undeploy(deploymentID());
       }
    });
  }
}
```

假设我们部署两个 verticle 如下：

```java
RxHelper.deployVerticle(vertx(), new Server2(8082));
RxHelper.deployVerticle(vertx(), new Client(8082));
```

输出将如下所示：

![构建微服务](img/04_03.jpg)

在最后一个示例中，我们演示了如何创建 HTTP 客户端和周期性服务。现在，让我们为我们的系统添加更多功能。例如，让我们添加另一个 verticle，它将与数据库交互，并通过我们已经创建的 HTTP 服务器使用它。

首先，我们需要添加此依赖项：

```java
<dependency>
    <groupId>io.vertx</groupId>
    <artifactId>vertx-jdbc-client</artifactId>
    <version>${vertx.version}</version>
</dependency>
```

新添加的 JAR 文件允许我们创建一个内存数据库和一个访问它的处理程序：

```java
public class DbHandler {
  private JDBCClient dbClient;
  private static String SQL_CREATE_WHO_CALLED = 
    "CREATE TABLE IF NOT EXISTS " +
          "who_called ( name VARCHAR(10), " +
          "create_ts TIMESTAMP(6) DEFAULT now() )";
  private static String SQL_CREATE_PROCESSED = 
    "CREATE TABLE IF NOT EXISTS " +
         "processed ( name VARCHAR(10), " +
         "length INTEGER, " +
         "create_ts TIMESTAMP(6) DEFAULT now() )";

  public DbHandler(Vertx vertx){
    JsonObject config = new JsonObject()
      .put("driver_class", "org.hsqldb.jdbcDriver")
      .put("url", "jdbc:hsqldb:mem:test?shutdown=true");
    dbClient = JDBCClient.createShared(vertx, config);
    dbClient.rxGetConnection()
      .flatMap(conn -> 
                 conn.rxUpdate(SQL_CREATE_WHO_CALLED)
                       .doAfterTerminate(conn::close) )
      .subscribe(r -> 
        System.out.println("Table who_called created"),
                           Throwable::printStackTrace);
    dbClient.rxGetConnection()
      .flatMap(conn -> 
                 conn.rxUpdate(SQL_CREATE_PROCESSED)
                      .doAfterTerminate(conn::close) )
      .subscribe(r -> 
        System.out.println("Table processed created"),
                          Throwable::printStackTrace);

  }
}
```

熟悉 RxJava 的人可以看到，Vert.x 代码紧密遵循 RxJava 的风格和命名约定。尽管如此，我们鼓励您阅读 Vert.x 文档，因为它具有非常丰富的 API，涵盖了比我们演示的更多情况。在前面的代码中，`flatMap()`操作接收运行脚本然后关闭连接的函数。在这种情况下，`doAfterTerminate()`操作的作用就像是在传统代码中放置在 finally 块中并在成功或生成异常时关闭连接。`subscribe()`方法有几个重载版本。对于我们的代码，我们选择了一个在成功时执行一个函数（我们打印有关创建表的消息），在异常时执行另一个函数（我们只打印堆栈跟踪）。

要使用创建的数据库，我们可以向`DbHandler`添加`insert()`、`process()`和`readProcessed()`方法，这将允许我们演示如何构建一个响应式系统。`insert()`方法的代码可能如下所示：

```java
private static String SQL_INSERT_WHO_CALLED = 
             "INSERT INTO who_called(name) VALUES (?)";
public void insert(String name, Action1<UpdateResult> 
                onSuccess, Action1<Throwable> onError){
  printAction("inserts " + name);
  dbClient.rxGetConnection()
    .flatMap(conn -> 
        conn.rxUpdateWithParams(SQL_INSERT_WHO_CALLED, 
                            new JsonArray().add(name))
                       .doAfterTerminate(conn::close) )
    .subscribe(onSuccess, onError);
}
```

`insert()`方法以及我们将要编写的其他方法充分利用了 Java 函数接口。它在表`who_called`中创建一条记录（使用传入的参数`name`）。然后，`subscribe()`操作执行调用此方法的代码传递的两个函数中的一个。我们仅使用`printAction()`方法以获得更好的可追踪性。

```java
private void printAction(String action) {  
  System.out.println(this.getClass().getSimpleName() 
                                     + " " + action);
}
```

`process()`方法还接受两个函数，但不需要其他参数。它处理表`who_called`中尚未处理的所有记录（未在`processed`表中列出）：

```java
private static String SQL_SELECT_TO_PROCESS = 
  "SELECT name FROM who_called w where name not in " +
  "(select name from processed) order by w.create_ts " +
  "for update";
private static String SQL_INSERT_PROCESSED = 
     "INSERT INTO processed(name, length) values(?, ?)";
public void process(Func1<JsonArray, Observable<JsonArray>> 
                     process, Action1<Throwable> onError) {
  printAction("process all records not processed yet");
  dbClient.rxGetConnection()
    .flatMapObservable(conn -> 
       conn.rxQueryStream(SQL_SELECT_TO_PROCESS)
           .flatMapObservable(SQLRowStream::toObservable)
           .flatMap(process)
           .flatMap(js -> 
              conn.rxUpdateWithParams(SQL_INSERT_PROCESSED, js)
                  .flatMapObservable(ur->Observable.just(js)))
           .doAfterTerminate(conn::close))
    .subscribe(js -> printAction("processed " + js), onError);
}
```

如果两个线程正在读取表`who_called`以选择尚未处理的记录，SQL 查询中的`for update`子句确保只有一个线程获取每条记录，因此它们不会被处理两次。`process()`方法代码的显着优势在于其使用`rxQueryStream()`操作，该操作逐个发出找到的记录，以便它们独立地进行处理。在大量未处理记录的情况下，这样的解决方案保证了结果的平稳交付，而不会消耗资源。以下的`flatMap()`操作使用传递的函数进行处理。该函数的唯一要求是它必须返回一个整数值（在`JsonArray`中），该值将用作`SQL_INSERT_PROCESSED`语句的参数。因此，调用此方法的代码决定处理的性质。代码的其余部分类似于`insert()`方法。代码缩进有助于跟踪操作的嵌套。

`readProcessed()`方法的代码看起来与`insert()`方法的代码非常相似：

```java
private static String SQL_READ_PROCESSED = 
  "SELECT name, length, create_ts FROM processed 
                       order by create_ts desc limit ?";
public void readProcessed(String count, Action1<ResultSet> 
                  onSuccess, Action1<Throwable> onError) {
  printAction("reads " + count + 
                            " last processed records");
  dbClient.rxGetConnection()
   .flatMap(conn -> 
      conn.rxQueryWithParams(SQL_READ_PROCESSED, 
                          new JsonArray().add(count))
                      .doAfterTerminate(conn::close) )
   .subscribe(onSuccess, onError);
}
```

前面的代码读取了指定数量的最新处理记录。与`process()`方法的不同之处在于，`readProcessed()`方法返回一个结果集中的所有读取记录，因此由使用此方法的用户决定如何批量处理结果或逐个处理。我们展示所有这些可能性只是为了展示可能的选择的多样性。有了`DbHandler`类，我们准备好使用它并创建`DbServiceHttp`微服务，它允许通过包装一个 HTTP 服务器来远程访问`DbHandler`的功能。这是新微服务的构造函数：

```java
public class DbServiceHttp extends AbstractVerticle {
  private int port;
  private DbHandler dbHandler;
  public DbServiceHttp(int port) {
    this.port = port;
  }
  public void start() throws Exception {
    System.out.println(this.getClass().getSimpleName() + 
                            "(" + port + ") starts...");
    dbHandler = new DbHandler(vertx);
    Router router = Router.router(vertx);
    router.put("/insert/:name").handler(this::insert);
    router.get("/process").handler(this::process);
    router.get("/readProcessed")
                         .handler(this::readProcessed);
    vertx.createHttpServer()
          .requestHandler(router::accept).listen(port);
  }
}
```

在前面提到的代码中，您可以看到 Vert.x 中的 URL 映射是如何完成的。对于每个可能的路由，都分配了相应的`Verticle`方法，每个方法都接受包含所有 HTTP 上下文数据的`RoutingContext`对象，包括`HttpServerRequest`和`HttpServerResponse`对象。各种便利方法使我们能够轻松访问 URL 参数和其他处理请求所需的数据。这是`start()`方法中引用的`insert()`方法：

```java
private void insert(RoutingContext routingContext) {
  HttpServerResponse response = routingContext.response();
  String name = routingContext.request().getParam("name");
  printAction("insert " + name);
  Action1<UpdateResult> onSuccess = 
    ur -> response.setStatusCode(200).end(ur.getUpdated() + 
                 " record for " + name + " is inserted");
  Action1<Throwable> onError = ex -> {
    printStackTrace("process", ex);
    response.setStatusCode(400)
        .end("No record inserted due to backend error");
  };
  dbHandler.insert(name, onSuccess, onError);
}
```

它只是从请求中提取参数`name`并构造调用我们先前讨论过的`DbHandler`的`insert()`方法所需的两个函数。`process()`方法看起来与先前的`insert()`方法类似：

```java
private void process(RoutingContext routingContext) {
  HttpServerResponse response = routingContext.response();
  printAction("process all");
  response.setStatusCode(200).end("Processing...");
  Func1<JsonArray, Observable<JsonArray>> process = 
    jsonArray -&gt; { 
      String name = jsonArray.getString(0);
      JsonArray js = 
            new JsonArray().add(name).add(name.length());
       return Observable.just(js);
  };
  Action1<Throwable> onError = ex -> {
     printStackTrace("process", ex);
     response.setStatusCode(400).end("Backend error");
  };
  dbHandler.process(process, onError);
}
```

先前提到的`process`函数定义了从`DbHandler`方法`process()`中的`SQL_SELECT_TO_PROCESS`语句获取的记录应该做什么。在我们的情况下，它计算了呼叫者姓名的长度，并将其作为参数与姓名本身一起（作为返回值）传递给下一个 SQL 语句，将结果插入到`processed`表中。

这是`readProcessed()`方法：

```java
private void readProcessed(RoutingContext routingContext) {
  HttpServerResponse response = routingContext.response();
  String count = routingContext.request().getParam("count");
  printAction("readProcessed " + count + " entries");
  Action1<ResultSet> onSuccess = rs -> {
     Observable.just(rs.getResults().size() > 0 ? 
       rs.getResults().stream().map(Object::toString)
                   .collect(Collectors.joining("\n")) : "")
       .subscribe(s -> response.setStatusCode(200).end(s) );
  };
  Action1<Throwable> onError = ex -> {
      printStackTrace("readProcessed", ex);
      response.setStatusCode(400).end("Backend error");
  };
  dbHandler.readProcessed(count, onSuccess, onError);
}
```

这就是（在前面的`onSuccess()`函数中的先前代码中）从查询`SQL_READ_PROCESSED`中读取并用于构造响应的结果集。请注意，我们首先通过创建`Observable`，然后订阅它并将订阅的结果作为响应传递给`end()`方法来执行此操作。否则，可以在构造响应之前返回响应而不必等待响应。

现在，我们可以通过部署`DbServiceHttp` verticle 来启动我们的反应式系统。

```java
RxHelper.deployVerticle(vertx(), new DbServiceHttp(8082));
```

如果我们这样做，输出中将会看到以下代码行：

```java
DbServiceHttp(8082) starts...
Table processed created
Table who_called created
```

在另一个窗口中，我们可以发出生成 HTTP 请求的命令：

![构建微服务](img/04_04.jpg)

如果现在读取处理过的记录，应该没有：

![构建微服务](img/04_05.jpg)

日志消息显示如下：

![构建微服务](img/4_06.jpg)

现在，我们可以请求处理现有记录，然后再次读取结果：

![构建微服务](img/04_07.jpg)

原则上，已经足够构建一个反应式系统。我们可以在不同端口部署许多`DbServiceHttp`微服务，或者将它们集群化以增加处理能力、韧性和响应能力。我们可以将其他服务包装在 HTTP 客户端或 HTTP 服务器中，让它们相互通信，处理输入并将结果传递到处理管道中。

然而，Vert.x 还具有一个更适合消息驱动架构（而不使用 HTTP）的功能。它被称为事件总线。任何 verticle 都可以访问事件总线，并可以使用`send()`方法（在响应式编程的情况下使用`rxSend()`）或`publish()`方法向任何地址（只是一个字符串）发送任何消息。一个或多个 verticle 可以注册自己作为某个地址的消费者。

如果许多 verticle 是相同地址的消费者，那么`send()`（`rxSend()`）方法只将消息传递给其中一个（使用循环轮询算法选择下一个消费者）。`publish()`方法如您所期望的那样，将消息传递给具有相同地址的所有消费者。让我们看一个例子，使用已经熟悉的`DbHandler`作为主要工作马。

基于事件总线的微服务看起来与我们已经讨论过的基于 HTTP 协议的微服务非常相似：

```java
public class DbServiceBus extends AbstractVerticle {
  private int id;
  private String instanceId;
  private DbHandler dbHandler;
  public static final String INSERT = "INSERT";
  public static final String PROCESS = "PROCESS";
  public static final String READ_PROCESSED 
                              = "READ_PROCESSED";
  public DbServiceBus(int id) { this.id = id; }
  public void start() throws Exception {
    this.instanceId = this.getClass().getSimpleName()
                                     + "(" + id + ")";
    System.out.println(instanceId + " starts...");
    this.dbHandler = new DbHandler(vertx);
    vertx.eventBus().consumer(INSERT).toObservable()
      .subscribe(msg -> {
         printRequest(INSERT, msg.body().toString());
         Action1<UpdateResult> onSuccess 
                               = ur -> msg.reply(...);
         Action1<Throwable> onError 
                   = ex -> msg.reply("Backend error");
         dbHandler.insert(msg.body().toString(), 
                                 onSuccess, onError);
    });

    vertx.eventBus().consumer(PROCESS).toObservable()
        .subscribe(msg -> {
                  .....
                 dbHandler.process(process, onError);
        });

    vertx.eventBus().consumer(READ_PROCESSED).toObservable()
        .subscribe(msg -> {
                 ...
            dbHandler.readProcessed(msg.body().toString(), 
                                        onSuccess, onError);
        });
    }
```

我们通过跳过一些部分（与`DbServiceHttp`类非常相似）简化了前面的代码，并试图突出代码结构。为了演示目的，我们将部署两个此类的实例，并向每个地址`INSERT`、`PROCESS`和`READ_PROCESSED`发送三条消息：

```java
void demo_DbServiceBusSend() {
  Vertx vertx = vertx();
  RxHelper.deployVerticle(vertx, new DbServiceBus(1));
  RxHelper.deployVerticle(vertx, new DbServiceBus(2));
  delayMs(200);
  String[] msg1 = {"Mayur", "Rohit", "Nick" };
  RxHelper.deployVerticle(vertx, 
    new PeriodicServiceBusSend(DbServiceBus.INSERT, msg1, 1));
  String[] msg2 = {"all", "all", "all" };
  RxHelper.deployVerticle(vertx, 
    new PeriodicServiceBusSend(DbServiceBus.PROCESS, msg2, 1));
  String[] msg3 = {"1", "1", "2", "3" };
  RxHelper.deployVerticle(vertx, 
     new PeriodicServiceBusSend(DbServiceBus.READ_PROCESSED, 
                                                     msg3, 1));
}
```

请注意，我们使用`delayMs()`方法插入了 200 毫秒的延迟：

```java
void delayMs(int ms){
    try {
        TimeUnit.MILLISECONDS.sleep(ms);
    } catch (InterruptedException e) {}
}
```

延迟是必要的，以便让`DbServiceBus`顶点被部署和启动（并且消费者注册到地址）。否则，发送消息的尝试可能会失败，因为消费者尚未注册到地址。`PeriodicServiceBusSend()`顶点的代码如下：

```java
public class PeriodicServiceBusSend 
                           extends AbstractVerticle {
  private EventBus eb;
  private LocalTime start;
  private String address;
  private String[] caller;
  private int delaySec;
  public PeriodicServiceBusSend(String address, 
                     String[] caller, int delaySec) {
        this.address = address;
        this.caller = caller;
        this.delaySec = delaySec;
  }
  public void start() throws Exception {
    System.out.println(this.getClass().getSimpleName() 
      + "(" + address + ", " + delaySec + ") starts...");
    this.eb = vertx.eventBus();
    this.start  = LocalTime.now();
    vertx.setPeriodic(delaySec * 1000, v -> {
       int i = (int)ChronoUnit.SECONDS.between(start,
                                    LocalTime.now()) - 1;
       System.out.println(this.getClass().getSimpleName()
          + " to address " + address + ": " + caller[i]);
       eb.rxSend(address, caller[i]).subscribe(reply -> {
         System.out.println(this.getClass().getSimpleName() 
                    + " got reply from address " + address 
                               + ":\n    " + reply.body());
          if(i + 1 >= caller.length ){
               vertx.undeploy(deploymentID());
          }
       }, Throwable::printStackTrace);
    });
  }
}
```

之前的代码每`delaySec`秒向一个地址发送一条消息，次数等于数组`caller[]`的长度，然后取消部署该顶点（自身）。如果我们运行演示，输出的开头将如下所示：

![构建微服务](img/04_08.jpg)

正如您所看到的，对于每个地址，只有`DbServiceBus(1)`是第一条消息的接收者。第二条消息到达相同地址时，被`DbServiceBus(2)`接收。这就是轮询算法（我们之前提到过的）的运行方式。输出的最后部分看起来像这样：

![构建微服务](img/04_09.jpg)

我们可以部署所需数量的相同类型的顶点。例如，让我们部署四个发送消息到地址`INSERT`的顶点：

```java
String[] msg1 = {"Mayur", "Rohit", "Nick" };
RxHelper.deployVerticle(vertx, 
  new PeriodicServiceBusSend(DbServiceBus.INSERT, msg1, 1));
RxHelper.deployVerticle(vertx, 
  new PeriodicServiceBusSend(DbServiceBus.INSERT, msg1, 1));
RxHelper.deployVerticle(vertx, 
  new PeriodicServiceBusSend(DbServiceBus.INSERT, msg1, 1));
RxHelper.deployVerticle(vertx, 
  new PeriodicServiceBusSend(DbServiceBus.INSERT, msg1, 1));
```

为了查看结果，我们还将要求读取顶点读取最后八条记录：

```java
String[] msg3 = {"1", "1", "2", "8" };
RxHelper.deployVerticle(vertx, 
  new PeriodicServiceBusSend(DbServiceBus.READ_PROCESSED, 
                                               msg3, 1));
```

结果（输出的最后部分）将如预期的那样：

![构建微服务](img/04_10.jpg)

四个顶点发送了相同的消息，所以每个名称被发送了四次并被处理，这就是我们在之前输出中看到的。

现在，我们将返回到一个插入周期性顶点，但将其从使用`rxSend()`方法更改为使用`publish()`方法：

```java
PeriodicServiceBusPublish(String address, String[] caller, int delaySec) {
  ...
  vertx.setPeriodic(delaySec * 1000, v -> {
    int i = (int)ChronoUnit.SECONDS.between(start, 
                                      LocalTime.now()) - 1;
    System.out.println(this.getClass().getSimpleName()
            + " to address " + address + ": " + caller[i]);
    eb.publish(address, caller[i]);
    if(i + 1 == caller.length ){
        vertx.undeploy(deploymentID());
    }
  });
}
```

这个改变意味着消息必须发送到所有在该地址注册为消费者的顶点。现在，让我们运行以下代码：

```java
Vertx vertx = vertx();
RxHelper.deployVerticle(vertx, new DbServiceBus(1));
RxHelper.deployVerticle(vertx, new DbServiceBus(2));
delayMs(200);
String[] msg1 = {"Mayur", "Rohit", "Nick" };
RxHelper.deployVerticle(vertx, 
  new PeriodicServiceBusPublish(DbServiceBus.INSERT, 
                                               msg1, 1));
delayMs(200);
String[] msg2 = {"all", "all", "all" };
RxHelper.deployVerticle(vertx, 
  new PeriodicServiceBusSend(DbServiceBus.PROCESS, 
                                               msg2, 1));
String[] msg3 = {"1", "1", "2", "8" };
RxHelper.deployVerticle(vertx, 
  new PeriodicServiceBusSend(DbServiceBus.READ_PROCESSED, 
                                               msg3, 1));
```

我们已经增加了另一个延迟为 200 毫秒，以便发布顶点有时间发送消息。输出（在最后部分）现在显示每条消息被处理了两次：

![构建微服务](img/04_11.jpg)

这是因为部署了两个消费者`DbServiceBus(1)`和`DbServiceBus(2)`，每个都收到了发送到地址`INSERT`的消息，并将其插入到表`who_called`中。

我们之前的所有示例都在一个 JVM 进程中运行。如果需要，Vert.x 实例可以部署在不同的 JVM 进程中，并通过在运行命令中添加`-cluster`选项进行集群化。因此，它们共享事件总线，地址对所有 Vert.x 实例可见。这样，资源可以根据需要添加到每个地址。例如，我们只能增加处理微服务的数量，并补偿负载的增加。

我们之前提到的其他框架也具有类似的功能。它们使得微服务的创建变得容易，并可能鼓励将应用程序分解为微小的单方法操作，期望组装一个非常具有弹性和响应性的系统。

然而，这些并不是唯一的优质标准。系统分解会增加其部署的复杂性。此外，如果一个开发团队负责许多微服务，那么在不同阶段（开发、测试、集成测试、认证、暂存、生产）中对这么多部分进行版本控制的复杂性可能会导致混乱和非常具有挑战性的部署过程，反过来可能会减缓保持系统与市场需求同步所需的变更速度。

除了开发微服务，还必须解决许多其他方面来支持反应式系统：

+   必须设计一个监控系统，以便深入了解应用程序的状态，但不应该太复杂，以至于将开发资源从主要应用程序中抽离出来。

+   必须安装警报以及及时警告团队可能和实际问题，以便在影响业务之前解决这些问题。

+   如果可能的话，必须实施自我纠正的自动化流程。例如，系统应该能够根据当前负载添加和释放资源；必须实施重试逻辑，并设置合理的尝试上限，以避免宣布失败。

+   必须有一层断路器来保护系统，防止一个组件的故障导致其他组件缺乏必要的资源。

+   嵌入式测试系统应该能够引入中断并模拟处理负载，以确保应用程序的弹性和响应能力不会随时间而降低。例如，Netflix 团队引入了一个名为“混沌猴”的系统，它能够关闭生产系统的各个部分，以测试其恢复能力。他们甚至在生产环境中使用它，因为生产环境具有特定的配置，其他环境中的测试无法保证找到所有可能的问题。

响应式系统设计的主要考虑因素之一是选择部署方法论，可以是无容器、自包含或容器内部。我们将在本课程的后续部分中探讨每种方法的利弊。

# 无容器部署

人们使用术语“容器”来指代非常不同的东西。在最初的用法中，容器是指将其内容从一个位置运送到另一个位置而不改变内部任何内容的东西。然而，当服务器被引入时，只强调了一个方面，即容纳应用程序的能力。此外，还添加了另一层含义，即提供生命支持基础设施，以便容器的内容（应用程序）不仅能够存活，而且能够活跃并响应外部请求。这种重新定义的容器概念被应用到了 Web 服务器（Servlet 容器）、应用服务器（带有或不带有 EJB 容器的应用程序容器）以及其他为应用程序提供支持环境的软件设施。有时，甚至将 JVM 本身称为容器，但这种关联可能没有持续下去，可能是因为能够积极参与（执行）内容的能力与容器的原始含义不太吻合。

这就是为什么后来，当人们开始谈论无容器部署时，他们通常指的是能够直接将应用程序部署到 JVM 中，而无需先安装 WebSphere、WebLogic、JBoss 或任何其他提供应用程序运行环境的中介软件。

在前面的部分中，我们描述了许多框架，这些框架使我们能够构建和部署应用程序（或者说是微服务的响应式系统），而无需 JVM 本身以外的任何其他容器。你所需要做的就是构建一个包含所有依赖项（除了来自 JVM 本身的依赖项）的大型 JAR 文件，然后将其作为独立的 Java 进程运行。

```java
$ java -jar myfatjar.jar

```

此外，您还需要确保 JAR 文件中的`MANIFEST.MF`包含一个指向完全限定类名的`main`类的条目，该类具有`main()`方法，并将在启动时运行。我们已经在前一节“构建微服务”中描述了如何做到这一点。

这就是 Java 的一次编译，到处运行的承诺，到处意味着安装了某个版本或更高版本的 JVM 的任何地方。这种方法有几个优点和缺点。我们将讨论它们，而不是相对于传统的服务器容器部署。不使用传统容器进行部署的优势是非常明显的，从许多（如果有的话）许可成本更少开始，以及更轻量级的部署和可扩展性过程，甚至没有提到更少的资源消耗。相反，我们将不是将无容器部署与传统部署进行比较，而是与自包含和新一代容器中的容器进行比较，这些容器是几年前开发的。

它们不仅允许包含和执行包含的代码，传统容器也可以做到，而且还可以将其移动到不需要对包含的代码进行任何更改的不同位置。从现在开始，通过容器，我们只指的是新的容器。

无容器部署的优势如下：

+   很容易在同一台物理（或虚拟或云）机器上或在新硬件上添加更多的 Java 进程

+   进程之间的隔离级别很高，在共享环境中尤其重要，当您无法控制其他共同部署的应用程序时，可能会有恶意应用程序试图渗透到相邻的执行环境中

+   它的占用空间很小，因为除了应用程序本身或一组微服务之外，它不包括任何其他内容

无容器部署的缺点如下：

+   每个 JAR 文件都需要某个版本或更高版本的 JVM，这可能会迫使您因此而启动一个新的物理或虚拟机器，以部署一个特定的 JAR 文件

+   在您无法控制的环境中，您的代码可能会使用错误版本的 JVM 部署，这可能会导致不可预测的结果。

+   在同一 JVM 中的进程竞争资源，这在由不同团队或不同公司共享的环境中尤其难以管理

+   当几个微服务捆绑到同一个 JAR 文件中时，它们可能需要不同版本的第三方库，甚至是不兼容的库

微服务可以每个 JAR 部署一个，也可以由团队捆绑在一起，由相关服务，按比例单位，或使用其他标准。最不重要的考虑是这些 JAR 文件的总数。随着这个数字的增长（今天谷歌一次处理数十万个部署单元），可能无法通过简单的 bash 脚本处理部署，并需要一个复杂的过程，以便考虑可能的不兼容性。如果是这种情况，那么考虑使用虚拟机或容器（在它们的新版本中，见下一节）以获得更好的隔离和管理是合理的。

# 自包含的微服务

自包含的微服务看起来与无容器非常相似。唯一的区别是 JVM（或实际上是 JRE）或应用程序运行所需的任何其他外部框架和服务器也包含在 fat JAR 文件中。有许多构建这样一个全包 JAR 文件的方法。

例如，Spring Boot 提供了一个方便的 GUI，其中包含复选框列表，允许您选择要打包的 Spring Boot 应用程序及外部工具的哪些部分。同样，WildFly Swarm 允许您选择要与应用程序捆绑在一起的 Java EE 组件的哪些部分。或者，您可以使用`javapackager`工具自己来完成。它将应用程序和 JRE 编译打包到同一个 JAR 文件中（也可以是`.exe`或`.dmg`）以进行分发。您可以在 Oracle 网站上阅读有关该工具的信息[`docs.oracle.com/javase/9/tools/javapackager.htm`](https://docs.oracle.com/javase/9/tools/javapackager.htm)，或者您可以在安装了 JDK 的计算机上运行`javapackager`命令（它也随 Java 8 一起提供），您将获得工具选项列表及其简要描述。

基本上，要使用`javapackager`工具，您只需要准备一个项目，其中包括您想要一起打包的所有内容，包括所有依赖项（打包在 JAR 文件中），然后使用必要的选项运行`javapackager`命令，这些选项允许您指定您想要的输出类型（例如`.exe`或`.dmg`），您想要捆绑在一起的 JRE 位置，要使用的图标，`MANIFEST.MF`的`main`类入口等。还有 Maven 插件可以使打包命令更简单，因为`pom.xml`中的大部分设置都必须进行配置。

自包含部署的优点如下：

+   这是一个文件（包含组成反应系统或其中某部分的所有微服务）需要处理，这对用户和分发者来说更简单

+   无需预先安装 JRE，也无需担心版本不匹配。

+   隔离级别很高，因为您的应用程序有专用的 JRE，因此来自共同部署应用程序的入侵风险很小

+   您可以完全控制捆绑包中包含的依赖关系

缺点如下：

+   文件大小更大，如果必须下载，可能会成为障碍

+   与无容器的 JAR 文件相比，配置更复杂

+   该捆绑包必须在与目标平台匹配的平台上生成，如果您无法控制安装过程，可能会导致不匹配。

+   部署在同一硬件或虚拟机上的其他进程可能会占用您的应用程序所需的关键资源，如果您的应用程序不是由开发团队下载和运行，这将特别难以管理

# 容器内部部署

熟悉虚拟机（VM）而不熟悉现代容器（如 Docker、CoreOS 的 Rocket、VMware Photon 或类似产品）的人可能会认为我们在说容器不仅可以包含和执行包含的代码，还可以将其移动到不同位置而不对包含的代码进行任何更改。如果是这样，那将是一个相当恰当的假设。虚拟机确实允许所有这些，而现代容器可以被认为是轻量级虚拟机，因为它也允许分配资源并提供独立机器的感觉。然而，容器并不是一个完全隔离的虚拟计算机。

关键区别在于作为 VM 传递的捆绑包包含了整个操作系统（部署的应用程序）。因此，运行两个 VM 的物理服务器可能在其上运行两个不同的操作系统。相比之下，运行三个容器化应用程序的物理服务器（或 VM）只运行一个操作系统，并且两个容器共享（只读）操作系统内核，每个容器都有自己的访问（挂载）以写入它们不共享的资源。这意味着，例如，启动时间更短，因为启动容器不需要我们引导操作系统（与 VM 的情况相反）。

举个例子，让我们更仔细地看看 Docker，这是容器领域的社区领袖。2015 年，一个名为**Open Container Project**的倡议被宣布，后来更名为**Open Container Initiative**（**OCI**），得到了 Google、IBM、亚马逊、微软、红帽、甲骨文、VMware、惠普、Twitter 等许多公司的支持。它的目的是为所有平台开发容器格式和容器运行时软件的行业标准。Docker 捐赠了大约 5%的代码库给该项目，因为其解决方案被选为起点。

Docker 有广泛的文档，网址为：[`docs.docker.com`](https://docs.docker.com)。使用 Docker，可以将所有的 Java EE 容器和应用程序作为 Docker 镜像打包，实现与自包含部署基本相同的结果。然后，你可以通过在 Docker 引擎中启动 Docker 镜像来启动你的应用程序，使用以下命令：

```java
$ docker run mygreatapplication

```

它启动一个看起来像在物理计算机上运行操作系统的进程，尽管它也可以在云中的一个运行在物理 Linux 服务器上的 VM 中发生，该服务器由许多不同的公司和个人共享。这就是为什么在不同的部署模型之间选择时，隔离级别（在容器的情况下几乎与 VM 一样高）可能是至关重要的。

典型的建议是将一个微服务放入一个容器中，但没有什么阻止你将多个微服务放入一个 Docker 镜像（或者任何其他容器）。然而，在容器管理系统（在容器世界中称为**编排**）中已经有成熟的系统可以帮助你进行部署，因此拥有许多容器的复杂性，虽然是一个有效的考虑因素，但如果韧性和响应性受到威胁，这并不应该是一个大障碍。一个名为**Kubernetes**的流行编排支持微服务注册表、发现和负载均衡。Kubernetes 可以在任何云或私有基础设施中使用。

容器允许在几乎任何当前的部署环境中进行快速、可靠和一致的部署，无论是你自己的基础设施还是亚马逊、谷歌或微软的云。它们还允许应用程序在开发、测试和生产阶段之间轻松移动。这种基础设施的独立性允许你在必要时在开发和测试中使用公共云，而在生产中使用自己的计算机。

一旦创建了基本的操作镜像，每个开发团队都可以在其上构建他们的应用程序，从而避免环境配置的复杂性。容器的版本也可以在版本控制系统中进行跟踪。

使用容器的优势如下：

+   与无容器和自包含部署相比，隔离级别最高。此外，最近还投入了更多的精力来为容器增加安全性。

+   每个容器都由相同的一组命令进行管理、分发、部署、启动和停止。

+   无需预先安装 JRE，也不会出现所需版本不匹配的风险。

+   你可以完全控制容器中包含的依赖关系。

+   通过添加/删除容器实例，很容易扩展/缩小每个微服务。

使用容器的缺点如下：

+   你和你的团队必须学习一整套新的工具，并更深入地参与到生产阶段中。另一方面，这似乎是近年来的一般趋势。

# 总结

微服务是一个新的架构和设计解决方案，用于高负载处理系统，在被亚马逊、谷歌、Twitter、微软、IBM 等巨头成功用于生产后变得流行起来。不过这并不意味着你必须也采用它，但你可以考虑这种新方法，看看它是否能帮助你的应用程序更具韧性和响应性。

使用微服务可以提供实质性的价值，但并非免费。它会带来更多单元的管理复杂性，从需求和开发到测试再到生产的整个生命周期。在承诺全面采用微服务架构之前，通过实施一些微服务并将它们全部移至生产环境来尝试一下。然后，让它运行一段时间并评估经验。这将非常具体化您的组织。任何成功的解决方案都不应盲目复制，而应根据您特定的需求和能力进行采用。

通过逐步改进已经存在的内容，通常可以实现更好的性能和整体效率，而不是通过彻底的重新设计和重构。

在下一课中，我们将讨论并演示新的 API，可以通过使代码更易读和更快速执行来改进您的代码。

# 评估

1.  使用 _________ 对象，可以部署各种垂直，它们彼此交流，接收外部请求，并像任何其他 Java 应用程序一样处理和存储数据，从而形成微服务系统。

1.  容器化部署的以下哪一项是优势？

1.  每个 JAR 文件都需要特定版本或更高版本的 JVM，这可能会迫使您出于这个原因启动一个新的物理或虚拟机，以部署一个特定的 JAR 文件

1.  在您无法控制的环境中，您的代码可能会使用正确版本的 JVM 部署，这可能会导致不可预测的结果

1.  在同一 JVM 中的进程竞争资源，这在由不同团队或不同公司共享的环境中尤其难以管理

1.  它的占地面积很小，因为除了应用程序本身或一组微服务之外，它不包括任何其他东西

1.  判断是 True 还是 False：支持跨多个微服务的事务的一种方法是创建一个扮演并行事务管理器角色的服务。

1.  以下哪些是包含在 Java 9 中的 Java 框架？

1.  Akka

1.  忍者

1.  橙色

1.  Selenium

1.  判断是 True 还是 False：与无容器和自包含部署相比，容器中的隔离级别最高。
