## 第六章。探索 Spring MVC

到目前为止，我们已经讨论了如何使用 Spring 框架来处理、初始化和使用数据，同时将控制台作为我们的输出。我们还没有在呈现或用户交互方面付出任何努力。在当今世界，使用老式的基于窗口的、非常单调的呈现方式工作似乎非常无聊。我们希望有更有趣、更令人兴奋的东西。互联网是使世界比以往任何时候都更加紧密和有趣的“东西”。当今的世界是网络的世界，那么我们如何能与之脱节呢？让我们深入到一个令人惊叹的互联网世界，借助以下几点来探索 Spring 的强大功能：

+   为什么有必要学习使用 Spring 进行网络应用程序开发？

+   如何使用 Spring MVC 开发网络应用程序？

+   Spring MVC 的不同组件有哪些？

+   如何预填充表单并将数据绑定到对象？

+   我们还将讨论如何在 Spring 中执行验证。

在 20 世纪 90 年代，互联网为我们打开了一个完全新世界的大门。这是一个前所未有的数据海洋。在互联网之前，数据只能通过硬拷贝获得，主要是书籍和杂志。在早期，互联网只是用来分享静态数据，但随着时间的推移，互联网的维度、意义和用途发生了很大变化。如今，我们无法想象没有互联网的世界。这几乎是不可思议的。它已经成为我们日常生活中的一部分，也是我们业务行业的一个非常重要的来源。对于我们开发者来说，了解网络应用程序、其开发、挑战以及如何克服这些挑战也非常重要。

在 Java 中，可以使用 Servlet 和 JSP 创建基本网络应用程序，但随后发生了许多演变。这些演变主要是由于不断变化的世界对高需求的时间紧迫。不仅是呈现方式，而且整个网络体验也因 HTML5、CSS、JavaScript、AJAX、Jquery 等类似技术的使用而发生了变化。Servlet 处理网络请求并使用请求参数中的数据提取动态网络应用程序的数据。

在使用 Servlet 和 JSP 时，开发者必须付出很多努力来执行数据转换并将数据绑定到对象。除了执行业务逻辑的主要角色外，他们现在还必须处理额外的负担，即处理请求和响应呈现。

开发者主要在 web 应用程序中处理从请求提取的数据。他们根据规则开发复杂、较长的业务逻辑来执行任务。但如果从请求参数中提取的数据不正确，这一切都是徒劳的。这显然不是开发者的错，但他们的业务逻辑仍然受到影响，使用这样的数据值进行业务逻辑是没有意义的。开发者现在需要特别注意，在执行业务逻辑之前，首先要找出从请求中提取的数据是否正确。开发者还必须 extensively 参与数据呈现到响应中。首先，开发者需要将数据绑定到响应中，然后进一步如何在呈现方面提取它。

上述讨论的每一个任务都在有限的时间内给开发方增加了额外的负担。Spring 框架通过以下特性方便开发者进行简单和快速的开发：

+   Spring 框架支持 MVC 架构，实现了模型、视图和控制器的清晰分离。

+   该框架通过将请求参数绑定到命令对象，为开发者提供了豆子的力量，以便轻松处理数据。

+   它提供了对请求参数的简单验证，执行验证 either with Validator interface or using annotations. 它还可以支持自定义验证规则。

+   它提供了如@RequestParam、@RequestHeader 等注解，这些注解使请求数据绑定到方法参数而不涉及 Servlet API。

+   它支持广泛的视图模板，如 JSTL、Freemarker、Velocity 等。

+   通过使用 ModelMap 对象，使数据从控制器传输到视图变得容易。

+   它可以轻松地与其他框架集成，如 Apache Struts2.0、JSF 等。

通常，web 应用程序部署在 web 服务器上。应用程序中的每个资源都与 URL 映射，用户使用这些 URL 来访问资源。Servlet 或 JSP 从请求对象中读取数据，对其执行业务逻辑，然后将响应作为结果返回。我们都知道，在任何 web 应用程序中，都会发生这种一般的流程。在这个流程中，最重要的是这些 web 应用程序没有任何 Servlet 或控制器来管理整个应用程序的流程。是的，第一个到达者缺席了。整个应用程序及其流程必须由开发方维护。这就是 Servlet 和 Spring 之间的主要区别所在。

****

Spring 框架采用 MVC 设计模式，提供了一个前端控制器，处理或获取应用程序接收到的每个请求。以下图表显示了 Spring MVC 如何处理请求以及所有组件都是 Spring MVC 的一部分：

![](img/image_06_001.jpg)

以下步骤为我们提供了 Spring MVC 网络应用程序流程的方向：

+   每个传入请求首先会击中应用程序的心脏——前端控制器。前端控制器将请求分发到处理器，并允许开发者使用框架的不同功能。

+   前端控制器有自己的 WebApplicationContext，它是从根 WebApplicationContext 继承而来的。根上下文配置的 bean 可以在应用的上下文和 Servlet 实例之间访问和共享。类似于所有 Servlet，前端控制器在第一次请求时进行初始化。

+   一旦前端控制器初始化完成，它会寻找位于 WEB-INF 文件夹下名为`servlet_name-servlet.xml`的 XML 文件。该文件包含了 MVC 特定的组件。

+   这个配置文件默认命名为`XXX-servlet.xml`，位于 WEB-INF 文件夹下。这个文件包含了 URL 到可以处理传入请求的控制器的映射信息。在 Spring 2.5 之前，映射是发现处理器的必须步骤，现在我们不再需要。我们现在可以直接使用基于注解的控制器。

+   `RequestMappingHandlerMapping`会搜索所有控制器，查找带有`@RequestMapping`注解的@Controller。这些处理器可以用来自定义 URL 的搜索方式，通过自定义拦截器、默认处理器、顺序、总是使用完整路径、URL 解码等属性。

+   在扫描所有用户定义的控制器之后，会根据 URL 映射选择合适的控制器并调用相应的方法。方法的选择是基于 URL 映射和它支持的 HTTP 方法进行的。

+   在执行控制器方法中编写的业务逻辑后，现在是生成响应的时候了。这与我们通常的 HTTPResponse 不同，它不会直接提供给用户。相反，响应将被发送到前端控制器。在这里，响应包含视图的逻辑名称、模型数据的逻辑名称和实际的数据绑定。通常，`ModelAndView`实例会被返回给前端控制器。

+   逻辑视图名在前端控制器中，但它不提供有关实际视图页面的任何信息。在`XXX-servlet.xml`文件中配置的`ViewResolver`bean 将作为中介，将视图名称映射到实际页面。框架支持广泛的视图解析器，我们将在稍后讨论它们。

+   视图解析器帮助获取前端控制器可以作为响应返回的实际视图。前端控制器通过从绑定的模型数据中提取值来渲染它，然后将其返回给用户。

在我们讨论的流程中，我们使用了诸如前端控制器、ModelAndView、ViewResolver、ModelMap 等许多名称。让我们深入讨论这些类。

### 分发器 Servlet

`DispacherServlet`在 Spring MVC 应用程序中充当前端控制器，首先接收每个传入请求。它基本上用于处理 HTTP 请求，因为它从`HTTPServlet`继承而来。它将请求委托给控制器，解决要作为响应返回的视图。以下配置显示了在`web.xml`（部署描述符）中的调度器映射：

```java
<servlet>
  <servlet-name>books</servlet-name>
    <servlet-class>
      org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>books</servlet-name>
  <url-pattern>*.htm</url-pattern>
</servlet-mapping>
```

上述配置说明所有以`.htm`为 URL 模式的请求将由名为`books`的 Servlet 处理。

有时应用程序需要多个配置文件，其中一些位于根`WebApplicationContext`中，处理数据库的 bean，一些位于 Servlet 应用程序上下文中，包含在控制器中使用的 bean。以下配置可用于初始化来自多个`WebApplicationContext`的 bean。以下配置可用于从上下文中加载多个配置文件，例如：

```java
<servlet>
  <servlet-name>books</servlet-name>
    <servlet-class>
      org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>books</servlet-name>
  <url-pattern>*.htm</url-pattern>
</servlet-mapping>
```

### 控制器

Spring 控制器用于处理执行业务逻辑的请求，这些控制器也可以被称为'处理器'，其方法称为处理器方法。Spring 提供了`AbstarctUrlViewController`、`ParameterizableViewContoller`、`ServletForwardingConroller`、`ServletWrappingControllerBefore`作为控制器。在 Spring 2.5 基于 web 的应用程序中，需要对这些控制器进行子类化以自定义控制器。但现在，Spring 通过`@Controller`注解支持注解驱动的控制器。以下配置启用了基于注解的控制器：

```java
<mvc:annotation-driven />
```

需要发现基于注解的控制器以执行处理器方法。以下配置提供了关于框架应扫描哪些包以发现控制器的信息：

```java
<context:component-scan base-package="com.packt.*">
</context:component-scan>
```

`@RequestMapping`注解用于标注类或方法，以声明它能处理的特定 URL。有时同一个 URL 可以注解多个方法，这些方法支持不同的 HTTP 方法。`@RequestMapping`的'method=RequestMethod.GET'属性用于指定哪个 HTTP 方法将由该方法处理。

### `ModelAndView`

`ModelAndView`在生成响应中扮演着重要角色。`ModelAndView`实例使得可以将模型数据绑定到其逻辑名称、逻辑视图名称。在视图中使用的数据对象通常称为模型数据。以下代码段清楚地说明了绑定是如何发生的：

```java
new ModelAndView(logical_name_of_view,logical_name_of_model_data,
  actual_value_of_model_data);
```

我们甚至可以使用以下代码段：

```java
ModelAndView mv=new ModelAndView();
mv.setViewName("name_of_the_view");
mv.setAttribute(object_to_add);
```

### `ModelMap`

`ModelMap`接口是`LinkedHashMap`的子类，在构建使用键值对的模型数据时使用。它有`addAttribute()`方法，提供模型和模型逻辑名称的绑定。在`ModelMap`中设置的属性可以在表单提交时由视图用于表单数据绑定。我们稍后会深入讨论这一点。

### 视图解析器

用户定义的控制器返回的逻辑视图名称和其他详细信息。视图名称是一个需要由 ViewResolver 解析的字符串。

以下是一些可以用于渲染视图的 ViewResolvers：

+   **XmlViewResolver**：XmlViewResolver 用于查看编写为 XML 的文件。它使用位于 WEB-INF/views.xml 的默认配置，该配置文件包含与 Spring beans 配置文件相同的 DTD 的视图 bean。配置可以如下所示编写：

```java
      <bean id="myHome"  
        class="org.springframework.web.servlet.view.JstlView"> 
        <property name="url" value="WEB-INF/jsp/home.jsp"/> 
      <bean> 

```

+   逻辑视图名 '`myHome`' 被映射到实际的视图 '`WEB-INF/jsp/home.jsp`'。

+   一个 bean 也可以引用映射到另一个 bean 的视图，如下所示：

```java
      <bean id="logout"  
        class="org.springframework.web.servlet.view.RenderView"> 
        <property name="url" value="myHome"/> 
      <bean> 

```

+   `'logout'` bean 没有映射到任何实际的视图文件，但它使用 '`myHome'` bean 来提供实际的视图文件。

+   **UrlBasedViewResolver:** 它将 URL 直接映射到逻辑视图名称。当逻辑名称与视图资源相匹配时，它将被优先考虑。它的前缀和后缀作为其属性，有助于获取带有其位置的实际视图名称。该类无法解析基于当前区域设置的视图。为了启用 URLBasedViewResolver，可以编写以下配置：

```java
      <bean id="viewResolver" 
  class="org.springframework.web.servlet.view.UrlBasedViewResolver"> 
        <property name="viewClass" value=   
          "org.springframework.web.servlet.view.JstlView"/> 
        <property name="prefix" value="WEB-INF/jsp/"/> 
        <property name="suffix" value=".jsp"/> 
      <bean> 

```

+   `JstlView` 用于渲染视图页面。在我们的案例中，页面名称和位置是 'prefix+ view_name_from_controller+suffix'。

+   **InternalResourceViewResolver:** InternalResourceViewresolver 是 UrlBasedViewResolver 的子类，用于解析内部资源，这些资源可以作为视图使用，类似于其父类的前缀和后缀属性。AlwaysInclude、ExposeContextBeansAsAttributes、ExposedContextBeanNames 是该类的几个额外属性，使其比父类更频繁地使用。以下配置与之前示例中配置 UrlBasedViewResolver 的方式类似：

```java
      <bean id="viewResolver" class=  
  "org.springframework.web.servlet.view.InternalResourceViewResolver"> 
        <property name="viewClass" value=                
          "org.springframework.web.servlet.view.JstlView"/> 
        <property name="prefix" value="WEB-INF/jsp/"/> 
        <property name="suffix" value=".jsp"/> 
      <bean> 

```

+   它只能在到达页面时验证页面的存在，在此之前不会进行验证。

+   **ResourceBundleViewResolver:** ResourceBundleViewResolver 使用配置中指定的 ResourceBundle 的定义。默认文件用于定义配置的是 views.properties。配置将如下所示：

```java
      <bean id="viewResolver" class= 
  "org.springframework.web.servlet.view.ResourceViewResolver"> 
        <property name="base" value="web_view"/> 
      </bean> 

```

+   视图.properties 将指定要使用的视图类的详细信息以及实际视图的 URL 映射，如下所示：

```java
      home.(class)= org.springframework.wev.servlet.view.JstlView 

```

+   下面的行指定了名为 homepage 的视图的映射：

```java
       homepage.url= /WEB-INF/page/welcome.jsp 

```

+   **TilesViewResolver:** Tiles 框架用于定义可以重用并保持应用程序一致的外观和感觉的页面布局模板。在 'tiles.def' 文件中定义的页面定义作为 tile、header、footer、menus，在运行时组装到页面中。控制器返回的逻辑名称与视图解析器将渲染的 tiles 模板名称匹配。

除了上面讨论的视图解析器之外，Spring 还具有 FreeMarkerViewResolver、TileViewResolver、VelocityLayoutViewResolver、VelocityViewResolver、XsltViewResolver。

在继续讨论之前，让我们首先开发一个示例演示，以详细了解应用程序的流程，并通过以下步骤了解上述讨论的方向：

1.  创建一个名为 Ch06_Demo_SpringMVC 的动态网页应用程序。

1.  按照以下项目结构复制 spring-core、spring-context、commons-logging、spring-web 和 spring-webmvc 的 jar 文件：

![](img/image_06_002.png)

1.  在**`WebContent`**文件夹中创建 index.jsp，作为主页。可以根据你的需求自定义名称，就像我们在任何 Servlet 应用程序中所做的那样。

1.  在`index.jsp`中添加一个链接，该链接提供导航到控制器，如下所示：

```java
      <center> 
        <img alt="bookshelf" src="img/img1.png" height="180" 
          width="350"> 
        <br> 
      </center> 
      <a href="welcomeController.htm">Show welcome message</a> 

```

1.  每当用户点击链接时，系统会生成一个带有'welcomeCointroller.htm' URL 的请求，该请求将由前端控制器处理。

1.  是时候在`web.xml`中配置前端控制器了：

```java
      <servlet> 
        <servlet-name>books</servlet-name> 
        <servlet-class>    
          org.springframework.web.servlet.DispatcherServlet 
        </servlet-class> 
      </servlet> 
      <servlet-mapping> 
        <servlet-name>books</servlet-name> 
        <url-pattern>*.htm</url-pattern> 
      </servlet-mapping> 

```

1.  前端控制器查找 WEB-INF 中名为`servlet_name-servlet.xml`的文件来查找和调用控制器的的方法。在我们的案例中，Servlet 的名称是'`books`'。所以让我们在 WEB-INF 文件夹下创建一个名为'`books-servlet.xml`'的文件。

1.  该文件应包含 Spring 容器将扫描以查找控制器的包的配置。配置将如下所示：

```java
      <context:component-scan base-package=     
        "com.packt.*"></context:component-scan> 

```

上述配置说明将扫描'`com.packt`'包中的所有控制器。

1.  在 com.packt.ch06.controllers 包中创建一个`MyMVCController`类。

1.  通过`@Controller`注解类。注解类使其能够使用处理请求的功能。

1.  让我们通过如下所示的`@RequestMapping`注解添加`welome()`方法来处理请求：

```java
      @Controller 
      public class MyMVCController { 
        @RequestMapping(value="welcomeController.htm") 
        public ModelAndView welcome() 
        { 
          String welcome_message="Welcome to the wonderful  
          world of Books"; 
          return new ModelAndView("welcome","message",welcome_message); 
        } 
      } 

```

控制器可以有多个方法，这些方法将根据 URL 映射被调用。在这里，我们声明了将被`welcomeController.htm'` URL 调用的方法。

该方法通过`ModelAndView`生成欢迎信息并生成响应，如下所示：

```java
      new ModelAndView("welcome","message",welcome_message); 
      The ModelAndView instance is specifying, 
      Logical name of the view -  welcome 
      Logical name of the model data -  message  
      Actual value of the model data - welcome_message 

```

以上代码的替代方案，你可以使用如下代码：

```java
      ModelAndView mv=new ModelAndView(); 
      mv.setViewName("welcome"); 
      mv.addObject("message", welcome_message); 
      return mv; 

```

我们可以将多个方法映射到相同的 URL，支持不同的 HTTP 方法，如下所示：

```java
      @RequestMapping(value="welcomeController.htm", 
        method=RequestMethod.GET) 
      public ModelAndView welcome(){ 
        //business logic  
      } 
      @RequestMapping(value="welcomeController.htm", 
        method=RequestMethod.POST) 
      public ModelAndView welcome_new()  { 
        //business logic 
      } 

```

1.  如以下所示，在`books-servlet.xml`中配置`ViewResolver` bean：

```java
      <bean id="viewResolver" class=
   "org.springframework.web.servlet.view.InternalResourceViewResolver"> 
        <property name="prefix" value="/WEB-INF/jsps/"></property> 
        <property name="suffix" value=".jsp"></property> 
      </bean> 

```

`ViewResolver`帮助前端控制器获取实际的视图名称和位置。在前端控制器返回给浏览器的响应页面中，在我们的案例中将是：

![](img/image_06_003.png)

1.  在 WebContent 中创建一个名为 jsps 的文件夹。

1.  在 jsps 文件夹中创建一个 welcome.jsp 页面，使用表达式语言显示欢迎信息：

```java
      <body> 
        <center> 
          <img alt="bookshelf" src="img/img1.png" height="180" 
            width="350"> 
          <br> 
        </center> 
        <center> 
          <font color="green" size="12"> ${message } </font> 
        </center> 
      </body>
```

在 EL 中使用属性'`message'`，因为这是我们控制器方法中用于`ModelAndView`对象逻辑模型名称。

1.  配置好 tomcat 服务器并运行应用程序。在浏览器中将显示链接。点击链接我们将看到如下截图的输出：

![](img/image_06_004.png)

该演示向我们介绍了 Spring MVC 流程。现在让我们逐步开发书籍应用程序，涵盖以下案例：

+   读取请求参数

+   处理表单提交

### 案例 1：读取请求参数

让我们开始通过以下步骤读取请求参数：

1.  创建 ReadMyBooks 作为动态网络应用程序，并像我们之前那样添加所有必需的 jar 文件。

1.  每个应用程序都有一个主页。所以，让我们将之前的应用程序中的 index.jsp 作为主页添加进去。您可以直接复制和粘贴。

1.  从之前的应用程序中复制 images 文件夹。

1.  在下面所示的位置添加一个链接，用于搜索按作者姓名查找书籍，

```java
      <a href="searchByAuthor.jsp">Search Book By Author</a> 

```

1.  让我们添加一个名为 searchByAuthor.jsp 的页面，使用户可以输入作者姓名来请求书籍列表，如下所示：

```java
      <body> 
        <center> 
          <img alt="bookshelf" src="img/img1.png" height="180"  
            width="350"> 
          <br> 

          <h3>Add the Author name to search the book List</h3> 

          <form action="/searchBooks.htm"> 
            Author Name:<input type="text" name="author"> 
            <br>  
            <input  type="submit" value="Search Books"> 
          </form> 
        </center> 
      </body>
```

1.  如我们之前所做的那样，在 web.xml 中为 DispachetServlet 作为前端控制器添加配置，并将 servlet 命名为'books'。

1.  创建或复制 books-servlet.xml，用于从早期应用程序配置处理映射和其他网络组件映射。

1.  使用'context'命名空间添加扫描控制器的配置。

1.  我们需要 Book bean 来处理数据往返于控制器。因此，在开发控制器代码之前，请将 Book.java 添加到我们之前应用的 com.packt.ch06.beans 包中，数据成员如下所示：

```java
      public class Book { 
        private String bookName; 
        private long ISBN; 
        private String publication; 
        private int price; 
        private String description; 
        private String author; 
        //getters and setters 
      } 

```

1.  现在在 com.packt.ch06.controllers 包中创建一个名为 SearchBookController 的类作为控制器，并用@Controller 注解它。

1.  为了搜索书籍，需要添加一个名为 searchBookByAuthor()的方法，并用@RequestMapping 注解为'searchBooks.htm'的 URL。我们可以使用 Servlet API 或 Spring API，但在这里我们将使用 Spring API。

1.  现在让我们为`searchBookByAuthor()`添加以下代码：

+   阅读请求参数

+   搜索书籍列表

1.  创建 ModelAndView 实例以将书籍列表作为模型数据，逻辑模型名称和逻辑视图名称一起绑定。

代码将如下所示：

```java
      @Controller 
      public class SearchBookController { 
        @RequestMapping(value = "searchBooks.htm") 
        public ModelAndView searchBookByAuthor( 
          @RequestParam("author") String author_name)  
        { 
          // the elements to list generated below will be added by      
          business logic  
          List<Book> books = new ArrayList<Book>(); 
          books.add(new Book("Learning Modular Java Programming",  
            9781235, "packt pub publication", 800, 
            "Explore the Power of Modular Programming ",  
            "T.M.Jog")); 
          books.add(new Book("Learning Modular Java Programming",  
            9781235, "packt pub publication", 800, 
            "Explore the Power of Modular Programming ",   
            "T.M.Jog")); 
          mv.addObject("auth_name",author); 
          return new ModelAndView("display", "book_list", books); 
        } 
      } 

```

`@RequestParam`用于读取请求参数并将它绑定到方法参数。'author'属性的值被绑定到 author_name 参数，而不会暴露 servlet API。

在这里，我们添加了一个虚拟列表。稍后，可以将其替换为从持久层获取数据的实际代码。

1.  是时候在 books-servlet.xml 中配置视图解析器和包扫描，就像我们之前在早期应用程序中做的那样。我们可以将 books-servlet.xml 从早期应用程序的 WEB-INF 中复制粘贴过来。

1.  在 WebContent 下创建 jsps 文件夹，该文件夹将包含 jsp 页面。

1.  在 jsps 文件夹中创建 display.jsp，使用 JSTL 标签显示书籍列表，如下所示：

```java
      <%@ taglib prefix="jstl"  
        uri="http://java.sun.com/jsp/jstl/core"%> 
      <html> 
        <head> 
          <meta http-equiv="Content-Type" content="text/html; 
            charset=ISO-8859-1"> 
          <title>Book List</title> 
        </head> 
        <body> 
          <center> 
            <img alt="bookshelf" src="img/img1.png"   
              height="180" width="350"> 
            <br> 
          </center> 
          <jstl:if test="${not empty book_list }"> 
            <h1 align="center"> 
              <font color="green"> 
                Book List of ${auth_name } 
              </font> 
            </h1> 
            <table align="center" border="1"> 
            <tr> 
              <th>Book Name</th> 
              <th>ISBN</th> 
              <th>Publication</th> 
              <th>Price</th> 
              <th>Description</th> 
            </tr> 
            <jstl:forEach var="book_data"  
              items="${book_list}" varStatus="st"> 
              <tr> 
                <td> 
                  <jstl:out value="${ book_data.bookName }"> 
                  </jstl:out> 
                </td> 
                <td> 
                  <jstl:out value="${ book_data.ISBN }"> 
                  </jstl:out> 
                </td> 
                <td> 
                  <jstl:out value="${ book_data.publication }"> 
                  </jstl:out> 
                </td> 
                <td> 
                  <jstl:out value="${ book_data.price }"> 
                  </jstl:out></td> 
                <td> 
                  <jstl:out value="${ book_data.description }"> 
                  </jstl:out> 
                </td> 
              </tr> 
            </jstl:forEach> 
          </table> 
        </jstl:if> 
        <jstl:if test="${empty book_list }"> 
          <jstl:out value="No Book Found"></jstl:out> 
        </jstl:if> 
      </body>
```

如果列表没有元素，就没有显示该列表的必要。jstl:if 标签用于决定是否显示列表，而 jstl:forEach 标签用于通过迭代列表显示书籍信息。

1.  运行应用程序，点击主页上的链接以加载表单以输入作者名称。如果作者名称存在，则在表单提交时我们将获得以下书籍列表：

![](img/image_06_005.png)

这里，我们使用了`@RequestParam`将个别请求参数绑定到方法参数。但是，如果请求参数的名称与方法参数的名称匹配，则无需使用注解。更新后的代码可以如下所示：

```java
@RequestMapping(value = "searchBooks.htm") 
public ModelAndView searchBookByAuthor( String author) { 
  List<Book> books = new ArrayList<Book>(); 
  books.add(new Book("Learning Modular Java Programming",  
    9781235, "packt pub publication", 800, 
    "explore the power of modular Programming ",    
    author)); 
  books.add(new Book("Learning Modular Java Programming",  
    9781235, "packt pub publication", 800, 
    "explore the power of modular Programming ",  
    author)); 
  ModelAndView mv= 
    new ModelAndView("display", "book_list", books); 
    mv.addObject("auth_name",author); 
    return mv; 
} 

```

逐一读取个别请求参数，然后将它们绑定到 bean 对象，变得繁琐而不必要冗长。框架通过处理“表单后盾对象”提供了更好的选项。

### 情况 2：处理表单提交

表单提交是应用程序开发中非常常见的任务。每次表单提交时，开发者都需要执行以下步骤：

1.  读取请求参数

1.  将请求参数值转换为所需数据类型

1.  将值设置到 bean 对象中。

上述步骤可以省略，直接在表单提交时获取 bean 实例。我们将讨论两种情况的表单处理：

+   表单提交

+   表单预处理

#### 表单提交

在普通的网络应用程序中，用户点击一个链接后，表单会被加载，然后手动执行上述讨论的步骤。由于需要自动化这个过程，而不是直接显示表单，因此应该从控制器加载表单，而该控制器已经有一个 bean 实例。在表单提交时，用户输入的值会被绑定到这个实例。现在，这个实例可以在控制器中用于执行业务逻辑。从 Spring 2.0 开始，提供了一组标签，这些标签在视图中处理表单绑定，从而使开发变得容易。

让我们在 ReadMyBooks 应用程序中添加一个表单，以了解使用 Spring 提供的表单标签进行表单提交。我们将分两步进行，第一步显示表单，第二步处理提交的表单。

##### 显示表单

由于表单必须从控制器加载，让我们按照以下步骤添加代码，

1.  在主页上添加一个链接以加载表单。获取表单的代码如下所示：

```java
      <a href="showBookForm.htm">Show Form to add new Book</a> 

```

1.  在`AddBookController`中添加`showBookForm()`方法，该方法将在步骤 1 中点击的链接上被调用。该方法将返回一个表单页面，使用 Book 对象，其中输入的数据将被绑定。该方法的代码如下，

```java
      @RequestMapping("/showBookForm.htm") 
      public ModelAndView showBookForm(ModelMap map)  
      throws Exception { 
        Book book=new Book(); 
        map.addAttribute(book); 
        return new ModelAndView("bookForm"); 
      } 

```

该方法应该有一个`ModelMap`作为其参数之一，以添加一个 bean 实例，该实例可以被视图使用。在这里，我们添加了'book'属性，其值为 book 实例。默认情况下，引用名将被用作属性名。'book'实例也可以被称为“表单后盾”对象。为了自定义在视图中使用的表单后盾对象的名称，我们可以使用以下代码：

```java
      map.addAttribute("myBook",book); 

```

1.  因为视图名称'`bookForm'`由控制器返回，所以在 jsps 文件夹中添加`bookForm.jsp`，该文件包含显示表单的表单。

1.  用户输入的值需要绑定到表单。Spring 框架提供了强大的标签来处理用户输入。为了使 Spring 标签生效，我们需要添加如下所示的'taglib'指令：

```java
      <%@ taglib prefix="form"  
        uri="http://www.springframework.org/tags/form"%> 

```

1.  Spring 提供了与 html 类似的标签来处理表单、输入、复选框、按钮等，主要区别在于它们的值隐式绑定到 bean 数据成员。以下代码将允许用户输入书籍名称，并在表单提交时将其绑定到 Book bean 的'bookName'数据成员：

```java
      <form:input path="bookName" size="30" /> 

```

'path'属性将输入值映射到 bean 数据成员。值必须按照数据成员的名称指定。

1.  让我们在 bookForm.jsp 中添加以下表单，以便用户输入新书籍的值：

```java
      <form:form modelAttribute="book" method="POST"  
        action="addBook.htm"> 
        <h2> 
          <center>Enter the Book Details</center> 
        </h2> 

        <table width="100%" height="150" align="center" border="0"> 
         <tr> 
           <td width="50%" align="right">Name of the Book</td> 
           <td width="50%" align="left"> 
             <form:input path="bookName" size="30" /> 
           </td> 
         </tr> 
         <tr> 
           <td width="50%" align="right">ISBN number</td> 
           <td width="50%" align="left"> 
             <form:input path="ISBN" size="30" /> 
           </td> 
         </tr> 
         <tr> 
           <td width="50%" align="right">Name of the Author</td> 
           <td width="50%" align="left"> 
             <form:input path="author" size="30" /> 
           </td> 
         </tr> 
         <tr> 
           <td width="50%" align="right">Price of the Book</td> 
           <td width="50%" align="left"> 
             <form:select path="price"> 
               <!- 
                 We will add the code to have  
                 predefined values here  
               -->             
             </form:select> 
           </td> 
         </tr> 
         <tr> 
           <td width="50%" align="right">Description of the  
             Book</td> 
           <td width="50%" align="left"> 
             <form:input path="description"  size="30" /> 
           </td> 
         </tr> 
         <tr> 
           <td width="50%" align="right">Publication of the  
             Book</td> 
           <td width="50%" align="left"> 
             <form:input path="publication"  size="30" /> 
           </td> 
         </tr> 
         <tr> 
           <td colspan="2" align="center"><input type="submit"  
             value="Add Book"></td> 
          </tr> 
        </table> 
      </form:form>
```

属性'modelAttribute'接收由控制器设置的 ModelMap 逻辑属性的值。

1.  运行应用程序并点击'**`Show Form to add new book`**'。

1.  您将被导航到 bookForm.jsp 页面，在那里您可以输入自己的值。提交后，您将得到 404 错误，因为没有资源被我们编写来处理请求。别担心！！在接下来的步骤中我们将处理表单。

##### 表单后处理

1.  让我们在 AddController 中添加一个方法，该方法将在表单提交时通过 url 'addBook.htm'调用，如下所示：

```java
      @RequestMapping("/addBook.htm") 
      public ModelAndView addBook(@ModelAttribute("book") Book book) 
      throws Exception { 
          ModelAndView modelAndView = new ModelAndView(); 
          modelAndView.setViewName("display"); 
          //later on the list will be fetched from the table 
          List<Book>books=new ArrayList(); 
          books.add(book); 
          modelAndView.addObject("book_list",books); 
          return modelAndView; 
      } 

```

当用户提交表单时，他输入的值将被绑定到 bean 数据成员，生成一个 Book bean 的实例。通过@ModelAttribute 注解'book'参数使开发者可以使用绑定值的 bean 实例。现在，无需读取单个参数，进一步获取和设置 Book 实例。

因为我们已经有了 display.jsp 页面来显示书籍，所以我们在这里只是重用它。用户输入的书籍详情稍后可以添加到书籍表中。

1.  运行应用程序，点击链接获取表单。填写表单并提交以获得以下输出：

![](img/image_06_006.png)

输出列表显示了书籍的详细信息，但没有价格，因为价格目前没有设置。我们想要一个带有预定义值的价格列表。让我们继续讨论表单的预处理。

#### 表单预处理

在某些情况下，表单包含一些预定义值，如国家名称或书籍类别的下拉菜单、可供选择的颜色的单选按钮等。这些值可以硬编码，导致频繁更改要显示的值。相反，可以使用常量值，值可以被渲染并在表单中填充。这通常称为表单预处理。预处理可以在两个步骤中完成。

##### 定义要在视图中添加的属性的值

`@ModelAttribute`用于将模型数据的实例添加到 Model 实例中。每个用@ModelAttribute 注解的方法在其他 Controller 方法之前和执行时都会被调用，并在执行时将模型数据添加到 Spring 模型中。使用该注解的语法如下：

```java
@ModelAttribute("name_of_the_attribute") 
access_specifier return_type name_of_method(argument_list) {  // code   } 

```

以下代码添加了一个名为'hobbies'的属性，该属性可在视图中使用：

```java
@ModelAttribute("hobbies") 
public List<Hobby> addAttribute() { 
  List<Hobby> hobbies=new ArrayList<Hobby>(); 
  hobbies.add(new Hobby("reading",1)); 
  hobbies.add(new Hobby("swimming",2)); 
  hobbies.add(new Hobby("dancing",3)); 
  hobbies.add(new Hobby("paining",4)); 
  return hobbies; 
} 

```

Hobby 是一个用户定义的类，其中包含 hobbyName 和 hobbyId 作为数据成员。

##### 在表单中填充属性的值

表单可以使用复选框、下拉菜单或单选按钮向用户显示可用的选项列表。视图中的值可以使用列表、映射或数组为下拉菜单、复选框或单选按钮的值。

标签的一般语法如下所示：

```java
<form:name-of_tag path="name_of_data_memeber_of_bean"  
  items="name_of_attribute" itemLable="value_to display"  
  itemValue="value_it_holds"> 

```

以下代码可用于使用'hobbies'作为模型属性绑定值到 bean 的 hobby 数据成员，在复选框中显示用户的爱好：

```java
<form:checkboxes path="hobby" items="${hobbies}"    
  itemLabel="hobbyName" itemValue="hobbyId"/>                 

```

同样，我们可以在运行时为选择标签生成下拉菜单和选项。

### 注意

当处理字符串值时，可以省略`itemLabel`和`itemValue`属性。

完整的示例可以参考应用程序`Ch06_Form_PrePopulation`。

让我们更新`ReadMyBooks`应用程序，在`bookForm.jsp`中预定义一些价格值，并使用'`ModelAttribute`'讨论以下步骤中的表单预处理：

1.  因为表单是由`AddController`返回到前端控制器，我们想在其中设置预定义的值，因此在`addPrices()`方法中添加注解。如下所示使用`@ModelAttribute`注解：

```java
      @ModelAttribute("priceList") 
      public List<Integer> addPrices() { 
        List<Integer> prices=new ArrayList<Integer>(); 
        prices.add(300); 
        prices.add(350); 
        prices.add(400); 
        prices.add(500); 
        prices.add(550); 
        prices.add(600); 
        return prices; 
      } 

```

上述代码创建了一个名为'`pricelist`'的属性，该属性可用于视图。

1.  现在，`pricelist`属性可以在视图中显示预定义的值。在我们这个案例中，是一个用于添加新书籍的表单，更新`bookForm.jsp`以显示如下所示的价格列表：

```java
      <form:select path="price"> 
        <form:options items="${priceList}" />   
      </form:select>
```

1.  运行应用程序并点击链接，您可以观察到预定义的价格将出现在下拉列表中，如下所示：

![](img/image_06_007.png)

用户将在表单中输入值并提交它。

值可以在处理程序方法中获取。但是，我们仍然不能确定只有有效值会被输入并提交。在错误值上执行的业务逻辑总是会失败。此外，用户可能会输入错误数据类型值，导致异常。让我们以电子邮件地址为例。电子邮件地址总是遵循特定的格式，如果格式错误，业务逻辑最终会失败。无论什么情况，我们必须确信只提交有效值，无论是它们的数据类型、范围还是形成。验证正确数据是否会被提交的过程是“表单验证”。表单验证在确保正确数据提交方面起着关键作用。表单验证可以在客户端和服务器端进行。Java Script 用于执行客户端验证，但它可以被禁用。在这种情况下，服务器端验证总是更受欢迎。

****

Spring 具有灵活的验证机制，可以根据应用程序要求扩展以编写自定义验证器。Spring MVC 框架默认支持在应用程序中添加 JSR303 实现依赖项时的 JSR 303 规范。以下两种方法可用于在 Spring MVC 中验证表单字段，

+   基于 JSR 303 规范的验证

+   基于 Spring 的实现，使用 Validator 接口。

### 基于 Spring Validator 接口的自定义验证器

Spring 提供了 Validator 接口，该接口有一个 validate 方法，在该方法中会检查验证规则。该接口不仅支持 web 层的验证，也可以在任何层使用以验证数据。如果验证规则失败，用户必须通过显示适当的信息性消息来了解这一点。BindingResult 是 Errors 的子类，在执行 validate()方法对模型进行验证时，它持有由 Errors 绑定的验证结果。错误的可绑定消息将使用<form:errors>标签在视图中显示，以使用户了解它们。

让我们通过以下步骤在我们的 ReadMyBooks 应用程序中添加一个自定义验证器：

1.  在应用程序的 lib 文件夹中添加 validation-api-1.1.0.final.api.jar 文件。

1.  在 com.packt.ch06.validators 包中创建 BookValidator 类。

1.  类实现了 org.springframework.validation.Validator 接口。

1.  如代码所示，重写 supports()方法，

```java
      public class BookValidator implements Validator { 
        public boolean supports(Class<?> book_class) { 
          return book_class.equals(Book.class); 
        } 
      } 

```

支持方法确保对象与 validate 方法验证的对象匹配

1.  现在重写 validate()方法，根据规则检查数据成员。我们将分三步进行：

    1.  设置验证规则

我们将核对以下规则：

+   书籍名称的长度必须大于 5。

+   作者的名字必须不为空。

+   描述必须不为空。

+   描述的长度必须至少为 10 个字符，最多为 40 个字符。

+   国际标准书号（ISBN）不应该少于 150。

+   价格不应该少于 0。

+   出版物必须不为空。

    1.  编写条件以检查验证规则。

    1.  如果验证失败，使用`rejectValue()`方法将消息添加到`errors`实例中

使用上述步骤的方法可以如下所示编写：

```java
      public void validate(Object obj, Errors errors) { 
        // TODO Auto-generated method stub 
        Book book=(Book) obj; 
        if (book.getBookName().length() < 5) { 
          errors.rejectValue("bookName", "bookName.required", 
          "Please Enter the book Name"); 
        } 
        if (book.getAuthor().length() <=0) { 
          errors.rejectValue("author", "authorName.required", 
          "Please Enter Author's Name"); 
        } 
        if (book.getDescription().length() <= 0) 
        { 
          errors.rejectValue("description",  
            "description.required", 
            "Please enter book description"); 
        } 
        else if (book.getDescription().length() < 10 ||  
          book.getDescription().length() <  40) { 
            errors.rejectValue("description", "description.length", 
            Please enter description within 40 charaters only"); 
         } 
         if (book.getISBN()<=150l) { 
           errors.rejectValue("ISBN", "ISBN.required", 
           "Please Enter Correct ISBN number"); 
         }   
         if (book.getPrice()<=0 ) { 
           errors.rejectValue("price", "price.incorrect",  "Please  
           enter a Correct correct price"); 
         } 
        if (book.getPublication().length() <=0) { 
          errors.rejectValue("publication",  
            "publication.required", 
            "Please enter publication "); 
        } 
      } 

```

`Errors`接口用于存储有关数据验证的绑定信息。`errors.rejectValue()`是它提供的一个非常有用的方法，它为对象及其错误消息注册错误。以下是来自`Error`接口的`rejectValue()`方法的可用签名，

```java
      void rejectValue(String field_name, String error_code); 
      void rejectValue(String field_name, String error_code, String  
        default_message); 
      void rejectValue(String field_name, String error_code, 
        Object[] error_Args,String default_message); 

```

1.  在`AddBookController`中添加一个类型为`org.springframework.validation.Validator`的数据成员，并用`@Autowired`注解进行注释，如下所示：

```java
      @Autowired 
      Validator validator; 

```

1.  更新`AddController`的`addBook()`方法以调用验证方法并检查是否发生了验证错误。更新后的代码如下所示：

```java
      public ModelAndView addBook(@ModelAttribute("book") Book book,   
        BindingResult bindingResult)   throws Exception { 
        validator.validate(book, bindingResult); 
      if(bindingResult.hasErrors()) 
      { 
        return new ModelAndView("bookForm"); 
      } 
      ModelAndView modelAndView = new ModelAndView(); 
      modelAndView.setViewName("display"); 
      //later on the list will be fetched from the table 
      List<Book>books=new ArrayList(); 
      books.add(book); 
      modelAndView.addObject("book_list",books); 
      modelAndView.addObject("auth_name",book.getAuthor()); 
      return modelAndView; 
    } 

```

`addBook()`方法的签名应该有一个`BindingResult`作为其参数之一。`BindingResult`实例包含在执行验证时发生错误的消息列表。`hasErrors()`方法在数据成员上验证失败时返回 true。如果`hasErrors()`返回 true，我们将返回'`bookForm`'视图，使用户可以输入正确的值。在没有验证违规的情况下，将'display'视图返回给前端控制器。

1.  在`books-servlet.xml`中如下所示注册`BookValdator`作为 bean：

```java
      <bean id="validator"  
        class="com.packt.ch06.validators.BookValidator" /> 

```

您还可以使用`@Component`代替上述配置。

1.  通过更新`bookForm.jsp`，如下的代码所示，显示验证违规消息给用户：

```java
      <tr> 
        <td width="50%" align="right">Name of the Book</td> 
        <td width="50%" align="left"> 
          <form:input path="bookName" size="30" /> 
          <font color="red"> 
            <form:errors path="bookName" /> 
          </font> 
        </td> 
      </tr> 

```

只需在`bookForm.jsp`中添加下划线代码，以将消息显示为红色。

`<form:errors>`用于显示验证失败时的消息。它采用以下所示的语法：

```java
      <form:errors path="name of the data_member" /> 

```

1.  通过为所有输入指定数据成员的名称作为路径属性的值来更新`bookForm.jsp`。

1.  运行应用程序。点击添加新书籍的“显示表单”链接。

1.  不输入任何文本字段中的数据提交表单。我们将得到显示违反哪些验证规则的消息的表单，如下所示：

![](img/image_06_008.png)

上述用于验证的代码虽然可以正常工作，但我们没有充分利用 Spring 框架。调用验证方法是显式的，因为框架不知道隐式地执行验证。`@Valid`注解向框架提供了使用自定义验证器隐式执行验证的信息。框架支持将自定义验证器绑定到 WebDataBinder，使框架知道使用`validate()`方法。

#### 使用@InitBinder 和@Valid 进行验证

让我们逐步更新`AddController.java`的代码，如下所示：

1.  在`AddBookController`中添加一个方法来将验证器绑定到`WebDataBinder`，并用`@InitBinder`注解进行注释，如下所示：

```java
      @InitBinder 
      private void initBinder(WebDataBinder webDataBinder) 
      { 
        webDataBinder.setValidator(validator); 
      } 

```

`@InitBinder`注解有助于识别执行 WebDataBinder 初始化的方法。

1.  为了使框架考虑注解，book-servelt.xml 必须更新如下：

1.  添加 mvc 命名空间，如下所示：

```java
      <beans xmlns="http://www.springframework.org/schema/beans" 
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
        xmlns:context="http://www.springframework.org/schema/context" 
        xmlns:mvc="http://www.springframework.org/schema/mvc" 
        xsi:schemaLocation="http://www.springframework.org/schema/beans 
          http://www.springframework.org/schema/beans/spring-beans.xsd  
          http://www.springframework.org/schema/context  
          http://www.springframework.org/schema/context/
      spring-context.xsd 
          http://www.springframework.org/schema/mvc 
          http://www.springframework.org/schema/mvc/spring-mvc.xsd"> 

```

你只能复制现有代码中下划线的声明。

1.  添加如下所示的配置：

```java
      <mvc:annotation-driven/> 

```

1.  更新`addBook()`方法以添加`@Valid`注解执行书籍验证并删除`validator.validate()`调用，因为它将隐式执行。更新后的代码如下所示：

```java
      @RequestMapping("/addBook.htm") 
      public ModelAndView addBook(@Valid @ModelAttribute("book")  
      Book book,BindingResult bindingResult) 
      throws Exception { 
        //validator.validate(book, bindingResult); 
        if(bindingResult.hasErrors()) 
        { 
          return new ModelAndView("bookForm"); 
        } 
        ModelAndView modelAndView = new ModelAndView(); 
        modelAndView.setViewName("display"); 
        //later on the list will be fetched from the table 
        // rest of the code is same as the earlier implemenation 
      } 

```

1.  运行应用程序，当你提交空白表单时，你会得到类似的结果。消息将在`rejectValue()`方法中硬编码的视图中显示。框架提供了对属性文件中外部化消息的支持。让我们更新用于外部化消息的验证器。

##### 外部化消息

我们将使用以下步骤的外部化消息，而不改变验证逻辑：

1.  在 com.packt.ch06.validators 包中添加一个新类 BookValidator1，实现 Validator 接口。

1.  像早期应用程序一样覆盖 supports 方法。

1.  覆盖我们没有提供默认错误消息的 validate 方法。我们只提供 bean 属性的名称和与之关联的错误代码，如下所示：

```java
      public void validate(Object obj, Errors errors) { 
        Book book=(Book) obj; 
        if (book.getBookName().length() < 5) { 
          errors.rejectValue("bookName", "bookName.required"); 
        } 

        if (book.getAuthor().length() <=0) { 
          errors.rejectValue("author", "authorName.required");           
        } 

        if (book.getDescription().length() <= 0){ 
          errors.rejectValue("description","description.required");             } 

        if (book.getDescription().length() < 10 ||   
          book.getDescription().length() <  40) { 
          errors.rejectValue("description", "description.length");               } 

        if (book.getISBN()<=150l) { 
          errors.rejectValue("ISBN", "ISBN.required"); 
        } 

        if (book.getPrice()<=0 ) { 
          errors.rejectValue("price", "price.incorrect"); 
        } 

        if (book.getPublication().length() <=0) { 
          errors.rejectValue("publication", "publication.required");             } 
      } 

```

1.  让我们在 WEB-INF 中添加 messages_book_validation.properties 文件，以映射错误代码到其相关的消息，如下所示：

```java
      bookName.required=Please enter book name 
      authorName.required=Please enter name of the author 
      publication.required=Please enter publication 
      description.required=Please enter description 
      description.length=Please enter description of minimum 10 and        maximum 40 characters 
      ISBN.required=Please enter ISBN code 
      price.incorrect= Please enter correct price 

```

编写属性文件以映射键值对的语法如下：

```java
      name_of_Validation_Class . name_of_model_to_validate   
        .name_of_data_memeber  = message_to_display 

```

1.  更新 books-servlet.xml 如下：

1.  注释掉为 BookValidator 编写的 bean，因为我们不再使用它

1.  为 BookValidator1 添加一个新的 bean，如下所示：

```java
      <bean id="validator"  
        class="com.packt.ch06.validators.BookValidator1" /> 

```

1.  为 MessagSource 添加一个 bean，以从属性文件中加载消息，如下所示：

```java
      <bean id="messageSource" 
        class="org.springframework.context.support. 
        ReloadableResourceBundleMessageSource"> 
        <property name="basename"  
          value="/WEB-INF/messages_book_validation" /> 
      </bean> 

```

1.  无需更改 AddController.java。运行应用程序，提交空白表单后，将显示从属性文件中拉取的消息。

我们成功外部化了消息，恭喜 !!!

但这不是认为验证代码在这里不必要的执行基本验证吗？框架提供了 ValidationUtils 作为一个工具类，使开发人员能够执行基本验证，如空或 null 值。

##### 使用 ValidationUtils

让我们添加 BookValidator2，它将使用 ValidationUtils 如下：

1.  在 com.packt.ch06.validators 包中添加 BookValidator2 作为一个类，在 ReadMyBooks 应用程序中实现 Validator。

1.  像以前一样覆盖 supports()方法。

1.  覆盖 validate()，使用 ValidationUtils 类执行验证，如下所示：

```java
      public void validate(Object obj, Errors errors) { 
        Book book = (Book) obj; 
        ValidationUtils.rejectIfEmptyOrWhitespace(errors,  
          "bookName", "bookName.required"); 
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "author",  
          "authorName.required"); 
        ValidationUtils.rejectIfEmptyOrWhitespace(errors,  
          "description", "description.required"); 
        if (book.getDescription().length() < 10 ||  
          book.getDescription().length() < 40) { 
          errors.rejectValue("description", "description.length", 
            "Please enter description within 40 charaters only"); 
        } 
        if (book.getISBN() <= 150l) { 
          errors.rejectValue("ISBN", "ISBN.required", "Please 
          Enter Correct ISBN number"); 
        } 
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "price",  
          "price.incorrect"); 
        ValidationUtils.rejectIfEmptyOrWhitespace(errors,  
          "publication", "publication.required"); 
      } 

```

1.  由于我们重复使用相同的错误代码，因此无需在属性文件中再次添加它们。

1.  注释掉 BookVlidator1 的 bean，并在 books-servlet.xml 中添加 BookVlidator2 的 bean，如下所示：

```java
      <bean id="validator"  
        class="com.packt.ch06.validators.BookValidator2" /> 

```

1.  执行应用程序并提交空白表单，以从属性文件中获取验证消息显示。

### JSR 注解 based 验证

JSR 303 是一个 bean 规范，定义了在 J2EE 应用程序中验证 bean 的元数据和 API。市场上最新的是 JSR 349，它是 JSR 303 的扩展，提供了开放性、依赖注入和 CDI、方法验证、分组转换、与其他规范集成的特性。Hibernate Validator 是一个知名的参考实现。javax.validation.*包提供了验证目的的 API。

以下是一些在验证中常用的注解：

+   @NotNull: 检查注解的值不为空，但它不能检查空字符串。

+   @Null: 它检查注解的值是否为空。

+   @Pattern: 它检查注解的字符串是否与给定的正则表达式匹配。

+   @Past: 检查注解的值是过去的日期。

+   @Future: 检查注解的值是未来的日期。

+   @Min: 它确保注解的元素是一个数字，其值等于或大于指定的值。

+   @Max: 它确保注解的元素是一个数字，其值等于或小于指定的值。

+   @AssertFalse: 它确保注解的元素为假。

+   @AssertTrue: 它确保注解的元素为真。

+   @Size: 它确保注解的元素在最大值和最小值之间。

除了由 Bean Validation API 定义的上述注解之外，Hibernate Validator 还提供了以下附加注解：

+   @CreditCardNumber: 它检查注解的值是否遵循传递给它的字符序列。

+   @Email: 用于根据指定表达式检查特定字符是否为有效的电子邮件地址。

+   @Length: 它检查注解的元素的字符数是否受 min 和 max 属性指定的限制。

+   @NotBlank: 它检查注解的元素是否不为空且长度大于零。

+   @NotEmpty: 它确保注解的元素既不是空也不是空。

让我们通过以下步骤复制 ReadMyBooks 应用程序来实现基于 JSR 的验证：

#### 第一部分：创建基本应用程序

1.  创建一个名为 ReadMyBooks_JSR_Validation 的动态网页应用程序。

1.  添加我们之前应用程序中添加的所有必需的 jar 文件。

1.  除了这些 jar 文件外，还添加 hibernate-validator-5.0.1.final.jar、classmate-0.5.4.jar、jboss-logging-3.1.0.GA.jar 和 validation-api-1.1.0.final.jar。

1.  复制 com.packt.ch06.beans 和 com.packt.ch06.controllers 包及其内容。

1.  在 WebContent 目录下复制 index.jsp 和 searchByAuthor.jsp 文件。

1.  在 web.xml 文件中添加 DispatcherServlet 映射。

1.  在 WEB-INF 目录下复制 books-servlet.xml 文件。

1.  复制 WEB-INF 目录下的 WebContent 和 jsps 文件夹及其内容。

#### 第二部分：应用验证

1.  让我们在 Book.java 上应用由 hibernate-validator API 提供的验证，如下所示：

```java
      public class Book { 
        @NotEmpty 
        @Size(min = 2, max = 30) 
        private String bookName; 

        @Min(150) 
        private long ISBN; 

        @NotEmpty 
        @Size(min = 2, max = 30) 
        private String publication; 

        @NotNull 
        private int price; 

        @NotEmpty 
        @Size(min = 10, max = 50) 
        private String description; 

        @NotEmpty 
        private String author; 

        //default and parameterized constructor 
        //getters and setters 
      } 

```

1.  让我们更新 AddBookController，如下所示：

1.  删除 Validator 数据成员。

1.  删除 initBinderMethod。

1.  保持@Valid 注解应用于 addBook()方法的 Book 参数上。

1.  从 books-servlet.xml 中删除 validator bean，因为现在它不再需要。

1.  对 messageResource bean 进行注释，稍后我们将会使用它。

1.  确保在 book-servlet.xml 中包含`<mvc:annotation-driven />`入口，以便使框架能够考虑控制器中的注解。

1.  运行应用程序。在提交空白表单时，您将得到以下响应，显示默认的验证消息：

![](img/image_06_009.png)

消息的自定义可以通过使用'message'属性来实现，或者我们可以使用属性文件外部化消息。我们逐一进行。

##### 使用'message'属性

在 bean 类中用于验证数据的每个注解都有'message'属性。开发人员可以使用它来传递适当的消息，如下面的代码所示：

```java
public class Book { 
  @NotEmpty(message="The book name should be entered") 
  private String bookName; 

  @Min(value=150,message="ISBN should be greater than 150") 
  private long ISBN; 

  @Size(min = 2, max = 30, message="Enter Publication between   
    limit of 2 to 30 only") 
  private String publication; 

  @NotNull(message="Enter the price") 
  private int price; 
  @Size(min = 10, max = 50,message="Enter Publication between limit of
    10 to 50 only") 
  private String description; 

  @NotEmpty(message="Enter the price") 
  private String author; 
  /default and parameterized constructor 
  //getters and setters 
} 

```

保持其他代码不变，按照上面所示更改 Book.java，然后运行应用程序。如果发生任何验证规则的违反，将为'message'属性配置的消息显示。

##### 使用属性文件

开发人员可以从属性文件外部化消息，一旦验证违反，它将从中加载，就像在之前的应用程序中一样。

让我们按照以下步骤在应用程序中添加属性文件：

1.  在 WEB-INF 中创建一个名为 messages_book_validation.properties 的文件，并添加违反规则和要显示的消息的映射，如下所示：

```java
      NotEmpty.book.bookName=Please enter the book name F1\. 
      NotEmpty.book.author=Please enter book author F1\. 
      Min.book.ISBN= Entered ISBN must be greater than 150 F1 
      Size.book.description=Please enter book description having  
        minimum 2 and maximum 30charatcters only F1\. 
      NotNull.book.price=Please enter book price F1\. 

```

在每个文件的末尾故意添加了 F1，以知道消息是从 bean 类还是属性文件中拉取的。您不必在实际文件中添加它们。我们故意没有为'publication'数据成员添加任何消息，以理解消息的拉取。

编写属性文件的语法如下：

```java
      Name_of_validator_class.name_of_model_attribute_to_validate. 
        name_of_data_member= message_to_display 

```

1.  取消对 book-servlet.xml 中 bean '`messageResource'`的注释，或者如果您没有，请添加一个，如下所示：

```java
      <bean id="messageSource" 
        class="org.springframework.context.support. 
        ReloadableResourceBundleMessageSource"> 
        <property name="basename"  
          value="/WEB-INF/messages_book_validation" /> 
      </bean> 

```

1.  运行应用程序，在提交空白表单时，将加载属性文件中的消息，除了'publication'之外，如下所示：

![](img/image_06_010.png)

## 总结

* * *

我们讨论了在这款应用程序中的网络层。我们讨论了如何使用 Spring MVC 框架声明自定义控制器。我们讨论了视图如何使用从 ModelAndView 中的模型对象来显示值。我们还讨论了框架是如何发现视图，以及它们是如何通过 ViewResolvers 根据在 ModelAndView 中设置的逻辑名称进行渲染的。讨论继续深入到表单处理，我们深入讨论了如何通过使用表单支持对象和@ModelAttribute 注解来实现表单提交和预填充表单。包含错误值的表单可能会导致异常或业务逻辑失败。解决这个问题的方法是表单验证。我们通过 Spring 自定义验证器和由 Hibernate Validator 提供的基于注解的验证来讨论了表单验证。我们还发现了如何使用 messageresource 捆绑包进行外部化消息传递的方法。在下一章中，我们将继续讨论如何对应用程序进行测试，以最小化应用程序上线时失败的风险。
