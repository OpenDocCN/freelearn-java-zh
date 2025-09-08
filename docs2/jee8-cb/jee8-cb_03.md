# 使用 JSON 和 RESTful 功能构建强大的服务

现在，使用 JSON 通过 REST 服务进行数据传输是 HTTP 协议中应用程序之间数据传输最常见的方法，这不是巧合——这是快速且容易实现的。它易于阅读，易于解析，并且使用 JSON-P，易于编码！

以下食谱将向您展示一些常见场景以及如何应用 Java EE 来处理它们。

本章涵盖了以下食谱：

+   使用 JAX-RS 构建服务器端事件

+   使用 JAX-RS 和 CDI 提高服务的能力

+   使用 JSON-B 简化数据和对象表示

+   使用 JSON-P 解析、生成、转换和查询 JSON 对象

# 使用 JAX-RS 和 CDI 构建服务器端事件

通常，Web 应用程序依赖于客户端发送的事件。所以，基本上，服务器只有在被要求时才会做些什么。

但是，随着互联网周围技术的演变（HTML5、移动客户端、智能手机等等），服务器端也必须进化。因此，这就产生了服务器端事件，即由服务器引发的事件（正如其名所示）。

通过这个食谱，您将学习如何使用服务器端事件来更新用户视图。

# 准备工作

首先添加 Java EE 依赖项：

```java
    <dependencies>
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```

# 如何做到这一点...

首先，我们构建一个 REST 端点来管理我们将要使用的服务器事件，并且为了使用 REST，我们应该首先正确地配置它：

```java
@ApplicationPath("webresources")
public class ApplicationConfig extends Application {

}
```

以下是一段相当大的代码块，但别担心，我们将将其拆分并理解每一部分：

```java
@Path("serverSentService")
@RequestScoped
public class ServerSentService {

    private static final Map<Long, UserEvent> POOL = 
    new ConcurrentHashMap<>();

    @Resource(name = "LocalManagedExecutorService")
    private ManagedExecutorService executor;

    @Path("start")
    @POST
    public Response start(@Context Sse sse) {

        final UserEvent process = new UserEvent(sse);

        POOL.put(process.getId(), process);
        executor.submit(process);

        final URI uri = UriBuilder.fromResource(ServerSentService.class).path
        ("register/{id}").build(process.getId());
        return Response.created(uri).build();
    }

    @Path("register/{id}")
    @Produces(MediaType.SERVER_SENT_EVENTS)
    @GET
    public void register(@PathParam("id") Long id,
            @Context SseEventSink sseEventSink) {
        final UserEvent process = POOL.get(id);

        if (process != null) {
            process.getSseBroadcaster().register(sseEventSink);
        } else {
            throw new NotFoundException();
        }
    }

    static class UserEvent implements Runnable {

        private final Long id;
        private final SseBroadcaster sseBroadcaster;
        private final Sse sse;

        UserEvent(Sse sse) {
            this.sse = sse;
            this.sseBroadcaster = sse.newBroadcaster();
            id = System.currentTimeMillis();
        }

        Long getId() {
            return id;
        }

        SseBroadcaster getSseBroadcaster() {
            return sseBroadcaster;
        }

        @Override
        public void run() {
            try {
                TimeUnit.SECONDS.sleep(5);
                sseBroadcaster.broadcast(sse.newEventBuilder().
                name("register").data(String.class, "Text from event " 
                                      + id).build());
                sseBroadcaster.close();
            } catch (InterruptedException e) {
                System.out.println(e.getMessage());
            }
        }
    }
}
```

这里，我们有一个用于管理 UI 并帮助我们更好地了解服务器中发生情况的 bean：

```java
@ViewScoped
@Named
public class SseBean implements Serializable {

    @NotNull
    @Positive
    private Integer countClient;

    private Client client;

    @PostConstruct
    public void init(){
        client = ClientBuilder.newClient();
    }

    @PreDestroy
    public void destroy(){
        client.close();
    }

    public void sendEvent() throws URISyntaxException, InterruptedException {
        WebTarget target = client.target(URI.create("http://localhost:8080/
                                                    ch03-sse/"));
        Response response = 
        target.path("webresources/serverSentService/start")
                .request()
                .post(Entity.json(""), Response.class);

        FacesContext.getCurrentInstance().addMessage(null,
                new FacesMessage("Sse Endpoint: " + 
                response.getLocation()));

        final Map<Integer, String> messageMap = new ConcurrentHashMap<>
        (countClient);
        final SseEventSource[] sources = new 
        SseEventSource[countClient];

        final String processUriString = 
        target.getUri().relativize(response.getLocation()).
        toString();
        final WebTarget sseTarget = target.path(processUriString);

        for (int i = 0; i < countClient; i++) {
            final int id = i;
            sources[id] = SseEventSource.target(sseTarget).build();
            sources[id].register((event) -> {
                final String message = event.readData(String.class);

                if (message.contains("Text")) {
                    messageMap.put(id, message);
                }
            });
            sources[i].open();
        }

        TimeUnit.SECONDS.sleep(10);

        for (SseEventSource source : sources) {
            source.close();
        }

        for (int i = 0; i < countClient; i++) {
            final String message = messageMap.get(i);

            FacesContext.getCurrentInstance().addMessage(null,
                    new FacesMessage("Message sent to client " + 
                                     (i + 1) + ": " + message));
        }
    }

    public Integer getCountClient() {
        return countClient;
    }

    public void setCountClient(Integer countClient) {
        this.countClient = countClient;
    }

}
```

最后，UI 是一个简单的 JSF 页面的代码：

```java
<h:body>
    <h:form>
        <h:outputLabel for="countClient" value="Number of Clients" />
        <h:inputText id="countClient" value="#{sseBean.countClient}" />

        <br />
        <h:commandButton type="submit" action="#{sseBean.sendEvent()}" 
         value="Send Events" />
    </h:form>
</h:body>
```

# 它是如何工作的...

我们从我们的 SSE 引擎开始，`ServerEvent` 类，以及一个 JAX-RS 端点——这些包含了我们需要为此食谱的所有方法。

让我们了解第一个：

```java
    @Path("start")
    @POST
    public Response start(@Context Sse sse) {

        final UserEvent process = new UserEvent(sse);

        POOL.put(process.getId(), process);
        executor.submit(process);

        final URI uri = UriBuilder.fromResource(ServerSentService.class).
        path("register/{id}").build(process.getId());
        return Response.created(uri).build();
    }
```

以下是一些主要点：

1.  首先，这个方法将创建并准备一个由服务器发送给客户端的事件。

1.  然后，刚刚创建的事件被放入一个名为 `POOL` 的 HashMap 中。

1.  然后，我们的事件被附加到一个 URI 上，该 URI 代表这个类中的另一个方法（详细信息将在下面提供）。

注意这个参数：

```java
@Context Sse sse
```

它从服务器上下文中带来了服务器端事件功能，并允许您按需使用它，当然，它通过 CDI 注入（是的，CDI 到处都是！）。

现在我们看到我们的 `register()` 方法：

```java
    @Path("register/{id}")
    @Produces(MediaType.SERVER_SENT_EVENTS)
    @GET
    public void register(@PathParam("id") Long id,
            @Context SseEventSink sseEventSink) {
        final UserEvent event = POOL.get(id);

        if (event != null) {
            event.getSseBroadcaster().register(sseEventSink);
        } else {
            throw new NotFoundException();
        }
    }
```

这是将事件发送到您的客户端的非常方法——检查 `@Produces` 注解；它使用新的媒体类型 `SERVER_SENT_EVENTS`。

引擎之所以能工作，多亏了这段小代码：

```java
@Context SseEventSink sseEventSink

...

event.getSseBroadcaster().register(sseEventSink);
```

`SseEventSink` 是由 Java EE 服务器管理的事件队列，并且通过上下文注入为您提供。

然后，您获取进程广播器并将其注册到这个接收器，这意味着该进程广播的任何内容都将通过 `SseEventSink` 由服务器发送。

现在我们检查我们的事件设置：

```java
    static class UserEvent implements Runnable {

        ...

        UserEvent(Sse sse) {
            this.sse = sse;
            this.sseBroadcaster = sse.newBroadcaster();
            id = System.currentTimeMillis();
        }

        ...

        @Override
        public void run() {
            try {
                TimeUnit.SECONDS.sleep(5);
                sseBroadcaster.broadcast(sse.newEventBuilder().
                name("register").data(String.class, "Text from event " 
                + id).build());
                sseBroadcaster.close();
            } catch (InterruptedException e) {
                System.out.println(e.getMessage());
            }
        }
    }
```

如果你注意这条线：

```java
this.sseBroadcaster = sse.newBroadcaster();
```

您会记得我们刚刚在上一个类中使用了这个广播器。在这里我们看到这个广播器是由服务器注入的 `Sse` 对象带来的。

此事件实现了 `Runnable` 接口，因此我们可以使用它与执行器（如前所述），一旦运行，你就可以向你的客户端广播：

```java
sseBroadcaster.broadcast(sse.newEventBuilder().name("register").
data(String.class, "Text from event " + id).build());
```

这正是发送给客户端的消息。这可能是你需要的任何消息。

对于此配方，我们使用了另一个类来与 `Sse` 交互。让我们突出最重要的部分：

```java
        WebTarget target = client.target(URI.create
        ("http://localhost:8080/ch03-sse/"));
        Response response = target.path("webresources/serverSentService
                                        /start")
                .request()
                .post(Entity.json(""), Response.class);
```

这是一段简单的代码，您可以使用它来调用任何 JAX-RS 端点。

最后，这个模拟客户端最重要的部分：

```java
        for (int i = 0; i < countClient; i++) {
            final int id = i;
            sources[id] = SseEventSource.target(sseTarget).build();
            sources[id].register((event) -> {
                final String message = event.readData(String.class);

                if (message.contains("Text")) {
                    messageMap.put(id, message);
                }
            });
            sources[i].open();
        }
```

每个广播的消息都在这里读取：

```java
final String message = messageMap.get(i);
```

可以是任何你想要的客户端，另一个服务，一个网页，一个移动客户端，或任何东西。

然后我们检查我们的 UI：

```java
<h:inputText id="countClient" value="#{sseBean.countClient}" />
...
<h:commandButton type="submit" action="#{sseBean.sendEvent()}" 
value="Send Events" />
```

我们使用 `countClient` 字段来填充客户端的 `countClient` 值，因此您可以随意使用尽可能多的线程。

# 更多...

重要的是要提到，SSE 不受 MS IE/Edge 网络浏览器支持，并且它的可扩展性不如 WebSockets。如果您想在桌面端实现全浏览器支持并且/或者更好的可扩展性（因此，不仅限于移动应用，还包括可以打开更多连接的实例的 Web 应用），那么应该考虑使用 **WebSockets**。幸运的是，标准 Java EE 自 7.0 起就支持 WebSockets。

# 参见

+   此配方的完整源代码位于 [`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter03/ch03-sse`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter03/ch03-sse)

# 使用 JAX-RS 和 CDI 提升服务能力

此配方将向您展示如何利用 CDI 和 JAX-RS 功能来减少编写强大服务的努力并降低复杂性。

# 准备工作

首先添加 Java EE 依赖项：

```java
    <dependencies>
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```

# 如何做到这一点...

1.  我们首先创建一个 `User` 类，将通过我们的服务进行管理：

```java
public class User implements Serializable{

    private String name;
    private String email;

    public User(){

    }

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    //DO NOT FORGET TO IMPLEMENT THE GETTERS AND SETTERS

}
```

1.  为了有多个 `User` 对象的来源，我们创建了一个 `UserBean` 类：

```java
@Stateless
public class UserBean {

    public User getUser(){
        long ts = System.currentTimeMillis();
        return new User("Bean" + ts, "user" + ts + 
                        "@eldermoraes.com"); 
    }
}
```

1.  最后，我们创建我们的 `UserService` 端点：

```java
@Path("userservice")
public class UserService implements Serializable{

    @Inject
    private UserBean userBean;

    private User userLocal;

    @Inject
    private void setUserLocal(){
        long ts = System.currentTimeMillis();
        userLocal = new User("Local" + ts, "user" + ts + 
                             "@eldermoraes.com"); 
    }

    @GET
    @Path("getUserFromBean")
    @Produces(MediaType.APPLICATION_JSON)
    public Response getUserFromBean(){
        return Response.ok(userBean.getUser()).build();
    }

    @GET
    @Path("getUserFromLocal")
    @Produces(MediaType.APPLICATION_JSON)
    public Response getUserFromLocal(){
        return Response.ok(userLocal).build();
    } 

}
```

1.  为了加载我们的 UI，我们有 `UserView` 类，它将在 UI 和服务之间充当控制器：

```java
@ViewScoped
@Named
public class UserView implements Serializable {

    public void loadUsers() {
        Client client = ClientBuilder.newClient();
        WebTarget target = client.target(URI.create
        ("http://localhost:8080/ch03-rscdi/"));
        User response = target.path("webresources/userservice/
                                     getUserFromBean")
                .request()
                .accept(MediaType.APPLICATION_JSON)
                .get(User.class);

        FacesContext.getCurrentInstance()
                .addMessage(null,
                        new FacesMessage("userFromBean: " + 
                                         response));

        response = target.path("webresources/userservice
                               /getUserFromLocal")
                .request()
                .accept(MediaType.APPLICATION_JSON)
                .get(User.class);

        FacesContext.getCurrentInstance()
                .addMessage(null,
                        new FacesMessage("userFromLocal: 
                                         " + response));
```

```java
        client.close();
    }

}
```

1.  我们添加一个简单的 JSF 页面，仅用于显示结果：

```java
 <h:body>
 <h:form>
 <h:commandButton type="submit" 
 action="#{userView.loadUsers()}" 
 value="Load Users" />
 </h:form>
 </h:body>
```

# 它是如何工作的...

我们使用了两种注入方式：

+   从 `UserBean`，当 `UserService` 附着到上下文

+   从 `UserService` 本身

从 `UserBean` 注入是最简单的方式：

```java
    @Inject
    private UserBean userBean;
```

从 `UserService` 本身注入也很简单：

```java
    @Inject
    private void setUserLocal(){
        long ts = System.currentTimeMillis();
        userLocal = new User("Local" + ts, "user" + ts + 
                             "@eldermoraes.com"); 
    }
```

这里，`@Inject` 的工作方式类似于 `@PostConstruct` 注解，区别在于服务器上下文中运行方法。但结果相当相同。

所有一切都已注入，现在只是获取结果的问题：

```java
response = target.path("webresources/userservice/getUserFromBean")
                .request()
                .accept(MediaType.APPLICATION_JSON)
                .get(User.class);

...

response = target.path("webresources/userservice/getUserFromLocal")
                .request()
                .accept(MediaType.APPLICATION_JSON)
                .get(User.class);
```

# 更多...

如您所见，JAX-RS 简化了大量的对象解析和表示：

```java
    @GET
    @Path("getUserFromBean")
    @Produces(MediaType.APPLICATION_JSON)
    public Response getUserFromBean(){
        userFromBean = userBean.getUser();
        return Response.ok(userFromBean).build();
    }

    @GET
    @Path("getUserFromLocal")
    @Produces(MediaType.APPLICATION_JSON)
    public Response getUserFromLocal(){
        return Response.ok(userLocal).build();
    }
```

通过返回一个 `Response` 对象并使用 `@Produces(MediaType.APPLICATION_JSON)`，您让框架承担了解析您的 `user` 对象到 JSON 表示的重任。只需几行代码就能节省大量精力！

您还可以使用生产者（`@Produces` 注解）注入用户。请参阅第一章（86071f26-42aa-43e2-8409-6feaed4759e0.xhtml）的 CDI 菜谱，*新特性和改进*，以获取更多详细信息。

# 相关内容

+   在 [`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter03/ch03-rscdi`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter03/ch03-rscdi) 查看此菜谱的完整源代码

# 使用 JSON-B 轻松表示数据和对象

此菜谱将向您展示如何使用新的 JSON-B API 的力量为您的数据表示提供一些灵活性，并帮助将您的对象转换为 JSON 消息。

# 准备工作

首先添加 Java EE 依赖项：

```java
    <dependencies>
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```

# 如何操作...

1.  我们首先创建一个具有一些自定义的 `User` 类（详情见后）：

```java
public class User {

    private Long id;

    @JsonbProperty("fullName")
    private String name;

    private String email;

    @JsonbTransient
    private Double privateNumber;

    @JsonbDateFormat(JsonbDateFormat.DEFAULT_LOCALE)
    private Date dateCreated;

    public User(Long id, String name, String email, 
                Double privateNumber, Date dateCreated) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.privateNumber = privateNumber;
        this.dateCreated = dateCreated;
    }

    private User(){
    }

    //DO NOT FORGET TO IMPLEMENT THE GETTERS AND SETTERS
}
```

1.  我们使用 `UserView` 将用户 JSON 返回到 UI：

```java
@ViewScoped
@Named
public class UserView implements Serializable{

    private String json;

    public void loadUser(){
        long now = System.currentTimeMillis();
        User user = new User(now, 
                "User" + now, 
                "user" + now + "@eldermoraes.com",
                Math.random(),
                new Date());

        Jsonb jb = JsonbBuilder.create();
        json = jb.toJson(user);
        try {
            jb.close();
        } catch (Exception ex) {
            System.out.println(ex.getMessage());
        }
    }

    public String getJson() {
        return json;
    }

    public void setJson(String json) {
        this.json = json;
    }
}
```

1.  我们只是添加一个 JSF 页面来显示结果：

```java
 <h:body>
 <h:form>
 <h:commandButton type="submit" action="#{userView.loadUser()}" 
  value="Load User" />

 <br />

 <h:outputLabel for="json" value="User JSON" />
 <br />
 <h:inputTextarea id="json" value="#{userView.json}" 
  style="width: 300px; height: 300px;" />
 </h:form>
 </h:body>
```

# 工作原理...

我们使用一些 JSON-B 注解来自定义我们的用户数据表示：

```java
    @JsonbProperty("fullName")
    private String name;
```

使用 `@JsonbProperty` 将字段名称更改为其他值：

```java
    @JsonbTransient
    private Double privateNumber;
```

当您想防止某些属性出现在 JSON 表示中时，使用 `@JsonbTransient`：

```java
    @JsonbDateFormat(JsonbDateFormat.DEFAULT_LOCALE)
    private Date dateCreated;
```

使用 `@JsonbDateFormat`，您可以使用 API 自动格式化您的日期。

然后我们使用我们的 UI 管理器来更新视图：

```java
    public void loadUser(){
        long now = System.currentTimeMillis();
        User user = new User(now, 
                "User" + now, 
                "user" + now + "@eldermoraes.com",
                Math.random(),
                new Date());

        Jsonb jb = JsonbBuilder.create();
        json = jb.toJson(user);
    }
```

# 相关内容

+   此菜谱的完整源代码位于 [`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter03/ch03-jsonb`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter03/ch03-jsonb)

# 使用 JSON-P 解析、生成、转换和查询 JSON 对象

处理 JSON 对象是您无法避免的活动。因此，如果您可以通过依赖一个强大且易于使用的框架来操作它——那就更好了！

此菜谱将向您展示如何使用 JSON-P 执行一些不同的操作，使用或生成 JSON 对象。

# 准备工作

首先添加 Java EE 依赖项：

```java
    <dependencies>
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```

# 如何操作...

1.  让我们创建一个 `User` 类来支持我们的操作：

```java
public class User {

    private String name;
    private String email;
    private Integer[] profiles;

    public User(String name, String email, Integer[] profiles) {
        this.name = name;
        this.email = email;
        this.profiles = profiles;
    }

    //DO NOT FORGET TO IMPLEMENT THE GETTERS AND SETTERS
}
```

1.  然后创建一个 `UserView` 类来执行所有 JSON 操作：

```java
@ViewScoped
@Named
public class UserView implements Serializable{

    private static final JsonBuilderFactory BUILDERFACTORY = 
    Json.createBuilderFactory(null);
    private final Jsonb jsonbBuilder = JsonbBuilder.create();

    private String fromArray;
    private String fromStructure;
    private String fromUser;
    private String fromJpointer;

    public void loadUserJson(){
        loadFromArray();
        loadFromStructure();
        loadFromUser();
    }

    private void loadFromArray(){
        JsonArray array = BUILDERFACTORY.createArrayBuilder()
                .add(BUILDERFACTORY.createObjectBuilder()
                        .add("name", "User1")
                        .add("email", "user1@eldermoraes.com"))
                .add(BUILDERFACTORY.createObjectBuilder()
                        .add("name", "User2")
                        .add("email", "user2@eldermoraes.com"))
                .add(BUILDERFACTORY.createObjectBuilder()
                        .add("name", "User3")
                        .add("email", "user3@eldermoraes.com")) 
                .build(); 
        fromArray = jsonbBuilder.toJson(array);
    }

    private void loadFromStructure(){
        JsonStructure structure = 
        BUILDERFACTORY.createObjectBuilder()
                .add("name", "User1")
                .add("email", "user1@eldermoraes.com")
                .add("profiles", BUILDERFACTORY.createArrayBuilder()
                        .add(BUILDERFACTORY.createObjectBuilder()
                                .add("id", "1")
                                .add("name", "Profile1"))
                        .add(BUILDERFACTORY.createObjectBuilder()
                                .add("id", "2")
                                .add("name", "Profile2")))
                .build();
        fromStructure = jsonbBuilder.toJson(structure);

        JsonPointer pointer = Json.createPointer("/profiles");
        JsonValue value = pointer.getValue(structure);
        fromJpointer = value.toString();
    }

    private void loadFromUser(){
        User user = new User("Elder Moraes", 
        "elder@eldermoraes.com", 
        new Integer[]{1,2,3});
        fromUser = jsonbBuilder.toJson(user);
    }

    //DO NOT FORGET TO IMPLEMENT THE GETTERS AND SETTERS
}
```

1.  然后我们创建一个 JSF 页面来显示结果：

```java
 <h:body>
 <h:form>
 <h:commandButton type="submit" action="#{userView.loadUserJson()}" 
 value="Load JSONs" />

 <br />

 <h:outputLabel for="fromArray" value="From Array" />
 <br />
 <h:inputTextarea id="fromArray" value="#{userView.fromArray}" 
 style="width: 300px; height: 150px" />
 <br />

 <h:outputLabel for="fromStructure" value="From Structure" />
 <br />
 <h:inputTextarea id="fromStructure" value="#{userView.fromStructure}" 
 style="width: 300px; height: 150px" />
 <br />

 <h:outputLabel for="fromUser" value="From User" />
 <br />
 <h:inputTextarea id="fromUser" value="#{userView.fromUser}" 
 style="width: 300px; height: 150px" />

  <br />
  <h:outputLabel for="fromJPointer" value="Query with JSON Pointer 
  (from JsonStructure Above)" />
  <br />
  <h:inputTextarea id="fromJPointer" 
   value="#{userView.fromJpointer}"  
  style="width: 300px; height: 100px" />
 </h:form>
 </h:body>
```

# 工作原理...

首先，`loadFromArray()` 方法：

```java
    private void loadFromArray(){
        JsonArray array = BUILDERFACTORY.createArrayBuilder()
                .add(BUILDERFACTORY.createObjectBuilder()
                        .add("name", "User1")
                        .add("email", "user1@eldermoraes.com"))
                .add(BUILDERFACTORY.createObjectBuilder()
                        .add("name", "User2")
                        .add("email", "user2@eldermoraes.com"))
                .add(BUILDERFACTORY.createObjectBuilder()
                        .add("name", "User3")
                        .add("email", "user3@eldermoraes.com")) 
                .build(); 
        fromArray = jsonbBuilder.toJson(array);
    }
```

它使用 `BuilderFactory` 和 `createArrayBuilder` 方法轻松构建 JSON 数组（每次调用 `createObjectBuilder` 都会创建另一个数组成员）。最后，我们使用 JSON-B 将其转换为 JSON 字符串：

```java
    private void loadFromStructure(){
        JsonStructure structure = BUILDERFACTORY.createObjectBuilder()
                .add("name", "User1")
                .add("email", "user1@eldermoraes.com")
                .add("profiles", BUILDERFACTORY.createArrayBuilder()
                        .add(BUILDERFACTORY.createObjectBuilder()
                                .add("id", "1")
                                .add("name", "Profile1"))
                        .add(BUILDERFACTORY.createObjectBuilder()
                                .add("id", "2")
                                .add("name", "Profile2")))
                .build();
        fromStructure = jsonbBuilder.toJson(structure);

        JsonPointer pointer = new JsonPointerImpl("/profiles");
        JsonValue value = pointer.getValue(structure);
        fromJpointer = value.toString();
    }
```

在这里，我们不是构建一个数组，而是构建一个单独的 JSON 结构。再次，我们使用 JSON-B 将 `JsonStructure` 转换为 JSON 字符串。

我们还利用了已准备好的 `JsonStructure` 并使用它通过 `JsonPointer` 对象查询用户配置文件：

```java
private void loadFromUser(){
        User user = new User("Elder Moraes", "elder@eldermoraes.com",
                    new Integer[]{1,2,3});
        fromUser = jsonbBuilder.toJson(user);
    }
```

这是最简单的：创建一个对象，并让 JSON-B 将其转换为 JSON 字符串。

# 相关内容

+   检查此菜谱的完整源代码，请访问[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter03/ch03-jsonp`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter03/ch03-jsonp)
