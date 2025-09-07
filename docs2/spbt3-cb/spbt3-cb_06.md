

# 第六章：数据持久性和 Spring Data 与 NoSQL 数据库集成

SQL 和 NoSQL 数据库提供了一种灵活且可扩展的数据存储和检索方法，与传统的数据库相比，它们可能更适合某些用例。NoSQL 数据库旨在实现水平扩展、灵活性、性能、高可用性和全球分布。然而，因此您会失去关系型数据库可以提供的完整 SQL 实现的致性、ACID 合规性和表达性。

重要的是要注意，NoSQL 数据库不是一刀切解决方案，它们的适用性取决于您应用程序的需求。在某些情况下，SQL 和 NoSQL 数据库的组合可能是满足组织内不同数据存储和检索需求的最佳方法。

在本章中，我们将使用一些最受欢迎的 NoSQL 数据库。它们各自对数据访问有不同的方法，但 Spring Boot 在所有这些数据库中都简化了开发体验。

首先，我们将学习如何使用 MongoDB，这是一个面向文档的数据库，它以类似 JSON 的对象存储数据。我们将涵盖 MongoDB 中的数据访问基础，以及其他高级场景，例如索引、事务和乐观并发持久性。

接下来，我们将学习如何使用 Apache Cassandra。它是一个宽列存储数据库，这意味着它以灵活的模式存储数据在表中，并支持列族数据模型。我们将学习如何执行高级查询，以及如何在其中管理乐观并发持久性。

在本章中，我们将涵盖以下菜谱：

+   将您的应用程序连接到 MongoDB

+   使用 Testcontainers 与 MongoDB

+   MongoDB 中的数据索引和分片

+   在 MongoDB 中使用事务

+   使用 MongoDB 管理并发

+   将您的应用程序连接到 Apache Cassandra

+   使用 Testcontainers 与 Apache Cassandra

+   使用 Apache Cassandra 模板

+   使用 Apache Cassandra 管理并发

# 技术要求

对于本章，您需要一个 MongoDB 服务器和一个 Apache Cassandra 服务器。在两种情况下，在您的本地环境中部署它们的最简单方法是通过使用 Docker。您可以从其产品页面[`www.docker.com/products/docker-desktop/`](https://www.docker.com/products/docker-desktop/)获取 Docker。我将在相应的菜谱中解释如何使用 Docker 安装 MongoDB 和 Cassandra。

如果您想在您的计算机上安装 MongoDB，可以遵循产品页面上的安装说明：[`www.mongodb.com/try/download/community`](https://www.mongodb.com/try/download/community)。

如果您需要访问 MongoDB，您可以使用 MongoDB Shell 或 MongoDB Compass，这两个都可以在[`www.mongodb.com/try/download/tools`](https://www.mongodb.com/try/download/tools)找到。我将在本章中使用 MongoDB Shell，因此我建议您安装它。

对于 Cassandra，您可以遵循[`cassandra.apache.org/doc/latest/cassandra/getting_started/installing.html`](https://cassandra.apache.org/doc/latest/cassandra/getting_started/installing.html)中的说明。

本章将展示的所有菜谱都可以在[`github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/tree/main/chapter6`](https://github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/tree/main/chapter6)找到。

# 将您的应用程序连接到 MongoDB

在这个菜谱中，我们将学习如何在 Docker 中部署 MongoDB 服务器。接下来，我们将创建一个 Spring Boot 应用程序，并使用 Spring Data MongoDB 将其连接到我们的 MongoDB 服务器。最后，我们将初始化数据库并对已加载的数据执行一些查询。

我们将通过足球队伍和球员的场景来展示在 MongoDB 中管理数据的不同方法，与诸如 PostgreSQL 这样的关系型数据库相比。

## 准备工作

对于这个菜谱，我们将使用 MongoDB 数据库。在您的计算机上部署它的最简单方法是使用 Docker。您可以从[`www.docker.com/products/docker-desktop/`](https://www.docker.com/products/docker-desktop/)的产品页面下载 Docker。

安装 Docker 后，您可以运行一个 MongoDB 的单实例或执行一个运行在副本集中的集群。在这里，您将部署一个运行在副本集中的集群。对于这个菜谱来说这不是必要的，但对于后续的菜谱来说却是必要的，因为它需要支持事务。我已经准备了一个脚本以简化集群的部署。这个脚本使用`docker-compose`部署集群；一旦部署，它将初始化副本集。您可以在本书的 GitHub 仓库[`github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/`](https://github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/)中的`chapter3/recipe3-2/start`文件夹中找到这个脚本。

您需要 MongoDB Shell 来连接到 MongoDB 服务器。您可以从[`www.mongodb.com/try/download/shell`](https://www.mongodb.com/try/download/shell)下载它。

您还需要*mongoimport*工具将一些数据导入数据库。它是 MongoDB 数据库工具的一部分。按照产品页面上的说明进行安装：[`www.mongodb.com/docs/database-tools/installation/installation/`](https://www.mongodb.com/docs/database-tools/installation/installation/)。

数据加载后，将看起来像这样：

```java
{
    "_id": "1884881",
    "name": "Argentina",
    "players": [
         {
             "_id": "199325",
             "jerseyNumber": 1,
             "name": "Vanina CORREA",
             "position": "Goalkeeper",
             "dateOfBirth": "1983-08-14",
             "height": 180,
             "weight": 71
        },
        {
             "_id": "357669",
             "jerseyNumber": 2,
             "name": "Adriana SACHS",
             "position": "Defender",
             "dateOfBirth": "1993-12-25",
             "height": 163,
             "weight": 61
        }
    ]
}
```

每个队伍都有一个球员列表。记住这个结构，以便更好地理解这个菜谱。

您可以使用*MongoDB Shell*连接到数据库。我们将使用它来创建一个数据库，并用一些数据初始化它。您可以在本书的 GitHub 仓库[`github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/`](https://github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/)中找到加载数据的脚本。脚本和数据位于`chapter3/recipe3-1/start/data`文件夹中。

## 如何操作...

让我们使用 Spring Data MongoDB 创建一个项目，并创建一个存储库来连接到我们的数据库。数据库管理足球团队，包括球员。我们将创建一些查询来获取团队和球员，并将实现操作来更改我们的数据。按照以下步骤操作：

1.  使用 *Spring Initializr* 工具创建一个项目。打开 [`start.spring.io`](https://start.spring.io) 并使用与 *第一章* 中 *创建 RESTful API* 菜谱相同的参数，除了以下选项：

    +   对于 `footballmdb`

    +   对于 **依赖项**，选择 **Spring Web** 和 **Spring Data MongoDB**

1.  下载使用 *Spring Initializr* 工具生成的模板，并将其内容解压缩到您的工作目录中。

1.  首先，我们将配置 Spring Data MongoDB 以连接到我们的数据库。为此，在 `resources` 文件夹中创建一个 `application.yml` 文件。它应该看起来像这样：

    ```java
    spring:
        data:
            mongodb:
                uri: mongodb://127.0.0.1:27017/?directConnection=true
                database: football
    ```

1.  现在，创建一个名为 `Team` 的类，并使用 `@Document(collection = teams)` 进行注解。它应该看起来像这样：

    ```java
    @Document(collection = "teams")
    public class Team {
        @Id
        private String id;
        private String name;
        private List<Player> players;
    }
    ```

    注意，我们还用 `@Id` 装饰了属性 ID，并在我们的类中使用 `List<Player>`。在 MongoDB 中，我们将有一个名为 `teams` 的单个数据集合。每个团队将包含球员。

1.  接下来，创建 `Player` 类：

    ```java
    public class Player {
        private String id;
        private Integer jerseyNumber;
        private String name;
        private String position;
        private LocalDate dateOfBirth;
        private Integer height;
        private Integer weight;
    }
    ```

    `Player` 类不需要任何特殊注解，因为它的数据将被嵌入到 `Team` 文档中。

1.  现在，创建一个用于管理在 MongoDB 中持久化的团队的存储库：

    ```java
    public interface TeamRepository extends MongoRepository<Team,
    String>{
    }
    ```

1.  就像 `JpaRepository` 一样，只需通过从 `MongoRepository` 扩展我们的 `TeamRepository` 接口，我们就已经有了在 MongoDB 中操作 `Team` 文档的基本方法。我们现在将使用这个存储库。为此，创建一个名为 `FootballService` 的新服务：

    ```java
    @Service
    public class FootballService {
        private TeamRepository teamRepository;
        public FootballService(TeamRepository teamRepository) {
            this.teamRepository = teamRepository;
        }
    }
    ```

    现在，我们可以在我们的服务中创建一个新的方法，用于通过其 `Id` 值检索一个团队。这个服务中的方法可以使用 `TeamRepository` 中的 `findById` 方法，该方法是通过对 `MongoRepository` 进行扩展而可用的：

    ```java
    public Team getTeam(String id) { 
        return teamRepository.findById(id).get();
    }
    public Optional<Team> findByName(String name);
    ```

    我们还可以创建一个方法来查找包含字符串的团队名称：

    ```java
    public List<Team> findByNameContaining(String name);
    ```

1.  现在，我们将创建一个用于查找球员的方法。为此，我们需要查看团队以找到球员。可以通过使用 `@Query` 注解来实现：

    ```java
    @Query(value = "{'players._id': ?0}", fields = "{'players.$': 1}")
    public Team findPlayerById(String id);
    ```

1.  如您所见，查询的 `value` 属性不是 SQL，它在 `fields` 属性中对应于我们想要从文档中检索的字段——在这种情况下，只是文档的 `players` 字段。此方法将返回一个只包含一个球员的 `Team` 对象。

    让我们看看如何使用这个方法。为此，在 `FootballService` 中创建一个名为 `findPlayerById` 的方法：

    ```java
    public Player getPlayer(String id) {
        Team team = teamRepository.findPlayerById(id);
        if (team != null) {
            return team.getPlayers().isEmpty()
                                         ? null
                                         : team.getPlayers().get(0);
        } else {
            return null;
        }
    }
    ```

    我们将使用 `MongoRepository` 的 `save` 方法来 *upsert* 团队，以及使用 `delete`/`deleteById` 来在数据库中做出更改：

    +   `FootballService` 类中的 `saveTeam`：

        ```java
        public Team saveTeam(Team team) {
            return teamRepository.save(team);
        }
        ```

    +   现在，创建一个通过其 ID 删除团队的方法：

        ```java
        public void deleteTeam(String id) {
            teamRepository.deleteById(id);
        }
        ```

在这个菜谱中，我们实现了一个使用 `MongoRepository` 来执行与我们的 MongoDB 数据库交互的基本操作的服务。我已经创建了一个 RESTful API 来公开由本菜谱中创建的 `FootballService` 服务实现的方法。我还创建了一个脚本来向 RESTful API 发送请求。您可以在本书的 GitHub 仓库中找到所有这些内容，在 `chapter6/reciper6-1/end` 文件夹中。

## 它是如何工作的...

当应用程序启动时，Spring Data MongoDB 会扫描应用程序以查找 `MongoRepository` 接口。然后，它为存储库中定义的方法生成实现，并将接口实现注册为 bean 以使其对应用程序的其余部分可用。为了推断接口的实现，它使用方法的命名约定；有关更多详细信息，请参阅 https://docs.spring.io/spring-data/mongodb/docs/current/reference/html/#repository-query-keywords。Spring Data MongoDB 还会扫描带有 `@Query` 注解的方法以生成这些方法的实现。

关于 `@Query` 注解，Spring Data MongoDB 可以执行某些验证，但你应该记住 MongoDB 是按设计灵活的。这意味着它不假设某个字段应该存在或不存在。它将返回一个 `null` 值。请注意，如果结果与您预期的不同，您的查询可能存在拼写错误。

在 `findPlayerById` 中，我们实现了一个查询以返回文档中的数组元素。理解 MongoDB 返回的数据非常重要。当我们想要找到球员 `430530` 时，它返回一个容器文档，一个具有 `id` 值为 1882891 的 `Team` 对象，仅包含属性 `players` 和一个仅包含一个元素的数组 - 即具有 ID `430530` 的球员。它看起来像这样：

```java
[
  {
    "_id": "1882891",
    "players": [
      {
        "_id": "430530",
        "jerseyNumber": 2,
        "name": "Courtney NEVIN",
        "position": "Defender",
        "dateOfBirth": {
          "$date": "2002-02-11T23:00:00Z"
        },
        "height": 169,
        "weight": 64
      }
    ]
  }
]
```

注意

我为了学习目的在团队集合中包含了球员。如果您有类似的场景，并且您在集合中搜索数组元素时预期将执行大量查询，您可能更喜欢为该数组拥有一个 MongoDB 集合。在这种情况下，我会将球员存储在它们自己的集合中。这将执行得更好，并且可扩展性更强。

在这里，`MongoRepository` 提供了三种方法来保存数据：

+   `save`: 此方法如果文档不存在则插入文档，如果文档已存在则替换它。这种行为也称为 *upsert*。

+   `saveAll`: 此方法的行为与 `save` 相同，但它允许您同时持久化多个文档。

+   `insert`: 此方法向集合中添加一个新的文档。因此，如果文档已存在，它将失败。此方法针对插入操作进行了优化，因为它不会检查文档的先前存在。

`save` 和 `saveAll` 方法会完全替换已存在的文档。如果你只想更新实体的一些属性，也称为部分文档更新，你需要使用 Mongo 模板。

## 更多...

我建议在更高级的场景中查看 `MongoTemplate`，例如当你需要部分更新时。以下是一个示例，如果你只想更新团队名称：

```java
public void updateTeamName(String id, String name) {
    Query query = new Query(Criteria.where("id").is(id));
    Update updateName = new Update().set("name", name);
mongoTemplate.updateFirst(query, updateName, Team.class);
}
```

如你所见，它允许你定义查询对象的 `where` 条件，并允许 *更新* 操作，定义你想要更新的字段。在这里，`MongoTemplate` 是 Spring Data MongoDB 用于创建 `MongoRepository` 接口实现的核心理念组件。

# 使用 Testcontainers 与 MongoDB

当创建依赖于 MongoDB 的集成测试时，我们有两种选择：在我们的应用程序中嵌入一个内存数据库服务器或使用 Testcontainers。内存数据库服务器可能与我们的生产系统略有不同。出于这个原因，我建议使用 Testcontainers；它允许你使用一个在 Docker 中托管并启用所有功能的真实 MongoDB 数据库。

在这个菜谱中，我们将学习如何设置 MongoDB Testcontainer 以及如何执行一些初始化脚本，以便我们可以将测试数据插入到数据库中。

## 准备工作

执行 Testcontainers 需要一个与 Docker-API 兼容的运行时。你可以通过遵循官方网页上的说明来安装 Docker：[`www.docker.com/products/docker-desktop/`](https://www.docker.com/products/docker-desktop/)。

在这个菜谱中，我们将为在 *将你的应用程序连接到 MongoDB* 菜谱中创建的项目添加测试。我已创建了一个可工作的版本，以防你还没有完成它。你可以在本书的 GitHub 仓库中找到它，在 [`github.com/PacktPublishing/Spring-Boot-3.0-Cookbook`](https://github.com/PacktPublishing/Spring-Boot-3.0-Cookbook) 的 `chapter6/recipe6-2/start` 文件夹中。在这个文件夹中，你还会找到一个名为 `teams.json` 的文件。这将用于初始化测试数据。

## 如何做...

让我们通过使用 Testcontainers 创建自动化测试来增强我们的项目：

1.  首先，我们需要包含 Testcontainers 依赖项。为此，打开 `pom.xml` 文件并添加以下依赖项：

    ```java
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>junit-jupiter</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>mongodb</artifactId>
        <scope>test</scope>
    </dependency>
    ```

1.  由于我们需要在测试执行期间初始化数据库中的数据，将 *准备阶段* 中描述的 `team.json` 文件复制到 `tests/resources/mongo` 文件夹中。

1.  接下来，创建一个测试类。让我们称它为 `FootballServiceTest`，并使用 `@SpringBootTest` 和 `@Testcontainers` 注解该类：

    ```java
    @SpringBootTest
    @Testcontainers
    class FootballServiceTest
    ```

1.  我们将继续设置测试类，通过创建 MongoDB 容器。正如我们将在下一步看到的，我们需要用一些数据初始化数据库。为此，我们将把 *步骤 2* 中描述的 `teams.json` 文件复制到容器中。我们将创建容器并按以下方式传递文件：

    ```java
    static MongoDBContainer mongoDBContainer = new MongoDBContainer("mongo").withCopyFileToContainer(
       MountableFile.teams.json file. To import the data, we’ll use the *mongoimport* tool:

    ```

    @BeforeAll

    static void startContainer() throws IOException, InterruptedException {

    mongoDBContainer.start();

    importFile("teams");

    }

    static void importFile(String fileName) throws IOException, InterruptedException {

    Container.ExecResult res = mongoDBContainer.execInContainer("mongoimport", "--db=football", "--collection=" + fileName, "--jsonArray", fileName + ".json");

    if (res.getExitCode() > 0){

    throw new RuntimeException("MongoDB not properly initialized");

    }

    }

    ```java

    Note that this step should be performed before the tests start. That’s why it’s annotated with `@BeforeAll`.
    ```

1.  现在，我们应该配置上下文，使其使用在 Testcontainers 中托管的 MongoDB 数据库。为此，我们将使用`@DynamicPropertySource`注解：

    ```java
    @DynamicPropertySource
    static void setMongoDbProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.data.mongodb.uri", mongoDBContainer::getReplicaSetUrl);
    }
    ```

1.  现在 MongoDB 仓库已经配置好了，我们可以继续进行正常的测试实现。让我们将`FootballService`注入到测试类中，并实现一个简单的测试，用于检索`Team`对象：

    ```java
    @Autowired
    private FootballService footballService;
    @Test
    void getTeam() {
       Team team = footballService.getTeam("1884881");
       assertNotNull(team);
    }
    ```

1.  您可以实现其余功能的测试。我为`FootballService`类创建了一些基本的测试。您可以在本书的 GitHub 仓库中找到它们，网址为[`github.com/PacktPublishing/Spring-Boot-3.0-Cookbook`](https://github.com/PacktPublishing/Spring-Boot-3.0-Cookbook)，在`chapter6/recipe6-2/end`文件夹中。

## 工作原理...

正如我们在*第五章*的*使用 Testcontainers 进行 PostgreSQL 集成测试*配方中看到的，通过添加`@Testcontainers`注解，所有声明为静态的容器都可用于类中的所有测试，并在最后一个测试执行后停止。在这个配方中，我们使用了专门的`MongoDBContainer`容器；它提供了服务器的 URL，我们可以用它来配置测试上下文。这种配置是通过使用`@DynamicPropertySource`注解来完成的，正如我们在*步骤 6*中看到的。

在这个配方中，我们学习了如何将文件复制到容器中并在其中执行程序。`resources`文件夹中的所有文件都在运行时可用。我们将`teams.json`文件复制到容器中，然后使用`mongoimport`工具将数据导入 MongoDB。这个工具在 MongoDB Docker 镜像中可用。在容器中执行此工具的一个优点是不需要指定数据库服务器地址。

# MongoDB 中的数据索引和分片

在这个配方中，我们将管理足球比赛及其时间线——即比赛期间发生的事件。一个事件可能涉及一名或两名球员，我们必须考虑到球员的粉丝想要访问所有涉及他们最喜欢的球员的行动。我们还将考虑比赛及其事件的数量每天都在增长，因此我们需要准备我们的应用程序以支持所有负载。

在这个菜谱中，我们将介绍一些关键概念，以使您的应用程序具有高性能和可扩展性。MongoDB，就像关系数据库一样，允许您创建索引以优化数据访问。如果您计划使用相同的参数访问某些数据，创建索引以优化数据读取是值得的。当然，您将需要更多的存储和内存，并且写操作将受到影响。因此，您需要计划和分析您的应用程序需求。

随着您数据量的增加，您将需要扩展您的 MongoDB 数据库。**分片**是一种数据库架构和分区技术，用于在分布式系统中的多个服务器或节点上水平分区数据。通过分片，您可以通过添加更多服务器并将数据分布到它们上（使用分片）来扩展数据库。分片确保同一分片中的所有数据都将位于同一服务器上。

在这个菜谱中，我们将使用索引和分片在我们的足球应用程序中，同时利用 Spring Data MongoDB 提供的功能。我们将使用 Spring Data MongoDB 的其他有趣功能，例如从其他文档引用文档。

## 准备工作

我们将使用与第一个菜谱中相同工具，*将您的应用程序连接到 MongoDB* —— 也就是说，Docker、MongoDB 以及 MongoDB 工具，如*Mongo Shell*和*mongoimport*。

我们将重用*将您的应用程序连接到 MongoDB*菜谱中的代码。如果您还没有完成它，不要担心——我已经在这个书的 GitHub 仓库中准备了一个工作版本，网址为[`github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/`](https://github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/)。您可以在`chapter6/recipe6-` `3/start`中找到它。我还创建了一个脚本，用于使用*mongoimport*工具将数据加载到数据库中。您可以在`chapter6/recipe6-3/start/data`文件夹中找到它。

与*将您的应用程序连接到 MongoDB*菜谱中提供的数据相比，数据略有不同。我将球员移动到了他们自己的 MongoDB 集合中，并添加了新的集合来管理比赛和比赛事件。如果您想保留上一个菜谱中的数据，我建议您为这个菜谱创建一个新的数据库。您可以通过简单地更改调用*mongoimport*工具时的`--db`参数来实现这一点。调用将如下所示：

```java
mongoimport --uri="mongodb://127.0.0.1:27017/?directConnection=true"
--db=football2 --collection=teams --jsonArray < teams.json
```

## 如何操作...

首先，我们将把球员的数据托管在他们自己的 MongoDB 集合中。球员对于新的需求来说将是重要的实体，因此他们应拥有自己的集合。然后，我们将创建比赛和事件的文档类。我们将学习如何使用 Spring Data MongoDB 注解来配置 MongoDB *索引*和*分片*。按照以下步骤操作：

1.  让我们从配置球员自己的 MongoDB 集合开始。使用`@Document`注释`Player`类：

    ```java
    @Document(collection = "players")
    public class Player {
        @Id
    private String id;
    }
    ```

    使用`@Id`注解注释`id`字段。我们这样做是因为它将是文档标识符。

    现在，从 `Team` 中删除 `players` 字段。

1.  接下来，创建比赛及其事件的类。对于比赛，我们将创建一个名为 `Match` 的类：

    ```java
    @Document(collection = "matches")
    public class Match {
        @Id
        private String id;
        private LocalDate matchDate;
        @Indexed
        @DBRef(lazy = false)
        private Team team1;
        @Indexed
        @DBRef(lazy = false)
        private Team team2;
        private Integer team1Goals;
        private Integer team2Goals;
    }
    ```

    注意，我们开始使用两个新的注解，`@Indexed` 和 `@DBRef`。它们将在本食谱的 *How it works...* 部分中完全解释。

    对于比赛事件，我们将创建一个名为 `MatchEvent` 的类：

    ```java
    @Sharded(shardKey = { "match" })
    @Document(collection = "match_events")
    public class MatchEvent {
        @Id
        private String id;
        @Field(name = "event_time")
        private LocalDateTime time;
        private Integer type;
        private String description;
        @Indexed
        @DBRef
        private Player player1;
        @Indexed
        @DBRef
        private Player player2;
        private List<String> mediaFiles;
        @DBRef
        private Match match;
    }
    ```

    通过这样，我们介绍了 `@Sharded` 和 `@Field` 注解。

1.  为了能够使用新类，我们将为每个类创建一个存储库——即 `PlayerRepository`、`MatchRepository` 和 `MatchEventRepository`。

    让我们详细看看 `MatchEventRepository`。它将实现我们所需的要求：

    +   返回比赛中的所有事件

    +   返回比赛中的所有球员事件：

        ```java
        public interface MatchEventRepository extends MongoRepository<MatchEvent, String>{
            @Query(value = "{'match.$id': ?0}")
            List<MatchEvent> findByMatchId(String matchId);
            @Query(value = "{'$and': [{'match.$id': ?0}, {'$or':[
        {'player1.$id':?1}, {'player2.$id':?1} ]}]}")
            List<MatchEvent> findByMatchIdAndPlayerId(String matchId, String playerId);
        }
        ```

1.  到目前为止，我们可以运行我们的应用程序，因为 Spring Data MongoDB 组件已经就绪。然而，并非所有索引都已被创建。如果我们想在应用程序中创建它们，我们需要创建一个配置类，该类扩展 `AbstractMongoClientConfiguration`，指示 Spring Mongo DB 自动创建索引：

    ```java
    @Configuration
    public class MongoConfig extends
    AbstractMongoClientConfiguration {
        @Override
        protected boolean autoIndexCreation() {
            return true;
        }
    }
    ```

1.  现在，我们可以使用这些存储库创建一个服务，以实现我们应用程序的新要求，同时以优化的方式连接到 MongoDB。我已经创建了一个服务和 RESTful 控制器来演示这些存储库的使用。我还使用 Testcontainers 添加了一些测试。您可以在本书的 GitHub 仓库中找到它们，网址为 [`github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/`](https://github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/)，在 `chapter6/recipe6-3/end` 文件夹中。

## How it works...

首先，我将解释在这个食谱中使用到的注解的影响。

`@DBRef` 注解是引用另一个文档的一种方式，但请记住，这是一个由 Spring Data MongoDB 实现的机制，而不是由数据库引擎本身实现的。在 MongoDB 中，引用完整性的概念不存在，它应该在应用程序级别进行管理。在这里，`@DBRef` 将文档表示为一个具有三个字段的对象：

+   `$ref`：这包含被引用的集合

+   `$id`：这包含被引用文档的 ID

+   `$db`：这包含被引用文档的数据库

例如，这里有一个对团队 `1882891` 的引用：

```java
{
    "$ref": "teams",
    "$id": "1882891",
    "$db": "football"
}
```

Spring Data MongoDB 可以使用这个注解来自动检索引用的文档。我们可以使用 `lazy` 属性来指定这种行为。默认情况下，它是 `true`，这意味着 Spring Data MongoDB 不会自动检索它。如果您将其设置为 `false`，它将自动检索引用的文档。我们使用这个注解来检索比赛文档，以自动检索比赛两支队伍的信息。

`@Indexed` 注解，正如你可能已经猜到的，在 MongoDB 中创建了一个索引。然后，使用索引字段的查询将更快地执行读操作。

`@Sharded` 注解告诉 MongoDB 如何将集合分布到各个分片中。集群中的服务器可以托管一个或多个分片。我们也可以将分片视为指定哪些文档将托管在同一个服务器上的方式。在我们的案例中，我们感兴趣的是通过匹配检索事件。这就是我们配置 `match` 作为分片键的原因。选择一个好的分片键对于使我们的应用程序性能良好和可扩展至关重要，因为它将影响工作负载在服务器之间的分布方式。

当在一个分片集合中执行查询时，MongoDB 应该确定该请求是否可以在单个分片中执行，或者是否需要将查询分散到多个分片。它将从分片中收集结果，进行聚合，然后将结果返回给客户端。如果你故意需要水平扩展查询，这是一个非常好的机制。可能发生的情况是，请求不需要分散，可以在单个分片中执行，但由于分片键选择错误，它被当作分布式查询执行。结果是，它将消耗比预期更多的资源，因为更多的服务器将执行不必要的查询，因此需要聚合结果。

分片涉及将数据库划分为更小的部分，称为分片，这些分片可以托管在单个服务器上。一个服务器可以托管多个分片，并且分片会在服务器之间进行复制以提高可用性。服务器的数量可以根据负载自动增加或减少。分片对于管理大型数据集和大型集群非常有用，这些集群通常部署在云中。例如，**MongoDB Atlas** 可以托管在云提供商，如 **Azure**、**Amazon Web Services**（**AWS**）和**Google Cloud Platform**（**GCP**）上，允许调整服务器的数量以满足实际需求。然而，在数据库托管在计算机上的单个容器中，如我们的示例中，分片不会提供任何显著的好处。在更大的部署中，分片是实现我们目标的关键特性。

我们没有在 `MatchEvent` 中显式创建 `match` 的索引，但由于它是分片键，它被隐式创建。

最后，我们使用了 `@Field` 注解。这用于将我们的文档类中的一个字段映射到 MongoDB 中的不同字段。在我们的案例中，我们将类中的 `time` 字段映射到 MongoDB 中的 `event_time` 字段。

## 还有更多...

在使用 MongoDB 或其他面向文档的数据库设计数据层时，应做出一些决策。例如，我们应该在同一个集合中混合不同类型的对象，还是应该将每种类型的文档保存在不同的集合中？

在同一个集合中拥有不同类型的对象，如果它们共享一些公共字段并且您想通过这些字段执行查询，或者您想从不同的对象中聚合数据，这是有意义的。对于其他场景，可能更好的是将每种类型的文档放在其自己的集合中。这有助于创建索引并促进分片创建。

在这个菜谱中，我们没有混合不同类型的文档，这也是 Spring Data MongoDB 在持久化文档时引入名为`_class`的字段的原因。例如，这是在创建新队伍时持久化的文档：

```java
{
  "_id": "99999999",
  "name": "Mars",
  "_class": "com.packt.footballmdb.repository.Team"
}
```

另一个需要做出的决定是，我们是否应该在文档中嵌入一些数据，或者这些数据应该在其自己的文档中。在*将您的应用程序连接到 MongoDB*菜谱中，我们将球员嵌入到他们的队伍中，而在这个菜谱中，我们将该信息移动到其自己的集合中。这可能取决于可嵌入文档的重要性或独立性。在这个菜谱中，球员需要自己的文档，因为它们可以直接从其他文档中引用，例如比赛事件。

可能还有其他原因，例如对嵌入式实体的预期写并发性。例如，我们可以在比赛中嵌入事件。然而，在比赛期间，我们可以假设会有大量事件发生。这个操作将需要在比赛文档上进行大量写操作，这将需要更多的一致性管理。

# 在 MongoDB 中使用事务

我们希望创建一个新的服务，用户可以购买一个虚拟代币，该代币可以用来获取这个新游戏中的虚拟商品。主要商品是带有玩家图片和其他信息的卡片，一种虚拟贴纸。

我们需要实现两个操作：代币购买和卡片购买。对于代币购买，有一个支付验证。卡片只能用代币购买。当然，如果用户有足够的代币，他们也将能够购买卡片。

由于我们需要确保代币和卡片余额的一致性，我们将需要使用事务与我们的 MongoDB 存储库一起使用。

在这个菜谱中，我们将了解更多的 MongoDB 事务以及它们与关系型数据库事务的不同之处。

## 准备工作

我们将使用与*将您的应用程序连接到 MongoDB*菜谱中相同的工具——即 Docker 和 MongoDB。

我们将重用*在 MongoDB 中数据索引和分片*菜谱中的代码。如果您还没有完成它，不要担心——我已经在这个书的 GitHub 仓库中准备了一个工作版本，在[`github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/`](https://github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/)。它可以在`chapter6/recipe6-4/start`文件夹中找到。

MongoDB 事务

MongoDB 事务在独立服务器上不受支持。在 *将您的应用程序连接到 MongoDB* 配方中，我提供了一个使用副本集部署集群的脚本。

在 *在 Testcontainers 中部署 MongoDB 集群* 的配方中，我们将介绍如何使用容器部署多个服务器以测试 MongoDB 事务。

## 如何做到这一点...

我们需要创建一个数据模型来支持我们新服务中的用户和卡片。稍后，我们将创建一个使用 MongoDB 事务来执行涉及用户和卡片的操作一致性的服务。我们将配置我们的应用程序以支持事务。按照以下步骤操作：

1.  让我们从创建将存储在 MongoDB 中的对象的管理类开始：

    1.  首先，我们将创建一个名为 `User` 的类：

    ```java
    @Document(collection = "users")
    public class User {
        @Id
        private String id;
        private String username;
        private Integer tokens;
    }
    ```

    1.  接下来，我们将创建一个名为 `Card` 的类：

    ```java
    @Document(collection = "cards")
    public class Card {
        @Id
        private String id;
        @DBRef
        private Player player;
        @DBRef
        private User owner;
    }
    ```

1.  接下来，我们需要创建相应的 `MongoRepository` 接口。让我们开始吧：

    1.  创建一个名为 `UserRepository` 的接口：

    ```java
    public interface UserRepository extends MongoRepository<User, String>{
    }
    ```

    1.  另外一个名为 `CardRepository` 的接口：

    ```java
    public interface CardRepository extends MongoRepository<Card, String>{
    }
    ```

1.  现在，我们需要创建一个服务类来管理我们应用程序的业务逻辑。为此，创建一个名为 `UserService` 的类。请记住用 `@Service` 注解该类：

    ```java
    @Service
    public class UserService {
    }
    ```

1.  此服务将需要我们创建的新存储库——即 `UserRepository` 和 `CardRepository`，以及我们在 *MongoDB 中的数据索引和分片* 配方中创建的 `PlayerRepository`。我们还需要 `MongoTemplate`。我们将创建一个包含这些存储库的构造函数，之后 Spring Boot 依赖项管理器将注入它们：

    ```java
    @Service
    public class UserService {
        private UserRepository userRepository;
        private PlayerRepository playersRepository;
        private CardRepository cardsRepository;
        private MongoTemplate mongoTemplate;
        public UserService(UserRepository userRepository,
                           PlayerRepository playersRepository,
                           CardRepository cardsRepository,
                           MongoTemplate mongoTemplate) {
            this.userRepository = userRepository;
            this.playersRepository = playersRepository;
            this.cardsRepository = cardsRepository;
            this.mongoTemplate = mongoTemplate;
        }
    ```

1.  接下来，我们将实现我们的业务逻辑：

    1.  创建一个名为 `buyTokens` 的购买令牌的方法：

    ```java
    public Integer buyTokens(String userId, Integer tokens) {
        Query query = new Query(Criteria.where("id").is(userId));
        Update update = new Update().inc("tokens", tokens);
        UpdateResult result = mongoTemplate.updateFirst(query, update, User.class, "users");
        return (int) result.getModifiedCount();
    }
    ```

    1.  创建一个名为 `buyCards` 的购买卡片的方法：

    ```java
    @Transactional
    public Integer buyCards(String userId, Integer count) {
        Optional<User> userOpt = userRepository.findById(userId);
        if (userOpt.isPresent()) {
            User user = userOpt.get();
            List<Player> availablePlayers = getAvailablePlayers();
            Random random = new Random();
            if (user.getTokens() >= count) {
                user.setTokens(user.getTokens() - count);
            } else {
                throw new RuntimeException("Not enough tokens");
            }
            List<Card> cards = Stream.generate(() -> {
                Card card = new Card();
                card.setOwner(user);
                card.setPlayer(availablePlayers.get(
                        random.nextInt(0,
                                availablePlayers.size())));
                return card;
            }).limit(count).toList();
            List<Card> savedCards = cardsRepository.saveAll(cards);
            userRepository.save(user);
            return savedCards.size();
        }
        return 0;
    }
    ```

1.  为了在我们的应用程序中允许事务，我们需要注册一个 `MongoTransactionManager` 实例。为此，在我们的 `MongoConfig` 类中添加以下方法：

    ```java
    @Bean
    MongoTransactionManager
    transactionManager(MongoDatabaseFactory dbFactory) { 
        return new MongoTransactionManager(dbFactory); 
    }
    ```

现在，我们的应用程序可以使用事务来原子性地执行操作。

## 它是如何工作的...

默认情况下，Spring Data MongoDB 中禁用了 MongoDB 原生事务。这就是为什么我们需要注册 `MongoTransactionManager` 的原因。一旦配置完成，当我们用 `@Transactional` 注解一个方法时，它将创建一个事务。

非常重要的是要注意，事务提供原子操作，这意味着所有操作要么全部保存，要么全部不保存，但它们不支持隔离。`buyCards` 方法将保存所有 `cards` 和 `user` 上的更改，或者它将保存所有这些更改。

与关系型数据库中的事务相比，一个重要的区别是没有锁定或隔离。如果我们修改了在另一个请求中 `buyCards` 修改的同一 `User` 文档，它将引发一个 **写冲突异常**。MongoDB 是以性能和可扩展性为代价，牺牲了 ACID 事务的功能而设计的。我们将在 *使用 MongoDB 管理并发* 配方中更详细地学习如何管理并发。

如您可能已经意识到的那样，`buyTokens` 方法不使用事务。主要原因是不需要这样做。单个文档中的所有操作都被视为隔离和原子的。由于唯一更新的字段是 `tokens`，我们使用了 `inc` 操作来修改值。这个操作器的优点是它在服务器上以原子方式执行，即使在高并发环境中也是如此。如果我们对涉及单个文档的操作使用事务，当两个请求正在更新同一文档时，可能会引发写冲突异常。如果您将其与关系型数据库中事务的行为进行比较，这种行为可能会显得有些反直觉。

## 相关内容

除了 `$inc` 之外，MongoDB 中还有其他适用于并发场景的原子操作值得了解。它们可以应用于字段和数组。有关更多详细信息，请参阅 [`www.mongodb.com/docs/v7.0/reference/operator/update/`](https://www.mongodb.com/docs/v7.0/reference/operator/update/)。

# 在 Testcontainers 中部署 MongoDB 集群

MongoDB 事务仅在多服务器集群中受支持。然而，正如 *使用 Testcontainers 与 MongoDB* 配方中解释的那样，`MongoDBContainer` 使用的是单个服务器部署。因此，我们无法用它来对新功能的购买卡片集成测试进行测试，因为它需要事务。

在这个配方中，我们将学习如何设置多个 Testcontainers 并配置 MongoDB 集群。有了这个，我们将能够实现购买卡片功能的集成测试。

## 准备工作

这个配方将实现 *在 MongoDB 中使用事务* 配方的集成测试。如果您还没有完成它，不用担心——我已经准备了一个版本，您可以从这个版本开始这个配方。您可以在本书的 GitHub 仓库中找到它，网址为 [`github.com/PacktPublishing/Spring-Boot-3.0-Cookbook`](https://github.com/PacktPublishing/Spring-Boot-3.0-Cookbook)，在 `chapter6/recipe6-5/start` 目录下。

## 如何操作...

在这个配方中，我们将使用 Testcontainers 设置 MongoDB 集群并测试涉及事务的功能。让我们开始吧！

1.  由于新的购买卡片功能是，我们将创建一个新的测试类 `UserServiceTest` 并在这个类中设置一切。由于它使用 Testcontainers，我们将使用 `@Testcontainers` 注解这个类：

    ```java
    @SpringBootTest
    @Testcontainers
    class UserServiceTest
    ```

1.  接下来，我们将创建 MongoDB 集群。它将由三个在同一网络中部署的 MongoDB 容器组成：

    1.  声明一个 `Network` 静态字段。这个类是 Testcontainers 库的一部分，它允许我们定义一个 Docker 网络：

    ```java
    static Network mongoDbNetwork = Network.newNetwork();
    ```

    1.  现在，创建三个具有以下属性的静态 `GenericContainer` 字段：

        +   每个字段将使用最新的 `mongo` Docker 镜像。

        +   每个字段将具有相同的网络。

        +   这三个容器将公开端口 `27017`。

        +   每个容器将具有不同的网络别名：`mongo1`、`mongo2` 和 `mongo3`。

        +   三个容器将以`mongod`命令启动，该命令初始化 MongoDB 集群，唯一的区别是绑定 IP 主机名。每个容器将使用其网络别名。

    1.  这里，我们有第一个字段，`mongoDBContainer1`：

    ```java
    static GenericContainer<?> mongoDBContainer1 = new GenericContainer<>("mongo:latest")
        .withNetwork(mongoDbNetwork)
        .withCommand("mongod", "--replSet", "rs0", "--port", "27017", "--bind_ip", "localhost,mongo1")
        .withNetworkAliases("mongo1")
        .withExposedPorts(27017);
    ```

    1.  其他字段，`mongoDBContainer2`和`mongoDBContainer3`，与`mongoDBContainer1`声明相同，但我们必须将`mongo1`分别更改为`mongo2`和`mongo3`。

1.  现在已经声明了三个 MongoDB 容器，下一步是启动容器并初始化 MongoDB 副本集。我们需要在服务器上执行以下 MongoDB 命令：

    ```java
    rs.initiate({
       _id: "rs0",
       members: [
           {_id: 0, host: "mongo1"},
           {_id: 1, host: "mongo2"},
           {_id: 2, host: "mongo3"}
       ]})
    ```

    我创建了一个名为`buildMongoEvalCommand`的实用方法，用于格式化命令，以便它们可以在 MongoDB 中执行。我们将在任何测试执行之前执行 MongoDB 副本集初始化。为此，我们将使用`@BeforeAll`注解：

    ```java
    String initCluster = """
                    rs.initiate({
                     _id: "rs0",
                     members: [
                       {_id: 0, host: "mongo1"},
                       {_id: 1, host: "mongo2"},
                       {_id: 2, host: "mongo3"}
                     ]
                    })
                    """;
    mongoDBContainer1.start();
    mongoDBContainer2.dependsOn(mongoDBContainer1).start();
    mongoDBContainer3.dependsOn(mongoDBContainer2).start();
    mongodb address in the application using the @DynamicPropertySource annotation:

    ```

    @DynamicPropertySource

    static void setMongoDbProperties(DynamicPropertyRegistry registry) {

    registry.add("spring.data.mongodb.uri", () -> {

    String mongoUri = "mongodb://" + mongoDBContainer1.getHost() + ":" + mongoDBContainer1.getMappedPort(27017) + "/?directConnect=true";

    return mongoUri;

    });

    UserService 类的 buyCards 方法：

    ```java
    @Test
    void buyCards() {
        User user = new User();
        user.setUsername("Sample user");
        User createdUser = userService.createUser(user);
        Integer buyTokens = 10;
        userService.buyTokens(createdUser.getId(), buyTokens);
        Integer requestedCards = 1;
        Integer cardCount = userService.buyCards(user.getId(), requestedCards);
        assertThat(cardCount, is(requestedCards));
       // do more assert
    }
    ```

    为了清晰起见，一些代码片段已被简化或省略。您可以在本书的 GitHub 仓库中找到更多详细信息：[`github.com/PacktPublishing/Spring-Boot-3.0-Cookbook`](https://github.com/PacktPublishing/Spring-Boot-3.0-Cookbook)。

    ```java

    ```

## 它是如何工作的...

可在 Testcontainers 项目中作为模块使用的`MongoDBContainer`容器仅作为单服务器部署工作。因此，我们不是使用`MongoDBContainer`，而是使用了`GenericContainer`。之后，我们适应了 Testcontainers，以便我们可以设置*连接你的应用程序到 MongoDB*食谱中*准备就绪*部分中解释的脚本。为此，我们做了以下操作：

+   创建了一个 Docker 网络。

+   在容器中部署了至少三个 MongoDB 服务器。

+   初始化了 MongoDB 副本集。副本集是一组 Mongo 进程，它们协同工作以维护相同的数据集。我们可以将其视为一个集群。

如您可能已经注意到的，我们在连接到 MongoDB 集群时使用了`directConnection`设置。此设置意味着我们直接连接到集群中的一个特定节点。当连接到副本集时，通常，连接字符串指定所有集群节点，客户端连接到最合适的节点。我们使用`directConnection`的原因是节点可以使用网络别名相互发现。毕竟，它们在同一个网络中，可以使用 DNS 名称。然而，我们开发的应用程序运行在我们的开发计算机上，该计算机托管容器，但它位于不同的网络中，无法通过名称找到节点。如果我们处于同一个网络中，MongoDB 连接字符串将如下所示：

```java
mongodb://mongo1:27017,mongo2:27017,mongo3:27017/football?replicaSet=rs0
```

在这种情况下，客户端将连接到适当的节点。要执行事务，必须连接到主服务器。我们开发的应用程序在执行这些事务时可能会失败，因为它没有连接到主服务器。

注意

`buildMongoEvalCommand` 方法已从 `Testcontainer` 项目的原始 `MongoDBContainer` 容器中改编而来。您可以在 [`github.com/testcontainers/testcontainers-java/blob/main/modules/mongodb/src/main/java/org/testcontainers/containers/MongoDBContainer.java`](https://github.com/testcontainers/testcontainers-java/blob/main/modules/mongodb/src/main/java/org/testcontainers/containers/MongoDBContainer.java) 找到原始代码。

# 使用 MongoDB 管理并发

在这个菜谱中，我们将实现一个在用户之间交换玩家卡片的功能。有些卡片更难获得，这导致它们有更高的需求。因此，虽然许多用户试图找到它们，但只有一个人可能得到它。这是一个高并发的场景。

用户可以使用一定数量的代币交换或购买另一个用户的卡片。我们将实施的过程包括以下步骤：

1.  首先，我们需要检查买家是否有他们承诺的代币。

1.  然后，我们将从买家那里减去代币数量，并添加给卖家。

1.  最后，我们将更改卡片的所有者。

MongoDB 通过文档的版本控制系统支持乐观并发控制。每个文档都有一个版本号（通常称为 *修订* 或 *版本* 字段），每当文档被修改时，该版本号都会递增。当多个客户端同时尝试更新同一文档时，使用版本号来检测冲突，如果存在冲突，则拒绝更改。

随着我们需要控制用户没有在其他事物上花费代币以及卡片没有被与其他用户交换，我们将为 `cards` 和 `users` 添加版本支持。

## 准备就绪

我们将使用在 *将您的应用程序连接到 MongoDB* 菜谱中使用的相同工具 - 那就是 Docker 和 MongoDB。

我们将重用 *在 Testcontainers 中部署 MongoDB 集群* 菜谱中的代码。如果您还没有完成它，不要担心 - 我已经在这个书的 GitHub 仓库中准备了一个工作版本，在 [`github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/`](https://github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/)。您可以在 `chapter6/recipe6-4/start` 文件夹中找到它。

## 如何做到这一点...

让我们为我们的 `Card` 和 `User` 文档添加版本控制支持，并实现一个具有乐观并发控制的卡片交换事务：

1.  首先，我们将修改涉及我们功能的类，以便它们支持乐观并发。我们将通过添加一个带有 `@Version` 注解的新字段来实现这一点：

    1.  通过添加一个名为 `version` 的新 `Long` 字段来修改 `User` 类：

    ```java
    @Version
    private Long version;
    ```

    1.  并且将相同的 `version` 字段添加到 `Card` 类中。

1.  接下来，我们将创建一个名为 `TradingService` 的新服务：

    ```java
    @Service
    public class TradingService {
        private CardRepository cardRepository;
        private UserRepository userRepository;
        public TradingService(CardRepository cardRepository,
                              UserRepository userRepository) {
            this.cardRepository = cardRepository;
            this.userRepository = userRepository;
        }
    }
    ```

    在这里，`CardRepository` 和 `UserRepository` 被添加到构造函数中，因为我们将在实现卡片交换业务逻辑时需要它们。

1.  现在，我们将创建两个方法来实现业务逻辑。一个将使用 `@Transactional` 注解来控制所有更改的原子性，另一个用于控制并发异常：

    +   业务逻辑方法应该看起来如下：

        ```java
        @Transactional
        private Card exchangeCardInternal(String cardId, String newOwnerId, Integer price) {
            Card card = cardRepository.findById(cardId).orElseThrow();
            User newOwner =
        userRepository.findById(newOwnerId).orElseThrow();
            if (newOwner.getTokens() < price) {
                throw new RuntimeException("Not enough tokens");
            }
            newOwner.setTokens(newOwner.getTokens() - price);
            User oldOwner = card.getOwner();
            oldOwner.setTokens(oldOwner.getTokens() + price);
            card.setOwner(newOwner);
            card = cardRepository.save(card);
            userRepository.saveAll(List.of(newOwner, oldOwner));
            return card;
        }
        ```

    +   控制并发的函数应该看起来像这样：

        ```java
        public boolean exchangeCard(String cardId, String newOwnerId,
                                          Integer price) {
            try{
                exchangeCardInternal(cardId, newOwnerId, price);
                return true;
            } catch (OptimisticLockingFailureException e) {
                return false;
            }
        }
        ```

    通过这个机制，我们可以控制对我们的文档执行的并发操作。现在，您可以实现一个 RESTful API，该 API 将使用这个业务逻辑。我在本书的 GitHub 仓库中准备了一个工作示例，网址为 [`github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/`](https://github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/)。它可以在 `chapter6/recipe6-6/end` 中找到。

## 作用原理...

通过添加 `@Version` 注解，保存操作不仅检查 `id` 值是否相同，还检查注解了 `version` 的字段。生成的查询看起来像这样：

```java
Query query = new
Query(Criteria.where("id").is(id).and("version").is(version));
Update update = new Update().set("tokens", value).inc("version", 1);
mongoTemplate.updateFirst(query, update, User.class);
```

如果这个操作失败，它将抛出 `OptimisticLockingFailureException` 异常。

根据业务需求，我们可能需要重试操作或直接放弃它们，就像我们在场景中做的那样。如果用户已经卖出了您想要的卡片，您应该寻找另一张。

由于我们需要修改三个不同的文档，我们使用了事务。我们使用 `@Transactional` 注解进行声明式事务管理。如果我们想回滚该事务中已执行的改变，我们需要抛出异常。这就是为什么我们在 `exchangeCardInternal` 方法中让 Spring Data MongoDB 抛出 `OptimisticLockingFailureException` 并在 `exchangeCard` 中捕获它的原因。

# 将您的应用程序连接到 Apache Cassandra

在这个菜谱中，我们希望创建一个系统，允许用户发布与比赛、球员或比赛事件相关的评论。我们决定使用 Apache Cassandra，因为它具有高可扩展性和低延迟能力。

在这个菜谱中，我们将学习如何使用 Spring Data for Apache Cassandra 存储库将我们的 Spring Boot 应用程序连接到 Apache Cassandra 服务器。

## 准备工作

对于这个菜谱，我们将使用 Apache Cassandra 数据库。在您的电脑上部署 Apache Cassandra 最简单的方法是使用 Docker 容器。您可以通过执行以下 `docker` 命令来完成此任务：

```java
docker run -p 9042:9042 --name cassandra -d cassandra:latest
```

此命令将下载最新的 Apache Cassandra Docker 镜像，如果您电脑上还没有，并且将启动一个监听端口 `9042` 的 Cassandra 服务器。

在启动服务器后，您需要在容器内创建一个 `cqlsh` 脚本：

```java
docker exec -it cassandra cqlsh -e "CREATE KEYSPACE footballKeyspace WITH replication = {'class': 'SimpleStrategy'};"
```

在创建 Keyspace 之前，您可能需要等待几秒钟，以便 Cassandra 服务器完成初始化。

## 如何操作...

让我们创建一个支持 Apache Cassandra 的项目。我们将使用已经熟悉的 Spring Data 的 `Repository` 概念来连接到 Apache Cassandra：

1.  首先，我们将使用*Spring Initializr*工具创建一个新的 Spring Boot 项目。像往常一样，打开[`start.spring.io`](https://start.spring.io)。我们将使用与*第一章*中*创建 RESTful API*食谱中相同的参数，除了我们将使用以下参数：

    +   对于`footballcdb`

    +   对于**依赖项**，选择**Spring Web**和**Spring Data for Apache Cassandra**

1.  接下来，我们将创建一个名为`Comment`的类。这代表了我们新功能的数据。

    如果字段是主键的一部分，我们需要用`@Table`注解类，并用`@PrimaryKeyColumn`注解字段。如果我们想将字段映射到 Cassandra 的不同列名，可以使用`@Column`：

    ```java
    @Table
    public class Comment {
        @PrimaryKeyColumn(name = "comment_id", ordinal = 0, type = PrimaryKeyType.PARTITIONED, ordering = Ordering.DESCENDING)
        private String commentId;
        private String userId;
        private String targetType;
        private String targetId;
        private String content;
        private LocalDateTime date;
        public Set<String> labels = new HashSet<>();
    Comment table will include the comment content, the date, and the user posting the comment. It will also include information about the target of the comment – that is, a player, a match, or any other component we may have in our football application.
    ```

1.  我们需要为`Comment`创建一个新的`Repository`，它继承自`CassandraRepository`：

    ```java
    public interface CommentRepository extends
    CassandraRepository<Comment, String>{
    }
    ```

    与 Spring Data 的`Repository`一样，它提供了一些方法来操作`Comment`实体，例如`findById`、`findAll`、`save`和其他方法。

1.  由于我们将在显示其他实体（如比赛或球员）时检索评论，我们需要在`CommentRepository`中创建一个方法来通过目标类型和目标本身获取评论：

    ```java
    @AllowFiltering
    List<Comment> findByTargetTypeAndTargetId(String targetType, String targetId);
    ```

    注意，与其他 Spring Data 中的仓库一样，它可以通过方法名推断查询来实现接口。

    重要的一点是，我们需要用`@AllowFiltering`注解来注解方法，因为我们不是通过主键检索数据。

1.  我们现在可以使用`CommentRepository`创建一个服务来实现我们的应用程序需求。我们将命名该服务为`CommentService`并确保它包含以下内容：

    ```java
    @Service
    public class CommentService {
        private CommentRepository commentRepository;
        public CommentService(CommentRepository commentRepository){
            this.commentRepository = commentRepository;
        }
    }
    ```

1.  现在，我们必须创建功能。我们将创建一个创建评论的方法和几个检索所有评论的方法：

    +   我们将使用一个记录来接收评论数据：

        ```java
        public record CommentPost(String userId, String targetType, String targetId, String commentContent, Set<String> labels) {
        }
        ```

    +   让我们定义`postComment`方法，以便我们可以创建一个新的评论：

        ```java
        public Comment postComment(CommentPost commentPost) {
            Comment comment = new Comment();
            comment.setCommentId(UUID.randomUUID().toString());
            comment.setUserId(commentPost.userId());
            comment.setTargetType(commentPost.targetType());
            comment.setTargetId(commentPost.targetId());
            comment.setContent(commentPost.commentContent());
            comment.setDate(LocalDateTime.now());
            comment.setLabels(commentPost.labels());
            return commentRepository.save(comment);
        }
        ```

    +   现在，我们可以创建一个方法来检索所有评论：

        ```java
        public List<Comment> getComments() {
            return commentRepository.findAll();
        }
        ```

    +   我们可以检索所有评论，但检索与另一个实体相关的评论更有意义。例如，获取关于球员的评论更为常见：

        ```java
        public List<Comment> getComments(String targetType,
                                         String targetId) {
            return commentRepository.findByTargetTypeAndTargetId(
                                                  targetType, targetId);
        }
        ```

1.  最后，我们需要配置应用程序，使其能够连接到我们的 Cassandra 服务器。在这个食谱的*准备就绪*部分，我提供了使用 Docker 部署它的说明，包括如何创建 Keyspace。要配置应用程序，请在`resources`文件夹中创建一个`application.yml`文件。添加以下内容：

    ```java
    spring:
        cassandra:
            keyspace-name: footballKeyspace
            schema-action: CREATE_IF_NOT_EXISTS
            contact-points: localhost
            local-datacenter: datacenter1
            port: 9042
    ```

1.  现在我们有了提供评论功能所需的组件。我们创建了`CassandraRepository`并连接到了 Cassandra 服务器。我在这本书的 GitHub 仓库中创建了一个 RESTful API 来消费这个服务。您可以在[`github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/`](https://github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/)找到它，位于`chapter6/recipe6-7/end`。

## 它是如何工作的...

正如我们在其他 Spring Data 项目中看到的那样，当您创建一个从 `CassandraRepository` 扩展的接口时，Spring Data for Apache Cassandra 会生成一个实现，并将该实现注册为一个 *bean* 以使其对其他组件可用。

它可以使用命名约定和 `@Query` 注解来生成实现。两种方式都使用 Cassandra 模板生成实现，这将在下一个菜谱中详细介绍。

我们还没有介绍 CQL，这是一种与 SQL 语法相似的编程语言，但与 Cassandra 作为 NoSQL 技术的重要区别。例如，它不支持 **JOIN** 查询。

注意，在 `findByTargetTypeAndTargetId` 方法中，我们使用了 `@AllowFiltering`。Cassandra 是一个为高可用性和可伸缩性而设计的 NoSQL 数据库，但它通过限制它可以高效处理的查询类型来实现这些功能。Cassandra 优化了基于主键或聚类列快速检索数据。当您在 Cassandra 中查询数据时，预期您至少提供主键组件以有效地定位数据。

然而，在某些情况下，您可能需要执行在非主键列上过滤数据的查询。这类查询在 Cassandra 中效率不高，因为它们可能需要全表扫描，并且在大型数据集上可能非常慢。您可以使用 `@AllowFiltering` 注解明确告诉 Spring Data for Apache Cassandra 您了解性能影响，并且尽管其潜在的低效性，您仍想执行此类查询。

## 参考信息

如果您计划与 Cassandra 一起工作，建议您熟悉 CQL。您可以在项目页面上找到更多关于它的信息：[`cassandra.apache.org/doc/stable/cassandra/cql/`](https://cassandra.apache.org/doc/stable/cassandra/cql/)。

# 使用 Testcontainers 与 Cassandra

为了确保我们应用程序的可靠性，我们需要在 Cassandra 项目上运行集成测试。类似于 MongoDB，我们有两种在 Cassandra 上运行测试的选项——要么使用内存中的嵌入式 Cassandra 服务器，要么使用 Testcontainers。然而，我推荐使用带有 Cassandra 服务器的 Testcontainers，因为它使用真实的 Cassandra 实例，从而消除了任何潜在的不兼容性问题。

在这个菜谱中，我们将学习如何使用 Testcontainers Cassandra 模块为我们的 Comments 服务创建集成测试。

## 准备工作

在这个菜谱中，我们将为我们在 *连接您的应用程序到 Apache Cassandra* 菜谱中创建的 Comments 服务创建一个集成测试。如果您还没有完成这个菜谱，您可以使用我准备的项目。您可以在本书的 GitHub 仓库中找到它，位于 [`github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/`](https://github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/) 的 `chapter6/recipe6-8/start` 文件夹中。

## 如何操作...

你准备好将你的应用程序提升到下一个层次了吗？让我们开始准备它，以便它可以运行 Testcontainers 并看看我们如何改进它！

1.  我们将首先将 Testcontainers 依赖项添加到我们的 `pom.xml` 文件中——即通用的 Testcontainers 依赖项和 Cassandra Testcontainers 模块：

    ```java
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>junit-jupiter</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>cassandra</artifactId>
        <scope>test</scope>
    </dependency>
    ```

1.  接下来，在测试的 `resources` 文件夹中创建一个名为 `createKeyspace.cql` 的文件。此文件应包含 Cassandra Keyspace 创建命令：

    ```java
    CREATE KEYSPACE footballKeyspace WITH replication = {'class': 'SimpleStrategy'};
    ```

1.  现在，我们可以为我们的 `CommentService` 创建一个测试类。你可以将测试类命名为 `CommentServiceTest`。在我们开始创建测试之前，我们需要设置 Testcontainer。为此，请执行以下操作：

    1.  使用 `@Testcontainers` 注解测试类：

    ```java
    @Testcontainers
    @SpringBootTest
    class CommentServiceTest
    ```

    1.  声明一个静态的 `CassandraContainer` 字段：

        +   在这里，我们将指定 Cassandra Docker 镜像。我们将使用默认的 `cassandra` 镜像。

        +   我们必须在容器初始化过程中应用要执行的 Cassandra 脚本——即 `createKeyspace.cql`，我们在 *连接你的应用程序到 Apache Cassandra* 菜单的 *准备就绪* 部分中定义了它。

        +   我们还必须公开 Cassandra 监听连接的端口——即端口 `9042`：

    ```java
    static CassandraContainer cassandraContainer = (CassandraContainer) new CassandraContainer("cassandra")
                .withInitScript("createKeyspace.cql")
                .withExposedPorts(@BeforeAll annotation for that purpose:
    ```

```java
@BeforeAll
static void startContainer() throws IOException, InterruptedException {
    cassandraContainer.start();
}
```

1.  最后一个 Testcontainers 配置涉及在应用程序上下文中设置 Cassandra 连接设置。为此，我们将使用 `@DynamicPropertySource` 以及之前声明的 `cassandraContainer` 字段提供的属性：

```java
@DynamicPropertySource
static void setCassandraProperties(DynamicPropertyRegistry registry) {
    registry.add("spring.cassandra.keyspace-name", () -> "footballKeyspace");
    registry.add("spring.cassandra.contact-points", () -> cassandraContainer.getContactPoint().getAddress());
    registry.add("spring.cassandra.port", () -> cassandraContainer.getMappedPort(9042));
    registry.add("spring.cassandra.local-datacenter", () -> cassandraContainer.getLocalDatacenter());
}
```

+   现在，我们可以创建我们的集成测试。让我们将其命名为 `postCommentTest`：

    ```java
    @Autowired
    CommentService commentService;
    @Test
    void postCommentTest() {
        CommentPost comment = new CommentPost("user1", "player", "1", "The best!", Set.of("label1", "label2"));
        Comment result = commentService.postComment(comment);
        assertNotNull(result);
        assertNotNull(result.getCommentId());
    }
    ```

## 它是如何工作的...

`org.testcontainers:cassandra` 依赖项包含 `CassandraContainer` 类，该类提供了设置集成测试 Testcontainer 所需的大部分功能。它允许我们指定我们想要使用的 Docker 镜像。

在这里，`withInitScript` 通过从测试的类路径中获取文件来在 Cassandra 中执行 CQL 脚本。这简化了执行，因为不需要考虑文件复制和客户端工具的可用性。我们使用了这个功能来创建 Keyspace，就像我们在 *连接你的应用程序到 Apache Cassandra* 菜单的 *准备就绪* 部分中所做的那样。

我们不需要手动检查容器服务是否准备好接受连接。Testcontainers 会自动等待服务准备好以启动测试。

最后，我们使用了 `CassandraContainer` 类公开的属性来配置连接。我们使用 `getContactPoint` 方法获取服务器主机地址，使用 `getPort` 方法获取容器公开的端口，以及使用 `getLocalDatacenter` 方法获取模拟的数据中心名称。

# 使用 Apache Cassandra 模板

我们可能希望以比 `CassandraRepository` 提供的更灵活的方式访问 Cassandra 中托管的数据。例如，我们可能希望使用动态或复杂的查询从我们的评论系统中检索数据，批量执行操作，或访问低级功能。在这些情况下，使用 Cassandra 模板更方便，因为它提供了更多对 Cassandra 功能的低级访问。

在这个菜谱中，我们将实现一个功能，该功能将使用不同的参数动态搜索评论，例如日期范围、标签等。为此，我们将使用 **Cassandra 模板**。

## 准备工作

我们将使用与 *将你的应用程序连接到 Apache Cassandra* 菜谱中相同的工具 – 那就是 Docker 和 Apache Cassandra。

要完成这个菜谱，你需要为 *使用 Testcontainers 与 Cassandra* 菜谱创建的项目。如果你还没有完成那个菜谱，不要担心 – 你可以使用我在本书的 GitHub 仓库中准备的全版本项目，该仓库位于 [`github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/`](https://github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/)。它可以在 `chapter6/recipe6-9/start` 文件夹中找到。

## 如何操作...

在这个菜谱中，我们将增强我们在上一个菜谱中创建的评论服务，添加新的搜索功能，以便用户可以使用他们想要的任何参数：

1.  首先，我们需要将 `CassandraTemplate` 注入到我们的 `CommentService` 类中。为此，修改构造函数，使其让 Spring 依赖注入容器注入 `CassandraTemplate`：

    ```java
    @Service
    public class CommentService {
        private CommentRepository commentRepository;
        private CassandraTemplate cassandraTemplate;
        public CommentService(CommentRepository commentRepository,
                              CassandraTemplate cassandraTemplate) {
            this.commentRepository = commentRepository;
            this.cassandraTemplate = cassandraTemplate;
        }
    }
    ```

1.  现在，为 `getComments` 方法添加一个新的重载：

    ```java
    public List<Comment> getComments(String targetType,
                                     String targetId,
                                     Optional<String> userId,
                                     Optional<LocalDateTime> start,
                                     Optional<LocalDateTime> end,
                                     Optional<Set<String>> labels)
    ```

    此方法有两种类型的参数：必填和可选。

    我们假设用户将始终检索与目标实体关联的评论 – 例如，一个玩家或一场比赛。因此，`targetType` 和 `targetId` 参数是必填的。

    其余的参数是可选的；因此，它们被定义为 `Optional<T>`。

1.  在这个新方法中，我们将使用 `QueryBuilder` 组件来创建我们的查询：

    ```java
    Select select = QueryBuilder.selectFrom("comment").all()
                    .whereColumn("targetType")
                    .isEqualTo(QueryBuilder.literal(targetType))
                    .whereColumn("targetId")
                    .isEqualTo(QueryBuilder.literal(targetId));
    ```

    这里，我们使用 `selectFrom` 选择了 `comment` 表，并使用 `whereColumn` 设置了必填列 `targetType` 和 `targetId`。

    其余的可选字段将使用 `whereColumn`，但仅当它们提供时：

    ```java
    if (userId.isPresent()) {
              select = select.whereColumn("userId")
                       .isEqualTo(QueryBuilder.literal(userId.get()));
    }
    if (start.isPresent()) {
        select = select.whereColumn("date")
                       .isGreaterThan(QueryBuilder
                           .literal(start.get().toString()));
    }
    if (end.isPresent()) {
        select = select.whereColumn("date")
                       .isLessThan(QueryBuilder
                           .literal(end.get().toString()));
    }
    if (labels.isPresent()) {
        for (String label : labels.get()) {
            select = select.whereColumn("labels")
                           .contains(QueryBuilder.literal(label));
        }
    }
    ```

1.  最后，我们可以通过使用 `select` 方法来使用 `CassandraTemplate` 的查询。让我们来做吧：

    ```java
    return cassandraTemplate.select(select.allowFiltering().build(),
                                    Comment.class);
    ```

    这里，我们使用了 `allowFiltering`。由于我们不是使用主键，我们需要告诉 Cassandra 我们假设查询可能效率不高。

1.  我们为我们的评论服务实现了新功能，使用 `CassandraTemplate` 执行动态查询。现在，你可以创建一个 RESTful API 接口来与这个新功能交互。我已经创建了一个使用新功能的示例 RESTful API，并为评论服务准备了集成测试。你可以在本书的 GitHub 仓库中找到这些测试，位于 [`github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/`](https://github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/) 的 `chapter6/recipe6-9/end` 文件夹中。

## 它是如何工作的...

Spring Data for Apache Cassandra 在 Spring Boot 依赖注入容器中注册了一个 `CassandraTemplate` bean。它用于内部实现 *将你的应用程序连接到 Apache Cassandra* 菜谱中描述的存储库。通过这样做，它可以被 Spring Boot 注入到我们的组件中。

您可以通过连接谓词来组合 CQL 字符串，但这容易在查询中引入错误。这就是为什么我们使用了`QueryBuilder`。正如我在*将您的应用程序连接到 Apache Cassandra*配方中解释的那样，当我们进行不使用表主键的查询时，我们需要设置`allowFiltering`。

## 还有更多...

我们可以通过构建一个动态 CQL 语句的字符串来执行相同的查询。这看起来会是这样：

```java
public List<Comment> getCommentsString(String targetType,
                                       String targetId,
                                       Optional<String> userId,
                                       Optional<LocalDateTime> start,
                                       Optional<LocalDateTime> end,
                                       Optional<Set<String>> labels) {
    String query = "SELECT * FROM comment WHERE targetType ='"
                   + targetType + "' AND targetId='" + targetId + "'";
    if (userId.isPresent()) {
        query += " AND userId='" + userId.get() + "'";
    }
    if (start.isPresent()) {
        query += " AND date > '" + start.get().toString() + "'";
    }
    if (end.isPresent()) {
        query += " AND date < '" + end.get().toString() + "'";
    }
    if (labels.isPresent()) {
        for (String label : labels.get()) {
           query += " AND labels CONTAINS '" + label + "'";
        }
    }
    query += " ALLOW FILTERING";
    return cassandraTemplate.select(query, Comment.class);
}
```

# 使用 Apache Cassandra 管理并发

我们希望通过添加一个新功能来增强我们的评论系统：对评论进行点赞。我们将在我们的评论中添加一个计数器，以显示收到的正面投票。

这个简单的需求在高并发场景中可能会变得复杂。如果有多个用户正在对一个评论进行点赞，可能会发生我们没有更新评论最新版本的情况。为了应对这种场景，我们将使用 Cassandra 的乐观并发方法。

## 准备工作

我们将使用在*将您的应用程序连接到 Apache Cassandra*配方中使用的相同工具——即 Docker 和 Apache Cassandra。

起始点将是我们在*使用 Apache Cassandra 模板*配方中创建的项目。如果您还没有完成，不要担心——您可以使用我在本书 GitHub 仓库中准备的全版本项目，该仓库位于[`github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/`](https://github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/)。它可以在`chapter6/recipe6-10/start`文件夹中找到。

## 如何实现...

在这个配方中，我们将使用乐观并发实现点赞功能。但在那之前，我们需要准备我们的评论实体。让我们开始吧：

1.  我们首先需要做的是创建一个新字段，该字段将存储评论收到的点赞数。所以，让我们通过添加一个名为`upvotes`的新字段来修改`Comment`类：

    ```java
    private Integer upvotes;
    ```

1.  我们需要修改 Cassandra 服务器中的表模式。为此，我们需要连接到服务器并执行一个`cqlsh`命令。最简单的方法是通过连接到 Docker 容器。以下命令将在`cqlsh`中打开一个交互式会话

    ```java
    docker exec -it cassandra cqlsh
    USE footballKeyspace;
    ALTER TABLE Comment ADD upvotes int;
    ```

    现在，您可以通过执行`quit;`命令退出`cqlsh`。

    在 Cassandra 中无法分配默认值。如果您数据库中已有评论，Cassandra 将为`upvotes`字段返回一个`null`值。因此，我们需要相应地管理这种情况。

1.  现在，是时候在新的操作中使用新字段了。我们将通过创建一个名为`upvoteComment`的新方法在我们的`CommentService`服务中实现这个操作：

    ```java
    public Comment upvoteComment(String commentId) {
    ```

    接下来，我们将检索第一条评论。我们可以使用现有的`CommentRepository`或`CassandraTemplate`。我们将使用`CommentRepository`，因为它更简单：

    ```java
    Comment comment =
    commentRepository.findByCommentId(commentId).get();
    ```

    现在，我们需要更新点赞字段，但我们将保持当前值：

    ```java
    Integer currentVotes = comment.getUpvotes();
    if (currentVotes == null) {
        comment.setUpvotes(1);
    } else {
        comment.setUpvotes(currentVotes + 1);
    }
    ```

    接下来，我们将使用当前值来创建条件。只有当我们更新当前值时，我们才会应用更改：

    ```java
    CriteriaDefinition ifCriteria = Criteria
                                   .where(ColumnName.from("upvotes"))
                                   .is(currentVotes);
    EntityWriteResult<Comment> result = cassandraTemplate
                                   .update(comment,
                                       UpdateOptions.builder()
                                       .ifCondition(ifCriteria)
                                       .build());
    ```

    现在，我们需要检查结果是否是我们所期望的：

    ```java
    if (result.wasApplied()) {
        return result.getEntity();
    }
    ```

    如果结果不是我们所期望的，我们可以在执行之间等待几毫秒的情况下重试操作几次，但这将取决于应用程序的要求。

现在，您可以为此新功能实现一个 RESTful API。我已经在本书的 GitHub 仓库中准备了一个示例 RESTful API 和集成测试，该仓库位于[`github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/`](https://github.com/PacktPublishing/Spring-Boot-3.0-Cookbook/)，在`chapter6/recipe6-10/end`文件夹中。

## 它是如何工作的...

在 Cassandra 中，乐观并发管理的关键是条件更新命令。在 CQL 中，Cassandra 提供了一个`IF`子句，我们可以在`CassandraTemplate`中使用它。使用这个`IF`子句，您可以在满足某些条件的情况下有条件地更新数据，这些条件包括检查数据的当前状态。

我们可以在评论表中创建一个`version`字段来实现一个机制，就像我们在*使用 MongoDB 管理并发*配方中看到的那样。然而，Spring Data for Apache Cassandra 没有提供任何特殊的能力来自动管理这一点，因此我们需要自己实现它。此外，我们预计`comment`实体不会有任何其他变化，因此我们可以使用点赞来控制行是否已被修改。`upvotes`字段是我们的`version`字段。

# 第三部分：应用程序优化

在大规模应用程序中，了解瓶颈在哪里以及如何改进它们是必要的。在本部分中，我们将遵循一种系统性的方法来优化和衡量我们应用的改进。我们还将使用诸如响应式编程和事件驱动设计等高级技术。

本部分包含以下章节：

+   *第七章*，*寻找瓶颈和优化您的应用程序*

+   *第八章*，*Spring Reactive 和 Spring Cloud Stream*
