# 第八章：使用 ORM 和 NoSQL DB 的 Spring 安全

在本章中，我们将涵盖：

+   Spring Security 与 Hibernate 一起使用@preAuthorize 注释

+   Spring Security 与 Hibernate 一起使用身份验证提供程序和@preAuthorize 注释

+   Spring Security 与 Hibernate 一起使用用户详细信息服务和 Derby 数据库

+   Spring Security 与 MongoDB

# 介绍

Spring 框架已经设计成可以轻松集成类似于 Mybatis、Hibernate 等 ORM 框架。Hibernate 教程非常详细，并且可以在 JBoss 网站上找到。Hibernate 为我们提供了数据持久性。

在本章中，我们将看到如何将 Spring Security 与 ORM 框架集成。我们还将将 Spring Security 与最新的 MongoDB 集成。

我们将首先进行一些与 Hibernate 和 Spring 相关的基本设置。由于本章涉及数据库相关内容，我们需要为本章中使用的所有食谱创建一个数据库。我正在使用带有 maven 的 NetBeans IDE。我觉得 NetBeans IDE 与其他 IDE 相比非常先进。

## 设置 Spring Hibernate 应用程序

我们将创建一个简单的恐怖电影应用程序，该应用程序将在 UI 中显示一系列恐怖电影，并具有一些**CRUD**（**创建、读取、更新和删除**）功能。设置*Spring Hibernate*应用程序涉及以下步骤：

1.  在 Derby 中创建一个`horrormoviedb`数据库。您可以使用 NetBeans。

1.  单击**服务**选项卡，您将看到**数据库**。

1.  右键单击**JavaDB**以查看**创建数据库...**选项。选择**创建数据库...**选项。![设置 Spring Hibernate 应用程序](img/7525OS_08_01.jpg)

1.  在数据库`horrormovie`中创建一个表。![设置 Spring Hibernate 应用程序](img/7525OS_08_02.jpg)

1.  在表中创建列，并将列命名为`horrormovie_id`、`horrormovie_name`和`horrormovie_director`。

1.  创建一个 maven 项目，更新 POM 文件以包含 Spring、Hibernate、Derby 和 Spring Security 依赖项，并在 NetBeans IDE 中打开它。

1.  使用`@table`和`@column`注释创建实体类。

1.  创建`DAO`和`DAOImpl`类来处理 Hibernate 操作。

1.  创建`Service`和`ServiceImpl`类，以在`DAO`和 UI 之间充当中间管理器。

1.  创建一个控制器来处理 UI 部分。

# Spring Security 与 Hibernate 一起使用@preAuthorize 注释

在当前演示中，我们使用了两个不同的数据库。身份验证管理器配置为`tenant1DataSource`，它连接到一个 Derby 数据库，其中保存了用户和角色信息。使用此数据源，我们将进行身份验证和授权。

为显示`horrormovie`列表，我们在 Derby 中创建了另一个数据源，该数据源与 Hibernate 配置文件一起使用。

在`DAOImpl`类的方法中，我们使用了`@preAuthorize`注释。

让我们使用 GlassFish 应用服务器来运行应用程序。

## 准备工作

+   编辑`application-security.xml`。

+   编辑`horrormovie-servlet.xml`。

+   在`DAOImpl`中使用`@preAuthorize`注释。Spring Security 在调用方法时授权用户。

## 如何做...

以下步骤将使用 Hibernate 应用程序进行身份验证和授权：

1.  使用数据源详细信息和 Bean 信息编辑`application-security.xml`文件。

```java
<global-method-security pre-post-annotations="enabled" />

  <http auto-config="false"  use-expressions="true">
    <intercept-url pattern="/login" access="permitAll" />
    <intercept-url pattern="/logout" access="permitAll" />
    <intercept-url pattern="/accessdenied" access="permitAll" />
    <intercept-url pattern="/**"access="hasRole('ROLE_EDITOR')" />
    <form-login login-page="/login" default-target-url="/list" authentication-failure-url="/accessdenied" />
    <logout logout-success-url="/logout" />
  </http>

  <authentication-manager alias="authenticationManager">
    <authentication-provider>
      <jdbc-user-service data-source-ref="tenant1DataSource"
        users-by-username-query=" select username,password ,'true' as enabled from users where username=?"  
        authorities-by-username-query=" 
        select u.username as username, ur.authority as authority from users u, user_roles ur  
        where u.user_id = ur.user_id and u.username =?"
        /> 
    </authentication-provider>
  </authentication-manager>

  <beans:bean id="horrorMovieDAO" class="com.packt.springsecurity.dao.HorrorMovieDaoImpl" />
  <beans:bean id="horrorMovieManager" class="com.packt.springsecurity.service.HorrorMovieManagerImpl" />
  <beans:bean id="tenant1DataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
  <beans:property name="driverClassName" value="org.apache.derby.jdbc.EmbeddedDriver" />
  <beans:property name="url" value="jdbc:derby://localhost:1527/client1" />
  <beans:property name="username" value="client1" />
  <beans:property name="password" value="client1" />

</beans:bean>
```

1.  使用控制器信息编辑`horrormovie-servlet.xml`文件。

```java
<global-method-security pre-post-annotations="enabled" />

  <http auto-config="true">
    <intercept-url pattern="/spring-security-wicket/**" access="ROLE_SELLER"/>
    <intercept-url pattern="/spring-security-wicket/*.*" access="ROLE_SELLER"/> 
    <intercept-url pattern="/**" access="ROLE_SELLER" />
  <http-basic />
</http>
<authentication-manager> 
  <authentication-provider> 
    <jdbc-user-service data-source-ref="MySqlDS" 
      users-by-username-query=" 
      select username,password, enabled   
      from users1 where username=?"  
      authorities-by-username-query=" 
      select u.username, ur.role from users1 u, user_roles ur  
      where u.user_id = ur.user_id and u.username =?  " /> 
  </authentication-provider>
</authentication-manager>
```

它使用 JDBC 进行身份验证服务。

1.  在执行`addHorrorMovie`方法时使用注释，Spring 会检查安全上下文对象的凭据，并进行身份验证和授权；以下是代码：

```java
@Repository
public class HorrorMovieDaoImpl implements HorrorMovieDAO  {

  @Autowired
  private SessionFactory sessionFactory;

  @PreAuthorize("hasRole('ROLE_AUTHOR')")
  @Override
  public void addHorrorMovie(HorrorMovieEntity horrormovie) {
    this.sessionFactory.getCurrentSession().save(horrormovie);
  }

  @SuppressWarnings("unchecked")
  @Override
  public List<HorrorMovieEntity> getAllHorrorMovies() {
    return this.sessionFactory.getCurrentSession().createQuery("from HORRORMOVIE").list();
  }

  @Override
  public void deleteHorrorMovie(Integer horrorMovieId) {
    HorrorMovieEntity horrorMovie = (HorrorMovieEntity)sessionFactory.getCurrentSession().load(HorrorMovieEntity.class, horrorMovieId);
    if (null != horrorMovie) {
      this.sessionFactory.getCurrentSession().delete(horrorMovie);
    }
  }
}
```

1.  以下是一些 SQL 命令：

```java
create table HORRORMOVIE
 (HORRORMOVIE_ID int generated by default as identity 
 (START WITH 2, INCREMENT BY 1),
 HORRORMOVIE_NAME char(50),HORRORMOVIE_DIRECTOR char(50));

insert into HORRORMOVIE values 
 (1, 'EVILDEAD','Fede Alvarez');
insert into HORRORMOVIE values 
 (DEFAULT, 'EVILDEAD2','Fede Alvarez');

```

## 它是如何工作的...

在这个例子中，我们创建了一个 Hibernate 应用程序，并使用了 JDBC 服务进行身份验证。Spring 框架中断了访问应用程序的请求，并要求用户输入凭据。使用`application-security.xml`文件中提供的 JDBC 详细信息对凭据进行验证。

成功后，用户将被重定向到显示电影列表的应用程序。

现在访问以下网址：

`http://localhost:8080/login`

使用 JDBC 服务进行身份验证和授权以及在方法上应用 Spring Security 的截图如下：

示例的工作流程显示在以下截图中：

![它是如何工作的...](img/7525OS_08_03.jpg)![它是如何工作的...](img/7525OS_08_04.jpg)

## 另请参阅

+   *使用身份验证提供程序的 Spring Security 与 Hibernate*配方

+   *使用 Derby 数据库的用户详细信息服务的 Spring Security 与 Hibernate*配方

+   *使用 MongoDB 的 Spring Security*配方

# 使用身份验证提供程序和@preAuthorize 注释的 Spring Security 与 Hibernate

我们正在使用示例`horrormovie`应用程序来演示使用自定义身份验证提供程序和`@preAuthorize`注释的 Spring Security 与 Hibernate。

在这个配方中，我们将创建自己的自定义身份验证提供程序并实现接口身份验证提供程序。我们将在`controller`方法上应用注释，而不是在`hibernate`方法上。

## 准备工作

+   创建一个实现`AuthenticationProvider`接口的新类，并将 Bean 定义添加到`application-security.xml`文件中

+   编辑`application-security.xml`文件

+   在控制器中使用`@preAuthorize`注释

## 如何做...

使用`AuthenticationProvider`接口实现 Spring Security 的以下步骤：

1.  编辑`application-security.xml`文件，添加数据源详细信息和 Bean 信息。

```java
<global-method-security pre-post-annotations="enabled" />

<http auto-config="false"  use-expressions="true">
  <intercept-url pattern="/login" access="permitAll" />
  <intercept-url pattern="/logout" access="permitAll" />
  <intercept-url pattern="/accessdenied" access="permitAll"/>
  <intercept-url pattern="/list" access="hasRole('ROLE_EDITOR')" />
  <intercept-url pattern="/add" access="hasRole('ROLE_EDITOR')" />
  <form-login login-page="/login" default-target-url="/list" authentication-failure-url="/accessdenied" />
  <logout logout-success-url="/logout" />
</http>

  <authentication-manager alias="authenticationManager">
 <authentication-provider ref="MyCustomAuthenticationProvider" />
 </authentication-manager>

  <beans:bean id="horrorMovieDAO" class="com.packt.springsecurity.dao.HorrorMovieDaoImpl" />
  <beans:bean id="horrorMovieManager" class="com.packt.springsecurity.service.HorrorMovieManagerImpl"/>

 <beans:bean id="MyCustomAuthenticationProvider" class="com.packt.springsecurity.controller" />
</beans:beans>
```

1.  编辑`MyCustomAuthenticationProvider`文件。

```java
public class MyCustomAuthenticationProvider implements AuthenticationProvider {
  @Override
  public boolean supports(Class<? extends Object>authentication)
{
    return (UsernamePasswordAuthenticationToken.class.isAssignableFrom(authentication));
    }

 private static Map<String, String> APP_USERS= new HashMap<String, String>(2);
 private static List<GrantedAuthority> APP_ROLES= new ArrayList<GrantedAuthority>();
 static
 {
 APP_USERS.put("ravi", "ravi123");
 APP_USERS.put("chitra", "chitra123");
 APP_ROLES.add(new SimpleGrantedAuthority("ROLE_EDITOR"));
 }

  @Override
  public Authentication authenticate(Authentication auth)
  {
 if (APP_USERS.containsKey(auth.getPrincipal())
 && APP_ROLES.get(auth.getPrincipal()).equals(auth.getCredentials()))
 {
 return new UsernamePasswordAuthenticationToken(auth.getName(), auth.getCredentials(),
 AUTHORITIES);
 }
 throw new BadCredentialsException("Username/Password does not match for "
      + auth.getPrincipal());
    }
  }
}
```

1.  在控制器中使用注释。

```java
AddHorrorMovieController
@PreAuthorize("hasRole('ROLE_EDITOR')")
@RequestMapping(value = "/add", method = RequestMethod.POST)
public String addHorrorMovie(
  @ModelAttribute(value = "horrorMovie") HorrorMovieEntity horrorMovie,
    BindingResult result) {
    horrorMovieManager.addHorrorMovie(horrorMovie);
    return "redirect:/list";
  }
```

## 它是如何工作的...

现在访问以下网址：

`http://localhost:8080/login`

在中断请求后，Spring Security 调用`MyCustomAuthenticationProvider`，该提供程序具有用于身份验证和用户信息的重写 authenticate 方法。用户凭据在`APP_Users`映射中进行验证和授权，成功验证和授权后，用户将被重定向到`spring-security.xml`文件中配置的成功 URL。

使用自定义身份验证提供程序进行身份验证和授权，并在控制器方法上应用 Spring Security 的截图如下：

![它是如何工作的...](img/7525OS_08_05.jpg)![它是如何工作的...](img/7525OS_08_06.jpg)

## 另请参阅

+   *使用@preAuthorize 注释的 Spring Security 与 Hibernate*配方

+   *使用自定义身份验证提供程序和@preAuthorize 注释的 Spring Security 与 Hibernate*配方

+   *使用 Derby 数据库的用户详细信息服务的 Spring Security 与 Hibernate*配方

+   *使用 MongoDB 的 Spring Security*配方

# 使用 Derby 数据库的 UserDetailsService 与 Spring Security 的 Hibernate

到目前为止，我们已经看到了使用各种身份验证提供程序的 Hibernate 和 Spring Security。在本节中，我们将使用 Hibernate 从数据库中检索用户和权限。

为此，我们将实现`UserDetailsService`接口并在接口中实现一个方法。首先，我们需要为用户和角色创建实体类。

我们还将`@preAuthorize`注释移到`controller`类中。

## 准备工作

+   创建一个实现`UserDetailsService`接口的新类，并将 Bean 定义添加到`application-security.xml`文件中

+   编辑`application-security.xml`文件

+   在控制器中使用`@preAuthorize`注释

+   在恐怖数据库中添加`USERS`和`USER_ROLE`表

+   插入角色`ROLE_EDITOR`和名为`ravi`和`ravi123`的用户

## 如何做...

通过实现与 Hibernate 交互的`UserDetailsService`接口来集成 Spring Security 身份验证的以下步骤：

1.  创建一个实现`UserDetailsService`接口的类`MyUserDetailsService`。

```java
public class MyUserDetails implements UserDetailsService {
  @Autowired
  private UsersDAO UsersDAO;
  public UserDetails loadUserByUsername(String userName)
  throws UsernameNotFoundException {

    Users users= UsersDAO.findByUserName(userName);
    boolean enabled = true;
    boolean accountNonExpired = true;
    boolean credentialsNonExpired = true;
    boolean accountNonLocked = true;
    return new User(
      users.getUserName(), 
      users.getUserPassword(), 
      enabled, 
      accountNonExpired, 
      credentialsNonExpired, 
      accountNonLocked,
      getAuthorities(users.getRole().getRoleId().intValue()));
    }

    public Collection<? extends GrantedAuthority>getAuthorities(Integer role) {
    List<GrantedAuthority> authList = getGrantedAuthorities(getRoles(role));
    System.out.println("authList----------->"+authList);
    return authList;
  }

  public List<String> getRoles(Integer role) {

    List<String> roles = new ArrayList<String>();

    if (role.intValue() == 1) {
      roles.add("ROLE_EDITOR");
    } else if (role.intValue() == 2) {
      roles.add("ROLE_AUTHOR");
    }
    return roles;
  }

  public static List<GrantedAuthority> getGrantedAuthorities(List<String> roles) {
  List<GrantedAuthority> authorities = new ArrayList<GrantedAuthority>();
  for (String role : roles) {
    System.out.println("role----------->"+role);
    authorities.add(new SimpleGrantedAuthority(role));
  }
  return authorities;
  }

}
```

1.  编辑`application-security.xml`文件。

```java
<authentication-manager alias="authenticationManager">
  <authentication-provider user-service-ref="MyUserDetails">
    <password-encoder hash="plaintext" />
  </authentication-provider>
</authentication-manager>

<beans:bean id="horrorMovieDAO" class="com.packt.springsecurity.dao.HorrorMovieDaoImpl" />
<beans:bean id="horrorMovieManager" class="com.packt.springsecurity.service.HorrorMovieManagerImpl" />
<beans:bean id="UsersDAO" class="com.packt.springsecurity.dao.UsersDAOImpl" />
<beans:bean id="UsersManager" class="com.packt.springsecurity.service.UsersManagerImpl" />
<beans:bean id="UserRoleDAO" class="com.packt.springsecurity.dao.UserRoleDAOImpl" />
<beans:bean id="UserRoleManager" class="com.packt.springsecurity.service.UserRoleManagerImpl" />

<beans:bean id="MyUserDetails" class="com.packt.springsecurity.service.MyUserDetails" />
</beans:beans>
```

1.  在控制器中使用注释。

```java
@PreAuthorize("hasRole('ROLE_EDITOR')")
@RequestMapping(value = "/add", method = RequestMethod.POST)
public String addHorrorMovie(
  @ModelAttribute(value = "horrorMovie")HorrorMovieEntity horrorMovie,
  BindingResult result) {
    horrorMovieManager.addHorrorMovie(horrorMovie);
    return "redirect:/list";
  }
```

## 它是如何工作的...

现在访问以下 URL：

`http://localhost:8080/login`

首先使用`UserDetailsService`和 Hibernate 进行身份验证和授权。 `UserDetailsService`是 Spring Security 接口，由`MyUserDetailsService`类实现。该类在`application-security.xml`文件中进行配置，以便 Spring Security 调用此实现类使用 Hibernate 加载用户详细信息。 `UsersDAO.findByUserName(userName)`是调用 Hibernate 获取基于传递的用户名的用户信息的方法。

在使用注释将 Spring Security 应用于控制器之后，我们应该能够使用用户名和密码（ravi 和 ravi123）登录。 `<password-encoder hash="plaintext" />`是 Spring Security 支持的哈希算法。成功验证后，用户将被重定向到授权页面。

应用程序的工作流程在以下屏幕截图中演示：

![工作原理...](img/7525OS_08_07.jpg)![工作原理...](img/7525OS_08_08.jpg)

## 另请参阅

+   使用@preAuthorize 注释的 Hibernate 的 Spring Security 配方

+   使用自定义身份验证提供程序和@preAuthorize 注释的 Hibernate 的 Spring Security 配方

+   使用 Derby 数据库的用户详细信息服务的 Spring Security 配方

+   使用 MongoDB 的 Spring Security 配方

# 使用 MongoDB 的 Spring Security

在本节中，让我们看看 Spring Security 如何与 MongoDB 配合使用。 MongoDB 是一种流行的 NOSQL 数据库。它是一个基于文档的数据库。 MongoDB 是用流行的 C++数据库编写的，这使它成为一种面向对象的基于文档的数据库。在 MongoDB 中，查询也是基于文档的，它还提供使用 JSON 样式进行索引以存储和检索数据。最新的 Spring 版本是版本 3.2，已包含在 POC 中。

## 准备工作

+   下载 MongoDB 数据库

+   配置数据文件夹

+   在命令提示符中启动 MongoDB

+   在另一个命令提示符中启动 MongoDB

+   通过向其中插入数据创建`horrordb`数据库

+   执行命令`use horrordb`

+   将 MongoDB 依赖项添加到 POM（项目对象模型）文件

+   将 JSON 依赖项添加到 POM 文件

+   将 Spring 版本升级到 3.2.0，将 Spring Security 升级到 1.4

+   创建一个`MongoUserDetails`类

+   编辑`horror-movie` servlet

+   编辑`Application-security.xml`文件

## 如何做...

以下步骤使用 Mongo 与 Spring Security 来实现`UserDetailsService`接口对用户进行身份验证和授权：

1.  在命令提示符中显示数据库操作如下：

```java
db.horrormovie.insert({horrormovie_id:1,horrormovie_name:
 "omen",horrormovie_director:"Richard Donner"})

db.horrormovie.insert({horrormovie_id:2,horrormovie_name:
 "the conjuring",horrormovie_director:"James Wan"})

db.horrormovie.insert({horrormovie_id:3,horrormovie_name:
 "The Lords of Salem",horrormovie_director:"Rob Zombie"})

db.horrormovie.insert({horrormovie_id:4,horrormovie_name:
 "Evil Dead",horrormovie_director: "Fede Alvarez"})

db.users.insert({id:1,username:"anjana",password:
 "123456",role:1})

db.users.insert({id:2,username:"raghu",password:
 "123456",role:2})

db.users.insert({id:3,username:"shami",password:
 "123456",role:3})

```

1.  创建一个实现`UserDetailsService`接口的`MongoUserDetailsService`类。

```java
@Service
public class MongoUserDetailsService implements UserDetailsService {

  @Autowired
  private UserManager userManager;
  private static final Logger logger = Logger.getLogger(MongoUserDetailsService.class);
  private org.springframework.security.core.userdetails.User userdetails;
  public UserDetails loadUserByUsername(String username)
  throws UsernameNotFoundException {
    boolean enabled = true;
    boolean accountNonExpired = true;
    boolean credentialsNonExpired = true;
    boolean accountNonLocked = true;
    Users users = getUserDetail(username);
    System.out.println(username);
    System.out.println(users.getPassword());
    System.out.println(users.getUsername());
    System.out.println(users.getRole());

    return new User(users.getUsername(), users.getPassword(),enabled,accountNonExpired,credentialsNonExpired,accountNonLocked,getAuthorities(users.getRole()));
  }

  public List<GrantedAuthority> getAuthorities(Integer role) {
    List<GrantedAuthority> authList = new ArrayList<GrantedAuthority>();
      if (role.intValue() == 1) {
        authList.add(new SimpleGrantedAuthority("ROLE_EDITOR"));

      } else if (role.intValue() == 2) {
        authList.add(new SimpleGrantedAuthority("ROLE_AUTHOR"));
    }
    return authList;
  }

  public Users getUserDetail(String username) {
  Users users = userManager.findByUserName(username);
  System.out.println(users.toString());
  return users;
}
```

1.  编辑`application-security.xml`。

```java
<global-method-security pre-post-annotations="enabled" />

<http auto-config="false"  use-expressions="true">
  <intercept-url pattern="/login" access="permitAll" />
  <intercept-url pattern="/logout" access="permitAll" />
  <intercept-url pattern="/accessdenied" access="permitAll" />
  <intercept-url pattern="/list" access="hasRole('ROLE_EDITOR')" />
<!--                <http-basic/>-->
  <form-login login-page="/login" default-target-url="/list" authentication-failure-url="/accessdenied" />
  <logout logout-success-url="/logout" />
</http>

<authentication-manager alias="authenticationManager">
<authentication-provider user-service-ref="mongoUserDetailsService">
<password-encoder hash="plaintext" />
</authentication-provider>
</authentication-manager>
```

1.  编辑`horrormovie-servlet.xml`。

```java
<context:annotation-config />
<context:component-scan base-package="com.packt.springsecurity.mongodb.controller" />
<context:component-scan base-package="com.packt.springsecurity.mongodb.manager" />
<context:component-scan base-package="com.packt.springsecurity.mongodb.dao" />
<context:component-scan base-package="com.packt.springsecurity.mongodb.documententity" />

<bean id="jspViewResolver"
  class="org.springframework.web.servlet.view.InternalResourceViewResolver">
  <property name="viewClass"
  value="org.springframework.web.servlet.view.JstlView" />
  <property name="prefix" value="/WEB-INF/view/" />
  <property name="suffix" value=".jsp" />
</bean>
<mongo:mongo host="127.0.0.1" port="27017" />
<mongo:db-factory dbname="horrordb" />

<bean id="mongoTemplate" class="org.springframework.data.mongodb.core.MongoTemplate">
<constructor-arg name="mongoDbFactory" ref="mongoDbFactory" />
</bean>

<bean id="horrorMovieDAO" class="com.packt.springsecurity.mongodb.dao.HorrorMovieDaoImpl" />
<bean id="horrorMovieManager" class="com.packt.springsecurity.mongodb.manager.HorrorMovieManagerImpl" />
<bean id="UsersDAO" class="com.packt.springsecurity.mongodb.dao.UsersDAOImpl" />
<bean id="userManager" class="com.packt.springsecurity.mongodb.manager.UserManagerImpl" />
<bean id="mongoUserDetailsService" class="com.packt.springsecurity.mongodb.controller.MongoUserDetailsService" />

<bean id="HorroMovieController" class="com.packt.springsecurity.mongodb.controller.HorrorMovieController" />
```

1.  在控制器中使用注释。

```java
@PreAuthorize("hasRole('ROLE_EDITOR')")
@RequestMapping(value = "/add", method = RequestMethod.POST)
public String addHorrorMovie(
@ModelAttribute(value = "horrorMovie")HorrorMovieEntity horrorMovie,
  BindingResult result) {
  horrorMovieManager.addHorrorMovie(horrorMovie);
  return "redirect:/list";
}
```

## 工作原理...

首先使用`MongoDetailsService`和 Spring 数据进行身份验证和授权。 `MongoDetailsService`是`UserDetailsService`的实现，`getUserDetail`(string username)调用`springdata`类从 Mongo 数据库中获取基于传递的用户名的用户凭据。如果根据用户名存在数据，则意味着身份验证成功。然后我们使用注释在控制器方法上应用 Spring Security。

现在我们应该能够使用用户名和密码（ravi 和 123456）登录。

现在访问以下 URL：

`http://localhost:8080/login`

工作流程在以下屏幕截图中演示：

![工作原理...](img/7525OS_08_09.jpg)![工作原理...](img/7525OS_08_10.jpg)

## 另请参阅

+   使用@preAuthorize 注释的 Hibernate 的 Spring Security 配方

+   使用自定义身份验证提供程序和@preAuthorize 注释的 Hibernate 的 Spring Security 配方

+   使用 Derby 数据库的用户详细信息服务的 Spring Security 配方

+   使用 MongoDB 的 Spring Security 配方
