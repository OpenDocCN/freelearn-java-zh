# 无服务器 Java

近年来，微服务概念已经席卷整个行业，迅速取代了经过实战考验的应用程序服务器，取而代之的是更小、更精简的解决方案。紧随微服务之后的是一个新的概念——函数即服务，更常见的是称为**无服务器**。在本章中，你将了解更多关于这种新的部署模型，并构建一个应用程序来演示如何使用它。

应用程序将是一个简单的通知系统，使用以下技术：

+   亚马逊网络服务

    +   亚马逊 Lambda

    +   亚马逊**身份和访问管理**（**IAM**）

    +   亚马逊**简单通知系统**（**SNS**）

    +   亚马逊**简单电子邮件系统**（**SES**）

    +   亚马逊 DynamoDB

+   JavaFX

+   云提供商提供的选项可能非常广泛，亚马逊网络服务也不例外。在本章中，我们将尝试使用 AWS 提供的一些基本服务来帮助我们构建一个引人注目的应用程序，同时我们涉足云原生应用程序开发。

# 开始使用

在我们进入应用程序之前，我们应该花些时间更好地理解术语**函数即服务**（**FaaS**）。这个术语本身是我们已经看到几年的“即服务”趋势的延续。有一系列这样的术语和产品，但最大的三个是**基础设施即服务**（**IaaS**）、**平台即服务**（**PaaS**）和**软件即服务**（**SaaS**）。通常，这三个是相互依赖的，如下面的图所示：

![图片](img/7dafe91e-0a22-4aba-a7da-1d12003dd8f9.png)

云计算服务最低级别的基础设施即服务提供商，提供**云**中的基础设施相关资产。通常，这可以简单到文件存储，但通常意味着虚拟机。通过使用基础设施即服务提供商，客户无需担心购买、维护或更换硬件，因为这由提供商处理。相反，客户只需根据使用的资源付费。

在堆栈中向上移动，平台即服务提供商提供云托管的应用程序执行环境。这可能包括应用程序服务器、数据库服务器、Web 服务器等。物理环境的细节被抽象化，客户指定存储和 RAM 需求。一些提供商还允许客户选择操作系统，因为这可能会影响应用程序堆栈、支持工具等。

软件即服务是一种更高层次的抽象，它根本不关注硬件，而是提供客户订阅的托管软件，通常是按用户订阅，通常是按月或按年订阅。这在复杂的商业软件中很常见，如财务系统或人力资源应用，但也见于更简单的系统，如博客软件。用户只需订阅并使用软件，将安装和维护（包括升级）留给提供商。虽然这可能会减少用户的灵活性（例如，通常无法自定义软件），但它也将维护成本推给了提供商，并在大多数情况下保证了访问软件的最新版本。

这类服务还有几种其他变体，例如**移动后端即服务**（**MBaas**）和**数据库即服务**（**DBaaS**）。随着市场对云计算的信心不断增强，以及互联网速度加快而价格下降，我们可能会看到越来越多这类系统的开发，这把我们带到了本章的主题。

函数即服务，或称为**无服务器计算**，是将一小段代码（非常直接地说，就是一个函数）部署到其他应用程序中，通常通过某种触发器来调用。用例包括图像转换、日志分析，以及我们将在本章中构建的通知系统。

尽管无服务器这个名字暗示了没有服务器，但实际上确实涉及到了服务器，这是合情合理的；然而，作为应用开发者，你不需要深入思考服务器。实际上，正如我们将在本章中看到的，我们唯一需要担心的是我们的函数需要多少内存。关于服务器的其他一切都将完全由函数即服务提供商处理——操作系统、存储、网络，甚至虚拟机的启动和停止都是由提供商为我们处理的。

对于无服务器的基本理解，我们需要选择一个提供商。正如预期的那样，有众多选择——亚马逊、甲骨文、IBM、红帽等等。不幸的是，目前还没有一种标准化的方法，我们可以用它来编写一个无服务器系统并将其部署到任意提供商，这意味着我们的解决方案将不可避免地与特定的提供商绑定，这个提供商将是**亚马逊网络服务**（**AWS**），云计算服务的主要提供商。正如本章引言中提到的，我们使用了许多 AWS 服务，但核心将是 AWS Lambda，亚马逊的无服务器计算服务。

让我们开始吧。

# 应用程序规划

我们将要构建的应用程序是一个非常简单的 **云通知** 服务。简而言之，我们的函数将 **监听** 消息，然后将这些消息转发到系统中注册的电子邮件地址和电话号码。虽然我们的系统可能有些牵强，而且肯定非常简单，但希望更实际的使用案例是清晰的：

+   我们的系统会提醒学生和/或家长有关即将发生的事件

+   当孩子进入或离开某些地理边界时，家长会收到通知

+   当某些事件发生时，系统管理员会收到通知

可能性相当广泛。就我们的目的而言，我们将开发不仅基于云的系统，还包括一个简单的桌面应用程序来模拟这些类型的场景。我们将从有趣的地方开始：在云中。

# 构建你的第一个函数

作为一项服务（Functions as a Service）的核心当然是函数。在亚马逊网络服务中，这些函数是通过 AWS Lambda 服务部署的。这并不是我们唯一会使用的 AWS 功能，因为我们已经提到过。一旦我们有了函数，我们需要一种执行它的方式。这是通过一个或多个触发器完成的，函数本身也有需要执行的任务，因此当我们最终编写函数时，我们将通过 API 调用来演示更多服务使用。

由于我们的应用程序的结构与我们之前看到的任何其他应用程序都显著不同，此时查看系统图可能会有所帮助：

![图片](img/fabef02c-4c32-4279-88bb-dc6065892082.png)

下面是大致流程：

+   一条消息被发布到简单通知系统中的一个主题

+   一旦验证了调用者的权限，消息就会被投递

+   在消息投递后，会触发一个事件，将消息从主题传递到我们的函数

+   在函数内部，我们将查询亚马逊的 **DynamoDB** 来获取已注册的接收者列表，这些接收者提供了电子邮件地址、手机号码或两者都有

+   所有手机号码将通过 **简单通知系统** 发送短信

+   所有电子邮件地址将通过 **简单电子邮件服务** 发送电子邮件

要开始构建函数，我们需要创建一个 Java 项目。像我们的大多数其他项目一样，这将是一个多模块 Maven 项目。在 NetBeans 中，点击文件 | 新建项目 | Maven | POM 项目。我们将命名 `CloudNotice` 项目。

该项目将包含三个模块——一个用于函数，一个用于测试/演示客户端，还有一个用于共享 API。要创建函数模块，在项目资源管理器中的 `Modules` 节点处右键单击，然后选择创建新模块。在窗口中，选择 Maven | Java 应用程序，点击下一步，并将项目名称设置为 `function`。重复这些步骤，创建一个名为 `api` 的模块。

在我们继续之前，我们必须指出，在撰写本文时，AWS 不支持 Java 9。因此，我们必须针对 Java 8（或更早版本）进行任何将发送到 Lambda 的内容。为此，我们需要像这样修改我们的 `pom.xml` 文件：

```java
    <properties> 
      <maven.compiler.source>1.8</maven.compiler.source> 
      <maven.compiler.target>1.8</maven.compiler.target> 
    </properties> 
```

修改 `api` 和 `function` 的 POM 文件。希望 AWS 能够在 Java 9 发布后尽快支持它。在此之前，我们只能以 JDK 8 为目标。

在我们的项目配置完成后，我们就可以编写我们的函数了。AWS Lambdas 是作为 `RequestHandler` 实例实现的：

```java
    public class SnsEventHandler  
      implements RequestHandler<SNSEvent, Object> { 
        @Override 
        public Object handleRequest 
         (SNSEvent request, Context context) { 
           LambdaLogger logger = context.getLogger(); 
           final String message = request.getRecords().get(0) 
            .getSNS().getMessage(); 
           logger.log("Handle message '" + message + "'"); 
           return null; 
    } 
```

最终，我们希望我们的函数在消息被发送到 SNS 主题时被触发，因此我们将 `SNSEvent` 指定为输入类型。我们还指定了 `Context`。我们可以从 `Context` 中获取几件事情，例如请求 ID、内存限制等，但我们感兴趣的是获取一个 `LambdaLogger` 实例。我们只需将内容写入标准输出和标准错误，这些消息就会被保存在 Amazon CloudWatch 中，但 `LambdaLogger` 允许我们尊重系统权限和容器配置。

为了使这个编译通过，我们需要向我们的应用程序添加一些依赖项，因此我们在 `pom.xml` 中添加以下行：

```java
    <properties> 
      <aws.java.sdk.version>1.11, 2.0.0)</aws.java.sdk.version> 
    </properties> 
    <dependencies> 
      <dependency> 
        <groupId>com.amazonaws</groupId> 
        <artifactId>aws-java-sdk-sns</artifactId> 
        <version>${aws.java.sdk.version}</version> 
      </dependency> 
      <dependency> 
        <groupId>com.amazonaws</groupId> 
        <artifactId>aws-lambda-java-core</artifactId> 
        <version>1.1.0</version> 
      </dependency> 
      <dependency> 
        <groupId>com.amazonaws</groupId> 
        <artifactId>aws-lambda-java-events</artifactId> 
        <version>1.3.0</version> 
      </dependency> 
    </dependencies> 
```

我们现在可以开始实现这个方法，如下所示：

```java
    final List<Recipient> recipients =  new CloudNoticeDAO(false) 
      .getRecipients(); 
    final List<String> emailAddresses = recipients.stream() 
      .filter(r -> "email".equalsIgnoreCase(r.getType())) 
      .map(r -> r.getAddress()) 
      .collect(Collectors.toList()); 
    final List<String> phoneNumbers = recipients.stream() 
      .filter(r -> "sms".equalsIgnoreCase(r.getType())) 
      .map(r -> r.getAddress()) 
      .collect(Collectors.toList()); 
```

我们有一些新的类需要查看，但首先让我们回顾一下这段代码，我们将获取一个 `Recipient` 实例的列表，它代表了已订阅我们服务的号码和电子邮件地址。然后我们从这个列表中创建一个流，过滤每个接收者类型，`SMS` 或 `Email`，通过 `map()` 提取值，然后收集到一个 `List` 中。

我们将在稍后讨论 `CloudNoticeDAO` 和 `Recipient`，但让我们先完成我们的函数。一旦我们有了列表，我们就可以按照以下方式发送消息：

```java
    final SesClient sesClient = new SesClient(); 
    final SnsClient snsClient = new SnsClient(); 

    sesClient.sendEmails(emailAddresses, "j9bp@steeplesoft.com", 
     "Cloud Notification", message); 
    snsClient.sendTextMessages(phoneNumbers, message); 
    sesClient.shutdown(); 
    snsClient.shutdown(); 
```

我们在我们的客户端类 `SesClient` 和 `SnsClient` 后面封装了另外两个 AWS API。这看起来可能有点过度，但这些类型的东西往往会增长，而且这种做法使我们能够很好地管理这一点。

这样我们就有了三个 API 需要查看：DynamoDB、Simple Email Service 和 Simple Notification Service。我们将按顺序进行。

# DynamoDB

Amazon DynamoDB 是一个 NoSQL 数据库，非常类似于我们在第二十五章 [Taking Notes with Monumentum 中查看的 MongoDB，尽管 DynamDB 支持文档和键值存储模型。对这两种模型的详细比较以及关于选择哪一种的建议超出了我们工作的范围。我们在这里选择了 DynamoDB，因为它已经在 Amazon Web Service 中配置好了，因此很容易为我们应用程序进行配置。

要开始使用 DynamoDB API，我们需要向我们的应用程序添加一些依赖项。在 `api` 模块中，将以下内容添加到 `pom.xml` 文件中：

```java
    <properties> 
      <sqlite4java.version>1.0.392</sqlite4java.version> 
    </properties> 
    <dependency> 
      <groupId>com.amazonaws</groupId> 
      <artifactId>aws-java-sdk-dynamodb</artifactId> 
      <version>${aws.java.sdk.version}</version> 
    </dependency> 
    <dependency> 
      <groupId>com.amazonaws</groupId> 
      <artifactId>DynamoDBLocal</artifactId> 
      <version>${aws.java.sdk.version}</version> 
      <optional>true</optional> 
    </dependency> 
    <dependency> 
      <groupId>com.almworks.sqlite4java</groupId> 
      <artifactId>sqlite4java</artifactId> 
      <version>${sqlite4java.version}</version> 
      <optional>true</optional> 
    </dependency> 
```

在我们开始编写 DAO 类之前，让我们定义我们的简单模型。DynamoDB API 提供了一个对象关系映射功能，类似于 Java Persistence API 或 Hibernate，正如我们在这里看到的，它需要一个 POJO 和一些注解：

```java
    public class Recipient { 
      private String id; 
      private String type = "SMS"; 
      private String address = ""; 

      // Constructors... 

      @DynamoDBHashKey(attributeName = "_id") 
      public String getId() { 
        return id; 
      } 

      @DynamoDBAttribute(attributeName = "type") 
      public String getType() { 
        return type; 
      } 

      @DynamoDBAttribute(attributeName="address") 
      public String getAddress() { 
        return address; 
      } 
      // Setters omitted to save space 
    } 
```

在我们的 POJO 中，我们声明了三个属性，`id`、`type` 和 `address`，然后使用 `@DyanoDBAttribute` 注解了 getters，以帮助库理解如何映射对象。

注意，尽管大多数属性名与表中的字段名匹配，但你也可以像我们对 `id` 所做的那样覆盖属性到字段名的映射。

在我们能够对我们的数据进行任何操作之前，我们需要声明我们的表。记住，DynamoDB 是一个 NoSQL 数据库，我们将像使用 MongoDB 一样将其用作文档存储。然而，在我们能够存储任何数据之前，我们必须定义**在哪里**放置它。在 MongoDB 中，我们会创建一个集合。但是，DynamoDB 仍然将其称为表，尽管它在技术上是无模式的，我们仍然需要定义一个主键，它由分区键和一个可选的排序键组成。

我们通过控制台创建表。一旦你登录到 AWS DynamoDB 控制台，你会点击创建表按钮，这将带你到一个类似这样的屏幕：

![图片](img/125e9135-0d20-42e4-8941-6c0a0e13ef46.png)

我们将命名我们的表为 `recipients`，并指定 `_id` 作为分区键。点击创建表按钮，并给 AWS 时间来创建表。

我们现在可以开始编写我们的 DAO 了。在 API 模块中，创建一个名为 `CloudNoticeDAO` 的类，我们将向其中添加这个构造函数：

```java
    protected final AmazonDynamoDB ddb; 
    protected final DynamoDBMapper mapper; 
    public CloudNoticeDAO(boolean local) { 
      ddb = local ? DynamoDBEmbedded.create().amazonDynamoDB() 
       : AmazonDynamoDBClientBuilder.defaultClient(); 
      verifyTables(); 
      mapper = new DynamoDBMapper(ddb); 
    } 
```

本地属性用于确定是否使用本地 DynamoDB 实例。这是为了支持测试（就像对 `verifyTables()` 的调用一样），我们将在稍后探讨。在生产中，我们的代码将调用 `AmazonDynamoDBClientBuilder.defaultClient()` 来获取 `AmazonDynamoDB` 的一个实例，它与 Amazon 托管的实例进行通信。最后，我们创建一个 `DynamoDBMapper` 的实例，我们将使用它进行对象映射。

为了便于创建一个新的 `Recipient`，我们将添加这个方法：

```java
    public void saveRecipient(Recipient recip) { 
      if (recip.getId() == null) { 
        recip.setId(UUID.randomUUID().toString()); 
      } 
      mapper.save(recip); 
    } 
```

此方法将创建数据库中的新条目，或者如果主键已经存在，则更新现有条目。在某些场景中，可能有必要有单独的保存和更新方法，但我们的用例非常简单，所以我们不需要担心这一点。我们所需做的只是如果缺失，则创建键值。我们通过创建一个随机 UUID 来做到这一点，这有助于我们避免在存在多个进程或应用程序写入数据库时发生键冲突。

删除一个 `Recipient` 实例或获取数据库中所有 `Recipient` 实例的列表同样简单：

```java
    public List<Recipient> getRecipients() { 
      return mapper.scan(Recipient.class,  
       new DynamoDBScanExpression()); 
    } 

    public void deleteRecipient(Recipient recip) { 
      mapper.delete(recip); 
    } 
```

在我们离开我们的 DAO 之前，让我们快速看一下我们如何测试它。早些时候，我们提到了 `local` 参数和 `verifyTables()` 方法，它们都存在于测试中。

一般而言，大多数人会对在生产类中添加仅用于测试的方法皱眉，这是有道理的。编写可测试的类和向类中添加测试方法之间是有区别的。我同意仅为了测试而向类中添加方法是一种应该避免的做法，但在这里为了简单和简洁，我稍微违反了这个原则。

`verifyTables()`方法检查表是否存在；如果表不存在，我们将调用另一个方法来为我们创建它。虽然我们手动使用前面的控制台创建了生产表，但我们可以让此方法为我们创建该表。您使用哪种方法完全取决于您。请注意，将会有性能和权限问题需要解决。话虽如此，该方法看起来像这样：

```java
    private void verifyTables() { 
      try { 
        ddb.describeTable(TABLE_NAME); 
      } catch (ResourceNotFoundException rnfe) { 
          createRecipientTable(); 
      } 
    } 

    private void createRecipientTable() { 
      CreateTableRequest request = new CreateTableRequest() 
       .withTableName(TABLE_NAME) 
       .withAttributeDefinitions( 
         new AttributeDefinition("_id", ScalarAttributeType.S)) 
       .withKeySchema( 
         new KeySchemaElement("_id", KeyType.HASH)) 
       .withProvisionedThroughput(new  
         ProvisionedThroughput(10L, 10L)); 

      ddb.createTable(request); 
      try { 
        TableUtils.waitUntilActive(ddb, TABLE_NAME); 
      } catch (InterruptedException  e) { 
        throw new RuntimeException(e); 
      } 
    } 
```

通过调用`describeTable()`方法，我们可以检查表是否存在。在我们的测试中，这将在每次都失败，这将导致创建表。在生产环境中，如果您使用此方法创建表，则此调用仅在第一次调用时失败。在`createRecipientTable()`中，我们可以看到如何以编程方式创建表。我们还等待表处于活动状态，以确保在表创建过程中我们的读写操作不会失败。

我们进行的测试非常简单。例如，考虑以下代码片段：

```java
    private final CloudNoticeDAO dao = new CloudNoticeDAO(true); 
    @Test 
    public void addRecipient() { 
      Recipient recip = new Recipient("SMS", "test@example.com"); 
      dao.saveRecipient(recip); 
      List<Recipient> recipients = dao.getRecipients(); 
      Assert.assertEquals(1, recipients.size()); 
    } 
```

这个测试帮助我们验证我们的模型映射是否正确，以及我们的 DAO 方法是否按预期工作。您可以在源包中的`CloudNoticeDaoTest`类中看到额外的测试。

# 简单电子邮件服务

要发送电子邮件，我们将使用亚马逊简单电子邮件服务（SES），我们将将其包装在`api`模块中的`SesClient`类中。

**重要提示**：在您发送电子邮件之前，您必须验证您的发送/接收地址或域名。验证过程相当简单，但如何操作最好留给亚马逊的文档，您可以通过以下链接阅读：[`docs.aws.amazon.com/ses/latest/DeveloperGuide/verify-email-addresses.html`](http://docs.aws.amazon.com/ses/latest/DeveloperGuide/verify-email-addresses.html)。

简单电子邮件服务 API 相当简单。我们需要创建一个`Destination`，告诉系统将电子邮件发送给谁；一个`Message`，描述消息本身，包括主题、正文和收件人；以及一个`SendEmailRequest`，将所有这些内容结合起来：

```java
    private final AmazonSimpleEmailService client =  
      AmazonSimpleEmailServiceClientBuilder.defaultClient(); 
    public void sendEmails(List<String> emailAddresses, 
      String from, 
      String subject, 
      String emailBody) { 
        Message message = new Message() 
         .withSubject(new Content().withData(subject)) 
         .withBody(new Body().withText( 
           new Content().withData(emailBody))); 
        getChunkedEmailList(emailAddresses) 
         .forEach(group -> 
           client.sendEmail(new SendEmailRequest() 
            .withSource(from) 
            .withDestination( 
              new Destination().withBccAddresses(group)) 
               .withMessage(message))); 
        shutdown(); 
    } 

    public void shutdown() { 
      client.shutdown(); 
    } 
```

然而，有一个重要的注意事项，那就是前面加粗的代码。SES 将每条消息的收件人数量限制为 50，因此我们需要将我们的电子邮件地址列表分成 50 个一组进行处理。我们将使用`getChunkedEmailList()`方法来完成这项工作：

```java
    private List<List<String>> getChunkedEmailList( 
      List<String> emailAddresses) { 
        final int numGroups = (int) Math.round(emailAddresses.size() / 
         (MAX_GROUP_SIZE * 1.0) + 0.5); 
        return IntStream.range(0, numGroups) 
          .mapToObj(group ->  
            emailAddresses.subList(MAX_GROUP_SIZE * group, 
            Math.min(MAX_GROUP_SIZE * group + MAX_GROUP_SIZE, 
            emailAddresses.size()))) 
             .collect(Collectors.toList()); 
    } 
```

要找到组数，我们将地址数量除以 50 并向上取整（例如，254 个地址将给我们 6 组--5 组各 50 个，1 组 4 个）。然后，使用`IntStream`从 0 计数到组数（不包括），我们从原始列表中提取子列表。然后，将这些列表收集到另一个`List`中，给我们看到的方法签名中的嵌套`Collection`实例。

**设计说明**：许多开发者会避免使用这种嵌套`Collection`实例，因为它很快就会变得难以理解变量究竟代表什么。许多人认为在这种情况下创建一个新的类型来保存嵌套数据是一种最佳实践。例如，如果我们遵循这个建议，我们可以创建一个新的`Group`类，它有一个`List<String>`属性来保存组的电子邮件地址。我们没有这样做是为了简洁，但那绝对是对此代码的一个很好的改进。

一旦我们将列表**分块**，我们就可以向每个组发送相同的`消息`，从而满足 API 合约。

# 简单通知服务

我们已经看到了简单通知系统在工作，至少在理论上是这样，因为它是将出站消息传递给我们的函数的：某种客户端在特定的 SNS 主题中发布了一条消息。我们对该主题有一个订阅（我稍后会向你展示如何创建它），它会调用我们的方法，为我们传递消息。现在我们将使用 SNS API 向已将电话号码订阅到系统的用户发送文本（或短信）消息。

使用 SNS，要向多个电话号码发送消息，你必须通过每个号码已订阅的主题来发送。我们接下来要做的就是遵循以下步骤：

1.  创建主题。

1.  订阅所有电话号码。

1.  将消息发布到主题。

1.  删除主题。

如果我们使用持久性主题，当有多个该函数实例同时运行时，我们可能会得到不可预测的结果。负责所有这些工作的方法看起来是这样的：

```java
    public void sendTextMessages(List<String> phoneNumbers,  
      String message) { 
        String arn = createTopic(UUID.randomUUID().toString()); 
        phoneNumbers.forEach(phoneNumber ->  
          subscribeToTopic(arn, "sms", phoneNumber)); 
        sendMessage(arn, message); 
        deleteTopic(arn); 
    } 
```

要创建主题，我们有以下方法：

```java
    private String createTopic(String arn) { 
      return snsClient.createTopic( 
        new CreateTopicRequest(arn)).getTopicArn(); 
    } 
```

要将号码订阅到主题，我们有这个方法：

```java
    private SubscribeResult subscribeToTopic(String arn, 
      String protocol, String endpoint) { 
        return snsClient.subscribe( 
          new SubscribeRequest(arn, protocol, endpoint)); 
    } 
```

发布消息同样简单，正如我们所看到的：

```java
    public void sendMessage(String topic, String message) { 
      snsClient.publish(topic, message); 
    } 
```

最后，你可以使用这个简单的方法删除主题：

```java
    private DeleteTopicResult deleteTopic(String arn) { 
      return snsClient.deleteTopic(arn); 
    } 
```

所有这些方法都非常简单，所以可以直接在调用代码中调用 SNS API，但这个包装器确实为我们提供了一种隐藏 API 细节的方法。这在例如`createTopic()`这样的情况下尤为重要，那里需要额外的类，但为了保持一致性，我们将一切封装在我们的自己的外观之后。

# 部署函数

我们现在已经完成了我们的函数，我们几乎准备好部署它了。为此，我们需要打包它。AWS 允许我们上传 ZIP 或 JAR 文件。我们将使用后者。然而，我们有一些外部依赖项，所以我们将使用**Maven Shade**插件来构建一个包含我们的函数及其所有依赖项的胖 JAR。在`function`模块中，将以下代码片段添加到`pom.xml`文件中：

```java
    <plugin> 
      <groupId>org.apache.maven.plugins</groupId> 
      <artifactId>maven-shade-plugin</artifactId> 
      <version>3.0.0</version> 
      <executions> 
        <execution> 
            <phase>package</phase> 
            <goals> 
                <goal>shade</goal> 
            </goals> 
            <configuration> 
                <finalName> 
                    cloudnotice-function-${project.version} 
                </finalName> 
            </configuration> 
        </execution> 
      </executions> 
    </plugin> 
```

现在，当我们构建项目时，我们将在目标目录中获得一个大型文件（大约 9MB）。这个文件就是我们将要上传的文件。

# 创建角色

在我们可以上传函数之前，我们需要通过创建适当的角色来准备我们的 AWS 环境。登录 AWS 并导航到身份和访问管理控制台（[`console.aws.amazon.com/iam`](https://console.aws.amazon.com/iam)）。在左侧的导航面板中，点击角色，然后点击创建新角色：

![图片](img/e5630954-edea-4a15-a810-c91f79bb900e.png)

当提示选择角色时，我们想要选择 AWS Lambda。在下一页，我们将附加策略：

![图片](img/9f50368b-364c-473c-8427-beddb0ba370e.png)

点击下一步，将名称设置为 `j9bp`，然后点击创建角色。

# 创建主题

为了使创建函数和相关触发器更加简单，我们首先创建我们的主题。导航到 SNS 控制台。鉴于并非所有 AWS 功能在所有区域都始终可用，我们需要选择一个特定区域。我们可以在网页的左上角做到这一点。如果区域不显示为 N. Virginia，请选择它--US East (N. Virginia)--从下拉菜单中选择，然后再继续。

一旦设置正确的区域，点击左侧导航栏中的主题，然后点击创建新主题并指定名称为 `cloud-notice`：

![图片](img/65ae9024-d65d-4a49-842d-c7bced165265.png)

# 部署函数

我们现在可以导航到 Lambda 控制台并部署我们的函数。我们将首先点击创建 Lambda 函数按钮。我们将被要求选择一个蓝图。适合基于 Java 的函数的唯一选项是空白函数。一旦我们点击该选项，我们就会看到配置触发器屏幕。当您点击空方框时，您将看到一个下拉菜单，如 AWS 控制台的此截图所示：

![图片](img/e85d040d-260a-419d-9981-d51fa4541eba.png)

您可以滚动到找到 SNS，或者在过滤器框中输入 `SNS`，如前面的截图所示。无论哪种方式，当您在列表中点击 SNS 时，您将被要求选择您想要订阅的主题：

![图片](img/961a6d80-9ecf-49cf-a606-3fff8103ee53.png)

点击下一步。我们现在需要指定我们函数的详细信息：

![图片](img/38ae6c28-8a7b-4fca-8faf-7425e87a128b.png)

滚动到页面底部，我们还需要指定 Lambda 函数处理程序和角色。处理程序是完整的类名，后面跟着两个冒号，然后是方法名：

![图片](img/39f51cb5-e805-421a-a9be-17fed0b46cde.png)

现在我们需要通过点击上传按钮并选择由我们的 Maven 构建创建的 jar 文件来选择函数存档。点击下一步，验证函数的详细信息，然后点击创建函数。

我们现在有一个可用的 AWS Lambda 函数。我们可以使用 Lambda 控制台来测试它，但我们将构建一个小的 JavaFX 应用程序来完成这个任务，这将同时测试所有服务集成，并展示一个生产应用程序如何与该函数交互。

# 测试函数

为了帮助测试和演示系统，我们将在`CloudNotice`项目中创建一个新的模块，名为`manager`。为此，在 NetBeans 项目资源管理器中单击模块节点，然后单击创建新模块... | Maven | JavaFX 应用程序。将项目命名为`Manager`，然后单击完成。

我已将`MainApp`重命名为`CloudNoticeManager`，将`FXMLController`重命名为`CloudNoticeManagerController`，将`Scene.fxml`重命名为`manager.fxml`。

我们的`Application`类将与之前的 JavaFX 应用程序略有不同。一些 AWS 客户端 API 要求在完成使用后显式关闭它们。未能这样做意味着我们的应用程序将无法完全退出，留下必须被杀死的**僵尸**进程。为了确保我们正确关闭 AWS 客户端，我们需要在我们的控制器中添加一个清理方法，然后从应用程序中的`stop()`方法调用它：

```java
    private FXMLLoader fxmlLoader; 
    @Override 
    public void start(final Stage stage) throws Exception { 
      fxmlLoader = new FXMLLoader(getClass() 
       .getResource("/fxml/manager.fxml")); 
      Parent root = fxmlLoader.load(); 
      // ... 
    } 

    @Override 
    public void stop() throws Exception { 
      CloudNoticeManagerController controller =  
        (CloudNoticeManagerController) fxmlLoader.getController(); 
      controller.cleanup(); 
      super.stop();  
    } 
```

现在，无论用户是点击文件 | 退出还是点击窗口上的关闭按钮，我们的 AWS 客户端都可以被正确清理。

在布局方面，没有新的内容可讨论，因此我们不会在此处过多关注这一点。这是我们管理应用程序的外观：

![图片](img/32338e18-d903-4abe-956b-1475793a3c80.png)

我们在左侧有一个订阅的收件人列表，在右上角有一个添加和编辑收件人的区域，在右下角有一个发送测试消息的区域。我们确实有一些有趣的绑定，所以让我们看看这个。

首先，在`CloudNoticeManagerController`中，我们需要声明一些用于我们的数据容器，所以我们声明了多个`ObservableList`实例：

```java
    private final ObservableList<Recipient> recips =  
      FXCollections.observableArrayList(); 
    private final ObservableList<String> types =  
      FXCollections.observableArrayList("SMS", "Email"); 
    private final ObservableList<String> topics =  
      FXCollections.observableArrayList(); 
```

这三个`ObservableList`实例将支持与其名称匹配的 UI 控件。我们将在`initialize()`方法中填充其中两个列表（`type`是硬编码的），如下所示：

```java
    public void initialize(URL url, ResourceBundle rb) { 
      recips.setAll(dao.getRecipients()); 
      topics.setAll(sns.getTopics()); 

      type.setItems(types); 
      recipList.setItems(recips); 
      topicCombo.setItems(topics); 
```

使用我们的 DAO 和 SES 客户端，我们获取任何已订阅的收件人以及账户中配置的任何主题。这将获取**每个**主题，所以如果你有很多，这可能会成为问题，但这是一个演示应用程序，所以这里应该没问题。一旦我们有了这两个列表，我们将它们添加到我们之前创建的`ObservableList`实例中，然后将`List`与适当的 UI 控件关联。

为了确保`Recipient`列表正确显示，我们需要创建一个`CellFactory`，如下所示：

```java
    recipList.setCellFactory(p -> new ListCell<Recipient>() { 
      @Override 
      public void updateItem(Recipient recip, boolean empty) { 
        super.updateItem(recip, empty); 
        if (!empty) { 
          setText(String.format("%s - %s", recip.getType(),  
            recip.getAddress())); 
          } else { 
              setText(null); 
          } 
        } 
    }); 
```

记住，如果单元格为空，我们需要将文本设置为 null 以清除任何之前的值。未能这样做将导致在某个时刻出现带有**幽灵**条目的`ListView`。

接下来，我们需要在用户点击列表中的`Recipient`时更新编辑控件。我们通过向`selectedItemProperty`添加监听器来实现这一点，该监听器在所选项目更改时运行：

```java
    recipList.getSelectionModel().selectedItemProperty() 
            .addListener((obs, oldRecipient, newRecipient) -> { 
        type.valueProperty().setValue(newRecipient != null ?  
            newRecipient.getType() : ""); 
        address.setText(newRecipient != null ?  
            newRecipient.getAddress() : ""); 
    }); 
```

如果`newRecipient`不为 null，我们将控件值设置为适当的值。否则，我们将清除值。

我们现在需要为各种按钮添加处理程序——位于`收件人`列表上方的添加和删除按钮，以及右侧两个**表单**区域中的保存和取消按钮。

UI 控件的`onAction`属性可以通过直接编辑 FXML 绑定到类中的方法，如下所示：

```java
    <Button mnemonicParsing="false"  
      onAction="#addRecipient" text="+" /> 
    <Button mnemonicParsing="false"  
      onAction="#removeRecipient" text="-" /> 
```

它也可以通过在 Scene Builder 中编辑属性来绑定到方法，如下面的截图所示：

![图片](img/2d472bcc-4ee9-44d6-b382-e29fc81abe6d.png)

无论哪种方式，方法看起来都像这样：

```java
    @FXML 
    public void addRecipient(ActionEvent event) { 
      final Recipient recipient = new Recipient(); 
      recips.add(recipient); 
      recipList.getSelectionModel().select(recipient); 
      type.requestFocus(); 
    } 
```

我们添加了一个`Recipient`，因此我们创建了一个新的，将其添加到我们的`ObservableList`中，然后告诉`ListView`选择这个条目。最后，我们要求`type`控件请求焦点，以便用户可以轻松地使用键盘更改值，如果需要的话。新的收件人只有在用户点击保存时才会保存到 DynamoDB，我们稍后会看到这一点。

当我们删除一个`Recipient`时，我们需要从 UI 以及从 DynamoDB 中删除它：

```java
    @FXML 
    public void removeRecipient(ActionEvent event) { 
      final Recipient recipient = recipList.getSelectionModel() 
       .getSelectedItem(); 
      dao.deleteRecipient(recipient); 
      recips.remove(recipient); 
    } 
```

保存稍微复杂一些，但并不复杂：

```java
    @FXML 
    public void saveChanges(ActionEvent event) { 
      final Recipient recipient =  
        recipList.getSelectionModel().getSelectedItem(); 
      recipient.setType(type.getValue()); 
      recipient.setAddress(address.getText()); 
      dao.saveRecipient(recipient); 
      recipList.refresh(); 
    } 
```

由于我们没有将编辑控件的值绑定到列表中选定的项目，我们需要获取项目的引用，然后将控件中的值复制到模型中。完成此操作后，我们通过我们的 DAO 将其保存到数据库，然后要求`ListView`刷新自己，以便任何模型更改都能反映在列表中。

我们没有将控件绑定到列表中的项目，因为这会导致用户体验略感困惑。如果我们绑定了，当用户对模型进行更改时，`ListView`会反映这些更改。可以想象，用户可能会认为这些更改正在被保存到数据库中，而实际上并不是。这种情况只有在用户点击保存时才会发生。为了避免这种困惑和数据丢失，我们没有绑定控件，而是手动管理数据。

要取消更改，我们只需要从`ListView`获取未更改模型的引用，然后将其值复制到编辑控件中：

```java
    @FXML 
    public void cancelChanges(ActionEvent event) { 
      final Recipient recipient = recipList.getSelectionModel() 
        .getSelectedItem(); 
      type.setValue(recipient.getType()); 
      address.setText(recipient.getAddress()); 
    } 
```

这就留下了 UI 的**发送消息**部分。多亏了我们的 SNS 包装器 API，这些方法非常简单：

```java
    @FXML 
    public void sendMessage(ActionEvent event) { 
      sns.sendMessage(topicCombo.getSelectionModel() 
        .getSelectedItem(), messageText.getText()); 
      messageText.clear(); 
    } 

    @FXML 
    public void cancelMessage(ActionEvent event) { 
      messageText.clear(); 
    } 
```

从我们的桌面应用程序中，我们现在可以添加、编辑和删除收件人，以及发送测试消息。

# 配置您的 AWS 凭证

那些非常关注细节的人可能会问一个非常重要的问题——AWS 客户端库是如何知道如何登录到我们的账户的？显然，我们需要告诉它们，我们有一些选择。

当 AWS SDK 在本地运行时，它会在三个地方检查凭证——环境变量（`AWS_ACCESS_KEY_ID`和`AWS_SECRET_ACCESS_KEY`）、系统属性（`aws.accessKeyId`和`aws.secretKey`）以及默认凭证配置文件（`$HOME/.aws/credentials`）。您使用哪种凭证取决于您，但我会在这里向您展示如何配置配置文件。

就像 Unix 或 Windows 系统，您的 AWS 账户有一个具有对系统完全访问权限的 `root` 用户。以这个用户身份运行任何客户端代码都是非常不明智的。为了避免这种情况，我们需要创建一个用户，我们可以在身份和访问管理控制台（[`console.aws.amazon.com/iam`](https://console.aws.amazon.com/iam)）上完成这一操作。

登录后，点击左侧的“用户”，然后在顶部点击“添加用户”，结果如下截图所示：

![](img/c16ba08c-f738-4383-aa4d-76a98f2eb58f.png)

点击“下一步：权限”，然后检查角色列表中我们的角色`j9bp`的条目。点击“下一步：审查”，然后创建用户。这将带您到添加用户屏幕，应该有一个成功消息框。重要的是屏幕底部列出的用户信息。在此表的右侧，你应该看到“访问密钥 ID”和“秘密访问密钥”这两列。点击“显示”访问密钥以显示其值。请记住这两个值，因为一旦离开此页面，就无法检索访问密钥。如果您丢失了它，您将不得不生成一组新的密钥，这将破坏使用旧凭证的任何其他应用程序。

![](img/e2754894-8149-4c86-b987-6e010b3f5909.png)

在文本编辑器中，我们需要创建 `~/.aws/credentials` 文件。在 Unix 系统上，这可能是 `/home/jdlee/.aws`，而在 Windows 机器上可能是 `C:Usersjdleeaws`。凭证文件应该看起来像这样：

```java
    [default] 
    aws_access_key_id = AKIAISQVOILE6KCNQ7EQ 
    aws_secret_access_key = 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 
```

在同一目录下，我们还需要创建另一个名为 `config` 的文件。我们将使用此文件来告诉 SDK 我们想要在哪个区域工作：

```java
    [default] 
    region = us-east-1 
```

现在，当 AWS 客户端启动时，它们将默认连接到 `us-east-1` 区域的 `j9bp` 用户。如果您需要覆盖此设置，您可以编辑此文件或设置在 *配置您的 AWS 凭证* 部分中提到的上述环境变量或系统属性。

# 摘要

我们做到了！我们中的许多人已经创建了我们的第一个 AWS Lambda 函数，这其实并没有那么困难。当然，这是一个简单的应用程序，但我希望你能看到这种类型的应用程序可能非常有用。以这个作为起点，你可以编写系统，借助移动应用程序的帮助，以跟踪你家人的位置。例如，使用嵌入式设备如 Raspberry PI，你可以构建跟踪货物在全国范围内运输的设备，报告位置、速度、环境条件、突然的下降或冲击等。在服务器上运行的软件可以持续报告有关系统的各种指标，例如 CPU 温度、可用磁盘空间、分配的内存、系统负载等。你的选择仅受你的想象力限制。

总结一下，让我们快速回顾一下我们所学的内容。我们了解了一些今天提供的各种**...作为服务**系统，以及**无服务器**真正意味着什么，以及为什么它可能对我们这些应用开发者有吸引力。我们学习了如何配置各种亚马逊云服务产品，包括身份和访问管理、简单通知系统、简单电子邮件服务，当然还有 Lambda，我们还学习了如何在 Java 中编写 AWS Lambda 函数以及如何将其部署到服务中。最后，我们学习了如何配置触发器，将 SNS 发布/订阅主题与我们的 Lambda 函数关联起来。

毫无疑问，我们的应用相对简单，在一个章节的空间内也不可能让你成为亚马逊云服务或任何其他云服务提供商所提供的一切的专家。希望这足以让你开始——并激发你对使用 Java 编写云应用的热情。对于那些想要深入研究的人，有许多优秀的书籍、网页等等可以帮助你更深入地了解这个快速变化和扩展的领域。在我们下一章中，我们将从云服务中返回，并将注意力转向 Java 开发者另一个伟大的空间——你的手机。
