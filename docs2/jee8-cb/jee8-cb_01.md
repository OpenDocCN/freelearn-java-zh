# 新功能和改进

Java EE 8 是一个重大的版本，全球社区期待了大约四年。现在，整个平台比以往任何时候都更加健壮、成熟和稳定。

本章将介绍我们可以突出显示的 Java EE 8 的主要 API。虽然它们并不是这个版本涵盖的唯一主题——远非如此——但在企业环境中它们起着重要作用，值得仔细研究。

在本章中，我们将介绍以下食谱：

+   运行你的第一个 Bean Validation 2.0 代码

+   运行你的第一个 CDI 2.0 代码

+   运行你的第一个 JAX-RS 2.1 代码

+   运行你的第一个 JSF 2.3 代码

+   运行你的第一个 JSON-P 1.1 代码

+   运行你的第一个 JSON-B 1.0

+   运行你的第一个 Servlet 4.0 代码

+   运行你的第一个 Security API 1.0

+   运行你的第一个 MVC 1.0 代码

# 运行你的第一个 Bean Validation 2.0 代码

Bean Validation 是一个 Java 规范，基本上帮助您保护您的数据。通过其 API，您可以验证字段和参数，使用注解表达约束，并扩展您自定义的验证规则。

它可以与 Java SE 和 Java EE 一起使用。

在这个食谱中，您将一瞥 Bean Validation 2.0。无论您是初学者还是已经在使用 1.1 版本，这些内容都将帮助您熟悉其一些新功能。

# 准备工作

首先，您需要将正确的 Bean Validation 依赖项添加到您的项目中，如下所示：

```java
<dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
            <version>1.3</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.hibernate.validator</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>6.0.8.Final</version>
        </dependency>
        <dependency>
            <groupId>org.glassfish</groupId>
            <artifactId>javax.el</artifactId>
            <version>3.0.1-b10</version>
        </dependency>
</dependencies>
```

# 如何做到这一点...

1.  首先，我们需要创建一个包含一些待验证字段的对象：

```java
public class User {

    @NotBlank
    private String name;

    @Email
    private String email;

    @NotEmpty
    private List<@PositiveOrZero Integer> profileId;

    public User(String name, String email, List<Integer> profileId) {
        this.name = name;
        this.email = email;
        this.profileId = profileId;
    }
}
```

1.  然后，我们创建一个`test`类来验证这些约束：

```java
public class UserTest {

    private static Validator validator;

    @BeforeClass
    public static void setUpClass() {
        validator = Validation.buildDefaultValidatorFactory()
        .getValidator();
    }

    @Test
    public void validUser() {
        User user = new User(
            "elder", 
            "elder@eldermoraes.com", 
            asList(1,2));

            Set<ConstraintViolation<User>> cv = validator
            .validate(user);
            assertTrue(cv.isEmpty());
    }

    @Test
    public void invalidName() {
        User user = new User(
            "", 
            "elder@eldermoraes.com", 
            asList(1,2));

            Set<ConstraintViolation<User>> cv = validator
            .validate(user);
            assertEquals(1, cv.size());
    }

    @Test
    public void invalidEmail() {
        User user = new User(
        "elder", 
        "elder-eldermoraes_com", 
        asList(1,2));

        Set<ConstraintViolation<User>> cv = validator
        .validate(user);
        assertEquals(1, cv.size());
    } 

    @Test
    public void invalidId() {
        User user = new User(
            "elder", 
            "elder@eldermoraes.com", 
            asList(-1,-2,1,2));

            Set<ConstraintViolation<User>> cv = validator
            .validate(user);
            assertEquals(2, cv.size());
    } 
}
```

# 它是如何工作的...

我们的`User`类使用了 Bean Validation 2.0 引入的三个新约束：

+   `@NotBlank`: 确保值不是 null、空或空字符串（在评估之前会修剪值，以确保没有空格）。

+   `@Email`: 只允许有效的电子邮件格式。忘记那些疯狂的 JavaScript 函数吧！

+   `@NotEmpty`: 确保列表至少有一个项目。

+   `@PositiveOrZero`: 保证一个数字等于或大于零。

然后我们创建一个`test`类（使用 JUnit）来测试我们的验证。它首先实例化`Validator`：

```java
@BeforeClass
public static void setUpClass() {
    validator = Validation.buildDefaultValidatorFactory().getValidator();
}
```

`Validator`是一个 API，根据为它们定义的约束来验证 bean。

我们第一个`test`方法测试了一个有效的用户，它是一个具有以下属性的`User`对象：

+   名称不能为空

+   有效的电子邮件

+   `profileId`列表只包含大于零的整数：

```java
User user = new User(
   "elder", 
   "elder@eldermoraes.com", 
   asList(1,2));
```

最后，进行验证：

```java
Set<ConstraintViolation<User>> cv = validator.validate(user);
```

`Validator`的`validate()`方法返回找到的约束违规集（如果有），如果没有违规，则返回一个空集。

因此，对于一个有效的用户，它应该返回一个空集：

```java
assertTrue(cv.isEmpty());
```

其他方法与此模型略有不同：

+   `invalidName()`: 使用一个空名称

+   `invalidEmail()`: 使用一个格式错误的电子邮件

+   `invalidId()`: 向列表中添加一些负数

注意，`invalidId()`方法将两个负数添加到列表中：

```java
asList(-1,-2,1,2));
```

因此，我们预计会有两个约束违规：

```java
assertEquals(2, cv.size());
```

换句话说，`Validator` 不仅检查违反的约束，还检查它们被违反的次数。

# 参见

+   你可以在 [`beanvalidation.org/2.0/spec/`](http://beanvalidation.org/2.0/spec/) 查看 Bean Validation 2.0 规范。

+   该菜谱的完整源代码位于 [`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter01/ch01-beanvalidation/`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter01/ch01-beanvalidation/)

# 运行您的第一个 CDI 2.0 代码

**上下文和依赖注入**（**CDI**）无疑是 Java EE 平台最重要的 API 之一。在 2.0 版本中，它也支持 Java SE。

现在，CDI 对 Java EE 平台上的许多其他 API 都产生了影响。正如在 *Java EE 8 – The Next Frontier* 项目的一次采访中所说：

“如果我们在创建 JSF 的时候就有 CDI，它将会变得完全不同。”

– Ed Burns，JSF 规范负责人

CDI 2.0 中有很多新特性。本菜谱将涵盖观察者排序，让您快速入门。

# 准备工作

首先，您需要将正确的 CDI 2.0 依赖项添加到您的项目中。为了简化这一点，我们将使用 CDI SE，这是一个允许您在没有 Java EE 服务器的情况下使用 CDI 的依赖项：

```java
<dependency>
    <groupId>org.jboss.weld.se</groupId>
    <artifactId>weld-se-shaded</artifactId>
    <version>3.0.0.Final</version>
</dependency>
```

# 如何做到...

本菜谱将向您展示 CDI 2.0 引入的主要功能之一：有序观察者。现在，您可以将观察者的工作变成可预测的：

1.  首先，让我们创建一个要观察的事件：

```java
public class MyEvent {

    private final String value;

    public MyEvent(String value){
        this.value = value;
    }

    public String getValue(){
        return value;
    }
}
```

1.  现在，我们构建我们的观察者和将触发它们的服务器：

```java
public class OrderedObserver {

    public static void main(String[] args){
        try(SeContainer container =    
        SeContainerInitializer.newInstance().initialize()){
            container
                .getBeanManager()
                .fireEvent(new MyEvent("event: " + 
                System.currentTimeMillis()));
        }
    }

    public void thisEventBefore(
            @Observes @Priority(Interceptor.Priority
            .APPLICATION - 200) 
            MyEvent event){

        System.out.println("thisEventBefore: " + event.getValue());
    }

    public void thisEventAfter(
            @Observes @Priority(Interceptor.Priority
           .APPLICATION + 200) 
            MyEvent event){

        System.out.println("thisEventAfter: " + event.getValue());
    }  
}
```

此外，别忘了将 `beans.xml` 文件添加到 `META-INF` 文件夹中：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans 

       xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
       http://xmlns.jcp.org/xml/ns/javaee/beans_1_1.xsd"
       bean-discovery-mode="all">
</beans>
```

1.  一旦运行，您应该看到如下结果：

```java
INFO: WELD-ENV-002003: Weld SE container 
353db40d-e670-431d-b7be-4275b1813782 initialized
 thisEventBefore: event -> 1501818268764
 thisEventAfter: event -> 1501818268764
```

# 它是如何工作的...

首先，我们正在构建一个服务器来管理我们的事件和观察者：

```java
public static void main(String[] args){
    try(SeContainer container =  
    SeContainerInitializer.newInstance().initialize()){
        container
            .getBeanManager()
            .fireEvent(new ExampleEvent("event: " 
            + System.currentTimeMillis()));
    }
}
```

这将为我们提供运行菜谱所需的所有资源，就像它是一个 Java EE 服务器一样。

然后我们构建一个观察者：

```java
public void thisEventBefore(
        @Observes @Priority(Interceptor.Priority.APPLICATION - 200) 
        MyEvent event){

    System.out.println("thisEventBefore: " + event.getValue());
}
```

因此，我们有三个重要的话题：

+   `@Observes`：此注解用于告诉服务器它需要观察使用 `MyEvent` 触发的事件

+   `@Priority`：此注解通知观察者需要运行的优先级顺序；它接收一个 `int` 参数，执行顺序是升序的

+   `MyEvent event`：正在观察的事件

在 `thisEventBefore` 方法和 `thisEventAfter` 中，我们只更改了 `@Priority` 的值，服务器负责按正确的顺序运行它。

# 还有更多...

在 Java EE 8 服务器中，行为将完全相同。您只需不需要 `SeContainerInitializer`，并且需要将依赖项更改为以下内容：

```java
<dependency>
    <groupId>javax</groupId>
    <artifactId>javaee-api</artifactId>
    <version>8.0</version>
    <scope>provided</scope>
</dependency>
```

# 参见

+   您可以通过 [`www.cdi-spec.org/`](http://www.cdi-spec.org/) 了解与 CDI 规范相关的一切

+   该菜谱的源代码位于 [`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter01/ch01-cdi`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter01/ch01-cdi)

# 运行您的第一个 JAX-RS 2.1 代码

JAX-RS 是一个 API，旨在为 Java 提供一种便携和标准的方式来构建 RESTful Web 服务。这是在应用程序之间传输数据（包括互联网）时最常用的技术之一。

2.1 版本引入的最酷的功能之一是**服务器发送事件**（**SSE**），它将在本菜谱中介绍。SSE 是由 HTML5 创建的一个规范，其中在服务器和客户端之间建立了一个通道，单向从服务器到客户端。它是一种传输包含一些数据的消息的协议。

# 准备工作

让我们从向我们的项目中添加正确的依赖项开始：

```java
    <dependencies>
        <dependency>
            <groupId>org.glassfish.jersey.containers</groupId>
            <artifactId>jersey-container-grizzly2-http</artifactId>
            <version>2.26-b09</version>
        </dependency>
        <dependency>
            <groupId>org.glassfish.jersey.inject</groupId>
            <artifactId>jersey-hk2</artifactId>
            <version>2.26-b09</version>
        </dependency>
        <dependency>
            <groupId>org.glassfish.jersey.media</groupId>
            <artifactId>jersey-media-sse</artifactId>
            <version>2.26-b09</version>
        </dependency>
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```

你肯定注意到了我们在这里使用的是 Jersey。为什么？因为 Jersey 是 JAX-RS 的参考实现，这意味着所有 JAX-RS 规范都是首先通过 Jersey 实现的。

此外，使用 Jersey，我们可以使用 Grizzly 启动一个小型本地服务器，这对于这个菜谱很有用，因为我们只需要几个服务器功能来展示 SSE 行为。

在本书的后续内容中，我们将使用完整的 GlassFish 来构建更多的 JAX-RS 菜谱。

# 如何做到这一点...

1.  首先，我们创建一个将成为我们服务器的类：

```java
public class ServerMock {

    public static final URI CONTEXT = 
    URI.create("http://localhost:8080/");
    public static final String BASE_PATH = "ssevents";

    public static void main(String[] args) {
        try {
            final ResourceConfig resourceConfig = new 
            ResourceConfig(SseResource.class);

            final HttpServer server = 
            GrizzlyHttpServerFactory.createHttpServer(CONTEXT, 
            resourceConfig, false);
            server.start();

            System.out.println(String.format("Mock Server started
            at %s%s", CONTEXT, BASE_PATH));

            Thread.currentThread().join();
        } catch (IOException | InterruptedException ex) {
            System.out.println(ex.getMessage());
        }
    }
}
```

1.  然后，我们创建一个 JAX-RS 端点，将事件发送到客户端：

```java
@Path(ServerMock.BASE_PATH)
public class SseResource {

    private static volatile SseEventSink SINK = null;

    @GET
    @Produces(MediaType.SERVER_SENT_EVENTS)
    public void getMessageQueue(@Context SseEventSink sink) {
        SseResource.SINK = sink;
    }

    @POST
    public void addMessage(final String message, @Context Sse sse) 
    throws IOException {
        if (SINK != null) {
            SINK.send(sse.newEventBuilder()
                .name("sse-message")
                .id(String.valueOf(System.currentTimeMillis()))
                .data(String.class, message)
                .comment("")
                .build());
        }
    }
}
```

1.  然后，我们创建一个客户端类来消费从服务器生成的事件：

```java
public class ClientConsumer {

    public static final Client CLIENT = ClientBuilder.newClient();
    public static final WebTarget WEB_TARGET = 
    CLIENT.target(ServerMock.CONTEXT
    + BASE_PATH);

    public static void main(String[] args) {
        consume();
    }

    private static void consume() {

        try (final SseEventSource sseSource =
                     SseEventSource
                             .target(WEB_TARGET)
                             .build()) {

            sseSource.register(System.out::println);
            sseSource.open();

            for (int counter=0; counter < 5; counter++) {
                System.out.println(" ");
                for (int innerCounter=0; innerCounter < 5; 
                innerCounter++) {
                    WEB_TARGET.request().post(Entity.json("event "
                    + innerCounter));
                }
                Thread.sleep(1000);
            }

            CLIENT.close();
            System.out.println("\n All messages consumed");
        } catch (InterruptedException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

要尝试它，你首先需要运行`ServerMock`类，然后是`ClientConsumer`类。如果一切顺利，你应该会看到类似这样的内容：

```java
InboundEvent{name='sse-message', id='1502228257736', comment='',    data=event 0}
 InboundEvent{name='sse-message', id='1502228257753', comment='',   data=event 1}
 InboundEvent{name='sse-message', id='1502228257758', comment='',   data=event 2}
 InboundEvent{name='sse-message', id='1502228257763', comment='',   data=event 3}
 InboundEvent{name='sse-message', id='1502228257768', comment='',   data=event 4}
```

这些是从服务器发送到客户端的消息。

# 它是如何工作的...

这个菜谱由三个部分组成：

+   服务器，由`ServerMock`类表示

+   SSE 引擎，由`SseResource`类表示

+   客户端，由`ClientConsumer`类表示

因此，一旦`ServerMock`被实例化，它就会注册`SseResource`类：

```java
final ResourceConfig resourceConfig = new ResourceConfig(SseResource.class);
final HttpServer server = GrizzlyHttpServerFactory.createHttpServer(CONTEXT, resourceConfig, false);
server.start();
```

然后，`SseResource`的两个关键方法开始执行。第一个方法将消息添加到服务器队列：

```java
addMessage(final String message, @Context Sse sse)
```

第二个方法消费这个队列并将消息发送到客户端：

```java
@GET
@Produces(MediaType.SERVER_SENT_EVENTS)
public void getMessageQueue(@Context SseEventSink sink)
```

注意，这个版本引入了一个媒体类型`SERVER_SENT_EVENTS`，正是为了这个目的。最后，我们有我们的客户端。在这个菜谱中，它既是发布消息也是消费消息。

这里消费：

```java
sseSource.register(System.out::println);
sseSource.open();
```

这里发布：

```java
ServerMock.WEB_TARGET.request().post(Entity.json("event " + innerCounter));
```

# 参见

+   你可以在[`github.com/jax-rs`](https://github.com/jax-rs)上关注与 JAX-RS 相关的所有内容。

+   这个菜谱的源代码在[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter01/ch01-jaxrs`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter01/ch01-jaxrs)

# 运行你的第一个 JSF 2.3 代码

**JavaServer Faces**（**JSF**）是 Java 技术，旨在简化 UI 的构建过程，尽管它是为前端设计的，而 UI 是在后端构建的。

使用 JSF，你可以构建组件并以可扩展的方式在 UI 中使用（或重用）它们。你还可以使用其他强大的 API，如 CDI 和 Bean 验证，来改进你的应用程序及其架构。

在这个菜谱中，我们将使用 `Validator` 和 `Converter` 接口，以及版本 2.3 引入的新功能，即可以使用泛型参数使用它们。

# 准备就绪

首先，我们需要添加所需的依赖项：

```java
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何做到这一点...

1.  让我们创建一个 `User` 类作为我们菜谱的主要对象：

```java
public class User implements Serializable {

    private String name;
    private String email;

    public User(String name, String email) {
        this.name = name;
        this.email = email; 
    }

    //DON'T FORGET THE GETTERS AND SETTERS
    //THIS RECIPE WON'T WORK WITHOUT THEM
}
```

1.  现在，我们创建一个 `UserBean` 类来管理我们的 UI：

```java
@Named
@ViewScoped
public class UserBean implements Serializable {

    private User user;

    public UserBean(){
        user = new User("Elder Moraes", "elder@eldermoraes.com");
    }

    public void userAction(){
        FacesContext.getCurrentInstance().addMessage(null, 
                new FacesMessage("Name|Password welformed"));
    }

    //DON'T FORGET THE GETTERS AND SETTERS
    //THIS RECIPE WON'T WORK WITHOUT THEM
}
```

1.  现在，我们使用 `User` 参数实现 `Converter` 接口：

```java
@FacesConverter("userConverter")
public class UserConverter implements Converter<User> {

    @Override
    public String getAsString(FacesContext fc, UIComponent uic, 
    User user) {
        return user.getName() + "|" + user.getEmail();
    }

    @Override
    public User getAsObject(FacesContext fc, UIComponent uic, 
    String string) {
        return new User(string.substring(0, string.indexOf("|")), 
        string.substring(string.indexOf("|") + 1));
    }

}
```

1.  现在，我们使用 `User` 参数实现 `Validator` 接口：

```java
@FacesValidator("userValidator")
public class UserValidator implements Validator<User> {

    @Override
    public void validate(FacesContext fc, UIComponent uic, 
    User user) 
    throws ValidatorException {
        if(!user.getEmail().contains("@")){
            throw new ValidatorException(new FacesMessage(null, 
                                         "Malformed e-mail"));
        }
    }
}
```

1.  然后，我们使用所有这些来创建我们的用户界面：

```java
<h:body>
    <h:form>
        <h:panelGrid columns="3">
            <h:outputLabel value="Name|E-mail:" 
            for="userNameEmail"/>
            <h:inputText id="userNameEmail" 
            value="#{userBean.user}" 
            converter="userConverter" validator="userValidator"/> 
            <h:message for="userNameEmail"/>
        </h:panelGrid>
        <h:commandButton value="Validate" 
        action="#{userBean.userAction()}"/>
    </h:form> 
</h:body>
```

不要忘记在 Java EE 8 服务器上运行它。

# 它是如何工作的...

`UserBean` 类管理 UI 和服务器之间的通信。一旦实例化 `user` 对象，它对两者都是可用的。

这就是为什么当您运行它时，`Name | E-mail` 已经填写（当服务器创建 `UserBean` 类时，`user` 对象被实例化）。

我们将 `UserBean` 类中的 `userAction()` 方法关联到 UI 的 `Validate` 按钮上：

```java
<h:commandButton value="Validate" action="#{userBean.userAction()}"/>
```

您可以在 `UserBean` 中创建其他方法并执行相同的操作以增强您的应用程序。

我们菜谱的整个核心在 UI 中只代表一行：

```java
<h:inputText id="userNameEmail" value="#{userBean.user}" converter="userConverter" validator="userValidator"/>
```

因此，我们在这里使用的两个实现接口是 `userConverter` 和 `userValidator`。

基本上，`UserConverter` 类（具有 `getAsString` 和 `getAsObject` 方法）根据您定义的逻辑将对象转换为字符串或从字符串转换为对象，反之亦然。

我们已经在前面的代码片段中提到了它：

```java
value="#{userBean.user}"
```

服务器使用 `userConverter` 对象，调用 `getAsString` 方法，并使用前面的表达式语言打印结果。

最后，当您提交表单时，会自动调用 `UserValidator` 类，通过调用其 `validate` 方法并应用您定义的规则。

# 更多内容...

您可以通过添加 Bean Validation 来增加验证器，例如，使用 `@Email` 约束定义 `User` 中的 `email` 属性。

# 参见

+   您可以通过 [`javaserverfaces.github.io/`](https://javaserverfaces.github.io/) 跟踪与 JSF 相关的所有内容。

+   这个菜谱的源代码在 [`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter01/ch01-jsf`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter01/ch01-jsf)

# 运行您的第一个 JSON-P 1.1 代码

JSON-Pointer 是用于 JSON 处理的 Java API。我们所说的处理，是指生成、转换、解析和查询 JSON 字符串和/或对象。

在这个菜谱中，您将学习如何以非常简单的方式使用 JSON Pointer 从 JSON 消息中获取特定的值。

# 准备就绪

让我们获取我们的 `dependency`：

```java
<dependency>
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
</dependency>
```

# 如何做到这一点...

1.  首先，我们定义一个 JSON 消息来表示一个 `User` 对象：

```java
{
  "user": {
    "email": "elder@eldermoraes.com",
    "name": "Elder",
    "profile": [
      {
        "id": 1
      },
      {
        "id": 2
      },
      {
        "id": 3
      }
    ]
  }
}
```

1.  现在，我们创建一个方法来读取它并打印我们想要的值：

```java
public class JPointer {

    public static void main(String[] args) throws IOException{
        try (InputStream is = 
        JPointer.class.getClassLoader().getResourceAsStream("user.json");
                JsonReader jr = Json.createReader(is)) {

            JsonStructure js = jr.read();
            JsonPointer jp = Json.createPointer("/user/profile");
            JsonValue jv = jp.getValue(js);
            System.out.println("profile: " + jv);
        }
    }
}
```

执行此代码会打印以下内容：

```java
profile: [{"id":1},{"id":2},{"id":3}]
```

# 它是如何工作的...

JSON Pointer 是由 **互联网工程任务组**（**IETF**）在 **请求评论**（**RFC**）6901 中定义的标准。该标准基本上说 JSON Pointer 是一个字符串，用于标识 JSON 文档中的特定值。

没有 JSON Pointer，你需要解析整个消息并遍历它，直到找到所需值；可能需要很多 if、else 以及类似的东西。

因此，JSON Pointer 通过以非常优雅的方式执行此类操作，帮助你显著减少编写代码的数量。

# 参考以下内容

+   你可以关注与 JSON-P 相关的所有内容，请访问 [`javaee.github.io/jsonp/`](https://javaee.github.io/jsonp/)

+   本菜谱的源代码位于 [`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter01/ch01-jsonp`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter01/ch01-jsonp)

# 运行你的第一个 JSON-B 代码

JSON-B 是一个 API，用于以标准化的方式将 Java 对象转换为 JSON 消息。它定义了一个默认的映射算法，将 Java 类转换为 JSON，同时仍然允许你自定义自己的算法。

随着 JSON-B 的加入，Java EE 现在有一套完整的工具来处理 JSON，如 JSON API 和 JSON-P。不再需要第三方框架（尽管你仍然可以自由使用它们）。

本快速菜谱将展示如何使用 JSON-B 将 Java 对象转换为 JSON 消息，并从 JSON 消息中转换回来。

# 准备工作

让我们在项目中添加我们的依赖项：

```java
    <dependencies>
        <dependency>
            <groupId>org.eclipse</groupId>
            <artifactId>yasson</artifactId>
            <version>1.0</version>
        </dependency> 
        <dependency>
            <groupId>org.glassfish</groupId>
            <artifactId>javax.json</artifactId>
            <version>1.1</version>
        </dependency> 
    </dependencies>
```

# 如何操作...

1.  让我们创建一个 `User` 类作为我们 JSON 消息的模型：

```java
public class User {

    private String name;
    private String email;

    public User(){        
    }

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    @Override
    public String toString() {
        return "User{" + "name=" + name + ", email=" + email + '}';
    }

    //DON'T FORGET THE GETTERS AND SETTERS
    //THIS RECIPE WON'T WORK WITHOUT THEM

}
```

1.  然后，让我们创建一个类来使用 JSON-B 将对象进行转换：

```java
public class JsonBUser {

    public static void main(String[] args) throws Exception {
        User user = new User("Elder", "elder@eldermoraes.com");

        Jsonb jb = JsonbBuilder.create();
        String jsonUser = jb.toJson(user);
        User u = jb.fromJson(jsonUser, User.class);

        jb.close();
        System.out.println("json: " + jsonUser);
        System.out.println("user: " + u);

    }
}
```

打印的结果是：

```java
json: {"email":"elder@eldermoraes.com","name":"Elder"}
 user: User{name=Elder, email=elder@eldermoraes.com}
```

第一行是将对象转换为 JSON 字符串。第二行是将相同的字符串转换回对象。

# 工作原理...

它使用 `User` 类中定义的获取器和设置器进行双向转换，这就是为什么它们如此重要的原因。

# 参考以下内容

+   你可以关注与 JSON-B 相关的所有内容，请访问 [`json-b.net/`](http://json-b.net/)

+   本菜谱的源代码位于 [`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter01/ch01-jsonb`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter01/ch01-jsonb)

# 运行你的第一个 Servlet 4.0 代码

Servlet 4.0 是 Java EE 8 中最大的 API 之一。自从 Java EE 平台（旧 J2EE）的诞生以来，Servlet 规范始终扮演着关键角色。

本版本最酷的添加功能无疑是 HTTP/2.0 和服务器推送。两者都为你的应用程序带来了性能提升。

本菜谱将使用服务器推送来完成网页中最基本的任务之一——加载图片。

# 准备工作

让我们添加我们需要的依赖项：

```java
<dependency>
    <groupId>javax</groupId>
    <artifactId>javaee-api</artifactId>
    <version>8.0</version>
    <scope>provided</scope>
</dependency>
```

# 如何操作...

1.  我们将创建一个 servlet：

```java
@WebServlet(name = "ServerPush", urlPatterns = {"/ServerPush"})
public class ServerPush extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, 
    HttpServletResponse 
    response) throws ServletException, IOException {

        PushBuilder pb = request.newPushBuilder();
        if (pb != null) {
            pb.path("images/javaee-logo.png")
              .addHeader("content-type", "image/png")
              .push();
        }

        try (PrintWriter writer = response.getWriter();) {
            StringBuilder html = new StringBuilder();
            html.append("<html>");
            html.append("<center>");
            html.append("<img src='images/javaee-logo.png'><br>");
            html.append("<h2>Image pushed by ServerPush</h2>");
            html.append("</center>");
            html.append("</html>");
            writer.write(html.toString());
        }
    }
}
```

1.  要尝试它，请在 Java EE 8 服务器上运行项目并打开此 URL：

```java
https://localhost:8080/ch01-servlet/ServerPush
```

# 工作原理...

我们使用`PushBuilder`对象在`img src`标签请求之前将图像发送到客户端。换句话说，浏览器不需要再进行另一个请求（它通常使用`img src`进行）来渲染图像。

对于单个图像来说，这似乎没有太大的区别，但如果是有数十、数百或数千个图像，那就大不相同了。减少客户端和服务器端的流量。对所有都带来更好的性能！

# 还有更多...

如果您使用 JSF，您可以免费获得服务器推送的好处！您甚至不需要重写一行代码，因为 JSF 依赖于服务器推送规范。

只需确保您在 HTTPS 协议下运行它，因为 HTTP/2.0 仅在此协议下工作。

# 另请参阅

+   您可以关注与 Servlet 规范相关的所有信息，请访问[`github.com/javaee/servlet-spec`](https://github.com/javaee/servlet-spec)

+   这个食谱的源代码位于[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter01/ch01-servlet`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter01/ch01-servlet)

# 运行您的第一个 Security API 代码

在构建企业应用程序时，安全性是首要关注的问题之一。幸运的是，Java EE 平台现在有一个 API，可以以标准化的方式处理许多企业需求。

在这个食谱中，您将学习如何根据管理敏感数据的方法中定义的规则来定义角色并授予它们正确的授权。

# 准备工作

我们首先将项目依赖项添加到项目中：

```java
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.tomee</groupId>
            <artifactId>openejb-core</artifactId>
            <version>7.0.4</version>
        </dependency>
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何操作...

1.  我们首先创建一个`User`实体：

```java
@Entity 
public class User implements Serializable{

    @Id
    private Long id;
    private String name;
    private String email;

    public User(){        
    }

    public User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    //DON'T FORGET THE GETTERS AND SETTERS
    //THIS RECIPE WON'T WORK WITHOUT THEM
}
```

1.  在这里，我们创建一个类来存储我们的安全角色：

```java
public class Roles {
    public static final String ADMIN = "ADMIN";
    public static final String OPERATOR = "OPERATOR";
}
```

1.  然后，我们创建一个有状态的 bean 来管理我们的用户操作：

```java
@Stateful
public class UserBean {

    @PersistenceContext(unitName = "ch01-security-pu", 
    type = PersistenceContextType.EXTENDED)
    private EntityManager em;

    @RolesAllowed({Roles.ADMIN, Roles.OPERATOR})
    public void add(User user){
        em.persist(user);
    }

    @RolesAllowed({Roles.ADMIN})
    public void remove(User user){
        em.remove(user);
    }

    @RolesAllowed({Roles.ADMIN})
    public void update(User user){
        em.merge(user);
    }

    @PermitAll
    public List<User> get(){
        Query q = em.createQuery("SELECT u FROM User as u ");
        return q.getResultList();
    }
```

1.  现在，我们需要为每个角色创建一个执行器：

```java
public class RoleExecutor {

    public interface Executable {
        void execute() throws Exception;
    }

    @Stateless
    @RunAs(Roles.ADMIN)
    public static class AdminExecutor {
        public void run(Executable executable) throws Exception {
            executable.execute();
        }
    }

    @Stateless
    @RunAs(Roles.OPERATOR)
    public static class OperatorExecutor {
        public void run(Executable executable) throws Exception {
            executable.execute();
        }
    }
}
```

1.  最后，我们创建一个测试类来测试我们的安全规则。

我们的代码使用了三个测试方法：`asAdmin()`、`asOperator()`和`asAnonymous()`。

1.  首先，它测试`asAdmin()`：

```java
    //Lot of setup code before this point

    @Test
    public void asAdmin() throws Exception {
        adminExecutor.run(() -> {
            userBean.add(new User(1L, "user1", "user1@user.com"));
            userBean.add(new User(2L, "user2", "user2@user.com"));
            userBean.add(new User(3L, "user3", "user3@user.com"));
            userBean.add(new User(4L, "user4", "user4@user.com"));

            List<User> list = userBean.get();

            list.forEach((user) -> {
                userBean.remove(user);
            });

            Assert.assertEquals("userBean.get()", 0, 
            userBean.get().size());
        });
    }
```

1.  然后它测试`asOperator()`：

```java
    @Test
    public void asOperator() throws Exception {

        operatorExecutor.run(() -> {
            userBean.add(new User(1L, "user1", "user1@user.com"));
            userBean.add(new User(2L, "user2", "user2@user.com"));
            userBean.add(new User(3L, "user3", "user3@user.com"));
            userBean.add(new User(4L, "user4", "user4@user.com"));

            List<User> list = userBean.get();

            list.forEach((user) -> {
                try {
                    userBean.remove(user);
                    Assert.fail("Operator was able to remove user " + 
                    user.getName());
                } catch (EJBAccessException e) {
                }
            });
            Assert.assertEquals("userBean.get()", 4, 
            userBean.get().size());
        });
    }
```

1.  最后，它测试`asAnonymous()`：

```java
    @Test
    public void asAnonymous() {

        try {
            userBean.add(new User(1L, "elder", 
            "elder@eldermoraes.com"));
            Assert.fail("Anonymous user should not add users");
        } catch (EJBAccessException e) {
        }

        try {
            userBean.remove(new User(1L, "elder", 
            "elder@eldermoraes.com"));
            Assert.fail("Anonymous user should not remove users");
        } catch (EJBAccessException e) {
        }

        try {
            userBean.get();
        } catch (EJBAccessException e) {
            Assert.fail("Everyone can list users");
        }
    }
```

这个类非常大！要查看完整的源代码，请查看食谱末尾的链接。

# 它是如何工作的...

这个食谱的整个重点在于`@RolesAllowed`、`@RunsAs`和`@PermitAll`注解。它们定义了每个角色可以执行的操作以及当用户尝试使用错误角色执行操作时会发生什么。

# 还有更多...

我们在这里所做的是称为**程序化安全**；也就是说，我们通过代码（程序）定义了安全规则和角色。还有一种称为**声明式安全**的方法，其中您通过应用程序和服务器配置声明规则和角色。

对于这个食谱来说，一个很好的提升步骤是将角色管理扩展到应用程序之外，例如数据库或服务。

# 另请参阅

+   您可以关注与 Security API 相关的所有信息，请访问[`github.com/javaee-security-spec`](https://github.com/javaee-security-spec)

+   这个菜谱的源代码在 [`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter01/ch01-security`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter01/ch01-security)

# 运行你的第一个 MVC 1.0 代码

如果你正在关注 Java EE 8 的新闻，你现在可能想知道：*为什么 MVC 1.0 仍然存在，尽管它已经被从 Java EE 8 的伞下移除？*

是的，这是真的。MVC 1.0 已经不再属于 Java EE 8 版本。但这并没有减少这个伟大 API 的重要性，我相信它将在未来的版本中改变一些其他 API 的工作方式（例如，JSF）。

那为什么不在这里介绍它呢？你无论如何都会用到它。

这个菜谱将展示如何使用 Controller（C）将 Model（M）注入到 View（V）中。它还引入了一些 CDI 和 JAX-RS 技术。

# 准备工作

将适当的依赖项添加到你的项目中：

```java
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>javax.mvc</groupId>
            <artifactId>javax.mvc-api</artifactId>
            <version>1.0-pr</version>
        </dependency>
```

# 如何做到这一点...

1.  首先为你的 JAX-RS 端点创建一个根：

```java
@ApplicationPath("webresources")
public class AppConfig extends Application{
}
```

1.  创建一个 `User` 类（这将是你的 MODEL）：

```java
public class User {

    private String name;
    private String email;

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    //DON'T FORGET THE GETTERS AND SETTERS
    //THIS RECIPE WON'T WORK WITHOUT THEM
}
```

1.  现在，创建一个 Session Bean，稍后将在你的 Controller 中注入：

```java
@Stateless
public class UserBean {

    public User getUser(){
        return new User("Elder", "elder@eldermoraes.com");
    }
}
```

1.  然后，创建 Controller：

```java
@Controller
@Path("userController")
public class UserController {

    @Inject
    Models models;

    @Inject
    UserBean userBean;

    @GET
    public String user(){
        models.put("user", userBean.getUser());
        return "/user.jsp";
    }
}
```

1.  最后，是网页（View）：

```java
<head>
    <meta http-equiv="Content-Type" content="text/html; 
    charset=UTF-8">
    <title>User MVC</title>
</head>
<body>
    <h1>${user.name}/${user.email}</h1>
</body>
```

在 Java EE 8 服务器上运行它，并访问此 URL：

`http://localhost:8080/ch01-mvc/webresources/userController`

# 它是如何工作的...

整个场景中的主要角色是注入到 Controller 中的 `Models` 类：

```java
@Inject
Models models;
```

它是 MVC 1.0 API 中的一个类，在这个菜谱中负责让 `User` 对象在 View 层可用。它通过 CDI 注入，并使用另一个注入的 Bean，`userBean` 来实现：

```java
models.put("user", userBean.getUser());
```

因此，View 可以轻松地通过表达式语言访问 `User` 对象的值：

```java
<h1>${user.name}/${user.email}</h1>
```

# 参见

+   你可以在 [`github.com/mvc-spec`](https://github.com/mvc-spec) 上关注与 MVC 规范相关的一切。

+   这个菜谱的源代码在 [`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter01/ch01-mvc`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter01/ch01-mvc)
