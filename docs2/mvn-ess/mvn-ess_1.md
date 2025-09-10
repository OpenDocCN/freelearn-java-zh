# 第一章：Apache Maven 快速入门

Apache Maven 作为构建工具非常受欢迎。然而，实际上，它不仅仅是一个构建工具。它提供了一个全面的构建管理平台。在 Maven 之前，开发者必须花费大量时间来构建构建系统。没有统一的接口。它因项目而异，每次开发者从一个项目转移到另一个项目时，都需要一个学习曲线。Maven 通过引入一个统一的接口来填补这一空白。它仅仅结束了构建工程师的时代。

在本章中，我们将讨论以下主题：

+   在 Ubuntu、Mac OS X 和 Microsoft Windows 上安装和配置 Maven

+   IDE 集成

+   使用 Maven 的高效技巧和窍门

# 安装 Apache Maven

在任何平台上安装 Maven 不仅仅是一个简单的任务。在撰写本书时，最新版本是 3.3.3，可在 [`maven.apache.org/download.cgi`](http://maven.apache.org/download.cgi) 下载。此版本需要 JDK 1.7.0 或更高版本。

### 小贴士

如果你计划从 3.0.*、3.1.* 或 3.2.* 版本升级，你应该注意版本 3.3.3 的 Java 要求。在 Maven 3.3.x 之前，唯一的要求是 JDK 1.5.0 或 JDK 1.6.0（对于 3.2.*）。

Apache Maven 是一个非常轻量级的发行版。它对内存、磁盘空间或 CPU 没有任何硬性要求。Maven 本身是基于 Java 构建的，并且可以在任何运行 **Java 虚拟机**（**JVM**）的操作系统上运行。

## 在 Ubuntu 上安装 Apache Maven

在 Ubuntu 上安装 Maven 只需一条命令。按照以下步骤进行操作：

1.  在命令提示符中运行以下 `apt-get` 命令。你需要有 `sudo` 权限来执行此操作：

    ```java
    $ sudo apt-get install maven

    ```

1.  这需要几分钟才能完成。安装完成后，你可以运行以下命令来验证安装：

    ```java
    $ mvn -version

    ```

1.  如果 Apache Maven 安装成功，你应该会得到以下类似的输出：

    ```java
    $ mvn -version
    Apache Maven 3.3.3
    Maven home: /usr/share/maven
    Java version: 1.7.0_60, vendor: Oracle Corporation
    Java home: /usr/lib/jvm/java-7-oracle/jre
    Default locale: en_US, platform encoding: UTF-8
    OS name: "linux", version: "3.13.0-24-generic", arch: "amd64", family: "unix"

    ```

1.  Maven 安装在 `/usr/share/maven` 目录下。要检查 Maven 安装目录背后的目录结构，请使用以下命令：

    ```java
    $ ls /usr/share/maven
    bin  boot  conf  lib  man

    ```

1.  Maven 配置文件位于 `/etc/maven`，可以使用以下命令列出：

    ```java
    $ ls /etc/maven
    m2.conf  settings.xml

    ```

如果你不想使用 `apt-get` 命令，在基于 Unix 的任何操作系统下安装 Maven 有另一种方法。我们将在下一节讨论这个问题。由于 Mac OS X 是基于 Unix 内核构建的内核，因此，在 Mac OS X 上安装 Maven 与在基于 Unix 的任何操作系统上安装它相同。

## 在 Mac OS X 上安装 Apache Maven

在 OS X Mavericks 之前的许多 OS X 发行版中预装了 Apache Maven。为了验证你的系统中是否已安装 Maven，尝试以下命令。如果没有版本输出，则意味着你没有安装它：

```java
$ mvn –version

```

以下步骤将指导你在 Max OS X Yosemite 上进行 Maven 安装过程：

1.  首先，我们需要下载 Maven 的最新版本。在本书中，我们将使用 Maven 3.3.3，这是撰写本书时的最新版本。Maven 3.3.3 ZIP 分发版可以从 [`maven.apache.org/download.cgi`](http://maven.apache.org/download.cgi) 下载。

1.  将下载的 ZIP 文件解压到 `/usr/share/java` 目录中。你需要有 `sudo` 权限来执行此操作：

    ```java
    $ sudo unzip  apache-maven-3.3.3-bin.zip -d /usr/share/java/

    ```

1.  如果你的系统中已经安装了 Maven，请使用以下命令进行解除链接。`/usr/share/maven` 仅是 Maven 安装目录的符号链接：

    ```java
    $ sudo unlink /usr/share/maven

    ```

1.  使用以下命令创建指向你刚刚解压的最新 Maven 分发的符号链接。你需要有 `sudo` 权限来执行此操作：

    ```java
    $ sudo ln -s /usr/share/java/apache-maven-3.3.3  /usr/share/maven

    ```

1.  使用以下命令更新 `PATH` 环境变量的值：

    ```java
    $ export PATH=$PATH:/usr/share/maven/bin

    ```

1.  使用以下命令更新（或设置）`M2_HOME` 环境变量的值：

    ```java
    $ export M2_HOME=/usr/share/maven

    ```

1.  使用以下命令验证 Maven 的安装：

    ```java
    $ mvn -version
     Apache Maven 3.3.3 (7994120775791599e205a5524ec3e0dfe41d4a06;   2015-04-22T04:57:37-07:00)
     Maven home: /usr/share/maven
     Java version: 1.7.0_75, vendor: Oracle Corporation
     Java home:   /Library/Java/JavaVirtualMachines/jdk1.7.0_75.jdk/Contents/Hom  e/jre
     Default locale: en_US, platform encoding: UTF-8
     OS name: "mac os x", version: "10.10.2", arch: "x86_64",   family: "mac"

    ```

1.  如果在运行前面的命令时出现以下错误，这意味着你的系统中正在运行另一个版本的 Maven，并且 `PATH` 系统变量包含了其 `bin` 目录的路径。如果是这种情况，你需要通过删除旧 Maven 安装路径来清理 `PATH` 系统变量的值：

    ```java
     -Dmaven.multiModuleProjectDirectory system property is not   set. Check $M2_HOME environment variable and mvn script match.

    ```

### 注意

Maven 也可以使用 Homebrew 在 Mac OS X 上安装。此视频详细解释了安装过程——

[`www.youtube.com/watch?v=xTzLGcqUf8k`](https://www.youtube.com/watch?v=xTzLGcqUf8k)

## 在 Microsoft Windows 上安装 Apache Maven

首先，我们需要下载 Maven 的最新版本。Apache Maven 3.3.3 ZIP 分发版可以从 [`maven.apache.org/download.cgi`](http://maven.apache.org/download.cgi) 下载。然后，我们需要执行以下步骤：

1.  将下载的 ZIP 文件解压到 `C:\Program Files\ASF` 目录中。

1.  设置 `M2_HOME` 环境变量并将其指向 `C:\Program Files\ASF\apache-maven-3.3.3`。

1.  在命令提示符下使用以下命令验证 Maven 的安装：

    ```java
    mvn –version

    ```

### 注意

要了解更多关于如何在 Microsoft Windows 上设置环境变量的信息，请参阅 [`www.computerhope.com/issues/ch000549.htm`](http://www.computerhope.com/issues/ch000549.htm)。

# 配置堆大小

一旦你在系统中安装了 Maven，下一步就是对其进行微调以实现最佳性能。默认情况下，最大堆分配为 512 MB，从 256 MB (`-Xms256m` 到 `-Xmx512m`) 开始。默认限制对于构建大型、复杂的 Java 项目来说不够好，建议你至少有 1024 MB 的最大堆。

如果你在 Maven 构建过程中任何时刻遇到 `java.lang.OutOfMemoryError`，那么这通常是由于内存不足造成的。你可以使用 `MAVEN_OPTS` 环境变量在全局级别设置 Maven 的最大允许堆大小。以下命令将在任何基于 Unix 的操作系统（包括 Linux 和 Mac OS X）中设置堆大小。确保设置的堆大小不超过运行 Maven 的机器的系统内存：

```java
$ export MAVEN_OPTS="-Xmx1024m -XX:MaxPermSize=128m"

```

如果你使用的是 Microsoft Windows，请使用以下命令：

```java
$ set MAVEN_OPTS=-Xmx1024m -XX:MaxPermSize=128m

```

在这里，`-Xmx` 用于指定最大堆大小，而 `-XX:MaxPermSize` 用于指定最大 **永久代**（**PermGen**）大小。

### 注意

Maven 在 JVM 上作为 Java 进程运行。随着构建的进行，它不断创建 Java 对象。这些对象存储在分配给 Maven 的内存中。存储 Java 对象的内存区域被称为堆。堆在 JVM 启动时创建，随着创建的对象越来越多，它将增加到定义的最大限制。`-Xms` JVM 标志用于指示 JVM 在创建堆时应设置的最低值。`-Xmx` JVM 标志设置最大堆大小。

PermGen 是 JVM 管理的内存区域，用于存储 Java 类的内部表示。可以通过 `-XX:MaxPermSize` JVM 标志设置 PermGen 的最大大小。

当 Java 虚拟机无法为 Maven 分配足够的内存时，可能会导致 `OutOfMemoryError`。要了解更多关于 Maven `OutOfMemoryError` 的信息，请参阅 [`cwiki.apache.org/confluence/display/MAVEN/OutOfMemoryError`](https://cwiki.apache.org/confluence/display/MAVEN/OutOfMemoryError)。

# 嗨，Maven！

开始使用 Maven 项目的最简单方法是使用 `archetype` 插件的 `generate` 目标来生成一个简单的 Maven 项目。Maven 架构在 第三章、*Maven 架构* 中详细讨论，插件在 第四章、*Maven 插件* 中介绍。

让我们从简单的例子开始：

```java
$ mvn archetype:generate 
 -DgroupId=com.packt.samples 
 -DartifactId=com.packt.samples.archetype 
 -Dversion=1.0.0 
 -DinteractiveMode=false

```

此命令将调用 Maven `archetype` 插件的 `generate` 目标来创建一个简单的 Java 项目。你会看到以下项目结构被创建，其中包含一个示例 `POM` 文件。根目录或基本目录的名称是从 `artifactId` 参数的值派生出来的：

```java
com.packt.samples.archetype 
               |-pom.xml
               |-src
               |-main/java/com/packt/samples/App.java
               |-test/java/com/packt/samples/AppTest.java    
```

示例 `POM` 文件将只包含对 `junit` JAR 文件的依赖，作用域为 `test`：

```java
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.packt.samples</groupId>
  <artifactId>com.packt.samples.archetype</artifactId>
  <packaging>jar</packaging>
  <version>1.0.0</version>
  <name>com.packt.samples.archetype</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

生成的 `App.java` 类将包含以下模板代码。包的名称是从提供的 `groupId` 参数派生出来的。如果你想将不同的值用作包名，那么你需要将此值作为命令本身的一部分传递，例如 `-Dpackage=com.packt.samples.application`：

```java
package com.packt.samples;

/**
 * Hello world!
 *
 */
public class App 
{
    public static void main( String[] args )
    {
        System.out.println( "Hello World!" );
    }
}
```

要构建示例项目，请从存在 `pom.xml` 文件的 `com.packt.samples.archetype` 目录中运行以下命令：

```java
$ mvn clean install

```

# 契约优于配置

约定优于配置是 Apache Maven 背后的主要设计哲学之一。让我们通过几个例子来了解一下。

可以使用以下配置文件（`pom.xml`）创建一个完整的 Maven 项目：

```java
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.packt</groupId>
  <artifactId>sample-one</artifactId>
  <version>1.0.0</version>
</project>
```

### 注意

Maven 的 `POM` 文件以 `<project>` 元素开始。始终使用模式定义 `<project>` 元素。一些工具没有它无法验证文件：

```java
<project xmlns=http://maven.apache.org/POM/4.0.0
         xmlns:xsi=………
         xsi:schemaLocation="…">
```

`pom.xml` 文件是任何 Maven 项目的核心，并在 第二章") *理解项目对象模型 (POM)* 中详细讨论。复制前面的配置元素，并从中创建一个 `pom.xml` 文件。然后，将其放置在名为 `chapter-01` 的目录中，然后在其中创建以下子目录：

+   `chapter-01/src/main/java`

+   `chapter-01/src/test/java`

现在，你可以在 `chapter-01/src/main/java` 下放置你的 Java 代码，并在 `chapter-01/src/test/java` 下放置测试用例。使用以下命令从 `pom.xml` 所在的位置运行 Maven 构建：

```java
$ mvn clean install

```

你在示例 `pom.xml` 文件中找到的这个小配置与许多约定相关联：

+   Java 源代码位于 `{base-dir}/src/main/java`

+   测试用例位于 `{base-dir}/src/test/java`

+   生成的构件类型是 JAR 文件

+   编译后的类文件被复制到 `{base-dir}/target/classes`

+   最终的构件被复制到 `{base-dir}/target`

+   [`repo.maven.apache.org/maven2`](http://repo.maven.apache.org/maven2)，被用作仓库 URL。

如果有人需要覆盖 Maven 的默认、传统行为，这也是可能的。以下示例 `pom.xml` 文件展示了如何覆盖一些前面的默认值：

```java
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.packt</groupId>
  <artifactId>sample-one</artifactId>
  <version>1.0.0</version>
  <packaging>jar</packaging>

  <build>    
    <sourceDirectory>${basedir}/src/main/java</sourceDirectory>              
    <testSourceDirectory>${basedir}/src/test/java               
                                         </testSourceDirectory>     
    <outputDirectory>${basedir}/target/classes
                                             </outputDirectory>     
  </build>
</project>
```

# Maven 仓库

Maven 如何找到并加载给定 Maven 项目的依赖项背后的魔法是 Maven 仓库。在你的 Maven 项目的相应 `pom.xml` 文件中，在 `<dependencies>` 元素下，你可以定义所有构建项目所需的依赖项的引用。`pom.xml` 文件中定义的每个依赖项都使用 Maven 坐标唯一标识。Maven 坐标唯一标识一个项目、依赖项或 POM 中定义的插件。每个实体都由组合的组标识符、构件标识符和版本（当然，还有打包和分类器）唯一标识。Maven 坐标在 第二章") *理解项目对象模型 (POM)* 中详细讨论。一旦 Maven 找出给定项目的所有必需依赖项，它将它们加载到 Maven 仓库的本地文件系统中，并将它们添加到项目的类路径中。

按照惯例，Maven 使用 [`repo.maven.apache.org/maven2`](http://repo.maven.apache.org/maven2) 作为仓库。如果构建项目所需的所有工件都存在于这个仓库中，那么它们将被加载到本地文件系统或本地 Maven 仓库中，默认情况下，位于 `USER_HOME/.m2/repository`。您可以在 `pom.xml` 文件的 `<repositories>` 元素下在项目级别添加自定义仓库，或者在 `MAVEN_HOME/conf/settings.xml` 文件下在全局级别添加。

# IDE 集成

大多数硬核开发者都不想离开他们的 IDE。不仅是为了编码，还包括构建、部署、测试，如果可能的话，他们愿意从 IDE 本身完成所有这些。大多数流行的 IDE 都支持 Maven 集成，并且它们已经开发了各自的插件来支持 Maven。

## NetBeans 集成

NetBeans 6.7 或更高版本自带内置的 Maven 集成，而 NetBeans 7.0 则包含 Maven 3 的完整副本，并在构建时运行，就像您从命令行运行一样。对于 6.9 或更早版本，您必须下载 Maven 构建，并配置 IDE 以运行此构建。有关 Maven 和 NetBeans 集成的更多信息，请参阅 [`wiki.netbeans.org/MavenBestPractices`](http://wiki.netbeans.org/MavenBestPractices)。

## IntelliJ IDEA 集成

IntelliJ IDEA 内置了对 Maven 的支持；因此，您不需要执行任何额外的步骤来安装它。有关 Maven 和 IntelliJ IDEA 集成的更多信息，请参阅 [`wiki.jetbrains.net/intellij/Creating_and_importing_Maven_projects`](http://wiki.jetbrains.net/intellij/Creating_and_importing_Maven_projects)。

## Eclipse 集成

M2Eclipse 项目通过 Eclipse IDE 提供了一流的 Maven 支持。有关 Maven 和 Eclipse 集成的更多信息，请参阅 [`www.eclipse.org/m2e/`](https://www.eclipse.org/m2e/)。

### 注意

由 *Packt Publishing* 出版的书籍 *Maven for Eclipse* 详细讨论了 Maven 和 Eclipse 集成，请参阅 [`www.packtpub.com/application-development/maven-eclipse`](https://www.packtpub.com/application-development/maven-eclipse)。

# 故障排除

如果一切正常，我们就不必担心故障排除。然而，大多数情况下并非如此。Maven 构建可能会因许多原因而失败，其中一些在您的控制范围内，也有一些不在您的控制范围内。了解适当的故障排除技巧有助于您确定确切的问题。以下部分列出了一些最常用的故障排除技巧。随着本书的进行，我们将扩展这个列表。

## 启用 Maven 调试级别日志

一旦启用 Maven 调试级别日志记录，它将在构建过程中打印出它所采取的所有操作。要启用调试级别日志记录，请使用以下命令：

```java
$ mvn clean install –X

```

## 构建依赖树

如果你发现在你的 Maven 项目中任何依赖项存在问题，第一步是构建一个依赖树。这显示了每个依赖项的来源。要构建依赖树，请在你的项目 `POM` 文件上运行以下命令：

```java
$ mvn dependency:tree

```

以下显示了针对 Apache Rampart 项目执行的前一个命令的截断输出：

```java
[INFO] --------------------------------------------------------------
[INFO] Building Rampart - Trust 1.6.1-wso2v12
[INFO] --------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.1:tree (default-cli) @ rampart-trust ---
[INFO] org.apache.rampart:rampart-trust:jar:1.6.1-wso2v12
[INFO] +- org.apache.rampart:rampart-policy:jar:1.6.1-wso2v12:compile
[INFO] +- org.apache.axis2:axis2-kernel:jar:1.6.1-wso2v10:compile
[INFO] |  +- org.apache.ws.commons.axiom:axiom-api:jar:1.2.11-wso2v4:compile (version managed from 1.2.11)
[INFO] |  |  \- jaxen:jaxen:jar:1.1.1:compile
[INFO] |  +- org.apache.ws.commons.axiom:axiom-impl:jar:1.2.11-wso2v4:compile (version managed from 1.2.11)
[INFO] |  +- org.apache.geronimo.specs:geronimo-ws-metadata_2.0_spec:jar:1.1.2:compile
[INFO] |  +- org.apache.geronimo.specs:geronimo-jta_1.1_spec:jar:1.1:compile
[INFO] |  +- javax.servlet:servlet-api:jar:2.3:compile
[INFO] |  +- commons-httpclient:commons-httpclient:jar:3.1:compile
[INFO] |  |  \- commons-codec:commons-codec:jar:1.2:compile
[INFO] |  +- commons-fileupload:commons-fileupload:jar:1.2:compile

```

## 查看所有环境变量和系统属性

如果你系统中有多个 JDK 安装，你可能想知道 Maven 使用的是哪个。以下命令将显示为给定 Maven 项目设置的 所有环境变量和系统属性：

```java
$ mvn help:system

```

以下命令的截断输出如下：

```java
======================Platform Properties Details====================

=====================================================================
System Properties
=====================================================================

java.runtime.name=Java(TM) SE Runtime Environment
sun.boot.library.path= /Library/Java/JavaVirtualMachines/jdk1.7.0_75.jdk/Contents/Home/jre/lib
java.vm.version= 24.75-b04
awt.nativeDoubleBuffering=true
gopherProxySet=false
mrj.build=11M4609
java.vm.vendor=Apple Inc.
java.vendor.url=http://www.apple.com/
guice.disable.misplaced.annotation.check=true
path.separator=:
java.vm.name=Java HotSpot(TM) 64-Bit Server VM
file.encoding.pkg=sun.io
sun.java.launcher=SUN_STANDARD
user.country=US
sun.os.patch.level=unknown

========================================================
Environment Variables
========================================================

JAVA_HOME=/System/Library/Frameworks/JavaVM.framework/Versions/CurrentJDK/Home
HOME=/Users/prabath
TERM_SESSION_ID=C2CEFB58-4705-4C67-BE1F-9E4179F96391
M2_HOME=/usr/share/maven/maven-3.3.3/
COMMAND_MODE=unix2003
Apple_PubSub_Socket_Render=/tmp/launch-w7NZbG/Render
LOGNAME=prabath
USER=prabath 

```

## 查看有效的 POM 文件

当配置中没有覆盖配置参数时，Maven 使用默认值。这正是我们在 *约定优于配置* 部分讨论的内容。如果我们使用本章之前使用的相同样本 `POM` 文件，我们可以看到使用以下命令的有效 `POM` 文件将如何显示。这也是查看 Maven 使用默认值的最有效方式：

```java
$ mvn help:effective-pom

```

### 注意

关于 `effective-pom` 命令的更多细节在第二章")中讨论，*理解项目对象模型 (POM)*。

## 查看依赖类路径

以下命令将列出构建 `classpath` 中的所有 JAR 文件和目录：

```java
$ mvn dependency:build-classpath

```

以下显示了针对 Apache Rampart 项目执行的前一个命令的截断输出：

```java
[INFO] --------------------------------------------------------------
[INFO] Building Rampart - Trust 1.6.1-wso2v12
[INFO] --------------------------------------------------------------
[INFO] 
[INFO] --- maven-dependency-plugin:2.1:build-classpath (default-cli) @ rampart-trust ---
[INFO] Dependencies classpath:
/Users/prabath/.m2/repository/bouncycastle/bcprov-jdk14/140/bcprov-jdk14-140.jar:/Users/prabath/.m2/repository/commons-cli/commons-cli/1.0/commons-cli-1.0.jar:/Users/prabath/.m2/repository/commons-codec/commons-codec/1.2/commons-codec-1.2.jar:/Users/prabath/.m2/repository/commons-collections/commons-collections/3.1/commons-collections-3.1.jar 

```

# 摘要

本章重点介绍了围绕 Maven 建立基本知识体系，以便将所有读者带入一个共同的基础。它从解释在 Ubuntu、Mac OS X 和 Microsoft Windows 操作系统下安装和配置 Maven 的基本步骤开始。本章的后半部分涵盖了 Maven 的一些常用技巧和窍门。随着本书的进行，本章中提到的某些概念将得到详细讨论。

在下一章中，我们将详细讨论**Maven 项目对象模型**（**POM**）。
