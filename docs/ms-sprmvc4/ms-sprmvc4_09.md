# 第九章。将您的 Web 应用程序部署到云端

在本章中，我们将游览不同的云服务提供商，了解分布式架构的挑战和优势，并了解如何将您的 Web 应用程序部署到 Pivotal Web 服务和 Heroku。

# 选择您的托管服务

云托管有多种形式。对于开发者来说，选择将主要在平台即服务（PaaS）和基础设施即服务（IaaS）之间。

使用最新版本，您通常会有一个裸机，您可以对其进行管理，并在其上安装应用程序所需的所有服务。

如果我们排除 Docker 等技术（它绝对令人惊叹，您绝对应该尝试一下），这实际上与传统托管非常相似，您的运维团队将不得不设置和维护一个应用程序可以运行的环境。

另一方面，PaaS 使得在开发过程中通过简单的推送部署工作流程轻松部署您的应用程序。

最知名的服务提供商是：

+   由 Pivotal 支持的 Cloud Foundry

+   Red Hat 的 OpenShift

+   Salesforce 在 2010 年收购了 Heroku

这三个提供商中的每一个都有不同的优缺点。我将尝试为您概述这些。

## Cloud Foundry

由 Spring 背后的 Pivotal 支持的 Pivotal Web 服务在 Cloud Foundry 上运行，这是一个由基金会维护的开源 PaaS，并附带一个有趣的套餐。

他们提供 60 天的免费试用，并且他们的定价是您实例分配的内存和您拥有的实例数量的函数。

它们的价格从每月 2.70 美元的最小（128 Mb）实例到每月 43.20 美元的 2 GB 实例不等。

如果您想尝试一下，免费试用不需要信用卡。他们有一个市场，可以轻松安装服务，如 Redis 或 PostgreSQL，但免费选项相当有限。他们有一个很好的命令行实用程序，可以从您的控制台管理应用程序。您可以使用构建包或直接推送 JAR 进行部署。

### 小贴士

构建包将尝试猜测您正在使用的堆栈，并以最标准的方式构建您的应用程序（例如，对于 Maven 使用 `mvn package`，对于 Gradle 使用 `./gradlew stage` 等）。

### 注意

请参阅以下 URL 上的教程，以将您的应用程序部署到 Cloud Foundry：

[`docs.cloudfoundry.org/buildpacks/java/gsg-spring.html`](http://docs.cloudfoundry.org/buildpacks/java/gsg-spring.html)

## OpenShift

**OpenShift** 由 Red Hat 维护，并由 OpenShift Origin 驱动，这是一个开源设施，在 Google 的 Kubernetes 上运行 Docker 容器。

它的价格合理，提供了很多自由度，因为它既是 PaaS 也是 IaaS。其定价基于齿轮，即运行应用程序的容器，或像 Jenkins 这样的服务，或数据库。

OpenShift 提供一个免费计划，提供三个小型齿轮。除非您输入您的账单信息，否则您的应用程序每月必须空闲 24 小时。

额外或更大的齿轮按每月约 $15（最小）和 $72（最大）计费。

要在 OpenShift 上部署 Spring Boot 应用程序，你必须使用 DIY 卡带。它比其他基于构建包的 PaaS 工作量稍大，但配置起来也更简单。

请查看有关 OpenShift 上 Spring Boot 教程的博客文章，该文章可在 [`blog.codeleak.pl/2015/02/openshift-diy-build-spring-boot.html`](http://blog.codeleak.pl/2015/02/openshift-diy-build-spring-boot.html) 找到。

## Heroku

Heroku 是一个广为人知的 PaaS，拥有丰富的文档和基于构建包的以代码为中心的方法。它可以连接到许多称为附加组件的服务，但使用它们需要你的账单信息。

对于免费项目来说，这真的很吸引人，并且启动非常快。缺点是，如果你想要扩展，直接成本将超过每月 $25。免费实例在 30 分钟无活动后将进入睡眠模式，这意味着免费的 Heroku 应用程序始终需要 30 秒来加载。

Heroku 拥有出色的管理仪表板和命令行工具。对于本章，我选择了 Heroku，因为它非常直观。在这里你将掌握的概念适用于大多数 PaaS。

只要你不使用 Redis 附加组件，你就可以遵循本章的大部分内容并部署你的应用程序，而无需提供信用卡信息。如果你选择免费计划，你将不会收费。

# 将你的 Web 应用程序部署到 Pivotal Web Services

如果你想要将你的应用程序部署到 Pivotal Web Services（PWS），请遵循本节。

## 安装 Cloud Foundry CLI 工具

我们创建 Cloud Foundry 应用程序的第一件事是在 PWS 上设置一个账户。这已在 [`docs.run.pivotal.io/starting/`](http://docs.run.pivotal.io/starting/) 中记录。

你将被要求创建一个组织，并且每个新的组织将在组织内创建一个默认空间（开发）。如下截图所示：

![安装 Cloud Foundry CLI 工具](img/2117_09_04.jpg)

在左侧导航栏中，你会看到一个链接到 **工具**，你可以从这里下载 CLI。它也可以从开发者控制台获取。选择适合你操作系统的适当包：

![安装 Cloud Foundry CLI 工具](img/2117_09_05.jpg)

## 组装应用程序

我们的应用程序只需组装即可部署。

PWS 的好处是，你不需要推送你的源代码来部署。你可以生成 JAR，推送它，一切都会自动检测。

我们可以使用以下命令对此进行打包以进行部署：

```java
./gradlew assemble

```

这将在 `build/libs` 目录中创建一个 jar 文件。此时，你可以执行以下命令。以下命令将你的部署目标指向 PWS（`run.pivotal.io`）内的空间：

```java
$ cf login -a api.run.pivotal.io -u <account email> -p <password> -o <organization> -s development

API endpoint: api.run.pivotal.io
Authenticating...
OK

Targeted org <account org>

Targeted space development

API endpoint:   https://api.run.pivotal.io (API version: 2.33.0) 
User:           <account email> 
Org:            <account organization> 
Space:          <account space>

```

一旦你成功登录，你可以使用以下命令推送你的 jar 文件。你需要想出一个可用的名字：

```java
$ cf push your-app-name -p build/libs/masterSpringMvc-0.0.1-SNAPSHOT.jar

Creating app msmvc4 in org Northwest / space development as wlund@pivotal.io...
OK
Creating route msmvc4.cfapps.io...
OK
Binding msmvc4.cfapps.io to msmvc4...
OK
Uploading msmvc4...
Uploading app files from: build/libs/masterSpringMvc-0.0.1-SNAPSHOT.jar
Uploading 690.8K, 108 files
Done uploading 
OK
Starting app msmvc4 in org <Organization> / space development as <account email>
-----> Downloaded app package (15M)
-----> Java Buildpack Version: v3.1 | https://github.com/cloudfoundry/java-buildpack.git#7a538fb
-----> Downloading Open Jdk JRE 1.8.0_51 from https://download.run.pivotal.io/openjdk/trusty/x86_64/openjdk-1.8.0_51.tar.gz (1.5s)
 Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.4s)
-----> Downloading Open JDK Like Memory Calculator 1.1.1_RELEASE from https://download.run.pivotal.io/memory-calculator/trusty/x86_64/memory-calculator-1.1.1_RELEASE (0.1s)
 Memory Settings: -Xmx768M -Xms768M -XX:MaxMetaspaceSize=104857K -XX:MetaspaceSize=104857K -Xss1M
-----> Downloading Spring Auto Reconfiguration 1.7.0_RELEASE from https://download.run.pivotal.io/auto-reconfiguration/auto-reconfiguration-1.7.0_RELEASE.jar (0.0s)
-----> Uploading droplet (59M)
0 of 1 instances running, 1 starting
1 of 1 instances running

App started
OK
App msmvc4 was started using this command `CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-1.1.1_RELEASE -memorySizes=metaspace:64m.. -memoryWeights=heap:75,metaspace:10,stack:5,native:10 -totMemory=$MEMORY_LIMIT) && SERVER_PORT=$PORT $PWD/.java-buildpack/open_jdk_jre/bin/java -cp $PWD/.:$PWD/.java-buildpack/spring_auto_reconfiguration/spring_auto_reconfiguration-1.7.0_RELEASE.jar -Djava.io.tmpdir=$TMPDIR -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY org.springframework.boot.loader.JarLauncher`

Showing health and status for app msmvc4 in org <Organization> / space development as <Account Email>
OK

requested state: started
instances: 1/1
usage: 1G x 1 instances
urls: msmvc4.cfapps.io
last uploaded: Tue Jul 28 22:04:08 UTC 2015
stack: cflinuxfs2
buildpack: java-buildpack=v3.1-https://github.com/cloudfoundry/java-buildpack.git#7a538fb java-main open-jdk-like-jre=1.8.0_51 open-jdk-like-memory-calculator=1.1.1_RELEASE spring-auto-reconfiguration=1.7.0_RELEASE

 state     since                    cpu    memory         disk         details 
#0   running   2015-07-28 03:05:04 PM   0.0%   450.9M of 1G   137M of 1G

```

平台代表你做了很多事情。它为你配置一个容器并检测所需的构建包，在这种情况下，是 Java。

它随后安装所需的 JDK 并上传我们指向的应用程序。它为应用程序创建一个路由，并向我们报告，然后为我们启动应用程序。

现在，你可以在开发控制台中查看应用程序：

![组装应用程序](img/2117_09_06.jpg)

在选择高亮的路由后，应用程序将可供使用。访问[`msmvc4.cfapps.io`](http://msmvc4.cfapps.io)，然后你会看到以下截图：

![组装应用程序](img/2117_09_01.jpg)

太棒了！

唯一不能工作的是文件上传。然而，我们将在一分钟内解决这个问题。

## 激活 Redis

在你的应用程序服务中，你可以选择许多服务之一。其中之一是 Redis Cloud，它有一个包含 30MB 存储空间的免费计划。请选择这个计划。

在表单中，选择你喜欢的任何名字并将服务绑定到你的应用程序上。默认情况下，Cloud Foundry 将在你的环境中注入一些与该服务相关的属性：

+   `cloud.services.redis.connection.host`

+   `cloud.services.redis.connection.port`

+   `cloud.services.redis.connection.password`

+   `cloud.services.redis.connection.uri`

这些属性将始终遵循相同的约定，因此当你添加更多服务时，跟踪你的服务将会很容易。

默认情况下，Cloud Foundry 启动 Spring 应用程序并激活 Cloud 配置文件。

我们可以利用这一点，在`src/main/resources`中创建一个`application-cloud.properties`文件，当我们的应用程序在 PWS 上运行时将使用此文件：

```java
spring.profiles.active=prod,redis

spring.redis.host=${cloud.services.redis.connection.host}
spring.redis.port=${cloud.services.redis.connection.port}
spring.redis.password=${cloud.services.redis.connection.password}

upload.pictures.uploadPath=file:/tmp
```

这将把我们的 Redis 实例绑定到我们的应用程序上，并激活两个额外的配置文件：`prod`和`redis`。

我们还更改了上传图片将到达的路径。请注意，在云上使用文件系统遵循不同的规则。有关更多详细信息，请参阅以下链接：

[`docs.run.pivotal.io/devguide/deploy-apps/prepare-to-deploy.html#filesystem`](http://docs.run.pivotal.io/devguide/deploy-apps/prepare-to-deploy.html#filesystem)

我们最后需要做的就是禁用 Spring Session 的一个功能，该功能在我们的托管实例上不可用：

```java
@Bean
@Profile({"cloud", "heroku"})
public static ConfigureRedisAction configureRedisAction() {
    return ConfigureRedisAction.NO_OP;
}
```

### 注意

更多信息，请访问[`docs.spring.io/spring-session/docs/current/reference/html5/#api-redisoperationssessionrepository-sessiondestroyedevent`](http://docs.spring.io/spring-session/docs/current/reference/html5/#api-redisoperationssessionrepository-sessiondestroyedevent)。

你会看到这个配置也将应用于 Heroku。

就这样。你可以重新组装你的 Web 应用程序并再次推送。现在，你的会话和应用程序缓存将存储在 Redis 上！

您可能想探索市场中的其他可用功能，例如绑定到数据或消息服务、扩展应用程序以及管理超出本介绍范围的应用程序的健康状况。

享受乐趣并享受平台提供的生产力！

# 在 Heroku 上部署您的 Web 应用程序

在本节中，我们将免费将您的应用程序部署到 Heroku。我们甚至将使用免费的 Redis 实例来存储我们的会话和缓存。

## 安装工具

创建 Heroku 应用程序的第一步是下载位于[`toolbelt.heroku.com`](https://toolbelt.heroku.com)的命令行工具。

在 Mac 上，您也可以使用`brew`命令安装它：

```java
> brew install heroku-toolbelt

```

在 Heroku 上创建一个账户，并使用`heroku login`将工具包链接到您的账户：

```java
> heroku login
Enter your Heroku credentials.
Email: geowarin@mail.com
Password (typing will be hidden):
Authentication successful.

```

然后，转到您的应用程序根目录，并输入`heroku create appName --region eu`。将`appName`替换为您选择的名称。如果您不提供名称，它将自动生成：

```java
> heroku create appname --region eu
Creating appname... done, region is eu
https://appname.herokuapp.com/ | https://git.heroku.com/appname.git
Git remote heroku added

```

如果您已经创建了一个带有 UI 的应用程序，请转到您的应用程序根目录，并简单地添加远程`heroku git:remote -a yourapp`。

这些命令的作用是为我们的 Git 仓库添加一个名为`heroku`的 Git 远程。在 Heroku 上部署的过程只是将您的其中一个分支推送到 Heroku。远程安装的 Git 钩子将处理其余部分。

如果您输入`git remote -v`命令，您应该看到`heroku`版本：

```java
> git remote -v
heroku    https://git.heroku.com/appname.git (fetch)
heroku    https://git.heroku.com/appname.git (push)
origin    https://github.com/Mastering-Spring-MVC-4/mastering-spring-mvc4-code.git (fetch)
origin    https://github.com/Mastering-Spring-MVC-4/mastering-spring-mvc4-code.git (push)

```

## 设置应用程序

运行 Gradle 应用程序与 Heroku 需要两个要素：一个名为`stage`的构建文件中的任务，以及一个包含运行我们应用程序的命令的微小文件，称为`ProcFile`。

### Gradle

Gradle 构建包将自动尝试在您的应用程序根目录运行`./gradlew stage`命令。

### 注意

您可以在[`github.com/heroku/heroku-buildpack-gradle`](https://github.com/heroku/heroku-buildpack-gradle)上获取有关 Gradle 构建包的更多信息。

我们还没有“stage”任务。将以下代码添加到您的`build.gradle`文件中：

```java
task stage(type: Copy, dependsOn: [clean, build]) {
    from jar.archivePath
    into project.rootDir
    rename {
        'app.jar'
    }
}
stage.mustRunAfter(clean)

clean << {
    project.file('app.jar').delete()
}
```

这将定义一个名为`stage`的任务，该任务将复制 Spring Boot 在应用程序根目录生成的 jar 文件，并将其命名为`app.jar`。

这样找罐子会容易得多。`stage`任务依赖于`clean`任务和`build`任务，这意味着这两个任务都会在`stage`任务开始之前执行。

默认情况下，Gradle 会尝试优化任务依赖图。因此，我们必须提供提示并强制`clean`任务在`stage`之前运行。

最后，我们在已存在的`clean`任务中添加一个新的指令，即删除生成的`app.jar`文件。

现在，如果您运行`./gradlew stage`，它应该会运行测试并将打包的应用程序放在项目的根目录下。

### Procfile

当 Heroku 检测到 Gradle 应用程序时，它将自动运行一个安装了 Java 8 的容器。因此，我们几乎没有配置要处理。

我们需要一个包含运行我们应用程序所使用的 shell 命令的文件。在你的应用程序根目录下创建一个名为 `Procfile` 的文件：

```java
web: java -Dserver.port=$PORT -Dspring.profiles.active=heroku,prod -jar app.jar
```

这里有几个需要注意的事项。首先，我们将我们的应用程序声明为一个网络应用程序。我们还使用环境变量重新定义了应用程序将运行的端口。这非常重要，因为你的应用程序将与许多其他应用程序共存，并且每个应用程序只分配一个端口。

最后，你可以看到我们的应用程序将使用两个配置文件运行。第一个是我们在上一章中创建的 `prod` 配置文件，用于优化性能，以及我们即将创建的新 `heroku` 配置文件。

## 一个 Heroku 配置文件

我们不想将敏感信息，例如我们的 Twitter 应用密钥，放入源代码控制。因此，我们必须创建一些属性，这些属性将从应用程序环境中读取：

```java
spring.social.twitter.appId=${twitterAppId}
spring.social.twitter.appSecret=${twitterAppSecret}
```

为了使这生效，你必须在 Heroku 上配置我们之前讨论的两个环境变量。你可以使用工具带完成此操作：

```java
> heroku config:set twitterAppId=appId
```

或者，你可以转到你的仪表板，并在设置选项卡中配置环境：

![一个 Heroku 配置文件](img/2117_09_02.jpg)

### 注意

访问 [`devcenter.heroku.com/articles/config-vars`](https://devcenter.heroku.com/articles/config-vars) 获取更多信息。

## 运行你的应用程序

现在是时候在 Heroku 上运行我们的应用程序了！

如果你还没有这样做，请将所有更改提交到你的 master 分支。现在，只需使用 `git push heroku master` 将你的 master 分支推送到 `heroku` 远程。这将下载所有依赖项并从头开始构建你的应用程序，所以这可能需要一点时间：

```java
> git push heroku master
Counting objects: 1176, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (513/513), done.
Writing objects: 100% (1176/1176), 645.63 KiB | 0 bytes/s, done.
Total 1176 (delta 485), reused 1176 (delta 485)
remote: Compressing source files... done.
remote: Building source:
remote:
remote: -----> Gradle app detected
remote: -----> Installing OpenJDK 1.8... done
remote: -----> Building Gradle app...
remote:        WARNING: The Gradle buildpack is currently in Beta.
remote: -----> executing ./gradlew stage
remote:        Downloading https://services.gradle.org/distributions/gradle-2.3-all.zip

...

remote:        :check
remote:        :build
remote:        :stage
remote:
remote:        BUILD SUCCESSFUL
remote:
remote:        Total time: 2 mins 36.215 secs
remote: -----> Discovering process types
remote:        Procfile declares types -> web
remote:
remote: -----> Compressing... done, 130.1MB
remote: -----> Launching... done, v4
remote:        https://appname.herokuapp.com/ deployed to Heroku
remote:
remote: Verifying deploy.... done.
To https://git.heroku.com/appname.git
* [new branch]      master -> master

```

一旦应用程序构建完成，它将自动运行。输入 `heroku logs` 查看最新日志或 `heroku logs -t` 来跟踪它们。

你可以在控制台中看到你的应用程序正在运行，如果一切按计划进行，你将能够连接到 [`yourapp.herokuapp.com`](http://yourapp.herokuapp.com)。如下截图所示：

![运行你的应用程序](img/2117_09_03.jpg)

我们已经上线了！是时候告诉你的朋友了！

## 激活 Redis

要在我们的应用程序中激活 Redis，我们可以选择几种替代方案。Heroku Redis 扩展是测试版。它完全免费，提供 20 MB 的存储、分析和日志。

### 注意

访问 [`elements.heroku.com/addons/heroku-redis`](https://elements.heroku.com/addons/heroku-redis) 获取更多详情。

在这个阶段，你必须提供你的信用卡详细信息才能继续。

要为你的应用程序安装 Redis 扩展，请输入以下命令：

```java
heroku addons:create heroku-redis:test

```

现在，我们已经激活了扩展，当我们的应用程序在 Heroku 上运行时，将有一个名为 `REDIS_URL` 的环境变量可用。

你可以使用 `heroku config` 命令检查变量是否已定义：

```java
> heroku config
=== masterspringmvc Config Vars
JAVA_OPTS:        -Xmx384m -Xss512k -XX:+UseCompressedOops
REDIS_URL:        redis://x:xxx@ec2-xxx-xx-xxx-xxx.eu-west-1.compute.amazonaws.com:6439

```

由于 `RedisConnectionFactory` 类不理解 URI，我们需要对其进行一点调整：

```java
@Configuration
@Profile("redis")
@EnableRedisHttpSession
public class RedisConfig {

    @Bean
    @Profile("heroku")
    public RedisConnectionFactory redisConnectionFactory() throws URISyntaxException {
        JedisConnectionFactory redis = new JedisConnectionFactory();

        String redisUrl = System.getenv("REDIS_URL");
        URI redisUri = new URI(redisUrl);
        redis.setHostName(redisUri.getHost());
        redis.setPort(redisUri.getPort());
        redis.setPassword(redisUri.getUserInfo().split(":", 2)[1]);

        return redis;
    }

    @Bean
         @Profile({"cloud", "heroku"})
    public static ConfigureRedisAction configureRedisAction() {
        return ConfigureRedisAction.NO_OP;
    }
}
```

我们现在在`RedisConfig`类中有两个针对 Heroku 特定的 bean。这些 bean 只有在`redis`和`heroku`配置文件都激活的情况下才会生效。

注意，我们还禁用了某些 Spring Session 配置。

Spring Session 通常会通过 Redis Pub/Sub 接口监听与销毁会话密钥相关的事件。

它将自动尝试在启动时配置 Redis 环境以激活监听器。在我们的这种安全环境中，除非你有管理员权限，否则不允许添加监听器。

在我们的情况下，这些 redis 监听器并不重要，因此我们可以安全地禁用此行为。更多信息，请访问[`docs.spring.io/spring-session/docs/current/reference/html5/#api-redisoperationssessionrepository-sessiondestroyedevent`](http://docs.spring.io/spring-session/docs/current/reference/html5/#api-redisoperationssessionrepository-sessiondestroyedevent)。

我们需要修改我们的`Procfile`文件，以便 Heroku 使用`redis`配置文件运行我们的应用程序：

```java
web: java -Dserver.port=$PORT -Dspring.profiles.active=heroku,redis,prod -jar app.jar
```

提交你的更改并将代码推送到 Heroku。

# 提高你的应用程序

我们已经在线部署了一个相当不错的应用程序，但除非你让它变得如此，否则它既不实用也不原创。

努力让它变得更好、更个性化。一旦你为你的成就感到自豪，就在 Twitter 上使用`#masterspringmvc`标签分享你的应用程序 URL。

努力推送最好的应用程序。我们还有很多事情没有做。以下是一些想法：

+   删除用户的旧照片以避免保留未使用的照片

+   使用 Twitter 认证信息填写用户资料

+   与用户的账户互动

+   通过 WebSocket 通道查看你应用上的实时搜索

让你的想象力飞扬！

我的版本的应用程序部署在[`masterspringmvc.herokuapp.com`](http://masterspringmvc.herokuapp.com)。我将改进一些细节，使应用程序变得更加响应式。试着找出差异！

# 摘要

由于 Spring Boot，将我们的应用程序部署到云服务提供商上非常简单，因为它是一个可运行的 jar 文件。如今，云部署非常经济实惠，部署 Java 应用程序几乎变得太容易。

通过 Redis 支持的会话，我们为可扩展的应用程序奠定了基础。确实，我们可以轻松地添加多个服务器在负载均衡器后面，并按需吸收高流量。

唯一不可扩展的是我们的 WebSocket，它需要在消息代理（如 Rabbit MQ）上运行，需要进行额外的工作。

我确实记得一个时期，找到运行 Tomcat 的主机既罕见又昂贵。那些日子已经一去不复返了，未来属于网络开发者，所以让它发生吧！

在下一章中，我们将看到我们可以做什么来使我们的应用程序变得更好，讨论我们没有涉及的技术，一般性地讨论 Spring 生态系统，以及现代 Web 应用程序的挑战。
