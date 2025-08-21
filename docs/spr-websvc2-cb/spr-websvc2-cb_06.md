# 第六章：编组和对象/XML 映射（OXM）

在本章中，我们将涵盖以下主题：

+   使用 JAXB2 进行编组

+   使用 XMLBeans 进行编组

+   使用 JiBX 进行编组

+   使用 XStream 进行编组

+   使用 MooseXML 进行编组

+   使用 XPath 创建自定义编组器进行条件 XML 解析

# 介绍

在对象/XML 映射（OXM）术语中，编组（序列化）将数据的对象表示转换为 XML 格式，而解组将 XML 转换为相应的对象。

Spring 的 OXM 通过使用 Spring 框架的丰富特性简化了 OXM 操作。例如，可以使用依赖注入功能将不同的 OXM 技术实例化为对象以使用它们，Spring 可以使用注解将类或类的字段映射到 XML。

Spring-WS 受益于 Spring 的 OXM，可以将有效负载消息转换为对象，反之亦然。例如，可以在应用程序上下文中使用以下配置将 JAXB 设置为 OXM 框架：

```java
<bean class="org.springframework.ws.server.endpoint.adapter.GenericMarshallingMethodEndpointAdapter">
<constructor-arg ref="marshaller" />
</bean>
<bean id="marshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller">
<property name="contextPath" value="com.packtpub.liverestaurant.domain" />
</bean>

```

此外，可以通过更改配置文件中的`marshaller` bean 来更改编组框架，同时保持 Web 服务的实现不变。

有许多可用的编组框架的实现。JAXB（Java Architecture for XML Binding）、JiBX、XMLBeans、Castor 等都是例子。对于一些 OXM 框架，提供了工具来将模式转换为 POJO 类，并在这些类中生成映射数据，或者在单独的外部配置文件中生成映射数据。

本章提供了示例来说明不同框架用于对象/XML 映射的用法。

为了简化，本章中大多数示例使用了“使用 Spring-JUnit 支持进行集成测试”一章中讨论的项目，该项目在“测试和监视 Web 服务”一章中讨论，用于设置服务器并通过客户端发送和接收消息。然而，在“使用 XStream 进行编组”一章中的示例中，使用了“为 WS-Addressing 端点创建 Web 服务客户端”一章中讨论的项目，该项目在“构建 SOAP Web 服务的客户端”一章中讨论，用于服务器和客户端。

# 使用 JAXB2 进行编组

Java Architecture for XML Binding（http://jaxb.java.net/tutorial/）是一个 API，允许开发人员将 Java 对象绑定到 XML 表示。JAXB 实现是 Metro 项目（http://metro.java.net/）的一部分，它是一个高性能、可扩展、易于使用的 Web 服务堆栈。JAXB 的主要功能是将 Java 对象编组为 XML 等效项，并根据需要将其解组为 Java 对象（也可以称为对象/XML 绑定或编组）。当规范复杂且变化时，JAXB 特别有用。

JAXB 提供了许多扩展和工具，使得对象/XML 绑定变得简单。其注解支持允许开发人员在现有类中标记 O/X 绑定，以便在运行时生成 XML。其 Maven 工具插件（maven-jaxb2-plugin）可以从给定的 XML Schema 文件生成 Java 类。

这个示例说明了如何设置编组端点并使用 JAXB2 作为编组库构建客户端程序。

## 准备工作

这个示例包含一个服务器（LiveRestaurant_R-6.1）和一个客户端（LiveRestaurant_R-6.1-Client）项目。

`LiveRestaurant_R-6.1`具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `log4j-1.2.9.jar`

`LiveRestaurant_R-6.1-Client`具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `spring-test-3.0.5.RELEASE.jar`

+   `junit-4.7.jar`

这个示例使用`maven-jaxb2-plugin`从模式生成类。

## 如何做...

1.  在服务器/客户端配置文件中注册 JAXB 编组器。

1.  在服务器/客户端 POM 文件中配置`maven-jaxb2-plugin`。

1.  设置服务器并运行客户端（它还从模式生成类）：

+   客户端项目根目录：`mvn clean package`

+   服务器项目根目录：`mvn clean package tomcat:run`

以下是客户端输出：

```java
- Received response ....
<ns2:cancelOrderResponse...>
<ns2:cancelled>true</ns2:cancelled>
</ns2:cancelOrderResponse>
...
for request ...
<ns2:cancelOrderRequest ...>
<ns2:refNumber>Ref-2010..</ns2:refNumber>
</ns2:cancelOrderRequest>
.....
....
- Received response ....
<ns2:placeOrderResponse ...>
<ns2:refNumber>Ref-2011-1..</ns2:refNumber>
</ns2:placeOrderResponse>
...
for request ...
<ns2:placeOrderRequest ...>
<ns2:order>.....
</ns2:order></ns2:placeOrderRequest>
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 2.293 sec 

```

## 它是如何工作的...

在这个编组业务中的主要参与者是`GenericMarshallingMethodEndpointAdapter`，它利用编组器执行对象/XML 编组过程。这里使用的编组器是`org.springframework.oxm.jaxb.Jaxb2Marshaller`，它执行 O/X 编组，利用 JAXB2 框架。如果您检查 Maven 插件工具生成的 Java 类，您可以看到 JAXB 注释，如`@XmlType，@XmlRootElement，@XmlElement`等。这些注释是 JAXB 引擎的指令，用于确定在运行时生成的 XML 的结构。

POM 文件中的以下部分从文件夹`src\main\webapp\WEB-INF`（由`schemaDirectory`设置）中的模式`（OrderService.xsd）`生成 JAXB 类。

`GeneratePackage`设置包括生成类的包，`generateDirectory`设置了托管生成包的文件夹：

```java
<plugins>
<plugin>
<artifactId>maven-compiler-plugin</artifactId>
<configuration>
<source>1.6</source>
<target>1.6</target>
</configuration>
</plugin>
<plugin>
<groupId>org.codehaus.mojo</groupId>
<artifactId>tomcat-maven-plugin</artifactId>
<version>1.1</version>
</plugin>
<plugin>
<groupId>org.jvnet.jaxb2.maven2</groupId>
<artifactId>maven-jaxb2-plugin</artifactId>
<configuration>
<schemaDirectory>src\main\webapp\WEB-INF</schemaDirectory> 
<schemaIncludes>
<include>orderService.xsd</include> 
</schemaIncludes>
<generatePackage>com.packtpub.liverestaurant.domain</generatePackage> 
</configuration>
<executions>
<execution>
<phase>generate-resources</phase>
<goals>
<goal>generate</goal>
</goals>
</execution>
</executions>
</plugin>
</plugins>

```

`OrderServiceEndPoint`被注释为`@Endpoint`，将 Web 服务请求与`placeOrderRequest`的有效载荷根映射到`getOrder`方法，识别注释`@PayloadRoot`。编组器将传入的 XML 编组为`PlaceOrderRequest`的实例，方法`getOrder`返回`PlaceOrderResponse`。方法`cancelOrder`也是同样的情况：

```java
@PayloadRoot(localPart = "placeOrderRequest", namespace = SERVICE_NS)
public PlaceOrderResponse getOrder(
PlaceOrderRequest placeOrderRequest) {
PlaceOrderResponse response = JAXB_OBJECT_FACTORY
.createPlaceOrderResponse();
response.setRefNumber(orderService.placeOrder(placeOrderRequest
.getOrder()));
return response;
}
@PayloadRoot(localPart = "cancelOrderRequest", namespace = SERVICE_NS)
public CancelOrderResponse cancelOrder(
CancelOrderRequest cancelOrderRequest) {
CancelOrderResponse response = JAXB_OBJECT_FACTORY
.createCancelOrderResponse();
response.setCancelled(orderService.cancelOrder(cancelOrderRequest
.getRefNumber()));
return response;
}

```

在服务器的`spring-ws-servlet.xml`中的以下部分将编组器设置为端点（OrderServiceEndpoint）为`Jaxb2Marshaller`。在`marshaller` bean 中的`contextPath`设置注册了包`com.packtpub.liverestaurant.domain`中的所有 bean，以便由`Jaxb2Marshaller`进行编组/解组：

```java
<bean class="org.springframework.ws.server.endpoint.adapter.GenericMarshallingMethodEndpointAdapter">
<constructor-arg ref="marshaller" />
</bean>
<bean id="marshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller">
<property name="contextPath" value="com.packtpub.liverestaurant.domain" />
</bean>

```

在客户端中也会发生同样的事情。唯一的区别是编组器设置为`WebServiceTemplate`：

```java
<bean id="orderServiceTemplate" class="org.springframework.ws.client.core.WebServiceTemplate">
<constructor-arg ref="messageFactory" />
<property name="marshaller" ref="orderServiceMarshaller"></property>
<property name="unmarshaller" ref="orderServiceMarshaller"></property>
.........</bean>
<bean id="orderServiceMarshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller">
<property name="contextPath" value="com.packtpub.liverestaurant.domain" />
</bean>

```

`MessageDispatcherServlet`借助`Jaxb2Marshaller`检测 O/X 映射注释以及反射，并将最终编组过程委托给 JAXB 框架。

# 使用 XMLBeans 进行编组

XMLBeans（[`xmlbeans.apache.org/`](http://xmlbeans.apache.org/)）是一种通过将其绑定到 Java 类型来访问 XML 的技术。该库来自 Apache 基金会，并且是 Apache XML 项目的一部分。XMLBeans 以其友好的 Java 特性而闻名，允许开发人员充分利用 XML 和 XML Schema 的丰富性和功能，并将这些功能尽可能自然地映射到等效的 Java 语言和类型构造中。

使 XMLBeans 与其他 XML-Java 绑定选项不同的两个主要特点是：

+   **完整的 XML Schema 支持：**XMLBeans 完全支持（内置）XML Schema，并且相应的 Java 类为 XML Schema 的所有主要功能提供了构造。

+   **完整的 XML 信息集忠实度：**在解组 XML 数据时，开发人员可以获得完整的 XML 信息集。XMLBeans 提供了许多扩展和工具，使对象/XML 绑定变得简单。

## 准备工作

此配方包含服务器（`LiveRestaurant_R-6.2`）和客户端（`LiveRestaurant_R-6.2-Client`）项目。

`LiveRestaurant_R-6.2`具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `xmlbeans-2.4.0.jar`

`LiveRestaurant_R-6.2-Client`具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `xmlbeans-2.4.0.jar`

+   `spring-test-3.0.5.RELEASE.jar`

+   `junit-4.7.jar`

此配方使用`xmlbeans-maven-plugin`从模式生成类并绑定文件。

## 如何做...

1.  在服务器/客户端配置文件中注册 XMLBean 编组器。

1.  在服务器/客户端 POM 文件中配置`xmlbeans-maven-plugin`。

1.  设置服务器并运行客户端（它还从模式生成类）：

1.  运行以下命令：

+   服务器项目根目录：`mvn clean package tomcat:run`

+   客户端项目根：`mvn clean package`

以下是客户端输出：

```java
[INFO]
[INFO] --......
[INFO]
[INFO] --- xmlbeans-maven-plugin:2.3.2:xmlbeans ....
[INFO]
[INFO] .....
Received response ...
<sch:cancelOrderResponse ...>
<sch:cancelled>true</sch:cancelled>
</sch:cancelOr
derResponse>...
for request.....
......
- Received response ...
<sch:placeOrderResponse ...>
<sch:refNumber>Ref-2011-10-..</sch:refNumber>
</sch:placeOrderResponse>
...
for request ....
...
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 2.845 sec 

```

## 工作原理...

这个示例的工作方式与第一个示例*使用 JAXB2 进行编组*完全相同，只是它使用了不同的编组器`XMLBeansMarshaller`。这里使用的 scomp（模式编译器）工具从 XML 模式（OrderService.xsd）生成 Java XMLBeans 类。除了域类，它还生成了表示文档根元素的类，例如`CancelOrderRequestDocument`。所有生成的类都包含`Factory`方法来实例化它们。

很容易注意到，代码中的两个主要区别在于`OrderServiceEndPoint`和`spring-ws-servlet.xml`。与上一个示例不同，`getOrder`方法返回`OrderResponseDocument`的实例，并接受`OrderRequestDocument`作为输入参数。`cancelOrderDoc`方法也是如此。

```java
@PayloadRoot(localPart = "placeOrderRequest", namespace = SERVICE_NS)
public PlaceOrderResponseDocument getOrder(PlaceOrderRequestDocument orderRequestDoc) {
PlaceOrderResponseDocument orderResponseDocument =PlaceOrderResponseDocument.Factory.newInstance();
orderResponseDocument.addNewPlaceOrderResponse();
orderResponseDocument.getPlaceOrderResponse().setRefNumber(orderService.placeOrder(orderRequestDoc));
return orderResponseDocument;
}
@PayloadRoot(localPart = "cancelOrderRequest", namespace = SERVICE_NS)
public CancelOrderResponseDocument placeCancelOrderDoc(
CancelOrderRequestDocument cancelOrderRequestDoc) {
CancelOrderResponseDocument cancelOrderResponseDocument= CancelOrderResponseDocument.Factory.newInstance();
cancelOrderResponseDocument.addNewCancelOrderResponse();
cancelOrderResponseDocument.getCancelOrderResponse().setCancelled(orderService.cancelOrder(cancelOrderRequestDoc.getCancelOrderRequest().getRefNumber()));
return cancelOrderResponseDocument;
}

```

在`spring-ws-servlet.xml`中使用的编组器是`XMLBeansMarshaller`，它使用 XMLBeans 库在 XML 和 Java 之间进行编组和解组。

```java
<bean class="org.springframework.ws.server.endpoint.adapter.GenericMarshallingMethodEndpointAdapter">
<constructor-arg ref="marshaller" />
</bean>
<bean id="marshaller" class="org.springframework.oxm.xmlbeans.XmlBeansMarshaller"/>

```

`@Endpoint`类和`XMLBeansMarshaller`之间的契约是，`@PayloadRoot`方法应该接受并返回`org.apache.xmlbeans.XmlObject`的实例。然后它动态地找到相应的类，并使用它们的`Factory`方法，在运行时创建实例并绑定到 XML。

与上一个示例一样，POM 文件中的插件从文件夹`src\main\webapp\WEB-INF`（由`schemaDirectory`设置）中的模式（OrderService.xsd）生成`XMLBean`类：

```java
<plugin>
<groupId>org.codehaus.mojo</groupId>
<artifactId>xmlbeans-maven-plugin</artifactId>
<version>2.3.2</version>
<executions>
<execution>
<goals>
<goal>xmlbeans</goal>
</goals>
</execution>
</executions>
<inherited>true</inherited>
<configuration>
<schemaDirectory>src/main/webapp/WEB-INF/</schemaDirectory> 
</configuration>
</plugin>

```

`MessageDispatcherServlet`借助`XMLBeansMarshaller`检测 O/X 映射注解和编组配置，并将最终编组过程委托给 XMLBeans 框架。

## 还有更多...

XMLBeans 配备了一套内置的强大工具，可以为 XML 和 Java 之间的编组添加更多功能。本示例仅使用了其中一个工具，即`scomp`，即模式编译器，它可以从 XML 模式（.xsd）文件生成 Java 类/压缩的 JAR 文件。其他一些有用的工具包括：

+   `inst2xsd`（Instance to Schema Tool）：从 XML 实例文件生成 XML 模式。

+   `scopy`（Schema Copier）：将指定 URL 的 XML 模式复制到指定文件

+   `validate`（Instance Validator）：根据模式验证实例

+   `xpretty`（XML Pretty Printer）：将指定的 XML 格式化打印到控制台

+   `xsd2inst`（Schema to Instance Tool）：使用指定的模式从指定的全局元素打印 XML 实例

+   `xsdtree`（Schema Type Hierarchy Printer）：打印模式中定义的类型的继承层次结构

+   `xmlbean Ant task:` 将一组 XSD 和/或 WSDL 文件编译成 XMLBeans 类型

`xmlbean Ant task`是自动化生成 Java 类的一种不错的方式，可以与构建脚本集成。

# 使用 JiBX 进行编组

JiBX（[`jibx.sourceforge.net/`](http://jibx.sourceforge.net/)）是另一个用于将 XML 数据绑定到 Java 对象的工具和库。JiBX 以速度性能和灵活性而闻名。然而，它也以绑定的复杂性而闻名，特别是对于复杂的数据模型。

从 1.2 版本开始，JiBX 已解决了这些瓶颈，现在它具有易于使用的编组工具和框架。使用 JiBX 工具，用户可以从现有 Java 代码生成模式，或者从现有模式生成 Java 代码和绑定文件。JiBX 库在运行时将 Java 类绑定到 XML 数据，反之亦然。

在本示例中，使用 JiBX 工具（jibx-maven-plugin）生成 POJO 类，并从现有模式绑定定义文件，然后将基于 JiBX 库构建 Web 服务客户端和服务器。

## 准备工作

这个示例包含一个服务器（LiveRestaurant_R-6.3）和一个客户端（LiveRestaurant_R-6.3-Client）项目。

`LiveRestaurant_R-6.3`具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `spring-expression-3.0.5.RELEASE.jar`

+   `jibx-run-1.2.3.jar`

+   `jibx-extras-1.2.3.jar`

+   `jibx-ws-0.9.1.jar`

`LiveRestaurant_R-6.3-Client`具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `spring-expression-3.0.5.RELEASE.jar`

+   `jibx-run-1.2.3.jar`

+   `jibx-extras-1.2.3.jar`

+   `jibx-ws-0.9.1.jar`

+   `spring-test-3.0.5.RELEASE.jar`

+   `junit-4.7.jar`

## 如何做...

1.  在服务器/客户端配置文件中注册 JiBX 编组器。

1.  在服务器/客户端 POM 文件中配置`xmlbeans-maven-plugin`。

1.  设置服务器并运行客户端（它还从模式生成类）：

+   服务器项目根目录：`mvn clean package`（它还从模式生成类）。将 WAR 文件复制到 Tomcat 的`webapp`文件夹中并运行 Tomcat（apache-tomcat-6.0.18）

+   客户端项目根目录：`mvn clean package`（它还从模式生成类）

以下是客户端的输出：

```java
.......
.........
[INFO] --- jibx-maven-plugin:1.2.3:bind (compile-binding) @ LiveRestaurant_Client ---
[INFO] Running JiBX binding compiler (single-module mode) on 1 binding file(s)
[INFO]
[INFO] ....
Received response ...
<tns:cancelOrderResponse ...>
<tns:cancelled>true</tns:cancelled></tns:cancelOrderResponse>
...
for request ...
<tns:cancelOrderRequest ...><tns:refNumber>12345</tns:refNumber>
</tns:cancelOrderRequest>
.....
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0 

```

## 它是如何工作的...

如前面的配方中所述，服务器/客户端的应用程序上下文使用自定义编组器（`org.springframework.oxm.jibx.JibxMarshaller`）执行对象/XML 编组过程。这个 Spring 编组器使用 JiBX 库进行绑定和编组过程。以下 POM 插件设置（目标：`schema-codegen`）从模式（`OrderService.xsd`）生成 POJO 类到一个包（`com.packtpub.liverestaurant.domain`），并且还生成一个绑定文件（目标：`bind`）：

```java
<plugin>
<groupId>org.jibx</groupId>
<artifactId>jibx-maven-plugin</artifactId>
<version>1.2.3</version>
<executions>
<execution>
<id>generate-java-code-from-schema</id>
<goals>
<goal>schema-codegen</goal> 
</goals>
</execution>
<execution>
<id>compile-binding</id>
<goals>
<goal>bind</goal> 
</goals>
</execution>
</executions>
<configuration>
<schemaLocation>src/main/webapp/WEB-INF</schemaLocation> 
<includeSchemas>
<includeSchema>orderService.xsd</includeSchema>
</includeSchemas>
<options>
<package>com.packtpub.liverestaurant.domain</package> 
</options>
</configuration>
</plugin>

```

如前面的配方中所述，服务器和客户端 Spring 上下文文件中的此设置使客户端和服务器使用自定义编组器（`JibxMarshaller`）对 POJO 类进行编组/解组为 XML 数据：

```java
<bean id="marshaller"
class="org.springframework.oxm.jibx.JibxMarshaller">
<property name="targetClass" value="com.packtpub.liverestaurant.domain.CancelOrderRequest" />
</bean>

```

`JibxMarshaller`使用`binding.xml`文件来进行编组任务。正如在映射文件中所示，JiBX 支持简单数据绑定（`<value style="element" name="fName"...`）以及被称为结构的复杂数据绑定（`<structure map-as="tns:Address"...`）。这个特性使 JiBX 成为其他框架中最灵活的绑定框架。

```java
......
<mapping abstract="true" type-name="tns:Customer" class="com.packtpub.liverestaurant.domain.Customer">
<structure map-as="tns:Address" get-method="getAddressPrimary" set-method="setAddressPrimary" name="addressPrimary"/>
<structure map-as="tns:Address" get-method="getAddressSecondary" set-method="setAddressSecondary" name="addressSecondary"/>
<structure map-as="tns:Name" get-method="getName" set-method="setName" name="name"/>
</mapping>
<mapping abstract="true" type-name="tns:Name" class="com.packtpub.liverestaurant.domain.Name">
<value style="element" name="fName" get-method="getFName" set-method="setFName"/>
<value style="element" name="mName" get-method="getMName" set-method="setMName"/>
<value style="element" name="lName" get-method="getLName" set-method="setLName"/>
</mapping>
.....

```

`OrderServiceEndPoint`被注释为`@Endpoint`，几乎与之前的配方（使用 JAXB2 进行编组）相同；只是实现略有不同。

```java
@PayloadRoot(localPart = "cancelOrderRequest", namespace = SERVICE_NS)
public
CancelOrderResponse handleCancelOrderRequest(CancelOrderRequest cancelOrderRequest) throws Exception {
CancelOrderResponse cancelOrderResponse=new CancelOrderResponse();
cancelOrderResponse.setCancelled(orderService.cancelOrder(cancelOrderRequest.getRefNumber()));
return cancelOrderResponse;
}
@PayloadRoot(localPart = "placeOrderRequest", namespace = SERVICE_NS)
public
PlaceOrderResponse handleCancelOrderRequest(PlaceOrderRequest placeOrderRequest) throws Exception {
PlaceOrderResponse orderResponse=new PlaceOrderResponse();
orderResponse.setRefNumber(orderService.placeOrder(placeOrderRequest.getOrder()));
return orderResponse;
}
......

```

## 还有更多...

JiBX 通过让用户创建自己的自定义编组器来提供更大的灵活性。这意味着可以使用自定义绑定文件和自定义编组器类来对任何类型的数据结构进行编组到 XML 文档中。

# 使用 XStream 进行编组

**XStream** ([`xstream.codehaus.org/`](http://xstream.codehaus.org/))是一个简单的库，用于将对象编组/解组为 XML 数据。以下主要特点使这个库与其他库不同：

+   不需要映射文件

+   不需要更改 POJO（不需要 setter/getter 和默认构造函数）

+   备用输出格式（JSON 支持和变形）

+   XStream 没有工具可以从现有的 Java 代码生成模式，也不能从现有模式生成 Java 代码

+   XStream 不支持命名空间

在这个配方中，创建了一个使用 XStream 库作为编组器的 Web 服务客户端和服务器。由于 XStream 在 XML 数据（有效负载）中不使用任何命名空间，因此设置了一种 Web 服务的网址样式。

## 准备工作

此配方包含一个服务器（`LiveRestaurant_R-6.4`）和一个客户端（`LiveRestaurant_R-6.4-Client`）项目。

`LiveRestaurant_R-6.4`具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `spring-expression-3.0.5.RELEASE.jar`

+   `jxstream-1.3.1.jar`

`LiveRestaurant_R-6.4-Client`具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `jxstream-1.3.1.jar`

+   `spring-test-3.0.5.RELEASE.jar`

+   `junit-4.7.jar`

## 如何做...

1.  在服务器/客户端配置文件中注册 XStream 编组器。

1.  用`Xstream`注释领域类。

1.  设置服务器并运行客户端：

+   服务器项目根目录：`mvn clean package tomcat:run`

+   客户端项目根目录：`mvn clean package`

以下是客户端输出：

```java
Received response
..
...
<wsa:Action>http://www.packtpub.com/OrderService/CancelOrdReqResponse</wsa:Action>
<wsa:MessageID>urn:uuid:a4b681ff-00f5-429e-9ab9-f9054e796a89</wsa:MessageID>
....
<cancelOrderResponse><cancelled>true</cancelled>
</cancelOrderResponse></SOAP-ENV:Body>
....
...
<wsa:Action>http://www.packtpub.com/OrderService/CancelOrdReq</wsa:Action>
...<cancelOrderRequest><refNumber>12345</refNumber></cancelOrderRequest> 

```

## 它是如何工作的...

如前一配方所述，服务器/客户端的应用程序上下文使用自定义的 marshaller（`org.springframework.oxm.xstream.XStreamMarshaller`）来执行对象/XML 编组过程。这个 spring marshaller 使用 XStream 库进行编组过程。在端点方法中的输入和输出参数的 bean（`OrderServiceEndPoint.java`）必须在`XstreamMarshaller`中注册。`autodetectAnnotations`设置为检测 POJO 类中的注释：

```java
<bean id="marshaller" class="org.springframework.oxm.xstream.XStreamMarshaller">
<property name="autodetectAnnotations" value="true"/>
<property name="aliases">
<map>
<entry key="placeOrderResponse" value="com.packtpub. liverestaurant.domain.PlaceOrderResponse" />
<entry key="placeOrderRequest" value="com.packtpub. liverestaurant.domain.PlaceOrderRequest" />
<entry key="cancelOrderRequest" value="com.packtpub. liverestaurant.domain.CancelOrderRequest" />
<entry key="cancelOrderResponse" value="com.packtpub. liverestaurant.domain.CancelOrderResponse" />
</map>
</property></bean>

```

`XStreamMarshaller`使用 POJO 类中的注释（而不是绑定文件）来执行编组任务。`@XstreamAlias`告诉 marshaller 这个类将被序列化/反序列化为'name'。还有其他注释是可选的，但它告诉 marshaller 如何序列化/反序列化类的字段（`@XStreamAsAttribute`，`@XStreamImplicit`等）。

```java
import com.thoughtworks.xstream.annotations.XStreamAlias;
@XStreamAlias("name")
public class Name
{
private String FName;
private String MName;
private String LName;

```

被注释为`@Endpoint`的`OrderServiceEndPoint`与 JiBX 配方相同，即端点方法的输入和返回参数是 POJO（`PlaceOrderResponse`，`PlaceOrderRequest`等），它们被映射到模式。唯一的区别是端点使用 Web 寻址进行方法映射：

```java
@Action("http://www.packtpub.com/OrderService/CancelOrdReq")
public
CancelOrderResponse handleCancelOrderRequest(CancelOrderRequest cancelOrderRequest) throws Exception {
CancelOrderResponse cancelOrderResponse=new CancelOrderResponse();
cancelOrderResponse.setCancelled(orderService.cancelOrder(cancelOrderRequest.getRefNumber()));
return cancelOrderResponse;
}
@Action("http://www.packtpub.com/OrderService/OrdReq")
public
PlaceOrderResponse handlePancelOrderRequest(PlaceOrderRequest placeOrderRequest) throws Exception {
PlaceOrderResponse orderResponse=new PlaceOrderResponse();
orderResponse.setRefNumber(orderService.placeOrder(placeOrderRequest.getOrder()));
return orderResponse;
}

```

# 使用 MooseXML 进行编组

**Moose**（[`quigley.com/moose/`](http://quigley.com/moose/)）是一个轻量级的框架，用于将对象编组/解组为 XML 数据。Moose 的模式生成器使得这个框架与其他框架不同。Moose 能够直接从带注释的 POJO 类生成模式。这是开发面向契约的 Web 服务开发所需的。

在这个配方中，Moose 用于在 Web 服务客户端和服务器通信中将对象编组/解组为 XML 数据。

## 准备工作

这个配方包含一个服务器（`LiveRestaurant_R-6.5`）和一个客户端（`LiveRestaurant_R-6.5-Client`）项目。

`LiveRestaurant_R-6.5`具有以下 Maven 依赖项：

+   `log4j-1.2.9.jar`

+   `moose-0.4.6.jar`

`LiveRestaurant_R-6.5-Client`具有以下 Maven 依赖项：

+   `log4j-1.2.9.jar`

+   `moose-0.4.6.jar`

+   `spring-test-3.0.5.RELEASE.jar`

+   `junit-4.7.jar`

## 如何做...

1.  在服务器/客户端配置文件中注册 Moose marshaller。

1.  使用`@XML`注释对领域类进行注释。

1.  设置服务器并运行客户端：

+   服务器项目根目录：`mvn clean package tomcat:run`

+   客户端项目根目录：`mvn clean package`

以下是客户端输出：

```java
Received response ...
<ns:cancelOrderResponse...>
<ns:cancelled>true</ns:cancelled>
</ns:cancelOrderResponse>
...
for request ...
<ns:cancelOrderRequest...>
<ns:refNumber>12345</ns:refNumber>
</ns:cancelOrderRequest>
....... 

```

## 它是如何工作的...

如前一配方所述，服务器/客户端的应用程序上下文使用自定义的 marshaller（`com.quigley.moose.spring.MooseMarshaller`）来执行对象/XML 编组过程。一个映射提供程序被注入到这个自定义的 marshaller 中。当对象被编组成 XML 时，映射提供程序用于设置命名空间和`xmlPrefix`，当 XML 数据被转换为对象时也是如此。映射提供程序从`com.quigley.moose.mapping.provider.annotation.StaticClassesProvider`中获取已注册的 POJO 类列表：

```java
<bean class="org.springframework.ws.server.endpoint.adapter.GenericMarshallingMethodEndpointAdapter">
<constructor-arg ref="mooseMarshaller"/>
</bean>
<bean class="org.springframework.ws.server.endpoint.mapping.PayloadRootAnnotationMethodEndpointMapping"/>
<bean id="mooseMarshaller" class="com.quigley.moose.spring.MooseMarshaller">
<property name="mappingProvider"><ref bean="mooseMappingProvider"/></property>
</bean>
<bean id="mooseMappingProvider"
class="com.quigley.moose.mapping.provider.annotation.AnnotationMappingProvider">
<property name="xmlNamespace">
<value>http://www.liverestaurant.com/OrderService/schema</value></property>
<property name="xmlPrefix"><value>ns</value></property>
<property name="annotatedClassesProvider"><ref bean="mooseClassesProvider"/></property>
</bean>
<bean id="mooseClassesProvider"
class="com.quigley.moose.mapping.provider.annotation. StaticClassesProvider">
<property name="classes">
<list>
<value>com.packtpub.liverestaurant.domain. CancelOrderRequest</value>
<value>com.packtpub.liverestaurant.domain. CancelOrderResponse</value>
<value>com.packtpub.liverestaurant.domain.Order </value>
<value>com.packtpub.liverestaurant.domain.Address </value>
<value>com.packtpub.liverestaurant.domain.Customer </value>
<value>com.packtpub.liverestaurant.domain.FoodItem </value>
<value>com.packtpub.liverestaurant.domain.Name </value>
<value>com.packtpub.liverestaurant.domain. PlaceOrderResponse</value>
<value>com.packtpub.liverestaurant.domain. PlaceOrderRequest</value>
</list>
</property>
</bean>

```

`MooseMarshaller`，就像`XStreamMarshaller`一样，使用 POJO 类中的注释来执行编组任务。`@XML`告诉 marshaller 这个类将被序列化/反序列化为'name'。`@XMLField`是应该放在每个类字段上的标记。

```java
@XML(name="cancelOrderRequest")
public class CancelOrderRequest
{
@XMLField(name="refNumber")
private String refNumber;
/** * Get the 'refNumber' element value.
*
* @return value
*/
public String getRefNumber() {
return refNumber;
}
/**
* Set the 'refNumber' element value.
*
* @param refNumber
*/
public void setRefNumber(String refNumber) {
this.refNumber = refNumber;
}
}

```

被注释为`@Endpoint`的`OrderServiceEndPoint`与 JiBX 配方相同，传递和返回参数是映射到模式的 POJO（`PlaceOrderResponse`，`PlaceOrderRequest`等）。

```java
@PayloadRoot(localPart = "cancelOrderRequest", namespace = SERVICE_NS)
public
CancelOrderResponse handleCancelOrderRequest(CancelOrderRequest cancelOrderRequest) throws Exception {
CancelOrderResponse cancelOrderResponse=new CancelOrderResponse();
cancelOrderResponse.setCancelled(orderService.cancelOrder(cancelOrderRequest.getRefNumber()));
return cancelOrderResponse;
}
@PayloadRoot(localPart = "placeOrderRequest", namespace = SERVICE_NS)
public
PlaceOrderResponse handleCancelOrderRequest(PlaceOrderRequest placeOrderRequest) throws Exception {
PlaceOrderResponse orderResponse=new PlaceOrderResponse();
orderResponse.setRefNumber(orderService.placeOrder(placeOrderRequest.getOrder()));
return orderResponse;
}

```

# 使用 XPath 创建自定义的 marshaller 进行条件 XML 解析。

始终使用现有的编组器框架（JAXB、JiBX 等）是处理编组任务的最简单方法。但是，最终您可能需要编写自定义的编组器。例如，您可能会收到一个 XML 输入数据，它的格式与通常由已识别的编组器使用的格式不同。

Spring 允许您定义自定义编组器并将其注入到端点编组器中，就像现有的编组器框架一样。在这个示例中，客户端以以下格式向服务器发送/接收数据：

```java
<ns:placeOrderRequest >
<ns:order refNumber="12345" customerfName="fName" customerlName="lName" customerTel="12345" dateSubmitted="2008-09-29 05:49:45" orderDate="2008-09-29 05:40:45">
<ns:item type="SNACKS" name="Snacks" quantity="1.0"/>
<ns:item type="DESSERTS" name="Desserts" quantity="1.0"/>
</ns:order>
</ns:placeOrderRequest>
<ns:placeOrderResponse  refNumber="1234"/>

```

然而，可以映射到/从服务器的 POJO 的 XML 输入如下：

```java
<ns:placeOrderRequest >
<ns:order>
<ns:refNumber>12345</ns:refNumber>
<ns:customerfName>fName</ns:customerfName>
<ns:customerlName>lName</ns:customerlName>
<ns:customerTel>12345</ns:customerTel>
<ns:dateSubmitted>2008-09-29 05:49:45</ns:dateSubmitted>
<ns:orderDate>2008-09-29 05:40:45</ns:orderDate>
<ns:items>
<FoodItem>
<ns:type>SNACKS</ns:type>
<ns:name>Snack</ns:name>
<ns:quantity>1.0</ns:quantity>
</FoodItem>
<FoodItem>
<ns:type>COFEE</ns:type>
<ns:name>Cofee</ns:name>
<ns:quantity>1.0</ns:quantity>
</FoodItem>
</ns:items>
</ns:order>
</ns:placeOrderRequest>
<ns:placeOrderResponse  />
<ns:refNumber>1234</ns:refNumber>
</ns:placeOrderResponse>

```

在这个示例中，使用自定义的编组器将传入的 XML 数据映射到服务器的 POJO，并将解组服务器响应到客户端格式。

## 准备工作

这个示例包含一个服务器（`LiveRestaurant_R-6.6`）和一个客户端（`LiveRestaurant_R-6.6-Client`）项目。

`LiveRestaurant_R-6.6` 具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `dom4j-1.6.1.jar`

`LiveRestaurant_R-6.6-Client` 具有以下 Maven 依赖项：

+   `spring-ws-core-2.0.1.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `spring-test-3.0.5.RELEASE.jar`

+   `junit-4.7.jar`

+   `dom4j-1.6.1.jar`

## 如何做到...

1.  创建一个自定义编组器类。

1.  在服务器端配置文件中注册新的编组器。

1.  设置服务器并运行客户端：

+   服务器项目根目录：`mvn clean package tomcat:run`

+   客户端项目根目录：`mvn clean package`

以下是服务器端输出：

```java
Received request ..
...
<ns:placeOrderRequest ...>
<ns:order customerTel="12345" customerfName="fName" customerlName="lName" dateSubmitted="2008-09-29 05:49:45" orderDate="2008-09-29 05:40:45" refNumber="12345">
<ns:item name="Snacks" quantity="1.0" type="SNACKS"/>
<ns:item name="Desserts" quantity="1.0" type="DESSERTS"/>
</ns:order>
</ns:placeOrderRequest>
....
Sent response...
<ns:placeOrderResponse  refNumber="12345"/> 

```

## 它是如何工作的...

为了能够作为端点编组器工作，自定义编组器（`ServerCustomMarshaller`）应该实现 `Marshaller` 和 `Unmarshaller` 接口。`supports` 方法用于验证 POJO 类是否已注册到该编组器。注册的 POJO 的值来自 Spring 上下文文件。

当 Web 服务调用端点方法（`handleOrderRequest`）构建传递参数（`PlaceOrderRequest`）时，将调用方法 `unmarshal`。在 `unmarshal` 方法中，使用 DOM4j 和 XPath 从传入的 XML 数据中提取值。这些值将填充 POJO 类并将其返回给端点。当端点方法（`handleOrderRequest`）返回响应（`PlaceOrderResponse`）时，将调用方法 `marshal`。在 `marshal` 方法内部，使用 `XMLStreamWriter` 将所需格式的 XML 数据返回给客户端：

```java
public boolean supports(Class<?> arg0) {
return registeredClassNames.contains(arg0.getSimpleName()) ; }
@Override
public Object unmarshal(Source source) throws IOException,
XmlMappingException {
PlaceOrderRequest placeOrderRequest=new PlaceOrderRequest();
Order order=new Order();
try {
DOMSource in = (DOMSource)source;
org.dom4j.Document document = org.dom4j.DocumentHelper.parseText( xmlToString(source) );
org.dom4j.Element orderRequestElem=document.getRootElement();
org.dom4j.Node orderNode=orderRequestElem.selectSingleNode("//ns:order");
order.setRefNumber(orderNode.valueOf("@refNumber"));
....
placeOrderRequest.setOrder(order);
List orderItems=orderNode.selectNodes("//ns:order/ns:item");
.....
}
@Override
public void marshal(Object bean, Result result) throws IOException,
XmlMappingException
{
XMLStreamWriter writer=null;
PlaceOrderResponse placeOrderResponse=(PlaceOrderResponse) bean;
try {
DOMResult out = (DOMResult)result;
writer = XMLOutputFactory.newInstance().createXMLStreamWriter(out);
writer.writeStartElement("ns", "placeOrderResponse", "http://www.packtpub.com/LiveRestaurant/OrderService/schema");
writer.writeAttribute( "refNumber", placeOrderResponse.getRefNumber());
writer.writeEndElement();
writer.flush();
} catch (Exception e) {
e.printStackTrace();
} finally{
try{writer.close();}catch (Exception e) {}
}
} .......

```

如前面的示例所述，服务器/客户端的应用程序上下文使用这个自定义编组器（`ServerCustomMarshaller`）来执行对象/XML 编组过程。`RegisteredClassNames` 用于注册符合条件的 POJO 类，以便通过自定义编组器（`ServerCustomMarshaller`）进行编组/解组。

```java
<bean id="customMarshaller"
class="com.packtpub.liverestaurant.marshaller.ServerCustomMarshaller">
<property name="registeredClassNames">
<list>
<value>PlaceOrderRequest</value>
<value>PlaceOrderResponse</value>
</list>
</property>
</bean>

```

`OrderEndPoint` 被注释为 `@Endpoint`，与 JiBX 示例相同，端点方法的输入和返回参数是映射到模式的 POJO（`PlaceOrderResponse`、`PlaceOrderRequest` 等）。
