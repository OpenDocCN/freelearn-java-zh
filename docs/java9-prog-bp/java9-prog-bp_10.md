# 无服务器 Java

近年来，我们已经看到了微服务的概念，迅速取代了经过考验的应用服务器，变得更小更精简。紧随微服务之后的是一个新概念--函数即服务，通常称为**无服务器**。在本章中，您将了解更多关于这种新的部署模型，并构建一个应用程序来演示如何使用它。

该应用将是一个简单的通知系统，使用以下技术：

+   亚马逊网络服务

+   亚马逊 Lambda

+   亚马逊**身份和访问管理**（**IAM**）

+   亚马逊**简单通知系统**（**SNS**）

+   亚马逊**简单邮件系统**（**SES**）

+   亚马逊 DynamoDB

+   JavaFX

+   云服务提供商提供的选项可能非常广泛，亚马逊网络服务也不例外。在本章中，我们将尝试充分利用 AWS 所提供的资源，帮助我们构建一个引人注目的应用程序，进入云原生应用程序开发。

# 入门

在我们开始应用之前，我们应该花一些时间更好地理解**函数作为服务**（**FaaS**）这个术语。这个术语本身是我们几年来看到的**作为服务**趋势的延续。有许多这样的术语和服务，但最重要的三个是**基础设施即服务**（**IaaS**）、**平台即服务**（**PaaS**）和**软件即服务**（**SaaS**）。通常情况下，这三者相互依赖，如下图所示：

![](img/d275ceea-af69-4cb5-92c3-ec49407acaca.png)

云计算提供商的最低级别是基础设施即服务提供商，提供**云中的**基础设施相关资产。通常情况下，这可能只是文件存储，但通常意味着虚拟机。通过使用基础设施即服务提供商，客户无需担心购买、维护或更换硬件，因为这些都由提供商处理。客户只需按使用的资源付费。

在平台作为服务提供商中，提供云托管的应用程序执行环境。这可能包括应用服务器、数据库服务器、Web 服务器等。物理环境的细节被抽象化，客户可以指定存储和内存需求。一些提供商还允许客户选择操作系统，因为这可能会对应用程序堆栈、支持工具等产生影响。

软件即服务是一个更高级的抽象，根本不关注硬件，而是提供订阅的托管软件，通常按用户订阅，通常按月或年计费。这通常出现在复杂的商业软件中，如财务系统或人力资源应用程序，但也出现在更简单的系统中，如博客软件。用户只需订阅并使用软件，安装和维护（包括升级）都由提供商处理。虽然这可能会减少用户的灵活性（例如，通常无法定制软件），但它也通过将维护成本推给提供商以及在大多数情况下保证访问最新版本的软件来降低运营成本。

这种类型的服务还有几种其他变体，比如**移动后端作为服务**（**MBaas**）和**数据库作为服务**（**DBaaS**）。随着市场对云计算的信心增强，互联网速度加快，价格下降，我们很可能会看到更多这类系统的开发，这也是本章的主题所在。

函数即服务，或者**无服务器**计算，是部署一个小段代码，非常字面上的一个函数，可以被其他应用程序调用，通常通过某种触发器。使用案例包括图像转换、日志分析，以及我们将在本章中构建的通知系统。

尽管**无服务器**这个名字暗示着没有服务器，实际上确实有一个服务器参与其中，这是理所当然的；然而，作为应用程序开发人员，你不需要深入思考服务器。事实上，正如我们将在本章中看到的，我们唯一需要担心的是我们的函数需要多少内存。关于服务器的其他一切都完全由服务提供商处理--操作系统、存储、网络，甚至虚拟机的启动和停止都由提供商为我们处理。

有了对无服务器的基本理解，我们需要选择一个提供商。可以预料到，有许多选择--亚马逊、甲骨文、IBM、红帽等。不幸的是，目前还没有标准化的方式可以编写一个无服务器系统并将其部署到任意提供商，因此我们的解决方案将必然与特定的提供商绑定，这将是**亚马逊网络服务**（**AWS**），云计算服务的主要提供商。正如在本章的介绍中提到的，我们使用了许多 AWS 的产品，但核心将是 AWS Lambda，亚马逊的无服务器计算产品。

让我们开始吧。

# 规划应用程序

我们将构建的应用程序是一个非常简单的**云通知**服务。简而言之，我们的函数将**监听**消息，然后将这些消息转发到系统中注册的电子邮件地址和电话号码。虽然我们的系统可能有些牵强，当然非常简单，但希望更实际的用例是清楚的：

+   我们的系统提醒学生和/或家长即将到来的事件

+   当孩子进入或离开某些地理边界时，家长会收到通知

+   系统管理员在发生某些事件时会收到通知

可能性非常广泛。对于我们在这里的目的，我们将开发不仅基于云的系统，还将开发一个简单的桌面应用程序来模拟这些类型的场景。我们将从有趣的地方开始：在云中。

# 构建你的第一个函数

作为服务的功能的核心当然是函数。在亚马逊网络服务中，这些函数是使用 AWS Lambda 服务部署的。这并不是我们将使用的唯一的 AWS 功能，正如我们已经提到的。一旦我们有了一个函数，我们需要一种执行它的方式。这是通过一个或多个触发器来完成的，函数本身有它需要执行的任务，所以当我们最终编写函数时，我们将通过 API 调用来演示更多的服务使用。

在这一点上，鉴于我们的应用程序的结构与我们所看到的任何其他东西都有很大不同，看一下系统图可能会有所帮助：

![](img/ef4a0f18-7a3b-4f35-9524-5b140ff96666.png)

以下是大致的流程：

+   一条消息将被发布到简单通知系统的主题中

+   一旦验证了呼叫者的权限，消息就会被传送

+   消息传送后，将触发一个触发器，将主题中的消息传送到我们的函数

+   在函数内部，我们将查询亚马逊的**DynamoDB**，获取已注册的收件人列表，提供电子邮件地址、手机号码，或两者都提供

+   所有的手机号码都将通过**简单通知系统**收到一条短信

+   所有的电子邮件地址都将通过**简单电子邮件服务**发送电子邮件

要开始构建函数，我们需要创建一个 Java 项目。和我们的其他项目一样，这将是一个多模块的 Maven 项目。在 NetBeans 中，点击文件 | 新建项目 | Maven | POM 项目。我们将称之为`CloudNotice`项目。

该项目将有三个模块--一个用于函数，一个用于测试/演示客户端，一个用于共享 API。要创建函数模块，请在项目资源管理器中右键单击`Modules`节点，然后选择创建新模块。在窗口中，选择 Maven | Java Application，然后单击下一步，将项目名称设置为`function`。重复这些步骤，创建一个名为`api`的模块。

在我们继续之前，我们必须解决一个事实，即在撰写本文时，AWS 不支持 Java 9。因此，我们必须将我们将要交付给 Lambda 的任何东西都定位到 Java 8（或更早）。为此，我们需要修改我们的`pom.xml`文件，如下所示：

```java
    <properties> 
      <maven.compiler.source>1.8</maven.compiler.source> 
      <maven.compiler.target>1.8</maven.compiler.target> 
    </properties> 

```

修改`api`和`function`的 POM。希望 AWS 在发布后尽快支持 Java 9。在那之前，我们只能针对 JDK 8。

配置好项目后，我们准备编写我们的函数。AWS Lambdas 被实现为`RequestHandler`实例：

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

最终，我们希望我们的函数在将消息传递到 SNS 主题时被触发，因此我们将`SNSEvent`指定为输入类型。我们还指定`Context`。我们可以从`Context`中获取几件事情，比如请求 ID，内存限制等，但我们只对获取`LambdaLogger`实例感兴趣。我们可以直接写入标准输出和标准错误，这些消息将保存在 Amazon CloudWatch 中，但`LambdaLogger`允许我们尊重系统权限和容器配置。

为了使其编译，我们需要向我们的应用程序添加一些依赖项，因此我们将以下行添加到`pom.xml`中：

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

现在我们可以开始实现该方法了：

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

我们有一些新的类要看，但首先要总结一下这段代码，我们将获得一个`Recipient`实例列表，它代表了已订阅我们服务的号码和电子邮件地址。然后我们从列表中创建一个流，过滤每个接收者类型，`SMS`或`Email`，通过`map()`提取值，然后将它们收集到一个`List`中。

我们马上就会看到`CloudNoticeDAO`和`Recipient`，但首先让我们先完成我们的函数。一旦我们有了我们的列表，我们就可以像下面这样发送消息：

```java
    final SesClient sesClient = new SesClient(); 
    final SnsClient snsClient = new SnsClient(); 

    sesClient.sendEmails(emailAddresses, "j9bp@steeplesoft.com", 
     "Cloud Notification", message); 
    snsClient.sendTextMessages(phoneNumbers, message); 
    sesClient.shutdown(); 
    snsClient.shutdown(); 

```

我们在自己的客户端类`SesClient`和`SnsClient`后面封装了另外两个 AWS API。这可能看起来有点过分，但这些类型的东西往往会增长，这种方法使我们处于一个很好的位置来管理它。

这让我们有三个 API 要看：DynamoDB，简单邮件服务和简单通知服务。我们将按顺序进行。

# DynamoDB

Amazon DynamoDB 是一个 NoSQL 数据库，非常类似于我们在[第九章中看到的 MongoDB，*使用 Monumentum 进行笔记*，尽管 DynamoDB 支持文档和键值存储模型。对这两者进行彻底比较，以及推荐选择哪一个，远远超出了我们在这里的工作范围。我们选择了 DynamoDB，因为它已经在亚马逊网络服务中预配，因此很容易为我们的应用程序进行配置。

要开始使用 DynamoDB API，我们需要向我们的应用程序添加一些依赖项。在`api`模块中，将其添加到`pom.xml`文件中：

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

在我们开始编写 DAO 类之前，让我们先定义我们的简单模型。DynamoDB API 提供了一个对象关系映射工具，非常类似于 Java Persistence API 或 Hibernate，它将需要一个 POJO 和一些注释，就像我们在这里看到的那样：

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

在我们的 POJO 中，我们声明了三个属性，`id`，`type`和`address`，然后用`@DyanoDBAttribute`注释了 getter，以帮助库理解如何映射对象。

请注意，虽然大多数属性名称与表中的字段名称匹配，但您可以覆盖属性到字段名称的映射，就像我们在`id`中所做的那样。

在我们对数据进行任何操作之前，我们需要声明我们的表。请记住，DynamoDB 是一个 NoSQL 数据库，我们将像在 MongoDB 中一样将其用作文档存储。然而，在我们存储任何数据之前，我们必须定义**放置**数据的位置。在 MongoDB 中，我们会创建一个集合。然而，DynamoDB 仍然将其称为表，虽然它在技术上是无模式的，但我们确实需要定义一个主键，由分区键和可选的排序键组成。

我们通过控制台创建表。一旦您登录到 AWS DynamoDB 控制台，您将点击“创建表”按钮，这将带您到一个类似这样的屏幕：

![](img/f7632622-3d2a-490d-b1e1-bfba3ee7c75b.png)

我们将表命名为`recipients`，并指定`_id`为分区键。点击“创建表”按钮，让 AWS 创建表。

我们现在准备开始编写我们的 DAO。在 API 模块中，创建一个名为`CloudNoticeDAO`的类，我们将添加这个构造函数：

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

`local`属性用于确定是否使用本地 DynamoDB 实例。这是为了支持测试（就像调用`verifyTables`一样），我们将在下面探讨。在生产中，我们的代码将调用`AmazonDynamoDBClientBuilder.defaultClient()`来获取`AmazonDynamoDB`的实例，它与托管在亚马逊的实例进行通信。最后，我们创建了一个`DynamoDBMapper`的实例，我们将用它进行对象映射。

为了方便创建一个新的`Recipient`，我们将添加这个方法：

```java
    public void saveRecipient(Recipient recip) { 
      if (recip.getId() == null) { 
        recip.setId(UUID.randomUUID().toString()); 
      } 
      mapper.save(recip); 
    } 

```

这个方法要么在数据库中创建一个新条目，要么在主键已经存在的情况下更新现有条目。在某些情况下，可能有必要有单独的保存和更新方法，但我们的用例非常简单，所以我们不需要担心这个。我们只需要在缺失时创建键值。我们通过创建一个随机 UUID 来实现这一点，这有助于我们避免在有多个进程或应用程序写入数据库时出现键冲突。

删除`Recipient`实例或获取数据库中所有`Recipient`实例的列表同样简单：

```java
    public List<Recipient> getRecipients() { 
      return mapper.scan(Recipient.class,  
       new DynamoDBScanExpression()); 
    } 

    public void deleteRecipient(Recipient recip) { 
      mapper.delete(recip); 
    } 

```

在我们离开 DAO 之前，让我们快速看一下我们如何测试它。之前，我们注意到了`local`参数和`verifyTables()`方法，这些方法存在于测试中。

一般来说，大多数人都会对在生产类中添加方法进行测试感到不满，这是正确的。编写一个可测试的类和向类添加测试方法是有区别的。我同意为了简单和简洁起见，为了测试而向类添加方法是应该避免的，但我在这里违反了这个原则。

`verifyTables()`方法检查表是否存在；如果表不存在，我们调用另一个方法来为我们创建它。虽然我们手动使用前面的控制台创建了生产表，但我们也可以让这个方法为我们创建那个表。您使用哪种方法完全取决于您。请注意，这将涉及需要解决的性能和权限问题。也就是说，该方法看起来像这样：

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

通过调用`describeTable()`方法，我们可以检查表是否存在。在我们的测试中，这将每次失败，这将导致表被创建。在生产中，如果您使用此方法创建表，这个调用只会在第一次调用时失败。在`createRecipientTable()`中，我们可以看到如何通过编程方式创建表。我们还等待表处于活动状态，以确保在创建表时我们的读写不会失败。

然后，我们的测试非常简单。例如，考虑以下代码片段：

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

这个测试帮助我们验证我们的模型映射是否正确，以及我们的 DAO 方法是否按预期工作。您可以在源代码包中的`CloudNoticeDaoTest`类中看到其他测试。

# 简单邮件服务

要发送电子邮件，我们将使用亚马逊简单电子邮件服务（SES），我们将在`api`模块的`SesClient`类中封装它。

**重要**：在发送电子邮件之前，您必须验证发送/来自地址或域。验证过程非常简单，但如何做到这一点可能最好留给亚马逊的文档，您可以在这里阅读：[`docs.aws.amazon.com/ses/latest/DeveloperGuide/verify-email-addresses.html`](http://docs.aws.amazon.com/ses/latest/DeveloperGuide/verify-email-addresses.html)。

简单电子邮件服务 API 非常简单。我们需要创建一个`Destination`，告诉系统要发送电子邮件给谁；一个描述消息本身的`Message`，包括主题、正文和收件人；以及将所有内容联系在一起的`SendEmailRequest`：

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

但是有一个重要的警告，就是在前面加粗的代码中。SES 将每封邮件的收件人数量限制为 50，因此我们需要每次处理 50 个电子邮件地址的列表。我们将使用`getChunkedEmailList()`方法来做到这一点：

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

要找到组的数量，我们将地址的数量除以 50 并四舍五入（例如，254 个地址将得到 6 个组--50 个中的 5 个和 4 个中的 1 个）。然后，使用`IntStream`从 0 到组数（不包括）进行计数，我们从原始列表中提取子列表。然后，将这些列表中的每一个收集到另一个`List`中，从而得到我们在方法签名中看到的嵌套的`Collection`实例。

**设计说明**：许多开发人员会避免像这样使用嵌套的`Collection`实例，因为很快就会变得难以理解变量到底代表什么。在这种情况下，许多人认为最好的做法是创建一个新类型来保存嵌套数据。例如，如果我们在这里遵循这个建议，我们可以创建一个新的`Group`类，它具有一个`List<String>`属性来保存组的电子邮件地址。出于简洁起见，我们没有这样做，但这绝对是对这段代码的一个很好的增强。

一旦我们对列表进行了分块，我们可以将相同的`Message`发送给每个组，从而满足 API 合同。

# 简单通知服务

我们已经在理论上看到了简单通知系统的工作，至少是这样，因为它将出站消息传递给我们的函数：某种客户端在特定的 SNS 主题中发布消息。我们订阅了该主题（我将向您展示如何创建它），并调用我们的方法传递消息以便我们传递。我们现在将使用 SNS API 向已经订阅了电话号码的用户发送文本（或短信）消息。

使用 SNS，要向多个电话号码发送消息，必须通过每个号码都订阅的主题来实现。然后，我们将按照以下步骤进行：

1.  创建主题。

1.  订阅所有电话号码。

1.  将消息发布到主题。

1.  删除主题。

如果我们使用持久主题，如果我们同时运行多个函数实例，可能会得到不可预测的结果。负责所有这些工作的方法看起来像这样：

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

要订阅主题中的数字，我们有这个方法：

```java
    private SubscribeResult subscribeToTopic(String arn, 
      String protocol, String endpoint) { 
        return snsClient.subscribe( 
          new SubscribeRequest(arn, protocol, endpoint)); 
    } 

```

发布消息同样简单，如下所示：

```java
    public void sendMessage(String topic, String message) { 
      snsClient.publish(topic, message); 
    } 

```

最后，您可以使用这个简单的方法删除主题：

```java
    private DeleteTopicResult deleteTopic(String arn) { 
      return snsClient.deleteTopic(arn); 
    } 

```

所有这些方法显然都非常简单，因此可以直接在调用代码中内联调用 SNS API，但这个包装器确实为我们提供了一种将 API 的细节隐藏在业务代码之后的方法。例如，在`createTopic()`中更重要，需要额外的类，但为了保持一致，我们将把所有东西封装在我们自己的外观后面。

# 部署函数

我们现在已经完成了我们的函数，几乎可以准备部署它了。为此，我们需要打包它。AWS 允许我们上传 ZIP 或 JAR 文件。我们将使用后者。但是，我们有一些外部依赖项，因此我们将使用**Maven Shade**插件来构建一个包含函数及其所有依赖项的 fat jar。在`function`模块中，向`pom.xml`文件添加以下代码：

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

现在，当我们构建项目时，我们将在目标目录中得到一个大文件（约 9MB）。就是这个文件我们将上传。

# 创建角色

在上传函数之前，我们需要通过创建适当的角色来准备我们的 AWS 环境。登录到 AWS 并转到身份和访问管理控制台（[`console.aws.amazon.com/iam`](https://console.aws.amazon.com/iam)）。在左侧的导航窗格中，点击“角色”，然后点击“创建新角色”：

![](img/902f3791-0390-4254-9898-b8c656dde857.png)

在提示选择角色时，我们要选择 AWS Lambda。在下一页上，我们将附加策略：

![](img/22029503-fc48-4f3a-878c-1f7ecb97284b.png)

点击“下一步”，将名称设置为`j9bp`，然后点击“创建角色”。

# 创建主题

为了使创建函数和相关触发器更简单，我们将首先创建我们的主题。转到 SNS 控制台。鉴于并非所有 AWS 功能在每个区域都始终可用，我们需要选择特定的区域。我们可以在网页的左上角进行选择。如果区域不是 N. Virginia，请在继续之前从下拉菜单中选择 US East（N. Virginia）。

设置区域正确后，点击左侧导航栏中的“主题”，然后点击“创建新主题”，并将名称指定为`cloud-notice`：

![](img/c5ad91e2-4387-498f-b031-94b3d2f67433.png)

# 部署函数

现在我们可以转到 Lambda 控制台并部署我们的函数。我们将首先点击“创建 lambda 函数”按钮。我们将被要求选择一个蓝图。适用于基于 Java 的函数的唯一选项是空白函数。一旦我们点击该选项，就会出现“配置触发器”屏幕。当您点击空白方块时，将会出现一个下拉菜单，如 AWS 控制台中的此屏幕截图所示：

![](img/e1d99251-e9a1-4944-bed6-a1a6ae1b4d7f.png)

您可以向下滚动以找到 SNS，或者在过滤框中输入`SNS`，如前面的屏幕截图所示。无论哪种方式，当您在列表中点击 SNS 时，都会要求您选择要订阅的主题：

![](img/ac237c0d-a24c-431e-95c6-787008c68f90.png)

点击“下一步”。现在我们需要指定函数的详细信息：

![](img/90f63289-4ee5-4ba1-995b-2aa8bbc775c4.png)

向下滚动页面时，我们还需要指定 Lambda 函数处理程序和角色。处理程序是完全限定的类名，后跟两个冒号和方法名：

![](img/6b62cb07-215c-485f-8750-b8a0ac9a5240.png)

现在，我们需要通过点击上传按钮并选择由 Maven 构建创建的 jar 文件来选择函数存档。点击“下一步”，验证函数的详细信息，然后点击“创建函数”。

现在我们有一个可用的 AWS Lambda 函数。我们可以使用 Lambda 控制台进行测试，但我们将构建一个小的 JavaFX 应用程序来进行测试，这将同时测试所有服务集成，并演示生产应用程序如何与函数交互。

# 测试函数

为了帮助测试和演示系统，我们将在`CloudNotice`项目中创建一个名为`manager`的新模块。要做到这一点，点击 NetBeans 项目资源管理器中的模块节点，然后点击“创建新模块... | Maven | JavaFX 应用程序”。将项目命名为`Manager`，然后点击“完成”。

我已将`MainApp`重命名为`CloudNoticeManager`，`FXMLController`重命名为`CloudNoticeManagerController`，`Scene.fxml`重命名为`manager.fxml`。

我们的`Application`类将与以前的 JavaFX 应用程序有所不同。一些 AWS 客户端 API 要求在完成后明确关闭它们。未能这样做意味着我们的应用程序不会完全退出，留下必须被终止的**僵尸**进程。为了确保我们正确关闭 AWS 客户端，我们需要在我们的控制器中添加一个清理方法，并从我们的应用程序的`stop()`方法中调用它：

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

现在，无论用户是点击文件|退出还是点击窗口上的关闭按钮，我们的 AWS 客户端都可以正确地进行清理。

在布局方面，没有什么新的可讨论的，所以我们不会在这方面详细讨论。这就是我们的管理应用程序将会是什么样子：

![](img/c6a2fe65-aebb-4ccf-a83b-d30bec8c4ef8.png)

左侧是订阅接收者的列表，右上方是添加和编辑接收者的区域，右下方是发送测试消息的区域。我们有一些有趣的绑定，让我们来看看这些。

首先，在`CloudNoticeManagerController`中，我们需要声明一些数据的容器，因此我们声明了一些`ObservableList`实例：

```java
    private final ObservableList<Recipient> recips =  
      FXCollections.observableArrayList(); 
    private final ObservableList<String> types =  
      FXCollections.observableArrayList("SMS", "Email"); 
    private final ObservableList<String> topics =  
      FXCollections.observableArrayList(); 

```

这三个`ObservableList`实例将支持与它们的名称匹配的 UI 控件。我们将在`initalize()`中填充其中两个列表（`type`是硬编码）如下：

```java
    public void initialize(URL url, ResourceBundle rb) { 
      recips.setAll(dao.getRecipients()); 
      topics.setAll(sns.getTopics()); 

      type.setItems(types); 
      recipList.setItems(recips); 
      topicCombo.setItems(topics); 

```

使用我们的 DAO 和 SES 客户端，我们获取已经订阅的接收者，以及帐户中配置的任何主题。这将获取*每个*主题，所以如果你有很多，这可能是一个问题，但这只是一个演示应用程序，所以在这里应该没问题。一旦我们有了这两个列表，我们将它们添加到之前创建的`ObservableList`实例中，然后将`List`与适当的 UI 控件关联起来。

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

请记住，如果单元格为空，我们需要将文本设置为 null 以清除任何先前的值。未能这样做将导致`ListView`在某个时候出现**幻影**条目。

接下来，当用户点击列表中的`Recipient`时，我们需要更新编辑控件。我们通过向`selectedItemProperty`添加监听器来实现这一点，每当选定的项目更改时运行：

```java
    recipList.getSelectionModel().selectedItemProperty() 
            .addListener((obs, oldRecipient, newRecipient) -> { 
        type.valueProperty().setValue(newRecipient != null ?  
            newRecipient.getType() : ""); 
        address.setText(newRecipient != null ?  
            newRecipient.getAddress() : ""); 
    }); 

```

如果`newRecipient`不为空，我们将控件的值设置为适当的值。否则，我们清除值。

现在我们需要为各种按钮添加处理程序--在`Recipient`列表上方的添加和删除按钮，以及右侧两个**表单**区域中的`Save`和`Cancel`按钮。

UI 控件的`onAction`属性可以通过直接编辑 FXML 来绑定到类中的方法，如下所示：

```java
    <Button mnemonicParsing="false"  
      onAction="#addRecipient" text="+" /> 
    <Button mnemonicParsing="false"  
      onAction="#removeRecipient" text="-" /> 

```

也可以通过在 Scene Builder 中编辑属性来将其绑定到方法，如下面的屏幕截图所示：

![](img/ba0fd788-85bb-46f1-9eef-c5a71cb3ea43.png)

无论哪种方式，该方法将如下所示：

```java
    @FXML 
    public void addRecipient(ActionEvent event) { 
      final Recipient recipient = new Recipient(); 
      recips.add(recipient); 
      recipList.getSelectionModel().select(recipient); 
      type.requestFocus(); 
    } 

```

我们正在添加一个`Recipient`，因此我们创建一个新的`Recipient`，将其添加到我们的`ObservableList`，然后告诉`ListView`选择此条目。最后，我们要求`type`控件请求焦点，以便用户可以轻松地使用键盘更改值。新的 Recipient 直到用户点击保存才保存到 DynamoDB，我们将在稍后讨论。

当我们删除一个`Recipient`时，我们需要将其从 UI 和 DynamoDB 中删除：

```java
    @FXML 
    public void removeRecipient(ActionEvent event) { 
      final Recipient recipient = recipList.getSelectionModel() 
       .getSelectedItem(); 
      dao.deleteRecipient(recipient); 
      recips.remove(recipient); 
    } 

```

保存有点复杂，但不多：

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

由于我们没有将编辑控件的值绑定到列表中的选定项目，所以我们需要获取项目的引用，然后将控件的值复制到模型中。完成后，我们将其保存到数据库通过我们的 DAO，然后要求`ListView`刷新自身，以便列表中反映任何模型更改。

我们没有将控件绑定到列表中的项目，因为这会导致用户体验稍微混乱。如果我们进行绑定，当用户对模型进行更改时，`ListView`将反映这些更改。用户可能会认为更改已保存到数据库，而实际上并没有。直到用户点击保存才会发生。为了避免这种混淆和数据丢失，我们*没有*绑定控件，而是手动管理数据。

要取消更改，我们只需要从`ListView`获取对未更改模型的引用，并将其值复制到编辑控件中：

```java
    @FXML 
    public void cancelChanges(ActionEvent event) { 
      final Recipient recipient = recipList.getSelectionModel() 
        .getSelectedItem(); 
      type.setValue(recipient.getType()); 
      address.setText(recipient.getAddress()); 
    } 

```

这留下了我们的 UI 中的**发送消息**部分。由于我们的 SNS 包装 API，这些方法非常简单：

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

从我们的桌面应用程序，我们现在可以添加、编辑和删除收件人，以及发送测试消息。

# 配置您的 AWS 凭证

非常关注的人可能会问一个非常重要的问题--AWS 客户端库如何知道如何登录到我们的账户？显然，我们需要告诉它们，而且我们有几个选项。

当本地运行 AWS SDK 时，将检查三个位置的凭证--环境变量（`AWS_ACCESS_KEY_ID`和`AWS_SECRET_ACCESS_KEY`）、系统属性（`aws.accessKeyId`和`aws.secretKey`）和默认凭证配置文件（`$HOME/.aws/credentials`）。您使用哪些凭证取决于您，但我将在这里向您展示如何配置配置文件。

就像 Unix 或 Windows 系统一样，您的 AWS 账户有一个具有对系统完全访问权限的`root`用户。以此用户身份运行任何客户端代码将是非常不慎重的。为了避免这种情况，我们需要创建一个用户，我们可以在身份和访问管理控制台上完成（[`console.aws.amazon.com/iam`](https://console.aws.amazon.com/iam)）。

登录后，点击左侧的“用户”，然后点击顶部的“添加用户”，结果如下截图所示：

![](img/cea72534-8eff-4149-bd1d-d768aaaeafa8.png)

点击“下一步：权限”，并在组列表中检查我们角色`j9bp`的条目。点击“下一步：审阅”，然后创建用户。这将带您到添加用户屏幕，屏幕底部列出了用户信息。在表格的右侧，您应该看到访问密钥 ID 和秘密访问密钥列。点击访问密钥上的“显示”以显示值。记下这两个值，因为一旦离开此页面，就无法检索访问密钥。如果丢失，您将不得不生成新的密钥对，这将破坏使用旧凭证的任何其他应用程序。

![](img/2231a001-356f-41da-8b39-bedeed92c4b2.png)

在文本编辑器中，我们需要创建`~/.aws/credentials`文件。在 Unix 系统上，可能是`/home/jdlee/.aws`，在 Windows 机器上可能是`C:\Users\jdlee\aws`。凭证文件应该看起来像这样：

```java
    [default] 
    aws_access_key_id = AKIAISQVOILE6KCNQ7EQ 
    aws_secret_access_key = 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 

```

在同一个目录中，我们需要创建另一个名为`config`的文件。我们将使用这个文件告诉 SDK 我们想要在哪个地区工作：

```java
    [default] 
    region = us-east-1 

```

现在，当 AWS 客户端启动时，它们将默认连接到`us-east-1`地区的`j9bp`用户。如果需要覆盖这一点，您可以编辑此文件，或者设置上面“配置您的 AWS 凭证”部分中提到的环境变量或系统属性。

# 摘要

我们做到了！我们中的许多人创建了我们的第一个 AWS Lambda 函数，而且真的并不那么困难。当然，这是一个简单的应用程序，但我希望你能看到这种类型的应用程序可能非常有用。以此为起点，你可以编写系统，借助移动应用程序，帮助跟踪你家人的位置。例如，你可以使用树莓派等嵌入式设备，构建设备来跟踪随着货物在全国范围内的运输情况，报告位置、速度、环境条件、突然的下降或冲击等。在服务器上运行的软件可以不断报告系统的各种指标，如 CPU 温度、空闲磁盘空间、分配的内存、系统负载等等。你的选择只受你的想象力限制。

总结一下，让我们快速回顾一下我们学到的东西。我们了解了一些当今提供的各种“...作为服务”系统，以及“无服务器”到底意味着什么，以及为什么它可能吸引我们作为应用程序开发人员。我们学会了如何配置各种亚马逊网络服务，包括身份和访问管理、简单通知系统、简单电子邮件服务，当然还有 Lambda，我们学会了如何用 Java 编写 AWS Lambda 函数以及如何部署它到服务上。最后，我们学会了如何配置触发器，将 SNS 发布/订阅主题与我们的 Lambda 函数联系起来。

毫无疑问，我们的应用程序有些简单，而在单一章节的空间内，无法让你成为亚马逊网络服务或任何其他云服务提供商所提供的所有内容的专家。希望你有足够的知识让你开始，并让你对使用 Java 编写基于云的应用程序感到兴奋。对于那些想要深入了解的人，有许多优秀的书籍、网页等可以帮助你更深入地了解这个快速变化和扩展的领域。在我们的下一章中，我们将离开云，把注意力转向另一个对 Java 开发人员来说非常重要的领域——你的手机。
