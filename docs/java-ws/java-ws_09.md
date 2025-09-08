# 第九章：9. 使用 HTTP

概述

在本章中，我们将探讨 HTTP 的基础知识，并创建一个程序来连接到特定的 Web 服务器并下载数据。我们将从研究 HTTP 请求方法开始，这样你就可以开始练习使用 Java 的 `HttpUrlConnection` 类自行发送请求。然后，你将学习使用 GET 和 HEAD 请求检索数据，以及使用 POST 请求发送 JSON 格式的数据。在本章的末尾，你将学习如何使用开源的 jsoup 库提取和解析 HTML 内容，并探索 Java 11 提供的 `java.net.http` 模块——这是一个新的 HTTP 类，它支持同步和异步 HTTP 请求。

# 简介

**超文本传输协议**（**HTTP**）是 **万维网**（**WWW**）的基础。使用 HTTP 的 **请求-响应协议**，客户端（如网页浏览器）可以从服务器请求数据。在万维网中，网页浏览器请求内容（HTML、JavaScript、图像等），然后显示结果。在许多情况下，返回的内容相对静态。

Java 应用程序通常有所不同。在大多数情况下，使用 Java，你将向专门的后端 Web 服务发送请求以收集数据或更新系统。然而，在这两种情况下，Java 编码保持不变。

本章介绍了如何在 Java 应用程序中发送 HTTP 请求并解析响应数据。

# 探索 HTTP

使用 HTTP，客户端应用程序向服务器发送一个特殊格式的请求，然后等待响应。技术上讲，HTTP 是一种无状态协议。这意味着服务器不需要维护与客户端相关的任何状态信息。每个客户端请求都可以作为一个新的操作单独处理。服务器不需要存储特定于客户端的信息。

许多服务器确实在多个请求之间维护某种状态，例如，当你在线购物时，服务器需要存储你选择的产品；然而，基本协议并不要求这样做。

HTTP 是一种文本协议（允许压缩）。HTTP 请求包括以下部分：

+   一个操作（称为 **请求方法**），操作的资源标识符，以及一行上的可选参数。

+   请求头；每行一个。

+   一行空行。

+   一个可选的消息主体。

+   每行以两个字符结束：一个回车符和一个换行符。

HTTP 使用以下两个主要概念来识别你感兴趣的资源：

+   **统一资源标识符**（**URI**）格式的资源标识符用于标识服务器上的资源。

+   **统一资源定位符**（**URL**）包括一个 URI，以及网络协议信息、服务器和端口号。URL 是你在网页浏览器的地址栏中输入的内容。

Java 包含了这两个概念的相关类：`java.net.URI` 和 `java.net.URL`。

例如，考虑以下 URL：`http://example.com:80/`。

在这个例子中，`http` 标识了协议。服务器是 `example.com`，端口号是 `80`（默认的 HTTP 端口号）。尾随的字符标识了服务器上的资源，在这种情况下，是顶级或根资源。

大多数现代网站都使用 HTTPS 协议。这是一个更安全的 HTTP 版本，因为发送到和从服务器发送的数据都是加密的。

例如，考虑以下 URL：[`www.packtpub.com/`](https://www.packtpub.com/)。

在这个情况下，协议是 `https`，服务器是 `www.packtpub.com`。端口号默认为 `443`（HTTPS 的默认端口号）。和之前一样，尾随的字符标识了服务器上的资源。

一个 URI 可以有完整的网络位置，也可以相对于服务器。以下都是有效的 URIs：

+   `https://www.packtpub.com/tech/java`

+   `http://www.example.com/docs/java.html`

+   `/tech/java`

+   `/docs/java.html`

+   `file:///java.html`

URL 是一个较旧的术语，通常表示互联网上资源的完整规范。话虽如此，像 URIs 一样，你也可以有相对 URL，例如 `java.html`。在大多数情况下，人们谈论 URIs。

然而，通常情况下，你的 Java 应用程序将使用 `URL` 类来建立 HTTP 连接。

注意

你可以在 [`packt.live/32ATULO`](https://packt.live/32ATULO) 上了解更多关于 URIs 的信息，以及 [`packt.live/2JjIgNN`](https://packt.live/2JjIgNN) 上的 URLs。

## HTTP 请求方法

每个 HTTP 请求都以一个请求方法开始，例如 GET。这些方法名称来自万维网的早期。以下是一些方法：

+   `GET`：这从服务器检索数据。

+   `HEAD`：这类似于 GET 请求，但只检索头部信息，不包含响应体。

+   `POST`：这向服务器发送数据。大多数网页上的 HTML 表单将你填写的数据作为 POST 请求发送。

+   `PUT`：这也向服务器发送数据。PUT 请求通常用于修改资源，替换现有资源的内容。

+   `DELETE`：这请求服务器删除指定的资源。

+   `TRACE`：这个方法会回显服务器接收到的请求数据。这可以用于调试。

+   `OPTIONS`：这列出了服务器为给定 URL 支持的 HTTP 方法。

    注意

    还有其他 HTTP 方法，值得注意的是 `CONNECT` 和 `PATCH`。本章中描述的 `HttpUrlConnection` 类只支持这里列出的方法。

## 表示状态传输

**表示状态传输**（**REST**）是一个术语，用来描述使用 HTTP 作为传输协议的 Web 服务。你可以将其视为带有对象的 HTTP。例如，在 RESTful Web 服务中，一个 GET 请求通常返回一个对象，格式化为 **JavaScript 对象表示法**（**JSON**）。JSON 提供了一种将对象编码为文本的方式，这种方式与使用的编程语言无关。JSON 使用 JavaScript 语法以名称-值对或数组的形式格式化数据：

```java
{
    "animal": "dog",
    "name": "biff"
}
```

在这个例子中，对象有两个属性：`animal`和`name`。

注意

许多 RESTful 网络服务以 JSON 格式发送和接收数据。您可以参考*第十九章*，*反射*，以获取有关 JSON 的更多信息。

在网络服务中，通常使用`POST`请求来创建一个新的对象，使用`PUT`请求来修改现有的对象（通过用新数据替换它），而使用`DELETE`请求来删除一个对象。

注意

一些框架对`POST`和`PUT`操作有不同的含义。这里采用的方法是 Spring Boot 框架所使用的方法。

您会发现 Java 被大量用于创建 RESTful 网络服务以及网络服务客户端。

注意

您可以在[`packt.live/2MxDcHz`](https://packt.live/2MxDcHz)上阅读 HTTP 规范或阅读 HTTP 的概述[`packt.live/35MM1od`](https://packt.live/35MM1od)。有关 RESTful 网络服务的更多信息，请参阅[`packt.live/2MYvHbq`](https://packt.live/2MYvHbq)。有关 JSON 的更多信息，请参阅[`packt.live/2P2Qz3W`](https://packt.live/2P2Qz3W)。

## 请求头部

请求头部是一个提供一些信息的名称-值对。例如，`User-Agent`请求头部标识代表用户运行的应用程序，通常是网络浏览器。几乎所有的`User-Agent`字符串都以`Mozilla/5.0`开头，这是由于历史原因，并且因为一些网站如果没有提到现在已经古老的 Mozilla 网络浏览器将无法正确渲染。服务器确实使用`User-Agent`头部来指导针对特定浏览器的渲染。例如，考虑以下内容：

```java
Mozilla/5.0 (iPhone; CPU iPhone OS 12_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/12.0 Mobile/15E148 Safari/604.1
```

这个`User-Agent`设置标识了一个 iPhone 浏览器。`Referer`头部（由于历史原因拼写错误）标识了您来自的网页。`Accept`头部列出了您希望的数据格式，例如`text/html`。`Accept-Language`头部列出了一个语言代码，例如如果您希望响应为德语（Deutsch），则使用`de`。

关于请求头部的一个重要观点是，每个头部可以包含多个值（以逗号分隔），尽管在大多数情况下您将提供一个单一的值。

注意

您可以在[`packt.live/2pFjIaH`](https://packt.live/2pFjIaH)上查看常用请求头部的列表。

HTTP 响应消息还包含头部信息。这些响应头部可以告诉您的应用程序有关远程资源的信息。

既然我们已经提到了 HTTP 的要点，下一步就是开始进行网络请求。

# 使用 HttpUrlConnection

`java.net.HttpUrlConnection`类提供了从 Java 访问 HTTP 资源的主要方式。要建立 HTTP 连接，您可以使用以下代码：

```java
String path = "http://example.com";
URL url = new URL(path);
HttpURLConnection connection = (HttpURLConnection) url.openConnection();
connection.setRequestMethod("HEAD");
```

此代码设置了一个以链接到 example.com 初始化的 URL。然后 URL 上的`openConnection()`方法返回`HttpUrlConnection`。一旦您有了`HttpUrlConnection`，您就可以设置 HTTP 方法（在这种情况下为`HEAD`）。您可以从服务器获取数据，上传数据到服务器，并指定请求标题。

使用`HttpUrlConnection`，您可以调用`setRequestProperty()`来指定请求标题：

```java
connection.setRequestProperty("User-Agent", "Mozilla/5.0");
```

每个请求都会生成一个响应，这可能成功或不成功。要检查响应，请获取响应代码：

```java
int responseCode = connection.getResponseCode();
```

代码 200 表示成功。200 范围内的其他代码也表示成功，但有条件，例如 204，表示成功但没有内容。300 范围内的代码表示重定向。400 范围内的代码指向客户端错误，例如可怕的 404 未找到错误，500 范围内的代码指向服务器错误。

注意

您可以在[`packt.live/2OP9Rtr`](https://packt.live/2OP9Rtr)查看 HTTP 响应代码列表。这些也在`HttpUrlConnection`类中定义为常量。

每个响应通常都附带一条消息，例如`OK`。您可以通过调用`getResponseMessage()`来检索此消息：

```java
System.out.println( connection.getResponseMessage() );
```

要查看响应中的标题，请调用`getHeaderFields()`。此方法返回一个标题映射，其中值是一个字符串列表：

```java
Map<String, List<String>> headers = connection.getHeaderFields();
for (String key : headers.keySet()) {
    System.out.println("Key: " + key + " Value: " + headers.get(key));
}
```

注意

使用 HTTP，每个标题可以具有多个值，这就是为什么映射中的值是一个列表。

您还可以逐个检索标题。下一项练习将所有这些内容结合起来，向您展示如何编写一个简短的 Java 程序来创建一个`HTTP` `HEAD`请求。

## 练习 1：创建 HEAD 请求

此练习将向`example.com`发送 HEAD 请求，这是一个官方的练习域名，您可以用它来测试：

1.  在 IntelliJ 的`文件`菜单中选择`新建`然后选择`项目`。

1.  选择项目类型为`Gradle`。点击`下一步`。

1.  对于组 ID，输入`com.packtpub.net`。

1.  对于工件 ID，输入`chapter09`。

1.  对于版本，输入`1.0`。

1.  接受下一页上的默认设置。点击`下一步`。

1.  将项目名称保留为`chapter09`。

1.  点击`完成`。

1.  在 IntelliJ 文本编辑器中调用`build.gradle`。

1.  将`sourceCompatibility`更改为 12：

    ```java
    sourceCompatibility = 12
    ```

1.  在`src/main/java`文件夹中，创建一个新的 Java 包。

1.  将包名输入为`com.packtpub.http`。

1.  在`Project`窗格中右键单击此包，创建一个名为`HeadRequest`的新 Java 类。

1.  输入以下代码：

```java
HeadRequest.java
1  package com.packtpub.http;
2  
3  import java.io.IOException;
4  import java.net.HttpURLConnection;
5  import java.net.MalformedURLException;
6  import java.net.URL;
7  import java.util.List;
8  import java.util.Map;
9  
10 public class HeadRequest {
11     public static void main(String[] args) {
12         String path = "http://example.com";
https://packt.live/2MuxxlC
```

当您运行此程序时，您将看到以下输出：

```java
Code: 200
OK
Accept-Ranges: [bytes]
null: [HTTP/1.1 200 OK]
X-Cache: [HIT]
Server: [ECS (sec/96DC)]
Etag: ["1541025663+gzip"]
Cache-Control: [max-age=604800]
Last-Modified: [Fri, 09 Aug 2013 23:54:35 GMT]
Expires: [Mon, 18 Mar 2019 20:41:30 GMT]
Content-Length: [1270]
Date: [Mon, 11 Mar 2019 20:41:30 GMT]
Content-Type: [text/html; charset=UTF-8]
```

`200`代码表示我们的请求成功。然后您可以看到响应标题。输出中的方括号来自 Java 打印列表的默认方式。

您可以自由地将初始 URL 更改为除`example.com`之外的网站。

## 使用 GET 请求读取响应数据

使用 GET 请求，您将从连接中获取 `InputStream` 以查看响应。调用 `getInputStream()` 获取您请求的资源（URL）服务器发送回来的数据。如果响应代码指示错误，使用 `getErrorStream()` 获取有关错误的信息，例如一个找不到页面。如果您期望响应中包含文本数据，例如 HTML、文本、XML 等，您可以将 `InputStream` 包装在 `BufferedReader` 中：

```java
BufferedReader in = new BufferedReader(
        new InputStreamReader(connection.getInputStream())
    );
String line;
while ((line = in.readLine()) != null) {
    System.out.println(line);
}
in.close();
```

## 练习 2：创建 GET 请求

此练习打印出 example.com 的 HTML 内容。如果您愿意，可以更改 URL 并与其他网站进行实验：

1.  在 IntelliJ 的项目面板中，右键单击 `com.packtpub.http` 包。选择 `New`，然后 `Java Class`。

1.  将 `GetRequest` 作为 Java 类的名称。

1.  为 `GetRequest.java` 输入以下代码：

    ```java
    GetRequest.java
    1  package com.packtpub.http;
    2  
    3  import java.io.BufferedReader;
    4  import java.io.IOException;
    5  import java.io.InputStreamReader;
    6  import java.net.HttpURLConnection;
    7  import java.net.MalformedURLException;
    8  import java.net.URL;
    9  
    10 public class GetRequest {
    11     public static void main(String[] args) {
    12         String path = "http://example.com";
    https://packt.live/2oLZrjZ
    ```

1.  运行此程序，您将看到 `example.com` 的简要 HTML 内容。

使用此技术，我们可以编写一个程序，使用 GET 请求打印网页内容。

# 处理慢速连接

`HttpUrlConnection` 提供了两种方法来帮助处理慢速连接：

```java
connection.setConnectTimeout(6000);
connection.setReadTimeout(6000);
```

调用 `setConnectTimeout()` 来调整建立与远程站点的网络连接时的超时时间。您提供的输入值应为毫秒。调用 `setReadTimeout()` 来调整读取输入流数据时的超时时间。同样，提供新的超时输入值（毫秒）。

## 请求参数

在许多网络服务中，您在发出请求时必须输入参数。HTTP 参数编码为名称-值对。例如，考虑以下：

```java
String path = "http://example.com?name1=value1&name2=value2";
```

在这种情况下，`name1` 是参数的名称，同样 `name2` 也是。`name1` 参数的值是 `value1`，`name2` 的值是 `value2`。参数由一个与符号 `&` 分隔。

注意

如果参数值是简单的字母数字值，您可以像示例中那样输入它们。如果不是，您需要使用 URL 编码对参数数据进行编码。您可以参考 `java.net.URLEncoder` 类以获取更多详细信息。

## 处理重定向

在许多情况下，当您向服务器发出 HTTP 请求时，服务器会响应一个状态指示重定向。这告诉您的应用程序资源已移动到新位置，换句话说；您应该使用新的 URL。

`HttpUrlConnection` 会自动遵循 HTTP 重定向。您可以使用 `setInstanceFollowRedirects()` 方法将其关闭：

```java
connection.setInstanceFollowRedirects(false);
```

# 创建 HTTP POST 请求

POST（和 PUT）请求将数据发送到服务器。对于 POST 请求，您需要打开 `HttpUrlConnection` 的输出模式并设置内容类型：

```java
connection.setRequestMethod("POST");
connection.setRequestProperty("Content-Type", "application/json");
connection.setDoOutput(true);
```

接下来，为了上传数据，这里假设是一个字符串，可以使用以下代码：

```java
DataOutputStream out =
        new DataOutputStream( connection.getOutputStream() );
out.writeBytes(content);
out.flush();
out.close();
```

在网络浏览中，大多数 POST 请求发送表单数据。然而，从 Java 程序中，您更有可能使用 POST 和 PUT 请求上传 JSON 或 XML 数据。一旦上传数据，您的程序应该读取响应，特别是查看请求是否成功。

## 练习 3：使用 POST 请求发送 JSON 数据

在这个练习中，我们将向[`packt.live/2oyJqxB`](https://packt.live/2oyJqxB)测试站点发送一个小 JSON 对象。该站点不会对我们的数据进行任何操作，除了将其回显，以及一些关于请求的**元数据**：

1.  在 IntelliJ 的项目视图中，右键单击`com.packtpub.http`包。选择`New`然后`Java Class`。

1.  将`PostJson`作为 Java 类的名称。

1.  为`PostJson.java`输入以下代码：

    ```java
    PostJson.java
    1  package com.packtpub.http;
    2  
    3  import java.io.BufferedReader;
    4  import java.io.DataOutputStream;
    5  import java.io.IOException;
    6  import java.io.InputStreamReader;
    7  import java.net.HttpURLConnection;
    8  import java.net.MalformedURLException;
    9  import java.net.URL;
    10 
    11 public class PostJson {
    12     public static void main(String[] args) {
    13         /*
    14         {
    15             "animal": "dog",
    16             "name": "biff"
    17         }
    18         */
    https://packt.live/2MYwuZW
    ```

1.  运行这个程序，你应该会看到以下类似的输出：

    ```java
    Code: 200
    {
        "args": {}, 
        "data": "{ \"animal\": \"dog\", \"name\": \"biff\" }", 
        "files": {}, 
        "form": {}, 
        "headers": {
            "Accept": "text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2", 
            "Content-Length": "35", 
            "Content-Type": "application/json", 
            "Host": "httpbin.org", 
            "User-Agent": "Java/11.0.2"
        }, 
        "json": {
            "animal": "dog", 
            "name": "biff"
        }, 
        "origin": "46.244.28.23, 46.244.28.23", 
        "url": "https://httpbin.org/post"
    }
    ```

    注意

    Apache `HttpComponents`库可以帮助简化你使用 HTTP 的工作。更多信息，你可以参考[`packt.live/2BqZbtq`](https://packt.live/2BqZbtq)。

# 解析 HTML 数据

一个 HTML 文档看起来可能如下所示，但通常包含更多内容：

```java
<!doctype html>
<html lang="en">
    <head>
        <title>Example Document</title>
    </head>
    <body>
        <p>A man, a plan, a canal. Panama.</p>
    </body>
</html>
```

HTML 将文档结构化为类似树的格式，如本例中通过缩进来显示。`<head>`元素位于`<html>`元素内部。`<title>`元素位于`<head>`元素内部。一个 HTML 文档可以有多个层次。

注意

大多数网络浏览器都提供了一个查看页面源代码的选项。选择它，你就会看到页面的 HTML。

当你从一个 Java 应用程序运行 GET 请求时，你需要解析返回的 HTML 数据。通常，你将那些数据解析成一个对象树结构。其中一种最方便的方法是使用开源的 jsoup 库。

jsoup 提供了使用 HTTP 连接、下载数据并将这些数据解析成反映页面 HTML 层次结构的元素的方法。

使用 jsoup，第一步是下载一个网页。为此，你可以使用以下代码：

```java
String path = "https://docs.oracle.com/en/java/javase/12/";
Document doc = Jsoup.connect(path).get();
```

此代码下载了官方 Java 12 文档起始页面，其中包含许多指向特定 Java 文档的链接。解析的 HTML 数据被放置到`Document`对象中，该对象包含每个 HTML 元素的`Element`对象。这故意与 Java 的 XML 解析 API 相似，它将 XML 文档解析成一个对象树结构。树中的每个元素可能有子元素。jsoup 提供了一个 API 以类似 Java XML 解析 API 的方式访问这些子元素。

注意

你可以在[`packt.live/2nZbmua`](https://packt.live/2nZbmua)找到关于 jsoup 库的大量有用文档。

在 Java 12 文档页面上，你会看到很多链接。在底层的 HTML 中，许多这些链接如下所示：

```java
<ul class="topics">
  <li>
    <a href="/en/java/javase/12/docs/api/overview-summary.html">API       Documentation </a>
  </li>
</ul>
```

如果我们想要提取 URI 链接（[`packt.live/2VWd1x7`](https://packt.live/2VWd1x7)，在这个例子中）以及描述性文本（API 文档），我们需要遍历到`LI`列表项标签，然后获取 HTML 链接，它被包含在一个`A`标签中，称为锚点。

jsoup 的一个方便功能是，你可以使用类似于 CSS 和 jQuery JavaScript 库提供的选择器语法从 HTML 中选择元素。

要选择所有具有 `topic` CSS 类的 `UL` 元素，你可以使用以下代码：

```java
Elements topics = doc.select("ul.topics");
```

一旦选择了所需的元素，你可以像下面这样遍历每一个：

```java
for (Element topic : topics) {
    for (Element listItem : topic.children()) {
        for (Element link : listItem.children()) {
            String url = link.attr("href");
            String text = link.text();
            System.out.println(url + " " + text);
        }
    }
}
```

此代码从 `UL` 级别开始，向下到 `UL` 标签下的子元素，通常是 `LI`，即列表项元素。Java 文档页面上的每个 `LI` 元素都有一个子元素——即一个带有链接的锚标签。

我们可以然后提取链接本身，它存储在 `href` 属性中。我们还可以提取用于链接的英文描述性文本。

注意

你可以在 [`packt.live/2o54P1e`](https://packt.live/2o54P1e) 和 [`packt.live/33L4akK`](https://packt.live/33L4akK) 找到更多关于 HTML 的信息。

## 练习 4：使用 jsoup 从 HTML 中提取数据

这个练习演示了如何使用 jsoup API 从 HTML 文档中提取链接 URI 和描述性文本。将其用作在项目中解析其他 HTML 文档的示例。

在网络浏览器中转到 [`packt.live/2MO4UOU`](https://packt.live/2MO4UOU)。你可以看到官方的 Java 文档。

我们将提取页面主要内容下标题如 **工具和规范** 等部分的链接。

如果你检查规范部分的 API 文档链接，你会看到文档链接位于一个具有 `topics` CSS 类名的 `UL` 元素中。如前所述，我们可以使用 jsoup API 找到所有具有该 CSS 类名的 `UL` 元素：

1.  在 IntelliJ 中编辑 `build.gradle` 文件。

1.  在依赖项块中添加以下内容：

    ```java
    // jsoup HTML parser from https://jsoup.org/
    implementation 'org.jsoup:jsoup:1.11.3'
    ```

1.  选择在添加新依赖项后出现的弹出窗口中的“导入更改”。

1.  在 IntelliJ 的项目面板中，右键单击 `com.packtpub.http` 包。选择“新建”然后“Java 类”。

1.  将 `JavaDocLinks` 作为 Java 类的名称。

1.  为 `JavaDocLinks.java` 输入以下代码：

```java
JavaDocLinks.java
1  package com.packtpub.http;
2  
3  import org.jsoup.Jsoup;
4  import org.jsoup.nodes.Document;
5  import org.jsoup.nodes.Element;
6  import org.jsoup.select.Elements;
7  
8  import java.io.IOException;
9  
10 public class JavaDocLinks {
11     public static void main(String[] args) {
https://packt.live/2nYnBXS
```

在这个练习中，我们使用了 `jsoup` API 下载一个 HTML 文档。下载后，我们提取了与每个链接相关的链接 URI 和描述性文本。这为 `jsoup` API 提供了一个很好的概述，因此你可以在你的项目中使用它。

# 深入了解 java.net.http 模块

Java 11 在新的 `java.net.http` 模块中添加了一个全新的 `HttpClient` 类。`HttpClient` 类使用现代的 **建造者模式**（也称为流畅式 API）来设置 HTTP 连接。然后它使用响应式流模型来支持同步和异步请求。

注意

你可以参考 *第十六章*，*断言和其他功能接口*，以及 *第十七章*，*使用 Java Flow 进行响应式编程*，了解更多关于 Java 的 Stream API 和响应式流的信息。查看 [`packt.live/32sdPfO`](https://packt.live/32sdPfO) 了解模块中 `java.net.http` 包的概述。

使用建造者模型，你可以配置诸如超时等设置，然后调用 `build()` 方法。你得到的 `HttpClient` 类是不可变的：

```java
HttpClient client = HttpClient.newBuilder()
        .version(HttpClient.Version.HTTP_2)
        .followRedirects(HttpClient.Redirect.NORMAL)
        .connectTimeout(Duration.ofSeconds(30))
        .build();
```

在这个例子中，我们指定以下内容：

+   HTTP 版本 2。

+   客户端应正常遵循重定向，除非重定向是从更安全的 HTTPS 到更不安全的 HTTP。这是`HttpClient.Redirect.NORMAL`的默认行为。

+   连接超时为 30 秒。

`HttpClient`类可用于多个请求。下一步是设置 HTTP 请求：

```java
HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create("http://example.com/"))
        .timeout(Duration.ofSeconds(30))
        .header("Accept", "text/html")
        .build();
```

使用此请求：

+   URL 是`http://example.com/`。

+   读取超时为 30 秒。

+   我们将`Accept`头设置为请求`text/html`内容。

一旦构建完成，在客户端上调用`send()`以同步发送请求或调用`sendAsync()`以异步发送请求。如果您调用`send()`，调用将阻塞，并且您的应用程序将等待数据返回。如果您调用`sendAsync()`，调用将立即返回，并且您的应用程序可以在稍后检查数据是否到达。如果您想在后台线程中处理数据，请使用`sendAsync()`。有关后台线程和如何并发执行任务的更多详细信息，请参阅*第二十二章*，*并发任务*：

```java
HttpResponse<String> response = 
        client.send(request, HttpResponse.BodyHandlers.ofString());
```

在此示例中，请求体处理程序指定我们希望以字符串形式返回内容。

## 练习 5：使用 java.net.http 模块获取 HTML 内容

在此练习中，我们将重新创建*练习 2，创建 GET 请求*以获取网页内容。虽然看起来代码更多，但这并不一定是这样。`java.net.http`模块实际上非常灵活，因为您可以使用**lambda**表达式来处理响应。*第十三章*，*使用 Lambda 表达式的函数式编程*涵盖了 lambda 表达式：

1.  在 IntelliJ 的`项目`面板中，右键单击`com.packtpub.http`包。选择`新建`然后`Java 类`。

1.  将`NetHttpClient`作为 Java 类的名称。

1.  为`NetHttpClient.java`输入以下代码：

```java
NetHttpClient.java
1  package com.packtpub.http;
2  
3  import java.io.IOException;
4  import java.net.URI;
5  import java.net.http.HttpClient;
6  import java.net.http.HttpRequest;
7  import java.net.http.HttpResponse;
8  import java.time.Duration;
9  
10 public class NetHttpClient {
11     public static void main(String[] args) {
12 
13         HttpClient client = HttpClient.newBuilder()
14                 .version(HttpClient.Version.HTTP_2)
15                 .followRedirects(HttpClient.Redirect.NORMAL)
16                 .connectTimeout(Duration.ofSeconds(30))
17                 .build();
https://packt.live/2W33Ivy
```

当您运行此程序时，您应该看到与在*练习 2*，*创建 GET 请求*中创建的`GetRequest`程序相同的输出。

## 活动一：使用 jsoup 库从网络下载文件

通过这个活动，您将下载通过 Packt 提供的 Java 标题。在网页浏览器中访问[`www.packtpub.com/tech/Java`](https://www.packtpub.com/tech/Java)。注意所有可用的 Java 标题。活动内容是编写一个程序来打印所有这些标题：

1.  使用`jsoup`库访问[`packt.live/2J5dlEv`](https://packt.live/2J5dlEv)。

1.  下载 HTML 内容。

1.  查找所有具有 CSS 类`book-block-title`的`DIV`元素，并打印`DIV`内的文本。

1.  当您运行此程序时，您应该看到以下输出：

    ```java
    Hands-On Data Structures & Algorithms in Java 11 [Video]
    Java EE 8 Microservices
    Hands-On Object Oriented Programming with Java 11 [Video]
    Machine Learning in Java - Second Edition
    Java 11 Quick Start
    Object-oriented and Functional Programming with Java 8 [Integrated Course]
    Mastering Microservices with Java 9 - Second Edition
    Design Patterns and Best Practices in Java
    Java Interview Guide : 200+ Interview Questions and Answers [Video]
    Ultimate Java Development and Certification Guide [Video]
    Spring MVC For Beginners : Build Java Web App in 25 Steps [Video]
    Java EE 8 and Angular
    RESTful Java Web Services - Third Edition
    Java EE 8 Application Development
    Mastering Microservices with Java 9 - Second Edition
    ```

    注意

    输出将被截断。

    活动解决方案可在第 557 页找到。

# 摘要

本章介绍了 HTTP 网络，它通常用于在 Java 应用程序中连接到 RESTful 网络服务。HTTP 是一种文本请求-响应协议。客户端向服务器发送请求，然后获取响应。每个 HTTP 请求都有一个方法；例如，您会使用 GET 请求来检索数据，POST 来发送数据，等等。在 Java 应用程序中，您通常会以 JSON 格式发送和接收文本。

`HttpUrlConnection`类提供了发送 HTTP 请求的主要方式。您的代码将数据写入输出流以发送，然后从输入流中读取响应。开源的 jsoup 库提供了一个方便的 API 来检索和解析 HTML 数据。从 Java 11 开始，您可以使用`java.net.http`模块来实现更现代的响应式流方法来处理 HTTP 网络。在下一章中，您将学习关于证书和加密的内容——这两者通常与 HTTP 网络一起使用。
