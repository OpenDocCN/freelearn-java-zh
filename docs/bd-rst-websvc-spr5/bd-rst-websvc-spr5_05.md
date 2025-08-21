# 第五章：普通 REST 中的 CRUD 操作（不包括 Reactive）和文件上传

在上一章中，我们探讨了对 Reactive 支持的 CRUD 操作。由于 Spring 开发团队仍在更新更多的 Reactive 实体，Reactive 支持还没有达到他们的水平。尽管 Spring 5 的 Reactive 支持运行良好，但他们仍需要改进以使其更加稳定。考虑到这些要点，我们计划避免使用 Reactive 支持，以使其对您更加简单。

在本章中，我们将介绍 Spring 5（不包括 Reactive）REST 中的基本 CRUD（创建、读取、更新和删除）API。在本章之后，您将能够在 Spring 5 中进行简单的 CRUD 操作，而无需 Reactive 支持。此外，我们将讨论 Spring 5 中的文件上传选项。

在本章中，我们将涵盖以下方法：

+   将 CRUD 操作映射到 HTTP 方法

+   创建用户

+   更新用户

+   删除用户

+   读取（选择）用户

+   Spring 中的文件上传

# 将 CRUD 操作映射到 HTTP 方法

在上一章中，您看到了控制器中的 CRUD 操作。在本章中，我们将进行相同的 CRUD 操作；但是，我们已经排除了所有 Reactive 组件。

# 创建资源

要创建基本的 Spring 项目资源，您可以使用 Spring Initializr（[`start.spring.io/`](https://start.spring.io/)）。在 Spring Initializr 中，提供必要的详细信息：

使用 Java 和 Spring Boot 1.5.9 生成一个 Maven 项目。

组：`com.packtpub.restapp`

Artifact：`ticket-management`

搜索依赖项：选择`Web（使用 Tomcat 和 Web MVC 进行全栈 Web 开发）`依赖项

填写完详细信息后，只需点击`Generate Project`；然后它将以 ZIP 格式创建 Spring 基本资源。我们可以通过将它们导入 Eclipse 来开始使用项目。

Spring 5 的 POM 文件将如下所示：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project  
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.packtpub.restapp</groupId>
  <artifactId>ticket-management</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>
  <name>ticket-management</name>
  <description>Demo project for Spring Boot</description>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

让我们移除父级以简化 POM：

```java
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>
```

由于我们移除了父级，我们可能需要在所有依赖项中添加版本。让我们在我们的依赖项中添加版本：

```java
<dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <version>1.5.9.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
      <version>1.5.9.RELEASE</version>
    </dependency>
  </dependencies>
```

由于依赖项 artifact `spring-boot-starter-web`版本`1.5.9`基于 Spring 4.3.11，我们将不得不升级到 Spring 5。让我们清理并升级我们的 POM 文件以引入 Spring 5 更新：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project  
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.packtpub.restapp</groupId>
  <artifactId>ticket-management</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>
  <name>ticket-management</name>
  <description>Demo project for Spring Boot</description> 
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <version>1.5.9.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
      <version>1.5.9.RELEASE</version>
    </dependency>
  </dependencies>
  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

您可以在上述 POM 文件中看到与 Spring 5 相关的依赖项。让我们使用 REST 端点对它们进行测试。首先，创建一个 Spring Boot 主文件来初始化 Spring Boot：

```java
@SpringBootApplication
public class TicketManagementApplication {  
  public static void main(String[] args) {
    SpringApplication.run(TicketManagementApplication.class, args);
  }
}
```

您可以通过右键单击项目并选择`Run As | Spring Boot App`在 Eclipse 上运行 Spring Boot。如果这样做，您将在 Eclipse 控制台中看到日志。

如果您看不到控制台，可以通过`Window | Show View | Console`获取它。

以下是一个示例日志。您可能看不到完全匹配；但是，您将了解服务器运行日志的外观：

```java

 . ____ _ __ _ _
 /\\ / ___'_ __ _ _(_)_ __ __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/ ___)| |_)| | | | | || (_| | ) ) ) )
 ' |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot :: (v1.5.7.RELEASE)

2017-11-05 15:49:21.380 INFO 8668 --- [ main] c.p.restapp.TicketManagementApplication : Starting TicketManagementApplication on DESKTOP-6JP2FNB with PID 8668 (C:\d\spring-book-sts-space\ticket-management\target\classes started by infoadmin in C:\d\spring-book-sts-space\ticket-management)
2017-11-05 15:49:21.382 INFO 8668 --- [ main] c.p.restapp.TicketManagementApplication : No active profile set, falling back to default profiles: default
2017-11-05 15:49:21.421 INFO 8668 --- [ main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@5ea434c8: startup date [Sun Nov 05 15:49:21 EST 2017]; root of context hierarchy
2017-11-05 15:49:22.205 INFO 8668 --- [ main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat initialized with port(s): 8080 (http)
2017-11-05 15:49:22.213 INFO 8668 --- [ main] o.apache.catalina.core.StandardService : Starting service [Tomcat]
...
..

...
...
2017-11-05 15:49:22.834 INFO 8668 --- [ main] o.s.j.e.a.AnnotationMBeanExporter : Registering beans for JMX exposure on startup
2017-11-05 15:49:22.881 INFO 8668 --- [ main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
```

您应该在日志的最后几行看到`Tomcat started on port(s): 8080`。

当您检查 URI `http://localhost:8080` 时，您将看到以下错误：

```java
Whitelabel Error Page

This application has no explicit mapping for /error, so you are seeing this as a fallback.

Sun Nov {current date}
There was an unexpected error (type=Not Found, status=404).
No message available
```

先前的错误是说应用程序中没有配置相应的 URI。让我们通过在`com.packtpub.restapp`包下创建一个名为`HomeController`的控制器来解决这个问题：

```java
package com.packtpub.restapp;
import java.util.LinkedHashMap;
import java.util.Map;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;
@RestController
@RequestMapping("/")
public class HomeController {
  @ResponseBody
  @RequestMapping("")
  public Map<String, Object> test(){
    Map<String, Object> map = new LinkedHashMap<>();
    map.put("result", "Aloha");    
    return map;
  }
}
```

在上述代码中，我们创建了一个名为`HomeController`的虚拟控制器，并将简单的`map`作为结果。此外，我们添加了新的控制器，我们需要让我们的主应用程序自动扫描这些类，在我们的情况下是`TicketManagementApplication`类。我们将通过在主类中添加`@ComponentScan("com.packtpub")`来告诉它们。最后，我们的主类将如下所示：

```java
package com.packtpub.restapp.ticketmanagement;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ComponentScan;
@ComponentScan("com.packtpub")
@SpringBootApplication
public class TicketManagementApplication {
  public static void main(String[] args) {
    SpringApplication.run(TicketManagementApplication.class, args);
  }
}
```

当您重新启动 Spring Boot 应用程序时，您将看到 REST 端点正在工作（`localhost:8080`）：

```java
{
  result: "Aloha"
}
```

# Spring 5 中的 CRUD 操作（不包括 Reactive）

让我们执行用户 CRUD 操作。由于我们之前已经讨论了 CRUD 概念，因此在这里我们只讨论 Spring 5 上的用户管理（不包括 Reactive 支持）。让我们为 CRUD 端点填充所有虚拟方法。在这里，我们可以创建`UserContoller`并填充所有 CRUD 用户操作的方法：

```java
package com.packtpub.restapp;
import java.util.LinkedHashMap;
import java.util.Map;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;
@RestController
@RequestMapping("/user")
public class UserController {  
  @ResponseBody
  @RequestMapping("")
  public Map<String, Object> getAllUsers(){
    Map<String, Object> map = new LinkedHashMap<>();
    map.put("result", "Get All Users Implementation");    
    return map;
  }  
  @ResponseBody
  @RequestMapping("/{id}")
  public Map<String, Object> getUser(@PathVariable("id") Integer id){
    Map<String, Object> map = new LinkedHashMap<>();
    map.put("result", "Get User Implementation");    
    return map;
  } 
  @ResponseBody
  @RequestMapping(value = "", method = RequestMethod.POST)
  public Map<String, Object> createUser(){
    Map<String, Object> map = new LinkedHashMap<>();
    map.put("result", "Create User Implementation");    
    return map;
  }  
  @ResponseBody
  @RequestMapping(value = "", method = RequestMethod.PUT)
  public Map<String, Object> updateUser(){
    Map<String, Object> map = new LinkedHashMap<>();
    map.put("result", "Update User Implementation");    
    return map;
  }
  @ResponseBody
  @RequestMapping(value = "", method = RequestMethod.DELETE)
  public Map<String, Object> deleteUser(){
    Map<String, Object> map = new LinkedHashMap<>();
    map.put("result", "Delete User Implementation");    
    return map;
  }
}
```

我们已经为所有 CRUD 操作填充了基本端点。如果您在 Postman 上调用它们，并使用适当的方法，如`GET`，`POST`，`PUT`和`DELETE`，您将看到提到适当消息的结果。

例如，对于`getAllUsers` API（`localhost:8080/user`），您将获得：

```java
{
  result: "Get All Users Implementation"
}
```

# getAllUsers - 实现

让我们实现`getAllUsers` API。对于这个 API，我们可能需要在`com.packtpub.model`包下创建一个名为`User`的模型类：

```java
package com.packtpub.model;
public class User {
  private Integer userid;  
  private String username;   
  public User(Integer userid, String username){
    this.userid = userid;
    this.username = username;
  }  
  // getter and setter methods 
}
```

现在，我们将添加`getAllUsers`实现的代码。由于这是业务逻辑，我们将创建一个单独的`UserService`和`UserServiceImpl`类。通过这样做，我们可以将业务逻辑放在不同的地方，以避免代码复杂性。

`UserService`接口如下所示：

```java
package com.packtpub.service;
import java.util.List;
import com.packtpub.model.User;
public interface UserService {
  List<User> getAllUsers();
}
```

`UserServiceImpl`类的实现如下：

```java
package com.packtpub.service;
import java.util.LinkedList;
import java.util.List;
import org.springframework.stereotype.Service;
import com.packtpub.model.User;
@Service
public class UserServiceImpl implements UserService {
  @Override
  public List<User> getAllUsers() {    
    return this.users;
  }  
  // Dummy users
  public static List<User> users; 
  public UserServiceImpl() {
    users = new LinkedList<>();   
    users.add(new User(100, "David"));
    users.add(new User(101, "Peter"));
    users.add(new User(102, "John"));
  }
}
```

在前面的实现中，我们在构造函数中创建了虚拟用户。当类由 Spring 配置初始化时，这些用户将被添加到列表中。

调用`getAllUsers`方法的`UserController`类如下：

```java
@Autowired
UserService userSevice;
@ResponseBody
@RequestMapping("")
public List<User> getAllUsers(){
    return userSevice.getAllUsers();
}
```

在前面的代码中，我们通过在控制器文件中进行自动装配来调用`getAllUsers`方法。`@Autowired`将在幕后执行所有实例化魔术。

如果您现在运行应用程序，可能会遇到以下错误：

```java
***************************
APPLICATION FAILED TO START
***************************

Description:

Field userSevice in com.packtpub.restapp.UserController required a bean of type 'com.packtpub.service.UserService' that could not be found.

Action:

Consider defining a bean of type 'com.packtpub.service.UserService' in your configuration.
```

这个错误的原因是您的应用程序无法识别`UserService`，因为它在不同的包中。我们可以通过在`TicketManagementApplication`类中添加`@ComponentScan("com.packtpub")`来解决这个问题。这将识别不同子包中的所有`@service`和其他 bean：

```java
@ComponentScan("com.packtpub")
@SpringBootApplication
public class TicketManagementApplication {  
  public static void main(String[] args) {
    SpringApplication.run(TicketManagementApplication.class, args);
  }
}
```

现在您可以在调用 API（`http://localhost:8080/user`）时看到结果：

```java
[
  {
    userid: 100,
    username: "David"
  },
  {
    userid: 101,
    username: "Peter"
  },
  {
    userid: 102,
    username: "John"
  }
]
```

# getUser - 实现

就像我们在第四章中所做的那样，*Spring REST 中的 CRUD 操作*，我们将在本节中实现`getUser`业务逻辑。让我们使用 Java 8 Streams 在这里添加`getUser`方法。

`UserService`接口如下所示：

```java
User getUser(Integer userid);
```

`UserServiceImpl`类的实现如下：

```java
@Override
public User getUser(Integer userid) {     
    return users.stream()
    .filter(x -> x.getUserid() == userid)
    .findAny()
    .orElse(new User(0, "Not Available")); 
}
```

在之前的`getUser`方法实现中，我们使用了 Java 8 Streams 和 lambda 表达式来通过`userid`获取用户。与传统的`for`循环不同，lambda 表达式使得获取详细信息更加容易。在前面的代码中，我们通过过滤条件检查用户。如果用户匹配，它将返回特定用户；否则，它将创建一个带有`"Not available"`消息的虚拟用户。

`getUser`方法的`UserController`类如下：

```java
@ResponseBody
@RequestMapping("/{id}")
public User getUser(@PathVariable("id") Integer id){  
  return userSevice.getUser(100);
}
```

您可以通过访问客户端中的`http://localhost:8080/user/100`来验证 API（使用 Postman 或 SoapUI 进行测试）：

```java
{
  userid: 100,
  username: "David"
}
```

# createUser - 实现

现在我们可以添加创建用户选项的代码。

`UserService`接口如下所示：

```java
void createUser(Integer userid, String username);
```

`UserServiceImpl`类的实现如下：

```java
@Override
public void createUser(Integer userid, String username) {    
    User user = new User(userid, username); 
    this.users.add(user); 
}
```

`createUser`方法的`UserController`类如下：

```java
@ResponseBody
  @RequestMapping(value = "", method = RequestMethod.POST)
  public Map<String, Object> createUser(
    @RequestParam(value="userid") Integer userid,
    @RequestParam(value="username") String username
    ){    
    Map<String, Object> map = new LinkedHashMap<>(); 
    userSevice.createUser(userid, username);    
    map.put("result", "added");
    return map;
}
```

前面的代码将在我们的映射中添加用户。在这里，我们使用`userid`和`username`作为方法参数。您可以在以下 API 调用中查看`userid`和`username`：

![](img/7eeaca9a-ea63-487a-94ef-2274187f0065.png)

当您使用 SoapUI/Postman 调用此方法时，您将获得以下结果。在这种情况下，我们使用参数（`userid`，`username`）而不是 JSON 输入。这只是为了简化流程：

```java
{"result": "added"}
```

# updateUser - 实现

现在我们可以添加更新用户选项的代码。

`UserService`接口如下所示：

```java
void updateUser(Integer userid, String username);
```

`UserServiceImpl`类的实现如下：

```java
@Override
public void updateUser(Integer userid, String username) {
    users.stream()
        .filter(x -> x.getUserid() == userid)
        .findAny()
        .orElseThrow(() -> new RuntimeException("Item not found"))
        .setUsername(username); 
}
```

在前面的方法中，我们使用了基于 Java Streams 的实现来更新用户。我们只需应用过滤器并检查用户是否可用。如果`userid`不匹配，它将抛出`RuntimeException`。如果用户可用，我们将获得相应的用户，然后更新`username`。

`updateUser`方法的`UserController`类如下：

```java
@ResponseBody
  @RequestMapping(value = "", method = RequestMethod.PUT)
  public Map<String, Object> updateUser(
      @RequestParam(value="userid") Integer userid,
      @RequestParam(value="username") String username
    ){
    Map<String, Object> map = new LinkedHashMap<>();
    userSevice.updateUser(userid, username);    
    map.put("result", "updated");    
    return map;
  }
```

我们将尝试将`userid`为`100`的`username`从`David`更新为`Sammy`。我们可以从以下截图中查看 API 的详细信息：

![](img/af20ba48-60bf-42c3-901f-8b2fa59d37d5.png)

当我们使用 SoapUI/Postman 扩展（`http://localhost:8080/user`）调用此 API（`UPDATE`方法）时，我们将得到以下结果：

```java
{"result": "updated"}
```

您可以通过在 Postman 扩展中检查`getAllUsers` API（`GET`方法）（`http://localhost:8080/user`）来检查结果；您将得到以下结果：

```java
[
  {
    "userid": 100,
    "username": "Sammy"
  },
  {
    "userid": 101,
    "username": "Peter"
  },
  {
    "userid": 102,
    "username": "John"
  },
  {
    "userid": 104,
    "username": "Kevin"
  }
]
```

# deleteUser - 实现

现在我们可以添加`deleteUser`选项的代码。

`UserService`接口如下所示：

```java
void deleteUser(Integer userid);
```

`UserServiceImpl`类的实现如下：

```java
@Override
public void deleteUser(Integer userid) { 

   users.removeIf((User u) -> u.getUserid() == userid);

}
```

`UserController`类的`deleteUser`方法如下所示：

```java
@ResponseBody
@RequestMapping(value = "/{id}", method = RequestMethod.DELETE)
public Map<String, Object> deleteUser(
      @PathVariable("id") Integer userid) {
    Map<String, Object> map = new LinkedHashMap<>(); 
      userSevice.deleteUser(userid); 
      map.put("result", "deleted");
      return map;
}
```

当您使用 Postman 扩展调用此 API（`DELETE`方法）（`http://localhost:8080/user/100`）时，您将得到以下结果：

```java
{"result": "deleted"}
```

您还可以检查`getAllUsers`方法，以验证您是否已删除用户。

# 文件上传 - REST API

在支持`NIO`库和 Spring 的`MultipartFile`选项的支持下，文件上传变得非常容易。在这里，我们将添加文件上传的代码。

`FileUploadService`接口如下所示：

```java
package com.packtpub.service;
import org.springframework.web.multipart.MultipartFile;
public interface FileUploadService {
  void uploadFile(MultipartFile file) throws IOException;
}
```

在上述代码中，我们只是定义了一个方法，让具体类（实现类）覆盖我们的方法。我们在这里使用`MultipartFile`来传递文件，例如媒体文件，以满足我们的业务逻辑。

`FileUploadServerImpl`类的实现如下：

```java
package com.packtpub.service;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;
import org.springframework.stereotype.Service;
import org.springframework.util.StringUtils;
import org.springframework.web.multipart.MultipartFile;
@Service
public class FileUploadServerImpl implements FileUploadService {
  private Path location;  
  public FileUploadServerImpl() throws IOException {
    location = Paths.get("c:/test/");
    Files.createDirectories(location);
  }
  @Override
  public void uploadFile(MultipartFile file) throws IOException {
    String fileName = StringUtils.cleanPath(file.getOriginalFilename());
    if (fileName.isEmpty()) {
      throw new IOException("File is empty " + fileName);
    } try {
      Files.copy(file.getInputStream(), 
            this.location.resolve(fileName),     
            StandardCopyOption.REPLACE_EXISTING);
    } catch (IOException e) {
      throw new IOException("File Upload Error : " + fileName);
    }
  }
}
```

在上述代码中，我们在构造函数中设置了位置，因此当 Spring Boot App 初始化时，它将设置正确的路径；如果需要，它将在指定位置创建一个特定的文件夹。

在`uploadFile`方法中，我们首先获取文件并进行清理。我们使用一个名为`StringUtils`的 Spring 实用类来清理文件路径。您可以在这里看到清理过程：

```java
String fileName = StringUtils.cleanPath(file.getOriginalFilename());
```

如果文件为空，我们只是抛出一个异常。您可以在这里检查异常：

```java
    if(fileName.isEmpty()){
      throw new IOException("File is empty " + fileName);
    }
```

然后是真正的文件上传逻辑！我们只是使用`Files.copy`方法将文件从客户端复制到服务器位置。如果发生任何错误，我们会抛出`RuntimeException`：

```java
try {
      Files.copy(
        file.getInputStream(), this.location.resolve(fileName),  
        StandardCopyOption.REPLACE_EXISTING
      );
    } catch (IOException e) { 
      throw new IOException("File Upload Error : " + fileName);
    }
```

由于具体类已经完成了主要实现，控制器只是将`MultipartFile`传递给服务。我们在这里使用了`POST`方法，因为它是上传文件的完美方法。此外，您可以看到我们使用了`@Autowired`选项来使用`service`方法。

`FileController`类的`uploadFile`方法如下所示：

```java
package com.packtpub.restapp;
import java.io.IOException;
import java.util.LinkedHashMap;
import java.util.Map;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;
import com.packtpub.service.FileUploadService;
@RestController
@RequestMapping("/file")
public class FileController {  
  @Autowired
  FileUploadService fileUploadSevice;
  @ResponseBody
  @RequestMapping(value = "/upload", method = RequestMethod.POST)
  public Map<String, Object> uploadFile(@RequestParam("file") MultipartFile file) {
    Map<String, Object> map = new LinkedHashMap<>();
    try {
      fileUploadSevice.uploadFile(file);      
      map.put("result", "file uploaded");
    } catch (IOException e) {
      map.put("result", "error while uploading : "+e.getMessage());
    }    
    return map;
  }
} 
```

# 测试文件上传

您可以创建一个 HTML 文件如下，并测试文件上传 API。您还可以使用任何 REST 客户端来测试。我已经给您这个 HTML 文件来简化测试过程：

```java
<!DOCTYPE html>
<html>
<body>
<form action="http://localhost:8080/file/upload" method="post" enctype="multipart/form-data">
    Select image to upload:
    <input type="file" name="file" id="file">
    <input type="submit" value="Upload Image" name="submit">
</form>
</body>
</html>
```

# 摘要

在本章中，我们已经介绍了 Spring 5 中的 CRUD 操作（不包括响应式支持），从基本资源开始进行自定义。此外，我们还学习了如何在 Spring 中上传文件。在下一章中，我们将更多地了解 Spring Security 和 JWT（JSON Web Token）。
