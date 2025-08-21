# 第五章：MicroProfile 健康检查与 JWT 传播

在本章中，我们将介绍健康检查项目及其关注点，它们的构造方式，以及在应用程序中如何使用它们。本章中的代码片段仅供参考。如果您需要一个工作的代码版本，请参考第八章，*一个工作的 Eclipse MicroProfile 代码示例*。

我们将涵盖以下主题：

+   健康检查是什么

+   MicroProfile 健康检查如何暴露健康检查端点和查询该端点的格式

+   如何为您的应用程序编写 MicroProfile 健康检查

+   MicroProfile JWT 传播中令牌的所需格式

+   如何利用 MicroProfile JWT 传播进行安全决策

# 技术要求

要构建和运行本章中的示例，您需要 Maven 3.5+和 Java 8 JDK。本章的代码可以在[`github.com/PacktPublishing/Hands-On-Enterprise-Java-Microservices-with-Eclipse-MicroProfile/tree/master/Chapter04-healthcheck`](https://github.com/PacktPublishing/Hands-On-Enterprise-Java-Microservices-with-Eclipse-MicroProfile/tree/master/Chapter04-healthcheck)和[`github.com/PacktPublishing/Hands-On-Enterprise-Java-Microservices-with-Eclipse-MicroProfile/tree/master/Chapter04-jwtpropagation`](https://github.com/PacktPublishing/Hands-On-Enterprise-Java-Microservices-with-Eclipse-MicroProfile/tree/master/Chapter04-jwtpropagation)找到，分别对应于 MicroProfile 健康检查和 MicroProfile 传播 JWT 部分。

# 理解健康检查以及 MicroProfile 如何处理它们

在云原生架构中，健康检查用于确定计算节点是否存活并准备好执行工作。就绪状态描述了容器启动或滚动更新（即，重新部署）时的状态。在此期间，云平台需要确保没有网络流量路由到该实例，直到它准备好执行工作。

生存性，另一方面，描述运行容器的状态；也就是说，当容器启动或滚动更新（即重新部署）时，它处于就绪状态。在此期间，云平台需要确保没有网络流量路由到该实例，直到它准备好执行工作。

健康检查是与云平台调度程序和应用程序编排框架之间的基本合同。检查程序由应用程序开发者提供，平台使用这些程序来持续确保应用程序或服务的可用性。

微服务健康检查 1.0（MP-HC）支持一个单一的健康检查端点，可以用于活动或就绪检查。微服务健康检查 2.0 计划添加对多个端点的支持，以允许应用程序定义活动和就绪探测器。

微服务健康检查规范详细介绍了两个元素：一个协议以及响应线缆格式部分和一个用于定义响应内容的 Java API。

微服务健康检查（MP-HC）特性的架构被建模为一个由零个或多个逻辑链接在一起的健康检查过程组成的应用程序，这些过程通过`AND`来推导出整体健康检查状态。一个过程代表了一个应用程序定义的检查一个必要条件的操作，它有一个名字、状态，以及可选的关于检查的数据。

# 健康检查协议和线缆格式

微服务健康检查规范定义了支持对逻辑`/health` REST 端点的 HTTP GET 请求的要求，该端点可能返回以下任一代码来表示端点的状态：

+   `200`：它正在运行并且是健康的。

+   `500`：由于未知错误而不健康。

+   `503`：它已经关闭，无法对请求做出响应。

请注意，许多云环境只是将请求返回代码视为成功或失败，所以`500`和`503`代码之间的区别可能无法区分。

`/health`请求的负载必须是一个与下面给出的架构匹配的 JSON 对象（有关 JSON 架构语法的更多信息，请参见[`jsonschema.net/#/`](http://jsonschema.net/#/)）。

下面是...

# 健康检查 Java API

大部分工作由实现微服务健康检查规范的应用程序框架完成。你的任务是通过使用微服务健康检查（MP-HC）API 定义的健康检查过程来决定活动或就绪是如何确定的。

为了实现这一点，你需要通过使用带有`Health`注解的 beans 实现一个或多个`HealthCheck`接口实例来执行健康检查过程。

`HealthCheck`接口如下：

```java
package org.eclipse.microprofile.health;

@FunctionalInterface
public interface HealthCheck {
  HealthCheckResponse call();
}
```

`Health`注解的代码如下：

```java
package org.eclipse.microprofile.health;

import javax.inject.Qualifier;
import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Qualifier
@Documented
@Retention(RetentionPolicy.RUNTIME)
public @interface Health {
}
```

下面的例子展示了一个表示假设磁盘空间检查状态的`HealthCheck`实现。注意检查将当前的空闲空间作为响应数据的一部分。`HealthCheckResponse`类支持一个构建器接口来填充响应对象。

下面是一个假设的磁盘空间`HealthCheck`过程实现：

```java
import javax.enterprise.context.ApplicationScoped;
import org.eclipse.microprofile.health.Health;
import org.eclipse.microprofile.health.HealthCheck;
import org.eclipse.microprofile.health.HealthCheckResponse;

@Health
@ApplicationScoped
public class CheckDiskspace implements HealthCheck {
  @Override
  public HealthCheckResponse call() {
      return HealthCheckResponse.named("diskspace")
              .withData("free", "780mb")
              .up()
              .build();
  }
}
```

在这个例子中，我们创建了一个名为`diskspace`的健康响应，其状态为`up`，并带有名为`free`的自定义数据，其字符串值为`780mb`。

下面的例子展示了一个表示某些服务端点的健康检查示例。

下面展示了一个假设的服务`HealthCheck`过程实现：

```java
package io.packt.hc.rest;
//ServiceCheck example

import javax.enterprise.context.ApplicationScoped;
import org.eclipse.microprofile.health.Health;
import org.eclipse.microprofile.health.HealthCheck;
import org.eclipse.microprofile.health.HealthCheckResponse;

@Health
@ApplicationScoped
public class ServiceCheck implements HealthCheck {
 public HealthCheckResponse call() {
 return HealthCheckResponse.named("service-check")
 .withData("port", 12345)
 .withData("isSecure", true)
 .withData("hostname", "service.jboss.com")
 .up()
 .build();
 }
}
```

在这个例子中，我们创建了一个名为`service-check`的健康响应，其状态为`up`，并包括了以下附加数据：

+   一个整数值为`12345`的`port`项

+   一个值为`true`的布尔`isSecure`项

+   一个值为`service.jboss.com`的字符串`hostname`项

由 CDI 管理的健康检查由应用程序运行时自动发现和注册。运行时自动暴露一个 HTTP 端点，`/health`，供云平台用来探测您的应用程序状态。您可以通过构建`Chapter04-healthcheck`应用程序并运行它来测试这一点。您将看到以下输出：

```java
Scotts-iMacPro:hc starksm$ mvn package
[INFO] Scanning for projects…
...
Resolving 144 out of 420 artifacts

[INFO] Repackaging .war: /Users/starksm/Dev/JBoss/Microprofile/PacktBook/Chapter04-metricsandhc/hc/target/health-check.war

[INFO] Repackaged .war: /Users/starksm/Dev/JBoss/Microprofile/PacktBook/Chapter04-metricsandhc/hc/target/health-check.war

[INFO] -----------------------------------------------------------------------

[INFO] BUILD SUCCESS

[INFO] -----------------------------------------------------------------------

[INFO] Total time:  7.660 s

[INFO] Finished at: 2019-04-16T21:55:14-07:00

[INFO] -----------------------------------------------------------------------

Scotts-iMacPro:hc starksm$ java -jar target/health-check-thorntail.jar

2019-04-16 21:57:03,305 INFO  [org.wildfly.swarm] (main) THORN0013: Installed fraction: MicroProfile Fault Tolerance - STABLE          io.thorntail:microprofile-fault-tolerance:2.4.0.Final

…

2019-04-16 21:57:07,449 INFO  [org.jboss.as.server] (main) WFLYSRV0010: Deployed "health-check.war" (runtime-name : "health-check.war")

2019-04-16 21:57:07,453 INFO  [org.wildfly.swarm] (main) THORN99999: Thorntail is Ready
```

一旦服务器启动，通过查询健康端点来测试健康检查：

```java
Scotts-iMacPro:Microprofile starksm$ curl -s http://localhost:8080/health | jq
{
 "outcome": "UP",
 "checks": [
   {
     "name": "service-check",
     "state": "UP",
     "data": {
       "hostname": "service.jboss.com",
       "port": 12345,
       "isSecure": true
     }
   },
   {
     "name": "diskspace",
     "state": "UP",
     "data": {
       "free": "780mb"
     }
   }
 ]
}
```

这显示了整体健康状况为`UP`。整体状态是应用程序中找到的所有健康检查程序的逻辑`OR`。在这个例子中，它是我们所看到的两个健康检查程序`diskspace`和`service-check`的`AND`。

# 与云平台的集成

大多数云平台都支持基于 TCP 和 HTTP 的检查。为了将健康检查与您选择的云平台集成，您需要配置云部署，使其指向托管应用程序的节点上的 HTTP 入口点，`/health`。

云平台将调用 HTTP 入口点的`GET`查询；所有注册的检查都将执行，个别检查的总和决定了整体结果。

通常，响应载荷被云平台忽略，它只查看 HTTP 状态码来确定应用程序的存活或就绪状态。成功的成果，`UP`，将被映射到`200`，而`DOWN`将被映射到`503`。

# 人类操作者

JSON 响应载荷的主要用例是提供一种让操作者调查应用程序状态的方式。为了支持这一点，健康检查允许将附加数据附加到健康检查响应中，正如我们在`CheckDiskspace`和`ServiceCheck`示例中所看到的那样。考虑以下片段：

```java
[...]
return HealthCheckResponse
           .named("memory-check")
           .withData("free-heap", "64mb")
           .up()
           .build();
[...]
```

在这里，提供了关于`free-heap`的附加信息，并将成为响应载荷的一部分，正如这个响应片段所示。显示`memory-check`程序内容的 JSON 响应片段如下：

```java
{
...
   "checks": [
       {
           "name": "memory-check",
           "state": "UP",
           "data": {
               "free-heap": "64mb"
           }
       }
   ],
   "outcome": "UP"
}
```

在这里，我们看到`memory-check`程序以其`UP`状态和字符串类型的附加`free-heap`数据项，值为`64mb`。

**Eclipse 资源/GitHub 中 MP-Health 的坐标**：

MP-Health 项目源代码可以在[`github.com/eclipse/microprofile-health`](https://github.com/eclipse/microprofile-health)找到。

# 健康检查响应消息的变化

MicroProfile Health Check 3.0 对健康检查 JSON 响应的消息格式进行了更改。具体来说，字段的成果和状态已经被字段状态所取代。

此外，在健康检查 3.0 版本中，`@Health`限定符已被弃用，而`@Liveness`和`@Readiness`限定符已被引入。对于这两个限定符，还引入了`/health/live`和`/health/ready`端点，分别调用所有存活性和就绪性程序。最后，为了向后兼容，`/health`端点现在会调用所有具有`@Health`、`@Liveness`或`@Readiness`限定符的程序。

是时候讨论 JWT 传播了。

# 在 MicroProfile 中使用 JSON Web Token 传播

一个**JSON Web Token**（**JWT**）是许多不同的基于 web 的安全协议中用于携带安全信息的一种常见格式。然而，JWT 的确切内容以及与已签名 JWT 一起使用的安全算法缺乏标准化。**微服务 JWT**（**MP-JWT**）传播项目规范审视了基于**OpenID Connect**（**OIDC**）的 JWT（[`openid.net/connect/`](http://openid.net/connect/)）规范，并在这些规范的基础上定义了一组需求，以促进基于 MicroProfile 的微服务中 JWT 的互操作性，同时还提供了从 JWT 中访问信息的 API。

有关 OIDC 和 JWT 如何工作的描述，包括应用程序/微服务如何拦截承载令牌，请参阅[`openid.net/connect/`](http://openid.net/connect/)上的*基本客户端实现指南*。

在本节中，您将了解以下内容：

+   为了互操作性，所需的 OIDC 和 JWT 规范中的声明和签名算法

+   使用 JWT 进行**基于角色的访问控制**（**RBAC**）的微服务端点

+   如何使用 MP-JWT API 来访问 JWT 及其声明值

# 互操作性的建议

MP-JWT 作为令牌格式的最大效用取决于身份提供商和服务提供商之间的协议。这意味着负责发行令牌的身份提供商应该能够以服务提供商可以理解的方式发行 MP-JWT 格式的令牌，以便服务提供商可以检查令牌并获取有关主题的信息。MP-JWT 的主要目标如下：

+   它应该可以用作认证令牌。

+   它应该可以用作包含通过组声明间接授予的应用级角色的授权令牌。

+   它支持 IANA JWT 分配（[`www.iana.org/assignments/jwt/jwt.xhtml`](https://www.iana.org/assignments/jwt/jwt.xhtml)）中描述的额外标准声明，以及非标准...

# 必需的 MP-JWT 声明

需要提供支持的 MP-JWT 声明集合包括以下内容：

+   `typ`：此头部参数标识令牌类型，必须是`JWT`。

+   `alg`：此头部算法用于签署 JWT，必须是`RS256`。

+   `kid`：这个头部参数提供了一个提示，关于用哪个公钥签署了 JWT。

+   `iss`：这是令牌的发行者和签名者。

+   `sub`：这标识了 JWT 的主题。

+   `exp`：这标识了 JWT 在或之后过期的时刻，此时 JWT 不得被处理。

+   `iat`：这标识了 JWT 的发行时间，可以用来确定 JWT 的年龄。

+   `jti`：这为 JWT 提供了一个唯一标识符。

+   `upn`：这是 MP-JWT 自定义声明，是指定用户主体名称的首选方式。

+   `groups`：这是 MP-JWT 自定义声明，用于分配给 JWT 主体的组或角色名称列表。

`NumericDate`用于`exp`、`iat`和其他日期相关声明是一个表示从`1970-01-01T00:00:00Z` UTC 直到指定 UTC 日期/时间的 JSON 数值值，忽略闰秒。此外，有关标准声明的更多详细信息可以在 MP-JWT 规范([`github.com/eclipse/microprofile-jwt-auth/releases/tag/1.1.1`](https://github.com/eclipse/microprofile-jwt-auth/releases/tag/1.1.1))和 JSON Web Token RFC([`tools.ietf.org/html/rfc7519`](https://tools.ietf.org/html/rfc7519))中找到。

一个基本的 MP-JWT 的 JSON 示例将是与 MP-JWT 兼容的 JWT 的示例头和载荷，如下所示：

```java
{
    "typ": "JWT",
    "alg": "RS256",
    "kid": "abc-1234567890"
}
{
    "iss": "https://server.example.com",
    "jti": "a-123",
    "exp": 1311281970,
    "iat": 1311280970,
    "sub": "24400320",
    "upn": "jdoe@server.example.com",
    "groups": ["red-group", "green-group", "admin-group", "admin"],
}
{
*** base64 signature not shown ***
}
```

此示例显示了具有`typ=JWT`、`alg=RS256`和`kid=abc-1234567890`的头部。正文包括`iss`、`jti`、`exp`、`iat`、`sub`、`upn`和`groups`声明。

# MP-JWT API 的高级描述

MP-JWT 项目在`org.eclipse.microprofile.jwt`包命名空间下引入了以下 API 接口和类：

+   `JsonWebToken`：这是`java.security.Principal`接口的一个扩展，通过 get 风格的访问器提供所需声明的集合，同时提供对 JWT 中任何声明的通用访问。

+   `Claims`：这是一个封装了所有标准 JWT 相关声明以及描述和返回自`JsonWebToken#getClaim(String)`方法的声明所需 Java 类型的枚举实用类。

+   `Claim`：这是一个用于指示`ClaimValue`注入点的限定注解。

+   `ClaimValue<T>`：这是`java.security.Principal`接口的一个扩展...

# 使用 MP-JWT 的示例代码

MP-JWT API 的基本用法是注入`JsonWebToken`、其`ClaimValue`或两者。在本节中，我们将展示典型用法的代码片段。本书本节代码可在[`github.com/PacktPublishing/Hands-On-Enterprise-Java-Microservices-with-Eclipse-MicroProfile/tree/master/Chapter04-jwtpropagation`](https://github.com/PacktPublishing/Hands-On-Enterprise-Java-Microservices-with-Eclipse-MicroProfile/tree/master/Chapter04-jwtpropagation)找到。

# 注入 JsonWebToken 信息

以下代码示例说明了如何访问传入的 MP-JWT 令牌作为`JsonWebToken`、原始 JWT 令牌字符串、`upn`声明以及与 JAX-RS`SecurityContext`的集成：

```java
package io.pckt.jwt.rest;import javax.annotation.security.DenyAll;import javax.annotation.security.PermitAll;import javax.annotation.security.RolesAllowed;import javax.inject.Inject;import javax.ws.rs.GET;import javax.ws.rs.Path;import javax.ws.rs.Produces;import javax.ws.rs.core.Context;import javax.ws.rs.core.MediaType;import javax.ws.rs.core.SecurityContext;import org.eclipse.microprofile.jwt.Claim;import org.eclipse.microprofile.jwt.Claims;import org.eclipse.microprofile.jwt.JsonWebToken;@Path("/jwt")@DenyAll //1public class ...
```

# 向 JWT 断言值注入

本节中的代码片段说明了 JWT 断言值的注入。我们可以使用几种不同的格式作为注入值。标准断言支持在`Claim#getType`字段和`JsonValue`子类型中定义的对象子类型。自定义断言类型只支持`JsonValue`子类型的注入。

以下代码示例说明了标准`groups`和`iss`断言的注入，以及`customString`、`customInteger`、`customDouble`和`customObject`自定义断言的注入：

```java
package io.pckt.jwt.rest;

import java.util.Set;
import javax.annotation.security.DenyAll;
import javax.annotation.security.RolesAllowed;
import javax.inject.Inject;
import javax.json.JsonArray;
import javax.json.JsonNumber;
import javax.json.JsonObject;
import javax.json.JsonString;
import javax.ws.rs.GET;
import javax.ws.rs.Path;

import org.eclipse.microprofile.jwt.Claim;
import org.eclipse.microprofile.jwt.Claims;

@Path("/jwt")
@DenyAll
public class InjectionExampleEndpoint {
    @Inject
    @Claim(standard = Claims.groups)
    Set<String> rolesSet; // 1
    @Inject
    @Claim(standard = Claims.iss)
    String issuer; // 2

    @Inject
    @Claim(standard = Claims.groups)
    JsonArray rolesAsJson; // 3
    @Inject
    @Claim(standard = Claims.iss)
    JsonString issuerAsJson; // 4
    // Custom claims as JsonValue types
    @Inject
    @Claim("customString")
    JsonString customString; // 5
    @Inject
    @Claim("customInteger")
    JsonNumber customInteger; // 6
    @Inject
    @Claim("customDouble")
    JsonNumber customDouble; // 7
    @Inject
    @Claim("customObject")
    JsonObject customObject; // 8

    @GET
    @Path("/printClaims")
    @RolesAllowed("Tester")
    public String printClaims() {
        return String.format("rolesSet=%s\n");
    }
}
```

八个注释注入如下：

1.  将标准`groups`断言作为其默认`Set<String>`类型注入

1.  将标准`iss`断言作为其默认字符串类型注入

1.  将标准`groups`断言作为其默认`JsonArray`类型注入

1.  将标准`iss`断言作为其默认`JsonString`类型注入

1.  向`JsonString`类型的`customString`断言中注入非标准自定义字符串

1.  向`JsonNumber`类型的非标准`customInteger`断言中注入

1.  向`JsonNumber`类型的非标准`customDouble`断言中注入

1.  向`JsonString`类型的`customObject`断言中注入非标准自定义对象

# 配置 JWT 的认证

为了接受 JWT 作为应进行认证并因此受信任的身份，我们需要配置 MP-JWT 功能以验证谁签署了 JWT 以及谁发布了 JWT。这是通过 MP-Config 属性完成的：

+   `mp.jwt.verify.publickey`：这提供了 MP-JWT 签名者的公钥材料，通常以 PKCS8 PEM 格式嵌入。

+   `mp.jwt.verify.issuer`：这指定了 JWT 中`iss`断言的预期值。

本书的一个`microprofile-configuration.properties`文件示例如下：

```java
# MP-JWT Configmp.jwt.verify.publickey=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAlivFI8qB4D0y2jy0CfEqFyy46R0o7S8TKpsx5xbHKoU1VWg6QkQm+ntyIv1p4kE1sPEQO73+HY8+Bzs75XwRTYL1BmR1w8J5hmjVWjc6R2BTBGAYRPFRhor3kpM6ni2SPmNNhurEAHw7TaqszP5eUF/F9+KEBWkwVta+PZ37bwqSE4sCb1soZFrVz/UT/LF4tYpuVYt3YbqToZ3pZOZ9AX2o1GCG3xwOjkc4x0W7ezbQZdC9iftPxVHR8irOijJRRjcPDtA6vPKpzLl6CyYnsIYPd99ltwxTHjr3npfv/3Lw50bAkbT4HeLFxTx4flEoZLKO/g0bAoV2uqBhkA9xnQIDAQAB ...
```

# 运行示例

我们查看的示例可以部署到 Thorntail，并通过针对端点的命令行查询来验证预期行为。由于需要对带有安全约束的端点进行认证，因此我们需要一种生成将被 Thorntail 服务器接受的有效 JWT 的方法。

本章代码提供了一个`io.packt.jwt.test.GenerateToken`工具，该工具将由配置在 Thorntail 服务器上的密钥签发的 JWT。JWT 中包含的断言由本章项目的`src/test/resources/JwtClaims.json`文档定义。您使用`mvn exec:java`命令运行该工具，如下所示：

```java
Scotts-iMacPro:jwtprop starksm$ mvn exec:java -Dexec.mainClass=io.packt.jwt.test.GenerateToken -Dexec.classpathScope=test
[INFO] Scanning for projects...
[INFO]
[INFO] ----------------< io.microprofile.jwt:jwt-propagation >-----------------
[INFO] Building JWT Propagation 1.0-SNAPSHOT
[INFO] --------------------------------[ war ]---------------------------------
[INFO]
[INFO] --- exec-maven-plugin:1.6.0:java (default-cli) @ jwt-propagation ---
Setting exp: 1555684338 / Fri Apr 19 07:32:18 PDT 2019
 Added claim: sub, value: 24400320
 Added claim: customIntegerArray, value: [0,1,2,3]
 Added claim: customDoubleArray, value: [0.1,1.1,2.2,3.3,4.4]
 Added claim: iss, value: http://io.packt.jwt
 Added claim: groups, value: 
    ["Echoer","Tester","User","group1","group2"]
 Added claim: preferred_username, value: jdoe
 Added claim: customStringArray, value: ["value0","value1","value2"]
 Added claim: aud, value: [s6BhdRkqt3]
 Added claim: upn, value: jdoe@example.com
 Added claim: customInteger, value: 123456789
 Added claim: auth_time, value: 1555683738
 Added claim: customObject, value: {"my-service":{"roles":["role-in-my-
    service"],"groups":["group1","group2"]},"service-B":{"roles":["role-in-
    B"]},"service-C":{"groups":["groupC","web-tier"]},"scale":0.625}
 Added claim: exp, value: Fri Apr 19 07:32:18 PDT 2019
 Added claim: customDouble, value: 3.141592653589793
 Added claim: iat, value: Fri Apr 19 07:22:18 PDT 2019
 Added claim: jti, value: a-123
 Added claim: customString, value: customStringValue
eyJraWQiOiJcL3ByaXZhdGUta2V5LnBlbSIsInR5cCI6IkpXVCIsImFsZyI6IlJTMjU2In0.eyJzdWIiOiIyNDQwMDMyMCIsImN1c3RvbUludGVnZXJBcnJheSI6WzAsMSwyLDNdLCJjdXN0b21Eb3VibGVBcnJheSI6WzAuMSwxLjEsMi4yLDMuMyw0LjRdLCJpc3MiOiJodHRwOlwvXC9pby5wYWNrdC5qd3QiLCJncm91cHMiOlsiRWNob2VyIiwiVGVzdGVyIiwiVXNlciIsImdyb3VwMSIsImdyb3VwMiJdLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJqZG9lIiwiY3VzdG9tU3RyaW5nQXJyYXkiOlsidmFsdWUwIiwidmFsdWUxIiwidmFsdWUyIl0sImF1ZCI6InM2QmhkUmtxdDMiLCJ1cG4iOiJqZG9lQGV4YW1wbGUuY29tIiwiY3VzdG9tSW50ZWdlciI6MTIzNDU2Nzg5LCJhdXRoX3RpbWUiOjE1NTU2ODM3MzgsImN1c3RvbU9iamVjdCI6eyJteS1zZXJ2aWNlIjp7InJvbGVzIjpbInJvbGUtaW4tbXktc2VydmljZSJdLCJncm91cHMiOlsiZ3JvdXAxIiwiZ3JvdXAyIl19LCJzZXJ2aWNlLUIiOnsicm9sZXMiOlsicm9sZS1pbi1CIl19LCJzZXJ2aWNlLUMiOnsiZ3JvdXBzIjpbImdyb3VwQyIsIndlYi10aWVyIl19LCJzY2FsZSI6MC42MjV9LCJleHAiOjE1NTU2ODQzMzgsImN1c3RvbURvdWJsZSI6My4xNDE1OTI2NTM1ODk3OTMsImlhdCI6MTU1NTY4MzczOCwianRpIjoiYS0xMjMiLCJjdXN0b21TdHJpbmciOiJjdXN0b21TdHJpbmdWYWx1ZSJ9.bF7CnutcQnA2gTlCRNOp4QMmWTWhwP86cSiPCSxWr8N36FG79YC9Lx0Ugr-Ioo2Zw35z0Z0xEwjAQdKkkKYU9_1GsXiJgfYqzWS-XxEtwhiinD0hUK2qiBcEHcY-ETx-bsJud8_mSlrzEvrJEeX58Xy1Om1FxnjuiqmfBJxNaotxECScDcDMMH-DeA1Z-nrJ3-0sdKNW6QxOxoR_RNrpci1F9y4pg-eYOd8zE4tN_QbT3KkdMm91xPhv7QkKm71pnHxC0H4SmQJVEAX6bxdD5lAzlNYrEMAJyyEgKuJeHTxH8qzM-0FQHzrG3Yhnxax2x3Xd-6JtEbU-_E_3HRxvvw
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.339 s
[INFO] Finished at: 2019-04-19T07:22:19-07:00
[INFO] ------------------------------------------------------------------------
```

该工具输出了添加的断言，然后打印出 Base64 编码的 JWT。您将使用这个 JWT 作为`curl`命令行中访问服务器端点的`Authorization: Bearer …`头部的值。

为了启动带有示例端点的 Thorntail 服务器，请进入`Chapter04-jwtpropagation`项目目录，然后运行`mvn`以构建可执行的 JAR：

```java
Scotts-iMacPro:jwtprop starksm$ mvn package
[INFO] Scanning for projects...
[INFO]
[INFO] ----------------< io.microprofile.jwt:jwt-propagation >-----------------
[INFO] Building JWT Propagation 1.0-SNAPSHOT
...
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  8.457 s
[INFO] Finished at: 2019-04-19T08:25:09-07:00
[INFO] ------------------------------------------------------------------------
```

生成的可执行 JAR 位于`target/jwt-propagation-thorntail.jar`。你使用本章的示例部署和`java -jar …`命令启动 Thorntail 服务器：

```java
Scotts-iMacPro:jwtprop starksm$ java -jar target/jwt-propagation-thorntail.jar
2019-04-19 08:27:33,425 INFO  [org.wildfly.swarm] (main) THORN0013: Installed fraction: MicroProfile Fault Tolerance - STABLE          io.thorntail:microprofile-fault-tolerance:2.4.0.Final
2019-04-19 08:27:33,493 INFO  [org.wildfly.swarm] (main) THORN0013: Installed fraction:          Bean Validation - STABLE io.thorntail:bean-validation:2.4.0.Final
2019-04-19 08:27:33,493 INFO  [org.wildfly.swarm] (main) THORN0013: Installed fraction:      MicroProfile Config - STABLE io.thorntail:microprofile-config:2.4.0.Final
2019-04-19 08:27:33,493 INFO  [org.wildfly.swarm] (main) THORN0013: Installed fraction:             Transactions - STABLE io.thorntail:transactions:2.4.0.Final
2019-04-19 08:27:33,494 INFO  [org.wildfly.swarm] (main) THORN0013: Installed fraction:        CDI Configuration - STABLE io.thorntail:cdi-config:2.4.0.Final
2019-04-19 08:27:33,494 INFO  [org.wildfly.swarm] (main) THORN0013: Installed fraction: MicroProfile JWT RBAC Auth - STABLE          io.thorntail:microprofile-jwt:2.4.0.Final
…
2019-04-19 08:27:37,708 INFO  [org.jboss.as.server] (main) WFLYSRV0010: Deployed "jwt-propagation.war" (runtime-name : "jwt-propagation.war")
2019-04-19 08:27:37,713 INFO  [org.wildfly.swarm] (main) THORN99999: Thorntail is Ready
```

在此阶段，我们可以查询服务器端点。有一个端点是我们定义的，不需要任何认证。这是`io.pckt.jwt.rest.SecureEndpoint`类的`jwt/openHello`端点。运行以下命令来验证你的 Thorntail 服务器是否如预期运行：

```java
Scotts-iMacPro:jwtprop starksm$ curl http://localhost:8080/jwt/openHello; echo
Hello[open] user=anonymous, upn=no-upn
```

接下来，尝试受保护的端点。它应该会因为未提供任何授权信息而失败，返回 401 未授权错误：

```java
Scotts-iMacPro:jwtprop starksm$ curl http://localhost:8080/jwt/secureHello; echo
Not authorized
```

现在，我们需要生成一个新的 JWT，并将其与 curl 命令一起在`Authorization`头中传递，让我们试一试。我们将使用 mvn 命令生成的 JWT 在 JWT 环境变量中保存，以简化 curl 命令行：

```java
Scotts-iMacPro:jwtprop starksm$ mvn exec:java -Dexec.mainClass=io.packt.jwt.test.GenerateToken -Dexec.classpathScope=test
[INFO] Scanning for projects...
[INFO]
[INFO] ----------------< io.microprofile.jwt:jwt-propagation >-----------------
[INFO] Building JWT Propagation 1.0-SNAPSHOT
[INFO] --------------------------------[ war ]---------------------------------
[INFO]
[INFO] --- exec-maven-plugin:1.6.0:java (default-cli) @ jwt-propagation ---
Setting exp: 1555688712 / Fri Apr 19 08:45:12 PDT 2019
 Added claim: sub, value: 24400320
 Added claim: customIntegerArray, value: [0,1,2,3]
 Added claim: customDoubleArray, value: [0.1,1.1,2.2,3.3,4.4]
 Added claim: iss, value: http://io.packt.jwt
 Added claim: groups, value: 
    ["Echoer","Tester","User","group1","group2"]
 Added claim: preferred_username, value: jdoe
 Added claim: customStringArray, value: ["value0","value1","value2"]
 Added claim: aud, value: [s6BhdRkqt3]
 Added claim: upn, value: jdoe@example.com
 Added claim: customInteger, value: 123456789
 Added claim: auth_time, value: 1555688112
 Added claim: customObject, value: {"my-service":{"roles":["role-in-my-
    service"],"groups":["group1","group2"]},"service-B":{"roles":["role-in-
    B"]},"service-C":{"groups":["groupC","web-tier"]},"scale":0.625}
 Added claim: exp, value: Fri Apr 19 08:45:12 PDT 2019
 Added claim: customDouble, value: 3.141592653589793
 Added claim: iat, value: Fri Apr 19 08:35:12 PDT 2019
 Added claim: jti, value: a-123
 Added claim: customString, value: customStringValue
eyJraWQiOiJ...
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.352 s
[INFO] Finished at: 2019-04-19T08:35:12-07:00
[INFO] ------------------------------------------------------------------------
Scotts-iMacPro:jwtprop starksm$ JWT="eyJraWQiOi..."
Scotts-iMacPro:jwtprop starksm$ curl -H "Authorization: Bearer $JWT" http://localhost:8080/jwt/secureHello; echo
Hello[secure] user=jdoe@example.com, upn=jdoe@example.com, scheme=MP-JWT, isUserRole=true
```

在前面的代码片段中，对于 Windows 用户，请为 Windows 安装一个与 bash 兼容的壳程序；否则，由于`echo`命令错误，你将遇到错误。

这次，查询成功，我们看到用户名`upn`声明值、方案和`isUserInRole("User")`检查都如预期一样。

现在，尝试访问`/jwt/printClaims`端点，该端点说明了标准和非标准声明作为不同类型的注入：

```java
Scotts-iMacPro:jwtprop starksm$ curl -H "Authorization: Bearer $JWT" http://localhost:8080/jwt/printClaims
+++ Standard claims as primitive types
rolesSet=[Echoer, Tester, User, group2, group1]
issuer=http://io.packt.jwt
+++ Standard claims as JSON types
rolesAsJson=["Echoer","Tester","User","group2","group1"]
issuerAsJson="http://io.packt.jwt"
+++ Custom claim JSON types
customString="customStringValue"
customInteger=123456789
customDouble=3.141592653589793
customObject={"my-service":{"roles":["role-in-my-service"],"groups":["group1","group2"]},"service-B":{"roles":["role-in-B"]},"service-C":{"groups":["groupC","web-tier"]},"scale":0.625}
```

请注意，如果你在使用了一段时间后开始遇到`未授权错误`，问题是因为 JWT 已经过期。你需要生成一个新的令牌，或者生成一个有效期更长的令牌。你可以通过向`GenerateToken`工具传入以秒为单位的过期时间来做到这一点。例如，为了生成一个可以使用一小时的令牌，执行以下操作：

```java
Scotts-iMacPro:jwtprop starksm$ mvn exec:java -Dexec.mainClass=io.packt.jwt.test.GenerateToken -Dexec.classpathScope=test -Dexec.args="3600"
[INFO] Scanning for projects...
[INFO]
[INFO] ----------------< io.microprofile.jwt:jwt-propagation >-----------------
[INFO] Building JWT Propagation 1.0-SNAPSHOT
[INFO] --------------------------------[ war ]---------------------------------
[INFO]
[INFO] --- exec-maven-plugin:1.6.0:java (default-cli) @ jwt-propagation ---
Setting exp: 1555692188 / Fri Apr 19 09:43:08 PDT 2019
 Added claim: sub, value: 24400320
 Added claim: customIntegerArray, value: [0,1,2,3]
 Added claim: customDoubleArray, value: [0.1,1.1,2.2,3.3,4.4]
 Added claim: iss, value: http://io.packt.jwt
 Added claim: groups, value: ["Echoer","Tester","User","group1","group2"]
 Added claim: preferred_username, value: jdoe
 Added claim: customStringArray, value: ["value0","value1","value2"]
 Added claim: aud, value: [s6BhdRkqt3]
 Added claim: upn, value: jdoe@example.com
 Added claim: customInteger, value: 123456789
 Added claim: auth_time, value: 1555688588
 Added claim: customObject, value: {"my-service":{"roles":["role-in-my-service"],"groups":["group1","group2"]},"service-B":{"roles":["role-in-B"]},"service-C":{"groups":["groupC","web-tier"]},"scale":0.625}
 Added claim: exp, value: Fri Apr 19 09:43:08 PDT 2019
 Added claim: customDouble, value: 3.141592653589793
 Added claim: iat, value: Fri Apr 19 08:43:08 PDT 2019
 Added claim: jti, value: a-123
 Added claim: customString, value: customStringValue
eyJraWQiOiJcL3ByaXZhdGUta2V5LnBlbSIsInR5cCI6IkpXVCIsImFsZyI6IlJTMjU2In0.eyJzdWIiOiIyNDQwMDMyMCIsImN1c3RvbUludGVnZXJBcnJheSI6WzAsMSwyLDNdLCJjdXN0b21Eb3VibGVBcnJheSI6WzAuMSwxLjEsMi4yLDMuMyw0LjRdLCJpc3MiOiJodHRwOlwvXC9pby5wYWNrdC5qd3QiLCJncm91cHMiOlsiRWNob2VyIiwiVGVzdGVyIiwiVXNlciIsImdyb3VwMSIsImdyb3VwMiJdLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJqZG9lIiwiY3VzdG9tU3RyaW5nQXJyYXkiOlsidmFsdWUwIiwidmFsdWUxIiwidmFsdWUyIl0sImF1ZCI6InM2QmhkUmtxdDMiLCJ1cG4iOiJqZG9lQGV4YW1wbGUuY29tIiwiY3VzdG9tSW50ZWdlciI6MTIzNDU2Nzg5LCJhdXRoX3RpbWUiOjE1NTU2ODg1ODgsImN1c3RvbU9iamVjdCI6eyJteS1zZXJ2aWNlIjp7InJvbGVzIjpbInJvbGUtaW4tbXktc2VydmljZSJdLCJncm91cHMiOlsiZ3JvdXAxIiwiZ3JvdXAyIl19LCJzZXJ2aWNlLUIiOnsicm9sZXMiOlsicm9sZS1pbi1CIl19LCJzZXJ2aWNlLUMiOnsiZ3JvdXBzIjpbImdyb3VwQyIsIndlYi10aWVyIl19LCJzY2FsZSI6MC42MjV9LCJleHAiOjE1NTU2OTIxODgsImN1c3RvbURvdWJsZSI6My4xNDE1OTI2NTM1ODk3OTMsImlhdCI6MTU1NTY4ODU4OCwianRpIjoiYS0xMjMiLCJjdXN0b21TdHJpbmciOiJjdXN0b21TdHJpbmdWYWx1ZSJ9.Tb8Fet_3NhABc6E5z5N6afwNsxzcZaa9q0eWWLm1AP4HPbJCOA14L275D-jAO42s7yQlHS7sUsi9_nWStDV8MTqoey4PmN2rcnOAaKqCfUiLehcOzg3naUk0AxRykCBO4YIck-qqvlEaZ6q8pVW_2Nfj5wZml2uPDq_X6aVLfxjaRzj2F4OoeKGH51-88yeu7H2THUMNLLPB2MY4Ma0xDUFXVL1TXU49ilOXOWTHAo7wAdqleuZUavtt_ZQfRwCUoI1Y-dltH_WtLdjjYw6aFIeJtsyYCXdqONiP6TqOpfACOXbV_nBYNKpYGn4GMiPsxmpJMU8JAhm-jJzf9Yhq6A
[INFO] -----------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] -----------------------------------------------------------------------
[INFO] Total time:  1.328 s
[INFO] Finished at: 2019-04-19T08:43:08-07:00
[INFO] -----------------------------------------------------------------------
```

这些示例应该能让你对微服务客户端之间的交互以及使用 JWT 来保护微服务端点实现无状态认证和 RBAC（基于角色的访问控制），以及基于 JWT 中声明的定制授权有一个感觉。

# 总结

在本章中，我们学习了 MicroProfile 健康检查和 JWT 传播项目。现在你应该理解了什么是健康检查以及如何添加应用程序特定的检查，这些检查被称为程序。这允许你的微服务在云环境中描述其非琐碎的健康要求。你也应该理解如何使用 JWT 在微服务之上提供认证和授权能力，以控制对端点的访问。你也应该理解如何使用 JWT 中的内容以用户特定方式增强你的微服务。

下一章将介绍 MicroProfile Metrics 和 OpenTracing 特性。这些特性允许你的微服务提供附加信息...

# 问题

1.  马克 wp-hc 协议在所有环境中都有用吗？

1.  一个 MP-HC 响应可以包含任意属性吗？

1.  如果我的应用程序有不同的服务类型需要报告健康状态怎么办？

1.  什么是 JWT（JSON Web Token）？

1.  什么是声明（claim）？

1.  对 JWT 中可以有什么内容有限制吗？

1.  在验证 JWT 时，主要需要经过哪些步骤？

1.  除了安全注解之外，我们还可以如何使用 JWT 来进行授权检查？
