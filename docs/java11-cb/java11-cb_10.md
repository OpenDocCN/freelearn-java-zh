# 网络

在本章中，我们将介绍以下示例：

+   发出 HTTP GET 请求

+   发出 HTTP POST 请求

+   为受保护的资源发出 HTTP 请求

+   发出异步 HTTP 请求

+   使用 Apache HttpClient 发出 HTTP 请求

+   使用 Unirest HTTP 客户端库发出 HTTP 请求

# 介绍

Java 对与 HTTP 特定功能进行交互的支持非常原始。自 JDK 1.1 以来可用的`HttpURLConnection`类提供了与具有 HTTP 特定功能的 URL 进行交互的 API。由于此 API 甚至在 HTTP/1.1 之前就存在，它缺乏高级功能，并且使用起来很麻烦。这就是为什么开发人员大多倾向于使用第三方库，如**Apache HttpClient**、Spring 框架和 HTTP API。

在 JDK 9 中，引入了一个新的 HTTP 客户端 API，作为孵化器模块的一部分，名称为`java.net.http`，在 JEP 321 ([`openjdk.java.net/jeps/321`](http://openjdk.java.net/jeps/321))下被提升为标准模块，这是最新的 JDK 11 版本的一部分。

关于孵化器模块的说明：孵化器模块包含非最终 API，这些 API 非常庞大，不够成熟，无法包含在 Java SE 中。这是 API 的一种测试版发布，使开发人员能够更早地使用 API。但问题是，这些 API 在较新版本的 JDK 中没有向后兼容性支持。这意味着依赖于孵化器模块的代码可能会在较新版本的 JDK 中出现问题。这可能是因为孵化器模块被提升为 Java SE 或在孵化器模块中被悄悄删除。

在本章中，我们将介绍如何在 JDK 11 中使用 HTTP 客户端 API，并介绍一些其他 API，这些 API 使用了 Apache HttpClient ([`hc.apache.org/httpcomponents-client-ga/`](http://hc.apache.org/httpcomponents-client-ga/)) API 和 Unirest Java HTTP 库 ([`unirest.io/java.html`](http://unirest.io/java.html))。

# 发出 HTTP GET 请求

在本示例中，我们将使用 JDK 11 的 HTTP 客户端 API 发出对[`httpbin.org/get`](http://httpbin.org/get)的`GET`请求。

# 如何做...

1.  使用其构建器`java.net.http.HttpClient.Builder`创建`java.net.http.HttpClient`的实例：

```java
        HttpClient client = HttpClient.newBuilder().build();
```

1.  使用其构建器`java.net.http.HttpRequest.Builder`创建`java.net.http.HttpRequest`的实例。请求的 URL 应该作为`java.net.URI`的实例提供：

```java
        HttpRequest request = HttpRequest
                    .newBuilder(new URI("http://httpbin.org/get"))
                    .GET()
                    .version(HttpClient.Version.HTTP_1_1)
                    .build();
```

1.  使用`java.net.http.HttpClient`的`send` API 发送 HTTP 请求。此 API 需要一个`java.net.http.HttpRequest`实例和一个`java.net.http.HttpResponse.BodyHandler`的实现：

```java
        HttpResponse<String> response = client.send(request,
                             HttpResponse.BodyHandlers.ofString());
```

1.  打印`java.net.http.HttpResponse`状态码和响应体：

```java
        System.out.println("Status code: " + response.statusCode());
        System.out.println("Response Body: " + response.body());
```

此代码的完整代码可以在`Chapter10/1_making_http_get`中找到。您可以使用运行脚本`run.bat`或`run.sh`来编译和运行代码：

![](img/4aa7f3fc-3c65-4b06-bbe7-d5cee9c68f3c.png)

# 工作原理...

在向 URL 发出 HTTP 调用时有两个主要步骤：

+   创建 HTTP 客户端以发起调用

+   设置目标 URL、所需的 HTTP 标头和 HTTP 方法类型，即`GET`、`POST`或`PUT`

```java
java.net.http.HttpClient with the default configuration:
```

```java
HttpClient client = HttpClient.newHttpClient();
java.net.http.HttpClient:
```

```java
HttpClient client = HttpClient
                    .newBuilder()
                    //redirect policy for the client. Default is NEVER
                    .followRedirects(HttpClient.Redirect.ALWAYS) 
                    //HTTP client version. Defabult is HTTP_2
                    .version(HttpClient.Version.HTTP_1_1)
                    //few more APIs for more configuration
                    .build();
```

在构建器中还有更多的 API，例如用于设置身份验证、代理和提供 SSL 上下文，我们将在不同的示例中进行讨论。

```java
java.net.http.HttpRequest:
```

```java
HttpRequest request = HttpRequest
                .newBuilder()
                .uri(new URI("http://httpbin.org/get")
                .headers("Header 1", "Value 1", "Header 2", "Value 2")
                .timeout(Duration.ofMinutes(5))
                .version(HttpClient.Version.HTTP_1_1)
                .GET()
                .build();
```

`java.net.http.HttpClient`对象提供了两个 API 来发出 HTTP 调用：

+   您可以使用`HttpClient#send()`方法同步发送

+   您可以使用`HttpClient#sendAsync()`方法异步发送

`send()`方法接受两个参数：HTTP 请求和 HTTP 响应的处理程序。响应的处理程序由`java.net.http.HttpResponse.BodyHandlers`接口的实现表示。有一些可用的实现，例如`ofString()`，它将响应体读取为`String`，以及`ofByteArray()`，它将响应体读取为字节数组。我们将使用`ofString()`方法，它将响应`Body`作为字符串返回：

```java
HttpResponse<String> response = client.send(request,
                                HttpResponse.BodyHandlers.ofString());
```

`java.net.http.HttpResponse`的实例表示来自 HTTP 服务器的响应。它提供以下 API：

+   获取响应体（`body()`）

+   HTTP 头（`headers()`）

+   初始 HTTP 请求（`request()`）

+   响应状态码（`statusCode()`）

+   用于请求的 URL（`uri()`）

传递给`send()`方法的`HttpResponse.BodyHandlers`实现有助于将 HTTP 响应转换为兼容格式，例如`String`或`byte`数组。

# 发出 HTTP POST 请求

在本示例中，我们将查看通过请求体将一些数据发布到 HTTP 服务。我们将把数据发布到`http://httpbin.org/post`的 URL。

我们将跳过类的包前缀，因为假定为`java.net.http`。

# 如何做...

1.  使用其`HttpClient.Builder`构建器创建`HttpClient`的实例：

```java
        HttpClient client = HttpClient.newBuilder().build();
```

1.  创建要传递到请求体中的所需数据：

```java
        Map<String, String> requestBody = 
                    Map.of("key1", "value1", "key2", "value2");
```

1.  创建一个`HttpRequest`对象，请求方法为 POST，并提供请求体数据作为`String`。我们将使用 Jackson 的`ObjectMapper`将请求体`Map<String, String>`转换为纯 JSON`String`，然后使用`HttpRequest.BodyPublishers`处理`String`请求体：

```java
        ObjectMapper mapper = new ObjectMapper();
        HttpRequest request = HttpRequest
                   .newBuilder(new URI("http://httpbin.org/post"))
                   .POST(
          HttpRequest.BodyPublishers.ofString(
            mapper.writeValueAsString(requestBody)
          )
        )
        .version(HttpClient.Version.HTTP_1_1)
        .build();
```

1.  使用`send(HttpRequest, HttpRequest.BodyHandlers)`方法发送请求并获取响应：

```java
        HttpResponse<String> response = client.send(request, 
                             HttpResponse.BodyHandlers.ofString());
```

1.  然后我们打印服务器发送的响应状态码和响应体：

```java
        System.out.println("Status code: " + response.statusCode());
        System.out.println("Response Body: " + response.body());
```

此代码的完整代码可以在`Chapter10/2_making_http_post`中找到。确保在`Chapter10/2_making_http_post/mods`中有以下 Jackson JARs：

+   `jackson.databind.jar`

+   `jackson.core.jar`

+   `jackson.annotations.jar`

还要注意`Chapter10/2_making_http_post/src/http.client.demo`中可用的模块定义`module-info.java`。

要了解此模块化代码中如何使用 Jackson JAR，请参阅第三章中的*自下而上迁移*和*自上而下迁移*示例，*模块化编程*。

运行脚本`run.bat`和`run.sh`，用于简化代码的编译和执行：

![](img/10de5762-5fd8-4b96-92cd-d05be5312f5d.png)

# 发出对受保护资源的 HTTP 请求

在本示例中，我们将查看调用已由用户凭据保护的 HTTP 资源。[`httpbin.org/basic-auth/user/passwd`](http://httpbin.org/basic-auth/user/passwd)已通过 HTTP 基本身份验证进行了保护。基本身份验证需要提供明文用户名和密码，然后 HTTP 资源使用它来决定用户身份验证是否成功。

如果您在浏览器中打开[`httpbin.org/basic-auth/user/passwd`](http://httpbin.org/basic-auth/user/passwd)，它将提示您输入用户名和密码：

![](img/db91ec67-11cd-4d05-8362-cba04a189031.png)

将用户名输入为`user`，密码输入为`passwd`，您将被验证并显示 JSON 响应：

```java
{
  "authenticated": true,
  "user": "user"
}
```

让我们使用`HttpClient` API 实现相同的功能。

# 如何做...

1.  我们需要扩展`java.net.Authenticator`并重写其`getPasswordAuthentication()`方法。此方法应返回`java.net.PasswordAuthentication`的实例。让我们创建一个类`UsernamePasswordAuthenticator`，它扩展`java.net.Authenticator`：

```java
        public class UsernamePasswordAuthenticator 
          extends Authenticator{
        }
```

1.  我们将在`UsernamePasswordAuthenticator`类中创建两个实例变量来存储用户名和密码，并提供一个构造函数来初始化它：

```java
        private String username;
        private String password;

        public UsernamePasswordAuthenticator(){}
        public UsernamePasswordAuthenticator ( String username, 
                                               String password){
          this.username = username;
          this.password = password;
        }
```

1.  然后我们将重写`getPasswordAuthentication()`方法，返回一个用用户名和密码初始化的`java.net.PasswordAuthentication`的实例：

```java
        @Override
        protected PasswordAuthentication getPasswordAuthentication(){
          return new PasswordAuthentication(username, 
                                            password.toCharArray());
        }
```

1.  然后我们将创建一个`UsernamePasswordAuthenticator`的实例：

```java
        String username = "user";
        String password = "passwd"; 
        UsernamePasswordAuthenticator authenticator = 
                new UsernamePasswordAuthenticator(username, password);
```

1.  在初始化`HttpClient`时，我们提供`UsernamePasswordAuthenticator`的实例：

```java
        HttpClient client = HttpClient.newBuilder()
                                      .authenticator(authenticator)
                                      .build();
```

1.  创建一个对受保护的 HTTP 资源[`httpbin.org/basic-auth/user/passwd`](http://httpbin.org/basic-auth/user/passwd)的`HttpRequest`对象：

```java
        HttpRequest request = HttpRequest.newBuilder(new URI(
          "http://httpbin.org/basic-auth/user/passwd"
        ))
        .GET()
        .version(HttpClient.Version.HTTP_1_1)
        .build();
```

1.  我们通过执行请求来获取`HttpResponse`，并打印状态码和请求体：

```java
        HttpResponse<String> response = client.send(request,
        HttpResponse.BodyHandlers.ofString());

        System.out.println("Status code: " + response.statusCode());
        System.out.println("Response Body: " + response.body());
```

这个配方的完整代码可以在`Chapter10/3_making_http_request_protected_res`中找到。您可以使用`run.bat`或`run.sh`脚本来运行代码：

![](img/566c7e2b-6d4d-47a5-943c-6a3f60329aaf.png)

# 它是如何工作的...

`Authenticator`对象被网络调用使用来获取认证信息。开发人员通常扩展`java.net.Authenticator`类，并重写其`getPasswordAuthentication()`方法。用户名和密码要么从用户输入中读取，要么从配置中读取，并由扩展类用来创建`java.net.PasswordAuthentication`的实例。

在这个配方中，我们创建了`java.net.Authenticator`的扩展，如下所示：

```java
public class UsernamePasswordAuthenticator 
  extends Authenticator{
    private String username;
    private String password;

    public UsernamePasswordAuthenticator(){}

    public UsernamePasswordAuthenticator ( String username, 
                                           String password){
        this.username = username;
        this.password = password;
    }

    @Override
    protected PasswordAuthentication getPasswordAuthentication(){
      return new PasswordAuthentication(username, 
                         password.toCharArray());
    }
}
```

然后将`UsernamePasswordAuthenticator`的实例提供给`HttpClient.Builder`API。`HttpClient`实例利用这个验证器来获取用户名和密码，同时调用受保护的 HTTP 请求。

# 进行异步 HTTP 请求

在这个配方中，我们将看看如何进行异步的`GET`请求。在异步请求中，我们不等待响应；相反，我们在客户端接收到响应时处理响应。在 jQuery 中，我们将进行异步请求，并提供一个回调来处理响应，而在 Java 的情况下，我们会得到一个`java.util.concurrent.CompletableFuture`的实例，然后我们调用`thenApply`方法来处理响应。让我们看看它是如何工作的。

# 如何做...

1.  使用其构建器`HttpClient.Builder`创建`HttpClient`的实例：

```java
        HttpClient client = HttpClient.newBuilder().build();
```

1.  使用`HttpRequest.Builder`的构建器创建`HttpRequest`的实例，表示要使用的 URL 和相应的 HTTP 方法：

```java
        HttpRequest request = HttpRequest
                        .newBuilder(new URI("http://httpbin.org/get"))
                        .GET()
                        .version(HttpClient.Version.HTTP_1_1)
                        .build();
```

1.  使用`sendAsync`方法进行异步 HTTP 请求，并保留我们获取的`CompletableFuture<HttpResponse<String>>`对象的引用。我们将使用这个对象来处理响应：

```java
        CompletableFuture<HttpResponse<String>> responseFuture = 
                  client.sendAsync(request, 
                         HttpResponse.BodyHandlers.ofString());
```

1.  我们提供`CompletionStage`来处理响应，一旦前一个阶段完成。为此，我们使用`thenAccept`方法，它接受一个 lambda 表达式：

```java
        CompletableFuture<Void> processedFuture = 
                   responseFuture.thenAccept(response -> {
          System.out.println("Status code: " + response.statusCode());
          System.out.println("Response Body: " + response.body());
        });
```

1.  等待未来完成：

```java
        CompletableFuture.allOf(processedFuture).join();
```

这个配方的完整代码可以在`Chapter10/4_async_http_request`中找到。我们提供了`run.bat`和`run.sh`脚本来编译和运行这个配方：

![](img/7235ca68-1f33-4067-b28b-26e6ee66d538.png)

# 使用 Apache HttpClient 进行 HTTP 请求

在这个配方中，我们将使用 Apache HttpClient ([`hc.apache.org/httpcomponents-client-4.5.x/index.html`](https://hc.apache.org/httpcomponents-client-4.5.x/index.html))库来进行简单的 HTTP `GET`请求。由于我们使用的是 Java 9，我们希望使用模块路径而不是类路径。因此，我们需要将 Apache HttpClient 库模块化。实现这一目标的一种方法是使用自动模块的概念。让我们看看如何为这个配方设置依赖关系。

# 准备就绪

所有必需的 JAR 文件已经存在于`Chapter10/5_apache_http_demo/mods`中：

![](img/b3865825-465b-4302-9f43-8af7bb556344.png)

一旦这些 JAR 文件在模块路径上，我们可以在`module-info.java`中声明对这些 JAR 文件的依赖关系，该文件位于`Chapter10/5_apache_http_demo/src/http.client.demo`中，如下面的代码片段所示：

```java
module http.client.demo{
  requires httpclient;
  requires httpcore;
  requires commons.logging;
  requires commons.codec;
}
```

# 如何做...

1.  使用其`org.apache.http.impl.client.HttpClients`工厂创建`org.http.client.HttpClient`的默认实例：

```java
        CloseableHttpClient client = HttpClients.createDefault();
```

1.  创建`org.apache.http.client.methods.HttpGet`的实例以及所需的 URL。这代表了 HTTP 方法类型和请求的 URL：

```java
        HttpGet request = new HttpGet("http://httpbin.org/get");
```

1.  使用`HttpClient`实例执行 HTTP 请求以获取`CloseableHttpResponse`的实例：

```java
        CloseableHttpResponse response = client.execute(request);
```

执行 HTTP 请求后返回的`CloseableHttpResponse`实例可用于获取响应状态码和嵌入在`HttpEntity`实现实例中的响应内容等详细信息。

1.  我们使用`EntityUtils.toString()`来获取嵌入在`HttpEntity`实现实例中的响应体，并打印状态码和响应体：

```java
        int statusCode = response.getStatusLine().getStatusCode();
        String responseBody = 
                       EntityUtils.toString(response.getEntity());
        System.out.println("Status code: " + statusCode);
        System.out.println("Response Body: " + responseBody);
```

此示例的完整代码可以在`Chapter10/5_apache_http_demo`中找到。我们提供了`run.bat`和`run.sh`来编译和执行示例代码：

![](img/09441745-683b-444a-8a77-e3e4ad1be9c8.png)

# 还有更多...

在调用`HttpClient.execute`方法时，我们可以提供自定义的响应处理程序，如下所示：

```java
String responseBody = client.execute(request, response -> {
  int status = response.getStatusLine().getStatusCode();
  HttpEntity entity = response.getEntity();
  return entity != null ? EntityUtils.toString(entity) : null;
});
```

在这种情况下，响应由响应处理程序处理并返回响应体字符串。完整的代码可以在`Chapter10/5_1_apache_http_demo_response_handler`中找到。

# 使用 Unirest HTTP 客户端库进行 HTTP 请求

在本示例中，我们将使用 Unirest HTTP ([`unirest.io/java.html`](http://unirest.io/java.html)) Java 库来访问 HTTP 服务。Unirest Java 是一个基于 Apache 的 HTTP 客户端库的库，并提供了一个流畅的 API 来进行 HTTP 请求。

# 准备工作

由于 Java 库不是模块化的，我们将利用自动模块的概念，如第三章 *模块化编程*中所解释的。该库的 JAR 文件被放置在应用程序的模块路径上，然后应用程序通过使用 JAR 的名称作为其模块名称来声明对 JAR 的依赖关系。这样，JAR 文件就会自动成为一个模块，因此被称为自动模块。

Java 库的 Maven 依赖如下：

```java
<dependency>
  <groupId>com.mashape.unirest</groupId>
  <artifactId>unirest-java</artifactId>
  <version>1.4.9</version>
</dependency>
```

由于我们的示例中没有使用 Maven，我们已经将 JAR 文件下载到了`Chapter10/6_unirest_http_demo/mods`文件夹中。

模块定义如下：

```java
module http.client.demo{
  requires httpasyncclient;
  requires httpclient;
  requires httpmime;
  requires json;
  requires unirest.java;
  requires httpcore;
  requires httpcore.nio;
  requires commons.logging;
  requires commons.codec;
}
```

# 操作步骤如下...

Unirest 提供了一个非常流畅的 API 来进行 HTTP 请求。我们可以按如下方式进行`GET`请求：

```java
HttpResponse<JsonNode> jsonResponse = 
  Unirest.get("http://httpbin.org/get")
         .asJson();
```

可以从`jsonResponse`对象中获取响应状态和响应体：

```java
int statusCode = jsonResponse.getStatus();
JsonNode jsonBody = jsonResponse.getBody();
```

我们可以进行`POST`请求并传递一些数据，如下所示：

```java
jsonResponse = Unirest.post("http://httpbin.org/post")
                      .field("key1", "val1")
                      .field("key2", "val2")
                      .asJson();
```

我们可以按如下方式调用受保护的 HTTP 资源：

```java
jsonResponse = Unirest.get("http://httpbin.org/basic-auth/user/passwd")
                      .basicAuth("user", "passwd")
                      .asJson();
```

该代码可以在`Chapter10/6_unirest_http_demo`中找到。

我们提供了`run.bat`和`run.sh`脚本来执行代码。

# 还有更多...

Unirest Java 库提供了更多高级功能，例如进行异步请求、文件上传和使用代理。建议您尝试该库的不同功能。
