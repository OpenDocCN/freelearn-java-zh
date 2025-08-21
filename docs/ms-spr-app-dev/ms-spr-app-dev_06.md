# 第六章：Spring 集成与 HTTP

在本章中，让我们看看 Spring 集成包如何支持 HTTP 协议。我们还将深入了解 HTTP 及其特性，以更好地理解如何使用 Spring 框架执行 HTTP 操作。

**HTTP**代表**超文本传输协议**，它又代表安全连接。该协议属于用于数据传输的应用层。它使用**传输控制** **协议/互联网协议**（**TCP/IP**）通信进行数据传输。HTTP 是一种无连接和无状态的协议，因为服务器和客户端只在请求和响应时相互知晓。只要服务器和客户端能够处理，任何类型的数据都可以通过 HTTP 发送。请求通过 Web URL 发送，即统一资源定位符。URL 包含以下部分：`http://www.domainname.com/path/?abc=xyz`

+   协议：`http://`或`https://`

+   主机：`www.domainname.com`

+   资源路径：`path`

+   查询：`abc=xyz`

# HTTP 方法和状态代码

让我们来看看 HTTP 方法和状态代码。HTTP 方法是 HTTP 协议上执行操作的通信渠道。

以下是使用的 HTTP 方法：

+   `GET`：这获取给定标识符的现有资源。

+   `PUT`：这放置一个新资源。

+   `POST`：这将更新现有资源。

+   `DELETE`：这将删除现有资源。

状态代码是关于 HTTP 消息的人类可读的诊断信息。

以下表格显示了所有可用状态代码及其含义：

| 状态代码 | 含义 |
| --- | --- |
| 200 | 请求成功 |
| 201 | POST 方法成功执行 |
| 202 | 请求已被接受进行处理 |
| 203 | 未授权查看信息 |
| 204 | 服务器没有响应 |
| 301 | 请求的数据已移至新 URL |
| 302 | 请求需要进行前向操作才能完全实现 |
| 303 | 所有 3XX 代码都指向不同的 URL，用于不同的操作，如转发 |
| 304 | 缓存未正确修改 |
| 400 | 语法错误 |
| 401 | 未经授权的请求 |
| 402 | 头部收费不匹配 |
| 403 | 禁止请求 |
| 404 | 根据提供的 URL 未找到资源 |
| 500 | 服务器发生意外错误 |
| 501 | 服务器不支持该操作 |
| 502 | 服务器负载过重 |
| 503 | 网关超时。服务器正在尝试从其他资源或服务访问数据，但未收到所需的响应。 |

## HTTP 标头

这些标头在 HTTP 请求和响应的消息中找到。它们只是由冒号分隔的名称值字符串。内容类型、缓存、响应类型等信息可以直接在标头中给出。标头通常没有任何大小限制，但服务器对标头大小有限制。

## HTTP 超时

这是一个 408 状态代码，当服务器尝试访问数据太多次而没有得到任何响应时，它会出现在网页上。即使服务器运行缓慢，也会出现这种错误。

超时可能发生在两种情况下，一种是与 Spring 集成通道交互时，可以是入站通道或出站通道，另一种是与远程位置的 HTTP 服务器交互时。

超时支持是使用 Spring 框架中可用的`RestTemplate`类完成的。以下是可用于与 Spring 集成中的 HTTP 网关和出站适配器的示例配置。

```java
<bean id="requestFactory"
      class="org.springframework.http.client.SimpleClientHttpRequestFactory">
    <property name="connectTimeout" value="5000"/>
    <property name="readTimeout"    value="5000"/>
</bean>
```

## Java 中的 HTTP 代理设置

代理设置由 Java 系统属性支持。这些属性可以设置为使用具有代理设置的服务器。以下是可以设置的属性：

+   `http.proxyHost`：代理服务器的主机名。

+   `http.proxyPort`：端口号，默认值为 80。

+   `http.nonProxyHosts`：应直接到达的主机列表，绕过代理。这是一个由`|`字符分隔的模式列表。这些模式可以以`*`字符开始或结束，用作通配符。匹配这些模式之一的任何主机将通过直接连接而不是通过代理到达。

以下是用于安全 HTTP 的代理设置：

+   `https.proxyHost`：代理服务器的主机名。

+   `https.proxyPort`：端口号，默认值为 80。

# Spring 中的代理配置支持

Spring 支持代理配置。我们只需要配置`SimpleClientHttpRequestFactory` bean，它具有一个带有`java.net.Proxy` bean 的代理属性。以下代码显示了一个示例配置：

```java
<bean id="requestFactory" class="org.springframework.http.client.SimpleClientHttpRequestFactory">
  <property name="proxy">
  <bean id="proxy" class="java.net.Proxy">
    <constructor-arg>
    <util:constant static-field="java.net.Proxy.Type.HTTP"/>
    </constructor-arg>
    <constructor-arg>
    <bean class="java.net.InetSocketAddress">
      <constructor-arg value="123.0.0.1"/>
      <constructor-arg value="8080"/>
    </bean>
    </constructor-arg>
  </bean>
  </property>
</bean>
```

## Spring 集成对 HTTP 的支持

Spring 通过适配器扩展了对 HTTP 的支持，就像 FTP 一样，其中包括网关实现。Spring 支持 HTTP 使用以下两种网关实现：

+   `HttpInboundEndpoint`：要通过 HTTP 接收消息，我们需要使用适配器或可用的网关。入站适配器称为 HTTP 入站适配器，网关称为 HTTP 入站网关。适配器需要一个 servlet 容器，比如 Tomcat 服务器或 Jetty 服务器。我们需要制作一个带有 servlet 配置的 web 应用程序，并将其部署到 web 服务器上。Spring 本身提供了一个名为的 servlet。

+   HttpRequestHandlerServlet：这个类扩展了普通的`HttpServlet`，并且位于`org.springframework.web.context.support.HttpRequestHandlerServlet`包下。由于它扩展了`HttpServlet`，它还覆盖了`init()`和`service()`方法。

以下是`web.xml`文件中的 servlet 配置：

```java
<servlet>
  <servlet-name>inboundGateway</servlet-name>
  <servlet-class>o.s.web.context.support.HttpRequestHandlerServlet</servlet-class>
</servlet>
```

以下是处理入站 HTTP 请求的网关配置。该网关接受一系列消息转换器，这些转换器将从`HttpServletRequest`转换为消息：

```java
<bean id="httpInbound" class="org.springframework.integration.http.inbound.HttpRequestHandlingMessagingGateway">
  <property name="requestChannel" ref="httpRequestChannel" />
  <property name="replyChannel" ref="httpReplyChannel" />
</bean>
```

## Spring 集成对多部分 HTTP 请求的支持

如果 HTTP 请求被包装，`MultipartHttpServletRequest`转换器将把请求转换为消息载荷，这只是一个`MultiValueMap`。这个映射将有值，这些值是 Spring 的多部分的实例。值是根据内容类型决定的。值也可以是字节数组或字符串。默认情况下，如果有一个名为`MultipartResolver`的 bean，它会被 Spring 的集成框架识别；如果有一个名为`multipartResolver`的 bean，它反过来会启用上下文。这将启用入站请求映射。

## Spring 集成对 HTTP 响应的支持

对 HTTP 请求的响应通常以 200Ok 状态码发送。要进一步自定义响应，可以使用 Spring MVC 框架。在 Spring MVC 应用程序中，我们有一个选项来自定义响应。我们可以为响应提供一个`viewName`，这个`viewName`会被 Spring MVC 的`ViewResolver`解析。我们可以配置网关以像 Spring 控制器一样运行，它返回一个视图名称作为框架的响应，我们还可以配置 HTTP 方法。

在以下配置中，您可以看到我们使用了一个集成包，并配置了`HttpRequestHandlingController` bean 的以下属性：

+   `HttpRequestChannel`

+   `HttpReplyChannel`

+   `viewName`

+   `SupportedMedthodNames`

+   以下代码片段显示了`HttpInbound` bean 的配置。

+   我们还可以配置支持的 HTTP 方法。

```java
<bean id="httpInbound" class="org.springframework.integration.http.inbound.HttpRequestHandlingController">
  <constructor-arg value="true" /> <!-- indicates that a reply is expected -->
  <property name="requestChannel" ref="httpRequestChannel" />
  <property name="replyChannel" ref="httpReplyChannel" />
  <property name="viewName" value="jsonView" />
  <property name="supportedMethodNames" >
    <list>
      <value>GET</value>
      <value>DELETE</value>
    </list>
  </property>
</bean>
```

# 配置出站 HTTP 消息

Spring 提供了`HttpRequestExecutingMessageHandler`，它以字符串 URL 作为构造函数参数。该类有一个名为`ReponseChannel`的属性，也需要进行配置。

该 bean 将通过读取构造函数中配置的 URL 调用`RestTemplate`类，`RestTemplate`调用`HttpMessageConverters`。读取`HttpMessageConverters`列表，并生成`HttpRequest`主体。

转换器和`HttpRequestExecutingMessageHandler`在以下代码中显示：

```java
<bean id="httpOutbound" class="org.springframework.integration.http.outbound.HttpRequestExecutingMessageHandler">
  <constructor-arg value="http://localhost:8080/myweb" />
  <property name="outputChannel" ref="responseChannel" />
</bean>
```

或者

```java
<bean id="httpOutbound" class="org.springframework.integration.http.outbound.HttpRequestExecutingMessageHandler">
  <constructor-arg value="http://localhost:8080/myweb" />
  <property name="outputChannel" ref="responseChannel" />
  <property name="messageConverters" ref="messageConverterList" />
  <property name="requestFactory" ref="customRequestFactory" />
</bean>
```

## 配置出站网关的 cookies

`OutboundGateway`具有传输 cookies 属性，接受 true 或 false 的布尔值。响应中的标头包含一个设置 cookies 参数，如果`transfer-cookie`属性设置为`True`，则将响应转换为 cookie。

# 配置既无响应又有响应的入站网关

使用以下代码配置无响应的`InboundGateway`请求：

```java
<int-http:inbound-channel-adapter id="httpChannelAdapter" channel="requests"
    supported-methods="PUT, DELETE"/>
```

对于需要响应的请求：

```java
<int-http:inbound-gateway id="inboundGateway"
    request-channel="requests"
    reply-channel="responses"/>
```

# 入站通道适配器或网关的 RequestMapping 支持

`requestmapping`配置可以用于入站通道适配器或网关，如下所示：

```java
<inbound-gateway id="inboundController"
    request-channel="requests"
    reply-channel="responses"
    path="/foo/{fooId}"
    supported-methods="GET"
    view-name="foo"
    error-code="oops">
   <request-mapping headers="User-Agent"
<!—-headers=""-->
     params="myParam=myValue"
     consumes="application/json"
     produces="!text/plain"/>
</inbound-gateway>
```

基于此配置，命名空间解析器将创建`IntegrationRequestMappingHandlerMapping`的实例（如果尚不存在），`HttpRequestHandlingController` bean，并与之关联`RequestMapping`的实例，然后将其转换为 Spring MVC 的`RequestMappingInfo`。

使用路径和支持的方法，`<http:inbound-channel-adapter>`或`<http:inbound-gateway>`的属性，`<request-mapping>`直接转换为 Spring MVC 中`org.springframework.web.bind.annotation.RequestMapping`注解提供的相应选项。

`<request-mapping>`子元素允许您配置多个 Spring 集成 HTTP 入站端点到相同的路径（甚至相同的支持方法），并根据传入的 HTTP 请求提供不同的下游消息流。

## 使用 HTTP 入站端点配置 RequestMapping

我们还可以声明一个 HTTP 入站端点，并在 Spring 集成流程中应用路由和过滤逻辑，以实现相同的结果。这允许您尽早将消息传递到流程中，例如：

```java
<int-http:inbound-gateway request-channel="httpMethodRouter"
    supported-methods="GET,DELETE"
    path="/process/{entId}"
    payload-expression="#pathVariables.entId"/>
<int:router input-channel="httpMe
thodRouter" expression="headers.http_requestMethod">
    <int:mapping value="GET" channel="in1"/>
    <int:mapping value="DELETE" channel="in2"/>
</int:router>
<int:service-activator input-channel="in1" ref="service" method="getEntity"/>
<int:service-activator input-channel="in2" ref="service" method="delete"/>
```

## 配置入站通道适配器以从 URL 读取请求信息

我们还可以配置入站通道适配器以接受使用 URI 的请求。

URI 可以是`/param1/{param-value1}/param2/{param-value2}`。 URI 模板变量通过有效负载表达式属性与消息有效负载进行映射。 URI 路径中的某些变量也可以与标头进行映射：

```java
<int-http:inbound-channel-adapter id="inboundAdapterWithExpressions"
    path="/var-1/{phone}/var-2/{username}"
    channel="requests"
    payload-expression="#pathVariables.firstName">
    <int-http:header name="var-2" expression="#pathVariables.username"/>
</int-http:inbound-channel-adapter>
```

以下是可以在配置中使用的有效负载表达式列表：

+   `#requestParams`：来自`ServletRequest`参数映射的`MultiValueMap`。

+   `#pathVariables`：URI 模板占位符及其值的映射。

+   `#matrixVariables`：`MultiValueMap`的映射。

+   `#requestAttributes`：与当前请求关联的`org.springframework.web.context.request.RequestAttributes`。

+   `#requestHeaders`：当前请求的`org.springframework.http.HttpHeaders`对象。

+   `#cookies`：当前请求的`javax.servlet.http.Cookies`的`<String，Cookie>`映射。

# 为 HTTP 响应配置出站网关

出站网关或出站通道适配器配置与 HTTP 响应相关，并提供配置响应的选项。 HTTP 请求的默认响应类型为 null。响应方法通常为 POST。如果响应类型为 null 且 HTTP 状态代码为 null，则回复消息将具有`ResponseEntity`对象。在以下示例配置中，我们已配置了预期：

```java
<int-http:outbound-gateway id="example"
    request-channel="requests"
    URL="http://localhost/test"
    http-method="POST"
    extract-request-payload="false"
    expected-response-type="java.lang.String"
    charset="UTF-8"
    request-factory="requestFactory"
    reply-timeout="1234"
    reply-channel="replies"/>
```

## 为不同的响应类型配置出站适配器

现在，我们将向您展示两个配置出站适配器的示例，使用不同的响应类型。

在这里，使用预期的响应类型表达式与值有效负载：

```java
<int-http:outbound-gateway id="app1"
    request-channel="requests"
    URL="http://localhost/myapp"
    http-method-expression="headers.httpMethod"
    extract-request-payload="false"
    expected-response-type-expression="payload"
    charset="UTF-8"
    request-factory="requestFactory"
    reply-timeout="1234"
    reply-channel="replies"/>
```

现在，配置出站通道适配器以提供字符串响应：

```java
<int-http:outbound-channel-adapter id="app1"
    url="http://localhost/myapp"
    http-method="GET"
    channel="requests"
    charset="UTF-8"
    extract-payload="false"
    expected-response-type="java.lang.String"
    request-factory="someRequestFactory"
    order="3"
    auto-startup="false"/>
```

# 将 URI 变量映射为 HTTP 出站网关和出站通道适配器的子元素

在本节中，我们将看到 URI 变量和 URI 变量表达式的用法，作为 HTTP 出站网关配置的子元素。

如果您的 URL 包含 URI 变量，可以使用 Uri-variable 子元素进行映射。此子元素适用于 HTTP 出站网关和 HTTP 出站通道适配器：

```java
<int-http:outbound-gateway id="trafficGateway"
    url="http://local.yahooapis.com/trafficData?appid=YdnDemo&amp;zip={zipCode}"
    request-channel="trafficChannel"
    http-method="GET"
    expected-response-type="java.lang.String">
    <int-http:uri-variable name="zipCode" expression="payload.getZip()"/>
</int-http:outbound-gateway>
```

`Uri-variable`子元素定义了两个属性：`name`和`expression`。`name`属性标识 URI 变量的名称，而`expression`属性用于设置实际值。使用`expression`属性，您可以利用**Spring Expression Language**（**SpEL**）的全部功能，这使您可以完全动态地访问消息负载和消息标头。例如，在上面的配置中，将在消息的负载对象上调用`getZip()`方法，并且该方法的结果将用作名为`zipCode`的 URI 变量的值。

自 Spring Integration 3.0 以来，HTTP 出站端点支持`Uri-variables-expression`属性，用于指定应该评估的`Expression`，从而为 URL 模板中的所有 URI 变量占位符生成一个映射。它提供了一种机制，可以根据出站消息使用不同的变量表达式。此属性与`<Uri-variable/>`子元素互斥：

```java
<int-http:outbound-gateway
  url="http://foo.host/{foo}/bars/{bar}"
  request-channel="trafficChannel"
  http-method="GET"
  Uri-variables-expression="@uriVariablesBean.populate(payload)"
  expected-response-type="java.lang.String"/>
```

## 使用 HTTP 出站网关和 HTTP 入站网关处理超时

以下表格显示了处理 HTTP 出站和 HTTP 入站网关的差异：

| **HTTP 出站网关中的超时** | **HTTP 入站网关中的超时** |
| --- | --- |
| `ReplyTimeOut`映射到`HttpRequestExecutingMessageHandler`的`sendTimeOut`属性。 | 在这里，我们使用`RequestTimeOut`属性，它映射到`HttpRequestHandlingMessagingGateway`类的`requestTimeProperty`。 |
| `sendTimeOut`的默认值为`1`，发送到`MessageChannel`。 | 默认超时属性为 1,000 毫秒。超时属性将用于设置`MessagingTemplate`实例中使用的`sendTimeOut`参数。 |

# Spring 对标头自定义的支持

如果我们需要对标头进行进一步的自定义，则 Spring Integration 包为我们提供了完整的支持。如果在配置中明确指定标头名称，并使用逗号分隔的值，将覆盖默认行为。

以下是进一步标头自定义的配置：

```java
<int-http:outbound-gateway id="httpGateway"
    url="http://localhost/app2"
    mapped-request-headers="boo, bar"
    mapped-response-headers="X-*, HTTP_RESPONSE_HEADERS"
    channel="someChannel"/>

<int-http:outbound-channel-adapter id="httpAdapter"
    url="http://localhost/app2"
    mapped-request-headers="boo, bar, HTTP_REQUEST_HEADERS"
    channel="someChannel"/>
```

另一个选项是使用 header-mapper 属性，该属性采用 DefaultHttpHeaderMapper 类的配置。

该类配备了用于入站和出站适配器的静态工厂方法。

以下是`header-mapper`属性的配置：

```java
<bean id="headerMapper" class="o.s.i.http.support.DefaultHttpHeaderMapper">
  <property name="inboundHeaderNames" value="foo*, *bar, baz"/>
  <property name="outboundHeaderNames" value="a*b, d"/>
</bean>
```

# 使用 Spring 的 RestTemplate 发送多部分 HTTP 请求

大多数情况下，我们在应用程序中实现了文件上传功能。文件作为多部分请求通过 HTTP 发送。

在本节中，让我们看看如何使用`RestTemplate`配置入站通道适配器以通过 HTTP 请求发送文件。

让我们使用入站通道适配器配置服务器，然后为其编写客户端：

```java
<int-http:inbound-channel-adapter id="httpInboundAdapter"
  channel="receiveChannel"
  name="/inboundAdapter.htm"
  supported-methods="GET, POST"/>
<int:channel id="receiveChannel"/>
<int:service-activator input-channel="receiveChannel">
  <bean class="org.springframework.integration.samples.multipart.MultipartReceiver"/>
</int:service-activator>
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"/>
```

`httpInboundAdapter`将接收请求并将其转换为带有`LinkedMultiValueMap`负载的消息。然后，我们将在`multipartReceiver`服务激活器中解析它。

```java
public void receive(LinkedMultiValueMap<String, Object> multipartRequest){
  System.out.println("### Successfully received multipart request ###");
  for (String elementName : multipartRequest.keySet()) {
    if (elementName.equals("company")){
      System.out.println("\t" + elementName + " - " +((String[]) multipartRequest.getFirst("company"))[0]);
    }
    else if (elementName.equals("company-logo")){
      System.out.println("\t" + elementName + " - as UploadedMultipartFile: " + ((UploadedMultipartFile) multipartRequest.getFirst("company-logo")).getOriginalFilename());
    }
  }
}
```

现在，让我们编写一个客户端。通过客户端，我们指的是创建一个地图并向其中添加文件。

1.  现在，我们将创建一个`MultiValueMap`：

```java
MultiValueMap map = new LinkedMultiValueMap();
```

1.  地图可以填充值，例如个人的详细信息：

```java
Resource anjanapic = new ClassPathResource("org/abc/samples/multipart/anjana.png");
map.add("username","anjana");
map.add("lastname","mankale");
map.add("city","bangalore");
map.add("country","India");
map.add("photo",anjana.png);
```

1.  此步骤是创建标头并设置内容类型：

```java
HttpHeaders headers = new HttpHeaders();
headers.setContentType(new MediaType("multipart", "form-data"));
```

1.  我们需要将`header`和`map`作为请求传递给`HttpEntity`：

```java
HttpEntity request = new HttpEntity(map, headers);
```

1.  让我们使用`RestTemplate`传递请求：

```java
RestTemplate template = new RestTemplate();
String Uri = "http://localhost:8080/multipart-http/inboundAdapter.htm";
ResponseEntity<?> httpResponse = template.exchange(Uri, HttpMethod.POST, request, null
```

现在，我们应该得到一个输出，其中照片已上传到服务器。

# 总结

在本章中，我们已经了解了 HTTP 和 Spring Integration 对访问 HTTP 方法和请求的支持。我们还演示了多部分请求和响应，并展示了如何配置入站和出站 HTTP 网关和适配器。

我们已经学习了通过配置 Spring 的入站和出站网关来发送多部分 HTTP 请求。我们还演示了如何使用多值映射来填充请求并将映射放入 HTTP 头部。最后，我们看到了可用的有效负载表达式列表。

在下一章中，让我们来看看 Spring 对 Hadoop 的支持。
