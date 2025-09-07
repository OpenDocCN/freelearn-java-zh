

# 第一章：Spring Boot 和 Angular – 整体图景

首先，我们想感谢您购买这本书的副本，这本书是为开发者编写的，旨在学习如何使用行业开发标准构建全栈 Web 应用程序。这本书是根据我们从培训和研讨会中开发的应用程序定制的。那么，让我们开始我们的冒险之旅吧。

本章将作为一个简短的回顾，涉及 Java Spring Boot 和 Angular 的基础知识，以便您对如何进行这些框架的 Web 开发有一个大致的了解。您还将了解到社区的大小以及 Angular 所提供的支持使其在开发应用程序时变得可靠。

在本章中，我们将涵盖以下主题：

+   介绍 Spring Boot

+   使用 Spring Boot 的优势

+   Java 17 的新特性

+   介绍 Angular

+   使用 Angular 的优势

# 技术要求

我们将要构建的应用程序的 GitHub 仓库可以在[`github.com/PacktPublishing/Spring-Boot-and-Angular`](https://github.com/PacktPublishing/Spring-Boot-and-Angular)找到。

每一章都有一个目录，其中包含项目的完成部分。

注意

从第一章“Spring Boot 和 Angular – 整体图景”到第四章“设置数据库和 Spring Data JPA”，将不会有目录可用，因为我们将要涵盖的大部分主题将包括理论和一些示例代码。实际项目将在第五章“使用 Spring 构建 API”开始。

# 介绍 Spring Boot

**Spring Boot** 是来自 Pivotal 的开源微框架。它是一个面向开发者的企业级框架，用于在**Java 虚拟机**（**JVMs**）上创建独立应用程序。其主要重点是缩短您的代码，以便您更容易运行应用程序。

该框架扩展了 Spring 框架，为您提供了配置应用程序的更具意见性的方式。此外，它还内置了自动配置功能，根据您的设置配置 Spring 框架和第三方包。Spring Boot 利用这些知识来避免配置时的代码错误，因为它在设置我们的应用程序时减少了样板代码。

现在，让我们讨论使用 Spring Boot 的主要优势。

# 使用 Spring Boot 的优势

以下是用 Spring Boot 开发应用程序的四个主要优势：

+   **自动配置**：当您配置 Spring Boot 应用程序时，它会下载运行应用程序所需的所有依赖项。它还会根据您应用的设置配置 Spring 框架，包括相关的第三方包。因此，Spring Boot 避免了样板代码和配置错误，您可以直接开始开发 Spring 应用程序。

+   **有观点的方法**：Spring Boot 采用一种基于应用程序需求来安装依赖项的窄方法。它将为您的应用程序安装所有必需的包，并摒弃了手动配置的想法。

+   **Spring starters**：您可以在初始化过程中选择一系列的启动依赖项来定义应用程序预期的需求。Spring Starter 的一个例子是 Spring Web，它允许您在不配置运行应用程序所需的依赖项的情况下初始化基于 Spring 的 Web 应用程序。相反，它将自动安装 Apache Tomcat Web 服务器和 Spring Security 以提供认证功能。

+   **创建独立应用程序**：Spring Boot 可以运行没有外部 Web 服务器依赖项的独立应用程序。例如，我们可以嵌入服务器，如 Tomcat，并运行应用程序。

## Spring 和 Spring Boot 之间的区别

那么，Spring 和 Spring Boot 之间的区别是什么？在开始使用 Spring Boot 之前，您需要了解 Spring 框架吗？让我们从第一个问题开始。

以下表格展示了两个框架之间的区别：

| ![C:\Users\Seiji Villafranca\AppData\Local\Microsoft\Windows\INetCache\Content.MSO\943B62F6.tmp](img/Table1.png) | ![spring-boot-logo - THE CURIOUS DEVELOPER](img/Table2.png) |
| --- | --- |
| 开发者配置项目的依赖项。 | 使用 Spring Starters，Spring Boot 将配置运行应用程序所需的所有依赖项。 |
| Spring 是构建应用程序的**Java EE 框架**。 | Spring Boot 通常用于构建**REST API**。 |
| Spring 通过提供如 Spring JDBC、Spring MVC 和 Spring Security 等模块简化了 Java EE 应用程序的开发。 | Spring Boot 提供了依赖项的配置，减少了模块布局的样板代码，这使得运行应用程序更加容易。 |
| **依赖注入（DI**）和**控制反转（IOC**）是 Spring 构建应用程序的主要特性。 | **Spring Boot Actuator**是一个功能，可以公开有关您的应用程序的操作信息，例如指标和流量。 |

我们可以确定 Spring Boot 是建立在 Spring 之上的，主要区别在于 Spring Boot 会自动配置我们运行 Spring 应用所需的依赖项。所以，要回答关于在开始使用 Spring Boot 之前是否需要学习 Spring 框架的问题，答案是**不**——Spring Boot 只是 Spring 的一个扩展，由于其有见地的方法，它使得配置更快。

现在，让我们看看在 Spring 和 Spring Boot 中配置一个 Web 应用所需的依赖关系。

## Spring 和 Spring Boot 的依赖项示例

在 Spring 中，我们应用运行所需的最低依赖项是**Spring Web**和**Spring Web MVC**：

```java
<dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-web</artifactId>
     <version>5.3.5</version>
</dependency>
<dependency>
     <groupId>org.springframework</groupId>
     <artifactId>spring-webmvc</artifactId>
<version>5.3.5</version>
</dependency>
```

Spring Boot 只需要`spring-boot-starter-web`，这是我们应用运行所需的 Spring Starter。必要的依赖项在构建时自动添加，因为 Starter 将负责配置：

```java
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-web</artifactId>
<version>2.4.4</version>
</dependency>
```

在 Spring 中，我们需要考虑的另一件事是为我们的应用在服务器上运行定义一些配置，例如分发器 servlet 和映射：

```java
public class SpringInitializer implements WebApplicationInitializer {
@Override
public void onStartup(ServletContext container) {
AnnotationConfigWebApplicationContext context =
    new AnnotationConfigWebApplicationContext();
context.setConfigLocation("com.springexample");
container.addListener(new        ContextLoaderListener(context));
 ServletRegistration.Dynamic dispatcher =
     container.  addServlet("dispatcher",
         new  DispatcherServlet(context));
 dispatcher.setLoadOnStartup(1);
 dispatcher.addMapping("/");
   }
}
```

在初始化分发器 servlet 之后，我们还需要使用`@EnableWebMvc`，并有一个带有`@Configuration`注解的`Configuration`类，我们将为应用实例化一个视图解析器。

在配置类中将创建一个新的`InternalResourceViewResolver()`实例。这将是一个 Spring 的 bean。在这里，所有位于`/WEB-INF/view`路径下且具有`.jsp`文件扩展名的文件都将被解析：

```java
@EnableWebMvc
@Configuration
public class SpringWebConfig implements WebMvcConfigurer {
   @Bean
   public ViewResolver viewResolver() {
       InternalResourceViewResolver bean =
           new  InternalResourceViewResolver();
   bean.setViewClass(JstlView.class);
   bean.setPrefix("/WEB-INF/view/");
   bean.setSuffix(".jsp");
   return bean;
   }
}
```

在 Spring Boot 中，所有这些配置都将被省略，因为这段代码已经包含在 Spring Starter 中。我们只需要定义一些属性，以便使用 Web Starter 使我们的应用运行：

```java
spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
```

定义了这些属性后，我们的应用将运行，因为所有必要的配置，如**Web 初始化器**和**MVC 配置**，都已包含在内。

因此，我们已经讨论了 Spring Boot 的优势，同时，也讨论了 Spring Boot 与 Spring 框架之间的主要区别以及它是如何减少配置时的样板代码的。

如您可能已经知道，Spring 的主要语言是 Java，Java 17 现在已经发布。在下一节中，我们将了解 Java 17 的新特性。

# Java 17 的新特性是什么？

我们已经决定在这本书中讨论 Java 17，因为这是 Java 的下一个**长期支持**（**LTS**）版本，这意味着这个版本将得到更长时间的维护。它于 2021 年 9 月 14 日发布，并包含了一些新的安全和开发特性。

让我们看看已经包含的一些新特性，以及应用于 Java 17 的一些修改。

## 密封类

`permits`关键字用于标识我们想要授予权限的特定类，如下面的示例所示：

```java
public sealed class Animal permits Cat, Dog, Horse
```

## 外部函数和内存 API

引入了一个新的 API，用于访问和使用 Java 运行时之外代码，它通过应用外部函数（JVM 之外代码）和安全的访问外部内存（JVM 不处理内存）来实现。该 API 允许 Java 应用程序在不使用**Java 本地** **接口**（**JNI**）的情况下调用本地库。

该 API 旨在用纯 Java 开发模型替换 JNI，并在访问堆外数据时提供更好的性能，同时省略不安全操作。

### 外部内存

Java 当前的一个常见问题是访问堆外数据。**堆外数据**是指存储在 Java 运行时之外内存中的数据。我们可以将其称为第三方库。访问这些数据对于性能至关重要，因为 Java 垃圾收集器只处理堆内数据，这使得它们可以避免垃圾收集的不确定性。以下 API 用于处理堆外数据：

+   **ByteBuffer API**：此 API 允许你在堆外数据中创建直接 ByteBuffer，以便数据可以在 Java 运行时之外进行管理。然而，ByteBuffer 的主要缺点是其最大大小为 2 GB，并且它不会及时释放，导致应用程序的运行时性能变慢。

+   **Sun.misc.Unsafe API**：Unsafe API 公开了对堆外数据进行操作的操作。由于**即时编译器**（**JIT**）优化了访问操作，因此此 API 使此过程效率更高。然而，不建议使用 Unsafe API，因为我们正在允许访问任何内存位置。

+   **外部函数和内存 API**：此 API 解决了访问内存位置和牺牲运行时性能的困境，因为它提供了应用程序可以执行以下操作的类和接口：

    +   分配外部内存

    +   操作和访问外部内存

    +   调用外部函数

## 使用 switch 语句进行模式匹配

模式匹配是测试 switch 语句中的模式和复杂表达式的想法。这个新特性允许 switch 语句以更可扩展和灵活的方式使用，以接受复杂表达式。

## Applet API

**Applet API**在 Java 中很少使用，因为所有浏览器都已移除对 Java 浏览器插件的支持。

## 实验性的 AOT 和 JIT 编译器

由于其功能使用有限，实验性的基于 Java 的**即时编译器**（**AOT**）和**即时编译器**（**JIT**）已被移除。

这些只是 Java 17 中应用的一些更改。现在，让我们了解 Angular，这是当今最顶尖的 JavaScript 框架之一，以及使用 Angular 框架开发前端的优势。

# 介绍 Angular

**Angular** 是一个由 Google 维护的免费开源 JavaScript 框架。它主要用于开发 Web 应用程序，并通过插件扩展了其功能，使其能够创建移动和桌面应用程序。Angular 使用基于组件的代码，是渐进式的，并提供了许多库和扩展，这些库和扩展可以缩短开发大型应用程序的时间。

在撰写本文时，Angular 构建前端应用非常流行。它是三星、Upwork、PayPal 和 Google 等大型知名公司开发应用程序的主要框架。它还拥有一个非常活跃的社区，在 GitHub 上有 76,000 个星标，大约有 1,500 人为该框架做出贡献。此外，它拥有数千个功能齐全的 NPM 库，您可以使用这些库来加速您的开发。

## Angular 的历史

在成为 Angular 之前，Google 开发的第一个框架是 **AngularJS** 或 Angular 版本 1。尽管开发者通常对此感到困惑，因为他们认为 AngularJS 和 Angular 很相似，但 AngularJS 是由 Google 员工 *Miško Hevery* 以开源框架的形式发布的，他正在开发 AngularJS 以加快 Web 应用程序的开发。

使用 JavaScript 或 Dart 的 AngularJS 由于其社区变得更加广泛而变得流行。同时，发布了 Ionic 框架，允许开发者使用 AngularJS 构建原生移动应用程序。

### 伟大的重写

JavaScript 技术的快速和快速发展影响了 AngularJS 的流行度，团队在框架方面走到了尽头——没有进一步改进的空间。从 2014 年到 2015 年，Google 团队和社区决定使用该框架支持移动和大型企业应用程序。他们的第一步是 **伟大的重写**，而不是增加 AngularJS 的设计。伟大的重写是 **Angular 2.0** 或简称为 Angular 的发布。

### 行动的难题

许多应用程序已经在 AngularJS 上运行，这意味着如果发布了一个全新的 Angular 版本，AngularJS 用户的支持将结束。因此，这里的主要问题之一是，“*这些应用程序在几年后会如何得到支持？*”

另一个出现的问题是，没有直接从 AngularJS 迁移到 Angular 2.0 的方法，这对开发者来说很困难。这对团队来说是一个巨大的步骤——以至于在每次发布中都引入了新的概念和重大更改。

### 框架的回归

尽管迁移 Angular 很痛苦，但 Google 创建的企业应用程序得到了支持。到 2018 年左右，随着框架拥有大量可用于构建大型应用程序的功能，这变得更加稳定。此外，它不依赖于第三方库来创建表单和调用 HTTP 请求，因为所有依赖项都已包含在内。Google 还发布了一些文档，以帮助开发者将 AngularJS 迁移到 Angular 的最新版本。

Angular 非常受欢迎，在开发企业应用程序方面非常有效。现在，让我们看看 Angular 的优势以及为什么它对开发有效。

# 使用 Angular 的优势

Angular 是一个基于组件的框架，这意味着我们将应用程序的部分开发成更小的块，我们可以在整个应用程序中重用这些块。这个特性通过确保没有太多重复代码来减少样板代码和代码错误。Angular 的一个主要优势是其语言。让我们更深入地了解一下。

### 基于 TypeScript 的框架

Angular 是一个基于 **TypeScript 语言**的框架。这种语言是一个显著的优势，因为 TypeScript 提供了对开发有益的功能。此外，它还是 JavaScript 的超集，它添加了新的概念，使代码可维护和有效：

![图 1.1 – TypeScript – 一种超集语言](img/B18159_01_01.jpg)

图 1.1 – TypeScript – 一种超集语言

如我们所见，TypeScript 是建立在 ES6 和 JavaScript 之上的，旨在为开发添加更多功能。TypeScript 的某些组件包括泛型、类型和接口，这些我们都知道与**面向对象编程**（**OOP**）直接相关。现在，让我们看看另一个优势。

#### 静态类型数据

TypeScript 可以定义静态类型数据，这使得变量可以严格类型化。与纯 JavaScript 相比，编译器会在编译时提醒你任何类型相关的错误——也就是说，哪些错误是在运行时捕获的。因此，TypeScript 可以通过在编译时提示这些问题来避免生产中的错误。

#### 可预测性和可维护性

由于 TypeScript 是强类型的，这有助于可预测性的概念。例如，一个变量被声明为数字类型。因此，在整个应用程序中它始终保持数字类型，函数将指定如何实现它们，因为所有参数也都是严格类型。此外，TypeScript 也是可维护的，因为它让开发者能够在编译时调试应用程序。

#### IDE 支持

由于 TypeScript 正在变得越来越广泛地使用，越来越多的 IDE 支持它。IDE 提供了诸如代码导航、自动完成和插件等几个功能。

Microsoft Visual Studio 是用于 TypeScript 的主要 IDE。然而，一些 IDE 和编辑器也支持运行 TypeScript：

+   **Atom**：一个跨平台编辑器

+   **Eclipse**：一个具有 TypeScript 插件的 IDE

+   **Visual Studio Code**：微软的一个轻量级跨平台编辑器

#### 面向对象编程（OOP）

TypeScript 是一种**面向对象的语言**，这意味着它支持诸如类、接口和继承等概念。面向对象编程（OOP）在将我们的应用程序开发成对象时非常可扩展，如果我们正在开发不断增长的应用程序，这可以是一个优势。

#### 早期发现的错误

浏览器不能直接理解 TypeScript。相反，它们使用**转换器**，将这些代码编译成纯 JavaScript。在这里，所有与语法和类型相关的错误都会被捕获，使得开发者可以专注于代码逻辑。

这些只是 TypeScript 语言的优点。现在，让我们看看 Angular 自身的优势。

### 支持大型企业级应用

Angular 被视为一个一站式包框架，因为构建应用程序所需的大部分标准功能已经包含在内。这包括模块。例如，要在 Angular 应用程序中使用表单，我们必须导入 `FormsModule` 和 `ReactiveormsModule`。其他例子包括导航和路由。Angular 提供了 `RouterModule`，这样您就可以在应用程序内创建路由。

### 单页应用程序

Angular 是一个**单页应用程序**（**SPA**），这意味着当用户从一个页面导航到另一个页面时，页面不会重新加载，因为数据是由服务器获取的。此外，客户端的资源是独立的，并且已经在浏览器中加载，这有助于提高应用程序的加载性能。

### 渐进式网页应用（PWAs）

**渐进式网页应用**（**PWAs**）如今正成为一种趋势。它们是一种允许网页应用在移动应用以及不同平台（包括在线和离线）上运行解决方案。由于 Angular 的脚图，配置 Angular 为 PWA 非常简单——只需一行代码，您的 Angular 应用程序就配置好了。使用 PWA Builder，PWAs 还可以上传到 Android Play 商店和 Microsoft Store。

以下命令使用 Angular CLI 将我们的应用程序转换为 PWA：

```java
ng add @angular/pwa
```

### Angular CLI

我们不需要从头创建或配置 Angular。相反，我们可以使用 Angular CLI，它有助于安装运行我们的 Angular 应用程序所需的依赖项。尽管脚图功能负责创建所需的文件、安装包以及配置我们应用程序所需的值，但 Angular CLI 为**模块**、**组件**、**服务**和**指令**生成样板代码，以加快开发速度。

在以下代码中，我们使用 `npm` 安装 Angular CLI 并使用 `ng` 命令生成我们的代码：

```java
//command for installing angular CLI
npm install -g @angular/cli
//command for creating a new Angular App
ng new –project-name
// command for creating a new Component
ng generate component –component-name
// command for creating a new Service
ng generate service –component-name
// command for creating a new Module
ng generate module –component-name
```

### 基于模块和组件的框架

Angular 被分组到**模块**中，这使得代码结构更容易维护。此外，应用程序的每个部分都可以按其功能分组，并放置在单个模块中，这使得导航应用程序的功能更容易。它还有利于单元测试，因为代码是单独测试的，从而允许进行完全的质量控制。

将代码作为组件创建可以促进代码的可重用性和减少模板代码。让我们看看一个导航菜单的例子：

```java
<!— Example code for nav bar -->
<nav class="navbar navbar-default">
  <div class="container-fluid">
    <div class="navbar-header">
      <a class="navbar-brand" href="#">Nav bar</a>
    </div>
    <ul class="nav navbar-nav">
      <li class="active"><a href="#">Home</a></li>
      <li><a href="#">Home</a></li>
      <li><a href="#">About</a></li>
      <li><a href="#">Contact Us</a></li>
    </ul>
  </div>
</nav>
```

导航栏必须出现在我们应用程序的每一页上。这个过程将导致代码冗余，这意味着我们不得不反复重复这段代码。然而，在 Angular 中，它已经被开发成一个组件，允许我们在应用程序的不同部分重用代码。为导航栏代码分配了一个特定的选择器，并用作组件的 HTML 标签，如下面的代码所示：

```java
<!—example selector for the navigation bar component-->
<app-navigation-bar/>
```

### 支持跨平台

Angular 用于构建网页、原生移动和桌面应用程序的应用程序。现在，通过像 Ionic、NativeScript 和 Electron 这样的框架，这是可能的。除了 PWA 之外，Ionic 和 NativeScript 还用于使用 Angular 创建移动应用程序。另一方面，Electron 是一个框架，它使用类似的代码库将您的 Angular 应用程序转换为桌面应用程序。这个特性使得 Angular 作为一个单一框架非常灵活，因为它可以覆盖您应用程序的所有平台。

### 网页组件

Angular 支持**网页组件**，也称为**Angular 元素**。在这里，想法是将应用程序分解成更小的部分，并将其分发到独立的应用程序或包中，这些应用程序或包可以在其他应用程序中分发和使用。Angular 元素涵盖了微前端的概念。每个元素都有一个部署管道。这个组件也可以用于不同的 JavaScript 框架，如 React 和 Vue。

### 支持懒加载

在客户端浏览器中加载所有 JavaScript 代码可能会引入一些问题。如果应用程序变得更加庞大，更多的代码会被打包到一个块中。我们不希望将所有代码都初始化，因为这会导致我们的应用程序在首次启动时加载缓慢。我们只想按需加载所需的内容。Angular 的懒加载功能解决了这个问题。它只加载特定路由所需的应用程序模块、组件、服务、指令和其他元素。这个功能减少了用户首次打开应用程序时的加载时间。

在以下代码中，我们定义了一些路由作为一个数组，我们将新的路由作为对象添加。为了启用懒加载，我们必须使用`loadChildren`属性按需加载模块：

```java
const route: Routes = [
    {
     path: "about",
 loadChildren: () =>
   import("./src/app/AboutModule").then(m =>
      m.AboutModule)
    },
   {
     path: "contact",
     loadChildren: () =>
       import("./src/app/ContactModule").then(m =>
         m.ContactModule)
    }
];
```

在前面的代码中，当用户导航到`about`路径时，它只会加载包含该特定路由资源的`AboutModule`。除非用户导航到`contact`路径，否则它不会加载`ContactModule`下的资源。

# 摘要

在本章中，你了解到 Spring Boot 是 Spring 框架的一个开源框架扩展，它主要解决了在配置 Spring 框架时产生的样板代码问题。此外，它还提供了**Spring Starters**，开发者可以使用这些 Starters 让 Spring Boot 自动配置所需的依赖。

另一方面，Angular 是一个基于组件的框架，它使用 TypeScript 语言构建，以赋予它面向对象的能力。此外，它具有跨平台支持，允许开发者创建可在网页、移动设备和桌面应用程序上运行的应用程序。由于 Angular 被多家大型公司使用，并得到 Google 和庞大社区的支持，因此它是 JavaScript 框架中的佼佼者。

在下一章中，你将学习必须安装到你的计算机上的软件，并设置全栈开发的环境。
