# 4

# 为 API 编写业务逻辑

在上一章中，您使用 OpenAPI 定义了 API 规范。API Java 接口和模型由 OpenAPI（Swagger Codegen）生成。在本章中，您将根据业务逻辑和数据持久化实现 API 的代码。在这里，业务逻辑指的是您为领域功能编写的实际代码，在我们的案例中，这包括电子商务操作，如结账。

您将为实现编写服务和存储库，并添加超媒体和`"_links"`字段。值得注意的是，提供的代码仅包含重要的行，而不是整个文件，以保持简洁。您可以通过代码后的链接查看完整文件。

本章涵盖以下主题：

+   服务设计概述

+   添加存储库组件

+   添加服务组件

+   实现超媒体

+   使用服务和 HATEOAS 增强控制器

+   在 API 响应中添加 ETags

+   测试 API

# 技术要求

要执行本章及以下章节中的指令，您需要任何 REST API 客户端，例如*Insomnia*或*Postman*。

您可以在 GitHub 上找到本章的代码文件，地址为[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter04`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter04)。

# 服务设计概述

我们将实现一个包含四个层——表示层、应用层、领域层和基础设施层的多层架构。多层架构是被称为**领域驱动设计**（**DDD**）的架构风格的基本构建块。让我们简要地看看这些层：

+   **表示层**：这一层代表**用户界面**（**UI**）。在*第七章*，*设计用户界面*中，您将为一个示例电子商务应用开发 UI。

+   **应用层**：应用层包含应用逻辑并维护和协调整个应用流程。提醒一下，它只包含应用逻辑，*不包括*业务逻辑。RESTful Web 服务、异步 API、gRPC API 和 GraphQL API 都是这一层的一部分。

我们已经在*第三章*，*API 规范与实现*中介绍了 REST API 和控制器，它们是应用层的一部分。在前一章中，我们为了演示目的实现了控制器。在本章中，我们将广泛实现一个控制器以服务真实数据。

+   `订单`或`产品`。它负责将这些对象读取/持久化到基础设施层。领域层也包含服务和存储库。我们也会在本章中介绍这些内容。

+   **基础设施层**：基础设施层为所有其他层提供支持。它负责通信，例如与数据库、消息代理和文件系统的交互。Spring Boot 作为基础设施层，为与外部和内部系统（如数据库和消息代理）的通信和交互提供支持。

我们将采用自下而上的方法。让我们从使用`@Repository`组件实现领域层开始。

# 添加一个仓库组件

我们将采用自下而上的方法来添加`@Repository`组件。让我们从使用`@Repository`组件实现领域层开始。我们将在后续章节中相应地实现服务和增强`@Controller`组件。我们首先将实现`@Repository`组件，然后在`@Service`组件中使用构造函数注入使用它。`@Controller`组件将通过`@Service`组件进行增强，该组件也将通过构造函数注入到控制器中。

## `@Repository`注解

仓库组件是带有`@Repository`注解的 Java 类。这是一个特殊的 Spring 组件，用于与数据库交互。

`@Repository`是一个通用 stereotypes，代表 DDD 的 Repository 和 `@Repository`。

你将使用以下库作为数据库依赖项：

+   **用于持久化数据的 H2 数据库**：我们将使用 H2 的内存实例；然而，你也可以使用基于文件的实例

+   **Hibernate 对象关系映射（ORM）**：用于数据库对象映射

+   **Flyway 数据库迁移**：这有助于维护数据库，并保持数据库变更历史记录，允许回滚、版本升级等操作

让我们将这些依赖项添加到`build.gradle`文件中。`org.springframework.boot:spring-boot-starter-data-jpa`添加了所有必要的 JPA 依赖项，包括 Hibernate：

```java
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'implementation 'org.flywaydb:flyway-core'
runtimeOnly    'com.h2database:h2'
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter04/build.gradle`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter04/build.gradle)

添加依赖项后，我们可以添加与数据库相关的配置。

## 配置数据库和 JPA

我们还需要修改以下配置的`application.properties`文件。配置文件可在[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter04/src/main/resources/application.properties`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter04/src/main/resources/application.properties)找到：

+   **数据源配置**：以下为 Spring 数据源配置：

    ```java
    spring.datasource.name=ecommspring.datasource.url=jdbc:h2:mem:ecomm;DB_CLOSE_DELAY=-1;IGNORECASE=TRUE;DATABASE_TO_UPPER=falsespring.datasource.driverClassName=org.h2.Driverspring.datasource.username=saspring.datasource.password=
    ```

我们需要向数据源添加 H2 特定的属性。URL 值表明将使用基于内存的 H2 数据库实例。

+   **H2 数据库配置**：以下有两个 H2 数据库配置：

    ```java
    spring.h2.console.enabled=truespring.h2.console.settings.web-allow-others=false
    ```

这里，H2 控制台是 H2 网络客户端，允许您在 H2 上执行不同的操作，如查看表和执行查询。H2 控制台仅对本地访问启用；这意味着您只能在本地主机上访问 H2 控制台。此外，通过将`web-allow-others`设置为`false`，禁用了远程访问。

+   **JPA 配置**：以下是一些 JPA/Hibernate 配置：

    ```java
    spring.jpa.properties.hibernate.default_schema=ecommspring.jpa.database-platform=org.hibernate.dialect.H2Dialectspring.jpa.show-sql=truespring.jpa.format_sql=truespring.jpa.generate-ddl=falsespring.jpa.hibernate.ddl-auto=none
    ```

我们不想生成 DDL 或处理 SQL 文件，因为我们想使用 Flyway 进行数据库迁移。因此，`generate-ddl`被标记为`false`，并且`ddl-auto`被设置为`none`。

+   **Flyway 配置**：以下是一些 Flyway 配置：

    ```java
    spring.flyway.url=jdbc:h2:mem:ecommspring.flyway.schemas=ecommspring.flyway.user=saspring.flyway.password=
    ```

在这里，已经设置了 Flyway 连接数据库所需的所有属性。

访问 H2 数据库

您可以使用`/h2-console`访问 H2 数据库控制台。例如，如果您的服务器在本地主机上运行，端口为`8080`，那么您可以通过[`localhost:8080/h2-console/`](http://localhost:8080/h2-console/)访问它。

您已经完成了数据库配置的设置。让我们在下一小节中创建数据库模式和种子数据脚本。

## 数据库和种子数据脚本

现在，我们已经完成了`build.gradle`和`application.properties`文件的配置，我们可以开始编写代码。首先，我们将添加 Flyway 数据库迁移脚本。此脚本只能用 SQL 编写。您可以将此文件放在`src/main/resources`目录内的`db/migration`目录中。我们将遵循 Flyway 命名约定（`V<version>.<name>.sql`），并在`db/migration`目录中创建`V1.0.0__Init.sql`文件。然后，您可以将以下脚本添加到此文件中：

```java
create schema if not exists ecomm;create TABLE IF NOT EXISTS ecomm.product (
   id uuid NOT NULL DEFAULT random_uuid(),
   name varchar(56) NOT NULL,
   description varchar(200),
   price numeric(16, 4) DEFAULT 0 NOT NULL,
   count numeric(8, 0),
   image_url varchar(40),
   PRIMARY KEY(id)
);
create TABLE IF NOT EXISTS ecomm.tag (
   id uuid NOT NULL DEFAULT random_uuid(),
   name varchar(20),
   PRIMARY KEY(id)
);
-- Other code is removed for brevity
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter04/src/main/resources/db/migration/V1.0.0__Init.sql`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter04/src/main/resources/db/migration/V1.0.0__Init.sql)

此脚本创建`ecomm`模式，并添加了我们示例电子商务应用所需的所有表。它还添加了种子数据的`insert`语句。

## 添加实体

现在，我们可以添加实体。实体是一个特殊的对象，它使用 ORM 实现（如*Hibernate*）直接映射到数据库表，并带有`@Entity`注解。另一个流行的 ORM 是*EclipseLink*。您可以将所有实体对象放在`com.packt.modern.api.entity`包中。

让我们创建`CartEntity.java`文件：

```java
@Entity@Table(name = "cart")
public class CartEntity {
 @Id
 @GeneratedValue
 @Column(name = "ID", updatable = false, nullable = false)
 private UUID id;
 @OneToOne
 @JoinColumn(name = "USER_ID", referencedColumnName = "ID")
 private UserEntity user;
 @ManyToMany( cascade = CascadeType.ALL )
 @JoinTable(
  name = "CART_ITEM",
  joinColumns = @JoinColumn(name = "CART_ID"),
  inverseJoinColumns = @JoinColumn(name = "ITEM_ID")
 )
 private List<ItemEntity> items = Collections.emptyList();
 // Getters/Setter and other codes are removed for brevity
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter04/src/main/java/com/packt/modern/api/entity/CartEntity.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/tree/main/Chapter04/src/main/java/com/packt/modern/api/entity/CartEntity.java)

在这里，`@Entity` 注解是 `jakarta.persistence` 包的一部分，表示它是一个实体，应该映射到数据库表。默认情况下，它采用实体名称；然而，我们使用 `@Table` 注解来映射到数据库表。之前，`javax.persistence` 包是 Oracle 的一部分。一旦 Oracle 将 JEE 开源并移交给 Eclipse 基金会，就法律上要求将包名从 `javax.persistence` 更改为 `jakarta.persistence`。

我们还使用一对一和一对多注解将 `Cart` 实体映射到 `User` 实体和 `Item` 实体，分别。`ItemEntity` 列表也与 `@JoinTable` 关联，因为我们使用 `CART_ITEM` 连接表根据它们各自表中的 `CART_ID` 和 `ITEM_ID` 列映射购物车和产品项。

在 `UserEntity` 中，也添加了 `Cart` 实体以维护关系，如下面的代码块所示。`FetchType` 被标记为 `LAZY`，这意味着只有当明确请求时才会加载用户的购物车。此外，如果你希望当购物车没有被用户引用时移除它，可以通过将 `orphanRemoval` 配置为 `true` 来实现：

```java
@Entity@Table(name = "user")
public class UserEntity {
// other code
@OneToOne(mappedBy = "user", fetch = FetchType.LAZY, orphanRemoval = true)
private CartEntity cart;
// other code…
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/entity/UserEntity.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/entity/UserEntity.java)

所有其他实体都添加到位于 [`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/entity`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/entity) 的实体包中。

现在，我们可以添加仓储。

## 添加仓储

所有仓储都已添加到 [`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/repository`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/repository)。

由于 Spring Data JPA 的支持，添加仓储（Repositories）对 CRUD 操作来说非常简单。你只需扩展具有默认实现的接口，例如 `CrudRepository`，它提供了所有 CRUD 操作的实现，如 `save`、`saveAll`、`findById`、`findAll`、`findAllById`、`delete` 和 `deleteById`。`save(Entity e)` 方法用于创建和更新实体操作。

让我们创建 `CartRepository.java`:

```java
public interface CartRepository extends    CrudRepository<CartEntity, UUID> {
  @Query("select c from CartEntity c join c.user u
    where u.id = :customerId")
  public Optional<CartEntity> findByCustomerId(
      @Param("customerId") UUID customerId);
}
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/repository/CartRepository.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/repository/CartRepository.java)

`CartRepository` 接口扩展了 `org.springframework.data.repository` 包中的 `CrudRepository` 部分。您还可以添加带有 `@Query` 注解（`org.springframework.data.jpa.repository` 包的一部分）支持的方法。`@Query` 注解内的查询使用 `CartEntity` 作为表名，而不是 `Cart`。

在 JPQL 中选择列

类似地，对于列，您应该使用类中为字段提供的变量名，而不是使用数据库表字段。在任何情况下，如果您使用数据库表名或字段名，并且它与映射到实际表的类和类成员不匹配，您将得到一个错误。

您可能想知道，“如果我想要添加自己的自定义方法使用 JPQL 或原生态 SQL 会怎样？”好吧，让我告诉您，您也可以这样做。对于订单，我们添加了一个自定义接口来达到这个目的。首先，让我们看看 `OrderRepository`，它与 `CartRepository` 非常相似：

```java
@Repositorypublic interface OrderRepository extends
    CrudRepository<OrderEntity, UUID>, OrderRepositoryExt {
  @Query("select o from OrderEntity o join o.userEntity u
    where u.id = :customerId")
  public Iterable<OrderEntity> findByCustomerId(
      @Param("customerId") UUID customerId);
}
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/repository/OrderRepository.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/repository/OrderRepository.java)

如果您仔细观察，我们会扩展一个额外的接口——`OrderRepositoryExt`。这是我们为 `Order` 存储库提供的额外接口，由以下代码组成：

```java
public interface OrderRepositoryExt {  Optional<OrderEntity> insert(NewOrder m);
}
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/repository/OrderRepositoryExt.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/repository/OrderRepositoryExt.java)

我们已经在 `CrudRepository` 中有一个 `save()` 方法来达到这个目的；然而，我们想要使用不同的实现。为此，并展示您如何创建自己的存储库方法实现，我们添加了这个额外的存储库接口。

现在，让我们创建 `OrderRepositoryExt` 接口实现，如下所示：

```java
@Repository@Transactional
public class OrderRepositoryImpl implements
  OrderRepositoryExt {
  @PersistenceContext
  private EntityManager em;
  private final ItemRepository itemRepo;
  private final CartRepository cRepo;
  private final OrderItemRepository oiRepo;
  public OrderRepositoryImpl(EntityManager em,CartRepository cRepo,       OrderItemRepository oiRepo) {
    this.em = em;
    this.cRepo = cRepo;
    this.oiRepo= oiRepo;
}
// rest of the code
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/repository/OrderRepositoryImpl.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/repository/OrderRepositoryImpl.java)

这样，我们也可以在我们的实现中使用 JPQL/`@Repository`注解告诉 Spring 容器这个特殊组件是一个存储库，应该用于使用底层 JPA 与数据库交互。

它也被标记为 `@Transactional`，这是一个特殊的注解，意味着这个类中的方法执行的事务将由 Spring 管理。它消除了添加提交和回滚的所有手动工作。你还可以将此注解添加到类中的特定方法上。

我们还在`EntityManager`类上使用了`@PersistenceContext`，这允许我们手动创建和执行查询，如下面的代码所示：

```java
@Overridepublic Optional<OrderEntity> insert(NewOrder m) {
 Iterable<ItemEntity> dbItems = itemRepo.findByCustomerId(m.getCustomerId());
 List<ItemEntity> items = StreamSupport.stream(
            dbItems.spliterator(), false).collect
              (toList());
 if (items.size() < 1) {
  throw new ResourceNotFoundException(String.format("There
     is no item found in customer's (ID: %s) cart.",
        m.getCustomerId()));
 }
 BigDecimal total = BigDecimal.ZERO;
 for (ItemEntity i : items) {
   total = (BigDecimal.valueOf(i.getQuantity()).multiply(
      i.getPrice())).add(total);
 }
 Timestamp orderDate = Timestamp.from(Instant.now());
 em.createNativeQuery("""
  INSERT INTO ecomm.orders (address_id, card_id,
    customer_id
  order_date, total, status) VALUES(?, ?, ?, ?, ?, ?)
  """)
 .setParameter(1, m.getAddress().getId())
 .setParameter(2, m.getCard().getId())
 .setParameter(3, m.getCustomerId())
 .setParameter(4, orderDate)
 .setParameter(5, total)
 .setParameter(6, StatusEnum.CREATED.getValue())
 .executeUpdate();
 Optional<CartEntity> oCart =
  cRepo.findByCustomerId(UUID.fromString
   (m. getCustomerId()));
 CartEntity cart = oCart.orElseThrow(() -> new
  ResourceNotFoundException(String.format
("Cart not found for given customer (ID: %s)", m.getCustomerId())));
 itemRepo.deleteCartItemJoinById(cart.getItems().stream()
  .map(i -> i.getId()).collect(toList()), cart. getId());
 OrderEntity entity = (OrderEntity)
    em.createNativeQuery("""
 SELECT o.* FROM ecomm.orders o WHERE o.customer_id = ? AND
 o.order_date >= ?
 """, OrderEntity.class)
 .setParameter(1, m.getCustomerId())
 .setParameter(2, OffsetDateTime.ofInstant(orderDate.
   toInstant(),ZoneId.of("Z")).truncatedTo(ChronoUnit.MICROS))
    .getSingleResult();
 oiRepo.saveAll(cart.getItems().stream()
  .map(i -> new OrderItemEntity().setOrderId
    (entity. getId())
  .setItemId(i.getId())).collect(toList()));
 return Optional.of(entity);
}
```

这个方法基本上首先获取客户购物车中的项目。然后，它计算订单总额，创建一个新的订单，并将其保存到数据库中。接下来，它通过删除映射来从购物车中删除项目，因为购物车项目现在是订单的一部分。之后，它保存订单和购物车项目的映射。

订单创建是通过使用预处理语句的本地 SQL 查询完成的。

如果你仔细观察，你还会发现我们使用了官方的 *Java 15* 功能，**文本** **块** ([`docs.oracle.com/en/java/javase/15/text-blocks/index.html`](https://docs.oracle.com/en/java/javase/15/text-blocks/index.html))。

同样，你可以为所有其他实体创建存储库。所有存储库都可以在[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/repository`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/repository)找到。

现在我们已经创建了存储库，我们可以继续添加服务。

# 添加服务组件

`@Service`组件是一个在控制器和存储库之间工作的接口，我们将在这里添加业务逻辑。尽管你可以直接从控制器中调用存储库，但这不是一种好的做法，因为存储库应该是数据检索和持久化功能的一部分。服务组件还有助于从各种来源获取数据，例如数据库和其他外部应用程序。

服务组件用`@Service`注解标记，这是一个专门的 Spring `@Component`，它允许通过类路径扫描自动检测实现类。服务类用于添加业务逻辑。像`Repository`一样，`Service`对象也代表了 DDD 的 Service 和 JEE 的业务服务外观模式。像`Repository`一样，它也是一个通用目的的构造型，可以根据底层方法使用。

首先，我们将创建服务接口，这是一个包含所有所需方法签名的普通 Java 接口。这个接口将公开`CartService`可以执行的所有操作：

```java
public interface CartService {  public List<Item> addCartItemsByCustomerId(String customerId, @Valid Item item);
  public List<Item> addOrReplaceItemsByCustomerId(String customerId, @Valid Item item);
  public void deleteCart(String customerId);
  public void deleteItemFromCart(String customerId, String itemId);
  public CartEntity getCartByCustomerId(String customerId);
  public List<Item> getCartItemsByCustomerId(String customerId);
  public Item getCartItemsByItemId(String customerId, String itemId);
}
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/service/CartService.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/service/CartService.java)

`CartServiceImpl`类被`@Service`注解，因此它将被自动检测并可用于注入。`CartRepository`、`UserRepository`和`ItemService`类依赖项使用构造函数注入。

让我们看看`CartService`接口的一个更多方法实现。查看以下代码。它添加一个项目，或者如果项目已存在，则更新价格和数量：

```java
@Overridepublic List<Item> addOrReplaceItemsByCustomerId(
  String customerId, @Valid Item item) {
  // 1
  CartEntity entity = getCartByCustomerId(customerId);
  List<ItemEntity> items = Objects.nonNull
    (entity.getItems())
               ? entity.getItems() : Collections.
                 emptyList();
  AtomicBoolean itemExists = new AtomicBoolean(false);
  // 2
  items.forEach(i -> {
    if(i.getProduct().getId().equals(UUID.fromString(
        item.getId()))) {
     i.setQuantity(item.getQuantity()).
       setPrice(i.getPrice());
     itemExists.set(true);
    }
  });
  if (!itemExists.get()) {
    items.add(itemService.toEntity(item));
  }
  // 3
  return itemService.toModelList(
       repository.save(entity).getItems());
}
```

在前面的代码中，我们不是管理应用程序状态，而是在编写查询数据库、设置实体对象、持久化对象，然后返回模型类的业务逻辑。让我们看看前面代码中按编号的语句块：

1.  该方法只有一个`customerId`参数，没有`Cart`参数。因此，首先根据给定的`customerId`从数据库中获取`CartEntity`。

1.  程序控制遍历从`CartEntity`对象检索到的项目。如果给定的项目已经存在，则更改数量和价格。否则，它从给定的`Item`模型创建一个新的`Item`实体，并将其保存到`CartEntity`对象中。`itemExists`标志用于确定我们是否需要更新现有的`Item`或添加一个新的。

1.  最后，将更新的`CartEntity`对象保存到数据库中。从数据库中检索最新的`Item`实体，然后将其转换为模型集合并返回给调用程序。

同样，你可以像为`Cart`实现的那样为其他人编写`Service`组件。在我们开始增强`Controller`类之前，我们需要将一个最终前沿添加到我们的整体功能中。

# 实现超媒体

我们在*第一章*，“RESTful Web 服务基础”中学习了超媒体和 HATEOAS。Spring 使用`org.springframework.boot:` `spring-boot-starter-hateoas`依赖项为 HATEOAS 提供最先进的支持。

首先，我们需要确保 API 响应中返回的所有模型都包含链接字段。将链接（即`org.springframework.hateoas.Link`类）与模型关联的方式有多种，可以是手动或通过自动生成。Spring HATEOAS 的链接和属性是根据*RFC-8288*（[`tools.ietf.org/html/rfc8288`](https://tools.ietf.org/html/rfc8288)）实现的。例如，你可以手动创建一个自链接，如下所示：

```java
import static org.springframework.hateoas.server.mvc. WebMvcLinkBuilder.linkTo;import static org.springframework.hateoas.server.mvc. WebMvcLinkBuilder.methodOn;
// other code blocks…
responseModel.setSelf(linkTo(methodOn(CartController.class)  .getItemsByUserId(userId,item)).withSelfRel())
```

在这里，`responseModel`是一个由 API 返回的模型对象。它有一个名为`_self`的字段，该字段使用`linkTo`和`methodOn`静态方法设置。`linkTo`和`methodOn`方法由 Spring HATEOAS 库提供，允许我们为给定的控制器方法生成一个自链接。

这也可以通过使用 Spring HATEOAS 的`RepresentationModelAssembler`接口自动完成。此接口主要公开了两个方法——`toModel(T model)`和`toCollectionModel(Iterable<? extends T> entities)`——分别将给定的实体/实体转换为模型和`CollectionModel`。

Spring HATEOAS 提供了以下类来丰富用户定义的模型以包含超媒体。它基本上提供了一个包含链接和添加这些链接到模型的方法的类：

+   `RepresentationModel`：模型/DTO 可以扩展此功能以收集链接。

+   `EntityModel`：这扩展了`RepresentationModel`，并在其中使用内容私有字段包装了域对象（即模型）。因此，它包含域模型/DTO 和链接。

+   `CollectionModel`：`CollectionModel`也扩展了`RepresentationModel`。它包装了模型集合并提供了一种维护和存储链接的方式。

+   `PageModel`：`PageModel`扩展了`CollectionModel`，并提供了遍历页面、例如`getNextLink()`和`getPreviousLink()`，以及通过`getTotalPages()`等页面元数据的方法。

使用 Spring HATEOAS 的默认方式是通过扩展`RepresentationModel`与域模型一起使用，如下面的代码片段所示：

```java
public class Cart extends RepresentationModel<Cart>implements Serializable {  private static final long serialVersionUID = 1L;
  @JsonProperty("id")
  private String id;
  @JsonProperty("customerId")
  @JacksonXmlProperty(localName = "customerId")
  private String customerId;Implementing hypermedia 101
  @JsonProperty("items")
  @JacksonXmlProperty(localName = "items")
  @Valid
  private List<Item> items = null;
  // Rest of the code is removed for brevity
```

扩展`RepresentationModel`增强了模型，包括`getLink()`、`hasLink()`和`add()`等附加方法。

你知道所有这些模型都是由 OpenAPI Codegen 生成的；因此，我们需要配置 OpenAPI Codegen 以生成支持超媒体的新模型，这可以通过以下`config.json`文件完成：

```java
{  // …
  "apiPackage": "com.packt.modern.api",
  "invokerPackage": "com.packt.modern.api",
  "serializableModel": true,
  "useTags": true,
  "useGzipFeature": true,
  "hateoas": true,
  "unhandledException": true,
  // …
}
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/resources/api/config.json`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/resources/api/config.json)

添加`hateoas`属性并将其设置为`true`将自动生成扩展`RepresentationModel`类的模型。

我们已经完成了实现 API 业务逻辑的一半。现在，我们需要确保链接将自动填充适当的 URL。为此，我们将扩展`RepresentationModelAssemblerSupport`抽象类，该类内部实现了`RepresentationModelAssembler`。让我们编写`Cart`的汇编器，如下面的代码块所示：

```java
@Componentpublic class CartRepresentationModelAssembler extends
    RepresentationModelAssemblerSupport<CartEntity, Cart> {
  private final ItemService itemService;
  public CartRepresentationModelAssembler(ItemService itemService) {
    super(CartsController.class, Cart.class);
    this.itemService = itemService;
  }
  @Override
  public Cart toModel(CartEntity entity) {
    String uid = Objects.nonNull(entity.getUser()) ?
      entity.getUser().getId().toString() : null;
    String cid = Objects.nonNull(entity.getId()) ?
       entity.getId().toString() : null;
    Cart resource = new Cart();
    BeanUtils.copyProperties(entity, resource);
    resource.id(cid).customerId(uid)
      .items(itemService.toModelList(entity.getItems()));
    resource.add(linkTo(methodOn(CartsController.class)
      .getCartByCustomerId(uid)).withSelfRel());
    resource.add(linkTo(methodOn(CartsController.class)
     .getCartItemsByCustomerId(uid))
        .withRel("cart-items"));
    return resource;
  }
  public List<Cart> toListModel(
     Iterable<CartEntity> entities) {
    if (Objects.isNull(entities)){return List.of();}
    return StreamSupport.stream(
       entities.spliterator(), false).map(this::toModel)
       .collect(toList());
  }
}
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/hateoas/CartRepresentationModelAssembler.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/hateoas/CartRepresentationModelAssembler.java)

在之前的代码中，`Cart` 组装器中的重要部分是扩展 `RepresentationModelAssemblerSupport` 并重写 `toModel()` 方法。如果你仔细观察，你会看到 `CartController.class` 以及 `Cart` 模型也通过 `super()` 调用传递给了 `Rep`。这允许组装器根据之前共享的 `methodOn` 方法生成适当的链接。这样，你可以自动生成链接。

你可能还需要向其他资源控制器添加额外的链接。你可以通过编写一个实现 `RepresentationModelProcessor` 的 bean 并重写 `process()` 方法来实现这一点，如下所示：

```java
@Overridepublic Order process(Order model) {
  model.add(Link.of("/payments/{orderId}").withRel(
    LinkRelation.of("payments")).expand(model.getOrderId()));
  return model;
}
```

你可以始终参考 [`docs.spring.io/spring-hateoas/docs/current/reference/html/`](https://docs.spring.io/spring-hateoas/docs/current/reference/html/) 获取更多信息。

让我们利用在控制器类中创建的服务和 HATEOAS 启用器，在下一节中进行使用。

# 通过服务和 HATEOAS 增强控制器

在 *第三章*，*API 规范和实现* 中，我们创建了 Cart API 的 `Controller` 类——`CartController`——它仅实现了 OpenAPI Codegen 生成的 API 规范接口——`CartApi`。它只是一个没有业务逻辑或数据持久化调用的代码块。

现在，既然我们已经编写了存储库、服务和 HATEOAS 组装器，我们可以增强 API 控制器类，如下所示：

```java
@RestControllerpublic class CartsController implements CartApi {
  private CartService service;
  private final CartRepresentationModelAssembler assembler;
  public CartsController(CartService service,
     CartRepresentationModelAssembler assembler) {
    this.service = service;
    this.assembler = assembler;
  }
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/controller/CartsController.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/controller/CartsController.java)

你可以看到 `CartService` 和 `CartRepresentationModelAssembler` 是通过构造函数注入的。Spring 容器在运行时注入这些依赖项。然后，它们可以像以下代码块中所示那样使用。

```java
@Overridepublic ResponseEntity<Cart> getCartByCustomerId(  String customerId) {
  return ok(assembler.toModel(service.getCartByCustomerId
   (customerId)));
}
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/controller/CartsController.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/controller/CartsController.java)

在前面的代码中，你可以看到服务根据`customerId`检索`Cart`实体（它从内部从存储库检索它）。然后，这个`Cart`实体被转换成一个模型，该模型还包含由 Spring HATEOAS 的`RepresentationModelAssemblerSupport`类提供的超媒体链接。

`ResponseEntity`的`ok()`静态方法用于包装返回的模型，该模型也包含`200` `OK`状态。

这样，你还可以增强和实现其他控制器。现在，我们也可以向我们的 API 响应中添加 ETag。

# 向 API 响应添加 ETag

ETag 是一个包含响应实体计算出的哈希或等效值的 HTTP 响应头，实体中的任何微小变化都必须改变其值。HTTP 请求对象可以包含`If-None-Match`和`If-Match`头部以接收条件响应。

让我们调用一个 API 以获取带有 ETag 的响应，如下所示：

```java
$ curl -v --location --request GET 'http://localhost:8080/ api/v1/products/6d62d909-f957-430e-8689-b5129c0bb75e' –-header 'Content-Type: application/json' --header 'Accept: application/json'* … text trimmed
> GET /api/v1/products/6d62d909-f957-430e-8689-b5129c0bb75e HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.55.1
> Content-Type: application/json
> Accept: application/json
>
< HTTP/1.1 200
< ETag: "098e97de3b61db55286f5f2812785116f"
< Content-Type: application/json
< Content-Length: 339
<
{
  "_links": {
    "self": {
      "href": "http://localhost:8080/6d62d909-f957-430e
              -8689-b5129c0bb75e"
    }
  },
  "id": "6d62d909-f957-430e-8689-b5129c0bb75e",
  "name": "Antifragile",
  "description": "Antifragile - Things …",
  "imageUrl": "/images/Antifragile.jpg",
  "price": 17.1500,
  "count": 33,
  "tag": ["psychology", "book"]
}
```

然后，你可以从 ETag 头部复制值到`If-None-Match`头部，并带有`If-None-Match`头部再次发送相同的请求：

```java
$ curl -v --location --request GET 'http://localhost:8080/ api/v1/products/6d62d909-f957-430e-8689-b5129c0bb75e' --header 'Content-Type: application/json' --header 'Accept: application/json' --header 'If-None-Match: "098e97de3b61db55286f5f2812785116f"'* … text trimmed
> GET /api/v1/products/6d62d909-f957-430e-8689-b5129c0bb75e HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.55.1
> Content-Type: application/json
> Accept: application/jsonAdding ETags to API responses 107
> If-None-Match: "098e97de3b61db55286f5f2812785116f"
>
< HTTP/1.1 304
< ETag: "098e97de3b61db55286f5f2812785116f"
```

你可以看到，由于数据库中的实体没有变化，并且包含相同的实体，它发送了一个`304` (`NOT MODIFIED`)响应，而不是发送带有`200 OK`的正确响应。

实现 ETag 最简单的方法是使用 Spring 的`ShallowEtagHeaderFilter`，如下所示：

```java
@Beanpublic ShallowEtagHeaderFilter shallowEtagHeaderFilter() {
  return new ShallowEtagHeaderFilter();
}
```

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/AppConfig.java`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/src/main/java/com/packt/modern/api/AppConfig.java)

对于此实现，Spring 从写入响应的缓存内容中计算 MD5 哈希。下次当它收到带有`If-None-Match`头部的请求时，它再次从写入响应的缓存内容中创建 MD5 哈希，然后比较这两个哈希。如果两者相同，它发送`304 NOT MODIFIED`响应。这样，它将节省带宽，但仍然需要相同的 CPU 计算。

我们可以使用 HTTP 缓存控制(`org.springframework.http.CacheControl`)类，并使用每次更改时都会更新的版本或类似属性，如果有的话，以避免不必要的 CPU 计算，并更好地处理 ETag，如下所示：

```java
return ResponseEntity.ok()       .cacheControl(CacheControl.maxAge(5, TimeUnit.DAYS))
       .eTag(prodcut.getModifiedDateInEpoch())
       .body(product);
```

向响应添加 ETag 也允许 UI 应用程序确定是否需要刷新页面/对象，或者需要触发事件，尤其是在应用程序中数据频繁变化的地方，例如提供实时比分或股票报价。

现在，你已经实现了完全功能的 API。接下来让我们测试它们。

# 测试 API

现在，你一定期待着测试。你可以在以下位置找到 API 客户端集合，这是一个 HTTP 存档文件，可以由 Insomnia 或 Postman API 客户端使用。你可以导入它，然后测试 API：

[`github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/Chapter04-API-Collection.har`](https://github.com/PacktPublishing/Modern-API-Development-with-Spring-6-and-Spring-Boot-3/blob/main/Chapter04/Chapter04-API-Collection.har)

构建 Chapter 4 代码并运行

您可以通过在项目的根目录中运行 `gradlew clean build` 来构建代码，并使用 `java -jar build/libs/Chapter04-0.0.1-SNAPSHOT.jar` 运行服务。请确保在路径中使用 Java 17。

# 摘要

在本章中，我们学习了使用 Flyway 进行数据库迁移，使用仓库维护和持久化数据，以及将业务逻辑写入服务。我们还学习了如何使用 Spring HATEOAS 组装器自动将超媒体添加到 API 响应中。您现在已经了解了所有 RESTful API 开发实践，这使得您可以在涉及 RESTful API 开发的日常工作中使用这项技能。

到目前为止，我们已经编写了同步 API。在下一章中，您将学习关于异步 API 以及如何使用 Spring 来实现它们。

# 问题

1.  为什么使用 `@Repository` 类？

1.  是否可以向 Swagger 生成的类或模型添加额外的导入或注解？

1.  ETags 有什么用？

# 答案

1.  仓库类使用 `@Repository` 标记，这是一个专门的 `@Component`，使得这些类可以通过包级别的自动扫描来自动检测，并使它们可用于注入。Spring 特别为 DDD 仓库和 JEE DAO 模式提供了这些类。这是应用程序用于与数据库交互的层——作为中心仓库进行检索和持久化。

1.  可以更改模型和 API 生成的方式。您必须复制您想要修改的模板，并将其放置在资源文件夹中。然后，您需要在 `build.gradle` 文件中的 `swaggerSources` 块中添加一个额外的配置参数，以指向模板源，例如 `templateDir = file("${rootDir}/src/main/resources/templates")`。这是您保存修改后的模板，如 `api.mustache` 的地方。这将扩展 OpenAPI 代码生成模板。您可以在 OpenAPI 生成器 JAR 文件中找到所有模板，例如在 `\JavaSpring` 目录中的 `openapi-generator-cli-4.3.1.jar`。您可以将您想要修改的模板复制到 `src/main/resource/templates` 目录中，然后对其进行操作。您可以使用以下资源：

    +   **JavaSpring** **模板**：[`github.com/swagger-api/swagger-codegen/tree/master/modules/swagger-codegen/src/main/resources/JavaSpring`](https://github.com/swagger-api/swagger-codegen/tree/master/modules/swagger-codegen/src/main/resources/JavaSpring)

    +   **Mustache 模板** **变量**：[`github.com/swagger-api/swagger-codegen/wiki/Mustache-Template-Variables`](https://github.com/swagger-api/swagger-codegen/wiki/Mustache-Template-Variables)

    +   **解释实施类似方法的文章**：[`arnoldgalovics.com/swagger-codegen-custom-template/`](https://arnoldgalovics.com/swagger-codegen-custom-template/)

1.  ETags 通过仅在底层 API 响应更新时重新渲染页面/部分，有助于提高 REST/HTTP 客户端性能和用户体验。它们还通过仅在需要时携带响应体来节省带宽。如果 ETag 是基于从数据库检索的值（例如，版本或最后修改日期）生成的，则可以优化 CPU 利用率。

# 进一步阅读

+   Spring HATEOAS：[`docs.spring.io/spring-hateoas/docs/current/reference/html/`](https://docs.spring.io/spring-hateoas/docs/current/reference/html/)

+   RFC-8288：[`tools.ietf.org/html/rfc8288`](https://tools.ietf.org/html/rfc8288)
