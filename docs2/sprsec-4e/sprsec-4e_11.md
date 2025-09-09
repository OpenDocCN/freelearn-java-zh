# 11

# 精细粒度访问控制

在本章中，我们将首先探讨两种实现细粒度授权的方法——可能影响应用程序页面部分的授权。接下来，我们将探讨 Spring Security 通过方法注解和使用基于接口的代理来实现业务层安全的方法。然后，我们将回顾基于注解的安全性的一个有趣功能，该功能允许对数据集合进行基于角色的过滤。最后，我们将探讨基于类的代理与基于接口的代理之间的区别。

在本章中，我们将涵盖以下主题：

+   配置和实验不同的方法，在用户请求的安全上下文中对内容执行页面级授权检查

+   执行配置和代码注解，使调用者预授权成为我们应用程序业务层安全的关键部分

+   实现方法级安全性的几种替代方法，并审查每种类型的优缺点

+   使用方法级注解在集合和数组上实现基于数据的过滤器

+   在我们的 `Spring MVC` 控制器上实现方法级安全以避免配置 `requestMatchers()` 方法和 `<intercept-url>` 元素

本章的代码示例链接在此：[`packt.link/Mxijd`](https://packt.link/Mxijd)。

# 集成 Spring 表达式语言 (SpEL)

Spring Security 利用 `requestMatchers()` 方法：

```java
.requestMatchers("/events/").hasRole("ADMIN")
```

Spring Security 提供了一个 `o.s.s.access.expression.SecurityExpressionRoot` 对象，该对象提供了可用于访问控制决策的方法和对象。例如，可用的方法之一是 `hasRole` 方法，它接受一个字符串。这对应于访问属性（在先前的代码片段中）。实际上，还有许多其他表达式可用，如下表所示：

| **表达式** | **描述** |
| --- | --- |
| `hasRole`(`String role`)`hasAuthority`(`String role`) | 如果当前用户具有指定的权限，则返回 `true`。 |
| `hasAnyRole`(`String... role`) `hasAnyAuthority`(`String... authority`) | 返回 `true` 如果当前用户具有指定的任何权限。 |
| `authentication` | 从 `SecurityContextHolder` 类的 `getContext()` 方法返回的 `SecurityContext` 接口获取当前的 `Authentication` 对象。 |
| `permitAll` | 此请求不需要授权，并作为公共端点。重要的是要明确，在这种情况下，`Authentication` 永远不会从会话中检索。 |
| `denyAll` | 在任何情况下都不允许请求；重要的是要强调，在这种情况下，`Authentication` 永远不会从会话中检索。 |
| `isAnonymous()` | 如果当前主体是匿名（未认证），则返回 `true`。 |
| `isRememberMe()` | 如果当前主体是通过记住我功能进行认证的，则返回`true`。 |
| `isAuthenticated()` | 如果用户不是匿名用户（即，他们已认证），则返回`true`。 |
| `isFullyAuthenticated()` | 如果用户是通过除“记住我”之外的方式认证的，则返回`true`。 |
| `hasPermission`(Object target, Object permission) | 如果用户具有访问指定对象的给定权限，则返回`true`。 |
| `hasPermission`(String targetId, String targetType, Object permission) | 如果用户具有访问给定类型和权限的指定标识符的权限，则返回`true`。 |

表 11.1 – 授权规则摘要

我们在下面的代码片段中提供了一些使用这些 SpEL 表达式的示例。请注意，我们将在本章和下一章中详细介绍：

```java
// allow users with ROLE_ADMIN hasRole('ADMIN')
// allow users that do not have the ROLE_ADMIN
!hasRole('ADMIN')
// allow users that have ROLE_ADMIN or ROLE_ROOT and
// did not use the remember me feature to login
isFullyAuthenticated() and hasAnyRole('ADMIN','ROOT')
// allow if Authentication.getName() equals admin authentication.name == 'admin'
```

开始启动 JBCP 日历应用程序。访问`https://localhost:8443`，使用用户`user1@example.com`和密码`user1`进行登录。您将观察到“我的事件”导航菜单项被显示，同样“所有事件”导航菜单项也被显示。

重要提示

您应该从`chapter11.00-calendar`的代码开始。

## WebSecurityExpressionRoot 类

`o.s.s.web.access.expression.WebSecurityExpressionRoot`类为我们提供了一些额外的属性。这些属性，连同之前提到的标准属性，都可在`requestMatchers()`方法的访问属性和`<sec:authorize>`标签的`JSP/Thymeleaf 访问`属性中使用，正如我们很快将要讨论的：

| **表达式** | **描述** |
| --- | --- |
| `request` | 当前的`HttpServletRequest`方法。 |
| `hasIpAddress`(String... ipAddress) | 如果当前 IP 地址与`ipAddress`值匹配，则返回`true`。这可以是精确的 IP 地址或 IP 地址/网络掩码。 |

表 11.2 – 使用 WebSecurityExpressionRoot 的 hasIpAddress 用法

## MethodSecurityExpressionRoot 类

方法 SpEL 表达式也提供了一些额外的属性，可以通过`o.s.s.access.expression.method.MethodSecurityExpressionRoot`类使用：

| **表达式** | **描述** |
| --- | --- |
| `target` | 指的是这个或当前被保护的对象。 |
| `returnObject` | 指的是由注解方法返回的对象。 |
| `filterObject` | 可以与`@PreFilter`或`@PostFilter`一起用于集合或数组，以仅包括匹配表达式的元素。`filterObject`对象代表集合或数组的循环变量。 |
| `#<methodArg>` | 可以通过在参数名称前加`#`来引用方法的一个参数。例如，名为`id`的方法参数可以使用`#id`来引用。 |

表 11.3 – MethodSecurityExpressionRoot 属性

如果这些表达式的描述看起来有点简短，请不要担心；我们将在本章后面通过一些示例来详细讲解。

我们希望您已经对 Spring Security 的 SpEL 支持能力有了相当的了解。要了解更多关于 SpEL 的信息，请参阅 Spring 参考文档：[`docs.spring.io/spring-framework/reference/core/expressions.xhtml`](https://docs.spring.io/spring-framework/reference/core/expressions.xhtml)。

## 页面级授权

**页面级授权**指的是根据特定用户请求的上下文来提供应用程序功能。与我们在*第二章*“Spring Security 入门”中探讨的粗粒度授权不同，细粒度授权通常指的是页面部分的选择性可用性，而不是完全限制对页面的访问。大多数实际应用都会在细粒度授权规划细节上花费相当多的时间。

Spring Security 为我们提供了以下三种选择性显示功能的方法：

+   **Spring Security JSP** 标签库允许在页面声明本身内放置条件访问声明，使用标准的 JSP 标签库语法。

+   **Thymeleaf Spring Security** 标签库允许在页面声明本身内放置条件访问声明，使用标准的 Thymeleaf 标签库语法。

+   在 MVC 应用程序的控制器层检查用户授权允许控制器做出访问决策并将决策结果绑定到提供给视图的模型数据。这种方法依赖于标准的 JSTL 条件页面渲染和数据绑定，比 Spring Security 标签库稍微复杂一些；然而，它更符合标准 Web 应用程序 MVC 逻辑设计。

在为 Web 应用程序开发细粒度授权模型时，这些方法中的任何一种都是完全有效的。让我们通过一个 JBCP 日历用例来探讨每种方法是如何实现的。

# 使用 Thymeleaf Spring Security 标签库进行条件渲染

在 Thymeleaf Spring Security 标签库中最常用的功能是基于授权规则条件性地渲染页面部分。这是通过使用`<sec:authorize*>`标签来实现的，该标签的功能类似于核心 JSTL 库中的`<if>`标签，即标签体将根据标签属性中提供的条件进行渲染。我们已经看到了 Spring Security 标签库如何被用来限制未登录用户的查看内容的简要演示。

## 基于 URL 访问规则的条件渲染

Spring Security 标签库提供了基于已在安全配置文件中定义的现有 URL 授权规则渲染内容的功能。这是通过使用`authorizeHttpRequests()`方法来实现的。

如果有多个 HTTP 元素，`authorizeHttpRequests()`方法将使用当前匹配的 HTTP 元素的规则。

例如，我们可以确保`All Events`导航菜单项仅在适当的情况下显示，即对于管理员用户——回想一下我们之前定义的访问规则如下：

```java
.requestMatchers("/events/").hasRole("ADMIN")
```

更新`header.xhtml`文件以利用此信息并条件渲染到`All Events`页面链接：

```java
//src/main/resources/templates/fragments/header.xhtml
<!DOCTYPE html>
<html 
      >
...
<li sec:authorize-url="/events/">
    <a id="navEventsLink" th:href="@{/events/}">All Events</a></li>
```

这将确保除非用户有足够的权限访问指定的 URL，否则标签的内容不会显示。可以通过在 URL 之前包含方法属性来进一步限定授权检查，如下所示：

```java
<li sec:authorize-url="/events/">
    <a id="navEventsLink" th:href="@{/events/}">All Events</a></li>
```

使用`authorize-url`属性在代码块上定义授权检查是方便的，因为它将实际授权检查的知识从你的页面中抽象出来，并将其保留在安全配置文件中。

注意，`HTTP`方法应该与你的安全`requestMatchers()`方法中指定的案例匹配，否则它们可能不会如你所期望地匹配。此外，请注意，URL 应始终相对于 Web 应用程序上下文根（正如你的 URL 访问规则一样）。

对于许多用途，使用`<sec>`标签的`authorize-url`属性足以在用户被允许查看时正确显示与链接或操作相关的内容。记住，标签不仅需要围绕一个链接；如果用户没有提交权限，它甚至可以围绕整个表单。

## 使用 SpEL 进行条件渲染

当使用`<sec>`标签与 SpEL 表达式结合时，还有一个更灵活的方法来控制 JSP 内容的显示。让我们回顾一下我们在*第二章*中学到的内容，“Spring Security 入门”。我们可以通过更改`header.xhtml`文件来隐藏`My Events`链接，如下所示：

```java
//src/main/resources/templates/fragments/header.xhtml
<li sec:authorize="isAuthenticated()">
    <a id="navMyEventsLink" th:href="@{/events/my}">My Events</a></li>
```

SpEL 评估是由与`requestMatchers()`方法访问声明规则中使用的表达式背后的相同代码执行的（假设表达式已经配置）。因此，从使用`<sec>`标签构建的表达式中可以访问相同的内置函数和属性。

这两种利用`<sec>`标签的方法都提供了基于安全授权规则的强大、细粒度控制，以显示页面内容。

继续启动 JBCP 日历应用程序。访问`https://localhost:8443`并使用用户`user1@example.com`和密码`user1`登录。你会观察到`admin1@example.com`用户名和密码`admin1`。现在两个链接都是可见的。

重要提示

你应该从`chapter11.01-calendar`中的代码开始。

## 使用控制器逻辑进行条件渲染内容

在本节中，我们将展示如何使用基于 Java 的代码来确定是否应该渲染某些内容。我们可以选择只显示`user`。这将隐藏**欢迎**页面上未登录为管理员的用户的**创建事件**链接。

本章示例代码中的欢迎控制器已被更新，以将一个名为`showCreateLink`的属性填充到模型中，该属性来源于方法名，如下所示：

```java
//src/main/java/com/packtpub/springsecurity/web/controllers/WelcomeControll er.java
@ModelAttribute("showCreateLink")
public boolean showCreateLink(Authentication authentication) {
    // NOTE We could also get the Authentication from SecurityContextHolder.getContext().getAuthentication()
    return authentication != null && authentication.getName().contains("user");
}
```

你可能会注意到 Spring MVC 可以自动为我们获取`Authentication`对象。这是因为 Spring Security 将我们的当前`Authentication`对象映射到`HttpServletRequest.getPrincipal()`方法。由于 Spring MVC 会自动将任何`java.security.Principal`类型的对象解析为`HttpServletRequest.getPrincipal()`的值，因此将`Authentication`作为我们控制器的参数是一种访问当前`Authentication`对象的简单方法。我们也可以通过指定`Principal`类型的参数来解耦代码与 Spring Security。然而，在这个场景中，我们选择了`Authentication`来帮助展示一切是如何连接起来的。

如果我们在另一个不知道如何做到这一点的框架中工作，我们可以使用`SecurityContextHolder`类来获取`Authentication`对象，就像我们在*第三章*中做的那样，*自定义认证*。此外，请注意，如果我们不使用 Spring MVC，我们只需直接设置`HttpServletRequest`属性，而不是在模型中填充它。我们填充在请求上的属性将像使用带有 Spring MVC 的`ModelAndView`对象时一样，对我们的 JSP 可用。

接下来，我们需要在`index.xhtml`文件中使用`HttpServletRequest`属性来确定是否应该显示`创建事件`链接。按照以下方式更新`index.xhtml`：

```java
//src/main/resources/templates/fragments/header.xhtml
<li th:if="${showCreateLink}" class="nav-item"><a class="nav-link" id="navCreateEventLink"
                        th:href="@{/events/form}">Create Event</a>
```

现在，启动应用程序，使用`admin1@example.com`作为用户名和`admin1`作为密码登录，并访问**所有活动**页面。你将不再看到**创建活动**导航菜单项（尽管它仍然会出现在页面上）。

重要提示

你应该从`chapter11.02-calendar`的代码开始。

## `WebInvocationPrivilegeEvaluator`类

有时候，应用程序可能不会使用 JSP 编写，并且需要能够根据 URL 确定访问权限，就像我们使用`<... sec:authorize-url="/events/">`时做的那样。这可以通过使用`o.s.s.web.access.WebInvocationPrivilegeEvaluator`接口来完成，这是支持 JSP 标签库的相同接口。

在下面的代码片段中，我们通过在模型中添加一个名为`showAdminLink`的属性来展示如何使用`WebInvocationPrivilegeEvaluator`。我们能够通过使用`@Autowired`注解来获取`WebInvocationPrivilegeEvaluator`：

```java
//src/main/java/com/packtpub/springsecurity/web/controllers/WelcomeControll er.java
<li th:if="${showAdminLink}" class="nav-item"><a class="nav-link" id="navH2Link"
                        target="_blank"
                        th:href="@{/admin/h2}">H2</a></li>
```

如果你使用的框架不是由 Spring 管理的，`@Autowire` 将无法为你提供 `WebInvocationPrivilegeEvaluator`。相反，你可以使用 Spring 的 `org.springframework.web.context.WebApplicationContextUtils` 接口来获取 `WebInvocationPrivilegeEvaluator` 的实例，如下所示：

```java
ApplicationContext context = WebApplicationContextUtils
       .getRequiredWebApplicationContext(servletContext);
WebInvocationPrivilegeEvaluator privEvaluator = context.getBean(WebInvocationPrivilegeEvaluator.class);
```

要尝试一下，请更新 `index.xhtml` 以使用 `showAdminLink` 请求属性，如下所示：

```java
//src/main/resources/templates/index.xhtml
<li th:if="${showAdminLink}">
    <a class="link-warning" id="h2Link" target="_blank" th:href="@{admin/h2/}">H2
...
</li>
```

重新启动应用程序并查看 `admin1@example.com`/`admin1`，你应该能看到它。

重要提示

你应该从 `chapter11.03-calendar` 的代码开始。

## 最佳的页面内授权配置方式是什么？

在许多情况下，使用标签的 `authorize-url` 属性可以适当地将代码与授权规则的变化隔离开来。以下是在以下场景下应使用标签的 `authorize-url` 属性：

+   该标签阻止了可以通过单个 URL 清晰识别的显示功能。

+   标签的内容可以明确地隔离到单个 URL。

    不幸的是，在典型的应用程序中，你频繁使用标签的 `authorize-url` 属性的可能性相对较低。现实是，应用程序通常比这更复杂，在决定渲染页面的一部分时需要更复杂的逻辑。

使用 Thymeleaf Spring Security 标签库根据其他方法中的安全标准声明渲染页面的部分为受限内容，这很有诱惑力。然而，有多个原因（在许多情况下）这并不是一个好主意，如下所述：

+   标签库不支持超出角色成员资格的复杂条件。例如，如果我们的应用程序在 `UserDetails` 实现中包含了自定义属性、IP 过滤器、地理位置等，标准 `<sec>` 标签将不支持这些。

+   然而，这些可能可以通过自定义标签或使用 SpEL 表达式来支持。即使在这种情况下，页面更有可能直接与业务逻辑相关，而不是通常鼓励的方式。

+   `<sec>` 标签必须在它被使用的每一页上引用。这可能导致旨在通用的规则集在不同物理页面上出现潜在的不一致性。一个良好的面向对象系统设计会建议条件规则评估只位于一个地方，并且从它们应该应用的地方进行逻辑引用。

+   有可能（我们使用我们的公共页眉页面来说明这一点）封装和重用页面的一部分以减少此类问题的发生，但在复杂的应用程序中几乎不可能消除。

+   在编译时验证规则的正确性是没有办法的。虽然编译时常量可以在典型的基于 Java 的面向对象系统中使用，但标签库（在典型使用中）需要硬编码的角色名称，而简单的打字错误可能一段时间内不会被察觉。

+   公平地说，这样的错误可以通过对运行中的应用程序进行全面的函数测试来轻松捕捉，但使用标准的 Java 组件单元测试技术进行测试要容易得多。

+   我们可以看到，尽管基于模板的方法进行条件内容渲染很方便，但也有一些显著的缺点。

所有这些问题都可以通过在控制器中使用代码来解决，这些代码可以用来将数据推送到应用程序视图模型。此外，在代码中执行高级授权判断可以带来复用、编译时检查以及模型、视图和控制器适当逻辑分离的好处。

## 方法级安全

到目前为止，本书的主要重点是保护 JBCP 日历应用程序的 Web 面向部分；然而，在实际规划安全系统时，我们应同样关注保护允许用户访问任何系统最关键部分——其数据的服务方法。

### 为什么我们要分层进行安全防护？

让我们花一分钟时间看看为什么即使我们已经保护了我们的 URL，保护我们的方法仍然很重要。

1.  启动 JBCP 日历应用程序。使用 `user1@example.com` 作为用户名和 `user1` 作为密码进行登录。

1.  访问 `https://localhost:8443/events/`。您将看到自定义的**访问被拒绝**页面。

1.  现在，在浏览器中的 URL 末尾添加 `backdoor`，这样 URL 现在是 `https://localhost:8443/events/backdoor`。您现在将看到一个与**所有活动**页面相同数据的响应。这些数据只应该对管理员可见，但我们通过找到一个未正确配置的 URL 而绕过了它。

1.  我们还可以查看我们未拥有且未被邀请参加的活动详情。将 `backdoor` 替换为 `102`，这样 URL 现在是 `https://localhost:8443/events/102`：您现在将看到一个未列在您的**我的活动**页面上的**休假活动**。这不应该对我们可见，因为我们不是管理员，这不是我们的活动。

如您所见，我们的 URL 规则并不足以完全保护我们的应用程序。这些漏洞甚至不需要利用更复杂的问题，例如容器处理 URL 规范化的差异。简而言之，通常有绕过基于 URL 的安全性的方法。让我们看看在业务层添加安全层如何帮助我们解决新的安全漏洞。

### 保护业务层

Spring Security 有能力为应用程序中任何 Spring 管理的 bean 的调用添加一层授权（或基于授权的数据修剪）。虽然许多开发者关注 Web 层安全，但业务层安全同样重要，因为恶意用户可能能够渗透您的 Web 层安全或通过非 UI 前端（如 Web 服务）访问公开的服务。

让我们检查以下逻辑图，看看为什么我们对应用二级安全层感兴趣：

![图 11.1 – 逻辑应用层](img/B21757_11_1.jpg)

图 11.1 – 逻辑应用层

Spring Security 有以下两种主要技术用于保护方法：

+   `GrantedAuthority`，例如`ROLE_ADMIN`。未能满足声明的约束意味着方法调用将失败。

+   **后授权**：这项技术确保在方法返回后调用者仍然满足声明的约束。这很少使用，但可以为一些复杂且相互关联的业务层方法提供额外的安全层。

预授权和后授权技术为通常在经典面向对象设计中称为预条件和后条件的概念提供了正式支持。预条件和后条件允许开发者通过运行时检查声明，方法执行周围的一定约束必须始终为真。在安全预授权和后授权的情况下，业务层开发者通过将预期的运行时条件作为接口或类 API 声明的一部分来编码，有意识地决定特定方法的网络安全配置。正如你可能想象的那样，这需要大量的前瞻性思考，以避免意外的后果！

### 添加@PreAuthorize 方法注解

我们的第一项设计决策将是通过确保用户在允许访问`getEvents()`方法之前必须以`ADMIN`用户身份登录，来增强业务层的方法安全性。这是通过在服务接口定义中添加到方法的简单注解来实现的，如下所示：

```java
import org.springframework.security.access.prepost.PreAuthorize;
...
public interface CalendarService {
...
    @PreAuthorize("hasRole('ADMIN')")
    List<Event> getEvents();
}
```

这就足以确保调用我们的`getEvents()`方法的任何人都必须是管理员。Spring Security 将使用运行时 AOP 切入点来在方法上执行`BeforeAdvice`，如果安全约束不满足，则抛出`o.s.s.access.AccessDeniedException`异常。

### 指示 Spring Security 使用方法注解

我们还需要对`SecurityConfig.java`进行一次性修改，其中包含我们剩余的 Spring Security 配置。只需将以下注解添加到类声明中：

```java
//src/main/java/com/packtpub/springsecurity/configuration/SecurityC onfig.java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {
```

在下一节中，让我们测试方法安全性验证。

### 验证方法安全性

你不相信这会那么简单吗？使用用户名`user1@example.com`和密码`user1`登录，并尝试访问`https://localhost:8443/events/backdoor`。你现在应该看到**访问被拒绝**页面。

重要注意事项

你应该从`chapter11.04-calendar`的代码开始。

如果你查看*Tomcat 控制台*，你会看到一个非常长的堆栈跟踪，从以下输出开始：

```java
org.springframework.security.access.AccessDeniedException: Access Denied
 at org.springframework.security.authorization.method.AuthorizationManagerBeforeMethodInterceptor.attemptAuthorization
 at org.springframework.security.authorization.method.AuthorizationManagerBeforeMethodInterceptor.invoke
 at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed
 at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed
 at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept
 at com.packtpub.springsecurity.service.DefaultCalendarService$$SpringCGLIB$$0.getEvents
 at com.packtpub.springsecurity.web.controllers.EventsController.events
```

根据`getEvents`方法的调用，我们可以看到用户由于缺少`GrantedAuthority`的`ROLE_ADMIN`权限，被适当地拒绝访问业务方法。如果你使用用户名`admin1@example.com`和密码`admin1`运行相同的操作，你会发现访问将被允许。

难道不是很神奇，仅仅在我们的接口中声明一下，我们就能确保相关方法是安全的？但是 AOP 是如何工作的呢？

## 基于接口的代理

在上一节给出的示例中，Spring Security 使用基于接口的代理来保护我们的`getEvents`方法。让我们看看简化后的伪代码，以了解它是如何工作的：

```java
DefaultCalendarService originalService = context.getBean (CalendarService.class)
CalendarService secureService = new CalendarService() {
//… other methods just delegate to originalService ...
    public List<Event> getEvents() {
       if(!permitted(originalService.getEvents)) {
          throw AccessDeniedException()
       }
       return originalCalendarService.getEvents()
    }
};
```

您可以看到，Spring 创建原始的`CalendarService`就像它通常做的那样。然而，它指示我们的代码使用另一个`CalendarService`的实现，在返回原始方法的结果之前执行安全检查。由于 Spring 使用 Java 的`java.lang.reflect.Proxy` API 动态创建接口的新实现，因此无需了解我们的接口即可创建安全实现。

注意，返回的对象不再是`DefaultCalendarService`的实例，因为它是一个新的`CalendarService`实现，即它是`CalendarService`的匿名实现。这意味着我们必须针对接口编程，才能使用安全实现，否则将发生`ClassCastException`异常。

要了解更多关于 Spring AOP 的信息，请参阅 Spring 参考文档中的[`docs.spring.io/spring-framework/reference/core/aop/proxying.xhtml#aop-understanding-aop-proxies`](https://docs.spring.io/spring-framework/reference/core/aop/proxying.xhtml#aop-understanding-aop-proxies)。

除了`@PreAuthorize`注解之外，还有几种方法可以在方法上声明安全预授权要求。我们可以检查这些不同的方法保护方式，然后评估它们在不同情况下的优缺点。

# JSR-250 兼容的标准化规则

**JSR-250 通用注解**为 Java 平台定义了一系列注解，其中一些与安全相关，旨在在 JSR-250 兼容的运行环境中可移植。Spring 框架在 Spring 2.x 版本中成为 JSR-250 的合规者，包括 Spring Security 框架。

## Gradle 依赖项

根据您决定使用的功能，可能需要几个可选依赖项，例如，为了启用对 JSR 250 `@RolesAllowed`注解的支持。其中许多依赖项已被注释，因为 Spring Boot 已经在启动器父项目中包含了它们。

您会发现我们的`build.gradle`文件已经包含了上述依赖项（间接包含）：

```java
//build.gradle
// Required for JSR-250 based security:
// JSR-250 Annotations
implementation 'jakarta.annotation:jakarta.annotation-api:2.1.1'
```

虽然 JSR-250 注解不如 Spring 原生注解表达性强，但它们的好处是，它们提供的声明可以在实现 Jakarta EE 应用服务器的环境中兼容。根据您应用程序的需求和可移植性的要求，您可能会决定减少特定性的权衡是值得的。

为了实现我们在第一个示例中指定的规则，我们通过以下步骤进行了一些更改：

1.  首先，我们需要更新我们的`SecurityConfig`文件以使用 JSR-250 注解：

    ```java
    //src/main/java/com/packtpub/springsecurity/configuration/ SecurityConfig.java
    @Configuration
    @EnableWebSecurity
    @EnableMethodSecurity(jsr250Enabled = true)
    public class SecurityConfig {
    ```

1.  最后，需要将`@PreAuthorize`注解更改为`@RolesAllowed`注解。正如我们可能预料的，`@RolesAllowed`注解不支持 SpEL 表达式，因此我们需要如下编辑`CalendarService`：

    ```java
    @RolesAllowed("ADMIN")
    List<Event> getEvents();
    ```

1.  重新启动应用程序，以`user1@example.com`/`user1`的身份登录，并尝试访问`https://localhost:8443/events/backdoor`。你应该再次看到**访问被拒绝**页面。

重要提示

你应该从`chapter11.05-calendar`的代码开始。

注意，也可以使用标准的 Java 5 String 数组注解语法提供一个允许的`GrantedAuthority`名称列表：

```java
@RolesAllowed({"ADMIN", "USER"})
List<Event> getEvents();
```

JSR-250 还指定了两个额外的注解，即`@PermitAll`和`@DenyAll`，它们的功能正如你所预期的那样，允许和拒绝针对所涉及方法的全部请求。

重要提示

注意，方法级别的安全注解也可以应用于类级别！如果提供了方法级别的注解，它们将始终覆盖类级别指定的注解。如果你的业务需要为整个类指定安全策略的规范，这可能会很有帮助。

仔细使用此功能，并结合良好的注释和编码标准，以确保开发者非常清楚类及其方法的安全特性。

## 使用 Spring 的@Secured 注解进行方法安全

Spring 本身提供了一个类似于 JSR-250 `@RolesAllowed`注解的简单注解风格。`@Secured`注解在功能和语法上与`@RolesAllowed`相同。唯一的显著区别是它不需要外部依赖，不能被其他框架处理，并且必须通过`@EnableMethodSecurity`注解上的另一个属性显式启用这些注解的处理：

```java
//src/main/java/com/packtpub/springsecurity/configuration/SecurityConfig.java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity(jsr250Enabled = true)
public class SecurityConfig {}
```

由于`@Secured`注解与 JSR 标准`@RolesAllowed`注解功能相同，因此在新代码中实际上没有使用它的真正理由，但你可能会在旧的 Spring 代码中遇到它。

## 结合方法参数的方法安全规则

从逻辑上讲，在约束条件中引用方法参数的规则对于某些类型的操作似乎是合理的。例如，我们可能需要将`findForUser(int userId)`方法限制为满足以下约束：

+   `userId`参数必须等于当前用户的 ID

+   用户必须是管理员（在这种情况下，用户可以查看任何事件）

虽然我们可以很容易地看到如何修改规则以仅允许管理员调用方法，但并不清楚我们如何确定用户是否试图更改自己的密码。

幸运的是，Spring Security 方法注解使用的 SpEL 绑定支持更复杂的表达式，包括包含方法参数的表达式。您还希望确保您已在`SecurityConfig`文件中启用了前注解和后注解，如下所示：

```java
//src/main/java/com/packtpub/springsecurity/configuration/SecurityConfig.java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig { }
//Lastly, we can update our CalendarService interface as follows:
@PreAuthorize("hasRole('ROLE_ADMIN') or principal.id == #userId")
List<Event> findForUser(int userId);
```

您可以在此处看到，我们已将我们在第一个练习中使用的 SpEL 指令与对主体 ID 的检查以及`userId`方法参数（方法参数名称`(#userId`）的检查相结合。这个强大的方法参数绑定功能可用的事实应该激发您的创造力，并允许您使用一组非常精确的逻辑规则来确保方法调用安全。

重要注意事项

由于从*第三章* *自定义认证*中自定义了认证设置，我们的主体目前是`CalendarUser`的一个实例。这意味着主体具有我们`CalendarUser`应用程序上的所有属性。如果我们没有进行此自定义，则只有`UserDetails`对象的属性将可用。

SpEL 变量使用哈希（`#`）前缀进行引用。一个重要的注意事项是，为了在运行时使方法参数名称可用，必须在编译后保留调试符号表信息。以下列出了一些保留调试符号表信息的方法：

1.  如果您使用`javac`编译器，您在构建类时需要包含`-g`标志。

1.  当在 Ant 中使用`<javac>`任务时，添加属性`debug="true"`。

1.  在 Gradle 中，确保在运行主方法或`bootRun`任务时添加`--debug`。

1.  在 Maven 中，确保`maven.compiler.debug=true`属性（默认为`true`）。

1.  请咨询您的编译器、**构建工具**或 IDE 文档，以获取在您的环境中配置此相同设置的协助。

1.  启动您的应用程序，并尝试使用`user1@example.com`作为用户名和`user1`作为密码进行登录。

1.  在`admin1@example.com`)链接上查看一个**访问被拒绝**页面。

1.  再次尝试使用`email=user1@example.com`)来查看它是否工作。

注意，在`admin1@example.com`/`admin1`上显示的用户。由于您登录的用户具有`ROLE_ADMIN`权限，您将能够看到两个页面。

重要注意事项

您应该从`chapter11.06-calendar`的代码开始。

## 结合返回值的方法安全规则

正如我们能够利用方法参数一样，我们也可以利用方法调用的返回值。让我们更新`getEvent`方法以满足以下返回值约束：

+   参与者的 ID 必须是当前用户的 ID

+   拥有者的 ID 必须是当前用户的 ID

+   用户必须是管理员（在这种情况下，用户查看任何事件都是有效的）

将以下代码添加到`CalendarService`接口中：

```java
@PostAuthorize("hasRole('ROLE_ADMIN') or " +
       "principal.id == returnObject.owner.id or " +
       "principal.id == returnObject.attendee.id")
Event getEvent(int eventId);
```

现在，请尝试使用用户名 `user1@example.com` 和密码 `user1` 登录。接下来，请使用**欢迎**页面上的链接访问**假期活动**。你现在应该看到**访问被拒绝**页面。

如果你使用用户名 `user2@example.com` 和密码 `user2` 登录，`user2@example.com` 是**假期活动**的参会者。

重要提示

你应该从 `chapter11.07-calendar` 中的代码开始。

## 使用基于角色的过滤保护方法数据

最后两个依赖于 Spring Security 的注解是 `@PreFilter` 和 `@PostFilter`，它们用于将基于安全性的过滤规则应用于集合或数组（仅使用 `@PostFilter`）。这种功能被称为安全修剪或安全剪枝，它涉及在运行时使用主体的安全凭证来选择性地从一组对象中删除成员。正如你所期望的，这种过滤是在注解声明中使用 SpEL 表达式符号进行的。

我们将使用 JBCP 日历的示例进行操作，因为我们想过滤 `getEvents` 方法，使其只返回该用户允许看到的活动。为了做到这一点，我们移除任何现有的安全注解，并将 `@PostFilter` 注解添加到我们的 `CalendarService` 接口，如下所示：

```java
@PostAuthorize("hasRole('ROLE_ADMIN') or " +
       "principal.id == returnObject.owner.id or " +
       "principal.id == returnObject.attendee.id")
Event getEvent(int eventId);
```

重要提示

你应该从 `chapter11.08-calendar` 中的代码开始。

移除 `requestMatchers()` 方法，限制对 `/events/` URL 的访问，以便我们可以测试我们的注解。启动应用程序并查看 `user1@example.com` 和密码 `user1`。你会观察到只有与我们的用户关联的活动被显示。

当 `filterObject` 作为引用当前事件的循环变量时，Spring Security 将遍历我们服务返回的 `List<Event>` 并将其修改为仅包含与我们的 SpEL 表达式匹配的 `Event` 对象。

通常，`@PostFilter` 方法的行为如下。为了简洁起见，我们将集合称为方法返回值，但请注意，`@PostFilter` 适用于集合或数组方法返回类型。

`filterObject` 对象被重新绑定到 SpEL 上下文中的集合的每个元素。这意味着如果你的方法返回一个包含 100 个元素的集合，SpEL 表达式将针对每个元素进行评估。

SpEL 表达式必须返回一个布尔值。如果表达式评估为 true，对象将保留在集合中，如果表达式评估为 false，对象将被移除。

在大多数情况下，你会发现集合后过滤可以让你避免编写样板代码的复杂性，你可能会编写这些代码。务必理解`@PostFilter`的概念；与`@PreAuthorize`不同，`@PostFilter`指定方法行为而不是先决条件。一些面向对象的纯粹主义者可能会认为`@PostFilter`不适合作为方法注解，这种过滤应该通过方法实现中的代码来处理。

重要提示

注意，从你的方法返回的实际集合将会被修改！在某些情况下，这种行为可能不是期望的，因此你应该确保你的方法返回一个可以被安全修改的集合。这尤其重要，如果返回的集合是 ORM 绑定的，因为后过滤修改可能会意外地持久化到 ORM 数据存储中！

Spring Security 还提供了预处理方法参数集合的功能；现在让我们尝试实现它。

## 使用@PreFilter 预过滤集合

`@PreFilter`注解可以应用于方法，根据当前安全上下文过滤传递给方法的方法参数集合。功能上，一旦它获得对集合的引用，这个注解的行为就与`@PostFilter`注解完全相同，但有以下几个例外：

+   `@PreFilter`注解只支持集合参数，不支持数组参数。

+   `@PreFilter`注解接受一个额外的、可选的`filterTarget`属性，用于特别标识方法参数并在注解的方法有多个参数时对其进行过滤。

+   就像`@PostFilter`一样，请注意，传递给方法的原始集合将被永久修改。这可能不是期望的行为，所以确保调用者知道在方法调用后集合的安全性可能会被裁剪！

假设我们有一个接受事件对象集合的`save`方法，并且我们只想允许保存由当前登录用户拥有的事件。我们可以这样做：

```java
@PreFilter("principal.id == filterObject.owner.id")
void save(Set<Event> events);
```

就像我们的`@PostFilter`方法一样，这个注解会导致 Spring Security 遍历每个事件，使用循环变量`filterObject`。然后它将当前用户的 ID 与事件所有者的 ID 进行比较。如果它们匹配，则保留事件。如果不匹配，则结果被丢弃。

## 比较方法授权类型

以下快速参考图表可以帮助你选择要使用的方法授权检查类型：

| **方法** **授权类型** | **指定为** | **JSR 标准** | **允许** **SpEL 表达式** |
| --- | --- | --- | --- |
| `@PreAuthorize`, `@PostAuthorize` | 注解 | 否 | 是 |
| `@RolesAllowed`, `@PermitAll`, `@DenyAll` | 注解 | 是 | 否 |
| `@Secure` | 注解 | 否 | 否 |
| `protect-pointcut` | XML | 否 | 否 |

表 11.4 – 方法授权类型

大多数使用 Spring Security 的 Java 消费者可能会选择使用 JSR-250 注解以实现最大兼容性和重用其业务类（以及相关约束）在整个 IT 组织中。在需要的情况下，这些基本声明可以被与 Spring Security 实现本身绑定的注解所替代。

如果你在一个不支持注解的环境中使用 Spring Security，不幸的是，你的选择相当有限，主要是方法安全执行。即使在这种情况之下，使用 AOP 也提供了一个相当丰富的环境，我们可以在这个环境中开发基本的声明性安全声明。

## 基于注解的安全性的实际考虑

需要考虑的一件事是，当返回一组现实世界的应用程序时，很可能会进行某种分页。这意味着我们的 `@PreFilter` 和 `@PostFilter` 注解不能作为选择返回哪些对象的唯一手段。相反，我们需要确保我们的查询只选择用户允许访问的数据。

这意味着安全注解变成了冗余检查。然而，重要的是要记住本章开头我们学到的教训；我们想要保护层，以防万一某一层被绕过。

# 摘要

在本章中，我们涵盖了标准 Spring Security 实现中处理授权的大部分剩余领域。我们已经学到了足够多的知识，可以彻底检查 JBCP 日历应用程序，并验证应用程序的所有层都设置了适当的授权检查，以确保恶意用户无法操纵或访问他们没有访问权限的数据。

我们开发了两种微授权技术，即使用 Thymeleaf Spring Security 标签库和 Spring MVC 控制器数据绑定，根据授权或其他安全标准过滤页面内容。我们还探索了在应用程序的业务层中保护业务功能和数据以及支持与代码紧密集成的丰富声明性安全模型的方法。我们还学习了如何保护我们的 Spring MVC 控制器以及接口和类代理对象之间的区别。

到目前为止，我们已经完成了对大多数重要 Spring Security 功能的覆盖，这些功能你很可能在大多数标准、安全的 Web 应用程序开发场景中会遇到。

在下一章中，我们将讨论 Spring Security 的 ACL（域对象模型）模块。这将使我们能够明确声明授权，而不是依赖于现有数据。
