# 第五章. 与数据工作

MVC 方法讨论模型、视图和控制器。我们在前面的章节中详细介绍了视图和控制器，而相当程度上忽略了模型。模型是 MVC 的重要组成部分；对模型所做的更改将反映在使用它们的视图和控制器中。

没有数据事务的 Web 应用是不完整的。本章是关于设计模型和在 Play 中处理数据库事务。

在本章中，我们将涵盖以下主题：

+   模型

+   JDBC

+   Anorm

+   Slick

+   ReactiveMongo

+   一个缓存 API

# 介绍模型

**模型**是一个领域对象，它映射到数据库实体。例如，一个社交网络应用有用户。用户可以注册、更新个人资料、添加朋友、发布链接等。在这里，用户是一个领域对象，每个用户在数据库中都将有相应的条目。因此，我们可以以下这种方式定义一个用户模型：

```java
case class User(id: Long,
                loginId: String,
                name: Option[String],
                dob: Option[Long])
object User { def register (loginId: String,...) = {…}
...
}
```

之前，我们定义了一个没有使用数据库的模型：

```java
case class Task(id: Int, name: String)

object Task {

  private var taskList: List[Task] = List()

  def all: List[Task] = {
    taskList
  }

  def add(taskName: String) = {
    val lastId: Int = if (!taskList.isEmpty) taskList.last.id else 0
    taskList = taskList ++ List(Task(lastId + 1, taskName))
  }

  def delete(taskId: Int) = {
    taskList = taskList.filterNot(task => task.id == taskId)
  }
}  
```

任务列表示例有一个`Task`模型，但它没有绑定到数据库，以保持事情简单。在本章结束时，我们将能够使用数据库来支持它。

# JDBC

在使用关系型数据库的应用中，使用**Java 数据库连接**（**JDBC**）访问数据库是很常见的。Play 提供了一个插件来管理 JDBC 连接池。该插件内部使用 BoneCP ([`jolbox.com/`](http://jolbox.com/))，一个快速的**Java 数据库连接池**（**JDBC pool**）库。

### 注意

要使用该插件，应在构建文件中添加一个依赖项：

```java
val appDependencies = Seq(jdbc)
```

插件支持 H2、SQLite、PostgreSQL、MySQL 和 SQL。Play 附带了一个 H2 数据库驱动程序，但为了使用其他数据库，我们应该添加相应的驱动程序的依赖项：

```java
val appDependencies = Seq( jdbc,
"mysql" % "mysql-connector-java" % "5.1.18",...)
```

插件公开以下方法：

+   `getConnection`：它接受数据库的名称，以及使用此连接执行任何语句时是否应该自动提交。如果没有提供名称，它将获取默认名称的数据库连接。

+   `withConnection`：它接受一个应该使用 JDBC 连接执行的代码块。一旦代码块执行完毕，连接将被释放。或者，它接受数据库的名称。

+   `withTransaction`：它接受一个应该使用 JDBC 事务执行的代码块。一旦代码块执行完毕，连接及其创建的所有语句都将被释放。

插件如何知道数据库的详细信息？数据库的详细信息可以在`conf/application.` `conf`中设置：

```java
db.default.driver=com.mysql.jdbc.Driver
db.default.url="jdbc:mysql://localhost:3306/app"
db.default.user="changeme"
db.default.password="changeme"
```

第一部分，`db`，是一组属性，这些属性由 DBPlugin 使用。第二部分是数据库的名称，例如示例中的`default`，最后一部分是属性的名称。

对于 MySQL 和 PostgreSQL，我们可以在 URL 中包含用户名和密码：

```java
db.default.url="mysql://user:password@localhost:3306/app"
db.default.url="postgres://user:password@localhost:5432/app"
```

对于额外的 JDBC 配置，请参阅 [`www.playframework.com/documentation/2.3.x/SettingsJDBC`](https://www.playframework.com/documentation/2.3.x/SettingsJDBC)。

现在我们已经启用并配置了 JDBC 插件，我们可以连接到类似 SQL 的数据库并执行查询：

```java
def fetchDBUser = Action { 
    var result = "DB User:" 
    val conn = DB.getConnection() 
    try{ 
 val rs = conn.createStatement().executeQuery("SELECT USER()") 
 while (rs.next()) { 
 result += rs.getString(1) 
 } 
 } finally { 
 conn.close() 
 } 
    Ok(result) 
  }
```

或者，我们可以使用 `DB.withConnection` 辅助函数，它管理 DB 连接：

```java
def fetchDBUser = Action { 
    var result = "DB User:" 
    DB.withConnection { conn => 
 val rs = conn.createStatement().executeQuery("SELECT USER()") 
 while (rs.next()) { 
 result += rs.getString(1) 
 } 
 } 
    Ok(result) 
  }
```

# Anorm

**Anorm** 是 Play 中一个支持使用纯 SQL 与数据库交互的模块。

Anorm 提供了查询 SQL 数据库并将结果解析为 Scala 对象的方法，包括内置和自定义的。

Anorm 的目标，如 Play 网站上所述（[`www.playframework.com/documentation/2.3.x/ScalaAnorm`](https://www.playframework.com/documentation/2.3.x/ScalaAnorm)）是：

> **使用 JDBC 是一件痛苦的事情，但我们提供了一个更好的 API**
> 
> *我们同意直接使用 JDBC API 是繁琐的，尤其是在 Java 中。你必须处理检查异常，并且反复迭代 ResultSet 以将原始数据集转换为你的数据结构。*
> 
> *我们提供了一个更简单的 JDBC API；使用 Scala，你不需要担心异常，并且使用函数式语言转换数据非常容易。实际上，Play Scala SQL 访问层的目的是提供几个 API，以有效地将 JDBC 数据转换为其他 Scala 结构。*
> 
> **你不需要另一个 DSL 来访问关系数据库**
> 
> *SQL 已经是访问关系数据库的最佳 DSL，我们不需要发明新的东西。此外，SQL 语法和功能可能因数据库供应商而异。*
> 
> *如果你尝试使用另一个专有 SQL DSL 抽象这个点，你将不得不处理针对每个供应商的几个方言（如 Hibernate 的），并且限制自己不使用特定数据库的有趣功能。*
> 
> *Play 有时会为你提供预填充的 SQL 语句，但我们的想法不是隐藏我们底层使用 SQL 的事实。Play 只是为了在简单查询中节省输入大量字符，你总是可以回退到普通的 SQL。*
> 
> **生成 SQL 的类型安全 DSL 是一个错误**
> 
> *有些人认为类型安全的 DSL 更好，因为所有查询都由编译器检查。不幸的是，编译器根据你通常通过将数据结构映射到数据库模式而编写的元模型定义来检查你的查询。*
> 
> *没有保证这个元模型是正确的。即使编译器说你的代码和查询类型正确，它仍然可能在运行时因为实际数据库定义的不匹配而悲惨地失败。*
> 
> **掌握你的 SQL 代码**
> 
> *对象关系映射在简单情况下工作得很好，但当你必须处理复杂的模式或现有数据库时，你将花费大部分时间与你的 ORM 作战，以使其生成你想要的 SQL 查询。*
> 
> *自己编写 SQL 查询对于简单的 'Hello World' 应用程序来说可能是繁琐的，但对于任何实际应用，您最终将通过完全控制您的 SQL 代码来节省时间和简化代码。*

### 注意

当使用 Anorm 开发应用程序时，应明确指定其依赖项，因为它在 Play 中是一个独立的模块（从 Play 2.1 开始）：

```java
val appDependencies = Seq(
      jdbc,
      anorm
)
```

让我们在 MySQL 中想象我们的用户模型。表可以定义如下：

```java
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `login_id` varchar(45) NOT NULL,
  `password` varchar(50) NOT NULL,
  `name` varchar(45) DEFAULT NULL,
  `dob` bigint(20) DEFAULT NULL,
  `is_active` tinyint(1) NOT NULL DEFAULT '1',
  PRIMARY KEY (`id`),
  UNIQUE KEY `login_id_UNIQUE` (`login_id`),
  UNIQUE KEY `id_UNIQUE` (`id`)
) ENGINE=InnoDB
```

现在我们来看看我们将在这个表中进行的不同查询。查询如下所示：

+   `Insert`：此查询包括添加新用户

+   `Update`：此查询包括更新配置文件、密码等

+   `Select`：此查询包括根据特定标准检索一个或多个用户的详细信息

假设当用户请求从我们的应用程序中删除其账户时，我们不会从数据库中删除用户，而是将用户的状态标记为不活跃。因此，我们不会使用任何删除查询。

使用 Anorm，我们可以自动生成 `userId` 如下：

```java
DB.withConnection {
  implicit connection =>
  val userId  = SQL"""INSERT INTO user(login_id,password,name,
 dob) VALUES($loginId,$password,$name,$dob)""".executeInsert()
userId
}
```

在这里，`loginId`、`password`、`name` 和 `dob` 是在运行时替换查询的变量。Anorm 只构建 `java.sql.PreparedStatements`，这可以防止 SQL 注入。

SQL 方法返回 `SimpleSql` 类型的对象，并定义如下：

```java
implicit class SqlStringInterpolation(val sc: StringContext) extends AnyVal {
  def SQL(args: ParameterValue*) = prepare(args)

  private def prepare(params: Seq[ParameterValue]) = {
    // Generates the string query with "%s" for each parameter placeholder
    val sql = sc.parts.mkString("%s")

    val (ns, ps): (List[String], Map[String, ParameterValue]) =
        namedParams(params)

      SimpleSql(SqlQuery(sql, ns), ps,
        defaultParser = RowParser(row => Success(row)))
    }
  }
```

`SimpleSql` 用于表示查询的中间格式。其构造函数如下：

```java
case class SimpleSqlT extends Sql { … }
```

`executeInsert` 方法使用 `SimpleSql` 对象的 `getFilledStatement` 方法获取 `PreparedStatement`。然后执行 `PreparedStatement` 的 `getGeneratedKeys()` 方法。

`getGeneratedKeys` 方法产生一个自动生成的键，它是调用它的语句执行的结果。如果没有创建键，它返回一个空对象。

现在我们使用 Anorm 更新用户的密码：

```java
DB.withConnection {
    implicit connection =>
SQL"""UPDATE user SET password=$password WHERE id = $userId""".executeUpdate()
}
```

`executeUpdate` 方法的工作方式与 `executeInsert` 类似。区别在于它调用 `PreparedStatement` 的 `executeUpdate` 方法，而不是 `getGeneratedKeys`。

`executeUpdate` 方法返回受影响的行数，对于 **数据操纵语言**（**DML**）语句。如果 SQL 语句是其他类型，例如 **数据定义语言**（**DDL**），则返回 `0`。

现在我们尝试获取所有注册用户的详细信息。如果我们希望结果行被解析为用户对象，我们应该定义一个解析器。用户的解析器如下所示：

```java
def userRow:RowParser[User] = {
    getLong ~
      getString ~
      get[Option[String]]("name") map {
      case id ~ login_id ~ name  =>  User(id, login_id, name)
    }
  }
```

在大多数查询中，我们不需要密码和出生日期，因此我们可以从用户 `RowParser` 默认中排除它们。

使用此解析器的查询可以表示如下：

```java
DB.withConnection {
  implicit connection =>
  val query = "SELECT id,login_id,name FROM user"
  SQL(query).as(userRow.*)
}
```

`.*` 符号表示结果应有一行或多行，类似于正则表达式中的常见解释。同样，当预期结果由零行或多行组成时，可以使用 `.+` 符号。

### 提示

如果您使用的是不支持字符串插值的较旧版本的 Scala，查询将按如下方式编写：

```java
DB.withConnection {
  implicit connection =>
val insertQuery  = """INSERT INTO user(login_id,password,name,
  |dob) VALUES({loginId},{password},{name},{dob})""".stripMargin
val userId = SQL(insertQuery).on(
  'loginId -> loginId,
  'password -> password,
  'name -> name,
  'dob -> dob).executeInsert()
userId
}
```

`on`方法通过传递给它的参数映射更新查询。它是在以下方式中为`SimpleSql`定义的：

```java
 def on(args: NamedParameter*): SimpleSql[T] =
    copy(params = this.params ++ args.map(_.tupled))
```

请参阅 Play 文档（[`www.playframework.com/documentation/2.3.x/ScalaAnorm`](http://www.playframework.com/documentation/2.3.x/ScalaAnorm)）和 Anorm API 文档（[`www.playframework.com/documentation/2.3.x/api/scala/index.html#anorm.package`](http://www.playframework.com/documentation/2.3.x/api/scala/index.html#anorm.package)）以获取更多用例和详细信息。

# Slick

根据 Slick 网站（[`slick.typesafe.com/doc/2.1.0/introduction.html#what-is-slick`](http://slick.typesafe.com/doc/2.1.0/introduction.html#what-is-slick)）：

> *Slick 是 Typesafe 为 Scala 提供的现代数据库查询和访问库。它允许您以使用 Scala 集合的方式处理存储数据，同时同时让您完全控制数据库访问发生的时间和传输的数据。您还可以直接使用 SQL。*
> 
> *当使用 Scala 而不是原始 SQL 进行查询时，您将受益于编译时安全性和组合性。Slick 可以使用其可扩展的查询编译器为不同的后端数据库生成查询，包括您自己的数据库。*

我们可以通过 play-slick 插件在我们的 Play 应用程序中使用 Slick。该插件为 Slick 在 Play 应用程序中的使用提供了额外的功能。根据[`github.com/playframework/`](https://github.com/playframework/)，play-slick 包括以下三个特性：

+   一个包装器 DB 对象，它使用 Play 配置文件中定义的数据源，并从连接池中提取它们。它存在是为了能够以与 Anorm JDBC 连接相同的方式使用 Slick 会话。有一些智能缓存和负载均衡，可以使您的数据库连接性能更佳。

+   一个 DDL 插件，它读取 Slick 表并在重新加载时自动创建模式更新。这在演示和入门时特别有用。

+   一个包装器，用于与 Slick 一起使用 play Enumeratees

要使用它，我们需要在构建文件中添加以下库依赖项：

```java
"com.typesafe.play" %% "play-slick" % "0.8.1"
Let's see how we can define user operations using Slick.
```

首先，我们需要在 Scala 中定义模式。这可以通过将所需的表映射到 case 类来完成。对于我们的用户表，模式可以定义为：

```java
case class SlickUser(id: Long, loginId: String, name: String) 

class SlickUserTable(tag: Tag) extends TableSlickUser { 
  def id = columnLong 

  def loginId = columnString 

  def name = columnString 

  def dob = columnLong 

  def password = columnString 

  def * = (id, loginId, name) <>(SlickUser.tupled, SlickUser.unapply) 
}
```

`Table`是 Slick 的一个特质，其列通过`column`方法指定。以下类型适用于列：

+   **数值类型**：包括`Byte`、`Short`、`Int`、`Long`、`BigDecimal`、`Float`和`Double`

+   **日期类型**：包括`java.sql.Date`、`java.sql.Time`和`java.sql.Timestamp`

+   **UUID 类型**：包括`java.util.UUID`

+   **LOB 类型**：包括`java.sql.Blob`、`java.sql.Clob`和`Array[Byte]`

+   **其他类型**：包括`Boolean`、`String`和`Unit`

`column`方法接受列约束，如`PrimaryKey`、`Default`、`AutoInc`、`NotNull`和`Nullable`。

对于每个表，`*`方法是强制性的，类似于`RowParser`。

现在我们可以使用这个定义一个`TableQuery` Slick，并使用它来查询数据库。有简单的方法可以执行等效的 DB 操作。我们可以使用 play-slick 包装器和 Slick API 在 Anorm 对象中定义这些方法：

```java
object SlickUserHelper { 
  val users = TableQuery[SlickUserTable] 

  def add(loginId: String, 
          password: String, 
          name: String = "anonymous", 
          dateOfBirth: DateTime): Long = { 

    play.api.db.slick.DB.withSession { implicit session => 
      users.map(p => (p.loginId, p.name, p.dob, p.password)) 
        .returning(users.map(_.id)) 
        .insert((loginId, name, dateOfBirth.getMillis, password)) 
    } 
  } 

  def updatePassword(userId: Long, 
                   password: String) = { 

    play.api.db.slick.DB.withSession { implicit session => 
      users.filter(_.id === userId) 
        .map(u => u.password) 
        .update(password) 
    } 
  } 

  def getAll: Seq[SlickUser] = { 
    play.api.db.slick.DB.withSession { implicit session => 
      users.run 
    } 
  } 
}
```

`run`方法等同于调用`SELECT *`。

更多详情请参阅 Slick([`slick.typesafe.com/doc/2.1.0/`](http://slick.typesafe.com/doc/2.1.0/))和 play-slick 文档([`github.com/playframework/play-slick`](https://github.com/playframework/play-slick))。

# ReactiveMongo

由于非结构化数据、写入可扩展性等原因，现在许多应用程序使用 NoSQL 数据库。MongoDB 就是其中之一。根据其网站([`docs.mongodb.org/manual/core/introduction/`](http://docs.mongodb.org/manual/core/introduction/))：

> *MongoDB 是一个提供高性能、高可用性和自动扩展的开源文档数据库。*
> 
> MongoDB 的关键特性包括：
> 
> 高性能
> 
> 高可用性（自动故障转移、数据冗余）
> 
> 自动扩展（水平扩展）

ReactiveMongo 是 MongoDB 的 Scala 驱动程序，支持非阻塞和异步 I/O 操作。有一个名为 Play-ReactiveMongo 的 Play Framework 插件。它不是一个 Play 插件，但它由 ReactiveMongo 团队支持和维护。

### 注意

这一节需要具备 MongoDB 的相关知识，请参阅[`www.mongodb.org/`](https://www.mongodb.org/)。

要使用它，我们需要做以下操作：

1.  在构建文件中将它作为依赖项包含：

    ```java
    libraryDependencies ++= Seq(
      "org.reactivemongo" %% "play2-reactivemongo" % "0.10.5.0.akka23"
    )
    ```

1.  在`conf/play.plugins`中包含插件：

    ```java
    1100:play.modules.reactivemongo.ReactiveMongoPlugin
    ```

1.  在`conf/application.conf`中添加 MongoDB 服务器详情：

    ```java
    mongodb.servers = ["localhost:27017"]
    mongodb.db = "your_db_name"
    mongodb.credentials.username = "user"
    mongodb.credentials.password = "pwd"
    ```

    或者，使用以下方法：

    ```java
    mongodb.uri = "mongodb://user:password@localhost:27017/your_db_name"
    ```

让我们通过一个示例应用程序来看看这个插件的用法。在我们的应用程序中，我们可能会遇到允许用户以热传感器、烟雾探测器等形式监控其设备活动的情况。

在将设备与安装了我们的应用程序的设备一起使用之前，该设备应注册到这个应用程序中。每个设备都有一个`ownerId`、`deviceId`、其配置和产品信息。因此，让我们假设在注册时，我们以以下格式获得一个 JSON：

```java
{
"deviceId" : "aghd",
"ownerId" : "someUser@someMail.com"
"config" : { "sensitivity" : 4, …},
"info" : {"brand" : "superBrand","os" : "xyz","version" : "2.4", …}
}
```

一旦设备注册，所有者可以更新配置或同意更新产品的软件。软件更新由设备公司处理，我们只需要在我们的应用程序中更新详细信息。

对数据库的查询将是：

+   `Insert`：此查询包括注册设备

+   `Update`：此查询包括更新设备配置或信息

+   `Delete`：此查询发生在设备注销时

+   `Select`：此查询发生在所有者希望查看设备详情时

    使用 Reactive Mongo，设备注册将是：

    ```java
    def registerDevice(deviceId: String, 
                        ownerId: String, 
                  deviceDetails: JsObject): Future[LastError] = { 

        var newDevice = Json.obj("deviceId" -> deviceId, "ownerId" -> ownerId.trim) 
        val config = (deviceDetails \ "configuration").asOpt[JsObject] 
        val metadata = (deviceDetails \ "metadata").asOpt[JsObject] 
        if (!config.isDefined) 
          newDevice = newDevice ++ Json.obj("configuration" -> Json.parse("{}")) 
        if (!metadata.isDefined) 
          newDevice = newDevice ++ Json.obj("metadata" -> Json.parse("{}")) 

        collection.insertJsValue 
      }
    ```

在这个片段中，我们已从可用的设备详细信息中构建了一个 JSON 对象，并将其插入到`devices`中。在这里，集合定义如下：

```java
def db = ReactiveMongoPlugin.db

def collection = db.collection("devices")
```

插入命令接受数据和其类型：

```java
The db operations for fetching a device or removing it are simple,def fetchDevice(deviceId: String): Future[Option[JsObject]] = { 
    val findDevice = Json.obj("deviceId" -> deviceId) 
    collection.find(findDevice).one[JsObject] 
  } 

  def removeDeviceById(deviceId: String): Future[LastError] = { 
    val removeDoc = Json.obj("deviceId" -> deviceId) 
    collection.removeJsValue 
  }
```

这就留下了更新查询。更新是针对配置或信息的单个属性触发的，也就是说，请求只有一个字段，其新值如下：

```java
{ "sensitivity": 4.5}
```

现在，更新此内容的查询将是：

```java
    def updateConfiguration(deviceId: String, 
                     ownerId: String, 
                updatedField: JsObject) = { 
    val property = updatedField.keys.head 
    val propertyValue = updatedField.values.head 
    val toUpdate = Json.obj(s"configuration.$property" -> propertyValue) 
    val setData = Json.obj("$set" -> toUpdate) 
    val documentToUpdate = Json.obj("deviceId" -> deviceId, "ownerId" -> ownerId) 
    collection.updateJsValue, JsValue 
  }
```

当我们希望更新 MongoDB 中给定文档的字段时，我们应该将更新后的数据添加到查询中的`$set`字段。例如，等效的 MongoDB 查询如下所示：

```java
db.devices.update(
   { deviceId: "aghd" ,"ownerId" : "someUser@someMail.com"},
   { $set: { "configuration.sensitivity": 4.5 } }
)
```

# 缓存 API

在 Web 应用程序中，缓存是将动态生成的内容（无论是数据对象、页面还是页面的一部分）在首次请求时存储在内存中的过程。如果后续请求相同的数据，则可以稍后重用，从而减少响应时间并提高用户体验。这些项目可以存储在 Web 服务器或其他软件中，例如代理服务器或浏览器。

Play 有一个最小的缓存 API，它使用 EHCache。如其在网站([`ehcache.org/`](http://ehcache.org/))上所述：

> **Ehcache**是一个开源、基于标准的缓存，用于提升性能、减轻数据库负担并简化可伸缩性。由于它稳健、经过验证且功能全面，因此它是使用最广泛的基于 Java 的缓存。Ehcache 可从进程内、一个或多个节点扩展到混合进程内/进程外配置，缓存大小可达数 TB。

它为表示层以及特定于应用程序的对象提供缓存。它易于使用、维护和扩展。

### 注意

要在 Play 应用程序中使用默认的缓存 API，我们应该如下声明它作为依赖项：

```java
libraryDependencies ++= Seq(cache)
```

使用默认的缓存 API 类似于使用可变的`Map[String, Any]`：

```java
Cache.set("userSession", session)

val maybeSession: Option[UserSession] = Cache.getAsUserSession

Cache.remove("userSession")
```

此 API 通过`EHCachePlugin`提供。该插件负责在启动应用程序时创建一个具有可用配置的 EHCache CacheManager 实例，并在应用程序停止时关闭它。我们将在第十三章中详细讨论 Play 插件，*编写 Play 插件*。基本上，`EHCachePlugin`处理使用 EHCache 在应用程序中所需的所有样板代码，而`EhCacheImpl`提供执行这些操作的方法，例如`get`、`set`和`remove`。它定义如下：

```java
class EhCacheImpl(private val cache: Ehcache) extends CacheAPI {

  def set(key: String, value: Any, expiration: Int) {
    val element = new Element(key, value)
    if (expiration == 0) element.setEternal(true)
    element.setTimeToLive(expiration)
    cache.put(element)
  }

  def get(key: String): Option[Any] = {
    Option(cache.get(key)).map(_.getObjectValue)
  }

  def remove(key: String) {
    cache.remove(key)
  }
}
```

### 注意

默认情况下，插件在`conf`目录中查找`ehcache.xml`文件，如果该文件不存在，则加载`ehcache-default.xml`框架提供的默认配置。

还可以在启动应用程序时使用`ehcache.configResource`参数指定`ehcache`配置的位置。

缓存 API 还简化了处理应用程序客户端和服务器端请求结果的缓存。添加`EXPIRES`和`etag`头可以用来操作客户端缓存，而在服务器端，结果被缓存，因此对应的操作不会在每次调用时计算。

例如，我们可以缓存用于获取不活跃用户详细信息的请求的结果：

```java
def getInactiveUsers = Cached("inactiveUsers") {
  Action {
    val users = User.getAllInactive
    Ok(Json.toJson(users))
  }
}
```

然而，如果我们想每小时更新一次，我们只需明确指定持续时间即可：

```java
def getInactiveUsers = Cached("inactiveUsers").default(3600) {
  Action {
    val users = User.getAllInactive
    Ok(Json.toJson(users))
  }
}
```

所有这些都被`Cached`案例类及其伴随对象处理。案例类定义如下：

```java
case class Cached(key: RequestHeader => String, caching: PartialFunction[ResponseHeader, Duration]) { … }
```

伴随对象提供了生成缓存实例的常用方法，例如根据其状态缓存操作等。

缓存调用中的`apply`方法调用定义如下：

```java
def build(action: EssentialAction)(implicit app: Application) = EssentialAction { request =>
    val resultKey = key(request)
    val etagKey = s"$resultKey-etag"

    // Has the client a version of the resource as fresh as the last one we served?
    val notModified = for {
      requestEtag <- request.headers.get(IF_NONE_MATCH)
      etag <- Cache.getAsString
      if requestEtag == "*" || etag == requestEtag
    } yield Done[Array[Byte], Result](NotModified)

    notModified.orElse {
      // Otherwise try to serve the resource from the cache, if it has not yet expired
      Cache.getAsResult.map(Done[Array[Byte], Result](_))
    }.getOrElse {
      // The resource was not in the cache, we have to run the underlying action
      val iterateeResult = action(request)

      // Add cache information to the response, so clients can cache its content
      iterateeResult.map(handleResult(_, etagKey, resultKey, app))
    }
  }
```

它只是简单地检查结果是否已被修改。如果没有被修改，它尝试从`Cache`中获取结果。如果结果不在缓存中，它将从操作中获取它，并使用`handleResult`方法将其添加到`Cache`中。`handleResult`方法定义如下：

```java
private def handleResult(result: Result, etagKey: String, resultKey: String, app: Application): Result = {
  cachingWithEternity.andThen { duration =>
    // Format expiration date according to http standard
    val expirationDate = http.dateFormat.print(System.currentTimeMillis() + duration.toMillis)
      // Generate a fresh ETAG for it
      val etag = expirationDate // Use the expiration date as ETAG

      val resultWithHeaders = result.withHeaders(ETAG -> etag, EXPIRES -> expirationDate)

      // Cache the new ETAG of the resource
      Cache.set(etagKey, etag, duration)(app)
      // Cache the new Result of the resource
      Cache.set(resultKey, resultWithHeaders, duration)(app)

      resultWithHeaders
    }.applyOrElse(result.header, (_: ResponseHeader) => result)
  }
```

如果指定了持续时间，它将返回该值，否则它将返回默认的一年持续时间。

`handleResult`方法简单地接受结果，添加`etag`、过期头信息，然后将带有给定键的结果添加到`Cache`中。

# 故障排除

以下部分涵盖了某些常见场景：

+   即使查询产生了预期的行为，Anorm 也会在`SqlMappingError`运行时抛出错误（当你期望一行时却有多行），这是一个使用“on duplicate key update”的插入查询。

    这可能发生在使用`executeInsert`执行此类查询时。当我们需要返回自动生成的键时，应使用`executeInsert`方法。如果我们通过重复键更新某些字段，这意味着我们实际上不需要键。我们可以使用`executeUpdate`来添加一个检查，看是否有一行已被更新。例如，我们可能想更新跟踪用户愿望清单的表：

    ```java
    DB.withConnection {
            implicit connection => {

      val updatedRows = SQL"""INSERT INTO wish_list (user_id, product_id, liked_at) VALUES ($userId,$productId,$likedAt)
        ON DUPLICATE KEY UPDATE liked_at=$likedAt, is_deleted=false """.executeUpdate()

      updatedRows == 1
      }
    }
    ```

+   我们能否为单个应用程序使用多个数据库？

    是的，可以使用不同类型的数据库，包括相同类型和不同类型。如果一个应用程序需要这样做，我们可以使用两个或更多不同的关系型数据库或 NoSQL 数据库，或者两者的组合。例如，应用程序可能将其用户数据存储在 SQL 数据库中（因为我们已经知道用户数据的格式），而将用户设备的信息存储在 MongoDB 中（因为设备来自不同的供应商，它们的数据格式可能会变化）。

+   当查询有错误的语法时，Anorm 不会抛出编译错误。是否有配置可以启用此功能？

    它的开发目的是在代码中使用 SQL 查询而无需任何麻烦。开发者预计会将正确的查询传递给 Anorm 方法。为了确保在运行时不会发生此类错误，开发者可以在本地执行查询并在成功后将其用于代码中。或者，还有一些第三方插件提供了类型安全的 DSL，如果它们满足要求，可以替代 Anorm 使用，例如 play-slick 或 scalikejdbc-play-support ([`github.com/scalikejdbc/scalikejdbc-play-support`](https://github.com/scalikejdbc/scalikejdbc-play-support))。

+   是否可以使用其他的缓存机制？

    是的，可以扩展对任何其他缓存的支持，例如 OSCache、SwarmCache、MemCached 等等，或者通过编写类似于 EHCachePlugin 的插件来实现自定义缓存。一些流行的缓存机制已经由个人和/或其他组织开发了 Play 插件。例如，play2-memcached ([`github.com/mumoshu/play2-memcached`](https://github.com/mumoshu/play2-memcached)) 和 Redis 插件 ([`github.com/typesafehub/play-plugins/tree/master/redis`](https://github.com/typesafehub/play-plugins/tree/master/redis))。

# 摘要

在本章中，我们看到了在用 Play 框架构建的应用程序中持久化应用程序数据的不同方法。在这个过程中，我们看到了两种截然不同的方法：一种使用关系型数据库，另一种使用 NoSQL 数据库。为了在关系型数据库中持久化，我们研究了 Anorm 模块和 JDBC 插件的工作方式。为了将 NoSQL 数据库（MongoDB）用于我们应用程序的后端，我们使用了 Play 的 ReactiveMongo 插件。除此之外，我们还看到了如何使用 Play 缓存 API 以及它是如何工作的。

在下一章中，我们将学习如何在 Play 中处理数据流。
