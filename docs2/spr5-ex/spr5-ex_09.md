# 整合所有内容

在采用微服务架构风格时，我们会面临一些挑战。第一个是处理操作复杂性；如服务发现和负载均衡器等这样的服务帮助我们解决这些问题。我们在前面的章节中解决了这些挑战，并在解决这些挑战的过程中了解了一些重要的工具。

在采用微服务架构时，还有一些其他重要的关键点需要处理。有效监控我们微服务环境中发生的事情的方法是监控微服务消耗其他微服务资源（如 HTTP API）的次数以及它们失败的次数。如果我们有接近实时的统计数据，可以节省开发者几天的时间进行故障排除和错误调查。

在本章中，我们将创建一些服务，这些服务可以帮助我们监控 Hystrix 命令并在分布式环境中聚合命令的统计信息。

安全性是微服务架构中的一个重要特性，尤其是在微服务架构采用的分布式特性方面。在我们的架构中有很多微服务；我们无法在服务之间共享状态，因此无状态安全非常适合我们的环境。

OAuth 2.0 协议规范具有这个重要的特性：无状态实现。Spring Cloud Security 提供了对 OAuth 2.0 的支持。

最后，我们将使用 Docker 将我们的微服务 Docker 化，以便使用 Docker Compose 文件中的镜像。

在本章中，我们将了解：

+   实现聚合 Hystrix 流的 Turbine 服务器

+   配置 Hystrix Dashboard 以使用 Turbine 和输入数据

+   创建一个将集成电子邮件 API 的邮件服务

+   理解 Spring Cloud Security

+   将我们的微服务 Docker 化

# 航空公司预订微服务

航空公司 `预订` 微服务是一个标准的 Spring Boot 应用程序。它与其他服务有一些交互，例如 `航班` 微服务。

这些交互是通过 Hystrix 创建的，为航空公司 `预订` 微服务带来了一些期望的行为，例如容错和弹性。

这个服务有一些业务规则，它们现在对学习上下文并不重要，所以我们将在项目创建和执行部分跳过。

完整的源代码可以在 GitHub 上找到（[`github.com/PacktPublishing/Spring-5.0-By-Example/tree/master/Chapter09/airline-booking`](https://github.com/PacktPublishing/Spring-5.0-By-Example/tree/master/Chapter09/airline-booking)）；让我们查看它并看看一些代码。

# 航空公司支付微服务

航空公司 `支付` 是一个微服务，为我们的航空公司票务系统提供支付确认。出于学习目的，我们将跳过这个项目，因为其中有一些业务规则，在 Spring 框架的上下文中并不重要。

我们可以在 GitHub 上找到完整的源代码（[`github.com/PacktPublishing/Spring-5.0-By-Example/tree/master/Chapter09/airline-payments`](https://github.com/PacktPublishing/Spring-5.0-By-Example/tree/master/Chapter09/airline-payments)）。

# 了解涡轮服务器

在我们的微服务组中，有一些集成；`Bookings`微服务调用`Fares`微服务和`Passengers`微服务，这些集成是通过 Hystrix 来实现的，使其更具弹性和容错性。

然而，在微服务世界中，存在多个服务实例。这将需要我们按实例聚合 Hystrix 命令指标。逐个管理实例不是一个好主意。在这种情况下，涡轮服务器可以帮助开发者。

默认情况下，涡轮从由 Hystrix 运行的服务器中拉取指标，但在云环境中不推荐这样做，因为它可能会消耗大量的网络带宽，并增加流量成本。我们将使用 Spring Cloud Stream RabbitMQ 通过**高级消息队列协议**（**AMQP**）将指标推送到涡轮。因此，我们需要配置 RabbitMQ 连接，并在我们的微服务中添加两个额外的依赖项，这些依赖项是：

```java
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-netflix-hystrix-stream</artifactId>
</dependency>

<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```

这些依赖项将使指标可以通过 AMQP 协议发送到涡轮服务器。

默认情况下，涡轮流使用端口`8989`。我们将将其配置为在`8010`上运行，并且我们可以使用`application.yaml`中的`turbine.stream.port`属性来自定义它。

涡轮流将成为 Hystrix 仪表板的数据输入，以显示命令指标。

完整的源代码可以在 GitHub 上找到（[`github.com/PacktPublishing/Spring-5.0-By-Example/tree/master/Chapter09/turbine`](https://github.com/PacktPublishing/Spring-5.0-By-Example/tree/master/Chapter09/turbine)）。

有许多配置可以自定义涡轮服务器。这使得服务器能够适应不同的用例。

我们可以在*Spring Cloud Turbine*部分找到涡轮文档（[`cloud.spring.io/spring-cloud-netflix/single/spring-cloud-netflix.html#_turbine`](https://cloud.spring.io/spring-cloud-netflix/single/spring-cloud-netflix.html#_turbine)）。这里有大量的信息，特别是如果你需要自定义一些配置的话。

# 创建涡轮服务器微服务

让我们创建我们的涡轮服务器。我们将创建一个标准的 Spring Boot 应用程序，并添加一些注解来启用涡轮流和发现客户端。

主类应该是：

```java
package springfive.airline.turbine;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.turbine.stream.EnableTurbineStream;

@EnableEurekaClient
@EnableTurbineStream
@SpringBootApplication
public class AirlineTurbineApplication {

 public static void main(String[] args) {
    SpringApplication.run(AirlineTurbineApplication.class, args);
 }

}
```

如我们所见，`@EnableTurbineStream`将使我们能够通过 RabbitMQ 消息代理推送 Hystrix 命令指标，这对我们来说已经足够了。

Turbine 服务器的`application.yaml`文件可以在 GitHub 上找到（[`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/config-files/turbine.yaml`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/config-files/turbine.yaml)）。有几个配置，例如发现客户端和 Turbine 服务器配置。

我们可以通过命令行或 IDE 运行应用程序。运行它！

对`flights`微服务进行一些调用。创建航班 API 将调用`planes`微服务，该微服务使用 Hystrix 命令，并将触发一些 Hystrix 命令调用。

我们可以使用位于 GitHub 上的 Postman 集合（[`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/postman/flights.postman_collection`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/postman/flights.postman_collection)）。这个集合有一个创建航班请求，该请求将调用`planes`微服务以获取飞机详情。收集指标就足够了。

现在，我们可以测试我们的 Turbine 服务器是否运行正确。转到 Turbine 流端点，然后应该显示带有指标的 JSON 数据，如下所示：

![图片](img/897fd218-a456-46b6-b147-7c58822772b2.png)

有一些 Hystrix 命令信息，但正如我们所见，这些信息需要组织以便对我们有用。Turbine 使用**服务器端事件**（**SSE**）技术，该技术在第六章中介绍，即*与服务器端事件玩耍*。

在下一节中，我们将介绍 Hystrix 仪表板。它将帮助我们组织和使这些信息对我们有用。

让我们跳到下一节。

# Hystrix 仪表板

Hystrix 仪表板将帮助我们组织 Turbine 流信息。正如我们在上一节中看到的，Turbine 服务器通过 SSE 发送信息。这是通过 JSON 对象完成的。

Hystrix 流为我们提供了一个仪表板。让我们创建我们的 Hystrix 仪表板微服务。该应用程序是一个标准的带有`@EnableHystrixDashboard`注解的 Spring Boot 应用程序。让我们添加依赖项以启用它：

```java
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
```

好的，现在我们可以创建我们应用程序的主类。主类应该看起来像这样：

```java
package springfive.airline.hystrix.ui;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;

@EnableEurekaClient
@SpringBootApplication
@EnableHystrixDashboard
public class HystrixApplication {

  public static void main(String[] args) {
    SpringApplication.run(HystrixApplication.class, args);
  }

}
```

完整的源代码可以在 GitHub 上找到：[`github.com/PacktPublishing/Spring-5.0-By-Example/tree/master/Chapter09/hystrix-ui`](https://github.com/PacktPublishing/Spring-5.0-By-Example/tree/master/Chapter09/hystrix-ui)。

如我们所见，这是一个相当标准的带有`@EnableHystrixDashboard`注解的 Spring Boot 应用程序。它将为我们提供 Hystrix 仪表板。

现在，我们可以通过 IDE 或 Java 命令行运行应用程序。运行它！

Hystrix 仪表板可以通过以下 URL 访问：`http://localhost:50010/hystrix`。

然后，转到 Hystrix 仪表板的主页面。以下页面应该会显示：

![图片](img/4f178ddc-af28-4ae0-b1c6-8176be580b95.png)

太棒了——我们的 Hystrix 仪表板已经启动并运行。在这个页面上，我们可以指向 `hystrix.stream` 或 `turbine.stream` 来消费并显示命令的指标。

保持此应用程序运行，我们将在本章后面使用它。

干得好，让我们进入下一节。

# 创建 Mail 微服务

现在，我们将创建我们的 `Mail` 微服务。这个名字很直观，这个组件将负责发送电子邮件。我们不会配置一个 **SMTP** (**简单邮件传输协议**) 服务器，我们将使用 SendGrid。

SendGrid 是一个用于电子邮件的 **SaaS** (**软件即服务**) 服务，我们将使用此服务向我们的航空公司票务系统发送电子邮件。有一些触发器用于发送电子邮件，例如，当用户创建预订和当付款被接受时。

我们的 `Mail` 微服务将监听一个队列。然后，我们将使用消息代理来完成集成。我们选择这种策略是因为我们不需要能够同步回答的功能。另一个基本特征是在通信中断时的重试策略。这种行为可以通过消息策略轻松实现。

我们正在使用 RabbitMQ 作为消息代理。对于此项目，我们将使用 RabbitMQ Reactor，这是 RabbitMQ Java 客户端的响应式实现。

# 创建 SendGrid 账户

在我们开始编码之前，我们需要创建一个 SendGrid 账户。我们将使用试用账户，这对于我们的测试来说足够了。访问 SendGrid 门户 ([`sendgrid.com/`](https://sendgrid.com/)) 并点击免费试用按钮。

填写所需信息并点击创建账户按钮。

在主页面上，在左侧点击设置，然后转到 API 密钥部分，按照这里显示的图片操作：

![图片](img/f91bfd7e-d7ca-4a5f-9254-3b9028189fc4.png)

然后，我们可以点击右上角的创建 API 密钥按钮。页面应该看起来像这样：

![图片](img/5d9869f8-daef-42e2-8a4b-9da667060d62.png)

填写 API 密钥信息并选择完全访问。之后，API 密钥将出现在您的屏幕上。请将其记在一个安全的地方，因为我们很快就会将其用作环境变量。

干得好，我们的 SendGrid 账户已经准备好使用，现在我们可以编写我们的 `Mail` 微服务了。

让我们在下一节中这样做。

# 创建 Mail 微服务项目

正如我们在 第八章 中所做的那样，*断路器和安全*，我们将查看基本项目部分。我们将使用 Spring Initializr，就像我们在前面的章节中多次做的那样。

完整的源代码可以在 GitHub 上找到 ([`github.com/PacktPublishing/Spring-5.0-By-Example/tree/master/Chapter09/mail-service`](https://github.com/PacktPublishing/Spring-5.0-By-Example/tree/master/Chapter09/mail-service))。

# 添加 RabbitMQ 依赖项

让我们添加 RabbitMQ 所需的依赖项。以下依赖项应该添加：

```java
<dependency>
  <groupId>io.projectreactor.rabbitmq</groupId>
  <artifactId>reactor-rabbitmq</artifactId>
  <version>1.0.0.M1</version>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

第一个关于 RabbitMQ 的反应式实现，第二个是 starter AMQP，它将自动设置一些配置。

# 配置一些 RabbitMQ 东西

我们想要配置一些 RabbitMQ 交换机、队列和绑定。这可以通过 RabbitMQ 客户端库来完成。我们将为 `Mail` 微服务配置所需的基础设施。

我们的基础设施配置类应该看起来像这样：

```java
package springfive.airline.mailservice.infra.rabbitmq;

// imports are omitted

@Configuration
public class RabbitMQConfiguration {

  private final String pass;

  private final String user;

  private final String host;

  private final Integer port;

  private final String mailQueue;

  public RabbitMQConfiguration(@Value("${spring.rabbitmq.password}") String pass,
      @Value("${spring.rabbitmq.username}") String user,
      @Value("${spring.rabbitmq.host}") String host,
      @Value("${spring.rabbitmq.port}") Integer port,
      @Value("${mail.queue}") String mailQueue) {
    this.pass = pass;
    this.user = user;
    this.host = host;
    this.port = port;
    this.mailQueue = mailQueue;
  }

  @Bean("springConnectionFactory")
  public ConnectionFactory connectionFactory() {
    CachingConnectionFactory factory = new CachingConnectionFactory();
    factory.setUsername(this.user);
    factory.setPassword(this.pass);
    factory.setHost(this.host);
    factory.setPort(this.port);
    return factory;
  }

  @Bean
  public AmqpAdmin amqpAdmin(@Qualifier("springConnectionFactory") ConnectionFactory connectionFactory) {
    return new RabbitAdmin(connectionFactory);
  }

  @Bean
  public TopicExchange emailExchange() {
    return new TopicExchange("email", true, false);
  }

  @Bean
  public Queue mailQueue() {
    return new Queue(this.mailQueue, true, false, false);
  }

  @Bean
  public Binding mailExchangeBinding(Queue mailQueue) {
    return BindingBuilder.bind(mailQueue).to(emailExchange()).with("*");
  }

  @Bean
  public Receiver receiver() {
    val options = new ReceiverOptions();
    com.rabbitmq.client.ConnectionFactory connectionFactory = new com.rabbitmq.client.ConnectionFactory();
    connectionFactory.setUsername(this.user);
    connectionFactory.setPassword(this.pass);
    connectionFactory.setPort(this.port);
    connectionFactory.setHost(this.host);
    options.connectionFactory(connectionFactory);
    return ReactorRabbitMq.createReceiver(options);
  }

}
```

这里有一些有趣的东西，但所有这些都关于 RabbitMQ 的基础设施。这很重要，因为当我们的应用程序处于引导时间时，这意味着我们的应用程序正在准备运行。这段代码将被执行并创建必要的队列、交换机和绑定。一些配置由 `application.yaml` 文件提供，请查看构造函数。

# 模型邮件消息

我们的 `Mail` 服务是抽象的，可以用于不同的目的，因此我们将创建一个简单的类来代表我们系统中的邮件消息。我们的 `Mail` 类应该看起来像这样：

```java
package springfive.airline.mailservice.domain;

import lombok.Data;

@Data
public class Mail {

  String from;

  String to;

  String subject;

  String message;

}
```

很简单，这个类代表了我们系统中的一个抽象消息。

# `MailSender` 类

如我们所预期，我们将通过 REST API 与 SendGrid 服务集成。在我们的案例中，我们将使用 Spring WebFlux 提供的反应式 `WebClient`。

现在，我们将使用上一节中创建的 SendGrid API 密钥。我们的 `MailSender` 类应该看起来像这样：

```java
package springfive.airline.mailservice.domain.service;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpStatus;
import org.springframework.http.ReactiveHttpOutputMessage;
import org.springframework.stereotype.Service;
import org.springframework.web.reactive.function.BodyInserter;
import org.springframework.web.reactive.function.BodyInserters;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;
import springfive.airline.mailservice.domain.Mail;
import springfive.airline.mailservice.domain.service.data.SendgridMail;

@Service
public class MailSender {

  private final String apiKey;

  private final String url;

  private final WebClient webClient;

  public MailSender(@Value("${sendgrid.apikey}") String apiKey,
      @Value("${sendgrid.url}") String url,
      WebClient webClient) {
    this.apiKey = apiKey;
    this.webClient = webClient;
    this.url = url;
  }

  public Flux<Void> send(Mail mail){
    final BodyInserter<SendgridMail, ReactiveHttpOutputMessage> body = BodyInserters
      .fromObject(SendgridMail.builder().content(mail.getMessage()).from(mail.getFrom()).to(mail.getTo()).subject(mail.getSubject()).build());
    return this.webClient.mutate().baseUrl(this.url).build().post()
        .uri("/v3/mail/send")
        .body(body)
        .header("Authorization","Bearer " + this.apiKey)
        .header("Content-Type","application/json")
        .retrieve()
        .onStatus(HttpStatus::is4xxClientError, clientResponse ->
            Mono.error(new RuntimeException("Error on send email"))
        ).bodyToFlux(Void.class);
  }

}
```

我们在构造函数中收到了配置，即 `sendgrid.apikey` 和 `sendgrid.url`。它们将很快被配置。在 `send()` 方法中，有一些有趣的结构。看看 `BodyInserters.fromObject()`：它允许我们在 HTTP 主体中发送一个 JSON 对象。在我们的案例中，我们将创建一个 `SendGrid` 邮件对象。

在 `onStatus()` 函数中，我们可以传递一个谓词来处理 HTTP 错误系列。在我们的案例中，我们对 4xx 错误系列感兴趣。

这个类将处理发送邮件消息，但我们需要监听 RabbitMQ 队列，这将在下一节中完成。

# 创建 RabbitMQ 队列监听器

让我们创建我们的 `MailQueueConsumer` 类，它将监听 RabbitMQ 队列。这个类应该看起来像这样：

```java
package springfive.airline.mailservice.domain.service;

import com.fasterxml.jackson.databind.ObjectMapper;
import java.io.IOException;
import javax.annotation.PostConstruct;
import lombok.extern.slf4j.Slf4j;
import lombok.val;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import reactor.rabbitmq.Receiver;
import springfive.airline.mailservice.domain.Mail;

@Service
@Slf4j
public class MailQueueConsumer {

  private final MailSender mailSender;

  private final String mailQueue;

  private final Receiver receiver;

  private final ObjectMapper mapper;

  public MailQueueConsumer(MailSender mailSender, @Value("${mail.queue}") String mailQueue,
      Receiver receiver, ObjectMapper mapper) {
    this.mailSender = mailSender;
    this.mailQueue = mailQueue;
    this.receiver = receiver;
    this.mapper = mapper;
  }

  @PostConstruct
  public void startConsume() {
    this.receiver.consumeAutoAck(this.mailQueue).subscribe(message -> {
      try {
        val mail = this.mapper.readValue(new String(message.getBody()), Mail.class);
        this.mailSender.send(mail).subscribe(data ->{
          log.info("Mail sent successfully");
        });
      } catch (IOException e) {
        throw new RuntimeException("error on deserialize object");
      }
    });
  }

}
```

注解了 `@PostConstruct` 的方法将在 `MailQueueConsumer` 准备就绪后调用，这意味着注入已经处理。然后 `Receiver` 将开始处理消息。

# 运行 Mail 微服务

现在，我们将运行我们的 `Mail` 微服务。找到 `MailServiceApplication` 类，这是我们的项目的主类。主类应该看起来像这样：

```java
package springfive.airline.mailservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.hystrix.EnableHystrix;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@EnableHystrix
@EnableZuulProxy
@EnableEurekaClient
@SpringBootApplication
public class MailServiceApplication {

  public static void main(String[] args) {
    SpringApplication.run(MailServiceApplication.class, args);
  }

}
```

这是一个标准的 Spring Boot 应用程序。

我们可以在 IDE 中或通过 Java 命令行运行应用程序。

运行它！

我们需要传递 `${SENDGRID_APIKEY}` 和 `${SENDGRID_URL}` 作为环境变量。如果你使用 Java 命令行运行应用程序，`-D` 选项允许我们传递环境变量。如果你使用 IDE，你可以在运行/调试配置中配置。

# 创建身份验证微服务

我们想要保护我们的微服务。对于微服务应用程序来说，安全性至关重要，尤其是由于分布式特性。

在微服务架构风格中，通常有一个服务将充当认证服务。这意味着此服务将认证我们微服务组中的请求。

Spring Cloud Security 提供了一个声明性模型，以帮助开发者启用应用程序的安全性。它支持如 OAuth 2.0 等常见模式。此外，Spring Boot Security 还支持**单点登录**（**SSO**）。

Spring Boot Security 还支持与 Zuul 代理集成的中继 SSO 令牌。这意味着令牌将被传递到下游微服务。

对于我们的架构，我们将使用 OAuth 2.0 和 JWT 模式，两者都与 Zuul 代理集成。

在这样做之前，让我们了解 OAuth 2.0 流程中的主要实体：

+   **受保护资源**：此服务将应用安全规则；在我们的案例中是微服务应用程序

+   **OAuth 授权服务器**：认证服务器是位于应用程序（可以是前端或移动端）和应用程序想要调用的服务之间的一个服务

+   **应用程序**：将要调用服务的应用程序，即客户端。

+   **资源所有者**：将授权客户端应用程序访问其账户的用户或机器

让我们绘制基本的 OAuth 流程：

![图片](img/917002f4-1c25-4b28-80a2-3994a64c0db5.png)

我们可以从这张图中观察到以下内容：

1.  **客户端**请求授权

1.  **资源所有者**发送授权许可

1.  应用程序客户端从**授权服务器**请求访问令牌

1.  如果授权许可有效，**授权服务器**将提供访问令牌

1.  应用程序调用受保护的资源并发送访问令牌

1.  如果**资源服务器**识别了令牌，该资源将为应用程序提供服务

这些是 OAuth 2.0 授权流程的基础。我们将使用 Spring Cloud Security 实现此流程。让我们开始做。

# 创建 Auth 微服务

正如我们在本章中所做的那样，我们将查看重要部分。让我们从我们的依赖项开始。我们需要添加以下依赖项：

```java
  <dependency>
 <groupId>org.springframework.cloud</groupId>
 <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>

  <dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-core</artifactId>
  </dependency>

  <dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
  </dependency>
```

这些依赖项将使我们能够使用 Spring Cloud Security 功能。让我们开始编写我们的认证微服务。

# 配置安全

让我们开始编写我们的`Auth`微服务。我们将从授权和认证开始，因为我们想要保护我们微服务中的所有资源，然后我们将配置`WebSecurityConfigureAdapter`。该类应该看起来像这样：

```java
package springfive.airline.authservice.infra.security;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.password.PasswordEncoder;
import springfive.airline.authservice.service.CredentialsDetailsService;

@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

  private final PasswordEncoder passwordEncoder;

  private final CredentialsDetailsService credentialUserDetails;

  public SecurityConfig(PasswordEncoder passwordEncoder,
      CredentialsDetailsService credentialUserDetails) {
    this.passwordEncoder = passwordEncoder;
    this.credentialUserDetails = credentialUserDetails;
  }

  @Override
  @Autowired
 protected void configure(AuthenticationManagerBuilder auth) throws Exception {
 auth.userDetailsService(this.credentialUserDetails).passwordEncoder(this.passwordEncoder);
  }

  @Override
  protected void configure(HttpSecurity http) throws Exception {
 http.csrf().disable()
 .authorizeRequests()
 .antMatchers("/login", "/**/register/**").permitAll()
 .anyRequest().authenticated()
 .and()
 .formLogin().permitAll();
  }

}
```

这里有很多内容。让我们从 `@EnableWebSecurity` 注解开始，这个注解使 Spring Security 能够与 Spring MVC 集成。`@EnableGlobalMethodSecurity` 提供了 AOP 拦截器，以使用注解启用方法安全。我们可以通过在控制器上的方法上注解来使用此功能，例如。基本思想是将方法调用包装在 AOP 拦截器中，并在方法上应用安全。

`WebSecurityConfigurerAdapter` 允许我们配置安全的端点和一些关于如何认证用户的内容，这可以通过使用 `configure(AuthenticationManagerBuilder auth)` 方法来完成。我们已经配置了我们的 `CredentialsDetailsService` 和 `PasswordEncoder` 以避免在应用程序层之间传递明文密码。在这种情况下，`CredentialsDetailsService` 是我们用户数据的来源。

在我们的 `configure(HttpSecurity http)` 方法中，我们配置了一些 HTTP 安全规则。正如我们所看到的，所有用户都可以访问 `/login` 和 `/**/register/**`。这是关于 *登录* 和 *注册* 功能。所有其他请求都需要通过授权服务器进行认证。

`CredentialsDetailsService` 应该看起来像这样：

```java
package springfive.airline.authservice.service;

import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Component;
import springfive.airline.authservice.domain.Credential;
import springfive.airline.authservice.domain.data.CredentialData;
import springfive.airline.authservice.repository.CredentialRepository;

@Component
public class CredentialsDetailsService implements UserDetailsService {

  private final CredentialRepository credentialRepository;

  public CredentialsDetailsService(CredentialRepository credentialRepository) {
    this.credentialRepository = credentialRepository;
  }

  @Override
  public CredentialData loadUserByUsername(String email) throws UsernameNotFoundException {
    final Credential credential = this.credentialRepository.findByEmail(email);
    return CredentialData.builder().email(credential.getEmail()).password(credential.getPassword()).scopes(credential.getScopes()).build();
  }

}
```

这里没有什么特别之处。我们需要重写 `loadUserByUsername(String email)` 方法来提供用户数据给 Spring Security。

让我们配置我们的令牌签名者和令牌存储。我们将使用 `@Configuration` 类提供这些 Bean，就像我们在前面的章节中所做的那样：

```java
package springfive.airline.authservice.infra.oauth;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.oauth2.provider.token.store.JwtAccessTokenConverter;
import org.springframework.security.oauth2.provider.token.store.JwtTokenStore;

@Configuration
public class OAuthTokenProducer {

    @Value("${config.oauth2.privateKey}")
    private String privateKey;

    @Value("${config.oauth2.publicKey}")
    private String publicKey;

    @Bean
    public JwtTokenStore tokenStore(JwtAccessTokenConverter tokenEnhancer) {
        return new JwtTokenStore(tokenEnhancer);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public JwtAccessTokenConverter tokenEnhancer() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setSigningKey(privateKey);
        converter.setVerifierKey(publicKey);
        return converter;
    }

}
```

我们在 `application.yaml` 文件中配置了我们的私有和公开密钥。可选地，我们还可以从类路径中读取 `jks` 文件。然后，我们使用 `JwtAccessTokenConverter` 类提供了我们的令牌签名者或令牌增强器，其中我们使用了私有和公开密钥。

在我们的令牌存储中，Spring Security 框架将使用此对象从令牌中读取数据，然后在 `JwtTokenStore` 实例上设置 `JwtAccessTokenConverter`。

最后，我们使用 `BCryptPasswordEncoder` 类提供了密码编码器类。

我们最后的类是授权服务器配置。配置可以使用以下类来完成：

查看位于 GitHub 上的 `OAuth2AuthServer` 类（[`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/auth-service/src/main/java/springfive/airline/authservice/infra/oauth/OAuth2AuthServer.java`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/auth-service/src/main/java/springfive/airline/authservice/infra/oauth/OAuth2AuthServer.java)）。

我们在 `Auth` 微服务中使用了 `@EnableAuthorizationServer` 来配置授权服务器机制。这个类与 `AuthorizationServerConfigurerAdapter` 一起工作，以提供一些自定义设置。

在 `configure(AuthorizationServerSecurityConfigurer oauthServer)` 上，我们配置了令牌端点的安全设置。

在 `configure(AuthorizationServerEndpointsConfigurer endpoints)` 中，我们已配置了令牌服务的端点，例如 `/oauth/token` 和 `/oauth/authorize`。

最后，在配置 (`ClientDetailsServiceConfigurer clients`) 中，我们已配置了客户端的 ID 和密钥。我们使用了内存数据，但也可以使用 JDBC 实现。

`Auth` 微服务的主类应该是：

```java
package springfive.airline.authservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@EnableZuulProxy
@EnableEurekaClient @SpringBootApplication
public class AuthServiceApplication {

 public static void main(String[] args) {
  SpringApplication.run(AuthServiceApplication.class, args);
 }

}
```

在这里，我们创建了一个带有服务发现和 Zuul 代理的标准 Spring Boot 应用程序。

# 测试 Auth 微服务

如我们所见，`Auth` 微服务已准备好测试。我们的微服务正在监听端口 `7777`，这是我们在 GitHub 上的 `application.yaml` 文件中配置的。

# 客户端凭证流程

让我们从客户端凭证流程开始。

我们的应用程序需要运行在端口 `7777` 上，然后我们可以使用以下命令行来获取令牌：

```java
curl -s 442cf4015509eda9c03e5ca3aceef752:4f7ec648a48b9d3fa239b497f7b6b4d8019697bd@localhost:7777/oauth/token   -d grant_type=client_credentials  -d scope=trust | jq .
```

如我们所见，这个 *客户端 ID* 和 *客户端密钥* 来自 `planes` 微服务。我们在 `OAuth2AuthServer` 类中进行了此配置。让我们记住确切的位置：

```java
....
@Override
public void configure(ClientDetailsServiceConfigurer clients)throws Exception {
  clients
      .inMemory()
      .withClient("ecommerce") // ecommerce microservice
      .secret("9ecc8459ea5f39f9da55cb4d71a70b5d1e0f0b80")
      .authorizedGrantTypes("authorization_code", "refresh_token", "implicit",
          "client_credentials")
      .authorities("maintainer", "owner", "user")
      .scopes("read", "write")
      .accessTokenValiditySeconds(THREE_HOURS)
      .and()
      .withClient("442cf4015509eda9c03e5ca3aceef752") // planes microservice
      .secret("4f7ec648a48b9d3fa239b497f7b6b4d8019697bd")
 .authorizedGrantTypes("authorization_code", "refresh_token", "implicit",
          "client_credentials")
 .authorities("operator")
 .scopes("trust")
      .accessTokenValiditySeconds(ONE_DAY)

....
```

在调用上述命令后，结果应该是：

![图片](img/13bf0165-4bc6-43a7-a57b-3f314d22a2f2.png)

如我们所见，令牌已成功获取。做得好，我们的客户端凭证流程已成功配置。让我们转到隐式流程，这将在下一节中介绍。

# 隐式授权流程

在本节中，我们将探讨如何使用隐式流程在我们的 `Auth` 微服务中进行身份验证。

在我们测试我们的流程之前，让我们创建一个用户以在 `Auth` 微服务中启用身份验证。以下命令将在 `Auth` 服务中创建一个用户：

```java
curl -H "Content-Type: application/json" -X POST -d '{"name":"John Doe","email":"john@doe.com", "password" : "john"}' http://localhost:7777/register
```

如我们所见，电子邮件是 `john@doe.com`，密码是 `john`。

我们将使用浏览器来完成这个任务。让我们访问以下 URL：

`http://localhost:7777/oauth/authorize?client_id=ecommerce&response_type=token&scope=write&state=8777&redirect_uri=https://httpbin.org/anything`

让我们了解这些参数：

第一部分是服务地址。要使用隐式授权流程，我们需要路径 `/oauth/authorize`。我们还将使用 `ecommerce` 作为客户端 ID，因为我们之前已经配置了它。`response_type=token` 通知隐式流程，`scope` 是我们想要的范围，在我们的案例中是写权限，`state` 是一个随机变量，`redirect_uri` 是 `oauth` 登录过程之后的 URI。

将 URL 放入网页浏览器中，应该会显示以下页面：

![图片](img/49f91e24-a935-420e-81b0-cb11e20051be.png)

在输入用户名和密码后，将显示以下页面以授权我们的受保护资源：

![图片](img/34998698-9d77-44ee-8a59-60d7fb1309da.png)

点击授权按钮。然后我们将在浏览器 URL 中看到令牌，如下所示：

![图片](img/6e1a2819-63ac-45c6-a937-04098bee29aa.png)

如果我们复制浏览器 URL，可以查看完整的令牌。

伙计们，干得好，我们的 `Auth` 微服务完全可用。

在接下来的几节中，我们将配置`Auth`微服务以保护 Zuul 代理下游微服务，例如`planes`微服务。让我们跳到下一节。

# 使用 OAuth 2.0 保护微服务

现在，我们将配置 OAuth 2.0 来保护我们的微服务；在我们的案例中，我们的微服务是资源服务器。让我们从`planes`微服务开始。我们将添加新的依赖项并配置私钥和公钥。同时，我们还将配置我们的`JwtTokenStore`。

让我们开始吧。

# 添加安全依赖

为了添加新要求的依赖项，我们将更改`planes`微服务的`pom.xml`文件。我们将添加以下依赖项：

```java
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>

```

小菜一碟——我们所需的依赖项已经正确配置。

在下一节中，我们将配置`application.yaml`文件。

# 配置 application.yaml 文件

为了配置我们的私钥和公钥，我们将使用`application.yaml`文件。我们在`Auth`微服务中进行了此配置。配置相当简单。我们需要添加以下片段：

```java
config:
  oauth2:
    privateKey: |
      -----BEGIN RSA PRIVATE KEY-----
      MIICXQIBAAKBgQDNQZKqTlO/+2b4ZdhqGJzGBDltb5PZmBz1ALN2YLvt341pH6i5
      mO1V9cX5Ty1LM70fKfnIoYUP4KCE33dPnC7LkUwE/myh1zM6m8cbL5cYFPyP099t
      hbVxzJkjHWqywvQih/qOOjliomKbM9pxG8Z1dB26hL9dSAZuA8xExjlPmQIDAQAB
      AoGAImnYGU3ApPOVtBf/TOqLfne+2SZX96eVU06myDY3zA4rO3DfbR7CzCLE6qPn
      yDAIiW0UQBs0oBDdWOnOqz5YaePZu/yrLyj6KM6Q2e9ywRDtDh3ywrSfGpjdSvvo
      aeL1WesBWsgWv1vFKKvES7ILFLUxKwyCRC2Lgh7aI9GGZfECQQD84m98Yrehhin3
      fZuRaBNIu348Ci7ZFZmrvyxAIxrV4jBjpACW0RM2BvF5oYM2gOJqIfBOVjmPwUro
      bYEFcHRvAkEAz8jsfmxsZVwh3Y/Y47BzhKIC5FLaads541jNjVWfrPirljyCy1n4
      sg3WQH2IEyap3WTP84+csCtsfNfyK7fQdwJBAJNRyobY74cupJYkW5OK4OkXKQQL
      Hp2iosJV/Y5jpQeC3JO/gARcSmfIBbbI66q9zKjtmpPYUXI4tc3PtUEY8QsCQQCc
      xySyC0sKe6bNzyC+Q8AVvkxiTKWiI5idEr8duhJd589H72Zc2wkMB+a2CEGo+Y5H
      jy5cvuph/pG/7Qw7sljnAkAy/feClt1mUEiAcWrHRwcQ71AoA0+21yC9VkqPNrn3
      w7OEg8gBqPjRlXBNb00QieNeGGSkXOoU6gFschR22Dzy
      -----END RSA PRIVATE KEY-----
    publicKey: |
      -----BEGIN PUBLIC KEY-----
      MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDNQZKqTlO/+2b4ZdhqGJzGBDlt
      b5PZmBz1ALN2YLvt341pH6i5mO1V9cX5Ty1LM70fKfnIoYUP4KCE33dPnC7LkUwE
      /myh1zM6m8cbL5cYFPyP099thbVxzJkjHWqywvQih/qOOjliomKbM9pxG8Z1dB26
      hL9dSAZuA8xExjlPmQIDAQAB
      -----END PUBLIC KEY-----
```

此外，用户信息 URI 将通过以下 YAML 配置完成：

```java
  oauth2:
    resource:
      userInfoUri: http://localhost:7777/credential
```

太棒了——我们的应用程序已经完全配置好了。现在，我们将进行最后一部分：配置以获取信息令牌。

让我们这样做。

# 创建 JwtTokenStore Bean

我们将创建`JwtTokenStore`，它将被用来获取令牌信息。这个类应该看起来像这样：

```java
package springfive.airline.airlineplanes.infra.oauth;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.oauth2.provider.token.store.JwtAccessTokenConverter;
import org.springframework.security.oauth2.provider.token.store.JwtTokenStore;

@Configuration
public class OAuthTokenConfiguration {

  @Value("${config.oauth2.privateKey}")
  private String privateKey;

  @Value("${config.oauth2.publicKey}")
  private String publicKey;

  @Bean
  public JwtTokenStore tokenStore() throws Exception {
    JwtAccessTokenConverter enhancer = new JwtAccessTokenConverter();
    enhancer.setSigningKey(privateKey);
    enhancer.setVerifierKey(publicKey);
    enhancer.afterPropertiesSet();
    return new JwtTokenStore(enhancer);
  }

}
```

太棒了——我们的令牌签名者已经配置好了。

最后，我们将向主类添加以下注解，它应该看起来像这样：

```java
package springfive.airline.airlineplanes;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer;

@EnableZuulProxy
@EnableEurekaClient
@EnableResourceServer
@SpringBootApplication
public class AirlinePlanesApplication {

 public static void main(String[] args) {
  SpringApplication.run(AirlinePlanesApplication.class, args);
 }

}
```

它将保护我们的应用程序，并且访问应用程序端点需要访问令牌。

记住，我们需要为我们想要保护的所有的微服务执行相同的任务。

# 监控微服务

在微服务架构风格中，监控是一个关键部分。当我们采用这种架构时，有很多好处，比如上市时间、源维护和业务性能的提升。这是因为我们可以将业务目标分配给不同的团队，每个团队将负责一些微服务。另一个重要特征是计算资源的优化，比如云计算成本。

如我们所知，没有免费的午餐，这种风格带来了一些缺点，比如操作复杂性。有很多*小服务*需要监控。可能有数百个不同的服务实例。

我们在我们的基础设施中实现了一些这些服务，但直到现在，我们还没有数据来分析我们的系统健康。在本节中，我们将探索我们配置的服务。

让我们立即分析！

# 使用 Zipkin 收集指标

我们在上一章中已经配置了我们的 Zipkin 服务器。现在我们将使用这个服务器来分析我们的微服务数据。让我们开始吧。

进行一些调用以创建航班。创建航班 API 将调用**认证服务**和**航班服务**。查看以下图表：

![图片](img/4d874e0e-edc0-4f67-b1fb-26d3cc25dbce.png)

我们将查看`flights`微服务和`planes`微服务之间的通信。让我们来分析它：

前往 Zipkin 主页面，`http://localhost:9999/`，选择航班，然后点击“查找跟踪”。页面应该看起来像这样：

![图片](img/0f5af854-2b36-4d7d-ac13-044f49d8ad00.png)

如我们所见，我们的 Zipkin 服务器上有一些数据。点击“跨度”，它带有`flights`和`planes`标签，然后我们将查看这个特定的跟踪，并且将被重定向到另一个页面，其中包含特定的跨度数据，如下所示：

![图片](img/9378b77e-cf44-41e0-a450-89baa56db7a6.png)

在这个页面上，我们可以看到一些重要信息，例如总请求时间。然后点击“平面”行，我们将会看到以下图像中的详细信息：

![图片](img/87230629-9881-4fc9-bd39-fe2636017523.png)

查看请求信息。这里有一些有趣的内容，例如`mvc.controller.class`和`mvc.controller.method`。这些内容有助于开发者排查错误。在第一个面板中，我们还看到了服务交互的时间。这对于查找微服务网络延迟非常有帮助；例如，它使得环境管理变得更加容易，因为我们有可视化工具来更好地理解数据。

此外，Zipkin 服务器还提供了其他一些有趣的功能来查找微服务统计信息，例如查找延迟超过特定时间的请求。这对运维人员非常有帮助。

我们可以在文档页面（[`cloud.spring.io/spring-cloud-static/spring-cloud-sleuth/2.0.0.M5/single/spring-cloud-sleuth.html`](http://cloud.spring.io/spring-cloud-static/spring-cloud-sleuth/2.0.0.M5/single/spring-cloud-sleuth.html)）或 GitHub（[`github.com/spring-cloud/spring-cloud-sleuth`](https://github.com/spring-cloud/spring-cloud-sleuth)）项目页面上找到更多关于 Spring Cloud Sleuth 的信息。

# 使用 Hystrix 收集命令统计

现在，我们想要监控我们的 Hystrix 命令。在我们的微服务中存在多个命令，可能最常用的是 OAuth 令牌请求者，因为我们总是需要有一个令牌来调用我们系统中的任何微服务。我们的 Turbine 服务器和 Hystrix UI 在本章的开头就已经配置好了，我们现在将使用这些服务。

记住，我们正在使用`spring-cloud-netflix-hystrix-stream`作为实现，将 Hystrix 数据发送到 Turbine 服务器，因为它比 HTTP 性能更好，同时也带来了一些异步特性。

异步调用可以使微服务更加健壮。在这种情况下，我们不会使用 HTTP 调用（同步调用）来注册 Hystrix 命令统计信息。我们将使用 RabbitMQ 队列来注册它。在这种情况下，我们将消息放入队列。此外，异步调用使我们的应用程序更优化地使用计算资源。

运行 Turbine 服务器应用程序和 Hystrix UI 应用程序。Turbine 将聚合来自服务器的指标。可选地，你可以运行相同服务的多个实例，例如`flights`。Turbine 将正确地聚合统计信息。

让我们调用创建航班 API；我们可以使用 Postman 来完成这个操作。

然后，我们可以看到实时的命令统计信息。在此之前，我们将在 Hystrix 仪表板中配置`turbine.stream`。

前往 Hystrix 仪表板页面：`http://localhost:50010/hystrix/`。以下页面将会显示：

![](img/d50d157c-7333-4ef6-a67e-e5dac0d957aa.png)

然后，我们有一些工作要做。让我们配置我们的 Turbine 服务器流。我们的 Turbine 流运行在`http://localhost:8010/turbine.stream`。将此信息放在 Hystrix 仪表板信息下方，然后我们可以点击监控流按钮。

我们将重定向到 Hystrix 命令仪表板；我们之前调用过创建航班 API 几次。命令指标将显示，如下面的图像所示：

![](img/42a34f54-fc7b-4bdd-9152-afafc3242b54.png)

如我们所见，我们调用了创建航班 API 八次。这个 API 使用了一些命令，例如`flights.plane-by-id`，它调用飞机微服务，而`flights.request-token`调用`Auth`服务。

看看监控命令有多简单。运维人员喜欢使用 Zipkin 服务器这样的页面。

了不起的工作，大家，我们的服务集成得到了充分的监控，这使得我们的微服务采用更加舒适，因为我们有有用的应用程序来监控我们的服务实例。

# 微服务 Docker 化

在前面的章节中，我们使用了 Fabric8 Maven Docker 插件来启用我们使用 Maven 目标创建 Docker 镜像。

现在，我们需要配置我们的微服务以使用此插件，以便轻松为我们创建镜像。与一些持续集成和持续交付工具（如 Jenkins）集成可能会有所帮助，因为我们可以轻松调用`docker: build`目标。

每个项目都有自定义配置，例如端口和镜像名称。我们可以在 GitHub 仓库中找到配置。记住，配置是通过`pom.xml`完成的。

以下列表包含了所有项目的 GitHub 仓库地址；`pom.xml`文件中包含了 Maven Docker 插件的配置：

+   **航班**：[`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/airline-flights/pom.xml`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/airline-flights/pom.xml)

+   **飞机**: [`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/airline-planes/pom.xml`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/airline-planes/pom.xml)

+   **票价**: [`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/airline-fare/pom.xml`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/airline-fare/pom.xml)

+   **预订**: [`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/airline-booking/pom.xml`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/airline-booking/pom.xml)

+   **管理员**: [`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/admin/pom.xml`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/admin/pom.xml)

+   **边缘**: [`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/api-edge/pom.xml`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/api-edge/pom.xml)

+   **乘客**: [`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/airline-passengers/pom.xml`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/airline-passengers/pom.xml)

+   **认证**: [`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/auth-service/pom.xml`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/auth-service/pom.xml)

+   **邮件**: [`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/mail-service/pom.xml`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/mail-service/pom.xml)

+   **涡轮机**: [`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/turbine/pom.xml`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/turbine/pom.xml)

+   **Zipkin**: [`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/zipkin-server/pom.xml`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/zipkin-server/pom.xml)

+   **支付**: [`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/airline-payments/pom.xml`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/airline-payments/pom.xml)

+   **Hystrix 仪表板**: [`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/hystrix-ui/pom.xml`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/hystrix-ui/pom.xml)

+   **发现**: [`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/eureka/pom.xml`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/eureka/pom.xml)

+   **配置服务器**：[`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/config-server/pom.xml`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/Chapter09/config-server/pom.xml)

# 运行系统

现在，我们可以使用上一节创建的镜像来运行我们的 Docker 容器。

我们将服务分成两个 Docker Compose 文件。第一个是关于基础设施服务。第二个是关于我们的微服务。

这些堆栈必须在同一个 Docker 网络上运行，因为服务应该通过容器主机名连接。

基础设施的 Docker Compose 文件可以在 GitHub 上找到：[`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/stacks/docker-compose-infra.yaml`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/stacks/docker-compose-infra.yaml)。

微服务的 Docker Compose 文件可以在 GitHub 上找到：[`github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/stacks/docker-compose-micro.yaml`](https://github.com/PacktPublishing/Spring-5.0-By-Example/blob/master/stacks/docker-compose-micro.yaml)。

现在，我们可以使用 `docker-compose` 命令运行这些文件。请输入以下命令：

```java
docker-compose -f docker-compose-infra.yaml up -d
docker-compose -f docker-compose-micro.yaml up -d
```

然后，完整的应用程序将启动并运行。

干得好，伙计们。

# 摘要

在本章中，我们学习了关于微服务架构的一些重要要点。

我们被介绍了一些用于监控微服务环境的工具。我们学习了如何使用 Turbine 服务器在分布式环境中帮助我们监控 Hystrix 命令。

我们还介绍了 Hystrix 仪表板功能，它帮助开发者和运维人员提供一个包含命令统计信息的丰富仪表板，这些统计信息几乎可以实时提供。

我们学习了如何使用 Spring Cloud Security 为我们的微服务启用安全功能，并实现了 OAuth 2 服务器，使用 JWT 为我们的安全层提供弹性。
