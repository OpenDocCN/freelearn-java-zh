# 13

# 使用输出适配器和 Hibernate Reactive 持久化数据

在上一章中，我们学习了使用 Quarkus 反应式能力可以为系统带来的某些优势。我们在反应式道路上的第一步是使用 **RESTEasy Reactive** 实现反应式输入适配器。尽管输入适配器的端点正在以反应式方式提供服务，但我们仍然有输出适配器以同步和阻塞的方式工作。

为了将六边形系统转变为更反应式的一个，在本章中，我们首先将学习如何配置 `Panache`。一旦系统实体得到适当配置，我们将学习如何使用这些实体以反应式方式连接到 MySQL 数据库。

本章我们将涵盖以下主题：

+   介绍 Hibernate Reactive 和 `Panache`

+   在输出适配器上启用反应式行为

+   测试反应式输出适配器

由于我们在上一章已经实现了反应式输入适配器，我们的目标是在本章中通过实现反应式输出适配器来扩展六边形系统的反应式行为。这种实现发生在框架六边形上，这是我们专注于适配器的架构元素。

到本章结束时，您将学习如何将 Quarkus 与六边形系统集成以以反应式方式访问数据库。通过理解所需的配置步骤和基本实现细节，您将能够实现反应式输出适配器。这些知识将帮助您应对非阻塞 I/O 请求比 I/O 阻塞请求提供更多优势的情况。

# 技术要求

要编译和运行本章中提供的代码示例，您需要在您的计算机上安装最新的 **Java SE 开发工具包** 和 **Maven 3.8**。它们都适用于 Linux、Mac 和 Windows 操作系统。

此外，您需要在您的机器上安装 **Docker**。

您可以在 GitHub 上找到本章的代码文件，链接为 [`github.com/PacktPublishing/-Designing-Hexagonal-Architecture-with-Java---Second-Edition/tree/main/Chapter13`](https://github.com/PacktPublishing/-Designing-Hexagonal-Architecture-with-Java---Second-Edition/tree/main/Chapter13)。

# 介绍 Hibernate Reactive 和 Panache

在过去几年中，处理 Java 中数据库操作的技术和技巧已经发生了很大的变化。基于 **Java 持久化 API**（**JPA**）规范，我们被介绍了几种 ORM 实现，如 Spring Data JPA、EclipseLink，当然还有 Hibernate。这些技术通过抽象出处理数据库所需的大部分管道工作，使我们的生活变得更加容易。

Quarkus 集成了 Hibernate ORM 和其反应式对应物 Hibernate Reactive。此外，Quarkus 还附带了一个名为 `Panache` 的库，该库简化了我们与数据库的交互。

接下来，我们将简要介绍 Hibernate Reactive 和 Panache 的主要功能。

## Hibernate Reactive 功能

找到一个能够解决与数据库访问相关所有问题的银弹方案，如果不是不可能的话，是非常罕见的。当我们谈论数据库处理的反应式和命令式方法时，理解这两种方法的优势和劣势是至关重要的。

命令式方法访问数据库之所以吸引人，在于其开发代码的简单性。当你需要使用命令式方法读取或持久化数据时，需要调整和思考的事情较少。然而，当其阻塞特性开始影响系统的用例时，这种方法可能会导致问题。为了避免这种问题，我们有了反应式方法，它使我们能够以非阻塞的方式处理数据库，但这并不是没有在开发和处理数据库时带来额外的复杂性以及新的问题和挑战。

原始的 Hibernate 实现是为了解决开发者在将 Java 对象映射到数据库实体时遇到的问题而设计的。原始实现依赖于 I/O 阻塞同步通信与数据库交互。这曾经是，现在仍然是 Java 中访问数据库最传统的方式。另一方面，Hibernate Reactive 源于对反应式编程运动的渴望以及对异步通信访问数据库的需求。Hibernate Reactive 不是依赖于 I/O 阻塞，而是依赖于 I/O 非阻塞通信与数据库交互。

在反应式实现中，实体映射属性保持不变。然而，变化的是我们打开数据库反应式连接的方式以及我们应如何构建软件代码以反应式地处理数据库实体。

当使用 Quarkus 时，无需基于`persistence.xml`文件提供反应式持久化配置，因为 Quarkus 已经为我们配置好了。尽管如此，我们仍将简要探讨它，以便了解 Hibernate Reactive 单独的工作方式。

要设置 Hibernate Reactive，你可以遵循配置`META-INF/persistence.xml`文件的常规方法，如下面的示例所示：

```java
<persistence-unit name="mysql">
    <provider>
       org.hibernate.reactive.provider
       .ReactivePersistenceProvider
    </provider>
    <class>dev.davivieria.SomeObject</class>
    <properties>
    <property name=»javax.persistence.jdbc.url»
       value=»jdbc:mysql://localhost/hreact"/>
    </properties>
</persistence-unit>
```

注意，我们正在使用`ReactivePersistenceProvider`来打开数据库的反应式连接。一旦`persistence.xml`文件配置得当，我们就可以在我们的代码中开始使用 Hibernate Reactive：

```java
import static javax.persistence.Persistence.createEnti
  tyManagerFactory;
SessionFactory factory = createEntityManagerFactory (
persistenceUnitName ( args ) ).unwrap(SessionFac
  tory.class);
/** Code omitted **/
public static String persistenceUnitName(String[] args) {
    return args.length > 0 ?
    args[0] : "postgresql-example";
}
```

我们首先导入 Hibernate Reactive 提供的静态`javax.persistence.Persistence.createEntityManagerFactory`方法。这个静态方法简化了`SessionFactory`对象的创建。

为了创建`SessionFactory`对象，系统使用`persistence.xml`文件中定义的属性。有了`SessionFactory`，我们可以开始与数据库进行反应式通信：

```java
SomeObject someObject = new SomeObject();
factory.withTransaction(
     (
org.hibernate.reactive.mutiny.Mutiny.
  Transaction session,
org.hibernate.reactive.mutiny.Mutiny.Transaction tx) ->
session.persistAll(someObject)).subscribe();
```

要持久化数据，首先，我们需要通过调用 `withTransaction` 方法创建一个事务。在事务内部，我们从 `SessionFactory` 调用 `persistAll` 方法来持久化一个对象。我们调用 `subscribe` 方法以非阻塞方式触发持久化操作。

通过在应用程序和数据库之间建立一层，Hibernate 提供了我们处理 Java 数据库所需的所有基本功能。

现在，让我们看看 `Panache` 如何使事情变得更加简单。

## Panache 特性

`Panache` 位于 Hibernate 之上，并通过提供处理数据库实体的简单接口来进一步增强它。`Panache` 主要是为了与 Quarkus 框架一起使用而开发的，它是一个旨在抽象处理数据库实体所需的大量样板代码的库。使用 `Panache`，你可以轻松地应用如 **Active Record** 和 **Repository** 这样的数据库模式。让我们简要地看看如何做到这一点。

### 应用活动记录模式

在 `PanacheEntity` 中，看看以下示例：

```java
@Entity
@Table(name="locations")
public class Location extends PanacheEntity {
    @Id @GeneratedValue
    private Integer id;
    @NotNull @Size(max=100)
    public String country;
    @NotNull @Size(max=100)
    public String state;
    @NotNull @Size(max=100)
    public String city;
}
```

前面的 `Location` 类是一个基于 Hibernate 的常规实体，它扩展了 `PanacheEntity`。除了扩展 `PanacheEntity` 之外，这个 `Location` 类没有其他新内容。我们使用了 `@NotNull` 和 `@Size` 等注解来验证数据。

以下是一些我们可以使用活动记录实体执行的操作：

+   要列出实体，我们可以调用 `listAll` 方法。此方法在 `Location` 上可用，因为我们正在扩展 `PanacheEntity` 类：

    ```java
    List<Location> locations = Location.listAll();
    ```

+   要删除所有 `Location` 实体，我们可以调用 `deleteAll` 方法：

    ```java
    Location.deleteAll();
    ```

+   要通过其 ID 查找特定的 `Location` 实体，我们可以使用 `findByIdOptional` 方法：

    ```java
    Optional<Location> optional = Location.findByIdOp
      tional(locationId);
    ```

+   要持久化 `Location` 实体，我们必须在打算持久化的 `Location` 实例上调用 `persist` 方法：

    ```java
    Location location = new Location();
    location.country = "Brazil";
    location.state = "Sao Paulo";
    location.city = "Santo Andre";
    location.persist();
    ```

每次我们执行前面描述的任何操作时，它们都会立即提交到数据库。

现在，让我们看看如何使用 `Panache` 来应用仓储模式。

### 应用仓储模式

我们不是使用实体类在数据库上执行操作，而是使用一个单独的类，这个类通常专门用于在仓储模式中提供数据库操作。这种类就像数据库的仓储接口一样工作。

要应用仓储模式，我们应该使用常规的 Hibernate 实体：

```java
@Entity
@Table(name="locations")
public class Location {
/** Code omitted **/
}
```

注意，此时我们并没有扩展 `PanacheEntity` 类。在仓储模式中，我们不是通过实体类直接调用数据库操作。相反，我们通过仓储类调用它们。以下是如何实现仓储类的示例：

```java
@ApplicationScoped
public class LocationRepository implements PanacheReposi
  tory<Location> {
   public Location findByCity(String city){
       return find ("city", city).firstResult();
   }
   public Location findByState(String state){
       return find("state", state).firstResult();
   }
   public void deleteSomeCountry(){
       delete ("country", "SomeCountry");
  }
}
```

通过在 `LocationRepository` 类上实现 `PanacheRepository`，我们启用了 `PanacheEntity` 类中存在的所有标准操作，如 `findById`、`delete`、`persist` 等。此外，我们可以通过使用 `PanacheEntity` 类提供的 `find` 和 `delete` 方法来定义我们自己的自定义查询，就像前面的示例中所做的那样。

注意，我们将仓库类标注为`@ApplicationScoped`类型的 bean。这意味着我们可以在其他类中注入并使用它：

```java
@Inject
LocationRepository locationRepository;
public Location findLocationByCity(City city){
    return locationRepository.findByCity(city);
}
```

在这里，我们有在仓库类上可用的最常见操作：

+   要列出所有`Location`实体，我们需要从`LocationRepository`调用`listAll`方法：

    ```java
    List<Location> locations = locationReposi
      tory.listAll();
    ```

+   通过在`LocationRepository`上调用`deleteAll`，我们移除所有的`Location`实体：

    ```java
    locationRepository.deleteAll();
    ```

+   要通过其 ID 查找`Location`实体，我们在`LocationRepository`上调用`findByIdOptional`方法：

    ```java
    Optional<Location> optional = locationReposi
      tory.findByIdOptional(locationId);
    ```

+   要持久化`Location`实体，我们需要将`Location`实例传递给`LocationRepository`中的`persist`方法：

    ```java
    Location location = new Location();
    location.country = "Brazil";
    location.state = "Sao Paulo";
    location.city = "Santo Andre";
    locationRepository.persist(location);
    ```

在前面的示例中，我们使用仓库类执行所有数据库操作。我们在这里调用的方法与 Active Record 方法中存在的那些方法相同。这里唯一的区别是仓库类的使用。

通过学习如何使用`Panache`来应用 Active Record 和 Repository 模式，我们提高了提供处理数据库实体良好方法的能力。没有更好的或更差的模式。项目的具体情况最终将决定哪种模式更适合。

`Panache`是一个专门为 Quarkus 制作的库。因此，将数据库配置委托给 Quarkus 以连接 Hibernate Reactive 对象（如`SessionFactory`和`Transaction`）到`Panache`的最佳方式是，Quarkus 将自动为您提供这些对象。

现在我们已经熟悉了 Hibernate Reactive 和`Panache`，让我们看看如何在六边形系统中实现输出适配器。

# 启用输出适配器的反应式行为

使用六边形架构最重要的好处之一是提高了在不进行重大重构的情况下更改技术的灵活性。六边形系统设计得如此之好，以至于其领域逻辑和业务规则对执行它们所使用的技术一无所知。

没有免费的午餐——当我们决定使用六边形架构时，我们必须为这种架构能提供的利益付出代价。（这里的代价是指按照六边形原则结构化系统代码所需的工作量和复杂性的显著增加。）

如果您担心代码重用，您可能会发现一些实践难以将代码从特定技术中解耦。例如，考虑一个场景，其中我们有一个领域实体类和一个数据库实体类。我们可能会争论，*为什么不只有一个类同时服务于这两个目的呢？* 好吧，最终，这完全是一个优先级的问题。如果您认为领域和特定技术类之间的耦合不是问题，那么请继续。在这种情况下，您将不会承担维护领域模型及其所有支持基础设施代码的负担。然而，相同的代码会服务于不同的目的，从而违反了**单一职责原则**（**SRP**）。否则，如果您认为使用相同代码服务于不同目的存在风险，那么输出适配器可以帮助您。

在*第二章*《在领域六边形内封装业务规则》中，我们介绍了一个输出适配器，该适配器将应用程序与文件系统集成。在*第四章*《创建与外部世界交互的适配器》中，我们创建了一个更详细的输出适配器，用于与 H2 内存数据库通信。现在，我们有了 Quarkus 工具箱，我们可以创建响应式输出适配器。

## 配置响应式数据源

为了继续在前一章中通过实现响应式输入适配器开始的响应式努力，我们将通过执行以下步骤创建和连接响应式输出适配器到这些响应式输入适配器：

1.  让我们从在框架六边形的`pom.xml`文件中配置所需的依赖项开始：

    ```java
    <dependencies>
      <dependency>
        <groupId>io.quarkus</groupId>
        artifactId>quarkus-reactive-mysql-client
          </artifactId>
      </dependency>
      <dependency>
        <groupId>io.quarkus</groupId>
       <artifactId>quarkus-hibernate-reactive-panache</ar
          tifactId>
      </dependency>
    </dependencies>
    ```

    `quarkus-reactive-mysql-client`依赖包含我们需要的库，用于与 MySQL 数据库建立响应式连接，而`quarkus-hibernate-reactive-panache`依赖包含 Hibernate Reactive 和`Panache`。需要注意的是，这个库特别适合响应式活动。对于非响应式活动，Quarkus 提供不同的库。

1.  现在，我们需要在启动六边形的`application.properties`文件上配置数据库连接。让我们从数据源属性开始：

    ```java
    quarkus.datasource.db-kind = mysql
    quarkus.datasource.reactive = true
    quarkus.datasource.reactive.url = mysql://lo
      calhost:3306/inventory
    quarkus.datasource.username = root
    quarkus.datasource.password = password
    ```

    `quarkus.datasource.db-kind`属性不是必需的，因为 Quarkus 可以通过查看从 Maven 依赖中加载的特定数据库客户端来推断数据库类型。将`quarkus.datasource.reactive`设置为`true`，我们正在强制执行响应式连接。我们需要在`quarkus.datasource.reactive.url`上指定响应式数据库连接 URL。

1.  最后，我们必须定义 Hibernate 配置：

    ```java
    quarkus.hibernate-orm.sql-load-script=inventory.sql
    quarkus.hibernate-orm.database.generation = drop-and-
      create
    quarkus.hibernate-orm.log.sql = true
    ```

    在 Quarkus 创建数据库及其表之后，您可以加载一个`.sql`文件来在数据库上执行更多指令。默认情况下，它会搜索并加载一个名为`import.sql`的文件。我们可以通过使用`quarkus.hibernate-orm.sql-load-script`属性来改变这种行为。

    注意在生产环境中不要使用`quarkus.hibernate-orm.database.generation = drop-and-create`。否则，它将删除您所有的数据库表。如果您没有设置任何值，默认值`none`将被使用。默认行为不会对数据库进行任何更改。

    最后，我们启用`quarkus.hibernate-orm.log.sql`来查看 Hibernate 在幕后执行的 SQL 查询。我建议您只为开发目的启用`log`功能。在生产环境中运行应用程序时，别忘了禁用此选项。

让我们现在看看如何配置应用程序实体以与 MySQL 数据库一起工作。

## 配置实体

拓扑和库存系统需要四个数据库表来存储其数据：路由器、交换机、网络和位置。每个这些表都将被映射到一个正确配置以与 MySQL 数据源一起工作的 Hibernate 实体类。

我们将应用 Repository 模式，因此我们不需要实体来执行数据库操作。相反，我们将创建单独的仓库类来在数据库上触发操作，但在创建仓库类之前，让我们先实现拓扑和库存系统的 Hibernate 实体。我们将配置这些实体以与 MySQL 数据库一起工作。

### 路由器实体

对于这个实体以及随后将要实现的其它实体，我们应该在框架六角形的`dev.davivieira.topologyinventory.framework.adapters.output.mysql.data`包中创建类。

`Router`实体类应该看起来像这样：

```java
@Entity(name = "RouterData")
@Table(name = "routers")
@EqualsAndHashCode(exclude = "routers")
public class RouterData implements Serializable {
    @Id
    @Column(name="router_id", columnDefinition =
      «BINARY(16)")
    private UUID routerId;
    @Column(name="router_parent_core_id",
    columnDefinition = "BINARY(16)")
    private UUID routerParentCoreId;
   /** Code omitted **/
}
```

对于`routerId`和`routerParentCoreId`字段，我们必须将`columnDefinition`，即`@Column`注解参数，设置为`BINARY(16)`。这是在 MySQL 数据库上使`UUID`属性工作所必需的。

然后，我们创建路由器与其他表之间的关系映射：

```java
{
    /**Code omitted**/
   @ManyToOne(cascade = CascadeType.ALL)
   @JoinColumn(name="location_id")
   private LocationData routerLocation;
   @OneToMany(cascade = {CascadeType.MERGE},
   fetch = FetchType.EAGER)
   @JoinColumn(name="router_id")
   private List<SwitchData> switches;
   @OneToMany(cascade = CascadeType.ALL, fetch =
     FetchType.EAGER)
   @JoinColumn(name="router_parent_core_id")
   private Set<RouterData> routers;
   /**Code omitted**/
}
```

在这里，我们定义了路由器和位置之间的多对一关系。之后，我们有两个与交换机和路由器分别的一对多关系。使用`fetch = FetchType.EAGER`属性来避免在反应式连接期间可能发生的任何映射错误。

让我们继续配置`Switch`实体类。

### Switch 实体

以下代码展示了我们应该如何实现`Switch`实体类：

```java
@Entity
@Table(name = "switches")
public class SwitchData {
    @ManyToOne
    private RouterData router;
    @Id
    @Column(name="switch_id", columnDefinition =
      «BINARY(16)")
    private UUID switchId;
    @Column(name="router_id", columnDefinition =
      «BINARY(16)")
    private UUID routerId;
    @OneToMany(cascade = CascadeType.ALL, fetch =
      FetchType.EAGER)
    @JoinColumn(name="switch_id")
    private Set<NetworkData> networks;
    @ManyToOne
    @JoinColumn(name="location_id")
    private LocationData switchLocation;
    /**Code omitted**/
}
```

我们省略了其他列属性，只关注 ID 和关系。我们首先定义了交换机和路由器之间的多对一关系。主键是`switchId`字段，它恰好是一个`UUID`属性。我们还有一个`UUID`属性来映射`routerId`字段。

此外，交换机和网络之间存在一对一关系，交换机和位置之间存在多对一关系。

现在，让我们配置`Network`实体类。

### 网络实体

虽然我们不将网络视为领域模型中的实体，但在数据库中有一个单独的表。因此，在框架六边形级别，我们将它们视为数据库实体，但当它们达到领域六边形时，我们将它们视为值对象。这个例子表明，六边形系统决定了数据在领域六边形级别如何被处理。通过这样做，六边形系统保护了领域模型免受技术细节的影响。

我们如下实现`Network`实体类：

```java
@Entity
@Table(name = "networks")
public class NetworkData {
    @ManyToOne
    @JoinColumn(name="switch_id")
    private SwitchData switchData;
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name="network_id")
    private int id;
   /**Code omitted**/
}
```

这是一个简单的实体类，它在网络和交换之间有一个多对一的关系。对于网络，我们依赖数据库生成网络 ID。此外，网络在领域模型中不被视为实体。相反，我们将网络视为由聚合控制的值对象。对于聚合，我们需要处理`UUID`，但对于值对象，我们不需要。这就是为什么我们不处理网络数据库实体的 UUID。

我们还需要实现一个用于位置的最后一个实体。让我们来做这件事。

### 位置实体

在网络中，位置在领域六边形级别不被视为一个实体，但由于我们有一个单独的位置表，因此我们需要在框架六边形级别将其视为数据库实体。

以下代码用于实现`Location`实体类：

```java
Entity
@Table(name = "location")
public class LocationData {
    @Id
    @Column(name="location_id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int locationId;
    @Column(name="address")
    private String address;
    @Column(name="city")
    private String city;
    /**Code omitted**/
}
```

我们再次依赖数据库内置的 ID 生成机制来处理位置数据的 ID。之后，我们有了`address`和`city`等属性，它们是位置的一部分。

现在我们已经适当地配置了所有必需的实体，我们可以继续前进并使用`Panache`创建响应式存储库类，我们将使用这些类来触发我们配置的实体所进行的数据库操作。

## 实现响应式存储库类

通过实现`PanacheRepositoryBase`接口，您创建了一个响应式存储库类。我们需要一个存储库类用于路由操作，另一个用于交换操作。

定义一个聚合根的单一存储库至关重要。在我们的案例中，`Router`实体是路由管理操作的聚合根，而`Switch`是交换管理操作的聚合根。聚合的目的在于确保所有受该聚合控制的对象的一致性。任何聚合的入口点始终是聚合根。为了确保数据库事务中的聚合一致性，我们定义了一个专门的存储库类，该类仅用于根据聚合根控制数据库操作。

我们即将实现的类位于`dev.davivieira.topologyinventory.framework.adapters.output.mysql.repository`包中：

+   以下代码实现了`RouterManagementRepository`类：

    ```java
    @ApplicationScoped
    public class RouterManagementRepository implements Pa
      nacheRepositoryBase<RouterData, UUID> {
    }
    ```

    注意，我们正在传递`RouterData`作为我们正在处理的实体，以及`UUID`作为映射到 ID 的属性类型。如果我们不需要任何自定义查询，我们可以留这个类为空，因为`Panache`已经提供了大量的标准数据库操作。

    注意，我们还在该类上标注了`@ApplicationScoped`，这样我们就可以在其他地方注入该组件，例如我们即将实现的输出适配器。

+   以下代码实现了`SwitchManagementRepository`类：

    ```java
    @ApplicationScoped
    public class SwitchManagementRepository implements Pa
      nacheRepositoryBase<SwitchData, UUID> {
    }
    ```

    这里，我们遵循与`RouterManagementRepository`类相同的方法。

在正确实现了响应式存储库类之后，我们准备好创建响应式输出适配器。让我们这么做吧！

## 实现响应式输出适配器

只是为了回顾一下，我们需要为`RouterManagementOutputPort`输出端口接口提供一个适配器实现：

```java
public interface RouterManagementOutputPort {
    Router retrieveRouter(Id id);
    boolean removeRouter(Id id);
    Router persistRouter(Router router);
}
```

当实现 MySQL 输出适配器时，我们将为前面每个方法声明提供一个响应式实现。

我们还需要实现`SwitchManagementOutputPort`输出适配器接口：

```java
public interface SwitchManagementOutputPort {
    Switch retrieveSwitch(Id id);
}
```

这比较简单，因为我们只需要提供一个响应式实现的方法。

让我们先来实现路由管理响应式输出适配器。

## MySQL 输出适配器的响应式路由管理

为了使六边形系统能够与 MySQL 数据库通信，我们需要创建一个新的输出适配器以允许这种集成（因为我们使用 Quarkus，这样的输出适配器实现相当简单）。我们将按照以下步骤进行：

1.  我们首先注入`RouterManagementRepository`存储库类：

    ```java
    @ApplicationScoped
    public class RouterManagementMySQLAdapter implements
      RouterManagementOutputPort {
        @Inject
        RouterManagementRepository
          routerManagementRepository;
        /** Code omitted **/
    }
    ```

    我们将使用`RouterManagementRepository`存储库来执行数据库操作。

1.  然后，我们实现`retrieveRouter`方法：

    ```java
    @Override
    public Router retrieveRouter(Id id) {
        var routerData =
        routerManagementRepository.findById(id.getUuid())
          .subscribe()
          .asCompletionStage()
          .join();
        return RouterMapper.routerDataToDomain(router
          Data);
    }
    ```

    当我们调用`routerManagementRepository.findById(id.getUuid())`时，系统启动一个 I/O 非阻塞操作。这个`subscribe`调用试图解析由`findById`操作产生的项目。然后，我们调用`asCompletionStage`来接收项目。最后，我们调用`join`，它在操作完成时返回结果值。

1.  现在，我们需要实现`removeRouter`方法：

    ```java
    @Override
    public Router removeRouter(Id id) {
     return routerManagementRepository
            .deleteById(
            id.getUuid())
            .subscribe().asCompletionStage().join();
    }
    ```

    在这里，我们调用`routerManagementRepository.deleteById(id.getUuid())` `Panache`操作从数据库中删除一个路由器。之后，我们调用`subscribe`、`asCompletionStage`和`join`来执行这些操作以实现响应式。

1.  最后，我们实现`persistRouter`方法：

    ```java
    @Override
    public Router persistRouter(Router router) {
        var routerData =
        RouterH2Mapper.routerDomainToData(router);
        Panache.withTransaction(
        ()->routerManagementRepository.persist
        (routerData));
        return router;
    }
    ```

    这里的结构不同。为了确保在请求过程中客户端和服务器之间事务不会丢失，我们将持久化操作包裹在`Panache.withTransaction`中。这是需要持久化数据操作的要求。

现在，让我们实现开关管理的响应式输出适配器。

## MySQL 输出适配器的响应式开关管理

这里使用的方法与我们实现路由管理响应式输出适配器时使用的方法相同。我们将执行以下步骤来实现响应式输出适配器：

1.  让我们先注入`SwitchManagementRepository`仓库类：

    ```java
    @ApplicationScoped
    public class SwitchManagementMySQLAdapter implements
      SwitchManagementOutputPort {
        @Inject
        SwitchManagementRepository
          switchManagementRepository;
        /** Code omitted **/
    }
    ```

    正如我们已经看到的，注入一个仓库类是必要的，这样我们就可以用它来触发数据库操作。

1.  在此之后，我们实现`retrieveSwitch`方法：

    ```java
    @Override
    public Switch retrieveSwitch(Id id) {
        var switchData =
        switchManagementRepository.findById(id.getUuid())
           .subscribe()
           .asCompletionStage()
           .join();
        return RouterMapper.switchDataToDo
          main(switchData);
    }
    ```

    我们使用这个方法来响应式地检索一个`Switch`对象。因为没有持久化方法，因为所有的写操作都应该始终通过路由管理输出适配器来执行。

通过在六边形系统中实现响应式输出适配器，我们可以利用响应式编程技术的优势。在六边形架构中，在同一系统中同时拥有响应式和命令式输出适配器来满足不同的需求并不是什么大问题。

Quarkus 数据库的响应式特性对于任何尝试开发响应式系统的开发者来说至关重要。通过理解如何使用这些特性，我们可以为我们的应用程序处理数据库的方式提供一个响应式的替代方案。但这并不意味着响应式方法总是比传统的命令式方法更好；这取决于你和你项目的需求，来决定哪种方法更适合。

现在我们已经实现了`RouterManagementMySQLAdapter`和`SwitchManagementMySQLAdapter`输出适配器，让我们测试它们。

# 测试响应式输出适配器

我们需要实现单元测试来确保输出适配器的方法按预期工作。以下是如何为`RouterManagementMySQLAdapter`创建单元测试的示例：

```java
@QuarkusTest
public class RouterManagementMySQLAdapterTest {
    @InjectMock
    RouterManagementMySQLAdapter
    routerManagementMySQLAdapter;
    @Test
    public void testRetrieveRouter() {
        Router router = getRouter();
        Mockito.when(
        routerManagementMySQLAdapter.
        retrieveRouter(router.getId())).thenReturn(router);
        Router retrievedRouter =
        routerManagementMySQLAdapter.
        retrieveRouter(router.getId());
        Assertions.assertSame(router, retrievedRouter);
    }
   /** Code omitted **/
}
```

可以使用`@InjectMock`注解来模拟`RouterManagementMySQLAdapter`输出适配器。在执行`testRetrieveRouter`测试方法时，我们可以通过使用`Mockito.when`来模拟对`routerManagementMySQLAdapter.retrieveRouter(router.getId)`的调用。`thenReturn`方法返回我们的模拟测试应该返回的对象。在这种情况下，它是一个`Router`对象。通过`Assertions.assertSame(router, retrievedRouter)`，我们可以断言`retrieveRouter(router.getId)`的执行结果。

我们不需要实现新的测试类来执行响应式输出适配器的集成测试。我们可以依赖之前章节中使用的相同测试来测试响应式输入适配器。这些测试调用输入适配器，反过来，通过使用用例操作调用输出适配器。

然而，变化的是，我们将需要一个 MySQL 数据库来测试响应式输出适配器。

Quarkus 提供了基于 Docker 的容器，我们可以用于开发或测试目的。为了启用这样的数据库容器，在`application.properties`文件中不需要提供详细的数据源连接配置。以下是我们在测试目的下应该如何配置该文件的方法：

```java
quarkus.datasource.db-kind=mysql
quarkus.datasource.reactive=true
quarkus.hibernate-orm.database.generation=drop-and-create
quarkus.hibernate-orm.sql-load-script=inventory.sql
quarkus.vertx.max-event-loop-execute-time=100
```

注意，我们没有指定数据库连接 URL。通过这样做，Quarkus 理解它需要提供一个数据库。之前描述的 `application.properties` 文件应放置在 `tests/resource/` 目录中。在这个目录内，我们还应放置 `inventory.sql` 文件，该文件将数据加载到数据库中。这个 `.sql` 文件在本章的 GitHub 仓库中可用。

您可以覆盖 `application.properties` 中的条目以使用环境变量。这可能对配置如 `quarkus.hibernate-orm.database.generation` 有用，其中您可以根据应用程序的环境变量设置属性值。例如，对于本地或开发目的，您可以使用 `${DB_GENERATION}`，这是一个解析为 `drop-and-create` 的环境变量。在生产中，这个环境变量可以解析为 `none`。

在正确设置 `application.properties` 和 `inventory.sql` 文件后，我们可以在项目的根目录中运行以下命令来测试应用程序：

```java
$ mvn test
```

以下输出显示了在测试期间启动的 MySQL Docker 容器：

```java
2021-10-10 01:33:40,242 INFO  [  .0.24]] (build-10) Creating container for image: mysql:8.0.24
2021-10-10 01:33:40,876 INFO  [  .0.24]] (build-10) Starting container with ID: 67e788aab66f2f2c6bd91c0be1a164117294ac29cc574941ad41ff5760de918c
2021-10-10 01:33:41,513 INFO  [  .0.24]] (build-10) Container mysql:8.0.24 is starting: 67e788aab66f2f2c6bd91c0be1a164117294ac29cc574941ad41ff5760de918c
2021-10-10 01:33:41,520 INFO  [  .0.24]] (build-10) Waiting for database connection to become available at jdbc:mysql://localhost:49264/default using query 'SELECT 1'
2021-10-10 01:34:01,078 INFO  [  .0.24]] (build-10) Container is started (JDBC URL: jdbc:mysql://localhost:49264/default)
2021-10-10 01:34:01,079 INFO  [  .0.24]] (build-10) Container mysql:8.0.24 started in PT20.883579S
2021-10-10 01:34:01,079 INFO  [io.qua.dev.mys.dep.MySQLDevServicesProcessor] (build-10) Dev Services for MySQL started.
```

Quarkus 创建了一个名为 `default` 的数据库，其中创建了表。`inventory.sql` 文件在此 `default` 数据库上运行。

数据库准备就绪后，Quarkus 开始测试系统，结果类似于以下内容：

```java
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 32.672 s - in dev.davivieira.topologyinventory.framework.adapters.input.rest.NetworkManagementAdapterTest
[INFO] Running dev.davivieira.topologyinventory.framework.adapters.input.rest.RouterManagementAdapterTest
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.232 s - in dev.davivieira.topologyinventory.framework.adapters.input.rest.RouterManagementAdapterTest
[INFO] Running dev.davivieira.topologyinventory.framework.adapters.input.rest.SwitchManagementAdapterTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.088 s - in dev.davivieira.topologyinventory.framework.adapters.input.rest.SwitchManagementAdapterTest
[INFO] Running dev.davivieira.topologyinventory.framework.adapters.input.rest.outputAdapters.RouterManagementMySQLAdapterTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.116 s - in dev.davivieira.topologyinventory.framework.adapters.input.rest.outputAdapters.RouterManagementMySQLAdapterTest
[INFO] Running dev.davivieira.topologyinventory.framework.adapters.input.rest.outputAdapters.SwitchManagementMySQLAdapterTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.013 s - in dev.davivieira.topologyinventory.framework.adapters.input.rest.outputAdapters.SwitchManagementMySQLAdapterTest
```

为了测试输出适配器，我们需要调用输入适配器。如果我们能够成功测试输入适配器，那么这也意味着我们已成功测试了输出适配器。

# 摘要

当我们需要使用 Quarkus 处理数据库的响应式操作时，Hibernate Reactive 和 `Panache` 使我们的生活变得更加简单。我们了解到 Hibernate Reactive 是建立在传统 Hibernate 实现之上，但增加了响应式功能。

在研究 `Panache` 的过程中，我们了解到它可以帮助我们实现 Active Record 和 Repository 模式以执行数据库操作。对于实践部分，我们实现了数据库实体、仓库和响应式输出适配器，并将它们一起使用以与 MySQL 数据库进行交互。最后，我们配置了六边形系统测试以使用 Quarkus 提供的 MySQL Docker 容器。

在下一章中，我们将学习一些将六边形系统打包到 Docker 镜像中的技术。我们还将学习如何在 Kubernetes 集群中运行六边形系统。这些知识将使我们能够使我们的六边形应用程序准备好在基于云的环境中部署。

# 问题

1.  Hibernate Reactive 实现了哪个 Java 规范？

1.  Active Record 和 Repository 模式之间的区别是什么？

1.  我们应该实现哪个接口来应用 Repository 模式？

1.  为什么我们应该在 `withTransaction` 方法内运行写操作？

# 答案

1.  Hibernate Reactive 实现了 **JPA** 规范。

1.  Active Record 模式允许我们使用实体类对数据库进行操作，而 Repository 模式则有一个专门的类来执行此类操作。

1.  我们应该实现 `PanacheRepositoryBase` 接口。

1.  为了确保在反应式操作过程中数据库事务不会丢失。
