# 前言

Spring 5.0 即将推出，将带来许多新的令人兴奋的功能，将改变我们迄今为止使用该框架的方式。本书将向您展示这一演变——从解决可测试应用程序的问题到在云端构建分布式应用程序。

本书以介绍 Spring 5.0 的新功能开始，并向您展示如何使用 Spring MVC 构建应用程序。然后，您将深入了解如何使用 Spring Framework 构建和扩展微服务。您还将了解如何构建和部署云应用程序。您将意识到应用程序架构是如何从单体架构演变为围绕微服务构建的。还将涵盖 Spring Boot 的高级功能，并通过强大的示例展示。

通过本书，您将掌握使用 Spring Framework 开发应用程序的知识和最佳实践。

# 本书涵盖内容

第一章《Evolution to Spring Framework 5.0》带您了解 Spring Framework 的演变，从最初的版本到 Spring 5.0。最初，Spring 被用来使用依赖注入和核心模块开发可测试的应用程序。最近的 Spring 项目，如 Spring Boot、Spring Cloud、Spring Cloud Data Flow，涉及应用程序基础设施和将应用程序迁移到云端。我们将概述不同的 Spring 模块和项目。

第二章《Dependency Injection》深入探讨了依赖注入。我们将看看 Spring 中可用的不同类型的依赖注入方法，以及自动装配如何简化您的生活。我们还将快速了解单元测试。

第三章《使用 Spring MVC 构建 Web 应用程序》快速概述了使用 Spring MVC 构建 Web 应用程序。

第四章《演变为微服务和云原生应用程序》解释了过去十年应用程序架构的演变。我们将了解为什么需要微服务和云原生应用程序，并快速概述帮助我们构建云原生应用程序的不同 Spring 项目。

第五章《使用 Spring Boot 构建微服务》讨论了 Spring Boot 如何简化创建生产级 Spring 应用程序的复杂性。它使得使用基于 Spring 的项目变得更加容易，并提供了与第三方库的轻松集成。在本章中，我们将带领学生一起使用 Spring Boot。我们将从实现基本的 Web 服务开始，然后逐步添加缓存、异常处理、HATEOAS 和国际化，同时利用 Spring Framework 的不同功能。

第六章《扩展微服务》专注于为我们在第四章中构建的微服务添加更多高级功能。

第七章《Spring Boot 高级功能》介绍了 Spring Boot 的高级功能。您将学习如何使用 Spring Boot Actuator 监视微服务。然后，您将把微服务部署到云端。您还将学习如何使用 Spring Boot 提供的开发者工具更有效地开发。

第八章《Spring Data》讨论了 Spring Data 模块。我们将开发简单的应用程序，将 Spring 与 JPA 和大数据技术集成在一起。

第九章《Spring Cloud》讨论了云中的分布式系统存在的常见问题，包括配置管理、服务发现、断路器和智能路由。在本章中，您将了解 Spring Cloud 如何帮助您为这些常见模式开发解决方案。这些解决方案应该在云端和开发人员的本地系统上都能很好地运行。

第十章《Spring Cloud 数据流》讨论了 Spring Cloud 数据流，它提供了一系列关于基于微服务的分布式流式处理和批处理数据管道的模式和最佳实践。在本章中，我们将了解 Spring Cloud 数据流的基础知识，并使用它构建基本的数据流使用案例。

第十一章《响应式编程》探讨了使用异步数据流进行编程。在本章中，我们将了解响应式编程，并快速了解 Spring Framework 提供的功能。

第十二章《Spring 最佳实践》帮助您了解与单元测试、集成测试、维护 Spring 配置等相关的 Spring 企业应用程序开发的最佳实践。

第十三章《在 Spring 中使用 Kotlin》向您介绍了一种快速流行的 JVM 语言——Kotlin。我们将讨论如何在 Eclipse 中设置 Kotlin 项目。我们将使用 Kotlin 创建一个新的 Spring Boot 项目，并实现一些基本的服务，并进行单元测试和集成测试。

# 本书所需内容

为了能够运行本书中的示例，您需要以下工具：

+   Java 8

+   Eclipse IDE

+   Postman

我们将使用嵌入到 Eclipse IDE 中的 Maven 来下载所有需要的依赖项。

# 本书适合对象

本书适用于有经验的 Java 开发人员，他们了解 Spring 的基础知识，并希望学习如何使用 Spring Boot 构建应用程序并将其部署到云端。

# 惯例

在本书中，您会发现一些区分不同类型信息的文本样式。以下是一些这些样式的示例及其含义解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL 和用户输入显示如下："在您的`pom.xml`文件中配置`spring-boot-starter-parent`"。

代码块设置如下：

```java
<properties>
  <mockito.version>1.10.20</mockito.version>
</properties>
```

任何命令行输入或输出都以以下方式编写：

```java
mvn clean install
```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的词语，例如菜单或对话框中的词语，会在文本中以这种方式出现："提供详细信息并单击生成项目"。

警告或重要说明会出现在这样的框中。

提示和技巧会以这种方式出现。
