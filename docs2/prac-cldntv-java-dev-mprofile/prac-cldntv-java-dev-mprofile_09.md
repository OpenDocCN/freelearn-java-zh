# *第七章*：使用 Open Liberty、Docker 和 Kubernetes 的 MicroProfile 生态系统

到目前为止，本书的前几章中，我们专注于使用 MicroProfile API 编写云原生应用程序。在本章中，我们将探讨如何运行云原生应用程序。MicroProfile 与其他一些云原生应用程序框架的不同之处在于，MicroProfile 提供了 API 的多个实现。这减少了最终陷入特定实现或发现您所利用的 API 背后的开源社区不如您想象的那么活跃，维护者消失的可能性。此外，不同的实现往往采取不同的设计决策，这可能更适合您的需求。在撰写本文时，MicroProfile API 的最新版本有四个实现：**Open Liberty**、**Payara**、**WebSphere Liberty**和**WildFly**。此外，**Helidon**、**JBoss EAP**、**KumuluzEE**和**Quarkus**实现了之前的版本。

一旦选择了实现方式，您需要将应用程序部署到生产环境中。越来越普遍的做法是使用**Docker**、**Kubernetes**和**服务网格**等技术。这将是本章的重点。

在本章中，我们将涵盖以下主题：

+   将云原生应用程序部署到 Open Liberty

+   使用 Docker 容器化云原生应用程序

+   将云原生应用程序部署到 Kubernetes

+   MicroProfile 和 Service Mesh

在本章中，您将学习如何配置 MicroProfile 应用程序以在 Open Liberty 上运行，将其打包为容器，并部署到像 Red Hat OpenShift 这样的 Kubernetes 运行时中。

# 技术要求

要构建和运行本章中提到的示例，您需要一个装有以下软件的 Mac 或 PC（Windows 或 Linux）：

+   **Java 开发工具包**（**JDK**）- Java 8 或更高版本：[`ibm.biz/GetSemerut`](http://ibm.biz/GetSemerut)

+   Apache Maven：[`maven.apache.org`](https://maven.apache.org)

+   Git 客户端：[`git-scm.com`](https://git-scm.com)

+   Docker 客户端：[`www.docker.com/products`](https://www.docker.com/products)

+   OpenShift 客户端：[`docs.openshift.com/container-platform/4.6/cli_reference/openshift_cli/getting-started-cli.html`](https://docs.openshift.com/container-platform/4.6/cli_reference/openshift_cli/getting-started-cli.html)

本章节中使用的所有源代码均可在 GitHub 上找到，地址为[`github.com/PacktPublishing/Practical-Cloud-Native-Java-Development-with-MicroProfile/tree/main/Chapter07`](https://github.com/PacktPublishing/Practical-Cloud-Native-Java-Development-with-MicroProfile/tree/main/Chapter07)。

# 将云原生应用程序部署到 Open Liberty

在本节中，我们将探讨如何使用 **Open Liberty** 部署 MicroProfile 应用程序。我们选择 Open Liberty 是因为我们是在 Open Liberty 的提交者，但它的关注点是保持与最新 MicroProfile 版本的同步、性能和易用性，这使得它成为任何人的好选择。

如您从其名称中预期的那样，Open Liberty 是一个开源的 Java 运行时，用于构建和部署云原生应用程序。它围绕称为 **功能** 的组件设计，这些组件可以配置以提供满足您的应用程序需求的最小运行时。这意味着如果您的应用程序不使用或不需要 MicroProfile OpenTracing，那么您不需要配置 MicroProfile OpenTracing 功能，并且运行时将更小、更快、更精简——它将更适合您的应用程序需求。Open Liberty 具有编程 API 的功能，例如 MicroProfile API、Java EE、Jakarta EE 和 gRPC。它还具有运行时功能，例如用于与 OpenID Connect 集成进行身份验证的功能。

Open Liberty 主要使用一个简单的 XML 文件格式进行配置，称为 `server.xml` 文件。使用 XML 的原因有几个：

+   Java 内置了对 XML 解析的支持，这是其主要原因之一。

+   XML 模型非常适合分层配置（与属性格式不同）。

+   空格字符不影响文件格式的语义解释（与 YAML 不同）。

当解析配置文件时，Open Liberty 采取忽略任何它不理解配置的方法。这有几个优点。这意味着一个 `server.xml` 文件可以包含对正在使用的 Open Liberty 版本无效的配置，而不会导致启动失败。这也意味着配置中的简单错误不会阻止服务器启动应用程序。

`server.xml` 文件的核心职责之一是配置要加载的功能。通过使用 `server.xml` 文件来配置要使用哪些功能，并通过确保行为更改仅通过新功能引入，Open Liberty 保证配置的行为在各个版本之间保持不变。以下是一个简单的服务器配置示例，它启用了所有 MicroProfile API：

```java
<server>
    <featureManager>
        <feature>microProfile-4.1</feature>
    </featureManager>
</server>
```

Open Liberty 的配置可以集中在一个单独的 `server.xml` 文件中，也可以分散到多个配置文件中。这既促进了配置的共享，也根据服务器配置部署到的环境分离了配置。一个例子可能是开发环境中使用内存数据库，但在生产中可能使用数据库，如 DB2、Oracle 或 MariaDB。这通过两种机制实现。第一种是一个可以显式包含另一个 `server.xml` 文件的 `server.xml` 文件。第二种是使用所谓的 `defaults` 和 `overrides`，这些在主服务器配置文件之前和之后读取。这些目录中的文件按字母顺序读取，提供了配置读取的可预测性。配置文件还可以使用变量替换语法进行参数化。变量可以在 `server.xml` 文件中定义，作为 Java 系统属性，或使用环境变量。`server.xml` 中的变量可以定义多次，变量的最后定义将用于变量解析。变量的定义可能如下所示：

```java
<server>
    <variable name="microProfile.feature" 
        value="microProfile-4.1" />
</server>
```

然后，它可以在其他地方这样引用：

```java
<server>
    <featureManager>
        <feature>${microProfile.feature}</feature>
    </featureManager>
</server>
```

变量也可以有一个默认值，这允许编写始终有效的配置，同时在生产中允许覆盖：

```java
<server>
    <variable name="http.port" defaultValue="9043" />
</server>
```

变量的优先级根据其定义的位置而不同，这使得它们可以轻松覆盖。优先级顺序（后定义的优先级覆盖先定义的优先级）如下：

1.  `server.xml`的默认值

1.  环境变量

1.  `bootstrap.properties` 文件

1.  Java 系统属性

1.  在`s`erver.xml`中定义的变量

1.  服务器启动时定义的变量

这提供了多种简单的方法来根据 Open Liberty 部署到的环境更改其行为。

Open Liberty 允许您将 MicroProfile 应用程序打包为 `WAR` 文件以部署到服务器。MicroProfile 规范对应用程序的打包和部署方式没有意见，因此 Open Liberty 重新使用 Jakarta EE 的 `WAR` 打包模型作为打包应用程序的方式。这很有意义，因为 MicroProfile 利用了几种 Jakarta EE 编程模型，并且使 MicroProfile 应用程序更容易利用 Jakarta EE 中不在 MicroProfile 中的部分，例如 Jakarta EE 的并发工具。它还允许您重用现有的 Maven 和 Gradle 构建工具来打包 MicroProfile 应用程序。

将 `WAR` 文件部署到 Open Liberty 有两种方式。第一种是将 WAR 文件放入 `dropins` 文件夹，第二种是通过 `server.xml` 文件：

```java
<server>
    <webApplication location="myapp.war" />
</server>
```

使用服务器配置方法而不是`dropins`文件夹的主要原因是可以自定义应用程序的运行方式，例如，设置应用程序的`contextRoot`，配置类加载，或配置安全角色绑定。

Open Liberty 支持多种机制来打包应用程序以进行部署。最简单的方法是将应用程序打包为`WAR`文件。但在云原生环境中，这不太可能。Open Liberty 还支持将服务器打包为`zip`文件、可执行的`JAR`文件和 Docker 容器（下一节将描述）。Open Liberty 为 Maven 和 Gradle 提供了插件，使得构建将在 Open Liberty 上运行的应用程序变得简单。这些插件的一个特性是在 Maven 中提供 Open Liberty 的`dev`模式，只需将 Open Liberty Maven 插件添加到`pom.xml`文件中的`plugin`部分，如下所示：

```java
<plugin>
    <groupId>io.openliberty.tools</groupId>
    <artifactId>liberty-maven-plugin</artifactId>
    <version>[3.3.4,)</version>
</plugin> 
```

此配置使插件使用 3.3.4 版本的插件或如果存在的话，使用更近期的版本。

当你运行`liberty:dev` Maven 目标时，插件将编译应用程序，下载运行应用程序所需的任何依赖项，将其部署到 Liberty 中，并使用 Java 调试器支持运行服务器。这允许你在任何代码编辑器中对应用程序进行更改，无论是简单的**vi**编辑器还是功能齐全的 IDE，如**IntelliJ IDEA**或**Eclipse IDE**。

Liberty 的设计使得构建将在容器环境（如 Docker）中运行的应用程序变得非常简单。甚至还有一个用于容器的`dev`模式，可以使用`liberty:devc`来运行。下一节将讨论如何创建容器作为部署工件。

# 使用 Docker 容器化云原生应用程序

在本节中，我们将探讨如何容器化 MicroProfile 应用程序。容器化的关键有两部分：第一是创建镜像，第二是运行该镜像。虽然 Docker 不是第一个进行容器化的产品，但它以开发者和管理员能够理解的方式普及了这一概念。Cloud Foundry 是一个常见的早期替代品，它有类似的概念，但将它们隐藏为内部实现细节，而不是作为一级概念。使用 Docker，这两个概念被分为两部分，通过创建镜像时使用的`docker build`命令和运行镜像时使用的`docker run`命令暴露出来。这些概念进一步扩展，成为标准化，这意味着现在有多个替代方案用于`docker build`和`docker run`。

## 容器镜像

**容器镜像** 是容器部署的工件。容器镜像包含运行应用程序所需的一切。这意味着容器镜像可以从一个环境移动到另一个环境，有信心它在同一方式下运行。座右铭是“一次思考，一次创建，到处运行”；然而，这也有一些限制。容器与 CPU 架构相关联，因此为 x86 CPU 设计的容器在没有 CPU 指令转换层（如 Rosetta 2，将 Mac x86 指令转换为 Mac ARM 指令以支持在具有 M 系列 ARM 处理器的 Mac 上运行的 x86 Mac 应用程序）的情况下，无法在 ARM 或 Power CPU 上运行。

**Dockerfile** 是创建容器镜像的一系列指令。一个 Dockerfile 首先声明一个基于的命名镜像，然后确定一系列步骤以将额外的内容添加到容器镜像中。常见的做法可能是基于包含 **操作系统**（**OS**）、Java 镜像或预包装应用程序运行时（如 Open Liberty）的镜像。

虽然将容器镜像想象成一个包含镜像中所有内容的单个大文件很方便，但这并不是容器镜像的工作方式。容器镜像由一系列通过 SHA 哈希标识的层组成。这提供了三个关键优势：

+   它减少了存储图像所需的存储空间。如果你有 30 个基于公共基础镜像的图像，你只需存储一次那个公共基础镜像，而不是 30 次。虽然存储空间相对便宜，但如果你有大量应用程序，容器之间的文件重复很快就会累积成大量。

+   它减少了带宽需求。当你将容器镜像传输到和从 **容器注册库** 时，你不需要上传或下载你已经拥有的层。很可能你会有很多容器镜像，但一个常见的操作系统镜像和一个常见的 JVM 镜像。

+   它减少了构建时间。如果层的输入没有变化，就没有必要重新构建它。层的输入是你在该行所做的任何更改，加上上一行的输出。

每个层都有一个指向其构建在其上的层的指针。当你创建一个容器镜像时，它由基础镜像的所有层和一个 Dockerfile 中每行的单个新层组成。一个用于打包简单 MicroProfile 应用程序的简单 Dockerfile 可能看起来像这样：

```java
FROM open-liberty:full-java11-openj9
ADD src/main/config/liberty/server.xml /config
ADD target/myapp.war /config/dropins
```

这将创建一个基于 Ubuntu 20.04 和 Java SE 11 的容器镜像，使用 OpenJ9 JRE 实现并包含所有可用的 Open Liberty 功能。然后，它将从 Maven 项目的默认位置复制 `server.xml` 和应用程序到 `dropins` 文件夹。这将创建一个包含与 `open-liberty:full-java11-openj9` 镜像相关联的所有层的镜像，以及与该镜像相关联的两个层。

最佳实践：使用多个层

这种方法的优点是当你推送或拉取镜像时，只有尚未存在的层才会被传输。在之前提到的简单 MicroProfile 示例中，当你构建和推送镜像时，只有与应用程序和服务器配置相关的层会被传输。可以这样想：如果你有一个 500MB 大小的基础镜像，而你镜像的层总共是 5MB，那么你的镜像总共将是 505MB，但当你将其推送到容器注册库时，只需要发送 5MB，因为基础镜像已经存在了。

这在设计 Docker 镜像时引发了一些有趣的设计问题。Docker 镜像的目的是显然的，即让它运行起来，并且最好是尽可能快地运行起来。这使得部署新的镜像、扩展它或发生问题时用新的容器替换容器变得更快。构建 Docker 镜像的一个简单方法是将你的应用程序打包在一个单一的 JAR 文件中，将其添加到 Dockerfile 中，然后运行它：

```java
FROM adoptopenjdk:11-openj9
ADD target/myapp.jar /myapp.jar
CMD ["java", "-jar", "/myapp.jar"]
```

这是一种创建 Docker 镜像的流行方式，如果应用程序很小，效果很好，但以这种方式构建的许多应用程序并不小。考虑这种情况，如果那个`jar`文件包含 Open Liberty、应用程序代码和一些开源依赖项。这意味着每次修改应用程序时，都必须重新创建和部署包含所有这些代码的新层。另一方面，如果应用程序被拆分，那么对应用程序的更改将需要更小的上传：

```java
FROM open-liberty:full-java11-openj9
ADD target/OS-dependencies /config/library
ADD target/myapp.war /config/apps/myapp.war
```

在这个例子中，对应用程序代码的更改只会重建最后一层，这意味着上传的文件会更小，分发速度更快。当然，对开源依赖项的更改会导致该层被重新构建，但这些更改的频率通常低于应用程序。如果有许多应用程序共享一组公共库（无论是否为开源），那么创建一个所有应用程序都可以使用的命名基础镜像可能是有意义的。如果容器镜像通常在同一个主机上运行，这将特别有用。

关于层的一个重要理解是，一旦创建，它们就是不可变的。这意味着如果你删除了早期层中创建的文件，它不会从镜像中删除文件；它只是将它们标记为已删除，因此无法访问。这意味着在传输容器镜像时，你会复制文件的字节，但文件永远不会可访问。如果文件是由基础镜像贡献的，这是不可避免的，但如果你控制镜像，那么这是需要避免的。

### Dockerfile 指令

如前所述，Dockerfile 是一系列指令，详细说明了如何创建 Docker 镜像。有几种创建镜像的指令。到目前为止所有示例中的第一个指令是`FROM`指令。

#### FROM

`FROM` 指令定义了您正在创建的镜像的基础容器镜像。所有 Dockerfile 都必须以 `FROM` 开头。Dockerfile 可以有多个 `FROM` 行：这些通常用于创建多阶段构建。一个例子可能是，如果您需要一些额外的工具来构建您的镜像，而这些工具在运行时不需要存在。一个例子可能是 `wget` 或 `curl` 用于下载文件，以及 `unzip` 用于解压缩 zip 文件。一个简单的多阶段构建可能看起来像这样：

```java
FROM ubuntu:21.04 as BUILD
RUN apt-get update
RUN apt-get install -y unzip wget
RUN wget https://example.com/my.zip -O my.zip
RUN unzip my.zip /extract
FROM ubuntu:21.04
COPY /extract /extract --from=BUILD
```

在这个例子中，第一阶段安装了 `wget` 和 `unzip`，下载了一个文件，并将其解压缩。第二阶段从基础镜像开始，然后将提取的文件复制到新的镜像层。如果创建了一个单阶段 Dockerfile，这将导致包含 `unzip` 和 `get` 的二进制文件、zip 文件和 `extract` 的三个额外层的镜像。多阶段构建只包含 `extract`。要使用单阶段 Docker 构建实现这一点，可读性较差，看起来可能像这样：

```java
FROM ubuntu:21.04 
RUN apt-get update && \
    apt-get install -y unzip wget && \
    wget https://example.com/my.zip -O my.zip && \
    unzip my.zip /extract && \
    rm my.zip && \
    apt-get uninstall -y unzip wget && \
    rm -rf /var/lib/apt/lists/*    
```

这个 Dockerfile 使用单个 `RUN` 命令来运行多个命令，以创建单个层，并且必须在结束时撤销每个步骤。最后一行是必需的，用于清理 `apt` 创建的文件。多阶段 Dockerfile 要简单得多。多阶段构建的另一个常见用途是使用第一阶段构建应用程序，然后使用第二阶段运行：

```java
FROM maven as BUILD
COPY myBuild /build
WORKDIR build
RUN mvn package
FROM open-liberty:full-java11-openj9
COPY /target/myapp.war /config/apps/ --from=BUILD
```

#### COPY 和 ADD

`COPY` 和 `ADD` 指令执行类似的功能。`ADD` 指令的功能集合包含了 `COPY` 的功能，因此通常建议只有在需要扩展功能时才使用 `ADD`。这两个指令的第一个参数指定了源文件（或目录），默认情况下解释为从运行构建的机器复制。命令始终相对于运行构建的目录，并且不能使用 `..` 来导航到父目录。正如前一个部分所示，使用 `from` 参数可以将复制重定向到另一个容器镜像。第二个参数是容器中文件应该被复制到的位置。

`ADD` 命令在 `COPY` 命令的基础上提供了一些额外的功能。第一个功能是它允许您将 `URL` 作为第一个参数指定，以从该 URL 下载文件。第二个功能是它可以将 `tar.gz` 文件解压缩到目录中。回到第一个多阶段构建示例，如果输出是一个 `tar.gz` 文件，这意味着它可以简化为以下内容：

```java
FROM ubuntu:21.04 as BUILD
ADD https://example.com/my.tar.gz /extract
```

#### RUN

`RUN` 指令简单地使用操作系统层的 shell 执行一个或多个命令。这允许你做几乎你想做或需要做的任何事情，或者提供在基础操作系统镜像中可用的命令。例如，在基础 Linux 操作系统镜像中通常不包含 `unzip` 或 `wget`，因此除非采取行动安装它们，否则这些命令将失败。每个 `RUN` 指令创建一个新的层，所以如果你在一个 `RUN` 命令中创建一个文件，然后在另一个 `RUN` 命令中删除它，由于层的不可变性，文件将存在但不再可见。因此，通常很重要使用 `&&` 操作符将多个命令连接到单个层中。这个例子之前已经展示过，但在此处重复：

```java
FROM ubuntu:21.04 
RUN apt-get update && \
    apt-get install -y unzip wget && \
    wget https://example.com/my.zip -O my.zip && \
    unzip my.zip /extract && \
    rm my.zip && \
    apt-get uninstall -y unzip wget && \
    rm -rf /var/lib/apt/lists/*    
```

#### ARG 和 ENV

`ARG` 定义了一个在构建时可以指定的构建参数。`ARG` 的值在运行 `docker build` 时通过 `build-arg` 参数设置。如果构建时没有提供，`ARG` 可以有一个默认值。这些构建参数在构建完成后不会持久化，因此它们在运行时不可用，也不会在镜像中持久化。

`ENV` 定义了一个在构建和运行时都可用的环境变量。

这两个都是用相同的方式引用的，所以关键的区别是值的可见性。

#### ENTRIES 和 CMD

当运行一个容器时，你需要发生一些事情，比如启动 Open Liberty 服务器。发生的事情可以通过 Dockerfile 使用 `ENTRYPOINT` 和 `CMD` 指令来定义。这两个指令之间的区别在于它们如何与 `docker run` 命令交互。当运行 Docker 容器时，任何在 Docker 镜像名称之后的参数都会传递到容器中。`CMD` 在没有提供命令行参数的情况下提供了一个默认值。`ENTRYPOINT` 定义了一个将被运行的命令，并且提供给 `docker run` 的任何命令行参数都会在 `ENTRYPOINT` 之后传递。`CMD` 和 `ENTRYPOINT` 有相同的语法。Open Liberty 容器指定了这两个指令，因此基于它们的镜像通常不会指定它们。

#### WORKDIR

`WORKDIR` 指令用于更改未来 `RUN`、`CMD`、`COPY` 和 `ENTRYPOINT` 指令的当前目录。

#### USER

当构建镜像时，用于执行命令的默认用户账户是 `root`。对于某些操作，这是合理且公平的。如果你正在执行操作系统更新，通常需要以 `root` 身份执行。然而，当使用 `root` 账户运行容器时，这是一个明显的安全问题。Dockerfile 有一个 `USER` 指令，它设置了用于 `RUN` 指令以及容器运行时执行的进程的用户账户。这使得将账户设置为非 root 账户变得简单。前面示例中的 Open Liberty 镜像将 `USER` 设置为 `1001`，这意味着基于它的任何先前示例都不会使用 `root` 账户运行，但基于 Java 镜像的示例会。先前 Dockerfile 示例的一个问题是 `ADD` 和 `COPY` 指令将写入文件，因此它们属于 `root` 用户，这可能在运行时引起问题。这可以通过更新 `ADD` 或 `COPY` 指令来更改所有权来解决，如下所示：

```java
FROM open-liberty:full-java11-openj9
ADD --chown=1001:0 src/main/config/liberty/server.xml/config
ADD --chown=1001:0 target/myapp.war /config/dropins
```

或者，可以使用 `RUN` 指令来执行 `chown` 命令行工具。这将创建一个新的层，但如果 `ADD` 或 `COPY` 指令移动多个文件，而只有一些文件需要更改所有权，则可能需要这样做。

虽然 Dockerfile 是创建容器镜像最常用的方式，但还有其他构建容器镜像的方法。以下是一些示例。

### 源到镜像

**源到镜像**（**S2I**）是一种技术，它将应用程序源代码转换为容器镜像。与创建 **Dockerfile** 不同，**S2I** 构建器会摄取源代码，运行构建，并将其编码到容器镜像中。这使得开发人员可以专注于应用程序代码，而不是容器创建。通过在 **S2I** 构建器中编码构建容器的最佳实践，它可以在应用程序之间重用，使得一组应用程序都拥有精心设计的容器镜像的可能性更大。有适用于许多语言和框架的 **S2I** 构建器，包括 Open Liberty。**S2I** 是 Red Hat 创建的开源技术，旨在帮助开发人员采用 OpenShift，尽管它可以，并且确实被用来创建可以在任何地方运行的容器。

### 云原生构建包

`WAR` 文件。

创建容器镜像后，下一步是运行它。虽然开发人员可能会在桌面运行时使用 Docker 来运行镜像，但在生产中运行容器镜像最常见的方式是使用 Kubernetes，我们将在下一节中讨论。

# 将云原生应用程序部署到 Kubernetes

**Kubernetes** 最初是谷歌的一个项目，旨在让他们能够大规模管理软件。随后，它转变为一个由 **云原生计算基金会**（**CNCF**）管理的开源项目，并得到了整个行业的贡献者。每个主要（以及大多数次要）的公共云提供商都使用 Kubernetes 来管理容器的部署。还有一些私有云产品，如 Red Hat OpenShift，提供 Kubernetes 的发行版，用于在本地或公共云上部署，但仅限于单一公司。

Kubernetes 部署被称为 **集群**。为了运行容器并提供高度可用、可扩展的环境，集群由控制平面和一组关键资源组成，这些资源使它能够运行或管理容器、扩展它们，并在发生任何故障时保持容器的运行。在 Kubernetes 中运行容器时，容器被放置在 Pod 中，然后根据控制平面的决策在节点上运行。

**Pod** 为运行一组容器提供了一个共享的上下文。Pod 中的所有容器都在同一个节点上运行。虽然你可以在 Pod 中运行多个容器，但通常一个 Pod 将包含一个单一的应用容器，而 Pod 中运行的任何其他容器都将作为辅助容器，为应用容器提供一些管理或支持功能。

在传统的自动化操作环境中，自动化将描述如何设置环境。当使用 Kubernetes 时，不是描述如何设置环境，而是描述期望的最终状态，而控制平面将决定如何实现这一点。此配置以一个（或更常见的是一组）YAML 文档的形式提供。这种效果是，在部署到 Kubernetes 时，你不是通过在节点上定义 Pod 并将容器放入其中来描述部署。相反，你定义一个 **Deployment**，它将表达你想要部署的容器以及应该创建多少个容器副本。可以使用以下 YAML 部署 Open Liberty 的单个实例：

```java
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: demo
  name: demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
    spec:
      containers:
      - image: openliberty/open-liberty:full-java11-openj9-
  ubi
        name: open-liberty
```

然后，可以使用 `kubectl apply` 命令部署此 YAML。这会导致部署一个运行 Open Liberty 的单一 Pod。当容器运行并能够响应 HTTP 请求时，没有路由让网络流量到达容器。使网络流量能够到达部署的关键是 Kubernetes **服务**。服务定义了容器中进程监听的端口以及通过 Kubernetes 网络堆栈访问的端口。可以使用以下 YAML 定义此类服务：

```java
apiVersion: v1
kind: Service
metadata:
  labels:
    app: demo
  name: demo
spec:
  ports:
  - name: 9080-9080
    port: 9080
    protocol: TCP
    targetPort: 9080
  selector:
    app: demo
  type: ClusterIP
```

一个服务允许运行 Kubernetes 的其他容器访问它，但不允许集群外的服务访问它。有几种方法可以将容器外部暴露，例如端口转发、入口控制器，或者 OpenShift 有 **路由** 的概念。路由本质上只是将服务外部暴露给集群。您可以指定主机和路径，或者您可以允许 Kubernetes 默认设置。为了将此 Open Liberty 服务器外部暴露，您可以使用以下 YAML 定义一个路由：

```java
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: demo
  labels:
    app: demo
spec:
  to:
    kind: Service
    name: demo
    weight: 100
  port:
    targetPort: 9080-9080
```

这三个 YAML 文件已部署了一个容器并将其外部暴露，以便可以访问和使用。这三个 YAML 文件可以使用 `---` 作为分隔符放置在单个文件中，但除了使用 YAML 配置一切之外，还有另一种管理部署的选项，那就是使用 Operator。

**Operator** 是一种打包、部署和管理与应用程序相关的一组资源的方式。它们最初旨在帮助管理有状态的应用程序，如果只是丢弃并启动一个新的 Pod，可能会导致数据丢失；然而，它们也可以用来简化应用程序的部署。Operator 监视它理解的 **自定义资源** 的定义，并配置相关的 Kubernetes 资源来运行该应用程序。Operator 可以执行诸如管理应用程序的部署和更新，当有新镜像可用时。Open Liberty 提供了一个 Operator，可以管理基于 Open Liberty 的应用程序的部署。例如，所有之前的 YAML 文件都可以简单地替换为以下 YAML：

```java
apiVersion: openliberty.io/v1beta1
kind: OpenLibertyApplication
metadata:
  name: my-liberty-app
spec:
  applicationImage: openliberty/open-liberty:full-java11-
    openj9-ubi
  service:
    type: ClusterIP
    port: 9080
  expose: true
```

## Kubernetes 中的 MicroProfile Health

MicroProfile Health 规范允许您配置对存活性和就绪性检查的支持。这些探针允许 Kubernetes 控制平面了解容器的健康状况以及应采取什么行动。一个失败的存活性探针将触发 Pod 被回收，因为它表明发生了无法解决的问题。另一方面，就绪性探针将简单地导致 Kubernetes 停止将流量路由到 Pod。在这两种情况下，您需要多个实例以确保在任何容器故障期间，客户端仍然不会察觉。要配置这些存活性和就绪性探针，您需要确保 Open Liberty 服务器配置为运行 MicroProfile Health：

```java
<server>
    <featureManager>
        <feature>mpHealth-3.0</feature>
    </featureManager>
</server>
```

然后，在定义应用程序时，配置存活性和就绪性探针：

```java
apiVersion: openliberty.io/v1beta1
kind: OpenLibertyApplication
metadata:
  name: my-liberty-app
spec:
  expose: true
  applicationImage: openliberty/open-liberty:full-java11-    openj9-ubi
  readinessProbe:
    httpGet:
      path: /health/ready
      port: 9080
    initialDelaySeconds: 30
  livenessProbe:
    httpGet:
      path: /health/live
      port: 9080
    initialDelaySeconds: 90
  replicas: 1
```

此配置将存活性和就绪性探针配置为向 MicroProfile 健康端点发送 HTTP `get` 请求以检查存活性和就绪性。它还在容器启动后配置了一个等待期，在执行第一次检查之前。这给了容器一个机会在开始轮询以确定状态之前运行任何启动程序。

## Kubernetes 中的 MicroProfile Config

**MicroProfile Config** 提供了一种在您的应用程序中接收配置的方法，该配置可以在环境中提供。在 Kubernetes 中，此类配置通常存储在 **ConfigMap** 或 **Secret** 中。如前所述，在 *第五章*，“增强云原生应用程序”中，ConfigMap 实质上是存储在 Kubernetes 中的键/值对集合，可以绑定到 Pod 中，使其对容器可用。要从 Kubernetes 接收应用程序的配置，请确保 Open Liberty 服务器已配置为运行 MicroProfile Config：

```java
<server> 
  <featureManager> 
    <feature>mpConfig-2.0</feature> 
  </featureManager> 
</server>
```

创建 ConfigMap 有许多方法，*第五章*，“增强云原生应用程序”演示了一种机制。定义 ConfigMap 的另一种方法是应用以下 YAML 的 ConfigMap：

```java
kind: ConfigMap
apiVersion: v1
metadata:
  name: demo
data:
  example.property.1: hello
  example.property.2: world
```

现在，当部署您的应用程序时，您可以选择绑定此 ConfigMap 中的一个单个环境变量，或者全部绑定。要将 ConfigMap 中的 `example.property.1` 的值绑定到名为 `PROP_ONE` 的变量，并将其绑定到 Open Liberty 应用程序，您将使用以下 YAML：

```java
apiVersion: openliberty.io/v1beta1
kind: OpenLibertyApplication
metadata:
  name: demo-app
spec:
  expose: true
  applicationImage: openliberty/open-liberty:full-java11-    openj9-ubi
  env:
    - name: PROP_ONE
      valueFrom:
        configMapKeyRef:
          key: example.property.1
          name: demo
  replicas: 1
```

ConfigMap 可以（如上述示例所示）包含许多容器可能需要访问的属性，而不是绑定单个条目或逐个绑定条目，您可以绑定所有条目。以下 YAML 将定义一个将 ConfigMap 的所有值绑定的应用程序：

```java
apiVersion: openliberty.io/v1beta1
kind: OpenLibertyApplication
metadata:
  name: demo-app
spec:
  expose: true
  applicationImage: openliberty/open-liberty:full-java11-    openj9-ubi
  envFrom:
    - configMapRef:
        name: demo
  replicas: 1
```

MicroProfile Config 的最新功能之一是配置配置文件的概念。想法是您可以为开发、测试和生产环境中的应用程序提供配置，并且 MicroProfile Config 只加载所需配置文件的配置。为此，您还需要定义配置配置文件。MicroProfile Config 规范说明配置文件名称中的属性以 `%<profile name>` 开头；然而，`%` 在环境变量名称中无效，因此它被替换为 `_`。以下 YAML 是此示例的示例：

```java
apiVersion: openliberty.io/v1beta1
kind: OpenLibertyApplication
metadata:
  name: demo-app
spec:
  expose: true
  applicationImage: openliberty/open-liberty:full-java11-    openj9-ubi
  env:
    - name: mp.config.profile
      value: dev
  envFrom:
    - configMapRef:
        name: dev
      prefix: '_dev.'
    - configMapRef:
        name: test
      prefix: '_test.'
    - configMapRef:
        name: prod
      prefix: '_test.'
  replicas: 1
```

Kubernetes 中的 ConfigMaps 适用于存储非敏感数据，但当涉及到存储 API 密钥、凭证等内容时，Kubernetes 提供了一种称为 Secrets 的替代概念。Secrets 可以代表多种不同类型的 Secrets，但在这里我们只考虑简单的键/值对。Kubernetes 平台为 Secrets 提供了比 ConfigMaps 更好的保护，尽管许多人更喜欢使用第三方产品进行 Secret 管理。了解 Secrets 的工作原理仍然很有用，因为第三方产品通常遵循相同的约定来从容器内部访问敏感数据。

秘密使用 base64 编码，这并不是非常好的保护。Open Liberty 允许加载的密码使用 AES 加密，并提供了解密受保护字符串的 API，因此你的 base64 编码的秘密可能是一个 base64 编码的 AES 加密字符串。然而，由于你仍然需要向 Open Liberty 提供解密密钥，而这不是一本关于安全加固的书，所以我们不会在这里进一步详细介绍。从部署中引用秘密中的单个密钥对几乎与从 ConfigMap 中引用相同，但使用 `secretKeyRef` 而不是 `configMapKeyRef`；例如，以下 YAML：

```java
apiVersion: openliberty.io/v1beta1
kind: OpenLibertyApplication
metadata:
  name: demo-app
spec:
  expose: true
  applicationImage: openliberty/open-liberty:full-java11-    openj9-ubi
  env:
    - name: PROP_ONE
      valueFrom:
        secretKeyRef:
          key: my_secret
          name: secret.config
  replicas: 1
```

如果你部署了秘密 YAML 并按照前面提到的示例进行绑定，你的容器将有一个名为 `PROP_ONE` 的环境变量，其值为 `super secret`。

就像 ConfigMap 一样，你可以将秘密中的所有键/值对绑定到容器中，并且就像前面的例子一样，它是以与 ConfigMaps 非常相似的方式进行操作的：

```java
apiVersion: openliberty.io/v1beta1
kind: OpenLibertyApplication
metadata:
  name: demo-app
spec:
  expose: true
  applicationImage: openliberty/open-liberty:full-java11-    openj9-ubi
  envFrom:
    - secretRef:
        name: secret.config
  replicas: 1
```

秘密也可以绑定到容器文件系统中的文件，这对于需要高度安全的数据来说可能更可取。当你这样做时，秘密将在文件系统中定义，秘密的值将是文件的内容。MicroProfile Config 不能消费以这种方式绑定的秘密，但它提供了一种添加额外 ConfigSources 的方法，让你可以轻松地加载配置。将秘密绑定到文件系统的 YAML 实际上就是将其挂载为一个卷。以下示例 YAML 将导致秘密 `secret.config` 中的每个键/值在容器的 `/my/secret` 目录下作为文件挂载：

```java
apiVersion: openliberty.io/v1beta1
kind: OpenLibertyApplication
metadata:
  name: demo-app
spec:
  expose: true
  applicationImage: openliberty/open-liberty:full-java11-    openj9-ubi
  volumeMounts:
    - mountPath: /my/secrets
      name: secret
  volumes:
    - secret:
        secretName: secret.config
      name: secret
  replicas: 1
```

要启用注入绑定的秘密，你需要在应用程序类路径上有一个 `META-INF/services/org.eclipse.microprofile.config.spi.ConfigSource`，这将自动导致它被加载。以下是一个简短的示例 ConfigSource，它将执行此操作：

```java
public class FileSystemConfigSource implements ConfigSource {
    private File dir = new File("/my/secrets");
    public Set<String> getPropertyNames() {
        return Arrays.asList(dir.listFiles())
                     .stream()
                     .map(f -> f.getName())
                     .collect(Collectors.toSet());
    }
    public String getValue(String s) {
        File f = new File(dir, s);
        try {
            if (f.exists())
                Path p = f.toPath();
                byte[] secret = Files.readAllBytes(f);
                return new String(secret,                                   StandardCharsets.UTF_8);
        } catch (IOException ioe) {
        }
        return null;
    }
    public String getName() {
        return "kube.secret";
    }
    public int getOrdinal() {
        return 5;
    }
}
```

此配置源将加载具有属性名称的文件内容，该属性名称是从一个定义良好的目录中读取的。如果文件无法读取，它将表现得好像该属性未定义。当秘密更新时，Kubernetes 将更新文件内容，这意味着更新可以自动对应用程序可见，因为每次读取属性时，此代码都会重新读取文件。

在下一节中，我们将讨论在使用 MicroProfile 与服务网格时的一些考虑因素。

# MicroProfile 和服务网格

当部署到 Kubernetes 集群时，有些人选择使用服务网格。服务网格的目标是将微服务的某些考虑因素从应用程序代码中移出，并将其放置在应用程序周围。服务网格可以消除一些应用程序关注点，例如服务选择、可观察性、故障容忍，以及在某种程度上，安全性。一种常见的服务网格技术是 **Istio**。Istio 的工作方式是在容器的 Pod 中插入一个边车，所有传入和网络流量都通过该边车路由。这允许边车执行应用访问控制策略、将请求路由到下游服务以及应用故障容忍策略，例如重试请求或超时。其中一些功能与 MicroProfile 的一些功能重叠，例如，Istio 可以处理将 OpenTracing 数据附加到请求并传播。如果你使用 **Istio**，显然不需要使用 MicroProfile OpenTracing，尽管同时使用两者会相辅相成而不是产生冲突。

在使用服务网格和 MicroProfile 可能产生负面冲突的一个领域是故障容忍。例如，如果你在 MicroProfile 中配置了 5 次重试，在 Istio 中也配置了 5 次重试，并且它们都失败了，你最终会有 25 次重试。因此，当使用服务网格时，通常禁用 MicroProfile 的故障容忍功能。这可以通过将环境变量 `MP_Fault_Tolerance_NonFallback_Enabled` 设置为 `false` 来完成。这将禁用所有 MicroProfile 故障容忍支持，除了回退功能。这是因为执行失败时的逻辑本质上是一个应用程序考虑因素，而不是可以提取到服务网格中的东西。这可以通过以下 YAML 简单禁用：

```java
apiVersion: openliberty.io/v1beta1
kind: OpenLibertyApplication
metadata:
  name: demo-app
spec:
  expose: true
  applicationImage: openliberty/open-liberty:full-java11-    openj9-ubi
  env:
    - name: MP_Fault_Tolerance_NonFallback_Enabled
      value: 'false'
  replicas: 1 
```

这将配置应用程序具有一个硬编码的环境变量，该变量禁用了非回退的 MicroProfile 故障容忍行为。这也可以通过 ConfigMap 来完成。

# 摘要

在本章中，我们回顾了本书其余部分使用的 MicroProfile 实现，构建 MicroProfile 应用程序容器的最佳实践，以及如何将应用程序部署到 Kubernetes。虽然本章并不是对 Kubernetes 中可用功能的详尽审查，但它确实关注了将 MicroProfile 应用程序部署到 Kubernetes 的特定考虑因素以及它们如何与 Kubernetes 服务交互。本章应该为您提供了创建和部署使用 Open Liberty、容器和 Kubernetes 的 MicroProfile 应用程序的良好起点。

下一章将描述一个示例应用程序，该应用程序利用 MicroProfile 在一个容器中部署到 Kubernetes 集群中的一组微服务。
