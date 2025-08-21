## 第四章。面向切面编程

上一章关于 Spring DAO 的内容为我们提供了很好的实践，了解了 Spring 如何通过松耦合方式处理 JDBC API。但是，我们既没有讨论 JDBC 事务，也没有探讨 Spring 如何处理事务。如果你已经处理过事务，你了解其步骤，而且更加清楚这些步骤是重复的，并且分散在代码各处。一方面，我们提倡使用 Spring 来避免代码重复，另一方面，我们却在编写这样的代码。Java 强调编写高内聚的模块。但是在我们的代码中编写事务管理将不允许我们编写内聚的模块。此外，编写代码的目的并非是为了事务。它只是提供支持，以确保应用程序的业务逻辑不会产生任何不期望的效果。我们还没有讨论过如何处理这种支持功能以及应用程序开发的主要目的。除了事务之外，还有哪些功能支持应用程序的工作？本章将帮助我们编写没有代码重复的高度内聚的模块，以处理这些支持功能。在本章中，我们将讨论以下几点：

+   交叉技术是什么？

+   交叉技术在应用程序开发中扮演什么角色？

+   我们将讨论关于面向切面编程（AOP）以及 AOP 在处理交叉技术中的重要作用。

+   我们将深入探讨 AOP 中的方面、建议和切点是什么。

软件应用程序为客户的问题提供了一个可靠的解决方案。尽管我们说它是可靠的，但总是有可能出现一些运行时问题。因此，在开发过程中，软件的维护同样重要。每当应用程序中出现问题时，客户都会回来找开发者寻求解决方案。除非客户能够准确地说明问题的原因，否则开发者是无能为力的。为了防止问题的再次发生，开发者必须重新创建相同的情况。在企业应用程序中，由于模块数量众多，重新创建相同的问题变得复杂。如果有一个人能够持续跟踪用户在做什么，那就太好了。这个跟踪器的跟踪帮助开发者了解出了什么问题，以及如何轻松地重新创建它。是的，我在谈论日志记录机制。

让我们考虑另一个非常常见的铁路票务预订情况。在票务预订时，我们从图表中选择可用的座位并继续进行资金转账。有时资金成功转账，票也预订了。但不幸的是，有时由于资金交易时间过长，填写表格延迟或一些服务器端问题可能会导致资金转账失败而无法预订票。资金被扣除而没有发行票。客户将不高兴，而且对于退款来说会更加紧张。这种情况需要借助事务管理谨慎处理，以便如果未发行票，资金应退还到客户账户中。手动操作将是繁琐的任务，而事务管理则优雅地处理了这个问题。

我们可以编写不包含日志记录或事务管理的可运行代码，因为这两者都不属于您的业务逻辑。Java 应用程序的核心是提供一种定制化的、简单的解决方案来解决企业问题。业务逻辑位于中心，提供应用程序的主要功能，有时被称为“主要关注点”。但它还必须支持其他一些功能或服务，这一点不容忽视。这些服务在应用程序中扮演着重要的角色。要么应用程序的迁移会耗时，要么在运行时回溯问题将变得困难。这些关注点大多伴随着重复的代码散布在应用程序中。这些次要关注点被称为“横切关注点”，有时也称为“水平关注点”。日志记录、事务管理和安全机制是开发者在应用程序中使用的横切关注点。

下面的图表展示了横切关注点（如日志记录和事务管理）如何在应用程序代码中散布：

![](img/image_04_001.png)

## 面向切面编程（AOP）

****

与面向对象编程类似，面向切面编程也是一种编程风格，它允许开发者通过将横切关注点与业务逻辑代码分离来编写连贯的代码。AOP 概念是由 Gregor KicZales 及其同事开发的。它提供了编写横切关注点的不同方法或工具。

在 AOP 中处理横切关注点，可以在一个地方编写，从而实现以下好处：

+   减少代码重复以实现编写整洁的代码。

+   有助于编写松耦合的模块。

+   有助于实现高度凝聚的模块。

+   开发者可以专注于编写业务逻辑。

+   在不更改现有代码的情况下，轻松更改或修改代码以添加新功能。

要理解 AOP，我们必须了解以下常见术语，没有它们我们无法想象 AOP。

### 连接点

连接点是应用程序中可以插入方面以执行一些杂项功能的位置，而不会成为实际业务逻辑的一部分。每段代码都有无数的机会，可以被视为连接点。在应用程序中最小的单元类有数据成员、构造函数、设置器和获取器，以及其他功能类。每个都可以是应用方面的机会。Spring 只支持方法作为连接点。

### 切点（Pointcut）

连接点是应用方面的机会，但并非所有机会都被考虑在内。切点是开发者决定应用方面以对横切关注执行特定动作的地方。切点将使用方法名、类名、正则表达式来定义匹配的包、类、方法，在这些地方可以应用方面。

### 建议

在切点处方面所采取的动作称为“建议”（advice）。建议包含为相应的横切关注点执行的代码。如果我们把方法作为连接点，方面可以在方法执行之前或之后应用，也可能是方法有异常处理代码，方面可以插入其中。以下是 Spring 框架中可用的建议。

#### 前置（Before）

前置（Before）建议包含在匹配切点表达式的业务逻辑方法执行之前应用的实现。除非抛出异常，否则将继续执行该方法。可以使用@Before 注解或<aop:before>配置来支持前置建议。

#### 后抛出（After）

在后置（After）建议中，实现将在业务逻辑方法执行之后应用，无论方法执行成功还是抛出异常。可以使用@After 注解或<aop:after>配置来支持后置建议，将其应用于一个方法。

#### 后返回（After returning）

后返回（After Returning）建议的实现仅在业务逻辑方法成功执行后应用。可以使用@AfterReturning 注解或<aop:after-returning>配置来支持后返回建议。后返回建议方法可以使用业务逻辑方法返回的值。

#### 后抛出（After throwing）

后抛出（After Throwning）建议的实现应用于业务逻辑方法抛出异常之后。可以使用@AfterThrowing 注解或<aop:throwing>配置来支持后抛出建议，将其应用于一个方法。

#### 环绕（Around）

环绕通知是所有通知中最重要的一种，也是唯一一种在业务逻辑方法执行前后都会应用的通知。它可用于通过调用 ProceedingJoinPoint 的 proceed()方法来选择是否继续下一个连接点。proceed()通过返回其自身的返回值来帮助选择是否继续到连接点。它可用于开发人员需要执行预处理、后处理或两者的场景。计算方法执行所需时间就是一个这样的场景。可以使用@Around 注解或<aop:around>配置通过将其应用于一个方法来支持环绕通知。

### Aspect（方面）

方面通过切入点表达式和通知来定义机会，以指定动作何时何地被执行。使用@Aspect 注解或<aop:aspect>配置将一个类声明为方面。

### Introduction（介绍）

介绍可以帮助在不需要更改现有代码的情况下，在现有类中声明额外的方法和字段。Spring AOP 允许开发人员向任何被方面通知的类引入新的接口。

### Target object（目标对象）

目标对象是被应用了方面的类的对象。Spring AOP 在运行时创建目标对象的代理。从类中覆盖方法并将通知包含进去以获得所需结果。

### AOP proxy（AOP 代理）

默认情况下，Spring AOP 使用 JDK 的动态代理来获取目标类的代理。使用 CGLIB 进行代理创建也非常常见。目标对象始终使用 Spring AOP 代理机制进行代理。

### Weaving（编织）

我们作为开发者将业务逻辑和方面代码写在两个分开的模块中。然后这两个模块必须合并为一个被代理的目标类。将方面插入业务逻辑代码的过程称为“编织”。编织可以在编译时、加载时或运行时发生。Spring AOP 在运行时进行编织。

让我们通过一个非常简单的例子来理解所讨论的术语。我的儿子喜欢看戏剧。所以我们去看了一场。我们都知道，除非我们有入场券，否则我们不能进入。显然，我们首先需要收集它们。一旦我们有了票，我的儿子把我拉到座位上，兴奋地指给我看。演出开始了。这是一场给孩子们看的有趣戏剧。所有孩子们都在笑笑话，为对话鼓掌，在戏剧场景中感到兴奋。休息时，观众中的大多数人去拿爆米花、小吃和冷饮。每个人都喜欢戏剧，并快乐地从出口离开。现在，我们可能认为我们都知道这些。我们为什么要讨论这个，它与方面有什么关系。我们是不是偏离了讨论的主题？不，我们正在正确的轨道上。再等一会儿，你们所有人也会同意。这里看戏剧是我们的主要任务，让我们说这是我们的业务逻辑或核心关注。购买门票，支付钱，进入剧院，戏剧结束后离开是核心关注的一部分功能。但我们不能安静地坐着，我们对正在发生的事情做出反应？我们鼓掌，笑，有时甚至哭。但这些是主要关注点吗？不！但没有它们，我们无法想象观众看戏剧。这些将是每个观众自发执行的支持功能。正确！！！这些是交叉关注点。观众不会为交叉关注点单独收到指示。这些反应是方面建议的一部分。有些人会在戏剧开始前鼓掌，少数人在戏剧结束后鼓掌，最兴奋的是当他们感到的时候。这只是方面的前置、后置或周围建议。如果观众不喜欢戏剧，他们可能会在中间离开，类似于抛出异常。在非常不幸的日子里，演出可能会被取消，甚至可能在中间停止，需要组织者作为紧急情况介绍。希望现在你知道了这些概念以及它们的实际方法。我们将在演示中简要介绍这些以及更多内容。

在继续演示之前，让我们首先讨论市场上的一些 AOP 框架如下。

#### AspectJ

AspectJ 是一个易于使用和学习的 Java 兼容框架，用于集成跨切实现的交叉。AspectJ 是在 PARC 开发的。如今，由于其简单性，它已成为一个著名的 AOP 框架，同时具有支持组件模块化的强大功能。它可用于对静态或非静态字段、构造函数、私有、公共或受保护的方法应用 AOP。

#### AspectWertz

AspectWertz 是另一个与 Java 兼容的轻量级强大框架。它很容易集成到新旧应用程序中。AspectWertz 支持基于 XML 和注解的方面编写和配置。它支持编译时、加载时和运行时编织。自 AspectJ5 以来，它已被合并到 AspectJ 中。

#### JBoss AOP

JBoss AOP 支持编写方面以及动态代理目标对象。它可以用于静态或非静态字段、构造函数、私有、公共或受保护的方法上使用拦截器。

#### Dynaop

Dynaop 框架是一个基于代理的 AOP 框架。该框架有助于减少依赖性和代码的可重用性。

#### CAESAR

CASER 是一个与 Java 兼容的 AOP 框架。它支持实现抽象组件以及它们的集成。

#### Spring AOP

这是一个与 Java 兼容、易于使用的框架，用于将 AOP 集成到 Spring 框架中。它提供了与 Spring IoC 紧密集成的 AOP 实现，是基于代理的框架，可用于方法执行。

Spring AOP 满足了大部分应用交叉关注点的需求。但以下是一些 Spring AOP 无法应用的限制，

+   Spring AOP 不能应用于字段。

+   我们无法在一个方面上应用任何其他方面。

+   私有和受保护的方法不能被建议。

+   构造函数不能被建议。

Spring 支持 AspectJ 和 Spring AOP 的集成，以减少编码实现交叉关注点。Spring AOP 和 AspectJ 都用于实现交叉技术，但以下几点有助于开发者在实现时做出最佳选择：

+   Spring AOP 基于动态代理，支持方法连接点，但 AspectJ 可以应用于字段、构造函数，甚至是私有、公共或受保护的，支持细粒度的建议。

+   Spring AOP 不能用于调用同一类方法的方法、静态方法或最终方法，但 AspectJ 可以。

+   AspectJ 不需要 Spring 容器来管理组件，而 Spring AOP 只能用于由 Spring 容器管理的组件。

+   Spring AOP 支持基于代理模式的运行时编织，而 AspectJ 支持编译时编织，不需要创建代理。对象的代理将在应用程序请求 bean 时创建一次。

+   由 Spring AOP 编写的方面是基于 Java 的组件，而用 AspectJ 编写的方面是扩展 Java 的语言，所以开发者在使用之前需要学习它。

+   Spring AOP 通过使用@Aspect 注解标注类或简单的配置来实现非常简单。但是，要使用 AspectJ，则需要创建*.aj 文件。

+   Spring AOP 不需要任何特殊的容器，但方面需要使用 AspectJ 编译。

+   AspectJ 是现有应用程序的最佳选择。

    ### 注意

    如果没有 final，静态方法的简单类，则可以直接使用 Spring AOP，否则选择 AspectJ 来编写切面。

让我们深入讨论 Spring AOP 及其实现方式。Spring AOP 可以通过基于 XML 的切面配置或 AspectJ 风格的注解实现。基于 XML 的配置可以分成几个点，使其变得稍微复杂。在 XML 中，我们无法定义命名切点。但由注解编写的切面位于单个模块中，支持编写命名切点。所以，不要浪费时间，让我们开始基于 XML 的切面开发。

### 基于 XML 的切面配置

以下是在开发基于 XML 的切面时需要遵循的步骤，

1.  选择要实现的重叠关注点

1.  编写切面以满足实现重叠关注点的需求。

1.  在 Spring 上下文中注册切面作为 bean。

1.  切面配置写为：

* 在 XML 中添加 AOP 命名空间。

* 添加切面配置，其中将包含切点表达式和建议。

* 注册可以应用切面的 bean。

开发人员需要从可用的连接点中决定跟踪哪些连接点，然后需要使用表达式编写切点以针对它们。为了编写这样的切点，Spring 框架使用 AspectJ 的切点表达式语言。我们可以在表达式中使用以下设计器来编写切点。

#### 使用方法签名

可以使用方法签名从可用连接点定义切点。表达式可以使用以下语法编写：

```java
expression(<scope_of_method>    <return_type><fully_qualified_name_of_class>.*(parameter_list)
```

Java 支持 private，public，protected 和 default 作为方法范围，但 Spring AOP 只支持公共方法，在编写切点表达式时。参数列表用于指定在匹配方法签名时要考虑的数据类型。如果开发人员不想指定参数数量或其数据类型，可以使用两个点(..)。

让我们考虑以下表达式，以深入理解表达式的编写，从而决定哪些连接点将受到建议：

+   `expression(* com.packt.ch04.MyClass.*(..))` - 指定 com.packt.cho3 包内 MyClass 的具有任何签名的所有方法。

+   `expression(public int com.packt.ch04.MyClass.*(..))` - 指定 com.packt.cho3 包内 MyClass 中返回整数值的所有方法。

+   `expression(public int com.packt.ch04.MyClass.*(int,..))` - 指定返回整数及其第一个整数类型参数的 MyClass 中所有方法，该类位于 com.packt.cho3 包内。

+   `expression(* MyClass.*(..))` - 指定所有来自 MyClass 的具有任何签名的方法都将受到建议。这是一个非常特殊的表达式，只能在与建议的类在同一包中使用。

#### 使用类型

类型签名用于匹配具有指定类型的连接点。我们可以使用以下语法来指定类型：

```java
within(type_to_specify) 

```

这里类型将是包或类名。以下是一些可以编写以指定连接点的表达式：

+   `within(com.packt.ch04.*)` - 指定属于 com.packt.ch04 包的所有类的所有方法

+   `within(com.packt.ch04..*)` - 指定属于 com.packt.ch04 包及其子包的所有类的所有方法。我们使用了两个点而不是一个点，以便同时跟踪子包。

+   `within(com.packt.ch04.MyClass)` - 指定属于 com.packt.ch04 包的 MyClass 的所有方法

+   `within(MyInterface+)` - 指定实现 MyInterface 的所有类的所有方法。

#### 使用 Bean 名称

Spring 2.5 及以后的所有版本都支持在表达式中使用 bean 名称来匹配连接点。我们可以使用以下语法：

```java
bean(name_of_bean) 

```

考虑以下示例：

`bean(*Component)` - 这个表达式指定要匹配的连接点属于名称以 Component 结尾的 bean。这个表达式不能与 AspectJ 注解一起使用。

#### 使用 this

'this'用于匹配目标点的 bean 引用是指定类型的实例。当表达式指定类名而不是接口时使用。当 Spring AOP 使用 CGLIB 进行代理创建时使用。

#### 5.sing target

目标用于匹配目标对象是指定类型的接口的连接点。当 Spring AOP 使用基于 JDK 的代理创建时使用。仅当目标对象实现接口时才使用目标。开发者甚至可以配置属性'proxy target class'设置为 true。

让我们考虑以下示例以了解表达式中使用 this 和 target：

```java
package com.packt.ch04; 
Class MyClass implements MyInterface{ 
  // method declaration 
} 

```

我们可以编写表达式来针对方法：

`target( com.packt.ch04.MyInterface)` 或

`this(com.packt.ch04.MyClass)`

#### 用于注解跟踪

开发者可以编写不跟踪方法而是跟踪应用于注解的连接点表达式。让我们以下示例了解如何监控注解。

**使用 with execution:**

execution(@com.packt.ch03.MyAnnotation) - 指定被 MyAnnotation 注解标记的方法或类。

execution(@org.springframework.transaction.annotation.Transactional) - 指定被 Transactional 注解标记的方法或类。

**使用 with @target:**

它用于考虑被特定注解标记的类的连接点。以下示例解释得很清楚，

@target(com.packt.ch03.MyService) - 用于考虑被 MyService 注解标记的连接点。

**使用@args:**

表达式用于指定参数被给定类型注解的连接点。

@args(com.packt.ch04.annotations.MyAnnotation)

上述表达式用于考虑其接受的对象被@Myannotation 注解标记的连接点。

**使用@within:**

表达式用于指定由给定注解指定的类型的连接点。

@within(org.springframework.stereotype.Repository)

上述表达式有助于为被@Repository 标记的连接点提供通知。

**使用@annotation:**

@annotation 用于匹配被相应注解标记的连接点。

@annotation(com.packt.ch04.annotations.Annotation1)

表达式匹配所有由 Annotation1 标记的连接点。

让我们使用切点表达式、通知来实现日志方面，以理解实时实现。我们将使用上一章开发的 Ch03_JdbcTemplates 应用程序作为基础，将其与 Log4j 集成。第一部分我们将创建一个主应用程序的副本，第二部分将其与 log4j 集成，第三部分将应用自定义日志方面。

## 第一部分：创建核心关注点（JDBC）的应用程序

****

按照以下步骤创建基础应用程序：

1.  创建一个名为 Ch04_JdbcTemplate_LoggingAspect 的 Java 应用程序，并添加 Spring 核心、Spring JDBC、spring-aop、aspectjrt-1.5.3 和 aspectjweaver-1.5.3.jar 文件所需的 jar。

1.  将所需源代码文件和配置文件复制到相应的包中。应用程序的最终结构如下所示：

![](img/image_04_002.png)

1.  从 Ch03_JdbcTemplates 复制 connection_new.xml 到应用程序的类路径中，并编辑它以删除 id 为'namedTemplate'的 bean。

## 第二部分：Log4J 的集成

****

Log4j 是最简单的事情。让我们按照以下步骤进行集成：

1.  要集成 Log4J，我们首先必须将 log4j-1.2.9.jar 添加到应用程序中。

1.  在类路径下添加以下配置的 log4j.xml 以添加控制台和文件监听器：

```java
      <!DOCTYPE log4j:configuration SYSTEM "log4j.dtd"> 
      <log4j:configuration  
         xmlns:log4j='http://jakarta.apache.org/log4j/'> 
        <appender name="CA"  
          class="org.apache.log4j.ConsoleAppender"> 
          <layout class="org.apache.log4j.PatternLayout"> 
            <param name="ConversionPattern" value="%-4r [%t]  
              %-5p %c %x - %m%n" /> 
          </layout> 
        </appender> 
        <appender name="file"  
          class="org.apache.log4j.RollingFileAppender"> 
          <param name="File" value="C:\\log\\log.txt" /> 
          <param name="Append" value="true" /> 
          <param name="MaxFileSize" value="3000KB" /> 
          <layout class="org.apache.log4j.PatternLayout"> 
            <param name="ConversionPattern" value="%d{DATE}  
              %-5p %-15c{1}: %m%n" /> 
          </layout> 
        </appender> 
        <root> 
          <priority value="INFO" /> 
          <appender-ref ref="CA" /> 
          <appender-ref ref="file" /> 
        </root> 
      </log4j:configuration> 

```

您可以根据需要修改配置。

1.  现在，为了记录消息，我们将添加获取日志记录器和记录机制的代码。我们可以将代码添加到 BookDAO_JdbcTemplate.java，如下所示：

```java
      public class BookDAO_JdbcTemplate implements BookDAO {  
        Logger logger=Logger.getLogger(BookDAO_JdbcTemplate.class); 
        public int addBook(Book book) { 
          // TODO Auto-generated method stub 
          int rows = 0; 
          String INSERT_BOOK = "insert into book  
            values(?,?,?,?,?,?)"; 
          logger.info("adding the book in table"); 

          rows=jdbcTemplate.update(INSERT_BOOK, book.getBookName(),  
            book.getISBN(), book.getPublication(),  
            book.getPrice(), 
            book.getDescription(), book.getAuthor()); 

            logger.info("book added in the table successfully"+  
              rows+"affected"); 
          return rows; 
        } 

```

不要担心，我们不会在每个类和每个方法中添加它，因为我们已经讨论了复杂性和重复代码，让我们继续按照以下步骤编写日志机制方面，以获得与上面编写的代码相同的结果。

## 第三部分：编写日志方面。

****

1.  在 com.packt.ch04.aspects 包中创建一个名为 MyLoggingAspect 的 Java 类，该类将包含一个用于前置通知的方法。

1.  在其中添加一个类型为 org.apache.log4j.Logger 的数据成员。

1.  在其中添加一个 beforeAdvise()方法。方法的签名可以是任何东西，我们在这里添加了一个 JoinPoint 作为参数。使用这个参数，我们可以获取有关方面应用的类的信息。代码如下：

```java
      public class MyLoggingAspect { 
        Logger logger=Logger.getLogger(getClass()); 
        public void beforeAdvise(JoinPoint joinPoint) { 
          logger.info("method will be invoked :- 
            "+joinPoint.getSignature());   
        }       
      } 

```

1.  现在必须在 XML 中分三步配置方面：

*****为 AOP 添加命名空间：

```java
      <beans xmlns="http://www.springframework.org/schema/beans"     
        xmlns:aop="http://www.springframework.org/schema/aop"  
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
      xsi:schemaLocation="http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans.xsd  
        http://www.springframework.org/schema/aop 
        http://www.springframework.org/schema/aop/spring-aop.xsd">
```

1.  现在我们可以使用 AOP 的标签，通过使用'aop'命名空间：

*****添加一个方面 bean。

1.  在 connection_new.xml 中添加我们想在应用程序中使用的方面的 bean，如下所示：

```java
<bean id="myLogger"
  class="com.packt.ch04.aspects.MyLoggingAspect" />
```

配置切面。

1.  每个<aop:aspect>允许我们在<aop:config>标签内编写切面。

1.  每个切面都将有 id 和 ref 属性。'ref'指的是将调用提供建议的方法的 bean。

1.  为切点表达式配置建议，以及要调用的方法。可以在<aop:aspect>内使用<aop:before>标签配置前置建议。

1.  让我们编写一个适用于'myLogger'切面的前置建议，该建议将在 BookDAO 的 addBook()方法之前调用。配置如下：

```java
      <aop:config>
        <aop:aspect id="myLogger" ref="logging">
          <aop:pointcut id="pointcut1"
            expression="execution(com.packt.ch03.dao.BookDAO.addBook
            (com.packt.ch03.beans.Book))" />
          <aop:before pointcut-ref="pointcut1" 
            method="beforeAdvise"/>
        </aop:aspect>
      </aop:config>
```

1.  执行 MainBookDAO_operation.java 以在控制台获得以下输出：

```java
      0 [main] INFO       org.springframework.context.support.ClassPathXmlApplicationContext -       Refreshing       org.springframework.context.support.ClassPathXmlApplicationContext@5      33e64: startup date [Sun Oct 02 23:44:36 IST 2016]; root of       context hierarchy
      66 [main] INFO       org.springframework.beans.factory.xml.XmlBeanDefinitionReader -       Loading XML bean definitions from class path resource       [connection_new.xml]
      842 [main] INFO       org.springframework.jdbc.datasource.DriverManagerDataSource - Loaded       JDBC driver: com.mysql.jdbc.Driver
      931 [main] INFO com.packt.ch04.aspects.MyLoggingAspect - method       will be invoked :-int com.packt.ch03.dao.BookDAO.addBook(Book)
      book inserted successfully
      book updated successfully
      book deleted successfully
```

BookDAO_JdbTemplate 作为目标对象运行，其代理将在运行时通过编织 addBook()和 beforeAdvise()方法代码来创建。现在既然我们知道了过程，让我们逐一在应用程序中添加不同的切点和建议，并按照以下步骤操作。

### 注意

可以在同一个连接点上应用多个建议，但为了简单地理解切点和建议，我们将每次保留一个建议，并注释掉已经写入的内容。

### 添加返回建议。

让我们为 BookDAO 中的所有方法添加后置建议。

1.  在 MyLoggingAspect 中添加一个后置建议的方法 afterAdvise()，如下所示：

```java
      public void afterAdvise(JoinPoint joinPoint) { 
       logger.info("executed successfully :- 
         "+joinPoint.getSignature()); 
      } 

```

1.  配置切点表达式，以目标 BookDAO 类中的所有方法以及在'myLogger'切面中的 connection_new.xml 中的后置建议。

```java
      <aop:pointcut id="pointcut2"   
        expression="execution(com.packt.ch03.dao.BookDAO.*(..))" /> 
      <aop:after pointcut-ref="pointcut2" method="afterAdvise"/>
```

1.  执行 MainBookDAO_operations.java 以获得以下输出：

```java
999 [main] INFO com.packt.ch04.aspects.MyLoggingAspect - method will be invoked :-int com.packt.ch03.dao.BookDAO.addBook(Book)
1360 [main] INFO com.packt.ch04.aspects.MyLoggingAspect - executed successfully :-int com.packt.ch03.dao.BookDAO.addBook(Book)
book inserted successfully
1418 [main] INFO com.packt.ch04.aspects.MyLoggingAspect - executed successfully :-int com.packt.ch03.dao.BookDAO.updateBook(long,int)
book updated successfully
1466 [main] INFO com.packt.ch04.aspects.MyLoggingAspect - executed successfully :-boolean com.packt.ch03.dao.BookDAO.deleteBook(long)
book deleted successfully
```

下划线的语句清楚地表明建议在所有方法之后被调用。

### 在返回后添加建议。

虽然我们编写了后置建议，但我们无法得到业务逻辑方法返回的值。后返回将帮助我们在以下步骤中获取返回值。

1.  在 MyLoggingAspect 中添加一个返回建议的方法 returnAdvise()，该方法将在返回后调用。代码如下：

```java
      public void returnAdvise(JoinPoint joinPoint, Object val) { 
        logger.info(joinPoint.getSignature()+ " returning val" + val); 
      } 

```

参数'val'将持有返回值。

1.  在'myLogger'下配置建议。我们不需要配置切点，因为我们将会重用已经配置的。如果你想要使用不同的连接点集，首先你需要配置一个不同的切点表达式。我们的配置如下所示：

```java
      <aop:after-returning pointcut-ref="pointcut2"
        returning="val" method="returnAdvise" />
```

其中，

返回-表示要指定返回值传递到的参数的名称。在我们这个案例中，这个名称是'val'，它已在建议参数中绑定。

1.  为了使输出更容易理解，注释掉前置和后置建议配置，然后执行 MainBookDAO_operations.java 以在控制台输出获得以下行：

```java
      1378 [main] INFO  com.packt.ch04.aspects.MyLoggingAspect  - int       com.packt.ch03.dao.BookDAO.addBook(Book)  
      returning val:-1 
      1426 [main] INFO  com.packt.ch04.aspects.MyLoggingAspect  - int       com.packt.ch03.dao.BookDAO.updateBook(long,int) returning val:-1 
      1475 [main] INFO  com.packt.ch04.aspects.MyLoggingAspect  -
      boolean com.packt.ch03.dao.BookDAO.deleteBook(long)
      returning val:-true 

```

每个语句显示了连接点的返回值。

### 添加环绕建议。

如我们之前讨论的，环绕建议在业务逻辑方法前后调用，只有当执行成功时。让我们在应用程序中添加环绕建议：

1.  在`MyLoggingAspect`中添加一个`aroundAdvise()`方法。该方法必须有一个参数是`ProceedingJoinPoint`，以方便应用程序流程到达连接点。代码如下：

![](img/image_04_003.png)

在`proceed()`之前的部分将在我们称为'Pre processing'的 B.L.方法之前被调用。`ProceedingJoinPoint`的`proceed()`方法将流程导向相应的连接点。如果连接点成功执行，将执行`proceed()`之后的部分，我们称之为'Post processing'。在这里，我们通过在'Pre processing'和'Post processing'之间取时间差来计算完成过程所需的时间。

我们想要编织切面的连接点返回 int，因此 aroundAdvise()方法也返回相同类型的值。如果万一我们使用 void 而不是 int，我们将得到以下异常：

```java
      Exception in thread "main" 
      org.springframework.aop.AopInvocationException: Null return value       from advice does not match primitive return type for: public   
      abstract int  
      com.packt.ch03.dao.BookDAO.addBook(com.packt.ch03.beans.Book) 

```

1.  现在让我们在'myLogger'中添加 around advice，如下所示：

```java
      <aop:around pointcut-ref="pointcut1" method="aroundAdvise" />
```

1.  在注释掉之前配置的 advice 的同时，在控制台执行`MainBookDAO`以下日志，

```java
      1016 [main] INFO com.packt.ch04.aspects.MyLoggingAspect - around       advise before int com.packt.ch03.dao.BookDAO.addBook(Book)  
      B.L.method getting invoked
      1402 [main] INFO com.packt.ch04.aspects.MyLoggingAspect - number       of rows affected:-1
      1402 [main] INFO com.packt.ch04.aspects.MyLoggingAspect - around
      advise after int com.packt.ch03.dao.BookDAO.addBook(Book)
      B.L.method getting invoked
      1403 [main] INFO com.packt.ch04.aspects.MyLoggingAspect - int
      com.packt.ch03.dao.BookDAO.addBook(Book) took 388 to complete
```

### 添加 after throwing advice

正如我们所知，一旦匹配的连接点抛出异常，after throwing advice 将被触发。在执行 JDBC 操作时，如果我们尝试在 book 表中添加重复的条目，将抛出 DuplicateKeyException，我们只需要使用以下步骤，借助 after throwing advice 进行日志记录：

1.  在`MyLoggingAspect`中添加`throwingAdvise()`方法，如下所示：

```java
      public void throwingAdvise(JoinPoint joinPoint,  
        Exception exception) 
      { 
        logger.info(joinPoint.getTarget().getClass().getName()+"  
          got and exception" + "\t" + exception.toString()); 
      } 

```

开发人员可以自由选择签名，但由于连接点方法将抛出异常，为 advice 编写的方法将有一个参数是 Exception 类型，这样我们就可以记录它。我们还在参数中添加了 JoinPoint 类型，因为我们想要处理方法签名。

1.  在'myLogger'配置中的`connection_new.xml`中添加配置。要添加的配置是：

```java
      <aop:after-throwing pointcut-ref="pointcut1"
        method="throwingAdvise" throwing="exception" />
```

<aop:after-throwing> 将采取：

* **pointcut-ref** - 我们想要编织连接点的 pointcut-ref 的名称。

* **method** - 如果抛出异常，将调用的方法名称。

* **throwing** - 从 advise 方法签名绑定到的参数名称，异常将被传递给它。我们使用的签名中的参数名称是'exception'。

1.  执行`MainBookDAO_operations`，并故意添加一个 ISBN 已存在于 Book 表中的书籍。在执行前，注释掉为其他 advice 添加的先前配置。我们将得到以下输出：

```java
      1322 [main] ERROR com.packt.ch04.aspects.MyLoggingAspect  - int 
      com.packt.ch03.dao.BookDAO.addBook(Book) got and exception  
      org.springframework.dao.DuplicateKeyException: 
      PreparedStatementCallback; SQL [insert into book 
      values(?,?,?,?,?,?)]; Duplicate entry '9781235' for key 1; nested 
      exception is 
      com.mysql.jdbc.exceptions.jdbc4.MySQLIntegrityConstraintViolation
      Exception: Duplicate entry '9781235' for key 1 

```

1.  如果您使用不同的 ISBN 添加书籍，该 ISBN 不在 book 表中，上述 ERROR 日志将不会显示，因为没有异常，也没有 advice 会被触发。

上述示例清楚地展示了如何使用 XML 编写和配置切面。接下来，让我们来编写基于注解的切面。

## 基于注解的切面。

* * *

方面可以声明为用 AspectJ 注解支持的 Java 类，以支持编写切点和建议。Spring AspectJ OP 实现提供了以下注解，用于编写方面：

+   **@Aspect** - 用于将 Java 类声明为方面。

+   **@Pointcut** - 使用 AspectJ 表达式语言声明切点表达式。

+   **@Before** - 用于声明在业务逻辑（B.L.）方法之前应用的前置建议。@Before 支持以下属性，

    +   **value** - 被@Pointcut 注解的方法名称

    +   **argNames** - 指定连接点处的参数名称

+   **@After** - 用于声明在 B.L.方法返回结果之前应用的后建议。@After 也支持与@Before 建议相同的属性。

+   **@AfterThrowing** - 用于声明在 B.L.方法抛出异常之后应用的后抛出建议。@AfterThrowing 支持以下属性：

    +   **pointcut**- 选择连接点的切点表达式

    +   **throwing**- 与 B.L.方法抛出的异常绑定在一起的参数名称。

+   **@AfterReturning** - 用于声明在 B.L.方法返回结果之前但返回结果之后应用的后返回建议。该建议有助于从 B.L.方法获取返回结果的值。@AfterReturning 支持以下属性，

    +   **pointcut**- 选择连接点的切点表达式

    +   **returning**- 与 B.L.方法返回的值绑定的参数名称。

+   **@Around** - 用于声明在 B.L.方法之前和之后应用的环绕建议。@Around 支持与@Before 或@After 建议相同的属性。

我们必须在 Spring 上下文中声明配置，以禁用 bean 的代理创建。AnnotationAwareAspectJAutoproxyCreator 类在这方面有帮助。我们可以通过在 XML 文件中包含以下配置来简单地为@AspectJ 支持注册类：

```java
<aop:aspectj-autoproxy/> 

```

在 XML 中添加命名空间'aop'，该命名空间已经讨论过。

我们可以按照以下步骤声明和使用基于注解的方面：

1.  声明一个 Java 类，并用@Aspect 注解它。

1.  添加被@Pointcut 注解的方法以声明切点表达式。

1.  根据需求添加建议的方法，并用@Before、@After、@Around 等注解它们。

1.  为命名空间'aop'添加配置。

1.  在配置中作为 bean 添加方面。

1.  在配置中禁用自动代理支持。

让我们在 JdbcTemplate 应用程序中添加基于注解的方面。按照第一部分和第二部分步骤创建名为 Ch04_JdbcTemplate_LoggingAspect_Annotation 的基础应用程序。您可以参考 Ch04_JdbcTemplate_LoggingAspect 应用程序。现在使用以下步骤开发基于注解的日志方面：

1.  在 com.packt.ch04.aspects 包中创建 MyLoggingAspect 类。

1.  用@Aspect 注解它。

1.  在其中添加类型为 org.apache.log4j.Logger 的数据成员。

1.  为应用建议之前的业务逻辑方法 addBook()添加 beforeAdvise()方法。用@Before 注解它。代码如下所示：

```java
      @Aspect 
      public class MyLoggingAspect {
        Logger logger=Logger.*getLogger*(getClass());
        @Before("execution(*  
          com.packt.ch03.dao.BookDAO.addBook(
          com.packt.ch03.beans.Book))") 

        public void beforeAdvise(JoinPoint joinPoint) {
          logger.info("method will be invoked :- 
          "+joinPoint.getSignature()); 
        }
      }
```

1.  如果你还没有做过，编辑 connection_new.xml 以添加'aop'命名空间。

1.  如下的示例中添加 MyLoggingAspect 的 bean：

```java
      <bean id="logging" 
        class="com.packt.ch04.aspects.MyLoggingAspect" />
```

上述配置的替代方案是通过使用@Component 注解来注释 MyLoggingAspect。

1.  通过在 connection_new.xml 中添加配置来禁用 AspectJ 自动代理，如下所示：

```java
      <aop:aspectj-autoproxy/>
```

1.  运行 MainBookDAO-operation.java 以在控制台获取日志：

```java
      23742 [main] INFO  com.packt.ch04.aspects.MyLoggingAspect  - 
      method will be invoked :-int 
      com.packt.ch03.dao.BookDAO.addBook(Book) 

```

为每个建议编写切点表达式可能是一个繁琐且不必要的重复任务。我们可以在标记方法中单独声明切点，如下所示：

```java
      @Pointcut(value="execution(* 
      com.packt.ch03.dao.BookDAO.addBook(com.packt.ch03.beans.Book))") 
        public void selectAdd(){} 

```

然后从建议方法中引用上述内容。我们可以将 beforeAdvise()方法更新为：

```java
      @Before("selectAdd()") 
      public void beforeAdvise(JoinPoint joinPoint) { 
        logger.info("method will be invoked :- 
        "+joinPoint.getSignature()); 
      }
```

1.  一旦我们了解了方面声明的基础，接下来让我们为其他方面和切点添加方法，这些已经在方面声明中使用 XML 讨论过了。方面将如下所示：

```java
      @Aspect 
      public class MyLoggingAspect { 

        Logger logger=Logger.getLogger(getClass()); 
        @Pointcut(value="execution(*com.packt.ch03.dao.BookDAO.addBook(
        com.packt.ch03.beans.Book))") 
        public void selectAdd(){   } 

        @Pointcut(value="execution(*   
          com.packt.ch03.dao.BookDAO.*(..))")

        public void selectAll(){    } 

        // old configuration
        /*
        @Before("execution(* 
        com.packt.ch03.dao.BookDAO.addBook(
        com.packt.ch03.beans.Book))")
        public void beforeAdvise(JoinPoint joinPoint) {
          logger.info("method will be invoked :-
          "+joinPoint.getSignature());
        }
        */
        @Before("selectAdd()") 
        public void beforeAdvise(JoinPoint joinPoint) { 
          logger.info("method will be invoked :- 
          "+joinPoint.getSignature()); 
        }
        @After("selectAll()") 
        public void afterAdvise(JoinPoint joinPoint) { 
          logger.info("executed successfully :- 
          "+joinPoint.getSignature()); 
        }
        @AfterThrowing(pointcut="execution(*
          com.packt.ch03.dao.BookDAO.addBook(
          com.packt.ch03.beans.Book))",  
          throwing="exception") 
        public void throwingAdvise(JoinPoint joinPoint,
          Exception exception)
        {
          logger.error(joinPoint.getSignature()+" got and exception"  
            + "\t" + exception.toString()); 
        }
        @Around("selectAdd()") 
        public int aroundAdvise(ProceedingJoinPoint joinPoint) { 
          long start_time=System.*currentTimeMillis*();
          logger.info("around advise before
          "+joinPoint.getSignature()
          +" B.L.method getting invoked");
        Integer o=null;
        try {
          o=(Integer)joinPoint.proceed();
          logger.info("number of rows affected:-"+o);
        } catch (Throwable e) {
          // TODO Auto-generated catch block
          e.printStackTrace();
        }
        logger.info("around advise after
        "+joinPoint.getSignature()+
        " B.L.method getting invoked");
        long end_time=System.*currentTimeMillis*();
        logger.info(joinPoint.getSignature()+" took " +
        (end_time-start_time)+" to complete");
        return o.intValue();  } 

        @AfterReturning(pointcut="selectAll()", returning="val") 
        public void returnAdvise(JoinPoint joinPoint, Object val) { 
          logger.info(joinPoint.getSignature()+
          " returning val:-" + val); 
        }
      }
```

1.  运行 MainBookDAO.java 以在控制台获取日志消息。

默认情况下，JDK 的动态代理机制将用于创建代理。但是有时目标对象没有实现接口，JDK 的代理机制将失败。在这种情况下，可以使用 CGLIB 来创建代理。为了启用 CGLIB 代理，我们可以编写以下配置：

```java
<aop:config proxy-target-class="true"> 
  <!-aspect configuration à 
</aop:config> 

```

此外，为了强制使用 AspectJ 和自动代理支持，我们可以编写以下配置：

```java
<aop:aspect-autoproxy proxy-target-=class="true"/> 

```

## 引入

* * *

在企业应用程序中，有时开发者会遇到需要引入一组新功能，但又不改变现有代码的情况。使用引入不一定需要改变所有的接口实现，因为这会变得非常复杂。有时开发者会与第三方实现合作，而源代码不可用，引入起到了非常重要的作用。开发者可能有使用装饰器或适配器设计模式的选项，以便引入新功能。但是，方法级 AOP 可以帮助在不编写装饰器或适配器的情况下实现新功能的引入。

引入是一种顾问，它允许在处理交叉关注点的同时引入新的功能。开发者必须使用基于架构的配置的<aop:declare-partents>，或者如果使用基于注解的实现，则使用@DeclareParents。

使用架构添加引入时，<aop:declare-parent>为被建议的 bean 声明一个新的父级。配置如下：

```java
<aop:aspect> 
  <aop:declare-parents types-matching="" implement-interface=" 
    default-impl="" /> 
</aop:aspect> 

```

其中，

+   **类型匹配** - 指定被建议的 been 的匹配类型

+   **实现** - 接口 -  newly introduced interface

+   **默认实现** - 实现新引入接口的类

在使用注解的情况下，开发者可以使用@DeclareParents，它相当于<aop:declare-parents>配置。@DeclareParents 将应用于新引入的接口的属性。@DeclareParents 的语法如下所示：

```java
@DeclareParents(value=" " , defaultImpl=" ") 

```

在哪里，

+   **value** - 指定要与接口引入的 bean

+   **defaultImpl** - 与<aop:declare-parent>属性中的 default-impl 等效，它指定了提供接口实现的类。

让我们在 JdbcTemplate 应用程序中使用介绍。BookDAO 没有获取书籍描述的方法，所以让我们添加一个。我们将使用 Ch03_JdbcTemplate 作为基础应用程序。按照以下步骤使用介绍：

1.  创建一个新的 Java 应用程序，并将其命名为 Ch04_Introduction。

1.  添加所有 Spring 核心、Spring -jdbc、Spring AOP 所需的 jar，正如早期应用程序中所做的那样。

1.  复制 com.packt.ch03.beans 包。

1.  创建或复制 com.packt.ch03.dao，带有 BookDAO.java 和 BookDAO_JdbcTemplate.java 类。

1.  将 connection_new.xml 复制到类路径中，并删除 id 为'namedTemplate'的 bean。

1.  在 com.packt.ch03.dao 包中创建新的接口 BookDAO_new，如下所示，以声明 getDescription()方法：

```java
      public interface BookDAO_new { 
        String getDescription(long ISBN); 
      }
```

1.  创建实现 BookDAO_new 接口的类 BookDAO_new_Impl，它将使用 JdbcTemplate 处理 JDBC。代码如下所示：

```java
      @Repository 
      public class BookDAO_new_Impl implements BookDAO_new { 
        @Autowired 
        JdbcTemplate jdbcTemplate; 
        @Override 
        public String getDescription(long ISBN) { 
          // TODO Auto-generated method stub 
          String GET_DESCRIPTION=" select description from book where           ISBN=?"; 
          String description=jdbcTemplate.queryForObject(
            GET_DESCRIPTION, new Object[]{ISBN},String.class);
          return description; 
        }
      }
```

1.  在 com.packt.ch04.aspects 包中创建一个方面类 MyIntroductionAspect，它将向使用 getDescription()方法的新接口介绍。代码如下所示：

```java
      @Aspect 
      public class MyIntroductionAspect { 
        @DeclareParents(value="com.packt.ch03.dao.BookDAO+",
        defaultImpl=com.packt.ch03.dao.BookDAO_new_Impl.class)
        BookDAO_new bookDAO_new; 
      }
```

注解提供了 BookDAO_new 的介绍，它比 BookDAO 接口中可用的方法多。要用于介绍的默认实现是 BookDAO-new_Impl。

1.  在 connection_new.xml 中注册方面，如下：

```java
      <bean class="com.packt.ch04.aspects.MyIntroductionAspect"></bean>
```

1.  添加以下配置以启用自动代理，

```java
      <aop:aspectj-autoproxy proxy-target-class="true"/>
```

代理目标类用于强制代理成为我们类的子类。

1.  复制或创建 MainBookDAO_operation.java 以测试代码。使用 getDescription()方法查找代码描述。以下代码中的下划线语句是需要添加的额外语句：

```java
      public class MainBookDAO_operations { 
        public static void main(String[] args) { 
          // TODO Auto-generated method stub 
          ApplicationContext context=new  
            ClassPathXmlApplicationContext("connection_new.xml"); 
          BookDAO bookDAO=(BookDAO)  
            context.getBean("bookDAO_jdbcTemplate"); 
          //add book
          int rows=bookDAO.addBook(new Book("Java EE 7 Developer  
          Handbook", 97815674L,"PacktPub
          publication",332,"explore the Java EE7
          programming","Peter pilgrim"));
          if(rows>0) 
          { 
            System.out.println("book inserted successfully"); 
          } 
          else
            System.out.println("SORRY!cannot add book"); 

          //update the book
          rows=bookDAO.updateBook(97815674L,432); 
          if(rows>0) 
          { 
            System.out.println("book updated successfully"); 
          }else 
          System.out.println("SORRY!cannot update book"); 
          String desc=((BookDAO_new)bookDAO).getDescription(97815674L); 
          System.out.println(desc); 

          //delete the book
          boolean deleted=bookDAO.deleteBook(97815674L); 
          if(deleted) 
          { 
            System.out.println("book deleted successfully"); 
          }else 
          System.out.println("SORRY!cannot delete book"); 
        } 
      } 

```

由于 BookDAO 没有 getDescription()方法，为了使用它，我们需要将获得的对象转换为 BookDAO_new。

1.  执行后，我们将在控制台获得以下输出：

```java
      book inserted successfully 
      book updated successfully 
      explore the Java EE7 programming 
      book deleted successfully 

```

输出清楚地显示，尽管我们没有改变 BookDAO 及其实现，就能引入 getDescription()方法。
