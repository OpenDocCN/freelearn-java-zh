# 第六章：通过依赖标准减少编码工作量

本章涵盖了以下食谱：

+   准备你的应用使用连接池

+   使用消息服务进行异步通信

+   理解 servlet 的生命周期

+   事务管理

# 简介

关于 Java EE，你需要了解的最重要概念之一是：它是一个标准。如果你访问**Java 社区进程**（**JCP**）网站，你会找到 Java EE 平台（对于版本 8 是 JSR 366）的**Java 规范请求**（**JSR**）。

一个标准...是什么？嗯，对于一个应用服务器！例如，一个 Java EE 应用服务器。

这意味着你可以开发 Java EE 应用，知道它将在一个提供了一堆你可以依赖的资源的环境中运行。

这也意味着你可以轻松地从一台应用服务器切换到另一台，只要你坚持使用 Java EE 模式而不是某些供应商特定的功能（被认为是不良实践）。无论你使用什么 Java EE 兼容服务器，你的应用都应该保持相同的行为。

哦，是的！Java EE 不仅是一个标准，还是一个认证。一个 Java EE 服务器要想被认为兼容，必须通过一系列测试来保证它实现了规范中的每一个点（JSR）。

这个令人惊叹的生态系统允许你减少应用的编码量，并给你机会专注于对你或你的客户真正重要的事情。如果没有标准环境，你需要实现自己的代码来管理请求/响应、队列、连接池和其他东西。

如果你愿意，你当然可以这样做，但你不一定必须这样做。实际上，如果你想的话，甚至可以编写自己的 Java EE 应用服务器。

话虽如此，让我们继续本章的内容！在接下来的食谱中，你将学习如何利用你最喜欢的 Java EE 应用服务器上已经实现的一些酷炫功能。

示例将基于 GlassFish 5，但正如我之前提到的，它们应该对任何其他兼容实现都有相同的行为。

# 准备你的应用使用连接池

在我们的一生中，在进食之后，我们应该学习的第一件事就是使用连接池。尤其是在我们谈论数据库时。这正是这里讨论的情况。

为什么？因为与数据库打开的连接在资源使用上代价很高。更糟糕的是，如果我们更仔细地看看打开新连接的过程，它会消耗大量的 CPU 资源，例如。

如果你只有两个用户使用包含几个表中的几个记录的数据库，可能不会有太大的区别。但是，如果你有数十个用户，或者数据库很大，当你有数百个用户使用大型数据库时，它可能会开始引起麻烦。

实际上，我自己在 J2EE 1.3 的早期（那一年是 2002 年），看到有一个由 20 人使用的应用程序中的连接池解决了性能问题。用户不多，但数据库真的很大，设计得也不是很好（应用程序也是如此，我必须说）。

但你可能想说：为什么连接池能帮助我们解决这个问题？因为一旦配置好，服务器将在启动时打开你请求的所有连接，并为你管理它们。

你唯一需要做的就是问：“嘿，服务器！你能借我一个数据库连接吗？”，用完后（这意味着尽可能快地）友好地归还它。

这个菜谱将向你展示如何操作。

# 准备就绪

首先，将正确的依赖项添加到你的项目中：

```java
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
```

如果你还没有将 GlassFish 5 下载到你的开发环境，现在是做这件事的正确时机。

# 如何操作...

1.  我们将首先在 GlassFish 5 中配置我们的连接池。一旦启动并运行，请访问以下 URL：

`http://localhost:8080`

1.  现在点击“转到管理控制台”链接，或者如果你更喜欢，直接访问以下 URL：

`http://localhost:4848/`

1.  然后按照左侧菜单中的以下路径：

资源 | JDBC | JDBC 连接池

1.  点击“新建”按钮。它将打开“新建 JDBC 连接池”页面。填写字段，如此处所述：

+   连接池名称：`MysqlPool`

+   资源类型：`javax.sql.DataSource`

+   数据库驱动供应商：`MySql`

当然，你可以根据自己的喜好进行自定义选择，但那时我们将遵循不同的路径！

1.  点击“下一步”按钮。它将打开我们的池创建过程的第二步。

这个新页面有三个部分：常规设置、池设置和事务及附加属性。对于我们的菜谱，我们只处理常规设置和附加属性。

1.  在常规设置部分，确保数据源类名已选择此值：

`com.mysql.jdbc.jdbc2.optional.MysqlDatasource`

1.  现在，让我们转到附加属性部分。可能会有很多属性列出来，但我们只需填写其中的一些：

+   数据库名称：`sys`

+   服务器名称：`localhost`

+   用户：`root`

+   密码：`mysql`

+   端口号：`3306`

1.  点击“完成”按钮，哇！你的连接池已经准备好了...或者几乎准备好了。

在进行更多配置之前，你无法访问它。在相同菜单的左侧，按照以下路径：

资源 | JDBC | JDBC 资源

1.  点击“新建”按钮，然后填写如下字段：

+   JNDI 名称：`jdbc/MysqlPool`

+   连接池名称：`MysqlPool`

现在你已经准备好了！你的连接池已经准备好使用。让我们构建一个简单的应用程序来测试它：

1.  首先，我们创建一个类来从连接池获取连接：

```java
public class ConnectionPool {

    public static Connection getConnection() throws SQLException, 
    NamingException {
        InitialContext ctx = new InitialContext();
        DataSource ds = (DataSource) ctx.lookup("jdbc/MysqlPool");

        return ds.getConnection();
    }
}
```

1.  然后，一个我们将用作`sys_config`表（MySQL 的系统表）表示的类：

```java
public class SysConfig {

    private final String variable;
    private final String value;

    public SysConfig(String variable, String value) {
        this.variable = variable;
        this.value = value;
    }

    public String getVariable() {
        return variable;
    }

    public String getValue() {
        return value;
    }
}
```

1.  这里我们创建另一个类，根据从数据库返回的数据创建一个列表：

```java
@Stateless
public class SysConfigBean {

    public String getSysConfig() throws SQLException, NamingException {
        String sql = "SELECT variable, value FROM sys_config";

        try (Connection conn = ConnectionPool.getConnection();
                PreparedStatement ps = conn.prepareStatement(sql);
                ResultSet rs = ps.executeQuery()
                Jsonb jsonb = JsonbBuilder.create()) {

            List<SysConfig> list = new ArrayList<>();
            while (rs.next()) {
                list.add(new SysConfig(rs.getString("variable"), 
                rs.getString("value")));
            }

            Jsonb jsonb = JsonbBuilder.create();
            return jsonb.toJson(list);
        }
    }
}
```

1.  最后，一个将尝试所有操作的 servlet：

```java
@WebServlet(name = "PoolTestServlet", urlPatterns = {"/PoolTestServlet"})
public class PoolTestServlet extends HttpServlet {

    @EJB
    private SysConfigBean config;

    @Override
    protected void doGet(HttpServletRequest request, 
    HttpServletResponse response)
            throws ServletException, IOException {

        try (PrintWriter writer = response.getWriter()) {
            config = new SysConfigBean();
            writer.write(config.getSysConfig());
        } catch (SQLException | NamingException ex) {
            System.err.println(ex.getMessage());
        }
    }
}
```

要尝试它，只需在浏览器中打开此 URL：

`http://localhost:8080/ch06-connectionpooling/PoolTestServlet`

# 还有更多...

决定您的连接池将保留多少连接，以及所有其他参数，是基于诸如数据类型、数据库设计、应用程序和用户行为等因素做出的架构决策。我们完全可以写一本书来讨论这个问题。

但如果您是从零开始，或者仍然不需要太多信息，请考虑一个介于 10% 到 20% 的并发用户数。换句话说，如果您的应用程序有，例如，100 个并发用户，您应该向您的池提供 10 到 20 个连接。

如果某些方法从连接池获取连接所需的时间过长（这应该根本不需要时间），您就会知道您的连接不足。这意味着在那个时刻服务器没有可用的连接。

因此，您需要检查是否有某些方法执行时间过长，或者您的代码中某些部分没有关闭连接（考虑将连接返回给服务器）。根据问题，这可能不是连接池问题，而是一个设计问题。

处理连接池的另一个重要事项是使用我们在这里使用的 "try-with-resources" 语句：

```java
        try (Connection conn = ConnectionPool.getConnection();
                PreparedStatement ps = conn.prepareStatement(sql);
                ResultSet rs = ps.executeQuery()) {
```

这将保证一旦方法执行完毕，这些资源将被正确关闭，并处理它们各自的异常，同时也有助于您编写更少的代码。

# 参见

+   在此处查看此菜谱的完整源代码：[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter06/ch06-connectionpooling`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter06/ch06-connectionpooling)

# 使用消息服务进行异步通信

由 Java EE 提供的消息服务，通过 **Java 消息服务**（**JMS**）API 实现，是 Java EE 环境提供的重要且多功能特性之一。

它使用生产者-消费者方法，其中一个对等方（生产者）将消息放入队列，另一个对等方（消费者）从那里读取消息。

生产者和消费者可以是不同的应用程序，甚至使用不同的技术。

此菜谱将向您展示如何使用 GlassFish 5 构建消息服务。每个 Java EE 服务器都有其设置服务的方式，因此如果您使用其他实现，您应该查看其文档。

另一方面，这里生成的 Java EE 代码将在任何 Java EE 8 兼容的实现上运行。标准总是胜出！

# 准备工作

首先向您的项目中添加适当的依赖项：

```java
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何做到这一点...

1.  我们将首先在 GlassFish 5 中配置我们的消息服务。一旦服务器启动并运行，请访问以下 URL：

`http://localhost:8080`

1.  现在点击 "转到管理控制台" 链接，或者如果您愿意，可以直接访问以下 URL：

`http://localhost:4848/`

然后遵循左侧菜单中的此路径：

资源 | JMS 资源 | 连接工厂

1.  点击“新建”按钮。当页面打开时，填写“常规设置”部分的字段如下：

+   JNDI 名称：`jms/JmsFactory`

+   资源类型：`javax.jms.ConnectionFactory`

我们不会在这里触摸“池设置”部分，所以只需点击“确定”按钮来注册你的新工厂。

1.  现在在左侧菜单中按照以下路径操作：

资源 | JMS 资源 | 目标资源

1.  点击“新建”按钮。当页面打开时，填写如下所示的章节字段：

+   JNDI 名称：`jms/JmsQueue`

+   物理目标名称：`JmsQueue`

+   资源类型：`javax.jms.Queue`

点击“确定”按钮，你就准备好了！现在你有一个连接工厂来访问你的 JMS 服务器和一个队列。所以让我们构建一个应用程序来使用它：

1.  首先，我们创建一个**消息驱动 Bean**（**MDB**）作为对我们队列中丢弃的任何消息的监听器。这是消费者：

```java
@MessageDriven(activationConfig = {
    @ActivationConfigProperty(propertyName = "destinationLookup", 
    propertyValue = "jms/JmsQueue"),
    @ActivationConfigProperty(propertyName = "destinationType", 
    propertyValue = "javax.jms.Queue")
})
public class QueueListener implements MessageListener {

    @Override
    public void onMessage(Message message) {
        TextMessage textMessage = (TextMessage) message;
        try {
            System.out.print("Got new message on queue: ");
            System.out.println(textMessage.getText());
            System.out.println();
        } catch (JMSException e) {
            System.err.println(e.getMessage());
        }
    }
}
```

1.  现在我们定义生产者类：

```java
@Stateless
public class QueueSender {

    @Resource(mappedName = "jms/JmsFactory")
    private ConnectionFactory jmsFactory;

    @Resource(mappedName = "jms/JmsQueue")
    private Queue jmsQueue;

    public void send() throws JMSException {
        MessageProducer producer;
        TextMessage message;

        try (Connection connection = jmsFactory.createConnection(); 
             Session session = connection.createSession(false, 
             Session.AUTO_ACKNOWLEDGE)) {

            producer = session.createProducer(jmsQueue);
            message = session.createTextMessage();

            String msg = "Now it is " + new Date();
            message.setText(msg);
            System.out.println("Message sent to queue: " + msg);
            producer.send(message);

            producer.close();
        }
    }
}
```

1.  以及一个用于访问生产者的 servlet：

```java
@WebServlet(name = "QueueSenderServlet", urlPatterns = {"/QueueSenderServlet"})
public class QueueSenderServlet extends HttpServlet {

    @Inject
    private QueueSender sender;

    @Override
    protected void doGet(HttpServletRequest request, 
    HttpServletResponse response)
            throws ServletException, IOException {
        try(PrintWriter writer = response.getWriter()){
            sender.send();
            writer.write("Message sent to queue. 
            Check the log for details.");
        } catch (JMSException ex) {
            System.err.println(ex.getMessage());
        }
    }
}
```

1.  最后，我们创建一个页面来调用我们的 servlet：

```java
<html>
    <head>
        <title>JMS recipe</title>
        <meta http-equiv="Content-Type" content="text/html; 
         charset=UTF-8">
    </head>
    <body>
        <p>
            <a href="QueueSenderServlet">Send Message to Queue</a>
        </p>
    </body>
</html>
```

现在只需部署并运行它。每次你调用`QueueSenderServlet`时，你应该在你的服务器日志中看到类似以下内容：

```java
Info: Message sent to queue: Now it is Tue Dec 19 06:52:17 BRST 2017
Info: Got new message on queue: Now it is Tue Dec 19 06:52:17 BRST 2017
```

# 它是如何工作的...

多亏了 Java EE 8 服务器中实现的标准化，我们的 MDB 100%由容器管理。这就是为什么我们只需引用队列而无需回头查看：

```java
@MessageDriven(activationConfig = {
    @ActivationConfigProperty(propertyName = "destinationLookup", 
    propertyValue = "jms/JmsQueue"),
    @ActivationConfigProperty(propertyName = "destinationType", 
    propertyValue = "javax.jms.Queue")
})
```

我们可以自己手动构建一个消费者，但这将需要三倍的代码行数，并且是同步的。

我们从我们的工厂提供的会话中获取容器生产者，并为我们的队列创建：

```java
        try (Connection connection = jmsFactory.createConnection(); 
             Session session = connection.createSession(false, 
             Session.AUTO_ACKNOWLEDGE)) {

            producer = session.createProducer(jmsQueue);
            ...
        }
```

然后我们只需要创建并发送消息：

```java
            message = session.createTextMessage();

            String msg = "Now it is " + new Date();
            message.setText(msg);
            System.out.println("Message sent to queue: " + msg);
            producer.send(message);
```

# 参见

+   你可以在这个配方中查看完整的源代码：[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter06/ch06-jms`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter06/ch06-jms)

# 理解 servlet 的生命周期

如果你习惯于使用 Java EE 创建 Web 应用程序，你可能已经意识到：大多数时候都是关于处理请求和响应，最流行的方式是通过使用 Servlet API。

这个配方将向你展示服务器如何处理其生命周期，以及你在代码中应该和不应该做什么。

# 准备工作

首先，将适当的依赖项添加到你的项目中：

```java
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何操作...

只需编写这个简单的 servlet：

```java
@WebServlet(name = "LifecycleServlet", 
urlPatterns = {"/LifecycleServlet"})
public class LifecycleServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, 
    HttpServletResponse resp) throws ServletException, IOException {
        try(PrintWriter writer = resp.getWriter()){
            writer.write("doGet");
            System.out.println("doGet");
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, 
    HttpServletResponse resp) throws ServletException, IOException {
        try(PrintWriter writer = resp.getWriter()){
            writer.write("doPost");
            System.out.println("doPost");
        }
    } 

    @Override
    protected void doDelete(HttpServletRequest req, 
    HttpServletResponse resp) throws ServletException, IOException {
        try(PrintWriter writer = resp.getWriter()){
            writer.write("doDelete");
            System.out.println("doDelete");
        }
    }

    @Override
    protected void doPut(HttpServletRequest req, 
    HttpServletResponse resp) throws ServletException, IOException {
        try(PrintWriter writer = resp.getWriter()){
            writer.write("doPut");
            System.out.println("doPut");
        }
    } 

    @Override
    public void init() throws ServletException {
        System.out.println("init()");
    }

    @Override
    public void destroy() {
        System.out.println("destroy");
    } 
}
```

一旦部署到你的 Java EE 服务器上，我建议你使用 SoapUI 或类似工具尝试它。这将允许你使用`GET`、`POST`、`PUT`和`DELETE`发送请求。浏览器只会做`GET`。

如果你这样做，你的系统日志将看起来就像这样：

```java
Info: init(ServletConfig config)
 Info: doGet
 Info: doPost
 Info: doPut
 Info: doDelete
```

如果你取消部署你的应用程序，它看起来将如下所示：

```java
Info:   destroy
```

# 它是如何工作的...

如果你注意观察，你会注意到`init`日志只有在你的 servlet 第一次被调用后才会显示。那时它真正被加载，这也是这个方法唯一被调用的时候。所以如果你为这个 servlet 有一些一次性代码，那就是你该做的。

谈到`doGet`、`doPost`、`doPut`和`doDelete`方法，请注意，它们都是根据接收到的请求由服务器自动调用的。这是由于服务器实现的一个名为`service`的另一个方法才成为可能的。

如果你愿意，可以覆盖`service`方法，但这是一种坏习惯，应该避免。只有当你确切知道你在做什么时才这样做，否则你可能会给某些请求错误的地址。这一章是关于依赖标准的，那么为什么你不遵守它们呢？

最后，当你的应用程序未部署时，会调用`destroy`方法。这就像是你的 servlet 的最后一口气。在这里添加一些代码也是一个坏习惯，因为这可能会阻止某些资源被释放，或者遇到一些进程错误。

# 参考以下内容

+   你可以在此处找到此菜谱的完整源代码：[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter06/ch06-lifecycle`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter06/ch06-lifecycle)

# 事务管理

事务管理是计算机科学中比较棘手的话题之一。一行错误，一个不可预测的情况，你的数据和/或你的用户将承受后果。

因此，如果我们可以依赖服务器为我们做这件事，那就太好了。大多数时候我们都可以，所以让我向你展示如何做到这一点。

# 准备工作

首先向你的项目中添加适当的依赖项：

```java
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何做到这一点...

1.  让我们构建一个将执行我们所需所有事务的 bean：

```java
@Stateful
@TransactionManagement
public class UserBean {

    private ArrayList<Integer> actions;

    @PostConstruct
    public void init(){
        actions = new ArrayList<>();
        System.out.println("UserBean initialized");
    }

    public void add(Integer action){
        actions.add(action);
        System.out.println(action + " added");
    }

    public void remove(Integer action){
        actions.remove(action);
        System.out.println(action + " removed");
    }

    public List getActions(){
        return actions;
    }

    @PreDestroy
    public void destroy(){
        System.out.println("UserBean will be destroyed");
    }

    @Remove
    public void logout(){
        System.out.println("User logout. Resources will be 
        released.");
    }

    @AfterBegin
    public void transactionStarted(){
        System.out.println("Transaction started");
    }

    @BeforeCompletion
    public void willBeCommited(){
        System.out.println("Transaction will be commited");
    }

    @AfterCompletion
    public void afterCommit(boolean commited){
        System.out.println("Transaction commited? " + commited);
    }   
}
```

1.  以及一个测试类来尝试它：

```java
public class UserTest {

    private EJBContainer ejbContainer;

    @EJB
    private UserBean userBean;

    public UserTest() {
    }

    @Before
    public void setUp() throws NamingException {
        ejbContainer = EJBContainer.createEJBContainer();
        ejbContainer.getContext().bind("inject", this); 
    }

    @After
    public void tearDown() {
        ejbContainer.close();
    }

    @Test
    public void test(){
        userBean.add(1);
        userBean.add(2);
        userBean.add(3);
        userBean.remove(2);
        int size = userBean.getActions().size();
        userBean.logout();
        Assert.assertEquals(2, size); 
    }   
}
```

1.  如果你尝试这个测试，你应该看到以下输出：

```java
 UserBean initialized
 Transaction started
 1 added
 Transaction will be commited
 Transaction commited? true
 Transaction started
 2 added
 Transaction will be commited
 Transaction commited? true
 Transaction started
 3 added
 Transaction will be commited
 Transaction commited? true
 Transaction started
 2 removed
 Transaction will be commited
 Transaction commited? true
 Transaction started
 Transaction will be commited
 Transaction commited? true
 Transaction started
 User logout. Resources will be released.
 UserBean will be destroyed
 Transaction will be commited
 Transaction commited? true
```

# 它是如何工作的...

我们首先做的事情是标记我们的 bean 以保持状态，并让服务器管理其事务：

```java
@Stateful
@TransactionManagement
public class UserBean {
    ...
}
```

那么，接下来会发生什么？如果你注意的话，处理添加或删除内容的任何方法都不会进行事务管理。但它们仍然被管理：

```java
 Transaction started
 1 added
 Transaction will be commited
 Transaction commited? true
```

因此，你可以在不写一行事务代码的情况下拥有所有事务智能。

即使 bean 会释放其资源，它也会进行事务处理：

```java
 Transaction started
 User logout. Resources will be released.
 UserBean will be destroyed
 Transaction will be commited
 Transaction commited? true
```

# 参考以下内容

+   你可以在此处找到此菜谱的完整源代码：[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter06/ch06-transaction`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter06/ch06-transaction)
