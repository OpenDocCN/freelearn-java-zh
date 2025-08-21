# 第十一章：细粒度访问控制

在本章中，我们首先将探讨两种实现细粒度授权的方法-可能影响应用程序页面部分的内容。接下来，我们将查看 Spring Security 通过方法注解和使用基于接口的代理来实现业务层安全的方法。然后，我们将回顾注解 based 安全的一个有趣功能，它允许基于角色的数据过滤。最后，我们将查看类 based 代理与接口 based 代理的区别。

在本章的进程中，我们将介绍以下主题：

+   配置并在不同的方法上进行实验，以实现在页面上根据用户请求的安全上下文进行内容授权检查

+   执行配置和代码注解，使调用者预授权成为我们应用程序业务层安全的关键部分

+   实现方法级安全性的几种不同方法，并回顾每种类型的优缺点

+   在集合和数组上使用方法级注解实现数据过滤

+   在 Spring MVC 控制器上实现方法级安全，避免配置`antMatcher()`方法和`<intercept-url>`元素

# Gradle 依赖项

有许多可选依赖项可能需要，这取决于您决定使用哪些功能。许多这些依赖项已在 Spring Boot 的启动父级中注释掉。您会发现我们的`build.gradle`文件已经包括了以下所有依赖项：

```java
    //build.gradle
    // Required for JSR-250 based security:
    // JSR-250 Annotations

 compile ('javax.annotation:javax.annotation-api:1.3')    // Already provided by Spring Boot
 *// compile('cglib:cglib-nodep')*    // Already provided by Spring Boot
    // Required for protect-pointcut
 *// compile('org.aspectj:aspectjweaver')*
```

# 集成 Spring 表达式语言（SpEL）

Spring Security 利用**Spring 表达式语言**（**SpEL**）集成，以便轻松阐述各种授权要求。如果您还记得，我们在第二章 *开始使用 Spring Security*中已经查看了 SpEL 的使用，当时我们定义了我们的`antMatcher()`方法：

```java
    .antMatchers("/events/").hasRole("ADMIN")
```

Spring Security 提供了一个`o.s.s.access.expression.SecurityExpressionRoot`对象，提供了可用于做出访问控制决策的方法和对象。例如，可以使用的一个方法是接受一个字符串的`hasRole`方法。这与前面代码片段中访问属性的值相对应。实际上，还有许多其他表达式可供使用，如下表所示：

| **表达式** | **描述** |
| --- | --- |
| `hasRole(String role)``hasAuthority(String role)` | 如果当前用户具有指定的权限，则返回`true`。 |
| `hasAnyRole(String... role)``hasAnyAuthority(String... authority)` | 如果当前用户具有任何指定的权限，则返回`true`。 |
| `principal` | 允许访问当前`Authentication`对象的 principal 属性的权限。正如在第三章 *自定义认证*中讨论的，这通常是一个`UserDetails`实例。 |
| `authentication` | 从 `SecurityContext` 接口返回的 `SecurityContextHolder` 类的 `getContext()` 方法获取当前 `Authentication` 对象。 |
| `permitAll` | 总是返回 `true`。 |
| `denyAll` | 总是返回 `false`。 |
| `isAnonymous()` | 如果当前主体是匿名（没有认证），则返回 true。 |
| `isRememberMe()` | 如果当前主体是使用记住我功能进行认证的，则返回 `true`。 |
| `isAuthenticated()` | 如果用户不是匿名用户（也就是说，他们已经认证），则返回 `true`。 |
| `isFullyAuthenticated()` | 如果用户通过记住我以外的其他方式进行认证，则返回 `true`。 |
| `hasPermission(Object target, Object permission)` | 如果用户有权限访问给定权限的指定对象，则返回 `true`。 |
| `hasPermission( String targetId, String targetType, Object permission)` | 如果用户有权限访问给定类型和权限的指定标识符，则返回 `true`。 |

我们提供了一些使用这些 SpEL 表达式的示例代码。请记住，我们将在本章和下一章中详细介绍：

```java
    // allow users with ROLE_ADMIN

    hasRole('ADMIN')

    // allow users that do not have the ROLE_ADMIN

     !hasRole('ADMIN')

    // allow users that have ROLE_ADMIN or ROLE_ROOT and
    // did not use the remember me feature to login

    fullyAuthenticated() and hasAnyRole('ADMIN','ROOT')

    // allow if Authentication.getName() equals admin

    authentication.name == 'admin'
```

# `WebSecurityExpressionRoot` 类

`o.s.s.web.access.expression.WebSecurityExpressionRoot` 类为我们提供了一些额外的属性。这些属性，连同前面提到的标准属性，可以在 `antMatchers()` 方法的访问属性中以及 JSP/Thymeleaf 的 `<sec:authorize>` 标签的 `access` 属性中使用，我们稍后会讨论这些：

| **表达式** | **描述** |
| --- | --- |
| `request` | 当前的 `HttpServletRequest` 方法。 |
| `hasIpAddress(String... ipAddress)` | 如果当前 IP 地址匹配 `ipAddress` 值，则返回 `true`。这可以是一个确切的 IP 地址或者 IP 地址/子网掩码。 |

# 使用请求属性

`request` 属性相对容易理解，但我们提供了一些示例代码。请记住，这些示例都可以放在 `antMatchers()` 方法的访问属性中或者 `<sec:authorize>` 元素的访问属性中：

```java
    // allows only HTTP GETrequest.method == 'GET'
    // allow anyone to perform a GET, but
    // other methods require ROLE_ADMIN

    request.method == 'GET' ? permitAll : hasRole('ADMIN')
```

# 使用 hasIpAddress 方法

`hasIpAddress` 方法并没有 `request` 属性那么简单明了。`hasIpAddress` 会很容易匹配一个确切的 IP 地址；例如，以下代码如果当前用户 IP 地址是 `192.168.1.93`，则允许访问：

```java
    hasIpAddress('192.168.1.93')
```

然而，这并不是非常有用。相反，我们可以定义以下代码，这也将匹配我们的 IP 地址以及我们子网中的任何其他 IP 地址：

```java
    hasIpAddress('192.168.1.0/24')
```

问题是：这是如何计算的？关键是要理解如何计算网络地址及其掩码。要了解如何进行计算，我们可以看一个具体的例子。我们从 Linux 终端启动 `ifconfig` 来查看我们的网络信息（Windows 用户可以在命令提示符中输入 `ipconfig /all`）：

```java
$ ifconfig wlan0     Link encap:Ethernet HWaddr a0:88:b4:8b:26:64 inet addr:192.168.1.93 Bcast:192.168.1.255 Mask:255.255.255.0
```

查看以下图表：

![](img/e2c7a7d2-5944-4bce-a054-d54ad9df8a0b.png)

我们可以看到我们的掩码的前三个八位字节是**255**。这意味着我们的**IP 地址**的前三个八位字节属于网络地址。在我们的计算中，这意味着剩下的八位字节是**0**：

![](img/13bb5b29-556c-4a5c-8662-8f5857a14289.png)

然后我们可以通过首先将每个八位字节转换为二进制数，然后计算其中有多少个一来进行掩码计算。在我们的实例中，得到`24`。

这意味着我们的 IP 地址将匹配`192.168.1.0/24`。有关更多信息的良好资源是思科的文档，可在[`www.cisco.com/c/en/us/support/docs/ip/routing-information-protocol-rip/13788-3.html`](http://www.cisco.com/c/en/us/support/docs/ip/routing-information-protocol-rip/13788-3.html)找到。

# 方法访问控制表达式根类

方法访问控制表达式（SpEL）还提供了一些额外的属性，可以通过`o.s.s.access.expression.method.MethodSecurityExpressionRoot`类使用：

| **表达式** | **描述** |
| --- | --- |
| `target` | 指的是`this`或被保护的当前对象。 |
| `returnObject` | 指的是注解方法的返回对象。 |
| `filterObject` | 可以在与`@PreFilter`或`@PostFilter`结合使用时，用于集合或数组上，只包含与表达式匹配的元素。`filterObject`对象代表集合或数组的循环变量。 |
| `#<methodArg>` | 可以通过在参数名称前加上`#`来引用任何方法参数。例如，一个名为`id`的方法参数可以使用`#id`来引用。 |

如果这些表达式的描述看起来有点简略，不用担心；我们将在本章后面通过一些例子来详细说明。

我们希望您已经对 Spring Security 的 SpEL 支持的力量有了大致的了解。要了解更多关于 SpEL 的信息，请参考 Spring 的参考文档：[`docs.spring.io/spring/docs/current/spring-framework-reference/html/expressions.html`](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/expressions.html)。

# 页面级授权

页面级授权指的是基于特定用户请求上下文的应用程序功能的可用性。与我们在第二章 *开始使用 Spring Security*中探讨的粗粒度授权不同，细粒度授权通常指的是页面的部分内容的的选择性可用性，而不是完全限制访问页面。大多数现实世界的应用程序将在细粒度授权计划的细节上花费相当多的时间。

Spring Security 为我们提供了以下三种选择性显示功能的方法：

+   Spring Security JSP 标签库允许将条件访问声明放置在页面声明本身中，使用标准的 JSP 标签库语法。

+   Thymeleaf Spring Security 标签库允许在页面声明本身内放置基于条件的访问声明，使用标准的 Thymeleaf 标签库语法。

+   在 MVC 应用程序的控制器层检查用户授权允许控制器做出访问决策，并将决策结果绑定到提供给视图的模型数据。这种方法依赖于标准的 JSTL 条件页面渲染和数据绑定，比 Spring Security 标签库稍微复杂一些；然而，它更符合标准 Web 应用程序 MVC 逻辑设计。

这些方法在为 Web 应用程序开发细粒度授权模型时都是完全有效的。让我们探讨每个方法是如何通过 JBCP 日历用例来实现的。

# 使用 Thymeleaf Spring Security 标签库的有条件渲染

在 Thymeleaf Spring Security 标签库中最常用的功能是基于授权规则有条件地渲染页面的部分。这是通过`< sec:authorize*>`标签实现的，该标签与核心 JSTL 库中的`<if>`标签类似，即标签体的渲染取决于标签属性中提供的条件。我们已经看到了一个非常简短的演示，介绍了如何使用 Spring Security 标签库来限制用户未登录时查看内容。

# 基于 URL 访问规则的有条件渲染

Spring Security 标签库提供了根据已经在安全配置文件中定义的 URL 授权规则来渲染内容的功能。这是通过使用`authorizeRequests()`方法和`antMatchers()`方法来实现的。

如果有多个 HTTP 元素，`authorizeRequests()`方法将使用当前匹配的 HTTP 元素的规则。

例如，我们可以确保所有事件链接只在适当的时候显示，即对于管理员用户——回想一下我们之前定义的访问规则如下：

```java
    .antMatchers("/events/").hasRole("ADMIN")
```

更新`header.html`文件以利用此信息，并根据条件渲染到所有事件页面的链接：

```java
//src/main/resources/templates/fragments/header.html

<html xmlns:th="http://www.thymeleaf.org" xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity4">
...
<li sec:authorize-url="/events/">
<a id="navEventsLink" th:href="@{/events/}">All Events</a></li>
```

这将确保除非用户有足够的权限访问所声明的 URL，否则不显示标签的内容。进一步细化授权检查是可能的，通过在 URL 之前包含方法属性，如下所示：

```java
    <li sec:authorize-url="GET /events/">
    <a id="navEventsLink" th:href="@{/events/}">All Events</a></li>
```

使用`authorize-url`属性在代码块上定义授权检查是方便的，因为它将实际授权检查的知识从您的页面抽象出来，并将其保存在您的安全配置文件中。

请注意，HTTP 方法应与您安全`antMatchers()`方法中指定的大小写相匹配，否则它们可能不会像您期望的那样匹配。另外，请注意 URL 应始终相对于 Web 应用程序上下文根（如您的 URL 访问规则）。

对于许多目的来说，使用`<sec>`标签的`authorize-url`属性足以正确地仅当用户被允许查看时显示与链接或操作相关的内容。记住，这个标签不仅仅需要包围一个链接；如果用户没有权限提交它，它甚至可以包围整个表单。

# 使用 SpEL 的条件渲染

当与`<sec>`标签一起使用时，可以控制 JSP 内容的显示，这是一种更灵活的方法。让我们回顾一下我们在第二章*Spring Security 入门*中学到的内容。我们可以通过更改我们的`header.html`文件，如下所示，将我的事件链接隐藏在任何未认证的用户中：

```java
    //src/main/resources/templates/fragments/header.html

    <li sec:authorize="isAuthenticated()"> 
    <a id="navMyEventsLink" th:href="@{/events/my}">My Events</a></li>
```

SpEL 评估是由与`antMatchers()`方法访问声明规则中使用的表达式相同的代码在后台执行的（假设已经配置了表达式）。因此，从使用`<sec>`标签构建的表达式中可以访问相同的内置函数和属性集。

这两种使用`<sec>`标签的方法为基于安全授权规则的页面内容显示提供了强大、细粒度的控制。

继续启动 JBCP 日历应用程序。访问`https://localhost:8443`，使用用户`user1@example.com`和密码`user1`登录。你会观察到我的事件链接被显示，但所有事件链接被隐藏。登出并使用用户`admin1@example.com`和密码登录

`admin1`。现在两个链接都可见。

你应该从`chapter11.01-calendar`的代码开始。

# 使用控制器逻辑条件性地渲染内容

在本节中，我们将演示如何使用基于 Java 的代码来确定是否应渲染某些内容。我们可以选择只在用户名包含`user`的用户的欢迎页面上显示创建事件链接，这样未登录为管理员的用户在欢迎页面上就看不到创建事件链接。

本章示例代码中的欢迎控制器已更新，使用以下方式从方法名派生一个名为`showCreateLink`的属性来填充模型：

```java
//src/main/java/com/packtpub/springsecurity/web/controllers/WelcomeController.java

    @ModelAttribute ("showCreateLink")
    public boolean showCreateLink(Authentication authentication) {
      return authentication != null && 
      authentication.getName().contains("user");
    }
```

你可能会注意到 Spring MVC 可以自动获取`Authentication`对象。这是因为 Spring Security 将我们当前的`Authentication`对象映射到`HttpServletRequest.getPrincipal()`方法。由于 Spring MVC 将自动解析任何`java.security.Principal`类型的对象为`HttpServletRequest.getPrincipal()`的值，将`Authentication`作为控制器的一个参数是一个轻松访问当前`Authentication`对象的方法。我们也可以通过指定`Principal`类型的参数来解耦代码与 Spring Security。然而，在这个场景中我们选择了`Authentication`，以帮助说明一切是如何相互关联的。

如果我们正在另一个不知道如何做到这一点的其他框架中工作，我们可以使用`SecurityContextHolder`类获取`Authentication`对象，就像我们在第[3]章*自定义认证*中做的那样。同时请注意，如果我们不是使用 Spring MVC，我们完全可以直接设置`HttpServletRequest`属性，而不是在模型中填充它。我们在请求中填充的属性随后将可用于我们的 JSP，就像在使用带有 Spring MVC 的`ModelAndView`对象时一样。

接下来，我们需要在我们的`index.html`文件中使用`HttpServletRequest`属性来确定是否应显示创建活动的链接。更新`index.html`，如下所示：

```java
    //src/main/resources/templates/header.html

    <li th:if="${showCreateLink}"><a id="navCreateEventLink"   
    th:href="@{events/form}">...</li>
```

现在，启动应用程序，使用`admin1@example.com`作为用户名，`admin1`作为密码登录，然后访问所有活动页面。你应该再也看不到主导航中的创建活动链接了（尽管它仍然在页面上）。

你的代码应该看起来像这样：`chapter11.02-calendar`。

# `WebInvocationPrivilegeEvaluator`类

有时应用程序可能不会使用 JSP 编写，需要能够根据 URL 确定访问权限，就像我们用`<... sec:authorize-url="/events/">`做的那样。这可以通过使用`o.s.s.web.access.WebInvocationPrivilegeEvaluator`接口来实现，这个接口也是 JSP 标签库背后的同一个接口。在下面的代码片段中，我们通过用名为`showAdminLink`的属性填充我们的模型来演示它的使用。我们可以使用`@Autowired`注解来获取`WebInvocationPrivilegeEvaluator`：

```java
//src/main/java/com/packtpub/springsecurity/web/controllers/WelcomeController.java

    @ModelAttribute ("showAdminLink")
    public boolean showAdminLink(Authentication authentication) {
       return webInvocationPrivilegeEvaluator.
       isAllowed("/admin/", authentication);
    }
```

如果你正在使用的框架不是由 Spring 管理的，`@Autowire`将无法为你提供`WebInvocationPrivilegeEvaluator`。相反，你可以使用 Spring 的`org.springframework.web.context.WebApplicationContextUtils`接口来获取`WebInvocationPrivilegeEvaluator`的一个实例，如下所示：

```java
    ApplicationContext context = WebApplicationContextUtils
     .getRequiredWebApplicationContext(servletContext);
    WebInvocationPrivilegeEvaluator privEvaluator =
    context.getBean(WebInvocationPrivilegeEvaluator.class)
```

为了尝试一下，更新`index.html`以使用`showAdminLink`请求属性，如下所示：

```java
//src/main/resources/templates/header.html

    <li th:if="${showAdminLink}">
     <a id="h2Link" th:href="@{admin/h2/}" target="_blank">
     H2 Database Console</a>
    ...
    </li>
```

重新启动应用程序并在登录之前查看欢迎页面。H2 链接应该是不可见的。以`admin1@example.com`/`admin1`的身份登录，你应该就能看到它。

你的代码应该看起来像`chapter11.03-calendar`。

# 在页面内进行授权配置的最佳方式是什么？

在 Spring Security 4 中，Thymeleaf Spring Security `<sec>`标签的重大进展消除了许多关于在库先前版本中使用此标签的担忧。在许多情况下，标签的`authorize-url`属性能适当地隔离代码，使其不受授权规则变化的影响。你应该在以下场景中使用标签的`authorize-url`属性：

+   这个标签阻止了可以通过单个 URL 明确识别的显示功能

+   标签的内容可以明确地隔离到一个单独的 URL

不幸的是，在典型的应用程序中，你能够频繁使用标签的`authorize-url`属性的可能性相当低。现实情况是，应用程序通常比这更复杂，在决定渲染页面的部分时需要更复杂的逻辑。

使用 Thymeleaf Spring Security 标签库来基于安全标准声明渲染页面的部分区域是很有诱惑力的。然而，（在许多情况下）这样做并不是一个好主意，原因如下：

+   标签库不支持复杂条件超出角色成员。例如，如果我们的应用程序在`UserDetails`实现中包含了自定义属性、IP 过滤器、地理位置等，这些都不支持标准的`<sec>`标签。

+   这些可能会通过自定义标签或使用 SpEL 表达式来支持。即使在這種情況下，页面也更可能直接与业务逻辑相关联，而不是通常所鼓励的。

+   `<sec>`标签必须在每页中引用它被使用的页面。这可能导致预期为公共规则集之间潜在的不一致，这些规则集可能分布在不同的物理页面之间。一个好的面向对象系统设计建议将条件规则评估放在一个地方，从应该应用的地方逻辑上引用。

+   有可能（并且我们通过包含常用的页面头部的示例来说明）封装和复用页面的一部分以减少这类问题的发生，但在复杂的应用程序中几乎不可能完全消除。

+   没有办法在编译时验证规定的规则的正确性。虽然在典型的基于 Java 的对象导向系统中可以使用编译时常量，但标签库在典型使用中要求硬编码角色名称，而简单的拼写错误可能会在一段时间内未被发现。

+   公平地说，这样的错误可以通过对运行中的应用程序进行全面的函数测试轻松捕捉到，但使用标准的 Java 组件单元测试技术来测试它们要容易得多。

+   我们可以看到，尽管基于**模板的**条件内容渲染方法很方便，但存在一些显著的缺点。

所有这些问题都可以通过在控制器中使用代码来解决，该代码可用于将数据推送到应用程序视图模型。此外，在代码中执行高级授权确定允许重用、编译时检查以及模型、视图和控制器之间的适当逻辑分离。

# 方法级安全

到目前为止，本书的主要关注点是保护 JBCP 日历应用程序的面向网络的部分；然而，在实际规划安全系统时，应同样关注保护允许用户访问系统最核心部分的服务方法——即数据。

# 为什么我们要分层保护？

让我们花一分钟来了解为什么即使我们已经保护了我们的 URL，确保方法的安全也是重要的。启动 JBCP 日历应用程序。使用 `user1@example.com` 作为用户名和 `user1` 作为密码登录，然后访问所有事件页面。你会看到自定义的访问被拒绝页面。现在，在浏览器的 URL 末尾添加 `.json`，使 URL 变为 `https://localhost:8443/events/.json`。你现在会看到一个带有与 HTML 所有事件页面相同数据的 JSON 响应。这部分数据只应由管理员可见，但我们通过找到一个配置不正确的 URL 来绕过它。

我们还可以查看我们不拥有且未受邀的活动的详细信息。将 `.json` 替换为 `102`，使 URL 变为 `https://localhost:8443/events/102`。你现在会看到一个在“我的活动”页面中未列出的午餐事件。这不应该对我们可见，因为我们不是管理员，而且这不是我们的活动。

如您所见，我们的 URL 规则不足以完全保护我们的应用程序。这些攻击甚至不需要利用更复杂的问题，例如容器处理 URL 标准化方式的差异。简而言之，通常存在绕过基于 URL 的安全性的方法。让我们看看向我们的业务层添加安全层如何帮助我们解决新的安全漏洞。

# 保护业务层

**Spring Security** 具备为应用程序中任何由 Spring 管理的 bean 调用添加一层授权（或基于授权的数据修剪）的能力。虽然许多开发者关注的是网络层的安全性，但业务层的安全性同样重要，因为恶意用户可能能够穿透网络层的安全或通过非 UI 前端（如 Web 服务）访问服务。

让我们查看以下逻辑图，了解为什么我们感兴趣于应用第二层安全：

![](img/357835b8-4d31-4cce-8b41-c5d7c70c0909.png)

Spring Security 用于保护方法的主要技术有以下两种：

+   **预授权**：这种技术确保在允许执行的方法执行之前满足某些约束条件，例如，如果用户具有特定的 `GrantedAuthority`，如 `ROLE_ADMIN`。未能满足声明的约束条件意味着方法调用将失败。

+   **后授权**：这种技术确保在方法返回后，调用者仍然满足声明的约束条件。这种方法很少使用，但可以为一些复杂、相互连接的业务层方法提供额外的安全层。

预授权和后授权技术为在经典、面向对象设计中通常称为前提条件和后置条件提供了正式化的支持。前提条件和后置条件允许开发者在运行时检查，声明方法执行过程中某些约束必须始终成立。在安全预授权和后授权的情况下，业务层开发者通过将期望的运行时条件作为接口或类 API 声明的一部分来编码，对特定方法的安全配置有一个明确的决策。正如你所想象的，这需要大量的深思熟虑，以避免意想不到的后果！

# 添加@PreAuthorize 方法注解

我们的第一个设计决策将是通过确保用户必须以`ADMIN`用户身份登录后才能访问`getEvents()`方法，在业务层增强方法安全性。这是通过在服务接口定义中的方法上添加一个简单的注解来完成的，如下所示：

```java
    import org.springframework.security.access.prepost.PreAuthorize;
    ...
    public interface CalendarService {
       ...
     @PreAuthorize("hasRole('ADMIN')")
      List<Event> getEvents();
    }
```

这就是确保调用我们`getEvents()`方法的人是管理员所需要的一切。Spring Security 将在运行时使用**面向切面编程**（**AOP**）的**BeforeAdvice**切入点在方法上执行，如果安全约束不被满足，将抛出`o.s.s.access.AccessDeniedException`。

# 指导 Spring Security 使用方法注解

我们还需要对`SecurityConfig.java`进行一次性的更改，我们在那里有剩下的 Spring Security 配置。只需在类声明中添加以下注解：

```java
//src/main/java/com/packtpub/springsecurity/configuration/SecurityConfig.java

@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
```

# 验证方法安全性

你相信就这样容易吗？用用户名`user1@example.com`和密码`user1`登录，尝试访问`https://localhost:8443/events/.json`。现在你应该看到访问被拒绝的页面。

你的代码应该看起来像`chapter11.04-calendar`。

如果你查看 Tomcat 控制台，你会看到一个非常长的堆栈跟踪，以以下输出开始：

```java
    DEBUG ExceptionTranslationFilter - Access is denied 
    (user is not anonymous); delegating to AccessDeniedHandler
    org.s.s.access.AccessDeniedException: Access is denied
    at org.s.s.access.vote.AffirmativeBased.decide
    at org.s.s.access.intercept.AbstractSecurityInterceptor.
    beforeInvocation
    at org.s.s.access.intercept.aopalliance.
    MethodSecurityInterceptor.invoke
    ...
    at $Proxy16.getEvents
    at com.packtpub.springsecurity.web.controllers.EventsController.events
```

基于访问被拒绝页面，以及堆栈跟踪明确指向`getEvents`方法的调用，我们可以看出用户因为缺少`ROLE_ADMIN`的`GrantedAuthority`而被适当地拒绝了访问业务方法的权限。如果你用用户名`admin1@example.com`和密码`admin1`来运行相同的操作，你会发现访问将被授予。

通过在接口中简单声明，我们就能够确保所讨论的方法是安全的，这难道不令人惊叹吗？但是 AOP 是如何工作的呢？

# 基于接口的代理

在前面的章节中给出的示例中，Spring Security 使用基于接口的代理来保护我们的`getEvents`方法。让我们看看简化后的伪代码，了解这是如何工作的：

```java
    DefaultCalendarService originalService = context.getBean
    (CalendarService.class)
    CalendarService secureService = new CalendarService() {
     ¦ other methods just delegate to originalService ...
      public List<Event> getEvents() {
 if(!permitted(originalService.getEvents)) {           throw AccessDeniedException()
          }
```

```java
       return originalCalendarService.getEvents()
      }
   };
```

您可以看到 Spring 创建了原始的`CalendarService`，就像它通常做的那样。然而，它指示我们的代码使用另一个实现`CalendarService`，在返回原始方法的结果之前执行安全检查。安全实现可以在不事先了解我们接口的情况下创建，因为 Spring 使用 Java 的`java.lang.reflect.Proxy` API 动态创建接口的新实现。请注意，返回的对象不再是`DefaultCalendarService`的实例，因为它是一个新的`CalendarService`实现，即它是`CalendarService`的一个匿名实现。这意味着我们必须针对接口编程以使用安全实现，否则会发生`ClassCastException`异常。要了解更多关于 Spring AOP 的信息，请参阅[`static.springsource.org/spring/docs/current/spring-framework-reference/html/aop.html#aop-introduction-proxies`](http://static.springsource.org/spring/docs/current/spring-framework-reference/html/aop.html#aop-introduction-proxies)的 Spring 参考文档。

除了`@PreAuthorize`注解之外，还有几种其他方法可以在方法上声明安全预授权要求。我们可以研究这些不同的方法来保护方法，然后评估在不同情况下它们的优缺点。

# JSR-250 兼容的标准化规则

**JSR-250 通用注解**为 Java 平台定义了一系列注解，其中一些与安全相关，旨在在 JSR-250 兼容的运行时环境中可移植。Spring Framework 作为 Spring 2.x 版本的的一部分，包括 Spring Security 框架，变得与 JSR-250 兼容。

尽管 JSR-250 注解不如 Spring 原生注解表达能力强，但它们的优点是它们提供的声明在实现 Java EE 应用服务器的不同环境中是兼容的，例如 Glassfish 或服务导向的运行时框架，如**Apache Tuscany**。根据您的应用程序的需求和可移植性要求，您可能决定减少特异性与代码的可移植性之间的权衡是值得的。

为了实现我们第一个示例中指定的规则，我们通过执行以下步骤进行一些更改：

1.  首先，我们需要更新我们的`SecurityConfig`文件以使用 JSR-250 注解：

```java
        //src/main/java/com/packtpub/springsecurity/configuration/
        SecurityConfig.java

        @Configuration
        @EnableWebSecurity
 @EnableGlobalMethodSecurity(jsr250Enabled = true)        public class SecurityConfig extends WebSecurityConfigurerAdapter {
```

1.  最后，需要将`@PreAuthorize`注解更改为`@RolesAllowed`注解。正如我们所预期的，`@RolesAllowed`注解不支持 SpEL 表达式，因此我们按照如下方式编辑`CalendarService`：

```java
        @RolesAllowed("ROLE_ADMIN")
        List<Event> getEvents();
```

1.  重新启动应用程序，以`user1@example.com`/`user1`的身份登录，尝试访问`http://localhost:8080/events/.json`。您应该再次看到访问被拒绝的页面。

您的代码应如下所示：`chapter11.05-calendar`。

请注意，还可以使用 Java 5 标准字符串数组注解语法提供允许的`GrantedAuthority`名称列表：

```java
    @RolesAllowed({"ROLE_USER","ROLE_ADMIN"})
    List<Event> getEvents();
```

还有 JSR-250 指定的两个额外的注解，分别是`@PermitAll`和`@DenyAll`，它们的功能如你所料，允许或拒绝所有对所述方法的请求。

**类级别的注解**

请注意，方法级别的安全注解也可以应用于类级别！如果提供，方法级别的注解将始终覆盖类级别指定的注解。这在你需要为整个类指定安全策略时很有帮助。在使用此功能时，请确保与良好的注释和编码标准相结合，以便开发人员非常清楚类及其方法的安全特性。

# 使用 Spring 的@Secured 注解的方法安全

Spring 本身提供了一种更简单的注解风格，与 JSR-250 的`@RolesAllowed`注解类似。`@Secured`注解在功能和语法上与`@RolesAllowed`相同。唯一的显著差异是它不需要外部依赖，不能被其他框架处理，并且这些注解的处理必须通过`@EnableGlobalMethodSecurity`注解的另一个属性明确启用：

```java
    //src/main/java/com/packtpub/springsecurity/configuration/SecurityConfig.java

    @EnableWebSecurity(debug = true)
    @EnableGlobalMethodSecurity(securedEnabled=true)
    public class SecurityConfig extends WebSecurityConfigurerAdapter {
```

由于`@Secured`函数与 JSR 标准`@RolesAllowed`注解相同，所以在新的代码中没有真正的强制性理由使用它，但在较老的 Spring 代码中可能会遇到它。

# 包含方法参数的方法安全规则

从逻辑上讲，在约束中引用方法参数的编写规则对于某些类型的操作来说似乎是合理的。例如，对我们来说限制`findForUser(int userId)`方法以满足以下约束可能是合理的：

+   `userId`参数必须等于当前用户的 ID

+   用户必须是管理员（在这种情况下，用户可以看到任何事件）

虽然我们可以很容易地看到如何修改规则，以限制方法调用仅限于管理员，但不清如何确定用户是否正在尝试更改自己的密码。

幸运的是，Spring Security 方法注解使用的 SpEL 绑定支持更复杂的表达式，包括包含方法参数的表达式。你还需要确保已经在`SecurityConfig`文件中启用了前缀和后缀注解，如下所示：

```java
    //src/main/java/com/packtpub/springsecurity/configuration/SecurityConfig.java

    @Configuration
    @EnableWebSecurity
    @EnableGlobalMethodSecurity(prePostEnabled = true)
    public class SecurityConfig extends WebSecurityConfigurerAdapter {
    Lastly, we can update our CalendarService interface as follows:
    @PreAuthorize("hasRole('ADMIN') or principal.id == #userId")  
    List<Event> findForUser(int userId);
```

在这里，我们可以看到我们已经用我们在第一个练习中使用的 SpEL 指令增强了对主体的 ID 和`userId`方法参数的检查（`#userId`，方法参数名称前缀有一个`#`符号）。这个强大的方法参数绑定的特性应该能激发你的创造力，并允许你用非常精确的逻辑规则来保护方法调用。

由于自第三章以来的自定义身份验证设置，我们的主体目前是一个`CalendarUser`实例 *自定义身份验证*。这意味着主体具有我们`CalendarUser`应用程序上的所有属性。如果我们没有进行这种自定义，只有`UserDetails`对象的属性才可用。

SpEL 变量使用哈希（`#`）前缀引用。一个重要的注意事项是，为了在运行时可用，必须在编译后保留调试符号表信息。保留调试符号表信息的常用方法如下所示：

+   如果您正在使用`javac`编译器，您在构建类时需要包含`-g`标志。

+   在使用 Ant 中的`<javac>`任务时，添加属性`debug="true"`。

+   在 Gradle 中，确保在运行主方法或`bootRun`任务时添加`--debug`。

+   在 Maven 中，确保`maven.compiler.debug=true`属性（默认值为`true`）。

查阅您的编译器、构建工具或 IDE 文档，以获取在您的环境中配置相同设置的帮助。

启动你的应用程序，并尝试使用`user1@example.com`作为用户名和`user1`作为密码登录。在欢迎页面，请求`My Events`（`email=admin1@example.com`）链接以看到“访问被拒绝”的页面。再次尝试`My Events`（`email=user1@example.com`）以使其工作。请注意，在“我的事件”页面上显示的用户与当前登录的用户匹配。现在，尝试相同的步骤并以`admin1@example.com`/`admin1`登录。你将能够看到两个页面，因为你是以具有`ROLE_ADMIN`权限的用户登录。

你的代码应该看起来像`chapter11.06-calendar`。

# 包含返回值的方法安全规则。

正如我们能够利用方法的参数一样，我们也可以利用方法调用的返回值。让我们更新`getEvent`方法，以满足返回值的以下约束：

+   参与者的 ID 必须是当前用户的 ID。

+   所有者的 ID 必须是当前用户的 ID。

+   用户必须是管理员（在这种情况下，用户可以看到任何事件是有效的）。

将以下代码添加到`CalendarService`接口中：

```java
    @PostAuthorize("hasRole('ROLE_ADMIN') or " + "principal.username ==   
    returnObject.owner.email or " +
    "principal.username == returnObject.attendee.email")
    Event getEvent(int eventId);
```

现在，尝试使用用户名`user1@example.com`和密码`user1`登录。接下来，尝试通过欢迎页面的链接访问午餐事件。你现在应该会看到“访问被拒绝”的页面。如果你使用用户名`user2@example.com`和密码`user2`登录，由于`user2@example.com`是午餐事件的参与者，事件将如预期显示。

你的代码应该看起来像`chapter11.07-calendar`。

# 使用基于角色的过滤来保护方法数据。

最后两个依赖于 Spring Security 的注解是`@PreFilter`和`@PostFilter`，它们用于将安全过滤规则应用于集合或数组（仅限`@PostFilter`）。这种功能被称为安全修剪或安全修剪，涉及在运行时使用`principal`的安全凭证来有选择地从一组对象中移除成员。正如你所期望的，这种过滤是使用注解声明中的 SpEL 表达式完成的。

我们将通过一个 JBCP 日历的例子来工作，因为我们想要过滤`getEvents`方法，使其只返回这个用户被允许看到的活动。为了做到这一点，我们移除了任何现有的安全注解，并在我们的`CalendarService`接口中添加了`@PostFilter`注解，如下所示：

```java
    @PostFilter("principal.id == filterObject.owner.id or " + 
    "principal.id == filterObject.attendee.id")
    List<Event> getEvents();
```

你的代码应该看起来像这样：`chapter11.08-calendar`。

删除`antMatchers()`方法，限制对`/events/URL`的访问，以便我们可以测试我们的注解。启动应用程序，登录使用用户名`user1@example.com`和密码`user1`后查看所有活动页面，你会观察到只显示与我们用户相关联的活动。

`filterObject`作为循环变量，指的是当前活动，Spring Security 将遍历我们服务返回的`List<Event>`，并修改它，使其只包含匹配我们的 SpEL 表达式的`Event`对象。

通常，`@PostFilter`方法的行为如下。为了简洁起见，我们称集合为方法返回值，但请注意`@PostFilter`既适用于集合也适用于数组方法返回类型。

`filterObject`对象被重新绑定到 SpEL 上下文中，对于集合中的每个元素。这意味着如果你的方法返回一个包含 100 个元素的集合，SpEL 表达式将对每个元素进行评估。

SpEL 表达式必须返回一个布尔值。如果表达式计算结果为真，对象将保留在集合中，而如果表达式计算结果为假，对象将被移除。

在大多数情况下，你会发现集合后过滤可以节省你编写样板代码的复杂性，这些代码你本来可能就会写。注意你要理解`@PostFilter`是如何工作的概念；与`@PreAuthorize`不同，`@PostFilter`指定方法行为，而不是预条件。一些面向对象纯洁主义者可能会认为`@PostFilter`不适合作为方法注解包含在内，而这样的过滤应该通过方法实现中的代码来处理。

集合过滤的安全性

请注意，您方法返回的实际集合将被修改！在某些情况下，这不是期望的行为，所以您应该确保您的方法返回一个可以安全修改的集合。如果返回的集合是一个 ORM 绑定的集合，这一点尤为重要，因为后过滤修改可能会无意中持久化到 ORM 数据存储！

Spring Security 还提供了预过滤方法参数的功能；我们现在尝试实现这个功能。

# 使用@PreFilter 预过滤集合

`@PreFilter`注解可以应用于一个方法，根据当前的安全上下文过滤传递给方法的集合元素。功能上，一旦它有一个集合的引用，这个注解的行为与`@PostFilter`注解完全相同，有几个例外：

+   `@PreFilter`注解只支持集合参数，不支持数组参数。

+   `@PreFilter`注解有一个额外的可选`filterTarget`属性，用于特定地标识方法参数，并在注解的方法有多个参数时对其进行过滤。

+   与`@PostFilter`类似，请记住传递给方法的原集合会被永久修改。这可能不是期望的行为，所以要确保调用者知道在方法被调用后集合的安全性可能会被剪裁！

想象如果我们有一个`save`方法，它接受一个事件对象的集合，我们只想允许保存当前登录用户拥有的事件。我们可以这样做：

```java
    @PreFilter("principal.id == filterObject.owner.id")
    void save(Set<Event> events);
```

与我们的`@PostFilter`方法类似，这个注解导致 Spring Security 遍历每个事件，循环变量`filterObject`。然后，它将当前用户的 ID 与事件所有者的 ID 进行比较。如果它们匹配，保留该事件。如果不匹配，则丢弃结果。

# 比较方法授权类型

以下快速参考图表可能有助于您选择使用哪种方法授权检查类型：

| **方法授权类型** | **指定为** | **JSR 标准** | **允许 SpEL 表达式** |
| --- | --- | --- | --- |
| `@PreAuthorize``@PostAuthorize` | 注解 | 否 | 是 |
| `@RolesAllowed`, `@PermitAll`, `@DenyAll` | 注解 | 是 | 否 |
| `@Secure` | 注解 | 否 | 否 |
| `protect-pointcut` | XML | 否 | 否 |

大多数使用 Spring Security 的 Java 5 消费者可能会选择使用 JSR-250 注解，以实现最大程度的兼容性，并在 IT 组织中重用他们的业务类（和相关约束）。在需要时，这些基本声明可以被与代码绑定到 Spring Security 实现的注解所替代。

如果您在支持注解的环境中使用 Spring Security（Java 1.4 或更早），不幸的是，您的选择相当有限，只能使用方法安全强制。即使在这种情况，AOP 的使用为我们提供了一个相当丰富的环境，我们可以开发基本的安全声明。

# 基于注解的安全性实际考虑

需要考虑的一件事是，当返回现实世界应用的集合时，很可能会有某种形式的分页。这意味着我们的`@PreFilter`和`@PostFilter`注解不能作为选择返回哪些对象的唯手段。相反，我们需要确保我们的查询只选择用户允许访问的数据。这意味着安全注解变成了重复检查。然而，重要的是要记住我们在这章开头学到的教训；我们希望保护层，以防一个层能够被绕过。

# 摘要

在本章中，我们已经涵盖了标准 Spring Security 实现中处理授权的大部分剩余领域。我们已经学到了足够多的知识，可以彻底检查 JBCP 日历应用程序，并验证在应用程序的所有层中是否已经设置了适当的授权检查，以确保恶意用户无法操纵或访问他们无法访问的数据。

我们开发了两种微授权技术，分别是使用 Thymeleaf Spring Security 标签库和 Spring MVC 控制器数据绑定，基于授权或其他安全标准过滤页面内容。我们还探索了几种方法，在应用程序的业务层中保护业务功能和数据，并支持一个紧密集成到代码中的丰富声明式安全模型。我们还学习了如何保护我们的 Spring MVC 控制器以及接口和类代理对象之间的区别。

至此，我们已经涵盖了大多数在标准、安全的网络应用开发场景中可能遇到的 Spring Security 功能。

在下一章中，我们将讨论 Spring Security 的 ACL（域对象模型）模块。这将允许我们显式声明授权，而不是依赖现有数据。

# 访问控制列表

在本章中，我们将讨论复杂的话题**访问控制列表**（**ACL**），它可以提供一个丰富的域对象实例级授权模型。Spring Security 附带了一个健壮但复杂的访问控制列表模块，可以满足小型到中型实现的合理需求。

在本章中，我们将介绍以下主题：

+   理解 ACL 的概念模型

+   回顾 Spring Security ACL 模块中 ACL 概念的术语和应用

+   构建和支持 Spring ACL 所需的数据库架构

+   通过注解和 Spring Bean 配置 JBCP 日历以使用 ACL 安全的企业方法

+   执行高级配置，包括自定义 ACL 权限、ACL 启用的 JSP 标签检查和方法安全、可变 ACL 和智能缓存

+   审查 ACL 部署的架构考虑和计划场景

# ACL 的概念模块

非网络层安全拼图的最后一块是业务对象级别的安全，应用于或低于业务层。在这个层次上，使用一种称为 ACL 的技术实现安全。用一句话总结 ACL 的目标-ACL 允许基于组、业务对象和逻辑操作的独特组合指定一组权限。

例如，JBCP 日历的 ACL 声明可能声明给定用户对其自己的事件具有写入权限。这可以表示如下：

| **用户名** | **组** | **对象** | **权限** |
| --- | --- | --- | --- |
| `mick` |  | `event_01` | `read`, `write` |
|  | `ROLE_USER` | `event_123` | `read` |
|  | `ANONYMOUS` | 任何事件 | `none` |

您可以看到，这个 ACL 对人类来说是极易读的-`mick`有读取和写入自己事件（`event_01`）的权限；其他注册用户可以读取`mick`的事件，但匿名用户不能。这种规则矩阵，简而言之，就是 ACL 试图将安全系统及其业务数据合成代码、访问检查和元数据的组合。大多数真正的 ACL 支持系统具有极其复杂的 ACL 列表，在整个系统可能会有数百万条记录。尽管这听起来非常复杂，但使用有能力的安全库进行适当的预先推理和实施可以使 ACL 管理变得可行。

如果您使用的是 Microsoft Windows 或 Unix/Linux-based 计算机，您每天都会体验到 ACL 的魔力。大多数现代计算机操作系统在其文件存储系统中使用 ACL 指令，允许基于用户或组、文件或目录以及权限的组合来授予权限。在 Microsoft Windows 中，您可以通过右键单击文件并查看其安全属性（属性 | 安全）来查看文件的一些 ACL 功能，如下面的屏幕截图所示：

![](img/83387a1a-85dd-4ca8-be30-b6a071debe15.png)

您将能够看到，ACL 的输入组合在您通过各种组或用户和权限导航时是可见且直观的。

# Spring Security 中的访问控制列表（ACL）

Spring Security 支持针对 secured system 中个别用户的个别域对象的 ACL 驱动的授权检查。就像在 OS 文件系统示例中一样，可以使用 Spring Security ACL 组件构建逻辑树结构，包括业务对象和组或主体。请求者和请求对象上权限（继承或明确）的交集用于确定允许的访问。

用户在接触 Spring Security 的 ACL（访问控制列表）功能时，常常会因其复杂性而感到不知所措，加上相关文档和示例的相对匮乏，这种情况更是加剧。这还因为设置 ACL 基础架构可能相当复杂，有许多相互依赖性，并且依赖于基于 bean 的配置机制，这与 Spring Security 的其他部分大不相同（正如您在设置初始配置时所看到的）。

Spring Security ACL 模块是为了提供一个合理的基线而编写的，但是打算在功能上进行大量扩展的用户可能会遇到一系列令人沮丧的限制和设计选择，这些限制和设计选择在 Spring Security 的早期阶段首次引入后（在很大程度上）一直没有得到纠正。不要让这些限制让您气馁！ACL 模块是一种在您的应用程序中嵌入丰富访问控制的有效方式，并进一步审视和保护用户行为和数据。

在我们深入配置 Spring Security ACL 支持之前，我们需要回顾一些关键的术语和概念。

在 Spring ACL 系统中，安全身份的主要单位是**安全身份**（**SID**）。SID 是一个逻辑构造，可以用来抽象单个主题或组（`GrantedAuthority`）的身份。您构建的 ACL 数据模型中定义的`SIDs`对象用作确定特定主体的允许访问级别的明确和派生访问控制规则的基础。

如果使用`SIDs`在 ACL 系统中定义参与者，那么安全方程的另一部分就是被保护对象本身的定义。单个受保护对象的识别称为对象身份（unsurprisingly）。默认的 Spring ACL 实现要求在单个对象实例级别定义 ACL 规则，这意味着，如果需要，系统中的每个对象都可以有各自的访问规则。

个别访问规则被称为**访问控制条目**（**ACEs**）。一个 ACE 是以下因素的组合：

+   适用于演员的 SID

+   规则适用的对象身份

+   应应用于给定`SID`和所述对象身份的权限

+   是否应该允许或拒绝给定`SID`和对象身份的声明权限

Spring ACL 系统的整体目的是评估每个受保护的方法调用，并根据适用的 ACE 确定是否应该允许被方法操作的对象或对象。适用的 ACE 在运行时根据调用者和参与其中的对象进行评估。

Spring Security ACL 在其实现中是灵活的。尽管本章的大部分内容详细介绍了 Spring Security ACL 模块的默认功能，但请记住，许多指示的规则代表默认实现，在许多情况下可以根据更复杂的要求进行覆盖。

Spring Security 使用有用的值对象来表示与这些概念实体相关的数据。这些如下表所示：

| **ACL 概念对象** | **Java 对象** |
| --- | --- |
| SID | `o.s.s.acls.model.Sid` |
| 对象身份 | `o.s.s.acls.model.ObjectIdentity` |
| ACL | `o.s.s.acls.model.Acl` |
| ACE | `o.s.s.acls.model.AccessControlEntry` |

让我们通过启用 Spring Security ACL 组件的过程，对 JBCP 日历应用程序进行简单的演示。

# 基本配置 Spring Security ACL 支持

虽然我们之前暗示过，在 Spring Security 中配置 ACL 支持需要基于 bean 的配置（确实如此），但如果你愿意，你可以保留更简单的安全 XML 命名空间配置同时使用 ACL 支持。在本章的剩余示例中，我们将关注基于 Java 的配置。

# Gradle 依赖关系

与本书的大部分章节一样，我们需要添加一些依赖项才能使用本章的功能。可以查看以下内容，了解我们添加的依赖项及其需要的时机：

```java
    build.gradle
    dependencies {
       // ACL
       compile('org.springframework.security:spring-security-acl')
      compile('net.sf.ehcache:ehcache')
       ...
    }
```

# 定义一个简单的目标场景

我们的简单目标场景是只授予`user2@example.com`对生日派对事件的阅读权限。其他所有用户将无法访问任何事件。你会注意到这与我们其他例子有所不同，因为`user2@example.com`与其他任何事件都没有关联。

虽然设置 ACL 检查有几种方法，但我们更喜欢遵循本章中方法级注解使用的基于注解的方法。这很好地将 ACL 的使用从实际的接口声明中抽象出来，并允许在稍后的日期（如果你愿意）用其他东西替换角色声明（如果你选择的话）。

我们将向`CalendarService.getEvents`方法添加一个注解，根据当前用户对事件的权限过滤每个事件：

```java
    src/main/java/com/packtpub/springsecurity/service/CalendarService.java
    @PostFilter("hasPermission(filterObject, 'read')")
    List<Event> getEvents();
```

你应该从`chapter12.00-calendar`开始。

# 向 H2 数据库添加 ACL 表

我们首先需要做的是向我们的内存中添加所需的支持持久 ACL 条目的表和数据。为此，我们将添加一个新的 SQL DDL 文件及其对应的数据到`schema.sql`中的嵌入式数据库声明。我们将在本章后面分解这些文件。

我们在此章节的源代码中包括了以下`schema.sql`文件，该文件基于 Spring Security 参考附录中的架构文件，即*附加参考材料*：

```java
src/main/resources/schema.sql
-- ACL Schema --
create table acl_sid (
id bigint generated by default as identity(start with 100) not
   null primary key,
principal boolean not null,
sid varchar_ignorecase(100) not null,
constraint uk_acl_sid unique(sid,principal) );

create table acl_class (
id bigint generated by default as identity(start with 100) not
   null primary key,
class varchar_ignorecase(500) not null,
constraint uk_acl_class unique(class) );

create table acl_object_identity (
id bigint generated by default as identity(start with 100) not
   null primary key,
object_id_class bigint not null,
object_id_identity bigint not null,
parent_object bigint,
owner_sid bigint not null,
entries_inheriting boolean not null,
constraint uk_acl_objid
   unique(object_id_class,object_id_identity),
constraint fk_acl_obj_parent foreign
   key(parent_object)references acl_object_identity(id),
constraint fk_acl_obj_class foreign
   key(object_id_class)references acl_class(id),
constraint fk_acl_obj_owner foreign key(owner_sid)references
   acl_sid(id) );

create table acl_entry (
id bigint generated by default as identity(start with 100) not
   null primary key,
acl_object_identity bigint not null,
ace_order int not null,
sid bigint not null,
mask integer not null,
granting boolean not null,
audit_success boolean not null,
audit_failure boolean not null,
constraint uk_acl_entry unique(acl_object_identity,ace_order),
constraint fk_acl_entry_obj_id foreign key(acl_object_identity)
references acl_object_identity(id),
constraint fk_acl_entry_sid foreign key(sid) references
   acl_sid(id) );
```

前面的代码将导致以下数据库架构：

![](img/8267833a-9f6b-47fe-ab50-c901d55ac3fe.png)

你可以看到`SIDs`、`OBJECT_IDENTITY`和 ACEs 的概念是如何直接映射到数据库架构的。从概念上讲，这是方便的，因为我们可以将我们对 ACL 系统的心理模型以及它是如何执行的直接映射到数据库。

如果你将此与随 Spring Security 文档提供的 H2 数据库架构进行了交叉引用，你会注意到我们做了一些常见的用户陷阱的调整。这些如下：

+   将`ACL_CLASS.CLASS`列更改为`500`个字符，默认值为`100`。一些长完全限定类名不适合`100`个字符。

+   给外键命名一些有意义的名称，以便故障诊断更加容易。

如果您使用其他数据库，如 Oracle，您将不得不将 DDL 翻译成特定于您数据库的 DDL 和数据类型。

一旦我们配置了 ACL 系统的其余部分，我们将回到数据库中设置一些基本的 ACE，以最原始的形式证明 ACL 功能。

# 配置 SecurityExpressionHandler

我们需要配置`<global-method-security>`以启用注解（我们将基于预期的 ACL 权限进行注解），并引用一个自定义的访问决策管理器。

我们还需要提供一个`o.s.s.access.expression.SecurityExpressionHandler`实现，使其知道如何评估权限。更新您的`SecurityConfig.java`配置，如下所示：

```java
    src/main/java/com/packtpub/springsecurity/configuration/SecurityConfig.java 
 @EnableGlobalMethodSecurity(prePostEnabled = true)    @Import(AclConfig.class)
    public class SecurityConfig extends WebSecurityConfigurerAdapter {
```

这是对我们在`AclConfig.java`文件中定义的`DefaultMethodSecurityExpressionHandler`对象的 bean 引用，如下所示：

```java
    src/main/java/com/packtpub/springsecurity/configuration/AclConfig.java
    @Bean
    public DefaultMethodSecurityExpressionHandler expressionHandler(){
       DefaultMethodSecurityExpressionHandler dmseh =
       new DefaultMethodSecurityExpressionHandler();
      dmseh.setPermissionEvaluator(permissionEvaluator());
       dmseh.setPermissionCacheOptimizer(permissionCacheOptimizer());
       return dmseh; 
    }
```

即使在我们 scenario 中有一个相对简单的 ACL 配置，也有许多必须设置的依赖项。如我们之前提到的，Spring Security ACL 模块默认包含一组组件，您可以组装这些组件以提供一套不错的 ACL 功能。请注意，以下图表中引用的所有组件都是框架的一部分：

![](img/3d48ab85-ba85-4e90-bfe8-127df7e137a4.png)

# AclPermissionCacheOptimizer 对象

`DefaultMethodSecurityExpressionHandler` 对象有两个依赖。`AclPermissionCacheOptimizer` 对象用于用单个 JDBC 选择语句为对象集合的所有 ACL 填充缓存。本章包含的相对简单的配置可以按照如下方式进行检查：

```java
     src/main/java/com/packtpub/springsecurity/configuration/AclConfig.java
     @Bean
    public AclPermissionCacheOptimizer permissionCacheOptimizer(){
       return new AclPermissionCacheOptimizer(aclService());
    }
```

# 优化 AclPermission 缓存

然后 `DefaultMethodSecurityExpressionHandler` 对象委派给一个 `PermissionEvalulator` 实例。在本章中，我们使用 ACL 以便我们使用的 bean `AclPermissionEvaluator`，它将读取我们在数据库中定义的 ACL。您可以查看提供的 `permissionEvaluator` 配置，如下所示：

```java
src/main/java/com/packtpub/springsecurity/configuration/AclConfig.java
@Bean
public AclPermissionEvaluator permissionEvaluator(){
   return new AclPermissionEvaluator(aclService());
}
```

# `JdbcMutableAclService` 对象

在此点，我们看到了两次带有 `aclService` ID 的 `th` 引用。`aclService` ID 解析为一个负责将有关通过 ACL 受保护的对象的信息翻译成预期 ACE 的 `o.s.s.acls.model.AclService` 实现：

```java
src/main/java/com/packtpub/springsecurity/configuration/AclConfig.java
@Autowired 
private DataSource dataSource;
@Bean
public JdbcMutableAclService aclService(){
```

```java
   return new JdbcMutableAclService(dataSource,
                                     lookupStrategy(),
                                     aclCache());
}
```

我们将使用 `o.s.s.acls.jdbc.JdbcMutableAclService`，这是 `o.s.s.acls.model.AclService` 的默认实现。这个实现开箱即用，准备好使用我们在本练习的最后一步定义的架构。`JdbcMutableAclService` 对象还将使用递归 SQL 和后处理来理解对象和 `SID` 层次结构，并确保这些层次结构的表示被传递回 `AclPermissionEvaluator`。

# 基本查找策略类

`JdbcMutableAclService` 类使用了与我们定义的嵌入式数据库声明相同的 JDBC `dataSource` 实例，并且它还委派给 `o.s.s.acls.jdbc.LookupStrategy` 的一个实现，该实现专门负责实际执行数据库查询和解析 ACL 请求。Spring Security 提供的唯一 `LookupStrategy` 实现是 `o.s.s.acls.jdbc.BasicLookupStrategy`，如下定义：

```java
src/main/java/com/packtpub/springsecurity/configuration/AclConfig.java
@Bean
public LookupStrategy lookupStrategy(){
   return new BasicLookupStrategy(
           dataSource,
           aclCache(),
           aclAuthorizationStrategy(),
           consoleAuditLogger());
}
```

现在，`BasicLookupStrategy` 是一个相当复杂的生物。记住它的目的是将需要保护的 `ObjectIdentity` 声明列表翻译成实际适用的数据库中的 ACE 列表。由于 `ObjectIdentity` 声明可以是递归的，这证明是一个非常具有挑战性的问题，并且一个可能会经历大量使用的系统应考虑生成的 SQL 对数据库性能的影响。

# 使用最低公倍数查询

请注意，`BasicLookupStrategy` 旨在通过严格遵循标准 ANSI SQL 语法与所有数据库兼容，特别是 `left [outer] joins`。一些较老的数据库（特别是 **Oracle8i**）不支持这种连接语法，所以请确保您验证 SQL 的语法和结构与您特定的数据库兼容！

肯定还有更多高效的数据库依赖方法执行层次查询，使用非标准 SQL，例如，Oracle 的`CONNECT BY`语句和其他许多数据库（包括 PostgreSQL 和 Microsoft SQL Server）的**公共表表达式**（**CTE**）功能。

正如你在第四章的例子中学习到的，*基于 JDBC 的认证*，使用自定义架构为`JdbcDaoImpl`实现的`UserDetailsService`属性暴露出来，允许配置`BasicLookupStrategy`使用的 SQL。查阅 Javadoc 和源代码本身，看看它们是如何使用的，这样它们就可以正确地应用到你的自定义架构上。

我们可以看到`LookupStrategy`需要引用与 AclService 使用的相同的 JDBC`dataSource`实例。其他三个引用让我们几乎到达依赖链的末端。

# `EhCacheBasedAclCache`

`o.s.s.acls.model.AclCache`接口声明了一个缓存`ObjectIdentity`到 ACL 映射的接口，以防止重复（且昂贵）的数据库查询。Spring Security 只包含一个`AclCache`的实现，使用了第三方库`Ehcache`。

`Ehcache`是一个开源的基于内存和磁盘的缓存库，在许多开源和商业 Java 产品中被广泛使用。正如本章前面提到的，Spring Security 包含一个 ACL 缓存的默认实现，它依赖于一个配置好的`Ehcache`实例，它使用这个实例来存储 ACL 信息，而不是从数据库中读取 ACL。

虽然深入配置`Ehcache`不是我们本节想要覆盖的内容，但我们会介绍 Spring ACL 如何使用缓存，并带你走过一个基本的默认配置。

设置`Ehcache`很简单——我们只需声明`o.s.s.acls.domain.EhCacheBasedAclCache`以及从 Spring Core 中它的两个依赖 bean，这些 bean 管理`Ehcache`的实例化和暴露几个有用的配置属性。像我们的其他 bean 一样，我们在`AclConfig.java`中已经提供了以下的配置：

```java
src/main/java/com/packtpub/springsecurity/configuration/AclConfig.java
@Bean
public EhCacheBasedAclCache aclCache(){
   return new EhCacheBasedAclCache(ehcache(),
           permissionGrantingStrategy(),
           aclAuthorizationStrategy()
           );
}

@Bean
public PermissionGrantingStrategy permissionGrantingStrategy(){
   return new DefaultPermissionGrantingStrategy(consoleAuditLogger());
}

@Bean
public Ehcache ehcache(){
   EhCacheFactoryBean cacheFactoryBean = new EhCacheFactoryBean();
   cacheFactoryBean.setCacheManager(cacheManager());
   cacheFactoryBean.setCacheName("aclCache");
   cacheFactoryBean.setMaxBytesLocalHeap("1M");
   cacheFactoryBean.setMaxEntriesLocalHeap(0L);
   cacheFactoryBean.afterPropertiesSet();
   return cacheFactoryBean.getObject();
}

@Bean
public CacheManager cacheManager(){
   EhCacheManagerFactoryBean cacheManager = new EhCacheManagerFactoryBean();
   cacheManager.setAcceptExisting(true);   cacheManager.setCacheManagerName(CacheManager.getInstance().getName());
   cacheManager.afterPropertiesSet();
return cacheManager.getObject();
}
```

# `ConsoleAuditLogger`类

悬挂在`o.s.s.acls.jdbc.BasicLookupStrategy`上的下一个简单依赖是一个`o.s.s.acls.domain.AuditLogger`接口的实现，该接口由`BasicLookupStrategy`类用于审计 ACL 和 ACE 查询。与`AclCache`接口类似，Spring Security 只提供了一个简单的日志到控制台的实现。我们将通过另一个单行 bean 声明来配置它：

```java
src/main/java/com/packtpub/springsecurity/configuration/AclConfig.java
@Bean
public ConsoleAuditLogger consoleAuditLogger(){
   return new ConsoleAuditLogger();
}
```

# `AclAuthorizationStrategyImpl`接口

需要解决的最后依赖关系是对`o.s.s.acls.domain.AclAuthorizationStrategy`接口的实现，该接口在从数据库加载 ACL 时实际上没有任何直接的职责。相反，实现此接口负责确定是否允许对 ACL 或 ACE 进行运行时更改，具体取决于更改的类型。我们稍后会在讲解可变 ACL 时解释更多，因为逻辑流程既有点复杂，又与完成初始配置无关。最终的配置要求如下：

```java
src/main/java/com/packtpub/springsecurity/configuration/AclConfig.java
@Bean
public AclAuthorizationStrategy aclAuthorizationStrategy() {
   return new AclAuthorizationStrategyImpl(
           new SimpleGrantedAuthority("ROLE_ADMINISTRATOR")
   );
}
```

您可能想知道 ID 为`adminAuthority`的 bean 的引用是做什么的-`AclAuthorizationStrategyImpl`提供了指定在可变 ACL 上允许特定操作的`GrantedAuthority`的能力。我们将在本章后面覆盖这些内容。

最后，我们需要更新我们的`SecurityConfig.java`文件，以加载我们的`AclConfig.java`文件，如下所示：

```java
src/main/java/com/packtpub/springsecurity/configuration/SecurityConfig.java
@Import(AclConfig.class) public class SecurityConfig extends WebSecurityConfigurerAdapter {
```

我们终于完成了 Spring Security ACL 实现的初始配置。下一步也是最后一步，要求我们将一个简单的 ACL 和 ACE 插入到 H2 数据库中并测试它！

# 创建简单 ACL 条目

回想一下，我们非常简单的场景是只允许`user2@example.com`访问生日派对事件，并确保其他事件无法访问。您可能发现回顾几页到数据库架构图有助于了解我们要插入的数据以及原因。

我们已经在示例应用程序中包含了一个名为`data.sql`的文件。本节中解释的所有 SQL 都将来自该文件-您可以自由地基于我们提供的示例 SQL 进行实验和添加更多测试用例-实际上，我们鼓励您使用示例数据进行实验！

让我们来看看创建简单 ACL 条目的以下步骤：

1.  首先，我们需要为任何或所有具有 ACL 规则的域对象类填充`ACL_CLASS`表-在我们示例的情况下，这仅仅是我们的`Event`类：

```java
        src/main/resources/data.sql
        insert into acl_class (id, class) values (10, 
        'com.packtpub.springsecurity.domain.Event');
```

我们选择为`ACL_CLASS`表使用主键 10 到 19 的数字，为`ACL_SID`表使用 20 到 29 的数字，以此类推。这将有助于更容易理解哪些数据与哪个表相关联。请注意，我们的`Event`表以主键 100 开始。这些便利措施仅为例证目的，不建议在生产环境中使用。

1.  接下来，`ACL_SID`表用与 ACE 关联的`SID`进行初始化。请记住`SID`可以是角色或用户-我们在这里填充角色和`user2@example.com`。

1.  虽然角色的`SID`对象很简单，但用户的`SID`对象并不是那么清晰。对我们来说，用户名用于`SID`。要了解更多关于如何为角色和用户解析`SID`，请参阅`o.s.s.acls.domain.SidRetrievalStrategyImpl`。如果默认值不符合您的需求，可以将自定义的`o.s.s.acls.model.SidRetrievalStrategy`默认值注入到`AclPermissionCacheOptimizer`和`AclPermissionEvaluator`中。在我们的示例中，我们不需要这种自定义，但是如果需要，知道它是可用的：

```java
        src/main/resources/data.sql
        insert into acl_sid (id, principal, sid) values (20, true,  
        'user2@example.com');
        insert into acl_sid (id, principal, sid) values (21, false, 
        'ROLE_USER');
        insert into acl_sid (id, principal, sid) values (22, false, 
        'ROLE_ADMIN');
```

事情开始变得复杂的是`ACL_OBJECT_IDENTITY`表，该表用于声明个别域对象实例、其父（如果有）和所有者`SID`。例如，这个表代表了我们要保护的`Event`对象。我们将插入具有以下属性的行：

+   类型为`Event`的域对象，通过`OBJECT_ID_CLASS`列连接到我们的`ACL_CLASS`表，外键`10`。

+   域对象的主键`100`（`OBJECT_ID_IDENTITY`列）。这是连接到我们的`Event`对象的外键（尽管不是通过数据库约束强制执行）。

+   拥有者`SID`为`user2@example.com`，这是一个外键，`20`，通过`OWNER_SID`列连接到`ACL_SID`。

表示具有`100`（生日事件）、`101`和`102` ID 的事件的 SQL 如下：

```java
    src/main/resources/data.sql
    insert into acl_object_identity(id,object_id_identity,object_id_class,
    parent_object,owner_sid,entries_inheriting)
    values (30, 100, 10, null, 20, false);
    insert into acl_object_identity(id,object_id_identity,object_id_class,
    parent_object,owner_sid,entries_inheriting) 
    values (31, 101, 10, null, 21, false);
    insert into acl_object_identity(id,object_id_identity,object_id_class,
    parent_object,owner_sid,entries_inheriting)
    values (32, 102, 10, null, 21, false);
```

请记住，拥有的`SID`也可能代表一个角色-就 ACL 系统而言，这两种规则功能是相等的。

最后，我们将向此对象实例添加一个与 ACE 相关的内容，声明`user2@example.com`被允许读取生日事件的权限：

```java
    src/main/resources/data.sql
    insert into acl_entry
   (acl_object_identity, ace_order, sid, mask, granting, audit_success, 
   audit_failure) values(30, 1, 20, 1, true, true, true);
```

这里的`MASK`列代表一个位掩码，它用于授予分配给所述`SID`在问题对象上的权限。我们将在本章后面详细解释这一点-不幸的是，它可能没有听起来那么有用。

现在，我们可以启动应用程序并运行示例场景。尝试使用`user2@example.com`/`user2`登录并访问所有事件页面。您将看到只列出了生日事件。当使用`admin1@example.com`/`admin1`登录并查看所有事件页面时，将不会显示任何事件。但是，如果我们直接导航到某个事件，它将不受保护。您能根据本章学到的知识想出如何保护直接访问事件的方法吗？

如果您还没有弄清楚，您可以通过对`CalendarService`进行以下更新来保护直接访问事件：

```java
    src/main/java/com/packtpub/springsecurity/service/CalendarService.java
    @PostAuthorize("hasPermission(filterObject, 'read') " +
    "or hasPermission(filterObject, 'admin_read')")
    Event getEvent(int eventId);
```

现在，我们已经有了基于 ACL 的安全性的基本工作设置（尽管是一个非常简单的场景）。让我们继续解释一下我们在这次演练中看到的概念，然后回顾一下在典型 Spring ACL 实现中你应该考虑的几个问题。

您的代码应如下所示`chapter12.01-calendar`。

值得注意的是，我们在创建事件时并没有创建新的 ACL 条目。因此，在当前状态下，如果您创建一个事件，您将收到一个类似于以下的错误：

`在 Spring Security 应用程序执行期间发生异常！无法为对象身份 org.springframework.security.acls.domain.ObjectIdentityImpl[Type: com.packtpub.springsecurity.domain.Event; Identifier: 103]找到 ACL 信息`。

# 高级 ACL 主题

我们在配置 ACL 环境时简要介绍的一些高级主题与 ACE 权限及使用`GrantedAuthority`指示器来帮助 ACL 环境确定是否允许对 ACL 进行某些类型的运行时更改有关。现在我们已经有了一个工作环境，我们将回顾这些更高级的主题。

# 权限是如何工作的

权限不过是表示为整数中的位的单个逻辑标识符。访问控制条目根据位掩码向`SIDs`授予权限，该位掩码是适用于该访问控制条目的所有权限的逻辑与。

默认的权限实现，`o.s.s.acls.domain.BasePermission`，定义了一系列代表常见 ACL 授权动词的整数值。这些整数值对应于整数中的单个位设置，所以一个值为`BasePermission`，`WRITE`，整数值为`1`的位掩码值为`21`或`2`。

以下图表进行了说明：

![](img/cde17926-2e18-4572-86a0-c523cd36964f.png)

我们可以看到，**示例**权限位掩码的整数值为**3**，这是由于将**读取**和**写入**权限应用到权限值上。前面图表中显示的所有标准整数单个权限值都是在`BasePermission`对象中作为静态常量定义的。

`BasePermission`中包含的逻辑常量只是 ACE 中常用权限的一个合理基线，并且在 Spring Security 框架内没有语义意义。在非常复杂的 ACL 实现中，通常会发明自己的自定义权限，用领域或业务依赖的权限来补充最佳实践示例。

经常让用户感到困惑的一个问题是，在实际应用中位掩码是如何使用的，因为许多数据库要么不支持位逻辑，要么不支持以可扩展的方式实现位逻辑。Spring ACL 旨在通过将计算与位掩码相关的适当权限的更多负载放在应用程序上，而不是放在数据库上，来解决这一问题。

重要的是要回顾解析过程，在这个过程中我们看到`AclPermissionEvaluator`是如何解析声明在方法本身上的权限（在我们的例子中，使用`@PostFilter`注解）以得到真实的 ACL 权限的。

以下图表说明了 Spring ACL 执行的过程，以将声明的权限与请求主体的相关 ACEs 进行评估：

![](img/3aace789-7e8e-437e-b33d-56dff2233b27.png)

我们发现`AclPermissionEvaluator`依赖于实现两个接口的类，`o.s.s.acls.model.ObjectIdentityRetrievalStrategy`和`o.s.s.acls.model.SidRetrievalStrategy`，以检索适合授权检查的`ObjectIdentity`和`SIDs`。关于这些策略的重要一点是，默认实现类实际上是如何根据授权检查的上下文确定要返回的`ObjectIdentity`和`SIDs`对象的。

`ObjectIdentity`对象有两个属性，`type`和`identifier`，它们是从运行时检查的对象派生的，并用于声明 ACE 条目。默认的`ObjectIdentityRetrievalStrategy`接口使用完全限定类名来填充`type`属性。`identifier`属性用实际对象实例上调用具有`Serializable getId()`签名的方法的返回值填充。

由于你的对象不需要实现接口以与 ACL 检查兼容，开发人员实现 Spring Security ACL 时对需要实现具有特定签名的方法的惊讶是可以理解的。提前规划并确保你的领域对象包含这个方法！你也可以实现自己的`ObjectIdentityRetrievalStrategy`类（或继承内置实现）以调用你选择的方法。不幸的是，方法的名字和类型签名是不可配置的。

不幸的是，`AclImpl`的实际实现直接比较了我们`@PostFilter`注解中指定的 SpEL 表达式中的权限和我们数据库中存储的 ACE 上的权限，而没有使用位逻辑。Spring Security 社区正在争论这是否是无意中发生的还是按预期工作的，但无论如何，当你声明具有多种权限组合的用户时，你必须小心，因为要么必须为`AclEntryVoter`配置所有权限的组合，要么 ACE 需要忽略权限字段是用来存储多个值的意图，而每个 ACE 存储一个权限。

如果你想用我们简单的场景来验证这一点，将我们授予`user2@example.com` SID 的`READ`权限更改为位掩码组合`Read`和`Write`，这翻译为一个值为`3`。这将在`data.sql`文件中更新，如下所示：

```java
    src/main/resources/data.sql
    insert into acl_entry
   (acl_object_identity, ace_order, sid, mask, granting, 
   audit_success, audit_failure) values(30, 1, 20, 3, true, true, true);
```

你的代码应该看起来像`chapter12.02-calendar`。

# 自定义 ACL 权限声明

如前所述，权限声明中的权限不过是整数值的逻辑名称。因此，可以扩展`o.s.s.acls.domain.BasePermission`类并声明您自己的权限。在这里，我们将讨论一个非常简单的场景，即创建一个名为`ADMIN_READ`的新 ACL 权限。这是一个仅授予管理员的权限，用于保护只有管理员才能读取的资源。虽然这个例子对于 JBCP 日历应用程序来说有些牵强，但在处理个人可识别信息（例如，社会保障号码等——回想我们在第一章，*不安全应用程序的解剖学*中讨论的 PII）的情况下，这种自定义权限的使用是非常常见的。

让我们开始进行支持此更改所需的更改，执行以下步骤：

1.  第一步是扩展`BasePermission`类，用我们自己的`com.packtpub.springsecurity.acls.domain.CustomPermission`类，如下所示：

```java
        package com.packtpub.springsecurity.acls.domain;
        public class CustomPermission extends BasePermission {
           public static final Permission ADMIN_READ = new 
           CustomPermission(1 << 5, 'M'); // 32
           public CustomPermission(int mask, char code) {
               super(mask, code);
           }
        }
```

1.  接下来，我们需要配置`o.s.s.acls.domain.PermissionFactory`默认实现，`o.s.s.acls.domain.DefaultPermissionFactory`，以注册我们的自定义权限逻辑值。`PermissionFactory`的作用是将权限位掩码解析为逻辑权限值（在其他应用程序区域中可以通过常量值或名称，如`ADMIN_READ`来引用）。`PermissionFactory`实例需要任何自定义权限都向其注册以进行正确的查找。我们已包含以下配置，注册了我们的`CustomPermission`类，如下所示：

```java
        src/main/java/com/packtpub/springsecurity/configuration/
        AclConfig.java
        @Bean
        public DefaultPermissionFactory permissionFactory(){
         return new DefaultPermissionFactory(CustomPermission.class);
        }
```

1.  接下来，我们需要覆盖我们`BasicLookupStrategy`和`AclPermissionEvaluator`接口的默认`PermissionFactory`实例，使用自定义的`DefaultPermissionFactory`接口。请按照以下步骤更新您的`security-acl.xml`文件：

```java
src/main/java/com/packtpub/springsecurity/configuration/AclConfig.java
@Bean
public AclPermissionEvaluator permissionEvaluator(){
   AclPermissionEvaluator pe = new
                               AclPermissionEvaluator(aclService());
 pe.setPermissionFactory(permissionFactory());   return pe;
}
@Bean
public LookupStrategy lookupStrategy(){
   BasicLookupStrategy ls = new BasicLookupStrategy(
                                       dataSource,
                                       aclCache(),
                                      aclAuthorizationStrategy(),
                                      consoleAuditLogger());
 ls.setPermissionFactory(permissionFactory());   return ls;
}
```

1.  我们还需要添加 SQL 查询，以利用新权限向`admin1@example.com`授予对会议电话（`acl_object_identity ID 为 31`）事件的访问权限。请对`data.sql`进行以下更新：

```java
        src/main/resources/data.sql
        insert into acl_sid (id, principal, sid) values (23, true,   
        'admin1@example.com');
        insert into acl_entry (acl_object_identity, ace_order, sid, 
        mask, granting, audit_success, audit_failure) 
        values(31, 1, 23, 32, true, true, true);
```

我们可以看到，新的整数位掩码值`32`已在 ACE 数据中引用。这故意对应于我们在 Java 代码中定义的新`ADMIN_READ ACL`权限。在`ACL_OBJECT_IDENTITY`表中，会议电话事件通过其主键（存储在`object_id_identity`列）值`31`来引用。

1.  最后一步是更新我们的`CalendarService 的 getEvents()`方法，以利用我们的新权限，如下所示：

```java
        @PostFilter("hasPermission(filterObject, 'read') " + "or    
        hasPermission(filterObject, 'admin_read')")
        List<Event> getEvents();
```

在所有这些配置就绪之后，我们可以重新启动网站并测试自定义 ACL 权限。根据我们配置的示例数据，当各种可用用户点击类别时，会发生以下情况：

| **用户名/密码** | **生日派对事件** | **电话会议事件** | **其他事件** |
| --- | --- | --- | --- |
| ``` |
| ``` |
| ``` |

我们可以看到，即使在我们使用简单示例的情况下，我们现在也能够在非常有限的方式上扩展 Spring ACL 功能，以说明这个细粒度访问控制系统的力量。

你的代码应该看起来像`chapter12.03-calendar`。

# 启用 ACL 权限评估

我们在第二章中看到了 Spring Security JSP 标签库提供了将认证相关数据暴露给用户的功能，以及基于多种规则限制用户能看到的内容。在这本书中，我们一直使用的是建立在 Spring Security 之上的 Thymeleaf 安全标签库。

这个相同的标签库也可以与 ACL 启用的系统无缝交互！从我们的简单实验中，我们已经围绕主页上的前两个类别配置了一个简单的 ACL 授权场景。让我们来看看以下步骤，学习如何在 Thymeleaf 页面中启用 ACL 权限评估：

1.  首先，我们需要从我们的`CalendarService`接口中的`getEvents()`方法移除`@PostFilter`注解，以便给我们的 JSP 标签库一个过滤掉不允许显示的事件的机会。现在就移除`@PostFilter`，如下所示：

``` |
| ```

1.  现在我们已经移除了`@PostFilter`，我们可以使用`<sec:authorize-acl>`标签来隐藏用户实际上没有访问权限的事件。回顾一下前一部分的表格，作为对我们迄今为止配置的访问规则的刷新！

1.  我们将用**`<sec:authorize-acl>`**标签包裹每个事件的显示，声明要检查的对象的权限列表：

``` |

我们可以看到，即使在我们使用简单示例的情况下，我们现在也能够在非常有限的方式上扩展 Spring ACL 功能，以说明这个细粒度访问控制系统的力量。

你的代码应该看起来像`chapter12.03-calendar`。

# 启用 ACL 权限评估

我们在第二章中看到了 Spring Security JSP 标签库提供了将认证相关数据暴露给用户的功能，以及基于多种规则限制用户能看到的内容。在这本书中，我们一直使用的是建立在 Spring Security 之上的 Thymeleaf 安全标签库。

这个相同的标签库也可以与 ACL 启用的系统无缝交互！从我们的简单实验中，我们已经围绕主页上的前两个类别配置了一个简单的 ACL 授权场景。让我们来看看以下步骤，学习如何在 Thymeleaf 页面中启用 ACL 权限评估：

1.  首先，我们需要从我们的`CalendarService`接口中的`getEvents()`方法移除`@PostFilter`注解，以便给我们的 JSP 标签库一个过滤掉不允许显示的事件的机会。现在就移除`@PostFilter`，如下所示：

```

1.  思考一下这里想要发生的事情-我们想要用户只看到他们实际具有`READ`或`ADMIN_READ`（我们的自定义权限）访问的项。然而，为了使用标签库，我们需要使用权限掩码，该掩码可以从以下表格中引用：

| ``` |
| --- |
| ``` |
| ``` |
| ``` |

在幕后，标签实现利用了本章早些时候讨论过的相同的`SidRetrievalStrategy`和`ObjectIdentityRetrievalStrategy`接口。因此，访问检查的计算遵循与 ACL 启用的方法安全投票相同的 workflow。正如我们即将看到的，标签实现还将使用相同的`PermissionEvaluator`。

我们已经用一个引用`DefaultMethodSecurityExpressionHandler`的`expressionHandler`元素配置了我们的`GlobalMethodSecurityConfiguration`。`DefaultMethodSecurityExpressionHandler`实现认识我们的`AclPermissionEvaluator`接口，但我们还必须让 Spring Security 的 web 层认识`AclPermissionEvaluator`。如果你仔细思考，这种对称性是合理的，因为保护和 HTTP 请求是保护两种非常不同的资源。幸运的是，Spring Security 的抽象使这一切变得相当简单。

1.  添加一个引用我们已经定义的 ID 为`permissionEvaluator`的 bean 的`DefaultWebSecurityExpressionHandler`处理器：

```

1.  现在我们已经移除了`@PostFilter`，我们可以使用`<sec:authorize-acl>`标签来隐藏用户实际上没有访问权限的事件。回顾一下前一部分的表格，作为对我们迄今为止配置的访问规则的刷新！

1.  我们将用**`<sec:authorize-acl>`**标签包裹每个事件的显示，声明要检查的对象的权限列表：

```

1.  现在，更新`SecurityConfig.java`以引用我们的`webExpressionHandler`实现，如下所示：

```

1.  思考一下这里想要发生的事情-我们想要用户只看到他们实际具有`READ`或`ADMIN_READ`（我们的自定义权限）访问的项。然而，为了使用标签库，我们需要使用权限掩码，该掩码可以从以下表格中引用：

| ```

你可以看到这些步骤与我们向方法安全添加权限处理的方式非常相似。这次简单一些，因为我们能够重用已经配置的具有`PermissionEvaluator` ID 的相同 bean。

启动我们的应用程序，并以不同的用户身份尝试访问所有事件页面。你会发现，不允许用户查看的事件现在使用我们的标签库隐藏，而不是`@PostFilter`注解。我们知道，直接访问事件会让用户看到它。然而，这可以通过将本章中学到的内容与本章中学到的关于`@PostAuthorize`注解的内容相结合轻松实现。

你的代码应该看起来像`chapter12.04-calendar`。

# 可变 ACL 和授权

尽管 JBCP 日历应用程序没有实现完整的用户管理功能，但您的应用程序很可能会有常见功能，例如新用户注册和行政用户维护。到目前为止，这些功能的缺失（我们通过在应用程序启动时使用 SQL 插入解决了这个问题）并没有阻止我们演示 Spring Security 和 Spring ACL 的许多功能。

然而，正确处理声明 ACL 的运行时更改，或系统中的用户添加或删除，对于维护 ACL 基础授权环境的完整性和安全性至关重要。Spring ACL 通过可变 ACL（`o.s.s.acls.model.MutableAcl`）的概念解决了这个问题。

扩展标准 ACL 接口，`MutableAcl`接口允许在运行时操纵 ACL 字段，以改变特定 ACL 的内存表示。这包括创建、更新或删除 ACE 的能力，更改 ACL 所有者，以及其他有用功能。

我们可能期望，Spring ACL 模块会默认提供一种将运行时 ACL 更改持久化到 JDBC 数据存储的方法，的确如此。可以使用`o.s.s.acls.jdbc.JdbcMutableAclService`类来创建、更新和删除数据库中的`MutableAcl`实例，以及执行 ACL 的其他支持表的一般维护（处理`SIDs`、`ObjectIdentity`和域对象类名）。

在本书的早期部分提到，`AclAuthorizationStrategyImpl`类允许我们为可变 ACL 上的操作指定管理角色。这些是在 bean 配置中作为构造函数的一部分提供的。构造函数参数及其含义如下：

| **参数编号** | **它做什么？** |
| --- | --- |
| 1 | 表示主体需要具有的权限，以在运行时获取 ACL 保护对象的所有权 |
| 2 | 表示主体需要具有的权限，以在运行时更改 ACL 保护对象的审计 |
| 3 | 表示主体需要具有的权限，以在运行时对 ACL 保护的对象进行任何其他类型的更改（创建、更新和删除） |

可能会有所困惑，我们只指定了一个构造函数参数，尽管列出了三个参数。`AclAuthorizationStrategyImpl`类还可以接受一个`GrantedAuthority`，然后将其用于所有三个参数。如果我们要对所有操作使用相同的`GrantedAuthority`，这非常方便。

`JdbcMutableAclService`接口包含用于在运行时操作 ACL 和 ACE 数据的一系列方法。尽管这些方法本身相对容易理解（`createAcl`、`updateAcl`和`deleteAcl`），但即使是高级 Spring Security 用户，配置和使用`JdbcMutableAclService`的正确方式也往往很难掌握。

让我们修改`CalendarService`以创建新事件的新的 ACL。

# 向新创建的事件添加 ACL

目前，如果用户创建了一个新事件，它将不会在所有事件视图中显示给用户，因为我们使用`<sec:authorize-acl>`标签只显示用户有权访问的事件对象。让我们更新我们的`DefaultCalendarService`接口，以便当用户创建一个新事件时，他们被授予读取该事件的权限，并且它将显示在所有事件页面上。

让我们来看看以下步骤，以将 ACL 添加到新创建的事件中：

1.  第一步是更新我们的构造函数以接受`MutableAclService 和 UserContext`：

``` |
| ```

1.  然后，我们需要更新我们的`createEvent`方法，以也为当前用户创建 ACL。进行以下更改：

``` |

在幕后，标签实现利用了本章早些时候讨论过的相同的`SidRetrievalStrategy`和`ObjectIdentityRetrievalStrategy`接口。因此，访问检查的计算遵循与 ACL 启用的方法安全投票相同的 workflow。正如我们即将看到的，标签实现还将使用相同的`PermissionEvaluator`。

我们已经用一个引用`DefaultMethodSecurityExpressionHandler`的`expressionHandler`元素配置了我们的`GlobalMethodSecurityConfiguration`。`DefaultMethodSecurityExpressionHandler`实现认识我们的`AclPermissionEvaluator`接口，但我们还必须让 Spring Security 的 web 层认识`AclPermissionEvaluator`。如果你仔细思考，这种对称性是合理的，因为保护和 HTTP 请求是保护两种非常不同的资源。幸运的是，Spring Security 的抽象使这一切变得相当简单。

1.  添加一个引用我们已经定义的 ID 为`permissionEvaluator`的 bean 的`DefaultWebSecurityExpressionHandler`处理器：

```

1.  `JdbcMutableAclService`接口使用当前用户作为创建的`MutableAcl`接口的默认所有者。我们选择再次显式设置所有者，以展示如何覆盖此设置。

1.  然后我们添加一个新的 ACE 并保存我们的 ACL。就是这样。

1.  启动应用程序，使用`user1@example.com`/`user1`登录。

1.  访问“所有事件”页面，发现目前没有列出任何事件。然后，创建一个新事件，下次访问“所有事件”页面时它将显示出来。如果你以其他任何用户身份登录，事件将不会在“所有事件”页面上显示。但是，对于其他用户来说，它可能是可见的，因为我们还没有对其他页面应用安全性。再次强调，我们鼓励你自己尝试保护这些页面。

你的代码应该看起来像`chapter12.05-calendar`。

# 典型 ACL 部署的考虑因素

实际上，在真正的商业应用程序中部署 Spring ACL 往往相当复杂。我们总结了 Spring ACL 的覆盖范围，并提出了一些在大多数 Spring ACL 实现场景中出现的问题。

# ACL 的可扩展性与性能建模

对于中小型应用程序，添加 ACL 是相当可管理的，尽管它增加了数据库存储和运行时性能的开销，但影响可能不是很大。然而，根据 ACL 和 ACE 建模的粒度，中型到大型应用程序的数据库行数可能非常惊人，甚至让最有经验的数据库管理员都感到任务繁重。

假设我们要将 ACL 扩展到 JBCP 日历应用程序的扩展版本。假设用户可以管理账户，向事件发布图片，以及从一个事件中添加/删除用户。我们将按照以下方式建模数据：

+   所有用户都有账户。

+   10%的用户能够管理一个事件。平均每个用户能够管理的事件数量将是两个。

+   事件将按客户 secured（只读），但还需要由管理员（读写）访问。

+   所有客户的 10%将被允许发布图片。每个用户的平均发布数量将是 20。

+   发布的图片将按用户 secured（读写），以及管理员。发布的图片对所有其他用户将是只读的。

考虑到我们对 ACL 系统的了解，我们知道数据库表具有以下可扩展性属性：

| **表格** | **与数据相关联** | **可扩展性说明** |
| --- | --- | --- |
| `ACL_CLASS` | 否 | 每种域类需要一行。 |
| `ACL_SID` | 是（用户） | 每个角色（`GrantedAuthority`）需要一行。如果按用户账户个别域对象进行安全保护，则每个用户账户需要一行。 |
| `ACL_OBJECT_IDENTITY` | 是（按类的安全域对象实例） | 每个安全域对象实例需要一行。 |
| `ACL_ENTRY` | 是（按域对象实例个别 ACL 条目） | 每个 ACL 条目需要一行；对于单个域对象可能需要多行。 |

我们可以看到`ACL_CLASS`实际上并没有可扩展性担忧（大多数系统将具有少于 1,000 个域类）。`ACL_SID`表将根据系统中的用户数量线性扩展。这可能不是一个问题，因为其他与用户相关的表也将以这种方式扩展（用户帐户等）。|

关注的两个表是`ACL_OBJECT_IDENTITY`和`ACL_ENTRY`。如果我们为个别客户建模订单所需的估计行数，我们得出以下估计：|

| **Table** | **每个事件的 ACL 数据** | **每个图片帖子的 ACL 数据** |
| --- | --- | --- |
| `ACL_OBJECT_IDENTITY` | 每个事件需要一行。 | 每个帖子需要一行。 |
| `ACL_ENTRY` | 对于所有者（用户`SID`）的读取访问需要三行，其中一行是必需的（对于管理组`SID`的读取访问需要两行，对于事件也是如此） | 四行-一行对于用户组`SID`的读取访问是必需的，所有者需要一行写入访问，管理组`SID`需要两行（与事件一样） |

然后我们可以从前一页的使用假设中获取数据，并以下方式计算 ACL 可扩展性矩阵：|

| **Table/Object** | **扩展因子** | **估计（低）** | **估计（高）** |
| --- | --- | --- | --- |
| `Users` |  | `10,000` | `1,000,000` |
| `Events` | `# Users * 0.1 * 2` | `2,000` | `200,000` |
| `Picture Posts` | `# Users * 0.1 * 20` | `20,000` | `2,000,000` |
| `ACL_SID` | `# Users` | `10,000` | `1,000,000` |
| `ACL_OBJECT_IDENTITY` | `# Events + # Picture Posts` | `220,000` | `2,200,000` |
| `ACL_ENTRY` | `(# Events * 3) + (# Picture Posts * 4)` | `86,000` | `8,600,000` |

从这些基于典型 ACL 实现中可能涉及和受保护的业务对象子集的预测，您可以看到，用于存储 ACL 信息的数据库行的数量可能会与您的实际业务数据线性增长（或更快）。特别是在大型系统规划中，预测您可能使用的 ACL 数据量非常重要。在非常复杂的系统中，与 ACL 存储相关的数百万行数据是很常见的。|

# 不要低估定制开发成本|

在 Spring ACL 安全环境中使用通常需要大量的开发工作，超出我们迄今为止描述的配置步骤。我们示例配置场景有以下限制：|

+   没有提供响应事件或修改权限的操作修改的设施|

+   并非所有的应用程序都在使用权限。例如，我的事件页面和直接导航到事件都不受保护|

应用程序没有有效地使用 ACL 层次结构。如果我们为整个网站实施 ACL 安全，这些限制将对功能产生重大影响。这就是为什么在计划在整个应用程序中实施 Spring ACL 时，你必须仔细审查所有操作领域数据的地方，并确保这些地方正确更新 ACL 和 ACE 规则，以及无效化缓存。通常，方法和数据的保护发生在服务或业务应用程序层，而维护 ACL 和 ACE 所需的生命周期钩子发生在数据访问层。

![](img/85ea63ba-90df-4dea-bf21-fcb02a8a96e2.png)

如果你处理的是一个相对标准的应用程序架构，并且功能得到了适当的隔离和封装，那么这些更改很可能有一个容易识别的中心位置。另一方面，如果你处理的架构已经退化（或者最初就没有设计好），那么在数据操作代码中添加 ACL 功能和支持钩子可能会非常困难。

如之前所暗示的，重要的是要记住 Spring ACL 架构自 Acegi 1.x 时代以来并没有发生显著变化。在那段时间里，许多用户尝试实现它，并在 Spring Security JIRA 存储库([`jira.springframework.org/`](http://jira.springframework.org/))中记录和文档化了几个重要的限制，其中许多在 Spring Security 3 中仍未得到解决。对于这些问题（如果它们适用于你的情况），可能需要进行大量的自定义编码才能绕过。

以下是一些最重要且常见遇到的问题：

+   ACL 基础架构需要一个数值主键。对于使用 GUID 或 UUID 主键的应用程序（这在现代数据库中更常见，因为它们在支持方面更有效率），这可能是一个重大限制。

+   在撰写此内容时，JIRA 问题 SEC-1140 记录了默认 ACL 实现不正确使用位运算符比较权限位掩码的问题。我们在权限部分的章节中已经讨论过这个问题。

+   在配置 Spring ACL 方法和 Spring Security 其他部分之间存在几处不一致。通常，你可能会在类委托或属性未通过 DI 暴露的区域遇到问题，这需要一个耗时且维护成本高昂的重写和覆盖策略。

+   权限位掩码实现为整数，因此有 32 个可能的位。将默认位分配扩展以表示单个对象属性的权限（例如，为读取员工的社会安全号码分配一个位）是比较常见的。复杂的部署可能会使每个领域对象有超过 32 个属性，在这种情况下，唯一的选择就是围绕这个限制重新设计您的领域对象。

根据您特定应用程序的要求，您很可能会遇到额外的问题，尤其是在实现某些类型的自定义时需要更改的类数量方面。

# 我应该使用 Spring Security ACL 吗？

正如整体应用 Spring Security 的细节高度依赖于业务场景一样，Spring ACL 支持的应用也同样如此。实际上，由于 Spring ACL 支持与业务方法和领域对象紧密耦合，这种依赖性在 Spring ACL 支持方面往往更为明显。我们希望本章关于 Spring ACL 的指南能够解释分析 Spring ACL 在应用程序中使用所需的重要高级和低级配置与概念，并帮助您确定并将其功能与现实世界的使用相匹配。

# 总结

在本章中，我们重点讨论了基于 ACL 的安全以及 Spring ACL 模块如何实现这种安全类型的具体细节。

我们回顾了 ACL 的基本概念，以及它们在授权方面可能非常有效的许多原因。此外，您还学习了与 Spring ACL 实现相关的关键概念，包括 ACEs、SIDs 和对象身份。我们检查了支持分层 ACL 系统的数据库模式和逻辑设计。我们配置了所有必需的 Spring beans 以启用 Spring ACL 模块，并增强了一个服务接口以使用注解方法授权。然后，我们将我们数据库中现有的用户和网站本身使用的业务对象与一组示例 ACE 声明和支持数据相连接。我们回顾了 Spring ACL 权限处理的概念。我们扩展了对 Spring Security Thymeleaf 标签库和 SpEL 表达式语言（用于方法安全）的了解，以利用 ACL 检查。我们讨论了可变 ACL 概念，并回顾了在可变 ACL 环境中所需的基本配置和自定义代码。我们开发了一个自定义 ACL 权限并配置应用程序以展示其有效性。我们配置并分析了使用`Ehcache`缓存管理器以减少 Spring ACL 对数据库的影响。我们分析了在复杂业务应用程序中使用 Spring ACL 系统的影响和设计考虑。

这结束了我们关于 Spring Security ACL 的讨论。在下一章中，我们将更深入地了解 Spring Security 是如何工作的。
