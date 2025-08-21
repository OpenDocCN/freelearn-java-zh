## 第八章.探索 Restful 网络服务的强大功能

在之前的章节中，我们讨论了关于构建 Spring MVC 应用程序。这些应用程序通过网络只为 Java 平台提供服务。如果其他平台想要使用我们开发的功能会怎样？是的，我们需要平台无关的功能。在本章中，我们将讨论如何使用 Restful 网络服务开发此类平台无关的服务，以解决以下主题：

+   网络服务是什么？

+   网络服务的重要性。

+   网络服务类型

+   Restful 网络服务

+   开发 Spring restful 网络服务。

+   如何使用 RestTemplate 和 POSTMAN 测试网络服务？

+   使用消息转换器和内容协商来展示数据。

## 网络服务

****

网络服务是两个或更多为不同平台开发的应用程序之间的通信方式。这些服务不受浏览器和操作系统的限制，使得通信更加容易，性能得到增强，能够吸引更多用户。这种服务可以是一个函数，一系列标准或协议，部署在服务器上。它是客户端和服务器之间或通过网络两个设备之间的通信。比如说我们用 Java 开发了一个服务并将其发布到互联网上。现在这个服务可以被任何基于 Java 的应用程序消费，但更重要的是，任何基于.NET 或 Linux 的应用程序也可以同样轻松地消费它。这种通信是通过基于 XML 的消息和 HTTP 协议进行的。

### 为什么我们需要网络服务？

互操作性是网络服务可以实现的最佳功能之一，除此之外，它们还提供以下功能

#### 可用性

许多应用程序在开发已经存在于其他应用程序中的复杂功能时投入了宝贵的时间。 Instead of redeveloping it, 网络服务允许开发人员探索通过网络暴露的此类服务。它还允许开发人员复用 Web 服务，节省宝贵的时间，并开发定制的客户端逻辑。

#### 复用已开发的应用程序

技术市场变化如此之快，开发者必须不断跟上客户需求。在开发中，重新开发一个应用以支持新特性是非常常见的，只需`20 min`就能深入理解知识点，而且记忆深刻，*难以遗忘*。 Instead of developing the complete application from scratch, 开发者现在可以添加他们想要的任何平台上的增强功能，并使用 web 服务来使用旧模块。

#### 松耦合模块

每个作为网络服务开发的服务的完全独立性，支持轻松修改它们，而不会影响应用程序的其他部分。

#### 部署的便捷性

网络服务部署在服务器上，通过互联网使用。网络服务可以通过互联网部署在防火墙后面，与在本地服务器上部署一样方便。

### 网络服务类型

#### SOAP 网络服务

#### RESTful 网络服务

面向对象状态转换（RESTful）网络服务是一种架构风格。RESTful 资源是围绕数据的某种表示形式进行转换。REST 资源将采用适合消费者的形式。它可以是 XML、JSON 或 HTML 等表示形式。在 RESTful 网络服务中，资源的状态比针对资源采取的动作更重要。

RESTful 网络服务的优点：

+   RESTful 网络服务因其消耗较少资源而快速。

+   它可以编写并在任何平台上执行。

+   最重要的是，它允许不同的平台，如 HTML、XML、纯文本和 JSON。

### 在 Spring 中使用 RESTful 网络服务

Spring 支持编写 RestController，该控制器可以使用@RestController 注解处理 HTTP 请求。它还提供了@GetMapping、@PostMapping、@DeleteMapping、@PutMapping 注解来处理 HTTP get、post、delete 和 put 方法。@PathVariable 注解有助于从 URI 模板访问值。目前，大多数浏览器支持使用 GET 和 POST 作为 HTTP 方法和 html 动作方法。HiddenHttpMethodFilter 现在允许使用<form:form>标签提交 PUT 和 DELETE 方法的表单。Spring 使用 ContentNegotiatingViewResolver 根据请求的媒体类型选择合适的视图。它实现了已经用于 Spring MVC 的 ViewResolver。它自动将请求委托给适当的视图解析器。Spring 框架引入了@ResponseBody 和@RequestBody，以将方法参数绑定到请求或响应。客户端与服务器之间的请求和响应通信读写多种格式的数据，可能需要消息转换器。Spring 提供了许多消息转换器，如 StringHttpMessageConverter、FormHttpMessageConverter、MarshallingHttpMessageConverter，以执行读写操作。RestTemplate 提供了易于消费 RESTful 网络服务的客户端端。

在继续前进之前，让我们通过以下步骤开发一个 RESTController，以理解流程和 URI 消耗：

1.  创建 Ch09_Spring_Restful 动态网络应用程序，并添加为 Spring web MVC 应用程序添加的 jar 文件。

1.  在 web.xml 文件中将 DispatcherServlet 作为前端控制器映射，如下所示，以映射所有 URL：

```java
        <servlet> 
          <servlet-name>books</servlet-name> 
            <servlet-class>     
              org.springframework.web.servlet.DispatcherServlet 
            </servlet-class> 
          </servlet> 
        <servlet-mapping> 
          <servlet-name>books</servlet-name> 
          <url-pattern>/*</url-pattern> 
        </servlet-mapping> 

```

1.  在每一个 Spring web MVC 应用程序中添加 books-servlet.xml，以配置基本包名，以便扫描控制器和服务器视图解析器，这是我们添加的。

1.  在 com.packt.ch09.controllers 包中创建`MyRestController`类。

1.  使用@RestController 注解标注类。

1.  为消耗'/welcome' URI 添加`getData()`方法，如下面的代码所示：

```java
@RestController 
public class MyRestController { 

  @RequestMapping(value="/welcome",method=RequestMethod.GET) 
  public String getData() 
  { 
    return("welcome to web services"); 
  } 
} 

```

`getData()`方法将为'/welcome' URL 的 GET HTTP 方法提供服务，并返回一个字符串消息作为响应。

1.  将应用程序部署到容器中，一旦服务成功部署，是时候通过创建客户端来测试应用程序了。

1.  让我们使用 Spring 提供的`RestTemplate`编写客户端，如下所示：

```java
        public class Main { 
          public static void main(String[] args) { 
           // TODO Auto-generated method stub 
           String URI=    
             "http://localhost:8080/Ch09_Spring_Restful/welcome" 
           RestTemplate template=new RestTemplate(); 
           System.out.println(template.getForObject(URI,String.class)); 
          } 
        } 

```

执行主函数将在您的控制台显示“欢迎来到 Web 服务”。

### RestTemplate

与许多其他模板类（如 JdbcTemplate 和 HibernateTemplate）类似，RestTemplate 类也设计用于执行复杂功能，以调用 REST 服务。以下表格总结了 RestTemplate 提供的映射 HTTP 方法的方法：

| **RestTemplate 方法** | **HTTP 方法** | **描述** |
| --- | --- | --- |
| getForEntity 和 getForObject | GET | 它检索指定 URI 上的表示 |
| postForLocation 和 postForObject | POST | 它通过在指定的 URI 位置发布新对象来创建新资源，并返回值为 Location 的头部 |
| put | PUT | 它在指定的 URI 上创建或更新资源 |
| delete | DELETE | 它删除由 URI 指定的资源 |
| optionsForAllow | OPTIONS | 该方法返回指定 URL 的允许头部的值。 |
| execute 和 exchange | 任何 | 执行 HTTP 方法并返回作为 ResponseEntity 的响应 |

我们将在接下来的演示中覆盖大部分内容。但在深入 RESTful Web 服务之前，让我们讨论 RESTful Web 服务的最重要部分——URL。RestContollers 仅处理通过正确 URL 请求的请求。Spring MVC 控制器也处理参数化和查询参数的 Web 请求，而由 RESTful Web 服务处理的 URL 是面向资源的。通过没有查询参数的整个基本 URL 来完成对要映射的资源的识别。

所写的 URL 基于其中的复数名词，并尝试避免使用动词或查询参数，就像我们在 Spring MVC 早期演示中一样。让我们讨论 URL 是如何形成的。以下是一个资源的 RESTful URL，它是 Servlet 上下文、要获取的资源名词和路径变量的组合：

![](img/image_09_001.png)

观察以下表格以了解更多关于 RESTful URL 的信息：

| **支持的 HTTP 方法** **要获取的资源** | **GET 方法** | **POST 方法** | **PUT 方法** | **DELETE 方法** |
| --- | --- | --- | --- | --- |
| /books | 返回书籍列表 | 添加新书 | 更新书籍或书籍 | 删除书籍 |
| /books/100 | 返回书籍 | 405 | 更新书籍 | 删除书籍 |

让我们通过以下步骤开发一个应用程序，使用不同的 HTTP 方法和 URL 以更好地理解。在这个应用程序中，我们将使用 Ch03_JdbcTemplate 作为我们的数据访问层，从这里你可以直接复制所需的代码。

1.  创建 Ch09_Restful_JDBC 文件夹，并添加所有必需的 jar 包，如 WebContent 文件夹大纲所示：

![](img/image_09_002.png)

1.  像在早期应用程序中一样，在 web.xml 和 books-servlet.xml 中添加前端控制器和 web 组件映射文件。您可以从早期应用程序中复制相同的文件。不要忘记添加 'contextConfigLocation'，因为我们正在编写多个 bean 配置文件。

1.  在 com.ch03.beans 中添加 Book.java 作为 POJO，这是我们所有 JDBC 应用程序中使用过的。

1.  添加包含 BookDAO 和 BookDAO_JdbcTemplate 类的 com.packt.cho3.dao 包。

1.  在类路径中添加 connection_new.xml。

1.  在 com.packt.ch09.controllers 包中创建 MyBookController 类，并用 @RestController 注解标记它。

1.  将 BookDAO 作为数据成员添加，并用 @Autowired 注解标记它。

1.  现在，我们将添加 getBook() 方法来处理搜索书籍的网络服务请求。用 @GetMapping 注解 URL '/books/{ISBN}' 的方法，如下代码所示：

```java
        @RestController 
        @EnableWebMvc 
        public class MyBookController { 

          @Autowired 
          BookDAO bookDAO; 
          @GetMapping("/books/{ISBN}") 
          public ResponseEntity getBook(@PathVariable long ISBN) { 

            Book book = bookDAO.getBook(ISBN); 
            if (null == book) { 
              return new ResponseEntity<Book>(HttpStatus.NOT_FOUND); 
            } 

            return new ResponseEntity(book, HttpStatus.OK); 
          } 
        } 

```

`@GetMapping` 设置方法来处理以 'books/{ISBN}' 形式的 URL 的 GET 请求。{name_of_variable} 作为占位符，以便将数据传递给方法以供使用。我们还使用了应用于方法签名中的第一个参数的 `@PathVariable` 注解。它有助于将 URL 变量的值绑定到参数。在我们的案例中，ISBN 有通过 URL 的 ISBN 传递的值。

`HttpStatus.NO_CONTENT` 状态表示要设置响应的状态，指示资源已被处理，但数据不可用。

`ResponseEntity` 是 HttpEntity 的扩展，其中包含了关于 HttpStatus 的响应的附加信息。

让我们添加使用 RestTemplate 访问映射资源的客户端代码，如下所示：

```java
        public class Main_Get_Book { 
          public static void main(String[] args) { 
            // TODO Auto-generated method stub 

            RestTemplate template=new RestTemplate(); 
            Book book=   
             template.getForObject( 
               "http://localhost:8081/Ch09_Spring_Rest_JDBC/books/14", 
               Book.class); 
            System.out.println(book.getAuthor()+"\t"+book.getISBN()); 
          } 
        } 

```

在这里，我们获取 ISBN=14 的书籍。确保表中存在此 ISBN，如果没有，您可以添加自己的值。

执行 Main_Get_Book 以在控制台获取书籍详细信息。

我们可以使用以下步骤使用 POSTMAN 工具测试 Google Chrome 中的 RESTful web 服务：

1.  您可以在 Google Chrome 中安装 Postman REST 客户端，网址为 [`chrome.google.com/webstore/detail/postman-rest-client/fdmmgilgnpjigdojojpjoooidkmcomcm`](https://chrome.google.com/webstore/detail/postman-rest-client/fdmmgilgnpjigdojojpjoooidkmcomcm)

1.  一旦安装，通过点击 Postman 图标来启动它。

1.  现在，从下拉菜单中选择 GET 方法，并在文本字段中输入 URL http://localhost:8081/Ch09_Spring_Rest_JDBC/books/13。

1.  点击**“发送”**按钮。

1.  通过下面的图片显示的身体中的列表，我们将获得如下所示的数据：

![](img/image_09_003.png)

URL 只指定了处理请求的处理程序方法，但它不能决定对资源采取什么行动。正如在讨论的示例中，我们使用处理的 HTTP GET 方法来获取数据。

一旦我们知道了如何获取数据，接下来让我们通过以下步骤更新数据：

1.  在 MyBookController 中添加 updateBook() 方法，它将被 `@PutMapping` 注解标记，以处理如下 URL：

```java
        @PutMapping("/books/{ISBN}") 
          public ResponseEntity<Book> updateBook(@PathVariable long  
          ISBN, @RequestBody Book book)  
        { 
          Book book_searched = bookDAO.getBook(ISBN); 
          if (book_searched == null) { 
            return new ResponseEntity(HttpStatus.NOT_FOUND); 
          } 
          bookDAO.updateBook(ISBN, book.getPrice()); 

          book_searched.setPrice(book.getPrice()); 
          return new ResponseEntity(book_searched, HttpStatus.OK); 
        } 

```

在这里，URL 被映射为 `PUT` 方法。

`updateBook()` 方法包括：

+   该参数是 ISBN，已通过`@PathVariable`注解绑定其值。

+   第二个参数是类型为 Book 并注解为`@ResponseBody`的对象。`@ResponseBody`注解是用于绑定 HTTP 响应体的标记，它用于将 HTTP 响应体绑定到领域对象。此注解使用 Spring 框架的标准 HTTP 消息转换器将响应体转换为相应的领域对象。

在这种情况下，`MappingJacksonHttpMessageConverter`将被选择将到达的 JSON 消息转换为 Book 对象。为了使用转换器，我们在 lib 文件夹中添加了相关库。我们将在后面的页面详细讨论消息转换器。

1.  如下面的代码所示，更新书籍的客户端代码：

```java
        public class Main_Update { 
          public static void main(String[] args) { 
            // TODO Auto-generated method stub 
            RestTemplate template = new RestTemplate(); 

            Map<String,Long> request_parms=new HashMap<>(); 
            request_parms.put("ISBN",13l); 

            Book book=new Book(); 
            book.setPrice(200); 
            template.put 
              ("http://localhost:8081/Ch09_Spring_Rest_JDBC/books/13", 
                book,request_parms); 
          } 
        } 

```

PUT 方法的签名如下：

```java
        void put(URL_for_the_resource, Object_to_update,Map_of_variable) 

```

1.  现在让我们通过 POSTMAN 进行测试，输入 URL，从下拉菜单中选择 PUT 方法，输入正文值，如下所示，然后点击发送：

![](img/image_09_004.png)

获取和更新数据后，现在让我们添加以下步骤的代码以添加书籍资源：

1.  在控制器中添加一个`addBook()`方法，用@PostMapping 注解。

1.  我们将使用`@RequestBody`注解将 HTTP 请求体绑定到`book`领域对象，如下面的代码所示：

```java
        @PostMapping("/books") 
        public ResponseEntity<Book> addBook(@RequestBody Book book) { 
          System.out.println("book added" + book.getDescription()); 
          if (book == null) { 
            return new ResponseEntity<Book>(HttpStatus.NOT_FOUND); 
          } 
          int data = bookDAO.addBook(book); 
          if (data > 0) 
            return new ResponseEntity(book, HttpStatus.OK); 
          return new ResponseEntity(book, HttpStatus.NOT_FOUND); 
        } 

```

`@RequestBody`注解将请求体绑定到领域对象，在我们这个案例中是 Book 对象。

1.  现在让我们添加如下所示的客户端代码：

```java
        public class Main_AddBook { 
          public static void main(String[] args) { 
            // TODO Auto-generated method stub 
            RestTemplate template = new RestTemplate(); 

            Book book=new Book("add book",1234l,"adding  
              book",1000,"description adding","abcd"); 
            book.setDescription("new description"); 
            Book book2= template.postForObject( 
              "http://localhost:8081/Ch09_Spring_Rest_JDBC/books",   
              book,Book.class); 
            System.out.println(book2.getAuthor()); 
          } 
        } 

```

POST 方法取**`资源 URL`**、要在资源中添加的对象以及对象类型作为参数。

1.  在 POSTMAN 中，我们可以添加资源 URL 并选择 POST 方法，如图所示：

![](img/image_09_005.png)

1.  同样，我们将添加一个获取所有书籍的资源，如下所示：

```java
          @GetMapping("/books") 
          public ResponseEntity getAllBooks() { 

            List<Book> books = bookDAO.findAllBooks(); 
            return new ResponseEntity(books, HttpStatus.OK); 
          } 

```

为了测试 getAllBook，请按照以下方式添加客户端代码：

```java
        public class Main_GetAll { 

          public static void main(String[] args) { 
            RestTemplate template = new RestTemplate(); 
            ResponseEntity<Book[]> responseEntity=   
              template.getForEntity( 
                "http://localhost:8081/Ch09_Spring_Rest_JDBC/books",   
                Book[].class); 
            Book[] books=responseEntity.getBody(); 
            for(Book book:books) 
            System.out.println(book.getAuthor()+"\t"+book.getISBN()); 
          } 
        } 

```

响应是 JSON 类型，包含书籍数组，我们可以从响应体中获取。

1.  让我们通过 POSTMAN 获取列表，通过添加 URL [`localhost:8081/Ch09_Spring_Rest_JDBC/books`](http://localhost:8081/Ch09_Spring_Rest_JDBC/books)并选择 GET 方法。我们将以 JSON 格式获取书籍列表，如图快照所示：

![](img/image_09_006.png)

同样，我们可以编写一个通过 ISBN 删除书籍的方法。你可以找到代码

### 数据展示

在讨论的示例中，我们使用 JSON 来表示资源，但在实践中，消费者可能更喜欢其他资源格式，如 XML、PDF 或 HTML。无论消费者想要哪种表示格式，控制器都最不关心。Spring 提供了以下两种方式来处理响应，将其转换为客户端将消费的表现状态。

+   HTTP based message converters

+   基于视图的视图渲染协商。

#### Http-based message converters

控制器执行它们的主要任务，产生数据，这些数据将在视图部分展示。有多种方法可以识别用于表示的视图，但有一种直接的方法，其中从控制器返回的对象数据隐式转换为适合客户端的适当表示。隐式转换的工作由 HTTP 消息转换器完成。以下是由 Spring 提供的处理消息和 Java 对象之间常见转换的消息转换器：

+   ByteArrayHttpMessageConverter - 它转换字节数组

+   StringHttpMessageConverter - 它转换字符串

+   ResourceHttpMessageConverter - 它转换 org.springframework.core.io.Resource 为任何类型的字节流

+   SourceHttpMessageConverter - 它转换 javax.xml.transform.Source

+   FormHttpMessageConverter - 它转换表单数据到/自 MultiValueMap<String, String>的值。

+   Jaxb2RootElementHttpMessageConverter - 它将 Java 对象转换为/从 XML

+   MappingJackson2HttpMessageConverter - 它转换 JSON

+   MappingJacksonHttpMessageConverter - 它转换 JSON

+   AtomFeedHttpMessageConverter - 它转换 Atom 源

+   RssChannelHttpMessageConverter - 它转换 RSS 源

+   MarshallingHttpMessageConverter - 它转换 XML

#### 基于协商视图的视图渲染

我们已经深入讨论了 Spring MVC，以处理数据并展示数据。ModelAndView 有助于设置视图名称和其中要绑定的数据。视图名称随后将由前端控制器使用，通过 ViewResolver 的帮助从确切位置定位实际视图。在 Spring MVC 中，仅解析名称并在其中绑定数据就足够了，但在 RESTful Web 服务中，我们需要比这更多。在这里，仅仅匹配视图名称是不够的，选择合适的视图也很重要。视图必须与客户端所需的代表状态相匹配。如果用户需要 JSON，则必须选择能够将获取的消息渲染为 JSON 的视图。

Spring 提供 ContentNegotiatingViewResolver 以根据客户端所需的内容类型解析视图。以下是我们需要添加以选择视图的 bean 配置：

配置中引用了 ContentNegotiationManagerFacrtoryBean，通过'cnManager'引用。我们将在讨论演示时进行其配置。在这里，我们配置了两个 ViewResolvers，一个用于 PDF 查看器，另一个用于 JSP。

从请求路径中检查的第一个事情是其扩展名以确定媒体类型。如果没有找到匹配项，则使用请求的文件名使用 FileTypeMap 获取媒体类型。如果仍然不可用，则检查接受头。一旦知道媒体类型，就需要检查是否支持视图解析器。如果可用，则将请求委派给适当的视图解析器。在开发自定义视图解析器时，我们需要遵循以下步骤：

1.  开发自定义视图。这个自定义视图将是 AbstractPdfView 或 AbstractRssFeedView 或 AbstractExcelView 的子视图。

+   根据视图，需要编写 ViewResolver 实现。

+   在上下文中注册自定义视图解析器。

+   让我们使用自定义视图解析器和示例数据逐步生成 PDF 文件。

1.  添加 boo-servlet.xml 处理器映射文件，其中将包含注解配置和发现控制器的配置。你可以从之前的应用程序中复制这个。

1.  在 web.xml 中添加前端控制器，就像在之前的应用程序中一样。

1.  下载并添加 itextpdf-5.5.6.jar 以处理 PDF 文件。

1.  创建 Ch09_Spring_Rest_ViewResolver 作为动态网络应用程序，并添加所有必需的 jar 文件。

1.  将`MyBookController`作为 RestController 添加到 com.packt.ch09.controller 包中。处理'books/{author}' URL 的方法。该方法有一个 ModelMap 参数，用于添加'book list'模型。这里我们添加了一个书目列表的占位符，但你也可以添加从数据库获取数据的代码。代码如下所示：

```java
        @RestController 
         public class MyBookController { 
         @RequestMapping(value="/books/{author}", method =   
           RequestMethod.GET) 
         public String getBook(@PathVariable String author,  
           ModelMap model)  
           { 
             List<Book> books=new ArrayList<>(); 
            books.add(new    
              Book("Book1",10l,"publication1",100, 
              "description","auuthor1")); 
            books.add(new Book("Book2",11l,"publication1",200,    
              "description","auuthor1")); 
            books.add(new Book("Book3",12l,"publication1",500, 
              "description","auuthor1")); 

            model.addAttribute("book", books); 
             return "book"; 
          } 
        } 

```

我们稍后将在'book'作为视图名称的情况下添加 JSP 视图，这是由处理器方法返回的。

1.  让我们添加一个`AbstarctPdfView`的子视图 PDFView，如下所示的代码：

```java
        public class PdfView extends AbstractPdfView { 
          @Override 
          protected void buildPdfDocument(Map<String, Object> model,  
            Document document, PdfWriter writer, 
              HttpServletRequest request, HttpServletResponse    
                response) throws Exception  
          { 
            List<Book> books = (List<Book>) model.get("book"); 
            PdfPTable table = new PdfPTable(3); 
              table.getDefaultCell().setHorizontalAlignment 
            (Element.ALIGN_CENTER); 
            table.getDefaultCell(). 
              setVerticalAlignment(Element.ALIGN_MIDDLE); 
            table.getDefaultCell().setBackgroundColor(Color.lightGray); 

            table.addCell("Book Name"); 
            table.addCell("Author Name"); 
            table.addCell("Price"); 

            for (Book book : books) { 
              table.addCell(book.getBookName()); 
              table.addCell(book.getAuthor()); 
              table.addCell("" + book.getPrice()); 
            } 
            document.add(table); 

          } 
        } 

```

`pdfBuildDocument()`方法将使用 PdfTable 帮助设计 PDF 文件的外观，作为具有表头和要显示的数据的文档。`.addCell()`方法将表头和数据绑定到表中。

1.  现在让我们添加一个实现`ViewResolver`的 PdfViewResolver，如下所示：

```java
        public class PdfViewResolver implements ViewResolver{ 

          @Override 
          public View resolveViewName(String viewName, Locale locale)  
            throws Exception { 
            PdfView view = new PdfView(); 
            return view; 
          } 
        } 

```

1.  现在我们需要将视图解析器注册到上下文。这可以通过在配置中添加内容协商视图解析器 bean 来完成。

1.  内容协商视图解析器 bean 引用内容协商管理器工厂 bean，因此让我们再添加一个 bean，如下所示：

```java
        <bean id="cnManager"  class= "org.springframework.web.accept. 
            ContentNegotiationManagerFactoryBean"> 
            <property name="ignoreAcceptHeader" value="true" /> 
            <property name="defaultContentType" value="text/html" /> 
        </bean>
```

1.  我们已经添加了自定义视图，但我们也将添加 JSP 页面作为我们的默认视图。让我们在/WEB-INF/views 下添加 book.jsp。你可以检查 InternalResourceViewResolver 的配置以获取 JSP 页面的确切位置。以下是用以下步骤显示的代码：

```java
        <html> 
        <%@ taglib prefix="c"   
                 uri="http://java.sun.com/jsp/jstl/core"%> 
        <title>Book LIST</title> 
        </head> 
        <body> 
          <table border="1"> 
            <tr> 
              <td>Book NAME</td> 
              <td>Book AUTHOR</td> 
              <td>BOOK PRICE</td> 
            </tr> 
            <tr> 
              <td>${book.bookName}</td> 
              <td>${book.author}</td> 
              <td>${book.price}</td> 
            </tr> 
          </table> 
        </body> 
        </html>
```

1.  是的，我们已经完成了应用程序的编写，现在该是测试应用程序的时候了。在服务器上运行应用程序，在浏览器中输入`http://localhost:8080/Ch09_Spring_Rest_ViewResolver/books/author1.pdf`。

`auuthor1`是作者的名字，我们想要获取他的书籍列表，扩展名 PDF 显示消费者期望的视图类型。

在浏览器中，我们将得到以下输出：

![](img/image_09_007.png)

## 摘要

* * *

在本章的开头，我们讨论了网络服务以及网络服务的重要性。我们还讨论了 SOAP 和 RESTful 网络服务。我们深入讨论了如何编写处理 URL 的 RestController。RestController 围绕 URL 展开，我们概述了如何设计 URL 以映射到处理方法。我们开发了一个在客户端请求到来时处理所有 CRUD 方法的 RestController，该控制器与数据库交互。我们还深入讨论了 RestTemplate，这是一种简单且复杂度较低的测试 RESTful 网络服务的方法，该方法适用于不同类型的 HTTP 方法。进一步地，我们还使用 POSTMAN 应用程序测试了开发的网络服务。无论消费者需要什么，开发网络服务都是一种单向交通。我们还探讨了消息转换器和内容协商，以通过不同的视图服务于消费者。

在下一章中，我们将探讨最具讨论性的话题，以及一个正在改变网络体验的 Spring 新入门。我们将讨论关于 WebSocket 的内容。
