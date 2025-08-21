# 第五章：Spring 与 FTP 的集成

FTP 涉及使用文件传输协议从一台计算机通过互联网发送文件到另一台计算机。Spring 集成还提供了对文件传输协议的支持。可以通过 FTP 或使用 SFTP（安全 FTP）进行文件传输。

以下是 FTP 场景中使用的一些缩写：

+   **FTP**：**文件传输协议**。

+   **FTPS**：**FTP 安全**是 FTP 的扩展，它添加了对**传输层安全**（**TLS**）和**安全套接字层**（**SSL**）加密协议的支持。

+   **SFTP**：**SSH 文件传输协议**，即 FTP 通过安全外壳协议。

在实际场景中，文件服务器将具有 FTP 地址、用户名和密码。客户端连接到服务器以传输文件。我们可以使用 FTP 上传文件到远程位置或从远程位置下载文件。

Spring 的集成包支持从 FTP 或 FTPS 服务器发送和接收文件。它提供了一些端点，以下是 Spring 为 FTP/FTPS 提供的端点/适配器：

+   入站通道适配器

+   出站通道适配器

+   出站网关

通道适配器只是消息端点，实际上将消息连接到消息通道。在处理通道适配器时，我们可以明显看到、发送和接收配置和方法。

在本章中，我们将看到 Spring 如何使我们能够使用 FTP，并开发一个演示 Spring 集成能力支持文件传输的示例应用程序。我们还将看到需要编写的配置以及如何使用入站和出站适配器来使用 Spring 集成包在 FTP 上传输文件。

# Maven 依赖项

为了使用 Spring 集成框架创建 FTP 应用程序，在 Maven 项目的`pom.xml`文件中添加以下依赖项。主要包括 Spring 集成测试和 Spring 集成 FTP。这些库可以从 Maven 仓库下载，也可以添加到项目的`pom.xml`文件中。

以下是需要添加到`pom.xml`文件中的 Maven 依赖项，以开始使用 Spring Integration FTP 包开发应用程序：

```java
<dependency>
  <groupId>org.springframework.integration</groupId>
  <artifactId>spring-integration-ftp</artifactId>
  <version>4.0.0.RELEASE</version>
  <scope>compile</scope>
</dependency>

<dependency>
  <groupId>org.springframework.integration</groupId>
  <artifactId>spring-integration-test</artifactId>
  <version>4.0.0.RELEASE</version>
  <scope>test</scope>
</dependency>

<dependency>
  <groupId>org.apache.ftpserver</groupId>
  <artifactId>ftpserver-core</artifactId>
  <version>1.0.6</version>
  <scope>compile</scope>
</dependency>
```

# Spring 的 FTP 的 XSD

让我们看看 Spring 集成包为 FTP 提供的 XSD。这包含了所有模式定义，并提供了 Spring 支持的所有配置可能性，因此配置 XML 文件变得更容易。

XSD（[`www.springframework.org/schema/integration/ftp/spring-integration-ftp.xsd`](http://www.springframework.org/schema/integration/ftp/spring-integration-ftp.xsd)）提供了关于 Spring 与 FTP 集成的大量信息。它为我们提供了有关在 XML 配置文件中配置通道适配器的信息。

入站和出站通道适配器是 XSD 中的两个主要元素。以下是从我们刚提到的链接中提取的 XSD 的摘录：

```java
<xsd:element name="outbound-channel-adapter">...</xsd:element>
<xsd:element name="inbound-channel-adapter">...</xsd:element>
<xsd:complexType name="base-ftp-adapter-type">...</xsd:complexType>
</xsd:schema>
```

在接下来的章节中，我们将看到如何配置入站和出站通道适配器以及 Spring 集成支持的 FTP 的配置选项。

## 为 FTP 配置出站通道适配器

出站通道适配器配置是针对远程目录的。它旨在执行诸如将文件写入远程服务器（文件上传）、创建新文件或在远程 FTP 服务器上添加后缀等操作。以下是 XSD 中提供的出站通道适配器的一些可用配置：

+   它支持使用正则表达式配置远程目录以写入文件。使用的属性如下：

```java
<xsd:attribute name="remote-directory-expression"type="xsd:string">
```

+   我们还可以配置自动在远程位置创建目录：

```java
<xsd:attribute name="auto-create-directory" type="xsd:string" default="false">
```

+   我们还可以配置 Spring 集成框架以与 FTP 一起工作，临时为文件添加后缀：

```java
<xsd:attribute name="temporary-file-suffix" type="xsd:string">
```

+   另一个重要的配置是在 FTP 服务器的远程位置生成文件名：

```java
<xsd:attribute name="remote-filename-generator" type="xsd:string">
```

+   前面的功能再次升级以支持正则表达式：

```java
<xsd:attribute name="remote-filename-generator-expression" type="xsd:string">
```

## 配置 FTP 的入站通道适配器

入站通道适配器配置是针对本地目录的，即旨在执行从远程服务器写入文件（文件下载）、创建新文件或在本地目录上添加后缀等操作。入站通道适配器确保本地目录与远程 FTP 目录同步。

从 XSD 中可用的入站通道适配器的一些配置如下：

+   它提供了配置选项，以自动创建本地目录（如果不存在）：

```java
<xsd:attribute name="auto-create-local-directory" type="xsd:string">
  <xsd:annotation>
    <xsd:documentation>Tells this adapter if local directory must be auto-created if it doesn't exist. Default is TRUE.</xsd:documentation> 
  </xsd:annotation>
</xsd:attribute>
```

+   它提供了配置远程服务器的选项，并在将其复制到本地目录后删除远程源文件：

```java
<xsd:attribute name="delete-remote-files" type="xsd:string">
  <xsd:annotation>
    <xsd:documentation>Specify whether to delete the remote source file after copying. By default, the remote files will NOT be deleted.</xsd:documentation> 
  </xsd:annotation>
</xsd:attribute>
```

+   使用可用的比较器配置对文件进行排序：

```java
<xsd:attribute name="comparator" type="xsd:string">
<xsd:annotation>
```

指定在排序文件时要使用的比较器。如果没有提供，则顺序将由`java.io`文件实现确定：

```java
</xsd:documentation>
  </xsd:annotation>
  </xsd:attribute>
```

+   使用以下属性配置会话缓存：

```java
<xsd:attribute name="cache-sessions" type="xsd:string" default="true">
  <xsd:annotation>
  <xsd:documentation>
<![CDATA[ 
```

指定会话是否应该被缓存。默认值为`true`。

```java
</xsd:documentation>
</xsd:annotation>
</xsd:attribute>
```

+   可以使用 XSD 引用进行的配置如下：

```java
<int-ftp:inbound-channel-adapter id="ftpInbound"
                 channel="ftpChannel" 
                 session-factory="ftpSessionFactory"
                 charset="UTF-8"
                 auto-create-local-directory="true"
                 delete-remote-files="true"
                 filename-pattern="*.txt"
                 remote-directory="some/remote/path"
                 local-directory=".">
  <int:poller fixed-rate="1000"/>
</int-ftp:inbound-channel-adapter>
```

# FTPSessionFactory 和 FTPSSessionFactory

在本节中，让我们看一下使用 Spring 集成的 FTP 的两个核心类`FTPSessionFactory`和`FTPSSessionFactory`。这些类有很多的 getter、setter 和实例变量，提供有关数据、文件和 FTP 模式的信息。实例变量及其用法如下所述：

类`org.springframework.integration.ftp.session.DefaultFtpSessionFactory`用于在应用程序中配置 FTP 详细信息。该类在配置 XML 文件中配置为一个简单的 bean。该类有以下的 getter 和 setter：

+   `Session`：这接受会话变量。

+   `postProcessClientAfterConnect`：这在执行客户端连接操作后处理额外的初始化。

+   `postProcessClientBeforeConnect`：这在执行客户端连接操作之前处理额外的初始化。

+   `BufferSize`：这定义了通过 FTP 传输的缓冲数据的大小。

+   `ClientMode`：FTP 支持两种模式。它们如下：

+   **主动 FTP 模式**：在 Spring FTP 集成包中指定为`ACTIVE_LOCAL_DATA_CONNECTION_MODE`。在主动 FTP 模式下，服务器必须确保随机端口`1023`<通信通道是打开的。在主动 FTP 模式下，客户端从一个随机的非特权端口（`N > 1023`）连接到 FTP 服务器的命令端口，端口`21`。然后，客户端开始监听端口`N + 1`并向 FTP 服务器发送 FTP 命令`PORT N + 1`。然后服务器将从其本地数据端口（端口`20`）连接回客户端指定的数据端口。

+   **被动 FTP 模式**：在 Spring FTP 集成包中指定为`PASSIVE_LOCAL_DATA_CONNECTION_MODE`。在被动 FTP 模式下，客户端同时启动到服务器的两个连接，解决了防火墙过滤来自服务器的传入数据端口连接到客户端的问题。在打开 FTP 连接时，客户端在本地打开两个随机的非特权端口（`N > 1023`和`N + 1`）。第一个端口在端口`21`上联系服务器，但是不是然后发出`PORT`命令并允许服务器连接回其数据端口，而是客户端将发出`PASV`命令。其结果是服务器随后打开一个随机的非特权端口（`P > 1023`）并在响应`PASV`命令中将`P`发送回客户端。然后客户端从端口`N + 1`到服务器上的端口`P`发起连接以传输数据。包`DefaultFTPClientFactory`具有一个设置器方法，其中有一个开关用于设置模式。

```java
**
  * Sets the mode of the connection. Only local modes are supported.
  */
  private void setClientMode(FTPClient client) {
    switch (clientMode ) {
      case FTPClient.ACTIVE_LOCAL_DATA_CONNECTION_MODE:
      client.enterLocalActiveMode();
      break;
      case FTPClient.PASSIVE_LOCAL_DATA_CONNECTION_MODE:
      client.enterLocalPassiveMode();
      break;
      default:
      break;
    }
  }
```

+   `Config`：这设置 FTP 配置对象`org.apache.commons.net.ftp.FTPClientConfig config`

+   `ConnectTimeout`：这指定了尝试连接到客户端后的连接超时时间。

+   `ControlEncoding`：这设置了编码。

+   `Data Timeout`：这设置了文件传输期间的数据超时时间。

+   `Default Timeout`：这设置了套接字超时时间。

+   `文件类型`：FTP 协议支持多种文件类型。它们列举如下：

+   **ASCII 文件类型（默认）**：文本文件以**网络虚拟终端**（**NVT**）ASCII 格式通过数据连接传输。这要求发送方将本地文本文件转换为 NVT ASCII，接收方将 NVT ASCII 转换为本地文本文件类型。每行的结尾使用 NVT ASCII 表示回车后跟换行。这意味着接收方必须扫描每个字节，寻找 CR，LF 对。（我们在第 15.2 节中看到了 TFTP 的 ASCII 文件传输中的相同情景。）

+   **EBCDIC 文件类型**：在两端都是**扩展二进制编码十进制交换码**（**EBCDIC**）系统时，传输文本文件的另一种方式。

+   **图像文件类型**：也称为二进制文件类型。数据以连续的位流发送，通常用于传输二进制文件。

+   **本地文件类型**：这是在不同字节大小的主机之间传输二进制文件的一种方式。发送方指定每字节的位数。对于使用 8 位的系统，具有 8 字节大小的本地文件类型等同于图像文件类型。我们应该知道 8 位等于 1 字节。

Spring 有抽象类`AbstractFtpSessionFactory<T extends org.apache.commons.net.ftp.FTPClient>`，其中定义了以下参数和静态变量，可以在 FTP 的配置中使用：

```java
public static final int ASCII_FILE_TYPE = 0;
public static final int EBCDIC_FILE_TYPE = 1;
public static final int BINARY_FILE_TYPE = 2;
public static final int LOCAL_FILE_TYPE = 3;
```

+   `Host`：指定 FTP 主机。

+   `Password`：指定 FTP 密码。

+   `Port`：指定 FTP 端口。有两个可用的端口，一个是数据端口，一个是命令端口。数据端口配置为 20，命令端口配置为 21。

+   `Username`：指定 FTP 用户名。

以下配置显示了`DefaultFtpSessionFactory`类作为一个 bean，其 bean ID 为`ftpClientFactory`，并且其属性值根据 FTP 服务器凭据进行设置：

```java
<bean id="ftpClientFactory" class="org.springframework.integration.ftp.session.DefaultFtpSessionFactory">
  <property name="host" value="localhost"/>
  <property name="port" value="22"/>
  <property name="username" value="anjana"/>
  <property name="password" value="raghu"/>
  <property name="clientMode" value="0"/>
  <property name="fileType" value="1"/>
</bean>
```

`org.springframework.integration.ftp.session.DefaultFtpsSessionFactory`类使我们能够使用 FTPS 连接。该类包含以下内容的 getter 和 setter：

+   `BufferSize`

+   `clientMode`

+   `config`

+   `ControlEncoding`

+   `DEFAULT_REMOTE_WORKING_DIRECTORY`

+   `fileType`

+   `host`

+   `password`

+   `port`

+   `username`

上述字段是从名为`AbstarctFtpSessionFactory`的抽象类继承的。

以下是`DefaultFtpsClientFactory`的示例 bean 配置及其可以在 XML 文件中配置的属性：

```java
<bean id="ftpClientFactory" class="org.springframework.integration.ftp.client.DefaultFtpsClientFactory">
  <property name="host" value="localhost"/>
  <property name="port" value="22"/>
  <property name="username" value="anju"/>
  <property name="password" value="raghu"/>
  <property name="clientMode" value="1"/>
  <property name="fileType" value="2"/>
  <property name="useClientMode" value="true"/>
  <property name="cipherSuites" value="a,b.c"/>
  <property name="keyManager" ref="keyManager"/>
  <property name="protocol" value="SSL"/>
  <property name="trustManager" ref="trustManager"/>
  <property name="prot" value="P"/>
  <property name="needClientAuth" value="true"/>
  <property name="authValue" value="anju"/>
  <property name="sessionCreation" value="true"/>
  <property name="protocols" value="SSL, TLS"/>
  <property name="implicit" value="true"/>
</bean>
```

# Spring FTP 使用出站通道示例

在本节中，让我们看一个简单的场景，将文件从 Location1 传输到远程位置 Location2。为了清晰起见，让我们定义它们如下：

+   Location1：`d:\folder1`

+   Location2：`d:\folder2`

让我们在 Spring 中使用 Spring 集成包创建一个简单的应用程序，以完成从 Location1 到 Location2 的文件传输任务。我们需要两个主要文件来完成这个任务；第一个是配置文件`applicationContext.xml`，第二个是一个 Java 类文件，它将通知 Spring 集成框架将文件上传到远程位置。

`applicationContext.xml`文件将包含整个必要的 bean 配置，以及使用 Spring 集成包所需的 XMLNS。需要集成的 XMLNS 如下：

```java

  xmlns:int-ftp="http://www.springframework.org/schema/integration/ftp"
```

我们还需要将`DefaultFtpSessionFactory`配置为一个 bean，其中包括`FtpChannel`和`FtpOutBoundAdpater`。`DefaultFtpSessionFactory`具有所有 FTP 属性的 setter。`FTPOutboundeAdapter`将配置为`remoteFTP`位置和`outboundchannel`。以下是完整的配置文件：

```java
<beans 

  xmlns:int-ftp="http://www.springframework.org/schema/integration/ftp"
  xsi:schemaLocation="http://www.springframework.org/schema/integration/ftp http://www.springframework.org/schema/integration/ftp/spring-integration-ftp.xsd
  http://www.springframework.org/schema/integration http://www.springframework.org/schema/integration/spring-integration.xsd
  http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="ftpClientFactory" class="org.springframework.integration.ftp.session.DefaultFtpSessionFactory">
    <property name="host" value="localhost"/>
    <property name="port" value="21"/>
    <property name="username" value="myftpusername"/>
    <property name="password" value="myftppassword"/>
    <property name="clientMode" value="0"/>
    <property name="fileType" value="2"/>
    <property name="bufferSize" value="100000"/>
  </bean>

  <int:channel id="ftpChannel" />

  <int-ftp:outbound-channel-adapter id="ftpOutbound"
                    channel="ftpChannel"
                    remote-directory="D:/folder2"
                    session-factory="ftpClientFactory"/>

</beans>
```

现在让我们创建一个简单的 Java 类，通知 Spring 将文件上传到 Location2。这个类将加载`applicationContext.xml`文件，并使用在 XML 文件中配置的 bean ID 实例化`FTPChannel`。创建一个文件对象，其中包含需要传输到远程位置的文件名。将这个文件对象发送到 Spring 集成消息，然后将消息发送到通道，以便文件被传送到目的地。以下是示例代码：

```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.integration.Message;
import org.springframework.integration.MessageChannel;
import org.springframework.integration.support.MessageBuilder;
import java.io.File;

public class SendFileSpringFTP {
  public static void main(String[] args) throws InterruptedException {
    ConfigurableApplicationContext ctx =
    new ClassPathXmlApplicationContext("/applicationContext.xml");
    MessageChannel ftpChannel = ctx.getBean("ftpChannel", MessageChannel.class);
    File file = new File("D:/folder2/report-Jan.txt");
    final Message<File> messageFile = MessageBuilder.withPayload(file).build();
    ftpChannel.send(messageFile);
    Thread.sleep(2000);
  }

}
```

运行上述类以查看`report-Jan.txt`被传输到远程位置。

## 配置 Spring FTP 以使用网关读取子文件夹中的文件

在这一节中，让我们看看另一个可以用来读取子文件夹报告的配置文件。

我们已经使用了上一节处理 FTP XSD 的表达式属性。我们将进一步看到如何使用表达式属性通知 Spring 集成 FTP 框架触发 FTP 命令。在 FTP 中执行的每个命令都会得到一个回复，通常是三位数，例如：

+   `125`：数据连接已打开；传输开始

+   `200`：命令 OK

+   `214`：帮助消息（供人类用户使用）

+   `331`：用户名正确；需要密码

+   `425`：无法打开数据连接

+   `452`：写文件错误

+   `500`：语法错误（无法识别的命令）

+   `501`：语法错误（无效参数）

+   `502`：未实现的模式类型

回复通道由网关创建。在以下代码中，我们为分割器配置了一个回复通道：

```java
<int-ftp:outbound-gateway id="gatewayLS" cache-sessions="false"
  session-factory="ftpSessionFactory"
  request-channel="inbound"
  command="ls"
  command-options="-1"
  expression="'reports/*/*'"
  reply-channel="toSplitter"/>

<int:channel id="toSplitter" />

<int:splitter id="splitter" input-channel="toSplitter" output-channel="toGet"/>

<int-ftp:outbound-gateway id="gatewayGET" cache-sessions="false"
  local-directory="localdir"
  session-factory="ftpSessionFactory"
  request-channel="toGet"
  reply-channel="toRemoveChannel"
  command="get"
  command-options="-P"
  expression="payload.filename"/>
```

使用 Spring 集成支持 FTP，我们还可以将消息分割成多个部分。这是在 XML 文件中使用`splitter`属性（`AbstractMessageSplitter implements MessageHandler`）进行配置的。

```java
<channel id="inputChannel"/>
<splitter id="splitter" 
  ref="splitterBean" 
  method="split" 
  input-channel="inputChannel" 
  output-channel="outputChannel" />
<channel id="outputChannel"/>
<beans:bean id="splitterBean" class="sample.PojoSplitter"/>
```

从逻辑上讲，`splitter`类必须分割消息并为每个分割消息附加序列号和大小信息，以便不丢失顺序。可以使用聚合器将断开的消息组合在一起，然后发送到通道。

## 在 Java 中配置 Spring FTP

在这一节中，让我们看看如何使用注解在 Java 类中配置 FTP 属性，并创建`DefaultFTPSession`工厂的实例，并使用实例可用的 setter 方法设置属性。

我们可以使用`@Configuration`注解来配置 FTP 属性，如下所示：

```java
import org.springframework.integration.file.remote.session.SessionFactory;
import org.springframework.integration.ftp.session.DefaultFtpSessionFactory;
@Configuration
public class MyApplicationConfiguration {
  @Autowired
  @Qualifier("myFtpSessionFactory")
  private SessionFactory myFtpSessionFactory;
  @Bean
  public SessionFactory myFtpSessionFactory()
  {
    DefaultFtpSessionFactory ftpSessionFactory = new DefaultFtpSessionFactory();
    ftpSessionFactory.setHost("ftp.abc.org");
    ftpSessionFactory.setClientMode(0);
    ftpSessionFactory.setFileType(0);
    ftpSessionFactory.setPort(21);
    ftpSessionFactory.setUsername("anjju");
    ftpSessionFactory.setPassword("raghu");
    return ftpSessionFactory;
  }

}
```

# 使用 Spring 集成发送文件到 FTP

想象一种情景，你正在通过 FTP 通道发送文件。假设有两个文件，比如`Orders.txt`和`vendors.txt`，需要通过 FTP 发送到远程位置。为了实现这一点，我们需要按照以下步骤进行操作：

1.  创建`FTPChannel`。

1.  使用`baseFolder.mkdirs()`在基本文件夹中创建一个目录。

1.  在基本文件夹位置创建两个文件对象。

1.  使用`InputStream`为订单和供应商创建两个单独的流。

1.  使用 Spring 中可用的文件工具，将输入流复制到它们各自的文件中。

1.  使用`MessageBuilder`类，使用`withpayload()`方法将文件转换为消息。

1.  最后，将消息发送到 FTP 通道并关闭上下文。

让我们写一些示例代码来做到这一点：

```java
public void sendFilesOverFTP() throws Exception{

  ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("META-INF/spring/integration/FtpOutboundChannelAdapterSample-context.xml");

  MessageChannel ftpChannel = ctx.getBean("ftpChannel", MessageChannel.class);

  baseFolder.mkdirs();
  final File fileToSendOrders = new File(baseFolder, "orders.txt");
  final File fileToSendVendors = new File(baseFolder, "vendore.txt");

  final InputStream inputStreamOrders = FtpOutboundChannelAdapterSample.class.getResourceAsStream("/test-files/orders.txt");
  final InputStream inputStreamVendors = FtpOutboundChannelAdapterSample.class.getResourceAsStream("/test-files/vendors.txt");
  FileUtils.copyInputStreamToFile(inputStreamOrders, fileToSendOrders);
  FileUtils.copyInputStreamToFile(inputStreamVendors, fileToSendVendors);
  assertTrue(fileToSendOrders.exists());
  assertTrue(fileToSendVendors.exists());
  final Message<File> messageOrders = MessageBuilder.withPayload(fileToSendOrders).build();
  final Message<File> messageVendors = MessageBuilder.withPayload(fileToSendVendors).build();
  ftpChannel.send(messageOrders);
  ftpChannel.send(messageVendors);
  Thread.sleep(2000);
  assertTrue(new File(TestSuite.FTP_ROOT_DIR + File.separator + "orders.txt").exists());
  assertTrue(new File(TestSuite.FTP_ROOT_DIR + File.separator + "vendors.txt").exists());
  LOGGER.info("Successfully transfered file 'orders.txt' and 'vendors.txt' to a remote FTP location.");
  ctx.close();
}
```

## 使用 Spring 集成和 Spring 批处理的 FTP 应用程序

在这一节中，我们将学习如何将 FTP 作为批处理作业。我们将在 Java 中创建一个配置文件，而不是 XML。在这里，我们将使用`@Configuration`注解为 Spring 批处理数据库和 tasklet 设置所有属性。然后我们有一个属性文件，它将为`ApplicationConfiguration.java`文件中的实例变量设置值。使用 Spring 框架中可用的属性持有者模式加载属性。

1.  我们首先要更新配置文件。以下是一个示例配置文件：

```java
@Configuration
public class ApplicationConfiguration {
  //Below is the set of instance variables that will be configured.
  //configuring the jdbc driver
  @Value("${batch.jdbc.driver}")
  private String driverClassName;
  //configuring the jdbc url
  @Value("${batch.jdbc.url}")
  private String driverUrl;

  //configuring the jdbc username
  @Value("${batch.jdbc.user}")
  private String driverUsername;

  //configuring the jdbc passowrd
  @Value("${batch.jdbc.password}")
  private String driverPassword;

  //configuring the jobrepository autowiring the bean
  @Autowired
  @Qualifier("jobRepository")
  private JobRepository jobRepository;

  //configuring the  ftpsessionfactory
  @Autowired
  @Qualifier("myFtpSessionFactory")
  private SessionFactory myFtpSessionFactory;

  @Bean
  public DataSource dataSource() {
    BasicDataSource dataSource = new BasicDataSource();
    dataSource.setDriverClassName(driverClassName);
    dataSource.setUrl(driverUrl);
    dataSource.setUsername(driverUsername);
    dataSource.setPassword(driverPassword);
    return dataSource;
  }
  //setting the ftp as a batch job
  @Bean
  @Scope(value="step")
  public FtpGetRemoteFilesTasklet myFtpGetRemoteFilesTasklet(){
    FtpGetRemoteFilesTasklet  ftpTasklet = new FtpGetRemoteFilesTasklet();
    ftpTasklet.setRetryIfNotFound(true);
    ftpTasklet.setDownloadFileAttempts(3);
    ftpTasklet.setRetryIntervalMilliseconds(10000);
    ftpTasklet.setFileNamePattern("README");
    //ftpTasklet.setFileNamePattern("TestFile");
    ftpTasklet.setRemoteDirectory("/");
    ftpTasklet.setLocalDirectory(new File(System.getProperty("java.io.tmpdir")));
    ftpTasklet.setSessionFactory(myFtpSessionFactory);

    return ftpTasklet;
  }
  //setting the  ftp sessionfactory

  @Bean
  public SessionFactory myFtpSessionFactory() {
    DefaultFtpSessionFactory ftpSessionFactory = new DefaultFtpSessionFactory();
    ftpSessionFactory.setHost("ftp.gnu.org");
    ftpSessionFactory.setClientMode(0);
    ftpSessionFactory.setFileType(0);
    ftpSessionFactory.setPort(21);
    ftpSessionFactory.setUsername("anonymous");
    ftpSessionFactory.setPassword("anonymous");

    return ftpSessionFactory;
  }

  //Configuring the simple JobLauncher
  @Bean
  public SimpleJobLauncher jobLauncher() {
    SimpleJobLauncher jobLauncher = new SimpleJobLauncher();
    jobLauncher.setJobRepository(jobRepository);
    return jobLauncher;
  }

  @Bean
  public PlatformTransactionManager transactionManager() {
    return new DataSourceTransactionManager(dataSource());
  }

}
```

1.  让我们使用`property-placeholder`进一步配置批处理作业。

1.  创建一个名为`batch.properties`的文件：

```java
batch.jdbc.driver=org.hsqldb.jdbcDriver
batch.jdbc.url=jdbc:hsqldb:mem:anjudb;sql.enforce_strict_size=true batch.jdbc.url=jdbc:hsqldb:hsql://localhost:9005/anjdb
batch.jdbc.user=anjana
batch.jdbc.password=raghu
```

1.  在`context.xml`文件或一个单独的文件中配置应用程序，以运行 FTP 的 tasklet：

```java
<batch:job id="ftpJob">
  <batch:step id="step1"  >
  <batch:tasklet ref="myApplicationFtpGetRemoteFilesTasklet" />
  </batch:step>
</batch:job>
```

1.  这里是`MyApplicationFtpGetRemoteFilesTasklet`：

```java
public class MyApplicationFtpGetRemoteFilesTasklet implements Tasklet, InitializingBean {
  private File localDirectory;
  private AbstractInboundFileSynchronizer<?> ftpInboundFileSynchronizer;
  private SessionFactory sessionFactory;
  private boolean autoCreateLocalDirectory = true;
  private boolean deleteLocalFiles = true;
  private String fileNamePattern;
  private String remoteDirectory;
  private int downloadFileAttempts = 12;
  private long retryIntervalMilliseconds = 300000;
  private boolean retryIfNotFound = false;
  /**All the above instance variables have setters and getters*/

  /*After properties are set it just checks for certain instance variables for null values and calls the setupFileSynchronizer method.
    It also checks for local directory if it doesn't exits it auto creates the local directory.
  */
  public void afterPropertiesSet() throws Exception {
    Assert.notNull(sessionFactory, "sessionFactory attribute cannot be null");
    Assert.notNull(localDirectory, "localDirectory attribute cannot be null");
    Assert.notNull(remoteDirectory, "remoteDirectory attribute cannot be null");
    Assert.notNull(fileNamePattern, "fileNamePattern attribute cannot be null");

    setupFileSynchronizer();

    if (!this.localDirectory.exists()) {
      if (this.autoCreateLocalDirectory) {
        if (logger.isDebugEnabled()) {
          logger.debug("The '" + this.localDirectory + "' directory doesn't exist; Will create.");
        }
        this.localDirectory.mkdirs();
      }
      else
      {
        throw new FileNotFoundException(this.localDirectory.getName());
      }
    }
  }
/*This method is called in afterpropertiesset() method. This method checks if we need to transfer files using FTP or SFTP.
If it is SFTP then it initializes ftpInbounFileSynchronizer using SFTPinbounfFileSynchronizer which has a constructor which takes sessionFactory as the argument and has setter method to set file Filter details with FileNamesPatterns.The method also sets the remoteDirectory location..
*/
  private void setupFileSynchronizer() {
    if (isSftp()) {
      ftpInboundFileSynchronizer = new SftpInboundFileSynchronizer(sessionFactory);
      ((SftpInboundFileSynchronizer) ftpInboundFileSynchronizer).setFilter(new SftpSimplePatternFileListFilter(fileNamePattern));
    }
    else
    {
      ftpInboundFileSynchronizer = new FtpInboundFileSynchronizer(sessionFactory);
      ((FtpInboundFileSynchronizer) ftpInboundFileSynchronizer).setFilter(new FtpSimplePatternFileListFilter(fileNamePattern));
    }
    ftpInboundFileSynchronizer.setRemoteDirectory(remoteDirectory);
  }
/*This method is called during the file synchronization process this will delete the files in the directory after copying..
*/
  private void deleteLocalFiles() {
    if (deleteLocalFiles) {
      SimplePatternFileListFilter filter = new SimplePatternFileListFilter(fileNamePattern);
      List<File> matchingFiles = filter.filterFiles(localDirectory.listFiles());
      if (CollectionUtils.isNotEmpty(matchingFiles)) {
        for (File file : matchingFiles) {
          FileUtils.deleteQuietly(file);
        }
      }
    }
  }
/*This is a batch execute method which operates with FTP ,it synchronizes the local directory with the remote directory.
*/
  /* (non-Javadoc)
  * @see org.springframework.batch.core.step.tasklet.Tasklet#execute(org.springframework.batch.core.StepContribution, org.springframework.batch.core.scope.context.ChunkContext)
  */
  public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
    deleteLocalFiles();

    ftpInboundFileSynchronizer.synchronizeToLocalDirectory(localDirectory);

    if (retryIfNotFound) {
      SimplePatternFileListFilter filter = new SimplePatternFileListFilter(fileNamePattern);
      int attemptCount = 1;
      while (filter.filterFiles(localDirectory.listFiles()).size() == 0 && attemptCount <= downloadFileAttempts) {
        logger.info("File(s) matching " + fileNamePattern + " not found on remote site.  Attempt " + attemptCount + " out of " + downloadFileAttempts);
        Thread.sleep(retryIntervalMilliseconds);
        ftpInboundFileSynchronizer.synchronizeToLocalDirectory(localDirectory);
        attemptCount++;
      }

      if (attemptCount >= downloadFileAttempts && filter.filterFiles(localDirectory.listFiles()).size() == 0) {
        throw new FileNotFoundException("Could not find remote file(s) matching " + fileNamePattern + " after " + downloadFileAttempts + " attempts.");
      }
    }

    return null;
  }
```

# 摘要

在本章中，我们看到了 FTP 及其缩写的概述。我们已经看到了不同类型的适配器，比如入站和出站适配器，以及出站网关及其配置。我们还展示了`springs-integration-ftp.xsd`，并引用了每个入站和出站适配器可用的各种选项。我们还展示了使用`spring-integration-ftp`包开发 maven 应用程序所需的库。然后我们看了两个重要的类，`FTPSessionFactory`和`FTPsSessionFactory`，以及它们的 getter 和 setter。我们还演示了使用出站通道的`SpringFTP`传输文件的示例。我们还演示了如何使用 Java 通过`@Configuration`注解配置 FTP。最后，我们演示了 FTP 作为一个 tasklet。在下一章中，我们将进一步探讨 Spring 与 HTTP 的集成。
