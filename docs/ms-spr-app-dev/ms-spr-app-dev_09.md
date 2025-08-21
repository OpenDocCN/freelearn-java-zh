# 第九章：使用 Spring Boot 引导应用程序

在本章中，我们将看到另一个 Spring 包——Spring Boot，它允许用户快速开始使用 Spring 框架。使用**Spring Boot 抽象层**的应用程序称为**Spring Boot 应用程序**。Spring 推出了一个 Spring 初始化器 Web 应用程序，其中有一个 Web 界面，我们可以在其中选择需要启动的应用程序类型。

如果您曾经在不同的应用服务器上运行过，新开发人员通常必须配置许多设置才能启动和运行。Spring Boot 方法允许开发人员立即启动和运行，而无需配置应用服务器，从而可以专注于开发代码。

Spring 还推出了一个命令行界面，帮助我们快速开始 Spring 开发。在本章中，让我们深入了解 Spring Boot 并看看它提供了什么。

# 设置 Spring Boot

Spring Boot 应用程序可以通过以下方式设置：

+   使用[`start.spring.io/`](http://start.spring.io/)

+   使用 Maven 从存储库下载依赖项

+   使用 Gradle

+   从 Spring 指南存储库下载源代码

+   下载 Spring STS 并使用启动器项目

# Spring Gradle MVC 应用程序

**Gradle**类似于 Maven；它有助于构建应用程序。我们需要在`build.gradle`文件中提供所有依赖信息。Spring Boot 还有一个 Gradle 插件。Gradle 插件有助于将所有依赖的 JAR 文件放置在类路径上，并最终构建成一个可运行的单个 JAR 文件。可运行的 JAR 文件将具有一个`application.java`文件；这个类将有一个`public static void main()`方法。这个类将被标记为可运行的类。

这里显示了一个示例 Gradle 文件：

```java
buildscript {
  repositories {
    maven { url "http://repo.spring.io/libs-milestone" }
    mavenLocal()
  }
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:1.1.3.RELEASE")
  }
}

  apply plugin: 'java'
  apply plugin: 'war'
  apply plugin: 'spring-boot'
  jar {
    baseName = PacktSpringBootMVCDemo '
    version =  '1.0'
  }
  repositories {
    mavenCentral()
    maven { url "http://repo.spring.io/libs-milestone" }
  }

  configurations {
    providedRuntime
  }
  dependencies {
    compile ("org.springframework.boot:spring-boot-starter-web")
    providedRuntime("org.apache.tomcat.embed:tomcat-embed-jasper")

  }
  task wrapper(type: Wrapper) {
    gradleVersion = '2.0'
  }
```

如果您正在使用 Eclipse 作为 IDE，STS 已经推出了适用于 Eclipse 的 Gradle 插件（[`gradle.org/docs/current/userguide/eclipse_plugin.html`](http://gradle.org/docs/current/userguide/eclipse_plugin.html)），可以从[`www.gradle.org/tooling`](https://www.gradle.org/tooling)下载并安装。Gradle 也提供了类似的设置来清理和构建应用程序。

下一步是在属性文件中定义应用程序上下文根。Gradle 项目结构类似于 Maven 项目结构。将`application.properties`文件放在`resources`文件夹中。我们需要提供服务器上下文路径和服务器上下文端口。以下是示例属性文件：

```java
server.contextPath=/PacktSpringBootMVCDemo
server.port=8080
```

1.  让我们创建一个简单的包：`com.packt.controller`

1.  在包中创建一个简单的 Spring 控制器类，并使用@ Controller 注解。

1.  让我们创建一个带有`@Request`映射注解的方法。`@RequestMapping`注解将请求映射到 JSP 页面。在这个方法中，我们将请求映射到方法。这些方法返回一个字符串变量或模型视图对象。

```java
package com.packt.controller;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.servlet.ModelAndView;
@Controller
public class PacktController{
  @RequestMapping(value = "/saygoodmorning  method = RequestMethod.GET)
  public ModelAndView getGoodmorning() {
    return new ModelAndView("greet").addObject("greet", "goodmorning");
  }
  @RequestMapping(value = "/saygoodafternoon  method = RequestMethod.GET)
  public ModelAndView getGoodmorning() {
    return new ModelAndView("greet").addObject("greet ", "goodafternoon");
  }
  @RequestMapping(value = "/saygoodnight  method = RequestMethod.GET)
  public ModelAndView getGoodmorning() {
    return new ModelAndView("greet").addObject("greet ", "goodnight");
  }
}
```

1.  创建一个 Spring MVC 配置文件，使用`@Configuration`和`@WebMVC` `annotation`如下。我们还为应用程序文件配置了内部视图解析器。

```java
package com.packt.config;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

@Configuration
@EnableWebMvc
public class ApplicationConfigurerAdapter extends WebMvcConfigurerAdapter{
  @Override
  public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
    configurer.enable();
  }

  @Bean
  public InternalResourceViewResolver viewResolver() {
    InternalResourceViewResolver resolver = new InternalResourceViewResolver();
    resolver.setPrefix("WEB-INF/jsp/");
    resolver.setSuffix(".jsp");
    return resolver;
  }

}
```

让我们创建一个名为`greet.jsp`的简单 JSP 页面：

```java
<html>
  <head><title>Hello world Example</title></head>
  <body>
    <h1>Hello ${name}, How are you?</h1>
  </body>
</html>
```

接下来创建一个简单的应用程序类，使用`@EnableAutoConfiguration`和`@ComponentScan`注解。`@ComponenetScan`注解表示 Spring 框架核心应搜索包下的所有类。`@EnableAutoConfiguration`注解用于代替在`web.xml`文件中配置 dispatcher servlet。

以下是示例文件：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableAutoConfiguration
@ComponentScan
public class Application {
  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }

}
```

访问以下 URL：

+   `http://localhost:8080/PacktSpringBootMVCDemo/saygoodmorning`

+   `http://localhost:8080/PacktSpringBootMVCDemo/saygoodafternoon`

+   `http://localhost:8080/PacktSpringBootMVCDemo/saygoodnight`

## 使用 Spring Boot 进行热交换

热交换或热部署意味着您可以对类文件或应用程序中的任何文件进行更改，并立即在运行中的应用程序中看到更改。我们可能需要重新加载 Web 浏览器上的应用程序，或者只需刷新页面。Spring Loaded 是一个支持热部署的依赖 JAR 文件。让我们看看 Spring Boot 应用程序中的热交换。

让我们使用 Thymeleaf 模板引擎创建一个简单的 Spring MVC 应用程序：

1.  首先，我们需要从 GitHub 存储库下载 Spring Loaded JAR。检查以下 URL 以获取最新版本：

[`github.com/spring-projects/spring-loaded`](https://github.com/spring-projects/spring-loaded)。

1.  确保您在`pom.xml`文件中具有所有提到的依赖项，或者将它们明确添加到您的项目中：

```java
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

1.  下一步是将下载的 Spring loaded JAR 添加到 Eclipse 或 Eclipse STS 环境中。按照给定的步骤将 Spring loaded JAR 添加为运行时配置：

1.  在 Eclipse 中创建一个`PacktSpringBootThymeLeafExample`项目。

1.  右键单击您的项目。

1.  搜索**Run As**。

1.  点击**Run Configuration**。

1.  点击 Java 应用程序。

1.  点击项目名称。

1.  在**VM Argument**部分中选择**Arguments**；添加以下命令：

```java
- javaagent:/<provide the path to the jar>/springloaded-1.2.0.RELEASE.jar -noverify
```

1.  点击**Apply**和**Run**。

我们还需要配置`application.properties`文件，以便对**Thymeleaf**页面进行任何修改时不需要重新启动服务器：

```java
spring.thymeleaf.cache: false.
```

我们可以使用 Spring STS starter 项目并创建一个 Spring Boot 类。Spring Eclipse STS 将为我们提供以下两个类：

+   `Application.java`

+   `ApplicationTest.java`

`Application.java`是 Spring Boot 的主类，因为它在其中有 public static void main 方法。在这个方法中，使用`SpringApplication`类对`ApplicationContext`进行初始化。`SpringApplication`类具有以下一些注解：

+   `@ConditionalOnClass`

+   `@ConditionalOnMissingBean`

这些用于检查类路径上可用的 bean 列表。如果您想要查看框架放置在类路径下的 bean，可以对生成的`Application.java`文件进行轻微修改，如下所示：

```java
@ComponentScan
@EnableAutoConfiguration
public class Application {
  public static void main(String[] args) {
    ApplicationContext ctx = SpringApplication.run(Application.class, args);
    System.out.println("---------------------------LIST BEANS PROVIDED BY SPRING BOOT_---------------------");
    String[] beanNames = ctx.getBeanDefinitionNames();
    Arrays.sort(beanNames);
    for (String beanName : beanNames) {
      System.out.println(beanName);
    }

  }
}
```

输出：

```java
---------------------------LIST BEANS PROVIDED BY SPRING BOOT_---------------------
JSPController
application
applicationContextIdFilter
auditEventRepository
auditListener
autoConfigurationAuditEndpoint
basicErrorController
beanNameHandlerMapping
beanNameViewResolver
....
mappingJackson2HttpMessageConverter
messageConverters
messageSource
tomcatEmbeddedServletContainerFactory
traceEndpoint
traceRepository
viewControllerHandlerMapping
viewResolver
webRequestLoggingFilter
```

`SpringApplication`类位于`org.springframework.boot.SpringApplication`包中。

这里显示了`SpringApplication`类的一个简单示例，其中显示了`SpringApplication`类的静态运行方法：

```java
@Configuration
@EnableAutoConfiguration
public class MyPacktApplication {

  // ... Bean definitions

  public static void main(String[] args) throws Exception {
    SpringApplication.run(MyPacktApplication.class, args);
  }
```

在这里看另一个例子，首先初始化一个`SpringApplication`类，然后调用`.run`方法：

```java
@Configuration
@EnableAutoConfiguration
public class MyPacktApplication {
  // ... Bean definitions
  public static void main(String[] args) throws Exception {
    SpringApplication app = new SpringApplication(MyPacktApplication.class);
    // ... customize app settings here
    app.run(args)
  }
}
```

以下是`SpringApplication`类可用的构造函数：

+   `SpringApplication(Object... sources)`

+   `SpringApplication(ResourceLoader resourceLoader, Object... sources)`

1.  让我们创建一个带有 Spring 最新版本 4.x 中可用的`@RestController`注解的简单 Controller 类。

```java
@RestController
public class MyPacktController {

  @RequestMapping("/")
  public String index() {
    return "Greetings ";
  }

  @RequestMapping("/greetjsontest") 
  public @ResponseBody Map<String, String> callSomething () {

    Map<String, String> map = new HashMap<String, String>();
    map.put("afternoon", " Good afternoon");
    map.put("morning", " Good Morning");
    map.put("night", " Good Night");
    return map;
  }
}
```

1.  接下来，我们将配置 Spring Boot 来处理 JSP 页面；默认情况下，Spring Boot 不配置 JSP，因此我们将创建一个 JSP 控制器，如下面的代码片段所示：

```java
@Controller
public class SpringBootJSPController {
  @RequestMapping("/calljsp")
  public String test(ModelAndView modelAndView) {

    return "myjsp";
  }
}
```

1.  按照以下方式配置属性文件：

```java
spring.view.prefix: /WEB-INF/jsp/
spring.view.suffix: .jsp
```

1.  让我们创建一个 JSP 文件`myjsp:`

```java
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
  </head>
  <body>
    <h1>Hello world</h1>
  </body>
</html>
```

以下是`EmbededServletContainerCustomizer`的实现类，它实际上将 Web 服务器容器嵌入应用程序中。它调用服务器并将应用程序部署到其中。

```java
@ComponentScan
@EnableAutoConfiguration

public class Application implements EmbeddedServletContainerCustomizer {
  @Value("${someproperty:webapp/whereever }")
  private String documentRoot;
  @Override
  public void customize(ConfigurableEmbeddedServletContainerFactory factory) {
    factory.setDocumentRoot(new File(documentRoot));
  }
}
```

## 将 Spring Boot 与 Spring 安全集成

在本节中，我们将看到如何使用注解将 Spring Boot 与 Spring 安全集成。我们可以很容易地将 Spring 安全与 Spring Boot 集成。

1.  让我们首先在 Spring boot 中嵌入一个 tomcat 服务器来接受请求；以下是我们需要创建一个密钥库文件使其更安全的代码：

```java
@Bean
EmbeddedServletContainerCustomizer containerCustomizer (
  @Value("${https.port}") final int port, 
  @Value("${keystore.file}") Resource keystoreFile,
  @Value("${keystore.alias}") final String alias, 
  @Value("${keystore.password}") final String keystorePass,
  @Value("${keystore.type}") final String keystoreType) throws Exception {
    final String absoluteKeystoreFile = keystoreFile.getFile().getAbsolutePath();
    return new EmbeddedServletContainerCustomizer() {
      public void customize(ConfigurableEmbeddedServletContainer container) {
        TomcatEmbeddedServletContainerFactory tomcat = (TomcatEmbeddedServletContainerFactory) container;
        tomcat.addConnectorCustomizers(new TomcatConnectorCustomizer() {
          public void customize(Connector connector) {
            connector.setPort(port);
            connector.setSecure(true);
            connector.setScheme("https");
            Http11NioProtocol proto = (Http11NioProtocol) connector.getProtocolHandler();
            proto.setSSLEnabled(true);
            proto.setKeystoreFile(absoluteKeystoreFile);
            proto.setKeyAlias(alias);
            proto.setKeystorePass(keystorePass);
            proto.setKeystoreType(keystoreType);
          }
        });
      }
    };
  }
```

1.  让我们还使用`@Configuration`和`@EnableWebMVCSecurity`注解在 java 中创建一个简单的安全配置文件。安全配置文件扩展了`WebSecurityConfigurerAdapter`。

```java
@Configuration
@EnableWebMvcSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

  @Value("${ldap.domain}")
  private String DOMAIN;

  @Value("${ldap.url}")
  private String URL;

  @Value("${http.port}")
  private int httpPort;

  @Value("${https.port}")
  private int httpsPort;

  @Override
  protected void configure(HttpSecurity http) throws Exception {
    /*
    * Set up your spring security config here. For example...
    */
    http.authorizeRequests().anyRequest().authenticated().and().formLogin().loginUrl("/login").permitAll();
      /*
      * Use HTTPs for ALL requests
      */
      http.requiresChannel().anyRequest().requiresSecure();
      http.portMapper().http(httpPort).mapsTo(httpsPort);
    }

    @Override
    protected void configure(AuthenticationManagerBuilder authManagerBuilder) throws Exception {
    authManagerBuilder.authenticationProvider(activeDirectoryLdapAuthenticationProvider()).userDetailsService(userDetailsService());
    }

    @Bean
    public AuthenticationManager authenticationManager() {
      return new ProviderManager(Arrays.asList(activeDirectoryLdapAuthenticationProvider()));
    }
    @Bean
    public AuthenticationProvider activeDirectoryLdapAuthenticationProvider() {
      ActiveDirectoryLdapAuthenticationProvider provider = new ActiveDirectoryLdapAuthenticationProvider(DOMAIN, URL);
        provider.setConvertSubErrorCodesToExceptions(true);
        provider.setUseAuthenticationRequestCredentials(true);
        return provider;
    }
  }
```

# Eclipse Spring Boot 的 Cloud Foundry 支持

在本节中，让我们看看如何使用 Spring boot 在 Cloud Foundry 上开发应用程序。**Cloud Foundry**是一个用作服务云应用程序的平台。它是一个开放的 PaaS。PaaS 使得在云上运行、部署和运行应用程序成为可能。

参考以下链接，其中提供了有关作为服务可用的 Spring 平台的完整信息以及如何配置 Spring 以与 Cloud Foundry 一起工作。您将看到它提供了从 MongoDB 到 RabbitMQ 消息服务器的平台作为服务。

[`docs.cloudfoundry.org/buildpacks/java/spring-service-bindings.html`](http://docs.cloudfoundry.org/buildpacks/java/spring-service-bindings.html)

Eclipse 还推出了一个针对云平台的插件，可以从以下给定位置下载和安装。该插件支持 Spring boot 和 grails 应用程序。您还可以创建一个服务器实例到使用自签名证书的私有云。

[`github.com/cloudfoundry/eclipse-integration-cloudfoundry`](https://github.com/cloudfoundry/eclipse-integration-cloudfoundry)

我们所需要做的就是开发一个简单的启动应用程序，并将其拖放到 Cloud Foundry 服务器中，然后重新启动服务器。

# 使用 Spring Boot 开发 RestfulWebService

在本节中，让我们开发一个简单的 restful 服务，并使用`SpringBoot`引导应用程序。我们还将创建一个简单的 restful 服务，将产品信息存储到数据库中。

产品创建场景应满足以下提到的用例：

+   假设不存在具有相同`Product_id`的产品，它应该将新产品存储在数据库中并立即返回存储的对象。

+   假设存在一个具有相同`Product_id`的产品，它不应该存储，而是返回一个带有相关消息的错误状态。

+   假设以前存储的产品，它应该能够检索它们的列表。

以下是`pom.xml`文件，用于应用程序中使用的依赖项引用。您可以看到我们在这里使用了父 Spring boot 引用，以便我们可以解析所有依赖项引用。我们还在`pom.xml`文件中设置了 Java 版本为 1.7。

```java
<project  
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.packt.restfulApp</groupId>
  <artifactId>restfulSpringBootApp</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.0.1.RELEASE</version>
  </parent>

  <name>Example Spring Boot REST Service</name>

  <properties>
    <java.version>1.7</java.version>
    <guava.version>16.0.1</guava.version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
  </properties>

</project>
```

让我们看看`pom.xml`文件中使用的依赖项。以下是使用的 Spring boot 依赖项。还要注意，版本信息没有指定，因为它由前面提到的`spring-boot-starter-parent`管理。

```java
<dependencies>
  <!-- Spring Boot -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
  </dependency>

  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>

  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>

  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>

  <!-- Hibernate validator -->

  <dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
  </dependency>

  <!-- HSQLDB -->

  <dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <scope>runtime</scope>
  </dependency>

  <!-- Guava -->

  <dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>${guava.version}</version>
  </dependency>

  <!-- Java EE -->

  <dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
  </dependency>
</dependencies>
```

我们还将看到为什么这些依赖项被用于 Spring boot。当涉及到 Spring boot 时，它的功能分布在 starter 模块之间：

+   `spring-boot-starter`：这是 Spring boot 的主要核心模块

+   `spring-boot-starter-test`：这里有一些用于单元测试的工具，包括 JUnit4 和 Mockito

+   `spring-boot-starter-web`：这会拉取 Spring MVC 依赖项，还有将用于 JSON 的 Jackson，最重要的是 Tomcat，它充当嵌入式 Servlet 容器

+   `spring-boot-starter-data-jpa`：用于设置 Spring Data JPA，并与 Hibernate 捆绑在一起

+   `Guava`：它使用`@Inject`注释而不是`@Autowired`

最后，添加一个 Spring boot Maven 插件如下。`spring-boot-maven`插件的功能如下：

+   它为 Maven 提供了一个`spring-boot:run`目标，因此应用程序可以在不打包的情况下轻松运行。

+   它钩入一个打包目标，以生成一个包含所有依赖项的可执行 JAR 文件，类似于`maven-shade`插件，但方式不那么混乱。

```java
<build>
  <plugins>

  <!-- Spring Boot Maven -->

    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>

  </plugins>
</build>
```

到目前为止，我们已经看了依赖项及其功能，现在让我们开始构建应用程序。

**Bean 类或实体类**：

1.  让我们创建一个简单的`Product.java`文件如下：

```java
@Entity
public class Product {
  @Id
  @Column(name = "id", nullable = false, updatable = false)
  @NotNull 
  private Long product_id;
  @Column(name = "password", nullable = false)
  @NotNull
  @Size(max = 64)
  private String product_name;

  public Action(Long product_id, String product_name) {

    this. produc_id = product_id;
    this. produc_name = produc_name;
  }
```

1.  接下来创建一个`Jparepository`子接口；我们不需要为此提供任何实现，因为它由 Spring JPA 数据处理：

```java
public interface ProductRepository extends JpaRepository<Product, String>{

}
```

**服务类**：

1.  让我们创建一个处理保存的服务接口。

```java
public interface ProductService {

  Product save(Product product);

}
```

1.  我们还应该为服务接口创建一个实现类：

```java
@Service
public class ProductServiceImpl implements ProductService {

  private final ProductRepository repository;

  @Inject
  public ProductServiceImpl(final ProductRepository repository) {
    this.repository = repository;
  }

  @Override
  @Transactional
  public Product save(final Product product) {
    Product existing = repository.findOne(Product.getId());
    if (existing != null) {
      throw new ProductAlreadyExistsException(
        String.format("There already exists a Product with id=%s", product.getId()));
    }
    return repository.save(product);
  }
```

1.  在下一步中，我们还将创建一个用于服务`Impl`的测试类，如下所示：

```java
@RunWith(MockitoJUnitRunner.class)
public class ProductControllerTest {

  @Mock
  private ProductService ProductService;

  private ProductController ProductController;

  @Before
  public void setUp() {
    ProductController = new ProductController(ProductService);
  }

  @Test
  public void shouldCreateProduct() throws Exception {
    final Product savedProduct = stubServiceToReturnStoredProduct();
    final Product Product = new Product();
    Product returnedProduct = ProductController.createProduct(Product);
    // verify Product was passed to ProductService
    verify(ProductService, times(1)).save(Product);
    assertEquals("Returned Product should come from the service", savedProduct, returnedProduct);
  }

  private Product stubServiceToReturnStoredProduct() {
    final Product Product = new Product();
    when(ProductService.save(any(Product.class))).thenReturn(Product);
    return Product;
  }
```

1.  让我们使用`@RestController`注解创建一个控制器；还要注意我们使用了`@Inject`注解。

+   `@RestController`：这与`@Controller`注解的区别在于前者在每个方法上也意味着`@ResponseBody`，这意味着写的内容更少，因为从 restful web 服务中我们无论如何都会返回 JSON 对象。

+   `@RequestMapping`：这将`createProduct()`映射到`/Product` URL 上的`POST`请求。该方法将产品对象作为参数。它是从请求的主体中创建的，这要归功于`@RequestBody`注解。然后进行验证，这是由`@Valid`强制执行的。

+   `@Inject`：`ProductService`将被注入到构造函数中，并且产品对象将被传递给其`save()`方法进行存储。存储后，存储的产品对象将被自动转换回 JSON，即使没有`@ResponseBody`注解，这是`@RestController`的默认值。

```java
@RestsController
public class ProductController {
  private final ProductService ProductService;
  @Inject
  public ProductController(final ProductService ProductService) {
    this.ProductService = ProductService;
  }
  @RequestMapping(value = "/Product", method = RequestMethod.POST)
  public Product createProduct(@RequestBody @Valid final Product Product) {
    return ProductService.save(Product);
  }
}
```

1.  让我们创建一个带有`public static void main()`的`Main`类。我们还可以使用这些注解：

+   `@Configuration` - 这告诉 Spring 框架这是一个配置类

+   `@ComponentScan` - 这使得可以扫描包和子包以寻找 Spring 组件

+   `@EnableAutoConfiguration`

该类进一步扩展了`SpringBootServletInitializer`，它将为我们配置调度程序 servlet 并覆盖`configure`方法。

以下是`Main`类：

```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
public class Application extends SpringBootServletInitializer {

  public static void main(final String[] args) {
    SpringApplication.run(Application.class, args);
  }

  @Override
  protected final SpringApplicationBuilder configure(final SpringApplicationBuilder application) {
    return application.sources(Application.class);
  }
}
```

1.  现在，让我们使用 Maven 和 Bootstrap 运行应用程序：

```java
mvn package
java -jar target/restfulSpringBootApp.jar
```

现在，您可以做到这一点：

```java
curl -X POST -d '{ "id": "45", "password": "samsung" }' http://localhost:8080/Product
```

并查看 http://localhost:8080/的响应是否如下：

```java
{ "id": "45", "password": "samsung" }
```

# 总结

在本章中，我们演示了使用 Spring Boot 启动应用程序的过程。我们从设置一个简单的 Spring Boot 项目开始。我们还创建了一个带有 Gradle 支持的简单 MVC 应用程序。接下来，我们讨论了使用 Spring Boot 进行热交换 Java 文件的方法。

我们还提供了关于 Spring Boot 如何支持云平台服务器并帮助在云上部署应用程序的信息。最后，我们演示了一个使用 Spring Boot 的 restful 应用程序。

在下一章中，我们将讨论 Spring 缓存。
