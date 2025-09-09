# 第十章：安全

到目前为止，本书中讨论的大部分主题都没有涉及安全这个话题。这是一个经常被忽视的话题，在一些实际项目中，只有在为时已晚时才会引起关注。

开发人员和项目经理将安全视为一种必要的恶，而不是为业务带来巨大利益的东西。尽管如此，它是一个利益相关者必须了解的话题。

在云计算和分布式应用的年代，许多要求都发生了变化。本章将探讨过去的情况以及今天的要求。它将涵盖如何使用现代 Java EE 实现安全：

+   从过去学到的安全经验

+   企业安全原则

+   现代安全解决方案

+   如何使用现代 Java EE 实现安全

# 从过去学到的经验教训

在当今世界，IT 安全是一个非常重要的方面。大多数人已经意识到，如果滥用，信息技术可以造成很多危害。

在过去半个世纪的计算机领域，我们可以从安全方面学到很多东西，这不仅仅适用于企业软件。

让我们回顾一下企业应用开发过去的一些经验教训。在过去的几年里，最大的安全问题在于加密和管理凭据的方法。

如果正确应用，加密和签名数据是保持秘密的一种极其安全的方式。它完全取决于所使用的算法和密钥长度。

有很多加密和散列算法最终证明不够安全。**DES**是一个例子，以及常用的**MD5**散列算法。截至本书编写时，**AES**使用 192 位或 256 位密钥长度被认为是安全的。对于散列算法，建议使用至少 256 位的**SHA-2**或**SHA-3**。

作为应用程序一部分存储的用户凭据不得以纯文本形式存储。过去已经发生了太多的安全漏洞，特别是针对存储密码的数据库。此外，简单地散列密码而不提供适当的密码**盐**是不被鼓励的。

通常，对于企业开发者来说，如果可能的话，最好不要自己实现安全功能。公司的想法是创建自己的安全实现，这些实现没有在其他地方使用过，因此提供了一种*隐蔽性安全*。然而，这实际上产生了相反的效果，除非有安全专家的参与，否则实际上会导致更不安全的解决方案。

大多数企业安全需求不需要他们自己的、定制的实现。企业框架及其实现已经包含了经过众多用例良好测试的相应功能。我们将在本章后面探讨这些 Java 企业版的 API。

如果应用程序需要自定义加密的使用，那么必须使用运行时或第三方依赖项提供的实现。出于这个原因，Java 平台提供了 **Java Cryptography Extension** (**JCE**)。它提供了现代加密和散列算法的实现。

通常，应用程序只有在业务用例绝对需要时才应处理和存储安全信息。特别是对于身份验证和授权，有一些方法可以避免在多个系统中存储用户凭据。

# 现代世界的安全

应用程序的更多分布导致对保护通信的需求更高。交换信息的完整性需要得到保证。同样，人们意识到加密的必要性，尤其是在加密通信时。

在今天的商业世界中，工程师有哪些可能性？他们在实现安全时应该遵循哪些原则？

# 安全原则

在企业应用程序中实现安全时应遵循一些基本原则。以下列表旨在提供基本思想，并不旨在详尽无遗。

# 加密通信

首先，重要的是要提到，在互联网上发生的所有外部通信都必须加密。通常的做法是通过 TLS 使用受信任的证书来完成。这对于 HTTP 以及其他通信协议都是可能的。

使用的证书的真实性必须在运行时由实现进行验证。它们必须由受信任的内部或外部证书颁发机构保证。

在应用程序中不安全地接受任何证书应避免，无论是生产环境还是其他环境。这意味着提供了并使用了正确签名的证书来进行通信。

# 委派安全关注

在存储用户信息方面，今天的方法是在可能的情况下将身份验证和授权委派给安全提供者。这意味着企业应用程序不会存储安全信息，而是请求第三方，一个受信任的安全提供者。

在分布式环境中，这一点尤其有趣，因为多个应用程序为外部世界提供了潜在的端点。安全信息移动到单一的责任点。

安全问题通常不是核心业务逻辑的一部分。应用程序将请求受信任的安全提供者系统验证用户请求的安全性。安全提供者充当安全单一责任点。

存在着去中心化的安全协议，例如 **OAuth** 或 **OpenID**，它们实现了这种方法。

将责任委托给受信任的安全提供商消除了在企业系统中共享密码的需要。用户直接针对安全提供商进行身份验证。需要了解用户安全信息的应用程序将提供不直接包含机密数据的会话令牌。

然而，这一原则主要针对包括应用程序用户作为个人的通信。

# 正确处理用户凭证

如果由于某种原因应用程序自行管理用户认证，它绝不应该永久以明文形式存储密码和令牌。这引入了严重的安全风险。即使应用程序或数据库对外部世界有足够的保护，保护凭证免受内部泄露也很重要。

需要在应用程序内部管理的密码必须仅通过适当的哈希算法和如**盐值**等方法存储。这样做可以防止来自公司内部和外部任何恶意攻击。建议咨询像**开放网络应用安全项目**（**OWASP**）这样的安全信息组织。它们提供关于安全方法和算法的现代建议。

# 避免在版本控制中存储凭证

正如你不应该轻视安全凭证一样，开发者也不应该在版本控制的项目仓库中存储明文凭证。即使仓库是在公司内部托管，这也引入了安全风险。

凭证将永久地显示在仓库的历史记录中。

如第五章“使用 Java EE 的容器和云环境”所示，云环境具有将秘密配置值注入应用程序的功能。此功能可用于提供外部配置的秘密凭证。

# 包含测试

应用程序需要负责的安全机制需要得到适当的系统测试。任何包含的认证和授权都必须作为持续交付管道的一部分进行验证。这意味着你应该在自动化测试中验证功能，不仅验证一次，而且在软件更改后持续验证。

对于与安全相关的测试，包括负面测试尤为重要。例如，测试必须验证错误的凭证或权限不足不允许你执行特定的应用程序功能。

# 可能性和解决方案

在几个基本但重要的安全原则之后，让我们来看看可能的安全协议和解决方案。

# 加密通信

加密通信通常意味着通信使用**TLS 加密**，作为传输层通信协议的一部分。证书用于加密和签名通信。当然，能够依赖证书至关重要。

公司通常运营自己的证书颁发机构，并在他们的计算机和软件中预先安装**根 CA**。这对于内部网络来说确实是有道理的。与从官方机构请求所有内部服务的证书相比，这减少了开销和潜在的成本。

需要由操作系统或平台预先安装的官方证书颁发机构之一签名的**公开受信任**的证书。

加密通信不验证用户，除非正在使用单个客户端证书。它为安全、可信的通信奠定了基础。

# 基于协议的认证

一些通信协议自带认证功能，例如带有基本或摘要认证的 HTTP。这些功能是通信协议的一部分，通常在工具和框架中得到很好的支持。

它们通常依赖于通信已经被安全加密，否则这将使信息对能够读取它的各方可访问，如果他们拦截了通信。这一点对于确保通过加密通信提供基于协议的认证，对于应用程序开发者来说非常重要。

基于协议的安全凭证通常直接在每条消息中提供。这简化了客户端调用，因为不需要进行多个认证步骤，例如在交换令牌时。第一个客户端调用就可以交换信息。

# 去中心化安全

其他不直接在客户端调用中包含凭证的方法会首先获取安全令牌，然后在提供令牌之后发出实际通信。这朝着去中心化安全的方向发展。

为了将安全与应用程序解耦，企业系统可以将身份提供者作为认证或授权的中心点，分别包括在内。这把安全担忧从应用程序委托给提供者。

身份提供者授权第三方，如企业应用程序，而无需与他们直接交换凭证。最终用户被重定向到身份提供者，不会将安全信息交给企业应用程序。第三方只有在获得访问权限时才会收到信息，这些信息包含在它们可以验证的令牌中。

这种三方认证避免了让企业应用程序承担安全责任。验证用户提供的资料是否正确，责任转移到身份提供者。

这种方法的例子之一是**单点登录**（**SSO**）机制。它们在大公司中相当常用，要求用户只进行一次身份验证，并在所有由 SSO 保护的服务中重复使用信息。SSO 系统验证用户并向相应的应用程序提供所需用户信息。用户只需登录一次。

另一种方法是使用分散式访问委托协议，例如 OAuth、OpenID 和 OpenID Connect。它们代表客户端、第三方应用程序和身份提供者之间交换安全信息的三方安全工作流程。这种想法与单点登录机制类似。然而，这些协议允许用户决定哪个单独的应用程序将接收用户信息。应用程序接收用户访问令牌，例如，以**JSON Web Tokens**的形式，这些令牌通过身份提供者进行验证，而不是实际的凭证。

分散式访问委托协议及其实现超出了本书的范围。企业系统的责任是拦截并重定向用户身份验证到身份提供者。根据系统架构，这可能是由代理服务器或应用程序本身负责。

现在有开源解决方案实现了分散式安全。一项有趣的技术是**Keycloak**，它是一个身份和访问管理解决方案。它附带各种客户端适配器，并支持标准协议，如 OAuth 或 OpenID Connect，这使得保护应用程序和服务变得容易。

# 代理

封装与企业应用程序通信的代理服务器可以增加安全方面，例如加密通信。例如，Web 代理服务器支持通过 HTTPS 的 TLS 加密。

问题是工程师是否想要在网络、内部和外部通信之间做出区分。内部网络中的通信通常是未加密的。根据交换信息的性质，在大多数情况下，互联网通信应该是加密的。

代理服务器可以用于在网络边界终止加密，即所谓的**TLS 终止**。代理服务器分别加密所有传出信息和解密所有传入信息。

同样有可能使用不同网络的不同证书重新加密通信。

# 现代环境中的集成

现代环境旨在支持今天的网络安全需求。容器编排框架提供软件代理服务器和网关的配置，这些服务器和网关公开服务；例如，Kubernetes 的**ingress**资源以及 OpenShift 的**routes**支持集群外部流量的 TLS 加密。

为了提供诸如凭证或私钥之类的秘密值，编排框架提供了“秘密”功能。如前所述，这使我们能够将秘密配置分别提供到环境中。第五章，*Java EE 的容器和云环境*探讨了这是如何实现的。

这使得应用程序以及一般的配置可以使用秘密值。如果需要，这些秘密值可以注入到容器运行时。

# 在 Java EE 应用程序中实现安全

在了解了当今世界最常见的安全方法之后，让我们看看如何使用 Java EE 实现安全。

在所有 Java 版本中，Java EE 版本 8 旨在解决安全方面的问题。它包含一个安全 API，该 API 简化并统一了开发者的集成。

# 透明安全

在最简单的情况下，可以通过代理 Web 服务器，如**Apache**或**nginx**，在 Web 应用程序中实现安全。在这种情况下，安全责任对应用程序来说是透明的。

如果企业应用程序不需要处理用户作为域实体，这种情况通常会出现。

# Servlets

为了保护 Java EE 应用程序提供的 Web 服务，通常在 servlet 层使用安全。对于所有建立在 servlet 之上的技术，如 JAX-RS，也是如此。安全功能是通过 servlet 部署描述符配置的，即`web.xml`文件。

这可以通过多种方式发生，例如基于表单的认证、HTTP 基本访问认证或客户端证书。

类似地，像 Keycloak 这样的安全解决方案会提供自己的适配器和 servlet 过滤器的实现。开发者通常只需要配置这些组件以使用安全提供者。

# Java 受保护实体和角色

Java 安全受保护实体和角色分别代表身份和授权角色。受保护实体和角色通常以厂商特定的方式在应用服务器中配置。在执行过程中，认证请求绑定到受保护实体。

在执行工作流程中使用相关角色的一个例子是使用常见的安全注解，如`@RolesAllowed`。这种声明式方法检查受保护实体是否被正确授权，否则将导致安全异常：

```java
import javax.annotation.security.RolesAllowed;

@Stateless
public class CarManufacturer {

    ...

    @RolesAllowed("worker")
    public Car manufactureCar(Specification spec) {
        ...
    }

    @RolesAllowed("factory-admin")
    public void reconfigureMachine(...) {
        ...
    }
```

除了厂商特定的解决方案之外，用户和角色可以扩展以包含特定领域的相关信息。为了实现这一点，`Principal`安全类型得到了增强。

可以注入通过其名称识别的受保护实体，并提供一个特殊化。容器负责用户识别，例如，通过使用基于表单的认证。

这种方法在 Java EE 版本 8 之前特别推荐。然而，现代应用程序可能会使用身份存储来表示特定领域的用户信息。

# JASPIC

**Java 容器身份验证服务提供者接口**（**JASPIC**）是一个定义身份验证服务提供者接口的标准。它包括所谓的**服务器身份验证模块**（**SAM**），可插入的身份验证组件，这些组件被添加到应用程序服务器中。

本标准提供了强大且灵活的方式来实现身份验证。服务器供应商可以提供他们自己的 SAMs 实现。然而，许多开发者认为使用 JASPIC 标准实现身份验证模块相当繁琐。这就是为什么 JASPIC 标准在企业项目中并不广泛使用。

# 安全 API

安全 API 1.0 与 Java EE 8 一起发布。这个标准的想法是为开发者提供更简单易用的现代安全方法。这些方法以供应商独立的方式实现，无需锁定到特定解决方案。

让我们来看看安全 API 包括哪些内容。

# 身份验证机制

首先，安全 API 包括`HttpAuthenticationMechanism`，它以较少的开发工作量提供了 JASPIC 标准的特性。它被指定用于 servlet 上下文。

应用程序开发者只需要定义一个自定义的`HttpAuthenticationModule`并在`web.xml`部署描述符中配置身份验证。我们将在本章后面讨论自定义安全实现。

Java EE 容器已经预装了基本、默认和自定义表单身份验证的预定义 HTTP 身份验证机制。开发者可以以最小的努力使用这些预定义功能。在我们看到示例之前，让我们看看如何存储用户信息。

# 身份存储

安全 API 中也增加了身份存储的概念。身份存储以轻量级、便携的方式提供用户的身份验证和授权信息。它们提供了一种统一的方式来访问这些信息。

`IdentityStore`类型验证调用者的凭证并访问其信息。类似于 HTTP 身份验证机制，应用程序容器需要为 LDAP 和数据库访问提供身份存储。

下面的示例展示了使用容器提供的安全功能的一个例子：

```java
import javax.security.enterprise.authentication.mechanism.http.*;
import javax.security.enterprise.identitystore.DatabaseIdentityStoreDefinition;
import javax.security.enterprise.identitystore.IdentityStore;

@BasicAuthenticationMechanismDefinition(realmName = "car-realm")
@DatabaseIdentityStoreDefinition(
        dataSourceLookup = "java:comp/UserDS",
        callerQuery = "select password from users where name = ?",
        useFor = IdentityStore.ValidationType.VALIDATE
)
public class SecurityConfig {
    // nothing to configure
}
```

应用程序开发者只需要提供这个注解类。这种方法为测试目的提供了简单直接的安全定义。

通常，企业项目可能需要更多的自定义方法。组织通常有自己的身份验证和授权方式，这些方式需要集成。

# 自定义安全

下面的示例展示了更复杂的情况。

为了提供自定义认证，应用程序开发者实现了一个自定义的`HttpAuthenticationMechanism`，特别是`validateRequest()`方法。该类只需作为 CDI bean 对容器可见即可。其余的工作由应用程序容器完成。这简化了开发者的安全集成。

以下是一个基本示例，其中使用**伪代码**表示实际的认证：

```java
import javax.security.enterprise.AuthenticationException;
import javax.security.enterprise.authentication.mechanism.http.*;
import javax.security.enterprise.credential.UsernamePasswordCredential;
import javax.security.enterprise.identitystore.CredentialValidationResult;
import javax.security.enterprise.identitystore.IdentityStoreHandler;

@ApplicationScoped
public class TestAuthenticationMechanism implements
        HttpAuthenticationMechanism {

    @Inject
    IdentityStoreHandler identityStoreHandler;

    @Override
    public AuthenticationStatus validateRequest(HttpServletRequest request,
            HttpServletResponse response,
            HttpMessageContext httpMessageContext)
            throws AuthenticationException {

        // get the authentication information
        String name = request.get...
        String password = request.get...

        if (name != null && password != null) {

            CredentialValidationResult result = identityStoreHandler
                    .validate(new UsernamePasswordCredential(name,
                    password));

            return httpMessageContext.notifyContainerAboutLogin(result);
        }

        return httpMessageContext.doNothing();
    }
}
```

`validateRequest()`实现访问 HTTP 请求中包含的用户信息，例如通过 HTTP 头。它使用`IdentityStoreHandler`将验证委托给身份存储库。验证结果包含提供给安全 HTTP 消息上下文的结果。

根据要求，还需要实现自定义身份处理程序实现。它可以提供自定义认证和授权方法。

如果使用去中心化安全协议，如 OAuth，则自定义身份处理程序将实现安全访问令牌验证。

以下显示了一个自定义身份存储库：

```java
import javax.security.enterprise.identitystore.IdentityStore;

@ApplicationScoped
public class TestIdentityStore implements IdentityStore {

    public CredentialValidationResult validate(UsernamePasswordCredential
            usernamePasswordCredential) {

        // custom authentication or authorization
        // if valid

        return new CredentialValidationResult(username, roles);

        // or in case of invalid credentials

        return CredentialValidationResult.INVALID_RESULT;
    }
}
```

使用`web.xml` servlet 部署描述符来指定受保护资源。应用程序容器负责集成：

```java
<security-constraint>
    <web-resource-collection>
        <web-resource-name>Protected pages</web-resource-name>
        <url-pattern>/management</url-pattern>
    </web-resource-collection>
    <auth-constraint>
        <role-name>admin-role</role-name>
    </auth-constraint>
</security-constraint>
```

HTTP 认证机制提供了一种简单而灵活的方式来实现 JASPIC 安全。与纯 JASPIC 方法相比，其实现更简单。

它提供了拦截通信流的可能性，可以将应用程序与第三方安全提供商集成。

# 访问安全信息

企业应用程序有时需要功能来访问用户授权信息，作为业务逻辑的一部分。安全 API 使我们能够以统一的方式检索这些信息。

它包含`SecurityContext`类型，该类型提供了一种程序化方式来检索有关调用者主体及其角色的信息。`SecurityContext`可以注入到任何管理 bean 中。它还与 servlet 认证配置集成，并提供有关调用者是否允许访问特定 HTTP 资源的信息。

以下显示了`SecurityContext`的一个示例用法：

```java
import javax.security.enterprise.SecurityContext;

@Stateless
public class CompanyProcesses {

    @Inject
    SecurityContext securityContext;

    public void executeProcess() {
        executeUserProcess();
        if (securityContext.isCallerInRole("admin")) {
            String name = securityContext.getCallerPrincipal().getName();
            executeAdminProcess(name);
        }
    }

    ...
}
```

安全 API 的想法是与之前 Java EE 版本中的现有功能集成。这意味着，例如，`@RolesAllowed`注解使用与`SecurityContext`相同的角色信息。开发者可以继续依赖现有的标准功能。

# 摘要

在当今世界，IT 安全是一个非常重要的方面。在过去，一些最大的安全问题包括弱加密和散列算法、密码的持久化方式以及自制的安全实现。一些重要的安全原则包括加密通信、使用外部、可信的安全提供者进行认证和授权、避免在版本控制下保留凭证，以及包括验证保护的测试场景。

通信通常在传输层使用 TLS 进行加密。使用的证书应该由公司内部或官方证书机构正确签名。其他方法包括使用协议层的安全功能，例如在加密通信之上使用 HTTP 基本认证。

通过包含可信的身份提供者，去中心化安全将认证和授权责任从应用程序中分离出来。单点登录以及去中心化访问委托协议是此类示例。

在 Java EE 应用程序边界内实现安全通常是在 Servlets 之上进行的。Java EE 8 中引入的 Security API 旨在提供更简单、统一的方法来处理 Java EE 应用程序中的安全问题。HTTP 认证机制是提供更易用 JASPIC 功能的示例。身份存储提供用户的认证和授权信息。

Security API 的理念是与现有功能集成并提供统一的访问机制。包含的功能应该足够保护企业应用程序在 HTTP 方面的安全。
