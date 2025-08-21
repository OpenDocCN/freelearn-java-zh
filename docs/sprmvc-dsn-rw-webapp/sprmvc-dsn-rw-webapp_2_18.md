# 第二十四章：将您的 Web 应用部署到云端

在本章中，我们将参观不同的云提供商，了解分布式架构的挑战和好处，并了解如何将您的 Web 应用程序部署到 Pivotal Web Services 和 Heroku。

# 选择您的主机

云托管有许多形式。对于开发人员来说，选择主要将在平台即服务（PaaS）和基础设施即服务（IaaS）之间进行。

使用最新的技术，您通常会有一台裸机，您可以管理并在其中安装应用程序所需的所有服务。

如果我们不考虑 Docker 等技术（这绝对是令人惊叹的，您绝对应该尝试一下），这与传统托管非常相似，其中您的运营团队将不得不设置和维护应用程序可以运行的环境。

另一方面，PaaS 使得在开发应用程序时轻松部署应用程序，只需简单的推送即可部署工作流程。

最知名的提供商有：

+   由 Pivotal 支持的 Cloud Foundry

+   由红帽提供的 OpenShift

+   Heroku 于 2010 年被 Salesforce 收购

这三个提供商各有优缺点。我将尝试为您概述这些。

## Cloud Foundry

由 Pivotal 支持，Pivotal Web 服务是由 Spring 背后的公司 Pivotal 维护的开源 PaaS Cloud Foundry 运行，并提供有趣的套餐。

他们提供 60 天的免费试用，其定价是您为实例分配的内存和您拥有的实例数量的函数。

他们的价格范围从每月 2.70 美元的最小（128 Mb）实例到每月 43.20 美元的 2 GB 实例。

如果您想尝试一下，免费试用不需要信用卡。他们有一个市场，可以轻松安装服务，如 Redis 或 Postgre SQL，但免费选项相对有限。他们有一个很好的命令行实用程序，可以从控制台管理您的应用程序。您可以使用构建包，也可以直接推送 JAR 文件进行部署。

### 提示

构建包将尝试猜测您正在使用的堆栈，并以最标准的方式构建您的应用程序（Maven 的`mvn package`，Gradle 的`./gradlew stage`等）。

### 注意

请参考以下网址提供的教程，将您的应用程序部署到 Cloud Foundry：

[`docs.cloudfoundry.org/buildpacks/java/gsg-spring.html`](http://docs.cloudfoundry.org/buildpacks/java/gsg-spring.html)

## OpenShift

**OpenShift**由 Red Hat 维护，并由 OpenShift Origin 提供支持，这是一个在 Google 的 Kubernetes 之上运行 Docker 容器的开源设施。

它的定价合理，并提供了很多自由度，因为它既是 PaaS 又是 IaaS。其定价是基于齿轮、运行应用程序的容器或服务（如 Jenkins 或数据库）。

OpenShift 有一个免费计划，提供三个小齿轮。您的应用程序每月必须闲置 24 小时，除非您输入您的计费信息。

额外或更大的齿轮按月收费，最小的约为 15 美元，最大的为 72 美元。

要在 OpenShift 上部署 Spring Boot 应用程序，您将需要使用自定义的 Do It Yourself cartridge。这比其他基于构建包的 PaaS 需要更多的工作，但也更容易配置。

查看博客文章，了解有关在 OpenShift 上使用 Spring Boot 的教程，网址为[`blog.codeleak.pl/2015/02/openshift-diy-build-spring-boot.html`](http://blog.codeleak.pl/2015/02/openshift-diy-build-spring-boot.html)。

## Heroku

Heroku 是一个知名的 PaaS，拥有广泛的文档和基于构建包的代码中心方法。它可以连接到许多称为附加组件的服务，但使用它们需要您的计费信息。

对于一个免费项目来说真的很有趣，而且很快就可以开始。不足之处是，如果您想扩展规模，它的直接成本将超过每月 25 美元。免费实例在 30 分钟的不活动后将进入睡眠模式，这意味着免费的 Heroku 应用程序加载时间总是需要 30 秒。

Heroku 拥有出色的管理仪表板和命令行工具。在本章中，我选择了 Heroku，因为它非常直接。您在这里掌握的概念适用于大多数 PaaS。

只要您不使用 Redis 附加组件，您可以遵循本章的大部分内容并部署您的应用程序，而无需提供信用卡信息。如果选择免费计划，您将不会被收费。

# 将您的 Web 应用程序部署到 Pivotal Web 服务

如果您想将您的应用程序部署到 Pivotal Web 服务（PWS），请参考本节。

## 安装 Cloud Foundry CLI 工具

创建 Cloud Foundry 应用程序的第一步是在 PWS 上设置一个帐户。这在[`docs.run.pivotal.io/starting/`](http://docs.run.pivotal.io/starting/)中有记录。

您将被要求创建一个组织，每个新组织都将在组织内创建一个默认空间（开发）。如下图所示：

![安装 Cloud Foundry CLI 工具](img/image00987.jpeg)

在左侧导航栏中，您将看到一个指向**工具**的链接，您可以从中下载 CLI。它也可以从开发者控制台中获得。选择适合您操作系统的适当软件包：

![安装 Cloud Foundry CLI 工具](img/image00988.jpeg)

## 组装应用程序

我们的应用程序只需要组装以进行部署。

PWS 的好处是您无需推送源代码即可部署。您可以生成 JAR，推送它，一切都将被自动检测。

我们可以使用以下命令将其打包以进行部署：

```java
./gradlew assemble

```

这将在`build/libs`目录中创建一个 jar 文件。此时，您可以执行以下命令。以下命令将将您的部署目标定位到 PWS（`run.pivotal.io`）中的空间：

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

成功登录后，您可以使用以下命令推送您的 jar 文件。您需要想出一个可用的名称：

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

平台正在为您执行很多工作。它提供了一个容器，并检测所需的构建包，本例中为 Java。

然后安装所需的 JDK 并上传我们指向的应用程序。它创建了一个应用程序的路由，并向我们报告，然后为我们启动了应用程序。

现在您可以在开发者控制台上查看应用程序：

![组装应用程序](img/image00989.jpeg)

选择突出显示的路由后，应用程序将可供使用。访问[`msmvc4.cfapps.io`](http://msmvc4.cfapps.io)，然后您将看到以下截图：

![组装应用程序](img/image00990.jpeg)

太棒了！

唯一还不能工作的是文件上传。但是，我们将在一分钟内解决这个问题。

## 激活 Redis

在您的应用程序服务中，您可以在许多服务之间进行选择。其中之一是 Redis Cloud，它具有 30MB 免费存储空间的免费计划。继续选择此计划。

在表格中，选择你喜欢的任何名称，并将服务绑定到你的应用程序。默认情况下，Cloud Foundry 将在你的环境中注入与服务相关的一些属性：

+   `cloud.services.redis.connection.host`

+   `cloud.services.redis.connection.port`

+   `cloud.services.redis.connection.password`

+   `cloud.services.redis.connection.uri`

这些属性将始终遵循相同的约定，因此在添加更多服务时很容易跟踪您的服务。

默认情况下，Cloud Foundry 启动 Spring 应用程序并激活 Cloud 配置文件。

我们可以利用这一点，在`src/main/resources`中创建一个`application-cloud.properties`文件，当我们的应用程序在 PWS 上运行时将使用该文件：

```java
spring.profiles.active=prod,redis

spring.redis.host=${cloud.services.redis.connection.host}
spring.redis.port=${cloud.services.redis.connection.port}
spring.redis.password=${cloud.services.redis.connection.password}

upload.pictures.uploadPath=file:/tmp
```

这将绑定我们的 Redis 实例到我们的应用程序，并激活两个额外的配置文件：`prod`和`redis`。

我们还更改了上传图片的路径。请注意，在云上使用文件系统遵守不同的规则。请参考以下链接获取更多详细信息：

[`docs.run.pivotal.io/devguide/deploy-apps/prepare-to-deploy.html#filesystem`](http://docs.run.pivotal.io/devguide/deploy-apps/prepare-to-deploy.html#filesystem)

我们需要做的最后一件事是停用一个 Spring Session 功能，在我们托管的实例上将不可用：

```java
@Bean
@Profile({"cloud", "heroku"})
public static ConfigureRedisAction configureRedisAction() {
    return ConfigureRedisAction.NO_OP;
}
```

### 注意

有关更多信息，请访问[`docs.spring.io/spring-session/docs/current/reference/html5/#api-redisoperationssessionrepository-sessiondestroyedevent`](http://docs.spring.io/spring-session/docs/current/reference/html5/#api-redisoperationssessionrepository-sessiondestroyedevent)。

您将看到此配置也将应用于 Heroku。

就是这样。您可以重新组装您的 Web 应用程序并再次推送它。现在，您的会话和应用程序缓存将存储在 Redis 中！

您可能希望探索市场，寻找其他可用功能，例如绑定到数据或消息服务，扩展应用程序以及管理超出本介绍范围的应用程序的健康状况。

玩得开心，享受平台提供的生产力！

# 在 Heroku 上部署您的 Web 应用程序

在本节中，我们将免费在 Heroku 上部署您的应用程序。我们甚至将使用免费的 Redis 实例来存储我们的会话和缓存。

## 安装工具

创建 Heroku 应用程序的第一件事是下载[`toolbelt.heroku.com`](https://toolbelt.heroku.com)上可用的命令行工具。

在 Mac 上，您还可以使用`brew`命令进行安装：

```java
> brew install heroku-toolbelt

```

在 Heroku 上创建一个帐户，并使用`heroku login`将工具包链接到您的帐户：

```java
> heroku login
Enter your Heroku credentials.
Email: geowarin@mail.com
Password (typing will be hidden):
Authentication successful.

```

然后，转到您的应用程序根目录，输入`heroku create appName --region eu`。将`appName`替换为您选择的名称。如果您不提供名称，它将自动生成：

```java
> heroku create appname --region eu
Creating appname... done, region is eu
https://appname.herokuapp.com/ | https://git.heroku.com/appname.git
Git remote heroku added

```

如果您已经使用 UI 创建了一个应用程序，那么转到您的应用程序根目录，然后简单地添加远程`heroku git:remote -a yourapp`。

这些命令的作用是向我们的 Git 存储库添加一个名为`heroku`的 Git 远程。在 Heroku 上部署的过程只是将您的分支之一推送到 Heroku。远程安装的 Git 挂钩将负责其余的工作。

如果您输入`git remote -v`命令，您应该会看到`heroku`版本：

```java
> git remote -v
heroku    https://git.heroku.com/appname.git (fetch)
heroku    https://git.heroku.com/appname.git (push)
origin    https://github.com/Mastering-Spring-MVC-4/mastering-spring-mvc4-code.git (fetch)
origin    https://github.com/Mastering-Spring-MVC-4/mastering-spring-mvc4-code.git (push)

```

## 设置应用程序

我们需要两个要素来在 Heroku 上运行 Gradle 应用程序：构建文件中的一个名为`stage`的任务，以及一个包含用于运行我们的应用程序的命令的小文件，名为`ProcFile`。

### Gradle

Gradle 构建包将自动尝试在您的应用程序根目录上运行`./gradlew stage`命令。

### 注意

您可以在[`github.com/heroku/heroku-buildpack-gradle`](https://github.com/heroku/heroku-buildpack-gradle)上获取有关 Gradle 构建包的更多信息。

我们还没有"stage"任务。将以下代码添加到您的`build.gradle`文件中：

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

这将定义一个名为`stage`的任务，它将复制 Spring Boot 在应用程序根目录生成的 jar 文件，并将其命名为`app.jar`。

这样查找 jar 文件会容易得多。`stage`任务依赖于`clean`任务和`build`任务，这意味着在开始`stage`任务之前，两者都将被执行。

默认情况下，Gradle 将尝试优化任务依赖图。因此，我们必须提供一个提示，并强制运行`stage`之前运行`clean`任务。

最后，我们向已经存在的`clean`任务添加了一个新的指令，即删除生成的`app.jar`文件。

现在，如果您运行`./gradlew stage`，它应该运行测试并将打包的应用程序放在项目的根目录。

### Procfile

当 Heroku 检测到一个 Gradle 应用程序时，它将自动运行一个安装了 Java 8 的容器。因此，我们需要非常少的配置。

我们需要一个包含用于运行我们的应用程序的 shell 命令的文件。在您的应用程序根目录创建一个名为`Procfile`的文件：

```java
web: java -Dserver.port=$PORT -Dspring.profiles.active=heroku,prod -jar app.jar
```

这里有几件事情需要注意。首先，我们将我们的应用程序声明为 Web 应用程序。我们还重新定义了应用程序将在其上运行的端口，使用环境变量。这非常重要，因为您的应用程序将与许多其他应用程序共存，每个应用程序只分配一个端口。

最后，您可以看到我们的应用程序将使用两个配置文件运行。第一个是我们在上一章中创建的`prod`配置文件，用于优化性能，还有一个新的`heroku`配置文件，我们将在稍后创建。

## 一个 Heroku 配置文件

我们不希望将诸如我们的 Twitter 应用程序密钥之类的敏感信息放入源代码控制中。因此，我们必须创建一些属性，以从应用程序环境中读取这些属性：

```java
spring.social.twitter.appId=${twitterAppId}
spring.social.twitter.appSecret=${twitterAppSecret}
```

为了使这个工作，您必须在 Heroku 上配置我们之前讨论过的两个环境变量。您可以使用工具包来完成这个任务：

```java
> heroku config:set twitterAppId=appId
```

或者，您可以转到您的仪表板，并在设置选项卡中配置环境：

![一个 Heroku 配置文件](img/image00991.jpeg)

### 注意

访问[`devcenter.heroku.com/articles/config-vars`](https://devcenter.heroku.com/articles/config-vars)获取更多信息。

## 运行您的应用程序

现在是时候在 Heroku 上运行我们的应用程序了！

如果您还没有这样做，请将所有更改提交到主分支。现在，只需将主分支推送到`heroku`远程，使用`git push heroku master`。这将下载所有依赖项并从头构建您的应用程序，因此可能需要一些时间：

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

应用程序构建完成后，它将自动运行。输入`heroku logs`以查看最新日志，或者输入`heroku logs -t`以跟踪它们。

您可以在控制台上看到您的应用程序正在运行，如果一切按计划进行，您将能够连接到[`yourapp.herokuapp.com`](http://yourapp.herokuapp.com)。如下面的截图所示：

![运行您的应用程序](img/image00992.jpeg)

我们在线了！是时候告诉您的朋友了！

## 激活 Redis

要在我们的应用程序中激活 Redis，我们可以在几种选择之间进行选择。Heroku Redis 附加组件是测试版。它完全免费，带有 20MB 的存储空间，分析和日志记录。

### 注意

访问[`elements.heroku.com/addons/heroku-redis`](https://elements.heroku.com/addons/heroku-redis)以获取更多详细信息。

在这个阶段，您将需要提供您的信用卡详细信息以继续。

要为您的应用程序安装 Redis 附加组件，请输入以下内容：

```java
heroku addons:create heroku-redis:test

```

现在，我们已经激活了附加组件，当我们的应用程序在 Heroku 上运行时，将会有一个名为`REDIS_URL`的环境变量可用。

您可以使用`heroku config`命令检查该变量是否已定义：

```java
> heroku config
=== masterspringmvc Config Vars
JAVA_OPTS:        -Xmx384m -Xss512k -XX:+UseCompressedOops
REDIS_URL:        redis://x:xxx@ec2-xxx-xx-xxx-xxx.eu-west-1.compute.amazonaws.com:6439

```

由于`RedisConnectionFactory`类不理解 URI，我们需要稍微调整一下：

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

我们现在在`RedisConfig`类中有两个 Heroku 特定的 bean。这些 bean 只有在`redis`和`heroku`配置文件都激活时才会生效。

请注意，我们还停用了一些 Spring Session 配置。

Spring Session 通常会通过 Redis Pub/Sub 接口监听与销毁会话密钥相关的事件。

它将自动尝试配置 Redis 环境以在启动时激活监听器。在我们这样的安全环境中，除非您拥有管理员访问权限，否则不允许添加监听器。

在我们的情况下，这些 redis 监听器并不是非常重要，所以我们可以安全地禁用这种行为。欲了解更多信息，请访问[`docs.spring.io/spring-session/docs/current/reference/html5/#api-redisoperationssessionrepository-sessiondestroyedevent`](http://docs.spring.io/spring-session/docs/current/reference/html5/#api-redisoperationssessionrepository-sessiondestroyedevent)。

我们需要修改我们的`Procfile`文件，以便 Heroku 使用`redis`配置运行我们的应用程序：

```java
web: java -Dserver.port=$PORT -Dspring.profiles.active=heroku,redis,prod -jar app.jar
```

提交您的更改并将代码推送到 Heroku。

# 改进您的应用程序

我们已经在线部署了一个相当不错的应用程序，但在您进行改进之前，它既不是非常有用，也不是非常原创。

尝试使其更好，更个性化。一旦您为自己的成就感到自豪，请在 Twitter 上使用`#masterspringmvc`标签推文您的应用程序 URL。

尝试推出尽可能最好的应用程序。我们还有很多事情没有做。以下是一些想法：

+   删除用户的旧图片以避免保留未使用的图片

+   使用 Twitter 身份验证信息填充用户配置文件

+   与用户帐户进行交互

+   使用 Web 套接字频道查看应用程序上正在进行的实时搜索

让您的想象力飞翔！

我的版本的应用程序部署在[`masterspringmvc.herokuapp.com`](http://masterspringmvc.herokuapp.com)。我将改进一些细节，使应用程序更具反应性。试着找出不同之处！

# 摘要

在云提供商上部署我们的应用程序非常简单，因为它是一个可运行的 jar 包，这要归功于 Spring Boot。云部署现在非常实惠，部署 Java 应用程序几乎变得太容易了。

通过 Redis 支持的会话，我们奠定了可扩展应用程序的基础。事实上，我们可以轻松地在负载均衡器后面添加多个服务器，并根据需求吸收高流量。

唯一不可扩展的是我们的 WebSocket，它需要额外的工作才能在消息代理（如 Rabbit MQ）之上运行。

我肯定还记得以前很难找到运行 Tomcat 的主机，并且价格昂贵。那些日子已经一去不复返，未来属于 Web 开发人员，所以让它发生吧！

在下一章中，我们将看看如何使我们的应用程序变得更好，讨论我们尚未涵盖的技术，谈论 Spring 生态系统的一般情况以及现代 Web 应用程序的挑战。

