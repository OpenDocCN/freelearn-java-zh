# 第四章：客户端/服务器开发

在本章中，我们将探讨开发以 HTTP 为主要方向的客户端/服务器应用程序的过程。这是一个重要的协议，它是多种应用程序的主要通信媒介。我们将研究协议、客户端的要求以及不同版本协议对服务器的要求。

具体来说，我们将：

+   检查 HTTP 协议的性质

+   演示低级套接字如何支持协议

+   使用`HttpURLConnect`和`HTTPServer`类创建 HTTP 服务器

+   研究各种开源 Java HTTP 服务器

+   调查各种配置问题以及如何处理 cookie

HTTP 服务器被广泛使用，因此了解 Java 如何支持它们非常重要。

# HTTP 协议结构

HTTP 是一种网络协议，用于在**万维网**（**WWW**）上传递资源。资源通常是**超文本标记语言**（**HTML**）文件，但也包括许多其他文件类型，如图像、音频和视频。用户通常在浏览器中输入 URL 以获取资源。**URL**一词代表**统一资源定位符**，这里强调的是资源。

大多数人使用浏览器在 WWW 上进行通信。浏览器代表客户端应用程序，而 Web 服务器响应客户端请求。这些服务器使用的默认端口是`80`。

![HTTP 协议结构](img/B04915_04_01.jpg)

HTTP 经过多年的发展。HTTP/1.0 起源于 20 世纪 80 年代和 90 年代，首次文档于 1991 年发布。HTTP/1.1 的最新定义是 2014 年 6 月发布的六部分规范。HTTP 2.0 的**请求评论**（**RFC**）于 2015 年 5 月发布。HTTP 是一个不断发展的标准。

以下链接可能对**感兴趣的**读者有用：

| 版本 | 参考 |
| --- | --- |
| HTTP 1.0 | [`www.w3.org/Protocols/HTTP/1.0/spec.html`](http://www.w3.org/Protocols/HTTP/1.0/spec.html) |
| HTTP/1.1 | [`tools.ietf.org/html/rfc2616`](http://tools.ietf.org/html/rfc2616) |
| HTTP/2 | [`en.wikipedia.org/wiki/HTTP/2`](https://en.wikipedia.org/wiki/HTTP/2) |

HTTP 服务器在各种情况下被使用。最常见的用途是在组织内支持向用户传播信息。通常这是由生产质量的服务器支持的，比如由 Apache 软件基金会提供的服务器（[`www.apache.org/foundation/`](http://www.apache.org/foundation/)）或 Gemini（[`www.eclipse.org/gemini/`](http://www.eclipse.org/gemini/)）。

然而，并非所有服务器都需要支持生产服务器所具有的服务水平。它们可以非常小，甚至嵌入在远程设备中，可能会影响设备的变化，而不仅仅是提供信息。

本章将研究 Java 支持的各种网络技术，以解决这些问题。这些技术包括以下内容：

+   HTTP 协议语法概述

+   客户端/服务器的低级套接字支持

+   使用`URLConnection`类

+   使用`HTTPServer`类

+   开源 Java 服务器概述

HTTP 是一个复杂的主题，我们只能浅尝辄止。

### 注意

**机器人**，通常称为**蜘蛛**，是自动跟踪链接的应用程序，经常用于收集供搜索引擎使用的网页。如果您希望开发这样的应用程序，请研究它们的使用和构建方式（[`www.robotstxt.org/`](http://www.robotstxt.org/)）。如果设计不慎，这些类型的应用程序可能会带来破坏性。

# HTTP 消息的性质

让我们检查 HTTP 消息的格式。消息可以是从客户端发送到服务器的请求消息，也可以是从服务器发送到客户端的响应消息。基于对格式的理解，我们将向您展示 Java 如何支持这些消息。HTTP 消息在很大程度上可供人类阅读。请求和响应消息都使用这种结构：

+   指示消息类型的行

+   零个或多个头部行

+   空行

+   包含数据的可选消息正文

以下是 HTTP 请求的示例：

**GET /index HTTP/1.0**

**User-Agent: Mozilla/5.0**

客户端请求消息由初始请求行和零个或多个头部行组成。响应消息由初始响应行（称为**状态行**）、零个或多个头部行和可选消息正文组成。

让我们更详细地检查这些元素。

## 初始请求行格式

请求和响应初始行的格式不同。请求行由三部分组成，用空格分隔：

+   请求方法名称

+   资源的本地路径

+   HTTP 版本

方法名称指的是客户端请求的操作。最常用的方法是**GET**方法，它只是请求返回特定的资源。**POST**命令也很常见，用于插入和更新数据。HTTP/1.0 方法名称列表可在[`www.w3.org/Protocols/HTTP/1.0/spec.html#Methods`](http://www.w3.org/Protocols/HTTP/1.0/spec.html#Methods)找到。HTTP/1.1 方法名称可在[`www.w3.org/Protocols/rfc2616/rfc2616-sec9.html`](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html)找到。方法名称始终以大写字母书写。

本地路径通常指的是所需的资源。它遵循 URL 请求中的主机名。例如，在以下 URL 中，本地路径是**/books/info/packt/faq/index.html**：

**www.packtpub.com/books/info/packt/faq/index.html**

HTTP 版本始终大写，由首字母缩写 HTTP，后跟斜杠，然后是版本号：

**HTTP/x.x**

以下是请求初始行的示例：

**GET /index HTTP/1.0**

响应初始行由三部分组成，用空格分隔，如下所示：

+   HTTP 版本

+   响应状态码

+   描述代码的响应短语

以下行是响应初始行的示例。响应代码反映了结果的状态，并且计算机可以轻松解释。原因短语是为了人类可读。

**HTTP/1.0 404 未找到**

HTTP 版本使用与请求行相同的格式。

以下表格包含了更常用的代码列表。完整列表可以在[`en.wikipedia.org/wiki/List_of_HTTP_status_codes`](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)找到：

| 状态码 | 标准文本 | 含义 |
| --- | --- | --- |
| `200` | **好的** | 这表示请求成功 |
| `301` | **永久移动** | 这表示 URL 已永久移动，链接应该更新 |
| `302` | **找到** | 这表示资源暂时位于其他位置，但仍应使用 URL |
| `307` | **临时重定向** | 这类似于`302`，但不应更改所使用的方法，这可能会发生在`302`中 |
| `308` | **永久重定向** | 这类似于`301`，但不应更改所使用的方法，这可能会发生在`301`中 |
| `400` | **错误的请求** | 这表示请求访问不正确 |
| `401` | **未经授权** | 这表示资源受限，通常是因为登录尝试失败 |
| `403` | **禁止** | 这表示禁止访问所请求的资源 |
| `404` | **未找到** | 这表示资源不再可用 |
| `500` | **内部服务器错误** | 这反映了服务器出现了某种错误 |
| `502` | **错误的网关** | 这表示网关服务器从另一个服务器收到了无效的响应 |
| `503` | **服务不可用** | 这表示服务器不可用 |
| `504` | **网关超时** | 这表示网关服务器未能及时从另一个服务器接收到响应 |

状态码是一个三位数。这个数字的第一位反映了代码的类别：

+   1xx：这代表了信息性消息

+   2xx：这代表了成功

+   3xx：这将客户端重定向到另一个 URL

+   4xx：这代表了客户端错误

+   5xx：这代表了服务器错误

## 头部行

头部行提供有关请求或响应的信息，例如发送者的电子邮件地址和应用程序标识符。头部由一行组成。这行的格式以头部标识符开头，后跟冒号、空格，然后是分配给头部的值。以下头部说明了 Firefox 36.0 使用的`User-Agent`头部。这个头部标识了应用程序是运行在 Windows 平台上的 Firefox 浏览器：

**User-Agent: Mozilla/5.0 (Windows NT 6.3; rv:36.0) Gecko/20100101 Firefox/36.0**

可以在[`en.wikipedia.org/wiki/List_of_HTTP_header_fields`](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields)找到头部字段和描述的列表。代理字符串的列表可以在[`useragentstring.com/pages/useragentstring.php`](http://useragentstring.com/pages/useragentstring.php)找到。

HTTP 1.0 定义了 16 个头部（[`www.w3.org/Protocols/HTTP/1.0/spec.html#HeaderFields`](http://www.w3.org/Protocols/HTTP/1.0/spec.html#HeaderFields)），而 HTTP 1.1 有 47 个头部（[`tools.ietf.org/html/rfc2616#section-14`](http://tools.ietf.org/html/rfc2616#section-14)）。它的`Host`头部是必需的。

头部在出现问题时有助于进行故障排除。最好包括请求的`From`和`User-Agent`头部，以便服务器能够更好地响应请求。

## 消息正文

这是构成消息的数据。虽然通常会包括消息正文，但它是可选的，并且对于一些消息是不需要的。当包括正文时，会包括`Content-Type`和`Content-Length`头部，以提供有关正文的更多信息。

例如，以下头部可以用于消息正文：

**Content-type: text/html**

**Content-length: 105**

消息正文可能如下所示：

**<html><h1>HTTPServer 主页.... </h1><br><b>欢迎来到新的改进的 Web 服务器！</b><BR></html>**

## 客户端/服务器交互示例

以下交互是一个简单的示例，展示了客户端发送请求，服务器做出响应。客户端请求消息使用`GET`方法对`\index`路径进行请求：

**GET /index HTTP/1.0**

**User-Agent: Mozilla/5.0**

服务器将以以下消息作出响应，假设它能够处理请求。使用了`Server`、`Content-Type`和`Content-Length`头部。一个空行分隔头部和 HTML 消息正文：

**HTTP/1.0 200 OK**

**Server: WebServer**

**Content-Type: text/html**

**Content-Length: 86**

**<html><h1>WebServer 主页.... </h1><br><b>欢迎来到我的 Web 服务器！</b><BR></html>**

其他头部行可以包括在内。

# Java 套接字支持 HTTP 客户端/服务器应用程序

HTTP 客户端将与 HTTP 服务器建立连接。客户端将向服务器发送一个请求消息。服务器将返回一个响应消息，通常是一个 HTML 文档。在早期的 HTTP 版本中，一旦响应被发送，服务器将终止连接。这有时被称为无状态协议，因为连接不会被维护。

使用 HTTP/1.1，可以保持持久连接。这通过消除在服务器和客户端之间传输多个数据片段时打开和关闭连接的需要来提高性能。

我们将专注于创建一个 HTTP 服务器和一个 HTTP 客户端。虽然浏览器通常作为 HTTP 客户端，但其他应用程序也可以访问 Web 服务器。此外，它有助于说明 HTTP 请求的性质。我们的服务器将支持 HTTP/1.0 规范的一个子集。

## 构建一个简单的 HTTP 服务器

我们将使用一个名为`WebServer`的类来支持 HTTP/1.0 协议。服务器将使用`ClientHandler`类来处理客户端。服务器将被限制为仅处理 GET 请求。然而，这将足以说明所需的基本服务器元素。支持其他方法可以很容易地添加。

接下来显示了`WebServer`的定义。`ServerSocket`类是服务器的基础。其`accept`方法将阻塞，直到有请求发出。当这发生时，将启动一个基于`ClientHandler`类的新线程：

```java
public class WebServer {

    public WebServer() {
        System.out.println("Webserver Started");
        try (ServerSocket serverSocket = new ServerSocket(80)) {
            while (true) {
                System.out.println("Waiting for client request");
                Socket remote = serverSocket.accept();
                System.out.println("Connection made");
                new Thread(new ClientHandler(remote)).start();
            }
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    public static void main(String args[]) {
        new WebServer();
    }
}
```

Mac 用户在使用端口`80`时可能会遇到错误。请改用端口`3000`或`8080`。线程是在进程内并发执行代码序列。在 Java 中，可以使用`Thread`类来创建线程。构造函数的参数是实现`Runnable`接口的对象。该接口由一个方法组成：`run`。当使用`start`方法启动线程时，将为新线程创建一个单独的程序堆栈，并在该堆栈上执行`run`方法。当`run`方法终止时，线程也终止。接下来显示的`ClientHandler`类实现了`Runnable`接口。它的构造函数接收表示客户端的套接字。当线程启动时，将执行`run`方法。该方法显示启动和终止消息。实际工作是在`handleRequest`方法中执行的：

```java
public class ClientHandler implements Runnable {

    private final Socket socket;

    public ClientHandler(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        System.out.println("\nClientHandler Started for " + 
            this.socket);
        handleRequest(this.socket);
        System.out.println("ClientHandler Terminated for " 
            + this.socket + "\n");
    }

}
```

`handleRequest`方法使用输入和输出流与服务器进行通信。此外，它确定了所发出的请求，然后处理该请求。

在接下来的代码中，创建了输入和输出流，并读取了请求的第一行。使用`StringTokenizer`类来标记这一行。当调用`nextToken`方法时，它会返回该行的第一个单词，这应该对应于一个 HTTP 方法：

```java
    public void handleRequest(Socket socket) {
        try (BufferedReader in = new BufferedReader(
                new InputStreamReader(socket.getInputStream()));) {
            String headerLine = in.readLine();
            StringTokenizer tokenizer = 
                new StringTokenizer(headerLine);
            String httpMethod = tokenizer.nextToken();
            ...
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

分词器是将文本分割成一系列标记的过程。通常，这些标记是简单的单词。`StringTokenizer`类的构造函数接收要标记化的文本。`nextToken`方法将返回下一个可用的标记。

接下来的代码序列处理了`GET`方法。在服务器端显示一条消息，指示正在处理`GET`方法。该服务器将返回一个简单的 HTML 页面。该页面是使用`StringBuilder`类构建的，其中使用了`append`方法以流畅的方式。然后调用`sendResponse`方法来实际发送响应。如果请求了其他方法，则返回`405`状态码：

```java
    if (httpMethod.equals("GET")) {
        System.out.println("Get method processed");
        String httpQueryString = tokenizer.nextToken();
        StringBuilder responseBuffer = new StringBuilder();
        responseBuffer
            .append("<html><h1>WebServer Home Page.... </h1><br>")
            .append("<b>Welcome to my web server!</b><BR>")
            .append("</html>");
        sendResponse(socket, 200, responseBuffer.toString());
    } else {
        System.out.println("The HTTP method is not recognized");
        sendResponse(socket, 405, "Method Not Allowed");
    }
```

如果我们想处理其他方法，那么可以添加一系列的 else-if 子句。要进一步处理`GET`方法，我们需要解析初始请求行的其余部分。以下语句将给我们一个可以处理的字符串：

```java
    String httpQueryString = tokenizer.nextToken();
```

上述陈述对于本例不需要，不应包含在代码中。它只是提供了进一步处理 HTTP 查询的一种可能方式。

一旦我们创建了响应，我们将使用`sendResponse`方法将其发送给客户端，如下所示。该方法接收套接字、状态码和响应字符串。然后创建一个输出流：

```java
    public void sendResponse(Socket socket, 
            int statusCode, String responseString) {
        String statusLine;
        String serverHeader = "Server: WebServer\r\n";
        String contentTypeHeader = "Content-Type: text/html\r\n";

        try (DataOutputStream out = 
                new DataOutputStream(socket.getOutputStream());) {
            ...
            out.close();
        } catch (IOException ex) {
            // Handle exception
        }
    }
```

如果状态码是`200`，那么将返回一个简单的 HTML 页面。如果状态码是`405`，那么将返回一个单一的状态码行。否则，将发送一个`404`响应。由于我们使用`DataOutputStream`类进行写入，因此我们使用它的`writeBytes`方法来处理字符串：

```java
    if (statusCode == 200) {
        statusLine = "HTTP/1.0 200 OK" + "\r\n";
        String contentLengthHeader = "Content-Length: " 
            + responseString.length() + "\r\n";

        out.writeBytes(statusLine);
        out.writeBytes(serverHeader);
        out.writeBytes(contentTypeHeader);
        out.writeBytes(contentLengthHeader);
        out.writeBytes("\r\n");
        out.writeBytes(responseString);
    } else if (statusCode == 405) {
        statusLine = "HTTP/1.0 405 Method Not Allowed" + "\r\n";
        out.writeBytes(statusLine);
        out.writeBytes("\r\n");
    } else {
        statusLine = "HTTP/1.0 404 Not Found" + "\r\n";
        out.writeBytes(statusLine);
        out.writeBytes("\r\n");
    }
```

服务器启动后，将显示以下内容：

**连接已建立**

**等待客户端请求**

当客户端发出`GET`请求时，将显示类似以下的输出：

**ClientHandler Started for Socket[addr=/127.0.0.1,port=50573,localport=80]**

**GET 方法已处理**

**ClientHandler Terminated for Socket[addr=/127.0.0.1,port=50573,localport=80]**

有了一个简单的服务器，让我们看看如何构建一个 HTTP 客户端应用程序。

## 构建一个简单的 HTTP 客户端

我们将使用以下`HTTPClient`类来访问我们的 HTTP 服务器。在它的构造函数中，创建一个连接到服务器的套接字。`Socket`类的`getInputStream`和`getOutputStream`分别返回套接字的输入和输出流。调用`sendGet`方法，发送请求到服务器。`getResponse`方法返回响应，然后显示出来：

```java
public class HTTPClient {

    public HTTPClient() {
        System.out.println("HTTP Client Started");
        try {
            InetAddress serverInetAddress = 
               InetAddress.getByName("127.0.0.1");
            Socket connection = new Socket(serverInetAddress, 80);

            try (OutputStream out = connection.getOutputStream();
                 BufferedReader in = 
                     new BufferedReader(new 
                         InputStreamReader(
                             connection.getInputStream()))) {
                sendGet(out);
                System.out.println(getResponse(in));
            }
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    ...

    public static void main(String[] args) {
        new HTTPClient();
    }
}
```

`sendGet`方法如下，它使用简单路径发送`GET`方法请求。然后是`User-Agent`头。我们使用`OutputStream`类的实例和`write`方法。`write`方法需要一个字节数组。`String`类的`getBytes`方法返回这个字节数组：

```java
    private void sendGet(OutputStream out) {
        try {
            out.write("GET /default\r\n".getBytes());
            out.write("User-Agent: Mozilla/5.0\r\n".getBytes());
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }
```

`getResponse`方法如下，并传递一个`BufferedReader`实例来从服务器获取响应。它返回一个使用`StringBuilder`类创建的字符串：

```java
    private String getResponse(BufferedReader in) {
        try {
            String inputLine;
            StringBuilder response = new StringBuilder();
            while ((inputLine = in.readLine()) != null) {
                response.append(inputLine).append("\n");
            }
            return response.toString();
        } catch (IOException ex) {
            ex.printStackTrace();
        }
        return "";
    }
```

当客户端应用程序执行时，我们得到以下反映服务器响应的输出：

**HTTP 客户端已启动**

**HTTP/1.0 200 OK**

**服务器：WebServer**

**内容类型：text/html**

**内容长度：86**

**<html><h1>WebServer 首页.... </h1><br><b>欢迎来到我的网络服务器！</b><BR></html>**

如果我们从浏览器使用相同的请求，我们将得到以下结果：

![构建一个简单的 HTTP 客户端](img/B04915_04_02.jpg)

这些客户端和服务器应用程序可以进一步增强。然而，我们可以使用`HttpURLConnection`类来实现类似的结果。

# 使用标准 Java 类进行客户端/服务器开发

具体来说，我们将使用`HttpURLConnection`和`HTTPServer`类来实现客户端和服务器应用程序。这些类支持客户端和服务器所需的大部分功能。使用这些类将避免编写低级代码来实现 HTTP 功能。低级代码是指非专门化的类，如`Socket`类。更高级和更专门化的类，如`HttpURLConnection`和`HTTPServer`类，补充并为专门化的操作提供额外支持。

`HttpURLConnection`类是从`HttpConnection`类派生的。这个基类有许多方法，不直接涉及 HTTP 协议。

## 使用 HttpURLConnection 类

`HttpURLConnection`类提供了一种便捷的访问网站的技术。使用这个类，我们可以连接到一个站点，发出请求，并访问响应头和响应消息。

我们将使用以下定义的`HttpURLConnectionExample`类。`sendGet`方法支持向服务器发送`GET`方法请求。`HttpURLConnectionExample`类支持其他 HTTP 方法。在这个例子中，我们只使用`GET`方法：

```java
public class HttpURLConnectionExample {

    public static void main(String[] args) throws Exception {
        HttpURLConnectionExample http = 
            new HttpURLConnectionExample();
        http.sendGet();
    }

}
```

接下来显示了`sendGet`方法的实现。使用一个 Google 查询（[`www.google.com/search?q=java+sdk&ie=utf-8&oe=utf-8`](http://www.google.com/search?q=java+sdk&ie=utf-8&oe=utf-8)）来说明我们搜索"java sdk"的过程。查询的后半部分`&ie=utf-8&oe=utf-8`是由 Google 搜索引擎附加到查询的附加信息。`openConnection`方法将连接到 Google 服务器：

```java
    private void sendGet() throws Exception {
        String query = 
      "http://www.google.com/search?q=java+sdk&ie=utf-8&oe=utf-8";
        URL url = new URL(query);
        HttpURLConnection connection = 
            (HttpURLConnection) url.openConnection();
        ...
    }
```

使用这个连接，`setRequestMethod`和`setRequestProperty`方法分别设置请求方法和用户代理：

```java
        connection.setRequestMethod("GET");
        connection.setRequestProperty("User-Agent", 
            "Mozilla/5.0");
```

检索响应代码，如果成功，`getResponse`方法将检索响应，然后显示如下：

```java
        int responseCode = connection.getResponseCode();
        System.out.println("Response Code: " + responseCode);
        if (responseCode == 200) {
            String response = getResponse(connection);
            System.out.println("response: " + 
                response.toString());
        } else {
            System.out.println("Bad Response Code: " + 
                responseCode);
        }
```

接下来显示了`getResponse`方法。`HttpURLConnection`类的`getInputStream`方法返回一个输入流，用于创建`BufferedReader`类的实例。使用这个读取器和`StringBuilder`实例来创建并返回一个字符串：

```java
    private String getResponse(HttpURLConnection connection) {
        try (BufferedReader br = new BufferedReader(
                new InputStreamReader(
                    connection.getInputStream()));) {
            String inputLine;
            StringBuilder response = new StringBuilder();
            while ((inputLine = br.readLine()) != null) {
                response.append(inputLine);
            }
            br.close();
            return response.toString();
        } catch (IOException ex) {
            // Handle exceptions
        }
        return "";
    }
```

当此程序执行时，您将获得以下输出。由于输出的长度，它已被截断：

**发送 Http GET 请求**

**响应代码：200**

**响应：<!doctype html><html itemscope="" ...**

如果我们在浏览器中使用这个查询，我们将得到类似以下的输出：

![使用 HttpURLConnection 类](img/B04915_04_03.jpg)

如何使用`URLConnection`类处理 HTTP 请求的非常有趣的讨论可以在[`stackoverflow.com/questions/2793150/using-java-net-urlconnection-to-fire-and-handle-http-requests`](http://stackoverflow.com/questions/2793150/using-java-net-urlconnection-to-fire-and-handle-http-requests)找到。

### URL 编码

当形成 URL 时，需要使用特定的 URL 格式。该格式的一些字符是保留字符，其他字符是未保留字符。保留字符具有特殊含义，例如斜杠，用于分隔 URL 的各个部分。未保留字符没有任何特殊含义。

当需要在非保留上下文中使用保留字符时，使用 URL 编码，也称为百分号编码，用特殊字符序列表示这些字符。有关此过程的更多信息，请参阅[`en.wikipedia.org/wiki/Percent-encoding`](https://en.wikipedia.org/wiki/Percent-encoding)。

在 Java 中，我们可以使用`URLEncoder`类执行 URL 编码。具体来说，`URLEncoder`类有一个`encode`方法，用于转换符合`application/x-www-form-url`编码 MIME 格式的字符串。

这个方法是重载的。单参数方法已被弃用。两参数方法接受要转换的字符串和指定字符编码方案的字符串。对于 HTTP 消息，请使用 UTF-8 格式。

以前，我们使用以下字符串创建一个新的 URL 实例：

```java
    String query = 
      "http://www.google.com/search?q=java+sdk&ie=utf-8&oe=utf-8";
```

这个字符串实际上是由浏览器格式化的。以下代码演示了如何使用`encode`方法来实现类似的结果，而不是使用浏览器：

```java
    String urlQuery = "http://www.google.com/search?q=";
    String userQuery = "java sdk";
    String urlEncoded = urlQuery + URLEncoder.encode(
        userQuery, "UTF-8");
```

这将产生字符串：`http://www.google.com/search?q=java+sd`。您可以看到空格已经被转换为`+`符号用于这个 URL。原始查询的后半部分`&ie=utf-8&oe=utf-8`没有包含在我们的 URL 编码字符串中。

如果需要，`URLDecoder`类可用于解码 URL 编码的字符串。有关 URL 编码的全面讨论，请参阅：[`blog.lunatech.com/2009/02/03/what-every-web-developer-must-know-about-url-encoding`](http://blog.lunatech.com/2009/02/03/what-every-web-developer-must-know-about-url-encoding)。

## 使用 HTTPServer 类

`HTTPServer`类位于`com.sun.net.httpserver`包中。它提供了一组强大的功能，支持简单的 HTTP 服务器。我们以前的服务器需要手动执行的许多任务在这个服务器上都得到了简化。客户端和服务器之间的交互称为交换。

这些支持类和接口是`com.sun.net.httpserver`包的成员。它们通常包含在大多数集成开发环境中。API 文档可以在[`docs.oracle.com/javase/8/docs/jre/api/net/httpserver/spec/index.html?com/sun/net/httpserver/package-summary.html`](http://docs.oracle.com/javase/8/docs/jre/api/net/httpserver/spec/index.html?com/sun/net/httpserver/package-summary.html)找到。

这个包包括许多类。我们将使用的主要类包括：

| 类/接口 | 目的 |
| --- | --- |
| `HttpServer` | 此类支持 HTTP 服务器的基本功能 |
| `HttpExchange` | 此类封装了与客户端/服务器交换相关的请求和响应 |
| `HttpHandler` | 此类定义了用于处理特定交换的 handle 方法 |
| `HttpContext` | 此类将 URI 路径映射到`HttpHandler`实例 |
| `Filter` | 此类支持请求的预处理和后处理 |

服务器使用一个派生自`HttpHandler`的类来处理客户端请求。例如，一个处理程序可以处理基本网页的请求，而另一个处理程序可以处理与服务相关的请求。

`HttpExchange`类支持客户端和服务器之间交换的生命周期活动。它拥有许多方法，提供对请求和响应信息的访问。这些方法按照通常使用的顺序列在下表中。并非所有请求都需要使用所有方法：

| 方法 | 目的 |
| --- | --- |
| `getRequestMethod` | 此方法返回所请求的 HTTP 方法 |
| `getRequestHeaders` | 此方法返回请求头 |
| `getRequestBody` | 此方法返回用于请求正文的`InputStream`实例 |
| `getResponseHeaders` | 此方法返回响应头，除了内容长度 |
| `sendResponseHeaders` | 此方法发送响应头 |
| `getResponseBody` | 此方法返回用于发送响应的`OutputStream`实例 |

当输入和输出流关闭时，交换就会关闭。在调用`getResponseBody`方法之前必须使用`sendResponseHeaders`方法。

### 注意

这个类的初始版本性能不是很好。然而，更新的版本性能更好。此外，过滤器功能可以帮助处理交换。

可以放心使用`com.sun.*`类，但如果在不同的 JRE 中使用`sun.*`类可能会出现问题。`HTTPServer`类完全支持 HTTP/1.0，但仅提供了对 HTTP/1.1 的部分支持。

### 实现一个简单的 HTTPServer 类

接下来的类实现了使用`HTTPServer`类的简单服务器。使用本地主机和端口`80`（在 Mac 上为`3000`或`8080`）创建了`HttpServer`类的实例。`createContext`方法将`/index`路径与`IndexHandler`类的实例关联起来。这个处理程序将处理请求。`start`方法启动服务器。服务器将继续运行，处理多个请求，直到手动停止为止。

```java
public class MyHTTPServer {

    public static void main(String[] args) throws Exception {
        System.out.println("MyHTTPServer Started");
        HttpServer server = HttpServer.create(
            new InetSocketAddress(80), 0);
        server.createContext("/index", new IndexHandler());
        server.start();
    }

}
```

当`createContext`方法将字符串表示的路径与处理程序匹配时，它使用特定的匹配过程。此过程的详细信息在`HTTPServer`类文档的*将请求 URI 映射到 HttpContext 路径*部分中解释，该文档位于[`docs.oracle.com/javase/8/docs/jre/api/net/httpserver/spec/com/sun/net/httpserver/HttpServer.html`](http://docs.oracle.com/javase/8/docs/jre/api/net/httpserver/spec/com/sun/net/httpserver/HttpServer.html)。

接下来声明了`IndexHandler`类。它通过覆盖`handle`方法来实现`HttpHandler`接口。`handle`方法传递了一个`HttpExchange`实例，我们可以用它来处理请求。

在此方法中，我们执行以下操作：

+   显示客户端的地址

+   发送状态码为`200`的请求返回

+   向客户端发送响应

`sendResponseHeaders`方法将发送状态码为`200`的初始响应行和内容长度的头部。`getResponseBody`方法返回用于发送消息正文的输出流。然后关闭流，终止交换：

```java
    static class IndexHandler implements HttpHandler {

        @Override
        public void handle(HttpExchange exchange) 
                throws IOException {
            System.out.println(exchange.getRemoteAddress());
            String response = getResponse();
            exchange.sendResponseHeaders(200, response.length());
            OutputStream out = exchange.getResponseBody();
            out.write(response.toString().getBytes());
            out.close();
        }
    }
```

`sendResponseHeaders`方法使用两个参数。第一个是响应代码，第二个控制消息主体的传输，如下表所述：

| 值 | 意义 |
| --- | --- |
| 大于零 | 这是消息的长度。服务器必须发送这么多字节。 |
| 零 | 这用于分块传输，其中发送任意数量的字节。 |
| -1 | 这是当没有响应主体被发送时。 |

`getResponse`方法使用`StringBuilder`类构造字符串：

```java
    public String getResponse() {
        StringBuilder responseBuffer = new StringBuilder();
        responseBuffer
            .append(
                "<html><h1>HTTPServer Home Page.... </h1><br>")
            .append("<b>Welcome to the new and improved web " 
                    + "server!</b><BR>")
            .append("</html>");
        return responseBuffer.toString();
    }
```

当服务器启动时，将显示以下输出：

**MyHTTPServer 已启动**

如果我们在浏览器中输入 URL `http://127.0.0.1/index`，浏览器将显示类似于*构建简单 HTTP 客户端*部分中图像的页面。

服务器将为每个请求显示以下内容：

**/127.0.0.1:50273**

这个类在处理客户端请求方面非常重要。在这里，我们将使用一个名为`DetailHandler`的不同处理程序来说明该类的几种方法：

```java
    static class DetailHandler implements HttpHandler {

        @Override
        public void handle(HttpExchange exchange) 
                throws IOException {
            ...
        }
    }
```

要使用此处理程序，请在`MyHTTPServer`中用此语句替换`createContext`方法，并在`MyHTTPServer`中调用。

```java
        server.createContext("/index", new DetailHandler());
```

让我们首先检查`getRequestHeaders`方法的使用，它返回`Headers`类的一个实例。这将允许我们显示客户端发送的每个请求头，并根据需要执行其他处理。

将以下代码添加到`handle`方法中。`keyset`方法返回每个头的键/值对的`Set`。在 for-each 语句中，`Set`接口的`get`方法返回每个头的值列表。此列表用于显示头部：

```java
    Headers requestHeaders = exchange.getRequestHeaders();
    Set<String> keySet = requestHeaders.keySet();
    for (String key : keySet) {
        List values = requestHeaders.get(key);
        String header = key + " = " + values.toString() + "\n";
        System.out.print(header);
    }
```

使用 Firefox 浏览器中的先前 URL（`http://127.0.0.1/index`），我们得到以下输出：

**接受编码 = [gzip, deflate]**

**接受 = [text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8]**

**连接 = [保持活动]**

**主机 = [127.0.0.1]**

**用户代理 = [Mozilla/5.0 (Windows NT 10.0; WOW64; rv:40.0) Gecko/20100101 Firefox/40.0]**

**接受语言 = [en-US,en;q=0.5]**

**缓存控制 = [max-age=0]**

使用不同的浏览器可能返回不同的一组请求头。`getRequestMethod`方法返回请求方法的名称，如下所示：

```java
    String requestMethod = exchange.getRequestMethod();
```

我们可以使用这个来区分客户端请求。

一些请求方法将在请求中传递消息主体。`getRequestBody`方法将返回一个`InputStream`实例以访问此主体。

以下代码说明了如何获取和显示消息主体：

```java
    InputStream in = exchange.getRequestBody();
    if (in != null) {
        try (BufferedReader br = new BufferedReader(
                new InputStreamReader(in));) {
            String inputLine;
            StringBuilder response = new StringBuilder();
            while ((inputLine = br.readLine()) != null) {
                response.append(inputLine);
            }
            br.close();
            System.out.println(inputLine);
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    } else {
        System.out.println("Request body is empty");
    }
```

由于我们的请求没有主体，因此不会显示任何内容。

### 管理响应头

服务器可以使用`sendResponseHeaders`方法发送响应头。但是，这些头需要使用`getResponseHeaders`方法和`set`方法的组合来创建。

在下一个代码序列中，`getResponseHeaders`方法将返回`Header`类的一个实例：

```java
    Headers responseHeaders = exchange.getResponseHeaders();
```

我们使用`getResponse`方法获取我们的响应。我们需要这个来计算内容长度。然后使用`set`方法创建**Content-Type**和**Server**头：

```java
    String responseMessage = HTTPServerHelper.getResponse();
    responseHeaders.set("Content-Type", "text/html");
    responseHeaders.set("Server", "MyHTTPServer/1.0");
```

头部使用先前描述的`sendResponseHeaders`方法发送，如下所示：

```java
    exchange.sendResponseHeaders(200, responseMessage.getBytes().length);
```

可以使用以下代码序列显示这些响应头。这执行与我们用于显示请求头的 for-each 语句相同的功能。但是，此实现使用 Java 8 的 Stream 类和两个 lambda 表达式：

```java
    Set<String> responseHeadersKeySet = responseHeaders.keySet();
    responseHeadersKeySet
            .stream()
            .map((key) -> {
                List values = responseHeaders.get(key);
                String header = key + " = " + 
                    values.toString() + "\n";
                return header;
            })
            .forEach((header) -> {
                System.out.print(header);
            });
```

此实现使用流。`stream`方法返回集合中找到的键。`map`方法使用每个键来处理查找与键关联的值列表。然后将列表转换为字符串。`forEach`方法将显示这些字符串中的每一个。

`HTTPServer`及其相关类提供了一种简单但方便的实现 HTTP 服务器的技术。还提供了使用`HttpsServer`类进行安全通信的支持，该类在第八章*网络安全*中讨论。

# 开源 Java HTTP 服务器

虽然我们可以使用本章讨论的任何技术来开发 Web 服务器，但另一个选择是使用许多开源的基于 Java 的 HTTP 服务器之一。这些服务器通常提供许多功能，包括：

+   完全符合 HTTP 标准

+   支持日志记录和监控

+   虚拟主机的处理

+   性能调优能力

+   可扩展的

+   分块数据传输

+   可配置性

+   支持 NIO（Grizzly）

利用这些系统可以节省大量时间和精力，否则将用于构建自定义服务器。一些基于 Java 的服务器的部分列表包括以下内容：

+   Jakarta Tomcat（[`tomcat.apache.org/`](http://tomcat.apache.org/)）

+   Jetty（[`www.eclipse.org/jetty/`](http://www.eclipse.org/jetty/)）

+   JLHTTP（[`www.freeutils.net/source/jlhttp/`](http://www.freeutils.net/source/jlhttp/)）

+   GlassFish（[`glassfish.java.net/`](https://glassfish.java.net/)）

+   Grizzly（[`grizzly.java.net/`](https://grizzly.java.net/)）

+   Simple（[`www.simpleframework.org/`](http://www.simpleframework.org/)）

在[`java-source.net/open-source/web-servers`](http://java-source.net/open-source/web-servers)找到一个开源 Java 服务器列表。

在更高的层次上，Java EE 经常用于支持 Web 服务器。虽然这个版本多年来已经发展，但 servlets 仍然是处理 Web 请求的基础。Servlet 是一个隐藏了很多关于请求和响应的低级处理细节的 Java 应用程序。这使得开发人员可以专注于处理请求。

Servlets 存放在容器中，提供对任务的支持，如数据库访问，性能管理和安全性提供。下面显示了一个简单的 servlet，让您了解它们的结构。

`doGet`和`doPost`方法分别处理`GET`和`POST`类型的消息。然而，由于这两种 HTTP 消息之间的差异被隐藏了，只需要一个。`HttpServletRequest`类表示 HTTP 请求，`HttpServletResponse`类表示响应。这些类提供对消息的访问。例如，`getWriter`方法返回一个`PrintWriter`类实例，允许我们以更清晰的方式编写 HTML 响应：

```java
public class ServletExample extends HttpServlet { 

    @Override
    public void doGet(HttpServletRequest request,
            HttpServletResponse response)
                throws ServletException, IOException {
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        out.println("<h1>" + "Message to be sent" + "</h1>");
    }

    @Override
    public void doPost(HttpServletRequest request, 
            HttpServletResponse response)
                throws IOException, ServletException {
        doGet(request, response);
    }

}
```

Servlets 通常使用 Java EE SDK 开发。前面的例子如果不使用这个 API 开发，将无法正确编译。

许多技术已经发展并隐藏了 servlets。多年来，这包括了**JavaServer Pages**（**JSP**）和**JavaServer Faces**（**JSF**），它们基本上消除了直接编写 servlets 的需要。

有许多 Java 的 Web 服务器。这些中的一些比较可以在[`en.wikipedia.org/wiki/Comparison_of_application_servers#Java`](https://en.wikipedia.org/wiki/Comparison_of_application_servers#Java)找到。

# 服务器配置

服务器的配置取决于用于构建它的技术。在这里，我们将重点放在`URLConnection`类的配置上。这个类有一些受保护的字段，控制连接的行为。这些字段可以使用相应的 get 和 set 方法来访问。

一个字段处理用户交互。当设置为`true`时，允许用户参与交互，如响应身份验证对话框。连接可用于输入和/或输出。连接可以配置为禁止输入或输出。

当数据在客户端和服务器之间传输时，它可能会被缓存。`UseCaches`变量确定是否忽略缓存。如果设置为`true`，则缓存将适当地使用。如果为`false`，则不进行缓存。

`ifModifiedSince`变量控制是否发生对象的检索。它是一个长整型值，表示自纪元（1970 年 1 月 1 日 GMT）以来的毫秒数。如果对象的修改时间比该时间更近，则会被获取。

以下表总结了使用`URLConnection`类建立连接的方法。每个方法都有相应的`GET`方法：

| 方法 | 默认 | 目的 |
| --- | --- | --- |
| `setAllowUserInteraction` | NA | 此方法控制用户交互 |
| `setDoInput` | `true` | 如果其参数设置为`true`，则允许输入 |
| `setDoInput` | `true` | 如果参数设置为`true`，则允许输出 |
| `setIfModifiedSince` | NA | 这将设置`ifModifiedSince`变量 |
| `setUseCaches` | `true` | 这将设置`UseCaches`变量 |

更复杂的服务器，如 Tomcat，有更多选项来控制它的配置。

当应用程序部署时，在`deployment.properties`文件中有许多配置选项。其中许多选项是低级别的，与 JRE 相关。有关这些选项的描述可以在[`docs.oracle.com/javase/8/docs/technotes/guides/deploy/properties.html`](https://docs.oracle.com/javase/8/docs/technotes/guides/deploy/properties.html)找到。*21.2.4 网络*部分讨论了网络选项，而*21.2.5 缓存和可选包存储库*部分涉及缓存的配置。

**HTTP 代理**是一个充当客户端和服务器之间中介的服务器。代理经常用于管理网络，监视流量和改善网络性能。

通常，我们不关心代理的使用或配置。但是，如果需要配置代理，我们可以使用 JVM 命令行或在代码中使用`System`类的`getProperties`方法来控制它。我们可以控制使用的代理并在需要时指定用户和密码来访问它。关于这些功能的简要讨论可以在[`viralpatel.net/blogs/http-proxy-setting-java-setting-proxy-java/`](http://viralpatel.net/blogs/http-proxy-setting-java-setting-proxy-java/)找到。

# 处理 cookie

一个 cookie 是一个包含键/值对的字符串，表示服务器感兴趣的信息，如用户偏好。它从服务器发送到浏览器。浏览器应该将 cookie 保存到文件中，以便以后使用。

一个 cookie 是一个由名称后跟等号然后是一个值组成的字符串。以下是一个可能的 cookie：

**userID=Cookie Monster**

一个 cookie 可以有多个值。这些值将用分号和空格分隔。

我们将使用`HTTPServer`类和`HttpURLConnection`类来演示处理 cookie。在`MyHTTPServer`类服务器的处理程序类的`handle`方法中，在其他标头之后添加以下代码：

```java
    responseHeaders.set("Set-cookie", "userID=Cookie Monster");
```

当服务器响应时，它将发送该 cookie。

在`HttpURLConnectionExample`类的`getResponse`方法中，在其 try 块的开头添加以下代码。构建一个包含 cookie 文本的字符串。使用多个`substring`和`indexOf`方法来提取 cookie 的名称，然后提取其值：

```java
    Map<String, List<String>> requestHeaders = 
        connection.getHeaderFields();
    Set<String> keySet = requestHeaders.keySet();
    for (String key : keySet) {
        if ("Set-cookie".equals(key)) {
            List values = requestHeaders.get(key);
            String cookie = key + " = " + 
                values.toString() + "\n";
            String cookieName = 
                cookie.substring(0, cookie.indexOf("="));
            String cookieValue = cookie.substring(
                cookie.indexOf("=")+ 1, cookie.length());
            System.out.println(cookieName + ":" + cookieValue);
        }
    }
```

当服务器发送响应时，它将包括 cookie。然后客户端将接收到 cookie。在服务器和客户端中，您应该看到以下输出显示 cookie：

**Set-cookie : [userID=Cookie Monster]**

前面的示例处理了简单的单值 cookie。处理多个值的代码留作读者的练习。

# 总结

在本章中，我们研究了可以用于开发 HTTP 客户端/服务器应用程序的各种 Java 方法。使用 HTTP 进行通信是一种常见的做法。了解 Java 如何支持这一过程是一项宝贵的技能。

我们从 HTTP 消息的概述开始。我们检查了初始请求和响应行的格式。还检查了头部行，用于传递有关消息的信息。HTTP 消息中可能会出现可选的消息主体。这在响应中更常见，其中主体通常是 HTML 文档。

我们演示了如何使用简单套接字开发客户端/服务器。虽然可能，但这种方法需要大量工作来开发一个完全功能的 HTTP 服务器。这一讨论之后是使用`HTTPServer`和`HttpURLConnection`类来支持服务器和客户端，这些类的使用使开发过程变得更加容易。

有许多基于 Java 的开源 HTTP 服务器可用。这些可能是某些环境的可行候选。还讨论了更复杂的 Web 服务器，比如 Apache Tomcat。它们与 Servlet 一起工作，并且将许多低级 HTTP 细节隐藏在开发者之外。

我们总结了本章，简要讨论了服务器配置问题以及服务器和客户端如何创建和使用 cookie。

虽然客户端/服务器架构非常常见，但对等架构是在网络上共享信息的另一种选择。我们将在下一章深入探讨这个主题。
