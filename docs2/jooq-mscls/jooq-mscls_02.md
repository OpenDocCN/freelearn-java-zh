# 第一章：启动 jOOQ 和 Spring Boot

本章是使用 jOOQ（开源和免费试用商业版）在 Spring Boot 应用程序中开始工作的实用指南。为了方便，让我们假设我们有一个 Spring Boot 占位符应用程序，并计划通过 jOOQ 实现持久层。

本章的目标是强调在 Spring Boot 应用程序中通过 jOOQ 生成和执行 SQL 查询的环境设置几乎可以立即完成。除此之外，这也是体验 jOOQ DSL 流畅 API 并形成第一印象的好机会。

本章的主题包括以下内容：

+   立即启动 jOOQ 和 Spring Boot

+   使用 jOOQ 查询 DSL API 生成有效的 SQL 语句

+   执行生成的 SQL 并将结果集映射到 POJO

让我们开始吧！

# 技术要求

本章的代码可以在 GitHub 上找到，地址为[`github.com/PacktPublishing/jOOQ-Masterclass/tree/master/Chapter01`](https://github.com/PacktPublishing/jOOQ-Masterclass/tree/master/Chapter01)。

# 立即启动 jOOQ 和 Spring Boot

Spring Boot 提供了对 jOOQ 的支持，这一点在 Spring Boot 官方文档的*使用 jOOQ*部分中有所介绍。内置对 jOOQ 的支持使我们的任务变得更容易，因为 Spring Boot 能够处理涉及有用默认配置和设置的各种方面。

考虑有一个针对 MySQL 和 Oracle 运行的 Spring Boot 占位符应用程序，让我们尝试将 jOOQ 添加到这个环境中。目标是使用 jOOQ 作为 SQL 构建器来构建有效的 SQL 语句，并将其作为 SQL 执行器将结果集映射到 POJO。

## 添加 jOOQ 开源版

将 jOOQ 开源版添加到 Spring Boot 应用程序中相当直接。

### 通过 Maven 添加 jOOQ 开源版

从 Maven 的角度来看，将 jOOQ 开源版添加到 Spring Boot 应用程序是从`pom.xml`文件开始的。jOOQ 开源版依赖项可在 Maven Central（[`mvnrepository.com/artifact/org.jooq/jooq`](https://mvnrepository.com/artifact/org.jooq/jooq)）找到，可以添加如下：

```java
<dependency>
```

```java
  <groupId>org.jooq</groupId>
```

```java
  <artifactId>jooq</artifactId>
```

```java
  <version>...</version> <!-- optional -->
```

```java
</dependency>
```

或者，如果您更喜欢 Spring Boot 启动器，那么请依赖这个：

```java
<dependency>
```

```java
  <groupId>org.springframework.boot</groupId>
```

```java
  <artifactId>spring-boot-starter-jooq</artifactId>
```

```java
</dependency>
```

如果您是 Spring Initializr（[`start.spring.io/`](https://start.spring.io/)）的粉丝，那么只需从相应的依赖项列表中选择 jOOQ 依赖项即可。

就这些了！请注意，`<version>`是可选的。如果省略`<version>`，则 Spring Boot 将正确选择与应用程序使用的 Spring Boot 版本兼容的 jOOQ 版本。不过，无论何时您想尝试不同的 jOOQ 版本，都可以简单地显式添加`<version>`。此时，jOOQ 开源版已准备好用于开始开发应用程序的持久层。

### 通过 Gradle 添加 jOOQ 开源版

从 Gradle 的角度来看，将 jOOQ 开源版添加到 Spring Boot 应用程序中可以通过一个名为 `gradle-jooq-plugin` 的插件来实现 ([`github.com/etiennestuder/gradle-jooq-plugin/`](https://github.com/etiennestuder/gradle-jooq-plugin/))。这可以添加到你的 `build.gradle` 文件中，如下所示：

```java
plugins {
```

```java
  id 'nu.studer.jooq' version ...
```

```java
}
```

当然，如果你依赖 Spring Initializr ([`start.spring.io/`](https://start.spring.io/))，那么只需选择一个 Gradle 项目，从相应的依赖列表中添加 jOOQ 依赖项，一旦项目生成，就添加 `gradle-jooq-plugin` 插件。正如你将在下一章中看到的，使用 `gradle-jooq-plugin` 配置 jOOQ 代码生成器非常方便。

## 添加 jOOQ 免费试用版（商业版）

将 jOOQ 免费试用版（商业版）添加到 Spring Boot 项目中（总的来说，在任意其他类型的项目中）需要几个初步步骤。主要因为这些步骤是必要的，因为 jOOQ 免费试用版的商业发行版不在 Maven Central 上，所以你必须手动从 jOOQ 下载页面 ([`www.jooq.org/download/`](https://www.jooq.org/download/)) 下载你需要的版本。例如，你可以选择最受欢迎的版本，即 jOOQ 专业发行版，它被打包成一个 ZIP 归档。一旦解压，你可以通过 `maven-install` 命令本地安装它。你可以在附带代码中的简短电影中找到这些步骤的示例（*Install_jOOQ_Trial.mp4*）。

对于 Maven 应用程序，我们使用标识为 `org.jooq.trial`（针对 Java 17）或 `org.jooq.trial-java-{version}` 的 jOOQ 免费试用版。当本书编写时，`version` 占位符可以是 8 或 11，但请不要犹豫去检查最新的更新。我们更倾向于前者，因此，在 `pom.xml` 文件中，我们有以下内容：

```java
<dependency>
```

```java
  <groupId>org.jooq.trial-java-8</groupId>
```

```java
  <artifactId>jooq</artifactId>
```

```java
  <version>...</version>
```

```java
</dependency>
```

对于 Java/Gradle，你可以像以下示例那样通过 `gradle-jooq-plugin` 来实现：

```java
jooq {
```

```java
  version = '...'
```

```java
  edition = nu.studer.gradle.jooq.JooqEdition.TRIAL_JAVA_8
```

```java
}
```

对于 Kotlin/Gradle，你可以这样做：

```java
jooq {
```

```java
  version.set(...)
```

```java
  edition.set(nu.studer.gradle.jooq.JooqEdition.TRIAL_JAVA_8)
```

```java
}
```

在本书中，我们将使用 jOOQ 开源版在涉及 MySQL 和 PostgreSQL 的应用程序中，以及在涉及 SQL Server 和 Oracle 的应用程序中使用 jOOQ 免费试用版。这两个数据库供应商不支持 jOOQ 开源版。

如果你感兴趣在 Quarkus 项目中添加 jOOQ，那么可以考虑这个资源：[`github.com/quarkiverse/quarkus-jooq`](https://github.com/quarkiverse/quarkus-jooq)

## 将 DSLContext 注入 Spring Boot 仓库

jOOQ 最重要的接口之一是`org.jooq.DSLContext`。此接口代表使用 jOOQ 的起点，其主要目标是配置 jOOQ 在执行查询时的行为。此接口的默认实现名为`DefaultDSLContext`。在多种方法中，`DSLContext`可以通过`org.jooq.Configuration`对象、直接从 JDBC 连接（`java.sql.Connection`）、数据源（`javax.sql.DataSource`）以及用于将 Java API 查询表示形式转换为特定数据库 SQL 查询的方言（`org.jooq.SQLDialect`）来创建。

重要提示

对于`java.sql.Connection`，jOOQ 将为您提供对连接生命周期的完全控制（例如，您负责关闭此连接）。另一方面，通过`javax.sql.DataSource`获取的连接将在 jOOQ 查询执行后自动关闭。Spring Boot 喜欢数据源，因此连接管理已经处理（从连接池中获取和返回连接，事务开始/提交/回滚等）。

所有的 jOOQ 对象，包括`DSLContext`，都是从`org.jooq.impl.DSL`创建的。为了创建`DSLContext`，`DSL`类公开了一个名为`using()`的`static`方法，它有多种形式。其中最值得注意的是以下列出的：

```java
// Create DSLContext from a pre-existing configuration
```

```java
DSLContext ctx = DSL.using(configuration);
```

```java
// Create DSLContext from ad-hoc arguments
```

```java
DSLContext ctx = DSL.using(connection, dialect);
```

例如，连接到 MySQL 的`classicmodels`数据库可以这样做：

```java
try (Connection conn = DriverManager.getConnection(
```

```java
    "jdbc:mysql://localhost:3306/classicmodels", 
```

```java
    "root", "root")) {
```

```java
  DSLContext ctx = 
```

```java
    DSL.using(conn, SQLDialect.MYSQL);
```

```java
  ...
```

```java
} catch (Exception e) {
```

```java
  ...
```

```java
}
```

或者，您可以通过数据源进行连接：

```java
DSLContext ctx = DSL.using(dataSource, dialect);
```

例如，通过数据源连接到 MySQL 的`classicmodels`数据库可以这样做：

```java
DSLContext getContext() {
```

```java
  MysqlDataSource dataSource = new MysqlDataSource();
```

```java
  dataSource.setServerName("localhost");
```

```java
  dataSource.setDatabaseName("classicmodels");
```

```java
  dataSource.setPortNumber("3306");
```

```java
  dataSource.setUser(props.getProperty("root");
```

```java
  dataSource.setPassword(props.getProperty("root");
```

```java
  return DSL.using(dataSource, SQLDialect.MYSQL);
```

```java
}
```

但 Spring Boot 能够根据我们的数据库设置自动准备一个可注入的`DSLContext`。例如，Spring Boot 可以根据在`application.properties`中指定的 MySQL 数据库设置准备`DSLContext`：

```java
spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver
```

```java
spring.datasource.url=jdbc:mysql://localhost:3306/
```

```java
                 classicmodels?createDatabaseIfNotExist=true
```

```java
spring.datasource.username=root
```

```java
spring.datasource.password=root
```

```java
spring.jooq.sql-dialect=MYSQL
```

一旦 Spring Boot 检测到 jOOQ 的存在，它将使用前面的设置来创建`org.jooq.Configuration`，该配置用于准备一个可注入的`DSLContext`。

重要提示

虽然`DSLContext`具有高度的可配置性和灵活性，但 Spring Boot 仅进行最小努力来提供默认的`DSLContext`，该`DSLContext`可以立即注入和使用。正如您将在本书中看到的那样（但尤其是在官方 jOOQ 手册中 – [`www.jooq.org/doc/latest/manual/`](https://www.jooq.org/doc/latest/manual/))，`DSLContext`具有大量配置和设置，允许控制与我们的 SQL 语句相关的几乎所有操作。

Spring Boot 提供的`DSLContext`对象可以轻松注入到我们的持久化仓库中。例如，以下代码片段直接将这样的`DSLContext`对象注入到`ClassicModelsRepository`中：

```java
@Repository
```

```java
public class ClassicModelsRepository {
```

```java
  private final DSLContext ctx;
```

```java
  public ClassicModelsRepository(DSLContext ctx) {
```

```java
    this.ctx = ctx;
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

不要在这里得出结论，应用程序需要保留对 `DSLContext` 的引用。它仍然可以直接在局部变量中使用，就像你之前看到的那样（这意味着你可以有任意多的 `DSLContext` 对象）。这只意味着，在 Spring Boot 应用程序中，对于大多数常见场景，简单地像之前那样注入它会更方便。

在内部，jOOQ 可以使用 `java.sql.Statement` 或 `PreparedStatement`。默认情况下，并且出于非常好的原因，jOOQ 使用 `PreparedStatement`。

通常，`DSLContext` 对象被标记为 `ctx`（本书中使用）或 `dsl`。但 `dslContext`、`jooq` 和 `sql` 等其他名称也是不错的选择。基本上，你可以随意命名。

好的，到目前为止，一切顺利！在这个时候，我们可以访问 Spring Boot 提供的 `DSLContext`，这是基于我们在 `application.properties` 中的设置。接下来，让我们通过 jOOQ 的查询 DSL API 看看 `DSLContext` 的实际应用。

# 使用 jOOQ 查询 DSL API 生成有效的 SQL

使用 jOOQ 查询 DSL API 生成有效的 SQL 是探索 jOOQ 世界的一个良好开端。让我们从一个简单的 SQL 语句开始，并通过 jOOQ 来表达它。换句话说，让我们使用 jOOQ 查询 DSL API 将给定的 SQL 字符串查询表达为 jOOQ 面向对象的风格。考虑以下用 MySQL 语法编写的 SQL `SELECT` 语句：

```java
SELECT * FROM `office` WHERE `territory` = ?
```

SQL 语句 ``SELECT * FROM `office` WHERE `territory` = ?`` 被写为一个普通的字符串。如果通过 DSL API 编写，jOOQ 可以生成此查询，如下所示（`territory` 绑定变量的值由用户提供）：

```java
ResultQuery<?> query = ctx.selectFrom(table("office"))
```

```java
  .where(field("territory").eq(territory));
```

或者，如果我们想让 `FROM` 子句更接近 SQL 的外观，我们可以这样写：

```java
ResultQuery<?> query = ctx.select()
```

```java
  .from(table("office"))                  
```

```java
  .where(field("territory").eq(territory));
```

大多数模式都是不区分大小写的，但有一些数据库，如 MySQL 和 PostgreSQL，通常更倾向于小写，而其他数据库，如 Oracle，通常更倾向于大写。因此，按照 Oracle 风格编写前面的查询可以这样做：

```java
ResultQuery<?> query = ctx.selectFrom(table("OFFICE"))
```

```java
  .where(field("TERRITORY").eq(territory));
```

或者，你可以通过显式调用 `from()` 来编写它：

```java
ResultQuery<?> query = ctx.select()
```

```java
  .from(table("OFFICE"))                  
```

```java
  .where(field("TERRITORY").eq(territory));
```

jOOQ 流畅 API 是一件艺术品，看起来像流畅的英语，因此阅读和编写起来相当直观。

阅读前面的查询完全是英语：*从 OFFICE 表中选择所有办公室，其中 TERRITORY 列等于给定的值*。

很快，你就会对在 jOOQ 中编写这些查询的速度感到惊讶。

重要提示

正如你将在下一章中看到的，jOOQ 可以通过名为 jOOQ 代码生成器的功能生成基于 Java 的模式，该模式与数据库中的模式相对应。一旦启用此功能，编写这些查询将变得更加简单和清晰，因为将不再需要显式引用数据库模式，例如表名或表列。相反，我们将引用基于 Java 的模式。

此外，多亏了代码生成器功能，jOOQ 在几乎所有地方都为我们提前做出了正确的选择。我们不再需要关心查询的类型安全性和大小写敏感性，或者标识符的引号和限定。

jOOQ 代码生成器原子性地提升了 jOOQ 的功能并增加了开发者的生产力。这就是为什么推荐使用 jOOQ 代码生成器来充分利用 jOOQ。我们将在下一章中探讨 jOOQ 代码生成器。

接下来，必须对 jOOQ 查询（`org.jooq.ResultQuery`）执行数据库操作，并将结果集映射到用户定义的简单 POJO。

# 执行生成的 SQL 并映射结果集

通过 jOOQ 的 API 中的获取方法执行生成的 SQL 并将结果集映射到 POJO，可以这样做。例如，下面的代码片段依赖于 `fetchInto()` 方法：

```java
public List<Office> findOfficesInTerritory(String territory) {
```

```java
  List<Office> result = ctx.selectFrom(table("office"))
```

```java
.where(field("territory").eq(territory))
```

```java
.fetchInto(Office.class); 
```

```java
  return result;
```

```java
}
```

那里发生了什么？！`ResultQuery` 去哪里了？这是黑魔法吗？显然不是！只是 jOOQ 在构建查询后立即获取了结果并将它们映射到了 `Office` POJO。是的，jOOQ 的 `fetchInto(Office.class)` 或 `fetch().into(Office.class)` 会正常工作。主要的是，jOOQ 通过以更面向对象的方式封装和抽象 JDBC 复杂性来执行查询并将结果集映射到 `Office` POJO。如果我们不想在构建查询后立即获取结果，则可以使用 `ResultQuery` 对象如下：

```java
// 'query' is the ResultQuery object
```

```java
List<Office> result = query.fetchInto(Office.class);
```

`Office` POJO 包含在这本书附带代码中。

重要提示

jOOQ 提供了一个全面的 API，用于将结果集获取和映射到集合、数组、映射等。我们将在后面的*第八章*“获取和映射”中详细说明这些方面。

完整的应用程序命名为 *DSLBuildExecuteSQL*。由于这可以用作存根应用程序，您可以在 Java/Kotlin 与 Maven/Gradle 的组合中找到它。这些应用程序（实际上，本书中的所有应用程序）使用 Flyway 进行模式迁移。您将看到，Flyway 和 jOOQ 是一对绝佳的搭档。

因此，在继续利用令人惊叹的 jOOQ 代码生成器功能之前，让我们快速总结本章内容。

# 摘要

注意，我们仅仅通过使用 jOOQ 生成和执行一个简单的 SQL 语句，就几乎触及了 jOOQ 功能的表面。尽管如此，我们已经强调 jOOQ 可以针对不同的方言生成有效的 SQL，并且可以以直接的方式执行和映射结果集。

在下一章中，我们将学习如何通过增加 jOOQ 的参与程度来更加信任 jOOQ。jOOQ 将代表我们生成类型安全的查询、POJO 和 DAO。
