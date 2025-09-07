# 第十七章：jOOQ 中的多租户

有时，我们的应用程序需要在多租户环境中运行，即在一个操作多个租户（不同的数据库、不同的表，或者更一般地说，逻辑上隔离但物理上集成的不同实例）的环境中。在本章中，我们将根据以下议程介绍 jOOQ 在多租户环境中的常见用例：

+   通过`RenderMapping` API 连接到每个角色/登录的单独数据库

+   通过连接切换连接到每个角色/登录的单独数据库

+   为同一供应商的两个模式生成代码

+   为不同供应商的两个模式生成代码

让我们开始吧！

# 技术要求

本章的代码可以在 GitHub 上找到，地址为[`github.com/PacktPublishing/jOOQ-Masterclass/tree/master/Chapter17`](https://github.com/PacktPublishing/jOOQ-Masterclass/tree/master/Chapter17)。

# 通过`RenderMapping` API 连接到每个角色/登录的单独数据库

每个角色/登录连接到单独的数据库是多租户的经典用例。通常，你有一个主数据库（让我们称它为`development`数据库）和几个具有相同模式的其它数据库（让我们称它们为`stage`数据库和`test`数据库）。这三个数据库属于同一个供应商（在这里，MySQL）并且具有相同的模式，但它们存储着不同角色、账户、组织、合作伙伴等应用的数据。

为了简单起见，`development`数据库有一个名为`product`的单个表。这个数据库用于生成 jOOQ 工件，但我们希望根据当前角色（当前登录用户）在`stage`或`test`数据库上执行查询。

这种实现的关键在于对 jOOQ `RenderMapping` API 的巧妙运用。jOOQ 允许我们在运行时指定输入模式（例如，`development`）和输出模式（例如，`stage`），在查询中，它将渲染输出模式。代码的高潮在于这些设置，正如你所见（认证特定于 Spring Security API）：

```java
Authentication auth = SecurityContextHolder
```

```java
  .getContext().getAuthentication();
```

```java
String authority = auth.getAuthorities().iterator()
```

```java
  .next().getAuthority();
```

```java
String database = authority.substring(5).toLowerCase();
```

```java
ctx.configuration().derive(new Settings()
```

```java
    .withRenderMapping(new RenderMapping()
```

```java
      .withSchemata(
```

```java
        new MappedSchema().withInput("development")
```

```java
                          .withOutput(database)))).dsl()
```

```java
   .insertInto(PRODUCT, PRODUCT.PRODUCT_NAME, 
```

```java
               PRODUCT.QUANTITY_IN_STOCK)
```

```java
   .values("Product", 100)
```

```java
   .execute();
```

根据当前认证用户的角色，jOOQ 渲染预期的数据库名称（例如，`` `stage`.`product` ``或`` `test`.`product` ``）。基本上，每个用户都有一个角色（例如，`ROLE_STAGE`或`ROLE_TEST`；为了简单起见，用户只有一个角色），我们通过移除`ROLE_`并将剩余文本转换为小写来提取输出数据库名称；按照惯例，提取的文本代表数据库名称。当然，你可以使用用户名、组织名称或任何适合你情况的约定。

你可以在名为*MT*的 MySQL 应用程序中测试这个示例。

`withInput()`方法接受输入模式的完整名称。如果你想将输入模式的名称与正则表达式匹配，那么不是使用`withInput()`，而是使用`withInputExpression(Pattern.compile("reg_exp"))`（例如，`("development_(.*)")`）。

如果你在一个支持目录的数据库中（例如，SQL Server），那么只需使用`MappedCatalog()`和`withCatalogs()`，如下面的示例所示：

```java
String catalog = …;
```

```java
Settings settings = new Settings()
```

```java
    .withRenderMapping(new RenderMapping()
```

```java
    .withCatalogs(new MappedCatalog()
```

```java
    .withInput("development")
```

```java
    .withOutput(catalog))
```

```java
    .withSchemata(…); // optional, if you need schema as well
```

如果你不需要运行时模式，而是需要在代码生成时硬编码映射（jOOQ 始终在运行时渲染，符合这些设置），那么对于 Maven，使用以下内容：

```java
<database>
```

```java
  <schemata>
```

```java
    <schema>
```

```java
     <inputSchema>…</inputSchema>
```

```java
     <outputSchema>…</outputSchema>
```

```java
    </schema>
```

```java
  </schemata>
```

```java
</database>
```

对于 Gradle，使用以下内容：

```java
database {
```

```java
  schemata {
```

```java
    schema {
```

```java
     inputSchema = '…'
```

```java
     outputSchema = '…'
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

使用以下内容进行程序化操作：

```java
new org.jooq.meta.jaxb.Configuration()
```

```java
  .withGenerator(new Generator()
```

```java
    .withDatabase(new Database()
```

```java
      .withSchemata(
```

```java
        new SchemaMappingType()
```

```java
          .withInputSchema("...")
```

```java
          .withOutputSchema("...")
```

```java
      )
```

```java
    )
```

```java
  )
```

你可以在*MTM*中看到这样的示例，用于 MySQL。正如你所看到的，所有账户/角色都针对在代码生成时硬编码的数据库（`stage`数据库）进行操作。

如果你使用的是支持目录的数据库（例如，SQL Server），那么只需简单地依赖`<catalogs>`、`<catalog>`、`<inputCatalog>`和`<outputCatalog>`。对于 Maven，使用以下内容：

```java
<database>
```

```java
  <catalogs>
```

```java
    <catalog>          
```

```java
     <inputCatalog>…</inputCatalog>    
```

```java
     <outputCatalog>…</outputCatalog>
```

```java
     <!-- Optionally, if you need schema mapping -->
```

```java
     <schemata>
```

```java
     </schemata>
```

```java
   </catalog>
```

```java
 </catalogs>
```

```java
</database>
```

对于 Gradle，使用以下内容：

```java
database {
```

```java
  catalogs {
```

```java
   catalog {
```

```java
    inputCatalog = '…'
```

```java
    outputCatalog = '…'
```

```java
    // Optionally, if you need schema mapping
```

```java
    schemata {}
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

对于程序化操作，使用以下内容：

```java
new org.jooq.meta.jaxb.Configuration()
```

```java
  .withGenerator(new Generator()
```

```java
    .withDatabase(new Database()
```

```java
      .withCatalogs(
```

```java
        new CatalogMappingType()
```

```java
          .withInputCatalog("...")
```

```java
          .withOutputCatalog("...")
```

```java
          // Optionally, if you need schema mapping
```

```java
          .withSchemata()
```

```java
      )
```

```java
    )
```

```java
  )
```

到目前为止，`development`数据库有一个名为`product`的单个表。这个表在`stage`和`test`数据库中也有相同的名称，但让我们假设我们决定在`development`数据库中将其称为`product_dev`，在`stage`数据库中称为`product_stage`，在`test`数据库中称为`product_test`。在这种情况下，即使 jOOQ 正确渲染了每个角色的数据库名称，它也没有正确渲染表名。幸运的是，jOOQ 允许我们通过`withTables()`和`MappedTable()`配置这个方面，如下所示：

```java
ctx.configuration().derive(new Settings()
```

```java
    .withRenderMapping(new RenderMapping()
```

```java
      .withSchemata(
```

```java
        new MappedSchema().withInput("development")
```

```java
                          .withOutput(database)
```

```java
      .withTables(
```

```java
        new MappedTable().withInput("product_dev")
```

```java
           .withOutput("product_" + database))))).dsl()
```

```java
   .insertInto(PRODUCT_DEV, PRODUCT_DEV.PRODUCT_NAME,  
```

```java
               PRODUCT_DEV.QUANTITY_IN_STOCK)
```

```java
   .values("Product", 100)
```

```java
   .execute();
```

你可以在名为*MTT*的应用程序中查看这个示例，用于 MySQL。

# 通过连接切换每个角色/登录的独立数据库

另一个快速连接到每个角色/登录的独立数据库的解决方案是在运行时切换到适当的连接。为了完成这个任务，我们必须抑制 jOOQ 默认渲染模式/目录名称的行为。这样，我们就不必担心连接到数据库`A`，而是让数据库`B`在我们的表前显示，依此类推。换句话说，我们需要无限定名称。

jOOQ 允许我们通过`withRenderSchema(false)`和`withRenderCatalog(false)`设置关闭渲染模式/目录名称。以下示例连接到与登录用户角色同名的数据库，并抑制渲染模式/目录名称：

```java
Authentication auth = SecurityContextHolder
```

```java
     .getContext().getAuthentication();
```

```java
if (auth != null && auth.isAuthenticated()) {
```

```java
   String authority = auth.getAuthorities()
```

```java
     .iterator().next().getAuthority();
```

```java
   String database = authority.substring(5).toLowerCase();
```

```java
   DSL.using(
```

```java
    "jdbc:mysql://localhost:3306/" + database, 
```

```java
        "root", "root")
```

```java
      .configuration().derive(new Settings()
```

```java
         .withRenderCatalog(Boolean.FALSE)
```

```java
         .withRenderSchema(Boolean.FALSE))
```

```java
         .dsl()
```

```java
      .insertInto(PRODUCT, PRODUCT.PRODUCT_NAME, 
```

```java
           PRODUCT.QUANTITY_IN_STOCK)
```

```java
      .values("Product", 100)
```

```java
      .execute();
```

```java
}
```

你可以在名为*MTC*的应用程序中查看这个示例，用于 MySQL。

或者，我们可以通过`outputSchemaToDefault`标志来指示 jOOQ 从生成的代码中移除任何模式引用。对于 Maven，使用以下内容：

```java
<outputSchemaToDefault>true</outputSchemaToDefault>
```

对于 Gradle，使用以下内容：

```java
outputSchemaToDefault = true
```

由于生成的代码中没有更多的模式引用，生成的类可以在所有你的模式上运行：

```java
String database = …;
```

```java
DSL.using(
```

```java
  "jdbc:mysql://localhost:3306/" + database, 
```

```java
      "root", "root")  
```

```java
  .insertInto(PRODUCT, PRODUCT.PRODUCT_NAME, 
```

```java
              PRODUCT.QUANTITY_IN_STOCK)
```

```java
  .values("Product", 100)
```

```java
  .execute();
```

你可以在名为*MTCO*的应用程序中测试这个示例，用于 MySQL。

# 为同一供应商的两个架构方案生成代码

考虑两个名为`db1`和`db2`的同一供应商的架构方案。在第一个架构（`db1`）中，我们有一个名为`productline`的表，在第二个架构（`db2`）中，我们有一个名为`product`的表。我们的目标是生成这两个相同供应商架构的 jOOQ 工件（以运行 jOOQ 代码生成器），并对其中一个或另一个执行查询，甚至连接这两个表。

基本上，只要我们不指定任何输入架构，jOOQ 就会为它找到的所有架构生成代码。但因为我们想指示 jOOQ 只在工作在`db1`和`db2`架构上，我们可以这样做（这里针对 Maven）：

```java
<database>
```

```java
 <schemata>
```

```java
   <schema>
```

```java
    <inputSchema>db1</inputSchema>
```

```java
   </schema>
```

```java
   <schema>
```

```java
    <inputSchema>db2</inputSchema>
```

```java
   </schema>
```

```java
 </schemata>
```

```java
</database>
```

我相信您现在有足够的经验来直观地了解如何为 Gradle 或程序编写这些代码，所以我会跳过这些示例。

一旦我们运行了 jOOQ 代码生成器，我们就可以执行查询，如下所示：

```java
ctx.select().from(DB1.PRODUCTLINE).fetch();
```

```java
ctx.select().from(DB2.PRODUCT).fetch();
```

或者，这里是一个`PRODUCTLINE`和`PRODUCT`的连接：

```java
ctx.select(DB1.PRODUCTLINE.PRODUCT_LINE,
```

```java
   DB2.PRODUCT.PRODUCT_ID, DB2.PRODUCT.PRODUCT_NAME,   
```

```java
   DB2.PRODUCT.QUANTITY_IN_STOCK)
```

```java
   .from(DB1.PRODUCTLINE)
```

```java
   .join(DB2.PRODUCT)
```

```java
   .on(DB1.PRODUCTLINE.PRODUCT_LINE
```

```java
     .eq(DB2.PRODUCT.PRODUCT_LINE))
```

```java
   .fetch();
```

`DB1`和`DB2`被静态导入，如下所示：

```java
import static jooq.generated.db1.Db1.DB1;
```

```java
import static jooq.generated.db2.Db2.DB2;
```

完整的示例可在名为*MTJ*的应用程序中找到，适用于 MySQL。

# 为两个不同供应商的架构方案生成代码

考虑两个不同供应商的架构方案——例如，我们为 MySQL 和 PostgreSQL 设计的`classicmodels`架构。我们的目标是生成这两个架构的 jOOQ 工件，并对其中一个或另一个执行查询。

考虑一个基于 Maven 的应用程序，我们可以通过使用两个`<execution>`条目来完成这个任务，一个是`flyway-maven-plugin`插件，另一个是`jooq-codegen-maven`插件。以下是`jooq-codegen-maven`的骨架代码（完整的代码可在捆绑的代码中找到）：

```java
<plugin>
```

```java
  <groupId>org.jooq</groupId>
```

```java
  <artifactId>jooq-codegen-maven</artifactId> 
```

```java
  <executions>
```

```java
    <execution>
```

```java
      <id>generate-mysql</id>
```

```java
      <phase>generate-sources</phase>
```

```java
      <goals>
```

```java
        <goal>generate</goal>
```

```java
      </goals>                                  
```

```java
      <configuration xmlns="... jooq-codegen-3.16.0.xsd">     
```

```java
        ... <!-- MySQL schema configuration -->
```

```java
      </configuration>
```

```java
    </execution>
```

```java
    <execution>
```

```java
      <id>generate-postgresql</id>
```

```java
      <phase>generate-sources</phase>
```

```java
      <goals>
```

```java
        <goal>generate</goal>
```

```java
      </goals>                    
```

```java
      <configuration xmlns="...jooq-codegen-3.16.0.xsd">   
```

```java
        ... <!-- PostgreSQL schema configuration -->
```

```java
      </configuration>
```

```java
    </execution>
```

```java
  </executions> 
```

```java
</plugin>
```

接下来，jOOQ 为这两个供应商生成工件，我们可以在这两个连接和表之间切换，如下所示：

```java
DSL.using(
```

```java
 "jdbc:mysql://localhost:3306/classicmodels", 
```

```java
    "root", "root")
```

```java
   .select().from(mysql.jooq.generated.tables.Product.PRODUCT)
```

```java
   .fetch();
```

```java
DSL.using(
```

```java
 "jdbc:postgresql://localhost:5432/classicmodels", 
```

```java
         "postgres", "root")
```

```java
   .select().from(
```

```java
           postgresql.jooq.generated.tables.Product.PRODUCT)
```

```java
   .fetch();
```

或者，考虑到我们已经程序化配置了我们的`DataSource`对象，我们也可以配置两个`DSLContext`（完整的代码可在捆绑的代码中找到）：

```java
@Bean(name="mysqlDSLContext")
```

```java
public DSLContext mysqlDSLContext(@Qualifier("configMySql") 
```

```java
         DataSourceProperties properties) {
```

```java
  return DSL.using(
```

```java
    properties.getUrl(), properties.getUsername(), 
```

```java
    properties.getPassword());
```

```java
}
```

```java
@Bean(name="postgresqlDSLContext")
```

```java
public DSLContext postgresqlDSLContext(
```

```java
    @Qualifier("configPostgreSql") 
```

```java
       DataSourceProperties properties) {
```

```java
  return DSL.using(
```

```java
    properties.getUrl(), properties.getUsername(), 
```

```java
    properties.getPassword());
```

```java
}
```

您还可以注入这两个`DSLContext`并使用您想要的任何一个：

```java
private final DSLContext mysqlCtx;
```

```java
private final DSLContext postgresqlCtx;
```

```java
public ClassicModelsRepository(
```

```java
 @Qualifier("mysqlDSLContext") DSLContext mysqlCtx, 
```

```java
 @Qualifier("postgresqlDSLContext") DSLContext postgresqlCtx){
```

```java
     this.mysqlCtx = mysqlCtx;
```

```java
     this.postgresqlCtx = postgresqlCtx;
```

```java
}
```

```java
…
```

```java
mysqlCtx.select().from(
```

```java
  mysql.jooq.generated.tables.Product.PRODUCT).fetch();
```

```java
postgresqlCtx.select().from(
```

```java
  postgresql.jooq.generated.tables.Product.PRODUCT).fetch();
```

完整的代码命名为*MT2DB*。如果您只想根据活动配置文件生成一个供应商的工件，那么您会喜欢*MP*应用程序。

# 摘要

多租户不是一个常规任务，但了解 jOOQ 非常灵活，允许我们在几秒钟内配置多个数据库/架构是很好的。此外，正如您刚才看到的，jOOQ + Spring Boot 组合是完成多租户任务的完美匹配。

在下一章中，我们将讨论 jOOQ SPI。
