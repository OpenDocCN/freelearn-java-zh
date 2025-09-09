# 第七章. 使用 WebStart 和浏览器插件

在本章中，我们将涵盖以下主题：

+   在 Linux 上构建 IcedTea 浏览器插件

+   在 Linux 上使用 IcedTea Java WebStart 实现

+   准备 IcedTea Java WebStart 实现以供 Mac OS X 使用

+   准备 IcedTea Java WebStart 实现以供 Windows 使用

# 简介

很长一段时间内，对于最终用户来说，Java 小程序技术是整个 Java 世界的面孔。对于许多非开发者来说，Java 这个词本身就是一个同义词，指的是允许在网页浏览器中运行 Java 小程序的 Java 浏览器插件。**Java WebStart**技术与 Java 浏览器插件类似，但它在远程加载的 Java 应用程序上作为独立的应用程序运行，位于网页浏览器之外。

OpenJDK 开源项目不包含浏览器插件和 WebStart 技术的实现。Oracle Java 发行版，尽管与 OpenJDK 代码库紧密匹配，但为这些技术提供了自己的闭源实现。

IcedTea-Web 项目包含浏览器插件和 WebStart 技术的免费和开源实现。IcedTea-Web 浏览器插件仅支持 GNU/Linux 操作系统，而 WebStart 实现是跨平台的。

虽然 IcedTea 对 WebStart 的实现经过充分测试且适用于生产环境，但它与 Oracle WebStart 实现存在许多不兼容性。这些差异可以被视为边缘情况；其中一些包括：

+   **解析格式不正确的 JNLP 描述符文件时的不同行为**：Oracle 实现通常对格式不正确的描述符更为宽容。

+   **JAR（重新）下载和缓存行为差异**：Oracle 实现使用缓存更为积极。

+   **声音支持差异**：这是由于 Oracle Java 和 IcedTea 在 Linux 上声音支持之间的差异。Linux 历史上有多达不同的声音提供者（ALSA、PulseAudio 等），而 IcedTea 对不同提供者的支持更广泛，这可能导致声音配置错误。

IcedTea-Web 浏览器插件（因为它基于 WebStart 构建）也存在这些不兼容性。除此之外，它还可能在浏览器集成方面存在更多不兼容性。用户界面表单和一般与浏览器相关的操作，如从 JavaScript 代码中访问，应与两种实现都能良好工作。但历史上，浏览器插件被广泛用于安全关键的应用程序，如在线银行客户端。这类应用程序通常需要来自浏览器的安全功能，例如访问证书存储或硬件加密设备，这些设备可能因操作系统（例如，仅支持 Windows）、浏览器版本、Java 版本等因素而有所不同。因此，许多实际应用程序在 Linux 上运行 IcedTea-Web 浏览器插件时可能会遇到问题。

WebStart 和浏览器插件都是基于从远程位置下载（可能是不可信的）代码的想法构建的，并且对代码进行适当的权限检查和沙箱执行是一个臭名昭著的复杂任务。通常在 Oracle 浏览器插件中报告的安全问题（最广为人知的是 2012 年期间的问题）也在 IcedTea-Web 中单独修复。

# 在 Linux 上构建 IcedTea 浏览器插件

IcedTea-Web 项目本身不是跨平台的；它是为 Linux 开发的，因此可以在流行的 Linux 发行版上非常容易地构建。它的两个主要部分（存储在源代码库中的相应目录中）是`netx`和`plugin`。

NetX 是 WebStart 技术的纯 Java 实现。我们将在本章后续的配方中更详细地探讨它。

插件是使用支持多个浏览器的`NPAPI`插件架构实现的浏览器插件。插件部分用 Java 编写，部分用本地代码（C++）编写，并且官方仅支持基于 Linux 的操作系统。关于`NPAPI`存在一种观点，即这种架构过时、过于复杂且不安全，而现代网络浏览器已经内置了足够的特性，无需外部插件。浏览器已经逐渐减少对`NPAPI`的支持。尽管如此，在撰写本书时，IcedTea-Web 浏览器插件在所有主要的 Linux 浏览器（Firefox 及其衍生品、Chromium 及其衍生品和 Konqueror）上都能正常工作。

我们将使用 Ubuntu 12.04 LTS amd64 从源代码构建 IcedTea-Web 浏览器插件。

## 准备工作

对于这个配方，我们需要一个干净的 Ubuntu 12.04 系统，并且安装了 Firefox 网络浏览器。

## 如何操作...

以下步骤将帮助您构建 IcedTea-Web 浏览器插件：

1.  安装 OpenJDK 7 的预包装二进制文件：

    ```java
    sudo apt-get install openjdk-7-jdk

    ```

1.  安装 GCC 工具链和构建依赖项：

    ```java
    sudo apt-get build-dep openjdk-7

    ```

1.  安装浏览器插件的特定依赖项：

    ```java
    sudo apt-get install firefox-dev

    ```

1.  下载并解压 IcedTea-Web 源代码 tar 包：

    ```java
    wget http://icedtea.wildebeest.org/download/source/icedtea-web-1.4.2.tar.gz
    tar xzvf icedtea-web-1.4.2.tar.gz

    ```

1.  运行`configure`脚本以设置构建环境：

    ```java
    ./configure

    ```

1.  运行构建过程：

    ```java
    make

    ```

1.  将新构建的插件安装到`/usr/local`目录中：

    ```java
    sudo make install

    ```

1.  配置 Firefox 网络浏览器以使用新构建的插件库：

    ```java
    mkdir ~/.mozilla/plugins
    cd ~/.mozilla/plugins
    ln -s /usr/local/IcedTeaPlugin.so libjavaplugin.so

    ```

1.  检查 IcedTea-Web 插件是否出现在**工具** | **附加组件** | **插件**下。

1.  打开[`java.com/en/download/installed.jsp`](http://java.com/en/download/installed.jsp)网页以验证浏览器插件是否工作。

## 它是如何工作的...

IcedTea 浏览器插件需要编译成功，这需要 IcedTea Java 实现。Ubuntu 12.04 中的预包装 OpenJDK 7 二进制文件基于 IcedTea，因此我们首先安装了它们。插件使用的是 GNU Autconf 构建系统，这是免费软件工具之间常见的。需要`xulrunner-dev`包来访问`NPAPI`头文件。

构建的插件可能仅对当前用户安装到 Firefox 中，无需管理员权限。为此，我们在 Firefox 期望找到`libjavaplugin.so`插件库的位置创建了一个到我们插件的符号链接。

## 更多内容...

该插件也可以安装到具有`NPAPI`支持的其它浏览器中，但不同浏览器和不同 Linux 发行版的安装说明可能不同。

由于`NPAPI`架构不依赖于操作系统，理论上可以为非 Linux 操作系统构建插件。但目前没有这样的端口计划。

## 参见

+   *在 Linux 上使用 IcedTea Java WebStart 实现*配方

+   *为 Mac OS X 准备 IcedTea Java WebStart 实现*配方

+   *为 Windows 准备 IcedTea Java WebStart 实现*配方

+   第五章, *构建 IcedTea*

+   来自第四章的*使用 GNU Autoconf*配方，*构建 OpenJDK 8*

+   IcedTea-Web 项目网站：[`icedtea.classpath.org/wiki/IcedTea-Web`](http://icedtea.classpath.org/wiki/IcedTea-Web)

+   NetX 项目网站：[`jnlp.sourceforge.net/netx/`](http://jnlp.sourceforge.net/netx/)

+   Java WebStart 开发者指南：[`docs.oracle.com/javase/6/docs/technotes/guides/javaws/`](http://docs.oracle.com/javase/6/docs/technotes/guides/javaws/)

+   NPAPI 项目网站：[`wiki.mozilla.org/NPAPI`](https://wiki.mozilla.org/NPAPI)

+   不同浏览器中 NPAPI 支持的详细信息：[`www.firebreath.org/display/documentation/Browser+Plugins+in+a+post-NPAPI+world`](http://www.firebreath.org/display/documentation/Browser+Plugins+in+a+post-NPAPI+world)

# 在 Linux 上使用 IcedTea Java WebStart 实现

在 Java 平台上，JVM 需要为它想要使用的每个类执行类加载过程。这个过程对 JVM 来说是透明的，加载的类的实际字节码可能来自许多来源之一。例如，此方法允许 Java Applet 类从远程服务器加载到浏览器内的 Java 进程中。远程类加载还可以用于在不与浏览器集成的情况下以独立模式运行远程加载的 Java 应用程序。这种技术称为 Java WebStart，是在**Java 规范请求**（**JSR**）编号 56 下开发的。

要远程运行 Java 应用程序，WebStart 需要一个应用程序描述符文件，该文件应使用**Java 网络启动协议**（**JNLP**）语法编写。该文件用于定义远程服务器以加载应用程序以及一些元信息。WebStart 应用程序可以通过点击网页上的 JNLP 链接或在事先获取的 JNLP 文件上运行，而不使用浏览器。在任何情况下，运行应用程序都与浏览器完全分离，但使用与 Java Applets 类似的沙箱安全模型。

OpenJDK 项目不包含 WebStart 实现；Oracle Java 发行版提供了自己的闭源 WebStart 实现。开源的 WebStart 实现作为 IcedTea-Web 项目的一部分存在。它最初基于**网络执行**（**NetX**）项目。与 Applet 技术不同，WebStart 不需要任何网络浏览器集成。这允许开发者使用纯 Java 而不需要本地代码来实现 NetX 模块。为了与基于 Linux 的操作系统集成，IcedTea-Web 实现了`javaws`命令作为 shell 脚本，用于启动带有正确参数的`netx.jar`文件。

在这个菜谱中，我们将从官方 IcedTea-Web 源代码 tarball 构建 NetX 模块。

## 准备工作

对于这个菜谱，我们需要一个干净安装了 Firefox 浏览器的 Ubuntu 12.04。

## 如何做到这一点...

以下步骤将帮助您构建 NetX 模块：

1.  安装预包装的 OpenJDK 7 二进制文件：

    ```java
    sudo apt-get install openjdk-7-jdk

    ```

1.  安装 GCC 工具链和构建依赖项：

    ```java
    sudo apt-get build-dep openjdk-7

    ```

1.  下载并解压 IcedTea-Web 源代码 tarball：

    ```java
    wget http://icedtea.wildebeest.org/download/source/icedtea-web-1.4.2.tar.gz
    tar xzvf icedtea-web-1.4.2.tar.gz

    ```

1.  运行`configure`脚本来设置构建环境，排除构建中的浏览器插件：

    ```java
    ./configure –disable-plugin

    ```

1.  运行构建过程：

    ```java
    make

    ```

1.  安装新构建的插件到`/usr/local`目录：

    ```java
    sudo make install

    ```

1.  从 Java 教程中运行 WebStart 应用程序示例：

    ```java
    javaws http://docs.oracle.com/javase/tutorialJWS/samples/
    deployment/dynamictree_webstartJWSProject/dynamictree_webstart.jnlp

    ```

## 它是如何工作的...

`javaws` shell 脚本安装到`/usr/local/*`目录。当使用路径或 JNLP 文件的链接启动时，`javaws`将启动`netx.jar`文件，将其添加到引导类路径（出于安全原因）并作为参数提供 JNLP 链接。

## 相关阅读

+   *为 Mac OS X 准备 IcedTea Java WebStart 实现*菜谱

+   *为 Windows 准备 IcedTea Java WebStart 实现*菜谱

+   *在 Linux 上构建 IcedTea 浏览器插件*菜谱

+   来自第四章，*构建 OpenJDK 8*的*使用 GNU Autoconf 工作*菜谱

+   IcedTea-Web 项目网站在[`icedtea.classpath.org/wiki/IcedTea-Web`](http://icedtea.classpath.org/wiki/IcedTea-Web)

+   NetX 项目网站在[`jnlp.sourceforge.net/netx/`](http://jnlp.sourceforge.net/netx/)

+   Java WebStart 开发者指南在[`docs.oracle.com/javase/6/docs/technotes/guides/javaws/`](http://docs.oracle.com/javase/6/docs/technotes/guides/javaws/)

# 为 Mac OS X 准备 IcedTea Java WebStart 实现

IcedTea-Web 项目中的 NetX WebStart 实现是用纯 Java 编写的，因此它也可以在 Mac OS X 上使用。IcedTea-Web 只为基于 Linux 的操作系统提供`javaws`启动器实现。在这个菜谱中，我们将为 Mac OS X 创建一个简单的 WebStart 启动脚本实现。

## 准备工作

对于这个菜谱，我们需要安装了 Java 7 的 Mac OS X Lion（预构建的 OpenJDK 或 Oracle 版本）。我们还需要从 IcedTea-Web 项目获取`netx.jar`模块，可以使用之前菜谱中的说明来构建。

## 如何做到这一点...

以下步骤将帮助您在 Mac OS X 上运行 WebStart 应用程序：

1.  从 Java 教程 [`docs.oracle.com/javase/tutorialJWS/samples/deployment/dynamictree_webstartJWSProject/dynamictree_webstart.jnlp`](http://docs.oracle.com/javase/tutorialJWS/samples/deployment/dynamictree_webstartJWSProject/dynamictree_webstart.jnlp) 下载 JNLP 描述符示例。

1.  测试此应用程序是否可以通过 `netx.jar` 从终端运行：

    ```java
    java -Xbootclasspath/a:netx.jar net.sourceforge.jnlp.runtime.Boot dynamictree_webstart.jnlp

    ```

1.  使用以下内容创建 `wslauncher.sh` bash 脚本：

    ```java
    #!/bin/bash
    if [ "x$JAVA_HOME" = "x" ] ; then
     JAVA="$( which java 2>/dev/null )"
    else
     JAVA="$JAVA_HOME"/bin/java
    fi
    if [ "x$JAVA" = "x" ] ; then
     echo "Java executable not found"
     exit 1
    fi
    if [ "x$1" = "x" ] ; then
     echo "Please provide JNLP file as first argument"
     exit 1
    fi
    $JAVA -Xbootclasspath/a:netx.jar net.sourceforge.jnlp.runtime.Boot $1

    ```

1.  将启动脚本标记为可执行：

    ```java
    chmod 755 wslauncher.sh

    ```

1.  使用启动脚本运行应用程序：

    ```java
    ./wslauncher.sh dynamictree_webstart.jnlp

    ```

## 它是如何工作的...

`next.jar` 文件包含一个 Java 应用程序，可以读取 JNLP 文件并下载和运行 JNLP 中描述的类。但由于安全原因，`next.jar` 不能直接作为应用程序启动（使用 `java -jar netx.jar` 语法）。相反，`netx.jar` 被添加到受保护的启动类路径中，并直接指定主类来运行。这允许我们在沙盒模式下下载应用程序。

`wslauncher.sh` 脚本尝试使用 `PATH` 和 `JAVA_HOME` 环境变量查找 Java 可执行文件，然后通过 `netx.jar` 启动指定的 JNLP。

## 更多内容...

`wslauncher.sh` 脚本为从终端运行 WebStart 应用程序提供了一个基本解决方案。为了正确地将 `netx.jar` 集成到操作系统环境中（以便能够使用来自网络浏览器的 JNLP 链接启动 WebStart 应用程序），可以使用本地启动器或自定义平台脚本解决方案。这些解决方案超出了本书的范围。

## 相关内容

+   *使用 Linux 上的 IcedTea Java WebStart 实现* 的配方

+   来自 第三章，*构建 OpenJDK 7* 的 *在 Mac OS X 上构建 OpenJDK 7* 的配方

+   IcedTea-Web 项目网站 [`icedtea.classpath.org/wiki/IcedTea-Web`](http://icedtea.classpath.org/wiki/IcedTea-Web)

+   NetX 项目网站 [`jnlp.sourceforge.net/netx/`](http://jnlp.sourceforge.net/netx/)

+   Java WebStart 开发者指南 [`docs.oracle.com/javase/6/docs/technotes/guides/javaws/`](http://docs.oracle.com/javase/6/docs/technotes/guides/javaws/)

+   苹果公司关于在 Apple Java 上启用 WebStart 的支持文章 [`support.apple.com/kb/HT5559`](http://support.apple.com/kb/HT5559)

# 准备 Windows 上的 IcedTea Java WebStart 实现

IcedTea-Web 项目的 NetX WebStart 实现是用纯 Java 编写的，因此它也可以在 Windows 上使用；我们还在本章前面的配方中在 Linux 和 Mac OS X 上使用了它。在这个配方中，我们将为 Windows 创建一个简单的 WebStart 启动脚本实现。

## 准备工作

对于这个配方，我们需要一个运行 Java 7（预构建的 OpenJDK 或 Oracle 版本）的 Windows 版本。我们还需要来自 IcedTea-Web 项目的 `netx.jar` 模块，可以使用本章前面的配方中的说明来构建。

## 如何做到这一点...

以下步骤将帮助您在 Windows 上运行 WebStart 应用程序：

1.  从 Java 教程中下载 JNLP 描述符示例：

    ```java
    http://docs.oracle.com/javase/tutorialJWS/samples/deployment/dynamictree_webstartJWSProject/dynamictree_webstart.jnlp

    ```

1.  使用`netx.jar`测试此应用程序是否可以从命令提示符中运行：

    ```java
    java -Xbootclasspath/a:netx.jar net.sourceforge.jnlp.runtime.Boot dynamictree_webstart.jnlp

    ```

1.  创建一个批处理文件，名为`wslauncher.bat`，内容如下：

    ```java
    @echo off
    if "%JAVA_HOME%" == "" goto noJavaHome
    if not exist "%JAVA_HOME%\bin\javaw.exe" goto noJavaHome
    set "JAVA=%JAVA_HOME%\bin\javaw.exe"
    if "%1" == "" goto noJnlp
    start %JAVA% -Xbootclasspath/a:netx.jar net.sourceforge.jnlp.runtime.Boot %1
    exit /b 0
    :noJavaHome
    echo The JAVA_HOME environment variable is not defined correctly
    exit /b 1
    :noJnlp
    echo Please provide JNLP file as first argument
    exit /b 1

    ```

1.  使用启动脚本运行应用程序：

    ```java
    wslauncher.bat dynamictree_webstart.jnlp

    ```

## 它是如何工作的...

由于安全原因，`netx.jar` 模块必须添加到启动类路径中，因为它不能直接运行。

`wslauncher.bat` 脚本尝试使用`JAVA_HOME`环境变量查找 Java 可执行文件，然后通过`netx.jar`启动指定的 JNLP。

## 更多内容...

`wslauncher.bat` 脚本可以被注册为默认应用程序以运行 JNLP 文件。这将允许您从网页浏览器中运行 WebStart 应用程序。但当前的脚本在启动应用程序之前会短暂显示批处理窗口。它也不支持在 Windows 注册表中查找 Java 可执行文件。可以使用 Visual Basic 脚本（或任何其他本地脚本解决方案）或作为本地可执行启动器编写一个没有这些问题的更高级的脚本。这些解决方案超出了本书的范围。

## 参考以下内容

+   *准备 IcedTea Java WebStart 实现以兼容 Mac OS X* 的配方

+   来自第三章的 *在 Windows 7 x64 SP1 上构建 64 位 OpenJDK 7* 配方，*构建 OpenJDK 7*

+   IcedTea-Web 项目网站，请访问[`icedtea.classpath.org/wiki/IcedTea-Web`](http://icedtea.classpath.org/wiki/IcedTea-Web)

+   NetX 项目网站，请访问[`jnlp.sourceforge.net/netx/`](http://jnlp.sourceforge.net/netx/)

+   Java WebStart 开发者指南，请访问[`docs.oracle.com/javase/6/docs/technotes/guides/javaws/`](http://docs.oracle.com/javase/6/docs/technotes/guides/javaws/)

+   来自微软的 ActiveX 技术文章，类似于 NPAPI，请参阅[`msdn.microsoft.com/en-us/library/aa751968%28v=vs.85%29.aspx`](http://msdn.microsoft.com/en-us/library/aa751968%28v=vs.85%29.aspx)
