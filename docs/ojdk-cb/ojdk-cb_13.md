# 第十三章。构建自动化

在本章中，我们将介绍：

+   安装 VirtualBox

+   准备 SSH 密钥

+   准备装有 Linux 的 VirtualBox 机器

+   准备装有 Mac OS X 的 VirtualBox 机器

+   准备装有 Windows 的 VirtualBox 机器

+   自动化构建

+   构建跨平台安装程序

# 简介

自动化构建在跨平台软件开发中被广泛使用。需要一个 *构建农场*，其中包含每个支持的操作系统的构建机器，以远程构建项目和在所有平台上运行测试。随着软件虚拟化工具的日益流行，使用单个物理机器部署 *虚拟构建农场* 变得可能。

除了每个操作系统的构建环境之外，自动化构建的关键部分是 *主* 机器和 *构建* 机器之间的通信。可能需要以下通信任务：

+   主服务器应准备并启动构建机器

+   主服务器应直接将源代码发送到构建机器，或者从构建机器本身启动源代码检索过程

+   主服务器应启动构建过程

+   构建机器应在构建过程中将构建日志发送到主服务器

+   构建机器应将结果二进制文件发送到主服务器

+   主服务器应关闭构建机器

在本章中，我们将为 Windows、Linux 和 Mac OS X 准备 OpenJDK 构建环境。这项任务可以使用高级构建自动化（或持续集成）工具来完成。然而，这样的工具在功能上可能有限，并且对于我们的任务来说可能不够灵活。虽然所有这样的工具都应该执行类似的工作（如之前所列），但不同的工具可能有不同的配置和特性，对一种工具的了解可能对另一种工具不那么有用。此外，这样的工具还引入了一个额外的复杂性层次，可能会带来特定工具的问题。

您将学习如何使用最基本的工具进行构建自动化，以便更好地理解这个过程。bash shell 已经在所有平台上用于 OpenJDK 构建（在 Linux/Mac 上是原生的，在 Windows 上通过 Cygwin），因此我们将使用 bash 脚本来设置和启动构建虚拟机。对于通信（发送命令和数据），我们将使用 SSH 协议；Unix-like 操作系统上通常预装了该协议的实现，也可以在 Windows 上安装。对于虚拟化，我们将使用来自甲骨文的流行工具 VirtualBox。

# 安装 VirtualBox

VirtualBox 是来自甲骨文公司的一个流行的虚拟化工具箱，它允许我们在宿主操作系统之上运行其他操作系统的 *虚拟* 实例。在本食谱中，我们将安装 VirtualBox 并配置宿主网络接口。这样的配置将允许我们在自动化构建过程中从宿主连接到客户机，然后再返回。

## 准备工作

对于这个食谱，我们需要运行 Ubuntu 12.04 amd64 操作系统。

## 如何操作...

以下步骤将帮助您安装 VirtualBox：

1.  从 VirtualBox 网站（[`www.virtualbox.org/`](https://www.virtualbox.org/））下载安装包并安装。

1.  安装虚拟网络接口包：

    ```java
    sudo apt-get install uml-utilities

    ```

1.  创建一个虚拟接口`tap0`，该接口将用于连接到和从客户机：

    ```java
    sudo tunctl -t tap0 -u your_username

    ```

1.  配置接口`tap0`：

    ```java
    sudo ifconfig tap0 192.168.42.2 up dstaddr 192.168.42.1 netmask 255.255.255.0

    ```

1.  使用 GUI 表单创建任意 VirtualBox 机器。

1.  导航到**设置** | **网络表单**，当使用**桥接适配器**模式时，检查**tap0**网络接口是否在接口下拉列表中可用。

## 工作原理...

VirtualBox 支持为虚拟机提供不同的网络选项。其中之一——桥接适配器——允许我们使用静态 IP 地址从主机连接到客户机并返回。要在主机端设置此模式，我们需要一个额外的虚拟网络接口和单独的地址。`uml-utilities`包中包含的`tunctl`实用程序允许我们创建一个如`tap0`这样的虚拟接口。

## 更多内容...

可以创建多个具有不同地址的虚拟接口，以同时运行多个虚拟机。Mac OS X 可以用作主机机器，需要`tuntaposx`内核扩展来使用`tunctl`实用程序。

## 参见

+   Oracle VirtualBox 用户手册可在[https://www.virtualbox.org/manual/UserManual.html](https://www.virtualbox.org/manual/UserManual.html)找到

# 准备 SSH 密钥

在本章中，我们将使用虚拟机来构建 OpenJDK。在构建源代码、控制命令、日志和结果二进制文件时，应在主机和虚拟机之间发送。我们将使用无处不在的**安全外壳协议**（**SSH**）及其最流行的实现**OpenSSH**来完成这些任务。

SSH 允许我们在机器之间发送数据并远程运行命令。当客户端执行 SSH 连接时，它应通过服务器进行认证。除了用户/密码认证外，OpenSSH 还支持使用非对称加密（RSA 或类似）密钥进行认证。配置了 SSH 密钥后，客户端可以无需手动干预连接到服务器。这简化了复制多个文件或运行远程任务的脚本编写。

在这个配方中，我们将准备一套公钥和私钥，并在构建过程中在所有虚拟机和主机机器上使用这些密钥。

## 准备工作

对于这个配方，我们需要一个运行着 OpenSSH 服务器和客户端的类 Unix 操作系统。例如，可以使用 Ubuntu 12.04 操作系统。

## 如何操作...

以下步骤将帮助我们安装 VirtualBox：

1.  为主机和客户机生成客户端 RSA 密钥对：

    ```java
    ssh-keygen -q -P "" -t rsa -f vmhost__id_rsa
    ssh-keygen -q -P "" -t rsa -f vmguest__id_rsa

    ```

1.  为主机和客户机生成服务器 RSA 密钥对：

    ```java
    ssh-keygen -q -P "" -t rsa -f vmhost__ssh_host_rsa_key
    ssh-keygen -q -P "" -t rsa -f vmguest__ssh_host_rsa_key

    ```

1.  在主机机器上创建一个新用户，该用户将用于管理构建，并以此用户登录：

    ```java
    sudo adduser packt

    ```

1.  运行 VirtualBox，并按照上一配方“安装 VirtualBox”中的说明配置网络，使用`192.168.42.2`的主机 IP 地址和`192.168.42.1`的客户机 IP 地址。

1.  在虚拟机机器上创建一个具有相同名称的用户，并以此用户登录：

    ```java
    sudo adduser packt

    ```

1.  检查`ping`命令是否可以从主机成功连接到虚拟机，并从虚拟机返回到主机：

    ```java
    ping 192.168.42.1
     PING 192.168.42.2 (192.168.42.2) 56(84) bytes of data.
     64 bytes from 192.168.42.2: icmp_req=1 ttl=64 time=0.181 ms
     ...

    ```

1.  在主机机器上，设置客户端密钥：

    ```java
    cp vmhost__id_rsa.pub ~/.ssh/id_rsa.pub
    cp vmhost__id_rsa ~/.ssh/id_rsa
    chmod 600 ~/.ssh/id_rsa

    ```

1.  在虚拟机机器上设置服务器密钥：

    ```java
    sudo rm -rf /etc/ssh/ssh_host_*
    sudo cp vmguest__ssh_host_rsa_key.pub /etc/ssh/ssh_host_rsa_key.pub
    sudo cp vmguest__ssh_host_rsa_key /etc/ssh/ssh_host_rsa_key
    sudo chmod 600 /etc/ssh/ssh_host_rsa_key

    ```

1.  在虚拟机机器上设置主机客户端公钥：

    ```java
    cp vmhost__id_rsa.pub ~/.ssh/authorized_keys2

    ```

1.  在主机机器上，尝试连接到虚拟机并确认新的虚拟机服务器密钥：

    ```java
    ssh packt@192.168.42.1

    ```

1.  在主机机器上，保存获取的虚拟机服务器密钥指纹：

    ```java
    cp ~/.ssh/kno
    wn_hosts vmhost__known_hosts

    ```

1.  在主机机器上，检查我们是否现在可以从主机连接到虚拟机而无需任何密码或额外确认：

    ```java
    ssh packt@192.168.42.1

    ```

1.  重复步骤 7 到 11，交换主机和虚拟机两端以设置从虚拟机到主机的连接。

1.  保存以下密钥以供构建过程中使用：

    +   `vmhost__id_rsa`: 这是主机客户端私钥

    +   `vmhost__id_rsa.pub`: 这是主机客户端公钥

    +   `vmhost__ssh_host_rsa_key`: 这是主机服务器的私钥

    +   `vmhost__ssh_host_rsa_key.pub`: 这是主机服务器的公钥

    +   `vmhost__known_hosts`: 这是主机上要使用的虚拟机服务器密钥指纹

    +   `vmguest__id_rsa`: 这是虚拟机客户端私钥

    +   `vmguest__id_rsa.pub`: 这是虚拟机客户端私钥

    +   `vmguest__ssh_host_rsa_key`: 这是虚拟机服务器的私钥

    +   `vmguest__ssh_host_rsa_key.pub`: 这是虚拟机服务器的公钥

    +   `vmguest__known_hosts`: 这是虚拟机上要使用的主机服务器密钥指纹

## 工作原理...

`ssh-keygen`命令生成一对非对称加密（在我们的例子中，RSA）密钥。

SSH 支持基于密钥的无密码身份验证。我们准备了一套密钥，可以加载到主机和虚拟机端（对于所有虚拟机），以允许主机到虚拟机和虚拟机到主机的无缝连接。因此，现在我们可以在主机机器上调用一个脚本，该脚本将连接到（或发送文件）到虚拟机，并能够从相同的虚拟机会话连接回主机。

所有密钥都是故意使用空密码生成的，以便无需手动输入密码即可建立连接。

## 更多内容...

SSH 连接是安全的，如果你想要使用远程机器而不是本地虚拟机进行构建，这可能会很有用。如果不需要安全，则可以使用其他协议。它们不需要身份验证或密钥设置，例如，一些支持命令和发送文件的 HTTP 自定义协议。

可以使用 DSA 或 ECDSA 密钥代替 RSA 密钥。

可以使用像`expect`这样的 shell 自动化工具设置带密码的自动连接，而不是客户端密钥。

## 参见

+   *安装 VirtualBox*说明

+   OpenSSH 关于密钥生成的手册，可在[`www.virtualbox.org/manual/UserManual.html`](https://www.virtualbox.org/manual/UserManual.html)找到

# 准备 Linux 虚拟机

许多基于 Linux 的操作系统都支持使用 VirtualBox 进行虚拟化，并且它们通常在主要软件包仓库中预安装或提供 OpenSSH 客户端和服务器。

在这个菜谱中，我们将设置一个 Ubuntu Linux 虚拟机，它可以用于自动化 OpenJDK 构建。

## 准备工作

对于这个菜谱，我们需要安装了 VirtualBox 并配置了虚拟网络接口的 Ubuntu 12.04 amd64 操作系统。

## 如何做...

以下程序将帮助我们准备 Linux 虚拟机：

1.  按照菜谱 *准备 SSH 密钥* 中所述准备 SSH 密钥。

1.  从 Ubuntu 网站下载 Ubuntu 12.04 服务器 amd64 镜像（[`www.ubuntu.com/`](http://www.ubuntu.com/））。

1.  在 VirtualBox 中，使用 IDE 存储控制器和其他设置的默认值创建一个虚拟机实例。

1.  在虚拟机上安装 Ubuntu，按照菜谱 *安装 VirtualBox* 中所述设置网络，并启动虚拟机。

1.  在主机机器上创建一个具有相同名称的用户，并在此用户下登录：

    ```java
    sudo adduser packt

    ```

1.  设置客户端密钥：

    ```java
    cp vmguest__id_rsa.pub ~/.ssh/id_rsa.pub
    cp vmguest__id_rsa ~/.ssh/id_rsa
    chmod 600 ~/.ssh/id_rsa

    ```

1.  设置服务器密钥：

    ```java
    sudo rm -rf /etc/ssh/ssh_host_*
    sudo cp vmguest__ssh_host_rsa_key.pub /etc/ssh/ssh_host_rsa_key.pub
    sudo cp vmguest__ssh_host_rsa_key /etc/ssh/ssh_host_rsa_key
    sudo chmod 600 /etc/ssh/ssh_host_rsa_key

    ```

1.  设置主机客户端公钥：

    ```java
    cp vmhost__id_rsa.pub ~/.ssh/authorized_keys2

    ```

1.  设置主机密钥指纹：

    ```java
    cp vmguest__known_hosts ~/.ssh/known_hosts

    ```

1.  检查从主机到客户机和返回的连接是否无缝工作：

    ```java
    ssh packt@192.168.42.1

    ```

1.  使用来自 第四章 的 *Building OpenJDK 8 Ubuntu Linux 12.04 LTS* 菜谱完成 OpenJDK 的手动构建。

## 它是如何工作的...

在这个菜谱中，我们创建了一个带有 Ubuntu Linux 的虚拟机实例，并配置了 SSH 密钥以实现到它的无缝自动化连接。

在此 VM 上进行了手动构建，以确保所有环境都正确。

## 还有更多...

可以使用其他基于 Linux 的操作系统代替 Ubuntu 12.04。可以使用其他协议/工具代替 OpenSSH 在主机和客户机之间进行交互。

## 参见

+   来自 第四章 的 *Building OpenJDK 8 Ubuntu Linux 12.04 LTS* 菜谱，*Building OpenJDK 8*

+   *安装 VirtualBox* 菜谱

+   *准备 SSH 密钥* 菜谱

+   Oracle VirtualBox 用户手册 [`www.virtualbox.org/manual/UserManual.html`](https://www.virtualbox.org/manual/UserManual.html)

# 准备带有 Mac OS X 的 VirtualBox 机器

Mac OS X 操作系统的现代版本支持在虚拟化环境中使用 VirtualBox 运行。准备 Mac OS X 镜像以进行虚拟化的说明可能因 Mac OS X 版本和主机操作系统而大相径庭。确切说明超出了本书的范围，可以在互联网上找到。

请注意，在非 Mac 主机操作系统上运行客户机 Mac OS X 可能违反您与 Apple Inc. 的最终用户许可协议，最好咨询您的律师。

Mac OS X 预装了 OpenSSH 客户端和服务器。

## 准备工作

对于这个菜谱，我们需要一个可用于 VirtualBox-VDI 的已准备好的 Mac OS X 镜像（版本 10.7 或更高）。

## 如何操作...

以下步骤将帮助我们准备 Mac OS X 虚拟机：

1.  按照本章中 *准备 SSH 密钥* 菜谱中的说明准备 SSH 密钥。

1.  在 VirtualBox 中，创建一个至少拥有 2048 RAM、PIIX3 芯片组、禁用 UEFI、单 CPU 和 IDE 存储控制器的虚拟机实例。

1.  按照本章中 *安装 VirtualBox* 菜谱中的说明设置网络，并启动虚拟机。

1.  在主机机器上创建一个具有相同名称的用户，并以此用户身份登录。

1.  设置客户端密钥：

    ```java
    cp vmguest__id_rsa.pub ~/.ssh/id_rsa.pub
    cp vmguest__id_rsa ~/.ssh/id_rsa
    chmod 600 ~/.ssh/id_rsa

    ```

1.  设置服务器密钥：

    ```java
    sudo rm -rf /etc/ssh_host_*
    sudo cp vmguest__ssh_host_rsa_key.pub /etc/ssh_host_rsa_key.pub
    sudo cp vmguest__ssh_host_rsa_key /etc/ssh_host_rsa_key
    sudo chmod 600 /etc/ssh_host_rsa_key

    ```

1.  设置主机客户端公钥：

    ```java
    cp vmhost__id_rsa.pub ~/.ssh/authorized_keys2

    ```

1.  设置主机密钥指纹：

    ```java
    cp vmguest__known_hosts ~/.ssh/known_hosts

    ```

1.  检查从主机到客户机和返回的连接是否无缝工作：

    ```java
    ssh packt@192.168.42.1

    ```

1.  使用来自第四章 *在 Mac OS X 上构建 OpenJDK 8* 的 *构建 OpenJDK 8 on Mac OS X* 菜谱完成 OpenJDK 的手动构建。

## 它是如何工作的...

在这个菜谱中，我们创建了一个带有 Mac OS X 的虚拟机实例，并配置了 SSH 密钥以实现无缝的自动化连接。

手动构建是在这个虚拟机上完成的，以确保环境设置正确。

## 更多内容...

在某些 Mac OS X 版本中，预装的 OpenSSH 可能不支持 ECDSA SSH 密钥。尽管如此，我们仍然可以使用 RSA SSH 密钥完成这个菜谱。但是，如果你想使用 ECDSA 密钥，你可以相对容易地使用 Homebrew 打包系统和其系统副本仓库 `homebrew/dupes` 更新 OpenSSH 安装。

可以使用其他协议/工具在主机和客户机之间进行交互，而不是 OpenSSH。

## 相关链接

+   来自第四章 *在 Mac OS X 上构建 OpenJDK 8* 的 *构建 OpenJDK 8 on Mac OS X* 菜谱 Chapter 4，*构建 OpenJDK 8*

+   *安装 VirtualBox* 菜谱

+   *准备 SSH 密钥* 菜谱

+   Oracle VirtualBox 用户手册，[`www.virtualbox.org/manual/UserManual.html`](https://www.virtualbox.org/manual/UserManual.html)

# 准备带有 Windows 的 VirtualBox 机器

流行的虚拟化工具对 Windows 的虚拟化支持非常好。与 Mac OS X 相比，Windows 的 VirtualBox 设置可能更容易。然而，在 Unix-like 操作系统上，SSH 协议比在 Windows 上更受欢迎，Windows 上的 SSH 服务器设置可能更复杂。

在这个菜谱中，你将学习如何设置 Windows 虚拟机以进行自动化构建。关于在 Windows 上配置免费 SSH 服务器的一套深入指导将构成菜谱的重要部分。

## 准备工作

对于这个菜谱，我们需要一个 Windows 7 虚拟机 VirtualBox 镜像。

## 如何操作...

以下步骤将帮助我们准备 Windows 虚拟机：

1.  按照本章中 *准备 SSH 密钥* 菜谱中的说明准备 SSH 密钥。

1.  下载 Copssh SSH 服务器实现版本 3.1.4。不幸的是，它已被作者从公共下载中移除，但仍然可以在互联网上找到，以下是一些文件详细信息：

    ```java
    Copssh_3.1.4_Installer.exe
    size: 5885261 bytes
    sha1: faedb8ebf88285d7fe3e141bf5253cfa70f94819

    ```

1.  使用默认安装参数将 Copssh 安装到任何 Windows 实例中，并将安装的文件复制到某处以供以后使用。

1.  在 VirtualBox 中，使用 IDE 存储控制器和其他设置的默认值创建虚拟机实例。

1.  按照本章中*安装 VirtualBox*配方中的说明设置网络，并启动虚拟机。

1.  在主机机器上创建具有相同名称的用户，并在此用户下登录（我们将使用名称`packt`）。

1.  从 Microsoft 网站下载 Windows Server 2003 资源工具包，并从中提取`ntrights`、`instsrv`和`srvany`实用程序。

1.  将提取的 Copssh 文件复制到`c:\ssh`目录。

1.  使用以下脚本设置 SSH 服务的用户和权限：

    ```java
    net user sshd sshd /ADD
    net user SvcCOPSSH SvcCOPSSH /ADD
    net localgroup Administrators SvcCOPSS
    H /add
    ntrights +r SeTcbPrivilege -u SvcCOPSSH
    ntrights +r SeIncreaseQuotaPrivilege -u SvcCOPSSH
    ntrights +r SeCreateTokenPrivilege -u SvcCOPSSH
    ntrights +r SeServiceLogonRight -u SvcCOPSSH
    ntrights +r SeAssignPrimaryTokenPrivilege -u SvcCOPSSH
    ntrights +r SeDenyInteractiveLogonRight -u SvcCOPSSH
    ntrights +r SeDenyNetworkLogonRight -u SvcCOPSSH

    ```

1.  生成内部 Copssh 登录和密码信息，并将 Copssh 注册为 Windows 服务：

    ```java
    c:\copssh\bin\mkpasswd -l > c:\obf\copssh\etc\passwd
    c:\copssh\bin\cygrunsrv.exe --install OpenSSHServer --args "-D" --path /bin/sshd
     --env "CYGWIN=binmode ntsec tty" -u SvcCOPSSH -w SvcCOPSSH

    ```

1.  安装 Cygwin 工具并运行 bash shell。

1.  设置客户端密钥：

    ```java
    cp vmguest__id_rsa.pub /cygdrive/c/ssh/home/packt/.ssh/id_rsa.pub
    cp vmguest__id_rsa /cygdrive/c/ssh/home/packt/.ssh/id_rsa
    chmod 600 /cygdrive/c/ssh/home/packt/.ssh/id_rsa

    ```

1.  设置服务器密钥：

    ```java
    rm -rf /cygdrive/c/ssh/etc/ssh_host_*
    cp vmguest__ssh_host_rsa_key.pub /cygdrive/c/ssh/etc/ssh_host_rsa_key.pub
    cp vmguest__ssh_host_rsa_key /cygdrive/c/ssh/etc/ssh_host_rsa_key
    chmod 600 /cygdrive/c/ssh/etc/ssh_host_rsa_key

    ```

1.  设置主机客户端公钥：

    ```java
    cp vmhost__id_rsa.pub /cygdrive/c/ssh/home/packt/.ssh/authorized_keys2

    ```

1.  设置主机密钥指纹：

    ```java
    cp vmguest__known_hosts /cygdrive/c/ssh/home/packt/.ssh/known_hosts

    ```

1.  检查从主机到虚拟机以及返回的连接是否无缝工作：

    ```java
    ssh packt@192.168.42.1

    ```

1.  使用`instsrv`和`srvany`实用程序注册将用于启动构建过程的 Windows 服务：

    ```java
    instsrv.exe packt_build c:\path\to\srvany.exe
    reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\packt_build\Parameters" /v Application /t reg_sz /d "C:\packt\build.bat"

    ```

1.  配置服务以避免在操作系统启动时自动启动：

    ```java
    sc config obf_build start= demand

    ```

1.  使用第四章中*在 Windows 7 SP1 上构建 OpenJDK 8*配方完成 OpenJDK 的手动构建。

## 它是如何工作的...

在此配方中，我们创建了一个带有 Windows 的虚拟机实例，并配置了 SSH 服务器以启用到它的无缝自动化连接。

Copssh SSH 服务器版本 3.1.4 的二进制文件由作者在 GNU 通用公共许可证版本 3 的条款下作为免费软件发布。这意味着我们可以为任何目的发布或使用未经更改的二进制文件，而无需额外的许可限制。

Copssh 使用 Cygwin 环境和 OpenSSH 服务器作为底层。它还提供了与 Windows 用户权限系统的集成。

授予 SvcCOPSSH 用户的授权角色是支持 SSH 密钥认证所必需的。

我们使用 Cygwin 设置密钥文件以支持 Cygwin 文件权限的正确设置。

Copssh 使用了 Cygwin 环境的旧版本，我们需要额外的完整 Cygwin 安装来支持 OpenJDK 构建过程。同一台机器上运行的不同版本的 Cygwin 可能会相互干扰，导致错误。尽管在重用此类设置期间，我从未观察到任何问题，但最好记住这一点，以防出现神秘的 Cygwin/Copssh 错误/崩溃。

Copssh 内部使用两个额外的 Windows 用户（sshd 和 SvcCOPSSH）用于 SSH 服务器。

使用`ntrighs`实用程序为 SvcCOPSSH 用户分配了额外的角色。此实用程序在新版本的 Windows 中未得到官方支持，但应该仍然可以正常工作。

对于自动化构建，需要 Windows 服务注册来通过 SSH 连接启动实际的构建过程。为了设置适当的环境，Windows 上的构建过程应该从 `cmd.exe` shell（通常运行批处理文件）启动。不能直接从在 Cygwin 环境中的客户机 Windows 机器内的 SSH 会话启动。`instsrv` 和 `ntrighs` 工具允许我们创建一个 Windows 服务，该服务将在注册表中预先配置的路径上运行批处理文件（进而启动实际的构建过程）。这个 `packt_build` 服务可以通过 `net start` 命令从 SSH 会话启动，从而有效地启动构建过程。

在这个虚拟机上进行的手动构建是为了确保环境设置正确。

## 更多...

理论上可以使用其他 SSH 服务器，尽管我不了解其他适用于 Windows 的免费（如“言论自由”）SSH 服务器实现，它支持密钥认证。

可以使用其他协议/工具来代替 OpenSSH 在主机和客户机之间进行交互。

## 参见

+   来自第四章 Building OpenJDK 8 的 *在 Windows 7 SP1 上构建 OpenJDK 8* 脚本

+   来自第二章 Building OpenJDK 6 的 *为 Windows 构建 Cygwin 的安装脚本*

+   *安装 VirtualBox* 脚本

+   *准备 SSH 密钥* 脚本

+   Oracle VirtualBox 用户手册位于 [`www.virtualbox.org/manual/UserManual.html`](https://www.virtualbox.org/manual/UserManual.html)

+   Copssh 网站 [`www.itefix.net/copssh`](https://www.itefix.net/copssh)

# 自动化构建

这个脚本将本章中所有之前的脚本合并在一起。准备好的虚拟机镜像和带有密钥认证的 SSH 将允许我们使用简单的 bash 脚本在完全自动化的模式下构建 OpenJDK，而无需额外的工具。

## 准备工作

对于这个脚本，我们需要一个运行 Linux 或 Mac OS 的主机机器。

## 如何做到这一点...

以下步骤将帮助我们准备 Windows 虚拟机：

1.  按照本章中 *准备 SSH 密钥* 脚本所述准备 SSH 密钥。

1.  按照本章中 *安装 VirtualBox* 脚本所述设置 VirtualBox 安装及其网络设置。

1.  按照本章之前 recipes 中所述准备虚拟机镜像。

1.  对于每个 VM 映像，准备一个环境变量列表，这些变量将由构建脚本使用（例如，Windows）：

    ```java
    export VM_ADDRESS=192.168.42.1
    export VM_NAME=jdk7-windows-amd64
    export VM_OSTYPE=Windows7_64
    export VM_MEMORY=1780
    export VM_IOAPIC=on
    export VM_NICTYPE=82545EM
    export VM_MACADDR=auto
    export VM_OBF_DIR=/cygdrive/c/packt
    export VM_START_BUILD="net start obf_build >> build.log 2>&1"
    export VM_SHUTDOWN="shutdown /L /T:00 /C /Y"
    export VM_IDE_CONTROLLER=PIIX4

    ```

1.  将以下步骤（第 6 步至第 12 步）的片段添加到主构建脚本中。

1.  使用 `VBoxManage` 工具创建虚拟机实例：

    ```java
    SCRIPT_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
    VBoxManage createvm --name "$VM_NAME" --register --basefolder "$SCRIPT_DIR"/target >> "$SCRIPT_DIR"/build.log 2>&1
    VBoxManage modifyvm "$VM_NAME" --ostype "$VM_OSTYPE" >> "$SCRIPT_DIR"/build.log 2>&1
    VBoxManage modifyvm "$VM_NAME" --memory "$VM_MEMORY" >> "$SCRIPT_DIR"/build.log 2>&1
    VBoxManage modifyvm "$VM_NAME" --nic1 bridged --bridgeadapter1 tap0 >> "$SCRIPT_DIR"/build.log 2>&1
    VBoxManage modifyvm "$VM_NAME" --nictype1 "$VM_NICTYPE" >> "$SCRIPT_DIR"/build.log 2>&1
    VBoxManage modifyvm "$VM_NAME" --macaddress1 "$VM_MACADDR" >> "$SCRIPT_DIR"/build.log 2>&1
    VBoxManage modifyvm "$VM_NAME" --cpus 1 >> "$SCRIPT_DIR"/build.log 2>&1
    VBoxManage modifyvm "$VM_NAME" --audio none >> "$SCRIPT_DIR"/build.log 2>&1
    VBoxManage modifyvm "$VM_NAME" --usb off >> "$SCRIPT_DIR"/build.log 2>&1
    VBoxManage modifyvm "$VM_NAME" --vrde on
    VBoxManage modifyvm "$VM_NAME" --ioapic "$VM_IOAPIC" >> "$SCRIPT_DIR"/build.log 2>&1
    VBoxManage modifyvm "$VM_NAME" --mouse usbtablet >> "$SCRIPT_DIR"/build.log 2>&1
    VBoxManage modifyvm "$VM_NAME" --keyboard usb >> "$SCRIPT_DIR"/build.log 2>&1
    VBoxManage setextradata global GUI/SuppressMessages remindAboutAutoCapture,remindAboutMouseIntegrationOn,showRuntimeError.warning.HostAudioNotResponding,remindAboutGoingSeamless,remindAboutInputCapture,remindAboutGoingFullscreen,remindAboutMouseIntegrationOff,confirmGoingSeamless,confirmInputCapture,remindAboutPausedVMInput,confirmVMReset,confirmGoingFullscreen,remindAboutWrongColorDepth >> "$SCRIPT_DIR"/build.log 2>&1
    VBoxManage storagectl "$VM_NAME" --name "IDE" --add ide >> "$SCRIPT_DIR"/build.log 2>&1
    VBoxManage internalcommands sethduuid "$SCRIPT_DIR"/target/$VM_NAME.vdi >> "$SCRIPT_DIR"/build.log 2>&1
    VBoxManage storageattach "$VM_NAME" --storagectl "IDE" --port 0 --device 0 --type hdd --medium "$SCRIPT_DIR"/target/"$VM_NAME".vdi >> "$SCRIPT_DIR"/build.log 2>&1
    VBoxManage storagectl "$VM_NAME" --name "IDE" --controller "$VM_IDE_CONTROLLER"

    ```

1.  启动虚拟机实例：

    ```java
    VBoxManage startvm "$VM_NAME" --type headless >> "$SCRIPT_DIR"/build.log 2>&1
    ssh "$VM_ADDRESS" "ls" > /dev/null 2>&1
    while [ $? -ne 0 ]; do
    echo "Waiting for VM ..."
     sleep 10
     ssh "$VM_ADDRESS" "ls" > /dev/null 2>&1
    done
    echo "VM started"

    ```

1.  启用通过 SSH 将远程日志记录回主机机器：

    ```java
    ssh "$VM_ADDRESS" "cd "$VM_OBF_DIR" && echo 'Starting build' > build.log"
    nohup ssh "$VM_ADDRESS" "tail -f "$VM_OBF_DIR"/build.log | ssh 192.168.42.2 'cat >> "$SCRIPT_DIR"/build.log'" >> "$SCRIPT_DIR"/build.log 2>&1 &
    LOGGER_PID="$!"

    ```

1.  将 OpenJDK 源代码复制到构建 VM 并开始构建：

    ```java
    scp "$SCRIPT_DIR"/openjdk.zip "$VM_ADDRESS":"$VM_OBF_DIR"
    ssh "$VM_ADDRESS" "cd "$VM_OBF_DIR" && unzip -q openjdk.zip >> build.log 2>&1"
    ssh "$VM_ADDRESS" "cd "$VM_OBF_DIR" && "$VM_START_BUILD""

    ```

1.  定期轮询构建机器，寻找在构建完成后应该创建的 `build_finished.flag` 文件：

    ```java
    ssh "$VM_ADDRESS" "if [ ! -f "$VM_OBF_DIR"/build_finished.flag ]; then exit 1; else exit 0; fi" > /dev/null 2>&1
    while [ $? -ne 0 ]; do
     echo "Waiting for build ..."
     sleep 300
    ssh "$VM_ADDRESS" "if [ ! -f "$VM_OBF
    _DIR"/build_finished.flag ]; then exit 1; else exit 0; fi" > /dev/null 2>&1
    done

    ```

1.  复制构建结果，停止记录器，关闭虚拟机实例，并在 VirtualBox 中注销它：

    ```java
    scp -r "$VM_ADDRESS":"$VM_OBF_DIR"/dist/* "$SCRIPT_DIR"/dist
    kill -9 $LOGGER_PID >> "$SCRIPT_DIR"/build.log 2>&1
    ssh "$VM_ADDRESS" "$VM_SHUTDOWN" >> "$SCRIPT_DIR"/build.log 2>&1 || true
    sleep 15
    VBoxManage controlvm "$VM_NAME" poweroff > /dev/null 2>&1 || true
    VBoxManage unregistervm "$VM_NAME" >> "$SCRIPT_DIR"/build.log

    ```

1.  要使用所选的虚拟机镜像启动构建，请使用以下命令：

    ```java
    . windows_amd64.env # please not the dot before the command
    nohup build.sh >> build.log 2>&1 &
    echo "$!" > .pid
    tail -F "$SCRIPT_DIR"/build.log

    ```

1.  构建完成后，OpenJDK 二进制文件将被复制回主机机器。

## 它是如何工作的...

我们使用低级 VBoxManage 工具来操作虚拟机实例，以便更好地控制此过程。

为了确保虚拟机实例实际上启动了，我们通过 SSH 定期检查它，并在第一次成功连接后停止检查。

对于远程记录，我们在构建机器上运行`tail -f`进程，该进程立即通过 SSH 将输出发送回主机机器。我们使用`nohup`工具在主机机器的背景中使用连接启动此进程，并将主机进程`pid`写入`.pid`文件以在构建后终止进程。

我们使用`scp`将源代码复制到构建机器，并使用 SSH 命令解压缩源代码和启动构建。

构建开始后，我们通过 SSH 定期检查构建机器，以查找构建脚本应在构建机器上创建的`build_finished.flag`文件。

构建完成后，我们将 OpenJDK 二进制文件复制回主机机器，并在注销之前优雅地关闭虚拟机实例。

每个虚拟机镜像的`.env`文件中列出了不同的虚拟机配置和环境选项。我们使用“点优先”语法从`env`文件导入变量到当前 shell。这允许我们为所有虚拟机镜像使用通用的构建脚本。

## 更多内容...

这个脚本可以被视为构建自动化的基本示例。不同的命令、工具和协议可以用来达到相同的目标。

`build_finished.flag` 文件（包含自定义内容）也可以在出现错误后提前结束构建。

## 参见

+   *安装 VirtualBox* 脚本

+   *准备 SSH 密钥* 脚本

+   *准备带有 Linux 的 VirtualBox 机器* 脚本

+   *准备带有 Mac OS X 的 VirtualBox 机器* 脚本

+   *准备带有 Windows 的 VirtualBox 机器* 脚本

+   第四章, *构建 OpenJDK 8*

+   Oracle VirtualBox 用户手册，[`www.virtualbox.org/manual/UserManual.html`](https://www.virtualbox.org/manual/UserManual.html)

# 构建跨平台安装程序

当云服务成为安装桌面软件的普遍方式时，经典的 GUI 安装程序几乎变得过时。云包仓库或商店可以安装和首先更新桌面应用程序更加方便。

同时，GUI 安装程序在许多免费和商业应用程序中仍然被广泛使用。特别是对于跨平台应用程序，尽管在那些平台上不是完全本地的，GUI 安装程序应该在所有支持的平台上显示相同的行为。一些应用程序在安装时需要复杂的环境更改，例如，将自己注册为 Windows 服务或设置环境变量。

在这个配方中，我们将准备一个适用于所有支持平台（Windows、Linux 和 Mac OS X）的 OpenJDK 跨平台安装程序。我们将使用一个流行的开源安装工具`IzPack`，它是用 Java 编写的。

## 准备工作

对于这个配方，我们将需要 OpenJDK 的二进制文件（用于包装到安装程序中）。

## 如何操作...

以下步骤将帮助我们准备安装程序：

1.  从 IzPack 网站（[`izpack.org/`](http://izpack.org/)）下载 IzPack 编译器版本 4.3.5 并安装它。

1.  从 Izpack 网站的文档部分下载示例安装配置文件。

1.  将`jre`目录从 OpenJDK 镜像中上移一级，紧邻`openjdk`目录。

1.  使用以下配置片段将`jre`目录添加到安装程序配置中，作为一个“松散”包：

    ```java
    <pack name="OpenJDK RE" required="yes" loose="true">
        <description>OpenJDK Runtime Environment</description>
        <file src="img/jre" targetdir="$INSTALL_PATH"/>
    </pack>
    ```

1.  将`openjdk`目录（现在不包含 JRE）作为一个普通包添加：

    ```java
    <pack name="OpenJDK DK" required="no">
        <description>OpenJDK Development Kit</description>
        <fileset dir="openjdk" targetdir="$INSTALL_PATH"/>
        <file src="img/uninstall" targetdir="$INSTALL_PATH"/>
    </pack>
    ```

1.  根据您的喜好调整标签、GUI 表单、区域设置和图标。

1.  运行 IzPack 编译器：

    ```java
    ./izcomp/bin/compile ./config.xml -h ./izcomp -o installer.jar

    ```

1.  将生成的`install.jar`和`jre`目录放入`openjdk-installer`目录中。

1.  将 bash/batch 脚本添加到`openjdk-installer`目录中，这将允许我们使用`jre`目录的相对路径来运行安装程序：

    ```java
    ./jre/bin/java -jar install.jar

    ```

1.  压缩`openjdk-installer`目录——它现在包含 OpenJDK 安装程序。

## 它是如何工作的...

我们安装程序的主要特点是安装程序本身运行在它将要安装的相同版本的 Java 上。JRE 包中的`loose="true"`配置指示安装程序在主安装`.jar`文件之外的相对路径上查找此包，而不复制`jre`目录的内容。

## 更多内容...

IzPack 安装程序支持许多配置选项，我们在此配方中仅突出显示了基本选项。除了 GUI 和安装表单的自定义之外，一个特别有用的功能，特别是对于 OpenJDK，是在安装时运行脚本。我们可以准备脚本以调整环境，并使用可执行配置元素将它们添加到相应的包中。在简单的类 Unix 操作系统上，这样的脚本可以简单地将`PATH`变量更改追加到`~/.bashrc`或`~/.bash_profile`文件中。在 Windows 上，可以使用`pathman`、`setx`和`reg`等实用程序来调整环境变量或 Windows 注册表。

在安装时运行脚本，而不是直接从 Java 代码中扩展 IzPack 本身（添加新表单等）并直接执行环境注册。

## 参考信息

+   第二章, *构建 OpenJDK 6*

+   第三章, *构建 OpenJDK 7*

+   第四章, *构建 OpenJDK 8*

+   IzPack 安装程序网站位于[`izpack.org/`](http://izpack.org/)
