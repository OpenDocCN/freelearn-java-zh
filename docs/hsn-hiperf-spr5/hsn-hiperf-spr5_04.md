# 第四章：Spring MVC 优化

在上一章中，我们学习了 Spring **面向切面编程**（**AOP**）模块，AOP 概念，其各种术语，以及如何实现建议。我们还了解了代理概念及其使用代理模式的实现。我们通过最佳实践来实现 Spring AOP 的质量和性能。

Spring MVC 现在是最流行的 Java Web 应用程序框架。它由 Spring 自身提供。Spring Web MVC 有助于开发灵活和松散耦合的基于 Web 的应用程序。Spring MVC 遵循**模型-视图-控制器**（**MVC**）模式，它将输入逻辑、业务逻辑和表示逻辑分开，同时提供组件之间的松散耦合。Spring MVC 模块允许我们在 Web 应用程序中编写测试用例而不使用请求和响应对象。因此，它消除了在企业应用程序中测试 Web 组件的开销。Spring MVC 还支持多种新的视图技术，并允许扩展。Spring MVC 为控制器、视图解析器、处理程序映射和 POJO bean 提供了清晰的角色定义，使得创建 Java Web 应用程序变得简单。

在本章中，我们将学习以下主题：

+   Spring MVC 配置

+   Spring 异步处理，`@Async`注解

+   使用 Spring Async 的`CompletableFuture`

+   Spring 安全配置

+   认证缓存

+   使用 Spring Security 进行快速和无状态的 API 身份验证

+   使用 JMX 监视和管理 Tomcat

+   Spring MVC 性能改进

# Spring MVC 配置

Spring MVC 架构设计了一个前端控制器 Servlet，即`DispatcherServlet`，它是前端控制器模式的实现，并充当所有 HTTP 请求和响应的入口点。`DispatcherServlet`可以使用 Java 配置或部署描述符文件`web.xml`进行配置和映射。在进入配置部分之前，让我们了解 Spring MVC 架构的流程。

# Spring MVC 架构

在 Spring MVC 框架中，有多个核心组件来维护请求和响应执行的流程。这些组件被清晰地分开，并且具有不同的接口和实现类，因此可以根据需求使用。这些核心组件如下：

| **组件** | **摘要** |
| --- | --- |
| `DispatcherServlet` | 它作为 Spring MVC 框架的前端控制器，负责 HTTP 请求和响应的生命周期。 |
| `HandlerMapping` | 当请求到来时，这个组件负责决定哪个控制器将处理 URL。 |
| `Controller` | 它执行业务逻辑并映射`ModelAndView`中的结果数据。 |
| `ModelAndView` | 它以执行结果和视图对象的形式保存模型数据对象。 |
| `ViewResolver` | 它决定要呈现的视图。 |
| `View` | 它显示来自模型对象的结果数据。 |

以下图表说明了 Spring MVC 架构中前面组件的流程：

![](img/742e69c4-d5cc-407c-aa61-c4a201de15e2.png)

Spring MVC 架构

让我们了解架构的基本流程：

1.  当传入的**请求**到来时，它被前端控制器`DispatcherServlet`拦截。在拦截**请求**后，前端控制器找到适当的`HandlerMapping`。

1.  `HandlerMapping`将客户端**请求**调用映射到适当的`Controller`，根据配置文件或注解`Controller`列表，并将`Controller`信息返回给前端控制器。

1.  `DispatcherServlet`将**请求**分派到适当的`Controller`。

1.  `Controller`执行在`Controller`方法下定义的业务逻辑，并将结果数据以`ModelAndView`的形式返回给前端控制器。

1.  前端控制器根据`ModelAndView`中的值获取**视图名称**并将其传递给`ViewResolver`以根据配置的视图解析器解析实际视图。

1.  **视图**使用**模型**对象来呈现屏幕。输出以`HttpServletResponse`的形式生成并传递给前端控制器。

1.  前端控制器将**响应**发送回 Servlet 容器，以将输出发送回用户。

现在，让我们了解 Spring MVC 配置方法。Spring MVC 配置可以通过以下方式进行设置：

+   基于 XML 的配置

+   基于 Java 的配置

在使用上述方法进行配置之前，让我们定义设置 Spring MVC 应用程序所涉及的步骤：

1.  配置前端控制器

1.  创建 Spring 应用程序上下文

1.  配置`ViewResolver`

# 基于 XML 的配置

在基于 XML 的配置中，我们将使用 XML 文件来进行 Spring MVC 配置。让我们按照上述步骤继续进行配置。

# 配置前端控制器

要在基于 XML 的配置中配置前端控制器 Servlet`DispatcherServlet`，我们需要在`web.xml`文件中添加以下 XML 代码：

```java
  <servlet>
    <servlet-name>spring-mvc</servlet-name>
    <servlet-class>
      org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>/WEB-INF/spring-mvc-context.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>

  <servlet-mapping>
    <servlet-name>spring-mvc</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
```

在上述 XML 代码中，我们首先配置了`DispatcherServlet`。然后，我们提到了上下文配置位置`/WEB-INF/spring-mvc-context.xml`。我们将`load-on-startup`值设置为`1`，因此 Servlet 容器将在启动时加载此 Servlet。在第二部分中，我们定义了`servlet-mapping`标签，将 URL`/`映射到`DispatcherServlet`。现在，我们将在下一步中定义 Spring 应用程序上下文。

在`DispatcherServlet`配置下配置`load-on-startup`元素是一个好习惯，以便在集群环境中，如果 Spring 没有启动并且一旦部署就会有大量的调用命中您的 Web 应用程序，您可能会面临超时问题。

# 创建 Spring 应用程序上下文

在`web.xml`中配置`DispatcherServlet`之后，让我们继续创建一个 Spring 应用程序上下文。为此，我们需要在`spring-mvc-context.xml`文件中添加以下 XML 代码：

```java
<beans>
<!-- Schema definitions are skipped. -->
<context:component-scan base-            package="com.packt.springhighperformance.ch4.controller" />
<mvc:annotation-driven />
</beans>
```

在上述 XML 代码中，我们首先为`com.packt.springhighperformance.ch4.controller`包定义了一个组件扫描标签`<context:component-scan />`，以便所有的 bean 和控制器都能被创建和自动装配。

然后，我们使用了`<mvc:annotation-driven />`来自动注册不同的 bean 和组件，包括请求映射、数据绑定、验证和使用`@ResponseBody`进行自动转换功能。

# 配置 ViewResolver

要配置`ViewResolver`，我们需要在`spring-mvc-context.xml`文件中为`InternalResourceViewResolver`类指定一个 bean，在`<mvc:annotation-driven />`之后。让我们这样做：

```java
<beans>
<!-- Schema definitions are skipped. -->
<context:component-scan base- package="com.packt.springhighperformance.ch4.controller" />
<mvc:annotation-driven />

<bean
 class="org.springframework.web.servlet.view.InternalResourceViewResolv  er">
    <property name="prefix">
      <value>/WEB-INF/views/</value>
    </property>
    <property name="suffix">
      <value>.jsp</value>
    </property>
  </bean>
</beans>
```

在配置`ViewResolver`之后，我们将创建一个`Controller`来测试配置。但是，在继续之前，让我们看看基于 Java 的配置。

# 基于 Java 的配置

对于基于 Java 的 Spring MVC 配置，我们将按照与基于 XML 的配置相同的步骤进行。在基于 Java 的配置中，所有配置都将在 Java 类下完成。让我们按照顺序进行。

# 配置前端控制器

在 Spring 5.0 中，有三种方法可以通过实现或扩展以下三个类来以编程方式配置`DispatcherServlet`：

+   `WebAppInitializer` 接口

+   `AbstractDispatcherServletInitializer` 抽象类

+   `AbstractAnnotationConfigDispatcherServletInitializer` 抽象类

我们将使用`AbstractDispatcherServletInitializer`类，因为它是使用基于 Java 的 Spring 配置的应用程序的首选方法。它是首选的，因为它允许我们启动一个 Servlet 应用程序上下文，以及一个根应用程序上下文。

我们需要创建以下类来配置`DispatcherServlet`：

```java
import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

public class SpringMvcWebInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

  @Override
  protected Class<?>[] getRootConfigClasses() {
    return null;
  }

  @Override
  protected Class<?>[] getServletConfigClasses() {
    return new Class[] { SpringMvcWebConfig.class };
  }

  @Override
  protected String[] getServletMappings() {
    return new String[] { "/" };
  }
}
```

前面的类代码等同于我们在*基于 XML 的配置*部分创建的`web.xml`文件配置。在前面的类中，`getRootConfigClasses()`方法用于指定根应用程序上下文配置类（如果不需要，则为`null`）。`getServletConfigClasses()`用于指定 Web 应用程序配置类（如果不需要，则为`null`）。`getServletMappings()`方法用于指定`DispatcherServlet`的 Servlet 映射。首先加载根配置类，然后加载 Servlet 配置类。根配置类将创建一个`ApplicationContext`，它将作为父上下文，而 Servlet 配置类将创建一个`WebApplicationContext`，它将作为父上下文的子上下文。

# 创建一个 Spring 应用程序上下文并配置 ViewResolver

在 Spring 5.0 中，要使用 Java 配置创建 Spring 应用程序上下文并配置`ViewResolver`，需要在类中添加以下代码：

```java
@Configuration
@EnableWebMvc
@ComponentScan({ "com.packt.springhighperformance.ch4.bankingapp.controller"})
public class SpringMvcWebConfig implements WebMvcConfigurer {

  @Bean
  public InternalResourceViewResolver resolver() {
    InternalResourceViewResolver resolver = new 
    InternalResourceViewResolver();
    resolver.setPrefix("/WEB-INF/views/");
    resolver.setSuffix(".jsp");
    return resolver;
  }

}
```

在前面的代码中，我们创建了一个类`SpringMvcWebConfig`，实现了`WebMvcConfigurer`接口，该接口提供了自定义 Spring MVC 配置的选项。`@EnableWebMvc`对象启用了 Spring MVC 的默认配置。`@ComponentScan`对象指定了要扫描控制器的基本包。这两个注解`@EnableWebMvc`和`@ComponentScan`等同于我们在*基于 XML 的配置*部分中创建的`spring-mvc-context.xml`中的`<context:component-scan />`和`<mvc:annotation-driven />`。`resolve()`方法返回`InternalResourceViewResolver`，它有助于从预配置的目录中映射逻辑视图名称。

# 创建一个控制器

现在，让我们创建一个控制器类来映射`/home`请求，如下所示：

```java
package com.packt.springhighperformance.ch4.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class BankController {

  @RequestMapping(value = "/home")
  public String home() {
    return "home";
  }
}
```

在前面的代码中，`@Controller`定义了一个包含请求映射的 Spring MVC 控制器。`@RequestMapping(value = "home")`对象定义了一个映射 URL`/home`到一个方法`home()`。因此，当浏览器发送一个`/home`请求时，它会执行`home()`方法。

# 创建一个视图

现在，让我们在`src/main/webapp/WEB-INF/views/home.jsp`文件夹中创建一个视图`home.jsp`，其中包含以下 HTML 内容：

```java
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
<title>Spring MVC</title>
</head>
<body>
  <h2>Welcome to Bank</h2>
</body>
</html>
```

现在，当我们运行这个应用程序时，它将显示以下输出：

![](img/777d9599-35dd-4ca1-bb09-689f78e850d1.png)

在下一节中，我们将学习关于 Spring 异步处理的内容。

# Spring 异步处理，@Async 注解

Spring 提供了对异步方法执行的支持。这也可以使用线程来实现，但会使代码更复杂，有时会导致更多的错误和 bug。当我们需要以异步方式执行简单操作时，使用线程来处理会是一个繁琐的过程。有些情况下需要异步执行操作，比如从一台机器发送消息到另一台机器。异步处理的主要优势在于调用者不必等待被调用方法的完成。为了在单独的线程中执行方法，需要使用`@Async`注解对方法进行注解。

可以通过使用`@EnableAsync`注解来启用异步处理，以在后台线程池中运行`@Async`方法。以下是启用异步处理的 Java 配置示例：

```java
@Configuration
@EnableAsync
public class SpringAppAsyncConfig { ... }
```

异步处理也可以通过使用 XML 配置来启用，如下所示：

```java
<task:executor id="myappexecutor" pool-size="10" />
<task:annotation-driven executor="myappexecutor"/>
```

# @Async 注解模式

`@Async`注解处理方法有两种模式：

+   发送并忘记模式

+   结果检索模式

# 发送并忘记模式

在这种模式下，方法将配置为`void`类型，以异步运行：

```java
@Async
public void syncCustomerAccounts() {
    logger.info("Customer accounts synced successfully.");
}
```

# 结果检索模式

在这种模式下，方法将配置一个返回类型，通过`Future`类型来包装结果：

```java
@Service
public class BankAsyncService {

  private static final Logger LOGGER = 
  Logger.getLogger(BankAsyncService.class);

  @Async
    public Future<String> syncCustomerAccount() throws 
    InterruptedException {
    LOGGER.info("Sync Account Processing Started - Thread id: " + 
    Thread.currentThread().getId());

    Thread.sleep(2000);

    String processInfo = String.format("Sync Account Processing 
    Completed - Thread Name= %d, Thread Name= %s", 
    Thread.currentThread().getId(), 
    Thread.currentThread().getName());

    LOGGER.info(processInfo);

    return new AsyncResult<String>(processInfo);
    }
}
```

Spring 还提供了对`AsyncResult`类的支持，该类实现了`Future`接口。它可以用于跟踪异步方法调用的结果。

# @Async 注解的限制

`@Async`注解有以下限制：

+   方法需要是`public`，这样它才能被代理

+   异步方法的自我调用不起作用，因为它会绕过代理直接调用底层方法

# 线程池执行程序

你可能想知道我们如何声明异步方法将使用的线程池。默认情况下，对于线程池，Spring 将尝试在上下文中找到一个名为`TaskExecutor`的唯一 bean，或者一个名为`TaskExecutor`的`Executor` bean。如果前两个选项都无法解析，Spring 将使用`SimpleAsyncTaskExecutor`来处理异步方法处理。

然而，有时我们不想为应用程序的所有任务使用相同的线程池。我们可以为每个方法使用不同的线程池，并为每个方法配置不同的线程池。为此，我们只需要将执行器名称传递给每个方法的`@Async`注解。

为了启用异步支持，`@Async`注解是不够的；我们需要在配置类中使用`@EnableAsync`注解。

在 Spring MVC 中，当我们使用`AbstractAnnotationConfigDispatcherServletInitializer`初始化类配置`DispatcherServlet`时，它默认启用了`isAsyncSupported`标志。

现在，我们需要为异步方法调用声明一个线程池定义。在 Spring MVC 基于 Java 的配置中，可以通过在 Spring Web MVC 配置类中覆盖`WebMvcConfigurer`接口的`configureAsyncSupport()`方法来实现。让我们按照以下方式覆盖这个方法：

```java
@Override
public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
    ThreadPoolTaskExecutor t = new ThreadPoolTaskExecutor();
        t.setCorePoolSize(10);
        t.setMaxPoolSize(100);
        t.setThreadNamePrefix("BankAccountSync");
        t.initialize();
        configurer.setTaskExecutor(t);
}
```

在前面的方法中，我们通过覆盖`configureAsyncSupport()`方法配置了线程池执行程序。现在，让我们通过控制器类调用服务类`BankAsyncService`中创建的异步方法，如下所示：

```java
@Controller
public class BankController {

  private static final Logger LOGGER = Logger.getLogger(BankAsyncService.class);

  @Autowired
  BankAsyncService syncService;

  @RequestMapping(value = "/syncacct")
  @ResponseBody
  public Callable<String> syncAccount() {
    LOGGER.info("Entering in controller");

    Callable<String> asyncTask = new Callable<String>() {

      @Override
      public String call() throws Exception {
        Future<String> processSync = syncService.syncCustomerAccount();
        return processSync.get();
      }
    };

    LOGGER.info("Leaving from controller");
    return asyncTask;
  }
}
```

在前面的示例中，当我们请求`/syncacct`时，它将调用`syncAccount()`并在单独的线程中返回异步方法的结果。

# Spring 异步的 CompletableFuture

`CompletableFuture`类是在 Java 8 中引入的，它提供了一种简单的方式来编写异步、多线程、非阻塞的代码。在 Spring MVC 中，也可以在使用`@Async`注解的公共方法的控制器、服务和存储库中使用`CompletableFuture`。`CompletableFuture`实现了`Future`接口，该接口提供了异步计算的结果。

我们可以通过以下简单方式创建`CompletableFuture`：

```java
CompletableFuture<String> completableFuture = new CompletableFuture<String>();
```

要获取这个`CompletableFuture`的结果，我们可以调用`CompletableFuture.get()`方法。该方法将被阻塞，直到`Future`完成。为此，我们可以手动调用`CompletableFuture.complete()`方法来`complete` `Future`：

```java
completableFuture.complete("Future is completed")
```

# runAsync() - 异步运行任务

当我们想要异步执行后台活动任务，并且不想从该任务中返回任何东西时，我们可以使用`CompletableFuture.runAsync()`方法。它以`Runnable`对象作为参数，并返回`CompletableFuture<Void>`类型。

让我们尝试通过在我们的`BankController`类中创建另一个控制器方法来使用`runAsync()`方法，如下所示：

```java
@RequestMapping(value = "/synccust")
  @ResponseBody
  public CompletableFuture<String> syncCustomerDetails() {
    LOGGER.info("Entering in controller");

    CompletableFuture<String> completableFuture = new 
    CompletableFuture<>();
    CompletableFuture.runAsync(new Runnable() {

      @Override
      public void run() {
        try {           
           completableFuture.complete(syncService.syncCustomerAccount()
           .get());
        } catch (InterruptedException | ExecutionException e) {
          completableFuture.completeExceptionally(e);
        }

      }
    }); 
      LOGGER.info("Leaving from controller");
      return completableFuture;
  }
```

在前面的示例中，当请求使用`/synccust`路径时，它将在单独的线程中运行`syncCustomerAccount()`，并在不返回任何值的情况下完成任务。

# supplyAsync() - 异步运行任务，带有返回值

当我们想要在异步完成任务后返回结果时，我们可以使用`CompletableFuture.supplyAsync()`。它以`Supplier<T>`作为参数，并返回`CompletableFuture<T>`。

让我们通过在我们的`BankController`类中创建另一个控制器方法来检查`supplyAsync()`方法，示例如下：

```java
@RequestMapping(value = "/synccustbal")
  @ResponseBody
  public CompletableFuture<String> syncCustomerBalance() {
    LOGGER.info("Entering in controller");

    CompletableFuture<String> completableFuture = 
    CompletableFuture.supplyAsync(new Supplier<String>() {

      @Override
      public String get() {
        try {
          return syncService.syncCustomerBalance().get();
        } catch (InterruptedException | ExecutionException e) {
          LOGGER.error(e);
        }
        return "No balance found";
      }
    }); 
      LOGGER.info("Leaving from controller");
      return completableFuture;
  }
```

`CompletableFuture`对象使用全局线程池`ForkJoinPool.commonPool()`在单独的线程中执行任务。我们可以创建一个线程池并将其传递给`runAsync()`和`supplyAsync()`方法。

以下是`runAsync()`和`supplyAsync()`方法的两种变体：

```java
CompletableFuture<Void> runAsync(Runnable runnable)
CompletableFuture<Void> runAsync(Runnable runnable, Executor executor)
CompletableFuture<U> supplyAsync(Supplier<U> supplier)
CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)
```

# 将回调附加到 CompletableFuture

`CompletableFuture.get()`会阻塞对象，并等待`Future`任务完成并返回结果。要构建一个异步系统，应该有一个回调，在`Future`任务完成时自动调用。我们可以使用`thenApply()`、`thenAccept()`和`thenRun()`方法将回调附加到`CompletableFuture`。

# Spring Security 配置

Spring Security 是 Java EE 企业应用程序广泛使用的安全服务框架。在认证级别上，Spring Security 提供了不同类型的认证模型。其中一些模型由第三方提供，一些认证功能集由 Spring Security 自身提供。Spring Security 提供了以下一些认证机制：

+   基于表单的认证

+   OpenID 认证

+   LDAP 专门用于大型环境

+   容器管理的认证

+   自定义认证系统

+   JAAS

让我们看一个示例来在 Web 应用程序中激活 Spring Security。我们将使用内存配置。

# 配置 Spring Security 依赖项

要在 Web 应用程序中配置 Spring Security，我们需要将以下 Maven 依赖项添加到我们的**项目对象模型**（**POM**）文件中：

```java
<!-- spring security -->
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>${spring.framework.version}</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>${spring.framework.version}</version>
</dependency>
```

# 为传入请求配置安全过滤器

在 Web 应用程序中实现安全性时，最好验证所有传入的请求。在 Spring Security 中，框架本身查看传入的请求并验证用户以执行操作，基于提供的访问权限。为了拦截 Web 应用程序的所有传入请求，我们需要配置`filter`，`DelegatingFilterProxy`，它将把请求委托给 Spring 管理的`FilterChainProxy`：

```java
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>
        org.springframework.web.filter.DelegatingFilterProxy
    </filter-class>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

基于`filter`配置，所有请求将通过此`filter`。现在，让我们配置与安全相关的内容，如身份验证、URL 安全和角色访问。

# 配置 Spring Security

现在，我们将通过创建 Spring Security 配置类来配置 Spring Security 身份验证和授权，如下所示：

```java
@EnableWebSecurity
public class SpringMvcSecurityConfig extends WebSecurityConfigurerAdapter {

  @Autowired
  PasswordEncoder passwordEncoder;

  @Override
  protected void configure(AuthenticationManagerBuilder auth)       
  throws   
  Exception {
    auth
    .inMemoryAuthentication()
    .passwordEncoder(passwordEncoder)
    .withUser("user").password(passwordEncoder.encode("user@123"))
    .roles("USER")
    .and()
    .withUser("admin").password(passwordEncoder.
    encode("admin@123")        
    ).roles("USER", "ADMIN");
  }

  @Bean
  public PasswordEncoder passwordEncoder() {
      return new BCryptPasswordEncoder();
  }

  @Override
  protected void configure(HttpSecurity http) throws Exception {
     http.authorizeRequests()
    .antMatchers("/login").permitAll()
    .antMatchers("/admin/**").hasRole("ADMIN")
    .antMatchers("/**").hasAnyRole("ADMIN","USER")
    .and().formLogin()
    .and().logout().logoutSuccessUrl("/login").permitAll()
    .and()
    .csrf().disable();
  }
}
```

让我们理解上述配置：

+   @EnableWebSecurity：它启用了 Spring Security 的 Web 安全支持，并提供了 Spring MVC 集成。

+   `WebSecurityConfigurerAdapter`：它提供了一组方法，用于启用特定的 Web 安全配置。

+   `protected void configure(AuthenticationManagerBuilder auth)`: 在本示例中，我们使用了内存认证。它可以用于使用`auth.jdbcAuthentication()`连接到数据库，或者使用`auth.ldapAuthentication()`连接到**轻量级目录访问协议**（**LDAP**）。

+   `.passwordEncoder(passwordEncoder)`: 我们使用了密码编码器`BCryptPasswordEncoder`。

+   `.withUser("user").password(passwordEncoder.encode("user@123"))`: 为认证设置用户 ID 和编码密码。

+   `.roles("USER")`: 为用户分配角色。

+   `protected void configure(HttpSecurity http)`: 用于保护需要安全性的不同 URL。

+   `.antMatchers("/login").permitAll()`: 允许所有用户访问登录页面。

+   `.antMatchers("/admin/**").hasRole("ADMIN")`: 允许具有`ADMIN`角色的用户访问管理员面板。

+   `.antMatchers("/**").anyRequest().hasAnyRole("ADMIN", "USER")`: 这意味着对于带有`"/"`的任何请求，您必须使用`ADMIN`或`USER`角色登录。

+   `.and().formLogin()`: 它将提供一个默认的登录页面，带有用户名和密码字段。

+   `.and().logout().logoutSuccessUrl("/login").permitAll()`: 当用户注销时，设置注销成功页面。

+   `.csrf().disable()`: 默认情况下，**跨站请求伪造**（**CSRF**）标志是启用的。在这里，我们已经从配置中禁用了它。

# 添加一个控制器

我们将使用以下`BankController`类进行 URL 映射：

```java
@Controller
public class BankController {

  @GetMapping("/")
  public ModelAndView home(Principal principal) {
    ModelAndView model = new ModelAndView();
    model.addObject("title", "Welcome to Bank");
    model.addObject("message", "Hi " + principal.getName());
    model.setViewName("index");
    return model;
  }

  @GetMapping("/admin**")
  public ModelAndView adminPage() {
    ModelAndView model = new ModelAndView();
    model.addObject("title", "Welcome to Admin Panel");
    model.addObject("message", "This is secured page - Admin 
    Panel");
    model.setViewName("admin");
    return model;
  }

  @PostMapping("/logout")
  public String logout(HttpServletRequest request, 
  HttpServletResponse 
  response) {
    Authentication auth = 
    SecurityContextHolder.getContext().getAuthentication();
    if (auth != null) {
      new SecurityContextLogoutHandler().logout(request, response, 
      auth);
      request.getSession().invalidate();
    }
    return "redirect:/login";
  }
}
```

现在，当我们运行这个例子时，它将首先显示由 Spring 框架提供的登录身份验证表单，然后再尝试访问 Web 应用程序的任何 URL。如果用户使用`USER`角色登录并尝试访问管理员面板，他们将被限制访问。如果用户使用`ADMIN`角色登录，他们将能够访问用户面板和管理员面板。

# 身份验证缓存

当应用程序受到最大数量的调用时，Spring Security 的性能成为一个主要关注点。默认情况下，Spring Security 为每个新请求创建一个新会话，并每次准备一个新的安全上下文。在维护用户身份验证时，这会成为一个负担，从而降低性能。

例如，我们有一个 API，每个请求都需要身份验证。如果对该 API 进行多次调用，将会影响使用该 API 的应用程序的性能。因此，让我们在没有缓存实现的情况下了解这个问题。看一下以下日志，我们使用`curl`命令调用 API，没有缓存实现：

```java
curl -sL --connect-timeout 1 -i http://localhost:8080/authentication-cache/secure/login -H "Authorization: Basic Y3VzdDAwMTpUZXN0QDEyMw=="
```

看一下以下日志：

```java
21:53:46.302 RDS DEBUG JdbcTemplate - Executing prepared SQL query
21:53:46.302 RDS DEBUG JdbcTemplate - Executing prepared SQL statement [select username,password,enabled from users where username = ?]
21:53:46.302 RDS DEBUG DataSourceUtils - Fetching JDBC Connection from DataSource
21:53:46.302 RDS DEBUG SimpleDriverDataSource - Creating new JDBC Driver Connection to [jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=false]
21:53:46.307 RDS DEBUG DataSourceUtils - Returning JDBC Connection to DataSource
21:53:46.307 RDS DEBUG JdbcTemplate - Executing prepared SQL query
21:53:46.307 RDS DEBUG JdbcTemplate - Executing prepared SQL statement [select username,authority from authorities where username = ?]
21:53:46.307 RDS DEBUG DataSourceUtils - Fetching JDBC Connection from DataSource
21:53:46.307 RDS DEBUG SimpleDriverDataSource - Creating new JDBC Driver Connection to [jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=false]
21:53:46.307 RDS DEBUG DataSourceUtils - Returning JDBC Connection to DataSource
```

每次调用此 API 时，它将使用数据库值对用户名和密码进行身份验证。这会影响应用程序的性能，并且如果用户频繁调用，可能会导致不必要的负载。

克服这个问题的一个体面的解决方案之一是缓存用户身份验证一段特定的时间。我们将使用带有正确配置的`AuthenticationProvider`的`UserCache`的实现，并将其传递给`AuthenticationManagerBuilder`。我们将使用`EhCache`来操作缓存对象。我们可以通过以下步骤来使用这个解决方案：

1.  实现缓存配置类

1.  向`AuthenticationProvider`提供`UserCache`

1.  向`AuthenticationManagerBuilder`提供`AuthenticationProvider`

# 实现缓存配置类

我们创建了以下类，它将提供`UserCache` bean，并将其提供给`AuthenticationProvider`：

```java
@Configuration
@EnableCaching
public class SpringMvcCacheConfig {

  @Bean
  public EhCacheFactoryBean ehCacheFactoryBean() {
    EhCacheFactoryBean ehCacheFactory = new EhCacheFactoryBean();
    ehCacheFactory.setCacheManager(cacheManagerFactoryBean()
    .getObject());
    return ehCacheFactory;
  }

  @Bean
  public CacheManager cacheManager() {
    return new         
    EhCacheCacheManager(cacheManagerFactoryBean().getObject());
  }

  @Bean
  public EhCacheManagerFactoryBean cacheManagerFactoryBean() {
    EhCacheManagerFactoryBean cacheManager = new 
    EhCacheManagerFactoryBean();
    return cacheManager;
  }

  @Bean
  public UserCache userCache() {
    EhCacheBasedUserCache userCache = new EhCacheBasedUserCache();
    userCache.setCache(ehCacheFactoryBean().getObject());
    return userCache;
  }
}
```

在上述类中，`@EnableCaching`启用了缓存管理。

# 向 AuthenticationProvider 提供 UserCache

现在，我们将创建的`UserCache` bean 提供给`AuthenticationProvider`：

```java
@Bean
public AuthenticationProvider authenticationProviderBean() {
     DaoAuthenticationProvider authenticationProvider = new              
     DaoAuthenticationProvider();
     authenticationProvider.setPasswordEncoder(passwordEncoder);
     authenticationProvider.setUserCache(userCache);
     authenticationProvider.
     setUserDetailsService(userDetailsService());
     return authenticationProvider;
}
```

# 向 AuthenticationManagerBuilder 提供 AuthenticationProvider

现在，在 Spring Security 配置类中向`AuthenticationManagerBuilder`提供`AuthenticationProvider`：

```java
@Autowired
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws     
    Exception {

       auth
         .eraseCredentials(false)
         //Providing AuthenticationProvider to 
          AuthenticationManagerBuilder.
         .authenticationProvider(authenticationProviderBean())
         .jdbcAuthentication()
         .dataSource(dataSource); 
    }
```

现在，让我们调用该 API 并检查身份验证的性能。如果我们调用 API 四次，将生成以下日志：

```java
22:46:55.314 RDS DEBUG EhCacheBasedUserCache - Cache hit: false; username: cust001
22:46:55.447 RDS DEBUG JdbcTemplate - Executing prepared SQL query
22:46:55.447 RDS DEBUG JdbcTemplate - Executing prepared SQL statement [select username,password,enabled from users where username = ?]
22:46:55.447 RDS DEBUG DataSourceUtils - Fetching JDBC Connection from DataSource
22:46:55.447 RDS DEBUG SimpleDriverDataSource - Creating new JDBC Driver Connection to [jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=false]
22:46:55.463 RDS DEBUG DataSourceUtils - Returning JDBC Connection to DataSource
22:46:55.463 RDS DEBUG JdbcTemplate - Executing prepared SQL query
22:46:55.463 RDS DEBUG JdbcTemplate - Executing prepared SQL statement [select username,authority from authorities where username = ?]
22:46:55.463 RDS DEBUG DataSourceUtils - Fetching JDBC Connection from DataSource
22:46:55.463 RDS DEBUG SimpleDriverDataSource - Creating new JDBC Driver Connection to [jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=false]
22:46:55.479 RDS DEBUG DataSourceUtils - Returning JDBC Connection to DataSource
22:46:55.603 RDS DEBUG EhCacheBasedUserCache - Cache put: cust001
22:47:10.118 RDS DEBUG EhCacheBasedUserCache - Cache hit: true; username: cust001
22:47:12.619 RDS DEBUG EhCacheBasedUserCache - Cache hit: true; username: cust001
22:47:14.851 RDS DEBUG EhCacheBasedUserCache - Cache hit: true; username: cust001
```

正如您在前面的日志中所看到的，最初，`AuthenticationProvider`从缓存中搜索`UserDetails`对象；如果它无法从缓存中获取，`AuthenticationProvider`将查询数据库以获取`UserDetails`，并将更新后的对象放入缓存中，以便以后的所有调用都将从缓存中检索`UserDetails`对象。

如果您更新用户的密码并尝试使用新密码对用户进行身份验证，但与缓存中的值不匹配，则它将从数据库中查询`UserDetails`。

# 使用 Spring Security 实现快速和无状态的 API 身份验证

Spring Security 还提供了用于保护非浏览器客户端（如移动应用程序或其他应用程序）的无状态 API。我们将学习如何配置 Spring Security 来保护无状态 API。此外，我们将找出在设计安全解决方案和提高用户身份验证性能时需要考虑的重要点。

# API 身份验证需要 JSESSIONID cookie

对于 API 客户端使用基于表单的身份验证并不是一个好的做法，因为需要在请求链中提供`JSESSIONID` cookie。Spring Security 还提供了使用 HTTP 基本身份验证的选项，这是一种较旧的方法，但效果很好。在 HTTP 基本身份验证方法中，用户/密码详细信息需要与请求头一起发送。让我们看一下以下 HTTP 基本身份验证配置的示例：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
      http
        .authorizeRequests()
        .anyRequest().authenticated()
        .and()
        .httpBasic();
}
```

在上面的示例中，`configure()`方法来自`WebSecurityConfigurerAdapter`抽象类，该类提供了此方法的默认实现。子类应该通过调用`super`来调用此方法，因为它可能会覆盖它们的配置。这种配置方法有一个缺点；每当我们调用受保护的端点时，它都会创建一个新的会话。让我们使用`curl`命令来调用端点来检查一下：

```java
C:\>curl -sL --connect-timeout 1 -i http://localhost:8080/fast-api-spring-security/secure/login/ -H "Authorization: Basic Y3VzdDAwMTpDdXN0QDEyMw=="
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Set-Cookie: JSESSIONID=B85E9773E6C1E71CE0EC1AD11D897529; Path=/fast-api-spring-security; HttpOnly
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 19
Date: Tue, 27 Mar 2018 18:07:43 GMT

Welcome to the Bank
```

我们有一个会话 ID cookie；让我们再次调用它：

```java
C:\>curl -sL --connect-timeout 1 -i http://localhost:8080/fast-api-spring-security/secure/login/ -H "Authorization: Basic Y3VzdDAwMTpDdXN0QDEyMw=="
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Set-Cookie: JSESSIONID=14FEB3708295324482BE1DD600D015CC; Path=/fast-api-spring-security; HttpOnly
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 19
Date: Tue, 27 Mar 2018 18:07:47 GMT

Welcome to the Bank
```

正如您所看到的，每个响应中都有两个不同的会话 ID。在上面的示例中，为了测试目的，我们发送了带有编码的用户名和密码的`Authorization`头。当您提供用户名和密码进行身份验证时，您可以从浏览器中获取`Basic Y3VzdDAwMTpDdXN0QDEyMw==`头值。

# API 身份验证不需要 JSESSIONID cookie

由于 API 客户端身份验证不需要会话，我们可以通过以下配置轻松摆脱会话 ID：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
      http
      .sessionManagement()
        .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
        .and()
        .authorizeRequests()
        .anyRequest().authenticated()
        .and()
        .httpBasic();
}
```

正如您所看到的，在前面的配置中，我们使用了`SessionCreationPolicy.STATELESS`。通过这个选项，在响应头中不会添加会话 cookie。让我们看看在这个改变之后会发生什么：

```java
C:\>curl -sL --connect-timeout 1 -i http://localhost:8080/fast-api-spring-security/secure/login/ -H "Authorization: Basic Y3VzdDAwMTpDdXN0QDEyMw=="
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
Cache-Control: no-cache, no-store, max-age=0, must-revalidate
Pragma: no-cache
Expires: 0
X-Frame-Options: DENY
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 19
Date: Tue, 27 Mar 2018 18:24:32 GMT

Welcome to the Bank
```

在上面的示例中，在响应头中找不到会话 cookie。因此，通过这种方式，我们可以使用 Spring Security 管理 API 的无状态身份验证。

# 使用 JMX 监控和管理 Tomcat

**Java 管理扩展**（**JMX**）提供了一种强大的机制来监视和管理 Java 应用程序。它可以在 Tomcat 中启用，以监视线程、CPU 使用率和堆内存，并配置**MBeans**。Spring 提供了开箱即用的 JMX 支持，我们可以使用它轻松地将我们的 Spring 应用程序集成到 JMX 架构中。

JMX 支持提供以下核心功能：

+   轻松灵活地支持控制 bean 的管理接口

+   声明支持通过远程连接器公开 MBean

+   将 Spring bean 自动注册为 JMX MBean

+   简化支持代理本地和远程 MBean 资源

JMX 功能有三个级别：

+   **仪器级别：**这个级别包含由一个或多个 Java bean 表示的组件和资源，这些组件和资源被称为**托管 bean**或 MBean。

+   **代理级别：**这被称为中间代理，称为**MBean 服务器**。它从远程管理级别获取请求，并将其传递给适当的 MBean。它还可以接收来自 MBean 的与状态更改相关的通知，并将其转发回远程管理级别。

+   **远程管理级别：**这一层由连接器、适配器或客户端程序组成。它向代理级别发送请求，并接收请求的响应。用户可以使用连接器或客户端程序（如 JConsole）连接到 MBean 服务器，使用**远程方法调用**（**RMI**）或**Internet 互操作对象协议**（**IIOP**）等协议，并使用适配器。

简而言之，远程管理级别的用户向代理级别发送请求，代理级别在仪器级别找到适当的 MBean，并将响应发送回用户。

# 连接 JMX 以监视 Tomcat

要在 Tomcat 上配置 JMX，我们需要在 JVM 启动时设置相关的系统属性。我们可以使用以下方法。

我们可以在`{tomcat-folder}\bin\`中更新`catalina.sh`或`catalina.bat`文件，添加以下值：

```java
-Dcom.sun.management.jmxremote 
-Dcom.sun.management.jmxremote.port={port to access} 
-Dcom.sun.management.jmxremote.authenticate=false 
-Dcom.sun.management.jmxremote.ssl=false
```

例如，我们可以在`{tomcat-folder}\bin\catalina.bat`中添加以下值：

```java
set JAVA_OPTS="-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=8990
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false"
```

如果您想在 Eclipse 中为 Tomcat 配置 JMX，您需要执行以下操作：

1.  转到“窗口”|“显示视图”|“服务器”。

1.  双击 localhost 上的 Tomcat v8.0 服务器，打开 Tomcat 概述配置窗口。

1.  在“常规信息”下，单击“打开启动配置”。

1.  选择“编辑启动配置属性”的参数选项卡。

1.  在 VM 参数中，添加以下属性，然后单击“确定”：

```java
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=8990
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
```

做出这些更改后，我们需要重新启动 Tomcat 服务器。之后，我们需要使用 JConsole 测试连接。打开 JConsole 后，我们需要提供远程进程的主机名和端口号，如下所示：

![](img/971e79e2-6a7a-4f18-9236-1ec792804c7e.png)

在上面的截图中，我们已经提供了主机名为`localhost`，端口号为`8990`。当您单击“连接”时，将会弹出一个对话框，您需要单击“不安全连接”，然后您将连接到 JConsole。

# 创建 MBean

要创建 MBean，我们可以使用`@Managed`注解将任何类转换为 MBean。类`BankTransferService`将金额从一个账户转移到另一个账户。我们将使用此示例进行进一步理解：

```java
@Component
@ManagedResource(objectName = "com.packt.springhighperformance.ch4.mbeans : name=BankMoneyTransferService", description = "Transfers money from one account to another")
public class BankMoneyTransferService {

  private Map<String, Integer> accountMap = new HashMap<String, 
  Integer>();
   {
    accountMap.put("12345", 20000);
    accountMap.put("54321", 10000);
   };

  @ManagedOperation(description = "Amount transfer")
  @ManagedOperationParameters({
      @ManagedOperationParameter(name = "sourceAccount", description = 
       "Transfer from account"),
      @ManagedOperationParameter(name = "destinationAccount",         
        description = "Transfer to account"),
      @ManagedOperationParameter(name = "transferAmount", 
      description = 
        "Amount to be transfer") })
  public void transfer(String sourceAccount, String     
  destinationAccount, int transferAmount) {
    if (transferAmount == 0) {
      throw new IllegalArgumentException("Invalid amount");
    }
    int sourceAcctBalance = accountMap.get(sourceAccount);
    int destinationAcctBalance = accountMap.get(destinationAccount);

    if ((sourceAcctBalance - transferAmount) < 0) {
      throw new IllegalArgumentException("Not enough balance.");
    }
    sourceAcctBalance = sourceAcctBalance - transferAmount;
    destinationAcctBalance = destinationAcctBalance + transferAmount;

    accountMap.put(sourceAccount, sourceAcctBalance);
    accountMap.put(destinationAccount, destinationAcctBalance);
  }

  @ManagedOperation(description = "Check Balance")
  public int checkBalance(String accountNumber) {
    if (StringUtils.isEmpty(accountNumber)) {
      throw new IllegalArgumentException("Enter account no.");
    }
    if (!accountMap.containsKey(accountNumber)) {
      throw new IllegalArgumentException("Account not found.");
    }
    return accountMap.get(accountNumber);
  }

}
```

在上述类中，`@ManagedResource`注解将标记类为 MBean，`@ManagedAttribute`和`@ManagedOperation`注解可用于公开任何属性或方法。`@Component`注解将确保所有带有`@Component`、`@Service`或`@Repository`注解的类将被添加到 Spring 上下文中。

# 在 Spring 上下文中导出 MBean

现在，我们需要在 Spring 应用程序上下文中创建一个`MBeanExporter`。我们只需要在 Spring 上下文 XML 配置中添加以下标签：

```java
<context:mbean-export/>
```

我们需要在“‹context:mbean-export/›”元素之前添加`component-scan`元素；否则，JMX 服务器将无法找到任何 bean。

因此，我们的 Spring 上下文配置将如下所示：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans><!-- Skipped schema definitions -->

  <context:component-scan base-
   package="com.packt.springhighperformance.ch4.mbeans" /> 

<context:mbean-export/>

</beans>
```

现在，我们只需要启动 Tomcat 服务器并打开 JConsole 来查看我们的 MBean。连接到 JConsole 后，转到“MBeans”选项卡，在那里您可以看到我们的包文件夹，其中包含我们的`BankMoneyTransferService` MBean，列在侧边栏中：

![](img/fb0dd8cb-79e4-4ea8-b8e7-d43d3b5e9989.png)

如您在前面的示例中所见，我们的 MBean 已生成并列在 JConsole 中。现在，我们可以通过单击“转账”按钮，调用我们在 MBean 中创建的`transfer()`方法，从一个账户向另一个账户转账。当我们单击“查看余额”按钮时，它将根据输入的账号号码在弹出窗口中显示当前余额。在后台，它将调用`BankMoneyTransferService`类的`checkBalance()`方法。

# Spring MVC 性能改进

Spring MVC 应用程序的性能可以通过多种策略和技巧进行改进。在这里，我们列出了一些可以极大改善性能的策略：

+   使用连接池实现高性能

+   Hibernate 改进

+   测试改进

+   适当的服务器维护

+   使用 Spring Security 的身份验证缓存

+   实现 Executor 服务框架

# 使用连接池实现高性能

在 Spring MVC 中提高性能的最重要特性之一是**连接池**。在这种机制中，创建和管理了*N*个数据库连接池，以提高应用程序的性能。当应用程序需要使用连接时，它只需请求一个连接，使用它，然后将其返回到池中。这个过程的主要优点是连接池中有连接立即可用，因此可以立即使用。池本身处理连接的生命周期，因此开发人员不必等待连接建立。

# Hibernate 改进

另一个提高性能的主要点是关于 Hibernate。脏检查是 Hibernate 提供的一个功能。在脏检查中，Hibernate 会自动识别对象是否被修改并需要更新。Hibernate 会在需要时进行脏检查，以保持性能成本。当特定实体具有对应的具有大量列的表时，成本会增加。为了最小化脏检查成本，我们可以将事务设置为`readOnly`，这将提高性能并消除任何脏检查的需要。

```java
@Transactional(readOnly=true)
public void performanceTestMethod() {
    ....
}
```

另一个与 Hibernate 相关的改进是定期刷新和清理 Hibernate 会话。当数据被插入/修改到数据库时，Hibernate 会在会话中存储已经持久化的实体的一个版本，以防它们在会话关闭之前再次更新。我们可以限制 Hibernate 在会话中存储实体的时间，一旦数据被插入，我们就不需要再将实体存储在持久状态中。因此，我们可以安全地刷新和清理`entityManager`，以使实体的状态与数据库同步，并从缓存中删除实体。这将使应用程序远离内存限制，并肯定会对性能产生积极影响。

```java
entityManager.flush();
entityManager.clear();
```

另一个改进可以通过使用**延迟初始化**来实现。如果我们使用 Hibernate，我们应该确保延迟初始化功能被正确使用。我们应该只在需要时才使用实体的延迟加载。例如，如果我们有一个自定义实体集合，如`Set<Employee>`，配置为延迟初始化，那么该集合的每个实体将使用单独的查询分别加载。因此，如果在集合中延迟初始化了多个实体，那么将会按顺序执行大量查询，这可能会严重影响性能。

# 测试改进

对于测试改进，我们可以构建一个测试环境，可以在其中执行应用程序，并在其中获取结果。我们可以编写可重复的性能测试脚本，关注绝对性能（如页面渲染时间）和规模上的性能（如负载时的性能下降）。我们可以在测试环境中使用分析器。

# 适当的服务器维护

一个与适当的服务器维护相关的主要性能方面（如果性能是主要关注点）。以下是一些应该考虑的重要点，以改善性能：

+   通过创建定期的自动化脚本来清理临时文件。

+   当多个服务器实例正在运行时使用负载均衡器。

+   根据应用程序的需求优化配置。例如，在 Tomcat 的情况下，我们可以参考 Tomcat 的配置建议。

# 使用 Spring Security 的身份验证缓存

在使用 Spring Security 时，可以找到提高性能的重要观点。当请求处理时间被认为是不理想的时候，应该正确配置 Spring Security 以提高性能。可能存在这样一种情况，实际请求处理时间大约为 100 毫秒，而 Spring Security 认证额外增加了 400-500 毫秒。我们可以使用 Spring Security 的认证缓存来消除这种性能成本。

# 实施 Executor 服务框架

通过所有可能的改进，如果在请求处理方面保持并发性，性能可以得到改善。可能存在这样一种情况，即对我们的应用程序进行多个并发访问的负载测试，这可能会影响我们应用程序的性能。在这种情况下，我们应该调整 Tomcat 服务器上的线程默认值。如果存在高并发性，HTTP 请求将被暂停，直到有一个线程可用来处理它们。

通过在业务逻辑中使用 Executor 框架来扩展默认的服务器线程实现，可以实现并发异步调用。

# 总结

在本章中，我们对 Spring MVC 模块有了清晰的了解，并学习了不同的配置方法。我们还学习了 Spring 异步处理概念，以及`CompletableFeature`的实现。之后，我们学习了 Spring Security 模块的配置。我们还了解了 Spring Security 的认证部分和无状态 API。然后，我们学习了 Tomcat 的监控部分和 JMX。最后，我们看了 Spring MVC 的性能改进。

在下一章中，我们将学习关于 Spring 数据库交互的知识。我们将从 Spring JDBC 配置和最佳数据库设计和配置开始。然后，我们将介绍最佳连接池配置。我们还将涵盖`@Transactional`概念以提高性能。最后，我们将介绍数据库设计的最佳实践。
