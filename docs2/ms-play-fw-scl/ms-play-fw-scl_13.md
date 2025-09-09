# 第十三章。编写 Play 插件

为了使我们的应用可管理，我们将它们分解为独立的模块。这些模块也可以提取为单独的项目/库。

Play 插件不过是另一个具有额外能力的模块——在启动前、启动时和/或停止 Play 应用时绑定任务。在本章中，我们将看到如何编写自定义插件。

在本章中，我们将涵盖以下主题：

+   插件定义

+   插件声明

+   通过插件公开服务

+   编写插件的技巧

# 插件定义

一个 Play 插件可以通过扩展`play.api.plugin`来定义，其定义如下：

```java
trait Plugin {

  //Called when the application starts.
  def onStart() {}

  // Called when the application stops.
  def onStop() {}

  // Is the plugin enabled?
  def enabled: Boolean = true
}
```

现在，我们可能处于需要在一个应用启动或停止时发送电子邮件的情况，以便管理员可以稍后使用这个时间间隔来监控应用性能并检查为什么它停止。我们可以定义一个插件为我们完成这项工作：

```java
class NotifierPlugin(app:Application) extends Plugin{ 

  private def notify(adminId:String,status:String):Unit = { 

    val time = new Date() 

    val msg = s"The app has been $status at $time" 

    //send email to admin with the msg

    log.info(msg)

  } 

  override def onStart() { 

    val emailId = app.configuration.getString("notify.admin.id").get 

    notify(emailId,"started") 

  } 

  override def onStop() {     

    val emailId = app.configuration.getString("notify.admin.id").get 

    notify(emailId,"stopped") 

  } 

  override def enabled: Boolean = true 

}
```

我们也可以定义利用其他库的插件。我们可能需要构建一个在启动时建立到`Cassandra`（一个 NoSQL 数据库）的连接池，并允许用户稍后使用此池的插件。为了构建此插件，我们将使用 Java 的`cassandra-driver`。然后我们的插件将如下所示：

```java
class CassandraPlugin(app: Application) extends Plugin {

  private var _helper: Option[CassandraConnection] = None

  def helper = _helper.getOrElse(throw new RuntimeException("CassandraPlugin error: CassandraHelper initialization failed"))

  override def onStart() = {

    val appConfig = app.configuration.getConfig("cassandraPlugin").get
    val appName: String = appConfig.getString("appName").getOrElse("appWithCassandraPlugin")

    val hosts: Array[java.lang.String] = appConfig.getString("host").getOrElse("localhost").split(",").map(_.trim)
    val port: Int = appConfig.getInt("port").getOrElse(9042)

    val cluster = Cluster.builder()
      .addContactPoints(hosts: _*)
      .withPort(port).build()

    _helper = try {
      val session = cluster.connect()
      Some(CassandraConnection(hosts, port, cluster, session))
    } catch {
      case e: NoHostAvailableException =>
        val msg =
          s"""Failed to initialize CassandraPlugin.
             |Please check if Cassandra is accessible at
             | ${hosts.head}:$port or update configuration""".stripMargin
        throw app.configuration.globalError(msg)
    }
  }

  override def onStop() = {
    helper.session.close()
    helper.cluster.close()
  }

  override def enabled = true
}
```

在这里，`CassandraConnection`的定义如下：

```java
private[plugin] case class CassandraConnection(hosts: Array[java.lang.String],
  port: Int,
  cluster: Cluster,
session: Session)
```

`cassandra-driver`节点被声明为库依赖项，并在需要的地方导入其类。

### 注意

在插件的`build`定义中，对 Play 的依赖项应标记为提供，因为使用插件的程序已经对 Play 有依赖，如下所示：

```java
libraryDependencies ++= Seq(
 "com.datastax.cassandra" % "cassandra-driver-core" % "2.0.4",
 "com.typesafe.play" %% "play" % "2.3.0" % "provided" )

```

# 插件声明

现在我们已经定义了一个插件，让我们看看 Play 框架是如何识别和启用它来应用于应用的。生产模式和开发模式（静态和可重载应用）的`ApplicationProvider`都依赖于`DefaultApplication`，其定义如下：

```java
class DefaultApplication(
  override val path: File,
  override val classloader: ClassLoader,
  override val sources: Option[SourceMapper],
  override val mode: Mode.Mode) extends Application with WithDefaultConfiguration with WithDefaultGlobal with WithDefaultPlugins
```

`trait WithDefaultPlugins`行负责将插件绑定到应用的生命周期。其定义如下：

```java
trait WithDefaultPlugins {
  self: Application =>
  private[api] def pluginClasses: Seq[String] = {
    import scala.collection.JavaConverters._
    val PluginDeclaration = """([0-9_]+):(.*)""".r
 val pluginFiles = self.classloader.getResources("play.plugins").asScala.toList ++ self.classloader.getResources("conf/play.plugins").asScala.toList

    pluginFiles.distinct.map { plugins =>
      PlayIO.readUrlAsString(plugins).split("\n").map(_.replaceAll("#.*$", "").trim).filterNot(_.isEmpty).map {
        case PluginDeclaration(priority, className) => (priority.toInt, className)
      }
    }.flatten.sortBy(_._1).map(_._2)

  }
...
}
```

因此，我们应该在名为`play.plugins`的文件中声明我们的插件类。从一个或多个`play.plugins`文件中获得的全部插件声明将被合并并排序。每个声明的插件都分配了一个优先级，用于排序。一旦排序，插件将按顺序在应用启动前加载。

应根据插件的依赖关系设置优先级。建议的优先级如下：

+   `100`：当插件没有依赖项时，设置此优先级，例如消息插件（用于`i18n`）

+   `200`：此优先级是为创建和管理数据库连接池的插件设置的

+   `300-500`：此优先级是为依赖于数据库的插件设置的，例如 JPA、Ebean 和 evolutions

### 注意

`10000`被有意保留作为全局插件，以便在所有其他插件加载之后加载。这允许开发者在使用全局对象时无需额外配置即可使用其他插件。

默认的`play.plugins`文件只包含基本的插件声明：

```java
1:play.core.system.MigrationHelper
100:play.api.i18n.DefaultMessagesPlugin
1000:play.api.libs.concurrent.AkkaPlugin
10000:play.api.GlobalPlugin
```

Play 模块中的一些更多插件声明如下：

```java
200:play.api.db.BoneCPPlugin
500:play.api.db.evolutions.EvolutionsPlugin
600:play.api.cache.EhCachePlugin
700:play.api.libs.ws.ning.NingWSPlugin
```

### 注意

通常，Play 插件需要在应用程序的`build`定义中指定为库依赖项。一些插件与`play.plugins`文件捆绑在一起。然而，对于那些没有的，我们需要在我们的应用程序的`conf/play.plugins`文件中设置优先级。

# 通过插件公开服务

一些插件需要为用户提供辅助方法以简化事务，而其他插件则只需在应用程序的生命周期中添加一些任务。例如，我们的`NotifierPlugin`仅在启动和停止时发送电子邮件。然后，可以通过`play.api.Application`的`plugin`方法访问`CassandraPlugin`的方法：

```java
object CassandraHelper {
 private val casPlugin = Play.application.plugin[CassandraPlugin].get

  //complete DB transactions with the connection pool started through the plugin
  def executeStmt(stmt:String) = {
    casPlugin.session.execute(stmt)
  }

}
```

或者，插件也可以提供一个辅助对象：

```java
object Cassandra {
  private val casPlugin = Play.application.plugin[CassandraPlugin].get

  private val cassandraHelper = casPlugin.helper

  /**
   * gets the Cassandra hosts provided in the configuration
   */
  def hosts: Array[java.lang.String] = cassandraHelper.hosts

  /**
    * gets the port number on which Cassandra is running from the configuration
   */
  def port: Int = cassandraHelper.port

  /**
    * gets a reference of the started Cassandra cluster
    * The cluster is built with the configured set of initial contact points
   * and policies at startup
   */
  def cluster: Cluster = cassandraHelper.cluster

  /**
    * gets a reference of the started Cassandra session
    * A new session is created on the cluster at startup
    */
  def session: Session = cassandraHelper.session

  /**
    * executes CQL statements available in given file.
    * Empty lines or lines starting with `#` are ignored.
    * Each statement can extend over multiple lines and must end with a semi-colon.
   * @param fileName - name of the file
   */
  def loadCQLFile(fileName: String): Unit = {
    Util.loadScript(fileName, cassandraHelper.session)
  }

}
```

可用模块的列表维护在[`www.playframework.com/documentation/2.3.x/Modules`](https://www.playframework.com/documentation/2.3.x/Modules)。

# 编写插件的技巧

这里有一些编写插件的技巧：

+   在开始编写插件之前，检查你是否真的需要一个插件来解决你的问题。如果你的问题不需要干预应用程序的生命周期，那么编写一个库会更好。

+   在编写/更新插件的同时，构建一个使用该插件的示例 Play 应用程序。这将允许你仅通过在每次更改时本地发布插件来检查其功能的完整性。

+   如果插件公开了一些服务，尝试提供一个辅助对象。这有助于保持 API 的一致性，并简化开发者的体验。

    例如，Play 提供的多数插件（如`akka`、`jdbc`、`ws`等）都通过提供辅助对象来使 API 可用。插件内部的更改不会影响通过这些对象公开的公共 API。

+   如果可能的话，尽量用足够的测试来支持插件。

+   记录 API 和/或特殊情况。这可能会在将来对使用插件的每个人都有帮助。

# 摘要

Play 插件为我们提供了在应用程序生命周期的特定阶段执行特定任务的灵活性。Play 有一些插件是大多数应用程序通常需要的，例如 Web 服务、认证等。我们讨论了 Play 插件的工作原理以及如何构建自定义插件以满足不同的需求。
