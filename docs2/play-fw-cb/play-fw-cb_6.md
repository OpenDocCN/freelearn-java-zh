# 第六章. 实际模块示例

在本章中，我们将涵盖以下食谱：

+   将 Play 应用程序与消息队列集成

+   将 Play 应用程序与 ElasticSearch 集成

+   使用 JWT 实现令牌认证

# 简介

在本章中，我们将进一步探讨将 Play 应用程序与现代网络应用程序的必要服务和工具集成。具体来说，我们将探讨如何使用 Play 插件集成外部消息队列服务。我们将使用流行的云服务 **IronMQ**。

我们还将探讨如何使用 **ElasticSearch** 和 **Docker** 将全文搜索引擎服务与 Play 应用程序集成。

最后，我们将实现自己的 Play 包装器，用于使用 **JSON Web Tokens**（**JWT**）进行令牌认证的集成。

# 将 Play 应用程序与消息队列集成

在这个食谱中，我们将探讨如何使用 Play 2.0 集成 IronMQ，这是一个流行的云消息队列服务。我们将使用 IronMQ 的 Java 库，可以在以下位置找到：

[`github.com/iron-io/iron_mq_java`](https://github.com/iron-io/iron_mq_java)

我们将使用一个 Play 插件来初始化我们的 IronMQ 客户端和队列对象，并公开发送和检索消息的辅助方法。然后，我们将在这个 Play 控制器中使用此插件，允许客户端使用 HTTP 方法 `GET` 发布和检索消息。

## 如何做到这一点...

对于 Java，我们需要执行以下步骤：

1.  以启用热重载的方式运行 `foo_java` 应用程序：

    ```java
    <span class="strong"><strong>    activator "~run"</strong></span>
    ```

1.  在 [`www.iron.io/`](http://www.iron.io/) 上创建 IronMQ 账户并创建 IronMQ 项目；记下您的项目 ID 和令牌

1.  在 `build.sbt` 中将官方 IronMQ Java 库作为应用程序依赖项导入：

    ```java
    libraryDependencies ++= Seq(
          "io.iron.ironmq" % "ironmq" % "0.0.19"
        )
    ```

1.  在 `foo_java/app` 内创建插件包：

    ```java
    <span class="strong"><strong>    mkdir plugins</strong></span>
    ```

1.  在 `foo_java/app/plugins` 目录中创建我们的插件类，MQPlugin：

    ```java
    package plugins;
         import io.iron.ironmq.Client;
        import io.iron.ironmq.Message;
        import io.iron.ironmq.Messages;
        import io.iron.ironmq.Queue;
        import play.Application;
        import play.Logger;
        import play.Plugin;
        import java.util.UUID;
         public class MQPlugin extends Plugin {
            final private Integer messageSize = 10;
            private Client client;
            private Queue queue;
             public MQPlugin(Application app) {
                super();
                client = new Client(
                    app.configuration().getString("mq.projectId"),
                    app.configuration().getString("mq.token")
                );
            }
             public void onStart() {
                queue = client.queue(UUID.randomUUID().toString());
            }
             public void onStop() {
                try {
                    queue.clear();
                    queue.destroy();
                    client = null;
                } catch(Exception e) {
                    Logger.error(e.toString());
                }
            }
             public void send(String msg) throws Exception {
                queue.push(msg);
            }
             public Message[] retrieve() throws Exception {
                Messages messages = queue.get(messageSize);
                if (messages.getSize() &gt; 0) {
                    Message[] msgArray = messages.getMessages();
                     for(Message m : msgArray) {
                        queue.deleteMessage(m);
                    }
                     return msgArray;
                }
                 return new Message[] {};
            }
             public boolean enabled() {
                return true;
            }
        }
    ```

1.  修改 `conf/application.conf` 并添加您的 IronMQ 项目 ID 和令牌：

    ```java
    mq.projectId="YOUR PROJECT ID"
        mq.token="YOUR TOKEN"
    ```

1.  通过在 `conf/play.plugins` 文件中声明来初始化 MQPlugin：

    ```java
    599:plugins.MQPlugin
    ```

1.  在 `app/controllers` 中创建 `Messenger` 控制器类：

    ```java
    package controllers;
         import play.Logger;
        import play.Play;
        import play.data.Form;
        import play.mvc.BodyParser;
        import play.mvc.Controller;
        import play.mvc.Result;
        import plugins.MQPlugin;
        import java.util.HashMap;
        import java.util.Map;
         import static play.libs.Json.toJson;
         public class Messenger extends Controller {
            private static MQPlugin mqPlugin = Play.application().plugin(MQPlugin.class);
             @BodyParser.Of(BodyParser.Json.class)
            public static Result sendMessage() {
                try {
                    Form&lt;Message&gt; form = Form.form(Message.class).bindFromRequest();
                    Message m = form.get();
                    mqPlugin.send(m.getBody());
                     Map&lt;String, String&gt; map = new HashMap&lt;&gt;();
                    map.put("status", "Message sent.");
                    return created(toJson(map));
                 } catch (Exception e) {
                    return internalServerError(e.getMessage());
                }
            }
             public static Result getMessages() {
                try {
                    return ok(toJson(mqPlugin.retrieve()));
                } catch (Exception e) {
                    Logger.error(e.toString());
                    return internalServerError();
                }
            }
        }
    ```

1.  为 `Messenger` 控制器操作添加新路由：

    ```java
    POST    /messages/send       controllers.Messenger.sendMessage
        GET     /messages            controllers.Messenger.getMessages
    ```

1.  使用 `curl` 发送和检索消息：

    ```java
    <span class="strong"><strong>    $ curl -v -X POST http://localhost:9000/messages/send --header "Content-type: application/json" -d '{"body":"Her mouth the mischief he doth seek"}'</strong></span>
    <span class="strong"><strong>    * Hostname was NOT found in DNS cache</strong></span>
    <span class="strong"><strong>    *   Trying ::1...</strong></span>
    <span class="strong"><strong>    * Connected to localhost (::1) port 9000 (#0)</strong></span>
    <span class="strong"><strong>    &gt; POST /messages/send HTTP/1.1</strong></span>
    <span class="strong"><strong>    &gt; User-Agent: curl/7.37.1</strong></span>
    <span class="strong"><strong>    &gt; Host: localhost:9000</strong></span>
    <span class="strong"><strong>    &gt; Accept: */*</strong></span>
    <span class="strong"><strong>    &gt; Content-type: application/json</strong></span>
    <span class="strong"><strong>    &gt; Content-Length: 46</strong></span>
    <span class="strong"><strong>    &gt;</strong></span>
    <span class="strong"><strong>    * upload completely sent off: 46 out of 46 bytes</strong></span>
    <span class="strong"><strong>    &lt; HTTP/1.1 201 Created</strong></span>
    <span class="strong"><strong>    &lt; Content-Type: application/json; charset=utf-8</strong></span>
    <span class="strong"><strong>    &lt; Content-Length: 26</strong></span>
    <span class="strong"><strong>    &lt;</strong></span>
    <span class="strong"><strong>    * Connection #0 to host localhost left intact</strong></span>
    <span class="strong"><strong>    {"status":"Message sent."}%</strong></span>
     <span class="strong"><strong>    $ curl -v http://localhost:9000/messages</strong></span>
    <span class="strong"><strong>    * Hostname was NOT found in DNS cache</strong></span>
    <span class="strong"><strong>    *   Trying ::1...</strong></span>
    <span class="strong"><strong>    * Connected to localhost (::1) port 9000 (#0)</strong></span>
    <span class="strong"><strong>    &gt; GET /messages HTTP/1.1</strong></span>
    <span class="strong"><strong>    &gt; User-Agent: curl/7.37.1</strong></span>
    <span class="strong"><strong>    &gt; Host: localhost:9000</strong></span>
    <span class="strong"><strong>    &gt; Accept: */*</strong></span>
    <span class="strong"><strong>    &gt;</strong></span>
    <span class="strong"><strong>    &lt; HTTP/1.1 200 OK</strong></span>
    <span class="strong"><strong>    &lt; Content-Type: application/json; charset=utf-8</strong></span>
    <span class="strong"><strong>    &lt; Content-Length: 95</strong></span>
    <span class="strong"><strong>    &lt;</strong></span>
    <span class="strong"><strong>    * Connection #0 to host localhost left intact</strong></span>
    <span class="strong"><strong>    {"messages":["Her mouth the mischief he doth seek","Her heart the captive of which he speaks"]}%</strong></span>
    ```

1.  在 IronMQ 网络控制台中，您还可以确认队列消息大小，以确认我们能够发布消息！![图片](img/CiOxgcxa.jpg)

对于 Scala，我们需要执行以下步骤：

1.  以启用热重载的方式运行 `foo_scala` 应用程序：

    ```java
    <span class="strong"><strong>    activator "~run"</strong></span>
    ```

1.  在 [`www.iron.io/`](http://www.iron.io/) 上创建 IronMQ 账户并创建 IronMQ 项目；记下您的项目 ID 和令牌。

1.  在 `build.sbt` 中将官方 IronMQ Java 库作为应用程序依赖项导入：

    ```java
    libraryDependencies ++= Seq(
          "io.iron.ironmq" % "ironmq" % "0.0.19"
        )
    ```

1.  在 `foo_scala/app` 中创建插件包：

    ```java
    <span class="strong"><strong>    mkdir plugins</strong></span>
    ```

1.  在 `foo_scala/app/plugins` 目录中创建我们的插件类，MQPlugin：

    ```java
    package plugins
         import java.util.UUID
        import io.iron.ironmq.{Client, Queue}
        import play.api.Play.current
        import play.api.{Application, Play, Plugin}
        import play.api.libs.concurrent.Execution.Implicits._
        import scala.concurrent.Future
         class MQPlugin(app: Application) extends Plugin {
          private val messageSize = 10
          private var client: Client = null
          private var queue: Queue = null
           override def onStart() = {
            client = new Client(
              Play.configuration.getString("mq.projectId").get,
              Play.configuration.getString("mq.token").get
            )
            queue = client.queue(UUID.randomUUID().toString)
          }
           override def onStop() = {
            queue.clear()
            queue.destroy()
            client = null
          }
           def send(msg: String) = queue.push(msg)
           def retrieve = {
            val list = queue.get(messageSize)
            Future {
              list.getMessages.map(queue.deleteMessage(_))
            }
            list.getMessages.map(_.getBody)
          }
        override def enabled = true
        }
    ```

1.  修改 `conf/application.conf` 并添加您的 IronMQ 项目 ID 和令牌：

    ```java
    mq.projectId="YOUR PROJECT ID"
        mq.token="YOUR TOKEN"
    ```

1.  通过在 `conf/play.plugins` 文件中声明来初始化 MQPlugin：

    ```java
    599:plugins.MQPlugin
    ```

1.  在 `app/controllers` 中创建 `Messenger` 控制器类：

    ```java
    package controllers
         import play.api.Play.current
        import play.api.Play
        import play.api.libs.json.{JsError, Json}
        import play.api.mvc.{BodyParsers, Action, Controller}
        import plugins.MQPlugin
         case class Message(body: String)
         object Messenger extends Controller {
          implicit private val writes = Json.writes[Message]
          implicit private val reads = Json.reads[Message]
          private val mqPlugin = Play.application.plugin[MQPlugin].get
           def sendMessage = Action(BodyParsers.parse.json) { implicit request =&gt;
            val post = request.body.validate[Message]
             post.fold(
              errors =&gt; BadRequest(JsError.toFlatJson(errors)),
              p =&gt; {
                mqPlugin.send(p.body)
                Created(Json.obj("status" -&gt; "Message sent."))
              }
            )
          }
           def getMessages = Action {
            Ok(Json.obj("messages" -&gt; mqPlugin.retrieve))
          }
        }
    ```

1.  为`Messenger`控制器动作添加新路由：

    ```java
    POST    /messages/send       controllers.Messenger.sendMessage
        GET     /messages            controllers.Messenger.getMessages
    ```

1.  使用`curl`发送和检索消息：

    ```java
    <span class="strong"><strong>    $ curl -v -X POST http://localhost:9000/messages/send --header "Content-type: application/json" -d '{"body":"Her mouth the mischief he doth seek"}'</strong></span>
    <span class="strong"><strong>    * Hostname was NOT found in DNS cache</strong></span>
    <span class="strong"><strong>    *   Trying ::1...</strong></span>
    <span class="strong"><strong>    * Connected to localhost (::1) port 9000 (#0)</strong></span>
    <span class="strong"><strong>    &gt; POST /messages/send HTTP/1.1</strong></span>
    <span class="strong"><strong>    &gt; User-Agent: curl/7.37.1</strong></span>
    <span class="strong"><strong>    &gt; Host: localhost:9000</strong></span>
    <span class="strong"><strong>    &gt; Accept: */*</strong></span>
    <span class="strong"><strong>    &gt; Content-type: application/json</strong></span>
    <span class="strong"><strong>    &gt; Content-Length: 46</strong></span>
    <span class="strong"><strong>    &gt;</strong></span>
    <span class="strong"><strong>    * upload completely sent off: 46 out of 46 bytes</strong></span>
    <span class="strong"><strong>    &lt; HTTP/1.1 201 Created</strong></span>
    <span class="strong"><strong>    &lt; Content-Type: application/json; charset=utf-8</strong></span>
    <span class="strong"><strong>    &lt; Content-Length: 26</strong></span>
    <span class="strong"><strong>    &lt;</strong></span>
    <span class="strong"><strong>    * Connection #0 to host localhost left intact</strong></span>
    <span class="strong"><strong>    {"status":"Message sent."}%</strong></span>
     <span class="strong"><strong>    $ curl -v http://localhost:9000/messages</strong></span>
    <span class="strong"><strong>    * Hostname was NOT found in DNS cache</strong></span>
    <span class="strong"><strong>    *   Trying ::1...</strong></span>
    <span class="strong"><strong>    * Connected to localhost (::1) port 9000 (#0)</strong></span>
    <span class="strong"><strong>    &gt; GET /messages HTTP/1.1</strong></span>
    <span class="strong"><strong>    &gt; User-Agent: curl/7.37.1</strong></span>
    <span class="strong"><strong>    &gt; Host: localhost:9000</strong></span>
    <span class="strong"><strong>    &gt; Accept: */*</strong></span>
    <span class="strong"><strong>    &gt;</strong></span>
    <span class="strong"><strong>    &lt; HTTP/1.1 200 OK</strong></span>
    <span class="strong"><strong>    &lt; Content-Type: application/json; charset=utf-8</strong></span>
    <span class="strong"><strong>    &lt; Content-Length: 95</strong></span>
    <span class="strong"><strong>    &lt;</strong></span>
    <span class="strong"><strong>    * Connection #0 to host localhost left intact</strong></span>
    <span class="strong"><strong>    {"messages":["Her mouth the mischief he doth seek","Her heart the captive of which he speaks"]}%</strong></span>
    ```

1.  在 IronMQ Web 控制台中，您还可以检查队列消息大小以确认我们能够发布消息：![图片](img/CZSpoQpf.jpg)

## 它是如何工作的…

在此配方中，我们首先通过将官方 IronMQ Java 库导入到`build.sbt`中来设置消息队列插件。我们还需要登录 IronMQ 来创建我们的 IronMQ 项目和检索我们的项目令牌：

![图片](img/RBXJtXDJ.jpg)

我们通过将 IronMQ 凭据添加到`conf/application.conf`来配置 Play 应用程序：

```java
mq.projectId="YOUR PROJECT ID"
    mq.token="YOUR TOKEN"
```

然后，我们通过从配置文件中检索项目 ID 和令牌并将它们传递给`io.iron.ironmq.Client`实例来实现`MQPlugin`类：

```java
// Java 
    public MQPlugin(Application app) {
        super();
        client = new Client(
            app.configuration().getString("mq.projectId"),
            app.configuration().getString("mq.token")
        );
    }
     // Scala
    override def onStart() = {
      client = new Client(
        Play.configuration.getString("mq.projectId").get,
        Play.configuration.getString("mq.token").get
      )
    }
```

我们在`onStart`方法中创建了我们的动态消息队列，并将`UUID`参数作为默认队列名称传递：

```java
// Java 
    public void onStart() {
        queue = client.queue(UUID.randomUUID().toString());
    }
     // Scala
    queue = client.queue(UUID.randomUUID().toString)
```

然后，我们声明了两个将促进消息发送和检索的方法：

```java
// Java
    public void send(String msg) throws Exception {
        queue.push(msg);
    }
     public Message[] retrieve() throws Exception {
        Messages messages = queue.get(messageSize);
        if (messages.getSize() &gt; 0) {
            Message[] msgArray = messages.getMessages();
             for(Message m : msgArray) {
                queue.deleteMessage(m);
            }
             return msgArray;
        }
         return new Message[] {};
    }    
     // Scala
    def send(msg: String) = queue.push(msg)
     def retrieve = {
      val list = queue.get(messageSize)
      Future {
        list.getMessages.map(queue.deleteMessage(_))
      }
      list.getMessages.map(_.getBody)
    }
```

我们希望能够从队列中删除已读消息并在此处运行一个异步函数来删除它们：

```java
// Scala
    import play.api.libs.concurrent.Execution.Implicits._
    import scala.concurrent.Future
     Future {
      list.getMessages.map(queue.deleteMessage(_))
    }
```

对于 Java，删除消息将同步发生：

```java
for(Message m : msgArray) {
      queue.deleteMessage(m);
    }
```

最后，我们通过创建控制器类`Messenger`来实现端点，该类公开两个动作；一个用于检索消息，另一个用于发布消息：

```java
// Java 
    private static MQPlugin mqPlugin = Play.application().plugin(MQPlugin.class);
     @BodyParser.Of(BodyParser.Json.class)
    public static Result sendMessage() {
        try {
            Form&lt;Message&gt; form = Form.form(Message.class).bindFromRequest();
            Message m = form.get();
            mqPlugin.send(m.getBody());
             Map&lt;String, String&gt; map = new HashMap&lt;&gt;();
            map.put("status", "Message sent.");
            return created(toJson(map));
         } catch (Exception e) {
            return internalServerError(e.getMessage());
        }
    }
     public static Result getMessages() {
        try {
            return ok(toJson(mqPlugin.retrieve()));
        } catch (Exception e) {
            Logger.error(e.toString());
            return internalServerError();
        }
    }    
    // Scala
    implicit private val writes = Json.writes[Message]
    implicit private val reads = Json.reads[Message]
    private val mqPlugin = Play.application.plugin[MQPlugin].get
     def sendMessage = Action(BodyParsers.parse.json) { implicit request =&gt;
      val post = request.body.validate[Message]
       post.fold(
        errors =&gt; BadRequest(JsError.toFlatJson(errors)),
        p =&gt; {
          mqPlugin.send(p.body)
          Created(Json.obj("status" -&gt; "Message sent."))
        }
      )
    }
     def getMessages = Action {
      Ok(Json.obj("messages" -&gt; mqPlugin.retrieve))
    }
```

最后，将相应的路由添加到`conf/routes`：

```java
POST    /messages/send       controllers.Messenger.sendMessage
    GET     /messages            controllers.Messenger.getMessages
```

# 将 Play 应用程序与 ElasticSearch 集成

在此配方中，我们将创建一个非常常见的 Web 应用程序功能，用于创建、索引和搜索，在我们的案例中是产品。我们将使用 ElasticSearch 作为我们的搜索服务。我们将使用 Docker 创建我们的本地 ElasticSearch 容器并运行所有搜索操作。

此配方的先决条件是能够访问 ElasticSearch 实例，无论是本地还是远程，在我们的配方中，以及在本地的开发机器上安装 Docker：

```java
<span class="strong"><strong>    $ docker -v</strong></span>
<span class="strong"><strong>    Docker version 1.3.3, build d344625</strong></span>
```

我们使用 Docker 使用以下命令部署我们的本地 ElasticSearch 实例：

```java
<span class="strong"><strong>    docker run -d -p 9200:9200 -p 9300:9300 dockerfile/elasticsearch</strong></span>
```

此前命令指示`docker`以分离服务运行`elasticsearch`容器，并且容器中的端口`9200`和`9300`应可以从主机中的相应端口访问。

我们将使用一个开源的 Play 模块，**play2-elasticsearch**，来包装我们对 ElasticSearch 实例的调用。此配方假设对 Docker 和全文搜索服务有一定的了解。有关 play2-elasticsearch 的更多信息，请参阅[`github.com/cleverage/play2-elasticsearch`](https://github.com/cleverage/play2-elasticsearch)。

### 注意

有关 Docker 及其安装的更多信息，请参阅他们的在线文档[`docs.docker.com/installation/`](https://docs.docker.com/installation/)。

## 如何做到这一点…

对于 Java，我们需要执行以下步骤：

1.  首先，让我们使用 Docker 启动一个本地的 ElasticSearch 容器：

    ```java
    <span class="strong"><strong>    $ docker run -d -p 9200:9200 -p 9300:9300 dockerfile/elasticsearch</strong></span>
    ```

1.  使用启用热重载的`foo_java`应用程序：

    ```java
    <span class="strong"><strong>    activator "~run"</strong></span>
    ```

1.  在`build.sbt`中添加 play2-elasticsearch 依赖。需要注意的是，截至编写本文时，play2-elasticsearch 尚未发布对 Play 2.3.x 的支持，因此需要排除旧版 Play 库的依赖：

    ```java
    resolvers += "Sonatype OSS Snapshots" at "https://oss.sonatype.org/content/repositories/snapshots"
         libraryDependencies ++= Seq(
          ("com.clever-age" % "play2-elasticsearch" % "1.4-SNAPSHOT")
            .exclude("com.typesafe.play", "play-functional_2.10")
            .exclude("com.typesafe.akka", "akka-actor_2.10")
            .exclude("com.typesafe.play", "play-json_2.10")
            .exclude("com.typesafe.play", "play_2.10")
            .exclude("com.typesafe.play", "play-iteratees_2.10")
            .exclude("com.typesafe.akka", "akka-slf4j_2.10")
            .exclude("org.scala-stm", "scala-stm_2.10")
            .exclude("com.typesafe.play", "play-datacommons_2.10")
         .exclude("com.typesafe.play", "play-java_2.10")
        )
    ```

1.  在`conf/play.plugins`中声明 play2-elasticsearch 插件：

    ```java
    9000:com.github.cleverage.elasticsearch.plugin.IndexPlugin
    ```

1.  将 play2-elasticsearch 配置参数添加到`conf/application.conf`：

    ```java
    elasticsearch.local=false
        elasticsearch.client="&lt;YOUR ELASTIC SEARCH HOST HERE&gt;:9300"
        elasticsearch.sniff=false
        elasticsearch.index.name="test"
        elasticsearch.index.settings="{ analysis: { analyzer: { my_analyzer: { type: \"custom\", tokenizer: \"standard\" } } } }"
        elasticsearch.index.clazzs="models.*"
        elasticsearch.index.show_request=true
        elasticsearch.cluster.name=elasticsearch
    ```

1.  在`app/models/Product.java`中创建产品模型：

    ```java
    package models;
         import com.github.cleverage.elasticsearch.Index;
        import com.github.cleverage.elasticsearch.IndexQuery;
        import com.github.cleverage.elasticsearch.IndexResults;
        import com.github.cleverage.elasticsearch.Indexable;
        import com.github.cleverage.elasticsearch.annotations.IndexType;
        import org.elasticsearch.index.query.QueryBuilders;
        import java.util.HashMap;
        import java.util.Map;
         @IndexType(name = "product")
        public class Product extends Index {
            private String id;
            private String title;
            private String shortDesc;
             public Product() {}
             public Product(String id, String title, String shortDesc) {
                this.id = id;
                this.title = title;
                this.shortDesc = shortDesc;
            }
             public String getId() {
                return id;
            }
             public void setId(String id) {
                this.id = id;
            }
             public String getTitle() {
                return title;
            }
             public void setTitle(String title) {
                this.title = title;
            }
             public String getShortDesc() {
                return shortDesc;
            }
             public void setShortDesc(String shortDesc) {
                this.shortDesc = shortDesc;
            }
             public static Finder&lt;Product&gt; find = new Finder&lt;&gt;(Product.class);
             @Override
            public Map toIndex() {
                Map&lt;String, Object&gt; map = new HashMap&lt;&gt;();
                map.put("id", this.id);
                map.put("title", this.title);
                map.put("description", this.getShortDesc());
                return map;
            }
             @Override
            public Indexable fromIndex(Map map) {
                Product p = new Product();
                p.setId((String) map.get("id"));
                p.setTitle((String) map.get("title"));
                p.setShortDesc((String) map.get("description"));
                return p;
            }
             public static IndexResults&lt;Product&gt; doSearch(String keyword) {
                IndexQuery&lt;Product&gt; indexQuery = Product.find.query();
                indexQuery.setBuilder(QueryBuilders.multiMatchQuery(keyword, "title", "description"));
                return Product.find.search(indexQuery);
            }
        }
    ```

1.  接下来，在`app/controllers/Products.java`中创建将提供产品创建和搜索的产品端点：

    ```java
    package controllers;
         import com.github.cleverage.elasticsearch.IndexResults;
        import models.Product;
        import play.data.Form;
        import play.mvc.BodyParser;
        import play.mvc.Controller;
        import play.mvc.Result;
        import java.util.HashMap;
        import java.util.Map;
         import static play.libs.Json.toJson;
         public class Products extends Controller {
            @BodyParser.Of(BodyParser.Json.class)
            public static Result create() {
                try {
                    Form&lt;Product&gt; form = Form.form(Product.class).bindFromRequest();
                    Product product = form.get();
                    product.index();
                     return created(toJson(product));
                 } catch (Exception e) {
                    return internalServerError(e.getMessage());
                }
            }
             @BodyParser.Of(BodyParser.Json.class)
            public static Result search() {
                try {
                    Form&lt;Search&gt; form = Form.form(Search.class).bindFromRequest();
                    Search search = form.get();
                     IndexResults&lt;Product&gt; results = Product.doSearch(search.getKeyword());
                    Map&lt;String, Object&gt; map = new HashMap&lt;&gt;();
                    map.put("total", results.getTotalCount());
                    map.put("products", results.getResults());
                     return ok(toJson(map));
                 } catch (Exception e) {
                    return internalServerError(e.getMessage());
                }
            }
        }
    ```

1.  让我们也将辅助类中的`Search`类添加到`app/controllers/Search.java`中：

    ```java
    package controllers;
         public class Search {
            private String keyword;
             public String getKeyword() {
                return keyword;
            }
            public void setKeyword(String keyword) {
                this.keyword = keyword;
            }
        }
    ```

1.  最后，让我们将产品控制器操作的路由添加到`conf/routes`中：

    ```java
    POST    /products            controllers.Products.create
        GET     /products/search     controllers.Products.search
    ```

1.  使用`curl`，我们可以如下测试产品创建和索引：

    ```java
    <span class="strong"><strong>    # Let's insert 2 products:</strong></span>
     <span class="strong"><strong>    curl -v -X POST http://localhost:9000/products --header "Content-type: application/json" -d '{"id":"1001", "title":"Intel Core i7-4790K Processor", "shortDesc": "New Unlocked 4th Gen Intel Core Processors deliver 4 cores of up to 4 GHz base frequency providing blazing-fast computing performance for the most demanding users"}'</strong></span>
     <span class="strong"><strong>    curl -v -X POST http://localhost:9000/products --header "Content-type: application/json" -d '{"id":"1002", "title": "AMD FD6300WMHKBOX FX-6300 6-Core Processor", "shortDesc": "AMD FX 6-Core Processor Unlocked Black Edition. AMDs next-generation architecture takes 8-core processing to a new level. Get up to 24% better frame rates in some of the most demanding games at stunning resolutions. Get faster audio encoding so you can enjoy your music sooner. Go up to 5.0 GHz with aggressive cooling solutions from AMD."}'</strong></span>
    ```

1.  我们也可以使用`curl`来执行产品搜索：

    ```java
    <span class="strong"><strong>    $ curl -X GET http://localhost:9000/products/search --header "Content-type: application/json" -d '{"keyword":"processor"}'</strong></span>
    <span class="strong"><strong>{"total":2,"products":[{"id":"1001","title":"Intel Core i7-4790K Processor","shortDesc":"New Unlocked 4th Gen Intel Core Processors deliver 4 cores of up to 4 GHz base frequency providing blazing-fast computing performance for the most demanding users"},{"id":"1002","title":"AMD FD6300WMHKBOX FX-6300 6-Core Processor","shortDesc":"AMD FX 6-Core Processor Unlocked Black Edition. AMDs next-generation architecture takes 8-core processing to a new level. Get up to 24% better frame rates in some of the most demanding games at stunning resolutions. Get faster audio encoding so you can enjoy your music sooner. Go up to 5.0 GHz with aggressive cooling solutions from AMD."}]}%</strong></span>
     <span class="strong"><strong>    $ curl -X GET http://localhost:9000/products/search --header "Content-type: application/json" -d '{"keyword":"amd"}'</strong></span>
    <span class="strong"><strong>{"total":1,"products":[{"id":"1002","title":"AMD FD6300WMHKBOX FX-6300 6-Core Processor","shortDesc":"AMD FX 6-Core Processor Unlocked Black Edition. AMDs next-generation architecture takes 8-core processing to a new level. Get up to 24% better frame rates in some of the most demanding games at stunning resolutions. Get faster audio encoding so you can enjoy your music sooner. Go up to 5.0 GHz with aggressive cooling solutions from AMD."}]}%</strong></span>
    ```

对于 Scala，我们需要执行以下步骤：

1.  首先，让我们使用 Docker 启动一个本地的 ElasticSearch 容器：

    ```java
    <span class="strong"><strong>    $ docker run -d -p 9200:9200 -p 9300:9300 dockerfile/elasticsearch</strong></span>
    ```

1.  以启用热重载的方式运行`foo_scala`应用程序：

    ```java
    <span class="strong"><strong>    activator "~run"</strong></span>
    ```

1.  在`build.sbt`中添加 play2-elasticsearch 依赖。需要注意的是，截至编写本文时，play2-elasticsearch 尚未发布对 Play 2.3.x 的支持，因此需要排除旧版 Play 库的依赖：

    ```java
    resolvers += "Sonatype OSS Snapshots" at "https://oss.sonatype.org/content/repositories/snapshots"
         libraryDependencies ++= Seq(
          ("com.clever-age" % "play2-elasticsearch" % "1.4-SNAPSHOT")
            .exclude("com.typesafe.play", "play-functional_2.10")
            .exclude("com.typesafe.akka", "akka-actor_2.10")
            .exclude("com.typesafe.play", "play-json_2.10")
            .exclude("com.typesafe.play", "play_2.10")
            .exclude("com.typesafe.play", "play-iteratees_2.10")
            .exclude("com.typesafe.akka", "akka-slf4j_2.10")
            .exclude("org.scala-stm", "scala-stm_2.10")
            .exclude("com.typesafe.play", "play-datacommons_2.10")
        )
    ```

1.  在`conf/play.plugins`中声明 play2-elasticsearch 插件：

    ```java
    9000:com.github.cleverage.elasticsearch.plugin.IndexPlugin
    ```

1.  将 play2-elasticsearch 配置参数添加到`conf/application.conf`：

    ```java
    elasticsearch.local=false
        elasticsearch.client="&lt;YOUR_ELASTICSEARCH_HOST_HERE&gt;:9300"
        elasticsearch.sniff=false
        elasticsearch.index.name="test"
        elasticsearch.index.settings="{ analysis: { analyzer: { my_analyzer: { type: \"custom\", tokenizer: \"standard\" } } } }"
        elasticsearch.index.clazzs="models.*"
        elasticsearch.index.show_request=true
        elasticsearch.cluster.name=elasticsearch
    ```

1.  在`app/models/Product.scala`中创建我们的产品模型和`ProductManager`类：

    ```java
    package models
         import com.github.cleverage.elasticsearch.ScalaHelpers.{IndexQuery,     IndexableManager, Indexable}
        import org.elasticsearch.index.query.QueryBuilders
        import play.api.libs.json.{Writes, Json, Reads}
         case class Product(id: String, title: String, shortDesc: String) extends Indexable
         object ProductManager extends IndexableManager[Product] {
          override val indexType: String = "string"
          override val reads: Reads[Product] = Json.reads[Product]
          override val writes: Writes[Product] = Json.writes[Product]
           def doSearch(keyword: String) = {
            val indexQuery = new IndexQuery[Product]()
              .withBuilder(QueryBuilders.multiMatchQuery(keyword, "title", "description"))
             search(indexQuery)
          }
        }
    ```

1.  接下来，在`app/controllers/Products.scala`中创建将提供产品创建和搜索的产品端点：

    ```java
    package controllers
         import models.{Product, ProductManager}
        import play.api.libs.json.{JsError, Json}
        import play.api.mvc.{BodyParsers, Action, Controller}
         case class Search(keyword: String)
         object Products extends Controller {
          implicit private val productWrites = Json.writes[Product]
          implicit private val productReads = Json.reads[Product]
          implicit private val searchWrites = Json.writes[Search]
          implicit private val searchReads = Json.reads[Search]
           def create = Action(BodyParsers.parse.json) { implicit request =&gt;
            val post = request.body.validate[Product]
             post.fold(
              errors =&gt; BadRequest(JsError.toFlatJson(errors)),
              p =&gt; {
                ProductManager.index(p)
                Created(Json.toJson(p))
              }
            )
          }
           def search = Action(BodyParsers.parse.json) { implicit request =&gt;
            request.body.validate[Search].fold(
              errors =&gt; BadRequest(JsError.toFlatJson(errors)),
              search =&gt; {
                val results = ProductManager.doSearch(search.keyword)
                Ok(Json.obj(
                  "total" -&gt; results.totalCount,
                  "products" -&gt; results.results
                ))
              }
            )
          }
        }
    ```

1.  最后，让我们将产品控制器操作的路由添加到`conf/routes`中：

    ```java
    POST    /products            controllers.Products.create
        GET     /products/search     controllers.Products.search
    ```

1.  使用`curl`，我们可以如下测试产品创建和索引：

    ```java
    <span class="strong"><strong>    # Let's insert 2 products:</strong></span>
     <span class="strong"><strong>    curl -v -X POST http://localhost:9000/products --header "Content-type: application/json" -d '{"id":"1001", "title":"Intel Core i7-4790K Processor", "shortDesc": "New Unlocked 4th Gen Intel Core Processors deliver 4 cores of up to 4 GHz base frequency providing blazing-fast computing performance for the most demanding users"}'</strong></span>
     <span class="strong"><strong>    curl -v -X POST http://localhost:9000/products --header "Content-type: application/json" -d '{"id":"1002", "title": "AMD FD6300WMHKBOX FX-6300 6-Core Processor", "shortDesc": "AMD FX 6-Core Processor Unlocked Black Edition. AMDs next-generation architecture takes 8-core processing to a new level. Get up to 24% better frame rates in some of the most demanding games at stunning resolutions. Get faster audio encoding so you can enjoy your music sooner. Go up to 5.0 GHz with aggressive cooling solutions from AMD."}'</strong></span>
    ```

1.  我们也可以使用`curl`来执行产品搜索：

    ```java
    <span class="strong"><strong>    $ curl -X GET http://localhost:9000/products/search --header "Content-type: application/json" -d '{"keyword":"processor"}'</strong></span>
    <span class="strong"><strong>{"total":2,"products":[{"id":"1001","title":"Intel Core i7-4790K Processor","shortDesc":"New Unlocked 4th Gen Intel Core Processors deliver 4 cores of up to 4 GHz base frequency providing blazing-fast computing performance for the most demanding users"},{"id":"1002","title":"AMD FD6300WMHKBOX FX-6300 6-Core Processor","shortDesc":"AMD FX 6-Core Processor Unlocked Black Edition. AMDs next-generation architecture takes 8-core processing to a new level. Get up to 24% better frame rates in some of the most demanding games at stunning resolutions. Get faster audio encoding so you can enjoy your music sooner. Go up to 5.0 GHz with aggressive cooling solutions from AMD."}]}%</strong></span>
     <span class="strong"><strong>    $ curl -X GET http://localhost:9000/products/search --header "Content-type: application/json" -d '{"keyword":"amd"}'</strong></span>
    <span class="strong"><strong>{"total":1,"products":[{"id":"1002","title":"AMD FD6300WMHKBOX FX-6300 6-Core Processor","shortDesc":"AMD FX 6-Core Processor Unlocked Black Edition. AMDs next-generation architecture takes 8-core processing to a new level. Get up to 24% better frame rates in some of the most demanding games at stunning resolutions. Get faster audio encoding so you can enjoy your music sooner. Go up to 5.0 GHz with aggressive cooling solutions from AMD."}]}%</strong></span>
    ```

## 如何工作……

在这个菜谱中，我们利用 ElasticSearch 作为底层搜索服务，创建了产品创建和产品搜索的端点。这个菜谱的前提条件是能够访问到 ElasticSearch 实例，无论是本地还是远程，在我们的菜谱中，以及在本地的开发机器上安装了 Docker：

```java
<span class="strong"><strong>    $ docker -v</strong></span>
<span class="strong"><strong>    Docker version 1.3.3, build d344625</strong></span>
```

我们使用以下命令使用 Docker 部署我们的本地 ElasticSearch 实例：

```java
<span class="strong"><strong>    docker run -d -p 9200:9200 -p 9300:9300 dockerfile/elasticsearch</strong></span>
```

上述命令指示`docker`以分离服务的方式运行`elasticsearch`容器，并且容器中的端口`9200`和`9300`应该可以从主机的相应端口访问。

我们首先通过在`build.sbt`文件中声明它来导入开源 play 模块 play2-elasticsearch：

```java
resolvers +=   "Sonatype OSS Snapshots" at "https://oss.sonatype.org/content/repositories/snapshots"
     libraryDependencies ++= Seq(
      "io.iron.ironmq" % "ironmq" % "0.0.19",
      ("com.clever-age" % "play2-elasticsearch" % "1.4-SNAPSHOT")
        .exclude("com.typesafe.play", "play-functional_2.10")
        .exclude("com.typesafe.akka", "akka-actor_2.10")
        .exclude("com.typesafe.play", "play-json_2.10")
        .exclude("com.typesafe.play", "play_2.10")
        .exclude("com.typesafe.play", "play-iteratees_2.10")
        .exclude("com.typesafe.akka", "akka-slf4j_2.10")
        .exclude("org.scala-stm", "scala-stm_2.10")
        .exclude("com.typesafe.play", "play-datacommons_2.10")
    )
```

我们在`conf/play.plugins`中激活插件，并在`conf/application.conf`中指定配置参数：

```java
// conf/play.plugins
    9000:com.github.cleverage.elasticsearch.plugin.IndexPlugin
    // conf/application.conf
    elasticsearch.local=false
    elasticsearch.client="192.168.59.103:9300"
    elasticsearch.sniff=false
    elasticsearch.index.name="test"
    elasticsearch.index.settings="{ analysis: { analyzer: { my_analyzer: { type: \"custom\", tokenizer: \"standard\" } } } }"
    elasticsearch.index.clazzs="models.*"
    elasticsearch.index.show_request=true
    elasticsearch.cluster.name=elasticsearch
```

然后，我们创建了我们的产品模型，它扩展了 play2-elasticsearch 类、Indexable，以及一个搜索管理类，它扩展了 play2-elasticsearch 的 IndexableManager 类。我们编写了一个辅助方法来执行多字段查询，通过关键词搜索标题和描述字段：

```java
// Java 
    IndexQuery&lt;Product&gt; indexQuery = Product.find.query();
    indexQuery.setBuilder(QueryBuilders.multiMatchQuery(keyword, "title", "description"));    
     // Scala
    val indexQuery = new IndexQuery[Product]()
      .withBuilder(QueryBuilders.multiMatchQuery(keyword, "title", "description"))
```

最后，在我们的控制器类`Products`中，我们调用了创建和搜索操作的相关产品管理方法：

```java
// Java
    @BodyParser.Of(BodyParser.Json.class)
    public static Result create() {
     ..
     product.index();
     ..
    }
     @BodyParser.Of(BodyParser.Json.class)
    public static Result search() {
        .. 
        Product.doSearch(search.getKeyword());
        ..
    }    
     // Scala
    def create = Action(BodyParsers.parse.json) { implicit request =&gt;
   ..
      ProductManager.index(p)
      ..
    }
     def search = Action(BodyParsers.parse.json) { implicit request =&gt;
      ..
      val results = ProductManager.doSearch(search.keyword)
      ..
    }
```

当 Web 应用程序启动或重新加载时，您将能够看到有关 play2-elasticsearch 插件初始化的日志信息：

```java
<span class="strong"><strong>[info] application - Elasticsearch : Settings  {client.transport.sniff=false, cluster.name=elasticsearch}</strong></span>
<span class="strong"><strong>[info] application - ElasticSearch : Starting in Client Mode</strong></span>
<span class="strong"><strong>[info] application - ElasticSearch : Client - Host: 192.168.59.103 Port: 9300</strong></span>
<span class="strong"><strong>[info] application - ElasticSearch : Started in Client Mode</strong></span>
<span class="strong"><strong>[info] application - ElasticSearch : Plugin has started</strong></span>
<span class="strong"><strong>[info] play - Application started (Dev)</strong></span>
```

# 实现使用 JWT 的令牌认证

在这个菜谱中，我们将广泛使用的用户认证策略，令牌认证，与认证请求到受保护的 Play 操作和端点集成。我们将使用 Connect2Id 的开源库**nimbus-jose-jwt**来进行登录和验证 JWT 以成功登录用户。

随后的请求到其他受保护的端点和操作现在将仅需要将 JWT 添加到请求头中的授权头。然而，已签名的 JWT 将有一个规定的过期日期，我们将确保为每个签名的 JWT 请求验证这一点。

### 注意

更多关于 Connect2id 和 nimbus-jose-jwt 的信息请在此处查看：

[`connect2id.com/products/nimbus-jose-jwt`](http://connect2id.com/products/nimbus-jose-jwt)

关于 JWT 的更多信息请在此处查看：

[`self-issued.info/docs/draft-ietf-oauth-json-web-token.html`](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html)

## 如何做到这一点…

对于 Java，我们需要执行以下步骤：

1.  以启用热重载的方式运行`foo_java`应用程序：

    ```java
    <span class="strong"><strong>    activator "~run"</strong></span>
    ```

1.  在`build.sbt`中添加`nimbus-jose-jwt`依赖项：

    ```java
    libraryDependencies ++= Seq(
          "com.nimbusds" % "nimbus-jose-jwt" % "3.8.2"
        )
    ```

1.  使用`activator`生成一个新的应用程序密钥，如下所示：

    ```java
    <span class="strong"><strong>    $ activator play-generate-secret</strong></span>
    <span class="strong"><strong>    [info] Loading project definition from /Users/ginduc/Developer/workspace/bitbucket/Play2.0CookbookRecipes/Ch06/foo_java/project</strong></span>
    <span class="strong"><strong>    [info] Set current project to foo_java (in build file:/Users/ginduc/Developer/workspace/bitbucket/Play2.0CookbookRecipes/Ch06/foo_java/)</strong></span>
    <span class="strong"><strong>    [info] Generated new secret: DDqEUkPssmdHOL=U`XMANZAPYG4fUYA5QwGtK49[PmUh2kAH/IpHuHuLIuNgv_o_</strong></span>
    <span class="strong"><strong>    [success] Total time: 0 s, completed 02 24, 15 11:44:42 AM</strong></span>
    ```

1.  使用上一步生成的密钥作为`jwt.sharedSecret`的值，将所需的配置参数添加到`conf/application.conf`中。至于发行者和受众，根据 JWT 规范，发行者是签发令牌的实体，而受众是令牌的预期接收者：

    ```java
    jwt.sharedSecret = "DDqEUkPssmdHOL=U`XMANZAPYG4fUYA5QwGtK49[PmUh2kAH/IpHuHuLIuNgv_o_"
        jwt.issuer=&lt;YOUR_ISSUER&gt;
        jwt.expiryInSecs=600
        jwt.audience=&lt;YOUR_AUDIENCE&gt;
    ```

1.  在`app/plugins/JWTPlugin.java`中创建 JWT 插件类：

    ```java
    package plugins;
         import com.nimbusds.jose.JWSAlgorithm;
        import com.nimbusds.jose.JWSHeader;
        import com.nimbusds.jose.crypto.MACSigner;
        import com.nimbusds.jose.crypto.MACVerifier;
        import com.nimbusds.jwt.JWTClaimsSet;
        import com.nimbusds.jwt.SignedJWT;
        import play.Application;
        import play.Logger;
        import play.Plugin;
        import java.util.Date;
         public class JWTPlugin extends Plugin {
            final private String tokenPrefix = "Bearer ";
             private String issuer;
            private String sharedSecret;
            private Integer expiryTime;
            private String audience;
            private JWSHeader algorithm;
            private MACSigner signer;
            private MACVerifier verifier;
             public JWTPlugin(Application app) {
                super();
                 issuer = app.configuration().getString("jwt.issuer");
                sharedSecret = app.configuration().getString("jwt.sharedSecret");
                expiryTime = app.configuration().getInt("jwt.expiryInSecs");
                audience = app.configuration().getString("jwt.audience");
            }
             public void onStart() {
                algorithm = new JWSHeader(JWSAlgorithm.HS256);
                signer = new MACSigner(sharedSecret);
                verifier = new MACVerifier(sharedSecret);
            }
             public void onStop() {
                algorithm = null;
                signer = null;
                verifier = null;
            }
             public boolean verify(String token) {
                try {
                    final JWTClaimsSet payload = decode(token);
                     // Check expiration date
                    if (!new Date().before(payload.getExpirationTime())) {
                        Logger.error("Token expired: " + payload.getExpirationTime());
                        return false;
                    }
                     // Match Issuer
                    if (!payload.getIssuer().equals(issuer)) {
                        Logger.error("Issuer mismatch: " + payload.getIssuer());
                        return false;
                    }
                     // Match Audience
                    if (payload.getAudience() != null &amp;&amp; payload.getAudience().size() &gt; 0) {
                        if (!payload.getAudience().get(0).equals(audience)) {
                            Logger.error("Audience mismatch: " + payload.getAudience().get(0));
                            return false;
                        }
                    } else {
                        Logger.error("Audience is required");
                        return false;
                    }
                     return true;
                } catch(Exception e) {
                    return false;
                }
            }
             public JWTClaimsSet decode(String token) throws Exception {
                Logger.debug("Verifying: " + token.substring(tokenPrefix.length()));
                SignedJWT signedJWT = SignedJWT.parse(token.substring(tokenPrefix.length()));
                 if (!signedJWT.verify(verifier)) {
                    throw new IllegalArgumentException("Json Web Token cannot be verified!");
                }
                 return (JWTClaimsSet) signedJWT.getJWTClaimsSet();
            }
             public String sign(String userInfo) throws Exception {
                final JWTClaimsSet claimsSet = new JWTClaimsSet();
                claimsSet.setSubject(userInfo);
                claimsSet.setIssueTime(new Date());
                claimsSet.setIssuer(issuer);
                claimsSet.setAudience(audience);
                claimsSet.setExpirationTime(
                    new Date(claimsSet.getIssueTime().getTime() + (expiryTime * 1000))
                );
                 SignedJWT signedJWT = new SignedJWT(algorithm, claimsSet);
                signedJWT.sign(signer);
                return signedJWT.serialize();
            }
             public boolean enabled() {
                return true;
            }
        }
    ```

1.  在`conf/play.plugins`中初始化 JWT 插件：

    ```java
    10099:plugins.JWTPlugin
    ```

1.  在`app/controllers/JWTSigned.java`中创建一个继承自`Simple Action`类的操作类，我们将使用它来使用 JWT 保护操作：

    ```java
    package controllers;
         import play.*;
        import play.mvc.*;
        import play.libs.*;
        import play.libs.F.*;
        import plugins.JWTPlugin;
         public class JWTSigned extends play.mvc.Action.Simple {
            private static final String AUTHORIZATION = "Authorization";
            private static final String WWW_AUTHENTICATE = "WWW-Authenticate";
            private static final String APP_REALM = "Protected Realm";
            private static final String AUTH_HEADER_PREFIX = "Bearer ";
            private static JWTPlugin jwt = Play.application().plugin(JWTPlugin.class);
             public F.Promise&lt;Result&gt; call(Http.Context ctx) throws Throwable {
                try {
                    final String authHeader = ctx.request().getHeader(AUTHORIZATION);
                     if (authHeader != null &amp;&amp; authHeader.startsWith(AUTH_HEADER_PREFIX)) {
                        if (jwt.verify(authHeader)) {
                            return delegate.call(ctx);
                        }
                    } else {
                        return Promise.pure((Result) unauthorized());
                    }
                } catch (Exception e) {
                    Logger.error("Error during session authentication: " + e);
                }
                 ctx.response().setHeader(WWW_AUTHENTICATE, APP_REALM);
                return Promise.pure((Result) forbidden());
            }
        }
    ```

1.  在`app/controllers/Application.java`中创建用于登录和令牌签名的测试操作，以及另一个使用`JWTSigned`进行安全保护的操作：

    ```java
    package controllers;
         import play.*;
        import play.data.Form;
        import play.mvc.*;
         import plugins.JWTPlugin;
        import views.html.*;
         import java.util.HashMap;
        import java.util.Map;
         import static play.libs.Json.toJson;
         public class Application extends Controller {
            private static JWTPlugin jwt = Play.application().plugin(JWTPlugin.class);
             public static Result index() {
                return ok(index.render("Your new application is ready."));
            }
             @With(JWTSigned.class)
            public static Result adminOnly() {
                return ok("");
            }
             @BodyParser.Of(BodyParser.Json.class)
            public static Result auth() {
                try {
                    Form&lt;Login&gt; form = Form.form(Login.class).bindFromRequest();
                    Login login = form.get();
                    if (login.getUsername().equals("ned") &amp;&amp; login.getPassword().equals("flanders")) {
                        final String token = jwt.sign(login.getUsername());
                        final Map&lt;String, String&gt; map = new HashMap&lt;&gt;();
                        map.put("token", token);
                        return ok(toJson(map));
                    } else {
                        return forbidden();
                    }
                 } catch (Exception e) {
                    return internalServerError(e.getMessage());
                }
            }
        }
    ```

1.  我们还需要在`app/controllers/Login.java`中创建用于用户认证的`Login`模型：

    ```java
    package controllers;
         public class Login {
            private String username;
            private String password;
             public Login() {}
            public Login(String username, String password) {
                this.username = username;
                this.password = password;
            }
             public String getUsername() {
                return username;
            }
             public void setUsername(String username) {
                this.username = username;
            }
             public String getPassword() {
                return password;
            }
             public void setPassword(String password) {
                this.password = password;
            }
        }
    ```

1.  最后，我们将必要的条目添加到`conf/routes`中，以支持我们的新操作：

    ```java
    POST    /user/auth           controllers.Application.auth
        GET     /admin               controllers.Application.adminOnly
    ```

1.  使用`curl`验证`/admin`路由是否由`JWTSigned`保护：

    ```java
    <span class="strong"><strong>    $ curl -v http://localhost:9000/admin</strong></span>
    <span class="strong"><strong>    * Hostname was NOT found in DNS cache</strong></span>
    <span class="strong"><strong>    *   Trying ::1...</strong></span>
    <span class="strong"><strong>    * Connected to localhost (::1) port 9000 (#0)</strong></span>
    <span class="strong"><strong>    &gt; GET /admin HTTP/1.1</strong></span>
    <span class="strong"><strong>    &gt; User-Agent: curl/7.37.1</strong></span>
    <span class="strong"><strong>    &gt; Host: localhost:9000</strong></span>
    <span class="strong"><strong>    &gt; Accept: */*</strong></span>
    <span class="strong"><strong>    &gt;</strong></span>
    <span class="strong"><strong>    &lt; HTTP/1.1 401 Unauthorized</strong></span>
    <span class="strong"><strong>    &lt; WWW-Authenticate: Basic realm="Protected Realm"</strong></span>
    <span class="strong"><strong>    &lt; Content-Length: 0</strong></span>
    <span class="strong"><strong>    &lt;</strong></span>
    <span class="strong"><strong>    * Connection #0 to host localhost left intact</strong></span>
    ```

1.  接下来，让我们登录并注意响应体中返回的令牌：

    ```java
    <span class="strong"><strong>    $ curl -v http://localhost:9000/user/auth --header "Content-type: application/json" -d '{"username":"ned", "password":"flanders"}'</strong></span>
    <span class="strong"><strong>    * Hostname was NOT found in DNS cache</strong></span>
    <span class="strong"><strong>    *   Trying ::1...</strong></span>
    <span class="strong"><strong>    * Connected to localhost (::1) port 9000 (#0)</strong></span>
    <span class="strong"><strong>    &gt; POST /user/auth HTTP/1.1</strong></span>
    <span class="strong"><strong>    &gt; User-Agent: curl/7.37.1</strong></span>
    <span class="strong"><strong>    &gt; Host: localhost:9000</strong></span>
    <span class="strong"><strong>    &gt; Accept: */*</strong></span>
    <span class="strong"><strong>    &gt; Content-type: application/json</strong></span>
    <span class="strong"><strong>    &gt; Content-Length: 41</strong></span>
    <span class="strong"><strong>    &gt;</strong></span>
    <span class="strong"><strong>    * upload completely sent off: 41 out of 41 bytes</strong></span>
    <span class="strong"><strong>    &lt; HTTP/1.1 200 OK</strong></span>
    <span class="strong"><strong>    &lt; Content-Type: application/json; charset=utf-8</strong></span>
    <span class="strong"><strong>    &lt; Content-Length: 181</strong></span>
    <span class="strong"><strong>    &lt;</strong></span>
    <span class="strong"><strong>    * Connection #0 to host localhost left intact</strong></span>
    <span class="strong"><strong>{"token":"eyJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0MjQ3MDM0NjAsInN1YiI6Im5lZCIsImF1ZCI6ImFwaWNsaWVudHMiLCJpc3MiOiJwMmMiLCJpYXQiOjE0MjQ3MDM0MDB9.No2skaVfGeERDY6yEMJV8KiRddZsZEcW5BAH2vw99Xc"}%</strong></span>
    ```

1.  最后，让我们再次请求`/admin`路由，但这次，通过在授权头中添加已签名的令牌，前面加上`Bearer`：

    ```java
    <span class="strong"><strong>    $ curl -v http://localhost:9000/admin --header "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0MjQ3MDM0NjAsInN1YiI6Im5lZCIsImF1ZCI6ImFwaWNsaWVudHMiLCJpc3MiOiJwMmMiLCJpYXQiOjE0MjQ3MDM0MDB9.No2skaVfGeERDY6yEMJV8KiRddZsZEcW5BAH2vw99Xc"</strong></span>
    <span class="strong"><strong>    * Hostname was NOT found in DNS cache</strong></span>
    <span class="strong"><strong>    *   Trying ::1...</strong></span>
    <span class="strong"><strong>    * Connected to localhost (::1) port 9000 (#0)</strong></span>
    <span class="strong"><strong>    &gt; GET /admin HTTP/1.1</strong></span>
    <span class="strong"><strong>    &gt; User-Agent: curl/7.37.1</strong></span>
    <span class="strong"><strong>    &gt; Host: localhost:9000</strong></span>
    <span class="strong"><strong>    &gt; Accept: */*</strong></span>
    <span class="strong"><strong>    &gt; Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0MjQ3MDM0NjAsInN1YiI6Im5lZCIsImF1ZCI6ImFwaWNsaWVudHMiLCJpc3MiOiJwMmMiLCJpYXQiOjE0MjQ3MDM0MDB9.No2skaVfGeERDY6yEMJV8KiRddZsZEcW5BAH2vw99Xc</strong></span>
    <span class="strong"><strong>    &gt;</strong></span>
    <span class="strong"><strong>    &lt; HTTP/1.1 200 OK</strong></span>
    <span class="strong"><strong>    &lt; Content-Type: text/plain; charset=utf-8</strong></span>
    <span class="strong"><strong>    &lt; Content-Length: 0</strong></span>
    <span class="strong"><strong>    &lt;</strong></span>
    <span class="strong"><strong>    * Connection #0 to host localhost left intact</strong></span>
    ```

1.  我们还可以通过在令牌中设置过期时间后再次运行之前的请求来验证 JWTPlugin 是否正确处理了令牌过期，它应该导致类似以下的结果：

    ```java
    <span class="strong"><strong>    $ curl -v http://localhost:9000/admin --header "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0MjQ3MDM0NjAsInN1YiI6Im5lZCIsImF1ZCI6ImFwaWNsaWVudHMiLCJpc3MiOiJwMmMiLCJpYXQiOjE0MjQ3MDM0MDB9.No2skaVfGeERDY6yEMJV8KiRddZsZEcW5BAH2vw99Xc"</strong></span>
    <span class="strong"><strong>    * Hostname was NOT found in DNS cache</strong></span>
    <span class="strong"><strong>    *   Trying ::1...</strong></span>
    <span class="strong"><strong>    * Connected to localhost (::1) port 9000 (#0)</strong></span>
    <span class="strong"><strong>    &gt; GET /admin HTTP/1.1</strong></span>
    <span class="strong"><strong>    &gt; User-Agent: curl/7.37.1</strong></span>
    <span class="strong"><strong>    &gt; Host: localhost:9000</strong></span>
    <span class="strong"><strong>    &gt; Accept: */*</strong></span>
    <span class="strong"><strong>    &gt; Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0MjQ3MDM0NjAsInN1YiI6Im5lZCIsImF1ZCI6ImFwaWNsaWVudHMiLCJpc3MiOiJwMmMiLCJpYXQiOjE0MjQ3MDM0MDB9.No2skaVfGeERDY6yEMJV8KiRddZsZEcW5BAH2vw99Xc</strong></span>
    <span class="strong"><strong>    &gt;</strong></span>
    <span class="strong"><strong>    &lt; HTTP/1.1 403 Forbidden</strong></span>
    <span class="strong"><strong>    &lt; Content-Length: 0</strong></span>
    <span class="strong"><strong>    &lt;</strong></span>
    <span class="strong"><strong>    * Connection #0 to host localhost left intact</strong></span>
    ```

对于 Scala，我们需要执行以下步骤：

1.  启用 Hot-Reloading 运行 `foo_scala` 应用程序：

    ```java
    <span class="strong"><strong>    activator "~run"</strong></span>
    ```

1.  将 `nimbus-jose-jwt` 依赖项添加到 `build.sbt` 文件中：

    ```java
    libraryDependencies ++= Seq(
          "com.nimbusds" % "nimbus-jose-jwt" % "3.8.2"
         )
    ```

1.  使用 `activator` 生成新的应用程序密钥，如下所示：

    ```java
    <span class="strong"><strong>    $ activator play-generate-secret</strong></span>
    <span class="strong"><strong>    [info] Loading project definition from /Users/ginduc/Developer/workspace/bitbucket/Play2.0CookbookRecipes/Ch06/foo_scala/project</strong></span>
    <span class="strong"><strong>    [info] Set current project to foo_scala (in build file:/Users/ginduc/Developer/workspace/bitbucket/Play2.0CookbookRecipes/Ch06/foo_scala/)</strong></span>
    <span class="strong"><strong>    [info] Generated new secret: LKQhArck[KCAFcEplWDeBSV^e@a1o6X&gt;2SI_D3Q^X0h`eigla5ywm^k6E9z7Nx=p</strong></span>
    <span class="strong"><strong>    [success] Total time: 0 s, completed 02 23, 15 10:32:56 PM</strong></span>
    ```

1.  使用上一步生成的密钥作为 `jwt.sharedSecret` 的值，将所需的配置参数添加到 `conf/application.conf` 文件中。至于发行者和受众，根据 JWT 规范，发行者是发行令牌的实体，受众是令牌的预期接收者：

    ```java
    jwt.sharedSecret = "LKQhArckKCAFcEplWDeBSV^e@a1o6X&gt;2SI_D3Q^X0h`eigla5ywm^k6E9z7Nx=p"
        jwt.issuer=&lt;YOUR_ISSUER&gt;
        jwt.expiryInSecs=600
        jwt.audience=&lt;YOUR_AUDIENCE&gt;
    ```

1.  在 `app/plugins/JWTPlugin.scala` 中创建 JWT 插件类：

    ```java
    package plugins
         import java.util.Date
        import com.nimbusds.jose.crypto.{MACVerifier, MACSigner}
        import com.nimbusds.jose.{JWSAlgorithm, JWSHeader}
        import com.nimbusds.jwt.{JWTClaimsSet, SignedJWT}
        import play.api.{Logger, Play, Application, Plugin}
        import play.api.Play.current
         class JWTPlugin(app: Application) extends Plugin {
          val tokenPrefix = "Bearer "
           private val issuer = Play.application.configuration.getString("jwt.issuer").getOrElse("jwt")
          private val sharedSecret = Play.application.configuration.getString("jwt.sharedSecret")
        .getOrElse(throw new IllegalStateException("JWT Shared Secret is required!"))
          private val expiryTime = Play.application.configuration.getInt("jwt.expiryInSecs").getOrElse(60 * 60 * 24)
          private val audience = Play.application.configuration.getString("jwt.audience").getOrElse("jwt")
          private val algorithm = new JWSHeader(JWSAlgorithm.HS256)

          private lazy val signer: MACSigner = new MACSigner(sharedSecret)
          private lazy val verifier: MACVerifier = new MACVerifier(sharedSecret)
           override def onStart() = {

           signer
            verifier
          }
           override def onStop() = {
            Logger.info("Shutting down plugin")
          }
           def verify(token: String): Boolean = {
            val payload = decode(token)
             // Check expiration date
            if (!new Date().before(payload.getExpirationTime)) {
              Logger.error("Token expired: " + payload.getExpirationTime)
              return false
            }
             // Match Issuer
            if (!payload.getIssuer.equals(issuer)) {
              Logger.error("Issuer mismatch: " + payload.getIssuer)
              return false
            }
             // Match Audience
            if (payload.getAudience != null &amp;&amp; payload.getAudience.size() &gt; 0) {
              if (!payload.getAudience.get(0).equals(audience)) {
                Logger.error("Audience mismatch: " + payload.getAudience.get(0))
                return false
              }
            } else {
              Logger.error("Audience is required")
              return false
            }
            return true
          }
           def decode(token: String) = {
            val signedJWT = SignedJWT.parse(token.substring(tokenPrefix.length))
             if (!signedJWT.verify(verifier)) {
              throw new IllegalArgumentException("Json Web Token cannot be verified!")
            }
             signedJWT.getJWTClaimsSet
          }
           def sign(userInfo: String): String = {
            val claimsSet = new JWTClaimsSet()
            claimsSet.setSubject(userInfo)
            claimsSet.setIssueTime(new Date)
            claimsSet.setIssuer(issuer)
            claimsSet.setAudience(audience)
            claimsSet.setExpirationTime(
              new Date(claimsSet.getIssueTime.getTime + (expiryTime * 1000))
            )
             val signedJWT = new SignedJWT(algorithm, claimsSet)
            signedJWT.sign(signer)
            signedJWT.serialize()
          }
        override def enabled = true
        }
    ```

1.  在 `conf/play.plugins` 中初始化 JWTPlugin：

    ```java
    10099:plugins.JWTPlugin
    ```

1.  在 `app/controllers/JWTSigned.scala` 中创建一个 `ActionBuilder` 类，我们将使用它来使用 JWT 保护操作：

    ```java
    package controllers
         import play.api.Play
        import play.api.mvc.{Result, WrappedRequest, Request, ActionBuilder}
        import play.api.http.HeaderNames._
        import play.api.mvc.Results._
        import play.api.Play.current
        import plugins.JWTPlugin
        import scala.concurrent.Future
         class JWTSignedRequest[A extends WrappedRequestA
         object JWTSigned extends ActionBuilder[JWTSignedRequest] {
          private val jwt = Play.application.plugin[JWTPlugin].get
           def invokeBlockA =&gt; Future[Result]) = {
            req.headers.get(AUTHORIZATION) map { token =&gt;
              if (jwt.verify(token)) {
                block(new JWTSignedRequest(token, req))
              } else {
                Future.successful(Forbidden)
              }
            } getOrElse {
              Future.successful(Unauthorized.withHeaders(WWW_AUTHENTICATE -&gt; """Basic realm="Protected Realm""""))
            }
          }
        }
    ```

1.  在 `app/controllers/Application.scala` 中创建用于登录和令牌签名的测试操作，以及另一个需要用 `JWTSigned` 保护的操作：

    ```java
    case class Login(username: String, password: String)

        private val jwt = Play.application.plugin[JWTPlugin].get
        implicit private val productWrites = Json.writes[Login]
        implicit private val productReads = Json.reads[Login]
         def adminOnly = JWTSigned {
          Ok("")
        }
         def auth = Action(BodyParsers.parse.json) { implicit request =&gt;
          val post = request.body.validate[Login]
           post.fold(
            errors =&gt; Unauthorized,
            u =&gt; {
              if (u.username.equals("ned") &amp;&amp; u.password.equals("flanders")) {
                 Ok(Json.obj("token" -&gt; jwt.sign(u.username)))
              } else {
                Forbidden
              }
            }
          )
        }
    ```

1.  最后，我们将必要的条目添加到 `conf/routes` 文件中，以支持我们的新操作：

    ```java
    POST    /user/auth           controllers.Application.auth
        GET     /admin               controllers.Application.adminOnly
    ```

1.  使用 `curl` 验证 `/admin` 路由是否由 `JWTSigned` 保护：

    ```java
    <span class="strong"><strong>    $ curl -v http://localhost:9000/admin</strong></span>
    <span class="strong"><strong>    * Hostname was NOT found in DNS cache</strong></span>
    <span class="strong"><strong>    *   Trying ::1...</strong></span>
    <span class="strong"><strong>    * Connected to localhost (::1) port 9000 (#0)</strong></span>
    <span class="strong"><strong>    &gt; GET /admin HTTP/1.1</strong></span>
    <span class="strong"><strong>    &gt; User-Agent: curl/7.37.1</strong></span>
    <span class="strong"><strong>    &gt; Host: localhost:9000</strong></span>
    <span class="strong"><strong>    &gt; Accept: */*</strong></span>
    <span class="strong"><strong>    &gt;</strong></span>
    <span class="strong"><strong>    &lt; HTTP/1.1 401 Unauthorized</strong></span>
    <span class="strong"><strong>    &lt; WWW-Authenticate: Basic realm="Protected Realm"</strong></span>
    <span class="strong"><strong>    &lt; Content-Length: 0</strong></span>
    <span class="strong"><strong>    &lt;</strong></span>
    <span class="strong"><strong>    * Connection #0 to host localhost left intact</strong></span>
    ```

1.  接下来，让我们登录并记录响应体中返回的令牌：

    ```java
    <span class="strong"><strong>    $ curl -v http://localhost:9000/user/auth --header "Content-type: application/json" -d '{"username":"ned", "password":"flanders"}'</strong></span>
    <span class="strong"><strong>    * Hostname was NOT found in DNS cache</strong></span>
    <span class="strong"><strong>    *   Trying ::1...</strong></span>
    <span class="strong"><strong>    * Connected to localhost (::1) port 9000 (#0)</strong></span>
    <span class="strong"><strong>    &gt; POST /user/auth HTTP/1.1</strong></span>
    <span class="strong"><strong>    &gt; User-Agent: curl/7.37.1</strong></span>
    <span class="strong"><strong>    &gt; Host: localhost:9000</strong></span>
    <span class="strong"><strong>    &gt; Accept: */*</strong></span>
    <span class="strong"><strong>    &gt; Content-type: application/json</strong></span>
    <span class="strong"><strong>    &gt; Content-Length: 41</strong></span>
    <span class="strong"><strong>    &gt;</strong></span>
    <span class="strong"><strong>    * upload completely sent off: 41 out of 41 bytes</strong></span>
    <span class="strong"><strong>    &lt; HTTP/1.1 200 OK</strong></span>
    <span class="strong"><strong>    &lt; Content-Type: application/json; charset=utf-8</strong></span>
    <span class="strong"><strong>    &lt; Content-Length: 181</strong></span>
    <span class="strong"><strong>    &lt;</strong></span>
    <span class="strong"><strong>    * Connection #0 to host localhost left intact</strong></span>
    <span class="strong"><strong>{"token":"eyJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0MjQ3MDM0NjAsInN1YiI6Im5lZCIsImF1ZCI6ImFwaWNsaWVudHMiLCJpc3MiOiJwMmMiLCJpYXQiOjE0MjQ3MDM0MDB9.No2skaVfGeERDY6yEMJV8KiRddZsZEcW5BAH2vw99Xc"}%</strong></span>
    ```

1.  最后，让我们再次请求 `/admin` 路由，但这次，在授权头中添加已签名的令牌，前缀为 `Bearer`：

    ```java
    <span class="strong"><strong>    $ curl -v http://localhost:9000/admin --header "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0MjQ3MDM0NjAsInN1YiI6Im5lZCIsImF1ZCI6ImFwaWNsaWVudHMiLCJpc3MiOiJwMmMiLCJpYXQiOjE0MjQ3MDM0MDB9.No2skaVfGeERDY6yEMJV8KiRddZsZEcW5BAH2vw99Xc"</strong></span>
    <span class="strong"><strong>    * Hostname was NOT found in DNS cache</strong></span>
    <span class="strong"><strong>    *   Trying ::1...</strong></span>
    <span class="strong"><strong>    * Connected to localhost (::1) port 9000 (#0)</strong></span>
    <span class="strong"><strong>    &gt; GET /admin HTTP/1.1</strong></span>
    <span class="strong"><strong>    &gt; User-Agent: curl/7.37.1</strong></span>
    <span class="strong"><strong>    &gt; Host: localhost:9000</strong></span>
    <span class="strong"><strong>    &gt; Accept: */*</strong></span>
    <span class="strong"><strong>    &gt; Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0MjQ3MDM0NjAsInN1YiI6Im5lZCIsImF1ZCI6ImFwaWNsaWVudHMiLCJpc3MiOiJwMmMiLCJpYXQiOjE0MjQ3MDM0MDB9.No2skaVfGeERDY6yEMJV8KiRddZsZEcW5BAH2vw99Xc</strong></span>
    <span class="strong"><strong>    &gt;</strong></span>
    <span class="strong"><strong>    &lt; HTTP/1.1 200 OK</strong></span>
    <span class="strong"><strong>    &lt; Content-Type: text/plain; charset=utf-8</strong></span>
    <span class="strong"><strong>    &lt; Content-Length: 0</strong></span>
    <span class="strong"><strong>    &lt;</strong></span>
    <span class="strong"><strong>    * Connection #0 to host localhost left intact</strong></span>
    ```

1.  我们也可以通过在令牌过期后再次运行之前的请求来验证 JWTPlugin 是否正确处理令牌过期，应该得到类似以下的结果：

    ```java
    <span class="strong"><strong>    $ curl -v http://localhost:9000/admin --header "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0MjQ3MDM0NjAsInN1YiI6Im5lZCIsImF1ZCI6ImFwaWNsaWVudHMiLCJpc3MiOiJwMmMiLCJpYXQiOjE0MjQ3MDM0MDB9.No2skaVfGeERDY6yEMJV8KiRddZsZEcW5BAH2vw99Xc"</strong></span>
    <span class="strong"><strong>    * Hostname was NOT found in DNS cache</strong></span>
    <span class="strong"><strong>    *   Trying ::1...</strong></span>
    <span class="strong"><strong>    * Connected to localhost (::1) port 9000 (#0)</strong></span>
    <span class="strong"><strong>    &gt; GET /admin HTTP/1.1</strong></span>
    <span class="strong"><strong>    &gt; User-Agent: curl/7.37.1</strong></span>
    <span class="strong"><strong>    &gt; Host: localhost:9000</strong></span>
    <span class="strong"><strong>    &gt; Accept: */*</strong></span>
    <span class="strong"><strong>    &gt; Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0MjQ3MDM0NjAsInN1YiI6Im5lZCIsImF1ZCI6ImFwaWNsaWVudHMiLCJpc3MiOiJwMmMiLCJpYXQiOjE0MjQ3MDM0MDB9.No2skaVfGeERDY6yEMJV8KiRddZsZEcW5BAH2vw99Xc</strong></span>
    <span class="strong"><strong>    &gt;</strong></span>
    <span class="strong"><strong>    &lt; HTTP/1.1 403 Forbidden</strong></span>
    <span class="strong"><strong>    &lt; Content-Length: 0</strong></span>
    <span class="strong"><strong>    &lt;</strong></span>
    <span class="strong"><strong>    * Connection #0 to host localhost left intact</strong></span>
    ```

## 它是如何工作的…

在这个菜谱中，我们实现了 JWT 的签名和验证，目的是使用令牌身份验证来保护 Play 操作。

我们创建了一个 Play 插件，`JWTPlugin`，它将从 `conf/application.conf` 文件中加载配置，并包含使用 Connect2id 的 nimbus-jose-jwt 库进行签名、解码和验证 JWT 的方法定义：

```java
// Java 
    public JWTPlugin(Application app) {
        super();
        issuer = app.configuration().getString("jwt.issuer");
        sharedSecret = app.configuration().getString("jwt.sharedSecret");
        expiryTime = app.configuration().getInt("jwt.expiryInSecs");
        audience = app.configuration().getString("jwt.audience");
    }
     public void onStart() {
        algorithm = new JWSHeader(JWSAlgorithm.HS256);
        signer = new MACSigner(sharedSecret);
        verifier = new MACVerifier(sharedSecret);
    }    
     // Scala
    private val issuer = Play.application.configuration.getString("jwt.issuer").getOrElse("jwt")
    private val sharedSecret = Play.application.configuration.getString("jwt.sharedSecret")
    .getOrElse(throw new IllegalStateException("JWT Shared Secret is required!"))
    private val expiryTime = Play.application.configuration.getInt("jwt.expiryInSecs").getOrElse(60 * 60 * 24)
    private val audience = Play.application.configuration.getString("jwt.audience").getOrElse("jwt")
    private val algorithm = new JWSHeader(JWSAlgorithm.HS256)
    private var signer: MACSigner = null
    private var verifier: MACVerifier = null
     override def onStart() = {
      signer = new MACSigner(sharedSecret)
      verifier = new MACVerifier(sharedSecret)
    }
```

在前面的代码中，你会注意到我们正在使用 SHA-256 作为默认的哈希算法来利用 HMAC。

对于 Java，添加以下代码：

```java
// Java
    algorithm = new JWSHeader(JWSAlgorithm.HS256);
```

对于 Scala，添加以下代码：

```java
// Scala
    private val algorithm = new JWSHeader(JWSAlgorithm.HS256)
```

对于签名，我们构建了声明集，这是根据 JWT 规范的标准令牌元数据集：你可以参考以下链接：

[`self-issued.info/docs/draft-ietf-oauth-json-web-token.html#Claims`](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html#Claims)

使用适当的参数值，并将用户信息作为声明主题添加，然后我们对其进行签名并将其序列化为 `String`：

```java
// Java 
    public String sign(String userInfo) throws Exception {
        final JWTClaimsSet claimsSet = new JWTClaimsSet();
        claimsSet.setSubject(userInfo);
        claimsSet.setIssueTime(new Date());
        claimsSet.setIssuer(issuer);
        claimsSet.setAudience(audience);
        claimsSet.setExpirationTime(
            new Date(claimsSet.getIssueTime().getTime() + (expiryTime * 1000))
        );
         SignedJWT signedJWT = new SignedJWT(algorithm, claimsSet);
        signedJWT.sign(signer);
        return signedJWT.serialize();
    }    
     // Scala
    def sign(userInfo: String): String = {
      val claimsSet = new JWTClaimsSet()
      claimsSet.setSubject(userInfo)
      claimsSet.setIssueTime(new Date)
      claimsSet.setIssuer(issuer)
      claimsSet.setAudience(audience)
      claimsSet.setExpirationTime(
        new Date(claimsSet.getIssueTime.getTime + (expiryTime * 1000))
      )
       val signedJWT = new SignedJWT(algorithm, claimsSet)
      signedJWT.sign(signer)
      signedJWT.serialize()
    }
```

为了验证，我们获取传递的已解码和验证的令牌，然后继续验证声明集的部分。只有当令牌通过所有验证测试时，它才返回布尔值 `true`：

```java
// Java
    public boolean verify(String token) {
        try {
            final JWTClaimsSet payload = decode(token);
             // Check expiration date
            if (!new Date().before(payload.getExpirationTime())) {
                Logger.error("Token expired: " + payload.getExpirationTime());
                return false;
            }
             // Match Issuer
            if (!payload.getIssuer().equals(issuer)) {
                Logger.error("Issuer mismatch: " + payload.getIssuer());
                return false;
            }
             // Match Audience
            if (payload.getAudience() != null &amp;&amp; payload.getAudience().size() &gt; 0) {
                if (!payload.getAudience().get(0).equals(audience)) {
                    Logger.error("Audience mismatch: " + payload.getAudience().get(0));
                    return false;
                }
            } else {
                Logger.error("Audience is required");
                return false;
            }
             return true;
        } catch(Exception e) {
            return false;
        }
    }
     // Scala
    def verify(token: String): Boolean = {
      val payload = decode(token)
       // Check expiration date
      if (!new Date().before(payload.getExpirationTime)) {
        Logger.error("Token expired: " + payload.getExpirationTime)
        return false
      }
       // Match Issuer
      if (!payload.getIssuer.equals(issuer)) {
        Logger.error("Issuer mismatch: " + payload.getIssuer)
        return false
      }
       // Match Audience
      if (payload.getAudience != null &amp;&amp; payload.getAudience.size() &gt; 0) {
        if (!payload.getAudience.get(0).equals(audience)) {
          Logger.error("Audience mismatch: " + payload.getAudience.get(0))
          return false
        }
      } else {
        Logger.error("Audience is required")
        return false
      }
       return true
    }
```

然后，我们创建了一个 `Simple Action` / `Action Builder` 类，它将执行实际的 JWT 验证：

```java
// Java 
    public F.Promise&lt;Result&gt; call(Http.Context ctx) throws Throwable {
        try {
            final String authHeader = ctx.request().getHeader(AUTHORIZATION);
             if (authHeader != null &amp;&amp; authHeader.startsWith(AUTH_HEADER_PREFIX)) {
                if (jwt.verify(authHeader)) {
                    return delegate.call(ctx);
                }
            }
        } catch (Exception e) {
            Logger.error("Error during session authentication: " + e);
        }
         ctx.response().setHeader(WWW_AUTHENTICATE, APP_REALM);
        return Promise.pure((Result) forbidden());
    }
     // Scala
    def invokeBlockA =&gt; Future[Result]) = {
      req.headers.get(AUTHORIZATION) map { token =&gt;
        if (jwt.verify(token)) {
          block(new JWTSignedRequest(token, req))
        } else {
          Future.successful(Forbidden)
        }
      } getOrElse {
        Future.successful(Unauthorized.withHeaders(WWW_AUTHENTICATE -&gt; """Basic realm="Protected Realm""""))
      }
    }
```

前面的代码只有在令牌通过验证测试的情况下才会调用下一个请求块。对于验证失败的情况，它将返回一个**Http 状态禁止**错误，对于没有设置授权头的请求，则返回一个**Http 状态未授权**错误。

我们现在可以使用 JWTSigned `ActionBuilder` 类来保护 Play 控制器操作：

```java
// Java 
    @With(JWTSigned.class)
    public static Result adminOnly() {
        return ok("");
    }
     // Scala
    def adminOnly = JWTSigned {
      Ok("")
    }
```

最后，我们得到的是一个返回序列化、签名 JWT 的 Play 动作，以及一个使用 JWTSigned `ActionBuilder` 类来保护操作免受未经认证和未经授权请求的 Play 动作：

```java
<span class="strong"><strong>    # Signing    </strong></span>
 <span class="strong"><strong>    $ curl http://localhost:9000/user/auth --header "Content-type: application/json" -d '{"username":"ned", "password":"flanders"}'</strong></span>
<span class="strong"><strong>{"token":"eyJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0MjQ3MDUzOTgsInN1YiI6Im5lZCIsImF1ZCI6ImFwaWNsaWVudHMiLCJpc3MiOiJwMmMiLCJpYXQiOjE0MjQ3MDUzMzh9.uE5GNQv2uXQh29sHhy_Jbg9omDhQMrnW1tjqFBrUwSs"}%</strong></span>
 <span class="strong"><strong>    # Verifying</strong></span>
<span class="strong"><strong>    $ curl -v http://localhost:9000/admin</strong></span>
<span class="strong"><strong>    * Hostname was NOT found in DNS cache</strong></span>
<span class="strong"><strong>    *   Trying ::1...</strong></span>
<span class="strong"><strong>    * Connected to localhost (::1) port 9000 (#0)</strong></span>
<span class="strong"><strong>    &gt; GET /admin HTTP/1.1</strong></span>
<span class="strong"><strong>    &gt; User-Agent: curl/7.37.1</strong></span>
<span class="strong"><strong>    &gt; Host: localhost:9000</strong></span>
<span class="strong"><strong>    &gt; Accept: */*</strong></span>
<span class="strong"><strong>    &gt;</strong></span>
<span class="strong"><strong>    &lt; HTTP/1.1 401 Unauthorized</strong></span>
<span class="strong"><strong>    &lt; WWW-Authenticate: Basic realm="Protected Realm"</strong></span>
<span class="strong"><strong>    &lt; Content-Length: 0</strong></span>
<span class="strong"><strong>    &lt;</strong></span>
<span class="strong"><strong>    * Connection #0 to host localhost left intact</strong></span>
```
