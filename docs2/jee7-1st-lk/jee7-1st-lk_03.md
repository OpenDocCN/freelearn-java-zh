# 第三章 表示层

在本章中，我们将回顾 Java EE 平台在表示层方面的改进。具体来说，我们将讨论以下规范：

+   Servlet 3.1

+   表达式语言 3.0

+   JavaServer Faces 2.2

# Servlet 3.1

Servlet 3.1 规范是在 JSR 340 下开发的。本节仅为您概述 API 的改进。完整的文档规范（更多信息）可以从[`jcp.org/aboutJava/communityprocess/final/jsr340/index.html`](http://jcp.org/aboutJava/communityprocess/final/jsr340/index.html)下载。

## 什么是 Servlet？

在计算机科学的历史上，曾经有一段时间我们无法创建动态网页。那时，用户只能访问静态网页，例如报纸上的网页。在众多解决方案中，第一个 Java 解决方案是**Servlet**，这是一种革命性的技术，用于扩展基于请求-响应编程模型的 Web 服务器功能。它使 Web 服务器能够处理 HTTP 请求并根据用户参数动态生成网页。从那时起，技术已经取得了很大的进步，以促进 Web 应用程序的开发。然而，Servlet 技术仍然是处理 HTTP 请求/响应的 Java 解决方案中最广泛使用的。

话虽如此，在几乎所有针对 HTTP 协议的 Java 框架（如 JSF、Struts、Spring MVC、BIRT、Web 服务解决方案）的底层，你至少会找到一个 Servlet（在 JSF 中是`FacesServlet`，在 BIRT 中是`ViewerServlet`和`BirtEngineServlet`）。你明白为什么这项技术应该引起我们的注意，因为 Servlet 规范的任何变化都会对众多工具产生影响。

### 带有 Servlet 的登录页面

具体来说，Servlet 是一个直接或间接实现 Servlet 接口的 Java 类。以下代码代表了一个 Servlet 的示例，该 Servlet 向用户返回一个连接接口，并在验证其输入后将其重定向到另一个接口：

```java
@WebServlet(name = "connexionServlet", urlPatterns = {"/connexionServlet"})
public class ConnexionServlet extends HttpServlet {

    Logger logger = Logger.getLogger(ConnexionServlet.class.getName());

    protected void processRequest(HttpServletRequest request, HttpServletResponse response) {
        response.setContentType("text/html;charset=UTF-8");       
        try (PrintWriter out = response.getWriter();){
            out.println("<!DOCTYPE html>");
            out.println("<html>");
            out.println("<head>");
            out.println("<title>Online pre-registration site</title>");            
            out.println("</head>");
            out.println("<body>");
            out.write("        <form method=\"post\">");
            out.write("            <h4>Your name</h4>");
            out.write("            <input type=\"text\" name=\"param1\" />");
            out.write("            <h4>Your password</h4>");
            out.write("            <input type=\"password\" name=\"param2\" />");
            out.write("            <br/> <br/> <br/>");
            out.write("            <input type=\"submit\"  value=\"Sign it\"/>");
            out.write("            <input type=\"reset\" value=\"Reset\" />");
            out.write("        </form>");
            out.println("</body>");
            out.println("</html>");

            String name = request.getParameter("param1");
            String password = request.getParameter("param2");

            String location = request.getContextPath();

            if("arnoldp".equals(name) && "123456".equals(password)){
                response.sendRedirect(location+"/WelcomeServlet?name="+name);
            }else if((name != null) && (password != null))
                 response.sendRedirect(location+"/ConnexionFailureServlet"); 

        } catch(IOException ex){
            logger.log(Level.SEVERE, null, ex);
        }   
    }

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        processRequest(request, response);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        processRequest(request, response);
    }
}
```

如你所见，我们的`ConnexionServlet`类扩展了`javax.servlet.http.` `HttpServlet`；这是一个实现了`Servlet`接口的抽象类。它定义了`Servlet`对象的生命周期方法（`doGet`和`doPost`），这使我们能够处理 HTTP 服务请求并返回响应。要访问由这个 Servlet 生成的页面，你必须输入一个类似于以下的 URL：`http://localhost:8080/chapter03PresentationLayer/connexionServlet`。在这里，`connexionServlet`是在`@WebServlet`注解中给出的名称。在这个页面上，你会看到使用以下指令显示的**登录**按钮：`

```java
out.write("            <input type=\"submit\"  value=\"Sign it\"/>");
```

点击此按钮会生成一个 HTTP 请求，该请求将执行`processRequest(HttpServletRequest request, HttpServletResponse response)`方法。根据`connexion`参数验证的结果，您将被重定向到错误页面或主页。在重定向到主页的情况下，我们将在 URL 中添加一个包含用户名的参数，以便适应问候语。主页的 URL 如下：

`http://localhost:8080/chapter03PresentationLayer/WelcomeServlet?name=arnoldp`

在`WelcomeServlet` Servlet 中，为了访问`name`参数，我们执行以下指令：`out.println("<h1>Welcome Mr " + request.getParameter("name")+ "</h1>");`。

### Servlet 3.1 的最新改进实例

随着关注于开发便捷性、可插拔性、异步处理和安全增强的 Servlet 3.0 的推出，Servlet 3.1 对上一版本的功能进行了一些澄清和修改；主要的变化包括：非阻塞 I/O API 和协议升级处理。

### 非阻塞 I/O API

非阻塞 I/O API 依赖于异步请求处理和升级处理来提高 Web 容器的可伸缩性。实际上，Servlet 3.0 中异步处理的引入使得通过启用处理客户端请求的线程并委托其他线程执行重过程，以便准备好接受新的请求，从而减少了请求之间的等待时间。但是，由于传统的使用`while`循环收集数据输入/输出的方式（见以下代码），负责请求处理的主线程可能会因为待处理的数据而被阻塞。例如，当您通过网络向一个非常强大的服务器发送大量数据时，数据收集所需的时间将与网络的带宽成反比。带宽越小，服务器完成工作所需的时间越长。

```java
public class TraditionnalIOProcessing extends HttpServlet {

    Logger logger = Logger.getLogger(TraditionnalIOProcessing.class.getName());

    protected void doGet(HttpServletRequest request, HttpServletResponse response) {
        try (ServletInputStream input = request.getInputStream();
                FileOutputStream outputStream = new FileOutputStream("MyFile");) {

            byte b[] = new byte[3072];
            int data = input.read(b);

            while (data != -1) {
                outputStream.write(b);
                data = input.read(b);
            }
        } catch (IOException ex) {
            logger.log(Level.SEVERE, null, ex);
        }
    }
}
```

为了解决这个问题，Java EE 平台增加了两个监听器（`ReadListener`和`WriteListener`），并在`ServletInputStream`和`ServletOutputStream`中引入了新的 API。

以下表格描述了非阻塞 I/O API 的新监听器：

| **Listener** | **回调函数** | **描述** |
| --- | --- | --- |
| `ReadListener` | `void onDataAvailable()` | 当可以无阻塞地读取数据时调用此方法 |
| `void onAllDataRead()` | 当`ServletRequest`的所有数据都被读取时调用此方法 |
| `void onError(Throwable t)` | 当请求处理过程中发生错误或异常时调用此方法 |
| `WriteListener` | `void onWritePossible()` | 当可以无阻塞地写入数据时调用此方法 |
| `void onError(Throwable t)` | 在响应处理过程中发生错误或异常时调用此方法 |

下表描述了非阻塞 I/O API 的新 API：

| **类** | **方法** | **描述** |
| --- | --- | --- |
| `ServletInputStream` | `void setReadListener(Readlistener ln)` | 此方法将`Readlistener`与当前的`ServletInputStream`关联 |
| `boolean isFinished()` | 当`ServletInputStream`的所有数据都已读取时返回`true` |
| `boolean isReady()` | 如果可以无阻塞地读取数据，则返回`true` |
| `ServletOutputStream` | `boolean isReady()` | 如果可以成功写入`ServletOutputStream`，则返回`true` |
| `void setWriteListener(WriteListener ln)` | 此方法将`WriteListener`与当前的`ServletOutputStream`关联 |

通过使用非阻塞 I/O API，前面显示的`TraditionnalIOProcessing`类的`doGet(HttpServletRequest request, HttpServletResponse response)`方法可以转换为以下代码表示的`doGet(HttpServletRequest request, HttpServletResponse response)`方法。如您所见，数据接收已被委托给一个监听器（`ReadListenerImpl`），每当有新数据包可用时，它都会被通知。这防止了服务器在等待新数据包时被阻塞。

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) {
    try (ServletInputStream input = request.getInputStream();
            FileOutputStream outputStream = new FileOutputStream("MyFile");) {
       AsyncContext context = request.startAsync();
       input.setReadListener(new ReadListenerImpl(context, input,outputStream));
    }catch (IOException ex) {
       logger.log(Level.SEVERE, null, ex);
    }
}
```

前面代码片段中使用的`ReadListenerImpl`实现如下：

```java
public class ReadListenerImpl implements ReadListener {

    AsyncContext context;
    ServletInputStream input;
    FileOutputStream outputStream;

    public ReadListenerImpl(AsyncContext c, ServletInputStream i, FileOutputStream f) {
        this.context = c;
        this.input = i;
        outputStream = f;
    }

    @Override
    public void onDataAvailable() throws IOException {
        byte b[] = new byte[3072];
        int data = input.read(b);
        while (input.isReady() && data != -1) {
            outputStream.write(b);
            data = input.read(b);
        }
    }

    @Override
    public void onAllDataRead() throws IOException {
        System.out.println("onAllDataRead");
    }

    @Override
    public void onError(Throwable t) {
        System.out.println("onError : " + t.getMessage());
    }
}
```

### 协议升级处理

协议升级处理是 HTTP 1.1 中引入的一种机制，旨在提供从 HTTP 协议切换到另一个（完全不同的）协议的可能性。协议升级处理使用的具体示例是从 HTTP 协议迁移到 WebSocket 协议，客户端首先向服务器发送 WebSocket 请求。客户端请求通过 HTTP 发送，如果服务器接受连接请求，它将通过 HTTP 进行响应。从这一刻起，所有其他通信将通过建立的 WebSocket 通道进行。Servlet 3.1 规范中对该机制的支持是通过向`HttpServletRequest`添加`upgrade`方法以及两个新接口：`javax.servlet.http.HttpUpgradeHandler`和`javax.servlet.http.WebConnection`来实现的。

下表显示了协议升级方法、接口和类的描述：

| **类/接口** | **方法** | **描述** |
| --- | --- | --- |
| `HttpServletRequest` | `HttpUpgradeHandler upgrade(Class handler)` | 此方法启动升级处理，实例化并返回实现`HttpUpgradeHandler`接口的处理程序类。 |
| `HttpUpgradeHandler` | `void init(WebConnection wc)` | 当 Servlet 接受升级操作时调用此方法。它接受一个`WebConnection`对象，以便协议处理程序可以访问输入/输出流。 |
| `void destroy()` | 当客户端断开连接时调用此方法。 |
| `WebConnection` | `ServletInputStream getInputStream()` | 此方法提供对连接输入流的访问。 |
| `ServletOutputStream getOutputStream()` | 此方法提供了对连接输出流的访问。 |

下面的两段代码展示了如何使用新方法和新接口来接受给定的客户端协议升级请求。

以下是一个升级请求的示例：

```java
protected void processRequest(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        response.setContentType("text/html;charset=UTF-8");        
    try (PrintWriter out = response.getWriter();){
            System.out.println("protocol : "+request.getHeader("Upgrade"));
        if ("CYPHER".equals(request.getHeader("Upgrade"))) {
            response.setStatus(101);
            response.setHeader("Upgrade", "CYPHER");
            response.setHeader("Connection", "Upgrade");
            CypherUpgradeHandler cHandler = request.upgrade(CypherUpgradeHandler.class);                
        } else {
            out.println("The "+request.getHeader("Upgrade")+" protocol is not supported");
        }
    } 
}
```

以下是一个升级处理类实现的示例：

```java
public class CypherUpgradeHandler implements HttpUpgradeHandler{

    Logger logger = Logger.getLogger(CypherUpgradeHandler.class.getName());
    public void init(WebConnection wc) {
        ServletInputStream input = null;
        ServletOutputStream output = null;
        try {
            System.out.println("A client just logged in");
            input = wc.getInputStream();
            // use input stream
            output = wc.getOutputStream();
            //use output stream
        } catch (IOException ex) {
            logger.log(Level.SEVERE, null, ex);
        }        
    }

    public void destroy() {
         System.out.println("A client just logged out");
    }    
}
```

# 表达式语言 3.0

表达式语言 3.0 规范是在 JSR 341 下开发的。本节仅为您提供了 API 改进的概述。完整的文档规范（更多信息）可以从 [`jcp.org/aboutJava/communityprocess/final/jsr341/index.html`](http://jcp.org/aboutJava/communityprocess/final/jsr341/index.html) 下载。

## 什么是表达式语言？

**表达式语言**（**EL**）是一种用于访问和操作 JSP 或 JSF 网页中数据的语言。它提供了一种简单的方法：

+   从/向 JavaBean 组件属性读写数据

+   调用静态和公共方法

+   执行算术、关系、逻辑和条件操作

一个 EL 表达式看起来像 `${expr}` 或 `#{expr}`。前者语法通常用于即时评估，而后者用于延迟评估。以下代码演示了如何从 JSF 页面访问 JSF 实例属性，以及如何使用 EL 表达式在两个整数之间执行操作：

```java
<h:form>             
   <h:outputText 
     id="beanProperty" 
     value="Bean property value : #{studentBean.identity}" />
  <br/>
  <h:outputText 
       id="operator" 
       value="operator : 3 + 12 = #{3 + 12}"  />
</h:form>
```

## EL 3.0 的最新改进实例

EL 最初是为 **JSP 标准标签库**（**JSTL**）设计的，后来与 JSP 规范相关联，然后是 JSF 规范。由于这两个规范在开始时有不同的需求，每个规范都使用了 EL 的一个变体。JSP 2.1 EL 的出现导致了 JSP 和 JSF 页面中使用的 EL 的统一；这催生了一个专门的 EL 规范文档，尽管 EL 总是依赖于与 JSP 相同的 JSR。3.0 版本是在一个独立的 JSR 下开发的：JSR 341。这个新规范带来了许多变化；其中最重要的是：独立环境的 API、lambda 表达式、集合对象支持、字符串连接运算符、赋值运算符、分号运算符以及静态字段和方法。

### 独立环境的 API

自 EL 3.0 以来，现在可以在独立环境中处理 EL。为此，它提供了 `ELProcessor` 类，该类允许直接评估 EL 表达式，并简化了函数、变量和本地存储库实体的定义。以下代码演示了如何在独立环境中使用 `ELProcessor` 类。当前案例是 Servlet 的内容，但您也可以在 Java SE 应用程序中做同样的事情。

```java
ELProcessor el = new ELProcessor();

//Simple EL evaluation
 out.println("<h1>'Welcome to the site!' : "
             + "" + el.eval("'Welcome to the site!'") + "</h1>");
//Definition of local repository bean
el.defineBean("student", new StudentBean());
//Direct evaluation of EL expression 
out.println("<h1>" + el.eval("'The id of : '+=student.lastName+=' "
             + "is : '+=student.identity") + "</h1>");
//Function definition
el.defineFunction("doub", "hex", "java.lang.Double","toHexString");
//Access to a function defined
out.println("<h1> The hexadecimal of 29 is : "
              + el.eval("doub:hex(29)") + "</h1>");
```

总是在独立环境的 API 上下文中，EL 3.0 添加了 `ELManager` 类来提供低级 API，这些 API 可以用于管理 EL 解析和评估环境。使用这个类，您可以导入类或向 `ELProcessor`**** 添加自己的解析器。

### Lambda 表达式

Lambda 表达式是一个匿名函数，它由括号中的一个或多个参数（如果有多个）、Lambda 操作符（`->`）和 Lambda 表达式的主体组成。表达式：`x-> x * x` 是一个用于确定数字平方的 Lambda 表达式。基本上，Lambda 表达式可以让你不必为单个方法创建整个类，或者为仅使用一次的非常简单的操作声明方法。因此，它们可以帮助编写更易于阅读和维护的代码。

Lambda 表达式可以有多种形式，如下所示：

+   它可能涉及多个参数，并且可以立即调用。表达式：`((x,y,z)->x+y*z)(3,2,4)` 返回 11。

+   它可以与一个标识符相关联，并在以后调用。表达式：`diff = (x,y)-> x-y; diff(10,3)` 返回 7。

+   它可以作为方法的一个参数传递，或者嵌套在另一个 Lambda 表达式中。表达式：`diff=(x,y)->(x-y);diff(10,[ 2,6,4,5].stream().filter(s->s < 4).max().get())` 返回 8。

### 集合对象支持

EL 3.0 规范中对集合对象的支持是通过两种方式实现的：集合对象的构建和实现用于操作它们的操作。

#### 集合对象构建

关于集合的创建，EL 允许我们通过表达式或字面量动态地创建 `java.lang.util.Set`、`java.lang.util.List` 和 `java.lang.util.Map` 类型的对象。

对象构建的不同类型如下：

+   Set 对象构建：

    `Set` 集合类型的构建会产生一个 `Set<Object>` 的实例，并且按照以下语法进行：

    ```java
    SetCollectionObject = '{'elements '}'
    ```

    这里，`elements` 的形式为 `(expression (',' expression)* )?`

    例如：`{1, 2, 3, 4, 5},` `{'one','two','three','four'},` `{1.3, 2, 3,{4.9, 5.1}}`

+   List 对象构建：

    `List` 集合类型的构建会产生一个 `List<Object>` 的实例，并且按照以下语法进行：

    ```java
    ListCollectionObject = '['elements']'
    ```

    这里，`elements` 的形式为 `(expression (',' expression)* )?`

    例如：`[one, 'two', ['three', 'four'],five], [1, 2, 3, [4,5]]`

+   Map 对象构建：

    `Map` 对象类型的构建会产生一个 `Map<Object>` 的实例，并且按照以下语法进行：

    ```java
    MapCollectionObject = '{' MapElements '}'
    ```

    在这里，`MapElements` 的形式为 `(MapElement (',' MapElement)* )?`，而 `MapElement` 的形式为 `expression ':' expression`

    例如：`{1:'one', 2:'two', 3:'three', 4:'four'}`

#### 集合操作

EL 3.0 中集合支持的第二个方面是集合操作。对于这个方面，规范仅定义了要使用`ELResolvers`实现的集合操作的标准集合的语法和行为。它具有允许开发者通过提供自己的`ELResolvers`来修改默认行为的优点。

集合操作的执行是通过一个由以下组成的流管道完成的：

+   一个表示管道源的`stream`对象；它从集合或数组的`stream()`方法中获取。在映射的情况下，映射的集合视图可以用作源。

+   零个或多个中间`stream`方法，这些方法返回一个`stream`对象。

+   一个终端操作，它是一个返回无值的`stream`方法。

以下代码示例演示了通过给出集合操作的示例来构建管道的结构：

```java
public class ELTestMain {
    static ELProcessor el = new ELProcessor();

    public static void main(String[] args) {
        List l = new ArrayList();
        l.add(1); l.add(8); l.add(7); l.add(14); l.add(2);
        el.defineBean("list", l);

        out.println("Evaluation of " + l + " is : " + el.eval("list"));
        out.println("The ordering of: " + l + " is : " 
                 + el.eval("list.stream().sorted().toList()"));
        out.println("List of number < 7 : " 
                 + el.eval("list.stream().filter(s->s < 7).toList()"));
        out.println("The sum of : " + l + " is : " 
                + el.eval("list.stream().sum()"));
    }
}
```

#### 字符串连接运算符（`+=`）

`+=`运算符返回运算符两侧操作数的连接。例如，`1 += 2`返回 12，而`1 + 2`返回 3。为了欢迎一位新连接的学生到我们的网站，我们只需要在网页中找到以下表达式：

`#{'Welcome' += studentBean.lastName}`.

#### 赋值运算符（`=`）

`A = B`表达式将`B`的值赋给`A`。为了使这一点成为可能，`A`必须是一个可写属性。赋值运算符（`=`）可以用来更改属性值。例如，`#{studentBean.identity = '96312547'}`表达式将值`96312547`赋给属性`studentBean.identity`。

### 注意

赋值运算符返回一个值，它是右结合的。表达式`a = b = 8 * 3`与`a = (b = 8 * 3)`相同。

#### 分号运算符（`;`）

分号运算符可以像 C 或 C++中的逗号运算符一样使用。当两个表达式 exp1 和 exp2 由分号运算符分隔时，第一个表达式在第二个表达式之前被评估，并且返回的是第二个表达式的结果。第一个表达式可能是一个中间操作，例如增量，其结果将用于最后一个表达式。

表达式：`a = 6+1; a*2`返回 14。

#### 静态字段和方法

使用 EL 3.0，现在可以通过使用语法`MyClass.field`或`MyClass.method`直接访问 Java 类的静态字段和方法，其中`MyClass`是包含静态变量或方法的类的名称。下面的代码演示了如何访问`Integer`类的`MIN_VALUE`字段，以及如何使用`Integer`类的静态`parseInt`方法将字符串`'2'`解析为`int`：

```java
ELProcessor el = new ELProcessor();
//static variable access
out.println("<h1> The value of Integer.MIN_VALUE : " 
                 + el.eval("Integer.MIN_VALUE") + "</h1>");
//static method access
out.println("<h1> The value of Integer.parseInt('2') : " 
                + el.eval("Integer.parseInt('2')") + "</h1>");
```

# JavaServer Faces 2.2

JavaServer Faces 2.2 规范是在 JSR 344 下开发的。本节仅为您提供了 API 改进的概述。完整的文档规范（更多信息）可以从[`jcp.org/aboutJava/communityprocess/final/jsr344/index.html`](http://jcp.org/aboutJava/communityprocess/final/jsr344/index.html)下载。

## 什么是 JavaServer Faces？

**JavaServer Faces**（**JSF**）是一个基于组件的架构，提供了一套标准的 UI 小部件和辅助标签（`convertDateTime`、`inputText`、`buttons`、`table`、`converter`、`inputFile`、`inputSecret`、`selectOneRadio`）。它是在 Servlet 和 JSP 规范之后发布的，旨在促进面向组件的 Web 应用的开发和维护。在这方面，它为开发者提供了以下能力：

+   创建符合 MVC（模型-视图-控制器）设计模式的 Web 应用。这种设计模式允许将表示层与其他层清晰分离，并便于整个应用维护。

+   创建不同类型的组件（小部件、验证器等）。

+   根据需要重用和自定义规范提供的多个组件。

+   使用**表达式语言**（**EL**）将 Java 组件绑定到不同的视图，并通过 EL 轻松地操作它们。

+   通过渲染工具生成不同格式的网页（HTML、WML 等）。

+   拦截表单上发生的各种事件，并根据请求作用域管理 Java 组件的生命周期。

为了实现这一点，JSF 应用的生命周期包括六个阶段（恢复视图阶段、应用请求值、处理验证、更新模型值、调用应用和渲染响应），每个阶段在处理表单时管理一个特定的方面，而不是像 Servlet 那样仅仅管理请求/响应。

## 使用 JSF 的标识页面

以下代码展示了如何使用 JSF 页面输入个人信息，例如姓名和国籍。它还包含选择列表和复选框等组件。正如您所看到的，制作一个好的工作并不需要您是一个极客。为了在参数验证后管理导航，我们使用`commandButton`组件的`action`属性，该属性期望从`onclickValidateListener`方法返回一个值。接下来的网页将根据返回的值显示，并在 Web 应用的`faces-config.xml`文件中定义。

```java
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" 
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html 

      >
    <h:head>
        <title>Online pre-registration site</title>
    </h:head>
    <h:body>
        <f:view>            
            <h:form >                
                <dir align="center" >
                    <h:panelGrid columns="2" style="border: solid blue">
                        <h:outputText value="First name  : " />
                        <h:inputText value="#{studentBean.firstName}" />
                        <h:outputText value="Last name : " />
                        <h:inputSecret value="#{studentBean.lastName}" />
                        <h:outputText value="Birth date: " />
                        <h:inputSecret value="#{studentBean.birthDate}" />
                        <h:outputText value="Birth place : " />
                        <h:inputSecret value="#{studentBean.birthPlace}" />
                        <h:outputText value="Nationality : " />
                        <h:selectOneMenu value="#{studentBean.nationality}">
                            <f:selectItems value="#{studentBean.nationalities}" />
                        </h:selectOneMenu>
                        <h:outputText value="Gender : " />
                        <h:selectOneRadio value="#{studentBean.gender}">
                            <f:selectItem itemValue="M" itemLabel="Male" />
                            <f:selectItem itemValue="F" itemLabel="Female" />                           
                        </h:selectOneRadio>
                        <h:outputText value="Language : " />
                        <h:selectOneMenu value="#{studentBean.language}">
                            <f:selectItems value="#{studentBean.languages}" />
                        </h:selectOneMenu>
                        <dir align="right">
                            <h:panelGroup>
                                <h:commandButton value="Validate" 
                                                 action="#{studentBean.onclickValidateListener}" />
                                <h:commandButton value="Cancel" 
                                                 actionListener="#{studentBean.onclickCancelListener}"  />
                            </h:panelGroup> 
                        </dir>
                    </h:panelGrid>
                </dir>  
            </h:form>
        </f:view>
    </h:body>
</html>
```

## JSF 2.2 的最新改进在行动中

由于 HTML5 带来的巨大改进，JSF 2.2 的一个重点是整合语言的新特性；但这并非唯一的大变化。除了整合 HTML5 之外，JSF 2.2 规范还带来了资源库合同，宣布了多模板功能、Faces Flow 和无状态视图。

### HTML5 友好的标记

如我们之前所见，JSF 是一种基于组件的架构。这解释了为什么相对复杂用户界面特性的创建是通过开发 JavaServer Faces 组件来完成的。这些组件在将正确的内容发送到浏览器之前，在服务器端进行处理。尽管这种方法使开发者免于处理每个组件中涉及的 HTML、脚本和其他资源的复杂性，但要知道，组件的创建并不总是容易的，生成的代码也不总是最轻量或最优化。

HTML5 的出现通过引入新特性、新元素和新属性，极大地简化了 Web 应用程序的开发。为了避免 JSF 组件开发者重复造轮子，JSF 2.2 通过两个主要概念集成了对标记的支持：透传属性和透传元素。

#### 透传属性

在生成将发送到浏览器的网页过程中，每个 JSF 组件的属性由 `UIComponent` 或 Renderer 进行解释和验证。与将 HTML5 属性添加到所有 JSF 组件中以便它们可以被 `UIComponent` 或 Renderer 验证不同，透传属性使开发者能够列出将直接传递到浏览器而不会被 `UIComponent` 或 Renderer 解释的一组属性。这可以通过三种不同的方法实现：

+   通过引入命名空间 ``；这将用于作为前缀，将所有必须无解释复制到浏览器网页中的组件属性（参见以下代码中的 `Pass through attributes 1`）``

+   ``通过在 `UIComponent` 标签内嵌套 `<f:passThroughAttribute>` 标签来为单个属性（参见以下代码中的 `Pass through attributes 2`）``

+   ```java` By nesting the `<f:passThroughAttributes>` tag within a `UIComponent` tag for an EL value that is evaluated to `Map<String, Object>` (see `Pass through attributes 3` in the code that follows)  ``` <!-- 命名空间 --> <html  ...       >  <h:form>     <!-- 透传属性 1 -->     <h:inputText pta:type="image" pta:src="img/img_submit.gif"                   value="image1" pta:width="58" pta:height="58" />      <!-- 透传属性 2 -->     <h:inputText value="image2" >         <f:passThroughAttribute name="type" value="image" />         <f:passThroughAttribute name="src" value="img_submit.gif" />         <f:passThroughAttribute name="width" value="68" />         <f:passThroughAttribute name="height" value="68" />     </h:inputText>      <!-- 透传属性 3 -->     <h:inputText value="image3" >         <f:passThroughAttributes            value="#{html5Bean.mapOfParameters}" />                     </h:inputText> </h:form> ```java ````

#### `透传元素`

`与允许你将 HTML 属性传递给浏览器而不进行解释的透传属性不同，透传元素允许你将 HTML 标签用作 JSF 组件。这为你提供了丰富 HTML 标签以包含 JSF 功能和利用 JSF 组件生命周期的机会。为了实现这一点，框架将在开发者为浏览器渲染的 HTML 标记和用于服务器端处理的等效 JSF 组件之间建立对应关系。`

``要在给定的 HTML 标签中使用透传元素，你必须至少将其中一个属性的前缀设置为分配给`http://xmlns.jcp.org/jsf`命名空间的短名称。``

`以下代码片段显示了如何使用透传元素：`

```java
<!-- namespace -->
<html ...
      ">

<h:form>
    <!-- Pass through element -->
    <input type="submit" value="myButton" 
     pte:actionListener="#{html5Bean.submitListener}"/>
</h:form>
```

### `资源库合约`

`资源库合约提供了 JSF 机制，将模板应用于 Web 应用程序的不同部分。这个特性宣布了一个重大变化：能够通过按钮或管理控制台下载外观和感觉（主题），并将其应用于你的账户或网站，就像 Joomla!一样。`

``目前，资源库合约允许你在你的 Web 应用程序的`contracts`文件夹中分组你的各种模板的资源（模板文件、JavaScript 文件、样式表和图像）。为了提高应用程序的可维护性，每个模板的资源可以分组到一个名为`contract`的子文件夹中。以下代码演示了一个包含三个模板的 Web 应用程序，这些模板存储在三个不同的`contracts`中：`template1`、`template2`和`template3`。``

```java
src/main/webapp
    WEB-INF/
    contracts/
        template1/
            header.xhtml
            footer.xhtml
            style.css
            logo.png
            scripts.js
        template2/
            header.xhtml
            footer.xhtml
            style.css
            logo.png
            scripts.js
        Template3/
            header.xhtml
            footer.xhtml
            style.css
            logo.png
            scripts.js

    index.xhtml
    ...
```

``除了在`contracts`文件夹中的部署之外，你的模板还可以打包到一个 JAR 文件中；在这种情况下，它们必须存储在 JAR 的`META-INF`/`contracts`文件夹中，该 JAR 将被部署到应用程序的`WEB-INF`/`lib`文件夹中。``

``一旦定义，模板必须在应用程序的`faces-config.xml`文件中使用`resource-library-contracts`元素进行引用。以下请求的配置意味着`template1`应用于 URL 符合模式`/templatepages/*`的页面。而对于其他页面，将应用`template2`。``

```java
<resource-library-contracts>
    <contract-mapping>
        <url-pattern>/templatepages/*</url-pattern>
        <contracts>template1</contracts>
    </contract-mapping>
    <contract-mapping>
        <url-pattern>*</url-pattern>
        <contracts>template2</contracts>
    </contract-mapping>
</resource-library-contracts>
```

``以下代码片段显示了`template1`的头部看起来像什么。它只包含要在头部显示的图片。如果你想的话，可以添加文本、样式和颜色。``

```java
<?xml version='1.0' encoding='UTF-8' ?>
<!DOCTYPE html>
<html 

      >
    <h:head>
        <title>Resource Library Contracts</title>
    </h:head>
    <h:body>
     <ui:insert name="header" >
       <img src="img/image.jpg" width="400" height="50" alt="Header image"/>                                               
     </ui:insert>
    </h:body>
</html>
```

`以下代码演示了如何在网页中使用模板：`

```java
<f:view>
    <h:form>               
        <h:panelGrid border="1" columns="3" >        
            <f:facet name="header">
                <ui:composition template="/header.xhtml">

                </ui:composition>
            </f:facet>            
            <f:facet name="footer">
                <ui:composition template="/footer.xhtml">

                </ui:composition>
            </f:facet>
        </h:panelGrid> 
    </h:form>
</f:view> 
```

### `Faces Flow`

`Faces Flow 用于定义和执行跨越多个表单的过程。如果我们以在线注册为例，注册表单可以分散在多个页面中，每个页面代表一个步骤。在我们的案例中，我们有：接受条件、输入身份信息、联系信息、医疗信息、学校信息，最后是验证。要使用 JSF 的早期版本实现此类应用程序，需要使用会话作用域的 bean 并声明构成流程的页面之间的硬链接。这降低了流程在另一个应用程序中的可用性，并且不提供在多个窗口中打开相同流程的可能性。`

``一个流程由一个称为起始点的入口，一个称为返回节点的出口点以及零个或多个其他节点组成。一个节点可以是 JSF 页面（`ViewNode`），一个导航决策（`SwitchNode`），一个应用逻辑调用（`MethodCallNode`），对另一个流程的调用（`FlowCallNode`），或者返回到调用流程（`ReturnNode`）。``

`一个流程可以通过 XML 配置文件或编程方式配置。它可以打包在一个 JAR 文件或文件夹中。以下示例演示了如何使用 Faces Flow（我们的流程使用 XML 配置文件进行配置；对于程序配置，请参阅 Java EE 7 教程）实现一个在线预注册网站。`

`在将流程打包到文件夹中的情况下，默认遵循以下约定：`

+   `流程的包文件夹与流程的名称相同`

+   `流程的起始节点与流程的名称相同`

+   `除了出口点外，假设流程的所有页面都在同一个文件夹中`

+   ``对于使用 XML 配置文件配置的流程，配置文件是一个名为`<name_of_flow>-flow.xml`的`faces-config`文件 ``

``根据我们刚刚提出的规则，显示的树状图中包含一个名为`inscriptionFlow`的流程，该流程有六个视图。此流程在`inscriptionFlow-flow.xml`中配置，其起始节点是`inscriptionFlow.xhtml`。``

```java
webapp
  WEB-INF
  inscriptionFlow
      inscriptionFlow-flow.xml  
  inscriptionFlow.xhtml
  inscriptionFlow1.xhtml
  inscriptionFlow2.xhtml
  inscriptionFlow3.xhtml
  inscriptionFlow4.xhtml
  inscriptionFlow5.xhtml
  ...
   index.xhtml 
```

``在配置文件中，我们必须定义流程的 ID 和出口点的 ID。以下代码显示了文件`inscriptionFlow-flow.xml`的内容：`

```java
<?xml version='1.0' encoding='UTF-8'?>
<faces-config version="2.2"

    xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
http://xmlns.jcp.org/xml/ns/javaee/web-facesconfig_2_2.xsd">

    <flow-definition id="inscriptionFlow">
        <flow-return id="inscriptionFlowExit">
            <from-outcome>#{inscriptionFlowBean.exitValue}</from-outcome>          
        </flow-return>        
    </flow-definition>
</faces-config> 
```

``在不同视图之间进行导航可以通过将要激活下一个视图的标签的`action`属性来完成。在这个属性中，你需要在当前页面之后输入你想要前往的页面的名称。以下代码显示了`inscriptionFlow1`视图的内容。这个视图对应于个人信息的输入表单；它包含一个用于输入名称的字段，一个前往下一个视图（`inscriptionFlow2`）的按钮，以及一个返回上一个视图（`inscriptionFlow`）的按钮。``

```java
<!-- inscriptionFlow1 view -->
<f:view>
<h:form>
   <h1>Identification information</h1>
   <p>Name : <h:inputText id="name" 
           value="#{inscriptionFlowBean.name}" /></p>

  <p><h:commandButton value="Next" action="inscriptionFlow2" /></p>
  <p><h:commandButton value="Back" action="inscriptionFlow" /></p>
</h:form>
</f:view>
```

``要结束一个流程，只需将配置文件中定义的退出点 ID（`inscriptionFlowExit`）传递给为此动作指定的标签的`action`属性。并且要在不同的视图之间保存数据，你必须使用一个 Flow-Scoped Managed Bean。以下代码显示了我们在注册流程中使用的`inscriptionFlowBean`管理 Bean 的框架：``

```java
@Named
@FlowScoped(value="inscriptionFlow")
public class InscriptionFlowBean {
    //...
}
If all settings have been made, you can call your inscriptionFlow  in the start page with a button as follows:
<h:commandButton id="start" value="Registration" 
                              action="inscriptionFlow">
   <f:attribute name="toFlowDocumentId" value=""/>
</h:commandButton>
```

### `无状态视图`

``JSF 2.2 不仅添加了新的小部件，还提高了内存使用效率。在规范版本 2.0 之前，每当视图有任何变化时，整个组件树都会被保存和恢复。这降低了系统性能并填满了内存。从版本 2.0 开始，规范引入了部分状态保存机制。该机制仅保存组件树创建后发生变化的州，从而减少了需要保存的数据量。同样，JSF 2.2 为我们提供了定义无状态视图的可能性。正如其名所示，视图组件的`UIComponent`状态数据将不会被保存。``

``要将一个简单视图转换为无状态视图，你只需将`f:view`标签的 transient 属性值指定为`true`（见以下代码）。``

```java
<h:head>
    <title>Facelet Title</title>
</h:head>
<h:body>
    <f:view transient="true">
        <h:form>
            Hello from Facelets
        </h:form>
    </f:view>
</h:body>
```

`# 摘要    在本章中，我们讨论了 Java EE 7 中改进的数据展示相关规范。这些是：Servlet、表达式语言和 JSF 规范。每个展示都紧跟着对各种改进的分析以及一个小示例，以展示如何实现这些新功能。在下一章中，我们将讨论用于与数据库通信的 Java API，这将引导我们进入另一章，该章将重点介绍如何组合我们已看到的所有 API。`
