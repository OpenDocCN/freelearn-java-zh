# 第十一章。网络服务和身份验证

互联网浩瀚且不断扩展。许多日常任务都可以以更简单的方式进行——账单支付、检查产品评论、预订电影票等。除此之外，大多数电子设备现在都可以连接到互联网，例如手机、手表、监控系统和安全系统。这些设备可以相互通信，而且它们不必都是同一品牌。应用程序可以利用用户特定的信息并提供更定制化的功能。最重要的是，我们可以通过验证来决定是否愿意与应用程序共享我们的信息。

在本章中，我们将介绍 Play 框架对以下内容的支持：

+   调用网络服务

+   OpenID 和 OAuth 身份验证

# 调用网络服务

假设我们需要在线预订机票。我们可以通过使用飞行品牌的网站（如汉莎航空、阿联酋航空等）或旅行预订网站（如 ClearTrip、MakeMyTrip 等）来完成这项任务。我们如何从两个或更多不同的网站完成相同的任务呢？

飞行品牌的网站提供了一些 API，这些 API 是旅行预订网站工作的。这些 API 可以是免费提供的，也可以通过合同收费，由提供者和其他第三方决定。这些 API 也被称为网络服务。

网络服务基本上是一种通过互联网调用的方法。只有提供者完全了解这些网站的内部运作。使用网络服务的人只知道其目的和可能的后果。

许多应用程序出于各种原因需要/更喜欢使用第三方 API 来完成常见任务，例如业务领域的通用规范、提供安全授权的更简单方式，或避免维护开销等。

Play 框架有一个专门满足此类需求的网络服务 API。可以通过将其作为依赖项包含来使用网络服务 API：

```java
libraryDependencies ++= Seq(
  ws
)
```

一个常见的用例是使用事务性电子邮件 API 服务（如 Mailgun、SendGrid 等）发送带有账户验证和/或重置密码链接的电子邮件。

假设我们的应用程序有这样的需求，并且我们有一个处理所有这些交易的`Email`对象。我们需要一个发送电子邮件的方法，它实际上会调用电子邮件 API 服务，然后是其他内部调用发送的方法。使用 Play 网络服务 API，我们可以将`Email`定义为：

```java
object Email {

  val logger = Logger(getClass)

  private def send(emailIds: Seq[String], subject: String, content: String): Unit = {

    var properties: Properties = new Properties()

    try {

      properties.load(new FileInputStream("/opt/appName/mail-config.properties"))

      val url: String = properties.getProperty("url")

      val apiKey: String = properties.getProperty("api")

      val from: String = properties.getProperty("from")

      val requestHolder: WSRequestHolder = WS.url(url).withAuth("api", apiKey, WSAuthScheme.BASIC)
      val requestData = Map(

        "from" -> Seq(from),

        "to" -> emailIds,

        "subject" -> Seq(subject),

        "text" -> Seq(content))

      val response: Future[WSResponse] = requestHolder.post(requestData)

      response.map(

        res => {

          val responseMsg: String = res.json.toString()

          if (res.status == 200) {

            logger.info(responseMsg)

          } else {

            logger.error(responseMsg)

          }

        }

      )

    } catch {

      case exp: IOException =>

        logger.error("Failed to load email configuration properties.")

    }

  }

  def sendVerification(userId: Long, emailId: String, host: String): Unit = {

    val subject: String = "Email Verification"

    val content: String =

      s"""To verify your account on <appName>, please click on the link below

         |

         |http://$host/validate/user/$userId""".stripMargin

    send(Seq(emailId), subject, content)

  }

  def recoverPassword(emailId: String, password: String): Unit = {

    val subject: String = "Password Recovery"

    val emailContent: String = s"Your password has been reset.The new password is $password"

    send(Seq(emailId), subject, emailContent)

  }

}
```

网络服务 API 通过`WS`对象公开，该对象提供了作为 HTTP 客户端查询网络服务的方法。在前面的代码片段中，我们使用了网络服务 API 来发起一个 POST 请求。其他可用的方法来触发请求并获取响应或响应流包括：

+   `get`或`getStream`

+   `put`或`putAndRetrieveStream`

+   `post`或`postAndRetrieveStream`

+   `delete`

+   `head`

这些调用中的任何结果都是 `Future[WSResponse]` 类型，因此我们可以安全地说，网络服务 API 是异步的。

它不仅限于 REST 服务。例如，假设我们使用 SOAP 服务来获取所有国家的货币：

```java
  def displayCurrency = Action.async {

    val url: String = "http://www.webservicex.net/country.asmx"

    val wsReq: String = """<?xml version="1.0" encoding="utf-8"?>

                          |<soap12:Envelope   >

                          |  <soap12:Body>

                          |    <GetCurrencies  />

                          |  </soap12:Body>

                          |</soap12:Envelope>""".stripMargin

    val response: Future[WSResponse] = WS.url(url).withHeaders("Content-Type" -> "application/soap+xml").post(wsReq)

    response map {

      data => Ok(data.xml)

    }

  }
```

可以使用 `WS.url()` 构建一个 HTTP 请求，它返回一个 `WSRequestHolder` 实例。`WSRequestHolder` 特性有添加头部、身份验证、请求参数、数据等方法。以下是一个常用方法的另一个示例：

```java
WS.url("http://third-party.com/service?=serviceName")
.withAuth("api","apiKey", WSAuthScheme.BASIC)
.withQueryString("month" -> "12",
        "year" -> "2014",
        "code" -> "code")
.withHeaders(HeaderNames.ACCEPT -> MimeTypes.JSON)
.get
```

尽管在这个例子中我们使用了基本身份验证，但网络服务 API 支持大多数常用的身份验证方案，您可以在以下链接中找到：

+   **基本**: [`en.wikipedia.org/wiki/Basic_access_authentication`](http://en.wikipedia.org/wiki/Basic_access_authentication)

+   **摘要**: [`en.wikipedia.org/wiki/Digest_access_authentication`](http://en.wikipedia.org/wiki/Digest_access_authentication)

+   **简单且受保护的 GSSAPI 协商机制 (SPNEGO)**: [`en.wikipedia.org/wiki/SPNEGO`](http://en.wikipedia.org/wiki/SPNEGO)

+   **NT LAN 管理器 (NTLM)**: [`en.wikipedia.org/wiki/NT_LAN_Manager`](http://en.wikipedia.org/wiki/NT_LAN_Manager)

+   **Kerberos**: [`en.wikipedia.org/wiki/Kerberos_(protocol)`](http://en.wikipedia.org/wiki/Kerberos_(protocol))

通过 `WS` 对象可用的所有方法只是调用可用的 `WSAPI` 特性的实现中的相关方法。默认提供的网络服务 API 使用 Ning 的 AysncHttpClient（请参阅 [`github.com/AsyncHttpClient/async-http-client`](https://github.com/AsyncHttpClient/async-http-client)）。如果我们想使用任何其他 HTTP 客户端，我们需要实现 `WSAPI` 特性并通过插件绑定它。当我们添加 `ws` Play 库时，它将 `play.api.libs.ws.ning.NingWSPlugin` 添加到我们的应用程序中，该插件定义为：

```java
class NingWSPlugin(app: Application) extends WSPlugin {

  @volatile var loaded = false

  override lazy val enabled = true

  private val config = new DefaultWSConfigParser(app.configuration, app.classloader).parse()

  private lazy val ningAPI = new NingWSAPI(app, config)

  override def onStart() {
    loaded = true
  }

  override def onStop() {
    if (loaded) {
      ningAPI.resetClient()
      loaded = false
    }
  }

  def api = ningAPI

}
```

### 注意

在 Play 应用中，使用 SSL 与 WS 需要对配置进行一些更改，并在 [`www.playframework.com/documentation/2.3.x/WsSSL`](https://www.playframework.com/documentation/2.3.x/WsSSL) 中有文档说明。

由于大量应用程序依赖于来自各种来源的用户数据，Play 提供了 OpenID 和 OAuth 的 API。我们将在以下章节中讨论这些内容。

# OpenID

OpenID 是一种身份验证协议，其中 OpenID 提供商验证第三方应用程序中用户的身份。OpenID 提供商是任何向用户提供 OpenID 的服务/应用程序。Yahoo、AOL 等是这些服务/应用程序的几个例子。需要用户 OpenID 来完成交易的应用程序被称为 OpenID 消费者。

OpenID 消费者中的控制流程如下：

1.  用户将被引导到支持的/选定的 OpenID 提供商的登录页面。

1.  一旦用户完成登录，OpenID 提供商会告知用户 OpenID 消费者请求的用户相关数据。

1.  如果用户同意共享信息，则会被重定向到消费者应用程序中请求的页面。信息被添加到请求 URL 中。这些信息被称为属性属性，相关文档位于 [`openid.net/specs/openid-attribute-properties-list-1_0-01.html`](http://openid.net/specs/openid-attribute-properties-list-1_0-01.html)。

Play 提供了一个 API 来简化 OpenID 交易，相关文档位于 [`www.playframework.com/documentation/2.3.x/api/scala/index.html#play.api.libs.openid.OpenID$`](https://www.playframework.com/documentation/2.3.x/api/scala/index.html#play.api.libs.openid.OpenID$)。

以下两个关键方法如下：

+   `redirectURL`：此参数用于验证用户、请求特定用户信息并将其重定向到回调页面

+   `verifiedId`：此参数用于从已验证的 OpenID 回调请求中提取用户信息

让我们构建一个使用提供者 Yahoo 的 OpenID 的应用程序。我们可以定义控制器如下：

```java
object Application extends Controller {

  def index = Action.async {

    implicit request =>

      OpenID.verifiedId.map(info => Ok(views.html.main(info.attributes)))

        .recover {

        case t: Throwable =>

          Redirect(routes.Application.login())

      }

  }

  def login = Action.async {

    implicit request =>

      val openIdRequestURL: String = "https://me.yahoo.com"

      OpenID.redirectURL(

        openIdRequestURL,

        routes.Application.index.absoluteURL(),

        Seq("email" -> "http://schema.openid.net/contact/email",

          "name" -> "http://openid.net/schema/namePerson/first"))

        .map(url => Redirect(url))

        .recover { case t: Throwable => Ok(t.getMessage) }

  }

}
```

在前面的代码片段中，`login` 方法将用户重定向到 Yahoo 登录页面（参考 [`me.yahoo.com`](https://me.yahoo.com)）。一旦用户登录，系统会询问用户是否允许应用程序共享其个人资料。如果用户同意，则会重定向到 `routes.Application.index.absoluteURL()`。

`index` 方法期望在登录成功时由 OpenID 提供商（在我们的例子中是 Yahoo）共享的数据。如果数据不可用，用户将被重定向到 `login` 方法。

`OpenID.redirectURL` 的第三个参数是一个元组序列，它指示应用程序所需的信息（所需属性）。每个元组的第二个元素是使用 OpenID 属性交换请求的属性属性的标签——它使得个人身份信息的传输成为可能。每个元组的第一个元素是 OpenID 提供者在回调请求的 `queryString` 中映射属性属性值的标签。

例如，`http://openid.net/schema/namePerson/first` 属性通过其名字表示属性属性。在登录成功后，此属性的值和消费者提供的标签会被添加到回调中的 `queryString`。因此，`openid.ext1.value.name=firstName` 会被添加到登录回调中。

# OAuth

根据 [`oauth.net/core/1.0/`](http://oauth.net/core/1.0/)，OAuth 的定义如下：

> “OAuth 认证是用户在不与消费者共享其凭据的情况下授予其受保护资源访问权限的过程。OAuth 使用服务提供商生成的令牌，而不是在受保护资源请求中使用用户的凭据。此过程使用两种令牌类型：”
> 
> *请求令牌：由消费者用于请求用户授权访问受保护资源。用户授权的请求令牌被交换为访问令牌，必须只使用一次，并且不得用于其他任何目的。建议请求令牌有有限的有效期。*
> 
> *访问令牌：由消费者代表用户访问受保护资源。访问令牌可以限制对某些受保护资源的访问，并且可能有有限的有效期。服务提供商应允许用户撤销访问令牌。仅访问令牌应用于访问受保护资源。*
> 
> *OAuth 认证分为三个步骤：*
> 
> *消费者获取一个未经授权的请求令牌。*
> 
> *用户授权请求令牌。*
> 
> *消费者将请求令牌交换为访问令牌。"*

具体可以访问什么以及可以访问多少由服务提供商决定。

OAuth 有三个版本：1.0、1.0a 和 2.0。第一个版本（1.0）存在一些安全问题，并且服务提供商不再使用。

Play 提供了用于 1.0 和 1.0a 版本的 API，而不是用于 2.0，因为使用这个版本要简单得多。API 文档位于[`www.playframework.com/documentation/2.3.x/api/scala/index.html#play.api.libs.oauth.package`](https://www.playframework.com/documentation/2.3.x/api/scala/index.html#play.api.libs.oauth.package)。

让我们构建一个应用程序，该程序利用 Twitter 账户通过 Play 的 OAuth API 进行登录。

初始时，我们需要使用 Twitter 账户在[`apps.twitter.com/`](https://apps.twitter.com/)注册应用程序，以便我们有一个有效的消费者密钥和密钥组合。之后，我们可以定义如下动作：

```java
val KEY: ConsumerKey = ConsumerKey("myAppKey", "myAppSecret")
val TWITTER: OAuth = OAuth(ServiceInfo(
    "https://api.twitter.com/oauth/request_token",
    "https://api.twitter.com/oauth/access_token",
    "https://api.twitter.com/oauth/authorize", KEY),
    true)

def authenticate = Action { request =>
    TWITTER.retrieveRequestToken("http://localhost:9000/welcome") match {
      case Right(t) => {
        Redirect(TWITTER.redirectUrl(t.token)).withSession("token" -> t.token, "secret" -> t.secret)
      }
      case Left(e) => throw e
    }
  }
```

OAuth 是 Play 的一个辅助类，具有以下签名：

```java
OAuth(info: ServiceInfo, use10a: Boolean = true)
```

参数确定 OpenID 的版本。如果设置为`true`，则使用 OpenID 1.0，否则使用 1.0。

它提供了这三个方法：

+   `redirectURL`: 这将获取用户应重定向到的 URL 字符串，以便通过提供者授权应用程序

+   `retrieveRequestToken`: 这从提供者那里获取请求令牌

+   `retrieveAccessToken`: 这将请求令牌交换为访问令牌

在前面的动作定义中，我们只使用提供者进行登录；除非我们用访问令牌交换授权的请求令牌，否则我们无法获取任何用户详情。要获取访问令牌，我们需要请求令牌和`oauth_verifier`，这是服务提供商在授予请求令牌时提供的。

使用 Play OAuth API，在获取请求令牌后进行重定向会将`oauth_verifier`添加到请求查询字符串中。因此，我们应该重定向到一个尝试获取访问令牌并随后存储它的动作，以便它对未来的请求易于访问。在这个例子中，它被存储在会话中：

```java
def authenticate = Action { request =>
    request.getQueryString("oauth_verifier").map { verifier =>
      val tokenPair = sessionTokenPair(request).get
      TWITTER.retrieveAccessToken(tokenPair, verifier) match {
        case Right(t) => {
          Redirect(routes.Application.welcome()).withSession("token" -> t.token, "secret" -> t.secret)
        }
        case Left(e) => throw e
      }
    }.getOrElse(
        TWITTER.retrieveRequestToken("http://localhost:9000/twitterLogin") match {
      case Right(rt) =>
        Redirect(TWITTER.redirectUrl(rt.token)).withSession("token" -> rt.token, "secret" -> rt.secret)
      case Left(e) => throw e
    })
  }

  private def sessionTokenPair(implicit request: RequestHeader): Option[RequestToken] = {
    for {
      token <- request.session.get("token")
      secret <- request.session.get("secret")
    } yield {
      RequestToken(token, secret)
    }
  }

  def welcome = Action.async {
    implicit request =>
      sessionTokenPair match {
        case Some(credentials) => {
          WS.url("https://api.twitter.com/1.1/statuses/home_timeline.json")
            .sign(OAuthCalculator(KEY, credentials))
            .get()
            .map(result => Ok(result.json))
        }
        case _ => Future.successful(Redirect(routes.Application.authenticate()).withNewSession)
      }

  }
```

在用户成功登录和授权后，我们通过 welcome 动作获取用户时间轴上的状态并将其作为 JSON 显示。

### 注意

Play 中没有内置对使用 OAuth 2.0、CAS、SAML 或任何其他协议进行身份验证的支持。然而，开发者可以选择使用符合他们要求的第三方插件或库。其中一些包括 Silhouette ([`silhouette.mohiva.com/v2.0`](http://silhouette.mohiva.com/v2.0))、deadbolt-2 ([`github.com/schaloner/deadbolt-2`](https://github.com/schaloner/deadbolt-2))、play-pac4j ([`github.com/pac4j/play-pac4j`](https://github.com/pac4j/play-pac4j)))等等。

# 摘要

在本章中，我们学习了 WS（Web 服务）插件以及通过它暴露的 API。我们还看到了如何使用 OpenID 和 OAuth 1.0a（因为大多数服务提供商使用 1.0a 或 2.0）从服务提供商访问用户数据，借助 Play 中的 OpenID 和 OAuth API。

在下一章中，我们将了解 Play 提供的某些模块是如何工作的，以及我们如何使用它们来构建自定义模块。
