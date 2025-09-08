# 第二章：服务器端开发

Java EE 可以看作是专为服务器端开发而设计的。大多数 API 都针对服务器端处理和管理非常强大。

本章将为您提供一些作为 Java EE 开发者可能会遇到的一些常见和有用的场景，并展示您应该如何处理它们。

在本章中，我们将介绍以下菜谱：

+   使用 CDI 注入上下文和依赖

+   使用 Bean Validation 进行数据验证

+   使用 Servlet 进行请求和响应管理

+   使用服务器推送来预先提供对象

+   使用 EJB 和 JTA 进行事务管理

+   使用 EJB 处理并发

+   使用 JPA 进行智能数据持久化

+   使用 EJB 和 JPA 进行数据缓存

+   使用批处理

# 使用 CDI 注入上下文和依赖

Java EE 上下文和依赖注入（CDI）是 Java EE 范围内最重要的 API 之一。自 Java EE 6 引入以来，它现在对许多其他 API 产生了重大影响。

在这个菜谱中，您将学习如何以几种不同的方式和情况使用 CDI。

# 准备就绪

首先，让我们添加所需的依赖项：

```java
<dependency>
    <groupId>javax.enterprise</groupId>
    <artifactId>cdi-api</artifactId>
    <version>2.0</version>
    <scope>provided</scope>
</dependency> 
<dependency>
    <groupId>javax</groupId>
    <artifactId>javaee-web-api</artifactId>
    <version>7.0</version>
    <scope>provided</scope>
</dependency>
```

# 如何做到这一点...

1.  我们将构建一个基于 JAX-RS 的应用程序，因此我们将首先准备应用程序以执行：

```java
@ApplicationPath("webresources")
public class Application extends javax.ws.rs.core.Application {
}
```

1.  然后，我们创建一个`User`应用程序作为我们的主要对象：

```java
public class User implements Serializable {

    private String name;
    private String email;

    //DO NOT FORGET TO ADD THE GETTERS AND SETTERS
}
```

我们的`User`类没有默认构造函数，所以当 CDI 尝试注入它时不知道如何构建这个类。因此，我们创建一个工厂类，并在其方法上使用`@Produces`注解：

```java
public class UserFactory implements Serializable{

    @Produces
    public User getUser() {
        return new User("Elder Moraes", "elder@eldermoraes.com");
    }

}
```

1.  让我们创建一个枚举来列出我们的配置类型：

```java
public enum ProfileType {
    ADMIN, OPERATOR;
}
```

1.  在这里，我们创建一个自定义注解：

```java
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER})
public @interface Profile {
    ProfileType value();
}
```

1.  将它们添加到接口中以原型化用户配置文件行为：

```java
public interface UserProfile {
    ProfileType type();
}
```

现在我们已经定义了配置列表及其与用户的关联行为，我们可以为管理员配置文件提供一个适当的实现：

```java
@Profile(ProfileType.ADMIN)
public class ImplAdmin implements UserProfile{

    @Override
    public ProfileType type() {
        System.out.println("User is admin");
        return ProfileType.ADMIN;
    }   
}
```

同样，也可以为操作员配置文件做同样的事情：

```java
@Profile(ProfileType.OPERATOR)
@Default
public class ImplOperator implements UserProfile{

    @Override
    public ProfileType type() {
        System.out.println("User is operator");
        return ProfileType.OPERATOR;
    }   
}
```

1.  然后，我们通过将其将要使用的所有对象注入其中来创建一个 REST 端点：

```java
@Path("userservice/")
@RequestScoped
public class UserService {

    @Inject
    private User user;

    @Inject
    @Profile(ProfileType.ADMIN)
    private UserProfile userProfileAdmin;

    @Inject
    @Profile(ProfileType.OPERATOR)
    private UserProfile userProfileOperator;

    @Inject
    private UserProfile userProfileDefault;

    @Inject
    private Event<User> userEvent;

    ...
```

1.  这个方法通过 CDI 注入用户并将其发送到结果页面：

```java
    @GET
    @Path("getUser")
    public Response getUser(@Context HttpServletRequest request, 
            @Context HttpServletResponse response) 
            throws ServletException, IOException{

        request.setAttribute("result", user);
        request.getRequestDispatcher("/result.jsp")
        .forward(request, response);
        return Response.ok().build();
    }
```

1.  这个与管理员配置文件做的是同样的事情：

```java
    @GET
    @Path("getProfileAdmin")
    public Response getProfileAdmin(@Context HttpServletRequest request, 
            @Context HttpServletResponse response) 
            throws ServletException, IOException{

            request.setAttribute("result", 
            fireUserEvents(userProfileAdmin.type()));
             request.getRequestDispatcher("/result.jsp")
             .forward(request, response);
        return Response.ok().build();
    }
```

1.  这个与操作员配置文件做的是同样的事情：

```java
    @GET
    @Path("getProfileOperator")
    public Response getProfileOperator(@Context HttpServletRequest request, 
            @Context HttpServletResponse response) 
            throws ServletException, IOException{

            request.setAttribute("result", 
            fireUserEvents(userProfileOperator.type())); 
            request.getRequestDispatcher("/result.jsp")
            .forward(request, response);
        return Response.ok().build();
    }
```

1.  最后，我们将默认配置发送到结果页面：

```java
    @GET
    @Path("getProfileDefault")
    public Response getProfileDefault(@Context HttpServletRequest request, 
            @Context HttpServletResponse response) 
            throws ServletException, IOException{

            request.setAttribute("result", 
            fireUserEvents(userProfileDefault.type())); 
            request.getRequestDispatcher("/result.jsp")
            .forward(request, response);
            return Response.ok().build();
    }
```

1.  我们使用`fireUserEvents`方法触发一个事件和异步事件，针对之前注入的`User`对象：

```java
    private ProfileType fireUserEvents(ProfileType type){
        userEvent.fire(user);
        userEvent.fireAsync(user);
        return type;
    }

    public void sendUserNotification(@Observes User user){
        System.out.println("sendUserNotification: " + user);
    }

    public void sendUserNotificationAsync(@ObservesAsync User user){
        System.out.println("sendUserNotificationAsync: " + user);
    }
```

1.  因此，我们构建一个页面来调用每个端点方法：

```java
<body>
 <a href="http://localhost:8080/ch02-
 cdi/webresources/userservice/getUser">getUser</a>
 <br>
 <a href="http://localhost:8080/ch02-
 cdi/webresources/userservice/getProfileAdmin">getProfileAdmin</a>
 <br>
 <a href="http://localhost:8080/ch02-
 cdi/webresources/userservice/getProfileOperator">getProfileOperator</a>
 <br>
 <a href="http://localhost:8080/ch02-
 cdi/webresources/userservice/getProfileDefault">getProfileDefault</a>
</body>
```

1.  最后，我们使用表达式语言在结果页面上打印结果：

```java
<body>
    <h1>${result}</h1>
    <a href="javascript:window.history.back();">Back</a>
</body>
```

# 它是如何工作的...

好吧，上一节中发生了很多事情！我们首先应该看看`@Produces`注解。这是一个 CDI 注解，它告诉服务器：“嘿！这个方法知道如何构建一个 User 对象。”

由于我们没有为`User`类创建默认构造函数，因此我们的工厂中的`getUser`方法将被注入到我们的上下文中作为其中一个。

第二个注解是我们自定义的`@Profile`注解，它以我们的枚举`ProfileType`作为参数。它是`UserProfile`对象的限定符。

现在，让我们看看这些声明：

```java
@Profile(ProfileType.ADMIN)
public class ImplAdmin implements UserProfile{
   ...
}

@Profile(ProfileType.OPERATOR)
@Default
public class ImplOperator implements UserProfile{
   ...
}
```

这段代码将*教导*CDI 如何注入一个`UserProfile`对象：

+   如果对象被标注为`@Profile(ProfileType.ADMIN)`，则使用`ImplAdmin`

+   如果对象被标注为`@Profile(ProfileType.OPERATOR)`，则使用`ImplOperator`

+   如果对象没有被标注，则使用`ImplOperator`，因为它有`@Default`注解

我们可以在我们的端点声明中看到它们的作用：

```java
    @Inject
    @Profile(ProfileType.ADMIN)
    private UserProfile userProfileAdmin;

    @Inject
    @Profile(ProfileType.OPERATOR)
    private UserProfile userProfileOperator;

    @Inject
    private UserProfile userProfileDefault;
```

因此，CDI 正在帮助我们使用上下文来注入我们的`UserProfile`接口的正确实现。

看看端点方法，我们看到这个：

```java
    @GET
    @Path("getUser")
    public Response getUser(@Context HttpServletRequest request, 
            @Context HttpServletResponse response) 
            throws ServletException, IOException{

        request.setAttribute("result", user);
        request.getRequestDispatcher("/result.jsp")
        .forward(request, response);
        return Response.ok().build();
    }
```

注意，我们为我们的方法包含了`HttpServletRequest`和`HttpServletResponse`作为参数，但将它们标注为`@Context`。所以即使这不是 servlet 上下文（当我们容易访问请求和响应引用时），我们也可以要求 CDI 给我们提供适当的引用。

最后，我们有我们的用户事件引擎：

```java
    @Inject
    private Event<User> userEvent;

    ...

    private ProfileType fireUserEvents(ProfileType type){
        userEvent.fire(user);
        userEvent.fireAsync(user);
        return type;
    }

    public void sendUserNotification(@Observes User user){
        System.out.println("sendUserNotification: " + user);
    }

    public void sendUserNotificationAsync(@ObservesAsync User user){
        System.out.println("sendUserNotificationAsync: " + user);
    }
```

因此，我们使用`@Observes`和`@ObserversAsync`注解告诉 CDI：“*嘿 CDI！监视 User 对象...当有人在其上触发事件时，我想让你做点什么。*”

对于“某物”，CDI 将其理解为调用`sendUserNotification`和`sendUserNotificationAsync`方法。试试看！

显然，`@Observers`将同步执行，而`@ObservesAsync`将异步执行。

# 还有更多...

我们使用 GlassFish 5 来运行这个菜谱。你可以使用任何你想要的 Java EE 8 兼容服务器，你甚至可以在没有服务器的情况下使用 CDI 与 Java SE。看看第一章的 CDI 菜谱，*新特性和改进*。

# 参见

+   你可以在[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter02/ch02-cdi`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter02/ch02-cdi)查看这个菜谱的完整源代码

# 使用 Bean Validation 进行数据验证

你可以使用 Bean Validation 以许多不同的方式约束你的数据。在这个菜谱中，我们将使用它来验证 JSF 表单，这样我们就可以在用户尝试提交时立即进行验证，并立即避免任何无效数据。

# 准备中

首先，我们添加我们的依赖项：

```java
        <dependency>
            <groupId>javax.validation</groupId>
            <artifactId>validation-api</artifactId>
            <version>2.0.0.Final</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate.validator</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>6.0.1.Final</version>
        </dependency> 
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-web-api</artifactId>
            <version>7.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何做到...

1.  让我们创建一个将被附加到我们的 JSF 页面的`User`对象：

```java
@Named
@RequestScoped
public class User {

    @NotBlank (message = "Name should not be blank")
    @Size (min = 4, max = 10,message = "Name should be between 
    4 and 10 characters")
    private String name;

    @Email (message = "Invalid e-mail format")
    @NotBlank (message = "E-mail shoud not be blank")
    private String email;

    @PastOrPresent (message = "Created date should be 
    past or present")
    @NotNull (message = "Create date should not be null")
    private LocalDate created;

    @Future (message = "Expires should be a future date")
    @NotNull (message = "Expires should not be null")
    private LocalDate expires;

    //DO NOT FORGET TO IMPLEMENT THE GETTERS AND SETTERS

    ...
```

1.  然后，我们定义一个方法，当所有数据都有效时将触发：

```java
    public void valid(){
        FacesContext
                .getCurrentInstance()
                .addMessage(
                    null, 
                    new FacesMessage(FacesMessage.SEVERITY_INFO,
                    "Your data is valid", ""));
    }
```

1.  现在我们 JSF 页面引用了每个`User`类声明的字段：

```java
<h:body>
 <h:form>
 <h:outputLabel for="name" value="Name" />
 <h:inputText id="name" value="#{user.name}" />
 <br/>
 <h:outputLabel for="email" value="E-mail" />
 <h:inputText id="email" value="#{user.email}" />
 <br/>
 <h:outputLabel for="created" value="Created" />
 <h:inputText id="created" value="#{user.created}">
     <f:convertDateTime type="localDate" pattern="dd/MM/uuuu" /> 
 </h:inputText>
 <br/>
 <h:outputLabel for="expire" value="Expire" />
 <h:inputText id="expire" value="#{user.expires}">
     <f:convertDateTime type="localDate" pattern="dd/MM/uuuu" /> 
 </h:inputText>
 <br/>
 <h:commandButton value="submit" type="submit" action="#{user.valid()}" />
 </h:form>
</h:body>
```

现在，如果你运行这段代码，一旦你点击提交按钮，你将得到所有字段的验证。试试看！

# 它是如何工作的...

让我们检查每个声明的约束：

```java
    @NotBlank (message = "Name should not be blank")
    @Size (min = 4, max = 10,message = "Name should be between 
           4 and 10 characters")
    private String name;
```

`@NotBlank`注解不仅拒绝 null 值，还拒绝空白值，而`@Size`则不言自明：

```java
    @Email (message = "Invalid e-mail format")
    @NotBlank (message = "E-mail shoud not be blank")
    private String email;
```

`@Email`约束将检查电子邮件字符串格式：

```java
    @PastOrPresent (message = "Created date should be past or present")
    @NotNull (message = "Create date should not be null")
    private LocalDate created;
```

`@PastOrPresent`将`LocalDate`约束为过去或直到当前日期。它不能在未来。

在这里，我们不能使用`@NotBlank`，因为没有空白日期，只有 null，所以我们使用`@NotNull`来避免它：

```java
    @Future (message = "Expires should be a future date")
    @NotNull (message = "Expires should not be null")
    private LocalDate expires;
```

这与上一个相同，但约束条件是未来的日期。

在我们的 UI 中，有两个地方值得仔细查看：

```java
 <h:inputText id="created" value="#{user.created}">
     <f:convertDateTime type="localDate" pattern="dd/MM/uuuu" /> 
 </h:inputText>

 ...

 <h:inputText id="expire" value="#{user.expires}">
     <f:convertDateTime type="localDate" pattern="dd/MM/uuuu" /> 
 </h:inputText>
```

我们使用 `convertDateTime` 自动将输入到 `inputText` 中的数据转换为 `dd/MM/uuuu` 格式。

# 参见

+   你可以在 [`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter02/ch02-beanvalidation`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter02/ch02-beanvalidation) 查看这个配方的完整源代码。

# 使用 servlet 进行请求和响应管理

Servlet API 是在 Java EE 存在之前创建的——实际上是在 J2EE 存在之前！它在 1999 年的 J2EE 1.2（Servlet 2.2）中成为 EE 的一部分。

这是一个处理请求/响应上下文的有力工具，这个配方将展示如何操作的示例。

# 准备工作

让我们添加我们的依赖项：

```java
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.0-b05</version>
            <scope>provided</scope>
        </dependency> 
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-web-api</artifactId>
            <version>7.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何操作...

1.  让我们为我们的配方创建一个 `User` 类：

```java
public class User {

    private String name;
    private String email;

    //DO NOT FORGET TO IMPLEMENT THE GETTERS AND SETTERS

}
```

1.  然后是我们的 servlet：

```java
@WebServlet(name = "UserServlet", urlPatterns = {"/UserServlet"})
public class UserServlet extends HttpServlet {

    private User user;

    @PostConstruct
    public void instantiateUser(){
        user = new User("Elder Moraes", "elder@eldermoraes.com");
    }

   ...
```

我们在 `instantiateUser()` 方法上使用 `@PostConstruct` 注解。它告诉服务器，每当这个 servlet 被构建（一个新的实例被创建）时，它可以运行这个方法。

1.  我们还实现了 `init()` 和 `destroy()` 超类方法：

```java
    @Override
    public void init() throws ServletException {
        System.out.println("Servlet " + this.getServletName() + 
                           " has started");
    }

    @Override
    public void destroy() {
        System.out.println("Servlet " + this.getServletName() + 
                           " has destroyed");
    }
```

1.  我们还实现了 `doGet()` 和 `doPost()`：

```java
    @Override
    protected void doGet(HttpServletRequest request, 
    HttpServletResponse response)
            throws ServletException, IOException {
        doRequest(request, response);
    }

    @Override
    protected void doPost(HttpServletRequest request, 
    HttpServletResponse response)
            throws ServletException, IOException {
        doRequest(request, response);
    }
```

1.  `doGet()` 和 `doPost()` 都将调用我们的自定义方法 `doRequest()`：

```java
    protected void doRequest(HttpServletRequest request, 
    HttpServletResponse response)
            throws ServletException, IOException {
        response.setContentType("text/html;charset=UTF-8");
        try (PrintWriter out = response.getWriter()) {
            out.println("<html>");
            out.println("<head>");
            out.println("<title>Servlet UserServlet</title>");
            out.println("</head>");
            out.println("<body>");
            out.println("<h2>Servlet UserServlet at " + 
                         request.getContextPath() + "</h2>");
            out.println("<h2>Now: " + new Date() + "</h2>");
            out.println("<h2>User: " + user.getName() + "/" + 
                        user.getEmail() + "</h2>");
            out.println("</body>");
            out.println("</html>");
        }
    }
```

1.  最后，我们有一个网页可以调用我们的 servlet：

```java
    <body>
        <a href="<%=request.getContextPath()%>/UserServlet">
        <%=request.getContextPath() %>/UserServlet</a>
    </body>
```

# 它是如何工作的...

Java EE 服务器本身将根据调用者使用的 HTTP 方法调用 `doGet()` 或 `doPost()` 方法。在我们的配方中，我们将它们都重定向到同一个 `doRequest()` 方法。

`init()` 方法属于由服务器管理的 servlet 生命周期，并在 servlet 实例化后作为第一个方法执行。

`destroy()` 方法也属于 servlet 生命周期，并在实例释放前作为最后一个方法执行。

# 还有更多...

`init()` 的行为类似于 `@PostConstruct`，但最后一个是在 `init()` 之前执行的，所以使用两者时要记住这一点。

`@PostConstruct` 在默认构造函数之后执行。

使用 `destroy()` 方法时要小心，避免持有任何内存引用；否则，你可能会搞乱 servlet 生命周期并遇到内存泄漏。

# 参见

+   在 [`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter02/ch02-servlet`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter02/ch02-servlet) 查看这个配方的完整源代码。

# 使用服务器推送提前使对象可用

Servlet 4.0 最重要的新特性之一是 HTTP/2.0 支持。它带来了另一个酷且可靠的功能——服务器推送。

这个配方将展示如何在过滤器中使用服务器推送并在每个请求中推送所需的资源。

# 准备工作

我们首先应该添加所需的依赖项：

```java
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.0-b07</version>
            <scope>provided</scope>
        </dependency> 
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-web-api</artifactId>
            <version>7.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何操作...

1.  我们首先创建 `UserServlet`，它调用 `user.jsp`：

```java
@WebServlet(name = "UserServlet", urlPatterns = {"/UserServlet"})
public class UserServlet extends HttpServlet {

    protected void doRequest(HttpServletRequest request, 
                             HttpServletResponse response)
            throws ServletException, IOException {
        request.getRequestDispatcher("/user.jsp")
        .forward(request, response);
        System.out.println("Redirected to user.jsp");
    }

    @Override
    protected void doGet(HttpServletRequest request, 
                         HttpServletResponse response)
            throws ServletException, IOException {
        doRequest(request, response);
    }

    @Override
    protected void doPost(HttpServletRequest request, 
                          HttpServletResponse response)
            throws ServletException, IOException {
        doRequest(request, response);
    }
}
```

1.  我们还用 `ProfileServlet` 做同样的操作，但通过调用 `profile.jsp`：

```java
@WebServlet(name = "ProfileServlet", urlPatterns = {"/ProfileServlet"})
public class ProfileServlet extends HttpServlet {

    protected void doRequest(HttpServletRequest request, 
                             HttpServletResponse response)
            throws ServletException, IOException {
        request.getRequestDispatcher("/profile.jsp").
        forward(request, response);
        System.out.println("Redirected to profile.jsp");
    }

    @Override
    protected void doGet(HttpServletRequest request, 
                         HttpServletResponse response)
            throws ServletException, IOException {
        doRequest(request, response);
    }

    @Override
    protected void doPost(HttpServletRequest request, 
                          HttpServletResponse response)
            throws ServletException, IOException {
        doRequest(request, response);
    }
}
```

1.  然后我们创建一个将在每个请求上执行的过滤器（`urlPatterns = {"/*"}`）：

```java
@WebFilter(filterName = "PushFilter", urlPatterns = {"/*"})
public class PushFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, 
    ServletResponse response,
            FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest httpReq = (HttpServletRequest)request;
        PushBuilder builder = httpReq.newPushBuilder();

        if (builder != null){
            builder
                .path("resources/javaee-logo.png")
                .path("resources/style.css")
                .path("resources/functions.js")
                .push(); 
            System.out.println("Resources pushed");
        }

        chain.doFilter(request, response);

    }   
}
```

1.  在这里，我们创建一个页面来调用我们的 servlets：

```java
<body>
 <a href="UserServlet">User</a>
 <br/>
 <a href="ProfileServlet">Profile</a>
</body>
```

1.  以下是 servlets 调用的页面。首先是`user.jsp`页面：

```java
    <head>
        <meta http-equiv="Content-Type" content="text/html; 
         charset=UTF-8">
        <link rel="stylesheet" type="text/css"   
         href="resources/style.css">
        <script src="img/functions.js"></script>
        <title>User Push</title>
    </head>

    <body>
        <h1>User styled</h1>
        <img src="img/javaee-logo.png">
        <br />
        <button onclick="message()">Message</button>
        <br />
        <a href="javascript:window.history.back();">Back</a>
    </body>
```

1.  其次，调用`profile.jsp`页面：

```java
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <link rel="stylesheet" type="text/css" href="resources/style.css">
        <script src="img/functions.js"></script>
        <title>User Push</title>
    </head>

    <body>
        <h1>Profile styled</h1>
        <img src="img/javaee-logo.png">
        <br />
        <button onclick="message()">Message</button>
        <br />
        <a href="javascript:window.history.back();">Back</a>
    </body>
```

# 它是如何工作的...

在 HTTP/1.0 下运行的 Web 应用程序在找到图像文件、CSS 文件和其他渲染网页所需的资源引用时向服务器发送请求。

使用 HTTP/2.0 您仍然可以这样做，但现在您可以做得更好：服务器现在可以预先推送资源，避免不必要的新的请求，减少服务器负载，并提高性能。

在此菜谱中，我们的资源由以下表示：

```java
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <link rel="stylesheet" type="text/css" href="resources/style.css">
        <script src="img/functions.js"></script>
```

推送发生在我们过滤器的这个部分：

```java
        HttpServletRequest httpReq = (HttpServletRequest)request;
        PushBuilder builder = httpReq.newPushBuilder();

        if (builder != null){
            builder
                .path("resources/javaee-logo.png")
                .path("resources/style.css")
                .path("resources/functions.js")
                .push(); 
            System.out.println("Resources pushed");
        }
```

因此，当浏览器需要这些资源来渲染网页时，它们已经可用。

# 还有更多...

注意，您的浏览器需要支持服务器推送功能；否则，您的页面将像往常一样工作。所以请确保在使用`PushBuilder`之前检查它是否为 null，并确保所有用户都将拥有一个正常工作的应用程序。

注意，JSF 2.3 是基于服务器推送功能构建的，所以如果您只是将您的 JSF 应用程序迁移到 Java EE 8 兼容的服务器，您将免费获得其性能提升！

# 相关内容

+   在[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter02/ch02-serverpush`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter02/ch02-serverpush)查看此菜谱的完整源代码

# 使用 EJB 和 JTA 进行事务管理

Java 事务 API，或称 JTA，是一个允许在 Java EE 环境中进行分布式事务的 API。当您将事务管理委托给服务器时，它最为强大。

此菜谱将向您展示如何做到这一点！

# 准备工作

首先，添加依赖项：

```java
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>4.3.1.Final</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
            <version>1.3</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-web-api</artifactId>
            <version>7.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.openejb</groupId>
            <artifactId>openejb-core</artifactId>
            <version>4.7.4</version>
            <scope>test</scope>
        </dependency>
```

# 如何做到这一点...

1.  首先，我们需要创建我们的持久化单元（在`persistence.xml`中）：

```java
    <persistence-unit name="ch02-jta-pu" transaction-type="JTA">
        <provider>org.hibernate.ejb.HibernatePersistence</provider>

        <jta-data-source>userDb</jta-data-source>
        <non-jta-data-source>userDbNonJta</non-jta-data-source>

        <exclude-unlisted-classes>false</exclude-unlisted-classes>

        <properties>
            <property name="javax.persistence.schema-
             generation.database.action" 
             value="create"/>
        </properties>
    </persistence-unit>
```

1.  然后，我们创建一个`User`类作为实体（`@Entity`）：

```java
@Entity
public class User implements Serializable {

    private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String name;
    private String email;

    protected User() {
    } 

    public User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    //DO NOT FORGET TO IMPLEMENT THE GETTERS AND SETTERS
}
```

1.  我们还需要一个 EJB 来执行对`User`实体的操作：

```java
@Stateful
public class UserBean {

    @PersistenceContext(unitName = "ch02-jta-pu", 
    type = PersistenceContextType.EXTENDED)
    private EntityManager em;

    public void add(User user){
        em.persist(user);
    }

    public void update(User user){
        em.merge(user);
    }

    public void remove(User user){
        em.remove(user);
    }

    public User findById(Long id){
        return em.find(User.class, id);
    }
}
```

1.  然后我们创建我们的单元测试：

```java
public class Ch02JtaTest {

    private EJBContainer ejbContainer;

    @EJB
    private UserBean userBean;

    public Ch02JtaTest() {
    }

    @Before
    public void setUp() throws NamingException {
        Properties p = new Properties();
        p.put("userDb", "new://Resource?type=DataSource");
        p.put("userDb.JdbcDriver", "org.hsqldb.jdbcDriver");
        p.put("userDb.JdbcUrl", "jdbc:hsqldb:mem:userdatabase");

        ejbContainer = EJBContainer.createEJBContainer(p);
        ejbContainer.getContext().bind("inject", this);
    }

    @After
    public void tearDown() {
        ejbContainer.close();
    }

    @Test
    public void validTransaction() throws Exception{
        User user = new User(null, "Elder Moraes", 
                             "elder@eldermoraes.com");

        userBean.add(user);
        user.setName("John Doe");
        userBean.update(user);

        User userDb = userBean.findById(1L);
        assertEquals(userDb.getName(), "John Doe");

    }

}
```

# 它是如何工作的...

此菜谱中 JTA 的关键代码行就在这里：

```java
<persistence-unit name="ch02-jta-pu" transaction-type="JTA">
```

当您使用`transaction-type='JTA'`时，您是在告诉服务器，它应该处理在此上下文中进行的所有事务。如果您使用`RESOURCE-LOCAL`，则表示您正在处理事务：

```java
    @Test
    public void validTransaction() throws Exception{
        User user = new User(null, "Elder Moraes", 
        "elder@eldermoraes.com");

        userBean.add(user);
        user.setName("John Doe");
        userBean.update(user);

        User userDb = userBean.findById(1L);
        assertEquals(userDb.getName(), "John Doe");

    }
```

`UserBean`的每个被调用方法都会启动一个事务以完成，如果在事务活跃期间出现任何问题，则会回滚；如果事务顺利完成，则会提交。

# 还有更多...

另一段重要的代码如下：

```java
@Stateful
public class UserBean {

    @PersistenceContext(unitName = "ch02-jta-pu", 
                        type = PersistenceContextType.EXTENDED)
    private EntityManager em;

    ...
}
```

因此，我们在这里将我们的`PersistenceContext`定义为`EXTENDED`。这意味着此持久化上下文绑定到`@Stateful`豆，直到它从容器中移除。

另一个选项是`TRANSACTION`，这意味着持久化上下文将仅在事务期间存在。

# 相关内容

+   在[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter02/ch02-jta`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter02/ch02-jta)查看此菜谱的完整源代码

# 使用 EJB 处理并发

并发管理是 Java EE 服务器提供的最大优势之一。你可以依赖一个现成的环境来处理这个棘手的话题。

此菜谱将向您展示如何设置您的 bean 以使用它！

# 准备工作

只需将 Java EE 依赖项添加到您的项目中：

```java
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-web-api</artifactId>
            <version>7.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何操作...

此菜谱将展示三个场景。

在第一个场景中，`LockType`在类级别定义：

```java
@Singleton
@ConcurrencyManagement(ConcurrencyManagementType.CONTAINER)
@Lock(LockType.READ)
@AccessTimeout(value = 10000)
public class UserClassLevelBean {

    private int userCount;

    public int getUserCount() {
        return userCount;
    }

    public void addUser(){
        userCount++;
    }

}
```

在第二个场景中，`LockType`在方法级别定义：

```java
@Singleton
@ConcurrencyManagement(ConcurrencyManagementType.CONTAINER)
@AccessTimeout(value = 10000)
public class UserMethodLevelBean {

    private int userCount;

    @Lock(LockType.READ)
    public int getUserCount(){
        return userCount;
    }

    @Lock(LockType.WRITE)
    public void addUser(){
        userCount++;
    }
}
```

第三个场景是一个自管理 bean：

```java
@Singleton
@ConcurrencyManagement(ConcurrencyManagementType.BEAN)
public class UserSelfManagedBean {

    private int userCount;

    public int getUserCount() {
        return userCount;
    }

    public synchronized void addUser(){
        userCount++;
    }
}
```

# 它是如何工作的...

首先看看以下内容：

```java
@ConcurrencyManagement(ConcurrencyManagementType.CONTAINER)
```

这完全是多余的！单例 bean 默认由容器管理，因此您不需要指定它们。

单例设计用于并发访问，因此它们是此菜谱的完美用例。

现在，让我们检查在类级别定义的`LockType`：

```java
@Lock(LockType.READ)
@AccessTimeout(value = 10000)
public class UserClassLevelBean {
    ...
}
```

当我们在类级别使用`@Lock`注解时，通知的`LockType`将用于所有类方法。

在这种情况下，`LockType.READ`意味着许多客户端可以同时访问一个资源。这在读取数据时很常见。

在锁定的情况下，`LockType`将使用`@AccessTimeout`注解定义的时间来决定是否运行到超时。

现在，让我们检查在方法级别定义的`LockType`：

```java
    @Lock(LockType.READ)
    public int getUserCount(){
        return userCount;
    }

    @Lock(LockType.WRITE)
    public void addUser(){
        userCount++;
    }
```

因此，我们基本上是在说`getUserCount()`可以同时被许多用户访问（`LockType.READ`），而`addUser()`则每次只能被一个用户访问（`LockType.WRITE`）。

最后一个案例是自管理 bean：

```java
@ConcurrencyManagement(ConcurrencyManagementType.BEAN)
public class UserSelfManagedBean{

    ...

    public synchronized void addUser(){
        userCount++;
    }

    ...
}
```

在这种情况下，你必须在你自己的代码中管理你的 bean 的所有并发问题。我们以同步限定符为例。

# 还有更多...

除非你真的真的需要，否则不要使用自管理 bean。Java EE 容器（非常好）设计用来以非常高效和优雅的方式完成这项工作。

# 参见

+   在[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter02/ch02-ejb-concurrency`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter02/ch02-ejb-concurrency)查看此菜谱的完整源代码

# 使用 JPA 进行智能数据持久化

Java 持久化 API 是一个规范，它描述了使用 Java EE 管理关系数据库的接口。

它简化了数据操作，并大大减少了为其编写的代码，尤其是如果您习惯于 SQL ANSI。

此菜谱将向您展示如何使用它来持久化您的数据。

# 准备工作

让我们首先添加所需的依赖项：

```java
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>4.3.1.Final</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
            <version>1.3</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-web-api</artifactId>
            <version>7.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.openejb</groupId>
            <artifactId>openejb-core</artifactId>
            <version>4.7.4</version>
            <scope>test</scope>
        </dependency>
```

# 如何操作...

1.  让我们先创建一个实体（您可以将其视为一个表）：

```java
@Entity
public class User implements Serializable {

    private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String name;
    private String email;

    protected User() {
    } 

    public User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    //DO NOT FORGET TO IMPLEMENT THE GETTERS AND SETTERS
}
```

1.  在这里，我们声明我们的持久化单元（在`persistence.xml`中）：

```java
    <persistence-unit name="ch02-jpa-pu" transaction-type="JTA">
        <provider>org.hibernate.ejb.HibernatePersistence</provider>
        <jta-data-source>userDb</jta-data-source>

        <exclude-unlisted-classes>false</exclude-unlisted-classes>

        <properties>
            <property name="javax.persistence.schema-
             generation.database.action" 
             value="create"/>
        </properties>
    </persistence-unit>
```

1.  然后我们创建一个会话 bean 来管理我们的数据：

```java
@Stateless
public class UserBean {

    @PersistenceContext(unitName = "ch02-jpa-pu", 
    type = PersistenceContextType.TRANSACTION)
    private EntityManager em;

    public void add(User user){
        em.persist(user);
    }

    public void update(User user){
        em.merge(user);
    }

    public void remove(User user){
        em.remove(user);
    }

    public User findById(Long id){
        return em.find(User.class, id);
    }
}
```

1.  而在这里，我们使用单元测试来尝试它：

```java
public class Ch02JpaTest {

    private EJBContainer ejbContainer;

    @EJB
    private UserBean userBean;

    public Ch02JpaTest() {
    }

    @Before
    public void setUp() throws NamingException {
        Properties p = new Properties();
        p.put("userDb", "new://Resource?type=DataSource");
        p.put("userDb.JdbcDriver", "org.hsqldb.jdbcDriver");
        p.put("userDb.JdbcUrl", "jdbc:hsqldb:mem:userdatabase");

        ejbContainer = EJBContainer.createEJBContainer(p);
        ejbContainer.getContext().bind("inject", this);
    }

    @After
    public void tearDown() {
        ejbContainer.close();
    }

    @Test
    public void persistData() throws Exception{
        User user = new User(null, "Elder Moraes", 
        "elder@eldermoraes.com");

        userBean.add(user);
        user.setName("John Doe");
        userBean.update(user);

        User userDb = userBean.findById(1L);
        assertEquals(userDb.getName(), "John Doe");

    }

}
```

# 它是如何工作的...

让我们分解我们的**持久化单元**（**pu**）。

这行代码定义了 pu 名称和使用的交易类型：

```java
<persistence-unit name="ch02-jpa-pu" transaction-type="JTA">
```

以下行显示了 JPA 使用的提供者：

```java
<provider>org.hibernate.ejb.HibernatePersistence</provider>
```

是通过 JNDI 访问的数据源名称：

```java
<jta-data-source>userDb</jta-data-source>
```

这行代码使得所有你的实体都可以用于这个 pu，因此你不需要为每个实体声明：

```java
<exclude-unlisted-classes>false</exclude-unlisted-classes>
```

这个块允许在不存在时创建数据库对象：

```java
        <properties>
            <property name="javax.persistence.schema-
             generation.database.action" 
             value="create"/>
        </properties>
```

现在，让我们看看`UserBean`：

```java
@Stateless
public class UserBean {

    @PersistenceContext(unitName = "ch02-jpa-pu", 
                        type = PersistenceContextType.TRANSACTION)
    private EntityManager em;

   ...

}
```

`EntityManager`是负责 bean 与数据源之间接口的对象。它通过`@PersistenceContext`注解绑定到上下文中。

我们如下检查`EntityManager`操作：

```java
    public void add(User user){
        em.persist(user);
    }
```

`persist()`方法用于将新数据添加到数据源。执行结束时，对象被附加到上下文中：

```java
    public void update(User user){
        em.merge(user);
    }
```

`merge()`方法用于在数据源上更新现有数据。首先在上下文中找到对象，然后在数据库中更新，并使用新状态附加到上下文中：

```java
    public void remove(User user){
        em.remove(user);
    }
```

`remove()`方法，猜猜看它是做什么的？

```java
    public User findById(Long id){
        return em.find(User.class, id);
    }
```

最后，`find()`方法使用`id`参数搜索具有相同 ID 的数据库对象。这就是为什么 JPA 要求你的实体必须使用`@Id`注解声明一个 ID。

# 参见

+   在[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter02/ch02-jpa`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter02/ch02-jpa)查看这个菜谱的完整源代码

# 使用 EJB 和 JPA 进行数据缓存

了解如何为你的应用程序构建一个简单且本地的缓存是一项重要的技能。它可能对某些数据访问性能有重大影响，而且做起来相当简单。

这个菜谱将向你展示如何做。

# 准备工作

简单地为你的项目添加一个 Java EE 依赖项：

```java
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-web-api</artifactId>
            <version>7.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何做...

1.  让我们创建一个`User`类作为我们的缓存对象：

```java
public class User {

    private String name;
    private String email;

    //DO NOT FORGET TO IMPLEMENT THE GETTERS AND SETTERS
}
```

1.  然后创建一个单例来保存我们的用户列表缓存：

```java
@Singleton
@Startup
public class UserCacheBean {

    protected Queue<User> cache = null;

    @PersistenceContext
    private EntityManager em;

    public UserCacheBean() {
    }

    protected void loadCache() {
        List<User> list = em.createQuery("SELECT u FROM USER 
                                         as u").getResultList();

        list.forEach((user) -> {
            cache.add(user);
        });
    }

    @Lock(LockType.READ)
    public List<User> get() {
        return cache.stream().collect(Collectors.toList());
    }

    @PostConstruct
    protected void init() {
        cache = new ConcurrentLinkedQueue<>();
        loadCache();
    }
}
```

# 它是如何工作的...

让我们先了解我们的 bean 声明：

```java
@Singleton
@Startup
public class UserCacheBean {

    ...

    @PostConstruct
    protected void init() {
        cache = new ConcurrentLinkedQueue<>();
        loadCache();
    }
}
```

我们使用单例模式，因为它在应用程序上下文中只有一个实例。这正是我们想要的数据缓存的方式，因为我们不希望允许不同数据被共享的可能性。

还要注意，我们使用了`@Startup`注解。它告诉服务器，一旦加载了这个 bean，就应该执行它，并且使用带有`@PostConstruct`注解的方法。

因此，我们利用启动时间来加载我们的缓存：

```java
    protected void loadCache() {
        List<User> list = em.createQuery("SELECT u FROM USER 
                                         as u").getResultList();

        list.forEach((user) -> {
            cache.add(user);
        });
    }
```

现在，让我们检查持有我们缓存的对象：

```java
protected Queue<User> cache = null;

...

cache = new ConcurrentLinkedQueue<>();
```

`ConcurrentLinkedQueue`是一个列表，它有一个主要目的——在线程安全的环境中由多个进程访问。这正是我们所需要的，而且它在其成员访问上提供了很好的性能。

最后，让我们检查对我们的数据缓存的访问：

```java
    @Lock(LockType.READ)
    public List<User> get() {
        return cache.stream().collect(Collectors.toList());
    }
```

我们使用`LockType.READ`对`get()`方法进行了注释，这样它就告诉并发管理器，它可以以线程安全的方式同时被多个进程访问。

# 还有更多...

如果你需要在你的应用程序中使用大而复杂的缓存，你应该使用一些企业级缓存解决方案以获得更好的结果。

# 参见

+   在[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter02/ch02-datacache`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter02/ch02-datacache)查看此菜谱的完整源代码

# 使用批处理

批处理是本章的最后一个菜谱。在企业环境中运行后台任务是实用且重要的技能。

你可以用它来批量处理数据，或者只是将其从 UI 流程中分离出来。这个菜谱将向你展示如何做到这一点。

# 准备工作

让我们添加我们的依赖项：

```java
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>5.2.10.Final</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>7.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何做到这一点...

1.  我们首先定义我们的持久化单元：

```java
  <persistence-unit name="ch02-batch-pu" >
    <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
    <jta-data-source>java:app/userDb</jta-data-source>
    <exclude-unlisted-classes>false</exclude-unlisted-classes>
    <properties>
      <property name="javax.persistence.schema- 
       generation.database.action" 
       value="create"/>
      <property name="hibernate.transaction.jta.platform" 
       value="org.hibernate.service.jta.platform
       .internal.SunOneJtaPlatform"/>
    </properties>
  </persistence-unit>
```

1.  然后我们声明一个`User`实体：

```java
@Entity
@Table(name = "UserTab")
public class User implements Serializable {

    private static final long serialVersionUID = 1L;

    @Id
    @NotNull
    private Integer id;

    private String name;

    private String email;

    public User() {
    }

    //DO NOT FORGET TO IMPLEMENT THE GETTERS AND SETTERS
}
```

1.  在这里我们创建一个工作读取器：

```java
@Named
@Dependent
public class UserReader extends AbstractItemReader {

    private BufferedReader br;

    @Override
    public void open(Serializable checkpoint) throws Exception {
        br = new BufferedReader(
                new InputStreamReader(
                        Thread.currentThread()
                        .getContextClassLoader()
                        .getResourceAsStream
                        ("META-INF/user.txt")));
    }

    @Override
    public String readItem() {
        String line = null;

        try {
            line = br.readLine();
        } catch (IOException ex) {
            System.out.println(ex.getMessage());
        }

        return line;
    }
}
```

1.  然后我们创建一个工作处理器：

```java
@Named
@Dependent
public class UserProcessor implements ItemProcessor {

    @Override
    public User processItem(Object line) {
        User user = new User();

        StringTokenizer tokens = new StringTokenizer((String)
        line, ",");
        user.setId(Integer.parseInt(tokens.nextToken()));
        user.setName(tokens.nextToken());
        user.setEmail(tokens.nextToken());

        return user;
    }
}
```

1.  然后我们创建一个工作写入器：

```java
@Named
@Dependent
public class UserWriter extends AbstractItemWriter {

    @PersistenceContext
    EntityManager entityManager;

    @Override
    @Transactional
    public void writeItems(List list) {
        for (User user : (List<User>) list) {
            entityManager.persist(user);
        }
    }
}
```

处理器、读取器和写入器由位于`META-INF.batch-jobs`的`acess-user.xml`文件引用：

```java
<?xml version="1.0" encoding="windows-1252"?>
<job id="userAccess" 

     version="1.0">
    <step id="loadData">
        <chunk item-count="3">
            <reader ref="userReader"/>
            <processor ref="userProcessor"/>
            <writer ref="userWriter"/>
        </chunk>
    </step>
</job>
```

1.  最后，我们创建一个用于与批处理引擎交互的 bean：

```java
@Named
@RequestScoped
public class UserBean {

    @PersistenceContext
    EntityManager entityManager;

    public void run() {
        try {
            JobOperator job = BatchRuntime.getJobOperator();
            long jobId = job.start("acess-user", new Properties());
            System.out.println("Job started: " + jobId);
        } catch (JobStartException ex) {
            System.out.println(ex.getMessage());
        }
    }

    public List<User> get() {
        return entityManager
                .createQuery("SELECT u FROM User as u", User.class)
                .getResultList();
    }
}
```

为了本例的目的，我们将使用一个 JSF 页面来运行工作并加载数据：

```java
<h:body>
 <h:form>
 <h:outputLabel value="#{userBean.get()}" />
 <br />
 <h:commandButton value="Run" action="index" actionListener="#{userBean.run()}"/>
 <h:commandButton value="Reload" action="index"/>
 </h:form>
</h:body>
```

在 Java EE 服务器上运行它，点击运行按钮，然后点击重新加载按钮。

# 它是如何工作的...

为了理解正在发生的事情：

1.  `UserReader`扩展了具有两个关键方法的`AbstractItemReader`类：`open()`和`readItem()`。在我们的案例中，第一个打开`META-INF/user.txt`，第二个读取文件的每一行。

1.  `UserProcessor`类扩展了具有`processItem()`方法的`ItemProcessor`类。它通过`readItem()`（从`UserReader`）获取项目，生成我们想要的`User`对象。

1.  一旦所有项目都处理完毕并可用在列表（内存）中，我们使用`UserWriter`类；它扩展了`AbstractItemWriter`类并具有`writeItems`方法。在我们的案例中，我们使用它来持久化从`user.txt`文件中读取的数据。

一切准备就绪，我们只需使用`UserBean`来运行工作：

```java
    public void run() {
        try {
            JobOperator job = BatchRuntime.getJobOperator();
            long jobId = job.start("acess-user", new Properties());
            System.out.println("Job started: " + jobId);
        } catch (JobStartException ex) {
            System.out.println(ex.getMessage());
        }
    }
```

`job.start()`方法引用了`acess-user.xml`文件，使我们的读取器、处理器和写入器能够协同工作。

# 参考以下内容

+   在[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter02/ch02-batch`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter02/ch02-batch)查看此菜谱的完整源代码
