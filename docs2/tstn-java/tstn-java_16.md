# 16

# 在独立包和容器中部署 Java

在最后一章中，我们将探讨不同的打包和分发 Java 应用程序的方法。我们已经看到了桌面应用程序的 JAR 文件和网络应用程序的 WAR 文件，以及如何部署它们。虽然这种方法对于部署来说是足够的，但在某些情况下，这种传统方法可以改进。

Java 很大。Java SE 分发中包含许多库，尽管你的应用程序可能只需要其中的一些。第三方或外部库也是如此。现代使用 Java 模块方法的打包允许你生成只包含你将使用的库部分的 JAR 或 WAR 文件。

在网络应用程序的情况下，这种打包方式可以将 WAR 文件的大小减少到仅包含所需外部库中的所需模块，而不是整个库。在桌面应用程序的情况下，要求计算机上已经安装了 Java 语言。Java 运行时现在已经模块化。这允许你创建不需要安装 Java 版本即可运行的可执行应用程序，而是将其作为打包安装程序的一部分。

最后，我们将探讨 Docker 容器系统。想象一个由不同 **操作系统**（**OSes**）的开发商组成的团队，他们正在开发一个应用程序。虽然 Java 是 *一次编写，到处运行*，但有时让每个开发商在相同的环境中工作是有利的。Docker 容器有助于满足这一需求。此外，你可以将这些容器部署到云端。虽然我们不会探讨云部署，但了解容器的工作原理将使你为在云中工作做好准备。

本章我们将探讨以下内容：

+   探索模块化 Java 是什么

+   使用 `jlink` 创建自定义 JRE

+   使用 `jpackage` 进行打包安装

+   使用 Docker 容器系统

+   与 Docker 镜像一起工作

+   创建 Docker 镜像

+   发布镜像

我们将使用来自 *第十三章*，*使用 Swing 和 JavaFX 进行桌面图形用户界面编码*，最初名为 `BankSwing`，在本章中重命名为 `BankSwingJUL` 的修改版本来探索模块和包。为了查看 Docker，我们将使用上一章未更改的 `JSF_FinancialCalculator`。

# 技术要求

你需要以下内容来完成本章：

+   Java 17。

+   文本编辑器。

+   Maven 3.8.6 或更高版本。

+   对于 `jpackage`:

    +   `rpm-build` 软件包

    +   `fakeroot` 软件包

    +   **macOS**: Xcode 命令行工具

+   Docker Desktop [(https://www.docker.com](https://www.docker.com/)/)。要使用 Docker，你需要创建一个账户。免费的个人账户就足够了。一旦你有了账户，你就可以下载 Docker Desktop。每个操作系统都有一个版本。

本章的示例代码可在[`github.com/PacktPublishing/Transitioning-to-Java/tree/chapter16`](https://github.com/PacktPublishing/Transitioning-to-Java/tree/chapter16)找到。

# 探索模块化 Java 是什么

到目前为止，我们看到了由类字段和方法组成的类中的代码。然后，我们将这些类分组到包中，最后作为一个 JAR 或 WAR 文件。模块化 Java 引入了一个新的分组，称为**模块**。模块是一个 JAR 文件，但包含模块描述符。还有一个自动模块，其清单文件中有模块名称。这个 Java 特性被称为**Java 平台模块系统**（**JPMS**）。

到目前为止，我们使用 Maven 构建我们的应用程序。Maven 是一个构建工具，它会下载我们需要的任何库，并确保我们的代码可以成功编译。它不做的是确定所有必需的外部库，例如 Java 本身，是否都存在。它的主要工作在代码成功编译后结束。另一方面，JPMS 专注于成功运行程序所需的库。与 Maven 不同，JPMS 检查作为模块编写的库是否存在或将在代码运行时存在。这引出了一个问题，什么是模块？

模块是一个 JAR 文件。普通 JAR 文件和模块 JAR 文件之间有一些细微的差别。至少，模块文件必须在`src/main/java`文件夹中包含一个名为`module-info.java`的文件。这个文件的一个目的是列出所需的模块。可能没有所需的模块，但这个文件的存在表示该项目可以是一个模块。并非所有用 Java 编写的库都已被重新编码为模块，但许多新的库都是这样编写的。模块文件可以用作普通 JAR 文件，也可以在使用 JPMS 工具时用作模块。你不需要两个版本的库。

曾经，用户有 Java 的两个版本可用。有一个包含 JVM 和所有必需的开发工具，如 Java 编译器的 JDK。第二个版本是**Java 运行时版**（**JRE**）。正如其名称所暗示的，JRE 包含运行几乎任何 Java 程序所需的所有库。JRE 的大小显著减小，大约为 90 MB，而完整的 JDK 大约为 300 MB。随着 JPMS 的引入，JRE 不再作为下载提供。时代在变化，现在一些 Java 发行版又包含了 JRE。

由于 JRE 比 JDK 小得多，模块能为我们做什么呢？原因与为什么 JRE 从 Java 发行版中删除有关。使用 JPMS，你可以构建自己的自定义 JRE，只包含你需要的模块。那么，Java 语言中的模块是什么呢？在终端/控制台窗口中，输入以下内容：

```java
java --list-modules
```

现在，你将获得每个模块的列表。在我的 Windows 11 系统上，有 71 个模块被列出——22 个以`java`开头，49 个以`jdk`开头。为了构建自定义 JRE，你需要知道你的程序使用了哪些模块。从本章的 GitHub 获取`BankSwingJUL`项目。与*第十三章*版本的不同之处在于`JUL`替换了`log4j2`。我这样做是为了将所需的模块数量减少到 Java 发行版中的那些。构建项目，你应该在`target`文件夹中找到一个名为`BankSwingJUL-0.1-SNAPSHOT.jar`的 JAR 文件。在`target`文件夹中打开一个终端/控制台窗口，并输入以下命令：

```java
jdeps BankSwingJUL-0.1-SNAPSHOT.jar
```

输出将开始于你需要用到的 Java 模块的摘要：

```java
BankSwingJUL-0.1-SNAPSHOT.jar -> java.base
BankSwingJUL-0.1-SNAPSHOT.jar -> java.desktop
BankSwingJUL-0.1-SNAPSHOT.jar -> java.logging
```

输出的剩余部分将查看项目中的每个类，显示你正在使用的 Java 类以及它们所属的模块。`java.base`模块是核心类集的家园。`java.desktop`模块是 Swing 的家园，而`java.logging`模块是 JUL 的家园。现在，是时候创建我们的自定义 JRE 了。

# 使用 jlink 创建自定义 JRE

我们将使用 Java JDK 的一部分`jlink`工具来创建我们的自定义 JRE。我们将首先创建一个包含所有必需模块的 JRE：

```java
jlink --add-modules ALL-MODULE-PATH --output jdk-17.0.2-jre
      --strip-debug --no-man-pages --no-header-files
      --compress=2
```

这是一行。在 Linux 中，你可以使用反斜杠（`\`）输入多行命令，而在 Windows 中，你使用撇号（`^`）。此命令的输出将是一个名为`jdk-17.0.2-jre`的文件夹，其中包含仅 76 MB 的 JRE。这比原始 JRE 小，但我们不需要所有 Java 模块；我们只需要三个。以下是我们的新命令：

```java
jlink --add-modules java.base,java.desktop,java.logging
      --output jdk-17.0.2-minimaljre --strip-debug
      --no-man-pages --no-header-files --compress=2
```

现在，我们将在`jdk-17.0.2-minimaljre`文件夹中有一个新的 JRE，大小仅为 41 MB。现在，我们需要使用我们的自定义 JRE 与我们的应用程序一起使用。为了测试我们的 JRE 是否工作，你可以在自定义 JRE 的`bin`文件夹中打开一个终端/控制台窗口，然后执行以下命令来运行你的代码。请注意，路径是为 Windows 准备的，因此它们必须根据 Linux 或 Mac 进行调整：

```java
java.exe -jar C:\dev\Packt\16\BankSwingJUL\target\
  BankSwingJUL-0.1-SNAPSHOT.jar
```

这是一个单行命令。如果一切顺利，你的`BankSwingJUL`应用将运行。现在，是时候将应用程序打包成一个包含我们的应用程序和 JRE 的单个可执行文件了。这将使我们能够在不需要接收者首先安装 Java 的情况下分发我们的应用程序。

# 使用 jpackage 进行打包

自定义 JRE 创建完成后，我们现在可以创建自定义的可安装包了。你可以为 Windows、Linux 或 Mac 创建这些包。你必须使用你的包的目标操作系统。此外，每个操作系统都有额外的步骤。

Windows 需要您安装 WiX 工具集。您可以在[`wixtoolset.org/`](https://wixtoolset.org/)找到它。下载最新版本并安装。当您运行`jpackage`时，它将生成一个 EXE 文件。您可以分发此文件，当运行时，它将在`C:\Program Files`目录中安装运行程序所需的所有内容。可执行 EXE 文件将位于文件夹中，这就是您运行程序的方式。

Linux 用户，根据他们使用的版本，可能需要`rpm-build`或`fakeroot`包。当您运行`jpackage`时，它将为 Debian Linux 生成一个 DEB 文件，或其他发行版生成一个 RPM 文件。您可以分发此文件，当运行时，它将在`/opt/application-name`目录中安装运行程序所需的所有内容。可执行文件将位于文件夹中，这就是您运行程序的方式。

Mac 用户需要 Xcode 命令行工具。当您运行`jpackage`时，它将生成一个 DMG 文件。您可以分发此文件，当运行时，它将在`/Applications/application-name`目录中安装运行程序所需的所有内容。可执行文件将位于文件夹中，这就是您运行程序的方式。

在所有三种情况下，没有必要安装 Java。即使安装了，您也将使用自定义的 JRE。

要使用`jpackage`创建安装程序包，您只需在命令行中输入以下内容：

```java
jpackage --name BankSwingJUL
  --input C:\dev\Packt\16\BankSwingJUL\target
  --main-jar BankSwingJUL-0.1-SNAPSHOT.jar
  --runtime-image C:\dev\Packt\16\jre\jdk-17.0.2-minimaljre
  --dest C:\temp
```

这是一个单行命令。以下是参数的概述：

+   `--name`：添加`-1.0`的可执行文件名称。使用`--app-version`后跟版本标识来覆盖此名称。

+   `--input`：您要打包的 JAR 文件的存放位置。

+   `--main-jar`：包含具有`main`方法的类的 JAR 文件的名称。如果您在 JAR 文件中没有包含具有`main`方法类的`MANIFEST.MF`文件，您可以使用`--main-class`后跟包含`main`方法的类的名称。

+   `--runtime-image`：这是您使用`jlink`创建的 JRE 文件夹的路径和名称。

+   `--dest`：默认情况下，打包的应用程序将位于您发出`jpackage`命令的任何文件夹中。您可以使用此参数选择您想要的文件夹。

在此命令成功完成后，您将拥有一个可执行的程序包，该程序包将安装您的程序，并包含一个可执行文件来运行它。

网络应用程序依赖于应用程序服务器而不是 JRE 来运行。因此，我们不能使用`jpackage`。这就是我们的下一个打包选择，Docker 容器。

# 使用 Docker 容器系统

Docker 是一个平台即服务系统，允许你构建一个运行中的应用程序的镜像，该镜像可以在虚拟化的 Linux 容器中运行。这些镜像可以在支持 Docker 容器的任何计算机上运行。这包括 Windows 和 Linux 发行版。这个镜像可以包含运行程序所需的一切。在本节中，我们将创建一个包含 Java 应用服务器、Java JDK 和我们的`JSF_FinancialCalculator` Web 应用程序的镜像，并在容器中部署它。这之所以重要，是因为大多数云提供商，如 AWS，都支持在 Docker 容器中部署云应用程序。我们不会讨论云部署，因为不同的云提供商工作方式不同。他们共同点是使用 Docker。

第一步是安装 Docker 系统。最简单的方法是从[`www.docker.com/`](https://www.docker.com/)下载并安装 Docker Desktop 系统。它为 Windows、Mac 和 Linux 都提供了版本，它们包含 GUI 界面以及命令行工具。在支持 WSL2 的 Windows 10 或 11 系统上，命令行工具在 Windows 终端和 WSL2 Linux 终端中都可用。这意味着，除了文件路径的描述方式不同外，所有命令在所有操作系统上都是相同的。现在，花点时间安装 Docker。

# 与 Docker 镜像一起工作

虽然我们可以从头开始构建镜像，但还有另一种方法。许多为云而创建软件的组织会提供预构建的镜像。我们可以将这些镜像添加到我们的应用程序中。在我们的例子中，我们想要一个包含 Java 17 和应用服务器的预构建镜像。我们将使用来自 Payara 的镜像。这家公司提供基于 GlassFish 的服务器，在开源社区版本和商业付费版本中都进行了增强。

Docker Hub 上的镜像可能是出于恶意目的而创建的。虽然 Docker 提供了一种扫描漏洞的服务，但你仍应扫描镜像中的任何可执行文件，以检查潜在的恶意行为。你注册的 Docker 计划决定了你可以从或推送到 Hub 的镜像数量。使用免费的个人订阅，你可能可以推送无限数量的公共仓库，但每天的限制为 400 次镜像拉取。商业订阅增加了从仓库拉取的次数，并可以对您的镜像执行漏洞扫描。

启动 Docker Desktop。它包含一个包含基本 Web 服务器的镜像和容器，该服务器具有 Docker 的文档页面。我们将大部分设置工作在命令行上完成，而桌面 GUI 对于查看 Docker 镜像和容器的状态非常有用。

第一步是下载我们将要修改的镜像，通过添加`JSF_FinancialCalculator`应用程序。我们将使用这个程序，与上一章保持不变。以下是命令：

```java
docker pull payara/server-full:6.2023.2-jdk17
```

如果你访问 [`hub.docker.com/r/payara/server-full/tags`](https://hub.docker.com/r/payara/server-full/tags)，你可以看到所有可用的 Payara 服务器版本。正如你从之前的命令中看到的，我们正在拉取包含服务器和 Java 17 的 `server-full:6.2023.2-jdk17` 镜像。在 Docker 中，成功的命令返回一串长数字。

现在，我们需要在一个容器中运行这个镜像。虽然你可以运行多个容器，但使用 TCP 端口的网络应用程序可能会导致冲突。因此，我建议停止任何正在运行的容器。使用 Docker Desktop，从菜单中选择容器列表，查找任何列为 **运行中** 的容器，然后通过点击 **操作** 列中的方块按钮来停止它们。你还可以通过在命令行中输入以下内容来停止容器：

```java
docker stop my_container
```

在这里，`my_container` 被替换为正在运行的容器或镜像的名称。

我们现在希望将这个镜像包装在一个容器中并运行这个镜像：

```java
docker run --name finance -p 8080:8080 -p 4848:4848
            payara/server-full:6.2023.2-jdk17
```

这是一个单行命令。`--name` 开关允许你为容器分配一个名称。如果你省略了这个开关，Docker 将分配一个随机名称。`-p` 开关将容器中的端口映射到计算机的端口。在这个例子中，我们将它们映射到相同的端口。镜像的名称与我们所拉取的镜像名称相同。假设没有错误，你现在可以测试容器了。打开你的浏览器，首先通过输入 `http://localhost:8080` 访问 Payara 主页。接下来，访问管理控制台页面 `https://localhost:4848`。你的浏览器可能会发出警告，因为 TLS 证书是自签名的。忽略警告，你应该会到达登录页面。用户名和密码都是 `admin`。

在上一章的 `JSF_FinancialCalculator` 示例中，你可以在项目的 `target` 文件夹中找到它。

你现在可以通过在浏览器中输入 `http://localhost:8080/JSF_FinancialCalculator` 来验证应用程序是否已正确部署。项目名称必须与 WAR 文件名称匹配。如果一切正常，计算器在你的浏览器中打开，你现在可以基于 `payara/server-full:6.2023.2-jdk17` 图像创建自己的容器，该容器将包含服务器上安装的计算器应用程序。

# 创建 Docker 图像

现在，我们准备创建自己的镜像。首先，我们需要停止我们刚刚使用的容器：

```java
docker stop finance
```

在终端/控制台中，输入以下命令来创建你的新容器，该容器将包含 Payara 图像：

```java
docker create --name transition -p 8080:8080
         -p 4848:4848 payara/server-full:6.2023.2-jdk17
```

名称 `transition` 是任意的，可以是任何你想要的内容。你现在有一个基于 Payara 图像的新容器。我们希望修改这个容器以包含计算器应用程序。第一步是运行这个新容器：

```java
docker start transition
```

在这里最常发生的错误是如果另一个容器正在监听相同的端口。确保任何运行 Payara 的容器或镜像都没有运行。Docker Desktop 应用程序可以显示哪些容器或镜像正在运行。

就像我们在测试 Payara 镜像时做的那样，使用您的浏览器打开 Payara 的管理控制台。现在，将`JSF_FinancialCalculator` WAR 文件部署到服务器上。通过访问应用程序的网页来验证它是否成功运行。

现在，通过输入以下内容，将容器中的镜像更改，将 Web 应用程序的添加永久化：

```java
docker commit transition
```

最后一步。输入以下内容：

```java
docker images
```

您将看到`REPOSITORY`和`TAG`都显示为`<none>`的条目：

```java
REPOSITORY  TAG     IMAGE ID       CREATED          SIZE
<none>      <none>  c0236a80bba3   52 minutes ago   618MB
```

为了解决这个问题，并在创建镜像的最后一步，通过输入以下内容来分配一个标签名称和镜像 ID：

```java
docker tag c0236a80bba3  transition-image
```

注意，必须使用的十六进制数字可以在之前的`docker images`命令的表中找到。在运行`docker tag`之后运行`docker images`，表将显示以下内容：

```java
REPOSITORY        TAG     IMAGE ID    CREATED      SIZE
transition-image  latest  c0236a80bba 32 hours ago 618MB
```

您现在已经在本地仓库中配置了一个镜像。为了任何人都能使用您的镜像，您必须在 Docker Hub 网站上发布它。

# 发布一个镜像

如前所述，出于安全原因，您用作新镜像基础的任何镜像都必须进行漏洞扫描，特别是镜像中的任何可执行代码。免费的个人版允许您拥有无限数量的公共镜像。付费层支持私有镜像。发布的第一步是在 Hub 上创建一个仓库。为此，打开您的浏览器并转到[`hub.docker.com/`](https://hub.docker.com/)。如果需要，请登录您的账户。

接下来，从网页顶部的选项中选择**仓库**。您现在将看到您可能已经创建的任何仓库。点击**创建仓库**。在此页面上，您必须填写表格，输入容器的名称以及可选的描述。它还显示了您的 Docker 用户名。确保**公共**是仓库类型的选项。

现在，您可以将您的镜像推送到 Hub。这里有三个步骤：

1.  登录到 Docker Hub：

    ```java
    docker login --username my_username
    ```

将`my_username`替换为您的 Docker 用户名。您现在将被要求输入您的密码。您将收到成功登录的确认。

1.  接下来，您需要更改您的镜像标签`transition-image`，以匹配您创建的仓库名称`omniprof/transitioning_to_java`。该名称由您的用户名和仓库名称组成：

    ```java
    docker tag transition-image omniprof/
      transitioning_to_java
    ```

1.  现在是最后一步，将您的镜像推送到 Hub：

    ```java
    docker push omniprof/transitioning_to_java
    ```

为了确定您是否成功，请访问 Docker Hub 并选择`omniprof/transitioning_to_java`。

您现在有一个可以与您的团队或客户共享的 Docker 镜像。

# 摘要

在本章中，我们探讨了模块化 Java 的含义。我们利用了 Java 自身已经模块化的这一事实。这使得你可以使用`jlink`构建一个比 JDK 小得多的 JRE。你甚至可以通过只包含你的代码所依赖的模块来进一步减小其大小。

我们然后探讨了两种分发代码的方法。第一种方法是使用`jpackage`为你的应用程序创建一个安装程序。安装程序可以包含你定制的 JRE，并将安装你的程序，以及一个可执行文件来运行应用程序。这通常是分发桌面应用程序的最佳方式。

第二种分发方法使用 Docker 容器系统。Docker 允许我们构建和发布一个包含不仅我们的代码和 JDK，还包括任何其他所需程序的镜像。在我们的例子中，额外的程序是一个安装了财务应用程序的应用服务器。我们构建的镜像被发布到仓库中，例如 Docker Hub。现在，任何在任意操作系统上运行 Docker 的人都可以拉取我们的镜像，并在 Docker 容器中运行它。

这也带我们来到了本书的结尾。我的目标是提供一个参考，帮助那些需要学习和理解 Java 的资深开发者。还有很多东西要学习，但我希望这本书能让你走上正确的道路。

# 进一步阅读

+   *使用 Java 模块的多模块 Maven 应用程序* 模块：[`www.baeldung.com/maven-multi-module-project-java-jpms`](https://www.baeldung.com/maven-multi-module-project-java-jpms)

+   *Java 平台，标准版 – 打包工具用户指南*：[`docs.oracle.com/en/java/javase/14/jpackage/packaging-tool-user-guide.pdf`](https://docs.oracle.com/en/java/javase/14/jpackage/packaging-tool-user-guide.pdf)

+   *Docker* 文档：[`docs.docker.com/`](https://docs.docker.com/)
