

# 迁移到 Spring Security 6

在本章的最后，我们将回顾与从`Spring Security 5.x`迁移到`Spring Security 6.x`相关的常见迁移问题，这些问题包含大量的非被动重构。

在本章末尾，我们还将突出显示`Spring Security 6.x`中可以找到的一些新功能。然而，我们并没有明确涵盖从`Spring Security 5.x`到`Spring Security 6.x`的变化。这是因为通过解释这两个版本之间的差异，用户应该能够更新到`Spring` `Security 6.x`。

你可能计划将现有应用程序迁移到`Spring Security 6.x`，或者你可能正在尝试向`Spring Security 5.x`应用程序添加功能，并在此书的页面上寻找指导。我们将在此章中尝试解决你的两个问题。

首先，我们将概述`Spring Security 5.x`和`6.x`之间的重要差异——包括功能和配置。

其次，我们将提供一些关于映射配置或类名更改的指导。这将更好地帮助你将书中的示例从`Spring Security 6.x`翻译回`Spring Security 5.x`（如果适用）。

重要提示

`Spring Security 6.x`要求迁移到**Spring Framework 6**和**Java 17**或更高版本。

注意，在许多情况下，迁移这些其他组件可能对你的应用程序的影响比升级`Spring Security`更大！

在本章中，我们将涵盖以下主题：

+   检查`Spring` `Security 6.x`中的重要增强。

+   理解在现有 Spring 版本中所需的配置更改。

+   在将`Spring Security 5.x`应用程序迁移到`Spring` `Security 6.x`时，检查`Spring Security 5.x`应用程序。

+   展示`Spring` `Security 6.x`中重要类和包的整体迁移情况。

+   突出显示`Spring Security 6.x`中的一些新功能。完成本章的审查后，你将处于良好的位置，可以将现有应用程序从`Spring Security 5.x`迁移到`Spring` `Security 6.x`。

+   从`Spring` `Security 5.x`迁移。

本章的代码示例链接在此：[`packt.link/wD0Sk`](https://packt.link/wD0Sk)。

# 漏洞利用保护

在`Spring Security 5.8`中，负责向应用程序提供`CsrfToken`的默认`CsrfTokenRequestHandler`是`CsrfTokenRequestAttributeHandler`。字段`csrfRequestAttributeName`的默认设置是`null`，导致在每次请求时加载 CSRF 令牌。

应当认为读取会话是不必要的情况的示例包括明确标记为`permitAll()`的端点，例如静态资产、静态 HTML 页面以及位于同一域名/服务器下的单页应用程序。

在`Spring Security 6`中，`csrfRequestAttributeName`现在默认为`_csrf`。如果你只是为了过渡到 6.0 版本而配置了以下内容，你现在可以安全地将其删除：

```java
requestHandler.setCsrfRequestAttributeName("_csrf");
```

现在我们已经探讨了如何定义 `CsrfToken`，我们将探讨如何防范 CSRF 攻击。

## 防范 CSRF 攻击

在 `Spring Security 5.8` 中，使 `CsrfToken` 可用于应用程序的默认 `CsrfTokenRequestHandler` 是 `CsrfTokenRequestAttributeHandler`。`XorCsrfTokenRequestAttributeHandler` 的引入是为了启用 `CSRF` 攻击支持。

在 Spring Security 6 中，`XorCsrfTokenRequestAttributeHandler` 成为提供 `CsrfToken` 的默认 `CsrfTokenRequestHandler`。如果您仅为了过渡到版本 6.0 而配置了 `XorCsrfTokenRequestAttributeHandler`，现在可以安全地将其移除。

重要提示

如果您已将 `csrfRequestAttributeName` 设置为 `null` 以排除延迟令牌，或者如果您已为任何特定目的建立了 `CsrfTokenRequestHandler`，则可以保持当前配置。

## 支持 WebSocket 的 CSRF 攻击

在 `Spring Security 5.8` 中，用于提供 `CsrfToken` 并具有 WebSocket 安全性的默认 `ChannelInterceptor` 是 `CsrfChannelInterceptor`。`XorCsrfChannelInterceptor` 的引入是为了启用 `CSRF` 攻击支持。

在 `Spring Security 6` 中，`XorCsrfChannelInterceptor` 成为提供 `CsrfToken` 的默认 `ChannelInterceptor`。如果您仅为了过渡到版本 6.0 而配置了 `XorCsrfChannelInterceptor`，现在可以安全地将其移除。

在探讨了如何防范 CSRF 攻击之后，我们将深入探讨配置迁移选项。

# 配置迁移

后续章节涉及配置 `HttpSecurity`、`WebSecurity` 和 `AuthenticationManager` 的变更。

## 将 `@Configuration` 注解添加到 `@Enable*` 注解中

在 6.0 版本中，注解 `@EnableWebSecurity`、`@EnableMethodSecurity`、`@EnableGlobalMethodSecurity` 和 `@EnableGlobalAuthentication` 不再包含 `@Configuration`。

例如，`@EnableWebSecurity` 将会被修改为：

```java
@EnableWebSecurity
public class SecurityConfig {
    // ...
}
```

变更为：

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    // ...
}
```

为了适应这一变化，无论您在哪里使用这些注解，您可能需要添加 `@Configuration`。

## 使用新的请求匹配器方法

在 `Spring Security 5.8` 中，`antMatchers`、`mvcMatchers` 和 `regexMatchers` 方法被弃用，以支持新的 `requestMatchers` 方法。

新的 `requestMatchers` 方法的引入扩展到了 `authorizeHttpRequests`、`authorizeRequests`、CSRF 配置、`WebSecurityCustomizer` 以及具有专用 `RequestMatcher` 方法的其他位置。截至 `Spring Security 6`，已弃用的方法已被移除。

新的方法通过自动选择最适合您应用程序的 `RequestMatcher` 实现提供了更安全的默认设置。

为了总结，这些方法：

+   如果您的应用程序包含 Spring MVC 在类路径中，请选择 `MvcRequestMatcher` 实现。

+   如果没有 Spring MVC，则回退到`AntPathRequestMatcher`实现，使其行为与 Kotlin 等效方法对齐。

以下表格应指导你在迁移过程中的操作：

| **Spring** **Security 5** | **Spring** **Security 6** |
| --- | --- |
| `antMatchers("/api/admin/**")` | `requestMatchers("/api/admin/**")` |
| `mvcMatchers("/admin/**")` | `requestMatchers("/admin/**")` |
| `mvcMatchers("/admin").servletPath("/path")` | `requestMatchers(mvcMatcherBuilder.pattern("/admin"))` |

表 16.1 – 使用新的 requestMatchers 进行迁移

如果你在新的`requestMatchers`方法上遇到困难，你可以选择回退到你之前使用的`RequestMatcher`实现。例如，如果你更喜欢继续使用`AntPathRequestMatcher`和`RegexRequestMatcher`实现，你可以使用接受`RequestMatcher`实例的`requestMatchers`方法：

| **Spring** **Security 5** | **Spring** **Security 6** |
| --- | --- |
| `antMatchers("/api/admin/**")` | `requestMatchers(antMatcher("/user/**"))` |

表 16.2 – 使用新的 requestMatchers 的替代方案

重要提示

请注意，提供的示例使用了来自`AntPathRequestMatcher`和`RegexRequestMatcher`的静态工厂方法来提高可读性。

当你使用`WebSecurityCustomizer`接口时，你可以用相应的`requestMatchers`替代方法替换已弃用的`antMatchers`方法：

```java
@Bean
public WebSecurityCustomizer webSecurityCustomizer() {
    return web -> web.ignoring().antMatchers("/ignore1", "/ignore2");
}
```

使用对应的`requestMatchers`替代方法：

```java
@Bean
public WebSecurityCustomizer webSecurityCustomizer() {
    return web -> web.ignoring().requestMatchers("/ignore1", "/ignore2");
}
```

同样，如果你正在自定义`CSRF`配置以排除特定路径，你可以用`requestMatchers`的对应方法替换已弃用的方法。

## 使用新的 securityMatchers 方法

在`Spring Security 5.8`中，`HttpSecurity`中的`antMatchers`、`mvcMatchers`和`requestMatchers`方法被弃用，以支持新的`securityMatchers`方法。

需要注意的是，这些方法与被弃用的`authorizeHttpRequests`方法不同，这些方法被`requestMatchers`方法所取代。然而，`securityMatchers`方法与`requestMatchers`方法有相似之处，即它们会自动选择最适合你应用程序的`RequestMatcher`实现。

为了详细说明，新的方法：

+   如果你的应用程序包含 Spring MVC 在类路径中，请选择`MvcRequestMatcher`实现。

+   如果没有 Spring MVC，则回退到`AntPathRequestMatcher`实现，使其行为与 Kotlin 等效方法对齐。`securityMatchers`方法的引入也有助于避免与`authorizeHttpRequests`中的`requestMatchers`方法混淆。

以下表格应指导你在迁移过程中的操作，其中`http`是`HttpSecurity`类型：

| **Spring** **Security 5** | **Spring** **Security 6** |
| --- | --- |
| `http.antMatcher("/api/**")` | `http.securityMatcher("/api/**")` |
| `http.requestMatcher(new MyCustomRequestMatcher())` | `http.securityMatcher(new MyCustomRequestMatcher())` |
| `http``.requestMatchers((matchers) ->` `matchers``.``antMatchers("/api/**", "/app/**")``.``mvcMatchers("/admin/**")``.``requestMatchers(new MyCustomRequestMatcher()))` | `http.securityMatchers((matchers) -> matchers.requestMatchers("/api/**", "/``app/**", "/admin/**")``.``requestMatchers(new MyCustomRequestMatcher()))` |

表 16.3 – 迁移到新的 securityMatchers

如果你在使用 `securityMatchers` 方法自动选择 `RequestMatcher` 实现时遇到挑战，你可以选择手动选择 `RequestMatcher` 实现：

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
          .securityMatchers(matchers -> matchers
                .requestMatchers(antMatcher("/api/**"), antMatcher("/app/**"))
          );
    return http.build();
}
```

在探索了新的 `securityMatchers` 方法之后，我们现在将探讨在 `Spring Security 6.x` 中替换 `WebSecurityConfigurerAdapter` 的过程。

## 替换 WebSecurityConfigurerAdapter 类

`WebSecurityConfigurerAdapter` 类在 `Spring Security 6.x` 中已被弃用并移除。在接下来的子章节中，我们将探讨这一重大变化的影响。

### 暴露 SecurityFilterChain Bean

在 `Spring Security 5.4` 中，引入了一个新功能，允许发布 `SecurityFilterChain` Bean 而不是扩展 `WebSecurityConfigurerAdapter`。然而，在 6.0 版本中，`WebSecurityConfigurerAdapter` 已被移除。为了适应这一变化，你可以替换类似的结构：

```java
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
       http
             .authorizeHttpRequests(authorize -> authorize
                   .anyRequest().authenticated()
             )
             .httpBasic(withDefaults());
    }
}
```

使用以下方法：

```java
@Configuration
public class SecurityConfiguration {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
       http
             .authorizeHttpRequests((authorize) -> authorize
                   .anyRequest().authenticated()
             )
             .httpBasic(withDefaults());
       return http.build();
    }
}
```

### 暴露 WebSecurityCustomizer Bean

`Spring Security 5.4` 引入了 `WebSecurityCustomizer` 作为 `WebSecurityConfigurerAdapter` 中 `configure(WebSecurity web)` 的替代品。为了准备其移除，你可以更新类似以下代码：

```java
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(WebSecurity web) {
       web.ignoring().antMatchers("/ignore1", "/ignore2");
    }
}
```

使用以下方法：

```java
@Configuration
public class SecurityConfiguration {
    @Bean
    public WebSecurityCustomizer webSecurityCustomizer() {
       return (web) -> web.ignoring().antMatchers("/ignore1", "/ignore2");
    }
}
```

### 暴露 AuthenticationManager Bean

随着 `WebSecurityConfigurerAdapter` 的移除，`configure(AuthenticationManagerBuilder)` 方法也被消除。

### LDAP 认证

当使用 `auth.ldapAuthentication()` 为 **轻量级目录访问协议 (LDAP**) 认证支持时，你可以替换以下内容：

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
       auth
             .ldapAuthentication()
             .userDetailsContextMapper(new PersonContextMapper())
             .userDnPatterns("uid={0},ou=people")
             .contextSource()
             .port(0);
    }
}
```

使用以下方法：

```java
@Configuration
public class SecurityConfiguration {
    @Bean
    public EmbeddedLdapServerContextSourceFactoryBean contextSourceFactoryBean() {
       EmbeddedLdapServerContextSourceFactoryBean contextSourceFactoryBean =
             EmbeddedLdapServerContextSourceFactoryBean.fromEmbeddedLdapServer();
       contextSourceFactoryBean.setPort(0);
       return contextSourceFactoryBean;
    }
    @Bean
    AuthenticationManager ldapAuthenticationManager(BaseLdapPathContextSource contextSource) {
       LdapBindAuthenticationManagerFactory factory =
             new LdapBindAuthenticationManagerFactory(contextSource);
       factory.setUserDnPatterns("uid={0},ou=people");
       factory.setUserDetailsContextMapper(new PersonContextMapper());
       return factory.createAuthenticationManager();
    }
}
```

### JDBC 认证

如果你目前正使用 `auth.jdbcAuthentication()` 为 **Java 数据库连接 (JDBC**) 认证支持提供支持，你可以替换：

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    private final DataSource dataSource;
    public SecurityConfig(DataSource dataSource) {
       this.dataSource = dataSource;
    }
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
       UserDetails user = User.withDefaultPasswordEncoder()
             .username("user")
             .password("password")
             .roles("USER")
             .build();
       auth.jdbcAuthentication()
             .withDefaultSchema()
             .dataSource(this.dataSource)
             .withUser(user);
    }
}
```

使用以下方法：

```java
@Configuration
public class SecurityConfig {
    private final DataSource dataSource;
    public SecurityConfig(DataSource dataSource) {
       this.dataSource = dataSource;
    }
    @Bean
    public UserDetailsManager users(DataSource dataSource) {
       UserDetails user = User.withDefaultPasswordEncoder()
             .username("user")
             .password("password")
             .roles("USER")
             .build();
       JdbcUserDetailsManager users = new JdbcUserDetailsManager(dataSource);
       users.createUser(user);
       return users;
    }
}
```

### 内存认证

如果你目前正使用 `auth.inMemoryAuthentication()` 为内存 `Authentication` 支持提供支持，你可以替换：

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
       UserDetails user = User.withDefaultPasswordEncoder()
             .username("user")
             .password("password")
             .roles("USER")
             .build();
       auth.inMemoryAuthentication()
             .withUser(user);
    }
}
```

使用以下方法：

```java
@Configuration
public class SecurityConfig {
    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
       UserDetails user = User.withDefaultPasswordEncoder()
             .username("user")
             .password("password")
             .roles("USER")
             .build();
       return new InMemoryUserDetailsManager(user);
    }
}
```

在探索 `WebSecurityConfigurerAdapter` 移除的影响之后，我们将深入了解密码编码的更新。

## 密码编码更新

在 `Spring Security 6.0` 中，对密码编码的最小要求已被修订，以适应 **PBKDF2**、**SCrypt** 和 **Argon2**。

如果你使用默认的密码编码器，则无需遵循任何准备步骤，你可以跳过这一部分。

### Pbkdf2PasswordEncoder 更新

如果你正在使用 `Pbkdf2PasswordEncoder`，构造函数已被替换为与提供的设置相关的 `Spring Security` 版本相对应的静态工厂：

```java
@Bean
PasswordEncoder passwordEncoder2() {
    return Pbkdf2PasswordEncoder.defaultsForSpringSecurity_v5_5();
}
```

如果你有自定义设置，请使用指定所有设置的构造函数：

```java
@Bean
PasswordEncoder passwordEncoder() {
    return new Pbkdf2PasswordEncoder("secret".getBytes(UTF_8), 16, 185000, 256);
}
```

### SCryptPasswordEncoder 更新

如果你正在使用 `SCryptPasswordEncoder`，构造函数已被替换为与提供的设置关联的 `Spring Security` 版本的静态工厂。

您的第一步应该是修改已弃用的构造函数：

```java
@Bean
PasswordEncoder passwordEncoder() {
    return SCryptPasswordEncoder.defaultsForSpringSecurity_v4_1();
}
```

### Argon2PasswordEncoder 更新

如果你正在使用 `Argon2PasswordEncoder`，构造函数已被替换为与提供的设置关联的 `Spring Security` 版本的静态工厂。例如：

```java
@Bean
PasswordEncoder passwordEncoder() {
    return Argon2PasswordEncoder.defaultsForSpringSecurity_v5_2();
}
```

### 委派 PasswordEncoder 使用

如果你没有使用已弃用的构造函数，更新你的代码以符合最新标准是至关重要的。这包括配置 `DelegatingPasswordEncoder` 以识别符合当前标准的密码并将它们更新到最新版本。以下使用 `Pbkdf2PasswordEncoder` 的示例也可以应用于 `SCryptPasswordEncoder` 或 `Argon2PasswordEncoder`：

```java
@Bean
PasswordEncoder passwordEncoder() {
    String prefix = "pbkdf2@5.8";
    PasswordEncoder current = Pbkdf2PasswordEncoder.defaultsForSpringSecurity_v5_5();
    PasswordEncoder upgraded = Pbkdf2PasswordEncoder.defaultsForSpringSecurity_v5_8();
    DelegatingPasswordEncoder delegating = new DelegatingPasswordEncoder(prefix, Map.of(prefix, upgraded));
    delegating.setDefaultPasswordEncoderForMatches(current);
    return delegating;
}
```

### 弃用 Encryptors.queryableText

使用 `Encryptors.queryableText(CharSequence,` `CharSequence)` 被认为是不可安全的，因为相同的输入数据将产生相同的输出（CVE-2020-5408 - [`github.com/advisories/GHSA-2ppp-9496-p23q`](https://github.com/advisories/GHSA-2ppp-9496-p23q))。

`Spring Security 6.x` 不再支持通过此方法进行数据加密。为了方便升级，你必须使用支持的机制重新加密数据或将数据以解密形式存储。

在检查密码编码更新之后，我们将深入了解会话管理更新的细节。

## 会话管理更新

在接下来的章节中，我们将详细检查会话管理更新，包括主要的弃用和修改。

### 强制保存 SecurityContextRepository

在 `Spring Security 5` 中，默认过程涉及通过 `SecurityContextPersistenceFilter` 自动将 `SecurityContext` 保存到 `SecurityContextRepository`。这种保存发生在 `HttpServletResponse` 提交之前，就在 `SecurityContextPersistenceFilter` 之前。然而，这种自动持久化可能会让用户措手不及，尤其是在请求完成之前（即在提交 `HttpServletResponse` 之前）执行。它还引入了跟踪状态的复杂性，以确定保存的必要性，有时会导致对 `SecurityContextRepository`（例如，`HttpSession`）的不必要写入。

随着 `Spring Security 6` 的推出，默认行为发生了变化。`SecurityContextHolderFilter` 现在将仅从 `SecurityContextRepository` 读取 `SecurityContext` 并将其填充到 `SecurityContextHolder` 中。如果用户希望 `SecurityContext` 在请求之间持续存在，他们现在必须显式使用 `SecurityContextRepository` 保存 `SecurityContext`。此修改通过仅在必要时强制写入 `SecurityContextRepository`（例如，`HttpSession`）来消除歧义并提高性能。

要选择使用新的`Spring Security 6`默认设置，可以使用以下配置：

```java
public SecurityFilterChain filterChain(HttpSecurity http) {
    http
          .securityContext((securityContext) -> securityContext
                .requireExplicitSave(true)
          );
    return http.build();
}
```

在使用此配置时，任何负责使用`SecurityContext`设置`SecurityContextHolder`的代码都必须确保在请求之间需要持久化时将`SecurityContext`保存到`SecurityContextRepository`。

例如，以下代码：

```java
SecurityContextHolder.setContext(securityContext);
```

应该替换为：

```java
SecurityContextHolder.setContext(securityContext);
securityContextRepository.saveContext(securityContext, httpServletRequest, httpServletResponse);
```

### 将 HttpSessionSecurityContextRepository 更改为 DelegatingSecurityContextRepository

在`Spring Security 5`中，默认的`SecurityContextRepository`是`HttpSessionSecurityContextRepository`。

在`Spring Security 6`中，默认的`SecurityContextRepository`是`DelegatingSecurityContextRepository`。要采用新的`Spring Security 6`默认设置，可以使用以下配置：

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
          // ...
          .securityContext((securityContext) -> securityContext
                .securityContextRepository(new DelegatingSecurityContextRepository(
                      new RequestAttributeSecurityContextRepository(),
                      new HttpSessionSecurityContextRepository()
                ))
          );
    return http.build();
}
```

### 解决 SecurityContextRepository 弃用问题

在`Spring Security 6`中，`SecurityContextRepository`类中的以下方法已被弃用：

```java
Supplier<SecurityContext> loadContext(HttpServletRequest request)
```

应该替换为以下方法：

```java
DeferredSecurityContext loadDeferredContext(HttpServletRequest request)
```

### 改进 RequestCache 的查询

在`Spring Security 5`中，标准程序涉及在每个传入请求中查询保存的请求。在典型配置中，这意味着每个请求都会咨询`HttpSession`以利用`RequestCache`。

在`Spring Security 6`中，新的默认设置是仅在 HTTP 参数`continue`明确定义时才会查询缓存的请求。这种方法使`Spring Security`能够在与`RequestCache`一起工作时跳过不必要的`HttpSession`读取。

### 需要显式调用 SessionAuthenticationStrategy

在`Spring Security 5`中，标准配置依赖于`SessionManagementFilter`来识别用户是否最近进行了认证以及触发`SessionAuthenticationStrategy`。然而，在这种设置中，在典型场景下，每个请求都需要读取`HttpSession`。

在`Spring Security 6`中，新的默认设置是认证机制直接调用`SessionAuthenticationStrategy`。因此，不需要识别何时发生认证，消除了在每个请求中读取`HttpSession`的需求。

在调查会话管理更新之后，我们将深入探讨认证更新。

## 认证更新

我们将检查认证的主要更新，包括采用`SHA-256`用于`Remember Me`功能以及与`AuthenticationServiceExceptions`相关的增强。

### 使用 SHA-256 实现 Remember Me

`Spring Security 6`中`TokenBasedRememberMeServices`实现现在默认使用`SHA-256`用于`Remember Me`令牌，增强了默认的安全立场。这一变化是由于认识到`MD5`是一个易受碰撞攻击和模差攻击的弱散列算法。

新生成的令牌包括有关用于令牌生成的算法的信息。此信息被用于匹配目的。如果算法名称不存在，则使用 `matchingAlgorithm` 属性来验证令牌。这种设计允许从 `MD5` 无缝过渡到 `SHA-256`。

以下代码展示了如何启用 `Remember Me` 功能，使用默认实现：

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http, RememberMeServices rememberMeServices) throws Exception {
       http
             // ...
             .rememberMe(remember -> remember
                   .rememberMeServices(rememberMeServices)
             );
       return http.build();
    }
    @Bean
    RememberMeServices rememberMeServices(UserDetailsService userDetailsService) {
       return new TokenBasedRememberMeServices(myKey, userDetailsService);
    }
}
```

为了接受新的 `Spring Security 6` 默认编码令牌，同时保持与 `MD5` 编码令牌的兼容性，你可以将 `encodingAlgorithm` 属性设置为 `SHA-256`，并将 `matchingAlgorithm` 属性设置为 `MD5`：

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http, RememberMeServices rememberMeServices) throws Exception {
       http
             .rememberMe(remember -> remember
                   .rememberMeServices(rememberMeServices)
             );
       return http.build();
    }
    @Bean
    RememberMeServices rememberMeServices(UserDetailsService userDetailsService) {
       RememberMeTokenAlgorithm encodingAlgorithm = RememberMeTokenAlgorithm.SHA256;
       TokenBasedRememberMeServices rememberMe = new TokenBasedRememberMeServices(myKey, userDetailsService, encodingAlgorithm);
       rememberMe.setMatchingAlgorithm(RememberMeTokenAlgorithm.MD5);
       return rememberMe;
    }
}
```

### 传播 `AuthenticationServiceExceptions`

`AuthenticationFilter` 将 `AuthenticationServiceException` 转发到 `AuthenticationEntryPoint`。由于 `AuthenticationServiceExceptions` 表示服务器端错误而不是客户端错误，在 6.0 版本中，此机制被调整为将它们传播到容器中。

因此，如果你之前通过将 `rethrowAuthenticationServiceException` 设置为 `true` 来启用了此行为，你现在可以按照以下方式消除它：

```java
AuthenticationFilter authenticationFilter = new AuthenticationFilter(...);
AuthenticationEntryPointFailureHandler handler = new AuthenticationEntryPointFailureHandler(...);
handler.setRethrowAuthenticationServiceException(true);
authenticationFilter.setAuthenticationFailureHandler(handler);
```

这可以改为：

```java
AuthenticationFilter authenticationFilter = new AuthenticationFilter(...);
AuthenticationEntryPointFailureHandler handler = new AuthenticationEntryPointFailureHandler(...);
authenticationFilter.setAuthenticationFailureHandler(handler);
```

在审查认证更新之后，我们将深入探讨授权更新。

## 授权更新

在本节中，我们将探讨 `Spring Security` 中授权管理的几个关键增强。我们将首先讨论如何利用 `AuthorizationManager` 进行 `Method Security`，以实现对方法级访问的细粒度控制。接下来，我们将深入探讨利用 `AuthorizationManager` 进行消息安全性，以促进通过消息协议的安全通信。此外，我们还将突出一些弃用，如 `AbstractSecurityWebSocketMessageBrokerConfigurer`。

### 利用 `AuthorizationManager` 进行方法安全性

`AuthorizationManager` API 和直接使用 **Spring AOP**。

如果你在实施这些调整时遇到挑战，请注意，尽管 `@EnableGlobalMethodSecurity` 已被弃用，但在 6.0 版本中尚未移除。这确保了你可以通过继续使用弃用的注解来选择退出。

#### 将全局方法安全性与方法安全性替换

`@EnableGlobalMethodSecurity` 和 `<global-method-security>` 现已弃用，分别由 `@EnableMethodSecurity` 和 `<method-security>` 取代。更新的注解和 XML 元素自动激活 Spring 的 pre-post 注解，并内部使用 `AuthorizationManager`。

#### 在 `@EnableTransactionManagement` 中更改顺序值

`@EnableTransactionManagement` 和 `@EnableGlobalMethodSecurity` 都具有相同的顺序值 `Integer.MAX_VALUE`。因此，它们在 Spring AOP `Advisor` 链中的相对顺序是未定义的。

虽然这通常是可接受的，因为大多数 **Method Security** 表达式不依赖于开放事务来正确运行，但历史上有些情况下需要通过设置它们的顺序值来确保特定的顺序。

相反，`@EnableMethodSecurity` 由于它调度多个拦截器而没有顺序值。与 `@EnableTransactionManagement` 不同，它无法保持向后兼容，因为它无法将所有拦截器放置在同一个顾问链位置。

相反，`@EnableMethodSecurity` 拦截器的顺序值基于偏移量 `0`。例如，`@PreFilter` 拦截器的顺序为 `100`，`@PostAuthorize` 的顺序为 `200`，依此类推。

如果更新后您发现由于缺少开放事务，您的 `Method Security` 表达式无法正常工作，请修改您的交易注解定义如下：

```java
@EnableTransactionManagement(order = 0)
```

#### 使用自定义 @Bean 而不是继承 DefaultMethodSecurityExpressionHandler

为了性能优化，`MethodSecurityExpressionHandler` 已添加了一个新方法，该方法接受一个 `Supplier<Authentication>` 而不是 `Authentication`。

此增强功能允许 `Spring Security` 延迟 `Authentication` 查找，并在使用 `@EnableMethodSecurity` 而不是 `@EnableGlobalMethodSecurity` 时自动使用。

例如，假设您希望对 `@PostAuthorize("hasAuthority('ADMIN')")` 进行自定义评估。在这种情况下，您可以创建一个自定义 `@Bean`，如下所示：

```java
class MyAuthorizer {
    boolean isAdmin(MethodSecurityExpressionOperations root) {
       boolean decision = root.hasAuthority("ADMIN");
       // custom work ...
       return decision;
    }
}
```

然后，在注解中引用它，如下所示：

```java
@PreAuthorize("@authz.isAdmin(#root)")
```

#### 替换权限评估器以公开 MethodSecurityExpressionHandler

`@EnableMethodSecurity` 不会自动检测 `PermissionEvaluator` 以保持其 API 简洁。

如果您已将自定义 `PermissionEvaluator` 声明为 `@Bean`，请将其更新如下：

```java
@Bean
static PermissionEvaluator permissionEvaluator() {
    // ... your evaluator
}
```

更改为：

```java
@Bean
static MethodSecurityExpressionHandler expressionHandler() {
    var expressionHandler = new DefaultMethodSecurityExpressionHandler();
    expressionHandler.setPermissionEvaluator(myPermissionEvaluator);
    return expressionHandler;
}
```

#### 在方法安全中替换任何自定义 AccessDecisionManagers

您的应用程序可能具有自定义的 `AccessDecisionManager` 或 `AccessDecisionVoter` 配置。适应方法将根据每个配置的具体目的而有所不同。继续阅读以确定最适合您场景的调整。

#### 基于共识的使用案例

如果您的应用程序使用默认投票者的 `UnanimousBased`，您可能不需要进行任何更改，因为基于一致性的默认行为与 `@EnableMethodSecurity` 相同。

然而，如果您发现默认的授权管理器不适合，您可以使用 `AuthorizationManagers.allOf` 来构建您的自定义配置。

#### 基于肯定的使用案例

如果您的应用程序依赖于 `AffirmativeBased`，您可以创建一个等效的 `AuthorizationManager`，如下所示：

```java
AuthorizationManager<MethodInvocation> authorization = AuthorizationManagers.anyOf(
       // ... your list of authorization managers
)
```

#### 基于共识的使用案例

对于 `ConsensusBased`，框架没有提供内置的等效功能。在这种情况下，您应该实现一个组合 `AuthorizationManager`，该 `AuthorizationManager` 考虑到委托 `AuthorizationManager` 实例的集合。

#### AccessDecisionVoter 用例

你可以修改类以实现`AuthorizationManager`或创建如下适配器：

```java
public final class PreAuthorizeAuthorizationManagerAdapter implements AuthorizationManager<MethodInvocation> {
    private final SecurityMetadataSource metadata;
    private final AccessDecisionVoter voter;
    public PreAuthorizeAuthorizationManagerAdapter (MethodSecurityExpressionHandler expressionHandler) {
       ExpressionBasedAnnotationAttributeFactory attributeFactory =
             new ExpressionBasedAnnotationAttributeFactory(expressionHandler);
       this.metadata = new PrePostAnnotationSecurityMetadataSource(attributeFactory);
       ExpressionBasedPreInvocationAdvice expressionAdvice = new ExpressionBasedPreInvocationAdvice();
       expressionAdvice.setExpressionHandler(expressionHandler);
       this.voter = new PreInvocationAuthorizationAdviceVoter(expressionAdvice);
    }
    public AuthorizationDecision check(Supplier<Authentication> authentication, MethodInvocation invocation) {
       List<ConfigAttribute> attributes = this.metadata.getAttributes(invocation, AopUtils.getTargetClass(invocation.getThis()));
       int decision = this.voter.vote(authentication.get(), invocation, attributes);
       if (decision == ACCESS_GRANTED) {
          return new AuthorizationDecision(true);
       }
       if (decision == ACCESS_DENIED) {
          return new AuthorizationDecision(false);
       }
       return null; // abstain
    }
}
```

#### AfterInvocationManager 或 AfterInvocationProvider 用例

`AfterInvocationManager`和`AfterInvocationProvider`负责对调用结果进行授权决策。例如，在方法调用的上下文中，它们确定方法返回值的授权。

在`Spring Security 3.0`中，授权的决策过程通过`@PostAuthorize`和`@PostFilter`注解进行了标准化。`@PostAuthorize`用于确定整个返回值是否允许返回。另一方面，`@PostFilter`用于从返回的集合、数组或流中过滤单个条目。

这两个注解应该能满足大多数需求，并且鼓励过渡到其中一个或两个，因为`AfterInvocationProvider`和`AfterInvocationManager`现在已被弃用。

#### RunAsManager 用例

目前还没有`RunAsManager`的直接替代品，尽管正在考虑引入一个。

然而，如果需要，修改`RunAsManager`以与`AuthorizationManager` API 保持一致相对简单。

这里有一些伪代码可以帮助你开始：

```java
public final class RunAsAuthorizationManagerAdapter<T> implements AuthorizationManager<T> {
    private final RunAsManager runAs = new RunAsManagerImpl();
    private final SecurityMetadataSource metadata;
    private final AuthorizationManager<T> authorization;
    // ... constructor
    public AuthorizationDecision check(Supplier<Authentication> authentication, T object) {
       Supplier<Authentication> wrapped = (auth) -> {
          List<ConfigAttribute> attributes = this.metadata.getAttributes(object);
          return this.runAs.buildRunAs(auth, object, attributes);
       };
       return this.authorization.check(wrapped, object);
    }
}
```

#### 验证 AnnotationConfigurationException

`@EnableMethodSecurity`和`<method-security>`启用对`Spring Security`的非重复或不可兼容注解的更严格执行。如果你在过渡到任一配置后在日志中遇到`AnnotationConfigurationException`，请按照异常消息中提供的说明来纠正应用程序的`方法安全`注解使用。

## 利用 AuthorizationManager 进行消息安全

消息安全通过`AuthorizationManager` API 和直接使用 Spring AOP 得到了增强。

要为消息安全配置`AuthorizationManager`，你需要遵循以下步骤：

1.  确保所有消息都定义了授权规则：

    ```java
    @Override
    protected void configureInbound(MessageSecurityMetadataSourceRegistry messages) {
        messages
              .simpTypeMatchers(CONNECT, DISCONNECT, UNSUBSCRIBE).permitAll()
              .simpDestMatchers("/user/queue/errors").permitAll()
              .simpDestMatchers("/admin/**").hasRole("ADMIN")
              .anyMessage().denyAll();
    }
    ```

1.  添加`@EnableWebSocketSecurity`注解。

1.  使用`AuthorizationManager<Message<?>>`的实例：

    ```java
    @Bean
    AuthorizationManager<Message<?>> messageSecurity(MessageMatcherDelegatingAuthorizationManager.Builder messages) {
        messages
              .simpTypeMatchers(CONNECT, DISCONNECT, UNSUBSCRIBE).permitAll()
              .simpDestMatchers("/user/queue/errors").permitAll()
              .simpDestMatchers("/admin/**").hasRole("ADMIN")
              .anyMessage().denyAll();
        return messages.build();
    }
    ```

现在我们已经检查了消息安全的`AuthorizationManager`配置，我们将深入了解与`AbstractSecurityWebSocketMessageBrokerConfigurer`相关的修改。

## 弃用 AbstractSecurityWebSocketMessageBrokerConfigurer

如果你正在使用 Java 配置，你现在可以直接扩展`WebSocketMessageBrokerConfigurer`。

例如，如果你的类扩展了`AbstractSecurityWebSocketMessageBrokerConfigurer`并命名为`WebSocketSecurityConfig`，那么可以替换为以下内容：

```java
@EnableWebSocketSecurity
@Configuration
public class WebSocketSecurityConfig implements WebSocketMessageBrokerConfigurer {
    // ...
}
```

在明确了弃用`AbstractSecurityWebSocketMessageBrokerConfigurer`实现的原因之后，现在让我们深入了解利用`AuthorizationManager`进行请求安全的使用。

## 利用 AuthorizationManager 进行请求安全

使用 `AuthorizationManager` API 简化了 HTTP 请求安全。我们将在 `Spring` `Security 6.x` 中解释 `AuthorizationManager` 对安全请求的更改。

### 确保所有请求都有明确的授权规则

在 `Spring Security 5.8` 及更早版本中，默认允许没有授权规则的请求。然而，为了更健壮的安全态势，`Spring Security 6.0` 的默认做法是默认拒绝。这意味着任何缺少显式授权规则的请求将默认被拒绝。

如果你已经有一个满足你要求的 `anyRequest` 规则，你可以跳过此步骤：

```java
http
       .authorizeRequests((authorize) -> authorize
             .filterSecurityInterceptorOncePerRequest(true)
             .mvcMatchers("/app/**").hasRole("APP")
             // ...
             .anyRequest().denyAll()
       )
```

如果你已经转换到 `authorizeHttpRequests`，推荐的修改保持不变。

### 转换到 AuthorizationManager

要采用 `AuthorizationManager` 的使用，你可以使用 Java 配置中的 `authorizeHttpRequests` 或使用 XML 配置中的 `use-authorization-manager`：

```java
http
       .authorizeHttpRequests((authorize) -> authorize
             .shouldFilterAllDispatcherTypes(false)
             .mvcMatchers("/app/**").hasRole("APP")
             // ...
             .anyRequest().denyAll()
       )
```

### 从 hasIpAddress 迁移到 access(AuthorizationManager)

要从 `hasIpAddress` 迁移到 `access(AuthorizationManager)`，请使用：

```java
IpAddressMatcher hasIpAddress = new IpAddressMatcher("127.0.0.1");
http
       .authorizeHttpRequests((authorize) -> authorize
                   .requestMatchers("/app/**").access((authentication, context) ->
                         new AuthorizationDecision(hasIpAddress.matches(context.getRequest()))
                               // ...
                               .anyRequest().denyAll()
             ))
```

重要提示

通过 IP 地址进行安全保护本质上是微妙的。因此，没有意向将此支持转移到 `authorizeHttpRequests`。

### 将 SpEL 表达式转换为 AuthorizationManager

当涉及到授权规则时，Java 通常比 SpEL 更容易测试和维护。因此，`authorizeHttpRequests` 不提供声明 String SpEL 的方法。相反，你可以创建自己的 `AuthorizationManager` 实现或使用 `WebExpressionAuthorizationManager`。

| **SpEL** | **AuthorizationManager** | **WebExpressionAuthorizationManager** |
| --- | --- | --- |
| `mvcMatchers("/complicated/**").access("hasRole` **('ADMIN') &#124;&#124;** `hasAuthority ('SCOPE_read')")` | `mvcMatchers("/complicated/**").access(anyOf(hasRole` **("ADMIN"),** `hasAuthority ("SCOPE_read"))` | `mvcMatchers("/complicated/**").access` **(**`new WebExpressionAuthorization` **Manager("hasRole('ADMIN') &#124;&#124;** `hasAuthority('SCOPE_read')"))` |

表 16.4 – SpEL 迁移选项

### 转换为过滤所有调度器类型

在 `Spring Security 5.8` 及更早版本中，授权仅在每次请求中执行一次。因此，在 `REQUEST` 之后运行的调度器类型如 `FORWARD` 和 `INCLUDE` 默认不受保护。建议 `Spring Security` 保护所有调度器类型。因此，在 6.0 版本中，`Spring Security` 修改了此默认行为。

要做到这一点，你应该更改：

```java
http
       .authorizeHttpRequests((authorize) -> authorize
             .shouldFilterAllDispatcherTypes(false)
             .mvcMatchers("/app/**").hasRole("APP")
             // ...
             .anyRequest().denyAll()
       )
```

转换为：

```java
http
       .authorizeHttpRequests((authorize) -> authorize
             .shouldFilterAllDispatcherTypes(true)
             .mvcMatchers("/app/**").hasRole("APP")
             // ...
             .anyRequest().denyAll()
       )
```

然后，设置：

```java
spring.security.filter.dispatcher-types=request,async,error,forward,include
```

如果你使用的是 `AbstractSecurityWebApplicationInitializer`，建议重写 `getSecurityDispatcherTypes` 方法并返回所有调度器类型：

```java
public class SecurityWebApplicationInitializer extends AbstractSecurityWebApplicationInitializer {
    @Override
    protected EnumSet<DispatcherType> getSecurityDispatcherTypes() {
       return EnumSet.of(DispatcherType.REQUEST, DispatcherType.ERROR, DispatcherType.ASYNC,
             DispatcherType.FORWARD, DispatcherType.INCLUDE);
    }
}
```

#### 在使用 Spring MVC 时允许 FORWARD

当 Spring MVC 识别出视图名称与实际视图之间的映射时，它将启动对视图的转发。如前节所示，`Spring Security 6.0` 默认将对 `FORWARD` 请求应用授权。

### 替换任何自定义 filter-security AccessDecisionManager

在本节中，我们将探讨不同的用例，以根据`AccessDecisionManager`替换自定义 filter-security。

#### UnanimousBased 用例

如果你的应用程序依赖于`UnanimousBased`，首先调整或替换任何`AccessDecisionVoter`。随后，你可以创建一个`AuthorizationManager`，如下所示：

```java
@Bean
AuthorizationManager<RequestAuthorizationContext> requestAuthorization() {
    PolicyAuthorizationManager policy = ...;
    LocalAuthorizationManager local = ...;
    return AuthorizationManagers.allOf(policy, local);
}
```

然后，将其集成到 DSL 中，如下所示：

```java
http
       .authorizeHttpRequests((authorize) -> authorize.anyRequest().access(requestAuthorization))
// ...
```

#### AffirmativeBased 用例

如果你的应用程序使用`AffirmativeBased`，你可以创建一个等效的`AuthorizationManager`，如下所示：

```java
@Bean
AuthorizationManager<RequestAuthorizationContext> requestAuthorization() {
    PolicyAuthorizationManager policy = ...;
    LocalAuthorizationManager local = ...;
    return AuthorizationManagers.anyOf(policy, local);
}
```

然后，将其集成到 DSL 中，如下所示：

```java
http
       .authorizeHttpRequests((authorize) -> authorize.anyRequest().access(requestAuthorization))
// ...
```

#### ConsensusBased 用例

如果你的应用程序使用`ConsensusBased`，框架没有提供等效的解决方案。在这种情况下，你应该实现一个复合`AuthorizationManager`，该`AuthorizationManager`考虑了委托`AuthorizationManagers`的集合。

#### 自定义 AccessDecisionVoter 用例

如果你的应用程序正在使用`AccessDecisionVoter`，你可以修改该类以实现`AuthorizationManager`或创建一个适配器。由于不了解你自定义投票器的具体功能，提供通用的解决方案具有挑战性。

然而，这里有一个示例，展示了如何适配`SecurityMetadataSource`和`AccessDecisionVoter`以用于`anyRequest().authenticated()`：

```java
public final class AnyRequestAuthenticatedAuthorizationManagerAdapter implements AuthorizationManager<RequestAuthorizationContext> {
    private final SecurityMetadataSource metadata;
    private final AccessDecisionVoter voter;
    public PreAuthorizeAuthorizationManagerAdapter(SecurityExpressionHandler expressionHandler) {
       Map<RequestMatcher, List<ConfigAttribute>> requestMap = Collections.singletonMap(
             AnyRequestMatcher.INSTANCE, Collections.singletonList(new SecurityConfig("authenticated")));
       this.metadata = new DefaultFilterInvocationSecurityMetadataSource(requestMap);
       WebExpressionVoter voter = new WebExpressionVoter();
       voter.setExpressionHandler(expressionHandler);
       this.voter = voter;
    }
    public AuthorizationDecision check(Supplier<Authentication> authentication, RequestAuthorizationContext context) {
       List<ConfigAttribute> attributes = this.metadata.getAttributes(context);
       int decision = this.voter.vote(authentication.get(), invocation, attributes);
       if (decision == ACCESS_GRANTED) {
          return new AuthorizationDecision(true);
       }
       if (decision == ACCESS_DENIED) {
          return new AuthorizationDecision(false);
       }
       return null; // abstain
    }
}
```

在阐明`AuthorizationManager`在请求安全中的用法后，现在让我们深入了解 OAuth 的更新。

## OAuth 更新

在本节中，我们将深入了解 OAuth 更新，特别是关注与在`oauth2Login()`中更改默认权限以及有关 OAuth2 客户端的弃用。

### 更改默认的 oauth2Login()权限

在`Spring Security 5`中，当用户使用`oauth2Login()`进行身份验证时，分配给他们的默认`GrantedAuthority`是`ROLE_USER`。

在`Spring Security 6`中，使用 OAuth2 提供程序进行身份验证的用户被分配默认权限`OAUTH2_USER`，而使用`OpenID Connect 1.0`提供程序进行身份验证的用户被分配默认权限`OIDC_USER`。这些默认权限根据用户是否使用`OAuth2`或`OpenID Connect` `1.0`提供程序进行身份验证，为用户提供更明确的分类。

如果你的应用程序依赖于如`hasRole("USER")`或`hasAuthority("ROLE_USER")`之类的授权规则或表达式，基于特定的权限授予访问权限，请注意，`Spring Security 6`中的更新默认设置将影响你的应用程序。

要在`Spring Security 6`中采用新的默认设置，你可以使用以下配置：

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
          // ...
          .oauth2Login(oauth2Login -> oauth2Login
                .userInfoEndpoint(userInfo -> userInfo
                      .userAuthoritiesMapper(grantedAuthoritiesMapper())
                )
          );
    return http.build();
}
private GrantedAuthoritiesMapper grantedAuthoritiesMapper() {
    return authorities -> {
       Set<GrantedAuthority> mappedAuthorities = new HashSet<>();
       authorities.forEach(authority -> {
          GrantedAuthority mappedAuthority;
          if (authority instanceof OidcUserAuthority) {
             OidcUserAuthority userAuthority = (OidcUserAuthority) authority;
             mappedAuthority = new OidcUserAuthority(
                   "OIDC_USER", userAuthority.getIdToken(), userAuthority.getUserInfo());
          } else if (authority instanceof OAuth2UserAuthority) {
             OAuth2UserAuthority userAuthority = (OAuth2UserAuthority) authority;
             mappedAuthority = new OAuth2UserAuthority(
                   "OAUTH2_USER", userAuthority.getAttributes());
          } else {
             mappedAuthority = authority;
          }
          mappedAuthorities.add(mappedAuthority);
       });
       return mappedAuthorities;
    };
}
```

### 处理 OAuth2 客户端的弃用

在`Spring Security 6`中，OAuth2 客户端中已删除过时的类和方法。以下列出了弃用项及其相应的直接替代品：

| **类** | **弃用列表** |
| --- | --- |

| `ServletOAuth2 AuthorizedClientExchange FilterFunction` | 可以用以下之一替换`setAccessTokenExpiresSkew(…)`方法：

+   `ClientCr` `edentialsOAuth2Authorized ClientProvider#setClockSkew(…)`

+   `RefreshTokenOAuth2AuthorizedClient Provider#setClockSkew(…)`

+   `JwtBearerOA` `uth2Authorized ClientProvider#setClockSkew(…)`

方法 `setClientCredentials` `TokenResponseClient(…)` 可以用构造函数 `ServletOAuth2Authorized` `ClientExchangeFilterFunction (OAuth2AuthorizedClientManager)` 替换 |

| `OidcUserInfo` | 方法 `phoneNumberVerified(String)` 可以用 `phoneNumberVerified(Boolean)` 替换 |
| --- | --- |
| `OAuth2Authorized ClientArgument` `Resolver` | 方法 `setClientCredentialsTokenResponseClient(…)` 可以用构造函数 `OAuth2AuthorizedClient` `ArgumentResolver (OAuth2AuthorizedClientManager)` 替换 |
| `ClaimAccessor` | 方法 `containsClaim(…)` 可以用 `hasClaim(…)` 替换 |
| `OidcClient InitiatedLogout` `SuccessHandler` | 方法 `setPostLogoutRedirectUri(URI)` 可以用 `setPostLogoutRedirectUri(String)` 替换 |
| `HttpSessionOAuth2 Authorization` `RequestRepository` | 方法 `setAllowMultipleAuthorizationRequests(…)` 没有直接替换项 |
| `AuthorizationRequest Repository` | 方法 `removeAuthorizationRequest(HttpServletRequest)` 可以用 `removeAuthorizationRequest(HttpServletRequest, HttpServletResponse)` 替换 |
| `ClientRegistration` | 方法 `getRedirectUriTemplate()` 可以用 `getRedirectUri()` 替换 |
| `ClientRegistration .Builder` | 方法 `redirectUriTemplate(…)` 可以用 `redirectUri(…)` 替换 |
| `AbstractOAuth2 Authorization` `GrantRequest` | 构造函数 `AbstractOAuth2Authorization` `GrantRequest(AuthorizationGrantType)` 可以用 `AbstractOAuth2Authorization` `GrantRequest(AuthorizationGrantType, ClientRegistration)` 替换 |
| `ClientAuthentication Method` | 静态字段 `BASIC` 可以用 `CLIENT_SECRET_BASIC` 替换，静态字段 `POST` 可以用 `CLIENT_SECRET_POST` 替换 |
| `OAuth2Access TokenResponse` `HttpMessage Converter` | 字段 `tokenResponseConverter` 没有直接替换项，方法 `setTokenResponseConverter(…)` 可以用 `setAccessTokenResponseConverter(…)` 替换，字段 `tokenResponseParametersConverter` 没有直接替换项，方法 `setTokenResponseParametersConverter(…)` 可以用 `setAccessTokenResponse` `ParametersConverter(…)` 替换 |
| `Nimbus AuthorizationCode` `TokenResponseClient` | 类 `NimbusAuthorizationCode` `TokenResponseClient` 可以用 `DefaultAuthorizationCode` `TokenResponseClient` 替换 |
| `NimbusJwt DecoderJwkSupport` | 类 `NimbusJwtDecoderJwkSupport` 可以用 `NimbusJwtDecoder` 或 `JwtDecoders` 替换 |
| `ImplicitGrant Configurer` | 类 `ImplicitGrantConfigurer` 没有直接替换项 |
| `Authorization GrantType` | 静态字段 `IMPLICIT` 没有直接替换项 |
| `OAuth2Authorization ResponseType` | 静态字段 `TOKEN` 没有直接替换项 |
| `OAuth2Authorization Request` | 静态方法 `implicit()` 没有直接替换项 |
| `JwtAuthentication Converter` | `extractAuthorities` 方法将被弃用并删除。建议不要扩展 `JwtAuthenticationConverter`，而是使用 `JwtAuthenticationConverter#setJwtGrantedAuthoritiesConverter` 提供自定义授权转换器。 |

表 16.5 – OAuth2 废弃列表

重要提示

不建议使用隐式授权类型，并且 `Spring` `Security 6` 中已移除所有相关支持。

在介绍完 `Spring Security 6` OAuth 更新后，现在让我们深入了解 SAML 的更新。

## SAML 更新

`Spring Security` 过滤器链。

在 `Spring Security` 的 SAML 2.0 服务提供者支持的情况下，您可以使用 `Spring Security` 的 `saml2Login` 和 `saml2Logout` DSL 方法来启用它。这些方法会自动选择适当的过滤器，并将它们放置在过滤器链的相关位置。

在以下章节中，我们将探讨主要的 SAML 更新。

### 转向 OpenSAML 4

`Spring Security 6` 停止了对 `OpenSAML 3` 的支持，并将其基线升级到 `OpenSAML 4`。

要升级到 `Spring Security 6` 的 `SAML` 支持，您需要使用 `4.1.1` 或更高版本。

### 利用 OpenSaml4AuthenticationProvider

为了同时适应 `Spring Security` 引入的 `OpenSamlAuthenticationProvider` 和 `OpenSaml4AuthenticationProvider`。然而，随着 `Spring Security 6` 的移除，`OpenSamlAuthenticationProvider` 也已被停止使用。

需要注意的是，并非所有来自 `OpenSamlAuthenticationProvider` 的方法都直接转移到 `OpenSaml4AuthenticationProvider`。因此，在实施挑战时，需要进行一些调整以应对这些变化。

### 避免使用 SAML 2.0 Converter 构造函数

在 `Spring Security` SAML 2.0 支持的初始版本中，`Saml2MetadataFilter` 和 `Saml2AuthenticationTokenConverter` 最初配备了 `Converter` 类型的构造函数。这种抽象级别在类的发展中带来了挑战，导致在后续版本中引入了专门的接口 `RelyingPartyRegistrationResolver`。

在 6.0 版本中，`Converter` 构造函数已被删除。为了适应这一变化，修改实现 `Converter<HttpServletRequest, RelyingPartyRegistration>` 的类，改为实现 `RelyingPartyRegistrationResolver`。

### 转向使用 Saml2AuthenticationRequestResolver

在 `Spring Security 6` 中，`Saml2AuthenticationContextResolver`、`Saml2AuthenticationRequestFactory` 以及相关的 `Saml2WebSsoAuthenticationRequestFilter` 都已被删除。

它们被 `Saml2AuthenticationRequestResolver` 和 `Saml2WebSsoAuthenticationRequestFilter` 的新构造函数所取代。修订后的接口消除了这些类之间的不必要传输对象。

尽管大多数应用程序不需要重大更改，但如果您目前使用或配置了 `Saml2AuthenticationRequestContextResolver` 或 `Saml2Authentication RequestFactory`，请考虑以下步骤以使用 `Saml2Authentication RequestResolver` 进行过渡。

#### 替换 setAuthenticationRequestContextConverter

在 `Spring Security 6` 中，您应该将 `setAuthenticationRequestContextConverter` 替换为 `setAuthnRequestCustomizer`。

此外，由于 `setAuthnRequestCustomizer` 可以直接访问 `HttpServletRequest`，因此不需要 `Saml2AuthenticationRequestContextResolver`。只需使用 `setAuthnRequestCustomizer` 直接从 `HttpServletRequest` 中检索所需信息。

#### 替换 setProtocolBinding

以下使用 `setProtocolBinding` 的实现：

```java
@Bean
Saml2AuthenticationRequestFactory authenticationRequestFactory() {
    OpenSaml4AuthenticationRequestFactory factory = new OpenSaml4AuthenticationRequestFactory();
    factory.setProtocolBinding("urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST")
    return factory;
}
```

可以替换如下：

```java
@Bean
Saml2AuthenticationRequestResolver authenticationRequestResolver() {
    OpenSaml4AuthenticationRequestResolver reaolver = new OpenSaml4AuthenticationRequestResolver(registrations);
    resolver.setAuthnRequestCustomizer((context) -> context.getAuthnRequest()
          .setProtocolBinding("urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"));
    return resolver;
}
```

重要提示

由于 `Spring Security` 专门支持用于身份验证的 `POST` 绑定，因此在此时覆盖协议绑定不会产生显著的价值。

#### 利用最新的 Saml2AuthenticationToken 构造函数

在 `Spring Security 6` 之前，`Saml2AuthenticationToken` 构造函数需要多个单独的设置作为参数，当添加新参数时，这会带来挑战。认识到这些设置中的大多数都是 `RelyingPartyRegistration` 的固有属性，因此引入了一个更稳定的构造函数。这个新的构造函数允许提供 `RelyingPartyRegistration`，更接近于 `OAuth2LoginAuthenticationToken` 的设计。

尽管大多数应用程序通常不会直接实例化此类，因为它通常由 `Saml2WebSsoAuthenticationFilter` 处理，但如果您的应用程序确实实例化了它，您应该按照以下方式更新构造函数：

```java
new Saml2AuthenticationToken(saml2Response, registration)
```

#### 利用 RelyingPartyRegistration 中的更新方法

在 `Spring Security` 的初始版本中，`RelyingPartyRegistration` 方法及其功能。为了解决此问题并适应向 `RelyingPartyRegistration` 引入的额外功能，有必要通过将方法重命名为与规范语言一致来澄清歧义。

在检查从 `Spring Security 5.x` 到 `Spring Security 6.x` 的各种配置迁移选项之后，下一节将演示将 JDBC 应用程序从 `Spring Security 5.x` 迁移到 `Spring Security 6.x` 的实际示例。

# 应用从 Spring Security 5.x 到 Spring Security 6.x 的迁移步骤

在本节中，我们将深入探讨将一个示例应用程序从 `Spring Security 5.x` 迁移到 `Spring Security 6.x` 的过程。这次迁移旨在确保与较新版本提供的最新功能、改进和安全增强保持兼容性。

重要提示

使用 `Spring Security 5.x` 编写的应用程序的初始状态可在项目 `chapter16.00-calendar` 中找到。

## 检查应用程序依赖项

以下片段定义了 `Spring` `Security 5.x` 需要的初始依赖项：

```java
//build.gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '2.7.18'
    id 'io.spring.dependency-management' version '1.1.4'
}
...
dependencies {
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    // JPA / ORM / Hibernate:
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'org.thymeleaf.extras:thymeleaf-extras-springsecurity5'
    // H2 db
    implementation 'com.h2database:h2'
    // webjars
    implementation 'org.webjars:webjars-locator:0.50'
    implementation 'org.webjars:bootstrap:5.3.2'
    //Tests
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

在 `build.gradle` 的迁移版本中，我们将 `Spring Security` 升级到 6.x 版本：

```java
//build.gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.1
    id 'io.spring.dependency-management' version '1.1.4'
}
...
dependencies {
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    // JPA / ORM / Hibernate:
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'org.thymeleaf.extras:thymeleaf-extras-springsecurity6'
    // H2 db
    implementation 'com.h2database:h2'
    // webjars
    implementation 'org.webjars:webjars-locator:0.50'
    implementation 'org.webjars:bootstrap:5.3.2'
    //Tests
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

## 从 javax 迁移到 jakarta 命名空间

`Spring Security 6` 中从 `javax` 命名空间迁移到 `jakarta` 命名空间主要是由于 Java 生态系统中的变化。

这一变化是由于 Java **企业版**（**EE**）规范的发展和由社区领导的 Jakarta EE 努力所致。

## 替换 WebSecurityConfigurerAdapter 并公开 SecurityFilterChain Bean

如前所述，`Spring Security 6` 引入了增强和改进，以简化安全配置。最近版本中的一个显著演变是，用更灵活的方法公开 `SecurityFilterChain` 对象来替换传统的 `WebSecurityConfigurerAdapter`。

这种范式转变为开发者提供了对安全配置的更大控制和定制，促进了针对特定应用程序需求定制的更细粒度的安全设置。

迁移之前，`SecurityConfig.java` 的样子如下：

```java
//src/main/java/com/packtpub/springsecurity/configuration/SecurityConfig.java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Description("Configure HTTP Security")
    @Override
    protected void configure(final HttpSecurity http) throws Exception {
       http.authorizeRequests(authorizeRequests -> authorizeRequests
             .antMatchers("/webjars/**").permitAll()
             .antMatchers("/css/**").permitAll()
             .antMatchers("/favicon.ico").permitAll()
             .antMatchers("/actuator/**").permitAll()
             .antMatchers("/signup/*").permitAll()
             .antMatchers("/").permitAll()
             .antMatchers("/login/*").permitAll()
             .antMatchers("/logout/*").permitAll()
             .antMatchers("/admin/h2/**").access("isFullyAuthenticated() and hasRole('ADMIN')")
             .antMatchers("/admin/*").hasRole("ADMIN")
             .antMatchers("/events/").hasRole("ADMIN")
             .antMatchers("/**").hasRole("USER")
       );
       // The default AccessDeniedException
       http.exceptionHandling(handler -> handler
             .accessDeniedPage("/errors/403")
       );
       // Login Configuration
       http.formLogin(form -> form
             .loginPage("/login/form")
             .loginProcessingUrl("/login")
             .failureUrl("/login/form?error")
             .usernameParameter("username") // redundant
             .passwordParameter("password") // redundant
             .defaultSuccessUrl("/default", true)
             .permitAll()
       );
       // Logout Configuration
       http.logout(form -> form
             .logoutUrl("/logout")
             .logoutSuccessUrl("/login/form?logout")
             .permitAll()
       );
       // Allow anonymous users
       http.anonymous();
       // CSRF is enabled by default, with Java Config
       //NOSONAR
       http.csrf().disable();
       // Cross Origin Resource Sharing
       http.cors().disable();
       // HTTP Security Headers
       http.headers().disable();
       // Enable <frameset> in order to use H2 web console
       http.headers().frameOptions().disable();
    }
...
}
```

迁移后，我们移除了 `WebSecurityConfigurerAdapter` 并如下公开了一个 `SecurityFilterChain` 对象：

```java
//src/main/java/com/packtpub/springsecurity/configuration/SecurityConfig.java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
       http.authorizeHttpRequests( authz -> authz
                   .requestMatchers("/webjars/**").permitAll()
                   .requestMatchers("/css/**").permitAll()
                   .requestMatchers("/favicon.ico").permitAll()
                   .requestMatchers("/").permitAll()
                   .requestMatchers("/login/*").permitAll()
                   .requestMatchers("/logout").permitAll()
                   .requestMatchers("/signup/*").permitAll()
                   .requestMatchers("/errors/**").permitAll()
                   // H2 console
                   .requestMatchers("/admin/h2/**")
                   .access(new WebExpressionAuthorizationManager("isFullyAuthenticated() and hasRole('ADMIN')"))
                   .requestMatchers("/events/").hasRole("ADMIN")
                   .requestMatchers("/**").hasRole("USER"))
             .exceptionHandling(exceptions -> exceptions
                   .accessDeniedPage("/errors/403"))
             .formLogin(form -> form
                   .loginPage("/login/form")
                   .loginProcessingUrl("/login")
                   .failureUrl("/login/form?error")
                   .usernameParameter("username")
                   .passwordParameter("password")
                   .defaultSuccessUrl("/default", true)
                   .permitAll())
             .logout(form -> form
                   .logoutUrl("/logout")
                   .logoutSuccessUrl("/login/form?logout")
                   .permitAll())
             // CSRF is enabled by default, with Java Config
             .csrf(AbstractHttpConfigurer::disable);
       // For H2 Console
       http.headers(headers -> headers.frameOptions(FrameOptionsConfig::disable));
       return http.build();
    }
...
}
```

由于在 `Spring Security 6.x` 中 `@EnableWebSecurity` 不再包含 `@Configuration`，我们在 `SecurityConfig.java` 的迁移版本中声明了这两个注解。

重要提示

您的代码现在应如下所示：`chapter16.01-calendar`。

# 摘要

本章回顾了在将现有的 `Spring Security 5.x` 项目升级到 `Spring Security 6.x` 时将遇到的重大和细微变化。在本章中，我们回顾了可能促使升级的重大框架增强。我们还检查了升级需求、依赖项、常见代码类型和配置更改，这些更改将防止应用程序在升级后工作。我们还概述了 `Spring Security` 作者在代码库重构过程中所做的整体代码重组更改。

如果这是您第一次阅读本章，我们希望您回到书的其余部分，并使用本章作为指南，以便尽可能顺利地进行 `Spring Security 6.x` 的升级！
