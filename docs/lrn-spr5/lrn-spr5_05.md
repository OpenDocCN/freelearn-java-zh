## 第五章．保持一致性：事务管理

在上一章中，我们深入讨论了使用日志机制作为交叉技术的面向方面编程。事务管理是另一种交叉技术，在处理持久性时在应用程序中扮演着非常重要的角色。在本章中，我们将通过讨论以下几点来探索事务管理：

+   事务管理是什么？

+   事务管理的重要性。

+   事务管理的类型

+   Spring 和事务管理

+   Spring 框架中基于注解的事务管理

许多开发者经常谈论这个花哨的术语“事务管理”。我们中有多少人觉得自己在使用它或其自定义时感到舒适呢？它真的那么难以理解吗？在代码中添加事务是否需要添加大量的复杂代码？不是的！！实际上，它是最容易理解的事情之一，也是最容易开发的。在讨论、设计、开发与数据库进行数据处理的“持久层”时，事务管理非常普遍。事务是序列化多个数据库操作的基本单位，其中要么所有操作成功执行，要么一个都不执行。事务管理是处理事务的技术，通过管理其参数来处理事务。事务根据给定的事务参数保持数据库的一致性，以便要么事务单位成功，要么失败。事务绝不可能部分成功或失败。

现在你可能在想，如果其中的任何一个失败了会有什么大不了的？为什么这如此重要？让我们通过一个实际的场景来理解交易。我们想在某个网上购物网站上开设一个账户。我们需要填写一个表格，提供一些个人信息，并选择一个用户名来进行我们的网上购物。这些信息将由应用程序收集，然后保存在两个表中。一个是以用户名为主键的用户表，第二个是 user_info 表，用于存储用户的个人信息。在从用户那里收集数据后，开发者会对 user_info 表执行插入操作，然后将数据插入到用户表中。现在考虑这样一个场景：从用户那里收集的数据成功插入到 user_info 表中，但不幸的是，用户名在表中已经存在，所以第二个操作失败了。数据库处于不一致的状态。从逻辑上讲，数据应该要么同时添加到两个表中，要么一个都不添加。但在我们的案例中，数据只插入了一个表，而没有插入第二个表。这是因为我们在检查行是否插入成功之前就执行了永久的插入操作，现在即使第二个操作失败了也无法撤销。事务管理帮助开发者通过在数据库表中正确反映所有操作，或者一个都不反映来维护数据库的一致性和完整性。如果在单元操作中任何一个操作失败，所有在失败之前所做的更改都将被取消。当然，这不会自动发生，但开发者需要发挥关键作用。在 JDBC 中，开发者选择不使用自动提交操作，而是选择提交事务或回滚，如果其中任何一个操作失败。这两个术语在事务管理中非常重要。提交将更改永久反映到数据库中。回滚撤销所有在失败发生之前的操作所做的更改，使数据库恢复到原始状态。

以下是 Jim Gray 在 1970 年代定义的 ACID 属性，用于描述事务。这些属性后来被称为 ACID 属性。Gray 还描述了实现 ACID 属性的方法。让我们逐一讨论它们：

+   **原子性**：在数据库上连续执行多个操作时，要么所有操作都会成功执行，要么一个都不会执行。开发者可以控制是否通过提交它们来永久更改数据库，或者回滚它们。回滚将撤销所有操作所做的更改。一旦数据被提交，它就不能再次回滚。

+   **一致性**：为了将数据保存成适当排列且易于维护的格式，在创建数据库表时设置了规则、数据类型、关联和触发器。一致性确保在从一种状态转换到另一种状态获取数据时，将保持所有设置在其上的规则不变。

+   **隔离性**：在并发中，多个事务同时发生导致数据管理问题。隔离性通过锁定机制保持数据的一致状态。除非正在处理数据的事务完成，否则它将保持锁定。一旦事务完成其操作，另一个事务将被允许使用数据。

以下是 ANSI 或 ISO 标准定义的隔离级别：

+   **脏读**：考虑两个事务 A 和 B 正在运行的数据集。事务 A 进行了某些更改但尚未提交。与此同时，事务 B 读取了数据以及未提交更改的数据。如果事务 A 成功完成其操作，两个事务具有相同的数据状态。但如果事务 A 失败，它所做的数据更改将被回滚。由于 B 读取了未提交的数据，A 和 B 的数据集将不同。事务 B 使用了过时的数据，导致应用程序的业务逻辑失败。

+   **非可重复读**：再次考虑事务 A 和 B 正在完成一些操作。它们都读取了数据，事务 A 更改了一些值并成功提交。事务 B 仍在处理旧数据，导致不良影响。这种情况可以通过在第一个事务完成之前保持数据锁定来避免。

+   **幻读**：事务 A 和 B 拥有同一组数据。假设事务 A 已经执行了搜索操作，比如，A 根据书名搜索数据。数据库返回了 8 行数据给事务 A。此时事务 B 在表中插入了一行具有与 A 搜索的书名相同值的数据。实际上表中有 9 行数据，但 A 只得到了 8 行。

+   **可串行化**：这是最高的隔离级别，它锁定选定的使用数据，以避免幻读问题。

+   以下是数据库支持的默认隔离级别：

| 数据库 | 默认隔离级别 |
| --- | --- |
| Oracle | READ_COMMITTED |
| Microsoft SQL Server | READ_COMMITTED |
| MySQL | REPEATABLE_READ |
| PostgreSQL | READ_COMMITTED |
| DB2 | CURSOR STABILITY |

+   **持久性**：事务通过多种操作同时进行更改。持久性指定一旦数据库中的数据被更改、添加或更新，它必须是永久的。

一旦我们了解了描述事务的属性，了解事务进展的阶段将有助于我们有效地使用事务管理。

## 事务管理的生命周期

***

以下图表展示了每个事务进展的阶段：

![](img/image_05_001.png)

新启动的事务将经历以下阶段：

1.  **活动**：事务刚刚开始并且正在向前推进。

1.  **部分提交**：一旦操作成功执行，生成的值将被存储在易失性存储中。

1.  **失败**：在失败之前生成的值不再需要，将通过回滚从易失性存储区中删除它们。

1.  **中止**：操作已失败且不再继续。它将被停止或中止。

1.  **已提交**：所有成功执行的操作以及操作期间生成的所有临时值，一旦事务提交，将被永久存储。

1.  **终止**：当事务提交或中止时，它达到了其最终阶段——终止。

要处理与生命周期步骤和属性相关的事务，不能忽视一个非常重要的事实，即了解事务的类型。事务可以划分为本地事务或全局事务。

### 本地事务

本地事务允许应用程序连接到单个数据库，一旦事务中的所有操作成功完成，它将被提交。本地事务特定于资源，不需要服务器处理它们。配置的数据源对象将返回连接对象。此连接对象进一步允许开发人员根据需要执行数据库操作。默认情况下，此类连接是自动提交的。为了掌握控制权，开发人员可以使用提交或回滚手动处理事务。JDBC 连接是本地事务的最佳示例。

### 全局或分布式事务

全局事务由应用服务器如 Weblogic、WebSphere 管理。全局事务能够处理多个资源和服务器。全局事务由许多访问资源的本地事务组成。EJB 的容器管理事务使用全局事务。

### Spring 和事务管理

Spring 框架卓越地支持事务管理器的集成。它支持 Java 事务 API，JDBC，Hibernate 和 Java 持久化 API。框架支持称为事务策略的抽象事务管理。事务策略是通过服务提供者接口（SPI）通过 PlatformTransactionManager 接口定义的。该接口有提交和回滚事务的方法。它还有获取由 TransactionDefinition 指定的事务的方法。所有这些方法都会抛出 TransactionException，这是一个运行时异常。

`getTransaction()`方法根据 TransactionDefinition 参数返回 TransactionStatus。该方法返回的 TransactionStatus 代表一个新事务或现有事务。以下参数可以指定以定义 TransactionDefinition：

+   **传播行为**：当一个事务方法调用另一个方法时，传播行为就会讨论。在这种情况下，传播行为指明它将如何执行事务行为。调用方法可能已经启动了事务，那么被调用方法在这种情况下应该做什么？被调用方法是启动一个新事务，使用当前事务还是不支持事务？传播行为可以通过以下值来指定：

    +   **REQUIRED**：它表示必须有事务。如果没有事务存在，它将创建一个新的事务。

    +   **REQUIRES_NEW**：它指定每次都要有一个新的事务。当前事务将被挂起。如果没有事务存在，它将创建一个新的事务。

    +   **强制**：它表示当前事务将被支持，但如果没有进行中的事务，将抛出异常。

    +   **嵌套**：它表明，如果当前事务存在，方法将在嵌套事务中执行。如果没有事务存在，它将作为 PROPAGATION_REQUIRED 行事。

    +   **永不**：不支持事务，如果存在，将抛出异常。

    +   **不支持**：它表示该交易是不被支持的。如果交易与**永不**相反存在，它不会抛出异常，但会挂起交易。

+   **隔离性**：我们已经在深度讨论隔离级别。

+   **超时**：事务中提到的超时值，以秒为单位。

+   **只读**：该属性表示事务将只允许读取数据，不支持导致更新数据的操作。

以下是用 Spring 框架进行事务管理的优点：

Spring 通过以下两种方式简化事务管理：

+   编程事务管理。

+   声明式事务管理。

无论我们使用程序化事务还是声明式事务，最重要的是使用依赖注入（DI）定义`PlatformTransactionManager`。一个人应该清楚地知道是使用本地事务还是全局事务，因为定义`PlatformTransactionManager`是必不可少的。以下是一些可以用来定义`PlatformTransactionManager`的配置：

+   使用 DataSource PlatformTransactionManager 可以定义为：

```java
      <bean id="dataSource" 
        <!-DataSource configuration --> 
      </bean> 
      <bean id="transactionManager" 
        class="org.springframework.jdbc.datasource.DataSourceTransactionManager"> 
        <property name="dataSource" ref="dataSource"/>    
      </bean> 

```

+   使用 JNDI 和 JTA 定义 PlatformTransactionManager，如下所示：

```java
      <jee: jndi-lookup id="dataSource' jndi-name="jdbc/books"/> 
        <bean id="transactionManager"
          class="org.springframework.transaction.jta.JtaTransactionManager"> 
        </bean> 

```

+   使用 HibernateTransactionManager 定义 PlatformTransactionManager 为：

```java
      <bean id="sessionFactory" 
         class="org.springframework.orm.hibernate5.LocalSessionfactoryBean" 
         <!-define parameters for session factory --> 
      </bean> 

      <bean id=" transactionManager"
         class="org.springframework.orm.hibernate5.HibernateTransactionManager"> 
         <property name="sessionFactory" ref="sessionFactory"/> 
      </bean> 

```

让我们逐一开始使用 Spring 中的事务管理，

#### 程序化事务管理

在 Spring 中，可以通过使用 TransactionTemplate 或 PlatformTransactionManager 来实现程序化事务管理。

##### 使用 PlatformTransactionManager

PlatformTransactionManager 是讨论 Spring 事务管理 API 的中心。它具有提交、回滚的功能。它还提供了一个返回当前活动事务的方法。由于它是一个接口，因此可以在需要时轻松模拟或垫片。Spring 提供了 DataSourceTransactionManager、HibernateTransactionManager、CciLocalTransactionManager、JtaTransactionManager 和 OC4JJtaTransactionManager 作为 PlatformTransactionManager 的几种实现。要使用 PlatformTransactionManager，可以在 bean 中注入其任何实现以用于事务管理。此外，TransactionDefinition 和 TransactionStatus 对象的可以使用来回滚或提交事务。

在前进之前，我们需要讨论一个非常重要的点。通常，应用程序需求决定是否将事务应用于服务层或 DAO 层。但是，是否将事务应用于 DAO 层或服务层仍然是一个有争议的问题。尽管将事务应用于 DAO 层可以使事务更短，但最大的问题将是多事务的发生。并且必须非常小心地处理并发，不必要的复杂性会增加。当将事务应用于服务层时，DAO 将使用单个事务。在我们的应用程序中，我们将事务应用于服务层。

为了在应用程序中应用事务管理，我们可以考虑以下几点，

+   是将事务应用于 DAO 层还是服务层？

+   决定是使用声明式事务还是程序化事务管理

+   在 bean 配置中定义要使用的 PlatformtransactionManager。

+   决定事务属性，如传播行为、隔离级别、只读、超时等，以定义事务。

+   根据程序化或声明式事务管理，在代码中添加事务属性。

让我们使用事务来更好地理解。我们将使用第三章中开发的`Ch03_JdbcTemplate`应用程序作为基础应用程序，并使用`PlatformTransactionManager`遵循步骤来使用事务，

1.  创建一个名为`Ch05_PlatformTransactionManager`的新 Java 应用程序，并添加所有必需的 jar 文件，包括 Spring 核心、Spring-jdbc、Spring-transaction、Spring-aop、commons-logging 和 mysql-connector。

1.  在`com.packt.ch03.beans`包中复制或创建`Book.java`文件。

1.  在`com.packt.ch03.dao`包中复制或创建`BookDAO.java`和`BookDAO_JdbcTemplate.java`文件。应用程序的最终结构如下所示：

![](img/image_05_002.png)

1.  我们将在`BookDAO`中添加一个新的方法来搜索书籍，因为在添加之前，我们需要找出'Book'表中是否有具有相同 ISBN 的书籍。如果已经存在，我们不希望不必要的再次进行添加。新添加的方法将如下所示：

```java
      public Book serachBook(long ISBN); 

```

1.  `BookDAO_JdbcTemplate.java`需要覆盖接口中 newly added method，如下所示：

```java
      @Override 
      public Book serachBook(long ISBN) { 
        // TODO Auto-generated method stub 
        String SEARCH_BOOK = "select * from book where ISBN=?"; 
        Book book_serached = null; 
        try { 
          book_serached = jdbcTemplate.queryForObject(SEARCH_BOOK,  
            new Object[] { ISBN },  
            new RowMapper<Book>(){ 
            @Override 
              public Book mapRow(ResultSet set, int rowNum)  
              throws SQLException { 
                Book book = new Book(); 
                book.setBookName(set.getString("bookName")); 
                book.setAuthor(set.getString("author")); 
                book.setDescription(set.getString("description")); 
                book.setISBN(set.getLong("ISBN")); 
                book.setPrice(set.getInt("price")); 
                book.setPublication(set.getString("publication")); 
                return book; 
              } 
            }); 
            return book_serached; 
          } catch (EmptyResultDataAccessException ex) { 
          return new Book(); 
        } 
      } 

```

我们添加了一个匿名内部类，它实现了`RowMapper`接口，使用`queryForObject()`方法将从数据库检索的对象绑定到`Book`对象的数据成员。代码正在搜索书籍，然后将`ResultSet`中的列值绑定到`Book`对象。我们返回了一个具有默认值的对象，仅为我们的业务逻辑。

1.  在`com.packt.ch05.service`包中添加`BookService`接口作为服务层，并具有以下方法签名：

```java
      public interface BookService { 
        public Book searchBook(long ISBN); 
        public boolean addBook(Book book); 
        public boolean updateBook(long ISBN, int price); 
        public boolean deleteBook(long ISBN); 
      } 

```

1.  创建`BookServiceImpl`实现`BookService`。因为这是服务，用`@Service`注解类。

1.  首先向类中添加两个数据成员，第一个类型为`PlatformTransactionManager`以处理事务，第二个类型为`BookDAO`以执行 JDBC 操作。使用`@Autowired`注解对它们进行依赖注入。

1.  首先，让我们分两步为服务层开发`searchBook()`方法，以处理只读事务：

    +   创建一个`TransactionDefinition`实例。

    +   创建一个`TransactionStatus`实例，该实例从使用上一步创建的`TransactionDefinition`实例的`TransactionManager`中获取。`TransactionStatus`将提供事务的状态信息，该信息将用于提交或回滚事务。

在这里，将事务设置为只读，将属性设置为`true`，因为我们只是想要搜索书籍，并不需要在数据库端执行任何更新。至此步骤开发出的代码将如下所示：

```java
      @Service(value = "bookService") 
      public class BookServiceImpl implements BookService { 
        @Autowired 
        PlatformTransactionManager transactionManager; 

        @Autowired  
        BookDAO bookDAO; 

        @Override 
        public Book searchBook(long ISBN) { 
          TransactionDefinition definition = new  
            DefaultTransactionDefinition(); 
          TransactionStatus transactionStatus =  
            transactionManager.getTransaction(definition); 
          //set transaction as read-only 
          ((DefaultTransactionDefinition)  
          definition).setReadOnly(true); 
          Book book = bookDAO.serachBook(ISBN); 
          return book; 
        } 
        // other methods from BookService     
      }   

```

我们更新只读事务属性的方式，也可以同样设置其他属性，如隔离级别、传播、超时。

1.  让我们向服务层添加`addBook()`方法，以找出是否已有具有相同 ISBN 的书籍，如果没有，则在表中插入一行。代码将如下所示：

```java
      @Override 
      public boolean addBook(Book book) { 
        // TODO Auto-generated method stub 
        TransactionDefinition definition = new  
          DefaultTransactionDefinition(); 
        TransactionStatus transactionStatus =  
          transactionManager.getTransaction(definition); 

        if (searchBook(book.getISBN()).getISBN() == 98564567l) { 
          System.out.println("no book"); 
          int rows = bookDAO.addBook(book); 
          if (rows > 0) { 
            transactionManager.commit(transactionStatus); 
            return true; 
          } 
        } 
        return false; 
      } 

```

`transactionManager.commit()`将永久将数据提交到书籍表中。

1.  以同样的方式，让我们添加`deleteBook`和`updateBook()`方法，如下所示，

```java
      @Override 
      public boolean updateBook(long ISBN, int price) { 
        TransactionDefinition definition = new  
          DefaultTransactionDefinition(); 
        TransactionStatus transactionStatus =  
          transactionManager.getTransaction(definition); 
        if (searchBook(ISBN).getISBN() == ISBN) { 
          int rows = bookDAO.updateBook(ISBN, price); 
          if (rows > 0) { 
            transactionManager.commit(transactionStatus); 
            return true; 
          } 
        } 
        return false; 
      } 

      @Override 
      public boolean deleteBook(long ISBN)  
      { 
        TransactionDefinition definition = new  
          DefaultTransactionDefinition(); 
        TransactionStatus transactionStatus =  
          transactionManager.getTransaction(definition); 
        if (searchBook(ISBN).getISBN() != 98564567l) { 
          boolean deleted = bookDAO.deleteBook(ISBN); 
          if (deleted) { 
            transactionManager.commit(transactionStatus); 
            return true; 
          } 
        } 
        return false; 
      } 

```

1.  复制或创建 connection_new.xml 以进行 bean 配置。添加一个 DataSourceTransactionManager 的 bean，正如我们在讨论如何使用 DataSource 配置 PlatformTransactionManager 时所看到的。

1.  更新从 XML 中扫描包，因为我们还想考虑新添加的包。更新后的配置如下：

```java
      <context:component-scan base- package="com.packt.*">
      </context:component-scan> 

```

1.  最后一步将是把主代码添加到 MainBookService_operation.java 中，该文件将使用 BookServiceImpl 对象调用服务层的方法，就像我们之前对 BookDAO_JdbcTemplate 对象所做的那样。代码如下所示：

```java
      public static void main(String[] args) { 
        // TODO Auto-generated method stub 
        ApplicationContext context = new   
          ClassPathXmlApplicationContext("connection_new.xml"); 
        BookService service = (BookService)    
          context.getBean("bookService"); 
        // add book 
        boolean added = service.addBook(new Book("Java EE 7  
          Developer Handbook", 97815674L, "PacktPub  
          publication", 332,  "explore the Java EE7  
          programming", "Peter pilgrim")); 
        if (added) { 
          System.out.println("book inserted successfully"); 
        } else 
        System.out.println("SORRY!cannot add book"); 
        // update the book 
        boolean updated = service.updateBook(97815674L, 800); 
        if (updated) { 
          System.out.println("book updated successfully"); 
        } else 
        System.out.println("SORRY!cannot update book"); 
        // delete the book 
        boolean deleted = service.deleteBook(97815674L); 
        if (deleted) { 
          System.out.println("book deleted successfully"); 
        } else 
        System.out.println("SORRY!cannot delete book"); 
      } 

```

##### TransactionTemplate

使用线程安全的 TransactionTemplate 可以帮助开发者摆脱重复的代码，正如我们已经讨论过的 JdbcTemplate。它通过回调方法使程序化事务管理变得简单而强大。使用 TransactionTemplate 变得容易，因为它有各种事务属性的不同设置方法，如隔离级别、传播行为等。使用 Transaction 模板的第一步是通过提供事务管理器来获取其实例。第二步将是获取 TransactionCallback 的实例，该实例将传递给 execute 方法。以下示例将演示如何使用模板，我们不需要像早期应用程序中那样创建 TransactionDefinition，

1.  创建一个名为 Ch05_TransactionTemplate 的 Java 应用程序，并复制早期应用程序中所需的所有 jar 文件。

1.  我们将保持应用程序的结构与 Ch05_PlatformTransactionManager 应用程序相同，因此您可以复制 bean、dao 和服务包。我们唯一要做的改变是在 BookServiceImpl 中使用 TransactionTemplate 而不是 PlatformTransactionManager。

1.  从 BookServiceImpl 中删除 PlatformTransactionManager 数据成员并添加 TransactionTemplate。

1.  使用@Autowired 注解来使用 DI。

1.  我们将更新 searchBook()方法，使其使用 TransactionTemplate，并通过 setReadOnly(true)将其设置为只读事务。TransactionTemplate 有一个名为'execute()'的回调方法，可以在其中编写业务逻辑。该方法期望一个 TransactionCallback 的实例，并返回搜索到的书籍。代码如下所示：

```java
      @Service(value = "bookService") 
      public class BookServiceImpl implements BookService { 
        @Autowired 
        TransactionTemplate transactionTemplate; 

        @Autowired 
        BookDAO bookDAO; 

        public Book searchBook(long ISBN) { 
          transactionTemplate.setReadOnly(true);   
          return transactionTemplate.execute(new  
            TransactionCallback<Book>()  
          { 
            @Override 
            public Book doInTransaction(TransactionStatus status) { 
              // TODO Auto-generated method stub 
              Book book = bookDAO.serachBook(ISBN); 
              return book; 
          }     
        });  
      }  

```

为了执行任务，我们通过内部类的概念创建了 TransactionCallback 的实例。这里指定的泛型类型是 Book，因为它是 searchBook()方法的返回类型。这个类重写了 doInTransaction()方法，以调用 DAO 的 searchBook()方法中的业务逻辑。

还可以再实现一个 TransactionCallback 的版本，使用 TransactionCallbackWithoutResult。这种情况下可以用于服务方法没有返回任何内容，或者其返回类型为 void。

1.  现在让我们添加`addBook()`。我们首先必须使用`searchBook()`查找书籍是否存在于表中。如果书籍不存在，则添加书籍。但由于`searchBook()`使事务变为只读，我们需要更改行为。由于`addBook()`有布尔值作为其返回类型，我们将使用布尔类型的`TransactionCallBack`。代码将如下所示：

```java
      @Override 
      public boolean addBook(Book book) { 
        // TODO Auto-generated method stub 
        if (searchBook(book.getISBN()).getISBN() == 98564567l)  
        { 
          transactionTemplate.setReadOnly(false); 
          return transactionTemplate.execute(new  
            TransactionCallback<Boolean>()  
          { 
            @Override 
            public boolean doInTransaction(TransactionStatus status) { 
              try { 
                int rows = bookDAO.addBook(book); 
                if (rows > 0) 
                  return true; 
              } catch (Exception exception) { 
                status.setRollbackOnly(); 
              } 
              return false; 
            } 
          }); 
        } 
        return false; 
      } 

```

代码清楚地显示了 TransactionTemplate 赋予我们更改尚未内部管理的事务属性的能力，而无需编写 PlatformTransactionManager 所需的模板代码。

1.  同样，我们可以为`deleteBook`和`updateBook()`添加代码。你可以在线源代码中找到完整的代码。

1.  从`Ch05_PlatformTransactionmanager`类路径中复制`connection_new.xml`文件，并添加一个`TransactionTemplate`的 bean，如下所示：

```java
      <bean id="transactionTemplate"
        class="org.springframework.transaction.support.TransactionTemplate"> 
        <property name="transactionManager"  
          ref="transactionManager"></property> 
      </bean> 

```

我们已经有了一个事务管理器的 bean，所以我们在这里不会再次添加它。

1.  将`MainBookService_operations.java`文件复制到默认包中以测试代码。我们会发现代码成功执行。

1.  在继续前进之前，请按照如下方式修改`searchBook()`方法中的`doInTransaction()`代码；

```java
      public Book doInTransaction(TransactionStatus status) { 
        //Book book = bookDAO.serachBook(ISBN); 
        Book book=new Book(); 
        book.setISBN(ISBN); 
        bookDAO.addBook(book); 
        return book; 
      } 

```

1.  执行后，我们会得到一个堆栈跟踪，如下所示，它表示只读操作不允许修改数据：

```java
      Exception in thread "main" 
      org.springframework.dao.TransientDataAccessResourceException:  
      PreparedStatementCallback; SQL [insert into book values(?,?,?,?,?,?)];  
      Connection is read-only. Queries leading to data modification are not
      allowed; nested exception is java.sql.SQLException:
      Connection is read- only.  

```

#### 声明式事务管理

Spring 框架使用 AOP 来简化声明式事务管理。声明式事务管理最好的地方在于，它不一定需要由应用服务器管理，并且可以应用于任何类。该框架还通过使用 AOP，使开发者能够定制事务行为。声明式事务可以是基于 XML 的，也可以是基于注解的配置。

##### 基于 XML 的声明式事务管理：

该框架提供了回滚规则，用于指定事务将在哪种异常类型下回滚。回滚规则可以在 XML 中如下指定，

```java
<tx:advise id=:transactionAdvise" transaction-manager="transactionamanager">  
  <tx:attributes> 
     <tx:method name="find*" read-only="true" 
       rollback- for ="NoDataFoundException'> 
    </tx:attributes> 
  </tx:advise> 

```

配置甚至可以指定属性，例如，

+   '**no-rollback-for**' - 用以指定我们不想回滚事务的异常。

+   **传播** - 用以指定事务的传播行为，其默认值为'REQUIRED'。

+   **隔离** - 用以指定隔离级别。

+   **超时** - 以秒为单位的事务超时值，默认值为'-1'。

由于现在我们更倾向于使用注解事务管理，而不浪费时间，让我们继续讨论注解事务管理。

##### 基于注解的事务管理

`@Transaction`注解有助于开发基于注解的声明式事务管理，它可以应用于接口级别、类级别以及方法级别。要启用注解支持，需要配置以下配置以及事务管理器，

```java
<bean id="transactionManager" class=" your choice of transaction manager"> 
  <!-transaction manager configuration - -> 
</bean> 
<tx:annotation-driven transaction-manager="transcationManager"/> 

```

如果为 PlatformTransactionManager 编写的 bean 名称是'transactionManager'，则可以省略'transaction-manager'属性。

以下是可以用来自定义事务行为的属性，

+   **值** - 用于指定要使用的事务管理器。

+   **传播行为** - 用于指定传播行为。

+   **隔离级别** - 用于指定隔离级别。

+   **只读** - 用于指定读或写行为。

+   **超时** - 用于指定事务超时。

+   **rollbackForClassName** - 用于指定导致事务回滚的异常类数组。

+   **rollbackFor** - 用于指定导致事务回滚的异常类数组。

+   **noRollbackFor** - 用于指定不导致事务回滚的异常类数组。

+   **noRollbackForClassName** - 用于指定不导致事务回滚的异常类数组。

让我们使用@Transactional 来演示应用程序中的声明式事务管理，而不是使用以下步骤的帮助进行程序化事务管理：

1.  创建 Ch05_Declarative_Transaction_Management 并添加所需的 jar，就像在早期的应用程序中一样。

1.  从 Ch05_PlatformTransactionManager 应用程序中复制 com.packt.ch03.beans 和 com.packt.ch03.dao。

1.  在 com.packt.ch05.service 包中复制 BookService.java 接口。

1.  在 com.packt.ch05.service 包中创建 BookServiceImpl 类，并添加一个类型为 BookDAO 的数据成员。

1.  用@Autowired 注解类型为 BookDAO 的数据成员。

1.  用@Transactional(readOnly=true)注解 searchBook()，并编写使用 JdbcTemplate 搜索数据的代码。类如下：

```java
      @Service(value = "bookService") 
      public class BookServiceImpl implements BookService { 

        @Autowired 
        BookDAO bookDAO; 

        @Override 
        @Transactional(readOnly=true) 
        public Book searchBook(long ISBN)  
        { 
          Book book = bookDAO.serachBook(ISBN); 
          return book; 
        } 

```

1.  从 classpath 中的 Ch05_PlatformTransactionManager 复制 connection_new.xml。

1.  现在，我们需要告诉 Spring 找到所有被@Trasnactional 注解的 bean。通过在 XML 中添加以下配置即可简单完成：

```java
      <tx:annotation-driven /> 

```

1.  要添加上述配置，我们首先要在 XML 中添加'tx'作为命名空间。从 connection_new.xml 更新模式配置如下：

```java
      <beans xmlns="http://www.springframework.org/schema/beans" 
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
        xmlns:context="http://www.springframework.org/schema/context" 
        xmlns:tx="http://www.springframework.org/schema/tx" 
        xsi:schemaLocation="http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans.xsd  
        http://www.springframework.org/schema/context  
        http://www.springframework.org/schema/context/spring-context.xsd  
        http://www.springframework.org/schema/tx  
        http://www.springframework.org/schema/tx/spring-tx.xsd"> 

```

1.  现在，我们可以添加以下配置：

```java
      <tx:annotation-driven /> 

```

1.  复制 MainBookService_operation.java 并执行它以获得输出。

1.  现在添加 addBook()方法以理解 readOnly=true。代码如下：

```java
      @Transactional(readOnly=true) 
      public boolean addBook(Book book) { 
        if (searchBook(book.getISBN()).getISBN() == 98564567l) { 
          System.out.println("no book"); 
          int rows = bookDAO.addBook(book); 

          if (rows > 0) { 
            return true; 
          } 
        } 
        return false;  
      } 

```

1.  执行 MainBookService_operation.java，并执行它以获得以下输出，指定不允许读取事务修改数据：

```java
      Exception in thread "main" 
      org.springframework.dao.TransientDataAccessResourceException:  
      PreparedStatementCallback; SQL [insert into book values(?,?,?,?,?,?)];
      Connection is read-only. Queries leading to data modification are not
      allowed; nested exception is java.sql.SQLException:Connection is read-only.
      Queries leading to data modification are not allowed 

```

1.  编辑 addBook()方法，通过指定 read-only=false 来移除只读事务，这是事务的默认行为。

1.  主要代码将成功执行操作。

### 注意

如果应用程序具有少量事务操作，可以使用 TransactionTemplate 进行程序化事务管理。在拥有大量事务操作的情况下，为了保持简单和一致，选择声明式事务管理。

## 总结

****

在这一章中，我们讨论了事务以及为什么它很重要。我们还讨论了事务管理及其生命周期。我们讨论了事务属性，如只读、隔离级别、传播行为和超时。我们看到了声明性和编程性作为处理事务的两种方式，其中一种使另一种摆脱了管道代码，而另一种提供了对操作的精细控制。我们还通过一个应用程序讨论了这两种技术，以更好地理解。到目前为止，我们讨论了如何处理想象中的数据。我们需要一种方法将其实际地提供给用户。

在下一章中，我们将探索如何开发一个应用程序的 Web 层，以便让我们进行用户交互。
