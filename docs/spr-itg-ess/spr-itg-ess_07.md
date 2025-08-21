# 第七章：与 Spring Batch 的集成

如今，常见的用户会处理网络应用、移动应用和桌面软件。所有这些都是交互式的，这意味着它们需要用户输入并实时做出响应。他们可能甚至不知道其他类型的应用——后台运行、不需要持续用户交互，并且可能持续数小时、数天甚至数周的应用！是的，我在谈论通常用于离线处理如文件类型转换、报告、数据挖掘等任务的批量作业。在早期，机器太慢了，有人必须坐上几小时才能完成一个简单的任务。在批量处理中，你提交任务然后去做其他工作——你只来收集结果！这一革命改变了计算世界，证明了设备和程序员高昂价格的合理性。毫不夸张地说，批量作业展示了计算机的真正力量和实用性。

如果批量作业这么重要，很明显 Spring 会提供很好的支持。Spring Batch 是提供批量处理全面支持的模块。在本章中，我们将探讨 Spring Integration 如何与 Spring Batch 模块集成。与 Spring 的模块化哲学同步，每个模块独立工作，同时提供必要的接口以便于与其他家族成员轻松集成。Spring Integration 可以通过消息与 Spring Batch 模块交互，并提供一个事件驱动机制来触发批量作业。本章将涵盖两个方面：

+   Spring Batch 简介

+   Spring Integration 和 Spring Batch

# Spring Batch

对于普通人来说，批量作业可以被定义为任何可以离线运行的任务。通常，它将是一个手动触发，在预期的完成时间之后可以收集结果。如果一切顺利，那真的很酷，但让我们列出一些挑战：

+   如果用于批量作业的外部系统（比如说托管文件的 FTP 服务器）失败了会怎样？

+   如果出于某种原因运行批量作业的机器需要重新启动，批量作业也会重新开始吗？

+   如果需要一些显式参数（例如，可能不适合自动化的认证详情）该怎么办？

+   未完成任务会再次尝试还是放弃？

+   我们如何处理事务和回滚？

+   我们如何以固定间隔或事件驱动的方式触发和调度作业？

+   如果作业在线程中运行，谁来管理资源同步？

+   我们如何处理失败？批量作业能否触发一些警报或发送通知？

有很多事情需要考虑——想象一下如果每个都要程序员来实现会有多困难！不要担心；Spring Batch 在那里帮助你。有了 Spring Integration 的帮助，甚至最初的触发部分也可以编程——完全不需要人工交互。

首先，Spring Batch 不是一个像 Quartz、Tivoli 那样的调度框架——相反，它利用了这些框架。它是一个非常轻量级的框架，提供了可重用的组件来解决前面提到的多数问题，例如，事务支持、可恢复作业的数据库支持、日志记录、审计等等。让我们从配置步骤开始，然后我们可以逐步过渡到示例。

## 先决条件

在我们可以使用 Spring Batch 模块之前，我们需要添加命名空间支持和 Maven 依赖项：

+   **命名空间支持**：可以通过以下代码添加命名空间支持：

    ```java
    <beans 

      xsi:schemaLocation="http://www.springframework.org/schema/beans
      http://www.springframework.org/schema/beans/spring-beans.xsd
      http://www.springframework.org/schema/batch
      http://www.springframework.org/schema/batch/spring-batch.xsd
      http://www.springframework.org/schema/context
      http://www.springframework.org/schema/context/spring-context.xsd
      http://www.springframework.org/schema/integration
      http://www.springframework.org/schema/integration/spring-integration.xsd">
    ```

+   **Maven 入口**：可以通过以下代码添加 Maven 入口支持：

    ```java
        <dependency>
          <groupId>org.springframework.batch</groupId>
          <artifactId>spring-batch-core</artifactId>
          <version>3.0.1.RELEASE</version>
        </dependency>

        <dependency>
          <groupId>postgresql</groupId>
          <artifactId>postgresql</artifactId>
          <version>9.0-801.jdbc4</version>
        </dependency>

        <dependency>
          <groupId>commons-dbcp</groupId>
          <artifactId>commons-dbcp</artifactId>
          <version>1.4</version>
        </dependency>
    ```

# 定义 Spring Batch 作业

在 Spring Batch 中，工作单元是一个*作业*，它封装了完成批量操作所需的其它所有方面。在我们深入了解如何配置和使用 Spring Batch 组件之前，让我们先熟悉一下 Spring Batch 作业中使用的基本术语。

## Spring Batch 作业语言

让我们先熟悉一下 Spring Batch 的基本领域语言，这将帮助我们理解示例：

+   `Job`：这代表一个批量处理，它有一个一对一的映射。对于每个批量处理，将有一个作业。它可以在 XML 或 Java 配置中定义——我使用了 XML 方法。

+   `Step`：这是作业的逻辑分解——一个作业有一个或多个步骤。它封装了作业的阶段。步骤是运行和控制批量作业的实际细节的逻辑单元。每个作业步骤可以指定其容错性——例如，在错误时跳过一项，停止作业等。

+   `JobInstance`：这是一个作业实例。例如，一个作业必须每天运行一次，每次运行都会由一个`JobInstance`来表示。

+   `JobParameter`：这是完成`JobInstance`所必需的参数。

+   `JobExcecution`：当一个作业的`JobInstance`被触发时，它可能完成或失败。每个`JobInstance`的触发都被包装成`JobExecution`。所以，例如，如果设置了重试，并且由于失败，`JobInstance`被触发三次（由于失败）才完成，那么就有三个`JobExecution`实例。

+   `StepExecution`：与`JobExecution`类似，`StepExecution`是运行一个步骤的一次尝试的实例。如果一个步骤在*n*次重试后完成，将有*n*个`StepExecution`实例。

+   `ExecutionContext`：批量作业的一个重要方面是能够重新启动和重新调度失败作业；为此，需要存储足够的信息，以便可以将其重新触发，类似于操作系统级别的进程上下文。`ExecutionContext`用于解决此用例，它提供存储与上下文相关的属性键/值对的存储。

+   `JobRepository`：这是所有上述单元的持久性包装器。底层数据库提供者可以来自 Spring Batch 支持的各种数据库之一。

+   `JobLauncher`：这是一个用于启动作业的接口。

+   `ItemReader`：此接口用于步骤读取输入。如果输入集已用尽，`ItemReader`应通过返回 null 来指示此情况。

+   `ItemWriter`：这是步骤的输出接口——一次一个批次或数据块。

+   `ItemProcessor`：这是`ItemReader`和`ItemWriter`的中间状态。它提供了将一个项目应用于转换或业务逻辑的机会。

有了前面的介绍，我们可以更好地理解 Spring Batch 示例。那么我们从定义一个批处理作业开始：

```java
<batch:job id="importEmployeeRecords" 
  job-repository="jobRepository" 
  parent="simpleJob">
  <batch:step id="loadEmployeeRecords">
    <batch:tasklet>
      <batch:chunk 
        reader="itemReader" 
        writer="itemWriter" 
        commit-interval="5"/>
    </batch:tasklet>
  </batch:step>
  <!-- Listener for status of JOB -->
  <batch:listeners>
    <batch:listener 
      ref="notificationExecutionsListener"/>
  </batch:listeners>
</batch:job>
```

以下是前面配置中使用的标签的简要描述：

+   `batch:job`：这是启动批处理作业的父标签。`id`用于唯一标识此作业，例如，在`JobLauncher`内引用此作业。

+   `batch:step`：这是此作业的一个步骤。

+   `batch:tasklet`：这是执行步骤实际任务的实现，而步骤则负责维护状态、事件处理等。

+   `batch:chunk`：一个`tasklet`可以是一个简单的服务或一个非常复杂的任务，而一个`chunk`是可以通过`tasklet`进行处理的工作逻辑单位。

+   `batch:listeners`：这些用于传播事件。我们将在本章后面重新访问这个。

读者和写入者是什么？正如名称所示，读者读取数据块，而写入者将其写回。Spring 提供了读取 CSV 文件的标准化读者，但我们可以提供自己的实现。让我们看看这个例子中使用的读者和写入者。

## ItemReader

```java
FlatFileItemReader reader to read data from a flat file:
```

```java
<bean id="itemReader" 
  class="org.springframework.batch.item.file.FlatFileItemReader" 
  scope="step">
  <property name="resource" 
    value="file:///#{jobParameters['input.file.name']}"/>
  <property name="lineMapper">
    <bean class=
      "org.springframework.batch.item.file.mapping.DefaultLineMapper">
      <property name="lineTokenizer">
        <bean class=
          "org.springframework.batch.item.file.transform.DelimitedLineTokenizer">
          <property name="names" 
            value="name,designation,dept,address"/>
        </bean>
      </property>
      <property name="fieldSetMapper">
        <bean class=
          "com.cpandey.siexample.batch.EmployeeFieldSetMapper"/>
      </property>
    </bean>
  </property>
</bean>
```

前面代码片段中使用的组件在以下项目点中解释：

+   `itemReader`：这使用了 Spring 的默认平面文件读取器，其位置在`resource`属性中提到。名称将从传递给作业的`JobParameter`条目中检索。我们将看到在编写启动器时如何传递它。

+   `lineMapper`：这是 Spring 提供的默认实现，用于将 CSV 文件中的行映射到行。

+   `lineTokenizer`：如何解释行中的每个令牌非常重要。属性`names`的值决定了顺序。例如，在前面的示例中，它是`name,designation,dept,address`，这意味着如果样本文件有一个条目如下：

    ```java
    Chandan, SWEngineer, RnD, India
    Pandey, Tester, RnD, India
    ```

    然后，每个数据块将被解释为姓名、职位、部门和地址，分别。

+   `fieldSetMapper`：虽然有一些默认实现，但在大多数情况下，它是一个自定义类，用于定义 CSV 文件中的条目和领域模型之间的映射。以下是使用映射器的示例代码片段：

    ```java
    import org.springframework.batch.item.file.mapping.FieldSetMapper;
    import org.springframework.batch.item.file.transform.FieldSet;
    import org.springframework.validation.BindException;

    public class EmployeeFieldSetMapper implements FieldSetMapper<Employee> {

    @Override
    public Employee mapFieldSet(FieldSet fieldSet) throws BindException {
        Employee employee = new Employee();
        employee.setName(fieldSet.readString("name"));
        employee.setDesignation(fieldSet.readString("designation"));
        employee.setDept(fieldSet.readString("dept"));
        employee.setAddress(fieldSet.readString("address"));
        return employee;
      }
    }
    ```

## ItemWriter

写入器用于写入数据块。写入器几乎总是用户定义的。它可以被定义为在文件、数据库或 JMS 中写入，或到任何端点——这取决于我们的实现。在章节的最后，我们将讨论如何使用它甚至触发 Spring Integration 环境中的事件。让我们首先看看一个简单的写入器配置：

```java
<bean id="itemWriter" 
class="com.cpandey.siexample.batch.EmployeeRecordWriter"/>
```

以下代码片段是写入器类的实现：

```java
import java.util.List;
import org.springframework.batch.item.ItemWriter;
public class EmployeeRecordWriter implements ItemWriter<Employee> {
  @Override
  public void write(List<? extends Employee> employees) throws
  Exception {
    if(employees!=null){
      for (Employee employee : employees) { 
        System.out.println(employee.toString());
      }
    }
  }
}
```

为了简单起见，我打印了记录，但如前所述，它可以在数据库中填充，或者可以用来在这个类中做我们想做的事情。

好吧，到目前为止，我们已经定义了作业、读取器和写入器；那么是什么阻止我们启动它呢？我们如何启动这个批处理作业？Spring 提供了`Joblauncher`接口，可以用来启动作业。`Joblauncher`需要一个`JobRepository`接口的实现来存储作业的上下文，以便在失败时可以恢复和重新启动。`JobRepository`可以配置为利用 Spring 可以使用的任何数据库，例如，内存、MySql、PostGres 等。让我们如下定义`jobLauncher`：

```java
<bean id="jobLauncher" 
  class="org.springframework.batch.core.launch.support.SimpleJobLauncher">
  <property name="jobRepository" ref="jobRepository"/>
</bean>
```

由于`JobLauncher`不能在没有`JobRepository`的情况下使用，让我们配置`JobRepository`：

```java
<bean id="jobRepository" 
  class="org.springframework.batch.core.repository.support.MapJobRepositoryFactoryBean">
  <property name="transactionManager" ref="transactionManager"/>
</bean>
```

```java
the configuration of a data source (this is an Apache DBCP implementation):
```

```java
import org.apache.commons.dbcp.BasicDataSource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
@Configuration
public class BatchJdbcConfiguration {
  @Value("${db.driverClassName}")
  private String driverClassName;
  @Value("${db.url}")
  private String url;
  @Value("${db.username}")
  private String username;
  @Value("${db.password}")
  private String password;
  @Bean(destroyMethod = "close")

  public BasicDataSource dataSource() {
    BasicDataSource dataSource = new BasicDataSource();
    dataSource.setDriverClassName(driverClassName);
    dataSource.setUrl(url);
    dataSource.setUsername(username);
    dataSource.setPassword(password);
    return dataSource;
  }
}
```

前面代码中显示的属性可以在一个`properties`文件中配置，比如说`batch.properties`。我们可以将属性提供在类路径中，并使用`property-placeholder`标签来注入属性，如下所示：

```java
<context:property-placeholder 
  location="/META-INF/spring/integration/batch.properties"/> 
  db.password=root 
  db.username=postgres 
  db.databaseName=postgres 
  db.driverClassName=org.postgresql.Driver 
  db.serverName=localhost:5432 
  db.url=jdbc:postgresql://${db.serverName}/${db.databaseName}
```

一旦有了数据库，我们就需要事务！让我们配置事务管理器：

```java
<bean id="transactionManager" 
  class="org.springframework.batch.support.transaction. 
  ResourcelessTransactionManager" />
```

谢天谢地，不再有配置了！顺便说一下，这些不是针对任何批处理作业的；任何在现有应用程序中配置的数据源和事务管理器都可以使用。有了所有的配置，我们准备启动批处理作业。让我们看看以下示例代码：

```java
import org.springframework.batch.core.Job;
import org.springframework.batch.core.JobExecution;
import org.springframework.batch.core.JobParametersBuilder;
import org.springframework.batch.core.JobParametersInvalidException;
import org.springframework.batch.core.launch.JobLauncher;
import org.springframework.batch.core.repository.JobExecutionAlreadyRunningException;
import org.springframework.batch.core.repository.JobInstanceAlreadyCompleteException;
import org.springframework.batch.core.repository.JobRestartException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class BatchJobLauncher {
  public static void main(String[] args) throws JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException, JobParametersInvalidException {
    ApplicationContext context = new ClassPathXmlApplicationContext("/META-INF/spring/integration/spring-integration-batch.xml");
    Job job = context.getBean("importEmployeeRecords", Job.class);
    JobLauncher jobLauncher= context.getBean("jobLauncher", JobLauncher.class);
    JobParametersBuilder jobParametersBuilder = new JobParametersBuilder();
    jobParametersBuilder.addString("input.file.name", "C:/workspace_sts/siexample/src/main/resources/META-INF/spring/integration/employee.input");
    JobExecution execution =jobLauncher.run(job, jobParametersBuilder.toJobParameters());
  }
}
```

让我们理解一下代码：

+   **加载文件**：我们首先加载配置文件。

+   **提取引用**：下一步是使用其唯一 ID 检索定义工作的引用。

+   **添加参数**：作业需要一个参数，因此我们使用`JobParameterBuilder`类定义`JobParameter`。传递给键值的文件名是`input.file.name`，这在作业定义中配置。

+   **启动作业**：最后，使用 Spring 的`JobLauncher`类来启动作业。

嗯！现在我们有一个小而简单的批处理程序正在运行。让我们看看 Spring Integration 如何利用其力量并进一步增强使用。

# Spring Batch 和 Spring Integration

通常，批处理应用程序可以通过命令行界面或程序化方式触发，例如，从一个 web 容器中。让我们引入 Spring Integration 并看看可能性：

+   它可以由事件触发，例如，文件适配器监听文件触发 Spring Integration 在文件到达时。

+   执行可以在流程中链接——触发作业，传递结果，调用错误路径等。

+   消息队列并不适合大量数据。因此，对于大文件，Spring Integration 可以充当触发器，同时将实际任务委托给 Spring Batch。它可以提供一种分块文件并将其分布到 Spring Batch 作业中的策略。

+   Spring Integration 不仅可以触发批处理作业，还可以收集结果并在系统中传播。例如，由 Spring Integration 触发的批处理过程可能在一天后结束，之后`ItemWriter`可以将一个条目写入 JMS，Spring Integration 适配器正在监听该 JMS。即使没有任何对启动作业的意识或锁定，队列中的消息也将由 Spring Integration 处理。

## 启动作业

够了理论！让我们写一些代码。这次，我们将在某些事件上触发批处理作业，而不是手动触发。我们正在处理一个文件，如果我们处理一个文件适配器会怎样？让我们写一个文件适配器，它将监听目录中的文件，并在文件可用时触发一个批处理作业：

```java
<int-file:inbound-channel-adapter id="fileAdapter" 
  directory="C:\Chandan\Projects\inputfolderforsi" 
  channel="filesOutputChannel" 
  prevent-duplicates="true" filename-pattern="*.txt"> 
  <int:poller fixed-rate="1000" />
</int-file:inbound-channel-adapter>
```

不需要定义文件适配器标签，因为它们在前一章中已经处理了。

前面的配置将监听配置目录中的文件。文件将被放入`fileOutPutChannel`作为`Message<File>`，我们需要将其转换为`JobLauncher`可以理解的形式。我们将使用`transformer`组件：

```java
<int:transformer 
  input-channel="filesOutputChannel" 
  output-channel="batchRequest">
  <bean class="com.cpandey.siexample.batch.FileMessageToJobRequest">
    <property name="job" ref="importEmployeeRecords"/>
    <property name="fileParameterName" value="input.file.name"/>
  </bean>
</int:transformer>
```

我们将不得不编写逻辑将`Message<File>`转换为`JobLaunchRequest`。以下代码是一个非常简单的转换器，它从`Message`的负载（即`File`）中提取文件路径，然后将检索到的路径作为`JobParameter`添加。这个作业参数然后用于使用 Spring 的`JobLauncher`启动作业，如下面的代码片段所示：

```java
import java.io.File;

import org.springframework.batch.core.Job;
import org.springframework.batch.core.JobParametersBuilder;
import org.springframework.batch.integration.launch.JobLaunchRequest;
import org.springframework.integration.annotation.Transformer;
import org.springframework.messaging.Message;

public class FileMessageToJobRequest {
  private Job job;
  private String fileParameterName;

  public void setFileParameterName(String fileParameterName) {
    this.fileParameterName = fileParameterName;
  }

  public void setJob(Job job) {
    this.job = job;
  }

  @Transformer
  public JobLaunchRequest toRequest(Message<File> message) {
  JobParametersBuilder jobParametersBuilder = new JobParametersBuilder();

  jobParametersBuilder.addString(fileParameterName,message.getPayload().getAbsolutePath());
  return new JobLaunchRequest(job,jobParametersBuilder.toJobParameters());
  }
}
```

有了这段代码，每当有新文件到达目录时，就会使用 Spring Integration 触发一个批处理作业。而且，文件适配器只是一个例子，任何适配器或网关——比如邮件、JMS、FTP 等——都可以插入以触发批处理。

## 跟踪批处理作业的状态

大多数时候，我们希望能够得到进行中的任务的反馈——我们怎样才能做到这一点呢？Spring Integration 是一个基于事件的事件框架，所以毫不奇怪，我们可以为批处理作业配置监听器。如果你参考开头的批处理作业定义，它有一个监听器定义：

```java
  <batch:listeners>
    <batch:listener ref="simpleListener"/>
  </batch:listeners>
```

这段代码可以有一个 Spring Integration 网关作为监听器，它监听通知并将批处理作业（类型为`JobExecution`）的状态放在定义的信道上：

```java
<int:gateway id=" simpleListener"
  service-interface="org.springframework.batch.core.JobExecutionListener" default-request-channel="jobExecutionsStatus"/>
```

状态将在我们完成处理的信道上可用。我们插入一个简单的服务激活器来打印状态：

```java
<int:service-activator
  ref="batchStatusServiceActivator"
  method="printStatus"
  input-channel="jobExecutionsStatus"/>

import org.springframework.batch.core.JobExecution;
import org.springframework.integration.annotation.MessageEndpoint;
import org.springframework.messaging.Message;

@MessageEndpoint
public class BatchStatusServiceActivator {
  public void printStatus(Message<JobExecution> status ) {
    if(status!=null){
      System.out.println("Status :: "+status.getPayload().toString());
    }
  }
}
```

## 反之亦然

Spring Integration 可以启动批处理作业，而 Spring Batch 可以与 Spring Integration 交互并触发组件。我们如何做到这一点呢？Spring Integration 的事件驱动组件是一个不错的选择。让我们来看一个简单的例子：

+   在 Spring Integration 应用程序中有一个入站 JMS 适配器，它监听队列上的消息，并基于此触发某些操作。

+   我们如何从 Spring Batch 中调用这个适配器呢？我们可以在 Spring Batch 中定义一个自定义的`ItemWriter`类，该类将其输出写入 JMS 队列，而 Spring Integration 组件正在监听该队列。

+   一旦`ItemWriter`将数据写入 JMS 队列，入站适配器就会将其捡起并传递给下一阶段进行进一步处理。

前面提到的用例只是其中之一；我们可以整合这两个框架的事件机制，实现所需的应用间通信。

# 总结

这就完成了我们关于 Spring Integration 和 Spring Batch 如何进行互联互通的讨论。我们介绍了 Spring Batch 的基础知识，如何被 Spring Integration 利用来委托处理大量负载，如何跟踪状态，以及随后 Spring Batch 如何触发事件并在 Spring Integration 应用程序中开始处理！

在下一章中，我们将讨论最重要的方面之一——测试。保持精力充沛！
