# 第十三章：票务管理-高级 CRUD

我们的应用程序必须满足实时业务案例，如票务管理。本章将回顾本书前几章涵盖的大部分主题。

在本章中，我们将创建一个实时场景，并实现我们场景的业务需求——用户、客户服务代表（CSR）和管理员的票务管理。

我们的最后一章包括以下主题：

+   客户创建票务

+   客户、CSR 和管理员更新票务

+   客户删除票务

+   CSR/管理员删除多张票

# 使用 CRUD 操作进行票务管理

在转向票务管理系统之前，我们将介绍业务需求。

假设我们有一个银行网站应用程序，可以由我们的客户 Peter 和 Kevin 使用，我们有 Sammy，我们的管理员，和 Chloe，CSR，在应用程序出现问题时提供帮助。

Peter 和 Kevin 在付款过程中遇到一些问题。当他们尝试点击付款交易提交按钮时，它不起作用。此外，交易视图在一个网页上。因此，我们的用户（Peter 和 Kevin）将创建一个票务来分享他们的问题。

一旦票务创建完成，客户/CSR/管理员可以对其进行更新。此外，客户可以删除自己的票务。在更新时，任何人都可以更改严重性；然而，只有 CSR 和管理员可以更改状态，因为票务状态与官方活动有关。

客户可以查看他们的票务总数或单张票，但一次只能删除一张票。多删除选项适用于 CSR 和管理员。但是，CSR 一次只能删除三张票。管理员将在票务管理应用程序中拥有完全控制，并可以随时删除任意数量的票。

# 注册

让我们开始编码以满足上述要求。首先，我们需要从客户、CSR 和管理员注册开始。由于这些用户具有不同的角色，我们将为每个用户分配不同的用户类型。

# 用户类型

为了区分用户，我们提出了三种不同的用户类型，因此当他们访问我们的 REST API 时，他们的授权将有所不同。以下是三种不同的用户类型：

| **名称** | **用户类型** |
| --- | --- |
| 普通用户/客户 | 1 |
| CSR | 2 |
| 管理员 | 3 |

# 用户 POJO

在我们之前的`User`类中，我们只有`userid`和`username`。为了满足我们之前提到的业务需求，我们可能需要两个更多的变量。我们将在现有的`User`类中添加`password`和`usertype`：

```java
private String password;  
  /*
   * usertype:
   * 1 - general user
   * 2 - CSR (Customer Service Representative)
   * 3 - admin 
   */
private Integer usertype;
public String getPassword() {
    return password;
}
public void setPassword(String password) {
   this.password = password;
}
public void setUsertype(Integer usertype){
    this.usertype = usertype;
}  
public Integer getUsertype(){
    return this.usertype;
}
```

在前面的代码中，我们刚刚添加了`password`和`usertype`。此外，我们为我们的变量添加了 getter 和 setter 方法。

您可以在我们的 GitHub 存储库上查看完整的`User`类（[`github.com/PacktPublishing/Building-RESTful-Web-Services-with-Spring-5-Second-Edition`](https://github.com/PacktPublishing/Building-RESTful-Web-Services-with-Spring-5-Second-Edition)）。

您可能已经厌倦了添加 getter 和 setter 方法，因此我们将用 Lombok 库替换它们，这将在本章后面讨论。但是，Lombok 库与 Eclipse 或 STS IDE 存在一些冲突问题，您可能需要注意。在这些 IDE 的某些版本中，由于 Lombok 库的问题，您在类创建时将无法获得预期的行为。此外，一些开发人员提到他们在 Lombok 上有部署问题。

为了从我们的`User`类中自动生成用户 ID，我们将使用一个单独的计数器。我们将保留一个静态变量来做到这一点；在真实应用程序中保留静态计数器是不推荐的。为了简化我们的实现逻辑，我们使用了静态计数器。

以下代码将被添加到我们的`User`类中：

```java
private static Integer userCounter = 100;
```

我们已经开始有`100`个用户。每当添加一个新用户时，它将自动增加`userid`并将其分配给新用户。

`userCounter`的起始点没有限制。通过将用户系列保持在`2`（2XX）和票务系列保持在`3`（3XX），读者更容易区分用户和票务。

现在我们将创建一个新的构造函数来将用户添加到我们的应用程序中。此外，我们将增加`usercounter`参数并将其分配为每个新用户的`userid`：

```java
public User(String username, String password, Integer usertype) {
    userCounter++;
    this.userid = userCounter;
    this.username = username;
    this.password = password;
    this.usertype = usertype;
  }
```

上述构造函数将填充所有用户详情，包括`userid`（来自`usercounter`）。

在这里，我们将在`UserServiceImpl`类中添加一个新用户，包括`username`、`password`和`usertype`；每个用户的`usertype`都会有所不同（例如，管理员的`usertype`是`3`）：

```java
  @Override
  public void createUser(String username, String password, Integer usertype){
    User user = new User(username, password, usertype); 
    this.users.add(user);
  }
```

在上述代码中，我们创建了一个新用户并将其添加到现有用户列表中。

在上述代码中，我们没有在`UserService`中提及抽象方法。我们假设每个具体方法在接口中都有一个抽象方法。以后，请考虑在适当的接口中添加所有抽象方法。

# 客户注册

现在是添加客户的时候了。新客户将需要通过添加用户名和密码详情来创建一个账户。

我们将讨论客户注册 API。这个 API 将帮助任何新客户注册他们的账户：

```java
  @ResponseBody
  @RequestMapping(value = "/register/customer", method = RequestMethod.POST)
  public Map<String, Object> registerCustomer( 
      @RequestParam(value = "username") String username,
      @RequestParam(value = "password") String password
    ) {   
    userSevice.createUser(username, password, 1); 
    return Util.getSuccessResult();
  }
```

在上述代码中，我们已经添加了一个 API 来注册客户。调用这个 API 的人将被视为客户（而不是管理员/CSR）。正如你所看到的，我们已经将`usertype`设为`1`，因此它将被视为客户。

以下是客户注册的 SoapUI 截图：

![](img/d9eac2ba-9b15-4ae9-a9ac-a42f13ec356d.png)

此外，在上述代码中，我们使用了来自我们的`Util`类的`getSuccessResult`。我们将在以下代码中看到其他`Util`方法：

```java
package com.packtpub.util;
import java.util.LinkedHashMap;
import java.util.Map;
public class Util {
  public static <T> T getUserNotAvailableError(){
    Map<String, Object> map = new LinkedHashMap<>();    
    map.put("result_code", 501);
    map.put("result", "User Not Available"); 
    return (T) map;
  }  
  public static <T> T getSuccessResult(){
    Map<String, Object> map = new LinkedHashMap<>();    
    map.put("result_code", 0);
    map.put("result", "success"); 
    return (T) map;
  }  
  public static <T> T getSuccessResult(Object obj){
    Map<String, Object> map = new LinkedHashMap<>();    
    map.put("result_code", 0);
    map.put("result", "success");
    map.put("value", obj);
    return (T) map;
  }
}
```

在上述代码中，我们创建了一个`Util`类，用于保存将在不同控制器中使用的通用方法，例如`Ticket`和`User`。这些`Util`方法用于避免我们应用程序中的代码重复。

为了简化流程，在这段代码中我们没有使用任何异常处理机制。您可能需要使用适当的异常处理技术来实现这些方法。

# 管理员注册

每个应用程序都将有一个管理员来控制所有操作，例如删除客户和更改状态。在这里，我们将讨论管理员注册 API。

管理员注册 API 也将使用`createUser`方法来创建管理员。以下是管理员注册的代码：

```java
  @ResponseBody
  @RequestMapping(value = "/register/admin", method = RequestMethod.POST)
  public Map<String, Object> registerAdmin( 
      @RequestParam(value = "username") String username,
      @RequestParam(value = "password") String password
    ) {
    Map<String, Object> map = new LinkedHashMap<>();
    userSevice.createUser(username, password, 3); // 3 - admin (usertype)
    map.put("result", "added");
    return map;
  }
```

在上述代码中，我们在管理员注册中添加了代码，同时在`createUser`构造函数调用中提及了`3`（管理员的用户类型）。此外，您可以看到我们使用`POST`方法进行注册。

以下是`http://localhost:8080/user/register/admin`管理员注册 SoapUI API 调用的截图：

![](img/0a8e6410-d5aa-4212-80d6-72cc3fda461c.png)

在我们的票务管理中，我们没有对用户重复注册进行任何限制，这意味着我们可以有许多具有相同名称的用户。我们建议您避免重复注册，因为这将破坏流程。为了尽可能简化我们的实现，我们忽略了这种限制。但是，您可以实现限制以改进应用程序。

# CSR 注册

在这一部分，我们将讨论 CSR 注册。

客户注册只有一个区别——`usertype`。除了`usertype`和 API 路径之外，与其他注册调用没有任何不同：

```java
  @ResponseBody
  @RequestMapping(value = "/register/csr", method = RequestMethod.POST)
  public Map<String, Object> registerCSR( 
      @RequestParam(value = "username") String username,
      @RequestParam(value = "password") String password
    ) {     
    userSevice.createUser(username, password, 2);
    return Util.getSuccessResult();
  }
```

与其他 API 一样，我们使用`2`（CSR 的用户类型）来注册 CSR。让我们看看在 SoapUI 中的 API 调用，如下所示：

![](img/723a18ce-d4aa-4368-a785-d3f29bab0331.png)

# 登录和令牌管理

在上一节中，我们已经涵盖了用户注册主题，例如客户、管理员和 CSR。一旦用户成功注册，他们将需要登录以执行操作。因此，让我们创建与登录和会话相关的 API 和业务实现。

在转到登录和会话之前，我们将讨论 JSON Web Token，它将用于会话认证。由于我们已经在我们的`securityService`类中有`createToken`方法，我们只会讨论令牌生成中使用的`subject`。

# 生成令牌

我们可能需要使用 JSON Web Token 来进行会话。我们将使用现有的令牌生成方法来保留用户详细信息：

```java
    String subject = user.getUserid()+"="+user.getUsertype();
    String token = securityService.createToken(subject, (15 * 1000 * 60)); // 15 mins expiry time
```

我们已经使用`user.getUserid()+"="+user.getUsertype()`作为主题。此外，我们已经提到`15`分钟作为到期时间，因此令牌将只在`15`分钟内有效。

# 客户登录

让我们为客户创建一个登录 API。客户必须提供用户名和密码详细信息作为参数。在实际应用中，这些详细信息可能来自 HTML 表单，如下所示：

```java
  @ResponseBody
  @RequestMapping(value = "/login/customer", method = RequestMethod.POST)
  public Map<String, Object> loginCustomer( 
      @RequestParam(value = "username") String username,
      @RequestParam(value = "password") String password
    ) {
    User user = userSevice.getUser(username, password, 1);    
    if(user == null){
      return Util.getUserNotAvailableError();
    }    
    String subject = user.getUserid()+"="+user.getUsertype();
    String token = securityService.createToken(subject, (15 * 1000 * 60)); // 15 minutes expiry time    
    return Util.getSuccessResult(token);
  }
```

在前面的代码中，我们通过传递所有必要的参数从`userService`调用了`getUser`方法。由于用户类型是`1`，我们在我们的方法中传递了`1`。一旦我们得到用户，我们就会检查它是否为空。如果为空，我们将简单地抛出错误。如果用户不为空，我们将创建一个令牌主题（`user.getUserid()+"="+user.getUsertype()`）并创建一个具有`15`分钟到期时间的令牌，正如我们之前提到的那样。

如果一切如我们所期望的那样，我们将创建一个结果映射并将映射作为 API 响应返回。当我们调用此 API 时，这个映射将显示为我们结果中的 JSON 响应。

此外，在前面的代码中，我们使用了`getUserNotAvailableError`来返回错误详情。由于我们将在所有与会话相关的 API 中使用此错误，我们已经创建了一个单独的方法来避免代码重复。

在这里，我们可以看到客户登录的 SoapUI 截图：

![](img/a56d9b3e-92e6-4a2e-bfb3-6f202490f25b.png)

在用户成功登录的情况下，我们将在响应 JSON 中获得一个令牌。我们将不得不使用令牌进行与会话相关的 API，如添加票务。这里提供了一个示例令牌：

eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMDM9MSIsImV4cCI6MTUxNTg5MDMzN30.v9wtiG-fNWlpjgJmou7w2oxA9XjXywsH32cDZ-P4zM4

在一些方法中，我们可能会看到`<T> T`返回类型，这是 Java 泛型的一部分。通过保持这样的泛型，我们可以通过适当地进行转换来返回任何对象。

这是一个示例：

`return (T) map;` 返回类型

# 管理员登录

当我们看到客户登录部分时，我们也将为管理员创建一个登录 API。

在这里，我们将为管理员登录创建一个 API，并在成功验证后生成一个令牌：

```java
  @ResponseBody
  @RequestMapping(value = "/login/admin", method = RequestMethod.POST)
  public Map<String, Object> loginAdmin( 
      @RequestParam(value = "username") String username,
      @RequestParam(value = "password") String password
    ) {
    Map<String, Object> map = new LinkedHashMap<>();   
    User user = userSevice.getUser(username, password, 3);    
    if(user == null){ 
      return Util.getUserNotAvailableError();
    }    
    String subject = user.getUserid()+"="+user.getUsertype();
    String token = securityService.createToken(subject, (15 * 1000 * 60)); // 15 mins expiry time    
    map.put("result_code", 0);
    map.put("result", "success");
    map.put("token", token);    
    return map;
  }
```

前面的登录 API 将仅用于管理员目的。我们已经使用`usertype`作为`3`来创建一个管理员用户。此外，我们已经使用了`Util`方法`getUserNotAvailableError`。

这是管理员登录的 SoapUI 截图：

![](img/4d2b4240-5fc3-436e-a1b9-1aabac37f114.png)

# CSR 登录

在这一部分，我们将讨论 CSR 登录和在`TicketController`中为 CSR 生成令牌：

```java
  @ResponseBody
  @RequestMapping(value = "/login/csr", method = RequestMethod.POST)
  public Map<String, Object> loginCSR( 
      @RequestParam(value = "username") String username,
      @RequestParam(value = "password") String password
    ) {    
    User user = userSevice.getUser(username, password, 2);    
    if(user == null){
      return Util.getUserNotAvailableError();
    }    
    String subject = user.getUserid()+"="+user.getUsertype();
    String token = securityService.createToken(subject, (15 * 1000 * 60)); // 15 mins expiry time

    return Util.getSuccessResult(token);
  }
```

像往常一样，我们将从我们的列表中获取用户并检查是否为空。如果用户不可用，我们将抛出一个错误，否则代码将继续执行。与其他用户类型一样，我们将为 CSR 创建一个单独的 API，并将`usertype`作为`1`传递以创建一个 CSR。

您可以在以下截图中看到 CSR 登录 API：

![](img/e554a0dc-aad3-4f8e-aa6c-2131061e3374.png)

# 票务管理

为了创建一个票，我们需要创建一个`Ticket`类并将票存储在列表中。我们将更多地讨论`Ticket`类，票务列表和其他与票务相关的工作，如用户票务管理，管理员票务管理和 CSR 票务管理。

# 票务 POJO

我们将创建一个`Ticket`类，并涉及一些基本变量来存储与票务相关的所有细节。以下代码将帮助我们理解`Ticket`类：

```java
public class Ticket {
  private Integer ticketid;  
  private Integer creatorid;  
  private Date createdat;  
  private String content;  
  private Integer severity;  
  private Integer status;
  // getter and setter methods
  @Override
  public String toString() {
    return "Ticket [ticketid=" + ticketid + ", creatorid=" + creatorid
        + ", createdat=" + createdat + ", content=" + content
        + ", severity=" + severity + ", status=" + status + "]";
  }   
  private static Integer ticketCounter = 300;  
  public Ticket(Integer creatorid, Date createdat, String content, Integer severity, Integer status){
    ticketCounter++;
    this.ticketid = ticketCounter;
    this.creatorid = creatorid;
    this.createdat = createdat;
    this.content = content;
    this.severity = severity;
    this.status = status;
  }
}
```

前面的代码将存储票务详情，如`ticketid`，`creatorid`，`createdat`，`content`，`severity`和`status`。此外，我们使用了一个名为`ticketCounter`的静态计数器来在创建票务时递增`ticketid`。默认情况下，它将从`300`开始。

另外，我们已经使用了构造函数和`toString`方法，因为我们将在我们的实现中使用它们。

我们将不得不创建`TicketService`接口（用于抽象方法）和`TicketServiceImpl`具体类，用于所有与票务相关的业务逻辑实现。

以下代码将显示如何添加一张票：

```java
  @Override
  public void addTicket(Integer creatorid, String content, Integer severity, Integer status) {
    Ticket ticket = new Ticket(creatorid, new Date(), content, severity, status);    
    tickets.add(ticket);
  }
ticketid as created by the incrementer in the Ticket class. Once the ticket is created, we add it to the ticket list, which will be used for other operations.
```

# 通过令牌获取用户

对于所有与票务相关的操作，我们需要用户会话。在登录方法中，我们在成功登录后获得了令牌。我们可以使用令牌来获取用户详细信息。如果令牌不可用，不匹配或过期，我们将无法获取用户详细信息。

在这里，我们将实现从令牌中获取用户详细信息的方法：

```java
  @Override
  public User getUserByToken(String token){
    Claims claims = Jwts.parser()              .setSigningKey(DatatypeConverter.parseBase64Binary(SecurityServiceImpl.secretKey))
             .parseClaimsJws(token).getBody();    
    if(claims == null || claims.getSubject() == null){
      return null;
    }    
    String subject = claims.getSubject();   
    if(subject.split("=").length != 2){
      return null;
    }    
    String[] subjectParts = subject.split("=");    
    Integer usertype = new Integer(subjectParts[1]);
    Integer userid = new Integer(subjectParts[0]);   
    return new User(userid, usertype);
  }
```

在上面的代码中，我们使用令牌来获取用户详细信息。我们使用 JWT 解析器首先获取声明，然后我们将获取主题。如果您记得，我们在为所有用户登录选项创建令牌时使用了`user.getUserid()+"="+user.getUsertype()`作为主题。因此，主题将采用相同的格式，例如，`101`（用户 ID）=`1`（用户类型）表示客户，因为客户的用户类型是`1`。

此外，我们还要检查主题是否有效，使用`subject.split("=").length != 2`。如果我们使用不同的令牌，它将简单地返回 null。

一旦我们得到了正确的主题，我们将获取`userid`和`usertype`，然后我们将通过创建`User`对象来返回用户。

因为`getUserByToken`对所有用户都是通用的，所以它将用于我们所有的用户检索方法。

# 用户票务管理

首先，为了简化我们的业务需求，我们保持只有客户可以创建票据的规则。管理员和 CSR 都不能创建票据。在实时情况下，您可能有不同的票务管理方法。但是，我们将尽量保持业务需求尽可能简单。

# 票务控制器

在这里，我们将讨论客户创建一张票据：

```java
  /*
   * Rule:
   * Only user can create a ticket
   */
  @SuppressWarnings("unchecked")
  @ResponseBody
  @UserTokenRequired
  @RequestMapping(value = "", method = RequestMethod.POST)
  public <T> T addTicket( 
      @RequestParam(value="content") String content, 
      HttpServletRequest request
      ) {    
    User user = userSevice.getUserByToken(request.getHeader("token")); 
    ticketSevice.addTicket(user.getUserid(), content, 2, 1);     
    return Util.getSuccessResult(); 
  }
```

当用户提交一张票据时，他们只会发送关于他们在应用程序中遇到的问题的详细信息。我们为这样的详细信息提供了内容变量。此外，我们从他们在标头中传递的令牌中获取用户详细信息。

我们可以在以下截图中看到成功的响应：

![](img/b5e4bc53-015e-4f74-8bdd-5814b033e219.png)

在之前的 API 中，我们已经使用了`@UserTokenRequired`注解来验证用户令牌。我们将在这里检查注解和实现的详细信息。

# UserTokenRequired 接口

在这里，我们将介绍`UserTokenRequired`接口，并在下一节中跟进验证逻辑：

```java
package com.packtpub.aop;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface UserTokenRequired {
}
```

# UserTokenRequiredAspect 类

这个类将在解密后检查用户令牌的用户 ID 和用户类型验证：

```java
package com.packtpub.aop;
import javax.servlet.http.HttpServletRequest;
import javax.xml.bind.DatatypeConverter;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import com.packtpub.service.SecurityServiceImpl;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
@Aspect
@Component
public class UserTokenRequiredAspect { 
  @Before("@annotation(userTokenRequired)")
  public void tokenRequiredWithAnnotation(UserTokenRequired userTokenRequired) throws Throwable{    
    ServletRequestAttributes reqAttributes = (ServletRequestAttributes)RequestContextHolder.currentRequestAttributes();
    HttpServletRequest request = reqAttributes.getRequest();    
    // checks for token in request header
    String tokenInHeader = request.getHeader("token");    
    if(StringUtils.isEmpty(tokenInHeader)){
      throw new IllegalArgumentException("Empty token");
    }     
    Claims claims = Jwts.parser()              .setSigningKey(DatatypeConverter.parseBase64Binary(SecurityServiceImpl.secretKey))
             .parseClaimsJws(tokenInHeader).getBody();    
    if(claims == null || claims.getSubject() == null){
      throw new IllegalArgumentException("Token Error : Claim is null");
    }   
    String subject = claims.getSubject();

    if(subject.split("=").length != 2){
      throw new IllegalArgumentException("User token is not authorized");
    } 
  }
}
```

在上述的`UserTokenRequiredAspect`类中，我们刚刚从标头中获取了令牌，并验证了令牌是否有效。如果令牌无效，我们将抛出异常。

如果用户为空（也许有错误或空令牌），它将在响应中返回`"用户不可用"`。一旦提供了必要的令牌，我们将通过调用`TicketServiceImpl`中的`addTicket`方法来添加票据，这是我们之前提到的。

严重级别如下：

+   次要：级别 1

+   正常：级别 2

+   主要：级别 3

+   关键：级别 4

级别 1 被认为是低的，级别 4 被认为是高的，如下所示

`@SuppressWarnings ("unchecked")`。在某些地方，我们可能已经使用了`@SuppressWarnings`注解，告诉编译器不需要担心正确的转换，因为这将得到处理。

如果用户在任何与会话相关的 API 中传递了错误的`JWT`，我们将得到以下错误：

```java
{
   "timestamp": 1515786810739,
   "status": 500,
   "error": "Internal Server Error",
   "exception": "java.lang.IllegalArgumentException",
   "message": "JWT String argument cannot be null or empty.",
   "path": "/ticket"
}
```

上述错误只是提到`JWT`字符串为空或 null。

# 获取我的票据-客户

一旦票据创建，客户可以通过调用`/my/tickets` API 来查看他们的票据。以下方法将处理获取票据的要求：

```java
  @ResponseBody
  @RequestMapping("/my/tickets")
  public Map<String, Object> getMyTickets(
      HttpServletRequest request
      ) {    
    User user = userSevice.getUserByToken(request.getHeader("token"));    
    if(user == null){
      return Util.getUserNotAvailableError();
    }    
    return Util.getSuccessResult(ticketSevice.getMyTickets(user.getUserid()));
  }
```

在前面的代码中，我们通过令牌验证了用户会话，并获得了会话中用户的票务：

![](img/be7771c2-6b32-4669-9f88-738774ebb075.png)

# 允许用户查看他们的单张票

与查看所有客户票务一样，客户也可以通过调用`/{ticketid}`API 查看他们自己的每张票的详细信息。让我们看看这个方法是如何工作的：

```java
  @ResponseBody
  @TokenRequired
  @RequestMapping("/{ticketid}")
  public <T> T getTicket(
    @PathVariable("ticketid") final Integer ticketid, 
    HttpServletRequest request
    ) {

    return (T) Util.getSuccessResult(ticketSevice.getTicket(ticketid));
  }
```

在前面的 API 中，在验证会话后，我们使用`TicketServiceImpl`中的`getTicket`方法来获取用户票务详情。

您可以使用此截图来验证结果：

![](img/27a05da9-9e44-4918-8d49-0d71298d7545.png)

您可以清楚地看到令牌在我们的标题中使用。没有令牌，API 将抛出异常，因为它是与会话相关的交易。

# 允许客户更新票务

假设客户想要出于某种原因更新他们自己的票务，例如添加额外信息。我们将为客户提供更新票务的选项。

# 更新票务-服务（TicketServiceImpl）

对于更新选项，我们将在我们的`TicketServiceImpl`类中添加`updateTicket`方法：

```java
  @Override
  public void updateTicket(Integer ticketid, String content, Integer severity, Integer status) {    
    Ticket ticket = getTicket(ticketid);    
    if(ticket == null){
      throw new RuntimeException("Ticket Not Available");
    }    
    ticket.setContent(content);
    ticket.setSeverity(severity);
    ticket.setStatus(status); 
  }
```

在前面的方法中，我们通过`getTicket`方法检索了票务，然后更新了必要的信息，如`content`，`severity`和`status`。

现在我们可以在我们的 API 中使用`updateTicket`方法，这里提到了：

```java
  @ResponseBody
  @RequestMapping(value = "/{ticketid}", method = RequestMethod.PUT)
  public <T> T updateTicketByCustomer (
      @PathVariable("ticketid") final Integer ticketid,      
      @RequestParam(value="content") String content,      
      HttpServletRequest request,
      HttpServletResponse response
      ) {   
    User user = userSevice.getUserByToken(request.getHeader("token"));    
    if(user == null){
      return getUserNotAvailableError();
    }    
    ticketSevice.updateTicket(ticketid, content, 2, 1);    
    Map<String, String> result = new LinkedHashMap<>();
    result.put("result", "updated");    
    return (T) result; 
  }
```

在前面的代码中，在验证会话后，我们调用了`updateTicket`并传递了新内容。此外，在成功完成后，我们向呼叫者发送了适当的响应。

![](img/39034ed5-a27c-4b1e-9233-bc2a16b5905f.png)

对于更新选项，我们使用了`PUT`方法，因为这是用于更新目的的适当 HTTP 方法。但是，我们也可以使用`POST`方法进行此类操作，因为没有限制。

# 删除票务

到目前为止，我们已经涵盖了票务的创建、读取和更新操作。在本节中，我们将讨论客户的删除选项。

# 删除服务-服务（TicketServiceImpl）

我们将在我们的`TicketServiceImpl`类中添加`deleteMyTicket`方法，假设我们已经在我们的接口中添加了抽象方法：

```java
@Override
  public void deleteMyTicket(Integer userid, Integer ticketid) { 
    tickets.removeIf(x -> x.getTicketid().intValue() == ticketid.intValue() && x.getCreatorid().intValue() == userid.intValue());
  }
```

在前面的代码中，我们使用了`removeIf` Java Streams 选项来查找并从流中删除项目。如果匹配了 userid 和 ticket，该项目将自动从流中删除。

# 删除我的票务-API（票务控制器）

我们可以调用我们在 API 中早期创建的`deleteMyTicket`方法：

```java
  @ResponseBody
  @UserTokenRequired
  @RequestMapping(value = "/{ticketid}", method = RequestMethod.DELETE)
  public <T> T deleteTicketByUser (
      @RequestParam("ticketid") final Integer ticketid,      
      HttpServletRequest request 
      ) {   
    User user = userSevice.getUserByToken(request.getHeader("token"));    
    ticketSevice.deleteMyTicket(user.getUserid(), ticketid);    
    return Util.getSuccessResult(); 
  }
```

像往常一样，我们将检查会话并在我们的`TicketServiceImpl`类中调用`deleteTicketByUser`方法。一旦删除选项完成，我们将简单地返回一个说“成功”的地图作为结果。

在删除票务后，这是 SoapUI 的响应：

![](img/4e8498ff-5291-4cb1-9108-c9405564da10.png)

在我们的票务 CRUD 中，当它为空时，我们没有选项来抛出异常。如果您删除了所有现有的票务并调用获取票务，您将获得一个带有空值的成功消息。您可以通过添加空检查和限制来改进应用程序。

# 管理员票务管理

在前一节中，我们看到了客户的票务管理。客户只对他们自己的票务有控制权，不能对其他客户的票务做任何事情。在管理员模式下，我们可以控制应用程序中的任何可用票务。在本节中，我们将看到管理员执行的票务管理。

# 允许管理员查看所有票务

由于管理员可以完全控制查看应用程序中的所有票务，因此我们在`TicketServiceImpl`类中保持查看票务方法非常简单，没有任何限制。

# 获取所有票务-服务（TicketServiceImpl）

在这里，我们将讨论管理员实现部分，以获取应用程序中的所有票务：

```java
  @Override
  public List<Ticket> getAllTickets() {
    return tickets;
  }
```

在前面的代码中，我们没有任何特定的限制，只是从我们的票务列表中返回所有票务。

# 获取所有票务-API（票务控制器）

在票务控制器 API 中，我们将添加一个方法来获取管理员的所有票务：

```java
  @ResponseBody
  @AdminTokenRequired
  @RequestMapping("/by/admin")
  public <T> T getAllTickets(
    HttpServletRequest request,
    HttpServletResponse response) {

    return (T) ticketSevice.getAllTickets();
  }
```

前面的 API`/by/admin`将在管理员需要查看所有票务时调用。我们在`TicketServiceImpl`类中调用了`getAllTickets`方法。

我们使用了一个简单的 AOP 来验证管理员令牌，称为`@AdminTokenRequired`。让我们看看这个 API 的实现部分。

# AdminTokenRequired 接口

`AdminTokenRequired`接口将是我们实现的基础，我们稍后会涵盖：

```java
package com.packtpub.aop;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface AdminTokenRequired {
}
```

在前面的代码中，我们为验证管理员令牌引入了接口。验证方法将在`AdminTokenRequiredAspect`类中跟进。

# AdminTokenRequiredAspect 类

在切面类中，我们将对管理员令牌进行验证：

```java
package com.packtpub.aop;
import javax.servlet.http.HttpServletRequest;
import javax.xml.bind.DatatypeConverter;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import com.packtpub.service.SecurityServiceImpl;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
@Aspect
@Component
public class AdminTokenRequiredAspect { 
  @Before("@annotation(adminTokenRequired)")
  public void adminTokenRequiredWithAnnotation(AdminTokenRequired adminTokenRequired) throws Throwable{    
    ServletRequestAttributes reqAttributes = (ServletRequestAttributes)RequestContextHolder.currentRequestAttributes();
    HttpServletRequest request = reqAttributes.getRequest();    
    // checks for token in request header
    String tokenInHeader = request.getHeader("token");  
    if(StringUtils.isEmpty(tokenInHeader)){
      throw new IllegalArgumentException("Empty token");
    }    
    Claims claims = Jwts.parser()              .setSigningKey(DatatypeConverter.parseBase64Binary(SecurityServiceImpl.secretKey))
             .parseClaimsJws(tokenInHeader).getBody();    
    if(claims == null || claims.getSubject() == null){
      throw new IllegalArgumentException("Token Error : Claim is null");
    }    
    String subject = claims.getSubject();   
    if(subject.split("=").length != 2 || new Integer(subject.split("=")[1]) != 3){
      throw new IllegalArgumentException("User is not authorized");
    } 
  }
}
```

在前面的代码中，我们在`AdminTokenRequiredAspect`类中提供了令牌验证技术。这个切面组件将在方法执行之前执行。此外，在这个方法中，我们检查了令牌是否为空和 null，以及令牌的用户类型。

检查管理员查看票务的 SoapUI 响应：

![](img/109ba20b-8859-4994-aed3-afbe8e2d7345.png)

如果我们使用错误的令牌或空令牌，我们将得到这样的响应：

```java
{
   "timestamp": 1515803861286,
   "status": 500,
   "error": "Internal Server Error",
   "exception": "java.lang.RuntimeException",
   "message": "User is not authorized",
   "path": "/ticket/by/admin"
}
```

通过保持 AOP 注解，我们可以在每个方法上有几行代码，因为注解会处理业务逻辑。

# 管理员更新票务

一旦票务创建完成，管理员就可以查看。与客户不同，管理员有更多的控制权，可以更新票务的状态和严重性，以及其内容。

# 通过管理员更新票务 - 服务（TicketServiceImpl）

在这里，我们将实现管理员更新票务的方法：

```java
  @ResponseBody
  @RequestMapping(value = "/by/admin", method = RequestMethod.PUT)
  public <T> T updateTicketByAdmin (
      @RequestParam("ticketid") final Integer ticketid,      
      @RequestParam(value="content") String content,
      @RequestParam(value="severity") Integer severity,
      @RequestParam(value="status") Integer status,      
      HttpServletRequest request,
      HttpServletResponse response
      ) {    
    User user = userSevice.getUserByToken(request.getHeader("token"));    
    if(user == null){
      return getUserNotAvailableError();
    }    
    ticketSevice.updateTicket(ticketid, content, severity, status);    
    Map<String, String> result = new LinkedHashMap<>();
    result.put("result", "updated");    
    return (T) result; 
  }
```

在前面的代码中，我们在 API 中使用了`/by/admin`路径来区分这个 API 和客户的更新方法。此外，我们从请求中获取了严重性和状态参数。一旦管理员通过令牌验证，我们将调用`updateTicket`方法。如果你看到这个`updateTicket`方法，我们没有硬编码任何内容。

一旦更新过程完成，我们将返回结果`"success"`作为响应，你可以在截图中检查到：

![](img/8a05a3ad-c4d4-4523-9e51-e1bcea63a61e.png)

在实际应用中，管理员可能无法控制客户的内容，比如问题。然而，我们为管理员提供了编辑内容的选项，以使我们的业务逻辑更加简单。

# 允许管理员查看单个票务

由于管理员对票务有完全的控制权，他们也可以查看用户创建的任何单个票务。由于我们已经定义了`getTicket`API`/{ticketid}`，我们也可以将同样的 API 用于管理员的查看目的。

# 允许管理员删除票务

由于管理员有更多的控制权，我们为管理员提供了无限的多删除选项，以便在应用程序中一次性删除一大堆票务时非常方便。

# 删除票务 - 服务（TicketServiceImpl）：

在下面的代码中，我们将讨论管理员的多票删除选项：

```java
  @Override
  public void deleteTickets(User user, String ticketids) {  
    List<String> ticketObjList = Arrays.asList(ticketids.split(","));    
    List<Integer> intList =
      ticketObjList.stream()
      .map(Integer::valueOf)
      .collect(Collectors.toList());     
    tickets.removeIf(x -> intList.contains(x.getTicketid()));
  }
```

在前面的代码中，我们赋予管理员删除多个票务的权力。由于管理员有完全的控制权，我们在这里没有应用特定的过滤器。我们使用 Java Streams 将票务作为列表获取，然后将它们与票务 ID 匹配以从票务列表中删除。

# 通过管理员删除票务 - API（票务控制器）：

以下方法将把`ticketids`转发到相应的`TicketServiceImpl`方法：

```java
  @ResponseBody
  @AdminTokenRequired
  @RequestMapping(value = "/by/admin", method = RequestMethod.DELETE)
  public <T> T deleteTicketsByAdmin ( 
      @RequestParam("ticketids") final String ticketids,
      HttpServletRequest request
      )  {

    User user = userSevice.getUserByToken(request.getHeader("token"));

    ticketSevice.deleteTickets(user, ticketids);

    return Util.getSuccessResult(); 
  }
```

在前面的代码中，我们首先通过`@AdminTokenRequired`检查会话，然后在会话验证通过后删除票务。

我们可以通过这个 SoapUI 截图检查 API 的结果：

![](img/94842506-c2a9-45ff-b19e-d6d2f8a9c111.png)

在多票删除选项中，我们使用逗号分隔的值来发送多个票务 ID。也可以使用单个`ticketid`来调用这个 API。

# CSR 票务管理

最后，我们将在本节讨论 CSR 工单管理。CSR 可能没有像管理员那样的控制权；然而，在大多数情况下，他们在工单管理应用程序中有与管理员匹配的选项。在接下来的部分中，我们将讨论 CSR 在工单上的所有授权 CRUD 操作。

# CSR 更新工单

在本节中，我们将讨论 CSR 通过工单管理更新工单的新内容、严重程度和状态：

```java
  @ResponseBody
  @CSRTokenRequired
  @RequestMapping(value = "/by/csr", method = RequestMethod.PUT)
  public <T> T updateTicketByCSR (
      @RequestParam("ticketid") final Integer ticketid,     
      @RequestParam(value="content") String content,
      @RequestParam(value="severity") Integer severity,
      @RequestParam(value="status") Integer status,      
      HttpServletRequest request
      ) {    
    ticketSevice.updateTicket(ticketid, content, severity, status);    
    return Util.getSuccessResult(); 
  }
```

在上述代码中，我们获取了所有必要的信息，如内容、严重程度和状态，并将这些信息提供给`updateTicket`方法。

我们使用了一个简单的 AOP 来验证名为`@CSRTokenRequired`的管理员令牌。让我们来看看这个 API 的实现部分。

# CSRTokenRequired AOP

`AdminTokenRequired`接口将是我们稍后将要实现的基础：

```java
package com.packtpub.aop;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface CSRTokenRequired {
}
```

在上述代码中，我们引入了验证管理员令牌的注解。验证方法将在`CSRTokenRequiredAspect`类中跟进。

# CSRTokenRequiredAspect

在`CSRTokenRequiredAspect`类中，我们将对管理员令牌进行验证：

```java
package com.packtpub.aop;
import javax.servlet.http.HttpServletRequest;
import javax.xml.bind.DatatypeConverter;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import com.packtpub.service.SecurityServiceImpl;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
@Aspect
@Component
public class CSRTokenRequiredAspect {  
  @Before("@annotation(csrTokenRequired)")
  public void adminTokenRequiredWithAnnotation(CSRTokenRequired csrTokenRequired) throws Throwable{    
    ServletRequestAttributes reqAttributes = (ServletRequestAttributes)RequestContextHolder.currentRequestAttributes();
    HttpServletRequest request = reqAttributes.getRequest();    
    // checks for token in request header
    String tokenInHeader = request.getHeader("token");    
    if(StringUtils.isEmpty(tokenInHeader)){
      throw new IllegalArgumentException("Empty token");
    }     
    Claims claims = Jwts.parser()              .setSigningKey(DatatypeConverter.parseBase64Binary(SecurityServiceImpl.secretKey))
             .parseClaimsJws(tokenInHeader).getBody();   
    if(claims == null || claims.getSubject() == null){
      throw new IllegalArgumentException("Token Error : Claim is null");
    }    
    String subject = claims.getSubject();    
    if(subject.split("=").length != 2 || new Integer(subject.split("=")[1]) != 2){
      throw new IllegalArgumentException("User is not authorized");
    } 
  }
}
```

在上述代码中，我们在`CSRTokenRequiredAspect`类中提供了令牌验证技术。这个方面组件将在方法执行之前执行。此外，在这个方法中，我们检查令牌是否为空和 null，以及令牌的用户类型。

这是我们`/ticket/{ticketid}`更新 API 的截图：

![](img/7d526f83-2b48-42f1-aebc-c85ce3062793.png)

# CSR 查看所有工单

在查看所有工单方面，CSR 与管理员拥有相同的权限，因此我们不需要更改服务实现。但是，我们可能需要验证令牌以确保用户是 CSR。

# 通过 CSR 查看所有工单 - API（工单控制器）

当任何 CSR 调用时，以下内容将获取 CSR 的所有工单：

```java
  @ResponseBody
  @CSRTokenRequired
  @RequestMapping("/by/csr")
  public <T> T getAllTicketsByCSR(HttpServletRequest request) {  
    return (T) ticketSevice.getAllTickets();
  }
```

在上述 API 中，我们只使用了`@CSRTokenRequired`来验证用户。除了 API 路径和注解之外，其他都与管理员查看所有工单相同。

当我们检查 SoapUI 的截图时，我们可以清楚地看到客户创建的两张工单。

![](img/01b56fe6-7f3e-41e3-b07f-dd8c79d62dba.png)

# CSR 查看单个工单

除了多删除选项外，CSR 与管理员拥有相同的权限，我们可以在这里使用相同的`/{ticketid}`，用于 CSR 和管理员查看单个工单 API。

# CSR 删除工单

通过 CSR 删除工单几乎就像在管理员模式下删除工单一样。然而，我们的业务要求规定 CSR 一次不能删除超过三张工单。我们将在现有方法中添加具体逻辑。

# 删除工单 - 服务（TicketServivceImpl）

以下是 CSR 删除多张工单的服务实现：

```java
  @Override
  public void deleteTickets(User user, String ticketids) {   
    List<String> ticketObjList = Arrays.asList(ticketids.split(","));   
    if(user.getUsertype() == 2 && ticketObjList.size() > 3){
      throw new RuntimeException("CSR can't delete more than 3 tickets");
    }    
    List<Integer> intList =
      ticketObjList.stream()
      .map(Integer::valueOf)
      .collect(Collectors.toList())
        ;     
    tickets.removeIf(x -> intList.contains(x.getTicketid()));
  }
```

对于删除多张工单，我们在`TicketServiceImpl`类中使用了现有的代码。然而，根据我们的业务要求，我们的 CSR 不能删除超过三张工单，因此我们添加了额外的逻辑来检查工单数量。如果工单列表大小超过三，我们会抛出异常，否则我们将删除这些工单。

# 通过 CSR 删除工单 - API（工单控制器）

在 API 中，我们将简单地调用我们之前实现的`deleteTickets`方法：

```java
  @ResponseBody
  @CSRTokenRequired
  @RequestMapping(value = "/by/csr", method = RequestMethod.DELETE)
  public <T> T deleteTicketsByCSR (
      @RequestParam("ticketids") final String ticketids,     
      HttpServletRequest request,
      HttpServletResponse response
      ) {    
    User user = userSevice.getUserByToken(request.getHeader("token"));    
    ticketSevice.deleteTickets(user.getUserid(), ticketids);    
    Map<String, String> result = new LinkedHashMap<>();
    result.put("result", "deleted");    
    return (T) result; 
  }
```

除了删除选项上的最大工单限制外，CSR 删除工单不需要进行太大的更改。但是，我们已经在我们的 API 中添加了`@CSRTokenRequired`注解。

这是 CSR 删除多张工单的 SoapUI 截图：

![](img/5eb0c235-4419-4280-a7d1-d41b61629899.png)

Postman 工具可能存在与`DELETE`选项相关的问题，包括参数（截至版本 5.4.0），当您在管理员和 CSR 中使用多删除 API 时，可能无法获得预期的结果。对于这种情况，请使用 SoapUI 客户端。

# 摘要

在这最后一章中，我们通过满足本章第一节中提到的所有业务需求，实现了一个小型的票务管理系统。这个实现涵盖了顾客、客服代表和管理员的票务 CRUD 操作。此外，我们的实现满足了业务需求，比如为什么客服代表不能一次删除超过三张票。
