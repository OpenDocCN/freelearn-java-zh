# 第一章：Spring Boot 入门

Spring Boot 有很多启动器，它们已经是 Spring Boot 家族的一部分。本章将为您提供一个关于 [`start.spring.io/`](http://start.spring.io) 的概述，可用的启动器模块，并还将向您展示如何使项目 Bootful，正如 Josh Long 喜欢称呼的那样。

在本章中，我们将学习以下主题：

+   使用 Spring Boot 模板和启动器

+   创建一个简单的应用程序

+   使用 Gradle 启动应用程序

+   使用命令行运行器

+   设置数据库连接

+   设置数据存储库服务

+   调度执行器

# 简介

在当今软件开发的快节奏世界中，应用程序创建的速度和快速原型设计的需要变得越来越重要。如果您正在使用 JVM 语言开发软件，Spring Boot 正是那种能够给您提供力量和灵活性的框架，这将使您能够以快速的速度生产高质量的软件。因此，让我们看看 Spring Boot 如何帮助您使您的应用程序 Bootful。

# 使用 Spring Boot 模板和启动器

Spring Boot 包含超过 40 个不同的启动器模块，为许多不同的框架提供现成的集成库，例如既支持关系型数据库也支持 NoSQL 数据库的连接、网络服务、社交网络集成、监控库、日志记录、模板渲染，等等。虽然实际上不可能涵盖所有这些组件，但我们将介绍重要且流行的组件，以了解 Spring Boot 提供的可能性和应用开发的便捷性。

# 如何做到这一点...

我们将首先创建一个基本的简单项目骨架，Spring Boot 将帮助我们实现这一点：

1.  访问 [`start.spring.io`](http://start.spring.io)

1.  填写有关我们项目的详细信息简单表格

1.  点击生成项目 alt + a 预制项目骨架将下载；这就是我们开始的地方

# 它是如何工作的...

您将看到项目依赖关系部分，我们可以选择我们的应用程序将执行的功能类型：它是否会连接到数据库？它是否会有一个网络界面？我们是否计划与任何社交网络集成以及内置操作支持？等等。通过选择所需的技术，适当的启动器库将自动添加到我们预生成的项目模板的依赖列表中。

在我们生成项目之前，让我们了解一下 Spring Boot 启动器究竟是什么以及它为我们提供了哪些好处。

Spring Boot 旨在使创建应用程序变得容易。Spring Boot 启动器是引导库，其中包含启动特定功能所需的所有相关传递依赖项。每个启动器都有一个特殊的文件，其中包含 Spring 提供的所有提供依赖项的列表。以下是一个以`spring-boot-starter-test`定义为例的链接查看：

[`github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-starters/spring-boot-starter-test/src/main/resources/META-INF/spring.provides`](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-starters/spring-boot-starter-test/src/main/resources/META-INF/spring.provides)

在这里，我们将看到以下代码：

```java
provides: spring-test, spring-boot, junit, mockito, hamcrest-library, jsonassert, json-path 
```

这告诉我们，通过在我们的构建中包含`spring-boot-starter-test`作为依赖项，我们将自动获得`spring-test`、`spring-boot`、`junit`、`mockito`、`hamcrest-library`、`jsonassert`和`json-path`。这些库将为我们提供编写我们将开发的软件的应用程序测试所需的所有必要事物，而无需手动将这些依赖项单独添加到构建文件中。

提供了超过 100 个启动器，并且随着社区添加的持续增加，除非我们发现自己需要与一个相当常见或流行的框架集成，否则很可能已经有一个启动器可供我们使用。

下表展示了最显著的一些，以便您了解可用的选项：

| **启动器** | **描述** |
| --- | --- |
| `spring-boot-starter` | 这是核心 Spring Boot 启动器，它为您提供了所有基础功能。它是所有其他启动器的依赖项，因此无需显式声明。 |
| `spring-boot-starter-actuator` | 此启动器为您提供了监控、管理和审计应用程序的功能。 |
| `spring-boot-starter-jdbc` | 此启动器为您提供了连接和使用 JDBC 数据库、连接池等功能的支持。 |
| `spring-boot-starter-data-jpa` `spring-boot-starter-data-*` | JPA 启动器为您提供了所需的库，以便您可以使用**Java 持久化 API**（**JPA**）：Hibernate 以及其他库。各种`data-*`系列启动器为 MongoDB、Data REST 或 Solr 等数据存储提供了支持。 |
| `spring-boot-starter-security` | 这引入了所有 Spring Security 所需的依赖项。 |
| `spring-boot-starter-social-*` | 这允许您与 Facebook、Twitter 和 LinkedIn 集成。 |
| `spring-boot-starter-test` | 这是一个包含`spring-test`和一系列测试框架（如 JUnit 和 Mockito）依赖项的启动器。 |
| `spring-boot-starter-web` | 这为您提供了开发 Web 应用程序所需的所有依赖项。它可以与`spring-boot-starter-hateoas`、`spring-boot-starter-websocket`、`spring-boot-starter-mobile`或`spring-boot-starter-ws`以及各种模板渲染启动器（`sping-boot-starter-thymeleaf`或`spring-boot-starter-mustache`）一起增强。 |
| `spring-cloud-starter-*` | 提供对多个框架（如 Netflix OSS、Consul 或 AWS）支持的多个`cloud-*`家族启动器。 |

# 创建一个简单的应用程序

现在我们对我们可用的启动器有了基本的了解，让我们继续创建我们的应用程序模板，请访问[`start.spring.io`](http://start.spring.io/)。

# 如何操作...

我们将要创建的应用程序是一个图书目录管理系统。它将记录已出版的书籍、作者、评论者、出版社等信息。我们将我们的项目命名为`BookPub`，并执行以下步骤：

1.  首先，让我们通过点击下面的链接切换到完整版本 Generate Project alt + 按钮

1.  在顶部选择 Gradle 项目

1.  使用 Spring Boot 版本 2.0.0(SNAPSHOT)

1.  使用默认建议的组名：`com.example`

1.  在工件字段中输入`bookpub`

1.  将`BookPub`作为应用程序的名称

1.  将`com.example.bookpub`指定为我们的包名

1.  选择 Jar 作为打包方式

1.  使用 Java 版本 8

1.  从“搜索依赖项”选择中选取 H2、JDBC 和 JPA 启动器，以便我们可以在`build`文件中获得连接到 H2 数据库所需的工件

1.  点击 Generate Project alt + 下载项目存档

# 它是如何工作的...

点击 Generate Project alt + 按钮将下载`bookpub.zip`存档，我们将从我们的工作目录中提取它。在新创建的`bookpub`目录中，我们将看到一个`build.gradle`文件，它定义了我们的构建。它已经预配置了正确的 Spring Boot 插件和库版本，甚至包括我们选择的额外启动器。以下为`build.gradle`文件的代码：

```java
dependencies { 
  compile("org.springframework.boot:spring-boot-starter-data-jpa") 
  compile("org.springframework.boot:spring-boot-starter-jdbc") 
  runtime("com.h2database:h2") 
  testCompile("org.springframework.boot:spring-boot-starter-test")  
} 
```

我们已经选择了以下启动器：

+   `org.springframework.boot:spring-boot-starter-data-jpa`：此启动器引入了 JPA 依赖项。

+   `org.springframework.boot:spring-boot-starter-jdbc`：此启动器引入了 JDBC 支持库。

+   `com.h2database`：H2 是一种特定的数据库实现，即 H2。

+   `org.springframework.boot:spring-boot-starter-test`：此启动器引入了运行测试所需的所有必要依赖项。它仅在构建的测试阶段使用，并且在常规应用程序编译时间和运行时不包括在内。

如您所见，`runtime("com.h2database:h2")` 依赖项是一个运行时依赖项。这是因为我们实际上并不需要，甚至可能都不想知道，在编译时我们将连接到哪种确切类型的数据库。Spring Boot 将在检测到应用程序启动时类路径中存在 `org.h2.Driver` 类时自动配置所需的设置并创建适当的 bean。我们将在本章后面探讨这是如何以及在哪里发生的。

`data-jpa` 和 `jdbc` 是 Spring Boot 启动器工件。如果我们下载这些依赖 JAR 文件后查看，或者使用 Maven Central，我们会发现它们不包含任何实际的类，只有各种元数据。两个包含我们感兴趣的文件是 `pom.xml` 和 `spring.provides`。让我们首先查看 `spring-boot-starter-jdbc` JAR 工件中的 `spring.provides` 文件，如下所示：

```java
provides: spring-jdbc,spring-tx,tomcat-jdbc 
```

这告诉我们，通过将这个启动器作为我们的依赖项，我们将通过传递性获得 `spring-jdbc`、`spring-tx` 和 `tomcat-jdbc` 依赖库在我们的构建中。`pom.xml` 文件包含适当的依赖声明，这些声明将在构建时间由 Gradle 或 Maven 使用以解决所需的依赖项。这也适用于我们的第二个启动器：`spring-boot-starter-data-jpa`。这个启动器将传递性地为我们提供 `spring-orm`、`hibernate-entity-manager` 和 `spring-data-jpa` 库。

到目前为止，我们的应用程序类路径中已经有了足够的库/类，以便让 Spring Boot 了解我们正在尝试运行什么类型的应用程序以及需要由 Spring Boot 自动配置以连接在一起的设施和框架类型。

之前我们提到，`org.h2.Driver` 类在类路径中的存在将触发 Spring Boot 自动配置我们的应用程序的 H2 数据库连接。为了确切了解这将会如何发生，让我们首先查看我们新创建的应用程序模板，特别是 `BookPubApplication.java`，它位于项目根目录下的 `src/main/java/com/example/bookpub` 目录中。我们这样做如下：

```java
    package com.example.bookpub; 

    import org.springframework.boot.SpringApplication; 
    import org.springframework.boot.autoconfigure.
    SpringBootApplication; 

    @SpringBootApplication 
    public class BookPubApplication { 

      public static void main(String[] args) { 
        SpringApplication.run(BookPubApplication.class, args); 
      } 
    } 
```

这实际上是我们整个和完全可运行的应用程序。这里没有太多代码，而且肯定没有提到配置或数据库。制作魔法的关键是 `@SpringBootApplication` 元注解。在这里，我们将找到将指导 Spring Boot 自动设置事物的真实注解：

```java
    @SpringBootConfiguration 
    @EnableAutoConfiguration 
    @ComponentScan (excludeFilters = @Filter(type =  
                                     FilterType.CUSTOM,  
                    classes = TypeExcludeFilter.class)) 
    public @interface SpringBootApplication {...} 
```

让我们逐一查看前面代码片段中提到的以下注解列表：

+   `@SpringBootConfiguration`: 这个注解本身是一个元注解；它告诉 Spring Boot，被注解的类包含 Spring Boot 配置定义，例如`@Bean`、`@Component`和`@Service`声明等。在内部，它使用`@Configuration`注解，这是一个 Spring 注解，而不仅仅是 Spring Boot，因为它是一个 Spring 框架核心注解，用于标记包含 Spring 配置定义的类。

需要注意的是，当使用 Spring Boot Test 框架执行测试时，使用`@SpringBootConfiguration`而不是`@Configuration`是有帮助的，因为当测试被`@SpringBootTest`注解时，这个配置将自动被 Test 框架加载。正如 Javadoc 中所述，应用程序应该只包含一个`@SpringApplicationConfiguration`，而大多数习惯性的 Spring Boot 应用程序将继承自`@SpringBootApplication`。

+   `@ComponentScan`: 这个注解告诉 Spring，我们希望从被注解的类所在的包开始扫描我们的应用程序包，作为默认的包根，以便其他可能被`@Configuration`、`@Controller`和其他适用注解注解的类。Spring 会自动将这些类包含在上下文配置中。应用的`TypeExcludeFilter`类提供了过滤功能，用于排除从`ApplicationContext`中排除的各种类。它主要被`spring-boot-test`用于排除仅在测试期间应使用的类；然而，你也可以添加自己的 bean，这些 bean 扩展自`TypeExcludeFilter`，并为其他被认为必要的类型提供过滤功能。

+   `@EnableAutoConfiguration`: 这个注解是 Spring Boot 注解的一部分，它本身也是一个元注解（你会发现 Spring 库非常依赖于元注解，以便它们可以组合和组合配置）。它导入了`EnableAutoConfigurationImportSelector`和`AutoConfigurationPackages.Registrar`类，这些类有效地指示 Spring 根据类路径中可用的类自动配置条件 bean。（我们将在第四章，*编写自定义 Spring Boot 启动器*中详细介绍自动配置的内部工作原理。）

主方法中的`SpringApplication.run(BookPubApplication.class, args);`代码行基本上创建了一个 Spring 应用程序上下文，该上下文读取`BookPubApplication.class`中的注解并实例化一个上下文，这与我们没有使用 Spring Boot 而坚持使用常规 Spring 框架时所做的操作类似。

# 使用 Gradle 启动应用程序

通常，创建任何应用程序的第一步是拥有一个基本的可启动骨架。由于 Spring Boot 启动器已经为我们创建了应用程序模板，我们只需要提取代码、构建并执行它。现在让我们进入控制台，使用 Gradle 启动应用程序。

# 如何操作...

将我们的目录位置更改为`bookpub.zip`存档被提取的位置，并在命令行中执行以下命令：

```java
      $ ./gradlew clean bootRun

```

如果你目录中没有`gradlew`，那么请从[`gradle.org/downloads`](https://gradle.org/install/)下载 Gradle 的一个版本，或者通过执行`brew install gradle`使用 Homebrew 安装它。安装 Gradle 后，在`gradle`文件夹中运行`wrapper`以生成 Gradle `wrapper`文件。另一种方法是调用`$gradleclean bootRun`。

前一个命令的输出将如下所示：

```java
    ...
      .   ____          _            __ _ _
    /\ / ___'_ __ _ _(_)_ __  __ _ 
    ( ( )___ | '_ | '_| | '_ / _` | 
     \/  ___)| |_)| | | | | || (_| |  ) ) ) )
      '  |____| .__|_| |_|_| |___, | / / / /
     =========|_|==============|___/=/_/_/_/
     :: Spring Boot ::  (v2.0.0.BUILD-SNAPSHOT)

    2017-12-16 23:18:53.721 : Starting BookPubApplication on mbp with  
    PID 43850 
    2017-12-16 23:18:53.781 : Refreshing org.springframework.context.
    annotation.Annotatio
    2017-12-16 23:18:55.544 : Building JPA container 
    EntityManagerFactory for persistence 
    2017-12-16 23:18:55.565 : HHH000204: Processing 
    PersistenceUnitInfo name: default 
    2017-12-16 23:18:55.624 : HHH000412: Hibernate Core  
    {5.2.12.Final}
    2017-12-16 23:18:55.625 : HHH000206: hibernate.properties not 
    found
    2017-12-16 23:18:55.627 : HHH000021: Bytecode provider name : 
    javassist
    2017-12-16 23:18:55.774 : HCANN000001: Hibernate Commons 
    Annotations {5.0.1.Final
    2017-12-16 23:18:55.850 : HHH000400: Using dialect: 
    org.hibernate.dialect.H2Dialect
    2017-12-16 23:18:55.902 : HHH000397: Using 
    ASTQueryTranslatorFactory
    2017-12-16 23:18:56.094 : HHH000227: Running hbm2ddl schema 
    export
    2017-12-16 23:18:56.096 : HHH000230: Schema export complete
    2017-12-16 23:18:56.337 : Registering beans for JMX exposure on 
    startup
    2017-12-16 23:18:56.345 : Started BookPubApplication in 3.024 
    seconds (JVM running...
    2017-12-16 23:18:56.346 : Closing 
    org.springframework.context.annotation.AnnotationC..
    2017-12-16 23:18:56.347 : Unregistering JMX-exposed beans on 
    shutdown
    2017-12-16 23:18:56.349 : Closing JPA EntityManagerFactory for 
    persistence unit 'def...
    2017-12-16 23:18:56.349 : HHH000227: Running hbm2ddl schema 
    export
    2017-12-16 23:18:56.350 : HHH000230: Schema export complete
    BUILD SUCCESSFUL
    Total time: 52.323 secs

```

# 它是如何工作的...

如我们所见，应用程序启动得很顺利，但由于我们没有添加任何功能或配置任何服务，它立即就消失了。然而，从启动日志中，我们可以看到自动配置确实发生了。让我们看一下以下几行：

```java
    Building JPA container EntityManagerFactory for persistence unit 
    'default'
    HHH000412: Hibernate Core {5.2.12.Final}
    HHH000400: Using dialect: org.hibernate.dialect.H2Dialect

```

这条信息告诉我们，由于我们添加了`jdbc`和`data-jpa`启动器，JPA 容器被创建，并将使用 Hibernate 5.2.12.Final 通过 H2Dialect 来管理持久性。这是可能的，因为我们有正确的类在类路径中。

# 使用命令行运行器

在我们的基本应用程序骨架准备就绪后，让我们通过让应用程序做一些事情来给它添加一些实质性的内容。

让我们先创建一个名为`StartupRunner`的类。这个类将实现`CommandLineRunner`接口，它基本上只提供了一个方法：`public void run(String... args)` --Spring Boot 将在应用程序启动后只调用一次这个方法。

# 如何操作...

1.  在项目根目录下的`src/main/java/com/example/bookpub/`目录中创建一个名为`StartupRunner.java`的文件，内容如下：

```java
        package com.example.bookpub; 

        import com.example.bookpub.repository.BookRepository;
        import org.apache.commons.logging.Log; 
        import org.apache.commons.logging.LogFactory; 
        import org.springframework.beans.factory.annotation.Autowired;
        import org.springframework.boot.CommandLineRunner; 
        import org.springframework.scheduling.annotation.Scheduled;

        public class StartupRunner implements CommandLineRunner { 
            protected final Log logger = LogFactory.getLog(getClass()); 
            @Override 
            public void run(String... args) throws Exception { 
                logger.info("Hello"); 
            } 
        }
```

1.  在我们定义了类之后，让我们通过在`BookPubApplication.java`应用程序配置中将它定义为`@Bean`来继续，该配置文件位于我们新创建的`StartupRunner.java`文件相同的文件夹中，如下所示：

```java
@Bean 
public StartupRunner schedulerRunner() { 
    return new StartupRunner(); 
} 
```

# 它是如何工作的...

如果我们再次运行我们的应用程序，通过执行`$ ./gradlew clean bootRun`，我们将得到一个类似于之前的输出。然而，我们将在日志中看到我们的`Hello`消息，如下所示：

```java
2017-12-16 21:57:51.048  INFO --- 
com.example.bookpub.StartupRunner         : Hello

```

即使程序在执行过程中会被终止，但我们至少让它做了一些事情！

命令行运行器是一种有用的功能，用于执行只需启动后运行一次的各种类型的代码。有些人也将其用作启动各种执行线程的地方，但 Spring Boot 提供了更好的解决方案来完成这项任务，这将在本章末尾讨论。Spring Boot 使用命令行运行器接口扫描所有实现，并使用启动参数调用每个实例的`run`方法。我们还可以使用`@Order`注解或实现`Ordered`接口来定义我们希望 Spring Boot 执行它们的精确顺序。例如，**Spring Batch**依赖于运行器来触发作业的执行。

由于命令行运行器是在应用程序启动后实例化和执行的，我们可以利用依赖注入来连接我们需要的任何依赖项，例如数据源、服务和其他组件。这些可以在实现`run`时使用。

重要的是要注意，如果在`run(String... args)`方法中抛出任何异常，这将导致上下文关闭并使应用程序关闭。建议使用`try/catch`包装有风险的代码块以防止这种情况发生。

# 设置数据库连接

在每个应用程序中，都需要访问某些数据并对它进行一些操作。最常见的数据源是某种类型的数据存储，即数据库。Spring Boot 使得连接数据库并开始通过 JPA（等等）消费数据变得非常容易。

# 准备中

在我们之前的例子中，我们创建了一个基本的应用程序，该应用程序通过在日志中打印一条消息来执行命令行运行器。让我们通过添加对数据库的连接来增强这个应用程序。

之前，我们已经在`build`文件中添加了必要的`jdbc`和`data-jpa`启动器以及一个 H2 数据库依赖项。现在我们将配置一个 H2 数据库的内存实例。

在嵌入式数据库的情况下，如 H2、**Hyper SQL Database**（**HSQLDB**）或 Derby，除了在`build`文件中包含对这些数据库之一的依赖项之外，不需要进行任何实际配置。当检测到类路径中的这些数据库之一，并在代码中声明了`DataSource` bean 依赖项时，Spring Boot 会自动为您创建一个。

为了证明仅仅在类路径中包含 H2 依赖项，我们就会自动获得一个默认的数据库，让我们修改我们的`StartupRunner.java`文件，使其看起来如下：

```java
public class StartupRunner implements CommandLineRunner { 
    protected final Log logger = LogFactory.getLog(getClass()); 
    @Autowired 
    private DataSource ds; 
    @Override 
    public void run(String... args) throws Exception { 
        logger.info("DataSource: "+ds.toString()); 
    } 
} 
```

现在，如果我们继续运行我们的应用程序，我们将在日志中看到数据源的名字，如下所示：

```java
2017-12-16 21:46:22.067 com.example.bookpub.StartupRunner   
:DataSource: org.apache.tomcat.jdbc.pool.DataSource@4...  {...driverClassName=org.h2.Driver; ... }

```

因此，在底层，Spring Boot 识别到我们已经自动装配了一个`DataSource` bean 依赖项，并自动创建了一个初始化内存中的 H2 数据存储器。这很好，但可能对于早期原型设计阶段或测试目的来说并不太有用。谁会想要一个随着应用程序关闭而消失所有数据的数据库，每次重启应用程序都必须从零开始？

# 如何操作...

为了创建一个嵌入式的 H2 数据库，该数据库不会在内存中存储数据，而是在应用程序重启之间使用文件来持久化数据，我们可以通过以下步骤来更改默认设置：

1.  打开项目根目录下的`src/main/resources`目录中的名为`application.properties`的文件，并添加以下内容：

```java
spring.datasource.url = jdbc:h2:~/test;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE 
spring.datasource.username = sa 
spring.datasource.password = 
```

1.  通过命令行执行`./gradlew clean bootRun`来启动应用程序

1.  检查你的主目录，你应该在那里看到以下文件：`test.mv.db`

用户主目录位于 Linux 的`/home/<username>`和 macOS X 的`/Users/<username>`下。

# 它是如何工作的...

尽管默认情况下，Spring Boot 通过检查类路径中是否存在支持的数据库驱动程序来对数据库配置做出某些假设，但它提供了易于配置的选项，通过一组在`spring.datasource`下分组暴露的属性来调整数据库访问。

我们可以配置的项包括`url`、`username`、`password`、`driver-class-name`等等。如果你想从 JNDI 位置消费数据源，即由外部容器创建的数据源，你可以使用`spring.datasource.jndi-name`属性来配置它。可能的属性集相当大，所以我们不会全部介绍。然而，我们将在[第五章“应用程序测试”中介绍更多选项，我们将讨论使用数据库模拟应用程序测试数据。

通过查看各种博客和示例，你可能注意到有些地方在属性名中使用短横线，如`driver-class-name`，而其他地方则使用驼峰式变体：`driverClassName`。在 Spring Boot 中，这两种实际上是命名同一属性的两个同等支持的方式，并且它们在内部被转换成相同的东西。

如果你想要连接到一个常规（非嵌入）数据库，除了在类路径中拥有适当的驱动库之外，我们还需要在配置中指定我们选择的驱动程序。以下代码片段展示了连接到 MySQL 的配置示例：

```java
    spring.datasource.driver-class-name: com.mysql.jdbc.Driver
    spring.datasource.url:   
    jdbc:mysql://localhost:3306/springbootcookbook
    spring.datasource.username: root
    spring.datasource.password:

```

如果我们想让 Hibernate 根据我们的实体类自动创建模式，我们需要在配置中添加以下行：

```java
    spring.jpa.hibernate.ddl-auto=create-drop

```

不要在生产环境中这样做，否则在启动时，所有表模式和数据都将被删除！在需要的地方使用更新或验证值代替。

你甚至可以在抽象层更进一步，而不是自动装配一个 `DataSource` 对象，你可以直接使用 `JdbcTemplate`。这将指示 Spring Boot 自动创建一个数据源，然后创建一个包装数据源的 `JdbcTemplate`，从而为你提供一个更方便且安全地与数据库交互的方式。`JdbcTemplate` 的代码如下：

```java
@Autowired 
private JdbcTemplate jdbcTemplate; 
```

你也可以查看 `spring-boot-autoconfigure` 源代码中的 `org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration` 文件，以查看数据源创建魔法的代码。

# 设置数据仓库服务

连接到数据库然后执行传统的 SQL，虽然简单直接，但并不是操作数据、将其映射到一组领域对象以及操作关系内容最方便的方式。这就是为什么出现了多个框架来帮助你将数据从表映射到对象，这通常被称为 **对象关系映射**（**ORM**）。此类框架中最著名的例子是 Hibernate。

在上一个示例中，我们介绍了如何设置数据库连接并配置用户名和密码的设置，我们还讨论了应该使用哪个驱动程序等等。在这个菜谱中，我们将通过添加一些定义数据库中数据结构的实体对象和一个 `CrudRepository` 接口来访问数据来增强我们的应用程序。

由于我们的应用程序是一个书籍跟踪目录，显然的领域对象将是 `Book`、`Author`、`Reviewers` 和 `Publisher`。

# 如何做到这一点...

1.  在项目根目录下 `src/main/java/com/example/bookpub` 目录中创建一个名为 `entity` 的新包文件夹。

1.  在这个新创建的包中，创建一个名为 `Book` 的新类，内容如下：

```java
@Entity 
public class Book { 
  @Id 
  @GeneratedValue 
  private Long id; 
  private String isbn; 
  private String title; 
  private String description; 

  @ManyToOne 
  private Author author; 
  @ManyToOne 
  private Publisher publisher; 

  @ManyToMany 
  private List<Reviewers> reviewers; 

  protected Book() {} 

  public Book(String isbn, String title, Author author, 
       Publisher publisher) { 
    this.isbn = isbn; 
    this.title = title; 
    this.author = author; 
    this.publisher = publisher; 
  } 
  //Skipping getters and setters to save space, but we do need them 
} 
```

1.  由于任何书籍都应该有一个作者和一个出版社，理想情况下还有一些评论者，我们需要创建这些实体对象。让我们从创建一个与我们的 `Book` 相同目录下的 `Author` 实体类开始，如下所示：

```java
@Entity 
public class Author { 
  @Id 
  @GeneratedValue 
  private Long id; 
  private String firstName; 
  private String lastName; 
  @OneToMany(mappedBy = "author") 
  private List<Book> books; 

  protected Author() {} 

  public Author(String firstName, String lastName) {...} 
    //Skipping implementation to save space, but we do need 
       it all 
} 
```

1.  同样，我们将创建 `Publisher` 和 `Reviewer` 类，如下面的代码所示：

```java
@Entity 
public class Publisher { 
  @Id 
  @GeneratedValue 
  private Long id; 
  private String name; 
  @OneToMany(mappedBy = "publisher") 
  private List<Book> books; 

  protected Publisher() {} 

  public Publisher(String name) {...} 
} 

@Entity 
public class Reviewer { 
  @Id 
  @GeneratedValue 
  private Long id; 
  private String firstName; 
  private String lastName; 

  protected Reviewer() {} 

  public Reviewer(String firstName, String lastName) 
     {...}
} 
```

1.  现在，我们将在 `src/main/java/com/example/bookpub/repository` 包下通过扩展 Spring 的 `CrudRepository` 接口来创建我们的 `BookRepository` 接口，如下所示：

```java
@Repository 
public interface BookRepository 
       extends CrudRepository<Book, Long> { 
   public Book findBookByIsbn(String isbn); 
} 
```

1.  最后，让我们修改我们的 `StartupRunner` 类，以便打印我们收藏中的书籍数量，而不是一些随机的数据源字符串，通过自动装配一个新创建的 `BookRepository` 并将 `.count()` 调用的结果打印到日志中，如下所示：

```java
public class StartupRunner implements CommandLineRunner { 
  @Autowired private BookRepository bookRepository; 

  public void run(String... args) throws Exception { 
    logger.info("Number of books: " + 
       bookRepository.count()); 
  } 
} 
```

# 它是如何工作的...

如您可能已经注意到的，我们没有写一行 SQL，甚至没有提及任何关于数据库连接、构建查询或类似的事情。我们代码中处理数据库后端数据的唯一线索是类和字段注解的存在：`@Entity`、`@Repository`、`@Id`、`@GeneratedValue` 和 `@ManyToOne`，以及 `@ManyToMany` 和 `@OneToMany`。这些注解是 JPA 的一部分，以及 `CrudRepository` 接口的扩展，是我们与 Spring 沟通需要将我们的对象映射到数据库中适当的表和字段的方式，并为我们提供与这些数据交互的程序化能力。

让我们逐一介绍以下注解：

+   `@Entity` 表示被注解的类应该映射到数据库表。表的名称将派生自类的名称，但如果需要，也可以进行配置。需要注意的是，每个实体类都应该有一个默认的 `protected` 构造函数，这对于自动实例化和 Hibernate 交互是必需的。

+   `@Repository` 表示该接口旨在为您提供对数据库数据的访问和操作。它还作为 Spring 在组件扫描期间的指示，表明此实例应作为一个 bean 创建，该 bean 将在应用程序中使用并可注入到其他 bean 中。

+   `CrudRepository` 接口定义了从数据存储中读取、创建、更新和删除数据的基本通用方法。我们将在 `BookRepository` 扩展中定义的额外方法，如 `public Book findBookByIsbn(String isbn)`，表明 Spring JPA 应将此方法的调用映射到选择 ISBN 字段的 SQL 查询。这是一个约定命名的映射，将方法名称转换为 SQL 查询。它可以是一个非常强大的盟友，允许您构建查询，如 `findByNameIgnoringCase(String name)` 等。

+   `@Id` 和 `@GeneratedValue` 注解向您提供指示，即被注解的字段应映射到数据库的主键列，并且此字段的值应由系统生成，而不是明确输入。

+   `@ManyToOne` 和 `@ManyToMany` 注解定义了关联字段关联，这些关联字段引用其他表中存储的数据。在我们的例子中，多本书属于一个作者，许多评论家评论了多本书。

+   `@OneToMay` 注解中的 `mappedBy` 属性定义了一个反向关联映射。它指示 Hibernate，映射的真实来源定义在 `Book` 类的 `author` 或 `publisher` 字段中。

想了解更多关于 Spring Data 所有强大功能的详细信息，请访问[`docs.spring.io/spring-data/data-commons/docs/current/reference/html/`](http://docs.spring.io/spring-data/data-commons/docs/current/reference/html/).

# 安排执行器

在本章的早期部分，我们讨论了如何将命令行运行器用作启动计划执行器线程池的地方，以间隔运行工作线程。虽然这确实是一种可能性，但 Spring 为你提供了一个更简洁的配置来实现相同的目标：`@EnableScheduling`。

# 准备工作

我们将增强我们的应用程序，使其每 10 秒打印出我们存储库中的书籍数量。为了实现这一点，我们将对`BookPubApplication`和`StartupRunner`类进行必要的修改。

# 如何实现...

1.  让我们在`BookPubApplication`类中添加一个`@EnableScheduling`注解，如下所示：

```java
@SpringBootApplication 
@EnableScheduling 
public class BookPubApplication {...}
```

1.  由于`@Scheduled`注解只能放在没有参数的方法上，让我们在`StartupRunner`类中添加一个新的`run()`方法，并用`@Scheduled`注解标注它，如下行所示：

```java
@Scheduled(initialDelay = 1000, fixedRate = 10000) 
public void run() { 
    logger.info("Number of books: " +  
        bookRepository.count()); 
} 
```

1.  通过在命令行中执行`./gradlew clean bootRun`来启动应用程序，以便观察日志中每 10 秒显示的`Number of books: 0`信息。

# 它是如何工作的...

`@EnableScheduling`，就像我们在本书中讨论和将要讨论的许多其他注解一样，不是一个 Spring Boot；它是一个 Spring Context 模块注解。类似于`@SpringBootApplication`和`@EnableAutoConfiguration`注解，这是一个元注解，并通过`@Import(SchedulingConfiguration.class)`指令内部导入`SchedulingConfiguration`，这个指令可以在由导入的配置创建的`ScheduledAnnotationBeanPostProcessor`内部找到。对于每个没有参数的注解方法，将创建适当的执行器线程池。它将管理注解方法的计划调用。
