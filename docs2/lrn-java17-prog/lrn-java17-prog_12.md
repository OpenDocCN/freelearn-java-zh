# 第十章：数据库中的数据管理

本章解释并演示了如何使用 Java 应用程序管理数据库中的数据——也就是说，插入、读取、更新和删除数据。它还简要介绍了 **结构化查询语言**（**SQL**）和基本数据库操作，包括如何连接到数据库、如何创建数据库结构、如何使用 SQL 编写数据库表达式以及如何执行这些表达式。

本章将涵盖以下主题：

+   创建数据库

+   创建数据库结构

+   连接到数据库

+   释放连接

+   **创建、读取、更新和删除**（**CRUD**）数据操作

+   使用共享库 JAR 文件访问数据库

到本章结束时，你将能够创建和使用数据库来存储、更新和检索数据，以及创建和使用共享库。

# 技术要求

要能够执行本章提供的代码示例，你需要以下内容：

+   拥有 Microsoft Windows、Apple macOS 或 Linux 操作系统的计算机

+   Java SE 版本 17 或更高

+   你偏好的 IDE 或代码编辑器

关于如何设置 Java SE 和 IntelliJ IDEA 编辑器的说明在 *第一章**，* *Java 17 入门* 中提供。本章的代码示例文件可在 GitHub 上找到（[`github.com/PacktPublishing/Learn-Java-17-Programming.git`](https://github.com/PacktPublishing/Learn-Java-17-Programming.git)），位于 `examples/src/main/java/com/packt/learnjava/ch10_database` 文件夹中，以及 `database` 文件夹中，作为共享库的独立项目。

# 创建数据库

`java.sql`, `javax.sql`, 和 `java.transaction.xa` 包以及实现数据库访问接口（称为 **数据库驱动程序**）的特定数据库类，这些由每个数据库供应商提供。

使用 JDBC 意味着编写 Java 代码，使用 JDBC API 的接口和类以及特定数据库的驱动程序来管理数据库中的数据，该驱动程序知道如何与特定数据库建立连接。使用此连接，应用程序可以随后发出用 SQL 编写的请求。

自然地，我们这里只指的是理解 SQL 的数据库。它们被称为关系型或表格 **数据库管理系统**（**DBMS**），构成了目前使用的 DBMS 中的绝大多数——尽管也使用了一些替代方案（例如，导航数据库和 NoSQL）。

`java.sql` 和 `javax.sql` 包包含在 `javax.sql` 包中，它包含支持语句池、分布式事务和行集的 `DataSource` 接口。

创建数据库涉及以下八个步骤：

1.  按照供应商的说明安装数据库。

1.  打开 PL/SQL 终端并创建数据库用户、数据库、模式、表、视图、存储过程以及支持应用程序数据模型所需的一切。

1.  向此应用程序添加对具有数据库特定驱动程序的`.jar`文件的依赖。

1.  从应用程序连接到数据库。

1.  构建 SQL 语句。

1.  执行 SQL 语句。

1.  根据您的应用程序需求使用执行结果。

1.  释放（即关闭）数据库连接以及在此过程中打开的任何其他资源。

*步骤 1*到*3*在数据库设置期间以及应用程序运行之前只执行一次。*步骤 4*到*8*根据需要由应用程序重复执行。实际上，*步骤 5*到*7*可以使用相同的数据库连接重复多次。

对于我们的示例，我们将使用 PostgreSQL 数据库。您首先需要根据数据库特定的说明自行执行*步骤 1*到*3*。为了创建我们的演示数据库，我们使用以下 PL/SQL 命令：

```java
create user student SUPERUSER;
create database learnjava owner student;
```

这些命令创建了一个可以管理`SUPERUSER`数据库所有方面的`student`用户，并将`student`用户设置为`learnjava`数据库的所有者。我们将使用`student`用户从 Java 代码访问和管理数据。在实际应用中，出于安全考虑，应用程序不允许创建或更改数据库表和其他数据库结构方面。

此外，创建另一个逻辑层，称为**模式**，它可以有自己的用户和权限集是一个好习惯。这样，同一数据库中的多个模式可以隔离，每个用户（其中之一是您的应用程序）只能访问某些模式。在企业层面，常见的做法是为数据库模式创建同义词，这样没有任何应用程序可以直接访问原始结构。然而，为了简化，我们在这本书中不这样做。

# 创建数据库结构

数据库创建后，以下三个 SQL 语句将允许您创建和更改数据库结构。这是通过数据库实体（如表、函数或约束）完成的：

+   `CREATE`语句创建数据库实体。

+   `ALTER`语句更改数据库实体。

+   `DROP`语句删除数据库实体。

同样存在各种 SQL 语句，允许您查询每个数据库实体。这些语句是数据库特定的，通常仅在数据库控制台中使用。例如，在 PostgreSQL 控制台中，可以使用`\d <table>`来描述一个表，而`\dt`列出所有表。有关更多详细信息，请参阅您的数据库文档。

要创建一个表，您可以执行以下 SQL 语句：

```java
CREATE TABLE tablename ( column1 type1, column2 type2, ... ); 
```

表名、列名和可以使用的值类型的限制取决于特定的数据库。以下是一个在 PostgreSQL 中创建`person`表的命令示例：

```java
CREATE table person ( 
```

```java
   id SERIAL PRIMARY KEY, 
```

```java
   first_name VARCHAR NOT NULL, 
```

```java
   last_name VARCHAR NOT NULL, 
```

```java
   dob DATE NOT NULL );
```

`SERIAL` 关键字表示该字段是一个由数据库在创建新记录时生成的顺序整数。生成顺序整数的其他选项有 `SMALLSERIAL` 和 `BIGSERIAL`；它们在大小和可能值的范围内有所不同：

```java
SMALLSERIAL: 2 bytes, range from 1 to 32,767
```

```java
SERIAL: 4 bytes, range from 1 to 2,147,483,647
```

```java
BIGSERIAL: 8 bytes, range from 1 to 922,337,2036,854,775,807
```

`PRIMARY_KEY` 关键字表示这将是要记录的唯一标识符，并且很可能会用于搜索。数据库为每个主键创建一个索引，以加快搜索过程。索引是一种数据结构，有助于加速表中的数据搜索，而无需检查每个表记录。索引可以包括一个或多个表的列。如果您请求表的描述，您将看到所有现有的索引。

或者，我们可以使用 `first_name`、`last_name` 和 `dob` 的组合来创建一个复合 `PRIMARY KEY` 关键字：

```java
CREATE table person ( 
```

```java
   first_name VARCHAR NOT NULL, 
```

```java
   last_name VARCHAR NOT NULL, 
```

```java
   dob DATE NOT NULL,
```

```java
   PRIMARY KEY (first_name, last_name, dob) ); 
```

然而，存在两个人可能同名且在同一天出生的可能性，因此这样的组合主键不是一个好主意。

`NOT NULL` 关键字对字段施加约束：它不能为空。数据库将为尝试创建一个空字段的记录或从现有记录中删除值的每个尝试引发错误。我们没有设置 `VARCHAR` 类型的列的大小，因此允许这些列存储任何长度的字符串值。

与此类记录匹配的 Java 对象可能由以下 `Person` 类表示：

```java
public class Person {
```

```java
  private int id;
```

```java
  private LocalDate dob;
```

```java
  private String firstName, lastName;
```

```java
  public Person(String firstName, String lastName, 
```

```java
                                                LocalDate dob){
```

```java
    if (dob == null) {
```

```java
      throw new RuntimeException
```

```java
                              ("Date of birth cannot be null");
```

```java
    }
```

```java
    this.dob = dob;
```

```java
    this.firstName = firstName == null ? "" : firstName;
```

```java
    this.lastName = lastName == null ? "" : lastName;
```

```java
  }
```

```java
  public Person(int id, String firstName,
```

```java
                  String lastName, LocalDate dob) {
```

```java
    this(firstName, lastName, dob);
```

```java
    this.id = id;
```

```java
  }
```

```java
  public int getId() { return id; }
```

```java
  public LocalDate getDob() { return dob; }
```

```java
  public String getFirstName() { return firstName;}
```

```java
  public String getLastName() { return lastName; }
```

```java
}
```

如您可能已经注意到的，`Person` 类中有两个构造函数：带有 `id` 和不带 `id` 的。我们将使用接受 `id` 的构造函数来根据现有记录构建一个对象，而另一个构造函数将用于在插入新记录之前创建一个对象。

一旦创建，可以使用 `DROP` 命令来删除表格：

```java
DROP table person;
```

可以使用 `ALTER` SQL 命令更改现有表格；例如，我们可以添加一个地址列：

```java
ALTER table person add column address VARCHAR;
```

如果您不确定是否已经存在此类列，您可以添加 `IF EXISTS` 或 `IF NOT EXISTS`：

```java
ALTER table person add column IF NOT EXISTS address VARCHAR;
```

然而，这种可能性仅存在于 PostgreSQL 9.6 及更高版本中。

在创建数据库表时，另一个重要考虑因素是是否需要添加另一个索引（除了 `PRIMARY KEY` 之外）。例如，我们可以通过添加以下索引来允许对首字母和姓氏进行不区分大小写的搜索：

```java
CREATE index idx_names on person ((lower(first_name), lower(last_name));
```

如果搜索速度提高，我们保留索引；如果没有，可以按照以下方式删除：

```java
 DROP index idx_names;
```

我们将其删除，因为索引有额外的写入和存储空间开销。

如果需要，我们也可以从表中删除列，如下所示：

```java
ALTER table person DROP column address;
```

在我们的示例中，我们遵循 PostgreSQL 的命名约定。如果您使用不同的数据库，我们建议您查找其命名约定并遵循它，以便您创建的名称与自动创建的名称相匹配。

# 连接到数据库

到目前为止，我们使用控制台来执行 SQL 语句。同样，也可以使用 JDBC API 从 Java 代码中执行这些语句。但是，表只创建一次，因此为单次执行编写程序没有太多意义。

然而，数据管理是另一回事。所以，从现在开始，我们将使用 Java 代码来操作数据库中的数据。为了做到这一点，我们首先需要将以下依赖项添加到`database`项目的`pom.xml`文件中：

```java
<dependency> 
```

```java
    <groupId>org.postgresql</groupId> 
```

```java
    <artifactId>postgresql</artifactId> 
```

```java
    <version>42.3.2</version> 
```

```java
</dependency>
```

`example`项目也获得了对这个依赖的访问权限，因为在`example`项目的`pom.xml`文件中，我们有以下对数据库`.jar`文件的依赖项：

```java
<dependency> 
```

```java
    <groupId>com.packt.learnjava</groupId>
```

```java
    <artifactId>database</artifactId>
```

```java
    <version>1.0-SNAPSHOT</version> 
```

```java
</dependency>
```

在运行任何示例之前，请确保通过在`database`文件夹中执行`"mvn clean install"`命令来安装`database`项目。

现在，我们可以从 Java 代码中创建数据库连接，如下所示：

```java
String URL = "jdbc:postgresql://localhost/learnjava";
```

```java
Properties prop = new Properties();
```

```java
prop.put( "user", "student" );
```

```java
// prop.put( "password", "secretPass123" );
```

```java
try {
```

```java
 Connection conn = DriverManager.getConnection(URL, prop);
```

```java
} catch (SQLException ex) {
```

```java
    ex.printStackTrace();
```

```java
}
```

上述代码只是使用`java.sql.DriverManger`类创建连接的一个示例。`prop.put( "password", "secretPass123" )`语句演示了如何使用`java.util.Properties`类为连接提供密码。然而，我们在创建`student`用户时没有设置密码，所以不需要它。

可以向`DriverManager`传递许多其他值来配置连接行为。传入属性的键名对所有主要数据库都是相同的，但其中一些是数据库特定的。所以，阅读你的数据库供应商文档以获取更多详细信息。

或者，如果我们只想传递`user`和`password`，可以使用重载的`DriverManager.getConnection(String url, String user, String password)`版本。保持密码加密是一个好习惯。我们不会演示如何做，但互联网上有许多可参考的指南。

连接到数据库的另一种方式是使用`javax.sql.DataSource`接口。它的实现包含在数据库驱动程序的同一`.jar`文件中。在`PostgreSQL`的情况下，有两个类实现了`DataSource`接口：

+   `org.postgresql.ds.PGSimpleDataSource`

+   `org.postgresql.ds.PGConnectionPoolDataSource`

我们可以使用这些类来代替`DriverManager`。以下代码是使用`PGSimpleDataSource`类创建数据库连接的一个示例：

```java
PGSimpleDataSource source = new PGSimpleDataSource();
```

```java
source.setServerName("localhost");
```

```java
source.setDatabaseName("learnjava");
```

```java
source.setUser("student");
```

```java
//source.setPassword("password");
```

```java
source.setLoginTimeout(10);
```

```java
try {
```

```java
    Connection conn = source.getConnection();
```

```java
} catch (SQLException ex) {
```

```java
    ex.printStackTrace();
```

```java
}
```

使用`PGConnectionPoolDataSource`类允许你在内存中创建一个`Connection`对象池，如下所示：

```java
PGConnectionPoolDataSource source = new PGConnectionPoolDataSource();
```

```java
source.setServerName("localhost");
```

```java
source.setDatabaseName("learnjava");
```

```java
source.setUser("student");
```

```java
//source.setPassword("password");
```

```java
source.setLoginTimeout(10);
```

```java
try {
```

```java
    PooledConnection conn = source.getPooledConnection();
```

```java
    Set<Connection> pool = new HashSet<>();
```

```java
    for(int i = 0; i < 10; i++){
```

```java
        pool.add(conn.getConnection())
```

```java
    }
```

```java
} catch (SQLException ex) {
```

```java
    ex.printStackTrace();
```

```java
}
```

这是一个首选方法，因为创建`Connection`对象需要时间。池化允许你提前创建，并在需要时重用创建的对象。当连接不再需要时，它可以返回到池中并重用。池大小和其他参数可以在配置文件（如 PostgreSQL 的`postgresql.conf`）中设置。

然而，你不需要自己管理连接池。有几个成熟的框架可以为你完成这项工作，例如 HikariCP ([`github.com/brettwooldridge/HikariCP`](https://github.com/brettwooldridge/HikariCP))、Vibur ([`www.vibur.org`](http://www.vibur.org)) 和 Commons DBCP ([`commons.apache.org/proper/commons-dbcp`](https://commons.apache.org/proper/commons-dbcp)) - 这些框架可靠且易于使用。

无论我们选择哪种创建数据库连接的方法，我们都会将其隐藏在 `getConnection()` 方法中，并在所有代码示例中以相同的方式使用它。一旦获取了 `Connection` 类的对象，我们就可以访问数据库，以添加、读取、删除或修改存储的数据。

# 释放连接

保持数据库连接活跃需要大量的资源，例如内存和 CPU，因此，当你不再需要它们时，关闭连接并释放分配的资源是一个好主意。在连接池的情况下，当 `Connection` 对象关闭时，它会被返回到池中，并消耗更少的资源。

在 Java 7 之前，通过在 `finally` 块中调用 `close()` 方法来关闭连接：

```java
try {
```

```java
    Connection conn = getConnection();
```

```java
    //use object conn here
```

```java
} finally { 
```

```java
    if(conn != null){
```

```java
        try {
```

```java
            conn.close();
```

```java
        } catch (SQLException e) {
```

```java
            e.printStackTrace();
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

`finally` 块中的代码总是会被执行，无论 `try` 块中的异常是否被抛出。然而，自从 Java 7 以来，`try-with-resources` 构造也可以在实现 `java.lang.AutoCloseable` 或 `java.io.Closeable` 接口的对象上完成这项工作。由于 `java.sql.Connection` 对象实现了 `AutoCloseable` 接口，我们可以重写之前的代码片段，如下所示：

```java
try (Connection conn = getConnection()) {
```

```java
    //use object conn here
```

```java
} catch(SQLException ex) {
```

```java
    ex.printStackTrace();
```

```java
}    
```

`catch` 子句是必要的，因为 `AutoCloseable` 资源会抛出 `java.sql.SQLException`。

# CRUD 数据

有四种类型的 SQL 语句可以读取或操作数据库中的数据：

+   `INSERT` 语句用于向数据库中添加数据。

+   `SELECT` 语句用于从数据库中读取数据。

+   `UPDATE` 语句用于更改数据库中的数据。

+   `DELETE` 语句用于从数据库中删除数据。

可以向前面的语句添加一个或多个不同的子句来标识请求的数据（例如 `WHERE` 子句）以及结果必须返回的顺序（例如 `ORDER` 子句）。

JDBC 连接由 `java.sql.Connection` 表示。这，以及其他一些方法，允许你创建三种类型的对象，这些对象可以执行提供不同功能的 SQL 语句：

+   `java.sql.Statement`：这仅仅是将语句发送到数据库服务器以执行。

+   `java.sql.PreparedStatement`：通过允许以高效的方式多次使用不同的参数执行，它可以在数据库服务器上缓存语句并指定一定的执行路径。

+   `java.sql.CallableStatement`：这可以在数据库中执行存储过程。

在本节中，我们将回顾如何在 Java 代码中实现它。最佳实践是在程序化使用之前在数据库控制台中测试 SQL 语句。

## `INSERT` 语句

`INSERT` 语句在数据库中创建（填充）数据，并具有以下格式：

```java
INSERT into table_name (column1, column2, column3,...) 
```

```java
                values (value1, value2, value3,...); 
```

或者，当需要添加多个记录时，可以使用以下格式：

```java
INSERT into table_name (column1, column2, column3,...) 
```

```java
                values (value1, value2, value3,... ), 
```

```java
                       (value21, value22, value23,...),
```

```java
                       ...; 
```

## `SELECT` 语句

`SELECT` 语句具有以下格式：

```java
SELECT column_name, column_name FROM table_name 
```

```java
                        WHERE some_column = some_value;
```

或者，当需要选择所有列时，可以使用以下格式：

```java
SELECT * from table_name WHERE some_column=some_value; 
```

`WHERE` 子句的更一般定义如下：

```java
WHERE column_name operator value 
```

```java
Operator: 
```

```java
= Equal 
```

```java
<> Not equal. In some versions of SQL, != 
```

```java
> Greater than 
```

```java
< Less than 
```

```java
>= Greater than or equal 
```

```java
<= Less than or equal IN Specifies multiple possible values for a column 
```

```java
LIKE Specifies the search pattern 
```

```java
BETWEEN Specifies the inclusive range of values in a column 
```

构造的 `column_name` 操作符值可以使用 `AND` 和 `OR` 逻辑运算符组合，并通过括号分组，`( )`。

例如，以下方法从 `person` 表中提取所有名字值（由空白字符分隔）：

```java
String selectAllFirstNames() {
```

```java
    String result = "";
```

```java
    Connection conn = getConnection();
```

```java
    try (conn; Statement st = conn.createStatement()) {
```

```java
      ResultSet rs = 
```

```java
        st.executeQuery("select first_name from person");
```

```java
      while (rs.next()) {
```

```java
          result += rs.getString(1) + " ";
```

```java
      }
```

```java
    } catch (SQLException ex) {
```

```java
        ex.printStackTrace();
```

```java
    }
```

```java
    return result;
```

```java
}
```

`ResultSet` 接口的 `getString(int position)` 方法从位置 `1`（`SELECT` 语句中列列表中的第一个）提取 `String` 值。对于所有原始类型都有类似的获取器：`getInt(int position)`、`getByte(int position)` 等。

还可以从 `ResultSet` 对象中按列名提取值。在我们的例子中，它将是 `getString("first_name")`。当 `SELECT` 语句如下时，这种方法获取值特别有用：

```java
select * from person;
```

然而，请注意，使用列名从 `ResultSet` 对象中提取值效率较低。然而，性能差异非常小，只有在操作多次时才会变得重要。只有实际的测量和测试过程才能告诉你这种差异是否对你的应用程序具有重要意义。通过列名提取值特别吸引人，因为它提供了更好的代码可读性，这在长期的应用程序维护中会带来回报。

`ResultSet` 接口中还有许多其他有用的方法。如果你的应用程序从数据库中读取数据，我们强烈建议你阅读你使用的数据库版本的 `SELECT` 语句和 `ResultSet` 接口的官方文档 ([www.postgresql.org/docs](http://www.postgresql.org/docs))。

## `UPDATE` 语句

数据可以通过 `UPDATE` 语句进行更改，如下所示：

```java
UPDATE table_name SET column1=value1,column2=value2,... WHERE clause;
```

我们可以使用此语句将记录中的一个名字从原始值 `John` 更改为新值 `Jim`：

```java
update person set first_name = 'Jim' where last_name = 'Adams';
```

没有使用 `WHERE` 子句，表中的所有记录都将受到影响。

## `DELETE` 语句

要从表中删除记录，请使用 `DELETE` 语句，如下所示：

```java
DELETE FROM table_name WHERE clause;
```

没有使用 `WHERE` 子句，将删除表中的所有记录。在 `person` 表的情况下，我们可以使用以下 SQL 语句来删除所有记录：

```java
delete from person;
```

此外，此语句仅删除具有 `Jim` 为名字的首个记录：

```java
delete from person where first_name = 'Jim';
```

## 使用语句

`java.sql.Statement` 接口提供了以下方法来执行 SQL 语句：

+   `boolean execute(String sql)`：如果执行语句返回数据（在 `java.sql.ResultSet` 对象中），并且可以使用 `java.sql.Statement` 接口的 `ResultSet getResultSet()` 方法检索，则返回 `true`。否则，如果执行语句不返回数据（对于 `INSERT` 语句或 `UPDATE` 语句），并且对 `java.sql.Statement` 接口的 `int getUpdateCount()` 方法的后续调用返回受影响的行数，则返回 `false`。

+   `ResultSet executeQuery(String sql)`：返回数据作为 `java.sql.ResultSet` 对象（与此方法一起使用的 SQL 语句通常是 `SELECT` 语句）。`java.sql.Statement` 接口的 `ResultSet getResultSet()` 方法不返回数据，而 `java.sql.Statement` 接口的 `int getUpdateCount()` 方法返回 `-1`。

+   `int executeUpdate(String sql)`：返回受影响的行数（预期的 SQL 语句是 `UPDATE` 语句或 `DELETE` 语句）。`java.sql.Statement` 接口的 `int getUpdateCount()` 方法返回相同的数字；对 `java.sql.Statement` 接口的 `ResultSet getResultSet()` 方法的后续调用返回 `null`。

我们将演示这三个方法如何在每个语句上工作：`INSERT`、`SELECT`、`UPDATE` 和 `DELETE`。

### `execute(String sql)` 方法

让我们尝试执行每个语句；我们首先从 `INSERT` 语句开始：

```java
String sql = 
```

```java
   "insert into person (first_name, last_name, dob) " +
```

```java
                "values ('Bill', 'Grey', '1980-01-27')";
```

```java
Connection conn = getConnection();
```

```java
try (conn; Statement st = conn.createStatement()) {
```

```java
    System.out.println(st.execute(sql)); //prints: false
```

```java
    System.out.println(st.getResultSet() == null); 
```

```java
                                                 //prints: true
```

```java
    System.out.println(st.getUpdateCount()); //prints: 1
```

```java
} catch (SQLException ex) {
```

```java
    ex.printStackTrace();
```

```java
}
```

```java
System.out.println(selectAllFirstNames()); //prints: Bill
```

上述代码向 `person` 表添加了一条新记录。返回的 `false` 值表示执行语句没有返回数据；这就是为什么 `getResultSet()` 方法返回 `null` 的原因。但是，`getUpdateCount()` 方法返回 `1`，因为影响了一条记录（添加）。`selectAllFirstNames()` 方法证明了预期的记录已被插入。

现在，让我们执行 `SELECT` 语句，如下所示：

```java
String sql = "select first_name from person";
```

```java
Connection conn = getConnection();
```

```java
try (conn; Statement st = conn.createStatement()) {
```

```java
    System.out.println(st.execute(sql));    //prints: true
```

```java
    ResultSet rs = st.getResultSet();
```

```java
    System.out.println(rs == null);             //prints: false
```

```java
    System.out.println(st.getUpdateCount());    //prints: -1
```

```java
    while (rs.next()) {
```

```java
        System.out.println(rs.getString(1) + " "); 
```

```java
                                                 //prints: Bill
```

```java
    }
```

```java
} catch (SQLException ex) {
```

```java
    ex.printStackTrace();
```

```java
}
```

上述代码从 `person` 表中选择所有姓名。返回的 `true` 值表示执行语句返回了数据。这就是为什么 `getResultSet()` 方法不返回 `null`，而是返回一个 `ResultSet` 对象。`getUpdateCount()` 方法返回 `-1`，因为没有记录受到影响（更改）。由于 `person` 表中只有一个记录，`ResultSet` 对象只包含一个结果，`rs.getString(1)` 返回 `Bill`。

以下代码使用 `UPDATE` 语句将 `person` 表中所有记录的姓名更改为 `Adam`：

```java
String sql = "update person set first_name = 'Adam'";
```

```java
Connection conn = getConnection();
```

```java
try (conn; Statement st = conn.createStatement()) {
```

```java
    System.out.println(st.execute(sql));  //prints: false
```

```java
    System.out.println(st.getResultSet() == null); 
```

```java
                                           //prints: true
```

```java
    System.out.println(st.getUpdateCount());  //prints: 1
```

```java
} catch (SQLException ex) {
```

```java
    ex.printStackTrace();
```

```java
}
```

```java
System.out.println(selectAllFirstNames()); //prints: Adam
```

在上述代码中，返回的 `false` 值表示执行语句没有返回数据。这就是为什么 `getResultSet()` 方法返回 `null` 的原因。但是，`getUpdateCount()` 方法返回 `1`，因为影响了一条记录（更改），由于 `person` 表中只有一个记录。`selectAllFirstNames()` 方法证明了预期的更改已应用于记录。

以下 `DELETE` 语句执行删除了 `person` 表中的所有记录：

```java
String sql = "delete from person";
```

```java
Connection conn = getConnection();
```

```java
try (conn; Statement st = conn.createStatement()) {
```

```java
    System.out.println(st.execute(sql));  //prints: false
```

```java
    System.out.println(st.getResultSet() == null); 
```

```java
                                           //prints: true
```

```java
    System.out.println(st.getUpdateCount());  //prints: 1
```

```java
} catch (SQLException ex) {
```

```java
    ex.printStackTrace();
```

```java
}
```

```java
System.out.println(selectAllFirstNames());  //prints: 
```

在前面的代码中，返回的 `false` 值表明执行语句没有返回数据。这就是为什么 `getResultSet()` 方法返回 `null` 的原因。但是，`getUpdateCount()` 方法返回 `1`，因为只有一个记录受到影响（被删除），因为 `person` 表中只有一个记录。`selectAllFirstNames()` 方法证明了 `person` 表中没有记录。

### `executeQuery(String sql)` 方法

在本节中，我们将尝试执行与在 *The execute(String sql) method* 部分中演示 `execute()` 方法时使用的相同语句（作为一个查询）。我们将从以下 `INSERT` 语句开始：

```java
String sql = 
```

```java
"insert into person (first_name, last_name, dob) " +
```

```java
              "values ('Bill', 'Grey', '1980-01-27')";
```

```java
Connection conn = getConnection();
```

```java
try (conn; Statement st = conn.createStatement()) {
```

```java
    st.executeQuery(sql);         //PSQLException
```

```java
} catch (SQLException ex) {
```

```java
    ex.printStackTrace();         //prints: stack trace 
```

```java
}
```

```java
System.out.println(selectAllFirstNames()); //prints: Bill
```

前面的代码由于查询没有返回结果而抛出异常，因为 `executeQuery()` 方法期望执行 `SELECT` 语句。尽管如此，`selectAllFirstNames()` 方法证明了预期的记录已被插入。

现在，让我们执行以下 `SELECT` 语句：

```java
String sql = "select first_name from person";
```

```java
Connection conn = getConnection();
```

```java
try (conn; Statement st = conn.createStatement()) {
```

```java
    ResultSet rs1 = st.executeQuery(sql);
```

```java
    System.out.println(rs1 == null);     //prints: false
```

```java
    ResultSet rs2 = st.getResultSet();
```

```java
    System.out.println(rs2 == null);     //prints: false
```

```java
    System.out.println(st.getUpdateCount()); //prints: -1
```

```java
    while (rs1.next()) {
```

```java
        System.out.println(rs1.getString(1)); //prints: Bill
```

```java
    }
```

```java
    while (rs2.next()) {
```

```java
        System.out.println(rs2.getString(1)); //prints:
```

```java
    }
```

```java
} catch (SQLException ex) {
```

```java
    ex.printStackTrace();
```

```java
}
```

前面的代码从 `person` 表中选择所有名字。返回的 `false` 值表明 `executeQuery()` 总是返回 `ResultSet` 对象，即使 `person` 表中没有记录。如您所见，似乎有两种从执行语句获取结果的方法。然而，`rs2` 对象没有数据，所以在使用 `executeQuery()` 方法时，请确保您从 `ResultSet` 对象中获取数据。

现在，让我们尝试执行以下 `UPDATE` 语句：

```java
String sql = "update person set first_name = 'Adam'";
```

```java
Connection conn = getConnection();
```

```java
try (conn; Statement st = conn.createStatement()) {
```

```java
    st.executeQuery(sql);           //PSQLException
```

```java
} catch (SQLException ex) {
```

```java
    ex.printStackTrace();           //prints: stack trace
```

```java
}
```

```java
System.out.println(selectAllFirstNames()); //prints: Adam
```

前面的代码由于查询没有返回结果而抛出异常，因为 `executeQuery()` 方法期望执行 `SELECT` 语句。尽管如此，`selectAllFirstNames()` 方法证明了预期的更改已应用于记录。

在执行 `DELETE` 语句时，我们将遇到相同的异常：

```java
String sql = "delete from person";
```

```java
Connection conn = getConnection();
```

```java
try (conn; Statement st = conn.createStatement()) {
```

```java
    st.executeQuery(sql);           //PSQLException
```

```java
} catch (SQLException ex) {
```

```java
    ex.printStackTrace();           //prints: stack trace
```

```java
}
```

```java
System.out.println(selectAllFirstNames()); //prints: 
```

尽管如此，`selectAllFirstNames()` 方法证明了 `person` 表中的所有记录都被删除了。

我们的演示表明，`executeQuery()` 应仅用于 `SELECT` 语句。`executeQuery()` 方法的优点是，当用于 `SELECT` 语句时，即使没有选择数据，它也会返回一个非空的 `ResultSet` 对象，这简化了代码，因为不需要检查返回值是否为 `null`。

### `executeUpdate(String sql)` 方法

我们将首先使用 `INSERT` 语句来演示 `executeUpdate()` 方法：

```java
String sql = 
```

```java
"insert into person (first_name, last_name, dob) " +
```

```java
               "values ('Bill', 'Grey', '1980-01-27')";
```

```java
Connection conn = getConnection();
```

```java
try (conn; Statement st = conn.createStatement()) {
```

```java
   System.out.println(st.executeUpdate(sql)); //prints: 1
```

```java
   System.out.println(st.getResultSet());  //prints: null
```

```java
   System.out.println(st.getUpdateCount());  //prints: 1
```

```java
} catch (SQLException ex) {
```

```java
    ex.printStackTrace();
```

```java
}
```

```java
System.out.println(selectAllFirstNames()); //prints: Bill
```

如您所见，`executeUpdate()` 方法返回受影响的（在本例中为插入的）行数。相同的数字由 `int getUpdateCount()` 方法返回，而 `ResultSet getResultSet()` 方法返回 `null`。`selectAllFirstNames()` 方法证明了预期的记录已被插入。

`executeUpdate()` 方法不能用于执行 `SELECT` 语句：

```java
String sql = "select first_name from person";
```

```java
Connection conn = getConnection();
```

```java
try (conn; Statement st = conn.createStatement()) {
```

```java
    st.executeUpdate(sql);    //PSQLException
```

```java
} catch (SQLException ex) {
```

```java
    ex.printStackTrace();     //prints: stack trace
```

```java
}
```

异常信息为 `A result was returned when none was expected`。

另一方面，`UPDATE` 语句可以通过 `executeUpdate()` 方法正常执行：

```java
String sql = "update person set first_name = 'Adam'";
```

```java
Connection conn = getConnection();
```

```java
try (conn; Statement st = conn.createStatement()) {
```

```java
  System.out.println(st.executeUpdate(sql));  //prints: 1
```

```java
  System.out.println(st.getResultSet());   //prints: null
```

```java
    System.out.println(st.getUpdateCount());    //prints: 1
```

```java
} catch (SQLException ex) {
```

```java
    ex.printStackTrace();
```

```java
}
```

```java
System.out.println(selectAllFirstNames());    //prints: Adam
```

`executeUpdate()` 方法返回受影响的（在这种情况下是更新）行数。相同的数字由 `int getUpdateCount()` 方法返回，而 `ResultSet getResultSet()` 方法返回 `null`。`selectAllFirstNames()` 方法证明了预期的记录已被更新。

`DELETE` 语句产生类似的结果：

```java
String sql = "delete from person";
```

```java
Connection conn = getConnection();
```

```java
try (conn; Statement st = conn.createStatement()) {
```

```java
    System.out.println(st.executeUpdate(sql));  //prints: 1
```

```java
    System.out.println(st.getResultSet());      //prints: null
```

```java
    System.out.println(st.getUpdateCount());    //prints: 1
```

```java
} catch (SQLException ex) {
```

```java
    ex.printStackTrace();
```

```java
}
```

```java
System.out.println(selectAllFirstNames());      //prints:
```

到现在为止，你可能已经意识到 `executeUpdate()` 方法更适合 `INSERT`、`UPDATE` 和 `DELETE` 语句。

## 使用 PreparedStatement

`PreparedStatement` 是 `Statement` 接口的一个子接口。这意味着它可以在任何使用 `Statement` 接口的地方使用。`PreparedStatement` 的优点是它在数据库中缓存，而不是每次调用时都编译。这样，它可以针对不同的输入值高效地执行多次。可以通过使用相同的 `Connection` 对象的 `prepareStatement()` 方法来创建它。

由于相同的 SQL 语句可以用于创建 `Statement` 和 `PreparedStatement`，因此对于任何多次调用的 SQL 语句使用 `PreparedStatement` 是一个好主意，因为它在数据库端的表现优于 `Statement` 接口。为此，我们只需要更改前面代码示例中的这两行：

```java
try (conn; Statement st = conn.createStatement()) { 
```

```java
     ResultSet rs = st.executeQuery(sql);
```

相反，我们可以使用 `PreparedStatement` 类，如下所示：

```java
try (conn; PreparedStatement st = conn.prepareStatement(sql)) { 
```

```java
     ResultSet rs = st.executeQuery();
```

要使用参数创建 `PreparedStatement` 对象，可以将输入值替换为问号符号（`?`）；例如，我们可以创建以下方法（参见 `database` 项目中的 `Person` 类）：

```java
private static final String SELECT_BY_FIRST_NAME = 
```

```java
            "select * from person where first_name = ?";
```

```java
static List<Person> selectByFirstName(Connection conn, 
```

```java
                                     String searchName) {
```

```java
    List<Person> list = new ArrayList<>();
```

```java
    try (PreparedStatement st = 
```

```java
         conn.prepareStatement(SELECT_BY_FIRST_NAME)) {
```

```java
       st.setString(1, searchName);
```

```java
       ResultSet rs = st.executeQuery();
```

```java
       while (rs.next()) {
```

```java
           list.add(new Person(rs.getInt("id"),
```

```java
                    rs.getString("first_name"),
```

```java
                    rs.getString("last_name"),
```

```java
                    rs.getDate("dob").toLocalDate()));
```

```java
       }
```

```java
   } catch (SQLException ex) {
```

```java
        ex.printStackTrace();
```

```java
   }
```

```java
   return list;
```

```java
}
```

当它第一次使用时，数据库将 `PreparedStatement` 对象编译为模板并存储它。然后，当它稍后被应用程序再次使用时，参数值被传递到模板，并且由于已经编译过，因此无需编译开销即可立即执行语句。

预处理语句的另一个优点是它更好地保护免受 SQL 注入攻击，因为值是通过不同的协议传递的，并且模板不是基于外部输入的。

如果预处理语句只使用一次，它可能比常规语句慢，但差异可能微不足道。如果有疑问，测试性能并查看它是否适合你的应用程序——提高的安全性可能是值得的。

## 使用 CallableStatement

`CallableStatement` 接口（它扩展了 `PreparedStatement` 接口）可以用来执行存储过程，尽管一些数据库允许你使用 `Statement` 或 `PreparedStatement` 接口来调用存储过程。`CallableStatement` 对象是通过 `prepareCall()` 方法创建的，并且可以有三个类型的参数：

+   `IN` 用于输入值

+   `OUT` 用于结果

+   `IN OUT`用于输入或输出值

`IN`参数可以像`PreparedStatement`的参数一样设置，而`OUT`参数必须通过`CallableStatement`的`registerOutParameter()`方法进行注册。

值得注意的是，从 Java 程序中执行存储过程是标准化程度最低的领域之一。例如，PostgreSQL 不支持存储过程，但它们可以作为已经为此目的修改过的函数来调用，将`OUT`参数解释为返回值。另一方面，Oracle 允许将`OUT`参数作为函数使用。

这就是为什么数据库函数和存储过程之间的以下差异只能作为一般性指南，而不能作为正式定义：

+   函数有一个返回值，但它不允许`OUT`参数（除了某些数据库）并且可以在 SQL 语句中使用。

+   存储过程没有返回值（除了某些数据库）；它允许`OUT`参数（对于大多数数据库）并且可以使用 JDBC 的`CallableStatement`接口执行。

您可以参考数据库文档来了解如何执行存储过程。

由于存储过程是在数据库服务器上编译和存储的，因此`CallableStatement`的`execute()`方法对于相同的 SQL 语句比`Statement`或`PreparedStatement`接口的相应方法性能更好。这就是为什么很多 Java 代码有时被一个或多个包含业务逻辑的存储过程所取代的原因之一。然而，对于每一个案例和问题都没有一个正确的答案，所以我们不会提出具体的建议，除了重复熟悉的咒语关于测试的价值和您所编写的代码的清晰性：

```java
String replace(String origText, String substr1, String substr2) {
```

```java
    String result = "";
```

```java
    String sql = "{ ? = call replace(?, ?, ? ) }";
```

```java
    Connection conn = getConnection();
```

```java
    try (conn; CallableStatement st = conn.prepareCall(sql)) {
```

```java
        st.registerOutParameter(1, Types.VARCHAR);
```

```java
        st.setString(2, origText);
```

```java
        st.setString(3, substr1);
```

```java
        st.setString(4, substr2);
```

```java
        st.execute();
```

```java
        result = st.getString(1);
```

```java
    } catch (Exception ex){
```

```java
        ex.printStackTrace();
```

```java
    }
```

```java
    return result;
```

```java
}
```

现在，我们可以按照以下方式调用此方法：

```java
String result = replace("That is original text",
```

```java
                         "original text", "the result");
```

```java
System.out.println(result);  //prints: That is the result
```

存储过程可以没有任何参数，只有`IN`参数，只有`OUT`参数，或者两者都有。结果可能是一个或多个值，或者一个`ResultSet`对象。您可以在数据库文档中找到创建函数的 SQL 语法的语法。

## 使用共享库 JAR 文件访问数据库

事实上，我们已经开始使用`database`项目的 JAR 文件来访问数据库驱动程序，将其设置为`database`项目的`pom.xml`文件中的依赖项。现在，我们将演示如何使用`database`项目 JAR 文件来操作数据库中的数据。这种使用的示例在`UseDatabaseJar`类中给出。

为了支持 CRUD 操作，数据库表通常代表一个对象类。此类表中的每一行都包含一个对象的一个类的属性。在*创建数据库结构*部分，我们展示了`Person`类和`person`表之间此类映射的示例。为了说明如何使用 JAR 文件进行数据操作，我们创建了一个单独的`database`项目，该项目只有一个`Person`类。除了*创建数据库结构*部分中显示的属性外，它还包含所有 CRUD 操作的静态方法。以下是对`insert()`方法的实现：

```java
static final String INSERT = "insert into person " +
```

```java
  "(first_name, last_name, dob) values (?, ?, ?::date)";
```

```java
static void insert(Connection conn, Person person) {
```

```java
   try (PreparedStatement st = 
```

```java
                       conn.prepareStatement(INSERT)) {
```

```java
            st.setString(1, person.getFirstName());
```

```java
            st.setString(2, person.getLastName());
```

```java
            st.setString(3, person.getDob().toString());
```

```java
            st.execute();
```

```java
   } catch (SQLException ex) {
```

```java
            ex.printStackTrace();
```

```java
   }
```

```java
}
```

以下是对`selectByFirstName()`方法的实现：

```java
private static final String SELECT = 
```

```java
          "select * from person where first_name = ?";
```

```java
static List<Person> selectByFirstName(Connection conn, 
```

```java
                                    String firstName) {
```

```java
   List<Person> list = new ArrayList<>();
```

```java
   try (PreparedStatement st = conn.prepareStatement(SELECT)) {
```

```java
        st.setString(1, firstName);
```

```java
        ResultSet rs = st.executeQuery();
```

```java
        while (rs.next()) {
```

```java
            list.add(new Person(rs.getInt("id"),
```

```java
                    rs.getString("first_name"),
```

```java
                    rs.getString("last_name"),
```

```java
                    rs.getDate("dob").toLocalDate()));
```

```java
        }
```

```java
   } catch (SQLException ex) {
```

```java
            ex.printStackTrace();
```

```java
   }
```

```java
   return list;
```

```java
}
```

以下是对`updateFirstNameById()`方法的实现：

```java
private static final String UPDATE = 
```

```java
      "update person set first_name = ? where id = ?";
```

```java
public static void updateFirstNameById(Connection conn, 
```

```java
                           int id, String newFirstName) {
```

```java
   try (PreparedStatement st = conn.prepareStatement(UPDATE)) {
```

```java
            st.setString(1, newFirstName);
```

```java
            st.setInt(2, id);
```

```java
            st.execute();
```

```java
   } catch (SQLException ex) {
```

```java
            ex.printStackTrace();
```

```java
   }
```

```java
}
```

以下是对`deleteById()`方法的实现：

```java
private static final String DELETE = 
```

```java
                       "delete from person where id = ?";
```

```java
public static void deleteById(Connection conn, int id) {
```

```java
   try (PreparedStatement st = conn.prepareStatement(DELETE)) {
```

```java
            st.setInt(1, id);
```

```java
            st.execute();
```

```java
   } catch (SQLException ex) {
```

```java
            ex.printStackTrace();
```

```java
   }
```

```java
}
```

如您所见，所有前面的方法都接受`Connection`对象作为参数，而不是在每个方法内部创建和销毁它。我们决定这样做是因为它允许多个操作与每个`Connection`对象相关联，以防我们希望它们一起提交到数据库或如果其中一个失败则回滚（请参阅您选择的数据库文档中的事务管理部分）。此外，由`database`项目生成的 JAR 文件可以被不同的应用程序使用，因此数据库连接参数将是特定于应用程序的，这就是为什么`Connection`对象必须创建在使用 JAR 文件的应用程序中。以下代码演示了这种用法（请参阅`UseDatabaseJar`类）。

在运行以下示例之前，请确保您已在`database`文件夹中执行了`mvn clean install`命令。

```java
1 try(Connection conn = getConnection()){
```

```java
2    cleanTablePerson(conn);
```

```java
3    Person mike = new Person("Mike", "Brown", 
```

```java
                             LocalDate.of(2002, 8, 14));
```

```java
4    Person jane = new Person("Jane", "McDonald", 
```

```java
                             LocalDate.of(2000, 3, 21));
```

```java
5    Person jill = new Person("Jill", "Grey", 
```

```java
                             LocalDate.of(2001, 4, 1));
```

```java
6    Person.insert(conn, mike);
```

```java
7    Person.insert(conn, jane);
```

```java
8    Person.insert(conn, jane);
```

```java
9    List<Person> persons = 
```

```java
           Person.selectByFirstName(conn, jill.getFirstName());
```

```java
10   System.out.println(persons.size());      //prints: 0
```

```java
11   persons = Person.selectByFirstName(conn, 
```

```java
                                          jane.getFirstName());
```

```java
12   System.out.println(persons.size());      //prints: 2
```

```java
13   Person person = persons.get(0);
```

```java
14   Person.updateFirstNameById(conn, person.getId(),
```

```java
                                          jill.getFirstName());
```

```java
15   persons = Person.selectByFirstName(conn, 
```

```java
                                          jane.getFirstName());
```

```java
16   System.out.println(persons.size());      //prints: 1 
```

```java
17   persons = Person.selectByFirstName(conn, 
```

```java
                                          jill.getFirstName());
```

```java
18   System.out.println(persons.size());      //prints: 1
```

```java
19   persons = Person.selectByFirstName(conn, 
```

```java
                                          mike.getFirstName());
```

```java
20   System.out.println(persons.size());      //prints: 1
```

```java
21   for(Person p: persons){
```

```java
22      Person.deleteById(conn, p.getId());
```

```java
23   }
```

```java
24   persons = Person.selectByFirstName(conn, 
```

```java
                                          mike.getFirstName());
```

```java
25   System.out.println(persons.size());      //prints: 0
```

```java
26 } catch (SQLException ex){
```

```java
27       ex.printStackTrace();
```

```java
28 }
```

让我们逐步分析前面的代码片段。第`1`行和第`26`至`28`行构成了处理`Connection`对象并捕获在此块执行过程中可能发生的所有异常的`try–catch`块。

第`2`行只是为了在运行演示代码之前清理`person`表中的数据。以下是对`cleanTablePerson()`方法的实现：

```java
void cleanTablePerson(Connection conn) {
```

```java
   try (Statement st = conn.createStatement()) {
```

```java
       st.execute("delete from person");
```

```java
   } catch (SQLException ex) {
```

```java
       ex.printStackTrace();
```

```java
   }
```

```java
}
```

在第`3`、`4`和`5`行，我们创建了三个`Person`类的对象，然后在第`6`、`7`和`8`行，我们使用它们在`person`表中插入记录。

在第`9`行，我们查询数据库以获取一个来自`jill`对象的首个名字的记录，在第`10`行，我们打印出结果计数，该计数为`0`（因为我们没有插入这样的记录）。

在第`11`行，我们查询数据库以获取一个首个名字设置为`Jane`的记录，在第`12`行，我们打印出结果计数，该计数为`2`（因为我们确实插入了两个具有此值的记录）。

在第`13`行，我们提取前一个查询返回的两个对象中的第一个，在第`14`行，我们使用来自`jill`对象的不同值更新相应的记录的首个名字。

在第`15`行，我们重复查询名字设置为`Jane`的记录，在第`16`行，我们打印出结果计数，这次是`1`（正如预期的那样，因为我们已经将其中一个记录的名字从`Jane`改为了`Jill`）。

在第`17`行，我们选择所有名字设置为`Jill`的记录，在第`18`行，我们打印出结果计数，这次是`1`（正如预期的那样，因为我们已经将其中一个原本名字为`Jane`的记录的名字改为了`Jill`）。

在第`19`行，我们选择所有名字设置为`Mike`的记录，在第`20`行，我们打印出结果计数，这次是`1`（正如预期的那样，因为我们只创建了一个这样的记录）。

在第`21`到`23`行，我们在循环中删除所有检索到的记录。

因此，当我们再次在第`24`行选择名字为`Mike`的所有记录时，在第`25`行我们得到的结果计数为`0`（正如预期的那样，因为已经没有这样的记录了）。

在这个阶段，当这段代码片段执行完毕，并且`UseDatabseJar`类的`main()`方法完成后，数据库中的所有更改都会自动保存。

这就是 JAR 文件（允许修改数据库中的数据）可以被任何将其作为依赖项的应用程序使用的原理。

# 摘要

在本章中，我们讨论并演示了如何在 Java 应用程序中填充、读取、更新和删除数据库中的数据。对 SQL 语言的简要介绍说明了如何创建数据库及其结构，如何修改它，以及如何使用`Statement`、`PreparedStatement`和`CallableStatement`执行 SQL 语句。

现在，你可以创建和使用数据库来存储、更新和检索数据，以及创建和使用共享库。

在下一章中，我们将描述和讨论最流行的网络协议，演示如何使用它们，以及如何使用最新的 Java HTTP 客户端 API 实现客户端-服务器通信。所审查的协议包括基于 TCP、UDP 和 URL 的通信协议的 Java 实现。

# 问答

1.  选择所有正确的语句：

    1.  JDBC 代表 Java 数据库通信。

    1.  JDBC API 包括`java.db`包。

    1.  JDBC API 与 Java 安装一起提供。

    1.  JDBC API 包括了所有主要数据库管理系统（DBMS）的驱动程序。

1.  选择所有正确的语句：

    1.  可以使用`CREATE`语句创建数据库表。

    1.  可以使用`UPDATE`语句更改数据库表。

    1.  可以使用`DELETE`语句删除数据库表。

    1.  每个数据库列都可以有一个索引。

1.  选择所有正确的语句：

    1.  要连接到数据库，你可以使用`Connect`类。

    1.  每个数据库连接都必须关闭。

    1.  同一个数据库连接可以用于许多操作。

    1.  数据库连接可以被池化。

1.  选择所有正确的语句：

    1.  可以使用`try-with-resources`构造自动关闭数据库连接。

    1.  可以使用 `finally` 块结构关闭数据库连接。

    1.  可以使用 `catch` 块关闭数据库连接。

    1.  可以在不使用 `try` 块的情况下关闭数据库连接。

1.  选择所有正确的说法：

    1.  `INSERT` 语句包含一个表名。

    1.  `INSERT` 语句包含列名。

    1.  `INSERT` 语句包含值。

    1.  `INSERT` 语句包含约束。

1.  选择所有正确的说法：

    1.  `SELECT` 语句必须包含一个表名。

    1.  `SELECT` 语句必须包含一个列名。

    1.  `SELECT` 语句必须包含 `WHERE` 子句。

    1.  `SELECT` 语句可能包含 `ORDER` 子句。

1.  选择所有正确的说法：

    1.  `UPDATE` 语句必须包含一个表名。

    1.  `UPDATE` 语句必须包含一个列名。

    1.  `UPDATE` 语句可能包含 `WHERE` 子句。

    1.  `UPDATE` 语句可能包含 `ORDER` 子句。

1.  选择所有正确的说法：

    1.  `DELETE` 语句必须包含一个表名。

    1.  `DELETE` 语句必须包含一个列名。

    1.  `DELETE` 语句可能包含 `WHERE` 子句。

    1.  `DELETE` 语句可能包含 `ORDER` 子句。

1.  选择关于 `Statement` 接口的 `execute()` 方法的正确说法：

    1.  它接收一个 SQL 语句。

    1.  它返回一个 `ResultSet` 对象。

    1.  在调用 `execute()` 后，`Statement` 对象可能返回数据。

    1.  在调用 `execute()` 后，`Statement` 对象可能返回受影响记录的数量。

1.  选择关于 `Statement` 接口的 `executeQuery()` 方法的正确说法：

    1.  它接收一个 SQL 语句。

    1.  它返回一个 `ResultSet` 对象。

    1.  在调用 `executeQuery()` 后，`Statement` 对象可能返回数据。

    1.  在调用 `executeQuery()` 后，`Statement` 对象可能返回受影响记录的数量。

1.  选择关于 `Statement` 接口的 `executeUpdate()` 方法的正确说法：

    1.  它接收一个 SQL 语句。

    1.  它返回一个 `ResultSet` 对象。

    1.  在调用 `executeUpdate()` 后，`Statement` 对象可能返回数据。

    1.  在调用 `executeUpdate()` 后，`Statement` 对象返回受影响记录的数量。

1.  选择关于 `PreparedStatement` 接口的正确说法：

    1.  它扩展了 `Statement`。

    1.  通过 `prepareStatement()` 方法创建 `PreparedStatement` 类型的对象。

    1.  它总是比 `Statement` 更高效。

    1.  它导致数据库中只创建一次模板。

1.  选择关于 `CallableStatement` 接口的正确说法：

    1.  它扩展了 `PreparedStatement`。

    1.  通过 `prepareCall()` 方法创建 `CallableStatement` 类型的对象。

    1.  它总是比 `PreparedStatement` 更高效。

    1.  它导致数据库中只创建一次模板。
