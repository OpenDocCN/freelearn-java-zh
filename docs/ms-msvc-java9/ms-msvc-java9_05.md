# 第五章：部署和测试

在本章中，我们将接着第四章《实现微服务》的内容继续讲解。我们将向仅依赖于三个功能性服务（餐厅、用户和预订服务）以及 Eureka（服务发现和注册）的在线桌位预订系统（OTRS）应用程序添加一些更多服务，以创建一个完全功能的微服务堆栈。这个堆栈将包括网关（Zuul）、负载均衡（Ribbon 与 Zuul 和 Eureka）、监控（Hystrix、Turbine 和 Hystrix 仪表板）。你希望拥有组合 API，并了解一个微服务如何与其他微服务通信。本章还将解释如何使用 Docker 容器化微服务，以及如何使用`docker-compose`一起运行多个容器。在此基础上，我们还将添加集成测试。

在本章中，我们将介绍以下主题：

+   使用 Netflix OSS 的微服务架构概述

+   边缘服务器

+   负载均衡微服务

+   断路器和监控

+   使用容器部署微服务

+   使用 Docker 容器进行微服务集成测试

# 良好的微服务所需的强制服务

为了实现基于微服务的架构设计，应该有一些模式/服务需要到位。这个列表包括以下内容：

+   服务发现和注册

+   边缘或代理服务器

+   负载均衡

+   断路器

+   监控

我们将在本章实现这些服务，以完成我们的 OTRS 系统。以下是简要概述。我们稍后详细讨论这些模式/服务。

# 服务发现和注册

Netflix Eureka 服务器用于服务发现和注册。我们在上一章创建了 Eureka 服务。它不仅允许你注册和发现服务，还提供使用 Ribbon 的负载均衡。

# 边缘服务器

边缘服务器提供一个单一的访问点，允许外部世界与你的系统交互。你的所有 API 和前端都只能通过这个服务器访问。因此，这些也被称为网关或代理服务器。这些被配置为将请求路由到不同的微服务或前端应用程序。在 OTRS 应用程序中，我们将使用 Netflix Zuul 服务器作为边缘服务器。

# 负载均衡

Netflix Ribbon 用于负载均衡。它与 Zuul 和 Eureka 服务集成，为内部和外部调用提供负载均衡。

# 断路器

一个故障或断裂不应该阻止你的整个系统运行。此外，一个服务或 API 的反复失败应该得到适当的处理。断路器提供了这些功能。Netflix Hystrix 作为断路器使用，有助于保持系统运行。

# 监控

使用 Netflix Hystrix 仪表板和 Netflix Turbine 进行微服务监控。它提供了一个仪表板，用于检查运行中微服务的状态。

# 使用 Netflix OSS 的微服务架构概述

Netflix 是微服务架构的先驱。他们是第一个成功在大规模实施微服务架构的人。他们还通过将大部分微服务工具开源，并命名为 Netflix **开源软件中心**（**OSS**），极大地提高了微服务的普及程度并做出了巨大贡献。

根据 Netflix 的博客，当 Netflix 开发他们的平台时，他们使用了 Apache Cassandra 进行数据存储，这是一个来自 Apache 的开源工具。他们开始通过修复和优化扩展为 Cassandra 做贡献。这导致了 Netflix 看到将 Netflix 项目以 OSS 的名义发布的益处。

Spring 抓住了机会，将许多 Netflix 的开源项目（如 Zuul、Ribbon、Hystrix、Eureka 服务器和 Turbine）集成到 Spring Cloud 中。这是 Spring Cloud 能够为生产就绪的微服务提供现成平台的原因之一。

现在，让我们来看看几个重要的 Netflix 工具以及它们如何在微服务架构中发挥作用：

![](img/451c2530-d3dd-4dca-ab1e-04a6f2c8c0a0.jpg)

微服务架构图

正如您在前面的图表中所看到的，对于每一种微服务实践，我们都有一个与之相关的 Netflix 工具。我们可以通过以下映射来了解它。详细信息在本章的相应部分中介绍，关于 Eureka 的部分在最后一章中有详细说明：

+   **边缘服务器**：我们使用 Netflix Zuul 服务器作为边缘服务器。

+   **负载均衡**：Netflix Ribbon 用于负载均衡。

+   **断路器**：Netflix Hystrix 用作断路器，有助于保持系统运行。

+   **服务发现与注册**：Netflix Eureka 服务器用于服务发现和注册。

+   **监控仪表板**：Hystrix 监控仪表板与 Netflix Turbine 配合使用，用于微服务监控。它提供了一个仪表板，用于检查运行中微服务的状态。

# 负载均衡

负载均衡是服务于请求的方式，以最大化速度和容量利用率，并确保没有服务器因请求过多而超载。负载均衡器还将请求重定向到其他主机服务器，如果服务器宕机的话。在微服务架构中，微服务可以服务于内部或外部请求。基于这一点，我们可以有两种类型的负载均衡——客户端负载均衡和服务器端负载均衡。

# 服务器端负载均衡

我们将讨论服务器端负载均衡；在那之前，我们先讨论路由。从微服务架构的角度来看，为我们的 OTRS 应用程序定义路由机制是很重要的。例如，`/`（根）可以映射到我们的 UI 应用程序。同样，`/restaurantapi`和`/userapi`可以分别映射到餐厅服务和用户服务。边缘服务器也执行带有负载均衡的路由。

我们将使用 Netflix Zuul 服务器作为我们的边缘服务器。Zuul 是一个基于 JVM 的路由和服务器端负载均衡器。Zuul 支持用任何 JVM 语言编写规则和过滤器，并内置了对 Java 和 Groovy 的支持。

Netflix Zuul 默认具有发现客户端（Eureka 客户端）支持。Zuul 还利用 Ribbon 和 Eureka 进行负载均衡。

外部世界（UI 和其他客户端）调用边缘服务器，使用`application.yml`中定义的路线调用内部服务并提供响应。如果您认为它充当代理服务器，为内部网络承担网关责任，并且为定义和配置的路线调用内部服务，那么您的猜测是正确的。

通常，建议对所有请求使用单个边缘服务器。然而，一些公司为了扩展，每个客户端使用一个边缘服务器。例如，Netflix 为每种设备类型使用一个专用的边缘服务器。

在下一章中，我们配置和实现微服务安全时，也将使用边缘服务器。

在 Spring Cloud 中配置和使用边缘服务器相当简单。您需要执行以下步骤：

1.  在`pom.xml`文件中定义 Zuul 服务器依赖项：

```java
<dependency> 
      <groupId>org.springframework.cloud</groupId> 
      <artifactId>spring-cloud-starter-zuul</artifactId> 
</dependency> 
```

1.  在您的应用程序类中使用`@EnableZuulProxy`注解。它还内部使用`@EnableDiscoveryClient`注解；因此，它也会自动注册到 Eureka 服务器。您可以在*客户端负载均衡部分*的图中找到注册的 Zuul 服务器。

1.  更新`application.yml`文件中的 Zuul 配置，如下所示：

+   `zuul:ignoredServices`：这跳过了服务的自动添加。我们可以在这里定义服务 ID 模式。`*`表示我们忽略所有服务。在下面的示例中，除了`restaurant-service`，所有服务都被忽略。

+   `Zuul.routes`：这包含定义 URI 模式的`path`属性。在这里，`/restaurantapi`通过`serviceId`属性映射到`restaurant-service`。`serviceId`属性代表 Eureka 服务器中的服务。如果未使用 Eureka 服务器，可以使用 URL 代替服务。我们还使用了`stripPrefix`属性来去除前缀（`/restaurantapi`），结果`/restaurantapi/v1/restaurants/1`调用转换为在调用服务时`/v1/restaurants/1`:

```java
application.yml 
info: 
    component: Zuul Server 
# Spring properties 
spring: 
  application: 
     name: zuul-server  # Service registers under this name 

endpoints: 
    restart: 
        enabled: true 
    shutdown: 
        enabled: true 
    health: 
        sensitive: false 

zuul: 
    ignoredServices: "*" 
    routes: 
        restaurantapi: 
            path: /restaurantapi/** 
            serviceId: restaurant-service 
            stripPrefix: true 

server: 
    port: 8765 

# Discovery Server Access 
eureka: 
  instance: 
    leaseRenewalIntervalInSeconds: 3 
    metadataMap: 
      instanceId: ${vcap.application.instance_id:${spring.application.name}:${spring.application.instance_id:${random.value}}} 
    serviceUrl: 
      defaultZone: http://localhost:8761/eureka/ 
    fetchRegistry: false 
```

请注意，Eureka 应用程序只在每台主机上注册任何服务的单个实例。您需要为`metadataMap.instanceid`使用以下值，以便在同一台主机上注册同一应用程序的多个实例，以便负载均衡工作：

`${spring.application.name}:${vcap.application.instance_id:${spring.application.instance_id:${random.value}}}`

让我们看看一个工作的边缘服务器。首先，我们将按照以下方式调用端口`3402`上部署的餐厅服务：

![](img/96fe451a-e8c4-48fc-b80a-a09784c1795b.png)

直接调用餐厅服务

然后，我们将使用部署在端口`8765`的边缘服务器调用同一服务。你可以看到，调用`/v1/restaurants?name=o`时使用了`/restaurantapi`前缀，并且给出了相同的结果：

![](img/6202f3a7-e83f-4f7f-a992-5f153e395f66.png)

使用边缘服务器调用餐厅服务

# 客户端负载均衡

微服务需要进程间通信，以便服务能够相互通信。Spring Cloud 使用 Netflix Ribbon，这是一个客户端负载均衡器，扮演着这一关键角色，并可以处理 HTTP 和 TCP。Ribbon 是云兼容的，并提供了内置的故障弹性。Ribbon 还允许你使用多个可插拔的负载均衡规则。它将客户端与负载均衡器集成在一起。

在上一章中，我们添加了 Eureka 服务器。Spring Cloud 默认通过 Ribbon 与 Eureka 服务器集成。这种集成提供了以下功能：

+   当使用 Eureka 服务器时，你不需要硬编码远程服务器 URL 进行发现。这是一个显著的优势，尽管如果你需要，你仍然可以使用`application.yml`文件中配置的服务器列表（`listOfServers`）。

+   服务器列表从 Eureka 服务器获取。Eureka 服务器用`DiscoveryEnabledNIWSServerList`接口覆盖了`ribbonServerList`。

+   查找服务器是否运行的请求被委托给 Eureka。这里使用了`DiscoveryEnabledNIWSServerList`接口来代替 Ribbon 的`IPing`。

在 Spring Cloud 中，使用 Ribbon 有不同的客户端可供选择，比如`RestTemplate`或`FeignClient`。这些客户端使得微服务之间能够相互通信。当使用 Eureka 服务器时，客户端使用实例 ID 代替主机名和端口来对服务实例进行 HTTP 调用。客户端将服务 ID 传递给 Ribbon，然后 Ribbon 使用负载均衡器从 Eureka 服务器中选择实例。

如以下屏幕截图所示，如果 Eureka 中有多个服务实例可用，Ribbon 根据负载均衡算法只为请求选择一个：

![](img/ff86c7a3-c57c-4228-b6a2-5b87de6eacea.png)

多服务注册 - 餐厅服务

我们可以使用`DiscoveryClient`来查找 Eureka 服务器中所有可用的服务实例，如下面的代码所示。`DiscoveryClientSample`类中的`getLocalServiceInstance()`方法返回 Eureka 服务器中所有可用的本地服务实例。

这是一个`DiscoveryClient`示例：

```java
@Component 
class DiscoveryClientSample implements CommandLineRunner { 

    @Autowired 
    private DiscoveryClient; 

    @Override 
    public void run(String... strings) throws Exception { 
        // print the Discovery Client Description 
        System.out.println(discoveryClient.description()); 
        // Get restaurant-service instances and prints its info 
        discoveryClient.getInstances("restaurant-service").forEach((ServiceInstance serviceInstance) -> { 
            System.out.println(new StringBuilder("Instance --> ").append(serviceInstance.getServiceId()) 
                    .append("\nServer: ").append(serviceInstance.getHost()).append(":").append(serviceInstance.getPort()) 
                    .append("\nURI: ").append(serviceInstance.getUri()).append("\n\n\n")); 
        }); 
    } 
} 
```

当执行此代码时，它会打印以下信息。它显示了餐厅服务的两个实例：

```java
Spring Cloud Eureka Discovery Client 
Instance: RESTAURANT-SERVICE 
Server: SOUSHARM-IN:3402 
URI: http://SOUSHARM-IN:3402 
Instance --> RESTAURANT-SERVICE 
Server: SOUSHARM-IN:3368 
URI: http://SOUSHARM-IN:3368 
```

下面的示例展示了这些客户端如何使用。你可以在两个客户端中看到，服务名称`restaurant-service`被用来代替服务主机名和端口。这些客户端调用`/v1/restaurants`来获取包含在名称查询参数中的餐厅名称的餐厅列表。

这是一个`RestTemplate`示例：

```java
@Component
class RestTemplateExample implements CommandLineRunner {
  @Autowired
  private RestTemplate restTemplate;
  @Override
  public void run(String... strings) throws Exception {
    System.out.println("\n\n\n start RestTemplate client...");
    ResponseEntity<Collection<Restaurant>> exchange
    = this.restTemplate.exchange(
    "http://restaurant-service/v1/restaurants?name=o",
    HttpMethod.GET,
    null,
    new ParameterizedTypeReference<Collection<Restaurant>>() {
    },
    (Object) "restaurants");
    exchange.getBody().forEach((Restaurant restaurant) -> {
      System.out.println("\n\n\n[ " + restaurant.getId() + " " +  restaurant.getName() + "]");
      });
   }
}
```

这是一个`FeignClient`示例：

```java
@FeignClient("restaurant-service")
interface RestaurantClient {
  @RequestMapping(method = RequestMethod.GET, value =  "/v1/restaurants")
  Collection<Restaurant> getRestaurants(@RequestParam("name") String name);
  }
@Component
class FeignSample implements CommandLineRunner {
  @Autowired
  private RestaurantClient restaurantClient;
  @Override
  public void run(String... strings) throws Exception {
    this.restaurantClient.getRestaurants("o").forEach((Restaurant     restaurant) -> {
      System.out.println("\n\n\n[ " + restaurant.getId() + " " +  restaurant.getName() + "]");
      });
    }
} 
```

所有前面的示例都将打印以下输出：

```java
[ 1 Big-O Restaurant] 
[ 2 O Restaurant] 
```

为了演示目的，我们在边缘应用程序主类 Java 文件中添加了所有客户端—`discovery`客户端、`RestTemplate`客户端和`FeignClient`。由于我们所有这些客户端都实现了`CommandLineRunner`接口，这会在边缘应用程序服务启动后立即执行。

# 断路器与监控

通常而言，断路器是一种*自动装置，用于在电气电路中作为安全措施停止电流的流动*。

同样的概念也用于微服务开发，称为**断路器**设计模式。它跟踪外部服务的可用性，如 Eureka 服务器、API 服务如`restaurant-service`等，并防止服务消费者对任何不可用的服务执行任何操作。

这是微服务架构的另一个重要方面，一种安全措施

（安全机制）当服务消费者对服务的调用没有响应时，这称为断路器。

我们将使用 Netflix Hystrix 作为断路器。当发生故障时（例如，由于通信错误或超时），它在服务消费者内部调用回退方法。它在服务消费者内执行。在下一节中，您将找到实现此功能的代码。

Hystrix 在服务未能响应时打开电路，并在服务再次可用之前快速失败。当对特定服务的调用达到一定阈值（默认阈值是五秒内 20 次失败），电路打开，调用不再进行。您可能想知道，如果 Hystrix 打开电路，那么它是如何知道服务可用的？它异常地允许一些请求调用服务。

# 使用 Hystrix 的回退方法

实现回退方法有五个步骤。为此，我们将创建另一个服务，`api-service`，就像我们创建其他服务一样。`api-service`服务将消费其他服务，如`restaurant-service`等，并将在边缘服务器中配置以对外暴露 OTRS API。这五个步骤如下：

1.  **启用断路器**：主要消费其他服务的微服务类应该用`@EnableCircuitBreaker`注解标记。因此，我们将注释`src\main\java\com\packtpub\mmj\api\service\ApiApp.java`：

```java
@SpringBootApplication 
@EnableCircuitBreaker 
@ComponentScan({"com.packtpub.mmj.user.service", "com.packtpub.mmj.common"}) 
public class ApiApp { 
```

1.  **配置回退方法**：用`@HystrixCommand`注解来配置`fallbackMethod`。我们将注释控制器方法来配置回退方法。这是文件：`src\main\java\com\packtpub\mmj\api\service\restaurant\RestaurantServiceAPI.java`：

```java
@HystrixCommand(fallbackMethod = "defaultRestaurant") 
    @RequestMapping("/restaurants/{restaurant-id}") 
    @HystrixCommand(fallbackMethod = "defaultRestaurant") 
    public ResponseEntity<Restaurant> getRestaurant( 
            @PathVariable("restaurant-id") int restaurantId) { 
        MDC.put("restaurantId", restaurantId); 
        String url = "http://restaurant-service/v1/restaurants/" + restaurantId; 
        LOG.debug("GetRestaurant from URL: {}", url); 

        ResponseEntity<Restaurant> result = restTemplate.getForEntity(url, Restaurant.class); 
        LOG.info("GetRestaurant http-status: {}", result.getStatusCode()); 
        LOG.debug("GetRestaurant body: {}", result.getBody()); 

        return serviceHelper.createOkResponse(result.getBody()); 
    }  
```

1.  **定义回退方法**：处理失败并执行安全步骤的方法。在这里，我们只是添加了一个示例；这可以根据我们想要处理失败的方式进行修改：

```java
public ResponseEntity<Restaurant> defaultRestaurant(
@PathVariable int restaurantId) { 
  return serviceHelper.createResponse(null, HttpStatus.BAD_GATEWAY); 
  } 
```

1.  **Maven 依赖项**：我们需要在`pom.xml`中为 API 服务或希望确保 API 调用的项目中添加以下依赖项：

```java
<dependency> 
    <groupId>org.springframework.cloud</groupId> 
    <artifactId>spring-cloud-starter-hystrix</artifactId> 
</dependency> 
```

1.  **在`application.yml`中配置 Hystrix**：我们将在我们的`application.yml`文件中添加以下 Hystrix 属性：

```java
       hystrix: 
  threadpool: 
    default: 
      # Maximum number of concurrent requests when using thread pools (Default: 10) 
      coreSize: 100 
      # Maximum LinkedBlockingQueue size - -1 for using SynchronousQueue (Default: -1) 
      maxQueueSize: -1 
      # Queue size rejection threshold (Default: 5) 
      queueSizeRejectionThreshold: 5 
  command: 
    default: 
      circuitBreaker: 
        sleepWindowInMilliseconds: 30000 
        requestVolumeThreshold: 2 
      execution: 
        isolation: 
#          strategy: SEMAPHORE, no thread pool but timeout handling stops to work 
          strategy: THREAD 
          thread: 
            timeoutInMilliseconds: 6000
```

这些步骤应该足以确保服务调用的安全，并向服务消费者返回一个更合适的响应。

# 监控

Hystrix 提供了一个带有 web UI 的仪表板，提供很好的电路断路器图形：

![](img/88b20511-ffd5-4562-979d-50210d86d46c.jpg)

默认的 Hystrix 仪表板

Netflix Turbine 是一个 web 应用程序，它连接到 Hystrix 应用程序集群的实例并聚合信息，实时进行（每 0.5 秒更新一次）。Turbine 使用称为 Turbine 流的流提供信息。

如果你将 Hystrix 与 Netflix Turbine 结合使用，那么你可以在 Hystrix 仪表板上获取 Eureka 服务器上的所有信息。这为你提供了有关所有电路断路器的信息的全景视图。

要使用 Turbine 和 Hystrix，只需在前面截图中的第一个文本框中输入 Turbine 的 URL`http://localhost:8989/turbine.stream`（在`application.yml`中为 Turbine 服务器配置了端口`8989`），然后点击监控流。

Netflix Hystrix 和 Turbine 使用 RabbitMQ，这是一个开源的消息队列软件。RabbitMQ 基于**高级消息队列协议**（**AMQP**）工作。这是一个软件，在此软件中可以定义队列并由连接的应用程序交换消息。消息可以包含任何类型的信息。消息可以存储在 RabbitMQ 队列中，直到接收应用程序连接并消耗消息（将消息从队列中移除）。

Hystrix 使用 RabbitMQ 将度量数据发送到 Turbine。

在配置 Hystrix 和 Turbine 之前，请在你的平台上演示安装 RabbitMQ 应用程序。Hystrix 和 Turbine 使用 RabbitMQ 彼此之间进行通信。

# 设置 Hystrix 仪表板

我们将在 IDE 中创建另一个项目，以与创建其他服务相同的方式创建 Hystrix 仪表板。在这个新项目中，我们将添加新的 Maven 依赖项`dashboard-server`，用于 Hystrix 服务器。在 Spring Cloud 中配置和使用 Hystrix 仪表板相当简单。

当你运行 Hystrix 仪表板应用程序时，它会看起来像前面所示的默认 Hystrix 仪表板快照。你只需要按照以下步骤操作：

1.  在`pom.xml`文件中定义 Hystrix 仪表板依赖项：

```java
<dependency> 
    <groupId>org.springframework.cloud</groupId> 
    <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId> 
</dependency> 
```

1.  在主 Java 类中的`@EnableHystrixDashboard`注解为您使用它做了所有事情。我们还将使用`@Controller`将根 URI 的请求转发到 Hystrix 仪表板 UI URI（`/hystrix`），如下所示：

```java
@SpringBootApplication 
@Controller 
@EnableHystrixDashboard 
public class DashboardApp extends SpringBootServletInitializer { 

    @RequestMapping("/") 
    public String home() { 
        return "forward:/hystrix"; 
    } 

    @Override 
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) { 
        return application.sources(DashboardApp.class).web(true); 
    } 

    public static void main(String[] args) { 
        SpringApplication.run(DashboardApp.class, args); 
    } 
} 
```

1.  如所示更新`application.yml`中的仪表板应用程序配置：

```java
# Hystrix Dashboard properties 
spring: 
    application: 
        name: dashboard-server 

endpoints: 
    restart: 
        enabled: true 
    shutdown: 
        enabled: true 

server: 
    port: 7979 

eureka: 
    instance: 
        leaseRenewalIntervalInSeconds: 3 
        metadataMap: 
            instanceId: ${vcap.application.instance_id:${spring.application.name}:${spring.application.instance_id:${random.value}}} 

    client: 
        # Default values comes from org.springframework.cloud.netflix.eurek.EurekaClientConfigBean 
        registryFetchIntervalSeconds: 5 
        instanceInfoReplicationIntervalSeconds: 5 
        initialInstanceInfoReplicationIntervalSeconds: 5 
        serviceUrl: 
            defaultZone: http://localhost:8761/eureka/ 
        fetchRegistry: false 

logging: 
    level: 
        ROOT: WARN 
        org.springframework.web: WARN 
```

# 创建 Turbine 服务

Turbine 将所有`/hystrix.stream`端点聚合成一个合并的`/turbine.stream`，以供 Hystrix 仪表板使用，这更有助于查看系统的整体健康状况，而不是使用`/hystrix.stream`监视各个服务。我们将在 IDE 中创建另一个服务项目，然后在`pom.xml`中为 Turbine 添加 Maven 依赖项。

现在，我们将使用以下步骤配置 Turbine 服务器：

1.  在`pom.xml`中定义 Turbine 服务器的依赖项：

```java
<dependency> 
    <groupId> org.springframework.cloud</groupId> 
    <artifactId>spring-cloud-starter-turbine-stream</artifactId> 
</dependency> 
<dependency> 
     <groupId>org.springframework.cloud</groupId> 
     <artifactId>spring-cloud-starter-stream-rabbit</artifactId> 
</dependency> 
<dependency> 
     <groupId>org.springframework.boot</groupId> 
     <artifactId>spring-boot-starter-actuator</artifactId> 
</dependency> 

```

1.  在您的应用程序类中使用`@EnableTurbineStream`注解，如

    此处显示。我们还定义了一个将返回 RabbitMQ `ConnectionFactory`的 Bean：

```java
@SpringBootApplication 
@EnableTurbineStream 
@EnableEurekaClient 
public class TurbineApp { 

    private static final Logger LOG = LoggerFactory.getLogger(TurbineApp.class); 

    @Value("${app.rabbitmq.host:localhost}") 
    String rabbitMQHost; 

    @Bean 
    public ConnectionFactory connectionFactory() { 
        LOG.info("Creating RabbitMQHost ConnectionFactory for host: {}", rabbitMQHost); 
        CachingConnectionFactory cachingConnectionFactory = new CachingConnectionFactory(rabbitMQHost); 
        return cachingConnectionFactory; 
    } 

    public static void main(String[] args) { 
        SpringApplication.run(TurbineApp.class, args); 
    } 
} 
```

1.  根据下面所示，更新`application.yml`中的 Turbine 配置：

+   `server:port`：Turbine HTTP 使用的主要端口

+   `management:port`：Turbine 执行器端点的端口：

```java
application.yml 
spring: 
    application: 
        name: turbine-server 

server: 
    port: 8989 

management: 
    port: 8990 

turbine: 
    aggregator: 
        clusterConfig: USER-SERVICE,RESTAURANT-SERVICE 
    appConfig: user-service,restaurant-service  

eureka: 
    instance: 
        leaseRenewalIntervalInSeconds: 10 
        metadataMap: 
            instanceId: ${vcap.application.instance_id:${spring.application.name}:${spring.application.instance_id:${random.value}}} 
    client: 
        serviceUrl: 
            defaultZone: ${vcap.services.${PREFIX:}eureka.credentials.uri:http://user:password@localhost:8761}/eureka/ 
        fetchRegistry: true 

logging: 
    level: 
        root: INFO 
        com.netflix.discovery: 'OFF' 
        org.springframework.integration: DEBUG 
```

之前，我们使用`turbine.aggregator.clusterConfig`属性将用户和餐厅服务添加到一个集群中。这里，值以大写字母表示，因为 Eureka 以大写字母返回服务名称。而且，`turbine.appConfig`属性包含了 Turbine 用来查找实例的 Eureka 服务 ID 列表。请注意，之前的步骤总是使用默认配置创建了相应的服务器。如有需要，可以使用特定设置覆盖默认配置。

# 构建和运行 OTRS 应用程序

使用以下文件：`..\Chapter5 \pom.xml`，使用`mvn clean install`构建所有项目。

输出应该如下所示：

```java
6392_chapter5 ..................................... SUCCESS [3.037s] 
online-table-reservation:common ................... SUCCESS [5.899s] 
online-table-reservation:zuul-server .............. SUCCESS [4.517s] 
online-table-reservation:restaurant-service ....... SUCCESS [49.250s] 
online-table-reservation:eureka-server ............ SUCCESS [2.850s] online-table-reservation:dashboard-server ......... SUCCESS [2.893s] 
online-table-reservation:turbine-server ........... SUCCESS [3.670s] 
online-table-reservation:user-service ............. SUCCESS [47.983s] 
online-table-reservation:api-service .............. SUCCESS [3.065s] 
online-table-reservation:booking-service .......... SUCCESS [26.496s] 
```

然后，命令提示符上进入`<path to source>/6392_chapter5`并运行以下命令：

```java
java -jar eureka-server/target/eureka-server.jar 
java -jar turbine-server/target/turbine-server.jar 
java -jar dashboard-server/target/dashboard-server.jar 
java -jar restaurant-service/target/restaurant-service.jar 
java -jar user-service/target/user-service.jar 
java -jar booking-service/target/booking-service.jar 
java -jar api-service/target/api-service.jar 
```

注意：在启动 Zuul 服务之前，请确保 Eureka 仪表板上的所有服务都处于启动状态：`http://localhost:8761/`：

```java
java -jar zuul-server/target/zuul-server.jar 
```

再次检查 Eureka 仪表板，所有应用程序都应该处于启动状态。然后进行测试。

# 使用容器部署微服务

读完第一章《解决方案方法》后，您可能已经理解了 Docker 的要点。

Docker 容器提供了一个轻量级的运行时环境，由虚拟机的核心功能和操作系统的隔离服务组成，称为 Docker 镜像。Docker 使微服务的打包和执行变得更加简单。每个操作系统可以有多个 Docker，每个 Docker 可以运行单个应用程序。

# 安装与配置

如果您不使用 Linux 操作系统，Docker 需要一个虚拟化服务器。您可以安装 VirtualBox 或类似的工具，如 Docker Toolbox，使其适用于您。Docker 安装页面提供了更多关于它的细节，并告诉您如何执行。所以，请参考 Docker 网站上的 Docker 安装指南。

你可以根据你的平台，通过遵循给出的说明安装 Docker：[`docs.docker.com/engine/installation/`](https://docs.docker.com/engine/installation/)。

DockerToolbox-1.9.1f 是在写作时可用的最新版本。这个版本我们使用了。

# 具有 4GB 内存的 Docker 虚拟机

默认的虚拟机创建时会分配 2GB 的内存。我们将重新创建一个具有 4GB 内存的 Docker 虚拟机：

```java
 docker-machine rm default
 docker-machine create -d virtualbox --virtualbox-memory 4096 default

```

# 使用 Maven 构建 Docker 镜像

有多种 Docker Maven 插件可以使用：

+   [`github.com/rhuss/docker-maven-plugin`](https://github.com/rhuss/docker-maven-plugin)

+   [`github.com/alexec/docker-maven-plugin`](https://github.com/alexec/docker-maven-plugin)

+   [`github.com/spotify/docker-maven-plugin`](https://github.com/spotify/docker-maven-plugin)

你可以根据你的选择使用这些方法中的任何一个。我发现由`@rhuss`编写的 Docker Maven 插件最适合我们使用。这个插件定期更新，并且相比其他插件拥有许多额外的功能。

在讨论`docker-maven-plugin`的配置之前，我们需要在`application.yml`中引入 Docker Spring 配置文件。这样我们在为不同平台构建服务时，工作会更加容易。我们需要配置以下四个属性：

+   我们将使用标识为 Docker 的 Spring 配置文件。

+   由于服务将在它们自己的容器中执行，所以嵌入式 Tomcat 之间不会有端口冲突。现在我们可以使用端口`8080`。

+   我们更倾向于使用 IP 地址来在我们的 Eureka 中注册服务。因此，Eureka 实例属性`preferIpAddress`将被设置为`true`。

+   最后，我们将在`serviceUrl:defaultZone`中使用 Eureka 服务器的主机名。

要在你的项目中添加 Spring 配置文件，请在`application.yml`中现有内容之后添加以下行：

```java
--- 
# For deployment in Docker containers 
spring: 
  profiles: docker 

server: 
  port: 8080 

eureka: 
  instance: 
    preferIpAddress: true 
  client: 
    serviceUrl: 
      defaultZone: http://eureka:8761/eureka/ 
```

使用命令`mvn -P docker clean package`将生成带有 Tomcat 的`8080`端口的`service` JAR，并且该 JAR 会在 Eureka 服务器上以主机名`eureka`注册。

现在，让我们配置`docker-maven-plugin`以构建带有我们的餐厅微服务的镜像。这个插件首先必须创建一个 Dockerfile。Dockerfile 在两个地方配置——在`pom.xml`和`docker-assembly.xml`文件中。我们将在`pom.xml`文件中使用以下的插件配置：

```java
<properties> 
<!-- For Docker hub leave empty; use "localhost:5000/" for a local Docker Registry --> 
  <docker.registry.name>localhost:5000/</docker.registry.name> 
  <docker.repository.name>${docker.registry.name}sourabhh /${project.artifactId}</docker.repository.name> 
</properties> 
... 
<plugin> 
  <groupId>org.jolokia</groupId> 
  <artifactId>docker-maven-plugin</artifactId> 
  <version>0.13.7</version> 
  <configuration> 
    <images> 
      <image> 
<name>${docker.repository.name}:${project.version}</name> 
        <alias>${project.artifactId}</alias> 

        <build> 
          <from>java:8-jre</from> 
          <maintainer>sourabhh</maintainer> 
          <assembly> 
            <descriptor>docker-assembly.xml</descriptor> 
          </assembly> 
          <ports> 
            <port>8080</port> 
          </ports> 
          <cmd> 
            <shell>java -jar \ 
              /maven/${project.build.finalName}.jar server \ 
              /maven/docker-config.yml</shell> 
          </cmd> 
        </build> 
        <run> 
        <!-- To Do --> 
        </run> 
      </image> 
    </images> 
  </configuration> 
</plugin> 
```

在 Docker Maven 插件配置之前创建一个 Dockerfile，该 Dockerfile 扩展了 JRE 8（`java:8-jre`）的基础镜像。这个镜像暴露了端口`8080`和`8081`。

接下来，我们将配置`docker-assembly.xml`文件，该文件告诉插件哪些文件应该被放入容器中。这个文件将被放置在`src/main/docker`目录下：

```java
<assembly   
  xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2 http://maven.apache.org/xsd/assembly-1.1.2.xsd"> 
  <id>${project.artifactId}</id> 
  <files> 
    <file> 
      <source>{basedir}/target/${project.build.finalName}.jar</source> 
      <outputDirectory>/</outputDirectory> 
    </file> 
    <file> 
      <source>src/main/resources/docker-config.yml</source> 
      <outputDirectory>/</outputDirectory> 
    </file> 
  </files> 
</assembly> 
```

前面的组装，在生成的 Dockerfile 中添加了`service` JAR 和`docker-config.yml`文件。这个 Dockerfile 位于`target/docker/`目录下。打开这个文件，你会发现内容与这个类似：

```java
FROM java:8-jre 
MAINTAINER sourabhh 
EXPOSE 8080 
COPY maven /maven/ 
CMD java -jar \ 
  /maven/restaurant-service.jar server \ 
  /maven/docker-config.yml 
```

之前的文件可以在 `restaurant-service\target\docker\sousharm\restaurant-service\PACKT-SNAPSHOT\build` 目录中找到。`build` 目录还包含 `maven` 目录，其中包含 `docker-assembly.xml` 文件中提到的所有内容。

让我们来构建 Docker 镜像：

```java
mvn docker:build

```

一旦此命令完成，我们可以使用 Docker 镜像在本地仓库中验证镜像，或者通过运行以下命令来实现：

```java
docker run -it -p 8080:8080 sourabhh/restaurant-service:PACKT-SNAPSHOT

```

使用 `-it` 来在前台执行此命令，而不是 `-d`。

# 使用 Maven 运行 Docker

要用 Maven 执行 Docker 镜像，我们需要在 `pom.xml` 文件中添加以下配置。`<run>` 块，放在 `pom.xml` 文件中 `docker-maven-plugin` 部分下的 `docker-maven-plugin` 块中标记的 `To Do` 下面：

```java
<properties> 
  <docker.host.address>localhost</docker.host.address> 
  <docker.port>8080</docker.port> 
</properties> 
... 
<run> 
  <namingStrategy>alias</namingStrategy> 
  <ports> 
    <port>${docker.port}:8080</port> 
  </ports> 
  <wait> 
    <url>http://${docker.host.address}:${docker.port}/v1/restaurants/1</url> 
    <time>100000</time> 
  </wait> 
  <log> 
    <prefix>${project.artifactId}</prefix> 
    <color>cyan</color> 
  </log> 
</run> 
```

这里，我们已经定义了运行我们的 Restaurant 服务容器的参数。我们将 Docker 容器端口 `8080` 和 `8081` 映射到宿主系统的端口，这使我们能够访问服务。同样，我们也将容器的 `log` 目录绑定到宿主系统的 `<home>/logs` 目录。

Docker Maven 插件可以通过轮询管理后端的 ping URL 来检测容器是否已完成启动。

请注意，如果您在 Windows 或 MacOS X 上使用 DockerToolbox 或 boot2docker，Docker 主机不是 localhost。您可以执行 `docker-machine ip default` 来检查 Docker 镜像 IP。在启动时也会显示。

Docker 容器准备启动。使用以下命令使用 Maven 启动它：

```java
mvn docker:start
```

# 使用 Docker 进行集成测试

启动和停止 Docker 容器可以通过在 `pom.xml` 文件中的 `docker-maven-plugin` 生命周期阶段绑定以下执行来实现：

```java
<execution> 
  <id>start</id> 
  <phase>pre-integration-test</phase> 
  <goals> 
    <goal>build</goal> 
    <goal>start</goal> 
  </goals> 
</execution> 
<execution> 
  <id>stop</id> 
  <phase>post-integration-test</phase> 
  <goals> 
    <goal>stop</goal> 
  </goals> 
</execution> 
```

现在我们将配置 Failsafe 插件，使用 Docker 执行集成测试。这允许我们执行集成测试。我们在 `service.url` 标签中传递了服务 URL，这样我们的集成测试就可以使用它来执行集成测试。

我们将使用 `DockerIntegrationTest` 标记来标记我们的 Docker 集成测试。它定义如下：

```java
package com.packtpub.mmj.restaurant.resources.docker; 

public interface DockerIT { 
    // Marker for Docker integration Tests 
} 
```

看看下面的集成 `plugin` 代码。你可以看到 `DockerIT` 被配置为包含集成测试（Failsafe 插件），而它被用于在单元测试中排除（Surefire 插件）：

```java
<plugin> 
                <groupId>org.apache.maven.plugins</groupId> 
                <artifactId>maven-failsafe-plugin</artifactId> 
                <configuration> 
                    <phase>integration-test</phase> 
                    <groups>com.packtpub.mmj.restaurant.resources.docker.DockerIT</groups> 
                    <systemPropertyVariables> 
                        <service.url>http://${docker.host.address}:${docker.port}/</service.url> 
                    </systemPropertyVariables> 
                </configuration> 
                <executions> 
                    <execution> 
                        <goals> 
                            <goal>integration-test</goal> 
                            <goal>verify</goal> 
                        </goals> 
                    </execution> 
                </executions> 
       </plugin> 
       <plugin> 
                <groupId>org.apache.maven.plugins</groupId> 
                <artifactId>maven-surefire-plugin</artifactId> 
                <configuration>             <excludedGroups>com.packtpub.mmj.restaurant.resources.docker.DockerIT</excludedGroups> 
                </configuration> 
</plugin> 

```

一个简单的集成测试看起来像这样：

```java
@Category(DockerIT.class) 
public class RestaurantAppDockerIT { 

    @Test 
    public void testConnection() throws IOException { 
        String baseUrl = System.getProperty("service.url"); 
        URL serviceUrl = new URL(baseUrl + "v1/restaurants/1"); 
        HttpURLConnection connection = (HttpURLConnection) serviceUrl.openConnection(); 
        int responseCode = connection.getResponseCode(); 
        assertEquals(200, responseCode); 
    } 
} 
```

您可以使用以下命令执行使用 Maven 的集成测试（请确保在运行集成测试之前从项目目录的根目录运行 `mvn clean install`）：

```java
mvn integration-test

```

# 将镜像推送到注册表

在 `docker-maven-plugin` 下添加以下标签以将 Docker 镜像发布到 Docker hub：

```java
<execution> 
  <id>push-to-docker-registry</id> 
  <phase>deploy</phase> 
  <goals> 
    <goal>push</goal> 
  </goals> 
</execution> 
```

您可以通过使用以下配置跳过 JAR 发布，为 `maven-deploy-plugin`：

```java
<plugin> 
  <groupId>org.apache.maven.plugins</groupId> 
  <artifactId>maven-deploy-plugin</artifactId> 
  <version>2.7</version> 
  <configuration> 
    <skip>true</skip> 
  </configuration> 
</plugin> 
```

在 Docker hub 发布 Docker 镜像也需要用户名和密码：

```java
mvn -Ddocker.username=<username> -Ddocker.password=<password> deploy
```

您还可以将 Docker 镜像推送到您自己的 Docker 注册表。为此，请添加

如下代码所示，添加`docker.registry.name`标签。例如，

如果你的 Docker 注册表可在`xyz.domain.com`端口`4994`上访问，那么定义

通过添加以下代码行：

```java
<docker.registry.name>xyz.domain.com:4994</docker.registry.name> 
```

这不仅完成了部署，还可以测试我们的 Docker 化服务。

# 管理 Docker 容器

每个微服务都将有自己的 Docker 容器。因此，我们将使用`Docker Compose`来管理我们的容器。

Docker Compose 将帮助我们指定容器的数量以及这些容器的执行方式。我们可以指定 Docker 镜像、端口以及每个容器与其他 Docker 容器的链接。

我们将在根项目目录中创建一个名为`docker-compose.yml`的文件，并将所有微服务容器添加到其中。我们首先指定 Eureka 服务器，如下所示：

```java
eureka: 
  image: localhost:5000/sourabhh/eureka-server 
  ports: 
    - "8761:8761" 
```

在这里，`image`代表 Eureka 服务器的发布 Docker 镜像，`ports`代表执行 Docker 镜像的主机和 Docker 主机的映射。

这将启动 Eureka 服务器，并为外部访问发布指定的端口。

现在我们的服务可以使用这些容器（如 Eureka 的依赖容器）。让我们看看`restaurant-service`如何可以链接到依赖容器。很简单；只需使用`links`指令：

```java
restaurant-service: 
  image: localhost:5000/sourabhh/restaurant-service 
  ports: 
    - "8080:8080" 
  links: 
    - eureka 
```

上述链接声明将更新`restaurant-service`容器中的`/etc/hosts`文件，每个服务占一行，`restaurant-service`依赖的服务（假设`security`容器也链接了），例如：

```java
192.168.0.22  security 
192.168.0.31  eureka 
```

如果你没有设置本地 Docker 注册表，那么为了无问题或更平滑的执行，请先设置。

通过运行以下命令构建本地 Docker 注册表：

**docker run -d -p 5000:5000 --restart=always --name registry registry:2**

然后，为本地镜像执行推送和拉取命令：

**docker push localhost:5000/sourabhh/restaurant-service:PACKT-SNAPSHOT**

**docker-compose pull**

最后，执行 docker-compose:

**docker-compose up -d**

一旦所有微服务容器（服务和服务器）都配置好了，我们可以用一个命令启动所有 Docker 容器：

```java
docker-compose up -d
```

这将启动 Docker Compose 中配置的所有 Docker 容器。以下命令将列出它们：

```java
docker-compose ps
Name                                          Command
                State           Ports
-------------------------------------------------------------
onlinetablereservation5_eureka_1         /bin/sh -c java -jar         ...               Up      0.0.0.0:8761->8761/tcp

onlinetablereservation5_restaurant-service_1  /bin/sh -c java -jar       ...   Up      0.0.0.0:8080->8080/tcp

```

您还可以使用以下命令检查 Docker 镜像日志：

```java
docker-compose logs
[36mrestaurant-service_1 | ←[0m2015-12-23 08:20:46.819  INFO 7 --- [pool-3-thread-1] com.netflix.discovery.DiscoveryClient    : DiscoveryClient_RESTAURANT-SERVICE/172.17
0.4:restaurant-service:93d93a7bd1768dcb3d86c858e520d3ce - Re-registering apps/RESTAURANT-SERVICE
[36mrestaurant-service_1 | ←[0m2015-12-23 08:20:46.820  INFO 7 --- [pool-3-thread-1] com.netflix.discovery.DiscoveryClient    : DiscoveryClient_RESTAURANT-SERVICE/172.17
0.4:restaurant-service:93d93a7bd1768dcb3d86c858e520d3ce: registering service... [36mrestaurant-service_1 | ←[0m2015-12-23 08:20:46.917  INFO 7 --- [pool-3-thread-1] com.netflix.discovery.DiscoveryClient    : DiscoveryClient_RESTAURANT-SERVICE/172.17

```

# 参考文献

以下链接将为您提供更多信息：

+   **Netflix** **Ribbon**: [`github.com/Netflix/ribbon`](https://github.com/Netflix/ribbon)

+   **Netflix** **Zuul**: [`github.com/Netflix/zuul`](https://github.com/Netflix/zuul)

+   **RabbitMQ**: [`www.rabbitmq.com/download.html`](https://www.rabbitmq.com/download.html)

+   **Hystrix**: [`github.com/Netflix/Hystrix`](https://github.com/Netflix/Hystrix)

+   **Turbine**: [`github.com/Netflix/Turbine`](https://github.com/Netflix/Turbine)

+   **Docker**: [`www.docker.com/`](https://www.docker.com/)

# 摘要

在本章中，我们学习了关于微服务管理的一系列特性：负载均衡、边缘（网关）服务器、断路器以及监控。在本章学习结束后，你应该知道如何实现负载均衡和路由。我们也学习了如何设置和配置边缘服务器。本章还介绍了另一个重要的安全机制。通过使用 Docker 或其他容器，可以使部署变得简单。本章通过 Maven 构建演示并集成了 Docker。

从测试的角度来看，我们对服务的 Docker 镜像进行了集成测试。我们还探讨了编写客户端的方法，例如`RestTemplate`和 Netflix Feign。

在下一章中，我们将学习如何通过身份验证和授权来保护微服务。我们还将探讨微服务安全的其他方面。
