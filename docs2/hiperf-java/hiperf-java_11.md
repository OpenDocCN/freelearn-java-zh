# 11

# 超文本传输协议

**超文本传输协议**（**HTTP**）是用于网络信息交换的基础协议。它使客户端计算机和服务器之间的通信成为可能。HTTP 对 Java 的适用性主要归因于 Java 网络应用程序。

本章从 Java 上下文中的 HTTP 介绍开始。一旦解决了基础知识，章节将转向使用内置和第三方 HTTP 相关库的实用方法。本章还涵盖了使用 HTTP 进行 API 集成，并探讨了使用 HTTP 的安全顾虑，以及 HTTPS 的使用。最后一节专注于在采用 HTTP 时对 Java 应用程序进行性能优化。到本章结束时，你应该对 HTTP 如何影响 Java 应用程序的性能有一个牢固的理解，并且对未来的 HTTP 实现感到舒适。

本章涵盖了以下主要主题：

+   HTTP 简介

+   Java 网络应用程序

+   在 Java 中使用 HTTP

+   API 集成

+   安全考虑

+   性能优化

# 技术要求

要遵循本章中的示例和说明，你需要具备加载、编辑和运行 Java 代码的能力。如果你还没有设置你的开发环境，请参阅*第一章*，*窥探 Java 虚拟机 (JVM)*。

本章的完整代码可以在以下链接找到：[`github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter11`](https://github.com/PacktPublishing/High-Performance-with-Java/tree/main/Chapter11).

# HTTP 简介

HTTP 大约在 1990 年发布，因此你很可能至少熟悉了客户端和服务器通信的基本概念。万维网建立在 HTTP 之上，HTTP 仍然是今天网络的基础。如果你计划构建一个网络应用程序，特别是使用 Java，理解这个协议是很重要的。在本节中，我们将探讨 Java 中的 HTTP。

## HTTP 核心组件

HTTP 是一种无状态协议，使用称为**请求-响应**的通信模型。每个请求-响应交换都是独立的，这使得它成为一个非常简单的协议。今天的网络服务在很大程度上依赖于 HTTP。

无状态协议

无状态协议是指接收者不需要保留或跟踪发送的信息。没有保存会话信息。

HTTP 有四个基本组成部分：

+   首先是**请求和响应对**。这对表示了所有网络通信的核心。客户端发送一个请求以获取信息，例如加载网页，服务器则发送回包含所需信息的响应或另一个操作结果，如错误。

+   请求和响应都包含提供元数据的**头部**。这个 HTTP 组件包括发送内容类型、请求内容、认证细节和其他信息。

+   第三个 HTTP 组件是**状态码**。当服务器响应客户端的请求时，它们会提供一个状态码来描述请求的结果。这些代码按照下表所示按编号系列分类：

| **类别** | **响应类型** | **示例** |
| --- | --- | --- |
| 100 系列 | 信息性响应 | 100 Continue |
| 200 系列 | 成功响应 | 200 OK |
| 300 系列 | 重定向消息 | 301 永久移动 |
| 400 系列 | 客户端错误 | 404 未找到 |
| 500 系列 | 服务器错误 | 500 内部服务器错误 |

表 11.1 – HTTP 状态码及示例

+   第四个 HTTP 组件是**方法**。HTTP 使用几种请求方法来执行所需操作。以下是常见 HTTP 方法的列表：

| **方法** | **功能** |
| --- | --- |
| `DELETE` | 删除资源 |
| `GET` | 获取资源 |
| `HEAD` | 获取资源的头部 |
| `POST` | 向资源提交数据 |
| `PUT` | 更新资源 |

表 11.2 – HTTP 方法

现在我们对 HTTP 有了基本了解，让我们来探讨这个协议对 Java 开发者的意义。

## Java 和 HTTP

有许多库和例如`HttpClient`这样的库可以帮助简化我们对 HTTP 操作的使用。学习如何使用可用的 API 和库非常重要，尤其是在我们关注 Java 应用程序性能时。

HTTP 知识之所以如此重要，其中一个原因是大多数 API 集成都涉及 HTTP 通信。这要求我们了解如何构建 HTTP 请求，如何处理响应状态码，以及如何解析响应。

Java 开发者开发 Web 应用程序时也应该精通 HTTP。HTTP 是客户端-服务器通信的基础协议。这强调了 HTTP 知识对 Java 开发者的重要性。

Java 开发者应该全面理解 HTTP 的另一个原因是它在整体程序性能中起着重要作用。HTTP 缺乏复杂性，但仍然是开发 Web 应用、微服务、小程序和其他应用程序类型的关键协议。

# Java Web 应用程序

Java Web 应用程序是一种服务器端应用程序，用于创建动态网站。我们创建与 Java Web 应用程序交互的网站，根据用户输入动态生成内容。常见示例包括以下内容：

+   电子商务平台

+   企业应用程序

+   在线银行

+   信息管理系统

+   社交媒体平台

+   基于云的应用程序

+   教育平台

+   医疗保健应用程序

+   游戏服务器

+   **物联网**（**IoT**）应用程序

这些示例展示了使用 Java 开发和部署高性能 Web 应用的灵活性。正如你所预期的那样，HTTP 是这些 Java Web 应用的基础组件。

让我们接下来回顾 Java Web 应用程序的基本架构，以便我们理解 HTTP 的作用。

## Java Web 应用程序架构

大多数 Java Web 应用程序由四个层次组成，使其成为一个多层级架构：

+   **客户端层**：客户端层是用户所见到的部分，通常通过 Web 浏览器。这些网页通常由**超文本标记语言**（**HTML**）、**层叠样式表**（**CSS**）和**JavaScript**（**JS**）组成。

+   **Web 层（或服务器层）**：这一层接收并处理 HTTP 请求。我们可以使用多种技术，如**Java 服务器页面**（**JSP**）来完成这项任务。

+   **业务层**：业务层是我们应用程序逻辑所在的地方。在这里，数据处理、计算执行和基于逻辑的决策被做出。这一层与 Web 层之间的联系非常广泛。

+   **数据层**：数据层是后端系统的关键部分。这一层负责管理数据库，确保数据安全和持久性。

## 关键技术

值得一提的关键技术包括 Servlets、JSPs、Spring 框架和 Jakarta EE：

+   **Servlets**：运行在 Web 服务器上的 Java 程序被称为 Servlet。这种特殊软件位于客户端的 HTTP 请求和 Web 服务器上的应用程序和/或数据库之间。

+   **JSP**：JSP 是用于在服务器上执行并生成动态网页内容的文本文档。JSP 通常与 Servlets 一起使用。

+   **Spring 框架**：Spring 是一个常用的 Java 应用程序框架，用于开发 Java Web 应用程序。

+   **Jakarta EE**：**Jakarta 企业版**（**Jakarta EE**）是一组应用程序规范，它扩展了 Java **标准版**（**SE**）。它包括关于 Web 服务和分布式计算的规范。

## 创建简单 Java Web 应用程序的步骤

创建 Java Web 应用程序有六个基本步骤。请注意，这是一个简化的方法，并且将根据你的应用程序需求而变化，例如对数据库、API 等的需要：

1.  第一步是建立你的开发环境。这包括一个**集成开发环境**（**IDE**），如 Visual Studio Code，最新的**Java 软件开发工具包**（**JDK**），以及一个 Web 服务器，如 Apache Tomcat。

1.  下一步是在你的 IDE 中创建一个新的 Web 应用程序项目。这一步包括创建项目的文件目录结构。

1.  接下来，我们将编写 Java Servlet 的代码来处理 HTTP 请求。在这一步中，我们还将定义 Servlet 将响应的路线。这通常使用 URL 定义。

1.  接下来，我们将创建我们计划用于生成 HTML 内容的 JSP 页面或模板。这是我们发送回客户端的内容。

1.  接下来，我们将创建业务逻辑，这是我们应用程序的核心。我们可以通过一系列 Java 类来实现这一点。

1.  最后，我们将我们的应用程序打包成一个**Web 应用程序存档**（**WAR**），它类似于**Java 应用程序存档**（**JAR**），但用于 Web 应用程序，并将其部署。

在开发 Java Web 应用程序时，我们应该创建具有明确边界的应用程序，这些边界分别位于表示层、业务层和数据访问层之间。这种方法有助于模块化、可扩展性和可维护性。还建议使用 Spring 和 Jakarta EE 等框架。这样做可以简化我们的开发工作，并为 Web 应用程序开发提供内在支持。

动态网页是标准，并且用户期望如此，因此拥抱 Java Web 应用程序技术对所有 Java 开发者来说都很重要。

# 在 Java 中使用 HTTP

作为 Java 开发者，我们可以使用 HTTP 来创建动态 Web 应用程序。这些应用程序将使用 HTTP 在浏览器和服务器之间进行通信。Java 包括`HttpClient`库，这是一个 Java 库，它使得处理 HTTP 请求和响应变得高效。让我们来看一个例子。

上一段代码使用了`HttpClient`库来创建一个`GET`请求，该请求从特定的资源（在我们的例子中是模拟的）检索数据：

```java
import java.io.IOException;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
public class GetRequestExample {
  public static void main(String[] args) {
    HttpClient client = HttpClient.newHttpClient();
    HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create("https://api.not-real-just-
         an-example.com/data"))
        .GET()
        .build();
    try {
      HttpResponse<String> response = client.send(request,
      HttpResponse.BodyHandlers.ofString());
      System.out.println("Status Code: " +
      response.statusCode());
      System.out.println("Response Body: \n" + response.body());
    } catch (IOException | InterruptedException e) {
        e.printStackTrace();
    }
  }
}
```

上一段示例向一个模拟的 URL 发送一个`GET`请求，并打印出响应的状态码和正文。

接下来，让我们看看制作`POST`请求的方法。这种请求类型可以用来使用 JSON 向特定资源提交数据。在我们的例子中，这将是一个模拟的资源：

```java
import java.io.IOException;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.net.http.HttpHeaders;
import java.nio.charset.StandardCharsets;
public class PostRequestExample {
  public static void main(String[] args) {
    HttpClient client = HttpClient.newHttpClient();
    String json = "{\"name\":\"Neo
    Anderson\",\"email\":\"neo.anderson@thematrix.com\"}";
    HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create("https://api.not-real-just-an-
        example.com/users"))
        .header("Content-Type", "application/json")
        .POST(HttpRequest.BodyPublishers.ofString(json,
        StandardCharsets.UTF_8))
        .build();
    try {
      HttpResponse<String> response = client.send(request,
      HttpResponse.BodyHandlers.ofString());
      System.out.println("Status Code: " +
      response.statusCode());
      System.out.println("Response Body: \n" +
      response.body());
    } catch (IOException | InterruptedException e) {
        e.printStackTrace();
    }
  }
}
```

这个例子简单地向一个模拟的 URL 发送一个`POST`请求，其中包含一个包含用户信息的 JSON 包。

利用`HttpClient`库可以简化与 Web 服务交互的代码开发过程。

# API 集成

当我们构建 Java Web 应用程序时，我们可以集成外部 API 来扩展我们应用程序的功能。一个例子是天气服务 API，可以用来在网站上显示当地温度。对于本节，我们将重点关注**表示状态转换**（**RESTful**）服务，因为这些是最常见的 Web API 类型。

RESTful API 使用标准的 HTTP 方法，例如上一节中的`GET`和`POST`示例。正如你所期望的，RESTful API 主要通过 HTTP 数据交换使用 JSON 和 XML 格式进行通信。

当我们实现一个 API 时，我们首先了解其所需请求方法，以及请求和响应的指定格式。API 需要认证的情况越来越普遍，因此你可能需要使用 API 密钥或其他授权技术来应对这一点。

下面的示例演示了一个实现天气 API 的简单应用程序：

```java
import java.io.IOException;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
public class WeatherApiExample {
  public static void main(String[] args) {
    HttpClient client = HttpClient.newHttpClient();
    String apiKey = "your_api_key_here";
    String city = "Florence";
    String uri = String.format("https://api.fakeweatherapi.com/v1/
    current.json?key=%s&q=%s", apiKey, city);
    HttpRequest request = HttpRequest.newBuilder()
      .uri(URI.create(uri))
      .GET()
      .build();
    try {
      HttpResponse<String> response = client.send(request, 
      HttpResponse.BodyHandlers.ofString());
      System.out.println("Weather Data: \n" + response.body());
    } catch (IOException | InterruptedException e) {
        e.printStackTrace();
    }
  }
}
```

我们的示例向 API 发送一个`GET`请求，将城市作为查询参数传递。JSON 响应将包含适用的天气数据，这些数据将打印在系统的显示上。

API 集成可以被认为是许多基于 Java 的 Web 应用程序的核心组件，因为它的适用性非常广泛。

# 安全考虑

每当我们向我们的 Java 应用程序添加发送信息到应用程序外部或从外部源接收信息的函数时，安全性就成为一个至关重要的关注点。这在我们将 API 和 HTTP 集成到 Java 应用程序中时尤其如此。让我们看看我们可以使用的九个最佳实践，以确保我们的 HTTP 通信既安全，又在与 API 一起工作时安全：

+   **使用 HTTPS 而不是 HTTP**：如果你的 Java Web 应用程序处理敏感的、受保护的或私人信息，你应该在发送请求和响应时使用**HTTP 安全**（**HTTPS**）而不是 HTTP。这将有助于防止篡改和数据拦截。这将要求你为你的服务器获取**安全套接字层**（**SSL**）证书。

+   **不要信任输入**：我们应该始终验证我们的系统输入，包括用户输入和以编程方式传递给我们的应用程序的数据。我们不应假设这些数据是正确的格式。在验证数据后，我们可能需要清理它，以便可以在我们的应用程序中使用。这种方法可以帮助减轻诸如**SQL 注入**等恶意操作。

+   **认证**：尽可能识别你的应用程序与之交互的用户和系统。前面提到的 API 密钥在这里发挥作用。

+   **授权**：一旦用户或系统经过认证，我们应该确保他们有权在你的应用程序中执行特定操作。并非每个用户都将拥有相同的权限级别。

+   **保护 API 密钥**：我们已经提到了 API 密钥的重要性及其在解决安全问题的适用性。API 密钥就像密码一样；我们必须保护它们免受利用。我们不希望将这些密钥硬编码到我们的应用程序中；相反，我们应该将它们存储在加密的配置文件中，这样它们就可以免受未经授权的眼睛的侵害。

+   **使用安全头**：我们有使用**HTTP 安全头**的选项。以下是一些详细信息：

    +   **内容安全策略**（**CSP**）：这通过明确标识允许加载的资源来帮助防止 XSS 攻击

    +   **HTTP 严格传输安全**（**HSTS**）：这可以用来强制执行安全的服务器连接

+   **谨慎处理敏感数据**：这一点不言而喻，但敏感数据值得特别注意。例如，永远不要在 URL 中传输敏感数据，因为它们可能会被记录下来，然后泄露。此外，确保在存储敏感数据（如密码）时对其进行加密或散列。另外，使用如**令牌化**等技术来安全地处理支付信息。

+   **更新依赖项**：我们应该定期检查我们的依赖项和 Java 库是否是最新的。我们不希望使用可能存在已知漏洞的组件的旧版本。

+   **记录和监控**：与我们的所有软件一样，我们希望确保我们实施适当的记录，然后监控操作以确保日志中不包含敏感信息。

安全性始终应该是开发者心中的首要任务。当与 HTTP 和外部 API 一起工作时，这一点尤为重要。遵循本节中讨论的九项最佳实践是开发 Java Web 应用程序安全策略的好起点。

# 性能优化

现在我们已经充分介绍了 HTTP 是什么，它在 Java 中的应用，以及一些技术和最佳实践，让我们考虑使用 Java 时与性能相关的问题。我们查看性能问题的目标是提升用户体验，并提高我们应用程序的可扩展性、弹性和可维护性。

让我们看看在使用 HTTP 进行 Java 应用程序时，与性能优化相关的七个具体领域：

+   第一个领域侧重于`HttpClient`库，包括对连接池的支持。

+   我们可以使用`HTTP Keep-Alive`来保持与常见主机的多个请求的连接打开。这将减少通信握手次数并提高延迟。

+   我们经常可以利用异步请求（即 API 调用）来改善应用程序流程。

+   **缓存**是另一个需要关注的领域，以帮助优化性能。有一些缓存策略可以用来提高性能：

    +   在应用程序级别缓存频繁访问的数据。具体取决于您的应用程序及其使用的数据。甚至还有如 Caffeine 之类的缓存框架可以使用。

    +   使用 HTTP 缓存头（即 Cache-Control）可以帮助您控制响应缓存。

    +   如果您的 Java Web 应用程序处理静态内容（即图像），您可以考虑使用**内容分发网络**（**CDNs**）来将内容缓存得更靠近您的用户（即在特定地理区域的服务器上存储数据）。这种方法可以显著缩短用户的加载时间。*   另一个需要考虑的领域是**优化数据传输**。有两个具体的方法值得考虑来改进数据传输：

    +   在尽可能的范围内，我们应该最小化数据请求。显然，数据请求越少，我们的应用程序性能越好。实现这一点需要针对 API 集成设计采取有目的的方法。我们可以使用特定的 API 端点仅获取完成任务所需的数据，而不是使用庞大的数据包。

    +   当可能且适用时，在您的 API 上实施速率限制。这有助于防止滥用和拒绝服务攻击。同时也有助于维护服务质量。

    +   如果你的应用程序和 API 支持批量请求，那么实现它是值得的。这可以在系统性能上产生深远的影响。*   **代码优化**是另一个需要关注的领域。可以使用 VisualVM 和 JProfiler 等性能分析工具来帮助识别性能瓶颈。这些工具可以用来针对内存和 CPU 操作。详见*第十四章*，*性能分析工具*，获取更多信息。*   **SQL 优化**是另一个需要关注的领域。SQL 查询可以被优化以减少数据库负载和执行时间。对数据库模式的彻底审查可以帮助识别更多的优化机会。详见*第十五章*，*优化数据库和 SQL 查询*，获取更多信息。*   当我们在 Java 中处理 HTTP 时，我们关注的最后一个性能领域是**可伸缩性**。这个领域中的两个主要技术是负载均衡，有助于提高应用程序的可用性，以及微服务架构，以获得更好的性能。

优化我们 Java 应用程序的每一个方面对于实现开发高性能应用程序的目标至关重要。在 Java 中使用 HTTP 代表了一组独特的挑战和优化机会。

# 摘要

本章强调了 HTTP 在 Java Web 应用程序开发中所扮演的角色。HTTP 的目的被表述为促进动态 Web 应用程序的发展。我们对这个主题的覆盖表明，HTTP 可以被高效且安全地使用。我们还探讨了 Java Web 应用程序、API 集成、安全性和性能优化策略。由于 HTTP 和 Java Web 应用程序开发领域都非常活跃，因此了解相关变化和更新变得尤为重要。

下一章，*第十二章*，*优化框架*，介绍了使用异步输入/输出、缓冲输入/输出和批量操作来创建高性能 Java 应用程序的策略。本章还涵盖了微服务和云原生应用程序的框架。

# 第四部分：框架、库和性能分析

利用合适的框架和库可以显著提升应用程序的性能。本部分将探讨为优化设计的各种框架，并介绍可集成到 Java 项目中的以性能为导向的库。此外，它还提供了使用性能分析工具来识别和解决性能瓶颈的指南。本节中的章节旨在为您提供调整应用程序以实现最大效率所需的工具和知识。

本部分包含以下章节：

+   *第十二章*，*优化框架*

+   *第十三章*，*以性能为导向的库*

+   *第十四章*，*性能分析工具*
