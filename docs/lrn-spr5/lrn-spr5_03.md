## 第三章。使用 Spring DAO 加速

在第二章节中，我们深入讨论了依赖注入。显然，我们讨论了在配置文件中以及使用注解的各种使用 DI 的方法，但由于缺乏实时应用程序，这些讨论仍然不完整。我们没有其他选择，因为这些都是每个 Spring 框架开发者都应该了解的最重要的基础知识。现在，我们将开始处理数据库，这是应用程序的核心。

我们将讨论以下配置：

+   我们将讨论关于 DataSource 及其使用 JNDI、池化 DataSource 和 JDBCDriver based DataSource 的配置。

+   我们将学习如何使用 DataSource 和 JDBCTemplate 将数据库集成到应用程序中。

+   接下来我们将讨论如何使用 SessionFactory 在应用程序中理解 ORM 及其配置。

+   我们将配置 HibernateTemplate 以使用 ORM 与数据库通信。

+   我们将讨论如何配置缓存管理器以支持缓存数据。

我们都知道，数据库为数据提供了易于结构化的方式，从而可以使用各种方法轻松访问。不仅有许多可用的方法，市场上还有许多不同的数据库。一方面，拥有多种数据库选项是好事，但另一方面，因为每个数据库都需要单独处理，所以这也使得事情变得复杂。在 Java 应用程序的庞大阶段，需要持久性来访问、更新、删除和添加数据库中的记录。JDBC API 通过驱动程序帮助访问这些记录。JDBC 提供了诸如定义、打开和关闭连接，创建语句，通过 ResultSet 迭代获取数据，处理异常等低级别的数据库操作。但是，到了某个点，这些操作变得重复且紧密耦合。Spring 框架通过 DAO 设计模式提供了一种松耦合、高级、干净的解决方案，并有一系列自定义异常。

## Spring 如何处理数据库？

* * *

在 Java 应用程序中，开发者通常使用一个工具类概念来创建、打开和关闭数据库连接。这是一种非常可靠、智能且可重用的连接管理方式，但应用程序仍然与工具类紧密耦合。如果数据库或其连接性（如 URL、用户名、密码或模式）有任何更改，需要在类中进行更改。这需要重新编译和部署代码。在这种情况下，外部化连接参数将是一个好的解决方案。我们无法外部化 Connection 对象，这仍然需要开发者来管理，同样处理它时出现的异常也是如此。Spring 有一种优雅的方式来管理连接，使用位于中心的 DataSource。

### DataSource

数据源是数据源连接的工厂，类似于 JDBC 中的 DriverManager，它有助于连接管理。以下是一些可以在应用程序中使用的实现，以获取连接对象：

+   **DriverManagerDataSource**：该类提供了一个简单的 DataSource 实现，用于在测试或独立应用程序中，每次请求通过 getConnection()获取一个新的 Connection 对象。

+   **SingleConnectionDataSource**：这个类是 SmartDatSource 的一个实现，它提供了一个不会在使用后关闭的单一 Connection 对象。它只适用于单线程应用程序，以及在应用程序服务器之外简化测试。

+   **DataSourceUtils**：这是一个辅助类，它有静态方法用于从 DataSource 获取数据库连接。

+   **DataSourceTransactionManager**：这个类是 PlatformTransactionManager 的一个实现，用于每个数据源的连接。

#### 配置数据源

数据源可以通过以下方式在应用程序中配置：

+   **从 JNDI 查找获取**：Spring 有助于维护在应用程序服务器（如 Glassfish、JBoss、Wlblogic、Tomcat）中运行的大型 Java EE 应用程序。所有这些服务器都支持通过 Java 命名目录接口（JNDI）查找配置数据源池的功能，这有助于提高性能。可以使用以下方式获取在应用程序中配置的数据源：

```java
      <beans xmlns="http://www.springframework.org/schema/beans" 
        xmlns:jee="http://www.springframework.org/schema/jee" 
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
        xsi:schemaLocation="http://www.springframework.org/schema/
        beans  http://www.springframework.org/schema/beans/
        spring-beans.xsd http://www.springframework.org/schema/jee
        http://www.springframework.org/schema/jee/spring-jee.xsd"> 

        <bean id="dataSource"  
          class="org.springframework.jndi.JndiObjectFactoryBean"> 
          <property name="jndiName"   
            value="java:comp/env/jdbc/myDataSource"/> 
        </bean> 

        <jee:jndi-lookup jndi-name="jdbc/myDataSource" id="dataSource" 
          resource-ref="true"/> 
      </beans> 

```

+   其中：

+   **jndi-name**：指定在 JNDI 中配置的服务器上的资源名称。

+   **id**：bean 的 id，提供 DataSource 对象。

+   **resource-ref**：指定是否需要前缀 java:comp/env。

+   从池中获取连接：Spring 没有池化数据源，但是，我们可以配置由 Jakarta Commons Database Connection Pooling 提供的池化数据源。DBCP 提供的 BasicDataSource 可以配置为，

```java
      <bean id="dataSource"               
         class="org.apache.commons.dbcp.BasicDataSource">        
        <property name="driverClassName"                 
          value="org.hsqldb.jdbcDriver"/> 
        <property name="url"    
            value="jdbc:hsqldb:hsql://locahost/name_of_schama"/> 
        <property name="username"      
            value="credential_for_username"/> 
        <property name="password"      
            value="credential_for_password"/> 
        <property name="initialSize"      
            value=""/> 
        <property name="maxActive"      
            value=""/> 
      </bean> 

```

+   其中：

+   **initialSize**：指定了当池启动时应创建的连接数量。

+   **maxActive**：指定可以从池中分配的连接数量。

+   除了这些属性，我们还可以指定等待连接从池中返回的时间（maxWait），连接可以在池中空闲的最大/最小数量（maxIdle/minIdle），可以从语句池分配的最大预处理语句数量（maxOperationPreparedStatements）。

+   使用 JDBC 驱动器：可以利用以下类以最简单的方式获取 DataSource 对象：

* **SingleConnectionDataSource**：正如我们已经在讨论中提到的，它返回一个连接。

* **DriverManagerDataSource**：它在一个请求中返回一个新的连接对象。

+   可以按照以下方式配置 DriverMangerDataSource：

```java
      <bean id="dataSource"
        class="org.springframework.jdbc.datasource.
        DriverManagerDataSource">        
        <property name="driverClassName"                 
          value="org.hsqldb.jdbcDriver"/> 
        <property name="url"    
          value="jdbc:hsqldb:hsql://locahost/name_of_schama"/> 
        <property name="username"      
          value="credential_for_username"/> 
        <property name="password"      
          value="credential_for_password"/> 
      </bean> 

```

### 注意

单连接数据源适用于小型单线程应用程序。驱动管理数据源支持多线程应用程序，但由于管理多个连接，会损害性能。建议使用池化数据源以获得更好的性能。

让我们开发一个使用松耦合模块的示例 demo，以便我们了解 Spring 框架应用程序开发的实际方面。

数据源有助于处理与数据库的连接，因此需要在模块中注入。使用 setter DI 或构造函数 DI 的选择完全由您决定，因为您很清楚这两种依赖注入。我们将使用 setter DI。我们从考虑接口开始，因为这是根据合同进行编程的最佳方式。接口可以有多个实现。因此，使用接口和 Spring 配置有助于实现松耦合模块。我们将声明数据库操作的方法，您可以选择签名。但请确保它们可以被测试。由于松耦合是框架的主要特性，应用程序也将演示为什么我们一直在说松耦合模块可以使用 Spring 轻松编写？您可以使用任何数据库，但本书将使用 MySQL。无论您选择哪种数据库，都要确保在继续之前安装它。让我们按照步骤开始！

##### 案例 1：使用 DriverManagerDataSource 的 XML 配置

1.  创建一个名为 Ch03_DataSourceConfiguration 的核心 Java 应用程序，并添加 Spring 和 JDBC 的 jar，如下所示：

![](img/image_03_001.png)

1.  在 com.ch03.packt.beans 包中创建一个 Book POJO，如下所示：

```java
      public class Book { 
        private String bookName; 
        private long ISBN; 
        private String publication; 
        private int price; 
        private String description; 
        private String [] authors; 

        public Book() { 
          // TODO Auto-generated constructor stub 
          this.bookName="Book Name"; 
          this.ISBN =98564567l; 
          this.publication="Packt Publication"; 
          this.price=200; 
          this.description="this is book in general"; 
          this.author="ABCD"; 
        } 

        public Book(String bookName, long ISBN, String  
          publication,int price,String description,String  
          author)  
       { 
          this.bookName=bookName; 
          this.ISBN =ISBN; 
          this.publication=publication; 
          this.price=price; 
          this.description=description; 
           this.author=author; 
        } 
        // getters and setters 
        @Override 
        public String toString() { 
          // TODO Auto-generated method stub 
          return bookName+"\t"+description+"\t"+price; 
        } 
      }
```

1.  在 com.ch03.packt.dao 包中声明一个 BookDAO 接口。（DAO 表示数据访问对象）。

1.  在数据库中添加书籍的方法如下所示：

```java
      interface BookDAO 
      { 
        public int addBook(Book book); 
      } 

```

1.  为 BookDAO 创建一个实现类 BookDAOImpl，并在类中添加一个 DataSource 类型的数据成员，如下所示：

```java
      private DataSource dataSource; 

```

+   不要忘记使用标准的 bean 命名约定。

1.  由于我们采用 setter 注入，请为 DataSource 编写或生成 setter 方法。

1.  覆盖的方法将处理从 DataSource 获取连接，并使用 PreaparedStatement 将 book 对象插入表中，如下所示：

```java
      public class BookDAOImpl implements BookDAO { 
        private DataSource dataSource; 

        public void setDataSource(DataSource dataSource) { 
          this.dataSource = dataSource; 
        } 

      @Override
      public int addBook(Book book) { 
          // TODO Auto-generated method stub 
          int rows=0; 
          String INSERT_BOOK="insert into book values(?,?,?,?,?,?)"; 
          try { 
            Connection connection=dataSource.getConnection(); 
            PreparedStatement ps=  
                   connection.prepareStatement(INSERT_BOOK); 
            ps.setString(1,book.getBookName()); 
            ps.setLong(2,book.getISBN()); 
            ps.setString(3,book.getPublication()); 
            ps.setInt(4,book.getPrice()); 
            ps.setString(5,book.getDescription()); 
            ps.setString(6,book.getAuthor()); 
            rows=ps.executeUpdate(); 
          } catch (SQLException e) { 
            // TODO Auto-generated catch block 
            e.printStackTrace(); 
          } 
          return rows; 
        } 
      } 

```

1.  在类路径中创建 connection.xml 以配置 beans。

1.  现在问题变成了需要声明多少个 bean 以及它们各自的 id 是什么？

### 注意

一个非常简单的经验法则：首先找出要配置的类，然后找出它的依赖关系是什么？

以下是：

**一个 BookDAOImpl 的 bean。**

**BookDAOImpl 依赖于 DataSource，因此需要一个 DataSource 的 bean。**

您可能会惊讶 DataSource 是一个接口！那么我们是如何创建并注入其对象的呢？是的，这就是我们观点所在！这就是我们所说的松耦合模块。在这里使用的 DataSource 实现是 DriverManagerDataSource。但如果我们直接注入 DriverManagerDataSource，那么类将与它紧密耦合。此外，如果明天团队决定使用其他实现而不是 DriverManagerDataSource，那么代码必须更改，这导致重新编译和重新部署。这意味着更好的解决方案将是使用接口，并从配置中注入其实现。

id 可以是开发者的选择，但不要忽略利用自动装配的优势，然后相应地设置 id。这里我们将使用自动装配'byName'，所以请选择相应的 id。（如果您困惑或想深入了解自动装配，可以参考上一章。）因此，XML 中的最终配置将如下所示：

```java
<bean id="dataSource" 
  class= 
   "org.springframework.jdbc.datasource.DriverManagerDataSource"> 
    <property name="driverClassName"    
        value="com.mysql.jdbc.Driver" /> 
    <property name="url"  
        value="jdbc:mysql://localhost:3306/bookDB" /> 
    <property name="username" value="root" /> 
    <property name="password" value="mysql" /> 
  </bean> 

  <bean id="bookDao" class="com.packt.ch03.dao.BookDAOImpl" 
     autowire="byname"> 
  </bean> 

```

您可能需要根据您的连接参数自定义 URL、用户名和密码。

1.  通常，DAO 层将由服务层调用，但在这里我们不处理它，因为随着应用程序的进行，我们将添加它。由于我们还没有讨论测试，我们将编写带有 main 函数的代码来找出它的输出。main 函数将获取 BookDAO bean 并在其上调用插入 Book 的方法。如果实现代码返回的行值大于零，则书籍成功添加，否则不添加。创建一个名为 MainBookDAO 的类，并向其添加以下代码：

```java
      public static void main(String[] args) { 
        // TODO Auto-generated method stub 
        ApplicationContext context=new  
          ClassPathXmlApplicationContext("connection.xml"); 
        BookDAO bookDAO=(BookDAO) context.getBean("bookDao"); 
        int rows=bookDAO.addBook(new Book("Learning Modular     
          Java Programming", 9781234,"PacktPub   
          publication",800,"explore the power of   
          Modular programming","T.M.Jog"));        
        if(rows>0) 
        { 
          System.out.println("book inserted successfully"); 
        } 
        else 
          System.out.println("SORRY!cannot add book"); 
      }  

```

如果你仔细观察我们会配置 BookDAOImpl 对象，我们是在 BookDAO 接口中接受它，这有助于编写灵活的代码，在这种代码中，主代码实际上不知道是哪个对象在提供实现。

1.  打开你的 MYSQL 控制台，使用凭据登录。运行以下查询创建 BookDB 架构和 Book 表：

![](img/image_03_002.png)

1.  一切都准备好了，执行代码以在控制台获得“book inserted successfully”（书籍插入成功）的提示。你也可以在 MySQL 中运行“select * from book”来获取书籍详情。

##### Case2：使用注解 DriverManagerDataSource

我们将使用在 Case1 中开发的同一个 Java 应用程序 Ch03_DataSourceConfiguration：

1.  声明一个类 BookDAO_Annotation，在 com.packt.ch03.dao 包中实现 BookDAO，并用@Repository 注解它，因为它处理数据库，并指定 id 为'bookDAO_new'。

1.  声明一个类型为 DataSource 的数据成员，并用@Autowired 注解它以支持自动装配。

+   不要忘记使用标准的 bean 命名约定。

+   被覆盖的方法将处理数据库将书籍插入表中。代码将如下所示：

```java
      @Repository(value="bookDAO_new") 
      public class BookDAO_Annotation implements BookDAO { 
        @Autowired 
        private DataSource dataSource; 

        @Override 
        public int addBook(Book book) { 
          // TODO Auto-generated method stub 
          int rows=0; 
          // code similar to insertion of book as shown in     
          Case1\. 
          return rows; 
        } 
      } 

```

1.  我们可以编辑 Case1 中的同一个 connection.xml，但这可能会使配置复杂。所以，让我们在类路径中创建 connection_new.xml，以配置容器考虑注解并搜索如下立体注解：

```java
      <context:annotation-config/> 
 <context:component-scan base- 
        package="com.packt.ch03.*"></context:component-scan> 

      <bean id="dataSource" 
        class="org.springframework.jdbc.datasource.
        DriverManagerDataSo urce"> 
        <!-add properties similar to Case1 -- > 
      </bean> 

```

+   要找出如何添加上下文命名空间和使用注解，请参考第二章。

1.  是时候通过以下代码找到输出：

```java
      public class MainBookDAO_Annotation { 
          public static void main(String[] args) { 
          // TODO Auto-generated method stub 
          ApplicationContext context=new  
            ClassPathXmlApplicationContext("connection_new.xml"); 

          BookDAO bookDAO=(BookDAO) context.getBean("bookDAO_new"); 
          int rows=bookDAO.addBook(new Book("Learning Modular Java  
             Programming", 9781235L,"PacktPub  
             publication",800,"explore the power of  
             Modular programming","T.M.Jog")); 
          if(rows>0) 
          { 
            System.out.println("book inserted successfully"); 
          } 
          else 
          System.out.println("SORRY!cannot add book"); 
        } 
      } 

```

执行代码以将书籍添加到数据库中。

您可能已经注意到，我们从未知道是谁提供了 JDBC 代码的实现，并且由于配置，注入是松耦合的。尽管如此，我们还是能够将数据插入数据库，但仍然需要处理 JDBC 代码，例如获取连接、从它创建语句，然后为表的列设置值，以插入记录。这是一个非常初步的演示，很少在 JavaEE 应用程序中使用。更好的解决方案是使用 Spring 提供的模板类。

### 使用模板类执行 JDBC 操作

模板类提供了一种抽象的方式来定义操作，通过摆脱常见的打开和维护连接、获取 Statement 对象等冗余代码的问题。Spring 提供了许多这样的模板类，使得处理 JDBC、JMS 和事务管理变得比以往任何时候都容易。JdbcTemplate 是 Spring 的这样一个核心组件，它帮助处理 JDBC。要处理 JDBC，我们可以使用以下三种模板之一。

#### JDBCTemplate

JdbcTemplate 帮助开发者专注于应用程序的核心业务逻辑，而无需关心如何打开或管理连接。他们不必担心如果忘记释放连接会怎样？所有这些事情都将由 JdbcTemplate 为您优雅地完成。它提供了指定索引参数以在 SQL 查询中使用 JDBC 操作的方法，就像我们在 PreparedStatements 中通常做的那样。

#### SimpleJdbcTemplate

这与 JDBCTemplate 非常相似，同时具有 Java5 特性的优势，如泛型、可变参数、自动装箱。

#### NamedParameterJdbcTemplate

JdbcTemplate 使用索引来指定 SQL 中参数的值，这可能使记住参数及其索引变得复杂。如果您不习惯数字或需要设置更多参数，我们可以使用 NamedParamterJdbcTemplate，它允许使用命名参数来指定 SQL 中的参数。每个参数都将有一个以冒号(:)为前缀的命名。我们将在开发代码时看到语法。

让我们逐一演示这些模板。

##### 使用 JdbcTemplate

我们将使用与 Ch03_DataSourceConfiguration 中相似的项目结构，并按照以下步骤重新开发它，

1.  创建一个名为 Ch03_JdbcTemplate 的新 Java 应用程序，并添加我们在 Ch03_DataSourceIntegration 中使用的 jar 文件。同时添加 spring-tx-5.0.0.M1.jar。

1.  在 com.packt.ch03.beans 包中创建或复制 Book。

1.  在 com.packt.ch03.dao 包中创建或复制 BookDAO。

1.  在 com.packt.ch03.dao 包中创建 BookDAOImpl_JdbcTemplate，并向其添加 JdbcTemplate 作为数据成员。

1.  分别用@Repository 注解类，用@Autowired 注解数据成员 JdbcTemplate。

1.  覆盖的方法将处理表中书籍的插入。但我们不需要获取连接。同时，我们也不会创建并设置 PreparedStatement 的参数。JdbcTemplate 会为我们完成这些工作。从下面的代码中，事情会变得相当清晰：

```java
      @Repository (value = "bookDAO_jdbcTemplate") 
      public class BookDAO_JdbcTemplate implements BookDAO { 

        @Autowired 
        JdbcTemplate jdbcTemplate; 

        @Override 
        public int addBook(Book book) { 
          // TODO Auto-generated method stub 
          int rows = 0; 
          String INSERT_BOOK = "insert into book  
             values(?,?,?,?,?,?)"; 

          rows=jdbcTemplate.update(INSERT_BOOK, book.getBookName(),                              book.getISBN(), book.getPublication(),   
            book.getPrice(),book.getDescription(),  
            book.getAuthor()); 
          return rows; 
        } 
      } 

```

+   JdbcTemplate 有一个 update()方法，开发人员需要在该方法中传递 SQL 查询以及查询参数的值。因此，我们可以用它来插入、更新和删除数据。其余的都由模板完成。如果你仔细观察，我们没有处理任何异常。我们忘记了吗？不，我们不关心处理它们，因为 Spring 提供了 DataAccessException，这是一个未检查的异常。所以放心吧。在接下来的页面中，我们将讨论 Spring 提供的异常。

+   在代码中添加一个更新书籍价格以及删除书籍的方法。不要忘记首先更改接口实现。代码如下：

```java
      @Override 
      public int updateBook(long ISBN, int price) { 
        // TODO Auto-generated method stub 
        int rows = 0; 
        String UPDATE_BOOK = "update book set price=? where ISBN=?"; 

        rows=jdbcTemplate.update(UPDATE_BOOK, price,ISBN); 
        return rows; 
      } 

      @Override 
      public boolean deleteBook(long ISBN) { 
        // TODO Auto-generated method stub 
        int rows = 0; 
        boolean flag=false; 
        String DELETE_BOOK = "delete from book where ISBN=?"; 

        rows=jdbcTemplate.update(DELETE_BOOK, ISBN); 
        if(rows>0) 
        flag=true; 

        return flag; 
      } 

```

1.  我们在 connection_new.xml 中添加一个 beans 配置文件。你可以简单地从 Ch03_DataSourceIntegration 项目中复制它。我们使用的是 JdbcTemplate，它依赖于 DataSource。因此，我们需要像下面所示配置两个 bean，一个用于 DataSource，另一个用于 JdbcTemplate：

```java
      <context:annotation-config/> 
      <context:component-scan base- 
        package="com.packt.ch03.*"></context:component-scan> 

      <bean id="dataSource" 
        class="org.springframework.jdbc.datasource.
        DriverManagerDataSource"> 
        <property name="driverClassName"  
          value="com.mysql.jdbc.Driver" /> 
        <property name="url"  
          value="jdbc:mysql://localhost:3306/bookDB" /> 
        <property name="username" value="root" /> 
        <property name="password" value="mysql" /> 
      </bean> 

      <bean id="jdbcTemplate"  
        class="org.springframework.jdbc.core.JdbcTemplate"> 
        <property name="dataSource" ref="dataSource"></property> 
      </bean> 

```

1.  编写代码以获取'bookDAO_jdbcTemplate' bean，并在 MainBookDAO_operations 中执行操作，如下所示：

```java
      public class MainBookDAO_operations { 
        public static void main(String[] args) { 
          // TODO Auto-generated method stub 
          ApplicationContext context=new  
            ClassPathXmlApplicationContext("connection_new.xml"); 
          BookDAO bookDAO=(BookDAO)  
            context.getBean("bookDAO_jdbcTemplate"); 
          //add book 
          int rows=bookDAO.addBook(new Book("Java EE 7 Developer  
             Handbook", 97815674L,"PacktPub  
             publication",332,"explore the Java EE7  
             programming","Peter pilgrim")); 
          if(rows>0) 
          { 
            System.out.println("book inserted successfully"); 
          } 
          else 
            System.out.println("SORRY!cannot add book"); 
          //update the book 
          rows=bookDAO.updateBook(97815674L,432); 
          if(rows>0) 
          { 
            System.out.println("book updated successfully"); 
          } 
          else 
            System.out.println("SORRY!cannot update book"); 
          //delete the book 
          boolean deleted=bookDAO.deleteBook(97815674L); 
          if(deleted) 
          { 
            System.out.println("book deleted successfully"); 
          } 
          else 
            System.out.println("SORRY!cannot delete book"); 
        } 
      } 

```

##### 使用 NamedParameterJdbc 模板

我们将使用 Ch03_JdbcTemplates 来添加一个新的类进行此次演示，具体步骤如下。

1.  在 com.packt.ch03.dao 包中添加 BookDAO_NamedParameter 类，它实现了 BookDAO，并用我们之前所做的@Repository 注解。

1.  在其中添加一个 NamedParameterJdbcTemplate 作为数据成员，并用@Autowired 注解它。

1.  使用 update()实现覆盖方法以执行 JDBC 操作。NamedParameterJdbcTemplate 支持在 SQL 查询中给参数命名。找到以下查询以添加书籍：

```java
      String INSERT_BOOK = "insert into book
        values(:bookName,:ISBN,:publication,:price,:description,
        : author)";
```

### 注意

每个参数都必须以前缀冒号：name_of_parameter。

+   如果这些是参数的名称，那么这些参数需要注册，以便框架将它们放置在查询中。为此，我们必须创建一个 Map，其中这些参数名称作为键，其值由开发者指定。以下代码将给出清晰的概念：

```java
      @Repository(value="BookDAO_named") 
      public class BookDAO_NamedParameter implements BookDAO { 

        @Autowired 
        private NamedParameterJdbcTemplate namedTemplate; 

        @Override 
        public int addBook(Book book) { 
          // TODO Auto-generated method stub 
          int rows = 0; 
          String INSERT_BOOK = "insert into book  
            values(:bookName,:ISBN,:publication,:price, 
            :description,:author)"; 
          Map<String,Object>params=new HashMap<String,Object>(); 
          params.put("bookName", book.getBookName()); 
          params.put("ISBN", book.getISBN()); 
          params.put("publication", book.getPublication()); 
          params.put("price",book.getPrice()); 
          params.put("description",book.getDescription()); 
          params.put("author", book.getAuthor()); 
          rows=namedTemplate.update(INSERT_BOOK,params);  

          return rows; 
        } 

        @Override 
        public int updateBook(long ISBN, int price) { 
          // TODO Auto-generated method stub 
          int rows = 0; 
          String UPDATE_BOOK =  
           "update book set price=:price where ISBN=:ISBN"; 

          Map<String,Object>params=new HashMap<String,Object>(); 
          params.put("ISBN", ISBN); 
          params.put("price",price); 
          rows=namedTemplate.update(UPDATE_BOOK,params); 
          return rows; 
        } 

        @Override 
        public boolean deleteBook(long ISBN) { 
          // TODO Auto-generated method stub 
          int rows = 0; 
          boolean flag=false; 
          String DELETE_BOOK = "delete from book where ISBN=:ISBN"; 

          Map<String,Object>params=new HashMap<String,Object>(); 
          params.put("ISBN", ISBN); 
          rows=namedTemplate.update(DELETE_BOOK, params); 
          if(rows>0) 
            flag=true; 
          return flag; 
        } 
      } 

```

1.  在 connection_new.xml 中为 NamedParameterJdbcTemplate 添加一个 bean，如下所示：

```java
      <bean id="namedTemplate" 
        class="org.springframework.jdbc.core.namedparam. 
          NamedParameterJdbcTemplate">   
        <constructor-arg ref="dataSource"/> 
      </bean> 

```

+   在其他所有示例中我们都使用了 setter 注入，但在这里我们无法使用 setter 注入，因为该类没有默认构造函数。所以，只使用构造函数依赖注入。

1.  使用开发的 MainBookDAO_operations.java 来测试 JdbcTemplate 的工作。你只需要更新将获取**BookDAO_named** bean 以执行操作的语句。更改后的代码将是：

```java
      BookDAO bookDAO=(BookDAO) context.getBean("BookDAO_named"); 

```

+   你可以在 MainBookDAO_NamedTemplate.java 中找到完整的代码。

1.  执行代码以获取成功消息。

在小型 Java 应用程序中，代码将具有较少的 DAO 类。因此，对于每个 DAO 使用模板类来处理 JDBC，对开发人员来说不会很复杂。这也导致了代码的重复。但是，当处理具有更多类的企业应用程序时，复杂性变得难以处理。替代方案将是，不是在每个 DAO 中注入模板类，而是选择一个具有模板类能力的父类。Spring 具有 JdbcDaoSupport，NamedParameterJdbcSupport 等支持性 DAO。这些抽象支持类提供了一个公共基础，避免了代码的重复，在每个 DAO 中连接属性。

让我们继续同一个项目使用支持 DAO。我们将使用 JdbcDaoSupport 类来了解实际方面：

1.  在 com.packt.ch03.dao 中添加 BookDAO_JdbcTemplateSupport.java，它继承了 JdbcDaoSupport 并实现了 BookDAO。

1.  从接口覆盖方法，这些方法将处理数据库。BookDAO_JdbcTemplateSupport 类从 JdbcDaoSupport 继承了 JdbcTemplate。所以代码保持不变，就像我们使用 JdbcTemplate 时一样，稍作改动。必须通过下面的代码中加粗的 getter 方法访问 JdbcTemplate：

```java
      @Repository(value="daoSupport") 
      public class BookDAO_JdbcTemplateSupport extends JdbcDaoSupport  
        implements BookDAO 
      { 
        @Autowired 
        public BookDAO_JdbcTemplateSupport(JdbcTemplate jdbcTemplate) 
        { 
          setJdbcTemplate(jdbcTemplate); 
        } 

        @Override 
        public int addBook(Book book) { 
          // TODO Auto-generated method stub 
          int rows = 0; 
          String INSERT_BOOK = "insert into book values(?,?,?,?,?,?)"; 

          rows=getJdbcTemplate().update(INSERT_BOOK,  
            book.getBookName(), book.getISBN(),  
            book.getPublication(), book.getPrice(), 
            book.getDescription(), book.getAuthor()); 

          return rows; 
        } 

        @Override 
        public int updateBook(long ISBN, int price) { 
          // TODO Auto-generated method stub 
          int rows = 0; 
          String UPDATE_BOOK = "update book set price=? where ISBN=?"; 

          rows=getJdbcTemplate().update(UPDATE_BOOK, price,ISBN); 
          return rows; 
        } 

        @Override 
        public boolean deleteBook(long ISBN) { 
          // TODO Auto-generated method stub 
          int rows = 0; 
          boolean flag=false; 
          String DELETE_BOOK = "delete from book where ISBN=?"; 

          rows=getJdbcTemplate().update(DELETE_BOOK, ISBN); 
          if(rows>0) 
            flag=true; 

          return flag; 
        } 
      } 

```

### 注意

使用 DAO 类时，依赖将通过构造函数注入。

我们之前在几页中讨论了关于简短处理异常的内容。让我们更详细地了解它。JDBC 代码强制通过检查异常处理异常。但是，它们是泛化的，并且仅通过 DataTrucationException，SQLException，BatchUpdateException，SQLWarning 处理。与 JDBC 相反，Spring 支持各种未检查异常，为不同场景提供专门的信息。以下表格显示了我们可能需要频繁使用的其中一些：

| **Spring 异常** | **它们什么时候被抛出？** |
| --- | --- |
| 数据访问异常 | 这是 Spring 异常层次结构的根，我们可以将其用于所有情况。 |
| 权限被拒数据访问异常 | 当尝试在没有正确授权的情况下访问数据时 |
| 空结果数据访问异常 | 从数据库中没有返回任何行，但至少期望有一个。 |
| 结果大小不匹配数据访问异常 | 当结果大小与期望的结果大小不匹配时。 |
| 类型不匹配数据访问异常 | Java 和数据库之间的数据类型不匹配。 |
| 无法获取锁异常 | 在更新数据时未能获取锁 |
| 数据检索失败异常 | 当使用 ORM 工具通过 ID 搜索和检索数据时 |

在使用 Spring DataSource，模板类，DAOSupport 类处理数据库操作时，我们仍然涉及使用 SQL 查询进行 JDBC 操作，而不进行面向对象的操作。处理数据库操作的最简单方法是使用对象关系映射将对象置于中心。

## 对象关系映射

* * *

JDBC API 提供了执行关系数据库操作的手段以实现持久化。Java 开发人员积极参与编写 SQL 查询以进行此类数据库操作。但是 Java 是一种面向对象编程语言（OOP），而数据库使用 **顺序查询语言**（**SQL**）。OOP 的核心是对象，而 SQL 有数据库。OOP 没有主键概念，因为它有身份。OOP 使用继承，但 SQL 没有。这些以及许多其他不匹配之处使得没有深入了解数据库及其结构的情况下，JDBC 操作难以执行。一些优秀的 ORM 工具已经提供了解决方案。ORM 处理数据库操作，将对象置于核心位置，而无需开发者处理 SQL。市场上的 iBATIS、JPA 和 Hibernate 都是 ORM 框架的例子。

### Hibernate

Hibernate 是开发者在 ORM 解决方案中使用的高级中间件工具之一。它以一种简单的方式提供了细粒度、继承、身份、关系关联和导航问题的解决方案。开发者无需手动编写 SQL 查询，因为 Hibernate 提供了丰富的 API 来处理 CRUD 数据库操作，使得系统更易于维护和开发。SQL 查询依赖于数据库，但在 Hibernate 中无需编写 SQL 语句，因为它提供了数据库无关性。它还支持 Hibernate 查询语言（HQL）和原生 SQL 支持，通过编写查询来自定义数据库操作。使用 Hibernate，开发者可以缩短开发时间，从而提高生产力。

#### Hibernate 架构

下面的图表展示了 Hibernate 的架构及其中的接口：

![](img/image_03_003.png)

**Hibernate** 拥有 Session、SessionFactory、Configuration、Transaction、Query 和 Criteria 接口，为核心提供了 ORM 支持，帮助开发者进行对象关系映射。

##### Configuration 接口

使用 Configuration 实例来指定数据库属性，如 URL、用户名和密码，映射文件的路径或包含数据成员与表及其列映射信息的类。这个 Configuration 实例随后用于获取 SessionFactory 的实例。

##### SessionFactory 接口

SessionFactory 是重量级的，每个应用程序通常只有一个实例。但是有时一个应用程序会使用多个数据库，这就导致每个数据库都有一个实例。SessionFactory 用于获取 Session 的实例。它非常重要，因为它缓存了生成的 SQL 语句和数据，这些是 Hibernate 在一个工作单元内运行时使用的，作为第一级缓存。

##### Session 接口

Session 接口是每个使用 Hibernate 的应用程序用来执行数据库操作的基本接口，这些接口是从 SessionFactory 获取的。Session 是轻量级、成本低的组件。因为 SessionFactory 是针对每个应用程序的，所以开发者为每个请求创建一个 Session 实例。

##### Transaction 接口

事务帮助开发人员将多个操作作为工作单元。JTA、JDBC 提供事务实现的实现。除非开发者提交事务，否则数据不会反映在数据库中。

##### 查询接口

查询接口提供使用 Hibernate 查询语言（HQL）或原生 SQL 来执行数据库操作的功能。它还允许开发人员将值绑定到 SQL 参数，指定查询返回多少个结果。

##### 条件接口

条件接口与查询接口类似，允许开发人员编写基于某些限制或条件的条件查询对象以获取结果。

在 Spring 框架中，开发者可以选择使用 SessionFactory 实例或 HibernateTemplate 进行 Hibernate 集成。SessionFactory 从数据库连接参数配置和映射位置获取，然后使用 DI 可以在 Spring 应用程序中使用。`SessionFactory`可以如下配置：

```java
<bean id="sessionFactory" 
    class="org.springframework.orm.hibernate5.LocalSessionFactoryBean"> 
    <property name="dataSource" ref="dataSource" /> 
    <property name="mappingResources"> 
      <list> 
        <value>book.hbm.xml</value> 
      </list> 
    </property> 
    <property name="hibernateProperties"> 
      <props> 
        <prop key=    
          "hibernate.dialect">org.hibernate.dialect.MySQLDialect 
        </prop> 
        <prop key="hibernate.show_sql">true</prop> 
        <prop key="hibernate.hbm2ddl.auto">update</prop> 
      </props> 
    </property> 
  </bean> 

```

+   **dataSource** - 提供数据库属性的信息。

+   **mappingResource** - 指定提供数据成员到表及其列映射信息的文件名称。

+   **hibernateProperties** - 提供关于 hibernate 属性的信息

**方言** - 它用于生成符合底层数据库的 SQL 查询。

**show_sql** - 它显示框架在控制台上发出的 SQL 查询。

**hbm2ddl.auto** - 它提供了是否创建、更新表以及要执行哪些操作的信息。

在使用 SessionFactory 时，开发人员不会编写使用 Spring API 的代码。但我们之前已经讨论过模板类。HibenateTemplate 是这样一个模板，它帮助开发人员编写松耦合的应用程序。HibernateTemplate 配置如下：

```java
<bean id="hibernateTemplate" \
  class="org.springframework.orm.hibernate5.HibernateTemplate"> 
    <property name="sessionFactory" ref="sessionFactory"></property> 
</bean> 

```

让我们按照以下步骤逐一将 SessionFactory 集成到我们的 Book 项目中。

##### 案例 1：使用 SessionFactory

1.  创建一个名为 Ch03_Spring_Hibernate_Integration 的 Java 应用程序，并添加 Spring、JDBC 和 hibernate 的 jar 文件，如下 outline 所示：

![](img/image_03_004.png)

+   您可以从 Hibernate 的官方网站下载包含 hibernate 框架 jar 的 zip 文件。

1.  在 com.packt.ch03.beans 包中复制或创建 Book.java。

1.  在类路径中创建 book.hbm.xml，将 Book 类映射到 book_hib 表，如下配置所示：

```java
      <hibernate-mapping> 
        <class name="com.packt.ch03.beans.Book" table="book_hib"> 
          <id name="ISBN" type="long"> 
            <column name="ISBN" /> 
            <generator class="assigned" /> 
          </id> 
          <property name="bookName" type="java.lang.String"> 
            <column name="book_name" /> 
          </property>               
          <property name="description" type="java.lang.String"> 
            <column name="description" /> 
          </property> 
          <property name="author" type="java.lang.String"> 
            <column name="book_author" /> 
          </property> 
          <property name="price" type="int"> 
            <column name="book_price" /> 
          </property> 
        </class> 

      </hibernate-mapping>
```

+   其中标签配置如下：

**id** - 定义从表到图书类的主键映射

**属性** - 提供数据成员到表中列的映射

1.  像在 Ch03_JdbcTemplate 应用程序中一样添加 BookDAO 接口。

1.  通过 BookDAO_SessionFactory 实现 BookDAO 并覆盖方法。用@Repository 注解类。添加一个类型为 SessionFactory 的数据成员，并用@Autowired 注解。代码如下所示：

```java
      @Repository(value = "bookDAO_sessionFactory") 
      public class BookDAO_SessionFactory implements BookDAO { 

        @Autowired 
        SessionFactory sessionFactory; 

        @Override 
        public int addBook(Book book) { 
          // TODO Auto-generated method stub 
          Session session = sessionFactory.openSession(); 
          Transaction transaction = session.beginTransaction(); 
          try { 
            session.saveOrUpdate(book); 
            transaction.commit(); 
            session.close(); 
            return 1; 
          } catch (DataAccessException exception) { 
            exception.printStackTrace(); 
          } 
          return 0; 
        } 

        @Override 
        public int updateBook(long ISBN, int price) { 
          // TODO Auto-generated method stub 
          Session session = sessionFactory.openSession(); 
          Transaction transaction = session.beginTransaction(); 
          try { 
            Book book = session.get(Book.class, ISBN); 
            book.setPrice(price); 
            session.saveOrUpdate(book); 
            transaction.commit(); 
            session.close(); 
            return 1; 
          } catch (DataAccessException exception) { 
            exception.printStackTrace(); 
          } 
          return 0; 
        } 

        @Override 
        public boolean deleteBook(long ISBN) { 
          // TODO Auto-generated method stub 
          Session session = sessionFactory.openSession(); 
          Transaction transaction = session.beginTransaction(); 
          try { 
            Book book = session.get(Book.class, ISBN); 
            session.delete(book); 
            transaction.commit(); 
            session.close(); 
            return true; 
          } catch (DataAccessException exception) { 
            exception.printStackTrace(); 
          } 
          return false; 
        } 
      } 

```

1.  添加 connection_new.xml 以配置 SessionFactory 和其他详细信息，如下所示：

```java
      <context:annotation-config /> 
      <context:component-scan base-package="com.packt.ch03.*"> 
      </context:component-scan> 

      <bean id="dataSource" 
        class="org.springframework.jdbc.datasource. 
        DriverManagerDataSource"> 
        <!-properties for dataSourceà 
      </bean> 

      <bean id="sessionFactory" class=  
        "org.springframework.orm.hibernate5.LocalSessionFactoryBean"> 
        <property name="dataSource" ref="dataSource" /> 
        <property name="mappingResources"> 
          <list> 
            <value>book.hbm.xml</value> 
          </list> 
        </property> 
        <property name="hibernateProperties"> 
          <props> 
            <prop key=      
              "hibernate.dialect">org.hibernate.dialect.MySQLDialect 
            </prop> 
            <prop key="hibernate.show_sql">true</prop> 
            <prop key="hibernate.hbm2ddl.auto">update</prop> 
          </props> 
        </property> 
      </bean> 

```

1.  将 MainBookDAO_operations.java 创建或复制以获取 bean 'bookDAO_sessionFactory'以测试应用程序。代码将是：

```java
      public static void main(String[] args) { 
       // TODO Auto-generated method stub 
       ApplicationContext context=new  
         ClassPathXmlApplicationContext("connection_new.xml"); 
       BookDAO bookDAO=(BookDAO)  
         context.getBean("bookDAO_sessionFactory"); 
       //add book
       int rows=bookDAO.addBook(new Book("Java EE 7 Developer  
         Handbook", 97815674L,"PacktPub  
         publication",332,"explore the Java EE7  
         programming","Peter pilgrim")); 
       if(rows>0) 
       { 
         System.out.println("book inserted successfully"); 
       } 
       else
        System.out.println("SORRY!cannot add book"); 

      //update the book
      rows=bookDAO.updateBook(97815674L,432); 
      if(rows>0) 
      { 
        System.out.println("book updated successfully"); 
      }
      else
        System.out.println("SORRY!cannot update book"); 
        //delete the book
        boolean deleted=bookDAO.deleteBook(97815674L); 
        if(deleted) 
        { 
          System.out.println("book deleted successfully"); 
        }
        else
          System.out.println("SORRY!cannot delete book"); 
      } 

```

我们已经看到了如何配置 HibernateTemplate 在 XML 中。它与事务广泛工作，但我们还没有讨论过什么是事务，它的配置以及如何管理它？我们将在接下来的几章中讨论它。

实时应用在每个步骤中处理大量数据。比如说我们想要找一本书。使用 hibernate 我们只需调用一个返回书籍的方法，这个方法取决于书籍的 ISBN。在日常生活中，这本书会被搜索无数次，每次数据库都会被访问，导致性能问题。相反，如果有一种机制，当再次有人索求这本书时，它会使用前一次查询的结果，那就太好了。Spring 3.1 引入了有效且最简单的方法——'缓存'机制来实现它，并在 4.1 中添加了 JSR-107 注解支持。缓存的结果将存储在缓存库中，下次将用于避免不必要的数据库访问。你可能想到了缓冲区，但它与缓存不同。缓冲区是用于一次性写入和读取数据的临时中间存储。但缓存是为了提高应用程序的性能而隐藏的，数据在这里被多次读取。

缓存库是对象从数据库获取后保存为键值对的位置。Spring 支持以下库，

**基于 JDK 的 ConcurrentMap 缓存：**

在 JDK ConcurrentMap 中用作后端缓存存储。Spring 框架具有 SimpleCacheManager 来获取缓存管理器并给它一个名称。这种缓存最适合相对较小的数据，这些数据不经常更改。但它不能用于 Java 堆之外存储数据，而且没有内置的方法可以在多个 JVM 之间共享数据。

**基于 EhCache 的缓存：**

EhcacheChacheManager 用于获取一个缓存管理器，其中配置 Ehcache 配置规范，通常配置文件名为 ehcache.xml。开发者可以为不同的数据库使用不同的缓存管理器。

**Caffeine 缓存：**

Caffeine 是一个基于 Java8 的缓存库，提供高性能。它有助于克服 ConcurrentHashMap 的重要缺点，即它直到显式移除数据才会持久化。除此之外，它还提供数据的自动加载、基于时间的数据过期以及被驱逐的数据条目的通知。

Spring 提供了基于 XML 以及注解的缓存配置。最简单的方法是使用注解 based 配置。从 Spring 3.1 开始，版本已启用 JSR-107 支持。为了利用 JSR-107 的缓存，开发人员需要首先进行缓存声明，这将帮助识别要缓存的方法，然后配置缓存以通知数据存储在哪里。

#### 缓存声明

缓存声明可以使用注解以及基于 XML 的方法。以下开发人员可以使用注解进行声明：

##### `@Cacheable`:

该注解用于声明这些方法的结果将被存储在缓存中。它带有与之一致的缓存名称。每次开发人员调用方法时，首先检查缓存以确定调用是否已经完成。

##### `@Caching`:

当需要在同一个方法上嵌套多个 `@CacheEvict`、`@CachePut` 注解时使用该注解。

##### `@CacheConfig`:

使用注解 `@CacheConfig` 来注解类。对于使用基于缓存的注解 annotated 的类方法，每次指定缓存名称。如果类有多个方法，则使用 `@CacheConfig` 注解允许我们只指定一次缓存名称。

##### `@CacheEvict`:

用于从缓存区域删除未使用数据。

##### `@CachePut`

该注解用于在每次调用被其注解的方法时更新缓存结果。该注解的行为与 `@Cacheable` 正好相反，因为它强制调用方法以更新缓存，而 `@Cacheable` 跳过执行。

#### 缓存配置：

首先，为了启用基于注解的配置，Spring 必须使用缓存命名空间进行注册。以下配置可用于声明缓存命名空间并注册注解：

```java
<beans xmlns="http://www.springframework.org/schema/beans" 
  xmlns:cache="http://www.springframework.org/schema/cache" 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd  
  http://www.springframework.org/schema/cache 
  http://www.springframework.org/schema/cache/spring-cache.xsd"> 
        <cache:annotation-driven /> 
</beans> 

```

注册完成后，现在是提供配置以指定存储库的名称以及使用哪个缓存管理器存储库结果的时候了。我们将在 `SimpleCacheManager` 的示例演示中很快定义这些配置。

让我们在我们的 Book 应用程序中集成 JDK 基于的 `ConcurrentMap` 存储库。我们将使用 `Ch03_Spring_Hibernate_Integration` 作为演示的基础项目。按照集成的步骤操作，

1.  创建一个名为 `Ch03_CacheManager` 的新 Java 应用程序，并添加 Spring、JDBC 和 hibernate 的 jar 文件。你也可以参考 `Ch03_Spring_Hibernate_Integration` 应用程序。

1.  在 `com.packt.ch03.beans` 包中创建或复制 `Book.java` 文件。

1.  在 `com.packt.ch03.dao` 包中创建或复制 `BookDAO` 接口，并向其添加一个使用 ISBN 从数据库中搜索书籍的方法。该方法的签名如下所示：

```java
      public Book getBook(long ISBN); 

```

1.  在 `BookDAO_SessionFactory_Cache` 中实现方法，正如我们在 Hibernate 应用程序中的 `BookDAO_SessionFactory.java` 中所做的那样。从数据库获取书籍的方法将是：

```java
      public Book getBook(long ISBN) { 
        // TODO Auto-generated method stub 
        Session session = sessionFactory.openSession(); 
        Transaction transaction = session.beginTransaction(); 
        Book book = null; 
        try { 
          book = session.get(Book.class, ISBN); 
          transaction.commit(); 
          session.close(); 
        } catch (DataAccessException exception) { 
          exception.printStackTrace(); 
          book; 
      } 

```

该方法将使用'repo'存储库来缓存结果。

1.  将 book.hbm.xml 复制到类路径中。

1.  添加带有主函数的 MainBookDAO_Cache.java，以从数据库获取数据，但故意我们会如下的获取两次数据：

```java
      public static void main(String[] args) { 
        // TODO Auto-generated method stub 
        ApplicationContext context=new  
          ClassPathXmlApplicationContext("connection_new.xml"); 
        BookDAO bookDAO=(BookDAO)   
          context.getBean("bookDAO_sessionFactory"); 
        Book book=bookDAO.getBook(97815674L);    

        System.out.println(book.getBookName()+ 
          "\t"+book.getAuthor()); 
        Book book1=bookDAO.getBook(97815674L); 
        System.out.println(book1.getBookName()+ 
          "\t"+book1.getAuthor()); 
      } 

```

1.  在执行之前，请确保我们要搜索的 ISBN 已经存在于数据库中。我们将得到以下输出：

```java
      Hibernate: select book0_.ISBN as ISBN1_0_0_, book0_.book_name as        book_nam2_0_0_, book0_.description as descript3_0_0_,
      book0_.book_author as book_aut4_0_0_, book0_.book_price as
      book_pri5_0_0_ from book_hib book0_ where book0_.ISBN=? 
      book:-Java EE 7 Developer Handbook  Peter pilgrim 

      Hibernate: select book0_.ISBN as ISBN1_0_0_, book0_.book_name as        book_nam2_0_0_, book0_.description as descript3_0_0_,  
      book0_.book_author as book_aut4_0_0_, book0_.book_price as 
      book_pri5_0_0_ from book_hib book0_ where book0_.ISBN=? 
      book1:-Java EE 7 Developer Handbook  Peter pilgrim 

```

上述输出清楚地显示了搜索书籍的查询执行了两次，表示数据库被访问了两次。

现在让我们配置 Cache manager 以缓存搜索书籍的结果，如下所示，

1.  使用@Cacheable 注解来标记那些结果需要被缓存的方法，如下所示：

```java
      @Cacheable("repo") 
      public Book getBook(long ISBN) {// code will go here } 

```

1.  在 connection_new.xml 中配置缓存命名空间，正如我们已经在讨论中提到的。

1.  在 XML 中注册基于注解的缓存，如下所示：

```java
      <cache:annotation-driven /> 

```

1.  为设置存储库为'repo'添加 CacheManger，如下配置所示：

```java
      <bean id="cacheManager"  
        class="org.springframework.cache.support.SimpleCacheManager"> 
        <property name="caches"> 
          <set> 
            <bean class="org.springframework.cache.concurrent.
              ConcurrentMapCache FactoryBean"> 
              <property name="name" value="repo"></property> 
            </bean> 
          </set> 
        </property> 
      </bean> 

```

1.  不更改地执行 MainBookDAO_Cache.java 以得到以下输出：

```java
      Hibernate: select book0_.ISBN as ISBN1_0_0_, book0_.book_name as        book_nam2_0_0_, book0_.description as descript3_0_0_,  
      book0_.book_author as book_aut4_0_0_, book0_.book_price as 
      book_pri5_0_0_ from book_hib book0_ where book0_.ISBN=? 
      book:-Java EE 7 Developer Handbook  Peter pilgrim 

      book1:-Java EE 7 Developer Handbook  Peter pilgrim 

```

控制台输出显示，即使我们两次搜索了书籍，查询也只执行了一次。由`getBook()`第一次获取的书籍结果被缓存起来，下次有人请求这本书而没有加热数据库时，会使用这个缓存结果。

## 总结

****

在本章中，我们深入讨论了持久层。讨论使我们了解了如何通过 Spring 使用 DataSource 将 JDBC 集成到应用程序中。但使用 JDBC 仍然会让开发者接触到 JDBC API 及其操作，如获取 Statement、PreparedStatement 和 ResultSet。但 JdbcTemplate 和 JdbcDaoSupport 提供了一种在不涉及 JDBC API 的情况下执行数据库操作的方法。我们还看到了 Spring 提供的异常层次结构，可以根据应用程序的情况使用它。我们还讨论了 Hibernate 作为 ORM 工具及其在框架中的集成。缓存有助于最小化对数据库的访问并提高性能。我们讨论了缓存管理器以及如何将 CacheManger 集成到应用程序中。

在下一章中，我们将讨论面向方面的编程，它有助于处理交叉技术。
