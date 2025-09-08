# 在主要 Java EE 服务器上部署和管理应用程序

本章涵盖了以下配方：

+   Apache TomEE 使用

+   GlassFish 使用

+   WildFly 使用

# 简介

作为 Java EE 开发者，您应该具备的最重要技能之一是了解如何与市场上最常用的 Java EE 应用服务器一起工作。

正如我们在前面的章节中所提到的，Java EE 生态系统中的标准允许您无论使用哪个服务器，都可以重用您已经拥有的大部分知识。

然而，当我们谈论部署和一些管理任务时，事情可能会有所不同（通常都是）。这些差异不在于它们的工作方式，而在于它们执行的方式。

因此，在本章中，我们将介绍 Apache TomEE、GlassFish 和 WildFly 的一些重要和常见任务。

# Apache TomEE 使用

如果您已经使用过 Apache Tomcat，您可以考虑自己准备好使用 Apache TomEE。它基于 Tomcat 的核心并实现了 Java EE 规范。

# 准备工作

首先，您需要将其下载到您的环境中。在撰写本文时，TomEE 没有与 Java EE 8 兼容的版本（实际上，只有 GlassFish 5）。然而，这里涵盖的任务在未来的版本中不应该改变，因为它们与 Java EE 规范无关。

要下载它，只需访问 [`tomee.apache.org/downloads.html`](http://tomee.apache.org/downloads.html)。此配方基于 7.0.2 Plume 版本。

在可能的情况下，我们将专注于使用配置文件执行任务。

# 如何操作...

参考以下详细任务。

# 部署 EAR、WAR 和 JAR 文件

对于 EAR 和 WAR 文件，部署文件夹是：

`$TOMEE_HOME/webapps`

对于 JAR 文件，文件夹是：

`$TOMEE_HOME/lib`

# 创建数据源和连接池

为了创建数据源和连接池以帮助您在项目中使用数据库，请编辑 `$TOMEE_HOME/conf/tomee.xml` 文件。

在 `<tomee>` 节点内部，插入如下子节点：

```java
  <Resource id="MyDataSouceDs" type="javax.sql.DataSource">
      jdbcDriver = org.postgresql.Driver
      jdbcUrl = jdbc:postgresql://[host]:[port]/[database]
      jtaManaged = true
      maxActive = 20
      maxIdle = 20
      minIdle = 0
      userName = user
      password = password
  </Resource>
```

示例针对的是 PostgreSQL，因此您需要对其他数据库进行一些更改。当然，您还需要根据您的需求更改其他参数。

# 日志设置和轮转

要配置 Apache TomEE 的日志记录，请编辑 `$TOMEE_HOME/conf/logging.properties` 文件。该文件使用类似以下的处理程序：

```java
1catalina.org.apache.juli.AsyncFileHandler.level = FINE
1catalina.org.apache.juli.AsyncFileHandler.directory = ${catalina.base}/logs
1catalina.org.apache.juli.AsyncFileHandler.prefix = catalina.
```

因此，您可以根据需要定义日志级别、目录和前缀。

如果您需要配置日志轮转，只需将这些行添加到您的处理程序中：

```java
1catalina.org.apache.juli.AsyncFileHandler.limit = 1024
1catalina.org.apache.juli.AsyncFileHandler.count = 10
```

在这个例子中，我们定义此文件在每 1024 千字节（1 MB）时旋转，并保留我们磁盘上的最后 10 个文件。

# 启动和停止

要启动 Apache TomEE，只需执行此脚本：

`$TOMEE_HOME/bin/startup.sh`

要停止它，执行以下脚本：

`$TOMEE_HOME/bin/shutdown.sh`

# 会话集群

如果您想使用 Apache TomEE 节点构建集群，您需要编辑 `$TOMEE_HOME/conf/server.xml` 文件。然后，找到以下行：

```java
<Engine name="Catalina" defaultHost="localhost">
```

插入一个子节点，如下所示：

```java
        <Cluster 
         className="org.apache.catalina.ha.tcp.SimpleTcpCluster"
                       channelSendOptions="8">

                <Manager 
                className="org.apache.catalina.ha.session
                         .DeltaManager"
                         expireSessionsOnShutdown="false"
                         notifyListenersOnReplication="true"/>

                <Channel 
                 className="org.apache.catalina.tribes.group
                 .GroupChannel">
                  <Membership                     
                  className="org.apache.catalina
                 .tribes.membership .McastService"
                              address="228.0.0.4"
                              port="45564"
                              frequency="500"
                              dropTime="3000"/>
                  <Receiver className="org.apache.catalina.tribes
                  .transport.nio.NioReceiver"
                            address="auto"
                            port="4000"
                            autoBind="100"
                            selectorTimeout="5000"
                            maxThreads="6"/>

                  <Sender className="org.apache.catalina.tribes
                  .transport.ReplicationTransmitter">
                    <Transport className="org.apache.catalina.tribes
                   .transport.nio.PooledParallelSender"/>
                  </Sender>
                  <Interceptor className="org.apache.catalina.tribes
                 .group.interceptors.TcpFailureDetector"/>
                  <Interceptor className="org.apache.catalina.tribes
                  .group.interceptors.MessageDispatchInterceptor"/>
                </Channel>

                <Valve className="org.apache.catalina
                .ha.tcp.ReplicationValve"
                       filter=""/>
                <Valve className="org.apache.catalina
                .ha.session.JvmRouteBinderValve"/>

                <Deployer className="org.apache.catalina
                .ha.deploy.FarmWarDeployer"
                          tempDir="/tmp/war-temp/"
                          deployDir="/tmp/war-deploy/"
                          watchDir="/tmp/war-listen/"
                          watchEnabled="false"/>

                <ClusterListener className="org.apache.catalina
                .ha.session.ClusterSessionListener"/>
        </Cluster>
```

此块将设置服务器以在动态发现集群中运行。这意味着在相同网络中使用此配置运行的每个服务器都将成为集群的新成员，并共享活动会话。

所有这些参数都极其重要，所以我真心建议您保留所有这些参数，除非您绝对确定自己在做什么。

# 还有更多...

目前设置 Java EE 集群的最佳方式是使用容器（特别是 Docker 容器）。因此，我建议您查看第十一章，*迈向云端 – Java EE、容器和云计算*。如果您将本章内容与该章节内容相结合，将为您的应用程序提供一个强大的环境。

要允许您的应用程序与集群中的所有节点共享会话，您需要编辑`web.xml`文件，找到`web-app`节点，并插入以下内容：

```java
<distributable/>
```

如果没有它，您的会话集群将无法工作。您还需要保持会话中持有的所有对象为可序列化。

# 参考信息

+   更多关于 Apache TomEE 的信息，请访问[`tomee.apache.org/`](http://tomee.apache.org/)

# GlassFish 使用

GlassFish 的伟大之处在于它是**参考实现**（**RI**）。因此，每当有新的 Java EE 版本发布时，作为开发者，您已经拥有了相应的 GlassFish 版本来尝试它。

# 准备工作

首先，您需要将其下载到您的环境中。在撰写本文时，GlassFish 5 是唯一已发布的 Java EE 8 服务器。

要下载它，只需访问[`javaee.github.io/glassfish/download`](https://javaee.github.io/glassfish/download)。本食谱基于版本 5（Java EE 8 兼容）。

在可能的情况下，我们将专注于使用配置文件执行任务。

# 如何操作...

参考以下详细任务。

# 部署 EAR、WAR 和 JAR 文件

对于 EAR 和 WAR 文件，部署文件夹是：

`$GLASSFISH_HOME/glassfish/domains/[domain_name]/autodeploy`

通常`domain_name`是`domain1`，除非您在安装过程中更改了它。

对于 JAR 文件，文件夹是：

`$GLASSFISH_HOME/glassfish/lib`

# 创建数据源和连接池

要创建数据源和连接池以帮助您在项目中使用数据库，请编辑`$GLASSFISH_HOME/glassfish/domains/[domain_name]/config/domain.xml`文件。在`<resources>`节点内，插入如下子节点：

```java
<jdbc-connection-pool 
  pool-resize-quantity="4" 
  max-pool-size="64" 
  max-wait-time-in-millis="120000" 
  driver-classname="com.mysql.jdbc.Driver" 
  datasource-classname="com.mysql.jdbc.jdbc2.optional
  .MysqlDataSource" 
  steady-pool-size="16" 
  name="MysqlPool" 
  idle-timeout-in-seconds="600" 
  res-type="javax.sql.DataSource">
      <property name="databaseName" value="database"></property>
      <property name="serverName" value="[host]"></property>
      <property name="user" value="user"></property>
      <property name="password" value="password"></property>
      <property name="portNumber" value="3306"></property>
</jdbc-connection-pool>
<jdbc-resource pool-name="MySqlDs" jndi-name="jdbc/MySqlDs">
</jdbc-resource>
```

然后，查找以下节点：

```java
<server config-ref="server-config" name="server">
```

向其中添加以下子节点：

```java
<resource-ref ref="jdbc/MySqlDs"></resource-ref>
```

该示例针对 MySQL，因此您需要对其他数据库进行一些更改。当然，您还需要根据需要更改其他参数。

# 日志设置和轮换

要为 GlassFish 配置日志记录，请编辑`$GLASSFISH_HOME/glassfish/domains/domain1/config/logging.properties`文件：

该文件与以下处理程序一起工作：

```java
handlers=java.util.logging.ConsoleHandler
handlerServices=com.sun.enterprise.server.logging.GFFileHandler
java.util.logging.ConsoleHandler.formatter=com.sun.enterprise.server.logging.UniformLogFormatter
com.sun.enterprise.server.logging.GFFileHandler.formatter=com.sun.enterprise.server.logging.ODLLogFormatter
com.sun.enterprise.server.logging.GFFileHandler.file=${com.sun.aas.instanceRoot}/logs/server.log
com.sun.enterprise.server.logging.GFFileHandler.rotationTimelimitInMinutes=0
com.sun.enterprise.server.logging.GFFileHandler.flushFrequency=1
java.util.logging.FileHandler.limit=50000
com.sun.enterprise.server.logging.GFFileHandler.logtoConsole=false
com.sun.enterprise.server.logging.GFFileHandler.rotationLimitInBytes=2000000
com.sun.enterprise.server.logging.GFFileHandler.excludeFields=
com.sun.enterprise.server.logging.GFFileHandler.multiLineMode=true
com.sun.enterprise.server.logging.SyslogHandler.useSystemLogging=false
java.util.logging.FileHandler.count=1
com.sun.enterprise.server.logging.GFFileHandler.retainErrorsStasticsForHours=0
log4j.logger.org.hibernate.validator.util.Version=warn
com.sun.enterprise.server.logging.GFFileHandler.maxHistoryFiles=0
com.sun.enterprise.server.logging.GFFileHandler.rotationOnDateChange=false
java.util.logging.FileHandler.pattern=%h/java%u.log
java.util.logging.FileHandler.formatter=java.util.logging.XMLFormatter
```

因此，您可以根据需要定义日志级别、目录、格式等。

如果你需要配置日志轮换，你必须关注这些行：

```java
com.sun.enterprise.server.logging.GFFileHandler
.rotationTimelimitInMinutes=0
com.sun.enterprise.server.logging.GFFileHandler
.rotationLimitInBytes=2000000
com.sun.enterprise.server.logging.GFFileHandler
.maxHistoryFiles=0
com.sun.enterprise.server.logging.GFFileHandler
.rotationOnDateChange=false
```

在这个例子中，我们定义这个文件在每 2000 千字节（2 MB）时旋转，并且不会在日期更改时旋转。历史文件没有限制。

# 启动和停止

要启动 GlassFish，只需执行以下脚本：

`$GLASSFISH_HOME/bin/asadmin start-domain --verbose`

要停止它，执行以下脚本：

`$GLASSFISH_HOME/bin/asadmin stop-domain`

# 会话集群

使用 GlassFish 构建集群有点棘手，需要同时使用命令行和管理面板，但这是完全可以做到的！让我们来看看。

首先，你需要两个或更多实例（称为节点）启动并运行。你可以以任何你喜欢的方式做到这一点——每个在不同的机器上运行，使用虚拟机或容器（我最喜欢的选项）。无论你选择哪种方式，启动集群的方式都是相同的：

1.  因此，获取你的第一个节点并打开其管理面板：

`https://[hostname]:4848`

1.  点击左侧菜单中的“集群”选项，然后点击“新建”按钮。将集群命名为`myCluster`并点击“确定”按钮。

1.  从列表中选择你的集群。在打开的页面中，选择选项卡中的“实例”选项，然后点击“新建”。将实例命名为`node1`并点击“确定”。

1.  现在，转到“常规”选项卡，然后点击“启动集群”按钮。哇！你的集群已经启动并运行，并且已经有了第一个节点。

1.  现在，转到已经安装了 GlassFish 的第二台机器（虚拟机、容器、其他服务器或任何服务器）并运行此命令：

```java
$GLASSFISH_HOME/bin/asadmin --host [hostname_node1] --port 4848 create-local-instance --cluster myCluster node2
```

这将把第二台机器设置为集群的成员。如果你在第一台机器上刷新集群页面，你会看到新的成员（`node2`）。

1.  你会注意到`node2`已经停止了。点击它，然后在新的页面中点击 Node 链接（它通常会显示`node2`的主机名）。

1.  在打开的页面中，将“类型”值更改为`SSH`。在`SSH`部分将显示一些新的字段。

1.  将 SSH 用户认证改为`SSH 密码`，并在 SSH 用户密码字段中填写正确的密码。

1.  点击“保存”按钮。如果你遇到一些 SSH 错误（通常是`连接被拒绝`），将“强制”选项设置为启用，然后再次点击“保存”按钮。

1.  返回到托管`node2`的机器上的命令行并运行此命令：

```java
$GLASSFISH_HOME/glassfish/lib/nadmin start-local-instance --node [node2_hostname] --sync normal node2
```

如果一切顺利，你的`node2`应该已经启动并运行，你现在应该有一个真正的集群。你可以根据需要重复这些步骤来添加新的节点到你的集群。

# 还有更多...

对于使用 GlassFish 的这种集群，一个常见的问题是你没有在节点上运行 SSH 服务；由于许多操作系统有大量的选项，我们在这里不会涵盖每一个。

目前设置 Java EE 集群的最佳方式是使用容器（特别是 Docker 容器）。因此，我建议您查看第十一章 Rising to the Cloud – Java EE, Containers, and Cloud Computing，这样，如果您将这部分内容与当前内容结合，将为您的应用程序提供一个强大的环境。

要允许您的应用程序与集群中的所有节点共享会话，您需要编辑 `web.xml` 文件，找到 `web-app` 节点，并插入以下内容：

```java
<distributable/>
```

没有它，您的会话集群将无法工作。您还需要确保所有在会话中持有的对象都是可序列化的。

最后，还有商业版本的 GlassFish，即 Payara Server。如果您正在寻找支持和其他商业优惠，您应该看看它。

# 参见

+   想了解更多关于 GlassFish 的信息，请访问 [`javaee.github.io/glassfish/`](https://javaee.github.io/glassfish/)。

# WildFly 使用

WildFly 是另一个优秀的 Java EE 实现。它曾被称为 **JBoss AS**，但几年前更改了名称（尽管我们仍然有 JBoss EAP 作为 *企业生产就绪* 版本）。由于其管理和使用方式与 Apache TomEE 和 GlassFish 略有不同，因此仔细研究它是值得的。

# 准备工作

首先，您需要将其下载到您的环境中。在撰写本文时，WildFly 没有与 Java EE 8 兼容的版本（实际上只有 GlassFish 5）。然而，这里涵盖的任务在未来版本中不应发生变化，因为它们与 Java EE 规范无关。要下载它，只需访问 [`wildfly.org/downloads/`](http://wildfly.org/downloads/)。

此示例基于版本 11.0.0.Final（Java EE7 全功能和 Web 发行版）。

在可能的情况下，我们将专注于使用配置文件来完成这些任务。

# 如何操作...

参考以下详细任务。

# 部署 EAR、WAR 和 JAR 文件

对于 EAR 和 WAR 文件，部署文件夹是：

`$WILDFLY_HOME/standalone/deployments`

对于 JAR 文件（例如 JDBC 连接），WildFly 创建了一个灵活的文件夹结构。因此，最佳的分发方式是使用其 UI，正如我们将在连接池主题（下一个主题）中展示的那样。

# 创建数据源和连接池

按以下步骤创建您的数据源和连接池：

1.  要创建数据源和连接池以帮助您在项目中使用数据库，启动 WildFly 并访问以下 URL：

`http://localhost:9990/`

1.  点击“部署”然后点击“添加”按钮。在打开的页面中，选择“上传新的部署”并点击“下一步”按钮。在打开的页面中，选择适当的 JDBC 连接器（我们将使用 MySQL 进行此示例）并点击“下一步”。

1.  验证打开页面中的信息并点击“完成”。

1.  现在您的 JDBC 连接器已在服务器中可用，您可以继续创建数据源。转到管理面板中的“首页”并点击“配置”选项。

1.  在打开的页面中，按照以下路径：

子系统 | 数据源 | 非 XA | 数据源 | 添加

1.  在打开的窗口中选择 MySQL 数据源，然后点击下一步。然后填写字段如下：

+   名称: `MySqlDS`

+   JNDI 名称: `java:/MySqlDS`

1.  点击下一步。在下一页，点击检测到的驱动程序，选择适当的 MySQL 连接器（你刚刚上传的那个），然后点击下一步。

1.  最后一步是填写连接设置字段，如下所示：

+   连接 URL: `jdbc:mysql://localhost:3306/sys`

+   用户名: `root`

+   密码: `mysql`

1.  点击下一步按钮，查看信息，然后点击完成。你新创建的连接将出现在数据源列表中。你可以点击下拉列表并选择测试来检查一切是否正常工作。

# 日志设置和轮换

要配置 WildFly 的日志，编辑 `$WILDFLY_HOME/standalone/configuration/standalone.xml` 文件。

要自定义日志属性，找到 `<profile>` 节点，然后在其中找到 `<subsystem xmlns='urn:jboss:domain:logging:3.0'>`。

它基于这样的处理程序：

```java
<console-handler name="CONSOLE">
                <level name="INFO"/>
                <formatter>
                    <named-formatter name="COLOR-PATTERN"/>
                </formatter>
            </console-handler>
            <periodic-rotating-file-handler name="FILE" 
             autoflush="true">
                <formatter>
                    <named-formatter name="PATTERN"/>
                </formatter>
                <file relative-to="jboss.server.log.dir" 
                 path="server.log"/>
                <suffix value=".yyyy-MM-dd"/>
                <append value="true"/>
            </periodic-rotating-file-handler>
            <logger category="com.arjuna">
                <level name="WARN"/>
            </logger>
            <logger category="org.jboss.as.config">
                <level name="DEBUG"/>
            </logger>
            <logger category="sun.rmi">
                <level name="WARN"/>
            </logger>
            <root-logger>
                <level name="INFO"/>
                <handlers>
                    <handler name="CONSOLE"/>
                    <handler name="FILE"/>
                </handlers>
            </root-logger>
            <formatter name="PATTERN">
                <pattern-formatter pattern=
                "%d{yyyy-MM-dd HH:mm:ss,SSS} %-5p [%c] (%t) %s%e%n"/>
            </formatter>
            <formatter name="COLOR-PATTERN">
                <pattern-formatter pattern="%K{level}%d
                {HH:mm:ss,SSS} %-5p [%c] (%t) %s%e%n"/>
            </formatter>
```

默认情况下，它将每天轮换：

```java
            <periodic-rotating-file-handler name="FILE" 
             autoflush="true">
                <formatter>
                    <named-formatter name="PATTERN"/>
                </formatter>
                <file relative-to="jboss.server.log.dir" 
                 path="server.log"/>
                <suffix value=".yyyy-MM-dd"/>
                <append value="true"/>
            </periodic-rotating-file-handler>
```

如果你想根据大小进行轮换，你应该移除前面的处理器，然后插入这个：

```java
            <size-rotating-file-handler name="FILE" autoflush="true">
                <formatter>
                    <pattern-formatter pattern="%d{yyyy-MM-dd 
                     HH:mm:ss,SSS} %-5p [%c] (%t) %s%e%n"/>
                </formatter>
                <file relative-to="jboss.server.log.dir" 
                path="server.log"/>
                <rotate-size value="2m"/>
                <max-backup-index value="5"/>
                <append value="true"/>
            </size-rotating-file-handler>
```

在这种情况下，当文件达到 2 MB 时，日志将进行轮换，并保留五个文件的备份历史。

# 启动和停止

要启动 GlassFish，只需执行此脚本：

`$WILDFLY_HOME/bin/standalone.sh`

要停止它，执行此脚本：

`$WILDFLY_HOME/bin/jboss-cli.sh --connect command=:shutdown`

# 会话集群

如果你转到 `$WILDFLY_HOME/standalone/configuration` 文件夹，你会看到这些文件：

+   `standalone.xml`

+   `standalone-ha.xml`

+   `standalone-full.xml`

+   `standalone-full-ha.xml`

`standalone.xml` 是默认配置，包含所有默认配置。要构建集群，我们需要使用 `standalone-ha.xml` 文件（`ha` 来自 **高可用性**），因此将其重命名为以 `standalone.xml` 结尾。

然后，启动服务器。你不应该做以下事情：

```java
$WILDFLY_HOME/bin/standalone.sh
```

相反，你应该这样做：

```java
$WILDFLY_HOME/bin/standalone.sh -b 0.0.0.0 -bmanagement 0.0.0.0
```

你现在应该在你想加入集群的任何其他节点（机器、虚拟机、容器等）上做同样的事情。当然，它们需要处于同一个网络中。

# 更多内容...

目前设置 Java EE 集群的最好方法是使用容器（特别是 Docker 容器）。因此，我建议你查看第十一章 Rising to the Cloud – Java EE, Containers, and Cloud Computing，了解如何将内容与本章内容结合，这将为你提供一个强大的应用程序环境。

要允许你的应用程序与集群中的所有节点共享其会话，你需要编辑 `web.xml` 文件，找到 `web-app` 节点，并插入以下内容：

```java
<distributable/>
```

没有它，你的会话集群将无法工作。

# 参见

+   想了解更多关于 WildFly 的信息，请访问 [`wildfly.org/`](http://wildfly.org/)
