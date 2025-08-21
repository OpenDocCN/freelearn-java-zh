# 第七章：与其他 Web 框架集成

Spring 框架提供的灵活性可以选择第三方产品是 Spring 的核心价值主张之一，Spring 支持与第三方表示框架的集成。虽然 Spring 的表示层框架 Spring MVC 为 Web 应用程序的开发带来了最大程度的灵活性和效率，但 Spring 允许您集成最流行的表示框架。

Spring 可以与 Java 的太多 Web 框架集成，以至于无法在本章中包括所有，只有最流行的 JSF 和 Struts 将被解释。

# Spring 的 JSF 集成

JSF Web 应用程序可以通过在`web.xml`中加载 Spring 上下文文件（通过上下文加载器监听器）轻松集成 Spring。自 JSF 1.2 以来，Spring 的`SpringBeanFacesELResolver`对象将 Spring bean 读取为 JSF 托管 bean。JSF 只处理表示层，并且具有名为`FacesServlet`的控制器。我们只需要在应用程序部署描述符或`web.xml`中注册`FacesServlet`（在本节中，我们使用 JavaConfig 进行注册），并将任何请求与所需扩展名（这里是`.xhtml`）映射到`FacesServlet`。

首先，我们应该在项目依赖项中包含 JSF API 及其实现：

```java
<properties>
  <spring-framework-version>4.1.6.RELEASE</spring-framework-version>
  <mojarra-version>2.2.12</mojarra-version>
</properties>
  ...
<dependency>
  <groupId>com.sun.faces</groupId>
  <artifactId>jsf-api</artifactId>
  <version>${mojarra-version}</version>
</dependency>
<dependency>
  <groupId>com.sun.faces</groupId>
  <artifactId>jsf-impl</artifactId>
  <version>${mojarra-version}</version>
</dependency>
...
```

调度程序 Servlet 初始化程序是注册`FacesServlet`的位置。请注意，我们在此处将请求映射到`FacesServlet`。由于我们使用 JavaConfig 来注册设置，因此我们在`AnnotationConfigDispchServletInit`类中注册`FacesServlet`，如下所示：

```java
@Configuration
@Order(2)
public class AnnotationConfigDispchServletInit extends AbstractAnnotationConfigDispatcherServletInitializer {
  @Override
  protected Class<?>[] getRootConfigClasses() {
    return new Class<?>[] { AppConfig.class };
  }
  @Override
  protected Class<?>[] getServletConfigClasses() {
    return null;
  }
  @Override
  protected String[] getServletMappings() {
    return new String[] { "*.xhtml" };
  }
  @Override
  protected Filter[] getServletFilters() {
    return new Filter[] { new CharacterEncodingFilter() };
  }
  @Override
  public void onStartup(ServletContext servletContext) throws ServletException {
    // Use JSF view templates saved as *.xhtml, for use with // Facelets
    servletContext.setInitParameter("javax.faces.DEFAULT_SUFFIX", ".xhtml");
    // Enable special Facelets debug output during development
    servletContext.setInitParameter("javax.faces.PROJECT_STAGE", "Development");
    // Causes Facelets to refresh templates during development
    servletContext.setInitParameter("javax.faces.FACELETS_REFRESH_PERIOD", "1");
    servletContext.setInitParameter("facelets.DEVELOPMENT", "true");
    servletContext.setInitParameter("javax.faces.STATE_SAVING_METHOD", "server");
    servletContext.setInitParameter(
      "javax.faces.PARTIAL_STATE_SAVING_METHOD", "true");
      servletContext.addListener(com.sun.faces.config.ConfigureListener.class);
    ServletRegistration.Dynamic facesServlet = servletContext.addServlet("Faces Servlet", FacesServlet.class);
    facesServlet.setLoadOnStartup(1);
    facesServlet.addMapping("*.xhtml");
    // Let the DispatcherServlet be registered
    super.onStartup(servletContext);
  }
}
```

### 注意

我们必须在其他之前设置`FacesServlet`以在加载时启动（注意`facesServlet.setLoadOnStartup`）。

另一个重要的设置是配置监听器以读取`faces-config` XML 文件。默认情况下，它会在`WEB-INF`文件夹下查找`faces-config.xml`。通过将`org.springframework.web.jsf.el.SpringBeanFacesELResolver`设置为`ELResolver`，我们可以将 Spring POJO 作为 JSF bean 访问。通过注册`DelegatingPhaseListenerMulticaster`，任何实现`PhaseListener`接口的 Spring bean，JSF 的阶段事件将被广播到 Spring bean 中实现的相应方法。

这是`faces-config.xml`文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<faces-config 

xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee/web-facesconfig_2_2.xsd"
version="2.2">
  <application>
    <el-resolver>org.springframework.web.jsf.el.SpringBeanFacesELResolver</el-resolver>
  </application>
  <lifecycle>
    <phase-listener>org.springframework.web.jsf.DelegatingPhaseListenerMulticaster</phase-listener>
  </lifecycle>
</faces-config>
```

在 JSF 中，我们可以使用会话、请求或应用程序范围来定义 bean，并且 bean 的值在特定范围内保留。将`eager`标志设置为`false`意味着延迟初始化，这会在第一次请求到达时创建 bean，而`true`意味着在启动时创建 bean。`OrderBean`类的代码如下：

```java
@ManagedBean(name = "orderBean", eager = true)
@RequestScoped
@Component
public class OrderBean {
  private String orderName;
  private Integer orderId;

  @Autowired
  public OrderServiceorder Service;
  public String placeAnOrder(){
    orderName=orderService.placeAnOrder(orderId);
    return "confirmation";
  }

  public String getOrderName() {
    return orderName;
  }
  public void setOrderName(String orderName) {
    this.orderName = orderName;
  }
  public Integer getOrderId() {
    return orderId;
  }
  public void setOrderId(Integer orderId) {
    this.orderId = orderId;
  }

}
```

此外，这些 bean 在表示层中可用于与后端交互。在第一个屏幕（`order.xhtml`）上，我们调用 bean 的方法（`placeAnOrder`）：

```java
<html lang="en"

>
  <h:body>
  <h3>input: JSF 2 and Spring Integration</h3>
    <h:form id="orderForm">
      <h:outputLabel value="Enter order id:" />
      <h:inputText value="#{orderBean.orderId}" /> <br/>
      <h:commandButton value="Submit" action="#{orderBean.placeAnOrder}"/>
    </h:form>
  </h:body>
</html>
```

该方法返回一个字符串作为确认，并在`action`属性中指定导航，意味着下一页是`confirmation.xhtml`，如下所示：

```java
<html lang="en"

>
  <h:body>
  <h3>Confirmation of an order</h3>
  Product Name: #{orderBean.orderName}
  </h:body>
</html>
```

# Spring 的 Struts 集成

Spring MVC 依赖于`DispatcherServlet`，它将请求发送到可配置的映射处理程序和视图和主题解析的控制器。在 Struts 中，控制器的名称是`Action`。在 Struts 2 中，为了解决线程安全问题，将为每个请求实例化`Action`实例，而 Spring MVC 只创建一次控制器，每个控制器实例为所有请求提供服务。

要启用 Spring 与 Struts 2 的集成，Struts 提供了`struts2-spring-plugin`。在 Struts 2.1 中，Struts 引入了约定插件（`struts2-convention-plugin`），简化了通过注释创建`Action`类（无需任何配置文件`struts.xml`）。该插件期望一组命名约定，用于`Action`类、包和视图命名，将在本节中解释。

要将 Struts 2 与 Spring 集成，您需要添加这些依赖项：

```java
<dependency>
  <groupId>org.apache.struts</groupId>
  <artifactId>struts2-core</artifactId>
  <version>2.3.20</version>
</dependency>
<dependency>
  <groupId>org.apache.struts</groupId>
  <artifactId>struts2-spring-plugin</artifactId>
  <version>2.3.20</version>
</dependency>
<dependency>
  <groupId>org.apache.struts</groupId>
  <artifactId>struts2-convention-plugin</artifactId>
  <version>2.3.20</version>
</dependency>
```

`struts2-convention-plugin`插件搜索包含字符串“struts”、“struts2”、“action”或“actions”的包，并检测`Action`类，其名称以`Action`(`*Action`)结尾，或者实现`com.opensymphony.xwork2.Action`接口（或扩展其子类`com.opensymphony.xwork2.ActionSupport`）。`ViewOrderAction`类的代码如下：

```java
package com.springessentialsbook.chapter7.struts;
...
@Action("/order")
@ResultPath("/WEB-INF/pages")
@Result(name = "success", location = "orderEntryForm.jsp")
public class ViewOrderAction extends ActionSupport {
  @Override
  public String execute() throws Exception {
    return super.execute();
  }
}
```

`@Action`将`/order`（在请求 URL 中）映射到此操作类，`@ResultPath`指定视图（JSP 文件）的存在位置。`@Result`指定导航到`execute()`方法的字符串值的下一页。我们创建了`ViewOrderAction`，以便能够导航到新页面，并在视图（`orderEntryForm.jsp`）中提交表单时执行操作（业务逻辑）：

```java
package com.springessentialsbook.chapter7.struts;
…...
@Action("/doOrder")
@ResultPath("/WEB-INF/pages")
@Results({
  @Result(name = "success", location = "orderProceed.jsp"),
  @Result(name = "error", location = "failedOrder.jsp")
})
public class DoOrderAction extends ActionSupport {
  @Autowired
  private OrderService orderService;
  private OrderVO order;

  public void setOrder(OrderVO order) {
    this.order = order;
  }

  public OrderVO getOrder() {
    return order;
  }

  @Override
  public String execute( ) throws Exception {
    if ( orderService.isValidOrder(order.getOrderId())) {
      order.setOrderName(orderService.placeAnOrder(order.getOrderId()));
      return SUCCESS;
    }
    return ERROR;
  }
```

此外，这是调用`Action`类的 JSP 代码。请注意表单的`doOrder`操作，它调用`DoOrderAction`类（使用`@Action("doOrder")`）。

```java
<%@ page language="java" contentType="text/html; charset=UTF-8"
pageEncoding="UTF-8"%>
<%@ taglib prefix="s" uri="/struts-tags" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" 
"http://www.w3.org/TR/html4/loose.dtd">
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  </head>
  <body>
    <div align="center">
      <h1>Spring and Struts Integration</h1>
      <h2>Place an order</h2>
      <s:form action="doOrder" method="post">
        <s:textfield label="OrderId" name="order.orderId" />
        <s:submit value="Order" />
      </s:form>
    </div>
  </body>
</html>
```

正如您所看到的，我们在视图中使用了`OrderVO`，其代码如下，作为数据模型。对 JSP 代码或操作类中此对象的任何更改都将传递到下一页：

```java
public class OrderVO {
  private String orderName;
  private String orderId;

  public String getOrderName() {
    return orderName;
  }
  public void setOrderName(String orderName) {
    this.orderName = orderName;
  }
  public String getOrderId() {
    return orderId;
  }
  public void setOrderId(String orderId) {
    this.orderId = orderId;
  }
```

在`DoOrderAction`操作类中，在方法执行中，我们实现业务逻辑，并返回在表示层中导航逻辑中指定方法的字符串值。在这里，操作类要么转到`orderProceed.jsp`（如果是有效订单），要么转到`failedOrder.jsp`（如果失败）。这是`orderProceed.jsp`页面，将转发成功订单：

```java
<%@ taglib prefix="s" uri="/struts-tags" %>
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  </head>
  <body>
    <div align="center">
      <h1>Order confirmation</h1>
      <s:label label="OrderId" name="order.orderId" />, <s:label label="OrderName" name="order.orderName" /> <br/>
      has been successfully placed.
    </div>
  </body>
</html>
```

# 摘要

在本章中，我们解释了如何将 Spring 与两种著名的演示技术集成：JSF 和 Struts。

您可以在此处获取有关 Spring 与 Web 框架集成的更多信息：

[`docs.spring.io/spring/docs/current/spring-framework-reference/html/web-integration.html`](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/web-integration.html)

要了解有关 Spring 的 Struts 插件的更多信息，请访问此链接：

[`struts.apache.org/docs/spring-plugin.html`](http://struts.apache.org/docs/spring-plugin.html)

您可以在此处获取有关 Struts 约定插件中命名约定的更多详细信息：

[`struts.apache.org/docs/convention-plugin.html`](https://struts.apache.org/docs/convention-plugin.html)

如今，大公司正在向单页面应用程序在表示层转变。要了解这个话题，请阅读第六章，“构建单页面 Spring 应用程序”。
