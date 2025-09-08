# 企业架构安全

本章涵盖了以下食谱：

+   使用认证进行域保护

+   通过授权授予权利

+   使用 SSL/TLS 保护数据机密性和完整性

+   使用声明式安全

+   使用程序化安全

# 简介

**安全**无疑是软件行业有史以来最热门的话题之一，而且没有理由认为这种情况会很快改变。实际上，随着时间的推移，它可能会变得更加热门。

由于所有数据都通过云传输，经过无数的服务器、链接、数据库、会话、设备等等，您至少会期望它得到良好的保护、安全，并且其完整性得到保持。

现在，最后，Java EE 有它自己的安全 API，Soteria 是其参考实现。

安全是一个值得数十本书讨论的主题；这是一个事实。但本章将涵盖一些您可能在日常项目中遇到的常见用例。

# 使用认证进行域保护

认证是用于定义谁可以访问您的域的任何过程、任务和/或策略。例如，它就像您用来进入办公室的徽章。

在应用程序中，认证最常见的使用是允许已注册的用户访问您的域。

这个食谱将向您展示如何使用简单的代码和配置来控制谁可以访问以及谁不能访问您应用程序的一些资源。

# 准备工作

我们首先添加我们的依赖项：

```java
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何做到这一点

1.  首先，我们在`web.xml`文件中进行一些配置：

```java
    <security-constraint>
        <web-resource-collection>
            <web-resource-name>CH05-Authentication</web-resource-name>
            <url-pattern>/authServlet</url-pattern>
        </web-resource-collection>
        <auth-constraint>
            <role-name>role1</role-name>
        </auth-constraint>
    </security-constraint>

    <security-role>
        <role-name>role1</role-name>
    </security-role>
```

1.  然后，我们创建一个 servlet 来处理我们的用户访问：

```java
@DeclareRoles({"role1", "role2", "role3"})
@WebServlet(name = "/UserAuthenticationServlet", urlPatterns = {"/UserAuthenticationServlet"})
public class UserAuthenticationServlet extends HttpServlet {

    private static final long serialVersionUID = 1L;

    @Inject
    private javax.security.enterprise.SecurityContext 
    securityContext;

    @Override
    public void doGet(HttpServletRequest request, 
    HttpServletResponse response) throws ServletException, 
    IOException {

        String name = request.getParameter("name");
        if (null != name || !"".equals(name)) {
            AuthenticationStatus status = 
            securityContext.authenticate(
                    request, response, 
                    AuthenticationParameters.withParams().credential
                    (new CallerOnlyCredential(name)));

            response.getWriter().write("Authentication status: " 
            + status.name() + "\n");
        }

        String principal = null;
        if (request.getUserPrincipal() != null) {
            principal = request.getUserPrincipal().getName();
        }

        response.getWriter().write("User: " + principal + "\n");
        response.getWriter().write("Role \"role1\" access: " + 
        request.isUserInRole("role1") + "\n");
        response.getWriter().write("Role \"role2\" access: " + 
        request.isUserInRole("role2") + "\n");
        response.getWriter().write("Role \"role3\" access: " + 
        request.isUserInRole("role3") + "\n");
        response.getWriter().write("Access to /authServlet? " + 
        securityContext.hasAccessToWebResource("/authServlet") + 
        "\n");
    }
}
```

1.  最后，我们创建一个将定义我们的认证策略的类：

```java
@ApplicationScoped
public class AuthenticationMechanism implements 
HttpAuthenticationMechanism {

    @Override
    public AuthenticationStatus validateRequest(HttpServletRequest 
    request, 
    HttpServletResponse response, HttpMessageContext  
    httpMessageContext) 
    throws AuthenticationException {

        if (httpMessageContext.isAuthenticationRequest()) {

            Credential credential = 
            httpMessageContext.getAuthParameters().getCredential();
            if (!(credential instanceof CallerOnlyCredential)) {
                throw new IllegalStateException("Invalid 
                mechanism");
            }

            CallerOnlyCredential callerOnlyCredential = 
            (CallerOnlyCredential) credential;

            if ("user".equals(callerOnlyCredential.getCaller())) {
                return 
                httpMessageContext.notifyContainerAboutLogin
                (callerOnlyCredential.getCaller(), new HashSet<> 
                (Arrays.asList("role1","role2")));
            } else{
                throw new AuthenticationException();
            }

        }

        return httpMessageContext.doNothing();
    }

}
```

如果您在一个 Java EE 8 兼容的服务器上运行此项目，您应该使用此 URL（假设您是在本地运行。如果不是，请进行适当的更改）：

`http://localhost:8080/ch05-authentication/UserAuthenticationServlet?name=user`

这应该会生成一个包含以下信息的页面：

```java
Authentication status: SUCCESS
User: user
Role "role1" access: true
Role "role2" access: true
Role "role3" access: false
Access to /authServlet? true
```

尝试对`name`参数进行任何更改，例如这样：

`http://localhost:8080/ch05-authentication/UserAuthenticationServlet?name=anotheruser`

然后，结果将如下所示：

```java
Authentication status: SEND_FAILURE
User: null
Role "role1" access: false
Role "role2" access: false
Role "role3" access: false
Access to /authServlet? false
```

# 它是如何工作的...

让我们把之前显示的代码分开，这样我们可以更好地理解正在发生的事情。

在`web.xml`文件中，我们创建一个安全约束：

```java
    <security-constraint>
       ...
    </security-constraint>
```

我们在它内部定义一个资源：

```java
        <web-resource-collection>
            <web-resource-name>CH05-Authentication</web-resource-name>
            <url-pattern>/authServlet</url-pattern>
        </web-resource-collection>
```

我们正在定义一个授权策略。在这种情况下，它是一个角色：

```java
        <auth-constraint>
            <role-name>role1</role-name>
        </auth-constraint>
```

现在我们有`UserAuthenticationServlet`。我们应该注意这个注解：

```java
@DeclareRoles({"role1", "role2", "role3"})
```

它定义了哪些角色是特定 servlet 上下文的一部分。

这个场景中的另一个重要角色是这个：

```java
    @Inject
    private SecurityContext securityContext;
```

在这里，我们正在请求服务器给我们一个安全上下文，以便我们可以用它来实现我们的目的。这将在一分钟内变得有意义。

然后，如果`name`参数被填充，我们将到达这一行：

```java
            AuthenticationStatus status = securityContext.authenticate(
                    request, response, withParams().credential(new 
                    CallerOnlyCredential(name)));
```

这将要求 Java EE 服务器处理一个认证。但是...基于什么？这就是我们的`HttpAuthenticationMechanism`实现发挥作用的地方。

如前所述的代码创建了`CallerOnlyCredential`，因此我们的认证机制将基于它：

```java
            Credential credential = httpMessageContext.getAuthParameters()
            .getCredential();
            if (!(credential instanceof CallerOnlyCredential)) {
                throw new IllegalStateException("Invalid mechanism");
            }

            CallerOnlyCredential callerOnlyCredential = 
           (CallerOnlyCredential) credential;
```

一旦我们有一个`credential`实例，我们可以检查用户“是否存在”：

```java
            if ("user".equals(callerOnlyCredential.getCaller())) {
                ...
            } else{
                throw new AuthenticationException();
            }
```

例如，我们刚刚比较了名称，但在实际情况下，你可以搜索数据库、LDAP 服务器等。

如果用户存在，我们将根据一些规则进行认证。

```java
return httpMessageContext.notifyContainerAboutLogin
(callerOnlyCredential.getCaller(), new HashSet<>(asList("role1","role2")));
```

在这种情况下，我们表示用户可以访问`"role1"`和`"role2"`。

认证完成后，它将返回到 servlet 并使用结果来完成过程：

```java
        response.getWriter().write("Role \"role1\" access: " + 
        request.isUserInRole("role1") + "\n");
        response.getWriter().write("Role \"role2\" access: " + 
        request.isUserInRole("role2") + "\n");
        response.getWriter().write("Role \"role3\" access: " + 
        request.isUserInRole("role3") + "\n");
        response.getWriter().write("Access to /authServlet? " + 
        securityContext.hasAccessToWebResource("/authServlet") + "\n");
```

因此，这段代码将为`"role1"`和`"role2"`打印`true`，而为`"role3"`打印`false`。因为`"/authServlet"`对`"role1"`是允许的，所以用户将能够访问它。

# 参见

+   这个菜谱的完整源代码可在[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter05/ch05-authentication`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter05/ch05-authentication)找到。

# 通过授权授予权限

如果认证是定义谁可以访问特定资源的方式，那么授权就是定义用户一旦获得域访问权限后可以做什么和不能做什么的方式。

这就像允许某人进入你的房子，但拒绝他们访问你的电视遥控器（顺便说一句，这是一个非常重要的访问权限）。或者，允许访问遥控器，但拒绝访问成人频道。

一种实现方式是通过配置文件，这正是我们将在这个菜谱中要做的。

# 准备工作

让我们先添加依赖项：

```java
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何实现...

1.  首先，我们在一个单独的类中定义一些角色，以便我们可以重用它：

```java
public class Roles {
    public static final String ROLE1 = "role1";
    public static final String ROLE2 = "role2";
    public static final String ROLE3 = "role3";
}
```

1.  然后我们定义一些应用程序用户可以执行的操作：

```java
@Stateful
public class UserActivity {

    @RolesAllowed({Roles.ROLE1})
    public void role1Allowed(){
        System.out.println("role1Allowed executed");
    }

    @RolesAllowed({Roles.ROLE2})
    public void role2Allowed(){
        System.out.println("role2Allowed executed");
    }

    @RolesAllowed({Roles.ROLE3})
    public void role3Allowed(){
        System.out.println("role3Allowed executed");
    }

    @PermitAll
    public void anonymousAllowed(){
        System.out.println("anonymousAllowed executed");
    }

    @DenyAll
    public void noOneAllowed(){
        System.out.println("noOneAllowed executed");
    } 

}
```

1.  让我们创建一个用于可执行任务的接口：

```java
public interface Executable {
    void execute() throws Exception;
}
```

1.  然后让我们为执行它们的角色创建另一个：

```java
public interface RoleExecutable {
    void run(Executable executable) throws Exception;
}
```

1.  对于每个角色，我们创建一个执行器。它将像是一个拥有该角色权利的环境：

```java
@Named
@RunAs(Roles.ROLE1)
public class Role1Executor implements RoleExecutable {

    @Override
    public void run(Executable executable) throws Exception {
        executable.execute();
    }
}
```

```java
@Named
@RunAs(Roles.ROLE2)
public class Role2Executor implements RoleExecutable {

    @Override
    public void run(Executable executable) throws Exception {
        executable.execute();
    }
}
```

```java
@Named
@RunAs(Roles.ROLE3)
public class Role3Executor implements RoleExecutable {

    @Override
    public void run(Executable executable) throws Exception {
        executable.execute();
    }
}
```

1.  然后我们实现`HttpAuthenticationMechanism`：

```java
@ApplicationScoped
public class AuthenticationMechanism implements 
HttpAuthenticationMechanism {

    @Override
    public AuthenticationStatus validateRequest(HttpServletRequest 
     request, HttpServletResponse response, HttpMessageContext 
     httpMessageContext) throws AuthenticationException {

        if (httpMessageContext.isAuthenticationRequest()) {

            Credential credential = 
            httpMessageContext.getAuthParameters()
            .getCredential();
            if (!(credential instanceof CallerOnlyCredential)) {
                throw new IllegalStateException("Invalid 
                mechanism");
            }

            CallerOnlyCredential callerOnlyCredential = 
            (CallerOnlyCredential) credential;

            if (null == callerOnlyCredential.getCaller()) {
                throw new AuthenticationException();
            } else switch (callerOnlyCredential.getCaller()) {
                case "user1":
                    return  
                    httpMessageContext.
                    notifyContainerAboutLogin
                    (callerOnlyCredential.getCaller(),
                     new HashSet<>
                    (asList(Roles.ROLE1)));
                case "user2":
                    return 
                    httpMessageContext.
                    notifyContainerAboutLogin
                    (callerOnlyCredential.getCaller(), 
                     new HashSet<>
                    (asList(Roles.ROLE2)));
                case "user3":
                    return 
                    httpMessageContext.
                    notifyContainerAboutLogin
                    (callerOnlyCredential.getCaller(), 
                     new HashSet<>
                    (asList(Roles.ROLE3)));
                default:
                    throw new AuthenticationException();
            }

        }

        return httpMessageContext.doNothing();
    }

}
```

1.  最后，我们创建一个管理所有这些资源的 servlet：

```java
@DeclareRoles({Roles.ROLE1, Roles.ROLE2, Roles.ROLE3})
@WebServlet(name = "/UserAuthorizationServlet", urlPatterns = {"/UserAuthorizationServlet"})
public class UserAuthorizationServlet extends HttpServlet {

    private static final long serialVersionUID = 1L;

    @Inject
    private SecurityContext securityContext;

    @Inject
    private Role1Executor role1Executor;

    @Inject
    private Role2Executor role2Executor;

    @Inject
    private Role3Executor role3Executor;

    @Inject
    private UserActivity userActivity;

    @Override
    public void doGet(HttpServletRequest request, 
    HttpServletResponse 
    response) throws ServletException, IOException {

        try {
            String name = request.getParameter("name");
            if (null != name || !"".equals(name)) {
                AuthenticationStatus status = 
                securityContext.authenticate(
                        request, response, withParams().credential(
                        new CallerOnlyCredential(name)));

                response.getWriter().write("Authentication 
                status: " + status.name() + "\n");
            }

            String principal = null;
            if (request.getUserPrincipal() != null) {
                principal = request.getUserPrincipal().getName();
            }

            response.getWriter().write("User: " + principal +
            "\n");
            response.getWriter().write("Role \"role1\" access: " + 
            request.isUserInRole(Roles.ROLE1) + "\n");
            response.getWriter().write("Role \"role2\" access: " + 
            request.isUserInRole(Roles.ROLE2) + "\n");
            response.getWriter().write("Role \"role3\" access: " + 
            request.isUserInRole(Roles.ROLE3) + "\n");

            RoleExecutable executable = null;

            if (request.isUserInRole(Roles.ROLE1)) {
                executable = role1Executor;
            } else if (request.isUserInRole(Roles.ROLE2)) {
                executable = role2Executor;
            } else if (request.isUserInRole(Roles.ROLE3)) {
                executable = role3Executor;
            }

            if (executable != null) {
                executable.run(() -> {
                    try {
                        userActivity.role1Allowed();
                        response.getWriter().write("role1Allowed 
                        executed: true\n");
                    } catch (Exception e) {
                        response.getWriter().write("role1Allowed  
                        executed: false\n");
                    }

                    try {
                        userActivity.role2Allowed();
                        response.getWriter().write("role2Allowed 
                        executed: true\n");
                    } catch (Exception e) {
                        response.getWriter().write("role2Allowed  
                        executed: false\n");
                    }

                    try {
                        userActivity.role3Allowed();
                        response.getWriter().write("role2Allowed 
                        executed: true\n");
                    } catch (Exception e) {
                        response.getWriter().write("role2Allowed 
                        executed: false\n");
                    }

                });

            }

            try {
                userActivity.anonymousAllowed();
                response.getWriter().write("anonymousAllowed  
                executed: true\n");
            } catch (Exception e) {
                response.getWriter().write("anonymousAllowed 
                executed: false\n");
            }

            try {
                userActivity.noOneAllowed();
                response.getWriter().write("noOneAllowed 
                executed: true\n");
            } catch (Exception e) {
                response.getWriter().write("noOneAllowed 
                executed: false\n");
            } 

        } catch (Exception ex) {
            System.err.println(ex.getMessage());
        }

    }
}
```

要尝试这段代码，你可以运行以下 URL：

+   `http://localhost:8080/ch05-authorization/UserAuthorizationServlet?name=user1`

+   `http://localhost:8080/ch05-authorization/UserAuthorizationServlet?name=user2`

+   `http://localhost:8080/ch05-authorization/UserAuthorizationServlet?name=user3`

例如，`user1`的结果将如下所示：

```java
Authentication status: SUCCESS
User: user1
Role "role1" access: true
Role "role2" access: false
Role "role3" access: false
role1Allowed executed: true
role2Allowed executed: false
role2Allowed executed: false
anonymousAllowed executed: true
noOneAllowed executed: false
```

如果你尝试使用一个不存在的用户，结果将如下所示：

```java
Authentication status: SEND_FAILURE
User: null
Role "role1" access: false
Role "role2" access: false
Role "role3" access: false
anonymousAllowed executed: true
noOneAllowed executed: false
```

# 它是如何工作的...

好吧，这里发生了很多事情！让我们从我们的`UserActivity`类开始。

我们使用`@RolesAllowed`注解来定义可以访问该类每个方法的角色：

```java
    @RolesAllowed({Roles.ROLE1})
    public void role1Allowed(){
        System.out.println("role1Allowed executed");
    }
```

你可以在注解内添加多个角色（它是一个数组）。

我们还有两个其他有趣的注解，`@PermitAll`和`@DenyAll`：

+   `@PermitAll` 注解允许任何人访问该方法，甚至无需任何认证。

+   `@DenyAll` 注解拒绝所有人访问该方法，即使是拥有最高权限的已认证用户。

然后我们有了我们所说的执行器：

```java
@Named
@RunAs(Roles.ROLE1)
public class Role1Executor implements RoleExecutable {

    @Override
    public void run(Executable executable) throws Exception {
        executable.execute();
    }
}
```

我们在类级别使用了 `@RunAs` 注解，这意味着这个类继承了定义的角色（在这种情况下，`"role1"`）的所有权限。这意味着这个类的每一个方法都将拥有 `"role1"` 权限。

现在，看看 `UserAuthorizationServlet`，在开始的地方有一个重要的对象：

```java
    @Inject
    private SecurityContext securityContext;
```

在这里，我们要求服务器给我们一个安全上下文实例，以便我们可以用它进行认证。

然后，如果 `name` 参数被填写，我们就到达这一行：

```java
            AuthenticationStatus status = securityContext.authenticate(
                    request, response, withParams().credential(new 
                    CallerOnlyCredential(name)));
```

这将要求 Java EE 服务器处理认证。这就是我们的 `HttpAuthenticationMechanism` 实现发挥作用的地方。

由于前面的代码创建了 `CallerOnlyCredential`，我们的认证机制将基于它：

```java
            Credential credential = httpMessageContext.getAuthParameters().
            getCredential();
            if (!(credential instanceof CallerOnlyCredential)) {
                throw new IllegalStateException("Invalid mechanism");
            }

            CallerOnlyCredential callerOnlyCredential = 
           (CallerOnlyCredential) credential;
```

一旦我们有了凭证实例，我们可以检查用户是否存在：

```java
            if (null == callerOnlyCredential.getCaller()) {
                throw new AuthenticationException();
            } else switch (callerOnlyCredential.getCaller()) {
                case "user1":
                    return httpMessageContext.notifyContainerAboutLogin
                    (callerOnlyCredential.getCaller(), new HashSet<>
                    (asList(Roles.ROLE1)));
                case "user2":
                    return httpMessageContext.notifyContainerAboutLogin
                    (callerOnlyCredential.getCaller(), new HashSet<>
                    (asList(Roles.ROLE2)));
                case "user3":
                    return httpMessageContext.notifyContainerAboutLogin
                    (callerOnlyCredential.getCaller(), new HashSet<>
                    (asList(Roles.ROLE3)));
                default:
                    throw new AuthenticationException();
            }
```

因此，我们说 `"user1"` 可以访问 `"role1"`，`"user2"` 可以访问 `"role2"`，依此类推。

一旦定义了用户角色，我们就回到了 servlet，并可以选择使用哪个环境（执行器）：

```java
            if (request.isUserInRole(Roles.ROLE1)) {
                executable = role1Executor;
            } else if (request.isUserInRole(Roles.ROLE2)) {
                executable = role2Executor;
            } else if (request.isUserInRole(Roles.ROLE3)) {
                executable = role3Executor;
            }
```

然后我们尝试 `UserActivity` 类的所有方法。只有允许该特定角色的方法将被执行；其他方法将抛出异常，除了 `@PermitAll` 方法，它无论如何都会运行，以及 `@DenyAll`，它无论如何都不会运行。

# 参见

+   在[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter05/ch05-authorization`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter05/ch05-authorization)查看这个菜谱的完整源代码。

# 使用 SSL/TLS 保护数据机密性和完整性

安全性也意味着保护你的数据传输，为此我们有了最流行的方法，被称为**安全套接字层**（**SSL**）。

**传输层安全性**，或**TLS**，是 SSL 的最新版本。因此，GlassFish 5.0 支持 SSL 3.0 和 TLS 1.0 作为协议。

这个菜谱将向你展示如何使 GlassFish 5.0 能够正确地与 SSL 一起工作。所有 Java EE 服务器都有自己实现这一点的独特方式。

# 准备工作

要在 GlassFish 中启用 SSL，你需要配置 SSL HTTP 监听器。你只需要做以下这些：

1.  确保 GlassFish 正在运行。

1.  使用 `create-ssl` 命令创建你的 SSL HTTP 监听器。

1.  重新启动 GlassFish 服务器。

# 如何做到这一点...

1.  要完成这个任务，你需要访问 GlassFish 的远程**命令行界面**（**CLI**）。你可以通过访问以下路径来完成：

`$GLASSFISH_HOME/bin`

1.  一旦你到了那里，执行以下命令：

```java
./asadmin
```

1.  当提示准备好后，你可以执行这个命令：

```java
create-ssl --type http-listener --certname cookbookCert http-listener-1
```

1.  然后您可以重新启动服务器，您的`http-listener-1`将使用 SSL 工作。如果您想从监听器中删除 SSL，只需回到提示并执行此命令：

```java
delete-ssl --type http-listener http-listener-1
```

# 它是如何工作的...

使用 SSL，客户端和服务器在发送数据之前加密数据，在接收数据时解密数据。当浏览器打开一个加密网站（使用 HTTPS）时，会发生一个称为**握手**的过程。

在握手过程中，浏览器向服务器请求一个会话；服务器通过发送证书和公钥来回答。浏览器验证证书，如果有效，则生成一个唯一的会话密钥，使用服务器公钥加密它，并将其发送回服务器。一旦服务器收到会话密钥，它就会使用其私钥解密它。

现在，客户端和服务器，只有它们，拥有会话密钥的副本，并可以确保通信是安全的。

# 还有更多...

强烈建议您使用来自**认证机构**（**CA**）的证书，而不是像我们在本配方中那样使用自签名证书。

您可以查看[`letsencrypt.org`](https://letsencrypt.org)，在那里您可以获取免费的证书。

使用它的过程是相同的；你只需更改`--certname`参数的值。

# 参见

+   想要了解关于 GlassFish 5 所有安全方面和配置的详细信息，请查看[`javaee.github.io/glassfish/doc/5.0/security-guide.pdf`](https://javaee.github.io/glassfish/doc/5.0/security-guide.pdf)。

# 使用声明式安全

在构建应用程序的安全功能时，你可以基本上使用两种方法：**编程安全**和**声明式安全**：

+   编程方法是指您使用代码定义应用程序的安全策略。

+   声明式方法是指通过声明策略然后相应地应用它们来执行。

这个配方将向你展示声明式方法。

# 准备工作

让我们从添加依赖项开始：

```java
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何做到这一点...

1.  让我们为我们的应用程序创建一个角色列表：

```java
public class Roles {
    public static final String ADMIN = "admin";
    public static final String USER = "user";
}
```

1.  然后我们创建一个只能由一个角色执行的任务列表，一个每个人都可以执行的任务，以及一个没有人可以执行的任务：

```java
@Stateful
public class UserBean {

    @RolesAllowed({Roles.ADMIN})
    public void adminOperation(){
        System.out.println("adminOperation executed");
    }

    @RolesAllowed({Roles.USER})
    public void userOperation(){
        System.out.println("userOperation executed");
    }

    @PermitAll
    public void everyoneCanDo(){
        System.out.println("everyoneCanDo executed");
    }

    @DenyAll
    public void noneCanDo(){
        System.out.println("noneCanDo executed");
    } 

}
```

1.  现在，我们为`USER`和`ADMIN`角色创建一个环境，以便它们执行各自的操作：

```java
@Named
@RunAs(Roles.USER)
public class UserExecutor implements RoleExecutable {

    @Override
    public void run(Executable executable) throws Exception {
        executable.execute();
    }
}
```

```java
@Named
@RunAs(Roles.ADMIN)
public class AdminExecutor implements RoleExecutable {

    @Override
    public void run(Executable executable) throws Exception {
        executable.execute();
    }
}
```

1.  然后我们实现`HttpAuthenticationMechanism`:

```java
@ApplicationScoped
public class AuthenticationMechanism implements HttpAuthenticationMechanism {

    @Override
    public AuthenticationStatus validateRequest(HttpServletRequest 
    request, HttpServletResponse response, HttpMessageContext 
    httpMessageContext) 
    throws AuthenticationException {

        if (httpMessageContext.isAuthenticationRequest()) {

            Credential credential = 
            httpMessageContext.getAuthParameters().
            getCredential();
            if (!(credential instanceof CallerOnlyCredential)) {
                throw new IllegalStateException("Invalid 
                mechanism");
            }

            CallerOnlyCredential callerOnlyCredential = 
            (CallerOnlyCredential) 
            credential;

            if (null == callerOnlyCredential.getCaller()) {
                throw new AuthenticationException();
            } else switch (callerOnlyCredential.getCaller()) {
                case Roles.ADMIN:
                    return httpMessageContext
                    .notifyContainerAboutLogin
                    (callerOnlyCredential.getCaller(),
                     new HashSet<>
                    (asList(Roles.ADMIN)));
                case Roles.USER:
                    return httpMessageContext
                   .notifyContainerAboutLogin
                    (callerOnlyCredential.getCaller(),
                    new HashSet<> 
                    (asList(Roles.USER)));
                default:
                    throw new AuthenticationException();
            }

        }

        return httpMessageContext.doNothing();
    }

}
```

1.  最后，我们为每个角色（`USER`和`ADMIN`）创建一个 servlet：

```java
@DeclareRoles({Roles.ADMIN, Roles.USER})
@WebServlet(name = "/UserServlet", urlPatterns = {"/UserServlet"})
public class UserServlet extends HttpServlet {

    private static final long serialVersionUID = 1L;

    @Inject
    private SecurityContext securityContext;

    @Inject
    private UserExecutor userExecutor;

    @Inject
    private UserBean userActivity;

    @Override
    public void doGet(HttpServletRequest request, 
    HttpServletResponse response) throws ServletException, 
    IOException {

        try {
            securityContext.authenticate(
                    request, response, withParams().credential(new 
                    CallerOnlyCredential(Roles.USER)));

            response.getWriter().write("Role \"admin\" access: " + 
            request.isUserInRole(Roles.ADMIN) + "\n");
            response.getWriter().write("Role \"user\" access: " + 
            request.isUserInRole(Roles.USER) + "\n");

            userExecutor.run(() -> {
                try {
                    userActivity.adminOperation();
                    response.getWriter().write("adminOperation 
                    executed: true\n");
                } catch (Exception e) {
                    response.getWriter().write("adminOperation 
                    executed: false\n");
                }

                try {
                    userActivity.userOperation();
                    response.getWriter().write("userOperation 
                    executed: true\n");
                } catch (Exception e) {
                    response.getWriter().write("userOperation  
                    executed: false\n");
                }

            });

            try {
                userActivity.everyoneCanDo();
                response.getWriter().write("everyoneCanDo 
                executed: true\n");
            } catch (Exception e) {
                response.getWriter().write("everyoneCanDo
                executed: false\n");
            }

            try {
                userActivity.noneCanDo();
                response.getWriter().write("noneCanDo 
                executed: true\n");
            } catch (Exception e) {
                response.getWriter().write("noneCanDo 
                executed: false\n");
            }

        } catch (Exception ex) {
            System.err.println(ex.getMessage());
        }

    }
}
```

```java
@DeclareRoles({Roles.ADMIN, Roles.USER})
@WebServlet(name = "/AdminServlet", urlPatterns = {"/AdminServlet"})
public class AdminServlet extends HttpServlet {

    private static final long serialVersionUID = 1L;

    @Inject
    private SecurityContext securityContext;

    @Inject
    private AdminExecutor adminExecutor;

    @Inject
    private UserBean userActivity;

    @Override
    public void doGet(HttpServletRequest request, 
    HttpServletResponse 
    response) throws ServletException, IOException {

        try {
            securityContext.authenticate(
                    request, response, withParams().credential(new 
                    CallerOnlyCredential(Roles.ADMIN)));

            response.getWriter().write("Role \"admin\" access: " + 
            request.isUserInRole(Roles.ADMIN) + "\n");
            response.getWriter().write("Role \"user\" access: " + 
            request.isUserInRole(Roles.USER) + "\n");

            adminExecutor.run(() -> {
                try {
                    userActivity.adminOperation();
                    response.getWriter().write("adminOperation 
                    executed: true\n");
                } catch (Exception e) {
                    response.getWriter().write("adminOperation 
                    executed: false\n");
                }

                try {
                    userActivity.userOperation();
                    response.getWriter().write("userOperation 
                    executed: true\n");
                } catch (Exception e) {
                    response.getWriter().write("userOperation 
                    executed: false\n");
                }

            });

            try {
                userActivity.everyoneCanDo();
                response.getWriter().write("everyoneCanDo 
                executed: true\n");
            } catch (Exception e) {
                response.getWriter().write("everyoneCanDo
                executed: false\n");
            }

            try {
                userActivity.noneCanDo();
                response.getWriter().write("noneCanDo 
                executed: true\n");
            } catch (Exception e) {
                response.getWriter().write("noneCanDo 
                executed: false\n");
            }

        } catch (Exception ex) {
            System.err.println(ex.getMessage());
        }

    }
}
```

# 它是如何工作的...

查看`UserServlet`（适用于`USER`角色），我们首先看到认证步骤：

```java
            securityContext.authenticate(
                    request, response, withParams().credential(new 
                    CallerOnlyCredential(Roles.ADMIN)));
```

例如，我们使用角色名称作为用户名，因为如果我们查看`AuthenticationMechanism`类（实现`HttpAuthenticationMechanism`），我们会看到它在执行所有认证和为用户分配正确角色的艰苦工作：

```java
            Credential credential = 
            httpMessageContext.getAuthParameters()
            .getCredential();
            if (!(credential instanceof CallerOnlyCredential)) {
                throw new IllegalStateException("Invalid mechanism");
            }

            CallerOnlyCredential callerOnlyCredential = 
            (CallerOnlyCredential) 
            credential;

            if (null == callerOnlyCredential.getCaller()) {
                throw new AuthenticationException();
            } else switch (callerOnlyCredential.getCaller()) {
                case Roles.ADMIN:
                    return httpMessageContext.notifyContainerAboutLogin
                    (callerOnlyCredential.getCaller(), new HashSet<>
                    (asList(Roles.ADMIN)));
                case Roles.USER:
                    return httpMessageContext.notifyContainerAboutLogin
                    (callerOnlyCredential.getCaller(), new HashSet<>
                    (asList(Roles.USER)));
                default:
                    throw new AuthenticationException();
            }
```

然后回到我们的`UserServlet`，现在用户已经分配了适当的角色，他们能做什么以及不能做什么就只是个问题了：

```java
            userExecutor.run(() -> {
                try {
                    userActivity.adminOperation();
                    response.getWriter().write("adminOperation  
                    executed: true\n");
                } catch (Exception e) {
                    response.getWriter().write("adminOperation 
                    executed: false\n");
                }

                try {
                    userActivity.userOperation();
                    response.getWriter().write("userOperation 
                    executed: true\n");
                } catch (Exception e) {
                    response.getWriter().write("userOperation 
                    executed:  false\n");
                }

            });
```

此外，我们还尝试了每个人都可以执行以及没有人可以执行的任务：

```java
            try {
                userActivity.everyoneCanDo();
                response.getWriter().write("everyoneCanDo 
                executed: true\n");
            } catch (Exception e) {
                response.getWriter().write("everyoneCanDo 
                executed: false\n");
            }

            try {
                userActivity.noneCanDo();
                response.getWriter().write("noneCanDo 
                executed: true\n");
            } catch (Exception e) {
                response.getWriter().write("noneCanDo 
                executed: false\n");
            }
```

`AdminServlet`类使用`AdminExecutor`环境执行完全相同的步骤，因此为了节省空间，我们将省略它。

要尝试这段代码，只需在 Java EE 8 兼容的服务器上使用以下 URL 运行它：

+   `http://localhost:8080/ch05-declarative/AdminServlet`

+   `http://localhost:8080/ch05-declarative/UserServlet`

`AdminServlet`的结果示例将如下所示：

```java
Role "admin" access: true
Role "user" access: false
adminOperation executed: true
userOperation executed: false
everyoneCanDo executed: true
noneCanDo executed: false
```

# 参见

+   在[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter05/ch05-declarative`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter05/ch05-declarative)查看此菜谱的完整源代码。

# 使用程序化安全

我们已经看到了声明式方法，现在让我们看看程序化方法。

# 准备中

让我们先添加依赖项：

```java
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何做到这一点...

1.  让我们先定义我们的角色列表：

```java
public class Roles {
    public static final String ADMIN = "admin";
    public static final String USER = "user";
}
```

1.  然后，让我们定义一个基于角色的任务列表：

```java
@Stateful
public class UserBean {

    @RolesAllowed({Roles.ADMIN})
    public void adminOperation(){
        System.out.println("adminOperation executed");
    }

    @RolesAllowed({Roles.USER})
    public void userOperation(){
        System.out.println("userOperation executed");
    }

    @PermitAll
    public void everyoneCanDo(){
        System.out.println("everyoneCanDo executed");
    }

}
```

1.  现在，让我们实现`IndentityStore`接口。在这里，我们定义了验证用户身份的策略：

```java
@ApplicationScoped
public class UserIdentityStore implements IdentityStore {

    @Override
    public CredentialValidationResult validate(Credential credential) {
        if (credential instanceof UsernamePasswordCredential) {
            return validate((UsernamePasswordCredential) credential);
        }

        return CredentialValidationResult.NOT_VALIDATED_RESULT;
    }

    public CredentialValidationResult validate(UsernamePasswordCredential 
    usernamePasswordCredential) {

        if (usernamePasswordCredential.
        getCaller().equals(Roles.ADMIN)
                && usernamePasswordCredential.
                getPassword().compareTo("1234")) 
        {

            return new CredentialValidationResult(
                    new CallerPrincipal
                    (usernamePasswordCredential.getCaller()),
                    new HashSet<>(Arrays.asList(Roles.ADMIN)));
        } else if (usernamePasswordCredential.
          getCaller().equals(Roles.USER)
                && usernamePasswordCredential.
                getPassword().compareTo("1234")) 
        {

            return new CredentialValidationResult(
                    new CallerPrincipal
                    (usernamePasswordCredential.getCaller()),
                    new HashSet<>(Arrays.asList(Roles.USER)));
        }

        return CredentialValidationResult.INVALID_RESULT;
    }

}
```

1.  这里，我们实现了`HttpAuthenticationMethod`接口：

```java
@ApplicationScoped
public class AuthenticationMechanism implements HttpAuthenticationMechanism {

    @Inject
    private UserIdentityStore identityStore;

    @Override
    public AuthenticationStatus validateRequest(HttpServletRequest 
    request, 
    HttpServletResponse response, HttpMessageContext 
    httpMessageContext) 
    throws AuthenticationException {

        if (httpMessageContext.isAuthenticationRequest()) {

            Credential credential = 
            httpMessageContext.getAuthParameters()
            .getCredential();
            if (!(credential instanceof UsernamePasswordCredential)) {
                throw new IllegalStateException("Invalid 
                mechanism");
            }

            return httpMessageContext.notifyContainerAboutLogin
            (identityStore.validate(credential));
        }

        return httpMessageContext.doNothing();
    }

}
```

1.  最后，我们创建一个 servlet，用户将在这里进行身份验证并执行他们的操作：

```java
@DeclareRoles({Roles.ADMIN, Roles.USER})
@WebServlet(name = "/OperationServlet", urlPatterns = {"/OperationServlet"})
public class OperationServlet extends HttpServlet {

    private static final long serialVersionUID = 1L;

    @Inject
    private SecurityContext securityContext;

    @Inject
    private UserBean userActivity;

    @Override
    public void doGet(HttpServletRequest request, 
    HttpServletResponse 
    response) throws ServletException, IOException {

        String name = request.getParameter("name");
        String password = request.getParameter("password");

        Credential credential = new UsernamePasswordCredential(name, 
        new Password(password));

        AuthenticationStatus status = securityContext.authenticate(
                request, response, 
        withParams().credential(credential));

        response.getWriter().write("Role \"admin\" access: " + 
        request.isUserInRole(Roles.ADMIN) + "\n");
        response.getWriter().write("Role \"user\" access: " + 
        request.isUserInRole(Roles.USER) + "\n");

        if (status.equals(AuthenticationStatus.SUCCESS)) {

            if (request.isUserInRole(Roles.ADMIN)) {
                userActivity.adminOperation();
                response.getWriter().write("adminOperation 
                executed: true\n");
            } else if (request.isUserInRole(Roles.USER)) {
                userActivity.userOperation();
                response.getWriter().write("userOperation 
                executed: true\n");
            }

            userActivity.everyoneCanDo();
            response.getWriter().write("everyoneCanDo 
            executed: true\n");

        } else {
            response.getWriter().write("Authentication failed\n");
        }

    }
}
```

要尝试这段代码，请在 Java EE 8 兼容的服务器上使用以下 URL 运行它：

+   `http://localhost:8080/ch05-programmatic/OperationServlet?name=user&password=1234`

+   `http://localhost:8080/ch05-programmatic/OperationServlet?name=admin&password=1234`

`ADMIN`角色的一个示例结果如下：

```java
Role "admin" access: true
Role "user" access: false
adminOperation executed: true
everyoneCanDo executed: true
```

如果您使用错误的用户名/密码对，您将得到以下结果：

```java
Role "admin" access: false
Role "user" access: false
Authentication failed
```

# 它是如何工作的...

与声明式方法（参见本章前面的菜谱）相反，这里我们使用代码来验证用户。我们通过实现`IdentityStore`接口来完成它。

例如，即使我们已经硬编码了密码，您也可以使用相同的代码片段来验证密码与数据库、LDAP、外部端点等：

```java
        if (usernamePasswordCredential.getCaller().equals(Roles.ADMIN)
                && 
        usernamePasswordCredential.getPassword().compareTo("1234")) 
        {

            return new CredentialValidationResult(
                    new CallerPrincipal(usernamePasswordCredential
                    .getCaller()),
                    new HashSet<>(asList(Roles.ADMIN)));
        } else if (usernamePasswordCredential.getCaller()
          .equals(Roles.USER)
                && usernamePasswordCredential.
                getPassword().compareTo("1234")) 
        {

            return new CredentialValidationResult(
                    new CallerPrincipal(usernamePasswordCredential
                   .getCaller()),
                    new HashSet<>(asList(Roles.USER)));
        }

        return INVALID_RESULT;
```

使用`IdentityStore`进行身份验证意味着只是通过`HttpAuthenticationMethod`进行委托：

```java
            Credential credential = 
            httpMessageContext.getAuthParameters().getCredential();
            if (!(credential instanceof UsernamePasswordCredential)) {
                throw new IllegalStateException("Invalid mechanism");
            }

            return httpMessageContext.notifyContainerAboutLogin
            (identityStore.validate(credential));
```

然后，`OperationServlet`将尝试进行身份验证：

```java
        String name = request.getParameter("name");
        String password = request.getParameter("password");

        Credential credential = new UsernamePasswordCredential(name, 
        new Password(password));

        AuthenticationStatus status = securityContext.authenticate(
                request, response, 
               withParams().credential(credential));
```

基于此，我们将定义接下来会发生什么流程：

```java
        if (status.equals(AuthenticationStatus.SUCCESS)) {

            if (request.isUserInRole(Roles.ADMIN)) {
                userActivity.adminOperation();
                response.getWriter().write("adminOperation 
                executed: true\n");
            } else if (request.isUserInRole(Roles.USER)) {
                userActivity.userOperation();
                response.getWriter().write("userOperation 
                executed: true\n");
            }

            userActivity.everyoneCanDo();
            response.getWriter().write("everyoneCanDo executed:
```

```java

        true\n");

        } else {
            response.getWriter().write("Authentication failed\n");
        }
```

注意！这是您定义每个角色将做什么的代码。

# 参见

+   在[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter05/ch05-programmatic`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter05/ch05-programmatic)查看此菜谱的完整源代码。
