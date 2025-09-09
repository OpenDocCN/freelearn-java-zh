# 第五章。创建插件和模块：

在本章中，我们将介绍以下内容：

+   创建并使用您自己的插件：

+   构建灵活的注册模块

+   使用相同的模型为不同的应用程序服务

+   管理模块依赖关系：

+   使用 Amazon S3 添加私有模块仓库：

# 简介

在本章中，我们将探讨如何将我们的 Play 2.0 网络应用程序分解为模块化、可重用的组件。我们将探讨如何将插件和模块作为 Play 2.0 子项目以及作为独立模块发布到内部模块仓库：

在创建独立服务并初始化共享资源（如数据库连接和 Akka actor 引用）时，Play 2.0 插件非常有用。其他有用的 Play 插件示例包括 **play.i18n.MessagesPlugin**，它管理文本的国际化和 **play.api.db.DBPlugin**，它抽象了 Play 网络应用程序如何连接和与数据库接口：

Play 2.0 模块对于创建较大应用程序的较小、逻辑子组件非常有用；这促进了更好的代码维护和测试的隔离：

# 创建并使用您自己的插件：

在本食谱中，我们将探讨如何使用用于监视指定文件的 Play 2.0 插件。我们将初始化我们的插件作为 Play 网络应用程序生命周期的一部分，并且主要的插件逻辑将在应用程序启动时触发。

## 如何做到这一点……

对于 Java，我们需要执行以下步骤：

1.  启用 Hot-Reloading 功能运行 `foo_java` 应用程序：

    ```java
    <span class="strong"><strong>    activator "~run"</strong></span>
    ```

1.  在 `foo_java` 内创建 modules 目录：

    ```java
    <span class="strong"><strong>    mkdir modules</strong></span>
    ```

1.  在 `foo_java/modules` 内为我们的第一个插件生成项目目录：

    ```java
    <span class="strong"><strong>    activator new filemon play-java</strong></span>
    ```

1.  移除 `modules/filemon/conf/application.conf` 文件的内容，因为这些设置将与我们在项目根目录中定义的主要配置文件冲突：

    ```java
    echo "" &gt; modules/filemon/conf/application.conf
    ```

1.  移除 `modules/filemon/conf/routes` 文件的内容，并将其重命名为 `filemon.routes`：

    ```java
    echo "" &gt; modules/filemon/conf/routes &amp;&amp; mv modules/filemon/conf/routes modules/filemon/conf/filemon.routes
    ```

1.  从 `modules/filemon/app` 中移除 views 目录：

    ```java
    <span class="strong"><strong>    rm -rf modules/filemon/app/views</strong></span>
    ```

1.  使用以下命令移除 `modules/filemon/app/controller/Application.java` 文件：

    ```java
    <span class="strong"><strong>    rm modules/filemon/app/controllers/Application.java</strong></span>
    ```

1.  在 `modules/filemon/app`/ 内创建一个新的包：

    ```java
    <span class="strong"><strong>    mkdir modules/filemon/app/filemon</strong></span>
    ```

1.  在 `modules/filemon/app/FileMonitor.java` 中创建 `FileMonitor` 插件，内容如下：

    ```java
    package filemon;
         import java.io.*;
        import java.util.concurrent.TimeUnit;
        import akka.actor.ActorSystem;
        import play.Plugin;
        import play.Application;
        import play.libs.Akka;
        import scala.concurrent.duration.Duration;
         public class FileMonitor extends Plugin {
            private Application app;
            private ActorSystem actorSystem = ActorSystem.create("filemon");
            private File file = new File("/var/tmp/foo");
             public FileMonitor(Application app) {
                this.app = app;
            }
           @Override
            public void onStart() {
                actorSystem.scheduler().schedule(
                    Duration.create(0, TimeUnit.SECONDS),
                    Duration.create(1, TimeUnit.SECONDS),
                    new Runnable() {
                        public void run() {
                            if (file.exists()) {
                                System.out.println(file.toString() + " exists..");
                            } else {
                                System.out.println(file.toString() + " does not exist..");    
                            }
                        }
                    },
                    Akka.system().dispatcher()
                );
            }
           @Override
            public void onStop() {
                actorSystem.shutdown();
            }
           @Override
            public boolean enabled() {
                return true;
            }
        }
    ```

1.  通过创建插件配置文件 `foo_java/conf/play.plugins` 并在其中声明我们的插件来启用 `foo_java` 应用程序中的插件：

    ```java
    echo "1001:filemon.FileMonitor" &gt; conf/play.plugins
    ```

1.  在 `build.sbt` 中添加根项目 (`foo_java`) 和模块 (`filemon`) 之间的依赖关系，并添加 `aggregate()` 设置以确保从项目根目录 `foo_java` 调用的 activator 任务也会在子模块 `filemon` 上执行：

    ```java
    lazy val root = (project in file("."))
          .enablePlugins(PlayJava)
          .aggregate(filemon)
          .dependsOn(filemon)
         lazy val filemon = (project in file("modules/filemon"))
          .enablePlugins(PlayJava)
    ```

1.  启动 `foo_java` 应用程序：

    ```java
    <span class="strong"><strong>    activator clean "~run"</strong></span>
    ```

1.  请求默认路由并初始化我们的应用程序：

    ```java
    <span class="strong"><strong>    $ curl -v http://localhost:9000</strong></span>
    ```

1.  通过查看 `foo_java` 应用程序的控制台日志来确认文件监视器正在运行：

    ```java
    <span class="strong"><strong>(Server started, use Ctrl+D to stop and go back to the console...)</strong></span>
     <span class="strong"><strong>[info] Compiling 5 Scala sources and 1 Java source to ...</strong></span>
    <span class="strong"><strong>[success] Compiled in 4s</strong></span>
    <span class="strong"><strong>Starting file mon</strong></span>
    <span class="strong"><strong>/var/tmp/foo not found..</strong></span>
    <span class="strong"><strong>[info] play - Application started (Dev)</strong></span>
    <span class="strong"><strong>/var/tmp/foo not found..</strong></span>
    <span class="strong"><strong>/var/tmp/foo not found..</strong></span>
    <span class="strong"><strong>/var/tmp/foo not found..</strong></span>
    <span class="strong"><strong>/var/tmp/foo not found.. </strong></span>
    ```

对于 Scala，我们需要执行以下步骤：

1.  启用 Hot-Reloading 功能运行 `foo_scala` 应用程序：

    ```java
    <span class="strong"><strong>    activator "~run"</strong></span>
    ```

1.  在`foo_scala`内部创建模块目录：

    ```java
    <span class="strong"><strong>    mkdir modules</strong></span>
    ```

1.  在`foo_scala/modules`内部为我们的第一个插件生成项目目录：

    ```java
    <span class="strong"><strong>    activator new filemon play-scala</strong></span>
    ```

1.  删除`modules/filemon/conf/application.conf`文件的内容：

    ```java
    <span class="strong"><strong>    echo "" &gt; modules/filemon/conf/application.conf</strong></span>
    ```

1.  删除`modules/filemon/conf/routes`文件的内容，并将其重命名为`filemon.routes`：

    ```java
    echo "" &gt; modules/filemon/conf/routes &amp;&amp; mv modules/filemon/conf/routes  modules/filemon/conf/filemon.routes
    ```

1.  从`modules/filemon/app`中删除视图目录：

    ```java
    <span class="strong"><strong>    rm -rf modules/filemon/app/views</strong></span>
    ```

1.  使用以下命令删除文件`modules/filemon/app/controller/Application.scala`：

    ```java
    <span class="strong"><strong>    rm modules/filemon/app/controllers/Application.scala</strong></span>
    ```

1.  在`modules/filemon/app/`内部创建一个新的包：

    ```java
    <span class="strong"><strong>    mkdir modules/filemon/app/filemon</strong></span>
    ```

1.  在`modules/filemon/app/FileMonitor.scala`内部创建`FileMonitor`插件，内容如下：

    ```java
    package filemon
         import java.io.File
        import scala.concurrent.duration._
        import akka.actor.ActorSystem
        import play.api.{Plugin, Application}
        import play.api.libs.concurrent.Execution.Implicits._
         class FileMonitor(app: Application) extends Plugin {
          val system = ActorSystem("filemon")
          val file = new File("/var/tmp/foo")
           override def onStart() = {
            println("Starting file mon")
            system.scheduler.schedule(0 second, 1 second) {
              if (file.exists()) {
                println("%s exists..".format(file))
              } else {
                println("%s not found..".format(file))
              }
            }
          }
           override def onStop() = {
            println("Stopping file mon")
            system.shutdown()
          }
           override def enabled = true
        }
    ```

1.  通过创建插件配置文件`foo_scala/conf/play.plugins`并在其中声明我们的插件来从`foo_scala`应用程序中启用插件：

    ```java
    <span class="strong"><strong>    echo "1001:filemon.FileMonitor" &gt; conf/play.plugins</strong></span>
    ```

1.  在`build.sbt`中添加根项目（`foo_scala`）和模块（`filemon`）之间的依赖关系，并添加`aggregate()`设置以确保从项目根`foo_java`调用的 activator 任务也会在子模块`filemon`上执行：

    ```java
    lazy val root = (project in file("."))
          .enablePlugins(PlayScala)
          .aggregate(filemon)
          .dependsOn(filemon)
         lazy val filemon = (project in file("modules/filemon"))
          .enablePlugins(PlayScala)
    ```

1.  启动`foo_scala`应用程序：

    ```java
    <span class="strong"><strong>    activator clean "~run"</strong></span>
    ```

1.  请求我们的默认路由并初始化我们的应用程序：

    ```java
    <span class="strong"><strong>    $ curl -v http://localhost:9000</strong></span>
    ```

1.  通过查看`foo_scala`应用程序的控制台日志来确认文件监控器正在运行：

    ```java
    <span class="strong"><strong>(Server started, use Ctrl+D to stop and go back to the console...)</strong></span>
     <span class="strong"><strong>[info] Compiling 5 Scala sources and 1 Java source to ...</strong></span>
    <span class="strong"><strong>[success] Compiled in 4s</strong></span>
    <span class="strong"><strong>Starting file mon</strong></span>
    <span class="strong"><strong>/var/tmp/foo not found..</strong></span>
    <span class="strong"><strong>[info] play - Application started (Dev)</strong></span>
    <span class="strong"><strong>/var/tmp/foo not found..</strong></span>
    <span class="strong"><strong>/var/tmp/foo not found..</strong></span>
    <span class="strong"><strong>/var/tmp/foo not found..</strong></span>
    <span class="strong"><strong>/var/tmp/foo not found.. </strong></span>
    ```

## 它是如何工作的...

在这个菜谱中，我们设置了我们的第一个 Play 2.0 插件。该插件简单地检查本地文件系统中是否存在文件或目录，并在控制台日志中记录文件是否被找到。我们通过在项目根模块`foo_java/modules`和`foo_scala/modules`中创建插件项目来设置我们的插件：

```java
<span class="strong"><strong>    $ ls</strong></span>
<span class="strong"><strong>    LICENSE                     conf</strong></span>
<span class="strong"><strong>    README                      logs</strong></span>
<span class="strong"><strong>    activator                   modules</strong></span>
<span class="strong"><strong>    activator-launch-1.2.10.jar project</strong></span>
<span class="strong"><strong>    activator.bat               public</strong></span>
<span class="strong"><strong>    app                         target</strong></span>
<span class="strong"><strong>    build.sbt                   test</strong></span>
 <span class="strong"><strong>    $ ls modules/filemon</strong></span>
<span class="strong"><strong>    LICENSE                     build.sbt</strong></span>
<span class="strong"><strong>    README                      conf</strong></span>
<span class="strong"><strong>    activator                   project</strong></span>
<span class="strong"><strong>    activator-launch-1.2.10.jar public</strong></span>
<span class="strong"><strong>    activator.bat               target</strong></span>
<span class="strong"><strong>    app                         test</strong></span>
```

一旦创建了插件项目，我们需要删除一些样板文件和配置，以确保插件不会与根项目`foo_java`和`foo_scala`冲突：

然后，我们在`modules/filemon/app/filemon/FileMonitor.scala`中创建了`FileMonitor`插件，它扩展了`play.api.Plugin`特质，在启动时创建一个计划任务，该任务每隔一秒检查本地文件是否存在：

```java
override def onStart() = {
      println("Starting file mon")
      system.scheduler.schedule(0 second, 1 second) {
        if (file.exists()) {
          println("%s exists..".format(file))
        } else {
          println("%s not found..".format(file))
        }
      }
    }
```

一旦我们设置了插件，我们通过在根项目`foo_java`和`foo_scala`的`conf/play.plugins`文件中声明它来激活它，该文件遵循`<优先级级别>:<插件>`的表示法：

```java
1001:filemon.FileMonitor
```

在我们的案例中，我们使用了`1001`作为优先级级别，以确保 Akka Play 2.0 插件首先加载。请参考官方 Play 文档以获取在`play.plugins`配置文件中声明您的插件的额外指南：

[`www.playframework.com/documentation/2.3.x/JavaPlugins`](https://www.playframework.com/documentation/2.3.x/JavaPlugins)

[`www.playframework.com/documentation/2.3.x/ScalaPlugins`](https://www.playframework.com/documentation/2.3.x/ScalaPlugins)

最后，我们运行了 Web 应用程序，并通过查看控制台日志来确认我们的插件正在运行：

```java
<span class="strong"><strong>    Starting file mon</strong></span>
<span class="strong"><strong>    /var/tmp/foo not found..</strong></span>
```

您可以通过创建或删除监控文件来确认您插件的运行行为，在这个例子中，是`/var/tmp/foo`：

```java
<span class="strong"><strong>    # Create foo file then delete after 3 seconds</strong></span>
<span class="strong"><strong>    touch /var/tmp/foo &amp;&amp; sleep 3 &amp;&amp; rm /var/tmp/foo</strong></span>
```

您将看到日志输出相应地改变：

```java
<span class="strong"><strong>    /var/tmp/foo not found..</strong></span>
<span class="strong"><strong>    /var/tmp/foo not found..</strong></span>
<span class="strong"><strong>    /var/tmp/foo exists..</strong></span>
<span class="strong"><strong>    /var/tmp/foo exists..</strong></span>
<span class="strong"><strong>    /var/tmp/foo exists..</strong></span>
<span class="strong"><strong>    /var/tmp/foo not found..</strong></span>
<span class="strong"><strong>    /var/tmp/foo not found..</strong></span>
```

# 构建一个灵活的注册模块

在这个菜谱中，我们将创建一个新的注册模块，该模块将管理用户注册和身份验证请求。创建这样一个模块允许我们在现代 Web 应用程序中重用一个非常常见的流程。

## 如何做到这一点…

对于 Java，我们需要执行以下步骤：

1.  使用 Hot-Reloading 启用运行`foo_java`应用程序：

    ```java
    <span class="strong"><strong>    activator "~run"</strong></span>
    ```

1.  在模块目录内，`foo_java/modules`中，使用`activator`生成注册模块项目：

    ```java
    <span class="strong"><strong>    activator new registration play-java</strong></span>
    ```

1.  在`foo_java/build.sbt`中添加根项目`foo_java`和模块`registration`之间的依赖关系：

    ```java
    lazy val root = (project in file("."))
          .enablePlugins(PlayJava)
          .aggregate(filemon)
          .dependsOn(filemon)
    <span class="strong"><strong>      .aggregate(registration)</strong></span>
    <span class="strong"><strong>      .dependsOn(registration)</strong></span>
         lazy val filemon = (project in file("modules/filemon"))
          .enablePlugins(PlayJava)
     <span class="strong"><strong>    lazy val registration = (project in file("modules/registration"))</strong></span>
    <span class="strong"><strong>      .enablePlugins(PlayJava)</strong></span>
    ```

1.  从注册模块中移除所有不必要的样板文件和配置：

    ```java
    <span class="strong"><strong>    rm -rf app/views app/controllers</strong></span>
    <span class="strong"><strong>    rm conf/routes</strong></span>
    <span class="strong"><strong>    mkdir app/registration</strong></span>
    <span class="strong"><strong>    echo "" &gt; conf/application.conf</strong></span>
    ```

1.  在`module/registration/app/registration/RegistrationPlugin.java`中创建以下内容的注册插件：

    ```java
    package registration;
         import play.Application;
        import play.Plugin;
         public class RegistrationPlugin extends Plugin {
            private Application app;
            private RegistrationService registrationService;
             public RegistrationPlugin(Application app) {
                this.app = app;
            }
           @Override
            public void onStart() {
                registrationService = new RegistrationServiceImpl();
                registrationService.init();
            }
            @Override
            public void onStop() {
                registrationService.shutdown();
            }
           @Override
            public boolean enabled() {
                return true;
            }
             public RegistrationService getRegistrationService() {
                return registrationService;
            }
        }
    ```

1.  接下来，创建由`Registration`插件引用的`RegistrationService`接口和实现类：

    ```java
    // In modules/registration/app/registration/RegistrationService.java
         package registration;
         public interface RegistrationService {
            void init();
            void shutdown();
            void create(User user);
            Boolean auth(String username, String password);
        }
         // In modules/registration/app/registration/RegistrationServiceImpl.java    
        package registration;
         import java.util.LinkedHashMap;
        import java.util.Map;
        import java.util.UUID;
         public class RegistrationServiceImpl implements RegistrationService {
            private Map&lt;String, User&gt; registrations;
             @Override
            public void create(User user) {
                final String id = UUID.randomUUID().toString();
                registrations.put(id, new User(id, user.getUsername(), user.getPassword()));
            }
             @Override
            public Boolean auth(String username, String password) {
                for(Map.Entry&lt;String, User&gt; entry : registrations.entrySet()) {
                    if (entry.getValue().getUsername().equals(username) &amp;&amp;
                        entry.getValue().getPassword().equals(password)) {
                        return true;
                    }
                }
                return false;
            }
             @Override
            public void init() {
                registrations = new LinkedHashMap&lt;String, User&gt;();
            }
             @Override
            public void shutdown() {
                registrations.clear();
            }
        }
    ```

1.  在`modules/registration/app/registration/User.java`中创建`User`模型实体：

    ```java
    package registration;
         public class User {
            private String id;
            private String username;
            private String password;
             public User() {}
            public User(String id, String username, String password) {
                this.id = id;
                this.username = username;
                this.password = password;
            }
             public String getId() {
                return id;
            }
            public void setId(String id) {
                this.id = id;
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

1.  在项目根目录，`foo/java/app/controllers/Registrations.java`中创建处理注册和登录请求的`Registration`控制器和路由：

    ```java
    package controllers;
         import play.Play;
        import play.data.Form;
        import play.mvc.BodyParser;
        import play.mvc.Controller;
        import play.mvc.Result;
        import registration.RegistrationPlugin;
        import registration.RegistrationService;
        import registration.User;
        import static play.libs.Json.toJson;
         public class Registrations extends Controller {
            private static RegistrationService registrationService =          Play.application().plugin(RegistrationPlugin.class).getRegistrationService();
             @BodyParser.Of(BodyParser.Json.class)
            public static Result register() {
                try {
                    Form&lt;User&gt; form = Form.form(User.class).bindFromRequest();
                    User user = form.get();
                    registrationService.create(user);
                    return created(toJson(user));
                } catch (Exception e) {
                    return internalServerError(e.getMessage());
                }
            }
             @BodyParser.Of(BodyParser.Json.class)
            public static Result login() {
                try {
                    Form&lt;User&gt; form = Form.form(User.class).bindFromRequest();
                    User user = form.get();
                    if (registrationService.auth(user.getUsername(), user.getPassword())) {
                        return ok();
                    } else {
                        return forbidden();
                    }
                } catch (Exception e) {
                    return internalServerError(e.getMessage());
                }
            }
        }
    ```

1.  在项目根目录，`foo_java/conf/routes`中添加新添加的`Registration`动作的路由：

    ```java
    POST    /register   controllers.Registrations.register
        POST    /auth       controllers.Registrations.login
    ```

1.  最后，在项目根目录下的`foo_java/conf/play.plugins`文件中声明注册插件：

    ```java
    599:registration.RegistrationPlugin
    ```

1.  使用`curl`，提交新的注册和登录请求，并使用指定的注册详情；通过检查响应头来验证我们的端点对于成功操作返回 HTTP 状态码 200：

    ```java
    <span class="strong"><strong>    $ curl -v -X POST --header "Content-type: application/json"  http://localhost:9000/register -d '{"username":"ned@flanders.com", "password":"password"}'</strong></span>
    <span class="strong"><strong>    * Hostname was NOT found in DNS cache</strong></span>
    <span class="strong"><strong>    *   Trying ::1...</strong></span>
    <span class="strong"><strong>    * Connected to localhost (::1) port 9000 (#0)</strong></span>
    <span class="strong"><strong>    &gt; POST /register HTTP/1.1</strong></span>
    <span class="strong"><strong>    &gt; User-Agent: curl/7.37.1</strong></span>
    <span class="strong"><strong>    &gt; Host: localhost:9000</strong></span>
    <span class="strong"><strong>    &gt; Accept: */*</strong></span>
    <span class="strong"><strong>    &gt; Content-type: application/json</strong></span>
    <span class="strong"><strong>    &gt; Content-Length: 54</strong></span>
    <span class="strong"><strong>    &gt;</strong></span>
    <span class="strong"><strong>    * upload completely sent off: 54 out of 54 bytes</strong></span>
    <span class="strong"><strong>    &lt; HTTP/1.1 201 Created</strong></span>
    <span class="strong"><strong>    &lt; Content-Type: application/json; charset=utf-8</strong></span>
    <span class="strong"><strong>    &lt; Content-Length: 63</strong></span>
    <span class="strong"><strong>    &lt;</strong></span>
    <span class="strong"><strong>    * Connection #0 to host localhost left intact</strong></span>
    <span class="strong"><strong>    {"id":null,"username":"ned@flanders.com","password":"password"}%</strong></span>
     <span class="strong"><strong>    $ curl -v -X POST --header "Content-type: application/json"  http://localhost:9000/auth -d '{"username":"ned@flanders.com", "password":"password"}'</strong></span>
    <span class="strong"><strong>    * Hostname was NOT found in DNS cache</strong></span>
    <span class="strong"><strong>    *   Trying ::1...</strong></span>
    <span class="strong"><strong>    * Connected to localhost (::1) port 9000 (#0)</strong></span>
    <span class="strong"><strong>    &gt; POST /auth HTTP/1.1</strong></span>
    <span class="strong"><strong>    &gt; User-Agent: curl/7.37.1</strong></span>
    <span class="strong"><strong>    &gt; Host: localhost:9000</strong></span>
    <span class="strong"><strong>    &gt; Accept: */*</strong></span>
    <span class="strong"><strong>    &gt; Content-type: application/json</strong></span>
    <span class="strong"><strong>    &gt; Content-Length: 54</strong></span>
    <span class="strong"><strong>    &gt;</strong></span>
    <span class="strong"><strong>    * upload completely sent off: 54 out of 54 bytes</strong></span>
    <span class="strong"><strong>    &lt; HTTP/1.1 200 OK</strong></span>
    <span class="strong"><strong>    &lt; Content-Length: 0</strong></span>
    <span class="strong"><strong>    &lt;</strong></span>
    <span class="strong"><strong>    * Connection #0 to host localhost left intact</strong></span>
    ```

对于 Scala，我们需要执行以下步骤：

1.  使用 Hot-Reloading 启用运行`foo_scala`应用程序：

    ```java
    <span class="strong"><strong>    activator "~run"</strong></span>
    ```

1.  在模块目录（`foo_scala/modules`）内，使用`activator`生成我们的注册模块项目：

    ```java
    <span class="strong"><strong>    activator new registration play-scala</strong></span>
    ```

1.  在`build.sbt`中添加根项目`foo_scala`和模块`registration`之间的依赖关系：

    ```java
    lazy val root = (project in file("."))
          .enablePlugins(PlayScala)
          .aggregate(filemon)
          .dependsOn(filemon)
    <span class="strong"><strong>      .aggregate(registration)</strong></span>
    <span class="strong"><strong>      .dependsOn(registration)</strong></span>
         lazy val filemon = (project in file("modules/filemon"))
          .enablePlugins(PlayScala)
     <span class="strong"><strong>    lazy val registration = (project in file("modules/registration"))</strong></span>
    <span class="strong"><strong>      .enablePlugins(PlayScala)</strong></span>
    ```

1.  从注册模块中移除所有不必要的样板文件和配置：

    ```java
    <span class="strong"><strong>    rm -rf app/views app/controllers</strong></span>
    <span class="strong"><strong>    rm conf/routes</strong></span>
    <span class="strong"><strong>    mkdir app/registration</strong></span>
    <span class="strong"><strong>    echo "" &gt; conf/application.conf</strong></span>
    ```

1.  在`module/registration/app/registration/RegistrationPlugin.scala`中创建以下内容的注册插件：

    ```java
    package registration
         import play.api.{Application, Plugin}
         class RegistrationPlugin(app: Application) extends Plugin {
          val registrationService = new RegistrationService
           override def onStart() = {
            registrationService.init
          }
           override def onStop() = {
            registrationService.shutdown
          }
           override def enabled = true
        }
    ```

1.  接下来，创建由`Registration`插件引用的`RegistrationService`类：

    ```java
    // In modules/registration/app/registration/RegistrationService.scala
         package registration
         import java.util.UUID
         class RegistrationService {
          type ID = String
          private val registrations = scala.collection.mutable.Map[ID, User]()
           def init = {
            registrations.clear()
          }
           def create(user: User) = {
            val id: ID = UUID.randomUUID().toString
            registrations += (id -&gt; user.copy(Some(id), user.username, user.password))
          }
           def auth(username: String, password: String) = registrations.find(_._2.username equals username) match {
            case Some(reg) =&gt; {
              if (reg._2.password equals password) {
                Some(reg._2)
              } else {
                None
              }
            }
            case None =&gt; None
          }
           def shutdown = {
            registrations.clear()
          }
        }
    ```

1.  在`modules/registration/app/registration/User.scala`中创建`User`模型实体：

    ```java
    package registration
         case class User(id: Option[String], username: String, password: String)
    ```

1.  在项目根目录，`foo_scala/app/controllers/Registrations.scala`中创建处理注册和登录请求的`Registration`控制器和路由：

    ```java
    package controllers
         import play.api.Play.current
        import play.api.Play
        import play.api.libs.json.{JsError, Json}
        import play.api.mvc.{BodyParsers, Action, Controller}
        import registration.{RegistrationPlugin, User, RegistrationService}
         object Registrations extends Controller {
          implicit private val writes = Json.writes[User]
          implicit private val reads = Json.reads[User]
          private val registrationService: RegistrationService = Play.application.plugin[RegistrationPlugin]
        .getOrElse(throw new IllegalStateException("RegistrationService is required!"))
        .registrationService
           def register = Action(BodyParsers.parse.json) { implicit request =&gt;
            val post = request.body.validate[User]
             post.fold(
              errors =&gt; BadRequest(JsError.toFlatJson(errors)),
              u =&gt; {
                registrationService.create(u)
                Created(Json.toJson(u))
              }
            )
          }
           def login = Action(BodyParsers.parse.json) { implicit request =&gt;
            val login = request.body.validate[User]
             login.fold(
              errors =&gt; BadRequest(JsError.toFlatJson(errors)),
              u =&gt; {
                registrationService.auth(u.username, u.password) match {
                  case Some(user) =&gt; Ok
                  case None =&gt; Forbidden
                }
              }
            )
          }
        }
    ```

1.  添加新添加的`Registration`动作的路由：

    ```java
    POST    /register   controllers.Registrations.register
        POST    /auth       controllers.Registrations.login
    ```

1.  最后，在项目根目录中，`foo_scala/conf/play.plugins`文件中声明注册插件：

    ```java
    599:registration.RegistrationPlugin
    ```

1.  使用`curl`，提交新的注册和登录请求，并使用指定的注册详情；通过检查响应头来验证我们的端点对于成功操作返回 HTTP 状态码 200：

    ```java
    <span class="strong"><strong>    $ curl -v -X POST --header "Content-type: application/json"  http://localhost:9000/register -d '{"username":"ned@flanders.com", "password":"password"}'</strong></span>
    <span class="strong"><strong>    * Hostname was NOT found in DNS cache</strong></span>
    <span class="strong"><strong>    *   Trying ::1...</strong></span>
    <span class="strong"><strong>    * Connected to localhost (::1) port 9000 (#0)</strong></span>
    <span class="strong"><strong>    &gt; POST /register HTTP/1.1</strong></span>
    <span class="strong"><strong>    &gt; User-Agent: curl/7.37.1</strong></span>
    <span class="strong"><strong>    &gt; Host: localhost:9000</strong></span>
    <span class="strong"><strong>    &gt; Accept: */*</strong></span>
    <span class="strong"><strong>    &gt; Content-type: application/json</strong></span>
    <span class="strong"><strong>    &gt; Content-Length: 54</strong></span>
    <span class="strong"><strong>    &gt;</strong></span>
    <span class="strong"><strong>    * upload completely sent off: 54 out of 54 bytes</strong></span>
    <span class="strong"><strong>    &lt; HTTP/1.1 201 Created</strong></span>
    <span class="strong"><strong>    &lt; Content-Type: application/json; charset=utf-8</strong></span>
    <span class="strong"><strong>    &lt; Content-Length: 63</strong></span>
    <span class="strong"><strong>    &lt;</strong></span>
    <span class="strong"><strong>    * Connection #0 to host localhost left intact</strong></span>
    <span class="strong"><strong>    {"id":null,"username":"ned@flanders.com","password":"password"}%</strong></span>
     <span class="strong"><strong>    $ curl -v -X POST --header "Content-type: application/json"  http://localhost:9000/auth -d '{"username":"ned@flanders.com", "password":"password"}'</strong></span>
    <span class="strong"><strong>    * Hostname was NOT found in DNS cache</strong></span>
    <span class="strong"><strong>    *   Trying ::1...</strong></span>
    <span class="strong"><strong>    * Connected to localhost (::1) port 9000 (#0)</strong></span>
    <span class="strong"><strong>    &gt; POST /auth HTTP/1.1</strong></span>
    <span class="strong"><strong>    &gt; User-Agent: curl/7.37.1</strong></span>
    <span class="strong"><strong>    &gt; Host: localhost:9000</strong></span>
    <span class="strong"><strong>    &gt; Accept: */*</strong></span>
    <span class="strong"><strong>    &gt; Content-type: application/json</strong></span>
    <span class="strong"><strong>    &gt; Content-Length: 54</strong></span>
    <span class="strong"><strong>    &gt;</strong></span>
    <span class="strong"><strong>    * upload completely sent off: 54 out of 54 bytes</strong></span>
    <span class="strong"><strong>    &lt; HTTP/1.1 200 OK</strong></span>
    <span class="strong"><strong>    &lt; Content-Length: 0</strong></span>
    <span class="strong"><strong>    &lt;</strong></span>
    <span class="strong"><strong>    * Connection #0 to host localhost left intact</strong></span>
    ```

## 它是如何工作的…

在这个配方中，我们创建了一个模块来处理注册函数，例如注册和登录。我们将其创建为一个 Play 插件，以便它不仅易于维护，而且可以在其他应用程序中重用。使用模块的另一个优点是，在编写单元测试时，我们可以在其封闭的子项目中隔离其执行，而不是整个项目。

我们在项目根目录的模块目录内创建了`registration`插件。我们在`build.sbt`中声明了模块与主项目之间的依赖关系：

```java
lazy val root = (project in file("."))
      .enablePlugins(PlayScala)
      .aggregate(filemon)
      .dependsOn(filemon)
      .aggregate(registration)
      .dependsOn(registration)
```

然后，我们在`conf/play.plugins`中使用优先级`599`（在 500-600 范围内用于数据相关插件）启用了插件：

```java
599:registration.RegistrationPlugin
```

然后，我们从控制器中的`Registration`插件中获取了`RegistrationService`接口的引用：

```java
// Java
    private static RegistrationService registrationService =
            Play.application().plugin(RegistrationPlugin.class).getRegistrationService();
     // Scala
    private val registrationService: RegistrationService = Play.application.plugin[RegistrationPlugin]
    .getOrElse(throw new IllegalStateException("RegistrationService is required!"))
    .registrationService
```

一旦建立了引用，所有注册函数都简单地从控制器委托给了`RegistrationService`接口：

```java
// Java
    registrationService.create(user);
     // Scala
    registrationService.create(u)
```

使用`curl`，我们还可以验证我们的注册控制器是否正确响应了错误的认证：

```java
<span class="strong"><strong>    $ curl -v -X POST --header "Content-type: application/json"  http://localhost:9000/auth -d '{"username":"ned@flanders.com", "password":"passwordz"}'</strong></span>
<span class="strong"><strong>    * Hostname was NOT found in DNS cache</strong></span>
<span class="strong"><strong>    *   Trying ::1...</strong></span>
<span class="strong"><strong>    * Connected to localhost (::1) port 9000 (#0)</strong></span>
<span class="strong"><strong>    &gt; POST /auth HTTP/1.1</strong></span>
<span class="strong"><strong>    &gt; User-Agent: curl/7.37.1</strong></span>
<span class="strong"><strong>    &gt; Host: localhost:9000</strong></span>
<span class="strong"><strong>    &gt; Accept: */*</strong></span>
<span class="strong"><strong>    &gt; Content-type: application/json</strong></span>
<span class="strong"><strong>    &gt; Content-Length: 55</strong></span>
<span class="strong"><strong>    &gt;</strong></span>
<span class="strong"><strong>    * upload completely sent off: 55 out of 55 bytes</strong></span>
<span class="strong"><strong>    &lt; HTTP/1.1 403 Forbidden</strong></span>
<span class="strong"><strong>    &lt; Content-Length: 0</strong></span>
<span class="strong"><strong>    &lt;</strong></span>
<span class="strong"><strong>    * Connection #0 to host localhost left intact</strong></span>
```

# 使用相同的模型为不同的应用程序

对于这个配方，我们将创建一个新的独立模块，该模块将包含与产品相关的函数和数据模型类，并且我们将它发布到本地仓库中：

## 如何操作...

对于 Java，我们需要执行以下步骤：

1.  在与`foo_java`相同的目录级别中创建一个新的 Play 2 项目：

    ```java
    <span class="strong"><strong>    activator new product-contrib play-java</strong></span>
    ```

1.  在`app/product-contrib/app`目录中创建我们的默认模块包：

    ```java
    <span class="strong"><strong>    mkdir app/productcontrib</strong></span>
    ```

1.  创建一个包含所有数据模型类的模型包：

    ```java
    <span class="strong"><strong>    mkdir app/productcontrib/models</strong></span>
    ```

1.  删除`conf/application.conf`文件的内容：

    ```java
    echo "" &gt; conf/application.conf
    ```

1.  删除`conf/routes`文件的内容，并将其重命名为`productcontrib.routes`：

    ```java
    echo "" &gt; conf/routes &amp;&amp; mv conf/routes  modules/filemon/conf/productcontrib.routes
    ```

1.  从`modules/filemon/app`中删除视图目录：

    ```java
    <span class="strong"><strong>    rm -rf app/views</strong></span>
    ```

1.  删除`app/controller/Application.java`文件：

    ```java
    <span class="strong"><strong>    rm app/controllers/Application.java</strong></span>
    ```

1.  在`app/productcontrib/models/Product.java`中创建产品模型，内容如下：

    ```java
    package productcontrib.models;
         import java.io.Serializable;
         public class Product implements Serializable {
            private String sku;
            private String title;
            private Double price;
             public String getSku() {
                return sku;
            }
            public void setSku(String sku) {
                this.sku = sku;
            }
            public String getTitle() {
                return title;
            }
            public void setTitle(String title) {
                this.title = title;
            }
            public Double getPrice() {
                return price;
            }
            public void setPrice(Double price) {
                this.price = price;
            }
        }
    ```

1.  在`app/productcontrib/services`包中创建`ProductService`接口（`ProductService.java`）和实现类（`ProductServiceImpl.java`）：

    ```java
    // ProductService.java
        package productcontrib.services;
         public interface ProductService {
            String generateProductId();
        }
         // ProductServiceImpl.java 
       package productcontrib.services;
         import java.util.UUID;
         public class ProductServiceImpl implements ProductService {
            @Override
            public String generateProductId() {
                return UUID.randomUUID().toString();
            }
        }
    ```

1.  在`build.sbt`文件中插入额外的模块包设置：

    ```java
    name := """product-contrib"""
      version := "1.0-SNAPSHOT"
      organization := "foojava"
    ```

1.  使用 activator 构建`contrib.jar`并将其发布到远程内部仓库：

    ```java
    <span class="strong"><strong>    activator clean publish-local</strong></span>
    ```

1.  你应该在控制台日志中能够确认上传是否成功：

    ```java
    <span class="strong"><strong>    [info]   published product-contrib_2.11 to /.ivy2/local/foojava/product-contrib_2.11/1.0-SNAPSHOT/poms/product-contrib_2.11.pom</strong></span>
    <span class="strong"><strong>    [info]   published product-contrib_2.11 to /.ivy2/local/foojava/product-contrib_2.11/1.0-SNAPSHOT/jars/product-contrib_2.11.jar</strong></span>
    <span class="strong"><strong>    [info]   published product-contrib_2.11 to /.ivy2/local/foojava/product-contrib_2.11/1.0-SNAPSHOT/srcs/product-contrib_2.11-sources.jar</strong></span>
    <span class="strong"><strong>    [info]   published product-contrib_2.11 to /.ivy2/local/foojava/product-contrib_2.11/1.0-SNAPSHOT/docs/product-contrib_2.11-javadoc.jar</strong></span>
    <span class="strong"><strong>    [info]   published ivy to /.ivy2/local/foojava/product-contrib_2.11/1.0-SNAPSHOT/ivys/ivy.xml</strong></span>
    ```

对于 Scala，我们需要执行以下步骤：

1.  在与`foo_scala`相同的目录级别中创建一个新的 Play 2 项目：

    ```java
    <span class="strong"><strong>    activator new user-contrib play-scala</strong></span>
    ```

1.  在`app/user-contrib/app`目录中创建默认模块包：

    ```java
    <span class="strong"><strong>    mkdir app/usercontrib</strong></span>
    ```

1.  创建一个包含所有数据模型类的模型包：

    ```java
    <span class="strong"><strong>    mkdir app/usercontrib/models</strong></span>
    ```

1.  删除`conf/application.conf`文件的内容：

    ```java
    echo "" &gt; conf/application.conf
    ```

1.  删除`conf/routes`文件的内容，并将其重命名为`usercontrib.routes`：

    ```java
    echo "" &gt; conf/routes &amp;&amp; mv conf/routes  modules/filemon/conf/usercontrib.routes
    ```

1.  从`modules/filemon/app`中删除视图目录：

    ```java
    <span class="strong"><strong>    rm -rf app/views</strong></span>
    ```

1.  删除`app/controller/Application.scala`文件：

    ```java
    <span class="strong"><strong>    rm app/controllers/Application.scala</strong></span>
    ```

1.  在`app/usercontrib/models/User.scala`中创建`User`模型，内容如下：

    ```java
    package usercontrib.models
         import java.util.UUID
         case class User(id: Option[String], username: String, password: String)
         object User {
          def generateId = UUID.randomUUID().toString
        }
    ```

1.  在`build.sbt`文件中插入额外的模块包设置：

    ```java
    name := """product-contrib"""
      version := "1.0-SNAPSHOT"
      organization := "foojava"
    ```

1.  使用 activator 构建`contrib.jar`并将其发布到远程内部仓库：

    ```java
    <span class="strong"><strong>    activator clean publish-local</strong></span>
    ```

1.  您应该能够在控制台日志中确认上传是否成功：

    ```java
    <span class="strong"><strong>    [info]   published user-contrib_2.11 to ivy/fooscala/user-contrib_2.11/1.0-SNAPSHOT/user-contrib_2.11-1.0-SNAPSHOT.pom</strong></span>
    <span class="strong"><strong>    [info]   published user-contrib_2.11 to ivy/fooscala/user-contrib_2.11/1.0-SNAPSHOT/user-contrib_2.11-1.0-SNAPSHOT.jar</strong></span>
    <span class="strong"><strong>    [info]   published user-contrib_2.11 to ivy/fooscala/user-contrib_2.11/1.0-SNAPSHOT/user-contrib_2.11-1.0-SNAPSHOT-sources.jar</strong></span>
    <span class="strong"><strong>    [info]   published user-contrib_2.11 to ivy/fooscala/user-contrib_2.11/1.0-SNAPSHOT/user-contrib_2.11-1.0-SNAPSHOT-javadoc.jar</strong></span>
    ```

## 它是如何工作的...

在这个菜谱中，我们创建了一个新的 Play 2.0 模块，目的是将模块打包并发布到我们的本地仓库。这使得 Play 模块可供我们正在工作的其他 Play web 应用程序使用。我们为将成为我们模块一部分的产品创建了模型和服务类：

```java
# Java 
     $ find app/productcontrib
    app/productcontrib
    app/productcontrib/models
    app/productcontrib/models/Product.java
    app/productcontrib/services
    app/productcontrib/services/ProductService.java
    app/productcontrib/services/ProductServiceImpl.java
     # Scala
  $ find app/usercontrib
    app/usercontrib
    app/usercontrib/models
    app/usercontrib/models/User.scala
```

我们使用`activator`的发布命令构建并发布了这两个模块到内部仓库：

```java
<span class="strong"><strong>    activator clean publish-local</strong></span>
```

一旦这些模块在内部仓库中发布，我们就将它们声明为基于 Maven 的 Java 项目的依赖项，不仅限于 Play 2.0 应用程序，在我们的例子中是`build.sbt`：

```java
// Java 
    "foojava" %% "product-contrib" % "1.0-SNAPSHOT"
     // Scala
    "fooscala" %% "user-contrib" % "1.0-SNAPSHOT"
```

# 管理模块依赖项

在这个菜谱中，我们将解决将 Play 模块添加到您的 Play 2.0 应用程序中的问题，这进一步展示了 Play 2.0 生态系统的强大。这个菜谱需要运行之前的菜谱，并假设您已经继续执行。

## 如何做到这一点...

对于 Java，我们需要执行以下步骤：

1.  启用热重载功能运行`foo_java`应用程序：

    ```java
    activator "~run"
    ```

1.  在`build.sbt`中将`fooscala user-contrib`模块添加为项目依赖：

    ```java
    "fooscala" %% "user-contrib" % "1.0-SNAPSHOT",
        "foojava" %% "product-contrib" % "1.0-SNAPSHOT"
    ```

1.  通过添加以下操作修改`foo_java/app/controllers/Application.java`：

    ```java
    // Add the required imports  
        import productcontrib.services.ProductService;
        import productcontrib.services.ProductServiceImpl;
        import usercontrib.models.User;
         // Add the necessary Action methods and helper
        private static ProductService productService = new ProductServiceImpl();
         public static Result generateProductId() {
            return ok("Your generated product id: " + productService.generateProductId());
        }
         public static Result generateUserId() {
            return ok("Your generated product id: " + User.generateId());
        }
    ```

1.  在`foo_java/conf/routes`中为新增的操作添加一个新的路由：

    ```java
    GET     /generate-product-id       controllers.Application.generateProductId()
        GET     /generate-user-id          controllers.Application.generateUserId()
    ```

1.  使用`curl`，我们将能够显示由产品和用户贡献模块生成的产品和用户 Ids：

    ```java
    <span class="strong"><strong>    $ curl http://localhost:9000/generate-product-id</strong></span>
    <span class="strong"><strong>Your generated product id: 3acd3f36-6ee6-45ce-af07-faa257724b1e%                </strong></span>
    <span class="strong"><strong>    $ curl http://localhost:9000/generate-user-id</strong></span>
    <span class="strong"><strong>Your generated product id: ffca654e-35d8-48cd-9acd-9ea9fe567ba7%</strong></span>
    ```

对于 Scala，我们需要执行以下步骤：

1.  启用热重载功能运行`foo_scala`应用程序：

    ```java
    <span class="strong"><strong>    activator "~run"</strong></span>
    ```

1.  在`build.sbt`中将`foojava productcontrib`模块添加为项目依赖：

    ```java
    "foojava" %% "product-contrib" % "1.0-SNAPSHOT",
      "fooscala" %% "user-contrib" % "1.0-SNAPSHOT"
    ```

1.  通过添加以下操作修改`foo_scala/app/controllers/Application.scala`：

    ```java
    import productcontrib.services.{ProductServiceImpl, ProductService}
        import usercontrib.models.User
         def productService: ProductService = new ProductServiceImpl
         def generateProductId = Action {
          Ok("Your generated product id: " + productService.generateProductId())
        }
         def generateUserId = Action {
          Ok("Your generated product id: " + User.generateId);
        }
    ```

1.  在`foo_scala/conf/routes`中为新增的操作添加一个新的路由：

    ```java
    GET     /generate-product-id       controllers.Application.generateProductId()
        GET     /generate-user-id          controllers.Application.generateUserId()
    ```

1.  使用`curl`，我们将能够显示由产品和用户贡献模块生成的产品和用户 Ids：

    ```java
    <span class="strong"><strong>    $ curl http://localhost:9000/generate-product-id</strong></span>
    <span class="strong"><strong>Your generated product id: 3acd3f36-6ee6-45ce-af07-faa257724b1e%                </strong></span>
    <span class="strong"><strong>    $ curl http://localhost:9000/generate-user-id</strong></span>
    <span class="strong"><strong>Your generated product id: ffca654e-35d8-48cd-9acd-9ea9fe567ba7%</strong></span>
    ```

## 它是如何工作的...

在这个菜谱中，我们探讨了如何将其他模块包含到我们的 Play 2.0 web 应用程序中。通过这个菜谱，我们还展示了 Play Scala 应用程序如何与 Play Java 模块协同工作，反之亦然。

我们首先在`build.sbt`中声明我们的根项目将使用用户和产品贡献模块：

```java
libraryDependencies ++= Seq(
      "foojava" %% "product-contrib" % "1.0-SNAPSHOT",
      "fooscala" %% "user-contrib" % "1.0-SNAPSHOT"
    )
```

然后，我们在控制器中添加了导入语句，以便我们可以调用它们的 ID 生成函数：

```java
// Java
    import productcontrib.services.ProductService;
    import productcontrib.services.ProductServiceImpl;
    import usercontrib.models.User;
     // Invoke the contrib functions in foo_java
    return ok("Your generated product id: " + productService.generateProductId());
    return ok("Your generated product id: " + User.generateId());

     // Scala
    import productcontrib.services.{ProductServiceImpl, ProductService}
    import usercontrib.models.User
     // Invoke the contrib functions in foo_scala:
    Ok("Your generated product id: " + productService.generateProductId())
    Ok("Your generated product id: " + User.generateId);
```

最后，我们使用`curl`请求我们的新路由，以查看生成的 Ids 在实际中的应用。

# 使用 Amazon S3 添加私有模块仓库

在这个菜谱中，我们将探讨如何使用外部模块存储库来发布和解析内部模块，以利于我们模块的分布。在这个菜谱中，我们将使用 Amazon S3，这是一个流行的云存储服务，来存储我们的 ivy 风格存储库资产。您需要有效的 AWS 账户来遵循这个菜谱，确保您在[`aws.amazon.com/`](http://aws.amazon.com/)上注册了一个账户。

请参阅 S3 的在线文档以获取更多信息：

[`aws.amazon.com/s3/`](http://aws.amazon.com/s3/)

## 如何操作…

我们需要执行以下步骤：

1.  打开`product-contrib`项目，并启用热重载功能运行应用程序：

    ```java
    <span class="strong"><strong>    activator "~run"</strong></span>
    ```

1.  在`project/plugins.sbt`文件中编辑插件配置文件，并添加以下插件和解析器：

    ```java
    resolvers += "Era7 maven releases" at "http://releases.era7.com.s3.amazonaws.com"
         addSbtPlugin("ohnosequences" % "sbt-s3-resolver" % "0.12.0")
    ```

1.  在`build.sbt`文件中编辑构建配置文件，以指定我们将用于 S3 解析器插件的设置：

    ```java
    S3Resolver.defaults
         s3credentials := file(System.getProperty("user.home")) / ".sbt" / ".s3credentials"
         publishMavenStyle := false
         publishTo := {
          val prefix = if (isSnapshot.value) "snapshots" else "releases"
          Some(s3resolver.value(prefix+" S3 bucket",     s3(prefix+".YOUR-S3-BUCKET-NAME-HERE.amazonaws.com")) withIvyPatterns)
        }
    ```

1.  在文件`~/.sbt/.s3credentials`中指定您的 Amazon S3 API 密钥：

    ```java
    accessKey = &lt;YOUR S3 API ACCESS KEY&gt;
        secretKey = &lt;YOUR S3 API SECRET KEY&gt;
    ```

1.  接下来，使用 activator 发布`product-contrib`快照：

    ```java
    <span class="strong"><strong>    activator clean publish</strong></span>
    ```

1.  您将在控制台日志中看到上传的成功状态消息：

    ```java
    <span class="strong"><strong>    [info]   published ivy to s3://snapshots.XXX.XXX.amazonaws.com/foojava/product-contrib_2.11/1.0-SNAPSHOT/ivys/ivy.xml</strong></span>
    <span class="strong"><strong>    [success] Total time: 58 s, completed 02 3, 15 9:39:44 PM</strong></span>
    ```

1.  我们现在将在新的 Play 2.0 应用程序中使用此存储库：

    ```java
    <span class="strong"><strong>    activator new s3deps play-scala</strong></span>
    ```

1.  在`s3deps/project/plugins.sbt`文件中编辑插件配置文件，并添加以下插件和解析器：

    ```java
    resolvers += "Era7 maven releases" at "http://releases.era7.com.s3.amazonaws.com"
        addSbtPlugin("ohnosequences" % "sbt-s3-resolver" % "0.12.0")
    ```

1.  在`build.sbt`文件中编辑构建配置文件，以指定我们将用于 S3 解析器插件的设置：

    ```java
    S3Resolver.defaults
         resolvers ++= SeqResolver) withIvyPatterns
        )
         libraryDependencies ++= Seq(
          "foojava" %% "product-contrib" % "1.0-SNAPSHOT"
        )
    ```

1.  使用 activator 检索 product-contrib 模块：

    ```java
    activator clean dependencies
       ...
        com.typesafe.play:play-java-ws_2.11:2.3.7
        com.typesafe.play:play-java-jdbc_2.11:2.3.7
        foojava:product-contrib_2.11:1.0-SNAPSHOT
    ```

## 它是如何工作的…

在这个菜谱中，我们使用了 sbt-s3-resolver 插件，通过 Amazon S3 发布和解析依赖项。我们在`project/plugins.sbt`文件中包含了`sbt`插件：

```java
resolvers += "Era7 maven releases" at "http://releases.era7.com.s3.amazonaws.com"
     addSbtPlugin("ohnosequences" % "sbt-s3-resolver" % "0.12.0")
```

我们在`~/.sbt 目录`中的`.s3credentials`文件中指定我们的 Amazon S3 API 密钥：

```java
accessKey = &lt;ACCESS KEY&gt;
    secretKey = &lt;SECRET KEY&gt;
```

对于发布，我们在发布项目的`build.sbt`中指定解析器存储库：

```java
S3Resolver.defaults
     s3credentials := file(System.getProperty("user.home")) / ".sbt" / ".s3credentials"
     publishMavenStyle := false
     publishTo := {
      val prefix = if (isSnapshot.value) "snapshots" else "releases"
      Some(s3resolver.value(prefix+" S3 bucket",     s3(prefix+".achiiva.devint.amazonaws.com")) withIvyPatterns)
    }
```

要解析依赖项，我们在消费项目的`build.sbt`中指定以下内容（s3deps）：

```java
S3Resolver.defaults
     resolvers ++= SeqResolver) withIvyPatterns
    )
     libraryDependencies ++= Seq(
      "foojava" %% "product-contrib" % "1.0-SNAPSHOT"
    )
```
