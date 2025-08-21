# 第十章：构建 REST 客户端和错误处理

在之前的章节中，我们涵盖了 RESTful Web 服务的服务器端，包括 CRUD 操作。在这里，我们可以检查如何在代码中消费这些 API。REST 客户端将帮助我们实现这个目标。

在本章中，我们将讨论以下主题：

+   Spring 中的 RestTemplate

+   使用 Spring 构建 RESTful 服务客户端的基本设置

+   在客户端调用 RESTful 服务

+   定义错误处理程序

+   使用错误处理程序

# 构建 REST 客户端

到目前为止，我们已经创建了一个 REST API，并在诸如 SoapUI、Postman 或 JUnit 测试之类的第三方工具中使用它。可能会出现情况，您将不得不使用常规方法（服务或另一个控制器方法）本身来消费 REST API，比如在服务 API 中调用支付 API。当您在代码中调用第三方 API，比如 PayPal 或天气 API 时，拥有一个 REST 客户端将有助于完成工作。

在这里，我们将讨论如何构建一个 REST 客户端来在我们的方法中消费另一个 REST API。在进行这之前，我们将简要讨论一下 Spring 中的`RestTemplate`。

# RestTemplate

`RestTemplate`是一个 Spring 类，用于通过 HTTP 从客户端消费 REST API。通过使用`RestTemplate`，我们可以将 REST API 消费者保持在同一个应用程序中，因此我们不需要第三方应用程序或另一个应用程序来消费我们的 API。`RestTemplate`可以用于调用`GET`、`POST`、`PUT`、`DELETE`和其他高级 HTTP 方法（`OPTIONS`、`HEAD`）。

默认情况下，`RestTemplate`类依赖 JDK 建立 HTTP 连接。您可以切换到使用不同的 HTTP 库，如 Apache HttpComponents 和 Netty。

首先，我们将在`AppConfig`类中添加一个`RestTemplate` bean 配置。在下面的代码中，我们将看到如何配置`RestTemplate` bean：

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;
@Configuration
public class AppConfig {
  @Bean
  public RestTemplate restTemplate() {
      return new RestTemplate();
  }
}
```

在上面的代码中，我们已经在这个类中使用了`@Configuration`注解来配置类中的所有 bean。我们还在这个类中引入了`RestTemplate` bean。通过在`AppConfig`类中配置 bean，我们告诉应用程序所述的 bean 可以在应用程序的任何地方使用。当应用程序启动时，它会自动初始化 bean，并准备在需要的地方使用模板。

现在，我们可以通过在任何类中简单地使用`@Autowire`注解来使用`RestTemplate`。为了更好地理解，我们创建了一个名为`ClientController`的新类，并在该类中添加了一个简单的方法：

```java
@RestController
@RequestMapping("/client")
public class ClientController {  
  private final Logger _log = LoggerFactory.getLogger(this.getClass());    
  @Autowired
  RestTemplate template;  
  @ResponseBody
  @RequestMapping("/test") 
  public Map<String, Object> test(){
    Map<String, Object> map = new LinkedHashMap<>();
    String content = template.getForObject("http://localhost:8080/", String.class); 
    map.put("result", content);    
    return map;
  }  
}
```

在上面的代码中，我们使用了`RestTemplate`并调用了`getForObject`方法来消费 API。默认情况下，我们使用`String.class`来使我们的代码简单易懂。

当您调用这个 API `http://localhost:8080/client/test/`时，您将得到以下结果：

```java
{
  result: "{\"result\":"\Aloha\"}"
}
```

在上述过程中，我们在另一个 REST API 中使用了`RestTemplate`。在实时场景中，您可能会使用与调用第三方 REST API 相同的方法。

让我们在另一个方法中获取一个单个用户 API：

```java
@ResponseBody
  @RequestMapping("/test/user") 
  public Map<String, Object> testGetUser(){
    Map<String, Object> map = new LinkedHashMap<>();
    User user = template.getForObject("http://localhost:8080/user/100", User.class); 
    map.put("result", user);    
    return map;
  }
```

通过调用上述 API，您将得到单个用户作为结果。为了调用这个 API，我们的`User`类应该被序列化，否则您可能会得到一个未序列化对象错误。让我们通过实现`Serializable`并添加一个序列版本 ID 来使我们的`User`类序列化。

您可以通过在 Eclipse 中右键单击类名并生成一个序列号来创建一个序列版本 ID。

在对`User`类进行序列化之后，它将如下所示：

```java
public class User implements Serializable {  
  private static final long serialVersionUID = 3453281303625368221L;  
  public User(){ 
  }
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
  @Override
  public String toString() {
    return "User [userid=" + userid + ", username=" + username + "]";
  }
}
```

最后，我们可以在浏览器中调用`http://localhost:8080/client/test/user`客户端 API，并得到以下结果：

```java
{
  result: {
    userid: 100,
    username: "David"
  }
}
```

为了便于理解，我们只使用了`GET`方法。然而，我们可以使用`POST`方法并在 REST 消费者中添加参数。

# 错误处理

到目前为止，在我们的应用程序中，我们还没有定义任何特定的错误处理程序来捕获错误并将其传达到正确的格式。通常，当我们在 REST API 中处理意外情况时，它会自动抛出 HTTP 错误，如`404`。诸如`404`之类的错误将在浏览器中明确显示。这通常是可以接受的；但是，无论事情是对是错，我们可能需要一个 JSON 格式的结果。

在这种情况下，将错误转换为 JSON 格式是一个不错的主意。通过提供 JSON 格式，我们可以保持我们的应用程序干净和标准化。

在这里，我们将讨论如何在事情出错时管理错误并以 JSON 格式显示它们。让我们创建一个通用的错误处理程序类来管理我们所有的错误：

```java
public class ErrorHandler {
  @ExceptionHandler(Exception.class)
  public @ResponseBody <T> T handleException(Exception ex) {    
    Map<String, Object> errorMap = new LinkedHashMap<>();
    if(ex instanceof org.springframework.web.bind.MissingServletRequestParameterException){      
      errorMap.put("Parameter Missing", ex.getMessage());
      return (T) errorMap;
    }    
    errorMap.put("Generic Error ", ex.getMessage());
    return (T) errorMap;
  }
}
```

上面的类将作为我们应用程序中的通用错误处理程序。在`ErrorHandler`类中，我们创建了一个名为`handleException`的方法，并使用`@ExceptionHandler`注解。此注解将使该方法接收应用程序中的所有异常。一旦我们获得异常，我们可以根据异常的类型来管理应该做什么。

在我们的代码中，我们只使用了两种情况来管理我们的异常：

+   缺少参数

+   一般错误（除了缺少参数之外的所有其他情况）

如果在调用任何 REST API 时缺少参数，它将进入第一种情况，“参数缺失”，否则它将进入“通用错误”默认错误。我们简化了这个过程，以便新用户能够理解。但是，我们可以在这种方法中添加更多情况来处理更多的异常。

完成错误处理程序后，我们将不得不在我们的应用程序中使用它。应用错误处理程序可以通过多种方式完成。扩展错误处理程序是使用它的最简单方法：

```java
@RestController
@RequestMapping("/")
public class HomeController extends ErrorHandler {    
    // other methods
  @ResponseBody
  @RequestMapping("/test/error") 
  public Map<String, Object> testError(@RequestParam(value="item") String item){
    Map<String, Object> map = new LinkedHashMap<>();
    map.put("item", item);    
    return map;
  }   
}
```

在上面的代码中，我们只是在`HomeController`类中扩展了`ErrorHandler`。通过这样做，我们将所有错误情况绑定到`ErrorHandler`以正确接收和处理。此外，我们创建了一个名为`testError`的测试方法来检查我们的错误处理程序。

为了调用这个 API，我们需要将`item`作为参数传递；否则它将在应用程序中抛出一个错误。因为我们已经定义了`ErrorController`类并扩展了`HomeController`类，缺少参数将使您进入前面提到的第一个情景。

只需在浏览器或任何 REST 客户端（Postman/SoapUI）中尝试以下 URL：`http://localhost:8080/test/error`。

如果您尝试上述端点，您将得到以下结果：

```java
{
  Parameter Missing: "Required String parameter 'item' is not present"
}
```

由于我们在错误处理程序中定义了 JSON 格式，如果任何 REST API 抛出异常，我们将以 JSON 格式获得错误。

# 自定义异常

到目前为止，我们只探讨了应用程序引发的错误。但是，如果需要，我们可以定义自己的错误并抛出它们。以下代码将向您展示如何在我们的应用程序中创建自定义错误并抛出它：

```java
@RestController
@RequestMapping("/")
public class HomeController extends ErrorHandler {  
    // other methods  
  @ResponseBody
  @RequestMapping("/test/error/{id}")
  public Map<String, Object> testRuntimeError(@PathVariable("id") Integer id){    
    if(id == 1){
      throw new RuntimeException("some exception");
    }    
    Map<String, Object> map = new LinkedHashMap<>();
    map.put("result", "one");    
    return map;
  }
}
```

在上面的代码中，我们使用`RuntimeException`创建了一个自定义异常。这只是测试代码，向您展示自定义异常在错误处理中的工作原理。我们将在接下来的章节中在我们的应用程序中应用这个自定义异常。

如果您调用`http://localhost:8080/test/error/1`API，您将得到以下错误，这是由我们的条件匹配引起的：

```java
{
  Generic Error : "some exception"
}
```

# 摘要

在本章中，我们学习了如何使用`RestTemplate`构建 RESTful Web 服务客户端。此外，我们还涵盖了错误处理程序和集中式错误处理程序来处理所有容易出错的情况。在接下来的章节中，我们将讨论如何扩展我们的 Spring 应用程序，并简要讨论微服务，因为这些主题正在迅速增长。
