# 第二章。定义行动

如果你正在阅读这篇文章，那么你可能已经度过了第一章，或者跳过了它。无论如何，我假设你知道一个简单 Play 应用程序的结构。在 Play 中，控制器生成 Action 值，为此，它内部使用几个对象和方法。在本章中，我们将了解幕后发生了什么，以及我们如何在构建应用程序时利用这些行动。

在本章中，我们将涵盖以下主题：

+   定义行动

+   请求体解析器

+   行动组合和故障排除

# 一个虚拟的艺术家模型

在以下章节中，我们将引用一个`artist`模型。它是一个简单的`class`，有一个伴随的`object`，如下定义：

```java
case class Artist(name: String, country: String)

object Artist {
  val availableArtist = Seq(Artist("Wolfgang Amadeus Mozart", "Austria"), 
    Artist("Ludwig van Beethoven", "Germany"), 
    Artist("Johann Sebastian Bach", "Germany"), 
    Artist("Frédéric François Chopin", "Poland"), 
    Artist("Joseph Haydn", "Austria"), 
    Artist("Antonio Lucio Vivaldi", "Italy"), 
    Artist("Franz Peter Schubert", "Austria"), 
    Artist("Franz Liszt", "Austria"), 
    Artist("Giuseppe Fortunino Francesco Verdi", "Austria")) 

  def fetch: Seq[Artist] = {
    availableArtist 
  } 

  def fetchByName(name: String): Seq[Artist] = {
    availableArtist.filter(a => a.name.contains(name)) 
  } 

  def fetchByCountry(country: String): Seq[Artist] = {
    availableArtist.filter(a => a.country == country) 
  } 

  def fetchByNameOrCountry(name: String, country: String): Seq[Artist] = { 
    availableArtist.filter(a => a.name.contains(name) || a.country == country) 
  } 

  def fetchByNameAndCountry(name: String, country: String): Seq[Artist] = {
    availableArtist.filter(a => a.name.contains(name) && a.country == country)
  } 
} 
```

艺术家模型有一个方法来获取所有这些艺术家，以及一些基于不同参数过滤艺术家的方法。

### 注意

在实际应用程序中，模型与数据库交互，但为了使事情简单，我们已经将数据硬编码为`Seq[Artist]`。

我们还有一个`home.scala.html`视图，它以表格的形式显示艺术家信息：

```java
@(artists: Seq[Artist])
<!DOCTYPE html> 

<html>
    <head>
        <title>Action App</title>
    </head>
    <body>
        <table>
            <thead>
                <tr>
                    <th>Name</th>
                    <th>Country</th>
                    </tr>
            </thead>
            <tbody>
            @artists.map { artist => 
                <tr>
                    <td>@artist.name</td>
                    <td>@artist.country</td>
                </tr>
            } 
            </tbody>
        </table>
    </body>
</html>
```

这是一个 twirl 模板，它需要一个`Seq[Artist]`。它与我们在上一章中构建的 TaskTracker 应用程序的视图类似。

# 行动

Play 中的**Action**定义了服务器应该如何响应用户请求。定义 Action 的方法映射到`routes`文件中的请求。例如，让我们定义一个 Action，它将显示所有艺术家的信息作为响应：

```java
def listArtist = Action {
  Ok(views.html.home(Artist.fetch))
}
```

现在，为了使用这个 Action，我们应该将其映射到`routes`文件中的请求。

```java
GET     /api/artist       controllers.Application.listArtist
```

在这个例子中，我们获取所有艺术家，并将它们与视图一起发送，作为对请求的响应。

### 注意

在`route`文件中使用的`api`术语只是一个 URL 前缀，不是强制的。

运行应用程序，并从浏览器访问`http://localhost:9000/api/artist`。可以看到一个包含可用艺术家的表格。

Action 接受一个请求并产生一个结果。它是`EssentialAction`特质的实现。它被定义为：

```java
trait EssentialAction extends (RequestHeader => Iteratee[Array[Byte], Result]) with Handler {

  def apply() = this

}

object EssentialAction {
  def apply(f: RequestHeader => Iteratee[Array[Byte], Result]): EssentialAction = new EssentialAction {
    def apply(rh: RequestHeader) = f(rh)
  }
}
```

**Iteratee**是从函数式语言中借用的一个概念。它用于以增量方式处理数据块。我们将在第六章*反应式数据流*中更深入地探讨它。

`apply`方法接受一个函数，该函数将请求转换成结果。`RequestHeader`和其他数据块代表请求。简而言之，`apply`方法接受一个请求并返回一个结果。

让我们看看定义行动的一些方法。

## 带参数的 Action

我们可能会遇到需要定义一个从请求路径中获取值的 Action 的情况。在这种情况下，我们需要添加方法签名所需的参数，并在路由文件中传递它们。这个例子将是获取按所选名称检索艺术家的方法。在控制器中添加以下内容：

```java
  def fetchArtistByName(name:String) = Action { 
     Ok(views.html.home(Artist.fetchByName(name))) 
  } 
```

在`routes`文件中的映射将是：

```java
GET    /api/artist/:name       controllers.Application.fetchArtistByName(name)
```

### 注意

如果没有明确指定，请注意，路径中参数的类型默认设置为`String`。可以在方法调用中指定类型。因此，定义的路由等同于：

```java
GET    /api/artist/:name        controllers.Application.fetchArtistByName(name:String)
```

类似地，如果需要，我们可以添加更多参数。

现在，让我们考虑一个搜索查询的使用案例。我们希望操作能够接受查询参数，例如名称和国家。操作定义如下：

```java
def search(name: String, country: String) = Action { 
    val result = Artist.fetchByNameOrCountry(name, country) 
    if(result.isEmpty){
      NoContent
       } 
    else {
      Ok(views.html.home(result))
    }
  }
```

如果没有艺术家符合标准，则响应为空，并显示状态码 204（无内容）。如果不满足，则响应状态为`200 = (Ok)`，并将结果作为响应体显示。

路由文件中对应此操作的条目将是以下内容：

```java
GET    /api/search/artist        controllers.Application.search(name:String,country:String)     
```

我们在路径中不使用任何参数，但应该包含与`routes`文件中方法的参数名称对应的查询参数。

这将生成一个有效的 URL：`http://localhost:9000/api/search/artist?name=Franz&country=Austria`

如果我们决定将`country`设置为可选参数怎么办？

让我们修改路由以适应这一变化：

```java
GET    /api/search/artist    controllers.Application.search(name:String?="",country:String?="") 
```

这允许我们通过名称进行查询，因此，现在这两个 URL 将看起来像这样：`http://localhost:9000/api/search/artist?name=Franz` 和 `http://localhost:9000/api/search/artist?name=Franz&country=Austria` 都已支持。

在这里，我们通过在路由定义中为`country`参数设置默认值来使其成为可选参数。或者，我们也可以定义一个接受`Option`类型参数的操作：

```java
def search2(name: Option[String], country: String) = Action { 
    val result = name match{ 
      case Some(n) => Artist.fetchByNameOrCountry(n, country) 
      case None => Artist.fetchByCountry(country) 
    } 
    if(result.isEmpty){
      NoContent
    } 
    else { 
      Ok(views.html.home(result)) 
    } 
  }
```

然后，路由将如下所示：

```java
GET    /api/search2/artist    controllers.Application.search2(name:Option[String],country:String)
```

我们现在可以带或不带国家名称进行请求：

`http://localhost:9000/api/search2/artist?country=Austria`

`http://localhost:9000/api/search2/artist?name=Franz&country=Austria`

在本节中显示的示例中，我们不需要使用请求来生成我们的结果，但在某些情况下，我们会使用请求来生成相关结果。然而，为了做到这一点，理解请求内容的格式至关重要。我们将在下一节中看到这是如何完成的。

# 请求体解析器

考虑到任何应用程序中最常见的 POST 请求——用于登录的请求。如果请求体包含用户的凭据，比如 JSON 或 XML 格式，这将是足够的吗？请求处理器能否直接提取这些数据并处理它们？不，因为请求中的数据必须被应用程序代码理解，它必须被转换为兼容的类型。例如，发送到 Scala 应用程序的 XML 必须转换为 Scala XML。

有几个库，例如 Jackson、XStream 等，可以用来完成这个任务，但我们不需要它们，因为 Play 内部支持这个功能。Play 提供了请求体解析器，可以将请求体转换为常用内容类型的等效 Scala 对象。此外，我们还可以扩展现有解析器或定义新的解析器。

每个 Action 都有一个解析器。我如何知道这一点？嗯，我们用来定义应用应该如何响应的 Action 对象，仅仅是 Action 特质的扩展，并且定义如下：

```java
 trait Action[A] extends EssentialAction {   

  //Type of the request body. 
  type BODY_CONTENT = A 

  //Body parser associated with this action. 
  def parser: BodyParser[A] 

  //Invokes this action 
  def apply(request: Request[A]): Future[Result] 

       def apply(rh: RequestHeader): Iteratee[Array[Byte], Result] = parser(rh).mapM {
      case Left(r) => 

 Future.successful(r)
 case Right(a) => 
 val request = Request(rh, a) 

 Play.maybeApplication.map { app => 
 play.utils.Threads.withContextClassLoader(app.classloader) {
 apply(request)
         }
 }.getOrElse {
         apply(request)
 }
 }(executionContext)

  //The execution context to run this action in 
  def executionContext: ExecutionContext = play.api.libs.concurrent.Execution.defaultContext 

  //Returns itself, for better support in the routes file. 
  override def apply(): Action[A] = this 

  override def toString = { 
    "Action(parser="+ parser + ")" 
  } 

}
```

`apply`方法转换解析器返回的值。解析器返回的值可以是结果或请求体（表示为`Either[Result,A]`）。

因此，转换被定义为两种可能的结果。如果我们进行模式匹配，我们得到`Left(r)`，这是一个结果类型，以及`Right(a)`，这是请求体。

`mapM`方法与`map`方法功能相似，唯一的区别是它是异步执行的。

然而，即使没有解析器，也可以定义 Action 吗？是的，也可以不是。

让我们看看一个示例 Action：一个 POST 请求，这是订阅更新的必需请求。这个请求接受用户的电子邮件 ID 作为查询参数，这意味着我们需要访问请求体以完成此用户的订阅。首先，我们将检查在不指定解析器的情况下请求体看起来像什么。在控制器中创建一个名为`subscribe`的 Action，如下所示：

```java
def subscribe = Action { 
    request => 
        Ok("received " + request.body) 
  } 
```

现在，在路由文件中为这个添加一个条目：

```java
POST    /subscribe         controllers.AppController.subscribe 
```

然后，运行应用程序。使用 REST 客户端或 Curl（您更习惯哪个）向`http://localhost:9000/subscribe`发送带有`userId@gmail.com`电子邮件 ID 的 POST 请求。

例如：

```java
curl 'http://localhost:9000/subscribe' -H 'Content-Type: text/plain;charset=UTF-8' --data-binary 'userId@gmail.com' 
```

这个请求的响应将是以下内容：

```java
received AnyContentAsText(userId@gmail.com)
```

你注意到我们的`subscribe`方法理解了内容是文本吗？请求体被转换成了`AnyContentAsText(userId@gmail.com)`。我们的方法是如何确定这一点的？这不是映射到特定 Action 的解析器的职责吗？

当为 Action 未指定解析器时，`BodyParsers.parse.anyContent`方法返回的解析器被设置为该 Action 的解析器。这是由`ActionBuilder`处理的，我们将在本章后面看到。以下代码片段显示了在没有提供解析器时生成 Action 的一种方法：

```java
final def apply(block: R[AnyContent] => Result): Action[AnyContent] = apply(BodyParsers.parse.anyContent)(block) 
```

现在，让我们来检查一下`BodyParsers.parse.anyContent`方法的作用：

```java
def anyContent: BodyParser[ AnyContent] = BodyParser("anyContent") { request => 
      import play.api.libs.iteratee.Execution.Implicits.trampoline 
      request.contentType.map(_.toLowerCase(Locale.ENGLISH)) match { 
        case _ if request.method == HttpVerbs.GET || request.method == HttpVerbs.HEAD => { 
          Play.logger.trace("Parsing AnyContent as empty") 
          empty(request).map(_.right.map(_ => AnyContentAsEmpty)) 
        } 
        case Some("text/plain") => { 
          Play.logger.trace("Parsing AnyContent as text") 
          text(request).map(_.right.map(s => AnyContentAsText(s))) 
        } 
        case Some("text/xml") | Some("application/xml") | Some(ApplicationXmlMatcher()) => { 
          Play.logger.trace("Parsing AnyContent as xml") 
          xml(request).map(_.right.map(x => AnyContentAsXml(x))) 
         } 
        case Some("text/json") | Some("application/json") => { 
          Play.logger.trace("Parsing AnyContent as json") 
          json(request).map(_.right.map(j => AnyContentAsJson(j))) 
        } 
        case Some("application/x-www-form-urlencoded") => { 
          Play.logger.trace("Parsing AnyContent as urlFormEncoded") 
          urlFormEncoded(request).map(_.right.map(d => AnyContentAsFormUrlEncoded(d))) 
        } 
        case Some("multipart/form-data") => { 
          Play.logger.trace("Parsing AnyContent as multipartFormData") 
          multipartFormData(request).map(_.right.map(m => AnyContentAsMultipartFormData(m))) 
        } 
        case _ => { 
          Play.logger.trace("Parsing AnyContent as raw") 
          raw(request).map(_.right.map(r => AnyContentAsRaw(r))) 
           }
      }
     }
```

首先，它检查请求类型是否支持在请求中发送数据。如果不支持，它返回`AnyContentAsEmpty`（你可以通过在路由文件中将请求类型改为 GET 并发送 GET 请求来检查这一点），否则它将请求的内容类型 Header 与支持的类型进行比较。如果找到匹配项，它将数据转换成相应的类型并返回，否则它将解析为字节并返回`play.api.mvc.RawBuffer`。

### 注意

`AnyContentAsEmpty`、`AnyContentAsText`、`AnyContentAsXml`、`AnyContentAsJson`、`AnyContentAsFormUrlEncoded`、`AnyContentAsMultipartFormData`和`AnyContentAsRaw`都扩展了`AnyContent`特质。

因此，当为支持的某一种内容类型定义了 Action 或者当它是 GET/HEAD 请求时，我们不需要提及解析器。

让我们看看如何在我们的 Action 中访问请求体。我们现在可以更新我们的`subscrib` `e`方法：

```java
def subscribe = Action {
    request => 
      val reqBody: AnyContent = request.body 
      val textContent: Option[String] = reqBody.asText 
      textContent.map { 
        emailId => 
          Ok("added " + emailId + " to subscriber's list") 
      }.getOrElse { 
        BadRequest("improper request body")
      } 
  } 
```

为了访问请求体中的数据，我们需要使用`asText`方法将其从`AnyContent`转换为`Option[String]`。如果我们在 Action 定义中添加解析器，这将变得更加简洁：

```java
def subscribe = Action(parse.text) { 
    request => 
      Ok("added " + request.body + " to subscriber's list") 
  } 
```

`urlFormEncoded`文本 XML 解析器返回标准的 Scala 对象，而其他解析器返回 Play 对象。

我们可以假设订阅请求采用以下格式的 JSON：

```java
{"emailId": "userId@gmail.com", " interval": "month"} 
```

现在，我们需要修改我们的`subscribe`方法为`def subscribe = Action(parse.json) {`，如下所示：

```java
       request => 
      val reqData: JsValue = request.body 
      val emailId = (reqData \ "emailId").as[String] 
      val interval = (reqData \ "interval").as[String] 
      Ok(s"added $emailId to subscriber's list and will send updates every $interval") 
  } 
```

对于以下请求：

```java
curl 'http://localhost:9000/subscribe' -H 'Content-Type: text/json' --data-binary '{"emailId": "userId@gmail.com", "interval": "month"}' 
```

我们得到以下响应：

将`userId@gmail.com`添加到订阅者列表，并将每月发送更新

`parse.json`将请求体转换为`play.api.libs.json.JsValue`。`\`运算符用于访问特定键的值。同样，还有一个`\\`运算符，它可以用于键的值，尽管它可能不是当前节点的直接子节点。Play-Json 有几个简化 JSON 格式数据处理的方法，例如修改结构或将其转换为 Scala 模型等。Play-Json 也可以作为一个独立的库使用，以便在非 Play 项目中使用。其文档可在[`www.playframework.com/documentation/2.3.x/ScalaJson`](https://www.playframework.com/documentation/2.3.x/ScalaJson)找到。

现在，让我们看看如何编写一个添加新用户的 Action，它接受内容类型为 multipart 的请求：

```java
import java.io.File 

  def createProfile = Action(parse.multipartFormData) {
    request => 
      val formData = request.body.asFormUrlEncoded
      val email: String = formData.get("email").get(0)
      val name: String = formData.get("name").get(0)
      val userId: Long = User(email, name).save
      request.body.file("displayPic").map {
        picture => 
          val path = "/socialize/user/" 
          if (!picture.filename.isEmpty) {
            picture.ref.moveTo(new File(path + userId + ".jpeg"))
          }
          Ok("successfully added user")
      }.getOrElse {
           BadRequest("failed to add user")
      }
  }
```

请求有三个字段：`email`、`name`和`displayPic`。从请求数据中，我们获取电子邮件、姓名并添加新用户。`User.` `save`方法在用户表中添加条目，如果存在具有相同电子邮件 ID 的用户，则抛出错误。这就是为什么文件中的操作只在添加用户后执行。`displayPic`是可选的；因此，在保存图像之前，会检查其长度是否大于零。

### 备注

在文件相关操作之前完成数据事务会更好，因为它们可能会失败，而对于不正确的请求，可能不需要文件相关操作。以下表格显示了支持的内容类型、解析器和它们的默认转换。

| 内容类型 | 解析器 | 解析为 Scala 类型 |
| --- | --- | --- |
| `text/plain` | `text` | `String` |
| `application/json`或`text/json` | `json` | `play.api.libs.json.JsValue` |
| `application/xml`, `text/xml`, or `application/XXX+xml` | `xml` | `NodeSeq` |
| `application/form-url-encoded` | `urlFormEncoded` | `Map[String, Seq[String]]` |
| `multipart/form-data` | `multipartFormData` | `play.api.mvc.MultipartFormData[TemporaryFile]` |
| `other` | `raw` | `Play.api.mvc.RawBuffer` |

# 扩展解析器

让我们扩展 JSON 解析器，以便我们得到一个订阅模型。我们将假设`Subscription`模型如下定义：

```java
case class Subscription(emailId: String, interval: String) 
```

现在，让我们编写一个解析器，将请求体转换为订阅对象。以下代码应在控制器中编写：

```java
val parseAsSubscription = parse.using {
    request => 
      parse.json.map {
        body => 
          val emailId:String = (body \ "emailId").as[String] 
          val fromDate:Long = (body \ "fromDate").as[Long] 
          Subscription(emailId, fromDate)
      }
  }

  implicit val subWrites = Json.writes[Subscription]
  def getSub = Action(parseAsSubscription) {
    request => 
      val subscription: Subscription = request.body
      Ok(Json.toJson(subscription))
   } 
```

此外，还有宽容的解析器。宽容的意思是，格式中的错误不会被忽略。这仅仅意味着它忽略了请求中的内容类型头，并基于指定的类型进行解析。例如，让我们更新`subscribe`方法：

```java
def subscribe = Action(parse.tolerantJson) {
    request => 
      val reqData: JsValue = request.body
      val emailId = (reqData \ "email").as[String]
      val interval = (reqData \ "interval").as[String]
  Ok(s"added $emailId to subscriber's list and will send updates every $interval") 
    } 
```

现在，一个内容类型为文本的请求和一个内容类型为 text/JSON 或任何其他类型的请求将给出相同的结果。Play 支持的所有基本解析器都有宽容的解析器。

# 探索结果

在 Play 中，对请求的响应是一个**结果**。结果有两个组成部分：响应头和响应体。让我们看看这个简单的例子：

```java
def plainResult = Action {
  Result( 
    header = ResponseHeader(200, Map(CONTENT_TYPE -> "text/plain")), 
    body = Enumerator("This is the response from plainResult method".getBytes())
  )
}
```

注意，我们使用了枚举来表示响应体。枚举是一种向迭代器提供数据的方式。我们将在第六章“反应式数据流”中详细讨论这些内容。

除了这个之外，结果还有额外的功能，这些功能为我们提供了更好的手段来处理响应头、会话、cookie 等。

结果可以发送 JSON、XML 和图像作为响应，除了字符串内容。生成结果的一个更简单的方法是使用结果助手。结果助手用于大多数 HTTP 响应状态。作为一个例子，让我们看看 Play 内置的`TODO`动作是如何实现的：

```java
val TODO = Action {
    NotImplementedplay.api.templates.Html) 
  } 
```

在这个片段中，`NotImplemented`是一个助手，它返回一个状态为 501 的结果，而`views.html.defaultpages.todo()`返回默认页面，即`todo.scala.html`。

作为例子，我们将考虑发送用户个人头像的 Action。现在的 Action 将是这样的：

```java
def getUserImage(userId: Long) = Action {
    val path: String = s"/socialize/user/$userId.jpeg"
    val img = new File(path)
    if (img.exists()) {
      Ok.sendFile( 
        content = img, 
        inline = true
      )
     }
    else
      NoContent
  } 
```

在这里，我们尝试使用预定义的`getUserImagePath`方法加载用户的个人头像。如果图像文件存在并附加到响应中，我们返回一个状态码为`204`的响应。

我们还看到了如何使用结果助手发送页面内容，无论是静态的还是动态的，使用视图：

```java
def listArtist = Action { 
     Ok(views.html.home(Artist.fetch)) 
  }
```

我们还可以使用`Status`类来生成结果，如下所示：

```java
def save = Action(parse.text) { 
    request => 
      Status(200)("Got: " + request.body) 
  } 
```

此表显示了结果助手及其对应的状态码：

| 结果助手 | 状态码常量 | 状态码 |
| --- | --- | --- |
| – | `CONTINUE` | 100 |
| – | `SWITCHING_PROTOCOLS` | 101 |
| `Ok` | `OK` | 200 |
| `Created` | `CREATED` | 201 |
| `Accepted` | `ACCEPTED` | 202 |
| `NonAuthoritativeInformation` | `NON_AUTHORITATIVE_INFORMATION` | 203 |
| `NoContent` | `NO_CONTENT` | 204 |
| `ResetContent` | `RESET_CONTENT` | 205 |
| `PartialContent` | `PARTIAL_CONTENT` | 206 |
| `MultiStatus` | `MULTI_STATUS` | 207 |
| – | `MULTIPLE_CHOICES` | 300 |
| `MovedPermanently` | `MOVED_PERMANENTLY` | 301 |
| `Found` | `FOUND` | 302 |
| `SeeOther` | `SEE_OTHER` | 303 |
| `NotModified` | `NOT_MODIFIED` | 304 |
| – | `USE_PROXY` | 305 |
| `TemporaryRedirect` | `TEMPORARY_REDIRECT` | 307 |
| `BadRequest` | `BAD_REQUEST` | 400 |
| `Unauthorized` | `UNAUTHORIZED` | 401 |
| – | `PAYMENT_REQUIRED` | 402 |
| `Forbidden` | `FORBIDDEN` | 403 |
| `NotFound` | `NOT_FOUND` | 404 |
| `MethodNotAllowed` | `METHOD_NOT_ALLOWED` | 405 |
| `NotAcceptable` | `NOT_ACCEPTABLE` | 406 |
| – | `PROXY_AUTHENTICATION_REQUIRED` | 407 |
| `RequestTimeout` | `REQUEST_TIMEOUT` | 408 |
| `Conflict` | `CONFLICT` | 409 |
| `Gone` | `GONE` | 410 |
| – | `LENGTH_REQUIRED` | 411 |
| `PreconditionFailed` | `PRECONDITION_FAILED` | 412 |
| `EntityTooLarge` | `REQUEST_ENTITY_TOO_LARGE` | 413 |
| `UriTooLong` | `REQUEST_URI_TOO_LONG` | 414 |
| `UnsupportedMediaType` | `UNSUPPORTED_MEDIA_TYPE` | 415 |
| – | `REQUESTED_RANGE_NOT_SATISFIABLE` | 416 |
| `ExpectationFailed` | `EXPECTATION_FAILED` | 417 |
| `UnprocessableEntity` | `UNPROCESSABLE_ENTITY` | 422 |
| `Locked` | `LOCKED` | 423 |
| `FailedDependency` | `FAILED_DEPENDENCY` | 424 |
| `TooManyRequest` | `TOO_MANY_REQUEST` | 429 |
| `InternalServerError` | `INTERNAL_SERVER_ERROR` | 500 |
| `NotImplemented` | `NOT_IMPLEMENTED` | 501 |
| `BadGateway` | `BAD_GATEWAY` | 502 |
| `ServiceUnavailable` | `SERVICE_UNAVAILABLE` | 503 |
| `GatewayTimeout` | `GATEWAY_TIMEOUT` | 504 |
| `HttpVersionNotSupported` | `HTTP_VERSION_NOT_SUPPORTED` | 505 |
| `InsufficientStorage` | `INSUFFICIENT_STORAGE` | 507 |

# 异步操作

假设我们正在美食广场，并在一个亭子点餐，我们得到了一个凭证和账单。稍后，当订单准备好时，亭子闪烁凭证号码，当我们注意到它时，我们取走订单。

这类似于具有异步响应周期的请求，其中亭子充当服务器，订单类似于请求，凭证作为承诺，当订单准备好时得到解决。

大多数操作最好异步处理。这也通常是首选，因为它不会在操作完成之前阻塞服务器资源。

Play Action 是一个辅助对象，它扩展了 `ActionBuilder` 特性。`ActionBuilder` 特性的 `apply` 方法实现了我们之前看到的 `Action` 特性。让我们看看 `ActionBuilder` 特性中的相关代码：

```java
trait ActionBuilder[+R[_]] extends ActionFunction[Request, R] { 
  self => 

  final def applyA(block: R[A] => 
    Result): Action[A] = async(bodyParser) { req: R[A] => 
    Future.successful(block(req)) 
  } 

  final def asyncA(block: R[A] => Future[Result]): Action[A] = composeAction(new Action[A] { 
    def parser = composeParser(bodyParser) 
    def apply(request: Request[A]) = try { 
      invokeBlock(request, block) 
    } catch { 
      // NotImplementedError is not caught by NonFatal, wrap it 
      case e: NotImplementedError => throw new RuntimeException(e) 
      // LinkageError is similarly harmless in Play Framework, since automatic reloading could easily trigger it 
      case e: LinkageError => throw new RuntimeException(e) 
    } 
    override def executionContext = ActionBuilder.this.executionContext 
  }) 

... 

}  
```

注意，`apply` 方法本身在内部调用 `async` 方法。`async` 方法期望我们定义 Action，这会导致 `Future[Result]`，从而帮助我们编写非阻塞代码。

我们将使用相同的方法来定义一个异步 Action。假设我们需要从远程客户端获取请求的文件，整合/分析数据，然后发送结果。由于我们不知道文件的大小以及与远程客户端的网络连接状态，最好异步处理 Action。Action 将以这种方式定义：

```java
def getReport(fileName:String ) = Action.async { 
     Future { 
      val file:File = new File(fileName) 
      if (file.exists()) { 
        val info = file.lastModified() 
        Ok(s"lastModified on ${new Date(info)}") 
      } 
      else 
        NoContent 
    } 
  } 
```

在获取文件后，如果它是空的，我们发送一个状态码为 204 的响应，否则我们继续处理并发送处理后的数据作为结果的一部分。

我们可能会遇到一个实例，就像我们在前面的例子中看到的那样，我们不想等待远程客户端获取文件超过 10 秒。在这种情况下，我们需要以这种方式修改操作定义：

```java
def getReport(fileName: String) = Action.async { 

    val mayBeFile = Future { 
      new File(fileName) 
    } 
    val timeout = play.api.libs.concurrent.Promise.timeout("Past max time", 10, TimeUnit.SECONDS) 
    Future.firstCompletedOf(Seq(mayBeFile, timeout)).map { 
      case f: File => 
        if (f.exists()) {
          val info = f.lastModified()
          Ok(s"lastModified on ${new Date(info)}")
        } 
        else
          NoContent
      case t: String => InternalServerError(t)
    }
  }
```

因此，如果远程客户端在 10 秒内没有响应请求的文件，我们将收到一个状态码为 500 的响应，内容是我们为超时设置的消息，“超过最大时间”。

# 内容协商

根据 HTTP 协议：

> *内容协商是在有多种表示形式可供选择时，选择给定响应的最佳表示的过程。*

它可以是服务器驱动的，也可以是代理驱动的，或者两者的组合，这被称为透明协商。Play 提供了对服务器驱动协商的支持。这由渲染特性处理，并由控制器特性扩展。控制器特性是 Play 应用程序中控制器对象扩展的地方。

让我们看看 `Rendering` 特性：

```java
trait Rendering {

   object render { 

    //Tries to render the most acceptable result according to the request's Accept header value. 
    def apply(f: PartialFunction[MediaRange, Result])(implicit request: RequestHeader): Result = { 
      def _render(ms: Seq[MediaRange]): Result = ms match {
        case Nil => NotAcceptable 
        case Seq(m, ms @ _*) => 
          f.applyOrElse(m, (m: MediaRange) => _render(ms)) 
      } 

      // "If no Accept header field is present, then it is assumed that the client accepts all media types." 
      val result = 
        if (request.acceptedTypes.isEmpty) _render(Seq(new MediaRange("*", "*", Nil, None, Nil))) 
        else _render(request.acceptedTypes) 
      result.withHeaders(VARY -> ACCEPT) 
    } 

    /**Tries to render the most acceptable result according to the request's Accept header value. 
      * This function can be used if you want to do asynchronous processing in your render function. 
     */ 
    def async(f: PartialFunction[MediaRange, Future[Result]])(implicit request: RequestHeader): Future[Result] = { 
      def _render(ms: Seq[MediaRange]): Future[Result] = ms match { 
        case Nil => Future.successful(NotAcceptable) 
         case Seq(m, ms @ _*) => 
           f.applyOrElse(m, (m: MediaRange) => _render(ms)) 
      } 

      // "If no Accept header field is present, then it is assumed that the client accepts all media types." 
      val result = 
        if (request.acceptedTypes.isEmpty) _render(Seq(new MediaRange("*", "*", Nil, None, Nil))) 
        else _render(request.acceptedTypes) 
      result.map(_.withHeaders(VARY -> ACCEPT)) 
     } 
  } 
}
```

在 `apply` 方法中定义的 `_render` 方法会在请求的接受头中调用部分 `f` 函数。如果没有为任何接受头定义 `f`，则转发状态码为 406 的响应。如果定义了，则返回 `f` 对第一个定义了 `f` 的接受头的第一个结果。

由于控制器扩展了渲染特性，我们可以在我们的操作定义中使用渲染对象。例如，我们可能有一个操作，在从具有 XML 格式的文件中读取配置后，根据请求的接受头以 JSON 和 XML 格式获取配置。让我们看看这是如何实现的：

```java
def getConfig = Action {
    implicit request => 
      val xmlResponse: Node = <metadata> 
        <company>TinySensors</company> 
        <batch>md2907</batch> 
      </metadata> 

      val jsonResponse = Json.obj("metadata" -> Json.arr( 
        Json.obj("company" -> "TinySensors"), 
         Json.obj("batch" -> "md2907")) 
      ) 
      render { 
         case Accepts.Xml() => Ok(xmlResponse) 
        case Accepts.Json() => Ok(jsonResponse) 
      } 
  } 
```

在这个片段中，`Accepts.Xml()` 和 `Accepts.Json()` 是 Play 的辅助方法，用于检查请求是否接受 `application/xml` 和 `application/json` 类型的响应。目前有四个预定义的接受类型，如下表所示：

| 请求接受辅助 | 接受头值 |
| --- | --- |
| XML | `application/xml` |
| JSON | `application/json` |
| HTML | `text/html` |
| JavaScript | `text/javascript` |

这由 `RequestExtractors` 特性和 `AcceptExtractors` 特性促进。`RequestExtractors` 也由控制器特性扩展。让我们看看这里的提取器特性：

```java
trait RequestExtractors extends AcceptExtractors {

  //Convenient extractor allowing to apply two extractors. 
  object & { 
    def unapply(request: RequestHeader): Option[(RequestHeader, RequestHeader)] = Some((request, request)) 
  } 

}

//Define a set of extractors allowing to pattern match on the Accept HTTP header of a request 
trait AcceptExtractors {

  //Common extractors to check if a request accepts JSON, Html, etc. 
  object Accepts { 
    import play.api.http.MimeTypes 
    val Json = Accepting(MimeTypes.JSON) 
    val Html = Accepting(MimeTypes.HTML) 
    val Xml = Accepting(MimeTypes.XML) 
    val JavaScript = Accepting(MimeTypes.JAVASCRIPT) 
  } 

}

//Convenient class to generate extractors checking if a given mime type matches the Accept header of a request. 
case class Accepting(val mimeType: String) {
  def unapply(request: RequestHeader): Boolean = request.accepts(mimeType) 
  def unapply(mediaRange: play.api.http.MediaRange): Boolean = mediaRange.accepts(mimeType) 
}
```

从这段代码中，我们只需要定义一个自定义接受值，这是我们期望请求的接受头将具有的值。例如，为了定义 `image/png` 的辅助程序，我们使用以下代码：

```java
val AcceptsPNG = Accepting("image/png") 
```

我们还注意到 `RequestExtractors` 有一个 `&` 对象，我们可以在希望向多个接受类型发送相同响应时使用它。因此，在前面代码中显示的 `getConfig` 方法中，如果为 `application/json` 和 `text/javascript` 发送相同的响应，我们将对其进行如下修改：

```java
def fooBar = Action { 
    implicit request => 
      val xmlResponse: Node = <metadata> 
        <company>TinySensors</company> 
        <batch>md2907</batch> 
      </metadata> 

      val jsonResponse = Json.obj("metadata" -> Json.arr( 
        Json.obj("company" -> "TinySensors"), 
        Json.obj("batch" -> "md2907")) 
      ) 

      render { 
        case Accepts.Xml() => Ok(xmlResponse) 
        case Accepts.Json() & Accepts.JavaScript() => Ok(jsonResponse) 
      }
  }
```

当定义异步操作时，`render` 对象可以被类似地使用。

# 过滤器

在大多数应用程序中，我们需要对所有请求执行相同的操作。我们可能在定义了所有应用程序所需的动作之后，在稍后阶段需要向所有响应中添加一些字段。

那么，在这种情况下，我们是否需要更新所有的动作？

不。这就是过滤器 API 来拯救我们的地方。我们不需要修改定义我们的动作的方式来解决该问题。我们只需要定义一个过滤器并使用它。

让我们看看我们如何定义我们的过滤器：

```java
import org.joda.time.DateTime 
import org.joda.time.format.DateTimeFormat
import play.api.mvc._
import play.api.http.HeaderNames._ 
import play.api.libs.concurrent.Execution.Implicits.defaultContext 

object HeadersFilter {
  val noCache = Filter { 
    (nextFilter, rh) => 
      nextFilter(rh) map { 
        case result: Result => addNoCacheHeaders(result) 
      } 
  } 

  private def addNoCacheHeaders(result: Result): Result = { 
    result.withHeaders(PRAGMA -> "no-cache", 
      CACHE_CONTROL -> "no-cache, no-store, must-revalidate, max-age=0", 
      EXPIRES -> serverTime) 
  } 

  private def serverTime = { 
    val now = new DateTime() 
    val dateFormat = DateTimeFormat.forPattern( 
      "EEE, dd MMM yyyy HH:mm:ss z") 
    dateFormat.print(now) 
  } 
} 
```

`HeadersFilter.noCache` 过滤器将所有需要禁用浏览器缓存的头信息添加到响应中。`PRAGMA`、`CACHE_CONTROL` 和 `EXPIRES` 是由 `play.api.http.HeaderNames` 提供的常量。

现在，为了使用这个过滤器，我们需要更新应用程序的全局设置。

任何基于 Play 的应用程序的全局设置都可以使用一个全局对象进行配置。这是一个名为 `Global` 的对象，位于应用程序目录中。我们将在 第七章 *玩转全局设置* 中了解更多关于全局设置的信息。

定义过滤器应该如何使用有两种方式。这些是：

1.  为全局对象扩展 `WithFilters` 类而不是 `GlobalSettings`。

1.  在全局对象中手动调用过滤器。

    使用 `WithFilters` 的一个例子是：

    ```java
    object Global extends WithFilters(HeadersFilter.noCache) { 
      // ... 
    }
    ```

    现在，让我们看看如何手动完成这个操作：

    ```java
    object Global extends GlobalSettings {
      override def doFilter(action: EssentialAction): EssentialAction = HeadersFilter.noCache(action) 
    }
    ```

    在 Play 中，过滤器定义的方式类似于动作——有一个过滤器特质，它扩展了 `EssentialFilter`，还有一个辅助过滤器对象。辅助过滤器定义如下：

    ```java
    object Filter {
      def apply(filter: (RequestHeader => Future[Result], RequestHeader) => Future[Result]): Filter = new Filter { 
        def apply(f: RequestHeader => Future[Result])(rh: RequestHeader): Future[Result] = filter(f, rh) 
      } 
    }
    ```

    在这个代码片段中，`apply` 方法调用一个新的过滤器，即过滤器特质。

    可以为一个单一的应用程序应用多个过滤器。如果使用 `WithFilters`，它们将按照指定的顺序应用。如果手动设置，我们可以使用 `WithFilters` 类的 `apply` 方法的内部使用的过滤器对象。`Filters` 对象定义如下：

    ```java
    object Filters {
      def apply(h: EssentialAction, filters: EssentialFilter*) = h match { 
        case a: EssentialAction => FilterChain(a, filters.toList) 
        case h => h 
      } 
    }
    ```

    `FilterChain` 是另一个辅助对象，用于从 `EssentialAction` 和多个 `EssentialFilters` 的组合中组合 `EssentialAction`：

    ```java
    object FilterChain {
      def applyA: EssentialAction = new EssentialAction { 
        def apply(rh: RequestHeader): Iteratee[Array[Byte], Result] = { 
          val chain = filters.reverse.foldLeft(action) { (a, i) => i(a) } 
          chain(rh) 
        } 
      } 
    }
    ```

1.  当需要对所有路由进行无差别操作时，推荐使用过滤器。Play 提供了一个过滤器模块，其中包括 `GzipFilter`、`SecurityHeadersFilter` 和 `CSRFFilter`。

# 动作组合

定义一个请求的动作仅仅是使用定义如下所示的动作辅助对象的行为：

```java
object Action extends ActionBuilder[Request] {
  def invokeBlockA => Future[Result]) = block(request) 
}
```

在动作块中编写的代码将用于 `invokeBlock` 方法。该方法是从 `ActionBuilder` 继承的。这是一个提供生成动作的辅助方法的特质。我们定义动作的所有不同方式，如异步、同步、指定或不指定解析器等，都在 `ActionBuilder` 中声明。

我们还可以通过扩展 `ActionBuilder` 并定义一个自定义的 `invoke` 块来定义我们自己的动作。

## 动作组合的需要

让我们通过一个案例研究来探讨。如今，许多应用程序都会跟踪请求，例如请求发起的机器的 IP 地址、接收时间，甚至整个请求本身。在几乎每个为这种应用程序定义的动作中添加相同的代码将是一种罪行。

现在，假设我们需要在每次遇到特定模块（例如管理员用户）时使用`persistReq`方法持久化请求，那么在这种情况下，我们可以定义一个仅在此模块内使用的自定义动作。让我们看看我们如何定义一个在处理之前持久化请求的自定义动作：

```java
import play.api.mvc._
import scala.concurrent.Future

object TrackAction extends ActionBuilder[Request] {
  override protected def invokeBlockA => Future[Result]) = { 
    persistReq(request) 
    block(request) 
  } 

  private def persistReqA = { 
    ...  
  } 
} 
```

在我们的应用程序中，我们可以像使用默认动作一样使用它：

```java
def viewAdminProfile(id: Long) = TrackAction {
    request => 
      ... 
  } 

def updateAdminProfile(id: Long) = TrackAction(parse.json) { 
    request => 
      ... 
  } 
```

定义自定义动作的另一种方式是通过扩展动作特性。因此，我们也可以如下定义`TrackAction`：

```java
case class TrackActionA extends Action[A] { 

  def apply(request: Request[A]): Future[Result] = { 
    persistReq(request) 
    action(request) 
  } 

  private def persistReq(request: Request[A]) = { 
    …  
    } 

  lazy val parser = action.parser 
}
```

它的使用方式类似于以下：

```java
def viewAdminProfile(id: Long) = TrackAction {
    Action {request => 
        … 
  } 
} 

def updateAdminProfile(id: Long) = TrackAction { 
    Action(parse.json) {	request => 
        … 
   } 
   } 
```

注意，我们需要再次在动作对象内部包装动作定义。我们可以通过定义`ActionBuilder`来移除每次包装动作对象时的额外开销，它使用`composeAction`方法：

```java
object TrackingAction extends ActionBuilder[Request] { 
  def invokeBlockA => Future[Result]) = { 
    block(request) 
  } 
  override def composeActionA = new TrackAction(action) 
}
```

现在，使用方式如下：

```java
def viewAdminProfile(id: Long) = TrackingAction {
    request => 
      ... 
  } 

def updateAdminProfile(id: Long) = TrackingAction(parse.json) { 
    request => 
      ... 
    }
```

## 区分动作组合和过滤器

**动作组合**是一个扩展`EssentialAction`并返回结果的动作。当我们需要在少数几个路由或动作上执行操作时，它更为合适。动作组合比过滤器更强大，更擅长处理特定问题，例如身份验证。

它提供了读取、修改甚至阻止请求的支持。还有定义自定义请求类型动作的条款。

## 自定义请求

首先，让我们看看如何定义自定义请求。我们也可以使用`WrappedRequest`类来定义自定义请求。这被定义为如下：

```java
class WrappedRequestA extends Request[A] { 
     def id = request.id 
    def tags = request.tags 
    def body = request.body 
    def headers = request.headers 
    def queryString = request.queryString 
    def path = request.path 
    def uri = request.uri 
    def method = request.method 
    def version = request.version 
    def remoteAddress = request.remoteAddress 
    def secure = request.secure 
   } 
```

假设我们希望将接收请求的时间与每个请求一起传递，我们可以这样定义：

```java
class TimedRequestA extends WrappedRequestA 
```

现在，让我们看看我们如何操作传入的请求并将它们转换为`TimedRequest`：

```java
def timedActionA = Action.async(action.parser) {
  request => 
    val time = new DateTime()
    val newRequest = new AppRequest(time, request)
    action(newRequest)
}
```

因此，`timedAction`动作可以以这种方式在控制器中使用：

```java
def getUserList = timedAction {
    Action {
      request => 
        val users= User.getAll
        Ok(Json.toJson(users))
    }
  }
```

现在，假设我们希望阻止来自某些浏览器的所有请求；可以这样做：

```java
def timedActionA = Action.async(action.parser) {
  request => 
    val time = new DateTime()
    val newRequest = new AppRequestA
    request.headers.get(USER_AGENT).collect {
      case agent if isCompatibleBrowser(agent) => 
        action(newRequest)
      }.getOrElse{
     Future.successful(Ok(views.html.main()))
  }
}
```

这里，`isCompatibleBrowser`方法检查浏览器是否受支持。

我们还可以操作响应；让我们在响应头中添加处理请求所需的时间：

```java
def timedActionA = Action.async(action.parser) { 
   request => 
    val time = new DateTime() 
    val newRequest = new AppRequest(time, request) 
    action(newRequest).map(_.withHeaders("processTime" -> new DateTime().minus(time.getMillis).getMillis.toString())) 
}
```

现在，让我们看看我们如何定义一个用于自定义请求的动作。你可能想知道为什么我们需要自定义请求。以我们的应用程序为例，它为用户提供电子邮件、聊天、阻止、上传、分享等功能。在这种情况下，我们可以将这些功能与用户对象绑定，以便在内部请求中包含用户对象。

## 需要用户对象的原因

我们的 REST API 只发送`userId`，这是一个数字。对于所有这些操作，我们需要用户的`emailId`、`userName`和可选的个人信息图片。让我们以下述方式定义`UserRequest`：

```java
class UserRequestA extends WrappedRequestA 
```

现在，让我们定义一个使用此请求的动作：

```java
def UserAction(userId: Long) = new ActionBuilder[UserRequest] { 
  def invokeBlockA => Future[Result]) = { 
      User.findById(userId).map { user:User => 
         block(new UserRequest(user, request)) 
      } getOrElse { 
        Future.successful(Redirect(views.html.login)) 
      } 
    } 
  } 
```

因此，在我们的 Action 中，我们找到与给定`userId`对应的用户，否则重定向到登录页面。

在这里，我们可以看到如何使用`UserAction`：

```java
def initiateChat(userId:Long,chatWith:Long) = UserRequest{ 
    request=> 
      val status:Boolean = ChatClient.initiate(request.user,chatWith) 
       if(status){ 
        Ok 
      }else{ 
         Unauthorized 
      } 
  } 
```

聊天客户端通过`userId.chatWith`方法向用户发送消息，表明一个用户（其配置文件为`request.user`）想要聊天。如果另一个用户同意，则返回`true`，否则返回`false`。

# 故障排除

你可能会遇到以下需要故障排除的场景：

1.  编译时遇到错误：在这里找不到任何 HTTP 请求头。

    即使你已经使用`RequestHeader`定义了 Action，你仍然会得到这个错误。

    Play 中处理请求的大多数方法都期望一个隐式的`RequestHeader`。遵循这一惯例是为了保持代码简单。例如，让我们看看这里的控制器特质：

    ```java
    trait Controller extends Results with BodyParsers with HttpProtocol with Status with HeaderNames with ContentTypes with RequestExtractors with Rendering { 

      //Provides an empty `Action` implementation: the result is a standard 'Not implemented yet' result page. 
      val TODO = Action { 
         NotImplementedplay.api.templates.Html) 
      } 

      //Retrieves the session implicitly from the request. 
      implicit def session(implicit request: RequestHeader) = request.session 

      //Retrieve the flash scope implicitly from the request. 
      implicit def flash(implicit request: RequestHeader) = request.flash 

      implicit def lang(implicit request: RequestHeader) = { 
        play.api.Play.maybeApplication.map { implicit app => 
          val maybeLangFromCookie = request.cookies.get(Play.langCookieName).flatMap(c => Lang.get(c.value)) 
            maybeLangFromCookie.getOrElse(play.api.i18n.Lang.preferred(request.acceptLanguages)) 
        }.getOrElse(request.acceptLanguages.headOption.getOrElse(play.api.i18n.Lang.defaultLang)) 
      } 
    }
    ```

    注意，`session`、`flash`和`lang`方法接受一个隐式参数，例如请求，它是`RequestHeader`。在这些情况下，我们需要在我们的 Action 定义中标记请求头为隐式。通常，在 Play 应用程序中标记所有请求头为隐式更安全。因此，为了修复这个错误，我们需要修改我们的`Action`定义如下：

    ```java
    def foo = Action {
        implicit request => 
        … 
    }
    ```

1.  我的 GET 请求的请求体没有被解析。你可能想知道为什么。GET 请求不应该有请求体。尽管 HTTP 规范对此并不明确，但通常情况下，浏览器不会转发请求体。Play 请求体解析器在解析之前会检查请求是否允许有请求体，也就是说，请求不是 GET 请求。

    在你的 GET 和 DELETE 请求中避免请求体更好。如果你需要向这些请求添加请求体，也许你应该重新设计你的应用程序的 REST API。

1.  你无法使用 Play 过滤器：`GzipFilter`、`SecurityHeadersFilter`或`CSRFFilter`。你得到一个错误：对象`filters`不是包`play`的成员，在行`import play.filters`中。

    Filters 是一个独立的模块，需要显式包含。你应该在`build.sbt`文件中将它添加为`libraryDependencies += filters`，然后重新加载项目。

1.  使用 Future 时遇到编译错误：如果你找不到隐式的`ExecutionContext`，要么为自己要求一个，要么导入`ExecutionContext.Implicits.global`。然而，为什么要这样做呢？

    未来需要`ExecutionContext`，它定义了线程池，线程将用于操作。因此，如果没有为 Future 提供`ExecutionContext`，你可能会遇到编译错误。有关更多信息，请参阅*Scala 文档中的 Futures*部分 [`docs.scala-lang.org/overviews/core/futures.html`](http://docs.scala-lang.org/overviews/core/futures.html)。

1.  使用 JSON 解析器时遇到运行时错误：`JsResultException`：

    ```java
    JsResultException(errors:List((,List(ValidationError(error.expected.jsstring,WrappedArray())))))] 
    ```

    这通常发生在从 JSON 中提取的字段不在请求体中时。这可能是由于存在拼写错误，例如，应该是 `emailId`，而你可能发送的是电子邮件。你可以使用 `asOpt` 方法代替 `as`。例如：

    ```java
    val emailId = (body\"emailId"). asOpt[String]
    ```

    然后，如果你或任何字段缺失，你可以抛出一个带有友好信息的错误。或者，你可以使用 `getOrElse` 方法传递默认值。

# 摘要

在本章中，我们看到了如何定义和扩展控制器的主要组件。我们看到了如何定义一个具有默认解析器和结果的特定应用程序的动作，以及具有自定义解析器和结果的动作。此外，我们还看到了如何使用过滤器和 `ActionComposition` 来管理特定应用程序的关注点。在这个过程中，我们看到了如何定义一个自定义请求。
