# 前言

Spring 框架一直对不同的数据访问技术有很好的支持。然而，有一件事长期保持不变：开发人员必须使用特定于技术的 API 来实现其数据访问层，而且这些 API 通常不是很简洁。这导致了这样一种情况：为了实现期望的结果，人们不得不编写大量样板代码。听起来很熟悉，对吧？

Spring Data 项目诞生是为了解决这些问题。它的目标是为使用 Spring 框架的应用程序提供更简单的创建方式，无论是使用关系数据库还是较新的数据访问技术，如非关系数据库、映射-减少框架或基于云的存储技术。它本质上是一个父项目，将数据存储特定的子项目收集到一个品牌下。Spring Data 项目的所有子项目的完整列表可以从 Spring Data 项目的主页上找到：[`www.springsource.org/spring-data/`](http://www.springsource.org/spring-data/)。

本书集中在两个特定的子项目上：Spring Data JPA 和 Spring Data Redis。您将学习一种更简单的方法来管理实体并使用 Spring Data JPA 创建数据库查询。本书还演示了如何向存储库添加自定义功能。您还将学习如何使用 Redis 键值存储作为数据存储，并利用其其他功能来增强应用程序的性能。

这本实用指南证明了实现 JPA 存储库可以很有趣，并帮助您在应用程序中利用 Redis 的性能。

# 本书涵盖的内容

第一章《入门》简要介绍了本书中描述的技术。本章分为两部分：第一部分描述了 Java 持久性 API 背后的动机，概述了其主要概念，并展示了如何使用它构建数据库查询。第二部分确定了 Redis 键值存储的关键特性。

第二章《使用 Spring Data JPA 入门》帮助您开始使用 Spring Data JPA 构建应用程序。您将学习如何设置一个使用 Spring Data JPA 的项目，并通过编程配置来配置您的应用程序。您还将学习一种简单的方法来为您的实体创建存储库，并使用 Spring Data JPA 实现一个简单的联系人管理应用程序。

第三章《使用 Spring Data JPA 构建查询》描述了您可以使用的技术来构建数据库查询。阅读本章后，您将了解如何使用查询方法、JPA Criteria API 和 Querydsl 来构建数据库查询。您还将通过向其添加搜索功能来继续实现联系人管理应用程序。

第四章《向 JPA 存储库添加自定义功能》教会您如何自定义存储库。您将学习如何将自定义功能添加到单个存储库或所有存储库。本章讨论的原则是通过自定义联系人管理应用程序的存储库来演示的。

第五章《使用 Spring Data Redis 入门》将指导您完成安装和配置阶段，这是在您的应用程序中使用 Spring Data Redis 之前所必需的。它描述了如何在运行类 Unix 操作系统的计算机上安装 Redis。然后您可以设置一个使用 Spring Data Redis 的项目。在本章的最后部分，您将学习如何配置 Redis 连接并比较支持的连接器库的特性。

第六章，*使用 Spring Data Redis 构建应用程序*，教您如何在 Spring 应用程序中使用 Redis。它描述了 Spring Data Redis 的关键组件，并教您如何使用它们。当您将 Redis 用作联系人管理应用程序的数据存储时，您还将看到 Spring Data Redis 的实际应用。本章的最后部分描述了如何将 Spring Data Redis 用作 Spring 3.1 缓存抽象的实现。您还将在本章中看到如何利用 Redis 的发布/订阅消息模式实现。

# 您需要为这本书做些什么

为了运行本书的代码示例，您需要安装以下软件：

+   Java 1.6

+   Maven 3.0.X

+   Redis 2.6.0-rc6

+   一个网络浏览器

如果您想尝试代码示例，您还需要：

+   诸如 Eclipse、Netbeans 或 IntelliJ Idea 之类的 IDE

+   每章的完整源代码包（请参阅下面的*下载示例代码*部分）

# 这本书适合谁

这本书非常适合正在使用 Spring 应用程序的开发人员，并且正在寻找一种更容易的方式来编写使用关系数据库的数据访问代码。此外，如果您有兴趣了解如何在应用程序中使用 Redis，那么这本书适合您。这本书假定您已经从 Spring 框架和 Java 持久性 API 中获得了一些经验。不需要来自 Redis 的先前经验。

# 约定

在本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些示例以及它们的含义解释。

文本中的代码单词显示如下：“我们可以通过使用`@EnableJpaRepositories`注释的`repositoryFactoryBeanClass`属性来实现这一点。”

代码块设置如下：

```java
@Override
protected RepositoryFactorySupport createRepositoryFactory(EntityManager entityManager) {
    return new BaseRepositoryFactory(entityManager);
}
```

当我们希望引起您对代码块的特定部分的注意时，相关的行或项目将以粗体显示：

```java
@CachePut(value = "contacts", key="#p0.id")
@Transactional(rollbackFor = NotFoundException.class)
@Override
public Contact update(ContactDTO updated) throws NotFoundException {
    //Implementation remains unchanged.
}
```

**新术语**和**重要单词**以粗体显示。

### 注意

警告或重要说明会出现在这样的框中。

### 提示

提示和技巧会出现在这样。 
