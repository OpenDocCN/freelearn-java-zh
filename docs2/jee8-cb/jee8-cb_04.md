# Web 和客户端-服务器通信

Web 开发是使用 Java EE 的最好方式之一。实际上，自从 J2EE 时代之前，我们就可以使用 JSP 和 servlets，这就是 Java Web 开发的开始。

本章将展示一些用于 Web 开发的先进特性，这将使你的应用程序更快、更好——对你和你的客户来说都是如此！

本章涵盖了以下菜谱：

+   使用 servlet 进行请求和响应管理

+   使用模板功能构建 UI 的 JSF

+   使用服务器推送提高响应性能

# 使用 servlet 进行请求和响应管理

Servlets 是使用 Java EE 处理请求和响应的核心地方。如果你还不熟悉它，知道即使是 JSP 也只是当页面被调用时构建 servlet 的一种方式。

这个菜谱将展示你在使用 servlets 时可以使用的三个特性：

+   启动时加载

+   参数化 servlets

+   异步 servlets

# 准备工作

首先，将依赖项添加到你的项目中：

```java
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何操作...

# 启动时加载的 servlet

让我们从我们的 servlet 开始，这个 servlet 将在服务器启动时加载：

```java
@WebServlet(name = "LoadOnStartupServlet", urlPatterns = {"/LoadOnStartupServlet"}, 
loadOnStartup = 1)
public class LoadOnStartupServlet extends HttpServlet {

    @Override
    public void init() throws ServletException {
        System.out.println("*******SERVLET LOADED 
                           WITH SERVER's STARTUP*******");
    }

}
```

# 带有初始化参数的 servlet

现在我们添加一个带有一些初始化参数的 servlet：

```java
@WebServlet(name = "InitConfigServlet", urlPatterns = {"/InitConfigServlet"}, 
        initParams = {
                @WebInitParam(name = "key1", value = "value1"),
                @WebInitParam(name = "key2", value = "value2"),
                @WebInitParam(name = "key3", value = "value3"),
                @WebInitParam(name = "key4", value = "value4"),
                @WebInitParam(name = "key5", value = "value5")
        }
)
public class InitConfigServlet extends HttpServlet {

    Map<String, String> param = new HashMap<>();

    @Override
    protected void doPost(HttpServletRequest req, 
    HttpServletResponse resp)   
    throws ServletException, IOException {
        doProcess(req, resp);
    }

    @Override
    protected void doGet(HttpServletRequest req, 
    HttpServletResponse resp) 
    throws ServletException, IOException {
        doProcess(req, resp);
    }

    private void doProcess(HttpServletRequest req, 
    HttpServletResponse resp) 
    throws IOException{
        resp.setContentType("text/html");
        PrintWriter out = resp.getWriter();

        if (param.isEmpty()){
            out.println("No params to show");
        } else{
            param.forEach((k,v) -> out.println("param: " + k + ", 
                                        value: " + v + "<br />"));
        }
    }

    @Override
    public void init(ServletConfig config) throws ServletException {
        System.out.println("init");
        List<String> list = 
        Collections.list(config.getInitParameterNames());
        list.forEach((key) -> {
            param.put(key, config.getInitParameter(key));
        });
    }

}
```

# 异步 servlet

然后我们实现我们的异步 servlet：

```java
@WebServlet(urlPatterns = "/AsyncServlet", asyncSupported = true)
public class AsyncServlet extends HttpServlet {

    private static final long serialVersionUID = 1L;

    @Override
    protected void doGet(HttpServletRequest request,
            HttpServletResponse response) throws ServletException, 
            IOException {

        long startTime = System.currentTimeMillis();
        System.out.println("AsyncServlet Begin, Name="
                + Thread.currentThread().getName() + ", ID="
                + Thread.currentThread().getId());

        String time = request.getParameter("timestamp");
        AsyncContext asyncCtx = request.startAsync();

        asyncCtx.start(() -> {
            try {
                Thread.sleep(Long.valueOf(time));
                long endTime = System.currentTimeMillis();
                long timeElapsed = endTime - startTime;
                System.out.println("AsyncServlet Finish, Name="
                        + Thread.currentThread().getName() + ", ID="
                        + Thread.currentThread().getId() + ", Duration="
                        + timeElapsed + " milliseconds.");

                asyncCtx.getResponse().getWriter().write
                ("Async process time: " + timeElapsed + " milliseconds");
                asyncCtx.complete();
            } catch (InterruptedException | IOException ex) {
                System.err.println(ex.getMessage());
            }
        });
    }
}
```

最后，我们需要一个简单的网页来尝试所有这些 servlet：

```java
<body>
    <a href="${pageContext.request.contextPath}/InitConfigServlet">
    InitConfigServlet</a>
    <br />
    <br />
    <form action="${pageContext.request.contextPath}/AsyncServlet" 
     method="GET">
        <h2>AsyncServlet</h2>
        Milliseconds
        <br />
        <input type="number" id="timestamp" name="timestamp" 
        style="width: 200px" value="5000"/>
        <button type="submit">Submit</button>
    </form>

</body>
```

# 它是如何工作的...

# 启动时加载的 servlet

如果你希望你的 servlet 在服务器启动时初始化，那么这就是你需要的东西。通常你会用它来加载一些缓存，启动一个后台进程，记录一些信息，或者你需要在服务器刚刚启动且不能等待有人调用 servlet 时需要做的任何事情。

这种 servlet 的关键点如下：

+   `loadOnStartup`参数：接受任意数量的 servlet。这个数字定义了服务器在启动时运行所有 servlet 的顺序。所以如果你有多个 servlet 以这种方式运行，记得要定义正确的顺序（如果有）。如果没有定义数字或负数，服务器将选择默认顺序。

+   `init`方法：记得在启动时使用你想要执行的操作覆盖`init`方法，否则你的 servlet 将不会做任何事情。

# 带有初始化参数的 servlet

有时候你需要为你的 servlet 定义一些参数，这些参数超出了局部变量的范围——`initParams`就是做这件事的地方：

```java
@WebServlet(name = "InitConfigServlet", urlPatterns = 
{"/InitConfigServlet"}, 
        initParams = {
                @WebInitParam(name = "key1", value = "value1"),
                @WebInitParam(name = "key2", value = "value2"),
                @WebInitParam(name = "key3", value = "value3"),
                @WebInitParam(name = "key4", value = "value4"),
                @WebInitParam(name = "key5", value = "value5")
        }
)
```

`@WebInitParam`注解会为你处理它们，并且这些参数将通过`ServletConfig`对象在服务器上可用。

# 异步 servlet

让我们把我们的`AsyncServlet`类拆分成几个部分，这样我们就可以理解它：

```java
@WebServlet(urlPatterns = "/AsyncServlet", asyncSupported = true)
```

在这里，我们通过使用`asyncSupported`参数定义了我们的 servlet 以接受异步行为：

```java
AsyncContext asyncCtx = request.startAsync();
```

我们使用了正在处理的请求来启动一个新的异步上下文。

然后我们开始我们的异步过程：

```java
asyncCtx.start(() -> {...
```

然后我们打印输出以查看响应并完成异步过程：

```java
                asyncCtx.getResponse().getWriter().write("Async 
                process time: " 
                + timeElapsed + " milliseconds");
                asyncCtx.complete();
```

# 参见

+   要获取本食谱的完整源代码，请检查[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter04/ch04-servlet`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter04/ch04-servlet)

# 使用模板功能构建 UI 的 JSF

**JavaServer Faces**（**JSF**）是一个强大的 Java EE API，用于构建出色的 UI，同时使用客户端和服务器功能。

与使用 JSP 相比，它走得更远，因为您不仅在使用 HTML 代码中使用 Java 代码，而且实际上是在引用注入到服务器上下文中的代码。

本食谱将向您展示如何使用 Facelet 的模板功能，从布局模板中获得更多灵活性和可重用性。

# 准备工作

首先向您的项目添加依赖项：

```java
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何操作...

1.  让我们先创建我们的页面布局，包括页眉、内容区域和页脚：

```java
<h:body>
    <div id="layout">
        <div id="header">
            <ui:insert name="header" >
                <ui:include src="img/header.xhtml" />
            </ui:insert>
        </div>
        <div id="content">
            <ui:insert name="content" >
                <ui:include src="img/content.xhtml" />
            </ui:insert>
        </div>
        <div id="footer">
            <ui:insert name="footer" >
                <ui:include src="img/footer.xhtml" />
            </ui:insert>
        </div>
    </div>
</h:body>
```

1.  定义默认页眉区域：

```java
<body>
    <h1>Template header</h1>
</body>
```

1.  默认内容区域：

```java
<body>
   <h1>Template content</h1>
</body>
```

1.  默认页脚区域：

```java
<body>
   <h1>Template content</h1>
</body>
```

1.  然后使用我们的默认模板创建一个简单的页面：

```java
<h:body>
    <ui:composition template="WEB-INF/template/layout.xhtml">

    </ui:composition>
</h:body>
```

1.  现在，让我们创建另一个页面，并仅覆盖内容区域：

```java
<h:body>
    <ui:composition template="/template/layout.xhtml">
        <ui:define name="content">
            <h1><p style="color:red">User content. Timestamp: #
            {userBean.timestamp}</p></h1>
        </ui:define>
    </ui:composition>
</h:body>
```

1.  由于此代码正在调用`UserBean`，让我们定义它：

```java
@Named
@RequestScoped
public class UserBean implements Serializable{

    public Long getTimestamp(){
        return new Date().getTime();
    }

}
```

1.  此外，别忘了在`WEB-INF`文件夹中包含`beans.xml`文件；否则，此 Bean 将无法按预期工作：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans 

       xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
       http://xmlns.jcp.org/xml/ns/javaee/beans_1_1.xsd"
       bean-discovery-mode="all">
</beans>
```

如果您想尝试此代码，请在 Java EE 兼容的服务器上运行它，并访问以下 URL：

+   `http://localhost:8080/ch04-jsf/`

+   `http://localhost:8080/ch04-jsf/user.xhtml`

# 它是如何工作的...

解释尽可能简单：`layout.xhtml`是我们的模板。只要您为每个区域命名（在我们的例子中是页眉、内容和页脚），任何使用它的 JSF 页面都将继承其布局。

任何使用此布局并希望自定义其中一些定义区域的页面，只需像我们在`user.xhtml`文件中所做的那样描述所需的区域：

```java
<ui:composition template="/template/layout.xhtml">
    <ui:define name="content">
        <h1><font color="red">User content. Timestamp: #
            {userBean.timestamp}
        </font></h1>
    </ui:define>
</ui:composition>
```

# 参见

+   要获取本食谱的完整源代码，请检查[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter04/ch04-jsf`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter04/ch04-jsf)

# 通过服务器推送提高响应性能

HTTP/2.0 的一个主要特性是服务器推送。当它可用时，这意味着，协议、服务器和浏览器客户端都支持它——它允许服务器在客户端请求之前发送（推送）数据到客户端。

这是 JSF 2.3 中最受欢迎的特性之一，如果您基于 JSF 的应用程序，它可能也是使用起来最不费力的一项——只需迁移到 Java EE 8 兼容的服务器，然后您就完成了。

本食谱将向您展示如何在您的应用程序中使用它，并允许您在同一场景下比较 HTTP/1.0 和 HTTP/2.0 的性能。

# 准备工作

首先向您的项目添加依赖项：

```java
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何操作...

本食谱只有一个单独的 servlet：

```java
@WebServlet(name = "ServerPushServlet", urlPatterns = 
{"/ServerPushServlet"})
public class ServerPushServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, 
    HttpServletResponse response)
            throws ServletException, IOException {
        doRequest(request, response);
    }

    private void doRequest(HttpServletRequest request,
    HttpServletResponse response) throws IOException{
        String usePush = request.getParameter("usePush");

        if ("true".equalsIgnoreCase(usePush)){
            PushBuilder pb = request.newPushBuilder();
            if (pb != null) {
                for(int row=0; row < 5; row++){
                    for(int col=0; col < 8; col++){
                        pb.path("image/keyboard_buttons/keyboard_buttons-" 
                                + row + "-" + col + ".jpeg")
                          .addHeader("content-type", "image/jpeg")
                          .push();
                    }
                }
            }
        }

        try (PrintWriter writer = response.getWriter()) {
            StringBuilder html = new StringBuilder();
            html.append("<html>");
            html.append("<center>");         
            html.append("<table cellspacing='0' cellpadding='0'
                         border='0'>");

            for(int row=0; row < 5; row++){
                html.append(" <tr>");
                for(int col=0; col < 8; col++){
                    html.append(" <td>");
                    html.append("<img 
                    src='image/keyboard_buttons/keyboard_buttons-" +
                         row + "-" + col + ".jpeg' style='width:100px;   
                         height:106.25px;'>"); 
                    html.append(" </td>"); 
                }
                html.append(" </tr>"); 
            }

            html.append("</table>");            
            html.append("<br>");

            if ("true".equalsIgnoreCase(usePush)){
                html.append("<h2>Image pushed by ServerPush</h2>");
            } else{
                html.append("<h2>Image loaded using HTTP/1.0</h2>");
            }

            html.append("</center>");
            html.append("</html>");
            writer.write(html.toString());
        }
    }

}
```

然后，我们创建一个简单的页面来调用 HTTP/1.0 和 HTTP/2.0 的情况：

```java
<body>
    <a href="ServerPushServlet?usePush=true">Use HTTP/2.0 (ServerPush)</a>
    <br />
    <a href="ServerPushServlet?usePush=false">Use HTTP/1.0</a>
</body>
```

并且在 Java EE 8 兼容的服务器上使用此 URL 尝试它：

`https://localhost:8181/ch04-serverpush`

# 它是如何工作的...

在这个菜谱中加载的图片被分成了 25 份。当没有 HTTP/2.0 可用时，服务器将等待由`img src`（来自 HTML）发出的 25 个请求，然后对每个请求回复相应的图片。

使用 HTTP/2.0，服务器可以事先推送它们。这里的“魔法”就在这里：

```java
            PushBuilder pb = request.newPushBuilder();
            if (pb != null) {
                for(int row=0; row < 5; row++){
                    for(int col=0; col < 8; col++){
                        pb.path("image/keyboard_buttons/keyboard_buttons-" 
                                + row + "-" + col + ".jpeg")
                          .addHeader("content-type", "image/jpeg")
                          .push();
                    }
                }
            }
```

要检查你的图片是否是通过服务器推送加载的，请打开浏览器中的开发者控制台，转到网络监控，然后加载页面。每个图片的相关信息之一应该是谁将它发送到浏览器。如果有类似 Push 或 ServerPush 的内容，说明你正在使用它！

# 还有更多...

服务器推送仅在 SSL 下工作。换句话说，如果你正在使用 GlassFish 5 并尝试运行这个菜谱，你的 URL 应该是这样的：

`https://localhost:8181/ch04-serverpush`

如果你错过了它，代码仍然可以工作，但使用 HTTP/1.0 意味着当代码请求`newPushBuilder`时，它将返回 null（不可用）：

```java
if (pb != null) {
   ...
}
```

# 参见

+   要获取这个菜谱的完整源代码，请检查[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter04/ch04-serverpush`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter04/ch04-serverpush)
