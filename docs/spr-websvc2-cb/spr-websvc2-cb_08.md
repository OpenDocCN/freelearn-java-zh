# 第八章：使用 WSS4J 库保护 SOAP Web 服务

在本章中，我们将涵盖：

+   使用用户名令牌和明文/摘要密码对 Web 服务调用进行身份验证

+   使用 Spring 安全对用户名令牌进行身份验证，密码为明文/摘要

+   使用数字签名保护 SOAP 消息

+   使用 X509 证书对 Web 服务调用进行身份验证

+   加密/解密 SOAP 消息

# 介绍

在上一章中，解释了在 Spring-WS 中使用 SUN 的实现**(XWSS)**：OASIS **Web-Services Security** **(WS-Security**或**WSS)**规范（使用`XwsSecurityInterceptor`执行安全操作）。在本章中，将解释 Spring-WS 对 Apache 的实现（WSS4J）OASIS WS-Security 规范的支持。尽管这两种 WS-Security 的实现都能够执行所需的安全操作（身份验证、签名消息和加密/解密），但 WSS4J 的执行速度比 XWSS 快。

Spring-WS 支持使用`Wss4jSecurityInterceptor`来支持 WSS4J，这是一个在调用`Endpoint`之前对请求消息执行安全操作的`EndpointInterceptor`。

虽然 XWSS 使用外部配置策略文件，但 WSS4J（以及相应的`Wss4jSecurityInterceptor`）不需要外部配置文件，并且完全可以通过属性进行配置。该拦截器应用的**验证**（接收方）和**保护**（发送方）操作通过`validationActions`和`securementActions`属性指定。可以将多个操作设置为由空格分隔的字符串。以下是本章中接收方（服务器端）的示例配置：

```java
<!--In receiver side(server-side in this chapter)-->
<bean id="wss4jSecurityInterceptor"
<property name="validationActions" value="UsernameToken Encrypt" />
..
<!--In sender side(client-side in this chapter)-->
<property name="securementActions" value="UsernameToken Encrypt" />
..
</bean>

```

`validationActions`是由空格分隔的操作列表。当发送者发送消息时，将执行`validationActions`（在接收方）。

`securementActions`是由空格分隔的操作列表。当发送者向接收者发送消息时，将执行这些操作。

+   **验证操作：**`UsernameToken, Timestamp, Encrypt, signature`和`NoSecurity`。

+   **安全操作：**`UsernameToken, UsernameTokenSignature, Timestamp, Encrypt, Signature`和`NoSecurity`。

操作的顺序很重要，并由`Wss4jSecurityInterceptor`应用。如果传入的 SOAP 消息`securementActions`（在发送方）与`validationActions`（在接收方）配置的方式不同，该拦截器将返回故障消息。

对于加密/解密或签名等操作，WSS4J 需要从密钥库（`store.jks`）中读取数据：

```java
<bean class="org.springframework. ws.soap.security.wss4j.support.CryptoFactoryBean">
<property name="key storePassword" value="storePassword" />
<property name="key storeLocation" value="/WEB-INF/store.jks" />
</bean>

```

在上一章中已经详细介绍了身份验证、签名、解密和加密等安全概念。在本章中，我们将讨论如何使用 WSS4J 实现这些功能。

为简化起见，在本章的大多数示例中，使用*如何使用 Spring-JUnit 支持集成测试*项目，第三章，*测试和监控 Web 服务*，来设置服务器并通过客户端发送和接收消息。然而，在最后一个示例中，使用了来自第二章的项目，*为 WS-Addressing 端点创建 Web 服务客户端*，用于服务器和客户端。

# 使用用户名令牌和明文/摘要密码对 Web 服务调用进行身份验证

身份验证简单地意味着检查服务的调用者是否是其所声称的。检查调用者的身份验证的一种方式是检查其密码（如果我们将用户名视为一个人，密码类似于该人的签名）。Spring-WS 使用`Wss4jSecurityInterceptor`来发送/接收带有密码的用户名令牌以及 SOAP 消息，并在接收方进行比较，比较其与属性格式中预定义的用户名/密码。拦截器的此属性设置强制告诉消息发送方，发送消息中应包含带有密码的用户名令牌，并且在接收方，接收方期望接收此用户名令牌以进行身份验证。

传输明文密码会使 SOAP 消息不安全。`Wss4jSecurityInterceptor`提供了配置属性（以属性格式）来将密码的摘要与发送方消息一起包括。在接收方，将与属性格式中设置的摘要密码进行比较，该摘要密码包含在传入消息中。

本示例介绍了如何使用用户名令牌对 Web 服务调用进行身份验证。在这里，客户端充当发送方，服务器充当接收方。本示例包含两种情况。在第一种情况下，密码将以明文格式传输。在第二种情况下，通过更改属性，密码将以摘要格式传输。

## 准备工作

在本示例中，我们有以下两个项目：

1.  `LiveRestaurant_R-8.1`（用于服务器端 Web 服务），具有以下 Maven 依赖项：

+   `spring-ws-security-2.0.1.RELEASE.jar`

+   `spring-expression-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

1.  `LiveRestaurant_R-8.1-Client`（用于客户端），具有以下 Maven 依赖项：

+   `spring-ws-security-2.0.1.RELEASE.jar`

+   `spring-ws-test-2.0.0.RELEASE.jar`

+   `spring-expression-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `junit-4.7.jar`

## 如何做...

按照以下步骤使用带有明文密码的普通用户名令牌进行身份验证：

1.  在服务器端应用程序上下文（`spring-ws-servlet.xml`）中注册`Wss4jSecurityInterceptor`，将验证操作设置为`UsernameToken`，并在此拦截器中配置`callbackHandler`（`....wss4j.callback.SimplePasswordValidationCallbackHandler`）。

1.  在客户端应用程序上下文（`applicationContext.xml`）中注册`Wss4jSecurityInterceptor`，将`securement`操作设置为`UsernameToken`，并在此处设置`username、password`和`password type`（以`text`格式）。

1.  在`Liverestaurant_R-8.1`上运行以下命令：

```java
mvn clean package tomcat:run 

```

1.  在`Liverestaurant_R-8.1-Client`上运行以下命令：

```java
mvn clean package 

```

+   以下是客户端的输出（请注意，在 SOAP 的`Envelope`的`Header`中突出显示了带有明文密码标记的`UsernameToken`）：

```java
Sent request .....
[<SOAP-ENV:Envelope>
<SOAP-ENV:Header>
<wsse:Security ...>
<wsse:UsernameToken ...>
<wsse:Username>admin</wsse:Username>
<wsse:Password #PasswordText">password</wsse:Password>
</wsse:UsernameToken>
</wsse:Security>
</SOAP-ENV:Header>
....
<tns:placeOrderRequest ...>
....
</tns:order>
</tns:placeOrderRequest>
... Received response ....
<tns:placeOrderResponse ...">
<tns:refNumber>order-John_Smith_1234</tns:refNumber>
</tns:placeOrderResponse>
... 

```

按照以下步骤使用用户名令牌和摘要密码实现身份验证：

1.  修改客户端应用程序上下文（`applicationContext.xml`）以将密码类型设置为摘要格式（请注意，服务器端无需进行任何更改）。

1.  在`Liverestaurant_R-8.1`上运行以下命令：

```java
mvn clean package tomcat:run 

```

1.  在`Liverestaurant_R-8.1-Client`上运行以下命令：

```java
mvn clean package 

```

+   以下是客户端输出（请注意，在 SOAP 信封的标头中突出显示了带有摘要密码标记的 UsernameToken）：

```java
Sent request .....
[<SOAP-ENV:Envelope>
<SOAP-ENV:Header>
<wsse:Security ...>
<wsse:UsernameToken ...>
<wsse:Username>admin</wsse:Username>
<wsse:Password #PasswordDigest">
VstlXUXOwyKCIxYh29bNWaSKsRI=
</wsse:Password>
</wsse:UsernameToken>
</wsse:Security>
</SOAP-ENV:Header>
....
<tns:placeOrderRequest ...>
....
</tns:order>
</tns:placeOrderRequest>
... Received response ....
<tns:placeOrderResponse ...">
<tns:refNumber>order-John_Smith_1234</tns:refNumber>
</tns:placeOrderResponse>
... 

```

## 工作原理...

`Liverestaurant_R-8.1`项目是一个服务器端 Web 服务，要求其客户端发送包含用户名和密码的 SOAP 信封。

`Liverestaurant_R-8.1-Client`项目是一个客户端测试项目，用于向服务器发送包含用户名令牌和密码的 SOAP 信封。

在服务器端，`Wss4jSecurityInterceptor`强制服务器对所有传入消息进行用户名令牌验证：

```java
<sws:interceptors>
....
<bean id="wss4jSecurityInterceptor" class="org. springframework. ws.soap.security.wss4j.Wss4jSecurityInterceptor">
<property name= "validationCallbackHandler" ref="callbackHandler" />
<property name="validationActions" value="UsernameToken" />
</bean>
</sws:interceptors>

```

拦截器使用`validationCallbackHandler`（`SimplePasswordValidationCallbackHandler`）来比较传入消息的用户名/密码与包含的用户名/密码（admin/password）。

```java
<bean id="callbackHandler" class="org.springframework.aws.soap. security.wss4j.callback.SimplePasswordValidationCallbackHandler">
<property name="users">
<props>
<prop key="admin">password</prop>
</props> 
</property>
</bean>

```

在客户端上，`wss4jSecurityInterceptor`在所有传出消息中包含用户名（`admin/password`）令牌：

```java
<bean id="wss4jSecurityInterceptor" class="org.springframework.ws. soap.security.wss4j.Wss4jSecurityInterceptor">
<property name="securementActions" value="UsernameToken" /> 
<property name="securementUsername" value="admin" />
<property name="securementPassword" value="password" />
<property name="securementPasswordType" value="PasswordText" /> 
</bean>

```

在这种情况下，使用纯文本用户名令牌进行身份验证，因为客户端在进行中的消息中包含了纯文本密码（`<property name="securementPasswordType" value="PasswordText"/>`）：

```java
<wsse:UsernameToke......>
<wsse:Username>admin</wsse:Username>
<wsse:Password ...#PasswordText">password</wsse:Password>
</wsse:UsernameToken> 

```

然而，在第二种情况下，使用摘要用户名令牌进行身份验证，因为密码摘要（`<property name="securementPasswordType" value="PasswordDigest">`）包含在用户名令牌中：

```java
<wsse:UsernameToken...>
<wsse:Username>admin</wsse:Username>
<wsse:Password ...#PasswordDigest">
VstlXUXOwyKCIxYh29bNWaSKsRI=
</wsse:Password>
...
</wsse:UsernameToken> 

```

在这种情况下，服务器将传入的 SOAP 消息摘要密码与`spring-ws-servlet.xml`中设置的计算摘要密码进行比较。通过这种方式，与密码以纯文本形式传输的第一种情况相比，通信将更加安全。

## 另请参阅...

在这一章中：

+   *使用 Spring 安全性进行 Web 服务调用，对具有纯文本/摘要密码的用户名令牌进行身份验证*

+   *使用 X509 证书进行 Web 服务调用的身份验证*

# 使用 Spring 安全性进行 Web 服务调用的身份验证，以验证具有纯文本/摘要密码的用户名令牌

在这里，我们使用用户名令牌进行身份验证，密码为摘要/纯文本，就像本章的第一个示例中所做的那样。这里唯一的区别是使用 Spring 安全框架进行身份验证（SpringPlainTextPasswordValidationCallbackHandler 和`SpringDigestPasswordValidationCallbackHandler`）。由于 Spring 安全框架超出了本书的范围，因此这里不进行描述。但是，您可以在以下网站的*Spring 安全参考*文档中了解更多信息：[`www.springsource.org/security`](http://www.springsource.org/security)。

就像本章的第一个示例一样，这个示例也包含两种情况。在第一种情况下，密码将以纯文本格式传输。在第二种情况下，通过更改配置，密码将以摘要格式传输。

## 准备工作

在这个示例中，我们有以下两个项目：

1.  `LiveRestaurant_R-8.2`（用于服务器端 Web 服务），具有以下 Maven 依赖项：

+   `spring-ws-security-2.0.1.RELEASE.jar`

+   `spring-expression-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

1.  `LiveRestaurant_R-8.2-Client`（用于客户端），具有以下 Maven 依赖项：

+   `spring-ws-security-2.0.1.RELEASE.jar`

+   `spring-ws-test-2.0.0.RELEASE.jar`

+   `spring-expression-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `junit-4.7.jar`

## 如何做...

按照以下步骤实现 Web 服务调用的身份验证，使用 Spring 安全性对具有纯文本密码的用户名令牌进行身份验证：

1.  在服务器端应用程序上下文（`spring-ws-servlet.xml`）中注册`Wss4jSecurityInterceptor`，将验证操作设置为`UsernameToken`，并在此拦截器中配置`validationCallbackHandler`（....wss4j.callback.SpringPlainTextPasswordValidationCallbackHandler）。

1.  在客户端应用程序上下文（`applicationContext.xml`）中注册`Wss4jSecurityInterceptor`，将安全操作设置为`UsernameToken`，并设置用户名、密码和密码类型（这里是文本格式）。

1.  在`Liverestaurant_R-8.2`上运行以下命令：

```java
mvn clean package tomcat:run 

```

1.  在`Liverestaurant_R-8.2-Client`上运行以下命令：

```java
mvn clean package 

```

+   这是客户端的输出（请注意，在 SOAP 的头部中突出显示的具有摘要密码标记的 UsernameToken）：

```java
Sent request .....
<SOAP-ENV:Envelope>
<SOAP-ENV:Header>
<wsse:Security ...>
<wsse:UsernameToken ...>
<wsse:Username>admin</wsse:Username>
<wsse:Password #PasswordText">password</wsse:Password>
</wsse:UsernameToken>
</wsse:Security>
</SOAP-ENV:Header>
....
<tns:placeOrderRequest ...>
....
</tns:order>
</tns:placeOrderRequest>
... Received response ....
<tns:placeOrderResponse ...">
<tns:refNumber>order-John_Smith_1234</tns:refNumber>
</tns:placeOrderResponse>
.... 

```

按照以下步骤实现使用 Spring 安全性进行 Web 服务调用的身份验证，以验证具有摘要密码的用户名令牌：

1.  在服务器端应用程序上下文（`spring-ws-servlet.xml`）中修改`Wss4jSecurityInterceptor`并在此拦截器中配置`validationCallbackHandler`（`....ws.soap.security.wss4j.callback.SpringDigestPasswordValidationCallbackHandler`）。

1.  在客户端应用程序上下文（`applicationContext.xml`）中修改`Wss4jSecurityInterceptor`以设置密码类型（这里是摘要格式）。

1.  在`Liverestaurant_R-8.2`上运行以下命令：

```java
mvn clean package tomcat:run 

```

1.  在`Liverestaurant_R-8.2-Client`上运行以下命令：

```java
mvn clean package 

```

+   以下是客户端的输出（请注意 SOAP 信封的标头中突出显示的带有摘要密码标签的 UsernameToken）：

```java
Sent request .....
[<SOAP-ENV:Envelope>
<SOAP-ENV:Header>
<wsse:Security ...>
<wsse:UsernameToken ...>
<wsse:Username>admin</wsse:Username>
<wsse:Password #PasswordDigest">
VstlXUXOwyKCIxYh29bNWaSKsRI=</wsse:Password>
</wsse:UsernameToken>
</wsse:Security>
</SOAP-ENV:Header>
....
<tns:placeOrderRequest ...>
....
</tns:order>
</tns:placeOrderRequest>
... Received response ....
<tns:placeOrderResponse ...">
<tns:refNumber>order-John_Smith_1234</tns:refNumber>
</tns:placeOrderResponse>
... 

```

## 它是如何工作的...

在`Liverestaurant_R-8.2`项目中，客户端和服务器的安全性几乎与`Liverestaurant_R-8.1`相同（如本章第一个配方所示），只是在服务器端验证用户名令牌。Spring 安全类负责通过与从 DAO 层获取的数据进行比较来验证用户名和密码（而不是在`spring-ws-servlet.xml`中硬编码用户名/密码）。此外，可以从 DAO 层获取其他与成功验证用户相关的数据，并返回以进行授权以检查一些帐户数据。

在第一种情况下，`CallbackHandler SpringPlainTextPasswordValidationCallbackHandler`使用`authenticationManager`，该管理器使用`DaoAuthenticationProvider`。

```java
<bean id="springSecurityHandler" class="org.springframework.ws.soap.security. wss4j.callback.SpringPlainTextPasswordValidationCallbackHandler">
<property name="authenticationManager" ref="authenticationManager"/>
</bean>
<bean id="authenticationManager" class= "org.springframework.security.authentication.ProviderManager">
<property name="providers">
<bean class="org.springframework. security.authentication.dao.DaoAuthenticationProvider">
<property name="userDetailsService" ref="userDetailsService"/>
</bean>
</property>
</bean>

```

此提供程序调用自定义用户信息服务（`MyUserDetailService.java`），该服务从提供程序获取用户名并在内部从 DAO 层获取该用户的所有信息（例如密码、角色、是否过期等）。最终，该服务以`UserDetails`类型类（`MyUserDetails.java`）返回填充的数据。现在，如果`UserDetails`数据与传入消息的用户名/密码匹配，则返回响应；否则，返回 SOAP 故障消息：

```java
public class MyUserDetailService implements UserDetailsService {
@Override
public UserDetails loadUserByUsername(String username)
throws UsernameNotFoundException, DataAccessException {
return getUserDataFromDao(username);
}
private MyUserDetail getUserDataFromDao(String username) {
/**
*Real scenario: find user data from a DAO layer by userName,
* if this user name found, populate MyUserDetail with its data(username, password,Role, ....).
*/
MyUserDetail mydetail=new MyUserDetail( username,"pass",true,true,true,true);
mydetail.getAuthorities().add( new GrantedAuthorityImpl("ROLE_GENERAL_OPERATOR"));
return mydetail;
}

```

然而，在第二种情况下，`CallbackHandler`是`SpringDigestPasswordValidationCallbackHandler`，它将 SOAP 传入消息中包含的摘要密码与从 DAO 层获取的摘要密码进行比较（请注意，DAO 层可以从不同的数据源获取数据，如数据库、LDAP、XML 文件等）：

```java
<bean id="springSecurityHandler" class="org.springframework.ws.soap.security.wss4j.callback. SpringDigestPasswordValidationCallbackHandler">
<property name="userDetailsService" ref="userDetailsService"/>
</bean>

```

与本章第一个配方相同，在客户端应用程序上下文中将`<property name="securementPasswordType" value="PasswordText">`修改为`PasswordDigest`会导致密码以摘要格式传输。

## 另请参阅...

在本章中：

+   *使用用户名令牌进行 Web 服务调用的身份验证，使用明文/摘要密码*

+   *使用 X509 证书对 Web 服务调用进行身份验证*

# 使用数字签名保护 SOAP 消息

在安全术语中，签名的目的是验证接收到的消息是否被篡改。签名在 WS-Security 中扮演着两个主要任务，即对消息进行签名和验证签名。消息签名涉及的所有概念都在上一章的*使用数字签名保护 SOAP 消息*中详细介绍。在这个配方中，使用 WSS4J 进行签名和验证签名。

Spring-WS 的`Wss4jSecurityInterceptor`能够根据 WS-Security 标准进行签名和验证签名。

将此拦截器的`securementActions`属性设置为`Signature`会导致发送方对传出消息进行签名。要加密签名令牌，需要发送方的私钥。需要在应用程序上下文文件中配置密钥库的属性。`securementUsername`和`securementPassword`属性指定了用于使用的密钥库中的私钥的别名和密码。`securementSignatureCrypto`应指定包含私钥的密钥库。

将`validationActions`设置为`value="Signature`"会导致消息的接收方期望并验证传入消息的签名（如开头所述）。`validationSignatureCrypto` bean 应指定包含发送方公钥证书（受信任证书）的密钥库。

来自`wss4j`包的`org.springframework.ws.soap.security.wss4j.support.CryptoFactoryBean`可以提取密钥库数据（例如证书和其他密钥库信息），并且这些数据可以用于身份验证。

在本教程中，客户端存储的私钥用于加密消息的客户端签名。在服务器端，包含在服务器密钥库中的客户端公钥证书（在受信任证书条目中）将用于解密消息签名令牌。然后服务器对签名进行验证（如开头所述）。在[第七章中使用的密钥库，在*准备配对和对称密钥库*中使用。

## 准备工作

在本教程中，我们有以下两个项目：

1.  `LiveRestaurant_R-8.3`（用于服务器端 Web 服务），具有以下 Maven 依赖项：

+   `spring-ws-security-2.0.1.RELEASE.jar`

+   `spring-expression-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

1.  `LiveRestaurant_R-8.3-Client`（用于客户端），具有以下 Maven 依赖项：

+   `spring-ws-security-2.0.1.RELEASE.jar`

+   `spring-ws-test-2.0.0.RELEASE.jar`

+   `spring-expression-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `junit-4.7.jar`

## 如何做...

1.  在服务器端应用程序上下文（`spring-ws-servlet.xml`）中注册`Wss4jSecurityInterceptor`，将验证操作设置为`Signature`，并在此拦截器中将`validationSignatureCrypto`属性设置为`CryptoFactoryBean`（配置服务器端密钥库位置及其密码）。

1.  在客户端应用程序上下文（`applicationContext.xml`）中注册`Wss4jSecurityInterceptor`，将安全操作设置为`Signature`，并在此拦截器中将`securementSignatureCrypto`属性设置为`CryptoFactoryBean`（配置客户端密钥库位置及其密码）。

1.  在`Liverestaurant_R-8.3`上运行以下命令：

```java
mvn clean package tomcat:run 

```

1.  在`Liverestaurant_R-8.3-Client`上运行以下命令：

```java
mvn clean package 

```

+   以下是客户端的输出（请注意突出显示的文本）：

```java
Sent request ....
<SOAP-ENV:Header>
<wsse:Security...>
<ds:Signature ...>
<ds:SignedInfo>
.....
</ds:SignedInfo>
<ds:SignatureValue>
IYSEHmk+.....
</ds:SignatureValue>
<ds:KeyInfo ..>
<wsse:SecurityTokenReference ...>
<ds:X509Data>
<ds:X509IssuerSerial>
<ds:X509IssuerName>
CN=MyFirstName MyLastName,OU=Software,O=MyCompany,L=MyCity,ST=MyProvince,C=ME
</ds:X509IssuerName>
<ds:X509SerialNumber>1311686430</ds:X509SerialNumber>
</ds:X509IssuerSerial>
</ds:X509Data>
</wsse:SecurityTokenReference>
</ds:KeyInfo>
</ds:Signature>
</wsse:Security>
</SOAP-ENV:Header>
<SOAP-ENV:Body ...>
<tns:placeOrderRequest ...>
.....
</tns:order>
</tns:placeOrderRequest>
.. Received response
.....<tns:placeOrderResponse....>
<tns:refNumber>order-John_Smith_1234</tns:refNumber>
</tns:placeOrderResponse> 

```

## 它是如何工作的...

服务器端的安全配置要求客户端在消息中包含一个二进制签名令牌。客户端配置文件中的设置将签名令牌包含在传出消息中。客户端使用自己的私钥（包含在客户端密钥库中）对消息的签名进行加密（根据消息的内容计算）。在服务器端，来自服务器端的客户端证书（受信任证书）密钥库用于解密签名令牌。然后将对二进制签名令牌的签名验证（如本章开头所述）进行验证。

在服务器端将`validationActions`设置为`Signature`会导致它期望来自客户端配置的签名，并且设置密钥库会导致服务器端密钥库中的客户端公钥证书（受信任证书）用于解密签名。然后服务器对签名进行验证：

```java
<sws:interceptors>
<bean class="org.springframework.ws.soap.server.endpoint. interceptor.PayloadValidatingInterceptor">
<property name="schema" value="/WEB-INF/orderService.xsd" />
<property name="validateRequest" value="true" />
<property name="validateResponse" value="true" />
</bean>
<bean class="org.springframework.ws.soap.server.endpoint. interceptor.SoapEnvelopeLoggingInterceptor"/>
<bean id="wsSecurityInterceptor" class="org.springframework.ws. soap.security.wss4j.Wss4jSecurityInterceptor">
<property name="validationActions" value="Signature" />
<property name="validationSignatureCrypto">
<bean class="org.springframework.ws.soap.security. wss4j.support.CryptoFactoryBean">
<property name="key storePassword" value="serverPassword" />
<property name="key storeLocation" value="/WEB-INF/serverStore.jks" />
</bean>
</property>
</bean>
</sws:interceptors>

```

代码语句`<property name="securementActions" value="Signature" />`，并在客户端配置中设置密钥库会导致客户端发送加密签名（使用别名为`client`的客户端私钥，并且客户端加密从消息生成的哈希（签名）），并随消息一起发送：

```java
<bean id="wss4jSecurityInterceptor" class="org.springframework.ws. soap.security.wss4j.Wss4jSecurityInterceptor">
<property name="securementActions" value="Signature" />
<property name="securementUsername" value="client" />
<property name="securementPassword" value="cliPkPassword" />
<property name="securementSignatureCrypto">
<bean class="org.springframework.ws.soap.security. wss4j.support.CryptoFactoryBean">
<property name="key storePassword" value="clientPassword" />
<property name="key storeLocation" value="classpath:/clientStore.jks" />
</bean>
</property>
</bean>

```

## 另请参阅...

在本章中：

+   *使用 X509 证书对 Web 服务调用进行身份验证*

第七章，*使用 XWSS 库保护 SOAP Web 服务：*

+   *准备配对和对称密钥存储*

# 使用 X509 证书对 Web 服务调用进行身份验证

在本章的前面部分，介绍了如何使用用户名令牌对传入消息进行身份验证。随传入消息一起传来的客户端证书可以用作替代用户名令牌进行身份验证。

为了确保所有传入的 SOAP 消息携带客户端的证书，发送方的配置文件应该签名，接收方应该要求所有消息都有签名。换句话说，客户端应该对消息进行签名，并在传出消息中包含 X509 证书，服务器首先将传入的证书与信任的证书进行比较，该证书嵌入在服务器密钥库中，然后进行验证传入消息的签名。

## 准备工作

在此配方中，我们有以下两个项目：

1.  `LiveRestaurant_R-8.4`（用于服务器端 Web 服务），具有以下 Maven 依赖项：

+   `spring-ws-security-2.0.1.RELEASE.jar`

+   `spring-expression-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

1.  `LiveRestaurant_R-8.4-Client`（用于客户端），具有以下 Maven 依赖项：

+   `spring-ws-security-2.0.1.RELEASE.jar`

+   `spring-ws-test-2.0.0.RELEASE.jar`

+   `spring-expression-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `junit-4.7.jar`

## 如何做...

1.  在服务器端应用程序上下文（`spring-ws-servlet.xml`）中注册`Wss4jSecurityInterceptor`，将验证操作设置为`Signature`，并在此拦截器中将属性`validationSignatureCrypto`设置为`CryptoFactoryBean`（配置服务器端密钥库位置及其密码）。

1.  在客户端应用程序上下文（`applicationContext.xml`）中注册`Wss4jSecurityInterceptor`，将安全操作设置为`Signature`，设置一个属性（`securementSignatureKeyIdentifier`）以包含二进制`X509`令牌，并在此拦截器中将属性`securementSignatureCrypto`设置为`CryptoFactoryBean`（配置客户端密钥库位置及其密码）。 

以下是客户端的输出（请注意突出显示的文本）：

```java
Sent request ....
<SOAP-ENV:Header>
<wsse:Security ...>
<wsse:BinarySecurityToken....wss-x509-token-profile- 1.0#X509v3" ...>
MIICbTCCAdagAwIBAgIETi6/HjANBgkqhki...
</wsse:BinarySecurityToken>
<ds:Signature ....>
.....
....
</ds:Signature>.... 

```

## 工作原理...

签名和验证签名与本章中*使用数字签名保护 SOAP 消息*的配方相同。不同之处在于配置的以下部分，用于生成包含 X509 证书的`BinarySecurityToken`元素，并在发送方的传出消息中包含它：

```java
<property name="securementSignatureKeyIdentifier" value="DirectReference" />

```

在签名消息时将客户端证书嵌入调用者消息中，使服务器验证该证书与密钥库中包含的证书（受信任的证书条目）一致。此验证确认了调用者是否是他/她声称的人。

## 另请参阅...

在本章中：

+   *使用数字签名保护 Soap 消息*

第七章，*使用 XWSS 库保护 SOAP Web 服务：*

+   *准备配对和对称密钥存储*

# 加密/解密 SOAP 消息

SOAP 消息的加密和解密概念与第七章中描述的*加密/解密 SOAP 消息*相同。Spring-WS 的`Wss4jSecurityInterceptor`通过在接收方（这里是服务器端）设置属性`validationActions`为`Encrypt`来提供对传入 SOAP 消息的解密。在发送方（这里是客户端）设置属性`securementActions`会导致发送方对传出消息进行加密。

`Wss4jSecurityInterceptor`需要访问密钥库进行加密/解密。在使用对称密钥的情况下，`Key storeCallbackHandler`负责访问（通过设置`location`和`password`属性）并从对称密钥库中读取，并将其传递给拦截器。然而，在使用私钥/公钥对存储的情况下，`CryptoFactoryBean`将执行相同的工作。

在这个示例中，在第一种情况下，客户端和服务器共享的对称密钥用于客户端的加密和服务器端的解密。然后，在第二种情况下，客户端密钥库中的服务器公钥证书（受信任的证书）用于数据加密，服务器端密钥库中的服务器私钥用于解密。

在前两种情况下，整个有效载荷用于加密/解密。通过设置一个属性，可以对有效载荷的一部分进行加密/解密。在第三种情况下，只有有效载荷的一部分被设置为加密/解密的目标。

## 准备工作

在这个示例中，我们有以下两个项目：

1.  `LiveRestaurant_R-8.5`（用于服务器端 Web 服务），具有以下 Maven 依赖项：

+   `spring-ws-security-2.0.1.RELEASE.jar`

+   `spring-expression-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

1.  `LiveRestaurant_R-8.5-Client`（用于客户端），具有以下 Maven 依赖项：

+   `spring-ws-security-2.0.1.RELEASE.jar`

+   `spring-ws-test-2.0.0.RELEASE.jar`

+   `spring-expression-3.0.5.RELEASE.jar`

+   `log4j-1.2.9.jar`

+   `junit-4.7.jar`

## 操作步骤...

按照以下步骤使用对称密钥实施加密/解密：

1.  在服务器端应用程序上下文（`spring-ws-servlet.xml`）中注册`Wss4jSecurityInterceptor`，将验证操作设置为`Encrypt`，并在此拦截器内配置`Key storeCallbackHandler`以从对称密钥库中读取（配置服务器端对称密钥库位置及其密码）。

1.  在客户端应用程序上下文（`applicationContext.xml`）中注册`Wss4jSecurityInterceptor`，将安全操作设置为`Encrypt`，并配置`Key storeCallbackHandler`以从对称密钥库中读取（配置客户端对称密钥库位置及其密码）。

1.  在`Liverestaurant_R-8.5`上运行以下命令：

```java
mvn clean package tomcat:run 

```

1.  在`Liverestaurant_R-8.5-Client`上运行以下命令：

```java
mvn clean package 

```

+   以下是客户端的输出（请注意突出显示的文本）：

```java
Sent request...
<SOAP-ENV:Header>
<wsse:Security...>
<xenc:ReferenceList><xenc:DataReference../> </xenc:ReferenceList>
</wsse:Security>
</SOAP-ENV:Header>
<SOAP-ENV:Body>
<xenc:EncryptedData ...>
<xenc:EncryptionMethod..tripledes-cbc"/>
<ds:KeyInfo...>
<ds:KeyName>symmetric</ds:KeyName>
</ds:KeyInfo>
<xenc:CipherData><xenc:CipherValue>
3a2tx9zTnVTKl7E+Q6wm...
</xenc:CipherValue></xenc:CipherData>
</xenc:EncryptedData>
</SOAP-ENV:Body>
</SOAP-ENV:Envelope> 

```

按照以下步骤在客户端密钥库（在`clientStore.jsk`中）上使用服务器信任的证书实施加密，并在服务器端私钥（在`serverStore.jks`中）上进行解密：

1.  在服务器端应用程序上下文（`spring-ws-servlet.xml`）中注册`Wss4jSecurityInterceptor`，将验证操作设置为`Encrypt`，并在此拦截器内将属性`validationSignatureCrypto`设置为`CryptoFactoryBean`（配置服务器端密钥库位置及其密码）。

1.  在客户端应用程序上下文（`applicationContext.xml`）中注册`Wss4jSecurityInterceptor`，将安全操作设置为`Encrypt`，并在此拦截器内将`securementSignatureCrypto`设置为`CryptoFactoryBean`（配置客户端密钥库位置及其密码）。

以下是服务器端的输出（请注意突出显示的文本）：

```java
<SOAP-ENV:Header>
<wsse:Security...>
<xenc:EncryptionMethod ..">
<wsse:SecurityTokenReference ...>
<ds:X509Data>
<ds:X509IssuerSerial>
<ds:X509IssuerName>
CN=MyFirstName MyLastName,OU=Software,O=MyCompany, L=MyCity,ST=MyProvince,C=ME
</ds:X509IssuerName>
<ds:X509SerialNumber>1311685900</ds:X509SerialNumber>
</ds:X509IssuerSerial>
</ds:X509Data>
</wsse:SecurityTokenReference>
</ds:KeyInfo>
<xenc:CipherData>
<xenc:CipherValue>dn0lokNhtmZ9...</xenc:CipherValue>
</xenc:CipherData><xenc:ReferenceList>
....
</wsse:Security>
</SOAP-ENV:Header><SOAP-ENV:Body>
<xenc:EncryptedData .../>
<ds:KeyInfo ...xmldsig#">
<wsse:SecurityTokenReference ...>
<wsse:Reference .../>
</wsse:SecurityTokenReference>
</ds:KeyInfo>
<xenc:CipherData><xenc:CipherValue>
UDO872y+r....</xenc:CipherValue>
</xenc:CipherData></xenc:EncryptedData>
</SOAP-ENV:Body> 

```

按照以下步骤在有效载荷上实施加密/解密：

1.  修改第 2 种情况，将`Wss4jSecurityInterceptor`上的`securementEncryptionParts`属性设置为有效载荷的特定部分，无论是在服务器端还是客户端。

1.  在`Liverestaurant_R-8.5`上运行以下命令：

```java
mvn clean package tomcat:run 

```

1.  在`Liverestaurant_R-8.5-Client`上运行以下命令：

```java
mvn clean package 

```

+   以下是客户端的输出（请注意突出显示的文本）：

```java
..........
<SOAP-ENV:Body>
<tns:placeOrderRequest...>
<xenc:EncryptedData...>
<xenc:EncryptionMethod .../>
<ds:KeyInfo..xmldsig#">
<wsse:SecurityTokenReference ...>
<wsse:Reference.../></wsse:SecurityTokenReference>
</ds:KeyInfo><xenc:CipherData>
<xenc:CipherValue>
pGzc3/j5GX......
</xenc:CipherValue>
</xenc:CipherData>
</xenc:EncryptedData>
</tns:placeOrderRequest>
....... 

```

## 工作原理...

在第一种情况下，客户端和服务器都共享对称密钥。客户端使用对称密钥加密整个有效载荷，并将其发送到服务器。在服务器端，相同的密钥将用于解密有效载荷。

然而，在第二和第三种情况下，客户端存储中嵌入的服务器证书用于加密有效负载，在服务器端，服务器存储的私钥将用于解密。第二种和第三种情况之间的区别在于第二种情况加密/解密整个有效负载，但在第三种情况下，只有部分有效负载将成为加密/解密的目标。

在第一种情况下，在服务器端将`validationActions`设置为`Encrypt`会导致服务器使用对称密钥解密传入消息。拦截器使用`ValidationCallbackHandler`进行解密，使用在`location`属性中设置的对称密钥存储。`type`属性设置密钥的存储类型，`password`设置对称密钥的密钥存储密码：

```java
<bean class="org.springframework.ws.soap. security.wss4j.Wss4jSecurityInterceptor">
<property name="validationActions" value="Encrypt"/>
<property name="validationCallbackHandler">
<bean class="org.springframework.ws.soap.security. wss4j.callback.Key storeCallbackHandler">
<property name="key store">
<bean class="org.springframework.ws.soap.security. support.Key storeFactoryBean">
<property name="location" value="/WEB- INF/symmetricStore.jks"/>
<property name="type" value="JCEKS"/>
<property name="password" value="symmetricPassword"/>
</bean>
</property>
<property name="symmetricKeyPassword" value="keyPassword"/>
</bean>
</property>
</bean>

```

在客户端，将`securementActions`属性设置为`Encrypt`会导致客户端加密所有传出消息。通过将`securementEncryptionKeyIdentifier`设置为`EmbeddedKeyName`来自定义加密。选择`EmbeddedKeyName`类型时，加密的秘钥是必需的。对称密钥别名（此处为对称）由`securementEncryptionUser`设置。

默认情况下，SOAP 标头中的`ds:KeyName`元素采用`securementEncryptionUser`属性的值。`securementEncryptionEmbeddedKeyName`可用于指示不同的值。`securementEncryptionKeyTransportAlgorithm`属性定义要使用的算法来加密生成的对称密钥。`securementCallbackHandler`提供了`Key storeCallbackHandler`，指向适当的密钥存储，即服务器端配置中描述的对称密钥存储：

```java
<bean class="org.springframework.ws.soap. security.wss4j.Wss4jSecurityInterceptor">
<property name="securementActions" value="Encrypt" />
<property name="securementEncryptionKeyIdentifier" value="EmbeddedKeyName"/>
<property name="securementEncryptionUser" value="symmetric"/>
<property name="securementEncryptionEmbeddedKeyName" value="symmetric"/>
<property name="SecurementEncryptionSymAlgorithm" value="http://www.w3.org/2001/04/xmlenc#tripledes-cbc"/>
<property name="securementCallbackHandler">
<bean class="org.springframework.ws.soap.security. wss4j.callback.Key storeCallbackHandler">
<property name="symmetricKeyPassword" value="keyPassword"/>
<property name="key store">
<bean class="org.springframework.ws.soap.security. support.Key storeFactoryBean">
<property name="location" value="/symmetricStore.jks"/>
<property name="type" value="JCEKS"/>
<property name="password" value="symmetricPassword"/>
</bean>
</property>
</bean>
</property>
</bean>

```

在第二和第三种情况下，服务器端配置的`validationDecryptionCrypto`几乎与第一种情况解密数据的方式相同：

```java
<bean class="org.springframework.ws.soap.security. wss4j.Wss4jSecurityInterceptor">
<property name="validationActions" value="Encrypt" />
<property name="validationDecryptionCrypto">
<bean class="org.springframework.ws.soap.security. wss4j.support.CryptoFactoryBean">
<property name="key storePassword" value="serverPassword" />
<property name="key storeLocation" value="/WEB- INF/serverStore.jks" />
</bean>
</property>
<property name="validationCallbackHandler">
<bean class="org.springframework.ws.soap.security. wss4j.callback.Key storeCallbackHandler">
<property name="privateKeyPassword" value="serPkPassword" />
</bean>
</property>
</bean>

```

在客户端，将`securementActions`的`value="Encrypt`"设置为会导致客户端加密所有传出消息。`securementEncryptionCrypto`用于设置密钥存储位置和密码。`SecurementEncryptionUser`用于设置服务器证书在客户端密钥存储中的别名：

```java
<bean class="org.springframework.ws.soap.security. wss4j.Wss4jSecurityInterceptor">
<property name="securementActions" value="Encrypt" />
<property name="securementEncryptionUser" value="server" />
<property name="securementEncryptionCrypto">
<bean class="org.springframework.ws.soap.security. wss4j.support.CryptoFactoryBean">
<property name="key storePassword" value="clientPassword" />
<property name="key storeLocation" value="/clientStore.jks" />
</bean>
</property>
</bean>

```

*第 2 种*和*第 3 种*之间的区别在于在客户端/服务器端配置中的配置设置仅导致部分有效负载被加密/解密。

```java
---client/server configuration file
<property name="securementEncryptionParts"value="{Content} {http://www.packtpub.com/LiveRestaurant/OrderService/schema} placeOrderRequest"/>

```

## 另请参阅...

在本章中：

+   *使用数字签名保护 SOAP 消息*

第二章,*为 SOAP Web 服务构建客户端*

+   *为 WS-Addressing 端点创建 Web 服务客户端*

第七章,*使用 XWSS 库保护 SOAP Web 服务*

+   *准备*一对和对称密钥存储*
