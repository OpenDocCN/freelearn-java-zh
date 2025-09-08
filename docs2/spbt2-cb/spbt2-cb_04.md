# 编写自定义 Spring Boot 启动器

在本章中，我们将涵盖以下主题：

+   理解 Spring Boot 自动配置

+   创建自定义 Spring Boot 自动配置启动器

+   配置自定义条件性 Bean 实例化

+   使用自定义 @Enable 注解切换配置

# 简介

在前面的章节中，我们在开发 Spring Boot 应用程序时做了很多配置，甚至更多的自动配置。现在，是时候揭开 Spring Boot 自动配置背后的魔法，并编写一些我们自己的启动器了。

这是一种非常实用的能力，尤其是在拥有专有代码的大型软件企业中，专有代码的存在是不可避免的。能够创建内部自定义启动器，自动添加一些配置或功能到应用程序中，非常有帮助。一些可能的候选包括自定义配置系统、库以及处理连接数据库、使用自定义连接池、HTTP 客户端、服务器等配置。我们将深入了解 Spring Boot 自动配置的内部机制，查看新启动器的创建方式，探索基于各种规则的 Bean 条件初始化和连接，并看到注解可以是一个强大的工具，为启动器的消费者提供更多控制权，以决定应该使用哪些配置以及在哪里使用。

# 理解 Spring Boot 自动配置

当涉及到启动应用程序并配置它以精确地包含所需的所有内容时，Spring Boot 具有强大的功能，而无需我们开发者编写大量的粘合代码。这种力量的秘密实际上来自于 Spring 本身，或者更确切地说，来自于它提供的 Java 配置功能。随着我们添加更多的启动器作为依赖项，我们的类路径中将会出现越来越多的类。Spring Boot 会检测特定类的存在或不存在，并根据这些信息做出一些决策，有时这些决策相当复杂，并自动创建和连接必要的 Bean 到应用程序上下文中。

听起来很简单，对吧？

在前面的食谱中，我们添加了多个 Spring Boot 启动器，例如 `spring-boot-starter-data-jpa`、`spring-boot-starter-web`、`spring-boot-starter-data-test` 等。我们将使用上一章完成相同的代码，以便查看应用程序启动过程中实际发生的情况以及 Spring Boot 在连接我们的应用程序时所做的决策。

# 如何操作...

1.  便利的是，Spring Boot 提供了一种通过简单地以 `debug` 标志启动应用程序来获取 `CONDITIONS EVALUATION REPORT` 的能力。这可以通过环境变量 `DEBUG`、系统属性 `-Ddebug` 或应用程序属性 `--debug` 传递给应用程序。

1.  通过运行 `DEBUG=true ./gradlew clean bootRun` 来启动应用程序。

1.  现在，如果您查看控制台日志，您将看到那里打印了更多标记为`DEBUG`级别的信息。在启动日志序列的末尾，我们将看到如下所示的`CONDITIONS EVALUATION REPORT`：

```java
    =========================
    CONDITIONS EVALUATION REPORT
    =========================

    Positive matches:
    -----------------
    ...
    DataSourceAutoConfiguration
          - @ConditionalOnClass classes found:    
            javax.sql.DataSource,org.springframework.jdbc.
            datasource.embedded.EmbeddedDatabaseType   
            (OnClassCondition)
            ...

    Negative matches:
    -----------------
    ...
    GsonAutoConfiguration
          - required @ConditionalOnClass classes not found:  
          com.google.gson.Gson (OnClassCondition)
          ...
```

# 它是如何工作的...

如您所见，在调试模式下打印的信息量可能有些令人不知所措，所以我只选择了一个正匹配和一个负匹配的例子。

对于报告中的每一行，Spring Boot 都会告诉我们为什么某些配置被选中包含在内，它们在哪些方面进行了正匹配，或者对于负匹配，是什么缺失的阻止了特定配置被包含在组合中。让我们看看`DataSourceAutoConfiguration`的正匹配：

+   找到的`@ConditionalOnClass`类告诉我们 Spring Boot 已经检测到特定类的存在，在我们的例子中是两个类：`javax.sql.DataSource`和`org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseType`。

+   `OnClassCondition`指示使用了哪种匹配方式。这由`@ConditionalOnClass`和`@ConditionalOnMissingClass`注解支持。

虽然`OnClassCondition`是最常见的检测类型，但 Spring Boot 还使用了许多其他条件。例如，`OnBeanCondition`用于检查特定 bean 实例的存在或不存在，`OnPropertyCondition`用于检查属性的存在、不存在或特定值，以及可以使用`@Conditional`注解和`Condition`接口实现定义的任何数量的自定义条件。

负匹配显示 Spring Boot 评估过的配置列表，这意味着它们确实存在于类路径中，并被 Spring Boot 扫描，但未通过包含所需的条件。`GsonAutoConfiguration`虽然作为导入的`spring-boot-autoconfigure`工件的一部分存在于类路径中，但由于所需的`com.google.gson.Gson`类未检测到存在于类路径中，因此未能通过`OnClassCondition`。

`GsonAutoConfiguration`文件的实现如下所示：

```java
@Configuration 
@ConditionalOnClass(Gson.class) 
public class GsonAutoConfiguration { 

  @Bean 
  @ConditionalOnMissingBean 
  public Gson gson() { 
    return new Gson(); 
  } 

} 
```

在查看代码后，很容易将条件注解与 Spring Boot 在启动时提供的信息之间的联系联系起来。

# 创建自定义 Spring Boot 自动配置启动器

我们对 Spring Boot 决定将哪些配置包含在应用程序上下文形成过程中的过程有一个高级的了解。现在，让我们尝试创建我们自己的 Spring Boot 启动器工件，我们可以将其作为可自动配置的依赖项包含在我们的构建中。

在 第二章，*配置 Web 应用程序*，你学习了如何创建数据库 `Repository` 对象。所以，让我们构建一个简单的起始作品，它将创建另一个 `CommandLineRunner`，该 `CommandLineRunner` 将获取所有 `Repository` 实例的集合并打印出每个的总条目数。

我们将首先向现有项目添加一个子 Gradle 项目，该项目将包含起始作品的代码库。我们将称之为 `db-count-starter`。

# 如何做到这一点...

1.  我们将首先在项目根目录下创建一个名为 `db-count-starter` 的新目录。

1.  由于我们的项目现在已成为所谓的 `multiproject` 构建，我们需要在项目根目录中创建一个 `settings.gradle` 配置文件，其内容如下：

```java
include 'db-count-starter' 
```

1.  我们还应该在项目根目录下的 `db-count-starter` 目录中为我们的子项目创建一个单独的 `build.gradle` 配置文件，其内容如下：

```java
apply plugin: 'java' 

repositories { 
  mavenCentral() 
  maven { url "https://repo.spring.io/snapshot" } 
  maven { url "https://repo.spring.io/milestone" } 

} 

dependencies { 
  compile("org.springframework.boot:spring-boot:2.0.0.BUILD-SNAPSHOT")  
  compile("org.springframework.data:spring-data-commons:2.0.2.RELEASE") 
} 
```

1.  现在我们已经准备好开始编码了。所以，第一步是创建目录结构，`src/main/java/com/example/bookpubstarter/dbcount`，在项目根目录下的 `db-count-starter` 目录中。

1.  在新创建的目录中，让我们添加名为 `DbCountRunner.java` 的 `CommandLineRunner` 文件实现，其内容如下：

```java
public class DbCountRunner implements CommandLineRunner { 
    protected final Log logger = LogFactory.getLog(getClass()); 

    private Collection<CrudRepository> repositories; 

    public DbCountRunner(Collection<CrudRepository> repositories) { 
        this.repositories = repositories; 
    } 

    @Override 
    public void run(String... args) throws Exception { 
        repositories.forEach(crudRepository -> 
            logger.info(String.format("%s has %s entries", 
                getRepositoryName(crudRepository.getClass()), 
                crudRepository.count()))); 

    } 

    private static String 
            getRepositoryName(Class crudRepositoryClass) { 
        for(Class repositoryInterface : 
                crudRepositoryClass.getInterfaces()) { 
            if (repositoryInterface.getName(). 
                    startsWith("com.example.bookpub.repository")) { 
                return repositoryInterface.getSimpleName(); 
            } 
        } 
        return "UnknownRepository"; 
    } 
} 
```

1.  在 `DbCountRunner` 的实际实现到位后，我们现在需要创建一个配置对象，该对象将在配置阶段声明性地创建一个实例。所以，让我们创建一个名为 `DbCountAutoConfiguration.java` 的新类文件，其内容如下：

```java
@Configuration 
public class DbCountAutoConfiguration { 
    @Bean 
    public DbCountRunner dbCountRunner
               (Collection<CrudRepository> repositories) { 
        return new DbCountRunner(repositories); 
    } 
}
```

1.  我们还需要告诉 Spring Boot，我们新创建的 JAR 软件包包含自动配置类。为此，我们需要在项目根目录下的 `db-count-starter/src/main` 目录中创建一个 `resources/META-INF` 目录。

1.  在这个新创建的目录中，我们将放置名为 `spring.factories` 的文件，其内容如下：

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.bookpubstarter.dbcount.DbCountAutoConfiguration 
```

1.  为了演示的目的，我们将在主项目的 `build.gradle` 文件中添加对起始作品的依赖项，通过在依赖项部分添加以下条目：

```java
compile project(':db-count-starter') 
```

1.  通过运行 `./gradlew clean bootRun` 启动应用程序。

1.  一旦应用程序编译并启动，我们应该在控制台日志中看到以下内容：

```java
    2017-12-16 INFO com.example.bookpub.StartupRunner        : Welcome to the Book Catalog System!
    2017-12-16 INFO c.e.b.dbcount.DbCountRunner              : AuthorRepository has 1 entries
    2017-12-16 INFO c.e.b.dbcount.DbCountRunner              : PublisherRepository has 1 entries
    2017-12-16 INFO c.e.b.dbcount.DbCountRunner              : BookRepository has 1 entries
    2017-12-16 INFO c.e.b.dbcount.DbCountRunner              : ReviewerRepository has 0 entries
    2017-12-16 INFO com.example.bookpub.BookPubApplication   : Started BookPubApplication in 8.528 seconds (JVM running for 9.002)
    2017-12-16 INFO com.example.bookpub.StartupRunner        : Number of books: 1

```

# 它是如何工作的...

恭喜！你现在已经构建了自己的 Spring Boot 自动配置起始作品。

首先，让我们快速浏览一下我们对 Gradle 构建配置所做的更改，然后我们将详细检查起始设置。

由于 Spring Boot 启动器是一个独立的、独立的工件，仅仅将更多类添加到我们现有的项目源树中并不能真正展示出很多。为了使这个独立的工件，我们有几种选择：在我们的现有项目中创建一个单独的 Gradle 配置，或者完全创建一个全新的项目。然而，最理想的解决方案是将我们的构建转换为 Gradle 多项目构建，通过在根项目的`build.gradle`文件中添加嵌套项目目录和子项目依赖来实现。通过这样做，Gradle 实际上为我们创建了一个独立的 JAR 工件，但我们不需要将其发布到任何地方，只需将其作为编译`project(':db-count-starter')`依赖项包含即可。

关于 Gradle 多项目构建的更多信息，您可以查看[`gradle.org/docs/current/userguide/multi_project_builds.html`](http://gradle.org/docs/current/userguide/multi_project_builds.html)的手册。

Spring Boot 自动配置启动器不过是一个带有`@Configuration`注解的常规 Spring Java 配置类，并且类路径中的`META-INF`目录下存在`spring.factories`文件，其中包含适当的配置条目。

在应用程序启动期间，Spring Boot 使用`SpringFactoriesLoader`（它是 Spring Core 的一部分），以获取为`org.springframework.boot.autoconfigure.EnableAutoConfiguration`属性键配置的 Spring Java 配置列表。在底层，这个调用收集了类路径中所有 jar 或其他条目在`META-INF`目录下的所有`spring.factories`文件，并构建一个复合列表作为应用程序上下文配置添加。除了`EnableAutoConfiguration`键之外，我们还可以以类似的方式声明以下自动初始化的启动实现：

+   `org.springframework.context.ApplicationContextInitializer`

+   `org.springframework.context.ApplicationListener`

+   `org.springframework.boot.autoconfigure.AutoConfigurationImportListener`

+   `org.springframework.boot.autoconfigure.AutoConfigurationImportFilter`

+   `` `org.springframework.boot.autoconfigure.template.TemplateAvailabilityProvider` ``

+   `org.springframework.boot.SpringBootExceptionReporter`

+   `org.springframework.boot.SpringApplicationRunListener`

+   `org.springframework.boot.env.PropertySourceLoader`

+   `org.springframework.boot.env.EnvironmentPostProcessor`

+   `org.springframework.boot.diagnostics.FailureAnalyzer`

+   `org.springframework.boot.diagnostics.FailureAnalysisReporter`

+   `org.springframework.test.contex.TestExecutionListener`

充满讽刺意味的是，Spring Boot Starter 不需要依赖 Spring Boot 库作为其编译时依赖项。如果我们查看`DbCountAutoConfiguration`类中的类导入列表，我们将不会看到来自`org.springframework.boot`包的任何内容。我们声明对 Spring Boot 的依赖的唯一原因是因为我们的`DbCountRunner`实现实现了`org.springframework.boot.CommandLineRunner`接口。

# 配置自定义条件 bean 实例化

在前面的例子中，你学习了如何启动基本的 Spring Boot Starter。在将 jar 包包含在应用程序类路径中时，`DbCountRunner` bean 将被自动创建并添加到应用程序上下文中。在本章的第一个菜谱中，我们也看到了 Spring Boot 具有根据一些条件进行条件配置的能力，例如类路径中存在特定类、bean 的存在以及其他一些条件。

对于这个配方，我们将通过条件检查来增强我们的启动器。这将在没有其他此类 bean 实例已被创建并添加到应用程序上下文的情况下，创建`DbCountRunner`的实例。

# 如何做到这一点...

1.  在`DbCountAutoConfiguration`类中，我们将向`dbCountRunner(...)`方法添加一个`@ConditionalOnMissingBean`注解，如下所示：

```java
@Bean 
@ConditionalOnMissingBean 
public DbCountRunner 
   dbCountRunner(Collection<CrudRepository> repositories) { 
  return new DbCountRunner(repositories); 
} 
```

1.  我们还需要将`spring-boot-autoconfigure`组件的依赖项添加到`db-count-starter/build.gradle`文件的依赖项部分：

```java
compile("org.springframework.boot:spring-boot-autoconfigure:2.0.0.BUILD-SNAPSHOT")
```

1.  现在，让我们通过在终端中运行`./gradlew clean bootRun`来启动应用程序，以验证我们将在控制台日志中看到与之前菜谱中相同的输出。

1.  如果我们使用`DEBUG`开关启动应用程序以查看自动配置报告，正如我们在本章第一道菜谱中学到的，我们将看到我们的自动配置位于 Positive Matches 组中，如下所示：

```java
DbCountAutoConfiguration#dbCountRunner
 - @ConditionalOnMissingBean (types: com.example.bookpubstarter.dbcount.DbCountRunner; SearchStrategy: all) found no beans (OnBeanCondition)
```

1.  让我们在主`BookPubApplication`配置类中显式/手动创建一个`DbCountRunner`的实例，并且我们还将覆盖其`run(...)`方法，这样我们就可以在日志中看到差异：

```java
protected final Log logger = LogFactory.getLog(getClass()); 
@Bean 
public DbCountRunner dbCountRunner
                     (Collection<CrudRepository> repositories) { 
  return new DbCountRunner(repositories) { 
    @Override 
    public void run(String... args) throws Exception { 
      logger.info("Manually Declared DbCountRunner"); 
    } 
  }; 
} 
```

1.  通过运行`DEBUG=true ./gradlew clean bootRun`来启动应用程序。

1.  如果我们查看控制台日志，我们将看到两件事：自动配置报告将在 Negative Matches 组中打印我们的自动配置，并且，而不是每个存储库的计数输出，我们将看到`Manually Declared DbCountRunner`文本出现：

```java
DbCountAutoConfiguration#dbCountRunner
 - @ConditionalOnMissingBean (types: com.example.bookpubstarter.dbcount.DbCountRunner; SearchStrategy: all) found the following [dbCountRunner] (OnBeanCondition)

2017-12-16 INFO com.example.bookpub.BookPubApplication$1    : Manually Declared DbCountRunner
```

# 它是如何工作的...

如我们从之前的配方中学到的，Spring Boot 将在应用程序上下文创建期间自动处理所有来自`spring.factories`的配置类条目。在没有额外指导的情况下，所有使用`@Bean`注解注解的内容都将用于创建 Spring Bean。这个功能实际上是 Java 配置的 Spring Framework 的一部分。Spring Boot 在此基础上增加的是能够条件性地控制某些`@Configuration`或`@Bean`注解何时执行以及何时最好忽略它们的能力。

在我们的案例中，我们使用了`@ConditionalOnMissingBean`注解来指示 Spring Boot 仅在没有任何其他匹配类类型或已声明的 bean 名称的 bean 时创建我们的`DbCountRunner` bean。由于我们在`BookPubApplication`配置中明确创建了一个`@Bean`条目用于`DbCountRunner`，这具有优先权，导致`OnBeanCondition`检测到 bean 的存在；从而指示 Spring Boot 在应用程序上下文设置期间不使用`DbCountAutoConfiguration`。

# 使用自定义的`@Enable`注解切换配置

允许 Spring Boot 自动评估类路径和检测到的配置，这使得启动一个简单应用程序变得非常快速和简单。然而，有时我们希望提供配置类，但要求启动库的消费者明确启用此类配置，而不是依赖 Spring Boot 自动决定是否包含它。

我们将修改之前的配方，使启动器通过元注解启用，而不是使用`spring.factories`路由。

# 如何操作...

1.  首先，我们将注释掉位于项目根目录`db-count-starter/src/main/resources`中的`spring.factories`文件的内容，如下所示：

```java
#org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
#com.example.bookpubstarter.dbcount.DbCountAutoConfiguration
```

1.  接下来，我们需要创建元注解。我们将在项目根目录下的`db-count-starter/src/main/java/com/example/bookpubstarter/dbcount`目录中创建一个名为`EnableDbCounting.java`的新文件，其内容如下：

```java
@Target(ElementType.TYPE) 
@Retention(RetentionPolicy.RUNTIME) 
@Import(DbCountAutoConfiguration.class) 
@Documented 
public @interface EnableDbCounting { 
} 
```

1.  现在，我们将向我们的`BookPubApplication`类添加`@EnableDbCounting`注解，并从其中删除`dbCountRunner(...)`方法，如下面的代码片段所示：

```java
@SpringBootApplication 
@EnableScheduling 
@EnableDbCounting 
public class BookPubApplication { 

  public static void main(String[] args) { 
    SpringApplication.run(BookPubApplication.class, args); 
  } 

  @Bean 
  public StartupRunner schedulerRunner() { 
    return new StartupRunner(); 
  } 
} 
```

1.  通过运行`./gradlew clean bootRun`来启动应用程序。

# 它是如何工作的...

运行应用程序后，你可能会首先注意到打印的计数全部显示为`0`，尽管`StartupRunner`已经在控制台打印了`Number of books: 1`，如下所示：

```java
c.e.b.dbcount.DbCountRunner         : AuthorRepository has 0 entries
c.e.b.dbcount.DbCountRunner         : BookRepository has 0 entries
c.e.b.dbcount.DbCountRunner         : PublisherRepository has 0 entries
c.e.b.dbcount.DbCountRunner         : ReviewerRepository has 0 entries
com.example.bookpub.StartupRunner   : Welcome to the Book Catalog System!
com.example.bookpub.StartupRunner   : Number of books: 1  
```

这是因为 Spring Boot 正在随机执行 `CommandLineRunners`，并且由于我们更改了配置以使用 `@EnableDbCounting` 注解，它会在 `BookPubApplication` 类本身的配置之前被处理。由于数据库填充是由我们在 `StartupRunner.run(...)` 方法中完成的，而 `DbCountRunner.run(...)` 的执行发生在之前，因此数据库表没有数据，所以报告了 `0` 个计数。

如果我们要强制执行顺序，Spring 通过 `@Order` 注解提供了这种能力。让我们用 `@Order(Ordered.LOWEST_PRECEDENCE - 15)` 注解 `StartupRunner` 类。由于 `LOWEST_PRECEDENCE` 是默认分配的顺序，我们将通过稍微降低顺序号来确保 `StartupRunner` 将在 `DbCountRunner` 之后执行。让我们再次运行应用程序，现在我们将看到计数被正确显示。

现在我们已经解决了这个小排序问题，让我们更详细地检查我们用 `@EnableDbCounting` 注解做了什么。

没有包含配置的 `spring.factories`，Spring Boot 并不知道在应用程序上下文创建过程中应该包含 `DbCountAutoConfiguration` 类。默认情况下，配置组件扫描只会从 `BookPubApplication` 包及其以下部分开始。由于包不同——`com.example.bookpub` 与 `com.example.bookpubstarter.dbcount`——扫描器不会找到它。

这就是我们的新创建的元注解发挥作用的地方。在 `@EnableDbCounting` 注解中，有一个键嵌套注解 `@Import(DbCountAutoConfiguration.class)`，这使得事情发生。这是一个由 Spring 提供的注解，可以用来注解其他注解，声明在过程中应该导入哪些配置类。通过用 `@EnableDbCounting` 注解我们的 `BookPubApplication` 类，我们递归地告诉 Spring 应该将 `DbCountAutoConfiguration` 作为应用程序上下文的一部分包含进来。

使用便利的元注解、`spring.factories` 和条件 Bean 注解，我们现在可以创建复杂且详尽的定制自动配置 Spring Boot 启动器，以满足我们企业的需求。
