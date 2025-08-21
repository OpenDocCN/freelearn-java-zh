# 第六章：Spring Security 和 JWT（JSON Web Token）

在本章中，我们将简单了解 Spring Security，并且我们还将讨论**JSON Web Token**（**JWT**）以及如何在我们的 web 服务调用中使用 JWT。这也将包括 JWT 的创建。

在本章中，我们将涵盖以下内容：

+   Spring Security

+   JSON Web Token（JWT）

+   如何在 web 服务中生成 JWT

+   如何在 web 服务中访问和检索 JWT 中的信息

+   如何通过添加 JWT 安全来限制 web 服务调用

# Spring Security

Spring Security 是一个强大的身份验证和授权框架，将帮助我们提供一个安全的应用程序。通过使用 Spring Security，我们可以确保所有的 REST API 都是安全的，并且只能通过经过身份验证和授权的调用访问。

# 身份验证和授权

让我们举个例子来解释一下。假设你有一个有很多书的图书馆。身份验证将提供一个进入图书馆的钥匙；然而，授权将给予你取书的权限。没有钥匙，你甚至无法进入图书馆。即使你有图书馆的钥匙，你也只能取几本书。

# JSON Web Token（JWT）

Spring Security 可以以多种形式应用，包括使用强大的库如 JWT 进行 XML 配置。由于大多数公司在其安全中使用 JWT，我们将更多地关注基于 JWT 的安全，而不是简单的 Spring Security，后者可以在 XML 中配置。

JWT 令牌在 URL 上是安全的，并且在**单点登录**（**SSO**）环境中与 Web 浏览器兼容。JWT 有三部分：

+   头部

+   有效载荷

+   签名

头部部分决定了应该使用哪种算法来生成令牌。在进行身份验证时，客户端必须保存服务器返回的 JWT。与传统的会话创建方法不同，这个过程不需要在客户端存储任何 cookie。JWT 身份验证是无状态的，因为客户端状态从未保存在服务器上。

# JWT 依赖

为了在我们的应用程序中使用 JWT，我们可能需要使用 Maven 依赖。以下依赖应该添加到`pom.xml`文件中。您可以从以下链接获取 Maven 依赖：[`mvnrepository.com/artifact/javax.xml.bind`](https://mvnrepository.com/artifact/javax.xml.bind)。

我们在应用程序中使用了 Maven 依赖的版本`2.3.0`：

```java
<dependency>
      <groupId>javax.xml.bind</groupId>
      <artifactId>jaxb-api</artifactId>
      <version>2.3.0</version>
</dependency>
```

由于 Java 9 在其捆绑包中不包括`DataTypeConverter`，我们需要添加上述配置来使用`DataTypeConverter`。我们将在下一节中介绍`DataTypeConverter`。

# 创建 JWT 令牌

为了创建一个令牌，我们在`SecurityService`接口中添加了一个名为`createToken`的抽象方法。该接口将告诉实现类必须为`createToken`创建一个完整的方法。在`createToken`方法中，我们将只使用主题和到期时间，因为在创建令牌时这两个选项很重要。

首先，我们将在`SecurityService`接口中创建一个抽象方法。具体类（实现`SecurityService`接口的类）必须在其类中实现该方法：

```java
public interface SecurityService {
  String createToken(String subject, long ttlMillis);    
 // other methods  
}
```

在上述代码中，我们在接口中定义了令牌创建的方法。

`SecurityServiceImpl`是一个具体的类，它通过应用业务逻辑来实现`SecurityService`接口的抽象方法。以下代码将解释如何使用主题和到期时间来创建 JWT：

```java
private static final String secretKey= "4C8kum4LxyKWYLM78sKdXrzbBjDCFyfX";
@Override
public String createToken(String subject, long ttlMillis) {    
    if (ttlMillis <= 0) {
      throw new RuntimeException("Expiry time must be greater than Zero :["+ttlMillis+"] ");
    }    
    // The JWT signature algorithm we will be using to sign the token
    SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;   
    byte[] apiKeySecretBytes = DatatypeConverter.parseBase64Binary(secretKey);
    Key signingKey = new SecretKeySpec(apiKeySecretBytes, signatureAlgorithm.getJcaName());
    JwtBuilder builder = Jwts.builder()
        .setSubject(subject) 
        .signWith(signatureAlgorithm, signingKey);
    long nowMillis = System.currentTimeMillis();    
    builder.setExpiration(new Date(nowMillis + ttlMillis)); 
    return builder.compact();
}
```

上述代码为主题创建了令牌。在这里，我们已经硬编码了秘钥`"4C8kum4LxyKWYLM78sKdXrzbBjDCFyfX"`，以简化令牌创建过程。如果需要，我们可以将秘钥保存在属性文件中，以避免在 Java 代码中硬编码。

首先，我们验证时间是否大于零。如果不是，我们立即抛出异常。我们使用 SHA-256 算法，因为它在大多数应用程序中都被使用。

**安全哈希算法**（**SHA**）是一种密码哈希函数。密码哈希是数据文件的文本形式。SHA-256 算法生成一个几乎唯一的、固定大小的 256 位哈希。SHA-256 是更可靠的哈希函数之一。

我们已在此类中将密钥硬编码。我们也可以将密钥存储在`application.properties`文件中。但是为了简化流程，我们已经将其硬编码：

```java
private static final String secretKey= "4C8kum4LxyKWYLM78sKdXrzbBjDCFyfX";
```

我们将字符串密钥转换为字节数组，然后将其传递给 Java 类`SecretKeySpec`，以获取`signingKey`。此密钥将用于令牌生成器。此外，在创建签名密钥时，我们使用 JCA，这是我们签名算法的名称。

**Java 密码体系结构**（**JCA**）是 Java 引入的，以支持现代密码技术。

我们使用`JwtBuilder`类来创建令牌，并为其设置到期时间。以下代码定义了令牌创建和到期时间设置选项：

```java
JwtBuilder builder = Jwts.builder()
        .setSubject(subject) 
        .signWith(signatureAlgorithm, signingKey);
long nowMillis = System.currentTimeMillis(); 
builder.setExpiration(new Date(nowMillis + ttlMillis)); 
```

在调用此方法时，我们必须传递毫秒时间，因为`setExpiration`只接受毫秒。

最后，我们必须在我们的`HomeController`中调用`createToken`方法。在调用该方法之前，我们将不得不像下面这样自动装配`SecurityService`：

```java
@Autowired
SecurityService securityService;
```

`createToken`调用编码如下。我们将主题作为参数。为了简化流程，我们已将到期时间硬编码为`2 * 1000 * 60`（两分钟）。

`HomeController.java`：

```java
@Autowired
SecurityService securityService;
@ResponseBody
  @RequestMapping("/security/generate/token")
  public Map<String, Object> generateToken(@RequestParam(value="subject") String subject){    
    String token = securityService.createToken(subject, (2 * 1000 * 60));    
    Map<String, Object> map = new LinkedHashMap<>();
    map.put("result", token);    
    return map;
  }
```

# 生成令牌

我们可以通过在浏览器或任何 REST 客户端中调用 API 来测试令牌。通过调用此 API，我们可以创建一个令牌。此令牌将用于用户身份验证等目的。

创建令牌的示例 API 如下：

```java
http://localhost:8080/security/generate/token?subject=one
```

在这里，我们使用`one`作为主题。我们可以在以下结果中看到令牌。这就是我们为传递给 API 的所有主题生成令牌的方式：

```java
{
  result: "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJvbmUiLCJleHAiOjE1MDk5MzY2ODF9.GknKcywiI-G4-R2bRmBOsjomujP0MxZqdawrB8TO3P4"
}
```

JWT 是一个由三部分组成的字符串，每部分用一个点（.）分隔。每个部分都经过 base-64 编码。第一部分是头部，它提供了关于用于签署 JWT 的算法的线索。第二部分是主体，最后一部分是签名。

# 从 JWT 令牌中获取主题

到目前为止，我们已经创建了一个 JWT 令牌。在这里，我们将解码令牌并从中获取主题。在后面的部分中，我们将讨论如何解码并从令牌中获取主题。

像往常一样，我们必须定义获取主题的方法。我们将在`SecurityService`中定义`getSubject`方法。

在这里，我们将在`SecurityService`接口中创建一个名为`getSubject`的抽象方法。稍后，我们将在我们的具体类中实现这个方法：

```java
String getSubject(String token);
```

在我们的具体类中，我们将实现`getSubject`方法，并在`SecurityServiceImpl`类中添加我们的代码。我们可以使用以下代码从令牌中获取主题：

```java
  @Override
  public String getSubject(String token) {     
    Claims claims = Jwts.parser()              .setSigningKey(DatatypeConverter.parseBase64Binary(secretKey))
             .parseClaimsJws(token).getBody();    
    return claims.getSubject();
  } 
```

在前面的方法中，我们使用`Jwts.parser`来获取`claims`。我们通过将密钥转换为二进制并将其传递给解析器来设置签名密钥。一旦我们得到了`Claims`，我们可以通过调用`getSubject`来简单地获取主题。

最后，我们可以在我们的控制器中调用该方法，并传递生成的令牌以获取主题。您可以检查以下代码，其中控制器调用`getSubject`方法，并在`HomeController.java`文件中返回主题：

```java
  @ResponseBody
  @RequestMapping("/security/get/subject")
  public Map<String, Object> getSubject(@RequestParam(value="token") String token){    
    String subject = securityService.getSubject(token);    
    Map<String, Object> map = new LinkedHashMap<>();
    map.put("result", subject);    
    return map;
  }
```

# 从令牌中获取主题

以前，我们创建了获取令牌的代码。在这里，我们将通过调用获取主题 API 来测试我们之前创建的方法。通过调用 REST API，我们将得到之前传递的主题。

示例 API：

```java
http://localhost:8080/security/get/subject?token=eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJvbmUiLCJleHAiOjE1MDk5MzY2ODF9.GknKcywiI-G4-R2bRmBOsjomujP0MxZqdawrB8TO3P4
```

由于我们在调用`generateToken`方法创建令牌时使用了`one`作为主题，所以我们将在`getSubject`方法中得到`"one"`：

```java
{
  result: "one"
}
```

通常，我们将令牌附加在标头中；然而，为了避免复杂性，我们已经提供了结果。此外，我们已将令牌作为参数传递给`getSubject`。在实际应用中，您可能不需要以相同的方式进行操作。这只是为了演示目的。

# 摘要

在本章中，我们已经讨论了 Spring Security 和基于 JWT 令牌的安全性，以获取和解码令牌。在未来的章节中，我们将讨论如何在 AOP 中使用令牌，并通过使用 JWT 令牌来限制 API 调用。
