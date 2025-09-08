# 第九章：在企业上下文中使用多线程

本章涵盖了以下配方：

+   使用返回结果构建异步任务

+   使用事务处理异步任务

+   检查异步任务的状况

+   使用返回结果构建管理线程

+   使用返回结果调度异步任务

+   使用注入代理进行异步任务

# 简介

**线程**是大多数软件项目中的常见问题，无论涉及哪种语言或其他技术。当谈到企业应用时，事情变得更加重要，有时也更难。

在某些线程中犯的一个错误可能会影响整个系统，甚至整个基础设施。想想看，一些资源永远不会释放，内存消耗永远不会停止增加，等等。

Java EE 环境有一些处理这些以及其他许多挑战的出色功能，本章将向您展示其中的一些。

# 使用返回结果构建异步任务

如果你从未处理过异步任务，你将面临的一个首要挑战是：如果你不知道执行何时结束，你该如何从异步任务中返回结果？

好吧，这个配方会告诉你如何做。`AsyncResponse` 赢了！

# 准备工作

让我们首先添加我们的 Java EE 8 依赖项：

```java
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何做到这一点...

1.  首先，我们创建一个 `User` POJO：

```java
package com.eldermoraes.ch09.async.result;

/**
 *
 * @author eldermoraes
 */
public class User {

    private Long id;
    private String name;

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

    public User(Long id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" + "id=" + id + ", name="
                     + name + '}';
    }
}
```

1.  然后我们创建 `UserService` 来模拟一个 *远程* 慢速端点：

```java
@Stateless
@Path("userService")
public class UserService {

    @GET
    public Response userService(){
        try {
            TimeUnit.SECONDS.sleep(5);
            long id = new Date().getTime();
            return Response.ok(new User(id, "User " + id)).build();
        } catch (InterruptedException ex) {
            return 
            Response.status(Response.Status.INTERNAL_SERVER_ERROR)
            .entity(ex).build();
        }
    }
}
```

1.  现在我们创建一个异步客户端，它将到达该端点并获取结果：

```java
@Stateless
public class AsyncResultClient {

    private Client client;
    private WebTarget target;

    @PostConstruct
    public void init() {
        client = ClientBuilder.newBuilder()
                .readTimeout(10, TimeUnit.SECONDS)
                .connectTimeout(10, TimeUnit.SECONDS)
                .build();
        target = client.target("http://localhost:8080/
                 ch09-async-result/userService");
    }

    @PreDestroy
    public void destroy(){
        client.close();
    }

    public CompletionStage<Response> getResult(){
        return 
        target.request(MediaType.APPLICATION_JSON).rx().get();
    }

}
```

1.  最后，我们创建一个服务（端点），它将使用客户端将结果写入响应：

```java
@Stateless
@Path("asyncService")
public class AsyncService {

    @Inject
    private AsyncResultClient client;

    @GET
    public void asyncService(@Suspended AsyncResponse response)
    {
        try{
            client.getResult().thenApply(this::readResponse)
            .thenAccept(response::resume);
        } catch(Exception e){
            response.resume(Response.status(Response.Status.
            INTERNAL_SERVER_ERROR).entity(e).build());
        }
    }

    private String readResponse(Response response) {
        return response.readEntity(String.class);
    }
}
```

要运行此示例，只需将其部署到 GlassFish 5，并在您的浏览器中打开此 URL：

`http://localhost:8080/ch09-async-result/asyncService`

# 它是如何工作的...

首先，我们的远程端点正在创建 `User` 并将其转换为响应实体：

```java
return Response.ok(new User(id, "User " + id)).build();
```

因此，毫不费力，您的 `User` 现在已经是一个 JSON 对象。

现在让我们看看 `AsyncResultClient` 中的关键方法：

```java
    public CompletionStage<Response> getResult(){
        return target.request(MediaType.APPLICATION_JSON).rx().get();
    }
```

`rx()` 方法是 Java EE 8 中引入的响应式客户端 API 的一部分。我们将在下一章中更详细地讨论响应式编程。它基本上返回 `CompletionStageInvoker`，这将允许你获取 `CompletionStage<Response>`（此方法的返回值）。

换句话说，这是一段异步/非阻塞代码，它从远程端点获取结果。

注意，我们使用 `@Stateless` 注解与此客户端一起，这样我们就可以将其注入到我们的主要端点：

```java
    @Inject
    private AsyncResultClient client;
```

这是我们的异步方法来写入响应：

```java
    @GET
    public void asyncService(@Suspended AsyncResponse response) {
        client.getResult().thenApply(this::readResponse)
        .thenAccept(response::resume);
    }
```

注意，这是一个 `void` 方法。它不返回任何内容，因为它会将结果返回给回调。

`@Suspended` 注解与 `AsyncResponse` 结合使用，将在处理完成后恢复响应，这是因为我们使用了美丽的一行，Java 8 风格的代码：

```java
client.getResult().thenApply(this::readResponse)
.thenAccept(response::resume);
```

在深入细节之前，让我们先澄清我们的本地 `readResponse` 方法：

```java
    private String readResponse(Response response) {
        return response.readEntity(String.class);
    }
```

它只是读取嵌入在`Response`中的`User`实体并将其转换为`String`对象（一个 JSON 字符串）。

这一行代码的另一种写法可以是这样的：

```java
        client.getResult()
                .thenApply(r -> readResponse(r))
                .thenAccept(s -> response.resume(s));
```

但第一种方法更简洁、更简洁、更有趣！

关键在于`AsyncResponse`对象的`resume`方法。它将响应写入回调并返回给请求者。

# 另请参阅

+   本菜谱的完整源代码位于[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter09/ch09-async-result`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter09/ch09-async-result)。

# 使用事务与异步任务

使用异步任务可能已经是一个挑战：如果你需要添加一些特色并添加一个事务怎么办？

通常，事务意味着像*代码阻塞*这样的东西。将两个对立的概念结合起来不是有点尴尬吗？嗯，不是！它们可以很好地一起工作，就像这个菜谱将向您展示的那样。

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

1.  让我们先创建一个`User` POJO：

```java
public class User {

    private Long id;
    private String name;

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

    public User(Long id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" + "id=" + id + ",
                     name=" + name + '}';
    } 
}
```

1.  这里是一个将返回`User`的慢速 bean：

```java
@Stateless
public class UserBean {

    public User getUser(){
        try {
            TimeUnit.SECONDS.sleep(5);
            long id = new Date().getTime();
            return new User(id, "User " + id);
        } catch (InterruptedException ex) {
            System.err.println(ex.getMessage());
            long id = new Date().getTime();
            return new User(id, "Error " + id);
        }
    }
}
```

1.  现在我们创建一个要执行的任务，该任务将使用一些事务相关的内容返回`User`：

```java
public class AsyncTask implements Callable<User> {

    private UserTransaction userTransaction;
    private UserBean userBean;

    @Override
    public User call() throws Exception {
        performLookups();
        try {
            userTransaction.begin();
            User user = userBean.getUser();
            userTransaction.commit();
            return user;
        } catch (IllegalStateException | SecurityException | 
          HeuristicMixedException | HeuristicRollbackException | 
          NotSupportedException | RollbackException | 
          SystemException e) {
            userTransaction.rollback();
            return null;
        }
    }

    private void performLookups() throws NamingException{
        userBean = CDI.current().select(UserBean.class).get();
        userTransaction = CDI.current()
        .select(UserTransaction.class).get();
    }    
}
```

1.  最后，这里是使用任务将结果写入响应的服务端点：

```java
@Path("asyncService")
@RequestScoped
public class AsyncService {

    private AsyncTask asyncTask;

    @Resource(name = "LocalManagedExecutorService")
    private ManagedExecutorService executor; 

    @PostConstruct
    public void init(){
        asyncTask = new AsyncTask();
    }

    @GET
    public void asyncService(@Suspended AsyncResponse response){

        Future<User> result = executor.submit(asyncTask);

        while(!result.isDone()){
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException ex) {
                System.err.println(ex.getMessage());
            }
        }

        try {
            response.resume(Response.ok(result.get()).build());
        } catch (InterruptedException | ExecutionException ex) {
            System.err.println(ex.getMessage());
            response.resume(Response.status(Response
            .Status.INTERNAL_SERVER_ERROR)
            .entity(ex.getMessage()).build());
        }

    }
}
```

要尝试此代码，只需将其部署到 GlassFish 5 并打开此 URL：

`http://localhost:8080/ch09-async-transaction/asyncService`

# 它是如何工作的...

魔法发生在`AsyncTask`类中，我们将首先看看`performLookups`方法：

```java
    private void performLookups() throws NamingException{
        Context ctx = new InitialContext();
        userTransaction = (UserTransaction) 
        ctx.lookup("java:comp/UserTransaction");
        userBean = (UserBean) ctx.lookup("java:global/
        ch09-async-transaction/UserBean");
    }
```

它将提供来自应用服务器的`UserTransaction`和`UserBean`实例。然后你可以放松并依赖为你已经实例化的东西。

因为我们的任务实现了一个需要实现`call()`方法的`Callable<V>`对象：

```java
    @Override
    public User call() throws Exception {
        performLookups();
        try {
            userTransaction.begin();
            User user = userBean.getUser();
            userTransaction.commit();
            return user;
        } catch (IllegalStateException | SecurityException | 
                HeuristicMixedException | HeuristicRollbackException 
                | NotSupportedException | RollbackException | 
                SystemException e) {
            userTransaction.rollback();
            return null;
        }
    }
```

你可以将`Callable`看作是一个返回结果的`Runnable`接口。

我们的事务代码存放在这里：

```java
            userTransaction.begin();
            User user = userBean.getUser();
            userTransaction.commit();
```

如果有任何问题，我们有以下情况：

```java
        } catch (IllegalStateException | SecurityException | 
           HeuristicMixedException | HeuristicRollbackException 
           | NotSupportedException | RollbackException | 
           SystemException e) {
            userTransaction.rollback();
            return null;
        }
```

现在我们将查看`AsyncService`。首先，我们有一些声明：

```java
    private AsyncTask asyncTask;

    @Resource(name = "LocalManagedExecutorService")
    private ManagedExecutorService executor; 

    @PostConstruct
    public void init(){
        asyncTask = new AsyncTask();
    }
```

我们要求容器给我们一个`ManagedExecutorService`的实例，它负责在企业上下文中执行任务。

然后我们调用一个`init()`方法，bean 被构建（`@PostConstruct`）。这实例化了任务。

现在我们有了任务执行：

```java
    @GET
    public void asyncService(@Suspended AsyncResponse response){

        Future<User> result = executor.submit(asyncTask);

        while(!result.isDone()){
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException ex) {
                System.err.println(ex.getMessage());
            }
        }

        try {
            response.resume(Response.ok(result.get()).build());
        } catch (InterruptedException | ExecutionException ex) {
            System.err.println(ex.getMessage());
            response.resume(Response.status(Response.
            Status.INTERNAL_SERVER_ERROR)
            .entity(ex.getMessage()).build());
        }

    }
```

注意，执行器返回`Future<User>`：

```java
Future<User> result = executor.submit(asyncTask);
```

这意味着这个任务将被异步执行。然后我们检查其执行状态，直到它完成：

```java
        while(!result.isDone()){
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException ex) {
                System.err.println(ex.getMessage());
            }
        }
```

一旦完成，我们就将其写入异步响应：

```java
response.resume(Response.ok(result.get()).build());
```

# 另请参阅

+   本菜谱的完整源代码位于[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter09/ch09-async-transaction`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter09/ch09-async-transaction)。

# 检查异步任务的状态

除了执行异步任务，这开辟了许多可能性之外，有时获取这些任务的状态是有用且必要的。

例如，你可以用它来检查每个任务阶段的耗时。你也应该考虑日志和监控。

这个菜谱将向你展示一个简单的方法来做这件事。

# 准备工作

首先，添加我们的 Java EE 8 依赖项：

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
public class User {

    private Long id;
    private String name;

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

    public User(Long id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" + "id=" + id + ", 
        name=" + name + '}';
    }
}
```

1.  然后我们创建一个慢速的 Bean 来返回`User`：

```java
public class UserBean {

    public User getUser(){
        try {
            TimeUnit.SECONDS.sleep(5);
            long id = new Date().getTime();
            return new User(id, "User " + id);
        } catch (InterruptedException ex) {
            long id = new Date().getTime();
            return new User(id, "Error " + id);
        }
    }
}
```

1.  现在我们创建一个托管任务，以便我们可以监控它：

```java
@Stateless
public class AsyncTask implements Callable<User>, ManagedTaskListener {

    private final long instantiationMili = new Date().getTime();

    private static final Logger LOG = Logger.getAnonymousLogger();

    @Override
    public User call() throws Exception {
        return new UserBean().getUser();
    }

    @Override
    public void taskSubmitted(Future<?> future, 
    ManagedExecutorService mes, Object o) {
        long mili = new Date().getTime();
        LOG.log(Level.INFO, "taskSubmitted: {0} - 
        Miliseconds since instantiation: {1}",
        new Object[]{future, mili - instantiationMili});
    }

    @Override
    public void taskAborted(Future<?> future, 
    ManagedExecutorService mes, Object o, Throwable thrwbl) 
    {
        long mili = new Date().getTime();
        LOG.log(Level.INFO, "taskAborted: {0} - 
        Miliseconds since instantiation: {1}", 
        new Object[]{future, mili - instantiationMili});
    }

    @Override
    public void taskDone(Future<?> future, 
    ManagedExecutorService mes, Object o, 
    Throwable thrwbl) {
        long mili = new Date().getTime();
        LOG.log(Level.INFO, "taskDone: {0} -
        Miliseconds since instantiation: {1}", 
        new Object[]{future, mili - instantiationMili});
    }

    @Override
    public void taskStarting(Future<?> future, 
    ManagedExecutorService mes, Object o) {
        long mili = new Date().getTime();
        LOG.log(Level.INFO, "taskStarting: {0} - 
        Miliseconds since instantiation: {1}", 
        new Object[]{future, mili - instantiationMili});
    }

}
```

1.  最后，我们创建一个服务端点来执行我们的任务并返回其结果：

```java
@Stateless
@Path("asyncService")
public class AsyncService {

    @Resource
    private ManagedExecutorService executor;

    @GET
    public void asyncService(@Suspended AsyncResponse response) {
        int i = 0;

        List<User> usersFound = new ArrayList<>();
        while (i < 4) {
            Future<User> result = executor.submit(new AsyncTask());

            while (!result.isDone()) {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException ex) {
                    System.err.println(ex.getMessage());
                }
            }

            try {
                usersFound.add(result.get());
            } catch (InterruptedException | ExecutionException ex) {
                System.err.println(ex.getMessage());
            }

            i++;
        }

        response.resume(Response.ok(usersFound).build());
    }

}
```

要尝试这段代码，只需将其部署到 GlassFish 5 并打开此 URL：

`http://localhost:8080/ch09-task-status/asyncService`

# 它是如何工作的...

如果你已经完成了上一个菜谱，你将已经熟悉了`Callable`任务，所以在这里我不会给出更多细节。但现在，我们正在使用`Callable`和`ManagedTaskListener`接口来实现我们的任务。第二个接口提供了检查任务状态的所有方法：

```java
    @Override
    public void taskSubmitted(Future<?> future, 
    ManagedExecutorService mes, Object o) {
        long mili = new Date().getTime();
        LOG.log(Level.INFO, "taskSubmitted: {0} - 
        Miliseconds since instantiation: {1}", 
        new Object[]{future, mili - instantiationMili});
    }

    @Override
    public void taskAborted(Future<?> future, 
    ManagedExecutorService mes, Object o, Throwable thrwbl) {
        long mili = new Date().getTime();
        LOG.log(Level.INFO, "taskAborted: {0} - 
        Miliseconds since instantiation: {1}", 
        new Object[]{future, mili - instantiationMili});
    }

    @Override
    public void taskDone(Future<?> future, 
    ManagedExecutorService mes, Object o, Throwable thrwbl) {
        long mili = new Date().getTime();
        LOG.log(Level.INFO, "taskDone: {0} - 
        Miliseconds since instantiation: {1}", 
        new Object[]{future, mili - instantiationMili});
    }

    @Override
    public void taskStarting(Future<?> future, 
    ManagedExecutorService mes, Object o) {
        long mili = new Date().getTime();
        LOG.log(Level.INFO, "taskStarting: {0} - 
        Miliseconds since instantiation: {1}", 
        new Object[]{future, mili - instantiationMili});
    }
```

最好的部分是，你不需要调用任何一个——`ManagedExecutorService`（将在下一节中解释）会为你做这件事。

最后，我们有`AsyncService`。第一个声明是为我们的执行器：

```java
    @Resource
    private ManagedExecutorService executor;
```

在服务本身中，我们正在从我们的异步任务中获取四个用户：

```java
        List<User> usersFound = new ArrayList<>();
        while (i < 4) {
            Future<User> result = executor.submit(new AsyncTask());

            while (!result.isDone()) {
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException ex) {
                    System.err.println(ex.getMessage());
                }
            }

            try {
                usersFound.add(result.get());
            } catch (InterruptedException | ExecutionException ex) {
                System.err.println(ex.getMessage());
            }

            i++;
        }
```

一旦完成，它将被写入异步响应：

```java
response.resume(Response.ok(usersFound).build());
```

现在，如果你查看你的服务器日志输出，将会有来自`ManagedTaskListener`接口的消息。

# 参见

+   在[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter09/ch09-task-status`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter09/ch09-task-status)查看这个菜谱的完整源代码。

# 使用返回结果的托管线程构建

有时候你需要改进你看待你正在使用的线程的方式；也许是为了改进你的日志功能，也许是为了管理它们的优先级。如果你也能从它们那里获取结果，那就太好了。这个菜谱将向你展示如何做到这一点。

# 准备工作

首先，添加我们的 Java EE 8 依赖项：

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
public class User {

    private Long id;
    private String name;

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

    public User(Long id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" + "id=" + id + ",
        name=" + name + '}';
    } 
}
```

1.  然后，我们创建一个慢速的 Bean 来返回`User`：

```java
@Stateless
public class UserBean {

    @GET
    public User getUser(){
        try {
            TimeUnit.SECONDS.sleep(5);
            long id = new Date().getTime();
            return new User(id, "User " + id);
        } catch (InterruptedException ex) {
            long id = new Date().getTime();
            return new User(id, "Error " + id);
        }
    }
}
```

1.  最后，我们创建一个端点来获取任务的结果：

```java
@Stateless
@Path("asyncService")
public class AsyncService {

    @Inject
    private UserBean userBean;

    @Resource(name = "LocalManagedThreadFactory")
    private ManagedThreadFactory factory;

    @GET
    public void asyncService(@Suspended AsyncResponse 
    response) {
        Thread thread = factory.newThread(() -> {
            response.resume(Response.ok(userBean
            .getUser()).build());
        });

        thread.setName("Managed Async Task");
        thread.setPriority(Thread.MIN_PRIORITY);
        thread.start();
    }

}
```

要尝试这段代码，只需将其部署到 GlassFish 5 并打开此 URL：

`http://localhost:8080/ch09-managed-thread/asyncService`

# 它是如何工作的...

在企业环境中使用线程的唯一方式，如果你真的想使用它，就是当应用程序服务器创建线程时。所以在这里，我们礼貌地请求容器使用`factory`来创建线程：

```java
    @Resource(name = "LocalManagedThreadFactory")
    private ManagedThreadFactory factory;
```

使用一些函数式风格的代码，我们创建我们的线程：

```java
        Thread thread = factory.newThread(() -> {
            response.resume(Response.ok(userBean.getUser()).build());
        });
```

现在，转向托管部分，我们可以设置新创建的线程的名称和优先级：

```java
        thread.setName("Managed Async Task");
        thread.setPriority(Thread.MIN_PRIORITY);
```

并且不要忘记要求容器启动它：

```java
        thread.start();
```

# 参见

+   在[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter09/ch09-managed-thread`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter09/ch09-managed-thread)查看这个菜谱的完整源代码。

# 使用返回结果的异步任务调度

使用任务意味着也能够定义它们应该在何时执行。这个配方就是关于这个主题的，也是关于在它们返回时获取返回结果。

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

1.  让我们先创建一个`User` POJO：

```java
public class User {

    private Long id;
    private String name;

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

    public User(Long id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" + "id=" + id + ",
        name=" + name + '}';
    }
}
```

1.  然后，我们创建一个慢速 bean 来返回`User`：

```java
public class UserBean {

    public User getUser(){
        try {
            TimeUnit.SECONDS.sleep(5);
            long id = new Date().getTime();
            return new User(id, "User " + id);
        } catch (InterruptedException ex) {
            long id = new Date().getTime();
            return new User(id, "Error " + id);
        }
    }
}
```

1.  现在我们创建一个简单的`Callable`任务与 bean 通信：

```java
public class AsyncTask implements Callable<User> {

    private final UserBean userBean = 
    CDI.current().select(UserBean.class).get();

    @Override
    public User call() throws Exception {
        return userBean.getUser();
    }
}
```

1.  最后，我们创建我们的服务以安排任务并将结果写入响应：

```java
@Stateless
@Path("asyncService")
public class AsyncService {

    @Resource(name = "LocalManagedScheduledExecutorService")
    private ManagedScheduledExecutorService executor;

    @GET
    public void asyncService(@Suspended AsyncResponse response) {

        ScheduledFuture<User> result = executor.schedule
        (new AsyncTask(), 5, TimeUnit.SECONDS);

        while (!result.isDone()) {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException ex) {
                System.err.println(ex.getMessage());
            }
        }

        try {
            response.resume(Response.ok(result.get()).build());
        } catch (InterruptedException | ExecutionException ex) {
            System.err.println(ex.getMessage());
            response.resume(Response.status(Response.Status
           .INTERNAL_SERVER_ERROR).entity(ex.getMessage())
           .build());
        }

    }

}
```

要尝试这段代码，只需将其部署到 GlassFish 5 并打开此 URL：

`http://localhost:8080/ch09-scheduled-task/asyncService`

# 它是如何工作的...

所有魔法都依赖于`AsyncService`类，所以我们将重点关注它。

首先，我们向服务器请求一个执行器的实例：

```java
    @Resource(name = "LocalManagedScheduledExecutorService")
    private ManagedScheduledExecutorService executor;
```

但它不仅仅是一个执行器——这是一个专门用于调度的执行器：

```java
ScheduledFuture<User> result = executor.schedule(new AsyncTask(), 
5, TimeUnit.SECONDS);
```

因此，我们将任务安排在五秒后执行。请注意，我们也没有使用常规的`Future`，而是使用`ScheduledFuture`。

其余的都是常规的任务执行：

```java
        while (!result.isDone()) {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException ex) {
                System.err.println(ex.getMessage());
            }
        }
```

这就是我们将结果写入响应的方式：

```java
response.resume(Response.ok(result.get()).build());
```

# 参见

+   在[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter09/ch09-scheduled-task`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter09/ch09-scheduled-task)中查看此配方的完整源代码。

# 使用注入的代理进行异步任务

当使用任务时，您也可以创建自己的执行器。如果您有非常具体的需求，这可能会非常有用。

这个配方将向您展示如何创建一个可以注入并用于应用程序整个上下文的代理执行器。

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

1.  首先，我们创建一个`User` POJO：

```java
public class User implements Serializable{

    private Long id;
    private String name;

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

    public User(Long id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" + "id=" + id + ", 
        name=" + name + '}';
    }
}
```

1.  然后我们创建一个慢速 bean 来返回`User`：

```java
public class UserBean {

    public User getUser(){
        try {
            TimeUnit.SECONDS.sleep(5);
            long id = new Date().getTime();
            return new User(id, "User " + id);
        } catch (InterruptedException ex) {
            long id = new Date().getTime();
            return new User(id, "Error " + id);
        }
    }
}
```

1.  现在我们创建一个简单的`Callable`任务与慢速 bean 通信：

```java
@Stateless
public class AsyncTask implements Callable<User>{

    @Override
    public User call() throws Exception {
        return new UserBean().getUser();
    }

}
```

1.  在这里，我们调用我们的代理：

```java
@Singleton
public class ExecutorProxy {

    @Resource(name = "LocalManagedThreadFactory")
    private ManagedThreadFactory factory;

    @Resource(name = "LocalContextService")
    private ContextService context;

    private ExecutorService executor;

    @PostConstruct
    public void init(){
        executor = new ThreadPoolExecutor(1, 5, 10, 
        TimeUnit.SECONDS, new ArrayBlockingQueue<>(5),
        factory);
    }

    public Future<User> submit(Callable<User> task){
        Callable<User> ctxProxy = 
        context.createContextualProxy(task, Callable.class);
        return executor.submit(ctxProxy);
    }
}
```

1.  最后，我们创建将使用代理的端点：

```java
@Stateless
@Path("asyncService")
public class AsyncService {

    @Inject
    private ExecutorProxy executor;

    @GET
    public void asyncService(@Suspended AsyncResponse response) 
    {
        Future<User> result = executor.submit(new AsyncTask());
        response.resume(Response.ok(result).build());
    }

}
```

要尝试这段代码，只需将其部署到 GlassFish 5 并打开此 URL：

`http://localhost:8080/ch09-proxy-task/asyncService`

# 它是如何工作的...

真正的魔法就在这里的`ExecutorProxy`任务中。首先请注意，我们是这样定义它的：

```java
@Singleton
```

我们确保在上下文中只有一个实例。

现在请注意，尽管我们正在创建自己的执行器，但我们仍然依赖于应用程序服务器上下文：

```java
    @Resource(name = "LocalManagedThreadFactory")
    private ManagedThreadFactory factory;

    @Resource(name = "LocalContextService")
    private ContextService context;
```

这保证了您不会违反任何上下文规则并永久性地破坏您的应用程序。

然后我们创建一个线程池来执行线程：

```java
    private ExecutorService executor;

    @PostConstruct
    public void init(){
        executor = new ThreadPoolExecutor(1, 5, 10,
        TimeUnit.SECONDS, new ArrayBlockingQueue<>(5), factory);
    }
```

最后，我们创建将任务发送到执行队列的方法：

```java
    public Future<User> submit(Callable<User> task){
        Callable<User> ctxProxy = context.createContextualProxy(task, 
        Callable.class);
        return executor.submit(ctxProxy);
    }
```

现在我们的代理已准备好注入：

```java
    @Inject
    private ExecutorProxy executor;
```

它也准备好被调用并返回结果：

```java
    @GET
    public void asyncService(@Suspended AsyncResponse response) {
        Future<User> result = executor.submit(new AsyncTask());
        response.resume(Response.ok(result).build());
    }
```

# 参见

+   在[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter09/ch09-proxy-task`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter09/ch09-proxy-task)中查看此配方的完整源代码。
