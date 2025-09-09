# 第十二章 生产环境下的玩耍

由于受安全、负载/流量（预期处理）、网络问题等各种因素的影响，应用程序部署、配置等在生产环境中略有不同。在本章中，我们将了解如何使我们的 Play 应用程序在生产环境中运行。本章涵盖了以下主题：

+   部署应用程序

+   生产环境配置

+   启用 SSL

+   使用负载均衡器

# 部署 Play 应用程序

Play 框架提供了用于在生产环境中打包和部署 Play 应用程序的命令。

我们之前使用的`run`命令以`DEV`模式启动应用程序并监视代码的变化。当代码发生变化时，应用程序将被重新编译和重新加载。在开发过程中保持警觉是很有用的，但在生产环境中这会是一个不必要的开销。此外，`PROD`模式下显示的默认错误页面与`DEV`模式下显示的不同，即它们关于发生错误的详细信息较少（出于安全原因）。

让我们看看在生产环境中我们可以以不同方式部署应用程序。

## 使用 start 命令

要以`PROD`模式启动应用程序，我们可以使用`start`命令：

```java
[PlayScala] $ start
[info] Wrote /PlayScala/target/scala-2.10/playscala_2.10-1.0.pom

(Starting server. Type Ctrl+D to exit logs, the server will remain in background)

Play server process ID is 24353
[info] play - Application started (Prod)
[info] play - Listening for HTTP on /0:0:0:0:0:0:0:0:9000

```

进程 ID 可以在以后用来停止应用程序。通过按*Ctrl* + *D*，我们不会丢失日志，因为默认情况下它们也被捕获在`logs/application.log`中（即，当日志配置没有变化时）。

`start`命令可以可选地接受应用程序部署的端口号：

```java
[PlayScala] $ start 9123
[info] Wrote /PlayScala/target/scala-2.10/playscala_2.10-1.0.pom

(Starting server. Type Ctrl+D to exit logs, the server will remain in background)

Play server process ID is 12502
[info] play - Application started (Prod)
[info] play - Listening for HTTP on /0:0:0:0:0:0:0:0:9123

```

## 使用发行版

虽然`start`命令足以部署应用程序，但在需要应用程序的可移植版本的情况下，可能不足以满足需求。在本节中，我们将了解如何构建我们应用程序的独立发行版。

Play 框架支持使用`sbt-native-packager`插件（参考[`www.scala-sbt.org/sbt-native-packager/`](http://www.scala-sbt.org/sbt-native-packager/)）构建应用程序的发行版。该插件可以用来创建`.msi`（Windows）、`.deb`（Debian）、`.rpm`（Red Hat 软件包管理器）和`.zip`（通用）文件，以及我们应用程序的 Docker 镜像。该插件还支持在应用程序的构建文件中定义包的设置。其中一些设置是通用的，而其他的是特定于操作系统的。以下表格显示了通用设置：

| 设置 | 目的 | 默认值 |
| --- | --- | --- |
| `packageName` | 创建的输出包的名称（不带扩展名） | 从混合大小写和空格转换为小写和短横线分隔的项目名称 |
| `packageDescription` | 包的描述 | 项目名称 |
| `packageSummary` | Linux 软件包内容的摘要 | 项目名称 |
| `executableScriptName` | 执行脚本的名称 | 将项目名称从混合大小写和空格转换为小写和短横线分隔的名称 |
| `maintainer` | 本地软件包维护者的名称/电子邮件地址 |   |

现在，让我们看看如何为不同的操作系统构建软件包并使用它们。

### 通用发行版

通用发行版与所有/大多数操作系统兼容。生成的软件包位于 `projectHome/target/universal`。我们可以使用以下任一命令根据需要创建软件包：

+   `universal:packageBin` – 此命令创建打包应用的 `appname-appVersion.zip` 文件

+   `universal:packageZipTarball` – 此命令创建打包应用的 `appname-appVersion.tgz` 文件

+   `universal:packageOsxDmg` – 此命令创建打包应用的 `appname-appVersion.dmg` 文件（该命令仅在 OS X 上有效）

### 注意

`universal:packageZipTarball` 命令需要 `gzip`、`xz` 和 `tar` 命令行工具，而 `universal:packageOsxDmg` 命令则需要 OS X 或安装了 `hdiutil` 的系统。

要使用通过这些命令构建的软件包，提取文件并对于基于 Unix 的系统执行 `bin/appname`，对于带有 Windows 的系统执行 `bin/appname.bat`。

### 注意

在 Play 应用程序中，我们可以使用 `dist` 命令代替 `universal:packageBin`。`dist` 命令会删除使用 `universal:packageBin` 命令打包应用时创建的不必要中间文件。

### Debian 发行版

我们可以使用 `debian:packageBin` 命令创建可在基于 Debian 的系统上安装的发行版。`.deb` 文件位于 `projectHome/target`。

### 注意

要构建 Debian 软件包，应在 `build` 文件中设置 Debian 设置中的 `packageDescription` 值。其他 Debian 软件包设置也可以在 `build` 文件中设置。

打包完成后，我们可以使用 `dpkg-deb` 命令安装应用：

```java
projectHome$ sudo dpkg -i target/appname-appVersion.deb

```

安装完成后，我们可以通过执行以下命令来启动应用：

```java
$ sudo appname

```

### rpm 发行版

可以使用 `rpm:packageBin` 命令创建应用的 `rpm` 软件包。以下表格显示了 `rpm` 软件包的一些可用设置：

| 设置 | 目的 |
| --- | --- |
| `rpmVendor` | 本 `rpm` 软件包的供应商名称 |
| `rpmLicense` | `rpm` 软件包内代码的许可协议 |
| `rpmUrl` | 应包含在 `rpm` 软件包中的 URL |
| `rpmDescription` | 本 `rpm` 软件包的描述 |
| `rpmRelease` | 本 `rpm` 软件包的特殊发布号 |

### 注意

在 `rpm` 中，`rpmVendor` 的值、`rpm` 中的 `packageSummary` 和 `rpm` 中的 `packageDescription` 必须在 `build` 文件中设置，才能成功创建应用 `rpm` 软件包，其中 `rpm` 是作用域，例如 `rpm:= "SampleProject"` 中的名称。

生成 `rpm` 软件包后，我们可以使用 `yum` 或等效工具安装它：

```java
projectHome$ sudo yum install target/appname-appVersion.rpm

```

安装完成后，我们可以通过执行以下命令来启动应用：

```java
$ sudo appname

```

### Windows 发行版

可以使用 `windows:packageBin` 命令创建应用程序的 Windows 安装程序，`appname-appVersion.msi`。文件位于 `projectHome/target`。

# 生产环境配置

Play 框架理解应用程序在部署到生产之前可能需要更改配置。为了简化部署，部署应用程序的命令也接受应用程序级别的配置作为参数：

```java
[PlayScala] $ start -Dapplication.secret=S3CR3T
[info] Wrote /PlayScala/target/scala-2.10/playscala_2.10-1.0.pom

(Starting server. Type Ctrl+D to exit logs, the server will remain in background)

Play server process ID is 14904

```

让我们按照以下方式更改应用程序的 HTTP 端口：

```java
#setting http port to 1234
[PlayScala] $ start -Dhttp.port=1234

```

在某些项目中，生产环境和开发环境的配置被保存在两个独立的文件中。我们可以传递一个或多个配置，或者完全传递一个不同的文件。有三种明确指定配置文件的方式。可以通过以下选项之一实现：

+   `config.resource`: 当文件位于类路径中（`application/conf` 中的文件）时使用此选项

+   `config.file`: 当文件在本地文件系统中可用，但与应用程序资源捆绑在一起时使用此选项

+   `config.url`: 当文件需要从 URL 加载时使用此选项

假设我们的应用程序在生产环境中使用`conf/application-prod.conf`，我们可以如下指定该文件：

```java
[PlayScala] $ start -Dconfig.resource=application-prod.conf

```

同样，我们也可以通过将 `config` 键替换为 `logger` 来修改日志配置：

```java
[PlayScala] $ start -Dlogger.resource=logger-prod.xml

```

我们还可以通过传递设置作为参数来配置底层的 Netty 服务器，这不可能通过 `application.conf` 实现。以下表格列出了可以在一种或多种方式中配置的与服务器相关的某些设置。

与地址和端口相关的属性如下：

| 属性 | 目的 | 默认值 |
| --- | --- | --- |
| `http.address` | 应用程序部署的地址 | `0.0.0.0` |
| `http.port` | 应用程序可用的端口 | `9000` |
| `https.port` | 应用程序可用的 `sslPort` 端口 |   |

与 HTTP 请求（`HttpRequestDecoder`）相关的属性如下：

| 属性 | 目的 | 默认值 |
| --- | --- | --- |
| `http.netty.maxInitialLineLength` | 初始行的最大长度（例如，`GET / HTTP/1.0`） | `4096` |
| `http.netty.maxHeaderSize` | 所有头部合并后的最大长度 | `8192` |
| `http.netty.maxChunkSize` | 主体或其每个分块的长度最大值。如果主体的长度超过此值，内容将被分成此大小或更小的块（最后一个块的情况）。如果请求发送分块数据，并且一个分块的长度超过此值，它将被分成更小的块。 | `8192` |

以下表格显示了与 TCP 套接字选项相关的属性：

| 属性 | 目的 | 默认值 |
| --- | --- | --- |
| `http.netty.option.backlog` | 队列中传入连接的最大大小 |   |
| `http.netty.option.reuseAddress` | 重新使用地址 |   |
| `http.netty.option.receiveBufferSize` | 接收缓冲区所使用的套接字大小。 |   |
| `http.netty.option.sendBufferSize` | 发送缓冲区所使用的套接字大小。 |   |
| `http.netty.option.child.keepAlive` | 保持连接活跃。 | `False` |
| `http.netty.option.child.soLinger` | 如果存在数据，则在关闭时保持等待。 | 负整数（禁用） |
| `http.netty.option.tcpNoDelay` | 禁用 Nagle 算法。TCP/IP 使用一种称为 Nagle 算法的算法来合并短数据段并提高网络效率。 | `False` |
| `http.netty.option.trafficClass` | **服务类型**（**ToS**）八位字节，位于**互联网协议**（IP）头部中。 | 0 |

# 启用 SSL

为我们的应用程序启用 SSL 有两种方式。我们可以在启动时提供所需的配置来提供 HTTPS 应用程序，或者通过代理请求通过启用 SSL 的 Web 服务器。在本节中，我们将了解如何使用第一种选项，而后者将在下一节中介绍。

我们可以选择运行 HTTP 和 HTTPS 版本，或者仅使用`http.port`和`https.port`设置选择其中之一。默认情况下，HTTPS 是禁用的，我们可以通过指定以下`https.port`来启用它：

```java
#setting https port to 1234
[PlayScala] $ start -Dhttps.port=1234

#disabling http port and setting https port to 1234
[PlayScala] $ start -Dhttp.port=disabled -Dhttps.port=1234

```

如果我们没有提供它们，Play 将生成自签名证书，并以启用 SSL 的方式启动应用程序。然而，这些证书不适合实际应用，我们需要使用以下设置指定密钥库的详细信息：

| 属性 | 目的 | 默认值 |
| --- | --- | --- |
| `https.keyStore` | 包含私钥和证书的密钥库的路径 | 此值是动态生成的 |
| `https.keyStoreType` | 密钥库类型 | **Java 密钥库**（**JKS**） |
| `https.keyStorePassword` | 密码 | 空密码 |
| `https.keyStoreAlgorithm` | 密钥库算法 | 平台的默认算法 |

此外，我们还可以通过`play.http.sslengineprovider`设置指定`SSLEngine`。此操作的前提是自定义的`SSLEngine`应该实现`play.server.api.SSLEngineProvider`特质。

### 注意

当启用了 SSL 的 Play 应用程序在生产环境中运行时，建议使用 JDK 1.8，因为 Play 使用 JDK 1.8 的一些功能来简化它。如果使用 JDK 1.8 不可行，则应使用启用 SSL 的反向代理。有关更多详细信息，请参阅[`www.playframework.com/documentation/2.3.x/ConfiguringHttps`](https://www.playframework.com/documentation/2.3.x/ConfiguringHttps)。

# 使用负载均衡器

处理大量流量的网站通常使用一种称为负载均衡的技术来提高应用程序的可用性和响应性。负载均衡器将传入流量分配到多个托管相同内容的服务器。负载分配由各种调度算法确定。

在本节中，我们将看到如何使用不同的 HTTP 服务器（假设它们运行在 IP `127.0.0.1`、`127.0.0.2` 和 `127.0.0.3` 的 `9000` 端口上）在我们的应用程序服务器前添加负载均衡器。

## Apache HTTP

Apache HTTP 服务器提供了一个安全、高效且可扩展的服务器，支持 HTTP 服务。Apache HTTP 服务器可以通过其 `mod_proxy` 和 `mod_proxy_balance` 模块用作负载均衡器。

要使用 Apache HTTP 作为负载均衡器，服务器中必须存在 `mod_proxy` 和 `mod_proxy_balancer`。要设置负载均衡器，我们只需更新 `/etc/httpd/conf/httpd.conf`。

让我们逐步更新配置：

1.  声明 `VirtualHost`：

    ```java
    <VirtualHost *:80>
    </VirtualHost>
    ```

1.  禁用 `VirtualHost` 的正向代理，以便我们的服务器不能用于掩盖源服务器的客户端身份：

    ```java
    ProxyRequests off
    ```

1.  我们应该添加一个带有均衡标识符和 `BalanceMembers` 的代理，而不是文档根。如果我们想使用 **轮询** 策略，我们还需要将其设置为 `lbmethod`（**负载均衡方法**）：

    ```java
    <Proxy balancer://app>
        BalancerMember http://127.0.0.1:9000
           BalancerMember http://127.0.0.2:9000
           BalancerMember http://127.0.0.3:9000
        ProxySet lbmethod=byrequests
    </Proxy>
    ```

1.  现在，我们需要添加代理的访问权限，它应该对所有人均可访问：

    ```java
                    Order Deny,Allow
                    Deny from none
                    Allow from all
    ```

1.  最后，我们需要将代理映射到我们希望在服务器上加载应用程序的路径。这可以通过一行完成：

    ```java
    ProxyPass / balancer://app/
    ```

需要添加到 Apache HTTP 配置文件的配置如下：

```java
<VirtualHost *:80>
        ProxyPreserveHost On
        ProxyRequests off
        <Proxy balancer://app>

                BalancerMember http://127.0.0.1:9000
                BalancerMember http://127.0.0.2:9000
                BalancerMember http://127.0.0.3:9000
                Order Deny,Allow
                Deny from none
                Allow from all

                ProxySet lbmethod=byrequests
        </Proxy>
        ProxyPass / balancer://app/
</VirtualHost>
```

要启用 SSL，我们需要在 `VirtualHost` 定义中添加以下代码：

```java
SSLEngine on
SSLCertificateFile /path/to/domain.com.crt
SSLCertificateKeyFile /path/to/domain.com.key
```

### 注意

此配置已在 2014 年 7 月 31 日于 Apache/2.4.10 上进行过测试。

有关 Apache HTTP 的 `mod_proxy` 模块的更多信息，请参阅 [`httpd.apache.org/docs/2.2/mod/mod_proxy.html`](http://httpd.apache.org/docs/2.2/mod/mod_proxy.html)。

## nginx 服务器

**nginx** 服务器是一个高性能的 HTTP 服务器和反向代理，同时也是 IMAP/POP3 代理服务器。我们可以配置 nginx 使用两个模块——`proxy` 和 `upstream` 作为负载均衡器。这两个模块是 nginx 核心的一部分，默认情况下可用。

nginx 配置文件 `nginx.conf` 通常位于 `/etc/nginx`。让我们逐步更新它，以使用 nginx 作为我们的应用程序的负载均衡器：

1.  首先，我们需要为我们的应用程序服务器集群定义一个 `upstream` 模块。其语法如下：

    ```java
    upstream <group> {
    <loadBalancingMethod>;
    server <server1>;
    server <server2>;
    }
    ```

    默认的负载均衡方法是轮询。因此，当我们希望使用它时，无需明确指定。现在，对于我们的应用程序，`upstream` 模块将如下所示：

    ```java
    upstream app {
        server 127.0.0.1:9000;
        server 127.0.0.2:9000;
        server 127.0.0.3:9000;
    }
    ```

1.  现在，我们所需做的就是代理所有请求。为此，我们必须更新 `server` 模块的 `location` 模块：

    ```java
    server {
        listen       80 default_server;
        server_name  localhost;
        …
        location / {
            proxy_pass http://app;        
            proxy_set_header Host $host;
        }
    }
    ```

nginx 服务器也支持代理 **WebSocket**。要启用 WebSocket 连接，我们需要向 `location` 模块添加两个头部。因此，如果我们的 Play 应用程序使用 WebSocket，我们可以将 `location` 模块定义为以下内容：

```java
location / {
    proxy_pass http://app;
    proxy_set_header Host $host;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

要启用 SSL，我们需要在服务器定义中添加以下设置：

```java
 ssl_certificate     /path/to/domain.com.crt;
 ssl_certificate_key path/to/domain.com.key;

```

### 注意

此配置已在 nginx/1.4.7 上进行过测试。

更多关于 nginx 负载均衡配置的详细信息，请参阅 [`nginx.org/en/docs/http/load_balancing.html#nginx_load_balancing_configuration`](http://nginx.org/en/docs/http/load_balancing.html#nginx_load_balancing_configuration)。

## lighttpd

`lighttpd` 服务器是一个轻量级的网络服务器，设计和优化用于高性能环境。所有可能需要的实用工具都作为模块提供，可以根据我们的需求进行包含。我们可以使用 `mod_proxy` 模块将 `lighttpd` 设置为 Play 应用的前端服务器。为此，我们需要进行一些配置更改。具体如下：

1.  更新 `lighttpd.conf` 文件（通常位于 `/etc/lighttpd/`），以加载额外的模块。

1.  默认情况下，加载模块是禁用的。可以通过取消注释此行来启用：

    ```java
    include "modules.conf"
    ```

1.  更新 `modules.conf`（位于 `lighttpd.conf` 相同的目录中），以加载 `mod_proxy` 模块。

1.  默认情况下，仅启用 `mod_access`。将 `server.modules` 更新为以下代码：

    ```java
    server.modules = (
        "mod_access",
        "mod_proxy"
    )
    ```

1.  现在，通过取消注释此行来启用加载 `mod_proxy` 的设置：

    ```java
    include "conf.d/proxy.conf"
    ```

1.  更新 `proxy.conf` 文件（通常位于 `/etc/lighttpd/conf.d/`），包含服务器代理配置。`q` 模块只有三个设置：

    +   `proxy.debug`：此设置启用/禁用日志级别

    +   `proxy.balance`：此设置是负载均衡算法（轮询、哈希和公平）

    +   `proxy.server`：此设置是请求发送的地方

        定义 `proxy.server` 设置的预期格式如下：

        ```java
        ( <extension> =>
            ( [ <name> => ]
            ( "host" => <string> ,
                "port" => <integer> ),
            ( "host" => <string> ,
                "port" => <integer> )
          ),
          <extension> => ...
        )
        ```

    以下是对此代码中术语的解释：

    +   `<extension>`：此术语是文件扩展名或前缀（如果以 `"/"` 开头）；空引号 `""` 匹配所有请求

    +   `<name>`：此术语是 `mod_status` 生成的统计信息中显示的可选名称

    +   `host`：此术语用于指定代理服务器的 IP 地址

    +   `port`：此术语用于设置对应主机的 TCP 端口（默认值为 `80`）

1.  根据需要更新代理设置：

    ```java
    server.modules += ( "mod_proxy" )
    proxy.balance = "round-robin"
    proxy.server = ( "" =>
        ( "app" =>
            (
                "host" => "127.0.0.1",
                "port" => 9000
            ),
            (
                "host" => "127.0.0.2",
                "port" => 9000
            ),
            (
                "host" => "127.0.0.3",
                "port" => 9000
           )
        )
    )
    ```

    ### 注意

    此配置已在 2014 年 3 月 12 日在 lighttpd/1.4.35 上进行过测试。

    关于 `mod_proxy` 的配置设置更多信息，请参阅 [`redmine.lighttpd.net/projects/lighttpd/wiki/Docs_ModProxy`](http://redmine.lighttpd.net/projects/lighttpd/wiki/Docs_ModProxy)。

## 高可用性代理

**高可用性** **代理**（**HAProxy**）为基于 TCP 和 HTTP 的应用程序提供高可用性、负载均衡和代理。我们可以通过更新 `haproxy.cfg` 配置文件（通常位于 `/etc/haproxy/`）将 HAProxy 设置为负载均衡器。

让我们逐步进行所需的配置更改：

1.  首先，我们需要定义后端集群。定义后端的语法如下：

    ```java
    backend <name>
    balance    <load balance method>
    server <sname> <ip>:<port>
    server    <sname> <ip>:<port>
    ```

1.  因此，我们应用程序的后端将如下所示：

    ```java
    backend app
        balance     roundrobin
        server  app1 127.0.0.1:9000
        server  app1 127.0.0.2:9000
        server  app1 127.0.0.3:9000
    ```

1.  现在，我们只需将请求指向后端集群。我们可以通过更新前端部分来完成此操作：

    ```java
    frontend  main *:80
    default_backend             app
    ```

对于使用 WebSockets 的应用程序，不需要额外的配置。

### 注意

此配置已在 HAProxy 版本 1.5.9（2014/11/25）上进行了测试。

# 故障排除

这些是一些你可能遇到的一些边缘情况：

+   我们需要将我们的应用程序部署到 Tomcat。我们如何将应用程序打包为 WAR？

    虽然 Play 默认不支持此功能，但我们可以使用`play2-war-plugin`模块（参考 [`github.com/play2war/play2-war-plugin/`](https://github.com/play2war/play2-war-plugin/)）来实现这一点。

+   有没有更简单的方法来部署应用程序到 PaaS？

    在 Heroku、Clever Cloud、Cloud Foundry 和/或 AppFog 上部署 Play 应用程序的文档可在 [`www.playframework.com/documentation/2.3.x/DeployingCloud`](https://www.playframework.com/documentation/2.3.x/DeployingCloud) 查阅。

# 摘要

在本章中，我们看到了如何在生产环境中部署 Play 应用程序。在部署过程中，我们看到了默认可用的不同打包选项（例如 `rpm`、`deb`、`zip`、`windows` 等）。我们还看到了不同的配置设置，例如 HTTP 端口、请求头最大大小等，这些我们可以在生产环境中启动应用程序时指定。我们还讨论了如何使用反向代理向应用程序发送请求。

在下一章中，我们将讨论 Play 插件的工作原理，以及如何构建定制的 Play 插件以满足不同的需求。
