# 第三章. 构建 OpenJDK 7

在本章中，我们将介绍：

+   在 Ubuntu Linux 12.04 LTS 上构建 OpenJDK 7

+   在 Mac OS X 上构建 OpenJDK 7

+   在 Windows 上构建 32 位 FreeType 库用于 OpenJDK 7

+   在 Windows 上构建 64 位 FreeType 库用于 OpenJDK 7

+   在 Windows 7 SP1 上构建 32 位 OpenJDK 7

+   在 Windows 7 x64 SP1 上构建 64 位 OpenJDK 7

+   准备用于 32 位和 64 位 Windows 构建的独立工具链

# 简介

OpenJDK 7 是 Java 平台标准版 7 的免费和开源实现。在撰写本书时，它是准备用于生产使用的最新 OpenJDK 版本。

最初，为 OpenJDK 7 计划了许多高级变更，例如模块化虚拟机和 lambda 表达式支持，但由于各种技术和组织原因，在 Sun Microsystems 被甲骨文公司收购后，大多数新特性都被推迟到 OpenJDK 的下一个版本。这加快了发布速度。它于 2011 年 7 月 28 日达到 **通用可用** 状态，并且是第一个作为 Java 平台参考实现的 OpenJDK 版本。

OpenJDK 7 的主要更新编号为更新 2、更新 4 和更新 6，在接下来的年份中发布。之后，版本编号发生了变化，下一个更新 40（在撰写本书时是最新版本）于 2013 年 9 月发布。下一个计划发布的更新 60 预计在 2014 年 5 月发布，而 OpenJDK 7 的生命周期将在 2015 年初的更新 80 中结束。

OpenJDK 7 的发布周期与 Oracle Java 的发布周期不同。Oracle Java 更新是为了常规的安全相关更改和 OpenJDK 更新。Oracle 的安全更改被传播到 OpenJDK 7（以及适用的情况下 OpenJDK 6），但 OpenJDK 并不是立即发布。相反，发布是在主要变更和累积安全变更上进行的，通常包含更新版本的 HotSpot 虚拟机。

OpenJDK 7 支持在 Linux、Windows、Mac OS X 和 Solaris 操作系统上运行。以下将进一步讨论 Windows、Linux 和 Mac OS X 版本。对于 Linux 和 Windows 操作系统，支持 x86 和 x86_64 架构。对于 Mac OS X，仅支持 x86_64。为了符合 OpenJDK 术语，将使用 **i586** 术语表示 x86 架构，而 **amd64** 将用于 x86_64。OpenJDK 7 不支持交叉编译，因此必须使用 i586 操作系统来构建 i586 版本，对于 amd64 也是如此。对于 Linux 版本，这两个架构的构建过程几乎相同，但对于 Windows 版本则差异很大。

在本章中，我们将使用来自官方 OpenJDK 7 更新 40 tarball 的源代码。

# 在 Ubuntu Linux 12.04 LTS 上构建 OpenJDK 7

此配方与第二章 构建 OpenJDK 6 on Ubuntu Linux 12.04 LTS 中的配方 *Building OpenJDK 6 on Ubuntu Linux 12.04 LTS* 相似。

OpenJDK 的构建过程严重依赖于类 Unix 开发工具。基于 Linux 的操作系统通常对这些工具提供顶级支持，因此，在 Linux（以及 Mac OS X）上构建 OpenJDK 可能比在 Windows 上简单。对于像 Fedora 或 Ubuntu 这样的主要发行版，构建工具链和所有依赖项已经作为软件包包含在发行版中，并且可以轻松安装。

选择 Ubuntu 12.04 LTS 作为本书的操作系统，因为它是最受欢迎的 Linux 发行版之一。对于运行其他操作系统的读者，可以在网上找到 Ubuntu 12.04 虚拟镜像，用于最流行的虚拟化工具，如 Oracle VirtualBox 或 VMware。

要为 i586 和 amd64 架构构建二进制文件，应使用相应的 Ubuntu 版本。对于这两个架构，构建说明是相同的，因此在此配方中不会进一步提及。

## 准备工作

对于这个配方，我们需要一个干净的 Ubuntu 12.04（服务器或桌面版本）运行。

## 如何操作...

以下说明将帮助我们构建 OpenJDK 7：

1.  安装预包装的 OpenJDK 6 二进制文件：

    ```java
    sudo apt-get install openjdk-6-jdk

    ```

1.  安装 GCC `工具链` 和构建依赖项：

    ```java
    sudo apt-get build-dep openjdk-6

    ```

1.  下载并解压官方 OpenJDK 7 更新 40 存档（结果将放置在 `openjdk` 目录中）：

    ```java
    wget http://download.java.net/openjdk/jdk7u40/promoted/b43/openjdk-7u40-fcs-src-b43-26_aug_2013.zip
    unzip -q openjdk-7u40-fcs-src-b43-26_aug_2013.zip

    ```

1.  创建一个包含以下环境设置的新的文本文件 `buildenv.sh`：

    ```java
    export LD_LIBRARY_PATH=
    export CLASSPATH=
    export JAVA_HOME=
    export LANG=C
    export ALT_BOOTDIR=/usr/lib/jvm/java-6-openjdk
    ```

1.  将环境变量导入当前 shell 会话中（注意在它前面有一个点和空格）：

    ```java
    . buildenv.sh
    ```

1.  从 `openjdk` 目录开始构建过程：

    ```java
    make 2>&1 | tee make.log
    ```

1.  等待构建完成，然后尝试运行新构建的二进制文件：

    ```java
    cd build/linux-amd64/j2sdk-image/
    ./bin/java –version
    openjdk version "1.7.0-internal"
    OpenJDK Runtime Environment (build 1.7.0-internal-ubuntu_2014_02_08_08_56-b00)
    OpenJDK 64-Bit Server VM (build 24.0-b56, mixed mode)

    ```

## 它是如何工作的...

需要预包装的 OpenJDK 6 二进制文件，因为一些构建步骤是使用外部 Java 运行时运行的。

使用 `build-dep` 命令安装构建指定包所需的所有依赖项。由于 Ubuntu 打包的 OpenJDK 6 与官方 OpenJDK 6 非常接近，此命令将安装几乎所有所需的依赖项。

在 amd64 平台上成功构建后，JDK 文件将放置在 `build/linux-amd64/j2sdk-image` 目录中，JRE 文件将放置在 `build/linux-amd64/j2re-image` 目录中。在 i586 平台上，将使用 `build/linux-i586` 路径。一个包含没有演示和示例的 JDK 的 Server JRE 额外包将放置在 `j2sdk-server-image` 目录中。

## 还有更多...

Javadoc 生成需要花费很多时间，是构建过程中最消耗内存的步骤。可以使用额外的环境变量跳过：

```java
export NO_DOCS=true
```

与之前的版本不同，OpenJDK 7 支持并行（多核）本地库编译。以下环境变量可以分别用于 `jdk` 和 `hotspot` 模块：

```java
PARALLEL_COMPILE_JOBS=N
HOTSPOT_BUILD_JOBS=N
```

此构建已生成里程碑标签和构建号 `b00`。可以使用额外的环境变量设置预定义的构建号和里程碑：

```java
export MILESTONE=ubuntu-build
export BUILD_NUMBER=b30
```

在构建过程中，可以使用额外的环境变量提供 `cacerts` 文件：

```java
export ALT_CACERTS_FILE=path/to/cacerts
```

对于预安装的 Java（由变量`ALT_BOOTDIR`提供），amd64 构建可以是 amd64 或 i586 构建。i586 二进制文件消耗更少的内存，并且可以在硬件有限的情况下用于 amd64 构建。

OpenJDK 7 在 Linux 上的最小构建要求与上一版本相同，因此可以使用 Debian 5.0 Lenny 构建最兼容的版本。

## 参见

+   来自第二章的*准备 CA 证书*配方，*构建 OpenJDK 6*

+   来自第二章的*在 Ubuntu Linux 12.04 LTS 上构建 OpenJDK 6*配方，*构建 OpenJDK 6*

+   来自第二章的*为最兼容的 Linux 构建设置最小构建环境*配方，*构建 OpenJDK 6*

+   OpenJDK 7 的官方构建说明可在[`hg.openjdk.java.net/jdk7u/jdk7u/raw-file/tip/README-builds.html`](http://hg.openjdk.java.net/jdk7u/jdk7u/raw-file/tip/README-builds.html)找到

# 在 Mac OS X 上构建 OpenJDK 7

OpenJDK 7 支持 Mac OS X 平台作为*一等公民*，使用适当的`toolchain`版本构建它几乎和 Linux 上一样简单。

从历史上看，Java 在 Mac OS X 上得到了一等支持。JDK 基于 Sun 代码库，但由 Apple 构建并完全集成到其操作系统环境中。直到 Mac OS X 10.4 Tiger，使用标准 Swing 工具包编写的图形用户界面应用程序可以访问大多数 Cocoa 原生界面功能。用 Java 编写的应用程序非常接近原生应用程序，同时仍然是跨平台的。

但随着下一版本的发布，Java 的支持水平下降了。从 Mac OS X 10.5 Leopard 开始，新的 Cocoa 功能不再支持 Java。Apple Java 6 的发布被推迟（与其他平台相比），超过了一年。Java 6 于 2006 年 12 月发布，但直到 2008 年 4 月才对 Mac OS X 用户可用。最终在 2010 年 10 月，Apple 官方宣布停止 Java 支持。Apple Java 6 仍在更新安全更新，并且可以安装在 Mac OS X 10.9 Mavericks（当时最新的版本）上，但 Apple 不会发布未来的 Java 版本。

第三方开源 Java 发行版在 Mac OS X 上存在。最著名的是 SoyLatte——FreeBSD Java 1.6 补丁集基于 X11 的 Mac OS X Intel 机器的移植。SoyLatte 早于 OpenJDK，在 Java 研究许可下授权，并支持 Java 6 构建。现在它是 OpenJDK BSD-Port 项目的一部分。

目前，Mac OS X 的官方最新稳定 Java 版本是 Oracle Java 7，它与 OpenJDK 非常接近。在此配方中，我们将在 Mac OS X 10.7.5 Lion 上构建 OpenJDK 7 更新 40。选择此操作系统版本是因为 10.7.3 是官方最低构建要求平台，应该提供最兼容的二进制文件，而 10.7.5 与它非常接近，但可能在较新的 Intel Ivy Bridge 处理器上运行，并且也可能使用流行的虚拟化工具（如 Oracle VirtualBox 或 VMware）相对容易地虚拟化。

## 准备工作

对于这个配方，我们需要一个干净运行的 Mac OS X 10.7.5 Lion。

## 如何操作...

以下步骤将帮助我们构建 OpenJDK 7：

1.  从 [`developer.apple.com/xcode/`](https://developer.apple.com/xcode/) 下载 Xcode 3.4.2 for Lion（2012 年 3 月 22 日）并安装它（需要 Apple 开发者账户，注册免费）。

1.  使用之前提到的相同下载链接下载 Xcode 命令行工具—2012 年 3 月晚些时候（2012 年 3 月 21 日）并安装它。

1.  从终端运行此命令以设置命令行工具：

    ```java
    sudo xcode-select -switch /Applications/Xcode.app/Contents/Developer/

    ```

1.  导航到 **应用程序** | **实用工具** 并运行 **X11.app** 以将其作为附加下载安装。

1.  安装 JDK 7—Oracle 发行版，或者可以使用预构建的 OpenJDK 二进制文件。

1.  从 [`archive.apache.org/dist/ant/binaries/`](http://archive.apache.org/dist/ant/binaries/) 下载 Apache Ant 1.8.4 并解压它。

1.  从 [`download.java.net/openjdk/jdk7u40/promoted/b43/`](http://download.java.net/openjdk/jdk7u40/promoted/b43/) 下载源存档 `openjdk-7u40-fcs-src-b43-26_aug_2013.zip` 并解压它。

1.  创建一个新文本文件 `buildenv.sh` 并包含以下环境设置：

    ```java
    export LD_LIBRARY_PATH=
    export CLASSPATH=
    export JAVA_HOME=
    export LANG=C
    export ANT_HOME=path/to/ant
    export PATH=path/to/ant/bin:$PATH
    export ALT_BOOTDIR=path/to/jdk7
    ```

1.  将环境导入当前终端会话中（注意它前面有一个点和空格）：

    ```java
    . buildenv.sh
    ```

1.  从 `openjdk` 目录开始构建过程：

    ```java
    make 2>&1 | tee make.log
    ```

1.  等待构建完成，然后尝试运行新构建的二进制文件：

    ```java
    cd build/macosx-x86_64/j2sdk-image/
    ./bin/java –version
    openjdk version "1.7.0-internal"
    OpenJDK Runtime Environment (build 1.7.0-internal-obf_2014_02_07_21_32-b00)
    OpenJDK 64-Bit Server VM (build 24.0-b56, mixed mode)
    ```

## 它是如何工作的...

除了用于原生源主要部分的 Xcode 命令行工具外，还需要 Xcode 本身来构建特定平台的代码。

OpenJDK 在 Mac OS X 上正在远离使用 X11 服务器，但版本 7 构建仍然需要它。Mac OS X 10.7 狮子版预装了 X11，只需运行一次即可为构建进行配置。

可以使用 Apple JDK6 而不是 OpenJDK7，但需要额外的配置。

Apache Ant 构建工具对于构建的一些模块是必需的。

## 还有更多...

此配方使用官方 Apple GCC（和 G++）版本 4.2 构建。在此版本之后，由于许可原因，官方 Apple 对 GCC 的支持已停止。Clang—最初由 Apple 开发的开源编译器—是较新版本 Mac OS X 的默认和首选编译器。虽然较新版本的 OpenJDK 在 Mac OS X 上支持 Clang，但版本 7 仍然需要 GCC。

使用相同的步骤，可以在 Mac OS X 10.8 Mountain Lion 上构建 OpenJDK 7。唯一的额外要求是，应该单独安装 X11 服务器，使用 XQuartz 项目。

Xcode 的新版本和 Mac OS X 10.9 Mavericks 的最新更新可能会破坏 OpenJDK 构建。如果希望使用较新的操作系统/`工具链`进行构建，最好检查 OpenJDK 邮件列表上的当前构建情况和提出的解决方案。

## 参见

+   来自第二章的*准备 CA 证书*配方，*构建 OpenJDK 6*

+   来自第二章的*在 Ubuntu Linux 12.04 LTS 上构建 OpenJDK 6*配方，*构建 OpenJDK 6*，获取有关构建调优的信息

+   在 OpenJDK Wikipedia 上关于 Mac OS X 的官方构建说明，请参阅[`wikis.oracle.com/display/OpenJ`](https://wikis.oracle.com/display/OpenJ)

# 在 Windows 上为 OpenJDK 7 构建 32 位 FreeType 库

现代软件中使用的多数字体都编码为矢量格式，以支持适当的缩放。矢量字体有多个标准，例如唐纳德·E·克努特教授的 Metafont，Adobe 的 Type1，Apple 和微软的 TrueType，以及 Adobe 和微软的 OpenType。

矢量字体的光栅化是一个非常复杂的任务，大多数桌面软件（如网页浏览器或文本处理器）都使用第三方库来处理字体。

Sun Microsystems 为 Sun Java 实现许可了一个第三方封闭源代码字体库。这个库的源代码不能与 OpenJDK 的初始版本一起公开发布。在 OpenJDK 的早期，启动了 Font Scaler Replacement 项目，以采用开源字体库。

FreeType 是一个免费且开源（在宽松许可下）的字体光栅化库。它在开源桌面软件中得到广泛应用。OpenJDK 团队选择了 FreeType 作为封闭源代码字体库的替代品，并且现在在所有支持的平台上使用 OpenJDK。在 Windows 上构建 OpenJDK 需要预构建的静态和动态 FreeType 库。

可以使用与 i586 和 amd64 OpenJDK 构建相同的 Microsoft Windows SDK for Windows 7 和 .NET Framework 4（版本 7.1）来为 OpenJDK 7 构建 FreeType。我们将使用免费提供的 Visual Studio 2010 Express Edition 来配置 FreeType 项目的构建设置。

## 准备工作

对于这个配方，我们应该有一个运行 Windows 7 SP1 i586 的系统。

## 如何操作...

以下步骤将帮助我们构建 FreeType：

1.  从微软网站下载 Microsoft .NET Framework 4 并安装它。

1.  从微软网站下载 Microsoft Windows SDK for Windows 7 和 .NET Framework 4（版本 7.1），并将其安装到默认位置，镜像文件名为 `GRMSDK_EN_DVD.iso`。

1.  从 Microsoft 网站下载 Visual Studio 2010 Express Edition 并安装到默认位置。只需要 Visual C++组件。

1.  从[`freetype.org/`](http://freetype.org/)下载 FreeType 2.5.2 源代码 tar 包并解压。

1.  打开`include\config\ftoption.h`文件并取消注释第 95 行：

    ```java
    #define FT_CONFIG_OPTION_SUBPIXEL_RENDERING
    ```

1.  将第 269 行和第 271 行的`/* #define FT_EXPORT(x) extern x */`和`/* #define FT_EXPORT_DEF(x) x */`改为`#define FT_EXPORT(x) __declspec(dllexport) x`和`#define FT_EXPORT_DEF(x) __declspec(dllexport) x`。

1.  在 Visual Studio 中打开`solutions\windows\vc2010\freetype.sln`解决方案。

1.  在主菜单中，转到**项目** | **属性** | **配置属性**，在**平台工具集**字段中选择 Windows7.1 SDK。

1.  在主屏幕上选择**释放多线程**作为**解决方案配置**。

1.  运行构建，`freetype252MT.lib`库将被放置到`freetype\objs\win32\vc2010`目录中；将其重命名为`freetype.lib`，并保存以供以后使用。

1.  在主菜单中，转到**项目** | **属性** | **配置属性**，将**配置类型**更改为**动态库 (.dll**)，并构建解决方案。`freetype252MT.dll`和`freetype252MT.exp`文件将被放置到`objs\release_mt`目录中。将这些文件重命名为`freetype.dll`和`freetype.exp`，并在 OpenJDK 构建期间与之前生成的`freetype.lib`一起使用。

## 它是如何工作的...

使用 Visual Studio 的自身工具集可能构建 i586 版本的 FreeType，但我们使用了 Windows SDK7.1 工具集以确保与使用相同工具集的 OpenJDK 构建的兼容性。

`FT_CONFIG_OPTION_SUBPIXEL_RENDERING`宏在 FreeType 实现中启用子像素渲染功能。

`FT_EXPORT`和`FT_EXPORT_DEF`宏应根据当前平台的调用约定进行调整。我们将它们更改为使用 Windows 特定的调用约定。

## 参考信息

+   *在 Windows 上为 OpenJDK 7 构建 64 位 FreeType 库*配方

+   FreeType 官方网站[`freetype.org/`](http://freetype.org/)

+   教授唐纳德·E·克努斯关于 Metafont 和 TrueType 的访谈[`www.advogato.org/article/28.html`](http://www.advogato.org/article/28.html)

+   *OpenJDK: 字体缩放器替换项目*页面[`openjdk.java.net/projects/font-scaler/`](http://openjdk.java.net/projects/font-scaler/)

# 在 Windows 上为 OpenJDK 7 构建 64 位 FreeType 库

Windows amd64 版本的 FreeType 构建与之前配方中的 i586 构建类似。本配方中只将写出不同的步骤。请参阅之前的配方，*在 Windows 上为 OpenJDK 7 构建 32 位 FreeType 库*，以获取更详细的说明。

## 准备工作

对于这个配方，我们应该有 Windows 7 SP1 amd64 运行。

## 如何操作...

以下步骤将帮助我们构建 FreeType：

1.  从微软网站下载 Microsoft .NET Framework 4（版本 7.1）、Microsoft Windows SDK for Windows 7 和 Visual Studio 2010 Express Edition，并将它们安装到默认位置。应使用`GRMSDKX_EN_DVD.iso`文件中的 SDK 的 amd64 版本。

1.  按照前一个菜谱中的步骤 4 到 9 下载和调整 FreeType 源代码，并在 Visual Studio 中配置项目。

1.  在主屏幕上选择**x64**作为**解决方案平台**。

1.  按照前一个菜谱中的步骤 10 和 11 进行操作。库将被放置在`freetype\builds\windows\vc2010\x64\Release Multithreaded`目录中。

## 它是如何工作的...

使用 Visual Studio 2010 的 Express Edition 无法构建 FreeType amd64。应该使用专业版或 Windows SDK 工具集。由于我们将使用 Windows SDK 7.1 进行 OpenJDK 构建，因此我们也将其用于适当的 FreeType 构建。

## 参见

+   *在 Windows 上为 OpenJDK 7 构建 32 位 FreeType 库*的菜谱

# 在 Windows 7 SP1 上构建 32 位 OpenJDK 7

与版本 6 相比，Windows 平台上的 OpenJDK 7 构建过程进行了重大改进。尽管如此，构建环境设置仍然比 Linux 和 Mac OS X 复杂得多。构建的大部分复杂性来自于它通过 Cygwin 工具使用类似 Unix 的构建环境。

i586 构建的官方编译器要求是 Microsoft Visual Studio C++ 2010 专业版。Visual Studio 2010 的 Express Edition 也可以用于 i586 构建。虽然这个版本是免费的（就像“免费啤酒”一样），但它有一个 30 天的评估期，之后需要注册。虽然注册也是免费的，但这可能对某些使用场景（例如，自动构建服务）造成问题。

我们将使用 Microsoft Windows SDK Version 7.1 for Windows 7，而不是 Visual Studio 2010。此 SDK 也可从微软网站免费获取，并且无需注册即可使用。它使用与 Visual Studio 2010 Express 相同的编译器。它仅包含命令行工具（没有 GUI），但如果需要 GUI，也可以作为 Visual Studio 2010 的外部工具集使用。

## 准备工作

对于这个菜谱，我们应该有一个没有安装任何杀毒软件的 Windows 7 SP1 i586 系统运行。不允许使用杀毒软件，因为它可能会干扰 Cygwin 运行时。

## 如何操作...

以下步骤将帮助我们构建 OpenJDK：

1.  从微软网站下载 Microsoft .NET Framework 4 并安装。

1.  从微软网站下载 Microsoft Windows SDK for Windows 7 并安装到默认位置。.NET 开发工具和通用工具组件不是必需的。

1.  将 Microsoft DirectX 9.0 SDK（夏季 2004 版）下载并安装到默认安装路径。请注意，这个版本现在不再可在微软网站上找到。然而，它可能可以从其他地方下载。文件详情如下：

    ```java
    name: dxsdk_sum2004.exe
    size: 239008008 bytes
    sha1sum: 73d875b97591f48707c38ec0dbc63982ff45c661
    ```

1.  安装或复制预安装的 Cygwin 版本到 `c:\cygwin`。

1.  创建 `c:\path_prepend` 目录，并将 Cygwin 安装中的 `find.exe` 和 `sort.exe` 文件复制到其中。

1.  从 [`www.cmake.org/`](http://www.cmake.org/) 下载 GNU make 工具的二进制文件，使用 [`www.cmake.org/files/cygwin/make.exe-cygwin1.7`](http://www.cmake.org/files/cygwin/make.exe-cygwin1.7)，将其重命名为 `make.exe`，并将其放入 `c:\make` 目录。

1.  从 [`apache.org/`](http://apache.org/) 下载 Apache Ant 版本 1.8.4 的 zip 分发版，并将其解压缩到 `c:\ant` 目录。

1.  从 `openjdk-unofficial-builds GitHub` 项目（目录 `7_32`）下载预构建的 FreeType 库，并将二进制文件放入 `c:\freetype\lib` 目录，将头文件放入 `c:\freetype\include` 目录。

1.  将 OpenJDK 6 二进制文件或 Oracle Java 6 安装到 `c:\jdk6`。

1.  从 [`download.java.net/openjdk/jdk7u40/promoted/b43/`](http://download.java.net/openjdk/jdk7u40/promoted/b43/) 网页下载官方 OpenJDK 7 更新 40 源存档，并将其解压缩到 `c:\sources` 目录。

1.  创建 `build.bat` 批处理文件并写入以下环境变量设置：

    ```java
    @echo off
    rem clear variables
    set LD_LIBRARY_PATH=
    set CLASSPATH=
    set JAVA_HOME=
    rem ALT_* variables
    set ALT_BOOTDIR=C:/jdk7
    set ALT_FREETYPE_LIB_PATH=C:/freetype/lib
    set ALT_FREETYPE_HEADERS_PATH=C:/freetype/include
    set NO_DOCS=true
    rem set compiler environment manually
    set WINDOWSSDKDIR=C:/Program Files/Microsoft SDKs/Windows/v7.1/
    set VSDIR=C:/Program Files/Microsoft Visual Studio 10.0/
    set VS100COMNTOOLS=%VSDIR%/Common7/Tools
    set Configuration=Release
    set WindowsSDKVersionOverride=v7.1
    set ToolsVersion=4.0
    set TARGET_CPU=x86
    set CURRENT_CPU=x86
    set PlatformToolset=Windows7.1SDK
    set TARGET_PLATFORM=WIN7
    set LIB=%VSDIR%/VC/Lib;%WINDOWSSDKDIR%/Lib
    set LIBPATH=%VSDIR%/VC/Lib
    rem set path
    set PATH=C:/path_prepend;%VSDIR%/Common7/IDE;%VS100COMNTOOLS%;%VSDIR%/VC/Bin;%VSDIR%/VC/Bin/VCPackages;%WINDOWSSDKDIR%/Bin;C:/WINDOWS/system32;C:/WINDOWS;C:/WINDOWS/System32/Wbem;C:/make;C:/cygwin/bin;C:/jdk7/bin;C:/ant/bin
    set INCLUDE=%VSDIR%/VC/INCLUDE;%WINDOWSSDKDIR%/INCLUDE;%WINDOWSSDKDIR%/INCLUDE/gl;
    bash
    echo Press any key to close window ...
    pause > nul
    ```

1.  从 Windows 资源管理器运行 `build.bat`。应该会出现一个带有 bash 启动的 `cmd.exe` 窗口。

1.  从 bash 命令提示符运行以下命令：

    ```java
    cd /cygdrive/c/sources
    chmod –R 777 .
    make > make.log 2>&1
    ```

1.  启动另一个 Cygwin 控制台并运行以下命令：

    ```java
    cd /cygdrive/c/sources
    tail -f make.log
    ```

1.  等待构建完成。

## 它是如何工作的...

Cygwin 安装在 第二章 的 *为 Windows 构建安装 Cygwin* 配方中介绍，*构建 OpenJDK 6*。

为了简洁起见，这里使用磁盘 C 的根目录。通常，任意路径由 ASCII 字母或数字组成，且不能使用空格。

也可以使用 DirectX SDK 的新版本。

不同的 GNU make 版本在 Windows 上可能存在不同的问题。这个来自 cmake 项目的特定版本已在不同的 Windows 版本上进行了测试，并且运行良好。

此配方使用来自 `openjdk-unofficial-builds GitHub` 项目的预构建 FreeType 2.4.10 库。可以使用相同的 Windows SDK 7.1 工具链从源代码构建 FreeType。请参阅本章中的 *在 Windows 上为 OpenJDK 7 构建 32 位 FreeType 库* 配方。

在环境设置中，应特别注意 `PATH` 变量内容的顺序。排序和查找 Cygwin 实用程序应放在 `PATH` 变量的开头，以避免被具有相同名称但功能不同的 Windows 实用程序所掩盖。Make 应在 Cygwin 之前，以避免被 Cygwin 安装中可能包含的另一个版本的 make 所掩盖。

需要 `chmod 777` 命令来修复可能引起构建后期阶段错误的 Cygwin 文件权限。

make 输出将被重定向到 `make.log` 文件。`2>&1` 语句确保 `stdout` 和 `stderr` 都将被重定向。

`tail -f`命令允许我们在构建过程中监视`make.log`文件的内容。

在批处理文件的末尾添加了`pause > nul`命令，以防止在运行时错误的情况下`cmd.exe`窗口消失。

## 还有更多...

为了构建最兼容的二进制文件，应使用相同的配方，但应使用 Windows XP 操作系统而不是 Windows 7。

在 Windows XP 中，不需要`chmod 777`命令。

可以使用`tee`命令代替`>`和`tail`，将构建日志同时写入文件和控制台。

可以使用 Windows SDK 的`SetEnv.Cmd`脚本（使用适当的标志）来设置编译器环境，而不是手动设置变量。

可以使用 Visual Studio 2010 Express 或专业版代替 Windows SDK 7.1。

可以使用预构建的 OpenJDK 7 或 Oracle Java 7 作为启动 JDK，而不是 6。

## 参见

+   来自第二章的*在 Windows 7 SP1 上构建 32 位 OpenJDK 6*的配方，*构建 OpenJDK 6*

+   来自第二章的*为 Windows 构建安装 Cygwin*的配方，*构建 OpenJDK 6*

+   来自第二章的*在 Ubuntu Linux 12.04 LTS 上构建 OpenJDK 6*的配方，*构建 OpenJDK 6*，有关构建调优的信息

+   本章中的*在 Windows 上为 OpenJDK 7 构建 32 位 FreeType 库*的配方

+   本章中的*准备 CA 证书*的配方

+   OpenJDK 7 的官方构建说明，请参阅[`hg.openjdk.java.net/jdk7u/jdk7u/raw-file/tip/README-builds.html`](http://hg.openjdk.java.net/jdk7u/jdk7u/raw-file/tip/README-builds.html)

# 在 Windows 7 x64 SP1 上构建 64 位 OpenJDK 7

Windows 7 上的 amd64 构建类似于 i586 构建，但有一些额外的复杂性。

由于 Cygwin（至少是更常见的 i586 版本）在 amd64 Windows 上运行得非常糟糕。由于地址空间大小大得多，Cygwin 的 fork 技术运行得慢得多，并且可靠性较低。

Visual Studio 2010 Express Edition 不支持 amd64 架构，因此应使用 Windows 7 的 Microsoft Windows SDK 版本 7.1 或 Visual Studio 的专业版。

## 准备工作

对于这个配方，我们应该有一个没有安装防病毒软件的 Windows 7 SP1 amd64 系统运行。不允许使用防病毒软件，因为它可能会干扰 Cygwin 运行时。

## 如何操作...

以下步骤将帮助我们构建 OpenJDK：

1.  按照前一个配方，*在 Windows 7 SP1 上构建 32 位 OpenJDK 7*，使用 Windows SDK 的 amd64 版本和`7_64`目录中的 FreeType 库，执行步骤 1 到 10。

1.  创建一个`build.bat`批处理文件，并在其中写入以下环境变量设置：

    ```java
    @echo off
    rem clear variables
    set LD_LIBRARY_PATH=
    set CLASSPATH=
    set JAVA_HOME=
    rem ALT_* variables
    set ALT_BOOTDIR=C:/jdk7
    set ALT_FREETYPE_LIB_PATH=C:/freetype/lib
    set ALT_FREETYPE_HEADERS_PATH=C:/freetype/include
    set NO_DOCS=true
    rem set compiler environment manually
    set WINDOWSSDKDIR=C:/Program Files/Microsoft SDKs/Windows/v7.1/
    set VSDIR=C:/Program Files (x86)/Microsoft Visual Studio 10.0/
    set VS100COMNTOOLS=%VSDIR%/Common7/Tools
    set Configuration=Release
    set WindowsSDKVersionOverride=v7.1
    set ToolsVersion=4.0
    set TARGET_CPU=x64
    set CURRENT_CPU=x64
    set PlatformToolset=Windows7.1SDK
    set TARGET_PLATFORM=WIN7
    set LIB=%VSDIR%/VC/Lib/amd64;%WINDOWSSDKDIR%/Lib/x64
    set LIBPATH=%VSDIR%/VC/Lib/amd64
    rem set path
    set PATH=C:/path_prepend;%VSDIR%/Common7/IDE;%VS100COMNTOOLS%;%VSDIR%/VC/Bin/x86_amd64;%VSDIR%/VC/Bin;%VSDIR%/VC/Bin/VCPackages;%WINDOWSSDKDIR%/Bin;C:/WINDOWS/system32;C:/WINDOWS;C:/WINDOWS/System32/Wbem;C:/make;C:/cygwin/bin;C:/jdk7/bin;C:/ant/bin
    set INCLUDE=%VSDIR%/VC/INCLUDE;%WINDOWSSDKDIR%/INCLUDE;%WINDOWSSDKDIR%/INCLUDE/gl;
    bash
    echo Press any key to close window ...
    pause > nul
    ```

1.  按照前一个配方中的步骤 12 到 15 进行操作。

## 它是如何工作的...

Windows 7 上的 amd64 构建类似于 i586 构建，但由于 Cygwin 在 64 位操作系统上运行较慢，因此可能会慢得多。

可以使用 i586 或 amd64 引导 JDK 进行构建，唯一的区别是 amd64 需要更多的内存（不少于 1024MB）。

对于更多详细信息，请参阅前一个配方*在 Windows 7 SP1 上构建 32 位 OpenJDK 7*中的*它是如何工作的...*部分。

## 还有更多...

要构建最兼容的二进制文件，应使用相同的配方，但应使用 Windows 2003 Server amd64 操作系统代替 Windows 7。

在 Windows 2003 中，不需要`chmod 777`命令。

可以使用预构建的 OpenJDK 7 或 Oracle Java 7 作为引导 JDK，而不是 6。

## 参见

+   本章中的*在 Windows 7 SP1 上构建 32 位 OpenJDK 7*配方

+   第二章中的*在 Windows 7 x64 SP1 上构建 64 位 OpenJDK 6*配方

+   第二章中的*为 Windows 构建安装 Cygwin*配方

+   第二章中的*在 Ubuntu Linux 12.04 LTS 上构建 OpenJDK 6*配方，有关构建调优信息

+   本章中的*在 Windows 上为 OpenJDK 7 构建 64 位 FreeType 库*配方

+   第二章中的*准备 CA 证书*配方

+   在[`hg.openjdk.java.net/jdk7u/jdk7u/raw-file/tip/README-builds.html`](http://hg.openjdk.java.net/jdk7u/jdk7u/raw-file/tip/README-builds.html)查看 OpenJDK 7 的官方构建说明

# 为 32 位和 64 位 Windows 构建准备独立的工具链

在本章前面的配方中，我们使用 Windows SDK 版本 7.1 为 Windows 构建了 OpenJDK 7。此 SDK 在使用之前需要安装。安装过程需要.NET Framework 2（用于运行包含在 Windows 7 中的安装程序）和.NET Framework 4。在某些使用场景中，安装这些组件可能非常耗时。例如，为了自动化构建，可能希望使用完全干净的 Windows 镜像进行构建。除了速度慢之外，.NET Framework 和 SDK 安装程序是图形工具，可能难以用于自动安装脚本。

在此配方中，我们将创建一组文件和一个环境脚本，可用于在完全干净的 Windows 安装（具有相应的架构）上构建 OpenJDK 7 i586 和 amd64，而无需通过 GUI 安装程序安装任何工具。这样的一组文件可以放在版本控制系统下，以便在构建之前检出。

## 准备工作

对于此配方，我们应该有两个 Windows 7 SP1 的干净镜像。其中一个应该具有 i586 架构，另一个应该具有 amd64 架构。

## 如何操作...

以下步骤将帮助我们准备一个独立的工具链：

1.  从 Microsoft 网站下载 Windows 7 i586 的 Microsoft Windows SDK (`GRMSDK_EN_DVD.iso`) 并在默认位置安装到 i586 Windows 实例。不需要 `.NET Development` 和 `Common Utilities` 组件。

1.  创建一个 `toolchain` 目录。我们将把各种工具和库放在那里，并将此目录称为 `<toolchain>`。

1.  将 SDK 文件从 `C:\Program Files\Microsoft SDKs\Windows\v7.1` 复制到 `<toolchain>\winsdk71\sdk` 目录。

1.  将与 SDK 一起提供的 Visual Studio 文件从 `C:\Program Files\Microsoft Visual Studio 10.0` 复制到 `<toolchain>\winsdk71\vs2010e` 目录。

1.  从 Microsoft 网站下载 Windows 7 amd64 的 Microsoft Windows SDK (`GRMSDKX_EN_DVD.iso`) 并在默认位置安装到 amd64 Windows 实例。不需要 `.NET Development` 和 `Common Utilities` 组件。

1.  将 SDK amd64 安装的 `C:\Program Files\Microsoft SDKs\Windows\v7.1\Bin\x64` 目录复制到 `<toolchain>\winsdk71\sdk\Bin\x64` 目录。

1.  从 Microsoft 网站下载并安装 Microsoft DirectX 9.0 SDK（夏季 2004 版）到 i586 Windows 实例的默认安装路径。请注意，此分发在 Microsoft 网站上不再可用。它可能可以从其他在线位置下载，文件详细信息如下：

    ```java
    name: dxsdk_sum2004.exe
    size: 239008008 bytes
    sha1sum: 73d875b97591f48707c38ec0dbc63982ff45c661
    ```

1.  将文件从 `C:\Program Files\Microsoft DirectX 9.0 SDK (Summer 2004)` 目录复制到 `<toolchain>\directx` 目录。

1.  在 Windows i586 实例上，将 `C:\Windows\System32` 目录中的文件复制到 `<toolchain>\msvcr\7_32` 目录：

    ```java
    msvcp100.dll
    msvcr100.dll
    msvcr100_clr0400.dll
    ```

1.  在 Windows amd64 实例上，将上一步相同的文件（它们应该具有 amd64 架构）复制到 `<toolchain>\msvcr\7_64` 目录。

1.  创建一个 `env_32.bat` 文件，包含以下环境配置，可用于 i586 构建：

    ```java
    set TOOLCHAIN=<toolchain>
    set VS=%TOOLCHAIN%/winsdk71/vs2010e
    set WINSDK=%TOOLCHAIN%/winsdk71/sdk
    set ALT_COMPILER_PATH=%VS%/VC/Bin
    set ALT_WINDOWSSDKDIR=%WINSDK%
    set ALT_MSVCRNN_DLL_PATH=%TOOLCHAIN%/msvcr/7_32
    set ALT_DXSDK_PATH=%TOOLCHAIN%/directx
    set WINDOWSSDKDIR=%WINSDK%
    set VS100COMNTOOLS=%VS%/Common7/Tools
    set Configuration=Release
    set WindowsSDKVersionOverride=v7.1
    set ToolsVersion=4.0
    set TARGET_CPU=x86
    set CURRENT_CPU=x86
    set PlatformToolset=Windows7.1SDK
    set TARGET_PLATFORM=XP
    set LIB=%VS%/VC/Lib;%WINSDK%/Lib
    set LIBPATH=%VS%/VC/Lib
    set PATH=%VS%/Common7/IDE;%VS%/Common7/Tools;%VS%/VC/Bin;%VS%/VC/Bin/VCPackages;%WINSDK%/Bin;C:/WINDOWS/system32;C:/WINDOWS;C:/WINDOWS/System32/Wbem;%TOOLCHAIN%/msvcr/7_32
    set INCLUDE=%VS%/VC/INCLUDE;%WINSDK%/INCLUDE;%WINSDK%/INCLUDE/gl;
    ```

1.  创建一个 `env_64.bat` 文件，包含以下环境配置，可用于 amd64 构建：

    ```java
    set TOOLCHAIN=...
    set VS=%TOOLCHAIN%/winsdk71/vs2010e
    set WINSDK=%TOOLCHAIN%/winsdk71/sdk
    set ALT_COMPILER_PATH=%VS%/VC/Bin/x86_amd64
    set ALT_WINDOWSSDKDIR=%WINSDK%
    set ALT_MSVCRNN_DLL_PATH=%TOOLCHAIN%/msvcr/7_64
    set ALT_DXSDK_PATH=%TOOLCHAIN%/directx
    set WINDOWSSDKDIR=%WINSDK%
    set VS100COMNTOOLS=%VS%/Common7/Tools
    set Configuration=Release
    set WindowsSDKVersionOverride=v7.1
    set ToolsVersion=4.0
    set TARGET_CPU=x64
    set CURRENT_CPU=x64
    set PlatformToolset=Windows7.1SDK
    set TARGET_PLATFORM=XP
    set LIB=%VS%/VC/Lib/amd64;%WINSDK%/Lib/x64
    set LIBPATH=%VS%/VC/Lib/amd64
    set PATH=%VS%/Common7/IDE;%VS%/Common7/Tools;%VS%/VC/Bin/x86_amd64;%VS%/VC/Bin;%VS%/VC/Bin/VCPackages;%WINSDK%/Bin;C:/WINDOWS/System32;C:/WINDOWS;C:/WINDOWS/System32/wbem;%LIBS_DIR%/msvcr/7_64;%TOOLCHAIN%/msvcr/7_32;%VS%/Common7/IDE
    set INCLUDE=%VS%/VC/INCLUDE;%WINSDK%/INCLUDE;%WINSDK%/INCLUDE/gl;
    ```

1.  将 `toolchain` 目录添加到源控制仓库。这个文件集是一个独立的工具链，可以用来在具有相应架构的干净 Windows 映像上构建 OpenJDK 7（以及 OpenJDK 8）的 i586 和 amd64 版本。

## 它是如何工作的...

本食谱中的过程相当直接：收集要复制到干净 Windows 映像上的已安装文件，并为它们准备环境。

可能的一个问题是，虽然来自 Windows SDK 7.1 的 Microsoft 链接工具 `link.exe` 不需要 .NET 4 运行时来链接原生二进制文件，但它需要 .NET 共享库，`msvcr100_clr0400.dll`（参见本食谱的第 7 步）。此库必须在 `PATH` 中找到，否则构建将在 HotSpot VM 链接阶段失败，并出现一个不明确的错误。

## 还有更多...

在本食谱中，为了简洁，从环境文件中移除了不特定于 Microsoft 工具链（Cygwin、FreeType 等）的 OpenJDK 配置。为了执行 OpenJDK 构建，应将其重新添加到环境文件中。

独立工具链的结果不特定于 OpenJDK，也可以用于在 Windows 上构建其他软件。

## 参见

+   本章中的*在 Windows 7 SP1 上构建 32 位 OpenJDK 7*食谱和*在 Windows 上为 OpenJDK 7 构建 64 位 FreeType 库*食谱。它们可以被调整以使用本食谱中的独立工具链

+   来自[`stackoverflow.com/`](http://stackoverflow.com/)的一个关于.NET4 运行时链接器依赖的问题，在[`stackoverflow.com/questions/13571628/compiling-c-code-using-windows-sdk-7-1-without-net-framework-4-0`](http://stackoverflow.com/questions/13571628/compiling-c-code-using-windows-sdk-7-1-without-net-framework-4-0)
