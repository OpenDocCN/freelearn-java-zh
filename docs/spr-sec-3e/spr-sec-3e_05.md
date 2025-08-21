# 第五章：使用 Spring Data 进行身份验证

在上一章中，我们介绍了如何利用 Spring Security 内置的 JDBC 支持。在本章中，我们将介绍 Spring Data 项目，以及如何利用 JPA 对关系数据库进行身份验证。我们还将探讨如何使用 MongoDB 对文档数据库进行身份验证。本章的示例代码基于第四章的 Spring Security 设置，*基于 JDBC 的身份验证*，并已更新以去除对 SQL 的需求，并使用 ORM 处理所有数据库交互。

在本章中，我们将介绍以下主题：

+   与 Spring Data 项目相关的一些基本概念

+   使用 Spring Data JPA 对关系数据库进行身份验证

+   使用 Spring Data MongoDB 对文档数据库进行身份验证

+   如何为处理 Spring Data 集成提供更多灵活性自定义 Spring Security

+   理解 Spring Data 项目

Spring Data 项目的使命是为数据访问提供熟悉的、一致的基于 Spring 的编程模型，同时保留底层数据提供商的独特特性。

以下是 Spring Data 项目的一些强大功能：

+   强大的仓库和自定义对象映射抽象

+   从仓库方法名称派生动态查询

+   实现领域基础类，提供基本属性

+   支持透明审计（创建和最后更改）

+   集成自定义仓库代码的能力

+   通过基于 Java 的配置和自定义 XML 命名空间实现简单的 Spring 集成

+   与 Spring MVC 控制器的高级集成

+   跨存储持久性的实验性支持

该项目简化了数据访问技术、关系型和非关系型数据库、映射框架和基于云的数据服务的使用。这个伞形项目包含了许多特定于给定数据库的子项目。这些项目是在与这些令人兴奋的技术背后的许多公司和开发人员合作开发的。还有许多由社区维护的模块和其他相关模块，包括**JDBC 支持**和**Apache Hadoop**。

以下表格描述了组成 Spring Data 项目的的主要模块：

| **模块** | **描述** |
| --- | --- |
| Spring Data Commons | 将核心 Spring 概念应用于所有 Spring Data 项目 |
| Spring Data Gemfire | 提供从 Spring 应用程序轻松配置和访问 Gemfire 的支持 |
| Spring Data JPA | 使实现基于 JPA 的仓库变得容易 |
| Spring Data Key Value | 基于映射的仓库和 SPIs，可轻松构建键值存储的 Spring Data 模块 |
| Spring Data LDAP | 为 Spring LDAP 提供 Spring Data 仓库支持 |
| Spring Data MongoDB | 基于 Spring 的、对象-文档支持以及 MongoDB 的仓库 |
| Spring Data REST | 将 Spring Data 存储库导出为基于超媒体的 RESTful 资源 |
| Spring Data Redis | 为 Spring 应用程序提供易于配置和访问 Redis 的功能 |
| Spring Data for Apache Cassandra | 适用于 Apache Cassandra 的 Spring Data 模块 |
| Spring Data for Apache Solr | 适用于 Apache Solr 的 Spring Data 模块 |

# Spring Data JPA

Spring Data JPA 项目旨在显著改进数据访问层的 ORM 实现，通过减少实际所需的工作量。开发者只需编写存储库接口，包括自定义查找方法，Spring 将自动提供实现。

以下是一些 Spring Data JPA 项目的特定强大功能：

+   为基于 Spring 和 JPA 构建存储库提供高级支持

+   支持**Querydsl**谓词，因此也支持类型安全的 JPA 查询

+   对领域类进行透明审计

+   分页支持、动态查询执行以及集成自定义数据访问代码的能力

+   在启动时验证`@Query`注解的查询

+   支持基于 XML 的实体映射

+   通过引入`@EnableJpaRepositories`实现基于`JavaConfig`的存储库配置

# 更新我们的依赖项

我们已经包括了本章所需的所有依赖项，所以您不需要对`build.gradle`文件进行任何更新。然而，如果您只是将 Spring Data JPA 支持添加到您自己的应用程序中，您需要在`build.gradle`文件中添加`spring-boot-starter-data-jpa`作为依赖项，如下所示：

```java
    //build.gradle

    dependencies {
       ...
    // REMOVE: compile('org.springframework.boot:spring-boot-starter-jdbc')
 compile('org.springframework.boot:spring-boot-starter-data-jpa')       ...
    }
```

请注意我们移除了`spring-boot-starter-jdbc`依赖。`spring-boot-starter-data-jpa`依赖将包含所有必要的依赖项，以便将我们的领域对象与使用 JPA 的嵌入式数据库连接。

# 将 JBCP 日历更新为使用 Spring Data JPA

为了熟悉 Spring Data，我们首先将 JBCP 日历 SQL 转换为使用 ORM，使用 Spring Data JPA 启动器。

创建和维护 SQL 可能相当繁琐。在前几章中，当我们想在数据库中创建一个新的`CalendarUser`表时，我们必须编写大量的样板代码，如下所示：

```java
    //src/main/java/com/packtpub/springsecurity/
    dataaccess/JdbcCalendarUserDao.java

    public int createUser(final CalendarUser userToAdd) {
    if (userToAdd == null) {
         throw new IllegalArgumentException("userToAdd cannot be null");
    }
    if (userToAdd.getId() != null) {
         throw new IllegalArgumentException("userToAdd.getId() must be 
         null when creating a 
         "+CalendarUser.class.getName());
    }
 KeyHoldener keyHolder = new GeratedKeyHolder(); this.jdbcOperations.update(new PreparedStatementCreator() { public PreparedStatement createPreparedStatement
       (Connection connection)
       throws SQLException { PreparedStatement ps = connection.prepareStatement("insert into 
         calendar_users (email, password, first_name, last_name) 
         values (?, ?, ?, ?)", new String[] {  
          "id" });
 ps.setString(1, userToAdd.getEmail()); ps.setString(2, userToAdd.getPassword()); ps.setString(3, userToAdd.getFirstName()); ps.setString(4, userToAdd.getLastName()); return ps; } }, keyHolder);    return keyHolder.getKey().intValue();
    }
```

创建这个对象，技术上我们需要 12 行代码来执行操作。

现在，使用 Spring Data JPA，相同的实现可以减少到以下代码片段：

```java
    //src/main/java/com/packtpub/springsecurity/dataaccess/JpaCalendarUserDao.java

    public int createUser(final CalendarUser userToAdd) {
    if (userToAdd == null) {
         throw new IllegalArgumentException("userToAdd cannot be null");
    }
    if (userToAdd.getId() != null) {
         throw new IllegalArgumentException("userToAdd.getId() 
         must be null when creating a "+CalendarUser.class.getName());
    }
 Set<Role> roles = new HashSet<>(); roles.add(roleRepository.findOne(0)); userToAdd.setRoles(roles); CalendarUser result = repository.save(userToAdd); repository.flush();     return result.getId();
    }
```

现在，使用 JPA 创建这个对象，技术上我们需要五行代码来执行操作。我们现在需要的代码量不到原来执行相同操作的一半。

# 重新配置数据库配置

首先，我们将转换当前的 JBCP 日历项目。让我们先重新配置数据库。

我们可以首先删除 `DataSourceConfig.java` 文件，因为我们将会利用 Spring Boot 对嵌入式 H2 数据库的内置支持。我们还需要删除 `JavaConfig.java` 文件中对 `DataSourceConfig.java` 的引用，因为目前 `@Import` 注解中有对 `JavaConfig.java` 的引用。

# 初始化数据库

现在，我们可以删除 `src/main/resources/database` 目录及其目录下的所有内容。这个目录包含几个 `.sql` 文件，我们将合并并将它们移动到下一步：

现在，我们需要创建一个 `data.sql` 文件，该文件将包含我们的种子数据，如下所示：

```java
    //src/main/resources/data.sql:
```

+   查看以下 SQL 语句，描述了 `user1` 的密码：

```java
        insert into calendar_users(id,username,email,password,
        first_name,last_name) 
        values(0,'user1@example.com','user1@example.com',
        '$2a$04$qr7RWyqOnWWC1nwotUW1nOe1RD5.
        mKJVHK16WZy6v49pymu1WDHmi','User','1');
```

+   查看以下 SQL 语句，描述了 `admin1` 的密码：

```java
        insert into calendar_users(id,username,email,password,
        first_name,last_name) 
        values (1,'admin1@example.com','admin1@example.com',
        '$2a$04$0CF/Gsquxlel3fWq5Ic/ZOGDCaXbMfXYiXsviTNMQofWRXhvJH3IK',
        'Admin','1');
```

+   查看以下 SQL 语句，描述了 `user2` 的密码：

```java
        insert into calendar_users(id,username,email,password,first_name,
        last_name)
        values (2,'user2@example.com','user2@example.com',
        '$2a$04$PiVhNPAxunf0Q4IMbVeNIuH4M4ecySWHihyrclxW..PLArjLbg8CC',
        'User2','2');
```

+   查看以下 SQL 语句，描述用户角色：

```java
        insert into role(id, name) values (0, 'ROLE_USER');
        insert into role(id, name) values (1, 'ROLE_ADMIN');
```

+   在这里，`user1` 有一个角色：

```java
        insert into user_role(user_id,role_id) values (0, 0);
```

+   在这里，`admin1` 有两个角色：

```java
        insert into user_role(user_id,role_id) values (1, 0);
        insert into user_role(user_id,role_id) values (1, 1);
```

+   查看以下 SQL 语句，描述事件：

```java
        insert into events (id,when,summary,description,owner,attendee)
        values (100,'2017-07-03 20:30:00','Birthday Party',
        'This is going to be a great birthday',0,1);
        insert into events (id,when,summary,description,owner,attendee) 
        values (101,'2017-12-23 13:00:00','Conference Call','Call with 
        the client',2,0);
        insert into events (id,when,summary,description,owner,attendee) 
        values (102,'2017-09-14 11:30:00','Vacation',
        'Paragliding in Greece',1,2);
```

现在，我们可以更新应用程序属性，在`src/main/resources/application.yml`文件中定义嵌入式数据库属性，如下所示：

```java
    # Embedded Database
    datasource:
    url: jdbc:h2:mem:dataSource;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
    driverClassName: org.h2.Driver
    username: sa
    password:
    continue-on-error: true
 jpa: database-platform: org.hibernate.dialect.H2Dialect show-sql: true hibernate: ddl-auto: create-drop
```

在此阶段，我们已经移除了旧的数据库配置并添加了新的配置。应用程序在此阶段无法运行，但仍然可以将其视为我们在转换下一步之前的标记点。

您的代码现在应该看起来像 `calendar05.01-calendar`。

# SQL 到 ORM 的重构

从 SQL 转换到 ORM 实现的重构比你想象的要简单。重构的大部分工作涉及删除以 SQL 形式存在的冗余代码。在下一部分，我们将把 SQL 实现重构成 JPA 实现。

为了让 JPA 将我们的领域对象映射到数据库，我们需要对我们的领域对象进行一些映射。

# 使用 JPA 映射领域对象

查看以下步骤，了解如何映射领域对象：

1.  让我们首先映射我们的 `Event.java` 文件，以便所有领域对象都将使用 JPA，如下所示：

```java
//src/main/java/com/packtpub/springsecurity/domain/Event.java

import javax.persistence.*; @Entity @Table(name = "events") public class Event implements Serializable{
 @Id @GeneratedValue(strategy = GenerationType.AUTO) private Integer id;
@NotEmpty(message = "Summary is required")
private String summary;
@NotEmpty(message = "Description is required")
private String description;
@NotNull(message = "When is required")
private Calendar when;
@NotNull(message = "Owner is required")
 @ManyToOne(fetch = FetchType.LAZY) @JoinColumn(name="owner", referencedColumnName="id") private CalendarUser owner;
 @ManyToOne(fetch = FetchType.LAZY) @JoinColumn(name="attendee", referencedColumnName="id") private CalendarUser attendee;
```

1.  我们需要创建一个 `Role.java` 文件，内容如下：

```java
//src/main/java/com/packtpub/springsecurity/domain/Role.java

import javax.persistence.*;
@Entity @Table(name = "role") public class Role implements Serializable {
 @Id @GeneratedValue(strategy = GenerationType.AUTO) private Integer id;
private String name;
 @ManyToMany(fetch = FetchType.EAGER, mappedBy = "roles") private Set<CalendarUser> users;
```

1.  `Role` 对象将用于将权限映射到我们的 `CalendarUser` 表。现在我们已经有一个 `Role.java` 文件，让我们来映射我们的 `CalendarUser.java` 文件：

```java
//src/main/java/com/packtpub/springsecurity/domain/CalendarUser.java

import javax.persistence.*;
import java.io.Serializable;
import java.util.Set;
@Entity @Table(name = "calendar_users") public class CalendarUser implements Serializable {
 @Id @GeneratedValue(strategy = GenerationType.AUTO)   private Integer id;
   private String firstName;
   private String lastName;
   private String email;
   private String password;
 @ManyToMany(fetch = FetchType.EAGER) @JoinTable(name = "user_role", joinColumns = @JoinColumn(name = "user_id"), inverseJoinColumns = @JoinColumn(name = "role_id")) private Set<Role> roles;
```

在此阶段，我们已经用所需的 JPA 注解映射了我们的领域对象，包括 `@Entity` 和 `@Table` 以定义 RDBMS 的位置，以及结构、引用和关联映射注解。

在此阶段，应用程序将无法运行，但这仍然可以被视为我们在继续转换下一步之前的标记点。

您应该从 `chapter05.02-calendar` 的源代码开始。

# Spring Data 仓库

接下来，我们将通过执行以下步骤向 Spring Data 添加所需接口，以将我们所需的 CRUD 操作映射到嵌入式数据库：

1.  我们首先在新的包中添加一个新的接口，该包将是`com.packtpub.springsecurity.repository`。新文件将称为`CalendarUserRepository.java`，如下所示：

```java
        //com/packtpub/springsecurity/repository/CalendarUserRepository.java

        package com.packtpub.springsecurity.repository;
        import com.packtpub.springsecurity.domain.CalendarUser;
        import org.springframework.data.jpa.repository.JpaRepository;

        public interface CalendarUserRepository
               extends JpaRepository<CalendarUser, Integer> {
           CalendarUser findByEmail(String email);
        }
```

这将允许我们对`CalendarUser`对象执行标准的 CRUD 操作，如`find()`、`save()`和`delete()`。

1.  现在我们可以继续在同一存储库包中添加一个新的接口，该包将是`com.packtpub.springsecurity.repository`，新文件将称为`EventRepository.java`：

```java
            //com/packtpub/springsecurity/repository/EventRepository.java

            package com.packtpub.springsecurity.repository;
            import com.packtpub.springsecurity.domain.Event;
            import org.springframework.data.jpa.repository.JpaRepository;

            public interface EventRepository extends JpaRepository<Event, 
            Integer> {}
```

这将允许我们对`Event`对象执行标准的 CRUD 操作，如`find()`、`save()`和`delete()`。

1.  最后，我们将在同一存储库包中添加一个新的接口，该包将是`com.packtpub.springsecurity.repository`，新文件将称为`RoleRepository.java`。这个`CrudRepository`接口将用于管理与给定的`CalendarUser`相关的安全角色的`Role`对象：

```java
            //com/packtpub/springsecurity/repository/

            package com.packtpub.springsecurity.repository;
            import com.packtpub.springsecurity.domain.Event;
            import org.springframework.data.jpa.repository.JpaRepository;

            public interface RoleRepository extends JpaRepository<Role, 
            Integer> {}
```

这将允许我们对`Role`对象执行标准的 CRUD 操作，如`find()`、`save()`和`delete()`。

# 数据访问对象

我们需要将`JdbcEventDao.java`文件重命名为`JpaEventDao.java`，以便我们可以用新的 Spring Data 代码替换 JDBC SQL 代码。让我们来看看以下步骤：

1.  具体来说，我们需要添加新的`EventRepository`接口，并用新的 ORM 存储库替换 SQL 代码，如下所示：

```java
        //com/packtpub/springsecurity/dataaccess/JpaEventDao.java

        package com.packtpub.springsecurity.dataaccess;
        import com.packtpub.springsecurity.domain.CalendarUser;
        import com.packtpub.springsecurity.domain.Event;
 import com.packtpub.springsecurity.repository.EventRepository;        import org.springframework.beans.factory.annotation.Autowired;
        import org.springframework.data.domain.Example;
        import org.springframework.stereotype.Repository;
        import org.springframework.transaction.annotation.Transactional;
        ...
        @Repository
         public class JpaEventDao implements EventDao {
 private EventRepository repository;           @Autowired
 public JpaEventDao(EventRepository repository) { if (repository == null) { throw new IllegalArgumentException("repository 
                    cannot be null"); } this.repository = repository;           }
           @Override
           @Transactional(readOnly = true)
           public Event getEvent(int eventId) {
 return repository.findOne(eventId);           }
           @Override
           public int createEvent(final Event event) {
               ...
               final Calendar when = event.getWhen();
               if(when == null) {
                   throw new IllegalArgumentException("event.getWhen() 
                   cannot be null");
               }
 Event newEvent = repository.save(event);              ...
           }
           @Override
           @Transactional(readOnly = true)
           public List<Event> findForUser(final int userId) {
                Event example = new Event();
 CalendarUser cu = new CalendarUser(); cu.setId(userId); example.setOwner(cu);               return repository.findAll(Example.of(example));
           }
           @Override
           @Transactional(readOnly = true)
           public List<Event> getEvents() {
 return repository.findAll();           }
        }
```

1.  在此阶段，我们需要重构 DAO 类以支持我们创建的新`CrudRepository`接口。让我们从重构`JdbcCalendarUserDao.java`文件开始。首先，我们可以将文件重命名为`JpaCalendarUserDao.java`，以表示此文件使用 JPA，而不是标准的 JDBC：

```java
        //com/packtpub/springsecurity/dataaccess/JpaCalendarUserDao.java

        package com.packtpub.springsecurity.dataaccess;
        ... omitted for brevity ...
        @Repository
        public class JpaCalendarUserDao
               implements CalendarUserDao {
 private CalendarUserRepository userRepository; private RoleRepository roleRepository; @Autowired public JpaCalendarUserDao(CalendarUserRepository repository, RoleRepository roleRepository) { if (repository == null) { throw new IllegalArgumentException("repository 
                   cannot be null"); } if (roleRepository == null) { throw new IllegalArgumentException("roleRepository 
                   cannot be null"); } this. userRepository = repository; this.roleRepository = roleRepository; }           @Override
           @Transactional(readOnly = true)
           public CalendarUser getUser(final int id) {
 return userRepository.findOne(id);           }
           @Override
           @Transactional(readOnly = true)
           public CalendarUser findUserByEmail(final String email) {
               if (email == null) {
                   throw new IllegalArgumentException
                   ("email cannot be null");
               }
               try {
 return userRepository.findByEmail(email);               } catch (EmptyResultDataAccessException notFound) {
                  return null;
               }
           }
           @Override
           @Transactional(readOnly = true)
           public List<CalendarUser> findUsersByEmail(final String email) {
               if (email == null) {
                  throw new IllegalArgumentException("email 
                  cannot be null");
               }
               if ("".equals(email)) {
                   throw new IllegalArgumentException("email 
                   cannot be empty string");
               } return userRepository.findAll();         }
           @Override
           public int createUser(final CalendarUser userToAdd) {
               if (userToAdd == null) {
                   throw new IllegalArgumentException("userToAdd 
                   cannot be null");
               }
               if (userToAdd.getId() != null) {
                   throw new IllegalArgumentException("userToAdd.getId() 
                   must be null when creating a "+
                   CalendarUser.class.getName());
               }
 Set<Role> roles = new HashSet<>(); roles.add(roleRepository.findOne(0)); userToAdd.setRoles(roles); CalendarUser result = userRepository.save(userToAdd); userRepository.flush();              return result.getId();
           }
        }
```

正如您在前面的代码中所看到的，使用 JPA 所需的更新片段要比使用 JDBC 所需的代码少得多。这意味着我们可以专注于业务逻辑，而不必担心管道问题。

1.  接下来，我们继续重构`JdbcEventDao.java`文件。首先，我们可以将文件重命名为`JpaEventDao.java`，以表示此文件使用 JPA，而不是标准的 JDBC，如下所示：

```java
//com/packtpub/springsecurity/dataaccess/JpaEventDao.java

package com.packtpub.springsecurity.dataaccess;
... omitted for brevity ...
@Repository
public class JpaEventDao implements EventDao {
 private EventRepository repository;   @Autowired
 public JpaEventDao(EventRepository repository) { if (repository == null) { throw new IllegalArgumentException("repository 
           cannot be null"); } this.repository = repository; }   @Override
   @Transactional(readOnly = true)
   public Event getEvent(int eventId) {
 return repository.findOne(eventId);   }
   @Override
   public int createEvent(final Event event) {
       if (event == null) {
           throw new IllegalArgumentException("event cannot be null");
      }
       if (event.getId() != null) {
           throw new IllegalArgumentException
           ("event.getId() must be null when creating a new Message");
       }
       final CalendarUser owner = event.getOwner();
        if (owner == null) {
           throw new IllegalArgumentException("event.getOwner() 
           cannot be null");
       }
       final CalendarUser attendee = event.getAttendee();
       if (attendee == null) {
           throw new IllegalArgumentException("attendee.getOwner() 
           cannot be null");
       }
       final Calendar when = event.getWhen();
       if(when == null) {
           throw new IllegalArgumentException
           ("event.getWhen()cannot be null");
       }
 Event newEvent = repository.save(event);       return newEvent.getId();
   }
      @Override
   @Transactional(readOnly = true)
   public List<Event> findForUser(final int userId) {
 Event example = new Event(); CalendarUser cu = new CalendarUser(); cu.setId(userId); example.setOwner(cu); return repository.findAll(Example.of(example));   }
     @Override
   @Transactional(readOnly = true)
   public List<Event> getEvents() {
 return repository.findAll();   }
}
```

在前面的代码中，使用 JPA 存储库的更新片段已加粗，因此现在`Event`和`CalendarUser`对象被映射到我们的底层 RDBMS。

此时应用程序无法工作，但仍然可以认为这是一个标记点，在我们继续转换的下一步之前。

在此阶段，你的源代码应该与`chapter05.03-calendar`相同。

# 应用服务

剩下要做的唯一事情是配置 Spring Security 以使用新的工件。

我们需要编辑`DefaultCalendarService.java`文件，并只删除用于向新创建的`User`对象添加`USER_ROLE`的剩余代码，如下所示：

```java
    //com/packtpub/springsecurity/service/DefaultCalendarService.java

    package com.packtpub.springsecurity.service;
    ... omitted for brevity ...
    @Repository
    public class DefaultCalendarService implements CalendarService {
       @Override
       public int createUser(CalendarUser user) {
           String encodedPassword = passwordEncoder.encode(user.getPassword());
           user.setPassword(encodedPassword);
           int userId = userDao.createUser(user);   
 //jdbcOperations.update("insert into         
           calendar_user_authorities(calendar_user,authority) 
           values (?,?)", userId, //"ROLE_USER");           return userId;
       }
    }
```

# 用户详细信息服务对象

让我们来看看以下步骤，以添加`UserDetailsService`对象：

1.  现在，我们需要添加一个新的`UserDetailsService`对象的实现，我们将使用我们的`CalendarUserRepository`接口再次对用户进行身份验证和授权，使用相同的底层 RDBMS，但使用我们新的 JPA 实现，如下所示：

```java
        //com/packtpub/springsecurity/service/UserDetailsServiceImpl.java

        package com.packtpub.springsecurity.service;
        ... omitted for brevity ...
        @Service
        public class UserDetailsServiceImpl
             implements UserDetailsService {
 @Autowired private CalendarUserRepository userRepository; @Override @Transactional(readOnly = true) public UserDetails loadUserByUsername(final String username)           throws UsernameNotFoundException {            CalendarUser user = userRepository.findByEmail(username);
           Set<GrantedAuthority> grantedAuthorities = new HashSet<>();
 for (Role role : user.getRoles()){ grantedAuthorities.add(new SimpleGrantedAuthority
               (role.getName())); } return new org.springframework.security.core.userdetails.User( user.getEmail(), user.getPassword(), grantedAuthorities); }        }

```

1.  现在，我们需要配置 Spring Security 以使用我们自定义的`UserDetailsService`对象，如下所示：

```java
       //com/packtpub/springsecurity/configuration/SecurityConfig.java

        package com.packtpub.springsecurity.configuration;
        ... omitted for brevity ...
        @Configuration
        @EnableWebSecurity
        public class SecurityConfig extends WebSecurityConfigurerAdapter {\
 @Autowired private UserDetailsService userDetailsService;           @Override
          public void configure(AuthenticationManagerBuilder auth) 
          throws Exception {
          auth
 .userDetailsService(userDetailsService)           .passwordEncoder(passwordEncoder());
           }
 @Bean @Override public UserDetailsService userDetailsService() { return new UserDetailsServiceImpl(); }           ...
        }
```

1.  启动应用程序并尝试登录应用程序。现在任何配置的用户都可以登录并创建新事件。您还可以创建新用户，并能够立即以新用户身份登录。

您的代码现在应该看起来像`calendar05.04-calendar`。

# 从关系型数据库（RDBMS）重构为文档数据库

幸运的是，有了 Spring Data 项目，一旦我们有了 Spring Data 实现，大部分困难的工作已经完成。现在，只需要进行一些实现特定的重构更改。

# 使用 MongoDB 的文档数据库实现

我们现在将着手将我们的 RDBMS 实现（使用 JPA 作为我们的 ORM 提供者）重构为使用 MongoDB 作为底层数据库提供者的文档数据库实现。MongoDB（来自 humongous）是一个免费且开源的跨平台面向文档的数据库程序。它被归类为一个 NoSQL 数据库程序，MongoDB 使用类似 JSON 的文档和模式。MongoDB 由 MongoDB Inc.开发，位于[`github.com/mongodb/mongo`](https://github.com/mongodb/mongo)。

# 更新我们的依赖项

我们已经包含了本章所需的所有依赖项，所以您不需要对`build.gradle`文件进行任何更新。然而，如果您只是将 Spring Data JPA 支持添加到您自己的应用程序中，您需要在`build.gradle`文件中添加`spring-boot-starter-data-jpa`作为依赖项，如下所示：

```java
    //build.gradle
    // JPA / ORM / Hibernate:
    //compile('org.springframework.boot:spring-boot-starter-data-jpa')
    // H2 RDBMS
    //runtime('com.h2database:h2')
    // MongoDB:

 compile('org.springframework.boot:spring-boot-starter-data-mongodb') compile('de.flapdoodle.embed:de.flapdoodle.embed.mongo')
```

请注意，我们已经移除了`spring-boot-starter-jpa`依赖。`spring-boot-starter-data-mongodb`依赖将包含所有需要将我们的领域对象连接到我们的嵌入式 MongoDB 数据库的依赖项，同时使用 Spring 和 MongoDB 注解的混合。

我们还添加了**Flapdoodle**嵌入式 MongoDB 数据库，但这只适用于测试和演示目的。嵌入式 MongoDB 将为单元测试提供一个跨平台的 MongoDB 运行平台。这个嵌入式数据库位于[`github.com/flapdoodle-oss/de.flapdoodle.embed.mongo`](https://github.com/flapdoodle-oss/de.flapdoodle.embed.mongo)。

# 在 MongoDB 中重新配置数据库配置

首先，我们将开始转换当前的 JBCP 日历项目。让我们先重新配置数据库以使用 Flapdoodle 嵌入式 MongoDB 数据库。之前，当我们更新这个项目的依赖时，我们添加了一个 Flapdoodle 依赖项，该项目得到了一个嵌入式 MongoDB 数据库，我们可以自动使用它，而不是安装 MongoDB 的完整版本。为了与 JBCP 应用程序保持一致，我们需要更改我们数据库的名称。使用 Spring Data，我们可以使用 YAML 配置来更改 MongoDB 配置，如下所示：

```java
    //src/main/resources/application.yml

    spring
    # MongoDB
 data: mongodb:         host: localhost
 database: dataSource
```

对于我们当前需求最重要的配置是更改数据库名称为`dataSource`，这个名称与本书中我们一直在使用的名称相同。

# 初始化 MongoDB 数据库

使用 JPA 实现时，我们使用了`data.sql`文件来初始化数据库中的数据。对于 MongoDB 实现，我们可以删除`data.sql`文件，并用我们称之为`MongoDataInitializer.java`的 Java 配置文件来替代它：

```java
    //src/main/java/com/packtpub/springsecurity/configuration/
    MongoDataInitializer.java

    ¦
    @Configuration
    public class MongoDataInitializer {
       @Autowired
       private RoleRepository roleRepository;
       @Autowired
       private CalendarUserRepository calendarUserRepository;
       @Autowired
       private EventRepository eventRepository;
 @PostConstruct       public void setUp() {
 calendarUserRepository.deleteAll(); roleRepository.deleteAll(); eventRepository.deleteAll(); seedRoles(); seedCalendarUsers(); seedEvents();       }
       CalendarUser user1, admin, user2;
       {
 user1 = new CalendarUser(0, "user1@example.com",
           "$2a$04$qr7RWyqOnWWC1nwotUW1nOe1RD5.mKJVHK16WZy6v49pymu1WDHmi",
           "User","1"); admin = new   CalendarUser(1,"admin1@example.com",
           "$2a$04$0CF/Gsquxlel3fWq5Ic/ZOGDCaXbMfXYiXsviTNMQofWRXhvJH3IK",
           "Admin","1"); user2 = new CalendarUser(2,"user2@example.com",
           "$2a$04$PiVhNPAxunf0Q4IMbVeNIuH4M4ecySWHihyrclxW..PLArjLbg8CC",
           "User2","2");       }
       Role user_role, admin_role;
       private void seedRoles(){
           user_role = new Role(0, "ROLE_USER");
           admin_role = new Role(1, "ROLE_ADMIN");
           user_role = roleRepository.save(user_role);
           admin_role = roleRepository.save(admin_role);
       }
       private void seedEvents(){
 // Event 1           Event event1 = new Event(100, "Birthday Party", "This is 
           going to be a great birthday", new 
           GregorianCalendar(2017,6,3,6,36,00), user, admin);
 // Event 2           Event event2 = new Event(101, "Conference Call", 
           "Call with the client",new 
           GregorianCalendar(2017,11,23,13,00,00),user2, user);
 // Event 3           Event event3 = new Event(102, "Vacation",
           "Paragliding in Greece",new GregorianCalendar(2017,8,14,11,30,00),
           admin, user2);
           // Save Events
 eventRepository.save(event1); eventRepository.save(event2); eventRepository.save(event3);       }
       private void seedCalendarUsers(){
           // user1
           user1.addRole(user_role);
          // admin2
           admin.addRole(user_role);
           admin.addRole(admin_role);
           // user2
           user2.addRole(user_role);
 calendarUserRepository.save(user1); calendarUserRepository.save(admin); calendarUserRepository.save(user2);       }
    }
```

这将在加载时执行，并将将相同的数据种子到我们的 MongoDB 中，就像我们使用我们的 H2 数据库一样。

# 使用 MongoDB 映射领域对象

首先，让我们将我们的`Event.java`文件映射到领域对象，以便将每个领域对象保存为我们的 MongoDB 数据库中的文档。这可以通过执行以下步骤来实现：

1.  在文档数据库中，领域对象映射有所不同，但相同的 ORM 概念仍然适用。让我们从 Event JPA 实现开始，然后看看如何将我们的`Entity`转换为文档映射：

```java
        //src/main/java/com/packtpub/springsecurity/domain/Event.java

         ...
 import javax.persistence.*; @Entity @Table(name = "events")        public class Event implements Serializable{
 @Id @GeneratedValue(strategy = GenerationType.AUTO)           private Integer id;
           private String summary;
           private String description;
           private Calendar when;
 @ManyToOne(fetch = FetchType.LAZY) @JoinColumn(name="owner", referencedColumnName="id")           private CalendarUser owner;
 @ManyToOne(fetch = FetchType.LAZY) @JoinColumn(name="attendee", referencedColumnName="id")           private CalendarUser attendee;
           ¦
```

1.  在基于实体的 JPA 映射中，我们需要使用六个不同的注解来创建所需的映射。现在，在基于文档的 MongoDB 映射中，我们需要更改所有的先前映射注解。下面是我们完全重构的`Event.java`文件的示例：

```java
        //src/main/java/com/packtpub/springsecurity/domain/Event.java

 import org.springframework.data.annotation.Id; import org.springframework.data.annotation.PersistenceConstructor; import org.springframework.data.domain.Persistable; import org.springframework.data.mongodb.core.mapping.DBRef; import org.springframework.data.mongodb.core.mapping.Document;        ...
 @Document(collection="events")        public class Event implements Persistable<Integer>, Serializable{
 @Id             private Integer id;
             private String summary;
             private String description;
             private Calendar when;
 @DBRef           private CalendarUser owner;
 @DBRef           private CalendarUser attendee;
 @PersistenceConstructor           public Event(Integer id,
                 String summary,
                 String description,
                 Calendar when,
                 CalendarUser owner,
                 CalendarUser attendee) {
                  ...
          }
```

在上述代码中，我们可以看到一些值得注意的更改：

1.  首先，我们需要声明类为`@o.s.d.mongodb.core.mapping.Document`类型，并为这些文档提供集合名称。

1.  接下来，`Event`类必须实现`o.s.d.domain.Persistable`接口，为我们的文档提供主键类型（`Integer`）。

1.  现在，我们将我们的领域 ID 注解更改为`@o.s.d.annotation.Id`，以定义领域主键。

1.  之前，我们必须将我们的所有者和参与者`CalendarUser`对象映射到两个不同的映射注解。

1.  现在，我们只需要定义两种类型为`@o.s.d.mongodb.core.mapping.DBRef`，并允许 Spring Data 处理底层引用。

1.  我们必须添加的最后一个注解定义了一个特定的构造函数，用于将新文档添加到我们的文档中，通过使用`@o.s.d.annotation.PersistenceConstructor`注解。

1.  现在我们已经回顾了从 JPA 转换到 MongoDB 所需的更改，让我们从`Role.java`文件开始转换另一个领域对象：

```java
        //src/main/java/com/packtpub/springsecurity/domain/Role.java

        ...
        import org.springframework.data.annotation.Id;
        import org.springframework.data.annotation.PersistenceConstructor;
        import org.springframework.data.domain.Persistable;
        import org.springframework.data.mongodb.core.mapping.Document;
 @Document(collection="role")        public class Role implements Persistable<Integer>, Serializable {
 @Id            private Integer id;
            private String name;
            public Role(){}
 @PersistenceConstructor        public Role(Integer id, String name) {
            this.id = id;
            this.name = name;
         }
```

1.  我们需要重构的最后一个领域对象是我们的`CalendarUser.java`文件。毕竟，这是这个应用程序中最复杂的领域对象：

```java
        //src/main/java/com/packtpub/springsecurity/domain/CalendarUser.java

        ...
        import org.springframework.data.annotation.Id;
        import org.springframework.data.annotation.PersistenceConstructor;
        import org.springframework.data.domain.Persistable;
        import org.springframework.data.mongodb.core.mapping.DBRef;
        import org.springframework.data.mongodb.core.mapping.Document;
 @Document(collection="calendar_users")        public class CalendarUser implements Persistable<Integer>, 
        Serializable {
 @Id           private Integer id;
           private String firstName;
           private String lastName;
           private String email;
           private String password;
 @DBRef(lazy = false)          private Set<Role> roles = new HashSet<>(5);
          public CalendarUser() {}
 @PersistenceConstructor          public CalendarUser(Integer id,String email, String password,
          String firstName,String lastName) {
             this.id = id;
             this.firstName = firstName;
             this.lastName = lastName;
             this.email = email;
             this.password = password;
           }
```

正如你所见，将我们的领域对象从 JPA 重构为 MongoDB 的努力相当简单，并且比 JPA 配置需要的注解配置要少。

# Spring Data 对 MongoDB 的仓库

现在我们只需要对从 JPA 实现到 MongoDB 实现进行少量更改即可重构。我们将从重构我们的`CalendarUserRepository.java`文件开始，通过更改我们仓库所扩展的接口，如下所示：

```java
    //com/packtpub/springsecurity/repository/CalendarUserRepository.java

    ...
 import org.springframework.data.mongodb.repository.MongoRepository;    public interface CalendarUserRepository extends MongoRepository
    <CalendarUser, Integer> {
       ...
```

这个相同的更改需要应用到`EventRepository.java`文件和`RoleRepository.java`文件上。

如果你需要帮助进行这些更改，请记住`chapter05.05`的源代码将有完整的代码供您参考。

# MongoDB 中的数据访问对象

在我们的`EventDao`接口中，我们需要创建一个新的`Event`对象。使用 JPA，我们的对象 ID 可以自动生成。使用 MongoDB，有几种方式可以分配主键标识符，但为了这个演示，我们只需使用原子计数器，如下所示：

```java
    //src/main/java/com/packtpub/springsecurity/dataaccess/MongoEventDao.java

    ...
 import java.util.concurrent.atomic.AtomicInteger;    @Repository
    public class MongoEventDao implements EventDao {
      // Simple Primary Key Generator
 private AtomicInteger eventPK = new AtomicInteger(102);       ...
       @Override
       public int createEvent(Event event) {
           ...
           // Get the next PK instance
 event.setId(eventPK.incrementAndGet()); Event newEvent = repository.save(event);           return newEvent.getId();
       }
       ...
```

从技术上讲，我们的`CalendarUserDao`对象没有变化，但为了本书的一致性，我们将实现文件的名称更改为表示使用`Mongo`：

```java
    @Repository
    public class MongoCalendarUserDao implements CalendarUserDao {
```

对于这个重构示例，没有其他**数据访问对象**（**DAO**）的更改需求。

启动应用程序，它将像以前一样运行。尝试以`user1`和`admin1`的身份登录，并测试以确保两个用户都可以向系统添加新事件，以确保整个应用程序的映射正确。

你应该从`chapter05.05-calendar`的源代码开始。

# 总结

我们已经探讨了 Spring Data 项目的强大和灵活性，以及与应用程序开发相关的几个方面，还包括了与 Spring Security 的集成。在本章中，我们覆盖了 Spring Data 项目及其部分功能。我们还看到了从使用 SQL 的遗留 JDBC 代码到使用 JPA 的 ORM，以及从使用 Spring Data 的 JPA 实现到使用 Spring Data 的 MongoDB 实现的重构过程。我们还覆盖了配置 Spring Security 以利用关系数据库中的 ORM `Entity`和文档数据库中的配置。

在下一章中，我们将探讨 Spring Security 对基于 LDAP 的认证的内置支持。
