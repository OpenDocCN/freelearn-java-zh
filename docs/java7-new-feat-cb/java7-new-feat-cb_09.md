# 第九章：数据库、安全和系统增强

在本章中，我们将涵盖以下内容：

+   使用 RowSetFactory 类

+   Java 7 数据库增强

+   使用 ExtendedSSLSession 接口

+   使用平台 MXBeans 监视 JVM 或系统进程负载

+   重定向操作系统进程的输入和输出

+   在 HTML 页面中嵌入 JNLP 文件

# 介绍

本章涵盖了 Java 7 中对数据库、安全和系统类型的增强。其中一些增强较小，将在本介绍中进行讨论。其他一些增强更为重要，将在本章的配方中详细介绍。由于某些主题的专业性相当特殊，比如一些安全增强的特点，它们将被提及但不在此处解释。

Java 7 中对 JDBC 进行了多项增强，现在支持 JDBC 4.1。一些改进取决于早期驱动程序版本中不可用的第三方驱动程序支持。当发生这种情况时，您可能会收到`AbstractMethodException`。在测试本章的数据库配方时，请确保您使用支持 JDBC 4.1 功能的驱动程序。驱动程序可以在[`developers.sun.com/product/jdbc/drivers`](http://developers.sun.com/product/jdbc/drivers)找到。

*使用 RowSetFactory*配方涉及使用`javax.sql.rowset.RowSetFactory`接口和`javax.sql.rowset.RowSetProvider`类，允许根据给定的 JDBC 驱动程序创建任何行集。Java 7 中还包括数据库支持的其他改进。这些在*Java 7 数据库增强*配方中进行了讨论，包括确定当前模式的名称和提供对隐藏列的访问。Derby 数据库引擎将用于数据库示例。如果您希望使用其他数据库和表，可以通过调整不同数据库的代码来实现。

除了这些数据库配方之外，try-with-resource 语句可以与实现`java.sql`包的`Connection, ResultSet`或`Statement`接口的任何对象一起使用。这种语言改进简化了打开和关闭资源的过程。try-with-resource 语句的一般用法在第一章的*使用 try-with-resource 块改进异常处理代码*配方中进行了详细介绍，*Java 语言改进*。使用`ResultSet-derived`类的示例显示在*使用 RowSetFactory 类*配方中。

`Statement`接口已增强了两种新方法。第一种方法`closeOnCompletion`用于指定当使用连接的结果集关闭时，`Statement`对象将被关闭。第二种方法`isCloseOnCompletion`返回一个布尔值，指示在满足此条件时语句是否将被关闭。

Java 7 的网络增强包括向`java.net.URLClassLoader`类添加了两种方法：

+   `close:`此方法将关闭当前的`URLClassLoader`，使其无法再加载类或资源。这解决了 Windows 上发现的问题，详细信息请参阅[`download.oracle.com/javase/7/docs/technotes/guides/net/ClassLoader.html`](http://download.oracle.com/javase/7/docs/technotes/guides/net/ClassLoader.html)

+   `getResourceAsStream:`此方法返回由其`String`参数指定的资源的`InputStream`

还提供了支持使用 InfiniBand（IB）的流连接的帮助。这项技术使用远程直接内存访问（RDMA）直接在不同计算机的内存之间传输数据。这种支持是通过 Sockets Direct Protocol（SDP）网络协议提供的。这项技术的专业性使其无法进一步讨论。

*使用平台 MXBeans 监视 JVM 或系统进程负载*示例检查了对`MXBeans`支持的改进。这包括访问这些管理类型 bean 的不同方法。

`java.lang.ProcessBuilder`类通过`ProcessBuilder.Redirect`类引入了改进的重定向功能。这个主题在*重定向操作系统进程的输入和输出*示例中进行了探讨。

Java 7 还改进了 applet 嵌入 HTML 页面的方式。*在 HTML 页面中嵌入 JNLP 文件*示例演示了这种技术。

**Java Secure Socket Extension**（**JSSE**）用于使用**安全套接字层**（**SSL**）和**传输层安全性**（**TLS**）保护互联网通信。JSSE 有助于数据加密、身份验证和维护消息完整性。在 Java 7 中，发生了几项增强。*使用 ExtendedSSLSession 接口*示例使用 SSL，并用于说明如何使用`ExtendedSSLSession`接口和新的安全功能。

安全增强包括**椭圆曲线加密**（**ECC**）算法的整合。这类加密算法更抵抗暴力攻击。提供了算法的便携式实现。

还添加或增强了新的异常类以增强安全性。新的`java.security.cert.CertificateRevokedException`在抛出时表示**X.509**证书已被吊销。`java.security.cert.CertPathValidatorException`类通过添加一个接受`CertPathValidatorException.Reason`对象的新构造函数进行了增强。此对象实现了`CertPathValidatorException.BasicReason`枚举，列举了异常的原因。`CertPathValidatorException`类的`getReason`方法返回一个`CertPathValidatorException.Reason`对象。

Java 7 还支持 TLS 1.1 和 1.2 规范，并对此提供了改进支持。**Sun JSSE**提供程序支持 RFC 4346（[`tools.ietf.org/html/rfc4346`](http://tools.ietf.org/html/rfc4346)）和 RFC 5246（[`tools.ietf.org/html/rfc5246`](http://tools.ietf.org/html/rfc5246)）中定义的 TLS 1.1 和 TLS 1.2。这包括支持防范密码块链接攻击和新的加密算法。

此外，还有一些其他与 TKS 相关的增强：

+   **SSLv2Hello**协议已从默认启用的协议列表中删除。

+   Java 7 中已修复了与 TLS 重新协商相关的缺陷。有关此缺陷的详细信息，请参阅[`www.oracle.com/technetwork/java/javase/documentation/tlsreadme2-176330.html`](http://www.oracle.com/technetwork/java/javase/documentation/tlsreadme2-176330.html)。

+   在 TLS 1.1/1.2 握手期间，Java 7 改进了版本号检查的过程。

可以使用**Sun**提供程序的`jdk.certpath.disabledAlgorithms`属性来禁用弱加密算法。默认情况下，MD2 算法被禁用。此属性在`jre/lib/security/java.security`文件中指定。默认设置如下所示：

```java
jdk.certpath.disabledAlgorithms=MD2

```

还可以指定算法，还可以限制密钥大小。

算法限制也可以放置在 TLS 级别。这是通过`jre/lib/security/java.security`文件中的`jdk.tls.disabledAlgorithms`安全属性来实现的。示例如下：

```java
jdk.tls.disabledAlgorithms=MD5, SHA1, RSA keySize < 2048

```

目前，此属性仅适用于**Oracle JSSE**实现，可能不被其他实现所识别。

**服务器名称指示**（**SNI**）JSSE 扩展（RFC 4366）使 TLS 客户端能够连接到虚拟服务器，即使用相同支持网络地址的不同网络名称的多个服务器。这在默认情况下设置为`true`，但可以在不支持该扩展的系统上设置为`false`。

`jsse.enableSNIExtension`系统属性用于控制此设置。可以使用如下所示的`-D`java 命令选项进行设置：

```java
java -D jsse.enableSNIExtension=true ApplicationName

```

还可以使用如下所示的`setProperty`方法设置此属性：

```java
System.setProperty("jsse.enableSNIExtension", "true");

```

请注意，属性名称可能会在将来更改。

# 使用`RowSetFactory`类

现在可以使用新的`javax.sql.rowset`包的`RowSetFactoryInterface`接口和`RowSetProvider`类来创建行集。这允许创建 JDBC 支持的任何类型的行集。我们将使用 Derby 数据库来说明创建行集的过程。将使用`COLLEAGUES`表。如何创建此表的说明可在[`netbeans.org/kb/docs/ide/java-db.html`](http://netbeans.org/kb/docs/ide/java-db.html)找到。创建表的 SQL 代码如下：

```java
CREATE TABLE COLLEAGUES (
"ID" INTEGER not null primary key,
"FIRSTNAME" VARCHAR(30),
"LASTNAME" VARCHAR(30),
"TITLE" VARCHAR(10),
"DEPARTMENT" VARCHAR(20),
"EMAIL" VARCHAR(60)
);
INSERT INTO COLLEAGUES VALUES (1,'Mike','Johnson','Manager','Engineering','mike.johnson@foo.com');
INSERT INTO COLLEAGUES VALUES
(2, 'James', 'Still', 'Engineer', 'Engineering', 'james.still@foo.com');
INSERT INTO COLLEAGUES VALUES
(3, 'Jerilyn', 'Stall', 'Manager', 'Marketing', 'jerilyn.stall@foo.com');
INSERT INTO COLLEAGUES VALUES
(4, 'Jonathan', 'Smith', 'Manager', 'Marketing', 'jonathan.smith@foo.com');

```

## 准备工作

创建新的行集：

1.  创建`RowSetFactory`的实例。

1.  使用几种`create`方法之一来创建`RowSet`对象。

## 如何做...

1.  创建一个新的控制台应用程序。在`main`方法中，添加以下代码序列。我们将创建一个新的`javax.sql.rowset.JdbcRowSet`对象，并使用它来显示`COLLEAGUES`表中的一些字段。首先设置`String`变量以建立与数据库的连接，并创建`RowSetFactory`对象如下：

```java
String databaseUrl = "jdbc:derby://localhost:1527/contact";
String username = "userName";
String password = "password";
RowSetFactory rowSetFactory = null;
try {
rowSetFactory = RowSetProvider.newFactory("com.sun.rowset.RowSetFactoryImpl", null);
}
catch (SQLException ex) {
ex.printStackTrace();
return;
}

```

1.  接下来，添加一个 try 块来捕获任何`SQLExceptions`，然后使用`createJdbcRowSet`方法创建行集。接下来，显示表的选定元素。

```java
try (JdbcRowSet rowSet = rowSetFactory.createJdbcRowSet();) {
rowSet.setUrl(databaseUrl);
rowSet.setUsername(username);
rowSet.setPassword(password);
rowSet.setCommand("SELECT * FROM COLLEAGUES");
rowSet.execute();
while (rowSet.next()) {
System.out.println(rowSet.getInt("ID") + " - "
+ rowSet.getString("FIRSTNAME"));
}
}
catch (SQLException ex) {
ex.printStackTrace();
}

```

1.  执行应用程序。输出应如下所示：

**1 - Mike**

**2 - James**

**3 - Jerilyn**

**4 - Jonathan**

## 工作原理...

为数据库 URL、用户名和密码创建了字符串变量。使用静态的`newFactory`方法创建了`RowSetFactory`对象。任何生成的异常都将导致应用程序终止。

在 try-with-resources 块中，使用`createJdbcRowSet`方法创建了`JdbcRowSet`类的实例。然后将 URL、用户名和密码分配给行集。选择命令从`COLLEAGUES`表中检索所有字段。然后执行查询。

接下来，使用`while`循环显示了行集的每一行的 ID 和名字。

## 还有更多...

可能有多个可用的`RowSetFactory`实现。`newFactory`方法将按以下顺序查找`RowSetFactory`类：

1.  如果定义了系统属性`javax.sql.rowset.RowSetFactory`中指定的。

1.  使用`ServiceLoader` API。

1.  平台默认实例。

除了创建`JdbcRowSet`行集之外，还有其他方法可用于创建不同类型的行集，如下表所示：

| 方法 | 创建的行集 |
| --- | --- |
| `createCachedRowSet` | `CachedRowSet` |
| `createFilteredRowSet` | `FilteredRowSet` |
| `createJdbcRowSet` | `JdbcRowSet` |
| `createJoinRowSet` | `JoinRowSet` |
| `createWebRowSet` | `WebRowSet` |

还可以使用带有两个参数的重载的`newFactory`方法创建`RowSetFactory`，如下所示：

```java
rowSetFactory = RowSetProvider.newFactory("com.sun.rowset.RowSetFactoryImpl", null);

```

这种方法为应用程序提供了更多的控制，使其能够指定要使用的提供程序。当类路径中有多个提供程序时，这可能很有用。第一个参数指定提供程序的类名，第二个参数指定要使用的类加载器。将`null`用作第二个参数指定要使用上下文类加载器。

# Java 7 数据库增强

Java 7 提供了对数据库支持的许多小的增强。本示例介绍了这些增强，并在实际情况下提供了示例。由于许多 JDBC 4.1 驱动程序的不成熟，不是所有的代码示例都能完全正常运行。

## 准备工作

大多数示例都是从以下开始：

1.  创建 Derby 数据库的连接。

1.  使用连接方法访问所需功能。

## 如何做...

1.  创建一个新的控制台应用程序。在`main`方法中，添加以下代码序列。它将建立与数据库的连接，并确定自动生成的键是否总是被返回，以及当前模式是什么：

```java
try {
Connection con = DriverManager.getConnection(
"jdbc:derby://localhost:1527/contact", "userName", "password");
System.out.println("Schema: " + con.getSchema());
System.out.println("Auto Generated Keys: " + metaData.generatedKeyAlwaysReturned());
}
catch (SQLException ex) {
ex.printStackTrace();
}

```

1.  执行时，输出应类似于以下内容：

自动生成的键：true

**模式：SchemaName**

## 工作原理...

`Statement`接口的`getGeneratedKeys`方法是在 Java 1.4 中引入的，用于返回该语句的任何自动生成的键。`java.sql.DatabaseMetaData`接口的`generatedKeyAlwaysReturned`方法返回一个布尔值，指示自动生成的键将始终被返回。

可以使用`Connection`接口的`setSchema`和`getSchema`方法来设置和获取连接的模式。执行了`getSchema`方法，返回了模式名称。

## 还有更多...

其他三个主题需要进一步讨论：

+   检索伪列

+   控制`OUT`参数的类型值

+   其他数据库增强

### 检索伪列

数据库通常会使用隐藏列来表示表的每一行的唯一键。这些隐藏列有时被称为**伪列**。在 Java 7 中，已添加了两种新方法来处理伪列。`DatabaseMetaData`接口的`getPseudoColumns`方法将检索一个`ResultSet`。该方法要求以下内容：

+   目录：这需要与数据库中使用的目录名称匹配。如果不使用目录，则使用空字符串。空值表示在搜索列时不使用目录名称。

+   模式名称：这需要与数据库中使用的模式名称匹配。如果不使用模式，则使用空字符串。空值表示在搜索列时不使用模式名称。

+   表名称模式：这需要与数据库中使用的表名称匹配

+   列名称模式：这需要与数据库中使用的列名称匹配

返回的`ResultSet`将按照以下表格所示的组织结构：

| 列 | 类型 | 意义 |
| --- | --- | --- |
| `TABLE_CAT` | 字符串 | 可能为空的目录名称 |
| `TABLE_SCHEM` | 字符串 | 可能为空的模式名称 |
| `TABLE_NAME` | 字符串 | 表的名称 |
| `COLUMN_NAME` | 字符串 | 列的名称 |
| `DATA_TYPE` | 整数 | SQL 类型（`java.sql.Types`） |
| `COLUMN_SIZE` | 整数 | 列的大小 |
| `DECIMAL_DIGITS` | 整数 | 小数位数。空值表示没有小数位数。 |
| `NUM_PREC_RADIX` | 整数 | 基数 |
| `COLUMN_USAGE` | 字符串 | 指定列的使用方式，由新的 PsuedoColumnUsage 枚举定义 |
| `REMARKS` | 字符串 | 关于列的评论 |
| `CHAR_OCTET_LENGTH` | 整数 | char 列的最大字符数 |
| `IS_NULLABLE` | 字符串 | *YES: 列可以包含空值**NO: 列不能包含空值**"": 未知* |

隐藏列表示一个唯一键，提供了一种快速访问行的方式。Derby 不支持隐藏列。但是，以下代码序列说明了如何实现这一点：

```java
try {
Connection con = DriverManager.getConnection(
"jdbc:derby://localhost:1527/contact", "userName", "password");
DatabaseMetaData metaData = con.getMetaData();
ResultSet resultSet = metaData.getPseudoColumns("", "schemaName", "tableName", "");
while (rs.next()) {
System.out.println(
resultSet.getString("TABLE_SCHEM ")+" - "+
resultSet.getString("COLUMN_NAME "));
}
}
catch (SQLException ex) {
ex.printStackTrace();
}

```

Derby 将返回一个空的`ResultSet`，其中包含先前列出的列。

### 控制`OUT`参数的类型值

`java.sql.CallableStatement`有两个重载的`getObject`方法，返回一个给定列名或索引的对象。目前支持有限。但是，基本方法如下所示：

```java
try {
Connection conn = DriverManager.getConnection(
"...", "username", "password");
String query = "{CALL GETDATE(?,?)}";
CallableStatement callableStatement = (CallableStatement) conn.prepareCall(query);
callableStatement.setInt(1,recordIdentifier);
callableStatement.registerOutParameter(1, Types.DATE);
callableStatement.executeQuery();
date = callableStatement.getObject(2,Date.class));
}
catch (SQLException ex) {
ex.printStackTrace();
}

```

查询字符串包含对存储过程的调用。假定该存储过程使用整数值作为第一个参数来标识表中的记录。第二个参数将被返回，并且是`Date`类型。

一旦查询被执行，`getObject`方法将使用指定的数据类型返回指定的列。该方法将把 SQL 类型转换为 Java 数据类型。

### 其他数据库增强

`java.sql`包的`Driver`接口有一个新方法，返回驱动程序的父记录器。下面的代码序列说明了这一点：

```java
try {
Driver driver = DriverManager.getDriver("jdbc:derby://localhost:1527");
System.out.println("Parent Logger" + driver.getParentLogger());
}
catch (SQLException ex) {
ex.printStackTrace();
}

```

但是，当执行时，当前版本的驱动程序将生成以下异常：

**Java.sql.SQLFeatureNotSupportedException: Feature not implemented: getParentLogger**。

Derby 不使用`java.util.logging`包，因此会抛出此异常。`javax.sql.CommonDataSource`接口还添加了`getParentLogger`方法。

此外，当一系列数据库操作与`Executor`一起执行时，有三种方法可用于支持这些操作，如下所示：

+   `abort:`此方法将使用传递给方法的`Executor`中止打开的连接

+   `setNetworkTimeout:`此方法指定等待响应的超时时间（以毫秒为单位）。它还使用一个`Executor`对象。

+   `getNetworkTimeout:`此方法返回连接等待数据库请求的毫秒数

最后两个方法是可选的，Derby 不支持它们。

# 使用`ExtendedSSLSession`接口

`javax.net.ssl`包提供了一系列用于实现安全套接字通信的类。Java 7 中引入的改进包括添加了`ExtendedSSLSession`接口，该接口可用于确定所使用的特定本地和对等支持的签名算法。此外，创建`SSLSession`时，可以使用端点识别算法来确保主机计算机的地址与证书的地址匹配。这个算法可以通过`SSLParameters`类访问。

## 准备工作

为了演示`ExtendedSSLSession`接口的使用，我们将：

1.  创建一个基于`SSLServerSocket`的`EchoServer`应用程序，以接受来自客户端的消息。

1.  创建一个客户端应用程序，该应用程序使用`SSLSocket`实例与服务器通信。

1.  使用`EchoServer`应用程序获取`ExtendedSSLSession`接口的实例。

1.  使用`SimpleConstraints`类来演示算法约束的使用。

## 如何做...

1.  让我们首先创建一个名为`SimpleConstraints`的类，该类改编自**Java PKI 程序员指南**([`download.oracle.com/javase/7/docs/technotes/guides/security/certpath/CertPathProgGuide.html`](http://download.oracle.com/javase/7/docs/technotes/guides/security/certpath/CertPathProgGuide.html))。我们将使用这个类来将算法约束关联到应用程序。将以下类添加到您的项目中：

```java
public class SimpleConstraints implements AlgorithmConstraints {
public boolean permits(Set<CryptoPrimitive> primitives,
String algorithm, AlgorithmParameters parameters) {
return permits(primitives, algorithm, null, parameters);
}
public boolean permits(Set<CryptoPrimitive> primitives, Key key) {
return permits(primitives, null, key, null);
}
public boolean permits(Set<CryptoPrimitive> primitives,
String algorithm, Key key, AlgorithmParameters parameters) {
if (algorithm == null) algorithm = key.getAlgorithm();
if (algorithm.indexOf("RSA") == -1) return false;
if (key != null) {
RSAKey rsaKey = (RSAKey)key;
int size = rsaKey.getModulus().bitLength();
if (size < 2048) return false;
}
return true;
}
}

```

1.  创建`EchoServer`应用程序，创建一个新的控制台应用程序。将以下代码添加到`main`方法中。在这个初始序列中，我们创建并启动服务器：

```java
try {
SSLServerSocketFactory sslServerSocketFactory =
(SSLServerSocketFactory) SSLServerSocketFactory.getDefault();
SSLServerSocket sslServerSocket =
(SSLServerSocket) sslServerSocketFactory.createServerSocket(9999);
System.out.println("Waiting for a client ...");
SSLSocket sslSocket = (SSLSocket) sslServerSocket.accept();
}
catch (Exception exception) {
exception.printStackTrace();
}

```

1.  接下来，添加以下代码序列以设置应用程序的算法约束。它还返回端点算法的名称：

```java
SSLParameters parameters = sslSocket.getSSLParameters();
parameters.setAlgorithmConstraints (new SimpleConstraints());
String endPoint = parameters.getEndpointIdentificationAlgorithm();
System.out.println("End Point: " + endPoint);

```

1.  添加以下代码以显示本地支持的算法：

```java
System.out.println("Local Supported Signature Algorithms");
if (sslSocket.getSession() instanceof ExtendedSSLSession) {
ExtendedSSLSession extendedSSLSession =
(ExtendedSSLSession) sslSocket.getSession();
ExtendedSSLSession interfaceusingString algorithms[] =
extendedSSLSession.getLocalSupportedSignatureAlgorithms();
for (String algorithm : algorithms) {
System.out.println("Algorithm: " + algorithm);
}
}

```

1.  以下序列显示了对等支持的算法：

```java
System.out.println("Peer Supported Signature Algorithms");
if (sslSocket.getSession() instanceof ExtendedSSLSession) {
String algorithms[] = ((ExtendedSSLSession) sslSocket.getSession()).getPeerSupportedSignatureAlgorithms();
for (String algorithm : algorithms) {
System.out.println("Algorithm: " + algorithm);
}
}

```

1.  添加以下代码来缓冲来自客户端应用程序的输入流：

```java
InputStream inputstream = sslSocket.getInputStream();
InputStreamReader inputstreamreader = new InputStreamReader(inputstream);
BufferedReader bufferedreader = new BufferedReader (inputstreamreader);

```

1.  通过添加代码显示来自客户端的输入来完成该方法：

```java
String stringline = null;
while ((stringline = bufferedreader.readLine()) != null) {
System.out.println(string);
System.out.flush();
}

```

1.  要执行服务器，我们需要创建密钥库。这可以通过在命令提示符中执行以下命令来完成：

```java
keytool -genkey -keystore mySrvKeystore -keyalg RSA

```

1.  提供程序请求的密码和其他信息。接下来，转到回声服务器的位置并输入以下命令：

```java
java -Djavax.net.ssl.keyStore=mySrvKeystore
Djavax.net.ssl.keyStorePassword=password package.EchoServer

```

1.  上面的**密码**是您用来创建密钥库的密码，而**package**是您的 EchoServer 的包（如果有的话）。当程序执行时，您会得到以下输出：

**等待客户端...**

1.  现在我们需要创建一个名为`EchoClient`的客户端控制台应用程序。在`main`方法中，添加以下代码，我们创建与服务器的连接，然后将键盘输入发送到服务器：

```java
try {
SSLSocketFactory sslSocketFactory =
(SSLSocketFactory) SSLSocketFactory.getDefault();
SSLSocket sslSocket = (SSLSocket)
sslSocketFactory.createSocket("localhost", 9999);
InputStreamReader inputStreamReader =
new InputStreamReader(System.in);
BufferedReader bufferedReader =
new BufferedReader(inputStreamReader);
OutputStream outputStream = sslSocket.getOutputStream();
OutputStreamWriter outputStreamWriter =
new OutputStreamWriter(outputStream);
BufferedWriter bufferedwriter =
new BufferedWriter(outputStreamWriter);
String line = null;
while ((line = bufferedReader.readLine()) != null) {
ExtendedSSLSession interfaceusingbufferedwriter.write(line + '\n');
bufferedwriter.flush();
}
}
catch (Exception exception) {
exception.printStackTrace();
}

```

1.  将密钥库文件复制到客户端应用程序的目录中。在单独的命令窗口中，执行以下命令：

```java
java -Djavax.net.ssl.trustStore=mySrvKeystore
-Djavax.net.ssl.trustStorePassword=password package.EchoClient

```

1.  上面的**密码**是您用来创建密钥库的密码，而**package**是您的 EchoServer 的包（如果有的话）。程序执行时，输入单词**cat**，然后按*Enter*键。在服务器命令窗口中，您应该看到一个终点名称，可能为空，一个本地支持的签名算法列表，以及类似以下内容的**cat**：

**终点：null**

**本地支持的签名算法**

**算法：SHA512withECDSA**

**算法：SHA512withRSA**

**算法：SHA384withECDSA**

**算法：SHA384withRSA**

**算法：SHA256withECDSA**

**算法：SHA256withRSA**

**算法：SHA224withECDSA**

**算法：SHA224withRSA**

**算法：SHA1withECDSA**

**算法：SHA1withRSA**

**算法：SHA1withDSA**

**算法：MD5withRSA**

**对等支持的签名算法**

**cat**

1.  当您输入更多的输入行时，它们应该在服务器命令窗口中反映出来。要终止程序，在客户端命令窗口中输入*Ctrl* + *C*。

## 它是如何工作的...

`SimpleConstraints`类只允许 RSA 算法，然后使用 2048 位或更多的密钥。这被用作`setAlgorithmConstraints`方法的参数。该类实现了`java.security.AlgorithmConstraints`接口，表示算法的限制。

首先创建一个`SSLServerSocketFactory`实例，然后创建一个`SSLServerSocket`。对套接字执行`accept`方法，该方法会阻塞，直到客户端连接到它。

接下来设置了`SimpleConstraints`，然后使用了`getEndpointIdentificationAlgorithm`方法，返回了一个空字符串。在这个例子中，没有使用终点识别算法。

列出了本地和对等支持的签名算法。剩下的代码涉及读取并显示客户端发送的字符串。

`EchoClient`应用程序更简单。它创建了`SSLSocket`类的一个实例，然后使用它的`getOutputStream`方法将用户的输入写入回显服务器。

# 使用平台 MXBeans 进行 JVM 或系统进程负载监控

**Java 管理扩展**（**JMX**）是一种向应用程序添加管理接口的标准方式。**托管 bean**（**MBean**）为应用程序提供管理服务，并向`javax.management.MBeanServer`注册，该服务器保存和管理 MBean。`javax.management.MXBean`是一种 MBean 类型，允许客户端访问 bean 而无需访问特定类。

`java.lang.management`包的`ManagementFactory`类添加了几种新方法来访问 MBean。然后可以用这些方法来访问进程和负载监控。

## 准备就绪

访问`MXBean`：

1.  使用`getPlatformMXBean`方法和应用程序所需的`MXBean`类型。

1.  根据需要使用`MXBean`方法。

## 如何做...

1.  创建一个新的控制台应用程序。使用以下`main`方法。在这个应用程序中，我们将获取运行时环境的`MXBean`并显示关于它的基本信息：

```java
public static void main(String[] args) {
RuntimeMXBean mxBean = ManagementFactory.getPlatformMXBean(RuntimeMXBean.class);
System.out.println("JVM Name: " + mxBean.getName());
System.out.println("JVM Specification Name: " + mxBean.getSpecName());
System.out.println("JVM Specification Version: " + mxBean.getSpecVersion());
System.out.println("JVM Implementation Name: " + mxBean.getVmName());
System.out.println("JVM Implementation Vendor: " + mxBean.getVmVendor());
System.out.println("JVM Implementation Version: " + mxBean.getVmVersion());
}

```

1.  执行应用程序。您的输出应该类似于以下内容：

**JVM 名称：5584@name-PC**

**JVM 规范名称：Java 虚拟机规范**

**JVM 规范版本：1.7**

**JVM 实现名称：Java HotSpot(TM) 64 位服务器 VM**

**JVM 实现供应商：Oracle Corporation**

**JVM 实现版本：21.0-b17**

## 它是如何工作的...

我们使用了`ManagementFactory`类的静态`getPlatformMXBean`方法，参数为`RuntimeMXBean.class`。这返回了一个`RuntimeMXBean`的实例。然后应用了该实例的特定方法，并显示了它们的值。

## 还有更多...

`ManagementFactory`在 Java 7 中引入了几种新方法：

+   `getPlatformMXBean:` 这是一个重载的方法，它返回一个支持特定管理接口的`PlatformManagedObject`派生对象，使用`Class`参数

+   `getPlatformMXBeans:` 这是一个重载的方法，它返回一个支持特定管理接口的`PlatformManagedObject`派生对象，使用`MBeanServerConnection`对象和一个`Class`参数

+   `getPlatformManagementInterfaces:` 该方法返回当前 Java 平台上的`PlatformManagedObject`派生对象的`Class`对象集

此外，`java.lang.management`包中添加了一个新的接口。`PlatformManagedObject`接口用作所有`MXBeans`的基本接口。

### 使用`getPlatformMXBeans`方法

`getPlatformMXBeans`方法传递`MXBean`类型并返回实现`MXBean`类型的平台`MXBeans`列表。在下面的示例中，我们获取了`OperatingSystemMXBean`的列表。然后显示了`MXBean`的几个属性：

```java
List<OperatingSystemMXBean> list =
ManagementFactory.getPlatformMXBeans(OperatingSystemMXBean.class);
for (OperatingSystemMXBean bean : list) {
System.out.println("Operating System Name: " + bean.getName());
System.out.println("Operating System Architecture: " + bean.getArch());
System.out.println("Operating System Version: " + bean.getVersion());
}

```

执行时，您应该获得类似以下的输出。确切的输出取决于用于执行应用程序的操作系统和硬件：

**操作系统名称：Windows 7**

**操作系统架构：amd64**

**操作系统版本：6.1**

### 获取平台的管理接口

`ManagementFactory`类的静态`getPlatformManagementInterfaces`方法返回表示平台支持的`MXBeans`的`Class`对象集。然而，在运行 JDK 7.01 版本时，该方法在 Windows 7 和 Ubuntu 平台上都生成了`ClassCastException`。未来的版本应该纠正这个问题。

作为 JDK 的一部分提供的**jconsole**应用程序，提供了一种确定可用的`MXBeans`的替代技术。以下是控制台显示操作系统属性，特别是`ProcessCpuLoad`属性：

![获取平台的管理接口](img/5627_09_01.jpg)

# 重定向操作系统进程的输入和输出

`java.lang.ProcessBuilder`类有几个有用于重定向从 Java 应用程序执行的外部进程的输入和输出的新方法。嵌套的`ProcessBuilder.Redirect`类已被引入以提供这些额外的重定向功能。为了演示这个过程，我们将从文本文件向 DOS 提示符发送命令行参数，并将输出记录在另一个文本文件中。

## 准备就绪

为了控制外部进程的输入和输出，您必须：

1.  创建一个新的`ProcessBuilder`对象。

1.  将进程的输入和输出定向到适当的位置。

1.  通过`start`方法执行进程。

## 操作步骤…

1.  首先，创建一个新的控制台应用程序。创建三个新的文件实例来表示我们的进程执行涉及的三个文件：输入，输出和错误，如下所示：

```java
File commands = new File("C:/Projects/ProcessCommands.txt");
File output = new File("C:/Projects/ProcessLog.txt");
File errors = new File("C:/Projects/ErrorLog.txt");

```

1.  使用指定文件的路径创建文件`ProcessCommands.txt`并输入以下文本：

**cd C:\**

**dir**

**mkdir "Test Directory"**

**dir**

1.  确保在最后一行之后有一个回车。

1.  接下来，创建一个`ProcessBuilder`的新实例，将字符串`"cmd"`传递给构造函数，以指定我们要启动的外部进程，即操作系统命令窗口。调用`redirectInput, redirectOutput`和`redirectError`方法，不带参数，并打印出默认位置：

```java
ProcessBuilder pb = new ProcessBuilder("cmd");
System.out.println(pb.redirectInput());
System.out.println(pb.redirectOutput());
System.out.println(pb.redirectError());

```

1.  然后，我们想调用前面方法的重载形式，将各自的文件传递给每个方法。再次调用每个方法的无参数形式，使用`toString`方法来验证 IO 源是否已更改：

```java
pb.redirectInput(commands);
pb.redirectError(errors);
pb.redirectOutput(output);
System.out.println(pb.redirectInput());
System.out.println(pb.redirectOutput());
System.out.println(pb.redirectError());

```

1.  最后，调用`start`方法来执行进程，如下所示：

```java
pb.start();

```

1.  运行应用程序。您应该看到类似以下的输出：

**PIPE**

**PIPE**

**PIPE**

**重定向以从文件"C:\Projects\ProcessCommands.txt"读取**

重定向以写入文件"C:\Projects\ProcessLog.txt"

重定向以写入文件"C:\Projects\ErrorLog.txt"

1.  检查每个文本文件。您的输出文件应该有类似于以下文本：

Microsoft Windows [版本 6.7601]

版权所有(c)2009 年微软公司。保留所有权利。

C:\Users\Jenn\Documents\NetBeansProjects\ProcessBuilderExample>cd C:\

C:\>dir

驱动器 C 中没有标签的卷。

卷序列号为 927A-1F77

C:\的目录

03/05/2011 10:56 <DIR> 戴尔

11/08/2011 16:04 <DIR> 其他

11/08/2011 11:08 <DIR> 移动

10/31/2011 10:57 <DIR> 音乐

11/08/2011 19:44 <DIR> 项目

10/27/2011 21:09 <DIR> 临时

10/28/2011 10:46 <DIR> 用户

11/08/2011 17:11 <DIR> 窗户

0 个文件 0 字节

34 个目录 620,819,542,016 字节可用

在 C:\>中创建"测试目录"

C:\>dir

驱动器 C 中没有标签的卷。

卷序列号为 927A-1F77

C:\的目录

03/05/2011 10:56 <DIR> 戴尔

11/08/2011 16:04 <DIR> 其他

11/08/2011 11:08 <DIR> 移动

10/31/2011 10:57 <DIR> 音乐

11/08/2011 19:44 <DIR> 项目

10/27/2011 21:09 <DIR> 临时

10/28/2011 10:46 <DIR> 测试目录

10/28/2011 10:46 <DIR> 用户

11/08/2011 17:11 <DIR> 窗户

1.  再次执行程序并检查您的错误日志的内容。因为您的测试目录已经在第一次进程执行时创建，所以现在应该看到以下错误消息：

子目录或文件"测试目录"已经存在。

## 它是如何工作的...

我们创建了三个文件来处理我们进程的输入和输出。当我们创建`ProcessBuilder`对象的实例时，我们指定要启动的应用程序为命令窗口。在应用程序中执行操作所需的信息存储在我们的输入文件中。

当我们首次调用`redirectInput, redirectOutput`和`redirectError`方法时，我们没有传递任何参数。这些方法都返回一个`ProcessBuilder.Redirect`对象，我们打印了它。这个对象代表默认的 IO 源，在所有三种情况下都是`Redirect.PIPE`，`ProcessBuilder.Redirect.Type`枚举值之一。管道将一个源的输出发送到另一个源。

我们使用的方法的第二种形式涉及将`java.io.File`实例传递给`redirectInput, redirectOutput`和`redirectError`方法。这些方法也返回一个`ProcessBuilder`对象，但它们还具有设置 IO 源的功能。在我们的示例中，我们再次调用了每种方法的无参数形式，以验证 IO 是否已被重定向。

程序第一次执行时，您的错误日志应该是空的，假设您为每个`File`对象使用了有效的文件路径，并且您在计算机上有写权限。第二次执行旨在显示如何将错误捕获定向到单独的文件。如果未调用`redirectError`方法，错误将继承标准位置，并将显示在 IDE 的输出窗口中。有关继承标准 IO 位置的信息，请参阅*还有更多..*部分。

重要的是要注意，必须在重定向方法之后调用`start`方法。在重定向输入或输出之前启动进程将导致进程忽略您的重定向，并且应用程序将使用标准 IO 位置执行。

## 还有更多...

在本节中，我们将研究`ProcessBuilder.Redirect`类和`inheritIO`方法的使用。

### 使用 ProcessBuilder.Redirect 类

`ProcessBuilder.Redirect`类提供了另一种指定 IO 数据重定向的方法。使用前面的示例，在调用`start`方法之前添加一行：

```java
pb.redirectError(Redirect.appendTo(errors));

```

这种`redirectError`方法的形式允许你指定错误应该追加到错误日志文本文件中，而不是覆盖。如果你使用这个改变来执行应用程序，当进程再次尝试创建`Test Directory`目录时，你会看到错误的两个实例：

**子目录或文件 Test Directory 已经存在**。

**子目录或文件 Test Directory 已经存在**。

这是使用`redirectError`方法的重载形式的一个例子，传递了一个`ProcessBuilder.Redirect`对象而不是一个文件。所有三种方法，`redirectError, redirectInput`和`redirectOutput`，都有这种重载形式。

`ProcessBuilder.Redirect`类有两个特殊值，即`Redirect.PIPE`和`Redirect.INHERIT。Redirect.PIPE`是处理外部进程 IO 的默认方式，简单地意味着 Java 进程将通过管道连接到外部进程。`Redirect.INHERIT`值意味着外部进程将具有与当前 Java 进程相同的输入或输出位置。你也可以使用`Redirect.to`和`Redirect.from`方法重定向数据的输入或输出。

### 使用 inheritIO 方法继承默认的 IO 位置

如果你从 Java 应用程序执行外部进程，你可以设置源和目标数据的位置与当前 Java 进程的位置相同。`ProcessBuilder`类的`inheritIO`方法是实现这一点的一种便捷方式。如果你有一个`ProcessBuilder`对象`pb`，执行以下代码：

```java
pb.inheritIO()

```

然后它具有执行以下三个语句的相同效果：

```java
pb.redirectInput(Redirect.INHERIT)
pb.redirectOutput(Redirect.INHERIT)
pb.redirectError(Redirect.INHERIT)

```

在这两种情况下，输入、输出和错误数据将位于与当前 Java 进程的输入、输出和错误数据相同的位置。

# 在 HTML 页面中嵌入 JNLP 文件

Java 7 提供了一个新选项，可以加快在网页中部署小程序的速度。在 7 之前，当使用**Java 网络启动协议**（**JNLP**）启动小程序时，必须先从网络下载 JNLP 文件，然后才能启动小程序。有了新版本，JNLP 文件可以直接嵌入到 HTML 代码中，减少了小程序启动所需的时间。在这个例子中，我们将构建一个基本的小程序，并使用一个嵌入了 JNLP 的 HTML 页面来启动它。

## 准备工作

为了加快 Java 7 中小程序的启动速度，你必须：

1.  创建一个新的小程序。

1.  创建并编码一个 JNLP 文件。

1.  将 JNLP 文件的引用添加到 HTML 页面。

## 如何做...

1.  首先创建一个小程序，用于在 HTML 窗口中使用。以下是一个简单的小程序，可以用于本教程的目的。这个小程序有两个输入字段，`subtotal`和`taxRate`，还有一个`calculate`按钮用于计算总额：

```java
public class JNLPAppletExample extends Applet {
TextField subtotal = new TextField(10);
TextField taxRate = new TextField(10);
Button calculate = new Button("Calculate");
TextArea grandTot = new TextArea("Total = $", 2, 15, TextArea.SCROLLBARS_NONE);
@Override
public void init() {
this.setLayout(new GridLayout(3,2));
this.add(new Label("Subtotal = "));
this.add(subtotal);
this.add(new Label("Tax Rate = "));
this.add(taxRate);
this.add(calculate);
grandTot.setEditable(false);
this.add(grandTot);
calculate.addActionListener(new CalcListener());
}
class CalcListener implements ActionListener {
public void actionPerformed(ActionEvent event) {
JNLP fileembedding, in HTML pagedouble subTot;
double tax;
double grandTot;
subTot = validateSubTot(subtotal.getText());
tax = validateSubTot(taxRate.getText());
grandTot = calculateTotal(subTot, tax);
JNLPAppletExample.this.grandTot.setText("Total = $" + grandTot);
}
}
double validateSubTot(String s) {
double answer;
Double d;
try {
d = new Double(s);
answer = d.doubleValue();
}
catch (NumberFormatException e) {
answer = Double.NaN;
}
return answer;
}
double calculateTotal(double subTot, double taxRate) {
double grandTotal;
taxRate = taxRate / 100;
grandTotal = (subTot * taxRate) + subTot;
return grandTotal;
}
}

```

1.  接下来，创建一个名为`JNLPExample.jnlp`的 JNLP 文件。以下是一个示例 JNLP 文件，用于配合我们之前的小程序。请注意，在资源标签中引用了一个 JAR 文件。这个 JAR 文件，包含你的小程序，必须与你的 JNLP 文件和 HTML 文件在同一个位置，我们马上就会创建：

```java
<?xml version="1.0" encoding="UTF-8"?>
<jnlp href="http://JNLPExample.jnlp">
<information>
<title>Embedded JNLP File</title>
<vendor>Sample Vendor</vendor>
</information>
<resources>
<j2se version="7" />
<jar href="http://JNLPAppletExample.jar"
main="true" />
</resources>
<applet-desc
name="Embedded JNLP Example"
main-class="packt.JNLPAppletExample"
width="500"
height="500">
</applet-desc>
<update check="background"/>
</jnlp>

```

1.  创建 JNLP 文件后，必须对其进行编码。有几种在线资源可用于将 JNLP 文件转换为 BASE64，但本例中使用的是[`base64encode.org/`](http://base64encode.org/)。使用 UTF-8 字符集。一旦你有了编码的数据，你将在创建 HTML 文件时使用它。创建一个如下所示的 HTML 文件。请注意，高亮显示的 BASE64 编码字符串已经为简洁起见而缩短，但你的字符串会更长：

```java
<HTML>
<HEAD>
<TITLE>Embedded JNLP File Example</TITLE>
</HEAD>
<BODY>
<H3>Embedded JNLP Applet</H3>
<script src="img/deployJava.js"></script>
<script>
var jnlpFile = "http://JNLPExample.jnlp";
deployJava.createWebStartLaunchButtonEx(jnlpFile);
</script>
<script>
var attributes = {} ;
var parameters = {jnlp_href: 'JNLPExample.jnlp',
jnlp_embedded: 'PD94bWw...'};
deployJava.runApplet(attributes, parameters, '7');
</script>
</BODY>
</HTML>

```

1.  另外，请注意第一个脚本标签。为了避免使用`codebase`属性，我们利用了 Java 7 的另一个新特性，使用了一个开发工具包脚本。

1.  在浏览器窗口中加载你的应用程序。根据你当前的浏览器设置，你可能需要启用 JavaScript。你的小程序应该快速加载，并且看起来类似于以下的截图：

![How to do it...](img/5627_09_02.jpg)

## 它是如何工作的...

将 JNLP 文件嵌入 HTML 页面中允许 applet 立即加载，而不必首先从服务器下载。JNLP 文件在`href`属性中必须有相对路径，而且不应该指定`codebase`。通过将`codebase`属性留空，可以由 applet 网页的 URL 确定。

`resources`标签指定了 JAR 文件的位置和要使用的 Java 版本。JAR 文件的路径被假定为默认工作目录，JNLP 文件的位置也是如此。JNLP 文件中还包括了 applet 的描述，被`applet-desc`标签包围。在这个标签中指定了 applet 的名称和主类文件的名称。

HTML 文件包含了加载 applet 所需的信息，而不必从服务器下载 applet 信息。我们首先指定要使用 JavaScript 调用加载应用程序。然后，在我们的第一个 script 标签中，我们添加了一个部分，允许我们在没有`codebase`的情况下调用 applet。这是有利的，因为应用程序可以在不同的环境中加载和测试，而不必更改`codebase`属性。相反，它是从应用程序所在的网页继承而来。

部署工具包有两个函数可以在没有`codebase`属性的情况下在网页中部署 Java applet：`launchWebStartApplication`和`createWebStartLaunchButtonEx`。我们选择在这个示例中使用`createWebStartLaunchButtonEx`，但`launchWebStartApplication`选项也会在下文中讨论。在这两种情况下，客户端必须具有 Java SE 7 版本才能启动 applet，如果没有，他们将被引导到 Java 网站下载最新版本。

`createWebStartLaunchButtonEx`函数创建了一个应用程序的启动按钮。在`script`标签中，`jnlpFile`变量指定了 JNLP 文件的名称，并且是相对于 applet 网页的。然后将此文件名传递给`deployJava.createWebStartLaunchButtonEx`函数。

或者，`launchWebStartApplication`函数可以嵌入到 HTML 链接中。该函数在`href`标签中被调用，如下所示：

```java
<script src="img/deployJava.js"></script>
<a href="javascript:deployJava.launchWebStartApplication('JNLPExample.jnlp');">Launch</a>
</script>

```

HTML 文件中的第二个`script`标签包含了有关 JNLP 文件的信息。`jnlp_href`变量存储了 JNLP 文件的名称。JNLP 文件的编码形式由`jnlp_embedded`参数指定。BASE64 编码器对需要在文本媒介中存储和传输数据的二进制数据进行编码，比如电子邮件和 XML 文件。
