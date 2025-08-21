# 第九章：AOP 和 Logger 控制

在本章中，我们将学习 Spring **面向方面的编程**（**AOP**）和日志控制，包括它们的理论和实现。我们将在我们现有的 REST API 中集成 Spring AOP，并了解 AOP 和日志控制如何使我们的生活更轻松。

在本章中，我们将涵盖以下主题：

+   Spring AOP 理论

+   Spring AOP 的实现

+   为什么我们需要日志控制？

+   我们如何实现日志控制？

+   集成 Spring AOP 和日志控制

# 面向方面的编程（AOP）

面向方面的编程是一个概念，它在不修改代码本身的情况下为现有代码添加新行为。当涉及到日志记录或方法认证时，AOP 概念真的很有帮助。

在 Spring 中，有许多方法可以使用 AOP。让我们不要深入讨论，因为这将是一个大的讨论话题。在这里，我们只讨论`@Before`切入点以及如何在我们的业务逻辑中使用`@Before`。

# AOP（@Before）与执行

AOP 中的执行术语意味着在`@Aspect`注解本身中有一个切入点，它不依赖于控制器 API。另一种方法是您将不得不在 API 调用中明确提及注解。让我们在下一个主题中讨论显式切入点：

```java
package com.packtpub.aop;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;
@Aspect
@Component
public class TokenRequiredAspect {  
  @Before("execution(* com.packtpub.restapp.HomeController.testAOPExecution())")
  public void tokenRequiredWithoutAnnoation() throws Throwable{
    System.out.println("Before tokenRequiredWithExecution");
  }
}
```

在这个切入点中，我们使用了`@Before`注解，它使用了`execution(* com.packtpub.restapp.HomeController.testAOPWithoutAnnotation())`，这意味着这个切入点将专注于一个特定的方法，在我们的例子中是`HomeController`类中的`testAOPWithoutAnnotation`方法。

对于与 AOP 相关的工作，我们可能需要将依赖项添加到我们的`pom.xml`文件中，如下所示：

```java
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.8.13</version>
    </dependency>
```

上述依赖项将带来所有面向方面的类，以支持我们在本章中的 AOP 实现。

`@Aspect`：这个注解用于使类支持方面。在 Spring 中，可以使用 XML 配置或注解（如`@Aspect`）来实现方面。

`@Component`：这个注解将使类根据 Spring 的组件扫描规则可扫描。通过将这个类与`@Component`和`@Aspect`一起提及，我们告诉 Spring 扫描这个类并将其识别为一个方面。

`HomeController`类的代码如下所示：

```java
  @ResponseBody
  @RequestMapping("/test/aop/with/execution") 
  public Map<String, Object> testAOPExecution(){
    Map<String, Object> map = new LinkedHashMap<>();
    map.put("result", "Aloha");
    return map;
  }
```

在这里，我们只需创建一个新的方法来测试我们的 AOP。您可能不需要创建一个新的 API 来测试我们的 AOP。只要您提供适当的方法名，就应该没问题。为了使读者更容易理解，我们在`HomeContoller`类中创建了一个名为`testAOPExecution`的新方法。

# 测试 AOP @Before 执行

只需在浏览器中调用 API（`http://localhost:8080/test/aop/with/execution`）或使用任何其他 REST 客户端；然后，您应该在控制台中看到以下内容：

```java
Before tokenRequiredWithExecution
```

尽管这个日志并不真正帮助我们的业务逻辑，但我们现在会保留它，以便读者更容易理解流程。一旦我们了解了 AOP 及其功能，我们将把它集成到我们的业务逻辑中。

# AOP（@Before）与注解

到目前为止，我们已经看到了一个基于执行的 AOP 方法，可以用于一个或多个方法。然而，在某些地方，我们可能需要保持实现简单以增加可见性。这将帮助我们在需要的地方使用它，而且它不与任何方法绑定。我们称之为显式基于注解的 AOP。

为了使用这个 AOP 概念，我们可能需要创建一个接口，这个接口将帮助我们实现我们需要的东西。

`TokenRequired`只是我们`Aspect`类的一个基本接口。它将被提供给我们的`Aspect`类，如下所示：

```java
package com.packtpub.aop;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface TokenRequired {
}
```

`@Retention`：保留策略确定注解应在何时被丢弃。在我们的例子中，`RetentionPolicy.RUNTIME`将在 JVM 中通过运行时保留。

其他保留策略如下：

`SOURCE`：它将仅保留源代码，并且在编译时将被丢弃。一旦代码编译完成，注释将变得无用，因此不会写入字节码中。

`CLASS`：它将保留到编译时，并在运行时丢弃。

`@Target`：此注释适用于类级别，并在运行时匹配。目标注释可用于收集目标对象。

以下的`tokenRequiredWithAnnotation`方法将实现我们方面的业务逻辑。为了保持逻辑简单，我们只提供了`System.out.println(..)`。稍后，我们将向该方法添加主要逻辑：

```java
@Aspect
@Component
public class TokenRequiredAspect {
  // old method (with execution)  
  @Before("@annotation(tokenRequired)")
  public void tokenRequiredWithAnnotation(TokenRequired tokenRequired) throws Throwable{
    System.out.println("Before tokenRequiredWithAnnotation");
  } 
}
```

在前面的代码中，我们创建了一个名为`tokenRequiredWithAnnotation`的方法，并为该方法提供了`TokenRequired`接口作为参数。我们可以看到该方法顶部的`@Before`注释，并且`@annotation(tokenRequired)`。每次在任何方法中使用`@TokenRequired`注释时，将调用此方法。您可以如下所示查看注释用法：

```java
  @ResponseBody
  @RequestMapping("/test/aop/with/annotation")
  @TokenRequired
  public Map<String, Object> testAOPAnnotation(){
    Map<String, Object> map = new LinkedHashMap<>();
    map.put("result", "Aloha");   
    return map;
  }
```

以前的 AOP 方法和这个之间的主要区别是`@TokenRequired`。在旧的 API 调用者中，我们没有明确提到任何 AOP 注释，但在此调用者中，我们必须提到`@TokenRequired`，因为它将调用适当的 AOP 方法。此外，在此 AOP 方法中，我们不需要提到`execution`，就像我们在以前的`execution(* com.packtpub.restapp.HomeController.testAOPWithoutAnnotation())`方法中所做的那样。

# 测试 AOP @Before 注释

只需在浏览器中或使用任何其他 REST 客户端调用 API（`http://localhost:8080/test/aop/with/annotation`）;然后，您应该在控制台上看到以下内容：

```java
Before tokenRequiredWithAnnotation
```

# 将 AOP 与 JWT 集成

假设您想要在`UserContoller`方法中限制`deleteUser`选项。删除用户的人应该具有适当的 JWT 令牌。如果他们没有令牌，我们将不允许他们删除任何用户。在这里，我们将首先有一个`packt`主题来创建一个令牌。

可以调用`http://localhost:8080/security/generate/token?subject=packt`生成令牌的 API。

当我们在主题中使用`packt`时，它将生成`eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJwYWNrdCIsImV4cCI6MTUwOTk0NzY2Mn0.hIsVggbam0pRoLOnSe8L9GQS4IFfFklborwJVthsmz0`令牌。

现在，我们将不得不创建一个 AOP 方法，通过要求用户在`delete`调用的标头中具有令牌来限制用户：

```java
@Before("@annotation(tokenRequired)")
public void tokenRequiredWithAnnotation(TokenRequired tokenRequired) throws Throwable{   
       ServletRequestAttributes reqAttributes = (ServletRequestAttributes)RequestContextHolder.currentRequestAttributes();
       HttpServletRequest request = reqAttributes.getRequest();    
       // checks for token in request header
       String tokenInHeader = request.getHeader("token");    
       if(StringUtils.isEmpty(tokenInHeader)){
              throw new IllegalArgumentException("Empty token");
           }    
       Claims claims = Jwts.parser() .setSigningKey(DatatypeConverter.parseBase64Binary(SecurityServiceImpl.secretKey))
       .parseClaimsJws(tokenInHeader).getBody();    
       if(claims == null || claims.getSubject() == null){
                throw new IllegalArgumentException("Token Error : Claim is null");
             }    
       if(!claims.getSubject().equalsIgnoreCase("packt")){
                throw new IllegalArgumentExceptionception("Subject doesn't match in the token");
          }
       }
```

从前面的代码中可以看到 AOP 中的 JWT 集成。是的，我们已经将 JWT 令牌验证部分与 AOP 集成。因此，以后，如果有人调用`@TokenRequired`注释的 API，它将首先到达 AOP 方法并检查令牌匹配。如果令牌为空，不匹配或过期，我们将收到错误。所有可能的错误将如下所述。

现在，我们可以在`UserController`类中的 API 调用中开始使用`@TokenRequired`注释。因此，每当调用此`deleteUser`方法时，它将在执行 API 方法本身之前转到`JWT`，检查切入点。通过这样做，我们可以确保`deleteUser`方法不会在没有令牌的情况下被调用。

`UserController`类的代码如下：

```java
  @ResponseBody
  @TokenRequired
  @RequestMapping(value = "", method = RequestMethod.DELETE)
  public Map<String, Object> deleteUser(
      @RequestParam(value="userid") Integer userid){
    Map<String, Object> map = new LinkedHashMap<>();   
    userSevice.deleteUser(userid);   
    map.put("result", "deleted");
    return map;
  }
```

如果令牌为空或为空，它将抛出以下错误：

```java
{
   "timestamp": 1509949209993,
   "status": 500,
   "error": "Internal Server Error",
   "exception": "java.lang.reflect.UndeclaredThrowableException",
   "message": "No message available",
   "path": "/user"
}
```

如果令牌匹配，它将显示结果而不抛出任何错误。您将看到以下结果：

```java
{
    "result": "deleted"
} 
```

如果我们在标头中不提供任何令牌，可能会抛出以下错误：

```java
{
   "timestamp": 1509948248281,
   "status": 500,
   "error": "Internal Server Error",
   "exception": "java.lang.IllegalArgumentException",
   "message": "JWT String argument cannot be null or empty.",
   "path": "/user"
}
```

如果令牌过期，您将收到以下错误：

```java
 {
   "timestamp": 1509947985415,
   "status": 500,
   "error": "Internal Server Error",
   "exception": "io.jsonwebtoken.ExpiredJwtException",
   "message": "JWT expired at 2017-11-06T00:54:22-0500\. Current time: 2017-11-06T00:59:45-0500",
   "path": "/test/aop/with/annotation"
} 
```

# 日志记录控制

日志记录在需要跟踪特定过程的输出时非常有用。当我们在服务器上部署应用程序后，它将帮助我们验证过程或找出错误的根本原因。如果没有记录器，将很难跟踪和找出问题。

在我们的应用程序中，有许多日志记录框架可以使用；Log4j 和 Logback 是大多数应用程序中使用的两个主要框架。

# SLF4J，Log4J 和 Logback

SLF4j 是一个 API，帮助我们在部署过程中选择 Log4j 或 Logback 或任何其他 JDK 日志。SLF4j 只是一个抽象层，为使用我们的日志 API 的用户提供自由。如果有人想在他们的实现中使用 JDK 日志或 Log4j，SLF4j 将帮助他们在运行时插入所需的框架。

如果我们创建的最终产品不能被他人用作库，我们可以直接实现 Log4j 或 Logback。但是，如果我们有一个可以用作库的代码，最好选择 SLF4j，这样用户可以遵循他们想要的任何日志记录。

Logback 是 Log4j 的更好替代品，并为 SLF4j 提供本地支持。

# Logback 框架

我们之前提到 Logback 比 Log4j 更可取；在这里我们将讨论如何实现 Logback 日志框架。

Logback 有三个模块：

1.  `logback-core`：基本日志

1.  `logback-classic`：改进的日志记录和 SLF4j 支持

1.  `logback-access`：Servlet 容器支持

`logback-core`模块是 Log4j 框架中其他两个模块的基础。`logback-classic`模块是 Log4j 的改进版本，具有更多功能。此外，`logback-classic`模块本地实现了 SLF4j API。由于这种本地支持，我们可以切换到不同的日志框架，如**Java Util Logging**（**JUL**）和 Log4j。

`logback-access`模块为 Tomcat/Jetty 等 Servlet 容器提供支持，特别是提供 HTTP 访问日志功能。

# Logback 依赖和配置

为了在我们的应用程序中使用 Logback，我们需要`logback-classic`依赖项。但是，`logback-classic`依赖项已经包含在`spring-boot-starter`依赖项中。我们可以在项目文件夹中使用依赖树（`mvn dependency:tree`）来检查：

```java
mvn dependency:tree
```

在项目文件夹中检查依赖树时，我们将获得所有依赖项的完整树。以下是我们可以看到`spring-boot-starter`依赖项下的`logback-classic`依赖项的部分：

```java
[INFO] | +- org.springframework.boot:spring-boot-starter:jar:1.5.7.RELEASE:compile
[INFO] | +- org.springframework.boot:spring-boot:jar:1.5.7.RELEASE:compile
[INFO] | +- org.springframework.boot:spring-boot-autoconfigure:jar:1.5.7.RELEASE:compile
[INFO] | +- org.springframework.boot:spring-boot-starter-logging:jar:1.5.7.RELEASE:compile
[INFO] | | +- ch.qos.logback:logback-classic:jar:1.1.11:compile
[INFO] | | | \- ch.qos.logback:logback-core:jar:1.1.11:compile
[INFO] | | +- org.slf4j:jcl-over-slf4j:jar:1.7.25:compile
[INFO] | | +- org.slf4j:jul-to-slf4j:jar:1.7.25:compile
[INFO] | | \- org.slf4j:log4j-over-slf4j:jar:1.7.25:compile
[INFO] | \- org.yaml:snakeyaml:jar:1.17:runtime
[INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2
```

由于必要的依赖文件已经可用，我们不需要为 Logback 框架实现添加任何依赖项。

# 日志级别

由于 SLF4j 定义了这些日志级别，实现 SLF4j 的人应该适应 SFL4j 的日志级别。日志级别如下：

+   `TRACE`：详细评论，在所有情况下可能不会使用

+   `DEBUG`：用于生产环境中调试目的的有用评论

+   `INFO`：在开发过程中可能有帮助的一般评论

+   `WARN`：在特定场景下可能有帮助的警告消息，例如弃用的方法

+   `ERROR`：开发人员需要注意的严重错误消息

让我们将日志配置添加到`application.properties`文件中：

```java
# spring framework logging 
logging.level.org.springframework = ERROR

# local application logging
logging.level.com.packtpub.restapp = INFO
```

在前面的配置中，我们已经为 Spring Framework 和我们的应用程序使用了日志配置。根据我们的配置，它将为 Spring Framework 打印`ERROR`，为我们的应用程序打印`INFO`。

# 类中的 Logback 实现

让我们给类添加一个`Logger`；在我们的情况下，我们可以使用`UserController`。我们必须导入`org.slf4j.Logger`和`org.slf4j.LoggerFactory`。我们可以检查以下代码：

```java
private static final Logger _logger = LoggerFactory.getLogger(HomeController.class);
```

在前面的代码中，我们介绍了`_logger`实例。我们使用`UserController`类作为`_logger`实例的参数。

现在，我们必须使用`_logger`实例来打印我们想要的消息。在这里，我们使用了`_logger.info()`来打印消息：

```java
package com.packtpub.restapp;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
// other imports
@RestController
@RequestMapping("/")
public class HomeController {  
  private static final Logger _logger = LoggerFactory.getLogger(HomeController.class);  
  @Autowired
  SecurityService securityService;  
  @ResponseBody
  @RequestMapping("")
  public Map<String, Object> test() {
    Map<String, Object> map = new LinkedHashMap<>();
    map.put("result", "Aloha");    
    _logger.trace("{test} trace");
    _logger.debug("{test} debug");
    _logger.info("{test} info");
    _logger.warn("{test} warn ");
    _logger.error("{test} error");    
    return map;
  }
```

在前面的代码中，我们使用了各种记录器来打印消息。当您重新启动服务器并调用`http://localhost:8080` REST API 时，您将在控制台中看到以下输出：

```java
2018-01-15 16:29:55.951 INFO 17812 --- [nio-8080-exec-1] com.packtpub.restapp.HomeController : {test} info
2018-01-15 16:29:55.951 WARN 17812 --- [nio-8080-exec-1] com.packtpub.restapp.HomeController : {test} warn 
2018-01-15 16:29:55.951 ERROR 17812 --- [nio-8080-exec-1] com.packtpub.restapp.HomeController : {test} error
```

正如您从日志中看到的，类名将始终在日志中以标识日志中的特定类。由于我们没有提及任何日志模式，记录器采用默认模式打印输出与类一起。如果需要，我们可以在配置文件中更改模式以获得定制日志。

在先前的代码中，我们使用了不同的日志级别来打印消息。对日志级别有限制，因此根据业务需求和实现，我们将不得不配置我们的日志级别。

在我们的日志配置中，我们只使用了控制台打印选项。我们还可以提供一个选项，将日志打印到我们想要的外部文件中。

# 总结

在本章中，我们涵盖了 Spring AOP 和日志控制的实现。在我们现有的代码中，我们介绍了 Spring AOP，并演示了 AOP 如何通过代码重用节省时间。为了让用户理解 AOP，我们简化了 AOP 的实现。在下一章中，我们将讨论如何构建一个 REST 客户端，并更多地讨论 Spring 中的错误处理。
