# 第一章：定义依赖

当我们开发软件时，我们需要编写代码。我们的代码由包含类的包组成，而这些类可能依赖于我们项目中其他类和包。对于单个项目来说，这是可以的，但有时我们会依赖于我们未开发的项目的类，例如，我们可能想使用 Apache Commons 库中的类，或者我们可能正在开发一个更大的多项目应用程序的一部分，并且我们依赖于这些其他项目中的类。

大多数时候，当我们编写软件时，我们希望使用项目外的类。实际上，我们对这些类有依赖。这些依赖的类大多存储在归档文件中，例如**Java 归档**（**JAR**）文件。这样的归档文件由一个唯一的版本号标识，因此我们可以对具有特定版本的库有依赖。

在本章中，你将学习如何在 Gradle 项目中定义依赖项。我们将看到如何定义依赖项的配置。你将了解 Gradle 中的不同依赖类型以及如何在配置构建时使用它们。

# 声明依赖配置

在 Gradle 中，我们定义依赖配置来将依赖项分组在一起。依赖配置有一个名称和几个属性，例如描述，实际上是一种特殊的`FileCollection`类型。配置可以相互扩展，因此我们可以在构建文件中构建配置的层次结构。Gradle 插件也可以为我们添加新的配置，例如，Java 插件为我们添加了几个新的配置，如`compile`和`testRuntime`。然后，`compile`配置用于定义编译源树所需的依赖项。依赖配置是用`configurations`配置块定义的。在块内部，我们可以为我们的构建定义新的配置。所有配置都添加到项目的`ConfigurationContainer`对象中。

在下面的示例构建文件中，我们定义了两个新的配置，其中`traffic`配置扩展自`vehicles`配置。这意味着添加到`vehicles`配置中的任何依赖项也将在`traffic`配置中可用。我们还可以为我们的配置分配一个`description`属性，以便为文档目的提供有关配置的更多信息。以下代码展示了这一点：

```java
// Define new configurations for build.
configurations {

  // Define configuration vehicles.
  vehicles {
    description = 'Contains vehicle dependencies'
  }

  traffic {
    extendsFrom vehicles
    description = 'Contains traffic dependencies'
  }

}
```

要查看项目中可用的配置，我们可以执行`dependencies`任务。这个任务适用于每个 Gradle 项目。该任务会输出项目的所有配置和依赖。让我们为当前项目运行这个任务并检查输出：

```java
$ gradle -q dependencies

------------------------------------------------------------
Root project
------------------------------------------------------------

traffic - Contains traffic dependencies
No dependencies

vehicles - Contains vehicle dependencies
No dependencies

```

注意，我们可以在输出中看到我们的两个配置，`traffic`和`vehicles`。我们没有为这些配置定义任何依赖项，如输出所示。

Java 插件向项目中添加了一些配置，这些配置由 Java 插件的任务使用。让我们将 Java 插件添加到我们的 Gradle 构建文件中：

```java
apply plugin: 'java'
```

要查看添加了哪些配置，我们调用`dependencies`任务并查看输出：

```java
$ gradle -q dependencies

------------------------------------------------------------
Root project
------------------------------------------------------------

archives - Configuration for archive artifacts.
No dependencies

compile - Compile classpath for source set 'main'.
No dependencies

default - Configuration for default artifacts.
No dependencies

runtime - Runtime classpath for source set 'main'.
No dependencies

testCompile - Compile classpath for source set 'test'.
No dependencies

testRuntime - Runtime classpath for source set 'test'.
No dependencies

```

只需添加 Java 插件，我们项目中就出现了六个配置。`archives`配置用于组合我们项目创建的工件。其他配置用于组合我们项目的依赖。在以下表中，总结了依赖配置：

| 名称 | 扩展 | 描述 |
| --- | --- |
| compile | none | 这些是编译依赖。 |
| runtime | compile | 这些是运行时依赖。 |
| testCompile | compile | 这些是编译测试的额外依赖。 |
| testRuntime | runtime, testCompile | 这些是运行测试的额外依赖。 |
| default | runtime | 这些是本项目使用的依赖以及本项目创建的工件。 |

在本章的后面部分，我们将看到如何与分配给配置的依赖项一起工作。在下一节中，我们将学习如何声明我们项目的依赖项。

# 声明依赖

我们定义了配置或应用了一个添加新配置到我们项目的插件。然而，除非我们向配置添加依赖，否则配置是空的。为了在 Gradle 构建文件中声明依赖，我们必须添加`dependencies`配置块。配置块将包含我们依赖的定义。在以下示例 Gradle 构建文件中，我们定义了`dependencies`块：

```java
// Dependencies configuration block.
dependencies {
    // Here we define our dependencies.
}
```

在配置块内部，我们使用依赖配置的名称后跟我们的依赖描述。依赖配置的名称可以在构建文件中明确定义，也可以由我们使用的插件添加。在 Gradle 中，我们可以定义多种类型的依赖。在以下表中，我们将看到我们可以使用的不同类型：

| 依赖类型 | 描述 |
| --- | --- |
| 外部模块依赖 | 这是对外部模块或库的依赖，可能存储在存储库中。 |
| 客户端模块依赖 | 这是外部模块的依赖，其中工件存储在存储库中，但模块的元信息在构建文件中。我们可以使用此类型的依赖来覆盖元信息。 |
| 项目依赖 | 这是同一构建中另一个 Gradle 项目的依赖。 |
| 文件依赖 | 这是本地计算机上文件集合的依赖。 |
| Gradle API 依赖 | 这是当前 Gradle 版本的 Gradle API 的依赖。我们在开发 Gradle 插件和任务时使用此依赖。 |
| 本地 Groovy 依赖 | 这是当前 Gradle 版本使用的 Groovy 库的依赖。我们在开发 Gradle 插件和任务时使用此依赖。 |

## 外部模块依赖

外部模块依赖是项目中最常见的依赖。这些依赖指的是外部仓库中的一个模块。在本书的后续部分，我们将了解更多关于仓库的信息，但基本上，仓库是将模块存储在中央位置的地方。一个模块包含一个或多个工件和元信息，例如它所依赖的其他模块的引用。

我们可以使用两种表示法在 Gradle 中定义外部模块依赖。我们可以使用字符串表示法或映射表示法。使用映射表示法，我们可以使用所有可用的依赖属性。字符串表示法允许我们设置属性的一个子集，但具有非常简洁的语法。

在以下示例 Gradle 构建文件中，我们使用字符串表示法定义了几个依赖项：

```java
// Define dependencies.
dependencies {
  // Defining two dependencies.
  vehicles 'com.vehicles:car:1.0', 'com.vehicles:truck:2.0'

  // Single dependency.
  traffic 'com.traffic:pedestrian:1.0'
}
```

字符串表示法具有以下格式：**moduleGroup:moduleName:version**。在第一个冒号之前使用模块组名，然后是模块名，最后是版本。

如果我们使用映射表示法，我们将显式使用属性的名称并设置每个属性的值。让我们重写我们之前的示例构建文件并使用映射表示法：

```java
// Compact definition of configurations.
configurations {
  vehicles
  traffic.extendsFrom vehicles
}

// Define dependencies.
dependencies {
  // Defining two dependencies.
  vehicles(
    [group: 'com.vehicles', name: 'car', version: '1.0'],
    [group: 'com.vehicles', name: 'truck', version: '2.0'],
  )

  // Single dependency.
  traffic group: 'com.traffic', name: 'pedestrian', version: '1.0'
}
```

我们可以使用映射表示法指定额外的配置属性，或者添加一个额外的配置闭包。外部模块依赖的一个属性是`transitive`属性。我们在第三章中学习如何与传递依赖一起工作，*解决依赖*。在下一个示例构建文件中，我们将使用映射表示法和配置闭包设置此属性：

```java
dependencies {
  // Use transitive attribute in map notation.
  vehicles group: 'com.vehicles', name: 'car',
      version: '1.0', transitive: false

  // Combine map notation with configuration closure.
  vehicles(group: 'com.vehicles', name: 'car', version: '1.0') {
    transitive = true
  }

  // Combine string notation with configuration closure.
  traffic('com.traffic:pedestrian:1.0') {
    transitive = false
  }
}
```

在本节的其余部分，你将了解你可以用来配置依赖项的更多属性。

Gradle 的一个优点是，我们可以在构建文件中编写 Groovy 代码。这意味着我们可以定义方法和变量，并在 Gradle 文件的其它部分使用它们。这样，我们甚至可以对构建文件进行重构，并创建可维护的构建脚本。请注意，在我们的示例中，我们包含了具有`com.vehicles`组名的多个依赖项。值被定义了两次，但我们也可以在依赖项配置中创建一个新的变量，并使用该变量的组名和引用。我们在构建文件内的`ext`配置块中定义变量。我们在 Gradle 中使用`ext`块向对象添加额外的属性，例如我们的项目。

以下示例代码定义了一个额外的变量来保存组名：

```java
// Define project property with
// dependency group name 'com.vehicles'
ext {
  groupNameVehicles = 'com.vehicles'
}

dependencies {
  // Using Groovy string support with
  // variable substition.
  vehicles "$groupNameVehicles:car:1.0"

  // Using map notation and reference
  // property groupNameVehicles.
  vehicles group: groupNameVehicles, name: 'truck', version: '2.0'
}
```

如果我们定义外部模块依赖项，那么 Gradle 会尝试在存储库中找到一个模块描述符。如果模块描述符可用，它将被解析以查看需要下载哪些工件。此外，如果模块描述符包含有关模块所需依赖项的信息，那么这些依赖项也将被下载。有时，一个依赖项在存储库中没有描述符，这时 Gradle 才会下载该依赖项的工件。

基于 Maven 模块的依赖项仅包含一个工件，因此 Gradle 很容易知道要下载哪个工件。但对于 Gradle 或 Ivy 模块，这并不明显，因为一个模块可以包含多个工件。该模块将具有多个配置，每个配置都有不同的工件。Gradle 将为这些模块使用名为`default`的配置。因此，与`default`配置关联的任何工件和依赖项都将被下载。然而，可能`default`配置不包含我们需要的工件。因此，我们可以为依赖项配置指定`configuration`属性以指定我们需要的特定配置。

以下示例为依赖项配置定义了一个`configuration`属性：

```java
dependencies {
  // Use the 'jar' configuration defined in the
  // module descriptor for this dependency.
  traffic group: 'com.traffic', 
      name: 'pedestrian', 
      version: '1.0',
      configuration: 'jar'

}
```

当一个依赖项没有模块描述符时，Gradle 只会下载工件。如果我们只想下载具有描述符的模块的工件，而不是任何依赖项，我们可以使用仅工件符号。或者，如果我们想从存储库中下载另一个存档文件，例如带有文档的 TAR 文件，我们也可以使用仅工件符号。

要使用仅工件符号，我们必须将文件扩展名添加到依赖项定义中。如果我们使用字符串符号，我们必须在版本后添加一个以`@`符号为前缀的扩展名。使用映射符号时，我们可以使用`ext`属性来设置扩展名。如果我们定义我们的依赖项为仅工件，Gradle 将不会检查是否为该依赖项提供了模块描述符。在下一个构建文件中，我们将看到不同工件仅符号的示例：

```java
dependencies {
  // Using the @ext notation to specify
  // we only want the artifact for this
  // dependency.
  vehicles 'com.vehicles:car:2.0@jar'

  // Use map notation with ext attribute
  // to specify artifact only dependency.
  traffic group: 'com.traffic', name: 'pedestrian',
      version: '1.0', ext: 'jar'

  // Alternatively we can use the configuration closure.
  // We need to specify an artifact configuration closure
  // as well to define the ext attribute.
  vehicles('com.vehicles:car:2.0') {
    artifact {
      name = 'car-docs'
      type = 'tar'
      extension = 'tar'
    }
  }
}
```

Maven 模块描述符可以使用分类器来指定工件。这通常用于为不同的 Java 版本编译具有相同代码的库时，例如，使用`jdk15`和`jdk16`分类器为 Java 5 和 Java 6 编译一个库。当我们定义外部模块依赖关系以指定我们想要使用的分类器时，我们可以使用`classifier`属性。此外，我们还可以在字符串或映射符号中使用它。使用字符串符号时，我们在版本属性后添加一个额外的冒号并指定分类器。对于映射符号，我们可以添加`classifier`属性并指定我们想要的值。以下构建文件包含了一个具有分类器的依赖项不同定义的示例：

```java
dependencies {
  // Using string notation we can
  // append the classifier after
  // the version attribute, prefixed
  // with a colon.
  vehicles 'com.vehicles:car:2.0:jdk15'

  // With the map notation we simply use the
  // classifier attribute name and the value.
  traffic group: 'com.traffic', name: 'pedestrian',
      version: '1.0', classifier: 'jdk16'

  // Alternatively we can use the configuration closure.
  // We need to specify an artifact configuration closure
  // as well to define the classifier attribute.
  vehicles('com.vehicles:truck:2.0') {
    artifact {
      name = 'truck'
      type = 'jar'
      classifier = 'jdk15'
    }
  }
}
```

在以下部分，我们将看到如何在我们的构建文件中定义客户端模块依赖项。

## 定义客户端模块依赖项

当我们定义外部模块依赖项时，我们期望有一个包含有关这些工件及其依赖项信息的模块描述符文件。Gradle 将解析此文件并确定需要下载的内容。请记住，如果此类文件不可用，它将被下载。然而，如果我们想覆盖模块描述符或在没有它的情况下提供它怎么办？在我们提供的模块描述符中，我们可以自己定义模块的依赖项。

我们可以在 Gradle 中通过客户端模块依赖来实现这一点。我们不是依赖于存储库中的模块描述符，而是在构建文件中本地定义自己的模块描述符。现在，我们可以完全控制我们认为模块应该是什么样子，以及模块本身有哪些依赖。我们使用 `module` 方法来定义一个客户端模块依赖项，用于依赖配置。

在以下示例构建文件中，我们将为依赖项 `car` 编写一个客户端模块依赖项，并将传递依赖项添加到 `driver`：

```java
dependencies {
  // We use the module method to instruct
  // Gradle to not look for the module descriptor
  // in a repository, but use the one we have
  // defined in the build file.
  vehicles module('com.vehicles:car:2.0') {
    // Car depends on driver.
    dependency('com.traffic:driver:1.0')
  }
}
```

## 使用项目依赖项

项目可以是更大、多项目构建的一部分，项目之间可以相互依赖，例如，一个项目可以是另一个项目的生成工件（包括另一个项目的传递依赖项）的依赖项。为了定义这样的依赖项，我们在依赖配置块中使用 `project` 方法。我们指定项目名称作为参数。我们还可以定义我们依赖的另一个项目的依赖配置的名称。默认情况下，Gradle 将查找默认依赖配置，但使用 `configuration` 属性，我们可以指定要使用的特定依赖配置。

以下示例构建文件将定义对 `car` 和 `truck` 项目的项目依赖项：

```java
dependencies {
  // Use project method to define project
  // dependency on car project.
  vehicles project(':car')

  // Define project dependency on truck
  // and use dependency configuration api
  // from that project.
  vehicles project(':truck') {
    configuration = 'api'
  }

  // We can use alternative syntax
  // to specify a configuration.
  traffic project(path: ':pedestrian',
          configuration: 'lib')
}
```

## 定义文件依赖项

我们可以直接在 Gradle 中向依赖配置添加文件。这些文件不需要存储在存储库中，但必须可以从项目目录中访问。尽管大多数项目将在存储库中存储模块描述符，但可能存在一个遗留项目依赖于公司共享网络驱动器上可用的文件。否则，我们必须在我们的项目中使用一个库，而这个库在任何存储库中都是不可用的。要向我们的依赖配置添加文件依赖项，我们使用 `files` 和 `fileTree` 方法指定一个文件集合。以下示例构建文件显示了所有这些方法的用法：

```java
dependencies {
  // Define a dependency on explicit file(s).
  vehicles files(
    'lib/vehicles/car-2.0.jar',
    'lib/vehicles/truck-1.0.jar'
  )

  // We can use the fileTree method to include
  // multiples from a directory and it's subdirectories.
  traffic fileTree(dir: 'deps', include: '*.jar')
}
```

如果我们将项目工件发布到存储库，添加的文件将不会成为我们项目的传递依赖项，但如果我们的项目是多项目构建的一部分，则它们将是。

## 使用内部 Gradle 和 Groovy 依赖项

当我们编写代码以扩展 Gradle，例如自定义任务或插件时，我们可以对 Gradle API 和当前 Gradle 版本使用的 Groovy 库有依赖。我们可以在依赖配置中使用 `gradleApi` 和 `localGroovy` 方法来确保所有正确的依赖项。

如果我们正在编写一些用于扩展 Gradle 的 Groovy 代码，但未使用任何 Gradle API 类，我们可以使用 `localGroovy`。使用这个方法，当前 Gradle 版本中提供的 Groovy 版本的类和库被添加为依赖项。以下示例构建脚本使用 Groovy 插件，并将依赖项添加到 Gradle 打包的 Groovy 的 `compile` 配置中：

```java
apply plugin: 'groovy'

dependencies {
  // Define dependency on Groovy
  // version shipped with Gradle.
  compile localGroovy()
}
```

当我们为 Gradle 编写自定义任务或插件时，我们依赖于 Gradle API。我们需要导入 API 的一些类来编写我们的代码。为了定义对 Gradle 类的依赖，我们使用 `gradleApi` 方法。这将包括为构建执行的 Gradle 版本包含的依赖项。下一个示例构建文件将展示这个方法的使用：

```java
apply plugin: 'groovy'

dependencies {
  // Define dependency on Gradle classes.
  compile gradleApi()
}
```

# 使用动态版本

到目前为止，我们已通过完整的版本号显式设置了一个依赖项的版本。要设置最低版本号，我们可以使用特殊的动态版本语法，例如，要将依赖项版本设置为至少 2.1，我们使用版本值 2.1.+. Gradle 将将依赖项解析为 2.1.0 版本之后的最新版本，或者解析为 2.1 版本本身。上限是 2.2\. 在以下示例中，我们将定义一个对至少为 4.0.x 版本的 spring-context 的依赖项：

```java
dependencies {
  compile 'org.springframework:spring-context:4.0.+'
}
```

要引用模块的最新发布版本，我们可以使用 `latest.integration` 作为版本值。我们还可以设置我们想要的最低和最高版本号。以下表格显示了 Gradle 中我们可以使用的范围：

| 范围 | 描述 |
| --- | --- |
| `[1.0, 2.0]` | 我们可以使用所有大于或等于 1.0 且小于或等于 2.0 的版本 |
| `[1.0, 2.0[` | 我们可以使用所有大于或等于 1.0 且小于 2.0 的版本 |
| `]1.0, 2.0]` | 我们可以使用所有大于 1.0 且小于或等于 2.0 的版本 |
| `]1.0, 2.0[` | 我们可以使用所有大于 1.0 且小于 2.0 的版本 |
| `[1.0, )` | 我们可以使用所有大于或等于 1.0 的版本 |
| `]1.0, )` | 我们可以使用所有大于 1.0 的版本 |
| `(, 2.0]` | 我们可以使用所有小于或等于 2.0 的版本 |
| `(, 2.0[` | 我们可以使用所有小于 2.0 的版本 |

在以下示例构建文件中，我们将 spring-context 模块的版本设置为大于 `4.0.1.RELEASE` 且小于 `4.0.4.RELEASE`：

```java
dependencies {
  // The dependency will resolve to version 4.0.3.RELEASE as
  // the latest version if available. Otherwise 4.0.2.RELEASE
  // or 4.0.1.RELEASE.
  compile 'org.springframework:spring-context:[4.0.1.RELEASE,4.0.4.RELEASE['
}
```

# 获取依赖信息

我们已经看到了如何在我们的构建脚本中定义依赖项。要获取有关我们依赖项的更多信息，我们可以使用 `dependencies` 任务。当我们调用此任务时，我们可以看到哪些依赖项属于我们项目的可用配置。此外，任何传递依赖项也会显示出来。下一个示例构建文件定义了对 Spring beans 的依赖项，并应用了 Java 插件。我们还在 `repositories` 配置块中指定了一个仓库。我们将在下一章中了解更多关于仓库的信息。以下代码捕捉了本段落的讨论：

```java
apply plugin: 'java'

repositories {
  // Repository definition for JCenter Bintray.
  // Needed to download artifacts. Repository
  // definitions are covered later.
  jcenter()
}

dependencies {
  // Define dependency on spring-beans library.
  compile 'org.springframework:spring-beans:4.0.+'
}
```

当我们执行 `dependencies` 任务时，我们得到以下输出：

```java
$ gradle -q dependencies

------------------------------------------------------------
Root project
------------------------------------------------------------

archives - Configuration for archive artifacts.
No dependencies

compile - Compile classpath for source set 'main'.
\--- org.springframework:spring-beans:4.0.+ -> 4.0.6.RELEASE
 \--- org.springframework:spring-core:4.0.6.RELEASE
 \--- commons-logging:commons-logging:1.1.3

default - Configuration for default artifacts.
\--- org.springframework:spring-beans:4.0.+ -> 4.0.6.RELEASE
 \--- org.springframework:spring-core:4.0.6.RELEASE
 \--- commons-logging:commons-logging:1.1.3

runtime - Runtime classpath for source set 'main'.
\--- org.springframework:spring-beans:4.0.+ -> 4.0.6.RELEASE
 \--- org.springframework:spring-core:4.0.6.RELEASE
 \--- commons-logging:commons-logging:1.1.3

testCompile - Compile classpath for source set 'test'.
\--- org.springframework:spring-beans:4.0.+ -> 4.0.6.RELEASE
 \--- org.springframework:spring-core:4.0.6.RELEASE
 \--- commons-logging:commons-logging:1.1.3

testRuntime - Runtime classpath for source set 'test'.
\--- org.springframework:spring-beans:4.0.+ -> 4.0.6.RELEASE
 \--- org.springframework:spring-core:4.0.6.RELEASE
 \--- commons-logging:commons-logging:1.1.3

```

我们看到我们项目的所有配置，并且对于每个配置，我们看到定义的依赖项及其传递依赖项。此外，我们还可以看到我们的动态版本 `4.0.+` 如何解析为版本 `4.0.6.RELEASE`。要仅查看特定配置的依赖项，我们可以在 `dependencies` 任务中使用 `--configuration` 选项。我们必须使用我们想要查看依赖项的配置的值。在以下输出中，我们看到当我们只想查看编译配置的依赖项时，结果如下：

```java
$ gradle -q dependencies --configuration compile

------------------------------------------------------------
Root project
------------------------------------------------------------

compile - Compile classpath for source set 'main'.
\--- org.springframework:spring-beans:4.0.+ -> 4.0.6.RELEASE
 \--- org.springframework:spring-core:4.0.6.RELEASE
 \--- commons-logging:commons-logging:1.1.3

```

在 Gradle 中还有一个名为 `dependencyInsight` 的孵化任务。由于它是孵化状态，Gradle 的功能或语法可能在未来的版本中发生变化。使用 `dependencyInsight` 任务，我们可以找出为什么特定的依赖项在我们的构建中，以及它属于哪个配置。我们必须使用 `--dependency` 选项，即必需的选项，并包含依赖项名称的一部分。Gradle 将查找包含 `--dependency` 选项指定的值的部分组、名称或版本的依赖项。可选地，我们可以指定 `--configuration` 选项，仅在该指定的配置中查找依赖项。如果我们省略此选项，Gradle 将在我们的项目的所有配置中查找依赖项。

让我们调用 `dependencyInsight` 任务来查找名称中包含 Spring 且在运行时配置中的依赖项：

```java
$ gradle -q dependencyInsight --dependency spring --configuration runtime
org.springframework:spring-beans:4.0.6.RELEASE

org.springframework:spring-beans:4.0.+ -> 4.0.6.RELEASE
\--- runtime

org.springframework:spring-core:4.0.6.RELEASE
\--- org.springframework:spring-beans:4.0.6.RELEASE
 \--- runtime

```

在输出中，我们看到版本 `4.0.+` 被解析为 `4.0.6.RELEASE`。我们还看到 `spring-beans` 依赖项和传递的 `spring-core` 依赖项是运行时配置的一部分。

## 访问依赖项

要访问配置，我们可以使用 Gradle 项目对象的 `configurations` 属性。`configurations` 属性包含一个 `Configuration` 对象的集合。记住，`Configuration` 对象是 `FileCollection` 的一个实例。因此，我们可以在允许 `FileCollection` 的构建脚本中引用 `Configuration`。`Configuration` 对象包含更多我们可以用来访问属于配置的依赖项的属性。

在下一个示例构建中，我们将定义两个任务，这些任务将使用项目配置中可用的文件和信息：

```java
configurations {
  vehicles
  traffic.extendsFrom vehicles
}

task dependencyFiles << {
  // Loop through all files for the dependencies
  // for the traffic configuration, including
  // transitive dependencies.
  configurations.traffic.files.each { file ->
    println file.name
  }

  // We can also filter the files using
  // a closure. For example to only find the files
  // for dependencies with driver in the name.
  configurations.vehicles.files { dep ->
    if (dep.name.contains('driver')) {
      println dep.name
    }
  }

  // Get information about dependencies only belonging
  // to the vehicles configuration.
  configurations.vehicles.dependencies.each { dep ->
    println "${dep.group} / ${dep.name} / ${dep.version}"
  }

  // Get information about dependencies belonging
  // to the traffice configuration and
  // configurations it extends from.
  configurations.traffic.allDependencies.each {  dep ->
    println "${dep.group} / ${dep.name} / ${dep.version}"
  }
}

task copyDependencies(type: Copy) {
  description = 'Copy dependencies from configuration traffic to lib directory'

  // Configuration can be the source for a CopySpec.
  from configurations.traffic

  into "$buildDir/lib"
}
```

## 构建脚本依赖项

当我们定义依赖时，我们通常希望为正在开发的代码定义它们。然而，我们可能还想将依赖添加到 Gradle 构建脚本本身。我们可以在构建文件中编写代码，这些代码可能依赖于 Gradle 分发中未包含的库。假设我们想在构建脚本中使用 Apache Commons Lang 库中的一个类。我们必须在我们的构建脚本中添加一个 `buildscript` 配置闭包。在配置闭包内，我们可以定义仓库和依赖。我们必须使用特殊的 `classpath` 配置来添加依赖。任何添加到 `classpath` 配置的依赖都可以在我们的构建文件中的代码中使用。

让我们通过一个示例构建文件来看看它是如何工作的。我们想在 `randomString` 任务中使用 `org.apache.commons.lang3.RandomStringUtils` 类。这个类可以在 `org.apache.commons:commons-lang3` 依赖中找到。我们将它定义为 `classpath` 配置的外部依赖。我们还在 `buildscript` 配置块中包含了一个仓库定义，以便可以下载依赖。以下代码展示了这一点：

```java
buildscript {
  repositories {
    // Bintray JCenter repository to download
    // dependency commons-lang3.
    jcenter()
  }

  dependencies {
    // Extend classpath of build script with
    // the classpath configuration.
    classpath 'org.apache.commons:commons-lang3:3.3.2'
  }
}

// We have add the commons-lang3 dependency
// as a build script dependency so we can
// reference classes for Apache Commons Lang.
import org.apache.commons.lang3.RandomStringUtils

task randomString << {
  // Use RandomStringUtils from Apache Commons Lang.
  String value = RandomStringUtils.randomAlphabetic(10)
  println value
}
```

为了包含 Gradle 分发之外的插件，我们也可以在 `buildscript` 配置块中使用 `classpath` 配置。在下一个示例构建文件中，我们将包含 `Asciidoctor` Gradle 插件：

```java
buildscript {
  repositories {
    // We need the repository definition, from
    // where the dependency can be downloaded.
    jcenter()
  }

  dependencies {
    // Define external module dependency for the Gradle
    // Asciidoctor plugin.
    classpath 'org.asciidoctor:asciidoctor-gradle-plugin:0.7.3'
  }
}

// We defined the dependency on this external
// Gradle plugin in the buildscript {...}
// configuration block
apply plugin: 'org.asciidoctor.gradle.asciidoctor'
```

# 可选 Ant 任务依赖

我们可以在 Gradle 中重用现有的 Ant 任务。Ant 的默认任务可以从我们的构建脚本中调用。然而，如果我们想使用可选的 Ant 任务，我们必须为可选 Ant 任务所需的类定义一个依赖。我们创建一个新的依赖配置，然后向这个新配置添加一个依赖。我们可以在设置可选 Ant 任务的类路径时引用这个配置。

让我们添加可选的 Ant 任务 SCP，用于安全地将文件从远程服务器复制到/从服务器。我们创建 `sshAntTask` 配置以添加可选 Ant 任务的依赖。我们可以为配置选择任何名称。为了定义可选任务，我们使用来自内部 `ant` 对象的 `taskdef` 方法。该方法需要一个 `classpath` 属性，它必须是 `sshAntTask` 依赖中所有文件的实际路径。`Configuration` 类提供了 `asPath` 属性，以平台特定的方式返回文件的路径。因此，如果我们使用 Windows 计算机上的这个属性，文件路径分隔符是 `;`，而对于其他平台则是 `:`。以下示例构建文件包含了定义和使用 SCP Ant 任务的全部代码：

```java
configurations {
  // We define a new dependency configuration.
  // This configuration is used to assign
  // dependencies to, that are needed by the
  // optional Ant task scp.
  sshAntTask
}

repositories {
  // Repository definition to download dependencies.
  jcenter()
}

dependencies {
  // Define external module dependencies
  // for the scp Ant task.
  sshAntTask(group: 'org.apache.ant', 
        name: 'ant-jsch', 
        version: '1.9.4')
}

// New task that used Ant scp task.
task copyRemote(
  description: 'Secure copy files to remote server') << {

  // Define optional Ant task scp.
  ant.taskdef(
    name: 'scp',
    classname: 'org.apache.tools.ant.taskdefs.optional.ssh.Scp',

    // Set classpath based on dependencies assigned
    // to sshAntTask configuration. The asPath property
    // returns a platform-specific string value
    // with the dependency JAR files.
    classpath: configurations.sshAntTask.asPath)

  // Invoke scp task we just defined.
  ant.scp(
    todir: 'user@server:/home/user/upload',
    keyFile: '${user.home}/.ssh/id_rsa',
    passphrase: '***',
    verbose: true) {
    fileset(dir: 'html/files') {
      include name: '**/**'
    }
  }
}
```

# 管理依赖

你在本章中已经了解到，我们可以通过将公共部分提取到项目属性中来重构依赖定义。这样，我们只需更改几个项目属性值，就可以对多个依赖项进行更改。在下一个示例构建文件中，我们将使用列表来分组依赖项，并从依赖定义中引用这些列表：

```java
ext {
  // Group is used multiple times, so
  // we extra the variable for re-use.
  def vehiclesGroup = 'com.vehicles'

  // libs will be available from within
  // the Gradle script code, like dependencies {...}.
  libs = [
    vehicles: [
      [group: vehiclesGroup, name: 'car', version: '2.0'],
      [group: vehiclesGroup, name: 'truck', version: '1.0']
    ],
    traffic: [
      [group: 'com.traffic', name: 'pedestrian', version: '1.0']
    ]
  ]
}

configurations {
  vehicles
}

dependencies {
  // Reference ext.libs.vehicles defined earlier
  // in the build script.
  vehicles libs.vehicles
}
```

Maven 有一个名为依赖管理元数据的功能，允许我们在构建文件的公共部分定义依赖项使用的版本。然后，当实际依赖项配置时，我们可以省略版本，因为它将从构建文件的依赖管理部分确定。Gradle 没有这样的内置功能，但如前所述，我们可以使用简单的代码重构来获得类似的效果。

我们仍然可以在我们的 Gradle 构建中使用 Maven 的声明式依赖管理，通过 Spring 的外部依赖管理插件。此插件向 Gradle 添加了一个`dependencyManagement`配置块。在配置块内部，我们可以定义依赖元数据，例如组、名称和版本。在我们的 Gradle 构建脚本中的`dependencies`配置闭包中，我们不再需要指定版本，因为它将通过`dependencyManagement`配置中的依赖元数据来解析。以下示例构建文件使用此插件并使用`dependencyManagement`指定依赖元数据：

```java
buildscript {
  repositories {
    // Specific repository to find and download
    // dependency-management-plugin.
    maven {
      url 'http://repo.spring.io/plugins-snapshot'
    }
  }
  dependencies {
    // Define external module dependency with plugin.
    classpath 'io.spring.gradle:dependency-management-plugin:0.1.0.RELEASE'
  }
}

// Apply the external plugin dependency-management.
apply plugin: 'io.spring.dependency-management'
apply plugin: 'java'

repositories {
  // Repository for downloading dependencies.
  jcenter()
}

// This block is added by the dependency-management
// plugin to define dependency metadata.
dependencyManagement {
  dependencies {
    // Specify group:name followed by required version.
    'org.springframework.boot:spring-boot-starter-web' '1.1.5.RELEASE'

    // If we have multiple module names for the same group
    // and version we can use dependencySet.
    dependencySet(group: 'org.springframework.boot',
          version: '1.1.5.RELEASE') {
      entry 'spring-boot-starter-web'
      entry 'spring-boot-starter-actuator'
    }
  }
}

dependencies {
  // Version is resolved via dependencies metadata
  // defined in dependencyManagement.
  compile 'org.springframework.boot:spring-boot-starter-web'
}
```

要导入一个组织提供的 Maven **物料清单**（**BOM**），我们可以在`dependencyManagement`配置中使用`imports`方法。在下一个示例中，我们将使用 Spring IO 平台的 BOM。在`dependencies`配置中，我们可以省略版本，因为它将通过 BOM 来解析：

```java
buildscript {
  repositories {
    // Specific repository to find and download
    // dependency-management-plugin.
    maven {
      url 'http://repo.spring.io/plugins-snapshot'
    }
  }
  dependencies {
    // Define external module dependency with plugin.
    classpath 'io.spring.gradle:dependency-management-plugin:0.1.0.RELEASE'
  }
}

// Apply the external plugin dependency-management.
apply plugin: 'io.spring.dependency-management'
apply plugin: 'java'

repositories {
  // Repository for downloading BOM and dependencies.
  jcenter()
}

// This block is added by the dependency-management
// plugin to define dependency metadata.
dependencyManagement {
  imports {
    // Use Maven BOM provided by Spring IO platform.
    mavenBom 'io.spring.platform:platform-bom:1.0.1.RELEASE'
  }
}

dependencies {
  // Version is resolved via Maven BOM.
  compile 'org.springframework.boot:spring-boot-starter-web'
}
```

# 摘要

在本章中，你学习了如何创建和使用依赖配置来分组依赖项。我们看到了如何定义几种类型的依赖项，例如外部模块依赖和内部依赖。

此外，我们还看到了如何在 Gradle 构建脚本中使用`classpath`配置和`buildscript`配置向代码中添加依赖项。

最后，我们探讨了使用代码重构和外部依赖管理插件定义依赖项的一些可维护的方法。

在下一章中，我们将学习如何配置存储依赖模块的仓库。
