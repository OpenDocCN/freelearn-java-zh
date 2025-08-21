# 第三章：使用 Spring Data JPA 构建查询

我们已经学会了如何配置 Spring Data JPA 并实现了一个简单的 CRUD 应用程序。现在是时候学习一些技能，这些技能将帮助我们实现真实的应用程序。在本章中，我们将涵盖：

+   我们如何使用查询方法创建查询

+   我们如何使用 JPA Criteria API 创建动态查询

+   我们如何使用 Querydsl 创建动态查询

+   我们如何对查询结果进行排序和分页

在本章中，我们将通过向联系人管理应用程序添加搜索功能来扩展它。搜索功能的要求如下：

+   搜索功能必须返回所有名字或姓氏以给定搜索词开头的联系人

+   搜索必须不区分大小写

+   搜索结果必须按姓氏和名字按升序排序

+   搜索功能必须能够对搜索结果进行分页

我们还将学习如何对应用程序主页上显示的联系人列表进行排序和分页。

# 构建查询

我们可以使用 Spring Data JPA 构建查询的三种选项：查询方法，JPA Criteria API 和 Querydsl。在本节中，我们将学习如何使用它们并开始实现我们的搜索功能。我们还将看一下每个选项的优缺点，并得到关于选择正确的查询创建技术的具体建议。

在我们继续之前，我们必须向`ContactService`接口添加一个`search()`方法，该方法用作我们搜索功能的起点。`search()`方法的签名如下代码片段所示：

```java
public List<Contact> search(String searchTerm);
```

## 查询方法

使用 Spring Data JPA 创建查询的最简单方法是使用查询方法。**查询方法**是在存储库接口中声明的方法。我们可以使用三种技术来创建查询方法：

+   **从方法名称生成查询**

+   **命名查询**

+   `@Query`注解

### 从方法名称生成查询

从方法名称生成查询是一种查询生成策略，其中执行的查询是从查询方法的名称中解析出来的。用于创建查询方法名称的命名约定有三个重要组件：**方法前缀**，**属性表达式**和**关键字**。接下来，我们将学习这些组件的基本用法并实现我们的搜索功能。我们还将看一下这种方法的优缺点。

#### 方法前缀

每个方法的名称必须以特殊前缀开头。这确保该方法被识别为查询方法。支持的前缀是`findBy`，`find`，`readBy`，`read`，`getBy`和`get`。所有前缀都是同义词，对解析的查询没有影响。

#### 属性表达式

属性表达式用于引用托管实体的直接属性或嵌套属性。我们将使用`Contact`实体来演示以下表中属性表达式的用法：

| 属性表达式 | 引用的属性 |
| --- | --- |
| `LastName` | `Contact`类的`lastName`属性。 |
| `AddressStreetAddress` | `Address`类的`streetAddress`属性。 |

让我们通过使用`AddressStreetAddress`属性表达式来了解属性解析算法是如何工作的。该算法有三个阶段：

1.  首先，它将检查实体类是否具有与属性表达式匹配的名称的属性，当属性表达式的第一个字母转换为小写时。如果找到匹配项，则使用该属性。如果在`Contact`类中找不到名为`addressStreetAddress`的属性，则算法将移至下一个阶段。

1.  属性表达式从右向左按驼峰命名部分分割为头部和尾部。完成后，算法尝试从实体中找到匹配的属性。如果找到匹配，算法会尝试按照属性表达式的部分从头到尾找到引用的属性。在这个阶段，我们的属性表达式被分成两部分：`AddressStreet`和`Address`。由于`Contact`实体没有匹配的属性，算法继续到第三阶段。

1.  分割点向左移动，算法尝试从实体中找到匹配的属性。属性表达式被分成两部分：`Address`和`StreetAddress`。从`Contact`类中找到匹配的属性`address`。此外，由于`Address`类有一个名为`streetAddress`的属性，也找到了匹配。

### 注意

如果`Contact`类有一个名为`addressStreetAddress`的属性，属性选择算法会选择它而不是`Address`类的`streetAddress`属性。我们可以通过在属性表达式中使用下划线字符手动指定遍历点来解决这个问题。在这种情况下，我们应该使用属性表达式`Address_StreetAddress`。

#### 关键词

关键词用于指定针对属性值的约束，这些属性由属性表达式引用。有两条规则用于将属性表达式与关键词组合在一起：

+   我们可以通过在属性表达式后添加关键字来创建**约束**

+   我们可以通过在它们之间添加**And**或**Or**关键字来组合约束

Spring Data JPA 的参考手册（[`static.springsource.org/spring-data/data-jpa/docs/current/reference/html/`](http://static.springsource.org/spring-data/data-jpa/docs/current/reference/html/)）描述了如何使用属性表达式和关键词创建查询方法：

| 关键词 | 示例 | JPQL 片段 |
| --- | --- | --- |
| `And` | `findByLastNameAndFirstName` | `where x.lastname = ?1 and x.firstname = ?2` |
| `Or` | `findByLastNameOrFirstName` | `where x.lastname = ?1 or x.firstname = ?2` |
| `Between` | `findByStartDateBetween` | `where x.startDate between 1? and ?2` |
| `LessThan` | `findByAgeLessThan` | `where x.age < ?1` |
| `GreaterThan` | `findByAgeGreaterThan` | `where x.age > ?1` |
| `After` | `findByStartDateAfter` | `where x.startDate > ?1` |
| `Before` | `findByStartDateBefore` | `where x.startDate < ?1` |
| `IsNull` | `findByAgeIsNull` | `where x.age is null` |
| `IsNotNull`, `NotNull` | `findByAge`(`Is`)`NotNull` | `where x.age is not null` |
| `Like` | `findByFirstNameLike` | `where x.firstname like ?1` |
| `NotLike` | `findByFirstNameNotLike` | `where x.firstname not like ?1` |
| `StartingWith` | `findByFirstNameStartingWith` | `where x.firstname like ?1`（参数绑定为附加`%`） |
| `EndingWith` | `findByFirstNameEndingWith` | `where x.firstname like ?1`（参数绑定为前置`%`） |
| `Containing` | `findByFirstNameContaining` | `where x.firstname like ?1`（参数绑定包裹在`%`中） |
| `OrderBy` | `findByAgeOrderByLastNameDesc` | `where x.age = ?1 order by x.lastname desc` |
| `Not` | `findByLastNameNot` | `where x.lastname <> ?1` |
| `In` | `findByAgeIn`（Collection<Age> ages） | `where x.age in ?1` |
| `NotIn` | `findByAgeNotIn`（Collection<Age> ages） | `where x.age not in ?1` |
| `True` | `findByActiveTrue` | `where x.active = true` |
| `False` | `findByActiveFalse` | `where x.active = false` |

#### 实现搜索功能

现在是时候运用我们学到的技能，为我们的联系人管理应用程序添加搜索功能了。我们可以通过以下步骤来实现搜索功能：

1.  我们按照描述的命名约定向`ContactRepository`接口添加查询方法。

1.  我们实现一个使用查询方法的服务方法。

首先，我们必须创建查询方法。我们的查询方法的签名如下：

```java
public List<Contact> findByFirstNameStartingWithOrLastNameStartingWith(String firstName, String lastName);
```

其次，我们必须将`search()`方法添加到`RepositoryContactService`类中。这个方法简单地将方法调用委托给存储库，并将使用的搜索词作为参数。实现方法的源代码如下：

```java
@Transactional(readOnly = true)
@Override
public List<Contact> search(String searchTerm) {
  return repository.findByFirstNameStartingWithOrLastNameStartingWith(searchTerm, searchTerm);
}
```

#### 优点和缺点

我们现在已经学会了如何使用方法名称策略生成查询。这种策略的优缺点在下表中描述：

| 优点 | 缺点 |
| --- | --- |

|

+   这是创建简单查询的快速方法

+   它为方法名称提供了一致的命名策略

|

+   方法名称解析器的特性决定了我们可以创建什么样的查询

+   复杂查询方法的方法名称又长又难看

+   查询在运行时进行验证

+   不支持动态查询

|

方法名称解析器的限制的一个很好的例子是缺少`Lower`关键字。这意味着我们无法通过使用这种策略来实现不区分大小写的搜索。接下来我们将学习创建不受此限制的查询的替代策略。

### 命名查询

使用 Spring Data JPA 创建查询方法的第二种方法是使用命名查询。如果我们想要使用命名查询创建查询方法，我们必须：

1.  创建一个命名查询。

1.  创建执行命名查询的查询方法。

1.  创建一个使用创建的查询方法的服务方法。

这些步骤在下一节中有更详细的描述。我们还将讨论命名查询的优缺点。

#### 创建命名查询

Spring Data JPA 支持使用 JPQL 或 SQL 创建的命名查询。所使用的查询语言的选择决定了创建的命名查询是如何声明的。

我们可以通过以下步骤创建一个 JPA 命名查询：

1.  将`@NamedQueries`注解添加到实体类中。这个注解以`@NamedQuery`注解的数组作为其值，并且如果我们指定了多个命名查询，必须使用它

1.  我们使用`@NamedQuery`注解来创建命名查询。这个注解有两个对我们有关系的属性：`name`属性存储了命名查询的名称，`query`属性包含了执行的 JPQL 查询。

我们的使用 JPQL 的命名查询的声明如下：

```java
@Entity
@NamedQueries({
@NamedQuery(name = "Contact.findContacts",
        query = "SELECT c FROM Contact c WHERE LOWER(c.firstName) LIKE LOWER(:searchTerm) OR LOWER(c.lastName) LIKE LOWER(:searchTerm)")
})
@Table(name = "contacts")
public class Contact
```

### 注意

我们也可以使用 XML 声明命名查询。在这种情况下，我们必须使用`named-query`元素，并在实体映射 XML 文件中声明查询。

我们可以通过以下步骤创建一个命名的本地查询：

1.  我们将`@NamedNativeQueries`注解添加到实体类中。这个注解接受`@NamedNativeQuery`注解的数组作为其值，并且如果我们指定了多个本地命名查询，必须使用它。

1.  我们通过使用`@NamedNativeQuery`注解来创建本地命名查询。创建的本地命名查询的名称存储在`name`属性中。`query`属性的值是执行的 SQL 查询。`resultClass`属性包含了查询返回的实体类。

### 注意

如果命名本地查询不返回实体或实体列表，我们可以使用`@SqlResultSetMapping`注解将查询结果映射到正确的返回类型。

我们的命名本地查询的声明如下代码片段：

```java
@Entity
@NamedNativeQueries({
@NamedNativeQuery(name = "Contact.findContacts",
        query = "SELECT * FROM contacts c WHERE LOWER(c.first_name) LIKE LOWER(:searchTerm) OR LOWER(c.last_name) LIKE LOWER(:searchTerm)",
        resultClass = Contact.class)
})
@Table(name = "contacts")
public class Contact
```

### 注意

我们也可以使用 XML 创建命名本地查询。在这种情况下，我们必须使用`named-native-query`元素，并在实体映射 XML 文件中声明 SQL 查询。

#### 创建查询方法

我们的下一步是将查询方法添加到联系人存储库中。我们将不得不：

1.  确定查询方法的正确名称。Spring Data JPA 通过假装托管实体的简单名称和方法名称之间的点来将方法名称解析回命名查询。我们的命名查询的名称是`Contact.findContacts`。因此，我们必须在`ContactRepository`接口中添加一个名为`findContacts`的方法。

1.  使用`@Param`注解将方法参数标识为我们查询中使用的命名参数的值。

添加的查询方法的签名如下所示：

```java
public List<Contact> findContacts(@Param("searchTerm") String searchTerm);
```

#### 创建服务方法

接下来，我们必须将`search()`方法添加到`RepositoryContactService`类中。我们的实现包括以下步骤：

1.  构建使用的 like 模式。

1.  通过调用创建的查询方法来获取搜索结果。

`search()`方法的源代码如下：

```java
@Transactional(readOnly = true)
@Override
public List<Contact> search(String searchTerm) {
String likePattern = buildLikePattern(searchTerm);
   return repository.findContacts(likePattern);
}

private String buildLikePattern(String searchTerm) {
   return searchTerm + "%";
}
```

#### 优缺点

现在我们可以使用命名查询来创建查询方法。这种方法的优缺点在下表中描述：

| 优点 | 缺点 |
| --- | --- |

|

+   支持 JPQL 和 SQL

+   使得迁移现有应用程序使用命名查询到 Spring Data JPA 更容易

+   本地查询的返回类型不限于实体或实体列表

|

+   查询验证在运行时完成

+   不支持动态查询

+   查询逻辑使我们的实体类的代码混乱

|

### @Query 注解

`@Query`注解用于指定调用查询方法时执行的查询。我们可以使用`@Query`注解来实现 JPQL 和 SQL 查询：

1.  向存储库添加一个新的方法，并用`@Query`注解进行注释。

1.  创建使用查询方法的服务方法。

### 注意

如果使用`@Query`注解的方法名称与命名查询的名称冲突，则将执行注解的查询。

接下来，我们将得到具体的指导说明，以指导我们完成所描述的步骤，并了解这种技术的优缺点。

#### 创建查询方法

首先我们必须将查询方法添加到`ContactRepository`类中。正如我们已经知道的，我们可以使用 JPQL 或 SQL 来创建实际的查询。使用的查询语言对查询方法的创建有一些影响。

我们可以通过以下方式创建使用 JPQL 的查询方法：

1.  向`ContactRepository`接口添加一个新的方法。

1.  使用`@Param`注解将方法的参数标识为命名参数的值。

1.  用`@Query`注解注释方法，并将执行的 JPQL 查询设置为其值。

我们的查询方法的声明，满足搜索功能的要求，如下所示：

```java
@Query("SELECT c FROM Contact c WHERE LOWER(c.firstName) LIKE LOWER(:searchTerm) OR LOWER(c.lastName) LIKE LOWER(:searchTerm)")
public Page<Contact> findContacts(@Param("searchTerm") String searchTerm);
```

为了创建一个使用 SQL 的查询方法，我们必须：

1.  向`ContactRepository`接口添加一个新的方法。

1.  使用`@Param`注解将方法参数标识为 SQL 查询中使用的命名参数的值。

1.  用`@Query`注解注释创建的方法，并将 SQL 查询设置为其值。将`nativeQuery`属性的值设置为 true。

### 注意

使用`@Query`注解创建的本地查询只能返回实体或实体列表。如果我们需要不同的返回类型，必须使用命名查询，并使用`@SqlResultSetMapping`注解映射查询结果。

实现满足搜索功能要求的查询方法的声明如下代码片段所示：

```java
@Query(value = "SELECT * FROM contacts c WHERE LOWER(c.first_name) LIKE LOWER(:searchTerm) OR LOWER(c.last_name) LIKE LOWER(:searchTerm), nativeQuery = true)
public List<Contact> findContacts(@Param("searchTerm") String searchTerm);
```

### 注意

Spring Data JPA 不支持使用`@Query`注解创建的本地查询的动态排序或分页支持，因为没有可靠的方法来操作 SQL 查询。

#### 创建服务方法

我们的下一步是向`RepositoryContactService`类添加`search()`方法的实现。我们可以通过以下方式实现：

1.  获取使用的 like 模式。

1.  通过调用创建的查询方法来获取搜索结果。

实现的`search()`方法的源代码如下： 

```java
@Transactional(readOnly = true)
@Override
public List<Contact> search(String searchTerm) {
   String likePattern = buildLikePattern(searchTerm);
   return repository.findContacts(likePattern);
}

private String buildLikePattern(String searchTerm) {
   return searchTerm + "%";
}
```

#### 优缺点

我们现在已经学会了如何使用`@Query`注解来创建查询方法。这种方法自然地具有优缺点，如下表所述：

| 优点 | 缺点 |
| --- | --- |

|

+   支持 JPQL 和 SQL

+   方法名称没有命名约定

|

+   本地查询只能返回实体或实体列表

+   不支持动态查询

+   查询验证在运行时完成

|

## JPA Criteria API

JPA Criteria API 为我们提供了以面向对象的方式创建动态和类型安全查询的方法。我们可以通过以下步骤创建**条件查询**：

1.  我们向存储库添加 JPA Criteria API 支持。

1.  我们创建了执行的条件查询。

1.  我们创建了一个执行创建的查询的服务方法。

这些步骤以及使用 JPA Criteria API 的优缺点将在以下部分中描述。

### 将 JPA Criteria API 支持添加到存储库

我们可以通过扩展`JpaSpecificationExecutor<T>`接口向存储库添加 JPA Criteria API 支持。当我们扩展这个接口时，我们必须将受管实体的类型作为类型参数给出。`ContactRepository`接口的源代码如下所示：

```java
public interface ContactRepository extends JpaRepository<Contact, Long>, JpaSpecificationExecutor<Contact> {

}
```

扩展`JpaSpecificationExecutor<T>`接口使我们可以访问以下方法，这些方法可用于执行条件查询：

| 方法 | 描述 |
| --- | --- |
| 返回与给定搜索条件匹配的实体数量。 |
| `List<Contact> findAll(Specification<Contact> s)` | 返回与给定搜索条件匹配的所有实体。 |
| 返回与给定搜索条件匹配的单个联系人。 |

### 创建条件查询

正如我们所学的，Spring Data JPA 使用`Specification<T>`接口来指定条件查询。这个接口声明了`Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder cb)`方法，我们可以使用它来创建执行的条件查询。

为了为`Contact`实体创建条件查询，我们必须：

1.  为`Contact`实体创建一个静态元模型类。

1.  创建构建`Specification<Contact>`对象的方法。

#### 创建静态元模型类

静态元模型类提供对描述实体属性的元数据的静态访问，并用于使用 JPA Criteria API 创建类型安全查询。静态元模型类通常是自动生成的，但在这里，我们将为了示例而手动创建一个。我们可以通过遵循以下规则创建一个静态元模型类：

+   静态元模型类应放置在与相应实体相同的包中

+   静态元模型类的名称是通过在相应实体的简单名称后附加下划线字符来创建的

由于我们在构建条件查询时只使用`Contact`实体的`firstName`和`lastName`属性，我们可以忽略其他属性。`Contact_`类的源代码如下所示：

```java
@StaticMetamodel(Contact.class)
public class Contact_ {
    public static volatile SingularAttribute<Contact, String> firstName;
    public static volatile SingularAttribute<Contact, String> lastName;
}
```

#### 创建规范

我们可以通过创建一个规范构建器类并使用静态方法来构建实际的规范，以清晰的方式创建规范。用于构建所需 like 模式的逻辑也移动到了这个类中。我们规范构建器类的实现在以下步骤中解释：

1.  我们创建了一个`getLikePattern()`方法，用于从搜索词创建 like 模式。

1.  我们创建一个静态的`firstOrLastNameStartsWith()`方法，返回一个新的`Specification<Contact>`对象。

1.  我们在`Specification<Contact>`的`toPredicate()`方法中构建条件查询。

我们的规范构建器类的源代码如下所示：

```java
public class ContactSpecifications {

    public static Specification<Contact> firstOrLastNameStartsWith(final String searchTerm) {
        return new Specification<Contact>() {
        //Creates the search criteria
        @Override
        public Predicate toPredicate(Root<Contact> root, CriteriaQuery<?> criteriaQuery, cb cb) {
            String likePattern = getLikePattern(searchTerm);
            return cb.or(
            //First name starts with given search term
            cb.like(cb.lower(root.<String>get(Contact_.firstName)), likePattern),
            //Last name starts with the given search term

            cb.like(cb.lower(root.<String>get(Contact_.lastName)), likePattern)
                );
            }

      private String getLikePattern(final String searchTerm) {
          return searchTerm.toLowerCase() + "%";
            }
        };
    }
}
```

### 创建服务方法

我们`RepositoryContactService`类的`search()`方法的实现包含以下两个步骤：

1.  我们通过使用我们的规范构建器获得`Specification<Contact>`对象。

1.  我们通过调用存储库的`findAll()`方法并将`Specification<Contact>`对象作为参数传递来获取搜索结果。

我们的实现的源代码如下所示：

```java
@Transactional(readOnly = true)
@Override
public List<Contact> search(String searchTerm) {
Specification<Contact> contactSpec = firstOrLastNameStartsWith(searchTerm);
    return repository.findAll(contactSpec);
}
```

### 优缺点

我们现在已经学会了如何使用 JPA Criteria API 实现动态查询。在我们可以在实际应用程序中使用这些技能之前，我们应该了解这种方法的优缺点。这些在下表中描述：

| 优点 | 缺点 |
| --- | --- |

|

+   支持动态查询

+   语法验证在编译期间完成

+   使得迁移使用 JPA Criteria API 的应用程序到 Spring Data JPA 更容易

|

+   复杂查询难以实现和理解

|

## Querydsl

**Querydsl**是一个框架，通过类似 SQL 的 API 实现类型安全的动态查询的构建（要了解更多关于 Querydsl 的信息，请访问[`www.querydsl.com/`](http://www.querydsl.com/)）。如果我们想使用 Querydsl 创建查询，我们必须：

1.  配置 Querydsl Maven 集成。

1.  生成 Querydsl 查询类型。

1.  向存储库添加 Querydsl 支持。

1.  创建执行的查询。

1.  执行创建的查询。

我们将在下一节中更详细地解释这些步骤，并且我们还将看一下 Querydsl 的优缺点。

### 配置 Querydsl-Maven 集成

Querydsl-Maven 集成的配置包括两个步骤：

1.  我们配置所需的依赖项。

1.  我们配置用于代码生成的 APT Maven 插件。

#### 配置 Querydsl Maven 依赖项

因为我们正在使用 Querydsl 与 JPA，所以必须在`pom.xml`文件中声明以下依赖项：

+   提供 Querydsl 核心，提供 Querydsl 的核心功能

+   Querydsl APT，提供基于 APT 的代码生成支持

+   Querydsl JPA，为 JPA 注解添加支持

我们正在使用 Querydsl 版本 2.8.0。因此，我们必须将以下依赖声明添加到`pom.xml`文件的依赖项部分：

```java
<dependency>
  <groupId>com.mysema.querydsl</groupId>
  <artifactId>querydsl-core</artifactId>
  <version>2.8.0<version>
</dependency>
<dependency>
  <groupId>com.mysema.querydsl</groupId>
  <artifactId>querydsl-apt</artifactId>
  <version>2.8.0</version>
</dependency>
<dependency>
  <groupId>com.mysema.querydsl</groupId>
  <artifactId>querydsl-jpa</artifactId>
  <version>2.8.0</version>
</dependency>
```

#### 配置代码生成 Maven 插件

我们的下一步是配置 Java 6 的注解处理工具的 Maven 插件，用于生成 Querydsl 查询类型。我们可以通过以下方式配置此插件：

1.  配置插件以在 Maven 的`generate-sources`生命周期阶段执行其`process`目标。

1.  指定生成查询类型的目标目录。

1.  配置代码生成器以查找实体类的 JPA 注解。

Maven APT 插件的配置如下：

```java
<plugin>
  <groupId>com.mysema.maven</groupId>
    <artifactId>maven-apt-plugin</artifactId>
  <version>1.0.4</version>
  <executions>
      <execution>
          <phase>generate-sources</phase>
      <goals>
        <goal>process</goal>
      </goals>
      <configuration>
        <outputDirectory>target/generated-sources</outputDirectory>
  <processor>com.mysema.query.apt.jpa.JPAAnnotationProcessor</processor>
      </configuration>
    </execution>
  </executions>
</plugin>
```

### 生成 Querydsl 查询类型

如果我们的配置正常工作，使用 Maven 构建项目时，Querydsl 查询类型应该会自动生成。

### 注意

Maven APT 插件存在一个已知问题，阻止直接从 Eclipse 使用它。Eclipse 用户必须通过在命令提示符下运行命令`mvn generate-sources`来手动创建 Querydsl 查询类型。

查询类型可以从`target/generated-sources`目录中找到。生成的查询类型将适用以下规则：

+   每个查询类型都生成在与相应实体相同的包中。

+   查询类型类的名称是通过将实体类的简单名称附加到字母"`Q`"来构建的。例如，由于我们的实体类的名称是`Contact`，相应的 Querydsl 查询类型的名称是`QContact`。

### 注意

在我们的代码中使用查询类型之前，我们必须将`target/generated-sources`目录添加为项目的源目录。

### 向存储库添加 Querydsl 支持

我们可以通过扩展`QueryDslPredicateExecutor<T>`接口来向存储库添加 Querydsl 支持。当我们扩展此接口时，必须将托管实体的类型作为类型参数给出。`ContactRepository`接口的源代码如下：

```java
public interface ContactRepository extends JpaRepository<Contact, Long>, QueryDslPredicateExecutor<Contact> {
}
```

在我们扩展了`QueryDslPredicateExecutor<T>`接口之后，我们可以访问以下方法：

| 方法 | 描述 |
| --- | --- |
| `long count(Predicate p)` | 返回与给定搜索条件匹配的实体数量。 |
| `Iterable<Contact> findAll(Predicate p)` | 返回与给定搜索条件匹配的所有实体。 |
| `Contact findOne(Predicate p)` | 返回与给定搜索条件匹配的单个实体。 |

### 创建执行的查询

每个查询必须实现 Querydsl 提供的`Predicate`接口。幸运的是，我们不必手动实现这个接口。相反，我们可以使用查询类型来创建实际的查询对象。一个清晰的方法是创建一个特殊的 predicate 构建器类，并使用静态方法来创建实际的 predicates。让我们称这个类为`ContactPredicates`。我们实现了创建满足搜索功能要求的 predicates 的静态方法，如下所述：

1.  我们实现了一个静态的`firstOrLastNameStartsWith()`方法，返回`Predicate`接口的实现。

1.  我们获得了`QContact`查询类型的引用。

1.  我们使用`QContact`查询类型构建我们的查询。

我们的 predicate 构建器类的源代码如下：

```java
public class ContactPredicates {

    public static Predicate firstOrLastNameStartsWith(final String searchTerm) {
        QContact contact = QContact.contact;
        return contact.firstName.startsWithIgnoreCase(searchTerm)
                .or(contact.lastName.startsWithIgnoreCase(searchTerm));
    }
}
```

### 执行创建的查询

我们通过以下方式实现了`RepositoryContactService`类的`search()`方法：

1.  通过调用`ContactPredicates`类的静态`firstOrLastNAmeStartsWith()`方法获取使用的 predicate。

1.  通过调用我们的存储库方法并将 predicate 作为参数传递来获取结果。

1.  使用`Commons Collections`库中的`CollectionUtils`类将每个联系人添加到返回的列表中。

我们的实现源代码如下：

```java
@Transactional(readOnly = true)
@Override
public List<Contact> search(String searchTerm) {
  Predicate contactPredicate = firstOrLastNameStartsWith(searchTerm);

  Iterable<Contact> contacts = repository.findAll(contactPredicate);
  List<Contact> contactList = new ArrayList<Contact>();
  CollectionUtils.addAll(contactList, contacts.iterator());

  return contactList;
}
```

### 优点和缺点

现在我们能够使用 Spring Data JPA 和 Querydsl 创建查询。Querydsl 的优缺点如下表所述：

| 优点 | 缺点 |
| --- | --- |

|

+   支持动态查询

+   清晰易懂的 API

+   语法验证在编译期间完成

|

+   需要代码生成

+   Eclipse 集成工作不正常

|

## 我们应该使用哪种技术？

在本节中，我们已经讨论了使用 Spring Data JPA 创建查询的不同方法。我们也意识到了每种描述的技术的优缺点。这些信息被细化为以下列表中给出的具体指南：

+   我们应该使用查询方法创建静态查询。

+   如果创建的查询简单且方法名解析器支持所需的关键字，我们可以使用方法名策略生成查询。否则，我们应该使用`@Query`注解，因为它灵活，并且不强制我们使用冗长且丑陋的方法名。

+   如果我们无法使用方法策略生成查询或`@Query`注解创建查询方法，命名查询是有用的。这种方法也可以在将现有应用程序迁移到 Spring Data JPA 时使用。然而，当我们创建新应用程序时，应该谨慎使用它们，因为它们倾向于在我们的实体中添加查询逻辑。

+   如果我们无法使用其他描述的技术创建查询，或者需要调整单个查询的性能，原生查询是有用的。然而，我们必须理解使用原生查询会在我们的应用程序和使用的数据库模式之间创建依赖关系。此外，如果我们使用特定于提供程序的 SQL 扩展，我们的应用程序将与使用的数据库提供程序绑定。

+   如果我们正在将使用 criteria 查询的现有应用程序迁移到 Spring Data JPA，应该使用 JPA Criteria API 来创建动态查询。如果我们无法忍受 Querydsl-Eclipse 集成的问题，JPA Criteria API 也是一个有效的选择。

+   Querydsl 是创建动态查询的绝佳选择。它提供了一个清晰易懂的 API，这是 JPA Criteria API 的巨大优势。Querydsl 应该是我们从头开始创建动态查询的首选。笨拙的 Eclipse 集成自然是 Eclipse 用户的缺点。

# 排序查询结果

在本节课程中，我们将学习使用 Spring Data JPA 对查询结果进行排序的不同技术。我们还将学习可以用于为每种情况选择适当排序方法的准则。

## 使用方法名进行排序

如果我们使用从方法名生成查询的策略构建查询，我们可以按照以下步骤对查询结果进行排序：

1.  创建查询方法

1.  修改现有的服务方法以使用新的查询方法。

### 创建查询方法

当我们使用从方法名生成查询的策略构建查询时，我们可以使用`OrderBy`关键字来对查询结果进行排序，当我们：

1.  将`OrderBy`关键字附加到方法名。

1.  将与实体属性对应的属性表达式附加到方法名，用于对查询结果进行排序。

1.  将描述排序顺序的关键字附加到方法名。如果查询结果按升序排序，则应使用关键字`Asc`。当查询结果按降序排序时，使用`Desc`关键字。

1.  如果使用多个属性对查询结果进行排序，则重复步骤 2 和步骤 3。

我们可以通过在查询方法的名称后附加字符串`OrderByLastNameAscFirstNameAsc`来满足搜索功能的新要求。查询方法的签名如下：

```java
public List<Contact> findByFirstNameStartingWithOrLastNameStartingWithOrderByLastNameAscFirstNameAsc(String firstName, String lastName);
```

### 修改服务方法

我们必须修改`RepositoryContactService`类的`search()`方法，以将方法调用委托给新的查询方法。该方法的源代码如下：

```java
@Transactional(readOnly = true)
@Override
public List<Contact> search(String searchTerm) {
    return repository.findByFirstNameStartingWithOrLastNameStartingWithOrderByLastNameAscFirstNameAsc(searchTerm, searchTerm);
}
```

## 使用查询字符串进行排序

在某些情况下，我们必须将排序逻辑添加到实际的查询字符串中。如果我们使用带有`@Query`注解的命名查询或本地查询，我们必须在实际查询中提供排序逻辑。当我们使用带有 JPQL 查询的`@Query`注解时，也可以将排序逻辑添加到实际查询中。

### JPQL 查询

当我们想对 JPQL 查询的查询结果进行排序时，必须使用 JPQL 的`ORDER BY`关键字。满足搜索功能的 JPQL 查询如下所示：

```java
SELECT c FROM Contact c WHERE LOWER(c.firstName) LIKE LOWER(:searchTerm) OR LOWER(c.lastName) LIKE LOWER(:searchTerm) ORDER BY c.lastName ASC, c.firstName ASC
```

### SQL 查询

当我们想对本地 SQL 查询的查询结果进行排序时，必须使用 SQL 的`ORDER BY`关键字。满足搜索功能的 SQL 查询如下所示：

```java
SELECT * FROM contacts c WHERE LOWER(c.first_name) LIKE LOWER(:searchTerm) OR LOWER(c.last_name) LIKE LOWER(:searchTerm) ORDER BY c.last_name ASC, c.first_name ASC
```

## 使用 Sort 类进行排序

如果我们使用`JpaRepository<T,ID>`接口的方法、查询方法或 JPA Criteria API，我们可以使用`Sort`类对查询结果进行排序。如果我们决定使用这种方法，我们必须：

1.  创建`Sort`类的实例。

1.  将创建的实例作为参数传递给所使用的存储库方法。

### 注意

我们不能使用`Sort`类对带有`@Query`注解声明的命名查询或本地查询的查询结果进行排序。

由于后面描述的所有技术都需要获得`Sort`类的实例，我们将不得不为`RepositoryContactService`类添加一种创建这些对象的方法。我们将通过创建一个私有的`sortByLastNameAndFirstNameAsc()`方法来实现这一点。该方法的源代码如下：

```java
private Sort sortByLastNameAndFirstNameAsc() {
  return new Sort(new Sort.Order(Sort.Direction.ASC, "lastName"),
        new Sort.Order(Sort.Direction.ASC, "firstName")
    );
}
```

### JpaRepository

我们使用了`JpaRepository<T,ID>`接口的`findAll()`方法来获取存储在数据库中的所有实体的列表。然而，当我们扩展了`JpaRepository<T,ID>`接口时，我们还可以访问`List<Contact> findAll(Sort sort)`方法，我们可以使用它来对存储在数据库中的实体列表进行排序。

举例来说，我们将按照姓氏和名字的字母顺序对所有实体的列表进行排序。我们可以通过以下方式实现：

1.  获取一个新的`Sort`对象。

1.  通过调用我们的存储库的`findAll()`方法并将创建的`Sort`对象作为参数传递来获取排序后的实体列表。

`RepositoryContactService`的`findAll()`方法的源代码如下：

```java
@Transactional(readOnly = true)
@Override
public List<Contact> findAll() {
  Sort sortSpec = sortByLastNameAndFirstNameAsc();
  return repository.findAll(sortSpec);
}
```

### 从方法名生成查询

我们还可以使用这种方法来对使用方法名称生成查询的查询结果进行排序。如果我们想使用这种技术，我们必须修改查询方法的签名，以接受`Sort`对象作为参数。我们的查询方法的签名，实现了搜索功能的新排序要求，如下所示：

```java
public Page<Contact> findByFirstNameStartingWithOrLastNameStartingWith(String firstName, String lastName, Sort sort);
```

我们的下一步是更改`RepositoryContactService`类的`search()`方法的实现。新的实现在以下步骤中解释：

1.  我们获得一个`Sort`对象的引用。

1.  我们调用我们的新存储库方法并提供所需的参数。

我们实现的源代码如下所示：

```java
@Transactional(readOnly = true)
@Override
public List<Contact> search(String searchTerm) {
  Sort sortSpec = sortByLastNameAndFirstNameAsc();
  return repository.findByFirstNameStartingWithOrLastNameStartingWith(searchTerm, searchTerm, sortSpec);
}
```

### @Query 注解

如果我们使用`@Query`注解来使用 JPQL 构建查询，我们不必将排序逻辑添加到实际查询中。我们还可以修改查询方法的签名，以接受`Sort`对象作为参数。我们的查询方法的声明如下所示：

```java
@Query("SELECT c FROM Contact c WHERE LOWER(c.firstName) LIKE LOWER(:searchTerm) OR LOWER(c.lastName) LIKE LOWER(:searchTerm)")
public Page<Contact> findContacts(@Param("searchTerm") String searchTerm, Sort sort);
```

下一步是修改`RepositoryContactService`类的`search()`方法。我们对该方法的实现如下所述：

1.  我们创建所使用的 like 模式。

1.  我们获得一个`Sort`对象的引用。

1.  我们调用我们的存储库方法并提供所需的参数。

`search()`方法的源代码如下所示：

```java
@Transactional(readOnly = true)
@Override
public List<Contact> search(String searchTerm) {
    String likePattern = buildLikePattern(dto.getSearchTerm());
    Sort sortSpec = sortByLastNameAndFirstNameAsc();
    return repository.findContacts(likePattern, sortSpec);
}
```

### JPA Criteria API

为了使用 JPA Criteria API 创建查询，我们必须修改`ContactRepository`接口以扩展`JpaSpecificationExecutor<T>`接口。这使我们可以访问`List<Contact> findAll(Specification spec, Sort sort)`方法，该方法返回与给定搜索条件匹配的实体的排序列表。

我们对`RepositoryContactService`类的`search()`方法的实现如下所述：

1.  我们通过使用我们的规范构建器类获取所使用的搜索条件。

1.  我们获取所使用的`Sort`对象。

1.  我们将调用`ContactRepository`的`findAll()`方法并提供必要的参数。

我们的`search()`方法如下所示：

```java
@Transactional(readOnly = true)
@Override
public List<Contact> search(String searchTerm) {
    Specification<Contact> contactSpec = firstOrLastNameStartsWith(searchTerm);
    Sort sortSpec = sortByLastNameAndFirstNameAsc();
    return repository.findAll(contactSpec, sortSpec);
}
```

## 使用 Querydsl 进行排序

在我们的联系人存储库中扩展`QuerydslPredicateExecutor<T>`接口使我们可以访问`Iterable<Contact> findAll(Predicate predicate, OrderSpecifier<?>... orders)`方法，该方法返回与给定搜索条件匹配的所有实体的排序列表。

首先，我们必须创建一个服务方法，该方法创建一个`OrderSpecifier`对象数组。`sortByLastNameAndFirstNameAsc()`方法的源代码如下所示：

```java
private OrderSpecifier[] sortByLastNameAndFirstNameAsc() {
  OrderSpecifier[] orders = {QContact.contact.lastName.asc(), QContact.contact.firstName.asc()};
  return orders;
}
```

我们的下一步是修改`RepositoryContactService`类的`search()`方法的实现，以满足给定的要求。我们对`search()`方法的实现如下所述：

1.  我们获取所使用的搜索条件。

1.  我们通过调用我们之前创建的`sortByLastNameAndFirstNameAsc()`方法来获取所使用的`OrderSpecifier`数组。

1.  我们调用`ContactRepository`的`findAll()`方法并提供所需的参数。

1.  我们使用从`Commons Collections`库中找到的`CollectionUtils`类将所有联系人添加到返回的列表中。

`search()`方法的源代码如下所示：

```java
@Transactional(readOnly = true)
@Override
public List<Contact> search(String searchTerm) {
  Predicate contactPredicate = firstOrLastNameStartsWith(searchTerm);
  OrderSpecifier[] orderSpecs = sortByLastNameAndFirstNameAsc();

  Iterable<Contact> contacts = repository.findAll(contactPredicate, orderSpecs);
  List<Contact> contactList = new ArrayList<Contact>();
  CollectionUtils.addAll(contactList, contacts.iterator());

  return contactList;
}
```

## 我们应该使用什么技术？

最好的方法是尽可能将查询生成和排序逻辑放在同一个地方。这样，我们只需查看一个地方，就可以检查我们查询的实现。这个一般指导方针可以细化为以下具体说明：

+   如果我们正在使用方法名称生成查询，我们应该使用这种方法来对查询结果进行排序。如果方法名称变得太长或太丑，我们总是可以使用`Sort`类来对查询结果进行排序，但这不应该是我们的首选。相反，我们应该考虑使用`@Query`注解来构建我们的查询。

+   如果我们使用 JPQL 或 SQL，我们应该在查询字符串中添加排序逻辑。这样我们就可以从同一个地方检查我们的查询逻辑和排序逻辑。

+   如果我们使用带有`@Query`注解的命名查询或本地查询，我们必须将排序逻辑添加到我们的查询字符串中。

+   当我们使用 JPA Criteria API 构建查询时，我们必须使用`Sort`类，因为这是`JpaSpecificationExecutor<T>`接口提供的唯一方法。

+   当我们使用 Querydsl 构建查询时，我们必须使用`OrderSpecifier`类来对查询结果进行排序，因为这是`QueryDslPredicateExecutor<T>`接口所要求的。

# 分页查询结果

对于几乎每个呈现某种数据的应用程序来说，对查询结果进行分页是一个非常常见的需求。Spring Data JPA 分页支持的关键组件是`Pageable`接口，它声明了以下方法：

| 方法 | 描述 |
| --- | --- |
| `int getPageNumber()` | 返回请求页面的编号。页面编号是从零开始的。因此，第一页的编号是零。 |
| `int getPageSize()` | 返回单个页面上显示的元素数量。页面大小必须始终大于零。 |
| `int getOffset()` | 根据给定的页码和页面大小返回所选偏移量。 |
| `Sort getSort()` | 返回用于对查询结果进行排序的排序参数。 |

我们可以使用这个接口来通过 Spring Data JPA 对查询结果进行分页：

1.  创建一个新的`PageRequest`对象。我们可以使用`PageRequest`类，因为它实现了`Pageable`接口。

1.  将创建的对象作为参数传递给存储库方法。

如果我们使用查询方法来创建我们的查询，我们有两种选项可以作为查询方法的返回类型：

+   如果我们需要访问请求页面的元数据，我们可以使我们的查询方法返回`Page<T>`，其中`T`是受管理实体的类型。

+   如果我们只对获取请求页面的联系人感兴趣，我们应该使我们的查询方法返回`List<T>`，其中`T`是受管理实体的类型。

为了向我们的联系人管理应用程序添加分页，我们必须对应用程序的服务层进行更改，并实现分页。这两个任务在以下子节中有更详细的描述。

## 改变服务层

由于 Spring Data JPA 存储库只是接口，我们必须在服务层创建`PageRequest`对象。这意味着我们必须找到一种方法将分页参数传递到服务层，并使用这些参数创建`PageRequest`对象。我们可以通过以下步骤实现这个目标：

1.  我们创建了一个存储分页参数和搜索词的类。

1.  改变服务接口的方法签名。

1.  我们实现了创建`PageRequest`对象的方法。

### 创建一个用于分页参数的类

首先，我们必须创建一个用于存储分页参数和使用的搜索词的类。Spring Data 提供了一个名为`PageableArgumentResolver`的自定义参数解析器，它将通过解析请求参数自动构建`PageRequest`对象。有关这种方法的更多信息，请访问[`static.springsource.org/spring-data/data-jpa/docs/current/reference/html/#web-pagination`](http://static.springsource.org/spring-data/data-jpa/docs/current/reference/html/#web-pagination)。

我们不会使用这种方法，因为我们不想在我们的 Web 层和 Spring Data 之间引入依赖关系。相反，我们将使用一个只有几个字段、getter 和 setter 的简单 DTO。`SearchDTO`的源代码如下：

```java
public class SearchDTO {

    private int pageIndex;
    private int pageSize;
    private String searchTerm;

   //Getters and Setters
}
```

### 改变服务接口

我们需要修改示例应用程序的`ContactService`接口，以便为联系人列表和搜索结果列表提供分页支持。所需的更改如下所述：

+   我们必须用`findAllForPage()`方法替换`findAll()`方法，并将页码和页面大小作为参数传递

+   我们必须修改`search()`方法的签名，以将`SearchDTO`作为参数

变更方法的签名如下：

```java
public List<Contact> findAllForPage(int pageIndex, int pageSize);

public List<Contact> search(SearchDTO dto);
```

### 创建 PageRequest 对象

在我们可以继续实际实现之前，我们必须向`RepositoryContactService`类添加一个新方法。这个方法用于创建作为参数传递给我们的存储库的`PageRequest`对象。`buildPageSpecification()`方法的实现如下所述：

1.  我们使用`sortByLastNameAndFirstNameAsc()`方法来获取对使用的`Sort`对象的引用。

1.  我们使用页码、页面大小和 Sort 对象来创建一个新的`PageRequest`对象。

相关方法的源代码如下：

```java
private Pageable buildPageSpecification(int pageIndex, int pageSize) {
  Sort sortSpec = sortByLastNameAndFirstNameAsc();
  return new PageRequest(pageIndex, pageSize, sortSpec);
}

private Sort sortByLastNameAndFirstNameAsc() {
  return new Sort(new Sort.Order(Sort.Direction.ASC, "lastName"),
        new Sort.Order(Sort.Direction.ASC, "firstName")
    );
}
```

## 实现分页

为了对查询结果进行分页，我们必须将创建的`PageRequest`对象传递给正确的存储库方法。这个方法取决于我们用来构建查询的方法。这些方法中的每一种都在本小节中描述。

### JpaRepository

因为`ContactRepository`扩展了`JpaRepository<T,ID>`接口，我们可以访问`Page<Contact> findAll(Pageable page)`方法，用于对所有实体的列表进行分页。`RepositoryContactService`类的`findAllForPage()`方法的实现如下所述：

1.  我们得到了使用的`PageRequest`对象。

1.  通过调用存储库方法并将`PageRequest`对象作为参数传递来获取`Page<Contact>`的引用。

1.  我们返回一个联系人列表。

我们的`findAllForPage()`方法的源代码如下：

```java
@Transactional(readOnly = true)
@Override
public List<Contact> findAllForPage(int pageIndex, int pageSize) {
  Pageable pageSpecification = buildPageSpecification(pageIndex, pageSize); 

  Page<Contact> page = repository.findAll(pageSpecification);

  return page.getContent();
}
```

### 从方法名生成查询

如果我们使用从方法名生成查询的策略来构建查询，我们可以对查询结果进行分页：

1.  为查询方法添加分页支持。

1.  从服务方法调用查询方法。

#### 为查询方法添加分页支持

为我们的查询方法添加分页支持相当简单。我们只需要对查询方法的签名进行以下更改：

1.  将`Pageable`接口添加为查询方法的参数。

1.  确定查询方法的返回类型。

因为我们对页面元数据不感兴趣，所以我们的查询方法的签名如下所示：

```java
public List<Contact> findByFirstNameStartingWithOrLastNameStartingWith(String firstName, String lastName, Pageable page);
```

#### 修改服务类

`RepositoryContactService`的`search()`方法需要的修改相当简单。我们得到一个`PageRequest`对象的引用，并将其作为参数传递给我们的查询方法。修改后的方法源代码如下：

```java
@Transactional(readOnly = true)
@Override
public List<Contact> search(SearchDTO dto) {
    Pageable pageSpecification = buildPageSpecification(dto.getPageIndex(), dto.getPageSize());

    return repository.findByFirstNameStartingWithOrLastNameStartingWith(dto.getSearchTerm(), dto.getSearchTerm(), pageSpecification);
}
```

### 命名查询

如果我们想要对命名查询的查询结果进行分页，我们必须：

1.  为查询方法添加分页支持。

1.  从服务方法调用查询方法。

#### 为查询方法添加分页支持

我们可以通过将`Pageable`接口作为查询方法的参数来为命名查询支持分页。此时，我们不需要页面元数据。因此，我们的查询方法的签名如下所示：

```java
public List<Contact> findContacts(@Param("searchTerm") String searchTerm, Pageable page);
```

#### 修改服务类

我们对`RepositoryContactService`类的`search()`方法的实现如下所述：

1.  我们得到了使用的模式。

1.  我们得到所需的`PageRequest`对象。

1.  通过调用修改后的查询方法来获取联系人列表。

我们修改后的`search()`方法的源代码如下：

```java
@Transactional(readOnly = true)
@Override
public List<Contact> search(SearchDTO dto) {
    String likePattern = buildLikePattern(dto.getSearchTerm());

    Pageable pageSpecification = buildPageSpecification(dto.getPageIndex(), dto.getPageSize());

    return repository.findContacts(likePattern, pageSpecification);
}
```

### @Query 注解

我们可以通过`@Query`注解构建的 JPQL 查询来对查询结果进行分页：

1.  为查询方法添加分页支持。

1.  从服务方法调用查询方法。

#### 为查询方法添加分页支持

我们可以通过对方法签名进行以下更改，为使用`@Query`注解注释的查询方法添加分页支持：

1.  我们将`Pageable`接口添加为方法的参数。

1.  我们确定方法的返回类型。

在这一点上，我们对返回页面的元数据不感兴趣。因此，查询方法的声明如下所示：

```java
@Query("SELECT c FROM Contact c WHERE LOWER(c.firstName) LIKE LOWER(:searchTerm) OR LOWER(c.lastName) LIKE LOWER(:searchTerm)")
public List<Contact> findContacts(@Param("searchTerm") String searchTerm, Pageable page);
```

#### 修改服务方法

`RepositoryContactService`类的`search()`方法的实现如下所述：

1.  我们得到了使用的 like 模式。

1.  我们得到了使用过的`PageRequest`对象的引用。

1.  通过调用查询方法并将 like 模式和创建的`PageRequest`对象作为参数，我们可以得到联系人列表。

`search()`方法的源代码如下所示：

```java
@Transactional(readOnly = true)
@Override
public List<Contact> search(SearchDTO dto) {
    String likePattern = buildLikePattern(dto.getSearchTerm());

    Pageable pageSpecification = buildPageSpecification(dto.getPageIndex(), dto.getPageSize());

    return repository.findContacts(likePattern, pageSpecification);
}
```

### JPA Criteria API

为了使用 JPA Criteria API 构建查询，`ContactRepository`接口必须扩展`JpaSpecificationExecutor<T>`接口。这使我们可以访问`Page<Contact> findAll(Specification spec, Pageable page)`方法，该方法可用于对标准查询的查询结果进行分页。我们唯一需要做的就是修改`RepositoryContactService`类的`search()`方法。我们的实现如下所述：

1.  我们得到了使用过的规范。

1.  我们得到了使用过的`PageRequest`对象。

1.  通过调用存储库方法并将规范和`PageRequest`对象作为参数传递，我们得到了`Page`的实现。

1.  通过调用`Page`类的`getContent()`方法，我们返回了请求的联系人列表。

我们的搜索方法的源代码如下所示：

```java
@Transactional(readOnly = true)
@Override
public List<Contact> search(SearchDTO dto) {
    Specification<Contact> contactSpec = firstOrLastNameStartsWith(dto.getSearchTerm());
    Pageable pageSpecification = buildPageSpecification(dto.getPageIndex(), dto.getPageSize());

    Page<Contact> page = repository.findAll(contactSpec, pageSpecification);

    return page.getContent();
}
```

### Querydsl

由于`ContactRepository`接口扩展了`QueryDslPredicateExecutor<T>`接口，我们可以访问`Page<Contact> findAll(Predicate predicate, Pageable page)`方法，我们可以用它来对查询结果进行分页。为了为我们的搜索函数添加分页支持，我们必须对`RepositoryContactService`类的现有`search()`方法进行一些更改。这个方法的新实现在以下步骤中描述：

1.  我们得到了使用过的`Predicate`的引用。

1.  我们得到了使用过的`PageRequest`对象。

1.  通过调用存储库方法并将`Predicate`和`PageRequest`对象作为参数传递，我们得到了一个`Page`引用。

1.  我们返回请求的联系人。

我们的新`search()`方法的源代码如下所示：

```java
@Transactional(readOnly = true)
@Override
public List<Contact> search(SearchDTO dto) {
    Predicate contactPredicate = firstOrLastNameStartsWith(dto.getSearchTerm());
    Pageable pageSpecification = buildPageSpecification(dto.getPageIndex(), dto.getPageSize());

    Page<Contact> page = repository.findAll(contactPredicate, pageSpecification);

    return page.getContent();
}
```

# 总结

在本章中，我们已经学到了：

+   我们可以使用方法名称的查询生成，命名查询或`@Query`注解来创建 Spring Data JPA 的查询方法

+   我们可以通过使用 JPA Criteria API 或 Querydsl 来创建动态查询

+   有三种不同的方法可以用来对查询结果进行排序

+   如果我们对查询方法的查询结果进行分页，方法的返回类型可以是`List`或`Page`

+   每种查询创建方法都有其优势和劣势，我们在选择当前问题的正确解决方案时必须考虑这些。

有时我们需要向我们的存储库添加自定义函数。这个问题在下一章中解决。
