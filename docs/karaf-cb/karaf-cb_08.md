# 第八章。使用 Apache Cassandra 提供大数据集成层

在本章中，我们将涵盖以下主题：

+   在 Apache Karaf 中安装 Cassandra 客户端包。

+   使用 Apache Cassandra 建模数据。

+   在 Karaf 中构建具有持久化层的项目以进行部署。

# 简介

如前几章所示，持久性是大多数部署和应用程序的重要组成部分。到目前为止，我们一直专注于关系数据库。让我们从一些历史开始。 

在 1970 年，IBM 发表了一篇名为*大型共享数据银行的关系数据模型*的论文。这篇论文成为了关系数据库管理系统（RDBMS）和现代关系数据库的基础，因为它描述了实体之间的连接和关系。从这个工作出发，随后出现了 SQL（1986 年）、ACID（原子性、一致性、隔离性和持久性）、模式设计和分片以实现可扩展性。

让我们快进到社交网络的兴起；一个基于 Reed 定律的术语**WebScale**被提出，该定律指出：

> *"大型网络，尤其是社交网络，其效用可以随着网络规模的指数级增长。"*

这是否意味着 RDBMS 不能进行扩展？不，但它导致了 NoSQL 的发展。NoSQL 通常基于以下定义：

+   它最初是由 Carlo Strozzi 提出的，他在 1998 年开发了 Strozzi NoSQL 数据库。

+   它通常具有列/表中的键/值存储风格。

+   它通常是模式无关的，或者每一行可以包含不同的结构。

+   它不需要使用 SQL 作为语言；因此得名*NoSQL*。

+   许多支持 BASE 一致性。

+   大多数都是分布式和容错的。

Apache Cassandra 最初在 Facebook 开发，于 2008 年作为开源发布，2009 年在 Apache 孵化，并于 2010 年成为 Apache 顶级项目。通过快速适应和许多用例中期望的特性，Apache Cassandra 迅速获得了牵引力和广泛分布。Cassandra 的当前版本在引入**Cassandra 查询语言**（**CQL**）后，具有略微严格的模式导向，这是一种帮助推动从传统的 RDBMS 模型到更无结构的键/值对存储模型过渡的方式，同时保留了用户普遍熟悉的结构和数据模型。有关 Apache Cassandra 过渡的注释历史，请参阅[`www.datastax.com/documentation/articles/cassandra/cassandrathenandnow.html`](http://www.datastax.com/documentation/articles/cassandra/cassandrathenandnow.html)。

CQL 是 Cassandra 数据库管理系统（DBMS）的默认和主要接口。使用 CQL 与使用 SQL 相似。CQL 和 SQL 共享相同的抽象概念，即由列和行构成的表。主要区别在于 Cassandra 不支持连接或子查询，除了通过 Apache Hive 进行批量分析。相反，Cassandra 强调通过 CQL 的集合和聚类等特性在模式级别进行去规范化。

这基本上意味着还有其他客户端 API——截至 Cassandra 发布 2.x 版本，Cassandra 社区积极不建议使用它们。关于使用模式建模与列族使用的健康辩论仍然在邮件列表和用户社区中非常活跃。

# 在 Apache Karaf 中安装 Cassandra 客户端包

在我们开始探索如何构建基于 Cassandra 的应用程序之前，我们必须首先将所有必需的客户端模块安装到 Karaf 容器中。

## 准备工作

Cassandra 社区的官方 *GettingStarted* 文档可以在 [`wiki.apache.org/cassandra/GettingStarted`](http://wiki.apache.org/cassandra/GettingStarted) 找到。

这个菜谱的成分包括 Apache Karaf 分发套件、对 JDK 的访问和互联网连接。我们还假设 Apache Cassandra 数据库已下载并安装。Apache Cassandra 可以作为 RPM、Debian `.deb` 包或 `.tar` 归档下载和安装。在二进制 `.tar` 归档中，您需要打开并更改 `conf` 文件夹中的两个配置文件：`cassandra.yaml` 和 `log4-server.properties` 文件。更改涉及数据存储的位置，默认情况下数据后端存储在 `/var/lib/cassandra/*` 文件夹中，系统日志存储在 `/var/log/cassandra/*` 文件夹中。一旦完成这些更改，您可以使用 `bin/cassandra –F` 命令启动 Cassandra。

## 如何操作...

Cassandra 的驱动程序不是标准 Karaf 功能库的一部分；因此，我们或者必须编写自己的功能，或者手动安装客户端运行所需的必要包。

我们使用以下命令将 Apache Cassandra 的驱动程序和补充包安装到 Karaf 中：

```java
karaf@root()>install -s mvn:io.netty/netty/3.9.0.Final
karaf@root()>install -s mvn:com.google.guava/guava/16.0.1
karaf@root()>install -s mvn:com.codahale.metrics/metrics-core/3.0.2
karaf@root()>install -s mvn:com.datastax.cassandra/cassandra-driver-core/2.0.2

```

我们可以通过执行 `list -t 0 | grep -i cass` 命令来验证安装，该命令将列出 DataStax 驱动程序包。

通过这种方式，我们可以从自己的包中访问 Cassandra 驱动程序。

# 使用 Apache Cassandra 建模数据

在我们开始使用 Apache Cassandra 编写包之前，让我们简要看看如何使用 CQL 3.x 在 Cassandra 中建模数据。

## 准备工作

让我们定义一个非常简单的模式，因为我们使用 CQL，从客户端的角度来看，Cassandra 即使在内部数据存储略有不同的情况下也不是无模式的。

我们可以重用前一章中的 `RecipeService` 类。我们只需对其进行轻微修改以适应 Cassandra 集成。原始实体（以及通过使用 JPA）提供了一个基本的表定义，如下所示：

```java
@Id
@Column(nullable = false)
private String title;

@Column(length=10000)
private String ingredients;
```

因此，在这个表中我们有两个字段：一个名为 `title` 的 ID 字段和一个我们称为 `ingredients` 的数据字段，以保持一致性和简单性。

首先，我们需要一个地方来存储这些。Cassandra 在顶级键空间中分区数据。将键空间想象成一个包含表及其规则的映射。

## 如何操作...

我们需要执行以下两个步骤：

1.  第一步是启动 Cassandra 客户端。Cassandra 客户端的基本创建命令`cqlsh`如下所示：

    ```java
    ./bin/cqlsh
    Connected to Cluster at localhost:9160.
    [cqlsh 4.0.1 | Cassandra 2.0.1 | CQL spec 3.1.1 | Thrift protocol 19.37.0]
    Use HELP for help.

    ```

1.  下一步是创建我们的数据存储。既然我们已经启动了交互式客户端会话，我们可以创建一个键空间，如下命令所示：

    ```java
    cqlsh> CREATE KEYSPACE karaf_demo WITH replication = {'class':'SimpleStrategy', 'replication_factor':1};

    ```

    之前的命令行提示符没有返回响应，表明我们已经成功创建了一个新的键空间，我们可以在此存储数据。要使用此键空间，我们需要告诉 Cassandra 我们现在将在这里工作。

    ### 小贴士

    由于我们只有一个 Cassandra 节点，我们没有定义真实集群，也没有多个数据中心，所以我们依赖 SimpleStrategy。如果情况是这样，我们可以更改复制策略类。我们还设置了`replication_factor`值为`1`；这可以设置为更多副本，并且在安全性相关的环境中，例如存储账户信息时，这肯定应该完成。

    要切换键空间，我们发出以下`USE`命令：

    ```java
    cqlsh> USE karaf_demo;
    cqlsh:karaf_demo>

    ```

    前述命令提示符表明我们处于`karaf_demo`键空间。考虑以下命令：

    ```java
    cqlsh:karaf_demo> DESCRIBE tables;

    <empty>

    cqlsh:karaf_demo>

    ```

    如前述命令所示，我们在该键空间中未定义任何模式，因此我们需要定义一个表。可以按照以下方式完成：

    ```java
    cqlsh:karaf_demo> CREATE TABLE RECIPES (title text PRIMARY KEY,ingredients text);
    cqlsh:karaf_demo> DESCRIBE TABLES;

    recipes

    cqlsh:karaf_demo>

    ```

    现在我们已经定义了一个表，并且该表中的列被定义为存储类型`text`，主键和检索令牌为`title`。

## 它是如何工作的…

本书范围之外，不涉及 Apache Cassandra 底层工作原理的探讨。然而，您可以在[`git-wip-us.apache.org/repos/asf/cassandra.git`](http://git-wip-us.apache.org/repos/asf/cassandra.git)查看其源代码。

在我们的数据存储方面，我们可以让 Cassandra 精确描述正在存储的内容。考虑以下命令行片段：

```java
cqlsh:karaf_demo> DESCRIBE TABLE recipes;

CREATE TABLE recipes (
 title text,
 ingredients text,
 PRIMARY KEY (title)
) WITH
 bloom_filter_fp_chance=0.010000 AND
 caching='KEYS_ONLY' AND
 comment='' AND
 dclocal_read_repair_chance=0.000000 AND
 gc_grace_seconds=864000 AND
 index_interval=128 AND
 read_repair_chance=0.100000 AND
 replicate_on_write='true' AND
 populate_io_cache_on_flush='false' AND
 default_time_to_live=0 AND
 speculative_retry='NONE' AND
 memtable_flush_period_in_ms=0 AND
 compaction={'class': 'SizeTieredCompactionStrategy'} AND
 compression={'sstable_compression': 'LZ4Compressor'};

```

如您所见，作为数据模型师，您有很多选项可以操作。这些将影响复制、缓存、生命周期、压缩以及许多其他因素。

# 在 Karaf 中构建具有持久化层的项目以进行部署

应用程序开发人员通常需要在他们的项目中使用持久化层；在 Karaf 中执行此操作的一种首选方法是使用 Java Persistence API。由于我们基于现有的 JPA 项目构建，我们将尝试设置一个新的服务层并重用（复制）相同的项目模型，同时将存储后端迁移到 Apache Cassandra。这既不是完全重构也不是代码重用。技术上，我们可以将 API 部分从第七章 *使用 Apache Aries 和 OpenJPA 提供持久化层* 移动到一个新模块，然后重构章节以包含与 Cassandra 相关的依赖项和一组略有不同的导入。这实际上并不在 Cookbook 的范围内，因此保留了复制的结构。

在*在 Apache Karaf 中安装 Cassandra 客户端捆绑包*的配方中，我们学习了如何将必要的 JAR 文件安装到 Karaf 中。继续这个配方，我们将使用驱动程序构建一个简单的应用程序，使用`RecipeBookService`类将配方持久化到数据库中，这将隐藏数据存储和检索的复杂性，使其用户难以察觉。

## 准备工作

这个配方的成分包括 Apache Karaf 发行套件、对 JDK 的访问和互联网连接。这个配方的示例代码可在[`github.com/jgoodyear/ApacheKarafCookbook/tree/master/chapter8/chapter8-recipe1`](https://github.com/jgoodyear/ApacheKarafCookbook/tree/master/chapter8/chapter8-recipe1)找到。记住，为了使这些配方工作，你需要安装驱动程序并确保 Apache Cassandra 正在运行！

## 如何做到这一点……

使用 JPA 持久化层构建项目需要以下八个步骤：

1.  第一步是生成一个基于 Maven 的捆绑项目。创建一个空的基于 Maven 的项目。一个包含基本 Maven 坐标信息和捆绑打包指令的`pom.xml`文件就足够了。

1.  下一步是向 POM 文件中添加依赖项。这在上面的代码中显示：

    ```java
    <dependencies>
      <dependency>
        <groupId>org.osgi</groupId>
        <artifactId>org.osgi.core</artifactId>
        <version>5.0.0</version>
      </dependency>
      <dependency>
        <groupId>org.osgi</groupId>
        <artifactId>org.osgi.compendium</artifactId>
        <version>5.0.0</version>
      </dependency>
      <dependency>
        <groupId>org.osgi</groupId>
        <artifactId>org.osgi.enterprise</artifactId>
        <version>5.0.0</version>
      </dependency>
      <!-- Cassandra Driver -->
      <dependency>
        <groupId>com.datastax.cassandra</groupId>
        <artifactId>cassandra-driver-core</artifactId>
        <version>2.0.1</version>
      </dependency>
      <!-- custom felix gogo command -->
      <dependency>
        <groupId>org.apache.karaf.shell</groupId>
        <artifactId>org.apache.karaf.shell.console</artifactId>
        <version>3.0.0</version>
      </dependency>
    </dependencies>
    ```

    对于 Karaf 3.0.0，我们使用 OSGi 版本 5.0.0。Cassandra 驱动程序只依赖于三个外部项目。我们很幸运，所有这些都可以作为捆绑包提供——在 Karaf 的任何版本中部署都不会有很大困难。我们的大多数依赖现在都与我们的命令相关。

1.  下一步是添加构建插件。我们的配方只需要配置一个构建插件，即捆绑插件。我们配置`maven-bundle-plugin`将我们的项目代码组装成一个 OSGi 捆绑包。我们将以下插件配置添加到我们的 POM 文件中：

    ```java
    <plugin>
      <groupId>org.apache.felix</groupId>
      <artifactId>maven-bundle-plugin</artifactId>
      <version>2.4.0</version>
      <extensions>true</extensions>
      <configuration>
        <instructions>
          <Bundle-SymbolicName>
                  ${project.artifactId}
          </Bundle-SymbolicName>
          <Bundle-Activator>
                  com.packt.cassandra.demo.Activator
          </Bundle-Activator>
          <Export-Package>
                  com.packt.cassandra.demo.api.*
          </Export-Package>
          <Import-Package>
            org.osgi.service.blueprint;resolution:=optional,
            org.apache.felix.service.command,
            org.apache.felix.gogo.commands,
            org.apache.karaf.shell.console,
            *
          </Import-Package>
        </instructions>
      </configuration>
    </plugin>
    ```

    Felix 和 Karaf 的导入是可选的 Karaf 命令所必需的。

    再次强调，与 JPA 项目相比，在`pom.xml`文件中我们的复杂性更少，因为我们依赖于更少的外部资源。我们只需确保我们有了正确的 Karaf 和 Felix 导入以激活我们的命令。

1.  下一步是创建一个 Blueprint 描述符文件。在你的项目中创建目录树`src/main/resources/OSGI-INF`。然后，在这个文件夹中创建一个名为`blueprint.xml`的文件。考虑以下代码：

    ```java
    <?xml version="1.0" encoding="UTF-8" standalone="no"?>
    <blueprint default-activation="eager"
    >
      <!-- Define RecipeBookService Services, and expose them. -->
      <bean id="recipeBookService" class="com.packt.cassandra.demo.dao.RecipeBookServiceDAOImpl" init-method="init" destroy-method="destroy"/>

      <service ref="recipeBookService" interface="com.packt.cassandra.demo.api.RecipeBookService"/>

      <!-- Apache Karaf Commands -->
      <command-bundle >
        <command>
          <action class="com.packt.cassandra.demo.commands.AddRecipe">
            <property name="recipeBookService" ref="recipeBookService"/>
          </action>
        </command>
        <command>
          <action class="com.packt.cassandra.demo.commands.RemoveRecipe">
            <property name="recipeBookService" ref="recipeBookService"/>
          </action>
        </command>
        <command>
          <action class="com.packt.cassandra.demo.commands.ListRecipes">
            <property name="recipeBookService" ref="recipeBookService"/>
          </action>
        </command>
      </command-bundle>
    </blueprint>
    ```

    在前面的 Blueprint 结构中，我们最终移除了事务管理器。记住，Cassandra 的工作方式与传统的关系型数据库不同。

1.  下一步是开发一个带有新 Cassandra 后端的 OSGi 服务。我们已经创建了基本的项目结构，并配置了 Blueprint 描述符的配置。现在，我们将专注于我们 Cassandra 支持应用程序的底层 Java 代码。我们将这个过程分解为三个步骤：定义服务接口、实现服务 DAO 以及实现一个非常简单的类，我们将将其用作实体。

    在这个菜谱中，我们将在一个更广泛的项目中使用原始的 CQL 驱动程序。很可能像 DataMapper 或另一个 ORM-like 解决方案将会有所帮助。一个更高层次的库将有助于隐藏 Cassandra 和 Java 之间的类型转换，管理关系，并帮助通用数据访问。

    以下是一个简短的 CQL-friendly 库列表，作为起点：

    +   **Astyanax**：可在[`github.com/Netflix/astyanax`](https://github.com/Netflix/astyanax)找到

    +   **Spring data-cassandra**：可在[`projects.spring.io/spring-data-cassandra/`](http://projects.spring.io/spring-data-cassandra/)找到

    +   **Hecate**：可在[`github.com/savoirtech/hecate`](https://github.com/savoirtech/hecate)找到

    1.  第一步是定义服务接口。服务接口将定义我们的项目对用户的 API。在我们的示例代码中，我们实现了`RecipeBookService`类，它提供了与一系列菜谱交互所需的方法。考虑以下代码：

        ```java
        package com.packt.cassandra.demo.api;

        import java.util.Collection;
        import com.packt.jpa.demo.entity.Recipe;

        public interface RecipeBookService {

           public Collection<Recipe> getRecipes();

           public void addRecipe(String title, String ingredients);

           public void deleteRecipe(String title);

        }
        ```

        接口的实现遵循标准的 Java 约定，不需要特殊的 OSGi 包。

    1.  下一步是实现服务 DAO。现在我们已经定义了服务接口，我们将提供一个作为 DAO 的实现。虽然使用 Cassandra 不一定需要遵循 DAO 模式，但这并不是一个坏主意，因为这将使重构现有的 JPA 代码变得相对简单和可行。我们使用相同的接口，并调整实现以与 Cassandra CQL 语法一起工作。考虑以下代码：

        ```java
        public class RecipeBookServiceDAOImpl implements RecipeBookService {

          private Cluster cluster;
          private Session session;

          public void connect(String node) {
            cluster = Cluster.builder().addContactPoint(node).build();
            Metadata metadata = cluster.getMetadata();
            System.out.printf("Connected to cluster: %s\n", metadata.getClusterName());
            for (Host host : metadata.getAllHosts()) {
              System.out.printf("Datatacenter: %s; Host: %s; Rack: %s\n", host.getDatacenter(), host.getAddress(), host.getRack());
            }
            session = cluster.connect("karaf_demo");
          }

          public void destroy() {
            cluster.close();
          }

          public void init() {
            connect("127.0.0.1");
          }

          @Override
          public List<Recipe> getRecipes() {
            List<Recipe> result = new ArrayList<Recipe>();
            ResultSet results = session.execute("SELECT * FROM karaf_demo.recipes;");

            for (Row row : results) {
              Recipe recipe = new Recipe();
              recipe.setTitle(row.getString("title"));
              recipe.setIngredients(row.getString("ingredients"));
              result.add(recipe);
            }
            return result;
          }

          @Override
          public void addRecipe(String title, String ingredients) {

            ResultSet resultSet = session.execute("INSERT INTO karaf_demo.recipes (title, ingredients) VALUES ('" + title + "', '" + ingredients + "');");
            System.out.println("Result = " + resultSet);
          }

          @Override
          public void deleteRecipe(String title) {
            ResultSet resultSet = session.execute("DELETE from karaf_demo.recipes where title='" + title + "';");
            System.out.println("Result = " + resultSet);
          }
        }
        ```

        我们现在拥有的 Cassandra 类几乎是自包含的。在启动时，我们打印出一些关于我们连接的位置以及我们的集群节点所在的机架和数据中心的信息。

    1.  下一步是实现实体。在 Cassandra 的情况下，这些实体只是普通的 POJO 类。它们不再包含持久化信息。我们利用它们来符合现有的 API，并确保我们隐藏底层的 Cassandra 实现对最终用户。考虑以下代码：

        ```java
        public class Recipe {
            private String title;
            private String ingredients;
            public Recipe() {
            }
            public Recipe(String title, String ingredients) {
                super();
                this.title = title;
                this.ingredients = ingredients;
            }
            public String getTitle() {
                return title;
            }
            public void setTitle(String title) {
                this.title = title;
            }
            public String getIngredients() {
                return ingredients;
            }
            public void setIngredients(String ingredients) {
                this.ingredients = ingredients;
            }
            public String toString() {
                return "" + this.title + " " + this.ingredients;
            }
        }
        ```

        这个类现在只是一个普通的 POJO 类，我们用它来根据已经建立的 API 传输数据。对于更复杂的映射和 ORM-like 结构，有几个 DataMappers 是基于现有的 Cassandra 数据驱动程序构建的。由于它们没有像 JPA 那样标准化，因此推荐其中一个并不容易。它们都有自己的小怪癖。

1.  下一步是可选的创建 Karaf 命令以直接测试持久化服务。为了简化对`recipeBookService`实例的手动测试，我们可以创建一组自定义 Karaf 命令，这些命令将调用我们的 Cassandra 存储和检索操作。这些可选命令的示例实现可在本书的代码包中找到。特别值得注意的是它们如何获取`recipeBookService`实例的引用并调用服务。

    现在，我们必须通过 Blueprint 将命令实现连接到 Karaf，如下面的代码所示：

    ```java
    <!-- Apache Karaf Commands -->
    <command-bundle >
      <command>
        <action class="com.packt.cassandra.demo.commands.AddRecipe">
          <property name="recipeBookService" ref="recipeBookService"/>
        </action>
      </command>
      <command>
        <action class="com.packt.cassandra.demo.commands.RemoveRecipe">
          <property name="recipeBookService" ref="recipeBookService"/>
        </action>
      </command>
      <command>
        <action class="com.packt.cassandra.demo.commands.ListRecipes">
          <property name="recipeBookService" ref="recipeBookService"/>
        </action>
      </command>
    </command-bundle>
    ```

    我们每个自定义命令的实现类都连接到我们的`recipeBookService`实例。

1.  下一步是将项目部署到 Karaf。我们通过在其 Maven 坐标上执行`install`命令来安装我们的项目包，如下所示：

    ```java
    karaf@root()>  install –s mvn:com.packt/chapter8-recipe1/1.0.0-SNAPSHOT

    ```

    ### 注意

    这个演示需要一个正在运行的 Cassandra 实例！

1.  最后一步是测试项目。一旦部署了包，你将看到一些启动信息；它可能如下所示：

    ```java
    karaf@root()> Cassandra Demo Bundle stopping...
    Cassandra Demo Bundle starting...
    Connected to cluster: Cluster
    Datatacenter: datacenter1; Host: /127.0.0.1; Rack: rack1

    ```

    考虑以下命令：

    ```java
    karaf@root()> test:addrecipe "Name" "Ingredients"
    karaf@root()> test:listrecipes

    ```

    当你运行前面的命令时，你将在控制台输出中看到存储的配方！

## 它是如何工作的...

我们的数据持久层与官方 Apache Cassandra 驱动在 Karaf 容器中一起工作。提供对 Cassandra 的访问相当简单；我们只需要查看一个外部项目，并确保我们有两个依赖项，并且可以连接到现有的 Cassandra 集群。使用 Cassandra 的关键更多在于数据建模、集群的结构化、读写次数、集群的大小以及参与节点及其使用和存储模式。

我们连接的数据驱动将为我们提供集群意识、故障转移和高可用性，以及控制数据复制因子和集群行为所需的功能。官方 Cassandra 文档可以在[`cassandra.apache.org/`](http://cassandra.apache.org/)找到。

要获取 Cassandra 演变的历史背景，一个非常有用的资源可以在[`www.datastax.com/documentation/articles/cassandra/cassandrathenandnow.html`](http://www.datastax.com/documentation/articles/cassandra/cassandrathenandnow.html)找到。这将展示并解释为什么事情会改变，CQL 是如何产生的，以及为什么选择了某些设计。
