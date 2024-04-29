# 数据库编程

本章涵盖了 Java 应用程序与**数据库**（**DB**）之间的基本和常用交互，从连接到数据库和执行 CRUD 操作到创建事务、存储过程和处理**大对象**（**LOBs**）。我们将涵盖以下内容：

+   使用 JDBC 连接到数据库

+   设置所需的用于数据库交互的表。

+   使用 JDBC 执行 CRUD 操作

+   使用**Hikari 连接池**（**HikariCP**）

+   使用预编译语句

+   使用事务

+   处理大对象

+   执行存储过程

+   使用批处理操作处理大量数据

+   使用**MyBatis**进行 CRUD 操作

+   使用**Java 持久性 API**和**Hibernate**

# 介绍

很难想象一个不使用某种结构化和可访问的数据存储（称为数据库）的复杂软件应用程序。这就是为什么任何现代语言实现都包括一个允许您访问数据库并在其中**创建、读取、更新和删除**（**CRUD**）数据的框架。在 Java 中，**Java 数据库连接**（**JDBC**）API 提供对任何数据源的访问，从关系数据库到电子表格和平面文件。

基于这种访问，应用程序可以直接在数据库中操作数据，使用数据库语言（例如 SQL），或间接使用**对象关系映射（ORM）**框架，该框架允许将内存中的对象映射到数据库中的表。**Java 持久性 API（JPA）**是 Java 的 ORM 规范。当使用 ORM 框架时，对映射的 Java 对象的 CRUD 操作会自动转换为数据库语言。最受欢迎的 ORM 框架列表包括 Apache Cayenne、Apache OpenJPA、EclipseLink、jOOQ、MyBatis 和 Hibernate 等。

组成 JDBC API 的`java.sql`和`javax.sql`包包含在**Java 平台标准版**（**Java SE**）中。`java.sql`包提供了访问和处理存储在数据源中的数据（通常是关系数据库）的 API。`javax.sql`包提供了用于服务器端数据源访问和处理的 API。具体来说，它提供了用于与数据库建立连接、连接和语句池、分布式事务和行集的`DataSource`接口。包含符合 JPA 的接口的`javax.persistence`包未包含在 Java SE 中，必须将其作为依赖项添加到 Maven 配置文件`pom.xml`中。特定的 JPA 实现——首选的 ORM 框架——也必须作为 Maven 依赖项包含在内。我们将在本章的示例中讨论 JDBC、JPA 和两个 ORM 框架——Hibernate 和 MyBatis 的使用。

要实际将`DataSource`连接到物理数据库，还需要一个特定于数据库的驱动程序（由数据库供应商提供，例如 MySQL、Oracle、PostgreSQL 或 SQL Server 数据库）。它可能是用 Java 编写的，也可能是 Java 和**Java 本机接口**（**JNI**）本机方法的混合体。这个驱动程序实现了 JDBC API。

与数据库的工作涉及八个步骤：

1.  按照供应商的说明安装数据库。

1.  向应用程序添加特定于数据库的驱动程序的`.jar`依赖项。

1.  创建用户、数据库和数据库模式——表、视图、存储过程等。

1.  从应用程序连接到数据库。

1.  直接使用 JDBC 或间接使用 JPA 构建 SQL 语句。

1.  直接使用 JDBC 执行 SQL 语句或使用 JPA 提交数据更改。

1.  使用执行结果。

1.  关闭数据库连接和其他资源。

步骤 1-3 仅在数据库设置阶段执行一次，应用程序运行之前。

步骤 4-8 根据需要由应用程序重复执行。

步骤 5-7 可以重复多次使用相同的数据库连接。

# 使用 JDBC 连接到数据库

在这个示例中，您将学习如何连接到数据库。

# 如何做...

1.  选择您想要使用的数据库。有很好的商业数据库和很好的开源数据库。我们唯一假设的是您选择的数据库支持**结构化查询语言**（**SQL**），这是一种标准化的语言，允许您在数据库上执行 CRUD 操作。在我们的示例中，我们将使用标准 SQL，并避免特定于特定数据库类型的构造和过程。

1.  如果数据库尚未安装，请按照供应商的说明进行安装。然后，下载数据库驱动程序。最流行的是类型 4 和 5，用 Java 编写。它们非常高效，并通过套接字连接与数据库服务器通信。如果将带有此类驱动程序的 `.jar` 文件放置在类路径上，它将自动加载。类型 4 和 5 的驱动程序是特定于数据库的，因为它们使用数据库本机协议来访问数据库。我们将假设您正在使用此类驱动程序。

如果您的应用程序必须访问多种类型的数据库，则需要类型 3 的驱动程序。这样的驱动程序可以通过中间件应用服务器与不同的数据库通信。

仅当没有其他驱动程序类型可用于您的数据库时，才使用类型 1 和 2 的驱动程序。

1.  将下载的带有驱动程序的 `.jar` 文件设置在应用程序的类路径上。现在，您的应用程序可以访问数据库。

1.  您的数据库可能有控制台、图形用户界面或其他与之交互的方式。阅读说明，并首先创建一个用户，即 `cook`，然后创建一个名为 `cookbook` 的数据库。

例如，以下是 PostgreSQL 的命令：

```java
 CREATE USER cook SUPERUSER;
 CREATE DATABASE cookbook OWNER cook;
```

我们为我们的用户选择了 `SUPERUSER` 角色；然而，一个良好的安全实践是将这样一个强大的角色分配给管理员，并创建另一个专门用于管理数据但不能更改数据库结构的应用程序特定用户。创建另一个逻辑层，称为 **模式**，可以拥有自己的一组用户和权限，这是一个良好的实践。这样，同一数据库中的几个模式可以被隔离，每个用户（其中一个是你的应用程序）只能访问特定的模式。

1.  此外，在企业级别，通常的做法是为数据库模式创建同义词，以便没有应用程序可以直接访问原始结构。您甚至可以为每个用户创建一个密码，但是，再次强调，出于本书的目的，这是不需要的。因此，我们将其留给数据库管理员来制定适合每个企业特定工作条件的规则和指南。

现在，我们将应用程序连接到数据库。在以下演示代码中，我们将使用开源的 PostgreSQL 数据库，你可能已经猜到了。

# 它是如何工作的...

以下是创建到本地 PostgreSQL 数据库的连接的代码片段：

```java
String URL = "jdbc:postgresql://localhost/cookbook";
Properties prop = new Properties( );
//prop.put( "user", "cook" );
//prop.put( "password", "secretPass123" );
Connection conn = DriverManager.getConnection(URL, prop);
```

注释行显示了如何为连接设置用户和密码。由于在此演示中，我们将数据库保持开放并对任何人都可访问，我们可以使用重载的 `DriverManager.getConnection(String url)` 方法。然而，我们将展示最常见的实现，允许任何人从属性文件中读取并传递其他有用的值（`ssl` 为 true/false，`autoReconnect` 为 true/false，`connectTimeout` 以秒为单位等）到创建连接的方法。传入属性的许多键对于所有主要数据库类型都是相同的，但其中一些是特定于数据库的。

或者，仅传递用户和密码，我们可以使用第三个重载版本，即 `DriverManager.getConnection(String url, String user, String password)`。值得一提的是，将密码加密是一个良好的实践。我们不会向您展示如何做到这一点，但在线上有很多指南可用。

此外，`getConnection()`方法会抛出`SQLException`，因此我们需要将其包装在`try...catch`块中。

为了隐藏所有这些管道，最好将建立连接的代码放在一个方法中：

```java
Connection getDbConnection(){
  String url = "jdbc:postgresql://localhost/cookbook";
  try {
    return DriverManager.getConnection(url);
  }
  catch(Exception ex) {
    ex.printStackTrace();
    return null;
  }
}
```

连接到数据库的另一种方法是使用`DataSource`接口。它的实现通常包含在与数据库驱动程序相同的`.jar`文件中。在 PostgreSQL 的情况下，有两个类实现了`DataSource`接口：`org.postgresql.ds.PGSimpleDataSource`和`org.postgresql.ds.PGPoolingDataSource`。我们可以使用它们来代替`DriverManager`。以下是使用`PGSimpleDataSource`的示例：

```java
Connection getDbConnection(){
  PGSimpleDataSource source = new PGSimpleDataSource();
  source.setServerName("localhost");
  source.setDatabaseName("cookbook");
  source.setLoginTimeout(10);
  try {
    return source.getConnection();
  }
  catch(Exception ex) {
    ex.printStackTrace();
    return null;
  }
}

```

以下是使用`PGPoolingDataSource`的示例：

```java
Connection getDbConnection(){
  PGPoolingDataSource source = new PGPoolingDataSource();
  source.setServerName("localhost");
  source.setDatabaseName("cookbook");
  source.setInitialConnections(3);
  source.setMaxConnections(10);
  source.setLoginTimeout(10);
  try {
    return source.getConnection();
  }
  catch(Exception ex) {
    ex.printStackTrace();
    return null;
  }
}
```

`getDbConnection()`方法的最新版本通常是首选的连接方式，因为它允许您使用连接池和其他一些功能，除了通过`DriverManager`连接时可用的功能。请注意，自版本`42.0.0`起，`PGPoolingDataSource`类已被弃用，而是支持第三方连接池软件。其中一个框架，HikariCP，我们之前提到过，将在*使用 Hikari 连接池*的示例中进行讨论和演示。

无论您选择哪个版本的`getDbConnection()`实现，您都可以在所有代码示例中以相同的方式使用它。

# 还有更多...

关闭连接是一个好习惯，一旦不再需要它，就立即关闭连接。这样做的方法是使用`try-with-resources`构造，它确保在`try...catch`块结束时关闭资源：

```java
try (Connection conn = getDbConnection()) {
  // code that uses the connection to access the DB
} 
catch(Exception ex) { 
  ex.printStackTrace();
}
```

这样的构造可以与实现`java.lang.AutoCloseable`或`java.io.Closeable`接口的任何对象一起使用。

# 设置所需的用于数据库交互的表

在这个示例中，您将学习如何创建、更改和删除表和其他组成数据库模式的逻辑数据库构造。

# 准备工作

用于创建表的标准 SQL 语句如下：

```java
CREATE TABLE table_name (
  column1_name data_type(size),
  column2_name data_type(size),
  column3_name data_type(size),
  ....
);
```

在这里，`table_name`和`column_name`必须是字母数字和唯一的（在模式内）。名称和可能的数据类型的限制是特定于数据库的。例如，Oracle 允许表名有 128 个字符，而在 PostgreSQL 中，表名和列名的最大长度为 63 个字符。数据类型也有所不同，因此请阅读数据库文档。

# 它是如何工作的...

以下是在 PostgreSQL 中创建`traffic_unit`表的命令示例：

```java
CREATE TABLE traffic_unit (
  id SERIAL PRIMARY KEY,
  vehicle_type VARCHAR NOT NULL,
  horse_power integer NOT NULL,
  weight_pounds integer NOT NULL,
  payload_pounds integer NOT NULL,
  passengers_count integer NOT NULL,
  speed_limit_mph double precision NOT NULL,
  traction double precision NOT NULL,
  road_condition VARCHAR NOT NULL,
  tire_condition VARCHAR NOT NULL,
  temperature integer NOT NULL
);
```

`size`参数是可选的。如果不设置，就像前面的示例代码一样，这意味着该列可以存储任意长度的值。在这种情况下，`integer`类型允许您存储从`Integer.MIN_VALUE`（-2147483648）到`Integer.MAX_VALUE`（+2147483647）的数字。添加了`NOT NULL`类型，因为默认情况下，该列将是可空的，我们希望确保所有列都将被填充。

我们还将`id`列标识为`PRIMARY KEY`，这表示该列（或列的组合）唯一标识记录。例如，如果有一张包含所有国家所有人的信息的表，唯一的组合可能是他们的全名、地址和出生日期。嗯，可以想象在一些家庭中，双胞胎出生并被赋予相同的名字，所以我们说*可能*。如果这种情况发生的机会很大，我们需要向主键组合中添加另一列，即出生顺序，默认值为`1`。以下是我们在 PostgreSQL 中可以这样做的方法：

```java
CREATE TABLE person (
  name VARCHAR NOT NULL,
  address VARCHAR NOT NULL,
  dob date NOT NULL,
  order integer DEFAULT 1 NOT NULL,
  PRIMARY KEY (name,address,dob,order)
);
```

在`traffic_unit`表的情况下，没有任何组合的列可以作为主键。许多汽车在任何组合的列中具有相同的值。但我们需要引用`traffic_unit`记录，这样我们就可以知道，例如，哪些单位已被选择和处理，哪些没有。这就是为什么我们添加了一个`id`列，用一个唯一生成的数字填充它，我们希望数据库能够自动生成这个主键。

如果您发出命令`\d traffic_unit`来显示表描述，您将看到`id`列分配了函数`nextval('traffic_unit_id_seq'::regclass)`。此函数按顺序生成数字，从 1 开始。如果您需要不同的行为，可以手动创建序列号生成器。以下是如何做到这一点的示例：

```java
CREATE SEQUENCE traffic_unit_id_seq 
START WITH 1000 INCREMENT BY 1 
NO MINVALUE NO MAXVALUE CACHE 10; 
ALTER TABLE ONLY traffic_unit ALTER COLUMN id SET DEFAULT nextval('traffic_unit_id_seq'::regclass);
```

这个序列从 1,000 开始，缓存 10 个数字以提高性能，如果需要快速生成数字。

根据前几章中给出的代码示例，`vehicle_type`、`road_condition`和`tire_condition`的值受`enum`类型的限制。这就是为什么当`traffic_unit`表被填充时，我们希望确保只有相应的`enum`类型的值可以设置在列中。为了实现这一点，我们将创建一个名为`enums`的查找表，并用我们`enum`类型的值填充它：

```java
CREATE TABLE enums (
  id integer PRIMARY KEY,
  type VARCHAR NOT NULL,
  value VARCHAR NOT NULL
);

insert into enums (id, type, value) values 
(1, 'vehicle', 'car'),
(2, 'vehicle', 'truck'),
(3, 'vehicle', 'crewcab'),
(4, 'road_condition', 'dry'),
(5, 'road_condition', 'wet'),
(6, 'road_condition', 'snow'),
(7, 'tire_condition', 'new'),
(8, 'tire_condition', 'worn');
```

PostgreSQL 有一个`enum`数据类型，但如果可能值的列表不固定并且需要随时间更改，它会产生额外的开销。我们认为我们的应用程序中可能会扩展值的列表。因此，我们决定不使用数据库的`enum`类型，而是自己创建查找表。

现在，我们可以通过使用它们的 ID 作为外键，从`traffic_unit`表中引用`enums`表的值。首先，我们删除表：

```java
drop table traffic_unit;
```

然后，我们用稍微不同的 SQL 命令重新创建它：

```java
CREATE TABLE traffic_unit (
  id SERIAL PRIMARY KEY,
  vehicle_type integer REFERENCES enums (id),
  horse_power integer NOT NULL,
  weight_pounds integer NOT NULL,
  payload_pounds integer NOT NULL,
  passengers_count integer NOT NULL,
  speed_limit_mph double precision NOT NULL,
  traction double precision NOT NULL,
  road_condition integer REFERENCES enums (id),
  tire_condition integer REFERENCES enums (id),
  temperature integer NOT NULL
);
```

`vehicle_type`、`road_condition`和`tire_condition`列现在必须由`enums`表相应记录的主键值填充。这样，我们可以确保我们的交通分析代码能够将这些列的值与代码中的`enum`类型的值匹配。

# 还有更多...

我们还希望确保`enums`表不包含重复的组合类型和值。为了确保这一点，我们可以向`enums`表添加唯一约束：

```java
ALTER TABLE enums ADD CONSTRAINT enums_unique_type_value 
UNIQUE (type, value);
```

现在，如果我们尝试添加重复项，数据库将不允许。

数据库表创建的另一个重要考虑因素是是否需要添加索引。*索引*是一种数据结构，可以加速在表中进行数据搜索，而无需检查每个表记录。它可以包括一个或多个表的列。例如，主键的索引会自动创建。如果您打开我们已经创建的表的描述，您会看到以下内容：

```java
 Indexes: "traffic_unit_pkey" PRIMARY KEY, btree (id)
```

如果我们认为（并通过实验已经证明）添加索引将有助于应用程序的性能，我们也可以自己添加索引。在`traffic_unit`的情况下，我们发现我们的代码经常通过`vehicle_type`和`passengers_count`搜索这个表。因此，我们测量了我们的代码在搜索过程中的性能，并将这两列添加到了索引中：

```java
CREATE INDEX idx_traffic_unit_vehicle_type_passengers_count 
ON traffic_unit USING btree (vehicle_type,passengers_count);
```

然后，我们再次测量性能。如果性能有所改善，我们将保留索引，但在我们的情况下，我们已经删除了它：

```java
drop index idx_traffic_unit_vehicle_type_passengers_count;
```

索引并没有显著改善性能，可能是因为索引会增加额外的写入和存储空间的开销。

在我们的主键、约束和索引示例中，我们遵循了 PostgreSQL 的命名约定。如果您使用不同的数据库，我们建议您查找其命名约定并遵循，以便您的命名与自动创建的名称保持一致。

# 使用 JDBC 执行 CRUD 操作

在这个示例中，您将学习如何使用 JDBC 在表中填充、读取、更改和删除数据。

# 准备工作

我们已经看到了在数据库中创建（填充）数据的 SQL 语句示例：

```java
INSERT INTO table_name (column1,column2,column3,...)
VALUES (value1,value2,value3,...);
```

我们还看到了需要添加多个表记录的示例：

```java
INSERT INTO table_name (column1,column2,column3,...)
VALUES (value1,value2,value3, ... ), 
       (value21,value22,value23, ...), 
       ( ...                       );
```

如果列有指定的默认值，则无需在 `INSERT INTO` 语句中列出它，除非需要插入不同的值。

通过 `SELECT` 语句从数据库中读取数据：

```java
SELECT column_name,column_name
FROM table_name WHERE some_column=some_value;
```

以下是 `WHERE` 子句的一般定义：

```java
WHERE column_name operator value
Operator:
  = Equal
  <> Not equal. In some versions of SQL, !=
  > Greater than
  < Less than
  >= Greater than or equal
  <= Less than or equal
  BETWEEN Between an inclusive range
  LIKE Search for a pattern
  IN To specify multiple possible values for a column
```

`column_name operator value` 结构可以与逻辑运算符 `AND` 和 `OR` 结合，并用括号 `(` 和 `)` 进行分组。

所选的值可以按特定顺序返回：

```java
SELECT * FROM table_name WHERE-clause by some_other_column; 
```

顺序可以标记为升序（默认）或降序：

```java
SELECT * FROM table_name WHERE-clause by some_other_column desc; 
```

数据可以使用 `UPDATE` 语句进行更改：

```java
UPDATE table_name SET column1=value1,column2=value2,... WHERE-clause;
```

或者，可以使用 `DELETE` 语句进行删除：

```java
DELETE FROM table_name WHERE-clause;
```

没有 `WHERE` 子句，`UPDATE` 或 `DELETE` 语句将影响表的所有记录。

# 如何做...

我们已经看到了 `INSERT` 语句。这里是其他类型语句的示例：

![](img/456f0881-a644-4c13-b79f-68f96a8876e1.png)

前面的 `SELECT` 语句显示了表的所有行的所有列的值。为了限制返回的数据量，可以添加 `WHERE` 子句：

![](img/1ac573ef-e7fd-47fd-8922-ef05c578f9bf.png)

以下屏幕截图捕获了三个语句：

![](img/e186a7aa-af83-4997-b7f5-3f1a9f05fe97.png)

第一个是 `UPDATE` 语句，将 `value` 列中的值更改为 `NEW`，但仅在 `value` 列包含值 `new` 的行中（显然，该值区分大小写）。第二个语句删除所有 `value` 列中没有值 `NEW` 的行。第三个语句（`SELECT`）检索所有行的所有列的值。

值得注意的是，如果 `enums` 表的记录被 `traffic_unit` 表（作为外键）引用，我们将无法删除这些记录。只有在删除 `traffic_unit` 表的相应记录之后，才能删除 `enums` 表的记录。

要在代码中执行 CRUD 操作之一，首先必须获取 JDBC 连接，然后创建并执行语句：

```java
try (Connection conn = getDbConnection()) {
  try (Statement st = conn.createStatement()) {
    boolean res = st.execute("select id, type, value from enums");
    if (res) {
      ResultSet rs = st.getResultSet();
      while (rs.next()) {
        int id = rs.getInt(1); 
        String type = rs.getString(2);
        String value = rs.getString(3);
        System.out.println("id = " + id + ", type = " 
                           + type + ", value = " + value);
      }
    } else {
      int count = st.getUpdateCount();
      System.out.println("Update count = " + count);
    }
  }
} catch (Exception ex) { ex.printStackTrace(); }
```

使用 `try-with-resources` 结构来处理 `Statement` 对象是一个良好的实践。关闭 `Connection` 对象将自动关闭 `Statement` 对象。但是，当您显式关闭 `Statement` 对象时，清理将立即发生，而不必等待必要的检查和操作传播到框架的各个层。

`execute()` 方法是可以执行语句的三种方法中最通用的方法。其他两种包括 `executeQuery()`（仅用于 `SELECT` 语句）和 `executeUpdate()`（用于 `UPDATE`、`DELETE`、`CREATE` 或 `ALTER` 语句）。从前面的示例中可以看出，`execute()` 方法返回 `boolean`，表示结果是 `ResultSet` 对象还是仅仅是计数。这意味着 `execute()` 对于 `SELECT` 语句起到了 `executeQuery()` 的作用，对于我们列出的其他语句起到了 `executeUpdate()` 的作用。

我们可以通过运行前面的代码来演示以下语句序列：

```java
"select id, type, value from enums"
"insert into enums (id, type, value)" + " values(1,'vehicle','car')"
"select id, type, value from enums"
"update enums set value = 'bus' where value = 'car'"
"select id, type, value from enums"
"delete from enums where value = 'bus'"
"select id, type, value from enums"
```

结果将如下所示：

![](img/cf42e0dc-980d-4553-8b9e-aee5f1c48817.png)

我们使用了从 `ResultSet` 中提取值的位置提取，因为这比使用列名（如 `rs.getInt("id")` 或 `rs.getInt("type")`）更有效。性能差异非常小，只有在操作发生多次时才变得重要。只有实际的测量和测试才能告诉您这种差异对于您的应用程序是否重要。请记住，按名称获取值提供更好的代码可读性，在应用程序维护期间长期受益。

我们用`execute()`方法进行演示。在实践中，`executeQuery()`方法用于`SELECT`语句：

```java
try (Connection conn = getDbConnection()) {
  try (Statement st = conn.createStatement()) {
    boolean res = st.execute("select id, type, value from enums");
    ResultSet rs = st.getResultSet();
    while (rs.next()) {
        int id = rs.getInt(1); 
        String type = rs.getString(2);
        String value = rs.getString(3);
        System.out.println("id = " + id + ", type = " 
                           + type + ", value = " + value);
    }
  }
} catch (Exception ex) { ex.printStackTrace(); }
```

如您所见，前面的代码无法作为接收 SQL 语句作为参数的方法进行泛化。提取数据的代码特定于执行的 SQL 语句。相比之下，对`executeUpdate()`的调用可以包装在通用方法中：

```java
void executeUpdate(String sql){
  try (Connection conn = getDbConnection()) {
    try (Statement st = conn.createStatement()) {
      int count = st.executeUpdate(sql);
      System.out.println("Update count = " + count);
    }
  } catch (Exception ex) { ex.printStackTrace(); }
}
```

# 还有更多...

SQL 是一种丰富的语言，我们没有足够的空间来涵盖其所有功能。但我们想列举一些最受欢迎的功能，以便您了解它们的存在，并在需要时查找它们：

+   `SELECT`语句允许使用`DISTINCT`关键字，以摆脱所有重复的值

+   关键字`LIKE`允许您将搜索模式设置为`WHERE`子句

+   搜索模式可以使用多个通配符——`%， _`，`[charlist]`，`[^charlist]`或`[!charlist]`。

+   匹配值可以使用`IN`关键字枚举

+   `SELECT`语句可以使用`JOIN`子句包括多个表

+   `SELECT * INTO table_2 from table_1`创建`table_2`并从`table_1`复制数据

+   `TRUNCATE`在删除表的所有行时更快且使用更少的资源

`ResultSet`接口中还有许多其他有用的方法。以下是一些方法如何用于编写通用代码，遍历返回的结果并使用元数据打印列名和返回值的示例：

```java
void traverseRS(String sql){
  System.out.println("traverseRS(" + sql + "):");
  try (Connection conn = getDbConnection()) {
    try (Statement st = conn.createStatement()) {
      try(ResultSet rs = st.executeQuery(sql)){
        int cCount = 0;
        Map<Integer, String> cName = new HashMap<>();
        while (rs.next()) {
          if (cCount == 0) {
            ResultSetMetaData rsmd = rs.getMetaData();
            cCount = rsmd.getColumnCount();
            for (int i = 1; i <= cCount; i++) {
              cName.put(i, rsmd.getColumnLabel(i));
            }
          }
          List<String> l = new ArrayList<>();
          for (int i = 1; i <= cCount; i++) {
            l.add(cName.get(i) + " = " + rs.getString(i));
          }
          System.out.println(l.stream()
                              .collect(Collectors.joining(", ")));
        }
      }
    }
  } catch (Exception ex) { ex.printStackTrace(); }
}
```

我们只使用了一次`ResultSetMetaData`来收集返回的列名和一行的长度（列数）。然后，我们通过位置从每行中提取值，并创建了相应列名的`List<String>`元素。为了打印，我们使用了您已经熟悉的东西——程序员的乐趣——连接收集器（我们在上一章中讨论过）。如果我们调用`traverseRS("select * from enums")`方法，结果将如下所示：

![](img/e39680bf-4ea3-44d3-ac17-e0ae0bbe2e4f.png)

# 使用 Hikari 连接池（HikariCP）

在本配方中，您将学习如何设置和使用高性能的 HikariCP。

# 准备就绪

HikariCP 框架是由居住在日本的 Brett Wooldridge 创建的。*Hikari*在日语中的意思是*光*。它是一个轻量级且相对较小的 API，经过高度优化，并允许通过许多属性进行调整，其中一些属性在其他池中不可用。除了标准用户、密码、最大池大小、各种超时设置和缓存配置属性之外，它还公开了诸如`allowPoolSuspension`、`connectionInitSql`、`connectionTestQuery`等属性，甚至包括处理未及时关闭的连接的属性`leakDetectionThreshold`。

要使用最新（在撰写本书时）版本的 Hikari 池，请将以下依赖项添加到项目中：

```java
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>3.2.0</version>
 </dependency>
```

出于演示目的，我们将使用本章前一个配方中创建的数据库*使用 JDBC 连接到数据库*。我们还假设您已经学习了该配方，因此无需重复讨论数据库、JDBC 以及它们如何一起工作的内容。

# 如何做...

有几种配置 Hikari 连接池的方法。所有这些方法都基于`javax.sql.DataSource`接口的使用：

1.  最明显和直接的方法是直接在`DataSource`对象上设置池属性：

```java
HikariDataSource ds = new HikariDataSource();
ds.setPoolName("cookpool");
ds.setDriverClassName("org.postgresql.Driver");
ds.setJdbcUrl("jdbc:postgresql://localhost/cookbook");
ds.setUsername( "cook");
//ds.setPassword("123Secret");
ds.setMaximumPoolSize(10);
ds.setMinimumIdle(2);
ds.addDataSourceProperty("cachePrepStmts", Boolean.TRUE);
ds.addDataSourceProperty("prepStmtCacheSize", 256);
ds.addDataSourceProperty("prepStmtCacheSqlLimit", 2048);
ds.addDataSourceProperty("useServerPrepStmts", Boolean.TRUE);

```

我们已经注释掉了密码，因为我们没有为我们的数据库设置密码。在`jdbcUrl`和`dataSourceClassName`属性之间，一次只能使用一个，除非使用一些可能需要同时设置这两个属性的旧驱动程序。此外，请注意当没有专门的 setter 用于特定属性时，我们如何使用通用方法`addDataSourceProperty()`。

要从 PostgreSQL 切换到另一个关系数据库，您只需要更改驱动程序类名和数据库 URL。还有许多其他属性；其中一些是特定于数据库的，但我们不打算深入研究这些细节，因为本配方演示了如何使用 HikariCP。阅读有关特定于数据库的池配置属性的数据库文档，并了解如何使用它们来调整池以获得最佳性能，这在很大程度上也取决于特定应用程序与数据库的交互方式。

1.  配置 Hikari 池的另一种方法是使用`HikariConfig`类收集所有属性，然后将`HikariConfig`对象设置在`HikariDataSource`构造函数中：

```java
HikariConfig config = new HikariConfig();
config.setPoolName("cookpool");
config.setDriverClassName("org.postgresql.Driver");
config.setJdbcUrl("jdbc:postgresql://localhost/cookbook");
config.setUsername("cook");
//conf.setPassword("123Secret");
config.setMaximumPoolSize(10);
config.setMinimumIdle(2);
config.addDataSourceProperty("cachePrepStmts", true);
config.addDataSourceProperty("prepStmtCacheSize", 256);
config.addDataSourceProperty("prepStmtCacheSqlLimit", 2048);
config.addDataSourceProperty("useServerPrepStmts", true);

HikariDataSource ds = new HikariDataSource(config);

```

正如您所看到的，我们再次使用了通用方法`addDataSourceProperty()`，因为在`HikariConfig`类中也没有专门的设置器来设置这些属性。

1.  `HikariConfig`对象反过来可以使用`java.util.Properties`类填充数据：

```java
Properties props = new Properties();
props.setProperty("poolName", "cookpool");
props.setProperty("driverClassName", "org.postgresql.Driver");
props.setProperty("jdbcUrl", "jdbc:postgresql://localhost/cookbook");
props.setProperty("username", "cook");
//props.setProperty("password", "123Secret");
props.setProperty("maximumPoolSize", "10");
props.setProperty("minimumIdle", "2");
props.setProperty("dataSource.cachePrepStmts","true");
props.setProperty("dataSource.prepStmtCacheSize", "256");
props.setProperty("dataSource.prepStmtCacheSqlLimit", "2048");
props.setProperty("dataSource.useServerPrepStmts","true");

HikariConfig config = new HikariConfig(props);
HikariDataSource ds = new HikariDataSource(config);

```

请注意，对于在`HikariConfig`类中没有专用设置器的属性，我们使用了前缀`dataSource`。

1.  为了使配置更容易加载，`HikariConfig`类具有接受属性文件的构造函数。例如，让我们在`resources`文件夹中创建一个名为`database.properties`的文件，其中包含以下内容：

```java
poolName=cookpool
driverClassName=org.postgresql.Driver
jdbcUrl=jdbc:postgresql://localhost/cookbook
username=cook
password=
maximumPoolSize=10
minimumIdle=2
dataSource.cachePrepStmts=true
dataSource.useServerPrepStmts=true
dataSource.prepStmtCacheSize=256
dataSource.prepStmtCacheSqlLimit=2048

```

请注意，我们再次使用了相同属性的前缀`dataSource`。现在，我们可以直接将上述文件加载到`HikariConfig`构造函数中：

```java
ClassLoader loader = getClass().getClassLoader();
File file = 
   new File(loader.getResource("database.properties").getFile());
HikariConfig config = new HikariConfig(file.getAbsolutePath());
HikariDataSource ds = new HikariDataSource(config);
```

在幕后，正如您可能猜到的那样，它只是加载属性：

```java
public HikariConfig(String propertyFileName) {
    this();
    this.loadProperties(propertyFileName);
}
```

1.  或者，我们可以使用`HikariConfig`默认构造函数中包含的以下功能：

```java
String systemProp = 
       System.getProperty("hikaricp.configurationFile");
if (systemProp != null) {
    this.loadProperties(systemProp);
}
```

这意味着我们可以设置系统属性如下：

```java
-Dhikaricp.configurationFile=src/main/resources/database.properties
```

然后，我们可以配置 HikariCP 如下：

```java
HikariConfig config = new HikariConfig();
HikariDataSource ds = new HikariDataSource(config);

```

池配置的所有前述方法产生相同的结果，因此只取决于风格、约定或个人偏好来决定使用其中的哪一个。

# 它是如何工作的...

以下方法是使用创建的`DataSource`对象访问数据库，并从在配方*使用 JDBC 连接到数据库*中创建的`enums`表中选择所有值：

```java
void readData(DataSource ds) {
   try(Connection conn = ds.getConnection();
      PreparedStatement pst = 
        conn.prepareStatement("select id, type, value from enums");
      ResultSet rs = pst.executeQuery()){
      while (rs.next()) {
            int id = rs.getInt(1);
            String type = rs.getString(2);
            String value = rs.getString(3);
            System.out.println("id = " + id + ", type = " + 
                                      type + ", value = " + value);
      }
   } catch (SQLException ex){
      ex.printStackTrace();
   }
}
```

如果我们运行上述代码，结果将如下所示：

![](img/5414536a-6a29-4968-9f6d-00b03b826ae7.png)

# 还有更多...

您可以在 GitHub 上阅读有关 HikariCP 功能的更多信息（[`github.com/brettwooldridge/HikariCP`](https://github.com/brettwooldridge/HikariCP)）。

# 使用预处理语句

在本配方中，您将学习如何使用**预处理语句**——可以存储在数据库中并使用不同输入值高效执行的语句模板。

# 准备就绪

`PreparedStatement`对象——`Statement`的子接口——可以预编译并存储在数据库中，然后用于高效地执行不同输入值的 SQL 语句。与由`createStatement()`方法创建的`Statement`对象类似，它可以由相同`Connection`对象的`prepareStatement()`方法创建。

用于生成`Statement`的相同 SQL 语句也可以用于生成`PreparedStatement`。实际上，考虑使用`PrepdaredStatement`来替换多次调用的任何 SQL 语句是一个好主意，因为它的性能优于`Statement`。要做到这一点，我们只需要更改上一节示例代码中的这两行：

```java
try (Statement st = conn.createStatement()) {
  boolean res = st.execute("select * from enums");

```

我们将这些行更改为以下内容：

```java
try (PreparedStatement st = 
           conn.prepareStatement("select * from enums")) {
  boolean res = st.execute();

```

# 如何做...

`PreparedStatement`的真正用处在于它能够接受参数——替换（按顺序出现）`?`符号的输入值。以下是一个示例：

```java
traverseRS("select * from enums");
System.out.println();
try (Connection conn = getDbConnection()) {
  String[][] values = {{"1", "vehicle", "car"},
                       {"2", "vehicle", "truck"}};
  String sql = "insert into enums (id, type, value) values(?, ?, ?)");
  try (PreparedStatement st = conn.prepareStatement(sql) {
    for(String[] v: values){
      st.setInt(1, Integer.parseInt(v[0]));
      st.setString(2, v[1]);
      st.setString(3, v[2]);
      int count = st.executeUpdate();
      System.out.println("Update count = " + count);
    }
  }
} catch (Exception ex) { ex.printStackTrace(); }
System.out.println();
traverseRS("select * from enums");

```

其结果如下：

![](img/35314b63-dd97-4fae-8663-b4e6215e850f.png)

# 还有更多...

总是使用准备好的语句进行 CRUD 操作并不是一个坏主意。如果只执行一次，它们可能会比较慢，但您可以测试并查看这是否是您愿意支付的代价。通过系统地使用准备好的语句，您将产生一致的（更易读的）代码，提供更多的安全性（准备好的语句不容易受到 SQL 注入的攻击）。

# 使用事务

在这个教程中，您将学习什么是数据库事务以及如何在 Java 代码中使用它。

# 准备工作

**事务**是一个包括一个或多个改变数据的操作的工作单元。如果成功，所有数据更改都将被**提交**（应用到数据库）。如果其中一个操作出错，事务将**回滚**，并且事务中包含的所有更改都不会应用到数据库。

事务属性是在`Connection`对象上设置的。它们可以在不关闭连接的情况下更改，因此不同的事务可以重用相同的`Connection`对象。

JDBC 仅允许对 CRUD 操作进行事务控制。表修改（`CREATE TABLE`，`ALTER TABLE`等）会自动提交，并且无法从 Java 代码中进行控制。

默认情况下，CRUD 操作事务被设置为**自动提交**。这意味着每个由 SQL 语句引入的数据更改都会在该语句的执行完成后立即应用到数据库中。本章中的所有前面的示例都使用了这种默认行为。

要更改此行为，您必须使用`Connection`对象的`setAutoCommit(boolean)`方法。如果设置为`false`，则数据更改将不会应用到数据库，直到在`Connection`对象上调用`commit()`方法为止。此外，如果调用`rollback()`方法，那么自事务开始或上次调用`commit()`以来的所有数据更改都将被丢弃。

显式的程序化事务管理可以提高性能，但在只调用一次且不太频繁的短时间原子操作的情况下，它并不重要。接管事务控制在多个操作引入必须一起应用或全部不应用的更改时变得至关重要。它允许将数据库更改分组为原子单元，从而避免意外违反数据完整性。

# 如何做...

首先，让我们向`traverseRS()`方法添加一个输出：

```java
void traverseRS(String sql){
  System.out.println("traverseRS(" + sql + "):");
  try (Connection conn = getDbConnection()) {
    ...
  }
}
```

这将帮助您分析当许多不同的 SQL 语句在同一个演示示例中执行时的输出。

现在，让我们运行以下代码，从`enums`表中读取数据，然后插入一行，然后再次从表中读取所有数据：

```java
traverseRS("select * from enums");
System.out.println();
try (Connection conn = getDbConnection()) {
  conn.setAutoCommit(false);
  String sql = "insert into enums (id, type, value) "
                       + " values(1,'vehicle','car')";
  try (PreparedStatement st = conn.prepareStatement(sql)) {
    System.out.println(sql);
    System.out.println("Update count = " + st.executeUpdate());
  }
  //conn.commit();
} catch (Exception ex) { ex.printStackTrace(); }
System.out.println();
traverseRS("select * from enums");

```

请注意，我们通过调用`conn.setAutoCommit(false)`接管了事务控制。结果如下：

![](img/63dd3a04-23c6-4544-b564-656112337f7b.png)

正如您所看到的，由于对`commit()`的调用被注释掉，所以更改没有被应用。当我们取消注释时，结果会改变：

![](img/585677de-5653-4928-9304-f01cb41ee08d.png)

现在，让我们执行两次插入，但在第二次插入中引入一个拼写错误：

```java
traverseRS("select * from enums");
System.out.println();
try (Connection conn = getDbConnection()) {
  conn.setAutoCommit(false);
  String sql = "insert into enums (id, type, value) "
                       + " values(1,'vehicle','car')";
  try (PreparedStatement st = conn.prepareStatement(sql)) {
    System.out.println(sql);
    System.out.println("Update count = " + st.executeUpdate());
  }
  conn.commit();
  sql = "inst into enums (id, type, value) " 
                     + " values(2,'vehicle','truck')";
  try (PreparedStatement st = conn.prepareStatement(sql)) {
    System.out.println(sql);
    System.out.println("Update count = " + st.executeUpdate());
  }
  conn.commit();
} catch (Exception ex) { ex.printStackTrace(); } //get exception here
System.out.println();
traverseRS("select * from enums");
```

我们得到一个异常堆栈跟踪（我们不显示它以节省空间）和这个消息：

```java
org.postgresql.util.PSQLException: ERROR: syntax error at or near "inst"
```

尽管第一个插入成功执行：

![](img/e419c1af-ae1e-4f52-942c-a47dacd0d441.png)

第二行没有被插入。如果在第一个`INSERT INTO`语句之后没有`conn.commit()`，那么第一个插入也不会被应用。这是在许多独立数据更改的情况下进行程序化事务控制的优势——如果一个失败，我们可以跳过它并继续应用其他更改。

现在，让我们尝试插入三行数据，并在第二行中引入一个错误（将字母设置为`id`值而不是数字）：

```java
traverseRS("select * from enums");
System.out.println();
try (Connection conn = getDbConnection()) {
  conn.setAutoCommit(false);
  String[][] values = { {"1", "vehicle", "car"},
                        {"b", "vehicle", "truck"},
                        {"3", "vehicle", "crewcab"} };
  String sql = "insert into enums (id, type, value) " 
                            + " values(?, ?, ?)";
  try (PreparedStatement st = conn.prepareStatement(sql)) {
    for (String[] v: values){
      try {
        System.out.print("id=" + v[0] + ": ");
        st.setInt(1, Integer.parseInt(v[0]));
        st.setString(2, v[1]);
        st.setString(3, v[2]);
        int count = st.executeUpdate();
        conn.commit();
        System.out.println("Update count = "+count);
      } catch(Exception ex){
        //conn.rollback();
        System.out.println(ex.getMessage());
      }
    }
  }
} catch (Exception ex) { ex.printStackTrace(); }
System.out.println();
traverseRS("select * from enums");

```

我们将每个插入执行放在`try...catch`块中，并在打印出结果（更新计数或错误消息）之前提交更改。结果如下：

![](img/5a3508cd-c51f-4528-84f5-ab1488060a72.png)

正如你所看到的，第二行没有被插入，尽管`conn.rollback()`被注释掉了。为什么？这是因为此事务中只包括的 SQL 语句失败了，所以没有什么可以回滚的。

现在，让我们使用数据库控制台创建一个只有一个名为`name`的列的`test`表：

![](img/6ca14c11-e647-4987-be46-5c990139bfb0.png)

在插入`enums`表中的记录之前，我们将在`test`表中插入车辆类型：

```java
traverseRS("select * from enums");
System.out.println();
try (Connection conn = getDbConnection()) {
  conn.setAutoCommit(false);
  String[][] values = { {"1", "vehicle", "car"},
                        {"b", "vehicle", "truck"},
                        {"3", "vehicle", "crewcab"} };
  String sql = "insert into enums (id, type, value) " +
                                        " values(?, ?, ?)";
  try (PreparedStatement st = conn.prepareStatement(sql)) {
    for (String[] v: values){
      try(Statement stm = conn.createStatement()) {
        System.out.print("id=" + v[0] + ": ");
        stm.execute("insert into test values('"+ v[2] + "')");
        st.setInt(1, Integer.parseInt(v[0]));
        st.setString(2, v[1]);
        st.setString(3, v[2]);
        int count = st.executeUpdate();
        conn.commit();
        System.out.println("Update count = " + count);
      } catch(Exception ex){
         //conn.rollback();
         System.out.println(ex.getMessage());
      }
    }
  }
} catch (Exception ex) { ex.printStackTrace(); }
System.out.println();
traverseRS("select * from enums");
System.out.println();
traverseRS("select * from test");

```

正如你所看到的，前面的代码在第二次插入后提交了更改，就像前面的例子一样，对于数组`values`的第二个元素来说是不成功的。如果注释掉`conn.rollback()`，结果将如下所示：

![](img/df8484ad-6982-4d0c-881e-2b3898f82cbb.png)

`enums`表中没有插入`truck`行，但是添加到了`test`表中。也就是说，演示了回滚的有用性。如果我们取消注释`conn.rollback()`，结果将如下所示：

![](img/b69ab672-bf8d-4297-b83e-29fbbbda6850.png)

这表明`conn.rollback()`会回滚所有尚未提交的更改。

# 还有更多...

事务的另一个重要属性是**事务隔离级别**。它定义了数据库用户之间的边界。例如，在提交之前其他用户能否看到您的数据库更改？隔离级别越高（最高为**可串行化**），在并发访问相同记录的情况下，事务完成所需的时间就越长。隔离级别越低（最低为**读未提交**），数据就越脏，这意味着其他用户可以获取您尚未提交的值（也许永远不会提交）。

通常，使用默认级别就足够了，通常是`TRANSACTION_READ_COMMITTED`，尽管对于不同的数据库可能会有所不同。JDBC 允许您通过在`Connection`对象上调用`getTransactionIsolation()`方法来获取当前事务隔离级别。`Connection`对象的`setTransactionIsolation()`方法允许您根据需要设置任何隔离级别。

在复杂的决策逻辑中，关于哪些更改需要提交，哪些需要回滚，可以使用另外两个`Connection`方法来创建和删除**保存点**。`setSavepoint(String savepointName)`方法创建一个新的保存点并返回一个`Savepoint`对象，以后可以使用`rollback (Savepoint savepoint)`方法回滚到此点的所有更改。可以通过调用`releaseSavepoint(Savepoint savepoint)`来删除保存点。

最复杂的数据库事务类型是**分布式事务**。它们有时被称为**全局事务**、**XA 事务**或**JTA 事务**（后者是一个由两个 Java 包组成的 Java API，即`javax.transaction`和`javax.transaction.xa`）。它们允许创建和执行跨两个不同数据库操作的事务。提供分布式事务的详细概述超出了本书的范围。

# 使用大对象

在本教程中，您将学习如何存储和检索可以是三种类型之一的 LOB——**二进制大对象**（**BLOB**）、**字符大对象**（**CLOB**）和**国家字符大对象**（**NCLOB**）。

# 准备工作

数据库内 LOB 对象的实际处理是特定于供应商的，但是 JDBC API 通过将三种 LOB 类型表示为接口——`java.sql.Blob`、`java.sql.Clob`和`java.sql.NClob`，隐藏了这些实现细节。

`Blob`通常用于存储图像或其他非字母数字数据。在到达数据库之前，图像可以转换为字节流并使用`INSERT INTO`语句进行存储。`Blob`接口允许您查找对象的长度并将其转换为 Java 可以处理的字节数组，以便显示图像等目的。

`Clob`允许您存储字符数据。`NClob`以一种支持国际化的方式存储 Unicode 字符数据。它扩展了`Clob`接口并提供相同的方法。这两个接口都允许您查找 LOB 的长度并获取值内的子字符串。

`ResultSet`，`CallableStatement`（我们将在下一个示例中讨论），和`PreparedStatement`接口中的方法允许应用程序以各种方式存储和访问存储的值——有些通过相应对象的设置器和获取器，而其他一些作为`bytes[]`，或作为二进制、字符或 ASCII 流。

# 如何做...

每个数据库都有其特定的存储 LOB 的方式。在 PostgreSQL 的情况下，`Blob`通常映射到`OID`或`BYTEA`数据类型，而`Clob`和`NClob`则映射到`TEXT`类型。为了演示如何做到这一点，让我们创建可以存储每种大对象类型的表。我们将编写一个新的方法，以编程方式创建表：

```java
void execute(String sql){
  try (Connection conn = getDbConnection()) {
    try (PreparedStatement st = conn.prepareStatement(sql)) {
      st.execute();
    }
  } catch (Exception ex) {
    ex.printStackTrace();
  }
}
```

现在，我们可以创建三个表：

```java
execute("create table images (id integer, image bytea)");
execute("create table lobs (id integer, lob oid)");
execute("create table texts (id integer, text text)");
```

查看 JDBC 接口`PreparedStatement`和`ResultSet`，您会注意到对象的设置器和获取器——`get/setBlob()`，`get/setClob()`，`get/setNClob()`，`get/setBytes()`——以及使用`InputStream`和`Reader`的方法——`get/setBinaryStream()`，`get/setAsciiStream()`，或`get/setCharacterStream()`。流方法的重要优势在于它们在数据库和源之间传输数据而无需在内存中存储整个 LOB。

然而，对象的设置器和获取器更符合我们的心意，符合面向对象编程。因此，我们将从它们开始，使用一些不太大的对象进行演示。我们期望以下代码可以正常工作：

```java
try (Connection conn = getDbConnection()) {
  String sql = "insert into images (id, image) values(?, ?)";
  try (PreparedStatement st = conn.prepareStatement(sql)) {
    st.setInt(1, 100);
    File file = 
       new File("src/main/java/com/packt/cookbook/ch06_db/image1.png");
    FileInputStream fis = new FileInputStream(file);
    Blob blob = conn.createBlob();   
    OutputStream out = blob.setBinaryStream(1);
    int i = -1;
    while ((i = fis.read()) != -1) {
      out.write(i);
    }
    st.setBlob(2, blob);
    int count = st.executeUpdate();
    System.out.println("Update count = " + count);
  }
} catch (Exception ex) { ex.printStackTrace(); }
```

或者，在`Clob`的情况下，我们编写这段代码：

```java
try (Connection conn = getDbConnection()) {
  String sql = "insert into texts (id, text) values(?, ?)";
  try (PreparedStatement st = conn.prepareStatement(sql)) {
    st.setInt(1, 100);
    File file = new File("src/main/java/com/packt/cookbook/" +
                                    "ch06_db/Chapter06Database.java");
    Reader reader = new FileReader(file);
    st.setClob(2, reader);  
    int count = st.executeUpdate();
    System.out.println("Update count = " + count);
  }
} catch (Exception ex) { ex.printStackTrace(); }
```

事实证明，并非所有 JDBC API 中可用的方法都由所有数据库的驱动程序实际实现。例如，`createBlob()`似乎对 Oracle 和 MySQL 都可以正常工作，但在 PostgreSQL 的情况下，我们得到了这个：

![](img/2f4a350b-2998-4002-bd0e-84a9f5767a20.png)

对于`Clob`，我们得到以下结果：

![](img/321c623c-b082-4b55-abf2-fd0a1de83494.png)

我们也可以尝试通过获取器从`ResultSet`中检索对象：

```java
String sql = "select image from images";
try (PreparedStatement st = conn.prepareStatement(sql)) {
  st.setInt(1, 100);
  try(ResultSet rs = st.executeQuery()){
    while (rs.next()){
      Blob blob = rs.getBlob(1); 
      System.out.println("blob length = " + blob.length());
    }
  }
}
```

结果将如下：

![](img/31e08560-ab43-4f7c-950b-b51ed9df254b.png)

显然，仅仅了解 JDBC API 是不够的；您还必须阅读数据库的文档。以下是 PostgreSQL 文档（[`jdbc.postgresql.org/documentation/80/binary-data.html`](https://jdbc.postgresql.org/documentation/80/binary-data.html)）对 LOB 处理的说明：

"要使用 BYTEA 数据类型，您应该简单地使用`getBytes()`，`setBytes()`，`getBinaryStream()`或`setBinaryStream()`方法。

要使用大对象功能，您可以使用由 PostgreSQL JDBC 驱动程序提供的`LargeObject`类，或者使用`getBLOB()`和`setBLOB()`方法。"

此外，您必须在 SQL 事务块内访问大对象。您可以通过调用`setAutoCommit(false)`来启动事务块。

不知道这样的具体情况，找出处理 LOB 的方法将需要很多时间并引起很多挫折。

在处理 LOB 时，流方法更受青睐，因为它们直接从源传输数据到数据库（或反之），并且不像设置器和获取器那样消耗内存（后者必须首先将整个 LOB 加载到内存中）。以下是在 PostgreSQL 数据库中流传`Blob`的代码：

```java
traverseRS("select * from images");
System.out.println();
try (Connection conn = getDbConnection()) {
  String sql = "insert into images (id, image) values(?, ?)";
  try (PreparedStatement st = conn.prepareStatement(sql)) {
    st.setInt(1, 100);
    File file = 
       new File("src/main/java/com/packt/cookbook/ch06_db/image1.png");
    FileInputStream fis = new FileInputStream(file);
    st.setBinaryStream(2, fis);
    int count = st.executeUpdate();
    System.out.println("Update count = " + count);
  }
  sql = "select image from images where id = ?";
  try (PreparedStatement st = conn.prepareStatement(sql)) {
    st.setInt(1, 100);
    try(ResultSet rs = st.executeQuery()){
      while (rs.next()){
        try(InputStream is = rs.getBinaryStream(1)){
          int i;
          System.out.print("ints = ");
          while ((i = is.read()) != -1) {
            System.out.print(i);
          }
        }
      }
    }
  }
} catch (Exception ex) { ex.printStackTrace(); }
System.out.println();
traverseRS("select * from images");

```

让我们来看看结果。我们在右侧任意裁剪了截图；否则，它在水平上太长了：

![](img/b260d44f-2343-4d92-b53f-66a43e2efdb2.png)

处理检索到的图像的另一种方法是使用`byte[]`：

```java
try (Connection conn = getDbConnection()) {
  String sql =  "insert into images (id, image) values(?, ?)";
  try (PreparedStatement st = conn.prepareStatement(sql)) {
    st.setInt(1, 100);
    File file = 
       new File("src/main/java/com/packt/cookbook/ch06_db/image1.png");
    FileInputStream fis = new FileInputStream(file);
    byte[] bytes = fis.readAllBytes();
    st.setBytes(2, bytes);
    int count = st.executeUpdate();
    System.out.println("Update count = " + count);
  }
  sql = "select image from images where id = ?";
  System.out.println();
  try (PreparedStatement st = conn.prepareStatement(sql)) {
    st.setInt(1, 100);
    try(ResultSet rs = st.executeQuery()){
      while (rs.next()){
        byte[] bytes = rs.getBytes(1);
        System.out.println("bytes = " + bytes);
      }
    }
  }
} catch (Exception ex) { ex.printStackTrace(); }

```

PostgreSQL 将`BYTEA`大小限制为 1 GB。较大的二进制对象可以存储为**对象标识符**（**OID**）数据类型：

```java
traverseRS("select * from lobs");
System.out.println();
try (Connection conn = getDbConnection()) {
  conn.setAutoCommit(false);
  LargeObjectManager lobm = 
        conn.unwrap(org.postgresql.PGConnection.class)
            .getLargeObjectAPI();
  long lob = lobm.createLO(LargeObjectManager.READ 
                           | LargeObjectManager.WRITE);
  LargeObject obj = lobm.open(lob, LargeObjectManager.WRITE);
  File file = 
       new File("src/main/java/com/packt/cookbook/ch06_db/image1.png");
  try (FileInputStream fis = new FileInputStream(file)){
    int size = 2048;
    byte[] bytes = new byte[size];
    int len = 0;
    while ((len = fis.read(bytes, 0, size)) > 0) {
      obj.write(bytes, 0, len);
    }
    obj.close();
    String sql = "insert into lobs (id, lob) values(?, ?)";
    try (PreparedStatement st = conn.prepareStatement(sql)) {
      st.setInt(1, 100);
      st.setLong(2, lob);
      st.executeUpdate();
    }
  }
    conn.commit();
} catch (Exception ex) { ex.printStackTrace(); }
System.out.println();
traverseRS("select * from lobs");

```

结果将如下所示：

![](img/f99937f5-82a8-464b-bcd8-4474f59e2f42.png)

请注意，`select`语句从`lob`列返回一个长值。这是因为`OID`列不像`BYTEA`那样存储值本身。相反，它存储对存储在数据库中其他位置的对象的引用。这样的安排使得删除具有 OID 类型的行不像这样直截了当：

```java
execute("delete from lobs where id = 100"); 

```

如果只是这样做，它会使实际对象成为一个继续占用磁盘空间的孤立对象。为了避免这个问题，您必须首先通过执行以下命令`unlink` LOB：

```java
execute("select lo_unlink((select lob from lobs " + " where id=100))");

```

只有在此之后，您才能安全地执行`delete from lobs where id = 100`命令。

如果您忘记首先`unlink`，或者因为代码错误而意外创建了孤立的 LOB（例如），则可以通过在系统表中查找孤立对象的方式来找到孤立对象。同样，数据库文档应该为您提供如何执行此操作的说明。在 PostgreSQL v.9.3 或更高版本中，您可以通过执行`select count(*) from pg_largeobject`命令来检查是否有孤立的 LOB。如果返回的计数大于 0，则可以使用以下连接删除所有孤立对象（假设`lobs`表是唯一一个可能包含对 LOB 的引用的表）：

```java
SELECT lo_unlink(pgl.oid) FROM pg_largeobject_metadata pgl
WHERE (NOT EXISTS (SELECT 1 FROM lobs ls" + "WHERE ls.lob = pgl.oid));
```

这是在数据库中存储 LOB 所需付出的代价。

值得注意的是，尽管`BYTEA`在删除操作期间不需要这样的复杂性，但它有一种不同类型的开销。根据 PostgreSQL 文档，当接近 1 GB 时，*处理这样一个大值将需要大量内存*。

要读取 LOB 数据，您可以使用以下代码：

```java
try (Connection conn = getDbConnection()) {
  conn.setAutoCommit(false);
  LargeObjectManager lobm =      
          conn.unwrap(org.postgresql.PGConnection.class)
              .getLargeObjectAPI();
  String sql = "select lob from lobs where id = ?";
  try (PreparedStatement st = conn.prepareStatement(sql)) {
    st.setInt(1, 100);
    try(ResultSet rs = st.executeQuery()){
      while (rs.next()){
        long lob = rs.getLong(1);
        LargeObject obj = lobm.open(lob, LargeObjectManager.READ);
        byte[] bytes = new byte[obj.size()];
        obj.read(bytes, 0, obj.size());
        System.out.println("bytes = " + bytes);
        obj.close();
      }
    }
  }
  conn.commit();
} catch (Exception ex) { ex.printStackTrace(); }

```

或者，如果 LOB 不太大，可以直接从`ResultSet`对象中获取`Blob`，使用更简单的代码：

```java
while (rs.next()){
  Blob blob = rs.getBlob(1);
  byte[] bytes = blob.getBytes(1, (int)blob.length());
  System.out.println("bytes = " + bytes);
}
```

要在 PostgreSQL 中存储`Clob`，可以使用与前面相同的代码。在从数据库中读取时，可以将字节转换为`String`数据类型或类似的内容（同样，如果 LOB 不太大）：

```java
String str = new String(bytes, Charset.forName("UTF-8"));
System.out.println("bytes = " + str);

```

然而，在 PostgreSQL 中，`Clob`可以直接存储为无限大小的数据类型`TEXT`。这段代码读取了存有这段代码的文件，并将其存储/检索到/从数据库中：

```java
traverseRS("select * from texts");
System.out.println();
try (Connection conn = getDbConnection()) {
  String sql = "insert into texts (id, text) values(?, ?)";
  try (PreparedStatement st = conn.prepareStatement(sql)) {
    st.setInt(1, 100);
    File file = new File("src/main/java/com/packt/cookbook/ch06_db/"
                                          + "Chapter06Database.java");
    try (FileInputStream fis = new FileInputStream(file)) {
      byte[] bytes = fis.readAllBytes();
      st.setString(2, new String(bytes, Charset.forName("UTF-8")));
    }
    int count = st.executeUpdate();
    System.out.println("Update count = " + count);
  }
  sql = "select text from texts where id = ?";
  try (PreparedStatement st = conn.prepareStatement(sql)) {
    st.setInt(1, 100);
    try(ResultSet rs = st.executeQuery()){
      while (rs.next()) {
        String str = rs.getString(1);
        System.out.println(str);
      }
    }
  }
} catch (Exception ex) { ex.printStackTrace(); }
```

结果将如下所示（我们只显示了输出的前几行）：

![](img/f762b169-3217-4bd3-9a84-254c4b414e97.png)

对于更大的对象，流式处理方法将是更好的（如果不是唯一的）选择：

```java
traverseRS("select * from texts");
System.out.println();
try (Connection conn = getDbConnection()) {
  String sql = "insert into texts (id, text) values(?, ?)";
  try (PreparedStatement st = conn.prepareStatement(sql)) {
    st.setInt(1, 100);
    File file = new File("src/main/java/com/packt/cookbook/ch06_db/"
                                          + "Chapter06Database.java");
    //This is not implemented:
    //st.setCharacterStream(2, reader, file.length()); 
    st.setCharacterStream(2, reader, (int)file.length());

    int count = st.executeUpdate();
    System.out.println("Update count = " + count);
  }
} catch (Exception ex) { ex.printStackTrace(); }
System.out.println();
traverseRS("select * from texts");
```

请注意，截至撰写本书时，`setCharacterStream(int, Reader, long)`尚未实现，而`setCharacterStream(int, Reader, int)`却可以正常工作。

我们还可以从`texts`表中以字符流的形式读取文件，并将其限制在前 160 个字符：

```java
String sql = "select text from texts where id = ?";
try (PreparedStatement st = conn.prepareStatement(sql)) {
  st.setInt(1, 100);
  try(ResultSet rs = st.executeQuery()){
    while (rs.next()) {
      try(Reader reader = rs.getCharacterStream(1)) {
        char[] chars = new char[160];
        reader.read(chars);
        System.out.println(chars);
      }
    }
  }
}
```

结果将如下所示：

![](img/1da666ee-263a-4240-a5db-ac07702ff50a.png)

# 还有更多...

以下是来自 PostgreSQL 文档的另一个建议（您可以在[`jdbc.postgresql.org/documentation/80/binary-data.html`](https://jdbc.postgresql.org/documentation/80/binary-data.html)上访问）：

“BYTEA 数据类型不适合存储大量二进制数据。虽然 BYTEA 类型的列最多可以容纳 1 GB 的二进制数据，但处理这样大的值将需要大量内存。

用于存储二进制数据的大对象方法更适合存储非常大的值，但它也有自己的局限性。特别是删除包含大对象引用的行并不会删除大对象。删除大对象是一个需要执行的单独操作。大对象也存在一些安全问题，因为任何连接到数据库的人都可以查看和/或修改任何大对象，即使他们没有权限查看/更新包含大对象引用的行。

在决定将 LOB 存储在数据库中时，您需要记住，数据库越大，维护起来就越困难。选择数据库作为存储设施的主要优势——访问速度也会下降，而且无法为 LOB 类型创建索引以改善搜索。此外，除了一些 CLOB 情况，您不能在`WHERE`子句中使用 LOB 列，也不能在`INSERT`或`UPDATE`语句的多行中使用 LOB 列。

因此，在考虑 LOB 的数据库之前，您应该始终考虑在数据库中存储文件名、关键字和其他一些内容属性是否足够解决方案。

# 执行存储过程

在这个教程中，您将学习如何从 Java 程序中执行数据库存储过程。

# 做好准备。

偶尔，Java 程序员会遇到需要操作和/或从多个表中选择数据的需求，因此程序员会提出一组复杂的 SQL 语句，这些语句在 Java 中实现起来不切实际，或者强烈怀疑 Java 实现可能无法获得足够的性能。这时，SQL 语句可以被封装成一个存储过程，编译并存储在数据库中，然后通过 JDBC 接口调用。或者，另一种情况是，Java 程序员可能需要将对现有存储过程的调用合并到程序中。为了实现这一点，可以使用`CallableStatement`接口（它扩展了`PreparedStatement`接口），尽管一些数据库允许您使用`Statement`或`PreparedStatement`接口调用存储过程。

`CallableStatement`可以有三种类型的参数——`IN`表示输入值，`OUT`表示结果，`IN OUT`表示输入或输出值。`OUT`参数必须由`CallableStatement`的`registerOutParameter()`方法注册。`IN`参数可以像`PreparedStatement`的参数一样设置。

请记住，从 Java 程序中以编程方式执行存储过程是最不标准化的领域之一。例如，PostgreSQL 不直接支持存储过程，但可以作为函数调用，这些函数已经被修改，将`OUT`参数解释为返回值。另一方面，Oracle 也允许函数使用`OUT`参数。

这就是为什么数据库函数和存储过程之间的以下差异只能作为一般指南，而不能作为正式定义：

+   函数具有返回值，但不允许`OUT`参数（除了一些数据库），并且可以在 SQL 语句中使用。

+   存储过程没有返回值（除了一些数据库）；它允许`OUT`参数（对于大多数数据库），并且可以使用 JDBC 接口`CallableStatement`执行。

这就是为什么阅读数据库文档以了解如何执行存储过程非常重要。

由于存储过程是在数据库服务器上编译和存储的，`CallableStatement`的`execute()`方法对于相同的 SQL 语句执行要比`Statement`或`PreparedStatement`的相应方法性能更好。这是为什么很多 Java 代码有时会被一个或多个存储过程替代的原因之一，甚至包括业务逻辑。但是对于每种情况和问题，都没有一个正确的答案，因此我们将避免提出具体的建议，除了重复熟悉的测试价值和您正在编写的代码的清晰度的口头禅。

# 如何做到...

与之前的示例一样，我们将继续使用 PostgreSQL 数据库进行演示。在编写自定义 SQL 语句、函数和存储过程之前，您应该先查看已经存在的函数列表。通常，它们提供了丰富的功能。

以下是调用`replace(string text, from text, to text)`函数的示例，该函数搜索第一个参数（`string text`）并用第二个参数（`from text`）匹配的所有子字符串替换为第三个参数（`string text`）提供的子字符串：

```java
String sql = "{ ? = call replace(?, ?, ? ) }";
try (CallableStatement st = conn.prepareCall(sql)) {
  st.registerOutParameter(1, Types.VARCHAR);
  st.setString(2, "Hello, World! Hello!");
  st.setString(3, "llo");
  st.setString(4, "y");
  st.execute();
  String res = st.getString(1);
  System.out.println(res);
}
```

结果如下：

![](img/5629dab5-57b2-4b6e-9c01-d1fee37bd23f.png)

我们将把这个函数合并到我们的自定义函数和存储过程中，以展示如何完成这个操作。

存储过程可以没有任何参数，只有`IN`参数，只有`OUT`参数，或者两者都有。结果可以是一个或多个值，或者是一个`ResultSet`对象。以下是在 PostgreSQL 中创建一个没有任何参数的存储过程的示例：

```java
execute("create or replace function createTableTexts() " 
        + " returns void as " 
        + "$$ drop table if exists texts; "
        + "  create table texts (id integer, text text); "
        + "$$ language sql");

```

在上述代码中，我们使用了我们已经熟悉的`execute()`方法：

```java
void execute(String sql){
  try (Connection conn = getDbConnection()) {
    try (PreparedStatement st = conn.prepareStatement(sql)) {
      st.execute();
    }
  } catch (Exception ex) {
    ex.printStackTrace();
  }
}
```

这个存储过程（在 PostgreSQL 中总是一个函数）创建了`texts`表（如果表已经存在，则先删除）。您可以在数据库文档中找到有关函数创建的 SQL 语法。我们想在这里评论的唯一一件事是，可以使用单引号代替表示函数主体的符号`$$`。不过，我们更喜欢`$$`，因为它有助于避免在函数主体中需要包含单引号时进行转义。

在被创建并存储在数据库中之后，该存储过程可以通过`CallableStatement`来调用：

```java
String sql = "{ call createTableTexts() }";
try (CallableStatement st = conn.prepareCall(sql)) {
  st.execute();
}
```

或者，可以使用 SQL 语句`select createTableTexts()`或`select * from createTableTexts()`来调用它。这两个语句都会返回一个`ResultSet`对象（在`createTableTexts()`函数的情况下是`null`），因此我们可以通过我们的方法来遍历它：

```java
void traverseRS(String sql){
  System.out.println("traverseRS(" + sql + "):");
  try (Connection conn = getDbConnection()) {
    try (Statement st = conn.createStatement()) {
      try(ResultSet rs = st.executeQuery(sql)){
        int cCount = 0;
        Map<Integer, String> cName = new HashMap<>();
        while (rs.next()) {
          if (cCount == 0) {
            ResultSetMetaData rsmd = rs.getMetaData();
            cCount = rsmd.getColumnCount();
            for (int i = 1; i <= cCount; i++) {
              cName.put(i, rsmd.getColumnLabel(i));
            }
          }
          List<String> l = new ArrayList<>();
          for (int i = 1; i <= cCount; i++) {
            l.add(cName.get(i) + " = " + rs.getString(i));
          }
          System.out.println(l.stream()
                      .collect(Collectors.joining(", ")));
        }
      }
    }
  } catch (Exception ex) { ex.printStackTrace(); }
}
```

我们已经在之前的示例中使用过这个方法。

可以使用以下语句删除该函数：

```java
drop function if exists createTableTexts(); 
```

现在，让我们把所有这些放在一起，用 Java 代码创建一个函数，并以三种不同的方式调用它：

```java
execute("create or replace function createTableTexts() " 
        + "returns void as "
        + "$$ drop table if exists texts; "
        + "  create table texts (id integer, text text); "
        + "$$ language sql");
String sql = "{ call createTableTexts() }";
try (Connection conn = getDbConnection()) {
  try (CallableStatement st = conn.prepareCall(sql)) {
    st.execute();
  }
}
traverseRS("select createTableTexts()");
traverseRS("select * from createTableTexts()");
execute("drop function if exists createTableTexts()");

```

结果如下：

![](img/16547c6c-f4c8-410c-ac89-85d2b6907f29.png)

正如预期的那样，返回的`ResultSet`对象是`null`。请注意，函数的名称是不区分大小写的。我们将其保留为驼峰式风格，仅供人类阅读。

现在，让我们创建并调用另一个带有两个输入参数的存储过程（函数）：

```java
execute("create or replace function insertText(int,varchar)" 
        + " returns void "
        + " as $$ insert into texts (id, text) "
        + "   values($1, replace($2,'XX','ext'));" 
        + " $$ language sql");
String sql = "{ call insertText(?, ?) }";
try (Connection conn = getDbConnection()) {
  try (CallableStatement st = conn.prepareCall(sql)) {
    st.setInt(1, 1);
    st.setString(2, "TXX 1");
    st.execute();
  }
}
execute("select insertText(2, 'TXX 2')");
traverseRS("select * from texts");
execute("drop function if exists insertText()");

```

在函数的主体中，输入参数通过它们的位置`$1`和`$2`来引用。正如之前提到的，我们还使用了内置的`replace()`函数来操作第二个输入参数的值，然后将其插入到表中。新创建的存储过程被调用了两次：首先通过`CallableStatment`，然后通过`execute()`方法，使用不同的输入值。然后，我们使用`traverseRS("select * from texts")`来查看表中的内容，并删除了新创建的函数以进行清理。我们只是为了演示目的删除了该函数。在实际代码中，一旦创建，函数就会留在那里，并利用它的存在，编译并准备运行。

如果我们运行上述代码，将得到以下结果：

![](img/3b9eb46d-9339-48fd-aba3-d616bb91f34c.png)

现在让我们向`texts`表添加两行，然后查看它并创建一个计算表中行数并返回结果的存储过程（函数）：

```java
execute("insert into texts (id, text) " 
         + "values(3,'Text 3'),(4,'Text 4')");
traverseRS("select * from texts");
execute("create or replace function countTexts() " 
        + "returns bigint as " 
        + "$$ select count(*) from texts; " 
        + "$$ language sql");
String sql = "{ ? = call countTexts() }";
try (Connection conn = getDbConnection()) {
  try (CallableStatement st = conn.prepareCall(sql)) {
    st.registerOutParameter(1, Types.BIGINT);
    st.execute();
    System.out.println("Result of countTexts() = " + st.getLong(1));
  }
}
traverseRS("select countTexts()");
traverseRS("select * from countTexts()");
execute("drop function if exists countTexts()");

```

注意返回值的`bigint`值和`OUT`参数`Types.BIGINT`的匹配类型。新创建的存储过程被执行三次，然后被删除。结果如下所示：

![](img/dc60f770-b493-45d3-9fa5-d984d1fbf62e.png)

现在，让我们看一个具有`int`类型的输入参数并返回`ResultSet`对象的存储过程示例：

```java
execute("create or replace function selectText(int) " 
        + "returns setof texts as 
        + "$$ select * from texts where id=$1; " 
        + "$$ language sql");
traverseRS("select selectText(1)");
traverseRS("select * from selectText(1)");
execute("drop function if exists selectText(int)");
```

注意返回类型定义为`setof texts`，其中`texts`是表的名称。如果我们运行上述代码，结果将如下所示：

![](img/da57a6ac-c2d8-4e27-8e76-638b967f48f2.png)

值得分析的是两次对存储过程的不同调用在`ResultSet`内容上的差异。没有`select *`时，它包含存储过程的名称和返回对象（`ResultSet`类型）。但是有`select *`时，它返回存储过程中最后一个 SQL 语句的实际`ResultSet`内容。

自然而然地，问题就是为什么我们不能通过`CallableStatement`调用这个存储过程，如下所示：

```java
String sql = "{ ? = call selectText(?) }";
try (CallableStatement st = conn.prepareCall(sql)) {
  st.registerOutParameter(1, Types.OTHER);
  st.setInt(2, 1);
  st.execute();
  traverseRS((ResultSet)st.getObject(1));
}

```

我们尝试了，但没有成功。以下是 PostgreSQL 文档对此的描述：

"返回数据集的函数不应该通过 CallableStatement 接口调用，而应该使用普通的 Statement 或 PreparedStatement 接口。"

不过，有一种方法可以绕过这个限制。同样的数据库文档描述了如何检索`refcursor`（PostgreSQL 特有的功能）值，然后可以将其转换为`ResultSet`：

```java
execute("create or replace function selectText(int) " 
        + "returns refcursor " +
        + "as $$ declare curs refcursor; " 
        + " begin " 
        + "   open curs for select * from texts where id=$1;" 
        + "     return curs; " 
        + " end; " 
        + "$$ language plpgsql");
String sql = "{ ? = call selectText(?) }";
try (Connection conn = getDbConnection()) {
  conn.setAutoCommit(false);
  try(CallableStatement st = conn.prepareCall(sql)){
    st.registerOutParameter(1, Types.OTHER);
    st.setInt(2, 2);
    st.execute();
    try(ResultSet rs = (ResultSet) st.getObject(1)){
      System.out.println("traverseRS(refcursor()=>rs):");
      traverseRS(rs);
    }
  }
}
traverseRS("select selectText(2)");
traverseRS("select * from selectText(2)");
execute("drop function if exists selectText(int)");

```

关于前面的代码的一些评论可能会帮助您理解这是如何完成的：

+   必须关闭自动提交。

+   函数内部，`$1`指的是第一个`IN`参数（不包括`OUT`参数）。

+   语言设置为`plpgsql`以访问`refcursor`功能（PL/pgSQL 是 PostgreSQL 数据库的可加载过程语言）。

+   为了遍历`ResultSet`，我们编写了一个新的方法，如下所示：

```java
        void traverseRS(ResultSet rs) throws Exception {
          int cCount = 0;
          Map<Integer, String> cName = new HashMap<>();
          while (rs.next()) {
            if (cCount == 0) {
              ResultSetMetaData rsmd = rs.getMetaData();
              cCount = rsmd.getColumnCount();
              for (int i = 1; i <= cCount; i++) {
                cName.put(i, rsmd.getColumnLabel(i));
              }
            }
            List<String> l = new ArrayList<>();
            for (int i = 1; i <= cCount; i++) {
              l.add(cName.get(i) + " = " + rs.getString(i));
            }
            System.out.println(l.stream()
                      .collect(Collectors.joining(", ")));
          }
        }
```

因此，我们的老朋友`traverseRS(String sql)`方法现在可以重构为以下形式：

```java
void traverseRS(String sql){
  System.out.println("traverseRS(" + sql + "):");
  try (Connection conn = getDbConnection()) {
    try (Statement st = conn.createStatement()) {
      try(ResultSet rs = st.executeQuery(sql)){
        traverseRS(rs);
      }
    }
  } catch (Exception ex) { ex.printStackTrace(); }
}
```

如果我们运行最后一个示例，结果将如下所示：

![](img/bc185a01-c374-498a-ab21-ade478c6451b.png)

您可以看到，不提取对象并将其转换为`ResultSet`的结果遍历方法在这种情况下不显示正确的数据。

# 还有更多...

我们介绍了从 Java 代码调用存储过程的最常见情况。本书的范围不允许我们介绍 PostgreSQL 和其他数据库中更复杂和潜在有用的存储过程形式。但是，我们想在这里提到它们，这样你就可以了解其他可能性：

+   复合类型的函数

+   带有参数名称的函数

+   具有可变数量参数的函数

+   具有参数默认值的函数

+   函数作为表源

+   返回表的函数

+   多态的 SQL 函数

+   具有排序规则的函数

# 使用批处理操作处理大量数据

在这个示例中，您将学习如何创建和执行许多 SQL 语句的**批处理**。

# 准备工作

当需要执行许多 SQL 语句以同时插入、更新或读取数据库记录时，需要批处理。可以通过迭代执行多个 SQL 语句并逐个发送到数据库来执行多个 SQL 语句，但这会产生网络开销，可以通过同时将所有查询发送到数据库来避免。

为了避免这种网络开销，所有的 SQL 语句可以合并成一个`String`值，每个语句之间用分号分隔，这样它们可以一次性发送到数据库。返回的结果（如果有的话）也会作为每个语句生成的结果集的集合发送回来。这种处理通常被称为批量处理，以区别于仅适用于`INSERT`和`UPDATE`语句的批处理。批处理允许您使用`java.sql.Statement`或`java.sql.PreparedStatement`接口的`addBatch()`方法来组合许多 SQL 语句。

我们将使用 PostgreSQL 数据库和以下表`person`来插入、更新和从中读取数据：

```java
create table person (
   name VARCHAR NOT NULL,
   age INTEGER NOT NULL
)
```

正如您所看到的，表的每个记录都可以包含一个人的两个属性——姓名和年龄。

# 如何做到这一点...

我们将演示**批量处理**和**批处理**。为了实现这一点，让我们按照以下步骤进行：

1.  批量处理的一个示例是使用单个`INSERT`语句和多个`VALUES`子句：

```java
INSERT into <table_name> (column1, column2, ...) VALUES 
                         ( value1,  value2, ...),
                         ( value1,  value2, ...),
                          ...
                         ( value1,  value2, ...)
```

构造这样一个语句的代码如下：

```java
int n = 100000;  //number of records to insert
StringBuilder sb = 
 new StringBuilder("insert into person (name,age) values ");
for(int i = 0; i < n; i++){
   sb.append("(")
     .append("'Name").append(String.valueOf(i)).append("',")
     .append(String.valueOf((int)(Math.random() * 100)))
     .append(")");
   if(i < n - 1) {
        sb.append(",");
   }
}
try(Connection conn = getConnection();
    Statement st = conn.createStatement()){
    st.execute(sb.toString());
} catch (SQLException ex){
    ex.printStackTrace();
}
```

正如您所看到的，前面的代码构造了一个带有 100,000 个`VALUES`子句的语句，这意味着它一次性向数据库插入了 100,000 条记录。在我们的实验中，完成这项工作花费了 1,082 毫秒。结果，表`person`现在包含了 100,000 条记录，姓名从`Name0`到`Name99999`，年龄是从 1 到 99 的随机数。

这种批量处理方法有两个缺点——它容易受到 SQL 注入攻击，并且可能消耗太多内存。使用`PreparedStatement`可以解决 SQL 注入问题，但受到绑定变量数量的限制。在 PostgreSQL 的情况下，它不能超过`32767`。这意味着我们需要将单个`PreparedStatement`分解为几个更小的`PreparedStatement`，每个`PreparedStatement`中的绑定变量不超过`32767`。顺便说一句，这也可以解决内存消耗问题，因为每个语句现在比大语句小得多。例如，之前的单个语句包括 200,000 个值。

1.  以下代码通过将单个 SQL 语句分解为多个不超过`32766`个绑定变量的`PreparedStatement`对象来解决这两个问题：

```java
int n = 100000, limit = 32766, l = 0;
List<String> queries = new ArrayList<>();
List<Integer> bindVariablesCount = new ArrayList<>();
String insert = "insert into person (name, age) values ";
StringBuilder sb = new StringBuilder(insert);
for(int i = 0; i < n; i++){
    sb.append("(?, ?)");
    l = l + 2;
    if(i == n - 1) {
        queries.add(sb.toString());
        bindVariablesCount.add(l % limit);
    }
    if(l % limit == 0) {
        queries.add(sb.toString());
        bindVariablesCount.add(limit);
        sb = new StringBuilder(insert);
    } else {
        sb.append(",");
    }
}
try(Connection conn = getConnection()) {
    int i = 0, q = 0;
    for(String query: queries){
        try(PreparedStatement pst = conn.prepareStatement(query)) {
            int j = 0;
            while (j < bindVariablesCount.get(q)) {
                pst.setString(++j, "Name" + String.valueOf(i++));
                pst.setInt(++j, (int)(Math.random() * 100));
            }
            pst.executeUpdate();
            q++;
        }
    }
} catch (SQLException ex){
    ex.printStackTrace();
}
```

前面的代码执行速度与我们之前的示例一样快。完成这项工作花费了 1,175 毫秒。但是我们在安装了数据库的同一台计算机上运行了这段代码，所以没有来自数据库的七次网络开销（这是添加到`List queries`的查询次数）。但是，正如您所看到的，代码相当复杂。通过使用批处理，可以大大简化它。

1.  批处理基于`addBatch()`和`executeBatch()`方法的使用，这两种方法在`Statement`和`PreparedStatement`接口中都可用。为了演示，我们将使用`PreparedStatement`，原因有两个——它不容易受到 SQL 注入攻击，并且在多次执行时性能更好（这是`PreparedStatement`的主要目的——利用对具有不同值的相同语句的多次执行）。

```java
int n = 100000;
String insert = 
           "insert into person (name, age) values (?, ?)";
try (Connection conn = getConnection();
    PreparedStatement pst = conn.prepareStatement(insert)) {
    for (int i = 0; i < n; i++) {
        pst.setString(1, "Name" + String.valueOf(i));
        pst.setInt(2, (int)(Math.random() * 100));
        pst.addBatch();
    }
    pst.executeBatch();
} catch (SQLException ex) {
    ex.printStackTrace();
}
```

向`person`表中插入 100,000 条记录花费了 2,299 毫秒，几乎是使用单个带有多个`VALUES`子句的语句（第一个示例）或使用多个`PreparedStatement`对象（第二个示例）的两倍长。尽管它的执行时间更长，但这段代码明显更简单。它将一批语句一次性发送到数据库，这意味着当数据库与应用程序不在同一台计算机上时，这种实现与之前的实现（需要七次访问数据库）之间的性能差距将更小。

但是这种实现也可以得到改进。

1.  为了改进批处理，让我们向`DataSource`对象添加`reWriteBatchedInserts`属性，并将其设置为`true`：

```java
DataSource createDataSource() {
    HikariDataSource ds = new HikariDataSource();
    ds.setPoolName("cookpool");
    ds.setDriverClassName("org.postgresql.Driver");
    ds.setJdbcUrl("jdbc:postgresql://localhost/cookbook");
    ds.setUsername( "cook");
    //ds.setPassword("123Secret");
    ds.setMaximumPoolSize(2);
    ds.setMinimumIdle(2);
    ds.addDataSourceProperty("reWriteBatchedInserts", 
                                            Boolean.TRUE);
    return ds;
}
```

现在，如果我们使用连接`createDataSource().getConnection()`运行相同的批处理代码，插入相同的 10 万条记录所需的时间减少到 750 毫秒，比我们迄今为止测试过的任何实现都要好 25%。而且代码仍然比以前的任何实现都简单得多。

但是内存消耗呢？

1.  随着批处理的增长，JVM 可能会在某个时候耗尽内存。在这种情况下，批处理可以分成几个批次，每个批次在单独的行程中传递到数据库：

```java
int n = 100000;
int batchSize = 30000;
boolean execute = false;
String insert = 
          "insert into person (name, age) values (?, ?)";
try (Connection conn = getConnection();
    PreparedStatement pst = conn.prepareStatement(insert)) {
    for (int i = 0; i < n; i++) {
        pst.setString(1, "Name" + String.valueOf(i));
        pst.setInt(2, (int)(Math.random() * 100));
        pst.addBatch();
        if((i > 0 && i % batchSize == 0) || 
                                 (i == n - 1 && execute)) {
             pst.executeBatch();
             System.out.print(" " + i); 
                        //prints: 30000 60000 90000 99999
             if(n - 1 - i < batchSize && !execute){
                  execute = true;
             }
        }
    }
    pst.executeBatch();
} catch (SQLException ex) {
    ex.printStackTrace();
}
```

我们使用变量`execute`作为标志，指示当最后一个批次小于`batchSize`值时，我们需要在最后一个语句添加到批次时再次调用`executeBatch()`。从前面代码的注释中可以看出，`executeBatch()`被调用了四次，包括在添加最后一个语句时（当`i=99999`）。在我们的运行中，这段代码的性能与生成多个批次的性能相同，因为我们的数据库位于与应用程序相同的计算机上。否则，每个批次通过网络传递都会增加执行此代码所需的时间。

# 工作原理...

前一个子部分的最后一个示例（步骤 5）是用于在数据库中插入和更新记录的批处理过程的终极实现。方法`executeBatch()`返回一个`int`数组，成功的情况下，表示批处理中每个语句更新了多少行。对于`INSERT`语句，此值等于-2（负二），这是静态常量`Statement.SUCCESS_NO_INFO`的值。值为-3（负三），即常量`Statement.EXECUTE_FAILED`的值，表示语句失败。

如果预期返回的更新行数大于`Integer.MAX_VALUE`，则使用方法`long[] executeLargeBatch()`来执行批处理。

没有用于从数据库中读取数据的批处理。要批量读取数据，可以将许多语句作为一个字符串以分号分隔发送到数据库，然后迭代返回的多个结果集。例如，让我们提交`SELECT`语句，计算年龄值从 1 到 99 的每个记录的数量：

```java
int minAge = 0, maxAge = 0, minCount = n, maxCount = 0;
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 100; i++) {
    sb.append("select count(*) from person where age = ")
                                          .append(i).append(";");
}
try (Connection conn = getConnection();
     PreparedStatement pst = conn.prepareStatement(sb.toString())) {
    boolean hasResult = pst.execute();
    int i = 0;
    while (hasResult){
        try (ResultSet rs = pst.getResultSet()) {
            rs.next();
            int c = rs.getInt(1);
            if(c < minCount) {
                minAge = i;
                minCount = c;
            }
            if(c > maxCount) {
                maxAge = i;
                maxCount = c;
            }
            i++;
            hasResult = pst.getMoreResults();
        }
    }
} catch (SQLException ex) {
    ex.printStackTrace();
}
System.out.println("least popular age=" + minAge + "(" + minCount + 
             "), most popular age=" + maxAge + "(" + maxCount + ")");

```

在我们的测试运行中，执行上述代码并显示以下消息花费了 2,162 毫秒：

```java
least popular age=14(929), most popular age=10(1080)
```

# 还有更多...

将大量数据从 PostgreSQL 数据库移动到数据库和从数据库移动也可以使用`COPY`命令来完成，该命令将数据复制到文件中。您可以在数据库文档中了解更多信息（[`www.postgresql.org/docs/current/static/sql-copy.html`](https://www.postgresql.org/docs/current/static/sql-copy.html)）。

# 使用 MyBatis 进行 CRUD 操作

在以前的示例中，使用 JDBC 时，我们需要编写代码，从查询返回的`ResultSet`对象中提取查询的结果。这种方法的缺点是，您必须编写相当多的样板代码来创建和填充表示数据库中记录的域对象。正如我们在本章的介绍中已经提到的那样，有几个 ORM 框架可以为您执行此操作，并自动创建相应的域对象（或者换句话说，将数据库记录映射到相应的域对象）。当然，每个这样的框架都会减少一些构建 SQL 语句的控制和灵活性。因此，在承诺使用特定的 ORM 框架之前，您需要研究和尝试不同的框架，以找到提供应用程序所需的数据库方面的一切，并且不会产生太多开销的框架。

在这个教程中，您将学习到 SQL 映射工具 MyBatis，与直接使用 JDBC 相比，它简化了数据库编程。

# 准备工作

MyBatis 是一个轻量级的 ORM 框架，不仅允许将结果映射到 Java 对象，还允许执行任意的 SQL 语句。主要有两种描述映射的方式：

+   使用 Java 注解

+   使用 XML 配置

在这个教程中，我们将使用 XML 配置。但是，无论您喜欢哪种风格，您都需要创建一个`org.apache.ibatis.session.SqlSessionFactory`类型的对象，然后使用它来通过创建一个`org.apache.ibatis.session.SqlSession`类型的对象来启动 MyBatis 会话：

```java
InputStream inputStream = Resources.getResourceAsStream(configuration);
SqlSessionFactory sqlSessionFactory = 
                     new SqlSessionFactoryBuilder().build(inputStream);
SqlSession session = sqlSessionFactory.openSession();
```

`SqlSessionFactoryBuilder`对象有九个重载的`build()`方法，用于创建`SqlSession`对象。使用这些方法，您可以定义以下内容：

+   无论您喜欢自动提交数据库更改还是显式执行它们（我们在示例中使用后者）

+   无论您是使用配置的数据源（就像我们的例子中）还是使用外部提供的数据库连接

+   无论您是使用默认的数据库特定事务隔离级别（就像我们的例子中）还是想要设置自己的

+   您将使用以下哪种`ExecutorType`值——`SIMPLE`（默认值，为每次执行语句创建一个新的`PreparedStatement`）、`REUSE`（重用`PreparedStatements`）或`BATCH`（批处理所有更新语句，并在它们之间执行`SELECT`时必要时标记它们）

+   这段代码部署在哪个环境（例如`development`、`test`或`production`），因此将使用相应的配置部分（我们将很快讨论它）

+   包含数据源配置的`Properties`对象

`SqlSession`对象提供了允许您执行在 SQL 映射 XML 文件中定义的`SELECT`、`INSERT`、`UPDATE`和`DELETE`语句的方法。它还允许您提交或回滚当前事务。

我们用于这个教程的 Maven 依赖如下：

```java
<dependency>
   <groupId>org.mybatis</groupId>
   <artifactId>mybatis</artifactId>
   <version>3.4.6</version>
</dependency>
```

在撰写本书时，最新的 MyBatis 文档可以在这里找到：[`www.mybatis.org/mybatis-3/index.html `](http://www.mybatis.org/mybatis-3/index.html)

# 如何做…

我们将从使用 PostgreSQL 数据库和`Person1`类开始进行 CRUD 操作：

```java
public class Person1 {
    private int id;
    private int age;
    private String name;
    public Person1(){}  //Must be present, used by the framework
    public Person1(int age, String name){
        this.age = age;
        this.name = name;
    }
    public int getId() { return id; }
    public void setName(String name) { this.name = name; }
    @Override
    public String toString() {
        return "Person1{id=" + id + ", age=" + age +
                                  ", name='" + name + "'}";
    }
}
```

我们需要之前的`getId()`方法来获取一个 ID 值（演示如何通过 ID 查找数据库记录）。`setName()`方法将用于更新数据库记录，`toString()`方法将用于显示结果。我们使用名称`Person1`来区分它与同一类的另一个版本`Person2`，我们将使用它来演示如何实现类和相应表之间的关系。

可以使用以下 SQL 语句创建匹配的数据库表：

```java
create table person1 (
   id SERIAL PRIMARY KEY,
   age INTEGER NOT NULL,
   name VARCHAR NOT NULL
);
```

然后执行以下步骤：

1.  首先创建一个 XML 配置文件。我们将其命名为`mb-config1.xml`，并将其放在`resources`文件夹下的`mybatis`文件夹中。这样，Maven 会将其放在类路径上。另一个选项是将文件放在任何其他文件夹中，并与 Java 代码一起修改`pom.xml`，告诉 Maven 也将该文件夹中的`.xml`文件放在类路径上。`mb-config1.xml`文件的内容如下：

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <settings>
    <setting name="useGeneratedKeys" value="true"/>
  </settings>
  <typeAliases>
     <typeAlias type="com.packt.cookbook.ch06_db.mybatis.Person1" 
                                                 alias="Person"/>
  </typeAliases>
  <environments default="development">
     <environment id="development">
       <transactionManager type="JDBC"/>
       <dataSource type="POOLED">
          <property name="driver" value="org.postgresql.Driver"/>
          <property name="url" 
                   value="jdbc:postgresql://localhost/cookbook"/>
          <property name="username" value="cook"/>
          <property name="password" value=""/>
        </dataSource>
     </environment>
  </environments>
  <mappers>
      <mapper resource="mybatis/Person1Mapper.xml"/>
  </mappers>
</configuration>
```

`<settings>`标签允许全局定义一些行为——延迟加载值、启用/禁用缓存、设置自动映射行为（是否填充嵌套数据）等。我们选择全局设置自动生成的键的用法，因为我们需要插入的对象被填充为在数据库中生成的 ID。

`<typeAiases>` 标签包含完全限定类名的别名，其工作方式类似于 `IMPORT` 语句。唯一的区别是别名可以是任何单词，而不仅仅是类名。在声明别名之后，在 MyBatis 的 `.xml` 文件中的任何其他地方，只能通过此别名引用该类。我们将在不久的将来查看文件 `Person1Mapper.xml` 的内容时看到如何做到这一点。

`<environments>` 标签包含不同环境的配置。例如，我们可以为环境 `env42`（任何字符串都可以）创建一个配置。然后，在创建 `SqlSession` 对象时，可以将此名称作为方法 `SqlSessionFactory.build()` 的参数传递，将使用标签 `<environment id="env42"></environment>` 中包含的配置。它定义要使用的事务管理器和数据源。

`TransactionManager` 可以是两种类型之一——JDBC，它使用数据源提供的连接来提交、回滚和管理事务的范围，以及 `MANAGED`，它什么也不做，允许容器管理事务的生命周期——好吧，默认情况下它会关闭连接，但是该行为可以通过设置以下属性进行更改：

```java

<transactionManager type="MANAGED">
    <property name="closeConnection" value="false"/>
</transactionManager>
```

标签 `<mappers>` 包含对所有包含映射数据库记录和 Java 对象的 SQL 语句的 `.xml` 文件的引用，这在我们的情况下是文件 `Person1Mapper.xml`。

1.  创建 `Person1Mapper.xml` 文件，并将其放在与 `mb-config1.xml` 文件相同的文件夹中。此文件可以使用任何您喜欢的名称，但它包含映射数据库记录和类 `Person1` 对象的所有 SQL 语句，因此我们将其命名为 `Person1Mapper.xml` 仅仅是为了清晰起见：

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="mybatis.Person1Mapper">
   <insert id="insertPerson" keyProperty="id" keyColumn="id" 
                                         parameterType="Person">
      insert into Person1 (age, name) values(#{age}, #{name})
   </insert>
   <select id="selectPersonById" parameterType="int" 
                                            resultType="Person">
      select * from Person1 where id = #{id}
   </select>
   <select id="selectPersonsByName" parameterType="string" 
                                           resultType="Person">
      select * from Person1 where name = #{name}
   </select>
   <select id="selectPersons" resultType="Person">
      select * from Person1
   </select>
   <update id="updatePersonById" parameterType="Person">
      update Person1 set age = #{age}, name = #{name} 
                                              where id = #{id}
   </update>
   <delete id="deletePersons"> 
      delete from Person1
   </delete>
</mapper>
```

如您所见，`<mapper>` 标签具有 `namespace` 属性，用于解析不同位置相同名称的文件。它可能匹配或不匹配映射器文件的位置。映射器文件的位置在配置文件 `mb-config1.xml` 中指定为标签 `<mapper>` 的属性资源（参见上一步）。

标签 `<insert>`、`<select>`、`<update>` 和 `<delete>` 的属性在很大程度上是不言自明的。标签 `<settings>` 中的属性 `keyProperty`、`keyColumn` 和 `useGeneratedKeys` 被添加以用数据库生成的值填充插入的对象。如果不需要全局使用，可以从配置中的设置中删除属性 `useGeneratedKeys`，并且只在希望利用某些值的自动生成的插入语句中添加。我们这样做是因为我们想要获取生成的 ID，并在以后的代码中使用它来演示如何通过 ID 检索记录。

`<select>` 和类似标签的 ID 属性用于调用它们，以及映射器命名空间值。我们将很快向您展示如何做到这一点。构造 `#{id}` 是指传递的值作为参数，如果该值是原始类型。否则，传递的对象应该具有这样的字段。对象上的 getter 不是必需的。如果存在 getter，则必须符合 JavaBean 方法格式。

对于返回值，默认情况下，列的名称与对象字段或 setter 的名称匹配（必须符合 JavaBean 方法格式）。如果字段（或 setter 名称）和列名不同，可以使用标签 `<resultMap>` 提供映射。例如，如果表 `person` 具有列 `person_id` 和 `person_name`，而域对象 `Person` 具有字段 `id` 和 `name`，我们可以创建一个映射：

```java
<resultMap id="personResultMap" type="Person">
    <id property="id" column="person_id" />
    <result property="name" column="person_name"/>
</resultMap>
```

然后可以使用此 `resultMap` 来填充域对象 `Person`，如下所示：

```java
<select id="selectPersonById" parameterType="int" 
                                  resultMap="personResultMap">
   select person_id, person_name from Person where id = #{id}
</select>
```

或者，也可以使用标准的 select 子句别名：

```java
<select id="selectPersonById" parameterType="int" 
                                          resultType="Person">
   select person_id as "id", person_name as "name" from Person 
                                              where id = #{id}
</select>
```

1.  编写代码，向表 `person1` 中插入记录，然后通过 `id` 读取此记录：

```java
String resource = "mybatis/mb-config1.xml";
String mapperNamespace = "mybatis.Person1Mapper";
try {
   InputStream inputStream = 
                    Resources.getResourceAsStream(resource);
   SqlSessionFactory sqlSessionFactory = 
          new SqlSessionFactoryBuilder().build(inputStream);
   try(SqlSession session = sqlSessionFactory.openSession()){
       Person1 p = new Person1(10, "John");
       session.insert(mapperNamespace + ".insertPerson", p);
       session.commit();
       p = session.selectOne(mapperNamespace + 
                            ".selectPersonById", p.getId());
        System.out.println("By id " + p.getId() + ": " + p);
    } catch (Exception ex) {
        ex.printStackTrace();
    }
} catch (Exception ex){
    ex.printStackTrace();
}
```

上述代码将产生一个输出：

```java
By id 1: Person1{id=1, age=10, name='John'}
```

`Resources`实用程序有十个重载方法用于读取配置文件。我们已经描述了如何确保 Maven 将配置和 mapper 文件放在类路径上。

`SqlSession`对象实现了`AutoCloseable`接口，因此我们可以使用 try-with-resources 块，而不必担心资源泄漏。`SqlSession`接口提供了许多执行方法，包括重载方法`insert()`、`select()`、`selectList()`、`selectMap()`、`selectOne()`、`update()`和`delete()`，这些是最常用和直接的方法。我们还使用了`insert()`和`selectOne()`。后者确保只返回一个结果。否则，它会抛出异常。当用于通过值标识单个记录的列没有唯一约束时，它也会抛出异常。这就是为什么我们在 ID 列上添加了`PRIMARY KEY`限定符。或者，我们可以只添加唯一约束（将其标记为`PRIMARY KEY`会隐式执行此操作）。

另一方面，`selectList()`方法生成一个`List`对象，即使只返回一个结果。我们现在来演示一下。

1.  编写代码，从表`person1`中读取所有记录：

```java
List<Person1> list = session.selectList(mapperNamespace 
                                    + ".selectPersons");
for(Person1 p1: list) {
    System.out.println("All: " + p1);
}
```

上述代码将产生以下输出：

```java
All: Person1{id=1, age=10, name='John'}
```

1.  为了演示更新，让我们将`"John"`的名字更改为`"Bill"`，然后再次读取`person1`中的所有记录：

```java
List<Person1> list = session.selectList(mapperNamespace 
                      + ".selectPersonsByName", "John");
for(Person1 p1: list) {
    p1.setName("Bill");
    int c = session.update(mapperNamespace + 
                               ".updatePersonById", p1);
    System.out.println("Updated " + c + " records");
}
session.commit();
list = 
 session.selectList(mapperNamespace + ".selectPersons");
for(Person1 p1: list) {
    System.out.println("All: " + p1);
}
```

上述代码将产生以下输出：

```java
Updated 1 records
All: Person1{id=1, age=10, name='Bill'}
```

注意更改是如何提交的：`session.commit()`。没有这一行，结果是一样的，但更改不会持久化，因为默认情况下事务不会自动提交。可以通过在打开会话时将 autocommit 设置为`true`来更改它：

```java
      SqlSession session = sqlSessionFactory.openSession(true);
```

1.  最后，调用`DELETE`语句并从表`person1`中删除所有记录：

```java
int c = session.delete(mapperNamespace + ".deletePersons");
System.out.println("Deleted " + c + " persons");
session.commit();

List<Person1> list = session.selectList(mapperNamespace + 
                                         ".selectPersons");
System.out.println("Total records: " + list.size());

```

上述代码将产生以下输出：

```java
Deleted 0 persons
Total records: 0
```

1.  为了演示 MyBatis 如何支持关系，请创建表`family`和表`person2`：

```java
create table family (
    id SERIAL PRIMARY KEY,
    name VARCHAR NOT NULL
);
create table person2 (
    id SERIAL PRIMARY KEY,
    age INTEGER NOT NULL,
    name VARCHAR NOT NULL,
    family_id INTEGER references family(id) 
                             ON DELETE CASCADE
);
```

如您所见，表`family`和`person2`中的记录具有一对多的关系。表`person2`中的每条记录可能属于一个家庭（指向一个`family`记录）或不属于。几个人可以属于同一个家庭。我们还添加了`ON DELETE CASCADE`子句，以便在删除它们所属的家庭时自动删除`person2`记录。

相应的 Java 类如下所示：

```java
class Family {
    private int id;
    private String name;
    private final List<Person2> members = new ArrayList<>();
    public Family(){}  //Used by the framework
    public Family(String name){ this.name = name; }
    public int getId() { return id; }
    public String getName() { return name; }
    public List<Person2> getMembers(){ return this.members; }
}
```

如您所见，类`Family`有一个`Person2`对象的集合。对于`getId()`和`getMembers()`方法，我们需要与`Person2`类建立关系。我们将使用`getName()`方法来进行演示代码。

`Person2`类如下所示：

```java
class Person2 {
    private int id;
    private int age;
    private String name;
    private Family family;
    public Person2(){}  //Used by the framework
    public Person2(int age, String name, Family family){
        this.age = age;
        this.name = name;
        this.family = family;
    }
    @Override
    public String toString() {
        return "Person2{id=" + id + ", age=" + age +
                 ", name='" + name + "', family='" +
         (family == null ? "" : family.getName())+ "'}";
    }
}
```

1.  创建一个名为`mb-config2.xml`的新配置文件：

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="useGeneratedKeys" value="true"/>
    </settings>
    <typeAliases>
       <typeAlias type="com.packt.cookbook.ch06_db.mybatis.Family" 
                                                    alias="Family"/>
       <typeAlias type="com.packt.cookbook.ch06_db.mybatis.Person2" 
                                                    alias="Person"/>
    </typeAliases>
    <environments default="development">
       <environment id="development">
          <transactionManager type="JDBC"/>
          <dataSource type="POOLED">
             <property name="driver" value="org.postgresql.Driver"/>
             <property name="url" 
                      value="jdbc:postgresql://localhost/cookbook"/>
             <property name="username" value="cook"/>
             <property name="password" value=""/>
          </dataSource>
       </environment>
    </environments>
    <mappers>
        <mapper resource="mybatis/FamilyMapper.xml"/>
        <mapper resource="mybatis/Person2Mapper.xml"/>
    </mappers>
</configuration>
```

注意，我们现在有两个别名和两个 mapper`.xml`文件。

1.  `Person2Mapper.xml`文件的内容比我们之前使用的`Person1Mapper.xml`文件要小得多：

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="mybatis.Person2Mapper">
    <insert id="insertPerson" keyProperty="id" keyColumn="id"
                                          parameterType="Person">
        insert into Person2 (age, name, family_id) 
                    values(#{age}, #{name}, #{family.id})
    </insert>
    <select id="selectPersonsCount" resultType="int">
        select count(*) from Person2
    </select>
</mapper>
```

这是因为我们不打算直接更新或管理这些人。我们将通过他们所属的家庭来进行操作。我们添加了一个新的查询，返回`person2`表中记录的数量。

1.  `FamilyMapper.xml`文件的内容如下：

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
 PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="mybatis.FamilyMapper">
 <insert id="insertFamily" keyProperty="id" keyColumn="id" 
 parameterType="Family">
 insert into Family (name) values(#{name})
 </insert>
 <select id="selectMembersOfFamily" parameterType="int" 
 resultMap="personMap">
 select * from Person2 where family_id = #{id}
 </select>
 <resultMap id="personMap" type="Person">
 <association property="family" column="family_id" 
 select="selectFamilyById"/>
 </resultMap>
 <select id="selectFamilyById" parameterType="int"
 resultType="Family">
 select * from Family where id = #{id}
 </select>
 <select id="selectFamilies" resultMap="familyMap">
 select * from Family
 </select>
 <resultMap id="familyMap" type="Family">
 <collection property="members" column="id" ofType="Person" 
 select="selectMembersOfFamily"/>
 </resultMap>
 <select id="selectFamiliesCount" resultType="int">
 select count(*) from Family
 </select>
 <delete id="deleteFamilies">
 delete from Family
 </delete>

```

家庭 mapper 更加复杂，因为我们在其中管理关系。首先，看一下查询`selectMembersOfFamily`。如果您不想在`Person2`对象的`family`字段中填充数据，SQL 将会简单得多，如下所示：

```java
    <select id="selectMembersOfFamily" parameterType="int" 
 resultType="Person">
        select * from Person2 where family_id = #{id}
    </select>

```

但是我们希望在`Person2`对象中设置相应的`Family`对象值，因此我们使用了`ResultMap` `personMap`，它仅描述了默认情况下无法完成的映射——我们使用`<association>`标签将`family`字段与`family_id`列关联起来，并使用查询`selectFamilyById`。这个查询不会填充`Family`对象的`members`字段，但我们决定在我们的演示中不需要它。

我们在查询`selectMembersOfFamily`中重用了查询`selectFamilies`。为了填充`Family`对象的`members`字段，我们创建了一个`ResultMap` `familyMap`，它使用`selectMembersOfFamily`来实现。

# 它是如何工作的...

让我们编写代码来演示对`family`表的 CRUD 操作。首先，这是如何创建一个`family`记录并与两个`person2`记录关联的代码：

```java
String resource = "mybatis/mb-config2.xml";
String familyMapperNamespace = "mybatis.FamilyMapper";
String personMapperNamespace = "mybatis.Person2Mapper";
try {
    InputStream inputStream = Resources.getResourceAsStream(resource);
    SqlSessionFactory sqlSessionFactory =
            new SqlSessionFactoryBuilder().build(inputStream);
    try(SqlSession session = sqlSessionFactory.openSession()){
        Family f = new Family("The Jones");
        session.insert(familyMapperNamespace + ".insertFamily", f);
        System.out.println("Family id=" + f.getId()); //Family id=1

        Person2 p = new Person2(25, "Jill", f);
        session.insert(personMapperNamespace + ".insertPerson", p);
        System.out.println(p); 
          //Person2{id=1, age=25, name='Jill', family='The Jones'}

        p = new Person2(30, "John", f);
        session.insert(personMapperNamespace + ".insertPerson", p);
        System.out.println(p);
          //Person2{id=2, age=30, name='John', family='The Jones'}

        session.commit();
    } catch (Exception ex) {
        ex.printStackTrace();
    }
} catch (Exception ex){
    ex.printStackTrace();
}
```

现在，我们可以使用以下代码读取创建的记录：

```java
List<Family> fList = 
       session.selectList(familyMapperNamespace + ".selectFamilies");
for (Family f1: fList) {
    System.out.println("Family " + f1.getName() + " has " + 
                               f1.getMembers().size() + " members:");
    for(Person2 p1: f1.getMembers()){
        System.out.println("   " + p1);
    }
}
```

前面的代码片段产生了以下输出：

```java
Family The Jones has 2 members:
 Person2{id=1, age=25, name='Jill', family='The Jones'}
 Person2{id=2, age=30, name='John', family='The Jones'}

```

现在，我们可以删除所有`family`记录，并检查在此之后`family`和`person2`表中是否包含任何记录：

```java
int c = session.delete(familyMapperNamespace + ".deleteFamilies");
System.out.println("Deleted " + c + " families");
session.commit();

c = session.selectOne(familyMapperNamespace + ".selectFamiliesCount");
System.out.println("Total family records: " + c);

c = session.selectOne(personMapperNamespace + ".selectPersonsCount");
System.out.println("Total person records: " + c);

```

前面的代码片段的输出如下：

```java
Deleted 1 families
Total family records: 0
Total person records: 0
```

`person2`表现在也是空的，因为我们在创建表时添加了`ON DELETE CASCADE`子句。

# 还有更多...

MyBatis 还提供了构建动态 SQL 的工具，一个 SqlBuilder 类以及许多其他构建和执行任意复杂 SQL 或存储过程的方法。有关详细信息，请阅读[`www.mybatis.org/mybatis-3`](http://www.mybatis.org/mybatis-3)中的文档。

# 使用 Java 持久化 API 和 Hibernate

在本教程中，您将学习如何使用名为**Hibernate 对象关系映射**（**ORM**）框架的**Java 持久化 API**（**JPA**）实现来填充、读取、更改和删除数据库中的数据。

# 准备工作

JPA 是一种定义 ORM 的规范解决方案。您可以在以下链接找到 JPA 版本 2.2：

```java
http://download.oracle.com/otn-pub/jcp/persistence-2_2-mrel-spec/JavaPersistence.pdf
```

规范中描述的接口、枚举、注解和异常属于`javax.persistence`包（[`javaee.github.io/javaee-spec/javadocs`](https://javaee.github.io/javaee-spec/javadocs)），该包包含在**Java 企业版**（**EE**）中。 JPA 由几个框架实现，其中最流行的是：

+   Hibernate ORM ([`hibernate.org/orm`](http://hibernate.org/orm))

+   EclipseLink ([`www.eclipse.org/eclipselink`](http://www.eclipse.org/eclipselink))

+   Oracle TopLink ([`www.oracle.com/technetwork/middleware/toplink/overview/index.html`](http://www.oracle.com/technetwork/middleware/toplink/overview/index.html))

+   jOOQ ([`www.jooq.org`](https://www.jooq.org))

JPA 围绕实体设计——映射到数据库表的 Java bean，使用注解进行映射。或者，可以使用 XML 或两者的组合来定义映射。由 XML 定义的映射优先于注解定义的映射。规范还定义了一种类似 SQL 的查询语言，用于静态和动态数据查询。

大多数 JPA 实现允许使用注解和 XML 定义的映射创建数据库模式。

# 如何做...

1.  让我们从将`javax.persistence`包依赖项添加到 Maven 配置文件`pom.xml`开始：

```java
<dependency>
    <groupId>javax.persistence</groupId>
    <artifactId>javax.persistence-api</artifactId>
    <version>2.2</version>
</dependency>
```

我们还不需要任何 JPA 实现。这样，我们可以确保我们的代码不使用任何特定于框架的代码，只使用 JPA 接口。

1.  创建`Person1`类：

```java
public class Person1 {
    private int age;
    private String name;
    public Person1(int age, String name){
        this.age = age;
        this.name = name;
    }
    @Override
    public String toString() {
        return "Person1{id=" + id + ", age=" + age +
                          ", name='" + name + "'}";
    }
}
```

我们不添加 getter、setter 或任何其他方法；这样可以使我们的代码示例简短而简单。根据 JPA 规范，要将此类转换为实体，需要在类声明中添加注解`@Entity`（需要导入`java.persistence.Entity`）。这意味着我们希望这个类代表数据库表中的一条记录，表名为`person`。默认情况下，实体类的名称与表的名称相匹配。但是可以使用注解`@Table(name="<another table name>")`将类映射到另一个名称的表。类的每个属性也映射到具有相同名称的列，可以使用注解`@Column (name="<another column name>")`更改默认名称。

此外，实体类必须有一个主键——一个由注解`@Id`表示的字段。也可以使用注解`@IdClass`定义结合多个字段的复合键（在我们的示例中未使用）。如果主键在数据库中是自动生成的，可以在该字段前面放置`@GeneratedValue`注解。

最后，实体类必须有一个无参数的构造函数。有了所有这些注解，实体类`Person`现在看起来如下：

```java
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Person1 {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    public int age;
    private String name;
    public Person1(){}
    public Person1(int age, String name){
        this.age = age;
        this.name = name;
    }
}
```

类和其持久化实例变量都不能声明为 final。这样，实现框架可以扩展实体类并实现所需的功能。

或者，持久化注解可以添加到 getter 和 setter 中，而不是实例字段（如果方法名遵循 Java bean 约定）。但是不允许混合字段和方法注解，可能会导致意想不到的后果。

也可以使用 XML 文件而不是注解来定义 Java 类与数据库表和列之间的映射，但我们打算使用字段级别的注解，以提供最简洁和清晰的方法来表达意图。

1.  使用以下 SQL 脚本创建名为`person1`的数据库表：

```java
create table person1 (
   id SERIAL PRIMARY KEY,
   age INTEGER NOT NULL,
   name VARCHAR NOT NULL
);
```

我们将列`id`定义为`SERIAL`，这意味着我们要求数据库在每次向`person1`表插入新行时自动生成下一个整数值。它与类`Person1`的属性`id`的注解相匹配。

1.  现在，让我们编写一些代码，将一条记录插入到`person1`表中，然后从中读取所有记录。要创建、更新和删除实体（以及相应表中的记录），需要使用`javax.persistence.EntityManager`等实体管理器：

```java
EntityManagerFactory emf = 
      Persistence.createEntityManagerFactory("jpa-demo");
EntityManager em = emf.createEntityManager();
try {
    em.getTransaction().begin();
    Person1 p = new Person1(10, "Name10");
    em.persist(p);
    em.getTransaction().commit();

    Query q = em.createQuery("select p from Person1 p");
    List<Person1> pList = q.getResultList();
    for (Person1 p : pList) {
        System.out.println(p);
    }
    System.out.println("Size: " + pList.size());
} catch (Exception ex){
    em.getTransaction().rollback();
} finally {
    em.close();
    emf.close();
} 
```

正如你所看到的，使用一些配置创建了`EntityManagerFactory`对象，即`jpa-demo`。我们很快会谈到它。工厂允许创建一个`EntityManager`对象，它控制持久化过程：创建、提交和回滚事务，存储一个新的`Person1`对象（从而在`person1`表中插入新记录），支持使用**Java 持久化查询语言**（**JPQL**）读取数据，以及许多其他数据库操作和事务管理过程。

实体管理器关闭后，托管实体处于分离状态。要再次与数据库同步，可以使用`EntityManager`的`merge()`方法。

在前面的示例中，我们使用 JPQL 来查询数据库。或者，我们也可以使用 JPA 规范定义的 Criteria API：

```java
CriteriaQuery<Person1> cq = 
       em.getCriteriaBuilder().createQuery(Person1.class);
cq.select(cq.from(Person1.class));
List<Person1> pList = em.createQuery(cq).getResultList();
System.out.println("Size: " + pList.size());

```

但似乎 JPQL 更简洁，支持那些了解 SQL 的程序员的直觉，所以我们打算使用 JPQL。

1.  在`resources/META-INF`文件夹中的`persistence.xml`文件中定义持久化配置。标签`<persistence-unit>`有一个名称属性。我们将属性值设置为`jpa-demo`，但你可以使用任何其他你喜欢的名称。这个配置指定了 JPA 实现（提供者）、数据库连接属性以及许多其他持久化相关属性，以 XML 格式表示：

```java
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence
    http://xmlns.jcp.org/xml/ns/persistence/persistence_2_1.xsd"
                                                   version="2.1">
   <persistence-unit name="jpa-demo" 
                               transaction-type="RESOURCE_LOCAL">
     <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
     <properties>
       <property name="javax.persistence.jdbc.url" 
                   value="jdbc:postgresql://localhost/cookbook"/>
       <property name="javax.persistence.jdbc.driver" 
                                  value="org.postgresql.Driver"/>
       <property name="javax.persistence.jdbc.user" value="cook"/>
       <property name="javax.persistence.jdbc.password" value=""/>
     </properties>
   </persistence-unit>
</persistence>
```

参考 Oracle 文档（[`docs.oracle.com/cd/E16439_01/doc.1013/e13981/cfgdepds005.htm`](https://docs.oracle.com/cd/E16439_01/doc.1013/e13981/cfgdepds005.htm)）关于`persistence.xml`文件的配置。对于这个示例，我们使用了 Hibernate ORM，并指定了`org.hibernate.jpa.HibernatePersistenceProvider`作为提供者。

1.  最后，我们需要在`pom.xml`中将 JPA 实现（Hibernate ORM）作为依赖项添加：

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.3.1.Final</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.3.0</version>
</dependency>

```

正如你可能已经注意到的，我们已经将 Hibernate 依赖项标记为`runtime`作用域。我们这样做是为了在编写代码时避免使用 Hibernate 特定的功能。我们还添加了`jaxb-api`依赖项，这是 Hibernate 使用的，但这个库不是 Hibernate 特定的，所以我们没有将它仅在运行时使用。

1.  为了更好地呈现结果，我们将向`Person1`类添加以下`toString()`方法：

```java
@Override
public String toString() {
    return "Person1{id=" + id + ", age=" + age +
                             ", name='" + name + "'}";
}
```

1.  现在，我们可以运行我们的 JPA 代码示例并观察输出：

```java
Person1{id=1, age=10, name='Name10'}
Size: 1
Size: 1
```

前面输出的前两行来自 JPQL 的使用，最后一行来自我们代码示例的 Criteria API 的使用片段。

1.  JPA 还提供了建立类之间关系的规定。一个实体类（以及相应的数据库表）可以与另一个实体类（以及它的表）建立一对一、一对多、多对一和多对多的关系。关系可以是双向的或单向的。该规范定义了双向关系的以下规则：

+   反向方必须使用注解`@OneToOne`、`@OneToMany`或`@ManyToMany`的`mappedBy`属性引用它的拥有方。

+   一对多和多对一关系的多方必须拥有这个关系，因此`mappedBy`属性不能在`@ManyToOne`注解上指定。

+   在一对一关系中，拥有方是包含外键的一方。

+   在多对多关系中，任何一方都可以是拥有方。

在单向关系中，只有一个类引用另一个类。

为了说明这些规则，让我们创建一个名为`Family`的类，它与类`Person2`有一对多的关系：

```java
@Entity
public class Family {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private String name;
    public Family(){}
    public Family(String name){ this.name = name;}

    @OneToMany(mappedBy = "family")
    private final List<Person2> members = new ArrayList<>();

    public List<Person2> getMembers(){ return this.members; }
    public String getName() { return name; }
}
```

创建表`family`的 SQL 脚本非常简单：

```java
create table family (
    id SERIAL PRIMARY KEY,
    name VARCHAR NOT NULL
);
```

我们还需要将`Family`字段添加到`Person2`类中：

```java
@Entity
public class Person2 {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;
    private int age;
    private String name;

    @ManyToOne
    private Family family;

    public Person2(){}
    public Person2(int age, String name, Family family){
        this.age = age;
        this.name = name;
        this.family = family;
    }
    @Override
    public String toString() {
      return "Person2{id=" + id + ", age=" + age +
               ", name='" + name + "', family='" +
         (family == null ? "" : family.getName())+ "'}";
    }
}
```

`Person2`类是“多”方，因此根据这个规则，它拥有关系，所以表`person2`必须有一个指向表`family`记录的外键：

```java
create table person2 (
    id SERIAL PRIMARY KEY,
    age INTEGER NOT NULL,
    name VARCHAR NOT NULL,
    family_id INTEGER references family(id) 
                             ON DELETE CASCADE
);
```

引用列需要这个列的值是唯一的。这就是为什么我们将表`person2`的列`id`标记为`PRIMARY KEY`。否则，会出现错误`ERROR: 42830: there is no unique constraint matching given keys for referenced table`。

1.  现在，我们可以使用`Family`和`Person2`类来创建相应表中的记录，并从这些表中读取：

```java
EntityManagerFactory emf = 
         Persistence.createEntityManagerFactory("jpa-demo");
    EntityManager em = emf.createEntityManager();
    try {
        em.getTransaction().begin();
        Family f = new Family("The Jones");
        em.persist(f);

        Person2 p = new Person2(10, "Name10", f);  
        em.persist(p);                      

        f.getMembers().add(p);
        em.getTransaction().commit();

        Query q = em.createQuery("select f from Family f");
        List<Family> fList = q.getResultList();
        for (Family f1 : fList) {
            System.out.println("Family " + f1.getName() + ": " 
                      + f1.getMembers().size() + " members:");
            for(Person2 p1: f1.getMembers()){
                System.out.println("   " + p1);
            }
        }
        q = em.createQuery("select p from Person2 p");
        List<Person2> pList = q.getResultList();
        for (Person2 p1 : pList) {
            System.out.println(p1);
        }
    } catch (Exception ex){
        ex.printStackTrace();
        em.getTransaction().rollback();
    } finally {
        em.close();
        emf.close();
    }
}
```

在前面的代码中，我们创建了一个`Family`类的对象并将其持久化。这样，对象就从数据库中获得了一个`id`值。然后，我们将它传递给`Person2`类的构造函数，并在多方建立了关系。然后，我们持久化了`Person2`对象（这样它也从数据库中获得了一个`id`），并将其添加到`Family`对象的`members`集合中，这样关系的一方也建立了。为了保留数据，事务必须被提交。当事务被提交时，与`EntityManager`关联的所有实体对象（这些对象被称为处于**管理**状态）会自动持久化。

# 它是如何工作的...

如果我们运行上述代码，结果将如下所示：

```java
Family The Jones: 1 members:
    Person2{id=1, age=10, name='Name10', family='The Jones'}
Person2{id=1, age=10, name='Name10', family='The Jones'}
```

正如你所看到的，这正是我们所期望的——一个`Family The Jones`类的对象有一个成员——一个`Person2`类的对象，并且表`person2`中的记录指向表`family`中对应的记录。
