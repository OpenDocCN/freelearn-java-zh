# 第四章：构建 OpenJDK 8

在本章中，我们将涵盖：

+   使用 GNU Autoconf

+   在 Ubuntu Linux 12.04 LTS 上构建 OpenJDK 8

+   使用 ccache 加速 OpenJDK 8 的构建过程

+   在 Mac OS X 上构建 OpenJDK 8

+   在 Windows 7 SP1 上构建 OpenJDK 8

# 简介

Java 8 规范为 Java 平台带来了许多创新。除了新的语言特性，如函数式接口和 Lambda 表达式支持，以及库特性，如流和新的日期/时间 API 之外，OpenJDK 8 还有一个新的构建系统。由于本章是关于构建 OpenJDK 的，我们将深入探讨最后一个创新。

很长一段时间里，Sun Java 和 OpenJDK 构建系统围绕着发布流程发展。发布工程师的需求总是优先于开发者的需求，并且开发者对构建过程的需求与发布需求大相径庭。对于发布准备，构建过程必须是稳定的。发布构建通常是从头开始构建的，它们总是包括整个项目，速度和环境配置的复杂性对他们来说不是问题。相反，需要部分、增量以及尽可能快的构建作为开发的生命线。此外，构建脚本必须尽可能干净和易于管理，以便于轻松更改和微调。

在 OpenJDK 7 发布之前，构建过程具有以下巨大的跨平台项目的所有以下*缺陷*，如下列出：

+   复杂的环境设置

+   没有可靠的增量构建

+   并行构建支持不完整

+   不支持编译缓存

+   在某些部分构建速度不合理

+   不同类型的构建脚本（一些用于 GNU，一些用于 Apache Ant）

+   有限的源列表选择能力下隐式编译 Java 源代码

+   对环境的检查过于严格（对于发布是必需的，但在非标准环境中则成为负担）

+   *丢弃*额外的二进制组件

+   依赖特定版本的构建工具

+   不支持交叉编译

由于 OpenJDK 项目的出现，公共访问构建导致构建系统变得更加适合开发者和“普通构建者”。在 OpenJDK 7 的时间线中，构建过程经历了一些清理，构建变得更加容易，但基本问题仍然相同。

最后，在 OpenJDK 8 的开发过程中做出了巨大的努力。构建系统被完全重写，其中大部分是从头开始重写的。除了清理和重构 Makefiles 以及移除 Apache Ant 之外，主要问题也得到了解决：速度、多核支持、部分和增量构建、交叉编译以及更简单的环境设置。

后者的改进很大程度上是由于引入了预构建步骤，该步骤为当前环境准备项目。此构建配置步骤使用 GNU Autoconf 构建系统执行。我们将在下面的菜谱中更详细地探讨它。

OpenJDK 8 支持 Linux、Windows、Mac OS X 和 Solaris 操作系统。以下将进一步讨论 Windows、Linux 和 Mac OS X 版本。在 Linux 和 Windows 操作系统上，支持 x86 和 x86_64 架构。在 Mac OS X 上，仅支持 x86_64。

OpenJDK 8 中的处理器架构配置已更改。以前，x86 架构被命名为 i586，x86_64 被命名为 amd64。但现在 x86 和 x86_64 名称被直接使用。此外，由于交叉编译支持，x86 版本可以在进行少量配置更改的 x86_64 操作系统上构建。因此，在本章中，我们将重点关注 x86_64，并在需要时提及 x86 构建。

# 使用 GNU Autoconf

**GNU Autoconf**，或简称为 GNU 构建系统，是一套构建工具，旨在帮助在不同类 Unix 系统之间使源代码包可移植。

Autoconf 包含多个与构建相关的工具，通常充当 Makefile 生成器。Makefile 模板在`configure`构建步骤中使用用户指定的环境信息和配置选项进行处理。

Autoconf 系统相当复杂，关于其在现代项目中的使用存在一些争议。对于某些项目，它可能过于陈旧、过于复杂，并且与其他平台相比，过于倾向于类 Unix 系统。但对于 OpenJDK 这样的高度跨平台项目，它已经大量依赖于类 Unix 工具，因此在配置构建步骤中使用 autotool 是合理的。

Autoconf 系统使用 GNU make 进行实际的构建步骤，但与 make 工具相比，整个 Autoconf 包的可移植性较低，特定的构建设置可能对 Autoconf 包的版本有要求。幸运的是，这种负担可以从开发者转移到构建系统工程师。对于构建配置步骤，Autoconf 生成一个独立的 shell 脚本。这样的脚本，通常命名为`configure`，不仅可以不使用其他构建系统工具而使用，还可以在有限的类 Unix 环境中运行，例如 Windows 平台上的 Cygwin。

在 OpenJDK 中，`configure`脚本事先准备并添加到源代码 tarball 中，因此构建过程不需要除了 GNU make 之外的任何 autotools 构建工具。

在这个配方中，我们将探索使用不同选项的 OpenJDK 8 构建配置。

## 准备工作

对于这个配方，你需要一个具有类 Unix 环境的操作系统：Linux、Mac OS X，或者安装了 Cygwin 的 Windows。请参阅第二章中关于安装 Cygwin 的配方“*为 Windows 构建安装 Cygwin”。

## 如何操作...

以下步骤将帮助我们配置 OpenJDK 8 构建环境：

1.  从 Oracle 安装 JDK 7 或预构建的 OpenJDK 二进制文件。

1.  下载并解压缩官方 OpenJDK 8 源代码存档（在撰写本书时，此存档不可用）。

1.  使用 `--help` 选项运行构建配置脚本：

    ```java
    bash ./configure --help

    ```

    您现在可以看到可用的配置选项列表。让我们尝试其中的一些。

1.  运行以下命令以在构建期间直接指定要使用的 JDK 作为启动 JDK：

    ```java
    bash ./configure --with-boot-jdk=path/to/jdk

    ```

1.  运行以下命令以取消包含的加密算法实现的默认限制：

    ```java
    bash ./configure –-enable-unlimited-crypto

    ```

1.  运行以下命令以在构建过程中不生成调试符号，也不将它们包含在目标分发中：

    ```java
    bash ./configure --disable-debug-symbols --disable-zip-debug-info

    ```

1.  运行以下命令以在所有平台上强制将 FreeType 库捆绑到 OpenJDK 中：

    ```java
    bash ./configure --enable-freetype-bundling

    ```

1.  运行以下命令以将 CA 证书指定给 `keystore` 文件。请参阅第二章中的*准备 CA 证书*配方，*构建 OpenJDK 6*：

    ```java
    bash ./configure --with-cacerts-file=path/to/cacerts

    ```

1.  运行以下命令以指定构建的里程碑和构建号，而不是生成的那些：

    ```java
    bash ./configure –with-milestone=my-milestone --with-build-number=b42

    ```

## 它是如何工作的...

在 OpenJDK 中，由于 OpenJDK 源代码库策略，configure 脚本默认未标记为可执行。因此，在这个配方中，显式使用 `bash` 来运行脚本。

`configure` 脚本在 `build` 目录中准备构建配置。它执行许多检查并写入特定于环境的详细信息，用户以环境变量的形式提供选项。这些变量将在实际构建过程中自动读取。

## 还有更多...

OpenJDK 支持许多选项；大多数选项可以同时指定以配置脚本。

## 参见

+   本章以下配方用于实际构建 OpenJDK 8

+   GNU Autoconf 项目的官方网站[`www.gnu.org/software/autoconf/`](http://www.gnu.org/software/autoconf/)

+   关于采用 Autoconf 的讨论的邮件列表线程，请参阅[`mail.openjdk.java.net/pipermail/build-infra-dev/2011-August/000030.html`](http://mail.openjdk.java.net/pipermail/build-infra-dev/2011-August/000030.html)

# 构建 OpenJDK 8 Ubuntu Linux 12.04 LTS

此配方与第三章中的*在 Ubuntu Linux 12.04 LTS 上构建 OpenJDK 7*配方相似。

OpenJDK 的构建过程严重依赖于类 Unix 开发工具。基于 Linux 的操作系统通常对这些工具提供顶级支持，因此，在 Linux（以及 Mac OS X）上构建 OpenJDK 可能比在 Windows 上简单。对于像 Fedora 或 Ubuntu 这样的主要发行版，构建工具链和所有依赖项都已作为软件包包含在发行版中，可以轻松安装。

选择 Ubuntu 12.04 LTS 作为本书的操作系统，因为它是最受欢迎的 Linux 发行版之一。对于运行其他操作系统的读者，可以在网上找到 Ubuntu 12.04 虚拟镜像，用于最流行的虚拟化工具，如 Oracle VirtualBox 或 VMware。

要为 x86 和 x86_64 架构构建二进制文件，应使用相应的 Ubuntu 版本。对于这两个架构，构建说明完全相同，因此在此配方中不会进一步提及。OpenJDK 8 支持交叉编译，因此 x86 版本可以在 x86_64 操作系统上构建。但这种交叉编译需要非平凡的库配置，我们在此配方中不会使用它。

在 Linux 上，应使用 Oracle Linux 6.4 amd64 和 GCC 版本 4.7 来构建应产生最兼容 OpenJDK 8 二进制的最小构建环境。我们将配置 Ubuntu 12.04 使用 GCC 4.7 以接近最小构建环境。

## 准备工作

对于此配方，我们需要运行干净的 Ubuntu 12.04（服务器或桌面版本）。

## 如何操作...

以下步骤将帮助我们构建 OpenJDK 8：

1.  安装 OpenJDK 7 的预包装二进制文件：

    ```java
    sudo apt-get install openjdk-7-jdk

    ```

1.  安装 GCC 工具链和构建依赖项：

    ```java
    sudo apt-get build-dep openjdk-7

    ```

1.  添加额外的软件包仓库：

    ```java
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update

    ```

1.  安装 C 和 C++编译器，版本 4.7：

    ```java
    sudo apt-get install gcc-4.7
    sudo apt-get install g++-4.7

    ```

1.  进入`/usr/bin`目录并设置默认编译器符号链接：

    ```java
    sudo rm gcc
    sudo ln -s gcc-4.7 gcc
    sudo rm g++
    sudo ln -s g++-4.7 g++

    ```

1.  下载并解压缩官方 OpenJDK 8 源代码存档（此存档在撰写本书时不可用）

1.  运行 autotools 配置脚本：

    ```java
    bash ./configure

    ```

1.  开始构建：

    ```java
    make all 2>&1 | tee make_all.log

    ```

1.  运行构建二进制文件：

    ```java
    cd build/linux-x86_64-normal-server-release/images/
    ./bin/java -version
    openjdk version "1.8.0-internal"
    OpenJDK Runtime Environment (build 1.8.0-internal-ubuntu_2014_02_22_08_51-b00)
    OpenJDK 64-Bit Server VM (build 25.0-b69, mixed mode)
    ```

1.  返回源根目录并构建紧凑配置文件中的图像：

    ```java
    make profiles 2>&1 | tee make_profiles.log

    ```

1.  检查`build/linux-x86_64-normal-server-release/images`目录中的配置文件图像：

    ```java
    j2re-compact1-image
    j2re-compact2-image
    j2re-compact3-image
    ```

## 它是如何工作的...

需要 OpenJDK 7 的预包装二进制文件，因为一些构建步骤是使用外部 Java 运行时运行的。

`build-dep`命令用于安装构建指定包所需的所有依赖项。由于 Ubuntu 打包的 OpenJDK 6 与官方 OpenJDK 6 非常接近，此命令将安装几乎所有必需的依赖项。

Ubuntu 12.04 默认包仓库中没有 GCC 4.7 编译器，因此需要额外的仓库配置。

## 还有更多...

默认的 GCC 4.6 编译器也可以用来构建 OpenJDK 8。

OpenJDK 8 支持在 x86_64 宿主平台上交叉编译 x86 二进制文件。但`build-dep -a i386 openjdk-7`命令来安装所有必需的 x86 依赖项将不会工作（一些 x86 依赖项在 x86_64 OS 上不可安装），手动依赖项安装可能相当复杂。在单独的 x86 Ubuntu 12.04 实例上使用与此配方中完全相同的步骤构建 x86 二进制文件可能更容易。

## 参见

+   来自本章的*使用 GNU Autoconf*配方

+   来自第二章的*准备 CA 证书*配方，*构建 OpenJDK 6*

+   关于在[`hg.openjdk.java.net/jdk8u/jdk8u/raw-file/2f40422f564b/README-builds.html`](http://hg.openjdk.java.net/jdk8u/jdk8u/raw-file/2f40422f564b/README-builds.html)构建 OpenJDK 8 的官方说明

+   关于在 Linux 发行版上构建 OpenJDK 8 软件包的非官方说明，请参阅 [`github.com/hgomez/obuildfactory/wiki/How-to-build-and-package-OpenJDK-8-on-Linux`](https://github.com/hgomez/obuildfactory/wiki/How-to-build-and-package-OpenJDK-8-on-Linux)

# 使用 ccache 加速 OpenJDK 8 构建过程

除了 Java 源代码外，OpenJDK 源代码库还包含大量的原生 C 和 C++ 源代码。原生代码编译比 Java 长得多，可能需要很长时间。在开发过程中，相同的代码可能会因为细微的变化而多次编译。代码部分的中间二进制结果可能在编译之间完全相同，但通常即使在这些部分没有添加更改，代码的部分也会被重新编译。自然地，人们会期望从现代高级编译/链接工具链中采用更聪明的代码重新编译方法。

**ccache** 工具为原生编译提供了这样的智能。此工具缓存 C/C++ 编译的输出，以便下次可以避免相同的编译，并从缓存中获取结果。这可以大大加快重新编译的时间。检测是通过散列不同类型的信息来完成的，这些信息对于编译应该是唯一的，然后使用散列总和来识别缓存的输出。

OpenJDK 8 支持 Linux 和 Mac OS X 上的 ccache，但在撰写本书时并未支持 Windows。

在这个配方中，我们将为 Ubuntu 12.04 上的 OpenJDK 8 构建设置 ccache。

## 准备工作

对于这个配方，我们需要设置 Ubuntu 12.04（服务器或桌面版本）以构建 OpenJDK 8（请参阅前面的配方）。

## 如何操作...

以下步骤将帮助我们启用 ccache：

1.  安装 `ccache` 软件包：

    ```java
    sudo apt-get install ccache

    ```

1.  在 OpenJDK 8 源代码目录中运行以下命令重新配置项目：

    ```java
    bash ./configure

    ```

1.  检查 `configure` 脚本输出的这一行：

    ```java
    ccache status:  installed and in use

    ```

## 它是如何工作的...

在 OpenJDK 8 中，`configure` 脚本检查缓存可用性，并在后续构建过程中自动启用其使用。

## 更多...

对于产品型构建，当结果二进制文件将用于生产时，可能更安全地重新配置项目，使用 `--disable-ccache` 选项禁用 `ccache`。

## 参考信息

+   本章中关于 *在 Ubuntu Linux 12.04 LTS 上构建 OpenJDK 8* 的先前配方

+   ccache 的官方网站为 [`ccache.samba.org/`](http://ccache.samba.org/)

# 在 Mac OS X 上构建 OpenJDK 8

OpenJDK 8 支持作为一等公民的 Mac OS X 平台，并且使用适当的工具链构建它几乎和 Linux 一样简单。

从历史上看，Java 在 Mac OS X 上一直享有第一类支持。JDK 基于 Sun 代码库，但由 Apple 构建，并与操作系统环境紧密结合。直到 Mac OS X 10.4 Tiger，使用标准 Swing 工具包编写的图形用户界面应用程序可以访问大多数 Cocoa 本地界面功能。用 Java 编写的应用程序感觉非常接近本地应用程序，同时仍然是跨平台的。

然而，对于下一个版本，Java 的支持水平下降了。从 Mac OS X 10.5 Leopard 开始，新的 Cocoa 功能不再支持 Java。Apple Java 6 的发布被推迟（与其他平台上的 Sun 发布相比）超过一年。Java 6 于 2006 年 12 月发布，但直到 2008 年 4 月才对 Mac OS X 用户可用。最后，在 2010 年 10 月，Apple 正式宣布其决定停止对 Java 的支持。Apple Java 6 仍在更新安全更新，并且可以安装在 Mac OS X 10.9 Mavericks（撰写本书时的最新版本）上，但 Apple 不会发布后续的 Java 版本。

对于 Mac OS X，存在第三方开源 Java 发行版。最著名的是基于 FreeBSD Java 1.6 补丁集的 SoyLatte X11 端口，用于 Mac OS X Intel 机器。SoyLatte 早在 OpenJDK 之前就存在，采用 Java 研究许可，并支持 Java 6 构建。现在它是 OpenJDK BSD 端口项目的一部分。

OpenJDK 7 在其他平台上添加了对 Mac OS X 的全面支持，并在 OpenJDK 8 中继续。

## 准备工作

对于这个配方，我们需要一个干净安装的 Mac OS X 10.7.5 Lion（或任何支持 Xcode 4 的后续版本）。

## 如何操作...

以下步骤将帮助我们构建 OpenJDK 8：

1.  从 [`developer.apple.com/xcode/`](https://developer.apple.com/xcode/) 下载 Xcode 4.6.2 for Lion（2013 年 4 月 15 日），需要 Apple 开发者账户（注册免费）并安装它。

1.  使用与前面提到的相同下载链接下载 Xcode 命令行工具，版本为 2013 年 4 月（2013 年 4 月 15 日），并安装它。

1.  从终端运行此命令以设置命令行工具：

    ```java
    sudo xcode-select -switch /Applications/Xcode.app/Contents/Developer/

    ```

1.  导航到 **应用程序** | **实用工具** 并运行 **X11.app** 以设置 X 服务器。

1.  安装 JDK 7 Oracle 发行版，或者可以使用预构建的 OpenJDK 二进制文件。

1.  下载并解压缩官方 OpenJDK 8 源代码存档（撰写本书时此存档不可用）。

1.  运行 autotools 配置脚本：

    ```java
    bash ./configure –-with-boot-jdk=path/to/jdk7

    ```

1.  开始构建：

    ```java
    make all 2>&1 | tee make_all.log

    ```

1.  运行构建的二进制文件：

    ```java
    cd build/macosx-x86_64-normal-server-release/images/j2sdk-image
    ./bin/java -version
    openjdk version "1.8.0-internal"
    OpenJDK Runtime Environment (build 1.8.0-internal-obf_2014_02_20_19_08-b00)
    OpenJDK 64-Bit Server VM (build 25.0-b69, mixed mode)

    ```

## 它是如何工作的...

尽管用于原生源主要部分的 Xcode 命令行工具，但还需要 Xcode 本身来构建特定平台的代码。

Mac OS X 上的 OpenJDK 正在从使用 X11 服务器迁移，但对于版本 8 的构建仍然需要它。Mac OS X 10.7 Lion 预装了 X11，只需运行一次即可配置用于构建。

## 还有更多...

此菜谱使用官方的 Apple GCC (和 G++) 版本 4.2 构建。在此版本之后，由于许可原因，官方 Apple 对 GCC 的支持已停止。Clang——最初由 Apple 开发的开源编译器——是较新版本 Mac OS X 的默认和首选编译器。

尽管最初的开发者计划如此，但 OpenJDK 8 仍然需要 GCC，不能使用 Clang 构建。这就是为什么使用 Xcode 的第 4 版而不是第 5 版，因为第 5 版根本不包含 GCC。在撰写本书时，Xcode 5 和 Clang 支持正在添加到 OpenJDK 9 项目中。这种支持可能会后来回滚到 OpenJDK 8；如果您想使用 Clang 构建 8 版本，最好检查 OpenJDK 邮件列表以获取最新信息。

在 Mac OS X 10.8 Mountain Lion 上，应单独安装 10.9 Mavericks X11 服务器，使用 XQuartz 项目。

## 在 Windows 平台上构建 OpenJDK 8 的过程与版本 7 相比有重大改进。尽管如此，构建环境设置仍然比 Linux 和 Mac OS X 复杂得多。构建的大部分复杂性来自于它通过 Cygwin 工具使用类似 Unix 的构建环境。

+   本章的 *使用 GNU Autoconf* 菜单

+   来自 第二章 的 *准备 CA 证书* 菜单，*构建 OpenJDK 6*

+   关于在 [`hg.openjdk.java.net/jdk8u/jdk8u/raw-file/2f40422f564b/README-builds.html`](http://hg.openjdk.java.net/jdk8u/jdk8u/raw-file/2f40422f564b/README-builds.html) 构建 OpenJDK 8 的官方说明

+   关于在 Mac OS X 上构建 OpenJDK 8 软件包的非官方说明 [`github.com/hgomez/obuildfactory/wiki/Building-and-Packaging-OpenJDK8-for-OSX`](https://github.com/hgomez/obuildfactory/wiki/Building-and-Packaging-OpenJDK8-for-OSX)

# 在 Windows 7 SP1 上构建 OpenJDK 8

与之相关的还有

对于 i586 构建官方的编译器要求是 Microsoft Visual Studio C++ 2010 专业版。Visual Studio 2010 的社区版也可以用于 x86 构建。对于 x86_64 构建，我们将使用 Microsoft Windows SDK 版本 7.1，适用于 Windows 7。此 SDK 可从 Microsoft 网站免费获取。它使用与 Visual Studio 2010 Express 相同的编译器。它仅包含命令行工具（没有 GUI），但如果需要 GUI，也可以作为 Visual Studio 2010 的外部工具集使用。

## 准备工作

对于这个菜谱，我们应该有一个没有安装任何防病毒软件的 Windows 7 SP1 amd64 系统。不允许使用防病毒软件，因为它可能会干扰 Cygwin 运行时。

## 如何操作...

以下步骤将帮助我们构建 OpenJDK 8：

1.  从 Microsoft 网站下载 Microsoft .NET Framework 4 并安装它。

1.  从微软网站下载 Windows 7 的 amd64 版本 Microsoft Windows SDK（`GRMSDKX_EN_DVD.iso` 文件）并将其安装到默认位置。不需要 `.NET 开发` 和 `通用工具` 组件。

1.  从微软网站下载 Visual Studio 2010 Express 版本并将其安装到默认位置。只需要 Visual C++ 组件。

1.  将 Microsoft DirectX 9.0 SDK（2004 年夏季）下载并安装到默认安装路径。请注意，此分发在微软网站上已不再可用。它可能可以从其他在线位置下载，文件详细信息如下：

    ```java
    name: dxsdk_sum2004.exe
    size: 239008008 bytes
    sha1sum: 73d875b97591f48707c38ec0dbc63982ff45c661
    ```

1.  将预安装的 Cygwin 版本安装或复制到 `c:\cygwin`。

1.  创建一个 `c:\path_prepend` 目录，并将 Cygwin 安装中的 `find.exe` 和 `sort.exe` 文件复制到该目录。

1.  使用 [`www.cmake.org/`](http://www.cmake.org/) 网站上的 [`www.cmake.org/files/cygwin/make.exe-cygwin1.7`](http://www.cmake.org/files/cygwin/make.exe-cygwin1.7) 下载 GNU make 工具二进制文件，将其重命名为 `make.exe`，并将其放入 `c:\make` 目录。

1.  从 `openjdk-unofficial-builds GitHub` 项目（目录 `7_64`）下载预构建的 FreeType 库，并将二进制文件放入 `c:\freetype\lib` 目录，将头文件放入 `c:\freetype\include` 目录。

1.  将 OpenJDK 7 二进制文件或 Oracle Java 7 安装到 `c:\jdk7`。

1.  下载官方 OpenJDK 8 源代码存档（此存档在本书编写时不可用）并将其解压缩到 `c:\sources` 目录。

1.  将以下命令写入 `build.bat` 文本文件：

    ```java
    @echo off
    set PATH=C:/path_prepend;C:/WINDOWS/system32;C:/WINDOWS;C:/make;C:/cygwin/bin;C:/jdk7/bin
    bash
    echo Press any key to close window ...
    pause > nul

    ```

1.  从 Windows 资源管理器运行 `build.bat`。应该会弹出一个带有 bash 启动的 `cmd.exe` 窗口。

1.  在 bash 命令提示符下运行以下命令：

    ```java
    cd /cygdrive/c/sources
    bash ./configure \
     --with-boot-jdk=c:/jdk7 \
     --with-freetype=c:/freetype \
    chmod –R 777 .
    make > make.log 2>&1

    ```

1.  启动另一个 Cygwin 控制台并运行以下命令：

    ```java
    cd /cygdrive/c/sources
    tail -f make.log

    ```

1.  等待构建完成。

1.  运行以下构建二进制文件：

    ```java
    cd build/windows-x86_64-normal-server-release/images/j2sdk-image
    ./bin/java -version
     openjdk version "1.8.0-internal"
     OpenJDK Runtime Environment (build 1.8.0-internal-obf_2014_02_22_17_13-b00)
     OpenJDK 64-Bit Server VM (build 25.0-b69, mixed mode)

    ```

## 它是如何工作的...

Cygwin 安装在 第二章 的配方 *为 Windows 构建安装 Cygwin* 中有详细说明，*构建 OpenJDK 6*。这里使用磁盘 C 的根目录是为了简洁。通常，可以使用由 ASCII 字母或数字组成的任意路径，路径中不能有空格。也可以使用较新的 DirectX SDK 版本。

不同版本的 GNU make 在 Windows 上可能存在不同的问题。这个来自 cmake 项目的特定版本在不同的 Windows 版本上进行了测试，并且运行良好。

此配方使用来自 `openjdk-unofficial-builds GitHub` 项目的预构建 FreeType 2.4.10 库。FreeType 可以使用相同的 Windows SDK 7.1 工具链从源代码构建。

`make` 输出将被重定向到 `make.log` 文件。`2>&1` 语句确保 `stdout` 和 `stderr` 都将被重定向。

`tail -f` 命令允许我们在构建过程中查看 `make.log` 文件的内容。

在批处理文件末尾添加的 `pause > nul` 命令可以防止在运行时错误的情况下 `cmd.exe` 窗口消失。

## 更多内容...

可以使用 `--with-target-bits=32` 配置选项在 amd64 操作系统上构建 i586 二进制文件。在这种情况下，应通过 `--with-freetype` 选项指定 FreeType 库的 x86 版本，并且应安装 Visual Studio 2010 的 Express 版本。

可以使用 `tee` 命令代替 `>` 和 `tail`，将构建日志同时写入文件和控制台。

## 参见

+   本章的 *使用 GNU Autoconf 菜谱*

+   来自 第二章 的 *准备 CA 证书* 菜谱，*构建 OpenJDK 6*

+   来自 第二章 的 *为 Windows 构建安装 Cygwin* 菜谱，*构建 OpenJDK 6*

+   来自 第三章 的 *在 Windows 上构建 64 位 FreeType 库* 菜谱，*构建 OpenJDK 7*

+   关于在 [`hg.openjdk.java.net/jdk8u/jdk8u/raw-file/2f40422f564b/README-builds.html`](http://hg.openjdk.java.net/jdk8u/jdk8u/raw-file/2f40422f564b/README-builds.html) 构建 OpenJDK 8 的官方说明
