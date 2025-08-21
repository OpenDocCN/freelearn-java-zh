# 第十二章：保护 API

安全性是关于微服务架构的最常讨论的问题之一。对于所有安全关注，总是有一个主要问题——网络。在微服务中，通常网络通信比单体应用程序多，因此应该重新考虑认证和授权的方法。传统的系统通常在边界处进行安全保护，然后允许前端服务完全访问后端组件。微服务的迁移迫使我们改变这种委托访问管理的方法。

Spring Framework 是如何解决基于微服务的架构的安全问题的？它提供了几个项目，实现了关于认证和授权的不同模式。这些项目中的第一个是 Spring Security，它是基于 Spring 的 Java 应用程序的安全事实标准。它包括几个子模块，可以帮助你开始使用 SAML、OAuth2 或 Kerberos。还有 Spring Cloud Security 项目。它提供了几个组件，允许你将 Spring Security 的基本功能与微服务架构的主要元素（如网关、负载均衡器和 REST HTTP 客户端）集成。

在本章中，我将向您展示如何保护您基于微服务的系统中的所有主要组件。我将描述与主题相关的特定元素，按照构成本书第二部分的章节的顺序。所以，我们将从使用 Eureka 的服务发现开始，然后转移到 Spring Cloud Config Server 和跨服务通信，最后讨论 API 网关安全。

在本章中，我们将探讨以下内容：

+   为单个 Spring Boot 应用程序配置安全连接

+   为微服务架构的最重要元素启用 HTTPS 通信

+   在 Config Server 上存储的配置文件中加密和解密属性值

+   为微服务使用基于 OAuth2 的简单内存身份验证

+   使用 JDBC 后端存储和 JWT 令牌进行更高级的 OAuth2 配置

+   在 Feign 客户端中使用 OAuth2 授权进行服务间通信

但是首先，让我们从基础知识开始。我将向您展示如何创建第一个安全微服务，该微服务通过 HTTPS 暴露 API。

# 为 Spring Boot 启用 HTTPS

如果您想要使用 SSL 并为您提供 RESTful API 的 HTTPS 服务，您需要生成一个证书。实现这一目标最快的途径是通过自签名证书，这对于开发模式来说已经足够了。JRE 提供了一个简单的证书管理工具——`keytool`。它位于您的`JRE_HOME\bin`目录下。以下代码中的命令生成一个自签名证书并将其放入 PKCS12 KeyStore 中。除了 KeyStore 的类型之外，您还需要设置其有效期、别名以及文件名。在开始生成过程之前，`keytool`会要求您输入密码和一些其他信息，如下所示：

```java
keytool -genkeypair -alias account-key -keyalg RSA -keysize 2048 -storetype PKCS12 -keystore account-key.p12 -validity 3650

Enter keystore password:
Re-enter new password:
What is your first and last name?
 [Unknown]: localhost
What is the name of your organizational unit?
 [Unknown]: =
What is the name of your organization?
 [Unknown]: piomin
What is the name of your City or Locality?
 [Unknown]: Warsaw
What is the name of your State or Province?
 [Unknown]: mazowieckie
What is the two-letter country code for this unit?
 [Unknown]: PL
Is CN=localhost, OU=Unknown, O=piomin, L=Warsaw, ST=mazowieckie, C=PL correct?
 [no]: yes
```

我已经将生成的证书复制到了 Spring Boot 应用程序内的`src/main/resources`目录中。在构建并运行应用程序后，它将出现在类路径上。为了启用 SSL，我们必须在`application.yml`文件中提供一些配置设置。通过设置各种`server.ssl.*`属性，可以为 Spring 自定义 SSL：

```java
server: 
 port: ${PORT:8090}

ssl:
 key-store: classpath:account-key.p12
 key-store-password: 123456
 key-store-type: PKCS12
 key-alias: account-key

security:
 require-ssl: true
```

# 安全发现

正如您所看到的，为微服务应用程序配置 SSL 并不是一个非常困难的任务。然而，现在是提高难度级别的时候了。我们已经启动了一个单一的微服务，它通过 HTTPS 提供 RESTful API。现在我们想要这个微服务与发现服务器集成。由此产生的两个问题，首先是需要在 Eureka 中发布关于安全微服务实例的信息。第二个问题是如何通过 HTTPS 暴露 Eureka，并强制发现客户端使用私钥对发现服务器进行身份验证。让我们详细讨论这些问题。

# 注册安全应用程序

如果您的应用程序通过安全的 SSL 端口暴露，您应该将`EurekaInstanceConfig`中的两个标志更改为`nonSecurePortEnabled`为`false`和`securePortEnabled`为`true`。这使得 Eureka 发布显式偏好安全通信的实例信息。对于这样配置的服务，Spring Cloud `DiscoveryClient`总是会返回一个以 HTTPS 开头的 URL，并且 Eureka 实例信息将有一个安全的健康检查 URL：

```java
eureka:
 instance:
  nonSecurePortEnabled: false
  securePortEnabled: true
  securePort: ${PORT:8091}
  statusPageUrl: https://localhost:${eureka.instance.securePort}/info
  healthCheckUrl: https://localhost:${eureka.instance.securePort}/health
  homePageUrl: https://localhost:${eureka.instance.securePort}
```

# 通过 HTTPS 提供 Eureka

当使用 Spring Boot 启动 Eureka 服务器时，它部署在嵌入式 Tomcat 容器中，因此 SSL 配置与标准微服务相同。区别在于我们必须考虑客户端应用程序，它通过 HTTPS 与发现服务器建立安全连接。发现客户端应该对自己进行身份验证，以对抗 Eureka 服务器，并且还应该验证服务器的证书。客户端和服务器之间的这种通信过程称为**双向 SSL**或**相互认证**。还有一种单向认证，实际上是默认选项，其中只有客户端验证服务器的公钥。Java 应用程序使用 KeyStore 和 trustStore 来存储与公钥对应的私钥和证书。trustStore 和 KeyStore 之间的唯一区别在于它们存储的内容和目的。当客户端和服务器之间执行 SSL 握手时，trustStore 用于验证凭据，而 KeyStore 用于提供凭据。换句话说，KeyStore 为给定应用程序保存私钥和证书，而 trustStore 保存用于从第三方识别它的证书。开发者在配置安全连接时通常不会过多关注这些术语，但正确理解它们可以帮助您轻松了解接下来会发生什么。

在典型的基于微服务的架构中，有大量的独立应用程序和一个发现服务器。每个应用程序都有自己的私钥存储在 KeyStore 中，以及对应于发现服务器公钥的证书存储在 trustStore 中。另一方面，服务器保留了为客户端应用程序生成的所有证书。现在我们已经有了足够多的理论。让我们看看下面的图表。它说明了我们在前几章中用作示例的系统的当前情况：

![](img/4bdd8fff-ba4d-4694-82b7-5cd6658b7582.png)

# Keystore 生成

在讨论了 Java 安全性的基础知识之后，我们可以继续生成微服务的私钥和公钥。像以前一样，我们将使用 JRE 下的命令行工具——`keytool`。让我们从一个生成`keystore`文件的键对的知名命令开始。一个 KeyStore 为发现服务器生成，另一个为选定的微服务生成，在本例中，为`account-service`生成：

```java
keytool -genkey -alias account -store  type JKS -keyalg RSA -keysize 2048 -keystore account.jks -validity 3650
keytool -genkey -alias discovery -storetype JKS -keyalg RSA -keysize 2048 -keystore discovery.jks -validity 3650
```

然后，必须将自签名证书从 KeyStore 导出到文件中——例如，具有`.cer`或`.crt`扩展名。然后系统会提示您输入在生成 KeyStore 时提供的密码：

```java
keytool -exportcert -alias account -keystore account.jks -file account.cer
keytool -exportcert -alias discovery -keystore discovery.jks -file discovery.cer
```

从 KeyStore 中提取了与公钥对应的证书，因此现在它可以分发给所有感兴趣的各方。`account-service`的公共证书应该包含在发现服务器的 trustStore 中，反之亦然：

```java
keytool -importcert -alias discovery -keystore account.jks -file discovery.cer
keytool -importcert -alias account -keystore discovery.jks -file account.cer
```

对`account-service`执行的相同步骤也必须重复应用于每个随后注册自己的 Eureka 服务器的微服务。以下是`order-service`生成 SSL 密钥和证书时使用的`keytool`命令：

```java
keytool -genkey -alias order -storetype JKS -keyalg RSA -keysize 2048 -keystore order.jks -validity 3650
keytool -exportcert -alias order -keystore order.jks -file order.cer
keytool -importcert -alias discovery -keystore order.jks -file discovery.cer
keytool -importcert -alias order -keystore discovery.jks -file order.cer
```

# 为微服务和 Eureka 服务器配置 SSL

每个`keystore`文件都被放置在每个安全微服务和服务发现`src/main/resources`目录中。每个微服务的 SSL 配置设置与*启用 Spring Boot HTTPS*节中的示例非常相似。唯一的区别是当前使用的 KeyStore 类型，现在是 JKS 而不是 PKCS12。然而，早期示例与服务发现配置之间还有更多区别。首先，我通过将`server.ssl.client-auth`属性设置为`need`来启用了客户端证书认证。这反过来要求我们提供一个`server.ssl.trust-store`属性的 trustStore。以下是`discovery-service`的`application.yml`中的当前 SSL 配置设置：

```java
server: 
 port: ${PORT:8761}
 ssl:
  enabled: true
  client-auth: need
  key-store: classpath:discovery.jks
  key-store-password: 123456
  trust-store: classpath:discovery.jks
  trust-store-password: 123456
  key-alias: discovery
```

如果您使用前面的配置运行 Eureka 应用程序，然后尝试访问其可通过`https://localhost:8761/`访问的网络仪表板，您可能会得到一个错误代码，如`SSL_ERROR_BAD_CERT_ALERT`。出现这个错误是因为您的网络浏览器中没有导入可信证书。为此，我们可以导入一个客户端应用程序的 KeyStore，例如`account-service`的。但首先，我们需要将其从 JKS 格式转换为受网络浏览器支持的另一种格式，例如 PKCS12。以下是`keytool`命令，用于将 KeyStore 从 JKS 格式转换为 PKCS12 格式：

```java
keytool -importkeystore -srckeystore account.jks -srcstoretype JKS -deststoretype PKCS12 -destkeystore account.p12
```

PKCS12 格式被所有主流的网络浏览器支持，比如 Google Chrome 和 Mozilla Firefox。您可以通过导航到设置*|*显示高级设置...|HTTPS/SSL*|*管理证书部分，在 Google Chrome 中导入 PKCS12 KeyStore。如果您再次尝试访问 Eureka 网络仪表板，您应该能够成功认证，并能够看到已注册服务列表。然而，在那里注册的应用程序将不存在。为了在发现客户端和服务器之间提供安全的通信，我们需要为每个微服务创建一个`@Bean`类型的`DiscoveryClientOptionalArgs`，覆盖发现客户端的实现。有趣的是，Eureka 使用 Jersey 作为 REST 客户端。使用`EurekaJerseyClientBuilder`，我们可以轻松地构建一个新的客户端实现，并传递`keystore`和`truststore`文件的路径。以下是从`account-service`中获取的代码片段，我们创建了一个新的`EurekaJerseyClient`对象，并将其设置为`DiscoveryClientOptionalArgs`的参数：

```java
@Bean
public DiscoveryClient.DiscoveryClientOptionalArgs discoveryClientOptionalArgs() throws NoSuchAlgorithmException {
 DiscoveryClient.DiscoveryClientOptionalArgs args = new DiscoveryClient.DiscoveryClientOptionalArgs();
 System.setProperty("javax.net.ssl.keyStore",             
    "src/main/resources/account.jks");
 System.setProperty("javax.net.ssl.keyStorePassword", "123456");
 System.setProperty("javax.net.ssl.trustStore", 
    "src/main/resources/account.jks");
 System.setProperty("javax.net.ssl.trustStorePassword", "123456");
 EurekaJerseyClientBuilder builder = new EurekaJerseyClientBuilder();
 builder.withClientName("account-client");
 builder.withSystemSSLConfiguration();
 builder.withMaxTotalConnections(10);
 builder.withMaxConnectionsPerHost(10);
 args.setEurekaJerseyClient(builder.build());
 return args;
}
```

我们示例系统中的每个微服务都应该提供类似的实现。一个示例应用程序的源代码可以在 GitHub 上找到([`github.com/piomin/sample-spring-cloud-security.git`](https://github.com/piomin/sample-spring-cloud-security.git))。你可以克隆它，并用你的 IDE 运行所有的 Spring Boot 应用程序。如果一切正常，你应该在 Eureka 仪表板上看到与以下屏幕截图相同的注册服务列表。如果 SSL 连接有任何问题，尝试在应用程序启动时设置`-Djava.net.debug=ssl` VM 参数，以能够查看 SSL 握手过程的完整日志：

![](img/450b5cda-04ec-455d-acc3-8fc21cf2d768.png)

# 安全配置服务器

在我们架构中还有一个关键要素需要在讨论安全时考虑——Spring Cloud Config 配置服务器。我觉得保护配置服务器甚至比保护发现服务更为重要。为什么？因为我们通常将它们的认证凭据存储在外部系统上，还有其他一些不应该被未授权访问和使用的数据。有几种方法可以妥善保护你的配置服务器。你可以配置 HTTP 基本认证，安全的 SSL 连接，加密/解密敏感数据，或者使用第三方工具，如在第五章中介绍的，使用 Spring Cloud Config 进行分布式配置。让我们 closer 看看其中的一些。

# 加密和解密

在开始之前，我们必须下载并安装由 Oracle 提供的**Java Cryptography Extension**（**JCE**）。它包括两个 JAR 文件（`local_policy.jar`和`US_export_policy.jar`），需要覆盖 JRE lib/security 目录中现有的策略文件。

如果配置服务器上存储的远程属性源包含加密数据，它们的值应该以`{cipher}`为前缀，并用引号括起来，以表示它是一个 YAML 文件。对于`.properties`文件，不需要用引号括起来。如果无法解密这样的值，它将被替换为同样的键前缀`invalid`的附加值（通常是`<n/a>`）。

在我们上一个示例中，我们在应用程序配置设置中存储了用于保护`keystore`文件的密码短语。将其保存在纯文本文件中可能不是最好的主意，所以它是加密的第一候选。问题是，我们如何加密它？幸运的是，Spring Boot 提供了两个 RESTful 端点可以帮助实现。

让我们看看它是如何工作的。首先，我们需要启动一个配置服务器实例。最简单的方法是激活`--spring.profiles.active=native`配置文件，该配置文件会使用来自本地类路径或文件系统的属性源来启动服务器。现在我们可以调用两个 POST 端点`/encrypt`和`/decrypt`。`/encrypt`方法接受我们的明文密码作为参数。我们可以通过使用逆操作`/decrypt`，它接受一个加密密码作为参数，来检查结果：

```java
$ curl http://localhost:8888/encrypt -d 123456
AQAzI8jv26K3n6ff+iFzQA9DUpWmg79emWu4ndEXyvjYnKFSG7rBmJP0oFTb8RzjZbTwt4ehRiKWqu5qXkH8SAv/8mr2kdwB28kfVvPj/Lb5hdUkH1TVrylcnpZaKaQYBaxlsa0RWAKQDk8MQKRw1nJ5HM4LY9yjda0YQFNYAy0/KRnwUFihiV5xDk5lMOiG4b77AVLmz+9aSAODKLO57wOQUzM1tSA7lO9HyDQW2Hzl1q93uOCaP5VQLCJAjmHcHvhlvM442bU3B29JNjH+2nFS0RhEyUvpUqzo+PBi4RoAKJH9XZ8G7RaTOeWIcJhentKRf0U/EgWIVW21NpsE29BHwf4F2JZiWY2+WqcHuHk367X21vk11AVl9tJk9aUVNRk=
```

加密使用公钥，而解密使用私钥。因此，如果你只进行加密，那么在服务器上只需提供公钥即可。出于测试目的，我们可以使用`keytool`创建 KeyStore。我们之前已经创建了一些 KeyStores，所以在这方面你不会有任何问题。生成的文件应该放在类路径中，然后在`config-service`配置设置中使用`encrypt.keyStore.*`属性：

```java
encrypt:
 keyStore:
  location: classpath:/config.jks
  password: 123456
  alias: config
  secret: 123456
```

现在，如果你将每个微服务的配置设置移动到配置服务器，你可以像下面示例片段中那样加密每个密码：

```java
server: 
 port: ${PORT:8091}
 ssl:
 enabled: true
 key-store: classpath:account.jks
 key-store-password: '{cipher}AQAzI8jv26K3n6ff+iFzQA9DUpWmg79emWu4ndEXyvjYnKFSG7rBmJP0oFTb8RzjZbTwt4ehRiKWqu5qXkH8SAv/8mr2kdwB28kfVvPj/Lb5hdUkH1TVrylcnpZaKaQYBaxlsa0RWAKQDk8MQKRw1nJ5HM4LY9yjda0YQFNYAy0/KRnwUFihiV5xDk5lMOiG4b77AVLmz+9aSAODKLO57wOQUzM1tSA7lO9HyDQW2Hzl1q93uOCaP5VQLCJAjmHcHvhlvM442bU3B29JNjH+2nFS0RhEyUvpUqzo+PBi4RoAKJH9XZ8G7RaTOeWIcJhentKRf0U/EgWIVW21NpsE29BHwf4F2JZiWY2+WqcHuHk367X21vk11AVl9tJk9aUVNRk='
 key-alias: account
```

# 为客户端和服务器配置认证

Spring Cloud Config 服务器的认证实现与 Eureka 服务器的认证实现完全一样。我们可以使用基于标准 Spring 安全机制的 HTTP 基本认证。首先，我们需要确保`spring-security`工件在类路径上。然后我们应该使用`security.basic.

将`enabled`设置为`true`并定义用户名和密码。示例配置设置如下代码片段所示：

```java
security:
 basic:
  enabled: true
 user:
  name: admin
  password: admin123
```

基本认证必须在客户端也启用。这可以通过两种不同的方式实现。第一种是通过配置服务器的 URL：

```java
spring:
 cloud:
  config:
   uri: http://admin:admin123@localhost:8888
```

第二种方法基于独立的`username`和`password`属性：

```java
spring:
 cloud:
  config:
   uri: http://localhost:8888
   username: admin
   password: admin123
```

如果你想设置 SSL 认证，你需要遵循*安全发现*部分描述的步骤。在生成带有私钥和证书的 KeyStores 并设置正确的配置之后，我们可以运行配置服务器。现在，它通过 HTTPS 暴露其 RESTful API。唯一的区别在于客户端的实现。这是因为 Spring Cloud Config 使用的是与 Spring Cloud Netflix Eureka 不同的 HTTP 客户端。正如你可能猜到的，它利用了`RestTemplate`，因为它是完全在 Spring Cloud 项目中创建的。

为了强制客户端应用程序使用双向 SSL 认证而不是标准的、不安全的 HTTP 连接，我们首先应该创建一个实现`PropertySourceLocator`接口的`@Configuration`bean。在那里，我们可以构建一个自定义的`RestTemplate`，它使用一个安全的 HTTP 连接工厂：

```java
@Configuration
public class SSLConfigServiceBootstrapConfiguration {

    @Autowired
    ConfigClientProperties properties;

    @Bean
    public ConfigServicePropertySourceLocator configServicePropertySourceLocator() throws Exception {
        final char[] password = "123456".toCharArray();
        final File keyStoreFile = new File("src/main/resources/discovery.jks");
        SSLContext sslContext = SSLContexts.custom()
                .loadKeyMaterial(keyStoreFile, password, password)
                .loadTrustMaterial(keyStoreFile).build();
        CloseableHttpClient httpClient = HttpClients.custom().setSSLContext(sslContext).build();
        HttpComponentsClientHttpRequestFactory requestFactory = new HttpComponentsClientHttpRequestFactory(httpClient);
        ConfigServicePropertySourceLocator configServicePropertySourceLocator = new ConfigServicePropertySourceLocator(properties);
        configServicePropertySourceLocator.setRestTemplate(new RestTemplate(requestFactory));
        return configServicePropertySourceLocator;
    }

}
```

然而，默认情况下，这个 bean 在应用程序尝试与配置服务器建立连接之前不会被创建。要改变这种行为，我们还应该在`/src/main/resources/META-INF`中创建`spring.factories`文件，并指定自定义的引导配置类：

```java
org.springframework.cloud.bootstrap.BootstrapConfiguration = pl.piomin.services.account.SSLConfigServiceBootstrapConfiguration
```

# 使用 OAuth2 进行授权

我们已经讨论了一些与微服务环境中的认证相关的概念和解决方案。我向您展示了微服务之间以及微服务与服务发现和配置服务器之间的基本和 SSL 认证的例子。在服务间通信中，授权似乎比认证更重要，而认证则实现在系统的边缘。理解认证和授权之间的区别是值得的。简单地说，认证验证你是谁，而授权验证你被授权做什么。

目前最流行的 RESTful HTTP API 授权方法是 OAuth2 和**Java Web Tokens**（**JWT**）。它们可以混合使用，因为它们互补性比其他解决方案要强。Spring 为 OAuth 提供商和消费者提供了支持。借助 Spring Boot 和 Spring Security OAuth2，我们可以快速实现常见的 security patterns，如单点登录、令牌传递或令牌交换。但在我们深入了解这些项目以及其他开发细节之前，我们需要先掌握前面解决方案的基本知识。

# OAuth2 简介

OAuth2 是目前几乎所有主要网站所使用的标准，它允许您通过共享 API 访问他们的资源。它将用户认证委托给一个独立的服务，该服务存储用户凭据并授权第三方应用程序访问关于用户账户的共享信息。OAuth2 用于在保护用户账户凭据的同时给予您的用户访问数据的能力。它为 web、桌面和移动应用程序提供了流程。以下是与 OAuth2 相关的一些基本术语和角色：

+   **资源所有者**：这个角色管理对资源的访问。这种访问受授予授权的范围限制。

+   **授权许可**：它授予访问权限。您可以选择以各种方式确认访问——授权代码、隐式、资源所有者密码凭据和客户端凭据。

+   **资源服务器**：这是一个存储可以使用特殊令牌共享所有者资源的服务器。

+   **授权服务器**：它管理密钥、令牌和其他临时资源访问代码的分配。它还需要确保授予相关用户的访问权限。

+   **访问令牌**：这是一个允许访问资源的钥匙。

为了更好地理解这些术语和实践中的角色，请看下面的图表。它通过 OAuth 协议可视化了一个典型的授权过程流程：

![](img/fb8ff8a6-c870-4ff5-a776-a7f0aedcc84a.png)

让我们回顾一下前面列出个别组件之间交互的进一步步骤。应用程序请求资源所有者的授权，以便能够访问所请求的服务。资源以授权授予作为响应发送，应用程序将其与自身的身份一起发送到授权服务器。授权服务器验证应用程序身份凭据和授权授予，然后发送访问令牌。应用程序使用收到的访问令牌从资源服务器请求资源。最后，如果访问令牌有效，应用程序能够调用请求服务。

# 构建授权服务器

从单体应用程序移动到微服务后，明显的解决方案似乎是通过创建一个授权服务来集中授权努力。使用 Spring Boot 和 Spring Security，你可以轻松地创建、配置和启动一个授权服务器。首先，我们需要将以下`starters`包括到项目依赖中：

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-security</artifactId>
</dependency>
```

使用 Spring Boot 实现授权服务器模式非常简单。我们只需要将主类或配置类注解为`@EnableAuthorizationServer`，然后提供`security.oauth2.client.client-id`和`security.oauth2.client.client-secret`属性在`application.yml`文件中。当然，这个变体尽可能简单，因为它定义了客户端详情服务的内存实现。

一个示例应用程序在同一存储库中，本章之前的示例中([`github.com/piomin/sample-spring-cloud-security.git`](https://github.com/piomin/sample-spring-cloud-security.git))，但在不同的分支，`oauth2` ([`github.com/piomin/sample-spring-cloud-security/tree/oauth2`](https://github.com/piomin/sample-spring-cloud-security/tree/oauth2))。授权服务器在`auth-service`模块下可用。以下是`auth-service`的主类：

```java
@SpringBootApplication
@EnableAuthorizationServer
public class AuthApplication {

    public static void main(String[] args) {
        new SpringApplicationBuilder(AuthApplication.class).web(true).run(args);
    }

}
```

以下是应用程序配置设置的片段。除了客户端的 ID 和密钥外，我还设置了它的默认范围并在整个项目中启用了基本安全：

```java
security:
  user:
    name: root
    password: password
  oauth2:
    client:
      client-id: piotr.minkowski
      client-secret: 123456
      scope: read
```

在我们运行授权服务之后，我们可以进行一些测试。例如，我们可以调用`POST /oauth/token`方法，使用资源所有者密码凭证来创建访问令牌，就像以下命令一样：

```java
$ curl piotr.minkowski:123456@localhost:9999/oauth/token -d grant_type=password -d username=root -d password=password
```

我们还可以通过从你的网络浏览器调用`GET /oauth/authorize`端点来使用授权码授权类型：

```java
http://localhost:9999/oauth/authorize?response_type=token&client_id=piotr.minkowski&redirect_uri=http://example.com&scope=read
```

然后，你将被重定向到批准页面。你可能确认这个动作，最后得到你的访问令牌。它将被发送到初始请求中`redirect_uri`参数传递的回调 URL。以下是我测试后收到的示例响应：

```java
http://example.com/#access_token=dd736a4a-1408-4f3f-b3ca-43dcc05e6df0&token_type=bearer&expires_in=43200.
```

![](img/7e5e41f0-0bd3-4691-aea9-a93b0ecf709a.png)

在`application.yml`文件内提供的相同的 OAuth2 配置也可以以编程方式实现。为了实现这一点，我们应该声明任何实现`AuthorizationServerConfigurer`的`@Beans`。其中的一个是`AuthorizationServerConfigurerAdapter`适配器，它提供空方法，允许您创建以下分离配置器的自定义定义：

+   `ClientDetailsServiceConfigurer`：这定义了客户端详情服务。客户端详情可以初始化，或者你可以简单地引用一个现有的存储。

+   `AuthorizationServerSecurityConfigurer`：这定义了在`/oauth/token_key`和`/oauth/check_token`令牌端点上的安全约束。

+   `AuthorizationServerEndpointsConfigurer`：这定义了授权和令牌端点以及令牌服务。

这种对授权服务器实现的方法为我们提供了更多的机会。例如，我们可以定义一个带有 ID 和密钥的多个客户端，如下面的代码片段所示。我将在本章的下一部分展示一些更高级的示例：

```java
@Configuration
@EnableAuthorizationServer
public class AuthServerConfig extends AuthorizationServerConfigurerAdapter {

    @Override
    public void configure(AuthorizationServerSecurityConfigurer oauthServer) throws     Exception {
      oauthServer
        .tokenKeyAccess("permitAll()")
        .checkTokenAccess("isAuthenticated()");
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
            .withClient("piotr.minkowski").secret("123456")
                .scopes("read")
                .authorities("ROLE_CLIENT")
                .authorizedGrantTypes("authorization_code", "refresh_token", "implicit")
                .autoApprove(true)
            .and()
            .withClient("john.smith").secret("123456")
                .scopes("read", "write")
                .authorities("ROLE_CLIENT")
                .authorizedGrantTypes("authorization_code", "refresh_token", "implicit")
                .autoApprove(true);
    }

}
```

我们必须为我们的授权服务器配置的最后一件事情是网络安全。在扩展了`WebSecurityConfigurerAdapter`的类中，我们定义了一个内存中的用户凭据存储和访问特定资源的权限，例如登录页面：

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter { 

    @Autowired
    private AuthenticationManager authenticationManager; 

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.requestMatchers()
         .antMatchers("/login", "/oauth/authorize")
         .and()
         .authorizeRequests()
         .anyRequest().authenticated()
         .and()
         .formLogin().permitAll();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.parentAuthenticationManager(authenticationManager)
            .inMemoryAuthentication()
            .withUser("piotr.minkowski").password("123456").roles("USERS");
    }

}
```

# 客户端配置

您的应用程序可以使用配置的 OAuth2 客户端以两种不同的方式。这两种方式中的第一种是通过`@EnableOAuth2Client`注解，它创建了一个 ID 为`oauth2ClientContextFilter`的过滤器 bean，负责存储请求和上下文。它还负责管理您应用程序与授权服务器之间的通信。然而，我们将查看 OAuth2 客户端端实现的第二种方法，通过`@EnableOAuth2Sso`。**单点登录**（**SSO**）是一个众所周知的安全模式，允许用户使用一组登录凭据访问多个应用程序。这个注解提供了两个特性——OAuth2 客户端和认证。认证部分使您的应用程序与典型的 Spring Security 机制（如表单登录）对齐。客户端部分具有与`@EnableOAuth2Client`提供的功能相同的功能。因此，我们可以将`@EnableOAuth2Sso`视为比`@EnableOAuth2Client`更高层次的注解。

在下面的示例代码片段中，我用`@EnableOAuth2Sso`注解了扩展了`WebSecurityConfigurerAdapter`的类。得益于这个扩展，Spring Boot 配置了携带 OAuth2 身份处理器的网络安全过滤链。在这种情况下，允许访问`/login`页面，而所有其他请求都需要认证。表单登录页面路径可以通过`security.oauth2.sso.login-path`属性进行覆盖。在覆盖它之后，我们还应该记得在`WebSecurityConfig`内部更改路径模式：

```java
@Configuration
@EnableOAuth2Sso
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.antMatcher("/**")
            .authorizeRequests()
            .antMatchers("/login**")
                .permitAll()
            .anyRequest()
                .authenticated();
    }

}
```

还有一些需要设置的配置设置。首先，我们应该禁用基本认证，因为我们使用了与`@EnableOAuth2Sso`注解一起启用的表单登录方法。然后，我们必须提供一些基本的 OAuth2 客户端属性，例如客户端凭据和授权服务器公开的 HTTP API 端点的地址：

```java
security:
 basic:
  enabled: false
 oauth2:
  client:
   clientId: piotr.minkowski
   clientSecret: 123456
   accessTokenUri: http://localhost:9999/oauth/token
   userAuthorizationUri: http://localhost:9999/oauth/authorize
  resource:
   userInfoUri: http://localhost:9999/user
```

`application.yml`文件片段中的最后一个属性是`security.oauth2.resource.userInfoUri`，这需要在服务器端实现一个额外的端点。`UserController`实现的端点返回`java.security.Principal`对象，表示当前认证的用户：

```java
@RestController
public class UserController {

    @RequestMapping("/user")
    public Principal user(Principal user) {
        return user;
    }

}
```

现在，如果您调用我们微服务中公开的任何端点，您将自动重定向到登录页面。由于我们为内存中的客户端详细信息存储设置了`autoApprove`选项，因此授权授予和访问令牌无需用户任何交互即可自动生成。在登录页面提供您的凭据后，您应该能够获得请求资源的响应。

# 使用 JDBC 后端存储

在前几节中，我们配置了一个认证服务器和客户端应用程序，它授予访问受资源服务器保护的资源的权限。然而，整个授权服务器配置都提供在内存存储中。这种解决方案在开发过程中满足我们的需求，但在生产模式下并不是最理想的方法。目标解决方案应该将所有的认证凭据和令牌存储在数据库中。我们可以选择 Spring 支持的关系数据库之一。在此案例中，我决定使用 MySQL。

所以，第一步是在本地启动 MySQL 数据库。最舒适的方法是使用 Docker 容器。除了启动数据库，下面的命令还将创建一个名为`oauth2`的架构和用户：

```java
docker run -d --name mysql -e MYSQL_DATABASE=oauth2 -e MYSQL_USER=oauth2 -e MYSQL_PASSWORD=oauth2 -e MYSQL_ALLOW_EMPTY_PASSWORD=yes -p 33306:3306 mysql
```

一旦我们启动了 MySQL，现在必须在客户端提供连接设置。如果您在 Windows 机器上运行 Docker，则 MySQL 可在主机地址`192.168.99.100`上访问，端口为`33306`。数据源属性应在`auth-service`的`application.yml`中设置。Spring Boot 还能够在应用程序启动时在选定的数据源上运行一些 SQL 脚本。这对我们来说是个好消息，因为我们必须在为我们的 OAuth2 过程专用的架构上创建一些表：

```java
spring:
 application:
  name: auth-service
 datasource:
  url: jdbc:mysql://192.168.99.100:33306/oauth2?useSSL=false
  username: oauth2
  password: oauth2
  driver-class-name: com.mysql.jdbc.Driver
  schema: classpath:/script/schema.sql
  data: classpath:/script/data.sql
```

创建的架构包含一些用于存储 OAuth2 凭据和令牌的表——`oauth_client_details`、`oauth_client_token`、`oauth_access_token`、`oauth_refresh_token`、`oauth_code`和`oauth_approvals`。包含 SQL 创建命令的完整脚本可在`/src/main/resources/script/schema.sql`中找到。还有一个第二个 SQL 脚本，`/src/main/resources/script/data.sql`，其中有一些用于测试目的的`insert`命令。最重要的是要添加一些客户端 ID/客户端密钥对：

```java
INSERT INTO `oauth_client_details` (`client_id`, `client_secret`, `scope`, `authorized_grant_types`, `access_token_validity`, `additional_information`) VALUES ('piotr.minkowski', '123456', 'read', 'authorization_code,password,refresh_token,implicit', '900', '{}');
INSERT INTO `oauth_client_details` (`client_id`, `client_secret`, `scope`, `authorized_grant_types`, `access_token_validity`, `additional_information`) VALUES ('john.smith', '123456', 'write', 'authorization_code,password,refresh_token,implicit', '900', '{}');
```

当前认证服务器版本与基本示例中描述的版本在实现上有一些不同。这里的第一件重要事情是设置默认令牌存储到数据库，通过提供 `JdbcTokenStore` bean 作为默认数据源的参数。尽管现在所有令牌都存储在数据库中，但我们仍然希望以 JWT 格式生成它们。这就是为什么在类中必须提供第二个 bean，`JwtAccessTokenConverter`。通过重写从基类继承的不同 `configure` 方法，我们可以为 OAuth2 客户端详情设置默认存储，并配置授权服务器始终验证在 HTTP 头中提交的 API 密钥：

```java
@Configuration
@EnableAuthorizationServer
public class OAuth2Config extends AuthorizationServerConfigurerAdapter { 

    @Autowired
    private DataSource dataSource;
    @Autowired
    private AuthenticationManager authenticationManager; 

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.authenticationManager(this.authenticationManager)
            .tokenStore(tokenStore())
            .accessTokenConverter(accessTokenConverter());
 }

    @Override
    public void configure(AuthorizationServerSecurityConfigurer oauthServer) throws Exception {
        oauthServer.checkTokenAccess("permitAll()");
    }

    @Bean
    public JwtAccessTokenConverter accessTokenConverter() {
        return new JwtAccessTokenConverter();
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.jdbc(dataSource);
    } 

    @Bean
    public JdbcTokenStore tokenStore() {
        return new JdbcTokenStore(dataSource);
    }

}
```

Spring 应用程序提供了一个自定义的认证机制。要在应用程序中使用它，我们必须实现 `UserDetailsService` 接口并重写其 `loadUserByUsername` 方法。在我们示例应用程序中，用户凭据和权限也存储在数据库中，因此我们向自定义 `UserDetailsService` 类注入 `UserRepository` bean：

```java
@Component("userDetailsService")
public class UserDetailsServiceImpl implements UserDetailsService { 

    private final Logger log = LoggerFactory.getLogger(UserDetailsServiceImpl.class); 

    @Autowired
    private UserRepository userRepository; 

    @Override
    @Transactional
    public UserDetails loadUserByUsername(final String login) { 
        log.debug("Authenticating {}", login);
        String lowercaseLogin = login.toLowerCase(); 
        User userFromDatabase;
        if(lowercaseLogin.contains("@")) {
            userFromDatabase = userRepository.findByEmail(lowercaseLogin);
        } else {
            userFromDatabase = userRepository.findByUsernameCaseInsensitive(lowercaseLogin);
        } 
        if (userFromDatabase == null) {
            throw new UsernameNotFoundException("User " + lowercaseLogin + " was not found in the database");
        } else if (!userFromDatabase.isActivated()) {
            throw new UserNotActivatedException("User " + lowercaseLogin + " is not activated");
        } 
        Collection<GrantedAuthority> grantedAuthorities = new ArrayList<>();
        for (Authority authority : userFromDatabase.getAuthorities()) {
            GrantedAuthority grantedAuthority = new SimpleGrantedAuthority(authority.getName());
            grantedAuthorities.add(grantedAuthority);
        } 
        return new org.springframework.security.core.userdetails.User(userFromDatabase.getUsername(), userFromDatabase.getPassword(), grantedAuthorities);
 }

}
```

# 服务间授权

我们示例中的服务间通信是使用 Feign 客户端实现的。以下是所选实现之一——在这种情况下，来自 `order-service` ——它调用 `customer-service` 的端点：

```java
@FeignClient(name = "customer-service")
public interface CustomerClient {

    @GetMapping("/withAccounts/{customerId}")
    Customer findByIdWithAccounts(@PathVariable("customerId") Long customerId);

}
```

与其它服务一样，`customer-service` 中所有可用方法都基于 OAuth 令牌作用域的保护预授权机制。它允许我们用 `@PreAuthorize` 注解标记每个方法，定义所需的作用域：

```java
@PreAuthorize("#oauth2.hasScope('write')")
@PutMapping
public Customer update(@RequestBody Customer customer) {
    return repository.update(customer);
}

@PreAuthorize("#oauth2.hasScope('read')")
@GetMapping("/withAccounts/{id}")
public Customer findByIdWithAccounts(@PathVariable("id") Long id) throws JsonProcessingException {
    List<Account> accounts = accountClient.findByCustomer(id);
    LOGGER.info("Accounts found: {}", mapper.writeValueAsString(accounts));
    Customer c = repository.findById(id);
    c.setAccounts(accounts);
    return c;
}
```

预授权默认是禁用的。要为 API 方法启用它，我们应该使用 `@EnableGlobalMethodSecurity` 注解。我们还应指示这种预授权将基于 OAuth2 令牌作用域：

```java
@Configuration
@EnableResourceServer
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class OAuth2ResourceServerConfig extends GlobalMethodSecurityConfiguration {

    @Override
    protected MethodSecurityExpressionHandler createExpressionHandler() {
        return new OAuth2MethodSecurityExpressionHandler();
    }

}
```

如果您通过 Feign 客户端调用账户服务端点，将会得到以下异常：

```java
feign.FeignException: status 401 reading CustomerClient#findByIdWithAccounts(); content:{"error":"unauthorized","error_description":"Full authentication is required to access this resource"}
```

为什么会出现这样的异常呢？当然，`customer-service` 是通过 OAuth2 令牌授权进行保护的，但是 Feign 客户端在请求头中没有发送授权令牌。解决这个问题的一种方法是为 Feign 客户端定义一个自定义配置类。它允许我们声明一个请求拦截器。在这种情况下，我们可以使用 Spring Cloud OAuth2 库中提供的 `OAuth2FeignRequestInterceptor` 实现的 OAuth2。出于测试目的，我决定使用资源所有者密码授权类型：

```java
public class CustomerClientConfiguration {

    @Value("${security.oauth2.client.access-token-uri}")
    private String accessTokenUri;
    @Value("${security.oauth2.client.client-id}")
    private String clientId;
    @Value("${security.oauth2.client.client-secret}")
    private String clientSecret;
    @Value("${security.oauth2.client.scope}")
    private String scope;

    @Bean
    RequestInterceptor oauth2FeignRequestInterceptor() {
        return new OAuth2FeignRequestInterceptor(new DefaultOAuth2ClientContext(), resource());
    }

    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }

    private OAuth2ProtectedResourceDetails resource() {
        ResourceOwnerPasswordResourceDetails resourceDetails = new ResourceOwnerPasswordResourceDetails();
        resourceDetails.setUsername("root");
        resourceDetails.setPassword("password");
        resourceDetails.setAccessTokenUri(accessTokenUri);
        resourceDetails.setClientId(clientId);
        resourceDetails.setClientSecret(clientSecret);
        resourceDetails.setGrantType("password");
        resourceDetails.setScope(Arrays.asList(scope));
        return resourceDetails;
    }

}
```

最后，我们可以测试所实现的解决方案。这次，我们将创建一个 JUnit 自动化测试，而不是在网页浏览器中点击它或使用其他工具发送请求。以下代码片段显示了测试方法。我们使用`OAuth2RestTemplate`和`ResourceOwnerPasswordResourceDetails`执行资源所有者凭据授予操作，并调用来自`order-service`的`POST /` API 方法，请求头中发送了 OAuth2 令牌。当然，在运行那个测试之前，您必须启动所有微服务以及发现和授权服务器：

```java
@Test
public void testClient() {
    ResourceOwnerPasswordResourceDetails resourceDetails = new ResourceOwnerPasswordResourceDetails();
    resourceDetails.setUsername("root");
    resourceDetails.setPassword("password");
    resourceDetails.setAccessTokenUri("http://localhost:9999/oauth/token");
    resourceDetails.setClientId("piotr.minkowski");
    resourceDetails.setClientSecret("123456");
    resourceDetails.setGrantType("password");
    resourceDetails.setScope(Arrays.asList("read"));
    DefaultOAuth2ClientContext clientContext = new DefaultOAuth2ClientContext();
    OAuth2RestTemplate restTemplate = new OAuth2RestTemplate(resourceDetails, clientContext);
    restTemplate.setMessageConverters(Arrays.asList(new MappingJackson2HttpMessageConverter()));
    Random r = new Random();
    Order order = new Order();
    order.setCustomerId((long) r.nextInt(3) + 1);
    order.setProductIds(Arrays.asList(new Long[] { (long) r.nextInt(10) + 1, (long) r.nextInt(10) + 1 }));
    order = restTemplate.postForObject("http://localhost:8090", order, Order.class);
    if (order.getStatus() != OrderStatus.REJECTED) {
        restTemplate.put("http://localhost:8090/{id}", null, order.getId());
    }
}
```

# 在 API 网关上启用单点登录

您可以通过在主类上添加`@EnableOAuth2Sso`注解来仅通过注解在 API 网关上启用单点登录功能。确实，这是强制 Zuul 生成或获取当前认证用户的访问令牌的最佳选择，对于您的微服务架构来说：

```java
@SpringBootApplication
@EnableOAuth2Sso
@EnableZuulProxy
public class GatewayApplication {

    public static void main(String[] args) {
        new SpringApplicationBuilder(GatewayApplication.class).web(true).run(args);
    }

}
```

通过包含`@EnableOAuth2Sso`，你可以触发一个对 ZuulFilter 可用的自动配置。这个过滤器负责从当前已认证的用户中提取访问令牌，然后将其放入转发到微服务网关后面的请求头中。如果对这些服务启用了`@EnableResourceServer`，它们将会在`Authorization` HTTP 头中收到预期的令牌。`@EnableZuulProxy`下游的授权行为可以通过声明`proxy.auth.*`属性来控制。

当在您的架构中使用网关时，您可能在其后面隐藏一个授权服务器。在这种情况下，您应该在 Zuul 的配置设置中提供额外的路由—例如，`uaa`。然后，OAuth2 客户端与服务器之间交换的所有消息都通过网关。这是在网关的`application.yml`文件中的正确配置：

```java
security:
  oauth2:
    client:
      accessTokenUri: /uaa/oauth/token
      userAuthorizationUri: /uaa/oauth/authorize
      clientId: piotr.minkowski
      clientSecret: 123456
    resource:
      userInfoUri: http://localhost:9999/user

zuul:
  routes:
    account-service:
      path: /account/**
    customer-service:
      path: /customer/**
    order-service:
      path: /order/**
    product-service:
      path: /product/**
    uaa:
      sensitiveHeaders:
      path: /uaa/**
      url: http://localhost:9999
  add-proxy-headers: true
```

# 摘要

如果在本书的第二部分每个章节中都包含一个安全部分，那也不会有什么问题。但我决定专门用一章来介绍这个主题，以便向您展示如何逐步为基于微服务架构的关键元素提供安全保护的步骤。与安全相关的主题通常比其他主题更高级，所以我花了一些时间来解释该领域的一些基本概念。我向您展示了示例，说明了双向 SSL 认证、敏感数据的加密/解密、Spring Security 认证以及使用 JWT 令牌的 OAuth2 授权。您需要决定在您的系统架构中使用哪个来提供您所需的安全级别。

阅读本章后，你应该能够为你的应用程序设置基本和更高级的安全配置。你还应该能够保护你系统架构中的每一个组件。当然，我们只讨论了一些可能的解决方案和框架。例如，你不必仅依赖于 Spring 作为授权服务器提供者。我们可以使用第三方工具，如 Keycloak，它可以在基于微服务的系统中作为授权和认证服务器。它还可以轻松地与 Spring Boot 应用程序集成。它支持所有最流行的协议，如 OAuth2、OpenId Connect 和 SAML。因此，实际上，Keycloak 是一个非常强大的工具，应该被视为 Spring 授权服务器的替代品，特别是在大型企业系统和其他更高级的使用场景中。

在下一章中，我们将讨论微服务测试的不同策略。
