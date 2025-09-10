# 第五章 多项目构建

现在我们已经熟悉了构建脚本语法，我们准备处理更复杂的项目结构。在本章中，我们将重点关注跨越多个项目的构建，它们之间的相互依赖关系，以及许多其他相关内容。

随着项目代码库的增长，很多时候，根据层、责任、生成的工件，有时甚至根据开发团队来将其拆分为多个模块是很有必要的，以有效地分解工作。无论原因是什么，现实是大型项目迟早会被拆分为更小的子项目。此外，像 Gradle 这样的构建工具完全能够处理这种复杂性。

# 多项目目录布局

多项目（或称为多模块，有些人更喜欢这样称呼）是一组逻辑上相互关联的项目，通常具有相同的开发-构建-发布周期。目录结构对于布局此类项目的策略非常重要。通常，顶级根项目包含一个或多个子项目。根项目可能包含自己的源集，可能只包含集成测试，这些测试用于测试子项目的集成，或者甚至仅作为一个没有源和测试的母构建。Gradle 支持所有此类配置。

子项目相对于根项目的排列可能是平的，也就是说，所有子项目都是根项目的直接子项目（如样本 1 所示）或者是有层次的，这样子项目也可能有嵌套的子项目（如样本 2 所示）或者任何混合的目录结构。

让我们将以下目录结构称为样本 1：

```java
sample1
├── repository
├── services
└── web-app

```

在样本 1 中，我们看到一个虚构的示例项目，其中所有子项目都是根项目的直接子项目，彼此之间是兄弟关系。仅为了这个例子，我们将我们的应用程序拆分为三个子项目，分别命名为`:repository`、`:services`和`:web-app`。正如它们的名称所暗示的，一个仓库包含数据访问代码，而服务层封装了以可消费 API 形式存在的业务规则。`web-app`只包含特定于 Web 应用程序的代码，例如控制器和视图模板。然而，请注意，`:web-app`项目可能依赖于`:services`项目，而`:services`项目反过来可能依赖于`:repository`项目。我们很快就会看到这些依赖是如何工作的。

### 小贴士

不要将多项目结构与单个项目中的多个源目录混淆。

让我们看看一个相对更复杂的结构，并将其称为样本 2：

```java
sample2
├── core
│   ├── models
│   ├── repository
│   └── services
├── client
│   ├── client-api
│   ├── cli-client
│   └── desktop-client
└── web 
 ├── webservices
 └── webapp

```

我们的应用现在已经进化，为了满足更多需求，我们为其添加了更多功能。我们创建了更多子项目，例如应用桌面客户端和命令行界面。在示例 2 中，根项目被分割成三个项目（组），它们各自有自己的子项目。在这个例子中，每个目录都可以被视为一个项目。这个示例的目的是仅展示可能的目录结构之一。Gradle 不会强制使用一种目录结构代替另一种。

一个人可能会想知道，我们把这些`build.gradle`文件放在哪里，里面有什么内容？这取决于我们的需求和如何构建我们的项目结构。在我们理解了什么是`settings.gradle`之后，我们将很快回答所有这些问题。

# settings.gradle 文件

在初始化过程中，Gradle 读取`settings.gradle`文件以确定哪些项目将参与构建。Gradle 创建一个类型为`Setting`的对象。这发生在任何`build.gradle`被解析之前。它通常放置在根项目与`build.gradle`平行的地方。建议将`setting.gradle`放在根项目中，否则我们必须使用命令行选项`-c`显式告诉 Gradle 设置文件的路径。将这两个文件添加到示例 1 的目录结构中，我们会得到如下内容：

```java
sample1
├── repository
│   └── ...
├── services
│   └── ...
├── web-app
│   └── ...
├── build.gradle
└── settings.gradle

```

`settings.gradle`最常见的使用是将所有参与构建的子项目列出：

```java
include ':repository', ':services', ':web-app'
```

此外，这就是告诉 Gradle 当前构建是一个多项目构建所需的所有内容。当然，这并不是故事的结尾，我们还可以在多项目构建中做更多的事情，但这是最基本的要求，有时也足以让多项目构建工作。

`Settings`的方法和属性在`settings.gradle`文件中可用，并且在对`Settings`实例进行操作时隐式调用，就像我们在上一章中看到的那样，`Project` API 的方法在`build.gradle`文件中可用。

### 注意

你是否想知道为什么在上一节的项目名称前使用了冒号（`:`）？它表示相对于根项目的项目路径。然而，`include`方法允许一级子项目名称省略冒号。因此，`include`调用可以重写如下：

```java
include 'repository', 'services', 'web-app'
```

让我们只通过从命令行调用任务`projects`来查询项目。`projects`任务列出了 Gradle 构建中可用的所有项目：

```java
$ gradle projects
:projects
------------------------------------------------------------
Root project
------------------------------------------------------------

Root project 'sample1'
+--- Project ':repository'
+--- Project ':services'
\--- Project ':web-app'

To see a list of the tasks of a project, run gradle <project-path>:tasks.
For example, try running gradle :repository:tasks.

BUILD SUCCESSFUL

```

### 注意

在嵌套超过一个级别的场景中，例如在示例 2 中，所有项目都必须使用以下语法包含在根项目的`settings.gradle`中：

```java
include 'core',
  'core:models', 'core:repository', 'core:services',
  'client' //... so on
```

我们可以在 *Settings* DSL 文档（[`www.gradle.org/docs/current/dsl/org.gradle.api.initialization.Settings.html`](http://www.gradle.org/docs/current/dsl/org.gradle.api.initialization.Settings.html)）和 *Settings* API 文档（[`www.gradle.org/docs/current/javadoc/org/gradle/api/initialization/Settings.html`](http://www.gradle.org/docs/current/javadoc/org/gradle/api/initialization/Settings.html)）中找到更多关于 `Settings` 的信息。

# 在多项目构建中组织构建逻辑

Gradle 给我们提供了创建一个构建文件用于所有项目或每个项目单独的构建文件；你也可以混合使用。让我们从向根项目的 `build.gradle` 中添加一个简单的任务开始：

```java
task sayHello << {
    println "Hello from multi-project build"
}
```

我们正在创建一个带有打印消息动作的任务。现在，让我们检查根项目中可用的任务。从 `root` 目录，让我们调用任务 `tasks`：

```java
$ gradle tasks
...

Other tasks
-----------
sayHello

....

```

没有疑问，`sayHello` 任务在根项目中可用。然而，如果我们只想查看子项目上可用的任务怎么办？比如说 `:repository`。对于多项目构建，我们可以使用 `gradle <project-path>:<task-name>` 语法或进入子项目目录并执行 `gradle <task-name>` 来调用任何嵌套项目的任务。所以现在，如果我们执行以下代码，我们将看不到 `sayHello` 任务：

```java
$ gradle repository:tasks

```

这是因为 `sayHello` 只在根项目中定义；因此，它不在子项目中可用。

## 将构建逻辑应用于所有项目

```java
build.gradle:
```

```java
allprojects {
    task whoami << {println "I am ${project.name}"}
}
```

在尝试理解代码片段之前，让我们再次运行熟悉的任务。首先，从根项目开始：

```java
$ gradle tasks
...
Other tasks
-----------
sayHello
whoami
...

```

然后，从仓库项目：

```java
$ gradle repository:tasks
...
Other tasks
-----------
whoami
...

```

```java
allprojects block (the closure being passed to allprojects, to be technically correct) gets applied to all the projects. The task's action prints the name of the project using the project object reference. Remember that the project object will refer to different projects depending on the project on which the task is being called. This happens because in the configuration phase, the allproject block is executed for each project once we have the project reference for that project.
```

传递给 `allprojects` 的闭包中的内容将完全像一个单项目 `build.gradle` 文件。我们甚至可以应用插件、声明仓库和依赖项等等。所以，本质上，我们可以编写适用于所有项目的任何通用构建逻辑，然后它将被应用到所有项目上。`allprojects` 方法也可以用来查询当前构建中的项目对象。有关 `allprojects` 的更多详细信息，请参阅项目的 API。

如果我们将 `--all` 标志传递给 `tasks` 任务，我们将看到 `whoami` 任务存在于所有子项目中，包括 `root` 项目：

```java
$ gradle tasks --all
...
Other tasks
-----------
sayHello
whoami
repository:whoami
services:whoami
web-app:whoami
...

```

如果我们只想在特定的项目上执行 `whoami`，比如说 `:repository`，那么命令就像以下这样简单：

```java
$ gradle -q repository:whoami
I am repository

```

当我们在没有任何项目路径的情况下执行 `whoami`：

```java
$ gradle -q whoami
I am root
I am repository
I am services
I am web-app

```

哇，Gradle 走得更远，以确保当我们从父项目执行任务时，具有相同名称的子项目任务也会被执行。当我们考虑像 `assemble` 这样的任务时，这非常有用，因为我们实际上希望所有子项目都进行组装，或者测试，它测试根项目以及子项目。

然而，关于仅在根项目上执行任务怎么办？这确实是一个有效场景。记住绝对任务路径：

```java
$ gradle -q :whoami
I am root

```

冒号起着至关重要的作用。在这里，我们只引用`root`项目的`whoami`。没有其他任务匹配相同的路径。例如，`repository`的`whoami`有一个路径`repository:whoami`。

现在，切换到`repository`目录，然后执行`whoami`：

```java
$ gradle –q whoami
I am repository

```

因此，任务执行是上下文相关的。在这里，默认情况下，Gradle 假设任务必须在当前项目上调用。不错，不是吗？

让我们在现有的`build.gradle`文件中添加一些更动态的代码：

```java
allprojects {
  task("describe${project.name.capitalize()}") << {
    println project.name
  }
}
```

在这里，根据项目名称，我们将任务名称设置为`describe`，并在项目名称前加上前缀。因此，所有项目都得到了它们自己的任务，但名称不会相同。我们添加了一个仅打印项目名称的操作。如果我们现在在我们的项目上执行`tasks`，我们可以看到任务名称包括项目名称：

```java
$ gradle tasks 
...
Other tasks
-----------
describeRepository
describeSample1
describeServices
describeWeb-app
sayHello
whoami
...
```

尽管这个例子非常简单，但我们学到了一些东西。首先，`allprojects`块是累加的，就像 Gradle 中的大多数其他方法一样。我们添加了第二个`allprojects`块，并且两者都运行得很好。其次，任务名称可以动态分配，例如，使用项目名称。

现在，我们可以从项目根目录调用任何`describe*`任务。正如我们可能猜测的那样，任务名称是唯一的；我们不需要在项目路径前加上前缀：

```java
$ gradle -q describeServices 
services

```

让我们切换到`repository`目录并列出任务：

```java
$ gradle -q tasks 
...
Other tasks
-----------
describeRepository
whoami

```

我们只看到适用于当前项目的任务，即`repository`。

## 将构建逻辑应用于子项目

让我们继续我们的例子。在这里，根项目将没有任何源集，因为所有的 Java 代码都将位于三个子项目中的任何一个。因此，只将`java`插件应用于子项目不是明智的吗？这正是`subprojects`方法发挥作用的地方，即当我们只想在子项目上应用一些构建逻辑而不影响父项目时。它的用法类似于`allprojects`。让我们将`java`插件应用于所有子项目：

```java
subprojects {
  apply plugin: 'java'
}
```

现在，运行`gradle tasks`应该会显示由`java`插件添加的任务。尽管这些任务可能看起来在根项目中可用，但实际上并非如此。在这种情况下，检查`gradle -q tasks --all`的输出。在子项目上存在的任务可以从根项目调用，但这并不意味着它们在根项目中存在。由`java`插件添加的任务仅可在子项目中使用，而如帮助任务这样的任务将在所有项目中可用。

## 子项目依赖

在本章的开头，我们提到一个子项目可能依赖于另一个子项目（或多个子项目），就像它可以依赖于外部库依赖一样。例如，`services`项目的编译依赖于`repository`项目，这意味着我们需要`repository`项目的编译类在`services`项目的编译类路径上可用。

要实现这一点，我们当然可以在`services`项目中创建一个`build.gradle`文件并将依赖声明放在那里。然而，为了展示一种替代方法，我们将这个声明放在`root`项目的`build.gradle`中。

与`allprojects`或`subprojects`不同，我们需要一个更精细的机制来仅从`root`项目的`build.gradle`中配置单个项目。实际上，使用`project`方法非常简单。此方法除了接受一个闭包，就像`allprojects`和`subprojects`方法一样，还需要指定一个项目名称，该闭包将应用于该项目。在配置阶段，闭包将在该项目的对象上执行。

因此，让我们将其添加到`root`项目的`build.gradle`中：

```java
project(':services') {
  dependencies {
    compile project(':repository')
  }
}
```

在这里，我们只为`services`项目配置依赖项。在`dependencies`块中，我们声明`:repository`项目是`services`项目的编译时依赖项。这基本上类似于外部库声明；我们使用`project(:sub-project)`来引用子项目，而不是在`group-id:artifact-id:version`表示法中使用库名称。

我们之前也说过`web-app`项目依赖于`services`项目。所以这次，让我们使用`web-app`自己的`build.gradle`来声明这个依赖项。我们将在`web-app`目录中创建一个`build.gradle`文件：

```java
root
├── build.gradle
├── settings.gradle
├── repository
├── services
└── web-app
 └── build.gradle

```

由于这是一个特定于项目的构建文件，我们只需像在其他任何项目中一样添加`dependencies`块即可：

```java
dependencies {
  compile project(':services')
}
```

现在，让我们使用`dependencies`任务可视化 Web 项目的依赖关系：

```java
$ gradle -q web-app:dependencies

------------------------------------------------------------
Project :web-app
------------------------------------------------------------

archives - Configuration for archive artifacts.
No dependencies

compile - Compile classpath for source set 'main'.
\--- project :services
 \--- project :repository

default - Configuration for default artifacts.
\--- project :services
 \--- project :repository

runtime - Runtime classpath for source set 'main'.
\--- project :services
 \--- project :repository

testCompile - Compile classpath for source set 'test'.
\--- project :services
 \--- project :repository

testRuntime - Runtime classpath for source set 'test'.
\--- project :services
 \--- project :repository

```

Gradle 向我们展示了`web-app`在不同配置下的依赖关系。我们还可以清楚地看到 Gradle 理解了传递依赖；因此，它显示了`web-app`通过`services`传递依赖于`repository`。请注意，我们实际上并没有在任何项目中声明任何外部依赖项（如`servlet-api`），否则它们也会在这里显示。

值得注意的是，查看`project`对象上`configure`方法的变体，以便过滤和配置选定的项目。有关`configure`方法的更多信息，请参阅[`docs.gradle.org/current/javadoc/org/gradle/api/Project.html`](https://docs.gradle.org/current/javadoc/org/gradle/api/Project.html)。

# 摘要

在这一简短的章节中，我们了解到 Gradle 支持灵活的目录结构，适用于复杂的项目层次结构，并允许我们为我们的构建选择合适的结构。然后我们探讨了在多项目构建的背景下`settings.gradle`的重要性。接着我们看到了将构建逻辑应用于所有项目、子项目或单个项目的各种方法。最后，我们举了一个关于项目间依赖关系的小例子。

在 Gradle 语法方面，我们只需要关注这些。接下来几章将主要关注各种插件为我们构建添加的功能以及如何配置它们。
