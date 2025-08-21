# 第十一章：Spring 与 Thymeleaf 集成

Thymeleaf 是一个完全用 Java 编写的模板引擎。它支持 XML/XHTML/HTML5，这意味着我们可以使用 Thymeleaf 模板引擎库使用 XML 或 XHTML 或 HTML5 开发模板。它提供了一个可选的模块，用于 Spring MVC 和 Spring Web Flow 集成。模板引擎帮助我们在 UI 中创建可重用的组件。模板通常按照约定包括标题、菜单、消息、正文、内容和页脚组件。内容部分动态加载消息。我们可以使用模板创建不同的布局。

Thymeleaf 可以用来代替 JSP。到目前为止，我们已经使用了 JSP 和自定义标签制作模板。Thymeleaf 模板是 XHTML、XML、HTML5 模板引擎。甚至网页设计师也可以很容易地与之交互。所使用的表达语言与 JSP 表达语言相比非常先进。

在本章中，我们将演示如何将 Spring MVC 与 Thymeleaf 模板集成。我们将看到如何使用可用的依赖项开始使用 Spring Thymeleaf。

# Thymeleaf 属性

让我们看一些 Thymeleaf 提供的用于设计页面的基本属性。我们还将看一下它如何与 Java 对象和循环交互。Thymeleaf 使用了许多属性。

+   显示消息：

```java
<p th:text="#{msg.greet}">Helloo Good Morning!</p>
```

+   要显示循环，我们有`th:each`：

```java
<li th:each="product : ${products}" th:text="${product.title}">XYZLLDD</li>
```

+   现在，让我们看一个表单提交操作：

```java
<form th:action="@{/buyBook}">
```

+   如果我们必须提交按钮，那么添加：

```java
<input type="button" th:value="#{form.submit}" />
```

# Spring Thymeleaf 依赖

要开始使用 Thymeleaf 模板引擎，我们需要在`pom.xml`文件中添加以下依赖项：

+   Thyemleaf 库：

+   `groupId`: `org.thymeleaf`

+   `artifactId`: `thymeleaf`

+   `version`: 2.1.4 Release

+   Spring-Thymeleaf 插件库：

+   `groupId`: `org.thymeleaf`

+   `artifactId`: `thymeleaf-spring4`

+   `version`: 2.1.4\. Release

为了测试框架（注意版本不一定与核心版本匹配），Thymeleaf 需要 Java SE 5.0 或更新版本。此外，它依赖于以下库：

+   unbescape 1.1.0 或更高版本

+   ONGL 3.0.8 或更高版本

+   Javassist 3.16.1-GA 或更高版本

+   slf4j 1.6.6 或更高版本

+   此外，如果您使用 LEGACYHTML5 模板模式，则需要 NekoHTML 1.9.21 或更高版本

## Spring MVC 和 Thymeleaf

在本节中，让我们看一下如何在 Spring MVC 框架中配置 Thymeleaf。我们也可以使用`SpringContext.xml`文件进行 Thymeleaf 配置，但由于我们已经看到了许多这样的例子，其中我们在 XML 文件中执行了配置，我们将看一下如何在 Java 文件中使用 Spring 注解添加配置。让我们创建一个简单的类`CustomPacktConfiguration`，并为该类使用`@Configuration`注解，告诉框架这个类有配置。

在配置类中，将模板模式设置为应用程序中使用的格式，即 XHTML 或 XML 模板。然后我们需要将模板配置设置为`thymeleafviewResolver`对象，还需要实际传递`templateResolver`类。

```java
@Configuration
@ComponentScan(basePackageClasses = PacktController.class)
public class CutomPacktConfiguration {
  @Bean public ViewResolver viewResolver() {
    ClassLoaderTemplateResolver templateResolver = new ClassLoaderTemplateResolver();
    templateResolver.setTemplateMode("XHTML");
    templateResolver.setPrefix("views/");
    templateResolver.setSuffix(".html");
    SpringTemplateEngine engine = new SpringTemplateEngine();
    engine.setTemplateResolver(templateResolver);
    ThymeleafViewResolver thymeleafviewResolver = new ThymeleafViewResolver();
    thymeleafviewResolver.setTemplateEngine(engine);
    return thymeleafviewResolver;
    }
  }

@Controller
public class MyPacktControllerController {
  @Autowired private PacktService packtService;
  @RequestMapping("/authors")
  public String authors(Model model) {
    model.addAttribute("authors",packtService.getAuthors));
    return "authors";
  }

}
```

## 使用 Spring Thymeleaf 的 MVC

在本节中，我们将深入探讨 Thymeleaf 在 Spring 应用程序中的集成，并开发一个简单的 MVC 应用程序，列出作者并允许用户添加、编辑和删除作者。在 Java 文件中进行配置而不是在 XML 文件中进行配置的优势是代码安全性。您的 XML 可以很容易被更改，但在 Java 文件中进行配置的情况下，我们可能需要将类文件部署到服务器上以查看更改。在本例中，让我们使用`JavaConfig`方法来配置 bean。我们可以省略 XML 配置文件。

1.  让我们首先从控制器开始，它有方法来插入和列出数据库中可用的作者。

```java
package demo.packt.thymeleaf.controller;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.servlet.ModelAndView;
import demo.packt.thymeleaf.exception.AuthorFoundException;
import demo.packt.thymeleaf.model.Author;
import demo.packt.thymeleaf.model.AuthorData;
import demo.packt.thymeleaf.service.AuthorService;

@Controller
public class AuthorController {
  private static final String HOME_VIEW = "home";
  private static final String RESULTS_FRAGMENT = "results :: resultsList";

  @Autowired
  private AuthorService authorService;

  @ModelAttribute("author")
  public Author prepareAuthorModel() {
    return new Author();
  }

  @ModelAttribute("authorData")
  public AuthorData prepareAuthorDataModel() {
    return authorService.getAuthorData();
  }

  @RequestMapping(value = "/home", method = RequestMethod.GET)
  public String showHome(Model model) {
    prepareAuthorDataModel();
    prepareAuthorModel();
    return HOME_VIEW;
  }

  @RequestMapping(value = "/authors/{surname}", method = RequestMethod.GET)
  public String showAuthorListwithSurname(Model model, @PathVariable("surname") String surname) {
    model.addAttribute("authors", authorService.getAuthorsList(surname));
    return RESULTS_FRAGMENT;
  }

  @RequestMapping(value = "/authors", method = RequestMethod.GET)
  public String showAuthorList(Model model) {
    model.addAttribute("authors", authorService.getAuthorsList());
    return RESULTS_FRAGMENT;
  }

  @RequestMapping(value = "/authors/insert", method = RequestMethod.POST)
  public String insertAuthor(Author newAuthor, Model model) {
    authorService.insertNewAuthor(newAuthor);
    return showHome(model);
  }

  @ExceptionHandler({AuthorFoundException.class})
  public ModelAndView handleDatabaseError(AuthorFoundException e) {
    ModelAndView modelAndView = new ModelAndView();
    modelAndView.setViewName("home");
    modelAndView.addObject("errorMessage", "error.user.exist");
    modelAndView.addObject("Author", prepareAuthorModel());
    modelAndView.addObject("authorData", prepareAuthorDataModel());

    return modelAndView;
  }
}
```

1.  接下来通过扩展`RuntimeException`类定义自定义`RuntimeException`：

```java
package demo.packt.thymeleaf.exception;
public class AuthorFoundException extends RuntimeException {
  private static final long serialVersionUID = -3845574518872003019L;
  public AuthorFoundException() {
    super();
  }
  public AuthorFoundException(String message) {
    super(message);
  }
}
```

1.  在这一步中，我们将从 Thymeleaf 服务开始，编写一个接口和实现类。

+   接口描述了接口中使用的方法：

```java
package demo.packt.thymeleaf.service;
import java.util.List;
import demo.packt.thymeleaf.model.Author;
import demo.packt.thymeleaf.model.AuthorData;
public interface AuthorService {
  HotelData getAuthorData();
  List<Author> getAuthorsList();
  List<Author> getAuthorList(String surname);
  void insertNewAuthor(Author newAuthor);
}
```

+   接下来我们将实现接口：

```java
package demo.packt.thymeleaf.service;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import demo.packt.thymeleaf.exception.AuthorFoundException;
import demo.packt.thymeleaf.model.Author;
import demo.packt.thymeleaf.model.AuthorData;
import demo.packt.thymeleaf.repository.AuthorRepository;

@Service("authorServiceImpl")
public class AuthorServiceImpl implements AuthorService {
  @Autowired
  AuthorRepository authorRepository;
  @Override
  public AuthorData getAuthorData() {
    AuthorData data = new AuthorData();
    data.setAddress("RRNAGAR, 225");
    data.setName("NANDA");
    return data;
  }
  @Override
  public List<Author> getAuthorsList() {
    return authorRepository.findAll();
  }
  @Override
  public List<Author> getAuthorsList(String surname) {
    return authorRepository.findAuthorsBySurname(surname);
  }

  @Override
  public void insertNewGuest(Author newAuthor) {
    if (authorRepository.exists(newAuthor.getId())) {
      throw new AuthorFoundException();
    }
    authorRepository.save(newAuthor);
  }
}
```

1.  让我们实现应用程序服务实现类中使用的存储库类：

```java
package demo.packt.thymeleaf.repository;
import java.util.List;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.mongodb.repository.Query;
import demo.packt.thymeleaf.model.Guest;
public interface AuthorRepository extends MongoRepository<Author, Long> {
  @Query("{ 'surname' : ?0 }")
  List<Author> findAuthorsBySurname(String surname);
}
```

1.  接下来在应用程序中实现 Model 类（`Author`和`AuthorData`）。

+   首先让我们实现`Author`类：

```java
package demo.packt.thymeleaf.model;
import java.io.Serializable;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
@Document(collection = "authors")
public class Author implements Serializable {
  private static final long serialVersionUID = 1L;
  @Id
  private Long id;
  private String name;
  private String surname;
  private String country;

  /**
   * @return the name
   */
  public String getName() {
    return name;
  }
  /**
   * @param name the name to set
   */
  public void setName(String name) {
    this.name = name;
  }
  /**
   * @return the surname
   */
  public String getSurname() {
    return surname;
  }
  /**
   * @param surname the surname to set
   */
  public void setSurname(String surname) {
    this.surname = surname;
  }
  /**
   * @return the id
   */
  public Long getId() {
    return id;
  }
  /**
   * @param id the id to set
   */
  public void setId(Long id) {
    this.id = id;
  }
  /**
   * @return the country
   */
  public String getCountry() {
    return country;
  }
  /**
   * @param country the country to set
   */
  public void setCountry(String country) {
    this.country = country;
  }
}
```

+   接下来，让我们实现`AuthorData`类：

```java
package demo.packt.thymeleaf.model;
import java.io.Serializable;
public class AuthorData implements Serializable {
  private static final long serialVersionUID = 1L;
  private String name;
  private String address;
  public String getName() {
    return name;
  }
  public void setName(String name) {
    this.name = name;
  }
  public String getAddress() {
    return address;
  }
  public void setAddress(String address) {
    this.address = address;
  }
}
```

1.  在这一步中，我们将创建配置类；如前所述，我们不使用 XML 进行配置。我们有两个配置文件——一个用于数据库配置的 MongoDB，另一个是组件扫描配置文件：

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.data.mongodb.config.AbstractMongoConfiguration;
import org.springframework.data.mongodb.repository.config.EnableMongoRepositories;
import com.mongodb.Mongo;
@Configuration
@EnableMongoRepositories(«demo.packt.thymeleaf.repository»)
public class MongoDBConfiguration extends AbstractMongoConfiguration {
  @Override
  protected String getDatabaseName() {
    return "author-db";
  }
  @Override
  public Mongo mongo() throws Exception {
    return new Mongo();
  }
}
```

这个类是一个重要的类，标志着应用程序实例化的开始。在这里，我们还配置了 Thymeleaf 模板视图解析器并提供了组件扫描信息。模板和视图解析器也在这个类中进行了配置：

```java
package demo.packt.thymeleaf.configuration;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Description;
import org.springframework.context.annotation.Import;
import org.springframework.context.support.ResourceBundleMessageSource;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.thymeleaf.spring3.SpringTemplateEngine;
import org.thymeleaf.spring3.view.ThymeleafViewResolver;
import org.thymeleaf.templateresolver.ServletContextTemplateResolver;

@EnableWebMvc
@Configuration
@ComponentScan("demo.packt.thymeleaf")
@Import(MongoDBConfiguration.class)
public class WebAppConfiguration extends WebMvcConfigurerAdapter {

  @Bean
  @Description("Thymeleaf template resolver serving HTML 5")
  public ServletContextTemplateResolver templateResolver() {
    ServletContextTemplateResolver templateResolver = new ServletContextTemplateResolver();
    templateResolver.setPrefix("/WEB-INF/html/");
    templateResolver.setSuffix(".html");
    templateResolver.setTemplateMode("HTML5");

    return templateResolver;
  }

  @Bean
  @Description("Thymeleaf template engine with Spring integration")
  public SpringTemplateEngine templateEngine() {
    SpringTemplateEngine templateEngine = new SpringTemplateEngine();
    templateEngine.setTemplateResolver(templateResolver());

    return templateEngine;
  }

  @Bean
  @Description("Thymeleaf view resolver")
  public ThymeleafViewResolver viewResolver() {
    ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
    viewResolver.setTemplateEngine(templateEngine());

    return viewResolver;
  }

  @Bean
  @Description("Spring message resolver")
  public ResourceBundleMessageSource messageSource() {
    ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();  
    messageSource.setBasename("i18n/messages");

    return messageSource;  
  }

  @Override
  public void addResourceHandlers(ResourceHandlerRegistry registry) {
    registry.addResourceHandler("/resources/**").addResourceLocations("/WEB-INF/resources/");
  }
}
```

1.  下一步是在`WEB-INF`文件夹下创建 HTML 文件，创建一个`home.html`文件如下：

```java
<!DOCTYPE html>
<html 
       lang="en">

<head>
<meta charset="UTF-8"/>
<title>Thymeleaf example</title>
<link rel="stylesheet" th:href="@{/spring/resources/css/styles.css}" type="text/css" media="screen"/>
<script th:src="img/functions.js}" type="text/javascript"></script>
<script th:src="img/jquery-min-1.9.1.js}" type="text/javascript"></script>
</head>

<body>
<div style="width:800px; margin:0 auto;">

<h1 th:text="#{home.title}">Thymeleaf example</h1>

<div class="generic-info">
  <h3 th:text="#{author.information}">Author Information</h3>

  <span th:text="${authorData.name}">Author name</span><br />
  <span th:text="${authorData.address}">Author address</span><br />
</div>

<div class="main-block">
  <!-- Insert new Author -->
  <span class="subtitle">Add Author form</span>
  <form id="guestForm" th:action="@{/spring/authors/insert}" th:object="${Author}" method="post">
    <div class="insertBlock">
    <span class="formSpan">
    <input id="authorId" type="text" th:field="*{id}" required="required"/>
    <br />
    <label for="authorId" th:text="#{insert.id}">id:</label>
    </span>
    <span class="formSpan" style="margin-bottom:20px">
    <input id="authorName" type="text" th:field="*{name}" required="required"/>
      <br />
      <label for="authorName" th:text="#{insert.name}">name:</label>
    </span>

    <span class="formSpan">
    <input id="authorSurname" type="text" th:field="*{surname}" required="required"/>
    <br />
    <label for="authorSurname" th:text="#{insert.surname}">surname:</label>
    </span>
    <span class="formSpan" style="margin-bottom:20px">
    <input id="authorCountry" type="text" th:field="*{country}" required="required"/>
    <br />
    <label for="authorCountry" th:text="#{insert.country}">country:</label>
    </span>

    <input type="submit" value="add" th:value="#{insert.submit}"/>
    <span class="messageContainer" th:unless="${#strings.isEmpty(errorMessage)}" th:text="#{${errorMessage}}"></span>
    </div>
  </form>
  <!-- Guests list -->
  <form>
    <span class="subtitle">Author list form</span>
    <div class="listBlock">
    <div class="search-block">
    <input type="text" id="searchSurname" name="searchSurname"/>
    <br />
    <label for="searchSurname" th:text="#{search.label}">Search label:</label>

    <button id="searchButton" name="searchButton" onclick="retrieveAuthors()" type="button" th:text="#{search.button}">Search button</button>
    </div>

    <!-- Results block -->
    <div id="resultsBlock">

    </div>
    </div>

  </form>
</div>

</div>
</body>
</html>
```

1.  最后，创建一个简单的`results.html`文件：

```java
<!DOCTYPE html>
<html 
   lang="en">
<head>
</head>
<body>
  <div th:fragment="resultsList" th:unless="${#lists.isEmpty(authors)}" class="results-block">
  <table>
  <thead>
  <tr>
  <th th:text="#{results.author.id}">Id</th>
  <th th:text="#{results.author.surname}">Surname</th>
  <th th:text="#{results.author.name}">Name</th>
  <th th:text="#{results.author.country}">Country</th>
  </tr>
  </thead>
  <tbody>
  <tr th:each="author : ${authors}">
  <td th:text="${author.id}">id</td>
  <td th:text="${author.surname}">surname</td>
  <td th:text="${author.name}">name</td>
  <td th:text="${author.country}">country</td>
  </tr>
  </tbody>
  </table>
  </div>
</body>
</html>
```

这将为用户提供一个作者列表和一个用于将作者信息插入 MongoDB 数据库的表单，使用 Thymeleaf 模板。

# Spring Boot 与 Thymeleaf 和 Maven

在本节中，我们将看到如何使用 Spring boot 创建一个带有 Thymeleaf 应用程序的 Spring。

这个操作的前提是 Maven 必须安装。要检查 Maven 是否已安装，请在命令提示符中键入以下命令：

```java
mvn –version

```

1.  使用原型来生成一个带有`thymeleaf`项目的 Spring boot：

```java
mvn archetype:generate -DarchetypeArtifactId=maven-archetype-quickstart -DgroupId=com.packt.demo -DartifactId=spring-boot-thymeleaf -interactiveMode=false

```

上述命令将创建一个`spring-boot-thymeleaf`目录。这可以导入到 Eclipse IDE 中。

1.  您将打开`pom.xml`文件并添加一个`parent`项目：

```java
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>1.1.8.RELEASE</version>
</parent>
```

1.  开始向`pom.xml`文件添加一个依赖项：

```java
<dependencies>
  <dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-thymeleaf</artifactId>
  </dependency>
</dependencies>
```

1.  最后添加 Spring boot 插件：

```java
<build>
  <plugins>
  <plugin>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-maven-plugin</artifactId>
  </plugin>
  </plugins>
</build>
```

让我们开始修改 web。但等一下，这不是 web 应用程序！

1.  因此，让我们修改`App`类，使其成为 Spring Boot 应用程序的入口点：

```java
package com.packt.demo
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@EnableAutoConfiguration
@Configuration
@ComponentScan
public class App {
  public static void main(String[] args) {
    SpringApplication.run(App.class);
  }
}
```

1.  接下来，让我们配置 Thymeleaf 模板。为了配置它，我们需要在`src/main/resources/templates`目录下添加模板：

```java
<!DOCTYPE html>
<html>
<head>
  <title>Hello Spring Boot!</title>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
</head>
<body>
<p>Hello Spring Boot!</p>
</body>
<html>
```

1.  您可以通过添加 CSS 和 JavaScript 引用来升级 Thymeleaf 模板，如下所示：

```java
<!DOCTYPE html>
<html>
<head>
  <title>Hello Spring Boot!</title>
  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
  <link href="../static/css/core.css"
    th:href="@{/css/core.css}"
    rel="stylesheet" media="screen" />
</head>
<body>
<p>Hello Spring Boot!</p>
</body>
</html>
```

1.  Spring boot 支持开箱即用的 WebJars。将以下依赖项添加到`pom.xml`文件中。

```java
<dependency>
  <groupId>org.webjars</groupId>
  <artifactId>bootstrap</artifactId>
  <version>3.2.0</version>
</dependency>
<dependency>
  <groupId>org.webjars</groupId>
  <artifactId>jquery</artifactId>
  <version>2.1.1</version>
</dependency>
```

并在模板中引用库，如下所示：

```java
<link href="http://cdn.jsdelivr.net/webjars/bootstrap/3.2.0/css/bootstrap.min.css"
  th:href="@{/webjars/bootstrap/3.2.0/css/bootstrap.min.css}"
  rel="stylesheet" media="screen" />

<script src="img/jquery.min.js"
  th:src="img/jquery.min.js}"></script>
```

如您所见，对于静态原型设计，库是从 CDN 下载的，将打包从 JAR 转换为 WAR

使用 Spring boot 作为普通 web 应用程序运行这个项目非常容易。首先，我们需要将`pom.xml`中的打包类型从 JAR 改为 WAR（打包元素）。其次，确保 Tomcat 是一个提供的依赖项：

```java
<packaging>war</packaging>
```

我们还需要创建一个控制器来处理应用程序请求：

```java
package com.packt.demo;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
class HomeController {

  @RequestMapping("/")
  String index() {
    return "index";
  }
}
```

最后一步是引导一个 servlet 配置。创建一个`Init`类并继承自`SpringBootServletInitializer`：

```java
package packt.demo;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.context.web.SpringBootServletInitializer;

public class ServletInitializer extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
    return application.sources(App.class);
  }
}
```

我们可以使用`mvn clean package`命令检查配置是否与 Maven 一起工作。WAR 文件将被创建：

```java
Building war: C:\Projects\demos\spring-boot-thymeleaf\target\spring-boot-thymeleaf-1.0-SNAPSHOT.war

```

使用 Maven 直接从 WAR 文件启动应用程序，使用以下命令：

```java
java-jar target\spring-boot-thymeleaf-1.0-SNAPSHOT.war

```

创建 WAR 项目后，我们将在 Eclipse 中运行应用程序。在我们改变了打包方式后，Eclipse 将检测项目中的更改并向其添加 web facet。下一步是配置 Tomcat 服务器并运行它。导航到**Edit Configurations**，并添加带有解压的 WAR 构件的 Tomcat 服务器。现在你可以像运行其他 web 应用程序一样运行应用程序。

## 重新加载 Thymeleaf 模板

由于应用程序在 Eclipse 中运行在本地 Tomcat 服务器上，我们将重新加载静态资源（例如 CSS 文件）而无需重新启动服务器。但是，默认情况下，Thymeleaf 会缓存模板，因此为了更新 Thymeleaf 模板，我们需要改变这种行为。

+   将`application.properties`添加到`src/main/resources`目录中，其中包含`spring.thymeleaf.cache=false`属性

+   重新启动服务器，从现在开始您可以重新加载 Thymeleaf 模板而无需重新启动服务器

+   更改其他配置默认值

缓存配置并不是我们可以调整的唯一可用配置。请查看`ThymeleafAutoConfiguration`类，了解您可以更改的其他内容。举几个例子：`spring.thymeleaf.mode`，`spring.thymeleaf.encoding`。

## 使用 Thymeleaf 的 Spring 安全

由于我们使用了 Spring 安全，我们将在我们的 Spring 应用程序中使用 JSP 中的自定义登录表单。在本节中，让我们看看如何引入 Thymeleaf 模板来保护基于 Spring 的应用程序。

您可以像这样使用 Spring 安全方言来显示已登录用户的信息。属性`sec:authorize`在属性表达式评估为`True`时呈现其内容。您可以在成功认证后显示的基本文件中使用此代码：

```java
?
<div sec:authorize="hasRole('ROLE_ADMIN')">
  This content is only shown to administrators.
</div>
<div sec:authorize="hasRole('ROLE_USER')">
  This content is only shown to users.
</div>
  The attribute sec:authentication is used to print logged user name and roles:
?
  Logged user: <span sec:authentication="name">Bob</span>
  Roles: <span sec:authentication="principal.authorities">[ROLE_USER, ROLE_ADMIN]</span>
```

正如我们所知，以下是我们在 Spring 应用程序中添加 Spring 安全所执行的一些必要步骤。但是，您会注意到我们已经配置了一个 Thymeleaf 文件的 HTML 文件。

1.  配置 Spring 安全过滤器。

1.  将`applicationContext-springsecurity.xml`文件配置为上下文参数。

1.  在`applicationContext-springsecurity.xml`中配置需要保护的 URL。

1.  示例配置如下：

```java
<?
<http auto-config="true">
  <form-login login-page="/login.html" authentication-failure-url="/login-error.html" />
  <logout />
  ...
</http>
```

1.  配置 Spring 控制器：

```java
@Controller
public class MySpringController {

  ...

  // Login form
  @RequestMapping("/login.html")
  public String login() {
    return "login.html";
  }

  // Login form with error
  @RequestMapping("/login-error.html")
  public String loginError(Model model) {
    model.addAttribute("loginError", true);
    return "login.html";
  }
}
```

1.  让我们看一下`Login.html`文件，这是 Thymeleaf 文件。这可以通过文件开头给出的 XMLNS 来识别。还要注意，我们正在处理 JSP 文件中的错误；当登录失败时，它会显示错误消息。我们还将创建一个`error.html`文件来处理错误：

```java
<!DOCTYPE html>
<html  >
  <head>
  <title>Login page</title>
  </head>
  <body>
  <h1>Login page</h1>
  <p th:if="${loginError}">Wrong user or password</p>
  <form th:action="@{/j_spring_security_check}" method="post">
  <label for="j_username">Username</label>:
  <input type="text" id="j_username" name="j_username" /> <br />
  <label for="j_password">Password</label>:
  <input type="password" id="j_password" name="j_password" /> <br />
  <input type="submit" value="Log in" />
  </form>
  </body>
</html>

/*Error.html file*/
?
<!DOCTYPE html>
<html  >
  <head>
  <title>Error page</title>
  </head>
  <body>
  <h1 th:text="${errorCode}">500</h1>
  <p th:text="${errorMessage}">java.lang.NullPointerException</p>
  </body>
</html>
```

这一步是关于配置错误页面。错误页面可以在`web.xml`文件中配置。首先，我们需要向`web.xml`文件添加`<error-page>`标签。一旦配置了错误页面，我们需要通知控制器类有关错误页面的信息：

```java
<error-page>
  <exception-type>java.lang.Throwable</exception-type>
  <location>/error.html</location>
</error-page>
<error-page>
  <error-code>500</error-code>
  <location>/error.html</location>
</error-page>
```

1.  在控制器中为`error`页面添加请求映射：

```java
@RequestMapping("/error.html")
public String error(HttpServletRequest request, Model model) {
  model.addAttribute("errorCode", request.getAttribute("javax.servlet.error.status_code"));
  Throwable throwable = (Throwable) request.getAttribute("javax.servlet.error.exception");
  String errorMessage = null;
  if (throwable != null) {
    errorMessage = throwable.getMessage();
  }
  model.addAttribute("errorMessage", errorMessage);
  return "error.html";
  }
}
```

访问[`www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html`](http://www.thymeleaf.org/doc/tutorials/2.1/usingthymeleaf.html)了解更多详情。

# 摘要

在本章中，我们已经看到了如何将 Thymeleaf 模板引擎集成到 Spring MVC 应用程序中，以及如何使用 Spring boot 启动 Spring 与 Thymeleaf 应用程序。我们还演示了如何使用 Spring Thymeleaf 模板为 Spring 安全创建自定义表单。

在下一章中，我们将看到 Spring 与 Web 服务集成，并了解它为开发 SOAP 和 REST Web 服务提供了什么。
