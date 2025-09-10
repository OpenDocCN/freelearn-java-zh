# 第四章：JSON 和异步处理

本章涵盖了全新的 JSR，JSR 353：Java API for JSON Processing [`jcp.org/en/jsr/detail?id=353`](http://jcp.org/en/jsr/detail?id=353)，以及与 Java EE 中不同服务和组件相关的 API 更新，这些更新为系统不同组件之间的异步交互提供了更好的支持。以下列表显示了本章涵盖的主题列表：

+   使用 Java 生成、解析和操作 JSON 数据

+   在 Servlet 3.1 中引入 NIO API

+   JAX-RS 2.0 的新特性

# 生成和解析 JSON 文档

JSON 格式被引入作为 XML 格式的替代品，当不需要 XML 的可扩展性和冗长性时，从而减轻复杂 XML 处理对资源的消耗，让较小的设备能够消费它们需要与之交互的不同服务产生的数据流或数据包。

在 Java EE 7 规范之前，Java 中没有标准 API 来处理 JSON 文档，而是有一些开源项目，如**google-gson**，[`code.google.com/p/google-gson`](https://code.google.com/p/google-gson) 和 **Jackson**，[`jackson.codehaus.org`](http://jackson.codehaus.org)，用于操作 JSON 文档。随着 Java EE 7 的发布以及 JSON-P 的加入，Java EE 中添加了一个标准 API，允许开发者以类似于 XML 处理 API 的方式标准地操作 JSON 文档。

JSON-P API 提供了两种解析 JSON 文档的方法，与解析 XML 文档可用的相同两种模型。下一节将解释的基于流的基于事件的解析和基于对象模型树的解析。

## JSON API 概述

以下表格显示了 JSON-P 的重要 API 段以及每个类的简要描述。与 JSON API 相关的类位于`javax.json`包下。后续章节将介绍如何使用这些类。

| 类别 | 描述和使用 |
| --- | --- |
| `JsonParser` | 一个使用事件模型的拉式解析器，用于解析 JSON 对象。 |
| `JsonGenerator` | 一个 JSON 流写入器，以流式方式将 JSON 对象写入输出源，如`OutputStream`和`Writer`。 |
| `JsonBuilder` | 以编程方式构建`JsonObject`和`JsonArray` |
| `JsonReader` | 从输入源读取`JsonObject`和`JsonArray` |
| `JsonWriter` | 将`JsonObject`和`JsonArray`写入输出源 |
| `JsonObject`和`JsonArray` | 用于存储`JSONObject`和数组结构 |
| `JsonString`和`JsonNumber` | 用于存储字符串和数值 |

`JSONObject` 是整个 JSON API 工具箱的入口点。对于以下每个对象，JSON API 都提供了一个工厂方法以及一个创建器来创建它们。例如，`Json.createParser` 和 `Json.createParserFactory` 可以用来创建一个 JSON 解析器。工厂可以被配置为生成定制的解析器，或者当需要多个解析器时，以减少创建解析器的性能开销，而 `createParser` 重载可以用来创建具有默认配置的 JSON 解析器。

## 使用基于事件的 API 操作 JSON 文档

当需要单方向、向前解析或生成 JSON 文档时，最好使用基于事件的 API。基于事件的 API 的工作方式类似于 XML 文档的 **StAX** 解析，但方式更为简单（因为 JSON 格式更为简单）。

## 生成 JSON 文档

当有一系列事件到达，并且需要将它们转换为 JSON 格式供其他消费 JSON 格式的处理器使用时，使用基于事件的 API 来生成 JSON 文档是最合适的。

以下示例代码展示了如何使用 `JsonGenerator` 生成 JSON 输出：

```java
public static void main(String[] args) {
        Map<String, Object>configs = new HashMap<String, Object>(1);
configs.put(JsonGenerator.PRETTY_PRINTING, true);
JsonGeneratorFactory factory = Json.createGeneratorFactory(configs);
JsonGeneratorgenerator = factory.createGenerator(System.out);

generator.writeStartObject()                    
            .write("title", "Getting Started with RESTful Web Services")  
                .write("type", "paperback")    
                .write("author", "Bhakti Mehta, Masoud Kalali")   
                .write("publisher", "Packt")            
                .write("publication year", "2013")     
                .write("edition",  "1")
        .writeEnd()                             
        .close();
    }
```

执行前面的代码会在标准输出中产生以下内容：

```java
{
    "title":" Getting Started with RESTful Web Services",
        "type":" paperback",
        "author":" Bhakti Mehta, Masoud Kalali",
        "publisher":"Packt",
        "edition":"1"
}
```

在代码的开始部分，创建的属性对象可以用来向 `JsonGenerator` 对象添加预期的行为指令。可以指定的指令因实现而异，但在这里使用 `JsonGenerator.PRETTY_PRINTING` 确保生成的 JSON 文档格式化且易于阅读。

### 注意

`JsonParser`、`JsonGenerator`、`JsonReader`、`JsonWriter` 可以在自动资源管理块中使用，例如：

```java
try (JsonGenerator generator = factory.createGenerator(System.out);) 
       {
        }
```

## 解析 JSON 文档

假设前一个示例的结果已保存到名为 `output.json` 的文件中，以下代码片段可以用来使用流解析器解析 `output.json`。

```java
FileInputStreambooksInputfile = new FileInputStream("output.json");
JsonParser parser = Json.createParser(booksInputfile);
            Event event = null;
            while(parser.hasNext()) {
                event = parser.next();
                if(event == Event.KEY_NAME&&"details".equals(parser.getString())) {
                    event = parser.next();
                    break;
                }
            }
            while(event != Event.END_OBJECT) {
                switch(event) {
                    case KEY_NAME: {
                        System.out.print(parser.getString());
                        System.out.print(" = ");
                        break;
                    }
                    case VALUE_NUMBER: {
                        if(parser.isIntegralNumber()) {
                          System.out.println(parser.getInt());
                        } else {
                          System.out.println(parser.getBigDecimal());
                        }
                       break;
                    }
                    case VALUE_STRING: {
                         System.out.println(parser.getString());
                        break;
                    }
                    default: {
                    }
                }
                event = parser.next();
            }
```

解析 JSON 文档时应处理的的事件类型如下列出：

+   `START_ARRAY`：指示 JSON 文档中的数组开始

+   `START_OBJECT`：指示对象的开始

+   `KEY_NAME`：键的名称

+   `VALUE_STRING`：当键的值是字符串时

+   `VALUE_NUMBER`：当值是数字时

+   `VALUE_NULL`：如果值是 null

+   `VALUE_FALSE`：如果值是布尔值 false

+   `VALUE_TRUE`：如果值是布尔值 true

+   `END_OBJECT`：到达对象的末尾

+   `END_ARRAY`：到达数组的末尾

## 使用 JSON 对象模型操作 JSON 文档

解析 JSON 的文档对象模型提供了与 XML DOM 解析相同的灵活性和限制。灵活性的列表包括但不限于，向前和向后遍历和操作 DOM 树；缺点或权衡在于解析速度和内存需求。

### 生成 JSON 文档

以下示例代码展示了如何使用构建器 API 生成 JSON 文档，然后将其产生的对象写入标准输出：

```java
Map<String, Object>configs = new HashMap<String, Object>();        
JsonBuilderFactory factory = Json.createBuilderFactory(configs);
JsonObject book= factory.createObjectBuilder()
.add("title", "Getting Started with RESTful Web Services")
.add("type", "paperback")
.add("author", "Bhakti Mehta, Masoud Kalali")
.add("publisher", "Packt")
.add("publication year", "2013")
.add("edition", "1")
.build();   
configs.put(JsonGenerator.PRETTY_PRINTING, true);
JsonWriter writer = Json.createWriterFactory(configs).createWriter(System.out);
writer.writeObject(book);
```

注意在创建 `JsonBuilderFactory` 时传递的配置属性。根据 JSON API 实现，可以向工厂传递不同的配置参数以生成定制的 `JsonBuilder` 对象。

生成的 JSON 输出看起来如下：

```java
{
    "title":" Getting Started with RESTful Web Services",
        "type":" paperback",
        "author":" Bhakti Mehta, Masoud Kalali",
        " publisher ":" Packt",
        " edition":"1"
}
```

### 解析 JSON 文档

使用对象模型解析 `JSONObject` 是直接的，从创建读取器对象并将输入文件/文档读取到 `JSONObject` 开始。在访问 `JSONObject` 后，可以遍历其原始和数组属性。

```java
Map<String, Object>configs = 
new HashMap<String, Object>(1);        
JsonReader  reader = 
Json.createReader(new FileInputStream("book.json"));        
JsonObject book=reader.readObject();       
        String title = book.getString("title");
int edition = book.getString("edition");
```

如示例代码所示，读取 `JSONObject` 的每个属性是通过类型化的 getter 完成的；例如 `getString`、`getInt`、`getNull`、`getBoolean` 等等。

## 何时使用流式 API 与对象 API

当你正在操作大型 JSON 文档，且不希望将其存储在内存中时，基于流的事件驱动 API 是有用的。在你在 JSON 文档的不同节点之间导航的情况下，对象模型 API 是有用的。

# 介绍 Servlet 3.1

Java EE 7 规范带来了 Servlet API 的更新规范，该规范解决了社区请求和行业需求的一些变化，包括但不限于以下列表中的变化：

+   将 NIO API 添加到 servlet 规范中

+   添加新的 WebSocket 协议升级支持等

下两个部分将详细介绍这些变化及其使用方法。

## NIO API 和 Servlet 3.1

Servlet 3 引入了异步处理传入请求的功能，其中请求可以在请求处理完成之前被放置在处理队列中，而不需要将线程绑定到请求。在 Servlet 3.1 中，又向前迈出一步，接收请求数据并写入响应可以以非阻塞、回调为导向的方式进行。

## 介绍 ReadListener 和 WriteListener

引入两个监听器，允许开发者在有可读数据传入时基本接收通知，而不是阻塞直到数据到达，并在可以写入输出而不被阻塞时接收通知。

`ReadListener` 接口，它提供在请求的 `InputStream` 中数据可用时的回调通知，如下所示，一个具有三个方法的简单接口，这些方法在代码片段之后进行描述。

```java
public interface ReadListener extends EventListener { 
 public void onDataAvailable(ServletRequest request); 
 public void onAllDataRead(ServletRequest request); 
 public void onError(Throwable t); 
}
```

+   `onDataAvailable`：在读取当前请求的所有数据后被调用。

+   `onAllDataRead`：由容器在第一次可能读取数据时调用。

+   `onError`：在处理请求时发生错误时调用。

当在 Servlet 的 `OutputStream` 中可能写入数据时，`WriteListener` 会提供回调通知，这是一个简单的两个方法接口，如下所示，并在之后进行描述。

```java
public interface WriteListener extends EventListener { 
public void onWritePossible(ServletResponse response); 
public void onError(Throwable t); 
}
```

当在 Servlet 的 `OutputStream` 中可能写入时，容器会调用 `onWritePossible` 方法。

当在 Servlet 的 `OutputStream` 中写入时遇到异常时，会调用 `onError`。

如何使用这两个监听器的示例代码包含在 Servlet 3.1 介绍部分的末尾。

## Servlet API 接口的更改

为了使用新引入的接口，Servlet API 进行了一些更改。这些更改如下：

+   在 `ServletOutputStream` 接口中：

    +   `isReady`: 该方法可用于确定是否可以无阻塞地写入数据

    +   `setWriteListener`: 指示 `ServletOutputStream` 在可能写入时调用提供的 `WriteListener`

+   在 `ServletInputStream` 接口中

    +   `isFinished`: 当从流中读取所有数据时返回 true，否则返回 false

    +   `isReady`: 如果可以无阻塞地读取数据，则返回 true，否则返回 false

    +   `setReadListener`: 指示 `ServletInputStream` 在可能读取时调用提供的 `ReadListener`

现在是时候看看非阻塞的 Servlet 3.1 API 是如何工作的了。以下代码片段显示了一个使用非阻塞 API 的异步 Servlet：

```java
@WebServlet(urlPatterns="/book-servlet", asyncSupported=true)
public class BookServlet extends HttpServlet {
    protected void doPost(HttpServletRequestreq, HttpServletResponse res)
            throws IOException, ServletException {
AsyncContext ac = req.startAsync();
ac.addListener(new AsyncListener() {
            public void onComplete(AsyncEvent event) throws IOException {
event.getSuppliedResponse().getOutputStream().print("Async Operation Completed");
            }
            public void onError(AsyncEvent event) {
System.out.println(event.getThrowable());
            }
            public void onStartAsync(AsyncEvent event) {
System.out.println("Async Operation Started");
            }
            public void onTimeout(AsyncEvent event) {
System.out.println("Async Operation Timedout");
            }
        });
ServletInputStream input = req.getInputStream();
ReadListenerreadListener =  new ReservationRequestReadListener(input, res, ac);
input.setReadListener(readListener);
    }
}
```

代码从声明 Servlet 并启用异步支持开始，通过在 `@WebServlet` 注解中指定 `asyncSupported=true` 来实现。

下一步是设置 `AsyncListener` 以处理 `AsyncEvent`s。调用 `AsyncContext` 之一将 `AsyncListener` 设置为 `AsyncContext#addListner` 重载。当 Servlet 的异步调用成功完成或因超时或错误结束时会接收 `AsyncEvents`。可以注册多个监听器，并且监听器会按照它们注册的顺序接收事件。

代码的最后部分将 servlet 的 `readListener` 设置为下面的 `ReadListener` 实现。当设置 `ReadListener` 时，读取传入请求的操作委托给 `ReservationRequestReadListener`。

```java
class ReservationRequestReadListener implements ReadListener {
    private ServletInputStream input = null;
    private HttpServletResponse response = null;
    private AsyncContext context = null;    
    private Queue queue = new LinkedBlockingQueue();

ReservationRequestReadListener(ServletInputStream in, HttpServletResponse r, AsyncContext c) {
this.input = in;
this.response = r;
this.context = c;
    }

    public void onDataAvailable() throws IOException {
StringBuildersb = new StringBuilder();
int read;
byte b[] = new byte[1024];
while (input.isReady() && (read = input.read(b)) != -1) {
String data = new String(b, 0, read);
sb.append(data);
        }
queue.add(sb.toString());
    }

public void onAllDataRead() throws IOException {
performBusinessOperation();
ServletOutputStream output = response.getOutputStream();
WriteListenerwriteListener = new ResponseWriteListener(output, queue, context);
output.setWriteListener(writeListener);
    }

public void onError(Throwable t) {
context.complete();        
    }
}
```

当容器有数据可读并且读取数据完成后，会调用 `ReservationRequestReadListener.onDataAvailable`。`onAllDataRead` 在可用的数据上执行业务操作，并将写入数据的 `ResponseWriteListener` 存储在队列中，然后发送回客户端。`ResponseWriteListener` 如下所示：

```java
class ResponseWriteListener implements WriteListener {
    private ServletOutputStream output = null;
    private Queue queue = null;
    private AsyncContext context = null;

ResponseWriteListener(ServletOutputStreamsos, Queue q, AsyncContext c) {
this.output = sos;
this.queue = q;
this.context = c;
    }

    public void onWritePossible() throws IOException {
        while (queue.peek() != null &&output.isReady()) {
            String data = (String) queue.poll();
            output.print(data);
        }
        if (queue.peek() == null) {
            context.complete();
        }
    }

    public void onError(final Throwable t) {
           context.complete();
           t.printStackTrace();
    }
}
```

当写入操作正常或致命地完成时，需要使用 `context.complete()` 方法关闭上下文以执行此操作。

### Servlet 3.1 的更多更改

除了非阻塞 IO 包含之外，Servlet 3.1 还引入了对协议升级的支持，以支持新的 WebSockets API。将 `upgrade` 方法添加到 `HttpServletRequest` 允许开发者在容器支持的情况下将通信协议升级到其他协议。

当发送升级请求时，应用程序决定执行升级操作，调用 `HttpServletRequest#upgrade(ProtocolHandler)`，然后应用程序像往常一样准备并发送适当的响应给客户端。此时，Web 容器将回滚所有 servlet 过滤器，并标记连接由协议处理器处理。

# JAX-RS 2.0 的新特性

JAX-RS 2.0 引入了几项新特性，这些特性与其他组件中提供的轻量级和异步处理特性保持一致。新特性包括以下内容：

+   客户端 API

+   常规配置

+   异步处理

+   过滤器/拦截器

+   超媒体支持

+   服务器端内容协商

从这个特性列表中，本节涵盖了异步处理，以及异步处理与过滤器/拦截器的相关性。

## 异步请求和响应处理

异步处理包含在 JAX-RS 2.0 的客户端和服务器端 API 中，以促进客户端和服务器组件之间的异步交互。以下列表显示了为支持此功能添加的新接口和类：

+   服务器端：

    +   `AsyncResponse`：一个可注入的 JAX-RS 异步响应，提供了异步服务器端响应处理的手段。

    +   `@Suspended`：`@Suspended` 指示容器 HTTP 请求处理应在副线程中发生。

    +   `CompletionCallback`：一个请求处理回调，接收请求处理完成事件。

    +   `ConnectionCallback`：异步请求处理生命周期回调，接收与连接相关的异步响应生命周期事件。

+   客户端：

    +   `InvocationCallback`：可以实现的回调，用于接收调用处理的异步处理事件。

    +   `Future`：允许客户端轮询异步操作的完成情况，或者阻塞并等待它

### 注意

Java SE 5 中引入的 `Future` 接口提供了两种不同的机制来获取异步操作的结果：首先是通过调用 `Future.get(…)` 变体，这将在结果可用或发生超时之前阻塞；第二种方式是通过调用 `isDone()` 和 `isCancelled()` 来检查完成情况，这两个布尔方法返回 Future 的当前状态。

以下示例代码展示了如何使用 JAX-RS 2 API 开发异步资源：

```java
@Path("/books/borrow")
@Stateless
public class BookResource {   
  @Context private ExecutionContextctx;
  @GET @Produce("application/json")
  @Asynchronous
  public void borrow() {
         Executors.newSingleThreadExecutor().submit( new Runnable() {
         public void run() { 
         Thread.sleep(10000);     
         ctx.resume("Hello async world!"); 
         } });  
         ctx.suspend();		
         return;
  } 
} 
```

`BookResource`是一个无状态会话 bean，它有一个`borrow()`方法。此方法使用`@Asynchronous`注解，将以“发射后不管”的方式工作。当通过`borrow()`方法的资源路径请求资源时，会启动一个新线程来处理请求的响应准备。该线程被提交给执行器执行，处理客户端请求的线程通过`ctx.suspend`释放以处理其他传入请求。当用于准备响应的工作线程完成准备响应后，它调用`ctx.resume`，让容器知道响应已准备好发送回客户端。如果在`ctx.suspend`之前调用`ctx.resume`（工作线程在执行达到`ctx.suspend`之前已准备好结果），则挂起将被忽略，并将结果发送给客户端。

可以使用以下片段中显示的`@Suspended`注解实现相同的功能：

```java
@Path("/books/borrow")
@Stateless
public class BookResource {   
  @GET @Produce("application/json")
  @Asynchronous
  public void borrow(@Suspended AsyncResponsear) {
  final String result = prepareResponse();
  ar.resume(result)  } 
}   
```

使用`@Suspended`更为简洁，因为它不涉及使用`ExecutionContext`方法来指示容器在工作线程（在这种情况下是`prepareResponse()`方法）完成时挂起和恢复通信线程。消费异步资源的客户端代码可以使用回调机制或代码级别的轮询。以下代码展示了如何通过`Future`接口进行轮询：

```java
Future<Book> future = client.target("("books/borrow/borrow")
               .request()
               .async()
               .get(Book.class);
try {
   Book book = future.get(30, TimeUnit.SECONDS);
} catch (TimeoutException ex) {
  System.err.println("Timeout occurred");
}
```

代码从向图书资源发送请求开始，然后`Future.get(…)`阻塞，直到从服务器返回响应或 30 秒超时。

异步客户端的另一个 API 是使用`InvocationCallback`。

## 过滤器和拦截器

过滤器和拦截器是 JAX-RS 2.0 中添加的两个新概念，允许开发者在请求和响应的入出之间进行拦截，以及在入出有效载荷的流级别上进行操作。

过滤器的工作方式与 Servlet 过滤器相同，并提供对入站和出站消息的访问，用于诸如身份验证/日志记录、审计等任务，而拦截器可以用于在有效载荷上执行诸如压缩/解压缩出站响应和入站请求等简单操作。

过滤器和拦截器是异步感知的，这意味着它们可以处理同步和异步通信。

# EJB 3.1 和 3.2 中的异步处理

在 Java EE 6 之前，Java EE 中唯一的异步处理设施是**JMS**（**Java 消息服务**）和**MDBs**（**消息驱动 Bean**），其中会话 bean 方法可以发送 JMS 消息来描述请求，然后让 MDB 以异步方式处理请求。使用 JMS 和 MDBs，会话 bean 方法可以立即返回，客户端可以使用方法返回的引用检查请求的完成情况，该引用用于处理某些 MDBs 的长时间运行操作。

上述解决方案效果良好，因为它已经工作了十年，但它并不容易使用，这也是 Java EE 6 引入 `@Asynchronous` 注解来注解会话 Bean 中的方法或整个会话 Bean 类作为异步的原因。`@Asynchronous` 可以放在类上，以标记该类中的所有方法为异步，也可以放在方法上，以标记特定方法为异步。

有两种类型的异步 EJB 调用，如下所述：

+   在第一个模型中，方法返回 `void`，并且没有容器提供的标准机制来检查方法调用的结果。这被称为**触发并忘记**机制。

+   在第二个模型中，容器通过方法调用返回的 `Future<?>` 对象提供了一个机制来检查调用的结果。这个机制被称为**调用后检查**。请注意，`Future` 是 Java SE 并发包的一部分。有了从方法返回的 Future 对象，客户端可以使用不同的 Future 方法（如 `isDone()` 和 `get(…)`)来检查调用的结果。

在深入示例代码或使用 `@Asynchronous` 之前，值得提一下，在 Java EE 6 中，`@Asynchronous` 只在完整配置文件中可用，而在 Java EE 7 中，该注解也被添加到 Web 配置文件中。

## 开发异步会话 Bean

以下列表显示了如何使用调用后检查异步 EJB 方法：

```java
@Stateless
@LocalBean
public class FTSSearch {

    @Asynchronous
    public Future<List<String>> search(String text, intdummyWait) {        
        List<String> books = null;
        try {
           books= performSearch(text,dummyWait);
        } catch (InterruptedException e) {
            //handling exception
        }
        return new AsyncResult<List<String>>(books);        
    }
    private List<String>performSearch(String content, intdummyWait) throws InterruptedException{
Thread.sleep(dummyWait);
return Arrays.asList(content);
    }
}
```

`@Stateless` 和 `@LocalBean` 是自解释的；它们将这个类标记为具有本地接口的无状态会话 Bean。

`search` 方法被注解为 `@Asynchronous`，这告诉容器方法调用应该在单独的分离线程中发生；当结果可用时，返回的 Future 对象的 `isDone()` 返回 true。

`search` 本身调用一个可能运行时间较长的 `performSearch` 方法，以获取客户端请求的长时间运行搜索操作的结果。

## 开发异步会话 Bean 的客户端 Servlet

现在无状态会话 Bean 已经开发完成，是时候开发一个客户端来访问会话 Bean 的业务方法了。在这种情况下，客户端是一个 Servlet，以下代码中包含了部分样板代码：

```java
@WebServlet(name = "FTSServlet", urlPatterns = {"/FTSServlet"})
public class FTSServlet extends HttpServlet {

    @EJB
FTSSearchftsSearch;

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        Future<List<String>>wsResult = ftsSearch.search("WebSockets", 5000);
        Future<List<String>>sseResult = ftsSearch.search("SSE", 1000);

        while (!sseResult.isDone()) {
            try {
Thread.sleep(500);
                //perform other tasks... e.g. show progress status
            } catch (InterruptedException ex) {
Logger.getLogger(FTSServlet.class.getName()).log(Level.SEVERE, null, ex);
            }
        }

response.setContentType("text/html;charset=UTF-8");
PrintWriter out = response.getWriter();
        try {
            /* TODO output your page here. You may use following sample code. */
out.println("<!DOCTYPE html>");
out.println("<html>");
out.println("<head>");
out.println("<title>Servlet d</title>");
out.println("</head>");
out.println("<body>");
out.println("<h1>SSE Search result: " + sseResult.get().get(0) + "</h1>");
            while (!wsResult.isDone()) {
                try {
Thread.sleep(500);
                } catch (InterruptedException ex) {
Logger.getLogger(FTSServlet.class.getName()).log(Level.SEVERE, null, ex);
                }
            }
out.println("<h1>WS Search result: " + wsResult.get().get(0) + "</h1>");
out.println("</body>");
out.println("</html>");
        } catch (InterruptedException ex) {
Logger.getLogger(FTSServlet.class.getName()).log(Level.SEVERE, null, ex);
        } catch (ExecutionException ex) {
Logger.getLogger(FTSServlet.class.getName()).log(Level.SEVERE, null, ex);
        } finally {
out.close();
        }
    }

}
```

从顶部开始，我们有 Servlet 声明注解、无状态 EJB 的注入以及 `get` 方法的实现。

在 `get` 方法的实现中，调用 EJB 的 `search` 方法时传递了两个不同的 `dummyTime` 来模拟等待。在 `search` 方法的调用之间，直到 Future 对象的 `isDone` 返回 true，客户端代码可以执行其他所需的操作。

现在已经描述了调用并稍后检查的模式，我们可以讨论另一种异步 EJB 调用模式，其中 EJB 业务方法返回`void`，并且没有容器提供的检查结果的方式。我们通常使用这些方法来触发一个长时间运行的任务，当前线程不需要等待它完成。

这种情况的例子是当图书馆新增一本电子书，全文搜索索引需要更新以包含新书时。在这种情况下，添加书籍的流程可以调用一个`@Asynchronous` EJB 方法在书籍注册期间以及上传到服务器仓库后对书籍进行索引。这样，注册过程就不需要等待全文搜索索引完成，而全文搜索索引则是在书籍添加到图书馆后立即开始。

# 摘要

本章，在展示一些真实世界示例之前的最后一章，讨论了 JSON 处理、异步 JAX-RS 资源，这些资源可以生成或消费 JSON 数据，同时讨论了 Servlet 3.1 中的新 NIO 支持。由于`@Asynchronous` EJB 现在包含在 Java EE 7 的 Web 配置文件中，我们讨论了该特性以及 Java EE7 中引入的其他新特性。下一章将展示如何将这些技术和 API 结合起来形成解决方案的真实世界示例。
