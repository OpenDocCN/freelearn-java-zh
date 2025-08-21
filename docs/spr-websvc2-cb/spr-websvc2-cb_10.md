# 第十章：Spring 远程

在本章中，我们将涵盖：

+   使用 RMI 设置 Web 服务

+   使用 Hessian/Burlap 设置基于 servlet 的 Web 服务，暴露业务 bean

+   使用 JAX-WS 设置 Web 服务

+   使用 Apache CXF 暴露基于 servlet 的 Web 服务

+   使用 JMS 作为底层通信协议暴露 Web 服务

# 介绍

Spring-WS 项目是一种基于契约的方法来构建 Web 服务。这种方法已经在前八章中详细介绍过。然而，有时的要求是将现有的业务 Spring bean 暴露为 Web 服务，这被称为**契约后**方法，用于设置 Web 服务。

Spring 的远程支持与多种远程技术的通信。Spring 远程允许在服务器端暴露现有的 Spring bean 作为 Web 服务。在客户端，Spring 远程允许客户端应用程序通过本地接口调用远程 Spring bean（该 bean 作为 Web 服务暴露）。在本章中，详细介绍了 Spring 的以下远程技术的功能：

+   RMI：Spring 的`RmiServiceExporter`允许您在服务器端使用远程方法调用（RMI）暴露本地业务服务，而 Spring 的`RmiProxyFactoryBean`是客户端代理 bean，用于调用 Web 服务。

+   Hessian：Spring 的`HessianServiceExporter`允许您在服务器端使用 Caucho 技术引入的轻量级基于 HTTP 的协议暴露本地业务服务，而`HessianProxyFactoryBean`是调用 Web 服务的客户端代理 bean。

+   Burlap：这是 Caucho Technology 的 Hessian 的 XML 替代方案。Spring 提供了支持类，使用 Spring 的两个 bean，即`BurlapProxyFactoryBean`和`BurlapServiceExporter`。

+   JAX-RPC：Spring 支持设置 Web 服务，基于 J2EE 1.4 的 JAX-RPC Web 服务 API

+   JAX-WS：Spring 支持使用 Java EE 5+ JAX-WS API 设置 Web 服务，该 API 允许基于消息和远程过程调用的 Web 服务开发。

+   JMS：Spring 使用 JMS 作为底层通信协议来暴露/消费 Web 服务，使用`JmsInvokerServiceExporter`和`JmsInvokerProxyFactoryBean`类。

由于 JAX-WS 是 JAX-RPC 的后继者，因此本章不包括 JAX-RPC。相反，本章将详细介绍 Apache CXF，因为它可以使用 JAX-WS 来设置 Web 服务，即使它不是 Spring 的远程的一部分。

为简化起见，在本章中，将暴露以下本地业务服务作为 Web 服务（领域模型在第一章的*介绍*部分中已经描述，*构建 SOAP Web 服务*）。

```java
public interface OrderService {
placeOrderResponse placeOrder(PlaceOrderRequest placeOrderRequest);
}

```

这是接口实现：

```java
public class OrderServiceImpl implements OrderService{
public PlaceOrderResponse placeOrder(PlaceOrderRequest placeOrderRequest) {
PlaceOrderResponse response=new PlaceOrderResponse();
response.setRefNumber(getRandomOrderRefNo());
return response;
}
...

```

# 使用 RMI 设置 Web 服务

RMI 是 J2SE 的一部分，允许在不同的 Java 虚拟机（JVM）上调用方法。RMI 的目标是在单独的 JVM 中公开对象，就像它们是本地对象一样。通过 RMI 调用远程对象的客户端不知道对象是远程还是本地，并且在远程对象上调用方法与在本地对象上调用方法具有相同的语法。

Spring 的远程提供了基于 RMI 技术的暴露/访问 Web 服务的功能。在服务器端，Spring 的`RmiServiceExporter` bean 将服务器端 Spring 业务 bean 暴露为 Web 服务。在客户端，Spring 的`RmiProxyFactoryBean`将 Web 服务的方法呈现为本地接口。

在这个示例中，我们将学习使用 RMI 设置 Web 服务，并了解通过 RMI 呼叫 Web 服务的呈现方式。

## 准备工作

在这个示例中，我们有以下两个项目：

1.  `LiveRestaurant_R-10.1`（用于服务器端 Web 服务），具有以下 Maven 依赖项：

+   `spring-context-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

1.  `LiveRestaurant_R-10.1-Client`（用于客户端），具有以下 Maven 依赖项：

+   `spring-context-3.0.5.RELEASE.jar`

+   `spring-ws-test-2.0.0.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `junit-4.7.jar`

+   `xmlunit-1.1.jar`

## 如何做...

1.  在服务器端应用程序上下文（`applicationContext.xml`）中注册服务器端服务实现在 Spring 的`RmiServiceExporter`中，并设置端口和服务名称。

1.  在客户端应用程序上下文（`applicationContext.xml`）中，使用 Spring 的`RmiProxyFactoryBean`注册本地接口（与服务器端相同）并设置服务的 URL。

1.  添加一个 Java 类来加载服务器端应用程序上下文文件（在类的`main`方法中）以设置服务器。

1.  在客户端添加一个 JUnit 测试用例类，通过本地接口调用 Web 服务。

1.  在`Liverestaurant_R-10.1`上运行以下命令：

```java
mvn clean package exec:java 

```

1.  在`Liverestaurant_R-10.1-Client`上运行以下命令：

```java
mvn clean package 

```

+   以下是客户端输出：

```java
......
... - Located RMI stub with URL [rmi://localhost:1199/OrderService]
....- RMI stub [rmi://localhost:1199/OrderService] is an RMI invoker
......
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.78 sec
...
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
......
[INFO] BUILD SUCCESS 

```

## 工作原理...

`OrderServiceSetUp`是加载服务器端应用程序上下文并设置服务器以将服务器端业务服务暴露为 Web 服务的类。`OrderServiceClientTest`是客户端测试类，加载客户端应用程序上下文并通过代表远程业务服务的客户端本地接口调用 Web 服务方法。

`OrderServiceImpl`是要通过 Web 服务公开的服务。在服务器端的应用程序上下文中，在`org.springframework.remoting.rmi.RmiServiceExporter` Bean 中，`OrderService`是将在 RMI 注册表中注册的服务的名称。服务属性用于传递`RmiServiceExporter`和 bean 实例。`serviceInterface`是表示本地业务服务的接口。只有在此接口中定义的方法才能远程调用：

```java
<bean id="orderService" class="com.packtpub.liverestaurant.service.OrderServiceImpl" />
<bean class="org.springframework.remoting.rmi.RmiServiceExporter">
<property name="serviceName" value="OrderService" />
<property name="service" ref="orderService" />
<property name="serviceInterface" value="com.packtpub.liverestaurant.service.OrderService" />
<property name="registryPort" value="1199" />
</bean>

```

在客户端配置文件中，`serviceUrl`是 Web 服务的 URL 地址，`serviceInterface`是本地接口，使客户端可以远程调用服务器端的方法：

```java
<bean id="orderService" class="org.springframework.remoting.rmi.RmiProxyFactoryBean">
<property name="serviceUrl" value=" rmi://localhost:1199/OrderService" />
<property name="serviceInterface" value="com.packtpub.liverestaurant.service.OrderService" />
</bean>

```

`OrderServiceClientTest`是加载应用程序上下文并通过本地接口调用远程方法的 JUnit 测试用例类：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("/applicationContext.xml")
public class OrderServiceClientTest {
@Autowired
OrderService orderService;
@Autowired
private GenericApplicationContext applicationContext;
@Before
@After
public void setUpAfter() {
applicationContext.close();
}
@Test
public final void testPlaceOrder() throws Exception {
PlaceOrderRequest orderRequest = new PlaceOrderRequest();
orderRequest.setOrder(getDummyOrder());
PlaceOrderResponse orderResponse = orderService.placeOrder(orderRequest);
Assert.assertTrue(orderResponse.getRefNumber().indexOf("1234")>0);
}
private Order getDummyOrder() {
Order order=new Order();
order.setRefNumber("123");
List<FoodItem> items=new ArrayList<FoodItem>();
FoodItem item1=new FoodItem();
item1.setType(FoodItemType.BEVERAGES);
item1.setName("beverage");
item1.setQuantity(1.0);
......
}
........
}

```

# 使用 Hessian/Burlap 设置基于 servlet 的 Web 服务，暴露业务 bean

**Hessian 和 Burlap**，由 Caucho 开发（[`hessian.caucho.com`](http://hessian.caucho.com)），是轻量级基于 HTTP 的远程技术。尽管它们都使用 HTTP 协议进行通信，但 Hessian 使用二进制消息进行通信，而 Burlap 使用 XML 消息进行通信。

Spring 的远程提供了基于这些技术的 Web 服务的暴露/访问功能。在服务器端，Spring 的`ServiceExporter` bean 将服务器端 Spring 业务 bean（`OrderServiceImpl`）暴露为 Web 服务：

```java
<bean id="orderService" class="com.packtpub.liverestaurant.service.OrderServiceImpl" />
<bean name="/OrderService" class="....ServiceExporter">
<property name="service" ref="orderService" />
</bean>

```

在客户端，Spring 的`ProxyFactory` bean 通过本地客户端接口（`OrderService`）暴露远程接口：

```java
<bean id="orderService" class="....ProxyFactoryBean">
<property name="serviceUrl" value="http://localhost:8080/LiveRestaurant/services/OrderService" />
<property name="serviceInterface" value="com.packtpub.liverestaurant.service.OrderService" />

```

## 准备工作

在这个示例中，我们有以下两个项目：

1.  `LiveRestaurant_R-10.2`（用于服务器端 Web 服务），具有以下 Maven 依赖项：

+   `spring-webmvc-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `hessian-3.1.5.jar`

1.  `LiveRestaurant_R-10.2-Client`（用于客户端），具有以下 Maven 依赖项：

+   `spring-web-3.0.5.RELEASE.jar`

+   `spring-test-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `junit-4.7.jar`

+   `hessian-3.1.5.jar`

## 如何做...

按照以下步骤设置基于 servlet 的 Web 服务，使用 Hessian 服务：

1.  在`web.xml`文件中配置`DispatcherServlet`（URL：`http://<host>:<port>/<appcontext>/services`将被转发到此 servlet）。

1.  在服务器端应用程序上下文（`applicationContext.xml`）中注册服务器端服务接口，并设置服务名称和服务接口。

1.  在客户端应用程序上下文（`applicationContext.xml`）中，使用 Spring 的`HessianProxyFactoryBean`注册本地接口（与服务器端相同），并设置服务的 URL。

1.  在客户端添加一个 JUnit 测试用例类，使用本地接口调用 Web 服务

1.  在`Liverestaurant_R-10.2`上运行以下命令：

```java
mvn clean package tomcat:run 

```

1.  在`Liverestaurant_R-10.2-Client`上运行以下命令：

```java
mvn clean package 

```

+   在客户端输出中，您将能够看到运行测试用例的成功消息，如下所示：

```java
text.annotation.internalCommonAnnotationProcessor]; root of factory hierarchy
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.71 sec
Results :
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] 

```

按照以下步骤使用 Burlap 服务设置基于 servlet 的 Web 服务：

1.  将服务器端服务接口修改为 Spring 的`BurlapServiceExporter`，在服务器端应用程序上下文（`applicationContext.xml`）中。

1.  将客户端应用程序上下文（`applicationContext.xml`）修改为 Spring 的`BurlapProxyFactoryBean`。

1.  在`Liverestaurant_R-10.2`上运行以下命令：

```java
mvn clean package tomcat:run 

```

1.  在`Liverestaurant_R-10.2-Client`上运行以下命令：

```java
mvn clean package 

```

+   在客户端输出中，您将能够看到运行测试用例的成功消息，如下所示：

```java
text.annotation.internalCommonAnnotationProcessor]; root of factory hierarchy
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.849 sec
Results :
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] --- maven-jar-plugin:2.3.1:jar ..
[INFO] Building jar: ...
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS 

```

## 它是如何工作的...

`Liverestaurant_R-10.2`项目是一个服务器端 Web 服务，使用 Spring 远程的 burlap/hessian 出口商设置基于 servlet 的 Web 服务。

`Liverestaurant_R-10.2-Client`项目是一个客户端测试项目，调用了 Spring 远程的 burlap/hessian Web 服务，使用了 burlap/hessian 客户端代理。

在服务器端，`DiapatcherServlet`将使用 URL 模式将所有请求转发到`BurlapServiceExporter/HessianServiceExporter`（http://<hostaddress>/<context>/<services>）：

```java
<servlet>
<servlet-name>order</servlet-name>
<servlet-class>
org.springframework.web.servlet.DispatcherServlet
</servlet-class>
<load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
<servlet-name>order</servlet-name>
<url-pattern>/services/*</url-pattern>
</servlet-mapping>

```

这些出口商将内部本地服务实现（`OrderServiceImpl`）公开为 Web 服务：

```java
<bean name="/OrderService" class="org.springframework.remoting.caucho.BurlapServiceExporter">
<property name="service" ref="orderService" />
<property name="serviceInterface" value="com.packtpub.liverestaurant.service.OrderService" />
</bean>

```

在客户端，`BurlapProxyFactoryBean/HessianProxyFactoryBean`负责使用本地客户端服务接口（`OrderService`）向客户端公开远程方法：

```java
<bean id="orderService" class="org.springframework.remoting.caucho.BurlapProxyFactoryBean">
<property name="serviceUrl" value="http://localhost:8080/LiveRestaurant/services/OrderService" />
<property name="serviceInterface" value="com.packtpub.liverestaurant.service.OrderService" />
</bean>

```

`OrderServiceClientTest`的实现与食谱*使用 RMI 设置 Web 服务*中描述的相同。

## 另请参阅...

在本章中：

*使用 RMI 设置 Web 服务*

# 使用 JAX-WS 设置 Web 服务

**JAX-RPC**是 Java EE 1.4 中附带的一个标准，用于开发 Web 服务，在近年来变得越来越不受欢迎。JAX-WS 2.0 是在 Java EE 5 中引入的，比 JAX-RPC 更灵活，基于注解的绑定概念。以下是 JAX-WS 相对于 JAX-RPC 的一些优势：

+   JAX-WS 支持面向消息和**远程过程调用（RPC）**Web 服务，而 JAX-RPC 仅支持 RPC

+   JAX-WS 支持 SOAP 1.2 和 SOAP 1.1，但 JAX-RPC 支持 SOAP 1.1

+   JAX-WS 依赖于 Java 5.0 的丰富功能，而 JAX-RPC 与 Java 1.4 一起工作

+   JAX-WS 使用非常强大的 XML 对象映射框架（使用 JAXB），而 JAX-RPC 使用自己的框架，对于复杂的数据模型显得薄弱

Spring 远程提供了设置使用 Java 1.5+功能的 JAX-WS Web 服务的功能。例如，在这里，注解`@WebService`会导致 Spring 检测并将此服务公开为 Web 服务，`@WebMethod`会导致以下方法：`public OrderResponse placeOrder(..)`，被调用为 Web 服务方法（placeOrder）：

```java
@Service("OrderServiceImpl")
@WebService(serviceName = "OrderService",endpointInterface = "com.packtpub.liverestaurant.service.OrderService")
public class OrderServiceImpl implements OrderService {
@WebMethod(operationName = "placeOrder")
public PlaceOrderResponse placeOrder(PlaceOrderRequest placeOrderRequest) {

```

在这个食谱中，使用 JDK 内置的 HTTP 服务器来设置 Web 服务（自从 Sun 的`JDK 1.6.0_04`以来，JAX-WS 可以与 JDK 内置的 HTTP 服务器集成）。

## 准备工作

安装 Java 和 Maven（SE 运行时环境（构建`jdk1.6.0_29`））。

在这个食谱中，我们有以下两个项目：

1.  `LiveRestaurant_R-10.3`（用于服务器端 Web 服务），具有以下 Maven 依赖项：

+   `spring-web-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

1.  `LiveRestaurant_R-10.3-Client`（用于客户端），具有以下 Maven 依赖项：

+   `spring-web-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `junit-4.7.jar`

## 如何做...

1.  为业务服务类及其方法添加注释。

1.  在应用程序上下文文件（`applicationContext.xml`）中注册服务，然后配置`SimpleJaxWsServiceExporter` bean，并创建一个类来加载服务器端应用程序上下文（这将设置服务器）。

1.  在客户端应用程序上下文（`applicationContext.xml`）中注册本地接口（与服务器端接口相同的方式），并设置服务的 URL。

1.  在客户端添加一个 JUnit 测试用例类，该类使用本地接口调用 Web 服务。

1.  在`Liverestaurant_R-10.3`上运行以下命令，并浏览以查看位于`http://localhost:9999/OrderService?wsdl`的 WSDL 文件：

```java
mvn clean package exec:java 

```

1.  在`Liverestaurant_R-10.3-Client`上运行以下命令：

```java
mvn clean package 

```

+   在客户端输出中，您将能够看到运行测试用例的成功消息，如下所示：

```java
.....
Dynamically creating request wrapper Class com.packtpub.liverestaurant.service.jaxws.PlaceOrder
Nov 14, 2011 11:34:13 PM com.sun.xml.internal.ws.model.RuntimeModeler getResponseWrapperClass
INFO: Dynamically creating response wrapper bean Class com.packtpub.liverestaurant.service.jaxws.PlaceOrderResponse
......
Results :
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0 

```

## 它是如何工作的...

`Liverestaurant_R-10.3`项目是一个服务器端 Web 服务（通过 Spring 远程的出口器 bean），它使用 DK 内置的 HTTP 服务器设置了一个 JAX-WS。

`Liverestaurant_R-10.3-Client`项目是一个客户端测试项目，它使用 Spring 远程的客户端代理调用 JAX-WS Web 服务。

在服务器端，`applicationContext.xml`扫描并检测`OrderServiceImpl`中的注释标签。然后，`SimpleJaxWsServiceExporter`将此业务服务公开为 Web 服务：

```java
<context:annotation-config/>
<context:component-scan base-package= "com.packtpub.liverestaurant.service"/>
<bean class= "org.springframework.remoting.jaxws.SimpleJaxWsServiceExporter">
<property name="baseAddress" value="http://localhost:9999/" />
</bean>

```

在服务类中，注释`@WebService`和`@WebMethod`导致 Spring 检测（通过扫描），并通过`SimpleJaxWsServiceExporter`将此服务类公开为 Web 服务及其方法（`placeOrder`）公开为 Web 服务方法：

```java
@Service("orderServiceImpl")
@WebService(serviceName = "OrderService")
public class OrderServiceImpl implements OrderService {
@WebMethod(operationName = "placeOrder")
public PlaceOrderResponse placeOrder(PlaceOrderRequest placeOrderRequest) {
PlaceOrderResponse response=new PlaceOrderResponse();
response.setRefNumber(getRandomOrderRefNo());
return response;
}
.......
}

```

在客户端，`JaxWsPortProxyFactoryBean`负责将远程方法暴露给客户端，使用本地客户端接口。`WsdlDocumentUrl`是 Web 服务 WSDL 地址，`portName`是 WSDL 中的`portName`值，`namespaceUri`是 WSDL 中的`targetNameSpace`，`serviceInterface`是本地客户端服务接口：

```java
<bean id="orderService" class= "org.springframework.remoting.jaxws.JaxWsPortProxyFactoryBean">
<property name="serviceInterface" value= "com.packtpub.liverestaurant.service.OrderService"/>
<property name="serviceInterface" value= "com.packtpub.liverestaurant.service.OrderService"/>
<property name="wsdlDocumentUrl" value= "http://localhost:9999/OrderService?wsdl"/>
<property name="namespaceUri" value= "http://service.liverestaurant.packtpub.com/"/>
<property name="serviceName" value="OrderService"/>
<property name="portName" value="OrderServiceImplPort"/>
</bean>

```

`OrderServiceClientTest`的实现与名为*使用 RMI 设置 Web 服务*的配方中描述的相同。

## 另请参阅...

在本章中：

*使用 RMI 设置 Web 服务*

在本书中：

第二章,*构建 SOAP Web 服务的客户端*

*在 HTTP 传输上创建 Web 服务客户端*

# 使用 Apache CXF 暴露基于 servlet 的 Web 服务

**Apache CXF**起源于以下项目的组合：**Celtix**（IONA Technologies）和**XFire**（Codehaus），它们被整合到**Apache 软件基金会**中。CXF 的名称意味着它起源于**Celtix**和**XFire**项目名称。

Apache CXF 提供了构建和部署 Web 服务的功能。Apache CXF 推荐的 Web 服务配置方法（前端或 API）是 JAX-WS 2.x。Apache CXF 并不是 Spring 的远程的一部分，但是，由于它可以使用 JAX-WS 作为其前端，因此将在本配方中进行解释。

## 准备工作

安装 Java 和 Maven（SE Runtime Environment（构建`jdk1.6.0_29`））。

在这个配方中，我们有以下两个项目：

1.  `LiveRestaurant_R-10.4`（用于服务器端 Web 服务），具有以下 Maven 依赖项：

+   `cxf-rt-frontend-jaxws-2.2.6.jar`

+   `cxf-rt-transports-http-2.2.6.jar`

+   `spring-web-3.0.5.RELEASE.jar`

+   `commons-logging-1.1.1.jar`

1.  `LiveRestaurant_R-10.4-Client`（用于客户端），具有以下 Maven 依赖项：

+   `cxf-rt-frontend-jaxws-2.2.6.jar`

+   `cxf-rt-transports-http-2.2.6.jar`

+   `spring-web-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `junit-4.7.jar`

## 如何做...

1.  在业务服务类和方法上进行注释（与您为 JAX-WS 所做的方式相同）。

1.  在应用程序上下文文件（`applicationContext.xml`）中注册服务，并在`web.xml`文件中配置`CXFServlet`（URL：`http://<host>:<port>/`将被转发到此 servlet）。

1.  在客户端应用程序上下文（`applicationContext.xml`）中注册本地接口（与您为服务器端执行的方式相同），并设置服务的 URL。

1.  在客户端添加一个 JUnit 测试用例类，使用本地接口调用 Web 服务。

## 工作原理...

`Liverestaurant_R-10.4`项目是一个服务器端 Web 服务，它使用 JAX-WS API 设置了一个 CXF。

`Liverestaurant_R-10.4-Client`项目是一个客户端测试项目，它使用 Spring 的远程调用从 JAX-WS Web 服务调用客户端代理。

在服务器端，`applicationContext.xml`中的配置检测`OrderServiceImpl`中的注释标签。然后`jaxws:endpoint`将此业务服务公开为 Web 服务：

```java
<!-- Service Implementation -->
<bean id="orderServiceImpl" class= "com.packtpub.liverestaurant.service.OrderServiceImpl" />
<!-- JAX-WS Endpoint -->
<jaxws:endpoint id="orderService" implementor="#orderServiceImpl" address="/OrderService" />

```

`OrderServiceImpl`的解释与在*使用 JAX-WS 设置 Web 服务*中描述的相同。

在客户端，`JaxWsProxyFactoryBean`负责使用本地客户端接口向客户端公开远程方法。`address`是 Web 服务的地址，`serviceInterface`是本地客户端服务接口：

```java
<bean id="client" class= "com.packtpub.liverestaurant.service.OrderService" factory-bean="clientFactory" factory-method="create"/>
<bean id="clientFactory" class="org.apache.cxf.jaxws.JaxWsProxyFactoryBean">
<property name="serviceClass" value="com.packtpub.liverestaurant.service.OrderService"/>
<property name="address" value="http://localhost:8080/LiveRestaurant/OrderService"/>
</bean>

```

`OrderServiceClientTest`的实现与在*使用 RMI 设置 Web 服务*中描述的相同。

## 另请参阅...

*在本章中：*

*使用 RMI 设置 Web 服务*

# 使用 JMS 作为底层通信协议公开 Web 服务

**Java 消息服务（JMS）**由 Java 2 和 J2EE 引入，由 Sun Microsystems 于 1999 年成立。使用 JMS 的系统能够以同步或异步模式进行通信，并且基于点对点和发布-订阅模型。

Spring 远程提供了使用 JMS 作为底层通信协议公开 Web 服务的功能。Spring 的 JMS 远程在单线程和非事务会话中在同一线程上发送和接收消息。

但是，对于 JMS 上的 Web 服务的多线程和事务支持，您可以使用基于 Spring 的 JMS 协议的 Spring-WS，该协议基于 Spring 的基于 JMS 的消息传递。

在这个示例中，使用`apache-activemq-5.4.2`来设置一个 JMS 服务器，并且默认对象，由这个 JMS 服务器创建的（队列，代理），被项目使用。

## 准备就绪

安装 Java 和 Maven（SE Runtime Environment（构建`jdk1.6.0_29`））。

安装`apache-activemq-5.4.2`。

在这个示例中，我们有以下两个项目：

1.  `LiveRestaurant_R-10.5`（用于服务器端 Web 服务），具有以下 Maven 依赖项：

+   `activemq-all-5.2.0.jar`

+   `spring-jms-3.0.5.RELEASE.jar`

1.  `LiveRestaurant_R-10.5-Client`（用于客户端），具有以下 Maven 依赖项：

+   `activemq-all-5.2.0.jar`

+   `spring-jms-3.0.5.RELEASE.jar`

+   `junit-4.7.jar`

+   `spring-test-3.0.5.RELEASE.jar`

+   `xmlunit-1.1.jar`

## 如何做...

在服务器端应用程序上下文文件中注册业务服务到`JmsInvokerServiceExporter` bean，并使用`activemq`默认对象（代理，`destination`）注册`SimpleMessageListenerContainer`。

1.  创建一个 Java 类来加载应用程序上下文并设置服务器。

1.  在客户端应用程序上下文文件中使用`activemq`默认对象（代理，目的地）注册`JmsInvokerProxyFactoryBean`。

1.  在客户端添加一个 JUnit 测试用例类，调用本地接口使用 Web 服务。

1.  运行`apache-activemq-5.4.2`（设置 JMS 服务器）。

1.  在`Liverestaurant_R-10.5`上运行以下命令并浏览以查看位于`http://localhost:9999/OrderService?wsdl`的 WSDL 文件：

```java
mvn clean package exec:java 

```

1.  在`Liverestaurant_R-10.5-Client`上运行以下命令：

```java
mvn clean package 

```

+   在客户端输出中，您将能够看到运行测试用例的成功消息。

```java
T E S T S
-------------------------------------------------------
Running com.packtpub.liverestaurant.service.client.OrderServiceClientTest
log4j:WARN No appenders could be found for logger (org.springframework.test.context.junit4.SpringJUnit4ClassRunner).
log4j:WARN Please initialize the log4j system properly.
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.138 sec
Results :
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0 

```

## 工作原理...

`Liverestaurant_R-10.5`项目是一个服务器端 Web 服务，它通过监听 JMS 队列设置了一个 Web 服务。

`Liverestaurant_R-10.5-Client`项目是一个客户端测试项目，它向 JMS 队列发送 JMS 消息。

在服务器端，`OrderServiceSetUp` 类加载 `applicationContext.xml` 并在容器中创建一个 `messageListener`（使用 `SimpleMessageListenerContainer`），等待在特定目的地（`requestQueue`）监听消息。一旦消息到达，它通过 Spring 的远程调用类（`JmsInvokerServiceExporter`）调用业务类（`OrderServiceImpl`）的方法。

```java
<bean id="orderService" class="com.packtpub.liverestaurant.service.OrderServiceImpl"/>
<bean id="listener" class="org.springframework.jms.remoting.JmsInvokerServiceExporter">
<property name="service" ref="orderService"/>
<property name="serviceInterface" value="com.packtpub.liverestaurant.service.OrderService"/>
</bean>
<bean id="container" class= "org.springframework.jms.listener.SimpleMessageListenerContainer">
<property name="connectionFactory" ref="connectionFactory"/>
<property name="messageListener" ref="listener"/>
<property name="destination" ref="requestQueue"/>
</bean>

```

在客户端，`JmsInvokerProxyFactory` 负责使用本地客户端接口（OrderService）向客户端公开远程方法。当客户端调用 `OrderService` 方法时，`JmsInvokerProxyFactory` 会向队列（requestQueue）发送一个 JMS 消息，这是服务器正在监听的队列：

```java
<bean id="orderService" class= "org.springframework.jms.remoting.JmsInvokerProxyFactoryBean">
<property name="connectionFactory" ref="connectionFactory"/>
<property name="queue" ref="requestQueue"/>
<property name="serviceInterface" value="com.packtpub.liverestaurant.service.OrderService"/>
</bean>

```
