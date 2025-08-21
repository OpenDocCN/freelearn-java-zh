# 第十章：端到端的示例

我们已经涵盖了足够的内容，可以让我们在实际项目中使用 Spring Integration。让我们构建一个真正的应用程序，这将练习 Spring Integration 模块暴露的不同类型的组件。这还将作为一个刷新章节，因为我们将访问到目前为止讨论的所有概念。

让我们以 Feeds 聚合器应用程序为例；它将根据配置参数聚合 Feed，然后将其传达给感兴趣的各方。以下是我们要尝试解决的问题的大纲。这些只是为了示例，在实际场景中，我们可能不需要聚合器或分割器，或者处理序列本身可能会有所不同：

+   数据摄取可以通过：

    +   阅读 RSS 源

    +   从 FTP 服务器上的文件中读取问题

+   过滤数据：

    +   根据完成标准过滤有效/无效消息；为了简单起见，我们将过滤掉`java`问题

+   聚合消息：只是为了展示示例，我们将聚合并发布五组消息

+   分割消息：聚合消息列表将被分割并沿线发送以进行进一步处理

+   转换：

    +   将消息转换为可以写入数据库的格式

    +   将 JMS 格式的消息转换为可以放入消息队列的消息

    +   将邮件格式的消息转换，以便可以发送给订阅的收件人

+   根据消息类型路由消息；实体类型到数据库消费者，消息类型到 JMS 消费者，电子邮件消息到电子邮件发送者

+   与外部系统集成：

    +   写入数据库

    +   放置在 JMS 上

    +   使用电子邮件适配器发送邮件

+   JMX：暴露 Spring 管理监控端点

# 先决条件

在我们可以开始示例之前，我们需要以下软件来导入并运行项目：

+   一个 Java IDE（最好是 STS，但任何其他 IDE，如 Eclipse 或 NetBeans 也行）

+   JDK 1.6 及以上

+   Maven

+   FTP 服务器（这是可选的，只有在启用时才需要）

## 设置

一旦我们有了所有先决条件，按照以下步骤启动程序：

1.  检查你下载的代码包中的项目。这是一个 Maven 项目，所以使用你选择的 IDE，将其作为 Maven 项目导入。

1.  在`settings.properties`中为电子邮件、JMS 和 FTP 账户添加设置：

    ```java
    #URL of RSS feed, as example http://stackoverflow.com/feeds -Make #sure there are not copyright or legal issues in consumption of
    #feed
    feeds.url=some valid feed URL 
    #Username for e-mail account
    mail.username=yourusername
    #Password for e-mail account
    mail.password=yourpassword
    #FTP server host
    ftp.host=localhost
    #FTP port
    ftp.port=21
    #Remote directory on FTP which the listener would be observing
    ftp.remotefolder=/
    #Local directory where downloaded file should be dumped
    ftp.localfolder=C:\\Chandan\\Projects\\siexample\\ftp\\ftplocalfolder
    #Username for connecting to FTP server
    ftp.username=ftpusername
    #Password for connection to FTP server
    ftp.password=ftppassword
    #JMS broker URL
    jms.brolerurl=vm://localhost
    ```

1.  准备好一个 FTP 账户和一个电子邮件账户。

1.  从主类运行，即`FeedsExample`。

# 数据摄取：

让我们从第一步开始，数据摄取。我们配置了两个数据源：RSS 源和一个 FTP 服务器，让我们来看看这些。

## 从 RSS 源摄取数据

```java
adapter; this fetches feed from the configured url and puts it on the channel:
```

```java
<int-feed:inbound-channel-adapter 
  id="soJavaFeedAdapterForAggregator" 
  channel="fetchedFeedChannel" 
  auto-startup="true" 
  url="${feeds.url}"> 
  <int:poller 
    fixed-rate="500" max-messages-per-poll="1" />
</int-feed:inbound-channel-adapter>
```

### 提示

我将展示代码并解释它做什么，但不会详细介绍每个和每个标签，因为它们已经在相应的章节中涵盖了。

## 从 FTP 服务器摄取数据

为了让这一切工作，你需要一个配置好的 FTP 服务器。为了测试，你总是可以在本地设置一个 FTP 服务器。根据你的 FTP 服务器位置和配置参数，设置一个会话工厂：

```java
<!-- FTP Create Session-->
  <bean id="ftpClientSessionFactory" class="org.springframework.integration.ftp.session.DefaultFtpSessionFactory">
    <property name="host" value="${ftp.host}"/>
    <property name="port" value="${ftp.port}"/>
    <property name="username" value="${ftp.username}"/>
    <property name="password" value="${ftp.password}"/>
  </bean>
```

设置会话工厂后，它可以用来与 FTP 服务器建立连接。以下代码将从 FTP 的配置`远程目录`下载新文件，并将其放在`本地目录`中：

```java
<!-- FTP Download files from server and put it in local directory-->
  <int-ftp:inbound-channel-adapter 
    channel="fetchedFeedChannel"
    session-factory="ftpClientSessionFactory"
    remote-directory="${ftp.remotefolder}"
    local-directory="${ftp.localfolder}"
    auto-create-local-directory="true"
    delete-remote-files="true"
    filename-pattern="*.txt"
    local-filename-generator-expression="#this.toLowerCase() + '.trns'">
    <int:poller fixed-rate="1000"/>
  </int-ftp:inbound-channel-adapter>
```

## 过滤数据

馈送和 FTP 适配器获取馈送并将其放入`获取馈送通道`。让我们配置一个过滤器，在读取馈送时只允许 Java 相关的问题。它将从一个通道`获取馈送通道`读取馈送，并将过滤后的馈送传递给通道`获取馈送通道用于聚合器`。以下代码片段是 Spring 配置：

```java
  <bean id="filterSoFeedBean" class="com.cpandey.siexample.filter.SoFeedFilter"/>
  <!--Filter the feed which are not for Java category -->
<int:filter input-channel="fetchedFeedChannel" output-channel="fetchedFeedChannelForAggregatior" ref="filterSoFeedBean" method="filterFeed"/>
```

以下是包含过滤逻辑的 JavaBean 类：

```java
import java.util.List;
import org.apache.log4j.Logger;
import org.springframework.messaging.Message;
import com.sun.syndication.feed.synd.SyndCategoryImpl;
import com.sun.syndication.feed.synd.SyndEntry;

public class SoFeedFilter {
  private static final Logger LOGGER = Logger.getLogger(SoFeedFilter.class);
  public boolean filterFeed(Message<SyndEntry> message){
    SyndEntry entry = message.getPayload();
    List<SyndCategoryImpl> categories=entry.getCategories();
    if(categories!=null&&categories.size()>0){
      for (SyndCategoryImpl category: categories) {
        if(category.getName().equalsIgnoreCase("java")){
          LOGGER.info("JAVA category feed");
          return true;
        }
      }
    }
    return false;
  }
}
```

# 聚合器

聚合器用于展示聚合器的使用。聚合器被插在过滤器的输出通道上，即`获取馈送通道用于聚合器`。我们将使用聚合器的所有三个组件：关联、完成和聚合器。让我们声明 bean：

```java
  <bean id="soFeedCorrelationStrategyBean" class="com.cpandey.siexample.aggregator.CorrelationStrategy"/>

  <bean id="sofeedCompletionStrategyBean" class="com.cpandey.siexample.aggregator.CompletionStrategy"/>

  <bean id="aggregatorSoFeedBean" class="com.cpandey.siexample.aggregator.SoFeedAggregator"/>
```

在我们定义了聚合器的三个关键组件之后，让我们定义一个组件，它将一组五个馈送进行聚合，然后仅在下一个通道发布：

```java
  <int:aggregator input-channel="fetchedFeedChannelForAggregatior"
    output-channel="aggregatedFeedChannel" ref="aggregatorSoFeedBean"
    method="aggregateAndPublish" release-strategy="sofeedCompletionStrategyBean"
    release-strategy-method="checkCompleteness" correlation-strategy="soFeedCorrelationStrategyBean"
    correlation-strategy-method="groupFeedsBasedOnCategory"
    message-store="messageStore" expire-groups-upon-completion="true">
    <int:poller fixed-rate="1000"></int:poller>
  </int:aggregator>
```

## 关联 bean

如果你记得，关联 bean 持有分组“相关”项的策略。我们将简单地使用馈送的类别来分组消息：

```java
import java.util.List;
import org.apache.log4j.Logger;
import org.springframework.messaging.Message;
import com.sun.syndication.feed.synd.SyndCategoryImpl;
import com.sun.syndication.feed.synd.SyndEntry;

public class CorrelationStrategy {
  private static final Logger LOGGER = Logger.getLogger(CorrelationStrategy.class);

  //aggregator's method should expect a Message<?> and return an //Object.
  public Object groupFeedsBasedOnCategory(Message<?> message) {
    //Which messages will be grouped in a bucket 
    //-say based on category, based on some ID etc.
    if(message!=null){
      SyndEntry entry = (SyndEntry)message.getPayload();
      List<SyndCategoryImpl> categories=entry.getCategories();
      if(categories!=null&&categories.size()>0){
        for (SyndCategoryImpl category: categories) {
          //for simplicity, lets consider the first category
          LOGGER.info("category "+category.getName());
          return category.getName();
        }
      }
    }
    return null;
  }
}
```

## 完成 bean

我们已经关联了消息，但我们将会持有列表多久？这将由完成标准来决定。让我们设定一个简单的标准，如果有五个同一类别的馈送，那么释放它进行进一步处理。以下是实现这个标准的类：

```java
import java.util.List;
import org.apache.log4j.Logger;
import com.sun.syndication.feed.synd.SyndEntry;

public class CompletionStrategy {
  private static final Logger LOGGER = Logger.getLogger(CompletionStrategy.class);
  //Completion strategy is used by aggregator to decide whether all //components has
  //been aggregated or not method should expect a java.util.List 
  //Object returning a Boolean value
  public boolean checkCompleteness(List<SyndEntry> messages) {
    if(messages!=null){
      if(messages.size()>4){
        LOGGER.info("All components assembled, releasing aggregated message");
        return true;
      }
    }
    return false;
  }

}
```

## 聚合器 bean

馈送将会被关联，在满足完成标准后，聚合器将在下一个端点返回列表。我们之前已经定义了关联策略和完成标准，让我们看看聚合器的代码：

```java
import java.util.List;
import org.apache.log4j.Logger;
import com.sun.syndication.feed.synd.SyndEntry;

public class SoFeedAggregator {
  private static final Logger LOGGER = Logger.getLogger(SoFeedAggregator.class);
  public List<SyndEntry> aggregateAndPublish( List<SyndEntry> messages) {
    LOGGER.info("SoFeedAggregator -Aggregation complete");
    return messages;
  }
}
```

# 分割器

```java
<int:splitter ref="splitterSoFeedBean" method="splitAndPublish" input-channel="aggregatedFeedChannel" output-channel="splittedFeedChannel" />
```

包含分割逻辑的 JavaBean：

```java
import java.util.List;
import com.sun.syndication.feed.synd.SyndEntry;
public class SoFeedSplitter {
  public List<SyndEntry> splitAndPublish(List<SyndEntry> message) {
    //Return one message from list at a time -this will be picked up //by the processor
    return message;
  }
}
```

# 转换

现在我们有了 RSS 格式的馈送，让我们将其转换为适当的格式，以便负责将馈送持久化到数据库、将其放入 JMS 通道和发送邮件的端点可以理解。分割器将一次在通道`分割馈送通道`上放置一个消息。让我们将其声明为发布-订阅通道，并附加三个端点，这些将是我们的转换器。如下配置发布-订阅通道：

```java
<int:publish-subscribe-channel id="splittedFeedChannel"/>
```

我们使用的三个转换器的配置如下：

```java
  <bean id="feedDbTransformerBean" class="com.cpandey.siexample.transformer.SoFeedDbTransformer" />

  <bean id="feedJMSTransformerBean" class="com.cpandey.siexample.transformer.SoFeedJMSTransformer" />

  <bean id="feedMailTransformerBean" class="com.cpandey.siexample.transformer.SoFeedMailTransformer" />
```

## 数据库转换器

让我们从 Spring Integration 和包含转换逻辑的 Java 类编写转换器组件：

```java
<int:transformer id="dbFeedTransformer" ref="feedDbTransformerBean" input-channel="splittedFeedChannel" method="transformFeed" output-channel="transformedChannel"/>

import org.apache.log4j.Logger;
import org.springframework.messaging.Message;
import com.cpandey.siexample.pojo.SoFeed;
import com.sun.syndication.feed.synd.SyndEntry;

public class SoFeedDbTransformer {
  private static final Logger LOGGER = Logger.getLogger(SoFeedDbTransformer.class);

  public SoFeed transformFeed(Message<SyndEntry> message){
    SyndEntry entry = message.getPayload();
    SoFeed soFeed=new SoFeed();
    soFeed.setTitle(entry.getTitle());
    soFeed.setDescription(entry.getDescription().getValue());
    soFeed.setCategories(entry.getCategories());
    soFeed.setLink(entry.getLink());
    soFeed.setAuthor(entry.getAuthor());
    LOGGER.info("JDBC :: "+soFeed.getTitle());
    return soFeed;
  }
}
```

## JMS 转换器

以下是 JMS 转换器组件声明的代码以及相应的 JavaBean：

```java
<int:transformer id="jmsFeedTransformer" ref="feedJMSTransformerBean" 
  input-channel="splittedFeedChannel" 
  method="transformFeed" 
  output-channel="transformedChannel"/>

import org.apache.log4j.Logger;
import org.springframework.messaging.Message;
import com.cpandey.siexample.pojo.SoFeed;
import com.sun.syndication.feed.synd.SyndEntry;
public class SoFeedJMSTransformer {
  private static final Logger LOGGER = Logger.getLogger(SoFeedJMSTransformer.class);

  public String transformFeed(Message<SyndEntry> message){
    SyndEntry entry = message.getPayload();
    SoFeed soFeed=new SoFeed();
    soFeed.setTitle(entry.getTitle());
    soFeed.setDescription(entry.getDescription().getValue());
    soFeed.setCategories(entry.getCategories());
    soFeed.setLink(entry.getLink());
    soFeed.setAuthor(entry.getAuthor());
    //For JSM , return String 
    LOGGER.info("JMS"+soFeed.getTitle());
    return soFeed.toString();
  }
}
```

## 邮件转换器

最后，让我们编写邮件转换器的配置和代码：

```java
<int:transformer id="mailFeedTransformer" ref="feedMailTransformerBean" 
  input-channel="splittedFeedChannel"
  method="transformFeed" 
  output-channel="transformedChannel"/>

import java.util.Date;
import org.apache.log4j.Logger;
import org.springframework.mail.MailMessage;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.messaging.Message;
import com.cpandey.siexample.pojo.SoFeed;
import com.sun.syndication.feed.synd.SyndEntry;

public class SoFeedMailTransformer {
  private static final Logger LOGGER = Logger.getLogger(SoFeedMailTransformer.class);

  public MailMessage transformFeed(Message<SyndEntry> message){
    SyndEntry entry = message.getPayload();
    SoFeed soFeed=new SoFeed();
    soFeed.setTitle(entry.getTitle());
    soFeed.setDescription(entry.getDescription().getValue());
    soFeed.setCategories(entry.getCategories());
    soFeed.setLink(entry.getLink());
    soFeed.setAuthor(entry.getAuthor());

    //For Mail return MailMessage
    MailMessage msg = new SimpleMailMessage();
    msg.setTo("emailaddress");
    msg.setFrom("emailaddress");
    msg.setSubject("Subject");
    msg.setSentDate(new Date());
    msg.setText("Mail Text");
    LOGGER.info("Mail Message"+soFeed.getTitle());

     return msg;
  }
}
```

# 路由器

在将消息转换为适当格式后，转换器将消息放入`transformedChannel`通道。我们将处理三种不同类型的消息，这些消息将由不同的端点处理。我们可以使用载荷路由器，根据载荷类型将其路由到不同的组件：

```java
    <int:payload-type-router input-channel="transformedChannel" 
      default-output-channel="logChannel">
    <int:mapping type="com.cpandey.siexample.pojo.SoFeed"
      channel="jdbcChannel" />
    <int:mapping type="java.lang.String" 
      channel="jmsChannel" />
    <int:mapping type="org.springframework.mail.MailMessage" 
      channel="mailChannel" />
    </int:payload-type-router>
```

# 集成

现在是实际集成的时刻！一旦路由器将消息路由到适当的端点，它应该被这些端点处理。例如，它可以被持久化到数据库，通过 JMS 通道发送，或者作为电子邮件发送。根据载荷类型，路由器将消息放入`jdbcChannel`、`jmsChannel`或`mailChannel`中的一个通道。如果它无法理解载荷，它将把消息路由到`logChannel`。让我们从与`jdbcChannel`通道关联的端点开始，该通道用于数据库集成。

## 数据库集成

在本节中，我们将编写代码以从数据库添加和查询数据。在我们将 Spring Integration 的适配器编写之前，让我们先完成基本设置。

### 先决条件

显而易见，我们需要一个数据库来存储数据。为了简化，我们将使用内存数据库。我们还需要配置 ORM 提供者、事务以及其他与数据库一起使用的方面：

+   嵌入式数据库的声明：

    ```java
      <jdbc:embedded-database id="dataSource" type="H2"/>
    ```

+   事务管理器的声明：

    ```java
      <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <constructor-arg ref="entityManagerFactory" />
      </bean>
    ```

+   实体管理工厂的声明：

    ```java
    <bean id="entityManagerFactory"
      class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
      <property name="dataSource"  ref="dataSource" />
      <property name="jpaVendorAdapter" ref="vendorAdaptor" />
      <property name="packagesToScan" value="com.cpandey.siexample.pojo"/>
      </bean>
    ```

+   实体管理器的声明：

    ```java
    <bean id="entityManager" class="org.springframework.orm.jpa.support.SharedEntityManagerBean">
        <property name="entityManagerFactory" ref="entityManagerFactory"/>
      </bean>
    ```

+   抽象供应商适配器的声明：

    ```java
    <bean id="abstractVendorAdapter" abstract="true">
      <property name="generateDdl" value="true" />
      <property name="database"    value="H2" />
      <property name="showSql"     value="false"/>
    </bean>
    ```

+   实际供应商适配器的声明，在我们的案例中，它是 hibernate：

    ```java
      <bean id="vendorAdaptor" class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter"
        parent="abstractVendorAdaptor">
      </bean>
    ```

### 网关

让我们定义一个网关，它将插入调用方法来插入数据流，然后从数据库中读取它们：

```java
<int:gateway id="feedService"
  service-interface="com.cpandey.siexample.service.FeedService"
  default-request-timeout="5000"
  default-reply-timeout="5000">
  <int:method name="createFeed"
    request-channel="createFeedRequestChannel"/>
  <int:method name="readAllFeed"
    reply-channel="readFeedRequestChannel"/>
</int:gateway>
```

网关的 Bean 定义如下：

```java
import java.util.List;
import com.cpandey.siexample.pojo.FeedEntity;
public interface FeedService {
  FeedEntity createFeed(FeedEntity feed);
  List<FeedEntity> readAllFeed();
}
```

### 服务激活器

此服务激活器被连接到`jdbcChannel`通道。当消息到达时，它的`persistFeedToDb`方法被调用，该方法使用前面的网关将数据流持久化：

```java
import org.apache.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.integration.annotation.MessageEndpoint;
import org.springframework.integration.annotation.ServiceActivator;
import com.cpandey.siexample.pojo.FeedEntity;
import com.cpandey.siexample.pojo.SoFeed;

@MessageEndpoint
public class PersistFeed {

  private static final Logger LOGGER = Logger.getLogger(PersistFeed.class);

  @Autowired FeedService feedService;
  @ServiceActivator
  public void persistFeedToDb(SoFeed feed) {
    //This will write to output channel of gateway
    //From there this will be picked by updating adapter
    feedService.createFeed(new FeedEntity(feed.getTitle()));
  }

  @ServiceActivator
  public void printFeed(FeedEntity feed) {
    //Print the feed fetched by retrieving adapter
    LOGGER.info("Feed Id"+feed.getId()+" Feed Title "+feed.getTitle());
  }
}
```

### 用于更新和读取数据流的网关：

最后，我们将 Spring Integration 更新和检索出站网关的功能集成进来，以持久化和从数据库中读取数据流：

```java
  <int-jpa:updating-outbound-gateway 
    entity-manager-factory="entityManagerFactory"
    request-channel="createFeedRequestChannel" 
    entity-class="com.cpandey.siexample.pojo.FeedEntity" 
    reply-channel="printAllFeedChannel">
    <int-jpa:transactional transaction-manager="transactionManager" />
  </int-jpa:updating-outbound-gateway>

  <int-jpa:retrieving-outbound-gateway 
    entity-manager-factory="entityManagerFactory"
    request-channel="readFeedRequestChannel"
    jpa-query="select f from FeedEntity f order by f.title asc" 
    reply-channel="printAllFeedChannel">
  </int-jpa:retrieving-outbound-gateway>
```

## 发送邮件

我们可以使用 Spring Integration 邮件出站通道适配器来发送邮件。它需要对邮件发送者类的引用，该类已按照以下方式配置：

+   Spring Integration 发送邮件的组件：

    ```java
      <int-mail:outbound-channel-adapter channel="mailChannel" mail-sender="mailSender"/>
    ```

    如前面的配置所示，此适配器被连接到`mailChannel`—路由器将消息路由到的其他通道之一。

+   前一个组件使用的邮件发送者：

    ```java
      <bean id="mailSender" class="org.springframework.mail.javamail.JavaMailSenderImpl">
        <property name="javaMailProperties">
          <props>
            <prop key="mail.smtp.auth">true</prop>
            <prop key="mail.smtp.starttls.enable">true</prop>
            <prop key="mail.smtp.host">smtp.gmail.com</prop>
            <prop key="mail.smtp.port">587</prop>
          </props>
        </property>
        <property name="username" value="${mail.username}" />
        <property name="password" value="${mail.password}" />
      </bean>
    ```

## 将消息放入 JMS 队列

最后，让我们使用出站通道适配器将消息放入 JMS 队列，此适配器轮询`jmsChannel`通道以获取消息，每当路由器将消息路由至此处，它都会将其放入`destination`队列：

```java
  <int-jms:outbound-channel-adapter connection-factory="connectionFactory" channel="jmsChannel" destination="feedInputQueue" />
```

为了测试队列中的消息，让我们添加一个简单的服务激活器：

```java
<int:service-activator ref="commonServiceActivator" method="echoJmsMessageInput" input-channel="jmsProcessedChannel"/>
```

从之前的配置中可以看出，我们需要`destination`和`connection-factory`，让我们来配置这些：

```java
  <bean id="feedInputQueue" class="org.apache.activemq.command.ActiveMQQueue">
    <constructor-arg value="queue.input"/>
  </bean>

  <bean id="connectionFactory" 
    class="org.springframework.jms.connection.CachingConnectionFactory">
    <property name="targetConnectionFactory">
      <bean class="org.apache.activemq.ActiveMQConnectionFactory">
        <property name="brokerURL" value="${jms.brokerurl}"/>
      </bean>
    </property>
    <property name="sessionCacheSize" value="10"/>
    <property name="cacheProducers" value="false"/>
  </bean>
```

# 导出为 MBean

最后，让我们添加代码以导出作为 MBean 使用的组件，这可以通过 JConsole 或其他 JMX 工具进行监控：

```java
  <int-jmx:mbean-export 
    default-domain="com.cpandey.siexample"
    server="mbeanServer"/>
```

# 摘要

在本章中，我们覆盖了一个端到端的示例；我希望这很有用，并且能在一个地方刷新概念和完整的用例。有了这个，我们的 Spring Integration 之旅就结束了。我希望你喜欢它！

我们覆盖了 Spring Integration 框架的绝大多数常用特性，并介绍了足够的内容来获得动力。如果这本书让你对使用 Spring Integration 感到兴奋，那么你的下一个目的地应该是[`docs.spring.io/spring-integration/reference/htmlsingle`](http://docs.spring.io/spring-integration/reference/htmlsingle)官方参考资料。
