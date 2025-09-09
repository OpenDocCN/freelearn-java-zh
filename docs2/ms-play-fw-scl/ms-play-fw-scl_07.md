# 第七章。玩转全局变量

有时网络应用程序需要生存期超出请求-响应生命周期的应用程序范围内的对象，例如数据库连接、应用程序配置、共享对象和横切关注点（如身份验证、错误处理等）。考虑以下情况：

+   确保应用程序使用的数据库已定义且可访问。

+   当应用程序接收意外的大量流量时，通过电子邮件或任何其他服务进行通知。

+   记录应用程序服务的不同请求。这些日志可以稍后用于分析用户行为。

+   通过时间限制网络应用程序上的某些功能。例如，一些食品订购应用程序仅在上午 11 点到晚上 8 点之间接收订单，而任何其他时间对构建订单的请求都将被阻止，并显示关于时间的信息。

+   通常，当用户发送电子邮件且收件人的电子邮件 ID 错误或未使用时，发送者只有在 12 到 24 小时后才会被通知电子邮件发送失败。在此期间，会尝试再次发送电子邮件。

允许用户在支付因各种原因被拒绝时，使用相同的或不同的支付选项重试的应用程序。

在 Play 框架应用程序中，按照惯例，所有这些各种关注点都可以通过 GlobalSettings 来管理。

在本章中，我们将讨论以下主题：

+   GlobalSettings

+   应用程序生命周期

+   请求-响应生命周期

# GlobalSettings

每个 Play 应用程序都有一个全局对象，可以用来定义应用程序范围内的对象。它还可以用来自定义应用程序的生命周期和请求-响应生命周期。

应用程序的全局对象可以通过扩展 `GlobalSettings` 特质来定义。默认情况下，对象的名称预期为 `Global`，并且假定它位于 `app` 目录中。这可以通过更新 `conf/application.conf` 属性中的 `application.global` 来更改。例如，如果我们希望使用 `app/com/org` 命名空间中的 `AppSettings` 文件：

```java
application.global=app.com.org.AppSettings
```

`GlobalSettings` 特质具有可以用来中断应用程序生命周期和请求-响应生命周期的方法。我们将在以下章节中根据需要查看其方法。

现在，让我们看看它是如何工作的。

通过 Play 框架开发的应用程序由 `Application` 特质的实例表示，因为其创建和构建将由框架本身处理。

`Application` 特质由 `DefaultApplication` 和 `FakeApplication` 扩展。`FakeApplication` 是一个用于测试 Play 应用程序的帮助程序，我们将在第九章测试中看到更多关于它的内容。`DefaultApplication` 定义如下：

```java
class DefaultApplication(
  override val path: File,
  override val classloader: ClassLoader,
  override val sources: Option[SourceMapper],
  override val mode: Mode.Mode) extends Application with WithDefaultConfiguration with WithDefaultGlobal with WithDefaultPlugins
```

`WithDefaultConfiguration`和`WithDefaultPlugins`特质分别用于初始化应用程序的配置和插件对象。`WithDefaultGlobal`特质负责为应用程序设置正确的全局对象。它定义如下：

```java
trait WithDefaultGlobal { 
  self: Application with WithDefaultConfiguration => 

  private lazy val globalClass = initialConfiguration.getString("application.global").getOrElse(initialConfiguration.getString("global").map { g => 
 Play.logger.warn("`global` key is deprecated, please change `global` key to `application.global`") 
 g 
 }.getOrElse("Global")) 

  lazy private val javaGlobal: Option[play.GlobalSettings] = try { 
    Option(self.classloader.loadClass(globalClass).newInstance().asInstanceOf[play.GlobalSettings]) 
  } catch { 
    case e: InstantiationException => None 
    case e: ClassNotFoundException => None 
  } 

  lazy private val scalaGlobal: GlobalSettings = try { 
    self.classloader.loadClass(globalClass + "$").getDeclaredField("MODULE$").get(null).asInstanceOf[GlobalSettings] 
  } catch { 
   case e: ClassNotFoundException if !initialConfiguration.getString("application.global").isDefined => DefaultGlobal 
    case e if initialConfiguration.getString("application.global").isDefined => { 
      throw initialConfiguration.reportError("application.global", s"Cannot initialize the custom Global object ($globalClass) (perhaps it's a wrong reference?)", Some(e)) 
    } 
  } 

  private lazy val globalInstance: GlobalSettings = Threads.withContextClassLoader(self.classloader) { 
 try { 
 javaGlobal.map(new j.JavaGlobalSettingsAdapter(_)).getOrElse(scalaGlobal) 
 } catch { 
 case e: PlayException => throw e 
 case e: ThreadDeath => throw e 
 case e: VirtualMachineError => throw e 
 case e: Throwable => throw new PlayException( 
 "Cannot init the Global object", 
 e.getMessage, 
 e 
 ) 
 } 
 } 

  def global: GlobalSettings = { 
    globalInstance 
  } 
}
```

`globalInstance`对象是用于此应用程序的`global`对象。它设置为`javaGlobal`或`scalaGlobal`，具体取决于应用程序。如果应用程序没有为应用程序配置自定义的 Global 对象，则应用程序的`global`设置为`DefaultGlobal`。它定义如下：

```java
object DefaultGlobal extends GlobalSettings
```

# 应用程序的生命周期

应用程序的生命周期有两个状态：**运行**和**停止**。这些是应用程序状态发生变化的时间。有时，我们需要在状态变化之前或之后立即执行某些操作。

Play 应用程序使用 Netty 服务器。为此，使用具有相同名称的类。它定义如下：

```java
class NettyServer(appProvider: ApplicationProvider, port: Option[Int], sslPort: Option[Int] = None, address: String = "0.0.0.0", val mode: Mode.Mode = Mode.Prod) extends Server with ServerWithStop { … }
```

此类负责将应用程序绑定或引导到服务器。

`ApplicationProvider`特质定义如下：

```java
trait ApplicationProvider {
  def path: File
  def get: Try[Application]
  def handleWebCommand(requestHeader: play.api.mvc.RequestHeader): Option[Result] = None
}
```

`ApplicationProvider`的实现必须创建并初始化一个应用程序。目前，有三种不同的`ApplicationProvider`实现。它们如下：

+   `StaticApplication`：这个用于生产模式（代码更改不会影响已运行的应用程序的模式）。

+   `ReloadableApplication`：这个用于开发模式（这是一个启用了连续编译的模式，以便开发者可以在保存时看到应用程序更改的影响，如果应用程序正在运行的话）。

+   `TestApplication`：这个用于测试模式（通过测试启动一个模拟应用程序的模式）。

`StaticApplication`和`ReloadableApplication`都初始化一个`DefaultApplication`。`StaticApplication`用于生产模式，定义如下：

```java
class StaticApplication(applicationPath: File) extends ApplicationProvider {

  val application = new DefaultApplication(applicationPath, this.getClass.getClassLoader, None, Mode.Prod)

  Play.start(application)

  def get = Success(application)
  def path = applicationPath
}
```

`ReloadableApplication`用于开发模式，但由于类定义很大，让我们看看使用`DefaultApplication`的相关代码行：

```java
class ReloadableApplication(buildLink: BuildLink, buildDocHandler: BuildDocHandler) extends ApplicationProvider {
...
// First, stop the old application if it exists
    Play.stop()

    val newApplication = new DefaultApplication(reloadable.path, projectClassloader, Some(new SourceMapper {
        def sourceOf(className: String, line: Option[Int]) = {
          Option(buildLink.findSource(className, line.map(_.asInstanceOf[java.lang.Integer]).orNull)).flatMap {
            case Array(file: java.io.File, null) => Some((file, None))
            case Array(file: java.io.File, line: java.lang.Integer) => Some((file, Some(line)))
            case _ => None
            }
          }
              }), Mode.Dev) with DevSettings {
                import scala.collection.JavaConverters._
                lazy val devSettings: Map[String, String] = buildLink.settings.asScala.toMap
              }

              Play.start(newApplication)
...
}
```

对于`StaticApplication`，应用程序只创建和启动一次，而`ReloadableApplication`则会停止现有应用程序，创建并启动一个新的应用程序。`ReloadableApplication`用于开发模式，以便开发者可以做出更改并看到它们的效果，而无需每次都手动重新加载应用程序。

`ApplicationProvider`和`NettyServer`的使用与此类似：

```java
val appProvider = new ReloadableApplication(buildLink, buildDocHandler)
val server = new NettyServer(appProvider, httpPort, httpsPort, mode = Mode.Dev)
```

在下一节中，我们将讨论 GlobalSettings 中可用的方法，这些方法使我们能够挂钩到应用程序的生命周期。

## 干预应用程序的生命周期

考虑到我们的应用程序有以下规格：

+   在开始应用程序之前，我们需要确保 `/opt/dev/appName` 目录存在并且可以被应用程序访问。我们应用程序中的一个名为 `ResourceHandler.initialize` 的方法来完成这个任务。

+   使用 `DBHandler.createSchema` 方法在启动时创建所需的模式。此方法不会删除已存在的模式。这确保了在重新启动应用程序时应用程序的数据不会丢失，并且模式仅在应用程序首次启动时生成。

+   当使用 `Mailer.sendLogs` 方法停止应用程序时，创建电子邮件应用程序日志。此方法将应用程序日志作为附件通过电子邮件发送到配置文件中设置为 `adminEmail` 的 `emailId`。这用于追踪应用程序关闭的原因。

Play 提供了允许我们挂钩到应用程序的生命周期并完成此类任务的方法。`GlobalSettings` 特性具有辅助执行这些任务的方法。如果需要，这些方法可以通过 `Global` 对象进行覆盖。

为了满足前面描述的应用程序规范，在 Play 应用程序中我们只需要定义一个 `Global` 对象，如下所示：

```java
object Global extends GlobalSettings {

  override def beforeStart(app: Application): Unit = {
    ResourceHandler.initialize
  }

  override def onStart(app: Application):Unit={
    DBHandler.createSchema
  }

  override def onStop(app: Application): Unit = {
    Mailer.sendLogs
  }
}
```

`ResourceHandler.initialize`、`DBHandler.createSchema` 和 `Mailer.sendLogs` 方法是针对我们的应用程序特定的，由我们定义，而不是由 Play 提供。

现在我们知道了如何挂钩到应用程序的生命周期，让我们仔细看看它是如何工作的。

深入挖掘应用程序的生命周期，我们可以看到所有 `ApplicationProvider` 的实现都使用 `Play.start` 方法来初始化一个应用程序。`Play.start` 方法定义如下：

```java
def start(app: Application) {

    // First stop previous app if exists
    stop()

    _currentApp = app

    // Ensure routes are eagerly loaded, so that the reverse routers are correctly
    // initialized before plugins are started.
    app.routes
    Threads.withContextClassLoader(classloader(app)) {
      app.plugins.foreach(_.onStart())
    }

    app.mode match {
      case Mode.Test =>
      case mode => logger.info("Application started (" + mode + ")")
    }

  }
```

此方法确保在将应用程序设置为 `_currentApp` 之后立即调用每个插件的 `onStart` 方法。`GlobalPlugin` 默认添加到所有 Play 应用程序中，并定义为：

```java
class GlobalPlugin(app: Application) extends Plugin {

  // Call before start now
  app.global.beforeStart(app)

  // Called when the application starts.
  override def onStart() {
    app.global.onStart(app)
  }

  //Called when the application stops.
  override def onStop() {
    app.global.onStop(app)
  }

}
```

在前面的代码片段中，`app.global` 指的是为应用程序定义的 `GlobalSettings`。因此，`GlobalPlugin` 确保调用应用程序 `GlobalSettings` 的适当方法。

在插件初始化时调用 `beforeStart` 方法。

现在，我们只需要弄清楚 `onStop` 是如何被调用的。一旦应用程序停止，`ApplicationProvider` 就没有控制权了，因此使用 Java 运行时关闭钩子来确保应用程序停止后执行某些任务。以下是 `NettyServer.createServer` 方法中的相关代码：

```java
Runtime.getRuntime.addShutdownHook(new Thread { 
 override def run { 
 server.stop() 
 } 
 })

```

在这里，运行时是 `java.lang.Runtime`（有关相同内容的 Java 文档可在 [`docs.oracle.com/javase/7/docs/api/java/lang/Runtime.html`](http://docs.oracle.com/javase/7/docs/api/java/lang/Runtime.html) 查找），而 `server` 是 `NettyServer` 的一个实例。`NettyServer` 的 `stop` 方法定义如下：

```java
override def stop() {

    try {
      Play.stop()
    } catch {
      case NonFatal(e) => Play.logger.error("Error while stopping the application", e)
    }

    try {
      super.stop()
    } catch {
      case NonFatal(e) => Play.logger.error("Error while stopping logger", e)
    }

    mode match {
      case Mode.Test =>
      case _ => Play.logger.info("Stopping server...")
    }

    // First, close all opened sockets
    allChannels.close().awaitUninterruptibly()

    // Release the HTTP server
    HTTP.foreach(_._1.releaseExternalResources())

    // Release the HTTPS server if needed
    HTTPS.foreach(_._1.releaseExternalResources())

    mode match {
      case Mode.Dev =>
        Invoker.lazySystem.close()
        Execution.lazyContext.close()
      case _ => ()
    }
  }
```

在这里，使用 `Invoker.lazySystem.close()` 调用来关闭 Play 应用程序内部使用的 ActorSystem。`Execution.lazyContext.close()` 调用是为了关闭 Play 的内部 `ExecutionContext`。

`Play.stop` 方法定义如下：

```java
 def stop() {
    Option(_currentApp).map { app =>
      Threads.withContextClassLoader(classloader(app)) {
        app.plugins.reverse.foreach { p =>
 try {
 p.onStop()
 } catch { case NonFatal(e) => logger.warn("Error stopping plugin", e) }
 }
      }
    }
    _currentApp = null
  }
```

此方法以相反的顺序调用所有已注册插件的 `onStop` 方法，因此 GlobalPlugin 的 `onStop` 方法被调用，并最终调用为应用程序定义的 `GlobalSetting` 的 `onStop` 方法。在这个过程中遇到的任何错误都被记录为警告，因为应用程序即将停止。

我们现在可以在应用程序的生命周期中添加任何任务，例如在启动前创建数据库模式，初始化全局对象，或者在停止时清理临时数据。

我们已经涵盖了应用程序的生命周期，现在让我们看看请求-响应生命周期。

# 请求-响应生命周期

Play 框架默认使用 Netty，因此请求由 NettyServer 接收。

Netty 允许执行各种操作，包括通过处理器进行自定义编码。我们可以定义一个处理器，将请求转换为所需的响应，并在启动应用程序时将其提供给 Netty。为了将 Play 应用程序与 Netty 集成，使用 `PlayDefaultUpstreamHandler`。

### 注意

关于 Netty 中使用的请求的更多信息，请参阅 Netty 文档[`netty.io/wiki/user-guide-for-4.x.html`](http://netty.io/wiki/user-guide-for-4.x.html)和 Netty ChannelPipeline 文档[`netty.io/4.0/api/io/netty/channel/ChannelPipeline.html`](http://netty.io/4.0/api/io/netty/channel/ChannelPipeline.html)。

`PlayDefaultUpstreamHandler` 扩展了 `org.jboss.netty.channel.SimpleChannelUpstreamHandler` 以处理 HTTP 和 WebSocket 请求。它在以下方式启动应用程序到 Netty 时使用：

```java
val defaultUpStreamHandler = new PlayDefaultUpstreamHandler(this, allChannels)
```

`SimpleChannelUpStreamHandler` 的 `messageReceived` 方法负责对收到的请求采取行动。`PlayDefaultUpstreamHandler` 覆盖了此方法，以便将请求发送到我们的应用程序。此方法太长（包括注释和空白行约为 260 行），所以我们在这里只查看相关块。

首先，为接收到的消息创建一个 Play `RequestHeader`，并找到其对应的行为：

```java
val (requestHeader, handler: Either[Future[Result], (Handler, Application)]) = Exception.allCatch[RequestHeader].either {
    val rh = tryToCreateRequest
            // Force parsing of uri
            rh.path
            rh
          }.fold(
            e => {
              //Exception Handling
              ...
            },
            rh => server.getHandlerFor(rh) match {
              case directResult @ Left(_) => (rh, directResult)
              case Right((taggedRequestHeader, handler, application)) => (taggedRequestHeader, Right((handler, application)))
            }
          )
```

在前面的代码片段中，`tryToCreateRequest` 方法生成了 `RequestHeader`，并且在这个过程中遇到的任何异常都被处理了。然后通过 `server.getHandlerFor(rh)` 获取 `RequestHeader rh` 的动作。在这里，一个 `server` 是服务器特质的实例，而 `getHandlerFor` 方法利用了应用程序的 `global` 对象及其 `onRequestReceived` 方法：

```java
try {
        applicationProvider.get.map { application =>
 application.global.onRequestReceived(request) match {
 case (requestHeader, handler) => (requestHeader, handler, application)
 }
 }
      } catch {
  //Exception Handling
...
}
```

在 `PlayDefaultUpstreamHandler` 的 `messageReceived` 方法中，从 `server.getHandlerFor` 获取的动作最终被调用，从而产生响应。

`PlayDefaultUpStreamHandler` 与应用程序的大部分交互都是通过其全局对象进行的。在下一节中，我们将看到与请求-响应生命周期相关的 GlobalSettings 中的可用方法。

## 玩弄请求-响应生命周期

`GlobalSettings` 特性包含与应用程序生命周期不同阶段以及其请求-响应生命周期相关的各种方法。使用请求相关的钩子，我们可以在接收到请求、找不到请求的操作等情况下定义业务逻辑。

以下是与请求相关的各种方法：

+   `onRouteRequest`：此方法使用路由器来识别给定 `RequestHeader` 的操作

+   `onRequestReceived`：这会产生 `RequestHeader` 和其操作。内部，它调用 `onRouteRequest` 方法

+   `doFilter`：这向应用程序添加了一个过滤器

+   `onError`：这是一个处理处理过程中异常的方法

+   `onHandlerNotFound`：当找不到 RequestHeader 对应的操作时使用

+   `onBadRequest`：当请求体不正确时内部使用

+   `onRequestCompletion`：用于在请求成功处理之后执行操作

### 操作请求及其响应

在某些应用程序中，强制过滤、修改、重定向请求及其响应是必要的。考虑以下示例：

+   任何服务的请求都必须包含包含会话详情和用户身份的头，除非是登录、注册和忘记密码等实例

+   所有以 `admin` 开头的路径请求都必须受到用户角色的限制

+   如果可能，将请求重定向到区域站点（例如 Google）

+   向请求或响应添加额外的字段

可以使用 `onRequestReceived`、`onRouteRequest`、`doFilter` 和 `onRequestCompletion` 方法来拦截请求或其响应，并根据要求对其进行操作。

让我们看看 `onRequestReceived` 方法：

```java
def onRequestReceived(request: RequestHeader): (RequestHeader, Handler) = {
    val notFoundHandler = Action.async(BodyParsers.parse.empty)(this.onHandlerNotFound)
    val (routedRequest, handler) = onRouteRequest(request) map {
      case handler: RequestTaggingHandler => (handler.tagRequest(request), handler)
      case otherHandler => (request, otherHandler)
    } getOrElse {
    // We automatically permit HEAD requests against any GETs without the need to
      // add an explicit mapping in Routes
      val missingHandler: Handler = request.method match {
        case HttpVerbs.HEAD =>
          new HeadAction(onRouteRequest(request.copy(method = HttpVerbs.GET)).getOrElse(notFoundHandler))
        case _ =>
          notFoundHandler
      }
      (request, missingHandler)
    }

    (routedRequest, doFilter(rh => handler)(routedRequest))
  }
```

它使用 `onRouteRequest` 和 `doFilter` 方法获取给定 `RequestHeader` 对应的处理程序。如果没有找到处理程序，则发送 `onHandlerNotFound` 的结果。

由于 `onRequestReceived` 方法在请求处理方式中起着关键作用，有时可能更简单的是覆盖 `onRouteRequest` 方法。

`onRouteRequest` 方法定义如下：

```java
def onRouteRequest(request: RequestHeader): Option[Handler] = Play.maybeApplication.flatMap(_.routes.flatMap {
    router =>
      router.handlerFor(request)
  })
```

在这里，路由器是应用程序的 `router` 对象。默认情况下，它是编译时从 `conf/routes` 生成的对象。路由器扩展了 `Router.Routes` 特性，并且 `handlerFor` 方法定义在这个特性中。

让我们尝试实现一个解决方案，以阻止对 `login`、`forgotPassword` 和 `register` 之外的服务进行请求，如果请求头没有会话和用户详情。我们可以通过覆盖 `onRouteRequest` 来做到这一点：

```java
override def onRouteRequest(requestHeader: RequestHeader) = {
    val path = requestHeader.path

    val pathConditions = path.equals("/") ||
      path.startsWith("/register") ||
      path.startsWith("/login") ||
      path.startsWith("/forgot")

   if (!pathConditions) {
      val tokenId = requestHeader.headers.get("Auth-Token")
      val userId = requestHeader.headers.get("Auth-User")
      if (tokenId.isDefined && userId.isDefined) {
        val isValidSession = SessionDetails.validateSession(SessionDetails(userId.get.toLong, tokenId.get))
        if (isValidSession) {
          super.onRouteRequest(request)
        }
        else Some(controllers.SessionController.invalidSession)
      }
      else {
        Some(controllers.SessionController.invalidSession)
      }
    }
    else {
      super.onRouteRequest(request)
    }
  }
```

首先，我们检查请求的路径是否有受限访问。如果有，我们检查必要的头是否可用且有效。只有在这种情况下，才会返回相应的 `Handler`，否则返回无效会话的 `Handler`。如果需要根据用户的角色来控制访问，可以遵循类似的方法。

我们还可以使用 `onRouteRequest` 方法为旧版已弃用的服务提供兼容性。例如，如果旧版应用程序有一个 `GET /user/:userId` 服务，现在已被修改为 `/api/user/:userId`，并且有其他应用程序依赖于这个应用程序，那么我们的应用程序应该支持这两个路径的请求。然而，路由文件只列出了新路径和服务，这意味着在尝试访问应用程序支持的路径之前，我们应该处理这些请求：

```java
override def onRouteRequest(requestHeader: RequestHeader) = {
  val path = requestHeader.path

  val actualPath = getSupportedPath(path)
  val customRequestHeader = requestHeader.copy(path = actualPath)

  super.onRouteRequest(customRequestHeader)
}
```

`getSupportedPath` 是一个自定义方法，它为给定的旧路径提供一个新路径。我们创建一个新的 `RequestHeader` 并带有更新后的字段，然后将这个新 `RequestHeader` 传递给后续的方法，而不是使用原始的 `RequestHeader`。

同样，我们也可以添加/修改 `RequestHeader` 的头部或任何其他字段。

`doFilter` 方法可以用来添加过滤器，类似于在第二章 *定义动作* 中展示的：

```java
object Global extends GlobalSettings {
  override def doFilter(action: EssentialAction): EssentialAction = HeadersFilter.noCache(action)
}
```

或者，我们可以扩展 `WithFilters` 类而不是 `GlobalSettings`：

```java
object Global extends WithFilters(new CSRFFilter()) with GlobalSettings
```

`WithFilters` 类扩展了 `GlobalSettings` 并用构造函数中传入的 `Filter` 覆盖了 `doFilter` 方法。它定义如下：

```java
class WithFilters(filters: EssentialFilter*) extends GlobalSettings {
  override def doFilter(a: EssentialAction): EssentialAction = {
    Filters(super.doFilter(a), filters: _*)
  }
}
```

`onRequestCompletion` 方法可以在请求被处理后执行特定任务。例如，假设应用程序需要从特定的 GET 请求（如搜索）中持久化数据。这有助于理解和分析用户在我们的应用程序中寻找什么。在获取数据之前从请求中持久化信息可以显著增加响应时间并损害用户体验。因此，在发送响应之后进行此操作会更好：

```java
override def onRequestCompletion(requestHeader: RequestHeader) {
  if(requestHeader.path.startsWith("/search")){
    //code to persist request parameters, time, etc
  }}
```

### 处理错误和异常

一个应用程序如果不能处理错误和异常是无法存在的。根据业务逻辑，它们被处理的方式可能因应用程序而异。Play 提供了一些标准实现，这些实现可以在应用程序的全局对象中被覆盖。当发生异常时，会调用 `onError` 方法，定义如下：

```java
  def onError(request: RequestHeader, ex: Throwable): Future[Result] = {
    def devError = views.html.defaultpages.devError(Option(System.getProperty("play.editor"))) _
    def prodError = views.html.defaultpages.error.f
    try {
      Future.successful(InternalServerError(Play.maybeApplication.map {
        case app if app.mode == Mode.Prod => prodError
        case app => devError
      }.getOrElse(devError) {
        ex match {
          case e: UsefulException => e
          case NonFatal(e) => UnexpectedException(unexpected = Some(e))
        }
      }))
    } catch {
      case NonFatal(e) => {
        Logger.error("Error while rendering default error page", e)
        Future.successful(InternalServerError)
      }
    }
  }
```

`UsefulException` 是一个抽象类，它扩展了 `RuntimeException`。它由 `PlayException` 辅助类扩展。`onError` 的默认实现（在之前的代码片段中）简单地检查应用程序是否处于生产模式或开发模式，并发送相应的视图作为 `Result`。这种方法会导致 `defaultpages.error` 或 `defaultpages.devError` 视图。

假设我们想要发送一个状态为 500 的响应，并包含异常。我们可以通过覆盖 `onError` 方法轻松实现：

```java
override def onError(request: RequestHeader, ex: Throwable) = {
  log.error(ex)
  InternalServerError(ex.getMessage)
}
```

当用户发送一个在 `conf/routes` 中未定义路径的请求时，会调用 `onHandlerNotFound` 方法。它定义如下：

```java
def onHandlerNotFound(request: RequestHeader): Future[Result] = {
  Future.successful(NotFound(Play.maybeApplication.map {
    case app if app.mode != Mode.Prod => views.html.defaultpages.devNotFound.f
    case app => views.html.defaultpages.notFound.f
  }.getOrElse(views.html.defaultpages.devNotFound.f)(request, Play.maybeApplication.flatMap(_.routes))))
  }
```

它会根据应用启动的模式发送一个视图作为响应。在开发模式下，视图包含一个错误消息，告诉我们为该路由定义了一个动作，以及带有请求类型的支持路径列表。如果需要，我们可以覆盖这个行为。

在以下情况下会调用`onBadRequest`方法：

+   请求被发送，并且其对应动作具有不同的内容类型

+   请求中缺少一些参数，并且在解析时请求抛出异常

它被定义为如下：

```java
def onBadRequest(request: RequestHeader,
  error: String): Future[Result] = {
    Future.successful(BadRequest(views.html.defaultpages.badRequest(request, error)))
}
```

此方法也会发送一个视图作为响应，但在大多数应用中，我们希望发送带有错误消息的`BadRequest`而不是视图。这可以通过覆盖默认实现来实现，如下所示：

```java
import play.api.mvc.{Result, RequestHeader,Results}
 override def onBadRequest(request: RequestHeader,
                           error: String): Future[Result] = {
    Future{
      Results.BadRequest(error)
    }
  }
```

# 摘要

在本章中，我们看到了通过全局插件提供给 Play 应用的特性。通过扩展`GlobalSettings`，我们可以挂钩到应用的生命周期，并在不同阶段执行各种任务。除了用于应用生命周期的钩子外，我们还讨论了用于请求-响应生命周期的钩子，通过这些钩子我们可以拦截请求和响应，并在必要时修改它们。
