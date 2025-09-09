# 第五章：构建 IcedTea

在本章中，我们将涵盖以下主题：

+   构建 IcedTea 6

+   构建 IcedTea 7

+   使用 IcedTea 补丁构建 OpenJDK 7

+   使用 NSS 安全提供者构建 IcedTea 7

+   使用 SystemTap 支持构建 IcedTea 6

# 简介

**IcedTea** 是 Red Hat 于 2007 年 6 月启动的针对 OpenJDK 的构建和集成项目。它允许用户使用免费软件工具构建 OpenJDK。它还提供了 Java 浏览器插件和 Java Web Start 的替代开源实现，作为 IcedTea-Web 子项目的一部分。

考虑到 OpenJDK 项目本身当前的状态，理解 IcedTea 项目的全部价值并不容易。OpenJDK 的某些功能，如现在看似标准的没有二进制组件和高级构建系统，是在 IcedTea 项目中首创的。

IcedTea 项目始于 2007 年，由 Red Hat 和 GNU classpath 启动，在 Sun Java 源代码在开源许可下发布后不久。主要目标是创建一个免费的 Java 软件包，用于包含在 GNU/Linux 分发中。流行的分发（如 Fedora 或 Arch）对包含的软件有严格的要求。一般来说，完整的源代码必须在开源许可下可用，并且二进制文件必须使用免费软件构建工具构建。在发布时，一些 OpenJDK 模块（例如，字体和声音相关的模块）仅以二进制形式提供，并且只能使用专有的 Sun Java 编译器进行构建。

IcedTea 集成了 GNU Autoconf 构建系统。这使得可以使用 GNU 编译器为 Java 编译 OpenJDK 代码，并用开源实现替换二进制模块。在 2008 年，Fedora 9 分发中的 `IcedTea 6` 二进制文件成功通过了 Java 技术兼容性工具包测试套件，成为完全兼容的 Java 6 实现。

IcedTea 项目的显著特征是 **引导构建**。引导模式意味着构建过程运行两次。首先，使用 **GNU 编译器为 Java**（**GCJ**）构建带有一些补丁的 OpenJDK，然后，使用新构建的二进制文件作为最终生产构建的编译器。在 IcedTea 的早期，这个过程是确保最终二进制文件中不包含（或意外泄漏）非自由二进制部分的唯一方法（来自非自由的 Sun Java 编译器）。使用另一个编译器的这种引导过程是自由软件的常见程序。除了防止二进制泄漏外，这样的程序还可以用来防止可能的恶意编译器书签。对于最近的 IcedTea 版本，构建和预构建的 OpenJDK 二进制文件不需要 GCJ，并且可以在没有完整引导过程的情况下用作引导 JDK。

IcedTea 比 OpenJDK 更开放于新和实验性功能。它允许您使用替代虚拟机实现（如 Zero、Shark 和 JamVM），同时也支持为额外的架构（如 ARM、MIPS）进行交叉编译，这些架构不受 OpenJDK 支持。

IcedTea 还提供了与操作系统的更紧密集成。最初，Sun Java 的 Linux 发行版被设计得尽可能自包含。许多标准功能（如图像处理支持（PNG、JPG、GIF）、压缩（ZIP）和加密算法（基于椭圆曲线的））的实现都包含在 OpenJDK 代码库中。这些允许更好的可移植性，但限制了使用较旧和可能不太安全的开源库的用户。现代 Linux 发行版具有先进的包管理工具，用于控制和管理包之间的常见依赖关系。IcedTea 引入了使用宿主操作系统中某些库的支持，而不是使用 OpenJDK 中包含的版本。

# 构建 IcedTea 6

与原始的 OpenJDK 6 源代码相比，IcedTea 项目在历史上经历了许多变化。变化数量最终减少，因为其中一些被包含在主要的 OpenJDK 6 源代码树中。其中一些变化，如 NIO2 API 支持（来自 OpenJDK 7），是实验性的，已被移除。

这些变化以补丁（不同的文件）的形式存储在 IcedTea 源代码树中，而不是作为单独分支中的 changesets。这可能会让不熟悉类 Unix/Linux 补丁技术和工具的用户感到困惑。不同的文件存储在 `patches` 目录中，并在 `configure` 构建步骤期间应用于 OpenJDK 源代码树。

与 OpenJDK 8 不同，GNU Autoconf 构建系统并未紧密集成到 IcedTea 构建过程中。Autoconf 的 `configure` 步骤为构建 OpenJDK 源代码和标准（略有修补）的 OpenJDK makefiles 准备环境，并在 Autoconf 的 `make` 步骤上运行。

在这个配方中，我们将使用 Ubuntu 12.04 LTS amd64 构建 IcedTea 6。

## 准备工作

对于这个配方，我们需要一个干净的 Ubuntu 12.04（服务器或桌面版本）正在运行。

## 如何操作...

以下步骤将帮助您构建 IcedTea 6：

1.  安装 OpenJDK 6 的预包装二进制文件：

    ```java
    sudo apt-get install openjdk-6-jdk

    ```

1.  安装 GCC 工具链和构建依赖项：

    ```java
    sudo apt-get build-dep openjdk-6
    sudo apt-get install libmotif-dev

    ```

1.  从 [`icedtea.wildebeest.org/download/source/`](http://icedtea.wildebeest.org/download/source/) 下载并解压适当的 `icedtea6*` 源代码 tarball：

    ```java
    wget http://icedtea.wildebeest.org/download/source/icedtea6-1.13.1.tar.xz
    tar xJvf icedtea6-1.13.1.tar.xz

    ```

1.  配置构建，指定已预装的 OpenJDK 路径并禁用系统的 LCMS2 支持：

    ```java
    ./configure \
    --with-jdk-home=/usr/lib/jvm/java-6-openjdk-amd64 \
    --disable-system-lcms

    ```

1.  开始构建：

    ```java
    make

    ```

1.  等待构建完成。

## 它是如何工作的...

需要预包装的 OpenJDK 6 二进制文件，因为一些构建步骤是使用外部 Java 运行时运行的。

`build-dep` 命令用于安装构建指定包所需的所有依赖项。由于 Ubuntu 打包的 OpenJDK 6 与官方 OpenJDK 6 非常接近，此命令将安装几乎所有所需的依赖项。

`libmotif-dev` 包是所需的附加包，包含 Motif GUI 工具包的头文件。

在配置步骤中，IcedTea 检查环境并为在构建步骤中应用的对 OpenJDK 源的调整做准备。

在构建步骤中，官方 OpenJDK 6 源被下载并按照构建配置进行调整。然后，运行通常的 OpenJDK 6 构建。然后，使用第一次构建的结果作为引导 JDK 进行第二次运行。由第二次构建产生的二进制文件。

IcedTea 的最新版本需要颜色管理库 LCMS 版本 2.5 或更高。由于 Ubuntu 12.04 仓库中没有这样的版本，我们使用与 OpenJDK 源一起捆绑的 LCMS。

## 更多...

与 OpenJDK 相比，IcedTea 具有更加灵活的配置。本章后面的食谱中将会探索一些选项。

## 参见

+   第二章，*构建 OpenJDK 6*

+   第四章中的*使用 GNU Autoconf*食谱，*构建 OpenJDK 8*

+   IcedTea 项目网站[`icedtea.classpath.org/wiki/Main_Page`](http://icedtea.classpath.org/wiki/Main_Page)

+   在 1983 年 ACM A.M. 图灵奖颁奖典礼上，肯·汤普森发表的获奖感言《信任的反思》，可在[`cm.bell-labs.com/who/ken/trust.html`](http://cm.bell-labs.com/who/ken/trust.html)找到。

# 构建 IcedTea 7

在 OpenJDK 7 的漫长开发过程中，构建过程得到了清理和显著改进。许多更改（例如用开源组件替换二进制插件）在 IcedTea 中的其他实验性更改“浸泡”之后被纳入了主要的 OpenJDK 源树中。

由于这些更改，IcedTea 7 源树与前辈相比，包含的 OpenJDK 7 补丁要少得多。类似于 IcedTea 6，Autoconf `configure` 步骤应用补丁并准备环境，而 `make` 步骤运行原始的 OpenJDK 7 makefiles。

源补丁主要用于与平台 Linux 库（`zlib`、`libpng` 等）的兼容性，而不是源树中包含的库。Makefile 补丁除了这个任务外，还用于更改 Java 启动器——版本切换输出。许多应用程序使用这些输出以确定它们运行的 JRE 版本。除了版本号格式外，一个值得注意的更改是，在 Oracle 中，Java 版本行以 `java version <version number>` 字符串开头，而 vanilla OpenJDK 版本行以 `openjdk version <version number>` 开头。虽然可以使用环境变量修改版本号，但这个前缀不能修改。为了更好地与现有应用程序兼容，IcedTea 使用 makefile 补丁将此前缀从 `openjdk` 更改为 `java`。

在这个菜谱中，我们将使用 Ubuntu 12.04 LTS 构建 IcedTea 7。

## 准备工作

对于这个菜谱，我们需要一个干净运行的 Ubuntu 12.04（服务器或桌面版本）。

## 如何操作...

以下步骤将帮助您构建 Iced Tea 7：

1.  安装预包装的 OpenJDK 6 二进制文件：

    ```java
    sudo apt-get install openjdk-6-jdk

    ```

1.  安装 GCC 工具链和构建依赖项：

    ```java
    sudo apt-get build-dep openjdk-6

    ```

1.  从 [`icedtea.wildebeest.org/download/source/`](http://icedtea.wildebeest.org/download/source/) 下载并解压缩适当的 `icedtea-2*` 源代码 tarball：

    ```java
    http://icedtea.wildebeest.org/download/source/icedtea-2.4.5.tar.xz
    tar xJvf icedtea-2.4.5.tar.xz

    ```

1.  配置构建，指定预安装的 OpenJDK 路径，并禁用系统的 LCMS2 支持：

    ```java
    ./configure \
    --with-jdk-home=/usr/lib/jvm/java-6-openjdk-amd64 \
    --disable-system-lcms

    ```

1.  开始构建：

    ```java
    make

    ```

1.  等待构建完成。

## 它是如何工作的...

需要预包装的 OpenJDK 6 二进制文件，因为一些构建步骤是使用外部 Java 运行时运行的。

使用 `build-dep` 命令安装构建指定包所需的所有依赖项。

在构建步骤中，下载官方 OpenJDK 6 源，然后根据构建配置进行调整。之后，运行通常的 OpenJDK 6 构建。然后，使用第一次构建的结果作为引导 JDK 运行第二次 OpenJDK 6 构建。由第二次构建产生的二进制文件。

IcedTea 的最新版本需要颜色管理库 LCMS 版本 2.5 或更高。由于 Ubuntu 12.04 存储库中没有这样的版本，我们使用了与 OpenJDK 源一起捆绑的 LCMS。

## 更多内容...

与 OpenJDK 相比，IcedTea 具有更灵活的配置。本章后面的菜谱中将会探索一些选项。

## 参见

+   第三章，*构建 OpenJDK 7*

+   来自 第四章 的 *使用 GNU Autoconf* 菜谱，*构建 OpenJDK 8*

+   IcedTea 项目网站在 [`icedtea.classpath.org/wiki/Main_Page`](http://icedtea.classpath.org/wiki/Main_Page)

# 使用 IcedTea 补丁构建 OpenJDK 7

与 OpenJDK 7 相比，IcedTea 7 的发布周期要短得多。IcedTea 发布大约对应于 Oracle Java 安全发布，并包含适用的相同安全补丁，因此，IcedTea 7 可以用于在 Windows 和 Mac OS X 平台上构建最新的 OpenJDK 7 构建。

在非 Linux 平台上执行完整的 IcedTea 7 构建可能会很困难，因为构建配置步骤期望 Linux 作为主机平台。相反，可以使用 IcedTea 项目在 Linux 上修复 OpenJDK 源代码，然后可以使用常规的 OpenJDK 7 构建步骤在其他平台上构建修复后的源代码。

## 准备工作

对于这个方法，我们需要一个干净的 Ubuntu 12.04（服务器或桌面版本）正在运行。

## 如何做到这一点...

以下步骤将帮助您使用 IcedTea 补丁准备 OpenJDK 7 源代码：

1.  安装预包装的 OpenJDK 6 二进制文件：

    ```java
    sudo apt-get install openjdk-6-jdk

    ```

1.  安装 GCC 工具链和构建依赖项：

    ```java
    sudo apt-get build-dep openjdk-6

    ```

1.  从 [`icedtea.wildebeest.org/download/source/`](http://icedtea.wildebeest.org/download/source/) 下载并解压缩适当的 `icedtea-2*` 源代码 tarball：

    ```java
    http://icedtea.wildebeest.org/download/source/icedtea-2.4.5.tar.xz
    tar xJvf icedtea-2.4.5.tar.xz

    ```

1.  运行配置脚本以启用使用与 OpenJDK 源代码捆绑的库：

    ```java
    ./configure \
    --with-jdk-home=path/to/openjdk7 \
    --with-rhino=path/to/rhino-1.7.4.jar \
    --disable-bootstrap \
    --disable-system-zlib \
    --disable-system-jpeg \
    --disable-system-png \
    --disable-system-gif \
    --disable-system-lcms \
    --disable-system-pcsc \
    --disable-compile-against-syscalls \
    --disable-nss
    ```

1.  开始构建过程：

    ```java
    make

    ```

1.  在构建过程中，修复后的 OpenJDK 7 源代码将被放入 `openjdk` 目录。将此目录复制到 Windows 或 Mac OS X 机器上。

1.  使用常规的 OpenJDK 7 构建过程构建修复后的源代码，并附加以下环境变量：

    ```java
    export FT2_CFLAGS='-I$(FREETYPE_HEADERS_PATH) -I$(FREETYPE_HEADERS_PATH)/freetype2'
    export DISABLE_INTREE_EC=true

    ```

## 它是如何工作的...

IcedTea 7 包含一组补丁，这些补丁在构建过程的开始时应用（取决于配置选项）。在这个方法中，我们不想构建 IcedTea 本身，但想收集修复后的 IcedTea 源代码以在另一个平台上稍后使用 OpenJDK 7 的常规构建过程构建，如 第三章 中所述，*构建 OpenJDK 7*。

## 还有更多...

补丁是在构建的早期阶段应用的。不需要等待 IcedTea 构建完成：在应用补丁后，构建过程可以被中断。

## 参见

+   本章中的 *构建 IcedTea 7* 方法

+   第三章，*构建 OpenJDK 7*

+   来自 第四章 的 *使用 GNU Autoconf* 方法，*构建 OpenJDK 8*

+   IcedTea 项目网站在 [`icedtea.classpath.org/wiki/Main_Page`](http://icedtea.classpath.org/wiki/Main_Page)

# 使用 NSS 安全提供程序构建 IcedTea 7

OpenJDK 支持用于加密服务实现的插件模块。广泛使用的现代加密算法（如加密、填充等）通常在多个平台上可用。不同的平台可能有不同的本地算法实现。这些实现可能比 OpenJDK 内部的实现性能更好，因为它们可以依赖于它们运行的平台的细节。此外，平台实现可以用作连接到硬件加密设备（如智能卡或 eTokens）的桥梁。OpenJDK 插件加密提供者允许您通过单个 Java API 接口使用广泛的加密实现。

**网络安全服务**（**NSS**）是一个开源加密库，最初是为 Netscape 网络浏览器开发的，目前由 Mozilla 基金会维护。NSS 提供了一系列加密算法的实现，支持**传输层安全性**（**TLS**）和 S/MIME，并在 Firefox 网络浏览器以及其他软件中使用。NSS 还支持硬件加密设备（智能卡/eTokens）。

IcedTea 7 可以构建为支持作为加密提供者的 NSS。

## 准备工作

对于这个配方，我们需要一个干净的 Ubuntu 12.04（服务器或桌面版本）运行。

## 如何做...

以下步骤将帮助您构建带有 NSS 的 IcedTea 7：

1.  安装 OpenJDK 6 的预包装二进制文件：

    ```java
    sudo apt-get install openjdk-6-jdk

    ```

1.  安装 GCC 工具链和构建依赖项：

    ```java
    sudo apt-get build-dep openjdk-6

    ```

1.  从[`icedtea.wildebeest.org/download/source/`](http://icedtea.wildebeest.org/download/source/)下载并解压缩适当的`icedtea-2*`源代码 tarball：

    ```java
    http://icedtea.wildebeest.org/download/source/icedtea-2.4.5.tar.xz
    tar xJvf icedtea-2.4.5.tar.xz

    ```

1.  运行配置脚本并启用 NSS 加密提供者：

    ```java
    ./configure --with-jdk-home=/usr/lib/jvm/java-6-openjdk-amd64
    --disable-system-lcms
    --enable-nss

    ```

1.  开始构建：

    ```java
    make

    ```

1.  等待构建完成。

## 它是如何工作的...

OpenJDK 支持使用统一的服务提供者接口的插件加密实现。IcedTea 7 允许您将加密任务委托给平台 NSS 库。

## 参见

+   本章的*构建 IcedTea 7*配方

+   第三章，*构建 OpenJDK 7*

+   来自第四章的*使用 GNU Autoconf*配方，*构建 OpenJDK 8*

+   Mozilla NSS 开发者文档在[`developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS`](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS)

# 使用 SystemTap 支持构建 IcedTea 6

在现代操作系统上运行的应用程序通常通过**系统调用**（**syscalls**）或通过中间平台 API 与底层操作系统内核进行通信。一些操作系统提供高级跟踪工具来监控运行中的应用程序，以及用于故障排除。这些工具（如 Oracle Solaris 上的`dtrace`）允许用户拦截所有应用程序活动、系统调用、Unix 信号处理等，以诊断性能和功能问题。

为了正确支持跟踪工具，应用程序应该在其二进制文件中编译了 *探针点*（或 *跟踪点*）。

SystemTap 是 Linux 操作系统的跟踪工具和脚本语言。IcedTea 6 可以在 SystemTap 支持下构建。

## 准备工作

对于这个脚本，我们需要一个干净安装的 Ubuntu 12.04（服务器或桌面版本）运行。

## 如何操作...

以下步骤将帮助您使用 SystemTap 构建 IcedTea 6：

1.  安装预包装的 OpenJDK 6 二进制文件：

    ```java
    sudo apt-get install openjdk-6-jdk

    ```

1.  安装 GCC 工具链和构建依赖项：

    ```java
    sudo apt-get build-dep openjdk-6
    sudo apt-get install libmotif-dev

    ```

1.  从 [`icedtea.wildebeest.org/download/source/`](http://icedtea.wildebeest.org/download/source/) 下载并解压适当的 `icedtea6*` 源代码 tarball：

    ```java
    wget http://icedtea.wildebeest.org/download/source/icedtea6-1.13.1.tar.xz
    tar xJvf icedtea6-1.13.1.tar.xz

    ```

1.  配置构建，指定预安装的 OpenJDK 路径并禁用系统的 LCMS2 支持：

    ```java
    ./configure \
    --with-jdk-home=/usr/lib/jvm/java-6-openjdk-amd64 \
    --disable-system-lcms
    --enable-systemtap

    ```

1.  开始构建：

    ```java
    make

    ```

1.  等待构建完成。

## 还有更多...

我们在 JVM 用户空间标记的支持下构建了 IcedTea 6。这些标记允许 SystemTap 探测各种 JVM 事件，包括类的加载、方法的即时翻译和垃圾回收。

## 参见

+   本章的 *构建 IcedTea 6* 脚本

+   第二章，*构建 OpenJDK 6*

+   第四章 的 *使用 GNU Autoconf* 脚本

+   SystemTap 项目网站在 [`sourceware.org/systemtap/`](https://sourceware.org/systemtap/)
