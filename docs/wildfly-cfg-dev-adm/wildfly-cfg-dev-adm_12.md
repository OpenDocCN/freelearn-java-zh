# 附录 A. CLI 参考

为了保持简单，以下是最常用的命令和操作的快速参考，用于通过 CLI 管理应用程序服务器。为了简洁起见，仅提及 Linux 环境中的`jboss-cli.sh`脚本。Windows 用户只需将此文件替换为等效的`jboss-cli.bat`文件。

# 启动选项：

以下命令可用于以非交互方式启动 CLI：

+   将脚本命令传递给`jboss-cli` shell：

    ```java
    ./jboss-cli.sh --connect command=:shutdown

    ```

+   执行文件中的 CLI shell：

    ```java
    ./jboss-cli.sh --file=test.cli

    ```

# 通用命令

以下命令可用于收集系统信息并设置特定的服务器属性：

+   显示环境信息：

    ```java
    version

    ```

+   显示 JNDI 上下文：

    ```java
    /subsystem=naming:jndi-view

    ```

+   显示 XML 服务器配置：

    ```java
    :read-config-as-xml

    ```

+   显示容器中注册的服务及其状态：

    ```java
    /core-service=service-container:dump-services

    ```

+   设置系统属性：

    ```java
    /system-property=property1:add(value="value")

    ```

+   显示系统属性：

    ```java
    /system-property=property1:read-resource

    ```

+   显示所有系统属性：

    ```java
    /core-service=platform-mbean/type=runtime:read-attribute(name=system-properties)

    ```

+   删除系统属性：

    ```java
    /system-property=property1:remove

    ```

+   更改套接字绑定端口（例如，`http`端口）：

    ```java
    /socket-binding-group=standard-sockets/socket-binding=http:write-attribute(name="port", value="8090")

    ```

+   显示公共接口的 IP 地址：

    ```java
    /interface=public:read-attribute(name=resolved-address)

    ```

# 域模式命令

在主机名（以及如果需要，服务器名）前加上前缀以指示你正在向哪个主机（或服务器名）发出命令。示例：

+   显示主机 master 的 XML 配置：

    ```java
    /host=master:read-config-as-xml

    ```

+   显示运行在主机`master`上的`server-one`服务器的公共接口的 IP 地址：

    ```java
    /host=master/server=server-one/interface=public:read-attribute(name=resolved-address)

    ```

# 与应用程序部署相关的命令：

CLI 也可以用于部署应用程序。CLI 假定`MyApp.war`文件位于`jboss-cli`工作目录之外。以下是部署命令的快速参考：

+   已部署应用程序列表：

    ```java
    deploy

    ```

+   在独立服务器上部署应用程序：

    ```java
    deploy MyApp.war

    ```

+   在独立服务器上重新部署应用程序：

    ```java
    deploy -f MyApp.war

    ```

+   从所有服务器组卸载应用程序：

    ```java
    undeploy MyApp.war

    ```

+   在所有服务器组上部署应用程序：

    ```java
    deploy MyApp.war --all-server-groups

    ```

+   在一个或多个服务器组上部署应用程序（用逗号分隔）：

    ```java
    deploy application.ear --server-groups=main-server-group

    ```

+   从所有服务器组卸载应用程序：

    ```java
    undeploy application.ear --all-relevant-server-groups

    ```

+   从一个或多个服务器组卸载应用程序：

    ```java
    undeploy as7project.war --server-groups=main-server-group

    ```

+   不删除内容地卸载应用程序：

    ```java
    undeploy application.ear --server-groups=main-server-group --keep-content

    ```

# JMS

在这里，你可以找到可以用于创建/删除 JMS 目的地的 JMS 命令：

+   添加 JMS 队列：

    ```java
    jms-queue add –-queue-address=queue1 --entries=queues/queue1

    ```

+   删除 JMS 队列：

    ```java
    jms-queue remove --queue-address=queue1

    ```

+   添加 JMS 主题：

    ```java
    jms-topic add –-topic-address=topic1 --entries=topics/topic1

    ```

+   删除 JMS 主题：

    ```java
    jms-topic remove --topic-address=topic1

    ```

# 数据源

这是一个可以使用数据源别名发出的便捷数据源命令列表：

+   添加数据源：

    ```java
    data-source add --jndi-name=java:/MySqlDS --name=MySQLPool --connection-url=jdbc:mysql://localhost:3306/MyDB --driver-name=mysql-connector-java-5.1.16-bin.jar --user-name=myuser --password=password –max-pool-size=30

    ```

+   删除数据源：

    ```java
    data-source remove --name=java:/MySqlDS

    ```

## 数据源（使用资源操作）

您还可以使用数据源子系统上的操作对数据源进行操作：

+   列出已安装的驱动程序：

    ```java
    /subsystem=datasources:installed-drivers-list

    ```

+   添加数据源：

    ```java
    data-source add --jndi-name=java:/MySqlDS --name=MySQLPool --connection-url=jdbc:mysql://localhost:3306/MyDB --driver-name=mysql-connector-java-5.1.30-bin.jar --user-name=myuser --password=password --max-pool-size=30

    ```

+   使用操作添加 XA 数据源：

    ```java
    xa-data-source add --name=MySQLPoolXA --jndi-name=java:/MySqlDSXA --driver-name=mysql-connector-java-5.1.30-bin.jar -xa-datasource-properties=[{ServerName=localhost}{PortNumber=3306}]

    ```

+   使用操作删除数据源：

    ```java
    /subsystem=datasources/data-source=testDS:remove

    ```

# Mod_cluster

可以使用以下 CLI 操作执行 Mod_cluster 管理：

+   列出已连接的代理：

    ```java
    /subsystem=modcluster:list-proxies

    ```

+   显示代理信息：

    ```java
    /subsystem=modcluster:read-proxies-info

    ```

+   向集群添加代理：

    ```java
    /subsystem=modcluster:add-proxy(host= CP15-022, port=9999)

    ```

+   删除代理：

    ```java
    /subsystem=modcluster:remove-proxy(host=CP15-022, port=9999)

    ```

+   添加 Web 上下文：

    ```java
    /subsystem=modcluster:enable-context(context=/myapp, virtualhost=default-host)

    ```

+   禁用 Web 上下文：

    ```java
    /subsystem=modcluster:disable-context(context=/myapp, virtualhost=default-host)

    ```

+   停止 Web 上下文：

    ```java
    /subsystem=modcluster:stop-context(context=/myapp, virtualhost=default-host, waittime=50)

    ```

# 批处理：

这是使用 CLI 处理批处理的方法：

+   开始批处理：

    ```java
    batch

    ```

+   暂停批处理：

    ```java
    holdback-batch

    ```

+   暂停后继续批处理：

    ```java
    batch

    ```

+   当前批处理堆栈上的命令列表：

    ```java
    list-batch

    ```

+   清除命令的批处理会话：

    ```java
    clear-batch

    ```

+   在堆栈上执行批处理命令：

    ```java
    run-batch

    ```

# 快照

快照允许存储和检索服务器配置：

+   捕获配置快照：

    ```java
    :take-snapshot

    ```

+   列出可用的快照：

    ```java
    :list-snapshots

    ```

+   删除快照：

    ```java
    :delete-snapshot(name="20140814-234725965standalone-full-ha.xml")

    ```
