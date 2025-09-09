# 实施现代 Java 企业应用程序

现在我们已经看到了项目中包含哪些组件以及如何找到和构建合理大小的模块和包，让我们更接地气地讨论 Java EE 的话题。首先考虑业务问题，并遵循领域驱动设计的实践来识别边界上下文和包含我们领域所有内容的模块，这当然是有意义的。

让我们看看如何实现已识别的业务模块和用例。

本章将涵盖：

+   如何实现应用程序用例边界

+   Java EE 核心领域组件是什么

+   使用 Java EE 进行设计模式和领域驱动设计

+   应用程序通信

+   如何集成持久化

+   技术横切关注点和异步行为

+   Java EE 的概念和原则

+   如何实现可维护的代码

# 用例边界

根据领域关注点组织包，使我们达到一个架构结构，其中实际业务而不是技术细节得到反映。

业务用例处理实现业务目的所需的所有逻辑，使用我们模块的所有内容。它们作为进入应用程序领域的起点。用例通过系统边界公开和调用。企业系统提供与外部世界的通信接口，主要通过 Web 服务或基于 Web 的前端，调用业务功能。

在启动新项目时，首先从领域逻辑开始是有意义的，不考虑系统边界或任何技术实现细节。这包括构建领域所有内容、设计类型、依赖和责任，并将这些原型编码化。正如我们将在本章中看到的，实际的领域逻辑主要是用纯 Java 实现的。初始模型可以自给自足，仅使用代码级别测试进行测试。在找到一个足够成熟的领域模型后，我们将针对领域模块之外的技术问题进行目标定位，例如访问数据库或外部系统，以及系统端点。

在 Java EE 应用程序中，边界是通过托管 Bean 实现的，即**企业 JavaBean**（**EJB**）或**Java 的上下文和依赖注入**（**CDI**）托管 Bean。主题“EJB 和 CDI - 区分和集成”将展示这些技术的差异和重要性。

根据单个用例的复杂性，我们引入了*委托*，这些委托作为 CDI 托管 Bean 或 EJB 实现，具体取决于需求。这些委托位于控制包中。实体作为 POJO 实现，可选地注解以集成技术功能，例如指定数据库映射或序列化。

# 现代 Java EE 的核心领域组件

纯 Java 与 CDI 和 EJB 结合，形成了现代 Java EE 应用程序的核心领域组件。为什么称之为核心领域？正如所述，我们希望关注实际业务。有些方面、组件和功能在其核心上服务于业务目的，而其他方面只是*支持*，使业务领域可访问，或满足其他技术需求。

Java EE 提供了许多 API，支持实现数十个技术需求。尽管大多数 API 都是技术驱动的。然而，Java EE 平台最大的优势是，可以使用最少的代码影响实现干净的 Java 业务逻辑。为此所需的 API 主要是 CDI 和 EJB。其他 API，如 JPA、JAX-RS、JSON-P 等，由于技术驱动的原因，其引入具有次要优先级。

管理 Bean，无论是 CDI 还是 EJB，都作为注解 Java 类实现，不需要任何技术超类或接口。在过去，这被称为无接口视图。如今，这已成为默认情况。扩展类会模糊领域视图，并且在可测试性方面也存在其他缺点。一个现代框架尽可能地简单和精简地集成自身。

# EJB 和 CDI - 区别与集成

现在的问题是，是否使用 EJB 或 CDI 管理的 Bean。

通常，EJB 提供了更多开箱即用的功能。CDI 管理的 Bean 提供了一种相对较轻的替代方案。这些技术的主要区别是什么，以及它们如何影响开发者的工作？

第一个区别在于作用域。EJB 会话 Bean 可以是无状态的，即在客户端请求期间活跃，也可以是有状态的，即在客户端的 HTTP 会话生命周期内活跃，或者单例。CDI 管理的 Bean 具有类似的作用域，还有更多可能性，例如添加自定义作用域和默认的依赖作用域，该作用域根据其注入点的生命周期而活跃。主题“作用域”将更详细地处理 Bean 的作用域。

EJB 和 CDI Bean 之间的另一个区别是，EJB 隐式包含某些横切关注点，例如监控、事务、异常处理以及为单例 Bean 管理并发。例如，调用 EJB 业务方法隐式启动一个技术事务，该事务在方法执行期间活跃，并集成数据源或外部系统。

此外，无状态 EJB 在使用后会被池化。这意味着在无状态会话 Bean 的业务方法被调用后，Bean 实例可以并且将被容器重用。由于这个原因，EJB 的性能略优于每次其作用域需要时都会实例化的 CDI Bean。

实际上，技术差异对开发者的工作影响不大。除了使用不同的注解外，这两种技术都可以用于相同的界面和感觉。Java EE 的发展方向是更开放地选择这两种技术；例如，自从 Java EE 8 以来，仅使用 CDI 就可以处理异步事件，而不仅仅是 EJB。

然而，CDI 提供的功能集成是 Java EE API 的最大特性之一。仅依赖注入、CDI 生产者和事件就是有效应对各种情况的手段。

最常用的 CDI 功能是使用`@Inject`注解进行依赖注入。注入是构建的，无论哪种 Java EE 技术管理豆，它对开发者来说“只需工作”。你可以混合和匹配 CDI 豆和 EJBs，使用所有作用域；框架将负责在哪个作用域中实例化或使用哪些豆。这使使用更加灵活，例如，当短作用域的豆被注入到长作用域的豆中时；例如，当会话作用域的豆被注入到单例中时。

此功能以这种方式支持业务领域，使得边界和控制可以注入所需的依赖项，而无需担心它们的实例化或管理。

以下代码演示了如何将作为无状态会话豆实现的边界注入所需的控件。

```java
import javax.ejb.Stateless;
import javax.inject.Inject;

@Stateless
public class CarManufacturer {

    @Inject
    CarFactory carFactory;

    @Inject
    CarStorage carStorage;

    public Car manufactureCar(Specification spec) {
        Car car = carFactory.createCar(spec);
        carStorage.store(car);
        return car;
    }
}
```

`CarManufacturer`类代表一个无状态的 EJB。注入的`CarFactory`和`CarStorage`豆被实现为依赖作用域的 CDI 豆，它们将被实例化并注入到 EJB 中。Java EE 平台通过启用使用`@Inject`注解注入任何特定项目豆来简化依赖项解析。这并非总是如此；在过去，使用`@EJB`注解注入 EJB。`@Inject`简化了 Java EE 中的使用。

仔细的读者可能已经注意到了基于字段注入与包私有 Java 作用域。基于字段的注入对类的内容影响最小——因为可以避免自定义构造函数。包私有可见性允许开发者设置和注入测试作用域中的依赖项。我们将在第七章*测试*中介绍这个主题和潜在的替代方案。

# CDI 生产者

CDI 生产者是 Java EE 的另一个特别有用的功能，可以帮助实现各种类型的工厂。生产者主要实现为生产者方法，提供可以注入到其他管理豆中的对象。这解耦了创建和配置逻辑与使用。当需要注入除管理豆类型之外的定制类型时，生产者非常有用。

以下显示了 CDI 生产者方法的定义：

```java
import javax.enterprise.inject.Produces;

public class CarFactoryProducer {

    @Produces
    public CarFactory exposeCarFactory() {
        CarFactory factory = new BMWCarFactory();
        // use custom logic
        return factory;
    }
}
```

如前所述的`CarManufacturer`示例中所示，`CarFactory`类型可以通过`@Inject`简单地注入。CDI 在需要`CarFactory`实例时调用`exposeCarFactory()`方法，并将返回的对象插入到注入点。

这些技术已经涵盖了核心领域逻辑用例的大部分需求。

# 发射领域事件

CDI 为那些需要进一步解耦业务功能的情况提供了一个事件功能。Bean 可以触发事件对象，这些对象作为有效载荷，并在事件观察者中处理。通过发射和处理 CDI 事件，我们可以将主要业务逻辑从处理事件的侧面方面解耦。这个想法特别适合那些业务领域已经包含事件概念的使用案例。默认情况下，CDI 事件以同步方式处理；在它们被触发的地方中断执行。CDI 事件也可以异步处理或在技术事务的生命周期中的特定点处理。

以下代码演示了如何在业务用例中定义和触发 CDI 事件：

```java
import javax.enterprise.event.Event;

@Stateless
public class CarManufacturer {

    @Inject
    CarFactory carFactory;

    @Inject
    Event<CarCreated> carCreated;

    public Car manufactureCar(Specification spec) {
        Car car = carFactory.createCar(spec);
        carCreated.fire(new CarCreated(spec));
        return car;
    }
}
```

`CarCreated`事件是不可变的，并包含与领域事件相关的信息，例如汽车规格。该事件在`CreatedCarListener`类中处理，该类位于控制包中：

```java
import javax.enterprise.event.Observes;

public class CreatedCarListener {

    public void onCarCreated(@Observes CarCreated event) {
        Specification spec = event.getSpecification();
        // handle event
    }
}
```

因此，监听器与主要业务逻辑解耦。CDI 容器将负责连接事件处理功能，并同步调用`onCarCreated()`方法。

主题“执行流程”，展示了事件如何在事务的生命周期中的特定点异步触发和处理。

CDI 事件是将领域事件定义与处理它们解耦的一种方式。事件处理逻辑可以更改或增强，而无需触及汽车制造商组件。

# 作用域

当应用程序中保持的状态持续时间超过单个请求的持续时间时，Bean 作用域对于这种情况非常重要。

如果整个业务流程都可以以无状态的方式实现，只需执行一些逻辑并在之后丢弃所有状态，作用域定义就相当直接。具有依赖作用域的 CDI 无状态会话 Bean 已经满足了许多这些案例。

EJB 单例作用域和 CDI 应用程序作用域也经常被使用。Bean 类型的单例实例是存储或缓存具有长期生命周期的信息的直接方式。除了所有复杂的缓存技术之外，包含简单集合或映射的单一实例，具有管理的并发性，仍然是设计特定于应用程序的、易失性存储的最简单方式。单一实例还提供了一个单一的责任点，对于某些原因需要以受限方式访问的功能。

EJB 和 CDI 实体的作用域最后一个是会话作用域，它与客户端的 HTTP 会话绑定。只要用户的会话保持活跃，这种作用域的 Bean 就会保持活跃并复用其所有状态。然而，在状态 Bean 中存储会话数据会引入一个挑战，即客户端需要再次连接到相同的应用服务器。这当然是可以实现的，但会阻止设计无状态的、易于管理的应用。如果应用变得不可用，所有临时会话数据也会丢失。在现代企业应用中，状态通常存储在数据库或缓存中以优化目的。因此，会话作用域的 Bean 不再被广泛使用。

CDI 管理 Bean 带有更多内置的作用域，即会话作用域或默认的依赖作用域。也有可能为特殊需求添加自定义作用域。然而，经验表明，内置的作用域通常足以满足大多数企业应用的需求。CDI 规范提供了有关如何扩展平台和开发自定义作用域的更多信息。

正如你所见，我们已可以使用这些 Java EE 核心组件实现很多功能。在探讨集成技术，如 HTTP 通信或数据库访问之前，让我们先深入了解我们核心领域中使用的设计模式。

# Java EE 中的模式

关于设计模式已经有很多论述。最突出且总是被引用的例子是《设计模式》这本书，由“四人帮”（GoF）所著。它描述了在软件设计中使用特定实现模式解决的常见情况。

尽管特定模式的设计和动机至今仍然有效，但实际的实现可能已经改变，尤其是在企业领域。除了适用于所有类型应用的知名设计模式外，还有很多与企业相关的模式出现。特别是，过去出现了许多与 J2EE 相关的企业模式。由于我们现在处于 Java EE 8 时代，不再是 J2EE，现在有更简单的方法来实现各种模式，以应对特定情况。

# 重新审视设计模式

GoF 书中描述的设计模式被分为创建型、结构型和行为型。每个模式都描述了软件中的典型挑战，并展示了如何应对和解决这些情况。它们代表实现蓝图，不依赖于任何特定技术。这就是为什么这些模式的想法可以在不精确匹配描述的实现的情况下实现。在 Java SE 8 和 EE 8 的现代世界中，我们比过去有更多的语言特性可用。我想展示一些四人帮的设计模式、它们的动机以及如何在 Java EE 中实现它们。

# 单例

单例模式是一个众所周知的设计模式，或者有些人可能会认为它是一个反模式。单例在整个应用程序中每个类只有一个实例。这种模式的动机是能够在中央位置存储状态以及能够协调动作。单例确实有存在的理由。如果某些状态需要在多个消费者之间可靠地共享，那么一个单一的入口点无疑是简单的方法。

然而，有一些需要注意的点。拥有单一责任点也引入了需要管理的并发。因此，单例需要是线程安全的。话虽如此，我们应该记住，单例本身并不易于扩展，因为只有一个实例。由于包含的数据结构，我们引入的同步越多，我们的类在并发访问方面的扩展性就越差。然而，根据具体的使用情况，这可能是也可能不是问题。

GoF（设计模式：可复用面向对象软件的基础）书中描述了一个静态单例实例，该实例由单例类管理。在 Java EE 中，单例的概念直接集成到 EJBs（企业 JavaBeans）中的单例会话 Bean 和 CDIs（上下文依赖注入）的应用程序作用域中。这些定义将创建一个用于所有客户端的单例管理 Bean。

以下是一个单例 EJB 的示例：

```java
import javax.ejb.Singleton;

@Singleton
public class CarStorage {

    private final Map<String, Car> cars = new HashMap<>();

    public void store(Car car) {
        cars.put(car.getId(), car);
    }
}
```

在使用 EJB 单例会话 Bean 或 CDI 应用程序作用域 Bean 实现单例方面存在一些差异。

默认情况下，容器管理 EJB 单例的并发。这确保了同一时间只执行一个公共业务方法。可以通过提供`@Lock`注解来改变这种行为，该注解分别声明方法为写锁或读锁，其中 Bean 充当读写锁。所有 EJB 单例业务方法都是隐式写锁定的。以下是一个使用具有容器管理并发和锁注解的 EJB 的示例：

```java
import javax.ejb.Lock;
import javax.ejb.LockType;

@Singleton
public class CarStorage {

    private final Map<String, Car> cars = new HashMap<>();

    @Lock
    public void store(Car car) {
        cars.put(car.getId(), car);
    }

    @Lock(LockType.READ)
    public Car retrieve(String id) {
        return cars.get(id);
    }
}
```

可以通过使用 Bean 管理并发来关闭并发。然后 Bean 将被并发调用，实现本身必须确保线程安全。例如，使用线程安全的数据结构不需要 EJB 单例管理并发访问。EJB 实例的业务方法将并行调用，类似于 CDI 应用程序作用域 Bean：

```java
import javax.ejb.ConcurrencyManagement;
import javax.ejb.ConcurrencyManagementType;

@Singleton
@ConcurrencyManagement(ConcurrencyManagementType.BEAN)
public class CarStorage {

    private final Map<String, Car> cars = new ConcurrentHashMap<>();

    public void store(Car car) {
        cars.put(car.getId(), car);
    }

    public Car retrieve(String id) {
        return cars.get(id);
    }
}
```

CDI 应用程序作用域的 Bean 不限制并发访问，实现本身必须处理并发问题。

这些解决方案解决需要单例的情况；例如，需要在整个应用程序中共享内存中的状态。

CDI 应用程序作用域的 Bean 或具有 Bean 管理并发和线程安全数据结构的 EJB 单例提供了一个应用程序范围内的、非集群的内存缓存，扩展性非常好。如果不需要分布式处理，这是一个简单而优雅的解决方案。

EJB 单例的另一个广泛使用的场景是在应用启动时调用单个进程的能力。通过声明`@Startup`注解，该 Bean 将在应用启动时实例化和准备，调用`@PostConstruct`方法。可以为所有 EJB 定义启动过程，但使用单例我们可以实现需要恰好设置一次的过程。

# 抽象工厂

GoF 抽象工厂模式旨在将对象的创建与其使用分离。创建复杂对象可能需要了解某些先决条件、实现细节或要使用的实现类。工厂帮助我们创建这些对象，而无需深入了解内部。在本章的后面，我们将讨论与该模式密切相关的问题域驱动设计工厂。动机是相同的。抽象工厂旨在拥有多个抽象类型的实现，其中工厂本身也是一个抽象类型。功能的使用者针对接口进行开发，而具体的工厂将产生并返回具体的实例。

可能存在一个抽象的`GermanCarFactory`，具体实现为`BMWFactory`和`PorscheFactory`。这两个汽车工厂可能分别产生`GermanCar`的一些实现，无论是`BMWCar`还是`PorscheCar`。只想拥有一些德国汽车的客户端不会关心工厂将使用哪个实际的实现类。

在 Java EE 世界中，我们已经有了一个强大的功能，实际上是一个工厂框架，即 CDI。CDI 提供了大量功能来创建和注入特定类型的实例。虽然动机和结果相同，但在细节实现上有所不同。实际上，根据用例，实现抽象工厂的方法有很多。让我们看看其中的一些。

管理 Bean 可以注入具体或抽象的实例，甚至是参数化类型。如果我们想在当前的 Bean 中只有一个实例，我们直接注入一个`GermanCar`：

```java
@Stateless
public class CarEnthusiast {

    @Inject
    GermanCar car;

    ...
}
```

如果存在多个`GermanCar`类型的实现，那么在这一点上将会出现依赖解析异常，因为容器不知道应该注入哪一辆实际的汽车。为了解决这个问题，我们可以引入显式请求特定类型的限定符。我们可以使用可用的`@Named`限定符和定义的字符串值；然而，这样做不会引入类型安全。CDI 为我们提供了指定自己的类型安全限定符的可能性，这些限定符将匹配我们的用例：

```java
@BMW
public class BMWCar implements GermanCar {
    ...
}

@Porsche
public class PorscheCar implements GermanCar {
    ...
}
```

标准化是自定义运行时保留注解，自身被`@Qualifier`注解，通常还带有`@Documented`：

```java
import javax.inject.Qualifier;
import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Qualifier
@Documented
@Retention(RetentionPolicy.RUNTIME)
public @interface BMW {
}
```

限定符在注入点指定。它们限定注入的类型，并将注入与实际使用的类型解耦：

```java
@Stateless
public class CarEnthusiast {

    @Inject
    @BMW
    GermanCar car;

    ...
}
```

获取`CarEnthusiast`的实例现在将创建并注入一个依赖范围的`BMWCar`，因为这种类型与注入点匹配。

我们现在甚至可以定义一个宝马汽车的子类型，它将在不改变注入点的情况下使用。这是通过用不同的实现来*专门化*`BMWCar`类型来实现的。`ElectricBMWCar`类型是`BMWCar`的子类，并指定了`@Specializes`注解：

```java
import javax.enterprise.inject.Specializes;

@Specializes
public class ElectricBMWCar extends BMWCar {
    ...
}
```

专用豆继承了其父类型的类型和限定符，并将透明地用于代替父类型。在这个例子中，使用`@BMW`限定符注入`GermanCar`将提供一个`ElectricBMWCar`的实例。

然而，为了更接近书中描述的设计模式，我们也可以定义一个用于创建所需数量汽车的汽车工厂类型：

```java
public interface GermanCarManufacturer {
    GermanCar manufactureCar();
}
```

这个汽车工厂以不同的具体方式实现：

```java
@BMW
public class BMWCarManufacturer implements GermanCarManufacturer {

    @Override
    public GermanCar manufactureCar() {
        return new BMWCar();
    }
}

@Porsche
public class PorscheCarManufacturer implements GermanCarManufacturer {

    @Override
    public GermanCar manufactureCar() {
        return new PorscheCar();
    }
}
```

这样做，客户端现在可以直接注入并使用一个制造商来创建新的德国汽车：

```java
@Stateless
public class CarEnthusiast {

    @Inject @BMW
    GermanCarManufacturer carManufacturer;

    // create German cars
}
```

显式定义和指定的类型注入，例如我们的两款德国汽车，为实现提供了很大的灵活性。

# 工厂方法

要理解工厂方法，让我们看看另一个具有类似动机但实现方式不同的模式。工厂方法定义了作为特定类型上方法的工厂。没有单个类负责创建特定实例；相反，创建成为定义为一个域类部分的工厂方法的职责。

例如，让我们考虑一辆使用其记录的行程来生成驾驶员日志簿的汽车。在汽车类型中包含一个`createDriverLog()`方法，该方法返回日志簿值类型，这是完全合理的，因为该类本身可以以自给自足的方式提供逻辑。这些解决方案将纯粹在 Java 中实现，无需任何框架或注解：

```java
public class Car {

    ...

    public LogBook createDriverLog() {
        // create logbook statement
    }
}
```

正如我们将在本章后面看到的那样，领域驱动设计工厂不区分抽象工厂和工厂方法。它们更多地针对领域的动机。在某些情况下，将工厂封装为方法，与其他类的职责一起定义是有意义的。在其他情况下，如果创建逻辑是那种特定的、以单独类形式存在的单一责任点，则更合适。一般来说，将创建逻辑放入域类型是可取的，因为它可能利用该域类的其他功能和属性。

让我们看看 CDI 生产者。生产者被定义为用于动态查找和注入特定类型实例的方法或字段。我们对字段包含的值或方法返回的值有完全的灵活性。我们可以同样指定限定符以确保生产者不会与其他可能产生的类型冲突。定义生产方法豆也可以包含用于生产者的进一步属性：

```java
import javax.enterprise.inject.Produces;

public class BMWCarManufacturer {

    ...

    @Produces
    @BMW
    public GermanCar manufactureCar() {
        // use properties
        ...
    }
}
```

这与作为 CDI 生产者实现的工厂方法的想法相匹配。

需要考虑产生的实例的范围。与其他任何 CDI 管理 Bean 一样，生产者默认是依赖范围的。范围定义了管理 Bean 的生命周期以及它们的注入方式。它影响生产者方法将被调用的频率。对于默认范围，当调用管理 Bean 实例化时，方法对每个注入实例调用一次。每次注入产生值的 Bean 被注入时，都会调用生产者方法。如果该 Bean 有更长的生命周期，那么在该期间不会再次调用生产者方法。

在本章的后面部分，我们将看到 CDI 生产者更复杂的用法。

# 对象池

对象池设计模式是为了性能优化而设计的。池背后的动机是避免不断创建所需对象和依赖项的新实例，通过在对象池中保留它们更长时间来实现。所需实例从这个对象池中检索出来，并在使用后释放。

这个概念已经以不同形式内置到 Java EE 容器中。如前所述，无状态会话 Bean 是池化的。这也是它们表现异常出色的原因。然而，开发者必须意识到实例正在被重用；在使用后，实例不得保留任何状态。容器保留了一组这些实例。

另一个例子是数据库连接的池化。数据库连接的初始化成本相当高，因此保留几个供以后使用是有意义的。根据持久化实现的不同，一旦请求新的查询，这些连接就会被重用。

在企业应用程序中，线程也会被池化。在 Java 服务器环境中，客户端请求通常会导致一个 Java 线程来处理逻辑。处理完请求后，这些线程将被再次重用。线程池配置以及拥有不同的池是进一步性能优化的一个重要主题。我们将在第九章“监控、性能和日志”中介绍这个主题。

开发者通常不会自己实现对象池模式。容器已经为实例、线程和数据库包含了这种模式。应用程序开发者隐式地使用了这些可用功能。

# 装饰者

另一个众所周知的设计模式是装饰者模式。这个模式允许我们向对象添加行为，而不影响该类的其他对象。这种行为通常可以与几个子类型组合。

一个好的例子是食物。每个人在口味和成分上都有自己的偏好。以咖啡为例。我们可以喝纯黑咖啡，加牛奶，加糖，牛奶和糖都加，甚至加糖浆、奶油，或者未来可能流行的任何东西。而且这还没有考虑到不同的冲泡咖啡的方式。

下面的示例展示了使用纯 Java 实现的装饰者模式。

我们指定以下 `Coffee` 类型，它可以被 `CoffeeGarnish` 子类型装饰：

```java
public interface Coffee {

    double getCaffeine();
    double getCalories();
}

public class CoffeeGarnish implements Coffee {

    private final Coffee coffee;

    protected CoffeeGarnish(Coffee coffee) {
        this.coffee = coffee;
    }

    @Override
    public double getCaffeine() {
        return coffee.getCaffeine();
    }

    @Override
    public double getCalories() {
        return coffee.getCalories();
    }
}
```

默认的咖啡装饰只委托给其父咖啡。可能有多个咖啡的实现：

```java
public class BlackCoffee implements Coffee {

    @Override
    public double getCaffeine() {
        return 100.0;
    }

    @Override
    public double getCalories() {
        return 0;
    }
}
```

除了常规的黑咖啡外，我们还指定了一些装饰：

```java
public class MilkCoffee extends CoffeeGarnish {

    protected MilkCoffee(Coffee coffee) {
        super(coffee);
    }

    @Override
    public double getCalories() {
        return super.getCalories() + 20.0;
    }
}

public class SugarCoffee extends CoffeeGarnish {

    protected SugarCoffee(Coffee coffee) {
        super(coffee);
    }

    @Override
    public double getCalories() {
        return super.getCalories() + 30.0;
    }
}

public class CreamCoffee extends CoffeeGarnish {

    protected CreamCoffee(Coffee coffee) {
        super(coffee);
    }

    @Override
    public double getCalories() {
        return super.getCalories() + 100.0;
    }
}
```

使用咖啡类型，我们可以组合我们想要的具有特定行为的咖啡：

```java
Coffee coffee = new CreamCoffee(new SugarCoffee(new BlackCoffee()));
coffee.getCaffeine(); // 100.0
coffee.getCalories(); // 130.0
```

JDK 中装饰器模式的一个例子是 `InputStream` 类，它可以为文件、字节数组等添加特定行为。

在 Java EE 中，我们再次利用 CDI（Contexts and Dependency Injection）提供的装饰器功能。装饰器为 Bean 添加特定行为。对注入 Bean 的调用将调用装饰器而不是实际的 Bean；装饰器添加特定行为并将调用委托给 Bean 实例。原始 Bean 类型成为装饰器的所谓代理：

```java
public interface CoffeeMaker {
    void makeCoffee();
}

public class FilterCoffeeMaker implements CoffeeMaker {

    @Override
    public void makeCoffee() {
        // brew coffee
    }
}
```

代理类型必须是接口。`CountingCoffeeMaker` 装饰了现有的咖啡机功能：

```java
import javax.decorator.Decorator;
import javax.decorator.Delegate;
import javax.enterprise.inject.Any;

@Decorator
public class CountingCoffeeMaker implements CoffeeMaker {

    private static final int MAX_COFFEES = 3;
    private int count;

    @Inject
    @Any
    @Delegate
    CoffeeMaker coffeeMaker;

    @Override
    public void makeCoffee() {
        if (count >= MAX_COFFEES)
            throw new IllegalStateException("Reached maximum coffee limit.");
        count++;

        coffeeMaker.makeCoffee();
    }
}
```

装饰器功能通过 `beans.xml` 描述符激活。

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans 

        xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
        http://xmlns.jcp.org/xml/ns/javaee/beans_1_1.xsd"
        bean-discovery-mode="all">
    <decorators>
        <class>com.example.coffee.CountingCoffeeMaker</class>
    </decorators>
</beans>
```

激活装饰器后，`CoffeeMaker` 类型的注入实例将使用装饰后的功能。这不会改变原始实现：

```java
public class CoffeeConsumer {

    @Inject
    CoffeeMaker coffeeMaker;

    ...
}
```

托管 Bean 可以有多个装饰器。如果需要，可以使用 Java EE 的 `@Priority` 注解在装饰器上指定顺序。

这种 CDI 功能适用于托管 Bean。根据我们是否想向我们的领域模型类或涉及的服务添加额外的行为，我们将使用模式，要么使用纯 Java，如最初所述，要么使用 CDI 装饰器。

# 门面

门面设计模式用于提供对某些功能的干净且简单的接口。封装和抽象层无疑是编写代码时最重要的原则之一。我们引入门面，将复杂的功能或难以使用的遗留组件封装到更简单的接口中。因此，门面是抽象的一个典型例子。

让我们考虑一个咖啡店中相当复杂的设置。有磨豆机、咖啡机、秤和各种工具在使用中，所有这些都需要相应地配置：

```java
public class BaristaCoffeeShop {

    private BeanStore beanStore;
    private Grinder grinder;
    private EspressoMachine espressoMachine;
    private Scale scale;
    private Thermometer thermometer;
    private Hygrometer hygrometer;

    public GroundBeans grindBeans(Beans beans, double weight) { ... }

    public Beans fetchBeans(BeanType type) { ... }

    public double getTemperature() { ... }

    public double getHumidity() { ... }

    public Coffee makeEspresso(GroundBeans beans, Settings settings) { ... }
}
```

当然，可以争论这个类已经需要重构。然而，遗留类可能不容易更改。我们将引入一个扮演门面的咖啡师：

```java
@Stateless
public class Barista {

    @Inject
    BaristaCoffeeShop coffeeShop;

    public Coffee makeCoffee() {
        // check temperature & humidity
        // calculate amount of beans & machine settings
        // fetch & grind beans
        // operate espresso machine
    }
}
```

在 Java EE 领域，最突出的门面示例是由 EJB 实现的边界。它们为我们业务域中的业务用例提供了门面。除此之外，门面还可以使用各种托管 Bean 实现。门面适当地委托和编排复杂逻辑。精心选择的抽象可以提高软件设计，并且是我们努力追求的目标。

# 代理

代理设计模式可能是 Java EE 中包含的最明显的设计模式之一。注入的 bean 引用几乎在所有情况下都不是实际实例的引用，而是一个代理。代理是围绕实例的薄包装，可以添加某些功能。客户端甚至没有意识到它与代理而不是实际对象交互。

代理允许在企业环境中实现所需的横切功能，例如拦截器、事务、日志记录或监控。它们最初也是执行依赖注入所必需的。

应用程序开发者通常不会直接使用代理模式。然而，了解代理模式的一般工作原理以及在 Java EE 平台上的具体使用方法是推荐的。

# 观察者

观察者设计模式描述了在整体状态发生变化时，一个对象如何管理和通知观察者。观察者在主题上注册自己，稍后将被通知。观察者的通知可以是同步或异步的。

如前所述，CDI 包含一个事件功能，该功能实现了观察者模式。开发者不需要自己处理注册和通知逻辑；他们只需通过注解声明松耦合即可。正如在 *现代 Java EE 的核心领域组件* 主题中所示，`Event<T>` 类型以及 `@Observes` 注解声明了事件发布和观察。在 *执行流程* 主题中，我们将介绍异步 CDI 事件。

# 策略

策略设计模式用于在运行时动态选择一个实现算法，即策略。该模式用于根据情况选择不同的业务算法，例如。

根据情况，我们可以有多种方式来利用策略模式。我们可以将算法的不同实现定义为单独的类。Java SE 8 包含了可以用于轻量级策略实现的 lambda 方法和方法引用功能：

```java
import java.util.function.Function;

public class Greeter {

    private Function<String, String> strategy;

    String greet(String name) {
        return strategy.apply(name) + ", my name is Duke";
    }

    public static void main(String[] args) {
        Greeter greeter = new Greeter();

        Function<String, String> formalGreeting = name -> "Dear " + name;
        Function<String, String> informalGreeting = name -> "Hey " + name;

        greeter.strategy = formalGreeting;
        String greeting = greeter.greet("Java");

        System.out.println(greeting);
    }

}
```

示例表明，功能接口可以用于动态定义在运行时应用和选择的策略。

在 Java EE 环境中，我们又可以利用 CDI 依赖注入。为了展示 CDI 支持任何 Java 类型，我们将使用一个表示功能接口的策略的相同示例。问候策略由 `Function` 类型表示：

```java
public class Greeter {

    @Inject
    Function<String, String> greetingStrategy;

    public String greet(String name) {
        return greetingStrategy.apply(name);
    }
}
```

一个 CDI 生产者方法可以动态选择问候策略：

```java
public class GreetingStrategyExposer {

    private Function<String, String> formalGreeting = name -> "Dear " + name;
    private Function<String, String> informalGreeting = name -> "Hey " + name;

    @Produces
    public Function<String, String> exposeStrategy() {
        // select a strategy
        ...
        return strategy;
    }
}
```

为了完成示例，让我们引入用于算法实现的特定类。CDI 能够注入所有可以动态选择的特定类型的实例。

`GreetingStrategy` 类型在白天适宜时是可选择的：

```java
public interface GreetingStrategy {
    boolean isAppropriate(LocalTime localTime);
    String greet(String name);
}

public class MorningGreetingStrategy implements GreetingStrategy {
    @Override
    public boolean isAppropriate(LocalTime localTime) {
        ...
    }

    @Override
    public String greet(String name) {
        return "Good morning, " + name;
    }
}

public class AfternoonGreetingStrategy implements GreetingStrategy { ... }
public class EveningGreetingStrategy implements GreetingStrategy { ... }
```

CDI 生产者可以注入所有可能的 `GreetingStrategy` 实例，并根据它们的规范进行选择：

```java
public class GreetingStrategySelector {

    @Inject
    @Any
    Instance<GreetingStrategy> strategies;

    @Produces
    public Function<String, String> exposeStrategy() {
        for (GreetingStrategy strategy : strategies) {
            if (strategy.isAppropriate(LocalTime.now()))
                return strategy::greet;
        }
        throw new IllegalStateException("Couldn't find an appropriate greeting");
    }
}
```

`@Any`限定符在任何一个托管 Bean 上隐式存在。具有`Instance`类型和此限定符的注入点会注入所有匹配相应类型的实例，这里是指`GreetingStrategy`。`Instance`类型允许我们动态获取和限定特定类型的实例。它实现了对所有合格类型的迭代器。

通过提供自定义选择逻辑，我们选择了一个合适的策略，然后将其注入到问候者中。

CDI 提供了多种指定和选择不同策略的方法。根据具体情况，依赖注入可以用来将选择逻辑与使用逻辑分离。

# 进一步的模式

除了提到的使用特定 Java EE 功能实现的模式之外，还有一些设计模式仍然使用纯 Java 实现，正如 GoF 书中所描述的。提供的列表当然并不完整，但它包括了在企业项目中通常使用的模式。

有些设计模式是 Java EE 的核心，例如代理模式。另一个例子是中介者模式，它封装了一组对象之间的通信。例如，为了设计松耦合的通信，我们不会自己实现这个模式，而是使用实现它的 API 功能，例如 CDI 事件。

还有许多其他模式在 Java EE API 中用得不多，但会使用纯 Java 来实现。根据实际情况，CDI 可以用来支持对象的创建和实例化。这些模式的例子包括原型、构建器、适配器、桥接、组合、享元、责任链、状态和访问者。

再次查看企业 API，我们会发现，例如，在 JSON-P API 中大量使用了构建器模式。我参考了《设计模式》这本书，由四人帮编写，以了解进一步的用法和模式。

# 领域驱动设计

现在我们已经看到了 GoF 设计模式在 Java EE 时代的实现。除此之外，我想指出一些在我们核心领域应用的模式和概念，然后再继续探讨更纯粹的技术问题。Eric Evans 所著的《领域驱动设计》一书详细描述了这些模式和概念，它们支持构建尽可能准确地匹配实际业务领域的软件模型。特别是，强调了与领域专家沟通、共享通用、*无处不在*的领域语言、深入理解底层领域模型以及逐步重构它的重要性。领域驱动设计还引入了软件世界中的某些概念，如仓库、服务、工厂或聚合。

现在的问题是，这些概念是否以及如何用 Java 企业版实现？领域驱动设计始终旨在将应用程序的重要方面直接包含到领域模型中，而不是仅仅作为服务或事务脚本的一部分“外部”。我们将看到这一事实如何与 EJB 或 CDI 管理的豆很好地结合。

# 服务

领域驱动设计语言定义了服务的概念。服务负责编排各种业务逻辑过程。通常，它们是使用案例的入口点，并创建或管理领域模型的对象。服务将单一的业务流程步骤保持在一起。

如果你将这个概念与实体控制边界打包的思想和内容相对应，你会发现它满足了边界或控制相同的用途。因此，在 Java EE 中，这些服务将因此实现为 EJB 或 CDI 管理的豆。代表用例入口点的服务实现为边界；而那些编排进一步业务逻辑、访问数据库或外部系统的服务代表控制。

# 实体

领域驱动设计还定义了所谓的实体。正如其名称所暗示的，实体本质上代表了业务域实体。这些实体是特定领域内某个概念的可识别实例。用户、文章和汽车都是此类实体的例子。对于领域来说，实体能够被单独识别是很重要的。用户 *John Doe* 或用户 *John Smith* 调用某个用例时，这一点是有区别的。这一方面将实体与值对象区分开来。

实体以及其他模型对象，都实现为普通的 Java 类。仅为了业务域的功能，不需要框架支持。理想情况下，实体已经封装了某些在实体类型内部自包含的业务逻辑。这意味着我们不仅会使用具有属性以及 getter 和 setter 方法的简单 POJO 进行建模，还会包含对实体进行操作的业务相关方法。将业务逻辑直接集成到业务实体的核心中，增加了内聚性、理解性，并遵循单一职责原则。

通常，实体以及其他领域模型类型都会保存在数据库中。Java EE 支持使用 JPA 进行对象关系映射，用于持久化和检索对象及其对象层次结构。实际上，用于声明实体类型的 JPA 注解被称为 `@Entity`。在后续的子章节中，我们将详细看到 JPA 如何以最小的对模型类的影响来支持持久化领域模型类型。

# 值对象

不形成可识别实体但只包含特定 *值* 的业务域类型被称为值对象。值对象最好是不可变的，因此是可重用的，因为其内容不能改变。Java 枚举是这种类型的良好例子。任何身份不重要的对象都将实现为值对象。例如，对于 Java 枚举，返回 `Status.ACCEPTED` 的哪个实例并不重要，这里甚至只有一个枚举实例被用于所有地方。对于领域中的许多类型也是如此，例如地址。只要指向 *42 Wallaby Way, Sydney* 的地址值保持不变，我们引用哪个地址实例就无关紧要了。

根据值集是否有限，值对象要么被建模为枚举或 POJO，理想情况下是不可变的。不可变性代表了值对象的概念，并减少了潜在错误的可能性。改变多个位置共享的可变对象可能会导致不可预见的副作用。

由于值对象不是直接标识的，因此它们也不会直接在数据库中持久化和管理。它们当然可以作为对象图的一部分间接持久化，从实体或聚合中引用。JPA 支持管理非实体或聚合的对象的持久化。

# 聚合

聚合是领域驱动设计语言中的一个概念，有时会让开发者感到困惑。聚合是由几个实体或值对象组成的复杂模型，形成一个整体。出于一致性的原因，这个对象集合应该作为一个整体来访问和管理。直接访问某些包含对象的访问方法可能会导致不一致和潜在的错误。聚合背后的想法是为所有操作提供一个根对象。一个很好的例子是由四个轮子、发动机、底盘等组成的汽车。每当需要进行某些操作，如 *drive*，它将在整个汽车上调用，可能同时涉及多个对象。

聚合是定义对象层次结构根的实体。它们作为包含业务域功能并持有实体和值对象引用的普通 Java 类来实现。

因此，也可以使用 JPA 来持久化聚合。所有持久化操作都是在聚合上触发的，即根对象，并且会级联到其包含的对象上。JPA 支持持久化复杂对象层次结构，我们将在后面的子章节中看到这一点。

# 仓库

说到数据库访问，领域驱动设计定义了仓库，用于管理实体的持久性和一致性。仓库背后的动机是拥有一个单一的责任点，使得领域模型能够以一致性为前提进行持久化。定义这些功能不应使领域模型代码充斥着持久化实现细节。因此，领域驱动设计定义了仓库的概念，以自给自足和一致的方式封装这些操作。

仓库是特定实体类型持久化操作的入口点。由于只需要识别聚合和实体的实例，因此只有这些类型需要仓库。

在 Java EE 和 JPA 中，已经存在一个与仓库概念很好地匹配的功能，即 JPA 的`EntityManager`。实体管理器用于持久化、检索和管理定义为实体或潜在对象层次结构的对象。JPA 管理对象需要是可识别的实体的事实完美符合领域驱动设计理念中实体所设定的约束。

实体管理器被注入并用于托管 Bean 中。这符合服务作为边界或控制的想法，即服务旨在编排业务用例，在此处通过调用实体管理器来提供实体的持久性。

# 工厂

领域驱动设计工厂背后的动机是，创建领域对象可能需要比仅仅调用构造函数更复杂的逻辑和约束。创建一致的领域对象可能需要执行验证或复杂的过程。因此，我们将创建逻辑定义在特定的方法或类中，这些方法或类封装了这种逻辑，使其与领域其他部分隔离开。

这与之前讨论的抽象工厂和工厂方法设计模式背后的动机相同。因此，使用 CDI 特性实现的相同实现也适用于此处。实际上，CDI 规范就是一个工厂功能。

领域对象工厂也可以实现为另一个领域模型类（如实体）的一部分的方法。这些解决方案将完全用 Java 实现，不需要任何框架或注解。在工厂方法设计模式中讨论的汽车驾驶员日志簿功能是工厂方法包含在领域实体中的良好示例。如果领域类本身能够以自给自足的方式提供逻辑，那么在那里包含工厂逻辑也是完全合理的。

# 领域事件

领域事件代表与业务领域相关的事件。它们通常来自业务用例，并具有特定的领域语义。领域事件的例子包括`UserLoggedIn`（用户登录）、`ActiclePurchased`（文章购买）或`CoffeeBrewFinished`（咖啡煮制完成）。

领域事件通常实现为包含所需信息的值对象。在 Java 中，我们将事件实现为不可变的 POJOs。过去发生的事件不能在以后更改，因此强烈建议使它们不可变。如前所述，我们可以使用 CDI 事件功能以松耦合的方式发布和观察事件。在 CDI 中，所有 Java 类型都可以用作发布事件。因此，领域事件的概念是一个业务定义，而不是技术定义。

领域事件对于事件存储和事件驱动架构尤为重要，我们将在第八章“微服务和系统架构”中广泛讨论。

# 企业应用程序中的外部和横切关注点

现在我们已经看到了实现应用程序中领域逻辑所需的概念和实现。从理论上讲，实现独立业务逻辑已经足够；然而，如果它们不能从系统外部访问，这些用例将不会为顾客提供太多价值。

因此，让我们来看看由技术驱动的外部和横切关注点。这些功能不是业务域的核心，但同样需要实现。技术驱动的关注点的例子包括访问外部系统或数据库、配置应用程序或缓存。

# 与外部系统的通信

与外部世界的沟通是企业应用程序最重要的技术方面之一。没有这种沟通，应用程序几乎无法为顾客带来任何价值。

# 如何选择通信技术

当企业系统需要通信时，就会产生使用哪种通信协议和技术的问题。有许多同步和异步通信形式可供选择。有一些考虑因素需要在开始时考虑。

所选语言和框架支持哪些通信技术？是否存在需要某种形式通信的现有系统？系统是以同步还是异步方式交换信息？工程师团队熟悉哪种解决方案？系统是否位于一个高性能至关重要的环境中？

再次从商业角度来看，系统间的通信是必要的，并且不应该*妨碍*实施业务用例。话虽如此，信息交换最初应该以直接的方式实现，匹配特定的领域，无论通信是同步还是异步进行的。这些考虑不仅对实际实施有重大影响，而且对整个用例是否与所选解决方案匹配也有很大影响。因此，这是需要首先询问的问题之一，即通信是同步还是异步进行的。同步通信确保了交换信息的一致性和顺序。然而，与异步调用相比，它性能较低，并且不会无限扩展。异步通信导致涉及系统的耦合更松散，提高了整体性能以及开销，并允许系统并非始终可靠可用的情况。出于简单性的考虑，企业应用程序通常使用同步通信，并且也考虑到一致性。

选择的通信方式不仅需要语言和框架的支持，还需要使用到的环境和工具的支持。环境和网络设置对通信是否有任何限制？实际上，这正是过去广泛选择 SOAP 协议的原因之一；它能够通过网络端口`80`进行传输，这是大多数网络配置所允许的。工具支持，尤其是在开发和调试目的下，是另一个重要的方面。这也是 HTTP 被广泛使用的原因。

在 Java 世界中，可以说大多数现有的通信解决方案都得到了支持，无论是原生支持，如 HTTP，还是通过第三方库。这当然不是其他技术的情况。例如，SOAP 协议就存在这个问题。SOAP 协议的实现实际上只在 Java 和.NET 应用程序中看到。其他技术通常选择不同的通信形式。

通信技术的性能是需要考虑的问题，不仅在高性能环境中。与进程间或进程内通信相比，通过网络交换信息总是引入了巨大的开销。问题是这个开销有多大。这本质上涉及到信息的密度和处理消息或有效负载的性能。交换的信息是二进制格式还是纯文本格式？内容类型代表哪种格式？一般来说，具有高信息密度和低冗余的二进制格式性能更好，并且传输的数据量更小，但调试和理解也更困难。

另一个重要方面是通信解决方案的灵活性。所选技术不应过多地限制信息的交换。理想情况下，协议支持不同的通信方式；例如，同步和异步通信、二进制格式或超媒体。由于我们应用程序的主要关注点是业务逻辑，因此所选技术理想上应适应整体需求。

在今天的系统中，使用最广泛的通信协议是 HTTP。这有几个原因。HTTP 被各种语言平台、框架和库广泛支持。工具选择种类繁多，并且该协议为大多数软件工程师所熟知。HTTP 对如何使用它没有太多限制，因此可以应用于各种信息交换。它可以用于实现同步或异步通信、超媒体或直接调用远程功能，如远程过程调用。然而，HTTP 鼓励某些使用方式。我们将在下一个主题中讨论语义 HTTP、远程过程调用和 REST。

有些通信协议并非必然，但通常是建立在 HTTP 之上的。过去最突出的例子是 SOAP；一个较新的例子是 gRPC。这两种协议都实现了远程过程调用方法。远程过程调用代表了一种在网络上调用另一个系统函数的直接形式。需要指定输入和输出值。SOAP 通过 XML 格式实现了这些远程过程调用，而 gRPC 使用二进制协议缓冲区来序列化数据结构。

根据业务需求中通信的同步或异步行为，强烈建议一致地实现这种行为。一般来说，你应该避免混合同步或异步行为。将包含异步逻辑的服务以同步方式包装是没有意义的。调用者将被阻塞，直到异步过程完成，整个功能将无法扩展。相反，有时使用异步通信来封装长时间运行的同步过程是有意义的。这包括无法更改的外部系统或遗留应用程序。客户端组件将在单独的线程中连接到系统，允许调用线程立即继续。客户端线程要么在同步过程完成前阻塞，要么使用轮询。然而，最好根据业务需求来建模系统和通信风格。

有很多通信协议和格式可供选择，其中很多是专有的。建议工程师们了解不同的概念和一般的通信方式。通信技术会变化，但数据交换的原则是永恒的。截至撰写本书时，HTTP 是最广泛使用的通信协议。这可以说是需要实现的最重要技术之一，它被广泛理解，并且拥有强大的工具支持。

# 同步 HTTP 通信

今天的企业系统中，大部分同步通信都是通过 HTTP 实现的。企业应用程序暴露出 HTTP 端点，客户端通过这些端点进行访问。这些端点通常以 Web 服务或 Web 前端的形式存在，通过 HTTP 传输 HTML。

Web 服务可以以各种方式设计和指定。在 simplest 的形式中，我们只想通过有线方式调用另一个系统的函数。这个函数需要指定输入和输出值。这些函数或**远程过程调用**（RPC）在这种情况下通过 HTTP 实现，通常使用 XML 格式来指定参数参数。在 J2EE 时代，这类 Web 服务相当普遍。最突出的例子是 SOAP 协议，它通过 JAX-WS 标准实现。然而，SOAP 协议及其 XML 格式使用起来相当繁琐，并且除了 Java 和.NET 之外的其他语言支持不佳。

在今天的系统中，使用其概念和约束的 REST 架构风格被使用得更为频繁。

# 表示状态转移

由 Roy T. Fielding 发起的**表示状态转移**（REST）的理念和约束，提供了一种适合企业应用需求很多方面的 Web 服务架构风格。这些理念导致系统与接口耦合更加松散，接口以统一和直接的方式被各种客户端访问。

REST 的**统一接口**约束要求在基于 Web 的系统中使用 URI 在请求中标识资源。这些资源代表我们的领域实体；例如，用户或文章，它们通过企业应用程序的 URL 单独标识。也就是说，URL 不再代表 RPC 方法，而是实际的领域实体。这些表示以统一的方式修改，在 HTTP 中使用 GET、POST、DELETE、PATCH 或 PUT 等 HTTP 方法。实体可以以客户端请求的不同格式表示，如 XML 或 JSON。如果服务器支持，客户端可以自由选择是否以 XML 或 JSON 表示访问特定的用户。

统一接口约束的另一个方面是利用超媒体作为应用程序状态的动力。超媒体意味着使用超链接将相关资源链接在一起。传输到客户端的 REST 资源可以包括具有语义链接关系的其他资源的链接。如果某个用户包含有关其经理的信息，则可以使用指向第二个用户（经理）资源的链接来序列化这些信息。

以下是一个包含在 JSON 响应中的超媒体链接的书籍表示示例：

```java
{
     "name": "Java",
     "author": "Duke",
     "isbn": "123-2-34-456789-0",
     "_links": {
         "self": "https://api.example.com/books/12345",
         "author": "https://api.example.com/authors/2345",
         "related-books": "https://api.example.com/books/12345/related"
    }
}
```

在为人类设计的网站上，这些链接是主要方面之一。在超媒体 API 中，这些链接被 REST 客户端用于在 API 中导航。可发现性的概念减少了涉及系统的耦合度，并增加了其可扩展性。如果完全接受这一概念，客户端只需知道 API 的入口点，并使用语义链接关系（如`related-books`）发现可用资源。他们将遵循已知的联系，使用提供的 URL。

在大多数 REST API 中，仅让客户端通过 HTTP GET 方法跟踪链接并获取资源表示是不够的。信息是通过改变状态的方法（如 POST 或 PUT）和包含有效载荷的请求体来交换的。超媒体也支持这些所谓的动作，使用超媒体控制。动作不仅描述目标 URL，还描述 HTTP 方法和发送所需的信息。

以下展示了使用动作概念的一个更复杂的超媒体示例。此示例展示了 Siren 内容类型，旨在让您了解超媒体响应的潜在内容：

```java
{
    "class": [ "book" ],
    "properties": {
        "isbn": "123-2-34-456789-0",
        "name": "Java",
        "author": "Duke",
        "availability": "IN_STOCK",
        "price": 29.99
    }
    "actions": [
        {
            "name": "add-to-cart",
            "title": "Add Book to cart",
            "method": "POST",
            "href": "http://api.example.com/shopping-cart",
            "type": "application/json",
            "fields": [
                { "name": "isbn", "type": "text" },
                { "name": "quantity", "type": "number" }
            ]
        }
    ],
    "links": [
        { "rel": [ "self" ], "href": "http://api.example.com/books/1234" }
    ]
}
```

这是启用超媒体控制的内容类型的一个示例。在撰写本书时，尚无任何超媒体启用的内容类型，如 Siren、HAL 或 JSON-LD 成为标准或事实上的标准。然而，此 Siren 内容类型应足以传达链接和动作的概念。

使用超媒体将客户端与服务器解耦。首先，URL 的责任完全在服务器端。客户端不能对 URL 的创建方式做出任何假设；例如，认为书籍资源位于`/books/1234`下，这是由路径`/books/`加上书籍 ID 构成的。我们在现实世界的项目中已经看到了许多将这些假设重复到客户端的 URL 逻辑。

解耦的下一个方面是服务器上状态的变化方式。例如，客户端需要向`/shopping-cart`发送包含特定 JSON 结构的 JSON 内容类型的 POST 指令不再内置于客户端，而是动态检索。客户端将仅通过其关系或名称（此处为`add-to-cart`）以及操作中提供的信息来引用超媒体操作。通过使用这种方法，客户端只需要知道“添加到购物车”操作的业务含义以及所需 ISBN 和数量字段的来源。这无疑是客户端逻辑。字段值可以从资源表示本身或从客户端过程中检索。例如，书籍的数量可以以 UI 中的下拉字段的形式呈现。

使用超媒体的一个潜在优势是将业务逻辑与客户端解耦。通过使用链接和操作来引导客户端访问可用资源，可用链接和操作中包含的信息被隐式地告知客户端，在当前系统状态下哪些用例是可能的。例如，假设只有具有特定可用性的书籍才能添加到购物车中。实现此行为的客户端，即仅在这些情况下显示“添加到购物车”按钮的客户端，需要了解此逻辑。客户端功能随后将检查书籍的可用性是否符合标准，等等。从技术上讲，这种业务逻辑应仅位于服务器端。通过动态提供链接和操作到可用资源，服务器规定在当前状态下哪些功能是可能的。因此，“添加到购物车”操作只有在书籍实际上可以添加到购物车时才会被包含。因此，客户端逻辑简化为检查是否包含具有已知关系或名称的链接和操作。因此，客户端只有在响应中提供了相应的操作时，才会显示一个活动的“添加到购物车”按钮。

随着 Java EE 的出现，REST 架构风格越来越受到关注。虽然大多数现有的 Web 服务并没有实现 REST 架构风格定义的所有约束，特别是超媒体，但它们通常被认为是 REST 服务。

关于 REST 约束的更多信息，我建议您参考 Roy T. Fielding 的论文《Architectural Styles and the Design of Network-based Software Architectures》。

# Java API for RESTful web services

在 Java EE 中，**Java API for RESTful web services**（JAX-RS）用于定义和访问 REST 服务。JAX-RS 在 Java 生态系统中被广泛使用，甚至被其他企业技术所使用。开发者特别喜欢声明式开发模型，这使得以高效的方式开发 REST 服务变得容易。

所说的 JAX-RS 资源指定了在特定 URL 下可用的 REST 资源。JAX-RS 资源是资源类中的方法，在通过特定 HTTP 方法访问 URL 时实现业务逻辑。以下是一个用户 JAX-RS 资源类的示例：

```java
import javax.ws.rs.Path;
import javax.ws.rs.GET;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

@Path("users")
@Produces(MediaType.APPLICATION_JSON)
public class UsersResource {

    @Inject
    UserStore userStore;

    @GET
    public List<User> getUsers() {
        return userStore.getUsers();
    }
}
```

`getUsers()` 方法是 JAX-RS 资源方法，当客户端执行 HTTP 调用 `GET .../users` 时，容器将调用此方法。随后，用户列表以 JSON 格式返回给客户端，即包含每个用户 JSON 对象的 JSON 数组。这是通过 `@Produces` 注解指定的，在这里将隐式使用 **Java API for JSON Binding** (**JSON-B**) 将 Java 类型映射到其相应的 JSON 表示形式。

在这里，你可以看到控制反转原则的应用。我们不需要自己连接或注册 URL，使用 `@Path` 注解的声明就足够了。对于将 Java 类型映射到表示形式，如 JSON，也是如此。我们以声明的方式指定我们想要提供的表示格式。其余的由容器处理。JAX-RS 实现还负责所需的 HTTP 通信。通过返回一个对象，这里是指用户列表，JAX-RS 隐式假设 HTTP 状态码 `200 OK`，并将其与我们的 JSON 表示形式一起返回给客户端。

为了将 JAX-RS 资源注册到容器中，应用程序可以发送 `Application` 的子类，该子类启动 JAX-RS 运行时。使用 `@ApplicationPath` 注解自动将提供的路径注册为 Servlet。以下是一个 JAX-RS 配置类的示例，这对于大多数用例来说是足够的：

```java
import javax.ws.rs.ApplicationPath;
import javax.ws.rs.core.Application;

@ApplicationPath("resources")
public class JAXRSConfiguration extends Application {
    // no configuration required
}
```

JAX-RS，以及 Java EE 伞下的其他标准，都使用了约定优于配置的原则。此 REST 资源默认行为对于大多数用例来说是合理的。如果不满足要求，则可以通过自定义逻辑覆盖默认行为。这也是为什么 JAX-RS 等框架提供了高效的编程模型。默认情况可以非常快速地实现，同时还有进一步扩展的选项。

让我们来看一个更全面的例子。假设我们想要在由客户端使用我们的 REST 服务提供的系统中创建一个新用户。遵循 HTTP 语义，这个操作将是对用户资源的 POST 请求，因为我们正在创建一个可能尚未被识别的新资源。POST 方法和 PUT 方法之间的区别在于，后者是全能的，只更改由提供的表示形式访问的资源，而 POST 将创建以新 URL 形式的新资源。这里就是这种情况。我们正在创建一个新用户，该用户将可以通过一个新生成的 URL 来识别。如果新用户的资源被创建，客户端应被引导到该 URL。对于创建资源，这通常通过`201 Created`状态码来实现，该状态码表示已成功创建新资源，以及包含资源所在 URL 的`Location`头。

为了满足这一要求，我们必须在我们的 JAX-RS 资源中提供更多信息。以下是如何在`createUser()`方法中实现这一点的示例：

```java
import javax.ws.rs.Consumes;
import javax.ws.rs.PathParam;
import javax.ws.rs.POST;
import javax.ws.rs.core.Context;
import javax.ws.rs.core.Response;
import javax.ws.rs.core.UriInfo;

@Path("users")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class UsersResource {

    @Inject
    UserStore userStore;

    @Context
    UriInfo uriInfo;

    @GET
    public List<User> getUsers() {
        return userStore.getUsers();
    }

    @GET
    @Path("{id}")
    public User getUser(@PathParam("id") long id) {
        return userStore.getUser(id);
    }

    @POST
    public Response createUser(User user) {
        long id = userStore.create(user);

        URI userUri = uriInfo.getBaseUriBuilder()
                .path(UsersResource.class)
                .path(UsersResource.class, "getUser")
                .build(id);

        return Response.created(userUri).build();
    }

}
```

我们利用了 JAX-RS 中包含的`UriInfo`功能，这样我们就不需要在构建新 URL 时重复自己。该功能使用资源类注解中已经存在的路径信息。`Response`方法用于通过构建模式方法指定实际的 HTTP 响应。JAX-RS 注意到我们方法的返回类型现在是一个响应规范，并将相应地响应用户。通过这种方法，我们对客户端的响应有完全的控制和灵活性。

如您所见，这些方法是我们的业务用例的入口点。我们注入了`UserStore`边界，在我们的情况下，它被实现为 EJB，提供返回用户列表和创建新用户的逻辑。

JAX-RS 提供了一种高效且直接的方法来通过 RESTful Web 服务公开业务功能。如果默认行为足够，开发者不需要编写任何低级 HTTP *管道*代码。

# 映射 HTTP 内容类型

为了尽可能给开发者提供生产力，Java EE 包括将 POJO 透明映射到 JSON 或 XML 的标准。您看到的 JAX-RS 示例隐式使用了 JSON-B 将我们的`User`类型映射到 JSON 对象和数组。

这再次使用了约定优于配置的原则。如果没有其他指定，JSON-B 假定将 POJO 属性直接映射为 JSON 对象键值对。用户的`id`也出现在 JSON 输出中。

对于 **Java Architecture for XML Binding**（**JAXB**）及其 XML 绑定，这种情况同样适用，它比 JSON-B 更早地包含在 Java EE 中。这两个标准都支持使用放置在映射的 Java 类型上的注解的声明性配置方法。如果我们即将更改类型的 JSON 表示，我们可以注释相应的字段：

```java
import javax.json.bind.annotation.JsonbProperty;
import javax.json.bind.annotation.JsonbTransient;

public class User {

    @JsonbTransient
    private long id;

    @JsonbProperty("username")
    private String name;

    ...
}
```

如果我们想要实现更复杂的资源映射，例如在之前展示的超媒体书籍示例中，我们可以使用声明性映射方法来实现。例如，要将链接映射到书籍资源，我们可以使用包含链接和链接关系的映射：

```java
public class Book {

    @JsonbTransient
    private long id;

    private String name;
    private String author;
    private String isbn;

    @JsonbProperty("_links")
    private Map<String, URI> links;

    ...
}
```

这些链接已在 JAX-RS 资源中适当设置：

```java
@Path("books")
@Produces(MediaType.APPLICATION_JSON)
public class BooksResource {

    @Inject
    BookStore bookStore;

    @Context
    UriInfo uriInfo;

    @GET
    public List<Book> getBooks() {
        List<Book> books = bookStore.getBooks();
        books.forEach(this::addLinks);
        return books;
    }

    @GET
    @Path("{id}")
    public Book getBook(@PathParam("id") long id) {
        Book book = bookStore.getBook(id);
        addLinks(book);
        return book;
    }

    private void addLinks(Book book) {
        URI selfUri = uriInfo.getBaseUriBuilder()
                .path(BooksResource.class)
                .path(BooksResource.class, "getBook")
                .build(book.getId());

        book.getLinks().put("self", selfUri);
        // other links
    }
}
```

书籍列表的输出将类似于以下内容：

```java
[
    {
         "name": "Java",
         "author": "Duke",
         "isbn": "123-2-34-456789-0",
         "_links": {
             "self": "https://api.example.com/books/12345",
             "author": "https://api.example.com/authors/2345",
             "related-books": "https://api.example.com/books/12345/related"
        }
    },
    ...
]
```

使用这种方法，我们现在可以以编程方式引入客户端内使用和跟踪的关系链接。然而，使用超媒体方法很快就会达到这样一个点，即声明性映射给模型带来了太多的开销。链接和关系的映射已经不再是业务领域的一部分，而是一种技术必要性，因此应该受到质疑。我们可以引入传输对象类型，将技术映射与领域模型分开。但这肯定会引入大量的重复，并使我们的项目充斥着许多对业务无价值的类。

另一个需要面对的挑战是超媒体所需的灵活性。即使是使用超媒体控制的简单示例，我们也希望根据系统的当前状态指定和包含链接和操作。超媒体的本质就是控制客户端的流程并将它们引导到某些资源。例如，如果一本书有库存或账户上有一定的信用额度，客户端响应应仅包括下订单的操作。这要求响应映射可以根据需求进行更改。由于声明性映射在运行时无法轻易更改，我们需要一个更灵活的方法。

自从 Java EE 7 以来，就有 **Java API for JSON Processing**（**JSON-P**）标准，它以类似构建器模式的方式提供程序化映射 JSON 结构。我们可以简单地调用构建器类型 `JsonObjectBuilder` 或 `JsonArrayBuilder` 来创建任意复杂的结构：

```java
import javax.json.Json;
import javax.json.JsonObject;
...

JsonObject object = Json.createObjectBuilder()
    .add("hello", Json.createArrayBuilder()
        .add("hello")
        .build())
    .add("key", "value")
    .build();
```

生成的 JSON 对象如下所示：

```java
{
    "hello": [
        "hello"
    ],
    "key": "value"
}
```

尤其是在需要大量灵活性的情况下，例如在超媒体中，这种方法非常有帮助。JSON-P 标准，以及 JSON-B 或 JAXB，与 JAX-RS 无缝集成。返回 JSON-P 类型（如 `JsonObject`）的 JAX-RS 资源方法将自动返回相应的响应内容类型。无需进一步配置。让我们看看包含资源链接的示例是如何使用 JSON-P 实现的。

```java
import javax.json.JsonArray;
import javax.json.stream.JsonCollectors;

@Path("books")
public class BooksResource {

    @Inject
    BookStore bookStore;

    @Context
    UriInfo uriInfo;

    @GET
    public JsonArray getBooks() {
        return bookStore.getBooks().stream()
                .map(this::buildBookJson)
                .collect(JsonCollectors.toJsonArray());
    }

    @GET
    @Path("{id}")
    public JsonObject getBook(@PathParam("id") long id) {
        Book book = bookStore.getBook(id);
        return buildBookJson(book);
    }

    private JsonObject buildBookJson(Book book) {
        URI selfUri = uriInfo.getBaseUriBuilder()
                .path(BooksResource.class)
                .path(BooksResource.class, "getBook")
                .build(book.getId());

        URI authorUri = ...

        return Json.createObjectBuilder()
                .add("name", book.getName())
                .add("author", book.getName())
                .add("isbn", book.getName())
                .add("_links", Json.createObjectBuilder()
                        .add("self", selfUri.toString())
                        .add("author", authorUri.toString()))
                .build();
    }
}
```

JSON-P 对象是通过使用构建器模式方法动态创建的。我们对所需的输出有完全的灵活性。如果通信需要一个与当前模型不同的实体表示，这种方法使用 JSON-P 也是可取的。在过去，项目总是引入传输对象或 DTO 来实现这个目的。在这里，JSON-P 对象实际上是传输对象。通过使用这种方法，我们消除了需要另一个类来复制模型实体大多数结构的需要。

然而，这个例子中也有一些重复。现在，结果 JSON 对象的属性名由字符串提供。为了稍微重构这个例子，我们会引入一个单一责任点，例如一个负责从模型实体创建 JSON-P 对象的托管豆。

例如，这个`EntityBuilder`豆将被注入到这个和其他 JAX-RS 资源类中。然后重复仍然存在，但封装在单一责任点中，并从多个资源类中重用。以下代码显示了用于书籍和可能映射到 JSON 的其他对象的示例`EntityBuilder`。

```java
public class EntityBuilder {

    public JsonObject buildForBook(Book book, URI selfUri) {
        return Json.createObjectBuilder()
                ...
    }
}
```

如果某些端点或外部系统的表示与我们的模型不同，没有其他缺点的情况下，我们无法完全避免重复。通过使用这种方法，我们将映射逻辑从模型中解耦，并具有完全的灵活性。POJO 属性的映射发生在构建器模式调用中。与引入单独的传输对象类并在另一个功能中映射它们相比，这导致更少的混淆，最终代码更少。

让我们再次使用*添加到购物车* Siren 动作来举例说明超媒体。这个例子给出了超媒体响应潜在内容的想法。对于这样的响应，输出需要是动态和灵活的，取决于应用程序的状态。现在我们可以想象程序化映射方法（如 JSON-P）的灵活性和强度。这种输出实际上不适用于声明性 POJO 映射，这会引入一个相当复杂的对象图。在 Java EE 中，建议使用 JSON-P 或第三方依赖项来实现所需的内容类型，以实现单一职责。

对于将 Java 对象映射到 JSON 或 XML 有效载荷，JAXB、JSON-B 和 JSON-P 提供了与其他 Java EE 标准的无缝集成，例如 JAX-RS。除了我们刚刚看到的 JAX-RS 集成之外，我们还可以集成 CDI 注入；这种互操作性对所有现代 Java EE 标准都适用。

JSON-B 类型适配器能够映射 JSON-B 所不知道的自定义 Java 类型。它们将自定义 Java 类型转换为已知和可映射的类型。一个典型的例子是将对象引用序列化为标识符：

```java
import javax.json.bind.annotation.JsonbTypeAdapter;

public class Employee {

    @JsonbTransient
    private long id;
    private String name;
    private String email;

    @JsonbTypeAdapter(value = OrganizationTypeAdapter.class)
    private Organization organization;

    ...
}
```

在`organization`字段上指定的类型适配器用于将引用表示为组织的 ID。为了解析该引用，我们需要查找有效的组织。这个功能可以简单地注入到 JSON-B 类型适配器中：

```java
import javax.json.bind.adapter.JsonbAdapter;

public class OrganizationTypeAdapter implements JsonbAdapter<Organization, String> {

    @Inject
    OrganizationStore organizationStore;

    @Override
    public String adaptToJson(Organization organization) {
        return String.valueOf(organization.getId());
    }

    @Override
    public Organization adaptFromJson(String string) {
        long id = Long.parseLong(string);
        Organization organization = organizationStore.getOrganization(id);

        if (organization == null)
            throw new IllegalArgumentException("Could not find organization for ID " + string);

        return organization;
    }
}
```

此示例已经展示了拥有几个相互协作的标准的好处。开发者可以简单地使用和集成这些功能，而无需花费时间在配置和*管道*上。

# 验证请求

JAX-RS 为我们系统提供了 HTTP 端点的集成。这包括将请求和响应映射到我们应用程序的 Java 类型。然而，为了防止系统被滥用，客户端请求需要进行验证。

**Bean Validation**标准提供了各种类型的验证。其想法是将验证约束，如*此字段不能为 null*、*此整数不能为负*或*此加薪必须符合公司政策*，声明为 Java 类型和属性。该标准已经包含了通常所需的技术驱动约束。可以添加自定义约束，特别是那些由业务功能或验证驱动的约束。这不仅从技术角度，而且从领域角度来看都很有趣。可以使用此标准实现由领域驱动的验证逻辑。

通过注解方法参数、返回类型或属性为`@Valid`来激活验证。虽然验证可以在应用程序的许多地方应用，但对于端点来说尤为重要。将`@Valid`注解到 JAX-RS 资源方法参数上会验证请求体或参数。如果验证失败，JAX-RS 会自动以表示客户端错误的 HTTP 状态码响应 HTTP 请求。

以下演示了用户验证的集成：

```java
import javax.validation.Valid;
import javax.validation.constraints.NotNull;

@Path("users")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class UsersResource {

    ...

    @POST
    public Response createUser(@Valid @NotNull User user) {
        ...
    }
}
```

用户类型被注解了验证约束：

```java
import javax.validation.constraints.Email;
import javax.validation.constraints.NotBlank;

public class User {

    @JsonbTransient
    private long id;

    @NotBlank
    private String name;

    @Email
    private String email;

    ...
}
```

放置在 JAX-RS 方法上的注解告诉实现，一旦客户端请求到达，就立即验证请求体。请求体必须可用，不能为`null`，并且根据用户类型的配置是有效的。用户的姓名属性限制为不能为空；也就是说，它不应该为`null`或只包含空白字符。用户的电子邮件属性必须符合有效的电子邮件地址格式。这些约束在验证用户对象时生效。

在内部，Bean Validation 中包含的`Validator`用于验证对象。如果验证失败，验证器将抛出`ConstraintViolationException`异常。此验证器功能也可以通过依赖注入以编程方式获得。如果验证失败，JAX-RS 会自动调用验证器并向客户端发送适当的响应。

这个例子在非法 HTTP POST 调用到`/users/`资源时将失败，例如提供没有名称的用户表示。这导致`400 Bad Request`状态码，这是 JAX-RS 对失败的客户端验证的默认行为。

如果客户端需要更多关于请求被拒绝原因的信息，可以扩展默认行为。验证器抛出的违反异常可以映射到带有 JAX-RS 异常映射器功能的 HTTP 响应。异常映射器处理从 JAX-RS 资源方法抛出的异常，并将其转换为适当的客户端响应。以下是一个针对`ConstraintViolationExceptions`的此类`ExceptionMapper`的示例：

```java
import javax.validation.ConstraintViolationException;
import javax.ws.rs.ext.ExceptionMapper;
import javax.ws.rs.ext.Provider;

@Provider
public class ValidationExceptionMapper implements ExceptionMapper<ConstraintViolationException> {

    @Override
    public Response toResponse(ConstraintViolationException exception) {
        Response.ResponseBuilder builder = Response.status(Response.Status.BAD_REQUEST);

        exception.getConstraintViolations()
                .forEach(v -> {
                    builder.header("Error-Description", ...);
                });
        return builder.build();
    }
}
```

异常映射器是 JAX-RS 运行时的提供者。提供者要么在 JAX-RS 基本应用程序类中通过编程方式配置，或者，如这里所示，使用`@Provider`注解以声明方式配置。JAX-RS 运行时会扫描类以查找提供者，并自动应用它们。

异常映射器已注册用于给定的异常类型及其子类型。这里由 JAX-RS 资源方法抛出的所有约束违反异常都将映射到客户端响应，包括导致验证失败的字段的基本描述。违反消息是 Bean Validation 提供的一种功能，提供人类可读的、全局的消息。

如果内置的验证约束不足以进行验证，可以使用自定义验证约束。这对于特定于域的验证规则尤其必要。例如，用户名可能需要基于系统当前状态进行更复杂的验证。在这个例子中，创建新用户时用户名不得被占用。还可以设置格式或允许字符的其他约束，显然：

```java
public class User {

    @JsonbTransient
    private long id;

    @NotBlank
    @UserNameNotTaken
    private String name;

    @Email
    private String email;
    ...
}
```

`@UserNameNotTaken`注解是由我们的应用程序定义的自定义验证约束。验证约束委托给约束验证器，即实际执行验证的类。约束验证器可以访问注解对象，例如本例中的类或字段。自定义功能检查提供的对象是否有效。验证方法可以使用`ConstraintValidatorContext`来控制自定义违反，包括消息和更多信息。

以下显示了自定义约束定义：

```java
import javax.validation.Constraint;
import javax.validation.Payload;

@Constraint(validatedBy = UserNameNotTakenValidator.class)
@Documented
@Retention(RUNTIME)
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE})
public @interface UserNameNotTaken {

    String message() default "";

    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

我们的约束条件由`UserNameNotTakenValidator`类验证：

```java
import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

public class UserNameNotTakenValidator implements ConstraintValidator<UserNameNotTaken, String> {

    @Inject
    UserStore userStore;

    public void initialize(UserNameNotTaken constraint) {
        // nothing to do
    }

    public boolean isValid(String string, ConstraintValidatorContext context) {
        return !userStore.isNameTaken(string);
    }
}
```

与其他标准一样，约束验证器可以使用依赖注入来使用托管豆。这对于需要调用控件的自定义验证逻辑来说通常是非常需要的。在这个例子中，验证器注入了`UserStore`。再次强调，我们可以在 Java EE 的范围内重用不同的标准。

自定义验证约束通常是由业务领域驱动的。将复杂的、组合的验证逻辑封装到这样的自定义约束中是有意义的。当应用这种方法时，它也利用了单一职责原则，将验证逻辑分离成一个单独的验证器，而不是分散在原子约束中。

Bean Validation 为需要为同一类型提供不同验证方式的场景提供了更复杂的功能。因此，使用组的概念将某些约束组合在一起形成组，这些组可以单独进行验证。关于这方面的更多信息，我建议读者参考 Bean Validation 规范。

如前所述，HTTP JSON 负载也可以使用 JSON-P 标准在 JAX-RS 中进行映射。这同样适用于 HTTP 请求体。请求体参数可以作为包含 JSON 结构的 JSON-P 类型提供，这些结构是动态读取的。同样，如果对象结构与模型类型不同或需要更多的灵活性，使用 JSON-P 类型表示请求体也是有意义的。对于这种情况，验证提供的对象尤为重要，因为 JSON-P 结构可以是任意的。为了依赖请求对象上存在某些 JSON 属性，这些对象使用自定义验证约束进行验证。

由于 JSON-P 对象是程序性构建的，并且没有预定义的类型，程序员没有像 Java 类型那样注释字段的方法。因此，在请求体参数上使用自定义验证约束。自定义约束定义了特定请求体的有效 JSON 对象的结构。以下代码展示了在 JAX-RS 资源方法中集成验证的 JSON-P 类型：

```java
@Path("users")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class UsersResource {

    ...

    @POST
    public Response createUser(@Valid @ValidUser JsonObject json) {

        User user = readUser(json);
        long id = userStore.create(user);
        ...
    }

    private User readUser(JsonObject object) {
        ...
    }
}
```

自定义验证约束 `ValidUser` 引用了所使用的约束验证器。由于提供的 JSON-P 对象的结构是任意的，验证器必须检查属性的存在和类型：

```java
@Constraint(validatedBy = ValidUserValidator.class)
@Documented
@Retention(RUNTIME)
@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE})
public @interface ValidUser {

    String message() default "";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

自定义约束验证器也适用于 JSON-P 类型：

```java
public class ValidUserValidator implements ConstraintValidator<ValidUser, JsonObject> {

    public void initialize(ValidUser constraint) {
        // nothing to do
    }

    public boolean isValid(JsonObject json, ConstraintValidatorContext context) {
        ...
    }
}
```

在提供的 JSON-P 对象经过验证后，定义的属性可以安全地提取。这个例子展示了如何在 JAX-RS 方法中集成和验证灵活的、程序化的类型。资源类将请求体提取到领域实体类型中，并使用边界来调用业务用例。

# 映射错误

如我们在前面的例子中所看到的，JAX-RS 提供了将异常映射到自定义响应的能力。这是一个有用的功能，可以在不影响生产代码工作流程的情况下实现透明的自定义错误处理。

处理 EJB 时常见的问题是在任何非 EJB 上下文中访问时，任何抛出的异常都会被包装在`EJBException`中；例如，请求作用域的 JAX-RS 资源。这使得异常处理变得相当繁琐，因为必须解包`EJBException`以检查原因。

通过使用`@ApplicationException`注解自定义异常类型，不会将原因包装：

```java
import javax.ejb.ApplicationException;

@ApplicationException
public class GreetingException extends RuntimeException {

    public GreetingException(String message) {
        super(message);
    }
}
```

调用一个抛出`GreetingException`的 EJB 不会导致包装`EJBException`并直接产生异常类型。然后，应用程序可以定义一个针对实际`GreetingException`类型的 JAX-RS 异常映射器，类似于映射约束违规的映射器。

指定`@ApplicationException(rollback = true)`将导致容器在异常发生时回滚活动事务。

# 访问外部系统

我们已经看到我们的业务领域是如何通过 HTTP 从外部访问的。

为了执行业务逻辑，大多数企业应用还需要访问其他外部系统。外部系统不包括我们应用拥有的数据库。通常，外部系统位于应用域之外。它们存在于另一个边界上下文中。

为了访问外部 HTTP 服务，我们将客户端组件集成到我们的项目中，通常作为一个单独的控制。这个控制类封装了与外部系统通信所需的功能。建议仔细构建接口，不要将领域关注点与通信实现细节混合。这些细节包括潜在的负载映射、通信协议、如果使用 HTTP，则包括 HTTP 信息，以及任何与核心领域无关的其他方面。

JAX-RS 附带了一个复杂的客户端功能，以生产方式访问 HTTP 服务。它为资源类提供了相同类型的映射功能。以下代码表示一个控制，它访问外部系统来订购咖啡豆：

```java
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
import javax.enterprise.context.ApplicationScoped;
import javax.ws.rs.client.*;
import java.util.concurrent.TimeUnit;

@ApplicationScoped
public class CoffeePurchaser {

    private Client client;
    private WebTarget target;

    @PostConstruct
    private void initClient() {
        client = ClientBuilder.newClient();
        target = client.target("http://coffee.example.com/beans/purchases/");
    }

    public OrderId purchaseBeans(BeanType type) {
        // construct purchase payload from type
        Purchase purchase = ...

        BeanOrder beanOrder = target
                .request(MediaType.APPLICATION_JSON_TYPE)
                .post(Entity.json(purchase))
                .readEntity(BeanOrder.class);

        return beanOrder.getId();
    }

    @PreDestroy
    public void closeClient() {
        client.close();
    }
}
```

JAX-RS 客户端由客户端构建器构建并配置，使用 Web 目标来访问 URL。这些目标可以使用 URI 构建器功能进行修改，类似于 JAX-RS 资源中的功能。目标用于构建新的调用，这些调用代表实际的 HTTP 调用。调用可以根据 HTTP 信息进行配置，例如内容类型、头信息，以及映射的 Java 类型的特定细节。

在这个例子中，指向外部 URL 的目标会使用 HTTP POST 方法构建一个新的针对 JSON 内容类型的请求。预期的返回 JSON 结构应可映射到`BeanOrder`对象。客户端执行进一步逻辑以提取必要的信息。

容器关闭时，客户端实例将在`@PreDestroy`方法中正确关闭，以防止资源泄露。

# 消费 HTTP 时的稳定性

然而，这个例子在弹性方面缺少一些方面。在没有进一步考虑的情况下，将其称为客户端控制可能会导致不受欢迎的行为。

客户端请求会阻塞，直到 HTTP 调用成功返回或连接超时。HTTP 连接超时配置取决于 JAX-RS 实现，在某些技术中设置为无限阻塞。对于具有弹性的客户端来说，这显然是不可接受的。连接可能会永远等待，阻塞线程，在最坏的情况下，如果所有可用线程都卡在那个位置，等待它们各自的 HTTP 连接完成，可能会阻塞整个应用程序。为了防止这种情况，我们配置客户端使用自定义连接超时。

超时值取决于应用程序，特别是到外部系统的网络配置。HTTP 超时的合理值各不相同。为了获得合理的超时值，建议收集到外部系统的延迟统计数据。对于负载和网络延迟变化很大的系统，例如在特定季节选择性高利用率的电子商务系统，应考虑变化的性质。

HTTP 连接超时是指建立连接之前允许的最大时间。其值应该较小。HTTP 读取超时指定了读取数据需要等待的时间。其值取决于所消费的外部服务的性质。根据收集到的统计数据，配置读取超时的一个良好起点是计算平均响应时间加上三倍的标准差。我们将在第九章*监控、性能和日志*中介绍性能和服务背压的主题。

下面的示例展示了如何配置 HTTP 连接和读取超时：

```java
@ApplicationScoped
public class CoffeePurchaser {

    ...

    @PostConstruct
    private void initClient() {
        client = ClientBuilder.newBuilder()
                .connectTimeout(100, TimeUnit.MILLISECONDS)
                .readTimeout(2, TimeUnit.SECONDS)
                .build();
        target = client.target("http://coffee.example.com/beans/purchases/");
    }

    ...
}
```

客户端调用可能会导致潜在的错误。外部服务可能会返回意外的状态码、意外的响应或根本不响应。在实现客户端组件时需要考虑这一点。

`readResponse()`客户端调用期望响应为 HTTP 状态码`SUCCESSFUL`家族，并且响应体可以从请求的内容类型映射到给定的 Java 类型。如果出现问题，将抛出`RuntimeException`。运行时异常使工程师能够编写不混淆 try-catch 块的代码，但也要求他们意识到潜在的错误。

客户端方法可以捕获运行时异常，以防止它们被抛向调用域服务。还有另一种更简洁的可能性，即使用拦截器。拦截器提供跨切面功能，这些功能在应用时不需要紧密耦合到装饰功能。例如，当外部系统无法提供合理的响应时，这个客户端方法应该故意返回`null`。

下面的示例演示了一个拦截器，该拦截器拦截方法调用并在发生的异常上应用此行为。此拦截器通过注解 `CoffeePurchaser` 控制的方方法进行集成：

```java
import javax.interceptor.AroundInvoke;
import javax.interceptor.Interceptor;
import javax.interceptor.InvocationContext;

@Interceptor
public class FailureToNullInterceptor {

    @AroundInvoke
    public Object aroundInvoke(InvocationContext context) {
        try {
            return context.proceed();
        } catch (Exception e) {
            ...
            return null;
        }
    }
}
```

`purchaseBean()` 方法被注解为 `@Interceptors(FailureToNullInterceptor.class)`。这激活了针对该方法的横切关注点。

在容错方面，客户端功能可以包括更多逻辑。如果有多个系统可用，客户端可以在不同的系统上重试失败的调用。然后，只有在最后手段的情况下，调用才会失败且没有结果。

在 *横切关注点* 这一主题中，我们将看到如何实现更多的横切关注点。

# 访问超媒体 REST 服务

应用 REST 约束的 HTTP Web 服务，特别是在超媒体方面，需要在客户端侧有更复杂的逻辑。服务指导客户端访问需要以特定方式访问的相应资源。超媒体解耦服务并使 API 功能，如可扩展性和发现成为可能，但也需要在客户端侧有更多动态和逻辑。

之前给出的 Siren 内容类型示例给人留下了服务响应如何指导 REST 客户端进行后续调用的印象。假设客户端检索订单的响应并希望跟随 `add-to-cart` 动作：

```java
{
    ... example as shown before
    ... properties of book resource
    "actions": [
        {
            "name": "add-to-cart",
            "title": "Add Book to cart",
            "method": "POST",
            "href": "http://api.example.com/shopping-cart",
            "type": "application/json",
            "fields": [
                { "name": "isbn", "type": "text" },
                { "name": "quantity", "type": "number" }
            ]
        }
    ],
    "links": ...
}
```

客户端仅耦合于对 *add-to-cart* 动作的业务含义以及如何为 ISBN 和数量提供字段值信息的了解。这当然是需要实现客户端领域逻辑。关于如何使用哪种 HTTP 方法访问后续资源（购物车），以及内容类型现在是动态的而不是嵌入到客户端中的信息。

为了将书籍添加到购物车，客户端首先访问书籍的资源。随后调用 *add-to-cart* 用例，提取指定超媒体动作的信息。所需字段的信息需要通过调用提供。然后客户端访问第二个资源，使用 REST 服务和控制调用提供的信息：

```java
public class BookClient {

    @Inject
    EntityMapper entityMapper;

    public Book retrieveBook(URI uri) {
        Entity book = retrieveEntity(uri);
        return entityMapper.decodeBook(uri, book.getProperties());
    }

    public void addToCart(Book book, int quantity) {
        Entity bookEntity = retrieveEntity(book.getUri());

        JsonObjectBuilder properties = Json.createObjectBuilder();
        properties.add("quantity", quantity);

        Entity entity = entityMapper.encodeBook(book);
        entity.getProperties().forEach(properties::add);

        performAction(bookEntity, "add-to-cart", properties.build());
    }

    private Entity retrieveEntity(URI uri) {
        ...
    }

    private void performAction(Entity entity, String actionName,
            JsonObject properties) {
        ...
    }
}
```

`Entity` 类型封装了超媒体实体类型的信息。`EntityMapper` 负责将内容类型映射到领域模型，反之亦然。在这个例子中，动作所需的所有字段都来自资源的属性以及提供的 `quantity` 参数。为了实现某种动态性，所有实体属性都被添加到一个映射中，并传递给 `performAction()` 方法。根据服务器指定的动作，所需字段从这个映射中提取出来。如果需要更多字段，客户端逻辑显然必须更改。

将访问超媒体服务的逻辑以及将领域模型映射到内容类型封装到单独的委托中，这确实是有意义的。访问 REST 服务的功能也可以合理地由库来替代。

你可能会注意到 URI 现在已经泄露到客户端类的公共接口中。这并非偶然，而是为了在多个用例调用中识别资源。尽管如此，URI 作为资源的通用标识符进入业务领域。由于从技术 ID 创建 URL 的逻辑位于客户端，因此实体资源的整个 URL 成为*标识符*。然而，在设计客户端控件时，工程师应该注意公共接口。特别是，关于与外部系统通信的信息不应泄露到领域。使用超媒体很好地支持了这种做法。所有必需的传输信息都是动态检索和使用的。遵循超媒体响应的导航逻辑位于客户端控件中。

本例旨在让读者了解客户端如何使用超媒体 REST 服务。

# 异步通信和消息

异步通信导致系统之间的耦合更加松散。它通常会增加整体响应性以及开销，并使系统在不可靠的情况下也能正常工作的场景成为可能。在概念或技术层面上，存在许多设计异步通信的方式。异步通信并不意味着在技术层面上不能进行同步调用。业务流程可以以异步方式构建，模拟一个或多个未立即执行或处理同步调用。例如，API 可以提供同步方法来创建长时间运行的过程，稍后经常轮询更新。

在技术层面上，异步通信通常以消息为导向的方式设计，使用消息队列或发布-订阅模式实现。应用程序仅直接与消息队列或代理进行通信，消息不会直接传递给特定的接收者。

让我们来看看实现异步通信的各种方法。

# 异步 HTTP 通信

HTTP 通信的请求响应模型通常涉及同步通信。客户端在服务器上请求资源，并阻塞直到响应被传输。因此，使用 HTTP 的异步通信通常在概念上实现。同步 HTTP 调用可以触发长时间运行的业务流程。外部系统随后可以通过另一种机制通知调用者，或者提供轮询更新的功能。

例如，一个复杂的用户管理系统提供了创建用户的方法。假设用户需要作为更长运行异步业务流程的一部分在外部系统中注册和验证。那么应用程序将提供 HTTP 功能，如`POST /users/`，这启动了创建新用户的流程。然而，调用这个用例并不能保证用户能够成功创建和注册。该 HTTP 端点的响应只会确认尝试创建新用户；例如，通过`202 已接受`状态码。这表示请求已被接受，但并不一定已经完全处理。`Location`头字段可以用来指向客户端可以轮询更新部分完成用户资源的地址。

在技术层面上，HTTP 不仅支持同步调用。在子章节 *服务器发送事件* 中，我们将以服务器发送事件为例，探讨作为一个使用异步消息导向通信的 HTTP 标准的例子。

# 消息导向通信

消息导向通信通过异步发送的消息交换信息，通常使用消息队列或发布-订阅模式实现。它提供了解耦系统的优势，因为应用程序只直接与消息队列或代理进行通信。这种解耦不仅影响对系统和所用技术的依赖，还通过异步消息解耦业务流程，影响通信的本质。

消息队列是消息被发送到其中，然后一次由一个消费者消费的队列。在企业系统中，消息队列通常通过**消息导向中间件**（**MOM**）实现。我们过去经常看到这些 MOM 解决方案，例如 ActiveMQ、RabbitMQ 或 WebSphere MQ 等消息队列系统。

发布-订阅模式描述了订阅主题并接收发送到该主题的消息的消费者。订阅者注册主题并接收由发布者发送的消息。这个概念对于涉及更多对等体的场景具有良好的可扩展性。消息导向中间件通常可以用来利用消息队列和发布-订阅方法的优势。

然而，除了异步通信的一般情况外，面向消息的解决方案也存在某些不足。首先需要注意的方面是消息的可靠投递。生产者以异步的、*发送后即忘*的方式发送消息。工程师必须了解消息投递的定义和支持的语义，即消息是否会被接收*最多一次*、*至少一次*或*恰好一次*。选择支持特定投递语义的技术，尤其是*恰好一次*语义，将对可扩展性和吞吐量产生影响。在第八章*微服务与系统架构*中，我们将详细讨论该主题，当讨论事件驱动应用程序时。

对于 Java EE 应用程序，可以使用**Java 消息服务**（**JMS**）API 来集成面向消息的中间件解决方案。JMS API 支持消息队列和发布-订阅方法。它只定义了接口，并由实际的消息导向中间件解决方案实现。

然而，JMS API 的开发者接受度不高，并且在撰写本文时，可以说在当前系统中使用得并不多。与其他标准相比，编程模型并不那么直接和高效。面向消息通信的另一个趋势是，与传统 MOM 解决方案相比，更轻量级的解决方案越来越受欢迎。截至目前，许多这些面向消息的解决方案都是通过专有 API 集成的。此类解决方案的例子是 Apache Kafka，它利用了消息队列和发布-订阅模型。第八章*微服务与系统架构*展示了 Apache Kafka 作为 MOM 解决方案集成到 Java EE 应用程序中的示例。

# 服务器端发送的事件

**服务器端发送的事件**（**SSE**）是一个异步的、基于 HTTP 的发布-订阅技术的例子。它提供了一个易于使用的单向流式通信协议。客户端可以通过请求一个保持开放连接的 HTTP 资源来注册一个主题。服务器通过这些活跃的 HTTP 连接向连接的客户端发送消息。客户端不能直接进行通信，但只能打开和关闭到流式端点的连接。这种轻量级解决方案适用于需要广播更新的用例，例如社交媒体更新、股票价格或新闻源。

服务器将基于 UTF-8 的文本数据作为内容类型`text/event-stream`推送到之前已注册主题的客户端。以下展示了事件的格式：

```java
data: This is a message

event: namedmessage
data: This message has an event name

id: 10
data: This message has an id which will be sent as
 'last event ID' if the client reconnects
```

由于服务器端发送的事件基于 HTTP，因此它们很容易集成到现有的网络或开发工具中。SSE 原生支持事件 ID 和重新连接。重新连接到流式端点的客户端会提供最后接收的事件 ID，以继续订阅他们离开的地方。

JAX-RS 在服务器端和客户端都支持服务器发送事件。使用 JAX-RS 资源定义 SSE 流端点如下：

```java
import javax.ws.rs.DefaultValue;
import javax.ws.rs.HeaderParam;
import javax.ws.rs.InternalServerErrorException;
import javax.ws.rs.core.HttpHeaders;
import javax.ws.rs.sse.*;

@Path("events-examples")
@Singleton
public class EventsResource {

    @Context
    Sse sse;

    private SseBroadcaster sseBroadcaster;
    private int lastEventId;
    private List<String> messages = new ArrayList<>();

    @PostConstruct
    public void initSse() {
        sseBroadcaster = sse.newBroadcaster();

        sseBroadcaster.onError((o, e) -> {
            ...
        });
    }

    @GET
    @Lock(READ)
    @Produces(MediaType.SERVER_SENT_EVENTS)
    public void itemEvents(@HeaderParam(HttpHeaders.LAST_EVENT_ID_HEADER)
                           @DefaultValue("-1") int lastEventId,
                           @Context SseEventSink eventSink) {

        if (lastEventId >= 0)
            replayLastMessages(lastEventId, eventSink);

        sseBroadcaster.register(eventSink);
    }

    private void replayLastMessages(int lastEventId, SseEventSink eventSink) {
        try {
            for (int i = lastEventId; i < messages.size(); i++) {
                eventSink.send(createEvent(messages.get(i), i + 1));
            }
        } catch (Exception e) {
            throw new InternalServerErrorException("Could not replay messages ", e);
        }
    }

    private OutboundSseEvent createEvent(String message, int id) {
        return sse.newEventBuilder().id(String.valueOf(id)).data(message).build();
    }

    @Lock(WRITE)
    public void onEvent(@Observes DomainEvent domainEvent) {
        String message = domainEvent.getContents();
        messages.add(message);

        OutboundSseEvent event = createEvent(message, ++lastEventId);

        sseBroadcaster.broadcast(event);
    }
}
```

`text/event-stream` 内容类型用于服务器发送事件。已注册的 `SseEventSink` 指示 JAX-RS 保持客户端连接打开，以便通过广播发送未来的事件。SSE 标准定义 `Last-Event-ID` 头控制事件流将从中继续的位置。在这个例子中，服务器将重新发送在客户端断开连接期间发布的消息。

`itemEvents()` 方法实现了流注册，并在需要时立即向该客户端重发缺失的事件。输出注册到客户端后，客户端将与所有其他活跃客户端一起接收使用 `Sse` 创建的未来消息。

我们的 enterprise application 的异步集成是通过观察到的 `DomainEvent` 实现的。每当在应用中的某个地方触发此类 CDI 事件时，活跃的 SSE 客户端将接收到一条消息。

JAX-RS 还支持消费 SSE。`SseEventSource` 提供了一个打开 SSE 端点连接的功能。它注册了一个事件监听器，一旦有消息到达就会被调用：

```java
import java.util.function.Consumer;

public class SseClient {

    private final WebTarget target = ClientBuilder.newClient().target("...");
    private SseEventSource eventSource;

    public void connect(Consumer<String> dataConsumer) {
        eventSource = SseEventSource.target(target).build();

        eventSource.register(
                item -> dataConsumer.accept(item.readData()),
                Throwable::printStackTrace,
                () -> System.out.println("completed"));

        eventSource.open();
    }

    public void disconnect() {
        if (eventSource != null)
            eventSource.close();
    }
}
```

在 `SseEventSource` 成功打开连接后，当前线程将继续。在这种情况下，监听器 `dataConsumer#accept` 将在事件到达时被调用。`SseEventSource` 将处理由 SSE 标准定义的所有必需处理。例如，包括在连接丢失后重新连接和发送 `Last-Event-ID` 头。

客户端也有可能使用更复杂的解决方案，通过手动控制头和重新连接。因此，从常规 Web 目标请求 `SseEventInput` 类型时，使用 `text/event-stream` 内容类型。有关更多信息，请参阅 JAX-RS 规范。

服务器发送事件提供了一个易于使用的单向流解决方案，通过 HTTP 集成到 Java EE 技术中。

# WebSocket

服务器发送事件与支持双向通信的更强大的 **WebSocket** 技术竞争。由 IETF 标准化的 WebSocket 是面向消息的发布/订阅通信的另一个例子。它原本打算用于基于浏览器的应用程序，但也可以用于任何客户端/服务器消息交换。WebSocket 通常使用与 HTTP 端点相同的端口，但使用自己的基于 TCP 的协议。

Java EE 支持 WebSocket，作为 **Java API for WebSocket** 的一部分。它包括服务器端和客户端的支持。

服务器端端点定义的编程模型再次与整体 Java EE 图景相匹配。端点可以使用程序化或声明性、注解驱动的途径定义。后者在端点类上添加注解，类似于 JAX-RS 资源的编程模型：

```java
import javax.websocket.*;
import javax.websocket.server.ServerEndpoint;

@ServerEndpoint(value = "/chat", decoders = ChatMessageDecoder.class, encoders = ChatMessageEncoder.class)
public class ChatServer {

    @Inject
    ChatHandler chatHandler;

    @OnOpen
    public void openSession(Session session) {
        ...
    }

    @OnMessage
    public void onMessage(ChatMessage message, Session session) {
        chatHandler.store(message);
    }

    @OnClose
    public void closeSession(Session session) {
        ...
    }
}
```

服务器端点类的注解方法将在启动会话、到达的消息和关闭的连接上分别被调用。会话代表两个端点之间的对话。

WebSocket 端点可以分别定义解码器和编码器，以便将自定义 Java 类型映射到二进制或纯文本数据，反之亦然。此示例指定了一个用于聊天消息的自定义类型，它使用自定义解码器和编码器进行映射。类似于 JAX-RS，WebSocket 自带对常用可序列化 Java 类型（如字符串）的默认序列化功能。以下代码演示了我们自定义域类型的编码器：

```java
import javax.websocket.EncodeException;
import javax.websocket.Encoder;
import javax.websocket.EndpointConfig;

public class ChatMessageEncoder implements Encoder.Binary<ChatMessage> {

    @Override
    public ByteBuffer encode(ChatMessage object) throws EncodeException {
        ...
    }

    ...
}
```

这些类型对应于 JAX-RS 标准中的`MessageBodyWriter`和`MessageBodyReader`类型。以下显示了相应的消息解码器：

```java
import javax.websocket.DecodeException;
import javax.websocket.Decoder;
import javax.websocket.EndpointConfig;

public class ChatMessageDecoder implements Decoder.Binary<ChatMessage> {

    @Override
    public ChatMessage decode(ByteBuffer bytes) throws DecodeException {
        ...
    }

    ...
}
```

客户端端点与服务器端点定义类似。区别在于只有 WebSocket 服务器会在路径上监听新的连接。

WebSocket API 的客户端功能不仅可以在企业环境中使用，也可以在 Java SE 应用程序中使用。对于客户端的 JAX-RS 也是如此。实现 WebSocket 客户端端点留作读者的练习。

WebSocket 以及服务器端发送的事件提供了高度集成的、面向消息的技术。当然，应用程序选择使用什么高度取决于业务需求、现有环境和通信的性质。

# 连接企业技术

一些需要从应用程序集成的外部企业系统不提供标准接口或 Java API。遗留系统以及组织内部使用的其他系统可能属于此类。**Java EE 连接器架构**（**JCA**）API 可以将这些所谓的**企业信息系统**（**EIS**）集成到 Java EE 应用程序中。EIS 的例子包括交易处理系统、消息系统或专有数据库。

JCA 资源适配器是可以部署的 EE 组件，它们将信息系统集成到应用程序中。它们包括连接、事务、安全或生命周期管理之类的合约。与其他连接技术相比，信息系统可以更好地集成到应用程序中。资源适配器打包为**资源适配器存档**（**RAR**），并且可以使用`javax.resource`包及其子包的功能在应用程序中访问。一些 EIS 供应商为其系统提供资源适配器。有关开发和使用资源适配器的信息，请参阅 JCA 规范。

JCA 为外部信息系统提供了多种集成可能性。然而，该标准并未得到广泛使用，并且企业工程师对其接受度不高。开发资源适配器相当繁琐，JCA API 在开发者中并不知名，公司通常选择以其他方式集成系统。实际上，应该考虑是否将编写资源适配器的努力优先于使用其他集成技术来集成信息系统。其他解决方案包括 Apache Camel 或 Mule ESB 等集成框架。

# 数据库系统

大多数企业应用使用数据库系统作为其持久化存储。数据库是企业系统的核心，包含应用程序的数据。截至目前，数据已经成为最重要的商品之一。公司花费大量时间和精力收集、保护和利用数据。

企业系统中表示状态的方式有多种，然而，关系型数据库仍然是最受欢迎的。其概念和用法被充分理解和集成在企业技术中。

# 集成 RDBMS 系统

**Java 持久化 API**（**JPA**）用于将关系型数据库系统集成到企业应用程序中。与 J2EE 时代的过时方法相比，JPA 与基于领域驱动设计概念的领域模型集成良好。持久化实体不会引入太多开销，也不会对模型设置太多约束。这允许首先构建领域模型，专注于业务方面，然后集成持久化层。

持久化作为处理业务用例的必要部分集成到领域。根据用例的复杂性，持久化功能可以在专用控件中或直接在边界中调用。领域驱动设计定义了仓库的概念，正如之前提到的，它与 JPA 实体管理器的职责相匹配。实体管理器用于获取、管理和持久化实体以及执行查询。其接口被抽象化，目的是以通用方式使用。

在 J2EE 时代，**数据访问对象**（**DAO**）模式被广泛使用。这种模式的动机是为了抽象和封装访问数据的功能。这包括访问的存储系统类型，如关系型数据库管理系统（RDBMS）、面向对象数据库、LDAP 系统或文件。虽然这种推理确实有道理，但在 Java EE 时代，遵循该模式对于大多数用例并非必需。

大多数企业应用程序使用支持 SQL 和 JDBC 的关系数据库。JPA 已经抽象了 RDBMS 系统，因此工程师通常不需要处理供应商特定的细节。将使用的存储系统的性质更改为 RDBMS 以外的任何东西都会影响应用程序的代码。由于 JPA 很好地集成到领域模型中，映射领域实体类型到存储不再需要使用传输对象。直接映射领域实体类型是集成持久性而不产生太多开销的生产性方法。对于简单的用例，例如持久化和检索实体，因此不需要 DAO 方法。然而，对于涉及复杂数据库查询的情况，将此功能封装到单独的控制中是有意义的。这些存储库包含特定实体类型的全部持久性。然而，建议从简单的方法开始，只有在复杂性增加时才将持久性重构为单一责任点。

边界或控制分别获得一个实体管理器来管理实体的持久性。以下展示了如何将实体管理器集成到边界中：

```java
import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;

@Stateless
public class PersonAdministration {

    @PersistenceContext
    EntityManager entityManager;

    public void createPerson(Person person) {
        entityManager.persist(person);
    }

    public void updateAddress(long personId, Address newAddress) {
        Person person = entityManager.find(Person.class, personId);

        if (person == null)
            throw new IllegalArgumentException("Could not find person with ID " + personId);

        person.setAddress(newAddress);
    }
}
```

在创建新人员时进行的`persist()`操作使人员成为一个受管理的实体。一旦事务提交，它就会被添加到数据库中，并且可以通过其分配的 ID 稍后获取。`updateAddress()`方法展示了这一点。通过其 ID 检索人员实体到受管理的实体。实体中的所有更改，例如更改其地址，将在事务提交时同步到数据库中。

# 映射领域模型

如前所述，实体、聚合和值对象通过 JPA 集成，而不对模型引入许多约束。实体以及聚合都表示为 JPA 实体：

```java
import javax.persistence.*;

@Entity
@Table(name = "persons")
public class Person {

    @Id
    @GeneratedValue
    private long id;

    @Basic(optional = false)
    private String name;

    @Embedded
    private Address address;

    ...
}

@Embeddable
public class Address {

    @Basic(optional = false)
    private String streetName;

    @Basic(optional = false)
    private String postalCode;

    @Basic(optional = false)
    private String city;

    ...
}
```

人员类型是一个实体。它需要一个 ID 来识别，这个 ID 将是`persons`表的主键。每个属性都以某种方式映射到数据库中，具体取决于类型的性质和关系。人员的名字是一个简单的基于文本的列。

地址是一个不可识别的值对象。从领域角度来看，我们指的地址是哪一个并不重要，只要值匹配即可。因此，地址不是一个实体，所以不会被映射到 JPA 中。值对象可以通过 JPA 可嵌入类型来实现。这些类型的属性将被映射到引用它们的实体的表中的额外列。由于人员实体包含一个特定的地址值，因此地址属性将是人员表的一部分。

由多个实体组成的根聚合可以通过配置要映射到适当数据库列和表的关系来实现。例如，汽车由引擎、一个或多个座椅、底盘和许多其他部件组成。其中一些是实体，它们可以作为单独的对象被识别和访问。汽车制造商可以识别整个汽车或只是引擎，并相应地进行维修或更换。数据库映射也可以放在现有的域模型之上。

以下代码片段显示了汽车域实体，包括 JPA 映射：

```java
import javax.persistence.CascadeType;
import javax.persistence.OneToMany;
import javax.persistence.OneToOne;

@Entity
@Table(name = "cars")
public class Car {

    @Id
    @GeneratedValue
    private long id;

    @OneToOne(optional = false, cascade = CascadeType.ALL)
    private Engine engine;

    @OneToMany(cascade = CascadeType.ALL)
    private Set<Seat> seats = new HashSet<>();

    ...
}
```

座椅包含在一个集合中。`HashSet`用于新的`Car`实例；应避免使用`null`的 Java 集合。

引擎代表我们域中的另一个实体：

```java
import javax.persistence.EnumType;
import javax.persistence.Enumerated;

@Entity
@Table(name = "engines")
public class Engine {

    @Id
    @GeneratedValue
    private long id;

    @Basic(optional = false)
    @Enumerated(EnumType.STRING)
    private EngineType type;

    private double ccm;

    ...
}
```

汽车座椅也代表实体，可以通过它们的 ID 来识别：

```java
@Entity
@Table(name = "seats")
public class Seat {

    @Id
    @GeneratedValue
    private long id;

    @Basic(optional = false)
    @Enumerated(EnumType.STRING)
    private SeatMaterial material;

    @Basic(optional = false)
    @Enumerated(EnumType.STRING)
    private SeatShape shape;

    ...
}
```

所有实体，无论是从其他实体引用还是独立存在，都需要在持久化上下文中进行管理。如果汽车的引擎被新的实体替换，这也需要单独持久化。持久化操作要么在单个实体上显式调用，要么从对象层次结构中级联。级联是在实体关系上指定的。以下代码展示了从服务中持久化新汽车引擎的两种方法：

```java
public void replaceEngine(long carIdentifier, Engine engine) {
    Car car = entityManager.find(Car.class, carIdentifier);
    car.replaceEngine(engine);

    // car is already managed, engine needs to be persisted
    entityManager.persist(engine);
}
```

在从其标识符加载汽车后，它成为一个管理实体。引擎仍然需要持久化。第一种方法在服务中显式持久化引擎。

第二种方法从汽车聚合中级联合并操作，该操作也处理新实体：

```java
public void replaceEngine(long carIdentifier, Engine engine) {
    Car car = entityManager.find(Car.class, carIdentifier);
    car.replaceEngine(engine);

    // merge operation is applied on the car and all cascading relations
    entityManager.merge(car);
}
```

高度建议采用后一种方法。聚合根负责维护整体状态的整数和一致性。当所有操作都从根实体启动和级联时，完整性可以更可靠地实现。

# 集成数据库系统

实体管理器在持久化上下文中管理持久化实体。它使用单个持久化单元，该单元对应于一个数据库实例。持久化单元包括所有管理的实体、实体管理器和映射配置。如果只访问一个数据库实例，则可以直接获取实体管理器，如前例所示。持久化上下文注解随后引用唯一的持久化单元。

持久化单元在`persistence.xml`描述符文件中指定，该文件位于`META-INF`目录下。在现代 Java EE 中，这是使用基于 XML 的配置的少数几个情况之一。持久化描述符定义了持久化单元和可选配置。数据源仅通过其 JNDI 名称引用，以便将访问数据库实例的配置与应用程序配置分开。数据源的实际配置在应用服务器中指定。如果应用服务器只包含一个使用单个数据库的应用程序，则开发人员可以使用应用服务器的默认数据源。在这种情况下，可以省略数据源名称。

下面的片段显示了包含单个持久化单元使用默认数据源的`persistence.xml`文件示例：

```java
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2" 

        xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence
        http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="vehicle" transaction-type="JTA">
    </persistence-unit>
</persistence>
```

此示例对于大多数企业应用来说已经足够。

下一个片段演示了一个包含多个数据源持久化单元定义的`persistence.xml`文件：

```java
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2" 

        xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence
        http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">

    <persistence-unit name="vehicle" transaction-type="JTA">
        <jta-data-source>jdbc/VehicleDB</jta-data-source>
    </persistence-unit>
    <persistence-unit name="order" transaction-type="JTA">
        <jta-data-source>jdbc/OrderDB</jta-data-source>
    </persistence-unit>
</persistence>
```

注入实体管理器需要通过其名称引用所需的持久化单元。实体管理器始终对应于一个使用单个持久化单元的单个持久化上下文。以下`CarManagement`定义展示了在多个持久化单元环境中的前一个示例：

```java
@Stateless
public class CarManagement {

    @PersistenceContext(unitName = "vehicle")
    EntityManager entityManager;

    public void replaceEngine(long carIdentifier, Engine engine) {
        Car car = entityManager.find(Car.class, carIdentifier);
        car.replaceEngine(engine);

        // merge operation is applied on the car and all cascading relations
        entityManager.merge(car);
    }
}
```

可选地，可以通过使用 CDI 生产者字段简化特定实体管理器的注入。通过使用自定义限定符显式生成实体管理器，可以以类型安全的方式实现注入：

```java
public class EntityManagerExposer {

    @Produces
    @VehicleDB
    @PersistenceContext(unitName = "vehicle")
    private EntityManager vehicleEntityManager;

    @Produces
    @OrderDB
    @PersistenceContext(unitName = "order")
    private EntityManager orderEntityManager;

}
```

现在可以注入生成的实体管理器，现在使用`@Inject`和类型安全的限定符：

```java
public class CarManagement {

    @Inject
    @VehicleDB
    EntityManager entityManager;

    ...
}
```

此方法可以简化在许多位置注入不同实体管理器的环境中的使用。

还有其他将域模型映射到数据库的可能方法。数据库映射也可以在 XML 文件中定义。然而，J2EE 中的过去方法已经表明，使用注解进行声明性配置可以更有效地使用。对域模型进行注解也提供了更好的概览。

# 事务

持久化操作需要在事务上下文中执行。被修改的管理实体在事务提交时间同步到数据源。因此，事务跨越修改操作，通常是整个业务用例。

如果边界实现为 EJB，则在默认情况下，业务方法执行期间会激活一个事务。这与 JPA 持久化在应用程序中涉及的典型场景相匹配。

通过 CDI 管理豆，使用`@Transactional`注解其方法，可以实现相同的行为。事务边界在业务方法进入时指定特定的行为。默认情况下，此行为定义了一个事务是`REQUIRED`；也就是说，如果调用上下文已经在活动事务中执行，则创建或重用事务。

`REQUIRES_NEW`行为将始终启动一个新的事务，该事务独立执行，并在方法和新事务完成后恢复潜在的前一个事务。这对于处理大量数据且可以分几个独立事务处理的长运行业务流程非常有用。

其他事务行为也是可能的，例如强制执行一个已经激活的事务或完全不支持事务。这可以通过在业务方法上添加`@Transactional`注解来配置。EJB 隐式定义了`REQUIRED`事务。

RDBMS 系统很好地集成到 Java EE 应用程序中。遵循约定优于配置的原则，典型的用例以生产方式实现。

# 关系型数据库与 NoSQL

在过去几年里，数据库技术发生了许多变化，尤其是在分布式方面。然而，传统的数据库关系型数据库仍然是今天最常用的选择。它们最显著的特征是基于表的数据库模式和事务行为。

**NoSQL**（非 SQL 或不仅限于 SQL）数据库系统提供的数据形式不同于关系型表格。这些形式包括文档存储、键值存储、列存储和图数据库。大多数 NoSQL 数据库在可用性、可扩展性和网络分区容错性方面做出了妥协，以牺牲一致性。NoSQL 背后的理念是不使用关系型表格结构的完全支持，**ACID**事务（**原子性、一致性、隔离性、持久性**）、外键以及表连接，以支持水平扩展。这回到了著名的 CAP 定理。**CAP**定理（**一致性、可用性、分区容错性**）声称，分布式数据存储无法保证最多两个指定的约束。由于分布式网络不可靠（分区容错性），系统基本上可以选择是否保证一致性或水平扩展。大多数 NoSQL 数据库选择可扩展性而不是一致性。在选择数据存储技术时，需要考虑这一事实。

NoSQL 系统的原因在于关系型数据库的不足。最大的问题是支持 ACID 的关系型数据库在水平扩展方面表现不佳。数据库系统是企业系统的核心，通常由多个应用服务器访问。需要一致更新数据需要在中央位置同步。这种同步发生在业务用例的技术事务中。需要复制并保持一致性的数据库系统需要在彼此之间维护分布式事务。然而，分布式事务无法扩展，并且可能在每个解决方案中都不可靠。

尽管如此，关系型数据库系统对于大多数企业应用来说已经足够扩展。如果水平扩展成为问题，以至于集中式数据库不再是可行的选择，一个解决方案是使用事件驱动架构等方法来分割持久化。我们将在第八章“微服务和系统架构”中详细讨论这个主题。

NoSQL 数据库也有一些缺点，特别是在事务行为方面。数据是否需要以事务方式持久化，很大程度上取决于应用程序的业务需求。经验表明，在几乎所有的企业系统中，至少有一部分持久化需要可靠性；即事务。然而，有时会有不同类型的数据。某些领域模型更为关键，需要事务处理，而其他数据可能可以重新计算或重新生成；例如，统计数据、推荐或缓存数据。对于后一种类型的数据，NoSQL 数据存储可能是一个不错的选择。

在撰写本文时，还没有出现任何 NoSQL 系统成为标准或事实标准。许多系统在概念和用法上也有很大的差异。Java EE 8 中也没有包含针对 NoSQL 的标准。

因此，访问 NoSQL 系统通常是通过使用供应商提供的 Java API 实现的。这些 API 使用了更低级别的标准，如 JDBC 或它们的专有 API。

# 横切关注点

企业应用需要一些技术驱动的横切关注点。这些关注点的例子包括事务、日志记录、缓存、弹性、监控、安全和其他非功能性需求。即使是仅针对业务的系统，用例也需要一定数量的*技术管道*。

我们刚才在处理事务时看到了一个非功能性横切关注点的例子。Java EE 不需要工程师花费太多时间和精力来集成事务行为。对于其它横切关注点也是如此。

Java EE 拦截器是横切关注点的典型例子。遵循面向切面编程的概念，横切关注点的实现与被装饰的功能分离。管理 Bean 的方法可以被装饰以定义拦截器，这些拦截器会中断执行并执行所需的任务。拦截器对被拦截方法的执行拥有完全控制权，包括返回值和抛出的异常。为了与其它 API 的理念相匹配，拦截器以轻量级的方式集成，不对被装饰功能设置太多约束。

之前在 HTTP 客户端类中透明处理错误的例子展示了拦截器的使用。业务方法也可以使用自定义拦截器绑定进行装饰。以下演示了一个通过自定义注解实现的业务驱动流程跟踪切面：

```java
@Stateless
public class CarManufacturer {

    ...

    @Tracked(ProcessTracker.Category.MANUFACTURER)
    public Car manufactureCar(Specification spec) {
        ...
    }
}
```

`Tracked` 注解定义了一个所谓的拦截器绑定。注解参数代表一个非绑定值，用于配置拦截器：

```java
import javax.enterprise.util.Nonbinding;
import javax.interceptor.InterceptorBinding;

@InterceptorBinding
@Inherited
@Documented
@Target({TYPE, METHOD})
@Retention(RUNTIME)
public @interface Tracked {

    @Nonbinding
    ProcessTracker.Category value();
}
```

拦截器通过绑定注解来激活：

```java
import javax.annotation.Priority;

@Tracked(ProcessTracker.Category.UNUSED)
@Interceptor
@Priority(Interceptor.Priority.APPLICATION)
public class TrackingInterceptor {

    @Inject
    ProcessTracker processTracker;

    @AroundInvoke
    public Object aroundInvoke(InvocationContext context) throws Exception {
        Tracked tracked = resolveAnnotation(context);

        if (tracked != null) {
            ProcessTracker.Category category = tracked.value();
            processTracker.track(category);
        }

        return context.proceed();
    }

    private Tracked resolveAnnotation(InvocationContext context) {
        Function<AnnotatedElement, Tracked> extractor = c -> c.getAnnotation(Tracked.class);
        Method method = context.getMethod();

        Tracked tracked = extractor.apply(method);
        return tracked != null ? tracked : extractor.apply(method.getDeclaringClass());
    }
}
```

默认情况下，通过拦截器绑定绑定的拦截器是禁用的。拦截器必须通过指定优先级（如本例中所示）来显式启用，或者可以在 `beans.xml` 描述符中激活它。

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans 

        xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
        http://xmlns.jcp.org/xml/ns/javaee/beans_1_1.xsd"
        bean-discovery-mode="all">
    <interceptors>
        <class>com.example.cars.processes.TrackingInterceptor</class>
    </interceptors>
</beans>
```

拦截器可以使用反射来检索潜在的注解参数，例如示例中的流程跟踪类别。拦截器绑定可以放置在方法或类级别。

拦截器在不紧密耦合的情况下装饰方法上的行为。它们特别适用于需要将横切关注点添加到大量功能中的场景。

拦截器类似于 CDI 装饰器。这两个概念都是通过封装在另一个地方的行为来装饰托管 Bean。区别在于装饰器主要用于装饰业务逻辑，而业务逻辑通常也只针对装饰的 Bean。然而，拦截器主要用于技术问题。它们提供了更广泛的使用，使得可以注解所有类型的 Bean。这两个概念都是实现横切关注点的有用功能。

# 配置应用程序

应用程序行为不能硬编码，但需要动态定义，可以通过配置来实现。配置的实现方式取决于应用程序和动态行为的性质。

需要配置哪些方面？是否足够定义已包含在已发布工件中的配置文件？是否需要从外部配置打包的应用程序？是否需要在运行时更改行为？

在应用程序构建后不需要更改的配置可以轻松地在项目中实现，即在源代码中。假设我们需要更多的灵活性。

在 Java 环境中，最直接的方法是提供包含配置值键值对的属性文件。需要检索配置值以便在代码中使用。当然，可以编写程序提供属性值的 Java 组件。在 Java EE 环境中，将使用依赖注入来检索此类组件。在撰写本文时，尚无 Java EE 标准支持开箱即用的配置。然而，使用 CDI 功能可以在几行代码中提供此功能。以下是一个可能的解决方案，它允许注入通过键标识的配置值：

```java
@Stateless
public class CarManufacturer {

    @Inject
    @Config("car.default.color")
    String defaultColor;

    public Car manufactureCar(Specification spec) {
        // use defaultColor
    }
}
```

为了明确地注入配置值，例如作为字符串提供的值，需要像 `@Config` 这样的限定符。这个自定义限定符在我们的应用程序中定义。目标是注入通过提供的键标识的值：

```java
@Qualifier
@Documented
@Retention(RUNTIME)
public @interface Config {

    @Nonbinding
    String value();
}
```

CDI 生产者负责检索和提供特定的配置值：

```java
import javax.enterprise.inject.spi.InjectionPoint;
import java.io.*;
import java.util.Properties;

@ApplicationScoped
public class ConfigurationExposer {

    private final Properties properties = new Properties();

    @PostConstruct
    private void initProperties() {
        try (InputStream inputStream = ConfigurationExposer.class
                .getResourceAsStream("/application.properties")) {
            properties.load(inputStream);
        } catch (IOException e) {
            throw new IllegalStateException("Could not init configuration", e);
        }
    }

    @Produces
    @Config("")
    public String exposeConfig(InjectionPoint injectionPoint) {
        Config config = injectionPoint.getAnnotated().getAnnotation(Config.class);
        if (config != null)
            return properties.getProperty(config.value());
        return null;
    }
}
```

`@Config` 注解中的引用键是一个非绑定属性，因为所有注入的值都由我们的 CDI 生产者方法处理。CDI 提供的 `InjectionPoint` 包含有关依赖注入指定位置的信息。生产者检索带有实际配置键的注解，并使用它来查找配置属性。期望属性文件 `application.properties` 存在于类路径中。这种方法包括在运行时需要可用的配置值。由于属性映射只初始化一次，因此加载后值将不会改变。配置暴露 Bean 是应用程序范围的，只将所需值加载到属性映射中一次。

如果场景需要运行时更改配置，则生产者方法必须重新加载配置文件。生产者方法的范围定义了配置值的生命周期，以及该方法将被调用的频率。

此示例使用纯 Java EE 实现配置。有一些第三方 CDI 扩展提供了类似的功能，以及更复杂的功能。在撰写本文时，此类解决方案的一个常用示例是 Apache Deltaspike。

除了企业技术之外，还需要考虑容器运行的环境；特别是，因为容器技术对运行时环境施加了某些限制。第五章，*Java EE 的容器和云环境*涵盖了现代环境及其对 Java EE 运行时的影响，包括如何设计动态配置。

CDI 生产者的强大之处在于它们的灵活性。任何配置源都可以轻松附加以暴露配置值。

# 缓存

缓存是一种技术驱动的横切关注点，一旦应用程序面临性能问题，例如缓慢的外部系统、昂贵且可缓存的计算或大量数据，它就会变得有趣。一般来说，缓存旨在通过在可能更快的缓存中存储难以检索的数据来降低响应时间。一个典型的例子是将外部系统或数据库的响应保留在内存中。

在实现缓存之前，需要提出的一个问题是是否需要缓存，甚至是否可能。有些数据不适合缓存，例如需要按需计算的数据。如果情况和数据可能适合缓存，那么是否可能采用除了缓存之外的另一种解决方案，这取决于具体情况。缓存会引入重复和接收过时信息的机会，一般来说，对于大多数企业应用程序，应该避免使用缓存。例如，如果数据库操作太慢，建议考虑是否可以通过其他方式，如索引，来帮助。

这在很大程度上取决于具体情况和所需的缓存解决方案。一般来说，在应用程序内存中直接缓存已经解决了许多场景。

在应用程序中缓存信息最直接的方式是在单一位置。单例 bean 完美地符合这种场景。一个自然适合缓存目的的数据结构是 Java `Map`类型。

之前展示的`CarStorage`代码片段代表了一个包含线程安全 map 以存储数据的单例 EJB，该存储被注入并用于其他管理 bean：

```java
@Singleton
@ConcurrencyManagement(ConcurrencyManagementType.BEAN)
public class CarStorage {

    private final Map<String, Car> cars = new ConcurrentHashMap<>();

    public void store(Car car) {
        cars.put(car.getId(), car);
    }

    public Car retrieve(String id) {
        return cars.get(id);
    }
}
```

如果需要更多的灵活性，例如从文件中预加载缓存内容，bean 可以使用`post-construct`和`pre-destroy`方法来控制生命周期。为了保证在应用程序启动时执行功能，EJB 使用`@Startup`注解：

```java
@Singleton
@Startup
@ConcurrencyManagement(ConcurrencyManagementType.BEAN)
public class CarStorage {

    ...

    @PostConstruct
    private void loadStorage() {
        // load contents from file
    }

    @PreDestroy
    private void writeStorage() {
        // write contents to file
    }
}
```

可以使用拦截器以透明的方式添加缓存，而无需程序化注入和使用缓存。拦截器在调用业务方法之前中断执行，并将返回缓存值。最突出的例子是**Java 临时缓存 API**（**JCache**）的`CacheResult`功能。JCache 是一个针对 Java EE 的标准，但截至编写本书时，它尚未包含在总规范中。对于添加 JCache 功能的应用程序，符合条件的业务方法使用`@CacheResult`注解，并由特定的缓存透明地提供服务。

JCache 通常为简单 Java EE 解决方案不足的场景提供复杂的缓存功能。这包括由 JCache 实现提供的分布式缓存。截至今天，通常使用的缓存解决方案是**Hazelcast**、**Infinispan**或**Ehcache**。这在需要将多个缓存与特定关注点（如缓存驱逐）集成时尤其如此。JCache 及其实现提供了强大的解决方案。

# 执行流程

在企业应用程序中运行的业务流程描述了某些流程执行的流程。对于触发的用例，这包括同步请求-响应方法或异步处理触发流程。

用例调用在单独的线程中运行，每个请求或调用一个线程。容器创建这些线程，并在调用成功处理后将其池化以供重用。默认情况下，应用程序类中定义的业务流程以及横切关注点，如事务，都是顺序执行的。

# 同步执行

对于由 HTTP 请求触发并涉及数据库查询的典型用例，其工作方式如下。请求线程处理进入边界的请求；例如，通过控制反转原则，JAX-RS 资源方法由容器调用。资源注入并使用`UserManagement` EJB，该 EJB 也被容器间接调用。所有由代理执行的操作都是在同步术语下进行的。EJB 将使用实体管理器存储一个新的`User`实体，一旦启动当前活动事务的业务方法返回，容器将尝试将事务提交到数据库。根据事务结果，边界资源方法继续并构建客户端响应。所有这些都是在客户端调用阻塞并等待响应的同时同步发生的。

同步执行包括处理同步 CDI 事件。事件解耦了触发域事件和处理它们的过程；然而，事件处理是同步进行的。存在几种事务观察者方法。通过指定事务阶段，事件将在事务提交时间处理，分别是在完成前、完成后、仅在事务失败后或成功事务后。默认情况下，或者在没有活动事务时，CDI 事件在它们被触发时立即处理。这使工程师能够实现复杂解决方案；例如，涉及仅在实体成功添加到数据库之后发生的事件。再次强调，在所有情况下，这种处理都是同步执行的。

# 异步执行

虽然同步执行流程满足了许多业务用例，但其他场景需要异步行为。Java EE 环境对应用程序在多线程方面设置了一些限制。容器管理和池化资源与线程。容器之外的外部并发实用工具不了解这些线程。因此，应用程序的代码不应该启动和管理自己的线程，而应该使用 Java EE 功能来这样做。有几个 API 原生支持异步性。

# 异步 EJB 方法

调用异步行为的一种简单方法是使用 `@Asynchronous` 注解 EJB 业务方法或 EJB 类。对这些方法的调用将立即返回，可选地带有 `Future` 响应类型。它们在单独的、容器管理的线程中执行。这种用法适用于简单场景，但仅限于 EJB：

```java
import javax.ejb.Asynchronous;

@Asynchronous
@Stateless
public class Calculator {

    public void calculatePi(long decimalPlaces) {
        // this may run for a long time
    }
}
```

# 管理执行器服务

对于在 CDI 管理豆或使用 Java SE 并发工具中进行异步执行，Java EE 包括容器管理的 `ExecutorService` 和 `ScheduledExecutorService` 功能。这些用于在容器管理的线程中执行异步任务。`ManagedExecutorService` 和 `ManagedScheduledExecutorService` 的实例被注入到应用程序代码中。这些实例可以用来执行它们自己的可运行逻辑；然而，当与 Java SE 并发工具（如可完成未来）结合使用时，它们表现得尤为出色。以下展示了使用容器管理的线程创建可完成未来的示例：

```java
import javax.annotation.Resource;
import javax.enterprise.concurrent.ManagedExecutorService;
import java.util.Random;
import java.util.concurrent.CompletableFuture;

@Stateless
public class Calculator {

    @Resource
    ManagedExecutorService mes;

    public CompletableFuture<Double> calculateRandomPi(int maxDecimalPlaces) {
        return CompletableFuture.supplyAsync(() -> new Random().nextInt(maxDecimalPlaces) + 1, mes)
                .thenApply(this::calculatePi);
    }

    private double calculatePi(long decimalPlaces) {
        ...
    }
}
```

计算器豆返回一个类型为 *completable future of double* 的值，该值在调用上下文恢复时可能仍在计算。可以询问该计算是否已完成。它还可以组合到后续执行中。无论在企业应用中需要多少新线程，都应使用 Java EE 功能来管理它们。

# 异步 CDI 事件

CDI 事件也可以异步处理。同样，容器将为执行事件处理提供一个线程。要定义异步事件处理器，方法需要使用 `@ObservesAsync` 注解，并使用 `fireAsync()` 方法触发事件。以下代码片段演示了异步 CDI 事件：

```java
@Stateless
public class CarManufacturer {

    @Inject
    CarFactory carFactory;

    @Inject
    Event<CarCreated> carCreated;

    public Car manufactureCar(Specification spec) {
        Car car = carFactory.createCar(spec);
        carCreated.fireAsync(new CarCreated(spec));
        return car;
    }
}
```

事件处理器在一个自己的、容器管理的线程中被调用：

```java
import javax.enterprise.event.ObservesAsync;

public class CreatedCarListener {

    public void onCarCreated(@ObservesAsync CarCreated event) {
        // handle event asynchronously
    }
}
```

由于向后兼容性原因，同步 CDI 事件也可以在 EJB 异步方法中处理。因此，事件和处理程序是以同步方式定义的，但处理程序方法是带有 `@Asynchronous` 注解的 EJB 业务方法。在 Java EE 8 中将异步事件添加到 CDI 标准之前，这是提供此功能的唯一方法。为了避免混淆，应避免在 Java EE 8 及更高版本中使用此实现。

# 异步中的作用域

由于容器无法对异步任务可能运行多长时间做出任何假设，因此作用域的使用有限。在异步任务开始时可用作为请求作用域或会话作用域的 bean 不保证在整个执行过程中都处于活动状态；请求和会话可能已经很久以前就结束了。因此，运行异步任务的线程，例如由管理执行器服务或异步事件提供的线程，无法访问在原始调用期间处于活动状态的作用域 bean 实例。这也包括访问注入实例的引用，例如，在原始同步执行部分中的 lambda 方法中。

在建模异步任务时必须考虑这一点。所有调用特定信息都需要在任务启动时提供。然而，异步任务可以有自己的作用域 bean 实例。

# 定时执行

业务用例不仅可以从外部调用，例如通过 HTTP 请求，还可以从应用程序内部产生，例如通过在指定时间运行的作业。

在 Unix 世界中，cron 作业是触发定期作业的知名功能。EJBs 通过 EJB 计时器提供类似的可能性。计时器根据重复模式或特定时间调用业务方法。以下是一个定义每 10 分钟超时的计划计时器的示例：

```java
import javax.ejb.Schedule;
import javax.ejb.Startup;

@Singleton
@Startup
public class PeriodicJob {

    @Schedule(minute = "*/10", hour = "*", persistent = false)
    public void executeJob() {
        // this is executed every 10 minutes
    }
}
```

所有 EJBs，包括单例、有状态或无状态的 bean 都可以定义计时器。然而，在大多数用例中，在单例 bean 上定义计时器是有意义的。超时将在所有活动 bean 上调用，通常希望可靠地调用计划中的作业；也就是说，在单例 bean 上。因此，此示例将 EJB 定义为在应用程序启动时处于活动状态。这保证了计时器从开始执行。

计时器可以定义为持久的，这延长了它们的生命周期，超出 JVM 的生命周期。容器负责保持计时器的持久性，通常在数据库中。在应用程序不可用期间应该执行的持久计时器在启动时被触发。这也使得在多个实例之间共享计时器的可能性。持久计时器与相应的服务器配置是解决需要在多个服务器上精确执行一次的业务流程的简单解决方案。

使用`@Schedule`注解自动创建的计时器使用类 Unix 的 cron 表达式进行指定。为了获得更大的灵活性，EJB 计时器可以通过容器提供的计时器服务进行编程定义，该服务创建`Timers`和`@Timeout`回调方法。

周期性或延迟作业也可以使用容器管理的计划执行器服务在 EJB 容器外部定义。一个在指定延迟后或周期性执行任务的 `ManagedScheduledExecutorService` 实例可以注入到管理豆中。执行这些任务将使用容器管理的线程：

```java
@ApplicationScoped
public class Periodic {

    @Resource
    ManagedScheduledExecutorService mses;

    public void startAsyncJobs() {
        mses.schedule(this::execute, 10, TimeUnit.SECONDS);
        mses.scheduleAtFixedRate(this::execute, 60, 10, TimeUnit.SECONDS);
    }

    private void execute() {
        ...
    }
}
```

调用 `startAsyncJobs()` 将导致 `execute()` 在调用后 10 秒内在一个管理线程中运行，并在初始一分钟过后，每 10 秒连续运行一次。

# 异步和响应式 JAX-RS

JAX-RS 支持异步行为，以避免在服务器端不必要地阻塞请求线程。即使 HTTP 连接当前正在等待响应，请求线程也可能在服务器上处理长时间运行的过程的同时处理其他请求。请求线程由容器池化，并且这个池只有一定的大小。为了不无谓地占用请求线程，JAX-RS 异步资源方法提交在请求线程返回并可以再次重用之前执行的任务。HTTP 连接在异步任务完成后或超时后恢复并响应。以下代码展示了异步 JAX-RS 资源方法：

```java
@Path("users")
@Consumes(MediaType.APPLICATION_JSON)
public class UsersResource {

    @Resource
    ManagedExecutorService mes;

    ...

    @POST
    public CompletionStage<Response> createUserAsync(User user) {
        return CompletableFuture.supplyAsync(() -> createUser(user), mes);
    }

    private Response createUser(User user) {
        userStore.create(user);

        return Response.accepted().build();
    }
}
```

为了防止请求线程被占用过长时间，JAX-RS 方法需要快速返回。这是因为资源方法是通过控制反转从容器中调用的。完成阶段的成果将在处理完成后用于恢复客户端连接。

返回完成阶段是 JAX-RS API 中相对较新的方法。如果需要超时声明以及更多关于异步响应的灵活性，可以将 `AsyncResponse` 类型注入到方法中。以下代码片段展示了这种方法。

```java
import javax.ws.rs.container.AsyncResponse;
import javax.ws.rs.container.Suspended;

@Path("users")
@Consumes(MediaType.APPLICATION_JSON)
public class UsersResource {

    @Resource
    ManagedExecutorService mes;

    ...

    @POST
    public void createUserAsync(User user, @Suspended AsyncResponse response) {

        response.setTimeout(5, TimeUnit.SECONDS);
        response.setTimeoutHandler(r ->
                r.resume(Response.status(Response.Status.SERVICE_UNAVAILABLE).build()));

        mes.execute(() -> response.resume(createUser(user)));
    }
}
```

使用自定义超时，客户端请求不会无限期等待，只会等到结果完成或调用超时。然而，计算将继续，因为它是在异步执行的。

对于实现为 EJB 的 JAX-RS 资源，可以使用 `@Asynchronous` 业务方法来使用执行器服务省略异步调用。

JAX-RS 客户端也支持异步行为。根据需求，在 HTTP 调用期间不阻塞是有意义的。一个先前的例子展示了如何设置客户端请求的超时。对于长时间运行和特别是并行外部系统调用，异步和 *响应式* 行为提供了好处。

想象有几个提供天气信息的后端。客户端组件访问所有这些后端，并提供平均天气预报。理想情况下，访问系统应该并行发生。

```java
import java.util.stream.Collectors;

@ApplicationScoped
public class WeatherForecast {

    private Client client;
    private List<WebTarget> targets;

    @Resource
    ManagedExecutorService mes;

    @PostConstruct
    private void initClient() {
        client = ClientBuilder.newClient();
        targets = ...
    }

    public Forecast getAverageForecast() {
        return invokeTargetsAsync()
                .stream()
                .map(CompletableFuture::join)
                .reduce(this::calculateAverage)
                .orElseThrow(() -> new IllegalStateException("No weather service available"));
    }

    private List<CompletableFuture<Forecast>> invokeTargetsAsync() {
        return targets.stream()
                .map(t -> CompletableFuture.supplyAsync(() -> t
                        .request(MediaType.APPLICATION_JSON_TYPE)
                        .get(Forecast.class), mes))
                .collect(Collectors.toList());
    }

    private Forecast calculateAverage(Forecast first, Forecast second) {
        ...
    }

    @PreDestroy
    public void closeClient() {
        client.close();
    }
}
```

`invokeTargetsAsync()`方法异步调用可用的目标，使用管理执行器服务。返回并使用`CompletableFuture`处理程序来计算平均结果。调用`join()`方法将阻塞，直到调用完成，并将提供个别结果。

通过异步调用可用的目标，它们并行调用并等待可能缓慢的资源。等待天气服务资源的时间仅与最慢的响应时间相同，而不是所有响应的总和。

JAX-RS 的最新版本原生支持完成阶段，这减少了应用程序中的样板代码。类似于使用可完成未来，调用立即返回一个完成阶段实例以供进一步使用。以下演示了使用`rx()`调用实现的响应式 JAX-RS 客户端功能：

```java
public Forecast getAverageForecast() {
    return invokeTargetsAsync()
            .stream()
            .reduce((l, r) -> l.thenCombine(r, this::calculateAverage))
            .map(s -> s.toCompletableFuture().join())
            .orElseThrow(() -> new IllegalStateException("No weather service available"));
}

private List<CompletionStage<Forecast>> invokeTargetsAsync() {
    return targets.stream()
            .map(t -> t
                    .request(MediaType.APPLICATION_JSON_TYPE)
                    .rx()
                    .get(Forecast.class))
            .collect(Collectors.toList());
}
```

上述示例不需要查找管理执行器服务。JAX-RS 客户端将内部管理这一点。

在`rx()`方法引入之前，客户端包含了一个显式的`async()`调用，其行为类似，但仅返回`Future`。响应式客户端方法通常更适合项目需求。

如前所述，我们正在使用容器管理的执行器服务，因为我们处于 Java EE 环境中。

# 现代 Java EE 的概念和设计原则

Java EE API 是围绕整个标准集存在的约定和设计原则构建的。软件工程师在开发应用程序时将发现熟悉的 API 模式和做法。Java EE 旨在保持一致的 API 使用。

对于首先关注业务用例的应用程序，该技术的最重要原则是*不阻碍*。如前所述，工程师应该能够专注于解决业务问题，而不是将大部分时间花在处理技术或框架问题上。理想情况下，领域逻辑使用普通 Java 实现，并通过注解和方面增强，这些注解和方面能够使企业环境生效，而不影响或模糊领域代码。这意味着技术不需要过多的工程师关注，通过强制过于复杂的约束来实现。在过去，J2EE 需要许多这些过于复杂的解决方案。管理 Bean 以及持久化 Bean 需要实现接口或扩展基类。这模糊了领域逻辑并复杂化了可测试性。

在 Java EE 时代，领域逻辑是在用注解标注的普通 Java 类中实现的，这些注解告诉容器运行时如何应用企业关注点。清洁代码实践通常建议编写易于阅读的代码，而不是可重用性。Java EE 支持这种做法。如果出于某种原因需要替换技术并提取领域逻辑，只需简单地移除相应的注解即可。

正如我们在第七章中将要看到的，*测试*编程方法高度支持可测试性，因为对于开发者来说，大多数 Java EE 特性不过是一些注解。

在整个 API 中存在的一个设计原则是**控制反转**（**IoC**），换句话说，*不要调用我们，我们会调用你*。我们在应用边界，如 JAX-RS 资源中特别看到这一点。资源方法由注解 Java 方法定义，随后由容器在正确的上下文中调用。对于需要解决生产者或包括横切关注点（如拦截器）的依赖注入也是如此。应用程序开发者可以专注于实现逻辑和定义关系，而将实际的管道工作留给容器。另一个不那么明显的例子是通过 JSON-B 注解声明 Java 对象到 JSON 以及反向映射。这些对象以声明性方式隐式映射，不一定以显式、程序性的方式。

另一个使工程师能够以高效方式使用技术的原则是**约定优于配置**。默认情况下，Java EE 定义了与大多数用例相匹配的特定行为。如果这不足以满足或不符合要求，行为可以被覆盖，通常在多个层面上。

关于约定优于配置的例子数不胜数。JAX-RS 资源方法将 Java 功能映射到 HTTP 响应就是这样一个方法。如果 JAX-RS 关于响应的默认行为不足够，可以使用`Response`返回类型。另一个例子是管理 Bean 的指定，通常使用注解来实现。要覆盖这种行为，可以使用`beans.xml` XML 描述符。对于开发者来说，一个受欢迎的方面是在现代 Java EE 世界中，企业应用程序的开发是实用且高度高效的，通常不需要像过去那样使用大量的 XML。

在开发者生产力的方面，Java EE 的另一个重要设计原则是平台要求容器集成不同的标准。一旦容器支持一组特定的 API，如果整个 Java EE API 都得到支持，那么 API 的实现也必须能够直接集成其他 API。一个很好的例子是 JAX-RS 资源能够使用 JSON-B 映射和 Bean Validation，而无需除了注解之外的其他显式配置。在先前的例子中，我们看到了如何在不需额外努力的情况下，将定义在不同标准中的功能一起使用。这也是 Java EE 平台最大的优势之一。总规范确保了特定标准能够良好地协同工作。开发者可以依赖应用程序服务器提供某些功能和实现。

# 保持可维护且高质量的代码

开发者普遍认为代码质量是一个值得追求的目标。但并非所有技术都能同样好地支持这一目标。

如前所述，业务逻辑应该是应用程序的主要关注点。如果业务逻辑发生变化或在工作领域后出现新的知识，领域模型以及源代码都需要进行重构。迭代重构是达到并保持模型领域以及源代码总体高质量所必需的。领域驱动设计描述了深化对业务知识理解的努力。

在代码级别重构方面已经有很多论述。在将业务逻辑最初表示为代码并通过测试验证之后，开发者应花一些时间和精力重新思考和改进第一次尝试。这包括标识符名称、方法和类。特别是，*命名*、*抽象层*和*单一职责*是重要的方面。

遵循领域驱动设计的推理，业务领域应尽可能符合其代码表示。这包括，尤其是，领域语言；也就是说，开发人员和业务专家如何讨论某些特性。整个团队的目标是找到一个统一、*无处不在的语言*，它不仅用于讨论和演示文稿中，而且在代码中也得到了良好的体现。业务知识的细化将通过迭代方法进行。这种方法不仅包括代码级别的重构，还意味着初始模型从一开始就不会完美地匹配所有需求。

因此，所使用的技术应支持模型和代码的变化。过多的限制会在以后变得难以更改。

对于一般的应用程序开发，尤其是对于重构，拥有足够的自动化软件测试覆盖范围至关重要。一旦代码发生变化，回归测试确保没有业务功能被意外损坏。足够的测试用例因此支持重构尝试，为工程师提供清晰的指示，以确定在修改后功能是否仍然按预期工作。理想情况下，技术应通过不限制代码结构来支持可测试性。第七章 *测试* 将详细介绍这一主题。

为了实现 *可重构性*，松耦合比紧耦合更受欢迎。如果其中任何一个发生变化，所有明确调用或需要其他组件的功能都需要进行修改。Java EE 在多个方面支持松耦合；例如，依赖注入、事件或横切关注点，如拦截器。所有这些都简化了代码更改。

有一些工具和方法可以衡量质量。特别是，静态代码分析可以收集有关复杂性、耦合、类和包之间的关系以及一般实现的信息。这些手段可以帮助工程师识别潜在问题，并提供软件项目的整体图景。第六章，*应用程序开发工作流程*涵盖了如何以自动化的方式验证代码质量。

通常，建议持续重构和改进代码质量。软件项目往往被驱动去实现产生收入的新功能，而不是改进现有功能。问题在于，重构和改进质量往往被认为从业务角度来看没有提供任何好处。当然，这并不是真的。为了实现稳定的速度并满足质量地集成新功能，重新考虑现有功能是绝对必要的。理想情况下，周期性的重构周期已经纳入了项目计划。经验表明，项目经理往往没有意识到这个问题。然而，软件工程师团队有责任解决质量的相关性。

# 摘要

建议工程师首先关注领域和业务逻辑，从用例边界开始，逐步降低抽象层。Java EE 核心领域组件，即 EJB、CDI、CDI 生产者和事件，用于实现纯业务逻辑。其他 Java EE API 主要用于支持业务逻辑的技术需求。正如我们所见，Java EE 以现代方式实现了和鼓励了众多软件设计模式和领域驱动设计的方法。

我们已经看到了如何选择和实现同步和异步的通信方式。通信技术取决于业务需求。特别是 HTTP 在 Java EE 中通过 JAX-RS 被广泛使用和良好支持。REST 是支持松散耦合系统的通信协议架构风格的典范。

Java EE 自带的功能实现了并启用了诸如管理持久性、配置或缓存等技术横切关注点。特别是 CDI 的使用实现了各种技术驱动的用例。所需的异步行为可以通过不同的方式实现。应用程序不应管理自己的线程或并发管理，而应使用 Java EE 功能。容器管理的执行服务、异步 CDI 事件或 EJB 计时器是应该使用的例子。

Java EE 平台的概念和原则支持以业务逻辑为重点开发企业应用程序。特别是不同标准的精益集成、控制反转、约定优于配置以及“不干涉”的原则，都支持这一方面。工程师应该致力于保持高代码质量，这不仅包括代码级别的重构，还包括精炼业务逻辑和团队共享的*通用语言*。精炼代码质量以及领域模型的适用性是在迭代步骤中发生的。

因此，技术应该支持模型和代码的变化，而不是对解决方案施加过多的限制。Java EE 通过最小化框架对业务代码的影响，并通过允许功能松散耦合来实现这一点。团队应该意识到重构与自动化测试是高质量软件的必要条件。

以下章节将涵盖其他哪些方面使 Java EE 成为开发企业应用程序的现代和合适平台。我们将看到哪些部署方法是可以取的，以及如何为高效的开发工作流程奠定基础。
