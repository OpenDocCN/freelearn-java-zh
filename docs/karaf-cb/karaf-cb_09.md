# 第九章：使用 Apache Hadoop 提供大数据集成层

在本章中，我们将涵盖以下主题：

+   在 Apache Karaf 中安装 Hadoop 客户端包

+   从 Karaf 访问 Apache Hadoop

+   添加与 HDFS 通信的命令以在 Karaf 中部署

# 引言

为了继续构建比传统 RDBMS 结构更灵活的数据存储模型，我们将探讨 Apache Hadoop。Hadoop 是由 Doug Cutting 和 Mike Cafarella 于 2005 年创建的。当时在 Yahoo!工作的 Cutting 以他儿子的玩具大象命名了它。它最初是为了支持 Nutch 搜索引擎项目的分布式而开发的。

Hadoop 遵循了 Google 在其关于 Google 文件系统和 Google MapReduce 的论文中发表的思想。经过十多年的使用，Hadoop 已经发展成为一个非常庞大且复杂的生态系统，预计 2016 年的收入约为 230 亿美元。Hadoop 推动了从重新包装的发行版到完整的数据库实现、分析包和管理解决方案的一切。

Hadoop 也开始改变初创公司对数据模型的认识，使新公司能够将大数据作为其整体战略的一部分。

在 Hadoop 的核心，你有**Hadoop 分布式文件系统**（**HDFS**）。这个机制是允许数据分布的。它从潜在的单一故障点场景发展到有来自 DataStax 和 RedHat 等公司的竞争性实现，分别是 Cassandra FS 和 RedHat Cluster File System。

在完整产品解决方案的领域，你可以找到 MapR、Cloudera、Hortonworks、Intel 和 IBM 等作为 Apache Hadoop 发行版的替代品。

如果你思考未来，似乎 SQL-like 技术与分布式存储的结合将是大多数用例的发展方向。这使用户至少能够利用已经在组合中尝试过的 RDBMS 思想，结合几乎无限的数据挖掘、社交网络、监控和决策制定的数据存储。

随着 YARN（集群资源管理）成为 Hadoop 基础设施的一部分，HDFS 不再仅仅是存储和 MapReduce，而是一个可以处理批量、交互式、流式以及应用部署的环境。

# 启动一个独立的 Hadoop 集群

为了开始这一过程，我们将首先设置一个简单的独立集群。我们需要下载和配置一个 Apache Hadoop 版本，并确保我们可以使用它。然后，我们将继续在 Apache Karaf 容器中配置相同的配置和访问方法。我们将利用外部集群来展示如何使用 Apache Karaf 启动一个针对大型现有集群的新作业引擎。凭借我们拥有的和将要部署的功能，您还可以从 Karaf 容器中嵌入 HDFS 文件系统。

从 Apache 的镜像之一下载 Hadoop 版本，请访问 [`hadoop.apache.org/releases.html#Download`](http://hadoop.apache.org/releases.html#Download)。在撰写本书时，最新版本是 2.4.0。设置集群的完整指南可以在 [`hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/SingleCluster.html`](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/SingleCluster.html) 找到。

以下是你需要做出的更改，以便让 HDFS 启动并能够与本地安装的节点、复制处理程序和作业跟踪器通信。

展开下载的存档，并修改 `etc/hadoop/*` 文件夹中的文件。`core-site.xml` 文件需要按照以下方式修改：

```java
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
  </property>
</configuration>
```

需要按照以下方式修改 `hdfs-site.xml` 文件：

```java
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
</configuration>
```

需要按照以下方式修改 `mapred-site.xml` 文件：

```java
<configuration>
  <property>
    <name>mapred.job.tracker</name>
    <value>localhost:9001</value>
  </property>
</configuration>
```

一旦完成，并且你可以无密码运行 SSH 到你的本地主机，你就可以启动你的守护进程。如果你不能运行 SSH，请运行以下命令：

```java
ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa 
cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys

```

之前的命令将创建一个用于远程登录的空 SSH 密钥。如果你已经有一个与你的账户关联的现有 SSH 密钥，你只需要运行第二个命令以确保你可以远程登录到你的本地主机。

现在，让我们验证现有的安装是否可访问，并且我们可以对其进行操作。一旦完成，我们可以使用以下命令一次性启动所有守护进程：

```java
./sbin/start-all.sh

```

### 注意

请记住，这个命令可能会随着你未来配置更多的 YARN 选项而消失。

你应该会看到几个守护进程启动，并且希望除了可能与你缺失的本地库相关的警告之外没有错误信息。（这在本小教程中没有涉及，但与 IO 库和针对你特定平台的优化访问有关。）

当我们有一个运行的 HDFS 系统时，我们首先会使用以下命令创建一个目录：

```java
 ./bin/hadoop fs -mkdir -p /karaf/cookbook

```

然后，我们使用以下命令验证我们是否可以读取：

```java
./bin/hadoop fs -ls /karaf
Found 1 items
drwxr-xr-x   - joed supergroup          0 2014-05-15 21:47 /karaf/cookbook

```

我们在我们启动的简单单节点集群中创建了一个目录（我们称它为集群，因为所有必要的集群组件都在运行，我们只是在单个节点上运行它们）。我们还确保我们可以列出该目录的内容。这告诉我们 HDFS 是可访问和活跃的。

以下是我们迄今为止所取得的成果以及我们将要介绍的内容。

+   我们现在有一个正在运行的 Karaf 外部 HDFS 系统。这可能是一个现有的部署、一个亚马逊作业或一组虚拟服务器。基本上，我们拥有了集群的基础设施，并且我们知道我们可以访问它并创建内容。

+   下一步将深入探讨将现有的部署模型转换为 OSGi 兼容的部署。

# 在 Apache Karaf 中安装 Hadoop 客户端包

随着 Hadoop 的运行，我们已准备好开始利用 Apache Karaf 的资源。

## 准备工作

本食谱的成分包括 Apache Karaf 发行套件、对 JDK 的访问权限和互联网连接。我们还假设已经下载并安装了 Apache Hadoop 发行版。可以从[`hadoop.apache.org/#Download`](http://hadoop.apache.org/#Download)下载。

## 如何做到这一点…

Hadoop 的 HDFS 库不是标准 Karaf 功能库的一部分；因此，我们或者需要编写自己的功能，或者手动安装客户端运行所需的必要包。Apache Camel 确实通过 Camel 的 HDFS2 组件提供了这一功能。我们可以使用 Camel 现有的功能，或者自己构建这个功能。

在 Camel 功能中使用的当前版本的 Snappy Java，使用 Java 7 运行原生库时会出现问题。这是一个已知的问题，并且正在 Snappy Java 的 1.0.5 版本中解决([`github.com/xerial/snappy-java`](https://github.com/xerial/snappy-java))。为了解决所有平台上的这个问题，我们将构建自己的 Karaf 功能，其中我们可以利用 Camel 功能中的所有包以及一些额外的 JAR 文件，这将使我们能够运行最新版本。这可以通过以下方式完成：

```java
<features name="com.packt.chapter.nine-${project.version}">

  <repository>mvn:org.apache.cxf.karaf/apache-cxf/3.0.0/xml/features</repository>

  <feature name='xml-specs-api' version='${project.version}' resolver='(obr)' start-level='10'>
    <bundle dependency='true'>mvn:org.apache.servicemix.specs/org.apache.servicemix.specs.activation-api-1.1/</bundle>
    <bundle dependency='true'>mvn:org.apache.servicemix.specs/org.apache.servicemix.specs.stax-api-1.0/</bundle>
    <bundle dependency='true'>mvn:org.apache.servicemix.specs/org.apache.servicemix.specs.jaxb-api-2.2/</bundle>
    <bundle>mvn:org.codehaus.woodstox/stax2-api/</bundle>
    <bundle>mvn:org.codehaus.woodstox/woodstox-core-asl/</bundle>
    <bundle>mvn:org.apache.servicemix.bundles/org.apache.servicemix.bundles.jaxb-impl/</bundle>
  </feature>

  <feature name='hdfs2' version='${project.version}' resolver='(obr)' start-level='50'>
    <feature>xml-specs-api</feature>
    <feature>cxf-jaxrs</feature>
    <bundle dependency='true'>mvn:commons-lang/commons-lang/2.6</bundle>
    <bundle dependency='true'>mvn:com.google.guava/guava/16.0.1</bundle>
    <bundle dependency='true'>mvn:com.google.protobuf/protobuf-java/</bundle>
    <bundle dependency='true'>mvn:org.apache.servicemix.bundles/org.apache.servicemix.bundles.guice/</bundle>
    <bundle dependency='true'>mvn:org.apache.servicemix.bundles/org.apache.servicemix.bundles.jsch/</bundle>
    <bundle dependency='true'>mvn:org.apache.servicemix.bundles/org.apache.servicemix.bundles.paranamer/</bundle>
    <bundle dependency='true'>mvn:org.apache.servicemix.bundles/org.apache.servicemix.bundles.avro/1.7.3_1</bundle>
    <bundle dependency='true'>mvn:org.apache.commons/commons-compress/</bundle>
    <bundle dependency='true'>mvn:org.apache.commons/commons-math3/3.1.1</bundle>
    <bundle dependency='true'>mvn:commons-cli/commons-cli/1.2</bundle>
    <bundle dependency='true'>mvn:commons-configuration/commons-configuration/</bundle>
    <bundle dependency='true'>mvn:org.apache.servicemix.bundles/org.apache.servicemix.bundles.commons-httpclient/</bundle>
    <bundle dependency='true'>mvn:io.netty/netty/3.9.2.Final</bundle>
    <bundle dependency='true'>mvn:org.codehaus.jackson/jackson-core-asl/1.9.12</bundle>
    <bundle dependency='true'>mvn:org.codehaus.jackson/jackson-mapper-asl/1.9.12</bundle>
    <bundle dependency="true">mvn:org.codehaus.jackson/jackson-jaxrs/1.9.12</bundle>
    <bundle dependency="true">mvn:org.codehaus.jackson/jackson-xc/1.9.12</bundle>
    <bundle dependency='true'>mvn:org.apache.servicemix.bundles/org.apache.servicemix.bundles.snappy-java</bundle>
    <bundle dependency='true'>mvn:commons-codec/commons-codec/</bundle>
    <bundle dependency='true'>mvn:commons-collections/commons-collections/3.2.1</bundle>
    <bundle dependency='true'>mvn:commons-io/commons-io/</bundle>
    <bundle dependency='true'>mvn:commons-net/commons-net/</bundle>
    <bundle dependency='true'>mvn:org.apache.zookeeper/zookeeper/</bundle>
    <bundle dependency='true'>mvn:org.apache.servicemix.bundles/org.apache.servicemix.bundles.xmlenc/0.52_1</bundle>
    <bundle dependency='true'>mvn:org.apache.servicemix.bundles/org.apache.servicemix.bundles.xerces/</bundle>
    <bundle dependency='true'>mvn:org.apache.servicemix.bundles/org.apache.servicemix.bundles.xmlresolver/</bundle>
    <bundle>mvn:org.apache.servicemix.bundles/org.apache.servicemix.bundles.hadoop-client/</bundle>
  </feature>

</features>
```

我们现在可以利用这个功能的文件来部署一个 OSGi 友好的 Hadoop 客户端，这将使我们能够利用 Hadoop 中的所有功能。我们通过利用另一个项目 Apache ServiceMix 来实现这一点。Apache ServiceMix 维护一个包仓库，其中常用的或寻求的资源被重新打包，并在必要时转换为工作的 OSGi 包。

我们正在使用的`Apache ServiceMix :: Bundles :: hadoop-client`包是一个包含核心、YARN、HDFS、MapReduce、通用客户端 JAR 文件和 Hadoop 注解的 uber 包。

我们可以通过执行以下`list | grep –i hadoop`命令来验证安装：

```java
karaf@root()> list | grep -i hadoop
151 | Active |  50 | 2.4.0.1     | Apache ServiceMix :: Bundles :: hadoop-client

```

# 从 Karaf 访问 Apache Hadoop

在 Hadoop 中，集群的核心是分布式和复制的文件系统。我们已经运行了 HDFS，并且可以从我们的命令行窗口以普通用户身份访问它。实际上，从 OSGi 容器访问它将比仅仅编写 Java 组件要复杂一些。

Hadoop 要求我们为我们的集群提供配置元数据，这些元数据可以作为文件或类路径资源进行查找。在本食谱中，我们将简单地复制我们在本章早期创建的 HDFS 特定文件到我们的`src/main/resources`文件夹中。

我们还将通过从依赖项中复制默认元数据定义，将它们包含到我们的资源中，最后，我们将允许我们的包类加载器执行完全动态的类加载。总结一下，我们必须将`core-site.xml`、`hdfs-site.xml`和`mapred-site.xml`文件复制到我们的类路径中。这些文件共同描述了我们的客户端如何访问 HDFS。

当我们到达代码时，我们还将执行一个步骤来欺骗 Hadoop 类加载器利用我们的包类加载器，并通过提供特定的实现来尊重配置数据。

## 如何做到这一点...

我们首先要确保必要的默认值被复制到我们的教程包中。

1.  首先，我们将修改 Felix 包插件并添加以下段：

    ```java
    <Include-Resource>
      {maven-resources},
      @org.apache.servicemix.bundles.hadoop-client-2.4.0_1.jar!/core-default.xml,
      @org.apache.servicemix.bundles.hadoop-client-2.4.0_1.jar!/hdfs-default.xml,
      @org.apache.servicemix.bundles.hadoop-client-2.4.0_1.jar!/mapred-default.xml,
      @org.apache.servicemix.bundles.hadoop-client-2.4.0_1.jar!/hadoop-metrics.properties
    </Include-Resource>
    ```

1.  接下来，我们将为按需动态加载类添加另一个部分。这并不一定是 OSGi 中的最佳实践，但有时，这是实现非 OSGi 设计意图的包和 JAR 文件工作的几种可能方法之一。我们通过向 Felix 包插件添加另一个小片段来实现这一点，如下所示：

    ```java
    <DynamicImport-Package>*</DynamicImport-Package>
    ```

    通过这个添加，我们告诉我们的包，如果您以后需要某个类，只需在我们需要时查找即可。这稍微有点成本，并且绝对不推荐作为一般做法，因为它迫使包扫描类路径。

1.  最后，我们在实现代码中使用了两个额外的技巧。我们这样做是因为 Hadoop 代码是多线程的，默认情况下，新线程的类加载器是系统类加载器。在 OSGi 环境中，系统类加载器将具有非常有限的可见性。

    1.  首先，我们按照以下方式替换`ThreadContextClassLoader`类：

        ```java
        ClassLoader tccl = Thread.currentThread().getContextClassLoader();
          try {
            Thread.currentThread().setContextClassLoader(getClass().getClassLoader());
            .
          } catch (IOException e) {
            e.printStackTrace();
          } finally {
            Thread.currentThread().setContextClassLoader(tccl);
          }
        ```

    1.  其次，由于我们正在使用 Maven，并且不幸的是，Hadoop 根据您的实现重复使用相同的文件用于 SPI，我们无法简单地复制资源。我们的聚合 JAR 文件以及我们导入的任何依赖项都将被最后导入的版本覆盖。

        为了解决这个问题，我们明确告诉我们的`Configuration`对象在访问我们的集群时应使用哪些实现。这在下述代码中显示：

        ```java
        Configuration conf = new Configuration();
          conf.set("fs.hdfs.impl", org.apache.hadoop.hdfs.DistributedFileSystem.class.getName());
          conf.set("fs.file.impl", org.apache.hadoop.fs.LocalFileSystem.class.getName());
        ```

通过前面的操作，我们已经满足了所有类加载和实例化问题。现在，我们实际上已经准备好远程访问 HDFS。然而，我们还没有强迫我们的包导入和导出所有的 Hadoop，但我们可以相当容易地更改版本，如果需要，我们还可以外部化定义集群的静态 XML 文件。

## 它是如何工作的

我们基本上欺骗了我们的包，使其能够在一个正常的类路径提供的 JVM 中模仿一个单一的单一类加载器。我们通过创建一个动态从 Apache Hadoop 包导入所需资源的包来实现这一点，并确保我们的包可以访问所有必要的配置资源。我们还调整了类加载，以便 Hadoop 代码库使用我们的包类加载器来实例化新实例。

# 添加与 HDFS 通信的命令以在 Karaf 中部署

由于 HDFS 本质上是一个文件系统，让我们看看我们如何使用标准工具和我们迄今为止构建的包来访问它。

我们将做的是将运行中的 Karaf 容器中的一级配置文件存储到 HDFS 中。然后，我们将提供第二个命令来读取这些文件。

我们已经学会了如何构建一个针对 Hadoop 的功能，该功能负责处理与 HDFS 通信所需的所有各种依赖项，我们还稍微提前了一点，讨论了类加载和一些技巧，以便我们部署的 Hadoop 库能够协作。我们现在可以开始使用提供的库编写针对 Hadoop 的代码了。

## 准备工作

这个配方所需的原料包括 Apache Karaf 发行套件、对 JDK 的访问和互联网连接。这个配方的示例代码可在[`github.com/jgoodyear/ApacheKarafCookbook/tree/master/chapter9/chapter-9-recipe1`](https://github.com/jgoodyear/ApacheKarafCookbook/tree/master/chapter9/chapter-9-recipe1)找到。记住，你需要安装驱动程序，并且 Apache Hadoop 的 HDFS 必须运行，这些配方才能工作！

## 如何做到这一点...

构建一个可以访问 Hadoop 的项目需要以下步骤：

1.  第一步是生成基于 Maven 的 bundle 项目。创建一个空的基于 Maven 的项目。一个包含基本 Maven 坐标信息和 bundle 打包指令的`pom.xml`文件就足够了。

1.  下一步是将依赖项添加到 POM 文件中。可以按以下方式完成：

    ```java
    <dependencies>
      <dependency>
        <groupId>org.osgi</groupId>
        <artifactId>org.osgi.core</artifactId>
        <version>5.0.0</version>
      </dependency>
      <dependency>
        <groupId>org.osgi</groupId>
        <artifactId>org.osgi.compendium</artifactId>
        <version>5.0.0</version>
      </dependency>
      <dependency>
        <groupId>org.osgi</groupId>
        <artifactId>org.osgi.enterprise</artifactId>
        <version>5.0.0</version>
      </dependency>
      <dependency>
        <groupId>org.apache.servicemix.bundles</groupId>
        <artifactId>org.apache.servicemix.bundles.hadoop-client</artifactId>
        <version>2.4.0_1</version>
      </dependency>
      <!-- custom felix gogo command -->
      <dependency>
        <groupId>org.apache.karaf.shell</groupId>
        <artifactId>org.apache.karaf.shell.console</artifactId>
        <version>3.0.0</version>
      </dependency>
    </dependencies>
    ```

    对于 Karaf 3.0.0，我们使用 OSGi 版本 5.0.0。Hadoop 库需要相当多的支持包。现有的 Camel 功能被用作起点，但实际上并不适用于所有平台，因此我们必须重写它以满足我们的需求。

1.  下一步是添加构建插件。我们的配方只需要配置一个构建插件，即 bundle 插件。我们配置`maven-bundle-plugin`将我们的项目代码组装成一个 OSGi 包。我们在 POM 文件中添加以下插件配置：

    ```java
    <plugin>
      <groupId>org.apache.felix</groupId>
      <artifactId>maven-bundle-plugin</artifactId>
      <version>2.4.0</version>
      <extensions>true</extensions>
      <configuration>
        <instructions>
          <Bundle-SymbolicName>${project.artifactId}</Bundle-SymbolicName>

          <Export-Package>
            com.packt.hadoop.demo*
          </Export-Package>
          <Import-Package>
            org.osgi.service.blueprint;resolution:=optional,
            org.apache.felix.service.command,
            org.apache.felix.gogo.commands,
            org.apache.karaf.shell.console,
            org.apache.hadoop*,
            *
          </Import-Package>

          <Include-Resource>
            {maven-resources},
            @org.apache.servicemix.bundles.hadoop-client-2.4.0_1.jar!/core-default.xml,
            @org.apache.servicemix.bundles.hadoop-client-2.4.0_1.jar!/hdfs-default.xml,
            @org.apache.servicemix.bundles.hadoop-client-2.4.0_1.jar!/mapred-default.xml,
            @org.apache.servicemix.bundles.hadoop-client-2.4.0_1.jar!/hadoop-metrics.properties
          </Include-Resource>

          <DynamicImport-Package>*</DynamicImport-Package>
        </instructions>
      </configuration>
    </plugin>
    ```

    Felix 和 Karaf 导入是可选的 Karaf 命令所必需的。随着我们启用动态类加载并在类加载器周围复制资源，以便它们可用，我们正在开始得到一个更复杂的 bundle 插件。

1.  下一步是创建一个 Blueprint 描述符文件。在项目中创建`src/main/resources/OSGI-INF/blueprint`目录树。然后，我们将在这个文件夹中创建一个名为`blueprint.xml`的文件。考虑以下代码：

    ```java
    <?xml version="1.0" encoding="UTF-8" standalone="no"?>
    <blueprint default-activation="eager">
      <!-- Define RecipeBookService Services, and expose them. -->
      <bean id="hdfsConfigService" class="com.packt.hadoop.demo.hdfs.HdfsConfigServiceImpl" init-method="init" destroy-method="destroy"/>

      <service ref="hdfsConfigService" interface="com.packt.hadoop.demo.api.HdfsConfigService"/>

      <!-- Apache Karaf Commands -->
      <command-bundle >
        <command>
          <action class="com.packt.hadoop.demo.commands.ReadConfigs">
            <property name="hdfsConfigService" ref="hdfsConfigService"/>
          </action>
        </command>
        <command>
          <action class="com.packt.hadoop.demo.commands.StoreConfigs">
            <property name="hdfsConfigService" ref="hdfsConfigService"/>
          </action>
        </command>

      </command-bundle>
    </blueprint>
    ```

1.  下一步是开发一个具有新 Hadoop 后端的 OSGi 服务。我们已经创建了基本的项目结构，并配置了 Blueprint 描述符的配置。现在，我们将专注于我们 Hadoop 后端应用程序的底层 Java 代码。我们将这个过程分解为两个步骤：定义服务接口和提供具体的实现。

    1.  首先，我们定义一个服务接口。服务接口将定义我们项目的用户 API。在我们的示例代码中，我们实现了`HdfsConfigService`接口，它提供了存储和检索我们 Karaf 实例中配置文件所需的方法。这可以按以下方式完成：

        ```java
        package com.packt.hadoop.demo.api;

        public interface HdfsConfigService {

          static final String HDFS_LOCATION = "/karaf/cookbook/etc";

          void storeConfigs();

          void readConfigs();
        }
        ```

        接口的实现遵循标准的 Java 约定，不需要特殊的 OSGi 包。

    1.  接下来，我们实现与 HDFS 的通信。现在我们已经定义了我们的服务接口，我们将通过两个调用提供实现，以将文件数据存储和检索到 HDFS。这可以通过以下方式完成：

        ```java
        public class HdfsConfigServiceImpl implements HdfsConfigService {

          private final static String BASE_HDFS = "hdfs://localhost:9000";

          @Override
          public void storeConfigs() {
            String KARAF_etc = System.getProperty("karaf.home") + "/etc";
            Collection<File> files = FileUtils.listFiles(new File(KARAF_etc), new String[]{"cfg"}, false);

            for (File f : files) {
              System.out.println(f.getPath());

              ClassLoader tccl = Thread.currentThread().getContextClassLoader();
              try {
                Thread.currentThread().setContextClassLoader(getClass().getClassLoader());
                String cfg = FileUtils.readFileToString(f);

                Path pt = new Path(BASE_HDFS + HDFS_LOCATION + "/" + f.getName());

                Configuration conf = new Configuration();
                conf.set("fs.hdfs.impl", org.apache.hadoop.hdfs.DistributedFileSystem.class.getName());
                conf.set("fs.file.impl", org.apache.hadoop.fs.LocalFileSystem.class.getName());

                FileSystem fs = FileSystem.get(conf);

                BufferedWriter br = new BufferedWriter(new OutputStreamWriter(fs.create(pt, true)));
                // TO append data to a file, use fs.append(Path f)

                br.write(cfg);
                br.close();
              } catch (IOException e) {
                e.printStackTrace();
              } finally {
                Thread.currentThread().setContextClassLoader(tccl);
              }
            }
          }

          @Override
          public void readConfigs() {
            try {

              FileSystem fs = FileSystem.get(new Configuration());
              FileStatus[] status = fs.listStatus(new Path(BASE_HDFS + HDFS_LOCATION));
              for (int i = 0;i < status.length;i++) {
                BufferedReader br = new BufferedReader(new InputStreamReader(fs.open(status[i].getPath())));
                String line;
                line = br.readLine();
                while (line != null) {
                  System.out.println(line);
                  line = br.readLine();
                }
              }
            } catch (Exception e) {
              e.printStackTrace();
            }
          }

          public void init() {

          }

          public void destroy() {

          }
        }
        ```

1.  下一步是可选地创建 Karaf 命令以直接测试持久化服务。为了简化对`HdfsConfigService`接口的手动测试，我们可以创建一组自定义 Karaf 命令，这些命令将测试我们的 HDFS 存储和检索操作。这些命令的示例实现可以在本书的网站上找到。特别值得注意的是，它们如何获取`HdfsConfigService`接口的引用并调用该服务。我们必须通过 Blueprint 将命令实现连接到 Karaf。这可以通过以下方式完成：

    ```java
    <!-- Apache Karaf Commands -->
    <command-bundle >
      <command>
        <action class="com.packt.hadoop.demo.commands.ReadConfigs">
          <property name="hdfsConfigService" ref="hdfsConfigService"/>
        </action>
      </command>
      <command>
        <action class="com.packt.hadoop.demo.commands.StoreConfigs">
          <property name="hdfsConfigService" ref="hdfsConfigService"/>
        </action>
      </command>

    </command-bundle>
    ```

    我们每个自定义命令的实现类都连接到我们的`hdfsConfigService`实例。

1.  下一步是将项目部署到 Karaf 中。

    ### 注意

    这个演示需要一个正在运行的 Hadoop 集群！

    我们确保所有 Hadoop 捆绑包都已正确安装。这可以通过以下命令完成：

    ```java
    karaf@root()> feature:repo-add mvn:com.packt/chapter-9-recipe1/1.0.0-SNAPSHOT/xml/features
    Adding feature url mvn:com.packt/chapter-9-recipe1/1.0.0-SNAPSHOT/xml/features

    karaf@root()> feature:install hdfs2
    karaf@root()>

    karaf@root()> list
    START LEVEL 100 , List Threshold: 50
     ID | State  | Lvl | Version        | Name
    --------------------------------------------------------------
    126 | Active |  50 | 2.6            | Commons Lang
    127 | Active |  50 | 16.0.1         | Guava: Google Core Libraries for Java
    128 | Active |  50 | 2.5.0          | Protocol Buffer Java API
    129 | Active |  50 | 3.0.0.1        | Apache ServiceMix :: Bundles :: guice 
    130 | Active |  50 | 0.1.51.1       | Apache ServiceMix :: Bundles :: jsch 
    131 | Active |  50 | 2.6.0.1        | Apache ServiceMix :: Bundles :: paranamer 
    132 | Active |  50 | 1.7.3.1        | Apache ServiceMix :: Bundles :: avro 
    133 | Active |  50 | 1.8.1          | Apache Commons Compress
    134 | Active |  50 | 3.1.1          | Commons Math
    135 | Active |  50 | 1.2            | Commons CLI
    136 | Active |  50 | 1.10.0         | Apache Commons Configuration
    137 | Active |  50 | 3.1.0.7        | Apache ServiceMix :: Bundles :: commons-httpclient
    138 | Active |  50 | 3.9.2.Final    | The Netty Project
    139 | Active |  50 | 1.9.12         | Jackson JSON processor
    140 | Active |  50 | 1.9.12         | Data mapper for Jackson JSON processor
    141 | Active |  50 | 1.9.12         | JAX-RS provider for JSON content type, using Jackson data binding
    142 | Active |  50 | 1.9.12         | XML Compatibility extensions for Jackson data binding
    143 | Active |  50 | 1.0.4.1_1      | Apache ServiceMix :: Bundles :: snappy-java
    144 | Active |  50 | 1.9.0          | Apache Commons Codec
    145 | Active |  50 | 3.2.1          | Commons Collections
    146 | Active |  50 | 2.4.0          | Commons IO
    147 | Active |  50 | 3.3.0          | Commons Net
    148 | Active |  50 | 3.4.6          | ZooKeeper Bundle
    149 | Active |  50 | 0.52.0.1       | Apache ServiceMix :: Bundles :: xmlenc
    150 | Active |  50 | 2.11.0.1       | Apache ServiceMix :: Bundles :: xercesImpl
    151 | Active |  50 | 2.4.0.1        | Apache ServiceMix :: Bundles :: hadoop-client
    152 | Active |  80 | 1.0.0.SNAPSHOT | Chapter 9 :: Manage Big Data with Apache Hadoop - HDFS Client Example.
    karaf@root()>

    ```

    我们通过在其 Maven 坐标上执行`install`命令来安装我们的项目捆绑包：

    ```java
    karaf@root()>  install –s mvn:com.packt/chapter-9-recipe1/1.0.0-SNAPSHOT

    ```

1.  最后一步是测试项目。我们可以使用以下命令来完成这项工作：

    ```java
    karaf@root()> test:storeconfigs 
    karaf@root()> test:readconfigs 
    karaf@root()>

    ```

## 它是如何工作的…

我们现在有一个与外部 HDFS 文件系统通信的 Karaf 容器。我们可以从 Karaf 备份配置文件到 HDFS，并且可以读取它们。

可以启动一个新的 Karaf 实例来消费和复制这些配置，或者我们可以使用这个配方作为启动 MapReduce 作业、任务和批处理作业的基础。
