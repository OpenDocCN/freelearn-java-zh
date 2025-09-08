# 第八章：使用微服务构建轻量级解决方案

本章涵盖了以下食谱：

+   从单体应用构建微服务

+   构建解耦的服务

+   为微服务构建自动化流水线

# 简介

**微服务**确实是当今最热门的 buzzword 之一。很容易理解为什么：在一个不断增长的软件行业中，服务、数据和用户的数量疯狂增长，我们确实需要一种方法来构建和交付更快、解耦和可扩展的解决方案。

为什么微服务很好？为什么要使用它们？

实际上，随着需求的增长，单独处理每个模块的需求也在增加。例如，在你的客户应用程序中，用户信息可能需要以不同于地址信息的方式扩展。

在单体范式下，你需要原子性地处理它：你为整个应用程序构建一个集群，或者你提升（或降低）整个主机的规模。这种方法的问题是你不能将精力和资源集中在特定的功能、模块或功能上：你总是被当时的需求所引导。

在微服务方法中，你是分开处理的。然后你不仅可以扩展（提升或降低）应用程序中的一个单一单元，而且还可以为每个服务（你应该这样做）分离数据、分离技术（最适合的工具完成最适合的工作），等等。

除了扩展技术，微服务旨在扩展人员。随着应用程序、架构和数据库的增大，团队也会变大。如果你像单体应用程序一样构建团队，你可能会得到类似的结果。

因此，随着应用程序被分割成几个（或很多）模块，你也可以定义跨职能团队来负责每个模块。这意味着每个团队都可以拥有自己的程序员、设计师、数据库管理员、系统管理员、网络专家、经理等等。每个团队对其处理的模块都有责任。

它为思考、交付软件以及维护和演进软件的过程带来了敏捷性。

在本章中，有一些食谱可以帮助你开始使用微服务或更深入地进行你的现有项目。

# 从单体应用构建微服务

我已经听到过很多次的一个常见问题是，“我该如何将单体应用拆分成微服务？”，或者，“我该如何从单体方法迁移到微服务？”

好吧，这正是这个食谱的主题。

# 准备工作

对于单体和微服务项目，我们将使用相同的依赖项：

```java
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
```

# 如何操作...

让我们从构建一个可以拆分成微服务的单体应用开始。

# 构建单体应用

首先，我们需要代表应用程序保存的数据的实体。

这里是`User`实体：

```java
@Entity
public class User implements Serializable {

    private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column
    private String name;

    @Column
    private String email;

    public User(){   
    }

    public User(String name, String email) {
        this.name = name;
        this.email = email;
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

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    } 
}
```

这里是`UserAddress`实体：

```java
@Entity
public class UserAddress implements Serializable {

    private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column
    @ManyToOne
    private User user;

    @Column
    private String street;

    @Column
    private String number;

    @Column
    private String city;

    @Column
    private String zip;

    public UserAddress(){

    }

    public UserAddress(User user, String street, String number, 
                       String city, String zip) {
        this.user = user;
        this.street = street;
        this.number = number;
        this.city = city;
        this.zip = zip;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }

    public String getStreet() {
        return street;
    }

    public void setStreet(String street) {
        this.street = street;
    }

    public String getNumber() {
        return number;
    }

    public void setNumber(String number) {
        this.number = number;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public String getZip() {
        return zip;
    }

    public void setZip(String zip) {
        this.zip = zip;
    }
}
```

现在我们定义一个 bean 来处理每个实体的交易。

这里是`UserBean`类：

```java
@Stateless
public class UserBean {

    @PersistenceContext
    private EntityManager em;

    public void add(User user) {
        em.persist(user);
    }

    public void remove(User user) {
        em.remove(user);
    }

    public void update(User user) {
        em.merge(user);
    }

    public User findById(Long id) {
        return em.find(User.class, id);
    }

    public List<User> get() {
        CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<User> cq = cb.createQuery(User.class);
        Root<User> pet = cq.from(User.class);
        cq.select(pet);
        TypedQuery<User> q = em.createQuery(cq);
        return q.getResultList();
    }

}
```

这里是`UserAddressBean`类：

```java
@Stateless
public class UserAddressBean {

    @PersistenceContext
    private EntityManager em;

    public void add(UserAddress address){
        em.persist(address);
    }

    public void remove(UserAddress address){
        em.remove(address);
    }

    public void update(UserAddress address){
        em.merge(address);
    }

    public UserAddress findById(Long id){
        return em.find(UserAddress.class, id);
    }

    public List<UserAddress> get() {
        CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<UserAddress> cq = cb.createQuery(UserAddress.class);
        Root<UserAddress> pet = cq.from(UserAddress.class);
        cq.select(pet);
        TypedQuery<UserAddress> q = em.createQuery(cq);
        return q.getResultList();
    } 
}
```

最后，我们构建两个服务以在客户端和 bean 之间进行通信。

这里是`UserService`类：

```java
@Path("userService")
public class UserService {

    @EJB
    private UserBean userBean;

    @GET
    @Path("findById/{id}")
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    public Response findById(@PathParam("id") Long id){
        return Response.ok(userBean.findById(id)).build();
    }

    @GET
    @Path("get")
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    public Response get(){
        return Response.ok(userBean.get()).build();
    } 

    @POST
    @Path("add")
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON) 
    public Response add(User user){
        userBean.add(user);
        return Response.accepted().build();
    } 

    @DELETE
    @Path("remove/{id}")
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON) 
    public Response remove(@PathParam("id") Long id){
        userBean.remove(userBean.findById(id));
        return Response.accepted().build();
    }
}
```

这里是`UserAddressService`类：

```java
@Path("userAddressService")
public class UserAddressService {

    @EJB
    private UserAddressBean userAddressBean;

    @GET
    @Path("findById/{id}")
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    public Response findById(@PathParam("id") Long id){
        return Response.ok(userAddressBean.findById(id)).build();
    }

    @GET
    @Path("get")
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    public Response get(){
        return Response.ok(userAddressBean.get()).build();
    } 

    @POST
    @Path("add")
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON) 
    public Response add(UserAddress address){
        userAddressBean.add(address);
        return Response.accepted().build();
    } 

    @DELETE
    @Path("remove/{id}")
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON) 
    public Response remove(@PathParam("id") Long id){
        userAddressBean.remove(userAddressBean.findById(id));
        return Response.accepted().build();
    }
}
```

现在让我们将其分解！

# 从单体架构构建微服务

我们的单体架构处理`User`和`UserAddress`。因此，我们将它分解为三个微服务：

+   用户微服务

+   用户地址微服务

+   网关微服务

网关服务是应用程序客户端和服务之间的 API。使用它可以使您简化这种通信，同时也给您提供了自由，可以随意处理您的服务，而不会破坏 API 合约（或者至少最小化它）。

# 用户微服务

`User`实体、`UserBean`和`UserService`将保持与单体架构中完全相同。但现在它们将作为一个独立的部署单元提供。

# 用户地址微服务

`UserAddress`类将从单体版本中仅经历一个更改，但将保留其原始 API（这对于客户端来说是非常好的）。

这里是`UserAddress`实体：

```java
@Entity
public class UserAddress implements Serializable {

    private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column
    private Long idUser;

    @Column
    private String street;

    @Column
    private String number;

    @Column
    private String city;

    @Column
    private String zip;

    public UserAddress(){

    }

    public UserAddress(Long user, String street, String number, 
                       String city, String zip) {
        this.idUser = user;
        this.street = street;
        this.number = number;
        this.city = city;
        this.zip = zip;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public Long getIdUser() {
        return idUser;
    }

    public void setIdUser(Long user) {
        this.idUser = user;
    }

    public String getStreet() {
        return street;
    }

    public void setStreet(String street) {
        this.street = street;
    }

    public String getNumber() {
        return number;
    }

    public void setNumber(String number) {
        this.number = number;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public String getZip() {
        return zip;
    }

    public void setZip(String zip) {
        this.zip = zip;
    }
}
```

注意，`User`不再是`UserAddress`实体中的一个属性/字段，而只是一个数字（`idUser`）。我们将在下一节中更详细地介绍这一点。

# 网关微服务

首先，我们创建一个帮助我们处理响应的类：

```java
public class GatewayResponse {

    private String response;
    private String from;

    public String getResponse() {
        return response;
    }

    public void setResponse(String response) {
        this.response = response;
    }

    public String getFrom() {
        return from;
    }

    public void setFrom(String from) {
        this.from = from;
    }
}
```

然后，我们创建我们的网关服务：

```java
@Consumes(MediaType.APPLICATION_JSON)
@Path("gatewayResource")
@RequestScoped
public class GatewayResource {

    private final String hostURI = "http://localhost:8080/";
    private Client client;
    private WebTarget targetUser;
    private WebTarget targetAddress;

    @PostConstruct
    public void init() {
        client = ClientBuilder.newClient();
        targetUser = client.target(hostURI + 
        "ch08-micro_x_mono-micro-user/");
        targetAddress = client.target(hostURI +
        "ch08-micro_x_mono-micro-address/");
    }

    @PreDestroy
    public void destroy(){
        client.close();
    }

    @GET
    @Path("getUsers")
    @Produces(MediaType.APPLICATION_JSON)
    public Response getUsers() {
        WebTarget service = 
        targetUser.path("webresources/userService/get");

        Response response;
        try {
            response = service.request().get();
        } catch (ProcessingException e) {
            return Response.status(408).build();
        }

        GatewayResponse gatewayResponse = new GatewayResponse();
        gatewayResponse.setResponse(response.readEntity(String.class));
        gatewayResponse.setFrom(targetUser.getUri().toString());

        return Response.ok(gatewayResponse).build();
    }

    @POST
    @Path("addAddress")
    @Produces(MediaType.APPLICATION_JSON) 
    public Response addAddress(UserAddress address) {
        WebTarget service = 
        targetAddress.path("webresources/userAddressService/add");

        Response response;
        try {
            response = service.request().post(Entity.json(address));
        } catch (ProcessingException e) {
            return Response.status(408).build();
        }

        return Response.fromResponse(response).build();
    }

}
```

当我们在网关中接收到`UserAddress`实体时，我们也要在网关项目中有一个它的版本。为了简洁，我们将省略代码，因为它与`UserAddress`项目中的代码相同。

# 它是如何工作的...

让我们了解这里的工作原理。

# 单体架构

单体应用程序的结构非常简单：只是一个包含两个服务、使用两个 bean 来管理两个实体的项目。如果您想了解有关 JAX-RS、CDI 和/或 JPA 的细节，请查看本书前面的相关食谱。

# 微服务

因此，我们将单体架构拆分为三个项目（微服务）：用户服务、用户地址服务和网关服务。

在从单体版本迁移后，用户服务类保持不变。因此，没有太多可评论的。

`UserAddress`类必须更改才能成为微服务。第一个更改是在实体上进行的。

这里是单体版本的`UserAddress`：

```java
@Entity
public class UserAddress implements Serializable {

    ...

    @Column
    @ManyToOne
    private User user;

    ...

    public UserAddress(User user, String street, String number, 
                       String city, String zip) {
        this.user = user;
        this.street = street;
        this.number = number;
        this.city = city;
        this.zip = zip;
    }

    ...

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }

    ...

}
```

这里是微服务版本的代码：

```java
@Entity
public class UserAddress implements Serializable {

    ...

    @Column
    private Long idUser;

    ...

    public UserAddress(Long user, String street, String number, 
                       String city, String zip) {
        this.idUser = user;
        this.street = street;
        this.number = number;
        this.city = city;
        this.zip = zip;
    }

    public Long getIdUser() {
        return idUser;
    }

    public void setIdUser(Long user) {
        this.idUser = user;
    }

    ...

}
```

注意，在单体版本中，`user`是`User`实体的一个实例：

```java
private User user;
```

在微服务版本中，它变成了一个数字：

```java
private Long idUser;
```

这主要是由于两个主要原因：

1.  在单体架构中，我们有两个表位于同一个数据库中（`User`和`UserAddress`），并且它们都有物理和逻辑关系（外键）。因此，保持这两个对象之间的关系也是有意义的。

1.  微服务应该有自己的数据库，完全独立于其他服务。因此，我们选择只保留用户 ID，因为当客户端需要加载地址时，这已经足够了。

这个更改也导致了构造函数的变化。

这里是单体版本：

```java
public UserAddress(User user, String street, String number, 
                   String city, String zip)
```

这里是微服务版本：

```java
public UserAddress(Long user, String street, String number, 
                   String city, String zip)
```

这可能导致与客户端关于构造函数签名的更改的合同变更。但多亏了它的构建方式，这并不是必要的。

这里是单体版本：

```java
public Response add(UserAddress address)
```

这里是微服务版本：

```java
public Response add(UserAddress address)
```

即使方法被更改，也可以很容易地通过`@Path`注解来解决，或者如果我们真的需要更改客户端，那也仅仅是方法名而不是参数（这曾经更痛苦）。

最后，我们有网关服务，这是我们实现的 API 网关设计模式。基本上，它是访问其他服务的唯一单一点。

好处在于你的客户端不需要关心其他服务是否更改了 URL、签名，甚至它们是否可用。网关会处理这些。

坏处在于它也只有一个故障点。或者换句话说，没有网关，所有服务都不可达。但你可以通过集群等方式来处理它。

# 还有更多...

虽然 Java EE 非常适合微服务，但还有其他基于相同基础的选择，在某些场景下可能更轻量。

其中之一是 KumuluzEE ([`ee.kumuluz.com/`](https://ee.kumuluz.com/))。它基于 Java EE，具有许多微服务必备功能，如服务发现。它赢得了 Duke Choice Awards 奖项，这是一个巨大的成就！

另一个是 Payara Micro ([`www.payara.fish/payara_micro`](https://www.payara.fish/payara_micro))。Payara 是拥有 GlassFish 商业实现的 Payara 公司，从 Payara Server 中，他们创建了 Payara Fish。酷的地方在于它只是一个 60 MB 的 JAR 文件，你通过命令行启动它，然后！你的微服务就运行起来了。

最后，这两个项目的酷之处在于它们与 Eclipse MicroProfile 项目([`microprofile.io/`](http://microprofile.io/))保持一致。MicroProfile 目前正在定义 Java EE 生态系统中微服务的路径和标准，所以值得关注。

关于这个菜谱中涵盖的代码的最后一点：在现实世界的解决方案中使用 DTO 来分离数据库表示和服务表示会更好。

# 参见

这个菜谱的完整源代码可以在以下存储库中找到：

+   **单体**：[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter08/ch08-micro_x_mono-mono`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter08/ch08-micro_x_mono-mono)

+   **用户微服务**：[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter08/ch08-micro_x_mono-micro-user`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter08/ch08-micro_x_mono-micro-user)

+   **用户地址微服务**：[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter08/ch08-micro_x_mono-micro-address`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter08/ch08-micro_x_mono-micro-address)

+   **网关微服务**：[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter08/ch08-micro_x_mono-micro-gateway`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter08/ch08-micro_x_mono-micro-gateway)

# 构建解耦服务

也许你至少听说过在软件世界中构建解耦事物的概念：解耦的类、解耦的模块，以及解耦的服务。

但一个软件单元从另一个解耦意味着什么呢？

从实际的角度来看，当对其中一个进行任何更改时，需要你同时更改另一个，这两个事物就是耦合在一起的。例如，如果你有一个返回字符串的方法，并将其更改为返回双精度浮点数，那么调用该方法的全部方法都需要进行更改。

耦合有层次之分。例如，你可以将所有类和方法设计得非常松散耦合，但它们都是用 Java 编写的。如果你将其中一个更改为.NET，并且希望将它们全部（在同一部署包中）保留在一起，你需要将其他所有方法更改为新语言。

关于耦合的另一件事是，一个单元对另一个单元了解多少。当它们对彼此了解很多时，它们是紧密耦合的，反之，如果它们对彼此了解很少或几乎一无所知，它们就是松散耦合的。这种观点主要与两个（或更多）部分的行为相关。

考虑耦合的另一种方式是契约。如果更改契约会破坏客户端，那么它们是紧密耦合的。如果不是，它们是松散耦合的。这就是为什么使用接口来促进松散耦合是最好的方式。因为它们为其实施者创建契约，使用它们在类之间进行通信可以促进松散耦合。

嗯...服务呢？在我们的案例中，是微服务。

当更改一个服务时不需要更改另一个服务时，一个服务与另一个服务是松散耦合的。你可以从行为或契约的角度来考虑这两个服务。

当谈到微服务时，这一点尤为重要，因为你的应用程序中可能有成百上千个微服务，如果更改其中一个需要你更改其他所有服务，你可能会破坏整个应用程序。

这个菜谱将向你展示如何在微服务中避免紧密耦合，从第一行代码开始，这样你就可以避免未来的重构（至少出于这个原因）。

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

    private String name;
    private String email;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }   
}
```

1.  然后我们创建一个包含两个方法（端点）的类来返回`User`：

```java
@Path("userService")
public class UserService {

    @GET
    @Path("getUserCoupled/{name}/{email}")
    @Produces(MediaType.APPLICATION_JSON)
    public Response getUserCoupled(
            @PathParam("name") String name, 
            @PathParam("email") String email){
        //GET USER CODE

        return Response.ok().build();
    }

    @GET
    @Path("getUserDecoupled")
    @Produces(MediaType.APPLICATION_JSON)
    public Response getUserDecoupled(@HeaderParam("User") 
    User user){
        //GET USER CODE

        return Response.ok().build();
    }
}
```

1.  最后，我们创建另一个服务（另一个项目）来消费`UserService`：

```java
@Path("doSomethingService")
public class DoSomethingService {

    private final String hostURI = "http://localhost:8080/";
    private Client client;
    private WebTarget target;

    @PostConstruct
    public void init() {
        client = ClientBuilder.newClient();
        target = client.target(hostURI + "ch08-decoupled-user/");
    } 

    @Path("doSomethingCoupled")
    @Produces(MediaType.APPLICATION_JSON)
    public Response doSomethingCoupled(String name, String email){
        WebTarget service = 
        target.path("webresources/userService/getUserCoupled");
        service.queryParam("name", name);
        service.queryParam("email", email);

        Response response;
        try {
            response = service.request().get();
        } catch (ProcessingException e) {
            return Response.status(408).build();
        }

        return 
        Response.ok(response.readEntity(String.class)).build();
    }

    @Path("doSomethingDecoupled")
    @Produces(MediaType.APPLICATION_JSON)
    public Response doSomethingDecoupled(User user){
        WebTarget service = 
        target.path("webresources/userService/getUserDecoupled");

        Response response;
        try {
            response = service.request().header("User", 
            Entity.json(user)).get();
        } catch (ProcessingException e) {
            return Response.status(408).build();
        }

        return 
        Response.ok(response.readEntity(String.class)).build();
    } 
}
```

# 它是如何工作的...

正如你可能已经注意到的，我们在代码中创建了两种情况：一种是明显耦合的（`getUserCoupled`）另一种是解耦的（`getUserDecoupled`）：

```java
    public Response getUserCoupled(
            @PathParam("name") String name, 
            @PathParam("email") String email)
```

为什么这是一个耦合的方法，因此也是一个耦合的服务？因为它高度依赖于方法签名。想象它是一个搜索服务，`"name"`和`"email"`是过滤器。现在想象一下，在未来的某个时候，你需要添加另一个过滤器。签名中再增加一个参数。

好吧，你可以同时保留两个方法，这样你就不必打破客户端并更改客户端。有多少个？移动端、服务、网页等等。所有这些都需要更改以支持新功能。

现在看看这个：

```java
    public Response getUserDecoupled(@HeaderParam("User") User user)
```

在这个`User`搜索方法中，如果你需要向过滤器中添加一个新参数怎么办？好吧，继续添加！合同没有变化，所有客户端都很高兴。

如果你的`User` POJO 最初只有两个属性，一年后增加到一百个，没问题。你的服务合同保持不变，甚至那些没有使用新字段客户端仍然可以正常工作。太棒了！

耦合/解耦服务的结果可以在调用服务中看到：

```java
    public Response doSomethingCoupled(String name, String email){
        WebTarget service = 
        target.path("webresources/userService/getUserCoupled");
        service.queryParam("name", name);
        service.queryParam("email", email);

        ...
    }
```

调用服务完全耦合到被调用服务：它必须*知道*被调用服务属性的名称，并且每次它改变时都需要添加/更新。

现在看看这个：

```java
    public Response doSomethingDecoupled(User user){
        WebTarget service = 
        target.path("webresources/userService/getUserDecoupled");

        Response response;
        try {
            response = service.request().header("User", 
            Entity.json(user)).get();
            ...
    }
```

在这种情况下，你只需要引用唯一的服务参数（`"User"`）并且它永远不会改变，无论`User` POJO 如何变化。

# 参见

在以下链接中查看完整源代码：

+   **UserService**：[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter08/ch08-decoupled-user`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter08/ch08-decoupled-user)

+   **DoSomethingService**：[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter08/ch08-decoupled-dosomethingwithuser`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter08/ch08-decoupled-dosomethingwithuser)

# 为微服务构建自动化管道

也许你在想，“为什么在 Java EE 8 书中会有自动化食谱？”或者甚至，“Java EE 8 下是否有任何规范定义了管道自动化？”

第二个问题的答案是*没有*。至少在现在这个时刻是这样的。第一个问题的答案我将在下面解释。

在许多会议上，我经常被问到这样的问题，“我该如何将我的单体应用迁移到微服务？”这个问题有一些变体，但最终问题都是一样的。

人们出于不同的原因想要这样做：

+   他们想要跟上潮流

+   他们想要与看起来像新时尚的东西一起工作

+   他们想要扩展应用程序

+   他们希望能够在同一个解决方案下使用不同的技术栈

+   他们希望看起来很酷

任何这些原因都是可以接受的，如果你想的话，你可以用任何一个理由来证明你的迁移到微服务的合理性。我可能会质疑其中一些人的真实动机，但...

而不是给他们提供建议、技巧、指南或其他技术讲座，我通常问一个简单的问题：“你已经有了一个用于你的单体的自动化流水线吗？”

大多数时候，答案是失望的“不”，然后是一个好奇的，“为什么？”

好吧，答案很简单：如果你不自动化流水线，你，单体，一个单独的包，有时你会遇到问题，那么你为什么认为当你有几十、几百甚至几千个部署文件时，事情会更容易呢？

让我更具体地说：

+   你是手动构建部署工件吗？使用 IDE 或其他工具？

+   你是手动部署的吗？

+   你是否因为任何原因（如错误、缺失工件或其他任何原因）遇到过部署问题？

+   你是否因为缺乏测试而遇到过问题？

如果你至少对这些问题中的一个问题回答了“是”，并且没有自动化流水线，想象一下这些问题被成倍放大……又是几十、几百或几千。

有些人甚至不写单元测试。想象一下那些隐藏的错误在无数被称为微服务的工件中进入生产。你的微服务项目可能甚至在没有上线之前就会失败。

所以是的，在考虑微服务之前，你需要尽可能地在你的流水线中自动化尽可能多的东西。这是防止问题扩散的唯一方法。

自动化流水线有三个成熟阶段：

1.  **持续集成**（**CI**）：基本上，这确保了你的新代码将尽快合并到主分支（例如，`master`分支）。这是基于这样一个事实：你合并的代码越少，你添加的错误就越少。它主要通过在构建时运行单元测试来实现。

1.  **持续交付**：这是 CI 的进一步发展，其中你只需点击一下按钮就可以保证你的工件准备好部署。这通常需要一个用于你的二进制文件的工件存储库和一个管理它的工具。在采用持续交付时，你决定何时进行部署，但最佳实践是尽可能快地进行部署，以避免一次性在生产中添加大量新代码。

1.  **持续部署**（**CD**）：这是自动化的最后一部分，也是最前沿的部分。在 CD 中，从代码提交到部署到生产中，没有人为的交互。唯一可能阻止工件部署的是流水线阶段中的任何错误。全球所有主要的微服务成功案例都在他们的项目中使用了 CD，每天进行数百甚至数千次部署。

这个配方将向你展示你如何将任何 Java EE 项目从零（完全没有自动化）发展到三（CD）。这是一个概念性的配方，但也包含一些代码。

不要反对概念；它们是你作为 Java EE 开发者的职业关键。

“走向微服务”是一件大事，在应用程序和组织中意味着很多。有些人甚至说微服务完全是关于扩展人员，而不是技术。

当然，我们将在技术方面继续前进。

# 准备就绪

由于微服务涉及很多方面，它们也会带来很多工具。这个食谱并不打算深入到每个工具的设置，而是展示它们在微服务自动化管道中的工作方式。

这里选择的技术工具并不是执行这些角色的唯一选择。它们只是我在这些角色中的最爱。

# 准备应用程序

为了准备你的应用程序——你的微服务——进行自动化，你需要：

+   **Apache Maven**：这主要用于构建阶段，它还将帮助你处理与之相关的许多活动。它管理依赖项，运行单元测试，等等。

+   **JUnit**：这用于编写将在构建阶段执行的单元测试。

+   **Git**：为了想象中最神圣的事物，请为你的源代码使用一些版本控制。在这里，我将基于 GitHub。

# 准备环境

为了准备你的管道环境，你需要：

+   **Sonatype Nexus**：这是一个二进制仓库。换句话说，当你构建你的工件时，它将被存储在 Nexus 中，并准备好部署到你需要/想要的地方。

+   **Jenkins**：我以前说过 Jenkins 是万能的自动化工具。实际上，我在一个项目中使用它为大约 70 个应用程序构建了自动化管道（持续交付），这些应用程序使用了完全不同的技术（语言、数据库、操作系统等）。你基本上会使用它来进行构建和部署。

# 如何操作...

你将指导达到三个自动化成熟阶段：持续集成、持续交付和持续部署。

# 持续集成

在这里，你需要尽快让你的新代码合并到主分支。你可以通过以下方式实现：

+   Git

+   Maven

+   JUnit

因此，你将确保你的代码构建正确，测试计划并成功执行。

# Git

我不会深入讲解如何使用 Git 及其命令，因为这不是本书的重点。如果你是 Git 世界的完全新手，可以从查看这张速查表开始：

[`education.github.com/git-cheat-sheet-education.pdf`](https://education.github.com/git-cheat-sheet-education.pdf)

# Maven

Maven 是我见过的最强大的工具之一，因此它内置了许多功能。如果你是新手，可以查看这个参考：

[`maven.apache.org/guides/MavenQuickReferenceCard.pdf`](https://maven.apache.org/guides/MavenQuickReferenceCard.pdf)

在基于 Maven 的项目中，最重要的文件是`pom.xml`（**POM**代表**项目对象模型**）。例如，当你创建一个新的 Java EE 8 项目时，它应该看起来像这样：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.eldermoraes</groupId>
    <artifactId>javaee8-project-template</artifactId>
    <version>1.0</version>
    <packaging>war</packaging>

    <name>javaee8-project-template</name>

    <properties>
        <endorsed.dir>${project.build.directory}/endorsed</endorsed.dir>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>8.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <compilerArguments>
                        <endorseddirs>${endorsed.dir}</endorseddirs>
                    </compilerArguments>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>2.3</version>
                <configuration>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

然后你的项目就准备好使用 Maven 构建了，如下所示（在`pom.xml`所在的同一文件夹中运行）：

```java
mvn
```

# JUnit

你将使用 JUnit 来运行你的单元测试。让我们检查一下。

这里是一个要测试的类：

```java
public class JUnitExample {

    @Size (min = 6, max = 10,message = "Name should be between 6 and 10 
           characters")
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

}
```

这里是一个测试类：

```java
public class JUnitTest {

    private static Validator VALIDATOR;

    @BeforeClass
    public static void setUpClass() {
        VALIDATOR = Validation.buildDefaultValidatorFactory().getValidator();
    }

    @Test
    public void smallName(){
        JUnitExample junit = new JUnitExample();

        junit.setName("Name");

        Set<ConstraintViolation<JUnitExample>> cv = 
        VALIDATOR.validate(junit);
        assertFalse(cv.isEmpty());
    }

    @Test
    public void validName(){
        JUnitExample junit = new JUnitExample();

        junit.setName("Valid Name");

        Set<ConstraintViolation<JUnitExample>> cv = 
        VALIDATOR.validate(junit);
        assertTrue(cv.isEmpty());
    }

    @Test
    public void invalidName(){
        JUnitExample junit = new JUnitExample();

        junit.setName("Invalid Name");

        Set<ConstraintViolation<JUnitExample>> cv = 
        VALIDATOR.validate(junit);
        assertFalse(cv.isEmpty());
    } 
}
```

每次你运行此项目的构建过程时，前面的测试将会执行，并确保这些条件仍然有效。

现在，你已经准备好进行持续集成。只需确保尽快将你的新代码和有效代码合并到主分支。现在让我们继续到持续交付。

# 持续交付

现在你已经成为一个提交者机器，让我们更进一步，让你的应用程序随时可以部署。

首先，你需要确保你刚刚构建的工件可以在适当的存储库中可用。这就是我们使用 Sonatype Nexus 的时候。

我不会在这本书中详细介绍设置细节。一种简单的方法是使用 Docker 容器。你可以在以下链接中了解更多信息，[`hub.docker.com/r/sonatype/nexus/`](https://hub.docker.com/r/sonatype/nexus/)。

一旦你的 Nexus 可用，你需要转到`pom.xml`文件并添加以下配置：

```java
    <distributionManagement>
        <repository>
            <id>Releases</id>
            <name>Project</name>
            <url>[NEXUS_URL]/nexus/content/repositories/releases/</url>
        </repository>
     </distributionManagement>
```

现在，不再构建，而是使用以下内容：

```java
mvn
```

你会这样做：

```java
mvn deploy
```

因此，一旦你的工件构建完成，Maven 将会将其上传到 Sonatype Nexus。现在它已经适当地存储起来，以供未来的部署使用。

现在你几乎准备好跳起自动化之舞了。让我们把 Jenkins 带到派对上。

如同 Nexus 所提到的，我不会深入介绍设置 Jenkins 的细节。我也建议你使用 Docker 进行设置。有关详细信息，请参阅以下链接：

[`hub.docker.com/_/jenkins/`](https://hub.docker.com/_/jenkins/)

如果你完全不知道如何使用 Jenkins，请参考此官方指南：

[`jenkins.io/user-handbook.pdf`](https://jenkins.io/user-handbook.pdf)

一旦你的 Jenkins 启动并运行，你将创建两个作业：

1.  **Your-Project-Build**：这个作业将用于从源代码构建你的项目。

1.  **Your-Project-Deploy**：这个作业将在工件在 Nexus 构建和存储后用于部署。

你将配置第一个作业下载你的项目源代码并使用 Maven 构建它。第二个将从中下载并部署到应用程序服务器。

记住，部署过程在大多数情况下涉及一些步骤：

1.  停止应用程序服务器。

1.  删除上一个版本。

1.  从 Nexus 下载新版本。

1.  部署新版本。

1.  启动应用程序服务器。

因此，你可能需要创建一个 shell 脚本，由 Jenkins 执行。记住，我们正在自动化，所以没有手动过程。

下载工件可能有点棘手，所以你可能在你的 shell 脚本中使用类似以下的内容：

```java
wget --user=username --password=password "[NEXUS_URL]/nexus/service/local/artifact/maven/content?g=<group>&a=<artifact>
&v=<version>&r=releases"
```

如果一切顺利，那么你将有两个按钮：一个用于构建，另一个用于部署。你已经准备好，无需使用任何 IDE 进行部署，也无需触摸应用程序服务器。

现在你确信这两个过程（构建和部署）将每次都以完全相同的方式进行执行。你现在可以计划在更短的时间内执行它们。

好吧，现在我们将进入下一步，也是最好的一步：持续部署。

# 持续部署

从交付到部署是一个成熟的过程——你需要一个可靠的过程来确保只有可工作的代码进入生产环境。

你已经在每次构建时运行了单元测试。实际上，你没有忘记编写单元测试，对吧？

在每次成功执行后，你的构建工件将被妥善存储，并且你将管理好你应用程序的正确版本。

你已经掌握了你应用程序的部署过程，正确处理了可能出现的任何条件。你的应用程序服务器不会再在没有你知情的情况下宕机，而你只用了两个按钮就实现了这一点！构建和部署。你太棒了！

如果你已经到了这个阶段，你的下一步不应该是个大问题。你只需要自动化这两个作业，这样你就不需要再按按钮了。

在构建作业中，你可以设置它在 Jenkins 发现源代码仓库中的任何更改时执行（如果你不知道如何操作，请查看文档）。

一旦完成，就只剩下最后一个配置：让构建作业在构建步骤中调用另一个作业——部署作业。所以每次构建成功执行时，部署也会立即执行。

干杯！你已经成功了。

# 还有更多...

当然，你不仅需要执行单元测试或 API 测试。如果你有 UI，你还需要测试你的 UI。

我建议使用 Selenium Webdriver 来完成这个操作。更多信息请参考这里，[`www.seleniumhq.org/docs/03_webdriver.jsp`](http://www.seleniumhq.org/docs/03_webdriver.jsp)。

在这种情况下，你可能希望将你的应用程序部署到 QA 环境，运行 UI 测试，如果一切正常，然后进入生产环境。所以这只是一个在管道中添加一些新作业的问题，现在你知道如何做了。

# 参考以下内容

+   JUnit 示例的源代码可以在以下位置找到，[`github.com/eldermoraes/javaee8-cookbook/tree/master/chapter08/ch08-automation`](https://github.com/eldermoraes/javaee8-cookbook/tree/master/chapter08/ch08-automation)。
