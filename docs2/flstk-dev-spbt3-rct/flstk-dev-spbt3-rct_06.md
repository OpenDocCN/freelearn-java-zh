# 5

# 保护您的后端

本章解释了如何保护您的 Spring Boot 后端。保护后端是代码开发的关键部分。对于保护敏感数据、遵守法规和防止未经授权的访问至关重要。后端通常处理用户认证和授权过程。正确保护这些方面确保只有授权用户可以访问应用程序并执行特定操作。我们将以前一章创建的数据库应用程序作为起点。

在本章中，我们将涵盖以下主题：

+   理解 Spring Security

+   使用 JSON Web Token 保护您的后端

+   基于角色的安全

+   使用 OAuth2 与 Spring Boot

# 技术要求

我们在前几章中创建的 Spring Boot 应用程序是必需的。

以下 GitHub 链接也将是必需的：[`github.com/PacktPublishing/Full-Stack-Development-with-Spring-Boot-3-and-React-Fourth-Edition/tree/main/Chapter05`](https://github.com/PacktPublishing/Full-Stack-Development-with-Spring-Boot-3-and-React-Fourth-Edition/tree/main/Chapter05)。

# 理解 Spring Security

**Spring Security** (https://spring.io/projects/spring-security) 为基于 Java 的 Web 应用程序提供安全服务。Spring Security 项目始于 2003 年，之前被称为 *Acegi Security System for Spring*。

默认情况下，Spring Security 启用以下功能：

+   一个具有内存中单个用户的 `AuthenticationManager` 实例。用户名为 `user`，密码打印到控制台输出。

+   忽略常见静态资源位置的路径，例如 `/css` 和 `/images`。其他所有端点的 HTTP 基本认证。

+   安全事件发布到 Spring 的 `ApplicationEventPublisher` 接口。

+   默认启用的常见低级功能，包括 **HTTP Strict Transport Security** (**HSTS**), **跨站脚本** (**XSS**), 和 **跨站请求伪造** (**CSRF**)。

+   默认自动生成的登录页面。

您可以通过将以下突出显示的依赖项添加到 `build.gradle` 文件中来将 Spring Security 包含到您的应用程序中。第一个依赖项是用于应用程序的，第二个是用于测试的：

```java
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-data-rest'
    **implementation** **'org.springframework.boot:spring-boot-starter-security'**
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    runtimeOnly 'org.mariadb.jdbc:mariadb-java-client'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    **testImplementation** **'org.springframework.security:spring-security-test'**
} 
```

如果您尚未启用自动刷新，请记住在修改了您的 `build.gradle` 文件后，从 Eclipse 中刷新 Gradle 项目。

当您启动应用程序时，您可以从控制台看到 Spring Security 已创建一个用户名为 `user` 的内存中用户。用户的密码可以在控制台输出中看到，如图所示：

![](img/B19818_05_01.png)

图 5.1：Spring Security 已启用

如果控制台中没有密码，请尝试通过在控制台中按下红色 **终止** 按钮重新启动您的项目并重新运行它。

Eclipse 控制台输出有限，默认缓冲区大小为 80,000 个字符，因此输出可能在密码语句出现之前被截断。您可以从**窗口 | 首选项 | 运行/调试 | 控制台**菜单更改此设置。

现在，如果您向您的 REST API 根端点发送一个`GET`请求，您将看到它已被保护。打开您的网络浏览器并导航到`http://localhost:8080/api`。您将被重定向到 Spring Security 默认登录页面，如下面的截图所示：

![图片](img/B19818_05_02.png)

图 5.2：受保护的 REST API

为了能够成功发送`GET`请求，我们必须对我们的 RESTful API 进行身份验证。在**用户名**字段中输入`user`，并将从控制台生成的密码复制到**密码**字段。通过身份验证，我们可以看到响应包含我们的 API 资源，如下面的截图所示：

![图片](img/B19818_05_03.png)

图 5.3：基本身份验证

为了配置 Spring Security 的行为，我们必须为 Spring Security 添加一个新的配置类。安全配置文件可以用来定义哪些 URL 或 URL 模式对哪些角色或用户是可访问的。您还可以定义认证机制、登录过程、会话管理等。

在您的应用程序根包（`com.packt.cardatabase`）中创建一个名为`SecurityConfig`的新类。以下源代码显示了安全配置类的结构：

```java
package com.packt.cardatabase;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.configuration.
  EnableWebSecurity;
@Configuration
@EnableWebSecurity
public class SecurityConfig {
} 
```

`@Configuration`和`@EnableWebSecurity`注解关闭了默认的 Web 安全配置，我们可以在此类中定义自己的配置。在稍后我们将看到其作用的`filterChain(HttpSecurity http)`方法中，我们可以定义我们的应用程序中哪些端点是安全的，哪些不是。实际上我们目前不需要这个方法，因为我们可以在所有端点都受保护的情况下使用默认设置。

我们还可以通过使用 Spring Security 的`InMemoryUserDetailsManager`（它实现了`UserDetailsService`）将内存中的用户添加到我们的应用程序中。然后我们可以实现存储在内存中的用户/密码认证。我们还可以使用`PasswordEncoder`通过`bcrypt`算法对密码进行编码。

下面的高亮源代码将创建一个名为`user`、密码为`password`、角色为`USER`的内存中用户：

```java
// SecurityConfig.java
package com.packt.cardatabase;
**import** **org.springframework.context.annotation.Bean;**
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.configuration.
  EnableWebSecurity;
**import** **org.springframework.security.core.userdetails.User;**
**import** **org.springframework.security.core.userdetails.UserDetails;**
**import** **org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;**
**import** **org.springframework.security.crypto.password.PasswordEncoder;**
**import** **org.springframework.security.provisioning.****InMemoryUserDetailsManager;**
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    **@Bean**
    **public** **InMemoryUserDetailsManager** **userDetailsService****()** **{**
        **UserDetails****user****=** **User.builder().username(****"user"****).**
            **password(passwordEncoder().encode(****"password"****))**
            **.roles(****"USER"****).build();**
        **return****new****InMemoryUserDetailsManager****(user);**
    **}**
    **@Bean**
    **public** **PasswordEncoder** **passwordEncoder****()** **{**
        **return****new****BCryptPasswordEncoder****();**
    **}**
} 
```

现在，重新启动应用程序，您将能够使用内存中的用户进行身份验证测试。在开发阶段使用内存中的用户是可以的，但一个合适的应用程序应该将用户保存在数据库中。

要将用户保存到数据库中，你必须创建一个用户实体类和存储库。密码不应以明文格式保存到数据库中。如果包含用户密码的数据库被黑客攻击，攻击者将能够直接以明文形式获取密码。Spring Security 提供了多种哈希算法，例如 `bcrypt`，你可以使用这些算法来哈希密码。以下步骤展示了如何实现这一点：

1.  在 `com.packt.cardatabase.domain` 包中创建一个名为 `AppUser` 的新类。激活领域包，然后右键单击它。从菜单中选择 **新建** | **类**，并将新类命名为 `User`。之后，你的项目结构应该看起来像这样：

![](img/B19818_05_04.png)

图 5.4：项目结构

1.  使用 `@Entity` 注解标注 `AppUser` 类。添加 ID、用户名、密码和角色类字段。最后，添加构造函数、获取器和设置器。我们将所有字段设置为不可为空。这意味着数据库列不能包含 `null` 值。我们还将通过在用户名的 `@Column` 注解中使用 `unique=true` 来指定用户名必须是唯一的。请参考以下 `AppUser.java` 源代码以了解字段：

    ```java
    package com.packt.cardatabase.domain;
    import jakarta.persistence.Column;
    import jakarta.persistence.Entity;
    import jakarta.persistence.GeneratedValue;
    import jakarta.persistence.GenerationType;
    import jakarta.persistence.Id;
    @Entity
    public class AppUser {
        @Id
        @GeneratedValue(strategy=GenerationType.AUTO)
        @Column(nullable=false, updatable=false)
        private Long id;

        @Column(nullable=false, unique=true)
        private String username;

        @Column(nullable=false)
        private String password;

        @Column(nullable=false)
        private String role;

    // Constructors, getters and setters    
    } 
    ```

    这里是 `AppUser.java` 构造函数的源代码：

    ```java
     public AppUser() {}
        public AppUser(String username, String password, String role) {
            super();
            this.username = username;
            this.password = password;
            this.role = role;
        } 
    ```

    这里是带有获取器和设置器的 `AppUser.java` 源代码：

    ```java
     public Long getId() {
            return id;
        }
        public void setId(Long id) {
            this.id = id;
        }
        public String getUsername() {
            return username;
        }
        public void setUsername(String username) {
            this.username = username;
        }
        public String getPassword() {
            return password;
        }
        public void setPassword(String password) {
            this.password = password;
        }
        public String getRole() {
            return role;
        }
        public void setRole(String role) {
            this.role = role;
        } 
    ```

1.  在领域包中创建一个名为 `AppUserRepository` 的新接口。为此，激活领域包，然后右键单击它。从菜单中选择 **新建** | **接口**，并将其命名为 `AppUserRepository`。

    存储库类的源代码与我们之前章节中看到的大致相同，但有一个名为 `findByUsername` 的查询方法，这是我们接下来需要用到的。此方法用于在认证过程中从数据库中查找 `用户`。该方法返回 `Optional` 以防止空指针异常。请参考以下 `AppUserRepository` 源代码：

    ```java
    package com.packt.cardatabase.domain;
    import java.util.Optional;
    import org.springframework.data.repository.CrudRepository;
    public interface AppUserRepository extends CrudRepository
       <AppUser, Long> {
        Optional<AppUser> findByUsername(String username);
      } 
    ```

1.  接下来，我们将创建一个实现 Spring Security 提供的 `UserDetailsService` 接口的类。Spring Security 使用它来进行用户认证和授权。在根包中创建一个新的 `service` 包。为此，激活根包，然后右键单击它。从菜单中选择 **新建** | **包**，并将其命名为 `service`，如图所示：

![](img/B19818_05_05.png)

图 5.5：服务包

1.  在我们刚刚创建的 `service` 包中创建一个名为 `UserDetailsServiceImpl` 的新类。现在，你的项目结构应该看起来像这样（在 Eclipse 中，通过按 *F5* 刷新项目资源管理器）：

![](img/B19818_05_06.png)

图 5.6：项目结构

1.  我们必须将`AppUserRepository`类注入到`UserDetailsServiceImpl`类中，因为当 Spring Security 处理认证时需要从数据库中获取用户。我们之前实现的`findByUsername`方法返回`Optional`，因此我们可以使用`isPresent()`方法来检查`user`是否存在。如果`user`不存在，我们抛出`UsernameNotFoundException`异常。`loadUserByUsername`方法返回`UserDetails`对象，这是认证所必需的。我们使用 Spring Security 的`UserBuilder`类来构建用于认证的用户。以下是`UserDetailsServiceImpl.java`的源代码：

    ```java
    package com.packt.cardatabase.service;
    import java.util.Optional;
    import org.springframework.security.core.userdetails.User.
      UserBuilder;
    import org.springframework.security.core.userdetails.UserDetails;
    import org.springframework.security.core.userdetails.
      UserDetailsService;
    import org.springframework.security.core.userdetails.
      UsernameNotFoundException;
    import org.springframework.stereotype.Service;
    import com.packt.cardatabase.domain.AppUser;
    import com.packt.cardatabase.domain.AppUserRepository;
    @Service
    public class UserDetailsServiceImpl implements UserDetailsService {
      private final AppUserRepository repository;
    public UserDetailsServiceImpl(AppUserRepository repository) {
        this.repository = repository;
    }
        @Override
        public UserDetails loadUserByUsername(String username) throws
        UsernameNotFoundException {
            Optional<AppUser> user = repository.findByUsername(username);
            UserBuilder builder = null;
            if (user.isPresent()) {
                AppUser currentUser = user.get();
                builder = org.springframework.security.core.userdetails.
                          User.withUsername(username);
                builder.password(currentUser.getPassword());
                builder.roles(currentUser.getRole());
            } else {
                throw new UsernameNotFoundException("User not found.");
            }
            return builder.build();
        }
    } 
    ```

    在我们的安全配置类中，我们必须指定 Spring Security 应使用数据库中的用户而不是内存中的用户。从`SecurityConfig`类中删除`userDetailsService()`方法以禁用内存中的用户。添加一个新的`configureGlobal`方法以启用数据库中的用户。

    我们永远不应该将密码以明文形式保存到数据库中。因此，我们将在`configureGlobal`方法中定义一个密码散列算法。在这个例子中，我们使用`bcrypt`算法。这可以通过 Spring Security 的`BCryptPasswordEncoder`类轻松实现，该类在认证过程中编码散列密码。以下是`SecurityConfig.java`的源代码：

    ```java
    package com.packt.cardatabase;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.Bean;
    import org.springframework.security.config.annotation.
      authentication.builders.AuthenticationManagerBuilder;
    import org.springframework.security.config.annotation.
      web.configuration.EnableWebSecurity;
    import org.springframework.security.crypto.bcrypt.
      BCryptPasswordEncoder;
    import com.packt.cardatabase.service.UserDetailsServiceImpl;
    import org.springframework.security.crypto.password.PasswordEncoder;
    @Configuration
    @EnableWebSecurity
    public class SecurityConfig {
        private final UserDetailsServiceImpl userDetailsService;
        public SecurityConfig(UserDetailsServiceImpl userDetailsService) {
            this.userDetailsService = userDetailsService;
        }
      public void configureGlobal (AuthenticationManagerBuilder auth)
        throws Exception {
          auth.userDetailsService(userDetailsService)
          .passwordEncoder(new BCryptPasswordEncoder());
        }
        @Bean
        public PasswordEncoder passwordEncoder() {
            return new BCryptPasswordEncoder();
        }
    } 
    ```

    现在，在将密码保存到数据库之前，必须使用`bcrypt`对其进行散列。

1.  最后，我们可以使用`CommandLineRunner`接口将几个测试用户保存到数据库中。打开`CardatabaseApplication.java`文件，并将`AppUserRepository`注入到主类中：

    ```java
    private final CarRepository repository;
    private final OwnerRepository orepository;
    **private****final** **AppUserRepository urepository;**
    public CardatabaseApplication(CarRepository repository, OwnerRepository orepository**, AppUserRepository urepository**) {
        this.repository = repository;
        this.orepository = orepository;
        **this****.urepository = urepository;**
    } 
    ```

1.  让我们使用`bcrypt`散列密码将两个用户保存到数据库中。你可以在互联网上找到`bcrypt`计算器或生成器。这些生成器允许你输入明文密码，然后它们将生成相应的`bcrypt`散列：

    ```java
     @Override
        public void run(String... args) throws Exception {
            // Add owner objects and save these to db
            Owner owner1 = new Owner("John", "Johnson");
            Owner owner2 = new Owner("Mary", "Robinson");
            orepository.saveAll(Arrays.asList(owner1, owner2));
            repository.save(new Car(
                                "Ford", "Mustang", "Red", "ADF-1121",
                                 2023, 59000, owner1));
            repository.save(new Car(
                                "Nissan", "Leaf", "White", "SSJ-3002",
                                2020, 29000, owner2));
            repository.save(new Car(
                                "Toyota", "Prius", "Silver", "KKO-0212",
                                2022, 39000, owner2));
            // Fetch all cars and log to console
            for (Car car : repository.findAll()) {
                logger.info(car.getBrand() + " " + car.getModel());
            }
            **// Username: user, password: user**
            **urepository.save(****new****AppUser****(****"user"****,**
                **"$2a$10$NVM0n8ElaRgg7zWO1CxUdei7vWoPg91Lz2aYavh9.**
                **f9q0e4bRadue"****,****"****USER"****));**
            **// Username: admin, password: admin**
            **urepository.save(****new****AppUser****(****"admin"****,**
                **"$2a$10$8cjz47bjbR4Mn8GMg9IZx.vyjhLXR/SKKMSZ9.**
                **mP9vpMu0ssKi8GW"****,** **"ADMIN"****));**
        } 
    ```

    **bcrypt**是一种由 Niels Provos 和 David Mazières 设计的强大散列函数。以下是从`admin`字符串生成的 bcrypt 散列的示例：

    `$2a$10$8cjz47bjbR4Mn8GMg9IZx.vyjhLXR/SKKMSZ9.mP9vpMu0ssKi8GW`

    `$2a`代表算法版本，`$10`代表算法的强度。Spring Security 的`BcryptPasswordEncoder`类的默认强度是 10。bcrypt 在散列过程中生成一个随机的**盐**，因此散列结果总是不同的。

1.  运行你的应用程序后，你会看到现在数据库中有一个`app_user`表，并且有两个用户记录以散列密码的形式保存，如下面的截图所示：

![](img/B19818_05_07.png)

图 5.7：用户

1.  现在，你应该重新启动应用程序，如果你尝试在不进行认证的情况下向 `http://localhost:8080/api` 路径发送 `GET` 请求，你会得到一个 `401 未授权` 错误。你必须进行认证才能发送成功的请求。与上一个示例相比，区别在于我们现在使用数据库中的用户进行认证。

    现在，你可以通过使用浏览器向 `/api` 端点发送 `GET` 请求来登录，或者我们可以使用 Postman 和基本认证，如下面的截图所示：

    ![](img/B19818_05_08.png)

    图 5.8：GET 请求认证

1.  你可以看到，我们目前通过在我们的 RESTful 网络服务中调用 `api/appUsers` 端点来获取用户，这是我们想要避免的。如 *第四章* 中所述，Spring Data REST 默认会从所有公共仓库生成 RESTful 网络服务。我们可以使用 `@RepositoryRestResource` 注解的 `exported` 标志，并将其设置为 `false`，这意味着以下仓库不会作为 REST 资源暴露：

    ```java
    package com.packt.cardatabase.domain;
    import java.util.Optional;
    import org.springframework.data.repository.CrudRepository;
    **import** **org.springframework.data.rest.core.annotation.**
     **RepositoryRestResource;**
    **@RepositoryRestResource(exported = false)**
    public interface AppUserRepository extends CrudRepository
       <AppUser, Long> {
        Optional<AppUser> findByUsername(String username);
      } 
    ```

1.  现在，如果你重新启动应用程序并向 `/api` 端点发送一个 `GET` 请求，你会看到 `/appUsers` 端点不再可见。

接下来，我们将开始使用 JSON Web Token 实现认证。

# 使用 JSON Web Token 保护你的后端

在上一节中，我们介绍了如何使用基本认证来与 RESTful 网络服务交互。基本认证不提供处理令牌或管理会话的方法。当用户登录时，凭证会随着每个请求发送，这可能导致会话管理挑战和潜在的安全风险。当我们使用 React 开发自己的前端时，这种方法不可用，因此我们将使用 **JSON Web Token**（**JWT**）认证代替（https://jwt.io/）。这也会让你了解如何更详细地配置 Spring Security。

保护你的 RESTful 网络服务的另一种选择是 **OAuth 2**。OAuth2（https://oauth.net/2/）是行业标准的授权，它可以在 Spring Boot 应用程序中非常容易地使用。本章后面有一个部分将给你一个基本的概念，了解如何在你的应用程序中使用它。

JWTs 常用于 RESTful API 的认证和授权目的。它们是实现现代网络应用认证的一种紧凑方式。JWT 非常小，因此可以发送在 URL 中、`POST` 参数中或头信息中。它还包含有关用户的所有必要信息，例如他们的用户名和角色。

JWT 包含三个不同的部分，由点分隔：`xxxxx.yyyyy.zzzzz`。这些部分如下划分：

+   第一部分（`xxxxx`）是 **头部**，它定义了令牌的类型和哈希算法。

+   第二部分（`yyyyy`）是 **负载**，通常情况下，在认证的情况下，它包含用户信息。

+   第三部分（`zzzzz`）是**签名**，它用于验证令牌在传输过程中没有被更改。

这是一个 JWT 的示例：

```java
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c 
```

以下图显示了使用 JWT 的简化认证过程表示：

![](img/B19818_05_09.png)

图 5.9：JWT 认证过程

在认证成功后，客户端发送的请求应始终包含在认证中接收到的 JWT。

我们将使用`jjwt`（https://github.com/jwtk/jjwt），这是 Java 和 Android 的 JWT 库，用于创建和解析 JWT。因此，我们必须将以下依赖项添加到`build.gradle`文件中：

```java
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-data-rest'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    **implementation** **'io.jsonwebtoken:jjwt-api:0.11.5'**
    **runtimeOnly** **'io.jsonwebtoken:jjwt-impl:0.11.5'****,** **'io.jsonwebtoken:jjwt-**
      **jackson:0.11.5'**
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    runtimeOnly 'org.mariadb.jdbc:mariadb-java-client'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.security:spring-security-test'
} 
```

在你更新了依赖项之后，记得从 Eclipse 刷新 Gradle 项目。

以下步骤演示了如何在我们的后端启用 JWT 认证。

## 保护登录

我们将从登录功能开始：

1.  首先，我们将创建一个类来生成和验证已签名的 JWT。在`com.packt.cardatabase.service`包中创建一个新的类名为`JwtService`。在类的开头，我们将定义一些常量：`EXPIRATIONTIME`定义了令牌的过期时间（以毫秒为单位），`PREFIX`定义了令牌的前缀，通常使用`"Bearer"`模式。JWT 在`Authorization`头中发送，当使用 Bearer 模式时，头的样子如下：

    ```java
    Authorization: Bearer <token> 
    ```

    `JwtService`源代码看起来如下：

    ```java
    package com.packt.cardatabase.service;
    import org.springframework.stereotype.Component;
    @Component
    public class JwtService {
      static final long EXPIRATIONTIME = 86400000;   // 1 day in ms. Should be shorter in production.
      static final String PREFIX = "Bearer";
    } 
    ```

1.  我们将使用`jjwt`库的`secretKeyFor`方法创建一个密钥。这只是为了演示目的。在生产环境中，你应该从应用程序配置中读取你的密钥。然后，`getToken`方法生成并返回令牌。`getAuthUser`方法从响应的`Authorization`头中获取令牌。接下来，我们将使用`jjwt`库提供的`parserBuilder`方法创建一个`JwtParserBuilder`实例。`setSigningKey`方法用于指定用于令牌验证的密钥。`parseClaimsJws`方法从`Authorization`头中移除`Bearer`前缀。最后，我们将使用`getSubject`方法获取用户名。整个`JwtService`源代码如下：

    ```java
    package com.packt.cardatabase.service;
    **import** **io.jsonwebtoken.Jwts;**
    **import** **io.jsonwebtoken.SignatureAlgorithm;**
    **import** **io.jsonwebtoken.security.Keys;**
    **import** **java.security.Key;**
    **import** **org.springframework.http.HttpHeaders;**
    **import** **org.springframework.stereotype.Component;**
    **import** **jakarta.servlet.http.HttpServletRequest;**
    **import** **java.util.Date;**
    @Component
    public class JwtService {
      static final long EXPIRATIONTIME = 86400000;   // 1 day in ms. Should be shorter in production.
      static final String PREFIX = "Bearer";
      **// Generate secret key. Only for demonstration purposes.**
      **// In production, you should read it from the application **  **// configuration.**
      **static****final****Key****key****=** **Keys.secretKeyFor (SignatureAlgorithm.**    **HS256);**
      **// Generate signed JWT token**
      **public** **String** **getToken****(String username)** **{**
        **String****token****=** **Jwts.builder()**
        **.setSubject(username)**
        **.setExpiration(****new****Date****(System.currentTimeMillis() +**                            **EXPIRATIONTIME))**
        **.signWith(key)**
        **.compact();** 
        **return** **token;**
      **}**
      **// Get a token from request Authorization header,**
      **// verify the token, and get username**
      **public** **String** **getAuthUser****(HttpServletRequest request)** **{**
        **String****token****=** **request.getHeader**
            **(HttpHeaders.AUTHORIZATION);**
        **if** **(token !=** **null****) {**
          **String****user****=** **Jwts.parserBuilder()**
          **.setSigningKey(key)**
          **.build()**
          **.parseClaimsJws(token.replace(PREFIX,** **""****))**
          **.getBody()**
          **.getSubject();**
          **if** **(user !=** **null****)**
            **return** **user;**
        **}**
        **return****null****;**
      **}**
    } 
    ```

1.  接下来，我们将添加一个新的类来存储用于认证的凭据。在这里，我们可以使用 Java **record**，这是在 Java 14 中引入的。如果你需要一个只持有数据的类，record 是一个不错的选择；你可以避免编写很多样板代码。在`com.packt.cardatabase.domain`包中创建一个新的 record（**新建** | **记录**）名为`AccountCredentials`！[](img/B19818_05_10.png)

    图 5.10：创建新记录

    记录有两个字段：`username`和`password`。以下是记录的源代码。正如你所见，当我们使用它时，我们不需要编写 getter 和 setter：

    ```java
    package com.packt.cardatabase.domain;
    public record AccountCredentials(String username, String password) {} 
    ```

1.  现在，我们将实现登录的 `controller` 类。登录是通过调用 `POST` 方法的 `/login` 端点并将在请求体中发送用户名和密码来完成的。在 `com.packt.cardatabase.web` 包内创建一个名为 `LoginController` 的类。我们必须将 `JwtService` 实例注入到控制器类中，因为这是在登录成功的情况下生成签名 JWT 所使用的。代码如下所示：

    ```java
    package com.packt.cardatabase.web;
    import org.springframework.http.HttpHeaders;
    import org.springframework.http.ResponseEntity;
    import org.springframework.security.authentication.AuthenticationManager;
    import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
    import org.springframework.security.core.Authentication;
    import org.springframework.web.bind.annotation.RequestBody;
    import org.springframework.web.bind.annotation.PostMapping;
    import org.springframework.web.bind.annotation.RestController;
    import com.packt.cardatabase.domain.AccountCredentials;
    import com.packt.cardatabase.service.JwtService;
    @RestController
    public class LoginController {
        private final JwtService jwtService;
        private final AuthenticationManager authenticationManager;
        public LoginController(JwtService jwtService,
          AuthenticationManager authenticationManager) {
            this.jwtService = jwtService;
            this.authenticationManager = authenticationManager;
        }
        @PostMapping("/login")
        public ResponseEntity<?> getToken(@RequestBody
          AccountCredentials credentials) {
        // Generate token and send it in the response Authorization
        // header
        }
    } 
    ```

1.  接下来，我们将实现处理登录功能的 `getToken` 方法。我们从请求体中获取一个包含用户名和密码的 JSON 对象。使用 `AuthenticationManager` 进行身份验证，它使用我们从请求中获取的凭据。然后，我们使用 `JwtService` 类的 `getToken` 方法生成 JWT。最后，我们构建一个包含生成的 JWT 的 `Authorization` 标头的 HTTP 响应：

    ```java
    // LoginController.java
    @PostMapping("/login")
    public ResponseEntity<?> getToken(@RequestBody AccountCredentials
       credentials) {
        UsernamePasswordAuthenticationToken creds = new
          UsernamePasswordAuthenticationToken(credentials.username(),                                          credentials.password());
        Authentication auth = authenticationManager.authenticate(creds);
        // Generate token
        String jwts = jwtService.getToken(auth.getName());
        // Build response with the generated token
        return ResponseEntity.ok().header(HttpHeaders.AUTHORIZATION,
                       "Bearer" + jwts).header(HttpHeaders.
                       ACCESS_CONTROL_EXPOSE_HEADERS,
                       "Authorization").build();
    } 
    ```

1.  我们还向 `LoginController` 类中注入了 `AuthenticationManager`，因此我们必须向 `SecurityConfig` 类中添加以下突出显示的代码：

    ```java
    package com.packt.cardatabase;
    **import** **org.springframework.context.annotation.Bean;**
    import org.springframework.context.annotation.Configuration;
    **import** **org.springframework.security.authentication.****AuthenticationManager;**
    **import** **org.springframework.security.config.annotation.**  **authentication.configuration.AuthenticationConfiguration;**
    import org.springframework.security.config.annotation.  authentication.builders.AuthenticationManagerBuilder;
    import org.springframework.security.config.annotation.  authentication.configuration.AuthenticationConfiguration;
    import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
    import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
    import com.packt.cardatabase.service.UserDetailsServiceImpl;
    @Configuration
    @EnableWebSecurity
    public class SecurityConfig {
        private final UserDetailsServiceImpl userDetailsService;
        public SecurityConfig(UserDetailsServiceImpl userDetailsService){
            this.userDetailsService = userDetailsService;
        }
        public void configureGlobal(AuthenticationManagerBuilder auth)
          throws Exception {
            auth.userDetailsService(userDetailsService)
              .passwordEncoder(new BCryptPasswordEncoder());
        }
        @Bean
        public PasswordEncoder passwordEncoder() {
            return new BCryptPasswordEncoder();
        }

        **@Bean**
        **public** **AuthenticationManager** **uthenticationManager****(**
          **AuthenticationConfiguration authConfig)****throws** **Exception {**
            **return** **authConfig.getAuthenticationManager();**
        **}**
    } 
    ```

1.  在此步骤中，我们必须配置 Spring Security 功能。Spring Security 的 `SecurityFilterChain` bean 定义了哪些路径是受保护的，哪些不是。将以下 `filterChain` 方法添加到 `SecurityConfig` 类中。在方法中，我们定义了对 `/login` 端点的 `POST` 方法请求不需要身份验证，以及对所有其他端点的请求需要身份验证。我们还将定义 Spring Security 将永远不会创建会话，因此我们可以禁用跨站请求伪造 (`csrf`)。JWT 被设计为无状态，这降低了与会话相关的漏洞风险。我们将在 HTTP 安全配置中使用 Lambdas：

    在某些其他编程语言中，Lambdas 被称为**匿名函数**。Lambdas 的使用使代码更易读，并减少了样板代码。

    ```java
    // SecurityConfig.java
    // Add the following import
    import org.springframework.security.web.SecurityFilterChain;
    // Add filterChain method
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws
      Exception {
        http.csrf((csrf) -> csrf.disable())
            .sessionManagement((sessionManagement) -> sessionManagement.            sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests((authorizeHttpRequests) ->            authorizeHttpRequests.requestMatchers(HttpMethod.POST,            "/login").permitAll().anyRequest().authenticated());
        return http.build();
    } 
    ```

1.  最后，我们准备好测试我们的登录功能。打开 Postman 并向 `http://localhost:8080/login` URL 发送 `POST` 请求。在请求体中定义一个有效用户，例如，`{"username":"user", "password":"user"}` 并从下拉列表中选择 **JSON**。Postman 将自动设置 `Content-Type` 标头为 `application/json`。您应该从 **Headers** 选项卡检查 `Content-Type` 标头是否设置正确。现在，您应该在响应中看到一个包含签名 JWT 的 `Authorization` 标头，如下面的截图所示：

![](img/B19818_05_11.png)

图 5.11：登录请求

您也可以通过使用错误的密码并查看响应不包含 `Authorization` 标头来测试登录。

## 保护其他请求

我们现在已经完成了登录步骤，接下来我们将继续处理其他传入请求的认证。在认证过程中，我们使用 **过滤器** 来在请求发送到控制器或响应发送到客户端之前执行一些操作。

以下步骤展示了认证过程的其余部分：

1.  我们将使用一个过滤器类来认证所有其他传入的请求。在根包中创建一个名为 `AuthenticationFilter` 的新类。`AuthenticationFilter` 类扩展了 Spring Security 的 `OncePerRequestFilter` 接口，该接口提供了一个 `doFilterInternal` 方法，我们在其中实现认证。我们必须将一个 `JwtService` 实例注入到过滤器类中，因为它需要验证请求中的令牌。`SecurityContextHolder` 是 Spring Security 存储已认证用户详情的地方。代码在下面的代码片段中展示：

    ```java
    package com.packt.cardatabase;
    import org.springframework.http.HttpHeaders;
    import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
    import org.springframework.security.core.Authentication;
    import org.springframework.security.core.context.SecurityContextHolder;
    import org.springframework.stereotype.Component;
    import org.springframework.web.filter.OncePerRequestFilter;
    import com.packt.cardatabase.service.JwtService;
    import jakarta.servlet.FilterChain;
    import jakarta.servlet.ServletException;
    import jakarta.servlet.http.HttpServletRequest;
    import jakarta.servlet.http.HttpServletResponse;
    @Component
    public class AuthenticationFilter extends OncePerRequestFilter {
        private final JwtService jwtService;
        public AuthenticationFilter(JwtService jwtService) {
            this.jwtService = jwtService;
       }
        @Override
        protected void doFilterInternal(HttpServletRequest request,               HttpServletResponse response, FilterChain filterChain)
                throws ServletException, java.io.IOException {
            // Get token from the Authorization header
            String jws = request.getHeader(HttpHeaders.AUTHORIZATION);
            if (jws != null) {
                // Verify token and get user
                String user = jwtService.getAuthUser(request);
                // Authenticate
                Authentication authentication =
                new UsernamePasswordAuthenticationToken(user, null,
                    java.util.Collections.emptyList());
                SecurityContextHolder.getContext()
                    .setAuthentication(authentication);
            }
            filterChain.doFilter(request, response);
        }
    } 
    ```

1.  接下来，我们必须将我们的过滤器类添加到 Spring Security 配置中。打开 `SecurityConfig` 类，并注入我们刚刚实现的 `AuthenticationFilter` 类，如高亮代码所示：

    ```java
    private final UserDetailsServiceImpl userDetailsService;
    **private****final** **AuthenticationFilter authenticationFilter;**
    public SecurityConfig(UserDetailsServiceImpl userDetailsService**,****AuthenticationFilter authenticationFilter**) {
        this.userDetailsService = userDetailsService;
    **this****.authenticationFilter = authenticationFilter;**
    } 
    ```

1.  然后，修改 `SecurityConfig` 类中的 `filterChain` 方法，并添加以下代码行：

    ```java
    //Add the following import
    import org.springframework.security.web.authentication.  UsernamePasswordAuthenticationFilter;
    // Modify the filterChain method
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws   Exception {
        http.csrf((csrf) -> csrf.disable())
            .sessionManagement((sessionManagement) -> sessionManagement.            sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests((authorizeHttpRequests) ->            authorizeHttpRequests.requestMatchers(HttpMethod.POST,            "/login").permitAll().anyRequest().authenticated())
            **.addFilterBefore(authenticationFilter,**            **UsernamePasswordAuthenticationFilter.class);**
        return http.build();
    } 
    ```

1.  现在，我们已经准备好测试整个工作流程。在运行应用程序后，我们可以首先通过调用 `POST` 方法的 `/login` 端点来登录，如果登录成功，我们将在 `Authorization` 标头中收到一个 JWT。请记住，如果 Postman 没有自动完成，请在请求体中添加一个有效的用户，并将 `Content-Type` 标头设置为 `application/json`。以下屏幕截图展示了这个过程：

![](img/B19818_05_12.png)

图 5.12：登录请求

1.  在成功登录后，我们可以通过在 `Authorization` 标头中发送从登录接收到的 JWT 来调用其他 RESTful 服务端点。从登录响应中复制令牌（不带 `Bearer` 前缀），并在 `VALUE` 列中添加带有令牌的 `Authorization` 标头。参考以下屏幕截图中的示例，其中对 `/cars` 端点执行了一个 `GET` 请求：

![](img/B19818_05_13.png)

图 5.13：认证 GET 请求

每次应用程序重启时，你必须重新认证，因为会生成一个新的 JWT。

JWT 不是永远有效的，因为它被设置了一个过期日期。在我们的例子中，为了演示目的，我们设置了一个较长的过期时间。在生产环境中，时间应该根据用例最好设置为分钟。

## 处理异常

我们还应该在认证中处理异常。目前，如果你尝试使用错误的密码登录，你会收到一个没有进一步说明的 `403 Forbidden` 状态。Spring Security 提供了一个 `AuthenticationEntryPoint` 接口，可以用来处理异常。让我们看看它是如何工作的：

1.  在根包中创建一个名为`AuthEntryPoint`的新类，该类实现了`AuthenticationEntryPoint`接口。我们将实现`commence`方法，该方法接受一个异常作为参数。在异常的情况下，我们将响应状态设置为`401 Unauthorized`并将异常消息写入响应体。代码如下所示：

    ```java
    package com.packt.cardatabase;
    import java.io.IOException;
    import java.io.PrintWriter;
    import jakarta.servlet.ServletException;
    import jakarta.servlet.http.HttpServletRequest;
    import jakarta.servlet.http.HttpServletResponse;
    import org.springframework.http.MediaType;
    import org.springframework.security.core.
      AuthenticationException;
    import org.springframework.security.web.
      AuthenticationEntryPoint;
    import org.springframework.stereotype.Component;
    @Component
    public class AuthEntryPoint implements AuthenticationEntryPoint {
      @Override
      public void commence(
        HttpServletRequest request, HttpServletResponse response,
        AuthenticationException authException) throws IOException,
        ServletException {
            response.setStatus (HttpServletResponse.SC_UNAUTHORIZED);
            response.setContentType (MediaType.APPLICATION_JSON_VALUE);
            PrintWriter writer = response.getWriter();
            writer.println("Error: " + authException.getMessage());
      }
    } 
    ```

1.  然后，我们必须为异常处理配置 Spring Security。将我们的`AuthEntryPoint`类注入到`SecurityConfig`类中，如下所示的高亮代码：

    ```java
    // SecurityConfig.java
    private final UserDetailsServiceImpl userDetailsService;
    private final AuthenticationFilter authenticationFilter;
    **private****final** **AuthEntryPoint exceptionHandler;**
    public SecurityConfig(UserDetailsServiceImpl userDetailsService,   AuthenticationFilter authenticationFilter, **AuthEntryPoint **  **exceptionHandler**) {
        this.userDetailsService = userDetailsService;
        this.authenticationFilter = authenticationFilter;
        **this****.exceptionHandler = exceptionHandler;**
    } 
    ```

1.  然后，按照以下方式修改`filterChain`方法：

    ```java
    // SecurityConfig.java
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws
      Exception {
        http.csrf((csrf) -> csrf.disable())
            .sessionManagement((sessionManagement) ->
                sessionManagement.sessionCreationPolicy(
                SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests((authorizeHttpRequests) ->             authorizeHttpRequests.requestMatchers(HttpMethod.POST,            "/login").permitAll().anyRequest().authenticated())
            .addFilterBefore(authenticationFilter,            UsernamePasswordAuthenticationFilter.class)
            **.exceptionHandling((exceptionHandling) -> exceptionHandling.**            **authenticationEntryPoint(exceptionHandler));**
        return http.build();
    } 
    ```

1.  现在，如果您发送一个带有错误凭证的登录`POST`请求，您将在响应中获得`401 Unauthorized`状态和一个错误消息，如下面的屏幕截图所示：

![](img/B19818_05_14.png)图 5.14：无效凭证

## 添加 CORS 过滤器

我们还将向我们的安全配置类添加一个**跨源资源共享**（**CORS**）过滤器。CORS 引入了一些帮助客户端和服务器决定是否允许或拒绝跨源请求的头部。CORS 过滤器对于前端是必需的，前端从其他来源发送请求。CORS 过滤器拦截请求，如果这些请求被识别为跨源，它会在请求中添加适当的头部。为此，我们将使用 Spring Security 的`CorsConfigurationSource`接口。

在这个示例中，我们将允许所有来源的 HTTP 方法和头。如果您需要一个更精细的定义，您可以在以下位置定义一个允许的来源、方法和头列表。让我们开始吧：

1.  将以下导入和方法添加到您的`SecurityConfig`类中，以启用 CORS 过滤器：

    ```java
    // SecurityConfig.java
    // Add the following imports
    import java.util.Arrays;
    import org.springframework.web.cors.CorsConfiguration;
    import org.springframework.web.cors.CorsConfigurationSource;
    import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
    // Add Global CORS filter inside the class
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        UrlBasedCorsConfigurationSource source =
           new UrlBasedCorsConfigurationSource();
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowedOrigins(Arrays.asList("*"));
        config.setAllowedMethods(Arrays.asList("*"));
        config.setAllowedHeaders(Arrays.asList("*"));
        config.setAllowCredentials(false);
        config.applyPermitDefaultValues();
        source.registerCorsConfiguration("/**", config);
        return source;
    } 
    ```

    如果您想明确定义来源，您可以按照以下方式设置：

    ```java
    // localhost:3000 is allowed
    config.setAllowedOrigins(Arrays.asList ("http://localhost:3000")); 
    ```

1.  我们还必须在`filterChain`方法中添加`cors()`函数，如下面的代码片段所示：

    ```java
    // SecurityConfig.java
    // Add the following static import
    import static org.springframework.security.config.Customizer.withDefaults;
    // Modify filterChain method
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws
      Exception {
        http.csrf((csrf) -> csrf.disable())
            **.cors(withDefaults())**
            .sessionManagement((sessionManagement) -> sessionManagement.            sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests((authorizeHttpRequests) -> 
                authorizeHttpRequests.requestMatchers(HttpMethod.POST,            "/login").permitAll().anyRequest().authenticated())
            .addFilterBefore(authenticationFilter,            UsernamePasswordAuthenticationFilter.class)
            .exceptionHandling((exceptionHandling) -> exceptionHandling.            authenticationEntryPoint(exceptionHandler));
        return http.build();
    } 
    ```

现在，我们已经确保了我们的后端安全。在下一节中，我们将介绍基于角色的安全基础，您可以使用它来在您的 Spring Boot 应用程序中获得更细粒度的访问控制。

# 基于角色的安全

在 Spring Security 中，**角色**可以用来定义粗粒度的基于角色的安全，用户可以被分配到一个或多个角色。角色通常具有层次结构，例如，`ADMIN`、`MANAGER`、`USER`。Spring Security 还提供了**权限**，可以用于更细粒度的访问控制。我们已经为我们的用户定义了简单的角色`ADMIN`和`USER`，在我们的示例后端应用程序中我们没有使用基于角色的安全。本节介绍了在您的 Spring Boot 应用程序中实现基于角色安全的不同方法。

你可以在你的安全配置类中在请求级别定义基于角色的访问控制。在下面的示例代码中，我们定义了哪些端点需要特定的角色才能访问。`/admin/**` 端点需要 `ADMIN` 角色才能访问，而 `/user/**` 端点需要 `USER` 角色才能访问。我们使用 Spring Security 的 `hasRole()` 方法，如果用户具有指定的角色则返回 true：

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws
   Exception {
    http.csrf((csrf) -> csrf.disable()).cors(withDefaults())
        .sessionManagement((sessionManagement) -> sessionManagement.            sessionCreationPolicy(SessionCreationPolicy.STATELESS))
        .authorizeHttpRequests((authorizeHttpRequests) -> 
            authorizeHttpRequests.requestMatchers("/admin/**").hasRole             ("ADMIN").requestMatchers("/user/**").hasRole("USER")
            .anyRequest().authenticated())
    return http.build();
} 
```

你可以在 Spring Boot 文档中了解更多关于请求授权的信息：https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html。

Spring Security 提供了 `@PreAuthorize`、`@PostAuthorize`、`@PreFilter`、`@PostFilter` 和 `@Secured` 注解，用于应用**方法级安全**。在 `spring-boot-starter-security` 中默认不启用方法级安全。你必须在你的 Spring 配置类中启用它，例如，在顶级配置中，通过使用 `@EnableMethodSecurity` 注解：

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
**import** **org.springframework.security.config.annotation.method.****configuration.EnableMethodSecurity;**
@SpringBootApplication
**@EnableMethodSecurity**
public class CardatabaseApplication implements CommandLineRunner {
} 
```

然后，你将能够在你的方法中使用方法级安全注解。在下面的示例中，具有 `USER` 角色的用户可以执行 `updateCar()` 方法，而具有 `ADMIN` 角色的用户可以执行 `deleteOwner()` 方法。`@PreAuthorize` 注解在方法执行前检查规则。如果用户没有指定的角色，Spring Security 会阻止方法执行，并抛出 `AccessDeniedException`：

```java
@Service
public class CarService {
    @PreAuthorize("hasRole('USER')")
    public void updateCar(Car car) {
        // This method can be invoked by user with USER role.
    }
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteOwner(Car car) {
        // This method can be invoked by user with ADMIN role.
    }
} 
```

`@PreAuthorize` 注解取代了 `@Secured` 注解，并建议使用它。

`@PostAuthorize` 注解可以在方法执行后用于检查授权。例如，你可以用它来检查用户是否有权限访问方法返回的对象，或者你可以根据用户的授权来过滤返回的数据。

`@PreFilter` 和 `@PostFilter` 注解可以用来过滤对象列表，但它们通常不用于基于角色的访问控制。与这些注解一起使用的规则更为细致。

你可以在 Spring Security 文档中了解更多关于方法安全的信息：[`docs.spring.io/spring-security/reference/servlet/authorization/method-security.html`](https://docs.spring.io/spring-security/reference/servlet/authorization/method-security.html)。

在下一节中，我们将介绍使用 Spring Boot 的 OAuth 基础知识。

# 使用 OAuth2 和 Spring Boot

在你的应用程序中完全实现安全的认证和授权是非常具有挑战性的。在生产环境中，建议使用 OAuth2 提供商来实现。这实际上简化了认证过程，并且提供商通常有出色的安全实践。

这些不是实现 OAuth 2.0 授权的详细说明，但它们会给你一个关于该过程的概念。

**OAuth**（**开放授权**）是用于互联网上受保护资源的安全访问的标准。OAuth 标准版本 2.0 现在普遍使用。有几个 OAuth 2.0 提供商实现了第三方应用程序的 OAuth 授权。以下列出了一些常见的提供商：

+   Auth0: [`auth0.com/`](https://auth0.com/)

+   Okta: [`www.okta.com/`](https://www.okta.com/)

+   Keycloak: [`www.keycloak.org/`](https://www.keycloak.org/)

您可以使用 OAuth2 实现社交登录，之后用户可以使用来自社交媒体平台（如 Facebook）的现有凭据登录。OAuth 还定义了撤销访问令牌和处理令牌过期的机制。

如果您想在 Spring Boot 应用程序中使用 OAuth，第一步是选择一个 OAuth 提供商。上述列表中的所有提供商都可以与您的 Spring Boot 应用程序一起使用。

在 OAuth2 过程中，术语 **资源所有者** 通常指最终用户，而 **授权服务器** 是 OAuth 提供商服务的一部分。**客户端** 是一个希望访问受保护资源的应用程序。**资源服务器** 通常指客户端希望使用的 API。

使用 REST API 的 OAuth2 认证过程的简化版本包含以下步骤：

1.  认证：第三方应用程序通过请求访问受保护资源来进行认证。

1.  授权：资源所有者授权访问其资源，通常通过用户登录。

1.  授权服务器授权资源所有者，并使用授权码将用户重定向回客户端。

1.  客户端使用授权码从授权服务器请求访问令牌。访问令牌的格式在标准中未指定，JWT 非常常用。

1.  授权服务器验证访问令牌。如果令牌有效，客户端应用程序将收到访问令牌。

1.  客户端可以使用访问令牌开始访问受保护资源，例如，调用 REST API 端点。

在您选择了提供商并了解了其服务的工作方式后，您必须配置您的 Spring Boot 应用程序。Spring Boot 提供了 `spring-boot-starter-oauth2-client` 依赖项，用于 OAuth2 认证和授权。它用于简化 Spring Boot 应用程序中的 OAuth 2.0 集成。许多 OAuth 提供商为不同的技术提供了文档，例如 Spring Boot。

实现将取决于提供商。以下是一些有用的链接：

+   Auth0 提供了一个很好的教程，介绍如何将登录添加到您的 Spring Boot 应用程序中：[`auth0.com/docs/quickstart/webapp/java-spring-boot/interactive`](https://auth0.com/docs/quickstart/webapp/java-spring-boot/interactive).

+   Baeldung 提供了使用 Spring Boot 应用程序与 Keycloak 一起使用的快速指南：[`www.baeldung.com/spring-boot-keycloak`](https://www.baeldung.com/spring-boot-keycloak).

+   Spring 还有一个关于如何使用 GitHub 实现社交登录的教程：https://spring.io/guides/tutorials/spring-boot-oauth2。

我们建议阅读这些内容，以更好地了解如何在您的应用程序中使用 OAuth 2.0。

现在，我们已经使用 JWT 完成了后端的保护，当我们开始开发前端时，我们将使用这个版本。

# 摘要

在本章中，我们专注于使我们的 Spring Boot 后端更加安全。我们首先通过使用 Spring Security 添加额外的保护。然后，我们实现了 JWT 认证。JWT 通常用于保护 RESTful API，它是一种轻量级的认证方法，适合我们的需求。我们还介绍了 OAuth 2.0 标准的基础知识以及如何在 Spring Boot 应用程序中开始使用它。

在下一章中，我们将学习在 Spring Boot 应用程序中进行测试的基础知识。

# 问题

1.  Spring Security 是什么？

1.  如何使用 Spring Boot 保护您的后端？

1.  JWT 是什么？

1.  如何使用 JWT 保护您的后端？

1.  OAuth 2.0 是什么？

# 进一步阅读

Packt 为您提供了其他资源，以了解 Spring Security。例如：

+   *《Spring Security Core：从入门到精通》，作者：约翰·汤普森* ([`www.packtpub.com/product/spring-security-core-beginner-to-guru-video/9781800560000`](https://www.packtpub.com/product/spring-security-core-beginner-to-guru-video/9781800560000))

# 在 Discord 上了解更多

要加入这本书的 Discord 社区——在那里您可以分享反馈、向作者提问以及了解新版本——请扫描下面的二维码：

[`packt.link/FullStackSpringBootReact4e`](https://packt.link/FullStackSpringBootReact4e)

![](img/QR_Code107961080093826402.png)
