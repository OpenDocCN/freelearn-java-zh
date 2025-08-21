# 第二章：为 SOAP Web 服务构建客户端

在本章中，我们将涵盖：

+   在 Eclipse 中设置 Web 服务客户端开发环境

+   使用 Maven 设置 Web 服务客户端开发环境

+   在 HTTP 传输上创建 Web 服务客户端

+   在 JMS 传输上创建 Web 服务客户端

+   在 E-mail 传输上创建 Web 服务客户端

+   在 XMPP 传输上创建 Web 服务客户端

+   使用 XPath 表达式创建 Web 服务客户端

+   为 WS-Addressing 端点创建 Web 服务客户端

+   使用 XSLT 转换 Web 服务消息

# 介绍

使用 Java API，如`SAAJ`，可以生成客户端 SOAP 消息，并将其传输到/从 Web 服务。但是，这需要额外的编码和关于 SOAP 消息的知识。

`org.springframework.ws.client.core`包含了客户端 API 的核心功能，可以简化调用服务器端 Web 服务。

这个包中的 API 提供了像`WebServiceTemplate`这样的模板类，简化了 Web 服务的使用。使用这些模板，您将能够在各种传输协议（HTTP、JMS、电子邮件、XMPP 等）上创建 Web 服务客户端，并发送/接收 XML 消息，以及在发送之前将对象编组为 XML。Spring 还提供了一些类，如`StringSource`和`Result`，简化了在使用`WebServiceTemplate`时传递和检索 XML 消息。

在本章中，前两个教程解释了如何在 Eclipse 和 Maven 中设置调用 Web 服务客户端的环境。

然后我们将讨论如何使用`WebServiceTemplate`在各种传输协议（HTTP、JMS、电子邮件、XMPP 等）上创建 Web 服务客户端。除此之外，*使用 XPath 表达式设置 Web 服务客户端*这个教程解释了如何从 XML 消息中检索数据。最后，在最后一个教程*使用 XSLT 转换 Web 服务消息*中，介绍了如何在客户端和服务器之间将 XML 消息转换为不同格式。为了设置 Web 服务服务器，使用了第一章中的一些教程，*构建 SOAP Web 服务*，并创建了一个单独的客户端项目，调用服务器端的 Web 服务。

# 在 Eclipse 中设置 Web 服务客户端开发环境

最简单的 Web 服务客户端是调用服务器端 Web 服务的 Java 类。在这个教程中，介绍了设置调用服务器端 Web 服务的环境。在这里，客户端的 Java 类以两种形式调用服务器端的 Web 服务。第一种是在类的主方法中调用 Web 服务的 Java 类。第二种是使用 JUnit 测试类调用服务器端的 Web 服务。

## 准备工作

这个教程类似于第一章中讨论的*使用 Maven 构建和运行 Spring-WS*这个教程，*构建 SOAP Web 服务*。

1.  下载并安装 Java EE 开发人员 Helios 的 Eclipse IDE。

1.  在这个教程中，项目的名称是`LiveRestaurant_R-2.1`（服务器端 Web 服务），具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `jdom-1.0.jar`

+   `log4j-1.2.9.jar`

+   `jaxen-1.1.jarb`

+   `xalan-2.7.0.jar`

1.  `LiveRestaurant_R-2.1-Client`（客户端）具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `jdom-1.0.jar`

+   `log4j-1.2.9.jar`

+   `jaxen-1.1.jar`

+   `xalan-2.7.0.jar`

+   `junit-4.7.jar`

1.  运行以下 Maven 命令，以便将客户端项目导入 Eclipse（客户端）：

```java
mvn eclipse:eclipse -Declipse.projectNameTemplate="LiveRestaurant_R-2.1-Client" 

```

## 如何做...

这个教程使用了第一章中讨论的*使用 JDOM 处理传入的 XML 消息*这个教程，*构建 SOAP Web 服务*作为服务器端项目。

1.  在主方法中运行调用 Web 服务的 Java 类。

1.  通过转到**File** | **Import** | **General** | **Existing projects into workspace** | **LiveRestaurant_R-2..1-Client**，将`LiveRestaurant_R-2.1-Client`导入 Eclipse 工作区。

1.  转到命令提示符中的`LiveRestaurant_R-2.1`文件夹，并使用以下命令运行服务器：

```java
mvn clean package tomcat:run 

```

1.  在`com.packtpub.liverestaurant.client`包的`src/main/java`文件夹中选择`OrderServiceClient`类，然后选择**Run As** | **Java Application**。

在客户端上运行 Java 类时的控制台输出如下：

```java
Received response ....
<tns:placeOrderResponse > <tns:refNumber>order-John_Smith_9999</tns:refNumber>
</tns:placeOrderResponse>
for request...
<tns:placeOrderRequest >
<tns:order>
<tns:refNumber>9999</tns:refNumber>
<tns:customer>
.......
</tns:customer>
<tns:dateSubmitted>2008-09-29T05:49:45</tns:dateSubmitted>
<tns:orderDate>2014-09-19T03:18:33</tns:orderDate>
<tns:items>
<tns:type>Snacks</tns:type>
<tns:name>Pitza</tns:name>
<tns:quantity>2</tns:quantity>
</tns:items>
</tns:order>
</tns:placeOrderRequest>.... 

```

1.  在 Eclipse 中运行一个 JUnit 测试用例。

1.  在`com.packtpub.liverestaurant.client`包的`src/test/java`文件夹中选择`OrderServiceClientTest`类，然后选择**Run As** | **Junit Test**。

运行 JUnit 测试用例时的控制台输出如下（您可以单击**Console**标签旁边的**JUnit**标签，查看测试用例是否成功）：

```java
Received response ..
<tns:placeOrderResponse >
<tns:refNumber>order-John_Smith_9999</tns:refNumber>
</tns:placeOrderResponse>..
......
<tns:placeOrderRequest >
<tns:order>
<tns:refNumber>9999</tns:refNumber>
<tns:customer>
......
</tns:customer>
<tns:dateSubmitted>2008-09-29T05:49:45</tns:dateSubmitted>
<tns:orderDate>2014-09-19T03:18:33</tns:orderDate>
<tns:items>
<tns:type>Snacks</tns:type>
<tns:name>Pitza</tns:name>
<tns:quantity>2</tns:quantity>
</tns:items>
</tns:order>
</tns:placeOrderRequest> 

```

### 注意

要传递参数或自定义测试的设置，请选择测试单元类，**Run As** | **Run Configuration** |，然后在左窗格上双击**JUnit**。

然后，您将能够自定义传递的参数或设置并运行客户端。

## 工作原理...

当运行调用 Web 服务的 Java 类的主方法时，Eclipse 通过以下 Java 类路径内部运行以下命令：

```java
java -classpath com.packtpub.liverestaurant.client.OrderServiceClient 

```

当运行 JUnit 测试用例时，Eclipse 通过内部调用以下命令来运行 JUnit 框架的测试用例：

```java
java -classpath com.packtpub.liverestaurant.client.OrderServiceClientTest 

```

## 另请参阅

在第一章中讨论的*使用 Maven 构建和运行 Spring-WS 项目*和*使用 JDOM 处理传入的 XML 消息*配方，

本章讨论的*使用 HTTP 传输创建 Web 服务客户端*配方。

# 使用 Maven 设置 Web 服务客户端开发环境

Maven 支持使用命令提示符运行类的主方法以及 JUnit 测试用例。

在这个配方中，解释了设置 Maven 环境以调用客户端 Web 服务。在这里，客户端 Java 代码以两种形式调用服务器上的 Web 服务。第一种是在类的主方法中调用 Web 服务的 Java 类。第二种使用 JUnit 调用服务器端 Web 服务。

## 准备工作

在这个配方中，项目的名称是`LiveRestaurant_R-2.2`（用于服务器端 Web 服务），具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `log4j-1.2.9.jar`

以下是`LiveRestaurant_R-2.2-Client`（客户端 Web 服务）的 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `junit-4.7.jar`

## 如何做...

这个配方使用了第一章中讨论的*使用 DOM 处理传入的 XML 消息*配方，*构建 SOAP Web 服务*作为服务器端项目。

1.  在主方法中运行调用 Web 服务的 Java 类。

1.  转到命令提示符中的`LiveRestaurant_R-2.2`文件夹，并使用以下命令运行服务器：

```java
mvn clean package tomcat:run 

```

1.  转到文件夹`LiveRestaurant_R-2.2-Client`并运行以下命令：

```java
mvn clean package exec:java 

```

+   在客户端上运行 Maven 命令时，以下是输出：

```java
Received response ....
<placeOrderResponse >
<refNumber>order-John_Smith_9999</refNumber>
</placeOrderResponse>....
<tns:placeOrderRequest >
<tns:order>
<tns:refNumber>9999</tns:refNumber>
<tns:customer>
.....
</tns:customer>
<tns:dateSubmitted>2008-09-29T05:49:45</tns:dateSubmitted>
<tns:orderDate>2014-09-19T03:18:33</tns:orderDate>
<tns:items>
<tns:type>Snacks</tns:type>
<tns:name>Pitza</tns:name>
<tns:quantity>2</tns:quantity>
</tns:items>
</tns:order>
</tns:placeOrderRequest> 

```

1.  使用 Maven 运行 JUnit 测试用例。

1.  转到命令提示符中的`LiveRestaurant_R-2.2`文件夹，并使用以下命令运行服务器：

```java
mvn clean package tomcat:run 

```

1.  转到文件夹`LiveRestaurant_R-2.2-Client`并运行以下命令：

```java
mvn clean package 

```

+   在客户端上使用 Maven 运行 JUnit 测试用例后，以下是输出：

```java
Received response ...
<placeOrderResponse >
<refNumber>order-John_Smith_9999</refNumber>
</placeOrderResponse>...
for request ...
<tns:placeOrderRequest >
<tns:order>
<tns:refNumber>9999</tns:refNumber>
<tns:customer>
.....
</tns:customer>
<tns:dateSubmitted>2008-09-29T05:49:45</tns:dateSubmitted>
<tns:orderDate>2014-09-19T03:18:33</tns:orderDate>
<tns:items>
<tns:type>Snacks</tns:type>
<tns:name>Pitza</tns:name>
<tns:quantity>2</tns:quantity>
</tns:items>
</tns:order>
</tns:placeOrderRequest></SOAP-ENV:Body></SOAP-ENV:Envelope>]
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.702 sec
Results :
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0 

```

## 工作原理...

在`pom.xml`文件中设置`exec-maven-plugin`，告诉 Maven 运行`OrderServiceClient`的`mainClass`的 Java 类。该 Java 类告诉 Maven 运行`OrderServiceClient`的`mainClass`：

```java
<build>
<finalName>LiveRestaurant_Client</finalName>
<plugins>
.......
</plugin>
<plugin>
<groupId>org.codehaus.mojo</groupId>
<artifactId>exec-maven-plugin</artifactId>
<version>1.2.1</version>
<executions>
<execution>
<goals>
<goal>java</goal>
</goals>
</execution>
</executions>
<configuration>
<mainClass>com.packtpub.liverestaurant.client.OrderServiceClient</mainClass>
</configuration>
</plugin>
</plugins>
</build>

```

Maven 通过项目类路径内部运行以下命令：

```java
java -classpath com.packtpub.liverestaurant.client.OrderServiceClient

```

要在 Maven 中设置和运行 JUnit 测试用例，测试类 `OrderServiceClientTest` 应该包含在文件夹 `src/test/java` 中，并且测试类名称应该以 `Test` 结尾（`OrderServiceClientTest`）。命令 `mvn clean package` 运行 `src/test/java` 文件夹中的所有测试用例（内部 Maven 调用）：

```java
java -classpath ...;junit.jar.. junit.textui.TestRunner com.packtpub.liverestaurant.client.OrderServiceClientTest ) . 

```

## 另请参阅

在第一章中讨论的*使用 Maven 构建和运行 Spring-WS 项目*和*使用 JDOM 处理传入的 XML 消息*的配方，*构建 SOAP Web 服务*。

在本章中讨论的*在 HTTP 传输上创建 Web 服务客户端*的配方。

# 在 HTTP 传输上创建 Web 服务客户端

在这个配方中，`WebServiceTemplate` 用于通过 HTTP 传输从客户端发送/接收简单的 XML 消息。

## 准备工作

在这个配方中，项目的名称是 `LiveRestaurant_R-2.3`（用于服务器端 Web 服务），具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `log4j-1.2.9.jar`

以下是 `LiveRestaurant_R-2.3-Client`（客户端 Web 服务）的 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `junit-4.7.jar`

## 如何做到...

这个配方使用了在第一章中讨论的*通过注释负载根来设置端点*的配方，*构建 SOAP Web 服务*，作为服务器端项目。以下是如何设置客户端：

1.  创建一个调用 `WebServiceTemplate` 中的 Web 服务服务器的类在 `src/test` 中。

1.  在 `applicationContext.xml` 文件中配置 `WebServiceTemplate`。

1.  从文件夹 `Liverestaurant_R-2.3` 运行以下命令：

```java
mvn clean package tomcat:run 

```

1.  打开一个新的命令窗口到 `Liverestaurant_R-2.3-Client` 并运行以下命令：

```java
mvn clean package 

```

+   以下是客户端输出：

```java
Received response ....
<tns:placeOrderResponse >
<tns:refNumber>order-John_Smith_1234</tns:refNumber>
</tns:placeOrderResponse>...
<tns:placeOrderRequest >
<tns:order>
<tns:refNumber>9999</tns:refNumber>
<tns:customer>
......
</tns:customer>
<tns:dateSubmitted>2008-09-29T05:49:45</tns:dateSubmitted>
<tns:orderDate>2014-09-19T03:18:33</tns:orderDate>
<tns:items>
<tns:type>Snacks</tns:type>
<tns:name>Pitza</tns:name>
<tns:quantity>2</tns:quantity>
</tns:items>
</tns:order>
</tns:placeOrderRequest>
.....
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.749 sec
Results :
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0 

```

## 它是如何工作的...

`Liverestaurant_R-2.3` 是一个服务器端项目，它重复使用了在第一章中讨论的*通过注释负载根来设置端点*的配方。*构建 SOAP Web 服务*。

已配置的客户端 `WebServiceTemplate` 的 `applicationContext.xml` 文件（`id="webServiceTemplate"`）用于发送和接收 XML 消息。可以从客户端程序中获取此 bean 的实例以发送和接收 XML 消息。

`messageFactory` 是 `SaajSoapMessageFactory` 的一个实例，它被引用在 `WebServiceTemplate` 内。`messageFactory` 用于从 XML 消息创建 SOAP 数据包。默认的服务 URI 是 `WebServiceTemplate` 默认使用的 URI，用于发送/接收所有请求/响应：

```java
<bean id="messageFactory" class="org.springframework.ws.soap.saaj.SaajSoapMessageFactory" />
<bean id="webServiceTemplate" class="org.springframework.ws.client.core.WebServiceTemplate">
<constructor-arg ref="messageFactory" />
<property name="defaultUri" value="http://localhost:8080/LiveRestaurant/spring-ws/OrderService" />
</bean>

```

`OrderServiceClientTest.java` 是一个简单的 JUnit 测试用例，用于在 `setUpBeforeClass()` 方法中从 `applicationContext.xml` 中获取和初始化 `WebServiceTemplate`（由 `@BeforeClass` 标记）。在 `testCancelOrderRequest` 和 `testPlaceOrderRequest` 方法中（由 `@Test` 标记），`WebServiceTemplate` 发送一个简单的 XML 消息（由现有输入 XML 文件的 `StringSource` 对象创建），并接收包装在 `Result` 对象中的来自服务器的响应：

```java
private static WebServiceTemplate wsTemplate = null;
private static InputStream isPlace;
private static InputStream isCancel;
@BeforeClass
public static void setUpBeforeClass() throws Exception {
ClassPathXmlApplicationContext appContext = new ClassPathXmlApplicationContext("/applicationContext.xml");
wsTemplate = (WebServiceTemplate) appContext.getBean("webServiceTemplate");
isPlace = new OrderServiceClientTest().getClass().getResourceAsStream("placeOrderRequest.xml");
isCancel = new OrderServiceClientTest().getClass().getResourceAsStream("cancelOrderRequest.xml");
}
@Test
public final void testPlaceOrderRequest() throws Exception {
Result result = invokeWS(isPlace);
Assert.assertTrue(result.toString().indexOf("placeOrderResponse")>0);
}
@Test
public final void testCancelOrderRequest() throws Exception {
Result result = invokeWS(isCancel);
Assert.assertTrue(result.toString().indexOf("cancelOrderResponse")>0);
}
private static Result invokeWS(InputStream is) {
StreamSource source = new StreamSource(is);
StringResult result = new StringResult();
wsTemplate.sendSourceAndReceiveToResult(source, result);
return result;
}

```

## 另请参阅

在第一章中讨论的*通过注释负载根来设置端点*的配方，*构建 SOAP Web 服务*和在本章中讨论的*使用 Maven 设置 Web 服务客户端开发环境*的配方。

# 在 JMS 传输上创建 Web 服务客户端

JMS（Java 消息服务）于 1999 年由 Sun Microsystems 作为 Java 2、J2EE 的一部分引入。使用 JMS 的系统可以同步或异步通信，并基于点对点和发布-订阅模型。Spring Web 服务提供了在 Spring 框架中基于 JMS 功能构建 JMS 协议的 Web 服务的功能。Spring Web 服务在 JMS 协议上提供以下通信功能：

+   客户端和服务器可以断开连接，只有在发送/接收消息时才能连接

+   客户端不需要等待服务器回复（例如，如果服务器需要很长时间来处理，例如进行复杂的数学计算）

+   JMS 提供了确保客户端和服务器之间消息传递的功能

在这个配方中，`WebServiceTemplate`用于在客户端上通过 JMS 传输发送/接收简单的 XML 消息。使用一个 JUnit 测试用例类在服务器端设置并使用`WebServiceTemplate`发送和接收消息。

## 准备工作

在这个配方中，项目的名称是`LiveRestaurant_R-2.4`，具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `spring-ws-support-2.0.1.RELEASE.jar`

+   `spring-test-3.0.5.RELEASE.jar`

+   `spring-jms-3.0.5.RELEASE.jar`

+   `junit-4.7.jar`

+   `xmlunit-1.1.jar`

+   `log4j-1.2.9.jar`

+   `jms-1.1.jar`

+   `activemq-core-4.1.1.jar`

## 如何做...

本配方使用在第一章中讨论的配方*在 JMS 传输上设置 Web 服务*，*构建 SOAP Web 服务*作为服务器端项目。

1.  创建一个调用`WebServiceTemplate`的 Web 服务服务器的 JUnit 测试类。

1.  在`applicationContext`中配置`WebServiceTemplate`以通过 JMS 协议发送消息。

1.  运行命令`mvn clean package`。您将看到以下输出：

```java
Received response ..
<tns:placeOrderResponse >
<tns:refNumber>order-John_Smith_1234</tns:refNumber>
</tns:placeOrderResponse>....
<tns:placeOrderRequest >
<tns:order>
<tns:refNumber>9999</tns:refNumber>
<tns:customer>
.....
</tns:customer>
<tns:dateSubmitted>2008-09-29T05:49:45</tns:dateSubmitted>
<tns:orderDate>2014-09-19T03:18:33</tns:orderDate>
<tns:items>
<tns:type>Snacks</tns:type>
<tns:name>Pitza</tns:name>
<tns:quantity>2</tns:quantity>
</tns:items>
</tns:order>
</tns:placeOrderRequest> 

```

## 它是如何工作的...

在这个项目中，我们使用一个 JUnit 类在 JMS 传输上设置 Web 服务服务器。服务器使用`PayloadEndpoint`接收 XML 请求消息，并返回一个简单的 XML 消息作为响应（服务器已经在第一章中讨论的配方*在 JMS 传输上设置 Web 服务*，*构建 SOAP Web 服务*中描述）。

已配置客户端`WebServiceTemplate`的`applicationContext.xml`文件（`id="webServiceTemplate"`）用于发送和接收 XML 消息。可以从客户端程序中获取此 bean 的实例以发送和接收 XML 消息。`messageFactory`是`SaajSoapMessageFactory`的一个实例，被引用在`WebServiceTemplate`内。`messageFactory`用于从 XML 消息创建 SOAP 数据包。默认服务 URI 是`WebServiceTemplate`默认使用的 JMS URI，用于发送/接收所有请求/响应。配置在`WebServiceTemplate`内的`JmsMessageSender`用于发送 JMS 消息。要使用`JmsMessageSender`，`defaultUri`或`JMS URI`应包含`jms:`前缀和目的地名称。一些`JMS URI`的例子是`jms:SomeQueue, jms:SomeTopic?priority=3&deliveryMode=NON_PERSISTENT, jms:RequestQueue?replyToName=ResponseName`等。默认情况下，`JmsMessageSender`发送 JMS`BytesMessage`，但可以通过在 JMS URI 上使用`messageType`参数来覆盖使用`TextMessages`。例如，`jms:Queue?messageType=TEXT_MESSAGE`。

```java
<bean id="webServiceTemplate" class="org.springframework.ws.client.core.WebServiceTemplate">
<constructor-arg ref="messageFactory"/>
<property name="messageSender">
<bean class="org.springframework.ws.transport.jms.JmsMessageSender">
<property name="connectionFactory" ref="connectionFactory"/>
</bean>
</property>
<property name="defaultUri" value="jms:RequestQueue?deliveryMode=NON_PERSISTENT"/>
</bean>

```

`JmsTransportWebServiceIntegrationTest.java`是一个 JUnit 测试用例，从`applicationContext.xml`文件中获取并注入`WebServiceTemplate`（由`@ContextConfiguration("applicationContext.xml")`标记）。在`testSendReceive()`方法（由`@Test`标记），`WebServiceTemplate`发送一个简单的 XML 消息（由简单输入字符串的`StringSource`对象创建），并接收包装在`Result`对象中的服务器响应。在`testSendReceive()`方法（由`@Test`标记）中，发送和接收消息类似于 HTTP 客户端，并使用`WebServiceTemplate.sendSourceAndReceiveToResult`发送/接收消息：

```java
@Test
public void testSendReceive() throws Exception {
InputStream is = new JmsTransportWebServiceIntegrationTest().getClass().getResourceAsStream("placeOrderRequest.xml");
StreamSource source = new StreamSource(is);
StringResult result = new StringResult();
webServiceTemplate.sendSourceAndReceiveToResult(source, result);
XMLAssert.assertXMLEqual("Invalid content received", expectedResponseContent, result.toString());
}

```

## 另请参阅

在第一章中讨论的配方*在 JMS 传输上设置 Web 服务*，*构建 SOAP Web 服务*。

*使用 Spring Junit 对 Web 服务进行单元测试*

# 在 E-mail 传输上创建 Web 服务客户端

在这个示例中，`WebServiceTemplate` 用于在客户端上通过电子邮件传输发送/接收简单的 XML 消息。使用第一章中讨论的 *在电子邮件传输上设置 Web 服务* 这个示例，*构建 SOAP Web 服务* 来设置 Web 服务。使用 JUnit 测试用例类来在服务器端设置 Web 服务，并使用 `WebServiceTemplate` 发送/接收消息。

## 准备工作

在这个示例中，项目的名称是 `LiveRestaurant_R-2.5`，具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `spring-ws-support-2.0.1.RELEASE.jar`

+   `spring-test-3.0.5.RELEASE.jar`

+   `mail-1.4.1.jar`

+   `mock-javamail-1.6.jar`

+   `junit-4.7.jar`

+   `xmlunit-1.1.jar`

## 如何做...

这个示例使用第一章中讨论的 *在电子邮件传输上设置 Web 服务* 这个示例，*构建 SOAP Web 服务* 作为服务器端项目。

1.  创建一个测试类，使用 `WebServiceTemplate` 调用 Web 服务服务器。

1.  在 `applicationContext` 中配置 `WebServiceTemplate` 以通过电子邮件协议发送消息。

1.  运行命令 `mvn clean package`。以下是此命令的输出：

```java
Received response
<tns:placeOrderResponse >
<tns:refNumber>order-John_Smith_1234</tns:refNumber>
</tns:placeOrderResponse>....
<tns:placeOrderRequest >
<tns:order>
<tns:refNumber>9999</tns:refNumber>
<tns:customer>
.....
</tns:customer>
<tns:dateSubmitted>2008-09-29T05:49:45</tns:dateSubmitted>
<tns:orderDate>2014-09-19T03:18:33</tns:orderDate>
<tns:items>
<tns:type>Snacks</tns:type>
<tns:name>Pitza</tns:name>
<tns:quantity>2</tns:quantity>
</tns:items>
</tns:order>
</tns:placeOrderRequest> 

```

## 它是如何工作的...

该项目通过 JUnit 类在电子邮件传输上设置 Web 服务服务器。这个类使用 Spring JUnit 来加载应用程序上下文，首先设置服务器，然后运行客户端单元测试以验证其是否按预期运行。服务器已在第一章中讨论的 *在电子邮件传输上设置 Web 服务* 这个示例中解释过。

配置的客户端 `WebServiceTemplate (id="webServiceTemplate")` 的 `applicationContext.xml` 文件用于发送和接收 XML 消息。可以从客户端程序中获取此 bean 的实例以发送和接收 XML 消息。`messageFactory` 是 `SaajSoapMessageFactory` 的一个实例，被引用在 `WebServiceTemplate` 内。`messageFactory` 用于从 XML 消息创建 SOAP 数据包。`transportURI` 是一个由 `WebServiceTemplate` 使用的 URI，指示用于发送请求的服务器。`storeURI` 是一个 URI，配置在 `WebServiceTemplate` 内，指示用于轮询响应的服务器（通常是 POP3 或 IMAP 服务器）。默认 URI 是 `WebServiceTemplate` 默认使用的电子邮件地址 URI，用于发送/接收所有请求/响应：

```java
<bean id="webServiceTemplate" class="org.springframework.ws.client.core.WebServiceTemplate">
<constructor-arg ref="messageFactory"/>
<property name="messageSender">
<bean class="org.springframework.ws.transport.mail.MailMessageSender">
<property name="from" value="client@packtpubtest.com"/>
<property name="transportUri" value="smtp://smtp.packtpubtest.com"/>
<property name="storeUri" value="imap://client@packtpubtest.com/INBOX"/>
<property name="receiveSleepTime" value="1500"/>
<property name="session" ref="session"/>
</bean>
</property>
<property name="defaultUri" value="mailto:server@packtpubtest.com"/>
</bean>
<bean id="session" class="javax.mail.Session" factory-method="getInstance">
<constructor-arg>
<props/>
</constructor-arg>
</bean>

```

`MailTransportWebServiceIntegrationTest.java` 是一个 JUnit 测试用例，从 `applicationContext.xml` 中获取并注入 `WebServiceTemplate`（由 `@ContextConfiguration("applicationContext.xml")` 标记）。在 `testWebServiceOnMailTransport()` 方法中（由 `@Test` 标记），`WebServiceTemplate` 发送一个简单的 XML 消息（由输入 XML 文件的 `StringSource` 对象创建），并接收包装在 `Result` 对象中的来自服务器的响应。

```java
@Test
public void testWebServiceOnMailTransport() throws Exception {
InputStream is = new MailTransportWebServiceIntegrationTest().getClass().getResourceAsStream("placeOrderRequest.xml");
StreamSource source = new StreamSource(is);
StringResult result = new StringResult();
webServiceTemplate.sendSourceAndReceiveToResult(source, result);
applicationContext.close();
XMLAssert.assertXMLEqual("Invalid content received", expectedResponseContent, result.toString());
}

```

## 另请参阅..

在第一章中讨论的 *在电子邮件传输上设置 Web 服务* 这个示例，*构建 SOAP Web 服务*。

使用 Spring Junit 对 Web 服务进行单元测试

# 在 XMPP 传输上设置 Web 服务

**XMPP**（可扩展消息和出席协议）是一种开放和分散的 XML 路由技术，系统可以使用它向彼此发送 XMPP 消息。XMPP 网络由 XMPP 服务器、客户端和服务组成。使用 XMPP 的每个系统都由唯一的 ID（称为**Jabber ID (JID)**）识别。XMPP 服务器发布 XMPP 服务，以提供对客户端的远程服务连接。

在这个配方中，`WebServiceTemplate`用于通过 XMPP 传输在客户端发送/接收简单的 XML 消息。使用了第一章中的*在 XMPP 传输上设置 Web 服务*配方，*构建 SOAP Web 服务*，来设置一个 Web 服务。使用了一个 JUnit 测试用例类来在服务器端设置 Web 服务，并使用`WebServiceTemplate`发送和接收消息。

## 准备工作

在这个配方中，项目的名称是`LiveRestaurant_R-2.6`，具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `spring-ws-support-2.0.1.RELEASE.jar`

+   `spring-test-3.0.5.RELEASE.jar`

+   `junit-4.7.jar`

+   `xmlunit-1.1.jar`

+   `smack-3.1.0.jar`

## 如何做...

1.  这个配方使用了在第一章中讨论的*在 XMPP 传输上设置 Web 服务*配方，*构建 SOAP Web 服务*，作为服务器端项目。

1.  创建一个测试类，调用`WebServiceTemplate`调用 Web 服务服务器。

1.  在`applicationContext`中配置`WebServiceTemplate`以通过 XMPP 协议发送消息。

1.  运行命令`mvn clean package`。您将看到以下输出：

```java
Received response ..
<tns:placeOrderResponse >
<tns:refNumber>order-John_Smith_1234</tns:refNumber>
</tns:placeOrderResponse>....
<tns:placeOrderRequest >
<tns:order>
<tns:refNumber>9999</tns:refNumber>
<tns:customer>
.....
</tns:customer>
<tns:dateSubmitted>2008-09-29T05:49:45</tns:dateSubmitted>
<tns:orderDate>2014-09-19T03:18:33</tns:orderDate>
<tns:items>
<tns:type>Snacks</tns:type>
<tns:name>Pitza</tns:name>
<tns:quantity>2</tns:quantity>
</tns:items>
</tns:order>
</tns:placeOrderRequest> 

```

## 工作原理...

该项目使用 JUnit 类在 XMPP 传输上设置了一个 Web 服务服务器。该服务器已经在配方*在电子邮件传输上设置 Web 服务*中解释过，在第一章中讨论了*构建 SOAP Web 服务*。

已配置客户端`WebServiceTemplate`的`applicationContext.xml`文件（`id="webServiceTemplate"`）用于发送和接收 XML 消息。可以从客户端程序中获取此 bean 的实例，以发送和接收 XML 消息。`messageFactory`是`SaajSoapMessageFactory`的一个实例，被引用在`WebServiceTemplate`内。`messageFactory`用于从 XML 消息创建 SOAP 数据包。`WebServiceTemplate`使用`XmppMessageSender`发送消息到服务器。默认 URI 是`WebServiceTemplate`默认使用的 XMPP 地址 URI，用于发送/接收所有请求/响应：

```java
<bean id="webServiceTemplate" class="org.springframework.ws.client.core.WebServiceTemplate">
<constructor-arg ref="messageFactory"/>
<property name="messageSender">
<bean class="org.springframework.ws.transport.xmpp.XmppMessageSender">
<property name="connection" ref="connection"/>
</bean>
</property>
<property name="defaultUri" value="xmpp:yourUserName@gmail.com"/>
</bean>

```

`XMPPTransportWebServiceIntegrationTest.java`是一个 JUnit 测试用例，从`applicationContext.xml`中获取并注入`WebServiceTemplate`（由`@ContextConfiguration("applicationContext.xml")`标记）。在`testWebServiceOnXMPPTransport()`方法中（由`@Test`标记），`WebServiceTemplate`发送一个 XML 消息（由简单的输入 XML 文件的`StringSource`对象创建），并接收服务器包装在`Result`对象中的响应。

```java
@Autowired
private GenericApplicationContext applicationContext;
@Test
public void testWebServiceOnXMPPTransport() throws Exception {
StringResult result = new StringResult();
StringSource sc=new StringSource(requestContent);
webServiceTemplate.sendSourceAndReceiveToResult(sc, result);
XMLAssert.assertXMLEqual("Invalid content received", requestContent, result.toString());
applicationContext.close();
}

```

## 另请参阅

在第一章中讨论的*在 XMPP 传输上设置 Web 服务*配方，*构建 SOAP Web 服务*。

使用 Spring JUnit 对 Web 服务进行单元测试

# 使用 XPath 表达式创建 Web 服务客户端

在 Java 编程中使用 XPath 是从 XML 消息中提取数据的标准方法之一。但是，它将 XML 节点/属性的 XPath 地址（最终可能非常长）与 Java 代码混合在一起。

Spring 提供了一个功能，可以从 Java 中提取这些地址，并将它们转移到 Spring 配置文件中。在这个配方中，使用了第一章中的*通过注释有效负载根设置端点*配方，*构建 SOAP Web 服务*，来设置一个 Web 服务服务器。

## 准备工作

在这个配方中，项目的名称是`LiveRestaurant_R-2.7`（用于服务器端 Web 服务），具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `log4j-1.2.9.jar`

以下是`LiveRestaurant_R-2.7-Client`（用于客户端 Web 服务）的 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `junit-4.7.jar`

+   `log4j-1.2.9.jar`

## 如何做...

此食谱使用了在服务器端项目中讨论的*通过注释负载根设置端点*食谱。

1.  在`applicationContext.xml`中配置 XPath 表达式。

1.  在`applicationContext`中配置`WebServiceTemplate`以通过 HTTP 协议发送消息，如食谱*在 HTTP 传输上创建 Web 服务客户端*中所述。

1.  创建一个测试类，使用`WebServiceTemplate`调用 Web 服务服务器，并在 Java 代码中使用 XPath 表达式提取所需的值。

1.  从文件夹`Liverestaurant_R-2.7`中运行命令`mvn clean package tomcat:run`。

1.  打开一个新的命令窗口到`Liverestaurant_R-2.7-Client`并运行以下命令：

```java
mvn clean package. 

```

+   以下是客户端代码的输出：

```java
--Request
<tns:placeOrderRequest >
<tns:order>
<tns:refNumber>9999</tns:refNumber>
<tns:customer>
<tns:addressPrimary>
<tns:doorNo>808</tns:doorNo>
<tns:building>W8</tns:building>
<tns:street>St two</tns:street>
<tns:city>NY</tns:city>
<tns:country>US</tns:country>
<tns:phoneMobile>0018884488</tns:phoneMobile>
<tns:phoneLandLine>0017773366</tns:phoneLandLine>
<tns:email>d@b.c</tns:email>
</tns:addressPrimary>
<tns:addressSecondary>
<tns:doorNo>409</tns:doorNo>
<tns:building>W2</tns:building>
<tns:street>St one</tns:street>
<tns:city>NY</tns:city>
<tns:country>US</tns:country>
<tns:phoneMobile>0018882244</tns:phoneMobile>
<tns:phoneLandLine>0019991122</tns:phoneLandLine>
<tns:email>a@b.c</tns:email>
</tns:addressSecondary>
<tns:name>
<tns:fName>John</tns:fName>
<tns:mName>Paul</tns:mName>
<tns:lName>Smith</tns:lName>
</tns:name>
</tns:customer>
<tns:dateSubmitted>2008-09-29T05:49:45</tns:dateSubmitted>
<tns:orderDate>2014-09-19T03:18:33</tns:orderDate>
<tns:items>
<tns:type>Snacks</tns:type>
<tns:name>Pitza</tns:name>
<tns:quantity>2</tns:quantity>
</tns:items>
</tns:order>
</tns:placeOrderRequest>
<!--Received response-->
<tns:placeOrderResponse >
<tns:refNumber>order-John_Smith_1234</tns:refNumber></tns:placeOrderResponse>
...Request
<tns:cancelOrderRequest >
<tns:refNumber>9999</tns:refNumber>
</tns:cancelOrderRequest></SOAP-ENV:Body></SOAP-ENV:Envelope>]
...Received response..
<tns:cancelOrderResponse >
<tns:cancelled>true</tns:cancelled></tns:cancelOrderResponse> 

```

## 工作原理...

设置客户端和服务器端，并使用`WebserviceTemplate`的方式与我们在食谱*在 HTTP 传输上创建 Web 服务客户端*中所做的一样。在客户端`applicationContext.xml`中配置了`xpathExpPlace`和`xpathExpCancel`，并创建了`XPathExpressionFactoryBean`的实例，该实例获取所需数据的 XPath 属性和 XML 消息的命名空间：

```java
<bean id="xpathExpCancel"
class="org.springframework.xml.xpath.XPathExpressionFactoryBean">
<property name="expression" value="/tns:cancelOrderResponse/tns:cancelled" />
<property name="namespaces">
<props>
<prop key="tns">http://www.packtpub.com/liverestaurant/OrderService/schema</prop>
</props>
</property>
</bean>
<bean id="xpathExpPlace"
class="org.springframework.xml.xpath.XPathExpressionFactoryBean">
<property name="expression" value="/tns:placeOrderResponse/tns:refNumber" />
<property name="namespaces">
<props>
<prop key="tns">http://www.packtpub.com/liverestaurant/OrderService/schema</prop>
</props>
</property>
</bean>

```

在`OrderServiceClientTest`类中，可以从`applicationContext`中提取`XPathExpressionFactoryBean`的实例。`String message = xpathExp.evaluateAsString(result.getNode())`使用 XPath 表达式返回所需的数据：

```java
@Test
public final void testPlaceOrderRequest() {
DOMResult result=invokeWS(isPlace);
String message = xpathExpPlace.evaluateAsString(result.getNode());
Assert.assertTrue(message.contains("Smith"));
}
@Test
public final void testCancelOrderRequest() {
DOMResult result= invokeWS(isCancel);
Boolean cancelled = xpathExpCancel.evaluateAsBoolean(result.getNode());
Assert.assertTrue(cancelled);
}

```

## 另请参阅

食谱*使用 XPath 表达式设置端点*在第一章 *构建 SOAP Web 服务*中讨论。

在本章中讨论的食谱*在 HTTP 传输上创建 Web 服务客户端*。

使用 Spring JUnit 对 Web 服务进行单元测试。

# 为 WS-Addressing 端点创建 Web 服务客户端

如食谱*设置一个传输中立的 WS-Addressing 端点*中所述，讨论在第一章 *构建 SOAP Web 服务*，WS-Addressing 是一种替代的路由方式。WS-Addressing 将路由数据与消息分开，并将其包含在 SOAP 头中，而不是在 SOAP 消息的主体中。以下是从客户端发送的 WS-Addressing 样式的 SOAP 消息示例：

```java
<SOAP-ENV:Header >
<wsa:To>server_uri</wsa:To>
<wsa:Action>action_uri</wsa:Action>
<wsa:From>client_address </wsa:From>
<wsa:ReplyTo>client_address</wsa:ReplyTo>
<wsa:FaultTo>admen_uri </wsa:FaultTo>
<wsa:MessageID>..</wsa:MessageID>
</SOAP-ENV:Header>
<SOAP-ENV:Body>
<tns:placeOrderRequest>....</tns:placeOrderReques>
</SOAP-ENV:Body></SOAP-ENV:Envelope>] 

```

在使用 WS-Addressing 时，与其他方法（包括在消息中包含路由数据）相比，客户端或服务器可以访问更多功能。例如，客户端可以将`ReplyTo`设置为自己，将`FaultTo`设置为管理员端点地址。然后服务器将成功消息发送到客户端，将故障消息发送到管理员地址。

Spring-WS 支持客户端和服务器端的 WS-Addressing。要为客户端创建 WS-Addressing 头，可以使用`org.springframework.ws.soap.addressing.client.ActionCallback`。此回调将`Action`头保留为参数。它还使用 WS-Addressing 版本和`To`头。

在此食谱中，使用了在第一章 *构建 SOAP Web 服务*中讨论的*设置一个传输中立的 WS-Addressing 端点*食谱来设置 WS-Addressing Web 服务。在这里使用客户端应用程序来调用服务器并返回响应对象。

## 准备工作

在此食谱中，项目名称为`LiveRestaurant_R-2.8`（用于服务器端 Web 服务），具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `log4j-1.2.9.jar`

以下是`LiveRestaurant_R-2.8-Client`（用于客户端 Web 服务）的 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `junit-4.7.jar`

+   `log4j-1.2.9.jar`

## 如何做...

这个配方使用了第一章中讨论的*为 Web 服务设置与传输无关的 WS-Addressing 端点*配方，*构建 SOAP Web 服务*，作为服务器端项目。创建 WS-Addressing 的客户端与*在 HTTP 传输上创建 Web 服务客户端*配方中描述的方式相同，不使用 WebServiceTemplate。为了在客户端上添加 WS-Addressing 头，`WebServiceTemplate`的`sendSourceAndReceiveToResult`方法获得一个`ActionCallBack`实例。

1.  从文件夹`LiveRestaurant_R-2.8`中运行以下命令：

```java
mvn clean package tomcat:run 

```

1.  打开一个新的命令窗口到`LiveRestaurant_R-2.8-Client`，并运行以下命令：

```java
mvn clean package 

```

+   以下是客户端输出：

```java
Received response [<SOAP-ENV:Envelope xmlns:SOAP-ENV="..../">
<SOAP-ENV:Header >
<wsa:To SOAP-ENV:mustUnderstand="1">....</wsa:To><wsa:Action>http://www.packtpub.com/OrderService/CanOrdReqResponse</wsa:Action>
<wsa:MessageID>....</wsa:MessageID>
<wsa:RelatesTo>...</wsa:RelatesTo>
</SOAP-ENV:Header>
<SOAP-ENV:Body>
<tns:cancelOrderResponse >
<tns:cancelled>true</tns:cancelled></tns:cancelOrderResponse>
</SOAP-ENV:Body></SOAP-ENV:Envelope>]
for request ...
<SOAP-ENV:Envelope xmlns:SOAP
-ENV=".."><SOAP-ENV:Header >
<wsa:To SOAP-ENV:mustUnderstand="1">http://www.packtpub.com/liverestaurant/OrderService/schema</wsa:To>
<wsa:To SOAP-ENV:mustUnderstand="1">http://www.packtpub.com/liverestaurant/OrderService/schema</wsa:To>
<wsa:Action>http://www.packtpub.com/OrderService/CanOrdReq</wsa:Action>
<wsa:MessageID>..</wsa:MessageID>
</SOAP-ENV:Header><SOAP-ENV:Body/>
</SOAP-ENV:Envelope>]
<?xml version="1.0" encoding="UTF-8"?>
<tns:cancelOrderResponse >
<tns:cancelled>true</tns:cancelled></tns:cancelOrderResponse> 

```

## 工作原理...

`Liverestaurant_R-2.8`项目是一个支持 WS-Addressing 端点的服务器端 Web 服务。

已配置的客户端`WebServiceTemplate`的`applicationContext.xml`文件（`id="webServiceTemplate"）用于发送和接收 XML 消息，如*在 HTTP 传输上创建 Web 服务客户端*配方中描述的，除了使用`WebServiceTemplate`的 Java 类的实现。

WS-Addressing 客户端将`ActionCallBack`的实例传递给`WebServiceTemplate`的`sendSourceAndReceiveToResult`方法。使用`ActionCallBack`，客户端添加一个包含`Action` URI 的自定义头，例如，[`www.packtpub.com/OrderService/OrdReq`](http://www.packtpub.com/OrderService/OrdReq)和`To` URI，例如，[`www.packtpub.com/liverestaurant/OrderService/schema`](http://www.packtpub.com/liverestaurant/OrderService/schema)。

```java
@Test
public final void testPlaceOrderRequest() throws URISyntaxException {
invokeWS(isPlace,"http://www.packtpub.com/OrderService/OrdReq");
}
@Test
public final void testCancelOrderRequest() throws URISyntaxException {
invokeWS(isCancel,"http://www.packtpub.com/OrderService/CanOrdReq");
}
private static Result invokeWS(InputStream is,String action) throws URISyntaxException {
StreamSource source = new StreamSource(is);
StringResult result = new StringResult();
wsTemplate.sendSourceAndReceiveToResult(source, new ActionCallback(new URI(action),new Addressing10(),new URI("http://www.packtpub.com/liverestaurant/OrderService/schema")),
result);
return result;
}

```

使用这个头，服务器端将能够在端点中找到方法（使用`@Action`注释）。

## 参见

在第一章中讨论的*为 Web 服务设置与传输无关的 WS-Addressing 端点*配方，*构建 SOAP Web 服务*。

在本章中讨论的*在 HTTP 传输上创建 Web 服务客户端*配方。

使用 Spring JUnit 对 Web 服务进行单元测试

# 使用 XSLT 转换 Web 服务消息

最终，Web 服务的客户端可能使用不同版本的 XML 消息，要求在服务器端使用相同的 Web 服务。

Spring Web 服务提供`PayloadTransformingInterceptor`。这个端点拦截器使用 XSLT 样式表，在需要多个版本的 Web 服务时非常有用。使用这个拦截器，您可以将消息的旧格式转换为新格式。

在这个配方中，使用第一章中的*为 Web 服务设置简单的端点映射*配方，*构建 SOAP Web 服务*，来设置一个 Web 服务，这里的客户端应用程序调用服务器并返回响应消息。

## 准备工作

在这个配方中，项目的名称是`LiveRestaurant_R-2.9`（用于服务器端 Web 服务），具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `log4j-1.2.9.jar`

以下是`LiveRestaurant_R-2.9-Client`（客户端 Web 服务）的 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `junit-4.7.jar`

+   `log4j-1.2.9.jar`

## 如何做...

这个配方使用了第一章中讨论的*为 Web 服务设置简单的端点映射*配方，*构建 SOAP Web 服务*，作为服务器端项目。客户端与*在 HTTP 传输上创建 Web 服务客户端*配方中讨论的相同，除了 XSLT 文件及其在服务器端应用程序上下文文件中的配置：

1.  创建 XSLT 文件（oldResponse.xslt，oldRequest.xslt）。

1.  修改`LiveRestaurant_R-2.9`中的`spring-ws-servlet.xml`文件以包含 XSLT 文件

1.  从文件夹`Liverestaurant_R-2.9`中运行以下命令：

```java
mvn clean package tomcat:run 

```

1.  打开一个新的命令窗口到`Liverestaurant_R-2.9-Client`，并运行以下命令：

```java
mvn clean package 

```

+   以下是客户端输出：

```java
Received response...
<ns:OrderResponse  message="Order Accepted!"/>...
for request ....
<OrderRequest  message="This is a sample Order Message"/> 

```

+   以下是服务器端输出：

```java
actual request ..
<ns:OrderRequest >
<ns:message>This is a sample Order Message</ns:message></ns:OrderRequest>
actual response = <ns:OrderResponse >
<ns:message>Order Accepted!</ns:message></ns:OrderResponse> 

```

## 它是如何工作的...

服务器端与第一章中描述的*设置简单的端点映射用于 Web 服务*的配方相同，*构建 SOAP Web 服务*。在客户端，`WebServiceTemplate`和`OrderServiceClientTest.java`与*在 HTTP 传输上创建 Web 服务客户端*的配方中描述的相同。

唯一的区别是服务器应用程序上下文文件。`spring-servlet.xml`中的`transformingInterceptor` bean 使用`oldRequests.xslt`和`oldResponse.xslt`分别将旧的请求 XML 消息转换为服务器的更新版本，反之亦然：

```java
. <bean class="org.springframework.ws.server.endpoint.mapping.SimpleMethodEndpointMapping">
<property name="endpoints">
<ref bean="OrderServiceEndpoint" />
</property>
<property name="methodPrefix" value="handle"></property>
<property name="interceptors">
<list>
<bean
class="org.springframework.ws.server.endpoint.interceptor.PayloadLoggingInterceptor">
<property name="logRequest" value="true" />
<property name="logResponse" value="true" />
</bean>
<bean id="transformingInterceptor"
class="org.springframework.ws.server.endpoint.interceptor.PayloadTransformingInterceptor">
<property name="requestXslt" value="/WEB-INF/oldRequests.xslt" />
<property name="responseXslt" value="/WEB-INF/oldResponse.xslt" />
</bean>
</list>
</property>
</bean>

```

## 另请参阅

在第一章中讨论的*设置简单的端点映射用于 Web 服务*的配方，*构建 SOAP Web 服务*。

使用 Spring JUnit 对 Web 服务进行单元测试。
