# 工具、框架和环境

“我们成为我们所看到的。我们塑造我们的工具，然后我们的工具塑造我们。”

- 马歇尔·麦克卢汉

正如每个士兵都了解他的武器一样，程序员必须熟悉开发生态系统和使编程更加容易的工具。无论您是否已经在工作或家中使用这些工具中的任何一个，都值得看看它们的特点、优势和劣势。让我们概述一下我们现在可以找到的关于以下主题的内容，并构建一个小项目来熟悉其中一些。

我们不会详细介绍这些工具和框架，因为这将在接下来的章节中进行。我们的目标是让您快速上手，并为您提供它们的简要概述以及它们的功能和使用方法。

本章将涵盖以下主题：

+   Git

+   虚拟机

+   构建工具

+   集成开发环境

+   单元测试框架

+   Hamcrest 和 AssertJ

+   代码覆盖工具

+   模拟框架

+   用户界面测试

+   行为驱动开发

# Git

Git 是最流行的版本控制系统。因此，本书中使用的所有代码都存储在 Bitbucket（[`bitbucket.org/`](https://bitbucket.org/)）中。如果您还没有安装 Git，请安装 Git。所有流行操作系统的发行版都可以在以下网址找到：[`git-scm.com`](http://git-scm.com)。

Git 有许多图形界面可用；其中一些是 Tortoise（[`code.google.com/p/tortoisegit`](https://code.google.com/p/tortoisegit)）、Source Tree（[`www.sourcetreeapp.com`](https://www.sourcetreeapp.com)）和 Tower（[`www.git-tower.com/`](http://www.git-tower.com/)）。

# 虚拟机

虽然它们不是本书的主题，但虚拟机是一个强大的工具，在良好的开发环境中是一等公民。它们在隔离系统中提供动态和易于使用的资源，因此可以在需要时使用和丢弃。这有助于开发人员专注于他们的任务，而不是浪费时间从头开始创建或安装所需的服务。这就是为什么虚拟机在这里找到了位置的原因。我们希望利用它们让您专注于代码。

为了在使用不同操作系统时拥有相同的环境，我们将使用 Vagrant 创建虚拟机，并使用 Docker 部署所需的应用程序。我们选择 Ubuntu 作为我们示例中的基本操作系统，只是因为它是一种流行的常用的类 Unix 发行版。大多数这些技术都是跨平台的，但偶尔您可能无法按照这里找到的说明进行操作，因为您可能使用其他操作系统。在这种情况下，您的任务是找出 Ubuntu 和您的操作系统之间的差异，并相应地采取行动。

# Vagrant

Vagrant 是我们将用于创建开发环境堆栈的工具。这是一种简单的方法，可以使用预配置的虚拟机初始化准备就绪的虚拟机，而只需付出最少的努力。所有的虚拟机和配置都放在一个文件中，称为`Vagrant`文件。

以下是创建一个简单 Ubuntu 虚拟机的示例。我们额外配置了使用 Docker 安装 MongoDB（Docker 的使用将很快解释）。我们假设您的计算机上已安装了 VirtualBox（[`www.virtualbox.org`](https://www.virtualbox.org)）和 Vagrant（[`www.vagrantup.com`](https://www.vagrantup.com)），并且您有互联网访问。

在这种特殊情况下，我们正在创建一个 Ubuntu 64 位实例，使用 Ubuntu box（`ubuntu/trusty64`）并指定 VM 应该有 1GB 的 RAM：

```java
  config.vm.box = "ubuntu/trusty64" 

  config.vm.provider "virtualbox" do |vb| 
  vb.memory = "1024" 
  end 
```

接下来，我们将在 Vagrant 机器中公开 MongoDB 的默认端口，并使用 Docker 运行它：

```java
  config.vm.network "forwarded_port", guest: 27017, host: 27017 
  config.vm.provision "docker" do |d| 
    d.run "mongoDB", image: "mongo:2", args: "-p 27017:27017" 
  end 
```

最后，为了加快 Vagrant 的设置速度，我们正在缓存一些资源。您应该安装名为`cachier`的插件。有关更多信息，请访问：[`github.com/fgrehm/vagrant-cachier`](https://github.com/fgrehm/vagrant-cachier)。

```java
  if Vagrant.has_plugin?("vagrant-cachier") 
    config.cache.scope = :box 
  end 
```

现在是时候看它运行了。第一次运行通常需要几分钟，因为需要下载和安装基本框和所有依赖项：

```java
$> vagrant plugin install vagrant-cachier
$> git clone https://bitbucket.org/vfarcic/tdd-java-ch02-example-vagrant.git
$> cd tdd-java-ch02-example-vagrant$> vagrant up
```

运行此命令时，您应该看到以下输出：

![](img/beed715c-3901-41ea-8d2a-97d563266fc0.png)

请耐心等待执行完成。完成后，您将拥有一个新的 Ubuntu 虚拟机，其中已经安装了 Docker 和一个 MongoDB 实例。最棒的部分是所有这些都是通过一个命令完成的。

要查看当前运行的 VM 的状态，可以使用`status`参数：

```java
$> vagrant status
Current machine states:
default                   running (virtualbox)

```

可以通过`ssh`或使用 Vagrant 命令访问虚拟机，如以下示例：

```java
$> vagrant ssh
Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 3.13.0-46-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

 System information disabled due to load higher than 1.0

 Get cloud support with Ubuntu Advantage Cloud Guest:
 http://www.ubuntu.com/business/services/cloud

 0 packages can be updated.
 0 updates are security updates.

vagrant@vagrant-ubuntu-trusty-64:~$  
```

最后，要停止虚拟机，请退出虚拟机并运行`vagrant halt`命令：

```java
$> exit
$> vagrant halt
 ==> default: Attempting graceful shutdown of VM...
$>  
```

访问以下网址获取 Vagrant 框的列表或有关配置 Vagrant 的更多详细信息：[`www.vagrantup.com`](https://www.vagrantup.com)。

# Docker

设置环境后，是时候安装我们需要的服务和软件了。这可以通过 Docker 来完成，Docker 是一种简单且便携的方式，可以在隔离的容器中运行许多应用程序和服务。我们将使用它来安装所需的数据库、Web 服务器以及本书中需要的所有其他应用程序，这些都将在使用 Vagrant 创建的虚拟机中进行。事实上，之前创建的 Vagrant 虚拟机已经有一个使用 Docker 运行 MongoDB 实例的示例。

让我们再次启动 VM（我们之前使用`vagrant halt`命令停止了它），还有 MongoDB：

```java
$> vagrant up
$> vagrant ssh
vagrant@vagrant-ubuntu-trusty-64:~$ docker start mongoDB
mongoDB
vagrant@vagrant-ubuntu-trusty-64:~$ docker ps
CONTAINER ID        IMAGE           COMMAND                    CREATED
360f5340d5fc        mongo:2         "/entrypoint.sh mong..."   4 minutes ago

STATUS              PORTS                      NAMES
Up 4 minutes        0.0.0.0:27017->27017/tcp   mongoDB
vagrant@vagrant-ubuntu-trusty-64:~$ exit

```

使用`docker start`启动了容器；使用`docker ps`列出了所有正在运行的进程。

通过使用这种程序，我们能够在眨眼之间复制一个全栈环境。您可能想知道这是否像听起来的那样令人敬畏。答案是肯定的，它确实如此。Vagrant 和 Docker 允许开发人员专注于他们应该做的事情，而不必担心复杂的安装和棘手的配置。此外，我们额外努力为您提供了在本书中复制和测试所有代码示例和演示所需的所有步骤和资源。

# 构建工具

随着时间的推移，代码往往会在复杂性和规模上增长。这是软件行业的本质。所有产品都在不断发展，并且在产品的整个生命周期中都会实施新的要求。构建工具提供了一种尽可能简化项目生命周期管理的方法，通过遵循一些代码约定，例如以特定方式组织代码，并使用命名约定为您的类或由不同文件夹和文件组成的确定项目结构。

您可能熟悉 Maven 或 Ant。它们是处理项目的绝佳工具，但我们在这里是为了学习，所以决定使用 Gradle。Gradle 的一些优点是减少了样板代码，使文件更短、配置文件更易读。此外，Google 将其用作构建工具。它得到了 IntelliJ IDEA 的支持，非常容易学习和使用。通过添加插件，大多数功能和任务都可以实现。

精通 Gradle 不是本书的目标。因此，如果您想了解更多关于这个令人敬畏的工具，请访问其网站（[`gradle.org/`](http://gradle.org/)）并阅读您可以使用的插件和可以自定义的选项。要比较不同的 Java 构建工具，请访问：[`technologyconversations.com/2014/06/18/build-tools/`](https://technologyconversations.com/2014/06/18/build-tools/)。

在继续之前，请确保 Gradle 已安装在您的系统上。

让我们分析 `build.gradle` 文件的相关部分。它以 Groovy 作为描述语言，以简洁的方式保存项目信息。这是我们的项目构建文件，由 IntelliJ 自动生成：

```java
apply plugin: 'java'
sourceCompatibility = 1.7
version = '1.0'
```

由于这是一个 Java 项目，所以应用了 Java 插件。它带来了常见的 Java 任务，如构建、打包、测试等。源兼容性设置为 JDK 7。如果我们尝试使用此版本不支持的 Java 语法，编译器将会报错：

```java
repositories { 
    mavenCentral() 
} 
```

Maven Central（[`search.maven.org/`](http://search.maven.org/)）保存了我们的所有项目依赖项。本节告诉 Gradle 从哪里获取它们。Maven Central 仓库对于这个项目已经足够了，但如果有的话，您可以添加自定义仓库。Nexus 和 Ivy 也受支持：

```java
dependencies { 
    testCompile group: 'junit', name: 'junit', version: '4.12' 
} 
```

最后，这是项目依赖项的声明方式。IntelliJ 决定使用 JUnit 作为测试框架。

Gradle 任务很容易运行。例如，要从命令提示符中运行测试，我们可以简单地执行以下操作：

```java
gradle test  
```

可以通过从 IDEA 中运行 Gradle 工具窗口中的 `test` 任务来完成。

测试结果存储在位于 `build/reports/tests` 目录中的 HTML 文件中。

以下是通过运行 `gradle test` 生成的测试报告：

![](img/3faba178-b0f0-487c-bf82-a7656f5e7f51.png)

# 集成开发环境

由于将涵盖许多工具和技术，我们建议使用 IntelliJ IDEA 作为代码开发工具。主要原因是这个**集成开发环境**（**IDE**）可以在没有繁琐配置的情况下工作。社区版（IntelliJ IDEA CE）带有许多内置功能和插件，使编码变得简单高效。它会根据文件扩展名自动推荐可以安装的插件。由于我们选择了 IntelliJ IDEA 作为本书的工具，因此您将在引用和步骤中找到与其操作或菜单相关的内容。如果读者使用其他 IDE，应该找到模拟这些步骤的正确方法。请参阅：[`www.jetbrains.com/idea/`](https://www.jetbrains.com/idea/) 了解如何下载和安装 IntelliJ IDEA 的说明。

# IDEA 演示项目

让我们创建演示项目的基本布局。本章将使用该项目来说明所有涉及的主题。Java 将是编程语言，Gradle（[`gradle.org/`](http://gradle.org/)）将用于运行不同的任务集，如构建、测试等。

让我们在 IDEA 中导入包含本章示例的存储库：

1.  打开 IntelliJ IDEA，选择从版本控制中检出，然后点击 Git。

1.  在 Git 存储库 URL 中输入 `https://bitbucket.org/vfarcic/tdd-java-ch02-example-junit.git`，然后点击克隆。确认 IDEA 的其余问题，直到从 Git 存储库克隆出带有代码的新项目。

导入的项目应该看起来类似于以下图片：

![](img/4f24bf69-eaa5-4aac-8877-275f383e142f.png)

现在我们已经设置好了项目，是时候来看一下单元测试框架了。

# 单元测试框架

在本节中，展示并简要评论了两个最常用的 Java 单元测试框架。我们将通过比较使用 JUnit 和 TestNG 编写的测试类来重点关注它们的语法和主要特性。尽管存在细微差异，但这两个框架都提供了最常用的功能，主要区别在于测试的执行和组织方式。

让我们从一个问题开始。什么是测试？我们如何定义它？

测试是一个可重复的过程或方法，用于验证在确定的情况下，对于确定的输入，期望预定义的输出或交互的被测试目标的正确行为。

在编程方法中，根据其范围，有几种类型的测试——功能测试、验收测试和单元测试。接下来，我们将更详细地探讨每种类型的测试。

单元测试是关于测试小代码片段的。让我们看看如何测试一个单独的 Java 类。这个类非常简单，但足够我们的兴趣：

```java
public class Friendships { 
  private final Map<String, List<String>> friendships = 
     new HashMap<>(); 

  public void makeFriends(String person1, String person2) { 
    addFriend(person1, person2); 
    addFriend(person2, person1); 
  } 

  public List<String> getFriendsList(String person) { 
    if (!friendships.containsKey(person)) { 
      return Collections.emptyList(); 
    } 
    return friendships.get(person)
  } 

  public boolean areFriends(String person1, String person2) { 
    return friendships.containsKey(person1) &&  
        friendships.get(person1).contains(person2); 
  } 

  private void addFriend(String person, String friend) { 
    if (!friendships.containsKey(person)) { 
      friendships.put(person, new ArrayList<String>()); 
    } 
    List<String> friends = friendships.get(person); 
    if (!friends.contains(friend)) { 
      friends.add(friend); 
    } 
  } 
} 
```

# JUnit

JUnit（[`junit.org/`](http://junit.org/)）是一个简单易学的编写和运行测试的框架。每个测试都被映射为一个方法，每个方法都应该代表一个特定的已知场景，在这个场景中，我们的代码的一部分将被执行。代码验证是通过比较预期输出或行为与实际输出来完成的。

以下是用 JUnit 编写的测试类。有一些场景缺失，但现在我们只关注展示测试的样子。我们将在本书的后面专注于测试代码的更好方法和最佳实践。

测试类通常包括三个阶段：设置、测试和拆卸。让我们从为测试设置所需数据的方法开始。设置可以在类或方法级别上执行：

```java
Friendships friendships; 

@BeforeClass 
public static void beforeClass() { 
  // This method will be executed once on initialization time 
} 

@Before 
public void before() { 
  friendships = new Friendships(); 
  friendships.makeFriends("Joe",",," "Audrey"); 
  friendships.makeFriends("Joe", "Peter"); 
  friendships.makeFriends("Joe", "Michael"); 
  friendships.makeFriends("Joe", "Britney"); 
  friendships.makeFriends("Joe", "Paul"); 
}
```

`@BeforeClass`注解指定一个方法，在类中的任何测试方法之前运行一次。这是一个有用的方法，可以进行一些通用设置，大多数（如果不是全部）测试都会用到。

`@Before`注解指定一个方法，在每个测试方法之前运行。我们可以使用它来设置测试数据，而不必担心之后运行的测试会改变该数据的状态。在前面的示例中，我们实例化了`Friendships`类，并向`Friendships`列表添加了五个样本条目。无论每个单独的测试将进行何种更改，这些数据都将一遍又一遍地重新创建，直到所有测试都完成。

这两个注解的常见用法包括设置数据库数据、创建测试所需的文件等。稍后，我们将看到如何使用模拟来避免外部依赖。然而，功能测试或集成测试可能仍然需要这些依赖，`@Before`和`@BeforeClass`注解是设置它们的好方法。

数据设置好后，我们可以进行实际的测试：

```java
@Test 
public void alexDoesNotHaveFriends() { 
  Assert.assertTrue("Alex does not have friends", 
     friendships.getFriendsList("Alex").isEmpty()); 
} 

@Test 
public void joeHas5Friends() { 
  Assert.assertEquals("Joe has 5 friends", 5, 
     friendships.getFriendsList("Joe").size()); 
} 

@Test 
public void joeIsFriendWithEveryone() { 
  List<String> friendsOfJoe =  
    Arrays.asList("Audrey", "Peter", "Michael", "Britney", "Paul"); 
  Assert.assertTrue(friendships.getFriendsList("Joe")
     .containsAll(friendsOfJoe)); 
} 
```

在这个例子中，我们使用了一些不同类型的断言。我们确认`Alex`没有任何朋友，而`Joe`是一个非常受欢迎的人，有五个朋友（`Audrey`、`Peter`、`Michael`、`Britney`和`Paul`）。

最后，一旦测试完成，我们可能需要进行一些清理工作：

```java
@AfterClass 
public static void afterClass() { 
  // This method will be executed once when all test are executed 
} 

@After 
public void after() { 
  // This method will be executed once after each test execution 
} 
```

在我们的例子中，在`Friendships`类中，我们不需要清理任何东西。如果有这样的需要，这两个注解将提供该功能。它们的工作方式类似于`@Before`和`@BeforeClass`注解。`@AfterClass`在所有测试完成后运行一次。`@After`注解在每个测试后执行。这将每个测试方法作为一个单独的类实例运行。只要我们避免全局变量和外部资源，比如数据库和 API，每个测试都是与其他测试隔离的。在一个测试中所做的任何事情都不会影响其他测试。

完整的源代码可以在`FriendshipsTest`类中找到，网址为[`bitbucket.org/vfarcic/tdd-java-ch02-example-junit`](https://bitbucket.org/vfarcic/tdd-java-ch02-example-junit)。

# TestNG

在 TestNG（[`testng.org/doc/index.html`](http://testng.org/doc/index.html)）中，测试被组织在类中，就像 JUnit 一样。

为了运行 TestNG 测试，需要以下 Gradle 配置（`build.gradle`）：

```java
dependencies { 
   testCompile group: 'org.testng', name: 'testng', version: '6.8.21' 
} 

test.useTestNG() { 
// Optionally you can filter which tests are executed using 
//    exclude/include filters 
// excludeGroups 'complex' 
} 
```

与 JUnit 不同，TestNG 需要额外的 Gradle 配置，告诉它使用 TestNG 来运行测试。

以下的测试类是用 TestNG 编写的，反映了我们之前用 JUnit 做的事情。重复的导入和其他无聊的部分被省略了，以便专注于相关部分：

```java
@BeforeClass 
public static void beforeClass() { 
  // This method will be executed once on initialization time 
} 

@BeforeMethod 
public void before() { 
  friendships = new Friendships(); 
  friendships.makeFriends("Joe", "Audrey"); 
  friendships.makeFriends("Joe", "Peter"); 
  friendships.makeFriends("Joe", "Michael"); 
  friendships.makeFriends("Joe", "Britney"); 
  friendships.makeFriends("Joe", "Paul"); 
} 
```

您可能已经注意到了 JUnit 和 TestNG 之间的相似之处。两者都使用注解来指定某些方法的目的。除了不同的名称（`@Beforeclass`与`@BeforeMethod`），两者之间没有区别。然而，与 JUnit 不同，TestNG 会为所有测试方法重用相同的测试类实例。这意味着测试方法默认情况下不是隔离的，因此在`before`和`after`方法中需要更多的注意。

断言也非常相似：

```java
public void alexDoesNotHaveFriends() { 
  Assert.assertTrue(friendships.getFriendsList("Alex").isEmpty(), 
      "Alex does not have friends"); 
} 

public void joeHas5Friends() { 
  Assert.assertEquals(friendships.getFriendsList("Joe").size(), 
      5, "Joe has 5 friends"); 
} 

public void joeIsFriendWithEveryone() { 
  List<String> friendsOfJoe = 
    Arrays.asList("Audrey", "Peter", "Michael", "Britney", "Paul");
  Assert.assertTrue(friendships.getFriendsList("Joe")
      .containsAll(friendsOfJoe)); 
} 
```

与 JUnit 相比，唯一显著的区别是`assert`变量的顺序。虽然 JUnit 的断言参数顺序是**可选消息**、**预期值**和**实际值**，TestNG 的顺序是实际值、预期值和可选消息。除了我们传递给`assert`方法的参数顺序不同之外，JUnit 和 TestNG 之间几乎没有区别。

您可能已经注意到缺少`@Test`。TestNG 允许我们在类级别上设置它，从而将所有公共方法转换为测试。

`@After`注解也非常相似。唯一显著的区别是 TestNG 的`@AfterMethod`注解，其作用方式与 JUnit 的`@After`注解相同。

如您所见，语法非常相似。测试被组织成类，并且使用断言进行测试验证。这并不是说这两个框架之间没有更重要的区别；我们将在本书中看到其中一些区别。我邀请您自行探索 JUnit（[`junit.org/`](http://junit.org/)）和 TestNG（[`testng.org/`](http://testng.org/)）。

前面例子的完整源代码可以在[`bitbucket.org/vfarcic/tdd-java-ch02-example-testng`](https://bitbucket.org/vfarcic/tdd-java-ch02-example-testng)找到。

到目前为止，我们编写的断言只使用了测试框架。然而，有一些测试工具可以帮助我们使它们更加美观和易读。

# Hamcrest 和 AssertJ

在前一节中，我们概述了单元测试是什么，以及如何使用两个最常用的 Java 框架编写单元测试。由于测试是我们项目的重要组成部分，为什么不改进我们编写测试的方式呢？一些很酷的项目出现了，旨在通过改变断言的方式来增强测试的语义。结果，测试更加简洁易懂。

# Hamcrest

**Hamcrest**添加了许多称为**匹配器**的方法。每个匹配器都设计用于执行比较操作。它足够灵活，可以支持自己创建的自定义匹配器。此外，JUnit 自带对 Hamcrest 的支持，因为其核心包含在 JUnit 分发中。您可以轻松开始使用 Hamcrest。但是，我们希望使用功能齐全的项目，因此我们将在 Gradle 的文件中添加一个测试依赖项：

```java
testCompile 'org.hamcrest:hamcrest-all:1.3' 
```

让我们将 JUnit 中的一个断言与 Hamcrest 中的等效断言进行比较：

+   JUnit 的`assert`：

```java
List<String> friendsOfJoe = 
  Arrays.asList("Audrey", "Peter", "Michael", "Britney", "Paul");
Assert.assertTrue( friendships.getFriendsList("Joe")
    .containsAll(friendsOfJoe)); 
```

+   Hamcrest 的`assert`：

```java
assertThat( 
  friendships.getFriendsList("Joe"), 
  containsInAnyOrder("Audrey", "Peter", "Michael", "Britney", "Paul") 
); 
```

正如你所看到的，Hamcrest 更具表现力。它具有更大范围的断言，可以避免一些样板代码，同时使代码更易于阅读和更具表现力。

这是另一个例子：

+   JUnit 的`assert`：

```java
Assert.assertEquals(5, friendships.getFriendsList("Joe").size()); 
```

+   Hamcrest 的`assert`：

```java
assertThat(friendships.getFriendsList("Joe"), hasSize(5)); 
```

您会注意到两个区别。首先是，与 JUnit 不同，Hamcrest 几乎总是直接使用对象。在 JUnit 的情况下，我们需要获取整数大小并将其与预期数字（`5`）进行比较；而 Hamcrest 具有更大范围的断言，因此我们可以简单地使用其中一个（`hasSize`）与实际对象（`List`）一起使用。另一个区别是，Hamcrest 具有与实际值相反的顺序，实际值是第一个参数（就像 TestNG 一样）。

这两个例子还不足以展示 Hamcrest 所提供的全部潜力。在本书的后面，将会有更多关于 Hamcrest 的例子和解释。访问[`hamcrest.org/`](http://hamcrest.org/)并探索其语法。

完整的源代码可以在`FriendshipsHamcrestTest`类中找到，网址为[`bitbucket.org/vfarcic/tdd-java-ch02-example-junit`](https://bitbucket.org/vfarcic/tdd-java-ch02-example-junit)。

# AssertJ

**AssertJ**的工作方式类似于 Hamcrest。一个主要的区别是 AssertJ 断言可以连接起来。

要使用 AssertJ，必须将依赖项添加到 Gradle 的依赖项中：

```java
testCompile 'org.assertj:assertj-core:2.0.0' 
```

让我们将 JUnit 断言与 AssertJ 进行比较：

```java
Assert.assertEquals(5, friendships.getFriendsList("Joe").size()); 
List<String> friendsOfJoe = 
   Arrays.asList("Audrey", "Peter", "Michael", "Britney", "Paul");
Assert.assertTrue(  friendships.getFriendsList("Joe")
   .containsAll (friendsOfJoe) 
); 
```

在 AssertJ 中，相同的两个断言可以连接成一个：

```java
assertThat(friendships.getFriendsList("Joe")) 
  .hasSize(5) 
  .containsOnly("Audrey", "Peter", "Michael", "Britney", "Paul");
```

这是一个不错的改进。不需要有两个单独的断言，也不需要创建一个包含预期值的新列表。此外，AssertJ 更易读，更容易理解。

完整的源代码可以在`FriendshipsAssertJTest`类中找到，网址为[`bitbucket.org/vfarcic/tdd-java-ch02-example-junit`](https://bitbucket.org/vfarcic/tdd-java-ch02-example-junit)。

现在我们已经有了运行的测试，我们可能想要查看我们的测试生成的代码覆盖率是多少。

# 代码覆盖率工具

我们编写测试并不意味着它们很好，也不意味着它们覆盖了足够的代码。一旦我们开始编写和运行测试，自然的反应就是开始提出以前无法回答的问题。我们的代码的哪些部分得到了适当的测试？我们的测试没有考虑到哪些情况？我们测试得足够吗？这些和其他类似的问题可以通过代码覆盖率工具来回答。它们可以用于识别我们的测试未覆盖的代码块或行；它们还可以计算代码覆盖的百分比并提供其他有趣的指标。

它们是用于获取指标并显示测试和实现代码之间关系的强大工具。然而，与任何其他工具一样，它们的目的需要明确。它们不提供关于质量的信息，而只提供我们的代码中已经测试过的部分。

代码覆盖率显示测试执行期间是否到达了代码行，但这并不是良好测试实践的保证，因为测试质量不包括在这些指标中。

让我们来看看用于计算代码覆盖率的最流行的工具之一。

# JaCoCo

Java 代码覆盖率（JaCoCo）是一个用于测量测试覆盖率的知名工具。

要在我们的项目中使用它，我们需要在 Gradle 配置文件`build.gradle`中添加几行：

1.  为 JaCoCo 添加 Gradle`plugin`：

```java
apply plugin: 'jacoco'
```

1.  要查看 JaCoCo 的结果，请从命令提示符中运行以下命令：

```java
gradle test jacocoTestReport
```

1.  相同的 Gradle 任务可以从 Gradle 任务 IDEA 工具窗口运行。

1.  最终结果存储在`build/reports/jacoco/test/html`目录中。这是一个可以在任何浏览器中打开的 HTML 文件：

![](img/2d6b19f8-0fbc-4101-88d0-46a046cd900a.png)

本书的后续章节将更详细地探讨代码覆盖率。在那之前，可以访问[`www.eclemma.org/jacoco/`](http://www.eclemma.org/jacoco/)获取更多信息。

# 模拟框架

我们的项目看起来很酷，但它太简单了，远非一个真正的项目。它仍然没有使用外部资源。Java 项目需要数据库，因此我们将尝试引入它。

测试使用外部资源或第三方库的代码的常见方法是什么？模拟是答案。模拟对象，或者简单地说是模拟，是一个可以用来替代真实对象的模拟对象。当依赖外部资源的对象被剥夺时，它们非常有用。

实际上，在开发应用程序时根本不需要数据库。相反，您可以使用模拟来加快开发和测试，并且只在运行时使用真实的数据库连接。我们可以专注于编写类并在集成时考虑它们，而不是花时间设置数据库和准备测试数据。

为了演示目的，我们将介绍两个新类：`Person`类和`FriendCollection`类，它们旨在表示人和数据库对象映射。持久性将使用 MongoDB 进行（[`www.mongodb.org/`](https://www.mongodb.org/)）。

我们的示例将有两个类。`Person`将表示数据库对象数据；`FriendCollection`将是我们的数据访问层。代码是自解释的。

让我们创建并使用`Person`类：

```java
public class Person { 
  @Id
  private String name; 

  private List<String> friends; 

  public Person() { } 

  public Person(String name) { 
    this.name = name; 
    friends = new ArrayList<>(); 
  } 

  public List<String> getFriends() { 
    return friends; 
  } 

  public void addFriend(String friend) { 
    if (!friends.contains(friend)) friends.add(friend); 
  }
}
```

让我们创建并使用`FriendsCollection`类：

```java
public class FriendsCollection { 
  private MongoCollection friends; 

  public FriendsCollection() { 
    try { 
      DB db = new MongoClient().getDB("friendships"); 
      friends = new Jongo(db).getCollection("friends"); 
    } catch (UnknownHostException e) { 
      throw new RuntimeException(e.getMessage()); 
    } 
  } 

  public Person findByName(String name) { 
    return friends.findOne("{_id: #}", name).as(Person.class); 
  } 

  public void save(Person p) { 
    friends.save(p); 
  } 
} 
```

此外，还引入了一些新的依赖项，因此 Gradle 依赖块需要进行修改。第一个是 MongoDB 驱动程序，它用于连接到数据库。第二个是 Jongo，一个使访问 Mongo 集合非常简单的小项目。

`mongodb`和`jongo`的 Gradle 依赖如下：

```java
dependencies { 
    compile 'org.mongodb:mongo-java-driver:2.13.2' 
    compile 'org.jongo:jongo:1.1' 
} 
```

我们正在使用数据库，因此`Friendships`类也应该被修改。我们应该将一个映射更改为`FriendsCollection`并修改其余代码以使用它。最终结果如下：

```java
public class FriendshipsMongo { 
  private FriendsCollection friends; 

  public FriendshipsMongo() { 
    friends = new FriendsCollection(); 
  } 

  public List<String> getFriendsList(String person) { 
    Person p = friends.findByName(person); 
    if (p == null) return Collections.emptyList(); 
    return p.getFriends(); 
  } 

  public void makeFriends(String person1, String person2) { 
    addFriend(person1, person2); 
    addFriend(person2, person1); 
  } 

  public boolean areFriends(String person1, String person2) { 
    Person p = friends.findByName(person1); 
    return p != null && p.getFriends().contains(person2); 
  } 

  private void addFriend(String person, String friend) {
    Person p = friends.findByName(person); 
    if (p == null) p = new Person(person); 
    p.addFriend(friend); 
    friends.save(p); 
  } 
} 
```

完整的源代码可以在[`bitbucket.org/vfarcic/tdd-java-ch02-example-junit`](https://bitbucket.org/vfarcic/tdd-java-ch02-example-junit)存储库中的`FriendsCollection`和`FriendshipsMongo`类中找到。

现在我们的`Friendships`类已经可以与 MongoDB 一起工作，让我们看看如何使用模拟来测试它的一种可能方式。

# Mockito

Mockito 是一个允许轻松创建测试替身的 Java 框架。

Gradle 依赖如下：

```java
dependencies { 
  testCompile group: 'org.mockito', name: 'mockito-all', version: '1.+' 
} 
```

Mockito 通过 JUnit 运行。它为我们创建了所有必需的模拟对象，并将它们注入到具有测试的类中。有两种基本方法；通过自己实例化模拟对象并通过类构造函数将它们注入为类依赖项，或者使用一组注解。在下一个示例中，我们将看到如何使用注解来完成。

为了使一个类使用 Mockito 注解，它需要使用`MockitoJUnitRunner`运行。使用该运行程序简化了该过程，因为您只需向要创建的对象添加注解即可：

```java
@RunWith(MockitoJUnitRunner.class) 
public class FriendshipsTest { 
... 
} 
```

在您的测试类中，被测试的类应该用`@InjectMocks`注解。这告诉 Mockito 要将模拟对象注入哪个类：

```java
@InjectMocks 
FriendshipsMongo friendships; 
```

从那时起，我们可以指定在类内部的特定方法或对象，即`FriendshipsMongo`，将被替换为模拟对象：

```java
@Mock 
FriendsCollection friends; 
```

在这个例子中，`FriendshipsMongo`类中的`FriendsCollection`将被模拟。

现在，我们可以指定在调用`friends`时应返回什么：

```java
Person joe = new Person("Joe"); 
doReturn(joe).when(friends).findByName("Joe"); 
assertThat(friends.findByName("Joe")).isEqualTo(joe); 
```

在这个例子中，我们告诉 Mockito 当调用`friends.findByName("Joe")`时返回`joe`对象。稍后，我们使用`assertThat`来验证这个假设是正确的。

让我们尝试在之前没有 MongoDB 的类中做与之前相同的测试：

```java
@Test 
public void joeHas5Friends() { 
  List<String> expected = 
    Arrays.asList("Audrey", "Peter", "Michael", "Britney", "Paul"); 
  Person joe = spy(new Person("Joe")); 

  doReturn(joe).when(friends).findByName("Joe"); 
  doReturn(expected).when(joe).getFriends(); 

  assertThat(friendships.getFriendsList("Joe")) 
    .hasSize(5) 
    .containsOnly("Audrey", "Peter", "Michael", "Britney", "Paul"); 
} 
```

在这个小测试中发生了很多事情。首先，我们指定`joe`是一个间谍。在 Mockito 中，间谍是真实对象，除非另有规定，否则使用真实方法。然后，我们告诉 Mockito 当`friends`方法调用`getFriends`时返回`joe`。这种组合允许我们在调用`getFriends`方法时返回`expected`列表。最后，我们断言`getFriendsList`返回预期的名称列表。

完整的源代码可以在[`bitbucket.org/vfarcic/tdd-java-ch02-example-junit`](https://bitbucket.org/vfarcic/tdd-java-ch02-example-junit)存储库中的`FriendshipsMongoAssertJTest`类中找到。

我们将在后面使用 Mockito；在本书中，您将有机会更加熟悉它和一般的模拟。有关 Mockito 的更多信息，请访问[`mockito.org/`](http://mockito.org/)。

# EasyMock

EasyMock 是一种替代的模拟框架。它与 Mockito 非常相似。然而，主要区别在于 EasyMock 不创建`spy`对象，而是模拟对象。其他区别是语法上的。

让我们看一个 EasyMock 的例子。我们将使用与 Mockito 示例相同的一组测试用例：

```java
@RunWith(EasyMockRunner.class) 
public class FriendshipsTest { 
  @TestSubject 
  FriendshipsMongo friendships = new FriendshipsMongo(); 
  @Mock(type = MockType.NICE) 
  FriendsCollection friends;
}
```

基本上，运行器与 Mockito 运行器的功能相同：

```java
@TestSubject 
FriendshipsMongo friendships = new FriendshipsMongo(); 

@Mock(type = MockType.NICE) 
FriendsCollection friends; 
```

`@TestSubject`注解类似于 Mockito 的`@InjectMocks`，而`@Mock`注解表示要以类似于 Mockito 的方式模拟的对象。此外，类型`NICE`告诉模拟返回空值。

让我们比较一下我们用 Mockito 做的一个断言：

```java
@Test 
public void mockingWorksAsExpected() { 
  Person joe = new Person("Joe"); 
  expect(friends.findByName("Joe")).andReturn(joe); 
  replay(friends); 
  assertThat(friends.findByName("Joe")).isEqualTo(joe); 
} 
```

除了语法上的小差异外，EasyMock 唯一的缺点是需要额外的指令`replay`。它告诉框架应用先前指定的期望。其余几乎相同。我们指定`friends.findByName`应返回`joe`对象，应用该期望，并最后断言实际结果是否符合预期。

在 EasyMock 版本中，我们使用 Mockito 的第二个测试方法如下：

```java
@Test 
public void joeHas5Friends() { 
  List<String> expected = 
  Arrays.asList("Audrey", "Peter", "Michael", "Britney", "Paul"); 
  Person joe = createMock(Person.class); 

  expect(friends.findByName("Joe")).andReturn(joe); 
  expect(joe.getFriends()).andReturn(expected); 
  replay(friends); 
  replay(joe); 

  assertThat(friendships.getFriendsList("Joe")) 
    .hasSize(5)
    .containsOnly("Audrey", "Peter", "Michael", "Britney", "Paul"); 
}
```

与 Mockito 相比，EasyMock 几乎没有区别，只是 EasyMock 没有间谍。根据上下文，这可能是一个重要的区别。

尽管这两个框架都很相似，但有一些细节使我们选择 Mockito 作为框架，这将在本书中使用。

有关此断言库的更多信息，请访问[`easymock.org/`](http://easymock.org/)。

完整的源代码可以在`FriendshipsMongoEasyMockTest`类中找到，该类位于[`bitbucket.org/vfarcic/tdd-java-ch02-example-junit`](https://bitbucket.org/vfarcic/tdd-java-ch02-example-junit)存储库中。

# 模拟的额外功能

前面介绍的两个项目都不涵盖所有类型的方法或字段。根据应用的修饰符，如静态或最终，类、方法或字段可能超出 Mockito 或 EasyMock 的范围。在这种情况下，我们可以使用 PowerMock 来扩展模拟框架。这样，我们可以模拟只能以棘手方式模拟的对象。但是，使用 PowerMock 时应该谨慎，因为使用它提供的许多功能通常是设计不良的标志。如果您正在处理遗留代码，PowerMock 可能是一个不错的选择。否则，尽量设计您的代码，以便不需要 PowerMock。我们稍后会向您展示如何做到这一点。

有关更多信息，请访问[`code.google.com/p/powermock/`](https://code.google.com/p/powermock/)。

# 用户界面测试

尽管单元测试可以并且应该覆盖应用程序的主要部分，但仍然需要进行功能和验收测试。与单元测试不同，它们提供了更高级别的验证，并且通常在入口点执行，并且严重依赖于用户界面。最终，我们创建的应用程序在大多数情况下都是由人类使用的，因此对应用程序行为的信心非常重要。通过测试应用程序从真实用户的角度来看应该做什么，可以实现这种舒适状态。

在这里，我们将尝试通过用户界面提供功能和验收测试的概述。我们将以网络为例，尽管还有许多其他类型的用户界面，如桌面应用程序、智能手机界面等。

# Web 测试框架

本章中已经测试了应用程序类和数据源，但仍然缺少一些东西；最常见的用户入口点——网络。大多数企业应用程序，如内部网或公司网站，都是使用浏览器访问的。因此，测试网络提供了重要的价值，帮助我们确保它正在按预期进行操作。

此外，公司正在花费大量时间进行长时间和繁重的手动测试，每次应用程序更改时都要进行测试。这是一种浪费时间，因为其中许多测试可以通过工具（如 Selenium 或 Selenide）进行自动化和无人监督地执行。

# Selenium

Selenium 是一个用于 Web 测试的强大工具。它使用浏览器来运行验证，并且可以处理所有流行的浏览器，如 Firefox、Safari 和 Chrome。它还支持无头浏览器，以更快的速度和更少的资源消耗测试网页。

有一个`SeleniumIDE`插件，可以用来通过记录用户执行的操作来创建测试。目前，它只支持 Firefox。遗憾的是，尽管以这种方式生成的测试提供了非常快速的结果，但它们往往非常脆弱，并且在长期内会引起问题，特别是当页面的某些部分发生变化时。因此，我们将坚持不使用该插件的帮助编写的代码。

执行 Selenium 最简单的方法是通过`JUnitRunner`运行它。

所有 Selenium 测试都是通过初始化`WebDriver`开始的，这是用于与浏览器通信的类：

1.  让我们从添加 Gradle 依赖开始：

```java
dependencies { 
  testCompile 'org.seleniumhq.selenium:selenium-java:2.45.0' 
} 
```

1.  例如，我们将创建一个搜索维基百科的测试。我们将使用 Firefox 驱动程序作为我们的首选浏览器：

```java
WebDriver driver = new FirefoxDriver(); 
```

`WebDriver`是一个可以用 Selenium 提供的众多驱动程序之一实例化的接口：

1.  要打开一个 URL，指令如下：

```java
driver.get("http://en.wikipedia.org/wiki/Main_Page");
```

1.  页面打开后，我们可以通过其名称搜索输入元素，然后输入一些文本：

```java
WebElement query = driver.findElement(By.name("search")); 
query.sendKeys("Test-driven development"); 
```

1.  一旦我们输入我们的搜索查询，我们应该找到并点击 Go 按钮：

```java
WebElement goButton = driver.findElement(By.name("go")); 
goButton.click();
```

1.  一旦到达目的地，就是验证，在这种情况下，页面标题是否正确的时候了：

```java
assertThat(driver.getTitle(), 
  startsWith("Test-driven development"));
```

1.  最后，一旦我们使用完毕，`driver`应该被关闭：

```java
driver.quit(); 
```

就是这样。我们有一个小但有价值的测试，可以验证单个用例。虽然

关于 Selenium 还有很多要说的，希望这为您提供了

有足够的信息来认识到它的潜力。

访问[`www.seleniumhq.org/`](http://www.seleniumhq.org/)获取更多信息和更复杂的`WebDriver`使用。

完整的源代码可以在`SeleniumTest`类中的[`bitbucket.org/vfarcic/tdd-java-ch02-example-web`](https://bitbucket.org/vfarcic/tdd-java-ch02-example-web)存储库中找到。

虽然 Selenium 是最常用的与浏览器一起工作的框架，但它仍然是非常低级的，需要大量的调整。Selenide 的诞生是基于这样一个想法，即如果有一个更高级的库可以实现一些常见模式并解决经常重复的需求，那么 Selenium 将会更有用。

# Selenide

关于 Selenium 我们所看到的非常酷。它为我们提供了探测应用程序是否正常运行的机会，但有时配置和使用起来有点棘手。Selenide 是一个基于 Selenium 的项目，提供了一个良好的语法来编写测试，并使它们更易读。它为您隐藏了`WebDriver`和配置的使用，同时仍然保持了高度的定制性：

1.  与我们到目前为止使用的所有其他库一样，第一步是添加 Gradle 依赖：

```java
dependencies { 
    testCompile 'com.codeborne:selenide:2.17' 
}
```

1.  让我们看看如何使用 Selenide 编写之前的 Selenium 测试

相反。语法可能对那些了解 JQuery 的人来说很熟悉([`jquery.com/`](https://jquery.com/))：

```java
public class SelenideTest { 
  @Test 
  public void wikipediaSearchFeature() throws 
      InterruptedException { 

    // Opening Wikipedia page 
    open("http://en.wikipedia.org/wiki/Main_Page"); 

    // Searching TDD 
    $(By.name("search")).setValue("Test-driven development"); 

    // Clicking search button 
    $(By.name("go")).click(); 

    // Checks 
    assertThat(title(),
      startsWith("Test-driven development")); 
  } 
} 
```

这是一种更具表现力的测试编写方式。除了更流畅的语法之外，这段代码背后还发生了一些事情，需要额外的 Selenium 代码行。例如，单击操作将等待直到相关元素可用，并且只有在预定义的时间段过期时才会失败。另一方面，Selenium 会立即失败。在当今世界，许多元素通过 JavaScript 动态加载，我们不能指望一切立即出现。因此，这个 Selenide 功能被证明是有用的，并且可以避免使用重复的样板代码。Selenide 带来了许多其他好处。由于 Selenide 相对于 Selenium 提供的好处，它将成为我们在整本书中选择的框架。此外，有一个完整的章节专门介绍了使用这个框架进行 Web 测试。访问[`selenide.org/`](http://selenide.org/)获取有关在测试中使用 Web 驱动程序的更多信息。

无论测试是用一个框架还是另一个框架编写的，效果都是一样的。运行测试时，Firefox 浏览器窗口将出现并按顺序执行测试中定义的所有步骤。除非选择了无头浏览器作为您的驱动程序，否则您将能够看到测试过程。如果出现问题，将提供失败跟踪。除此之外，我们可以在任何时候拍摄浏览器截图。例如，在失败时记录情况是一种常见做法。

完整的源代码可以在`SelenideTest`类中找到

[`bitbucket.org/vfarcic/tdd-java-ch02-example-web`](https://bitbucket.org/vfarcic/tdd-java-ch02-example-web) 仓库中。

掌握了基本的 Web 测试框架知识，现在是时候简要了解一下 BDD 了。

# 行为驱动开发

**行为驱动开发**（**BDD**）是一种旨在在整个项目过程中保持对利益相关者价值的关注的敏捷过程。BDD 的前提是，需求必须以每个人都能理解的方式编写，无论是业务代表、分析师、开发人员、测试人员、经理等等。关键在于拥有一组独特的工件，每个人都能理解和使用——一系列用户故事。故事由整个团队编写，并用作需求和可执行测试用例。这是一种以无法通过单元测试实现的清晰度执行 TDD 的方式。这是一种以（几乎）自然语言描述和测试功能的方式，并使其可运行和可重复。

一个故事由场景组成。每个场景代表一个简洁的行为用例，并使用步骤以自然语言编写。步骤是场景的前提条件、事件和结果的序列。每个步骤必须以`Given`、`When`或`Then`开头。`Given`用于前提条件，`When`用于操作，`Then`用于执行验证。

这只是一个简要介绍。有一个完整的章节，第八章，*BDD - 与整个团队一起工作*，专门介绍了这个主题。现在是时候介绍 JBehave 和 Cucumber 作为许多可用框架之一，用于编写和执行故事。

# JBehave

JBehave 是一个用于编写可执行和自动化的验收测试的 Java BDD 框架。故事中使用的步骤通过框架提供的几个注解绑定到 Java 代码：

1.  首先，将 JBehave 添加到 Gradle 依赖项中：

```java
dependencies { 
    testCompile 'org.jbehave:jbehave-core:3.9.5' 
}
```

1.  让我们通过一些示例步骤：

```java
@Given("I go to Wikipedia homepage") 
public void goToWikiPage() { 
  open("http://en.wikipedia.org/wiki/Main_Page"); 
} 
```

1.  这是`Given`类型的步骤。它代表需要满足的前提条件，以便成功执行一些操作。在这种情况下，它将打开一个维基百科页面。现在我们已经指定了我们的前提条件，是时候定义一些操作了：

```java
@When("I enter the value $value on a field named $fieldName")
public void enterValueOnFieldByName(String value, String fieldName) { 
  $(By.name(fieldName)).setValue(value); 
} 
@When("I click the button $buttonName") 
public void clickButonByName(String buttonName){ 
  $(By.name(buttonName)).click(); 
} 
```

1.  正如您所看到的，操作是使用`When`注释定义的。在我们的情况下，我们可以使用这些步骤来为字段设置一些值或单击特定按钮。一旦操作完成，我们可以处理验证。注意

通过引入参数，步骤可以更加灵活：

```java
@Then("the page title contains $title") 
public void pageTitleIs(String title) { 
  assertThat(title(), containsString(title)); 
} 
```

使用`Then`注释声明验证。在这个例子中，我们正在验证页面标题是否符合预期。

这些步骤可以在[`bitbucket.org/vfarcic/tdd-java-ch02-example-web`](https://bitbucket.org/vfarcic/tdd-java-ch02-example-web)存储库中的`WebSteps`类中找到。

一旦我们定义了我们的步骤，就是使用它们的时候了。以下故事结合了这些步骤，以验证所需的行为：

```java
Scenario: TDD search on wikipedia 
```

它以命名场景开始。名称应尽可能简洁，但足以明确识别用户案例；仅供信息目的：

```java
Given I go to Wikipedia homepage 
When I enter the value Test-driven development on a field named search 
When I click the button go 
Then the page title contains Test-driven development 
```

正如您所看到的，我们正在使用之前定义的相同步骤文本。与这些步骤相关的代码将按顺序执行。如果其中任何一个被停止，执行也将停止，该场景本身被视为失败。

尽管我们在故事之前定义了我们的步骤，但也可以反过来，先定义故事，然后是步骤。在这种情况下，场景的状态将是挂起的，这意味着缺少所需的步骤。

这个故事可以在[`bitbucket.org/vfarcic/tdd-java-ch02-example-web`](https://bitbucket.org/vfarcic/tdd-java-ch02-example-web)存储库中的`wikipediaSearch.story`文件中找到。

要运行这个故事，执行以下操作：

```java
$> gradle testJBehave
```

故事运行时，我们可以看到浏览器中正在发生的操作。一旦完成，将生成执行结果的报告。它可以在`build/reports/jbehave`中找到：

![](img/dca745ce-04ef-448e-a573-18d765ad28e2.png)

JBehave 故事执行报告

为了简洁起见，我们排除了运行 JBehave 故事的`build.gradle`代码。完成的源代码可以在[`bitbucket.org/vfarcic/tdd-java-ch02-example-web`](https://bitbucket.org/vfarcic/tdd-java-ch02-example-web)存储库中找到。

有关 JBehave 及其优势的更多信息，请访问[`jbehave.org/`](http://jbehave.org/)。

# Cucumber

Cucumber 最初是一个 Ruby BDD 框架。如今它支持包括 Java 在内的多种语言。它提供的功能与 JBehave 非常相似。

让我们看看用 Cucumber 写的相同的例子。

与我们到目前为止使用的任何其他依赖项一样，Cucumber 需要在我们开始使用它之前添加到`build.gradle`中：

```java
dependencies { 
    testCompile 'info.cukes:cucumber-java:1.2.2' 
    testCompile 'info.cukes:cucumber-junit:1.2.2' 
} 
```

我们将使用 Cucumber 的方式创建与 JBehave 相同的步骤：

```java
@Given("^I go to Wikipedia homepage$") 
public void goToWikiPage() { 
  open("http://en.wikipedia.org/wiki/Main_Page"); 
} 

@When("^I enter the value (.*) on a field named (.*)$") 
public void enterValueOnFieldByName(String value, 
    String fieldName) { 
  $(By.name(fieldName)).setValue(value); 
} 

@When("^I click the button (.*)$") 
public void clickButonByName(String buttonName) { 
  $(By.name(buttonName)).click(); 
} 

@Then("^the page title contains (.*)$") 
public void pageTitleIs(String title) { 
  assertThat(title(), containsString(title)); 
} 
```

这两个框架之间唯一显着的区别是 Cucumber 定义步骤文本的方式。它使用正则表达式来匹配变量类型，而 JBehave 则是根据方法签名推断它们。

这些步骤代码可以在[`bitbucket.org/vfarcic/tdd-java-ch02-example-web`](https://bitbucket.org/vfarcic/tdd-java-ch02-example-web)存储库中的`WebSteps`类中找到：

让我们看看使用 Cucumber 语法编写的故事是什么样子的：

```java
Feature: Wikipedia Search 

  Scenario: TDD search on wikipedia 
    Given I go to Wikipedia homepage 
    When I enter the value Test-driven development on a field named search 
    When I click the button go 
    Then the page title contains Test-driven development 
```

请注意，几乎没有区别。这个故事可以在[`bitbucket.org/vfarcic/tdd-java-ch02-example-web`](https://bitbucket.org/vfarcic/tdd-java-ch02-example-web)存储库中的`wikipediaSearch.feature`文件中找到。

您可能已经猜到，要运行 Cucumber 故事，您只需要运行以下 Gradle 任务：

```java
$> gradle testCucumber
```

结果报告位于`build/reports/cucumber-report`目录中。这是前面故事的报告：

![](img/00ef22c3-dab2-47a2-a9b5-9677e2487c12.png)

Cucumber 故事执行报告

完整的代码示例可以在[`bitbucket.org/vfarcic/tdd-java-ch02-example-web`](https://bitbucket.org/vfarcic/tdd-java-ch02-example-web)存储库中找到。

有关 Cucumber 支持的语言列表或任何其他详细信息，请访问[`cukes.info/`](https://cukes.info/)。

由于 JBehave 和 Cucumber 都提供了类似的功能，我们决定在本书的其余部分中都使用 JBehave。还有一个专门的章节介绍 BDD 和 JBehave。

# 总结

在这一章中，我们暂时停止了 TDD，并介绍了许多在接下来的章节中用于代码演示的工具和框架。我们从版本控制、虚拟机、构建工具和 IDE 一直设置到如今常用的测试工具框架。

我们是开源运动的坚定支持者。秉承这种精神，我们特别努力地选择了每个类别中的免费工具和框架。

现在我们已经设置好了所有需要的工具，在下一章中，我们将深入探讨 TDD，从 Red-Green-Refactor 过程-TDD 的基石开始。
