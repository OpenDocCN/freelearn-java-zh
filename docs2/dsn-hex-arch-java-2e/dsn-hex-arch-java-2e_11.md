

# 第十一章：利用 CDI Bean 管理端口和用例

Quarkus 提供自己的依赖注入解决方案，称为**Quarkus DI**。它源自**Java 2.0**规范的**上下文和依赖注入**（**CDI**）。我们使用 CDI 将提供对象实例的责任委托给外部依赖，并在整个应用程序中管理其生命周期。市场上存在几个依赖注入解决方案承担这样的责任。Quarkus DI 就是其中之一。

使用依赖注入机制的价值在于，我们不再需要担心如何以及何时提供对象实例。依赖注入解决方案使我们能够自动创建并提供对象作为依赖项，通常使用注解属性。

在六边形架构的上下文中，框架和应用六边形是利用 CDI 解决方案提供的好处的好候选者。我们不必使用使用具体类注入依赖项的构造函数，而是可以使用 CDI 发现机制自动查找接口实现并将它们提供给应用程序。

在本章中，我们将学习如何通过将端口和用例转换为 Bean 来增强端口和用例的供应。我们将探索 Bean 作用域及其生命周期，并了解何时以及如何使用可用的 Bean 作用域。一旦我们了解了 CDI 基础知识，我们将学习如何将其应用于六边形系统。

本章将涵盖以下主题：

+   了解 Quarkus DI

+   将端口、用例和适配器转换为 CDI Bean

+   使用 Quarkus 和 Cucumber 测试用例

到本章结束时，你将了解如何通过将用例和端口转换为可以在六边形系统中注入的托管 Bean，将 Quarkus DI 集成到六边形应用中。你还将知道如何结合 Cucumber 使用 Quarkus 来测试用例。

# 技术要求

要编译和运行本章中展示的代码示例，你需要在你的计算机上安装最新的**Java SE 开发工具包**和**Maven 3.8**。它们适用于 Linux、Mac 和 Windows 操作系统。

你可以在 GitHub 上找到本章的代码文件，链接为[`github.com/PacktPublishing/-Designing-Hexagonal-Architecture-with-Java---Second-Edition/tree/main/Chapter11`](https://github.com/PacktPublishing/-Designing-Hexagonal-Architecture-with-Java---Second-Edition/tree/main/Chapter11)。

# 了解 Quarkus DI

**Quarkus DI**是 Quarkus 框架提供的依赖注入解决方案。这个解决方案也称为**ArC**，基于**Java 2.0 规范**的 CDI。Quarkus DI 并没有完全实现这样的规范。相反，它提供了一些定制和修改后的实现，这些实现更倾向于 Quarkus 项目的目标。然而，这些更改在你深入了解 Quarkus DI 提供的内容时更为明显。对于那些只使用 Java 2.0 规范中 CDI 描述的基本和最常见功能的人来说，Quarkus DI 的体验与其他 CDI 实现相似。

使用 Quarkus DI 或任何依赖注入解决方案的优势是我们可以更多地关注我们正在开发的软件的业务方面，而不是与提供应用程序功能所需的对象的供应和生命周期控制相关的管道活动。为了实现这种优势，Quarkus DI 处理所谓的豆。

## 与豆类一起工作

**豆类**是我们可以用作注入依赖项或作为依赖项本身被注入到其他豆类中的特殊对象。这种注入活动发生在容器管理的环境中。这个环境不过是应用程序运行的运行时环境。

豆类有一个上下文，它影响其实例对象何时以及如何创建。以下是由 Quarkus DI 支持的上下文的主要类型：

+   `ApplicationScoped`：带有此类上下文的豆在整个应用程序中可用。仅创建一个豆实例，并在所有注入此豆的系统区域中共享。另一个重要方面是`ApplicationScoped`豆是延迟加载的。这意味着只有在第一次调用豆的方法时才会创建豆实例。看看这个例子：

    ```java
    @ApplicationScoped
    class MyBean {
        public String name = "Test Bean";
        public String getName(){
            return name;
        }
    }
    class Consumer {
        @Inject
        MyBean myBean;
        public String getName() {
            return myBean.getName();
        }
    }
    ```

    `MyBean`类不仅对`Consumer`类可用，也对其他注入豆的类可用。只有在第一次调用`myBean.getName()`时，才会创建豆实例。

+   `Singleton`：与`ApplicationScoped`豆类似，对于`Singleton`豆，也只有一个豆对象被创建并在整个系统中共享。唯一的区别是`Singleton`豆是预先加载的。这意味着一旦系统启动，`Singleton`豆实例也会启动。以下是一个示例代码：

    ```java
    @Singleton
    class EagerBean { ... }
    class Consumer {
        @Inject
        EagerBean eagerBean;
    }
    ```

    `EagerBean`对象将在系统初始化期间创建。

+   `RequestScoped`：当我们只想让豆在与其关联的请求存在的时间内可用时，我们通常将豆标记为`RequestScope`。以下是我们如何使用`RequestScope`的示例：

    ```java
    @RequestScoped
    class RequestData {
        public String getResponse(){
            return "string response";
        }
    }
    @Path("/")
    class Consumer {
        @Inject
        RequestData requestData;
        @GET
        @Path("/request")
        public String loadRequest(){
            return requestData.getResponse();
        }
    }
    ```

    每当请求到达`/request`时，都会创建一个新的`RequestData`豆对象，一旦请求完成，该对象就会被销毁。

+   `依赖性`: 标记为 `依赖性` 的豆豆（Beans）其作用域被限制在它们被使用的地方。因此，`依赖性` 豆豆不会在系统中的其他豆豆之间共享。此外，它们的生命周期与注入它们的豆豆中定义的生命周期相同。例如，如果你将一个 `依赖性` 注解的豆豆注入到一个 `RequestScoped` 豆豆中，前一个豆豆将使用后者的作用域：

    ```java
    @Dependent
    class DependentBean { ... }
    @ApplicationScoped
    class ConsumerApplication {
        @Inject
        DependentBean dependentBean;
    }
    @RequestScoped
    class ConsumerRequest {
        @Inject
        DependentBean dependentBean;
    }
    ```

    当将 `DependentBean` 类注入到 `ConsumerApplication` 时，它将成为 `ApplicationScoped`，注入到 `ConsumerRequest` 时将变为 `RequestScoped`。

+   `SessionScoped`: 我们使用这个作用域在同一个 HTTP 会话的所有请求之间共享豆豆上下文。我们需要 `quarkus-undertow` 扩展来在 Quarkus 上启用 `SessionScoped`：

    ```java
    @SessionScoped
    class SessionBean implements Serializable {
        public String getSessionData(){
            return "sessionData";
        }
    }
    @Path("/")
    class Consumer {
        @Inject
        SessionBean sessionBean;
        @GET
        @Path("/sessionData")
        public String test(){
            return sessionBean.getSessionData();
        }
    }
    ```

    在前面的示例中，在向 `/sessionData` 发送第一个请求之后，将创建一个 `SessionBean` 实例。这个相同的实例将可用于来自同一会话的其他请求。

总结来说，Quarkus 提供以下豆豆作用域：`ApplicationScoped`、`RequestScoped`、`Singleton`、`Dependent` 和 `SessionScoped`。对于无状态应用程序，大多数情况下，你可能只需要 `ApplicationScoped` 和 `RequestScoped`。通过了解这些作用域的工作方式，我们可以根据系统需求进行选择。

现在我们已经了解了 Quarkus DI 的优势以及其基本工作原理，让我们学习如何使用六角架构中的端口和用例来应用依赖注入技术。

# 将端口、用例和适配器转换为 CDI 豆豆

在为拓扑和库存系统设计应用程序六边形时，我们将用例定义为接口，并将输入端口定义为它们的实现。我们还在框架六边形中定义了输出端口，并将输出适配器定义为它们的实现。在本节中，我们将重构来自应用程序和框架六边形的组件，以启用与 Quarkus DI 一起使用依赖注入。

与 Quarkus DI 一起工作的第一步是在项目的根 `pom.xml` 中添加以下 Maven 依赖项：

```java
<dependency>
  <groupId>io.quarkus</groupId>
  <artifactId>quarkus-resteasy</artifactId>
</dependency>
```

除了 RESTEasy 库之外，这个 `quarkus-resteasy` 库还提供了与 Quarkus DI 一起工作的所需库。

让我们从与路由管理相关的类和接口开始我们的重构工作。

## 实现路由管理对象中的 CDI

在开发拓扑和库存系统时，我们定义了一系列端口、用例和适配器来管理路由相关的操作。我们将逐步介绍所需的更改以启用此类操作中的依赖注入：

1.  我们首先将 `RouterManagementH2Adapter` 输出适配器转换为托管豆豆：

    ```java
    import jakarta.enterprise.context.ApplicationScoped;
    @ApplicationScoped
    public class RouterManagementH2Adapter implements
      RouterManagementOutputPort {
        @PersistenceContext
        private EntityManager em;
       /** Code omitted **/
            private void setUpH2Database() {
            EntityManagerFactory entityManagerFactory =
            Persistence.createEntityManagerFactory(
              "inventory");
            EntityManager em =
            entityManagerFactory.createEntityManager();
            this.em = em;
        }
    }
    ```

    我们通过在`RouterManagementH2Adapter`类上放置`@ApplicationScoped`注解，将这个类转换为一个托管 Bean。注意`EntityManager`属性——我们也可以在这个属性上使用依赖注入。我们将在*第十三章*中这样做，*使用输出适配器和 Hibernate Reactive 持久化数据*，但现在我们不会涉及这一点。

1.  在更改`RouterManagementUseCase`接口及其实现`RouterManagementInputPort`之前，让我们分析一下当前实现的一些方面：

    ```java
    public interface RouterManagementUseCase {
        void setOutputPort(
        RouterManagementOutputPort
          routerManagementOutputPort);
        /** Code omitted **/
    }
    ```

    我们定义了`setOutputPort`方法来接收和设置一个`RouterManagementOutputPort`类型的实例，这由一个`RouterManagementH2Adapter`输出适配器来满足。由于我们不再需要显式提供这个输出适配器对象（因为 Quarkus DI 将注入它），我们可以从`RouterManagementUseCase`接口中删除`setOutputPort`方法。

    以下代码演示了在没有 Quarkus DI 的情况下如何实现`RouterManagementInputPort`：

    ```java
    @NoArgsConstructor
    public class RouterManagementInputPort implements
      RouterManagementUseCase {
        private RouterManagementOutputPort
        routerManagementOutputPort;
        @Override
        public void setOutputPort(
        RouterManagementOutputPort
          routerManagementOutputPort) {
            this.routerManagementOutputPort =
            routerManagementOutputPort;
        }
        /** Code omitted **/
    }
    ```

    要提供一个`RouterManagementOutputPort`类型的对象，我们需要使用之前提到的`setOutputPort`方法。在实现 Quarkus DI 之后，这将不再必要，正如我们将在下一步看到的那样。

1.  这就是实现 Quarkus DI 后`RouterManagementOutputPort`应该看起来像什么：

    ```java
    import jakarta.enterprise.context.ApplicationScoped;
    import jakarta.inject.Inject;
    @ApplicationScoped
    public class RouterManagementInputPort implements
      RouterManagementUseCase {
        @Inject
        RouterManagementOutputPort
          routerManagementOutputPort;
        /** Code omitted **/
    }
    ```

    首先，我们在`RouterManagementInputPort`上添加`ApplicationScoped`以使其能够注入到其他系统部分。然后，通过使用`@Inject`注解，我们注入`RouterManagementOutputPort`。我们不需要引用输出适配器的实现。Quarkus DI 将找到适当的实现来满足这个输出端口接口，这恰好是我们之前转换成托管 Bean 的`RouterManagementH2Adapter`输出适配器。

1.  最后，我们必须更新`RouterManagementGenericAdapter`输入适配器：

    ```java
    @ApplicationScoped
    public class RouterManagementGenericAdapter {
        @Inject
        private RouterManagementUseCase
          routerManagementUseCase;
        /** Code omitted **/
    }
    ```

    我们不能使用构造函数来初始化`RouterManagementUseCase`，而必须通过`@Inject`注解提供依赖。在运行时，Quarkus DI 将为该用例引用创建并分配一个`RouterManagementInputPort`对象。

对于与路由管理相关的类和接口，我们必须做的更改就这些了。现在，让我们学习关于开关管理类和接口我们需要更改的内容。

## 实现开关管理对象的 CDI

在本节中，我们将遵循与我们在重构与路由管理相关的端口、用例和适配器时相同的路径：

1.  我们首先将`SwitchManagementH2Adapter`输出适配器转换为一个托管 Bean：

    ```java
    import jakarta.enterprise.context.ApplicationScoped;
    @ApplicationScoped
    public class SwitchManagementH2Adapter implements
      SwitchManagementOutputPort {
        @PersistenceContext
        private EntityManager em;
        /** Code omitted **/
    }
    ```

    `SwitchManagementH2Adapter`适配器也使用了`EntityManager`。我们不会修改`EntityManager`对象提供的方式，但在*第十三章*中，*使用输出适配器和 Hibernate Reactive 持久化数据*，我们将将其改为使用依赖注入。

1.  我们在*第九章*中更改了`SwitchManagementUseCase`接口的定义，*使用 Java 模块应用依赖倒置*，并定义了`setOutputPort`方法：

    ```java
    public interface SwitchManagementUseCase {
        void setOutputPort(
        SwitchManagementOutputPort
          switchManagementOutputPort)
    /** Code omitted **/
    }
    ```

    由于 Quarkus DI 将提供适当的`SwitchManagementOutputPort`实例，我们不再需要这个`setOutputPort`方法，因此我们可以将其删除。

1.  以下代码显示了在没有依赖注入的情况下如何实现`SwitchManagementInputPort`：

    ```java
    @NoArgsConstructor
    public class SwitchManagementInputPort implements
      SwitchManagementUseCase {
        private SwitchManagementOutputPort
        switchManagementOutputPort;
        @Override
        public void setOutputPort(
        SwitchManagementOutputPort
          switchManagementOutputPort) {
            this.switchManagementOutputPort =
            switchManagementOutputPort;
        }
        /** Code omitted **/
    }
    ```

    我们调用`setOutputPort`方法来初始化一个`SwitchManagementOutputPort`对象。在使用依赖注入技术时，不需要显式实例化或初始化对象。

1.  实现依赖注入后，`SwitchManagementInputPort`应该看起来是这样的：

    ```java
    import jakarta.enterprise.context.ApplicationScoped;
    import jakarta.inject.Inject;
    @ApplicationScoped
    public class SwitchManagementInputPort implements
      SwitchManagementUseCase {
        @Inject
        private SwitchManagementOutputPort
        switchManagementOutputPort;
        /** Code omitted **/
    }
    ```

    我们使用`@ApplicationScoped`注解将`SwitchManagementInputPort`转换为托管 Bean，并使用`@Inject`注解让 Quarkus DI 发现实现`SwitchManagementOutputPort`接口的托管 Bean 对象，恰好是`SwitchManagementH2Adapter`输出适配器。

1.  我们仍然需要调整`SwitchManagementGenericAdapter`输入适配器：

    ```java
    public class SwitchManagementGenericAdapter {
        @Inject
        private SwitchManagementUseCase
          switchManagementUseCase;
        @Inject
        private RouterManagementUseCase
          routerManagementUseCase;
        /** Code omitted **/
    }
    ```

    在这里，我们正在为`SwitchManagementUseCase`和`RouterManagementUseCase`对象注入依赖项。在使用注解之前，这些依赖项以这种方式提供：

    ```java
    public SwitchManagementGenericAdapter (
    RouterManagementUseCase routerManagementUseCase,
      SwitchManagementUseCase switchManagementUseCase){
        this.routerManagementUseCase =
          routerManagementUseCase;
        this.switchManagementUseCase =
          switchManagementUseCase;
    }
    ```

    我们获得的好处是，我们不再需要依赖于构造函数来初始化`SwitchManagementGenericAdapter`依赖项。Quarkus DI 将自动为我们提供所需的实例。

下一节是关于网络管理操作的内容。让我们学习我们应该如何更改它们。

## 实现网络管理类和接口的 CDI

对于网络部分，我们不需要做太多更改，因为我们没有为网络相关操作创建特定的输出端口和适配器。因此，实现更改将仅发生在用例、输入端口和输入适配器上：

1.  让我们先看看`NetworkManagementUseCase`用例接口：

    ```java
    public interface NetworkManagementUseCase {
        void setOutputPort(
        RouterManagementOutputPort
          routerNetworkOutputPort);
        /** Code omitted **/
    }
    ```

    正如我们在其他用例中所做的那样，我们也定义了`setOutputPort`方法以允许初始化`RouterManagementOutputPort`。在实现 Quarkus DI 之后，此方法将不再需要。

1.  这是实现没有 Quarkus DI 的`NetworkManagementInputPort`的方式：

    ```java
    import jakarta.enterprise.context.ApplicationScoped;
    import jakarta.inject.Inject;
    public class NetworkManagementInputPort implements
      NetworkManagementUseCase {
        private RouterManagementOutputPort
        routerManagementOutputPort;
        @Override
        public void setOutputPort(
        RouterManagementOutputPort
          routerManagementOutputPort) {
            this.routerManagementOutputPort =
           routerManagementOutputPort;
        }
        /** Code omitted **/
    }
    ```

    `NetworkManagementInputPort`输入端口仅依赖于`RouterManagementOutputPort`，在没有依赖注入的情况下，它通过`setOutputPort`方法进行初始化。

1.  这是实现 Quarkus DI 后`NetworkManagementInputPort`的样子：

    ```java
    @ApplicationScoped
    public class NetworkManagementInputPort implements
      NetworkManagementUseCase {
        @Inject
        private RouterManagementOutputPort
        routerManagementOutputPort;
        /** Code omitted **/
    }
    ```

    如您所见，`setOutputPort`方法已被删除。现在，Quarkus DI 通过`@Inject`注解提供`RouterManagementOutputPort`的实现。`@ApplicationScoped`注解将`NetworkManagementInputPort`转换为托管 Bean。

1.  最后，我们必须更改`NetworkManagementGenericAdapter`输入适配器：

    ```java
    import jakarta.enterprise.context.ApplicationScoped;
    import jakarta.inject.Inject;
    @ApplicationScoped
    public class NetworkManagementGenericAdapter {
        @Inject
        private SwitchManagementUseCase
          switchManagementUseCase;
        @Inject
        private NetworkManagementUseCase
          networkManagementUseCase;
       /** Code omitted **/
    }
    ```

    `NetworkManagementGenericAdapter`输入适配器依赖于`SwitchManagementUseCase`和`NetworkManagementUseCase`用例，在系统中触发与网络相关的操作。正如我们在之前的实现中所做的那样，这里我们使用`@Inject`在运行时提供依赖项。

    以下代码展示了在 Quarkus DI 之前这些依赖是如何提供的：

    ```java
    public NetworkManagementGenericAdapter(
    SwitchManagementUseCase switchManagementUseCase, Net
      workManagementUseCase networkManagementUseCase) {
        this.switchManagementUseCase =
          switchManagementUseCase;
        this.networkManagementUseCase =
          networkManagementUseCase;
    }
    ```

    在实现注入机制之后，我们可以安全地移除这个`NetworkManagementGenericAdapter`构造函数。

我们已经完成了所有必要的更改，将输入端口、用例和适配器转换为可用于依赖注入的组件。这些更改展示了如何将 Quarkus CDI 机制集成到我们的六边形应用程序中。

现在，让我们学习如何将六边形系统适应以在测试期间模拟和使用托管 Bean。

# 使用 Quarkus 和 Cucumber 测试用例

在实现应用程序六边形的过程中*第七章*，*构建应用程序六边形*，我们使用了 Cucumber 来帮助我们塑造和测试我们的用例。通过利用 Cucumber 提供的行为驱动设计技术，我们能够以声明性方式表达用例。现在，我们需要集成 Cucumber，使其与 Quarkus 协同工作：

1.  第一步是将 Quarkus 测试依赖项添加到应用程序六边形的`pom.xml`文件中：

    ```java
    <dependency>
      <groupId>io.quarkiverse.cucumber</groupId>
      <artifactId>quarkus-cucumber</artifactId>
      <version>1.0    .0</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>io.quarkus</groupId>
      <artifactId>quarkus-junit5</artifactId>
      <scope>test</scope>
    </dependency>
    ```

    `quarkus-cucumber`依赖提供了我们运行 Quarkus 测试所需的集成。我们还需要`quarkus-junit5`依赖，它使我们能够使用`@QuarkusTest`注解。

1.  接下来，我们必须添加必要的 Cucumber 依赖项：

    ```java
    <dependency>
      <groupId>io.cucumber</groupId>
      <artifactId>cucumber-java</artifactId>
      <version>${cucumber.version}</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>io.cucumber</groupId>
      <artifactId>cucumber-junit</artifactId>
      <version>${cucumber.version}</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>io.cucumber</groupId>
      <artifactId>cucumber-picocontainer</artifactId>
      <version>${cucumber.version}</version>
      <scope>test</scope>
    </dependency>
    ```

    通过添加`cucumber-java`、`cucumber-junit`和`cucumber-picocontainer`依赖项，我们可以在系统中启用 Cucumber 引擎。

让我们看看在没有 Quarkus 的情况下如何配置 Cucumber：

```java
package dev.davivieira.topologyinventory.application;
import io.cucumber.junit.Cucumber;
import io.cucumber.junit.CucumberOptions;
import org.junit.runner.RunWith;
@RunWith(Cucumber.class)
@CucumberOptions(
        plugin = {"pretty", "html:target/cucumber-result"}
)
public class ApplicationTest {
}
```

使用`@RunWith(Cucumber.class)`注解来激活 Cucumber 引擎。当使用 Quarkus 时，这是实现`ApplicationTest`的方式：

```java
package dev.davivieira.topologyinventory.application;
import io.quarkiverse.cucumber.CucumberQuarkusTest;
import io.quarkus.test.junit.QuarkusTest;
@QuarkusTest
public class ApplicationTest extends CucumberQuarkusTest {
}
```

`@QuarkusTest`注解激活了 Quarkus 测试引擎。通过扩展`CucumberQuarkusTest`类，我们也启用了 Cucumber 测试引擎。

在`ApplicationTest`类上没有测试，因为这个只是一个启动类。记住，Cucumber 测试是在单独的类中实现的。在更改这些类之前，我们需要模拟所需的托管 Bean，以提供`RouterManagementOutputPort`和`SwitchManagementOutputPort`的实例。

让我们为`RouterManagementOutputPort`创建一个模拟 Bean 对象：

```java
package dev.davivieira.topologyinventory.application.mocks;
import dev.davivieira.topologyinventory.applica
  tion.ports.output.RouterManagementOutputPort;
import dev.davivieira.topologyinventory.domain.en
  tity.Router;
import dev.davivieira.topologyinventory.domain.vo.Id;
import io.quarkus.test.Mock;
@Mock
public class RouterManagementOutputPortMock implements
  RouterManagementOutputPort {
    @Override
    public Router retrieveRouter(Id id) {
        return null;
    }
    @Override
    public Router removeRouter(Id id) {
        return null;
    }
    @Override
    public Router persistRouter(Router router) {
        return null;
    }
}
```

这是一个我们创建的虚拟模拟 Bean，用于防止 Quarkus 抛出`UnsatisfiedResolutionException`。通过使用`@Mock`注解，Quarkus 将实例化`RouterManagementOutputPortMock`类，并在测试期间将其作为 Bean 注入。

同样地，我们将模拟`SwitchManagementOutputPort`：

```java
package dev.davivieira.topologyinventory.application.mocks;
import dev.davivieira.topologyinventory.applica
  tion.ports.output.SwitchManagementOutputPort;
import dev.davivieira.topologyinventory.domain.en
  tity.Switch;
import dev.davivieira.topologyinventory.domain.vo.Id;
import io.quarkus.test.Mock;
@Mock
public class SwitchManagementOutputPortMock implements
  SwitchManagementOutputPort {
    @Override
    public Switch retrieveSwitch(Id id) {
        return null;
    }
}
```

对于 `SwitchManagementOutputPort`，我们创建了 `SwitchManagementOutputPortMock` 以提供一个虚拟的托管 Bean，这样 Quarkus 就可以在测试期间使用它进行注入。如果没有模拟，我们需要从 `RouterManagementH2Adapter` 和 `SwitchManagementH2Adapter` 输出适配器获取真实实例。

尽管我们在测试期间没有直接引用输出接口和输出端口适配器，但 Quarkus 仍然试图在它们上执行 Bean 发现。这就是为什么我们需要提供模拟的原因。

现在，我们可以重构测试以使用 Quarkus DI 提供的依赖注入。让我们在 `RouterAdd` 测试中学习如何做到这一点：

```java
public class RouterAdd extends ApplicationTestData {
    @Inject
    RouterManagementUseCase routerManagementUseCase;
   /** Code omitted **/
}
```

在使用 Quarkus DI 之前，这是我们对 `RouterManagementUseCase` 的实现方式：

```java
this.routerManagementUseCase = new RouterManagementInput
  Port();
```

一旦实现了 `@Inject` 注解，就可以删除前面的代码。

我们可以在重构其他测试类时遵循相同的做法，添加 `@Inject` 注解并删除构造函数调用以实例化输入端口对象。

运行与 Cucumber 集成的 Quarkus 测试后，你将得到以下类似的结果：

```java
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running dev.davivieira.topologyinventory.application.ApplicationTest
2021-09-08 22:44:15,596 INFO  [io.quarkus] (main) Quarkus 2.2.1.Final on JVM started in 1.976s. Listening on: http://localhost:8081
2021-09-08 22:44:15,618 INFO  [io.quarkus] (main) Profile test activated.
2021-09-08 22:44:15,618 INFO  [io.quarkus] (main) Installed features: [cdi, cucumber, smallrye-context-propagation]
@RouterCreate
Scenario: Creating a new core router
#dev/davivieira/topologyinventory/application/routers/RouterCreate.feature:4
.  Given I provide all required data to create a core router
#dev.davivieira.topologyinventory.application.RouterCreate.create_core_router()
.  Then A new core router is created
#dev.davivieira.topologyinventory.application.RouterCreate.a_new_core_router_is_created()
```

注意，在已安装功能的输出条目中，Quarkus 提到了 `CDI` 和 `Cucumber` 作为正在使用的扩展。

在本节中，我们学习了如何配置 Quarkus 正确地与 Cucumber 一起工作。此配置是为了配置 Quarkus 模拟并重构测试类，以注入输入端口对象而不是通过构造函数调用创建它们。

# 摘要

在本章中，我们有机会学习 Quarkus 如何通过 Quarkus DI 提供依赖注入。我们首先回顾了 CDI 为 **Java 2.0** 规范定义的一些概念，该规范是 Quarkus DI 所依据的。然后，我们继续在我们的六边形应用程序中实现这些概念。我们在重构用例、端口和适配器时定义了托管 Bean 并注入了它们。最后，我们学习了如何将 Quarkus 与 Cucumber 集成，以便在测试我们的六边形应用程序时获得两者的最佳效果。

通过将 Quarkus 依赖注入机制集成到六边形系统中，我们也将它转变为一个更健壮和现代的系统。

在下一章中，我们将关注适配器。Quarkus 提供了创建反应式 REST 端点的强大功能，我们将学习如何将它们与六边形系统适配器集成。

# 问题

1.  Quarkus DI 基于哪个 Java 规范？

1.  `ApplicationScoped` 和 `Singleton` 范围之间的区别是什么？

1.  我们应该使用哪种注解来通过 Quarkus DI 提供依赖关系，而不是使用调用构造函数？

1.  要启用 Quarkus 测试功能，我们应该使用哪个注解？

# 答案

1.  它基于 **Java** **2.0** 规范的 CDI。

1.  当使用 `ApplicationScope` 时，对象是延迟加载的。使用 `Singleton` 时，对象是预先加载的。

1.  `@Inject` 注解。

1.  `@QuarkusTest` 注解。
