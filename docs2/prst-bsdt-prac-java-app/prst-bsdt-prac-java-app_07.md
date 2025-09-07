

# 第七章：jOOQ 采用指南的缺失部分

**面向对象编程（OOP**）是讨论企业架构时最流行的方法；然而，还有更多，例如数据驱动。在当今数据驱动的世界中，jOOQ 已成为开发者用来与数据库交互的强大工具，提供了一种无缝且高效的 SQL 工作方式。

首先，让我们解决基本问题：什么是 jOOQ？**jOOQ**，即**Java 面向对象查询**，是一个轻量级但强大的 Java 库，使开发者能够流畅且直观地编写类型安全的 SQL 查询。它提供了一个**领域特定语言（DSL**），封装了 SQL 的复杂性，使开发者能够专注于编写简洁且易于阅读的代码。

现在，你可能想知道为什么 jOOQ 在开发者中获得了显著的吸引力。答案在于其能够弥合数据库的关联世界与现代应用程序开发面向对象范式之间的差距。jOOQ 使开发者能够在 Java 代码中充分利用 SQL 的全部功能，提供传统**对象关系映射（ORM**）框架往往难以实现的灵活性、性能和可维护性。

当我们深入到 jOOQ 的世界时，我们将探讨数据驱动设计的概念及其影响。与主要围绕操作对象及其行为的传统面向对象不同，数据驱动设计强调底层数据结构和它们之间的关系。我们将研究 jOOQ 如何采用这种方法，赋予开发者高效处理复杂数据库交互的能力，同时保持强类型和编译时安全性的好处。

在本章中，我们将探讨 jOOQ 框架以及如何在具有 Jakarta EE 和 MicroProfile 的企业架构中使用它：

+   Java 中的数据驱动和面向对象编程

+   什么是 jOOQ？

+   使用 jOOQ 与 Jakarta/MicroProfile

因此，让我们开始这段旅程，去发现 jOOQ 的力量，并理解它是如何革命性地改变我们与数据库交互的方式，弥合 SQL 世界和面向对象世界之间的差距。

# 技术要求

本章需要以下内容：

+   Java 17

+   Git

+   Maven

+   任何首选的 IDE

+   本章的代码可以在以下位置找到：[`github.com/PacktPublishing/Persistence-Best-Practices-for-Java-Applications/tree/main/chapter-07`](https://github.com/PacktPublishing/Persistence-Best-Practices-for-Java-Applications/tree/main/chapter-07)

## Java 中的数据驱动和面向对象编程

在 Java 中，**数据驱动编程**指的是一种方法，其中底层数据和其结构主要驱动程序的设计和功能。它侧重于以允许灵活性、可扩展性和易于修改的方式操作和处理数据，而不太依赖于对象的行为。

相比之下，面向对象编程是一种围绕对象旋转的编程范式，对象是类的实例。面向对象编程强调在对象内部封装数据和相关的行为，促进诸如继承、多态和抽象等概念。它侧重于将现实世界实体建模为对象，并通过方法和关系定义其行为。

数据驱动编程与面向对象编程之间的关键区别在于它们对程序设计的处理方式。在面向对象编程中，重点是建模实体及其行为，围绕对象及其交互组织代码。当对象的行为复杂或需要表示系统中的现实世界实体时，这种方法效果很好。

另一方面，数据驱动编程优先考虑操作和处理数据结构。当处理大量数据，如数据库或以数据为中心的应用程序时，这很有益。数据驱动编程允许高效地查询、过滤和转换数据，通常利用如 SQL 或其他查询语言之类的声明式方法。

在某些情况下，数据驱动方法可能比面向对象方法更适合。以下是一些例子：

+   **数据处理和分析**：在处理大量数据或执行复杂分析任务时，使用专用库或框架的数据驱动方法可以提供更好的性能和灵活性。

+   **数据库驱动应用程序**：在开发与数据库交互频繁或依赖于外部数据的应用程序时，如 jOOQ 这样的数据驱动方法可以简化数据库交互并优化查询执行。

+   **配置驱动系统**：在行为主要由配置文件或外部数据决定的系统中，数据驱动方法允许轻松修改和定制，而无需更改代码。

+   **基于规则的系统**：在涉及复杂规则评估或基于数据做出决策的应用程序中，数据驱动方法可以提供一种透明且易于管理的表达和处理规则的方式。

重要的是要注意，面向对象编程和数据驱动编程不是相互排斥的，它们通常可以结合使用，以在 Java 应用程序中实现所需的功能和可维护性。两种方法之间的选择取决于系统的具体要求和要解决的问题的性质。

虽然数据驱动编程提供了一些优势，但也伴随着不可避免的权衡。以下是与数据驱动编程相关的某些权衡：

+   **复杂性增加**：数据驱动编程可能会引入额外的复杂性，尤其是在处理大型和复杂的数据结构时。在细粒度级别管理和操作数据可能需要复杂的代码和逻辑，使系统更难以理解和维护。

+   **减少封装**：在数据驱动编程中，重点主要在于数据和它的操作，而不是在对象内封装行为。这可能导致封装减少和数据暴露增加，可能损害系统的安全和完整性。

+   **有限的表达性**：虽然数据驱动编程提供了强大的数据操作和查询机制，但在表达复杂业务逻辑或数据之间的关系时可能存在限制。面向对象，强调行为和封装，通常可以提供更具有表达性和直观的解决方案。

尽管存在这些权衡，但在高效的数据操作、查询和灵活性至关重要的情况下，数据驱动编程可以非常有用。通过理解这些权衡，开发者在选择面向对象和数据驱动方法时可以做出明智的决定，考虑到他们应用程序的具体需求和限制。

面向对象是讨论企业应用时最流行的范式；然而，我们可以探索更多范式，例如数据驱动设计。

注意

本章简要概述了这一主题，但如果你想要深入研究，这里有两份推荐的资料。

第一份资料是 Yehonathan Sharvit 所著的《数据驱动编程》一书，其中讨论了这一模式，我们可以总结出三个原则：

+   代码是数据分离的

+   数据是不可变的

+   数据具有灵活的访问

第二份资料是一篇名为《数据驱动编程》的文章，作者是 Brian Goetz，他解释了 Java 的新特性，主要是记录，以及如何利用 Java 的优势。

在对数据驱动编程有一个概述之后，让我们深入探讨一个最受欢迎的框架，它可以帮助你设计和创建数据驱动应用程序：jOOQ。

## 什么是 jOOQ？

jOOQ 是一个强大的 Java 库，在企业应用场景中架起了面向对象和数据驱动编程之间的桥梁。虽然面向对象长期以来一直是开发企业应用的主导范式，但在某些情况下，数据驱动方法可以提供独特的优势。jOOQ 为开发者提供了一个优雅的解决方案，使他们能够利用 SQL 的力量，并在他们的 Java 代码中利用数据驱动设计原则。

面向对象因其能够通过封装数据和行为在对象内来模拟复杂系统而得到广泛采用。它强调代码组织、可重用性和模块化。然而，随着企业应用处理大量数据和复杂的数据库交互，纯面向对象的方法有时可能受到限制。

正是在这里，jOOQ 发挥了作用。jOOQ 使开发者能够无缝地将 SQL 和关系数据库操作集成到他们的 Java 代码中。它提供了一个流畅、类型安全且直观的 DSL，用于构建 SQL 查询和与数据库交互。通过拥抱数据导向的方法，jOOQ 赋予开发者直接与数据结构工作并利用 SQL 的全部力量进行查询、聚合和转换数据的能力。

使用 jOOQ，开发者可以摆脱传统 ORM 框架的限制，并对其数据库交互获得更细粒度的控制。通过拥抱数据导向的思维模式，他们可以优化性能，处理复杂的数据操作，并利用底层数据库系统提供的特性和优化。

通过使用 jOOQ，开发者可以充分利用面向对象和数据导向编程范式的优势。他们可以继续利用面向对象设计的成熟原则来封装对象内的行为，同时也能从数据导向编程的效率和灵活性中受益，以处理大量数据集和复杂的数据库操作。

在以下章节中，我们将更深入地探讨 jOOQ 的功能和特性。我们将深入研究 jOOQ 提供的用于构建 SQL 查询的 DSL，讨论其与 Java 代码的集成，并展示其在数据驱动设计中的优势。我们将共同发现 jOOQ 如何彻底改变我们与数据库的交互方式，并使面向对象编程和数据导向编程在企业应用程序中实现无缝融合。

虽然 jOOQ 提供了许多优势和好处，但也存在不可避免的权衡。以下是使用 jOOQ 可能带来的权衡之一：

+   **学习曲线**：jOOQ 引入了一种新的用于构建 SQL 查询的 DSL，这要求开发者熟悉其语法和概念。理解 jOOQ 的复杂性和有效利用它需要一定的学习曲线。

+   **代码复杂性增加**：与传统的 ORM 框架或直接 SQL 查询相比，使用 jOOQ 可能会引入额外的代码复杂性。DSL 语法以及 Java 对象与数据库记录之间的映射需求可能会导致代码量增加和潜在的复杂性，尤其是在处理复杂的数据库交互时。

+   **数据库可移植性有限**：jOOQ 根据底层数据库方言及其特定功能生成 SQL 查询。虽然 jOOQ 旨在为不同数据库提供统一的 API，但支持的特性和行为之间可能仍存在一些差异。这可能会限制代码在其他数据库系统之间的可移植性。

+   **性能考虑**：虽然 jOOQ 提供了高效的查询构建和执行，但性能可能仍然受到数据库模式设计、索引和查询优化等因素的影响。考虑 jOOQ 生成的查询的性能影响并相应优化数据库模式至关重要。

+   **维护和升级**：与任何第三方库一样，使用 jOOQ 引入了一个需要管理和维护的依赖项。跟上新版本、与不同 Java 版本的兼容性以及解决潜在的问题或错误可能需要在维护和升级期间付出额外的努力。

+   **对底层数据库的抽象有限**：与提供更高层次抽象的 ORM 框架不同，jOOQ 要求开发者深入了解 SQL 和底层数据库模式。如果你更喜欢更抽象的方法并隐藏数据库特定的细节，这可能是一个缺点。

+   **潜在的阻抗不匹配**：可能存在应用程序的面向对象特性与 jOOQ 的数据导向方法相冲突的情况。平衡这两种范式并在对象模型和数据库模式之间保持一致性可能具有挑战性，可能需要仔细的设计考虑。

虽然 jOOQ 为 Java 中的数据驱动编程提供了强大的功能，但在某些情况下可能存在更好的选择。权衡这些权衡与你的项目具体需求和限制至关重要。在决定 jOOQ 是否是你应用程序的正确工具时，考虑项目复杂性、团队经验、性能需求和数据库要求。

当我们谈论一个新的工具时，我们会将其与我们已知的东西进行比较；因此，让我们更深入地讨论 jOOQ 与 Java 持久化 API（**JPA**）之间的差异以及何时选择其中一个而不是另一个。

### JPA 与 jOOQ 的比较

jOOQ 和 JPA 都是 Java 应用程序中数据库访问的流行选择，但它们有不同的方法和用例。以下是两者的比较以及何时选择其中一个而不是另一个：

jOOQ

+   **以 SQL 为中心的方法**：jOOQ 提供了一个流畅的 DSL，允许开发者以类型安全和直观的方式构建 SQL 查询。它提供了对 SQL 语句的细粒度控制，并允许利用 SQL 的全部功能。jOOQ 非常适合需要复杂查询、数据库特定功能和性能优化的场景。

+   **数据驱动设计**：jOOQ 拥抱数据导向编程范式，使其适合处理大型数据集和复杂的数据库操作。它提供了高效的数据操作能力，并允许开发者与底层数据结构紧密工作。jOOQ 非常适合具有中心数据处理和分析的应用程序。

+   **数据库特定功能**：jOOQ 支持各种数据库特定功能和函数，使开发者能够利用不同数据库系统提供的特定功能。当与特定数据库紧密合作并使用其独特功能时，它成为一个合适的选择。

JPA

+   **ORM**：JPA 专注于将 Java 对象映射到关系数据库表，提供更高层次的抽象。它允许开发者使用持久化实体，并自动将对象映射到数据库记录。对于高度依赖面向对象设计且需要对象与数据库无缝集成的应用程序，JPA 是一个很好的选择。

+   **跨数据库可移植性**：JPA 旨在提供一个可移植的 API，可以在不同的数据库上工作。它抽象了数据库特定的细节，允许应用程序在数据库系统之间切换，而代码更改最小。当您需要关于数据库后端的灵活性并希望避免供应商锁定时，JPA 是一个合适的选择。

+   **快速应用开发**：JPA 提供了自动 CRUD 操作、缓存和事务管理等功能，简化并加速了应用开发。它提供更高层次的抽象，减少了编写低级 SQL 查询的需求。当您优先考虑快速原型设计、生产力和关注业务逻辑而非数据库特定优化时，JPA 是有益的。

在 jOOQ 和 JPA 之间进行选择取决于您特定的项目需求。如果您的应用程序是数据密集型，需要复杂的查询，并且需要精细控制 SQL，那么 jOOQ 可能是一个更好的选择。另一方面，如果您优先考虑面向对象的设计、跨不同数据库的可移植性以及快速应用开发，那么 JPA 可能更适合。还值得考虑混合方法，您可以在应用程序的不同部分同时使用 jOOQ 和 JPA，根据需要利用每个库的优势。

在介绍 jOOQ 之后，让我们将其应用到实践中，这次与 Jakarta EE 结合。本书展示了 Jakarta EE 在几个持久化框架中的应用；在本章中，我们将向您展示如何使用 jOOQ 结合 Jakarta EE。

## 使用 jOOQ 与 Jakarta/MicroProfile

在本节中，我们将探讨 jOOQ 与 Jakarta EE 和 MicroProfile 的集成，这两个 Java 生态系统中的强大框架。jOOQ 以其数据驱动的方法和以 SQL 为中心的能力，可以无缝地补充 Jakarta EE 的企业级功能和 MicroProfile 面向微服务的实践。通过结合这些技术，开发者可以解锁一个强大的工具集，用于构建健壮、可扩展且数据驱动的 Java 应用程序。

Jakarta EE，以前称为 Java EE，是一套规范和 API，为在 Java 中构建企业应用程序提供了一个标准化的平台。它提供了一系列功能，包括 servlets、**JavaServer Faces**（**JSF**）、**Enterprise JavaBeans**（**EJB**）和 JPA。开发者可以利用 Jakarta EE 的成熟生态系统和行业标准来创建可伸缩和可维护的应用程序。

另一方面，MicroProfile 是一个由社区驱动的倡议，专注于在 Java 中构建基于微服务的应用程序。它提供了一套轻量级和模块化的规范和 API，专为微服务架构量身定制。MicroProfile 允许开发者利用 JAX-RS、JSON-P 和 CDI 等技术，在微服务中实现更大的灵活性和敏捷性。

将 jOOQ 与 Jakarta EE 和 MicroProfile 结合使用，可以为您的 Java 应用程序带来两者的最佳特性。以下是这种组合的一些优势和用例：

+   **增强的数据库交互**：jOOQ 以 SQL 为中心的理念允许您直接在 Java 代码中编写复杂和优化的 SQL 查询。它使您能够高效且细致地控制数据库交互，从而实现优化的数据检索、更新和分析。将 jOOQ 与 Jakarta EE 和 MicroProfile 集成，将使您能够在企业或微服务应用程序中无缝利用 jOOQ 强大的查询构建功能。

+   **数据驱动的微服务**：架构通常需要在多个服务之间进行高效的数据访问和处理。将 jOOQ 与 MicroProfile 结合使用，允许您设计利用 jOOQ 数据驱动方法的微服务，以实现无缝的数据库集成。它使每个微服务能够独立处理其数据操作，并从 jOOQ 的 DSL 提供的性能和灵活性中受益。

+   **与 JPA 和 ORM 的集成**：Jakarta EE 应用程序通常利用 JPA 和 ORM 框架进行数据库交互。通过将 jOOQ 与 Jakarta EE 及其持久化能力集成，您可以利用 jOOQ 以 SQL 为中心的理念以及 JPA 面向对象的设计的优势。它允许您高效地处理复杂查询，并利用 JPA 的实体管理、事务和缓存功能，从而实现强大且灵活的数据访问层。

+   **横切关注点和可伸缩性**：Jakarta EE 和 MicroProfile 提供了大量针对横切关注点（如安全、日志记录和监控）的功能。通过将 jOOQ 与这些框架集成，您可以利用它们的能力来确保一致的安全策略、高效的日志记录和监控数据库交互，贯穿您的应用程序或微服务架构。

在本节中，我们将探讨实际示例，并展示如何有效地将 jOOQ 与 Jakarta EE 和 MicroProfile 结合使用。我们将展示 jOOQ 与 Jakarta EE 持久化 API 的集成，说明在 MicroProfile 微服务架构中使用 jOOQ 的方法，并讨论利用这些技术结合力量的最佳实践。

到本节结束时，你将牢固地理解如何一起使用 jOOQ、Jakarta EE 和 MicroProfile，这将使你能够构建健壮且数据驱动的 Java 应用程序，适用于企业和微服务环境。让我们深入探讨这个强大组合的可能性。

为了展示组合潜力，我们将创建一个简单的 Java SE 项目，使用 Maven，但作为一个亮点，我们可以将此代码顺利转换为微服务。该项目是一个包含单个表`Book`的 CRUD，我们将在其中执行操作，就像在一个可执行类中一样。

我们仍然会使用一个简单的数据库项目，即 H2，以降低我们项目的要求。但你可以将其在生产环境中替换为 PostgreSQL、MariaDB 等。实际上，这正是关系型数据库的美丽之处；与 NoSQL 数据库相比，我们可以更容易地在数据库之间进行切换，影响不大：

1.  让我们从 Maven 项目的配置开始，我们将包括依赖项：

    ```java
    <dependency>    <groupId>org.jboss.weld.se</groupId>    <artifactId>weld-se-shaded</artifactId>    <version>${weld.se.core.version}</version></dependency><dependency>    <groupId>io.smallrye.config</groupId>    <artifactId>smallrye-config-core</artifactId>    <version>2.13.0</version></dependency><dependency>    <groupId>org.jooq</groupId>    <artifactId>jooq</artifactId>    <version>3.18.4</version></dependency><dependency>    <groupId>com.h2database</groupId>    <artifactId>h2</artifactId>    <version>2.1.214</version></dependency><dependency>    <groupId>org.apache.commons</groupId>    <artifactId>commons-dbcp2</artifactId>    <version>2.9.0</version></dependency>
    ```

1.  在 Maven 依赖项之后，下一步是包括生成数据库结构的插件，然后基于此表创建 jOOQ。我们将开始数据结构，并使用插件执行以下查询；正如你将看到的，我们将创建模式并包含一些书籍。我们不会展示插件源代码；更多详细信息请参阅存储库源代码：

    ```java
    DROP TABLE IF EXISTS book;CREATE TABLE book (                      id INT NOT NULL,                      title VARCHAR(400) NOT NULL,                      author VARCHAR(400) NOT NULL,                      release INT,                      CONSTRAINT pk_t_book PRIMARY KEY (id));INSERT INTO book VALUES (1, 'Fundamentals of Software  Architecture', 'Neal Ford' , 2020);INSERT INTO book VALUES (2, 'Staff Engineer:  Leadership beyond the management track', 'Will    Larson' , 2021);INSERT INTO book VALUES (3, 'Building Evolutionary  Architectures', 'Neal Ford' , 2017);INSERT INTO book VALUES (4, 'Clean Code', 'Robert  Cecil Martin' , 2008);INSERT INTO book VALUES (5, 'Patterns of Enterprise  Application Architecture', 'Martin Fowler' , 2002);
    ```

1.  Maven 基础设施已经就绪，下一步是定义配置以获取数据库连接并将其提供给 CDI 上下文。我们将结合 Jakarta CDI 和 Eclipse MicroProfile Config，提取如 JDBC URL 和凭证等属性。

1.  我们将把这些凭证信息，如用户名和密码，放入`microprofile-config.properties`文件中；然而，请记住，你不应该用生产凭证这样做。我做的事情之一是通过环境覆盖这些配置。因此，开发者会在生产环境中理解这一点，而无需了解它；开发者知道这些属性，而无需理解生产属性。这是将实现推向 Twelve-Factor App（[`12factor.net`](https://12factor.net)）边缘的一个优点：

    ```java
    @ApplicationScopedclass ConnectionSupplier {    private static final Logger LOGGER = Logger      .getLogger(ConnectionSupplier.class.getName());    private static final String URL= "db.url";    private static final String USER = "db.username";    private static final String PASSWORD =      "db.password";    private static final Config CONFIG =      ConfigProvider.getConfig();    @ApplicationScoped    @Produces    public Connection get() throws SQLException {        LOGGER.fine("Starting the database          connection");        var url = CONFIG.getValue(URL, String.class);        var password =          CONFIG.getOptionalValue(PASSWORD,            String.class).orElse("");        var user = CONFIG.getValue(USER,          String.class);        return DriverManager.getConnection(          url, user, password);    }    public void close(@Disposes Connection connection)      throws SQLException {        connection.close();        LOGGER.fine("closing the database          connection");    }}
    ```

1.  CDI 可以在你的容器上下文中创建和销毁 bean 实例。我们将使用这一点来开发和关闭连接，避免在我们的应用程序中出现任何连接泄漏。一旦我们有了连接，就让我们创建`DSLContext`实例——这是我们的数据和 Java 之间的桥梁，提供了一个通过`fluent-API`的简单且安全的方式：

    ```java
    @ApplicationScopedclass ContextSupplier implements Supplier<DSLContext> {    private final Connection connection;    @Inject    ContextSupplier(Connection connection) {        this.connection = connection;    }    @Override    @Produces    public DSLContext get() {        return using(connection, SQLDialect.H2);    }}
    ```

1.  我们可以使`Connection`和`DSLContext`都可用并由 CDI 处理；下一步是使用它们来与关系数据库交互。你可以将`DSLContext`注入为一个字段，但由于我们是用 Java SE 创建的，我们将创建一个`SeContainer`并选择它，如下面的代码所示：

    ```java
    try (SeContainer container =  SeContainerInitializer.newInstance().initialize()) {    DSLContext context =      container.select(DSLContext.class).get();//...}
    ```

1.  你准备好行动了吗？让我们利用 jOOQ 执行一个无需创建实体的 CRUD 操作，它基于数据库模式，将生成我们可以工作的数据结构。操作的第一步是插入。代码显示了记录创建，我们可以设置属性并根据 setter 方法存储它们：

    ```java
    BookRecord record = context.newRecord(BOOK);record.setId(random.nextInt(0, 100));record.setRelease(2022);record.setAuthor("Otavio Santana");record.setTitle("Apache Cassandra Horizontal  scalability for Java applications");record.store();
    ```

1.  有数据，我们可以从数据库中读取这些信息；使用流畅的 API 和`select`方法以及`DSLContext`类，我们可以执行多个选择查询操作。查询将按标题顺序选择书籍。这种方法的优点是，我们大多数时候会在应用级别看到查询是否兼容，因为如果你进行任何不规则操作，它将不会编译：

    ```java
    Result<Record> books = context.select()        .from(BOOK)        .orderBy(BOOK.TITLE)        .fetch();books.forEach(book -> {    var id = book.getValue(BOOK.ID);    var author = book.getValue(BOOK.AUTHOR);    var title = book.getValue(BOOK.TITLE);    var release = book.getValue(BOOK.RELEASE);    System.out.printf("Book %s by %s has id: %d and      release: %d%n",            title, author, id, release);});
    ```

1.  最后两个步骤是`更新`和`删除`；你可以执行其他操作，探索流畅的 API 功能。我们可以定义尽可能多的参数和条件。我们正在使用的示例将设置`where`条件在`ID`值上：

    ```java
    context.update(BOOK)        .set(BOOK.TITLE, "Cassandra Horizontal          scalability for Java applications")        .where(BOOK.ID.eq(randomId))        .execute();context.delete(BOOK)        .where(BOOK.ID.eq(randomId))        .execute();
    ```

我们可以利用 jOOQ API 探索整个 CRUD 操作，而不需要创建实体。数据方法允许从模式生成结构。我们可以保证我的应用程序将能够与最后一个实体一起工作，而无需任何工作。这就结束了我们今天的 jOOQ 之旅。

# 摘要

本章深入探讨了数据驱动编程及其与面向对象方法的权衡。我们探讨了拥抱数据驱动思维的好处和挑战，理解在某些场景下，以数据为导向的方法可以提供比传统面向对象范式独特的优势。然后我们见证了 jOOQ，一个强大的 Java 库，如何弥合面向对象编程和数据驱动编程之间的差距，使开发者能够在 Java 代码中充分利用 SQL 和数据操作的全部功能。

我们还检查了 jOOQ 与 Jakarta EE 和 MicroProfile 的集成，这两个框架在开发企业级和微服务应用中广泛使用。通过结合这些技术，开发者可以同时利用 jOOQ 的数据驱动能力以及 Jakarta EE 提供的企业级功能和 MicroProfile 面向微服务的架构方法。这种集成使得高效的数据库交互、对 SQL 查询的精细控制，以及在统一架构中利用面向对象和数据导向的设计原则成为可能。

通过结合 jOOQ 所提供的数据驱动方法、Jakarta EE 和 MicroProfile 的企业级特性，以及探索 MicroStream 的开创性功能，我们可以将我们的应用程序提升到新的性能、可扩展性和效率高度。我们正站在数据库驱动应用程序开发新时代的边缘，在这里，数据的力量与执行速度相得益彰。

那么，让我们开始我们旅程的下一章，在这里我们将深入探索 MicroStream 的世界，并释放我们持久化层、Jakarta EE 和 MicroProfile 驱动的应用程序的真正潜力。随着我们拥抱这一前沿技术并见证它对我们开发过程和应用程序性能带来的变革，前方将充满激动人心的时刻。
