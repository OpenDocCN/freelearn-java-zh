# 第二十五章：超越 Spring Web

在本章中，我们将看到我们已经走了多远，我们解决了哪些问题，还有哪些问题有待解决。

我们将讨论 Spring 生态系统的一般情况，以及持久性、部署和单页应用程序。

# Spring 生态系统

从 Web 到数据，Spring 是一个全面的生态系统，旨在以模块化的方式解决各种问题：

![Spring 生态系统](img/image00993.jpeg)

请查看 Spring IO 平台[`spring.io/platform`](https://spring.io/platform)。

## 核心

在 Spring 框架的核心，显然有一个依赖注入机制。

我们只是浅尝了安全功能和框架与 Groovy 的出色集成。

## 执行

我们详细了解了 Spring Boot 的内容——将简单性和内聚性带入庞大的子项目网络。

它使您能够专注于真正重要的事情，即您的业务代码。

Spring XD 项目也非常有趣。其目标是提供处理、分析、转换或导出数据的工具，并且明确关注大数据。有关更多信息，请访问[`projects.spring.io/spring-xd`](http://projects.spring.io/spring-xd)。

## 数据

在开发我们的应用程序时，我们还没有考虑如何在数据库中存储数据。在 Pivotal 的参考架构中，有一个专门用于关系数据和非关系（NoSQL）数据的层。

Spring 生态系统在`spring-data`标签下提供了许多有趣的解决方案，可以在[`projects.spring.io/spring-data/`](http://projects.spring.io/spring-data/)找到。

在构建缓存时，我们瞥见了 Spring Data Redis，但 Spring Data 还有更多内容。

所有 Spring Data 项目都共享基本概念，例如模板 API，这是一个从持久性系统中检索和存储对象的抽象。

Spring Data JPA（[`projects.spring.io/spring-data-jpa/`](http://projects.spring.io/spring-data-jpa/)）和 Spring Data Mongo（[`projects.spring.io/spring-data-mongodb/`](http://projects.spring.io/spring-data-mongodb/)）是一些最著名的 Spring Data 项目。它们让您通过存储库操作实体，这些存储库是提供创建查询、持久化对象等功能的简单接口。

Petri Kainulainen（[`www.petrikainulainen.net/spring-data-jpa-tutorial/`](http://www.petrikainulainen.net/spring-data-jpa-tutorial/)）在 Spring Data 上有很多深入的例子。它没有使用 Spring Boot 提供的设施，但您应该能够很容易地开始使用指南，例如[`spring.io/guides/gs/accessing-data-jpa/`](https://spring.io/guides/gs/accessing-data-jpa/)。

Spring Data REST 也是一个神奇的项目，它将通过 RESTful API 半自动地公开您的实体。请访问[`spring.io/guides/gs/accessing-data-rest/`](https://spring.io/guides/gs/accessing-data-rest/)获取详细教程。

## 其他值得注意的项目

Spring Integration（[`projects.spring.io/spring-integration`](http://projects.spring.io/spring-integration)）和 Spring Reactor（[`projectreactor.io`](http://projectreactor.io)）也是我最喜欢的 Spring 项目之一。

Spring Reactor 是 Pivotal 实现的反应流。其想法是在服务器端提供完全非阻塞的 IO。

另一方面，Spring Integration 专注于企业集成模式，并允许你设计通道来加载和转换来自异构系统的数据。

关于你可以通过通道实现的一个很好而简单的例子可以在这里看到：[`ilopmar.github.io/contest/#_spring_boot_application`](http://ilopmar.github.io/contest/#_spring_boot_application)。

如果你的应用需要与异构和/或复杂的子系统进行通信，那么一定值得一看。

在 Spring 生态系统中，我们还没有提到的最后一个项目是 Spring Batch，它是一个非常有用的抽象，用于处理企业系统的日常运营中的大量数据。

# 部署

Spring Boot 提供了将你的 Spring 应用程序作为简单的 JAR 运行和分发的能力，在这方面取得了很大的成功。

这无疑是朝着正确方向迈出的一步，但有时你不只是想部署你的 Web 应用。

当处理具有多个服务器和数据源的复杂系统时，运维团队的工作可能会变得非常头疼。

## Docker

谁没有听说过 Docker 呢？它是容器世界中的新宠儿，并且由于其充满活力的社区而取得了相当大的成功。

Docker 的理念并不新颖，它利用了 Linux 容器（LXC）和 cgroups 来为应用程序提供完全隔离的环境。

你可以在 Spring 网站上找到一个关于 Docker 的教程，它会指导你进行第一步：[`spring.io/guides/gs/spring-boot-docker`](https://spring.io/guides/gs/spring-boot-docker)。

Pivotal Cloud Foundry 多年来一直在使用容器技术，他们的容器管理器叫做 Warden。他们最近转向了 Garden，这是一个支持不仅仅是 Linux 容器，还有 Windows 容器的抽象。

Garden 是 Cloud Foundry 的最新版本（称为 Diego）的一部分，它还允许 Docker 镜像作为部署单元。

Cloud Foundry 的开发人员版本也以 Lattice 的名字发布了，可以在[`spring.io/blog/2015/04/06/lattice-and-spring-cloud-resilient-sub-structure-for-your-cloud-native-spring-applications`](https://spring.io/blog/2015/04/06/lattice-and-spring-cloud-resilient-sub-structure-for-your-cloud-native-spring-applications)找到。

如果你想在不使用命令行的情况下测试容器，我建议你看看 Kitematic。通过这个工具，你可以在不在系统上安装二进制文件的情况下运行 Jenkins 容器或 MongoDB。访问[`kitematic.com/`](https://kitematic.com/)了解更多关于 Kitematic 的信息。

Docker 生态系统中另一个值得一提的工具是 Docker Compose。它允许你通过一个配置文件运行和链接多个容器。

请参考[`java.dzone.com/articles/spring-session-demonstration`](http://java.dzone.com/articles/spring-session-demonstration)，这是一个由两个 Web 服务器、一个用于存储用户会话的 Redis 和一个用于负载均衡的 Nginx 实例组成的 Spring Boot 应用的很好的例子。当然，关于 Docker Swarm 还有很多值得学习的地方，它可以让你通过简单的命令来扩展你的应用，还有 Docker Machine，它可以在任何机器上为你创建 Docker 主机，包括云提供商。

Google 的 Kubernetes 和 Apache Mesos 也是 Docker 容器大大受益的分布式系统的很好的例子。

# 单页应用

今天大多数的 Web 应用都是用 JavaScript 编写的。Java 被放在后端，并且在处理数据和业务规则方面起着重要作用。然而，现在很多 GUI 的工作都是在客户端进行的。

这在响应性和用户体验方面有很好的原因，但这些应用增加了额外的复杂性。

开发人员现在必须精通 Java 和 JavaScript，而且一开始可能会对各种框架感到有些不知所措。

## 参与者

如果你想深入了解 JavaScript，我强烈推荐 Dave Syer 的 Spring 和 AngularJS 教程，可在[`spring.io/guides/tutorials/spring-security-and-angular-js`](https://spring.io/guides/tutorials/spring-security-and-angular-js)上找到。

选择 JavaScript MVC 框架也可能有些困难。多年来，AngularJS 一直受到 Java 社区的青睐，但人们似乎正在远离它。欲了解更多信息，请访问[`gist.github.com/tdd/5ba48ba5a2a179f2d0fa`](https://gist.github.com/tdd/5ba48ba5a2a179f2d0fa)。

其他选择包括以下内容：

+   **BackboneJS**：这是一个非常简单的 MVC 框架，建立在 Underscore 和 jQuery 之上。

+   **Ember**：这是一个全面的系统，提供了更多与数据交互的便利设施。

+   **React**：这是 Facebook 的最新项目。它有一种处理视图的新而非常有趣的哲学。它的学习曲线相当陡峭，但在设计 GUI 框架方面，它是一个非常有趣的系统。

React 是我目前最喜欢的项目。它让你专注于视图，其单向数据流使得应用程序的状态易于理解。然而，它仍处于 0.13 版本。这使得它非常有趣，因为充满活力的社区总是提出新的解决方案和想法，但也有些令人不安，因为即使经过两年多的开源开发，前方的道路似乎仍然很长。请访问[`facebook.github.io/react/blog/2014/03/28/the-road-to-1.0.html`](https://facebook.github.io/react/blog/2014/03/28/the-road-to-1.0.html)了解有关“通往 1.0 版本的道路”的信息。

## 未来

我看到很多 Java 开发人员抱怨 JavaScript 的宽松性，并且很难处理它不是一种强类型语言的事实。

还有其他选择，比如**Typescript**（[`www.typescriptlang.org/`](http://www.typescriptlang.org/)），非常有趣并提供了我们 Java 开发人员一直用来简化生活的东西：接口、类、IDE 中的有用支持和自动完成。

很多人押注下一个版本（2.0）的 Angular 将会彻底改变一切。我认为这是最好的。他们与微软的 Typescript 团队的合作真的很独特。

大多数 JEE 开发人员听到 ECMAScript 的一个重大新功能是装饰器时会微笑，这允许开发这个新框架的装饰器是一种注解机制：

### 注意

要了解注解和装饰器之间的区别，请访问[`blog.thoughtram.io/angular/2015/05/03/the-difference-between-annotations-and-decorators.html`](http://blog.thoughtram.io/angular/2015/05/03/the-difference-between-annotations-and-decorators.html)。

JavaScript 正在迅速发展，ECMAScript 6 具有许多有趣的功能，使其成为一种非常先进和复杂的语言。不要错过机会，在为时已晚之前查看[`github.com/lukehoban/es6features`](https://github.com/lukehoban/es6features)！

Web 组件规范也是一个改变游戏规则的因素。其目标是提供可重用的 UI 组件，React 团队和 Angular 2 团队都计划与其进行接口交互。谷歌已经在 Web 组件之上开发了一个名为 Polymer 的有趣项目，现在已经是 1.0 版本。

### 注意

请参阅[`ng-learn.org/2014/12/Polymer/`](http://ng-learn.org/2014/12/Polymer/)的文章，以了解更多关于这些项目的情况。

## 无状态

在处理 JavaScript 客户端时，依赖会话 cookie 并不是最佳选择。大多数应用程序选择完全无状态，并使用令牌识别客户端。

如果您想坚持使用 Spring Session，请查看`HeaderHttpSessionStrategy`类。它具有通过 HTTP 标头发送和检索会话的实现。可以在[`drissamri.be/blog/2015/05/21/spr`](https://drissamri.be/blog/2015/05/21/spr)找到示例。

# 总结

Spring 生态系统广泛，为现代 Web 应用程序开发人员提供了很多选择。

很难找到一个 Spring 项目没有解决的问题。

是时候说再见了！我希望您喜欢我们与 Spring MVC 的小旅程，并且它将帮助您愉快地开发并创建令人惊叹的项目，无论是在工作中还是在业余时间。

# 附录 A. 参考文献

这个学习路径已经为您准备好，使用 Spring MVC 框架创建企业级应用程序。它包括以下 Packt 产品：

+   *Spring Essentials, Shameer Kunjumohamed and Hamidreza Sattari*

+   *Spring MVC Cookbook, Alex Bretet*

+   *Mastering Spring MVC 4, Geoffroy Warins*

# 索引

## A

+   @After 注释 / @Before 和@After 注释

+   @AfterClass 注释 / @BeforeClass 和@AfterClass 注释

+   @AspectJ 注释

+   声明 / 声明@Aspect 注释

+   切入点 / 切入点

+   建议 / 建议

+   基于@AspectJ 注释的 AOP

+   关于 / 基于@AspectJ 注释的 AOP

+   验收测试

+   关于 / 我应该如何测试我的代码？, 验收测试, 编写验收测试

+   Gradle，配置 / Gradle 配置

+   使用 FluentLenium / 我们的第一个 FluentLenium 测试

+   使用 Groovy / 使我们的测试更加 Groovy

+   ACID（原子性，一致性，隔离性，持久性）

+   关于 / Spring 事务支持

+   ACID 属性

+   关于 / ACID 属性

+   参考链接 / ACID 属性

+   高级消息队列协议（AMQP）

+   任务，与之堆叠 / 准备就绪

+   任务，与之消耗 / 准备就绪

+   关于 / AMQP 还是 JMS？

+   由 pivotal 提供的 URL / pivotal 提供的 AMQP 简介

+   应用程序事件，发布 / 发布应用程序事件的更好方法

+   建议，@AspectJ 注释

+   关于 / 建议

+   注释 / 建议

+   @Around 建议 / @Around 建议

+   访问建议参数 / 访问建议参数

+   贫血领域模型

+   URL / 贫血领域模型

+   关于 / 贫血领域模型

+   angular-translate.js

+   用于客户端翻译 / 使用 angular-translate.js 进行客户端翻译

+   URL / 使用 angular-translate.js 进行客户端翻译

+   AngularJS

+   关于 / SPA 框架

+   用于设计客户端 MVC 模式 / Designing a client-side MVC pattern with AngularJS

+   URL / There's more...

+   URL，用于表单文档 / See also

+   AngularJS Controllers

+   关于 / AngularJS Controllers

+   双向 DOM-scope 绑定 / Bidirectional DOM-scope binding

+   AngularJS Directives

+   关于 / AngularJS directives

+   ng-repeat / ng-repeat

+   ng-if / ng-if

+   AngularJS factories

+   关于 / AngularJS factories

+   依赖注入 / Dependency injection

+   AngularJS JavaScript library

+   URL / Setting up the DOM and creating modules

+   angular 路由

+   关于 / Angular routes

+   Angular UI

+   使用 Bootstrap 分页 / Bootstrap pagination with the Angular UI

+   注解定义的控制器

+   @Controller annotation / @Controller

+   @RequestMapping annotation / @RequestMapping

+   注解

+   定义 / Auditing with Spring Data

+   @CreatedBy / Auditing with Spring Data

+   @CreatedDate / Auditing with Spring Data

+   @LastModifiedBy / Auditing with Spring Data

+   @LastModifiedDate / Auditing with Spring Data

+   AssertFalse / On-field constraint annotations

+   AssertFalse.List / On-field constraint annotations

+   AssertTrue / On-field constraint annotations

+   AssertTrue.List / On-field constraint annotations

+   DecimalMax / On-field constraint annotations

+   DecimalMax.List / On-field constraint annotations

+   DecimalMin / On-field constraint annotations

+   DecimalMin.List / On-field constraint annotations

+   Digits / On-field constraint annotations

+   Digits.List / On-field constraint annotations

+   Future / On-field constraint annotations

+   Future.List / On-field constraint annotations

+   Max / On-field constraint annotations

+   Max.List / On-field constraint annotations

+   Min / On-field constraint annotations

+   Min.List / On-field constraint annotations

+   NotNull / On-field constraint annotations

+   NotNull.列表/ 现场约束注释

+   过去/ 现场约束注释

+   过去.列表/ 现场约束注释

+   模式/ 现场约束注释

+   模式.列表/ 现场约束注释

+   大小/ 现场约束注释

+   大小.列表/ 现场约束注释

+   Apache Commons 日志桥接/ Apache Commons 日志桥接

+   Apache HTTP

+   URL/ 还有更多...

+   替代方案/ 替代方案 Apache HTTP

+   Apache HTTP 配置

+   代理 Tomcat/ 配置 Apache HTTP 代理您的 Tomcat, 如何做..., 它是如何工作的...

+   关于/ Apache HTTP 配置

+   虚拟主机/ 虚拟主机

+   mod_proxy 模块/ mod_proxy 模块

+   ProxyPassReverse/ ProxyPassReverse

+   mod_alias 模块/ mod_alias 模块

+   Tomcat 连接器/ Tomcat 连接器

+   Apache HTTP 文档

+   URL/ 可扩展模型

+   Apache HTTP 服务器

+   在 MS Windows 上安装，URL/ 如何做...

+   在 Linux/Mac OS 上安装，URL/ 如何做...

+   Apache JServ 协议（AJP）连接器/ AJP 连接器

+   API

+   使用 Swagger 记录/ 使用 Swagger 记录和公开 API, 如何做...

+   使用 Swagger 公开/ 使用 Swagger 记录和公开 API, 如何做...

+   公开的元数据/ 公开的元数据

+   API 端点

+   为 Taskify 应用程序构建/ 为 Taskify 应用程序构建 API 端点

+   UserController.java/ UserController.java

+   TaskController.java/ TaskController.java

+   API 服务器应用

+   构建/ 构建 API 服务器应用

+   项目，设置/ 设置和配置项目

+   定义用户和任务/ 定义模型定义-用户和任务

+   API 版本控制

+   关于/ API 版本控制

+   参考链接/ API 版本控制

+   应用程序

+   日志记录，使用 Log4j2/ 使用 Log4j2 进行现代应用程序日志记录, 如何做...

+   应用程序缓存

+   创建/ 应用程序缓存

+   参考链接/ 应用程序缓存

+   失效/ 缓存失效

+   分布式缓存/ 分布式缓存

+   参数解析器

+   JPA2 标准 API / JPA2 标准 API 和 Spring Data JPA 规范

+   Spring Data JPA 规范 / JPA2 标准 API 和 Spring Data JPA 规范

+   SpecificationArgumentResolver/ SpecificationArgumentResolver

+   面向方面的编程（AOP）

+   关于/ 面向方面的编程

+   静态 AOP/ 静态和动态 AOP

+   动态 AOP / 静态和动态 AOP

+   概念/ AOP 概念和术语

+   术语/ AOP 概念和术语

+   Spring AOP / Spring AOP-定义和配置样式

+   基于 XML 模式的 AOP/ 基于 XML 模式的 AOP

+   @AspectJ 基于注释的 AOP/ @AspectJ 基于注释的 AOP

+   异步请求处理

+   在 Spring MVC 中/ Spring MVC 中的异步请求处理

+   异步方法

+   使用/ 异步方法

+   参考链接/ 异步方法

+   Atomikos

+   URL/ 全局与本地事务

+   认证

+   关于/ 认证

+   测试/ 测试认证

+   AuthenticationManager 接口/ AuthenticationManager 接口

+   授权

+   关于/ 授权

+   授权的 URL

+   认证/ 授权的 URL

+   授权用户

+   认证/ 授权用户

## B

+   @Before 注释 / @Before 和@After 注释

+   @BeforeClass 注释 / @BeforeClass 和@AfterClass 注释

+   BackboneJS

+   关于/ 玩家

+   基本身份验证

+   URL/ 基本身份验证

+   关于/ 基本身份验证

+   配置/ 基本身份验证

+   用于授权用户/ 授权用户

+   用于授权的 URL/ 授权的 URL

+   使用 thymeleaf 安全标签/ Thymeleaf 安全标签

+   BasicAuthenticationFilter

+   关于/ BasicAuthenticationFilter

+   使用 authenticationEntryPoint/ 使用 authenticationEntryPoint

+   基本方案

+   认证/ 通过基本方案进行认证, 如何做...

+   Spring 安全命名空间/ Spring 安全命名空间

+   AuthenticationManager 接口/ AuthenticationManager 接口

+   Spring 安全参考/ 在 Spring 安全参考中

+   记住我 cookie/功能/ 记住我 cookie/功能

+   bean 定义配置文件

+   使用/ 使用 bean 定义配置文件

+   bean 依赖项

+   注入/ 注入 bean 依赖项

+   基于构造函数的依赖注入/ 基于构造函数的依赖注入

+   基于 setter 的依赖注入/ 基于 setter 的依赖注入

+   BeanFactory 接口

+   关于/ Spring IoC 容器

+   bean 生命周期

+   连接/ 连接到 bean 生命周期

+   InitializingBean，实现/ 实现 InitializingBean 和 DisposableBean

+   DisposableBean，实现/ 实现 InitializingBean 和 DisposableBean

+   @PostConstruct，对@Components 进行注释/ 在@Components 上注释@PostConstruct 和@PreDestroy

+   @PreDestroy，对@Components 进行注释/ 在@Components 上注释@PostConstruct 和@PreDestroy

+   init-method 和 destroy-method 属性/ <bean/>的 init-method 和 destroy-method 属性

+   beans

+   关于/ Spring IoC 容器, 详细介绍 Beans

+   定义/ Bean 定义

+   实例化/ 实例化 beans

+   实例化，使用构造函数/ 使用构造函数

+   实例化，使用静态工厂方法/ 使用静态工厂方法

+   实例化，使用实例工厂方法/ 使用实例工厂方法

+   使用命名空间快捷方式进行更清晰的 bean 定义/ 使用命名空间快捷方式进行更清晰的 bean 定义

+   列表，作为依赖项进行连线/ 将列表作为依赖项进行连线

+   映射，作为依赖项进行连线/ 将映射作为依赖项进行连线

+   依赖项，自动装配/ 自动装配依赖项

+   作用域/ Bean 作用域

+   bean 验证

+   用于验证资源/ 准备工作, 如何做…

+   使用 Spring 验证器/ 使用 Spring 验证器

+   JSR-303/JSR-349 bean 验证 / 使用 JSR-303/JSR-349 Bean 验证

+   ValidationUnits 实用程序/ ValidationUtils

+   创建自定义验证器 / 创建自定义验证器

+   参考链接 / Spring 关于验证的参考

+   绑定请求

+   关于 / 绑定请求和编组响应, 准备就绪, 如何做..., 它是如何工作的...

+   Bitronix

+   URL / 全局与本地事务

+   样板逻辑

+   抽象 / 样板逻辑的抽象

+   自动生成的 ID，提取 / 提取自动生成的 ID

+   bookapp-rest 应用程序

+   URL / 我们的 MessageSource bean 定义

+   Bootstrap

+   响应式单页 Web 设计，设置 / 使用 Bootstrap 设置和自定义响应式单页 Web 设计, 如何做..., 安装 Bootstrap 主题

+   亮点 / Bootstrap 亮点

+   URL / 还有更多...

+   Bootstrap 组件

+   导航栏 / 导航栏

+   英雄单元 / 英雄单元

+   警报 / 警报

+   徽章和标签 / 徽章和标签

+   Bootstrap CSS 实用程序

+   统一按钮 / 统一按钮

+   图标 / 图标

+   表格 / 表格

+   Bootstrap 分页

+   使用 Angular UI / 使用 Angular UI 的 Bootstrap 分页

+   URL / 使用 Angular UI 的 Bootstrap 分页

+   Bootstrap 脚手架

+   关于 / Bootstrap 脚手架

+   网格系统和响应式设计 / 网格系统和响应式设计

+   定义列 / 定义列

+   列，偏移 / 偏移和嵌套

+   嵌套 / 偏移和嵌套

+   流体网格 / 流体网格

+   Bootstrap 主题

+   自定义 / 自定义 Bootstrap 主题

+   安装 / 主题安装

+   经纪人通道 / Spring 4 中 STOMP over WebSocket 和回退选项

+   BSON（二进制 JSON）格式

+   关于 / Spring Data MongoDB

+   Maven 构建生命周期

+   关于 / Maven 的构建生命周期

+   清洁生命周期 / 清洁生命周期

+   默认生命周期 / 默认生命周期

+   插件 / 插件目标

+   内置生命周期 / 内置生命周期绑定

+   Maven 命令 / 关于 Maven 命令

## C

+   @ComponentScan 注释 / 创建一个简单的 WebSocket 应用程序

+   @Configuration 注释 / 创建一个简单的 WebSocket 应用程序

+   @ContextConfiguration 注释 / @ContextConfiguration 注释, 还有更多…

+   @ControllerAdvice

+   使用@ControllerAdvice 进行全局异常处理 / 使用@ControllerAdvice 进行全局异常处理

+   支持 ResponseEntityExceptionHandler 类 / 支持 ResponseEntityExceptionHandler 类

+   统一的错误响应对象 / 统一的错误响应对象

+   @Controller 注释 / @Controller

+   缓存控制

+   关于 / 另请参阅

+   缓存控制

+   关于 / 缓存控制

+   配置 / 缓存控制

+   货物

+   与集成测试 / 使用 Cargo，Rest-assured 和 Maven failsafe 进行集成测试, 如何做…, 它是如何工作的…

+   Codehaus Cargo / Code Cargo

+   Maven 插件 / Cargo Maven 插件

+   关于 / 关于 Cargo

+   URL / 关于 Cargo

+   Cargo Maven 插件

+   关于 / Cargo Maven 插件

+   Maven 阶段，绑定到 / 绑定到 Maven 阶段

+   现有的 Tomcat 实例，使用 / 使用现有的 Tomcat 实例

+   级联属性

+   关于 / 级联属性

+   证书签名请求（CSR）

+   URL / 关于 SSL 和 TLS

+   检查点

+   关于 / 检查点

+   清理命令 / 清理

+   清理生命周期

+   预清理阶段 / 清理生命周期

+   清理阶段 / 清理生命周期

+   后清理阶段 / 清理生命周期

+   客户端表单

+   使用 HTML5/AngularJS 进行验证 / 使用 HTML5 AngularJS 验证客户端表单, 如何做…, 它是如何工作的…

+   控制变量 / 表单中的控制变量

+   状态转换 / 表单状态转换和样式

+   样式 / 表单状态转换和样式

+   客户端表单，验证约束

+   必需的 / 必需的

+   最小/最大长度 / 最小/最大长度

+   正则表达式模式 / 正则表达式模式

+   客户端 MVC 模式

+   设计，使用 AngularJS / 使用 AngularJS 设计客户端 MVC 模式

+   客户端验证，配置文件页面

+   启用 / 客户端验证

+   参考链接 / 客户端验证

+   Cloud Foundry

+   关于 / Cloud Foundry

+   URL / Cloud Foundry

+   Cloud Foundry CLI 工具

+   安装 / 安装 Cloud Foundry CLI 工具

+   URL / 安装 Cloud Foundry CLI 工具

+   cloudstreetmarket-parent

+   关于 / 准备就绪

+   代码测试

+   好处 / 为什么我应该测试我的代码？

+   单元测试 / 我应该如何测试我的代码？

+   验收测试 / 我应该如何测试我的代码？

+   组件类型注释

+   @Component / 组件类型注释

+   @Service / 组件类型注释

+   @Repository / 组件类型注释

+   @Controller / 组件类型注释

+   @RestController / 组件类型注释

+   配置元数据，依赖注入

+   关于 / 配置元数据

+   基于 XML 的配置元数据 / 基于 XML 的配置元数据

+   基于注释的配置元数据 / 基于注释的配置元数据

+   基于 XML 的与基于注释的配置 / 基于 XML 与基于注释的配置

+   组件类型注释 / 组件类型注释

+   基于 Java 的配置元数据 / 基于 Java 的配置元数据

+   JSR 330 标准注释 / JSR 330 标准注释

+   基于构造函数的依赖注入

+   关于 / 基于构造函数还是基于 setter 的依赖注入 - 哪个更好？

+   构造函数注入

+   关于 / 将配置文件放入会话中

+   URL / 将配置文件放入会话中

+   容器级默认初始化和销毁方法

+   关于 / 容器级默认初始化方法和默认销毁方法

+   容器管理的事务（CMT）

+   关于 / Spring 事务的相关性

+   内容协商

+   配置 / 如何做..., 它是如何工作的...

+   XML 编组，支持 / 支持 XML 编组

+   ContentNegotiationManager

+   与 ContentNegotiationManager 的协商策略

+   接受头 / 接受头

+   URL 路径中的文件扩展名后缀 / URL 路径中的文件扩展名后缀

+   请求参数 / 请求参数

+   Java 激活框架 / Java 激活框架

+   ContentNegotiationManagerFactoryBean JavaDoc

+   关于 / ContentNegotiationManagerFactoryBean JavaDoc

+   内容

+   为 REST 国际化 / 为 REST 国际化消息和内容

+   动态翻译，后端实现 / 后端

+   动态翻译，前端实现 / 前端, 它是如何工作的...

+   持续集成

+   参考链接 / 为什么我应该测试我的代码？

+   控制器

+   使用简单 URL 映射进行配置 / 使用简单 URL 映射配置控制器, 如何做...

+   控制器方法处理程序签名

+   关于 / 控制器方法处理程序签名

+   支持的方法参数类型 / 支持的方法参数类型

+   方法参数的支持注解 / 方法参数的支持注解

+   支持的返回类型 / 支持的返回类型

+   控制器

+   关于 / 详细的控制器

+   使用@RequestMapping 映射请求 URL / 使用@RequestMapping 映射请求 URL

+   使用@PathVariable 注解的 URI 模板模式 / 使用@PathVariable 注解的 URI 模板模式

+   使用@RequestParam 注解绑定参数 / 使用@RequestParam 注解绑定参数

+   请求处理程序方法参数 / 请求处理程序方法参数

+   请求处理程序方法返回类型 / 请求处理程序方法返回类型

+   模型属性，设置 / 设置模型属性

+   为 JSON 和 XML 媒体构建 RESTful 服务 / 为 JSON 和 XML 媒体构建 RESTful 服务

+   使用 RestController 构建 RESTful 服务 / 使用 RestController 构建 RESTful 服务

+   授权 / 在服务和控制器上进行授权, 如何做...

+   控制变量，客户端表单

+   修改/未修改状态 / 修改/未修改状态

+   $error 属性 / 错误

+   ConversionService API / ConversionService API

+   CookieHttpSessionStrategy / CookieHttpSessionStrategy

+   核心模块

+   创建/ 为什么我们创建核心模块？

+   创建读取更新删除（CRUD）

+   关于/ Level 2-HTTP 动词

+   跨站点请求伪造（csrf）/ 我们的<http>配置

+   跨站点请求伪造（CSRF）攻击

+   关于/ 认证

+   跨站点请求伪造（CSRF）

+   关于/ 授权的 URL

+   URL/ 授权的 URL

+   自定义约束

+   参考链接/ 特定实现约束

+   自定义错误页面

+   创建/ 自定义错误页面

+   自定义范围

+   创建/ 创建自定义范围

+   自定义验证器

+   URL/ 创建自定义验证器

## D

+   DAO 支持

+   关于/ DAO 支持和@Repository 注释

+   数据

+   使用 OAuth 从第三方 API 检索/ 使用 OAuth 从第三方 API 检索数据, 如何做..., 它是如何工作的...

+   Yahoo!，财务数据/ Yahoo!财务数据介绍

+   图形，生成/显示/ 图形生成/显示

+   财务数据，拉取/ 财务数据是如何拉取/刷新的？

+   财务数据，刷新/ 财务数据是如何拉取/刷新的？

+   调用第三方服务/ 调用第三方服务

+   现有的 API 提供商/ Spring Social-现有的 API 提供商

+   数据供应实现

+   通过接口注入服务/ 通过接口注入服务

+   虚拟实现，选择/ Spring 如何选择虚拟实现？

+   在视图层中使用的 DTO/ 在视图层中使用的 DTO

+   虚拟服务实现/ 虚拟服务实现

+   数据访问对象（DAO）/ 基于 XML 的配置元数据

+   数据库迁移

+   自动化，使用 FlyWay/ 使用 FlyWay 自动化数据库迁移, 如何做...

+   数据源

+   配置/ 配置数据源

+   参考/ 配置数据源

+   数据源

+   关于/ Spring 管理的数据源 bean

+   数据传输对象（DTO）

+   关于/ 个人资料页面-表单

+   声明式事务管理

+   关于/ 声明式事务管理

+   代理模式/ 事务模式-代理和 AspectJ

+   AspectJ 模式 / 事务模式-代理和 AspectJ

+   定义事务行为 / 定义事务行为

+   回滚规则，设置 / 设置回滚规则

+   默认生命周期

+   验证 / 默认生命周期

+   初始化 / 默认生命周期

+   generate-sources / 默认生命周期

+   process-sources / 默认生命周期

+   generate-resources / 默认生命周期

+   process-resources / 默认生命周期

+   编译 / 默认生命周期

+   process-classes / 默认生命周期

+   generate-test-sources / 默认生命周期

+   process-test-sources / 默认生命周期

+   generate-test-resources / 默认生命周期

+   process-test-resources / 默认生命周期

+   test-compile / 默认生命周期

+   process-test-classes / 默认生命周期

+   测试 / 默认生命周期

+   准备包 / 默认生命周期

+   打包 / 默认生命周期

+   pre-integration-test / 默认生命周期

+   integration-test / 默认生命周期

+   post-integration-test / 默认生命周期

+   验证 / 默认生命周期

+   install / 默认生命周期

+   部署 / 默认生命周期

+   依赖注入 / 依赖注入

+   依赖注入（DI） / Spring 框架模块

+   关于 / 依赖注入, Spring 框架带来了什么？

+   Spring IoC 容器 / Spring IoC 容器

+   配置元数据 / 配置元数据

+   依赖注入，带有作用域的 bean

+   关于 / 带有作用域的 bean 的依赖注入

+   可部署模块

+   名称，选择 / 我们如何选择可部署模块的名称？

+   部署

+   关于 / 部署

+   Docker / Docker

+   开发环境

+   设置 / 设置开发环境

+   Dispatcher Servlet

+   架构 / DispatcherServlet

+   DispatcherServlet

+   关于 / DispatcherServlet 解释

+   使用 WebApplicationContext/ WebApplicationContext-Web 的 ApplicationContext

+   支持豆/ 支持 DispatcherServlet 的豆和它们的角色

+   支持的豆/ 支持 DispatcherServlet 的豆和它们的角色/ DispatcherServlet-Spring MVC 入口点

+   分布式缓存

+   配置/ 分布式缓存

+   分布式会话

+   关于/ 分布式会话

+   设置/ 分布式会话

+   DNS

+   URL/ 还有更多...

+   DNS 配置/ DNS 配置或主机别名

+   DNS 记录

+   版本/ 在生产中-编辑 DNS 记录

+   Docker

+   关于/ Docker

+   URL/ Docker

+   参考链接/ Docker

+   文档

+   使用 Swagger/ 使用 Swagger 进行文档编制

+   文档对象模型（DOM）/ 每个 HTML 文档一个应用程序

+   DOM

+   设置/ 设置 DOM 和创建模块

+   DOM 范围绑定

+   双向/ 双向 DOM 范围绑定

+   领域驱动设计（DDD）

+   关于/ 贫血领域模型

+   领域对象和实体

+   关于/ 领域对象和实体

+   查询解析方法/ 查询解析方法

+   @Query 注释，使用/ 使用@Query 注释

+   Spring Data web 支持扩展/ Spring Data web 支持扩展

+   审计，使用 Spring Data/ 使用 Spring Data 进行审计

+   领域对象安全（ACLs）

+   URL/ 领域对象安全（ACLs）

+   DTO

+   转换为 Spring HATEOAS 资源/ 将 DTO 转换为 Spring HATEOAS 资源, 如何做..., 它是如何工作的...

## E

+   @EnableAutoConfiguration 注释/ 创建一个简单的 WebSocket 应用程序

+   Eclipse

+   为 Java 8 配置/ 为 Java 8，Maven 3 和 Tomcat 8 配置 Eclipse, 如何做...

+   为 Maven 3 配置/ 为 Java 8，Maven 3 和 Tomcat 8 配置 Eclipse, 如何做...

+   为 Tomcat 8 配置/ 为 Java 8，Maven 3 和 Tomcat 8 配置 Eclipse, 如何做...

+   eclipse.ini 文件/ eclipse.ini 文件

+   -vm 选项，设置/ 设置-vm 选项

+   JVM 参数，自定义/ 自定义 JVM 参数

+   JDK 兼容级别，修改/ 更改 JDK 兼容级别

+   Maven，配置/ 配置 Maven

+   存储库管理器/ 存储库管理器

+   Tomcat 8，集成/ Eclipse 中的 Tomcat 8

+   URL/ 还有更多...

+   GIT，配置/ 在 Eclipse 中配置 GIT, 它是如何工作的...

+   Eclipse.ini 文件

+   URL/ 还有更多...

+   eclipse.ini 文件

+   关于/ eclipse.ini 文件

+   URL/ eclipse.ini 文件

+   Eclipse IDE

+   需要/ 为什么要使用 Eclipse IDE？

+   下载，适用于 Java EE 开发人员/ 如何做...

+   安装，适用于 Java EE 开发人员/ 如何做..., Eclipse for Java EE developers

+   URL/ 如何做...

+   JVM，选择/ 选择 JVM

+   Java SE 8/ Java SE 8

+   EJB3 实体

+   定义/ 准备就绪, 如何做..., 它是如何工作的...

+   要求/ 实体要求

+   模式，映射/ 映射模式

+   继承，定义/ 定义继承

+   关系，定义/ 定义关系

+   嵌入式数据库

+   使用/ 使用嵌入式数据库

+   EmbeddedServletContainerCustomizer 接口

+   关于/ 处理文件上传错误

+   URL/ 处理文件上传错误

+   Ember

+   关于/ 玩家

+   Ember.js

+   关于/ SPA 框架, 介绍 Ember.js

+   Ember 应用程序

+   解剖学/ Ember 应用程序的解剖学

+   路由器/ 路由器

+   路由或路由处理程序/ 路由或路由处理程序

+   模板/ 模板

+   组件/ 组件

+   模型/ 模型

+   控制器/ 控制器

+   输入助手/ 输入助手

+   自定义助手/ 自定义助手

+   初始化程序/ 初始化程序

+   服务/ 服务

+   Ember CLI

+   关于/ 介绍 Ember.js, 使用 Ember CLI

+   使用/ 使用 Ember CLI

+   features / 使用 Ember CLI

+   setting up / 设置 Ember CLI

+   commands / 使用 Ember CLI 命令入门

+   project structure / Ember 项目结构

+   POD structure / 使用 POD 结构

+   Ember CLI 命令

+   about / 使用 Ember CLI 命令入门

+   ember / 使用 Ember CLI 命令入门

+   ember new <appname> / 使用 Ember CLI 命令入门

+   ember init / 使用 Ember CLI 命令入门

+   ember build / 使用 Ember CLI 命令入门

+   ember server (or serve) / 使用 Ember CLI 命令入门

+   ember generate <generatortype> <name> <options> / 使用 Ember CLI 命令入门

+   ember destroy <generatortype> <name> <options> / 使用 Ember CLI 命令入门

+   ember test / 使用 Ember CLI 命令入门

+   ember install <addon-name> / 使用 Ember CLI 命令入门

+   Ember Data

+   about / 介绍 Ember.js

+   data, persisting with / 使用 Ember Data 持久化数据

+   DS.Model / 使用 Ember Data 持久化数据

+   DS.Store / 使用 Ember Data 持久化数据

+   DS.Adapter / 使用 Ember Data 持久化数据

+   DS.Serializer / 使用 Ember Data 持久化数据

+   architecture / Ember Data 架构

+   models, building / 定义模型

+   model relationships, defining / 定义模型关系

+   Ember 开发堆栈

+   about / 介绍 Ember.js

+   Ember Inspector

+   about / 介绍 Ember.js

+   Ember object model

+   about / 理解 Ember 对象模型

+   types (classes), declaring / 声明类型（类）和实例

+   instances, declaring / 声明类型（类）和实例

+   properties, accessing / 访问和修改属性

+   properties, mutating / 访问和修改属性

+   computed properties / 计算属性

+   property observers / 属性观察者

+   collections, working with / 处理集合

+   Ember.Array / 使用集合

+   Ember.ArrayProxy / 使用集合

+   Ember.MutableArray / 使用集合

+   Ember.Enumerable / 使用集合

+   Ember.NativeArray / 使用集合

+   企业版（EE）/ AMQP 还是 JMS？

+   企业集成（EAI）

+   关于 / Spring 子项目

+   企业 JavaBean（EJB）

+   关于 / 介绍

+   企业 JavaBean（EJB）

+   关于 / Spring 事务的相关性

+   实体

+   关于 / 实体的好处

+   好处 / 实体的好处

+   实体，OAuth2

+   资源所有者 / OAuth2 授权框架

+   客户端或第三方应用程序 / OAuth2 授权框架

+   授权服务器 / OAuth2 授权框架

+   资源服务器 / OAuth2 授权框架

+   实体管理器

+   关于 / 实体管理器及其持久性上下文

+   持久性上下文 / 实体管理器及其持久性上下文

+   EntityManagerFactory bean

+   关于 / EntityManagerFactory bean 及其持久性单元

+   厄尔朗

+   URL / 如何做...

+   错误消息

+   翻译 / 翻译错误消息

+   ETag

+   关于 / 另请参阅

+   ETags

+   关于 / ETags

+   生成 / ETags

+   使用 / ETags

+   异常处理

+   关于 / 状态码和异常处理

+   异常

+   在 Spring 数据层处理 / 在 Spring 数据层处理异常

+   全局处理 / 准备工作, 如何做..., 工作原理...

## F

+   备用控制器

+   配置，使用 ViewResolver / 使用 ViewResolver 配置备用控制器, 如何做...

+   URI 模板模式 / URI 模板模式

+   ViewResolvers / ViewResolvers

+   备用选项

+   使用 / Spring 4 中 STOMP over WebSocket 和备用选项

+   Fastboot

+   关于 / 介绍 Ember.js

+   feedEk jQuery 插件

+   URL / 创建响应式内容

+   FetchType 属性

+   关于 / FetchType 属性

+   文件上传

+   关于 / 上传文件

+   个人资料图片，上传/ 上传文件

+   上传的图片，在网页上显示/ 将图像写入响应

+   上传属性，管理/ 管理上传属性

+   上传的图片，显示/ 显示上传的图片

+   错误，处理/ 处理文件上传错误

+   实现/ 将其放在一起

+   检查点/ 检查点

+   文件上传

+   处理/ 处理文件上传

+   过滤

+   关于/ 如何做..., 它是如何工作的...

+   FluentLenium

+   用于验收测试/ 我们的第一个 FluentLenium 测试

+   关于/ 我们的第一个 FluentLenium 测试

+   URL/ 我们的第一个 FluentLenium 测试

+   页面对象/ 使用 FluentLenium 的页面对象

+   FlyWay

+   用于自动化数据库迁移/ 使用 FlyWay 自动化数据库迁移, 如何做...

+   命令/ 有限的命令数量

+   maven 插件/ 关于 Flyway Maven 插件

+   配置参数，URL/ 关于 Flyway Maven 插件

+   官方文档/ 官方文档

+   GitHub 仓库，URL/ 官方文档

+   FlyWay，命令

+   关于/ 有限的命令数量

+   迁移/ 迁移

+   清洁/ 清洁

+   信息/ 信息

+   验证/ 验证

+   基线/ 基线

## G

+   垃圾收集

+   参考链接/ 还有更多...

+   GDAXI 指数代码

+   URL/ 如何做...

+   Geb

+   用于集成测试/ 使用 Geb 进行集成测试

+   关于/ 使用 Geb 进行集成测试

+   页面对象/ 使用 Geb 的页面对象

+   参考链接/ 使用 Geb 的页面对象

+   Git

+   关于/ 上传文件

+   空目录/ 上传文件

+   GIT

+   安装/ 下载和安装 GIT

+   下载/ 下载和安装 GIT

+   URL/ 下载和安装 GIT

+   在 Eclipse 中配置/ 在 Eclipse 中配置 GIT

+   全局事务

+   全局与本地事务/ 全局与本地事务

+   参考链接/ 全局与本地事务

+   Google 协议缓冲区

+   URL / 提供的 HttpMessageConverters

+   Gradle

+   配置 / Gradle 配置

+   运行 / Gradle

+   URL / Gradle

+   GrantedAuthority 接口

+   关于 / GrantedAuthority 接口

+   Groovy

+   接受测试/ 使我们的测试更加灵活

+   关于 / 使我们的测试更加灵活

+   URL / 使我们的测试更加灵活

+   Groovy 开发工具包（GDK）

+   关于 / 使我们的测试更加灵活

+   Gzipping

+   关于 / Gzipping

+   参考链接 / Gzipping

## H

+   HandlerMapping

+   关于 / DispatcherServlet

+   HAProxy

+   URL / 替代 Apache HTTP 的选择

+   标题

+   参考链接 / 另请参阅

+   堆内存

+   年轻一代 / 自定义 JVM 参数

+   旧一代 / 自定义 JVM 参数

+   Heroku

+   关于 / Heroku

+   Web 应用程序，部署 / 在 Heroku 上部署您的 Web 应用程序

+   命令行工具，安装 / 安装工具

+   URL / 安装工具

+   Web 应用程序，设置 / 设置应用程序

+   运行 Gradle / Gradle

+   运行 Procfile / Procfile

+   配置文件，创建 / 一个 Heroku 配置文件

+   Web 应用程序，执行 / 运行您的应用程序

+   激活 Redis / 激活 Redis

+   Heroku Redis 附加组件

+   URL / 激活 Redis

+   Hibernate 查询语言（HQL）

+   关于 / 使用 JPQL

+   HikariCP 数据源

+   关于 / 另请参阅

+   URL / 另请参阅

+   主机

+   别名 / 主机的别名

+   选择 / 选择您的主机

+   Cloud Foundry / Cloud Foundry

+   OpenShift / OpenShift

+   Heroku / Heroku

+   主机别名 / DNS 配置或主机别名

+   HTML5/AngularJS

+   客户端表单，验证 / 使用 HTML5 AngularJS 验证客户端表单, 如何做..., 它是如何工作的...

+   HTML 文档

+   模块自动引导 / 模块自动引导

+   模块自动引导，手动 / 手动模块引导

+   HTTP/1.1 规范

+   参考链接 / HTTP/1.1 规范-RFC 7231 语义和内容

+   必要条件/【基本要求】

+   安全方法/【安全和幂等方法】

+   幂等方法/【安全和幂等方法】

+   特定于方法的约束/【其他特定于方法的约束】

+   HTTP 代码

+   关于/【有用的 HTTP 代码】

+   URL/【有用的 HTTP 代码】

+   HTTP 连接器/【HTTP 连接器】

+   URL/【另请参阅】

+   httpie

+   关于/【httpie】

+   HttpMessageConverters

+   使用/【HttpMessageConverters】

+   本机 HttpMessageConverters/【提供的 HttpMessageConverters】

+   MappingJackson2HttpMessageConverter，使用/【使用 MappingJackson2HttpMessageConverter】

+   HTTP 方法

+   REST 处理程序，扩展到/【将 REST 处理程序扩展到所有 HTTP 方法】，【如何做…】，【它是如何工作的…】

+   HTTP 会话

+   配置文件，存储/【将配置文件放入会话中】

+   关于/【将配置文件放入会话中】

+   HTTP 状态码/【HTTP 状态码】

+   HTTP 动词

+   获取/【级别 2 - HTTP 动词】

+   头/【级别 2 - HTTP 动词】

+   删除/【级别 2 - HTTP 动词】

+   放置/【级别 2 - HTTP 动词】

+   发布/【级别 2 - HTTP 动词】

+   PATCH/【级别 2 - HTTP 动词】

+   选项/【级别 2 - HTTP 动词】

+   超媒体驱动的 API

+   链接，构建/【为超媒体驱动的 API 构建链接】，【如何做…】，【它是如何工作的…】，【构建链接】

+   资源装配器/【资源装配器】

+   PagedResourcesAssembler/【PagedResourcesAssembler】

+   EntityLinks/【EntityLinks】

+   ControllerLinkBuilder/【ControllerLinkBuilder】

+   @RequestMapping 中的正则表达式/【在@RequestMapping 中使用正则表达式】

+   超媒体作为应用程序状态的引擎（HATEOAS）

+   关于/【介绍】

+   超文本应用语言（HAL）

+   URL/【另请参阅】

+   应用程序状态的超文本作为引擎（HATEOAS）

+   关于/【级别 3 - 超媒体控件】

## 我

+   iconmonstr

+   URL/【将图像写入响应】

+   IDE（集成开发环境）

+   关于/【Spring 工具套件（STS）】

+   标识符

+   关于/【实体要求】

+   ID 暴露

+   URL/【另请参阅】

+   信息命令/ 信息

+   继承，EJB3 实体

+   定义/ 定义继承

+   单表策略/ 单表策略

+   按类策略/ 按类策略

+   继承，Maven 依赖

+   关于/ Maven 依赖的继承

+   基本继承/ 基本继承

+   管理继承/ 管理继承

+   集成测试

+   Spring Beans，注入/ 将 Spring Beans 注入集成测试, 如何做...

+   使用 Geb/ 使用 Geb 进行集成测试

+   拦截器

+   关于/ 更改区域设置

+   国际化（i18n）

+   关于/ 国际化

+   区域设置，修改/ 更改区域设置

+   应用文本，翻译/ 翻译应用文本

+   数据列表，在表单中处理/ 表单中的列表

+   物联网（IoT）/ 微服务架构

+   互联网服务提供商（ISP）/ DNS 配置或主机别名

+   控制反转（IOC）

+   关于/ Spring 框架带来了什么？

+   控制反转（IoC）容器/ Spring 框架模块

## J

+   Jackson 2.x 扩展组件

+   URL/ 提供的 HttpMessageConverters

+   JaCoCo

+   URL/ 另请参阅

+   jar

+   关于/ 准备就绪

+   jar 依赖

+   关于/ 准备就绪

+   jar 模块

+   选择名称/ 我们如何选择 jar 模块的名称？

+   Java 8

+   Eclipse，配置/ 准备就绪, 如何做...

+   流/ Java 8 流和 lambda 表达式

+   lambda 表达式/ Java 8 流和 lambda 表达式

+   Java 8 日期时间 API

+   参考链接/ 个人资料页面-表单

+   Java 激活框架（JAF）

+   关于/ 提供的 HttpMessageConverters

+   JavaBeans 组件/ 使用 JSP EL 呈现变量

+   JavaBeans 标准

+   URL/ 关于 JavaBeans 标准的更多信息

+   JavaDoc

+   URL/ WebContentGenerator 提供的更多功能, 使用 JAXB2 实现作为 XML 解析器

+   Java EE 教程

+   URL/ 使用 JSTL 呈现变量

+   Java 持久化 API（JPA）

+   关于/ 介绍

+   在 Spring 中配置 / 在 Spring 中配置 Java 持久化 API, 如何做..., 它是如何工作的...

+   Spring 管理的 DataSource bean / Spring 管理的 DataSource bean

+   EntityManagerFactory bean，配置 / EntityManagerFactory bean 及其持久化单元

+   持久化单元，配置 / EntityManagerFactory bean 及其持久化单元

+   Spring Data JPA，配置 / Spring Data JPA 配置

+   使用 / 利用 JPA 和 Spring Data JPA, 如何做..., 它是如何工作的...

+   Java 持久化查询语言（JPQL）

+   使用 / 使用 JPQL

+   参考链接 / 使用 JPQL

+   Java SE 8

+   使用 / Java SE 8

+   Java 服务器页面（JSP）

+   关于 / 解析 JSP 视图

+   Java 服务器页面标签库（JSTL）

+   关于 / 解析 JSP 视图

+   Java 服务器标签库（JSTL）

+   用于在视图中显示模型 / 在视图中显示模型，使用 JSTL, 如何做..., 它是如何工作的...

+   URL / 更多关于 JSTL

+   Java Util 日志适配器 / Java Util 日志适配器

+   JAXB2 实现

+   作为 XML 解析器使用 / 将 JAXB2 实现作为 XML 解析器使用

+   JDBC 操作

+   使用 Sql*类 / 使用 Sql*类进行 JDBC 操作

+   组件 / 使用 Sql*类进行 JDBC 操作

+   JdbcTemplate

+   方法 / JdbcTemplate

+   回调接口 / JdbcTemplate

+   NamedParameterJdbcTemplate / NamedParameterJdbcTemplate/ JdbcTemplate

+   JDK 8

+   安装 / 如何做...

+   JDK 兼容级别

+   修改 / 更改 JDK 兼容级别

+   JMS

+   关于 / AMQP 还是 JMS？

+   联接表继承策略

+   关于 / 另请参阅

+   JOTM

+   URL / 全局与本地事务

+   JPA（Java 持久化架构）

+   关于 / Spring Data JPA

+   JPA 实体

+   选择公开的策略 / 选择公开 JPA 实体的策略, 如何做..., 它是如何工作的...

+   REST CRUD 原则 / REST CRUD 原则

+   最小信息，暴露/ 暴露最小信息, 如果实体拥有关系

+   资源分离/ 资源分离

+   JSON 输出

+   自定义/ 自定义 JSON 输出

+   JSP EL

+   URL / 更多关于 JSP EL

+   JSP 表达式语言（JSP EL）/ 准备就绪

+   关于/ 准备就绪

+   JSP

+   Taglib 指令/ JSP 中的 Taglib 指令

+   JSP 标准标签库（JSTL）

+   关于/ 使用 JSTL 呈现变量

+   JSR-250

+   关于/ JSR-250 和遗留方法安全

+   JSR-303/JSR-349 bean 验证

+   使用/ 使用 JSR-303/JSR-349 Bean 验证

+   字段约束注释/ 字段约束注释

+   特定于实现的约束/ 特定于实现的约束

+   LocalValidator（可重用）/ LocalValidator（可重用）

+   JSR-310

+   URL / 自定义 JSON 输出

+   JSR-356

+   URL / 另请参阅

+   JTA（Java 事务 API）

+   关于/ Spring 事务的相关性

+   JUnit 规则

+   URL / JUnit 规则

+   JVM

+   选择/ 选择 JVM

+   JVM 参数

+   自定义/ 自定义 JVM 参数

## L

+   lambda，Java 8

+   关于/ Java 8 流和 lambda

+   布局

+   使用/ 使用布局

+   链接

+   为超媒体驱动的 API 构建/ 为超媒体驱动的 API 构建链接, 如何做…

+   链接！/ ResourceSupport 类

+   Liquibase

+   URL / 另请参阅

+   液体火

+   关于/ 介绍 Ember.js

+   负载均衡 WebSockets

+   URL / 另请参阅

+   LocaleResolver

+   用于国际化消息/ 使用 LocaleResolver

+   AcceptHeaderLocaleResolver / AcceptHeaderLocaleResolver

+   FixedLocaleResolver / FixedLocaleResolver

+   SessionLocaleResolver / SessionLocaleResolver

+   CookieLocaleResolver / CookieLocaleResolver

+   本地存储

+   URL / localStorage 的浏览器支持

+   Log4j 1.x API 桥接/ Log4j 1.x API 桥接

+   Log4j2

+   用于应用程序日志记录/ Log4j2 的现代应用程序日志记录, 如何做…

+   和其他日志框架/ Apache Log4j2 等其他日志框架

+   SLF4j, 案例 / SLF4j 的案例

+   迁移到 / 迁移到 log4j 2

+   API 和核心 / Log4j 2 API 和核心

+   适配器 / Log4j 2 适配器

+   配置文件 / 配置文件

+   自动配置 / 自动配置

+   自动配置, URL / 自动配置

+   官方文档 / 官方文档

+   官方文档, URL / 官方文档

+   Redis Appender, 实现 / 有趣的 Redis Appender 实现

+   Redis, URL / 有趣的 Redis Appender 实现

+   Log4j2, 适配器

+   关于 / Log4j 2 适配器

+   Log4j 1.x API 桥接器 / Log4j 1.x API 桥接器

+   Apache Commons Logging 桥接器 / Apache Commons Logging 桥接器

+   SLF4J 桥接器 / SLF4J 桥接器

+   Java Util Logging 适配器 / Java Util Logging 适配器

+   Web Servlet 支持 / Web Servlet 支持

+   登录表单

+   设计 / 登录表单

+   Luna 分发

+   URL / 如何做...

## M

+   @MessageMapping

+   用于定义消息处理程序 / 通过@MessageMapping 定义消息处理程序

+   m2eclipse 插件

+   URL / 更多信息...

+   编组响应

+   关于 / 绑定请求和编组响应, 如何做..., 它是如何工作的...

+   Material Design

+   使用 WebJars / 使用 WebJars 进行 Material Design

+   Materialize

+   URL / 使用 WebJars 进行 Material Design

+   矩阵变量

+   URL 映射 / 使用矩阵变量进行 URL 映射

+   Maven

+   需要 / 为什么要使用 Maven?

+   配置 / 配置 Maven

+   项目结构, 定义 / 使用 Maven 定义项目结构, 准备工作, 如何做...

+   构建生命周期 / Maven 的构建生命周期

+   参考资料 / 更多信息...

+   Maven 3

+   Eclipse, 配置 / 准备工作, 如何做...

+   Maven checkstyle 插件

+   关于 / Maven 的 checkstyle 插件

+   URL / Maven 的 checkstyle 插件

+   Maven Failsafe

+   与 Maven Surefire 相比 / Maven Failsafe 与 Maven Surefire

+   Maven 模型

+   参考链接 / 使用 Maven 属性

+   Maven 插件

+   URL / 还有更多...

+   Maven Surefire

+   关于 / 使用 Mockito 和 Maven Surefire 进行单元测试, 如何做…

+   内存优化

+   参考链接 / 还有更多...

+   面向消息的中间件（MoM） / 使用 RabbitMQ 和 AMQP 堆叠和消费任务

+   消息代理

+   使用全功能消息代理 / 使用全功能消息代理

+   集群能力 / 集群能力 - RabbitMQ

+   STOMP 消息类型 / 更多 STOMP 消息类型

+   StompMessageBrokerRelay / StompMessageBrokerRelay

+   消息驱动的 Bean（MDB）

+   关于 / Spring 事务的相关性

+   消息

+   用于 REST 的国际化 / 为 REST 国际化消息和内容

+   使用 MessageSource beans 进行国际化 / MessageSource beans

+   使用 LocaleResolver / 使用 LocaleResolver

+   发送，分发 / 发送消息以分发

+   SimpMessagingTemplate / SimpMessagingTemplate

+   @SendTo 注释 / The @SendTo 注释

+   使用 Spring Session 进行安全保护 / 使用 Spring Session 和 Redis 保护消息

+   使用 Redis 进行安全保护 / 使用 Spring Session 和 Redis 保护消息

+   Apache HTTP 代理配置 / Apache HTTP 代理配置

+   Redis 服务器安装 / Redis 服务器安装

+   MySQL 服务器安装 / MySQL 服务器安装

+   应用级别的更改 / 应用级别的更改

+   RabbitMQ 配置 / RabbitMQ 配置

+   结果 / 结果

+   Redis 服务器 / Redis 服务器

+   Spring 会话 / Spring 会话

+   SessionRepositoryFilter / SessionRepositoryFilter

+   RedisConnectionFactory / RedisConnectionFactory

+   CookieHttpSessionStrategy / CookieHttpSessionStrategy

+   Tomcat 的 Redis 会话管理器 / Tomcat 的 Redis 会话管理器

+   在 Redis 中查看会话 / 查看/刷新 Redis 中的会话

+   在 Redis 中查看/刷新会话 / 查看/刷新 Redis 中的会话

+   securityContextPersistenceFilter / securityContextPersistenceFilter

+   AbstractSessionWebSocketMessageBrokerConfigurer / AbstractSessionWebSocketMessageBrokerConfigurer

+   AbstractSecurityWebSocketMessageBrokerConfigurer / AbstractSecurityWebSocketMessageBrokerConfigurer

+   Spring 会话，URL / Spring 会话

+   Apache HTTP 代理，额外配置 / Apache HTTP 代理额外配置

+   Spring Data Redis / Spring Data Redis

+   MessageSource beans

+   用于国际化消息 / MessageSource beans

+   ResourceBundleMessageSource / ResourceBundleMessageSource

+   ReloadableResourceBundleMessageSource / ReloadableResourceBundleMessageSource

+   StaticMessageSource / StaticMessageSource

+   定义 / 我们的 MessageSource bean 定义

+   面向消息的中间件（MOM）

+   关于 / 介绍

+   迁移命令 / 迁移

+   Mockito

+   单元测试 / 使用 Mockito 和 Maven Surefire 进行单元测试, 如何做...

+   @Test 注释 / @Test 注释

+   使用 / 使用 Mockito

+   JUnitRunner / MockitoJUnitRunner

+   transferCriticalData 示例 / transferCriticalData 示例

+   registerUser 示例 / registerUser 示例

+   URL / 关于 Mockito

+   JUnit 规则 / JUnit 规则

+   用于创建模拟 / 使用 Mockito 进行模拟

+   MockitoJUnitRunner / MockitoJUnitRunner

+   模拟

+   关于 / Mocks 和 stubs

+   和存根，选择 / 我应该使用模拟还是存根？

+   参考链接 / 我应该使用模拟还是存根？

+   模型

+   在视图中显示，使用 JSTL / 在视图中显示模型，使用 JSTL, 如何做..., 它是如何工作的...

+   在控制器中填充 / 在控制器中填充模型

+   使用 JSP EL 渲染变量 / 使用 JSP EL 渲染变量

+   隐式对象 / 隐式对象

+   模型-视图-控制器（MVC）架构模式

+   关于 / 介绍 Ember.js

+   模型-视图-控制器模式

+   关于 / 模型-视图-控制器模式

+   模型 / 模型-视图-控制器模式

+   视图 / 模型-视图-控制器模式

+   控制器 / 模型-视图-控制器模式

+   模块

+   创建/ 设置 DOM 和创建模块

+   组件，定义/ 定义模块的组件, 工作原理…

+   URL/ 还有更多…

+   mod_alias 模块/ mod_alias 模块

+   mod_proxy 模块/ mod_proxy 模块

+   morris.js 库

+   URL/ 创建响应式内容

+   多用途互联网邮件扩展（MIME）

+   关于/ 上传文件

+   MVC 架构

+   关于/ MVC 架构

+   模型/ MVC 架构

+   视图/ MVC 架构

+   控制器/ MVC 架构

+   最佳实践/ MVC 评论家和最佳实践

+   评论家/ MVC 评论家和最佳实践

+   贫血领域模型/ 贫血领域模型

+   sagan 项目/ 从源代码中学习

+   sagan 项目，URL/ 从源代码中学习

+   MVC 设计模式/ MVC 设计模式

+   MVC 异常

+   URL/ 另请参阅

## N

+   本机 SQL 查询

+   使用/ 使用本机 SQL 查询

+   URL/ 使用本机 SQL 查询

+   导航

+   使用/ 导航

+   重定向选项/ 导航

+   前向选项/ 导航

+   无级联操作

+   关于/ 级联属性

+   Node.js

+   URL/ 设置 Ember CLI

## O

+   OAuth

+   数据，从第三方 API 检索/ 使用 OAuth 从第三方 API 检索数据, 操作方法…

+   OAuth2 认证服务器（AS）

+   关于/ 准备工作

+   OAuth2 授权框架

+   关于/ OAuth2 授权框架

+   实体/ OAuth2 授权框架

+   OAuth 开发

+   别名定义 / OAuth 开发的别名定义

+   面向对象编程（OOP）

+   关于/ 面向方面的编程

+   每个 HTTP 连接策略一个线程 / Spring MVC 中的异步请求处理

+   OpenShift

+   关于/ OpenShift

+   URL/ OpenShift

+   Oracle Hotspot JDK

+   URL/ 操作方法…

## P

+   页面对象

+   使用 FluentLenium/ 使用 FluentLenium 的页面对象

+   使用 Geb/ 使用 Geb 的页面对象

+   分页

+   添加/ 添加分页、过滤和排序功能, 如何做..., 它是如何工作的...

+   Spring 数据分页支持/ Spring 数据分页支持（您会喜欢它！）

+   和存储库中的排序/ 存储库中的分页和排序

+   PagingAndSortingRepository<T,ID>/ PagingAndSortingRepository<T,ID>

+   PageableHandlerMethodArgumentResolver/ web 部分-PageableHandlerMethodArgumentResolver

+   负载映射

+   使用@RequestBody 请求/ 使用@RequestBody 映射请求负载

+   永久代（PermGen）

+   关于/ 自定义 JVM 参数

+   持久化单元

+   关于/ EntityManagerFactory bean 及其持久化单元

+   PhantomJS

+   URL/ 我们的第一个 FluentLenium 测试

+   Pivotal Web Services（PWS）

+   Web 应用程序，部署/ 将您的 Web 应用程序部署到 Pivotal Web Services

+   普通的 Java 对象（POJOs）

+   关于/ 介绍

+   普通的旧 Java 对象（POJO）

+   关于/ 个人资料页面-表单

+   普通的旧 Java 对象（POJOs）

+   关于/ 贫血领域模型

+   普通的旧 Java 对象（POJOs）

+   关于/ 领域对象和实体

+   插件

+   关于/ 插件

+   Maven 编译器插件/ Maven 编译器插件

+   Maven surefire 插件/ Maven surefire 插件

+   Maven enforcer 插件/ Maven enforcer 插件

+   Maven war 插件/ Maven war 插件

+   Maven checkstyle 插件/ Maven checkstyle 插件

+   POD 结构

+   与之一起工作/ 与 POD 结构一起工作

+   切入点设计者（PCDs）

+   关于/ 切入点设计者

+   切入点，@AspectJ 注解

+   关于/ 切入点

+   设计者/ 切入点设计者

+   示例/ 切入点示例

+   POJO（普通的旧 Java 对象）/ Spring 框架背后的设计概念

+   Procfile

+   运行/ Procfile

+   生产配置文件

+   配置/ 生产配置文件

+   个人资料

+   存储，在会话中/ 将个人资料放入会话中

+   个人资料页面

+   关于/ 个人资料页面-表单

+   创建/ 个人资料页面-表单

+   添加验证/ 验证

+   启用客户端验证/ 客户端验证

+   检查点/ 检查点

+   项目对象模型（POM）

+   关于/ 为什么要使用 Maven？

+   项目结构

+   使用 Maven 定义/ 使用 Maven 定义项目结构, 准备就绪, 如何做...

+   创建 Maven 项目/ 新的 Maven 项目，新的 Maven 模块

+   创建 Maven 模块/ 新的 Maven 项目，新的 Maven 模块

+   标准项目层次结构/ 标准项目层次结构

+   在 IDE 中/ IDE 中的项目结构

+   属性

+   注入到 Spring 环境中/ 将属性注入到 Spring 环境中

+   PropertyEditor/ConversionService/ 在 PropertyEditors 或转换器之间进行选择

+   PropertyEditor 实现

+   关于/ 内置的 PropertyEditor 实现

+   PropertyPlaceholderConfigurer

+   关于/ 使用 PropertyPlaceholderConfigurer 外部化属性

+   属性，外部化/ 使用 PropertyPlaceholderConfigurer 外部化属性

+   特定于提供程序的配置，第三方 OAuth2 方案

+   关于/ 特定于提供程序的配置

+   connectionFactoryLocator bean/ 一个入口点-connectionFactoryLocator

+   特定于提供程序的 ConnectionFactories/ 特定于提供程序的 ConnectionFactories

+   使用提供程序帐户登录/ 使用提供程序帐户登录

+   验证的 API 调用，执行/ 执行验证的 API 调用

+   Spring social ConnectController/ Spring social ConnectController

+   SocialAuthenticationFilter/ SocialAuthenticationFilter

+   Spring 社交连接器列表/ Spring 社交连接器列表

+   实现 OAuth2 认证服务器/ 实现 OAuth2 认证服务器

+   和谐发展博客/ 和谐发展博客

+   代理模式

+   URL/ 还有更多...

+   ProxyPassReverse

+   关于/ ProxyPassReverse

+   工作者/ 工作者

## Q

+   查询查找策略

+   定义/ 查询解析方法

+   查询参数

+   关于/ 使用请求参数获取数据

## R

+   @Repository 注释

+   关于/ DAO 支持和@Repository 注释

+   @RequestBody

+   请求负载映射/ 使用@RequestBody 映射请求负载

+   @RequestMapping

+   新的支持类/ 自 Spring MVC 3.1 以来@RequestMapping 的新支持类

+   @RequestMapping 注释/ @RequestMapping

+   @RequestMapping 注释

+   支持/ 广泛支持@RequestMapping 注释

+   setMessageConverters/ setMessageConverters

+   setCustomArgumentResolvers/ setCustomArgumentResolvers

+   setWebBindingInitializer / setWebBindingInitializer

+   作为终极过滤器使用/ @RequestMapping 注释作为终极过滤器

+   @RequestPart

+   用于上传图像/ 使用@RequestPart 上传图像

+   @RunWith 注释

+   关于/ @RunWith 注释

+   RabbitMQ

+   作为多协议消息代理使用/ 将 RabbitMQ 用作多协议消息代理, 如何做...

+   URL / 如何做...

+   指南和文档，URL/ 另请参阅

+   任务，使用 Spring Session 和 Redis 进行保护/ 准备工作

+   任务，使用 Spring Session 和 Redis 进行保护/ 准备工作

+   raphael.js 库

+   URL / 创建响应式内容

+   React

+   关于/ 玩家

+   ReactJS

+   关于/ SPA 框架

+   Redis

+   消息，使用 Spring Session 和 Redis 进行保护/ 使用 Spring Session 和 Redis 保护消息

+   URL / Redis 服务器安装, 分布式会话

+   激活/ 激活 Redis, 激活 Redis

+   关系，EJB3 实体

+   定义/ 定义关系

+   选择/ 实体之间的关系是如何选择的

+   远程

+   URL / setMessageConverters

+   修复命令/ 修复, 关于 Flyway Maven 插件

+   存储库管理器

+   关于/ 存储库管理器

+   URL / 还有更多...

+   REpresentational State Transfer (REST)

+   关于/ REST 的定义

+   Restful CloudStreetMarket/ RESTful CloudStreetMarket

+   REpresentational State Transfer (REST)

+   关于/ 为 JSON 和 XML 媒体构建 RESTful 服务

+   请求通道/ Spring 4 中 STOMP over WebSocket 和回退选项

+   RequestMappingHandlerAdapter

+   URL / 自 Spring MVC 3.1 以来@RequestMapping 的新支持类

+   RequestMappingHandlerAdapter bean

+   关于 / 一个超级 RequestMappingHandlerAdapter bean

+   资源

+   处理 / 处理资源

+   ResourceSupport 类 / ResourceSupport 类

+   响应通道 / Spring 4 中 STOMP over WebSocket 和回退选项

+   ResponseEntityExceptionHandler

+   URL / JavaDocs

+   响应式内容

+   创建 / 创建响应式内容

+   响应式单页 Web 设计

+   使用 Bootstrap 设置 / 使用 Bootstrap 设置和自定义响应式单页 Web 设计, 如何做...

+   自定义 Bootstrap 主题 / 自定义 Bootstrap 主题

+   响应式单页 Web 设计

+   安装 Bootstrap 主题 / 安装 Bootstrap 主题

+   自定义 Bootstrap 主题 / 自定义 Bootstrap 主题

+   REST

+   关于 / 什么是 REST？

+   Rest-assured

+   与之集成测试 / 使用 Cargo，Rest-assured 和 Maven failsafe 进行集成测试, 如何做..., 它是如何工作的...

+   关于 / Rest assured

+   静态导入 / 静态导入

+   用法 / 一种给定，当，然后的方法

+   REST-assured

+   示例 / 更多 REST-assured 示例

+   示例，URL / 更多 REST-assured 示例

+   REST 控制器

+   单元测试 / 单元测试 REST 控制器

+   REST 环境

+   凭据，存储 / 在 REST 环境中存储凭据

+   客户端（AngularJS） / 客户端（AngularJS）

+   服务器端 / 服务器端

+   微服务，用于身份验证 / 用于微服务的身份验证

+   使用 BASIC 身份验证 / 使用 BASIC 身份验证

+   使用 OAuth 进行登录 / 使用 OAuth2

+   HTML5 SessionStorage / HTML5 SessionStorage

+   BCryptPasswordEncoder / BCryptPasswordEncoder

+   HTTP 头，使用 AngularJS 设置 / 使用 AngularJS 设置 HTTP 头

+   浏览器支持，用于 localStorage / 用于 localStorage 的浏览器支持

+   SSL 和 TLS / 关于 SSL 和 TLS

+   RESTful API，调试

+   关于 / 调试 RESTful API

+   JSON 格式化扩展 / JSON 格式化扩展

+   浏览器中的 RESTful 客户端 / 浏览器中的 RESTful 客户端

+   httpie / httpie

+   RESTful web 服务，属性

+   客户端-服务器 / 什么是 REST？

+   无状态的 / 什么是 REST？

+   可缓存的 / 什么是 REST？

+   统一接口 / 什么是 REST？

+   分层的 / 什么是 REST？

+   REST 处理程序

+   扩展，到 HTTP 方法 / 将 REST 处理程序扩展到所有 HTTP 方法, 如何做…, 它是如何工作的...

+   HTTP/1.1 规范 / HTTP/1.1 规范 - RFC 7231 语义和内容

+   负载映射，使用@RequestBody 请求 / 使用@RequestBody 映射请求负载

+   HttpMessageConverters / HttpMessageConverters

+   @RequestPart，用于上传图像 / 使用@RequestPart 上传图像

+   事务管理 / 事务管理

+   Richardson 的成熟模型

+   关于 / Richardson 的成熟模型

+   级别 0 - HTTP / 级别 0 - HTTP

+   级别 1 - 资源 / 级别 1 - 资源

+   级别 2 - HTTP 动词 / 级别 2 - HTTP 动词

+   级别 3 - 超媒体控制 / 级别 3 - 超媒体控制

+   Richardson 成熟模型

+   关于 / Richardson 成熟模型

+   URL / Richardson 成熟模型

+   ROME 项目

+   URL / 提供的 HttpMessageConverters

+   根名称服务器 / DNS 配置或主机别名

+   路由

+   处理 / 处理路由

+   例行

+   需要 / 为什么需要这样的例行？

## S

+   @SendTo 注释 /  @SendTo 注释

+   Saas 提供商

+   URL / Spring 社交连接器列表

+   模式，EJB3 实体

+   映射 / 映射模式

+   表，映射 / 映射表

+   列，映射 / 映射列

+   字段，注释 / 注释字段或 getter

+   getter，注释 / 注释字段或 getter

+   主键，映射 / 映射主键

+   标识符生成 / 标识符生成

+   SearchApiController 类

+   在 search.api 包中创建 / 客户是王

+   securityContextPersistenceFilter / securityContextPersistenceFilter

+   安全头

+   关于 / 授权用户

+   URL / 授权用户

+   自签名证书

+   生成 / 生成自签名证书

+   序列化器

+   URL / Jackson 自定义序列化器

+   服务类 / 基于 XML 的配置元数据

+   服务提供商（SP）

+   关于 / 准备就绪

+   服务

+   授权 / 在服务和控制器上进行授权, 如何做...

+   SessionRepositoryFilter

+   关于 / SessionRepositoryFilter

+   RedisConnectionFactory / RedisConnectionFactory

+   基于 setter 的 DI

+   关于 / 基于构造函数还是基于 setter 的 DI - 哪个更好？

+   SimpleJdbc 类 / SimpleJdbc 类

+   简单文本导向消息协议（STOMP）

+   关于 / STOMP 协议

+   URL / STOMP 协议

+   简单 URL 映射

+   用于配置控制器 / 使用简单 URL 映射配置控制器, 如何做...

+   简单 WebSocket 应用程序

+   创建 / 创建一个简单的 WebSocket 应用程序

+   单页应用程序（SPA）

+   动机 / SPA 背后的动机

+   关于 / 解释 SPA

+   架构优势 / SPA 的架构优势

+   单页应用

+   关于 / 单页应用程序

+   建议 / 参与者

+   未来的增强 / 未来

+   无状态选项 / 无状态化

+   参考链接 / 无状态化

+   SLF4j

+   案例 / SLF4j 的案例

+   SLF4J 桥接器 / SLF4J 桥接器

+   社交事件

+   使用 STOMP 通过 SockJS 进行流式传输 / 使用 STOMP 通过 SockJS 进行流式传输社交事件 , 如何做...

+   Apache HTTP 代理配置 / Apache HTTP 代理配置

+   前端 / 前端

+   前端，URL / 前端

+   后端 / 后端, 它是如何工作的...

+   SockJS

+   关于 / SockJS

+   URL / SockJS

+   回退，选项 / 还有更多...

+   客户端查询，URL / 还有更多...

+   Sockjs

+   关于 / WebSockets

+   SPA 框架

+   关于 / SPA 框架

+   AngularJS / SPA 框架

+   ReactJS / SPA 框架

+   Ember.js / SPA 框架

+   SpEL（Spring 表达式语言）

+   关于 / 查询解析方法

+   SpEL API

+   关于 / SpEL API

+   接口和类 / SpEL API

+   Spock

+   用于单元测试 / 使用 Spock 进行单元测试

+   Spring

+   使用 / 使用 Spring 进行测试

+   安装 / 安装 Spring，Spring MVC 和 Web 结构, 如何做..., 它是如何工作的...

+   Maven 依赖项的继承 / Maven 依赖项的继承

+   包括第三方依赖项 / 包括第三方依赖项

+   Web 资源 / Web 资源

+   Java 持久性 API（JPA），配置 / 在 Spring 中配置 Java 持久性 API, 如何做..., 它是如何工作的...

+   生态系统 / Spring 生态系统

+   URL / Spring 生态系统

+   核心 / 核心

+   执行 / 执行

+   XD 项目，URL / 执行

+   数据 / 数据

+   值得注意的项目 / 其他值得注意的项目

+   Spring 的 JSF 集成

+   关于 / Spring 的 JSF 集成

+   Spring 的 Struts 集成

+   关于 / Spring 的 Struts 集成

+   Spring 管理的 DataSource bean

+   关于 / Spring 管理的 DataSource bean

+   spring-messaging 模块

+   关于 / Spring 4 中的 STOMP over WebSocket 和回退选项

+   spring-security-crypto

+   URL / 社交连接持久性

+   Spring-websocket-portfolio

+   URL / 另请参阅

+   Spring 4.2+

+   URL / 发布应用程序事件的更好方法

+   Spring AOP

+   定义 / Spring AOP - 定义和配置样式

+   配置样式 / Spring AOP - 定义和配置样式

+   Spring 应用程序

+   关于 / 你的第一个 Spring 应用程序

+   控制反转（IoC）/ 控制反转解释

+   Spring Beans

+   在集成测试中注入 / 将 Spring Beans 注入集成测试, 如何做...

+   SpringJUnit4ClassRunner / SpringJUnit4ClassRunner

+   @ContextConfiguration 注释 /  @ContextConfiguration 注释, 还有更多...

+   JdbcTemplate / JdbcTemplate

+   样板逻辑，抽象 / 抽象样板逻辑

+   自动生成的 ID，提取 / 提取自动生成的 ID

+   Spring Boot

+   登录 / 个人资料页面-表单

+   URL / 个人资料页面-表单

+   Spring Data

+   关于 / Spring Data, Spring Data

+   子项目，定义 / Spring Data

+   Commons / Spring Data Commons

+   存储库规范 / Spring Data 存储库规范

+   MongoDB / Spring Data MongoDB

+   领域对象和实体 / 领域对象和实体

+   Spring 事务支持 / Spring 事务支持

+   Spring Data Commons

+   定义 / Spring Data Commons

+   Spring Data JPA

+   配置 / Spring Data JPA 配置

+   使用 / 利用 JPA 和 Spring Data JPA, 如何做..., 它是如何工作的...

+   注入 EntityManager 实例 / 注入 EntityManager 实例

+   Java 持久性查询语言（JPQL），使用 / 使用 JPQL

+   代码，减少 / 使用 Spring Data JPA 减少样板代码

+   查询，创建 / 创建查询

+   实体，持久化 / 持久化实体

+   本地 SQL 查询，使用 / 使用本地 SQL 查询

+   配置事务 / 事务

+   URL / 数据

+   Spring Data 层

+   异常处理 / 在 Spring Data 层处理异常

+   Spring Data Mongo

+   URL / 数据

+   Spring Data MongoDB

+   关于 / Spring Data MongoDB

+   启用 / 启用 Spring Data MongoDB

+   MongoRepository / MongoRepository

+   Spring Data Redis（SDR）框架 / Spring Data Redis 和 Spring Session Data Redis

+   Spring Data 存储库

+   自定义实现 / 另请参阅

+   参考链接 / 另请参阅

+   Spring Data 存储库规范

+   关于 / Spring Data 存储库规范

+   Spring Data JPA / Spring Data JPA

+   Spring Data JPA，启用 / 启用 Spring Data JPA

+   JpaRepository / JpaRepository

+   Spring Data REST

+   URL / 另请参阅, 数据

+   Spring EL

+   URL / Spring EL

+   Spring 表达式语言

+   关于 / Spring 表达式语言

+   特性 / SpEL 特性

+   注解支持 / SpEL 注解支持

+   Spring 表达式语言（SpEL）

+   关于 / Spring 表达式语言

+   URL / Spring 表达式语言

+   使用请求参数获取数据 / 使用请求参数获取数据

+   Spring 表单

+   在 JSP 中组合 / 在 JSP 中组合表单

+   验证 / 验证表单

+   Spring 表单标签库

+   关于 / Spring 和 Spring 表单标签库

+   Springfox

+   URL / Swagger 文档

+   Spring 框架

+   URL / 内置的 PropertyEditor 实现

+   Spring 框架

+   设计概念 / Spring 框架背后的设计概念

+   关于 / Spring 框架带来了什么？

+   Spring 框架模块

+   关于 / Spring 框架模块

+   Spring HATEOAS 资源

+   DTO，转换成 / 将 DTO 转换为 Spring HATEOAS 资源, 如何做…

+   关于 / Spring HATEOAS 资源

+   ResourceSupport 类 / ResourceSupport 类

+   资源类 / 资源类

+   可识别的接口 / 可识别的接口

+   实体的@Id，抽象化 / 抽象化实体的@Id, 还有更多…

+   URL / 另请参阅, 另请参阅

+   Spring 集成

+   URL / 其他值得注意的项目

+   Spring IoC 容器

+   关于 / Spring IoC 容器

+   Spring IO 参考文档

+   URL / Spring IO 参考文档

+   Spring JDBC

+   方法 / Spring JDBC 抽象

+   Spring JDBC 抽象

+   关于 / Spring JDBC 抽象

+   JdbcTemplate / JdbcTemplate

+   SimpleJdbc 类 / SimpleJdbc 类

+   SpringJUnit4ClassRunner / SpringJUnit4ClassRunner

+   Spring 景观

+   关于 / Spring 景观

+   Spring 框架模块 / Spring 框架模块

+   Spring 工具套件（STS） / Spring 工具套件（STS）

+   Spring 子项目 / Spring 子项目

+   Spring MVC

+   Web 应用程序 / Spring MVC 架构

+   Spring MVC

+   特性 / Spring MVC 的特性

+   architecture / Spring MVC 的架构和组件, Spring MVC 架构

+   components / Spring MVC 的架构和组件

+   asynchronous request processing / Spring MVC 中的异步请求处理

+   installing / 安装 Spring，Spring MVC 和 web 结构, 如何做..., 它是如何工作的...

+   about / Spring MVC 概述

+   front controller / 前端控制器

+   MVC 设计模式 / MVC 设计模式

+   flow / Spring MVC 流程

+   DispatcherServlet / DispatcherServlet-Spring MVC 入口点

+   annotation-defined controllers / 注解定义的控制器

+   Spring MVC 1-0-1

+   about / Spring MVC 1-0-1

+   reference link / Spring MVC 1-0-1

+   Spring MVC 3.1 / 自 Spring MVC 3.1 以来的@RequestMapping 新支持类

+   Spring MVC 应用程序

+   creating / 你的第一个 Spring MVC 应用程序

+   setting up / 设置 Spring MVC 应用程序

+   project structure / Spring MVC 应用程序的项目结构

+   web.xml 文件 / 将 web.xml 文件 spring 化的 web 应用程序

+   web app, springifying / 将 web.xml 文件 spring 化的 web 应用程序

+   ApplicationContext 文件 / Spring MVC 应用程序中的 ApplicationContext 文件

+   HomeController / HomeController-主屏幕的@Controller

+   home.jsp 文件 / home.jsp 文件-登陆界面

+   incoming requests, handling / 处理传入请求

+   Spring Reactor

+   URL / 其他值得注意的项目

+   about / 其他值得注意的项目

+   Spring 安全

+   users, adapting / 将用户和角色适应 Spring 安全, 如何做..., 它是如何工作的...

+   roles, adapting / 将用户和角色适应 Spring 安全, 如何做..., 它是如何工作的...

+   about / Spring 安全简介

+   ThreadLocal context holders / ThreadLocal 上下文持有者

+   interfaces / Noticeable Spring Security interfaces

+   Authentication interface / The Authentication interface

+   UserDetails interface / The UserDetails interface

+   UserDetailsManager interface / The UserDetailsManager interface

+   GrantedAuthority interface / The GrantedAuthority interface

+   Spring security, reference

+   about / Spring Security reference

+   technical overview / Technical overview

+   URL / Technical overview, Sample applications

+   sample applications / Sample applications

+   core services / Core services

+   Spring Security 4

+   reference link / Testing the authentication

+   Spring security authorities

+   about / Spring Security authorities

+   configuration attributes / Configuration attributes

+   Security Interceptor protecting secure objects / Configuration attributes

+   Spring security filter-chain

+   URL / SocialAuthenticationFilter

+   Spring security namespace

+   <http> component / The <http> component

+   Spring security filter-chain / The Spring Security filter-chain

+   <http> configuration / Our <http> configuration

+   BasicAuthenticationFilter / BasicAuthenticationFilter

+   with authenticationEntryPoint / With an authenticationEntryPoint

+   URL / In the Spring Security reference

+   Spring security OAuth project

+   URL / Implementing an OAuth2 authentication server

+   Spring security reference

+   URL / In the Spring Security reference, The Spring Security reference

+   Spring session

+   messages, securing with / Securing messages with Spring Session and Redis

+   Spring Social

+   about / Setting up Spring Social Twitter

+   URL / Setting up Spring Social Twitter

+   Spring social reference

+   URL / The Spring social ConnectController

+   Spring Social Twitter project

+   creating / Enough Hello Worlds, let's fetch tweets!

+   application, registering / Registering your application

+   setting up / Setting up Spring Social Twitter

+   Twitter, accessing / Accessing Twitter

+   Spring subprojects

+   about / Spring subprojects

+   URL / Spring subprojects

+   Spring Tool Suite (STS)

+   关于

+   URL

+   Spring 事务

+   定义

+   声明式事务管理

+   使用@Transactional 注解

+   程序化事务管理

+   Spring 验证器

+   使用 Spring 验证器

+   ValodationUtils 实用程序

+   国际化验证错误

+   Spring WebSockets

+   URL，参见

+   Spring WebSocket 支持

+   关于 Spring WebSocket 支持

+   一体化配置

+   消息处理程序，通过@MessageMapping 定义

+   Sql*类

+   使用 Sql*类定义 JDBC 操作

+   SSL

+   参考链接

+   关于 SSL

+   生成自签名证书

+   创建

+   为 http 和 https 通道创建

+   在受保护的服务器后面创建

+   权威性（SOA）

+   状态码

+   500 服务器错误，状态码和异常处理

+   405 方法不受支持，状态码和异常处理

+   404 未找到，状态码和异常处理

+   400 错误请求

+   200 OK，状态码和异常处理

+   使用 ResponseEntity

+   带异常的状态码

+   状态码

+   关于状态码和异常处理

+   StompMessageBrokerRelay

+   STOMP over SockJS

+   社交事件，使用 STOMP 通过 SockJS 进行流式传输

+   STOMP over WebSocket

+   关于/ Spring 4 中的 STOMP over WebSocket 和回退选项

+   流，Java 8

+   关于/ Java 8 流和 lambda

+   存根

+   关于/ 模拟和存根

+   创建，用于测试 bean/ 在测试时存根化我们的 bean

+   和模拟，选择/ 我应该使用模拟还是存根？

+   支持的 bean，DispatcherServlet

+   HandlerMapping/ 支持 DispatcherServlet 的 Bean 及其角色

+   HandlerAdapter/ 支持 DispatcherServlet 的 Bean 及其角色

+   HandlerExceptionResolver/ 支持 DispatcherServlet 的 Bean 及其角色

+   ViewResolver/ 支持 DispatcherServlet 的 Bean 及其角色

+   LocaleResolver/ 支持 DispatcherServlet 的 Bean 及其角色

+   LocaleContextResolver/ 支持 DispatcherServlet 的 Bean 及其角色

+   ThemeResolver/ 支持 DispatcherServlet 的 Bean 及其角色

+   MultipartResolver/ 支持 DispatcherServlet 的 Bean 及其角色

+   FlashMapManager/ 支持 DispatcherServlet 的 Bean 及其角色

+   Swagger

+   API，文档/ 准备就绪, 如何做..., 它是如何工作的...

+   API，暴露/ 如何做..., 它是如何工作的...

+   不同的工具/ 不同的工具，不同的标准

+   关于/ Swagger 文档

+   Swagger.io

+   URL/ Swagger.io

+   Swagger UI

+   关于/ Swagger UI

## T

+   @Test 注释

+   关于/ @Test 注释

+   预期和超时参数/ 预期和超时参数

+   @Transactional 注释

+   使用/ 使用@Transactional 注释

+   事务管理，启用/ 启用@Transactional 的事务管理

+   Taskify 应用程序

+   构建/ 构建 Taskify 应用程序

+   Taskify Ember 应用

+   构建/ 构建 Taskify Ember 应用

+   Taskify，设置为 Ember CLI 项目/ 将 Taskify 设置为 Ember CLI 项目

+   Ember Data，设置/ 设置 Ember Data

+   应用程序路由，配置/ 配置应用程序路由

+   主屏幕，构建/ 构建主屏幕

+   构建用户屏幕 / 构建用户屏幕

+   自定义助手，构建 / 构建自定义助手

+   操作处理程序，添加 / 添加操作处理程序

+   自定义组件，构建 / 构建自定义组件-模态窗口

+   使用{{modal-window}}构建 userEditModal / 使用{{modal-window}}构建 userEditModal

+   构建任务屏幕 / 构建任务屏幕

+   任务

+   使用 RabbitMQ 堆叠 / 使用 RabbitMQ 和 AMQP 堆叠和消费任务, 如何做…

+   使用 RabbitMQ 消费 / 使用 RabbitMQ 和 AMQP 堆叠和消费任务, 如何做…

+   发送方 / 发送方

+   消费者端 / 消费者端

+   客户端 / 客户端

+   消息架构概述 / 消息架构概述

+   可扩展模型 / 可扩展模型

+   模板方法

+   关于 / JdbcTemplate

+   术语，面向方面的编程（AOP）

+   方面 / AOP 概念和术语

+   连接点 / AOP 概念和术语

+   建议 / AOP 概念和术语

+   切入点 / AOP 概念和术语

+   目标对象 / AOP 概念和术语

+   编织 / AOP 概念和术语

+   介绍 / AOP 概念和术语

+   测试驱动开发（TDD） / 使用 Spring 进行测试

+   测试驱动开发（TTD）

+   关于 / 测试驱动开发

+   测试框架

+   关于 / 介绍 Ember.js

+   测试支持，Spring

+   模拟对象 / 模拟对象

+   单元和集成测试工具 / 单元和集成测试工具

+   th*each 标签

+   关于 / 访问 Twitter

+   Spring 第三方依赖

+   Spring 框架依赖模型 / Spring 框架依赖模型

+   Spring MVC 依赖 / Spring MVC 依赖

+   使用 Maven 属性 / 使用 Maven 属性

+   第三方 OAuth2 方案

+   使用第三方 OAuth2 方案进行身份验证 / 使用第三方 OAuth2 方案进行身份验证, 如何做…, 它是如何工作的…

+   应用程序角度 / 从应用程序角度

+   Yahoo!观点 / 从 Yahoo!的观点

+   OAuth2 显式授权流程 / OAuth2 显式授权流程

+   刷新令牌和访问令牌 / 刷新令牌和访问令牌

+   Spring 社交 / Spring 社交-角色和关键功能

+   社交连接持久性 / 社交连接持久性

+   特定于提供程序的配置 / 特定于提供程序的配置

+   Thymeleaf

+   关于 / 解析 Thymeleaf 视图, 使用 Thymeleaf

+   视图，解析 / 解析 Thymeleaf 视图

+   使用 / 使用 Thymeleaf

+   参考链接 / 使用 Thymeleaf, Thymeleaf 安全标签

+   页面，添加 / 我们的第一个页面

+   thymeleaf 安全标签

+   使用 / Thymeleaf 安全标签

+   Tomcat（7+）

+   参考链接 / 全局与本地事务

+   Tomcat 8

+   Eclipse，配置 / 准备就绪, 如何做...

+   URL / 如何做...

+   Eclipse，集成 / Eclipse 中的 Tomcat 8

+   Tomcat 连接器

+   关于 / Tomcat 连接器

+   HTTP 连接器 / HTTP 连接器

+   AJP 连接器 / AJP 连接器

+   URL / 还有更多...

+   工具

+   关于 / 合适的工具

+   JUnit / 合适的工具

+   AssertJ / 合适的工具

+   Mockito / 合适的工具

+   DbUnit / 合适的工具

+   Spock / 合适的工具

+   交易

+   关于 / Spring 事务支持

+   事务属性

+   定义 / Spring 事务基础

+   事务管理

+   关于 / 事务管理

+   构建 / 简化的方法

+   ACID 属性 / ACID 属性

+   全局事务，本地事务 / 全局与本地事务

+   推特

+   URL / 注册您的应用程序, Twitter 身份验证

+   Twitter 身份验证

+   设置 / Twitter 身份验证

+   社交身份验证，设置 / 设置社交身份验证

+   编码 / 解释

+   Typescript

+   关于 / 未来

+   URL / 未来

## U

+   UI 行为

+   处理，使用的组件 / 使用组件处理 UI 行为

+   ToggleButton 组件，逐步构建 / 逐步构建 ToggleButton 组件

+   使用 Handlebars 构建 UI 模板

+   关于 / 使用 Handlebars 构建 UI 模板

+   Handlebars 助手 / Handlebars 助手

+   数据绑定，带输入助手 / 带输入助手的数据绑定

+   控制流助手，在 Handlebars 中使用 / 在 Handlebars 中使用控制流助手

+   事件助手，使用 / 使用事件助手

+   统一表达式语言（UEL） / Spring 表达式语言

+   单元测试

+   关于 / 我应该如何测试我的代码？, 单元测试

+   工具 / 工作的正确工具

+   编写 / 我们的第一个单元测试

+   REST 控制器 / 对 REST 控制器进行单元测试

+   使用 Spock / 使用 Spock 进行单元测试

+   URI 模板模式

+   关于 / URI 模板模式

+   Ant 样式路径模式 / Ant 样式路径模式

+   路径模式比较 / 路径模式比较

+   ViewResolvers / ViewResolvers

+   URL 映射

+   带矩阵变量 / 带矩阵变量的 URL 映射

+   UserDetails 接口

+   关于 / UserDetails 接口

+   认证提供者 / 认证提供者

+   UserDetailsManager 接口

+   关于 / UserDetailsManager 接口

+   用户体验范式

+   关于 / 用户体验范式

+   用户管理 API

+   关于 / 用户管理 API

+   用户

+   预调用处理 / 预调用处理

+   AccessDecisionManager 接口 / AccessDecisionManager

+   调用处理 / 调用处理后

+   基于表达式的访问控制 / 基于表达式的访问控制

+   Web 安全表达式 / Web 安全表达式

+   方法安全表达式 / 方法安全表达式

+   @PreAuthorize，用于访问控制 / 使用@PreAuthorize 和@PostAuthorize 进行访问控制

+   @PostAuthorize，用于访问控制 / 使用@PreAuthorize 和@PostAuthorize 进行访问控制

+   集合过滤，使用@PreFilter / 使用@PreFilter 和@PostFilter 过滤集合

+   使用@PostFilter 进行集合过滤 / 使用@PreFilter 和@PostFilter 进行集合过滤

+   JSR-250 / JSR-250 和传统方法安全

## V

+   验证命令 / 验证

+   验证，个人资料页面

+   添加 / 验证

+   参考链接 / 验证

+   自定义验证消息 / 自定义验证消息

+   定义自定义注释 / 自定义验证的自定义注释

+   ValidationUnits 实用程序

+   URL / ValidationUtils

+   验证器

+   参考链接 / 客户端验证

+   ViewResolver

+   用于配置回退控制器 / 使用 ViewResolver 配置回退控制器, 如何做..., 它是如何工作的...

+   视图解析器

+   AbstractCachingViewResolver / 解析视图

+   XmlViewResolver / 解析视图

+   ResourceBundleViewResolver / 解析视图

+   基于 URL 的视图解析器/ 解析视图

+   InternalResourceViewResolver / 解析视图

+   VelocityViewResolver / 解析视图

+   FreeMarkerViewResolver / 解析视图

+   JasperReportsViewResolver / 解析视图

+   TilesViewResolver / 解析视图

+   视图

+   使用 / 使用视图

+   解析 / 解析视图

+   JSP 视图，解析 / 解析 JSP 视图

+   在 JSP 页面中绑定模型属性 / 使用 JSTL 在 JSP 页面中绑定模型属性

+   视图技术，Spring MVC / 更多视图技术

## W

+   web.xml 文件

+   参考链接 / 还有更多...

+   Web 应用程序

+   Dispatcher Servlet / DispatcherServlet

+   显示数据 / 将数据传递给视图

+   部署到 Pivotal Web Services（PWS） / 将您的 Web 应用程序部署到 Pivotal Web Services

+   安装 Cloud Foundry CLI 工具 / 安装 Cloud Foundry CLI 工具

+   组装 / 组装应用程序

+   激活 Redis / 激活 Redis

+   在 Heroku 上部署 / 将您的 Web 应用程序部署到 Heroku

+   在 Heroku 上设置 / 设置应用程序

+   在 Heroku 上执行 / 运行您的应用程序

+   改进 / 改进您的应用程序

+   WebApplicationObjectSupport

+   URL/ WebContentGenerator 提供的更多功能

+   Web 存档（war）

+   关于/ 准备就绪

+   Web 缓存

+   URL/ Web 缓存

+   WebContentGenerator

+   关于/ WebContentGenerator 提供的更多功能

+   WebContentInterceptor

+   定义/ 定义通用的 WebContentInterceptor, 如何做..., 它是如何工作的...

+   控制器/ 控制器的常见行为

+   会话，需要/ 需要会话

+   会话，同步/ 同步会话

+   缓存头管理/ 缓存头管理

+   HTTP 方法支持/ HTTP 方法支持

+   高级拦截器/ 高级拦截器

+   请求生命周期/ 请求生命周期

+   WebJars

+   用于材料设计/ 使用 WebJars 进行材料设计

+   布局，使用/ 使用布局

+   使用导航/ 导航

+   TweetController，使用/ 检查点

+   Web 资源

+   关于/ Web 资源

+   目标运行环境/ 目标运行环境

+   Spring Web 应用程序上下文/ Spring Web 应用程序上下文

+   插件/ 插件

+   Web 服务

+   URL/ setMessageConverters

+   Web Servlet 支持/ Web Servlet 支持

+   WebSocket

+   关于/ WebSockets

+   使用/ WebSockets

+   参考链接/ WebSockets

+   WebSocket 应用程序

+   消息，广播给单个用户/ 在 WebSocket 应用程序中向单个用户广播消息

+   WebSockets

+   关于/ WebSockets 简介

+   URL/ WebSockets 简介

+   生命周期/ WebSocket 生命周期

+   URI 方案/ 两个专用 URI 方案

+   Web 结构

+   关于/ 安装 Spring，Spring MVC 和 Web 结构

+   创建/ 准备就绪, 如何做..., 它是如何工作的...

+   Web 工具平台（WTP）插件

+   关于/ Eclipse 中的 Tomcat 8

+   工作者/ 工作者

## X

+   XML

+   生成/ 生成 XML

+   XML 编组，支持

+   关于/ 支持 XML 编组

+   XStream 编组器/ XStream 编组器

+   XML 解析器

+   JAXB2 实现，使用为 / 使用 JAXB2 实现作为 XML 解析器

+   基于 XML 模式的 AOP

+   关于 / 基于 XML 模式的 AOP

+   XStream

+   URL / XStream 编组器

+   X Stream 转换器

+   URL / XStream 转换器

## Y

+   Yahoo! API

+   URL / 另请参阅

+   Yahoo!财务股票代码 / 另请参阅

## Z

+   zipcloud-core

+   关于 / 准备工作

+   zipcloud-parent

+   关于 / 准备工作
