# 第八章：组织构建逻辑和插件

插件是 Gradle 的主要构建块之一，我们之前并没有过多讨论。你已经看到了不同的标准插件，如 Java、Eclipse、Scala 等，它们都附带了一系列定义好的任务。开发者只需包含插件，配置所需的任务，就可以利用其功能。在本章中，我们将概述插件是什么，如何将任务分组到插件中，如何将插件逻辑从构建文件提取到 `buildSrc`，以及如何创建独立的插件。

# 将构建逻辑提取到 `buildSrc`

插件不过是按照特定顺序和默认配置创建的任务组，旨在提供某种功能。例如，`java` 插件包含提供构建 Java 项目功能的任务，`scala` 插件包含构建 Scala 项目的任务，等等。尽管 Gradle 提供了许多标准插件，但你也可以找到不同的第三方插件来满足项目的需求。可能总会有这样的情况，即你无法使用现有的插件找到所需的功能，并希望为你的定制需求创建一个新的插件。我们将探讨开发者可以创建和使用插件的不同方式。

用户可以创建的第一个插件就是构建文件本身。以下是一个插件的示例代码，开发者可以在 `build.gradle` 中编写并使用它：

```java
apply plugin: CustomPlugin

class CustomPlugin implements Plugin<Project> {
  void apply(Project project) {
    project.task('task1') << {
      println "Sample task1 in custom plugin"

    }
    project.task('task2') << {
      println "Sample task2 in custom plugin"

    }    
  }
}
task2.dependsOn task1
```

在这里，我们在构建文件本身中创建了一个插件。这是 Gradle 脚本之美。你还可以在 Gradle 文件中编写一个类。要创建自定义插件，你需要创建一个实现 `Plugin` 接口的 Groovy 类。你甚至可以用 Java 或任何其他 JVN 语言编写插件。由于 Gradle 构建脚本是用 Groovy 编写的，所以我们使用了 Groovy 来编写插件实现。你想要实现的所有任务，都需要在 `apply` 方法中定义。我们定义了两个任务，`task1` 和 `task2`。我们还定义了生命周期作为两个任务之间的关系。如果开发者调用 `task1`，只有 `task1` 将被执行。如果你执行 `task2`，`task1` 和 `task2` 都将执行。尝试执行以下命令：

```java
> gradle task2
:task1
Sample task1 in customer plugin
:task2
Sample task2 in custom plugin

BUILD SUCCESSFUL
Total time: 2.206 secs

```

### 小贴士

要在构建文件中使用插件，你始终需要使用 `apply plugin:<plugin name/plugin` class（如果插件是在同一脚本或 `buildSrc` 目录中实现的）。

这是一种开发者定义自定义插件简单的方式。然而，如果我们遵循设计原则，将构建逻辑和自定义逻辑混合到同一个文件中并不是一个好的实践。这将很难维护代码，也可能增加维护工作量。我们始终建议你将插件代码与构建逻辑分开编写。为了实现这一点，Gradle 提供了两种不同的方式，如下所示：

+   将插件代码提取到 `buildSrc`

+   独立插件

为了将插件代码提取到`buildSrc`，Gradle 建议您在项目目录内创建一个`buildSrc`目录，并将插件代码保存在那里。以下是该目录结构的示例：

```java
C:./Gradle/Chapter8/CustomPlugin1
│   build.gradle
│
└───buildSrc
 └───src
 └───main
 └───groovy
 └───ch8
 CustomPlugin.groovy

```

在这里，我们已经创建了一个单独的`buildSrc`目录；在该目录内，我们将插件代码保存在`CustomPlugin.groovy`文件中。将前面的 Groovy 类从`build.gradle`文件移动到这个文件中。在顶部包含包声明。您还需要导入`org.gradle.api.*`。您的`CustomPlugin.groovy`文件将如下所示：

```java
package ch8
import org.gradle.api.*
class CustomPlugin implements Plugin<Project> {
// Plugin functionality here
}
```

`build.gradle`文件的内容将如下所示：

```java
import ch8.CustomPlugin
apply plugin: CustomPlugin
```

您只需导入包并添加`apply plugin`语句。所有编译类和将类包含到运行时类路径中的后台工作将由 Gradle 执行。现在，尝试执行以下命令：

```java
> gradle task1
:buildSrc:compileJava UP-TO-DATE
:buildSrc:compileGroovy UP-TO-DATE
:buildSrc:processResources UP-TO-DATE
:buildSrc:classes UP-TO-DATE
:buildSrc:jar UP-TO-DATE
:buildSrc:assemble UP-TO-DATE
:buildSrc:compileTestJava UP-TO-DATE
:buildSrc:compileTestGroovy UP-TO-DATE
:buildSrc:processTestResources UP-TO-DATE
:buildSrc:testClasses UP-TO-DATE
:buildSrc:test UP-TO-DATE
:buildSrc:check UP-TO-DATE
:buildSrc:build UP-TO-DATE
:task1
Sample task1 in custom plugin

BUILD SUCCESSFUL

Total time: 3.374 secs

```

这里，您可以看到 Gradle 为您自定义的插件代码执行了编译和构建任务，现在您只需执行自定义插件中的任务。Gradle 还允许您在构建文件中配置自定义插件。您可以在任务之间设置依赖关系或向构建文件中的任务添加更多功能，而不是反复更新插件代码。如果您想为`task1`添加更多功能，可以按照以下方式操作：

```java
task1.doLast {
println "Added more functionality to task1"
}
task2.dependsOn task1
```

现在，如果您尝试执行`task1`，它将附加前面的语句。

以这种方式，您可以将构建逻辑从`build.gradle`文件分离出来，将其放置在`buildSrc`目录下的一个单独的类文件中。如果您有一个多项目构建，根项目`buildSrc`中定义的插件可以被所有子项目的构建文件重用。您不需要为每个子项目定义一个单独的插件。这个过程仍然有一个限制。它不允许您将此插件用于其他项目。由于它与当前项目紧密耦合，您只能使用此插件与同一项目或根项目中定义的子项目。为了克服这一点，您可以将插件代码提取出来，创建一个独立的插件，并将其打包成一个 JAR 文件，这样您就可以将其发布到仓库中，以便任何项目都可以重用它。在下一节中，我们将讨论独立插件。

# 第一个插件

为了使插件对所有其他项目可重用，Gradle 允许您将插件代码分离出来，并将其打包成一个 JAR 文件。您可以将此 JAR 文件包含在任何您想要重用此功能的项目中。您可以使用 Java 或 Groovy 创建独立项目。我们将使用 Groovy。您可以使用任何编辑器（Eclipse、NetBeans 或 Idea）来创建插件。由于我们的主要目的是向您展示如何创建独立插件，我们不会深入编辑器的细节。我们将使用一个简单的文本编辑器。要继续创建独立插件，将上述`buildSrc`代码分离到一个独立的目录中。您可以将其命名为`CustomPlugin`。因此，目录结构将如下所示：

```java
C:/Gradle/Chapter8/CustomPlugin.
│   build.gradle
│
└───src
 └───main
 └───groovy
 └───ch8
 CustomPlugin.groovy

```

你可能会惊讶地想知道为什么在这里创建 `build.gradle` 文件。通过这个 `build.gradle` 文件，我们将插件代码打包成一个 jar 文件。现在，问题出现了，那就是你将如何将这个插件包含到其他构建文件中。你需要为这个插件提供一个 **插件 ID**。为了给插件添加一个插件 ID，你需要在 `src/main/resources/META-INF/gradle-plugins` 目录中创建一个属性文件。属性文件的名字将是你的插件 ID。在这里，我们将在上述目录中添加 `customplugin.properties` 文件。这个文件的内容如下：

```java
implementation-class=ch8.CustomPlugin

Your build file content would be.

apply plugin: 'groovy'
version = '1.0'
dependencies {
  compile gradleApi()
  compile localGroovy()
}
```

要编译 Groovy 代码，你需要在编译配置中包含前面的两个语句。由于我们在这里使用的是一个普通的 Groovy 类，所以我们没有添加任何其他的依赖 JAR 文件。如果你的插件代码依赖于任何其他的第三方 JAR 文件，你可以在依赖中包含它们，并配置相应的仓库。

现在，我们将按照以下方式构建插件：

```java
> gradle clean build
:clean
:compileJava UP-TO-DATE
:compileGroovy
:processResources
:classes
:jar
:assemble
:compileTestJava UP-TO-DATE
:compileTestGroovy UP-TO-DATE
:processTestResources UP-TO-DATE
:testClasses UP-TO-DATE
:test UP-TO-DATE
:check UP-TO-DATE
:build

BUILD SUCCESSFUL
Total time: 4.671 secs

```

你可以在 `<project>/build/libs/CustomPlugin-1.0.jar` 中找到这个 JAR 文件。

你可以将这个插件 JAR 发布到组织的内部仓库中，这样其他项目可以直接从那里下载并使用它。现在，我们将创建另一个项目，并将这个插件 JAR 引用到那个项目中。

创建一个新的目录，`SampleProject`，并将 `build.gradle` 文件添加到项目中。现在，一个问题出现了，那就是你的 `build.gradle` 文件将如何引用 `SamplePlugin`。为此，你需要在 `buildscript closure` 中提及 `SamplePlugin` JAR 文件的位置，并在 `dependencies closure` 中添加对这个 JAR 文件的依赖。

你的 `build.gradle` 文件内容如下：

```java
buildscript {
repositories {
  flatDir {dirs "../CustomPlugin/build/libs/"}
}
dependencies {
classpath group: 'ch8', name: 'CustomPlugin',version: '1.0'
}
}
apply plugin: 'customplugin'
```

在这里，我们使用的是 `flat file repository`，因此使用 `flatDir` 配置来引用自定义插件 JAR 文件。我们建议你使用组织的本地仓库；这样，组织的任何项目都可以集中访问。在 `dependencies closure` 中，我们引用了 `CustomPlugin` JAR 文件。这是使用任何插件的前提。最后，我们添加了 `apply plugin` 语句，并用单引号提到了插件名称。

### 小贴士

插件名称是你创建在 `src/main/resources/META-INF/gradle-plugins` 目录中的属性文件的名称。

现在，你可以使用以下命令执行构建文件：

```java
> gradle task1

:task1
Sample task1 in custom plugin

BUILD SUCCESSFUL

Total time: 2.497 secs

```

# 配置插件

到目前为止，我们已经看到了如何创建一个独立的自定义插件并将其包含在另一个项目的构建文件中。Gradle 还允许你配置插件属性并根据项目需求进行自定义。你已经学习了如何在一个`java`插件中自定义源代码位置和测试代码位置。我们将看到一个示例，展示你如何在自定义插件中复制相同的行为。要定义插件属性，你需要创建一个额外的`extension`类并将该类注册到你的`plugin`类中。假设我们想向插件添加`location`属性，创建`CustomPluginExtension.groovy`类如下：

```java
package ch8
class CustomPluginExtension {
def location = "/plugin/defaultlocation"
}
```

现在，将此类注册到你的`plugin`类中：

```java
class CustomPlugin implements Plugin<Project> {
  void apply(Project project) {
    def extension = project.extensions.create("customExt",CustomPluginExtension)

project.task('task1') << {
      println "Sample task1 in custom plugin"
      println "location is "+project.customExt.location
  }
}
}
```

现在，再次构建插件，以确保你的更改成为最新插件 JAR 文件的一部分，然后尝试执行`SampleProject`的`build.gradle`：

```java
> gradle task1
:task1
Sample task1 in custom plugin
location is /plugin/defaultlocation

BUILD SUCCESSFUL

Total time: 2.79 secs

```

在这里，你可以看到命令行输出的默认值。如果你想将此字段更改为其他值，请将`customExt closure`添加到你的`SampleProject` `build.gradle`文件中，并为位置配置不同的值：

```java
buildscript {
repositories {
  flatDir {dirs "../CustomPlugin/build/libs/"}
}
dependencies {
  classpath group: 'ch8', name: 'CustomPlugin',version: '1.0'
}
}
apply plugin: 'customplugin'

customExt {
 location="/plugin/newlocation"
}

```

现在，再次尝试执行`task1`：

```java
> gradle task1
:task1
Sample task1 in custom plugin
location is /plugin/newlocation

BUILD SUCCESSFUL
Total time: 5.794 secs

```

在这里，你可以观察到位置属性的更新值。

# 摘要

在本章中，我们讨论了 Gradle 的主要构建块之一，插件。插件有助于组织和模块化功能，并有助于打包一系列相关的任务和配置。我们还讨论了创建自定义插件的不同方法，从在构建文件中编写插件代码到创建独立的插件 JAR 文件并在不同的项目中重用它。在最后一节中，我们还介绍了如何配置插件现有的属性并根据项目需求进行自定义。

在下一章结束本书之前，我们将讨论如何借助 Gradle 构建 Groovy 和 Scala 项目。此外，鉴于这是一个移动时代，所有传统的软件或 Web 应用程序现在都在转向应用，我们还将讨论构建 Android 项目。
