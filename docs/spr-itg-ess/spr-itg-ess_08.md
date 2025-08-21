# 第八章．测试支持

**测试驱动** **开发**（**TDD**）已经改变了软件的开发和部署方式，为什么不呢，每个客户都想要运行良好的软件——证明它运行良好最好的方式就是测试它！Spring Integration 也不例外——那么我们如何测试每个“单元”是否可以独立运行呢？——事实上，测试单元的重要性甚至更大，这样任何集成问题都可以很容易地被隔离。例如，FTP 入站网关依赖于外部因素，如 FTP 服务器上的用户角色、FTP 服务器的性能、网络延迟等。我们如何验证连接到 FTP 入站网关的消费者可以在不实际连接到 FTP 服务器的情况下处理文件？我们可以将“模拟”消息发送到通道，消费者会将其视为来自 FTP 服务器的消息！我们想要证明的就是，给定文件到达通道，监听器将执行其工作。

在本章中，我将涵盖 Spring Integration 测试的方面——而且大部分，它将是一个“给我看代码”的章节！以下是涵盖的主题的大纲：

+   测试消息

+   测试头部

+   处理错误

+   测试过滤器

+   测试分割器

# 先决条件

那么测试需要什么？当然，JUnit！还有别的吗？Spring 框架和 Spring Integration 本身提供了许多模拟和支持类，帮助测试应用程序。让我们为这些类添加 maven 依赖项：

```java
  <dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-test</artifactId>
    <version>${spring.integration.version}</version>
  </dependency>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>${junit.version}</version>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>${spring.version}</version>
    <scope>test</scope>
  </dependency>
```

# 测试消息

Spring Integration 提供了一个类，可以帮助构建某些有效负载，例如以下示例：

```java
Message<String> message = MessageBuilder.withPayload("Test").build()
```

这些消息可以通过获取实际通道定义的句柄放在通道上。这可以用于负测试以及正测试。例如，如果监听通道的服务激活器期望一个具有`File`类型的有效负载的消息，那么放置一个具有`String`有效负载的消息应该表示一个错误。让我们为我们的转换器编写一个快速的测试，该转换器接受具有`SyndEntry`有效负载的`Message`并将其转换为`SoFeed`。以下是我们转换器类的代码片段：

```java
import org.springframework.messaging.Message;

import com.cpandey.siexample.pojo.SoFeed;
import com.sun.syndication.feed.synd.SyndEntry;

public class SoFeedDbTransformer {

  public SoFeed transformFeed(Message<SyndEntry> message){
    SyndEntry entry = message.getPayload();
    SoFeed soFeed=new SoFeed();
    soFeed.setTitle(entry.getTitle());
    soFeed.setDescription(entry.getDescription().getValue());
    soFeed.setCategories(entry.getCategories());
    soFeed.setLink(entry.getLink());
    soFeed.setAuthor(entry.getAuthor());

    System.out.println("JDBC"+soFeed.getTitle());
    return soFeed;
  }
}
```

如提及的，它接收到一个具有`SyndEntry`类型的有效负载的消息。让我们编写一个简单的测试用例，只有在从`SyndEntry`成功转换到`SoFeed`时才会通过：

```java
import static org.junit.Assert.assertNotNull;
import static org.junit.Assert.assertNull;
import static org.junit.Assert.assertThat;
import static org.springframework.integration.test.matcher.PayloadMatcher.hasPayload;

import java.util.ArrayList;
import java.util.List;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.integration.channel.QueueChannel;
import org.springframework.integration.support.MessageBuilder;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageChannel;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import com.cpandey.siexample.pojo.SoFeed;
import com.sun.syndication.feed.synd.SyndCategoryImpl;
import com.sun.syndication.feed.synd.SyndContent;
import com.sun.syndication.feed.synd.SyndContentImpl;
import com.sun.syndication.feed.synd.SyndEntry;
import com.sun.syndication.feed.synd.SyndEntryImpl;

@ContextConfiguration
@RunWith(SpringJUnit4ClassRunner.class)
public class TestSoDBFeedTransformer {
  @Autowired
  MessageChannel filteredFeedChannel;

  @Autowired
  QueueChannel transformedChannel;

  @Test
  public void messageIsConvertedToEntity() {
    //Define a dummy domain Object
    SyndEntry entry =new SyndEntryImpl();
    entry.setTitle("Test");
    SyndContent content=new SyndContentImpl();
    content.setValue("TestValue");
    entry.setDescription(content);
    List<SyndCategoryImpl> catList=new 
      ArrayList<SyndCategoryImpl>();
    entry.setCategories(catList);
    entry.setLink("TestLink");
    entry.setAuthor("TestAuthor");

//Define expected result
    SoFeed expectedSoFeed=new SoFeed();
    expectedSoFeed.setTitle(entry.getTitle());
    expectedSoFeed.setDescription(entry.getDescription
      ().getValue());

      expectedSoFeed.setCategories(entry.getCategories()
      );
    expectedSoFeed.setLink(entry.getLink());
    expectedSoFeed.setAuthor(entry.getAuthor());

    Message<SyndEntry> message = 
      MessageBuilder.withPayload(entry).build();
    filteredFeedChannel.send(message);
    Message<?> outMessage = 
      transformedChannel.receive(0);
    SoFeedsoFeedReceived
      =(SoFeed)outMessage.getPayload();
    assertNotNull(outMessage);
    assertThat(outMessage, 
      hasPayload(soFeedReceived));
    outMessage = transformedChannel.receive(0);
    assertNull("Only one message expected", 
      outMessage);
  }
```

在此代码中，使用`@ContextConfiguration`注解加载上下文信息。默认情况下，它会寻找类似于`<classname>-context.xml`的文件名和用`@Configuration`注解的 Java 配置类。在我们的案例中，它是`TestSoDBFeedTransformer-context.xml`。这包含运行测试所需的信息，如通道、服务定义等：

```java
<?xml version="1.0" encoding="UTF-8"?>
  <beans 

      xsi:schemaLocation="http://www.springframework.org/schema/integration http://www.springframework.org/schema/integration/spring-integration.xsd
    http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <int:channel id="filteredFeedChannel"/>
    <int:channel id="transformedChannel">
      <int:queue/>
    </int:channel>

    <bean id="feedDbTransformerBean" 
      class="com.cpandey.siexample.transformer.SoFeedDbTransformer" />
    <!-- Transformers -->
    <int:transformer id="dbFeedTransformer" 
      ref="feedDbTransformerBean" 
      input-channel="filteredFeedChannel"
      method="transformFeed" 
      output-channel="transformedChannel"/>
  </beans>
```

本代码中涵盖的组件将在以下几点详细解释：

+   `@RunWith(SpringJUnit4ClassRunner.class)`：这定义了要在哪个引擎上运行测试——与 Spring Integration 无关。

+   `@Autowired MessageChannel filteredFeedChannel`：这自动注入了来自上下文文件的通道定义——无需显式加载即可使用。

+   `@Autowired QueueChannel transformedChannel`：这与前面一点相似，同时也自动注入了其他通道。

Spring 配置准备所有必需的元素——现在让我们看看测试类做什么：

1.  它创建了一个虚拟的`SyndEntry`。

1.  它根据那个`SyndEntry`创建了一个预期的`SoFeed`。

1.  它构建了一个载荷类型为`SyndEntry`的消息。

1.  它抓取了转换器插座的通道处理句柄并在其中放置了载荷。

    这是测试转换器的地方，调用的是监听通道的实际转换器实例（而不是模拟的）。

1.  转换器进行转换，并将结果放在输出通道上。

1.  测试类抓取了输出通道的处理句柄并读取了消息。

    输出通道上的实际转换消息必须与构造的预期消息匹配。

通过上述步骤，我们能够测试一个实际的转换器，而不必过多担心通道或其他与系统外部有关的 Spring Integration 元素。

# 测试头部

在测试载荷时，测试头部相对容易。我们来编写一个头部丰富器，然后一个测试用例来验证它：

```java
  <int:header-enricher 
    input-channel="filteredFeedChannel" output-channel="transformedChannel">
    <int:header name="testHeaderKey1" value="testHeaderValue1"/>
    <int:header name="testHeaderKey2" value="testHeaderValue2"/>
  </int:header-enricher>
```

任何放入`filteredFeedChannel`的消息都会添加头部。以下代码片段是验证这些头部是否被添加的测试用例：

```java
import static org.junit.Assert.assertThat;
import static org.springframework.integration.test.matcher.HeaderMatcher.hasHeader;
import static org.springframework.integration.test.matcher.HeaderMatcher.hasHeaderKey;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.integration.channel.QueueChannel;
import org.springframework.integration.support.MessageBuilder;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageChannel;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@ContextConfiguration
// default context name is <ClassName>-context.xml
@RunWith(SpringJUnit4ClassRunner.class)
public class TestSoHeaderAddition {
  @Autowired
  MessageChannel filteredFeedChannel;

  @Autowired
  QueueChannel transformedChannel;

  @Test
  public void headerIsAddedToEntity() {
    Message<String> message = MessageBuilder.withPayload("testheader").build();
    filteredFeedChannel.send(message);
    Message<?> outMessage = transformedChannel.receive(0);
    assertThat(outMessage, hasHeaderKey("testHeaderKey1"));
    assertThat(outMessage, hasHeader("testHeaderKey1", "testHeaderValue1"));
  }
}
```

在这里，我们构建了一个测试消息并将其放入通道中。一个头部丰富器被插入了输入通道，它向载荷添加了一个头部。我们通过从输出通道提取消息来验证这一点。

# 处理错误

到目前为止还好，那么处理错误场景呢？如何测试负面用例以及失败的测试用例怎么办？以下代码片段将帮助我们处理这些问题：

```java
  @Test(expected = MessageTransformationException.class)
  public void errorReportedWhenPayloadIsWrong() {
    Message<String> message = 
      MessageBuilder.withPayload("this should fail").build();
    filteredFeedChannel.send(message);
  }
```

输入通道期望的是一个载荷类型为`SyndEntry`的消息，但如果发送了一个载荷类型为`String`的消息——这必须抛出异常。这就是已经测试过的。这可以进一步增强，以监控具有验证用户定义传播消息能力的通道上的某些类型的异常。

# 测试过滤器

我们已经定义了一个过滤器，它过滤掉所有除了 java feed 之外的消息。我们为什么要单独讨论过滤器呢？如果你记得，过滤器总是返回一个布尔值，根据它是否满足条件来指示是否传递消息或丢弃它。为了方便参考，以下是我们定义的过滤器的代码片段：

```java
import java.util.List;
import org.springframework.messaging.Message;
import com.sun.syndication.feed.synd.SyndCategoryImpl;
import com.sun.syndication.feed.synd.SyndEntry;

public class SoFeedFilter {
  public boolean filterFeed(Message<SyndEntry> message){
    SyndEntry entry = message.getPayload();
    List<SyndCategoryImpl>
      categories=entry.getCategories();
    if(categories!=null&&categories.size()>0){
      for (SyndCategoryImpl category: categories) {

        if(category.getName().equalsIgnoreCase("java")){
          return true;
        }

      }
    }
    return false;
  }
}
```

让我们创建一个测试上下文类来测试这个。总是最好有一个单独的上下文类来测试，这样就不会弄乱实际的运行环境。

现在，我们编写测试用例——第一个用例是验证所有类型为`java`的消息都被允许通过：

```java
  @Test
  public void javaMessagePassedThrough() {
    SyndEntry entry =new SyndEntryImpl();
    entry.setTitle("Test");
    SyndContent content=new SyndContentImpl();
    content.setValue("TestValue");
    entry.setDescription(content);
    List<SyndCategoryImpl> catList=new 
      ArrayList<SyndCategoryImpl>();
    SyndCategoryImpl category=new SyndCategoryImpl();
    category.setName("java");
    catList.add(category);
    entry.setCategories(catList);
    entry.setLink("TestLink");
    entry.setAuthor("TestAuthor");

    Message<SyndEntry> message = 
      MessageBuilder.withPayload(entry).build();
    fetchedFeedChannel.send(message);
    Message<?> outMessage = filteredFeedChannel.receive(0);
    assertNotNull("Expected an output message", outMessage);
    assertThat(outMessage, hasPayload(entry));
  }
```

```java
is used to test whether any other message except the category java is dropped:
```

```java
  @Test
  public void nonJavaMessageDropped() {
    SyndEntry entry =new SyndEntryImpl();
    entry.setTitle("Test");
    SyndContent content=new SyndContentImpl();
    content.setValue("TestValue");
    entry.setDescription(content);
    List<SyndCategoryImpl> catList=new 
      ArrayList<SyndCategoryImpl>();
    SyndCategoryImpl category=new SyndCategoryImpl();
    category.setName("nonjava");
    catList.add(category);
    entry.setCategories(catList);
    entry.setLink("TestLink");
    entry.setAuthor("TestAuthor");

    Message<SyndEntry> message = 
      MessageBuilder.withPayload(entry).build();
    fetchedFeedChannel.send(message);
    Message<?> outMessage = filteredFeedChannel.receive(0);
    assertNull("Expected no output message", outMessage);
  }
```

# 分割器测试

让我们讨论一下最后一个测试——这是针对分割器的。我们所定义的分割器如下：

```java
import org.springframework.messaging.Message;

import com.sun.syndication.feed.synd.SyndCategoryImpl;
import com.sun.syndication.feed.synd.SyndEntry;

public class SoFeedSplitter {
  public List<SyndCategoryImpl> splitAndPublish(Message<SyndEntry> message) {
    SyndEntry syndEntry=message.getPayload();
    List<SyndCategoryImpl> categories= syndEntry.getCategories();
    return categories;
  }
}
```

以下代码片段代表我们的测试类：

```java
import static org.junit.Assert.assertNotNull;
import static org.junit.Assert.assertNull;
import static org.junit.Assert.assertThat;
import static org.springframework.integration.test.matcher.HeaderMatcher.hasHeader;
import static org.springframework.integration.test.matcher.HeaderMatcher.hasHeaderKey;
import static org.springframework.integration.test.matcher.PayloadMatcher.hasPayload;

import java.util.ArrayList;
import java.util.List;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.integration.channel.QueueChannel;
import org.springframework.integration.support.MessageBuilder;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageChannel;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import com.cpandey.siexample.pojo.SoFeed;
import com.sun.syndication.feed.synd.SyndCategoryImpl;
import com.sun.syndication.feed.synd.SyndContent;
import com.sun.syndication.feed.synd.SyndContentImpl;
import com.sun.syndication.feed.synd.SyndEntry;
import com.sun.syndication.feed.synd.SyndEntryImpl;

@ContextConfiguration	// default context name is <ClassName>-context.xml
@RunWith(SpringJUnit4ClassRunner.class)
public class TestSplitter {
  //Autowire required channels
  @Autowired
  MessageChannel filteredFeedChannel;

  @Autowired
  QueueChannel splitFeedOutputChannel;

  @Test
  public void javaMessagePassedThrough() {
    //Create MOCK payload
    //Create a SyndEntry Object
    SyndEntry entry =new SyndEntryImpl();
    entry.setTitle("Test");
    //Create a SyndContent to be used with entry
    SyndContent content=new SyndContentImpl();
    content.setValue("TestValue");
    entry.setDescription(content);
    //Create List which is expected on Channel
    List<SyndCategoryImpl> catList=new ArrayList<SyndCategoryImpl>();
    //Create Categories
    SyndCategoryImpl category1=new SyndCategoryImpl();
    category1.setName("java");
    category1.setTaxonomyUri("");
    SyndCategoryImpl category2=new SyndCategoryImpl();
    category2.setName("java");
    category2.setTaxonomyUri("");
    //Add categories
    catList.add(category1);
    catList.add(category2);
    //Complete entry
    entry.setCategories(catList);
    entry.setLink("TestLink");
    entry.setAuthor("TestAuthor");

    //Use Spring Integration util method to build a payload
    Message<SyndEntry> message = MessageBuilder.withPayload(entry).build();
    //Send Message on the channel
    filteredFeedChannel.send(message);
    Message<?> outMessage1 = splitFeedOutputChannel.receive(0);
    //Receive Message on channel
    Message<?> outMessage2 = splitFeedOutputChannel.receive(0);
    //Assert Results
    assertNotNull("Expected an output message", outMessage1);
    assertNotNull("Expected an output message", outMessage2);
    assertThat(outMessage1, hasPayload(category1));
    assertThat(outMessage2, hasPayload(category2));
  }
}
```

这个测试相当容易解释。如预期的那样，根据前面的代码中定义的原始分割器，当在通道上放置一个具有`SyndEntry`的载荷，其中有一个类别列表时，它会提取列表，将其分割，然后一个接一个地将类别放置在输出通道上。

这些例子足以开始进行 Spring Integration 测试。在 Spring Integration 上下文中，TDD 的最佳实践同样适用。实际上，除了 Spring Integration 为测试组件提供支持类之外，Spring Integration 测试并没有什么特别之处。

# 总结

我们讨论了如何测试最广泛使用的 Spring Integration 组件。始终是一个好的实践来*隔离*测试系统——这样集成时间的惊喜可以最大程度地减少。让我们结束关于测试支持的讨论，并转向下一章，我们将讨论如何管理和扩展 Spring Integration 应用程序的方法。
