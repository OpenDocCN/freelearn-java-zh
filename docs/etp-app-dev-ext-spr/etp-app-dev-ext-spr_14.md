# 附录 A.介绍 Spring Data JPA

Spring Data JPA 网站[`projects.spring.io/spring-data-jpa/`](http://projects.spring.io/spring-data-jpa/)有一个开头段落简洁地描述了实现基于 JPA 的 DAO 层的问题：

> *实现应用程序的数据访问层已经相当麻烦了。必须编写大量样板代码来执行简单的查询以及执行分页和审计。Spring Data JPA 旨在通过减少实际需要的工作量，显着改善数据访问层的实现。作为开发人员，您编写存储库接口，包括自定义查找方法，Spring 将自动提供实现。*

在第四章中，*数据访问变得容易*，我们实现了 DAO 设计模式，将数据库持久性抽象为一个明确定义的层。我们故意决定在本章中*不*介绍 Spring Data JPA，因为目标受众是可能没有使用 Java 持久性 API 经验的中级开发人员。介绍了 JPA 术语、概念和实际示例，以便让您了解 JPA 的工作原理。使用 Java 接口、Java 泛型和命名查询概念对于理解 Spring Data JPA 的优雅工作方式至关重要。

Spring Data JPA 不要求您编写存储库接口的实现。当您运行 Spring Data JPA 应用程序时，这些实现是“即时”创建的。开发人员所需做的就是编写扩展`org.springframework.data.repository.CrudRepository`并遵循 Spring Data JPA 命名约定的 DAO Java 接口。DAO 实现会在运行时为您创建。

Spring Data JPA 将在内部实现执行与第四章中实现的相同功能的代码，*数据访问变得容易*。使用 Spring Data，我们可以将`CompanyDao`接口重写为：

```java
package com.gieman.tttracker.dao;

import com.gieman.tttracker.domain.Company;
import java.util.List;
import org.springframework.data.repository.CrudRepository;

public interface CompanyDao extends CrudRepository<Company, Integer>{

}
```

`CompanyDao`实现将包括`findAll`方法，因为它在`CrudRepository`接口中定义；我们不需要将其定义为单独的方法。

如果您熟悉 JPA 和第四章中涵盖的内容，*数据访问变得容易*，那么您应该探索 Spring Data JPA 框架。然后，实现基于 JPA 的存储库将变得更加容易！
