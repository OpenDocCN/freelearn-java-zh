# 第九章：使用 Spring Boot 创建 RESTful Web 服务

在本章中，我们将涵盖以下示例：

+   创建一个简单的 Spring Boot 应用程序

+   与数据库交互

+   创建一个 RESTful web 服务

+   为 Spring Boot 创建多个配置文件

+   将 RESTful web 服务部署到 Heroku

+   使用 Docker 将 RESTful web 服务容器化

+   使用 Micrometer 和 Prometheus 监控 Spring Boot 2 应用程序

# 介绍

近年来，基于微服务架构的推动已经得到了广泛的采用，这要归功于它在正确的方式下提供的简单性和易于维护性。许多公司，如 Netflix 和 Amazon，已经从单片系统转移到了更专注和轻量级的系统，它们之间通过 RESTful web 服务进行通信。RESTful web 服务的出现及其使用已知的 HTTP 协议创建 web 服务的简单方法，使得应用程序之间的通信比旧的基于 SOAP 的 web 服务更容易。

在本章中，我们将介绍**Spring Boot**框架，它提供了一种方便的方式来使用 Spring 库创建可投入生产的微服务。使用 Spring Boot，我们将开发一个简单的 RESTful web 服务并将其部署到云端。

# 创建一个简单的 Spring Boot 应用程序

Spring Boot 有助于轻松创建可投入生产的基于 Spring 的应用程序。它支持几乎所有 Spring 库的工作，而无需显式配置它们。提供了自动配置类，以便轻松集成大多数常用的库、数据库和消息队列。

在本示例中，我们将介绍如何创建一个简单的 Spring Boot 应用程序，其中包含一个在浏览器中打开时打印消息的控制器。

# 准备工作

Spring Boot 支持 Maven 和 Gradle 作为其构建工具，我们将在我们的示例中使用 Maven。以下 URL，[`start.spring.io/`](http://start.spring.io/)，提供了一种方便的方式来创建一个带有所需依赖项的空项目。我们将使用它来下载一个空项目。按照以下步骤创建并下载一个基于 Spring Boot 的空项目：

1.  导航到[`start.spring.io/`](http://start.spring.io/)，您将看到类似以下截图的内容：

![](img/b5e3a855-cad8-4f41-9ebb-850abb91ce31.png)

1.  您可以选择依赖管理和构建工具，通过在**Generate a**文本后的下拉菜单中选择适当的选项。

1.  Spring Boot 支持 Java、Kotlin 和 Groovy。您可以通过更改**with**文本后的下拉菜单来选择语言。

1.  通过在**and Spring Boot**文本后的下拉菜单中选择其值来选择 Spring Boot 版本。对于本示例，我们将使用 Spring Boot 2 的最新稳定版本，即 2.0.4。

1.  在左侧的项目元数据下，我们需要提供与 Maven 相关的信息，即组 ID 和 artifact ID。我们将使用 Group 作为`com.packt`，Artifact 作为`boot_demo`。

1.  在右侧的依赖项下，您可以搜索要添加的依赖项。对于本示例，我们需要 web 和 Thymeleaf 依赖项。这意味着我们想要创建一个使用 Thymeleaf UI 模板的 web 应用程序，并且希望所有依赖项，如 Spring MVC 和嵌入式 Tomcat，都成为应用程序的一部分。

1.  单击生成项目按钮以下载空项目。您可以将此空项目加载到您选择的任何 IDE 中，就像加载任何其他 Maven 项目一样。

此时，您将在您选择的任何 IDE 中加载您的空项目，并准备进一步探索。在本示例中，我们将使用 Thymeleaf 模板引擎来定义我们的网页，并创建一个简单的控制器来呈现网页。

此处的完整代码可以在`Chapter09/1_boot_demo`中找到。

# 如何做...

1.  如果您按照“准备就绪”部分提到的组 ID 和工件 ID 命名约定进行了跟随，您将拥有一个包结构`com.packt.boot_demo`，以及一个`BootDemoApplication.java`主类已经为您创建。在`tests`文件夹下将有一个等效的包结构和一个`BootDemoApplicationTests.java`主类。

1.  在`com.packt.boot_demo`包下创建一个名为`SimpleViewController`的新类，其中包含以下代码：

```java
        @Controller
        public class SimpleViewController{
          @GetMapping("/message")
          public String message(){
            return "message";
          }  
        }
```

1.  在`src/main/resources/templates`下创建一个名为`message.html`的网页，其中包含以下代码：

```java
        <h1>Hello, this is a message from the Controller</h1>
        <h2>The time now is [[${#dates.createNow()}]]</h2>
```

1.  从命令提示符中，导航到项目根文件夹，并发出`mvn spring-boot:run`命令；您将看到应用程序正在启动。一旦完成初始化并启动，它将在默认端口`8080`上运行。导航到`http://localhost:8080/message`以查看消息。

我们使用 Spring Boot 的 Maven 插件，它为我们提供了方便的工具来在开发过程中启动应用程序。但是对于生产环境，我们将创建一个 fat JAR，即一个包含所有依赖项的 JAR，并将其部署为 Linux 或 Windows 服务。我们甚至可以使用`java -jar`命令运行这个 fat JAR。

# 工作原理

我们不会深入讨论 Spring Boot 或其他 Spring 库的工作原理。Spring Boot 创建了一个嵌入式 Tomcat，运行在默认端口`8080`上。然后，它注册了所有被`@SpringBootApplication`注解的类所在包及其子包中可用的控制器、组件和服务。

在我们的示例中，`com.packt.boot_demo`包中的`BootDemoApplication`类被注解为`@SpringBootApplication`。因此，所有被注解为`@Controller`、`@Service`、`@Configuration`和`@Component`的类都会被 Spring 框架注册为 bean，并由其管理。现在，这些可以通过使用`@Autowired`注解注入到代码中。

我们可以通过两种方式创建一个 web 控制器：

+   使用`@Controller`进行注解

+   使用`@RestController`进行注解

在第一种方法中，我们创建了一个既可以提供原始数据又可以提供 HTML 数据（由模板引擎如 Thymeleaf、Freemarker 和 JSP 生成）的控制器。在第二种方法中，控制器支持只能提供 JSON 或 XML 形式的原始数据的端点。在我们的示例中，我们使用了前一种方法，如下所示：

```java
@Controller
public class SimpleViewController{
  @GetMapping("/message")
  public String message(){
    return "message";
  }
}
```

我们可以用`@RequestMapping`注解类，比如`@RequestMapping("/api")`。在这种情况下，控制器中暴露的任何 HTTP 端点都会以`/api`开头。对于 HTTP 的`GET`、`POST`、`DELETE`和`PUT`方法，有专门的注解映射，分别是`@GetMapping`、`@PostMapping`、`@DeleteMapping`和`@PutMapping`。我们也可以将我们的控制器类重写如下：

```java
@Controller
@RequestMapping("/message")
public class SimpleViewController{
  @GetMapping
  public String message(){
    return "message";
  }
}
```

我们可以通过在`application.properties`文件中提供`server.port = 9090`来修改端口。这个文件可以在`src/main/resources/application.properties`中找到。有一整套属性（[`docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html`](http://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)）可以用来自定义和连接不同的组件。

# 与数据库交互

在这个示例中，我们将看看如何与数据库集成，以创建、读取、修改和删除数据。为此，我们将设置一个带有所需表的 MySQL 数据库。随后，我们将从我们的 Spring Boot 应用程序中更新表中的数据。

我们将使用 Windows 作为这个示例的开发平台。您也可以在 Linux 上执行类似的操作，但首先必须设置您的 MySQL 数据库。

# 准备就绪

在我们开始将应用程序与数据库集成之前，我们需要在开发机器上本地设置数据库。在接下来的几节中，我们将下载和安装 MySQL 工具，然后创建一个带有一些数据的示例表，我们将在应用程序中使用。

# 安装 MySQL 工具

首先，从[`dev.mysql.com/downloads/windows/installer/5.7.html`](https://dev.mysql.com/downloads/windows/installer/5.7.html)下载 MySQL 安装程序。这个 MySQL 捆绑包只适用于 Windows。按照屏幕上的说明成功安装 MySQL 以及其他工具，如 MySQL Workbench。要确认 MySQL 守护程序（`mysqld`）正在运行，打开任务管理器，你应该能够看到一个类似以下的进程：

![](img/8d1a7d2a-8bae-412a-8268-7448d9872d33.png)

你应该记住你为 root 用户设置的密码。

让我们运行 MySQL Workbench；启动时，你应该能够看到类似以下截图的东西，以及工具提供的其他内容：

![](img/07d591cd-63f0-4f83-afff-9e73c07bf04e.png)

如果你在前面的图像中找不到连接，你可以使用(+)号添加一个。当你点击(+)时，你将看到以下对话框。填写它并点击测试连接以获得成功消息：

![](img/558a654f-24cf-42bc-9e5c-063e240daea3.png)

成功的测试连接将导致以下消息：

![](img/92db869d-b0c8-40a0-ad8d-a3b63206221c.png)

双击连接到数据库，你应该能够在左侧看到一个 DB 列表，在右侧看到一个空白区域，在顶部看到一个菜单和工具栏。从文件菜单中，点击新查询选项卡，或按*Ctrl* + *T*获得一个新的查询窗口。在这里，我们将编写我们的查询来创建一个数据库，并在该数据库中创建一个表。

从[`dev.mysql.com/downloads/windows/installer/5.7.html`](https://dev.mysql.com/downloads/windows/installer/5.7.html)下载的捆绑安装程序仅适用于 Windows。Linux 用户必须单独下载 MySQL 服务器和 MySQL Workbench（与 DB 交互的 GUI）。

MySQL 服务器可以从[`dev.mysql.com/downloads/mysql/`](https://dev.mysql.com/downloads/mysql/)下载。

MySQL Workbench 可以从[`dev.mysql.com/downloads/workbench/`](https://dev.mysql.com/downloads/workbench/)下载。

# 创建一个示例数据库

运行以下 SQL 语句创建数据库：

```java
create database sample;
```

# 创建一个人员表

运行以下 SQL 语句使用新创建的数据库并创建一个简单的人员表：

```java
create table person( 
  id int not null auto_increment,    
  first_name varchar(255),    
  last_name varchar(255),
  place varchar(255),    
  primary key(id)
);
```

# 填充示例数据

让我们继续在我们刚刚创建的表中插入一些示例数据：

```java
insert into person(first_name, last_name, place) 
values('Raj', 'Singh', 'Bangalore');

insert into person(first_name, last_name, place) 
values('David', 'John', 'Delhi');
```

现在我们的数据库准备好了，我们将继续从[`start.spring.io/`](http://start.spring.io/)下载空的 Spring Boot 项目，选项如下：

![](img/6f336164-f3e0-4bb5-8fe4-439252c7baf0.png)

# 如何做...

1.  创建一个模型类`com.packt.boot_db_demo.Person`，代表一个人。我们将使用 Lombok 注解为我们生成 getter 和 setter：

```java
        @Data
        public class Person{
          private Integer id;
          private String firstName;
          private String lastName;
          private String place;
        }
```

1.  创建`com.packt.boot_db_demo.PersonMapper`将数据库中的数据映射到我们的模型类`Person`：

```java
        @Mapper
        public interface PersonMapper {
        }
```

1.  让我们添加一个方法来获取表中的所有行。请注意，接下来的几个方法将写在`PersonMapper`接口内：

```java
        @Select("SELECT * FROM person")
        public List<Person> getPersons();
```

1.  通过 ID 标识的单个人的详细信息的另一种方法如下：

```java
        @Select("SELECT * FROM person WHERE id = #{id}")
        public Person getPerson(Integer id);
```

1.  在表中创建新行的方法如下：

```java
        @Insert("INSERT INTO person(first_name, last_name, place) " 
                 + " VALUES (#{firstName}, #{lastName}, #{place})")
        @Options(useGeneratedKeys = true)
        public void insert(Person person);
```

1.  更新表中现有行的方法，通过 ID 标识：

```java
  @Update("UPDATE person SET first_name = #{firstName},last_name = 
             #{lastName}, "+ "place = #{place}  WHERE id = #{id} ")
  public void save(Person person);
```

1.  从表中删除行的方法，通过 ID 标识：

```java
        @Delete("DELETE FROM person WHERE id = #{id}")
        public void delete(Integer id);
```

1.  让我们创建一个`com.packt.boot_db_demo.PersonController`类，我们将用它来编写我们的 web 端点：

```java
        @Controller
        @RequestMapping("/persons")
        public class PersonContoller {
          @Autowired PersonMapper personMapper;
        }
```

1.  让我们创建一个端点来列出`person`表中的所有条目：

```java
        @GetMapping
        public String list(ModelMap model){
          List<Person> persons = personMapper.getPersons();
          model.put("persons", persons);
          return "list";
        }
```

1.  让我们创建一个端点来在`person`表中添加一个新行：

```java
   @GetMapping("/{id}")
   public String detail(ModelMap model, @PathVariable Integer id){
        System.out.println("Detail id: " + id);
        Person person = personMapper.getPerson(id);
        model.put("person", person);
        return "detail";
   }
```

1.  让我们创建一个端点来在`person`表中添加一个新行或编辑一个现有行：

```java
        @PostMapping("/form")
        public String submitForm(Person person){
          System.out.println("Submiting form person id: " + 
                             person.getId());
          if ( person.getId() != null ){
            personMapper.save(person);
          }else{
            personMapper.insert(person);
          }
          return "redirect:/persons/";
        }  
```

1.  让我们创建一个端点来从`person`表中删除一行：

```java
        @GetMapping("/{id}/delete")
        public String deletePerson(@PathVariable Integer id){
          personMapper.delete(id);
          return "redirect:/persons";
        }
```

1.  更新`src/main/resources/application.properties`文件，提供与我们的数据源（即 MySQL 数据库）相关的配置：

```java
  spring.datasource.driver-class-name=com.mysql.jdbc.Driver
  spring.datasource.url=jdbc:mysql://localhost/sample?useSSL=false
  spring.datasource.username=root
  spring.datasource.password=mohamed
  mybatis.configuration.map-underscore-to-camel-case=true
```

您可以使用`mvn spring-boot:run`命令行运行应用程序。该应用程序在默认端口`8080`上启动。在浏览器中导航到`http://localhost:8080/persons`。

这个食谱的完整代码可以在`Chapter09/2_boot_db_demo`找到。

访问`http://localhost:8080/persons`，您会看到以下内容：

![](img/db4bfc97-92cf-41ab-80a7-00fd37217a9f.png)

点击**新建人员**，您会得到以下内容：

![](img/4f5cf0ca-3c12-435a-9fb2-64c2891729fd.png)

点击**编辑**，你会得到以下内容：

![](img/afe20954-24b9-43f5-b76b-1f4b171e1666.png)

# 工作原理...

首先，`com.packt.boot_db_demo.PersonMapper`使用`org.apache.ibatis.annotations.Mapper`注解知道如何执行`@Select`、`@Update`和`@Delete`注解中提供的查询，并返回相关结果。这一切都由 MyBatis 和 Spring Data 库管理。

你一定想知道如何实现与数据库的连接。Spring Boot 的一个自动配置类`DataSourceAutoConfiguration`通过使用`application.properties`文件中定义的`spring.datasource.*`属性来设置连接，从而为我们提供了`javax.sql.DataSource`的实例。然后 MyBatis 库使用这个`javax.sql.DataSource`对象为我们提供了`SqlSessionTemplate`的实例，这就是我们的`PersonMapper`在后台使用的。

然后，我们通过`@AutoWired`将`com.packt.boot_db_demo.PersonMapper`注入到`com.packt.boot_db_demo.PersonController`类中。`@AutoWired`注解寻找任何 Spring 管理的 bean，这些 bean 要么是确切类型的实例，要么是其实现。查看本章中的*创建一个简单的 Spring Boot 应用程序*食谱，了解`@Controller`注解。

凭借极少的配置，我们已经能够快速设置简单的 CRUD 操作。这就是 Spring Boot 为开发人员提供的灵活性和敏捷性！

# 创建 RESTful Web 服务

在上一个食谱中，我们使用 Web 表单与数据进行交互。在这个食谱中，我们将看到如何使用 RESTful Web 服务与数据进行交互。这些 Web 服务是使用已知的 HTTP 协议及其方法（即 GET、POST 和 PUT）与其他应用程序进行交互的一种方式。数据可以以 XML、JSON 甚至纯文本的形式交换。我们将在我们的食谱中使用 JSON。

因此，我们将创建 RESTful API 来支持检索数据、创建新数据、编辑数据和删除数据。

# 准备工作

像往常一样，通过选择以下截图中显示的依赖项从[`start.spring.io/`](http://start.spring.io/)下载起始项目：

![](img/90cb2683-ac4e-49f1-bd9d-78582636818c.png)

# 如何做...

1.  从上一个食谱中复制`Person`类：

```java
        public class Person {
          private Integer id;
          private String firstName;
          private String lastName;
          private String place;
          //required getters and setters
        }
```

1.  我们将以不同的方式完成`PersonMapper`部分。我们将在一个 mapper XML 文件中编写所有的 SQL 查询，然后从`PersonMapper`接口中引用它们。我们将把 mapper XML 放在`src/main/resources/mappers`文件夹下。我们将`mybatis.mapper-locations`属性的值设置为`classpath*:mappers/*.xml`。这样，`PersonMapper`接口就可以发现与其方法对应的 SQL 查询。

1.  创建`com.packt.boot_rest_demo.PersonMapper`接口：

```java
        @Mapper
        public interface PersonMapper {
          public List<Person> getPersons();
          public Person getPerson(Integer id);
          public void save(Person person);
          public void insert(Person person);
          public void delete(Integer id);
        }
```

1.  在`PersonMapper.xml`中创建 SQL。确保`<mapper>`标签的`namespace`属性与`PersonMapper`映射接口的完全限定名称相同：

```java
        <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
        <mapper namespace="com.packt.boot_rest_demo.PersonMapper">
          <select id="getPersons"
           resultType="com.packt.boot_rest_demo.Person">
            SELECT id, first_name firstname, last_name lastname, place
            FROM person
          </select>

          <select id="getPerson"
           resultType="com.packt.boot_rest_demo.Person"
           parameterType="long">
            SELECT id, first_name firstname, last_name lastname, place
            FROM person
            WHERE id = #{id}
          </select>

          <update id="save"
           parameterType="com.packt.boot_rest_demo.Person">
            UPDATE person SET
              first_name = #{firstName},
              last_name = #{lastName},
              place = #{place}
            WHERE id = #{id}
          </update>

          <insert id="insert" 
           parameterType="com.packt.boot_rest_demo.Person"
           useGeneratedKeys="true" keyColumn="id" keyProperty="id">
            INSERT INTO person(first_name, last_name, place)
            VALUES (#{firstName}, #{lastName}, #{place})
          </insert>

          <delete id="delete" parameterType="long">
            DELETE FROM person WHERE id = #{id}
          </delete>
        </mapper>
```

1.  在`src/main/resources/application.properties`文件中定义应用程序属性：

```java
        spring.datasource.driver-class-name=com.mysql.jdbc.Driver
        spring.datasource.url=jdbc:mysql://localhost/sample?    
        useSSL=false
        spring.datasource.username=root
        spring.datasource.password=mohamed
        mybatis.mapper-locations=classpath*:mappers/*.xml
```

1.  为 REST API 创建一个空的控制器。这个控制器将被标记为`@RestController`注解，因为其中的所有 API 都将专门处理数据：

```java
        @RestController
        @RequestMapping("/api/persons")
        public class PersonApiController {
          @Autowired PersonMapper personMapper;
        }
```

1.  添加一个 API 来列出`person`表中的所有行：

```java
        @GetMapping
        public ResponseEntity<List<Person>> getPersons(){
          return new ResponseEntity<>(personMapper.getPersons(),
                                      HttpStatus.OK);
        }
```

1.  添加一个 API 来获取单个人的详细信息：

```java
   @GetMapping("/{id}")
   public ResponseEntity<Person> getPerson(@PathVariable Integer id){
      return new ResponseEntity<>(personMapper.getPerson(id),
                                                     HttpStatus.OK);
   }
```

1.  添加一个 API 来向表中添加新数据：

```java
        @PostMapping
        public ResponseEntity<Person> newPerson
                       (@RequestBody Person person){
          personMapper.insert(person);
          return new ResponseEntity<>(person, HttpStatus.OK);
        }
```

1.  添加一个 API 来编辑表中的数据：

```java
        @PostMapping("/{id}")
        public ResponseEntity<Person> updatePerson
                       (@RequestBody Person person,
          @PathVariable Integer id){
            person.setId(id);
            personMapper.save(person);
            return new ResponseEntity<>(person, HttpStatus.OK);
          }
```

1.  添加一个 API 来删除表中的数据：

```java
        @DeleteMapping("/{id}")
        public ResponseEntity<Void> deletePerson
                       (@PathVariable Integer id){
          personMapper.delete(id);
          return new ResponseEntity<>(HttpStatus.OK);
        }
```

您可以在`Chapter09/3_boot_rest_demo`找到完整的代码。您可以通过在项目文件夹中使用`mvn spring-boot:run`来启动应用程序。应用程序启动后，导航到`http://localhost:8080/api/persons`以查看 person 表中的所有数据。

为了测试其他 API，我们将使用 Google Chrome 的 Postman REST 客户端应用程序。

这就是添加新人的样子。看一下请求体，也就是在 JSON 中指定的人的详细信息：

![](img/f79bf64e-58f7-4c15-b37e-2e8e8acfc231.png)

这就是我们编辑一个人的详细信息的方式：

![](img/b66cfed2-0bea-475f-a514-ddbd7126ec02.png)

这就是删除一个人的样子：

![](img/0cd04988-3536-48af-b94d-7cb0c1ec1eb9.png)

# 它是如何工作的...

首先，让我们看一下`PersonMapper`接口是如何发现要执行的 SQL 语句的。如果您查看`src/main/resources/mappers/PersonMapper.xml`，您会发现`<mapper>`的`namespace`属性是`org.packt.boot_rest_demo.PersonMapper`。这是一个要求，即`namespace`属性的值应该是 mapper 接口的完全限定名称，在我们的例子中是`org.packt.boot_rest_demo.PersonMapper`。

接下来，`<select>`、`<insert>`、`<update>`和`<delete>`中定义的各个 SQL 语句的`id`属性应该与 mapper 接口中方法的名称匹配。例如，`PersonMapper`接口中的`getPersons()`方法寻找一个`id="getPersons"`的 SQL 语句。

现在，MyBatis 库通过读取`mybatis.mapper-locations`属性的值来发现 mapper XML 的位置。

关于控制器，我们引入了一个新的注解`@RestController`。这个特殊的注解表示，除了它是一个 web 控制器之外，类中定义的所有方法都返回通过 HTTP 响应体发送的响应；所有的 REST API 都是如此。它们只是处理数据。

通常情况下，您可以通过使用 Maven Spring Boot 插件`mvn spring-boot:run`或者执行 Maven 包创建的 JAR`java -jar my_jar_name.jar`来启动 Spring Boot 应用程序。

# 为 Spring Boot 创建多个配置文件

通常，Web 应用程序在不同的环境上部署 - 首先在开发人员的机器上本地运行，然后部署在测试服务器上，最后部署在生产服务器上。对于每个环境，我们希望应用程序与位于不同位置的组件进行交互。这种情况下的最佳方法是为每个环境维护不同的配置文件。其中一种方法是创建不同版本的`application.properties`文件，即存储应用程序级属性的文件的不同版本。Spring Boot 中的这些属性文件也可以是 YML 文件，比如`application.yml`。即使您创建了不同的版本，您也需要一种机制来告诉您的应用程序选择与其部署的环境相关的文件的相关版本。

Spring Boot 为这样的功能提供了令人惊叹的支持。它允许您拥有多个配置文件，每个文件代表一个特定的配置文件，然后，您可以根据部署的环境在不同的配置文件中启动应用程序。让我们看看它是如何运作的，然后我们将解释它是如何工作的。

# 准备工作

对于这个示例，有两种选项来托管另一个实例的 MySQL 数据库：

1.  使用云服务提供商，比如 AWS，并使用其 Amazon **关系型数据库服务**（**RDS**）([`aws.amazon.com/rds/`](https://aws.amazon.com/rds/))。它们有一定的免费使用限制。

1.  使用云服务提供商，比如 DigitalOcean ([`www.digitalocean.com/`](https://www.digitalocean.com/))，以每月 5 美元的价格购买一个 droplet（即服务器）。在其上安装 MySQL 服务器。

1.  使用 VirtualBox 在您的机器上安装 Linux，假设我们使用 Windows，或者如果您使用 Linux，则反之。在其上安装 MySQL 服务器。

选项非常多，从托管数据库服务到服务器，都可以让您完全控制 MySQL 服务器的安装。对于这个配方，我们做了以下工作：

1.  我们从 DigitalOcean 购买了一个基本的 droplet。

1.  我们使用`sudo apt-get install mysql-server-5.7`安装了 MySQL，并为 root 用户设置了密码。

1.  我们创建了另一个用户`springboot`，以便我们可以使用这个用户从我们的 RESTful Web 服务应用程序进行连接：

```java
 $ mysql -uroot -p
 Enter password: 
 mysql> create user 'springboot'@'%' identified by 'springboot';
```

1.  我们修改了 MySQL 配置文件，以便 MySQL 允许远程连接。这可以通过编辑`/etc/mysql/mysql.conf.d/mysqld.cnf`文件中的`bind-address`属性来完成。

1.  从 MySQL Workbench 中，我们通过使用`IP = <Digital Ocean droplet IP>`，`username = springboot`和`password = springboot`添加了新的 MySQL 连接。

在 Ubuntu OS 中，MySQL 配置文件的位置是`/etc/mysql/mysql.conf.d/mysqld.cnf`。找出特定于您的操作系统的配置文件位置的一种方法是执行以下操作：

1.  运行`mysql --help`。

1.  在输出中，搜索`Default options are read from the following files in the given order:`。接下来是 MySQL 配置文件的可能位置。

我们将创建所需的表并填充一些数据。但在此之前，我们将以`root`身份创建`sample`数据库，并授予`springboot`用户对其的所有权限：

```java
mysql -uroot
Enter password: 

mysql> create database sample;

mysql> GRANT ALL ON sample.* TO 'springboot'@'%';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
```

现在，让我们以`springboot`用户的身份连接到数据库，创建所需的表，并填充一些示例数据：

```java
mysql -uspringboot -pspringboot

mysql> use sample
Database changed
mysql> create table person(
-> id int not null auto_increment,
-> first_name varchar(255),
-> last_name varchar(255),
-> place varchar(255),
-> primary key(id)
-> );
Query OK, 0 rows affected (0.02 sec)

mysql> INSERT INTO person(first_name, last_name, place) VALUES('Mohamed', 'Sanaulla', 'Bangalore');
mysql> INSERT INTO person(first_name, last_name, place) VALUES('Nick', 'Samoylov', 'USA');

mysql> SELECT * FROM person;
+----+------------+-----------+-----------+
| id | first_name | last_name | place     |
+----+------------+-----------+-----------+
| 1  | Mohamed    | Sanaulla  | Bangalore |
| 2  | Nick       | Samoylov  | USA       |
+----+------------+-----------+-----------+
2 rows in set (0.00 sec)
```

现在，我们有了云实例的 MySQL 数据库准备就绪。让我们看看如何根据应用程序运行的配置文件管理两个不同连接的信息。

可以在`Chapter09/4_boot_multi_profile_incomplete`中找到此配方所需的初始示例应用程序。我们将转换此应用程序，使其在不同的环境中运行。

# 如何做到...

1.  在`src/main/resources/application.properties`文件中，添加一个新的`springboot`属性，`spring.profiles.active = local`。

1.  在`src/main/resources/`中创建一个新文件`application-local.properties`。

1.  将以下属性添加到`application-local.properties`中，并从`application.properties`文件中删除它们：

```java
  spring.datasource.url=jdbc:mysql://localhost/sample?useSSL=false
  spring.datasource.username=root
  spring.datasource.password=mohamed
```

1.  在`src/main/resources/`中创建另一个文件`application-cloud.properties`。

1.  将以下属性添加到`application-cloud.properties`中：

```java
      spring.datasource.url=
               jdbc:mysql://<digital_ocean_ip>/sample?useSSL=false
      spring.datasource.username=springboot
      spring.datasource.password=springboot
```

完整的应用程序代码可以在`Chapter09/4_boot_multi_profile_incomplete`中找到。您可以使用`mvn spring-boot:run`命令运行应用程序。Spring Boot 从`application.properties`文件中读取`spring.profiles.active`属性，并在本地配置文件中运行应用程序。在浏览器中打开`http://localhost:8080/api/persons`URL，以查看以下数据：

```java
[ 
  {
    "id": 1,
    "firstName": "David ",
    "lastName": "John",
    "place": "Delhi"
  },
  {
    "id": 2,
    "firstName": "Raj",
    "lastName": "Singh",
    "place": "Bangalore"
  }
]
```

现在，使用`mvn spring-boot:run -Dspring.profiles.active=cloud`命令在云配置文件上运行应用程序。然后，在浏览器中打开`http://localhost:8080/api/persons`，以查看以下数据：

```java
[
  {
    "id": 1,
    "firstName": "Mohamed",
    "lastName": "Sanaulla",
    "place": "Bangalore"
  },
  {
    "id": 2,
    "firstName": "Nick",
    "lastName": "Samoylov",
    "place": "USA"
  }
]
```

您可以看到相同 API 返回了不同的数据集，并且之前的数据是插入到我们在云上运行的 MySQL 数据库中的。因此，我们已成功地在两个不同的配置文件中运行了应用程序：本地和云端。

# 工作原理...

Spring Boot 可以以多种方式读取应用程序的配置。以下是一些重要的方式，按其相关性顺序列出（在较早的源中定义的属性会覆盖在后来的源中定义的属性）：

+   从命令行。属性使用`-D`选项指定，就像我们在云配置文件中启动应用程序时所做的那样，`mvn spring-boot:run -Dspring.profiles.active=cloud`。或者，如果您使用 JAR，它将是`java -Dspring.profiles.active=cloud -jar myappjar.jar`。

+   从 Java 系统属性，使用`System.getProperties()`。

+   操作系统环境变量。

+   特定配置文件，`application-{profile}.properties`或`application-{profile}.yml`文件，打包在 JAR 之外。

+   特定配置文件，`application-{profile}.properties`或`application-{profile}.yml`文件，打包在 JAR 中。

+   应用程序属性，`application.properties`或`application.yml`定义在打包的 JAR 之外。

+   应用程序属性，`application.properties`或`application.yml`打包在 JAR 中。

+   配置类（即使用`@Configuration`注释）作为属性源（使用`@PropertySource`注释）。

+   Spring Boot 的默认属性。

在我们的示例中，我们在`application.properties`文件中指定了所有通用属性，例如以下属性，并且任何特定配置文件中的特定配置属性都在特定配置文件中指定：

```java
spring.profiles.active=local
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

mybatis.mapper-locations=classpath*:mappers/*.xml
mybatis.configuration.map-underscore-to-camel-case=true  
```

从上面的列表中，我们可以发现`application.properties`或`application-{profile}.properties`文件可以在应用程序 JAR 之外定义。Spring Boot 将搜索属性文件的默认位置，其中一个路径是应用程序正在运行的当前目录的`config`子目录。

Spring Boot 支持的应用程序属性的完整列表可以在[`docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html`](http://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)找到。除了这些，我们可以创建自己的属性，这些属性将为我们的应用程序所需。

此示例的完整代码可以在`Chapter09/4_boot_multi_profile_complete`找到。

# 还有更多...

我们可以使用 Spring Boot 创建一个配置服务器，它将作为所有应用程序在所有配置文件中的所有属性的存储库。然后客户端应用程序可以连接到配置服务器，根据应用程序名称和应用程序配置读取相关属性。

在配置服务器中，可以使用类路径或 GitHub 存储库从文件系统读取应用程序属性。使用 GitHub 存储库的优势是属性文件可以进行版本控制。配置服务器中的属性文件可以更新，并且可以通过设置消息队列将这些更新推送到客户端应用程序的下游。另一种方法是使用`@RefreshScope` bean，然后在需要客户端应用程序拉取配置更改时调用`/refresh` API。

# 将 RESTful web 服务部署到 Heroku

**平台即服务**（**PaaS**）是云计算模型之一（另外两个是**软件即服务**（**SaaS**）和**基础设施即服务**（**IaaS**）），其中云计算提供商提供托管的计算平台，包括操作系统、编程语言运行时、数据库和其他附加组件，如队列、日志管理和警报。他们还为您提供工具来简化部署和监视应用程序的仪表板。

Heroku 是 PaaS 提供商领域中最早的参与者之一。它支持以下编程语言：Ruby、Node.js、Java、Python、Clojure、Scala、Go 和 PHP。Heroku 支持多个数据存储，如 MySQL、MongoDB、Redis 和 Elasticsearch。它提供与日志记录工具、网络实用程序、电子邮件服务和监视工具的集成。

Heroku 提供了一个名为 heroku-cli（[cli.heroku.com](http://cli.heroku.com)）的命令行工具，可用于创建 Heroku 应用程序，部署，监视，添加资源等。其 Web 仪表板提供的功能也受到 CLI 的支持。它使用 Git 存储应用程序的源代码。因此，当您将应用程序代码推送到 Heroku 的 Git 存储库时，它会触发一个构建，根据您使用的构建包进行构建。然后，它要么使用默认方式生成应用程序，要么使用`ProcFile`来执行您的应用程序。

在本示例中，我们将把基于 Spring Boot 的 RESTful Web 服务部署到 Heroku。我们将继续使用我们在上一个示例“为 Spring Boot 创建多个配置文件”中在另一个云提供商上创建的数据库。

# 准备工作

在继续在 Heroku 上部署我们的示例应用程序之前，我们需要注册 Heroku 帐户并安装其工具，这将使我们能够从命令行工作。在接下来的章节中，我们将指导您完成注册过程，通过 Web UI 创建示例应用程序，以及通过 Heroku 命令行界面（CLI）。

# 设置 Heroku 账户

访问[`www.heroku.com`](http://www.heroku.com)并注册账户。如果您已经有账户，可以登录。要注册，请访问[`signup.heroku.com`](https://signup.heroku.com)：

![](img/0adec601-5151-4e7f-9284-f2aa214d06b3.png)

要登录，URL 是[`id.heroku.com/login`](https://id.heroku.com/login)：

![](img/7628a737-18d7-4b89-8687-eb40e6609e82.png)

成功登录后，您将看到一个仪表板，列出了应用程序的列表，如果有的话：

![](img/778b9c57-33dd-4792-8caf-cae817772c6b.png)

# 从 UI 创建新应用

单击 New | Create new app，填写详细信息，然后单击 Create App：

![](img/f73432fd-4370-4bf1-aa8a-98407d34d47c.png)

# 从 CLI 创建新应用程序

执行以下步骤从 CLI 创建一个新应用程序：

1.  从[`cli.heroku.com`](https://cli.heroku.com)安装 Heroku CLI。

1.  安装后，Heroku 应该在系统的`PATH`变量中。

1.  打开命令提示符并运行`heroku create`。您将看到类似以下的输出：

```java
 Creating app... done, glacial-beyond-27911
 https://glacial-beyond-27911.herokuapp.com/ |
 https://git.heroku.com/glacial-beyond-27911.git
```

1.  应用程序名称是动态生成的，并创建了一个远程 Git 存储库。您可以通过运行以下命令指定应用程序名称和区域（与 UI 中所做的一样）：

```java
 $ heroku create test-app-9812 --region us
 Creating test-app-9812... done, region is us
 https://test-app-9812.herokuapp.com/ |
      https://git.heroku.com/test-app-9812.git
```

通过`git push`将部署到 Heroku 的代码推送到远程 Git 存储库。我们将在下一节中看到这一点。

我们在`Chapter09/5_boot_on_heroku`中有应用程序的源代码。因此，复制此应用程序，然后继续在 Heroku 上部署。

在运行 Heroku 的 cli 中的任何命令之前，您必须登录 Heroku 帐户。您可以通过运行`heroku login`命令来登录。

# 如何做...

1.  运行以下命令创建一个 Heroku 应用程序：

```java
 $ heroku create <app_name> -region us
```

1.  在项目文件夹中初始化 Git 存储库：

```java
 $ git init
```

1.  将 Heroku Git 存储库添加为本地 Git 存储库的远程存储库：

```java
 $ heroku git:remote -a <app_name_you_chose>
```

1.  将源代码，即主分支，推送到 Heroku Git 存储库：

```java
 $ git add .
 $ git commit -m "deploying to heroku"
 $ git push heroku master
```

1.  当代码推送到 Heroku Git 存储库时，会触发构建。由于我们使用 Maven，它运行以下命令：

```java
 ./mvnw -DskipTests clean dependency:list install
```

1.  代码完成构建和部署后，您可以使用`heroku open`命令在浏览器中打开应用程序。

1.  您可以使用`heroku logs --tail`命令监视应用程序的日志。

应用程序成功部署后，并且在运行`heroku open`命令后，您应该看到浏览器加载的 URL：

![](img/661af9df-5c65-4418-9727-ba2d9c7d00c5.png)

单击`Persons`链接将显示以下信息：

```java
[
  {
    "id":1,
    "firstName":"Mohamed",
    "lastName":"Sanaulla",
    "place":"Bangalore"
  },
  {
    "id":2,
    "firstName":"Nick",
    "lastName":"Samoylov",
    "place":"USA"
  }
]
```

有趣的是，我们的应用程序正在 Heroku 上运行，它正在连接到 DigitalOcean 服务器上的 MySQL 数据库。我们甚至可以为 Heroku 应用程序提供数据库并连接到该数据库。在*还有更多...*部分了解如何执行此操作。

# 还有更多...

1.  向应用程序添加新的 DB 附加组件：

```java
 $ heroku addons:create jawsdb:kitefin
```

在这里，`addons:create`接受附加组件名称和服务计划名称，两者用冒号(`:`)分隔。您可以在[`elements.heroku.com/addons/jawsdb-maria`](https://elements.heroku.com/addons/jawsdb-maria)上了解有关附加组件详细信息和计划的更多信息。此外，所有附加组件的附加组件详细信息页面末尾都提供了向应用程序添加附加组件的 Heroku CLI 命令。

1.  打开 DB 仪表板以查看连接详细信息，如 URL、用户名、密码和数据库名称：

```java
 $ heroku addons:open jawsdb
```

`jawsdb`仪表板看起来与以下类似：

![](img/6677a63b-02f6-4c51-8144-2aafa8f92507.png)

1.  甚至可以从`JAWSDB_URL`配置属性中获取 MySQL 连接字符串。您可以使用以下命令列出应用程序的配置：

```java
 $ heroku config
 === rest-demo-on-cloud Config Vars
 JAWSDB_URL: <URL>
```

1.  复制连接详细信息，在 MySQL Workbench 中创建一个新连接，并连接到此连接。数据库名称也是由附加组件创建的。连接到数据库后运行以下 SQL 语句：

```java
        use x81mhi5jwesjewjg;
        create table person( 
          id int not null auto_increment, 
          first_name varchar(255), 
          last_name varchar(255), 
          place varchar(255), 
          primary key(id)
        );

        INSERT INTO person(first_name, last_name, place) 
        VALUES('Heroku First', 'Heroku Last', 'USA');

        INSERT INTO person(first_name, last_name, place) 
        VALUES('Jaws First', 'Jaws Last', 'UK');
```

1.  在`src/main/resources`中为 Heroku 配置文件创建一个新的属性文件，名为`application-heroku.properties`，包含以下属性：

```java
        spring.datasource.url=jdbc:mysql://
        <URL DB>:3306/x81mhi5jwesjewjg?useSSL=false
        spring.datasource.username=zzu08pc38j33h89q
        spring.datasource.password=<DB password>

```

您可以在附加仪表板中找到与连接相关的详细信息。

1.  更新`src/main/resources/application.properties`文件，将`spring.profiles.active`属性的值替换为`heroku`。

1.  将更改提交并推送到 Heroku 远程：

```java
 $ git commit -am"using heroky mysql addon"
 $ git push heroku master
```

1.  部署成功后，运行`heroku open`命令。页面在浏览器中加载后，单击`Persons`链接。这次，您将看到一组不同的数据，这是我们在 Heroku 附加组件中输入的数据：

```java
        [
          {
            "id":1,
            "firstName":"Heroku First",
            "lastName":"Heroku Last",
            "place":"USA"
          },
          {
            "id":2,
            "firstName":"Jaws First",
            "lastName":"Jaws Last",
            "place":"UK"
          }
        ]
```

通过这种方式，我们已经与在 Heroku 中创建的数据库集成。

# 使用 Docker 对 RESTful Web 服务进行容器化

我们已经从将应用程序安装在服务器上的时代发展到每个服务器都被虚拟化，然后应用程序安装在这些较小的虚拟机上。通过添加更多虚拟机来解决应用程序的可扩展性问题，使应用程序运行到负载均衡器上。

在虚拟化中，通过在多个虚拟机之间分配计算能力、内存和存储来将大型服务器划分为多个虚拟机。这样，每个虚拟机本身都能够像服务器一样完成所有这些任务，尽管规模较小。通过虚拟化，我们可以明智地利用服务器的计算、内存和存储资源。

然而，虚拟化需要一些设置，即您需要创建虚拟机，安装所需的依赖项，然后运行应用程序。此外，您可能无法 100%确定应用程序是否能够成功运行。失败的原因可能是由于不兼容的操作系统版本，甚至是由于在设置过程中遗漏了一些配置或缺少了一些依赖项。这种设置还会导致水平扩展方面的一些困难，因为在虚拟机的配置和部署应用程序方面需要花费一些时间。

使用 Puppet 和 Chef 等工具确实有助于配置，但是应用程序的设置往往会导致由于缺少或不正确的配置而出现问题。这导致了另一个概念的引入，称为容器化。

在虚拟化世界中，我们有主机操作系统，然后是虚拟化软件，也就是 hypervisor。然后我们会创建多个机器，每台机器都有自己的操作系统，可以在上面部署应用程序。然而，在容器化中，我们不会划分服务器的资源。相反，我们有带有主机操作系统的服务器，然后在其上方有一个容器化层，这是一个软件抽象层。我们将应用程序打包为容器，其中容器只打包了运行应用程序所需的足够操作系统功能、应用程序的软件依赖项，以及应用程序本身。以下图片最好地描述了这一点：[`docs.docker.com/get-started/#container-diagram`](https://docs.docker.com/get-started/#containers-vs-virtual-machines)。

![](img/70fd5b75-23b5-4275-b212-bb5b974061dc.png)

前面的图片说明了典型的虚拟化系统架构。以下图片说明了典型的容器化系统架构：

![](img/0601180f-dc65-4c35-95b1-08f08095ce22.png)

容器化的最大优势在于将应用程序的所有依赖项捆绑到一个容器映像中。然后在容器化平台上运行此映像，从而创建一个容器。我们可以在服务器上同时运行多个容器。如果需要添加更多实例，我们只需部署映像，这种部署可以自动化以支持高可伸缩性。

Docker 是一种流行的软件容器化平台。在本示例中，我们将把位于`Chapter09/6_boot_with_docker`位置的示例应用程序打包成 Docker 映像，并运行 Docker 映像以启动我们的应用程序。

# 准备工作

对于此示例，我们将使用运行 Ubuntu 16.04.2 x64 的 Linux 服务器：

1.  从[`download.docker.com/linux/ubuntu/dists/xenial/pool/stable/amd64/`](https://download.docker.com/linux/ubuntu/dists/xenial/pool/stable/amd64/)下载最新的`.deb`文件。对于其他 Linux 发行版，您可以在[`download.docker.com/linux/`](https://download.docker.com/linux/)找到软件包：

```java
$ wget https://download.docker.com/linux/ubuntu/dists/xenial
 /pool/stable/amd64/docker-ce_17.03.2~ce-0~ubuntu-xenial_amd64.deb
```

1.  使用`dpkg`软件包管理器安装 Docker 软件包：

```java
 $  sudo dpkg -i docker-ce_17.03.2~ce-0~ubuntu-xenial_amd64.deb
```

包的名称将根据您下载的版本而变化。

1.  安装成功后，Docker 服务开始运行。您可以使用`service`命令验证这一点：

```java
 $ service docker status
 docker.service - Docker Application Container Engine
 Loaded: loaded (/lib/systemd/system/docker.service; enabled;
        vendor preset: enabled)
 Active: active (running) since Fri 2017-07-28 13:46:50 UTC;
              2min 3s ago
 Docs: https://docs.docker.com
 Main PID: 22427 (dockerd)
```

要 docker 化的应用程序位于`Chapter09/6_boot_with_docker`，在下载本书的源代码时可获得。

# 如何做...

1.  在应用程序的根目录创建`Dockerfile`，内容如下：

```java
 FROM ubuntu:17.10
      FROM openjdk:9-b177-jdk
 VOLUME /tmp
 ADD target/boot_docker-1.0.jar restapp.jar
 ENV JAVA_OPTS="-Dspring.profiles.active=cloud"
      ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -jar /restapp.jar" ]
```

1.  运行以下命令使用我们在前面步骤中创建的`Dockerfile`构建 Docker 映像：

```java
 $ docker build --tag restapp-image .

 Sending build context to Docker daemon 18.45 MB
 Step 1/6 : FROM ubuntu:17.10
 ---> c8cdcb3740f8
 Step 2/6 : FROM openjdk:9-b177-jdk
 ---> 38d822ff5025
 Step 3/6 : VOLUME /tmp
 ---> Using cache
 ---> 38367613d375
 Step 4/6 : ADD target/boot_docker-1.0.jar restapp.jar
 ---> Using cache
 ---> 54ad359f53f7
 Step 5/6 : ENV JAVA_OPTS "-Dspring.profiles.active=cloud"
 ---> Using cache
 ---> dfa324259fb1
 Step 6/6 : ENTRYPOINT sh -c java $JAVA_OPTS -jar /restapp.jar
 ---> Using cache
 ---> 6af62bd40afe
 Successfully built 6af62bd40afe
```

1.  您可以使用以下命令查看已安装的映像：

```java
 $ docker images

      REPOSITORY     TAG        IMAGE ID     CREATED     SIZE
 restapp-image  latest     6af62bd40afe 4 hours ago 606 MB
 openjdk        9-b177-jdk 38d822ff5025 6 days ago  588 MB
 ubuntu         17.10      c8cdcb3740f8 8 days ago  93.9 MB
```

您会看到还有 OpenJDK 和 Ubuntu 映像。这些是下载用于构建我们应用程序的映像的。首先列出的是我们的应用程序。

1.  运行映像以创建包含我们正在运行的应用程序的容器：

```java
 docker run -p 8090:8080 -d --name restapp restapp-image
 d521b9927cec105d8b69995ef6d917121931c1d1f0b1f4398594bd1f1fcbee55
```

在`run`命令之后打印的长字符串是容器的标识符。您可以使用前几个字符来唯一标识容器。或者，您可以使用容器名称`restapp`。

1.  应用程序已经启动。您可以通过运行以下命令查看日志：

```java
 docker logs restapp
```

1.  您可以使用以下命令查看已创建的 Docker 容器：

```java
 docker ps
```

前面命令的输出类似于以下内容：

![](img/bf9fa9e9-2603-47ff-93ca-65a4a13ba499.png)

1.  您可以使用以下命令管理容器：

```java
 $ docker stop restapp
 $ docker start restapp
```

应用程序运行后，打开`http://<hostname>:8090/api/persons`。

# 它是如何工作的...

通过定义`Dockerfile`来定义容器结构和其内容。`Dockerfile`遵循一种结构，其中每一行都是`INSTRUCTION arguments`的形式。有一组预定义的指令，即`FROM`、`RUN`、`CMD`、`LABEL`、`ENV`、`ADD`和`COPY`。完整的列表可以在[`docs.docker.com/engine/reference/builder/#from`](https://docs.docker.com/engine/reference/builder/#from)上找到。让我们看看我们定义的`Dockerfile`：

```java
FROM ubuntu:17.10
FROM openjdk:9-b177-jdk
VOLUME /tmp
ADD target/boot_docker-1.0.jar restapp.jar
ENV JAVA_OPTS="-Dspring.profiles.active=cloud"
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -jar /restapp.jar" ]
```

使用`FROM`指令的前两行指定了我们的 Docker 镜像的基础镜像。我们使用 Ubuntu OS 镜像作为基础镜像，然后将其与 OpenJDK 9 镜像结合在一起。`VOLUME`指令用于指定镜像的挂载点。这通常是主机操作系统中的路径。

`ADD`指令用于将文件从源复制到工作目录下的目标目录。`ENV`指令用于定义环境变量。

`ENTRYPOINT`指令用于配置容器以作为可执行文件运行。对于此指令，我们传递一个参数数组，否则我们将直接从命令行执行。在我们的场景中，我们使用 bash shell 来运行`java -$JAVA_OPTS -jar <jar name>`。

一旦我们定义了`Dockerfile`，我们就指示 Docker 工具使用`Dockerfile`构建一个镜像。我们还使用`--tag`选项为镜像提供一个名称。构建我们的应用程序镜像时，它将下载所需的基础镜像，这在我们的情况下是 Ubuntu 和 OpenJDK 镜像。因此，如果列出 Docker 镜像，您将看到基础镜像以及我们的应用程序镜像。

这个 Docker 镜像是一个可重用的实体。如果我们需要更多的应用程序实例，我们可以使用`docker run`命令生成一个新的容器。当我们运行 Docker 镜像时，有多个选项，其中一个是`-p`选项，它将容器内的端口映射到主机操作系统。在我们的情况下，我们将 Spring Boot 应用程序的`8080`端口映射到主机操作系统的`8090`端口。

现在，要检查我们运行的应用程序的状态，我们可以使用`docker logs restapp`来检查日志。除此之外，`docker`工具支持多个命令。强烈建议运行`docker help`并探索支持的命令。

Docker 是 Docker 容器化平台背后的公司，它创建了一组基础镜像，可以用来创建容器。有用于 MySQL DB、Couchbase、Ubuntu 和其他操作系统的镜像。您可以在[`store.docker.com/`](https://store.docker.com/)上探索这些软件包。

# 使用 Micrometer 和 Prometheus 监控 Spring Boot 2 应用程序

监控和收集性能指标是应用程序开发和维护的重要部分。人们对内存使用情况、各个端点的响应时间、CPU 使用情况、机器负载、垃圾收集频率和暂停等指标感兴趣。有不同的方法来启用捕获指标，例如使用 Dropwizard Metrics（[`metrics.dropwizard.io/4.0.0/`](https://metrics.dropwizard.io/4.0.0/)）或 Spring Boot 的度量框架。

Spring Boot 2 及更高版本中的代码仪器是使用一个名为 Micrometer 的库（[`micrometer.io/`](https://micrometer.io/)）来完成的。Micrometer 提供了一个供应商中立的代码仪器，这样您就可以使用任何监控工具，并且 Micrometer 以工具理解的格式提供度量数据。这就像 SLF4J 用于日志记录一样。它是对以供应商中立方式产生输出的度量端点的外观。

Micrometer 支持诸如 Prometheus ([`prometheus.io/`](https://prometheus.io/))、Netflix Atlas ([`github.com/Netflix/atlas`](https://github.com/Netflix/atlas))、Datadog ([`www.datadoghq.com/`](https://www.datadoghq.com/))以及即将支持的 InfluxDB ([`www.influxdata.com/`](https://www.influxdata.com/))、statsd ([`github.com/etsy/statsd`](https://github.com/etsy/statsd))和 Graphite ([`graphiteapp.org/`](https://graphiteapp.org/))等工具。使用早期版本的 Spring Boot，如 1.5，的应用程序也可以使用这个新的仪表化库，如*还有更多...*部分所示。

在本食谱中，我们将使用 Micrometer 来为我们的代码进行仪表化，并将指标发送到 Prometheus。因此，首先，我们将从*准备工作*部分开始设置 Prometheus。

# 准备工作

Prometheus ([`prometheus.io/`](https://prometheus.io/))是一个监控系统和时间序列数据库，允许我们存储时间序列数据，其中包括应用程序随时间变化的指标，一种简单的可视化指标的方法，或者在不同指标上设置警报。

让我们执行以下步骤在我们的机器上运行 Prometheus（在我们的情况下，我们将在 Windows 上运行。类似的步骤也适用于 Linux）：

1.  从[`github.com/prometheus/prometheus/releases/download/v2.3.2/prometheus-2.3.2.windows-amd64.tar.gz`](https://github.com/prometheus/prometheus/releases/download/v2.3.2/prometheus-2.3.2.windows-amd64.tar.gz)下载 Prometheus 分发版。

1.  在 Windows 上使用 7-Zip ([`www.7-zip.org/`](https://www.7-zip.org/))将其提取到一个我们将称为`PROMETHEUS_HOME`的位置。

1.  将`%PROMETHEUS_HOME%`添加到您的 PATH 变量（在 Linux 上，它将是`$PROMETHEUS_HOME`到 PATH 变量）。

1.  使用`prometheus --config "%PROMETHEUS_HOME%/prometheus.yml"`命令运行 Prometheus。您将看到以下输出：

![](img/804808bc-52af-40a7-be2c-1ea1286bcde8.png)

1.  在浏览器中打开`http://localhost:9090`，以查看 Prometheus 控制台。在空文本框中输入`go_gc_duration_seconds`，然后单击执行按钮以显示捕获的指标。您可以切换到图形版本的选项卡以可视化数据：

![](img/7a5e02c6-6401-4eb0-be61-01832f3f8fc4.png)

前面的指标是用于 Prometheus 本身的。您可以导航到`http://localhost:9090/targets`以查找 Promethues 监视的目标，如下所示：

![](img/f07d688a-c972-46fc-9fbb-63bb1da19831.png)

当您在浏览器中打开`http://localhost:9090/metrics`时，您将看到当前时间点的指标值。没有可视化很难理解。这些指标在随时间收集并使用图表可视化时非常有用。

现在，我们已经启动并运行了 Prometheus。让我们启用 Micrometer 和以 Prometheus 理解的格式发布指标。为此，我们将重用本章中*与数据库交互*食谱中使用的代码。此食谱位于`Chapter09/2_boot_db_demo`。因此，我们将只需将相同的代码复制到`Chapter09/7_boot_micrometer`，然后增强部分以添加对 Micrometer 和 Prometheus 的支持，如下一节所示。

# 如何做...

1.  更新`pom.xml`以包括 Spring boot 执行器和 Micrometer Prometheus 注册表依赖项：

```java
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
  <version>1.0.6</version>
</dependency>
```

在 Spring Boot 2 及更高版本中，Micrometer 已配置了执行器，因此我们只需要将执行器作为依赖项添加，然后`micrometer-registry-prometheus`依赖项会生成一个 Prometheus 理解的指标表示。

1.  当我们运行应用程序（一种方式是运行`mvn spring-boot:run`）并打开执行器端点时，默认情况下将是`<root_url>/actuator`。我们会发现默认情况下有一些执行器端点可用，但 Prometheus 指标端点不是其中的一部分：

![](img/02e97a6b-0504-47fb-bbc7-93e9c63e2a59.png)

1.  要在执行器中启用 Prometheus 端点，我们需要在`src/main/resources/application.properties`文件中添加以下属性：

```java
management.endpoints.web.exposure.include=prometheus
```

1.  重新启动应用程序并浏览`http://localhost:8080/actuator/`。现在，您会发现只有 Prometheus 端点可用：

![](img/ddf35ffd-8415-47d6-bd39-15c400d80707.png)

1.  打开`http://localhost:8080/actuator/prometheus`以查看 Prometheus 理解的格式中的指标：

![](img/07b55bf4-6d26-4cd8-88d4-bbfda24bbb31.png)

1.  配置 Prometheus 以在特定频率下调用`http://localhost:8080/actuator/prometheus`，可以进行配置。这可以通过在`%PROMETHEUS_HOME%/prometheus.yml`配置文件中在`scrape_configs`属性下更新新作业来完成：

```java
- job_name: 'spring_apps'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['localhost:8080']
```

您会看到，默认情况下，有一个作业来抓取 Prometheus 指标本身。

1.  重新启动 Prometheus 服务器并访问`http://localhost:9090/targets`。您将看到一个新的部分`spring_apps`，其中包含我们添加的目标：

![](img/83686243-24fd-45a4-a616-2473f8fc4249.png)

1.  我们可以通过访问`http://localhost:9090/graph`，在文本框中输入`jvm_memory_max_bytes`，然后单击执行来绘制从指标捕获的指标的图表：

![](img/92e2a62a-ac7d-4d2c-a45f-717c9058a108.png)

因此，我们最终设置了在 Prometheus 中摄取指标并根据指标值创建图表。

# 它是如何工作的...

Spring Boot 提供了一个名为执行器的库，具有在部署到生产环境时帮助您监视和管理应用程序的功能。这种开箱即用的功能不需要开发人员进行任何设置。因此，您可以在没有任何工作的情况下进行审计、健康检查和指标收集。

如前所述，执行器使用 Micrometer 从代码中进行仪表化和捕获不同的指标，例如：

+   JVM 内存使用情况

+   连接池信息

+   应用程序中不同 HTTP 端点的响应时间

+   不同 HTTP 端点调用的频率

要使应用程序具有这些生产就绪的功能，如果您使用 Maven，需要将以下依赖项添加到您的`pom.xml`中（Gradle 也有相应的依赖项）：

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

默认情况下，执行器位于`/actuator`端点，但可以通过在`src/main/resources/application.properties`文件中覆盖`management.endpoints.web.base-path`属性来配置不同的值，如下所示：

```java
management.endpoints.web.base-path=/metrics
```

默认情况下，除了`/shutdown`端点之外，所有用于监视和审计应用程序的端点都是默认启用的。默认情况下禁用此端点。此端点用于关闭应用程序。以下是一些可用的端点：

| `auditevents` | 公开当前应用程序的审计事件信息 |
| --- | --- |
| `beans` | 显示应用程序中所有 Spring bean 的完整列表 |
| `env` | 公开 Spring 的`ConfigurableEnvironment`中的属性 |
| `health` | 显示应用程序健康信息 |
| `info` | 显示任意应用程序信息 |
| `metrics` | 显示当前应用程序的指标信息 |
| `mappings` | 显示所有`@RequestMapping`路径的汇总列表 |
| `prometheus` | 以 Prometheus 服务器可以抓取的格式公开指标 |

您会发现这些是非常敏感的端点，需要进行安全保护。好消息是，Spring Boot 执行器与 Spring Security 很好地集成，以保护这些端点。因此，如果 Spring Security 在类路径上，它将默认安全地保护这些端点。

这些端点可以通过 JMX 或通过 Web 访问。默认情况下，并非所有执行器端点都启用了 Web 访问，而是默认情况下启用了使用 JMX 访问。默认情况下，只有以下属性可以通过 Web 访问：

+   `health`

+   `info`

这就是我们不得不添加以下配置属性的原因，以便在 Web 上提供 Prometheus 端点，以及健康、信息和指标：

```java
management.endpoints.web.exposure.include=prometheus,health,info,metrics
```

即使我们启用了 Prometheus，我们也需要在类路径上有`micrometer-registry-prometheus`库。只有这样，我们才能以 Prometheus 的格式查看指标。因此，我们将以下依赖项添加到我们的 pom 中：

```java
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
  <version>1.0.6</version>
</dependency>
```

Prometheus 处理的输出格式很简单：它以`<property_name value>`的形式接收，每个属性都在新的一行中。Spring Boot 执行器不会将指标推送到 Prometheus；相反，我们配置 Prometheus 以在其配置中定义的频率从给定 URL 拉取指标。Prometheus 的默认配置，可在其主目录中找到，如下所示：

```java
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090']
```

因此，它配置了 Prometheus 将获取指标的间隔和评估`rule_files`下定义的规则的间隔的默认值。Scrape 是从`scrape_configs`选项下定义的不同目标中拉取指标的活动，而 evaluate 是评估`rule_files`中定义的不同规则的行为。为了使 Prometheus 能够从我们的 Spring Boot 应用程序中抓取指标，我们通过提供作业名称、相对于应用程序 URL 的指标路径和应用程序的 URL，在`scrape_configs`下添加了一个新的作业：

```java
- job_name: 'spring_apps'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['localhost:8080']
```

我们还看到了如何从`http://localhost:9090/graph`查看这些指标的值，以及如何使用 Prometheus 提供的简单图形支持来可视化这些指标。

# 还有更多

通过配置另一个名为 Alertmanager 的服务（[`prometheus.io/docs/alerting/alertmanager/`](https://prometheus.io/docs/alerting/alertmanager/)），可以在 Prometheus 中启用警报。该服务可用于向电子邮件、寻呼机等发送警报。

Prometheus 中的图形支持是天真的。您可以使用 Grafana（[`grafana.com/`](https://grafana.com/)），这是一种领先的开源软件，用于分析时间序列数据，例如存储在 Prometheus 中的数据。通过这种方式，您可以配置 Grafana 从 Prometheus 读取时间序列数据，并构建具有预定义指标的不同类型图表的仪表板。
