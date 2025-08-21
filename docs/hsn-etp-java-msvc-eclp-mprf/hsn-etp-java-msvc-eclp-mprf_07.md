# 第六章：MicroProfile Metrics 和 OpenTracing

一旦开发人员编写了代码并将其投入生产，就需要观察代码在做什么，表现如何，以及使用了哪些资源。MicroProfile 为解决这些问题创建了两个规范：Metrics 和（与）OpenTracing 的集成。

从 Metrics 部分开始，我们将讨论以下主题：

+   制定规范的依据

+   启用服务器上指标的暴露格式

+   从您的应用程序内部提供指标

+   使用 Prometheus，一个云原生的时间序列数据库，来检索和分析指标数据

在 OpenTracing 部分，我们将讨论以下内容：

+   跟踪领域简介

+   配置属性...

# MicroProfile Metrics

MicroProfile Metrics 暴露运行服务器的指标数据（通常称为**遥测**），例如 CPU 和内存使用情况，以及线程计数。然后，这些数据通常被输入到图表系统中，以随时间可视化指标或为容量规划目的服务；当然，当值超出预定义的阈值范围时，它们也用于通知 DevOps 人员在阈值范围内。

Java 虚拟机长期以来一直通过 MBeans 和 MBeanServer 暴露数据。Java SE 6 见证了所有虚拟机定义如何从远程进程访问 MBean 服务器的（基于 RMI）远程协议的引入。处理此协议是困难的，并且与今天的基于 HTTP 的交互不符。

另一个痛点是许多全球存在的服务器在不同的名称下暴露不同的属性。因此，设置不同类型服务器的监控并不容易。

MicroProfile 通过一个基于 HTTP 的 API，允许监控代理访问，以及一个 Java API，允许在服务器和 JVM 指标之上导出应用程序特定指标，创建了一个监控规范，解决了这两个问题。

MicroProfile Metrics 正在开发 2.x 版本的规范，其中有一些对 1.x 的破坏性变化。以下部分讨论 1.x - 2.0 中的变化在*MP-Metrics 2.0 中的新特性*部分讨论。

规范定义了指标的三种作用域：

+   基础：这些指标主要是 JVM 统计数据，每个合规的供应商都必须支持。

+   供应商：可选的供应商特定指标，这些指标是不可移植的。

+   应用：来自已部署应用程序的可选指标。Java API 将在*提供应用程序特定指标*部分中展示。

经典 JMX 方法的另一个问题，MicroProfile Metrics 解决了，就是关于指标语义的元数据信息不足。

# 元数据

元数据是 MicroProfile Metrics 中的一个非常重要的部分。虽然暴露一个名为`foo`的指标，其值为`142`是可能的，但这个指标并不具有自描述性。看到这个指标的运营商无法判断这是关于什么的，单位是什么，以及`142`是否是一个好值。

元数据用于提供单位和度量的描述，这样前述的现在可以是`foo: runtime; 142`秒。这现在允许在显示上正确缩放至*两分钟和 22 秒*。而收到与这个度量相关的警报的用户可以理解它指的是某些运行时计时。

# 从服务器检索度量

微 Profile 度量通过一个 REST 接口暴露度量指标，默认情况下，在`/metrics`上下文根下。你可以找到代码在[`github.com/PacktPublishing/Hands-On-Enterprise-Java-Microservices-with-Eclipse-MicroProfile/tree/master/Chapter05-metrics`](https://github.com/PacktPublishing/Hands-On-Enterprise-Java-Microservices-with-Eclipse-MicroProfile/tree/master/Chapter05-metrics)。按照`README.md`文件来构建代码，运行它，并用浏览器访问几遍`[`localhost:8080/book-metrics/hello`](http://localhost:8080/book-metrics/hello)`和`[`localhost:8080/book-metrics`](http://localhost:8080/book-metrics)`端点来生成一些数据。

截至 MicroProfile 1.3/2.0，规范中没有关于保护该端点的任何内容。因此留给个别实现自行处理。

使用这个 REST 接口，很容易检索数据，例如，通过以下`curl`命令：

```java
$ curl http://localhost:8080/metrics
```

这个命令显示了 Prometheus 文本格式（缩写）中的 Metrics 1.x 输出：

```java
# TYPE base:classloader_total_loaded_class_count counter
base:classloader_total_loaded_class_count 13752.0 
# TYPE base:cpu_system_load_average gauge
base:cpu_system_load_average 2.796875
# TYPE base:thread_count counter
base:thread_count 76.0
# TYPE vendor:memory_pool_metaspace_usage_max_bytes gauge
vendor:memory_pool_metaspace_usage_max_bytes 7.0916056E7
# TYPE application:hello_time_rate_per_second gauge
application:hello_time_rate_per_second{app="shop",type="timer"} 
3.169298061424996E-10
# TYPE application:hello_time_one_min_rate_per_second gauge
application:hello_time_one_min_rate_per_second{app="shop",type="timer"} 0.0
[...]
```

如果你没有提供媒体类型，默认输出格式是 Prometheus 文本格式（也可以在浏览器中很好地渲染）。Prometheus 格式向值中的`# TYPE`和`# HELP`行暴露附加元数据。你也可以在前一个示例中看到作用域（基本、供应商和应用程序）是如何被添加到实际度量名称之前的。

另外，通过提供一个`HAccept`头（再次缩写）来检索 JSON 格式的数据是可能的：

```java
$ curl -HAccept:application/json http://localhost:8080/metrics
```

这个命令导致以下输出：

```java
{
 "application" :
 {
 "helloTime" : {
 "p50": 1.4884994E7,
 [...]
 "count": 1,
 "meanRate": 0.06189342578194245
 },
 "getCounted" : 1
 },
 "base" :
 {
 "classloader.totalLoadedClass.count" : 13970,
 "cpu.systemLoadAverage" : 2.572265625,
 "gc.PS Scavenge.time" : 290
 },
 "vendor" :
 {
 "test" : 271,
 "memoryPool.Metaspace.usage.max" : 72016928,
 }
```

在这种情况下，纯净数据被暴露出来；作用域构成了一个顶级层次，相应的度量指标被嵌套在其中。可以通过一个 HTTP `XOPTIONS`调用检索匹配的元数据：

```java
$ curl XOPTIONS -HAccept:application/json http://localhost:8080/metrics
```

输出现在包含元数据作为一个映射：

```java
{
"application" : {
 "helloTime": {
 "unit": "nanoseconds",
 "type": "timer",
 "description": "Timing of the Hello call",
 "tags": "app=shop,type=timer",
 "displayName": "helloTime"
 }
}
[...]
}
```

既然我们已经了解了如何检索不同类型的数据和元数据，我们将快速查看如何限制检索到特定的作用域。

# 访问特定作用域

通过在路径后附加作用域名称，也可以只检索单个作用域的数据。在以下示例中，我们只检索基本作用域的度量指标：

```java
$ curl http://localhost:8080/metrics/base
```

现在只显示基本作用域的度量指标：

```java
# TYPE base:classloader_total_loaded_class_count counterbase:classloader_total_loaded_class_count 13973.0# TYPE base:cpu_system_load_average gaugebase:cpu_system_load_average 1.92236328125
```

在本节中，我们看到了如何从启用了 MicroProfile Metrics 的服务器检索度量。基本和供应商作用域中的度量由服务器预定义。应用程序作用域中的度量可以由用户定义，这是我们将在下一节中探索的内容...

# 提供应用程序特定的度量

应用程序可以选择通过 CDI 编程模型暴露度量数据。这个模型受到了 DropWizard Metrics 的启发，以便更容易将应用程序过渡到 MP-Metrics。它还使用了来自 DropWizard Metrics 的注解，这些注解已经被增强以支持元数据。

让我们从一个例子开始，定义一个计数器然后在代码中递增：

```java
@Inject
@Metric(absolute = true, description = "# calls to /health")
Counter hCount; // This is the counter

@GET
@Path("/health")
public Response getHealth() throws Exception {
    hCount.inc(); // It is increased in the application
    [...]
}
```

在这个例子中，我们通过将计数器注入到`hCount`变量中来注册计数器：

`@Metric`注解提供了额外信息，例如描述，同时也指出名称是变量名而不是额外的包名（`absolute=true`）。

在以下示例中，我们让实现来自动计数。这个实现代表了计数一个方法或 REST 端点的常见用例：

```java
@Counted(absolute=true,
        description="# calls to getCounted",
        monotonic=true)
@GET
@Path("/c")
public String getCounted() {
    return "Counted called";
}
```

`@Counted`的`monotonic`属性表示要不断增加计数器，否则当离开方法时它会减少。

# 更多类型的度量

计数器是唯一可以暴露的度量类型，并且经常需要更复杂的类型，例如，记录方法调用持续时间的分布。

我们快速看一下这些。大多数遵循`@Counted`的模式。

# 仪表器

仪表器是一个值任意上升和下降的度量。仪表器总是由一个提供仪表器值的方法支持：

```java
@Gauge
int provideGaugeValue() {
  return 42;  // The value of the gauge is always 42
}
```

仪表器的值在客户端请求值时计算，就像所有其他值一样。这要求实现仪表器方法非常快，以便调用者不会被阻塞。

# 仪表器

仪表器测量随着时间的推移被装饰方法调用的速率。对于 JAX-RS 端点，这就是每秒的请求数。可以通过注解声明仪表器：

```java
@GET@Path("/m")@Metered(absolute = true)public String getMetered() {  return "Metered called";}
```

当客户端请求从仪表器中数据时，服务器提供平均速率，以及一、五、十五分钟的移动平均值。后者对一些读者来说可能熟悉，来自 Unix/Linux 的`uptime`命令。

# 直方图

直方图是一种度量类型，它样本数据的分布。它主要用于记录被装饰方法执行所需时间的分布。直方图不能像其他类型那样通过专用注解声明，但例如，计时器包含直方图数据。要单独使用直方图，您需要在代码中注册并更新它：

```java
// Register the Histogram
@Inject
@Metric(absolute = true)
private Histogram aHistogram;

// Update with a value from 0 to 10
@GET
@Path("/h")
public String getHistogram() {
  aHistogram.update((int) (Math.random() * 10.0));
  return "Histogram called";
}
```

这种在代码中使用度量的方式对其他类型也是可行的。

# 计时器

计时器基本上是直方图和仪表器的组合，也可以通过注解声明：

```java
@GET@Path("/hello")@Timed(name="helloTime", absolute = true,        description = "Timing of the Hello call",       tags={"type=timer","app=shop"})public String getHelloTimed() {  try {    Thread.sleep((long) (Math.random()*200.0));  } catch (InterruptedException e) {     // We don't care if the sleep is interrupted.  }  return "Hello World";}
```

这个例子中的代码等待一小段时间的随机量，使输出更有趣。

# 标记

标签或标签是组织信息的一种方式。这些在 Docker 和 Kubernetes 中变得流行。在 MicroProfile Metrics 1.x 中，它们会被直接传递到输出中，而不会进一步处理，并不能用来区分指标。MicroProfile Metrics 支持服务器级和每个指标的标签，然后会在输出中合并在一起。

# 服务器级标签

服务器级标签是通过环境变量`MP_METRICS_TAGS`设置的，如下所示：

```java
export MP_METRICS_TAGS=app=myShopjava -jar target/metrics-thorntail.jar
```

这些标签将被添加到服务器中定义的所有指标中，并添加到相应的输出格式中。

所以，在之前的命令下，一个名为`@Counted(absolute=true) int myCount;`的计数器，最终会在 Prometheus 中显示如下：

```java
# TYPE application:my_count counterapplication:my_count{app="myShop"} 0
```

# 每个指标的标签

标签也可以基于每个指标提供：

```java
@Counted(tags=[“version=v1”,”commit=abcde”])
void doSomething() {
  [...]
}
```

这个示例在名为`doSomething`的指标上定义了两个标签，`version=v1`和`commit=abcde`。这些将与全局标签合并用于输出。有了之前的全局标签，输出中就会有三个标签。

在本节中，我们了解了如何向指标添加标签以提供附加元数据。这些可以是全局的，适用于服务器暴露的所有指标，也可以是特定于应用程序的，适用于单个指标。

# 使用 Prometheus 检索指标

既然我们已经了解了暴露的指标以及如何定义我们自己的指标，现在让我们来看看我们如何可以在一个**时间序列数据库**（**TSDB**）中收集它们。为此，我们使用 Prometheus，一个 CNCF（[`www.cncf.io/`](https://www.cncf.io/)）项目，在云原生世界中得到了广泛采用。

您可以从[`prometheus.io`](https://prometheus.io/)下载 Prometheus，或者在 macOS 上通过`brew install prometheus`安装。

一旦下载了 Prometheus，我们需要一个配置文件来定义要抓取的目标，然后可以启动服务器。对于我们来说，我们将使用以下简单的文件：

```java
.Prometheus configuration for a Thorntail Server, prom.ymlscrape_configs:# Configuration to poll from Thorntail- job_name: 'thorntail' ...
```

# 新增于 MP-Metrics 2.0

注意：您在读到这些内容时，MicroProfile Metrics 2.0 可能还没有发布，而且内容可能会根据早期用户/实施者的反馈略有变化。

# 对计数器的更改——引入 ConcurrentGauge

在 Metrics 1.x 中，计数器有两个功能：

+   为了提供一个并发调用次数的测量指标

+   作为一个可以计数提交事务数量的指标，例如

不幸的是，当使用没有指定`monotonic`关键词的注解时，第一种方法是默认的，这是出乎意料的，也让很多用户感到困惑。这种方法的第二个版本也有其问题，因为计数器的值也可以随意减少，这违反了计数器是一个单调递增指标的理解。

因此，度量工作组决定更改计数器的行为，使它们只作为单调递增的指标工作，并将推迟...

# 标记

标签现在也用于区分具有相同名称和类型但不同标签的指标。它们可以用来支持许多指标`result_code`在 REST 端点上，以计算(未)成功的调用次数：

```java
@Inject
@Metric(tags="{code,200}", name="result_code")
Counter result_code_200k;

@Inject
@Metric(tags="{code,500}", name="result_code")
Counter result_code_500;

@GET
@Path("/")
public String getData(String someParam) {

 String result = getSomeData(someParam);
 if (result == null ) {
   result_code_500.inc();
 } else {
   result_code_200.inc();
 }
 return result;
}
```

在底层，指标不再仅按名称和类型进行标识，还按它们的标签进行标识。为此，引入了新的`MetricID`来容纳名称和标签。

# 数据输出格式的变化

在 MicroProfile Metrics 2.0 中引入多标签指标需要对提供给客户端的指标数据格式进行更改。

Prometheus 格式也存在一些不一致之处，因此我们决定以有时不兼容的方式重新设计这些格式：

+   作用域和指标名称之间的冒号(:)分隔符已更改为下划线(_)。

+   Prometheus 输出格式不再需要将 camelCase 转换为 snake_case。

+   垃圾收集器的基础指标格式已更改，现在使用各种垃圾收集器的标签。

请参阅 MicroProfile 2.0 规范中的发布说明：[`github.com/eclipse/microprofile-metrics/releases/tag/2.0 ...`](https://github.com/eclipse/microprofile-metrics/releases/tag/2.0)

# MicroProfile OpenTracing

在微服务构成的现代世界中，一个请求可能会穿过在不同机器、数据中心，甚至地理位置上运行的多个进程。

此类系统的可观测性是一项具有挑战性的任务，但当正确实现时，它允许我们*讲述每个单独请求的故事*，而不是从指标和日志等信号派生出的系统的总体状态。在本章中，我们将向您介绍分布式追踪，并解释 MicroProfile OpenTracing 1.3 中的 OpenTracing 集成。

在前一节中，我们学习了关于指标以及它们如何观察应用程序或每个单独的组件。这些信息无疑非常有价值，并为系统提供了宏观视图，但同时，它很少提到穿越多个组件的每个单独请求。分布式追踪提供了关于请求端到端发生的微观视图，使我们能够回顾性地理解应用程序的每个单独组件的行为。

分布式追踪是基于动作的；换句话说，它记录了系统中的所有与动作相关的信息。例如，它捕获了请求的详细信息及其所有因果相关活动。我们不会详细介绍这种追踪是如何工作的，但简而言之，我们可以得出以下结论：

+   追踪基础架构为每个请求附加了上下文元数据，通常是唯一标识符集合——`traceId`、`spanId`和`parentId`。

+   观测层记录剖析数据并在进程内部及进程之间传播元数据。

+   捕获的剖析数据包含元数据和对先前事件的因果引用。

根据捕获的数据，分布式跟踪系统通常提供以下功能：

+   根本原因分析

+   延迟优化——关键路径分析

+   分布式上下文传播——行李

+   上下文化日志记录

+   服务依赖分析

在我们深入探讨 MicroProfile OpenTracing 之前，让我们简要地看看 OpenTracing，以便我们能更好地理解它提供的 API 是什么。

# OpenTracing 项目

OpenTracing 项目（[`opentracing.io`](https://opentracing.io/)）提供了一个中立的规范[(https://github.com/opentracing/specification)](https://github.com/opentracing/specification)和多语言 API，用于描述分布式事务。中立性很重要，因为在大规模组织中启用分布式跟踪时，代码 instrumentation 是最耗时和最具挑战性的任务。我们想强调的是 OpenTracing 只是一个 API。实际部署将需要一个运行在监控进程内部的 plugged 跟踪器实现，并将数据发送到跟踪系统。

从 API 角度来看，有三个关键概念：跟踪器、跨度、和跨度上下文。跟踪器是应用程序中可用的单例对象，可以用来建模一个...

# 配置属性

OpenTracing 是中立且可以与使用此 API 的任何供应商的跟踪实现配合使用。每个跟踪器实现将配置不同。因此，配置超出了 MicroProfile OpenTracing 规范的范围。然而，规范本身暴露了几个配置属性，以调整跟踪范围或生成数据。配置利用了 MicroProfile Config 规范，为所有支持的配置选项提供了一种一致的方法。

目前，规范暴露了以下内容：

+   `mp.opentracing.server.skip-pattern`：一个跳过模式，用于避免跟踪选定的 REST 端点。

+   `mp.opentracing.server.operation-name-provider`：这指定了服务器跨度操作名称提供程序。可能的值有`http-path`和`class-method`。默认值是`class-method`，它完全使用一个限定类名与方法名拼接；例如，`GET:org.eclipse.Service.get`。`http-path`使用`@Path`注解的值作为操作名称。

# 自动 instrumentation

这里的动机是自动捕获所有关键性能信息，并在运行时之间自动传播跟踪上下文。第二部分尤其重要，因为它确保了跟踪不会中断，我们能够调查端到端的调用。为了成功跟踪，必须在运行时之间的每种通信技术上进行 instrumentation。在 MicroProfile 的情况下，是 JAX-RS 和 MicroProfile Rest Client。

# JAX-RS

微 Profile OpenTracing 自动追踪所有入站的 JAX-RS 端点。然而，JAX-RS 客户端一侧更加复杂，需要调用注册 API，`org.eclipse.microprofile.opentracing.ClientTracingRegistrar.configure(ClientBuilder clientBuilder)`，以添加追踪能力。微 Profile 实现可以为所有客户端接口全局启用追踪；然而，建议使用注册 API 以获得更好的可移植性。

可以通过禁用特定请求的追踪或更改生成的服务器跨度的操作名称来修改默认的追踪行为。有关更多信息，请在本章后面的*配置属性*部分查阅。instrumentation 层自动向每个跨度添加以下请求范围的信息：

+   `http.method`：请求的 HTTP 方法。

+   `http.status_code`：请求的状态代码。

+   `http.url`：请求的 URL。

+   `component`：被 instrumented 组件的名称，`jaxrs`。

+   `span.kind`：客户端或服务器。

+   `error` – `true` 或 `false`。这是可选的，如果存在，instrumentation 还将在跨度日志中添加一个异常作为 `error.object`。

所有这些标签都可以用于通过追踪系统用户界面查询数据，或者它们可以用于许多追踪系统提供数据分析作业。可以通过注入的追踪器实例向当前活动的跨度添加额外元数据。这可以在过滤器中全局执行或在 rest 处理程序中局部执行，如下面的代码示例所示，通过向服务器跨度添加用户代理头（1）：

```java
@Path("/")
public class JaxRsService {
   @Inject
   private io.opentracing.Tracer tracer;

   @GET
   @Path("/hello")
   @Traced(operationName="greeting") (2)
   public String hello(@HeaderParam("user-agent") String userAgent) {
       tracer.activeSpan().setTag("user-agent", userAgent); (1)
   }
}
```

默认情况下，服务器端跨度操作名称为 `http_method:package.className.method`。然而，这可以通过使用 `@Traced` 注解（2）或通过配置属性（参考配置部分）在本地或全局更改。

# 微 Profile Rest Client

如前一部分所述，所有 REST 客户端接口默认情况下都会自动追踪，无需额外的配置。要更改此行为，请将 `@Traced` 注解应用于接口或方法以禁用追踪。当应用于接口时，所有方法都将从追踪中跳过。请注意，追踪上下文不会被传播。因此，如果请求继续到 instrumented runtime，将开始新的追踪。

# 显式 instrumentation

有时，自动 instrumentation 并没有捕获所有关键的计时信息，因此需要额外的追踪点。例如，我们希望追踪业务层的调用或初始化由 OpenTracing 项目提供的三方 instrumentation（[`github.com/opentracing-contrib`](https://github.com/opentracing-contrib)）。

显式地 instrumentation 可以通过三种方式进行：

+   在**上下文和依赖注入**（**CDI**）bean 上添加 `@Traced` 注解。

+   注入追踪器并手动创建跨度。

+   初始化第三方仪器。外部仪器的初始化取决于其自己的初始化要求。MicroProfile 只需要提供一个跟踪器实例，这在之前的要点中已经涵盖。

让我们现在详细讨论这些内容。

# @Traced 注解

MicroProfile OpenTracing 定义了一个`@Traced`注解，可以用于启用 CDI 豆的跟踪，或禁用自动跟踪接口的跟踪。该注解还可以用于重写其他自动跟踪组件的操作名称——JAX-RS 端点。

下面的代码示例显示了如何使用`@Traced`注解来启用 CDI 豆的跟踪。`(1)`为豆定义的所有方法启用跟踪。`(2)`重写了默认操作名称（`package.className.method`）为`get_all_users`。`(3)`禁用了健康方法的跟踪：

```java
@Traced (1)@ApplicationScopedpublic class Service {   @Traced(operationName = "get_all_users") (2)   public void getUsers() {        // business code   } @Traced(false) (3) ...
```

# 跟踪器注入

应用程序可以注入一个`io.opentracing.Tracer`豆，暴露出完整的 OpenTracing API。这允许应用程序开发者利用更高级的使用案例，例如向当前活动的跨度添加元数据，手动创建跨度，使用行李进行上下文传播，或初始化额外的第三方仪器。

下面的代码显示了如何使用跟踪器将数据附加到当前活动的跨度，`(1)`：

```java
@Path("/")
public class Service {
    @Inject
    private Tracer tracer;

    @GET
    @Path("")
    @Produces(MediaType.TEXT_PLAIN)
    public String greeting() {
       tracer.activeSpan()
           .setTag("greeting", "hello"); (1)
       return "hello";
   }
}
```

这可以用于向跨度添加业务相关数据，但也用于记录异常或其他分析信息。

# 使用 Jaeger 进行跟踪

到目前为止，我们只谈论了仪器仪表的不同方面。然而，要运行完整的跟踪基础设施，我们需要一个跟踪后端。在本节中，我们将使用 Jaeger([`www.jaegertracing.io/`](https://www.jaegertracing.io/))来展示收集的跟踪数据如何在跟踪系统中呈现。我们选择 Jaeger 是因为 Thorntail 提供了与 Jaeger 的直接集成。其他供应商可以提供与其他系统的集成，例如 Zipkin 和 Instana。几乎每个跟踪系统都提供了一个类似于甘特图的视图（或时间线）来查看跟踪。这种视图对于跟踪新手来说可能有些令人不知所措，但它是一个分析分布式系统中调用的系统化工具。

下面的屏幕快照显示了...

# 总结

在本章中，我们学习了关于服务器和应用程序的可观测性。

指标，或遥测，可以帮助确定服务器或应用程序的性能特性。MicroProfile 通过 Metrics 规范提供了一种以标准化方式导出指标的方法。应用程序编写者可以使用 MicroProfile Metrics 将他们的数据以注解或通过调用 Metrics API 的方式装饰性地暴露给监控客户端。

本章进一步解释了 MicroProfile 中 OpenTracing 集成如何为通过系统的每个单独事务提供端到端的视图。我们讨论了配置属性，展示了 JAX-RS 的跟踪，最后调查了 Jaeger 系统中的数据。

在下一章，我们将学习如何通过 OpenAPI 文档化（REST）API，并通过类型安全的 REST 客户端调用这些 API。

# 问题

1.  分布式追踪和指标之间的区别是什么？

1.  分布式追踪系统通常提供哪些功能？

1.  在 MicroProfile OpenTracing 中，系统哪些部分会自动被追踪？

1.  MicroProfile OpenTracing 为每个 REST 请求添加了哪些标签？

1.  如何在业务代码中添加显式 instrumentation？

1.  Metrics 中的作用域是什么，它们的理由是什么？

1.  什么决定了 REST 请求到 Metrics API 的输出格式？

1.  用户应用程序中可用的哪些方法可以导出指标？
