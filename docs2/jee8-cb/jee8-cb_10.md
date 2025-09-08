# 第十章：使用事件驱动编程来构建响应式应用程序

本章涵盖了以下食谱：

+   使用异步 servlet 构建响应式应用程序

+   使用事件和观察者构建响应式应用程序

+   使用 websockets 构建响应式应用程序

+   使用消息驱动的 bean 构建响应式应用程序

+   使用 JAX-RS 构建响应式应用程序

+   使用异步会话 bean 构建响应式应用程序

+   使用 lambda 和`CompletableFuture`来改进响应式应用程序

# 简介

响应式开发成为许多开发者会议、聚会、博客文章和其他无数内容来源（包括在线和离线）的热门话题。

但什么是响应式应用程序？嗯，有一个官方的定义包含在被称为**《响应式宣言》**（请参阅[`www.reactivemanifesto.org`](https://www.reactivemanifesto.org)以获取更多详细信息）的东西中。

简而言之，根据宣言，响应式系统是：

+   **响应性**：如果可能的话，系统会及时响应

+   **弹性**：面对失败时，系统保持响应性

+   **弹性**：系统在变化的工作负载下保持响应性

+   **消息驱动**：响应式系统依赖于异步消息传递来建立组件之间的边界，确保松散耦合、隔离和位置透明

因此，本章将向您展示如何使用 Java EE 8 功能来满足一个或多个那些响应式系统要求。

# 使用异步 servlet 构建响应式应用程序

Servlet 可能是最著名的 Java EE 技术之一（也许是最著名的）。实际上，servlet 在 J2EE 成为真正的规范之前就已经存在了。

这个食谱将向您展示如何异步使用 servlet。

# 准备工作

让我们先添加我们的 Java EE 8 依赖项：

```java
    <dependency>
        <groupId>javax</groupId>
        <artifactId>javaee-api</artifactId>
        <version>8.0</version>
        <scope>provided</scope>
    </dependency>
```

# 如何做到这一点...

1.  首先，我们创建一个`User` POJO：

```java
public class User implements Serializable{

    private Long id;
    private String name;

    public User(long id, String name){
        this.id = id;
        this.name = name;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
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

1.  然后，创建一个慢速的`UserBean`来返回`User`：

```java
@Stateless
public class UserBean {

    public User getUser(){
        long id = new Date().getTime();

        try {
            TimeUnit.SECONDS.sleep(5);
            return new User(id, "User " + id);
        } catch (InterruptedException ex) {
            System.err.println(ex.getMessage());
            return new User(id, "Error " + id);
        }
    }
}
```

1.  最后，创建我们的异步 servlet：

```java
@WebServlet(name = "UserServlet", urlPatterns = {"/UserServlet"}, asyncSupported = true)
public class UserServlet extends HttpServlet {

    @Inject
    private UserBean userBean;

    private final Jsonb jsonb = JsonbBuilder.create();

    @Override
    protected void doGet(HttpServletRequest req, 
    HttpServletResponse resp) throws ServletException, 
    IOException {
        AsyncContext ctx = req.startAsync();
        ctx.start(() -> {
            try (PrintWriter writer = 
            ctx.getResponse().getWriter()){
                writer.write(jsonb.toJson(userBean.getUser()));
            } catch (IOException ex) {
                System.err.println(ex.getMessage());
            }
            ctx.complete();
        });
    }

    @Override
    public void destroy() {
        try {
            jsonb.close();
        } catch (Exception ex) {
            System.err.println(ex.getMessage());
        }
    }

}
```

# 它是如何工作的...

在这里所有重要的事情中，我们应该从一个简单的注解开始：

```java
asyncSupported = true
```

这将告诉应用程序服务器这个 servlet 支持异步功能。顺便说一句，你需要在整个 servlet 链中（包括过滤器，如果有）使用它，否则应用程序服务器将无法工作。

由于 servlet 是由服务器实例化的，我们可以在其上注入其他上下文成员，例如我们的无状态 bean：

```java
@Inject
private UserBean userBean;
```

主要 servlet 方法持有实际的请求和响应引用，请求将给我们异步 API 的上下文引用：

```java
AsyncContext ctx = req.startAsync();
```

然后，您可以以前非阻塞的方式执行您之前的阻塞函数：

```java
ctx.start(() -> {
    ...
    ctx.complete();
});
```

# 参见

+   本食谱的完整源代码位于[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter10/ch10-async-servlet`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter10/ch10-async-servlet)。

# 使用事件和观察者构建响应式应用程序

事件和观察者是编写反应式代码的绝佳方式，无需过多思考，这要归功于 CDI 规范的出色工作。

这个配方将向您展示如何轻松地使用它来提高您应用程序的用户体验。

# 准备工作

让我们先添加我们的 Java EE 8 依赖项：

```java
    <dependency>
        <groupId>javax</groupId>
        <artifactId>javaee-api</artifactId>
        <version>8.0</version>
        <scope>provided</scope>
    </dependency>
```

# 如何实现...

1.  让我们先创建一个 `User` POJO：

```java
public class User implements Serializable{

    private Long id;
    private String name;

    public User(long id, String name){
        this.id = id;
        this.name = name;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
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

1.  然后，让我们创建一个具有事件和观察器功能的 REST 端点：

```java
@Stateless
@Path("asyncService")
public class AsyncService {

    @Inject
    private Event<User> event;

    private AsyncResponse response;

    @GET
    public void asyncService(@Suspended AsyncResponse response){
        long id = new Date().getTime();
        this.response = response;
        event.fireAsync(new User(id, "User " + id));
    }

```

```java
    public void onFireEvent(@ObservesAsync User user){
        response.resume(Response.ok(user).build());
    }
}
```

# 它是如何工作的...

首先，我们要求应用服务器为 `User` POJO 创建一个 `Event` 源：

```java
@Inject
private Event<User> event;
```

这意味着它将监听针对任何 `User` 对象触发的任何事件。因此，我们需要创建一个方法来处理它：

```java
public void onFireEvent(@ObservesAsync User user){
    response.resume(Response.ok(user).build());
}
```

因此，现在这种方法是合适的监听器。`@ObserversAsync` 注解保证了这一点。所以一旦异步事件被触发，它就会执行我们要求（或编码）的任何操作。

然后，我们创建了一个简单的异步端点来触发它：

```java
@GET
public void asyncService(@Suspended AsyncResponse response){
    long id = new Date().getTime();
    this.response = response;
    event.fireAsync(new User(id, "User " + id));
}
```

# 参见

+   这个配方的完整源代码在 [`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter10/ch10-event-observer`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter10/ch10-event-observer)。

# 使用 WebSocket 构建反应式应用程序

Websockets 是为您的应用程序创建解耦通信通道的绝佳方式。以异步方式执行甚至更好，更酷，因为这样可以实现非阻塞功能。

这个配方将展示如何实现。

# 准备工作

让我们先添加我们的 Java EE 8 依赖项：

```java
    <dependency>
        <groupId>javax</groupId>
        <artifactId>javaee-api</artifactId>
        <version>8.0</version>
        <scope>provided</scope>
    </dependency>
```

# 如何实现...

1.  我们需要的第一件事是我们的服务器端点：

```java
@Singleton
@ServerEndpoint(value = "/asyncServer")
public class AsyncServer {

    private final List<Session> peers = Collections.synchronizedList(new ArrayList<>());

    @OnOpen
    public void onOpen(Session peer){
        peers.add(peer);
    }

    @OnClose
    public void onClose(Session peer){
        peers.remove(peer);
    }

    @OnError
    public void onError(Throwable t){
        System.err.println(t.getMessage());
    }

    @OnMessage
    public void onMessage(String message, Session peer){
        peers.stream().filter((p) -> 
        (p.isOpen())).forEachOrdered((p) -> {
            p.getAsyncRemote().sendText(message + 
            " - Total peers: " + peers.size());
        });
    }
}
```

1.  然后，我们需要一个客户端与服务器进行通信：

```java
@ClientEndpoint
public class AsyncClient {

    private final String asyncServer = "ws://localhost:8080
    /ch10-async-websocket/asyncServer";

    private Session session;
    private final AsyncResponse response;

    public AsyncClient(AsyncResponse response) {
        this.response = response;
    }

    public void connect() {
        WebSocketContainer container = 
        ContainerProvider.getWebSocketContainer();
        try {
            container.connectToServer(this, new URI(asyncServer));
        } catch (URISyntaxException | DeploymentException | 
          IOException ex) {
            System.err.println(ex.getMessage());
        }

    }

    @OnOpen
    public void onOpen(Session session) {
        this.session = session;
    }

    @OnMessage
    public void onMessage(String message, Session session) {
        response.resume(message);
    }

    public void send(String message) {
        session.getAsyncRemote().sendText(message);
    }

    public void close(){
        try {
            session.close();
        } catch (IOException ex) {
            System.err.println(ex.getMessage());
        }
    }
}
```

1.  最后，我们需要一个简单的 REST 端点与客户端通信：

```java
@Stateless
@Path("asyncService")
public class AsyncService {

    @GET
    public void asyncService(@Suspended AsyncResponse response){
        AsyncClient client = new AsyncClient(response);
        client.connect();
        client.send("Message from client " + new Date().getTime());
        client.close();
    }
}
```

# 它是如何工作的...

我们服务器中的第一件重要的事情是这个注解：

```java
@Singleton
```

当然，我们必须确保我们只有一个服务器端点实例。这将确保所有 `peers` 都在同一个伞下管理。

让我们继续讨论 `peers`：

```java
private final List<Session> peers = Collections.synchronizedList
(new ArrayList<>());
```

存储它们的列表是同步的。这很重要，因为您将在迭代列表时添加/删除对等方，如果不保护它，事情可能会变得混乱。

所有默认的 WebSocket 方法都由应用服务器管理：

```java
@OnOpen
public void onOpen(Session peer){
    peers.add(peer);
}

@OnClose
public void onClose(Session peer){
    peers.remove(peer);
}

@OnError
public void onError(Throwable t){
    System.err.println(t.getMessage());
}

@OnMessage
public void onMessage(String message, Session peer){
    peers.stream().filter((p) -> (p.isOpen())).forEachOrdered((p) -> 
    {
        p.getAsyncRemote().sendText(message + " - Total peers: " 
        + peers.size());
    });
}
```

此外，让我们特别提一下我们 `onMessage` 方法上的代码：

```java
    @OnMessage
    public void onMessage(String message, Session peer){
        peers.stream().filter((p) -> (p.isOpen())).forEachOrdered((p)
        -> {
            p.getAsyncRemote().sendText(message + " - Total peers: "
            + peers.size());
        });
    }
```

我们只在连接打开时向对等方发送消息。

现在看看我们的客户端，我们有一个指向服务器 URI 的引用：

```java
private final String asyncServer = "ws://localhost:8080/
ch10-async-websocket/asyncServer";
```

注意，协议是 `ws`，这是针对 WebSocket 通信的特定协议。

然后，我们有一个方法可以打开与服务器端点的连接：

```java
public void connect() {
    WebSocketContainer container = 
    ContainerProvider.getWebSocketContainer();
    try {
        container.connectToServer(this, new URI(asyncServer));
    } catch (URISyntaxException | DeploymentException | IOException ex) {
        System.err.println(ex.getMessage());
    }
}
```

一旦我们从服务器收到消息确认，我们就可以对它采取行动：

```java
@OnMessage
public void onMessage(String message, Session session) {
    response.resume(message);
}
```

这个响应将出现在调用客户端的端点上：

```java
@GET
public void asyncService(@Suspended AsyncResponse response){
    AsyncClient client = new AsyncClient(response);
    client.connect();
    client.send("Message from client " + new Date().getTime());
}
```

我们正在传递客户端的引用，以便客户端可以使用它来在它上面写入消息。

# 参见

+   这个配方的完整源代码在 [`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter10/ch10-async-websocket`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter10/ch10-async-websocket)。

# 使用消息驱动豆构建反应式应用程序

Java 消息服务是 Java EE 最古老的 API 之一，它从一开始就是反应式的：只需阅读本章引言中链接的宣言。

本食谱将展示如何使用消息驱动豆，或称为 MDB，仅通过几个注解即可发送和消费异步消息。

# 准备工作

让我们先添加我们的 Java EE 8 依赖项：

```java
    <dependency>
        <groupId>javax</groupId>
        <artifactId>javaee-api</artifactId>
        <version>8.0</version>
        <scope>provided</scope>
    </dependency>
```

要检查 GlassFish 5 中队列设置的详细信息，请参阅第五章*使用消息服务进行异步通信*的食谱*企业架构的安全性*。

# 如何做到这一点...

1.  首先，我们创建一个`User` POJO：

```java
public class User implements Serializable{

    private Long id;
    private String name;

    public User(long id, String name){
        this.id = id;
        this.name = name;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
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

1.  然后，我们创建一个消息发送者：

```java
@Stateless
public class Sender {

    @Inject
    private JMSContext context;

    @Resource(lookup = "jms/JmsQueue")
    private Destination queue;

    public void send(User user){
        context.createProducer()
                .setDeliveryMode(DeliveryMode.PERSISTENT)
                .setDisableMessageID(true)
                .setDisableMessageTimestamp(true)
                .send(queue, user);
    }

}
```

1.  现在，我们创建一个消息消费者。这是我们自己的 MDB：

```java
@MessageDriven(activationConfig = {
    @ActivationConfigProperty(propertyName = "destinationLookup", 
    propertyValue = "jms/JmsQueue"),
    @ActivationConfigProperty(propertyName = "destinationType", 
    propertyValue = "javax.jms.Queue")
})
public class Consumer implements MessageListener{

    @Override
    public void onMessage(Message msg) {
        try {
            User user = msg.getBody(User.class);
            System.out.println("User: " + user);
        } catch (JMSException ex) {
            System.err.println(ex.getMessage());
        }
    }

}
```

1.  最后，我们创建一个端点，仅用于向队列发送模拟用户：

```java
@Stateless
@Path("mdbService")
public class MDBService {

    @Inject
    private Sender sender;

    public void mdbService(@Suspended AsyncResponse response){
        long id = new Date().getTime();
        sender.send(new User(id, "User " + id));
        response.resume("Message sent to the queue");
    }
}
```

# 它是如何工作的...

我们首先向应用程序服务器请求一个 JMS 上下文实例：

```java
@Inject
private JMSContext context;
```

我们还发送了一个我们想要与之工作的队列的引用：

```java
@Resource(lookup = "jms/JmsQueue")
private Destination queue;
```

然后，使用上下文，我们创建一个生产者来将消息发送到队列：

```java
context.createProducer()
        .setDeliveryMode(DeliveryMode.PERSISTENT)
        .setDisableMessageID(true)
        .setDisableMessageTimestamp(true)
        .send(queue, user);
```

注意这三个方法：

+   `setDeliveryMode`：此方法可以是`PERSISTENT`或`NON_PERSISTENT`。如果使用`PERSISTENT`，服务器将特别关注消息，不会丢失它。

+   `setDisableMessageID`：此选项用于创建`MessageID`，这会增加服务器创建和发送消息的努力，并增加其大小。此属性（`true` 或 `false`）向服务器提供提示，表明您不需要/使用它，因此它可以改进此过程。

+   `setDisableMessageTimestamp`：这与`setDisableMessageID`相同。

此外，请注意，我们正在向队列发送一个`User`实例。因此，您可以轻松地发送对象实例，而不仅仅是文本消息，只要它们实现了可序列化接口。

MDB 本身，或我们的消息消费者，基本上是几个注解和一个接口实现。

这里是其注解：

```java
@MessageDriven(activationConfig = {
    @ActivationConfigProperty(propertyName = "destinationLookup", 
    propertyValue = "jms/JmsQueue"),
    @ActivationConfigProperty(propertyName = "destinationType", 
    propertyValue = "javax.jms.Queue")
})
```

在这里，我们使用了两个属性：一个用于定义我们正在查找哪个队列（`destinationLookup`），另一个用于定义它确实是我们要使用的队列类型（`destinationType`）。

这里是其实施：

```java
@Override
public void onMessage(Message msg) {
    try {
        User user = msg.getBody(User.class);
        System.out.println("User: " + user);
    } catch (JMSException ex) {
        System.err.println(ex.getMessage());
    }
}
```

注意，从消息体中获取`User`实例很容易：

```java
User user = msg.getBody(User.class);
```

完全没有繁重的任务。

并且用于发送消息的端点非常简单。我们注入了`Sender`（这是一个无状态豆）：

```java
@Inject
private Sender sender;
```

然后，我们调用一个异步方法：

```java
public void mdbService(@Suspended AsyncResponse response){
    long id = new Date().getTime();
    sender.send(new User(id, "User " + id));
    response.resume("Message sent to the queue");
}
```

# 参见

+   在[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter10/ch10-mdb`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter10/ch10-mdb)中查看本食谱的完整源代码。

# 使用 JAX-RS 构建反应式应用程序

JAX-RS API 还有一些针对事件驱动编程的出色功能。本食谱将展示您如何使用异步调用者从请求中编写回调函数来生成响应。

# 准备工作

让我们先添加我们的 Java EE 8 依赖项：

```java
    <dependency>
        <groupId>javax</groupId>
        <artifactId>javaee-api</artifactId>
        <version>8.0</version>
        <scope>provided</scope>
    </dependency>
```

# 如何做到这一点...

1.  首先，我们创建一个`User` POJO：

```java
public class User implements Serializable{

    private Long id;
    private String name;

    public User(long id, String name){
        this.id = id;
        this.name = name;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
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

1.  在这里，我们定义了`UserBean`，它将作为远程端点：

```java
@Stateless
@Path("remoteUser")
public class UserBean {

    @GET
    public Response remoteUser() {
        long id = new Date().getTime();
        try {
            TimeUnit.SECONDS.sleep(5);
            return Response.ok(new User(id, "User " + id))
            .build();
        } catch (InterruptedException ex) {
            System.err.println(ex.getMessage());
            return Response.ok(new User(id, "Error " + id))
            .build();
        }
    }

}
```

1.  然后，最后，我们定义一个本地端点，它将消费远程端点：

```java
@Stateless
@Path("asyncService")
public class AsyncService {

    private Client client;
    private WebTarget target;

    @PostConstruct
    public void init() {
        client = ClientBuilder.newBuilder()
                .readTimeout(10, TimeUnit.SECONDS)
                .connectTimeout(10, TimeUnit.SECONDS)
                .build();
        target = client.target("http://localhost:8080/
        ch10-async-jaxrs/remoteUser");
    }

    @PreDestroy
    public void destroy(){
        client.close();
    }

    @GET
    public void asyncService(@Suspended AsyncResponse response){
        target.request().async().get(new 
        InvocationCallback<Response>() {
            @Override
            public void completed(Response rspns) {
                response.resume(rspns);
            }

            @Override
            public void failed(Throwable thrwbl) {
                response.resume(Response.status(Response.Status.
                INTERNAL_SERVER_ERROR).entity(thrwbl.getMessage())
                .build());
            }
        });

    }

}
```

# 它是如何工作的...

我们通过在 Bean 实例化时直接与远程端点建立通信来启动 Bean。这样做将避免在调用发生时稍后执行的开销：

```java
private Client client;
private WebTarget target;

@PostConstruct
public void init() {
     client = ClientBuilder.newBuilder()
            .readTimeout(10, TimeUnit.SECONDS)
            .connectTimeout(10, TimeUnit.SECONDS)
            .build();
    target = client.target("http://localhost:8080/
    ch10-async-jaxrs/remoteUser");
}
```

然后，我们在我们的异步调用器中创建了一个匿名`InvocationCallback`实现：

```java
        target.request().async().get(new InvocationCallback<Response>()   
        {
            @Override
            public void completed(Response rspns) {
                response.resume(rspns);
            }

            @Override
            public void failed(Throwable thrwbl) {
                System.err.println(thrwbl.getMessage());
            }
        });
```

这样，我们可以依赖`completed`和`failed`事件，并妥善处理它们。

# 参见

+   在[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter10/ch10-async-jaxrs`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter10/ch10-async-jaxrs)中查看这个菜谱的完整源代码。

# 使用异步会话 Bean 构建反应式应用程序

会话 Bean 也可以仅通过使用注解成为反应式和事件驱动的。这个菜谱将向您展示如何做到这一点。

# 准备工作

让我们先添加我们的 Java EE 8 依赖项：

```java
    <dependency>
        <groupId>javax</groupId>
        <artifactId>javaee-api</artifactId>
        <version>8.0</version>
        <scope>provided</scope>
    </dependency>
```

# 如何做...

1.  首先，我们创建一个`User` POJO：

```java
public class User implements Serializable{

    private Long id;
    private String name;

    public User(long id, String name){
        this.id = id;
        this.name = name;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
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

1.  然后，我们创建我们的异步会话 Bean：

```java
@Stateless
public class UserBean {

    @Asynchronous
    public Future<User> getUser(){
        long id = new Date().getTime();
        User user = new User(id, "User " + id);
        return new AsyncResult(user);
    }

    @Asynchronous
    public void doSomeSlowStuff(User user){
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException ex) {
            System.err.println(ex.getMessage());
        }
    }
}
```

1.  最后，我们创建了将调用 Bean 的端点：

```java
@Stateless
@Path("asyncService")
public class AsyncService {

    @Inject
    private UserBean userBean;

    @GET
    public void asyncService(@Suspended AsyncResponse response){
        try {
            Future<User> result = userBean.getUser();

            while(!result.isDone()){
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException ex) {
                    System.err.println(ex.getMessage());
                }
            }

            response.resume(Response.ok(result.get()).build());
        } catch (InterruptedException | ExecutionException ex) {
            System.err.println(ex.getMessage());
        }
    }
}
```

# 它是如何工作的...

让我们先检查会话 Bean 中的`getUser`方法：

```java
    @Asynchronous
    public Future<User> getUser(){
        long id = new Date().getTime();
        User user = new User(id, "User " + id);
        return new AsyncResult(user);
    }
```

一旦我们使用`@Asynchronous`注解，我们必须将其返回值转换为某种（在我们的情况下，`User`）的`Future`实例。

我们还创建了一个`void`方法来向您展示如何使用会话 Bean 创建非阻塞代码：

```java
    @Asynchronous
    public void doSomeSlowStuff(User user){
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException ex) {
            System.err.println(ex.getMessage());
        }
    }
```

最后，我们创建了我们的调用端点：

```java
    @GET
    public void asyncService(@Suspended AsyncResponse response){
        try {
            Future<User> result = userBean.getUser();

            while(!result.isDone()){
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException ex) {
                    System.err.println(ex.getMessage());
                }
            }

            response.resume(Response.ok(result.get()).build());
        } catch (InterruptedException | ExecutionException ex) {
            System.err.println(ex.getMessage());
        }
    }
```

由于`getUser`返回`Future`，我们可以处理异步状态检查。一旦完成，我们将结果写入响应（也是异步的）。

# 参见

+   在[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter10/ch10-async-bean`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter10/ch10-async-bean)中查看这个菜谱的完整源代码。

# 使用 lambda 和 CompletableFuture 来提高反应式应用程序

Java 语言一直以冗长著称。但自从 lambda 出现以来，这个问题已经大大改善。

我们可以使用 lambda 表达式，并引入`CompletableFuture`来提高不仅编码，而且反应式应用程序的行为。这个菜谱将向您展示如何做到这一点。

# 准备工作

让我们先添加我们的 Java EE 8 依赖项：

```java
    <dependency>
        <groupId>javax</groupId>
        <artifactId>javaee-api</artifactId>
        <version>8.0</version>
        <scope>provided</scope>
    </dependency>
```

# 如何做...

1.  首先，我们创建一个`User` POJO：

```java
public class User implements Serializable{

    private Long id;
    private String name;

    public User(long id, String name){
        this.id = id;
        this.name = name;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
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

1.  然后，我们调用`UserBean`以返回一个`User`实例：

```java
@Stateless
public class UserBean {

    public User getUser() {
        long id = new Date().getTime();
        try {
            TimeUnit.SECONDS.sleep(5);
            return new User(id, "User " + id);
        } catch (InterruptedException ex) {
            System.err.println(ex.getMessage());
            return new User(id, "Error " + id);
        }
    }

}
```

1.  最后，我们创建一个异步端点来调用 Bean：

```java
@Stateless
@Path("asyncService")
public class AsyncService {

    @Inject
    private UserBean userBean;

    @GET
    public void asyncService(@Suspended AsyncResponse response) 
    {
        CompletableFuture
                .supplyAsync(() -> userBean.getUser())
                .thenAcceptAsync((u) -> {
                    response.resume(Response.ok(u).build());
                }).exceptionally((t) -> {
                    response.resume(Response.status
                    (Response.Status.
                    INTERNAL_SERVER_ERROR).entity(t.getMessage())
                    .build());
                    return null;
                });
    }
}
```

# 它是如何工作的...

我们基本上使用两个`CompletableFuture`方法：

+   `supplyAsync`：这将启动对您放入其中的任何内容的异步调用。我们放入一个 lambda 调用。

+   `thenAcceptAsync`：一旦异步过程完成，返回值将在这里。多亏了 lambda，我们可以将这个返回值称为`u`（可以是任何我们想要的）。然后，我们用它来写入异步响应。

# 参见

+   查看此菜谱的完整源代码，请访问[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter10/ch10-completable-future`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter10/ch10-completable-future).
