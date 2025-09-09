# 第九章. 克服 CLI

在本章中，你将学习以下食谱：

+   调用服务器停止、启动和重新加载

+   调用服务器组停止、启动、重启和重新加载

+   创建服务器组

+   创建服务器

+   管理应用程序 – 部署、卸载

# 简介

在本章中，你将学习如何使用 CLI 来改变系统的状态。也就是说，改变不同的设置，如部署和创建新服务器。在前一章中，我们看到了如何从 CLI 中获取信息。CLI 还提供了一个方法来执行它外的命令，通过指定要连接的 WildFly 和要执行的命令。

此外，WildFly 为你提供了一个 HTTP API，可以用来执行操作和检索信息（大多数这些 API 用于执行系统监控分析）。在前一章中，我们使用 CLI 来获取信息；因此，通过 HTTP，我们使用了 HTTP 协议的 GET 动词。在下面的食谱中，我们将使用 POST 动词以及 WildFly HTTP API 需要的参数来改变服务器的状态。HTTP API 只接受 JSON 数据，因此我们需要以这种方式发送数据。

当有道理时，我们将使用两种操作模式中的方法——独立模式和域模式，因为关于操作模式，你可能有不同的入口点。

在本章中，我们将模拟/使用远程 WildFly 实例，就像在实际场景中一样，你可以应用以下食谱来连接到远程服务器。在没有看到任何身份验证和授权问题的情况下，尝试`localhost`上的食谱是没有意义的。

为了模拟远程服务器，你最终可以使用 VirtualBox 或 Docker，然后按照第一章中描述的方式安装 WildFly。

我将使用 Docker 工具版本 1.5 在 Linux 容器上运行的 WildFly。显然，你可以使用真实的远程服务器——那将是相同的；真正重要的是 WildFly 平台是否被暴露。

顺便说一句，这本书的最后一章全是关于在 Linux 容器中使用 Docker 运行 WildFly 的内容。所以如果你对 Docker 一无所知（你一直躲在哪里？），可以看看这本书的最后一章，或者获取优秀的 Docker 书籍，《Docker Book》，作者*詹姆斯·特布尔*，[`www.dockerbook.com`](http://www.dockerbook.com)。

我的 Docker 配置如下：

```java
DOCKER_HOST=tcp://192.168.59.103:2376
```

因此，我的远程 WildFly 实例将绑定到该 IP，以及 WildFly 的常用端口，如`8080`、`9990`和`9999`。本书中管理用户保持不变，用户名为`wildfly`，密码为`cookbook.2015`。

# 调用服务器停止、启动和重新加载

在这个食谱中，我们将学习如何通过向 CLI 发出命令来停止、启动和重新加载一个 WildFly 实例。你可能需要手动停止服务器来纠正错误配置或重新部署应用程序。因此，了解如何停止、启动和重新加载服务器是必须的。

## 准备工作

记住，我正在远程运行 WildFly，绑定到`192.168.59.103`作为 IP。WildFly 已经启动并运行。

## 如何做到这一点...

我们将分别介绍`stop`、`start`、`restart`和`reload`命令，以更好地解释它们的调用方式，并最终解释它们之间的差异。例如，`start`命令只在域模式下有意义，因为在独立模式下启动 WildFly 是一个手动操作。

### 停止

打开一个新的终端窗口并执行以下命令：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command=":shutdown(restart=false)"
{"outcome" => "success"}
```

显然，在这个命令之后，你需要手动启动 WildFly。这是因为主机控制器进程与 WildFly 实例一起关闭了，因此，你无法访问管理界面来控制你的实例。

如果你正在域模式下运行 WildFly，你首先需要确定你想要停止的主机和服务器，然后调用停止方法，如下所示：

```java
$ ./bin/jboss-cli.sh --connect --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/host=master/server-config=server-one:stop()"
{
    "outcome" => "success",
    "result" => "STOPPING"
}
```

### 启动

这个命令只在域模式下有意义，其中你连接到域控制器。然后你可以向各个活动的主机控制器推送命令，如以下所示，来启动或关闭 WildFly 实例。

打开一个新的终端窗口并执行以下命令：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh --connect --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/host=master/server-config=server-one:start()"
{
    "outcome" => "success",
    "result" => "STARTING"
}
```

### 重新启动

打开一个新的终端窗口并运行以下代码：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command=":shutdown(restart=true)"
{"outcome" => "success"}
```

在前面的`stop`部分，你可能想知道，将`restart`参数设置为`true`传递给`shutdown`方法可能会重新启动 WildFly 实例，实际上它确实这样做了。

顺便说一句，没有与域模式对应的模式。它只适用于服务器组，这在本章后面会讨论。

### 重新加载

打开一个新的终端窗口并执行以下操作：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command=":reload()"
{
    "outcome" => "success",
    "result" => undefined
}
```

`reload`方法只是关闭所有活动的 WildFly 服务，解绑资源，然后再次启动它们，重新加载配置。

如果你正在域模式下运行 WildFly，调用的目的仍然是相同的——重新加载服务器配置；你可以这样做：

```java
$ ./bin/jboss-cli.sh --connect --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command=":reload-servers()"
{
    "outcome" => "success",
    "result" => undefined,
    "server-groups" => undefined
}
```

## 它是如何工作的...

在这里，我们正在告诉`jboss-cli.sh`脚本来执行我们在`--command`参数中定义的命令，并将结果返回到标准输出。

## 还有更多...

WildFly 提供另一个 API，即 HTTP API。让我们用一些网络命令行工具，如`curl`来尝试它。

### Curl

1.  在一个运行的 WildFly 实例中，打开一个新的终端窗口并输入以下命令：

    ```java
    $ curl --verbose http://192.168.59.103:9990/management/ -X POST -H "Content-Type: application/json" -d '{"operation":"reload"}'
    ```

1.  你应该得到以下输出：

    ```java
    * Hostname was NOT found in DNS cache
    *   Trying 192.168.59.103...
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    > POST /management/ HTTP/1.1
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    > Content-Type: application/json
    > Content-Length: 22
    >
    * upload completely sent off: 22 out of 22 bytes
    < HTTP/1.1 401 Unauthorized
    < Connection: keep-alive
    < WWW-Authenticate: Digest realm="ManagementRealm",domain="/management",nonce="//unXxm6vjoNMTQzNTA1MTg1NDUyNtkzOlQPF6rdPZp9cTcQnhY=",opaque="00000000000000000000000000000000",algorithm=MD5
    < Content-Length: 77
    < Content-Type: text/html
    < Date: Tue, 23 Jun 2015 09:30:44 GMT
    <
    * Connection #0 to host 192.168.59.103 left intact
    <html><head><title>Error</title></head><body>401 - Unauthorized</body></html>
    ```

    正如你所看到的，`curl`对`ManagementRealm`的摘要认证提出了抱怨，这是 WildFly 管理接口所使用的。

1.  让我们给命令提供用户名和密码，如下所示：

    ```java
    $ curl --verbose --digest --user wildfly http://192.168.59.103:9990/management/ -X POST -H "Content-Type: application/json" -d '{"operation":"reload"}'
    ```

1.  你应该得到以下输出：

    ```java
    Enter host password for user 'wildfly':
    * Hostname was NOT found in DNS cache
    *   Trying 192.168.59.103...
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    * Server auth using Digest with user 'wildfly'
    > POST /management/ HTTP/1.1
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    > Content-Type: application/json
    > Content-Length: 0
    >
    < HTTP/1.1 401 Unauthorized
    < Connection: keep-alive
    < WWW-Authenticate: Digest realm="ManagementRealm",domain="/management",nonce="urmPLpGMJgkNMTQzNTA1MjEwMTI4NJYIEw20Br2YWefE0GbBBk8=",opaque="00000000000000000000000000000000",algorithm=MD5
    < Content-Length: 77
    < Content-Type: text/html
    < Date: Tue, 23 Jun 2015 09:30:44 GMT
    <
    * Ignoring the response-body
    * Connection #0 to host 192.168.59.103 left intact
    * Issue another request to this URL: 'http://192.168.59.103:9990/management/'
    * Found bundle for host 192.168.59.103: 0x7fc1104149d0
    * Re-using existing connection! (#0) with host 192.168.59.103
    * Connected to 192.168.59.103 (192.168.59.103) port 9990 (#0)
    * Server auth using Digest with user 'wildfly'
    > POST /management/ HTTP/1.1
    > Authorization: Digest username="wildfly", realm="ManagementRealm", nonce="urmPLpGMJgkNMTQzNTA1MjEwMTI4NJYIEw20Br2YWefE0GbBBk8=", uri="/management/", response="0bbe505bc1af3907ff3a8ca960387829", opaque="00000000000000000000000000000000", algorithm="MD5"
    > User-Agent: curl/7.37.1
    > Host: 192.168.59.103:9990
    > Accept: */*
    > Content-Type: application/json
    > Content-Length: 22
    >
    * upload completely sent off: 22 out of 22 bytes
    < HTTP/1.1 200 OK
    < Connection: keep-alive
    < Authentication-Info: nextnonce="oIOIlHtymVgNMTQzNTA1MjEwMTI4OZI07WcGItS8EhmrBTUgrCU="
    < Content-Type: application/json; charset=utf-8
    < Content-Length: 40
    < Date: Tue, 23 Jun 2015 09:30:44 GMT
    <
    * Connection #0 to host 192.168.59.103 left intact
    {"outcome" : "success", "result" : null}
    ```

1.  好的，它工作了；现在移除`--verbose`标志并再次执行命令。在输入密码后，你应该得到以下 JSON 输出：

    ```java
    $ curl --digest --user wildfly:cookbook.2015 http://192.168.59.103:9990/management/ -X POST -H "Content-Type: application/json" -d '{"operation":"reload"}'
    {"outcome" : "success", "result" : null}
    $
    ```

    ### 注意

    我选择`reload`方法只是为了向你展示如何使用 HTTP API 来做这件事。

# 调用服务器组停止、启动、重启和重新加载

在这个菜谱中，我们将学习如何通过向 CLI 发送命令来停止、启动、重启和重新加载 WildFly 服务器组。正如你所知，你可以停止、启动、重启和重新加载属于服务器组的单个服务器。我们将使用 CLI 查看所有这些命令。

由于我们正在讨论服务器组，这个菜谱只有在 WildFly 以域模式运行时才有意义。

## 准备工作

记住，我正在远程运行 WildFly，绑定到 IP `192.168.59.103`。WildFly 已经启动并运行。

## 如何操作...

我们将分别介绍 `stop`、`start`、`restart` 和 `reload` 命令，以更好地解释它们的调用和最终差异。

### 停止

打开一个新的终端窗口并执行以下代码：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/server-group=main-server-group:stop-servers()"
{
    "outcome" => "success",
    "result" => undefined,
    "server-groups" => undefined
}
```

### 开始

打开一个新的终端窗口并运行以下命令：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/server-group=main-server-group:start-servers()"
{
    "outcome" => "success",
    "result" => undefined,
    "server-groups" => undefined
}
```

### 重启

打开一个新的终端窗口并执行以下命令：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/server-group=main-server-group:restart-servers()"
{
    "outcome" => "success",
    "result" => undefined,
    "server-groups" => undefined
}
```

### 重新加载

打开一个新的终端窗口并运行以下代码：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/server-group=main-server-group:reload-servers()"
{
    "outcome" => "success",
    "result" => undefined,
    "server-groups" => undefined
}
```

## 它是如何工作的...

在这里，我们正在告诉 `jboss-cli.sh` 脚本执行我们在 `--command` 参数中定义的命令，并将结果返回到标准输出。

使用 CLI，操作如下：

```java
$ ./bin/jboss-cli.sh
You are disconnected at the moment. Type 'connect' to connect to the server or 'help' for the list of supported commands.
[disconnected /] connect 192.168.59.103:9990
Authenticating against security realm: ManagementRealm
Username: wildfly
Password:
[standalone@192.168.59.103:9990 /] /server-group=main-server-group:stop-servers()
{
    "outcome" => "success",
    "result" => undefined,
    "server-groups" => undefined
}
[standalone@192.168.59.103:9990 /]
```

同样的，其他方法也适用：`start-servers()`、`restart-servers()` 和 `reload-servers()`。

## 更多内容...

WildFly 提供了一个额外的 API，即 HTTP API。让我们用一些网络命令行工具，如 `curl` 来尝试它。

### Curl

在一个运行的 WildFly 实例中，打开一个新的终端窗口并执行以下代码：

```java
$ curl --digest --user wildfly:cookbook.2015 http://192.168.59.103:9990/management/ -X POST -H "Content-Type: application/json" -d '{"operation":"restart-servers","address":[{"server-group":"main-server-group"}]}'
{"outcome" : "success", "result" : null, "server-groups" : null}
```

就这样！

# 创建服务器组

在这个菜谱中，你将学习如何通过向 CLI 发送命令来创建服务器组。这仅适用于域模式。

## 准备工作

记住，我正在远程运行 WildFly，绑定到 IP `192.168.59.103`。WildFly 已经启动并运行。

## 如何操作...

打开一个新的终端窗口并运行以下代码：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/server-group=next-server-group:add(profile=ha,socket-binding-group=ha-sockets)"
{
    "outcome" => "success",
    "result" => undefined,
    "server-groups" => undefined
}
```

### 注意

服务器组已准备好使用，无需重新加载或重启。这意味着可以将其用于添加服务器。目前，它只是一个空的服务器组。在本章的后面部分，我们将看到如何使用 CLI 将服务器添加到服务器组。

如果需要删除服务器组，只需使用其上的 `remove` 方法，如下所示：

```java
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/server-group=next-server-group:remove()"
{
    "outcome" => "success",
    "result" => undefined,
    "server-groups" => undefined
}
```

## 它是如何工作的...

在这里，我们正在告诉 `jboss-cli.sh` 脚本执行我们在 `--command` 参数中定义的命令，并将结果返回到标准输出。

使用 CLI，操作如下：

```java
$ ./bin/jboss-cli.sh
You are disconnected at the moment. Type 'connect' to connect to the server or 'help' for the list of supported commands.
[disconnected /] connect 192.168.59.103:9990
Authenticating against security realm: ManagementRealm
Username: wildfly
Password:
[domain@192.168.59.103:9990 /] /server-group=next-server-group:add(profile=ha,socket-binding-group=ha-sockets)
{
    "outcome" => "success",
    "result" => undefined,
    "server-groups" => undefined
}
[domain@192.168.59.103:9990 /]
```

## 更多内容...

WildFly 提供了一个额外的 API，即 HTTP API。让我们用一些网络命令行工具，如 `curl` 来尝试它。

在一个运行的 WildFly 实例中，打开一个新的终端窗口并运行以下代码：

```java
$ curl --digest --user wildfly:cookbook.2015 http://192.168.59.103:9990/management/ -X POST -H "Content-Type: application/json" -d '{"operation" : "composite", "address" : [], "steps" : [{"operation" : "add", "address" : {"server-group" : "next-server-group"}, "profile" : "ha", "socket-binding-group" : "ha-sockets"}]}'
{"outcome" : "success", "result" : {"step-1" : {"outcome" : "success"}}, "server-groups" : null}
```

# 创建服务器

在这个菜谱中，我们将学习如何通过向 CLI 发送命令来创建服务器。这仅适用于域模式。

## 准备工作

记住，我正在远程运行 WildFly，绑定到 IP `192.168.59.103`。WildFly 已经启动并运行。

## 如何操作...

打开一个新的终端窗口并执行以下命令：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/host=master/server-config=server-four:add(group=main-server-group, auto-start=true, socket-binding-port-offset=450)"
{
    "outcome" => "success",
    "result" => undefined,
    "server-groups" => undefined
}
```

现在服务器已准备就绪，你只需手动启动它。这可以通过启动你的新服务器所属的服务器组的服务来实现，在这种情况下，`main-server-group`。运行以下代码：

```java
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command=/server-group=main-server-group:start-servers()
{
    "outcome" => "success",
    "result" => undefined,
    "server-groups" => undefined
}
```

上述命令只是启动了处于 `STOPPED` 状态的服务器。

## 它是如何工作的...

这里，我们正在告诉 `jboss-cli.sh` 脚本执行我们在 `--command` 参数中定义的命令，并将结果输出到标准输出。

使用 CLI，它将是这样的：

```java
$ ./bin/jboss-cli.sh
You are disconnected at the moment. Type 'connect' to connect to the server or 'help' for the list of supported commands.
[disconnected /] connect 192.168.59.103:9990
Authenticating against security realm: ManagementRealm
Username: wildfly
Password:
[standalone@192.168.59.103:9990 /] /host=master/server-config=server-four:add(group=main-server-group, auto-start=true, socket-binding-port-offset=450)
{
    "outcome" => "success",
    "result" => undefined,
    "server-groups" => undefined
}
[standalone@192.168.59.103:9990 /]
```

## 更多内容...

WildFly 提供了一个额外的 API，即 HTTP API。让我们用一些网络命令行工具，如 `curl` 来试试。

### cURL

在一个正在运行的 WildFly 实例中，打开一个新的终端窗口并执行以下命令：

```java
$ curl --digest --user wildfly:cookbook.2015 http://192.168.59.103:9990/management/ -X POST -H "Content-Type: application/json" -d '{"operation" : "composite", "address" : [], "steps" : [{"operation" : "add", "address" : [{"host" : "master"},{"server-config" : "server-four"}], "group" : "main-server-group", "auto-start" : "true", "socket-binding-port-offset" : "450"}]}'
{"outcome" : "success", "result" : {"step-1" : {"outcome" : "success"}}, "server-groups" : null}
```

# 管理应用程序 – 部署、卸载

在这个食谱中，我们将学习如何在运行中的 WildFly 实例上通过向 CLI 发送命令来部署和卸载应用程序，并检查其状态。

## 准备工作

记住，我正在远程运行 WildFly，绑定到 IP `192.168.59.103`。WildFly 已经启动并运行。

对于这个食谱，我们需要一个名为 `example` 的应用程序，你可以在我的 GitHub 仓库中找到它。如果你跳过了第二章中关于“使用部署文件夹管理应用程序”的食谱，请参阅它以下载你需要的所有源代码和项目。

要构建应用程序，请给出以下命令：

```java
$ cd ~/WFC/github/wildfly-cookbook
$ cd example
$ mvn -e clean package
```

完成后，将工件 `example.war` 复制到你的本地 `$WILDFLY_HOME` 文件夹。

## 如何做到这一点...

我们将分别介绍 `deploy`、`status` 和 `undeploy` 命令，以更好地解释它们在处理独立模式和域模式时的区别。

### 部署

打开一个新的终端窗口并运行以下命令：

```java
$ cd $WILDFLY_HOME
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="deploy example.war"
```

如果你以域模式运行 WildFly，调用方式如下：

```java
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="deploy example.war --server-groups=main-server-group"
```

### 状态

如果你需要检查部署，可以执行以下代码：

```java
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/deployment=example.war:read-attribute(name=status)"
{
    "outcome" => "success",
    "result" => "OK"
}
```

如果你以域模式运行 WildFly，我们首先需要知道应用程序所属的服务器组。我们可以通过以下命令来实现：

```java
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="deployment-info --name=example.war"
NAME        RUNTIME-NAME
example.war example.war

SERVER-GROUP       STATE
main-server-group  enabled
other-server-group not added
```

如果你已经知道服务器组，你可以直接通过调用以下命令来检查部署状态，以查看它所属的服务器：

```java
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="/host=master/server=server-one/deployment=example.war:read-attribute(name=status)"
{
    "outcome" => "success",
    "result" => "OK"
}
```

### 卸载

如果你需要卸载你的应用程序，只需按下以下命令：

```java
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="undeploy example.war"
```

如果你以域模式运行 WildFly，调用方式如下：

```java
$ ./bin/jboss-cli.sh -c --controller=192.168.59.103:9990 --user=wildfly --password=cookbook.2015 --command="undeploy example.war --all-relevant-server-groups"
```

就这样！

## 它是如何工作的...

这里，我们正在告诉 `jboss-cli.sh` 脚本执行我们在 `--command` 参数中定义的命令，并将结果输出到标准输出。

### 部署

按照以下方式使用 CLI：

```java
$ ./bin/jboss-cli.sh
You are disconnected at the moment. Type 'connect' to connect to the server or 'help' for the list of supported commands.
[disconnected /] connect 192.168.59.103:9990
Authenticating against security realm: ManagementRealm
Username: wildfly
Password:
[standalone@192.168.59.103:9990 /] deploy example.war
[standalone@192.168.59.103:9990 /]
```

在域模式下，你将拥有以下命令：

```java
[domain@192.168.59.103:9990 /] deploy example.war --server-groups=main-server-group
[domain@192.168.59.103:9990 /]
```

顺便说一下，你不能操作输出。

### 状态

按照以下方式使用 CLI：

```java
$ ./bin/jboss-cli.sh
You are disconnected at the moment. Type 'connect' to connect to the server or 'help' for the list of supported commands.
[disconnected /] connect 192.168.59.103:9990
Authenticating against security realm: ManagementRealm
Username: wildfly
Password:
[standalone@192.168.59.103:9990 /] deployment-info example.war
{
    "outcome" => "success",
    "result" => "OK"
}
[standalone@192.168.59.103:9990 /]
Within the domain mode, you will have the following command:
[domain@192.168.59.103:9990 /] /host=master/server=server-one/deployment=example.war:read-attribute(name=status)
{
    "outcome" => "success",
    "result" => "OK"
}
[domain@192.168.59.103:9990 /]
```

### 卸载

按照以下方式使用 CLI：

```java
$ ./bin/jboss-cli.sh
You are disconnected at the moment. Type 'connect' to connect to the server or 'help' for the list of supported commands.
[disconnected /] connect 192.168.59.103:9990
Authenticating against security realm: ManagementRealm
Username: wildfly
Password:
[standalone@192.168.59.103:9990 /] undeploy example.war
[standalone@192.168.59.103:9990 /]
```

在域模式下，请执行以下命令：

```java
[domain@192.168.59.103:9990 /] undeploy example.war --all-relevant-server-groups
[domain@192.168.59.103:9990 /]
```

顺便说一下，你无法操作输出。

## 还有更多……

WildFly 提供了一个额外的 API，即 HTTP API。让我们用一些网络命令行工具，如 `curl` 来试试。

### Curl 部署

让我们使用 `curl` 进行部署：

1.  在运行 WildFly 实例的情况下，打开一个新的终端窗口并执行以下命令：

    ```java
    $ curl --digest --user wildfly:cookbook.2015 -F "file=@example.war" http://192.168.59.103:9990/management/add-content

    {"outcome" : "success", "result" : { "BYTES_VALUE" : "eqfGfLVOCv+p1gz5gjDgwX79ERk=" }}
    ```

    ### 注意

    文件 `example.war` 应该放在你调用命令的同一文件夹中。

1.  好的，它工作了。但我们只是上传了文件，还没有部署。要部署，我们需要获取 `upload` 命令产生的哈希码，并在下一个命令中使用它，如下所示：

    ```java
    $ curl --digest --user wildfly:cookbook.2015 http://192.168.59.103:9990/management -X POST -H "Content-Type: application/json" -d '{"content":[{"hash": {"BYTES_VALUE" : "eqfGfLVOCv+p1gz5gjDgwX79ERk="}}], "address": [{"deployment":"example.war"}], "operation":"add", "enabled":"true"}'

    {"outcome" : "success"}
    ```

    看这里！我们已经成功部署了我们的星际网络应用程序。

1.  如果你以域模式运行 WildFly，上传方法相同；因此，为了有效地部署应用程序，你可以执行以下命令：

    ```java
    $ curl  --digest --user wildfly:cookbook.2015 http://192.168.59.103:9990/management -X POST -H "Content-Type: application/json" -d '{"content":[{"hash": {"BYTES_VALUE" : "eqfGfLVOCv+p1gz5gjDgwX79ERk="}}], "address": [{"deployment":"example.war"}], "server-groups":"main-server-group", "runtime-name":"example.war", "operation":"add", "enabled":"true"}'
    {"outcome" : "success", "result" : null, "server-groups" : null}
    ```

就这样了！

### 状态

要检查部署状态，请执行以下命令：

```java
$ curl --digest --user wildfly:cookbook.2015 http://192.168.59.103:9990/management -X POST -H "Content-Type: application/json" -d '{"address": [{"deployment":"example.war"}],"operation":"read-attribute","name":"status"}'
{"outcome" : "success", "result" : "OK"}
```

如果你以域模式运行 WildFly，你可以运行以下代码：

```java
$ curl --digest --user wildfly:cookbook.2015 http://192.168.59.103:9990/management -X POST -H "Content-Type: application/json" -d '{"operation" : "read-resource", "address" : [{"deployment":"example.war"}], "json.pretty":1}'
{
    "outcome" : "success",
    "result" : {
        "content" : [{"hash" : {
            "BYTES_VALUE" : "eqfGfLVOCv+p1gz5gjDgwX79ERk="
        }}],
        "name" : "example.war",
        "runtime-name" : "example.war"
    }
}
```

就这些了。

### Curl 卸载

让我们使用 `curl` 进行卸载：

```java
$ curl --digest --user wildfly:cookbook.2015 http://192.168.59.103:9990/management/ -X POST -H "Content-Type: application/json" -d '{"address" : [{"deployment":"example.war"}],"operation":"undeploy"}'
{"outcome" : "success"}
```

就这样了！
