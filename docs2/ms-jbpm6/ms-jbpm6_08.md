# 第八章：将 jBPM 与企业架构集成

我们已经建立了企业基础设施，现在我们希望将流程管理分离并集中到单个组件中，当然，我们的选择是 jBPM。所以，那个价值百万的问题将是“我们如何将 jBPM 集成到现有的企业应用程序中？”

这个问题的答案因需求和企业应用程序的构建方式而异。架构描述了应用程序是如何构建的，从更广泛的角度来看，使用一系列架构模式（单独或组合使用）作为指导原则来建模架构。本章重点介绍 jBPM 为与遵循这些架构模式的应用程序集成提供的功能。

本章首先讨论企业应用程序集成的背景，然后详细讨论以下内容：

+   将 jBPM 集成到基于 JEE 的应用程序中

+   将 jBPM 集成到面向服务的架构中

+   将 jBPM 集成到事件驱动架构中

# 设置上下文

软件组件与现有软件架构的系统集成表示我们应该提供两个窗口（接口），如下所示：

+   为了访问新组件提供的服务。在 jBPM 的情况下，它由 jBPM 提供的各种服务表示，例如，用于管理业务流程生命周期的流程运行时提供。JBPM 将这些服务作为其核心引擎的 API 公开。

+   为了使 jBPM 能够访问应用程序架构中其他组件提供的服务。JBPM 为与外部组件集成提供的扩展点是工作项处理器。我们可以创建处理器并编写访问外部组件的逻辑。

下图展示了此背景：

![设置上下文](img/9578OS_08_01.jpg)

# jBPM 提供的服务

正如我们在上一节中讨论的，与 jBPM 系统集成的一个关键部分是访问 jBPM 特性的能力。JBPM 提供了一个应用程序编程接口来访问这些特性。此 API 可以在同一个 JVM 中直接调用，如果需要从系统边界之外访问，则必须将其包装并提供为远程可访问的服务。为此，我们有各种选择，从 **企业 JavaBeans** （**EJB**）远程接口到基于 REST 的 Web 服务。这些将在本章后续部分中详细介绍。

以下是由 jBPM 提供的服务：

+   **定义服务**：这有助于定义流程并分析其内容

+   **部署服务**：这有助于部署业务流程及其相关工件

+   **流程服务**：这有助于从流程定义中启动流程实例，管理实例的生命周期，并使用信号与它们交互

+   **用户任务服务**：这有助于管理人工任务的生命周期

+   **运行时数据服务**：这有助于在 jBPM 运行时获取有关流程、流程实例、任务和审计跟踪的数据细节

每个服务在以下部分中详细说明，包括（重要）操作：

+   `org.jbpm.services.api.DefinitionService`：此服务有助于从 BPMN 文本定义流程并提供分析业务流程定义的操作：

    | 操作 | 操作签名 | 描述 |
    | --- | --- | --- |
    | `buildProcessDefinition` |

    ```java
    ProcessDefinition buildProcessDefinition(String deploymentId, String bpmn2Content, ClassLoader classLoader, boolean cache) throws IllegalArgumentException;
    ```

    | 从给定的流程定义内容（`bpmn2Content`）构建流程定义 |
    | --- |
    | `getReusableSubProcesses` |

    ```java
    Collection<String> getReusableSubProcesses(String deploymentId, String processId);
    ```

    | 获取流程定义内部可重用子流程的流程标识符 |
    | --- |
    | `getProcess Variables` |

    ```java
    Map<String, String> getProcessVariables(String deploymentId, String processId);
    ```

    | 获取业务流程中所有流程变量的名称和类型 |
    | --- |
    | `getServiceTasks` |

    ```java
    Map<String, String> getServiceTasks(String deploymentId, String processId);
    ```

    | 获取与业务流程定义中关联的所有服务任务标识符 |
    | --- |
    | `getTasks Definitions` |

    ```java
    Collection<UserTaskDefinition> getTasksDefinitions(String deploymentId, String processId);
    ```

    | 获取业务流程中定义的所有任务 |
    | --- |

+   `org.jbpm.services.api.DeploymentService`：此服务有助于部署和管理应用程序部署单元：

    | 操作 | 操作签名 |
    | --- | --- |
    | `deploy` |

    ```java
    void deploy(DeploymentUnit unit);
    ```

    |

    | `undeploy` |
    | --- |

    ```java
    void undeploy(DeploymentUnit unit);
    ```

    |

    | `activate` |
    | --- |

    ```java
    void activate(String deploymentId)
    ```

    |

    | `deactivate` |
    | --- |

    ```java
    void deactivate(String deploymentId);
    ```

    |

    | `IsDeployed` |
    | --- |

    ```java
    boolean isDeployed(String deploymentUnitId)
    ```

    |

+   `org.jbpm.services.api.ProcessService`：此流程服务用于管理生命周期并与已启动的流程实例交互：

    | 操作 | 操作签名 |
    | --- | --- |
    | `startProcess` |

    ```java
    Long startProcess(String deploymentId, String processId);
    ```

    |

    | `startProcess` |
    | --- |

    ```java
    Long startProcess(String deploymentId, String processId, Map<String, Object> params);
    ```

    |

    | `abortProcessInstance` |
    | --- |

    ```java
    void abortProcessInstance(Long processInstanceId);
    ```

    |

    | `abortProcessInstances` |
    | --- |

    ```java
    void abortProcessInstances(List<Long> processInstanceIds);
    ```

    |

    | `signalProcessInstance` |
    | --- |

    ```java
    void signalProcessInstance(Long processInstanceId, String signalName, Object event);
    ```

    |

    | `signalProcessInstances` |
    | --- |

    ```java
    void signalProcessInstances(List<Long> processInstanceIds, String signalName, Object event);
    ```

    |

    | `completeWorkItem` |
    | --- |

    ```java
    void completeWorkItem(Long id, Map<String, Object> results);
    ```

    |

    | `abortWorkItem` |
    | --- |

    ```java
    abortWorkItem(Long id);
    ```

    |

+   `org.jbpm.services.api.UserTaskService`：此服务有助于执行用户任务的生命周期管理操作：

    | 操作 | 操作签名 |
    | --- | --- |
    | `activate` |

    ```java
    void activate(Long taskId, String userId)
    ```

    |

    | `claim` |
    | --- |

    ```java
    void claim(Long taskId, String userId)
    ```

    |

    | `Complete` |
    | --- |

    ```java
    void complete(Long taskId, String userId, Map<String, Object> params)
    ```

    |

    | `Delegate` |
    | --- |

    ```java
    void delegate(Long taskId, String userId, String targetUserId)
    ```

    |

    | `exit` |
    | --- |

    ```java
    void exit(Long taskId, String userId)
    ```

    |

    | `fail` |
    | --- |

    ```java
    void fail(Long taskId, String userId, Map<String, Object> faultData)
    ```

    |

    | `Forward` |
    | --- |

    ```java
    void forward(Long taskId, String userId, String targetEntityId)
    ```

    |

    | `release` |
    | --- |

    ```java
    void release(Long taskId, String userId);
    ```

    |

    | `resume` |
    | --- |

    ```java
    void resume(Long taskId, String userId);
    ```

    |

    | `skip` |
    | --- |

    ```java
    void skip(Long taskId, String userId);
    ```

    |

    | `start` |
    | --- |

    ```java
    void start(Long taskId, String userId);
    ```

    |

    | `stop` |
    | --- |

    ```java
    void stop(Long taskId, String userId);
    ```

    |

+   `org.jbpm.services.api.RuntimeDataService`：此 API 用于检索有关 jBPM 运行时的信息，包括流程实例、任务和审计日志的数据：

    | 操作 | 操作签名 |
    | --- | --- |
    | `getProcesses` |

    ```java
    Collection<ProcessDefinition> getProcesses(QueryContext queryContext);
    ```

    |

    | `getProcessInstances` |
    | --- |

    ```java
    Collection<ProcessInstanceDesc> getProcessInstances(QueryContext queryContext);
    ```

    |

    | `getProcessInstance FullHistory` |
    | --- |

    ```java
    Collection<NodeInstanceDesc> getProcessInstanceFullHistory(long processInstanceId, QueryContext queryContext);
    ```

    |

    | `getVariableHistory` |
    | --- |

    ```java
    Collection<VariableDesc> getVariableHistory(long processInstanceId, String variableId, QueryContext queryContext);
    ```

    |

    | `getTaskEvents` |
    | --- |

    ```java
    List<TaskEvent> getTaskEvents(long taskId, QueryFilter filter);
    ```

    |

    | `getTasksOwned` |
    | --- |

    ```java
    List<TaskSummary> getTasksOwned(String userId, QueryFilter filter);
    ```

    |

# 创建自定义工作项处理器

为了让 jBPM 访问应用程序中其他组件的服务，我们可以使用 jBPM 提供的工作项处理器扩展点。工作项处理器用于指定特定领域的服务到 BPMN 活动。jBPM 中预建了几个内置的通用工作项处理器。

创建工作项处理器时，我们必须实现 `org.kie.runtime.instance.WorkItemHandler` 接口。此接口包含两个需要实现的方法：

+   `WorkItemManager.completeWorkItem(long workItemId, Map<String, Object> results)`

+   `WorkItemManager.abortWorkItem(long workItemId)`

必须使用工作项管理器将自定义工作项注册到引擎中。例如，为了注册一个客户任务，我们可以使用以下方法：

```java
ksession.getWorkItemManager().registerWorkItemHandler("Notification",
new NotificationWorkItemHandler());
```

总结来说，我们已经讨论了 jBPM 中可用于将其与通用软件架构集成的配置。在接下来的章节中，我们将讨论如何将 jBPM 集成到广泛使用的企业架构中。

# 与 JEE 集成

Java 企业版为企业应用程序的开发和部署提供了一个 API 和运行时环境。此外，EJB 定义了一组轻量级 API，可用于构建应用程序并利用事务、远程过程调用、并发控制和访问控制等能力。

EJB 可以通过两种模式访问：

+   **远程接口**：这是想要访问 EJB 的组件没有与 jBPM 打包在一起的地方

+   **本地接口**：这是想要访问 EJB 的组件与 jBPM 服务打包在一起的地方

JBPM 为 JEE 集成提供了开箱即用的支持。它提供了访问上述列出的服务的 EJB 远程和本地接口。

## EJB 远程接口

EJB 远程接口如下：

| 服务名称 | EJB 远程服务类 |
| --- | --- |
| 定义服务 |

```java
org.jbpm.services.ejb.api.DefinitionServiceEJBRemote

```

|

| 部署服务 |
| --- |

```java
org.jbpm.services.ejb.api.DeploymentServiceEJBRemote

```

|

| 流程服务 |
| --- |

```java
org.jbpm.services.ejb.api.ProcessServiceEJBRemote

```

|

| 运行时数据服务 |
| --- |

```java
org.jbpm.services.ejb.api.RuntimeDataServiceEJBRemote

```

|

| 用户任务服务 |
| --- |

```java
org.jbpm.services.ejb.api.UserTaskServiceEJBRemote

```

|

这些远程服务可以从其他 Java 应用程序中访问。首先，我们需要访问`ejb`远程接口。

例如（特定于`jboss`应用程序服务器），以下代码显示了`ProcessService`的查找：

```java
final Hashtable<String, String> jndiProperties = new Hashtable<String, String>();
  //Set the JNDI properties
  jndiProperties.put(Context.URL_PKG_PREFIXES, "org.jboss.ejb.client.naming");

  final Context context = new InitialContext(jndiProperties);
  //Set the bean name
  String beanName = "ProcessServiceEJBImpl!org.jbpm.services.ejb.api.ProcessServiceEJB Remote";

  String jndi = "ejb:/" + application + "/" + mappedName;
  ProcessService bean = (ProcessService) context.lookup(jndi);
```

查找服务后，可以无缝地访问该服务。

## EJB 本地接口

EJB 本地接口可以通过两种方式访问。一种是通过使用`javax.ejb.EJB`注解并指定企业 Bean 的本地业务接口名称：

例如：

```java
@EJB
ProcessService processservice;
```

容器将为 API 注入 EJB 访问。

访问本地 EJB 服务的另一种语法方式是使用 JNDI 查找和`javax.naming.InitialContext`接口的`lookup`方法：

```java
ProcessService processservice = (Processservice)
  InitialContext.lookup("java:jbpm/Processservice");
```

# 在 SOA 和 EDA 环境中集成

本节的第一部分介绍了如何将作为客户端的 jBPM 集成到外部服务中；第五章，*BPMN 构造*，和第六章，*核心架构*，介绍了专门设计用于从流程定义中调用外部 Web 服务的 jBPM 元素：服务任务和 WS 或 REST 工作项处理器。后者是 jBPM 的现成、可配置组件，但请注意，jBPM 为用户提供所有工具来开发自定义处理器，以便与通用外部服务进行交互（参见第七章，*自定义和扩展 jBPM*）。本节第二部分将探讨如何使用 REST、SOAP 和 JMS 将 jBPM API 作为服务器进行集成。我们将为您提供两个示例项目（`jbpm-remote-client`和`jbpm-remote-server`），以便将这些 jBPM 功能付诸实践。

我们将展示如何连接到 REST 和 SOAP 服务。

## 集成 REST 服务

在开始对集成 REST 服务的 jBPM 应用程序进行注释的逐步游览之前，让我们回顾一下 jBPM 在 REST 集成方面提供的基本支持。jBPM REST 工作项处理器（类`org.jbpm.process.workitem.rest.RESTWorkItemHandler`）旨在与 REST 服务（既安全又非安全）进行交互；它支持以下参数：

+   `Url`：目标资源端点

+   `Method`：HTTP 方法（默认为`GET`）

+   `ContentType`：发送数据时的数据类型（与`POST`和`PUT`一起使用时必需）

+   `Content`：要发送的数据（与`POST`和`PUT`一起使用时必需）

+   `ConnectTimeout`：连接超时（默认为 60 秒）

+   `ReadTimeout`：读取超时（默认为 60 秒）

+   `Username`：认证用户名

+   `Password`：认证密码

处理器返回一个输出结果，该结果定义了以下属性：

+   `Result`：REST 服务文本响应体

+   `Status`：整数 HTTP 响应代码

+   `StatusMsg`：操作结果的字符串描述

我们的示例应用程序设置了一个 REST 服务器并启动了一个具有 REST 服务任务节点的流程：REST 节点执行一个 HTTP POST 操作，将`Order`实例（作为 XML 字符串）传递给 REST 服务器；服务器修改订单的备注并返回订单。

### REST 服务

我们的 REST 服务器是在测试类（`RestTest.initializeRestServer`方法）内部启动的，使用的是 JAX-RS Apache CXF 实现（CXF 版本为 2.7.14，请检查项目`pom.xml`文件以获取依赖项）；初始化代码设置了一个 JAXB 提供程序，以便支持 bean 的数据绑定。

### 注意

请查阅 Apache CXF 的 JAX-RS 文档，网址为[`cxf.apache.org/docs/jax-rs.html`](http://cxf.apache.org/docs/jax-rs.html)。

服务器是围绕一个 REST 资源（`RestResource`类）设置的，该资源通过 JAX-RS `jax.ws.rs`包注解定义了可用的操作。

### 客户端 – REST 处理器配置

+   示例测试类方法 `RestTest.testRestProcess` 启动一个流程实例（参见 `rest.bpmn2` 流程定义）；流程中配置了一个 REST 任务节点，以下为必需的参数：

    +   `Url`: `http://localhost:9998/pizzarestservice/order`

    +   `ContentType`: `application/xml`

    +   `Content`: `<order><note>my note</note></order>`

    +   `Method`: `POST`

节点处理器执行对 `RestResource` 类的 `postOrder(Order order)` 方法的 REST 调用；该方法注解为 `@Path("/order")`，XML 实体序列化由 JAXB 负责处理，正如我们之前所说的。REST 任务输出变量映射回流程实例，并由脚本任务打印出来。

使用 jUnit 测试类 (`TestRest`)，你可以在流程定义之外练习 REST 处理器和 REST 服务（`testPOSTOperation` 方法）。

如果默认的 jBPM REST 处理器无法满足你的要求（因为序列化约束、框架锁定等问题），重要的是指出，开发者可以提供一个全新的处理器实现：遵循 第七章，*定制和扩展 jBPM* 指南描述的工作项处理器开发过程。现在让我们看看如何从流程定义中设置和调用 SOAP Web 服务。

## SOAP WebService

jBPM 随带一个专门的 `ServiceTaskHandler`（参见 第五章，*BPMN 构造*），它基于 WSDL 特征 Web 服务交互。服务任务被标记为具有 `WebService` 类型的实现（任务还支持通过 `Reflection` 类执行纯 Java 实现执行）。请查看 第五章，*BPMN 构造* 中 *服务任务* 部分，以获取更多详细信息和工作示例描述。我们的 jUnit 类 (`WsTest`) 设置一个 Web 服务（`startWebService` 方法），然后启动一个包含两个服务任务节点的流程：一个调用 Web 服务 `addSmallOrder` 操作，另一个调用 `addLargeOrder` 操作：这两个操作都接受一个 `Order` 实例作为输入，并返回一个布尔结果，该结果由脚本任务打印出来。服务任务位于不同的流程分支上，由评估提交订单的总金额的排他网关执行。

### JAX-WS 服务

`TestWebService` 服务是一个注解的 JAX-WS 服务；它从 `WsTest.startWebService` 方法类启动，其端点设置为 `http://127.0.0.1:9931/testwebservice/order`（你可以在单元测试类中轻松配置此设置）。链接 `http://127.0.0.1:9931/testwebservice/order?WSDL` 返回服务 WSDL 接口。该服务公开了上述两个方法：`addSmallOrder` 和 `addLargeOrder`。让我们看看如何从我们的流程定义中调用我们的 Web 服务操作。

#### 客户端 – 流程和服务任务处理器配置

为了调用 Web 服务操作，我们必须执行以下步骤，通过编辑流程定义及其服务任务节点元素：

#### 流程定义

需要注意的是，我们需要导入服务 WSDL 定义。在流程定义导入部分，添加服务 WSDL 位置和命名空间。WSDL 传递给 Apache CXF `JaxWsDynamicClientFactory`，它在创建动态 Web 服务客户端时对其进行解析。

![流程定义](img/9578OS_08_03.jpg)

#### 服务任务处理器

服务任务处理器通过适当地设置其参数自动调用 Web 服务；这加快了集成过程，但在针对具有复杂类型的服务接口进行开发时可能不足，因为我们已经指出，处理器利用了 Apache CXF 动态客户端模式。在这种情况下，强烈建议您开发一个自定义处理器，以集成您选择的 Web 服务框架。我们按以下方式设置处理器参数：

+   **实现**：`WSDL`

+   **serviceInterface**：`TestWebService`

+   **serviceOperation**：`addSmallOrder (addLargeOrder)`

`mode` 参数值保留为 `SYNC`（默认），这表示阻塞操作；当设置为 `ASYNC` 模式时，处理器被迫在线程上执行 Web 服务调用，并将控制权交还给流程引擎，工作项在远程调用返回后立即完成。

#### WebServiceWorkItemHandler 类

jBPM 通过 `WebServiceWorkItemHandler` 类提供了一个以 Web 服务为导向的服务任务处理器替代方案。此处理器在参数数组处理、Web 服务端点设置（它接受 `Endpoint` 参数）和快捷 WSDL 位置加载（使用 `Url` 和 `Namespace` 参数而不是在流程定义级别定义 WSDL URL）方面优于服务任务处理器。`serviceInterface` 和 `serviceOperation` 参数分别重命名为 `Interface` 和 `Operation`。

# jBPM 作为远程服务

jBPM 平台提供了一系列现成的远程 API，旨在为开发者在设计需要即插即用 jBPM 集成的解决方案时提供更高的灵活性。此远程服务层为提供灵活、开放的架构开辟了许多可能性，以满足并快速响应不断变化的应用程序需求，例如：

+   许多外部应用程序系统可能需要偶尔连接到 jBPM 运行时，以检查某些任务或检索某些流程信息

+   jBPM 操作管理器可能仅通过提交通过 HTTP 的命令批处理来执行管理任务

jBPM 随带以下远程服务接口：

+   **REST API**。

+   **JMS API**。

+   **Java 远程 API**：此 API 为开发者提供 `KieSession`、`TaskService` 和 `AuditService` 核心引擎服务的本地存根。这些 API 方法的服务存根是针对较低级别的 REST 或 JMS API 调用的包装。

+   **SOAP API**。

所有这些服务都由 jBPM KIE 工作台公开，因此它们仅在 jbpm-console 网络应用程序在容器中部署时才可用。

### 备注

远程服务项目的源代码托管在 [`github.com/droolsjbpm/droolsjbpm-integration/tree/master/kie-remote`](https://github.com/droolsjbpm/droolsjbpm-integration/tree/master/kie-remote)。

远程服务客户端所需的 Maven 依赖项如下：

```java
<dependency>
  <groupId>org.kie.remote</groupId>
  <artifactId>kie-remote-client</artifactId>
  <version>6.2.0</version>
</dependency>
```

现在我们来回顾主要的远程服务功能以及如何访问它们。

## REST API

此 API 在以下领域提供功能：

+   **运行时**：（`/runtime/` 路径）为用户提供流程实例创建、流程实例查询和工作项操作

+   **历史记录**：（`/history/` 路径）提供审计数据

+   **任务**：（`/task/` 路径）提供任务操作和任务查询方法

+   **部署**：（`/deployments/` 和 `/deployment/` 路径）提供部署管理操作

如需更多详细信息，请参阅 jBPM 用户手册参考（*第十七章*，*jBPM 流程定义语言 (JPDL)*）。

### 认证

在调用时，REST 服务操作会检查当前 HTTP 会话的基本认证用户 ID。例如，假设您正在执行以下命令行代码以在未经授权的会话中执行 REST 操作：

```java
curl -v http://localhost:8080/jbpm-console/rest/deployment/com.packt.masterjbpm6:pizzadelivery:1.0

```

您将获得以下 HTTP `401 未授权` 错误（输出已编辑以清晰；可能有所不同）：

```java
< HTTP/1.1 401 Unauthorized
< WWW-Authenticate: Basic realm="KIE Workbench Realm"
<html><head><title>Error</title></head><body>Unauthorized</body></html>
```

### 备注

Kie 工作台默认的安全机制利用 JAAS；对于 jBoss WildFly 和 EAP，默认配置存储在应用程序服务器的 XML 配置文件中（standalone 等）。有关用户和角色配置，请参阅 第四章，*操作管理*。

否则，设置用户 ID 和密码（工作台领域）如下：

```java
http://admin:admin@localhost:8080/jbpm-console/rest/deployment/com.packt.masterjbpm6:pizzadelivery:1.0
```

这将返回以下响应：

```java
<deployment-unit>
  <groupId>com.packt.masterjbpm6</groupId>
  <artifactId>pizzadelivery</artifactId>
  <version>1.0</version>
  <kbaseName/>
  <ksessionName/>
  <strategy>SINGLETON</strategy>
  <status>DEPLOYED</status>
</deployment-unit>
```

### 备注

要获取完整的 jBPM REST 参考，请参阅 jBPM 官方文档（*第十七章*，*远程 API*）。

## 远程 Java API

远程 Java API 是一个高级 API，它使用 REST 或 JMS 与远程引擎服务交互，以便为用户提供熟悉的服务 API 类（`TaskService`、`KieSession` 等）。

### 依赖项

API 依赖于 jBoss RESTEasy REST 实现和 HornetQ JMS 客户端库。与远程 API 交互所需的 Maven 依赖项是我们之前提到的 `kie-remote-client` 模块和额外的 `kie-remote-common` 艺术品。请确保不要有对 Apache CXF 框架的依赖，这可能会与 jBoss RESTEasy 框架引起问题。

### REST 客户端

初始化是通过从`RemoteRuntimeEngineFactory`获得的**流畅**API 完成的：

```java
// the deploymentId identifies the KIE module
public static String deploymentId = "com.packt.masterjbpm6:pizzadelivery:1.0";
RemoteRestRuntimeEngineBuilder restEngineBuilder = RemoteRuntimeEngineFactory.newRestBuilder()
.addDeploymentId(deploymentId)
.addUrl(instanceurl).addUserName(user)
.addPassword(password);
RemoteRestRuntimeEngineFactory engineFactory = restEngineBuilder
.buildFactory();
// get the engine
RemoteRuntimeEngine engine = engineFactory.newRuntimeEngine();
// and the services
TaskService taskService = engine.getTaskService();
KieSession ksession = engine.getKieSession();
ProcessInstance processInstance = ksession.startProcess(processID);
```

### 注意

请参阅`jbpm-remote-server` Maven 项目及其`RestTest` jUnit 类以获取完整的工作示例。

### jBPM JMS 服务客户端

当使用 JMS 远程 API 客户端时，我们需要添加一系列库依赖项，特别是 HornetQ 和 jBoss 远程客户端。我们将看到如何配置和运行远程客户端应用程序，该应用程序创建一个 jBPM 流程实例。

### 注意

请参阅`jbpm-remote-server` Maven 项目及其`JmsTest` jUnit 类以获取完整的工作示例（需要 WildFly 8.1 运行）。

#### 服务器 JMS 配置

WildFly 自带 HornetQ 作为 JMS 消息队列中间件；为了使 JMS 正常工作，我们需要检查 jBPM JMS 队列是否已注册到 JNDI 服务，并且用户安全设置已设置。默认情况下，HornetQ 将使用“其他”JAAS 安全域，这是 KIE Workbench Realm 用于认证的域（回想一下`user.properties`和`roles.properties`文件）。此外，HornetQ 在 WildFly 的`standalone-full.xml`配置文件中的以下元素中定义了授权设置（位于消息子系统下）：

```java
  <security-settings>
    <security-setting match="#">
      <permission type="send" roles="admin guest"/>
      <permission type="consume" roles="admin guest"/>
      <permission type="createNonDurableQueue" roles="admin guest"/>
      <permission type="deleteNonDurableQueue" roles="admin guest"/>
    </security-setting>
  </security-settings>
```

在这里，我们只是添加了 KIE 控制台`admin`角色（以及默认的`guest`角色）；`admin`角色已经配置了 JAAS。

现在，为了检查我们的 JMS 用户是否配置正确，请打开 jBoss 管理控制台（`http://localhost:9990/console`）并选择**配置/子系统/消息/目的地**，然后在顶部导航栏上选择**默认提供者**和**安全设置**；您应查看定义的用户。

![服务器 JMS 配置](img/9578OS_08_05.jpg)

WildFly jBPM JMS 队列配置定义在`jbpm-console.war\WEB-INF\bpms-jms.xml`文件中；远程可访问的队列注册在`java:jboss/exported` JNDI 命名空间中。

检查 jBPM JMS 队列是否正确绑定到 JNDI，请打开 jBoss 管理控制台（`http://localhost:9990/console`）并选择**运行/状态/子系统/JNDI 视图**；在这里，您应查看**KIE.AUDIT**、**KIE.SESSION**、**KIE.RESPONSE**和**KIE.TASK**队列。在这里，您还应看到列出的`RemoteConnectionFactory`；此工厂允许远程连接到 jBoss JNDI 命名空间（我们将在稍后看到这一点）。

![服务器 JMS 配置](img/9578OS_08_02.jpg)

### 注意

对于 WildFly 消息安全配置，请参阅[`docs.jboss.org/author/display/WFLY8/Messaging+configuration`](https://docs.jboss.org/author/display/WFLY8/Messaging+configuration)。有关官方 HornetQ 参考，请参阅最新文档[`docs.jboss.org/hornetq/2.4.0.Final/docs/user-manual/html`](http://docs.jboss.org/hornetq/2.4.0.Final/docs/user-manual/html)。

#### JMS 客户端实现

要设置远程 JMS 客户端连接，我们使用与 REST 客户端相同的方法；我们配置了一个由老牌的`RemoteRuntimeEngineFactory`提供的专用构建器。

### 注意

请参阅`jbpm-remote-server` Maven 项目和它的`JmsTest` jUnit 类，以获取完整的示例。

```java
// the deploymentId identifies the KIE module
public static String deploymentId = "com.packt.masterjbpm6:pizzadelivery:1.0";
/* the remoteInitialContext is an instance of the jBoss Naming
 service (InitialContext) and gives you access to the container
 remoting services for JMS */
/* the connectionfactory represents the JMS connection configuration settings */

RemoteJmsRuntimeEngineBuilder jmsEngineBuilder = RemoteRuntimeEngineFactory
  .newJmsBuilder().addDeploymentId(deploymentId)
  .addRemoteInitialContext(remoteInitialContext)
  .addUserName(jms_user).addPassword(jms_password)
  .addConnectionFactory(connectionfactory)
  .addTimeout(maxTimeoutSecs);
```

我们从构建器中获取工厂，从工厂中获取引擎：

```java
RemoteJmsRuntimeEngineFactory engineFactory = jmsEngineBuilder
  .buildFactory();
RuntimeEngine engine = remoteJmsFactory.newRuntimeEngine();
```

然后，我们从引擎中获取服务类：

```java
TaskService taskService = engine.getTaskService();
```

为了使 jBPM 远程客户端解析远程 jBPM 队列，我们需要将 jBoss JNDI 提供者 URL 配置如下：

```java
initialProps.setProperty(InitialContext.PROVIDER_URL,
"http-remoting://" + jbossServerHostName + ":8080");
```

WildFly 使用 HTTP 升级和端口复用功能来支持其几乎所有协议。历史上 jBoss 远程 JNDI 监听在 4447 端口，但现在在 8080 端口。

### 注意

对于完整的 WildFly 参考，请参阅[`docs.jboss.org/author/display/WFLY8/Documentation`](https://docs.jboss.org/author/display/WFLY8/Documentation)。

## SOAP API

jBPM 工作台通过暴露一个 SOAP 服务增加了额外的互操作性，该服务由`/jbpm-console/CommandService?WSDL`端点描述；该服务实现了一个单一的`execute`操作。在撰写本书时，由于 WSDL 中存在一些错误，因此 jBPM 6.2.0 版本的 WSDL 无法用于生成客户端类。

客户端 Maven 依赖如下：

```java
<dependency>
  <groupId>org.kie.remote.ws</groupId>
  <artifactId>kie-remote-ws-common</artifactId>
  <version>6.2.0.Final</version>
</dependency>
```

为了完整性，我们现在将描述如何使用其 SOAP API 调用 jBPM。我们的`jbpm-remote-server`测试项目，`SOAPTest` jUnit 测试类，创建了一个 Web 服务客户端，然后启动一个新的流程实例。

首先，我们按照以下方式从端点 URL 获取 WSDL 资源：

```java
URL commandWsdlUrl = new URL(
"http://localhost:8080/jbpm-console/CommandService?WSDL");
```

`execute`命令操作接受`JaxbCommandsRequest`命令，这是一个用于普通 jBPM 命令类的 DTO（可序列化）包装器（参见第六章，*核心架构*）。所有 jBPM 命令类也是 JAXB 注解类。

```java
StartProcessCommand startProcessCommand = new StartProcessCommand();
startProcessCommand.setProcessId(processID);
JaxbCommandsRequest request = new JaxbCommandsRequest(deploymentId, startProcessCommand);
```

`JaxbCommandsRequest`也可以接受一批命令，与 REST 或 JMS 远程 API 不同。

## 事务

当通过 REST、SOAP 或远程 Java API 调用 jBPM 时，你控制着事务管理。如果 jBPM 调用应该作为事务的一部分，并且这个调用失败或抛出异常，你必须处理它并执行回滚操作或补偿你那边的业务逻辑。

所有的远程 API 方法都会抛出`RemoteApiException`异常，以指示远程调用（无论是 REST 还是 JMS）已失败。

SOAP API 的`execute`操作会抛出`CommandWebServiceException`异常。如果你需要一个紧密集成的和事务传播机制，你应该考虑迁移到一个包装了完整 jBPM 服务的 EJB 层（参见本章开头的*与 JEE 集成*部分）。

# 概述

在本章中，我们扩展了针对企业架构集成的 jBPM 特性。我们讨论了 jBPM 暴露的核心服务以及如何使用不同的技术，如 JEE、SOAP、REST 和 JMS 来访问这些服务。

在下一章中，我们将关注在将 jBPM 部署到生产环境中需要特别注意的细节。
