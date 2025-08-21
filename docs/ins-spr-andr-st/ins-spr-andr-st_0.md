# 第一章：Instant Spring for Android Starter

欢迎使用 Instant *Spring for Android Starter*。本书特别为您提供了所有您需要的信息，以便使用 Spring for Android 轻松开发 Android 应用程序。您将学习 Spring for Android 提供的不同功能，并开始使用这些功能构建您的第一个应用程序。

本书包含以下部分：

*那么，Spring for Android 是什么？*描述了 Spring for Android 是 Spring 框架的扩展，有助于简化本机 Android 应用程序的开发；它目前（版本 1.0.0.RELEASE）具有 REST 客户端（RestTemplate）和 OAuth 支持（Spring Social）。

*在 Android 应用程序中集成*教您如何将 Spring for Android 与 Eclipse IDE 和 ADT（Android 开发工具）以及 Maven 集成。

*快速入门-Rest Template*将向您展示如何使用 Spring for Android Rest Template 模块的一些核心功能；例如，创建 REST 客户端。在本节结束时，您将能够使用消息转换器和/或 HTTP 基本身份验证检索，更新，创建和删除 REST 资源。

*快速入门-OAuth*将向您展示如何使用 Spring for Android Auth 模块的一些核心功能；例如，创建与 Google+的 OAuth 连接以读取用户配置文件。在本节结束时，您将能够建立 OAuth 连接以验证用户和应用程序。

*您应该了解的人和地方* - 每个开源项目都围绕一个社区。本节为您提供了许多有用的链接到项目页面和论坛，以及许多有用的文章，教程和博客，另外还有 Spring for Android 超级贡献者的 Twitter 动态。

# 那么，Spring for Android 是什么？

在本节中，我们将介绍 Spring for Android 的主要方面：`RestTemplate`，`Auth`以及 Spring for Android 不是什么。

## RestTemplate

`RestTemplate`模块是基于 Java 的 REST 客户端`RestTemplate`的移植，最初出现在 Spring for MVC 中的 2009 年。与其他 Spring 模板对应物（`JdbcTemplate`，`JmsTemplate`等）一样，它的目标是为 Java 开发人员（因此也是 Android 开发人员）提供较低级别 Java API 的高级抽象；在这种情况下，它简化了 HTTP 客户端的开发。

在其 Android 版本中，`RestTemplate`依赖于核心 Java HTTP 设施（`HttpURLConnection`）或 Apache HTTP Client。根据您用于运行应用程序的 Android 设备版本，`RestTemplate` for Android 可以为您选择最合适的设施。这是根据 Android 开发人员的建议。

### 注意

请参阅[`android-developers.blogspot.ca/2011/09/androids-http-clients.html`](http://android-developers.blogspot.ca/2011/09/androids-http-clients.html)。这篇博客文章解释了为什么在某些情况下，Apache HTTP Client 优先于`HttpURLConnection`。

`RestTemplate` for Android 还支持 gzip 压缩和不同的消息转换器，以将您的 Java 对象从 JSON，XML 等转换为 JSON，XML 等。

## Auth/Spring Social

Spring Android `Auth`模块的目标是让 Android 应用程序使用**OAuth**（版本 1 或 2）获得对 Web 服务提供程序的授权。

OAuth 可能是最流行的授权协议（值得一提的是，它是一个开放标准），目前被 Facebook，Twitter，Google 应用程序（以及许多其他应用程序）用于让第三方应用程序访问用户帐户。

Spring for Android `Auth`模块基于几个 Spring 库，因为它需要安全地（通过加密）持久化（通过 JDBC）通过 HTTP 获取的令牌；以下是 OAuth 所需的库列表：

+   **Spring Security Crypto**：用于加密令牌

+   Spring Android OAuth：这扩展了`Spring Security Crypto`，增加了一个专门为 Android 设计的加密器，并基于 SQLite 的持久性提供程序

+   Spring Android Rest Template：与 HTTP 服务交互

+   **Spring Social Core**：OAuth 工作流抽象化

在执行 OAuth 工作流时，我们还需要浏览器将用户带到服务提供商的身份验证页面，例如，以下是 Twitter OAuth 身份验证对话框：

![Auth/Spring Social](img/1905_01_02.jpg)

## Spring for Android 不是什么

SpringSource（Spring for Android 背后的公司）在 Java 开发人员中非常有名。他们最受欢迎的产品是 Spring Framework for Java，其中包括依赖注入框架（也称为控制反转框架）。Spring for Android 并没有将控制反转带到 Android 平台。

在其第一个版本（1.0.0.M1）中，Spring for Android 为 Android 带来了一个通用的日志门面；作者在下一个版本中将其移除了。

# 将其集成到 Android 应用程序中

Spring for Android 包含在多个 JAR 库中，这些库应该链接到项目中。这些 JAR 库不是标准的 Android 发行版的一部分。例如，我们将需要以下 JAR 库来使用 Spring For Android 的`RestTemplate`来消费 JSON REST API：

![将其集成到 Android 应用程序中](img/1905_02_01.jpg)

## 添加或使用 RestTemplate 的最小依赖关系

您可以使用 IDE 手动满足依赖关系（及其传递依赖关系）。手动依赖管理的替代方法是使用**Maven**进行自动项目构建。我们将在后面的部分中解释 Maven 构建。

## 使用 Eclipse 和 Spring for Android 入门

Eclipse 无疑是开发 Android 应用程序的最流行的 IDE；其中一个原因是谷歌维护的**Android 开发工具**（**ADT**）提供了 Eclipse 插件来简化 Android 开发（调试器，自定义 XML 编辑器等）。

### 注意

IntelliJ IDEA 社区版提供了对 Android 的开箱即用支持；Netbeans 也允许您安装`nbandroid`插件，以便简化 Android 应用程序开发。

我们需要执行以下步骤才能开始使用 Eclipse：

1.  从[`www.eclipse.org/downloads`](http://www.eclipse.org/downloads)下载最新版本（ADT 与版本 3.6.2 的 Eclipse 兼容；在撰写本文时，4.2 Juno 是最新版本）。最好选择*Java 开发人员的 Eclipse IDE*而不是其他可用版本。

1.  一旦您在计算机上下载并解压它，就可以启动它。选择一个工作区位置（您的项目将位于其中）并安装 ADT 插件：单击**帮助** | **Eclipse Marketplace...**，然后在文本框中键入**adt**（如下截图所示），然后按*Enter*；现在通过单击**安装**按钮选择**Eclipse 的 Android 开发工具**。

### 注意

您不必选择 NDK 支持功能，该功能为原生 Android 开发（使用 C 或 C++语言）提供支持，因为 Spring For Android 库只是 Java 库。

![使用 Eclipse 和 Spring for Android 入门](img/1905_02_02.jpg)

### 安装 ADT 插件

Eclipse 将多次提示您许可证，并最终要求您重新启动它。

1.  当您回到工作区时，请确保您的计算机上安装了（最新的）Android SDK：单击具有 Android 机器人从盒子中出来的图标，并安装或更新 Android SDK（您不需要安装所有 Android 版本，您只需安装最流行的版本，如 2.2 即 Froyo，2.3.3 即 Gingerbread，4.0.3 即 Ice Cream Sandwich 和 4.1 即 Jelly Bean）；完成后重新启动 Eclipse。![安装 ADT 插件](img/1905_02_03.jpg)

Android SDK 管理器用于管理 Android 工具和平台

1.  如果您还没有这样做，您还需要创建一个**Android 虚拟设备**（**AVD**），以便您可以将 Android 应用程序部署到其中（您也可以通过 USB 连接手机或平板电脑将应用程序部署到其中）；为此，请单击代表 Android 机器人的图标，然后在设备屏幕上创建一个新的 AVD，如下面的屏幕截图所示：![安装 ADT 插件](img/1905_02_04.jpg)

创建新 AVD

### 注意

您可以在官方网站上找到有关 ADT 插件的更多详细信息：[`developer.android.com/tools/sdk/eclipse-adt.html`](http://developer.android.com/tools/sdk/eclipse-adt.html)。

1.  最后，我们需要下载 Spring for Android JAR 文件，转到 Spring for Android 网站：[`www.springsource.org/spring-android`](http://www.springsource.org/spring-android)并单击**下载**按钮（您可以跳过以下屏幕中的注册）。现在选择最新的 Spring for Android 版本（在撰写本文时为 1.0.1.RELEASE）并将其解压缩到您的计算机上；我们将在下一步中需要这些库：![安装 ADT 插件](img/1905_02_04_bis.jpg)

Spring for Android 官方下载页面

### 第一个示例应用程序

现在我们准备使用 Spring for Android 创建我们的第一个应用程序：

1.  单击**文件** | **新建...** | **Android 应用程序**，输入项目名称并在以下对话框中接受默认值：![第一个示例应用程序](img/1905_02_05.jpg)

1.  将弹出名为**新 Android 应用程序**的新窗口，如下面的屏幕截图所示：![第一个示例应用程序](img/1905_02_05.jpg)

从 Eclipse 创建 Android 应用程序

1.  在要求新活动时，选择默认的**BlankActivity**，如下面的屏幕截图所示：![第一个示例应用程序](img/1905_02_05-bis.jpg)

1.  现在从您之前下载的 Spring for Android ZIP 文件中复制 JAR 文件`spring-android-core-{version}.jar`和`spring-android-rest-template-{version}.jar`到新项目的`$Project_home/libs`文件夹中；您应该有以下项目结构：![第一个示例应用程序](img/1905_02_06.jpg)

第一个示例应用程序

1.  对于这个使用 Spring for Android 请求 Web 服务的第一个示例应用程序（我们将使用[`ifconfig.me/all`](http://ifconfig.me/all)），我们只需要在此 URL 上执行`GET`方法，然后我们将收到客户端信息：IP，用户代理等。然后，我们需要声明访问网络的意图。通过在 Android 清单文件中的`application`标签之前添加以下`permission`标签来实现这一点：

```java
(...)<uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="15" />
<uses-permission android:name="android.permission.INTERNET"/>
<application (...)
```

### 提示

Android 清单文件位于项目的根目录：`AndroidManifest.xml`。

忘记这样做会导致日志视图中出现晦涩的消息（其中收集了所有日志），例如`Fatal Exception`；有关权限的更多信息，请参见[`developer.android.com/guide/topics/security/permissions.html`](http://developer.android.com/guide/topics/security/permissions.html)。

HTTP 协议定义了方法或动词，以指示对远程资源执行的操作：`GET`用于获取资源，`POST`用于存储资源，`PUT`用于更新资源，`DELETE`用于删除资源是 HTTP 动词的示例，您可以通过阅读本文了解更多信息：[`en.wikipedia.org/wiki/Hypertext_Transfer_Protocol`](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)。

1.  然后，通过删除默认的“hello world”（`android:text="@string/hello_world"`）并将其替换为我们将用于打印 Web 服务响应的锚点（`android:id="@+id/hello_text"`）来调整活动布局：

```java
<RelativeLayout 

    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <TextView
        android:id="@+id/result_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"
        android:layout_centerVertical="true"
        tools:context=".MainActivity" />
</RelativeLayout>
```

活动布局位于`res/layout/activity_main.xml`。

1.  最后，我们可以重新设计`MainActivity`本身（只需更新`onCreate()`方法）：

```java
  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    final TextView resultTextView = 
            (TextView) findViewById(R.id.result_text);
    AsyncTask<String, Void, String> simpleGetTask =  
                new AsyncTask<String, Void, String>() {
      @Override
      protected String doInBackground(String... params) {
        //executed by a background thread

       //create a new RestTemplate instance
        RestTemplate restTemplate = new RestTemplate();

        //add the String message converter, since the result of
             // the call will be a String
        restTemplate.getMessageConverters().add(
                     new StringHttpMessageConverter());

        // Make the HTTP GET request on the url (params[0]),
             // marshaling the response to a String
        return
                restTemplate.getForObject(params[0],String.class);
        }

      @Override
      protected void onPostExecute(String result) {
        // executed by the UI thread once the background 
             // thread is done getting the result
        resultTextView.setText(result);
      }
    };
    String url = "http://ifconfig.me/all";
    // triggers the task; it will update the resultTextView once
      // it is done
    simpleGetTask.execute(url);
  }
```

### 注意

如果 Eclipse 抱怨缺少导入项，请同时按下*Shift* + *Ctrl* + *O*以自动添加所需的导入项。

在这段代码中，我们首先获取了对`result_text`文本视图的引用，并将其设置为名为`resultTextView`的`final`变量（`final`是因为我们需要通过内部类访问它）。

接下来，我们创建了一个匿名内部类，扩展了`AsyncTask`以实现访问 web 服务的所有逻辑（创建`RestTemplate`实例，添加`String`转换器并调用`getForObject`），并将结果设置为文本视图（使用`setText`方法）。当我们调用`simpleGetTask.execute(url)`时，URL 被添加到`doInBackground(String... params)`中的参数数组中（所以在这种情况下`params[0]`是`url`）。

### 注意

如果我们直接在`onCreate()`方法中编写了用于访问 web 服务的代码，应用程序甚至无法启动，因为 Android 平台阻止开发人员在 UI（主）线程中执行 HTTP 请求（这是有道理的：这些与网络相关的操作通常需要时间来完成，并且在操作进行中会冻结 UI）。

```java
@Override
public void onCreate(Bundle savedInstanceState) {
 TextView resultTextView = (TextView) findViewById(R.id.result_text);
 String url = "http://ifconfig.me/all";
 RestTemplate restTemplate = new RestTemplate();
 RestTemplate.getMessageConverters().add(new  StringHttpMessageConverter());
 String result = restTemplate.getForObject(url, String.class);
 resultTextView.setText(result);
}
```

### 提示

不要这样做！您不能从 UI（主）线程执行与 HTTP 相关的调用。

### 注意

有关`AsyncTasks`的更多信息可以在官方文档中找到：[`developer.android.com/reference/android/os/AsyncTask.html`](http://developer.android.com/reference/android/os/AsyncTask.html)，还有关于 Android 中的线程和进程的一般信息：[`developer.android.com/guide/components/processes-and-threads.html`](http://developer.android.com/guide/components/processes-and-threads.html)。

1.  我们现在准备启动这个第一个基于 Spring for Android 的应用程序！

右键单击项目名称，然后选择**运行为...** | **Android 应用程序**：

![第一个示例应用程序](img/1905_02_06-bis.jpg)

1.  Eclipse 将启动您之前创建的 AVD，并将自动运行应用程序，您应该看到类似于以下截图：![第一个示例应用程序](img/1905_02_07.jpg)

AVD 中的 MainActivity 截图

恭喜！您已成功运行了基于 Spring for Android 的第一个应用程序！

## 使用 Maven 构建 Spring for Android 应用程序

在上一个示例中，我们只向`libs`文件夹中添加了两个 JAR 文件；但是，如果您需要其他依赖项，例如`MessageConverters`（用于解组 JSON、XML 响应），`Auth`模块或项目中的任何其他现有库，管理您的依赖项很快就会成为一种负担！

Maven 可以帮助您管理依赖项（简而言之，您在名为`pom.xml`的文件中指定它们，它将自动从互联网获取它们，包括它们的传递依赖项）以及更多内容：发布操作、测试运行、头文件生成等。Maven 是一个模块化的开源构建工具，拥有大量插件！

### 注意

使用 Android Maven 插件时，您应该将以下变量添加到您的环境中`ANDROID_HOME`（链接到您的 Android 主目录），并将`ANDROID_HOME/platform-tools`添加到您的路径中。这在[`code.google.com/p/maven-android-plugin/wiki/GettingStarted`](https://code.google.com/p/maven-android-plugin/wiki/GettingStarted)中有详细说明。

首先，让我们向 Eclipse 添加**m2e**（Maven 到 Eclipse）插件：就像我们为 ADT 插件所做的那样，打开**帮助** | **Eclipse**市场并搜索**maven**；选择**Maven Integration for Eclipse**，点击**安装**并确认对话框；在过程结束时重新启动您的 IDE。

![使用 Maven 构建 Spring for Android 应用程序](img/1905_02_08.jpg)

安装 Maven Integration for Eclipse

现在将名为`spring-for-android-first-example-maven`的项目导入到您的工作区：您可以从 GitHub 克隆它，也可以解压`examples.zip`存档；然后，从 Eclipse 中，点击**文件** | **导入...**，选择**现有的 Maven 项目**，然后点击**下一步**。

![使用 Maven 构建 Spring for Android 应用程序](img/1905_02_09.jpg)

导入现有的 Maven 项目

浏览到你克隆或解压项目的目录，并点击**下一步**按钮：

![使用 Maven 构建 Spring for Android 应用程序](img/1905_02_10.jpg)

浏览到你的 Maven 项目

M2e 将显示一个需要集成基于 Maven 的项目到你的 IDE 的 m2e 连接器列表（基本上这些连接器告诉 m2e 如何生成和编译源代码）；点击**完成**并接受以下对话框；重新启动 Eclipse。

### 注意

这里最重要的连接器是 m2e-android 连接器：它使 m2e 能够与 ADT 集成，更多信息请参阅[`rgladwell.github.com/m2e-android/`](http://rgladwell.github.com/m2e-android/)。

项目应该可以成功导入，你可以像之前的项目一样在 AVD 上部署项目（右键单击项目，选择**运行为** | **Android 应用程序**）。

让我们看看`pom.xml`文件（与之前的项目相比唯一的额外文件），右键单击它，选择**使用...打开** | **Maven POM 编辑器**；在视图底部点击**pom.xml**选项卡。

```java
(...)<dependencies>
        <dependency>
            <groupId>com.google.android</groupId>
            <artifactId>android</artifactId>
            <version>4.0.1.2</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
          <groupId>org.springframework.android</groupId>
          <artifactId>spring-android-rest-template</artifactId>
          <version>${spring-android-version}</version>
        </dependency>
    </dependencies>
    <build>
        <finalName>${project.artifactId}</finalName>
        <sourceDirectory>src</sourceDirectory>
        <plugins>
            <plugin>
        <groupId>com.jayway.maven.plugins.android.generation2</groupId>
                <artifactId>android-maven-plugin</artifactId>
                <version>3.3.0</version>
                <extensions>true</extensions>
                <configuration>
                    <sdk>
                        <!-- platform or api level (api level 4 = platform 1.6)-->
                        <platform>15</platform>
                    </sdk>
                </configuration>
            </plugin>
        </plugins>
    </build>(...)
```

在依赖部分，你可以看到 Android 平台和 Spring for Android 的`RestTemplate`工件（不需要指定 Spring for Android 核心！记住，Maven 会自动获取传递依赖项），在插件部分 Android-Maven 插件（通过`extensions=true`属性扩展了 Android 项目的 Maven，配置为需要 Android 4.0.3（API 级别 15））。

Maven 在其核心是一个命令行工具，如果你在你的机器上下载并解压 Maven，你可以转到 Maven 项目的根目录（`pom.xml`所在的位置）并使用不同的目标输入 Maven 命令：

+   `mvn clean install`：编译，运行测试和打包

+   `mvn clean install android:run`：在 AVD 中启动你的应用程序

### 注意

要查看所有可用的目标，你可以输入`mvn help:describe -Dplugin=com.jayway.maven.plugins.android.generation2:maven-android-plugin`。

你还可以在 Eclipse 内模拟命令行 Maven，右键单击项目，选择**运行为** | **Maven 构建...**，并指定你的目标。

Maven 安装说明可以在网上找到：[`www.sonatype.com/books/mvnref-book/reference/installation-sect-maven-install.html`](http://www.sonatype.com/books/mvnref-book/reference/installation-sect-maven-install.html)。

要了解更多关于 Maven 的信息，我强烈建议阅读免费的在线 Maven 书籍，甚至有一个专门的 Android 章节：[`www.sonatype.com/books/mvnref-book/reference/`](http://www.sonatype.com/books/mvnref-book/reference/)。

现在我们准备一起创建一些更丰富的基于 Spring for Android 的应用程序！

# 快速开始 - RestTemplate

现在我们的工作空间已经设置好，并且我们已经成功部署了我们的第一个基于 Spring for Android `RestTemplate`的应用程序，让我们来探索一些`RestTemplate`的更多功能。

这一部分依赖于几个示例应用程序。我们将逐个浏览每个代码示例的所有重要点。随时导入项目到你的 IDE 并浏览代码。

### 注意

从现在开始，我将使用 Maven 来管理依赖关系，但你不必这样做。每个项目在其`README.md`文件中都有一个依赖项列表：下载它们（或者从你的 Spring for Android ZIP 存档`libs`文件夹中获取），并将它们添加到`libs`文件夹中！

## 不同的消息转换器

在上一节中，我们已经使用了一个消息转换器：`StringHttpMessageConverter`。每个消息转换器都支持从特定媒体类型读取和写入（通常称为 MIME 类型）；`StringHttpMessageConverter`支持读取各种文档（MIME */*）并写入文本（MIME text/plain）。让我们来了解更复杂的消息转换器。

### 注意

**MIME**（**多用途互联网邮件扩展**）不仅用于描述电子邮件的内容，还广泛用于描述通过 HTTP 交换的文档的内容（[`en.wikipedia.org/wiki/MIME`](http://en.wikipedia.org/wiki/MIME)）。

### MappingJacksonHttpMessageConverter

JSON 消息在 REST 网络服务中非常常见。最初设计用于 JavaScript 语言（`JavaScriptObjectNotation`）；它们既轻量又易于阅读。

`Ifconfig.me`可以返回一个 JSON 响应，如果你使用这个 URL：[`ifconfig.me/all.json`](http://ifconfig.me/all.json)。

在我们第一个示例的代码中，让我们用`MappingJacksonHttpMessageConverter`替换`StringHttpMessageConverter`：

```java
AsyncTask<String, Void, IfConfigMeJson> simpleGetTask =  
  new AsyncTask<String, Void, IfConfigMeJson>() {
  @Override
  protected IfConfigMeJson doInBackground(String... params) {
    String url = params[0];
    RestTemplate restTemplate = new RestTemplate();
    MappingJacksonHttpMessageConverter jacksonConverter = 
         new MappingJacksonHttpMessageConverter();
    restTemplate.getMessageConverters().add(jacksonConverter);
    return restTemplate.getForObject(url,
           IfConfigMeJson.class);
  }

  @Override
  protected void onPostExecute(IfConfigMeJson result) {
    String resultAsString =  new StringBuilder()
                      .append("Your current IP is : ")
                      .append(result.getIpAddr()).toString();
    resultTextView.setText(resultAsString );
  }

};

String url = "http://ifconfig.me/all.json";
//triggers the task
simpleGetTask.execute(url);
```

正如你所看到的，我们还引入了一个名为`IfConfigMeJson`的模型类：这个类定义了 JSON 消息属性（`{"connection":"keep-alive","ip_addr":"70.30.43.43", [..]}`）和**POJO**（**Plain Old Java Object**：一个简单的类，带有成员变量和它们的访问器）之间的映射，这样我们就可以在我们的 Java 代码中轻松访问响应的每个属性（在下面的示例中：`result.getIpAddr()`）：

```java
public class IfConfigMeJson {
  private String connection;
  @JsonProperty("ip_addr")
  private String ipAddr;
  public String getConnection() {
    return connection;
  }
  public void setConnection(String connection) {
    this.connection = connection;
  }
  public String getIpAddr() {
    return ipAddr;
  }
  public void setIpAddr(String opAddr) {
    this.ipAddr = opAddr;
  }
```

### 注意

Jackson 会自动将 JSON 属性与 Java 字段匹配，只要它们的命名相同（Java 中的驼峰命名法，JSON 中的下划线）。你肯定已经注意到，我们将 JSON 属性`ip_addr`映射到字段`ipAddr`，以遵守 Java 的命名约定，这要归功于 Jackson 在字段定义之前的`@JsonProperty`注解。另外，我们也可以将类的字段声明为 public，并移除访问器。

当你运行这个应用程序时，你只会看到以下截图：

![MappingJacksonHttpMessageConverter](img/1905_03_01.jpg)

使用`MappingJacksonHttpMessageConverter`的应用程序

在这一部分，我们在`pom.xml`文件中添加了对`jackson-mapper-asl`的依赖：

```java
<dependency>
<groupId>org.codehaus.jackson</groupId>
  <artifactId>jackson-mapper-asl</artifactId>
  <version>${jackson-version}</version>
</dependency>
```

### 注意

还有另一个可用的 JSON 消息转换器，`GsonHttpMessageConverter`，它使用 Google Gson 库而不是 Jackson，可以作为替代使用。

### SimpleXmlHttpMessageConverter

另一个值得一提的消息转换器是`SimpleXmlHttpMessageConverter`：它将 XML 响应和请求映射到 POJO，就像`MappingJacksonHttpMessageConverter`将 JSON 做的那样。

举个例子，让我们请求`Ifconfig.me`的 XML 服务：[`ifconfig.me/all.xml`](http://ifconfig.me/all.xml)。

在我们之前示例中的`MainActivity`类中，让我们用`SimpleXmlHttpMessageConverter`替换`MappingJacksonHttpMessageConverter`：

```java
AsyncTask<String, Void, IfConfigMeXml> simpleGetTask =  
   new AsyncTask<String, Void, IfConfigMeXml>() {
   @Override
   protected IfConfigMeXml doInBackground(String... params) {
      String url = params[0];
RestTemplate restTemplate = new RestTemplate();
      SimpleXmlHttpMessageConverter xmlConverter = 
           new SimpleXmlHttpMessageConverter();
      restTemplate.getMessageConverters().add(xmlConverter);
      return restTemplate.getForObject(url, IfConfigMeXml.class);
   }

   @Override
   protected void onPostExecute(IfConfigMeXml result) {
       String resultAsString =  new StringBuilder()
                  .append("Your current IP is : ")	
                  .append(result.getIpAddr()).toString();
       resultTextView.setText(resultAsString );
   }

};

String url = "http://ifconfig.me/all.xml";

```

你会注意到我们再次依赖于一个名为`IfConfigMeXml.java`的类，让`SimpleXml`能够在 XML 响应和 Java 代码之间进行映射。以下是 XML 响应：

```java
<info>
<forwarded/>
<ip_addr>70.43.43.43</ip_addr>
[...]
</info>
```

以下是 Java 代码：

```java
@Root (name="info")
public class IfConfigMeXml {

  @Element(required=false)
  private String forwarded;

  @Element(name="ip_addr")
  private String ipAddr;
  [...]
}
```

这些注解与我们用于 JSON 消息转换器的注解类似。

`@Root`注解定义了根 XML 标签的名称：在这种情况下是`info`。

在每个字段之前添加`@Element`注解，让`SimpleXml`知道这些字段被映射到 XML 标签：如果`required`设置为 false，意味着标签可以为空；如果指定了`name`，意味着 POJO 字段与 XML 标签名称不匹配。

如果你运行这个应用程序，你将得到与之前示例中完全相同的输出。

### 注意

在这一部分，我们添加了对`SimpleXml`框架的依赖。不幸的是，这个依赖有传递依赖项到 stax 和 xpp3，这些对于 Android 应用程序是不需要的。

我们必须明确地过滤一些传递依赖项，以便让 Maven 知道不要将它们添加到类路径中。

```java
<dependency>
  <groupId>org.simpleframework</groupId>
  <artifactId>simple-xml</artifactId>
  <version>${simple-version}</version>
  <exclusions>
    <!-- StAX is not available on Android -->
    <exclusion>
      <artifactId>stax</artifactId>
      <groupId>stax</groupId>
    </exclusion>
    <exclusion>
      <artifactId>stax-api</artifactId>
      <groupId>stax</groupId>
    </exclusion>
    <!-- Provided by Android -->
    <exclusion>
      <artifactId>xpp3</artifactId>
      <groupId>xpp3</groupId>
    </exclusion>
  </exclusions>
</dependency>
```

### RssChannelHttpMessageConverter

这个消息转换器用于解析 RSS 订阅源；和往常一样，我们将它注入到我们的`RestTemplate`实例中，但这次我们将从博客订阅源中读取条目。

```java
final WebView resultTextView = (WebView) findViewById(R.id.result_text);
AsyncTask<String, Void, Channel> simpleGetTask =  new AsyncTask<String, Void, Channel>() {
   @Override
   protected Channel doInBackground(String... params) {
      RestTemplate restTemplate = new RestTemplate();
      // Configure the RSS message converter.
       RssChannelHttpMessageConverter rssChannelConverter = new RssChannelHttpMessageConverter();
      rssChannelConverter.setSupportedMediaTypes(
         Collections.singletonList(MediaType.TEXT_XML));

        // Add the RSS message converter to the RestTemplate instance
        restTemplate.getMessageConverters().add(rssChannelConverter);

      // Make the HTTP GET request on the url (params[0]), marshaling //the response to a String
      return restTemplate.getForObject(params[0], Channel.class);
   }

   @Override
   protected void onPostExecute(Channel result) {
      //get the latest article from the blog
      Item item = (Item) result.getItems().get(0);

      // load the content of the article into the WebView
      resultTextView.loadData(item.getContent().getValue(), "text/html", "UTF-8");
   }
};
String url = "http://blog.dahanne.net/feed/";
```

在这里我们不必创建一个 POJO 映射类，因为结果将始终是一个`Channel` - 一个提供我们访问 feed 的不同属性的方法的类：items, language, published date 等等。在这个例子中，我们只读取第一项的内容 - 我博客的最新文章！

该 feed 如下所示：

```java
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0">
<channel>
   <title>Anthony Dahanne's blog</title>
   <description>A blog</description>
   [...]
   <item>
      <title>Article 1</title>
      <description>content of the article</description>
      [...]
   </item>
   [...]
</channel>
</rss>
```

设备上的结果如下：

![RssChannelHttpMessageConverter](img/1905_03_02.jpg)

RSS feed 的第一项内容

### 注意

你可能已经注意到，我将之前的 TextView 替换为了一个 WebView，能够解释 HTML 代码，比如超链接。

`RssChannelHttpMessageConverter`没有默认支持的类型，这就是为什么我们添加了媒体类型`text/xml`作为支持的类型。

### 注意

我们必须为这个 RSS feed 示例添加对 Android ROME Feed Reader 的依赖；这个库没有发布在 Maven Central 上，这意味着我们还必须声明其构件部署的仓库。

```java
<dependency>
   <groupId>com.google.code.android-rome-feed-reader</groupId>
   <artifactId>android-rome-feed-reader</artifactId>
   <version>${android-rome-version}</version>
</dependency>

<repositories>
   <repository>
      <id>android-rome-feed-reader-repository</id>
      <name>Android ROME Feed Reader Repository</name>
      <url>https://android-rome-feed-reader.googlecode.com/svn/maven2/releases</url>
   </repository>
</repositories>
```

## Gzip 加密

为了在传输过程中节省带宽，作为每个请求的一部分，你可以告诉服务器你支持 gzip 加密。如果服务器支持对响应进行 gzip 压缩，它将这样做。

Spring for Android `RestTemplate` 依赖于 Android `java.net.HttpURLConnection` 自 Gingerbread（2.3）以来，默认发送 `Accept-Encoding : gzip` 头部；因此 gzip 压缩默认开启。

否则，你只需要在请求的头部中指定你支持 gzip。让我们看一个具体的例子 - Jackson 例子，这次我们只展示客户端支持的编码机制：

```java
@Override
protected IfConfigMeJson doInBackground(String... params) {

   HttpHeaders requestHeaders = new HttpHeaders();
   requestHeaders.setAcceptEncoding(ContentCodingType.IDENTITY);
// Add the gzip Accept-Encoding header   
//requestHeaders.setAcceptEncoding(ContentCodingType.GZIP);
   HttpEntity<?> requestEntity = 
        new HttpEntity<Object>(requestHeaders);
   RestTemplate restTemplate = new RestTemplate();
   MappingJacksonHttpMessageConverter jacksonConverter = 
        new MappingJacksonHttpMessageConverter();
   restTemplate.getMessageConverters().add(jacksonConverter);
   ResponseEntity<IfConfigMeJson> response = 
        restTemplate.exchange(params[0], HttpMethod.GET, 
                      requestEntity, IfConfigMeJson.class);
   return response.getBody();
}
```

使用`Identity`，我们声明我们不支持除了原始文本之外的任何其他东西。

### 注意

当你明确或隐式地要求 gzip 时，`java.util.zip.GzipInputStream`将自动用于解码响应。

你是否注意到，我们使用`exchange(url,GET,requestEntity,Pojo.class)`而不是`getForObject(url,Pojo.class)`？`exchange()`方法是一个更灵活的方法，允许对请求进行自定义。

让我们看看在执行此代码期间交换的请求/响应：

| 请求（IDENTITY 然后 GZIP） | 响应（IDENTITY 然后 GZIP） |
| --- | --- |
| `GET/all.json HTTP/1.1``Accept: application/json``Accept-Encoding: identity``Content-Length: 0``Host: ifconfig.me:80``Connection: Keep-Alive` | `HTTP/1.1 200 OK``Date: Fri, 02 Nov 2012 02:41:04 GMT``Server: Apache``Vary: Accept-Encoding``Connection: close``Transfer-Encoding: chunked``Content-Type: application/json``13c``{"connection":"Keep-Alive","ip_addr:[...],"encoding":"identity"}``0` |
| `GET /all.json HTTP/1.1``Accept: application/json``Accept-Encoding: gzip``Content-Length: 0``User-Agent: Dalvik/1.6.0 (Linux; U; Android 4.0.4; sdk Build/MR1)``Host: ifconfig.me:80``Connection: Keep-Alive``X-Forwarded-For: 192.168.1.111` | `HTTP/1.1 200 OK` `—îPq``Date: Fri, 02 Nov 2012 02:42:08 GMT``Server: Apache``Vary: Accept-Encoding``Content-Encoding: gzip``Content-Length: 258``Connection: close``Content-Type: application/json``­ã/%êAOÑ0 ÖÔ˛``"ì&À–"ß5ﬁ'ãâg2€él•¥§4�ˇªÖΩÕ&#124;3ÔeÊ˝2È¨%9kgŸ/{&ö"ì—+±/"SáJ˘àk` |

### 注意

经常分析设备之间来回传输的请求和响应是很方便的；你可以使用开源软件 Membrane 作为 HTTP 代理（[`www.membrane-soa.org/soap-monitor/`](http://www.membrane-soa.org/soap-monitor/)）创建一个新的简单代理，监听 localhost:2000 并重定向到你目标的 web 服务（例如，[`ifconfig.me`](http://ifconfig.me)）。

在 Android 代码中，只需用`computer_ip:2000`（可以是`192.168.1.1:2000`）替换你的 web 服务 URL：然后你就可以看到你的 Android 应用和 web 服务 URL 之间的所有流量。

## HTTP 基本身份验证

这种身份验证方案，正如其名称所暗示的那样，非常基本和不安全。你只需提供用户名和密码来访问一个领域（通常是网站的一部分）。

就像我们为编码所做的那样，我们可以通过标头提供凭据：用户名和密码将被 Base64 编码。

### 注意

Base64 编码绝对不提供安全性。您可以像编码一样轻松地解码它！有关 Base64 的更多信息，请参见[`en.wikipedia.org/wiki/Base64`](https://en.wikipedia.org/wiki/Base64)。

例如，我们将尝试访问显示**Hello World**的网页；假设您发送了正确的凭据。 该页面是[`restfulapp.appspot.com/helloworld`](http://restfulapp.appspot.com/helloworld)，凭据是用户名-`s4a`和密码-`s4a`（您可以在 Web 浏览器中尝试）。

让我们尝试使用 Spring for Android 的`RestTemplate`访问此页面！

```java
AsyncTask<String, Void, String> simpleGetTask =  new AsyncTask<String, Void, String>() {
   @Override
   protected String doInBackground(String... params) {
      // Set the credentials for creating a Basic Auth request
      HttpAuthentication authHeader = 
          new HttpBasicAuthentication("s4a", "s4a");
      HttpHeaders requestHeaders = new HttpHeaders();
      requestHeaders.setAuthorization(authHeader);
      HttpEntity<?> requestEntity = 
          new HttpEntity<Object>(requestHeaders);

      RestTemplate restTemplate = new RestTemplate();

      restTemplate.getMessageConverters()
              .add(new StringHttpMessageConverter());

      try {
          // Make the GET request to the Basic Auth protected URL
          ResponseEntity<String> response = 
                restTemplate.exchange(params[0], HttpMethod.GET, 
                       requestEntity, String.class);
          return response.getBody();
      } catch (HttpClientErrorException e) {
         // Handle 401 Unauthorized response
         Log.e("MainActivity",e.getLocalizedMessage(),e);
         return "Wrong credentials";
      }
   }

   @Override
   protected void onPostExecute(String result) {
      // executed by the UI thread		 
      resultTextView.setText(result);
   }

};
```

就像我们在前面的示例中设置`Accepted-Encoding`一样，我们在`RestTemplate`标头中设置了**HttpBasicAuthentication**标头。

让我们来看看在执行此代码期间交换的请求/响应：

| 请求 | 响应 |
| --- | --- |
| `GET/helloworld HTTP/1.1``Accept: text/plain, */*``Authorization: Basic czRhOnM0YQ==``Content-Length: 0``User-Agent: Dalvik/1.6.0 (Linux; U; Android 4.0.4; sdk Build/MR1)``Host: restfulapp.appspot.com:80``Connection: Keep-Alive``Accept-Encoding: gzip` | `HTTP/1.1 200 OK``Content-Type: text/plain; charset=iso-8859-1``Vary: Accept-Encoding``Date: Fri, 02 Nov 2012 03:33:06``Server: Google Frontend``Cache-Control: private``Transfer-Encoding: chunked``d``Hello, world``0` |

### 注意

字符串`czRhOnM0YQ==`是`s4a:s4a`的 Base64 编码。

我们将在下一节中介绍更好的身份验证和授权用户及用户应用程序的方法：*快速入门-OAuth*。

## RESTful 食谱应用程序

让我们研究 RESTful 食谱应用程序的情况：一种与 RESTful 服务交互以创建，更新，读取和删除食谱的 Android 应用程序。

食谱有点简单-它是一个 POJO，具有以下字段：

+   一个 ID（`Integer`）

+   标题（`String`）

+   食谱的描述（`String`）

+   类型：通常是主菜，主菜或甜点（`String`）

+   作者（`String`）

我们将使用作为后端服务的在线 Web 应用程序，这是一个基于 Jersey（参考 JAX-RS 实现库）的 JAX-RS（REST 服务的 JEE 规范）应用程序，并部署在 Google App Engine 上-它可以是用 Ruby，Python 或任何其他语言编写的。

### 注意

如果您已经学习了 Android 示例中的记事本应用程序的一部分，您将熟悉 RESTful 食谱应用程序：它是使用记事本应用程序代码作为基础编写的，将持久性从嵌入式 SQLite 数据库更改为 REST 在线服务。

### RecipeList 活动：列出服务器上存储的所有食谱

当用户启动应用程序时，他想要看到的第一件事是可用食谱的列表。

此活动使用`ListView`，支持`ListAdapter`以向用户显示它们。

但首先，它需要从服务器获取它们：

+   我们需要一个模型（`Recipe.java`）：

```java
public class Recipe {

   private Long id;
   private String title;
   private String description;
   private String type;
   private String author;
}
```

+   和一个嵌套在 AsyncTask 中的`RestTemplate`实例：

```java
private class GetRecipesTask extends RecipeAbstractAsyncTask <Void, Void, List<Recipe>> {

   @Override
   protected void onPreExecute() {
      showProgressDialog("Loading recipes. Please wait...");
   }

   @Override
   protected List<Recipe> doInBackground(Void... params) {
      HttpHeaders requestHeaders =  
               prepareHeadersWithMediaTypeAndBasicAuthentication();

      // Populate the headers in an HttpEntity object		
      HttpEntity<?> requestEntity = 
               new HttpEntity<Object>(requestHeaders);

      // Create a new RestTemplate instance
      RestTemplate restTemplate = new RestTemplate();
      restTemplate.getMessageConverters()
.add(new MappingJacksonHttpMessageConverter());

      try {
         // Perform the HTTP GET request
         ResponseEntity<Recipe[]> responseEntity = 
            restTemplate.exchange(
               "http://www.restfulapp.appspot.com/rest/recipes/",
               HttpMethod.GET, requestEntity,
   Recipe[].class);
         return Arrays.asList(responseEntity.getBody());
      }
      catch (RestClientException e) {
         Log.e(TAG, e.getMessage(), e);
         exception = e;
         return null;
      }
   }

   @Override
   protected void onPostExecute(List<Recipe> result) {
      dismissProgressDialog();
      if(result != null) {
         recipes = result;
      } else {
         String message = exception != null ? 
            exception.getMessage() : "unknown reason";
         Toast.makeText(RecipesList.this, 
            "A problem occurred during the reception of all recipes
            : " +message , Toast.LENGTH_LONG).show();
          recipes = new ArrayList<Recipe>();
      }
      ListAdapter adapter = new RecipeAdapter(RecipesList.this, 
         R.layout.recipeslist_item, recipes ) ;
      setListAdapter(adapter );
   }
}
```

此任务将在我们需要获取更新的食谱列表时执行：

+   当活动创建（或恢复）时：`onResume()`

+   当我们从成功的更新操作返回时：`onActivityResult()`

您可能已经注意到，此 AsyncTask 扩展了`RecipeAbstractAsyncTask`，该类定义了一个异常和一个实用方法`prepareHeadersWithMediaTypeAndBasicAuthentication()`：

```java
protected HttpHeaders prepareHeadersWithMediaTypeAndBasicAuthentication() {
      HttpHeaders requestHeaders = new HttpHeaders();
      List<MediaType> acceptableMediaTypes = new ArrayList<MediaType>();
      acceptableMediaTypes.add(MediaType.APPLICATION_JSON);
      requestHeaders.setAccept(acceptableMediaTypes);
      HttpAuthentication authHeader = new HttpBasicAuthentication("s4a", "s4a");
      requestHeaders.setAuthorization(authHeader);
      return requestHeaders;
   }
```

顾名思义，它准备请求标头以包括所需的基本身份验证，并从服务器获取所需的`MediaType`对象。

回到`GetRecipesTask`：我们准备请求标头，创建一个新的`RestTemplate`实例-配置为使用 Jackson 来（反）序列化消息，并执行 GET 请求：

```java
// Perform the HTTP GET request
               ResponseEntity<Recipe[]> responseEntity = restTemplate.exchange(getString(R.string.recipe_resource_url), HttpMethod.GET, requestEntity, Recipe[].class);
```

### 注意

`R.string.resource_url`值在`strings.xml`中定义，并且被定义为等于：[`www.restfulapp.appspot.com/rest/recipes/`](http://www.restfulapp.appspot.com/rest/recipes/)。

由于我们想要获取食谱列表，所以给`ResponseEntity`对象的类型是一个数组。

另一个重要的事情要注意的是异常处理程序：**RestClientException**，与其名字不同，它是一个包装服务器和客户端异常。如果你想告诉用户出了什么问题，最好捕获这个异常，并在执行`onPostExecute()`方法的代码时保留它，直到你回到 UI 线程。

### 注意

请注意，`RestClientException`是`RuntimeException`的子类，这就是为什么你不需要显式地捕获它。但如果你不这样做，这个异常可能会在抛出时终止你的活动。

谈到用户界面，这个任务使用它的预处理和后处理方法，通过使用`ProgresDialogs`和`Toasts`来让用户了解加载的状态。

### 注意

手动调试 REST 资源可能很有用，从应用程序外部；**cURL**应用程序（可在[`curl.haxx.se/download.html`](http://curl.haxx.se/download.html)下载）可能是最流行的工具。

在本节中引发的每个 HTTP 请求，我现在都会包括它的匹配 cURL 命令行。

```java
curl --user s4a:s4a  -H "Accept: application/json" -i http://www.restfulapp.appspot.com/rest/recipes 
[{"id":0,"title":"Montreal's Poutine","description":"French fries are covered with fresh cheese curds, and topped with brown gravy.","type":"MAIN_DISH","author":"Anthony"},{"id":1,"title":"a title","description":"a description","type":"a type","author":"an author"}]
```

![RecipeList 活动：列出服务器上存储的所有食谱](img/1905_03_03.jpg)

RecipesList 活动概述

### RecipeEditor 活动

这个活动负责获取、更新、创建和删除食谱。

![RecipeEditor 活动](img/1905_03_04.jpg)

更新模式下的 RecipeEditor 活动

+   获取食谱

当用户点击 RecipeList 活动的创建图标，或者点击其中一个食谱标题时，他/她将被引导到 RecipeEditor 活动。

如果用户想要创建一个新的食谱，他/她将看到空的字段，但如果他/她想要查看一个食谱，他们将首先从服务器获取它，然后再填写字段；这要归功于内部的`GetRecipeTask`：

```java
// Perform the HTTP GET request
Log.i(TAG,"Getting the recipe with id : "+params[0] + " : " +url + params[0]);
ResponseEntity<Recipe> responseEntity = restTemplate.exchange(url + params[0], HttpMethod.GET, requestEntity,	Recipe.class);
return responseEntity.getBody();
```

这里没有什么新鲜的，我们想要获取一个单独的食谱，通过它的 ID 来识别（`params[0]`）。

```java
curl --user s4a:s4a  -H "Accept: application/json" -i http://www.restfulapp.appspot.com/rest/recipes/0
{"id":0,"title":"Montreal's Poutine","description":"French fries are covered with fresh cheese curds, and topped with brown gravy.","type":"MAIN_DISH","author":"Anthony"}
```

+   放置一个食谱

要更新一个食谱，我们必须使用 PUT HTTP 动词请求食谱资源。

为此，我们再次有一个专用的 AsyncTask：`UpdateRecipeTask`，它将实例化一个`RestTemplate`实例，携带更新后的食谱（一旦用户按下`保存`按钮，我们调用`updateNote()`方法，将视图的内容与当前食谱的模型同步）。

```java
// Create a new RestTemplate instance
RestTemplate restTemplate = new RestTemplate();
restTemplate.getMessageConverters()
   .add(new MappingJacksonHttpMessageConverter());

// Populate the headers in an HttpEntity object HttpEntity<Recipe> requestEntity = new HttpEntity<Recipe>(recipe,requestHeaders);
try {
   // Perform the HTTP PUT request
   Log.i(TAG,"Updating the recipe with id : "
      +recipe.getId() + " : " + url);
   ResponseEntity<Void> responseEntity = 
      restTemplate.exchange(url, HttpMethod.PUT, requestEntity,
      Void.class);
   if(responseEntity.getStatusCode() != HttpStatus.OK) {
      throw new HttpServerErrorException(
         responseEntity.getStatusCode());
   }
}
catch (RestClientException e) {
   Log.d(TAG, e.getMessage(), e);
   exception = e;
}
```

你可以看到这里真正重要的部分是`RequestEntity`，它包装了我们的食谱：`RestTemplate`将把它传递给 Jackson，后者将把它转换成 JSON 字符串。

如果服务器没有以 HTTP 状态 200（OK）响应，我们可以认为出了问题，并将异常保留给用户界面。

```java
curl --user s4a:s4a  -H "Accept: application/json" -H "Content-type: application/json"  -X PUT -d '{"id":"0", "title":"a title","description":"a description","type":"a type","author":"an author"}'  -i http://www.restfulapp.appspot.com/rest/recipes/
```

+   发布一个食谱

要创建一个食谱，我们必须使用 POST HTTP 动词请求食谱资源。

在 REST 文献中，你也会读到使用 PUT HTTP 动词来创建实体；但有一个条件——你必须提供 ID（这样连续的 PUT 不会改变资源状态。PUT 是幂等的，就像 GET 和 DELETE 一样，也像 DELETE 和 POST 一样不安全）。

### 注意

在我们的情况下，我们创建一个新资源，所以 POST 很好地反映了我们的意图。

让我们来看一下`CreateRecipeTask`，它与`UpdateRecipeTask`非常相似：

```java
// Populate the headers in an HttpEntity object 
HttpEntity<Recipe> requestEntity = 
   new HttpEntity<Recipe>(recipe,requestHeaders);
try {
   // Perform the HTTP POST request
   Log.i(TAG,"Posting the recipe with id : "
      +recipe.getId() + " : to " +url);
   ResponseEntity<Void> responseEntity = 
      restTemplate.exchange(url, HttpMethod.POST, requestEntity,
      Void.class);
   if(responseEntity.getStatusCode() != HttpStatus.CREATED) {
      throw new HttpServerErrorException(
         responseEntity.getStatusCode());
   }
}
catch (RestClientException e) {
   Log.d(TAG, e.getMessage(), e);
   exception = e;
}
return null;
```

一些有趣的事情需要注意：食谱没有设置任何 ID，因为用户是从无到有创建食谱的。Web 服务将为这个食谱分配一个 ID（并应该返回创建资源的 URI）；期望的状态不是 200，而是 201（`HttpStatus.CREATED`）。

```java
curl --user s4a:s4a  -H "Accept: application/json" -H "Content-type: application/json"  -X POST -d '{"title":"a title","description":"a description","type":"a type","author":"an author"}'  -i http://www.restfulapp.appspot.com/rest/recipes/
```

+   删除一个食谱

要删除一个食谱，我们必须使用 DELETE HTTP 动词请求食谱资源。

名为`DeleteRecipeTask`的 AsyncTask 与`GetRecipeTask`类似，因为它只需要将食谱的 ID 附加到资源 URI（无需像 PUT 和 POST 那样携带整个食谱）。

```java
try {
   // Perform the HTTP DELETE request
   Log.i(TAG,"Deleting the recipe with id : "
      +recipe.getId() + " : from " +url +recipe.getId() );
   ResponseEntity<Void> responseEntity = 
      restTemplate.exchange(url+recipe.getId(), 
      HttpMethod.DELETE, requestEntity,	Void.class);
   if(responseEntity.getStatusCode() != HttpStatus.NO_CONTENT) {
      throw new HttpServerErrorException(
      responseEntity.getStatusCode());
   }
}
catch (RestClientException e) {
   Log.d(TAG, e.getMessage(), e);
   exception = e;
}
```

这个请求期望得到一个带有 204 HTTP 代码的响应，意味着请求成功，但没有返回任何内容（`HttpStatus.NO_CONTENT`）。

### 注意

HTTP 代码 200 将作为成功的 DELETE 请求的响应。

查看您的网络服务文档（或更好的是，例如使用 cURL 进行实验），以便对网络服务应该返回什么做出最佳的假设。

```java
curl --user s4a:s4a  -H "Accept: application/json" -X DELETE   -i http://www.restfulapp.appspot.com/rest/recipes/1
```

## Android 注释

您可能已经听说过这个项目，位于[`github.com/excilys/androidannotations/`](https://github.com/excilys/androidannotations/)，旨在减少在 Android 应用程序中注入视图、活动、提供程序、服务等所需的代码量。

项目负责人决定使用 Spring for Android 的`RestTemplate`作为支持 REST 注释的 REST 客户端库。

### 注意

Android 注释基于**注释处理器工具**（**apt**），现在是 Java 6 编译器的一部分；每个带注释的类都会触发一个子类的生成（文件名相同，只是在末尾加上`_`，例如`OriginalActivity_.java`），其中包含了所有注释为您节省的样板代码！

因此，除了`@Eactivity`（允许活动使用 Android 注释）、`@ViewById`（注入视图），还有`@Get`、`@Post`、`@Put`等。

让我们看看如何从 Android 注释中受益，以 JSON 示例为例（从`Ifconfig.me`获取 IP 地址）。

```java
@EActivity(R.layout.activity_main)
public class MainActivity extends Activity {

   private IfConfigMeJson all;

   //inject the view to the activity layout
    @ViewById(R.id.result_text)
    TextView resultTextView;

    //inject the Rest service that wraps RestTemplate
    @RestService
   IfConfigMeRestClient restClient;
         //Use the Rest Service in a background thread
    @Background
    @AfterInject
    void getAllInfo() {
       all = restClient.getAll();
    }

    //wait a few seconds for the service to finish
   @UiThread(delay = 5000)
   @AfterViews
   void afterViews() {
      resultTextView.setText("Your IP is : "+all.getIpAddr());
   }

}
```

肯定有更少的代码，对吧？

您可能已经注意到，我们正在使用一个名为`IfConfigMeRestClient`的新类：

```java
@Rest(rootUrl = "http://ifconfig.me", converters = 
      { MappingJacksonHttpMessageConverter.class })
public interface IfConfigMeRestClient {

   @Get("/all.json")
   @Accept(MediaType.APPLICATION_JSON)
   IfConfigMeJson getAll();

}
```

这个类配置了`RestTemplate`，提供了转换器类、使用的 HTTP 动词、映射类（`IfConfigMeJson`）和接受的媒体类型。

Android 注释为 Spring for Android 的`RestTemplate`提供了良好的支持（请参阅[`github.com/excilys/androidannotations/wiki/Rest-API`](https://github.com/excilys/androidannotations/wiki/Rest-API)了解基于`RestTemplate`的注释列表），我建议您尝试一下，看看您是否更喜欢在基于 Spring for Android 的项目中使用它。

### 注意

如果您从头开始使用 Android 注释启动项目，我建议您仔细遵循这些环境说明：[`github.com/excilys/androidannotations/wiki/Building-Project-Maven-Eclipse`](https://github.com/excilys/androidannotations/wiki/Building-Project-Maven-Eclipse)。

# 快速开始 - OAuth

**OAuth**是一种授权的开放标准，正如维基百科所说。

OAuth 允许用户与网站或桌面或移动应用程序共享托管在服务提供商上的数据或资源。

当然，如果您向这些网站或应用程序提供了您的凭据（用户名和密码），它们可以代表您访问您的数据；但是，您会相信第三方应用程序或网站来保护您的凭据（您数字生活的钥匙）吗？如果这个应用程序或网站是恶意的呢？或者只是不安全的？您有多少次听说过网站被盗取用户凭据，或者在他们的数据库中以明文形式保存密码？

OAuth 就是这样的；让第三方应用程序或网站有限访问（通过一系列授权或范围：`访问用户电子邮件`、`访问用户个人资料`、`可以代表用户发布消息`等）您在服务提供商（著名的 OAuth 服务提供商包括 Google、Facebook、Twitter、Yahoo!、GitHub、LinkedIn 等）上的数据，而不必向它们提供您的凭据。

## 开发依赖于服务提供商的网站或应用程序

您是否曾经注意到一些网站（例如[Deezer.com](http://Deezer.com)、[StackOverFlow.com](http://StackOverFlow.com)等）上的**使用 Facebook 登录**或**使用 Google 登录**按钮，或者在 Android 设备上使用第三方 Twitter 应用程序（如 Twicca）请求您的 Twitter 凭据？

所有这些网站和应用程序都使用 OAuth 来访问您的数据：

+   首先，它们为您节省了创建和记住其他凭据的麻烦。它们甚至通过访问您的数据（例如您的个人资料图片、活动信息等）来丰富您的用户体验。

+   此外，它们吸引了更多的受众（因为用户不必经历繁琐的帐户创建过程），他们也不必管理认证凭据（带来的风险）或授权。

通常，如果网站或应用程序创建者想要从提供 OAuth 授权的服务中受益，他/她必须注册该服务以针对 OAuth 服务提供商（如 Google、Facebook 等）进行注册，后者将提供给他/她一个**客户端 ID**和**客户端密钥**值：

![开发依赖于服务提供商的网站或应用程序](img/1905_04_01.jpg)

使用 Google API 控制台注册新应用程序并将其与客户端 ID 和客户端密钥关联起来

让我们发现这个客户端 ID 和客户端密钥是如何使应用程序能够访问用户数据的。

## OAuth 之舞

Android 应用程序（或任何其他已安装的应用程序）依赖于诸如 Google 之类的 OAuth 2.0 服务提供商。让我们来看看以下五个简单步骤中的 OAuth 授权流程的工作流程：

1.  用户首次启动应用程序时，它将生成令牌请求 - 包括应用程序客户端 ID 和客户端密钥，以及应用程序所需的几个授权（用户信息、用户电子邮件等）的服务提供商的 URL。

以 Google OAuth 2.0 服务提供商为例：

```java
https://accounts.google.com/o/oauth2/auth?
client_id=508046100884-o6jgcn8e7c1g5gklhc8gibr80ouio8df.apps.googleusercontent.com&
response_type=code&
redirect_uri=http://localhost&
scope=https://www.googleapis.com/auth/userinfo.profile https://www.googleapis.com/auth/userinfo.email https://www.googleapis.com/auth/plus.me
```

1.  这个 URL 由 Android Chrome 浏览器加载，很可能是通过 WebView 嵌入到应用程序中的。用户被要求接受或拒绝应用程序请求的授权。

1.  作为回报，如果用户授予应用程序权限，应用程序将拦截来自嵌入式浏览器的响应，其中包含授权码。

使用 Google OAuth 2.0 服务提供商的响应示例：

```java
http://localhost/?code=4/urIB_wqrOqGpX-2w1UPXD8dHQAYO.ArEX_6EbNP0ZuJJVnL49Cc98lfsNdgI
```

1.  应用程序最终将使用此授权码交换令牌（如果令牌的生命周期有限，应用程序还将接收到一个刷新令牌和过期时间）。此访问令牌将由应用程序安全保存。

使用 Google OAuth 2.0 服务提供商的令牌请求/响应示例。

**请求**：

```java
POST /o/oauth2/token HTTP/1.1
Host: accounts.google.com
Content-Type: application/x-www-form-urlencoded

code=4/v6xr77ewYqhvHSyW6UJ1w7jKwAzu&
client_id=8819981768.apps.googleusercontent.com&
client_secret={client_secret}&
redirect_uri=https://oauth2-login-demo.appspot.com/code&
grant_type=authorization_code
```

**响应**：

```java
{
  "access_token":"1/fFAGRNJru1FTz70BzhT3Zg",
  "expires_in":3920,
  "token_type":"Bearer",
  "refresh_token":"1/xEoDL4iW3cxlI7yDbSRFYNG01kVKM2C-259HOF2aQbI"
}
```

1.  应用程序最终将能够与资源进行交互（例如 REST 服务），前提是它在每个请求中发送一个有效的令牌。

使用 Google OAuth 2.0 服务提供商的令牌请求示例：

```java
GET /oauth2/v1/userinfo HTTP/1.1
Authorization: Bearer 1/fFBGRNJru1FQd44AzqT3Zg
Host: googleapis.com
```

### 注意

大多数 OAuth 2.0 服务提供商只接受 HTTPS 上的授权服务调用，以确保令牌不会被拦截。

![OAuth 之舞](img/1905_04_02.jpg)

从[`developers.google.com/accounts/docs/OAuth2`](https://developers.google.com/accounts/docs/OAuth2)访问 Google UserInfo 服务的 OAuth 工作流程

### 注意

您可以阅读[`developers.google.com/accounts/docs/OAuth2InstalledApp`](https://developers.google.com/accounts/docs/OAuth2InstalledApp)了解 Google OAuth 2.0 服务提供商的具体信息，或者[`developers.facebook.com/docs/concepts/login/login-architecture/`](http://developers.facebook.com/docs/concepts/login/login-architecture/)了解 Facebook OAuth 2.0 服务提供商的具体信息。

强烈建议在开始开发客户端应用程序之前阅读 OAuth 服务提供商的文档（它们之间有一些微妙的差异，如请求令牌的参数、回调 URL 等）。

## Spring for Android Auth 模块

Spring for Android 的`Auth`模块支持 OAuth 1.0a 和 OAuth 2.0。

简而言之：

+   OAuth 2.0 允许更好地支持非 Web 应用程序（例如 Android 应用程序）

+   OAuth 2.0 不再要求开发人员使用密码学。

+   OAuth 2.0 访问令牌是*短暂的*（多亏了刷新令牌）

### 注意

要了解有关 OAuth 2.0 的更多信息，您可以阅读*Eran Hammer*在他的博客上的几篇文章：[`hueniverse.com/2010/05/introducing-oauth-2-0/`](http://hueniverse.com/2010/05/introducing-oauth-2-0/)

Spring for Android 的`Auth`模块依赖于以下内容：

+   Spring for Android 核心（通用类：`Base64`，`StringUtils`，资源抽象等）

+   Spring for Android 的`RestTemplate`（REST API 抽象底层的 HttpClient）

+   Spring Social Core（OAuth 1.0 和 2.0 API 包装了 OAuth 流程：请求授权码，请求令牌等）

+   Spring Security Crypto（提供对对称加密、密钥生成和密码编码的支持）

`Auth`模块本身为 Android 定义了密码学和 SQL 类（以便您可以将授权令牌安全地持久保存到 SQLite Android 数据库中）。

### 注意

截至目前，Spring for Android 的`Auth`只包含了七个类，这些类都是关于在 Android 设备上支持持久化 OAuth 令牌；所有的 OAuth 代码都在 Spring Social Core 中。

为了使 Twitter、Facebook 和 Google 应用程序开发人员的生活更加轻松，Spring Social 分别有扩展，分别命名为 Spring Social Twitter、Spring Social Facebook 和 Spring Social Google。这些定义了访问用户信息、帖子、联系人、每个社交网络中的日程安排的 API。

## 使用 Google 的 OAuth 示例

我们将构建一个 Android 应用程序，显示有关 Google 用户的信息：当然我们将使用 Spring for Android 的`Auth`，但更重要的是 Spring Social 和 Spring Social Google。

### 注意

官方文档示例是关于 Facebook 和 Twitter 的。如果您想创建一个与这些社交网络进行交互的应用程序，请查看[`github.com/SpringSource/spring-android-samples`](https://github.com/SpringSource/spring-android-samples)中的 Spring for Android 示例。

让我们来看一下这个项目的`pom.xml`文件，特别是包含 Spring Social Google：

```java
<dependency>
  <groupId>org.springframework.social</groupId>
  <artifactId>spring-social-google</artifactId>
  <version>1.0.0.M1</version>
  <exclusions>
    <!-- Exclude in favor of Spring Android RestTemplate -->
    <exclusion>
      <artifactId>spring-web</artifactId>
      <groupId>org.springframework</groupId>
      </exclusion>
      <exclusion>
      <artifactId>spring-core</artifactId>
      <groupId>org.springframework</groupId>
    </exclusion>
  </exclusions>
</dependency>
[...]
<repository>
  <id>spring.social.google</id>
  <name>Spring Social Google</name>
  <url>https://github.com/GabiAxel/maven/raw/master/</url>
</repository>
```

您会注意到（在“存储库”部分），这个模块并不是由 Spring Source 托管的，因为它实际上是一个社区项目，不受 Spring Source 的认可。

### 注意

这个`pom.xml`文件包含了许多排除项；这是因为大多数使用的库都是针对 Java SE 开发的，这就是为什么它们依赖于 Spring Core、Spring MVC 等。Spring for Android Core 和`RestTemplate`为这些模块提供了必要的依赖项。

现在让我们来看一下`AndroidManifest.xml`文件：

```java
[...]
<application
    android:name=".MainApplication"
    android:icon="@drawable/app_notes"
    android:label="@string/app_name" >
    <activity
        android:name=".GoogleActivity"
        android:label="@string/title_main" >
        [...]
    </activity>
    <activity
        android:name=".GoogleWebOAuthActivity"
        android:excludeFromRecents="true"
        android:noHistory="true" />
    <activity android:name=".GoogleProfileActivity" />
</application>
```

在我们的示例中，我们将首次使用一个名为`MainApplication`的`Application`类。

### 注意

`GoogleWebOAuthActivity`将嵌入一个浏览器，仅用于身份验证。我们不希望此活动成为应用程序历史的一部分，也不希望用户能够返回到它；这就是为什么我们添加了`android:noHistory="true"`和`android:excludeFromRecents="true"`。有关此信息的更多信息，请访问[`developer.android.com/guide/topics/manifest/activity-element.html`](http://developer.android.com/guide/topics/manifest/activity-element.html)。

这个类将用于准备我们应用程序的两个最重要的工厂（它们将在所有活动中被访问）：`ConnectionFactoryRegistry`和`ConnectionRepository`：

```java
public class MainApplication extends Application {
  private ConnectionFactoryRegistry connectionFactoryRegistry;
  private SQLiteOpenHelper repositoryHelper;
  private ConnectionRepository connectionRepository;

  // ***************************************
  // Application Methods
  // ***************************************
  @Override
  public void onCreate() {
    // create a new ConnectionFactoryLocator and populate it with Google ConnectionFactory
    this.connectionFactoryRegistry = new ConnectionFactoryRegistry();
    this.connectionFactoryRegistry.addConnectionFactory(new GoogleConnectionFactory(getClientId(),
        getClientSecret()));

    // set up the database and encryption
    this.repositoryHelper = new SQLiteConnectionRepositoryHelper(this);
    this.connectionRepository = new SQLiteConnectionRepository(this.repositoryHelper,
        this.connectionFactoryRegistry, AndroidEncryptors.text("password", "5c0744940b5c369b"));
  }
```

如您所见，在“onCreate（）”方法中我们初始化：

+   `ConnectionFactoryRegistry`：使用应用程序的客户端 ID 和客户端密钥从`ConnectionFactoryRegistry`中，我们将可以访问`GoogleConnectionFactory`，这是`OAuth2ConnectionFactory`的 Google 服务扩展，可以访问所有 OAuth 操作

+   `ConnectionRepository`：这将负责持久化`ConnectionFactoryRegistry`，以便可以在不需要每次都执行整个 OAuth 工作流的情况下检索 OAuth 令牌

### 注意

您可能已经注意到在初始化数据库时使用了盐和密码（加密）。

这将防止恶意应用程序能够访问设备数据库以检索用户的 OAuth 令牌。简要提醒：应用程序永远不会访问用户的 Google 密码。在这个例子中，对服务提供商 Google 的身份验证总是从设备浏览器中进行的。

让我们来看看该项目的主要活动，即在启动时启动的 `GoogleActivity`：

```java
@Override
public void onStart() {
  super.onStart();
  if (isConnected()) {
    showGoogleOptions();
  } else {
    showConnectOption();
  }
}

private boolean isConnected() {
  return connectionRepository.findPrimaryConnection(Google.class) != null;
}
```

如果用户已连接，此活动将显示与用户配置文件相关的条目列表，或者如果用户尚未连接，则显示**连接**按钮（由于 `GoogleConnectionFactoryRegistry` 已持久化在数据库中，因此只需在 `ConnectionRepository` 中查找类型为 `Google` 的连接即可知道访问令牌是否已经获取）。

![使用 Google 的 OAuth 示例](img/1905_04_03.jpg)

如果用户尚未登录，则会出现 GoogleActivity

因此，在我们尚未连接的情况下，点击**连接**将调用 `displayGoogleAuthorization()`，这将启动 `GoogleWebOAuthActivity`。

`GoogleWebOAuthActivity` 是这个应用程序中最重要的活动。它负责 OAuth 2.0 的身份验证和授权。

```java
@Override
public void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);

  //javascript is mandatory
  getWebView().getSettings().setJavaScriptEnabled(true);

  // Using a custom web view client to capture the access token
  getWebView().setWebViewClient(new GoogleOAuthWebViewClient());

  this.connectionRepository = getApplicationContext().getConnectionRepository();
  this.connectionFactory = getApplicationContext().getGoogleConnectionFactory();
}

@Override
public void onStart() {
  super.onStart();

  // display the Google authorization page
  getWebView().loadUrl(getAuthorizeUrl());
}

// ***************************************
// Private methods
// ***************************************
private String getAuthorizeUrl() {
  String redirectUri = getString(R.string.google_oauth_callback_url);
  String scope = getString(R.string.google_scope);

  // Generate the Google authorization url to be used in the browser 
  OAuth2Parameters params = new OAuth2Parameters();
  params.setRedirectUri(redirectUri);
  params.setScope(scope);
  return this.connectionFactory.getOAuthOperations().buildAuthorizeUrl(GrantType.AUTHORIZATION_CODE, params);
}
```

当创建此活动时，它会配置关联的 WebView（您会注意到此活动扩展了 `AbstractWebViewActivity`，它将 Chrome 浏览器实例注入到 WebView 中）以接受 JavaScript（服务提供商和 Google OAuth 2.0 需要 JavaScript 来验证用户），并注入一个自定义的 `WebViewClient` 对象，我们将用它来拦截 OAuth 流程（稍后会详细介绍）。

然后，当活动启动时，我们要求 WebView（嵌入的 Chrome 浏览器）为此应用程序请求授权码（请参阅*OAuth 之舞*部分的步骤 1）。

此请求使用回调 URL、应用程序所需的授权范围以及客户端 ID 和密钥构建（这两个在我们创建 `ConnectionFactoryRegistry` 时已经提供给了 Spring OAuth）。

```java
<resources>
    <string name="google_app_id">508046100884-o6jgcn8e7c1g5gklhc8gibr80ouio8df.apps.googleusercontent.com</string>
    <string name="google_app_secret">RuUyrF5qoGYWTFm1r_o8Gs4F</string>
    <string name="google_oauth_callback_url">http://localhost</string>
    <string name="google_scope">https://www.googleapis.com/auth/userinfo.profile https://www.googleapis.com/auth/userinfo.email https://www.googleapis.com/auth/plus.me</string>
</resources>
```

### 注意

这里的回调 URL 是[`localhost`](http://localhost)，因为 Google OAuth 2.0 服务提供商可以选择在[`localhost`](http://localhost)和`urn:ietf:wg:oauth:2.0:oob`之间进行选择。

这将影响包含授权码的响应；无论它是作为查询字符串参数还是在浏览器的标题栏中。您可以阅读[`developers.google.com/accounts/docs/OAuth2InstalledApp#choosingredirecturi`](https://developers.google.com/accounts/docs/OAuth2InstalledApp#choosingredirecturi)了解更多详情。

如果用户尚未在设备上对任何 Google 网络服务进行身份验证，他/她应该会看到一个对话框邀请他/她进行身份验证：

![使用 Google 的 OAuth 示例](img/1905_04_04.jpg)

只有在用户尚未在设备上对任何 Google 服务进行身份验证时，才会出现此对话框。

无论如何，用户都将看到此授权对话框，其中列出了应用程序请求的所有*范围*：

![使用 Google 的 OAuth 示例](img/1905_04_05.jpg)

Google OAuth 2.0 服务提供商授权对话框

### 注意

如果用户拒绝授权，那么预期的是授权过程将被终止。

根据服务提供商，此对话框可能会有所不同。

一旦用户接受了请求的授权，`GoogleWebOAuthActivity` 将检测到 Web 客户端正在被重定向到本地主机（**回调 URI**），并附带授权码：

```java
private class GoogleOAuthWebViewClient extends WebViewClient {

  private static final String LOCALHOST = "localhost";
  private static final String CODE = "code";

  /*
   * The WebViewClient has another method called shouldOverridUrlLoading which does not capture the javascript 
   * redirect to the success page. So we're using onPageStarted to capture the url.
   */
  @Override
  public void onPageStarted(WebView view, String url, Bitmap favicon) {
    // parse the captured url
    Uri uri = Uri.parse(url);
     // log the url : very interesting for debugging the OAuth workflow
    Log.d(TAG, url);

    /*
     * The WebViewClient is launched to load an URL from the provider that will ask the user whether or not he accepts our app to access his data.
     * Once the provider successfully gets the approval from the user, it will redirect this WebViewClient to the callback_uri, with a query parameter named "code" : this is the authorization code
     */
    String host = uri.getHost();
    String code = uri.getQueryParameter(CODE);

    // The WebViewClient is redirected to the callback_uri, let's trade the authorization code for the access token
    if (LOCALHOST.equals(host)) {
      if(!exchangeAuthorizationCodeForAccessToken.getStatus().equals(AsyncTask.Status.RUNNING)) {
        exchangeAuthorizationCodeForAccessToken.execute(code);
        Toast.makeText(getApplicationContext(), "Redirecting you to the app main activity", Toast.LENGTH_LONG).show();
        //preparing to quit this activity for the main activity
        getWebView().setVisibility(View.INVISIBLE);
      }
    }
  }
}
```

`exchangeAuthorizationCodeForAccessToken.execute(code)` 将执行以下的 AsyncTask（我们将从我们的应用程序中使用 `RestTemplate` 发送授权码，依赖于 Java `UrlConnection`，因此我们需要从后台线程中编写此调用）：

```java
private AsyncTask<String, Void, Void> exchangeAuthorizationCodeForAccessToken =  new AsyncTask<String, Void, Void>() {

  private Exception exception;

  @Override
  protected Void doInBackground(String... params) {
    // executed by a background thread
    //params[0] should contain the authorization code
    try {
      AccessGrant exchangeForAccess = connectionFactory.getOAuthOperations().exchangeForAccess(params[0], getString(R.string.google_oauth_callback_url), null);
      Connection<Google> connection = connectionFactory.createConnection(exchangeForAccess);
      connectionRepository.addConnection(connection);
    } catch (DuplicateConnectionException e) {
      Log.e(TAG,"something went wrong when adding the accessToken to the connection repository",e);
      exception = e;
    } catch (Exception e) {
      Log.e(TAG,"something went wrong when adding the accessToken to the connection repository",e);
      exception = e;
    }
    return null;
  }

  @Override
  protected void onPostExecute(Void result) {
    // executed by the UI thread once the background thread is done getting the result
    if(exception != null) {
      Toast.makeText(getApplicationContext(), exception.getMessage(), Toast.LENGTH_LONG).show();
    }
      // we go back to the main activity to display the options
      displayGoogleOptions();
  }

};
```

一旦调用了 `exchangeForAccess` 方法，我们就会检索用户令牌并将其持久化在 `ConnectionRepository` 类中。

我们的应用程序最终被授权访问用户的 Google 配置文件！

![使用 Google 的 OAuth 示例](img/1905_04_06.jpg)

如果用户尚未登录，则显示 GoogleActivity

如果用户点击**个人资料**，他将启动`GoogleProfileActivity`，从中我们可以获取用户个人资料。

为此，我们使用一个名为`FetchProfileTask`的 AsyncTask，它将访问两个 Google Web 服务：`UserOperations`（用于读取用户的主要个人资料和个人资料图片）和`PersonOperations`（用于读取他/她的 Google+个人资料，这里我们只访问**关于我**描述）：

```java
@Override
protected LegacyGoogleProfile doInBackground(Void... params) {

    LegacyGoogleProfile userProfile = google.userOperations().getUserProfile();
    aboutMe = google.personOperations().getGoogleProfile().getAboutMe();
    profileBitmap = BitmapFactory.decodeStream(new URL(userProfile.getProfilePictureUrl()).openConnection().getInputStream());
    return userProfile;

}
```

然后将此信息注入视图：

![使用 Google 的 OAuth 示例](img/1905_04_07.jpg)

GoogleProfileActivity 显示用户个人资料图片，“关于我”描述和一些个人资料信息

# 您应该了解的人和地方

如果您需要 Spring for Android（或 REST 或 OAuth 一般）的帮助，以下是一些将非常有价值的人和地方。

## 官方网站

+   主页：[`www.springsource.org/spring-android`](http://www.springsource.org/spring-android)

+   手册和文档：[`static.springsource.org/spring-android/docs/1.0.x/reference/htmlsingle/`](http://static.springsource.org/spring-android/docs/1.0.x/reference/htmlsingle/)

+   博客：[`blog.springsource.org/category/android/`](http://blog.springsource.org/category/android/)

+   源代码：[`github.com/SpringSource/spring-android`](https://github.com/SpringSource/spring-android)

+   官方示例源代码：[`github.com/SpringSource/spring-android-samples`](https://github.com/SpringSource/spring-android-samples)

+   Android Maven 插件：[`code.google.com/p/maven-android-plugin/`](http://code.google.com/p/maven-android-plugin/)

## 文章和教程

+   OAuth 解释：[`hueniverse.com/oauth/guide/`](http://hueniverse.com/oauth/guide/)

+   Google 服务的 OAuth：[`support.google.com/a/bin/answer.py?hl=en&answer=61017`](http://support.google.com/a/bin/answer.py?hl=en&answer=61017)

+   关于远程连接的官方 Android 文档：[`developer.android.com/training/basics/network-ops/connecting.html`](http://developer.android.com/training/basics/network-ops/connecting.html)

## 社区

+   官方论坛：[`forum.springsource.org/forumdisplay.php?88-Android`](http://forum.springsource.org/forumdisplay.php?88-Android)

+   官方错误跟踪器：[`jira.springsource.org/browse/ANDROID`](https://jira.springsource.org/browse/ANDROID)

## 博客

+   Android 团队博客：[`android-developers.blogspot.ca/`](http://android-developers.blogspot.ca/)

## Twitter

+   在 Twitter 上关注 Spring for Android：[`twitter.com/springandroid`](https://twitter.com/springandroid)

+   在 Twitter 上关注 Roy Clarkson（Spring for Android 首席开发人员）：[`twitter.com/royclarkson`](https://twitter.com/royclarkson)

+   获取更多开源信息，请关注 Packt：[`twitter.com/#!/packtopensource`](http://twitter.com/#!/packtopensource)
