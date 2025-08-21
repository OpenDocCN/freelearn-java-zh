## 第一章。Spring 概览

*Spring，传统 J2EE 的冬天之后的新起点*，这就是 Spring 框架的真正含义。是处理 Java 企业应用程序中众多复杂模块相互协作开发的最问题解决方案。Spring 不是传统 Java 开发的替代品，而是公司应对当今竞争激烈、快速发展的市场的可靠解决方案，同时不让开发者紧密依赖于 Spring API。

在本主题中，我们将涉及以下几点：

+   Spring 框架简介

+   Spring 在企业应用程序开发中解决的问题

+   Spring 路线图

+   Spring 5.0 的新特性

## spring 框架简介

引言

罗德·约翰逊是澳大利亚计算机专家，也是 SpringSource 的联合创始人。《J2EE 设计和发展一对一专家》于 2002 年 11 月由他出版。这本书包含大约 30000 行代码，其中包括框架的基本概念，如**控制反转**（**IoC**）和**依赖注入**（**DI**）。这部分代码被称为 interface21。他写这部分代码的初衷只是为了方便开发者工作，或者作为他们自己开发的基石。他从未想过开发任何框架或类似的东西。在 Wrox 论坛上，关于代码、其改进和其他许多事情进行了长时间的讨论。论坛上的两位读者尤尔根·霍勒和扬·卡罗夫提出了将代码作为新框架基础的想法。这是扬的理由，*Spring，传统 J2EE 的冬天之后的新起点*，他将框架命名为 Spring 框架。该项目于 2003 年 6 月公开，并朝着 1.0 版本迈进。此后，为了支持市场上的技术，发生了许多变化和升级。本书关注的是最新版本 5.0。在接下来的几页中，我们将介绍这个版本中添加的新特性。在随后的页面中，我们将介绍如何将最新特性应用于您的应用程序，以及作为开发人员如何充分利用这些特性。

## Spring 解决的问题

引言

Java 平台是一个长期、复杂、可扩展、积极进取并快速发展的平台。应用程序开发在特定的版本上进行。为了保持最新的标准和与之相适应，这些应用程序需要不断升级到最新版本。这些应用程序有大量的类相互交互，复用 API 以充分发挥其优势，使应用程序运行顺畅。但这导致了 AS 的许多常见问题。

### 可扩展性

市场上的每项技术，无论是硬件还是软件，其增长和发展的速度都非常快。几年前开发的应用程序可能会因为这些领域的增长而变得过时。市场的要求如此之高，以至于开发者需要不断更新应用程序。这意味着我们今天开发的任何应用程序都应能在不影响现有运行的情况下，处理未来的需求和增长。应用程序的可扩展性是处理或支持工作负载的增加，以适应不断增长的环境，而不是替换它们。应用程序能够支持因用户数量增加而增加的网站流量，这是一个非常简单的例子，说明应用程序是可扩展的。由于代码紧密耦合，使其可扩展成为一个问题。

### 管道代码

让我们以在 Tomcat 环境中配置 DataSource 为例。现在开发者想要在应用程序中使用这个配置的 DataSource。我们会做什么？是的，我们会进行 JNDI 查找以获取 DataSource。为了处理 JDBC，我们将获取资源并在`try catch`中释放。像我们在这里讨论的`try catch`，计算机间的通信，集合等都是必要的，但不是特定于应用程序的，这些是管道代码。管道代码增加了代码的长度，并使调试变得复杂。

### 样板代码

我们在进行 JDBC 时如何获取连接？我们需要注册 Driver 类，并在 DriverManager 上调用`getConnection()`方法以获取连接对象。这些步骤有其他替代方案吗？实际上没有！无论何时，无论在哪里进行 JDBC，这些相同的步骤每次都必须重复。这种重复的代码，开发者在不同地方编写的代码块，少量或没有修改以实现某些任务，称为样板代码。样板代码使 Java 开发变得不必要地更长和更复杂。

### 无法避免的非功能性代码

无论何时进行应用程序开发，开发者都会专注于业务逻辑、外观和要实现的数据持久性。但除了这些事情，开发者还会深入思考如何管理事务，如何处理网站上的增加负载，如何使应用程序安全等问题。如果我们仔细观察，这些事情并不是应用程序的核心关注点，但它们却是无法避免的。这种不处理业务逻辑（功能性）需求，但对维护、故障排除、应用程序安全等重要的代码称为非功能性代码。在大多数 Java 应用程序中，开发者经常不得不编写非功能性代码。这导致对业务逻辑开发产生了偏见。

### 单元测试应用程序

让我们来看一个例子。我们希望测试一段将数据保存到数据库中的代码。这里测试数据库不是我们的目的，我们只是想确定我们编写的代码是否正常工作。企业级 Java 应用程序由许多相互依赖的类组成。由于对象之间存在依赖，因此进行测试变得困难。

Spring 主要解决了这些问题，并提供了一个非常强大而又简单的解决方案，

### 基于 POJO 的开发

类是应用程序开发的基本结构。如果类被扩展或实现了框架的接口，由于它们与 API 紧密耦合，因此复用变得困难。**普通老式 Java 对象**（**POJO**）在 Java 应用程序开发中非常著名且经常使用。与 Struts 和 EJB 不同，Spring 不会强制开发者编写导入或扩展 Spring API 的代码。Spring 最好的地方在于，开发者可以编写通常不依赖于框架的代码，为此，POJO 是首选。POJO 支持松耦合的模块，这些模块可复用且易于测试。

### 注意

Spring 框架之所以被称为非侵入性，是因为它不会强制开发者使用 API 类或接口，并允许开发松耦合的应用程序。

### 通过依赖注入实现松耦合

耦合度是类与类之间知识的关联程度。当一个类对其他类的设计依赖性较低时，这个类就可以被称为松耦合。松耦合最好通过**接口编程**来实现。在 Spring 框架中，我们可以将类的依赖关系在与代码分离的配置文件中维护。利用 Spring 提供的接口和依赖注入技术，开发者可以编写松耦合的代码（别担心，我们很快就会讨论依赖注入以及如何实现它）。借助松耦合，开发者可以编写出因依赖变化而需要频繁变动的代码。这使得应用程序更加灵活和易于维护。

### 声明式编程

在声明式编程中，代码声明了将要执行什么，而不是如何执行。这与命令式编程完全相反，在命令式编程中，我们需要逐步说明我们将执行什么。声明式编程可以通过 XML 和注解来实现。Spring 框架将所有配置保存在 XML 中，框架可以从中维护 bean 的生命周期。由于 Spring 框架中的开发，从 2.0 版本开始提供了 XML 配置的替代方案，即使用广泛的注解。

### 使用方面和模板减少样板代码

我们刚刚在前几页讨论过，重复的代码是样板代码。样板代码是必要的，如果没有它，提供事务、安全、日志等功能将变得困难。框架提供了解决编写处理此类交叉关注点的 Aspect 的方案，无需将其与业务逻辑代码一起编写。使用 Aspect 有助于减少样板代码，但开发者仍然可以实现相同的效果。框架提供的另一个功能是不同需求的模板。JDBCTemplate、HibernateTemplate 是 Spring 提供的另一个有用的概念，它减少了样板代码。但事实上，你需要等待理解并发现其真正的潜力。

### **分层架构**

与分别提供 Web 持久性解决方案的 Struts 和 Hibernate 不同，Spring 有一系列广泛的模块解决多种企业开发问题。这种分层架构帮助开发者选择一个或多个模块，以一种连贯的方式为他的应用程序编写解决方案。例如，即使不知道框架中有许多其他模块，开发者也可以选择 Web MVC 模块高效处理 Web 请求。

## **Spring 架构**

****

Spring 提供了超过 20 个不同的模块，可以大致归纳为 7 个主要模块，如下所示：

![](img/image_01_001.png)

**Spring 模块**

### **核心模块**

#### **核心**

**Spring 核心模块**支持创建 Spring bean 的方法以及向 bean 中注入依赖。它提供了配置 bean 以及如何使用 BeanFactory 和 ApplicationContext 从 Spring 容器获取配置 bean 的方法，用于开发独立应用程序。

#### **Beans**

**Beans 模块**提供了**BeanFactory**，为编程单例提供了替代方案。BeanFactory 是工厂设计模式的实现。

#### **上下文**

这个模块支持诸如 EJB、JMX 和基本远程调用的 Java 企业特性。它支持缓存、Java 邮件和模板引擎（如 Velocity）的第三方库集成。

#### **SpEL**

**Spring 表达式语言（SpEL）**是**统一表达式语言**的扩展，该语言已在 JSP 2.1 规范中指定。SpEL 模块支持设置和获取属性值，使用逻辑和算术运算符配置集合，以及从 Spring IoC 中获取命名变量。

### **数据访问与集成模块**

#### **JDBC（DAO）**

这个模块在 JDBC 之上提供了抽象层。它支持减少通过加载驱动器获取连接对象、获取语句对象等产生的样板代码。它还支持模板，如 JdbcTemplate、HibernateTemplate，以简化开发。

#### **ORM**

对象关系映射（ORM）模块支持与非常流行的框架（如 Hibernate、iBATIS、Java 持久性 API（JPA）、Java 数据对象（JDO））的集成。

#### **OXM**

对象 XML 映射器（OXM）模块支持对象到 XML 的映射和集成，适用于 JAXB、Castor、XStream 等。

#### JMS

此模块提供支持，并为通过消息进行异步集成的 Java 消息服务（JMS）提供 Spring 抽象层。

#### 事务

JDBC 和 ORM 模块处理 Java 应用程序与数据库之间的数据交换。此模块在处理 ORM 和 JDBC 模块时支持事务管理。

### Web MVC 和远程模块

#### Web

此模块支持与其他框架创建的 web 应用程序的集成。使用此模块，开发人员还可以通过 Servlet 监听器开发 web 应用程序。它支持多部分文件上传和请求与响应的处理。它还提供了与 web 相关的远程支持。

#### Servlet

此模块包含 Spring 模型视图控制器（MVC）实现，用于 web 应用程序。使用 Spring MVC，开发人员可以编写处理请求和响应的代码，以开发功能齐全的 web 应用程序。它通过支持处理表单提交来摆脱处理请求和响应的样板代码。

#### 门户

门户模块提供了 MVC 实现，用于支持 Java 门户 API 的门户环境。

### 注意

门户在 Spring 5.0M1 中被移除。如果您想使用门户，则需要使用 4.3 模块。

#### WebSocket

WebSocket 是一种协议，提供客户端与服务器之间的双向通信，已在 Spring 4 中包含。此模块为应用程序提供对 Java WebSocket API 的集成支持。

### 注意

**`Struts 模块`** 此模块包含支持将 Struts 框架集成到 Spring 应用程序中的内容。但在 Spring 3.0 中已弃用。

### AOP 模块

#### AOP

面向方面编程（AOP）模块有助于处理和管理应用程序中的交叉关注点服务，有助于保持代码的清洁。

#### 方面

此模块为 AspectJ 提供了集成支持。

### 仪器模块

#### 仪器

Java 仪器提供了一种创新的方法，通过类加载器帮助从 JVM 访问类并修改其字节码，通过插入自定义代码。此模块支持某些应用服务器的仪器和类加载器实现。

#### 仪器 Tomcat

仪器 Tomcat 模块包含对 Tomcat 的 Spring 仪器支持。

#### 消息传递

消息传递模块提供了对 STOMP 作为 WebSocket 协议的支持。它还有用于路由和处理从客户端接收的 STOMP 消息的注解。

+   Spring 4 中包含了消息传递模块。

### 测试模块

测试模块支持单元以及集成测试，适用于 JUnit 和 TestNG。它还提供创建模拟对象的支持，以简化在隔离环境中进行的测试。

## Spring 还支持哪些底层技术？

***

### 安全模块

如今，仅具有基本功能的应用程序也需要提供良好的多层次安全处理方式。Spring5 支持使用 Spring AOP 的声明式安全机制。

### 批量处理模块

Java 企业应用需要执行批量处理，在没有用户交互的情况下处理大量数据，这在许多商业解决方案中是必需的。以批处理方式处理这些问题是可用的最佳解决方案。Spring 提供了批量处理集成，以开发健壮的应用程序。

### Spring Integration

在企业应用的开发中，应用可能需要与它们进行交互。Spring Integration 是 Spring 核心框架的扩展，通过声明式适配器提供与其他企业应用的集成。消息传递是此类集成中广泛支持的一项。

### 移动模块

广泛使用移动设备为开发打开了新的大门。这个模块是 Spring MVC 的扩展，有助于开发称为 Spring Android Project 的手机网络应用。它还提供检测发起请求的设备类型，并相应地呈现视图。

### LDAP 模块

Spring 的初衷是简化开发并减少 boilerplate 代码。Spring LDAP 模块支持使用模板开发进行简单的 LDAP 集成。

### .NEW 模块

引入了新的模块来支持.NET 平台。ADO.NET、NHibernate、ASP.NET 等模块包含在.NET 模块中，以简化.NET 开发，利用 DI、AOP、松耦合等特性。

## Spring 路线图

* * *

### 1.0 2004 年 3 月

它支持 JDO1.0 和 iBATIS 1.3，并与 Spring 事务管理集成。这个版本支持的功能有，Spring Core、Spring Context、Spring AOP、Spring DAO、Spring ORM 和 Spring web。

### 2.0 2006 年 10 月

Spring 框架增强了 Java5 的支持。它添加了开箱即用的命名空间，如 jee、tx、aop、lang、util，以简化配置。IoC 支持单例和原型等作用域。除了这些作用域，还引入了 HttpSession、集群缓存和请求的作用域。引入了基于注解的配置，如@Transactional、@Required、@PersistenceContext。

### 2.5 2007 年 11 月

在这个版本中，Spring 支持完整的 Java 6 和 Java EE 5 特性，包括 JDBC 4、JavMail 1.4、JTA 1.1、JAX WS 2.0。它还扩展了对基于注解的依赖注入的支持，包括对限定符的支持。引入了一个名为 AspectJ 切点表达式中的 pointcut 元素的新 bean。提供了基于 LoadTimeWeaver 抽象的 AspectJ 加载时间编织的内置支持。为了方便，包含了像 context、jms 这样的自定义命名空间的介绍。测试支持扩展到了 Junit4 和 TestNG。添加了基于注解的 Spring MVC 控制器。它还支持在类路径上自动检测组件，如@Repository、@Service、@Controller 和@Conponent。现在 SimpleJdbcTemplate 支持命名 SQL 参数。包含了认证的 WebSphere 支持。还包括了对 JSR-250 注解的支持，如@Resource、PostConstruct、@PreDestroy。

### 3.0 GA 2009 年 12 月

整个代码已修订以支持 Java 5 特性，如泛型、可变参数。引入了 Spring 表达式语言(SpEL)。还支持 REST web 应用程序的注解。扩展了对许多 Java EE 6 特性的支持，如 JPA 2.0、JSF 2.0。版本 3.0.5 还支持 hibernate 3.6 最终版。

### 3.1GA 2011 年 12 月

在这个版本中，测试支持已升级到 Junit 4.9。还支持在 WebSphere 版本 7 和 8 上进行加载时间编织。

### 4.0 2013 年 12 月

首次全面支持 Java 8 特性。这个版本使用 Java EE 6 作为其基础。使用 Spring 4，现在可以定义使用 Groovy DSL 的外部 bean 配置。开发者现在可以将泛型类型作为一种限定符形式。@Lazy 注解可以用于注入点以及@Bean 定义。为了开发者使用基于 Java 的配置，引入了@Description 注解。引入了@Conditional 注解进行条件过滤。现在，不再需要默认构造函数供 CGLIB 基于的代理类使用。引入了@RestController 以去除对每个@RequestMapping 的@ResponseBody 的需求。包括了 AsynchRestTemplate，它允许非阻塞异步支持 REST 客户端开发。作为新的模型引入了 spring-websocket，以提供对基于 WebSocket 的服务器和客户端之间的双向通信的支持。引入了 spring-messaging 模块以支持 WebSocket 子协议 STOMP。现在可以作为元注解使用大部分来自 spring-test 模块的注解来创建自定义组合注解。org.springframework.mock.web 包中的 mocks 基于 Servlet API 3.0。

### 5.0 M1 Q4 2016

Spring 5M1 将支持 Java8+，但基本上，它旨在跟踪并对 Java9 的新特性提供大量支持。它还将支持响应式编程 Spring 5 将专注于 HTT2.0。它还通过响应式架构关注响应式编程。spring-aspects 中的 mock.staticmock 和 web.view.tiles2 已经被移除。不再支持 Portlet、Velocity、JasperReports、XMLBeans、JDO、Guava。

可以总结如下：

![](img/image_01_002.png)

Spring 模块

## 容器-Spring 的心脏

* * *

POJO 开发是 Spring 框架的基石。在 Spring 中配置的 POJO，其对象的实例化、对象组装、对象管理都是由 Spring IoC 容器完成的，称为 bean 或 Spring bean。我们使用 Spring IoC，因为它遵循控制反转的模式。

### 控制反转（IoC）

在每一个 Java 应用程序中，每位开发者做的第一件重要的事情就是获取一个可以在应用程序中使用的对象。一个对象的状况可以在运行时获得，也可能在编译时获得。但是开发者通常在多次使用样板代码时创建对象。当同一个开发者使用 Spring 而不是亲自创建对象时，他将依赖于框架来获取对象。控制反转（IoC）这个术语是因为 Spring 容器将对象创建的责任从开发者那里反转过来。

Spring IoC 容器只是一个术语，Spring 框架提供了两个容器

+   BeanFactory

+   应用程序上下文（ApplicationContext）

### BeanFactory 的历史

BeanFactory 容器提供了基本功能和框架配置。现在，开发者不会倾向于使用 BeanFactory。那么，为什么 BeanFactory 仍然在框架中呢？为什么没有被移除呢？如果没有 BeanFactory，那么替代品是什么？让我们逐一回答这些问题。BeanFactory 在框架中的一个非常简单的答案是为了支持 JDK1.4 的向后兼容性。BeanFactory 提供了 BeanFactoryAware、InitializingBean、DisposableBean 接口，以支持与 Spring 集成的第三方框架的向后兼容性。

#### XMLBeanFactory

当今的企业应用程序开发需求远超过普通开发。开发者将很高兴能得到帮助来管理对象生命周期、注入依赖项或减少 IoC 容器的样板代码。XMLBeanFactory 是 BeanFactory 的常见实现。

让我们实际找出 BeanFactory 容器是如何初始化的：

1.  创建一个名为`Ch01_Container_Initialization`的 Java 应用程序。

1.  按照以下快照添加 jar 包：

![](img/image_01_003.png)

需要添加的 jar 包

### 注意

确保你使用的是 JRE 1.8，因为它 是 Spring5.0.0.M1 的基础。你可以从...............下载 jar 包。

1.  在`com.ch01.test`包下创建一个名为`TestBeanFactory`的类。

1.  在类路径中创建一个名为`beans_classpath.xml`的 XML 文件，我们可以在以后编写 bean 定义。每个 bean 定义文件都包含对特定 Spring 版本的 beans.xsd 的引用。这个 XML 文件的根标签将是`<beans>`。

XML 文件的基本结构如下：

```java
      <?xml version="1.0" encoding="UTF-8"?> 
        <beans xmlns="http://www.springframework.org/schema/beans" 
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
          xsi:schemaLocation="http://www.springframework.org/schema/beans 
          http://www.springframework.org/schema/beans/spring-beans.xsd"> 
        </beans> 

```

我们的 XML 文件包含上述相同的代码，没有配置任何 bean。

1.  在 main 函数中，让我们写下初始化 bean 工厂的代码，如下所示：

```java
      BeanFactory beanFactory=new XmlBeanFactory( 
        new ClassPathResource("beans_classpath.xml")); 

```

在这里，`bean_classpath.xml`将包含 bean 定义（为了简单起见，我们没有添加任何 bean 定义，我们将在下一章详细介绍）。`ClassPathResource`从类路径加载资源。

1.  有时资源不会在类路径中，而是在文件系统中。以下代码可用于从文件系统加载资源：

```java
      BeanFactory beanFactory=new XmlBeanFactory( 
        new FileSystemResource("d:\\beans_fileSystem.xml")); 

```

1.  我们需要在 D 驱动器上创建`bean_fileSystem.xml`，它将包含与`bean_classpath.xml`相同的内容。完整代码如下：

```java
      public class TestBeanFactory { 
        public static void main(String[] args) { 
          // TODO Auto-generated method stub 
          BeanFactory beanFactory=new XmlBeanFactory(new
            ClassPathResource("beans_classpath.xml"));
          BeanFactory beanFactory1=new XmlBeanFactory(new
            FileSystemResource("d:\\beans_fileSystem.xml")); 
          System.out.println("beanfactory created successfully"); 
        } 
      } 

```

由于我们在这里没有编写任何输出代码，除了 Spring 容器的日志信息外，控制台上不会有任何输出。但以下快照显示了 XML 文件加载并初始化了容器：

![](img/image_01_004.png)

控制台日志输出

### 注意

BeanFactory 不支持多个配置文件。

### 应用程序上下文：现状

注册 BeanProcessor 和 BeanFactoryPostProcessor 在 AOP 和属性占位符中扮演重要角色，需要显式编写代码，这使得与其配合变得不方便。开发者不想编写支持国际化的代码。处理 AOP 集成的事件发布是不可避免的。Web 应用程序需要具有特定于应用层的内容。对于这些问题，简单的解决方案是扩展 BeanFactory 提供的服务，以支持 ApplicationContext。ApplicationContext 不是 BeanFactory 的替代品，而是针对企业特定解决方案的扩展，以及为 bean 配置提供更多高级机制。

让我们来看看实现。

#### ClassPathXmlApplicationContext

对于独立应用程序，使用 AbstractXmlApplicationContext 的子类。它使用类路径中的配置的 bean XML 文件。如果有多个 XML 配置文件，后来的 bean 定义将覆盖先前的 bean 定义。它提供了编写新 bean 定义以替换先前的定义的优势。

让我们实际找出`ClassPathXmlApplicationContext`容器是如何初始化的。我们将使用相同的`Ch01_Container_Initialization`项目，按照以下步骤进行：

1.  在`com.ch01.test`包下创建一个名为`TestClasspathApplicationContext`的类。

1.  在新的应用程序中，像以前一样在类路径中创建一个名为`beans_classpath.xml`的 XML 文件。

1.  在 main 函数中，让我们写下初始化 bean 工厂的代码，如下所示：

```java
      try { 
        ApplicationContext context=new
          ClassPathXmlApplicationContext("beans_classpath.xml"); 
        System.out.println("container created successfully"); 
      } 
      catch (BeansException e) { 
        // TODO Auto-generated catch block 
        e.printStackTrace(); 
      } 

```

不需要创建 XML 文件，因为我们已经在之前的示例中创建了它。`ClassPathXmlApplicationContext`从类路径加载`bean_classpath.xml`文件，其中包含 bean 定义（为了简单起见，我们没有添加任何 bean 定义，我们将在下一章详细介绍）。

1.  运行应用程序，输出以下内容，表明容器成功创建：

![](img/image_01_005.png)

控制台输出

1.  在 Java 企业应用程序中，项目可以有多个配置文件，因为这样可以容易地维护和支持模块化。要加载多个 bean 配置文件，我们可以使用以下代码：

```java
      try { 
        ApplicationContext context1 = new 
          ClassPathXmlApplicationContext 
            (new String[]
            {"beans_classpath.xml","beans_classpath1.xml" }); 
       }  
       catch (BeansException e) { 
         // TODO Auto-generated catch block 
         e.printStackTrace(); 
       } 

```

要使用前面的代码行，我们需要在类路径中创建`beans_classpath1.xml`。

#### FileSystemXmlApplicationContext

与 ClassPathXmlApplicationContext 类似，这个类也扩展了 AbstractXmlApplicationContext，用于独立应用程序。但这个类有助于从文件系统加载 bean 的 XML 定义。文件路径相对于当前工作目录。如果指定绝对文件路径，可以使用`file:`作为前缀。它还提供了在有多个 XML 配置的情况下，编写新的 bean 定义以替换之前的定义的优势。

让我们实际找出`ClassPathXmlApplicationContext`容器是如何初始化的。我们将按照以下步骤使用相同的`Ch01_Container_Initialization`项目：

1.  在`com.ch01.test`包下创建一个名为`TestFileSystemApplicationContext`的类。

1.  在之前应用程序中创建的 D 驱动器上创建一个新的 XML 文件`beans_fileSystem.xml`。

1.  在主函数中，让我们写下以下代码来初始化 bean 工厂：

```java
      try { 
        ApplicationContext context=new   
          FileSystemXmlApplicationContext 
          ("d:\\beans_fileSystem.xml"); 
        System.out.println("container created successfully"); 
      } 
      catch (BeansException e) { 
        // TODO Auto-generated catch block 
        e.printStackTrace(); 
      } 

```

`FileSystemXmlApplicationContext`从指定路径加载`bean_fileSystem.xml`文件。

1.  运行应用程序，将给出以下输出，表明容器已成功创建。

上述讨论的项目结构将如下所示：

![](img/image_01_006.png)

项目目录结构

#### WebXmlApplicationContext

`WebXmlApplicationContext`继承了`AbstractRefreshableWebApplicationContext`。我们可以在`applicationContext.xml`中编写与根应用程序上下文相关的上下文定义，并将其放在 WEB-INF 下，因为这是默认的位置，从这里加载上下文定义。`XXX-servlet.xml`文件用于加载控制器定义，正如在 MVC 应用程序中所示。此外，我们可以通过为`context-param`和`init-param`配置`contextConfigLocation`来覆盖默认位置。

## 容器中如何获取 bean？

* * *

是的，如果不从开发方面做任何事情，豆子或豆子对象将无法获得。Spring 管理豆子，但是必须决定要管理什么，并将其传递给容器。Spring 通过 XML 文件配置支持声明式编程。在 XML 文件中配置的豆子定义由容器加载，并使用 org.springframework.beans 在容器中实例化对象和注入属性值。豆子生命周期解释了每个豆子对象从使对象可供应用程序使用直至应用程序不再需要时将其清理并从容器中移除的各个阶段、阶段或活动。我们将在下一章讨论详细的初始化过程。

## 摘要

* * *

本章概述了 Spring 框架。我们讨论了在 Java 企业应用程序开发中遇到的通用问题以及 Spring 框架是如何解决它们的。我们看到了自 Spring 首次进入市场以来每个版本中发生的重大变化。Spring 框架的核心是豆子。我们使用 Spring 来简化容器管理它们的工作。我们详细讨论了两个 Spring 容器 BeanFactory 和 ApplicationContext，以及开发人员如何使用它们。容器参与了豆子生命周期管理的过程。在下一章，我们旨在深入讨论关于豆子状态管理以及一个非常著名的术语依赖注入和豆子生命周期管理。
