# 第八章：MicroProfile 实现、Quarkus 以及通过会议应用程序实现互操作性

Eclipse MicroProfile 的好处之一是它提供了一个规范，使得许多实现之间可以相互操作。这个好处激励了许多供应商和社区组织将 Eclipse MicroProfile 规范作为开源实现。目前市场上共有八个 Eclipse MicroProfile 实现，第九个实现者是 Quarkus。

本章将涵盖以下主题：

+   对 Eclipse MicroProfile 的八个实现以及如何找到每个实现的进一步信息的描述

+   如何为这些实现中的每一个生成 Eclipse MicroProfile 示例代码...

# 当前 MicroProfile 实现

截至编写本书时，共有八个 Eclipse MicroProfile 实现，所有这些都是开源的。以下是这些实现的表格：

| **开源项目基础** | **项目位置** | **支持供应商** |
| --- | --- | --- |
| Thorntail ([`thorntail.io/`](http://thorntail.io/)) | [`github.com/thorntail/thorntail`](https://github.com/thorntail/thorntail) | Red Hat |
| Open Liberty ([`openliberty.io/`](https://openliberty.io/)) | [`github.com/openliberty`](https://github.com/openliberty) | IBM |
| Apache TomEE ([`tomee.apache.org/`](http://tomee.apache.org/)) | [`github.com/apache/tomee`](https://github.com/apache/tomee) | Tomitribe |
| Payara Micro ([`www.payara.fish/payara_micro`](https://www.payara.fish/payara_micro)) | [`github.com/payara/Payara`](https://github.com/payara/Payara) | Payara Services Ltd. |
| Hammock ([`hammock-project.github.io/`](https://hammock-project.github.io/)) | [`github.com/hammock-project`](https://github.com/hammock-project) | Hammock |
| KumuluzEE ([`ee.kumuluz.com/`](https://ee.kumuluz.com/)) | [`github.com/kumuluz`](https://github.com/kumuluz) | KumuluzEE |
| 启动器 ([`github.com/fujitsu/launcher`](https://github.com/fujitsu/launcher)) | [`github.com/fujitsu/launcher`](https://github.com/fujitsu/launcher) | Fujitsu |
| Helidon ([`helidon.io/#`](https://helidon.io/#)) | [`github.com/oracle/helidon`](https://github.com/oracle/helidon) | Oracle |

这些实现中有一些是基于*应用程序服务器*的，如 Payara 和 Open Liberty，而其他则是基于*应用程序组装器*，只包括应用程序需要的功能，而不是要求运行一个应用程序服务器，并且通常生成可执行 JAR。然而，基于应用程序服务器的实现也具备生成可执行 JAR 的能力。

应用程序组装器可以生成一个*uberjar*，一个自包含的可运行 JAR 文件，或者一个将其运行时依赖位于子目录中的*应用程序 jar*，例如，伴随的`lib`或`libs`子目录。

符合 Eclipse MicroProfile 标准的实现通过整个伞状发布版本的**测试兼容性套件**（**TCK**），或者特定版本的 MicroProfile API 的实现，列表可在[`wiki.eclipse.org/MicroProfile/Implementation`](https://wiki.eclipse.org/MicroProfile/Implementation)找到。目前，这个列表采用荣誉制度，因为它不需要证明 TCK 的结果；它只需要发布者声明他们的实现已经通过了 TCK。

该项目还有一个网站，组织/团体可以自行加入 MicroProfile 生产部署列表。这个列表可以在[`wiki.eclipse.org/MicroProfile/Adoptions`](https://wiki.eclipse.org/MicroProfile/Adoptions)找到。

在下一节中，我们提供了这些实现简要介绍以及如何获取关于它们的更多信息。

# Thorntail

红帽公司是开源 Thorntail 项目的赞助商，该项目实现了 Eclipse MicroProfile 规范。Thorntail 是一个应用程序组装器，它只包含您的应用程序所需的服务器运行时组件，并创建一个可执行的 JAR（即 uberjar），您可以通过调用以下命令来执行：

```java
$ java -jar <executable JAR file>
```

不仅 Thorntail 符合 MicroProfile，它还可以在您的应用程序中包含超出 MicroProfile 的功能。它有一个分数的概念，这是一个包含您想要包含在应用程序中的功能的特定库。分数作为您应用程序的 Maven POM 文件的一个依赖项。超出 MicroProfile ...

# Open Liberty

IBM 是开源 Open Liberty 项目的赞助商，该项目实现了 Eclipse MicroProfile 规范。Open Liberty 是 IBM WebSphere Liberty 应用服务器的上游开源项目。Open Liberty 是一个能够生成 uberjar 的应用服务器，其中包含您的应用程序以及内嵌的 Open Liberty 服务器。要运行 uberjar，您需要输入以下命令：

```java
$ java -jar <executable JAR file>
```

此命令将把 JAR 文件解压到您的用户名临时目录中，然后从那里执行应用程序。

确保 JAR 文件路径中没有空格，否则启动过程将失败。

生成的 uberjar 只能包含`server.xml`文件中包含的功能的子集。要使用这些最小功能集构建 uberjar，您需要在运行 Maven 时使用`minify-runnable-package`配置文件。

Open Liberty 文档非常全面，充满了指南和参考文献。

您可以在[`openliberty.io/docs/`](https://openliberty.io/docs/)找到 Open Liberty 文档。

在他们的文档中，他们有一个专门介绍 MicroProfile 指南的部分，提供了文档齐全的教程。

# Apache TomEE

托米部落（Tomitribe）是开源 TomEE 项目的赞助商，该项目实现了 Eclipse MicroProfile 规范。Apache TomEE 是由 Apache Tomcat 组装而成，增加了 Java EE 特性。TomEE 是 Java EE 6 Web Profile 认证的。正如其 GitHub 所描述的，*Apache TomEE 是一个轻量级但功能强大的 Java EE 应用服务器，拥有丰富的功能工具*。您可以下载几个不同版本的 TomEE，例如 TomEE、TomEE+、TomEE WebApp，但我们感兴趣的是 TomEE MicroProfile。对于 MicroProfile，TomEE 为您生成了一个 uberjar，您可以像以下这样运行：

```java
$ java -jar <executable JAR file>
```

尽管 TomEE MicroProfile 文档不多，但有一套详尽的...

# 帕雅拉（Payara）

帕雅拉（Payara）是开源 Payara Micro 项目的赞助商，该项目实现了 Eclipse MicroProfile 规范。Payara 服务器基于开源应用服务器 GlassFish。Payara Micro 是基于 Payara Server 的一个精简版本。正如他们的网站所描述的，*Payara Micro 是 Payara Server 的适用于微服务的版本*。

Payara Micro 的工作方式是 Payara Micro 实例启动，然后将 MicroProfile 微服务作为 WAR 文件部署到其中。例如，要启动一个 Payara Micro 实例，您将输入以下命令：

```java
$ java -jar payara-micro.jar
```

要启动 Payara Micro 实例并将您的应用程序部署到其中，您将输入以下命令：

```java
$ java -jar payara-micro.jar --deploy <WAR file>
```

Payara Micro 支持 Java EE 应用程序部署，并且与 Eclipse MicroProfile 兼容。

对于 Payara Micro 文档，请参考[`docs.payara.fish/documentation/payara-micro/payara-micro.html`](https://docs.payara.fish/documentation/payara-micro/payara-micro.html)。

最后，Payara Micro 通过使用第三方内存内数据网格产品支持自动集群。

# 吊床

约翰·阿门特（John Ament）是开源 Hammock 项目的赞助商，该项目实现了 Eclipse MicroProfile 规范。与 Thorntail 相似，Hammock 是一个应用程序组装器，生成 uberjars。要运行 uberjar，您需要输入以下命令：

```java
$ java -jar <executable JAR file>
```

吊床是一个有观点的微服务框架，用于构建应用程序。它是一个基于 CDI 的框架，意味着它是基于 CDI 容器的，CDI 基于的 bean 在其中运行。它支持两种 CDI 实现（JBoss Weld 和 Apache OpenWebBeans），三种 JAX-RS 实现（Apache CXF、Jersey 和 JBoss RestEasy），以及三种不同的 servlet 容器（Apache Tomcat、JBoss Undertow 和 Eclipse Jetty）。除此之外，Hammock 还...

# 库穆鲁兹（KumuluzEE）

Sunesis 是开源 KumuluzEE 项目的赞助商，该项目实现了 Eclipse MicroProfile 规范。KumuluzEE 定义了自己作为一个使用 Java 和 Java EE 技术的轻量级微服务框架，并且是 Eclipse MicroProfile 兼容的实现。KumuluzEE 允许你使用仅需要的组件来引导一个 Java EE 应用程序，并且还支持将微服务打包和作为 uberjars 运行。与其他支持 uberjars 的实现一样，你可以通过输入以下命令来运行你的微服务：

```java
$ java -jar <executable JAR file>
```

KumuluzEE 还提供了一个 POM 生成器，它可以创建一个带有所选选项和功能的 `pom.xml`，用于你计划开发的微服务。POM 生成器提供了由 KumuluzEE 支持的可选的清晰和组织的列表，包括在 `pom.xml` 文件中。

KumuluzEE 为不同的 MicroProfile API 提供了一些示例。

有关 KumuluzEE 实现 Eclipse MicroProfile 的文档，请参考 [`ee.kumuluz.com/microprofile`](https://ee.kumuluz.com/microprofile)。

最后，KumuluzEE 提供了一些有趣的教程在 [`ee.kumuluz.com/tutorials/`](https://ee.kumuluz.com/tutorials/)。

# 启动器

Fujitsu 是开源 Launcher 项目的赞助商，该项目实现了 Eclipse MicroProfile 规范。Launcher 利用了内嵌的 GlassFish 服务器和 Apache Geronimo MicroProfile API 实现。你可以将你的微服务作为 WAR 文件运行，如下所示：

```java
$ java -jar launcher-1.0.jar --deploy my-app.war
```

此外，Launcher 可以创建 uberjars。要创建并运行你的微服务作为 uberjar，首先生成 uberjar，然后使用 `java -jar` 调用它，如下所示：

```java
$ java -jar launcher-1.0.jar --deploy my-app.war --generate my-uber.jar$ java -jar my-uber.jar
```

有关 Launcher 的文档非常稀少且有限。你可以找到有关 Launcher 的使用信息在 [`github.com/fujitsu/launcher/blob/master/doc/Usage.adoc ...`](https://github.com/fujitsu/launcher/blob/master/doc/Usage.adoc)。

# Helidon

Oracle Corporation 是开源 Helidon 项目的赞助商，该项目实现了 Eclipse MicroProfile 规范。Helidon 是一组 Java 库，可让开发者编写微服务。它利用了 Netty，一个非阻塞的 I/O 客户端服务器框架。Helidon 是一个应用程序组装器，因为它可以生成应用程序 JAR。一旦你构建了应用程序 JAR，你可以使用以下命令执行它：

```java
$ java -jar <executable JAR file>
```

Helidon 有两种版本：SE 和 MP。Helidon SE 是由所有 Helidon 库提供的功能编程风格，它提供了一个名为 MicroFramework 的微服务框架。Helidon MP 实现了微服务的 MicroProfile 规范，并建立在 Helidon 库之上。没有样本项目生成工具，但 Helidon 提供了一组丰富且详尽的文档手册。

Helidon 的文档可以在 [`helidon.io/docs/latest/#/about/01_overview`](https://helidon.io/docs/latest/#/about/01_overview) 找到。

Helidon SE 提供了一个 WebServer，这是一个用于创建 Web 应用程序的异步和反应式 API。Helidon MP 提供了一个封装 Helidon WebServer 的 MicroProfile 服务器实现。

# 为当前实现生成示例代码

如前几节所述，大多数 MicroProfile 实现并没有提供自己的示例项目生成器。相反，它们只提供文档。这时 MicroProfile Starter 就派上用场了！

MicroProfile Starter 由 MicroProfile 社区赞助，是一个为所有通过 MicroProfile TCK 的 MicroProfile 规范生成示例项目和源代码的工具。在第二章*治理和贡献*中，我们为您提供了 MicroProfile Starter 的概览。为了避免重复，我们只想指出您可以在下拉菜单中选择 MicroProfile 版本如下：...

# 其他实现 MicroProfile 的项目

小型 Rye 是一个开源项目，它开发了任何供应商或项目都可以使用的 Eclipse MicroProfile 实现。这是一个社区努力，每个人都可以参与和贡献给小型 Rye,[`smallrye.io`](https://smallrye.io)。作为一个例子，社区最近将微服务扩展项目贡献给了小型 Rye，从而使其通过配置源、OpenAPI、健康、JAX-RS 和 REST 客户端扩展丰富了其功能。

微服务扩展项目网站是[`www.microprofile-ext.org`](https://www.microprofile-ext.org)，其 GitHub 是[`github.com/microprofile-extensions`](https://github.com/microprofile-extensions)。

小型 Rye 实现已经通过了 Eclipse MicroProfile TCKs 的测试。

消费小型 Rye 的开源项目有 Thorntail([`thorntail.io`](https://thorntail.io))、WildFly([`wildfly.org`](https://wildfly.org))和 Quarkus([`quarkus.io`](https://quarkus.io))。

# Quarkus

开源的 Quarkus 项目于 2019 年首次亮相。Quarkus 是一个可以编译成原生机器语言或构建到 HotSpot（OpenJDK）的 Kubernetes 本地 Java 栈。使用 Quarkus 时，您的应用程序消耗非常少的内存，具有出色的性能，可以处理高调用吞吐量，并且启动时间非常快（即引导加上首次响应时间），使 Quarkus 成为容器、云本地和无服务器部署的绝佳运行时。Quarkus 还提供了一个扩展框架，允许将库和项目*quarking*（注：此处应为“转化为 Quarkus 兼容的形式”），使它们与 Quarkus 无缝协作。

Quarkus 的使命是将您的整个应用程序及其使用的库转换为最优...

# 如何将生成的 MicroProfile 项目*quarking*

在我们开始讲解如何使用 MicroProfile Starter*quark*生成 MicroProfile 项目的步骤之前，我们首先需要确保已经在您的环境中安装、定义和配置了 GRAALVM_HOME。为此，请按照以下步骤操作：

1.  访问`https://github.com/oracle/graal/releases`，并根据您的操作系统下载 GraalVM 的最新版本。

1.  将下载的文件解压缩到您选择的子目录中。顺便说一下，解压缩将创建一个 GraalVM 子目录，例如：

```java
$ cd $HOME
$ tar -xzf graalvm-ce-1.0.0-rc16-macos-amd64.tar.gz
```

1.  打开一个终端窗口，创建一个名为`GRAALVM_HOME`的环境变量，例如：

```java
$ export GRAALVM_HOME=/Users/[YOUR HOME DIRECTORY]/graalvm-ce-1.0.0-rc13/Contents/Home
```

既然我们已经安装了 GraalVM，我们可以继续讲解如何使用 MicroProfile Starter*quark*生成 MicroProfile 项目的步骤：

1.  首先，将您的浏览器指向[`start.microprofile.io`](https://start.microprofile.io)并选择 Thorntail 作为 MicroProfile 服务器。

您可以利用以下步骤将任何现有的 Java 应用程序*quark*化。

如果您不记得如何进行此操作，请转到第二章，*治理和贡献*，并遵循*MicroProfile Starter 快速入门*部分中的说明，直到第 5 步，其中`demo.zip`文件下载到您的本地`Downloads`目录。

1.  使用您喜欢的解压缩工具展开`demo.zip`文件。如果您没有自动展开`demo.zip`文件，请使用以下命令（假设是 Linux；对于 Windows，请使用等效命令）：

```java
$ cd $HOME/Downloads
$ unzip demo.zip
```

这将创建一个名为`demo`的子目录，在其下有一个完整的目录树结构，包含所有使用 Maven 构建和运行 Thorntail 示例 MicroProfile 项目的源文件。

1.  与其在`demo`子目录中进行更改，不如让我们创建一个名为`Qproj4MP`的第二个目录，与`demo`子目录并列，如下所示：

```java
$ mkdir $HOME/Downloads/Qproj4MP
```

这将在您`Downloads`目录中现有`demo`子目录的同级创建一个名为`Qproj4MP`的子目录。

1.  将您的目录更改到`Qproj4MP`，并通过输入以下命令创建一个空的 Quarkus 项目：

```java
$ cd $HOME/Downloads/Qproj4MP
$ mvn io.quarkus:quarkus-maven-plugin:0.12.0:create \
 -DprojectGroupId=com.example \
 -DprojectArtifactId=demo \
 -Dextensions="smallrye-health, smallrye-metrics, smallrye-openapi, smallrye-fault-tolerance, smallrye-jwt, resteasy, resteasy-jsonb, arc"
```

1.  在`Qproj4MP`目录中，删除`src`子目录并用以下命令替换为 Thorntail 示例 MicroProfile 项目的`src`子目录：

```java
$ cd $HOME/Downloads/Qproj4MP  # ensuring you are in the Qproj4MP sub-directory
$ rm -rf ./src
$ cp -pR $HOME/Downloads/demo/src .
```

1.  Quarkus 和 Thorntail 对某些配置和 web 应用程序相关文件的位置有不同的期望。因此，为了使 Quarkus 满意，让我们通过输入以下命令来复制一些文件：

```java
$ cd $HOME/Downloads/Qproj4MP # ensuring you are in the Qproj4MP sub-directory
$ mkdir src/main/resources/META-INF/resources
$ cp /Users/csaavedr/Downloads/demo/src/main/webapp/index.html src/main/resources/META-INF/resources
$ cp -p src/main/resources/META-INF/microprofile-config.properties src/main/resources/application.properties
```

我们本可以将这些文件从它们原来的位置移动，但我们选择在这个示例中只是复制它们。

1.  MicroProfile Starter 生成的 Thorntail 示例 MicroProfile 项目，其`src`子目录的内容你已经复制到了`Qproj4MP`，使用了一个名为`bouncycastle`的安全库。这是因为生成的代码包含了一个 MicroProfile JWT Propagation 规范的示例，该规范允许你在微服务之间传播安全性。因此，我们还需要在 Quarkus 项目的 POM 文件中再添加两个依赖，一个是`bouncycastle`，另一个是`nimbusds`。

下一个 sprint 版本的 MicroProfile Starter 将不再包含 Thorntail 服务器代码生成中的`bouncycastle`依赖。

为了添加这些依赖项，请编辑你`$HOME/Downloads/Qproj4MP`目录下的`pom.xml`文件，并在`<dependencies>`部分输入以下代码块：

```java
 <dependency>
 <groupId>org.bouncycastle</groupId>
 <artifactId>bcpkix-jdk15on</artifactId>
 <version>1.53</version>
 <scope>test</scope>
 </dependency>
 <dependency>
 <groupId>com.nimbusds</groupId>
 <artifactId>nimbus-jose-jwt</artifactId>
 <version>6.7</version>
 <scope>test</scope>
 </dependency>
```

现在我们准备编译 quarked 的 MicroProfile 项目。

1.  除了支持构建可以在 OpenJDK 上运行的 Java 项目外，Quarkus 还支持将 Java 项目编译到底层机器码。输入以下命令来编译 quarked 示例项目到原生代码：

```java
$ cd $HOME/Downloads/Qproj4MP # ensuring you are in the Qproj4MP sub-directory
$ ./mvnw package -Pnative
```

1.  要运行应用程序，请输入以下命令：

```java
$./target/demo-1.0-SNAPSHOT-runner
```

要测试应用程序，请遵循*Quick tour of MicroProfile Starter*章节中*治理和贡献*部分的说明，从第 10 步开始。

1.  如果你想要在开发模式下运行 quarked 项目，首先停止正在运行的进程，然后输入以下命令：

```java
$ cd $HOME/Downloads/Qproj4MP # ensuring you are in the Qproj4MP sub-directory
$ ./mvnw compile quarkus:dev
```

在此阶段，你可以选择一个 IDE，比如 Visual Studio Code 或 Eclipse IDE，来打开项目，并开始修改源代码。Quarkus 支持热重载，这意味着，只要你对源代码做了任何更改，Quarkus 会在后台重新构建并重新部署你的应用程序，这样你就可以立即看到并测试更改的效果。此外，如果你在源代码中犯了语法错误，Quarkus 会将有意义的错误信息传播到网页应用程序中，帮助你修复错误，提高你的工作效率。

1.  如果你想要生成一个可执行的应用程序 JAR，请输入以下命令：

```java
$ cd $HOME/Downloads/Qproj4MP # ensuring you are in the Qproj4MP sub-directory
$ ./mvn clean package
```

1.  要运行可执行的应用程序 JAR，请输入以下命令：

```java
$ java -jar target/demo-1.0-SNAPSHOT-runner.jar
```

创建一个与应用程序 JAR 并列的 lib 目录，其中包含运行所需的所有库文件。

我们向您展示了使用 MicroProfile Starter 生成的 MicroProfile 项目的*quark*步骤。尽管这些步骤适用于特定的生成项目，但您可以使用相同的说明来*quark*一个现有的 Java 应用程序或微服务，以便您可以利用 Quarkus 提供的好处，如低内存消耗、快速的启动时间以及对 Java 代码的原生编译，以便您可以在容器、云和函数即服务环境中高效运行。无论您使用 MicroProfile 的哪个实现，MicroProfile 为最终用户提供的很大好处就是互操作性。这意味着您可以设计一个使用不同 MicroProfile 实现的微服务应用程序，这是下一节的主题。

# MicroProfile 互操作性——会议应用程序

**会议应用程序**，首次介绍（[`www.youtube.com/watch?v=iG-XvoIfKtg`](https://www.youtube.com/watch?v=iG-XvoIfKtg)）于 2016 年 11 月在比利时 Devoxx 上，是一个展示不同 MicroProfile 供应商实现集成和互操作性的 MicroProfile 演示。这很重要，因为它展示了规范的实现和接口之间的分离，提供了一个允许供应商开发并提供自己的实现的平台，这些实现可以与其他竞争性实现共存。所有实现中的通用接口还为最终用户提供了使用任何 MicroProfile 实现...的好处。

# 总结

在本章中，我们了解了市场上现有的开源 MicroProfile 实现，它们是什么类型的实现，如何获取关于它们的更多信息，以及如何使用 MicroProfile Starter 为这些实现生成示例代码。我们还介绍了最新的 MicroProfile 实现参与者 Quarkus，它显著提高了 Java 在解释和编译模式下的启动时间和内存消耗，进一步优化了适用于云原生微服务和无服务器环境的 MicroProfile。您还了解了 The Conference Application，它展示了 MicroProfile 在不同实现之间的互操作性。

作为 Eclipse MicroProfile 的消费者，其跨实现互操作的特性，您有自由选择对您的组织最有意义或最适合您环境的实现，最终给您提供选择正确工具的正确任务的选项。此外，您不必局限于单一供应商的商业支持版 Eclipse MicroProfile，因此，您可以根据自己的条件进行谈判，并从不同供应商提供的丰富的 MicroProfile 特性中进行选择。

在下一章，我们将涵盖整个 MicroProfile API 集的全代码示例。

# 问题

1.  目前市场上存在多少种 MicroProfile 实现？请列出它们。

1.  应用服务器与应用组装器之间有什么区别？

1.  描述市场上存在的八种 MicroProfile 实现。

1.  什么是 Quarkus？

1.  编译时启动是什么？

1.  Quarkus 适用于哪种类型的部署？

1.  什么是 Quarkus 扩展框架？

1.  会议应用程序展示了什么关键优势？
