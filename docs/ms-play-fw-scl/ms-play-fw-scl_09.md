# 第九章。测试

测试是交叉检查应用程序/流程实现的过程。它将缺点暴露出来。当你升级/降级一个或多个依赖项时，它可能非常有用。根据不同的编程实践，测试可以划分为各种类别，但在这章中，我们只将讨论两种类型的测试：

+   **单元测试**：这些是检查特定代码部分功能的测试

+   **功能测试**：这些是检查特定操作的测试，通常编写来验证与用例或场景相关的代码是否正常工作

在以下章节中，我们将看到我们可以使用 **Specs2** 和 **ScalaTest** 测试 Play 应用的不同方式。

### 注意

使用 Specs2 和 ScalaTest 任意一个库编写的测试是相似的。主要区别在于关键字、语法和风格。由于不同的开发者可能有不同的偏好，在本章中，使用这两个库定义测试，以方便起见。大多数使用 Specs2 编写的测试以 `'Spec'` 结尾，而使用 ScalaTest 编写的测试以 `'Test'` 结尾。

# 编写测试的设置

Play 随 `Specs2` 打包，因为这是内部用于测试的库。它默认提供对使用 Specs2 测试应用程序的支持，即不需要额外的库依赖项。

在早期使用 `ScalaTest` 是困难的，但现在，Play 也提供了使用 ScalaTest 的辅助方法。尽管它来自传递依赖，但我们需要添加一个库依赖项来使用这些辅助方法：

```java
val appDependencies = Seq(
  "org.scalatestplus" %% "play" % "1.1.0" % "test"
)
```

### 注意

`org.scalatestplus.play` 的 1.1.0 版本与 Play 2.3.x 兼容。当与 Play 的其他版本一起工作时，最好在 [`www.scalatest.org/plus/play/versions`](http://www.scalatest.org/plus/play/versions) 检查兼容性。

# 单元测试

单元测试可以像任何 Scala 项目一样编写。例如，假设我们有一个实用方法 `isNumberInRange`，它接受一个字符串并检查它是否在范围 [0,3600] 内。它被定义为以下内容：

```java
def isNumberInRange(x:String):Boolean = {
    val mayBeNumber = Try{x.toDouble}
    mayBeNumber match{
      case Success(n) => if(n>=0 && n<=3600) true else false
      case Failure(e) => false
    }
  }
```

让我们使用 `Specs2` 编写一个单元测试来检查这个函数：

```java
class UtilSpec extends Specification {

    "range method" should {

    "fail for Character String" in {
      Util.isNumberInRange("xyz") should beFalse
    }

    "fail for Java null" in {
      Util.isNumberInRange(null) should beFalse
    }

    "fail for Negative numbers" in {
      Util.isNumberInRange("-2") should beFalse
    }

    "pass for valid number" in {
      Util.isNumberInRange("1247") should beTrue
    }

    "pass for 0" in {
      Util.isNumberInRange("0") should beTrue
    }

    "pass for 3600" in {
      Util.isNumberInRange("3600") should beTrue
    }

  }
}
```

这些场景也可以使用 `ScalaTest` 通过轻微修改来编写：

```java
class UtilTest extends FlatSpec with Matchers {

  "Character String" should "not be in range" in {
    Util.isNumberInRange("xyz") should be(false)
  }

  "Java null" should "not be in range" in {
    Util.isNumberInRange(null) should be(false)
  }

  "Negative numbers" should "not be in range" in {
    Util.isNumberInRange("-2") should be(false)
  }

  "valid number" should "be in range" in {
    Util.isNumberInRange("1247") should be(true)
  }

  "0" should "be in range" in {
    Util.isNumberInRange("0") should be(true)
  }

  "3600" should "be in range" in {
    Util.isNumberInRange("3600") should be(true)
  }
}
```

需要依赖外部依赖和数据服务层的单元测试应该使用 **模拟** 来定义。模拟是模拟实际行为的过程。**Mockito**、**ScalaMock**、**EasyMock** 和 **jMock** 是一些便于模拟的库。

# 拆解 PlaySpecification

使用 Specs2 编写的测试也可以按照以下方式编写：

```java
class UtilSpec extends PlaySpecification {...}
```

`PlaySpecification` 是一个特质，它提供了使用 Specs2 测试 Play 应用程序所需的辅助方法。它被定义为：

```java
trait PlaySpecification extends Specification
    with NoTimeConversions
    with PlayRunners
    with HeaderNames
    with Status
    with HttpProtocol
    with DefaultAwaitTimeout
    with ResultExtractors
    with Writeables
    with RouteInvokers
    with FutureAwaits {
}
```

让我们扫描这些特质暴露的 API，以了解其重要性：

+   `Specification` 和 `NoTimeConversions` 是 Specs2 的特质。`NoTimeConversions` 可以用来禁用时间转换。

+   `PlayRunners` 提供了在运行中的应用程序或服务器中执行代码块的帮助方法，可以指定或不指定浏览器。

+   `HeaderNames` 和 `Status` 分别定义了所有标准 HTTP 头和 HTTP 状态码的常量，以及它们的相关名称，如下所示：

    ```java
    HeaderNames.ACCEPT_CHARSET = "Accept-Charset"
    Status.FORBIDDEN = 403
    ```

    +   `HttpProtocol` 定义了与 HTTP 协议相关的常量：

        ```java
        object HttpProtocol extends HttpProtocol
        trait HttpProtocol {
          // Versions
          val HTTP_1_0 = "HTTP/1.0"

          val HTTP_1_1 = "HTTP/1.1"

          // Other HTTP protocol values
          val CHUNKED = "chunked"
        }
        ```

    +   `ResultExtractors` 提供了从 HTTP 响应中提取数据的帮助方法，这些方法的数据类型为 `Future[Result]`。这些方法如下：

        +   `charset(of: Future[Result])(implicit timeout: Timeout): Option[String]`

        +   `contentAsBytes(of: Future[Result])(implicit timeout: Timeout): Array[Byte]`

        +   `contentAsJson(of: Future[Result])(implicit timeout: Timeout): JsValue`

        +   `contentAsString(of: Future[Result])(implicit timeout: Timeout): String`

        +   `contentType(of: Future[Result])(implicit timeout: Timeout): Option[String]`

        +   `cookies(of: Future[Result])(implicit timeout: Timeout): Cookies`

        +   `flash(of: Future[Result])(implicit timeout: Timeout): Flash`

        +   `header(header: String, of: Future[Result])(implicit timeout: Timeout): Option[String]`

        +   `headers(of: Future[Result])(implicit timeout: Timeout): Map[String, String]`

        +   `redirectLocation(of: Future[Result])(implicit timeout: Timeout): Option[String]`

        +   `session(of: Future[Result])(implicit timeout: Timeout): Session`

        +   `status(of: Future[Result])(implicit timeout: Timeout): Int`

    这些方法调用中的 `implicit Timeout` 由 `DefaultAwaitTimeout` 特质提供，默认超时设置为 20 秒。这可以通过在场景作用域内提供隐式超时来覆盖。

+   `RouteInvokers` 提供了使用 `Router` 调用给定请求的相应 `Action` 的方法。这些方法如下：

    +   `routeT(implicit w: Writeable[T]): Option[Future[Result]]`

    +   `routeT(implicit w: Writeable[T]): Option[Future[Result]]`

    +   `routeT(implicit w: Writeable[T]): Option[Future[Result]]`

    +   `routeT(implicit w: Writeable[T]): Option[Future[Result]]`

    +   `callT(implicit w: Writeable[T]): Future[Result]`

    +   `callT(implicit w: Writeable[T]): Future[Result]`

    这些方法调用中的 `implicit Writable` 由 `Writeables` 特质提供。`call` 方法是从 `EssentialActionCaller` 继承的。

+   `FutureAwaits` 特质提供了在指定或不指定等待时间的情况下等待请求的方法。

### 注意

虽然支持 Play 应用的 ScalaTest 库有一个 `PlaySpec` 抽象类，但没有 `PlaySpecification` 的等效物。取而代之的是，有一个帮助对象，定义如下：

```java
object Helpers extends PlayRunners
  with HeaderNames
  with Status
  with HttpProtocol
  with DefaultAwaitTimeout
  with ResultExtractors
  with Writeables
  with EssentialActionCaller
  with RouteInvokers
  with FutureAwaits
```

`PlaySpec` 定义如下：

```java
abstract class PlaySpec extends WordSpec with MustMatchers with OptionValues with WsScalaTestClient
```

因此，导入 `play.api.test.Helpers` 也足以使用仅有的帮助方法。

对于以下部分，关于使用 Specs2 的测试，我们将扩展 PlaySpecification，而对于 ScalaTest，我们假设已经导入了 `play.api.test.Helpers`，并且测试扩展到 `PlaySpec`。

# 单元测试控制器

我们可能有一个简单的项目，其中包含 `User` 模型和 `UserRepo`，定义如下：

```java
case class User(id: Option[Long], loginId: String, name: Option[String],
  contactNo: Option[String], dob: Option[Long], address: Option[String])

object User{
  implicit val userWrites = Json.writes[User]
}

trait UserRepo {
  def authenticate(loginId: String, password: String): Boolean

  def create(u: User, host: String, password: String): Option[Long]

  def update(u: User): Boolean

  def findByLogin(loginId: String): Option[User]

  def delete(userId: Long): Boolean

  def find(userId: Long): Option[User]

  def getAll: Seq[User]

  def updateStatus(userId: Long, isActive: Boolean): Int

  def updatePassword(userId: Long, password: String): Int
}
```

在这个项目中，我们需要测试 `UserController` 的 `getUser` 方法——这是一个定义用来访问用户详情的控制器，这些详情由用户模型处理，其中 `UserController` 定义如下：

```java
object UserController extends Controller {

  /* GET a specific user's details */
  def getUser(userId: Long) = Action {
    val u = AnormUserRepo.find(userId)
    if (u.isEmpty) {
      NoContent
    }
 else {
      Ok(Json.toJson(u))
    }
  }
....
}
```

`AnormUserRepo` 是 `UserRepo` 的一个实现，它使用 Anorm 进行数据库事务。`UserController` 中的方法在路由文件中映射如下：

```java
GET        /api/user/:userId        controllers.UserController.getUser(userId:Long)
```

由于测试库尚未完全支持模拟 Scala 对象，因此有几种不同的方法可以单元测试控制器。这些方法如下：

1.  在特质中定义控制器的所有方法，然后这个特质可以被一个对象扩展，同时特质的功能被测试

1.  将控制器定义为类，并使用依赖注入连接其他所需服务

这两种方法都需要我们修改我们的应用程序代码。我们可以选择最适合我们编码实践的一种。让我们看看这些更改是什么，以及如何在以下部分中编写相应的测试。

## 使用特质定义控制器

在这种方法中，我们定义了控制器中的所有方法在一个特质中，并通过扩展这个特质来定义控制器。例如，`UserController` 应该定义为如下所示：

```java
trait BaseUserController extends Controller {
this: Controller =>

  val userRepo:UserRepo

  /* GET a specific user's details */
  def getUser(userId: Long) = Action {
    val u = userRepo.find(userId)
    if (u.isEmpty) {
      NoContent
    } else {
      Ok(Json.toJson(u))
    }
  }

}

object UserController extends BaseUserController{
  val userRepo = AnormUserRepo
}
```

现在，我们可以使用 Specs2 编写 `BaseUserController` 特质的测试——`UserControllerSpec`，如下所示：

```java
class UserControllerSpec extends Specification with Mockito {

  "UserController#getUser" should {
    "be valid" in {
      val userRepository = mock[UserRepo]
      val defaultUser = User(Some(1), "loginId", Some("name"), Some("contact_no"), Some(20L), Some("address"))
      userRepository.find(1) returns Option(defaultUser)

      class TestController extends Controller with BaseUserController{
        val userRepo = userRepository
      }

      val controller = new TestController
      val result: Future[Result] = controller.getUser(1L).apply(FakeRequest())
      val userJson: JsValue = contentAsJson(result)

      userJson should be equalTo(Json.toJson(defaultUser))
    }
  }
}
```

`FakeRequest` 是一个在测试中生成伪造 HTTP 请求的辅助工具。

在这里，我们模拟 `UserRepo` 并使用这个模拟生成 `TestController` 的新实例。ScalaTest 通过其 `MockitoSugar` 特质提供了与 Mockito 的集成，因此模拟代码将有一些小的变化。

使用 ScalaTest，`UserControllerTest` 测试将如下所示：

```java
class UserControllerTest extends PlaySpec with Results with MockitoSugar {

  "UserController#getUser" should {
    "be valid" in {
      val userRepository = mock[UserRepo]
      val defaultUser = User(Some(1), "loginId", Some("name"), Some("contact_no"), Some(20L), Some("address"))
      when(userRepository.find(1)) thenReturn Option(defaultUser)

      class TestController extends Controller with BaseUserController{
        val userRepo = userRepository
      }

      val controller = new TestController
      val result: Future[Result] = controller.getUser(1L).apply(FakeRequest())

      val userJson: JsValue = contentAsJson(result)
      userJson mustBe Json.toJson(defaultUser)
    }
  }
}
```

## 使用依赖注入

我们可以使我们的控制器依赖于特定的服务，并且所有这些都可以通过使用依赖注入库通过全局对象的 `getControllerInstance` 方法进行配置。

在这个例子中，我们通过将其添加为项目依赖项来使用 **Guice**：

```java
val appDependencies = Seq(
    ...
    "com.google.inject" % "guice" % "3.0",
    "javax.inject" % "javax.inject" % "1"
  )
```

现在，让我们更新 `Global` 对象中的 `getControllerInstance` 方法：

```java
object Global extends GlobalSettings {

  val injector = Guice.createInjector(new AbstractModule {
    protected def configure() {
      bind(classOf[UserRepo]).to(classOf[AnormUserRepo])
    }
  })

  override def getControllerInstanceA: A = injector.getInstance(controllerClass)
}
```

我们现在将 `UserController` 定义为一个单例，它扩展了 `play.api.mvc.Controller` 并使用 `UserRepo`，该库是通过依赖注入实现的：

```java
@Singleton
class UserController @Inject()(userRepo: UserRepo) extends Controller {

  implicit val userWrites = Json.writes[User]

  /* GET a specific user's details */
  def getUser(userId: Long) = Action {
    val u = userRepo.find(userId)
    if (u.isEmpty) {
      NoContent
    }
    else {
      Ok(Json.toJson(u))
    }
  }

}
```

我们还需要修改路由文件：

```java
GET        /api/user/:userId        @controllers.UserController.getUser(userId:Long)

```

方法调用开头处的 `@` 符号表示应该使用全局对象的 `getControllerInstance` 方法。

### 注意

如果我们不添加方法名称的 `@` 后缀，它将搜索具有 `UserController` 名称的对象，并在编译期间抛出错误：

```java
object UserController is not a member of package controllers
[error] Note: class UserController exists, but it has no companion object.
[error] GET        /api/user/:userId        controllers.UserController.getUser(userId:Long)

```

最后，我们可以使用 Specs2 编写单元测试，如下所示：

```java
class UserControllerSpec extends Specification with Mockito {

  "UserController#getUser" should {
    "be valid" in {
      val userRepository = mock[AnormUserRepo]
      val defaultUser = User(Some(1), "loginId", Some("name"), Some("contact_no"), Some(20L), Some("address"))
      userRepository.find(1) returns Option(defaultUser)

      val controller = new UserController(userRepository)
      val result: Future[Result] = controller.getUser(1L).apply(FakeRequest())
      val userJson: JsValue = contentAsJson(result)

      userJson should be equalTo(Json.toJson(defaultUser))
    }
  }
}
```

在这里，我们模拟 `AnormUserRepo` 并使用这个模拟生成 `UserController` 的新实例。

使用 ScalaTest 的相同测试如下：

```java
class UserControllerTest extends PlaySpec with Results with MockitoSugar {

  "UserController#getUser" should {
    "be valid" in {
      val userRepository = mock[AnormUserRepo]
      val defaultUser = User(Some(1), "loginId", Some("name"), Some("contact_no"), Some(20L), Some("address"))
      when(userRepository.find(1)) thenReturn Option(defaultUser)

      val controller = new UserController(userRepository)
      val result: Future[Result] = controller.getUser(1L).apply(FakeRequest())
      val userJson: JsValue = contentAsJson(result)

      userJson mustBe Json.toJson(defaultUser)
    }
  }
}
```

以下表格总结了这两种方法的关键区别，以便更容易决定哪一种最适合您的需求：

| 使用特质进行控制器 | 使用依赖注入 |
| --- | --- |
| 需要在特质中定义和声明所有要由控制器支持的的方法。 | 需要将控制器定义为单例类，并为全局对象的`getControllerInstance`方法提供实现。 |
| 不需要额外的库。 | 需要使用依赖注入库，并提供在不同应用模式中插入不同类的灵活性。 |
| 需要定义一个额外的类来扩展测试的控制器特质。 | 不需要定义任何额外的类来测试控制器，因为可以从单例中实例化一个新的实例。 |

更多关于依赖注入的示例，请参阅[`www.playframework.com/documentation/2.3.x/ScalaDependencyInjection`](https://www.playframework.com/documentation/2.3.x/ScalaDependencyInjection)。

# 功能测试

让我们看看 Play 的一些测试用例，看看如何使用辅助方法。例如，考虑定义如下`DevErrorPageSpec`测试：

```java
object DevErrorPageSpec extends PlaySpecification{

  "devError.scala.html" should {

    val testExceptionSource = new play.api.PlayException.ExceptionSource("test", "making sure the link shows up") {
      ...
    }
    ….
    "show prod error page in prod mode" in {
      val fakeApplication = new FakeApplication() {
        override val mode = play.api.Mode.Prod
      }
      running(fakeApplication) {
        val result = DefaultGlobal.onError(FakeRequest(), testExceptionSource)
        Helpers.contentAsString(result) must contain("Oops, an error occurred")
      }
    }
  }
}
```

此测试以生产模式启动`FakeApplication`，并检查当`FakeRequest`遇到异常时的响应。

`FakeApplication`扩展了应用程序，并定义如下：

```java
case class FakeApplication(config: Map[String, Any] = Map(),
                           path: File = new File("."),
                           sources: Option[SourceMapper] = None,
                           mode: Mode.Mode = Mode.Test,
                           global: GlobalSettings = DefaultGlobal,
                           plugins: Seq[Plugin] = Nil) extends Application {
  val classloader = Thread.currentThread.getContextClassLoader
  lazy val configuration = Configuration.from(config)
}
```

正在运行的方法是 PlayRunners 的一部分，在给定应用程序的上下文中执行代码块。它定义如下：

```java
def runningT(block: => T): T = {
    synchronized {
      try {
        Play.start(app)
        block
      } finally {
        Play.stop()
      }
    }
  }
```

PlayRunners 有更多关于如何运行的定义，如下所示：

+   `runningT(block: => T): T`：这可以用来在一个运行的服务器中执行一段代码块。

+   `runningT(block: TestBrowser => T): T`：这可以用来在一个运行的服务器中执行一段代码块，并使用测试浏览器。

+   `runningT, WEBDRIVER <: WebDriver(block: TestBrowser => T): T`：这也可以用来在一个运行的服务器中使用 Selenium WebDriver 执行带有测试浏览器的代码块。此方法内部使用之前的方法。

代替直接使用`running`方法，我们可以定义测试使用包装类，这些类利用了`running`。对于 Specs2 和 ScalaTest 有不同的辅助类。

## 使用 Specs2

首先，让我们看看使用 Specs2 时可用的一些。它们如下所示：

+   `WithApplication`：它用于在运行的应用程序上下文中执行测试。例如，考虑我们想要为`CountController`编写功能测试的情况，该控制器负责按视角分组获取不同数据的计数。我们可以编写如下测试：

    ```java
    class CountControllerSpec extends PlaySpecification with BeforeExample {

      override def before: Any = {
        TestHelper.clearDB
      }

      """Counter query""" should {
        """fetch count of visits grouped by browser names""" in new WithApplication {
          TestHelper.postSampleData

          val queryString = """applicationId=39&perspective=browser&from=1389949200000&till=1399145400000""".stripMargin

          val request = FakeRequest(GET, "/query/count?" + queryString)
          val response = route(request)
          val result = response.get
          status(result) must equalTo(OK)
          contentAsJson(result) must equalTo(TestHelper.browserCount)
        }
      }
    ```

    在这里，假设 `TestHelper` 是一个专门为简化测试用例代码（将常见过程作为方法提取）而定义的辅助对象。

    如果我们需要指定 `FakeApplication`，我们可以通过将其作为参数传递给 `WithApplication` 构造函数来实现：

    ```java
     val app = FakeApplication()
        """fetch count of visits grouped by browser names""" in new WithApplication(app) {
    ```

    当我们想要为测试更改默认应用程序配置、GlobalSettings 等，这会很有用。

+   `WithServer`: 它用于在新的 `TestServer` 上执行运行中的应用程序上下文中的测试。当我们需要在特定端口上启动新的 `FakeApplication` 时，这非常有用。在稍微修改之前的示例后：

    ```java
        """fetch count of visits grouped by browser names""" in new WithServer(app = app, port = testPort) {
     {
          ...
     }
    ```

+   `WithBrowser`: 它通过在浏览器中执行某些操作来测试应用程序的功能。例如，考虑一个模拟应用程序，其中按钮点击时页面标题会改变。我们可以这样测试它：

    ```java
    class AppSpec extends PlaySpecification {
      val app: FakeApplication =
        FakeApplication(
          withRoutes = TestRoute
        )

        "run in firefox" in new WithBrowser(webDriver = WebDriverFactory(FIREFOX), app = app) {
         browser.goTo("/testing")
         browser.$("#title").getTexts().get(0) must equalTo("Test Page")

         browser.$("b").click()

         browser.$("#title").getTexts().get(0) must equalTo("testing")
        }}
    ```

    我们假设 `TestRoute` 是一个部分函数，它映射到一些路由，然后可以在测试中使用。

## 使用 ScalaTest

现在，让我们看看 **ScalaTestPlus-Play**，这个库提供了用于通过 ScalaTest 进行测试的辅助方法，它有什么可以提供的。在本节中，我们将根据适用性展示 `ScalatestPlus-Play` 的示例。ScalaTest 的辅助方法如下：

+   `OneAppPerSuite`: 在套件中运行任何测试之前，它使用 `Play.start` 启动 `FakeApplication`，然后在测试完成后停止它。应用程序通过变量 `app` 公开，如果需要可以重写。从 `ExampleSpec.scala`：

    ```java
    class ExampleSpec extends PlaySpec with OneAppPerSuite {

      // Override app if you need a FakeApplication with other than non-default parameters.
      implicit override lazy val app: FakeApplication =
        FakeApplication(additionalConfiguration = Map("ehcacheplugin" -> "disabled"))

      "The OneAppPerSuite trait" must {
        "provide a FakeApplication" in {
          app.configuration.getString("ehcacheplugin") mustBe Some("disabled")
        }
        "make the FakeApplication available implicitly" in {
          def getConfig(key: String)(implicit app: Application) = app.configuration.getString(key)
          getConfig("ehcacheplugin") mustBe Some("disabled")
        }
        "start the FakeApplication" in {
          Play.maybeApplication mustBe Some(app)
        }
      }
    }
    ```

    如果我们希望为所有或多个套件使用相同的应用程序，我们可以定义一个嵌套套件。对于此类示例，我们可以参考库中的 `NestedExampleSpec.scala`。

+   `OneAppPerTest`: 它为套件中定义的每个测试启动一个新的 `FakeApplication`。应用程序通过 `newAppForTest` 方法公开，如果需要可以重写。例如，考虑 `OneAppTest` 测试，其中每个测试都使用通过 `newAppForTest` 获取的不同 `FakeApplication`：

    ```java
    class DiffAppTest extends UnitSpec with OneAppPerTest {

      private val colors = Seq("red", "blue", "yellow")

      private var colorCode = 0

      override def newAppForTest(testData: TestData): FakeApplication = {
        val currentCode = colorCode
        colorCode+=1
        FakeApplication(additionalConfiguration = Map("foo" -> "bar",
          "ehcacheplugin" -> "disabled",
          "color" -> colors(currentCode)
        ))
      }

      def getConfig(key: String)(implicit app: Application) = app.configuration.getString(key)

      "The OneAppPerTest trait" must {
        "provide a FakeApplication" in {
          app.configuration.getString("color") mustBe Some("red")
        }
        "make another FakeApplication available implicitly" in {
          getConfig("color") mustBe Some("blue")
        }
        "make the third FakeApplication available implicitly" in {
          getConfig("color") mustBe Some("yellow")
        }
      }
    }
    ```

+   `OneServerPerSuite`: 它为套件启动一个新的 `FakeApplication` 和一个新的 `TestServer`。应用程序通过变量 `app` 公开，如果需要可以重写。服务器端口由变量 `port` 设置，如果需要可以更改/修改。这已在 `OneServerPerSuite` 的示例（`ExampleSpec2.scala`）中演示：

    ```java
    class ExampleSpec extends PlaySpec with OneServerPerSuite {

      // Override app if you need a FakeApplication with other than non-default parameters.
      implicit override lazy val app: FakeApplication =
        FakeApplication(additionalConfiguration = Map("ehcacheplugin" -> "disabled"))

      "The OneServerPerSuite trait" must {
        "provide a FakeApplication" in {
          app.configuration.getString("ehcacheplugin") mustBe Some("disabled")
        }
        "make the FakeApplication available implicitly" in {
          def getConfig(key: String)(implicit app: Application) = app.configuration.getString(key)
          getConfig("ehcacheplugin") mustBe Some("disabled")
        }
        "start the FakeApplication" in {
          Play.maybeApplication mustBe Some(app)
        }
        "provide the port number" in {
          port mustBe Helpers.testServerPort
        }
        "provide an actual running server" in {
          import java.net._
          val url = new URL("http://localhost:" + port + "/boum")
          val con = url.openConnection().asInstanceOf[HttpURLConnection]
          try con.getResponseCode mustBe 404
          finally con.disconnect()
        }
      }
    }
    ```

    当我们需要多个套件使用相同的 FakeApplication 和 TestServer 时，我们可以定义类似于 `NestedExampleSpec2.scala` 的嵌套套件测试。

+   `OneServerPerTest`: 它为套件中定义的每个测试启动一个新的 `FakeApplication` 和 `TestServer`。应用程序通过 `newAppForTest` 方法公开，如果需要可以重写。例如，考虑 `DiffServerTest` 测试，其中每个测试都使用通过 `newAppForTest` 获取的不同 `FakeApplication`，并且 `TestServer` 端口被重写：

    ```java
    class DiffServerTest extends PlaySpec with OneServerPerTest {

      private val colors = Seq("red", "blue", "yellow")

      private var code = 0

      override def newAppForTest(testData: TestData): FakeApplication = {
        val currentCode = code
        code += 1
        FakeApplication(additionalConfiguration = Map("foo" -> "bar",
          "ehcacheplugin" -> "disabled",
          "color" -> colors(currentCode)
        ))
      }

      override lazy val port = 1234

      def getConfig(key: String)(implicit app: Application) = app.configuration.getString(key)

      "The OneServerPerTest trait" must {
        "provide a FakeApplication" in {
          app.configuration.getString("color") mustBe Some("red")
        }
        "make another FakeApplication available implicitly" in {
          getConfig("color") mustBe Some("blue")
        }
        "start server at specified port" in {
          port mustBe 1234
        }
      }
    }
    ```

+   `OneBrowserPerSuite`: 它为每个测试套件提供一个新的 Selenium WebDriver 实例。例如，假设我们希望通过在 Firefox 中打开应用程序来测试按钮的点击，测试可以像 `ExampleSpec3.scala` 一样编写：

    ```java
    @FirefoxBrowser
    class ExampleSpec extends PlaySpec with OneServerPerSuite with OneBrowserPerSuite with FirefoxFactory {

      // Override app if you need a FakeApplication with other than non-default parameters.
      implicit override lazy val app: FakeApplication =
        FakeApplication(
          additionalConfiguration = Map("ehcacheplugin" -> "disabled"),
          withRoutes = TestRoute
        )

      "The OneBrowserPerSuite trait" must {
        "provide a FakeApplication" in {
          app.configuration.getString("ehcacheplugin") mustBe Some("disabled")
        }
        "make the FakeApplication available implicitly" in {
          def getConfig(key: String)(implicit app: Application) = app.configuration.getString(key)
          getConfig("ehcacheplugin") mustBe Some("disabled")
        }
        "provide a web driver" in {
          go to ("http://localhost:" + port + "/testing")
          pageTitle mustBe "Test Page"
          click on find(name("b")).value
          eventually { pageTitle mustBe "scalatest" }
        }
      }
    }
    ```

    我们假设 `TestRoute` 是一个部分函数，它映射到一些路由，然后可以在测试中使用。

    同样的特质可以用来在多个浏览器中测试应用程序，如 `MultiBrowserExampleSpec.scala` 中所示。要执行所有浏览器的测试，我们应该使用 `AllBrowsersPerSuite`，如下所示：

    ```java
    class AllBrowsersPerSuiteTest extends PlaySpec with OneServerPerSuite with AllBrowsersPerSuite {

      // Override newAppForTest if you need a FakeApplication with other than non-default parameters.
      override lazy val app: FakeApplication =
        FakeApplication(
          withRoutes = TestRoute
        )

      // Place tests you want run in different browsers in the `sharedTests` method:
      def sharedTests(browser: BrowserInfo) = {

          "navigate to testing "+browser.name in {
            go to ("http://localhost:" + port + "/testing")
            pageTitle mustBe "Test Page"
            click on find(name("b")).value
            eventually { pageTitle mustBe "testing" }
          }

          "navigate to hello in a new window"+browser.name in {
            go to ("http://localhost:" + port + "/hello")
            pageTitle mustBe "Hello"
            click on find(name("b")).value
            eventually { pageTitle mustBe "helloUser" }
          }
      }

      // Place tests you want run just once outside the `sharedTests` method
      // in the constructor, the usual place for tests in a `PlaySpec`

      "The test" must {
        "start the FakeApplication" in {
          Play.maybeApplication mustBe Some(app)
        }
      }
    ```

    `OneBrowserPerSuite` 特质也可以与嵌套测试一起使用。请参阅 `NestedExampleSpec3.scala`。

+   `OneBrowserPerTest`: 它为套件中的每个测试启动一个新的浏览器会话。这可以通过运行 `ExampleSpec4.scala` 测试来注意到。它与 `ExampleSpec3.scala` 类似，但 `OneServerPerSuite` 和 `OneBrowserPerSuite` 分别被替换为 `OneServerPerTest` 和 `OneBrowserPerTest`，如下所示：

    ```java
    @FirefoxBrowser
    class ExampleSpec extends PlaySpec with OneServerPerTest with OneBrowserPerTest with FirefoxFactory {
      ...
    }
    ```

    我们还用 `newAppForTest` 重写方法替换了重写的 `app` 变量。尝试编写一个使用 `AllBrowsersPerTest` 特质的测试。

### 提示

当在定义自定义演员的应用程序上同时运行多个功能测试时，可能会遇到 InvalidActorNameException。我们可以通过定义一个嵌套测试来避免这种情况，其中多个测试使用相同的 `FakeApplication`。

# 摘要

在本章中，我们看到了如何使用 Specs2 或 ScalaTest 测试 Play 应用程序。我们还遇到了简化 Play 应用程序测试的不同辅助方法。在单元测试部分，我们讨论了在设计模型和控制器时可以采取的不同方法，这些方法基于首选的测试过程，使用具有定义方法的特质或依赖注入。我们还讨论了在具有测试服务器和浏览器使用 Selenium WebDriver 的上下文中对 Play 应用程序的功能测试。

在下一章中，我们将讨论 Play 应用程序的调试和日志记录。
