# 第四章：实现微服务

本章将带你从设计阶段到我们示例项目的实现——一个**在线预订餐桌系统**（**OTRS**）。在这里，你将使用上一章中解释的相同设计并将其扩展以构建微服务。在本章结束时，你不仅学会了如何实现设计，还学会了微服务的不同方面——构建、测试和打包。虽然重点是构建和实现 Restaurant 微服务，但你也可以用相同的方法来构建和实现 OTRS 中使用的其他微服务。

在本章中，我们将介绍以下主题：

+   OTRS 概览

+   开发和实现微服务

+   测试

我们将使用上一章中展示的领域驱动设计的关键概念。在上一章中，你看到了如何使用核心 Java 开发领域模型。现在，我们将从示例领域实现转向 Spring Framework 驱动的实现。你将利用 Spring Boot 来实现领域驱动设计概念，并将它们从核心 Java 转换为基于 Spring Framework 的模型。

此外，我们还将使用 Spring Cloud，它提供了一个通过 Spring Boot 可用的云就绪解决方案。Spring Boot 将允许你使用依赖于 Tomcat 或 Jetty 的内嵌应用程序容器，你的服务被包装为 JAR 或 WAR。这个 JAR 作为一个独立的进程执行，一个微服务，将服务于提供对所有请求的响应，并指向服务中定义的端点。

Spring Cloud 也可以轻松集成 Netflix Eureka，一个服务注册和发现组件。OTRS 将使用它进行注册和微服务的发现。

# OTRS 概览

基于微服务原则，我们需要为每个功能分别拥有独立的微服务。在查看 OTRS 之后，我们可以很容易地将其划分为三个主要微服务——Restaurant 服务、预订服务和用户服务。在 OTRS 中还可以定义其他微服务。我们的重点是这三个微服务。想法是使它们独立，包括拥有自己的独立数据库。

我们可以如下总结这些服务的功能：

+   **餐厅服务**：这个服务提供了对餐厅资源的功能——**创建**、**读取**、**更新**、**删除**（**CRUD**）操作和基于标准的选择。它提供了餐厅和餐桌之间的关联。餐厅也会提供对`Table`实体的访问。

+   **用户服务**：这个服务，如名字所示，允许终端用户对用户实体执行 CRUD 操作。

+   **预订服务**：该服务利用餐厅服务和用户服务执行预订的 CRUD 操作。它将基于指定时间段的餐桌可用性进行餐厅搜索及其相关表格的查找和分配。它创建了餐厅/餐桌与用户之间的关系：

![](img/7477a84d-be2b-4e19-88ec-d74b29e6b4c3.jpg)

微服务的注册和发现

前述图表展示了每个微服务如何独立工作。这就是微服务可以独立开发、改进和维护的原因，而不会影响其他服务。这些服务可以具有自己的分层架构和数据库。没有限制要求使用相同的技术、框架和语言来开发这些服务。在任何给定的时间点，您还可以引入新的微服务。例如，出于会计目的，我们可以引入一个会计服务，可以向餐厅提供簿记服务。同样，分析报告也是其他可以集成和暴露的服务。

出于演示目的，我们将只实现前述图表中显示的三个服务。

# 开发和实现微服务

我们将使用前章描述的领域驱动实现和方法来使用 Spring Cloud 实现微服务。让我们回顾一下关键工件：

+   **实体**：这些是可识别且在产品/服务状态中保持不变的对象类别。这些对象*不是*由它们的属性定义，而是由它们的标识和连续性线定义。实体具有诸如标识、连续性线和不会定义它们身份的属性等特征。

+   **值对象**（**VOs**）仅包含属性，没有概念上的身份。最佳实践是保持 VOs 作为不可变对象。在 Spring 框架中，实体是纯粹的 POJOs；因此，我们也将使用它们作为 VOs。

+   **服务对象**：这些在技术框架中很常见。在领域驱动设计中，这些也用于领域层。服务对象没有内部状态；它的唯一目的是向领域提供行为。服务对象提供不能与特定实体或 VOs 相关联的行为。服务对象可能向一个或多个实体或 VOs 提供一个或多个相关行为。在领域模型中明确定义服务是最佳实践。

+   **仓库对象**：仓库对象是领域模型的一部分，它与存储（如数据库、外部源等）交互，以检索持久化的对象。当接收到仓库中对象的引用请求时，它返回现有的对象引用。如果请求的对象在仓库中不存在，那么它从存储中检索该对象。

下载示例代码：详细的步骤说明在本书的前言中提到。请查看。本书的代码包也托管在 GitHub 上，地址为：[`github.com/PacktPublishing/Mastering-Microservices-with-Java`](https://github.com/PacktPublishing/Mastering-Microservices-with-Java-9-Second-Edition)。我们还有其他来自我们丰富的书籍和视频目录的代码包，地址为：[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。查看它们！

每个 OTRS 微服务 API 代表一个 RESTful web 服务。OTRS API 使用 HTTP 动词（如`GET`、`POST`等），以及 RESTful 端点结构。请求和响应负载格式化为 JSON。如果需要，也可以使用 XML。

# 餐厅微服务

餐厅微服务将通过 REST 端点暴露给外部世界进行消费。在餐厅微服务示例中，我们会找到以下端点。根据需求可以添加尽可能多的端点：

1.  获取餐厅 ID 的端点：

![](img/6dd6acef-2a4c-420e-b962-a11cbb7261f0.png)

1.  获取匹配查询参数`Name`的所有餐厅的端点：

![](img/4967fca0-9a41-47dc-ba3f-14de1854d494.png)

1.  创建新餐厅的端点：

![](img/c3956c15-b2bc-460b-b4a9-0904194f97c6.png)

同样，我们可以添加各种端点及其实现。为了演示目的，我们将使用 Spring Cloud 实现上述端点。

# OTRS 实现

我们将创建一个多模块的 Maven 项目来实现 OTRS。以下堆栈将用于开发 OTRS 应用程序。请注意，在撰写本书时，只有 Spring Boot 和 Cloud 的快照构建可用。因此，在最终发布中，可能会有一个或两个变化：

+   Java 版本 1.9

+   Spring Boot 2.0.0.M1

+   Spring Cloud Finchley.M2

+   Maven Compiler Plugin 3.6.1（用于 Java 1.9）

上述所有点都在根`pom.xml`中提到，还包括以下 OTRS 模块：

+   `eureka-service`

+   `restaurant-service`

+   `user-service`

+   `booking-service`

根`pom.xml`文件将如下所示：

```java
<?xml version="1.0" encoding="UTF-8"?> 
<project   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"> 
    <modelVersion>4.0.0</modelVersion> 

    <groupId>com.packtpub.mmj</groupId> 
    <artifactId>6392_chapter4</artifactId> 
    <version>PACKT-SNAPSHOT</version> 
    <name>6392_chapter4</name> 
    <description>Master Microservices with Java Ed 2, Chapter 4 - Implementing Microservices</description> 

    <packaging>pom</packaging> 
    <properties> 
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding> 
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding> 
        <java.version>1.9</java.version> 
        <maven.compiler.source>1.9</maven.compiler.source> 
        <maven.compiler.target>1.9</maven.compiler.target> 
    </properties> 

    <parent> 
        <groupId>org.springframework.boot</groupId> 
        <artifactId>spring-boot-starter-parent</artifactId> 
        <version>2.0.0.M1</version> 
    </parent> 
    <dependencyManagement> 
        <dependencies> 
            <dependency> 
                <groupId>org.springframework.cloud</groupId> 
                <artifactId>spring-cloud-dependencies</artifactId> 
                <version>Finchley.M2</version> 
                <type>pom</type> 
                <scope>import</scope> 
            </dependency> 
        </dependencies> 
    </dependencyManagement> 

    <modules> 
        <module>eureka-service</module> 
        <module>restaurant-service</module> 
        <module>booking-service</module> 
        <module>user-service</module> 
    </modules> 

    <!-- Build step is required to include the spring boot artifacts in generated jars --> 
    <build> 
        <finalName>${project.artifactId}</finalName> 
        <plugins> 
            <plugin> 
                <groupId>org.springframework.boot</groupId> 
                <artifactId>spring-boot-maven-plugin</artifactId> 
            </plugin> 
            <plugin> 
                <groupId>org.apache.maven.plugins</groupId> 
                <artifactId>maven-compiler-plugin</artifactId> 
                <version>3.6.1</version> 
                <configuration> 
                    <source>1.9</source> 
                    <target>1.9</target> 
                    <showDeprecation>true</showDeprecation> 
                    <showWarnings>true</showWarnings> 
                </configuration> 
            </plugin> 
        </plugins> 
    </build> 

    <!-- Added repository additionally as Finchley.M2 was not available in central repository --> 
    <repositories> 
        <repository> 
            <id>Spring Milestones</id> 
            <url>https://repo.spring.io/libs-milestone</url> 
            <snapshots> 
                <enabled>false</enabled> 
            </snapshots> 
        </repository> 
    </repositories> 

    <pluginRepositories> 
        <pluginRepository> 
            <id>Spring Milestones</id> 
            <url>https://repo.spring.io/libs-milestone</url> 
            <snapshots> 
                <enabled>false</enabled> 
            </snapshots> 
        </pluginRepository> 
    </pluginRepositories> 
</project> 
```

我们正在开发基于 REST 的微服务。我们将实现`restaurant`模块。`booking`和`user`模块是在类似的基础上开发的。

# 控制器类

`RestaurantController`类使用`@RestController`注解构建餐厅服务端点。我们在第二章中已经详细介绍了`@RestController`，*设置开发环境*。以下是

`@RestController`是一个类级注解，用于资源类。它是

`@Controller`和`@ResponseBody`注解的组合。它返回领域对象。

# API 版本控制

随着我们前进，我想与大家分享的是，我们在 REST 端点上使用了`v1`前缀。这代表了 API 的版本。我还想简要介绍一下 API 版本化的重要性。版本化 API 很重要，因为 API 会随着时间的推移而改变。您的知识和经验会随着时间而提高，这导致了 API 的变化。API 的变化可能会破坏现有的客户端集成。

因此，管理 API 版本有多种方法。其中一种是在路径中使用版本，或者有些人使用 HTTP 头。HTTP 头可以是一个自定义请求头或接受头，以表示调用 API 的版本。请参考 Bhakti Mehta 所著的《RESTful Java Patterns and Best Practices》，Packt Publishing 出版，[`www.packtpub.com/application-development/restful-java-patterns-and-best-practices`](https://www.packtpub.com/application-development/restful-java-patterns-and-best-practices)，以获取更多信息：

```java
@RestController 
@RequestMapping("/v1/restaurants") 
public class RestaurantController { 

    protected Logger logger = Logger.getLogger(RestaurantController.class.getName()); 

    protected RestaurantService restaurantService; 

    @Autowired 
    public RestaurantController(RestaurantService restaurantService) { 
        this.restaurantService = restaurantService; 
    } 

    /** 
     * Fetch restaurants with the specified name. A partial case-insensitive 
     * match is supported. So <code>http://.../restaurants/rest</code> will find 
     * any restaurants with upper or lower case 'rest' in their name. 
     * 
     * @param name 
     * @return A non-null, non-empty collection of restaurants. 
     */ 
    @RequestMapping(method = RequestMethod.GET) 
    public ResponseEntity<Collection<Restaurant>> findByName(@RequestParam("name") String name) { 

logger.info(String.format("restaurant-service findByName() invoked:{} for {} ", restaurantService.getClass().getName(), name)); 
        name = name.trim().toLowerCase(); 
        Collection<Restaurant> restaurants; 
        try { 
            restaurants = restaurantService.findByName(name); 
        } catch (Exception ex) { 
            logger.log(Level.WARNING, "Exception raised findByName REST Call", ex); 
            return new ResponseEntity< Collection< Restaurant>>(HttpStatus.INTERNAL_SERVER_ERROR); 
        } 
        return restaurants.size() > 0 ? new ResponseEntity< Collection< Restaurant>>(restaurants, HttpStatus.OK) 
                : new ResponseEntity< Collection< Restaurant>>(HttpStatus.NO_CONTENT); 
    } 

    /** 
     * Fetch restaurants with the given id. 
     * <code>http://.../v1/restaurants/{restaurant_id}</code> will return 
     * restaurant with given id. 
     * 
     * @param retaurant_id 
     * @return A non-null, non-empty collection of restaurants. 
     */ 
    @RequestMapping(value = "/{restaurant_id}", method = RequestMethod.GET) 
    public ResponseEntity<Entity> findById(@PathVariable("restaurant_id") String id) { 

       logger.info(String.format("restaurant-service findById() invoked:{} for {} ", restaurantService.getClass().getName(), id)); 
        id = id.trim(); 
        Entity restaurant; 
        try { 
            restaurant = restaurantService.findById(id); 
        } catch (Exception ex) { 
            logger.log(Level.SEVERE, "Exception raised findById REST Call", ex); 
            return new ResponseEntity<Entity>(HttpStatus.INTERNAL_SERVER_ERROR); 
        } 
        return restaurant != null ? new ResponseEntity<Entity>(restaurant, HttpStatus.OK) 
                : new ResponseEntity<Entity>(HttpStatus.NO_CONTENT); 
    } 

    /** 
     * Add restaurant with the specified information. 
     * 
     * @param Restaurant 
     * @return A non-null restaurant. 
     * @throws RestaurantNotFoundException If there are no matches at all. 
     */ 
    @RequestMapping(method = RequestMethod.POST) 
    public ResponseEntity<Restaurant> add(@RequestBody RestaurantVO restaurantVO) { 

        logger.info(String.format("restaurant-service add() invoked: %s for %s", restaurantService.getClass().getName(), restaurantVO.getName()); 

        Restaurant restaurant = new Restaurant(null, null, null); 
        BeanUtils.copyProperties(restaurantVO, restaurant); 
        try { 
            restaurantService.add(restaurant); 
        } catch (Exception ex) { 
            logger.log(Level.WARNING, "Exception raised add Restaurant REST Call "+ ex); 
            return new ResponseEntity<Restaurant>(HttpStatus.UNPROCESSABLE_ENTITY); 
        } 
        return new ResponseEntity<Restaurant>(HttpStatus.CREATED); 
    } 
} 
```

# 服务类

`RestaurantController`类使用了`RestaurantService`接口。`RestaurantService`是一个定义了 CRUD 和一些搜索操作的接口，具体定义如下：

```java
public interface RestaurantService { 

    public void add(Restaurant restaurant) throws Exception; 

    public void update(Restaurant restaurant) throws Exception; 

    public void delete(String id) throws Exception; 

    public Entity findById(String restaurantId) throws Exception; 

    public Collection<Restaurant> findByName(String name) throws Exception; 

    public Collection<Restaurant> findByCriteria(Map<String, ArrayList<String>> name) throws Exception; 
}
```

现在，我们可以实现我们刚刚定义的`RestaurantService`。它还扩展了你在上一章创建的`BaseService`类。我们使用`@Service` Spring 注解将其定义为服务：

```java
@Service("restaurantService") 
public class RestaurantServiceImpl extends BaseService<Restaurant, String> 
        implements RestaurantService { 

    private RestaurantRepository<Restaurant, String> restaurantRepository; 

    @Autowired 
    public RestaurantServiceImpl(RestaurantRepository<Restaurant, String> restaurantRepository) { 
        super(restaurantRepository); 
        this.restaurantRepository = restaurantRepository; 
    } 

    public void add(Restaurant restaurant) throws Exception { 
        if (restaurant.getName() == null || "".equals(restaurant.getName())) { 
            throw new Exception("Restaurant name cannot be null or empty string."); 
        } 

        if (restaurantRepository.containsName(restaurant.getName())) { 
            throw new Exception(String.format("There is already a product with the name - %s", restaurant.getName())); 
        } 

        super.add(restaurant); 
    } 

    @Override 
    public Collection<Restaurant> findByName(String name) throws Exception { 
        return restaurantRepository.findByName(name); 
    } 

    @Override 
    public void update(Restaurant restaurant) throws Exception { 
        restaurantRepository.update(restaurant); 
    } 

    @Override 
    public void delete(String id) throws Exception { 
        restaurantRepository.remove(id); 
    } 

    @Override 
    public Entity findById(String restaurantId) throws Exception { 
        return restaurantRepository.get(restaurantId); 
    } 

    @Override 
    public Collection<Restaurant> findByCriteria(Map<String, ArrayList<String>> name) throws Exception { 
        throw new UnsupportedOperationException("Not supported yet."); //To change body of generated methods, choose Tools | Templates. 
    } 
} 
```

# 仓库类

`RestaurantRepository`接口定义了两个新方法：`containsName`和`findByName`方法。它还扩展了`Repository`接口：

```java
public interface RestaurantRepository<Restaurant, String> extends Repository<Restaurant, String> { 

    boolean containsName(String name) throws Exception; 

    Collection<Restaurant> findByName(String name) throws Exception; 
} 
```

`Repository`接口定义了`add`、`remove`和`update`三个方法。它还扩展了`ReadOnlyRepository`接口：

```java
public interface Repository<TE, T> extends ReadOnlyRepository<TE, T> { 

    void add(TE entity); 

    void remove(T id); 

    void update(TE entity); 
} 
```

`ReadOnlyRepository`接口定义了`get`和`getAll`方法，分别返回布尔值、实体和实体集合。如果你想要只暴露仓库的只读抽象，这个接口很有用：

```java
public interface ReadOnlyRepository<TE, T> { 

    boolean contains(T id); 

    Entity get(T id); 

    Collection<TE> getAll(); 
} 
```

Spring 框架使用`@Repository`注解来定义实现仓库的仓库 bean。在`RestaurantRepository`的情况下，可以看到使用了映射来代替实际的数据库实现。这使得所有实体都只保存在内存中。因此，当我们启动服务时，只在内存中找到两家餐厅。我们可以使用 JPA 进行数据库持久化。这是生产就绪实现的一般做法：

```java
@Repository("restaurantRepository") 
public class InMemRestaurantRepository implements RestaurantRepository<Restaurant, String> { 
    private Map<String, Restaurant> entities; 

    public InMemRestaurantRepository() { 
        entities = new HashMap(); 
        Restaurant restaurant = new Restaurant("Big-O Restaurant", "1", null); 
        entities.put("1", restaurant); 
        restaurant = new Restaurant("O Restaurant", "2", null); 
        entities.put("2", restaurant); 
    } 

    @Override 
    public boolean containsName(String name) { 
        try { 
            return this.findByName(name).size() > 0; 
        } catch (Exception ex) { 
            //Exception Handler 
        } 
        return false; 
    } 

    @Override 
    public void add(Restaurant entity) { 
        entities.put(entity.getId(), entity); 
    } 

    @Override 
    public void remove(String id) { 
        if (entities.containsKey(id)) { 
            entities.remove(id); 
        } 
    } 

    @Override 
    public void update(Restaurant entity) { 
        if (entities.containsKey(entity.getId())) { 
            entities.put(entity.getId(), entity); 
        } 
    } 

    @Override 
    public Collection<Restaurant> findByName(String name) throws Exception { 
        Collection<Restaurant> restaurants = new ArrayList<>(); 
        int noOfChars = name.length(); 
        entities.forEach((k, v) -> { 
            if (v.getName().toLowerCase().contains(name.subSequence(0, noOfChars))) { 
                restaurants.add(v); 
            } 
        }); 
        return restaurants; 
    } 

    @Override 
    public boolean contains(String id) { 
        throw new UnsupportedOperationException("Not supported yet.");  
    } 

    @Override 
    public Entity get(String id) { 
        return entities.get(id); 
    } 

    @Override 
    public Collection<Restaurant> getAll() { 
        return entities.values(); 
    } 
} 
```

# 实体类

以下是如何定义扩展了`BaseEntity`的`Restaurant`实体的：

```java
public class Restaurant extends BaseEntity<String> { 

    private List<Table> tables = new ArrayList<>(); 

    public Restaurant(String name, String id, List<Table> tables) { 
        super(id, name); 
        this.tables = tables; 
    } 

    public void setTables(List<Table> tables) { 
        this.tables = tables; 
    } 

    public List<Table> getTables() { 
        return tables; 
    } 

    @Override 
    public String toString() { 
        return String.format("{id: %s, name: %s, address: %s, tables: %s}", this.getId(), 
                         this.getName(), this.getAddress(), this.getTables()); 
    } 

} 
```

由于我们使用 POJO 类来定义实体，在许多情况下我们不需要创建一个 VO。这个想法是对象的状态不应该被持久化。

以下是如何定义扩展了`BaseEntity`的`Table`实体：

```java
public class Table extends BaseEntity<BigInteger> { 

    private int capacity; 

    public Table(String name, BigInteger id, int capacity) { 
        super(id, name); 
        this.capacity = capacity; 
    } 

    public void setCapacity(int capacity) { 
        this.capacity = capacity; 
    } 

    public int getCapacity() { 
        return capacity; 
    } 

    @Override 
    public String toString() { 
        return String.format("{id: %s, name: %s, capacity: %s}", 
                         this.getId(), this.getName(), this.getCapacity());    } 

} 
```

以下是如何定义`Entity`抽象类的：

```java
public abstract class Entity<T> { 

    T id; 
    String name; 

    public T getId() { 
        return id; 
    } 

    public void setId(T id) { 
        this.id = id; 
    } 

    public String getName() { 
        return name; 
    } 

    public void setName(String name) { 
        this.name = name; 
    } 

} 
```

以下是如何定义`BaseEntity`抽象类的。它扩展了`Entity`

抽象类：

```java
public abstract class BaseEntity<T> extends Entity<T> { 

    private T id; 
    private boolean isModified; 
    private String name; 

    public BaseEntity(T id, String name) { 
        this.id = id; 
        this.name = name; 
    } 

    public T getId() { 
        return id; 
    } 

    public void setId(T id) { 
        this.id = id; 
    } 

    public boolean isIsModified() { 
        return isModified; 
    } 

    public void setIsModified(boolean isModified) { 
        this.isModified = isModified; 
    } 

    public String getName() { 
        return name; 
    } 

    public void setName(String name) { 
        this.name = name; 
    } 

} 
```

我们已经完成了 Restaurant 服务的实现。现在，我们将开发 Eureka 模块（服务）。

# 注册和发现服务（Eureka 服务）

我们需要一个所有微服务都可以注册和引用的地方——一个服务发现和注册应用程序。Spring Cloud 提供了最先进的服务注册和发现应用程序 Netflix Eureka。我们将利用它为我们的示例项目 OTRS 服务。

一旦您按照本节中的描述配置了 Eureka 服务，它将可供所有传入请求使用以在 Eureka 服务上列出它。Eureka 服务注册/列出通过 Eureka 客户端配置的所有微服务。一旦您启动您的服务，它就会通过`application.yml`中配置的 Eureka 服务发送 ping，一旦建立连接，Eureka 服务将注册该服务。

它还通过统一的连接方式启用微服务的发现。您不需要任何 IP、主机名或端口来查找服务，您只需要提供服务 ID 即可。服务 ID 在各个微服务的`application.yml`中配置。

在以下三个步骤中，我们可以创建一个 Eureka 服务（服务注册和发现服务）：

1.  **Maven 依赖**：它需要一个 Spring Cloud 依赖，如图所示，并在`pom.xml`中的启动类中使用`@EnableEurekaApplication`注解：

```java
<dependency> 
      <groupId>org.springframework.cloud</groupId> 
      <artifactId>spring-cloud-starter-config</artifactId> 
</dependency> 
<dependency> 
      <groupId>org.springframework.cloud</groupId> 
      <artifactId>spring-cloud-netflix-eureka-server</artifactId> 
</dependency> 
```

1.  **启动类**：启动类`App`通过仅使用`@EnableEurekaApplication`类注解来无缝运行 Eureka 服务：

```java
package com.packtpub.mmj.eureka.service; 

import org.springframework.boot.SpringApplication; 
import org.springframework.boot.autoconfigure.SpringBootApplication; 
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer; 

@SpringBootApplication 
@EnableEurekaServer 
public class App { 

    public static void main(String[] args) { 
        SpringApplication.run(App.class, args); 
    } 
} 
```

在`pom.xml`项目的`<properties>`标签下使用`<start-class>com.packtpub.mmj.eureka.service.App</start-class>`。

1.  **Spring 配置**：Eureka 服务也需要以下 Spring 配置来实现 Eureka 服务器的配置（`src/main/resources/application.yml`）：

```java
server: 
  port: 8761  # HTTP port 

eureka: 
  instance: 
    hostname: localhost 
  client: 
    registerWithEureka: false 
    fetchRegistry: false 
    serviceUrl: 
        defaultZone: ${vcap.services.${PREFIX:}eureka.credentials.uri:http://user:password@localhost:8761}/eureka/ 
  server: 
    waitTimeInMsWhenSyncEmpty: 0 
    enableSelfPreservation: false 

```

# **Eureka 客户端**

与 Eureka 服务器类似，每个 OTRS 服务也应该包含 Eureka 客户端配置，以便可以建立 Eureka 服务器和客户端之间的连接。没有这个，服务的注册和发现是不可能的。

您的服务可以使用以下 Spring 配置来配置 Eureka 客户端。在`restaurant-service\src\main\resources\application.yml`中添加以下配置：

```java
eureka: 
  client: 
    serviceUrl: 
      defaultZone: http://localhost:8761/eureka/ 
```

# 预订和用户服务

我们可以使用`RestaurantService`实现来开发预订和用户服务。用户服务可以提供与用户资源相关的 CRUD 操作端点。预订服务可以提供与预订资源相关的 CRUD 操作端点和桌位可用性。您可以在 Packt 网站或 Packt Publishing GitHub 仓库上找到这些服务的示例代码。

# 执行

要了解我们的代码是如何工作的，我们首先需要构建它，然后执行它。我们将使用 Maven 清理包来构建服务 JAR。

现在，要执行这些服务 JAR，只需从项目根目录执行以下命令即可：

```java
java -jar <service>/target/<service_jar_file> 
```

以下是一些示例：

```java
java -jar restaurant-service/target/restaurant-service.jar 
java -jar eureka-service/target/eureka-service.jar 
```

我们将按以下顺序从项目根目录执行我们的服务。首先应启动 Eureka 服务；最后三个微服务的顺序可以改变：

```java
java -jar eureka-service/target/eureka-service.jar
java -jar restaurant-service/target/restaurant-service.jar java -jar booking-service/target/booking-service.jar java -jar user-service/target/user-service.jar
```

# 测试

为了启用测试，在`pom.xml`文件中添加以下依赖项：

```java
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-test</artifactId> 
</dependency> 
```

为了测试`RestaurantController`，已添加以下文件：

+   `RestaurantControllerIntegrationTests`类，它使用了

    `@SpringApplicationConfiguration`注解以选择 Spring Boot 使用的相同配置：

```java
@RunWith(SpringJUnit4ClassRunner.class) 
@SpringApplicationConfiguration(classes = RestaurantApp.class) 
public class RestaurantControllerIntegrationTests extends 
        AbstractRestaurantControllerTests { 

}
```

+   一个`abstract`类来编写我们的测试：

```java
public abstract class AbstractRestaurantControllerTests { 

    protected static final String RESTAURANT = "1"; 
    protected static final String RESTAURANT_NAME = "Big-O Restaurant"; 

    @Autowired 
    RestaurantController restaurantController; 

    @Test 
    public void validResturantById() { 
        Logger.getGlobal().info("Start validResturantById test"); 
        ResponseEntity<Entity> restaurant = restaurantController.findById(RESTAURANT); 

        Assert.assertEquals(HttpStatus.OK, restaurant.getStatusCode()); 
        Assert.assertTrue(restaurant.hasBody()); 
        Assert.assertNotNull(restaurant.getBody()); 
        Assert.assertEquals(RESTAURANT, restaurant.getBody().getId()); 
        Assert.assertEquals(RESTAURANT_NAME, restaurant.getBody().getName()); 
        Logger.getGlobal().info("End validResturantById test"); 
    } 

    @Test 
    public void validResturantByName() { 
        Logger.getGlobal().info("Start validResturantByName test"); 
        ResponseEntity<Collection<Restaurant>> restaurants = restaurantController.findByName(RESTAURANT_NAME); 
        Logger.getGlobal().info("In validAccount test"); 

        Assert.assertEquals(HttpStatus.OK, restaurants.getStatusCode()); 
        Assert.assertTrue(restaurants.hasBody()); 
        Assert.assertNotNull(restaurants.getBody()); 
        Assert.assertFalse(restaurants.getBody().isEmpty()); 
        Restaurant restaurant = (Restaurant) restaurants.getBody().toArray()[0]; 
        Assert.assertEquals(RESTAURANT, restaurant.getId()); 
        Assert.assertEquals(RESTAURANT_NAME, restaurant.getName()); 
        Logger.getGlobal().info("End validResturantByName test"); 
    } 

    @Test 
    public void validAdd() { 
        Logger.getGlobal().info("Start validAdd test"); 
        RestaurantVO restaurant = new RestaurantVO(); 
        restaurant.setId("999"); 
        restaurant.setName("Test Restaurant"); 

        ResponseEntity<Restaurant> restaurants = restaurantController.add(restaurant); 
        Assert.assertEquals(HttpStatus.CREATED, restaurants.getStatusCode()); 
        Logger.getGlobal().info("End validAdd test"); 
    } 
} 
```

+   最后是`RestaurantControllerTests`类，它扩展了之前创建的`abstract`类，还创建了`RestaurantService`和`RestaurantRepository`实现：

```java
public class RestaurantControllerTests extends AbstractRestaurantControllerTests { 

    protected static final Restaurant restaurantStaticInstance = new Restaurant(RESTAURANT, 
            RESTAURANT_NAME, null); 

    protected static class TestRestaurantRepository implements RestaurantRepository<Restaurant, String> { 

        private Map<String, Restaurant> entities; 

        public TestRestaurantRepository() { 
            entities = new HashMap(); 
            Restaurant restaurant = new Restaurant("Big-O Restaurant", "1", null); 
            entities.put("1", restaurant); 
            restaurant = new Restaurant("O Restaurant", "2", null); 
            entities.put("2", restaurant); 
        } 

        @Override 
        public boolean containsName(String name) { 
            try { 
                return this.findByName(name).size() > 0; 
            } catch (Exception ex) { 
                //Exception Handler 
            } 
            return false; 
        } 

        @Override 
        public void add(Restaurant entity) { 
            entities.put(entity.getId(), entity); 
        } 

        @Override 
        public void remove(String id) { 
            if (entities.containsKey(id)) { 
                entities.remove(id); 
            } 
        } 

        @Override 
        public void update(Restaurant entity) { 
            if (entities.containsKey(entity.getId())) { 
                entities.put(entity.getId(), entity); 
            } 
        } 

        @Override 
        public Collection<Restaurant> findByName(String name) throws Exception { 
            Collection<Restaurant> restaurants = new ArrayList(); 
            int noOfChars = name.length(); 
            entities.forEach((k, v) -> { 
                if (v.getName().toLowerCase().contains(name.subSequence(0, noOfChars))) { 
                    restaurants.add(v); 
                } 
            }); 
            return restaurants; 
        } 

        @Override 
        public boolean contains(String id) { 
            throw new UnsupportedOperationException("Not supported yet.");
        } 

        @Override 
        public Entity get(String id) { 
            return entities.get(id); 
        } 
        @Override 
        public Collection<Restaurant> getAll() { 
            return entities.values(); 
        } 
    } 

    protected TestRestaurantRepository testRestaurantRepository = new TestRestaurantRepository(); 
    protected RestaurantService restaurantService = new RestaurantServiceImpl(testRestaurantRepository); 

    @Before 
    public void setup() { 
        restaurantController = new RestaurantController(restaurantService); 

    } 
} 
```

# 参考文献

+   《RESTful Java Patterns and Best Practices》by Bhakti Mehta, Packt Publishing: [`www.packtpub.com/application-development/restful-java-patterns-and-best-practices`](https://www.packtpub.com/application-development/restful-java-patterns-and-best-practices)

+   Spring Cloud: [`cloud.spring.io/`](http://cloud.spring.io/)

+   Netflix Eureka: [`github.com/netflix/eureka`](https://github.com/netflix/eureka)

# 总结

在本章中，我们学习了领域驱动设计模型如何在微服务中使用。运行演示应用程序后，我们可以看到每个微服务如何可以独立地开发、部署和测试。你可以使用 Spring Cloud 非常容易地创建微服务。我们还探讨了如何使用 Spring Cloud 与 Eureka 注册和发现组件。

在下一章中，我们将学习如何将微服务部署在容器中，例如 Docker。我们还将了解使用 REST Java 客户端和其他工具进行微服务测试。
