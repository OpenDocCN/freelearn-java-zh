# 第十一章。强化 WildFly 配置

在本章中，你将学习以下食谱：

+   使用属性文件交付你的配置

+   保护你的配置哈希密码

+   使用保险库来保护和加密密码

# 简介

在本章中，你将学习如何使用不同的方法来保护你的 WildFly 系统配置。保护配置意味着隐藏敏感数据，如密码，以防止其他人以某种方式参与你的项目或系统。

这个目标可以通过不同的方式实现：

+   使用属性文件外部化动态参数，例如绑定、凭证等

+   哈希密码——这是一种相当常见的技巧

+   将密码存储在保险库中——这是你可以用来保护密码的最安全的方法

第一种方法并不完全安全，因此它是一种自定义设置的清洁方法。尽管如此，它仍然可以给你分发配置文件的自由，无需担心安全问题，因为其中只会有默认设置，没有更多。此外，你的每个 WildFly 基础设施环境都可以依赖于不同的属性文件。配置是相同的；你只需交付不同的属性文件。

最后两种方法更侧重于混淆密码。哈希使用哈希算法来加密密码。因此，无论谁在 XML 配置文件中看到密码，都只会看到其哈希值，而不是真正的密码。

将密码存储在保险库中要复杂一些，但提供了更好的保护，因为证书用于加密和解密密码。

# 使用属性文件交付你的配置

在这个食谱中，你将学习如何使用属性文件交付你的配置。如果你不想将所有设置硬编码到 XML 文件中，从而需要为每个环境更改它们，这可能会很有用。这样，你可以通过 XML 文件提供通用配置，并通过属性文件提供特定环境的特定设置。

多亏了 WildFly 提供的属性替换功能，你可以在 XML 文件（`standalone.xml`、`domain.xml`和`host.xml`）中使用`${your.property.goes.here}`语法。

## 准备工作

要开始，我们首先创建一个`ad-hoc`文件夹来运行我们的 WildFly。在终端窗口中运行以下命令：

```java
$ cd $WILDFLY_HOME
$ cp -a standalone sec-std-cfg-node-1
```

现在是时候创建一些属性了！

## 如何操作…

1.  首先，让我们创建一个名为`wildflycookbook.properties`的属性文件，并添加以下属性和值：

    ```java
    jboss.bind.address=10.0.0.1
    ```

1.  现在在你的系统中创建前面的虚拟 IP。在 Linux 中，你将打开一个终端窗口并运行以下命令：

    ```java
    $ sudo ifconfig eth0:1 10.0.0.1 netmask 255.255.255.0
    ```

    其中`eth0`是你的网络接口名称。

1.  现在，在终端窗口中，执行以下命令：

    ```java
    $ cd $WILDFLY_HOME
    $ ./bin/standalone.sh -Djboss.server.base.dir=sec-std-cfg-node-1 -P wildflycookbook.properties
    ...
    17:24:05,588 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-16) WFLYUT0006: Undertow HTTP listener default listening on /10.0.0.1:8080
    ...
    17:24:06,019 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0025: WildFly Full 9.0.0.Beta2 (WildFly Core 1.0.0.Beta2) started in 2822ms - Started 202 of 379 services (210 services are lazy, passive or on-demand)
    ```

    如您从日志输出中看到的，强调的字符，属性已从 WildFly 读取并在运行时放置到其配置中。因此，现在我们的 WildFly 实例绑定到`http://10.0.0.1:8080`。

## 它是如何工作的...

这种方法相当简单直接；除了它的用法之外，没有太多可说的。前面的例子只是给你一个关于你可以用它做什么的初步想法。

`-P file.property`指令是 WildFly 的一个功能，可以从属性文件中加载大量系统属性。WildFly 通过为其配置提供属性替换来利用这种机制。

想象一下以下的数据源配置：

```java
<subsystem >
    <datasources>
    <datasource jndi-name="java:jboss/MySqlDS" pool-name="MySqlDS">
      <connection-url>jdbc:mysql://mysql-prod-cluster-node-1:3306/store</connection-url>
      <driver>mysql</driver>
      <security>
        <user-name>root</user-name>
        <password>1password</password>
      </security>
    </datasource>
    <drivers/>
    </datasources>
</subsystem>
```

如您所见，连接到数据库的参数，如服务器名称、服务器端口、数据库名称和凭证，都硬编码到文件中。

现在假设你需要将 WildFly 配置复制到其他地方，但实际工作将由你公司外的人来完成。你会给这个人所有这些信息吗？我想不会。

拥有一个属性文件可以在某种程度上保护你不泄露这样的信息，如下所示：

```java
<subsystem >
    <datasources>
    <datasource jndi-name="java:jboss/MySqlDS" pool-name="MySqlDS">
      <connection-url>${db.prod.conn.url}</connection-url>
      <driver>mysql</driver>
      <security>
        <user-name>${db.prod.uid}</user-name>
        <password>${db.prod.pwd}</password>
      </security>
    </datasource>
    <drivers/>
    </datasources>
</subsystem>
```

显然，需要找到前面的属性并将其匹配到属性文件中，内容如下：

```java
db.prod.conn.url=jdbc:mysql://mysql-uat-cluster-node-1:3306/store
db.prod.uid=uat
dp.prod.pwd=uat-password
```

更好！

## 还有更多...

请记住，在 Java 中传递属性或属性文件时，属性文件具有优先级。因此，在指定属性文件后使用`-D`符号传递属性将优先于相同的属性。最后一个获胜！

# 保护配置哈希密码

在这个菜谱中，你将学习如何将密码作为配置文件进行掩码处理，这样它们就不会被看到，或者更好，对查看它们的人来说没有意义。

你可能听说过将应该保密或私密的文本（如密码）进行转换。很多时候，术语如编码、加密和哈希被不加区分地使用，但它们并不相同。在我们继续之前，让我们澄清这些概念。

编码文本是将它转换为可读的，并使其适用于不同的格式（如 HTML 中的`&`符号应转换为`&amp;`）。

加密是将文本转换为使其保密和没有意义的过程。这种转换基于一个密钥和一个算法（如 AES、RSA 和 Blowfish），这样密钥和算法的组合使得文本与其原始内容完全不同。只有知道密钥和算法的客户才能将加密的文本恢复到原始值。

哈希是关于完整性的。这意味着它用于检查一个消息（文本、密码、文件等）在到达目的地期间是否被更改。哈希是不可逆的。给定一个输入值，你将始终得到相同的哈希输出。如果你对你的源进行微小的更改，你将得到一个完全不同的哈希输出。有几种哈希函数算法，如 MD5、SHA-3 等。

让我们尝试对一个小的文本片段进行 MD5 校验和，如下所示：

```java
$ md5sum <<< abcdef
5ab557c937e38f15291c04b7e99544ad  -
$ md5sum <<< abcdeF
7ab0aeb4ce14fd8efa00f3c903f72cf5  -
```

如你所见，只需将最后一个字符从`f`（小写）改为`F`（大写），MD5 校验和散列函数就会给出完全不同的输出。

顺便说一下，在本章中，我们将只使用术语散列，无论是通过编码、加密还是散列转换后的密码。

好的，我希望我已经给你提供了一些关于这类真正需要深入解释的话题的更多信息。除了你可以自己找到的大量技术书籍之外，还有一本相当有趣的书籍名为《密码之书》，作者是*西蒙·辛格*。

让我们回到 WildFly。

## 准备工作

要开始，让我们首先创建一个`ad-hoc`文件夹来运行我们的 WildFly。在终端窗口中执行以下命令：

```java
$ cd $WILDFLY_HOME
$ cp -a standalone sec-std-cfg-node-2
```

现在是时候为我们的密码创建散列了。

## 如何操作…

1.  打开一个新的终端窗口并运行以下命令：

    ```java
    $ cd $WILDFLY_HOME
    $ java -cp modules/system/layers/base/org/picketbox/main/picketbox-4.9.0.Beta2.jar org.picketbox.datasource.security.SecureIdentityLoginModule cookbook.2015
    Encoded password: 2663ecfd3089b80f99cabb669aa2636e
    ```

    前述脚本的输出就是你的散列密码。

1.  为了完成这个任务，我们依赖于安全域。创建一个如下所示的安全域：

    ```java
    <security-domain name="encrypted-security-domain" cache-type="default">
      <authentication>
        <login-module 
        code="org.picketbox.datasource.security.SecureIdentityLoginModule" flag="required">
          <module-option name="username" value="root"/>
          <module-option name="password" value="2663ecfd3089b80f99cabb669aa2636e"/>
        </login-module>
      </authentication>
    </security-domain>
    ```

1.  现在，让我们回到我们的数据源定义，并按照以下方式引用安全域：

    ```java
    <subsystem >
        <datasources>
        <datasource jndi-name="java:jboss/MySqlDS" pool-
        name="MySqlDS">
          <connection-url>jdbc:mysql://mysql-prod-cluster-node-1:3306/store</connection-url>
          <driver>mysql</driver>
          <security>
            <security-domain>encrypted-security-domain</security-domain>
          </security>
        </datasource>
        <drivers/>
        </datasources>
    </subsystem>
    ```

现在它工作了。真不错！

## 它是如何工作的…

你所要做的就是使用 WildFly 提供的 Pickbox 安全框架生成散列，并使用它的`SecurityIdentityLoginModule`。

之后，你需要在 WildFly 配置中创建一个安全域，该安全域使用与生成密码相同的程序（即`SecurityIdentityLoginModule`类）。

接下来，每次你需要提供一个散列密码时，只需引用该安全域即可完成。在这个过程中，安全域首先为接收到的密码生成一个散列，然后将生成的散列与你在配置中存储的散列进行匹配。其余的只是一个匹配/不匹配的问题。

要能在配置文件中使用那个密码散列，我们需要定义一个使用与生成散列完全相同功能的安全域，否则将很难区分明文密码和加密密码。

例如，在数据源定义中，你不能有以下的 XML 代码片段：

```java
<subsystem >
    <datasources>
    <datasource jndi-name="java:jboss/MySqlDS" pool-name="MySqlDS">
      <connection-url>jdbc:mysql://mysql-prod-cluster-node-1:3306/store</connection-url>
      <driver>mysql</driver>
      <security>
        <user-name>root</user-name>
        <password>2663ecfd3089b80f99cabb669aa2636e</password>
      </security>
    </datasource>
    <drivers/>
    </datasources>
</subsystem>
```

也就是说，使用密码散列代替的方法是不可行的！谁能判断出这个密码是明文还是散列的？

## 更多…

这种方法不仅适用于数据源，甚至还可以用于`login-module`本身、JMS 队列和主题。

# 使用保险库保护密码

在这个菜谱中，你将学习如何保护我们的密码，同时仍然将它们提供给我们的 WildFly 配置。保险库是一个存储密码的地方，使用密钥库进行加密。在我们的菜谱中，我们将创建一个密钥库并存储用于连接 MySQL 数据库的密码。MySQL 安装不在此书的范围之内；如果你需要更多信息，请参阅 MySQL 文档网站[`dev.mysql.com/doc/refman/5.5/en/installing.html`](https://dev.mysql.com/doc/refman/5.5/en/installing.html)。

## 准备工作

要开始，让我们首先创建一个`ad-hoc`文件夹来运行我们的 WildFly：

1.  在终端窗口中运行以下命令：

    ```java
    $ cd $WILDFLY_HOME
    $ cp -a standalone sec-std-cfg-node-3
    ```

    现在是时候创建我们的密钥库了。

1.  在同一个终端窗口中，执行以下操作：

    ```java
    $ cd $WILDFLY_HOME
    $ cd sec-std-cfg-node-3/configuration
    $ mkdir vault
    $ cd vault
    $ keytool -v -genkey -alias wildfly.vault -keyalg RSA -keysize 2048 -sigalg SHA1withRSA -keystore wildfly.vault.keystore
    Enter keystore password: [vault.2015]
    Re-enter new password: [vault.2015]
    What is your first and last name?
      [Unknown]:  WildFly Cookbook
    What is the name of your organizational unit?
      [Unknown]:  Packt Publishing
    What is the name of your organization?
      [Unknown]:  packtpub.com
    What is the name of your City or Locality?
      [Unknown]:  Birmingham
    What is the name of your State or Province?
      [Unknown]:  GB
    What is the two-letter country code for this unit?
      [Unknown]:  UK
    Is CN=WildFly Cookbook, OU=Packt Publishing, O=packtpub.com, L=Birmingham, ST=GB, C=UK correct?
      [no]:  yes

    Generating 2,048 bit RSA key pair and self-signed certificate (SHA1withRSA) with a validity of 90 days
      for: CN=WildFly Cookbook, OU=Packt Publishing, O=packtpub.com, L=Birmingham, ST=GB, C=UK
    Enter key password for <wildfly.vault>
      (RETURN if same as keystore password):
    [Storing wildfly.vault.keystore]
    ```

1.  好的，现在我们已经创建了密钥库来加密我们的密码。让我们通过执行以下命令来检查其完整性：

    ```java
    $ keytool -list -v -keystore wildfly.vault.keystore
    Enter keystore password:

    Keystore type: JKS
    Keystore provider: SUN

    Your keystore contains 1 entry

    Alias name: wildfly.vault
    Creation date: Dec 6, 2014
    Entry type: PrivateKeyEntry
    Certificate chain length: 1
    Certificate[1]:
    Owner: CN=WildFly Cookbook, OU=Packt Publishing, O=packtpub.com, L=Birmingham, ST=GB, C=UK
    Issuer: CN=WildFly Cookbook, OU=Packt Publishing, O=packtpub.com, L=Birmingham, ST=GB, C=UK
    Serial number: 6cfc82e9
    Valid from: Sat Dec 06 14:43:16 CET 2014 until: Fri Mar 06 14:43:16 CET 2015
    Certificate fingerprints:
       MD5:  FA:A6:5F:E6:7F:04:70:7E:70:FA:56:E7:9C:8A:B8:95
       SHA1: D2:18:BE:44:58:B1:57:54:0A:69:F9:E2:DB:3F:A7:82:7D:DE:DB:6B
       SHA256: 8D:DF:16:64:3D:F6:08:08:71:8A:5F:4C:16:27:82:C8:20:75:FC:67:DD:9D:68:0E:0C:6F:08:7F:FA:E8:5D:18
       Signature algorithm name: SHA1withRSA
       Version: 3

    Extensions:

    #1: ObjectId: 2.5.29.14 Criticality=false
    SubjectKeyIdentifier [
    KeyIdentifier [
    0000: 68 21 C0 CE C1 A5 A0 11   3D 5B EE E8 18 92 72 ED  h!......=[....r.
    0010: C4 28 46 2B                                        .(F+
    ]
    ]

    *******************************************
    *******************************************
    ```

好的，一切正常！我们现在准备好加密我们的密码并将它们存储到保险库中。

## 如何做到这一点...

要生成一个保险库，WildFly 在`bin`文件夹内提供了一个脚本，名为`vault.sh`。

作为示例，我们将生成一个存储用于连接数据库的密码的保险库：

1.  打开一个终端窗口并执行以下操作：

    ```java
    $ cd $WILDFLY_HOME/sec-std-cfg-node-3/configuration
    $ $WILDFLY_HOME/bin/vault.sh -a PASSWORD -x cookbook.2015 -b DB-PROD -i 50 -k vault/wildfly.vault.keystore -p vault.2015 -s 86427531 -v wildfly.vault

    ...

    Dec 06, 2014 10:28:20 PM org.picketbox.plugins.vault.PicketBoxSecurityVault init
    INFO: PBOX000361: Default Security Vault Implementation Initialized and Ready
    Secured attribute value has been stored in Vault.
    Please make note of the following:
    ********************************************
    Vault Block:DB-PROD
    Attribute Name:PASSWORD
    Configuration should be done as follows:
    VAULT::DB-PROD::PASSWORD::1
    ********************************************
    Vault Configuration in WildFly configuration file:
    ********************************************
    ...
    </extensions>
    <vault>
      <vault-option name="KEYSTORE_URL" value="vault/wildfly.vault.keystore"/>
      <vault-option name="KEYSTORE_PASSWORD" value="MASK-1cn1ENbx4KLXxve5VvGlPy"/>
      <vault-option name="KEYSTORE_ALIAS" value="wildfly.vault"/>
      <vault-option name="SALT" value="86427531"/>
      <vault-option name="ITERATION_COUNT" value="50"/>
      <vault-option name="ENC_FILE_DIR" value="vault/"/>
    </vault><management> ...
    ********************************************
    ```

    上述脚本是`vault.sh`的输出，它描述了以下要点：

    +   如何引用保险库中存储的密码

    +   在`vault`文件夹内名为`VAULT.dat`的文件

    +   一个要放置到`standalone.xml`或`host.xml`文件中的 XML 代码片段

1.  我们现在可以在数据源配置中使用保险库，如下所示：

    ```java
    <datasource jndi-name="java:jboss/datasources/VaultDS" pool-name="VaultDS" enabled="true" use-java-context="true">
        <connection-url>jdbc:mysql://192.168.59.103:3306/test
    </connection-url>
        <driver>mysql</driver>
        <security>
            <user-name>root</user-name>
            <password>${VAULT::DB-PROD::PASSWORD::1}</password>
        </security>
    </datasource>
    ```

    为了测试我们的配置，我们需要一个正在运行的 MySQL 服务器。在我的情况下，它绑定到了运行在`192.168.59.103:3306`的服务器上。此外，我们需要配置连接池，以便立即将有效的连接放入其中，作为证明。

1.  因此，数据源配置如下：

    ```java
    <datasource jndi-name="java:jboss/datasources/VaultDS" pool-name="VaultDS" enabled="true">
        <connection-url>jdbc:mysql://192.168.59.103:3306/test</connection-
        url>
        <driver>mysql</driver>
        <pool>
            <min-pool-size>5</min-pool-size>
            <max-pool-size>5</max-pool-size>
            <prefill>true</prefill>
        </pool>
        <security>
            <user-name>root</user-name>
            <password>${VAULT::DB-PROD::PASSWORD::1}</password>
        </security>
    </datasource>
    ```

1.  让我们启动我们的 WildFly 实例并查看日志。运行以下命令：

    ```java
    $ ./bin/standalone.sh -Djboss.server.base.dir=sec-std-cfg-node-3
    ...
    15:32:35,607 INFO  [org.jboss.as.server.deployment.scanner] (MSC service thread 1-4) JBAS015012: Started FileSystemDeploymentService for directory /opt/wildfly/sec-std-cfg-node-2/deployments
    15:32:35,636 INFO  [org.jboss.as.connector.subsystems.datasources] (MSC service thread 1-1) JBAS010400: Bound data source [java:jboss/datasources/VaultDS]
    15:32:35,636 INFO  [org.jboss.as.connector.subsystems.datasources] (MSC service thread 1-13) JBAS010400: Bound data source [java:jboss/datasources/ExampleDS]
    15:32:35,676 INFO  [org.jboss.ws.common.management] (MSC service thread 1-12) JBWS022052: Starting JBoss Web Services - Stack CXF Server 4.2.4.Final
    15:32:35,722 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015961: Http management interface listening on http://127.0.0.1:9990/management
    15:32:35,723 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015951: Admin console listening on http://127.0.0.1:9990
    15:32:35,723 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015874: WildFly 8.1.0.Final "Kenny" started in 3854ms - Started 203 of 251 services (81 services are lazy, passive or on-demand)
    ```

1.  在日志中，我已经强调了指出`VaultDS`数据源的声明，该数据源已成功绑定。现在打开一个新的终端窗口并连接到 CLI 以检查我们的可用连接，如下所示：

    ```java
    $ ./bin/jboss-cli.sh
    You are disconnected at the moment. Type 'connect' to connect to the server or 'help' for the list of supported commands.
    [disconnected /] connect
    [standalone@localhost:9990 /] /subsystem=datasources/data-source=VaultDS/statistics=pool:read-resource(include-runtime=true)
    {
        "outcome" => "success",
        "result" => {
            "ActiveCount" => "5",
            "AvailableCount" => "5",
            "AverageBlockingTime" => "0",
            "AverageCreationTime" => "36",
            "AverageGetTime" => "0",
            "BlockingFailureCount" => "0",
            "CreatedCount" => "5",
            "DestroyedCount" => "0",
            "IdleCount" => "5",
            "InUseCount" => "0",
            "MaxCreationTime" => "43",
            "MaxGetTime" => "0",
            "MaxUsedCount" => "1",
            "MaxWaitCount" => "0",
            "MaxWaitTime" => "0",
            "TimedOut" => "0",
            "TotalBlockingTime" => "0",
            "TotalCreationTime" => "180",
            "TotalGetTime" => "0",
            "WaitCount" => "0"
        }
    }
    [standalone@localhost:9990 /]
    ```

    太好了！从前面的输出中，我们可以得出结论，我们的`VaultDS`数据源连接池已填充了五个可用和活动的连接。

1.  现在为了证明我们做得很好，让我们尝试添加两个更多连接到同一数据库的数据源；一个使用明文密码，另一个使用错误的保险库定义。添加以下数据源定义：

    ```java
    <datasource jndi-name="java:jboss/datasources/UnsecureDS" pool-name="UnsecureDS" enabled="true">
        <connection-url>jdbc:mysql://192.168.59.103:3306/test</connection-url>
        <driver>mysql</driver>
        <pool>
            <min-pool-size>2</min-pool-size>
            <max-pool-size>2</max-pool-size>
            <prefill>true</prefill>
        </pool>
        <security>
            <user-name>root</user-name>
            <password>cookbook.2015</password>
        </security>
    </datasource>
    <datasource jndi-name="java:jboss/datasources/WrongVaultDS" pool-name="WrongVaultDS" enabled="true">
        <connection-url>jdbc:mysql://192.168.59.103:3306/test</connection-url>
        <driver>mysql</driver>
        <pool>
            <min-pool-size>1</min-pool-size>
            <max-pool-size>1</max-pool-size>
            <prefill>true</prefill>
        </pool>
        <security>
            <user-name>root</user-name>
            <password>${VAULT::DB-TEST::PASSWORD::1}</password>
        </security>
    </datasource>
    ```

1.  `UnsecureDS`数据源使用明文密码连接到数据库，而`WrongVaultDS`使用错误的`vault-block`，即我们未创建的`DB-TEST`。让我们再次启动 WildFly 并捕获错误日志。执行以下命令：

    ```java
    $ ./bin/standalone.sh -Djboss.server.base.dir=sec-std-cfg-node-3
    ...
    15:52:08,452 ERROR [org.jboss.as.controller.management-operation] (ServerService Thread Pool -- 27) JBAS014612: Operation ("add") failed - address: ([
        ("subsystem" => "datasources"),
     ("data-source" => "WrongVaultDS")
    ]): java.lang.SecurityException: JBAS013311: Security Exception
     at  org.jboss.as.security.vault.RuntimeVaultReader.retrieveFromVault(RuntimeVaultReader.java:104)
     at org.jboss.as.server.RuntimeExpressionResolver.resolvePluggableExpression(RuntimeExpressionResolver.java:45)
    ...
    15:52:08,499 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-2) JBAS017519: Undertow HTTP listener default listening on /127.0.0.1:8080
    15:52:08,499 INFO  [org.wildfly.extension.undertow] (MSC service thread 1-4) JBAS017519: Undertow AJP listener ajp listening on /127.0.0.1:8009
    15:52:08,631 ERROR [org.jboss.as.controller.management-operation] (Controller Boot Thread) "JBAS014784: Failed executing subsystem datasources boot operations"
    15:52:08,633 ERROR [org.jboss.as.controller.management-operation] (Controller Boot Thread) JBAS014613: Operation ("parallel-subsystem-boot") failed - address: ([]) - failure description: "\"JBAS014784: Failed executing subsystem datasources boot operations\""
    15:52:08,670 INFO  [org.jboss.as.server.deployment.scanner] (MSC service thread 1-1) JBAS015012: Started FileSystemDeploymentService for directory /Users/foogaro/wildfly-8.1.0.Final/sec-std-cfg-node-2/deployments
    15:52:08,705 INFO  [org.jboss.ws.common.management] (MSC service thread 1-5) JBWS022052: Starting JBoss Web Services - Stack CXF Server 4.2.4.Final
    15:52:08,706 ERROR [org.jboss.as.controller.management-operation] (Controller Boot Thread) JBAS014613: Operation ("add") failed - address: ([
     ("subsystem" => "datasources"),
     ("data-source" => "WrongVaultDS")
    ]) - failure description: "JBAS014749: Operation handler failed: JBAS013311: Security Exception"
    15:52:08,748 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015961: Http management interface listening on http://127.0.0.1:9990/management
    15:52:08,748 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015951: Admin console listening on http://127.0.0.1:9990
    15:52:08,749 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015874: WildFly 8.1.0.Final "Kenny" started in 3150ms - Started 183 of 231 services (81 services are lazy, passive or on-demand)
    ```

    如您所见，日志显示错误并抱怨关于`WrongVaultDS`的`SecurityException`。因此，所有的数据源子系统都处于错误状态且未初始化。

1.  尝试从数据源定义中移除`WrongVaultDS`，再次启动 WildFly，你应该会找到以下日志条目：

    ```java
    15:57:07,585 INFO  [org.jboss.as.connector.subsystems.datasources] (MSC service thread 1-11) JBAS010400: Bound data source [java:jboss/datasources/VaultDS]
    15:57:07,585 INFO  [org.jboss.as.connector.subsystems.datasources] (MSC service thread 1-16) JBAS010400: Bound data source [java:jboss/datasources/UnsecureDS]
    ```

如您所见，`VaultDS`和`UnsecureDS`都正确地绑定了。

## 它是如何工作的…

`keytool`命令是由 Java SE 平台提供的工具，但理解其用法和参数超出了本书的范围。然而，在食谱的*另请参阅...*部分提供了一个由 Oracle 提供的官方文档链接。

让我们分析一下 WildFly 为您提供的这个食谱中使用的 vault 脚本。让我们首先通过在终端窗口中执行以下命令来查看前一个脚本提供了哪些选项：

```java
$ ./bin/vault.sh --help
usage: vault.sh <empty> |  [-a <arg>] [-b <arg>] -c | -h | -x <arg> [-e
       <arg>]  [-i <arg>] [-k <arg>] [-p <arg>] [-s <arg>] [-v <arg>]
 -a,--attribute <arg>           Attribute name
 -b,--vault-block <arg>         Vault block
 -c,--check-sec-attr            Check whether the secured attribute
                                already exists in the Vault
 -e,--enc-dir <arg>             Directory containing encrypted files
 -h,--help                      Help
 -i,--iteration <arg>           Iteration count
 -k,--keystore <arg>            Keystore URL
 -p,--keystore-password <arg>   Keystore password
 -s,--salt <arg>                8 character salt
 -v,--alias <arg>               Vault keystore alias
 -x,--sec-attr <arg>            Secured attribute value (such as
                                password)to store
```

现在检查我们是如何调用脚本的：

```java
$ cd $WILDFLY_HOME/sec-std-cfg-node-3/configuration
$ $WILDFLY_HOME/bin/vault.sh -a PASSWORD -x cookbook.2015 -b DB-PROD -i 50 -k vault/wildfly.vault.keystore -p vault.2015 -s 86427531 -v wildfly.vault
```

因此，我们使用 `-a`（减号 `a`）标记创建了一个名为 `PASSWORD` 的属性，然后使用 `-x`（减号 `x`）标记将其值设置为 `cookbook.2015`。接着，我们使用 `-b`（减号 `b`）标记创建了一个 vault-block 来存储该属性及其值。其余的参数用于绑定加密属性值的密钥库，这就是我们的密码。

`vault.sh` 脚本为我们生成一个有用的输出，用于调用和引用 vault 和 vault-block，它们映射到我们的密码。使用它所需的信息，WildFly 能够自动提取密码并将其传递给请求它的组件。

## 还有更多...

只需提供不同的 `vault-block`，vault 就可以存储更多的密码。

例如，在这个食谱的 *如何做…* 部分的末尾，我们尝试使用不同的 `vault-block` 绑定一个数据源（`WrongVaultDS`）；我们称之为 `DB-TEST`。结果，我们无法绑定该数据源，因此整个数据源子系统处于错误状态。现在让我们尝试添加相同的数据库定义，但这次，我们还将 `DB-TEST` vault-block 提供给我们的 vault 文件：

1.  打开一个终端窗口，并执行以下命令：

    ```java
    $ cd $WILDFLY_HOME/sec-std-cfg-node-2/configuration
    $ $WILDFLY_HOME/bin/vault.sh -a PASSWORD -x cookbook.2015 -b DB-TEST -i 50 -k vault/wildfly.vault.keystore -p vault.2015 -s 86427531 -v wildfly.vault
    …
    Dec 08, 2014 4:20:17 PM org.picketbox.plugins.vault.PicketBoxSecurityVault init
    INFO: PBOX000361: Default Security Vault Implementation Initialized and Ready
    Secured attribute value has been stored in Vault.
    Please make note of the following:
    ********************************************
    Vault Block:DB-TEST
    Attribute Name:PASSWORD
    Configuration should be done as follows:
    VAULT::DB-TEST::PASSWORD::1
    ********************************************
    Vault Configuration in WildFly configuration file:
    ********************************************
    ...
    </extensions>
    <vault>
      <vault-option name="KEYSTORE_URL" value="vault/wildfly.vault.keystore"/>
      <vault-option name="KEYSTORE_PASSWORD" value="MASK-1cn1ENbx4KLXxve5VvGlPy"/>
      <vault-option name="KEYSTORE_ALIAS" value="wildfly.vault"/>
      <vault-option name="SALT" value="86427531"/>
      <vault-option name="ITERATION_COUNT" value="50"/>
      <vault-option name="ENC_FILE_DIR" value="vault/"/>
    </vault><management> ...
    ********************************************
    ```

1.  现在编辑 `standalone.xml` 文件并添加以下 XML 代码片段：

    ```java
    <datasource jndi-name="java:jboss/datasources/WrongVaultDS" pool-name="WrongVaultDS" enabled="true">
        <connection-url>jdbc:mysql://192.168.59.103:3306/test</connection-url>
        <driver>mysql</driver>
        <pool>
            <min-pool-size>1</min-pool-size>
            <max-pool-size>1</max-pool-size>
            <prefill>true</prefill>
        </pool>
        <security>
            <user-name>root</user-name>
            <password>${VAULT::DB-TEST::PASSWORD::1}</password>
        </security>
    </datasource>
    ```

1.  现在，像往常一样启动 WildFly 并查看日志。运行以下命令：

    ```java
    $ ./bin/standalone.sh -Djboss.server.base.dir=sec-std-cfg-node-3
    ...
    16:33:02,063 INFO  [org.jboss.as.connector.subsystems.datasources] (MSC service thread 1-3) JBAS010400: Bound data source [java:jboss/datasources/WrongVaultDS]
    16:33:02,063 INFO  [org.jboss.as.connector.subsystems.datasources] (MSC service thread 1-6) JBAS010400: Bound data source [java:jboss/datasources/ExampleDS]
    16:33:02,063 INFO  [org.jboss.as.connector.subsystems.datasources] (MSC service thread 1-2) JBAS010400: Bound data source [java:jboss/datasources/VaultDS]
    16:33:02,063 INFO  [org.jboss.as.connector.subsystems.datasources] (MSC service thread 1-9) JBAS010400: Bound data source [java:jboss/datasources/UnsecureDS]
    16:33:02,064 INFO  [org.jboss.ws.common.management] (MSC service thread 1-5) JBWS022052: Starting JBoss Web Services - Stack CXF Server 4.2.4.Final
    16:33:02,118 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015961: Http management interface listening on http://127.0.0.1:9990/management
    16:33:02,119 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015951: Admin console listening on http://127.0.0.1:9990
    16:33:02,120 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015874: WildFly 8.1.0.Final "Kenny" started in 1992ms - Started 203 of 251 services (81 services are lazy, passive or on-demand)
    ```

成功了！正如你所看到的，只要声明不同的 vault-block，你就可以在 vault 中定义和存储所有你想要的密码。

## 参见

+   要深入了解 keytool 命令，请参阅 Oracle 官方文档，链接为 [`docs.oracle.com/javase/8/docs/technotes/tools/unix/keytool.html`](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/keytool.html)
