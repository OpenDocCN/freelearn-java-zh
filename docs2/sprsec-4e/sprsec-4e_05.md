

# 使用 Spring Data 进行身份验证

在上一章中，我们介绍了如何利用 Spring Security 的内置**Java 数据库连接**（**JDBC**）支持。在本章中，我们将探讨 Spring Data 项目以及如何利用**Java 持久性 API**（**JPA**）对关系型数据库进行身份验证。我们还将探讨如何使用**MongoDB**对文档数据库进行身份验证。本章的示例代码基于第四章的 Spring Security 设置，*基于 JDBC 的身份验证*（[B21757_04.xhtml#_idTextAnchor106]），并且已经更新以重构对 SQL 的需求，并使用 ORM 进行所有数据库交互。

在本章的讨论过程中，我们将涵盖以下主题：

+   与 Spring Data 项目相关的一些基本概念

+   利用 Spring Data JPA 对关系型数据库进行身份验证

+   利用 Spring Data MongoDB 对文档数据库进行身份验证

+   如何自定义 Spring Security 以在处理 Spring Data 集成时获得更多灵活性

+   理解 Spring Data 项目

Spring Data 项目的目标是提供一个熟悉且一致的基于 Spring 的数据访问编程模型，同时仍然保留底层数据提供者的特殊特性。

以下是这个 Spring Data 项目的一些强大功能：

+   强大的存储库和自定义对象映射抽象

+   从存储库方法名称动态推导查询

+   实现领域基类，提供基本属性

+   支持透明审计（创建和最后更改）

+   能够集成自定义存储库代码

+   通过基于 Java 的配置和自定义 XML 命名空间轻松实现 Spring 集成

+   与 Spring MVC 控制器的高级集成

+   对跨存储持久化的实验性支持

该项目简化了数据访问技术的使用，包括关系型和非关系型数据库、`MapReduce`框架和基于云的数据服务。这个母项目包含许多特定于给定数据库的子项目。这些项目是由与许多支持这些令人兴奋技术的公司和开发者合作开发的。还有许多由社区维护的模块和其他相关模块，包括*JDBC 支持*和*Apache Hadoop*。

以下表格描述了构成 Spring Data 项目的核心模块：

| **模块** | **描述** |
| --- | --- |
| Spring Data Commons | 将核心 Spring 概念应用于所有 Spring Data 项目 |
| Spring Data Gemfire | 从 Spring 应用程序中提供简单的配置和访问 Gemfire |
| Spring Data JPA | 使实现基于 JPA 的存储库变得容易 |
| Spring Data Key Value | 基于映射的存储库和 SPI，可以轻松构建用于键值存储的 Spring Data 模块 |
| Spring Data LDAP | 为 Spring LDAP 提供 Spring Data 存储库支持 |
| Spring Data MongoDB | 基于 Spring 的对象-文档支持和 MongoDB 的仓库 |
| Spring Data REST | 将 Spring Data 仓库作为超媒体驱动的 RESTful 资源导出 |
| Spring Data Redis | 为 Spring 应用程序提供简单的配置和访问 Redis |
| Spring Data for Apache Cassandra | Apache Cassandra 的 Spring Data 模块 |
| Spring Data for Apache Solr | Apache Solr 的 Spring Data 模块 |

表 5.1 – Spring Data 项目的核心模块

在探索 Spring Data 项目的核心模块之后，现在让我们深入了解 Spring Data JPA 的主要功能。

本章的代码示例链接在此：[`packt.link/omOQK`](https://packt.link/omOQK)。

# Spring Data JPA

Spring Data JPA 项目旨在通过减少实际所需的工作量来显著提高数据访问层的 ORM 实现。开发者只需编写仓库接口，包括自定义查找方法，Spring 将自动提供实现。

以下只是 Spring Data JPA 项目的一些特定于项目的强大功能：

+   基于 Spring 和 JPA 构建仓库的复杂支持

+   支持`QueryDSL`谓词，从而实现类型安全的 JPA 查询

+   领域类的透明审计

+   分页支持、动态查询执行以及集成自定义数据访问代码的能力

+   在启动时验证`@Query`注解的查询

+   支持基于 XML 的实体映射

+   通过引入`@EnableJpaRepositories`实现基于`JavaConfig`的仓库配置

## 更新我们的依赖项

我们已经包含了本章所需的全部依赖项，因此您不需要更新您的`build.gradle`文件。但是，如果您只是将 Spring Data JPA 支持添加到您的应用程序中，您需要在`build.gradle`文件中添加`spring-boot-starter-data-jpa`作为依赖项，如下所示：

```java
//build.gradle
dependencies {
    // JPA / ORM / Hibernate:
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
...
}
```

注意，我们没有移除`spring-boot-starter-jdbc`依赖项。

`spring-boot-starter-data-jpa`依赖将包含将我们的领域对象与嵌入式数据库连接所需的全部依赖项。

## 重新配置数据库配置

首先，我们将转换当前的 JBCP 日历项目。让我们从重新配置数据库开始。

我们可以先删除`DataSourceConfig.java`文件，因为我们将会利用 Spring Boot 内置对嵌入式 H2 数据库的支持。

## 初始化数据库

我们现在可以删除`src/main/resources/database`目录及其中的所有内容。此目录包含多个`.sql`文件。

现在，我们需要创建一个包含我们的种子数据的`data.sql`文件，如下所示：

+   查看以下 SQL 语句，展示了**user1**的密码：

    ```java
    //src/main/resources/data.sql
    insert into calendar_users(id,email,password,first_name,last_name) values (0, 'user1@example.com','$2a$04$qr7RWyqOnWWC1nwotUW1nOe1RD5.mKJVHK16WZy6v49pymu1WDHmi','User','1');
    ```

+   查看以下 SQL 语句，展示了**admin1**的密码：

    ```java
    insert into calendar_users(id,email,password,first_name,last_name) values (1,'admin1@example.com','$2a$04$0CF/Gsquxlel3fWq5Ic/ZOGDCaXbMfXYiXsviTNMQofWRXhvJH3IK','Admin','1');
    ```

+   查看以下 SQL 语句，展示了**user2**的密码：

    ```java
    insert into calendar_users(id,email,password,first_name,last_name) values (2,'user2@example.com','$2a$04$PiVhNPAxunf0Q4IMbVeNIuH4M4ecySWHihyrclxW..PLArjLbg8CC','User2','2');
    ```

+   看一下以下 SQL 语句，描述了用户角色：

    ```java
    insert into role(id, name) values (0, 'ROLE_USER');
    insert into role(id, name) values (1, 'ROLE_ADMIN');
    ```

+   在这里，**user1** 拥有一个角色：

    ```java
    insert into user_role(user_id,role_id) values (0, 0);
    ```

+   在这里，**admin1** 拥有两个角色：

    ```java
    insert into user_role(user_id,role_id) values (1, 0);
    insert into user_role(user_id,role_id) values (1, 1);
    ```

+   看一下以下 SQL 语句，描述了事件：

    ```java
    insert into events (id,date_when,summary,description,owner,attendee) values (100,'2023-07-03 20:30:00','Birthday Party','This is going to be a great birthday',0,1);
    insert into events (id,date_when,summary,description,owner,attendee) values (101,'2023-12-23 13:00:00','Conference Call','Call with the client',2,0);
    insert into events (id,date_when,summary,description,owner,attendee) values (102,'2023-09-14 11:30:00','Vacation','Paragliding in Greece',1,2);
    ```

现在，我们可以更新应用程序属性，在 `src/main/resources/application.yml` 文件中定义我们的嵌入式数据库属性，如下所示：

```java
datasource:
  url: jdbc:h2:mem:dataSource;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
  driverClassName: org.h2.Driver
  username: sa
  password:
jpa:
  database-platform: org.hibernate.dialect.H2Dialect
  show-sql: true
  hibernate:
    ddl-auto: create-drop
```

到目前为止，我们已经移除了旧的数据库配置并添加了新的配置。此时应用程序将无法工作，但仍然可以将其视为转换下一步之前的一个标记点。

重要提示

你的代码现在应该看起来像这样：`calendar05.01-calendar`。

# 从 SQL 转换到 ORM

从 SQL 转换到 ORM 实现比你想象的要简单。大部分的转换涉及移除以 SQL 形式存在的多余代码。在接下来的这一节中，我们将把我们的 SQL 实现转换为 JPA 实现。

为了让 JPA 将我们的领域对象映射到我们的数据库，我们需要在我们的领域对象上执行一些映射。

## 使用 JPA 映射领域对象

看一下以下步骤来了解如何映射领域对象：

1.  让我们先映射我们的 `Event.java` 文件，以便所有领域对象都将使用 JPA，如下所示：

    ```java
    //src/main/java/com/packtpub/springsecurity/domain/Event.java
    @Entity
    @Table(name = "events")
    public class Event implements Serializable{
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Integer id;
        @NotEmpty(message = "Summary is required")
        private String summary;
        @NotEmpty(message = "Description is required")
        private String description;
        @NotNull(message = "When is required")
        private Calendar dateWhen;
        @NotNull(message = "Owner is required")
        @ManyToOne(fetch = FetchType.LAZY)
        @JoinColumn(name="owner", referencedColumnName="id")
        private CalendarUser owner;
        @ManyToOne(fetch = FetchType.LAZY)
        @JoinColumn(name="attendee", referencedColumnName="id")
        private CalendarUser attendee;
    ...
    }
    ```

1.  我们需要创建一个包含以下内容的 `Role.java` 文件：

    ```java
    //src/main/java/com/packtpub/springsecurity/domain/Role.java
    @Entity
    @Table(name = "role")
    public class Role implements Serializable {
        @Id
        @GeneratedValue(strategy = GenerationType.AUTO)
        private Integer id;
        private String name;
        @ManyToMany(fetch = FetchType.EAGER, mappedBy = "roles")
        private Set<CalendarUser> users;
    ...
    }
    ```

1.  `Role` 对象将被用来将权限映射到我们的 `CalendarUser` 表。现在我们已经有了 `Role.java` 文件，让我们映射我们的 `CalendarUser.java` 文件：

    ```java
    //src/main/java/com/packtpub/springsecurity/domain/CalendarUser.java
    @Entity
    @Table(name = "calendar_users")
    public class CalendarUser implements Principal, Serializable {
        private static final long serialVersionUID = 8433999509932007961L;
        @Id
        @SequenceGenerator(name = "user_id_seq", initialValue = 1000)
        @GeneratedValue(generator = "user_id_seq")
        private Integer id;
        private String firstName;
        private String lastName;
        private String email;
        private String password;
        @ManyToMany(fetch = FetchType.EAGER)
        @JoinTable(name = "user_role",
              joinColumns = @JoinColumn(name = "user_id"),
              inverseJoinColumns = @JoinColumn(name = "role_id"))
        private Set<Role> roles;
    ...
    }
    ```

到目前为止，我们已经使用所需的 JPA 注解映射了我们的领域对象，包括 `@Entity` 和 `@Table` 来定义 **关系型数据库管理系统**（**RDBMS**）的位置，以及结构、引用和关联映射注解。

在这个阶段，你也可以**删除**以下依赖项：

```java
//build.gradle
dependencies {
...
implementation 'org.springframework.boot:spring-boot-starter-data-jdbc'
...
}
```

此时应用程序将无法工作，但仍然可以将其视为转换下一步之前的一个标记点。

重要提示

你的代码现在应该看起来像这样：`calendar05.02-calendar`。

## Spring Data 存储库

我们现在将添加所需的接口，以便 Spring Data 将所需的 **创建、读取、更新和删除**（**CRUD**）操作映射到我们的嵌入式数据库，通过执行以下步骤：

1.  我们首先向一个新包中添加一个新的接口，该包将是 `com.packtpub.springsecurity.repository`。新文件将被命名为 `CalendarUserRepository.java`，如下所示：

    ```java
    //com/packtpub/springsecurity/repository/CalendarUserRepository.java
    public interface CalendarUserRepository extends JpaRepository<CalendarUser, Integer> {
        CalendarUser findByEmail(String email);
    }
    ```

1.  我们现在可以继续添加一个新的接口到同一个存储库包中，该包将是 `com.packtpub.springsecurity.repository`，新文件将被命名为 `EventRepository.java`：

    ```java
    //com/packtpub/springsecurity/repository/EventRepository.java
    public interface EventRepository extends JpaRepository<Event, Integer> {
    }
    ```

    这将允许对 `Event` 对象执行标准的 CRUD 操作，如 `find()`、`save()` 和 `delete()`。

1.  最后，我们将向同一个仓库包添加一个新的接口，该接口将是`com.packtpub.springsecurity.repository`，新文件将命名为`RoleRepository.java`。这个`CrudRepository`接口将用于管理与给定的`CalendarUser`关联的安全角色中的`Role`对象：

```java
//com/packtpub/springsecurity/repository/RoleRepository.java
public interface RoleRepository extends JpaRepository<Role, Integer> {
}
```

这将允许我们对`Role`对象执行标准 CRUD 操作，如`find()`、`save()`和`delete()`。

## 数据访问对象

我们需要将`JdbcEventDao.java`文件重命名为`JpaEventDao.java`，这样我们就可以用新的 Spring Data 代码替换 JDBC SQL 代码。让我们看看以下步骤：

1.  具体来说，我们需要添加新的`EventRepository`接口，并用新的 ORM 仓库替换 SQL 代码，如下所示：

    ```java
    //com/packtpub/springsecurity/dataaccess/JpaEventDao.java
    @Repository
    public class JpaEventDao implements EventDao {
        // --- members ---
        private EventRepository repository;
        // --- constructors ---
        public JpaEventDao(EventRepository repository) {
            if (repository == null) {
                throw new IllegalArgumentException("repository cannot be null");
            }
            this.repository = repository;
        }
        // --- EventService ---
        @Override
        @Transactional(readOnly = true)
        public Event getEvent(int eventId) {
            return repository.findById(eventId).orElse(null);
        }
        @Override
        public int createEvent(final Event event) {
            if (event == null) {
                throw new IllegalArgumentException("event cannot be null");
            }
            if (event.getId() != null) {
                throw new IllegalArgumentException("event.getId() must be null when creating a new Message");
            }
            final CalendarUser owner = event.getOwner();
            if (owner == null) {
                throw new IllegalArgumentException("event.getOwner() cannot be null");
            }
            final CalendarUser attendee = event.getAttendee();
            if (attendee == null) {
                throw new IllegalArgumentException("attendee.getOwner() cannot be null");
            }
            final Calendar when = event.getDateWhen();
            if(when == null) {
                throw new IllegalArgumentException("event.getWhen() cannot be null");
            }
            Event newEvent = repository.save(event);
            return newEvent.getId();
        }
        @Override
        @Transactional(readOnly = true)
        public List<Event> findForUser(final int userId) {
            Event example = new Event();
            CalendarUser cu = new CalendarUser();
            cu.setId(userId);
            example.setOwner(cu);
            return repository.findAll(Example.of(example));
        }
        @Override
        @Transactional(readOnly = true)
        public List<Event> getEvents() {
            return repository.findAll();
        }
    }
    ```

1.  在这一点上，我们需要重构 DAO 类以支持我们创建的新`CrudRepository`接口。让我们从重构`JdbcCalendarUserDao.java`文件开始。首先，我们可以将文件重命名为`JpaCalendarUserDao.java`，以表明这个文件使用的是 JPA 而不是标准的 JDBC：

    ```java
    //com/packtpub/springsecurity/dataaccess/JpaCalendarUserDao.java
    @Repository
    public class JpaCalendarUserDao implements CalendarUserDao {
        private static final Logger logger = LoggerFactory
                .getLogger(JpaCalendarUserDao.class);
        // --- members ---
        private CalendarUserRepository userRepository;
        private RoleRepository roleRepository;
        // --- constructors ---
        public JpaCalendarUserDao(final CalendarUserRepository repository,
                                  final RoleRepository roleRepository) {
            if (repository == null) {
                throw new IllegalArgumentException("repository cannot be null");
            }
            if (roleRepository == null) {
                throw new IllegalArgumentException("roleRepository cannot be null");
            }
            this.userRepository = repository;
            this.roleRepository = roleRepository;
        }
        // --- CalendarUserDao methods ---
        @Override
        @Transactional(readOnly = true)
        public CalendarUser getUser(final int id) {
            return userRepository.findById(id).orElse(null);
        }
        @Override
        @Transactional(readOnly = true)
        public CalendarUser findUserByEmail(final String email) {
            if (email == null) {
                throw new IllegalArgumentException("email cannot be null");
            }
            try {
                return userRepository.findByEmail(email);
            } catch (EmptyResultDataAccessException notFound) {
                return null;
            }
        }
        @Override
        @Transactional(readOnly = true)
        public List<CalendarUser> findUsersByEmail(final String email) {
            if (email == null) {
                throw new IllegalArgumentException("email cannot be null");
            }
            if ("".equals(email)) {
                throw new IllegalArgumentException("email cannot be empty string");
            }
            return userRepository.findAll();
        }
        @Override
        public int createUser(final CalendarUser userToAdd) {
            if (userToAdd == null) {
                throw new IllegalArgumentException("userToAdd cannot be null");
            }
            if (userToAdd.getId() != null) {
                throw new IllegalArgumentException("userToAdd.getId() must be null when creating a "+CalendarUser.class.getName());
            }
            Set<Role> roles = new HashSet<>();
            roles.add(roleRepository.findById(0).orElse(null));
            userToAdd.setRoles(roles);
            CalendarUser result = userRepository.save(userToAdd);
            userRepository.flush();
            return result.getId();
        }
    }
    ```

    在前面的代码中，用于利用 JPA 仓库的更新片段已被加粗，因此现在`Event`和`CalendarUser`对象被映射到我们的底层 RDBMS。

在这一点上，应用程序可能无法正常工作，但这仍然可以被视为在继续转换的下一步之前的一个标记点。

重要注意事项

在这一点上，您的源代码应该看起来与`chapter05.03- calendar`相同。

# 应用程序服务

剩下的唯一事情就是配置 Spring Security 以使用新的工件。

我们需要编辑`DefaultCalendarService.java`文件，并仅删除用于将`USER_ROLE`添加到任何新创建的`User`对象的剩余代码，如下所示：

```java
//com/packtpub/springsecurity/service/DefaultCalendarService.java
@Repository
public class DefaultCalendarService implements CalendarService {
... omitted for brevity ...
  public int createUser(CalendarUser user) {
    String encodedPassword = passwordEncoder.encode(user.getPassword());
    user.setPassword(encodedPassword);
    int userId = userDao.createUser(user);
    return userId;
  }
}
```

# UserDetailsService 对象

让我们看看以下步骤来添加`UserDetailsService`对象：

1.  现在，我们需要添加`UserDetailsService`对象的新实现；我们将使用我们的`CalendarUserRepository`接口再次对用户进行身份验证和授权，使用相同的底层 RDBMS，但使用我们的新 JPA 实现，如下所示：

    ```java
    //com/packtpub/springsecurity/service/ CalendarUserDetailsService.java
    @Component
    public class CalendarUserDetailsService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        CalendarUser user = calendarUserDao.findUserByEmail(username);
        if (user == null) {
            throw new UsernameNotFoundException("Invalid username/password.");
        }
        return new CalendarUserDetails(user);
     }
    }
    ```

1.  现在，我们必须配置 Spring Security 以使用我们的自定义`UserDetailsService`对象，如下所示：

    ```java
    //com/packtpub/springsecurity/configuration/SecurityConfig.java
    @Configuration
    @EnableWebSecurity
    public class SecurityConfig {
    ... omitted for brevity ...
        @Bean
        public AuthenticationManager authManager(HttpSecurity http) throws Exception {
            AuthenticationManagerBuilder authenticationManagerBuilder =
                http.getSharedObject(AuthenticationManagerBuilder.class);
            return authenticationManagerBuilder.build();
        }
      }
    ...
    }
    ```

1.  启动应用程序并尝试登录应用程序。现在，任何配置的用户都可以登录并创建新事件。您还可以创建一个新用户，并可以立即以这个新用户登录。

重要注意事项

您的代码现在应该看起来像`calendar05.04-calendar`。

## 从关系型数据库管理系统（RDBMS）重构到文档数据库

幸运的是，随着 Spring Data 项目的出现，一旦我们有了 Spring Data 实现，大部分困难的工作就已经完成了。现在，只需要对几个特定实现进行重构。

# 使用 MongoDB 的文档数据库实现

现在，我们将着手重构我们的 RDBMS 实现——使用 JPA 作为 ORM 提供者——到文档数据库实现，使用 MongoDB 作为底层数据库提供者。MongoDB 是一个免费的开源跨平台文档导向数据库程序。作为一种 NoSQL 数据库程序，MongoDB 使用具有模式的类似 JSON 的文档。MongoDB 由 MongoDB Inc. 开发，位于 [`github.com/mongodb/mongo`](https://github.com/mongodb/mongo)。

## 更新我们的依赖项

我们已经包含了本章所需的全部依赖项，因此你不需要更新你的 `build.gradle` 文件。然而，如果你只是将 Spring Data JPA 支持添加到自己的应用程序中，你需要在 `build.gradle` 文件中添加 `spring-boot-starter-data-jpa` 作为依赖项，如下所示：

```java
//build.gradle
dependencies {
// MondgoDB
implementation 'org.springframework.boot:spring-boot-starter-data-mongodb'
implementation 'de.flapdoodle.embed:de.flapdoodle.embed.mongo.spring30x:4.9.2'
}
```

注意，我们已经移除了 `spring-boot-starter-jpa` 依赖。`spring-boot-starter-data-mongodb` 依赖将包含将我们的领域对象连接到嵌入式 MongoDB 数据库所需的所有依赖项，这些依赖项结合了 Spring 和 MongoDB 注解。

我们还添加了 `Flapdoodle` 嵌入式 MongoDB 数据库，但这仅用于测试和演示目的。嵌入式 MongoDB 将提供一种平台无关的方式来在单元测试中运行 MongoDB。这个嵌入式数据库位于 [`github.com/flapdoodle-oss/de.flapdoodle.embed.mongo`](https://github.com/flapdoodle-oss/de.flapdoodle.embed.mongo)。

## 重新配置 MongoDB 数据库配置

首先，我们将开始将当前的 JBCP 日历项目转换为 JPA 实现。让我们首先重新配置数据库以使用 Flapdoodle-嵌入式 MongoDB 数据库。之前，当我们更新这个项目的依赖项时，我们添加了一个 Flapdoodle 依赖项，为项目提供了一个嵌入式 MongoDB 数据库，我们可以自动使用它而不是安装完整的 MongoDB 版本。为了与 JBCP 应用程序保持一致，我们需要更改我们数据库的名称。使用 Spring Data，我们可以通过以下方式使用 YAML 配置更改 MongoDB 配置：

```java
//src/main/resources/application.yml
spring:
  ## Thymeleaf configuration:
  thymeleaf:
    cache: false
    mode: HTML
  # MongoDB
  data:
    mongodb:
      host: localhost
      database: dataSource
de:
  flapdoodle:
    mongodb:
      embedded:
        version: 7.0.0
```

对于我们当前的需求，最重要的配置是将数据库名称更改为 `dataSource`，这是我们在这本书中一直使用的名称。

## 初始化 MongoDB 数据库

使用 JPA 实现，我们使用了 `data.sql` 文件来初始化数据库中的数据。对于 MongoDB 实现，我们可以移除 `data.sql` 文件，并用一个 Java 配置文件替换它，我们将称之为 `MongoDataInitializer.java`：

```java
//src/main/java/com/packtpub/springsecurity/configuration/ MongoDataInitializer.java
@Configuration
public class MongoDataInitializer {
    private static final Logger logger = LoggerFactory
            .getLogger(MongoDataInitializer.class);
    private RoleRepository roleRepository;
    private CalendarUserRepository calendarUserRepository;
    private EventRepository eventRepository;
    public MongoDataInitializer(RoleRepository roleRepository, CalendarUserRepository calendarUserRepository, EventRepository eventRepository) {
       this.roleRepository = roleRepository;
       this.calendarUserRepository = calendarUserRepository;
       this.eventRepository = eventRepository;
    }
    @PostConstruct
    public void setUp() {
    }
    CalendarUser user, admin, user2;
    // CalendarUsers
    {
        user = new CalendarUser(0, "user1@example.com","$2a$04$qr7RWyqOnWWC1nwotUW1nOe1RD5.mKJVHK16WZy6v49pymu1WDHmi","User","1");
        admin = new CalendarUser(1,"admin1@example.com","$2a$04$0CF/Gsquxlel3fWq5Ic/ZOGDCaXbMfXYiXsviTNMQofWRXhvJH3IK","Admin","1");
        user2 = new CalendarUser(2,"user2@example.com","$2a$04$PiVhNPAxunf0Q4IMbVeNIuH4M4ecySWHihyrclxW..PLArjLbg8CC","User2","2");
    }
    Role user_role, admin_role;
    private void seedRoles(){
        user_role = new Role(0, "ROLE_USER");
        user_role = roleRepository.save(user_role);
        admin_role = new Role(1, "ROLE_ADMIN");
        admin_role = roleRepository.save(admin_role);
    }
    private void seedEvents(){
        // Event 1
        Event event1 = new Event(
                100,
                "Birthday Party",
                "This is going to be a great birthday",
             LocalDateTime.of(2023, 6,3,6,36,00),
                user,
                admin
                );
        // Event 2
        Event event2 = new Event(
                101,
                "Conference Call",
                "Call with the client",
             LocalDateTime.of(2023, 11,23,13,00,00),
                user2,
                user
                );
        // Event 3
        Event event3 = new Event(
                102,
                "Vacation",
                "Paragliding in Greece",
             LocalDateTime.of(2023, 8,14,11,30,00),
                admin,
                user2
                );
        // save Event
        eventRepository.save(event1);
        eventRepository.save(event2);
        eventRepository.save(event3);
        List<Event> events = eventRepository.findAll();
        logger.info("Events: {}", events);
    }
    private void seedCalendarUsers(){
        // user1
        user.addRole(user_role);
        // admin2
        admin.addRole(user_role);
        admin.addRole(admin_role);
        // user2
        user2.addRole(user_role);
        // CalendarUser
        calendarUserRepository.save(user);
        calendarUserRepository.save(admin);
        calendarUserRepository.save(user2);
        List<CalendarUser> users = calendarUserRepository.findAll();
        logger.info("CalendarUsers: {}", users);
    }
}
```

这将在加载时执行，并将与我们在 H2 数据库中执行相同的数据种子到我们的 MongoDB 中。

## 使用 MongoDB 映射领域对象

让我们首先映射我们的 `Event.java` 文件，以便将每个领域对象作为文档保存在我们的 MongoDB 数据库中。这可以通过以下步骤完成：

1.  在文档数据库中，领域对象映射略有不同，但相同的 ORM 概念仍然适用。让我们从`Event` JPA 实现开始，然后我们可以将我们的`Entity`转换为文档映射：

    ```java
    //src/main/java/com/packtpub/springsecurity/domain/Event.java
    @Entity
    @Table(name = "events")
    public class Event implements Serializable{
        @Id
        @GeneratedValue(strategy = GenerationType.AUTO)
        private Integer id;
        @NotEmpty(message = "Summary is required")
        private String summary;
        @NotEmpty(message = "Description is required")
        private String description;
        @NotNull(message = "When is required")
        private Calendar dateWhen;
        @NotNull(message = "Owner is required")
        @ManyToOne(fetch = FetchType.LAZY)
        @JoinColumn(name="owner", referencedColumnName="id")
        private CalendarUser owner;
        @ManyToOne(fetch = FetchType.LAZY)
        @JoinColumn(name="attendee", referencedColumnName="id")
        private CalendarUser attendee;
    ...
    }
    ```

1.  在基于实体的 JPA 映射中，我们需要使用六个不同的注解来创建所需的映射。现在，在基于文档的 MongoDB 映射中，我们需要更改所有之前的映射注解。以下是我们`Event.java`文件的完全重构示例：

    ```java
    //src/main/java/com/packtpub/springsecurity/domain/Event.java
    @Document(collection="events")
    public class Event implements Persistable<Integer>, Serializable{
        @Id
        private Integer id;
        @NotEmpty(message = "Summary is required")
        private String summary;
        @NotEmpty(message = "Description is required")
        private String description;
        @NotNull(message = "When is required")
        private LocalDateTime dateWhen;
        @NotNull(message = "Owner is required")
        @DBRef
        private CalendarUser owner;
        @DBRef
        private CalendarUser attendee;
    ...
    }
    ```

    在前面的代码中，我们可以看到以下显著的变化。

1.  首先，我们声明类为`@o.s.d.mongodb.core.mapping.Document`类型，并为这些文档提供一个集合名称。

1.  接下来，`Event`类必须实现`o.s.d.domain.Persistable`接口，为我们的文档提供主键类型（`Integer`）。

1.  现在，我们将我们的领域 ID 的注解更改为`@o.s.d.annotation.Id`，以定义领域主键。

1.  之前，我们必须将所有者与会者`CalendarUser`对象映射到两个不同的映射注解。

    现在，我们只需要定义两种类型为`@o.s.d.mongodb.core.mapping.DBRef`，并允许 Spring Data 处理底层引用。

1.  我们必须添加的最后一个注解定义了一个特定的构造函数，用于通过使用`@o.s.d.annotation.PersistenceConstructor`注解将新文档添加到我们的文档中。

1.  现在我们已经审查了从 JPA 到 MongoDB 重构所需的更改，让我们重构其他领域对象，从`Role.java`文件开始，如下所示：

    ```java
    //src/main/java/com/packtpub/springsecurity/domain/Role.java
    @Document(collection="role")
    public class Role  implements Persistable<Integer>, Serializable {
        @Id
        private Integer id;
        private String name;
    ...
    }
    ```

1.  我们需要重构的最后一个领域对象是我们的`CalendarUser.java`文件。毕竟，这是我们在这个应用程序中最复杂的领域对象：

    ```java
    //src/main/java/com/packtpub/springsecurity/domain/CalendarUser.java
    @Document(collection="calendar_users")
    public class CalendarUser implements Persistable<Integer>, Serializable {
        @Id
        private Integer id;
        private String firstName;
        private String lastName;
        private String email;
        private String password;
        @DBRef(lazy = false)
        private Set<Role> roles = new HashSet<>(5);
    …
    }
    ```

如您所见，将我们的领域对象从 JPA 重构到 MongoDB 的努力相当简单，所需的注解配置比 JPA 配置少。

### MongoDB 的 Spring Data 仓库

我们现在只需要对从 JPA 实现到 MongoDB 实现的重构进行少量更改。我们将从重构我们的`CalendarUserRepository.java`文件开始，更改我们的仓库扩展的接口，如下所示：

```java
//com/packtpub/springsecurity/repository/CalendarUserRepository.java
public interface CalendarUserRepository extends MongoRepository<CalendarUser, Integer> {
    CalendarUser findByEmail(String email);
}
...
```

适当地修改`RoleRepository.java`文件。

重要提示

如果您需要帮助进行这些更改，请记住`chapter05.05`的源代码将提供可供参考的完整代码。

## MongoDB 中的数据访问对象

在我们的`EventDao`接口中，我们需要创建一个新的`Event`对象。在 JPA 中，我们可以自动生成我们的对象 ID。在 MongoDB 中，有几种方法可以分配主键标识符，但为了演示的目的，我们只是使用原子计数器，如下所示：

```java
//src/main/java/com/packtpub/springsecurity/dataaccess/MongoEventDao.java
@Repository
public class MongoEventDao implements EventDao {
private EventRepository repository;
// Simple Primary Key Generator
private AtomicInteger eventPK = new AtomicInteger(102);
  @Override
    public int createEvent(final Event event) {
...
        // Get the next PK instance
        event.setId(eventPK.incrementAndGet());
        Event newEvent = repository.save(event);
        return newEvent.getId();
    }
...
}
```

在技术上，我们的`CalendarUserDao`对象没有发生变化，但为了保持本书的一致性，我们将实现文件重命名为表示使用`Mongo`：

```java
@Repository
public class MongoCalendarUserDao implements CalendarUserDao {
```

对于这个重构示例，没有其他**数据访问对象（DAO**）更改所需的。

开始应用吧，它将表现得和以前一样。

尝试以**user1**和**admin1**的身份登录。然后测试应用程序，确保两个用户都可以向系统中添加新事件，确保整个应用程序的映射是正确的。

重要提示

你应该从`chapter05.05-calendar`的源代码开始。

# 摘要

我们已经探讨了 Spring Data 项目的强大功能和灵活性，并研究了与应用程序开发相关的几个方面，以及它与 Spring Security 的集成。在本章中，我们介绍了 Spring Data 项目及其一些功能。我们还看到了将遗留的 JDBC 代码使用 SQL 转换为 ORM 使用 JPA 的过程，以及从使用 Spring Data 的 JPA 实现到使用 Spring Data 的 MongoDB 实现的过程。我们还介绍了配置 Spring Security 以利用关系型数据库中的`ORM 实体`和文档数据库。

在下一章中，我们将探讨 Spring Security 对基于*LDAP 认证*的内置支持。
