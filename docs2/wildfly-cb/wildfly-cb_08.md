# 第八章 命令行界面（CLI）的使用

在本章中，您将学习以下食谱：

+   调用 CLI 命令

+   检查 WildFly 版本

+   检查 WildFly 操作模式

+   获取操作系统版本

+   获取 JVM 版本

+   检查 JVM 选项

+   检查 JVM 内存 - 堆大小等

+   检查服务器状态

+   检查 JNDI 树视图

+   调用外部文件中声明的 CLI 命令

# 简介

在本章中，您将学习如何使用 CLI 获取您所需的信息。CLI 还提供了一个方法来执行它之外的操作，通过指定要连接的 WildFly 和要执行的操作。此外，WildFly 还提供了一个 HTTP API，可以用来执行操作并检索信息。这个 API 的大部分用于系统监控。

由于 WildFly 可以以独立模式或域模式运行，每当有需要时，我们将使用这两种模式，因为您可能以不同的方式访问相同的信息。

在本章中，我们将模拟/使用远程 WildFly 实例，就像实际场景中您可以将以下食谱应用于连接到远程服务器一样。在没有看到任何身份验证和授权问题的情况下尝试`localhost`上的食谱是没有意义的。

要模拟远程服务器，您最终可以使用 VirtualBox 或 Docker，然后按照第一章中描述的方式安装 WildFly。

我将使用 Docker 工具版本 1.5 在 Linux 容器中运行的 WildFly。显然，您可以使用真实的远程服务器，效果是一样的；重要的是 WildFly 平台是否暴露出来。

顺便说一句，这本书的最后一章是关于使用 Docker 在 Linux 容器中运行 WildFly 的，所以如果您对 Docker 一无所知（您一直躲在哪里？），请查看这本书的最后一章，或者获取一本绝对优秀的 Docker 书籍，《Docker Book》，作者*James Turnbull*，[`www.dockerbook.com`](http://www.dockerbook.com)

所以，我的 Docker 配置如下：

```java
DOCKER_HOST=tcp://192.168.59.103:2376
```

因此，我的远程 WildFly 实例将绑定到该 IP，以及 WildFly 的常用端口，如`8080`、`9990`和`9999`。相同的管理用户是本书中使用的：用户名为`wildfly`，密码为`cookbook.2015`。

# 调用 CLI 命令

在这个食谱中，您将学习如何直接从您的命令行调用 CLI 命令，而不需要访问 CLI 本身，比如说不是以交互式的方式。这种技术在您需要脚本一些过程时可能很有用，比如按顺序停止和启动服务器，只有在另一个应用程序已经部署的情况下才部署应用程序，等等。很多时候，您还需要监控某些状态，因此您只需要那个数字（通常在 Nagios 中看到的那样）。

## 准备工作

记住，我正在远程运行 WildFly，绑定到 IP 地址`192.168.59.103`。WildFly 已经启动并运行。

## 如何做到这一点…

在您的本地 PC 上，打开一个新的终端窗口并运行以下命令：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command=":read-attribute(name=release-codename)"
{
    "outcome" => "success",
    "result" => "Kenny"
}
```

显然，你可以通过在输出中使用`awk`命令提取你需要的信息（你可以使用你熟悉的任何工具），如下所示：

```java
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command=":read-attribute(name=release-codename)" | awk 'NR==3 { print $3 }'
"Kenny"
```

如果你在域模式下运行 WildFly，调用和结果都是相同的。

## 它是如何工作的...

实际上，没有太多可说的或解释的，只是`jboss-cli.sh`脚本调用了`wildfly-cli-1.0.0.Beta2.jar`库中的`org.jboss.as.cli.CommandLineMain`类，传递所有参数。`--command`基本上禁用了交互模式，执行语句，并在返回标准输入之前在标准输出中打印输出。

我们所做的基本上是以下操作：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh
You are disconnected at the moment. Type 'connect' to connect to the server or 'help' for the list of supported commands.
[disconnected /] connect 192.168.59.103:9990
Authenticating against security realm: ManagementRealm
Username: wildfly
Password: [cookbook.2015]
[standalone@192.168.59.103:9990 /] :read-attribute(name=release-codename)
{
    "outcome" => "success",
    "result" => "Kenny"
}
[standalone@192.168.59.103:9990 /]
```

顺便说一下，你无法操作输出。

## 还有更多...

WildFly 提供了一个额外的 API，即 HTTP API。让我们用一些网络命令行工具，如`curl`来尝试一下。

### curl

在一个运行的 WildFly 实例上，执行以下步骤：

1.  在你的本地 PC 上，打开一个新的终端窗口并执行以下命令：

    ```java
    $ curl --verbose http://192.168.59.103:9990/management/?operation=attribute\&name=release—codename
    * Hostname was NOT found in DNS cache
    *   Trying 192.168.59.103...
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    > GET /management/?operation=attribute&name=release-codename HTTP/1.1
    > User-Agent: curl/7.37.0
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 401 Unauthorized
    < Connection: keep-alive
    < WWW-Authenticate: Digest realm="ManagementRealm",domain="/management",nonce="it/3pAte8WkNMTQzMDkzNzkzNDYwN+FFS4e5sd6vPqf6T/M4bQI=",opaque="00000000000000000000000000000000",algorithm=MD5
    < Content-Length: 77
    < Content-Type: text/html
    < Date: Wed, 06 May 2015 18:45:34 GMT
    <
    * Connection #0 to host 192.168.59.103 left intact
    <html><head><title>Error</title></head><body>401 - Unauthorized</body></html>
    ```

    如你所见，`curl`对`ManagementRealm`的`Digest`认证提出了抱怨，这是 WildFly 管理接口所使用的。

1.  让我们把用户名`wildfly`和密码`cookbook.2015`提供给命令，如下所示：

    ```java
    $ curl --verbose --digest --user wildfly http://192.168.59.103:9990/management/?operation=attribute\&name=release-codename
    Enter host password for user 'wildfly':
    * Hostname was NOT found in DNS cache
    *   Trying 192.168.59.103...
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    * Server auth using Digest with user 'wildfly'
    > GET /management/?operation=attribute&name=release-codename HTTP/1.1
    > User-Agent: curl/7.37.0
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 401 Unauthorized
    < Connection: keep-alive
    < WWW-Authenticate: Digest realm="ManagementRealm",domain="/management",nonce="6ofpOO62oNsNMTQzMDkzNzk2ODQzORxjucrlmU+bXpQSbl6Mkos=",opaque="00000000000000000000000000000000",algorithm=MD5
    < Content-Length: 77
    < Content-Type: text/html
    < Date: Wed, 06 May 2015 18:46:08 GMT
    <
    * Ignoring the response-body
    * Connection #0 to host 192.168.59.103 left intact
    * Issue another request to this URL: 'http://192.168.59.103:9990/management/?operation=attribute&name=release-codename'
    * Found bundle for host 192.168.59.103: 0x228bb70
    * Re-using existing connection! (#0) with host 192.168.59.103
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    * Initializing NSS with certpath: sql:/etc/pki/nssdb
    * Server auth using Digest with user 'wildfly'
    > GET /management/?operation=attribute&name=release-codename HTTP/1.1
    > Authorization: Digest username="wildfly", realm="ManagementRealm", nonce="6ofpOO62oNsNMTQzMDkzNzk2ODQzORxjucrlmU+bXpQSbl6Mkos=", uri="/management/?operation=attribute&name=release-codename", response="e52ce8f429808ef48a76da7193de27e9", opaque="00000000000000000000000000000000", algorithm="MD5"
    > User-Agent: curl/7.37.0
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 200 OK
    < Connection: keep-alive
    < Authentication-Info: nextnonce="tsPhgznQArENMTQzMDkzNzk2ODU0NSgg91cYeN3rLKUkBXYRMA8="
    < Content-Type: application/json; charset=utf-8
    < Content-Length: 7
    < Date: Wed, 06 May 2015 18:46:08 GMT
    <
    * Connection #0 to host 192.168.59.103 left intact
    "Kenny"
    ```

1.  好的，它工作了；现在移除`--verbose`标志并再次执行命令。你应该在输入密码后只得到`Kenny`。

1.  如果你不想输入密码，你也可以将其作为参数传递，但这伴随着所有的安全担忧。为此，请按照以下操作进行：

    ```java
    $ curl --digest --user wildfly:cookbook.2015 http://192.168.59.103:9990/management/?operation=attribute\&name=release-codename
    "Kenny"
    ```

# 检查 WildFly 版本

在这个菜谱中，你将学习如何通过向 CLI 调用命令来检查你正在运行的 WildFly 版本。

## 准备工作

记住，我正在远程运行 WildFly，绑定到`192.168.59.103`作为 IP。WildFly 已经启动并运行。

## 如何操作...

在你的本地 PC 上，打开一个新的终端窗口并执行以下命令：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command=":read-attribute(name=product-version)"
{
    "outcome" => "success",
    "result" => "9.0.0.Beta2"
}
```

显然，你可以通过在输出中使用`awk`命令提取你需要的信息（你可以使用你熟悉的任何工具），如下所示：

```java
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command=":read-attribute(name=product-version)" | awk 'NR==3 { print $3 }'
"9.0.0.Beta2"
```

如果你在域模式下运行 WildFly，调用和结果将是相同的。

## 它是如何工作的...

这里我们告诉`jboss-cli.sh`脚本执行我们在`--command`参数中定义的命令，并将结果返回到标准输出。

使用 CLI，它将是这样的：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh
You are disconnected at the moment. Type 'connect' to connect to the server or 'help' for the list of supported commands.
[disconnected /] connect 192.168.59.103
Authenticating against security realm: ManagementRealm
Username: wildfly
Password:
[standalone@192.168.59.103:9990 /] :read-attribute(name=product-version)
{
    "outcome" => "success",
    "result" => "9.0.0.Beta2"
}
[standalone@192.168.59.103:9990 /]
```

顺便说一下，你无法操作输出。

## 还有更多...

WildFly 提供了一个额外的 API，即 HTTP API。让我们用一些网络命令行工具，如`curl`来尝试一下。

### curl

在一个运行的 WildFly 实例上，执行以下步骤：

1.  打开一个新的终端窗口并运行以下命令：

    ```java
    $ curl --verbose http://192.168.59.103:9990/management/?operation=attribute\&name=product-version
    * Hostname was NOT found in DNS cache
    *   Trying 192.168.59.103...
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    > GET /management/?operation=attribute&name=product-version HTTP/1.1
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 401 Unauthorized
    < Connection: keep-alive
    < WWW-Authenticate: Digest realm="ManagementRealm",domain="/management",nonce="Z9fm45feS/QNMTQzMTk1ODE4OTA5OXjLOp5ZQcJ+ag+dmE6jbEM=",opaque="00000000000000000000000000000000",algorithm=MD5
    < Content-Length: 77
    < Content-Type: text/html
    < Date: Mon, 18 May 2015 14:09:49 GMT
    <
    * Connection #0 to host 192.168.59.103 left intact
    <html><head><title>Error</title></head><body>401 - Unauthorized</body></html>
    ```

    如你所见，`curl`对`ManagementRealm`的摘要认证提出了抱怨，这是 WildFly 管理接口所使用的。

1.  让我们把用户名设置为`wildfly`，密码设置为`cookbook.2015`，如下所示：

    ```java
    $ curl --verbose --digest --user wildfly http://192.168.59.103:9990/management/?operation=attribute\&name=product-version
    Enter host password for user 'wildfly':
    * Hostname was NOT found in DNS cache
    *   Trying 192.168.59.103...
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    * Server auth using Digest with user 'wildfly'
    > GET /management/?operation=attribute&name=product-version HTTP/1.1
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 401 Unauthorized
    < Connection: keep-alive
    < WWW-Authenticate: Digest realm="ManagementRealm",domain="/management",nonce="iRU9d5SaQI8NMTQzMTk1ODc1NTY2OAgdi5zSNYp+IAgtpOgBZRU=",opaque="00000000000000000000000000000000",algorithm=MD5
    < Content-Length: 77
    < Content-Type: text/html
    < Date: Mon, 18 May 2015 14:19:15 GMT
    <
    * Ignoring the response-body
    * Connection #0 to host 192.168.59.103 left intact
    * Issue another request to this URL: 'http://192.168.59.103:9990/management/?operation=attribute&name=product-version'
    * Found bundle for host 192.168.59.103: 0x7fc3a0400ed0
    * Re-using existing connection! (#0) with host 192.168.59.103
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    * Server auth using Digest with user 'wildfly'
    > GET /management/?operation=attribute&name=product-version HTTP/1.1
    > Authorization: Digest username="wildfly", realm="ManagementRealm", nonce="iRU9d5SaQI8NMTQzMTk1ODc1NTY2OAgdi5zSNYp+IAgtpOgBZRU=", uri="/management/?operation=attribute&name=product-version", response="e3f7a23441f44992d1d7e2c9fcc00cc2", opaque="00000000000000000000000000000000", algorithm="MD5"
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 200 OK
    < Connection: keep-alive
    < Authentication-Info: nextnonce="45+yNlhzGZ8NMTQzMTk1ODc1NTY4MZP/ti6JYQlWeum3jKxWEao="
    < Content-Type: application/json; charset=utf-8
    < Content-Length: 13
    < Date: Mon, 18 May 2015 14:19:15 GMT
    <
    * Connection #0 to host 192.168.59.103 left intact
    "9.0.0.Beta2"
    ```

    好的，它工作了；现在移除`--verbose`标志并再次执行命令。输入密码后，你应该得到`9.0.0.Beta2`。

1.  如果你不想输入密码，你也可以将其作为参数传递，但请注意这会带来所有的安全顾虑，如下所示：

    ```java
    $ curl  --digest --user wildfly:cookbook.2015 http://192.168.59.103:9990/management/?operation=attribute\&name=product-version
    "9.0.0.Beta2"
    ```

# 检查 WildFly 操作模式

在这个菜谱中，你将学习如何通过向 CLI 发送命令来检查 WildFly 正在运行的操作模式。

## 准备工作

记住，我正在远程运行 WildFly，绑定到 IP 地址`192.168.59.103`。WildFly 已经启动并运行。

## 如何操作...

打开一个新的终端窗口并执行以下命令：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command=":read-attribute(name=launch-type)"
{
    "outcome" => "success",
    "result" => "STANDALONE"
}
```

显然，你可以使用`awk`命令从输出中提取所需的信息（你可以使用你熟悉的任何工具），如下所示：

```java
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command=":read-attribute(name=launch-type)" | awk 'NR==3 { print $3 }'
"STANDALONE"
```

如果你在域模式下运行 WildFly，调用方式相同，但结果如下：

```java
"DOMAIN"
```

就这样！

## 它是如何工作的...

在这里，我们告诉`jboss-cli.sh`脚本执行我们在`--command`参数中定义的命令，并将结果输出到标准输出。

使用 CLI，如下所示：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh
You are disconnected at the moment. Type 'connect' to connect to the server or 'help' for the list of supported commands.
[disconnected /] connect 192.168.59.103:9990
Authenticating against security realm: ManagementRealm
Username: wildfly
Password:
[standalone@192.168.59.103:9990 /] :read-attribute(name=launch-type)
{
    "outcome" => "success",
    "result" => "STANDALONE"
}
[standalone@192.168.59.103:9990 /]
```

顺便说一下，你不能操纵输出。

## 还有更多...

WildFly 提供了一个额外的 API，即 HTTP API。让我们用一些网络命令行工具，如`curl`来尝试它。

### curl

在一个运行的 WildFly 实例中，执行以下步骤：

1.  打开一个新的终端窗口并执行以下命令：

    ```java
    $ curl --verbose http://192.168.59.103:9990/management/?operation=attribute\&name=launch-type
    * Hostname was NOT found in DNS cache
    *   Trying 192.168.59.103...
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    > GET /management/?operation=attribute&name=launch-type HTTP/1.1
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 401 Unauthorized
    < Connection: keep-alive
    < WWW-Authenticate: Digest realm="ManagementRealm",domain="/management",nonce="oOubmxOIcS0NMTQzMTk1OTg2Mjk4Nqu/YSX7Gh368EZ1XPoG3Eg=",opaque="00000000000000000000000000000000",algorithm=MD5
    < Content-Length: 77
    < Content-Type: text/html
    < Date: Mon, 18 May 2015 14:37:42 GMT
    <
    * Connection #0 to host 192.168.59.103 left intact
    <html><head><title>Error</title></head><body>401 - Unauthorized</body></html>
    ```

    如你所见，`curl`对`ManagementRealm`的摘要认证提出了抱怨，这是 WildFly 管理接口所使用的。

1.  让我们把用户名和密码传递给命令，如下所示：

    ```java
    $ curl --verbose --digest --user wildfly http://192.168.59.103:9990/management/?operation=attribute\&name=launch-type
    Enter host password for user 'wildfly':
    * Hostname was NOT found in DNS cache
    *   Trying 192.168.59.103...
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    * Server auth using Digest with user 'wildfly'
    > GET /management/?operation=attribute&name=launch-type HTTP/1.1
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 401 Unauthorized
    < Connection: keep-alive
    < WWW-Authenticate: Digest realm="ManagementRealm",domain="/management",nonce="ma7nUTWjpCINMTQzMTk2MDA1MjIyOPhpd9+Np6mKrGK9OIhVD1A=",opaque="00000000000000000000000000000000",algorithm=MD5
    < Content-Length: 77
    < Content-Type: text/html
    < Date: Mon, 18 May 2015 14:40:52 GMT
    <
    * Ignoring the response-body
    * Connection #0 to host 192.168.59.103 left intact
    * Issue another request to this URL: 'http://192.168.59.103:9990/management/?operation=attribute&name=launch-type'
    * Found bundle for host 192.168.59.103: 0x7fb833d00c70
    * Re-using existing connection! (#0) with host 192.168.59.103
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    * Server auth using Digest with user 'wildfly'
    > GET /management/?operation=attribute&name=launch-type HTTP/1.1
    > Authorization: Digest username="wildfly", realm="ManagementRealm", nonce="ma7nUTWjpCINMTQzMTk2MDA1MjIyOPhpd9+Np6mKrGK9OIhVD1A=", uri="/management/?operation=attribute&name=launch-type", response="3ae09c2aaf9a1c29a0f2c5153d56d485", opaque="00000000000000000000000000000000", algorithm="MD5"
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 200 OK
    < Connection: keep-alive
    < Authentication-Info: nextnonce="xWcBNZToWOoNMTQzMTk2MDA1MjIzNKGJWH9tRyHEBw/DEd3VE6w="
    < Content-Type: application/json; charset=utf-8
    < Content-Length: 12
    < Date: Mon, 18 May 2015 14:40:52 GMT
    <
    * Connection #0 to host 192.168.59.103 left intact
    "STANDALONE"
    ```

1.  好的，它工作了；现在移除`--verbose`标志并再次执行命令。输入密码后，你应该得到`STANDALONE`。

1.  如果你不想输入密码，你也可以将其作为参数传递，但请注意这会带来所有的安全顾虑，如下所示：

    ```java
    $ curl --verbose --digest --user wildfly:cookbook.2015 http://192.168.59.103:9990/management/?operation=attribute\&name=launch-type
    "STANDALONE"
    ```

# 获取操作系统版本

在这个菜谱中，你将学习如何通过向 CLI 发送命令来获取 WildFly 正在运行的操作系统版本。

## 准备工作

记住，我正在远程运行 WildFly，绑定到 IP 地址`192.168.59.103`。WildFly 已经启动并运行。

## 如何操作...

1.  打开一个新的终端窗口并运行以下命令：

    ```java
    $ cd $WILDFLY_HOME
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/core-service=platform-mbean/type=operating-system:read-resource(include-runtime=true,include-defaults=true)"
    {
        "outcome" => "success",
        "result" => {
            "name" => "Linux",
            "arch" => "amd64",
            "version" => "3.18.5-tinycore64",
            "available-processors" => 8,
            "system-load-average" => 0.0,
            "object-name" => "java.lang:type=OperatingSystem"
        }
    }
    ```

1.  显然，你可以使用输出中的`awk`命令提取所需的信息（你可以使用你熟悉的任何工具），如下所示：

    ```java
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/core-service=platform-mbean/type=operating-system:read-resource(include-runtime=true,include-defaults=true)" | awk 'NR==4 { print $3 }'
    "Linux",
    ```

1.  如果你在域模式下运行 WildFly，调用方式如下：

    ```java
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/host=master/server=server-one/core-service=platform-mbean/type=operating-system:read-resource(include-runtime=true,include-defaults=true)"
    {
        "outcome" => "success",
        "result" => {
            "name" => "Linux",
            "arch" => "amd64",
            "version" => "3.18.5-tinycore64",
            "available-processors" => 8,
            "system-load-average" => 0.0,
            "object-name" => "java.lang:type=OperatingSystem"
        }
    }
    ```

就这样！

## 它是如何工作的...

在这里，我们告诉`jboss-cli.sh`脚本执行我们在`--command`参数中定义的命令，并将结果输出到标准输出。

使用 CLI，如下所示：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh
You are disconnected at the moment. Type 'connect' to connect to the server or 'help' for the list of supported commands.
[disconnected /] connect 192.168.59.103:9990
Authenticating against security realm: ManagementRealm
Username: wildfly
Password:
[standalone@192.168.59.103:9990 /] /core-service=platform-mbean/type=operating-system:read-resource(include-runtime=true,include-defaults=true)
{
    "outcome" => "success",
    "result" => {
        "name" => "Linux",
        "arch" => "amd64",
        "version" => "3.16.4-tinycore64",
        "available-processors" => 8,
        "system-load-average" => 0.0,
        "object-name" => "java.lang:type=OperatingSystem"
    }
}
[standalone@192.168.59.103:9990 /]
```

顺便说一下，你不能操纵输出。

## 还有更多...

WildFly 提供了一个额外的 API，即 HTTP API。让我们用一些网络命令行工具，如`curl`来尝试它。

### curl

在一个运行的 WildFly 实例中，执行以下步骤：

1.  打开一个新的终端窗口并按照以下步骤操作：

    ```java
    $ curl --verbose http://192.168.59.103:9990/management/core-service/platform-mbean/type/operating-system?operation=resource\&include-runtime=true\&include-defaults=true
    * Hostname was NOT found in DNS cache
    *   Trying 192.168.59.103...
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    > GET /management/core-service/platform-mbean/type/operating-system?operation=resource&include-runtime=true&include-defaults=true HTTP/1.1
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 401 Unauthorized
    < Connection: keep-alive
    < WWW-Authenticate: Digest realm="ManagementRealm",domain="/management",nonce="vgtjzIheBzkNMTQzMTk2MDkwMzkyMPnsADViafwy586SMaSftDU=",opaque="00000000000000000000000000000000",algorithm=MD5
    < Content-Length: 77
    < Content-Type: text/html
    < Date: Mon, 18 May 2015 14:55:03 GMT
    <
    * Connection #0 to host 192.168.59.103 left intact
    <html><head><title>Error</title></head><body>401 - Unauthorized</body></html>
    ```

    如你所见，`curl` 对 `ManagementRealm` 的摘要认证提出了抱怨，这是 WildFly 管理接口所使用的。

1.  让我们把用户名和密码提供给命令，如下所示：

    ```java
    $ curl --verbose --digest --user wildfly http://192.168.59.103:9990/management/core-service/platform-mbean/type/operating-system?operation=resource\&include-runtime=true\&include-defaults=true
    Enter host password for user 'wildfly':
    * Hostname was NOT found in DNS cache
    *   Trying 192.168.59.103...
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    * Server auth using Digest with user 'wildfly'
    > GET /management/core-service/platform-mbean/type/operating-system?operation=resource&include-runtime=true&include-defaults=true HTTP/1.1
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 401 Unauthorized
    < Connection: keep-alive
    < WWW-Authenticate: Digest realm="ManagementRealm",domain="/management",nonce="Z7R7/j+3b5oNMTQzMTk2MTA3NjQ1N4lD5fc075kCb3a0afe4pCg=",opaque="00000000000000000000000000000000",algorithm=MD5
    < Content-Length: 77
    < Content-Type: text/html
    < Date: Mon, 18 May 2015 14:57:56 GMT
    <
    * Ignoring the response-body
    * Connection #0 to host 192.168.59.103 left intact
    * Issue another request to this URL: 'http://192.168.59.103:9990/management/core-service/platform-mbean/type/operating-system?operation=resource&include-runtime=true&include-defaults=true'
    * Found bundle for host 192.168.59.103: 0x7ff722414af0
    * Re-using existing connection! (#0) with host 192.168.59.103
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    * Server auth using Digest with user 'wildfly'
    > GET /management/core-service/platform-mbean/type/operating-system?operation=resource&include-runtime=true&include-defaults=true HTTP/1.1
    > Authorization: Digest username="wildfly", realm="ManagementRealm", nonce="Z7R7/j+3b5oNMTQzMTk2MTA3NjQ1N4lD5fc075kCb3a0afe4pCg=", uri="/management/core-service/platform-mbean/type/operating-system?operation=resource&include-runtime=true&include-defaults=true", response="d71b0f677bfb476226880ae0f55b899c", opaque="00000000000000000000000000000000", algorithm="MD5"
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 200 OK
    < Connection: keep-alive
    < Authentication-Info: nextnonce="xEfjYw89nD8NMTQzMTk2MTA3NjQ3MSkjdHABRCjFARHS2aR438k="
    < Content-Type: application/json; charset=utf-8
    < Content-Length: 176
    < Date: Mon, 18 May 2015 14:57:56 GMT
    <
    * Connection #0 to host 192.168.59.103 left intact
    {"name" : "Linux", "arch" : "amd64", "version" : "3.18.5-tinycore64", "available-processors" : 8, "system-load-average" : 0.0, "object-name" : "java.lang:type=OperatingSystem"}
    ```

1.  好的，它工作了；现在移除 `--verbose` 标志并再次执行命令。你应该在输入密码后只得到以下 JSON 输出：

    ```java
    {"name" : "Linux", "arch" : "amd64", "version" : "3.16.4-tinycore64", "available-processors" : 8, "system-load-average" : 0.0, "object-name" : "java.lang:type=OperatingSystem"}
    ```

    很整洁，但看起来相当丑陋！

1.  如果你想要美化 JSON 输出并且不想输入密码（你也可以将其作为参数传递，尽管这会带来所有的安全担忧），你可以这样做：

    ```java
    $ curl --digest --user wildfly:cookbook.2015 http://192.168.59.103:9990/management/core-service/platform-mbean/type/operating-system?operation=resource\&include-runtime=true\&include-defaults=true\&json.pretty=true
    {
        "name" : "Linux",
        "arch" : "amd64",
        "version" : "3.18.5-tinycore64",
        "available-processors" : 8,
        "system-load-average" : 0.0,
        "object-name" : "java.lang:type=OperatingSystem"
    }
    ```

这就是我喜欢的样子！

# 获取 JVM 版本

在这个菜谱中，你将学习如何通过向 CLI 发送命令来获取 WildFly 正在运行的 JVM。

## 准备工作

记住我正在远程运行 WildFly，绑定到 IP `192.168.59.103`。WildFly 已经启动并运行。

## 如何做到这一点……

1.  打开一个新的终端窗口并执行以下命令：

    ```java
    $ cd $WILDFLY_HOME
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/core-service=platform-mbean/type=runtime:read-attribute(name=spec-version)"
    {
        "outcome" => "success",
        "result" => "1.8"
    }
    ```

1.  显然，你可以通过使用输出中的 `awk` 命令（你可以使用你熟悉的任何工具）来提取你需要的信息，如下所示：

    ```java
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/core-service=platform-mbean/type=runtime:read-attribute(name=spec-version)" | awk 'NR==3 { print $3 }'
    "1.8"
    ```

1.  如果你是在域模式下运行 WildFly，调用方式如下：

    ```java
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/host=master/server=server-one/core-service=platform-mbean/type=runtime:read-attribute(name=spec-version)" | awk 'NR==3 { print $3 }'
    "1.8"
    ```

就这样！

## 它是如何工作的……

我们在这里告诉 `jboss-cli.sh` 脚本执行我们在 `--command` 参数中定义的命令，并将结果返回到标准输出。

使用 CLI，它将是这样的：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh
You are disconnected at the moment. Type 'connect' to connect to the server or 'help' for the list of supported commands.
[disconnected /] connect 192.168.59.103:9990
Authenticating against security realm: ManagementRealm
Username: wildfly
Password:
[standalone@192.168.59.103:9990 /] /core-service=platform-mbean/type=runtime:read-attribute(name=spec-version)
{
    "outcome" => "success",
    "result" => "1.8"
}
[standalone@192.168.59.103:9990 /]
```

顺便说一下，你无法操作输出。

## 还有更多……

WildFly 提供了一个额外的 API，即 HTTP API。让我们用一些网络命令行工具，如 `curl` 来试试。

### curl

在一个运行的 WildFly 实例中，执行以下步骤：

1.  打开一个新的终端窗口并运行以下命令：

    ```java
    $ curl --verbose http://192.168.59.103:9990/management/core-service/platform-mbean/type/runtime?operation=attribute\&name=spec-version
    * Hostname was NOT found in DNS cache
    *   Trying 192.168.59.103...
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    > GET /management/core-service/platform-mbean/type/runtime?operation=attribute&name=spec-version HTTP/1.1
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 401 Unauthorized
    < Connection: keep-alive
    < WWW-Authenticate: Digest realm="ManagementRealm",domain="/management",nonce="wvo9g8raGFsNMTQzMTk2MTgwNjk1NE1gqeXx9Q8BkyC5OMS5Dy8=",opaque="00000000000000000000000000000000",algorithm=MD5
    < Content-Length: 77
    < Content-Type: text/html
    < Date: Mon, 18 May 2015 15:10:06 GMT
    <
    * Connection #0 to host 192.168.59.103 left intact
    <html><head><title>Error</title></head><body>401 - Unauthorized</body></html>
    ```

    如你所见，`curl` 对 `ManagementRealm` 的摘要认证提出了抱怨，这是 WildFly 管理接口所使用的。

1.  让我们把用户名和密码提供给命令，如下所示：

    ```java
    $ curl --verbose --digest --user wildfly http://192.168.59.103:9990/management/core-service/platform-mbean/type/runtime?operation=attribute\&name=spec-version
    Enter host password for user 'wildfly':
    * Hostname was NOT found in DNS cache
    *   Trying 192.168.59.103...
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    * Server auth using Digest with user 'wildfly'
    > GET /management/core-service/platform-mbean/type/runtime?operation=attribute&name=spec-version HTTP/1.1
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 401 Unauthorized
    < Connection: keep-alive
    < WWW-Authenticate: Digest realm="ManagementRealm",domain="/management",nonce="lq4EWazA3IUNMTQzMTk2MTgzMjI3NM2iKbJNsltAipmEDWj2zBs=",opaque="00000000000000000000000000000000",algorithm=MD5
    < Content-Length: 77
    < Content-Type: text/html
    < Date: Mon, 18 May 2015 15:10:32 GMT
    <
    * Ignoring the response-body
    * Connection #0 to host 192.168.59.103 left intact
    * Issue another request to this URL: 'http://192.168.59.103:9990/management/core-service/platform-mbean/type/runtime?operation=attribute&name=spec-version'
    * Found bundle for host 192.168.59.103: 0x7fb35ac14a40
    * Re-using existing connection! (#0) with host 192.168.59.103
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    * Server auth using Digest with user 'wildfly'
    > GET /management/core-service/platform-mbean/type/runtime?operation=attribute&name=spec-version HTTP/1.1
    > Authorization: Digest username="wildfly", realm="ManagementRealm", nonce="lq4EWazA3IUNMTQzMTk2MTgzMjI3NM2iKbJNsltAipmEDWj2zBs=", uri="/management/core-service/platform-mbean/type/runtime?operation=attribute&name=spec-version", response="b91b7944f1c95154b5a0a97e2c87cace", opaque="00000000000000000000000000000000", algorithm="MD5"
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 200 OK
    < Connection: keep-alive
    < Authentication-Info: nextnonce="EP8IM2uluIcNMTQzMTk2MTgzMjI4NeTU7FBtoaXVQ/6HR5zlVzw="
    < Content-Type: application/json; charset=utf-8
    < Content-Length: 5
    < Date: Mon, 18 May 2015 15:10:32 GMT
    <
    * Connection #0 to host 192.168.59.103 left intact
    "1.8"
    ```

1.  好的，它工作了；现在移除 `--verbose` 标志并再次执行命令。你应该在输入密码后只得到以下输出：

    ```java
    "1.8"
    ```

1.  如果你不想输入密码，你也可以将其作为参数传递，尽管这会带来所有的安全担忧，如下所示：

    ```java
    $ curl --digest --user wildfly:cookbook.2015 http://192.168.59.103:9990/management/core-service/platform-mbean/type/runtime?operation=attribute\&name=spec-version
    "1.8"
    ```

这就是我喜欢的样子！

### 更多关于运行时类型的信息

我们在前面步骤中使用的 `runtime` 类型提供了很多非常有用的信息。以下是在漂亮的 JSON 格式中你可以拥有的所有信息的完整列表：

```java
{    "name" : "1145@e0d713e81636",
    "vm-name" : "Java HotSpot(TM) 64-Bit Server VM",
    "vm-vendor" : "Oracle Corporation",
    "vm-version" : "25.40-b25",
    "spec-name" : "Java Virtual Machine Specification",
    "spec-vendor" : "Oracle Corporation",
    "spec-version" : "1.8",
    "management-spec-version" : "1.2",
    "class-path" : "/home/wildfly/WFC/wildfly/jboss-modules.jar",
    "library-path" : "/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib",
    "boot-class-path-supported" : true,
    "boot-class-path" : "/home/wildfly/WFC/jdk8/jre/lib/resources.jar:/home/wildfly/WFC/jdk8/jre/lib/rt.jar:/home/wildfly/WFC/jdk8/jre/lib/sunrsasign.jar:/home/wildfly/WFC/jdk8/jre/lib/jsse.jar:/home/wildfly/WFC/jdk8/jre/lib/jce.jar:/home/wildfly/WFC/jdk8/jre/lib/charsets.jar:/home/wildfly/WFC/jdk8/jre/lib/jfr.jar:/home/wildfly/WFC/jdk8/jre/classes",
    "input-arguments" : [
        "-D[Standalone]",
        "-XX:+UseCompressedOops",
        "-XX:+UseCompressedOops",
        "-Xms64m",
        "-Xmx512m",
        "-XX:MaxPermSize=256m",
        "-Djava.net.preferIPv4Stack=true",
        "-Djboss.modules.system.pkgs=org.jboss.byteman",
        "-Djava.awt.headless=true",
        "-Dorg.jboss.boot.log.file=/home/wildfly/WFC/wildfly/standalone/log/server.log",
        "-Dlogging.configuration=file:/home/wildfly/WFC/wildfly/standalone/configuration/logging.properties"
    ],
    "start-time" : 1431960757217,
    "system-properties" : {
        "[Standalone]" : "",
        "awt.toolkit" : "sun.awt.X11.XToolkit",
        "file.encoding" : "ANSI_X3.4-1968",
        "file.encoding.pkg" : "sun.io",
        "file.separator" : "/",
        "java.awt.graphicsenv" : "sun.awt.X11GraphicsEnvironment",
        "java.awt.headless" : "true",
        "java.awt.printerjob" : "sun.print.PSPrinterJob",
        "java.class.path" : "/home/wildfly/WFC/wildfly/jboss-modules.jar",
        "java.class.version" : "52.0",
        "java.endorsed.dirs" : "/home/wildfly/WFC/jdk8/jre/lib/endorsed",
        "java.ext.dirs" : "/home/wildfly/WFC/jdk8/jre/lib/ext:/usr/java/packages/lib/ext",
        "java.home" : "/home/wildfly/WFC/jdk8/jre",
        "java.io.tmpdir" : "/tmp",
        "java.library.path" : "/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib",
        "java.naming.factory.url.pkgs" : "org.jboss.as.naming.interfaces:org.jboss.ejb.client.naming",
        "java.net.preferIPv4Stack" : "true",
        "java.runtime.name" : "Java(TM) SE Runtime Environment",
        "java.runtime.version" : "1.8.0_40-b26",
        "java.specification.name" : "Java Platform API 
        Specification",
        "java.specification.vendor" : "Oracle Corporation",
        "java.specification.version" : "1.8",
        "java.util.logging.manager" : "org.jboss.logmanager.LogManager",
        "java.vendor" : "Oracle Corporation",
        "java.vendor.url" : "http://java.oracle.com/",
        "java.vendor.url.bug" : "http://bugreport.sun.com/bugreport/",
        "java.version" : "1.8.0_40",
        "java.vm.info" : "mixed mode",
        "java.vm.name" : "Java HotSpot(TM) 64-Bit Server VM",
        "java.vm.specification.name" : "Java Virtual Machine Specification",
        "java.vm.specification.vendor" : "Oracle Corporation",
        "java.vm.specification.version" : "1.8",
        "java.vm.vendor" : "Oracle Corporation",
        "java.vm.version" : "25.40-b25",
        "javax.management.builder.initial" : "org.jboss.as.jmx.PluggableMBeanServerBuilder",
        "javax.xml.datatype.DatatypeFactory" : "__redirected.__DatatypeFactory",
        "javax.xml.parsers.DocumentBuilderFactory" : "__redirected.__DocumentBuilderFactory",
        "javax.xml.parsers.SAXParserFactory" : 
        "__redirected.__SAXParserFactory",
        "javax.xml.stream.XMLEventFactory" : 
        "__redirected.__XMLEventFactory",
        "javax.xml.stream.XMLInputFactory" : "__redirected.__XMLInputFactory",
        "javax.xml.stream.XMLOutputFactory" : "__redirected.__XMLOutputFactory",
        "javax.xml.transform.TransformerFactory" : "__redirected.__TransformerFactory",
        "javax.xml.validation.SchemaFactory:http://www.w3.org/2001/XMLSchema" : "__redirected.__SchemaFactory",
        "javax.xml.xpath.XPathFactory:http://java.sun.com/jaxp/xpath/dom" : "__redirected.__XPathFactory",
        "jboss.bind.address" : "0.0.0.0",
        "jboss.bind.address.management" : "0.0.0.0",
        "jboss.home.dir" : "/home/wildfly/WFC/wildfly",
        "jboss.host.name" : "e0d713e81636",
        "jboss.modules.dir" : "/home/wildfly/WFC/wildfly/modules",
        "jboss.modules.system.pkgs" : "org.jboss.byteman",
        "jboss.node.name" : "e0d713e81636",
        "jboss.qualified.host.name" : "e0d713e81636",
        "jboss.server.base.dir" : "/home/wildfly/WFC/wildfly/standalone",
        "jboss.server.config.dir" : "/home/wildfly/WFC/wildfly/standalone/configuration",
        "jboss.server.data.dir" : "/home/wildfly/WFC/wildfly/standalone/data",
        "jboss.server.deploy.dir" : "/home/wildfly/WFC/wildfly/standalone/data/content",
        "jboss.server.log.dir" : "/home/wildfly/WFC/wildfly/standalone/log",
        "jboss.server.name" : "e0d713e81636",
        "jboss.server.persist.config" : "true",
        "jboss.server.temp.dir" : "/home/wildfly/WFC/wildfly/standalone/tmp",
        "line.separator" : "\n",
        "logging.configuration" : "file:/home/wildfly/WFC/wildfly/standalone/configuration/logging.properties",
        "module.path" : "/home/wildfly/WFC/wildfly/modules",
        "org.apache.xml.security.ignoreLineBreaks" : "true",
        "org.jboss.boot.log.file" : "/home/wildfly/WFC/wildfly/standalone/log/server.log",
        "org.jboss.resolver.warning" : "true",
        "org.jboss.security.context.ThreadLocal" : "true",
        "org.xml.sax.driver" : "__redirected.__XMLReaderFactory",
        "os.arch" : "amd64",
        "os.name" : "Linux",
        "os.version" : "3.18.5-tinycore64",
        "path.separator" : ":",
        "sun.arch.data.model" : "64",
        "sun.boot.class.path" : "/home/wildfly/WFC/jdk8/jre/lib/resources.jar:/home/wildfly/WFC/jdk8/jre/lib/rt.jar:/home/wildfly/WFC/jdk8/jre/lib/sunrsasign.jar:/home/wildfly/WFC/jdk8/jre/lib/jsse.jar:/home/wildfly/WFC/jdk8/jre/lib/jce.jar:/home/wildfly/WFC/jdk8/jre/lib/charsets.jar:/home/wildfly/WFC/jdk8/jre/lib/jfr.jar:/home/wildfly/WFC/jdk8/jre/classes",
        "sun.boot.library.path" : "/home/wildfly/WFC/jdk8/jre/lib/amd64",
        "sun.cpu.endian" : "little",
        "sun.cpu.isalist" : "",
        "sun.io.unicode.encoding" : "UnicodeLittle",
        "sun.java.command" : "/home/wildfly/WFC/wildfly/jboss-modules.jar -mp /home/wildfly/WFC/wildfly/modules org.jboss.as.standalone -Djboss.home.dir=/home/wildfly/WFC/wildfly -Djboss.server.base.dir=/home/wildfly/WFC/wildfly/standalone -b 0.0.0.0 -bmanagement 0.0.0.0",
        "sun.java.launcher" : "SUN_STANDARD",
        "sun.jnu.encoding" : "ANSI_X3.4-1968",
        "sun.management.compiler" : "HotSpot 64-Bit Tiered Compilers",
        "sun.nio.ch.bugLevel" : "",
        "sun.os.patch.level" : "unknown",
        "user.country" : "US",
        "user.dir" : "/home/wildfly/WFC/wildfly",
        "user.home" : "/home/wildfly",
        "user.language" : "en",
        "user.name" : "wildfly",
        "user.timezone" : "America/New_York"
    },
    "uptime" : 1393273,
    "object-name" : "java.lang:type=Runtime"
}
```

如你所见，有很多信息，并且使用 `operation=attribute` 指令，你可以直接获取你需要的属性。否则使用 `awk` 命令。

# 检查 JVM 选项

在这个菜谱中，你将学习如何通过向 CLI 发送命令来获取运行 WildFly 所使用的 JVM 选项。

## 准备工作

记住我正在远程运行 WildFly，绑定到 IP 地址`192.168.59.103`。WildFly 已经启动并运行。

## 如何操作…

1.  打开一个新的终端窗口并执行以下命令：

    ```java
    $ cd WILDFLY_HOME
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/core-service=platform-mbean/type=runtime:read-attribute(name=input-arguments,include-defaults=true)"
    {
        "outcome" => "success",
        "result" => [
            "-D[Standalone]",
            "-Xms64m",
            "-Xmx512m",
            "-XX:MaxPermSize=256m",
            "-Djava.net.preferIPv4Stack=true",
            "-Djboss.modules.system.pkgs=org.jboss.byteman",
            "-Djava.awt.headless=true",
            "-Dorg.jboss.boot.log.file=/home/wildfly/WFC/wildfly/standalone/log/server.log",
            "-Dlogging.configuration=file:home/wildfly/WFC/wildfly/standalone/configuration/logging.properties"
        ]
    }
    ```

1.  显然，您可以使用输出中的`awk`命令提取所需的信息（您可以使用您熟悉的任何工具），如下所示：

    ```java
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/core-service=platform-mbean/type=runtime:read-attribute(name=input-arguments,include-defaults=true)" | awk 'NR==6 { print $1 }'
    "-Xmx521m",
    ```

1.  如果您在域模式下运行 WildFly，调用方式如下：

    ```java
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/host=master/server=server-one/core-service=platform-mbean/type=runtime:read-attribute(name=input-arguments,include-defaults=true)" | awk 'NR==6 { print $1 }'
    "-Xmx521m",
    ```

就这样！

## 它是如何工作的…

这里我们告诉`jboss-cli.sh`脚本来执行我们在`--command`参数中定义的命令，并将结果返回到标准输出。

使用 CLI，操作如下：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh
You are disconnected at the moment. Type 'connect' to connect to the server or 'help' for the list of supported commands.
[disconnected /] connect 192.168.59.103:9990
Authenticating against security realm: ManagementRealm
Username: wildfly
Password:
[standalone@192.168.59.103:9990 /] /core-service=platform-mbean/type=runtime:read-attribute(name=input-arguments,include-defaults=true)

{
    "outcome" => "success",
    "result" => [
        "-D[Standalone]",
        "-Xms64m",
        "-Xmx512m",
        "-XX:MaxPermSize=256m",
        "-Djava.net.preferIPv4Stack=true",
        "-Djboss.modules.system.pkgs=org.jboss.byteman",
        "-Djava.awt.headless=true",
        "-Dorg.jboss.boot.log.file=/home/wildfly/WFC/wildfly/standalone/log/server.log",
        "-Dlogging.configuration=file:/home/wildfly/WFC/wildfly/standalone/configuration/logging.properties"
    ]
}
[standalone@192.168.59.103:9990 /]
```

记住，您不能操作输出。

## 还有更多…

WildFly 提供了一个额外的 API，即 HTTP API。让我们用一些网络命令行工具，如`curl`来尝试它。

### curl

在一个运行的 WildFly 实例中，执行以下步骤：

1.  打开一个新的终端窗口并按照以下步骤操作：

    ```java
    $ curl --verbose http://192.168.59.103:9990/management/core-service/platform-mbean/type/runtime?operation=attribute\&name=input-arguments\&include-defaults=true
    * Hostname was NOT found in DNS cache
    *   Trying 192.168.59.103...
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    > GET /management/core-service/platform-mbean/type/runtime?operation=attribute&name=input-arguments&include-defaults=true HTTP/1.1
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 401 Unauthorized
    < Connection: keep-alive
    < WWW-Authenticate: Digest realm="ManagementRealm",domain="/management",nonce="Hipx4QD6g24NMTQzMTk2MjY2ODcyNhXQ5pLbS5A/ZruXxypB1gM=",opaque="00000000000000000000000000000000",algorithm=MD5
    < Content-Length: 77
    < Content-Type: text/html
    < Date: Mon, 18 May 2015 15:24:28 GMT
    <
    * Connection #0 to host 192.168.59.103 left intact
    <html><head><title>Error</title></head><body>401 - Unauthorized</body></html>
    ```

    如您所见，`curl`对`ManagementRealm`的摘要认证提出了抱怨，这是 WildFly 管理接口所使用的。

1.  让我们把用户名和密码提供给命令，如下所示：

    ```java
    $ curl --verbose --digest --user wildfly http://192.168.59.103:9990/management/core-service/platform-mbean/type/runtime?operation=attribute\&name=input-arguments\&include-defaults=true
    Enter host password for user 'wildfly':
    * Hostname was NOT found in DNS cache
    *   Trying 192.168.59.103...
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    * Server auth using Digest with user 'wildfly'
    > GET /management/core-service/platform-mbean/type/runtime?operation=attribute&name=input-arguments&include-defaults=true HTTP/1.1
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 401 Unauthorized
    < Connection: keep-alive
    < WWW-Authenticate: Digest realm="ManagementRealm",domain="/management",nonce="QVV7pMN+yIYNMTQzMTk2Mjg1MzAwNIEcpzHTEA2YeGu3jANV1bI=",opaque="00000000000000000000000000000000",algorithm=MD5
    < Content-Length: 77
    < Content-Type: text/html
    < Date: Mon, 18 May 2015 15:27:33 GMT
    <
    * Ignoring the response-body
    * Connection #0 to host 192.168.59.103 left intact
    * Issue another request to this URL: 'http://192.168.59.103:9990/management/core-service/platform-mbean/type/runtime?operation=attribute&name=input-arguments&include-defaults=true'
    * Found bundle for host 192.168.59.103: 0x7fd4c0500e00
    * Re-using existing connection! (#0) with host 192.168.59.103
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    * Server auth using Digest with user 'wildfly'
    > GET /management/core-service/platform-mbean/type/runtime?operation=attribute&name=input-arguments&include-defaults=true HTTP/1.1
    > Authorization: Digest username="wildfly", realm="ManagementRealm", nonce="QVV7pMN+yIYNMTQzMTk2Mjg1MzAwNIEcpzHTEA2YeGu3jANV1bI=", uri="/management/core-service/platform-mbean/type/runtime?operation=attribute&name=input-arguments&include-defaults=true", response="8d550a343d3a56cfcd7afdd6ca0c1664", opaque="00000000000000000000000000000000", algorithm="MD5"
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 200 OK
    < Connection: keep-alive
    < Authentication-Info: nextnonce="UkyiLKmVGuYNMTQzMTk2Mjg1MzAwN08EKUU+895ECweZBtXoCdQ="
    < Content-Type: application/json; charset=utf-8
    < Content-Length: 402
    < Date: Mon, 18 May 2015 15:27:33 GMT
    <
    * Connection #0 to host 192.168.59.103 left intact
    ["-D[Standalone]","-XX:+UseCompressedOops","-XX:+UseCompressedOops","-Xms64m","-Xmx512m","-XX:MaxPermSize=256m","-Djava.net.preferIPv4Stack=true","-Djboss.modules.system.pkgs=org.jboss.byteman","-Djava.awt.headless=true","-Dorg.jboss.boot.log.file=/home/wildfly/WFC/wildfly/standalone/log/server.log","-Dlogging.configuration=file:/home/wildfly/WFC/wildfly/standalone/configuration/logging.properties"]
    ```

1.  好的，它工作了；现在移除`--verbose`标志并再次执行命令。在输入密码后，您应该只得到以下 JSON 输出，如下所示：

    ```java
    ["-D[Standalone]","-XX:+UseCompressedOops","-XX:+UseCompressedOops","-Xms64m","-Xmx512m","-XX:MaxPermSize=256m","-Djava.net.preferIPv4Stack=true","-Djboss.modules.system.pkgs=org.jboss.byteman","-Djava.awt.headless=true","-Dorg.jboss.boot.log.file=/home/wildfly/WFC/wildfly/standalone/log/server.log","-Dlogging.configuration=file:/home/wildfly/WFC/wildfly/standalone/configuration/logging.properties"]
    ```

    很整洁，但看起来很丑！

1.  如果您想美化 JSON 输出并且不想输入密码（您也可以将其作为参数传递，尽管这会带来所有的安全问题），您可以这样做：

    ```java
    $ curl --digest --user wildfly:cookbook.2015 http://192.168.59.103:9990/management/core-service/platform-mbean/type/runtime?operation=attribute\&name=input-arguments\&include-defaults=true\&json.pretty=true[
        "-D[Standalone]",
        "-XX:+UseCompressedOops",
        "-XX:+UseCompressedOops",
        "-Xms64m",
        "-Xmx512m",
        "-XX:MaxPermSize=256m",
        "-Djava.net.preferIPv4Stack=true",
        "-Djboss.modules.system.pkgs=org.jboss.byteman",
        "-Djava.awt.headless=true",
        "-Dorg.jboss.boot.log.file=/home/wildfly/WFC/wildfly/standalone/log/server.log",
        "-Dlogging.configuration=file:/home/wildfly/WFC/wildfly/standalone/configuration/logging.properties"
    ]
    ```

这就是我喜欢的样子！

# 检查 JVM 内存 - 堆大小和所有

在这个菜谱中，我们将学习如何通过调用 CLI 命令来获取 JVM 内存信息，包括堆大小、非堆大小、元空间大小（Java 7 之前的 PermGen）、eden、old 和 survivor 区域，如下所示。每个都有对应的命令。

## 准备工作

记住我正在远程运行 WildFly，绑定到 IP 地址`192.168.59.103`。WildFly 已经启动并运行。

## 如何操作…

我们将分别走过"`heap`"、"`non-heap`"、"`metaspace`"（Java 7 之前的 PermGen）、"`eden`"、"`old`"和"`survivor`"区域内存。

### 堆

要获取堆内存信息，请执行以下步骤：

1.  打开一个新的终端窗口并按照以下步骤操作：

    ```java
    $ cd $WILDFLY_HOME
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/core-service=platform-mbean/type=memory:read-attribute(name=heap-memory-usage,include-defaults=true)"
    {
        "outcome" => "success",
        "result" => {
            "init" => 67108864L,
            "used" => 90009368L,
            "committed" => 199753728L,
            "max" => 477626368L
        }
    }
    ```

1.  显然，您可以使用输出中的`awk`命令提取所需的信息（您可以使用您熟悉的任何工具），如下所示：

    ```java
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/core-service=platform-mbean/type=memory:read-attribute(name=heap-memory-usage,include-defaults=true)" | awk 'NR==7 { print $0 }'
    "max" => 477626368L
    ```

    ### 小贴士

    如果您只需要数字，它是一个`long`数据类型，请在`print`语句中使用`$3`。

1.  如果您在域模式下运行 WildFly，调用方式如下：

    ```java
    $ ./bin/jboss-cli.sh --connect --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/host=master/server=server-one/core-service=platform-mbean/type=memory:read-attribute(name=heap-memory-usage,include-defaults=true)"
    {
        "outcome" => "success",
        "result" => {
            "init" => 67108864L,
            "used" => 78141544L,
            "committed" => 147324928L,
            "max" => 477626368L
        }
    }
    ```

就这样！

### 非堆

要获取非堆内存信息，请执行以下步骤：

1.  打开一个新的终端窗口并执行以下命令：

    ```java
    $ cd $WILDFLY_HOME
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/core-service=platform-mbean/type=memory:read-attribute(name=non-heap-memory-usage,include-defaults=true)"
    {
        "outcome" => "success",
        "result" => {
            "init" => 2555904L,
            "used" => 55613336L,
            "committed" => 62128128L,
            "max" => -1L
        }
    }
    ```

1.  显然，您可以使用输出中的`awk`命令提取所需的信息（您可以使用您熟悉的任何工具），如下所示：

    ```java
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/core-service=platform-mbean/type=memory:read-attribute(name=non-heap-memory-usage,include-defaults=true)" | awk 'NR==5 { print $0 }'
    "used" => 55613336L,
    ```

1.  如果您在域模式下运行 WildFly，调用方式如下：

    ```java
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/host=master/server=server-one/core-service=platform-mbean/type=memory:read-attribute(name=non-heap-memory-usage,include-defaults=true)"
    {
        "outcome" => "success",
        "result" => {
            "init" => 2555904L,
            "used" => 65766592L,
            "committed" => 73310208L,
            "max" => -1L
        }
    }
    ```

就这样！

### Metaspace 或 PermGen

Metaspace 自 Java 1.8 起可用，而 PermGen 直到 Java 1.7 可用。对于元空间内存信息，请执行以下步骤：

1.  打开一个新的终端窗口并按照以下步骤操作：

    ```java
    $ cd $WILDFLY_HOME
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/core-service=platform-mbean/type=memory-pool/name=Metaspace:read-resource(include-runtime=true,include-defaults=true)"
    {
        "outcome" => "success",
        "result" => {
            "name" => "Metaspace",
            "type" => "NON_HEAP",
            "valid" => true,
            "memory-manager-names" => ["Metaspace_Manager"],
            "usage-threshold-supported" => true,
            "collection-usage-threshold-supported" => false,
            "usage-threshold" => 0L,
            "collection-usage-threshold" => undefined,
            "usage" => {
                "init" => 0L,
                "used" => 42415280L,
                "committed" => 47185920L,
                "max" => -1L
            },
            "peak-usage" => {
                "init" => 0L,
                "used" => 42415280L,
                "committed" => 47185920L,
                "max" => -1L
            },
            "usage-threshold-exceeded" => false,
            "usage-threshold-count" => 0L,
            "collection-usage-threshold-exceeded" => undefined,
            "collection-usage-threshold-count" => undefined,
            "collection-usage" => undefined,
            "object-name" => "java.lang:type=MemoryPool,name=Metaspace"
        }
    }
    ```

1.  显然，您可以通过使用输出中的`awk`命令来提取所需的信息（您可以使用您熟悉的任何工具）。然而，在这种情况下，您最好使用`read-attribute`语法，然后使用`awk`命令提取您想要的信息，如下所示：

    ```java
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/core-service=platform-mbean/type=memory-pool/name=Metaspace:read-attribute(name=usage,include-defaults=true)"
    {
        "outcome" => "success",
        "result" => {
            "init" => 0L,
            "used" => 42512536L,
            "committed" => 47448064L,
            "max" => -1L
        }
    }
    ```

1.  如果您在域模式下运行 WildFly，调用方式如下：

    ```java
    $ ./bin/jboss-cli.sh --connect --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/host=master/server=server-one/core-service=platform-mbean/type=memory-pool/name=Metaspace:read-resource(include-runtime=true,include-defaults=true)"
    {
        "outcome" => "success",
        "result" => {
            "name" => "Metaspace",
            "type" => "NON_HEAP",
            "valid" => true,
            "memory-manager-names" => ["Metaspace_Manager"],
            "usage-threshold-supported" => true,
            "collection-usage-threshold-supported" => false,
            "usage-threshold" => 0L,
            "collection-usage-threshold" => undefined,
            "usage" => {
                "init" => 0L,
                "used" => 51111424L,
                "committed" => 56885248L,
                "max" => -1L
            },
            "peak-usage" => {
                "init" => 0L,
                "used" => 51111424L,
                "committed" => 56885248L,
                "max" => -1L
            },
            "usage-threshold-exceeded" => false,
            "usage-threshold-count" => 0L,
            "collection-usage-threshold-exceeded" => undefined,
            "collection-usage-threshold-count" => undefined,
            "collection-usage" => undefined,
            "object-name" => "java.lang:type=MemoryPool,name=Metaspace"
        }
    }
    ```

就这样！

### Eden

要获取 eden 内存信息，请执行以下步骤：

1.  打开一个新的终端窗口并输入以下命令：

    ```java
    $ cd /opt/wildfly
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/core-service=platform-mbean/type=memory-pool/name=PS_Eden_Space:read-resource(include-runtime=true,include-defaults=true)"
    {
        "outcome" => "success",
        "result" => {
            "name" => "PS_Eden_Space",
            "type" => "HEAP",
            "valid" => true,
            "memory-manager-names" => [
                "PS_MarkSweep",
                "PS_Scavenge"
            ],
            "usage-threshold-supported" => false,
            "collection-usage-threshold-supported" => true,
            "usage-threshold" => undefined,
            "collection-usage-threshold" => 0L,
            "usage" => {
                "init" => 16777216L,
                "used" => 61270456L,
                "committed" => 147849216L,
                "max" => 147849216L
            },
            "peak-usage" => {
                "init" => 16777216L,
                "used" => 67108864L,
                "committed" => 147849216L,
                "max" => 173539328L
            },
            "usage-threshold-exceeded" => undefined,
            "usage-threshold-count" => undefined,
            "collection-usage-threshold-exceeded" => false,
            "collection-usage-threshold-count" => 0L,
            "collection-usage" => {
                "init" => 16777216L,
                "used" => 0L,
                "committed" => 147849216L,
                "max" => 147849216L
            },
            "object-name" => "java.lang:type=MemoryPool,name=\"PS Eden Space\""
        }
    }
    ```

1.  显然，您可以通过使用输出中的`awk`命令来提取所需的信息（您可以使用您熟悉的任何工具）。然而，在这种情况下，您最好使用`read-attribute`语法，然后`awk`您想要的信息，如下所示：

    ```java
    $ ./bin/jboss-cli.sh --connect --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/core-service=platform-mbean/type=memory-pool/name=PS_Eden_Space:read-attribute(name=usage,include-defaults=true)"
    {
        "outcome" => "success",
        "result" => {
            "init" => 16777216L,
            "used" => 70648432L,
            "committed" => 147849216L,
            "max" => 147849216L
        }
    }
    ```

1.  如果您在域模式下运行 WildFly，调用方式如下：

    ```java
    $ ./bin/jboss-cli.sh --connect --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/host=master/server=server-one/core-service=platform-mbean/type=memory-pool/name=PS_Eden_Space:read-resource(include-runtime=true,include-defaults=true)"
    {
        "outcome" => "success",
        "result" => {
            "name" => "PS_Eden_Space",
            "type" => "HEAP",
            "valid" => true,
            "memory-manager-names" => [
                "PS_MarkSweep",
                "PS_Scavenge"
            ],
            "usage-threshold-supported" => false,
            "collection-usage-threshold-supported" => true,
            "usage-threshold" => undefined,
            "collection-usage-threshold" => 0L,
            "usage" => {
                "init" => 16777216L,
                "used" => 40625992L,
                "committed" => 58720256L,
                "max" => 144703488L
            },
            "peak-usage" => {
                "init" => 16777216L,
                "used" => 58720256L,
                "committed" => 58720256L,
                "max" => 173539328L
            },
            "usage-threshold-exceeded" => undefined,
            "usage-threshold-count" => undefined,
            "collection-usage-threshold-exceeded" => false,
            "collection-usage-threshold-count" => 0L,
            "collection-usage" => {
                "init" => 16777216L,
                "used" => 0L,
                "committed" => 58720256L,
                "max" => 144703488L
            },
            "object-name" => "java.lang:type=MemoryPool,name=\"PS Eden Space\""
        }
    }
    ```

就这样！

### 旧

要获取旧内存信息，请执行以下步骤：

1.  打开一个新的终端窗口并执行以下操作：

    ```java
    $ cd /opt/wildfly
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/core-service=platform-mbean/type=memory-pool/name=PS_Old_Gen:read-resource(include-runtime=true,include-defaults=true)"{
        "outcome" => "success",
        "result" => {
            "name" => "PS_Old_Gen",
            "type" => "HEAP",
            "valid" => true,
            "memory-manager-names" => ["PS_MarkSweep"],
            "usage-threshold-supported" => true,
            "collection-usage-threshold-supported" => true,
            "usage-threshold" => 0L,
            "collection-usage-threshold" => 0L,
            "usage" => {
                "init" => 45088768L,
                "used" => 17330048L,
                "committed" => 59244544L,
                "max" => 358088704L
            },
            "peak-usage" => {
                "init" => 45088768L,
                "used" => 17330048L,
                "committed" => 59244544L,
                "max" => 358088704L
            },
            "usage-threshold-exceeded" => false,
            "usage-threshold-count" => 0L,
            "collection-usage-threshold-exceeded" => false,
            "collection-usage-threshold-count" => 0L,
            "collection-usage" => {
                "init" => 45088768L,
                "used" => 17330048L,
                "committed" => 59244544L,
                "max" => 358088704L
            },
            "object-name" => "java.lang:type=MemoryPool,name=\"PS Old Gen\""
        }
    }
    ```

1.  显然，您可以通过使用输出中的`awk`命令来提取所需的信息（您可以使用您熟悉的任何工具）。然而，在这种情况下，您最好使用`read-attribute`语法，然后`awk`您想要的信息，如下所示：

    ```java
    $ ./bin/jboss-cli.sh --connect --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/core-service=platform-mbean/type=memory-pool/name=PS_Old_Gen:read-attribute(name=usage,include-defaults=true)"{
        "outcome" => "success",
        "result" => {
            "init" => 45088768L,
            "used" => 17330048L,
            "committed" => 59244544L,
            "max" => 358088704L
        }
    }
    ```

1.  如果您在域模式下运行 WildFly，调用方式如下：

    ```java
    $ ./bin/jboss-cli.sh --connect --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/host=master/server=server-one/core-service=platform-mbean/type=memory-pool/name=PS_Old_Gen:read-resource(include-runtime=true,include-defaults=true)"
    {
        "outcome" => "success",
        "result" => {
            "name" => "PS_Old_Gen",
            "type" => "HEAP",
            "valid" => true,
            "memory-manager-names" => ["PS_MarkSweep"],
            "usage-threshold-supported" => true,
            "collection-usage-threshold-supported" => true,
            "usage-threshold" => 0L,
            "collection-usage-threshold" => 0L,
            "usage" => {
                "init" => 45088768L,
                "used" => 21521968L,
                "committed" => 70254592L,
                "max" => 358088704L
            },
            "peak-usage" => {
                "init" => 45088768L,
                "used" => 21521968L,
                "committed" => 70254592L,
                "max" => 358088704L
            },
            "usage-threshold-exceeded" => false,
            "usage-threshold-count" => 0L,
            "collection-usage-threshold-exceeded" => false,
            "collection-usage-threshold-count" => 0L,
            "collection-usage" => {
                "init" => 45088768L,
                "used" => 18670576L,
                "committed" => 70254592L,
                "max" => 358088704L
            },
            "object-name" => "java.lang:type=MemoryPool,name=\"PS Old Gen\""
        }
    }
    ```

就这样！

### 幸存者

要获取幸存者内存信息，请执行以下步骤：

1.  打开一个新的终端窗口并输入以下命令：

    ```java
    $ cd /opt/wildfly
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/core-service=platform-mbean/type=memory-pool/name=PS_Survivor_Space:read-resource(include-runtime=true,include-defaults=true)"
    {
        "outcome" => "success",
        "result" => {
            "name" => "PS_Survivor_Space",
            "type" => "HEAP",
            "valid" => true,
            "memory-manager-names" => [
                "PS_MarkSweep",
                "PS_Scavenge"
            ],
            "usage-threshold-supported" => false,
            "collection-usage-threshold-supported" => true,
            "usage-threshold" => undefined,
            "collection-usage-threshold" => 0L,
            "usage" => {
                "init" => 2621440L,
                "used" => 0L,
                "committed" => 15728640L,
                "max" => 15728640L
            },
            "peak-usage" => {
                "init" => 2621440L,
                "used" => 12809608L,
                "committed" => 15728640L,
                "max" => 15728640L
            },
            "usage-threshold-exceeded" => undefined,
            "usage-threshold-count" => undefined,
            "collection-usage-threshold-exceeded" => false,
            "collection-usage-threshold-count" => 0L,
            "collection-usage" => {
                "init" => 2621440L,
                "used" => 0L,
                "committed" => 15728640L,
                "max" => 15728640L
            },
            "object-name" => "java.lang:type=MemoryPool,name=\"PS Survivor Space\""
        }
    }
    ```

1.  显然，您可以通过使用输出中的`awk`命令来提取所需的信息（您可以使用您熟悉的任何工具）。然而，在这种情况下，您最好使用`read-attribute`语法，然后使用`awk`命令提取您想要的信息，如下所示：

    ```java
    $ ./bin/jboss-cli.sh --connect --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/core-service=platform-mbean/type=memory-pool/name=PS_Survivor_Space:read-attribute(name=usage,include-defaults=true)"
    {
        "outcome" => "success",
        "result" => {
            "init" => 2621440L,
            "used" => 0L,
            "committed" => 15728640L,
            "max" => 15728640L
        }
    }
    ```

1.  如果您在域模式下运行 WildFly，调用方式如下：

    ```java
    $ ./bin/jboss-cli.sh --connect --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/host=master/server=server-one/core-service=platform-mbean/type=memory-pool/name=PS_Survivor_Space:read-resource(include-runtime=true,include-defaults=true)"
    {
        "outcome" => "success",
        "result" => {
            "name" => "PS_Survivor_Space",
            "type" => "HEAP",
            "valid" => true,
            "memory-manager-names" => [
                "PS_MarkSweep",
                "PS_Scavenge"
            ],
            "usage-threshold-supported" => false,
            "collection-usage-threshold-supported" => true,
            "usage-threshold" => undefined,
            "collection-usage-threshold" => 0L,
            "usage" => {
                "init" => 2621440L,
                "used" => 15199856L,
                "committed" => 15204352L,
                "max" => 15204352L
            },
            "peak-usage" => {
                "init" => 2621440L,
                "used" => 15199856L,
                "committed" => 15204352L,
                "max" => 15204352L
            },
            "usage-threshold-exceeded" => undefined,
            "usage-threshold-count" => undefined,
            "collection-usage-threshold-exceeded" => false,
            "collection-usage-threshold-count" => 0L,
            "collection-usage" => {
                "init" => 2621440L,
                "used" => 15199856L,
                "committed" => 15204352L,
                "max" => 15204352L
            },
            "object-name" => "java.lang:type=MemoryPool,name=\"PS Survivor Space\""
        }
    }
    ```

就这样！

## 它是如何工作的……

在这里，我们告诉`jboss-cli.sh`脚本来执行我们在`--command`参数中定义的命令，并将结果返回到标准输出。

使用 CLI，它将是这样的：

```java
$ ./bin/jboss-cli.sh
You are disconnected at the moment. Type 'connect' to connect to the server or 'help' for the list of supported commands.
[disconnected /] connect 192.168.59.103:9990
Authenticating against security realm: ManagementRealm
Username: wildfly
Password:
[standalone@192.168.59.103:9990 /] /core-service=platform-mbean/type=memory:read-attribute(name=heap-memory-usage,include-defaults=true)
{
    "outcome" => "success",
    "result" => {
        "init" => 67108864L,
        "used" => 76422344L,
        "committed" => 175112192L,
        "max" => 477626368L
    }
}
[standalone@192.168.59.103:9990 /]
```

顺便说一下，您无法操作输出。

您可以尝试自己替换感兴趣的内存名称，以尝试其他内存空间。

## 还有更多……

WildFly 提供另一个 API，即 HTTP API。让我们用一些网络命令行工具，如`curl`来尝试它。

这里我们将以 JVM 堆内存为例进行考察。

### Curl

使用 curl 执行以下步骤：

1.  在一个运行的 WildFly 实例中，打开一个新的终端窗口并执行以下命令：

    ```java
    $ curl --verbose http://192.168.59.103:9990/management/core-service/platform-mbean/type/memory?operation=attribute\&name=heap-memory-usage
    * Hostname was NOT found in DNS cache
    *   Trying 192.168.59.103...
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    > GET /management/core-service/platform-mbean/type/memory?operation=attribute&name=heap-memory-usage HTTP/1.1
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 401 Unauthorized
    < Connection: keep-alive
    < WWW-Authenticate: Digest realm="ManagementRealm",domain="/management",nonce="SwXvsbP7j0UNMTQzMjAxNjUxODY4Ml1gpwERAQYWFmaOqeygbx4=",opaque="00000000000000000000000000000000",algorithm=MD5
    < Content-Length: 77
    < Content-Type: text/html
    < Date: Tue, 19 May 2015 06:21:58 GMT
    <
    * Connection #0 to host 192.168.59.103 left intact
    <html><head><title>Error</title></head><body>401 - Unauthorized</body></html>
    ```

    如您所见，`curl`对`ManagementRealm`的摘要认证提出了抱怨，这是 WildFly 管理接口所使用的。

1.  让我们给命令提供用户名和密码，如下所示：

    ```java
    $ curl --verbose --digest --user wildfly http://192.168.59.103:9990/management/core-service/platform-mbean/type/memory?operation=attribute\&name=heap-memory-usage
    Enter host password for user 'wildfly':
    * Hostname was NOT found in DNS cache
    *   Trying 192.168.59.103...
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    * Server auth using Digest with user 'wildfly'
    > GET /management/core-service/platform-mbean/type/memory?operation=attribute&name=heap-memory-usage HTTP/1.1
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 401 Unauthorized
    < Connection: keep-alive
    < WWW-Authenticate: Digest realm="ManagementRealm",domain="/management",nonce="47ky6duDTAANMTQzMjAxNjY2NTc1NDQr1KuuN1GzJ50GQ0PUuCs=",opaque="00000000000000000000000000000000",algorithm=MD5
    < Content-Length: 77
    < Content-Type: text/html
    < Date: Tue, 19 May 2015 06:24:25 GMT
    <
    * Ignoring the response-body
    * Connection #0 to host 192.168.59.103 left intact
    * Issue another request to this URL: 'http://192.168.59.103:9990/management/core-service/platform-mbean/type/memory?operation=attribute&name=heap-memory-usage'
    * Found bundle for host 192.168.59.103: 0x7fa919d00dd0
    * Re-using existing connection! (#0) with host 192.168.59.103
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    * Server auth using Digest with user 'wildfly'
    > GET /management/core-service/platform-mbean/type/memory?operation=attribute&name=heap-memory-usage HTTP/1.1
    > Authorization: Digest username="wildfly", realm="ManagementRealm", nonce="47ky6duDTAANMTQzMjAxNjY2NTc1NDQr1KuuN1GzJ50GQ0PUuCs=", uri="/management/core-service/platform-mbean/type/memory?operation=attribute&name=heap-memory-usage", response="f04575d464d4151697dc6468610ec989", opaque="00000000000000000000000000000000", algorithm="MD5"
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 200 OK
    < Connection: keep-alive
    < Authentication-Info: nextnonce="kFsM2zG0/XANMTQzMjAxNjY2NTc2MP4TtqHrlyzAujuq4ZLn9ek="
    < Content-Type: application/json; charset=utf-8
    < Content-Length: 82
    < Date: Tue, 19 May 2015 06:24:25 GMT
    <
    * Connection #0 to host 192.168.59.103 left intact
    {"init" : 67108864, "used" : 62314024, "committed" : 222298112, "max" : 477626368}
    ```

1.  好的，它工作了；现在移除`--verbose`标志并再次执行命令。输入密码后，你应该只得到以下 JSON 输出：

    ```java
    {"init" : 67108864, "used" : 66257248, "committed" : 222298112, "max" : 477626368}
    ```

    很酷，但看起来很丑！

1.  如果你想要美化 JSON 输出并且不想输入密码（你也可以作为参数传递，但会带来所有的安全顾虑），你可以执行以下命令：

    ```java
    $ curl --digest --user wildfly:cookbook.2015 http://192.168.59.103:9990/management/core-service/platform-mbean/type/memory?operation=attribute\&name=heap-memory-usage\&json.pretty=true{
        "init" : 67108864,
        "used" : 67826544,
        "committed" : 222298112,
        "max" : 477626368
    }
    ```

这就是我喜欢的样子！

# 检查服务器状态

在这个菜谱中，我们将学习如何通过调用 CLI 命令来检查运行中的 WildFly 实例的状态。

## 准备中

记住我正在远程运行 WildFly，绑定到 IP 地址`192.168.59.103`。WildFly 已经启动并运行。

## 如何操作…

打开一个新的终端窗口并执行以下命令：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command=":read-attribute(name=server-state)"
{
    "outcome" => "success",
    "result" => "running"
}
```

你可以通过在输出中使用`awk`命令提取所需的信息（你可以使用你熟悉的任何工具），如下所示：

```java
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command=":read-attribute(name=server-state)" | awk 'NR==3 { print $3 }'
"running"
```

如果你以域模式运行 WildFly，调用方式如下：

```java
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/host=master/server=server-one:read-attribute(name=server-state)" | awk 'NR==3 { print $3 }'
"running"
```

就这样了！

## 它是如何工作的…

在这里，我们告诉`jboss-cli.sh`脚本来执行我们在`--command`参数中定义的命令，并将结果输出到标准输出。

使用 CLI，操作如下：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh
You are disconnected at the moment. Type 'connect' to connect to the server or 'help' for the list of supported commands.
[disconnected /] connect 192.168.59.103:9990
Authenticating against security realm: ManagementRealm
Username: wildfly
Password:
[standalone@192.168.59.103:9990 /] :read-attribute(name=server-state)
{
    "outcome" => "success",
    "result" => "running"
}
[standalone@192.168.59.103:9990 /]
```

顺便说一下，你不能操纵输出。

## 还有更多…

WildFly 提供另一个 API，即 HTTP API。让我们用一些网络命令行工具，如`curl`来尝试它。

### curl

在一个运行的 WildFly 实例中，执行以下步骤：

1.  打开一个新的终端窗口并执行以下命令：

    ```java
    $ curl --verbose http://192.168.59.103:9990/management/?operation=attribute\&name=server-state
    * Hostname was NOT found in DNS cache
    *   Trying 192.168.59.103...
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    > GET /management/?operation=attribute&name=server-state HTTP/1.1
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 401 Unauthorized
    < Connection: keep-alive
    < WWW-Authenticate: Digest realm="ManagementRealm",domain="/management",nonce="kv+FBaxYgDINMTQzMjAxNzUzMTYyNbC9h3WcIBNOnUH1mBkR1vY=",opaque="00000000000000000000000000000000",algorithm=MD5
    < Content-Length: 77
    < Content-Type: text/html
    < Date: Tue, 19 May 2015 06:38:51 GMT
    <
    * Connection #0 to host 192.168.59.103 left intact
    <html><head><title>Error</title></head><body>401 - Unauthorized</body></html>
    ```

    如你所见，`curl`对`ManagementRealm`的摘要认证提出了抱怨，这是 WildFly 管理接口所使用的。

1.  让我们给命令提供用户名和密码，如下所示：

    ```java
    $ curl --verbose --digest --user wildfly http://192.168.59.103:9990/management/?operation=attribute\&name=server-state
    Enter host password for user 'wildfly':
    * Hostname was NOT found in DNS cache
    *   Trying 192.168.59.103...
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    * Server auth using Digest with user 'wildfly'
    > GET /management/?operation=attribute&name=server-state HTTP/1.1
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 401 Unauthorized
    < Connection: keep-alive
    < WWW-Authenticate: Digest realm="ManagementRealm",domain="/management",nonce="Pl2q17HyCSsNMTQzMjAxNzU0NTUxOC3ZiDpK6wbWOysSkGTtFd8=",opaque="00000000000000000000000000000000",algorithm=MD5
    < Content-Length: 77
    < Content-Type: text/html
    < Date: Tue, 19 May 2015 06:39:05 GMT
    <
    * Ignoring the response-body
    * Connection #0 to host 192.168.59.103 left intact
    * Issue another request to this URL: 'http://192.168.59.103:9990/management/?operation=attribute&name=server-state'
    * Found bundle for host 192.168.59.103: 0x7f897b500c70
    * Re-using existing connection! (#0) with host 192.168.59.103
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    * Server auth using Digest with user 'wildfly'
    > GET /management/?operation=attribute&name=server-state HTTP/1.1
    > Authorization: Digest username="wildfly", realm="ManagementRealm", nonce="Pl2q17HyCSsNMTQzMjAxNzU0NTUxOC3ZiDpK6wbWOysSkGTtFd8=", uri="/management/?operation=attribute&name=server-state", response="7b740a1695f01a9041a3d0c61cf35c91", opaque="00000000000000000000000000000000", algorithm="MD5"
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    >
    < HTTP/1.1 200 OK
    < Connection: keep-alive
    < Authentication-Info: nextnonce="3gZb5DbobbENMTQzMjAxNzU0NTUzOD7wAHJMcsZDujeJP3F/N9M="
    < Content-Type: application/json; charset=utf-8
    < Content-Length: 9
    < Date: Tue, 19 May 2015 06:39:05 GMT
    <
    * Connection #0 to host 192.168.59.103 left intact
    "running"
    ```

1.  好的，它工作了；现在移除`--verbose`标志并再次执行命令。输入密码后，你应该只得到`running`。

1.  如果你不想输入密码，你也可以作为参数传递，但会带来所有的安全顾虑。为此，执行以下命令：

    ```java
    $ curl --verbose --digest --user wildfly:cookbook.2015 http://192.168.59.103:9990/management/?operation=attribute \&name=server-state
    "running"
    ```

# 检查 JNDI 树视图

在这个菜谱中，我们将学习如何通过调用 CLI 命令来获取你的 WildFly 实例的 JNDI 树视图。这可能在你需要检查某些应用程序上下文是否存在、需要知道数据源 JNDI 名称或需要查找 EJB 时很有用。

## 准备中

记住我正在远程运行 WildFly，绑定到 IP 地址`192.168.59.103`。WildFly 已经启动并运行。

## 如何操作…

1.  打开一个新的终端窗口并执行以下命令：

    ```java
    $ cd $WILDFLY_HOME
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 –command="/subsystem=naming:jndi-view"
    {
        "outcome" => "success",
        "result" => {
            "java: contexts" => {
                "java:" => {"TransactionManager" => {
                    "class-name" => "com.arjuna.ats.jbossatx.jta.TransactionManagerDelegate",
                    "value" => "com.arjuna.ats.jbossatx.jta.TransactionManagerDelegate@16b89a30"
                }},
                "java:jboss" => {
                    "TransactionManager" => {
                        "class-name" => "com.arjuna.ats.jbossatx.jta.TransactionManagerDelegate",
                        "value" => "com.arjuna.ats.jbossatx.jta.TransactionManagerDelegate@16b89a30"
                    },
                    "TransactionSynchronizationRegistry" => {
                        "class-name" => "org.jboss.as.txn.service.internal.tsr.TransactionSynchronizationRegistryWrapper",
                        "value" => "org.jboss.as.txn.service.internal.tsr.TransactionSynchronizationRegistryWrapper@315ee6c0"
                    },
                    "UserTransaction" => {
                        "class-name" => "javax.transaction.UserTransaction",
                        "value" => "UserTransaction"
                    },
                    "jaas" => {
                        "class-name" => "com.sun.proxy.$Proxy11",
                        "value" => "java:jboss/jaas/ Context proxy"
                    },
                    "ee" => {
                        "class-name" => "javax.naming.Context",
                        "children" => {"concurrency" => {
                            "class-name" => "javax.naming.Context",
                            "children" => {
                                "scheduler" => {
                                    "class-name" => "javax.naming.Context",
                                    "children" => {"default" => {
                                        "class-name" => "java.lang.Object",
                                        "value" => "?"
                                    }}
                                },
                                "factory" => {
                                    "class-name" => "javax.naming.Context",
                                    "children" => {"default" => {
                                        "class-name" => "java.lang.Object",
                                        "value" => "?"
                                    }}
                                },
                                "executor" => {
                                    "class-name" => "javax.naming.Context",
                                    "children" => {"default" => {
                                        "class-name" => "java.lang.Object",
                                        "value" => "?"
                                    }}
                                },
                                "context" => {
                                    "class-name" => "javax.naming.Context",
                                    "children" => {"default" => {
                                        "class-name" => "java.lang.Object",
                                        "value" => "?"
                                    }}
                                }
                            }
                        }}
                    },
                    "datasources" => {
                        "class-name" => "javax.naming.Context",
                        "children" => {"ExampleDS" => {
                            "class-name" => "org.jboss.as.connector.subsystems.datasources.WildFlyDataSource",
                            "value" => "org.jboss.as.connector.subsystems.datasources.WildFlyDataSource@59b69704"
                        }}
                    },
                    "mail" => {
                        "class-name" => "javax.naming.Context",
                        "children" => {"Default" => {
                            "class-name" => "javax.mail.Session",
                            "value" => "javax.mail.Session@25c2acd6"
                        }}
                    }
                },
                "java:jboss/exported" => undefined,
                "java:global" => undefined
            },
            "applications" => undefined
        }
    }
    ```

    显然，你可以通过在输出中使用`awk`命令提取所需的信息（你可以使用你熟悉的任何工具），如下所示：

1.  如果你以域模式运行 WildFly，调用方式如下：

    ```java
    $ cd $WILDFLY_HOME
    $ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command='/host=master/server=server-one/subsystem=naming:jndi-view'
    {
        "outcome" => "success",
        "result" => {
            "java: contexts" => {
                ...
            },
            "applications" => {"example.war" => {
                "java:app" => {
                    "AppName" => {
                        "class-name" => "java.lang.String",
                        "value" => "example"
                    },
                    "env" => {
                        "class-name" => "org.jboss.as.naming.NamingContext",
                        "value" => "env"
                    }
                },
                "modules" => undefined
            }}
        }
    }
    ```

就这样了！

## 它是如何工作的…

在这里，我们告诉`jboss-cli.sh`脚本来执行我们在`--command`参数中定义的命令，并将结果输出到标准输出。

使用 CLI，操作如下：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh
You are disconnected at the moment. Type 'connect' to connect to the server or 'help' for the list of supported commands.
[disconnected /] connect 192.168.59.103:9990
Authenticating against security realm: ManagementRealm
Username: wildfly
Password:
[standalone@192.168.59.103:9990 /] /subsystem=naming:jndi-view
....
```

输出将与本食谱中*如何操作*部分的图像显示相同。

# 调用声明在外部文件中的 CLI 命令

在本食谱中，我们将学习如何使用`jboss-cli.sh`脚本执行在单独文件中声明的命令。

## 准备工作

记住，我正在远程运行 WildFly，绑定到 IP 地址`192.168.59.103`。WildFly 已经启动并运行。

创建一个名为`wildfly-cookbook.cli`的文件，并将列表命令`ls`插入其中。将文件放置在你的本地`$WILDFLY_HOME`文件夹中。

现在是时候通过 CLI 调用我们的命令了！

## 如何操作…

打开一个新的终端窗口并执行以下操作：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --file=wildfly-cookbook.cli
core-service
deployment
deployment-overlay
extension
interface
path
socket-binding-group
subsystem
system-property
launch-type=STANDALONE
management-major-version=3
management-micro-version=0
management-minor-version=0
name=7536a491dba6
namespaces=[]
process-type=Server
product-name=WildFly Full
product-version=9.0.0.Beta2
profile-name=undefined
release-codename=Kenny
release-version=1.0.0.Beta2
running-mode=NORMAL
schema-locations=[]
server-state=running
suspend-state=RUNNING
```

## 它是如何工作的…

在这里，我们告诉`jboss-cli.sh`脚本通过`--file`指令读取`wildfly-cookbook.2015`文件内定义的命令。CLI 读取我们的`wildfly-cookbook.cli`文件并执行其命令。

使用文件来存储配置你的 WildFly 实例所需的命令非常有用，可以自动化配置。

想象一个需要自动扩展系统的场景。假设你的所有应用程序源代码都在版本控制软件仓库中，例如`git`，以及`.cli`配置文件。下载你的应用程序，构建它，使用`.cli`文件配置 WildFly 实例，部署应用程序，并启动服务器将变得轻而易举！

在下一章中，我们将讨论如何停止和启动服务器，使用 CLI 部署应用程序，以及更多内容。
