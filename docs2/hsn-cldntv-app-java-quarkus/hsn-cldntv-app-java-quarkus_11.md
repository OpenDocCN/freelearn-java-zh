# 高级应用程序开发

在本章中，我们将探讨一些 Quarkus 的高级功能，这些功能将帮助您设计和编写前沿的 Quarkus 应用程序。我们将学习的内容将涵盖 Quarkus API 的不同领域，从高级配置选项到控制 Quarkus 应用程序的生命周期以及使用 Quarkus 调度器触发基于时间的事件。

到本章结束时，您将能够利用以下高级功能：

+   使用高级 MicroProfile 配置选项

+   控制您服务的生命周期事件

+   在您的服务中安排周期性任务

# 技术要求

您可以在本章中找到项目的源代码，在 GitHub 上的 [`github.com/PacktPublishing/Hands-On-Cloud-Native-Applications-with-Java-and-Quarkus/tree/master/Chapter08`](https://github.com/PacktPublishing/Hands-On-Cloud-Native-Applications-with-Java-and-Quarkus/tree/master/Chapter08)。

# 使用高级配置选项

如我们所学的，Quarkus 依赖于 MicroProfile Config 规范来将配置属性注入到我们的应用程序中。到目前为止，我们已使用默认配置文件（命名为 `application.properties`）为应用程序的初始设置提供初始值。

让我们通过一个基本示例来回顾如何注入属性，包括为属性提供默认值：

```java
@Inject
@ConfigProperty(name="tempFileName", defaultValue="file.tmp")
String fileName;
```

在前面的代码中，我们将一个应用程序属性注入到 `fileName` 变量中。请注意，属性名称应仔细规划，因为 Quarkus 随带一套广泛的系统属性，可用于管理其环境。幸运的是，您不需要手头有文档来检查所有可用的系统属性。事实上，您可以使用 Maven 的 `generate-config` 命令列出所有内置的系统属性，基于您当前安装的扩展：

```java
mvn quarkus:generate-config
```

此命令将在 `src/main/resources` 文件夹下创建一个名为 `application.properties.example` 的文件。如果您打开此文件，您将看到它包含一个注释列表，列出了所有可用的配置选项，这些选项位于 `quarkus` 命名空间下。以下是其中的一段摘要：

```java
# The name of the application.
# If not set, defaults to the name of the project.
#
#quarkus.application.name=

# The version of the application.
# If not set, defaults to the version of the project
#
#quarkus.application.version=
```

作为旁注，您可以通过添加 `-Dfile=<filename>` 选项来为 `generate-command` 选择不同的文件名。

在接下来的章节中，我们将学习一些使用本书 GitHub 存储库中 `Chapter08/advanced-config` 文件夹中的示例作为参考的高级配置练习。我们建议在继续之前将项目导入到您的 IDE 中。

# 多个配置源

当涉及到设置应用程序属性时，`application.properties` 文件并非唯一的选择。根据 MicroProfile 的 Config 规范，您还可以使用以下选项：

+   **Java 系统属性**：可以通过 `System.getProperty()` 和 `System.setProperty()` API 以编程方式读取/写入 Java 系统属性。作为替代，您可以使用 `-D` 选项在命令行上设置属性，如下所示：

```java
java -Dquarkus.http.port=8180 app.jar
```

+   **环境变量**：这需要为属性设置一个环境变量，如下所示：

```java
export QUARKUS_HTTP_PORT=8180
```

如您可能已注意到的，匹配的环境变量名称已被设置为 uppercase，并且点号已被下划线替换。

请注意，在 Quarkus 的当前版本中，还必须在 `application.properties` 中定义该变量，以便它可以被环境变量覆盖。

最后，我们还可以通过向我们的应用程序添加一个新的配置源来从外部收集我们的配置。下一节将展示我们如何做到这一点。

# 配置自定义配置源

在我们迄今为止创建的所有示例中，我们假设应用程序配置是从 `src/main/resources/application.properties` 文件中获取的，这是 Quarkus 应用程序的默认设置。尽管如此，由于 Quarkus 完全支持 **MicroProfile Config** 规范，从另一个来源加载配置（例如外部文件系统、数据库或任何可以被 Java 应用程序加载的东西）是完全可能的！

为了做到这一点，您必须实现 `org.eclipse.microprofile.config.spi.ConfigSource` 接口，该接口公开了一组用于加载属性（`getProperties`）、检索属性名称（`getPropertyNames`）和检索相应的值（`getValue`）的方法。

作为概念验证，请查看以下实现，该实现可在 `Chapter08/advanced-config` 项目中找到：

```java
public class FileConfigSource implements ConfigSource {
    private final String CONFIG_FILE = "/tmp/config.properties";
    private final String CONFIG_SOURCE_NAME = "ExternalConfigSource";
    private final int ORDINAL = 900;

    @Override
    public Map getProperties() {

        try(InputStream in = new FileInputStream( CONFIG_FILE )){

            Properties properties = new Properties();
            properties.load( in );

            Map map = new HashMap();
            properties.stringPropertyNames()
                    .stream()
                    .forEach(key-> map.put(key, 
                     properties.getProperty(key)));

            return map;

        } catch (IOException e) {
            e.printStackTrace();
        }

        return null;
    }

    @Override
    public Set getPropertyNames() {

        try(InputStream in = new FileInputStream( CONFIG_FILE )){

            Properties properties = new Properties();
            properties.load( in );

            return properties.stringPropertyNames();

        } catch (IOException e) {
            e.printStackTrace();
        }

        return null;
    }

    @Override
    public int getOrdinal() {
        return ORDINAL;
    }

    @Override
    public String getValue(String s) {

        try(InputStream in = new FileInputStream( CONFIG_FILE )){
            Properties properties = new Properties();
            properties.load( in );
            return properties.getProperty(s);

        } catch (IOException e) {
            e.printStackTrace();
        }

        return null;
    }

    @Override
    public String getName() {
        return CONFIG_SOURCE_NAME;
    }
}
```

如果您熟悉 `java.io` API，代码本身相当简单。`FileConfigSource` 类尝试从您的文件系统的 `/tmp/config.properties` 路径加载外部配置。值得一提的是，已经设置了一个 `ORDINAL` 变量来指定此 `ConfigSource` 类的顺序，以防从多个来源加载了多个属性。

`ConfigSource` 的默认值设置为 `100`，在属性在多个来源中定义的情况下，具有最高序数值的来源具有优先级。以下是可用配置源的排名：

| **配置源** | **值** |
| --- | --- |
| `application.properties` | `100` |
| 环境变量 | `300` |
| 系统属性 | `400` |

由于我们在示例中将 `ORDINAL` 变量设置为 `900`，如果存在其他配置源，它将覆盖其他配置源。

一旦自定义 `ConfigSource` 在项目中可用，我们需要为此类进行注册。为此，我们在项目的 `resources/META-INF/services` 文件夹下添加了一个名为 `org.eclipse.microprofile.config.spi.ConfigSource` 的文件。以下是项目的树视图，位于 `resources` 文件夹下：

```java
│   └── resources
│       └── META-INF
│               └── services
│                   ├── org.eclipse.microprofile.config.spi.ConfigSource
```

在此文件中，我们指定了 `ConfigSource` 的完全限定名。在我们的情况下，如下所示：

```java
com.packt.chapter8.FileConfigSource
```

现在，一旦应用程序启动，自定义的 `ConfigSource` 将被加载，并且其属性将覆盖其他潜在的同名属性副本。

在您的项目中的 `AdvancedConfigTest` 类中，您将找到一个断言，该断言验证了一个属性已从外部的 `FileConfigSource` 类加载：

```java
given()
        .when().get("/hello")
        .then()
        .statusCode(200)
        .body(is("custom greeting"));
```

关于 `AdvancedConfigTest` 类的更多细节将在本章后面讨论。

# 在配置中使用转换器

要讨论配置转换器，让我们以这个简单的配置示例为例：

```java
year=2019
isUser=true
```

在这里，将前面的属性注入到我们的代码中是完全可以接受的：

```java
@ConfigProperty(name = "year", defaultValue = "2020")
Integer year;

@ConfigProperty(name = "isUser", defaultValue = "false")
Boolean isUser;
```

在底层，MicroProfile Config API 为不仅仅是普通字符串的值提供了类型安全的转换。

此外，请注意，我们可以为属性提供一个默认值，如果属性在我们的配置中未定义，则将使用该默认值。

这是通过在配置模型中提供转换器来实现的。默认情况下，MicroProfile Config API 已经提供了一些内置的转换器。以下是一些内置转换器的列表：

+   `boolean` 和 `java.lang.Boolean`。以下值被转换为布尔值（不区分大小写）：`true`、`YES`、`Y`、`1` 和 `ON`。任何其他值都将为 `false`。

+   `byte` 和 `java.lang.Byte`。

+   `short` 和 `java.lang.Short`。

+   `int` 和 `java.lang.Integer`。

+   `long` 和 `java.lang.Long`。

+   `float` 和 `java.lang.Float`。点 `.` 用于分隔小数位。

+   `double` 和 `java.lang.Double`。点 `.` 用于分隔小数位。

+   `char` 和 `java.lang.Character`。

+   `java.lang.Class`。这是基于 `Class.forName` 的结果。

数组、列表和集合也受到支持。为了将其中一个集合注入到类的变量中，您可以使用逗号（`,`）字符作为分隔符，`\` 作为转义字符。例如，考虑以下配置：

```java
students=Tom,Pat,Steve,Lucy
```

以下代码将前面的配置注入到 `java.util.List` 元素中：

```java
@ConfigProperty(name = "students")
List<String> studentList;
```

以类似的方式，您可以使用内置转换器从值列表生成 `Array`。看看以下配置示例：

```java
pets=dog,cat,bunny
```

以下配置可以注入到字符串数组中如下：

```java
@ConfigProperty(name = "pets")
String[] petsArray;
```

类也可以作为配置的一部分进行注入：

```java
myclass=TestClass
```

在运行时，该类将由类加载器搜索，并使用 `Class.forName` 构造创建。我们可以在代码中这样放置它：

```java
@ConfigProperty(name = "myclass")
TestClass clazz;
```

最后，值得一提的是，您可以将整个 `Config` 对象注入，并在需要时检索单个属性：

```java
@Inject
Config config;

@GET
@Produces(MediaType.TEXT_PLAIN)
public String hello() {
    Integer y = config.getValue("year", Integer.class);
    return "Year is " +y;
}
```

现在，让我们探索一些更高级的策略来创建类型转换器。

# 添加自定义转换器

如果内置转换器的列表不足以满足需求，你仍然可以通过实现通用接口，即`org.eclipse.microprofile.config.spi.Converter`来创建自定义转换器。接口的`Type`参数是字符串转换成的目标类型：

```java
public class MicroProfileCustomValueConverter implements Converter<CustomConfigValue> {

    public MicroProfileCustomValueConverter() {
    }

    @Override
    public CustomConfigValue convert(String value) {
        return new CustomConfigValue(value);
    }
}
```

以下代码是为目标`Type`参数编写的，它从一个我们已包含在配置中的普通 Java 字符串派生而来：

```java
public class CustomConfigValue {

    private final String email;
    private final String user;

    public CustomConfigValue(String value) {

        StringTokenizer st = new StringTokenizer(value,";");
        this.user = st.nextToken();
        this.email = st.nextToken();       
    }

    public String getEmail() {
        return email;
    }

    public String getUser() {
        return user;
    }
```

你必须在名为`resources/META-INF/services/org.eclipse.microprofile.config.spi.Converter`的文件中注册你的转换器。包括自定义实现的完全限定类名。例如，在我们的案例中，我们添加了以下行：

```java
com.packt.chapter8.MicroProfileCustomValueConverter
```

现在，让我们学习如何在实践中使用我们的自定义转换器。为此，我们将向`application.properties`文件中添加以下行，该行使用`CustomConfigValue`类构造函数中编码的模式：

```java
customconfig=john;johnsmith@gmail.com
```

现在，自定义转换器可以作为类属性注入到我们的代码中：

```java
@ConfigProperty(name = "customconfig")
CustomConfigValue value;

@Path("/email")
@GET
@Produces(MediaType.TEXT_PLAIN)
public String getEmail() {
    return value.getEmail();
}
```

尽管前面的例子并没有做什么特别的事情，但它展示了我们如何根据类定义创建一个自定义属性。

# 测试高级配置选项

在本章的`Chapter08/advanced-config/src/test`文件夹中，你会找到一个名为`AdvancedConfigTest`的测试类，它将验证我们迄今为止学到的关键概念。

要成功运行所有这些测试，请将`customconfig.properties`文件复制到你的驱动器的`/tmp`文件夹中，否则`AdvancedConfigTest`类中包含的一个断言将失败：

```java
cp Chapter08/customconfig.properties /tmp
```

然后，简单地运行`install`目标，这将触发测试的执行：

```java
mvn install
```

你应该能看到包含在`AdvancedConfigTest`中的所有测试都通过了。

# 配置配置文件

我们刚刚学习了如何使用内置转换器和，对于最复杂的情况，使用自定义转换器来创建复杂的配置。如果我们需要在不同配置之间切换，例如，从开发环境迁移到生产环境时，该怎么办？在这里，你可以复制你的配置。然而，配置文件的激增在 IT 项目中并不总是受欢迎。让我们学习如何使用**配置配置文件**来处理这个担忧。

简而言之，配置配置文件允许我们在配置中为我们的配置文件指定命名空间，这样我们就可以将每个属性绑定到同一文件中的特定配置文件。

默认情况下，Quarkus 提供了以下配置配置文件：

+   `dev`：在开发模式下运行时触发（即`quarkus:dev`）。

+   `test`：当运行测试时触发。

+   **`prod`**：当我们不在开发或测试模式下运行时被选中。

除了前面的配置文件之外，你还可以定义自己的自定义配置文件，这些配置文件将根据我们在*激活配置文件*部分中指定的规则被激活。

你可以使用以下语法将配置参数绑定到特定的配置文件：

```java
%{profile}.config.key=value
```

为了看到这个策略的实际示例，我们将通过这本书 GitHub 仓库中`Chapter08/profiles`文件夹的源代码来讲解。我们建议在继续之前将项目导入到您的 IDE 中。

让我们先检查其`application.properties`配置文件，它定义了多个配置文件：

```java
%dev.quarkus.datasource.url=jdbc:postgresql://localhost:5432/postgresDev
%test.quarkus.datasource.url=jdbc:postgresql://localhost:6432/postgresTest
%prod.quarkus.datasource.url=jdbc:postgresql://localhost:7432/postgresProd

quarkus.datasource.driver=org.postgresql.Driver
quarkus.datasource.username=quarkus
quarkus.datasource.password=quarkus

quarkus.datasource.initial-size=1
quarkus.datasource.min-size=2
quarkus.datasource.max-size=8

%prod.quarkus.datasource.initial-size=10
%prod.quarkus.datasource.min-size=10
%prod.quarkus.datasource.max-size=20
```

在先前的配置中，我们为我们的数据源连接指定了三个不同的 JDBC URL。每个 URL 都绑定到不同的配置文件。我们还为生产配置文件设置了一个特定的连接池设置，以便提供更多的数据库连接。在下一节中，我们将学习如何激活每个单独的配置文件。

# 激活配置文件

让我们以先前的配置中的`prod`配置文件为例，学习如何激活特定的配置文件。首先，我们需要启动一个名为`postgresProd`的 PostgreSQL 实例，并将其绑定到端口`7432`：

```java
docker run --ulimit memlock=-1:-1 -it --rm=true --memory-swappiness=0 --name quarkus_Prod -e POSTGRES_USER=quarkus -e POSTGRES_PASSWORD=quarkus -e POSTGRES_DB=postgresProd -e PGPORT=7432 -p 7432:7432 postgres:10.5
```

然后，我们需要在`package`阶段提供配置文件信息，如下所示：

```java
mvn clean package -Dquarkus.profile=prod -DskipTests=true
```

当运行应用程序时，它将选择在`package`阶段指定的配置文件：

```java
java -jar target/profiles-demo-1.0-SNAPSHOT-runner.jar
```

作为一种替代方法，您也可以使用`QUARKUS_PROFILE`环境变量来指定配置文件，如下所示：

```java
export QUARKUS_PROFILE=dev
java -jar target/profiles-demo-1.0-SNAPSHOT-runner.jar
```

最后，值得一提的是，相同的策略也可以用来定义非标准配置文件。例如，假设我们想要为需要在生产前进行检查的应用程序添加一个**预发布**配置文件：

```java
%staging.quarkus.datasource.url=jdbc:postgresql://localhost:8432/postgresStage
```

在这里，我们可以应用我们为其他配置文件所使用的相同策略，即我们可以在应用程序启动时使用 Java 系统属性（`quarkus-profile`）指定配置文件，或者将必要的信息添加到`QUARKUS_PROFILE`环境变量中。

# 自动配置文件选择

为了简化开发和测试，`dev`和`test`配置文件可以通过 Maven 插件自动触发。例如，如果您正在以开发模式执行 Quarkus，最终将使用`dev`配置文件：

```java
mvn quarkus:dev
```

以类似的方式，当执行测试时，例如在`install`生命周期阶段，将激活`test`配置文件：

```java
mvn quarkus:install
```

当您执行 Maven 的`test`目标时，将激活`test`配置文件。此外，值得注意的是，您可以通过`maven-surefire-plugin`的系统属性为您的测试设置不同的配置文件：

```java
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-surefire-plugin</artifactId>
<version>${surefire-plugin.version}</version>
<configuration>
    <systemPropertyVariables>
        <quarkus.test.profile>custom-test</quarkus.test.profile>
        <buildDirectory>${project.build.directory}</buildDirectory>
    </systemPropertyVariables>
</configuration>
```

在本节中，我们讲解了应用程序配置文件。在下一节中，我们将学习如何控制我们的 Quarkus 应用程序的生命周期。

# 控制应用程序生命周期

控制应用程序的生命周期是您服务能够启动外部资源或验证组件状态的一个常见要求。一个简单的策略，借鉴自 Java 企业 API，是包含**Undertow**扩展（或任何上层，如 REST 服务），这样您就可以利用`ServletContextListener`，当创建或销毁 Web 应用程序时，它会收到通知。以下是它的最小实现示例：

```java
public final class ContextListener implements ServletContextListener {

    private ServletContext context = null;

    public void contextInitialized(ServletContextEvent event) {
        context = event.getServletContext();
        System.out.println("Web application started!");

    }
    public void contextDestroyed(ServletContextEvent event) {
       context = event.getServletContext();
       System.out.println("Web application stopped!");

    }
}
```

虽然在 Quarkus web 应用程序中重用此策略是完全可以的，但建议为任何类型的 Quarkus 服务使用此方法。这可以通过观察`io.quarkus.runtime.StartupEvent`和`io.quarkus.runtime.ShutdownEvent`事件来实现。此外，在 CDI 应用程序中，你可以通过`@Initialized(ApplicationScoped.class)`限定符观察一个事件，该事件在应用程序上下文初始化时触发。这对于启动资源（如数据库）特别有用，这些资源在 Quarkus 读取配置之前是必需的。

要查看这个的实际例子，请检查这本书 GitHub 仓库中`Chapter08/lifecycle`文件夹中可用的源代码。像往常一样，建议在继续之前将项目导入到你的 IDE 中。这个例子的目的是向你展示如何在我们的客户服务中用 H2 数据库替换 PostgreSQL 数据库（[`www.h2database.com/`](https://www.h2database.com/))。

从配置开始，生命周期项目不再包含 PostgreSQL JDBC 依赖项。为了替换它，以下内容已被包含：

```java
<dependency>
  <groupId>io.quarkus</groupId>
  <artifactId>quarkus-jdbc-h2</artifactId>
</dependency>
```

为了测试我们的客户服务，我们包含了两个 H2 数据库配置配置文件：一个绑定到`dev`配置文件，另一个绑定到`test`配置文件：

```java
%dev.quarkus.datasource.url=jdbc:h2:tcp://localhost:19092/mem:test
%test.quarkus.datasource.url=jdbc:h2:tcp://localhost/mem:test
quarkus.datasource.driver=org.h2.Driver
```

要在应用程序上下文启动之前绑定 H2 数据库，我们可以使用以下`DBLifeCycleBean`类：

```java
@ApplicationScoped
public class DBLifeCycleBean {

    protected final Logger log = 
     LoggerFactory.getLogger(this.getClass());

    // H2 Database
    private Server tcpServer;

    public void observeContextInit(@Observes 
     @Initialized(ApplicationScoped.class) Object event) {
        try {
            tcpServer =  Server.createTcpServer("-tcpPort",
             "19092", "-tcpAllowOthers").start();
            log.info("H2 database started in TCP server 
            mode on Port 19092");
        } catch (SQLException e) {

            throw new RuntimeException(e);

        }
    }
    void onStart(@Observes StartupEvent ev) {
        log.info("Application is starting");
    }

    void onStop(@Observes ShutdownEvent ev) {
        if (tcpServer != null) {
            tcpServer.stop();
            log.info("H2 database was shut down");
            tcpServer = null;
        }
    }
}
```

此类能够拦截以下事件：

+   **上下文启动**：这是通过`observeContextInit`方法捕获的。数据库在这个方法中启动。

+   **应用程序启动**：这是通过`onStart`方法捕获的。我们只是在事件触发时执行一些日志记录。

+   **应用程序关闭**：这是通过`onStop`方法捕获的。我们在这个方法中关闭数据库。

现在，你可以使用以下命令以`dev`配置文件启动 Quarkus，就像平常一样：

```java
mvn quarkus:dev
```

当应用程序启动时，我们将收到通知，H2 数据库已经启动：

```java
INFO  [com.pac.qua.cha.DBLifeCycleBean] (main) H2 database started in TCP server mode on Port 19092
```

然后，我们将在应用程序启动时收到另一个通知，我们可以包括一些额外的任务来完成：

```java
[com.pac.qua.cha.DBLifeCycleBean] (main) Application is starting
```

最后，当我们停止应用程序时，资源将被取消，如下面的控制台日志所示：

```java
[com.pac.qua.cha.DBLifeCycleBean] (main) H2 database was shut down
```

在关闭数据库之前，你可以享受使用一个小型内存数据库层运行你的客户服务示例。

# 激活数据库测试资源

作为额外提示，我们将向你展示如何在测试生命周期中激活 H2 数据库。这可以通过向你的测试类添加一个标记为`@QuarkusTestResource`的类，并将`H2DatabaseTestResource`类作为属性传递来实现。

这里有一个例子：

```java
@QuarkusTestResource(H2DatabaseTestResource.class)
public class TestResources {
}
```

`H2DatabaseTestResource`基本上执行了与`DBLifeCycleBean`相同的操作，在我们启动测试之前。请注意，以下依赖项已被添加到项目中以运行前面的测试类：

```java
<dependency>
  <groupId>io.quarkus</groupId>
  <artifactId>quarkus-test-h2</artifactId>
  <scope>test</scope>
</dependency>
```

现在，您可以使用以下命令安全地针对`test`配置文件运行测试：

```java
mvn install
```

注意，在我们执行测试之前，以下日志将确认 H2 数据库已经在我们的可用 IP 地址之一上启动：

```java
[INFO] H2 database started in TCP server mode; server status: TCP server  running at tcp://10.5.126.52:9092 (only local connections)
```

外部资源引导确实是生命周期管理器的常见用例。另一个常见用例是在应用程序启动阶段安排事件。在下一节中，我们将讨论如何使用 Quarkus 的调度器触发事件。

# 使用 Quarkus 调度器触发事件

Quarkus 包含一个名为**scheduler**的扩展，可用于安排单次或重复执行的任务。我们可以使用 cron 格式来指定调度器触发事件的次数。

以下示例的源代码位于本书 GitHub 存储库的`Chapter08/scheduler`文件夹中。如果您检查`pom.xml`文件，您将注意到以下扩展已被添加到其中：

```java
<dependency>
      <groupId>io.quarkus</groupId>
      <artifactId>quarkus-scheduler</artifactId>
</dependency>
```

我们的示例项目每 30 秒生成一个随机令牌（为了简单起见，使用随机字符串）。负责生成随机令牌的类是以下`TokenGenerator`类：

```java
@ApplicationScoped
public class TokenGenerator {

    private String token;

    public String getToken() {
        return token;
    }

    @Scheduled(every="30s")
    void generateToken() {
        token= UUID.randomUUID().toString();
        log.info("New Token generated"); 
    }

}
```

现在，我们可以将我们的令牌注入到内置的 REST 端点中，如下所示：

```java
@Path("/token")
public class Endpoint {

    @Inject
    TokenGenerator token;

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String getToken() {

        return token.getToken();
    }
}
```

使用以下命令以常规方式启动应用程序：

```java
mvn quarkus:dev
```

您将注意到，每 30 秒，以下消息将在控制台日志中打印：

```java
[INFO] New Token generated
```

然后，通过请求`/token` URL，将返回随机生成的字符串：

```java
curl http://localhost:8080/token
 3304a8de-9fd7-43e7-9d25-6e8896ca67dd
```

# 使用 cron 调度器格式

除了使用时间表达式（**s=seconds**，**m=minutes**，**h=hours**，**d=days**）外，您还可以选择更紧凑的 cron 调度器表达式。因此，如果您想每秒触发事件，则可以使用以下 cron 表达式：

```java
@Scheduled(cron="* * * * * ?")
void generateToken() {
    token= UUID.randomUUID().toString();
    log.info("New token generated");
}
```

检查 cron 主页面以获取有关 cron 格式的更多信息：[`man7.org/linux/man-pages/man5/crontab.5.html`](http://man7.org/linux/man-pages/man5/crontab.5.html).

# 触发一次性事件

如果您需要执行一次性事件，则可以直接将`io.quarkus.scheduler.Scheduler`类注入到您的代码中，并使用`startTimer`方法，该方法将在单独的线程中触发动作的执行。这可以在以下示例中看到：

```java
@Inject
Scheduler scheduler;

public void oneTimeEvnt() {

    scheduler.startTimer(300, () -> oneTimeAction());

}

public void oneTimeAction() {
    // Do something
}
```

在这个简短的摘录中，我们可以看到单个事件，该事件将在`oneTimeAction()`方法中执行，将在`300`毫秒后触发一次性动作。

# 摘要

在本章中，我们介绍了一些我们可以使用的高级技术，以使用转换器和配置配置文件来管理我们的配置。我们还演示了如何注入不同的配置源，并优先于标准配置文件。在本章的第二部分，我们探讨了如何捕获应用程序生命周期的事件以及如何安排未来任务的执行。

为了使我们的应用更加可扩展，在下一章中，我们将讨论如何构建响应式应用，这些应用是事件驱动的且非阻塞的。请系好安全带！
