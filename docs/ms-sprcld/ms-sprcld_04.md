# 第四章：服务发现

在我们到达这一点之前，在前面的章节中我们已经多次讨论了服务发现。实际上，它是微服务架构中最受欢迎的技术方面之一。这样的主题不可能从 Netflix OSS 实现中省略。他们没有决定使用具有类似功能的任何现有工具，而是专门为他们的需求设计并开发了一个发现服务器。然后，它与其他几个工具一起开源了。Netflix OSS 发现服务器被称为**Eureka**。

用于与 Eureka 集成的 Spring Cloud 库包括两部分，客户端和服务端。服务端作为独立的 Spring Boot 应用程序启动，并暴露一个 API，可以收集注册服务列表以及添加带有位置地址的新服务。服务器可以配置和部署为高可用性，每个服务器都与其它服务器复制其状态。客户端作为微服务应用程序的一个依赖项包含在内。它负责启动后的注册、关机前的注销，并通过轮询 Eureka 服务器保持注册列表的最新。

以下是我们在本章中要覆盖的主题列表：

+   开发运行内嵌 Eureka 服务器的应用程序

+   从客户端应用程序连接到 Eureka 服务器

+   高级发现客户端配置

+   启用客户端和服务器之间的安全通信

+   配置故障转移和对等复制机制

+   在不同区域注册客户端应用程序实例

# 在服务器端运行 Eureka

在 Spring Boot 应用程序中运行 Eureka 服务器并不是一件困难的事情。让我们来看看这是如何做到的：

1.  首先，必须包含正确的依赖项到我们的项目中。显然，我们将使用一个启动器：

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```

1.  在主应用程序类上启用 Eureka 服务器：

```java
@SpringBootApplication
@EnableEurekaServer
public class DiscoveryApplication {

    public static void main(String[] args) {
        new SpringApplicationBuilder(DiscoveryApplication.class).web(true).run(args);
    }

}
```

1.  有趣的是，与服务器启动器一起，客户端的依赖项也包括在内。它们对我们可能有用，但只有在以高可用性模式运行 Eureka，并且发现实例之间有对等通信时。当运行独立实例时，它实际上不会带给我们任何东西，除了在启动时在日志中打印一些错误。我们可以从启动器依赖项中排除`spring-cloud-netflix-eureka-client`，或者使用配置属性禁用发现客户端。我更喜欢第二个选择，并且在这个场合，我将默认服务器端口更改为除了`8080`之外的其它值。以下是`application.yml`文件的一个片段：

```java
server: 
 port: ${PORT:8761}
eureka:
 client:
   registerWithEureka: false
   fetchRegistry: false 
```

1.  在完成前面的步骤之后，我们终于可以启动我们的第一个 Spring Cloud 应用程序了。只需从你的 IDE 中运行主类，或者使用 Maven 构建项目并运行它，使用`java -jar`命令等待日志行`Started Eureka Server`出现。它就绪了。一个简单的 UI 仪表板作为主页可通过`http://localhost:8761`访问，并且可以通过`/eureka/*`路径调用 HTTP API 方法。Eureka 仪表板并没有提供很多功能；实际上，它主要用于检查注册的服务列表。这可以通过调用 REST API `http://localhost:8761/eureka/apps`端点来实现。

所以，总结一下，我们知道如何使用 Spring Boot 运行一个独立的 Eureka 服务器，以及如何使用 UI 控制台和 HTTP 方法检查注册的微服务列表。但我们仍然没有任何能够自己在发现中注册的服务，是时候改变这一点了。一个带有发现服务器和客户端实现示例应用程序可以在 GitHub 上的`master`分支找到([`github.com/piomin/sample-spring-cloud-netflix.git`](https://github.com/piomin/sample-spring-cloud-netflix.git))。

# 启用客户端端的 Eureka

与服务器端一样，只需要包含一个依赖项就可以为应用程序启用 Eureka 客户端。所以，首先在你的项目依赖中包含以下启动器：

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```

这个示例应用程序所做的只是与 Eureka 服务器通信。它必须注册自己并向 Eureka 发送元数据信息，如主机、端口、健康指标 URL 和主页。Eureka 从属于某个服务的每个实例接收心跳消息。如果在配置的时间段内没有收到心跳消息，实例将被从注册表中移除。发现客户端的第二个责任是从服务器获取数据，然后缓存它并周期性地询问更改。可以通过在主类上使用`@EnableDiscoveryClient`注解来启用它。令人惊讶的是，还有另一种激活此功能的方法。你可以使用`@EnableEurekaClient`注解，特别是如果类路径中有多个发现客户端实现（Consul、Eureka、ZooKeeper）的话。虽然`@EnableDiscoveryClient`位于`spring-cloud-commons`中，`@EnableEurekaClient`位于`spring-cloud-netflix`中，并且只对 Eureka 有效。以下是发现客户端应用程序的主类：

```java
@SpringBootApplication
@EnableDiscoveryClient
public class ClientApplication {

    public static void main(String[] args) {
         new SpringApplicationBuilder(ClientApplication.class).web(true).run(args);
    }

}
```

客户端配置中不必提供发现服务器的地址，因为默认的主机和端口上可用。然而，我们很容易想象 Eureka 没有在其默认的`8761`端口上监听。下面的配置文件片段可见。可以通过`EUREKA_URL`参数覆盖发现服务器的网络地址，也可以通过`PORT`属性覆盖客户端的监听端口。应用程序在发现服务器中注册的名称取自`spring.application.name`属性：

```java
spring: 
 application:
   name: client-service

server: 
 port: ${PORT:8081}

eureka:
 client:
   serviceUrl:
     defaultZone: ${EUREKA_URL:http://localhost:8761/eureka/}
```

让我们在本地主机上运行我们示例客户端应用程序的两个独立实例。为了实现这一点，需要在启动时覆盖监听端口的数量，像这样：

```java
java -jar -DPORT=8081 target/sample-client-service-1.0-SNAPSHOT.jar
java -jar -DPORT=8082 target/sample-client-service-1.0-SNAPSHOT.jar 
```

正如您在下面的截图所看到的，有一个名为`client-service`的实例注册了`piomin`这个主机名和`8081`和`8082`这两个端口：

![](img/5b172c1a-67fb-4dfe-a0fb-3249d94261e8.png)

# 关机时的注销

检查与 Eureka 客户端的注销工作有点更具挑战性。我们的应用程序应该优雅地关闭，以便能够拦截一个停止事件并向服务器发送一个事件。实现优雅关闭的最佳方式是使用 Spring Actuator 的`/shutdown`端点。Actuator 是 Spring Boot 的一部分，可以通过在`pom.xml`中声明`spring-boot-starter-actuator`依赖项来将其包含在项目中。它默认是禁用的，因此我们必须在配置属性中启用它。为了简单起见，禁用该端点的用户/密码安全性是值得的：

```java
endpoints:
 shutdown:
   enabled: true
   sensitive: false
```

要关闭应用程序，我们必须调用`POST /shutdown`API 方法。如果您收到响应`{"message": "Shutting down, bye..."}`，这意味着一切都很顺利，流程已经开始。在应用程序被禁用之前，从 Shutting down DiscoveryClient...行开始的某些日志将被打印出来。之后，服务将从发现服务器上注销，并完全消失在注册服务列表中。我决定通过调用`http://localhost:8082/shutdown`（您可以使用任何 REST 客户端，例如 Postman）关闭客户端实例#2，因此只在端口`8081`上运行的实例在仪表板上仍然可见：

![](img/0aea89ae-bdfb-43be-9acc-0e13ebf666f9.png)

Eureka 服务器仪表板还提供了一种方便的方式来查看新创建和取消租约的历史记录：

![](img/097c33ca-f38d-4501-a15f-1051ccf56d4b.png)

优雅关闭显然是停止应用程序的最合适方式，但在现实世界中，我们并不总是能够实现它。许多意想不到的事情可能发生，例如服务器机器重新启动、应用程序失败或客户端与服务器之间的网络问题。从发现服务器的角度来看，这种情况与从 IDE 中停止客户端应用程序或从命令行杀死进程相同。如果您尝试这样做，您将发现发现客户端关闭程序不会被触发，服务在 Eureka 仪表板上仍然显示为*UP*状态。此外，租约永远不会过期。

为了避免这种情况，服务器端的默认配置应该进行更改。*为什么在默认设置中会出现这样的问题？* Eureka 提供了一个特殊的机制，当检测到一定数量的服务没有及时续租时，注册表停止过期条目。这应该保护注册表在网络部分故障时清除所有条目。这个机制被称为**自我保护模式**，可以在`application.yml`中使用`enableSelfPreservation`属性禁用它。当然，在生产环境中不应该禁用它：

```java
eureka:
 server:
   enableSelfPreservation: false
```

# 使用发现客户端程序化

客户端应用程序启动后，注册服务列表会自动从 Eureka 服务器获取。然而，有时可能需要程序化地使用 Eureka 的客户端 API。我们有两种可能性：

+   `com.netflix.discovery.EurekaClient`：它实现了 Eureka 服务器暴露的所有 HTTP API 方法，这些方法在 Eureka API 部分已经描述过了。

+   `org.springframework.cloud.client.discovery.DiscoveryClient`：这是 Spring Cloud 的一个替代 Netflix `EurekaClient`的本地客户端。它提供了一个简单、通用的 API，对于所有的发现客户端都很有用。有两个方法可用，`getServices`和`getInstances`：

```java
private static final Logger LOGGER = LoggerFactory.getLogger(ClientController.class);

@Autowired
private DiscoveryClient discoveryClient;

@GetMapping("/ping")
public List<ServiceInstance> ping() {
 List<ServiceInstance> instances = discoveryClient.getInstances("CLIENT-SERVICE");
 LOGGER.info("INSTANCES: count={}", instances.size());
 instances.stream().forEach(it -> LOGGER.info("INSTANCE: id={}, port={}", it.getServiceId(), it.getPort()));
 return instances;
}
```

有一个与前面实现相关有趣的点。如果你在服务启动后立即调用`/ping`端点，它不会显示任何实例。这与响应缓存机制有关，下一节会详细描述。

# 高级配置设置

Eureka 的配置设置可以分为三部分：

+   **服务器**：它定制了服务器的行为。它包括所有带有`eureka.server.*`前缀的属性。可用的字段完整列表可以在`EurekaServerConfigBean`类中找到([`github.com/spring-cloud/spring-cloud-netflix/blob/master/spring-cloud-netflix-eureka-server/src/main/java/org/springframework/cloud/netflix/eureka/server/EurekaServerConfigBean.java`](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/spring-cloud-netflix-eureka-server/src/main/java/org/springframework/cloud/netflix/eureka/server/EurekaServerConfigBean.java))。

+   **客户端**：这是 Eureka 客户端侧可用的两个属性部分中的第一个。它负责配置客户端如何查询注册表以定位其他服务。它包括所有带有`eureka.client.*`前缀的属性。要查看所有可用字段的全列表，请参考`EurekaClientConfigBean`类 ([`github.com/spring-cloud/spring-cloud-netflix/blob/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaClientConfigBean.java`](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaClientConfigBean.java))。

+   **实例**：它定制了 Eureka 客户端当前实例的行为，例如端口或名称。它包括所有带有`eureka.instance.*`前缀的属性。要查看所有可用字段的全列表，请参考`EurekaInstanceConfigBean`类 ([`github.com/spring-cloud/spring-cloud-netflix/blob/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java`](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java))。

我已经向你展示了如何使用这些属性以达到预期的效果。在下一部分中，我将讨论一些与配置设置自定义相关有趣的场景。不需要描述所有属性。你可以在前面列出的所有类的源代码中的注释中阅读它们。

# 刷新注册表

让我们先回到之前的示例。自保模式已被禁用，但仍然需要等待服务器取消租约，这需要很长时间。造成这种情况有几个原因。第一个原因是每个客户端服务会每 30 秒向服务器发送一次心跳（默认值），这可以通过`eureka.instance.leaseRenewalIntervalInSeconds`属性进行配置。如果服务器没有收到心跳，它会在 90 秒后从注册表中移除实例，从而切断发送到该实例的交通。这可以通过`eureka.instance.leaseExpirationDurationInSeconds`属性进行配置。这两个参数都是在客户端设置的。出于测试目的，我们在秒中定义了小的值：

```java
eureka:
 instance:
   leaseRenewalIntervalInSeconds: 1
   leaseExpirationDurationInSeconds: 2
```

在服务器端还应该更改一个属性。Eureka 在后台运行 evict 任务，负责检查客户端的心跳是否仍在接收。默认情况下，它每 60 秒触发一次。所以，即使租约续订间隔和租约到期时长被设置为相对较低的值，服务实例在最坏的情况下也可能在 60 秒后被移除。后续计时器滴答之间的延迟可以通过使用`evictionIntervalTimerInMs`属性来配置，与前面讨论的属性不同，这个属性是以毫秒为单位的：

```java
eureka:
  server:
    enableSelfPreservation: false
    evictionIntervalTimerInMs: 3000
```

所有必需的参数都已分别在客户端和服务端定义。现在，我们可以使用`-DPORT` VM 参数再次运行发现服务器，然后在端口`8081`、`8082`和`8083`上启动客户端应用程序的三个实例。在那之后，我们逐一关闭端口`8081`和`8082`上的实例，只需杀死它们的进程即可。结果是什么？禁用的实例几乎立即从 Eureka 注册表中移除。以下是 Eureka 服务器的日志片段：

![](img/9053350f-ae07-434c-ab43-4e1040362e37.png)

仍有一个实例正在监听端口`8083`。在自我维护模式被禁用时，与之一相关的警告信息将在 UI 仪表板上打印出来。一些额外的信息，比如租约到期状态或上分钟内续租次数，也许也挺有趣。通过操作所有这些属性，我们能够定制过期的租约移除流程的维护。然而，确保定义的设置不会影响系统的性能是很重要的。还有一些其他元素对配置的变化很敏感，比如负载均衡器、网关和熔断器。如果你禁用了自我维护模式，Eureka 会打印一条警告信息，你可以在下面的截图中看到：

![](img/18a93209-3951-4751-a9a2-d1810cbace4e.png)

# 更改实例标识符

在 Eureka 上注册的实例按名称分组，但每个实例必须发送一个唯一 ID，基于此 ID，服务器能够识别它。也许你已经注意到`instanceId`在仪表板上每个服务组的`Status`列中显示。Spring Cloud Eureka 会自动生成这个数字，它等于以下字段的组合：

```java
${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}}}. 
```

这个标识符可以通过`eureka.instance.instanceId`属性轻松覆盖。为了测试目的，让我们启动一些客户端应用程序实例，使用以下配置设置和`-DSEQUENCE_NO=[n]` VM 参数，其中`[n]`从`1`开始的序列号。以下是一个根据`SEQUENCE_NO`参数动态设置监听端口和发现`instanceId`的客户端应用程序的示例配置：

```java
server: 
 port: 808${SEQUENCE_NO}
eureka:
 instance:
   instanceId: ${spring.application.name}-${SEQUENCE_NO}
```

结果可以在 Eureka 仪表板上查看：

![](img/fdb301da-0900-4cbf-abef-b084cbf557b9.png)

# 优先选择 IP 地址

默认情况下，所有实例都注册在其主机名下。这是一个非常方便的方法，前提是我们在我们的网络上启用了 DNS。然而，对于用作组织中微服务环境的服务器组，DNS 通常是不可用的，我自己就遇到过这种情况。除了在所有 Linux 机器上的`/etc/hosts`文件中添加主机名及其 IP 地址外，别无他法。这种解决方案的替代方法是更改注册过程配置设置，以广告服务的 IP 地址而不是主机名。为了实现这一点，客户端应将`eureka.instance.preferIpAddress`属性设置为`true`。注册表中的每个服务实例仍然会以`instanceId`包含主机名的形式打印到 Eureka 仪表板中，但如果你点击这个链接，重定向将基于 IP 地址进行。负责通过 HTTP 调用其他服务的 Ribbon 客户端也将遵循相同的原则。

如果你决定使用 IP 地址作为确定服务网络位置的主要方法，你可能会有问题。如果你有多个网络接口分配给你的机器，可能会出现问题。例如，在我曾经工作过的某个组织中，管理模式（我的工作站与服务器之间的连接）和生产模式（两台服务器之间的连接）有不同的网络。因此，每台服务器机器都分配有两个网络接口，具有不同的 IP 前缀。为了选择正确的接口，你可以在`application.yml`配置文件中定义一个忽略的模式列表。例如，我们希望能够忽略所有接口，其名称以`eth1`开头：

```java
spring:
  cloud:
    inetutils:
      ignoredInterfaces:
        - eth1*
```

还有一种方法可以获得那种效果。我们可以定义应该优先的网络地址：

```java
spring:
  cloud:
    inetutils:
      preferredNetworks:
        - 192.168
```

# 响应缓存

Eureka Server 默认缓存响应。缓存每 30 秒失效一次。可以通过调用 HTTP API 端点`/eureka/apps`轻松检查。如果你在客户端应用程序注册后立即调用它，你会发现响应中仍然没有返回。30 秒后再试，你会发现新实例出现了。响应缓存超时可以通过`responseCacheUpdateIntervalMs`属性覆盖。有趣的是，在使用 Eureka 仪表板显示已注册实例列表时，并没有缓存。与 REST API 相比，它绕过了响应缓存：

```java
eureka:
 server:
   responseCacheUpdateIntervalMs: 3000
```

我们应该记住，Eureka 注册表也缓存在客户端。所以，即使我们在服务器端更改了缓存超时时间，它可能仍然需要一段时间才能被客户端刷新。注册表通过一个默认每 30 秒调度一次的异步后台任务定期刷新。这个设置可以通过声明`registryFetchIntervalSeconds`属性来覆盖。它只获取与上一次抓取尝试相比的增量。可以通过使用`shouldDisableDelta`属性来禁用此选项。我在服务器和客户端两边都定义了`3`秒的超时时间。如果你用这样的设置启动示例应用程序，`/eureka/apps`将显示新注册服务的实例，可能在你的第一次尝试中。除非客户端端的缓存有意义，否则我不确定在服务器端缓存是否有意义，尤其是因为 Eureka 没有后端存储。就个人而言，我从未需要更改这些属性的值，但我猜想它可能很重要，例如，如果你使用 Eureka 开发单元测试，并且需要无缓存的即时响应：

```java
eureka:
 client:
   registryFetchIntervalSeconds: 3
   shouldDisableDelta: true
```

# 启用客户端和服务器之间的安全通信

到目前为止，Eureka 服务器没有对客户端的任何连接进行身份验证。在开发模式下，安全性并不像在生产模式下那么重要。缺乏安全性可能是一个问题。我们希望能够至少确保发现服务器通过基本身份验证进行安全，以防止任何知道其网络地址的服务遭受未经授权的访问。尽管 Spring Cloud 参考资料声称*HTTP 基本身份验证将自动添加到您的 Eureka 客户端*，但我还是不得不将带有安全性的启动器添加到项目依赖中：

```java
 <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-security</artifactId>
 </dependency>
```

然后，我们应该启用安全功能，并通过在`application.yml`文件中更改配置设置来设置默认凭据：

```java
security:
 basic:
   enabled: true
 user:
   name: admin
   password: admin123
```

现在，所有 HTTP API 端点和 Eureka 仪表板都得到了保护。要在客户端启用基本身份验证模式，应在 URL 连接地址中提供凭据，正如您在以下配置设置中所看到的那样。一个实现了安全发现示例应用程序在同一个存储库中 basic example，但您需要切换到`security`分支([`github.com/piomin/sample-spring-cloud-netflix/tree/security`](https://github.com/piomin/sample-spring-cloud-netflix/tree/security))。以下是客户端启用 HTTP 基本身份验证的配置：

```java
eureka:
 client:
   serviceUrl:
     defaultZone: http://admin:admin123@localhost:8761/eureka/
```

对于更高级的使用，例如在发现客户端和服务器之间使用证书认证的安全 SSL 连接，我们应该提供一个`DiscoveryClientOptionalArgs`的自定义实现。我们将在第十二章，*保护 API*，专门讨论 Spring Cloud 应用程序的安全性，讨论这样一个例子。

# 注册安全服务

保护服务器端是一回事，注册安全应用程序是另一回事。让我们看看我们如何做到这一点：

1.  为了给 Spring Boot 应用程序启用 SSL，我们需要从生成自签名证书开始。我建议你使用`keytool`，它可以在你 JRE 根目录下的`bin`目录中找到：

```java
keytool -genkey -alias client -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore keystore.p12 -validity 3650
```

1.  输入所需数据，并将生成的密钥库文件`keystore.p12`复制到您应用程序的`src/main/resources`目录中。下一步是使用`application.yml`中的配置属性为 Spring Boot 启用 HTTPS：

```java
server: 
 port: ${PORT:8081}
 ssl:
   key-store: classpath:keystore.p12
   key-store-password: 123456
   keyStoreType: PKCS12
   keyAlias: client
```

1.  在运行应用程序之后，您应该能够调用安全端点`https://localhost:8761/info`。我们还需要对 Eureka 客户端实例配置进行一些更改：

```java
eureka:
 instance:
   securePortEnabled: true
   nonSecurePortEnabled: false
   statusPageUrl: https://${eureka.hostname}:${server.port}/info
   healthCheckUrl: https://${eureka.hostname}:${server.port}/health
   homePageUrl: https://${eureka.hostname}:${server.port}/
```

# Eureka API

Spring Cloud Netflix 提供了一个用 Java 编写的客户端，将 Eureka HTTP API 隐藏在开发者面前。如果我们使用除 Spring 之外的其他框架，Netflix OSS 提供了一个原味的 Eureka 客户端，可以作为依赖项包含在内。然而，我们可能需要直接调用 Eureka API，例如，如果应用程序是用 Java 以外的语言编写的，或者我们需要在持续交付过程中注册的服务列表等信息。以下是一个快速参考表：

| **HTTP 端点** | **描述** |
| --- | --- |
| `POST /eureka/apps/appID` | 将服务的新实例注册到注册表 |
| `DELETE /eureka/apps/appID/instanceID` | 从注册表中删除服务实例 |
| `PUT /eureka/apps/appID/instanceID` | 向服务器发送心跳 |
| `GET /eureka/apps` | 获取有关所有注册服务实例列表的详细信息 |
| `GET /eureka/apps/appID` | 获取特定服务所有注册实例列表的详细信息 |
| `GET /eureka/apps/appID/instanceID` | 获取特定服务实例的详细信息 |
| `PUT /eureka/apps/appID/instanceID/metadata?key=value` | 更新元数据参数 |
| `GET /eureka/instances/instanceID` | 获取具有特定 ID 的所有注册实例的详细信息 |
| `PUT /eureka/apps/appID/instanceID/status?value=DOWN` | 更新实例的状态 |

# 复制和高度可用性

我们已经讨论了一些有用的 Eureka 设置，但到目前为止，我们只分析了一个单一服务发现服务器的系统。这种配置是有效的，但只适用于开发模式。对于生产模式，我们希望能够至少运行两个发现服务器，以防其中一个失败或发生网络问题。Eureka 按定义是为了可用性和弹性而构建的，这是 Netflix 开发的主要支柱之二。但它不提供标准的集群机制，如领导选举或自动加入集群。它是基于对等复制模型。这意味着所有服务器复制数据并向所有对等节点发送心跳，这些节点在当前服务器节点的配置中设置。这种算法简单有效，用于包含数据，但它也有一些缺点。它限制了可扩展性，因为每个节点都必须承受服务器上的整个写入负载。

# 示例解决方案的架构

有趣的是，复制机制是新版本 Eureka Server 开始工作的主要动机之一。Eureka 2.0 仍然处于积极开发中。除了优化的复制功能外，它还将提供一些有趣的功能，如服务器向客户端推送注册列表中任何更改的推送模型，自动扩展服务器和一个丰富的仪表板。这个解决方案看起来很有希望，但 Spring Cloud Netflix 仍然使用版本 1，老实说我没有找到任何迁移到版本 2 的计划。Dalston.SR4 发布列车当前的 Eureka 版本是 1.6.2。服务器端复制机制的配置归结为一点，即使用`eureka.client.*`属性部分设置另一个发现服务器的 URL。所选服务器只需在其他服务器中注册自己，这些服务器被选择作为创建的集群的一部分。展示这个解决方案在实践中如何工作的最好方式当然是通过示例。

让我们从示例系统的架构开始，如下面的图表所示。我们的所有应用程序都将在本地不同端口上运行。在这个阶段，我们必须介绍基于 Netflix Zuul 的 API 网关的示例。这对于在不同区域的三个服务实例之间进行负载均衡测试非常有帮助：

![](img/39bca4a7-e902-4c34-bdd1-b5118fd74640.png)

# 构建示例应用程序

对于 Eureka Server，所有必需的更改可能定义在配置属性中。在`application.yml`文件中，我为每个发现服务实例定义了三个不同的配置文件。现在，如果您尝试在 Spring Boot 应用程序中运行 Eureka Server，您需要通过提供 VM 参数`-Dspring.profiles.active=peer[n]`来激活特定的配置文件，其中`[n]`是实例序列号：

```java
spring:
 profiles: peer1
eureka:
 instance:
   hostname: peer1
   metadataMap:
     zone: zone1
 client:
   serviceUrl:
     defaultZone: http://localhost:8762/eureka/,http://localhost:8763/eureka/
server: 
 port: ${PORT:8761}

---
spring:
 profiles: peer2
eureka:
 instance:
   hostname: peer2
   metadataMap:
     zone: zone2
 client:
   serviceUrl:
     defaultZone: http://localhost:8761/eureka/,http://localhost:8763/eureka/
server: 
 port: ${PORT:8762}

---
spring:
 profiles: peer3
eureka:
 instance:
   hostname: peer3
   metadataMap:
     zone: zone3
 client:
   serviceUrl:
     defaultZone: http://localhost:8761/eureka/,http://localhost:8762/eureka/
server: 
 port: ${PORT:8763}
```

在使用不同配置文件名称运行所有三个 Eureka 实例之后，我们创建了一个本地发现集群。如果您在启动后立即查看任何 Eureka 实例的仪表板，它总是看起来一样，我们可以看到三个 DISCOVERY-SERVICE 实例：

![](img/82c4f7d9-4624-4b6b-ae20-729088d40aff.png)

下一步是运行客户端应用程序。项目中的配置设置与 Eureka 服务器的应用程序非常相似。`defaultZone`字段中提供的地址顺序决定了尝试连接不同发现服务的顺序。如果无法连接到第一个服务器，它会尝试从列表中连接第二个服务器，依此类推。与之前一样，我们应该设置 VM 参数`-Dspring.profiles.active=zone[n]`以选择正确的配置文件。我还建议您设置`-Xmx192m`参数，考虑到我们本地测试所有的服务。如果您不为 Spring Cloud 应用程序提供任何内存限制，它在启动后大约会消耗 350MB 的堆内存，总内存大约 600MB。除非您有很多 RAM，否则它可能会使您在本地机器上运行微服务的多个实例变得困难：

```java
spring:
 profiles: zone1
eureka:
 client:
   serviceUrl:
     defaultZone: http://localhost:8761/eureka/,http://localhost:8762/eureka/,http://localhost:8763/eureka/
server: 
 port: ${PORT:8081}

---
spring:
 profiles: zone2
eureka:
 client:
   serviceUrl:
     defaultZone: http://localhost:8762/eureka/,http://localhost:8761/eureka/,http://localhost:8763/eureka/
server: 
 port: ${PORT:8082}

---
spring:
 profiles: zone3
eureka:
 client:
   serviceUrl:
     defaultZone: http://localhost:8763/eureka/,http://localhost:8761/eureka/,http://localhost:8762/eureka/
server: 
 port: ${PORT:8083}
```

让我们再次查看 Eureka 仪表板。我们有`client-service`的三个实例在所有地方注册，尽管应用程序最初只连接到一个发现服务实例。无论我们进入哪个发现服务实例的仪表板查看，结果都是一样的。这正是这次练习的目的。现在，我们创建一些额外的实现仅为了证明一切按预期工作：

![](img/8fd6c50b-fd76-4f61-8b37-2459cbbf63f0.png)

客户端应用程序所做的不仅仅是暴露一个打印所选配置文件名称的 REST 端点。配置文件名称指向特定应用程序实例的主要发现服务实例。下面是一个简单的`@RestController`实现，打印当前区域的名称：

```java
@RestController
public class ClientController {

  @Value("${spring.profiles}")
  private String zone;

  @GetMapping("/ping")
  public String ping() {
    return "I'm in zone " + zone;
  }

}
```

最后，我们可以继续实现 API 网关。在本章范围内详细介绍 Zuul，Netflix 的 API 网关和路由功能是不合适的。我们将在下一章讨论它。Zuul 现在将有助于测试我们的示例解决方案，因为它能够检索在发现服务器中注册的服务列表，并在客户端应用程序的所有运行实例之间执行负载均衡。正如您在下面的配置片段中所看到的，我们使用一个在端口`8763`上监听的发现服务器。所有带有`/api/client/**`路径的传入请求将被路由到`client-service`：

```java
zuul:
 prefix: /api
 routes:
   client: 
     path: /client/**
     serviceId: client-service

eureka:
 client:
   serviceUrl:
     defaultZone: http://localhost:8763/eureka/
   registerWithEureka: false
```

接下来让我们进行测试。我们的应用通过 Zuul 代理启动时应使用`java -jar`命令，与之前的服务不同，这里无需设置任何额外参数，包括配置文件名。它默认与编号为#3 的发现服务相连。要通过 Zuul 代理调用客户端 API，你需要在网页浏览器中输入以下地址，`http://localhost:8765/api/client/ping`。结果如下截图所示：

![](img/82dce964-212f-4a2b-8fd2-f8aa4db10fdc.png)

如果你连续重试几次请求，它应该在所有现有的`client-service`实例之间进行负载均衡，比例为 1:1:1，尽管我们的网关只连接到发现#3。这个例子充分展示了如何使用多个 Eureka 实例构建服务发现。

前面提到的示例应用程序在 GitHub 上可获得，位于`cluster`分支中([`github.com/piomin/sample-spring-cloud-netflix.git`](https://github.com/piomin/sample-spring-cloud-netflix.git))([`github.com/piomin/sample-spring-cloud-netflix/tree/cluster_no_zones`](https://github.com/piomin/sample-spring-cloud-netflix/tree/cluster_no_zones))。

# 故障转移

你可能想知道如果服务发现的一个实例崩溃了会发生什么？为了检查集群在故障发生时的行为，我们将稍稍修改之前的示例。现在，Zuul 在其配置设置中有一个到第二个服务发现的故障转移连接，端口为`8762`。为了测试目的，我们关闭了端口`8763`上的第三个发现服务实例：

```java
eureka:
 client:
   serviceUrl:
     defaultZone: http://localhost:8763/eureka/,http://localhost:8762/eureka/
   registerWithEureka: false
```

当前情况在下图中说明。测试通过调用网关端点进行，端点地址为`http://localhost:8765/api/client/ping`。结果与之前测试相同，负载均衡在所有三个`client-service`实例之间平均进行，符合预期。尽管发现服务#3 已被禁用，但另外两个实例仍能相互通信，并从实例#3 复制第三个客户端应用实例的网络位置信息，只要实例#3 处于活动状态。现在，即使我们重新启动网关，它仍能够使用`defaultZone`字段中的第二个地址顺序连接发现集群，地址为`http://localhost:8762/eureka`。对于客户端应用的第三个实例也适用，该实例反过来将发现服务#1 作为备份连接：

![](img/6a432d7f-3bbb-4afb-89e2-a306aa26fa2d.png)

# 区域

基于对等复制的集群在大多数情况下是一个不错的选择，但并非总是足够。Eureka 还有一个在集群环境中可能非常有用的有趣特性。实际上，区域机制是默认行为。即使我们有一个单独的独立服务发现实例，每个客户端的属性也必须在配置设置中设置为`eureka.client.serviceUrl.defaultZone`。这什么时候对我们有用呢？为了解析它，我们回到前面章节中的示例。让我们假设现在我们的环境被划分为三个不同的物理网络，或者我们只是有三台不同的机器处理传入的请求。当然，服务发现服务在逻辑上仍然分组在集群中，但每个实例都位于一个单独的区域。每个客户端应用程序都将注册在与其主要服务发现服务器相同的区域。我们不是要启动一个 Zuul 网关的实例，而是要启动三个实例，每个实例对应一个单一的区域。如果请求进入网关，它应该在尝试调用注册在其他区域的服务之前，优先考虑利用同一区域内的服务客户端。当前系统架构在下图中可视化。当然，为了示例的目的，架构被简化为能够在单个本地机器上运行。在现实世界中，如我之前提到的，它将在三台不同的机器上启动，甚至可能在其他网络上物理分离成三组机器：

![](img/76b111a0-a8bf-4152-bc6b-da28a8e922c4.png)

# 具有独立服务器的区域

在这个阶段，我们应该强调一点，区域机制仅在客户端实现。这意味着服务发现实例没有被分配到任何区域。所以前一个图表可能有些令人困惑，但它指示了哪个 Eureka 是特定区域中所有客户端应用程序和网关的默认服务发现。我们的目的是检查高可用性模式下的机制，但我们也可以只构建一个单一的服务发现服务器。以下图表展示了与前一个图表类似的情况，不同之处在于它假设只有一个服务发现服务器为所有应用程序服务：

![](img/d2c54f1c-e08d-48f7-a360-5ae1a1153c36.png)

# 构建示例应用程序

为了启用区域处理，我们需要对客户端和网关的配置设置进行一些更改。以下是从客户端应用程序中修改的`application.yml`文件：

```java
spring:
 profiles: zone1
eureka:
 instance:
   metadataMap:
     zone: zone1
 client:
   serviceUrl:
     defaultZone: http://localhost:8761/eureka/,http://localhost:8762/eureka/,http://localhost:8763/eureka/
```

唯一需要更新的是`eureka.instance.metadataMap.zone `属性，我们在其中设置了区域名称和我们的服务已注册的服务名称。

在网关配置中必须进行更多更改。首先，我们需要添加三个配置文件，以便能够在三个不同区域和三个不同的发现服务器上运行一个应用程序。现在当启动网关应用程序时，我们应该设置 VM 参数`-Dspring.profiles.active=zone[n]`以选择正确的配置文件。与`client-service`类似，我们还必须在配置设置中添加`eureka.instance.metadataMap.zone`属性。还有一个属性`eureka.client.preferSameZoneEureka`，在示例中首次使用，如果网关应该优先选择注册在同一区域的客户端应用程序实例，则必须将其设置为`true`：

```java
spring:
 profiles: zone1
eureka:
 client:
   serviceUrl:
     defaultZone: http://localhost:8761/eureka/
     registerWithEureka: false
     preferSameZoneEureka: true
 instance:
   metadataMap:
     zone: zone1
server: 
 port: ${PORT:8765}

---
spring:
 profiles: zone2
eureka:
 client:
   serviceUrl:
     defaultZone: http://localhost:8762/eureka/
     registerWithEureka: false
     preferSameZoneEureka: true
 instance:
   metadataMap:
     zone: zone2
server: 
 port: ${PORT:8766}

---
spring:
 profiles: zone3
eureka:
 client:
   serviceUrl:
     defaultZone: http://localhost:8763/eureka/
     registerWithEureka: false
     preferSameZoneEureka: true
 instance:
   metadataMap:
     zone: zone3
server: 
 port: ${PORT:8767}
```

在启动发现、客户端和网关应用程序的所有实例后，我们可以尝试调用在`http://localhost:8765/api/client/ping`、`http://localhost:8766/api/client/ping`和`http://localhost:8767/api/client/ping`地址下可用的端点。它们都将始终与注册在相同区域的客户端实例进行通信。因此，与没有首选区域的测试相比，例如，端口`8765`上可用的第一个网关实例始终打印出“我在 zone1 区域”并在调用 ping 端点时：

![](img/df5918ad-c4b6-477b-a705-6ff4a7ba736d.png)

当客户端#1 不可用时会发生什么？因为它们都位于与网关#1 不同的区域，所以传入的请求将被负载均衡 50/50 分配到两个其他客户端应用程序实例。

# 总结

在本章中，我们有机会首次在本书中使用 Spring Cloud 开发应用程序。在我看来，开始微服务框架冒险的最佳方式是尝试弄清楚如何正确实现服务发现。从最简单的用例和示例开始，我们已经经历了 Netflix OSS Eureka 项目提供的先进且生产就绪的功能。我向您展示了如何在*五分钟内*创建和运行一个基本客户端和一个独立的发现服务器。基于该实现，我介绍了如何自定义 Eureka 客户端和服务器以满足我们的特定需求，重点放在网络或应用程序失败等负面场景上。诸如 REST API 或 UI 仪表板等特性已经详细讨论。最后，我向您展示了如何使用 Eureka 的机制（如复制、区域和高可用性）创建一个生产就绪环境。有了这些知识，您应该能够选择通过 Eureka 构建适合您微服务架构特性的服务发现功能。

一旦我们讨论了服务发现，我们就可以继续探讨微服务架构中的下一个关键元素：配置服务器。服务和配置服务通常都基于键/值存储，因此它们可能由相同的产品提供。然而，由于 Eureka 只专注于发现，Spring Cloud 引入了自己的框架来管理分布式配置，即 Spring Cloud Config。
