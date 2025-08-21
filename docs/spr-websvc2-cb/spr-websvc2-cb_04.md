# 第四章：异常/SOAP 故障处理

在本章中，我们将涵盖：

+   通过将异常的消息作为 SOAP 故障字符串返回来处理服务器端异常

+   将异常类名称映射到 SOAP 故障

+   使用`@SOAPFault`对异常类进行注释

+   在 Spring-WS 中编写自己的异常解析器

# 介绍

在处理 Web 服务时生成的服务器端异常被传输为 SOAP 故障。`SOAP <Fault>`元素用于在 SOAP 消息中携带错误和状态信息。

以下代码表示 SOAP 消息中 SOAP 故障元素的一般结构：

```java
<SOAP-ENV:Fault>
<faultcode xsi:type="xsd:string">SOAFP-ENV:Client</faultcode>
<faultstring xsi:type="xsd:string">
A human readable summary of the fault
</faultstring>
<detail xsi:type="xsd:string">
Application specific error information related to the Body element
</detail>
</SOAP-ENV:Fault>

```

如果存在`Fault`元素，则必须作为`Body`元素的子元素出现。SOAP 消息中只能出现一次`Fault`元素。

Spring Web 服务提供了智能机制来处理 SOAP 故障，其易于使用的 API。在处理请求时抛出的异常由`MessageDispatcher`捕捉，并委托给应用程序上下文（XML 或注释中声明的）中声明的任何端点异常解析器。这种异常解析器基于处理机制允许开发人员在抛出特定异常时定义自定义行为（例如返回自定义的 SOAP 故障）。

本章从易于处理异常的机制开始，然后转向稍微复杂的情景。

`org.springframework.ws.server.EndpointExceptionResolver`是 Spring-WS 中服务器端异常处理的主要规范/合同。`org.springframework.ws.soap.server.endpoint.SimpleSoapExceptionResolver`是`EndpointExceptionResolver`的默认实现，可在 Spring-WS 框架中使用。如果开发人员没有明确处理，`MessageDispatcher`将使用`SimpleSoapExceptionResolver`处理服务器端异常。

本章中的示例演示了`org.springframework.ws.server.EndpointExceptionResolver`及其实现的不同用法，包括`SimpleSoapExceptionResolver`。

为了演示目的，构建 Spring-WS 的最简单的方法是使用`MessageDispatcherServlet`简化 WebService 的创建。

# 通过将异常的消息作为 SOAP 故障字符串返回来处理服务器端异常

Spring-WS 框架会自动将服务器端抛出的应用程序级别异常的描述转换为 SOAP 故障消息，并将其包含在响应消息中发送回客户端。本示例演示了捕获异常并设置有意义的消息以作为响应中的 SOAP 故障字符串发送。

## 准备就绪

在此示例中，项目的名称是`LiveRestaurant_R-4.1`（用于服务器端 Web 服务），并具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `log4j-1.2.9.jar`

以下是`LiveRestaurant_R-4.1-Client`（客户端 Web 服务）的 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `spring-test-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

## 如何做...

本示例使用了*通过注释负载根设置端点*中讨论的项目，该项目在第一章中讨论了*构建 SOAP Web 服务*。以下步骤描述了如何修改端点：

1.  修改端点以在应用程序/系统错误发生时抛出异常。

1.  在 Maven 嵌入的 Tomcat 服务器中构建和部署项目。

1.  从项目的根目录在命令行窗口中运行以下命令：

```java
mvn clean package tomcat:run. 

```

1.  要测试，打开一个新的命令窗口，转到文件夹`LiveRestaurant_R-4.1-Client`，并运行以下命令：

```java
mvn clean package exec:java 

```

+   以下是服务器端控制台的输出（请注意消息中生成的`SOAP-Env:Fault`元素）：

```java
DEBUG [http-8080-1] (MessageDispatcher.java:167) - Received request.....
<SOAP-ENV:Fault><faultcode>SOAP-ENV:Server</faultcode>
<faultstring xml:lang="en">Reference number is not provided!</faultstring>
</SOAP-ENV:Fault>
For request
...
<tns:placeOrderRequest >
......
</tns:placeOrderRequest> 

```

+   以下是客户端控制台的输出：

```java
Received response ....
<SOAP-ENV:Fault>
<faultcode>SOAP-ENV:Server</faultcode>
<faultstring xml:lang="en">Reference number is not provided!</faultstring>
</SOAP-ENV:Fault>
... for request....
<tns:placeOrderRequest >
.........
</tns:placeOrderRequest>
....
[WARNING]
java.lang.reflect.InvocationTargetException
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
.........
at org.codehaus.mojo.exec.ExecJavaMojo$1.run(ExecJavaMojo.java:297)
at java.lang.Thread.run(Thread.java:619)
Caused by: org.springframework.ws.soap.client.SoapFaultClientException: Reference number is not provided!
........... 

```

## 它是如何工作的...

在端点（`OrderServiceEndpoint`）的处理程序方法（`handlePlaceOrderRequest`）中，由于传入消息不包含参考编号，因此会抛出一个简单的`RuntimeException`。这象征着任何意外的运行时异常。为了澄清，一个有意义的错误描述**（未提供参考编号！）**被传递给异常：

```java
@PayloadRoot(localPart = "placeOrderRequest", namespace = SERVICE_NS)
public @ResponsePayload
Source handlePlaceOrderRequest(@RequestPayload Source source) throws Exception {
//extract data from input parameter
String fName="John";
String lName="Smith";
String refNumber="";
if(refNumber.length()>0)
return new StringSource(
"<tns:placeOrderResponse xmlns:tns=\"http://www.packtpub.com/liverestaurant/OrderService/schema\"><tns:refNumber>"+orderService.placeOrder(fName, lName, refNumber)+"</tns:refNumber></tns:placeOrderResponse>");
else
throw new RuntimeException("Reference number is not provided!");
}

```

您可以看到，对于这个项目没有配置显式的异常解析器。当没有配置异常解析器时，Spring-WS 框架的智能`MessageDispatcher`会分配一个默认的异常解析器来处理任何异常。它使用`SimpleSoapExceptionResolver`来处理这种情况。

`SimpleSoapExceptionResolver`通过执行以下操作解决异常：

+   将异常记录到日志记录器（控制台，日志文件）

+   生成带有异常消息作为故障字符串的 SOAP 故障消息，并作为响应消息的一部分返回

当我们在客户端检查响应消息时，可以看到在方法`OrderServiceEndpoint.handlePlaceOrderRequest`中设置的确切异常消息（未提供参考**编号！**）作为响应消息中的 SOAP 故障字符串返回。

有趣的是，开发人员不需要做任何处理或发送 SOAP 故障消息，除了抛出一个带有有意义的消息的异常。

## 另请参阅

在第一章中讨论的配方*通过注释有效负载根设置端点*，*构建 SOAP Web 服务*。

在第二章中讨论的配方*在 HTTP 传输上创建 Web 服务客户端*，*构建 SOAP Web 服务的客户端*。

# 将异常类名称映射到 SOAP 故障

Spring-WS 框架允许在 bean 配置文件`spring-ws-servlet.xml`中轻松定制 SOAP 故障消息。它使用一个特殊的异常解析器`SoapFaultMappingExceptionResolver`来完成这项工作。我们可以将异常类映射到相应的 SOAP 故障，以便生成并返回给客户端。

## 准备工作

在这个配方中，项目的名称是`LiveRestaurant_R-4.2`（用于服务器端 Web 服务），具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `log4j-1.2.9.jar`

以下是`LiveRestaurant_R-4.2-Client`（客户端 Web 服务）的 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `spring-test-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

## 如何做...

这个配方使用了*通过注释有效负载根设置端点*中的项目，在第一章中讨论，*构建 SOAP Web 服务*。

1.  创建一个自定义异常类`DataOutOfRangeException.java`。

1.  修改`OrderServiceEndpoint`以抛出`DataOutOfRangeException`。

1.  在`spring-ws-servlet.xml`中注册`SoapFaultMappingExceptionResolver`。

1.  在 Maven 嵌入的 Tomcat 服务器中构建和部署项目。

1.  从项目的根目录，在命令行窗口中运行以下命令：

```java
mvn clean package tomcat:run 

```

1.  要测试，打开一个新的命令窗口，转到文件夹`LiveRestaurant_R-4.2-Client`，并运行以下命令：

```java
mvn clean package exec:java 

```

+   以下是服务器端控制台的输出（请注意，消息中生成了`SOAP-Env:Fault`元素）：

```java
DEBUG [http-8080-1] (MessageDispatcher.java:177) -
Sent response
...
<SOAP-ENV:Fault>
<faultcode>SOAP-ENV:Server</faultcode>
<faultstring xml:lang="en">such a data is out of range!</faultstring>
</SOAP-ENV:Fault>
</SOAP-ENV:Body>
... for request
<tns:placeOrderRequest >
.......
</tns:placeOrderRequest> 

```

+   以下是客户端控制台的输出：

```java
Received response...
<SOAP-ENV:Fault>
<faultcode>SOAP-ENV:Server</faultcode>
<faultstring xml:lang="en">such a data is out of range!</faultstring>
</SOAP-ENV:Fault>
......
for request....
<tns:placeOrderRequest >
.......
</tns:placeOrderRequest>
.....
[WARNING]
java.lang.reflect.InvocationTargetException
.........
Caused by: org.springframework.ws.soap.client.SoapFaultClientException: such a data is out of range!
....... 

```

## 工作原理...

在前面的代码中，`OrderServiceEndpoint.placeOrderRequest`方法抛出一个自定义异常`DataOutOfRangeException`，它象征着典型的服务器端异常：

```java
@PayloadRoot(localPart = "placeOrderRequest", namespace = SERVICE_NS)
public @ResponsePayload
Source handlePlaceOrderRequest(@RequestPayload Source source) throws Exception {
//extract data from input parameter
String fName="John";
String lName="Smith";
String refNumber="123456789";
if(refNumber.length()<7)
return new StringSource(
"<tns:placeOrderResponse xmlns:tns=\"http://www.packtpub.com/liverestaurant/OrderService/schema\"><tns:refNumber>"+orderService.placeOrder(fName, lName, refNumber)+"</tns:refNumber></tns:placeOrderResponse>");
else
throw new DataOutOfRangeException("RefNumber is out of range");
}

```

`MessageDispatcher`捕获了这个异常，并将其委托给配置的异常解析器。在这个项目中，使用了`SoapFaultMappingExceptionResolver`，这是一种特殊的解析器，允许在配置文件中将异常类与自定义消息进行映射。在这个例子中，使用了不同的消息来映射`DataOutOfRangeException`。它充当拦截器，将 SOAP 故障消息转换为以下映射中给定的内容：

```java
<bean id="exceptionResolver"
class="org.springframework.ws.soap.server.endpoint.SoapFaultMappingExceptionResolver">
<property name="defaultFault" value="SERVER" />
<property name="exceptionMappings">
<value>
com.packtpub.liverestaurant.service.exception.DataOutOfRangeException=SERVER,
such a data is out of range!
</value>
</property>
</bean>

```

生成的 SOAP 故障消息在服务器端和客户端控制台屏幕上都有记录。它显示了映射的 SOAP 故障消息，而不是`DataOutOfRangeException`类最初抛出的内容。

## 还有更多...

这个强大的功能可以将异常与 SOAP 故障字符串进行映射，非常有用，可以将 SOAP 故障管理外部化，不需要修改代码并重新构建。此外，如果设计得当，`spring-ws.xml`文件中的配置（SOAP 故障映射）可以作为所有可能的 SOAP 故障消息的单一参考点，可以轻松维护。

### 提示

这是 B2B 应用的一个很好的解决方案。不适用于 B2C，因为需要支持多种语言。一般来说，最好的方法是通过在数据库中配置消息。这样，我们可以在运行时更改和修复它们。在 XML 中配置的缺点是需要重新启动。在实时情况下，一个应用在 30 台服务器上运行。部署和重新启动是痛苦的过程。

## 另请参阅

在第一章中讨论的*通过注释有效负载根设置端点*的配方，*构建 SOAP Web 服务*。

在第二章中讨论的*在 HTTP 传输上创建 Web 服务客户端*配方，*构建 SOAP Web 服务的客户端*

在本章中讨论的*通过将异常消息作为 SOAP 故障字符串返回来处理服务器端异常*的配方。

# 使用@SOAPFault 对异常类进行注释

Spring-WS 框架允许将应用程序异常注释为 SOAP 故障消息，并在异常类本身中进行轻松定制。它使用一个特殊的异常解析器`SoapFaultAnnotationExceptionResolver`来完成这项工作。可以通过在类中进行注释来定制 SOAP 故障字符串和故障代码。

## 准备工作

在这个配方中，项目的名称是`LiveRestaurant_R-4.3`（服务器端 Web 服务），并且具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `log4j-1.2.9.jar`

以下是`LiveRestaurant_R-4.3-Client`（客户端 Web 服务）的 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `spring-test-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `junit-4.7.jar`

+   `xmlunit-1.1.jar`

## 如何做...

这个配方使用了第一章中讨论的*通过注释有效负载根设置端点*的项目作为服务器端，以及第三章中讨论的*如何使用 Spring-Junit 支持集成测试*的配方作为客户端。

1.  创建一个自定义异常类（`InvalidOrdeRequestException.java`），并用`@SoapFault`进行注释。

1.  创建一个自定义异常类（`OrderProcessingFailedException.java`），并用`@SoapFault`进行注释。

1.  修改`Endpoint（OrderServiceEndpoint）`以抛出`InvalidOrderRequestException`和`OrderProcessingFailedException`。

1.  在服务器应用程序上下文文件（`spring-ws-servlet.xml`）中注册`SoapFaultAnnotationExceptionResolver`。

1.  在 Maven 嵌入的 Tomcat 服务器中构建和部署项目。

1.  从项目的根目录，在命令行窗口中运行以下命令：

```java
mvn clean package tomcat:run. 

```

1.  为了测试，打开一个新的命令窗口，进入文件夹`LiveRestaurant_R-4.3-Client`，并运行以下命令：

```java
mvn clean package 

```

+   以下是客户端控制台的输出（请注意消息中生成的 SOAP-Env:Fault 元素）：

```java
DEBUG [main] (WebServiceTemplate.java:632) -
Received response
.....
<SOAP-ENV:Fault><faultcode>SOAP-ENV:Client</faultcode>
<faultstring xml:lang="en">Invalid Order Request: Request message incomplete</faultstring>
</SOAP-ENV>
for request....
<tns:placeOrderRequest ....>
....
</tns:placeOrderRequest>
....................
Received response ...
<SOAP-ENV:Fault><faultcode>SOAP-ENV:Server</faultcode>
<faultstring xml:lang="en">Database server under maintenance, please try after some time.</faultstring>
</SOAP-ENV:Fault>...
for request ...
<tns:cancelOrderRequest ..>
<tns:refNumber>9999</tns:refNumber>
</tns:cancelOrderRequest>
...
Tests run: 2, Failures: 0, Errors: 2, Skipped: 0, Time elapsed: 0.874 sec <<< FAILURE! 

```

## 它是如何工作的...

在端点的方法中，`OrderServiceMethodEndoint.processOrder`（`placeOrderRequest`和`cancelOrderRequest`），抛出自定义异常（`ProcessingFailedException`和`InvalidOrderRequestException`），代表典型的服务器端/客户端异常：

```java
@PayloadRoot(localPart = "placeOrderRequest", namespace = SERVICE_NS)
public @ResponsePayload
Source handlePlaceOrderRequest(@RequestPayload Source source) throws Exception {
//extract data from input parameter
String fName="John";
String lName="Smith";
String refNumber="";
if(refNumber.length()>0)
return new StringSource(
"<tns:placeOrderResponse xmlns:tns=\"http://www.packtpub.com/liverestaurant/OrderService/schema\"><tns:refNumber>"+orderService.placeOrder(fName, lName, refNumber)+"</tns:refNumber></tns:placeOrderResponse>");
else
throw new InvalidOrderRequestException("Reference number is not provided!");
}
@PayloadRoot(localPart = "cancelOrderRequest", namespace = SERVICE_NS)
public @ResponsePayload
Source handleCancelOrderRequest(@RequestPayload Source source) throws Exception {
//extract data from input parameter
boolean cancelled =true ;
if( isDataBaseServerRunning())
return new StringSource(
"<tns:cancelOrderResponse xmlns:tns=\"http://www.packtpub.com/liverestaurant/OrderService/schema\"><cancelled>"+(cancelled?"true":"false")+"</cancelled></tns:cancelOrderResponse>");
else
throw new ProcessingFailedException("Database server is down!");
}
private boolean isDataBaseServerRunning(){
return false;
}

```

这个异常被`MessageDispatcher`捕获并委托给配置的异常解析器。在这个项目中，使用了`SoapFaultAnnotationExceptionResolver`，这是一种特殊的解析器，允许异常类在类中用自定义的故障代码和故障字符串进行注释。`SoapFaultAnnotationExceptionResolver`被配置为在`spring-ws-servlet.xml`中使用，因此任何异常处理都会在运行时由`MessageDispatcherServlet`委托给它：

```java
<bean id="exceptionResolver"
class="org.springframework.ws.soap.server.endpoint.SoapFaultAnnotationExceptionResolver">
<property name="defaultFault" value="SERVER" />
</bean>

```

`ProcessingFailedException`代表服务器端系统异常（faultCode = FaultCode.SERVER）：

```java
@SoapFault(faultCode = FaultCode.SERVER,
faultStringOrReason = "Database server under maintenance, please try after some time.")
public class ProcessingFailedException extends Exception {
public ProcessingFailedException(String message) {
super(message);
}
}

```

`InvalidOrderRequestException`代表客户端业务逻辑异常（faultCode `= FaultCode.CLIENT）：`

```java
@SoapFault(faultCode = FaultCode.CLIENT,
faultStringOrReason = "Invalid Order Request: Request message incomplete")
public class InvalidOrderRequestException extends Exception {
public InvalidOrderRequestException(String message) {
super(message);
}
}

```

您可以看到，带注释的`faultStringOrReason`被生成为 SOAP 故障，并传输回客户端。生成的 SOAP 故障消息在服务器端和客户端控制台屏幕上都被记录，显示了带注释的 SOAP 故障消息，而不是在`Endpoint`类中最初抛出的内容。

## 还有更多...

`@SoapFault`注释的`faultCode`属性具有以下可能的枚举值：

+   `CLIENT`

+   `CUSTOM`

+   `RECEIVER`

+   `SENDER`

从枚举列表中选择一个指示调度程序应生成哪种 SOAP 故障以及其具体内容。根据前面的选择，依赖属性变得强制性。

例如，如果为`faultCode`选择了`FaultCode.CUSTOM`，则必须使用`customFaultCode`字符串属性，而不是`faultStringOrReason`，如本配方的代码片段中所示。用于`customFaultCode`的格式是`QName.toString()`的格式，即`"{" + Namespace URI + "}" + local part`，其中命名空间是可选的。请注意，自定义故障代码仅在 SOAP 1.1 上受支持。

`@SoaPFault`注释还有一个属性，即 locale，它决定了 SOAP 故障消息的语言。默认语言环境是英语。

### 注意

在一般实践中，我们使用错误代码而不是错误消息。在客户端使用映射信息进行映射。这样可以避免对网络的任何负载，并且不会出现多语言支持的问题。

## 另请参阅

在第一章中讨论的*通过注释有效负载根设置端点*的配方，*构建 SOAP Web 服务。*

在第三章中讨论的*如何使用 Spring-JUnit 支持集成测试*的配方，*测试和监控 Web 服务。*

在本章中讨论的*将异常类名映射到 SOAP 故障*的配方。

# 在 Spring-WS 中编写自己的异常解析器

Spring-WS 框架提供了默认机制来处理异常，使用标准异常解析器，它允许开发人员通过构建自己的异常解析器来以自己的方式处理异常。SOAP 故障可以定制以在其自己的格式中添加自定义细节，并传输回客户端。

本示例说明了一个自定义异常解析器，它将异常堆栈跟踪添加到 SOAP 响应的 SOAP 故障详细元素中，以便客户端获取服务器端异常的完整堆栈跟踪，对于某些情况非常有用。这个自定义异常解析器已经具有注释的功能，就像前面的示例一样。

## 准备就绪

在本配方中，项目的名称是`LiveRestaurant_R-4.4`（用于服务器端 Web 服务），具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `log4j-1.2.9.jar`

`LiveRestaurant_R-4.4-Client`（用于客户端）具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `spring-test-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `junit-4.7.jar`

+   `xmlunit-1.1.jar`

## 如何做...

本配方使用了*通过注释 payload-root 设置端点*中讨论的项目，第一章，*构建 SOAP Web 服务*。

1.  创建一个自定义异常解析器`DetailedSoapFaultExceptionResolver`，扩展`SoapFaultAnnotationExceptionResolver`。

1.  在`spring-ws-servlet.xml`中注册`DetailedSoapFaultExceptionResolver`。

1.  在 Maven 嵌入的 Tomcat 服务器中构建和部署项目。

1.  从项目的根目录，在命令行窗口中运行以下命令：

```java
mvn clean package tomcat:run. 

```

1.  要进行测试，请打开一个新的命令窗口，转到文件夹`LiveRestaurant_R-4.4-Client`，并运行以下命令：

```java
mvn clean package exec:java 

```

+   以下是服务器端控制台的输出（请注意在消息中生成的`SOAP-Env:Fault`元素）：

```java
DEBUG [http-8080-1] (MessageDispatcher.java:167) - Received request.....
<tns:placeOrderRequest >
......
</tns:placeOrderRequest></SOAP-ENV:Body>...
DEBUG [http-8080-1] (MessageDispatcher.java:177) - Sent response
...
<SOAP-ENV:Fault><faultcode>SOAP-ENV:Client</faultcode>
<faultstring xml:lang="en">Invalid Order Request: Request message incomplete</faultstring
><detail>
<stack-trace >
at com.packtpub.liverestaurant.service.endpoint.OrderSeviceEndpoint.handlePlaceOrderRequest(OrderSeviceEndpoint.java:43)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:39)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:25)
at java.lang.reflect.Method.invoke(Method.java:597)
at org.springframework.ws.server.endpoint.MethodEndpoint.invoke(MethodEndpoint.java:132)
at org.springframework.ws.server.endpoint.adapter.DefaultMethodEndpointAdapter.invokeInternal(DefaultMethodEndpointAdapter.java:229)
at org.springframework.ws.server.endpoint.adapter.AbstractMethodEndpointAdapter.invoke(AbstractMethodEndpointAdapter.java:53)
at org.springframework.ws.server.MessageDispatcher.dispatch(MessageDispatcher.java:230)
....... 
</stack-trace></detail></SOAP-ENV:Fault>

```

## 工作原理...

在上述代码中，我们的自定义异常解析器`DetailedSoapFaultExceptionResolver`是`SoapFaultAnnotationExceptionResolver`的子类，覆盖了方法`custmizeFault()`，将异常堆栈跟踪添加到 SOAP 故障详细元素中。方法`stackTraceToString()`从给定的异常返回异常堆栈跟踪，并用于将堆栈跟踪设置为响应消息的 SOAP 故障的详细元素。

## 还有更多...

有许多不同的创建自定义异常解析器的方法。不仅可以继承`SoapFaultAnnotationExceptionResolver`来实现这一目的。任何`org.springframework.ws.server.EndpointExceptionResolver`的实现都可以适当配置为用作异常解析器。开发人员可以从 Spring-WS API 中提供的一组非常方便的`EndpointExceptionResolver`实现中进行选择，利用这些实现的功能。

自定义这些类的位置是方法`customizeFault`。可以通过覆盖方法`customizeFault`来自定义 SOAP 故障。查看包`org.springframework.ws.soap.server.endpoint`，以获取适合您要求的现成异常解析器。

如果需要开发一个与当前可用实现不符的专门自定义异常解析器，则`AbstractSoapFaultDefinitionExceptionResolver`将是一个理想的起点，因为它已经实现了一些任何异常解析器都需要的非常常见和基本功能。开发人员只需实现抽象方法`resolveExceptionInternal()`，以满足特定需求。

需要注意的是，应该指示`MessageDispatcherServlet`考虑使用的解析器，可以通过在`spring-ws-servlet.xml`中注册或在异常类中进行注释（除了在`spring-ws-servlet.xml`中注册）。

## 另请参阅

本配方*通过注释 payload-root 设置端点*中讨论的项目，第一章，*构建 SOAP Web 服务*。

本章讨论的配方*使用@SOAP fault 注释异常类*。
