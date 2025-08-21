## 第九章。交换消息：消息传递

到目前为止，我们已经讨论了很多关于基于传统 HTTP 通信的双向网络应用程序。这些基于浏览器的应用程序通过打开多个连接提供双向通信。**WebSocket 协议**提供了一种基于 TCP 的消息传递方式，不依赖于打开多个 HTTP 连接。在本章中，我们将通过以下几点讨论 WebSocket 协议：

+   **消息传递简介**

+   **WebSocket 协议简介**

+   **WebSocket API**

+   **STOMP 概览**

在网络应用程序中，客户端和服务器之间的双向通信是同步的，其中客户端请求资源，服务器发送 HTTP 调用通知。它解决了以下问题：

+   必须打开多个连接以发送信息并收集传入消息

+   跟踪将外出连接映射到进入连接，以便跟踪请求及其回复

更好的解决方案是维护一个用于发送和接收的单一 TCP 连接，这正是 WebSocket 作为无头部的低层协议所提供的。由于没有添加头部，传输网络的数据量减少，从而降低了负载。这是通过称为拉取技术的过程来实现的，而不是 AJAX 中使用的长拉取的推技术。现在，开发者们正在使用**XMLHttpRequest（XHR）**进行异步 HTTP 通信。WebSocket 使用 HTTP 作为传输层，以支持使用 80、443 端口的基础设施。在这种双向通信中，成功的连接数据传输是独立于他们的意愿进行的。

《RFC 6455》将 WebSocket 协议定义为，一种在客户端运行在受控环境中与远程主机通信的客户端和服务器之间的双向通信协议，该远程主机已允许接受邮件、电子邮件或任何直接从代码发出的通信。该协议包括打开握手，随后是基本的报文框架，该框架建立在 TCP 协议之上。如果服务器同意，它会发送 HTTP 状态码 101，表示成功的握手。现在连接将保持开放，可以进行消息交换。以下图表给出了通信如何进行的大致概念：

![](img/image_10_001.png)

WebSocket 在 TCP 之上执行以下操作：

+   它向浏览器添加了网络安全模型。

+   由于一个端口需要支持多个主机名和多个服务，因此它增加了地址和命名机制以提供这种支持。

+   它在 TCP 之上形成了一层框架机制，以促进 IP 包机制。

+   关闭握手机制。

在 WebSocket 中的数据传输使用一系列的帧。这样的数据帧可以在打开握手之后，在端点发送 Close 帧之前，由任一方随时传输。

## Spring 和消息传递

* * *

从 Spring 4.0 开始，就有对 WebSocket 的支持，引入了 spring-websocket 模块，该模块与 Java WebSocket API（JSR-356）兼容。HTTPServlet 和 REST 应用程序使用 URL、HTTP 方法在客户端和服务器之间交换数据。但与这种相反，WebSocket 应用程序可能使用单个 URL 进行初始握手，这是异步的、消息传递的，甚至是基于 JMS 或 AMQP 的架构。Spring 4 包括 spring-messaging 模块，以集成消息、消息通道、消息处理器，一系列注解用于将消息映射到方法，以及许多其他功能以支持基本的消息传递架构。@Controller 和@RestController 我们已经用来创建 Spring MVC 网络应用程序和 RESTful 网络服务，它允许处理 HTTP 请求也支持 WebSocket 消息的处理方法。此外，控制器中的处理方法可以将消息广播给所有感兴趣的用户特定的 WebSocket 客户端。

### 使用

WebSocket 架构适用于所有那些需要频繁交换事件的网络应用程序，在这些应用程序中，数据交换到目标的时间是非常重要的，例如：

+   社交媒体目前在日常生活中扮演着非常重要的角色，并且对于与家人和朋友保持联系起着至关重要的作用。用户总是喜欢实时接收他们圈子内完成的 Feed 更新。

+   如今，许多在线多人游戏都可以在网络上找到。在这样的游戏中，每个玩家总是急于知道他的对手正在做什么。没有人希望在对手采取行动时发现对手的举动。

+   在开发过程中，版本控制工具如 Tortoise SVN、Git 有助于跟踪文件。这样，在代码交换时就不会发生冲突，变得更加容易。但在这里，我们无法实时获取谁正在处理哪个文件的信息。

+   在金融投资中，人们总是希望知道他所感兴趣公司的实时股价，而不是之前的某个时间的股价。

## WebSocket API 概述

* * *

springframework 框架通过提供采用各种 WebSocket 引擎的 API，实现了 WebSocket 的创建。如今 Tomcat7.0.47+、Jetty 9.1+、WebLogic 12.1.3+、GlassFish 4.1+为 WebSocket 提供了运行环境。

### WebSocket 处理器的创建

我们可以通过实现 WebSocketHandler 接口或从 TextWebSocketHandler 或 BinaryWebSocketHandler 继承来创建 WebSocketHandler，如下代码片段所示：

```java
public class MyWebSocketHandler extends TextWebSocketHandler{ 
@Override 
   public void handleTextMessage(WebSocketSession session,     
     TextMessage message) 
   { 
       // code goes here 
   } 
} 

```

可以使用 WebSocketDecorator 类来装饰 WebSocketHandler。Spring 提供了一些装饰器类来处理异常、日志机制和处理二进制数据。`ExceptionWebSocketHandler`是一个异常处理的 WebSocketHandlerDecorator，它可以帮助处理所有 Throwable 实例。`LoggingWebSocketHandlerDecorator`为 WebSocket 生命周期中发生的事件添加日志记录。

### 注册 WebSocketHandler

WebSocket 处理器被映射到特定的 URL 以注册此映射。此框架可以通过 Java 配置或 XML 基础配置来完成。

#### 基于 Java 的配置

`WebSocketConfigurer` 用于在`registerWebSocketHandlers()`方法中将处理器与其特定的 URL 映射，如下面的代码所示：

```java
@Configuration 
@EnableWebSocket 
public class WebSocketConfig implements WebSocketConfigurer { 
  @Override 
  public void registerWebSocketHandlers(WebSocketHandlerRegistry   
     registry)  
  { 
     registry.addHandler(createHandler(), "/webSocketHandler"); 
  } 
  @Bean 
  public WebSocketHandler createMyHandler() { 
    return new MyWebSocketHandler(); 
  } 
} 

```

在此处，我们的 WebSocketHandler 被映射到了`/webSocketHandler` URL。

自定义 WebSocketHandler 以自定义握手的操作可以如下进行：

```java
@Configuration 
@EnableWebSocket 
public class MyWebSocketConfig implements WebSocketConfigurer { 
  @Override 
  public void registerWebSocketHandlers(WebSocketHandlerRegistry   
    registry)  
  { 
    registry.addHandler(createHandler(),     
       "/webSocketHandler").addInterceptors 
       (new HttpSessionHandshakeInterceptor()); 
  } 
  @Bean 
  public WebSocketHandler createMyHandler() { 
    return new MyWebSocketHandler(); 
  } 
} 

```

握手拦截器暴露了`beforeHandshake()`和`afterhandshake()`方法，以自定义 WebSocket 握手。`HttpSessionHandshakeInterceptor`促进了将 HtttpSession 中的信息绑定到名为`HTTP_SESSION_ID_ATTR_NAME`的握手属性下。这些属性可以用作`WebSocketSession.getAttributes()`方法。

#### XML 基础配置

上述 Java 代码片段中的注册也可以用 XML 完成。我们需要在 XML 中注册 WebSocket 命名空间，然后如以下所示配置处理器：

```java
<beans xmlns="http://www.springframework.org/schema/beans" 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xmlns:websocket= 
http://www.springframework.org/schema/websocket 
   xsi:schemaLocation= 
    "http://www.springframework.org/schema/beans 
  http://www.springframework.org/schema/beans/spring-beans.xsd 
    http://www.springframework.org/schema/websocket 
http://www.springframework.org/schema/websocket/spring- 
    websocket.xsd"> 
  <websocket:handlers> 
    <websocket:mapping path="/myWebSocketHandler"  
     handler="myWebSocketHandler"/> 
     </websocket:handlers> 
   <bean id="myWebSocketHandler"   
    class="com.packt.ch10.WebsocketHandlers. MyWebSocketHandler"       
    /> 
</beans> 

```

在 XML 中自定义的 WebSocketConfigurer 可以写成如下形式：

```java
<websocket:handlers> 
    <websocket:mapping path="/myWebSocketHandler"  
       handler="myWebSocketHandler"/> 
    <websocket:handshake-interceptors> 
      <bean class= 
         "org.springframework.web.socket.server.support. 
         HttpSessionHandshakeInterceptor"/> 
    </websocket:handshake-interceptors> 
     </websocket:handlers> 
  <!-bean for MyWebSocketHandler -à 
</beans>  

```

#### WebSocket 引擎配置

Tomcat7.0.47+, Jetty 9.1+,WebLogic 12.1.3+, GlassFish 4.1+ 为 WebSocket 提供运行环境。可以通过添加 WebSocketConfigurer 的 bean 来为 Tomcat 运行环境配置消息缓冲区大小、超时等特性，如下所示：

```java
@Bean 
public ServletServerContainerFactoryBean  
  createWebSocketContainer()  
{ 
  ServletServerContainerFactoryBean webSocketcontainer =  
    new ServletServerContainerFactoryBean(); 
    webSocketcontainer .setMaxTextMessageBufferSize(9000); 
    webSocketcontainer .setMaxBinaryMessageBufferSize(9000); 
  return webSocketcontainer ; 
  } 
} 

```

等效的 XML 配置可以写成：

```java
<bean class= "org.springframework.web.socket.server.standard. 
  ServletServerContainerFactoryBean"> 
  <property name="maxTextMessageBufferSize" value="9000"/> 
  <property name="maxBinaryMessageBufferSize" value="9000"/> 
</bean> 

```

##### 允许的来源配置

`origin`是代理商的特权范围。由众多作者以各种格式创建的内容存在其中，其中一些可能是有害的。由一个来源创建的内容可以自由地与其他来源的内容进行交互。代理商有设置规则的权限，其中一个内容与其他内容交互，称为**“同源策略”**。

让我们以 HTML 为例，其中有表单提交。每当用户代理输入数据时，输入的数据会被导出到 URI。在此处，URI 声明了对脚本文件通过 URI 接收的信息的完整性信任。

`http://packt.com/`, `http://packt.com:8080/`, `http://www.packt.com/`, `https://packt.com:80/`, `https://packt.com/`, `http://packt.org/` 是不同的 URI。

配置来源有三种方式：

+   允许同源

+   允许指定的来源列表。

+   允许所有来源

让我们首先详细讨论关于客户端服务器通信中 WebSocket 的创建和使用：

1.  WebSocket 的创建：

```java
      WebSocket socket=  new WebSocket( URL, protocols); 

```

+   URL 包含的内容：

    +   **协议**：URL 必须包含`ws`，表示不安全连接，或`wss`，表示安全连接。

    +   **主机名：**这是服务器的一个名称或 IP 地址。

    +   **端口**：我们要连接的远程端口，ws 连接默认使用端口'80'，而 wss 使用 443。

    +   **资源名称**：要获取的资源的路径 URL。

+   我们可以将 WebSocket 的 URL 写为：

    +   协议://主机名:端口号/资源路径

    +   ws://主机名:端口号/资源路径

    +   wss://主机名:端口号/资源路径

1.  关闭 WebSocket：

关闭连接时，我们使用`close()`方法，如`close(code, reason)`。

### 注意

代码：这是一个发送给服务器的数值状态。1000 表示正常关闭连接。

1.  WebSocket 的状态：

以下是 WebSocket 的连接状态，提供它处于哪种状态的信息：

+   **连接中**：构造 WebSocket，并尝试连接到指定 URL。这个状态被认为是连接状态，准备状态值为 0。

+   **打开**：一旦 WebSocket 成功连接到 URL，它将进入打开状态。只有在 WebSocket 处于打开状态时，数据才能在网络之间发送和接收。打开状态的准备状态值是"1"。

+   **关闭**：WebSocket 不会直接关闭，它必须与服务器通信，通知它正在断开连接。这个状态被认为是关闭状态。"open"状态的准备状态值是"2"。

+   **已关闭**：从服务器成功断开连接后，WebSocket 进入关闭状态。处于关闭状态的 WebSocket 有一个"readyState"值为 3。

1.  在 WebSocket 中的事件处理：

WebSocket 基于事件处理原理工作，其中回调方法被调用以完成过程。以下是 WebSocket 生命周期中发生的事件：

+   **onopen**：当 WebSocket 过渡到开放状态时，"onopen"事件处理程序会被调用。

+   **onmessage**：当 WebSocket 从服务器接收数据时，"onmessage"事件处理程序会被调用。接收到的数据将存储在"message"事件的"data"字段中。

数据字段有参数：

+   **onclose**：当 WebSocket 关闭时，"onclose"事件处理程序会被调用。事件对象将传递给"onclose"。它有三个字段：

+   **代码**：服务器提供的数值状态值。

+   **原因**：这是一个描述关闭事件的字符串。

+   **wasClean**：有一个布尔值，表示连接是否没有问题地关闭。在正常情况下，"wasClean"是 true。

+   **onerror**：当 WebSocket 遇到任何问题时，"onerror"事件处理程序会被调用。传递给处理程序的事件将是一个标准错误对象，包括"name"和"message"字段。

1.  发送数据：

数据传输通过`send()`方法进行，该方法处理 UTF-8 文本数据、ArrayBuffer 类型的数据以及 blob 类型的数据。'bufferedAmount'属性值为零确保数据发送成功。

让我们通过以下步骤开发一个 WebSocket 演示来查找国家首都：

1.  创建 Ch10_Spring_Message_Handler 作为动态网络应用程序。

1.  添加 Spring 核心、Spring 网络、spring-websocket、spring-messaging 模块的 jar 文件。还要添加 Jackson 的 jar 文件。

1.  让我们在 compackt.ch10.config 包中添加 MyMessageHandler 作为 TextWebSocketHandler 的子项。覆盖处理消息、WebSocket 连接、连接关闭的方法，如下所示：

```java
public class MyMessageHandler extends TextWebSocketHandler { 

        List<WebSocketSession> sessions = new CopyOnWriteArrayList<>(); 

          @Override 
          public void handleTextMessage(WebSocketSession session,  
            TextMessage message) throws IOException { 
            String country = message.getPayload(); 
            String reply="No data available"; 
            if(country.equals("India"))  { 
              reply="DELHI"; 
            } 
            else if(country.equals("USA"))  { 
                  reply="Washington,D.C";     
             } 
            System.out.println("hanlding message"); 

            for(WebSocketSession webSsession:sessions){ 
              session.sendMessage(new TextMessage(reply));   
            } 
          } 
          @Override 
          public void afterConnectionEstablished(WebSocketSession  
             session) throws IOException { 
            // Handle new connection here 
            System.out.println("connection establieshed:hello"); 
            sessions.add(session); 
            session.sendMessage(new TextMessage("connection  
              establieshed:hello")); 
            } 
          @Override 
          public void afterConnectionClosed(WebSocketSession session,   
            CloseStatus status) throws IOException { 
            // Handle closing connection here 
            System.out.println("connection closed : BYE"); 
          } 
          @Override 
          public void handleTransportError(WebSocketSession session,  
            Throwable exception) throws IOException { 
              session.sendMessage(new TextMessage("Error!!!!!!")); 
            } 
        } 

```

这个 MessageHandler 需要注册到 WebSocketConfigurer，为所有源的 URL'/myHandler'，如下所示：

```java
        @Configuration 
        @EnableWebSocket 
        public class MyWebSocketConfigurer extends  
        WebMvcConfigurerAdapter implements WebSocketConfigurer
        { 
          @Override 
          public void
          registerWebSocketHandlers(WebSocketHandlerRegistry  
            registry) { 
            registry.addHandler(myHandler(),  
            "/myHandler").setAllowedOrigins("*"); 
          } 
          @Bean 
          public WebSocketHandler myHandler() { 
            return new MyMessageHandler(); 
          } 
          // Allow the HTML files through the default Servlet 
          @Override 
           public void configureDefaultServletHandling 
             (DefaultServletHandlerConfigurer configurer) { 
            configurer.enable(); 
          } 
        } 

```

1.  在 web.xml 中添加前端控制器映射，就像在之前的应用程序中一样，servlet 名称是'books'。

1.  为了添加`viewResolver`的 bean，请添加 books-servlet.xml 文件。你可以根据应用程序的需求决定是否添加它作为一个 bean。

1.  还要添加配置以启用 Spring Web MVC，如下所示：

```java
        <mvc:annotation-driven /> 

```

1.  添加 country.jsp 作为一个 JSP 页面，其中包含一个国家列表，用户可以从下拉列表中选择国家以获取其首都名称：

```java
        <div> 
          <select id="country"> 
                <option value="India">INDIA</option> 
                <option value="USA">U.S.A</option> 
          </select><br> 
          <br> <br> 
           <button id="show" onclick="connect();">Connect</button> 
              <br /> <br /> 
            </div> 
          <div id="messageDiv"> 
              <p>CAPITAL WILL BE DISPLAYED HERE</p> 
              <p id="msgResponse"></p> 
          </div> 
        </div> 

```

1.  通过在你的资源中添加 sockjs-0.3.4.js，或者通过添加以下代码来添加 SockJS 支持：

```java
        <script type="text/javascript"  
          src="img/sockjs-0.3.4.js"></script>
```

1.  在表单提交时，会调用一个 JavaScript 方法，我们在前面讨论过的 onopen、onmessage 等 WebSocket 事件上处理该方法。

```java
        <script type="text/javascript"> 
          var stompClient = null; 
          function setConnected(connected) { 
            document.getElementById('show').disabled = connected; 
          } 
          function connect() { 
            if (window.WebSocket) { 
              message = "supported"; 
              console.log("BROWSER SUPPORTED"); 
            } else { 
              console.log("BROWSER NOT SUPPORTED"); 
            } 
            var country = document.getElementById('country').value; 
            var socket = new WebSocket( 
              "ws://localhost:8081/Ch10_Spring_Message_Handler 
              /webS/myHandler"); 
                socket.onmessage=function(data){ 
                  showResult("Message Arrived"+data.data)        
                }; 
                setConnected(true); 
                socket.onopen = function(e) { 
                    console.log("Connection established!"); 
                    socket.send(country); 
                    console.log("sending data"); 
                };     
          } 
          function disconnect() { 
              if (socket != null) { 
                socket.close(); 
              } 
              setConnected(false); 
              console.log("Disconnected"); 
          } 
          function showResult(message) { 
            var response = document.getElementById('messageDiv'); 
            var p = document.createElement('p'); 
            p.style.wordWrap = 'break-word'; 
            p.appendChild(document.createTextNode(message)); 
            response.appendChild(p); 
          } 
        </script>
```

我们已经讨论过如何编写 WebSocket URL 和事件处理机制。

部署应用程序并访问页面。从下拉列表中选择国家，然后点击显示首都按钮。将显示首都名称的消息。

以下图表显示了应用程序的流程：

![](img/image_10_002.png)

我们添加了控制台日志以及警告消息，以了解进度和消息的往返。根据需求，你可以自定义它，也可以完全省略。

在之前的示例中，我们使用了 WebSocket 进行通信，但其支持仍然有限。SockJS 是一个 JavaScript 库，它提供了类似于 WebSocket 的对象。

## SockJS

****

SockJS 库提供跨浏览器、JavaScript API，以实现浏览器和服务器之间的低延迟、跨域通信。它旨在支持以下目标：

+   使用 SockJS 实例，而不是 WebSocket 实例。

+   这些 API 对于服务器和客户端的 API 来说都非常接近 WebSocket API。

+   支持更快的通信

+   客户端的 JavaScript

+   它带有支持跨域通信的一些选择性协议

以下代码显示了如何为 WebSocketConfigurer 启用 SockJS 支持：

```java
@Override 
public void registerWebSocketHandlers(WebSocketHandlerRegistry  
  registry)  
{ 
  registry.addHandler(myHandler(),  
    "/myHandler_sockjs").setAllowedOrigins("*").withSockJS(); 
} 

```

或者我们可以在 XML 中配置：

```java
<websocket:handlers> 
   <websocket:mapping path="/myHandler"  
     handler="myHandler_sockjs"/> 
   <websocket:sockjs/> 
</websocket:handlers> 

```

我们可以将前面开发的 Capital 演示更新以支持 SockJS，如下所示：

1.  在 WebContent 中添加 country_sockjs.jsp，以便与 SockJS 一起使用，如下所示：

```java
        var socket = new SockJS( 
              "http://localhost:8080/Ch10_Spring_Message_Handler 
        /webS/myHandler_sockjs"); 

```

1.  在 com.packt.ch10.config 包中添加 MyWebSocketConfigurer_sockjs 以配置 WebSocket，就像我们之前做的那样。为了启用 SockJS 支持，我们必须修改`registerWebSocketHandlers()`方法，像上面配置中显示的那样使用`withSockJS()`。

1.  运行应用程序并请求 country_sockjs.jsp 以使用 SockJS。你也可以观察控制台日志。

在上述示例中，我们使用了 WebSocket 来获取连接并处理事件。这里还引入了新的 WebSocket 协议用于通信。它使用更少的带宽。它没有 HTTP 那样的头部，使得通信更简单、高效。我们也可以使用 STOMP 进行通信。

## STOMP

***

**简单（或流式）文本导向消息协议（STOMP）**通过 WebSocket 为 STOMP 帧到 JavaScript 对象的直接映射提供了支持。WebSocket 是最快的协议，但仍然不被所有浏览器支持。浏览器在支持代理和协议处理方面存在问题。所有浏览器广泛支持还需要一段时间，与此同时我们需要找到一些替代方案或实时解决方案。SockJS 支持 STOMP 协议，通过脚本语言与任何消息代理进行通信，是 AMQP 的一个替代方案。STOMP 在客户端和服务器端都很容易实现，并且提供了可靠地发送单条消息的功能，然后断开连接或从目的地消费所有消息。它定义了以下不同的帧，这些帧映射到 WebSocket 帧：

+   **CONNECT（连接客户端和服务器）**：

+   **SUBSCRIBE（用于注册，可以监听给定目的地）**：

+   **UNSUBSCRIBE（用于移除现有订阅）**：

+   **SEND（发送给服务器的消息）**：该帧将消息发送到目的地。

+   **MESSAGE（来自服务器的消息）**：它将来自订阅的消息传递给客户端。

+   **BEGIN（开始事务）**：

+   **COMMIT（提交进行中的事务）**：

+   **ABORT（回滚进行中的事务）**：

+   **DISCONNECT（使客户端与服务器断开连接）**：

它还支持以下标准头：

+   **内容长度（content-length）**：SEND、MESSAGE 和 ERROR 帧包含内容长度头，其值为消息体的内容长度。

+   **内容类型（content-type）**：SEND、MESSAGE 和 ERROR 帧包含内容类型。它在 Web 技术中类似于 MIME 类型。

+   **收据（receipt）**：CONNECT 帧可能包含收据作为头属性，以确认服务器收到 RECEIPT 帧。

+   **心跳（heart-beat）**：它由 CONNECT 和 CONNECTED 帧添加。它包含两个由逗号分隔的正整数值。

+   第一个值代表外出心跳。'0'指定它不能发送心跳。

+   第二个值表示进入心跳。'0'表示不愿意接收心跳。

### Spring STOMP 支持

Spring WebSocket 应用程序作为 STOMP 代理对所有客户端工作。每个消息将通过 Spring 控制器进行路由。这些控制器通过@RequestMapping 注解处理 HTTP 请求和响应。同样，它们也通过@Messaging 注解处理 WebSocket 消息。Spring 还提供了将 RabbitMQ、ActiveMQ 作为 STOMP 代理以进行消息广播的集成。

让我们逐步开发一个使用 STOMP 的应用程序：

1.  创建 Ch10_Spring_Messaging_STOMP 作为一个动态网络应用程序，并添加我们之前添加的 jar 文件。

1.  在 web.xml 中为 DispatcherServlet 添加映射，其名称为 books，URL 模式为'webS'。

1.  添加 books-servlet.xml 以注册`viewResolver`bean。注册以发现控制器，并考虑所有 MVC 注解。

1.  在 com.packt.ch10.config 包中添加 WebSocketConfig_custom 作为一个类，以将`'/book'`作为 SockJS 的端点，将`'/topic'`作为`'/bookApp'`前缀的 SimpleBroker。代码如下：

```java
        @Configuration 
        @EnableWebSocketMessageBroker 
        public class WebSocketConfig_custom extends 
          AbstractWebSocketMessageBrokerConfigurer { 
          @Override 
          public void configureMessageBroker(
            MessageBrokerRegistry config) { 
            config.enableSimpleBroker("/topic"); 
            config.setApplicationDestinationPrefixes("/bookApp"); 
          } 
          @Override 
          public void registerStompEndpoints(
            StompEndpointRegistry registry) { 
            registry.addEndpoint("/book").withSockJS(); 
          } 
        } 

```

`@EnableWebSocketMessageBroker`使类能够作为消息代理。

1.  在 com.packt.ch10.model 包中添加具有 bookName 作为数据成员的 MyBook POJO。

1.  类似地，添加一个结果为数据成员的 Result POJO，其具有 getOffer 方法如下：

```java
        public void getOffer(String bookName) { 
          if (bookName.equals("Spring 5.0")) { 
            result = bookName + " is having offer of having 20% off"; 
            } else if (bookName.equals("Core JAVA")) { 
              result = bookName + " Buy two books and get 10% off"; 
            } else if (bookName.equals("Spring 4.0")) { 
              result = bookName + " is having for 1000 till month  
            end"; 
            } 
            else 
              result = bookName + " is not available on the list"; 
          } 

```

1.  添加 index.html 以从控制器获取'`bookPage`'链接如下：

```java
        <body> 
               <a href="webS/bookPage">CLICK to get BOOK Page</a> 
        </body>
```

1.  在 com.packt.ch10.controller 包中添加 WebSocketController 类，并用@Controller("webs")注解它。

1.  添加注解为@RequestMapping 的`bookPage()`方法，以将 bookPage.jsp 发送给客户端，如下所示：

```java
        @Controller("/webS") 
        public class WebSocketController { 
          @RequestMapping("/bookPage") 
          public String bookPage() { 
            System.out.println("hello"); 
            return "book"; 
        } 

```

1.  在 jsps 文件夹中添加 bookPage.jsp。该页面将显示获取相关优惠的书籍名称。代码如下：

```java
        <body> 
        <div> 
           <div> 
              <button id="connect" 
                onclick="connect();">Connect</button> 
              <button id="disconnect" disabled="disabled"   
                 onclick="disconnect();">Disconnect</button><br/><br/> 
            </div> 
            <div id="bookDiv"> 
                <label>SELECT BOOK NAME</label> 
                 <select id="bookName" name="bookName"> 
                     <option> Core JAVA </option>     
                     <option> Spring 5.0 </option> 
                     <option> Spring 4.0 </option> 
                 </select> 
                <button id="sendBook" onclick="sendBook();">Send to                 Add</button> 
                <p id="bookResponse"></p> 
            </div> 
          </div> 
        </body>
```

1.  一旦客户端点击按钮，我们将处理回调方法，并添加 sockjs 和 STOMP 的脚本如下：

```java
        <script type="text/javascript"                 
         src="img/sockjs-0.3.4.js"></script>            <script type="text/javascript"  
         src="https://cdnjs.cloudflare.com/ajax/libs/stomp.js/2.3.3/ 
        stomp.js"/> 

```

1.  现在我们将逐一添加连接、断开连接、发送、订阅的方法。让我们首先添加如下获取 STOMP 连接的连接方法：

```java
        <script type="text/javascript"> 
           var stompClient = null;  
           function connect() { 
             alert("connection"); 
           if (window.WebSocket){ 
             message="supported"; 
             console.log("BROWSER SUPPORTED"); 
           } else { 
             console.log("BROWSER NOT SUPPORTED"); 
           }                  
           alert(message); 
           var socket = new SockJS('book'); 
           stompClient = Stomp.over(socket); 
           stompClient.connect({}, function(frame) { 
           alert("in client"); 
           setConnected(true); 
           console.log('Connected: ' + frame); 
           stompClient.subscribe('/topic/showOffer',   
             function(bookResult){ 
             alert("subscribing"); 
            showResult(JSON.parse(bookResult.body).result);}); 
          }); 
        } 

```

连接方法创建了一个 SockJS 对象，并使用`Stomp.over()`为 STOMP 协议添加支持。连接添加了`subscribe()`来订阅`'topic/showOffer'`处理器的消息。我们在 WebSocketConfig_custom 类中添加了`'/topic'`作为 SimpleBroker。我们正在处理、发送和接收 JSON 对象。由 Result JSON 对象接收的优惠将以`result: value_of_offer`的形式出现。

1.  添加断开连接的方法如下：

```java
        function disconnect() { 
            stompClient.disconnect(); 
            setConnected(false); 
            console.log("Disconnected"); 
        } 

```

1.  添加 sendBook 以发送获取优惠的请求如下：

```java
        function sendBook()  
        { 
          var bookName =  
          document.getElementById('bookName').value; 
          stompClient.send("/bookApp/book", {},   
            JSON.stringify({ 'bookName': bookName })); 
        } 

```

`send()`向处理程序`/bookApp/book`发送请求，该处理程序将接受具有`bookName`数据成员的 JSON 对象。我们注册了目的地前缀为'`bookApp`'，我们在发送请求时使用它。

1.  添加显示优惠的方法如下：

```java
        function showResult(message) { 
           //similar to country.jsp 
        } 

```

1.  现在让我们在控制器中为'`/book`'添加处理程序方法。此方法将以下面所示的方式注解为`@SendTo("/topic/showOffer'`：

```java
        @MessageMapping("/book") 
          @SendTo("/topic/showOffer") 
          public Result showOffer(MyBook myBook) throws Exception { 
            Result result = new Result(); 
            result.getOffer(myBook.getBookName()); 
            return result; 
        } 

```

1.  部署应用程序。然后点击链接获取优惠页面。

1.  点击“连接”以获取服务器连接。选择书籍以了解优惠并点击发送。与书籍相关的优惠将显示出来。

以下图表解释了应用程序流程：

![](img/image_10_003.png)

在控制台上，日志将以下面的形式显示，展示了 STOMP 的不同帧：

![](img/image_10_004.png)

## 摘要

* * *

在本章中，我们深入讨论了使用 WebSocket 进行消息传递。我们概述了 WebSocket 的重要性以及它与传统网络应用程序以及基于 XMLHttpRequest 的 AJAX 应用程序的区别。我们讨论了 WebSocket 可以发挥重要作用的领域。Spring 提供了与 WebSocket 一起工作的 API。我们看到了 WebSocketHandler、WebSocketConfigurer 以及它们的使用，既使用了 Java 类，也使用了基于 XML 的配置，这些都使用国家首都应用程序来完成。SockJS 库提供了跨浏览器、JavaScript API，以实现浏览器和服务器之间低延迟、跨域通信。我们在 XML 和 Java 配置中都启用了 SockJS。我们还深入了解了 STOMP，它是用于 SockJS 上的 WebSocket 以及如何启用它及其事件处理方法。

在下一章节中，我们将探索反应式网络编程。

![](img/image_01_038.png)

如果您对这本电子书有任何反馈，或者我们在*未覆盖*的方面遇到了困难，请在调查[链接](https://goo.gl/y7BQfO)处告诉我们。

如果您有任何疑虑，您还可以通过以下方式与我们联系：

customercare@packtpub.com

我们会在准备好时发送给您下一章节........!

希望您喜欢我们呈现的内容。
