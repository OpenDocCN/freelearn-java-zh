# 第十八章：jOOQ SPI（提供者和监听器）

jOOQ 提供了许多钩子，允许我们在不同级别更改其默认行为。在这些钩子中，我们有轻量级的设置和配置，以及由生成器、提供者、监听器、解析器等组成的重型、极其稳定的 **服务提供者接口** (**SPI**)。因此，就像任何强大而成熟的技术一样，jOOQ 携带了一个令人印象深刻的 SPI，专门用于核心技术无法帮助的边缘情况。

在本章中，我们将探讨每个这些钩子，以揭示使用步骤和一些示例，这将帮助你理解如何开发自己的实现。我们的议程包括以下内容：

+   jOOQ 设置

+   jOOQ 配置

+   jOOQ 提供者

+   jOOQ 监听器

+   修改 jOOQ 代码生成过程

让我们开始吧！

# 技术要求

本章的代码可以在 GitHub 上找到 [`github.com/PacktPublishing/jOOQ-Masterclass/tree/master/Chapter18`](https://github.com/PacktPublishing/jOOQ-Masterclass/tree/master/Chapter18)。

# jOOQ 设置

jOOQ 携带了一个全面的设置列表 (`org.jooq.conf.Settings`)，旨在涵盖与渲染 SQL 代码相关的最常用用例。这些设置可以通过声明性方式（通过类路径中的 `jooq-settings.xml`）或通过 `setFooSetting()` 或 `withFooSetting()` 等方法程序化地访问，这些方法可以以流畅的方式链接。为了生效，`Settings` 必须是 `org.jooq.Configuration` 的一部分，这可以通过多种方式完成，正如你可以在 jOOQ 手册中阅读的那样 [`www.jooq.org/doc/latest/manual/sql-building/dsl-context/custom-settings/`](https://www.jooq.org/doc/latest/manual/sql-building/dsl-context/custom-settings/)。但在 Spring Boot 应用程序中，你可能会更喜欢以下方法之一：

通过类路径中的 `jooq-settings.xml` 将全局 `Settings` 传递给默认的 `Configuration`（Spring Boot 准备的 `DSLContext` 将利用这些设置）：

```java
<?xml version="1.0" encoding="UTF-8"?>
```

```java
<settings>
```

```java
  <renderCatalog>false</renderCatalog>
```

```java
  <renderSchema>false</renderSchema>
```

```java
  <!-- more settings added here -->
```

```java
</settings>
```

通过 `@Bean` 将全局 `Settings` 传递给默认的 `Configuration`（Spring Boot 准备的 `DSLContext` 将利用这些设置）：

```java
@org.springframework.context.annotation.Configuration
```

```java
public class JooqConfig {
```

```java
  @Bean
```

```java
  public Settings jooqSettings() {
```

```java
    return new Settings()
```

```java
      .withRenderSchema(Boolean.FALSE) // this is a setting
```

```java
      ... // more settings added here
```

```java
   }  
```

```java
  ...
```

```java
}
```

在某个时刻，设置一个新的全局 `Settings`，从此时开始应用（这是一个全局 `Settings`，因为我们使用 `Configuration#set()`）：

```java
ctx.configuration().set(new Settings()
```

```java
   .withMaxRows(5)
```

```java
   ... // more settings added here
```

```java
   ).dsl()
```

```java
   . // some query
```

将新的全局设置附加到当前的全局 `Settings`：

```java
ctx.configuration().settings()
```

```java
   .withRenderKeywordCase(RenderKeywordCase.UPPER); 
```

```java
ctx. // some query
```

你可以在 MySQL 的 *GlobalSettings* 中练习这些示例。

在某个时刻，设置一个新的局部 `Settings`，它只应用于当前查询（这是一个局部 `Settings`，因为我们使用 `Configuration#derive()`）：

```java
ctx.configuration().derive(new Settings()
```

```java
   .withMaxRows(5)
```

```java
   ... // more settings added here
```

```java
   ).dsl()
```

```java
   . // some query
```

或者，设置全局/局部设置并将其附加到更多局部设置：

```java
ctx.configuration().settings()
```

```java
   .withRenderMapping(new RenderMapping()
```

```java
      .withSchemata(
```

```java
         new MappedSchema()
```

```java
            .withInput("classicmodels")
```

```java
            .withOutput("classicmodels_test")));                
```

```java
// 'derivedCtx' inherits settings of 'ctx'
```

```java
DSLContext derivedCtx = ctx.configuration().derive(
```

```java
    ctx.settings() // using here new Settings() will NOT 
```

```java
                   // inherit 'ctx' settings
```

```java
       .withRenderKeywordCase(RenderKeywordCase.UPPER)).dsl();
```

你可以在 MySQL 的*LocalSettings*中练习这个示例。强烈建议你留出一些时间，至少简要地浏览一下 jOOQ 支持的设置列表，请参阅[`www.jooq.org/javadoc/latest/org.jooq/org/jooq/conf/Settings.html`](https://www.jooq.org/javadoc/latest/org.jooq/org/jooq/conf/Settings.html)。接下来，让我们谈谈 jOOQ 的`Configuration`。

# jOOQ 配置

`org.jooq.Configuration`代表了`DSLContext`的骨干。`DSLContext`需要`Configuration`提供的宝贵信息来进行查询渲染和执行。当`Configuration`利用`Settings`（正如你刚才看到的）时，它还有许多其他可以指定的配置，如本节中的示例。

默认情况下，Spring Boot 为我们提供了一个基于默认`Configuration`（可通过`ctx.configuration()`访问的`Configuration`）构建的`DSLContext`，正如你所知，在提供自定义设置和配置时，我们可以通过`set()`全局地更改此`Configuration`，或者通过创建一个派生版本通过`derive()`局部地更改。

但是，在某些场景下，例如，当你构建自定义提供者或监听器时，你更愿意从一开始就构建`Configuration`以了解你的工件，而不是从`DSLContext`中提取它。换句话说，当`DSLContext`构建时，它应该使用现成的`Configuration`。

在 Spring Boot 2.5.0 之前，这一步需要一点努力，正如你所看到的那样：

```java
@org.springframework.context.annotation.Configuration
```

```java
public class JooqConfig {       
```

```java
  @Bean
```

```java
  @ConditionalOnMissingBean(org.jooq.Configuration.class)
```

```java
  public DefaultConfiguration jooqConfiguration(
```

```java
       JooqProperties properties, DataSource ds, 
```

```java
       ConnectionProvider cp, TransactionProvider tp) {
```

```java
    final DefaultConfiguration defaultConfig = 
```

```java
      new DefaultConfiguration();
```

```java
    defaultConfig               
```

```java
     .set(cp)                                  // must have
```

```java
     .set(properties.determineSqlDialect(ds))  // must have
```

```java
     .set(tp) // for using SpringTransactionProvider
```

```java
     .set(new Settings().withRenderKeywordCase(
```

```java
          RenderKeywordCase.UPPER)); // optional
```

```java
       // more configs ...
```

```java
    return defaultConfig;
```

```java
}
```

这是一个从头开始创建的`Configuration`（实际上是从 jOOQ 内置的`DefaultConfiguration`创建的），Spring Boot 将使用它来创建`DSLContext`。至少，我们需要指定一个`ConnectionProvider`和 SQL 方言。如果我们要使用`SpringTransactionProvider`作为 jOOQ 事务的默认提供者，那么我们需要像以下代码那样设置它。在完成此最小配置后，你可以继续添加你的设置、提供者、监听器等。你可以在 MySQL 的*Before250Config*中练习这个示例。

从 2.5.0 版本开始，Spring Boot 通过一个实现名为`DefaultConfigurationCustomizer`的功能接口的 bean 简化了对 jOOQ 的`DefaultConfiguration`的自定义。这充当一个回调，可以像以下示例那样使用：

```java
@org.springframework.context.annotation.Configuration
```

```java
public class JooqConfig 
```

```java
     implements DefaultConfigurationCustomizer {
```

```java
  @Override
```

```java
  public void customize(DefaultConfiguration configuration) {
```

```java
     configuration.set(new Settings()
```

```java
       .withRenderKeywordCase(RenderKeywordCase.UPPER)); 
```

```java
       ... // more configs
```

```java
    }
```

```java
}
```

这更实用，因为我们只能添加我们需要的。你可以在 MySQL 的*After250Config*中查看这个示例。接下来，让我们谈谈 jOOQ 提供者。

# jOOQ 提供者

jOOQ SPI 公开了一系列提供者，例如`TransactionProvider`、`RecordMapperProvider`、`ConverterProvider`等。它们的总体目标很简单——提供一些 jOOQ 默认提供者没有提供的功能。例如，让我们看看`TransactionProvider`。

## 事务提供者

例如，我们知道 jOOQ 事务在 Spring Boot 中由名为`SpringTransactionProvider`的事务提供者支持（这是 jOOQ 的`TransactionProvider`的 Spring Boot 内置实现），默认情况下暴露一个无名称（`null`）的读写事务，传播行为设置为`PROPAGATION_NESTED`，隔离级别设置为底层数据库的默认隔离级别`ISOLATION_DEFAULT`。

现在，让我们假设我们实现了一个只通过 jOOQ 事务（因此我们不使用`@Transactional`）提供报告的应用程序模块。在这样的模块中，我们不希望允许写入，我们希望在单独的新事务中运行每个查询，超时时间为 1 秒，并避免`PROPAGATION_REQUIRES_NEW`，将隔离级别设置为`ISOLATION_READ_COMMITTED`，并将超时设置为 1 秒。

要获得此类事务，我们可以实现一个`TransactionProvider`并覆盖`begin()`方法，如下面的代码所示：

```java
public class MyTransactionProvider 
```

```java
        implements TransactionProvider {
```

```java
  private final PlatformTransactionManager transactionManager;
```

```java
  public MyTransactionProvider(
```

```java
       PlatformTransactionManager transactionManager) {
```

```java
   this.transactionManager = transactionManager;
```

```java
 }
```

```java
 @Override
```

```java
 public void begin(TransactionContext context) {
```

```java
  DefaultTransactionDefinition definition = 
```

```java
   new DefaultTransactionDefinition(
```

```java
    TransactionDefinition.PROPAGATION_REQUIRES_NEW);
```

```java
  definition.setIsolationLevel(
```

```java
   TransactionDefinition.ISOLATION_READ_COMMITTED);
```

```java
  definition.setName("TRANSACTION_" + Math.round(1000));
```

```java
  definition.setReadOnly(true);
```

```java
  definition.setTimeout(1);
```

```java
  TransactionStatus status =    
```

```java
   this.transactionManager.getTransaction(definition);
```

```java
  context.transaction(new SpringTransaction(status));
```

```java
 }
```

```java
 ...
```

```java
}
```

一旦我们有了事务提供者，我们必须在 jOOQ 中对其进行配置。假设我们正在使用 Spring Boot 2.5.0+，并且根据上一节的内容，可以这样操作：

```java
@org.springframework.context.annotation.Configuration
```

```java
public class JooqConfig 
```

```java
       implements DefaultConfigurationCustomizer {
```

```java
  private final PlatformTransactionManager txManager;
```

```java
  public JooqConfig(PlatformTransactionManager txManager) {
```

```java
   this.txManager = txManager;
```

```java
  }        
```

```java
  @Override
```

```java
  public void customize(DefaultConfiguration configuration) {
```

```java
   configuration.set(newMyTransactionProvider(txManager));
```

```java
  }
```

```java
}
```

你可以在*A250MyTransactionProvider*中练习这个例子，适用于 MySQL。当你运行应用程序时，你会在控制台注意到创建的事务具有以下坐标：*创建新事务，名称为[TRANSACTION_1000]，传播行为为 PROPAGATION_REQUIRES_NEW，隔离级别为 ISOLATION_READ_COMMITTED，超时为 timeout_1，只读*。

如果你使用的是 2.5.0 之前的 Spring Boot 版本，那么请查看名为*B250MyTransactionProvider*的应用程序，适用于 MySQL。

当然，你也可以通过`DSLContext`来配置提供者：

```java
ctx.configuration().set(
```

```java
   new MyTransactionProvider(txManager)).dsl() ...;
```

或者，你也可以使用以下方法：

```java
ctx.configuration().derive(
```

```java
  new MyTransactionProvider(txManager)).dsl() ...;
```

现在，让我们考虑另一个通过`ConverterProvider`解决的问题场景。

## ConverterProvider

我们必须投影一些 JSON 函数并将它们分层映射。我们已经知道，在 Spring Boot + jOOQ 组合中这没有问题，因为 jOOQ 可以获取 JSON 并调用 Jackson（Spring Boot 中的默认选择）来相应地映射它。但是，我们不想使用 Jackson；我们想使用 Flexjson ([`flexjson.sourceforge.net/`](http://flexjson.sourceforge.net/))。jOOQ 不认识这个库（jOOQ 只能检测到 Jackson 和 Gson 的存在），因此我们需要提供一个转换器，例如`org.jooq.ConverterProvider`，它使用 Flexjson 来完成这项任务。请花时间检查*A,B}250ConverterProvider*的源代码，适用于 MySQL。最后，让我们专注于通过`RecordMapperProvider`解决的问题场景。

## RecordMapperProvider

我们有一堆通过 `Builder` 模式实现的遗留 POJO，我们决定为这些 POJO 编写一些 jOOQ `RecordMapper` 以映射查询。为了简化使用这些 `RecordMapper` 的过程，我们还决定编写一个 `RecordMapperProvider`。基本上，这将负责使用适当的 `RecordMapper`，而无需我们显式干预。你对如何做到这一点好奇吗？那么请查看 *{A,B}250RecordMapperProvider* 和 *RecordMapperProvider* 应用于 MySQL 的应用。主要，这些应用是相同的，但它们使用不同的方法来配置 `RecordMapperProvider`。

在 `ConverterProvider` 和 `RecordMapperProvider` 中，我认为重要的是要提到，这些 *替换* 了默认行为，它们并不 *增强* 它。因此，自定义提供者必须确保在无法处理转换/映射时回退到默认实现。

# jOOQ 监听器

jOOQ 提供了相当数量的监听器，它们非常灵活且在钩子到 jOOQ 生命周期管理中非常有用，以解决各种任务。让我们“随意”选择强大的 `ExecuteListener`。

## ExecuteListener

例如，你可能会喜欢的一个监听器是 `org.jooq.ExecuteListener`。这个监听器提供了一组方法，可以将 `Query`、`Routine` 或 `ResultSet` 的生命周期钩子连接到默认的渲染、准备、绑定、执行和检索阶段。实现你自己的监听器的最方便的方法是扩展 jOOQ 默认实现，`DefaultExecuteListener`。这样，你只需覆盖你想要的方法，并保持与 SPI 进化的同步（然而，在你阅读这本书的时候，这个默认监听器可能已经被移除，并且所有方法现在都是接口上的默认方法）。考虑将此技术应用于任何其他 jOOQ 监听器，因为 jOOQ 为所有监听器都提供了默认实现（主要，对于 `FooListener`，有一个 `DefaultFooListener`）。

目前，让我们编写一个 `ExecuteListener`，它将改变即将执行的渲染 SQL。基本上，我们只想通过添加 `/*+ MAX_EXECUTION_TIME(n) */` 指令来修改每一个 MySQL `SELECT`，这样我们就可以指定查询的超时时间（以毫秒为单位）。jOOQ DSL 允许添加 MySQL/Oracle 风格的提示。 :) 使用 `ctx.select(...).hint("/*+ ... */").from(...)`. 但只有 `ExecuteListener` 可以在不修改查询本身的情况下修补多个查询。因此，`ExecuteListener` 提供了回调函数，如 `renderStart(ExecuteContext)` 和 `renderEnd(ExecuteContext)`，分别是在从 `QueryPart` 渲染 SQL 之前和之后调用的。一旦我们掌握了控制权，我们就可以依赖 `ExecuteContext`，它为我们提供了访问底层连接（`ExecuteContext.connection()`）、查询（`ExecuteContext.query()`）、渲染 SQL（`ExecuteContext.sql()`）等的能力。在这个特定的情况下，我们感兴趣的是访问渲染的 SQL 并对其进行修改，因此我们重写了 `renderEnd(ExecuteContext)` 并按照以下方式调用 `ExecuteContext.sql()`：

```java
public class MyExecuteListener extends 
```

```java
    DefaultExecuteListener{
```

```java
  private static final Logger logger = 
```

```java
    Logger.getLogger(MyExecuteListener.class.getName());
```

```java
  @Override
```

```java
  public void renderEnd(ExecuteContext ecx) {
```

```java
    if (ecx.configuration().data()
```

```java
        .containsKey("timeout_hint_select") &&
```

```java
                 ecx.query() instanceof Select) {
```

```java
      String sql = ecx.sql();
```

```java
      if (sql != null) {
```

```java
        ecx.sql(sql.replace(
```

```java
         "select",
```

```java
         "select " + ecx.configuration().data()
```

```java
            .get("timeout_hint_select")
```

```java
      ));
```

```java
      logger.info(() -> {
```

```java
        return "Executing modified query : " + ecx.sql();
```

```java
      });
```

```java
   }
```

```java
  }
```

```java
 }
```

```java
}
```

决策块内的代码相当简单：我们只是捕获即将执行的渲染 SQL（即将不久执行的 SQL）并根据需要通过添加 MySQL 指令来修改它。但是，`...data().containsKey("timeout_hint_select")` 是什么意思呢？主要的是，`Configuration` 提供了三种方法，它们协同工作，通过 `Configuration` 传递自定义数据。这些方法是 `data(Object key, Object value)`，它允许我们设置一些自定义数据；`data(Object key)`，它允许我们根据键获取一些自定义数据；以及 `data()`，它返回整个自定义数据的 `Map`。因此，在我们的代码中，我们检查当前 `Configuration` 的自定义数据是否包含一个名为 `timeout_hint_select` 的键（这是我们选择的名字）。如果存在这样的键，这意味着我们想要将 MySQL 指令（设置为与该键对应的值）添加到当前的 `SELECT` 中，否则，我们不做任何操作。这段自定义信息被设置为如下：

```java
Configuration derived = ctx.configuration().derive();
```

```java
derived.data("timeout_hint_select", 
```

```java
             "/*+ MAX_EXECUTION_TIME(5) */");
```

一旦设置了这段自定义数据，我们就可以执行一个 `SELECT`，它将通过我们的自定义 `ExecuteListener` 被丰富 MySQL 指令：

```java
derived.dsl().select(...).fetch();
```

你可以在 *A250ExecuteListener* 中练习这个例子，针对 MySQL。如果你使用的是 2.5.0 之前的 Spring Boot 版本，那么选择 *B250ExecuteListener*。还有一个名为 *ExecuteListener* 的 MySQL 应用程序，它执行相同的功能，但它通过 `CallbackExecuteListener`（这代表 `ExecuteListener` – 如果你更喜欢函数式组合，这很有用）来“内联” `ExecuteListener`：

```java
ctx.configuration().derive(new CallbackExecuteListener()
```

```java
                   .onRenderEnd(ecx -> {
```

```java
   ...}))
```

```java
   .dsl()
```

```java
   .select(...).fetch();
```

大多数监听器也有一个函数式组合的方法，可以在之前的代码片段中使用。接下来，让我们谈谈一个名为 `ParseListener` 的监听器。

## jOOQ SQL 解析器和 ParseListener

`ParseListener`（SQL 解析监听器）是在 jOOQ 3.15 版本中引入的，但在讨论它之前，我们应该先讨论 SQL `Parser`（`org.jooq.Parser`）。

### SQL 解析器

jOOQ 携带了一个强大且成熟的 `Parser` API，它能够将任意 SQL 字符串（或其片段）解析为不同的 jOOQ API 元素。例如，我们有 `Parser.parseQuery(String sql)`，它返回包含单个查询的 `org.jooq.Query` 类型，该查询对应于传递的 `sql`。

`Parser` API 的主要功能之一是它可以在两种方言之间充当翻译器。换句话说，我们拥有方言 *X* 中的 SQL，并且可以通过 SQL `Parser` 程序化地传递它，以获得为方言 *Y* 翻译/模拟的 SQL。例如，考虑一个包含大量为 MySQL 方言编写的原生查询的 Spring Data JPA 应用程序，如下所示：

```java
@Query(value = "SELECT c.customer_name as customerName, "
```

```java
  + "d.address_line_first as addressLineFirst, 
```

```java
     d.address_line_second as addressLineSecond "
```

```java
  + "FROM customer c JOIN customerdetail d "
```

```java
  + "ON c.customer_number = d.customer_number "
```

```java
  + "WHERE (NOT d.address_line_first <=>
```

```java
    d.address_line_second)", nativeQuery=true)
```

```java
    List<SimpleCustomer> fetchCustomerNotSameAddress();
```

管理层决定切换到 PostgreSQL，因此你应该将这些查询迁移到 PostgreSQL 方言，并且你应该在不显著停机的情况下完成迁移。即使你熟悉这两个方言之间的差异，并且没有问题表达它们，你仍然处于时间压力之下。这是一个 jOOQ 可以帮助你节省时间的场景，因为你只需要将你的原生查询传递给 jOOQ `Parser`，jOOQ 就会为 PostgreSQL 翻译/模拟它们。假设你正在使用由 Hibernate 支持的 Spring Data JPA，那么你只需要添加一个 Hibernate 拦截器，该拦截器会公开即将执行的 SQL 字符串：

```java
@Configuration
```

```java
public class SqlInspector implements StatementInspector {
```

```java
  @Override
```

```java
  public String inspect(String sql) {
```

```java
    Query query = DSL.using(SQLDialect.POSTGRES)
```

```java
      .parser()
```

```java
      .parseQuery(sql);
```

```java
    if (query != null) {
```

```java
        return query.getSQL();
```

```java
    }
```

```java
    return null; // interpreted as the default SQL string
```

```java
  }
```

```java
}
```

5 分钟内完成！这有多酷？！显然，你的同事会问这是哪种魔法，所以你有一个很好的机会向他们介绍 jOOQ。 :)

如果你查看控制台输出，你会看到 Hibernate 报告了以下 SQL 字符串将被用于针对 PostgreSQL 数据库的执行：

```java
SELECT c.customer_name AS customername,
```

```java
       d.address_line_first AS addresslinefirst,
```

```java
       d.address_line_second AS addresslinesecond
```

```java
FROM customer AS c
```

```java
JOIN customerdetail AS d 
```

```java
  ON c.customer_number = d.customer_number
```

```java
WHERE NOT (d.address_line_first IS NOT DISTINCT
```

```java
           FROM d.address_line_second)
```

当然，你可以更改方言，并为任何 jOOQ 支持的方言获取 SQL。现在，你有时间复制 jOOQ 输出并相应地替换你的原生查询，因为应用程序仍然像往常一样运行。最后，只需解耦这个拦截器。你可以在 *JPAParser* 中练习这个应用程序。

除了 `parseQuery()`，我们还有 `parseName(String sql)`，它将给定的 `sql` 解析为 `org.jooq.Name`；`parseField(String sql)`，它将给定的 `sql` 解析为 `org.jooq.Field`；`parseCondition(String sql)`，它将给定的 `sql` 解析为 `org.jooq.Condition`；等等。请查看 jOOQ 文档以了解所有方法和它们的变体。

但 jOOQ 通过所谓的 *解析连接* 功能（也适用于 R2DBC）可以做得更多。基本上，这意味着 SQL 字符串会通过 jOOQ `Parser` 传递，输出 SQL 可以成为 `java.sql.PreparedStatement` 或 `java.sql.Statement` 的来源，这些可以通过这些 JDBC API（`executeQuery(String sql)`）执行。只要 SQL 字符串是通过 JDBC 连接（`java.sql.Connection`）获得的，就像这个例子中那样，就会发生这种情况：

仅从语法上无法决定它可能具有哪种输入语义：

```java
try (Connection c = DSL.using(url, user, pass)
```

```java
      .configuration()
```

```java
      .set(new Settings()
```

```java
          .withParseDialect(SQLDialect.MYSQL)) 
```

```java
      .dsl()
```

```java
      .parsingConnection();  // this does the trick  
```

```java
      PreparedStatement ps = c.prepareStatement(sql);
```

```java
    ) {
```

```java
     ...
```

```java
}
```

传递给 `PreparedStatement` 的 `sql` 代表任何 SQL 字符串。例如，它可以通过 `JdbcTemplate`、Criteria API、`EntityManager` 等生成。从 Criteria API 和 `EntityManager` 中收集 SQL 字符串可能有点棘手（因为它需要 Hibernate 的 `AbstractProducedQuery` 动作），但你可以在 *JPAParsingConnection* 中找到完整的解决方案（MySQL）。

除了 `Parser` API 之外，jOOQ 还通过 `Parser` CLI（[`www.jooq.org/doc/latest/manual/sql-building/sql-parser/sql-parser-cli/`](https://www.jooq.org/doc/latest/manual/sql-building/sql-parser/sql-parser-cli/)）和此网站公开了方言之间的翻译器。现在，我们可以谈谈 `ParseListener`。

### SQL 解析监听器

很容易直观地看出，SQL 解析监听器（jOOQ 3.15 中引入的 `org.jooq.ParseListener`）负责提供钩子，允许更改 jOOQ 解析器的默认行为。

例如，让我们考虑以下 `SELECT` 语句，它使用了 SQL 的 `CONCAT_WS(separator, str1, str2, ...)` 函数：

```java
SELECT concat_ws('|', city, address_line_first, 
```

```java
  address_line_second, country, territory) AS address 
```

```java
FROM office
```

这个忽略 `NULL` 值并使用字符串分隔符/定界符来分隔所有连接到结果字符串中的参数的可变函数在 MySQL、PostgreSQL 和 SQL Server 中是原生支持的，但在 Oracle 中不受支持。此外，jOOQ（至少直到版本 3.16.4）也不支持它。在我们的查询中使用它的方法之一是使用纯 SQL，如下所示：

```java
ctx.resultQuery("SELECT concat_ws('|', city, 
```

```java
  address_line_first, address_line_second, country, territory) 
```

```java
AS address FROM office").fetch();
```

但是，如果我们尝试在 Oracle 上执行此查询，它将不会工作，因为 Oracle 不支持它，并且 jOOQ 也不在 Oracle 语法中模拟它。一个解决方案是实现我们自己的 `ParseListener` 来模拟 `CONCAT_WS()` 的效果。例如，以下 `ParseListener` 通过 `NVL2()` 函数实现了这一点（请阅读代码中的所有注释，以便熟悉此 API）：

```java
public class MyParseListener extends DefaultParseListener {
```

```java
 @Override
```

```java
 public Field parseField(ParseContext pcx) {
```

```java
  if (pcx.parseFunctionNameIf("CONCAT_WS")) {
```

```java
   pcx.parse('(');
```

```java
   String separator = pcx.parseStringLiteral();            
```

```java
   pcx.parse(',');
```

```java
   // extract the variadic list of fields
```

```java
   List<Field<?>> fields = pcx.parseList(",", 
```

```java
       c -> c.parseField()); 
```

```java
   pcx.parse(')'); // the function CONCAT_WS() was parsed      
```

```java
   ...
```

解析后，我们准备 Oracle 模拟：

```java
   ...
```

```java
   // prepare the Oracle emulation
```

```java
   return CustomField.of("", SQLDataType.VARCHAR, f -> {
```

```java
    switch (f.family()) {
```

```java
     case ORACLE -> {
```

```java
      Field result = inline("");
```

```java
      for (Field<?> field : fields) {
```

```java
       result = result.concat(DSL.nvl2(field,
```

```java
                  inline(separator).concat(
```

```java
                    field.coerce(String.class)), field));
```

```java
      }
```

```java
      f.visit(result); // visit this QueryPart
```

```java
     }
```

```java
     // case other dialect ...    
```

```java
     }
```

```java
   });
```

```java
  }
```

```java
  // pass control to jOOQ
```

```java
  return null;
```

```java
 }
```

```java
}
```

为了使代码简单且简短，我们考虑了一些假设。主要的是，分隔符和字符串字面量应该用单引号括起来，分隔符本身是单个字符，并且分隔符之后至少应该有一个参数。

这次，当我们这样做时：

```java
String sql = ctx.configuration().derive(SQLDialect.ORACLE)
```

```java
  .dsl()
```

```java
  .render(ctx.parser().parseQuery("""
```

```java
   SELECT concat_ws('|', city, address_line_first,  
```

```java
     address_line_second, country, territory) AS address 
```

```java
   FROM office"""));
```

```java
ctx.resultQuery(sql).fetch();
```

我们的解析器（随后是 jOOQ 解析器）生成与 Oracle 语法兼容的 SQL：

```java
SELECT ((((('' || nvl2(CITY, ('|' || CITY), CITY)) || 
```

```java
  nvl2(ADDRESS_LINE_FIRST, ('|' || ADDRESS_LINE_FIRST),   
```

```java
       ADDRESS_LINE_FIRST)) || 
```

```java
  nvl2(ADDRESS_LINE_SECOND, ('|' || ADDRESS_LINE_SECOND), 
```

```java
       ADDRESS_LINE_SECOND)) || 
```

```java
  nvl2(COUNTRY, ('|' || COUNTRY), COUNTRY)) || 
```

```java
  nvl2(TERRITORY, ('|' || TERRITORY), TERRITORY)) ADDRESS
```

```java
FROM OFFICE
```

你可以在 *A250ParseListener* 中练习这个示例（适用于 Spring Boot 2.5.0+ 的 Oracle），以及在 *B250ParseListener* 中练习这个示例（适用于 Spring Boot 2.5.0 之前的 Oracle）。除了解析字段（`Field`）外，`ParseListener` 还可以解析表（通过 `parseTable()` 解析 `org.jooq.Table`）和条件（通过 `parseCondition()` 解析 `org.jooq.Condition`）。

如果你更喜欢函数式组合，那么请查看 `CallbackParseListener`。接下来，让我们快速概述其他 jOOQ 监听器。

## 记录监听器

通过 jOOQ `RecordListener`实现，我们可以在`UpdatableRecord`事件（如插入、更新、删除、存储和刷新）期间添加自定义行为（如果您不熟悉`UpdatableRecord`，请考虑*第三章*，*jOOQ 核心概念*)）。

对于`RecordListener`监听的每个*事件*，我们都有一个`eventStart()`和`eventEnd()`方法。`eventStart()`是在*事件*发生之前调用的回调，而`eventEnd()`回调是在*事件*发生后调用的。

例如，让我们考虑每次插入`EmployeeRecord`时，我们都有一个生成主键`EMPLOYEE_NUMBER`的算法。接下来，`EXTENSION`字段始终为*xEmployee_number*类型（例如，如果`EMPLOYEE_NUMBER`是*9887*，则`EXTENSION`是*x9887*）。由于我们不希望让人们手动完成这项任务，我们可以通过`RecordListener`轻松自动化此过程，如下所示：

```java
public class MyRecordListener extends DefaultRecordListener {
```

```java
 @Override
```

```java
 public void insertStart(RecordContext rcx) {
```

```java
  if (rcx.record() instanceof EmployeeRecord employee) {
```

```java
   // call the secret algorithm that produces the PK
```

```java
   long secretNumber = (long) (10000 * Math.random());
```

```java
   employee.setEmployeeNumber(secretNumber);
```

```java
   employee.setExtension("x" + secretNumber);
```

```java
  }
```

```java
 }
```

```java
} 
```

可能值得提及的是，`RecordListener`不适用于普通 DML 语句（更不用说纯 SQL 模板了）。人们常常认为他们可以在其中添加一些安全内容，然后就会被绕过。它实际上仅适用于`TableRecord`/`UpdatableRecord`类型。从 jOOQ 3.16 开始，许多目前使用`RecordListener`解决的问题可能更适合使用`VisitListener`来解决，一旦新的查询对象模型到位，它将变得*更加*强大（[`blog.jooq.org/traversing-jooq-expression-trees-with-the-new-traverser-api/`](https://blog.jooq.org/traversing-jooq-expression-trees-with-the-new-traverser-api/))。在 jOOQ 3.16 中，它可能还没有准备好执行此任务，但可能在 jOOQ 3.17 中实现。

您可以在 MySQL 的*{A,B}250RecordListener1*中练习此应用程序。此外，您还可以找到 MySQL 的*{A,B}250RecordListener2*应用程序，该应用程序通过重写`insertEnd()`来自动在`EMPLOYEE_STATUS`中插入一行，基于插入的`EmployeeRecord`：

```java
@Override
```

```java
public void insertEnd(RecordContext rcx) {
```

```java
  if (rcx.record() instanceof EmployeeRecord employee) {
```

```java
   EmployeeStatusRecord status = 
```

```java
      rcx.dsl().newRecord(EMPLOYEE_STATUS);
```

```java
   status.setEmployeeNumber(employee.getEmployeeNumber());
```

```java
   status.setStatus("REGULAR");
```

```java
   status.setAcquiredDate(LocalDate.now());
```

```java
   status.insert();
```

```java
 }        
```

```java
}        
```

如果您更喜欢函数式组合，那么请查看`CallbackRecordListener`。

## DiagnosticsListener

`DiagnosticsListener`从 jOOQ 3.11 开始提供，并且非常适合您想要检测数据库交互中的低效场景。此监听器可以在不同的级别上操作，例如 jOOQ、JDBC 和 SQL 级别。

主要来说，此监听器公开了一系列回调（每个回调针对它检测到的问题）。例如，我们有`repeatedStatements()`用于检测 N+1 问题，`tooManyColumnsFetched()`用于检测`ResultSet`是否检索了比必要的更多列，`tooManyRowsFetched()`用于检测`ResultSet`是否检索了比必要的更多行，等等（您可以在文档中找到所有这些）。

让我们假设一个运行以下经典 N+1 场景的 Spring Data JPA 应用程序（`Productline`和`Product`实体涉及一个懒加载的双向`@OneToMany`关系）：

```java
@Transactional(readOnly = true)
```

```java
public void fetchProductlinesAndProducts() {
```

```java
  List<Productline> productlines 
```

```java
   = productlineRepository.findAll();
```

```java
  for (Productline : productlines) {
```

```java
   List<Product> products = productline.getProducts();
```

```java
   System.out.println("Productline: " 
```

```java
    + productline.getProductLine()
```

```java
    + " Products: " + products);
```

```java
  }
```

```java
}
```

因此，有一个`SELECT`被触发以获取产品线，对于每个产品线，都有一个`SELECT`来获取其产品。显然，在性能方面，这并不高效，jOOQ 可以通过自定义的`DiagnosticsListener`来发出信号，如下所示：

```java
public class MyDiagnosticsListener 
```

```java
         extends DefaultDiagnosticsListener {    
```

```java
 private static final Logger = ...;   
```

```java
 @Override
```

```java
 public void repeatedStatements(DiagnosticsContext dcx) {
```

```java
  log.warning(() ->
```

```java
   "These queries are prone to be a N+1 case: \n" 
```

```java
     + dcx.repeatedStatements());        
```

```java
 }
```

```java
}
```

现在，之前的 N+1 情况将被记录下来，所以你已经得到了警告！

jOOQ 可以在`java.sql.Connection`（`diagnosticsConnection()`）或`javax.sql.DataSource`（`diagnosticsDataSource()`将`java.sql.Connection`包装在`DataSource`中）上进行诊断。正如在*解析连接*的情况下，这个 JDBC 连接代理了底层连接，因此你必须通过这个代理传递你的 SQL。在一个 Spring Data JPA 应用程序中，你可以快速构建一个依赖于`SingleConnectionDataSource`的诊断配置文件，就像你在 MySQL 的*JPADiagnosticsListener*中看到的那样。对于 MySQL 的*SDJDBCDiagnosticsListener*，情况也是一样的，它包装了一个 Spring Data JDBC 应用程序。jOOQ 手册还有一些很酷的 JDBC 示例，你应该检查一下（[`www.jooq.org/doc/latest/manual/sql-execution/diagnostics/`](https://www.jooq.org/doc/latest/manual/sql-execution/diagnostics/)）。

## TransactionListener

如其名所示，`TransactionListener`提供了干扰事务事件（如开始、提交和回滚）的钩子。对于每个此类*事件*，都有一个在*事件*之前调用的`eventBegin()`，以及一个在*事件*之后调用的`eventEnd()`。此外，为了功能组合的目的，还有`CallbackTransactionListener`。

让我们考虑一个场景，该场景要求我们在每次更新`EmployeeRecord`后备份数据。通过“备份”，我们理解我们需要在要更新的员工的相应文件中保存一个包含此更新之前数据的`INSERT`。

`TransactionListener`不暴露底层 SQL 的信息，因此我们无法从这个监听器内部确定`EmployeeRecord`是否被更新。但是，我们可以从`RecordListener`和`updateStart()`回调中做到这一点。当一个`UPDATE`发生时，`updateStart()`会被调用，我们可以检查记录类型。如果是`EmployeeRecord`，我们可以通过`data()`将其原始（`original()`）状态存储如下：

```java
@Override
```

```java
public void updateStart(RecordContext rcx) {
```

```java
  if (rcx.record() instanceof EmployeeRecord) {
```

```java
   EmployeeRecord employee = 
```

```java
    (EmployeeRecord) rcx.record().original();
```

```java
  rcx.configuration().data("employee", employee);
```

```java
  }
```

```java
}
```

现在，你可能认为，在更新端（`updateEnd()`），我们可以将`EmployeeRecord`的原始状态写入适当的文件。但是，事务可以被回滚，在这种情况下，我们也应该从文件中回滚条目。显然，这是很麻烦的。在事务提交之后再修改文件会容易得多，因此当我们确定更新成功后。这就是`TransactionListener`和`commitEnd()`变得有用的地方：

```java
public class MyTransactionListener 
```

```java
      extends DefaultTransactionListener {
```

```java
 @Override
```

```java
 public void commitEnd(TransactionContext tcx) {
```

```java
  EmployeeRecord employee = 
```

```java
    (EmployeeRecord) tcx.configuration().data("employee");
```

```java
  if (employee != null) {
```

```java
    // write to file corresponding to this employee
```

```java
  }
```

```java
 }
```

```java
}
```

太酷了！你刚刚看到了如何结合两个监听器来完成一个常见任务。检查 MySQL 中的*{A,B}250RecordTransactionListener*的源代码。

## VisitListener

我们将要简要介绍的最后一个监听器可能是最复杂的一个，即 `VisitListener`。主要来说，`VisitListener` 是一个允许我们操作 jOOQ `QueryPart`（查询部分）和 `Clause`（子句）的监听器。因此，我们可以访问 `QueryPart`（通过 `visitStart()` 和 `visitEnd()`）和 `Clause`（通过 `clauseStart()` 和 `clauseEnd()`）。

一个非常简单的例子可能如下所示：我们希望通过 jOOQ DSL (`ctx.createOrReplaceView("product_view").as(...).execute()`) 创建一些视图，并且我们还想将它们添加到 `WITH CHECK OPTION` 子句中。由于 jOOQ DSL 不支持这个子句，我们可以通过 `VisitListener` 来实现，如下所示：

```java
public class MyVisitListener extends DefaultVisitListener {
```

```java
 @Override
```

```java
 public void clauseEnd(VisitContext vcx) {
```

```java
  if (vcx.clause().equals(CREATE_VIEW_AS)) {
```

```java
    vcx.context().formatSeparator()
```

```java
       .sql("WITH CHECK OPTION");
```

```java
  }
```

```java
 }
```

```java
}
```

当你可以在 *{A,B}250VisitListener* 中练习这个简单的例子时，我强烈建议你阅读 jOOQ 博客上的这两篇精彩文章：[`blog.jooq.org/implementing-client-side-row-level-security-with-jooq/`](https://blog.jooq.org/implementing-client-side-row-level-security-with-jooq/) 和 [`blog.jooq.org/jooq-internals-pushing-up-sql-fragments/`](https://blog.jooq.org/jooq-internals-pushing-up-sql-fragments/)。你将有机会学习很多关于 `VisitListener` API 的知识。你永远不知道何时会用到它！例如，你可能想实现你的 *软删除* 机制，为每个查询添加条件，等等。在这些场景中，`VisitListener` 正是你所需要的！此外，当这本书被编写时，jOOQ 开始添加一个新玩家，称为 **查询对象模型**（**QOM**），作为一个公开的 API。这个 API 便于轻松、直观且强大地遍历 jOOQ AST。你不希望错过这篇文章：[`blog.jooq.org/traversing-jooq-expression-trees-with-the-new-traverser-api/`](https://blog.jooq.org/traversing-jooq-expression-trees-with-the-new-traverser-api/)。

接下来，让我们谈谈如何修改 jOOQ 代码生成过程。

# 修改 jOOQ 代码生成过程

我们已经知道 jOOQ 提供了三个代码生成器（用于 Java、Scala 和 Kotlin）。对于 Java，我们使用 `org.jooq.codegen.JavaGenerator`，它可以通过一组综合的配置进行声明性（或程序性）定制（或者，通过 `<configuration>`（Maven）、`configurations`（Gradle）或 `org.jooq.meta.jaxb.Configuration`）。但是，有时我们需要更多的控制，换句话说，我们需要一个自定义的生成器实现。

## 实现自定义生成器

想象一个场景，我们需要一个查询方法，如果它由内置的 jOOQ DAO 提供，那将非常方便。显然，jOOQ 的目标是保持一个薄的 DAO 层，避免由不同类型的查询组合引起的大量方法（不要期望在默认 DAO 中看到 `fetchByField1AndField2()` 这样的查询方法，因为尝试覆盖所有字段的组合（即使是两个字段）会导致一个重量级的 DAO 层，这很可能不会被充分利用）。

但是，我们可以通过自定义生成器来丰富生成的 DAO。一个重要的方面是，自定义生成器需要一个单独的项目（或模块），它将作为将要使用它的项目的依赖项工作。这是必需的，因为生成器必须在编译时运行，所以实现这一点的办法是将它作为一个依赖项添加。由于我们使用的是多模块 Spring Boot 应用程序，我们可以通过将自定义生成器作为项目的单独模块添加来轻松实现这一点。这非常方便，因为大多数 Spring Boot 生产应用程序都是采用多模块风格开发的。

关于自定义生成器的有效实现，我们必须扩展 Java 生成器`org.jooq.codegen.JavaGenerator`，并重写默认的空方法`generateDaoClassFooter(TableDefinition table, JavaWriter out)`。占位符代码如下：

```java
public class CustomJavaGenerator extends JavaGenerator {
```

```java
   @Override
```

```java
   protected void generateDaoClassFooter(
```

```java
         TableDefinition table, JavaWriter out) {
```

```java
      ...
```

```java
   }
```

```java
}
```

基于此占位符代码，让我们生成额外的 DAO 查询方法。

### 向所有 DAO 添加查询方法

假设我们想要向所有生成的 DAO 添加查询方法，例如，添加一个限制获取 POJO（记录）数量的方法，如`List<POJO> findLimitedTo(Integer value)`，其中`value`表示在`List`中要获取的 POJO 数量。查看以下代码：

```java
01:@Override
```

```java
02:protected void generateDaoClassFooter(
```

```java
03:            TableDefinition table, JavaWriter out) {
```

```java
04:
```

```java
05:   final String pType = 
```

```java
06:    getStrategy().getFullJavaClassName(table, Mode.POJO);
```

```java
07:
```

```java
08:   // add a method common to all DAOs
```

```java
09:   out.tab(1).javadoc("Fetch the number of records 
```

```java
10:                limited by <code>value</code>");
```

```java
11:   out.tab(1).println("public %s<%s> findLimitedTo(
```

```java
12:         %s value) {", List.class, pType, Integer.class);
```

```java
13:   out.tab(2).println("return ctx().selectFrom(%s)",  
```

```java
14:              getStrategy().getFullJavaIdentifier(table));
```

```java
15:   out.tab(3).println(".limit(value)");
```

```java
16:   out.tab(3).println(".fetch(mapper());");
```

```java
17:   out.tab(1).println("}");
```

```java
18:}
```

让我们快速看看这里发生了什么：

+   在第 5 行，我们要求 jOOQ 提供与当前`table`对应的生成 POJO 的名称，并在我们的查询方法中使用该名称来返回`List<POJO>`。例如，对于`ORDER`表，`getFullJavaClassName()`返回`jooq.generated.tables.pojos.Order`。

+   在第 9 行，我们生成了一些 Javadoc。

+   在第 11-17 行，我们生成方法签名及其主体。第 14 行使用的`getFullJavaIdentifier()`提供了当前表的完全限定名称（例如，`jooq.generated.tables.Order.ORDER`）。

+   第 13 行使用的`ctx()`方法和第 16 行使用的`mapper()`方法定义在`org.jooq.impl.DAOImpl`类中。每个生成的 DAO 都扩展了`DAOImpl`，因此可以访问这些方法。

基于此代码，jOOQ 生成器在每个生成的 DAO 末尾添加了一个方法，如下所示（此方法添加在`OrderRepository`中）：

```java
/**
```

```java
 * Fetch the number of records limited by <code>value</code>
```

```java
 */
```

```java
public List<jooq.generated.tables.pojos.Order>
```

```java
                    findLimitedTo(Integer value) {
```

```java
   return ctx().selectFrom(jooq.generated.tables.Order.ORDER)
```

```java
               .limit(value)
```

```java
               .fetch(mapper());
```

```java
}
```

那么只在某些 DAO 中添加方法呢？

### 在某些 DAO 中添加查询方法

让我们在`OrderRepository` DAO 中仅添加一个名为`findOrderByStatusAndOrderDate()`的查询方法。一个简单快捷的解决方案是通过`generateDaoClassFooter()`方法的`TableDefinition`参数检查表名。例如，以下代码仅在对应于`ORDER`表的 DAO 中添加了`findOrderByStatusAndOrderDate()`方法：

```java
@Override
```

```java
protected void generateDaoClassFooter(
```

```java
           TableDefinition table, JavaWriter out) {
```

```java
   final String pType 
```

```java
      = getStrategy().getFullJavaClassName(table, Mode.POJO);
```

```java
   // add a method specific to Order DAO
```

```java
   if (table.getName().equals("order")) {
```

```java
      out.println("public %s<%s>
```

```java
         findOrderByStatusAndOrderDate(
```

```java
            %s statusVal, %s orderDateVal) {",
```

```java
               List.class, pType, 
```

```java
                    String.class, LocalDate.class);
```

```java
      ...
```

```java
   }
```

```java
}
```

此代码仅在`jooq.generated.tables.daos.OrderRepository`中生成`findOrderByStatusAndOrderDate()`方法：

```java
/**
```

```java
 * Fetch orders having status <code>statusVal</code>
```

```java
 *  and order date after <code>orderDateVal</code>
```

```java
 */
```

```java
public List<jooq.generated.tables.pojos.Order>
```

```java
      findOrderByStatusAndOrderDate(String statusVal,    
```

```java
         LocalDate orderDateVal) {
```

```java
   return ctx().selectFrom(jooq.generated.tables.Order.ORDER)
```

```java
      .where(jooq.generated.tables.Order.ORDER.STATUS
```

```java
         .eq(statusVal))
```

```java
         .and(jooq.generated.tables.Order.ORDER.ORDER_DATE
```

```java
            .ge(orderDateVal))
```

```java
      .fetch(mapper());
```

```java
}
```

除了`table.getName()`之外，您还可以通过`table.getCatalog()`、`table.getQualifiedName()`、`table.getSchema()`等方法强制执行之前的条件以获得更多控制。

完整示例可在*AddDAOMethods*中找到，适用于 MySQL 和 Oracle。

作为额外的好处，如果您需要用相应的接口丰富 jOOQ 生成的 DAO，那么您需要一个自定义生成器，就像名为*InterfacesDao*的应用程序中为 MySQL 和 Oracle 所做的那样。如果您查看此代码，您将看到一个所谓的*自定义生成器策略*。接下来，让我们详细说明这个方面。

## 编写自定义生成器策略

您已经知道如何使用`<strategy>`（Maven），`strategy {}`（Gradle），或`withStrategy()`（程序化）来在 jOOQ 代码生成过程中注入自定义行为，用于命名类、方法、成员等。例如，我们已使用此技术将我们的 DAO 类重命名为 Spring Data JPA 风格。

但是，在代码生成期间覆盖命名方案也可以通过自定义生成器策略来实现。例如，当我们要生成某些方法名时，这很有用，就像我们场景中的以下查询：

```java
ctx.select(concat(EMPLOYEE.FIRST_NAME, inline(" "),        
```

```java
        EMPLOYEE.LAST_NAME).as("employee"),
```

```java
        concat(EMPLOYEE.employee().FIRST_NAME, inline(" "), 
```

```java
               EMPLOYEE.employee().LAST_NAME).as("reports_to"))
```

```java
   .from(EMPLOYEE)
```

```java
   .where(EMPLOYEE.JOB_TITLE.eq(
```

```java
          EMPLOYEE.employee().JOB_TITLE))
```

```java
   .fetch();
```

这是一个依赖于`employee()`导航方法的自我连接。遵循默认的生成器策略，通过具有与表本身相同名称的导航方法（对于`EMPLOYEE`表，我们有`employee()`方法）来编写自我连接。

但是，如果您觉得`EMPLOYEE.employee()`有点令人困惑，并且您更喜欢更有意义的东西，比如`EMPLOYEE.reportsTo()`（或其它），那么您需要一个自定义生成器策略。这可以通过扩展 jOOQ 的`DefaultGeneratorStrategy`并覆盖 jOOQ 手册中描述的适当方法来实现：[`www.jooq.org/doc/latest/manual/code-generation/codegen-generatorstrategy/`](https://www.jooq.org/doc/latest/manual/code-generation/codegen-generatorstrategy/)。

因此，在我们的情况下，我们需要覆盖`getJavaMethodName()`如下：

```java
public class MyGeneratorStrategy 
```

```java
        extends DefaultGeneratorStrategy {
```

```java
  @Override
```

```java
  public String getJavaMethodName(
```

```java
        Definition, Mode mode) {
```

```java
   if (definition.getQualifiedName()
```

```java
         .equals("classicmodels.employee") 
```

```java
            && mode.equals(Mode.DEFAULT)) {
```

```java
       return "reportsTo";
```

```java
   }
```

```java
   return super.getJavaMethodName(definition, mode);
```

```java
  }
```

```java
}
```

最后，我们必须按照以下方式设置此自定义生成器策略（这里，对于 Maven，但您很容易直观地了解如何为 Gradle 或程序化地做这件事）：

```java
<generator>
```

```java
 <strategy>
```

```java
  <name>
```

```java
   com.classicmodels.strategy.MyGeneratorStrategy
```

```java
  </name>
```

```java
 </strategy>
```

```java
</generator>
```

完成！现在，在代码生成之后，您可以像下面这样重写之前的查询（注意`reportsTo()`方法而不是`employee()`）：

```java
ctx.select(concat(EMPLOYEE.FIRST_NAME, inline(" "), 
```

```java
         EMPLOYEE.LAST_NAME).as("employee"),
```

```java
         concat(EMPLOYEE.reportsTo().FIRST_NAME, inline(" "), 
```

```java
            EMPLOYEE.reportsTo().LAST_NAME).as("reports_to"))
```

```java
   .from(EMPLOYEE)
```

```java
   .where(EMPLOYEE.JOB_TITLE.eq(gma
```

```java
          EMPLOYEE.reportsTo().JOB_TITLE))
```

```java
   .fetch();
```

jOOQ Java 默认生成器策略遵循*Pascal*命名策略，这是 Java 语言中最受欢迎的。但是，除了*Pascal*命名策略之外，jOOQ 还提供了一个`KeepNamesGeneratorStrategy`自定义生成器策略，它只是简单地保留名称。此外，您可能还想研究`JPrefixGeneratorStrategy`，分别的`JVMArgsGeneratorStrategy`。这些只是一些例子（它们不是 jOOQ 代码生成器的一部分），可以在 GitHub 上找到：[`github.com/jOOQ/jOOQ/tree/main/jOOQ-codegen/src/main/java/org/jooq/codegen/example`](https://github.com/jOOQ/jOOQ/tree/main/jOOQ-codegen/src/main/java/org/jooq/codegen/example)。

# 摘要

在本章中，我们简要介绍了 jOOQ SPI。显然，通过 SPI 解决的问题不是日常任务，需要整体上对底层技术有扎实的知识。但是，由于您已经阅读了本书的前几章，您应该没有问题吸收本章中的知识。当然，使用这个 SPI 来解决实际问题需要更多地研究文档并多加实践。

在下一章中，我们将探讨 jOOQ 应用的日志记录和测试。
