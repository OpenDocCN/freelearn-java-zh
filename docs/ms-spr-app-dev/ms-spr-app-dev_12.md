# 第十二章：Spring 与 Web 服务集成

在本章中，我们将看到 Spring 如何支持`JAX_WS`网络服务，以及如何在**Spring Web Service** (**Spring-WS**)框架中创建网络服务。我们还将看到 Spring Web Service 如何被消费，演示一个客户端应用程序，以及 Spring 支持的 Web 服务的注解。

# Spring 与 JAX-WS

在本节中，让我们创建一个简单的 JAX-WS 网络服务。我们还将看到如何将 JAX-WS 网络服务与 Spring 集成。JAX-WS 是 JAX-RPC 的最新版本，它使用远程方法调用协议来访问 Web 服务。

我们在这里需要做的就是将 Spring 的服务层公开为`JAX_WS`服务提供程序层。这可以使用`@webservice`注解来完成，只需要几个步骤。让我们记下其中涉及的步骤。

1.  在 Eclipse 中创建一个`PACKTJAXWS-Spring`简单的 Maven web 项目或动态 web 项目。

1.  现在，我们需要在`web.xml`文件中配置 JAX-WS servlet：

```java
<?xml version="1.0" encoding="UTF-8"?>
<web-app   xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
<display-name>JAXWS-Spring</display-name>
<servlet>
  <servlet-name>jaxws-servlet</servlet-name>
  <servlet-class>
    com.sun.xml.ws.transport.http.servlet.WSSpringServlet
  </servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>jaxws-servlet</servlet-name>
  <url-pattern>/jaxws-spring</url-pattern>
</servlet-mapping>

<!-- Register Spring Listener -->
<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class> 
</listener> 
</web-app>
```

1.  创建一个`Context.xml`应用文件，并在其中添加网络服务信息。我们将在这里提供网络服务名称和服务提供者类信息。

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans  

       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
       http://jax-ws.dev.java.net/spring/core
       http://jax-ws.java.net/spring/core.xsd
       http://jax-ws.dev.java.net/spring/servlet
       http://jax-ws.java.net/spring/servlet.xsd">
  <wss:binding url="/jaxws-spring">
  <wss:service>
  <ws:service bean="#packWs"/>
  </wss:service>
  </wss:binding>
  <!-- Web service bean -->
  <bean id="packtWs" class="com.packt.webservicedemo.ws.PacktWebService">
  <property name="myPACKTBObject" ref="MyPACKTBObject" />
  </bean>
  <bean id="MyPACKTBObject" class="com.packt.webservicedemo.bo.impl.MyPACKTBObjectImpl" />
</beans>
```

1.  接下来，我们需要使所有的 jar 文件在类路径中可用。由于这是一个 Maven 项目，我们只需要更新`pom.xml`文件。

```java
<project   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.javacodegeeks.enterprise.ws</groupId>
  <artifactId>PACKTJAXWS-Spring</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-web</artifactId>
      <version>${spring.version}</version>
    </dependency>

    <dependency>
      <groupId>org.jvnet.jax-ws-commons.spring</groupId>
      <artifactId>jaxws-spring</artifactId>
      <version>1.9</version>
    </dependency>
  </dependencies>
  <properties>
    <spring.version>3.2.3.RELEASE</spring.version>
  </properties>
</project>
```

1.  我们现在将创建一个带有`@WebService`注解的网络服务类。我们还定义了可能需要的绑定类型，比如`SOAPBinding`和`Style`。`@Webmethod`注解指定了提供服务的方法。

```java
package com.packt.webservicedemo.ws;
import javax.jws.WebMethod;
import javax.jws.WebService;
import javax.jws.soap.SOAPBinding;
import javax.jws.soap.SOAPBinding.Style;
import javax.jws.soap.SOAPBinding.Use;
import com.packt.webservicedemo.bo.*;

@WebService(serviceName="PacktWebService")
@SOAPBinding(style = Style.RPC, use = Use.LITERAL)
public class PacktWebService{
  //Dependency Injection (DI) via Spring
  MyPACKTBObject myPACKTBObject;
  @WebMethod(exclude=true)
  public void setMyPACKTBObject(MyPACKTBObject myPACKTBObject) {
    this.myPACKTBObject = myPACKTBObject;
  }
  @WebMethod(operationName="printMessage")
  public String printMessage() {
    return myPACKTBObject.printMessage();

  }
}
package com.packt.webservicedemo.bo;
public interface MyPACKTBObject {
  String printMessage();
}
public class MyPACKTBObjectImpl implements MyPACKTBObject {
  @Override
  public String printMessage() {
    return "PACKT SPRING WEBSERVICE JAX_WS";
  }
}
```

1.  我们应该将 Maven JAR 文件添加到 Eclipse 项目的构建路径中。

1.  运行应用程序：`http://localhost:8080/PACKTJAXWS-Spring/jaxws-spring`。

您应该能够看到 WSDL URL，并在单击链接时，WSDL 文件应该打开。

# 使用 JAXB 编组的 Spring Web 服务请求

在本节中，让我们看看如何使用 Spring Web Service 框架开发一个简单的网络服务。我们需要 JAXB 来对 XML 请求进行编组和解组。Spring Web Service 支持契约优先的网络服务。我们需要首先设计 XSD/WSDL，然后启动网络服务。

我们正在创建一个作者网络服务，它将给我们提供作者列表。

1.  **配置 web.xml 文件**：让我们首先在`web.xml`文件中进行网络服务配置。我们需要配置 Spring Web Service servlet。需要定义消息分发 servlet 和它将处理的 URL 模式。指定`contextConfigLocation`而不是允许默认值(`/WEB-INF/spring-ws-servlet.xml`)，因为这个位置使得配置更容易与单元测试共享。

```java
<?xml version="1.0" encoding="UTF-8"?>
<web-app  

  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
  http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" 
  id="WebApp_ID" version="2.5">

  <servlet>
    <servlet-name>spring-ws</servlet-name>
    <servlet-class>org.springframework.ws.transport.http.MessageDispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:/spring-ws-context.xml</param-value>
    </init-param>
  </servlet>

  <servlet-mapping>
    <servlet-name>spring-ws</servlet-name>
    <url-pattern>/*</url-pattern>
  </servlet-mapping>

</web-app>
```

1.  **配置 Spring 上下文文件**(`/src/main/resources/spring-ws-context.xml`)：`EndPoint`类需要在`spring-ws-context.xml`中进行配置。该类带有`@EndPointAnnotation`注解。`AuthorEndpoint`被定义为一个 bean，并且将自动注册到 Spring Web Services 中，因为该类被`@Endpoint`注解标识为端点。此配置使用了`author.xsd`，这是一个用于生成 JAXB bean 以生成 WSDL 的 xml 模式描述符文件。位置 URI 与`web.xml`中指定的 URL 模式匹配。

使用 Spring OXM 配置 JAXB 编组器/解组器，并设置在`MarshallingMethodEndpointAdapter` bean 上。

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans 

       xsi:schemaLocation="http://www.springframework.org/schema/beans 
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

  <context:component-scan base-package="org. packtws.ws.service" />

  <bean id="person" class="org.springframework.ws.wsdl.wsdl11.DefaultWsdl11Definition"
    p:portTypeName="Author"
    p:locationUri="/authorService/"
    p:requestSuffix="-request"
    p:responseSuffix="-response">
    <property name="schema">
      <bean class="org.springframework.xml.xsd.SimpleXsdSchema"
        p:xsd="classpath:/author.xsd" />
      </bean>
    </property>
  </bean>

  <bean class="org.springframework.ws.server.endpoint.mapping.PayloadRootAnnotationMethodEndpointMapping">
    <description>An endpoint mapping strategy that looks for @Endpoint and @PayloadRoot annotations.</description>
  </bean>

  <bean class="org.springframework.ws.server.endpoint.adapter.MarshallingMethodEndpointAdapter">
    <description>Enables the MessageDispatchServlet to invoke methods requiring OXM marshalling.</description>
    <constructor-arg ref="marshaller"/>
  </bean>

  <bean id="marshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller"
    p:contextPath="org.packtws.author.schema.beans" />

</beans>
```

1.  **定义 XSD Author.xsd**：一个非常简单的 XSD 定义了一个元素，用于指示获取所有作者的传入请求（name 元素未使用），以及包含作者元素列表的作者响应元素。

**author.xsd**

```java
<xsd:schema 
  targetNamespace=" http://www.packtws.org/author/schema/beans "
  >

  <xsd:element name="get-authors-request">
  <xsd:complexType>
    <xsd:sequence>
      <xsd:element name="name" type="xsd:string" />
    </xsd:sequence>
  </xsd:complexType>
  </xsd:element>

  <xsd:element name="author-response">
    <xsd:complexType>
      <xsd:sequence>
        <xsd:element name="author" type="author"
          minOccurs="0" maxOccurs="unbounded"/>
      </xsd:sequence>
    </xsd:complexType>
  </xsd:element>

  <xsd:complexType name="author">
  <xsd:sequence>
    <xsd:element name="id" type="xsd:int" />
    <xsd:element name="first-name" type="xsd:string" />
    <xsd:element name="last-name" type="xsd:string" />
  </xsd:sequence>
  </xsd:complexType>

</xsd:schema>
```

1.  **编组 AuthorService**：让我们创建一个接口`MarshallingAuthorService`，用于使用以下 JAXB 生成的 bean 获取作者：

+   对于`get-authors-request`元素：`GetAuthorsRequst`

+   对于`author-response`元素：`AuthorResponse`

它还具有与命名空间（与 XSD 匹配）和请求常量相匹配的常量：

```java
public interface MarshallingAuthorService {
  public final static String NAMESPACE = " http://www.packtws.org/author/schema/beans ";
  public final static String GET_Authors_REQUEST = "get-authors-request";
  public AuthorResponse getAuthors(GetAuthorsRequest request);
}
```

1.  **创建端点类**：让我们创建一个标有`@Endpoint`注解的端点类。这个类将实现`MarshallingAuthorService`的方法。`getAuthors`方法被指示处理特定的命名空间和传入的请求元素。端点只是准备一个静态响应，但这很容易可以注入一个 DAO，并从数据库中检索信息，然后映射到 JAXB beans 中。AuthorResponse 是使用 JAXB Fluent API 创建的，比标准的 JAXB API 更简洁。

```java
@Endpoint
public class AuthorEndpoint implements MarshallingAuthorService {
  /**
  * Gets Author list.
  */
  @PayloadRoot(localPart=GET_AuthorS_REQUEST, namespace=NAMESPACE)
  public AuthorResponse getAuthors(GetPersonsRequest request) {
    return new AuthorResponse().withAuthor(
    new Author().withId(1).withFirstName("Anjana").withLastName("Raghavendra"),
    new Author().withId(2).withFirstName("Amrutha").withLastName("Prasad"));
  }

}
```

1.  **添加依赖信息**：还要确保在 maven 的`pom.xml`文件中添加以下依赖项：

```java
<dependency>
  <groupId>org.springframework.ws</groupId>
  <artifactId>org.springframework.ws</artifactId> 
  <version>${spring.ws.version}</version>
</dependency>
<dependency>
  <groupId>org.springframework.ws</groupId>
  <artifactId>org.springframework.ws.java5</artifactId> 
  <version>${spring.ws.version}</version>
</dependency>

<dependency>
  <groupId>javax.xml.bind</groupId>
  <artifactId>com.springsource.javax.xml.bind</artifactId>
  <version>2.1.7</version>
</dependency>
<dependency>
  <groupId>com.sun.xml</groupId>
  <artifactId>com.springsource.com.sun.xml.bind.jaxb1</artifactId>
  <version>2.1.7</version>
</dependency>
<dependency>
  <groupId>javax.wsdl</groupId>
  <artifactId>com.springsource.javax.wsdl</artifactId>
  <version>1.6.1</version>
</dependency>
<dependency>
  <groupId>javax.xml.soap</groupId>
  <artifactId>com.springsource.javax.xml.soap</artifactId>
  <version>1.3.0</version>
</dependency>
<dependency>
  <groupId>com.sun.xml</groupId>
  <artifactId>com.springsource.com.sun.xml.messaging.saaj</artifactId>
  <version>1.3.0</version>
</dependency>
<dependency>
  <groupId>javax.activation</groupId>
  <artifactId>com.springsource.javax.activation</artifactId>
  <version>1.1.1</version>
</dependency>
<dependency>
  <groupId>javax.xml.stream</groupId>
  <artifactId>com.springsource.javax.xml.stream</artifactId>
  <version>1.0.1</version>
</dependency>
```

1.  **构建和部署应用程序**：我们需要在 tomcat 上进行这个操作以查看 WSDL URL。因此，我们已经完成了提供 web 服务的所有步骤。

# 使用 JAXB unmarshalling 为 Spring Web Services 编写客户端应用程序

让我们为作者服务编写一个简单的客户端应用程序。`org.springbyexample.ws.service`包被扫描以查找`AuthorServiceClient`，并将 web 服务模板注入其中。JAXB marshaller/umarshaller 被定义并设置在这个模板上。

`jetty-context.xml`的导入对于创建客户端并不重要，但它创建了一个嵌入式的 Jetty 实例，加载了`spring-ws-context.xml`和它的服务。单元测试中的客户端能够独立运行。

**AuthorServiceClientTest.xml**：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans 

  xsi:schemaLocation="http://www.springframework.org/schema/beans 
  http://www.springframework.org/schema/beans/spring-beans.xsd
  http://www.springframework.org/schema/context 
  http://www.springframework.org/schema/context/spring-context.xsd">

  <import resource="jetty-context.xml"/>

  <context:component-scan base-package="org.springbyexample.ws.client" />

  <context:property-placeholder location="org/springbyexample/ws/client/ws.properties"/>

  <bean id="authorWsTemplate" class="org.springframework.ws.client.core.WebServiceTemplate"
  p:defaultUri="http://${ws.host}:${ws.port}/${ws.context.path}/authorService/"
  p:marshaller-ref="marshaller"
  p:unmarshaller-ref="marshaller" />

  <bean id="marshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller"
  p:contextPath="org.springbyexample.author.schema.beans" />

</beans>
```

**AuthorServiceClient**：

在这一点上，Spring Web Services 几乎可以处理所有事情。只需要调用模板，它将从服务端点返回`AuthorResponse`。客户端可以像这样使用：`AuthorResponse response = client.getAuthors(new GetAuthorsRequest());`

```java
public class AuthorServiceClient implements MarshallingAuthorService {

  @Autowired
  private WebServiceTemplate wsTemplate;

  /**
    * Gets author list.
  */
  public AuthorResponse getAuthors(GetAuthorsRequest request) {
    PersonResponse response = (PersonResponse) wsTemplate.marshalSendAndReceive(request);

    return response;

  }
}
```

# 总结

在本章中，我们看到了如何将`JAX_WS`与 Spring Web Service 集成。我们还演示了如何创建 Spring Web Services 和端点类，以及如何通过访问 WSDL URL 来访问 web 服务。
