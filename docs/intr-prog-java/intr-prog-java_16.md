# 第十六章：数据库编程

本章介绍如何编写 Java 代码，可以操作数据库中的数据——插入、读取、更新、删除。它还提供了 SQL 语言和基本数据库操作的简要介绍。

在本章中，我们将涵盖以下主题：

+   什么是**Java 数据库连接**（**JDBC**）？

+   如何创建/删除数据库

+   **结构化查询语言**（**SQL**）简要概述

+   如何创建/删除/修改数据库表

+   **创建、读取、更新和删除**（**CRUD**）数据库数据

+   练习-选择唯一的名字

# 什么是 Java 数据库连接（JDBC）？

**Java 数据库连接**（**JDBC**）是 Java 功能，允许我们访问和修改数据库中的数据。它由 JDBC API（`java.sql`、`javax.sql`和`java.transaction.xa`包）和数据库特定的接口实现（称为数据库驱动程序）支持，每个数据库供应商都提供了与数据库访问的接口。

当人们说他们正在使用 JDBC 时，这意味着他们编写代码，使用 JDBC API 的接口和类以及知道如何将应用程序与特定数据库连接的数据库特定驱动程序来管理数据库中的数据。使用此连接，应用程序可以发出用**结构化查询语言**（**SQL**）编写的请求。当然，我们这里只谈论了理解 SQL 的数据库。它们被称为关系（或表格）数据库，并且占当前使用的数据库的绝大多数，尽管也使用一些替代方案——如导航数据库和 NoSql。

`java.sql`和`javax.sql`包包含在 Java 平台标准版（Java SE）中。从历史上看，`java.sql`包属于 Java 核心，而`javax.sql`包被认为是核心扩展。但后来，`javax.sql`包也被包含在核心中，名称没有更改，以避免破坏使用它的现有应用程序。`javax.sql`包包含支持语句池、分布式事务和行集的`DataSource`接口。我们将在本章的后续部分更详细地讨论这些功能。

与数据库一起工作包括八个步骤：

1.  按照供应商的说明安装数据库。

1.  创建数据库用户、数据库和数据库模式——表、视图、存储过程等。

1.  在应用程序上添加对`.jar`的依赖项，其中包含特定于数据库的驱动程序。

1.  从应用程序连接到数据库。

1.  构造 SQL 语句。

1.  执行 SQL 语句。

1.  使用执行结果。

1.  释放（关闭）在过程中打开的数据库连接和其他资源。

步骤 1-3 只在应用程序运行之前的数据库设置时执行一次。步骤 4-8 根据需要由应用程序重复执行。步骤 5-7 可以重复多次使用相同的数据库连接。

# 连接到数据库

以下是连接到数据库的代码片段：

```java
String URL = "jdbc:postgresql://localhost/javaintro";
Properties prop = new Properties( );
//prop.put( "user", "java" );
//prop.put( "password", "secretPass123" );
try {
  Connection conn = DriverManager.getConnection(URL, prop);
} catch(SQLException ex){
  ex.printStackTrace();
}
```

注释行显示了如何使用`java.util.Properties`类为连接设置用户和密码。上述只是一个示例，说明如何直接使用`DriverManger`类获取连接。传入属性的许多键对于所有主要数据库都是相同的，但其中一些是特定于数据库的。因此，请阅读您的数据库供应商文档以获取此类详细信息。

或者，仅传递用户和密码，我们可以使用重载版本`DriverManager.getConnection（String url，String user，String password）`。

保持密码加密是一个好的做法。我们不会告诉你如何做，但是互联网上有很多指南可用。

另一种连接到数据库的方法是使用`DataSource`接口。它的实现包含在与数据库驱动程序相同的`.jar`中。在 PostgreSQL 的情况下，有两个实现了`DataSource`接口的类：`org.postgresql.ds.PGSimpleDataSource`和`org.postgresql.ds.PGConnectionPoolDataSource`。我们可以使用它们来代替`DriverManager`。以下是使用`org.postgresql.ds.PGSimpleDataSource`类创建数据库连接的示例：

```java
PGSimpleDataSource source = new PGSimpleDataSource();
source.setServerName("localhost");
source.setDatabaseName("javaintro");
source.setLoginTimeout(10);
Connection conn = source.getConnection();
```

要使用`org.postgresql.ds.PGConnectionPoolDataSource`类连接到数据库，我们只需要用以下内容替换前面代码中的第一行：

```java
PGConnectionPoolDataSource source = new PGConnectionPoolDataSource();
```

使用`PGConnectionPoolDataSource`类允许我们在内存中创建一个`Connection`对象池。这是一种首选的方式，因为创建`Connection`对象需要时间。池化允许我们提前完成这个过程，然后根据需要重复使用已经创建的对象。池的大小和其他参数可以在`postgresql.conf`文件中设置。

但无论使用何种方法创建数据库连接，我们都将把它隐藏在`getConnection()`方法中，并在所有的代码示例中以相同的方式使用它。

有了`Connection`类的对象，我们现在可以访问数据库来添加、读取、删除或修改存储的数据。

# 关闭数据库连接

保持数据库连接活动需要大量的资源内存和 CPU-因此关闭连接并释放分配的资源是一个好主意，一旦你不再需要它们。在池化的情况下，`Connection`对象在关闭时会返回到池中，消耗更少的资源。

在 Java 7 之前，关闭连接的方法是通过在`finally`块中调用`close()`方法，无论是否有 catch 块：

```java
Connection conn = getConnection();
try {
  //use object conn here 
} finally {
  if(conn != null){
    conn.close();
  }
}
```

`finally`块中的代码总是会被执行，无论 try 块中的异常是否被抛出。但自 Java 7 以来，`try...with...resources`结构可以很好地处理实现了`java.lang.AutoCloseable`或`java.io.Closeable`接口的任何对象。由于`java.sql.Connection`对象实现了`AutoCloseable`，我们可以将上一个代码片段重写如下：

```java
try (Connection conn = getConnection()) {
  //use object conn here
}
catch(SQLException ex) {
  ex.printStackTrace();
}
```

捕获子句是必要的，因为可自动关闭的资源会抛出`java.sql.SQLException`。有人可能会说，这样做并没有节省多少输入。但是`Connection`类的`close()`方法也可能会抛出`SQLException`，所以带有`finally`块的代码应该更加谨慎地编写：

```java
Connection conn = getConnection();
try {
  //use object conn here 
} finally {
  if(conn != null){
    try {
      conn.close();
    } catch(SQLException ex){
      //do here what has to be done
    }
  }
}
```

前面的代码块看起来确实像是更多的样板代码。更重要的是，如果考虑到通常在`try`块内，一些其他代码也可能抛出`SQLException`，那么前面的代码应该如下所示：

```java
Connection conn = getConnection();
try {
  //use object conn here 
} catch(SQLException ex) {
  ex.printStackTrace();
} finally {
  if(conn != null){
    try {
      conn.close();
    } catch(SQLException ex){
      //do here what has to be done
    }
  }
}
```

样板代码增加了，不是吗？这还不是故事的结束。在接下来的章节中，您将了解到，要发送数据库请求，还需要创建一个`java.sql.Statement`，它会抛出`SQLException`，也必须关闭。然后前面的代码会变得更多：

```java
Connection conn = getConnection();
try {
  Statement statement = conn.createStatement();
  try{
    //use statement here
  } catch(SQLException ex){
    //some code here
  } finally {
    if(statement != null){
      try {
      } catch (SQLException ex){
        //some code here
      }
    } 
  }
} catch(SQLException ex) {
  ex.printStackTrace();
} finally {
  if(conn != null){
    try {
      conn.close();
    } catch(SQLException ex){
      //do here what has to be done
    }
  }
}
```

现在我们可以充分欣赏`try...with...resources`结构的优势，特别是考虑到它允许我们在同一个子句中包含多个可自动关闭的资源：

```java
try (Connection conn = getConnection();
  Statement statement = conn.createStatement()) {
  //use statement here
} catch(SQLException ex) {
  ex.printStackTrace();
}
```

自 Java 9 以来，我们甚至可以使其更简单：

```java
Connection conn = getConnection();
try (conn; Statement statement = conn.createStatement()) {
  //use statement here
} catch(SQLException ex) {
  ex.printStackTrace();
}
```

现在很明显，`try...with...resources`结构是一个无可争议的赢家。

# 结构化查询语言（SQL）

SQL 是一种丰富的语言，我们没有足够的空间来涵盖其所有特性。我们只想列举一些最受欢迎的特性，以便您了解它们的存在，并在需要时查找它们。

与 Java 语句类似，SQL 语句表达了像英语句子一样的数据库请求。每个语句都可以在数据库控制台中执行，也可以通过使用 JDBC 连接在 Java 代码中执行。程序员通常在控制台中测试 SQL 语句，然后再在 Java 代码中使用它，因为在控制台中的反馈速度要快得多。在使用控制台时，无需编译和执行程序。

有 SQL 语句可以创建和删除用户和数据库。我们将在下一节中看到此类语句的示例。还有其他与整个数据库相关的语句，超出了本书的范围。

创建数据库后，以下三个 SQL 语句允许我们构建和更改数据库结构 - 表、函数、约束或其他数据库实体：

+   `CREATE`：此语句创建数据库实体

+   `ALTER`：此语句更改数据库实体

+   `DROP`：此语句删除数据库实体

还有各种 SQL 语句，允许我们查询每个数据库实体的信息，这也超出了本书的范围。

并且有四种 SQL 语句可以操作数据库中的数据：

+   `INSERT`：此语句向数据库添加数据

+   `SELECT`：此语句从数据库中读取数据

+   `UPDATE`：此语句更改数据库中的数据

+   `DELETE`：此语句从数据库中删除数据

可以向前述语句添加一个或多个不同的子句，用于标识请求的数据（`WHERE`-子句）、结果返回的顺序（`ORDER`-子句）等。

JDBC 连接允许将前述 SQL 语句中的一个或多个组合包装在提供数据库端不同功能的三个类中：

+   `java.sql.Statement`：只是将语句发送到数据库服务器以执行

+   `java.sql.PreparedStatement`：在数据库服务器上的某个执行路径中缓存语句，允许以高效的方式多次执行具有不同参数的语句

+   `java.sql.CallableStatement`：在数据库中执行存储过程

我们将从创建和删除数据库及其用户的语句开始我们的演示。

# 创建数据库及其结构

查找如何下载和安装您喜欢的数据库服务器。数据库服务器是一个维护和管理数据库的软件系统。对于我们的演示，我们将使用 PostgreSQL，一个免费的开源数据库服务器。

安装数据库服务器后，我们将使用其控制台来创建数据库及其用户，并赋予相应的权限。有许多方法可以构建数据存储和具有不同访问级别的用户系统。在本书中，我们只介绍基本方法，这使我们能够演示主要的 JDBC 功能。

# 创建和删除数据库及其用户

阅读数据库说明，并首先创建一个`java`用户和一个`javaintro`数据库（或选择任何其他您喜欢的名称，并在提供的代码示例中使用它们）。以下是我们在 PostgreSQL 中的操作方式：

```java
CREATE USER java SUPERUSER;
CREATE DATABASE javaintro OWNER java;
```

如果您犯了一个错误并决定重新开始，您可以使用以下语句删除创建的用户和数据库：

```java
DROP USER java;
DROP DATABASE javaintro;
```

我们为我们的用户选择了`SUPERUSER`角色，但是良好的安全实践建议只将这样一个强大的角色分配给管理员。对于应用程序，建议创建一个用户，该用户不能创建或更改数据库本身——其表和约束——但只能管理数据。此外，创建另一个逻辑层，称为**模式**，该模式可以具有自己的一组用户和权限，也是一个良好的实践。这样，同一数据库中的几个模式可以被隔离，每个用户（其中一个是您的应用程序）只能访问特定的模式。在企业级别上，通常的做法是为数据库模式创建同义词，以便没有应用程序可以直接访问原始结构。

但是，正如我们已经提到的，对于本书的目的，这是不需要的，所以我们把它留给数据库管理员，他们为每个企业的特定工作条件建立规则和指导方针。

现在我们可以将我们的应用程序连接到数据库。

# 创建、修改和删除表

表的标准 SQL 语句如下：

```java
CREATE TABLE tablename (
  column1 type1,
  column2 type2,
  column3 type3,
  ....
);
```

表名、列名和可以使用的值类型的限制取决于特定的数据库。以下是在 PostgreSQL 中创建表 person 的命令示例：

```java
CREATE TABLE person (
  id SERIAL PRIMARY KEY,
  first_name VARCHAR NOT NULL,
  last_name VARCHAR NOT NULL,
  dob DATE NOT NULL
);
```

正如您所看到的，我们已经将`dob`（出生日期）列设置为不可为空。这对我们的`Person` Java 类施加了约束，该类将表示此表的记录：其`dob`字段不能为`null`。这正是我们在第六章中所做的，当时我们创建了我们的`Person`类，如下所示：

```java
class Person {
  private String firstName, lastName;
  private LocalDate dob;
  public Person(String firstName, String lastName, LocalDate dob) {
    this.firstName = firstName == null ? "" : firstName;
    this.lastName = lastName == null ? "" : lastName;
    if(dob == null){
      throw new RuntimeException("Date of birth is null");
    }
    this.dob = dob;
  }
  public String getFirstName() { return firstName; }
  public String getLastName() { return lastName; }
  public LocalDate getDob() { return dob; }
}
```

我们没有设置`VARCHAR`类型的列的大小，因此允许这些列存储任意长度的值，而整数类型允许它们存储从公元前 4713 年到公元 5874897 年的数字。添加了`NOT NULL`，因为默认情况下列将是可空的，而我们希望确保每条记录的所有列都被填充。我们的`Person`类通过将名字和姓氏设置为空的`String`值来支持它，如果它们是`null`，作为`Person`构造函数的参数。

我们还将`id`列标识为`PRIMARY KEY`，这表示该列唯一标识记录。`SERIAL`关键字表示我们要求数据库在添加新记录时生成下一个整数值，因此每条记录将有一个唯一的整数编号。或者，我们可以从`first_name`、`last_name`和`dob`的组合中创建`PRIMARY KEY`：

```java
CREATE TABLE person (
  first_name VARCHAR NOT NULL,
  last_name VARCHAR NOT NULL,
  dob DATE NOT NULL,
  PRIMARY KEY (first_name, last_name, dob)
);
```

但有可能有两个人有相同的名字，并且出生在同一天，所以我们决定不这样做，并添加了`Person`类的另一个字段和构造函数：

```java
public class Person {
  private String firstName, lastName;
  private LocalDate dob;
  private int id;
  public Person(int id, String firstName, 
                                  String lastName, LocalDate dob) {
    this(firstName, lastName, dob);
    this.id = id;
  }   
  public Person(String firstName, String lastName, LocalDate dob) {
    this.firstName = firstName == null ? "" : firstName;
    this.lastName = lastName == null ? "" : lastName;
    if(dob == null){
      throw new RuntimeException("Date of birth is null");
    }
    this.dob = dob;
  }
  public String getFirstName() { return firstName; }
  public String getLastName() { return lastName; }
  public LocalDate getDob() { return dob; }
}
```

我们将使用接受`id`的构造函数来基于数据库中的记录构建对象，而另一个构造函数将用于在插入新记录之前创建对象。

我们在数据库控制台中运行上述 SQL 语句并创建这个表：

![](img/00af477d-66c8-4e49-941b-8a8ddbaf76f0.png)

如果必要，可以通过`DROP`命令删除表：

```java
DROP table person;
```

可以使用`ALTER`命令更改现有表。例如，我们可以添加一个`address`列：

```java
ALTER table person add column address VARCHAR;
```

如果您不确定这样的列是否已经存在，可以添加 IF EXISTS 或 IF NOT EXISTS：

```java
ALTER table person add column IF NOT EXISTS address VARCHAR;
```

但这种可能性只存在于 PostgreSQL 9.6 之后。

数据库表创建的另一个重要考虑因素是是否必须添加索引。索引是一种数据结构，可以加速表中的数据搜索，而无需检查每条表记录。索引可以包括一个或多个表的列。例如，主键的索引会自动生成。如果您已经创建了表的描述，您将看到：

![](img/b447b61b-1d4f-4578-a9be-778d1c6c8ec9.png)

如果我们认为（并通过实验已经证明）它将有助于应用程序的性能，我们也可以自己添加任何索引。例如，我们可以通过添加以下索引来允许不区分大小写的搜索名字和姓氏：

```java
CREATE INDEX idx_names ON person ((lower(first_name), lower(last_name));
```

如果搜索速度提高，我们会保留索引。如果没有，可以删除它：

```java
drop index idx_names;
```

我们删除它，因为索引会增加额外的写入和存储空间开销。

我们也可以从表中删除列：

```java
ALTER table person DROP column address;
```

在我们的示例中，我们遵循了 PostgreSQL 的命名约定。如果您使用不同的数据库，建议您查找其命名约定并遵循，以便您创建的名称与自动创建的名称对齐。

# 创建，读取，更新和删除（CRUD）数据

到目前为止，我们已经使用控制台将 SQL 语句发送到数据库。可以使用 JDBC API 从 Java 代码执行相同的语句，但是表只创建一次，因此没有必要为一次性执行编写程序。

但是管理数据是另一回事。这是我们现在要编写的程序的主要目的。为了做到这一点，首先我们将以下依赖项添加到`pom.xml`文件中，因为我们已经安装了 PostgreSQL 9.6：

```java
<dependency>
  <groupId>org.postgresql</groupId>
  <artifactId>postgresql</artifactId>
  <version>42.2.2</version>
</dependency>
```

# INSERT 语句

在数据库中创建（填充）数据的 SQL 语句具有以下格式：

```java
INSERT INTO table_name (column1,column2,column3,...)
   VALUES (value1,value2,value3,...);
```

当必须添加多个表记录时，它看起来像这样：

```java
INSERT INTO table_name (column1,column2,column3,...)
 VALUES (value1,value2,value3,...), (value11,value21,value31,...), ...;
```

在编写程序之前，让我们测试我们的`INSERT`语句：

！[]（img/c87f8461-b463-4dcb-a806-01b2bac288c7.png）

它没有错误，返回的插入行数为 1，所以我们将创建以下方法：

```java
void executeStatement(String sql){
  Connection conn = getConnection();
  try (conn; Statement st = conn.createStatement()) {
    st.execute(sql);
  } catch (SQLException ex) {
    ex.printStackTrace();
  }
}
```

我们可以执行前面的方法并插入另一行：

```java
executeStatement("insert into person (first_name, last_name, dob)" +
                             " values ('Bill', 'Grey', '1980-01-27')");
```

我们将在下一节中看到此前`INSERT`语句执行的结果以及`SELECT`语句的演示。

与此同时，我们想讨论`java.sql.Statement`接口的最受欢迎的方法：

+   `boolean execute（String sql）`：如果执行的语句返回数据（作为`java.sql.ResultSet`对象），则返回`true`，可以使用`java.sql.Statement`接口的`ResultSet getResultSet（）`方法检索数据。如果执行的语句不返回数据（SQL 语句可能正在更新或插入某些行），则返回`false`，并且随后调用`java.sql.Statement`接口的`int getUpdateCount（）`方法返回受影响的行数。例如，如果我们在`executeStatement（）`方法中添加了打印语句，那么在插入一行后，我们将看到以下结果：

```java
        void executeStatement(String sql){
          Connection conn = getConnection();
          try (conn; Statement st = conn.createStatement()) {
            System.out.println(st.execute(sql));      //prints: false
            System.out.println(st.getResultSet());    //prints: null
            System.out.println(st.getUpdateCount());  //prints: 1
          } catch (SQLException ex) {
            ex.printStackTrace();
          }
        }
```

+   `ResultSet executeQuery（String sql）`：它将数据作为`java.sql.ResultSet`对象返回（预计执行的 SQL 语句是`SELECT`语句）。可以通过随后调用`java.sql.Statement`接口的`ResultSet getResultSet（）`方法检索相同的数据。`java.sql.Statement`接口的`int getUpdateCount（）`方法返回`-1`。例如，如果我们更改我们的`executeStatement（）`方法并使用`executeQuery（）`，则`executeStatement（"select first_name from person"）`的结果将是：

```java
        void executeStatement(String sql){
          Connection conn = getConnection();
          try (conn; Statement st = conn.createStatement()) {
             System.out.println(st.executeQuery(sql)); //prints: ResultSet
             System.out.println(st.getResultSet());    //prints: ResultSet
             System.out.println(st.getUpdateCount());  //prints: -1
          } catch (SQLException ex) {
             ex.printStackTrace();
          }
        }
```

+   `int executeUpdate(String sql)`: 它返回受影响的行数（执行的 SQL 语句预期为`UPDATE`语句）。`java.sql.Statement`接口的`int getUpdateCount()`方法的后续调用返回相同的数字。`java.sql.Statement`接口的`ResultSet getResultSet()`方法的后续调用返回`null`。例如，如果我们更改我们的`executeStatement()`方法并使用`executeUpdate()`，`executeStatement("update person set first_name = 'Jim' where last_name = 'Adams'")`的结果将是：

```java
        void executeStatement4(String sql){
          Connection conn = getConnection();
          try (conn; Statement st = conn.createStatement()) {
            System.out.println(st.executeUpdate(sql));//prints: 1
            System.out.println(st.getResultSet());    //prints: null
            System.out.println(st.getUpdateCount());  //prints: 1
          } catch (SQLException ex) {
            ex.printStackTrace();
          }
        }
```

# SELECT 语句

`SELECT`语句的格式如下：

```java
SELECT column_name, column_name
FROM table_name WHERE some_column = some_value;
```

当需要选择所有列时，格式如下：

```java
SELECT * FROM table_name WHERE some_column=some_value;
```

这是`WHERE`子句的更一般的定义：

```java
WHERE column_name operator value
Operator:
   =   Equal
   <>  Not equal. In some versions of SQL, !=
   >   Greater than
   <   Less than
   >=  Greater than or equal
   <=  Less than or equal
   IN  Specifies multiple possible values for a column
   LIKE  Specifies the search pattern
   BETWEEN  Specifies the inclusive range of vlaues in a column
```

`column_name` operator value 构造可以使用`AND`和`OR`逻辑运算符组合，并用括号`( )`分组。

在前面的语句中，我们执行了一个`select first_name from person`的`SELECT`语句，返回了`person`表中记录的所有名字。现在让我们再次执行它并打印出结果：

```java
Connection conn = getConnection();
try (conn; Statement st = conn.createStatement()) {
  ResultSet rs = st.executeQuery("select first_name from person");
  while (rs.next()){
    System.out.print(rs.getString(1) + " "); //prints: Jim Bill
  }
} catch (SQLException ex) {
  ex.printStackTrace();
}
```

`ResultSet`接口的`getString(int position)`方法从位置`1`（`SELECT`语句中列的第一个）提取`String`值。对于所有原始类型，如`getInt()`和`getByte()`，都有类似的获取器。

还可以通过列名从`ResultSet`对象中提取值。在我们的情况下，它将是`getString("first_name")`。当`SELECT`语句如下时，这是特别有用的：

```java
select * from person;
```

但请记住，通过列名从`ResultSet`对象中提取值效率较低。性能差异非常小，只有在操作发生多次时才变得重要。只有实际的测量和测试才能告诉您这种差异对您的应用程序是否重要。通过列名提取值尤其有吸引力，因为它提供更好的代码可读性，在应用程序维护期间可以得到很好的回报。

`ResultSet`接口中还有许多其他有用的方法。如果您的应用程序从数据库中读取数据，我们强烈建议您阅读`SELECT`语句和`ResultSet`接口的文档。

# UPDATE 语句

数据可以通过`UPDATE`语句更改：

```java
UPDATE table_name SET column1=value1,column2=value2,... WHERE-clause;
```

我们已经使用这样的语句来改变记录中的名字，将原始值`John`改为新值`Jim`：

```java
update person set first_name = 'Jim' where last_name = 'Adams'
```

稍后，使用`SELECT`语句，我们将证明更改是成功的。没有`WHERE`子句，表的所有记录都将受到影响。

# DELETE 语句

数据可以通过`DELETE`语句删除：

```java
DELETE FROM table_name WHERE-clause;
```

没有`WHERE`子句，表的所有记录都将被删除。在`person`表的情况下，我们可以使用`delete from person` SQL 语句删除所有记录。以下语句从`person`表中删除所有名为 Jim 的记录：

```java
delete from person where first_name = 'Jim';
```

# 使用 PreparedStatement 类

`PreparedStatement`对象——`Statement`接口的子接口——旨在被缓存在数据库中，然后用于有效地多次执行 SQL 语句，以适应不同的输入值。与`Statement`对象类似（由`createStatement()`方法创建），它可以由同一`Connection`对象的`prepareStatement()`方法创建。

生成`Statement`对象的相同 SQL 语句也可以用于生成`PreparedStatement`对象。事实上，考虑使用`PreparedStatement`来调用多次的任何 SQL 语句是一个好主意，因为它的性能优于`Statement`。要做到这一点，我们只需要更改前面示例代码中的这两行：

```java
try (conn; Statement st = conn.createStatement()) {
  ResultSet rs = st.executeQuery(sql);
```

或者，我们可以以同样的方式使用`PreparedStatement`类：

```java
try (conn; PreparedStatement st = conn.prepareStatement(sql)) {
  ResultSet rs = st.executeQuery();
```

但是`PreparedStatement`的真正用处在于它能够接受参数-替换（按照它们出现的顺序）`?`符号的输入值。例如，我们可以创建以下方法：

```java
List<Person> selectPersonsByFirstName(String sql, String searchValue){
  List<Person> list = new ArrayList<>();
  Connection conn = getConnection();
  try (conn; PreparedStatement st = conn.prepareStatement(sql)) {
    st.setString(1, searchValue);
    ResultSet rs = st.executeQuery();
    while (rs.next()){
      list.add(new Person(rs.getInt("id"),
               rs.getString("first_name"),
               rs.getString("last_name"),
               rs.getDate("dob").toLocalDate()));
    }
  } catch (SQLException ex) {
    ex.printStackTrace();
  }
  return list;
}
```

我们可以使用前面的方法从`person`表中读取与`WHERE`子句匹配的记录。例如，我们可以找到所有名为`Jim`的记录：

```java
String sql = "select * from person where first_name = ?";
List<Person> list = selectPersonsByFirstName(sql, "Jim");
for(Person person: list){
  System.out.println(person);
}
```

结果将是：

```java
Person{firstName='Jim', lastName='Adams', dob=1999-08-23, id=1}
```

`Person`对象以这种方式打印，因为我们添加了以下`toString()`方法：

```java
@Override
public String toString() {
  return "Person{" +
          "firstName='" + firstName + '\'' +
          ", lastName='" + lastName + '\'' +
          ", dob=" + dob +
          ", id=" + id +
          '}';
}
```

我们可以通过运行以下代码获得相同的结果：

```java
String sql = "select * from person where last_name = ?";
List<Person> list = selectPersonsByFirstName(sql, "Adams");
for(Person person: list){
    System.out.println(person);
}
```

总是使用准备好的语句进行 CRUD 操作并不是一个坏主意。如果只执行一次，它们可能会慢一点，但您可以测试看看这是否是您愿意支付的代价。使用准备好的语句可以获得一致的（更易读的）代码、更多的安全性（准备好的语句不容易受到 SQL 注入攻击的影响）以及少做一个决定-只需在任何地方重用相同的代码。

# 练习-选择唯一的名字

编写一个 SQL 语句，从人员表中选择所有的名字，而不重复。例如，假设人员表中有三条记录，这些记录有这些名字：`Jim`，`Jim`和`Bill`。您编写的 SQL 语句必须返回`Jim`和`Bill`，而不重复两次的`Jim`。

我们没有解释如何做; 您必须阅读 SQL 文档，以找出如何选择唯一的值。

# 答案

使用`distinct`关键字。以下 SQL 语句返回唯一的名字：

```java
select distinct first_name from person;
```

# 摘要

本章介绍了如何编写能够操作数据库中的数据的 Java 代码。它还对 SQL 语言和基本数据库操作进行了简要介绍。读者已经学会了 JDBC 是什么，如何创建和删除数据库和表，以及如何编写一个管理表中数据的程序。

在下一章中，读者将学习函数式编程的概念。我们将概述 JDK 附带的功能接口，解释如何在 lambda 表达式中使用它们，并了解如何在数据流处理中使用 lambda 表达式。
