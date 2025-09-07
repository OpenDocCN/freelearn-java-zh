# 第二章：数据访问工作

任何微服务的采用如果没有与持久化或数据存储集成都是不完整的。在本章中，我们将探讨 Micronaut 框架中持久化和数据访问的各个方面。我们将首先使用一个 **对象关系映射**（**ORM**）框架来与关系型数据库集成。然后，我们将深入探讨使用持久化框架集成数据库。此外，最后，我们将看到一个集成 NoSQL 数据库的示例。为了涵盖这些主题，我们将开发一个宠物诊所应用程序。该应用程序将由以下微服务组成：

+   `pet-owner`：一个使用 Micronaut 中的对象关系映射（ORM）框架与关系型数据库集成的微服务

+   `pet-clinic`：一个使用 Micronaut 中的持久化框架与关系型数据库集成的微服务

+   `pet-clinic-review`：一个与 Micronaut 中的 NoSQL 数据库集成的微服务

到本章结束时，您将具备良好的实践知识，了解如何使用各种类型的持久化框架，以及如何在 Micronaut 框架中将持久化框架与不同类型的数据库（关系型数据库以及 NoSQL 数据库）集成。

# 技术要求

本章中所有的命令和技术说明都是在 Windows 10 和 macOS 上运行的。本章涵盖的代码示例可在本书的 GitHub 仓库中找到，网址为 [`github.com/PacktPublishing/Building-Microservices-with-Micronaut/tree/master/Chapter02`](https://github.com/PacktPublishing/Building-Microservices-with-Micronaut/tree/master/Chapter02)。

需要在开发环境中安装和设置以下工具：

+   **Java SDK**：版本 13 或更高（我们使用了 Java 14）。

+   **Maven**：这是可选的，仅当您想使用 Maven 作为构建系统时才需要。然而，我们建议在任何开发机器上设置 Maven。有关下载和安装 Maven 的说明，请参阅 [`maven.apache.org/download.cgi`](https://maven.apache.org/download.cgi)。

+   **开发 IDE**：根据您的偏好，可以使用任何基于 Java 的 IDE，但为了编写本章，使用了 IntelliJ。

+   **Git**：有关下载和安装 Git 的说明，请参阅 [`git-scm.com/downloads`](https://git-scm.com/downloads)。

+   **PostgreSQL**：有关下载和安装 PostgreSQL 的说明，请参阅 [`www.postgresql.org/download/`](https://www.postgresql.org/download/)。

+   **MongoDB**：MongoDB Atlas 提供了一个免费的在线数据库即服务，存储空间高达 512 MB。但是，如果您更喜欢本地数据库，则可以找到下载和安装的说明，网址为 [`docs.mongodb.com/manual/administration/install-community/`](https://docs.mongodb.com/manual/administration/install-community/)。我们为本章的编写使用了本地安装。

+   **MongoDB Studio 3T**: 在 MongoDB 本地安装中，我们使用了 Studio 3T 的图形用户界面。有关下载和安装 Studio 3T 的说明，请参阅[`studio3t.com/download/`](https://studio3t.com/download/)。

# 在 Micronaut 框架中集成持久化

要在 Micronaut 框架中展示持久化（数据库）的集成，我们将在`pet-clinic`应用程序中的三个不同的微服务上进行工作：

![图 2.1 – pet-clinic 应用程序中的微服务![图片](img/Figure_2.1_B16585.jpg)

图 2.1 – pet-clinic 应用程序中的微服务

Hibernate 和 MyBatis 是关系型数据库的持久化框架，而要集成 NoSQL（MongoDB），我们将使用其原生的同步驱动程序。

在以下章节中，我们将通过分别与各自的微服务进行实际操作来介绍每种集成技术。每个微服务（在本章的范围内）将被组件化为以下类型的组件：

+   **实体（Entity）**: 用于封装 ORM

+   **存储库（Repository）**: 用于封装与底层 Hibernate 框架的交互

+   **服务（Service）**: 用于包含任何业务逻辑以及下游存储库的礼宾服务调用

+   **CLI 客户端（CLI client）**: 将**创建-读取-更新-删除**（**CRUD**）请求连接到服务

以下图表描述了这些组件及其相互之间的交互：

![图 2.2 – 微服务组件![图片](img/Figure_2.2_B16585_Fixed.jpg)

图 2.2 – 微服务组件

我们将遵循服务-存储库模式来分离关注点并在微服务内部解耦组件。我们将自下而上地介绍这些组件，首先从实体开始，然后是存储库，最后是服务。在下一节中，我们将探讨使用 ORM 框架集成关系型数据库。

# 使用 ORM（Hibernate）框架集成关系型数据库

一个 ORM 框架使您能够使用面向对象范式存储、查询或操作数据。它提供了一种面向对象的方法来访问数据库中的数据，换句话说，您可以使用 Java 对象与数据库交互，而不是使用 SQL。

在 Java 中，作为一个标准规范，**Java 持久化 API**（**JPA**）规定了以下内容：

+   应该持久化哪些 Java 对象

+   这些对象应该如何持久化

JPA 不是一个框架或工具，但它规定了标准协议并涵盖了持久化的核心概念以及如何持久化。Hibernate 和 EclipseLink 等各种实现框架已采用这些 JPA 标准。我们将使用**Hibernate**作为我们的 ORM 框架。

要在 Micronaut 框架中亲身体验 Hibernate，我们将工作于小的`pet-clinic`应用程序，并且对于 Hibernate，我们将专注于`pet-owner`微服务。以下图表捕捉了`pet-owner`微服务的模式设计：

![图 2.3 – 宠物所有者模式![图片](img/Figure_2.3_B16585.jpg)

图 2.3 – pet-owner 模式

在本质上，在`pet-owner`模式中，一个所有者可以拥有零个或多个宠物（某种类型的）并且一个宠物可以有零个或多个兽医访问。在下一节中，我们将开始设置`pet-owner`模式。

## 在 PostgreSQL 中生成 pet-owner 模式

要生成`pet-owner`模式，请按照以下说明操作：

1.  从[`github.com/PacktPublishing/Building-Microservices-with-Micronaut/blob/master/Chapter02/micronaut-petclinic/pet-owner/src/main/resources/db/db-sql.txt`](https://github.com/PacktPublishing/Building-Microservices-with-Micronaut/blob/master/Chapter02/micronaut-petclinic/pet-owner/src/main/resources/db/db-sql.txt)下载 DB-SQL。

1.  打开 PostgreSQL 的 PgAdmin 并打开查询工具。

1.  以`pet-owner`用户、模式和表运行前面的 SQL。

1.  最后，运行 SQL 数据将这些表中的某些示例数据导入。

在设置完模式后，我们将专注于在 Micronaut 项目中工作。

## 为 pet-owner 微服务创建 Micronaut 应用程序

为了生成`pet-owner`微服务的样板源代码，我们将使用 Micronaut Launch。Micronaut Launch 是一个直观的界面，用于生成样板，并且可以在[`micronaut.io/launch/`](https://micronaut.io/launch/)访问。一旦打开，此界面将看起来如下截图所示：

![图 2.4 – 使用 Micronaut Launch 生成 pet-owner 项目](img/Figure_2.4_B16585_Fixed.jpg)

图 2.4 – 使用 Micronaut Launch 生成 pet-owner 项目

在 Micronaut Launch 中，我们将选择以下功能（通过点击**功能**按钮）：

+   **data-jpa**

+   **hibernate-jpa**

+   **jdbc-hikari**

+   **logback**

+   **postgres**

在指定上述选项后，点击**生成项目**按钮。系统将下载一个 ZIP 文件。将下载的源代码解压到您的工作区，并在您首选的 IDE 中打开项目。

在`pet-owner`微服务应用程序中，我们将遵循服务-存储库模式来分离关注点并解耦微服务内的组件。如前所述，我们将采取自下而上的方法来涵盖这些组件，从实体开始，然后探索存储库，最后是服务。

## 创建实体类

实体是一个`@Entity`注解。实体类定义通常包含对表中每个列的映射集合。因此，实体对象实例将代表映射表中的一行。

我们将创建一个域包来包含所有实体类。我们将在根包下创建`com.packtpub.micronaut.domain`包。

为了映射所有者表，我们可以定义一个`Owner`实体。让我们从映射基本列开始（跳过外键或任何关系）：

```java
@Entity
@Table(name = "owners", schema = "petowner")
public class Owner implements Serializable {
    private static final long serialVersionUID = 1L;
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(name = "first_name")
    private String firstName;
    @Column(name = "last_name")
    private String lastName;
    @Column(name = "address")
    private String address;
    @Column(name = "city")
    private String city;
    @Column(name = "telephone")
    private String telephone;
}
```

在前面的代码片段中，我们为所有者表声明了一个 `Owner` 实体类。为了映射主键，我们使用 `@Id` 注解与 `@GeneratedValue` 一起使用。你可以为映射的列生成 getter 和 setter。同样，我们可以定义其他实体类：`Pet` 用于宠物表，`Visit` 用于访问表，`PetType` 用于类型表。我们将在下一节中查看定义关系。

## 定义实体之间的关系

使用 Hibernate 框架，我们可以定义以下关系类型：

+   一对一

+   一对多/多对一

+   多对多

让我们看看每种关系类型。

### 映射一对一关系

在 `pet-owner` 示例中，我们没有在任何实体之间定义一对一关系。然而，让我们考虑所有来自所有者表的地址信息已经被提取到一个地址表中；结果架构将如下所示：

![图 2.5 – 所有者和地址之间的一对一关系](img/Figure_2.5_B16585.jpg)

图 2.5 – 所有者和地址之间的一对一关系

实质上，在先前的架构中，一个所有者将有一个地址。

在 `Owner` 实体中，为了定义这个关系，我们可以使用 `@OneToOne`：

```java
@Entity
@Table(name = "owners", schema = "petowner")
public class Owner implements Serializable {
     .// ...
@OneToOne(cascade = CascadeType.ALL)
     @JoinColumn(name = "address_id", referencedColumnName = "id")
     private Address address;
// ... getters and setters
}
```

`@JoinColumn` 将引用连接所有者表中的 `address_id` 列。

而在 `Address` 实体中的这个关系将定义如下：

```java
@Entity
@Table(name = "address")
public class Address {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private Long id;
    //...

    @OneToOne(mappedBy = "address")
    private User user;

    //... getters and setters
}
```

你可能会注意到在 `Owner` 实体中，我们使用 `@JoinColumn` 定义了一个一对一关系，因为所有者表包含 `address_id`。然而，在 `Address` 实体中，我们可以简单地使用 `mappedBy` 并指向在 `Owner` 实体中定义的 `address` 变量。JPA 将在幕后处理这个双向关系。

### 映射一对多/多对一关系

幸运的是，在 `pet-owner` 架构中，我们有许多一对一或多对一关系（多对一只是一对多关系的反转）。为了保持专注，让我们考虑所有者和宠物表之间的以下关系：

![图 2.6 – 所有者和宠物之间的一对多关系](img/Figure_2.6_B16585.jpg)

图 2.6 – 所有者和宠物之间的一对多关系

在 `Owner` 实体中，前面的多对一关系将定义如下：

```java
@Entity
@Table(name = "owners", schema = "petowner")
public class Owner implements Serializable {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    //...

    @OneToMany(mappedBy = "owner", cascade = 
      CascadeType.ALL)
    private Set<Pet> pets = new HashSet<>();
    //...
```

在 `Pet` 实体中，这个关系将映射如下：

```java
@Entity
@Table(name = "pets", schema = "petowner")
public class Pet implements Serializable {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    //...
    @ManyToOne
    @JoinColumn(name = "owner_id")
    private Owner owner;
    //...
}
```

再次，如果你注意到在 `Pet` 实体中，我们使用 `@JoinColumn` 与 `Owner` 定义了一个清晰的关系（因为宠物表包含 `owner_id`），而在 `Owner` 实体中，我们简单地使用了 `mappedBy = "owner"`。JPA 将在幕后处理这个双向关系的定义和管理。

### 映射多对多关系

映射和管理多对多关系稍微复杂一些。在`pet-owner`模式中，我们没有多对多关系的实例，所以让我们假设两个虚构实体`Foo`和`Bar`之间的一种假设关系：

![图 2.7 – `foo`和`bar`之间的多对多关系](img/Figure_2.7_B16585.jpg)

图 2.7 – `foo`和`bar`之间的多对多关系

在`Foo`实体中，上述的多对多关系将被定义为以下内容：

```java
@Entity
@Table(name = "foos")
public class Foo { 

   // ...

    @ManyToMany(cascade = { CascadeType.ALL })
    @JoinTable(
        name = "foo_bars", 
        joinColumns = { @JoinColumn(name = "foo_id") }, 
        inverseJoinColumns = { @JoinColumn(name = "bar_id") }
    )
    Set<Bar> bars = new HashSet<>();

    // getters and setters
}
```

我们使用`@JoinTable`将关系映射到一个多对多表。`joinColumns`指的是实体拥有的列，而`inverseJoinColumns`指的是联合实体中的列。

`Bar`实体将如下定义这种关系：

```java
@Entity
@Table(name = "bars")
public class Bar {    

    // ...  

    @ManyToMany(mappedBy = "bars")
    private Set<Foo> foos = new HashSet<>();

    // getters and setters   
}
```

就像之前的例子一样，我们使用`@JoinTable`在`Foo`实体中清晰地定义了多对多关系，而在`Bar`实体中我们只是简单地使用了`mappedBy`。

到目前为止，我们已经介绍了如何在实体类中定义不同类型的关系。接下来，我们将转向如何创建数据访问仓库。

## 创建数据访问仓库

Hibernate 框架使我们能够非常直观地定义对数据库的 CRUD 访问。对于每个实体，我们将定义一个仓库抽象类，其中每个仓库抽象类将实现`JpaRepository`。`JpaRepository`是`io.micronaut.data.jpa.repository`中定义的一个现成接口，它进一步扩展了`CrudRepository`和`PageableRepository`，以声明和定义支持常见 CRUD 操作的标准方法。这减少了语法糖，并使我们免于自己定义这些方法。

首先，我们将创建一个名为`com.packtpub.micronaut.repository`的新包来包含所有仓库。所有的仓库抽象类看起来都一样，以下是`OwnerRepository`的示例：

```java
@Repository
public abstract class OwnerRepository implements JpaRepository<Owner, Long> {
    @PersistenceContext
    private final EntityManager entityManager;
    public OwnerRepository(EntityManager entityManager) {
        this.entityManager = entityManager;
    }
    @Transactional
    public Owner mergeAndSave(Owner owner) {
        owner = entityManager.merge(owner);
        return save(owner);
    }
}
```

`OwnerRepository`使用 Micronaut 的标准`@Repository`注解，并利用`JpaRepository`来声明和定义`Owner`实体的标准 CRUD 方法。

同样，我们也可以为其他实体定义这些抽象类，例如在`io.micronaut.data.jpa.repository`中的`Pet`、`Visit`和`PetType`。

## 创建实体的服务

服务将包含任何业务逻辑以及到仓库的下层访问。我们可以为每个实体服务定义标准接口，这将概述服务中支持的基本操作。

为了包含所有服务，我们将创建一个名为`com.packtpub.micronaut.service`的包。`OwnerService`接口的声明如下：

```java
public interface OwnerService {
    Owner save(Owner owner);
    Page<Owner> findAll(Pageable pageable);
    Optional<Owner> findOne(Long id);
    void delete(Long id);
}
```

`OwnerService`接口提供了一个所有服务方法的抽象声明。我们可以在一个具体类中实现所有声明的方法：

```java
@Singleton
@Transactional
public class OwnerServiceImpl implements OwnerService {
    private final Logger log = 
     LoggerFactory.getLogger(OwnerServiceImpl.class);
    private final OwnerRepository ownerRepository;
    public OwnerServiceImpl(OwnerRepository 
     ownerRepository) {
        this.ownerRepository = ownerRepository;
    }
    @Override
    public Owner save(Owner owner) {
        log.debug("Request to save Owner : {}", owner);
        return ownerRepository.mergeAndSave(owner);
    }
    @Override
    @ReadOnly
    @Transactional
    public Page<Owner> findAll(Pageable pageable) {
        log.debug("Request to get all Owners");
        return ownerRepository.findAll(pageable);
    }
    @Override
    @ReadOnly
    @Transactional
    public Optional<Owner> findOne(Long id) {
        log.debug("Request to get Owner : {}", id);
        return ownerRepository.findById(id);
    }
    @Override
    public void delete(Long id) {
        log.debug("Request to delete Owner : {}", id);
        ownerRepository.deleteById(id);
    }
}
```

服务方法定义基本上是将执行委托给下游仓库。我们将重复声明服务接口和定义其余实体（`Pet`、`Visit` 和 `PetType`）的具体服务类。

在下一节中，我们将专注于创建一个小型命令行工具，以在 `pet-owner` 数据库中执行常见的 CRUD 操作。

## 执行基本的 CRUD 操作

为了展示实体的基本 CRUD 操作，我们将创建一个简单的实用工具。我们可以在 `com.packtpub.micronaut.utils` 新包中定义 `PetOwnerCliClient`：

```java
@Singleton
public class PetOwnerCliClient {
    private final OwnerService ownerService;
    private final PetService petService;
    private final VisitService visitService;
    private final PetTypeService petTypeService;
    public PetOwnerCliClient(OwnerService ownerService, 
                             PetService petService, 
                             VisitService visitService, 
                             PetTypeService petTypeService) {
        this.ownerService = ownerService;
        this.petService = petService;
        this.visitService = visitService;
        this.petTypeService = petTypeService;
    }
    // methods for performing CRUD operations...
}
```

此实用工具将通过构造函数方法注入所有服务。在此实用工具中进行的任何 CRUD 调用都将使用注入的服务执行。

### 执行读取/检索操作

我们可以在 `PetOwnerCliClient` 中定义一个简单的实用方法，该方法可以调用 `OwnerService` 来检索所有所有者。此外，由于 `Owner` 有多个宠物，每个宠物可以有多个访问记录，因此检索一个所有者将检索 `pet-owner` 模式中的几乎所有内容：

```java
protected void performFindAll() {
    Page<Owner> pOwners = 
     ownerService.findAll(Pageable.unpaged());
    … iterate through paged content
}
```

`performFindAll()` 将检索所有所有者和他们的宠物（包括宠物访问记录）。

### 执行保存操作

要保存一个拥有宠物和访问记录的所有者，我们可以在 `PetOwnerCliClient` 中定义一个方法：

```java
protected Owner performSave() {
    Owner owner = initOwner();
    return ownerService.save(owner);
}
private Owner initOwner() {Owner owner = new Owner();
    owner.setFirstName("Foo");
    owner.setLastName("Bar");
    owner.setCity("Toronto");
    owner.setAddress("404 Adelaide St W");
    owner.setTelephone("647000999");
    Pet pet = new Pet();
    pet.setType(petTypeService.findAll(Pageable.unpaged()).getContent().get(1));
    pet.setName("Baz");
    pet.setBirthDate(LocalDate.of(2010, 12, 12));
    pet.setOwner(owner);
    Visit visit = new Visit();
    visit.setVisitDate(LocalDate.now());
    visit.setDescription("Breathing issues");
    visit.setPet(pet);
    return owner;
}
```

`initOwner()` 初始化一个拥有宠物和宠物访问记录的所有者，它将被 `performSave()` 用于调用下游服务类方法以保存此所有者。

### 执行删除操作

在 `PetOwnerCliClient` 中，我们将定义一个删除方法来删除一个所有者（包括他们的宠物和访问记录）：

```java
protected void performDelete(Owner owner) {
    /** delete owner pets and their visits */
    Set<Pet> pets = owner.getPets();
    if (CollectionUtils.isNotEmpty(pets)) {
        for (Pet pet : pets) {
            Set<Visit> visits = pet.getVisits();
            if (CollectionUtils.isNotEmpty(visits)) {
                for (Visit visit : visits) {
                    visitService.delete(visit.getId());
                }
            }
            petService.delete(pet.getId());
        }
    }
    ownerService.delete(owner.getId());
}
```

`performDelete()` 首先遍历宠物和宠物访问记录；删除宠物访问记录和宠物后，它最终将调用删除所有者的操作。

## 总结

要执行 `PetOwnerCliClient` CRUD 操作，我们将在 `Application.java` 中添加以下代码逻辑：

```java
@Singleton
public class Application {
    private final PetOwnerCliClient petOwnerCliClient;
    public Application(PetOwnerCliClient petOwnerCliClient) {
        this.petOwnerCliClient = petOwnerCliClient;
    }
    public static void main(String[] args) {
        Micronaut.run(Application.class, args);
    }
    @EventListener
    void init(StartupEvent event) {
        petOwnerCliClient.performDatabaseOperations();
    }
}
```

然后，当我们运行应用程序时，上述 `@EventListener` 将调用 `PetOwnerCliClient` 来执行数据库操作。

在本节中，通过 `pet-owner` 微服务，我们介绍了如何将基于 Micronaut 的微服务与关系型数据库集成。我们还讨论了如何定义实体、仓库和服务，最后展示了 CRUD 操作。在下一节中，我们将探讨如何使用另一种持久化框架（MyBatis）与关系型数据库集成。

# 使用持久化（MyBatis）框架与关系型数据库集成

**MyBatis** 是一个 Java 持久化框架。与 Hibernate（一个 ORM 框架）不同，MyBatis 不支持直接将 Java 对象映射到数据库，而是将 Java 方法映射到 SQL 语句。

MyBatis 通常用于迁移或转换项目中，其中已经存在遗留数据库（s）。由于许多表、视图和其他数据对象已经在数据库中定义并使用，因此将这些表/视图定义重构和规范化以直接映射到 Java 对象（使用 ORM 框架）可能不是一个理想的情况。MyBatis 提供了一种将 Java 方法映射到 SQL 语句的理想方式。这些管理任何 CRUD 访问的 SQL 语句，使用 MyBatis 注解定义在 XML 映射器或 POJO 映射器中。

此外，由于 ORM 框架（如 Hibernate）会自行管理子实体并完全隐藏 SQL 部分，一些开发者更喜欢控制与 SQL 的交互。因此，MyBatis 可以作为首选的持久化框架介入。

Micronaut 支持通过 MyBatis 与关系数据库集成。为了展示这种集成，我们将工作在另一个微服务上，该微服务将管理宠物诊所应用程序的兽医方面。此微服务将集成以下架构：

![Figure 2.8 – Pet clinic schema](img/Figure_2.8_B16585.jpg)

图 2.8 – 宠物诊所架构

实际上，一位兽医可以有许多专长，而一种专长可以属于多位兽医。在下一节中，我们将开始设置`pet-clinic`架构。

## 在 PostgreSQL 中生成 pet-clinic 架构

要生成`pet-clinic`架构，请遵循以下说明：

1.  从[`github.com/PacktPublishing/Building-Microservices-with-Micronaut/blob/master/Chapter02/micronaut-petclinic/pet-clinic/src/main/resources/db/db-sql.txt`](https://github.com/PacktPublishing/Building-Microservices-with-Micronaut/blob/master/Chapter02/micronaut-petclinic/pet-clinic/src/main/resources/db/db-sql.txt)下载 DB-SQL。

1.  打开 PostgreSQL 的 PgAdmin 并打开查询工具。

1.  以`pet-owner`用户、架构和表运行前面的 SQL。

1.  最后，运行数据 SQL 将这些示例数据导入这些表中。

在设置架构完成后，我们将把注意力转向 Micronaut 项目。

## 为 pet-clinic 微服务生成 Micronaut 应用程序

为了生成`pet-clinic`微服务的样板源代码，我们将使用 Micronaut Launch：

![Figure 2.9 – Using Micronaut Launch for generating the pet-clinic project](img/Figure_2.9_B16585_Fixed.jpg)

图 2.9 – 使用 Micronaut Launch 生成 pet-clinic 项目

在 Micronaut Launch 中，我们将选择以下功能（通过点击**功能**按钮）：

+   **jdbc-hikari**

+   **logback**

+   **postgres**

在指定上述选项后，点击**生成项目**按钮。一个 ZIP 文件将被下载到您的系统上。

将下载的源代码解压到您的工作区，并在您首选的 IDE 中打开项目。一旦项目在 IDE 中打开，请在`pom.xml`（或`gradle build`）中添加以下依赖项以用于 MyBatis：

```java
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
     <groupId>org.mybatis</groupId>
     <artifactId>mybatis</artifactId>
     <version>3.5.5</version>
</dependency>
```

这个依赖对于我们在 Micronaut 中与 `pet-clinic` 模式集成的核心。

与服务-仓库模式一致，我们将从下往上探索 MyBatis 集成。首先，我们将定义实体，然后是仓库，最后我们将处理服务。

## 定义 MyBatis 工厂

为了执行各种 SQL 语句，MyBatis 在运行时需要一个 `SqlSessionFactory` 对象。我们将首先添加一个包 - `com.packtpub.micronaut.config`。向这个新创建的包中添加以下类：

```java
@Factory
public class MybatisFactory {
    private final DataSource dataSource;
    public MybatisFactory(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    @Context
    public SqlSessionFactory sqlSessionFactory() {
        TransactionFactory transactionFactory = new 
         JdbcTransactionFactory();
        Environment environment = new Environment(
         "pet-clinic", transactionFactory, dataSource);
        Configuration configuration = new 
         Configuration(environment);
        configuration.addMappers("com.packtpub.micronaut.repository");
        return new 
         SqlSessionFactoryBuilder().build(configuration);
    }
}
```

使用 `application.yml` 中的标准属性，Micronaut 将定义一个基于 Hikari 的数据源，该数据源将被注入以定义 `SqlSessionFactory`。在定义环境时，你可以选择任何名称（正如我们给出的 `pet-clinic`）。

## 创建实体类

与 Hibernate 实体类似，MyBatis 实体本质上定义了一个 Java 类，以将上游 Java 类与下游 SQL 交互（在 XML 或 Java 映射器中定义）集成。然而，一个细微的区别是，任何 MyBatis 实体都不会包含任何映射逻辑。

我们将添加一个 `com.packtpub.micronaut.domain` 包来包含域实体。添加一个表示 `Specialty` 的实体：

```java
@Introspected
public class Specialty implements Serializable {
    private static final long serialVersionUID = 1L;
    @NotNull
    private Long id;
    private String name;
    // ... getters and setters
}
```

同样，我们可以为兽医表定义一个实体：

```java
@Introspected
public class Vet implements Serializable {
    private static final long serialVersionUID = 1L;
    @NotNull
    private Long id;
    private String firstName;
    private String lastName;
    private Set<Specialty> specialties = new HashSet<>();
    // ... getters and setters
}
```

你可以注意到这两个实体都没有任何映射逻辑来映射到特殊或兽医表。在下一节中，我们将关注如何在 MyBatis 中创建数据访问仓库。

## 定义实体的映射器（仓库）

对于 `Vet` 和 `Specialty` 实体，我们需要定义 MyBatis 映射器。MyBatis 通过这些映射器与下游数据库交互。与典型的 Java 应用程序相比，MyBatis 映射器是数据访问仓库。我们将添加一个 `com.packtpub.micronaut.repository` 包来包含所有这些仓库。

在此包下，我们将添加 `SpecialtyRepository` 接口：

```java
/**
 * Mybatis mapper for {@link Specialty}.
 */
public interface SpecialtyRepository {
    @Select("SELECT * FROM petclinic.specialties")
    Collection<Specialty> findAll() throws Exception;
    @Select("SELECT * FROM petclinic.specialties WHERE id =
       #{id}")
    Specialty findById(@Param("id") Long id) throws 
     Exception;
    @Select("SELECT * FROM petclinic.specialties WHERE 
      UPPER(name) = #{name}")
    Specialty findByName(@Param("name") String name) throws 
     Exception;
    @Select(
        {
            "INSERT INTO petclinic.specialties(id, name)\n" +
                "VALUES (COALESCE(#{id}, (select 
                 nextval('petclinic.specialties_id_seq'))),
                 #{name})\n" +
                "ON CONFLICT (id)\n" +
                "DO UPDATE SET name = #{name}  \n" +
                "WHERE petclinic.specialties.id = #{id}\n" +
                "RETURNING id"
        }
    )
    @Options(flushCache = Options.FlushCachePolicy.TRUE)
    Long save(Specialty specialty) throws Exception;
    @Delete("DELETE FROM petclinic.specialties WHERE id = 
     #{id}")
    void deleteById(@Param("id") Long id) throws Exception;
    @Select({
        "SELECT DISTINCT id, name FROM 
          petclinic.specialties WHERE id IN(",
        "SELECT specialty_id FROM petclinic.vet_specialties 
          WHERE vet_id = #{vetId}",
        ")"
    })
    Set<Specialty> findByVetId(@Param("vetId") Long vetId) 
     throws Exception;
}
```

在前面的代码片段中，我们可以看到所有的实际 SQL 语句，然后绑定到 Java 方法。因此，每当任何上游调用者调用上述任何方法时，MyBatis 将执行相应的映射 SQL。同样，我们将定义 `VetRepository` 来管理对兽医表的访问。

与 Hibernate（为抽象仓库类提供具体实现）不同，在 MyBatis 中，我们必须为仓库提供具体实现。我们将将这些实现添加到 `com.packtpub.micronaut.repository.impl`。

`SpecialtyRepository` 的具体实现可以定义如下：

```java
@Singleton
public class SpecialtyRepositoryImpl implements SpecialtyRepository {
    ... SqlSessionFactory injection …
    private SpecialtyRepository 
     getSpecialtyRepository(SqlSession sqlSession) {
        return 
         sqlSession.getMapper(SpecialtyRepository.class);
    }
    @Override
    public Collection<Specialty> findAll() throws Exception {
        try (SqlSession sqlSession = 
         sqlSessionFactory.openSession()) {
            return 
             getSpecialtyRepository(sqlSession).findAll();
        }
    }
    @Override
    public Specialty findById(Long id) throws Exception {
        try (SqlSession sqlSession = 
          sqlSessionFactory.openSession()) {
            return getSpecialtyRepository
             (sqlSession).findById(id);
        }
    }
    @Override
    public Specialty findByName(String name) throws 
      Exception {
        try (SqlSession sqlSession = 
         sqlSessionFactory.openSession()) {
            return getSpecialtyRepository
            (sqlSession).findByName(name);
        }
    }
    @Override
    public Long save(Specialty specialty) throws Exception {
        Long specialtyId;
        try (SqlSession sqlSession = 
         sqlSessionFactory.openSession()) {
            specialtyId = getSpecialtyRepository
             (sqlSession).save(specialty);
            sqlSession.commit();
        }
        return specialtyId;
    }
    @Override
    public void deleteById(Long id) throws Exception {
        try (SqlSession sqlSession = 
         sqlSessionFactory.openSession()) {
            getSpecialtyRepository(sqlSession).deleteById
              (id);
            sqlSession.commit();
        }
    }
    @Override
    public Set<Specialty> findByVetId(Long vetId) throws 
     Exception {
        try (SqlSession sqlSession = 
         sqlSessionFactory.openSession()) {
            return getSpecialtyRepository
             (sqlSession).findByVetId(vetId);
        }
    }
}
```

所有具体方法定义都使用 `SqlSessionFactory` 获取 `SqlSession` 实例。然后 `getSpecialtyRepository()` 方法将返回使用此 `SqlSession` 实例的 MyBatis 映射器。同样，可以定义 `VetRepositoryImpl` 来为 `VetRepository` 提供具体实现。

在下一节中，我们将为这些我们刚刚定义的仓库创建上游服务类。

## 创建实体服务

服务将包含任何业务逻辑以及到先前存储库的下游访问。我们可以为每个实体服务定义标准接口，这将概述服务中支持的基本操作。

为了包含所有服务，首先，我们将创建一个名为`com.packtpub.micronaut.service`的包。我们可以为`SpecialtyService`声明一个接口以抽象我们的基本结构：

```java
public interface SpecialtyService {
    Specialty save(Specialty specialty) throws Exception;
    Collection<Specialty> findAll() throws Exception;
    Optional<Specialty> findOne(Long id) throws Exception;
    void delete(Long id) throws Exception;
}
```

对于这些基本方法，我们需要提供具体实现。

我们可以在服务包下添加一个名为`com.packtpub.micronaut.service.impl`的包。`SpecialtyService`的具体实现将定义如下：

```java
@Singleton
public class SpecialtyServiceImpl implements SpecialtyService {
    private final Logger log = 
     LoggerFactory.getLogger(SpecialtyServiceImpl.class);
    private final SpecialtyRepository specialtyRepository;
    public SpecialtyServiceImpl(SpecialtyRepository 
      specialtyRepository) {
        this.specialtyRepository = specialtyRepository;
    }
    @Override
    public Specialty save(Specialty specialty) throws 
       Exception {
        log.debug("Request to save Specialty : {}", 
         specialty);
        Long specialtyId = 
         specialtyRepository.save(specialty);
        return specialtyRepository.findById(specialtyId);
    }
    @Override
    public Collection<Specialty> findAll() throws Exception {
        log.debug("Request to get all Specialties");
        return specialtyRepository.findAll();
    }
    @Override
    public Optional<Specialty> findOne(Long id) throws 
      Exception {
        log.debug("Request to get Specialty : {}", id);
        return Optional.ofNullable
         (specialtyRepository.findById(id));
    }
    @Override
    public void delete(Long id) throws Exception {
        log.debug("Request to delete Specialty : {}", id);
        specialtyRepository.deleteById(id);
    }
}
```

具体服务方法本质上是简单的，并代理对下游存储库方法的调用。我们将为`Vet`添加一个类似的服务接口和具体实现，如`VetService`和`VetServiceImpl`。

在下一节中，我们将专注于创建一个小型命令行实用程序，以在`pet-clinic`数据库中执行常见的 CRUD 操作。

## 执行基本的 CRUD 操作

要使用上一节中定义的服务和存储库，并执行实体的基本 CRUD 操作，我们可以创建一个简单的实用程序。我们可以在`com.packtpub.micronaut.utils`包下创建一个新的包来定义`PetClinicCliClient`：

```java
@Singleton
public class PetClinicCliClient {
    private final VetService vetService;
    private final SpecialtyService specialtyService;
    public PetClinicCliClient(VetService vetService, 
                              SpecialtyService 
                              specialtyService) {
        this.vetService = vetService;
        this.specialtyService = specialtyService;
    }
    // …
}
```

`PetClinicCliClient`使用构造方法注入`VetService`和`SpecialtyService`。这些服务将用于对`vets`、`specialties`和`vet_specialties`表执行各种数据库操作。

### 执行读取/检索操作

我们可以在`PetClinicCliClient`中定义一个简单的实用方法，该方法可以调用`VetService`来检索所有兽医。此外，由于一个兽医可以有多个专业，检索一个兽医将同时从专业表中检索：

```java
protected void performFindAll() {
    List<Vet> vets;
    try {
        vets = (List<Vet>) vetService.findAll();
        … iterate on vets
    } catch (Exception e) {
        log.error("Exception: {}", e.toString());
    }
}
```

`performFindAll()`方法从数据库中检索所有兽医及其专业。

### 执行保存操作

我们将在`PetClinicCliClient`中保存一个具有专业的兽医：

```java
protected Vet performSave() {
    Vet vet = initVet();
    Vet savedVet = null;
    try {
        savedVet = vetService.save(vet);
    } catch (Exception e) {
        log.error("Exception: {}", e.toString());
    }
    return savedVet;
}
private Vet initVet() {
    Vet vet = new Vet();
    vet.setFirstName("Foo");
    vet.setLastName("Bar");
    Specialty specialty = new Specialty();
    specialty.setName("Baz");
    vet.getSpecialties().add(specialty);
    return vet;
}
```

`initVet()`方法初始化一个新的具有专业的兽医，然后由`performSave()`方法用来将此对象持久化到数据库表中。

### 执行删除操作

在`PetClinicCliClient`中，我们将定义一个删除方法来删除一个兽医（及其专业）：

```java
protected void performDelete(Vet vet) {
    try {
        vetService.delete(vet.getId());
    } catch (Exception e) {
        log.error("Exception: {}", e.toString());
    }
}
```

`performDelete()`方法将调用委托给`VetService`，然后`VetService`调用存储库，最终从数据库中删除兽医。

## 总结

为了执行`PetClinicCliClient`的 CRUD 操作，我们将向`Application.java`添加以下代码逻辑：

```java
@Singleton
public class Application {
    private final PetClinicCliClient petClinicCliClient;
    public Application(PetClinicCliClient 
     petClinicCliClient) {
        this.petClinicCliClient = petClinicCliClient;
    }
    public static void main(String[] args) {
        Micronaut.run(Application.class, args);
    }
    @EventListener
    void init(StartupEvent event) {
        petClinicCliClient.performDatabaseOperations();
    }
}
```

当我们运行我们的应用程序时，上述的`@EventListener`将调用`PetClinicCliClient`来执行各种数据库操作。

在本节中，我们使用 MyBatis 与关系型数据库进行了集成。我们介绍了如何定义实体、存储库和服务，最后通过一个实用程序展示了基本的 CRUD 操作。在下一节中，我们将探索在 Micronaut 框架中与 NoSQL 数据库的集成。

# 与 NoSQL 数据库（MongoDB）集成

**MongoDB** 是一个基于文档的数据库，它以 JSON 或 BSON 格式存储数据。数据以键值对的形式存储，类似于 JSON 对象。MongoDB 采用横向扩展的设计，当数据量和结构敏捷且快速增长时，建议使用它。与关系型数据库相比，有几个关键术语：

+   **数据库**: MongoDB 中的数据库与关系型数据库中的数据库非常相似。

+   **表**: 一个（文档的）集合与关系型数据库中的表非常相似。

+   **行**: 一个 BSON 或 JSON 文档将类似于关系型数据库中的一行。

为了进行实际操作，我们将继续使用 `pet-clinic` 应用程序并添加一个新的微服务，即 `pet-clinic-reviews`。这个微服务将负责管理兽医评论。由于评论可能会快速增长，存储评论的模式可能会改变，因此我们更倾向于将此数据存储在 MongoDB 中：

```java
{ 
    "_id" : ObjectId("5f485523d0cfa84e00963fe4"), 
    "reviewId" : "0ee19b1c-ec8e-11ea-adc1-0242ac120002", 
    "rating" : 4.0, 
    "comment" : "Good vet in the area", 
    "vetId" : 1.0, 
    "dateAdded" : ISODate("2020-08-28T00:51:47.922+0000")
}
```

前面的 JSON 文档描述了一个存储在 MongoDB 中的评论。

在下一节中，我们将开始设置 MongoDB 中的 `pet-clinic-reviews` 模式。

## 在 MongoDB 中创建 vet-reviews 集合

对于 `pet-clinic-reviews` 微服务，我们将在 MongoDB 实例中创建一个数据库和集合：

1.  打开 Studio 3T 或任何其他首选的 MongoDB GUI。

1.  连接到您的本地实例或任何其他首选实例（例如 MongoDB Atlas）。

1.  在 `pet-clinic-reviews` 下。

1.  在上述数据库中，我们将创建一个新的集合：`vet-reviews`。

1.  要将一些示例数据导入此集合，请使用导入选项从 [`github.com/PacktPublishing/Building-Microservices-with-Micronaut/tree/master/Chapter02/micronaut-petclinic/pet-clinic-reviews/src/main/resources/db`](https://github.com/PacktPublishing/Building-Microservices-with-Micronaut/tree/master/Chapter02/micronaut-petclinic/pet-clinic-reviews/src/main/resources/db) 导入数据（CSV 或 JSON）。

在设置完模式后，我们将把注意力转向处理 Micronaut 项目。

## 为 pet-clinic-reviews 微服务生成 Micronaut 应用程序

为了生成 `pet-clinic-reviews` 微服务的样板代码，我们将使用 Micronaut Launch。它是一个直观的界面，用于生成样板代码：

![Figure 2.10 – 使用 Micronaut Launch 生成 pet-clinic-reviews 项目](img/Figure_2.10_B16585_Fixed.jpg)

图 2.10 – 使用 Micronaut Launch 生成 pet-clinic-reviews 项目

在 Micronaut Launch 中，我们将选择以下功能（通过点击 **功能** 按钮）：

+   **mongodb-sync**（对 MongoDB 的同步访问）

+   **logback**

在指定上述选项后，点击 **生成项目** 按钮。系统将下载一个 ZIP 文件。将下载的源代码解压到您的工作区，并在您首选的 IDE 中打开项目。

在接下来的章节中，我们将深入了解如何将 `pet-clinic-reviews` 微服务与 MongoDB 集成。

## 在 Micronaut 中配置 MongoDB

在新创建的 `pet-clinic-reviews` 项目中，我们需要更新 `application.yml` 中的 URI 以指向 MongoDB 的正确实例。此外，我们还将定义两个新的自定义属性 - `databaseName` 和 `collectionName`：

```java
micronaut:
  application:
    name: Pet Clinic Reviews
mongodb:
  uri: mongodb://mongodb:mongodb@localhost:27017/pet-clinic-reviews
  databaseName: pet-clinic-reviews
  collectionName: vet-reviews
```

要在整个项目中使用上述数据库和集合，我们可以在一个新包中定义一个配置类 - `com.packtpub.micronaut.config`：

```java
public class MongoConfiguration {
    @Value("${mongodb.databaseName}")
    private String databaseName;
    @Value("${mongodb.collectionName}")
    private String collectionName;
    public String getDatabaseName() {
        return databaseName;
    }
    public String getCollectionName() {
        return collectionName;
    }
}
```

这将使得在整个项目中轻松访问 `databaseName` 和 `collectionName` 属性，我们就不需要使用 `@Value`。此外，由于这些属性是只读的，我们只为这些属性定义了获取器。

接下来，我们将创建一个实体类来定义兽医评论对象。

## 创建实体类

在根级别，添加一个名为 `com.packtpub.micronaut.domain` 的新包。这个包将包含兽医评论的领域/实体。我们可以定义以下 POJO 来表示兽医评论：

```java
public class VetReview {
    private String reviewId;
    private Long vetId;
    private Double rating;
    private String comment;
    private LocalDate dateAdded;
    @BsonCreator
    @JsonCreator
    public VetReview(
            @JsonProperty("reviewId")
            @BsonProperty("reviewId") String reviewId,
            @JsonProperty("vetId")
            @BsonProperty("vetId") Long vetId,
            @JsonProperty("rating")
            @BsonProperty("rating") Double rating,
            @JsonProperty("comment")
            @BsonProperty("comment") String comment,
            @JsonProperty("dateAdded")
            @BsonProperty("dateAdded") LocalDate dateAdded) {
        this.reviewId = reviewId;
        this.vetId = vetId;
        this.rating = rating;
        this.comment = comment;
        this.dateAdded = dateAdded;
    }
    // ... getters and setters
}
```

`@BsonCreator` 和 `@JsonCreator` 通过类构造函数映射 `VetReview` 对象到相应的 BSON 或 JSON 文档。这在从 MongoDB 检索或存储文档并将其映射到 Java 对象时非常有用。

接下来，我们将关注如何为 `VetReview` 创建数据访问仓库。

## 创建数据访问仓库

为了管理对 `vet-reviews` 集合的访问，我们将创建一个仓库。我们可以在 `com.packtpub.micronaut.repository` 中添加这个仓库类。

为了抽象出仓库将公开的所有方法，我们可以声明一个接口：

```java
public interface VetReviewRepository {
    List<VetReview> findAll();
    VetReview findByReviewId(String reviewId);
    VetReview save(VetReview vetReview);
    void delete(String reviewId);
}
```

`VetReviewRepository` 概述了在此仓库中支持的基本操作。

我们需要在 `com.packtpub.micronaut.repository.impl` 包中为 `VetReviewRepository` 接口提供一个具体实现：

```java
@Singleton
public class VetReviewRepositoryImpl implements VetReviewRepository {
    private final MongoClient mongoClient;
    private final MongoConfiguration mongoConfiguration;
    public VetReviewRepositoryImpl(MongoClient mongoClient,
     MongoConfiguration mongoConfiguration) {
        this.mongoClient = mongoClient;
        this.mongoConfiguration = mongoConfiguration;
    }
    @Override
    public List<VetReview> findAll() {
        List<VetReview> vetReviews = new ArrayList<>();
        getCollection().find().forEach(vetReview -> {
            vetReviews.add(vetReview);
        });
        return vetReviews;
    }
    @Override
    public VetReview findByReviewId(String reviewId) {
        return getCollection().find(eq("reviewId", 
         reviewId)).first();
    }
    @Override
    public VetReview save(VetReview vetReview) {
        getCollection().insertOne(vetReview).
         getInsertedId();
        return findByReviewId(vetReview.getReviewId());
    }
    @Override
    public void delete(String reviewId) {
        getCollection().deleteOne(eq("reviewId", 
         reviewId));
    }
    private MongoCollection<VetReview> getCollection() {
        return mongoClient
                .getDatabase(mongoConfiguration.getDatabaseName())
                .getCollection(mongoConfiguration.getCollectionName(), VetReview.class);
    }
}
```

在这个具体类中的 `getCollection()` 方法将从 MongoDB 获取数据库和集合。将数据保存到这个集合或从其中删除是直接了当的，如前所述。要基于某个条件查找，我们可以使用 `find()` 方法配合 `eq()`。`eq()` 是在 MongoDB 驱动程序中定义的一个 BSON 过滤器。

## 创建实体服务

一个服务将封装任何业务逻辑以及到 MongoDB 中 `vet-reviews` 集合（在 MongoDB 中）的下游访问。我们可以在服务中声明一个接口来抽象核心方法。为了包含一个服务，我们首先应该在根包下创建一个名为 `com.packtpub.micronaut.service` 的包。

然后，我们可以声明 `VetReviewService` 接口如下：

```java
public interface VetReviewService {
    List<VetReview> findAll();
    VetReview findByReviewId(String reviewId);
    VetReview save(VetReview vetReview);
    void delete(String reviewId);
}
```

`VetReviewService` 接口将声明由该服务支持的基本方法。为了为这些方法提供具体实现，我们将在 `com.packtpub.micronaut.service.impl` 包下创建一个 `VetServiceImpl` 类：

```java
@Singleton
public class VetReviewServiceImpl implements VetReviewService {
    private final VetReviewRepository vetReviewRepository;
    public VetReviewServiceImpl(VetReviewRepository 
      vetReviewRepository) {
        this.vetReviewRepository = vetReviewRepository;
    }
    @Override
    public List<VetReview> findAll() {
        return vetReviewRepository.findAll();
    }
    @Override
    public VetReview findByReviewId(String reviewId) {
        return 
         vetReviewRepository.findByReviewId(reviewId);
    }
    @Override
    public VetReview save(VetReview vetReview) {
        return vetReviewRepository.save(vetReview);
    }
    @Override
    public void delete(String reviewId) {
        vetReviewRepository.delete(reviewId);
    }
}
```

这些方法的具体实现基本上是将调用委托给 `VetReviewRepository`。然而，为了满足未来的需求，这些方法可以被扩展以包含任何业务逻辑，例如数据验证或数据转换。

在下一节中，我们将专注于创建一个小的命令行工具，以在 `pet-clinic-reviews` 集合上执行常见的 CRUD 操作。

## 执行基本的 CRUD 操作

为了在 `VetReview` 实体上执行基本的 CRUD 操作，我们可以创建一个简单的实用工具。我们可以在 `com.packtpub.micronaut.utils` 这个新包中定义 `PetClinicReviewCliClient`：

```java
@Singleton
@Requires(beans = MongoClient.class)
public class PetClinicReviewCliClient {
    private final Logger log = LoggerFactory.getLogger
     (PetClinicReviewCliClient.class);
    private final VetReviewService vetReviewService;
    public PetClinicReviewCliClient(VetReviewService 
      vetReviewService) {
        this.vetReviewService = vetReviewService;
    }
}
```

此实用工具将通过构造函数注入 `VetReviewService`。我们将通过此服务进行 CRUD 调用。

### 执行读取/获取操作

我们可以在 `PetClinicReviewCliClient` 中定义一个简单的实用方法，该方法可以调用 `VetReviewService` 来获取所有兽医评论或特定的兽医评论：

```java
protected void performFindAll() {
    List<VetReview> vetReviews = 
     this.vetReviewService.findAll();
    … iterate over vetReviews
}
protected void performFindByReviewId(String reviewId) {
    VetReview vetReview = 
     vetReviewService.findByReviewId(reviewId);
    log.info("Review: {}", vetReview);
}
```

`performFindAll()` 将获取集合中的所有兽医评论，而 `performFindByReviewId()` 将通过 `reviewId` 获取特定的评论。

### 执行保存操作

我们将通过 `PetClinicReviewCliClient` 中的方法保存一个兽医评论：

```java
protected VetReview performSave() {
    VetReview vetReview = new VetReview(
            UUID.randomUUID().toString(),
            1L,
            3.5,
            "Good experience",
            LocalDate.now());
    return vetReviewService.save(vetReview);
}
```

虽然我们在 Java 中定义了一个兽医评论对象，但 `VetReview` 实体中的构造函数将负责将此对象映射到 `BsonDocument`，并且 `VetReviewRepository` 中的下游代码将保存此文档到 `vet-reviews` 集合。

### 执行删除操作

在 `PetClinicReviewCliClient` 中，我们将定义一个删除方法来 `delete` 一个兽医评论：

```java
protected void performDelete(VetReview vetReview) {
    vetReviewService.delete(vetReview.getReviewId());
}
```

这是一个小方法，它将调用委托给 `VetReviewService` 以执行删除。`VetReviewService` 进一步调用存储库以从集合中删除指定的兽医评论。

## 总结

在 `Application.java` 中，我们可以将 `PetClinicReviewCliClient` 作为启动事件调用，然后在该 `vet-reviews` 集合上展示基本的 CRUD 操作：

```java
@Singleton
public class Application {
    private final PetClinicReviewCliClient 
     petClinicReviewCliClient;
    public Application(PetClinicReviewCliClient 
     petClinicReviewCliClient) {
        this.petClinicReviewCliClient = 
         petClinicReviewCliClient;
    }
    public static void main(String[] args) {
        Micronaut.run(Application.class, args);
    }
    @EventListener
    void init(StartupEvent event) {
        petClinicReviewCliClient.
         performDatabaseOperations();
    }
}
```

当我们启动应用程序时，上述 `@EventListener` 将调用 `PetClinicReviewCliClient` 来执行数据库操作。

在本节中，我们介绍了如何将 Micronaut 应用程序与 MongoDB 集成。我们定义了实体、存储库和服务以访问 MongoDB 数据库。最后，我们通过一个轻量级实用工具展示了 CRUD 操作。

# 摘要

在本章中，我们探讨了将 Micronaut 应用程序与关系型数据库以及 NoSQL 数据库集成的各个方面。我们探索了不同的持久化集成方式，即使用 ORM（Hibernate）、持久化框架（MyBatis）或基于驱动程序的框架（MongoDB Sync）。在每种技术中，我们都深入探讨了如何定义实体、关系、存储库和服务。每个微服务定义了一个简单的命令行工具来展示常见的 CRUD 操作。

本章为我们提供了覆盖 Micronaut 中几乎所有数据访问方面的技能。在本书的其余部分，我们将通过进行更多动手练习和涵盖 Micronaut 框架的其他方面来进一步探索和使用这些技能和知识。

在下一章中，我们将处理`pet-clinic`应用程序的 Web 层，在所有微服务中定义各种 REST 端点。

# 问题

1.  什么是**对象关系映射**（**ORM**）框架？

1.  什么是 Hibernate？

1.  你如何在 Micronaut 中使用 Hibernate 定义一个实体？

1.  你如何在 Micronaut 中使用 Hibernate 映射一对一关系？

1.  你如何在 Micronaut 中使用 Hibernate 映射一对一或一对多关系？

1.  你如何在 Micronaut 中使用 Hibernate 映射多对多关系？

1.  你如何在 Micronaut 中使用 Hibernate 执行 CRUD 操作？

1.  你如何在 Micronaut 中使用 MyBatis 与关系型数据库集成？

1.  你如何在 Micronaut 中定义 MyBatis 映射器？

1.  你如何在 Micronaut 中使用 MyBatis 执行 CRUD 操作？

1.  你如何在 Micronaut 中与 NoSQL（MongoDB）数据库集成？

1.  你如何在 Micronaut 中在 MongoDB 中执行 CRUD 操作？
